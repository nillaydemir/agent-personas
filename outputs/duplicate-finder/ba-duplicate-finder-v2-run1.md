# Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) İş Analizi

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v2
- **Girdi:** Flutter ile yazacağımız ve cep telefonlarında çalışacak bir Photo Library duplicate finder uygulaması hazırlayacağız. Gerekli algoritmaları araştırıp bana bir iş analizi çıkar.

---

## 1. Özet

Ürün, telefonun fotoğraf kütüphanesindeki kopya ve kopyaya çok yakın fotoğrafları bulur, kullanıcıya gruplar halinde gösterir ve hangisinin saklanacağını önerir. Silme işlemi güvenlidir: fotoğraflar önce platformun geri alınabilir çöp kutusuna gider. Ürün küçük/orta ölçekli, tek amaçlı bir mobil uygulamadır. Hedef platformlar iOS ve Android'dir (Flutter görevde verilmiş bir kısıttır).

Araştırmanın ana bulguları:

- **Platformların yerleşik çözümleri var.** iOS Photos, iOS 16'dan beri kopyaları "Duplicates" albümünde topluyor. Ancak tespit için iPhone'un kilitli ve şarjda olması gerekiyor ve bu işlem birkaç gün sürebiliyor [K1]. Android'de Files by Google'ın bir "Duplicate files" kartı var; kartta orijinal dosya "Original" rozetiyle işaretleniyor [K3]. Ürünün farkı şunlar olabilir:
  - kullanıcı istediği anda tarama başlatabilir,
  - fotoğraflar yan yana karşılaştırılabilir,
  - gizliliğe saygılı, tamamen cihaz içi işleme.
- **Algoritma seçenekleri dört aileye ayrılıyor:** birebir (bayt düzeyi) hash, aHash, dHash/pHash ve gömme vektörleri (embedding). Bir yazarın 150.000 görsellik kendi testinde:
  - aHash hızlı ama çok sayıda yanlış eşleşme üretti,
  - pHash hiç hata yapmadı ama en yavaşıydı,
  - dHash aHash kadar hızlı çalıştı ve çok az yanlış eşleşme verdi [K14].

  Seçim TA'ya bırakılmıştır. İş açısından artı ve eksiler 3.4'tedir.
- **En büyük iş riski Google Play politikasıdır.** `READ_MEDIA_IMAGES` izni yalnızca ana işlevi fotoğraflara geniş erişim olan uygulamalara veriliyor ve Play Console'da beyan gerektiriyor. Politika örnek olarak "gallery apps" diyor; kopya/dosya temizleyiciler açıkça anılmıyor [K8].
- **İki platformda da kullanıcı yalnızca seçili fotoğraflara erişim verebiliyor** (iOS 14+, Android 14+) [K9][K12]. Ürün bu durumda da çalışmalı.

İlk sürümün (MVP) sınırını **M** öncelikli gereksinimler çizer. Bu sınır şunları kapsar: izin yönetimi, cihaz içi tarama, birebir ve görsel olarak neredeyse aynı kopyaların tespiti, karşılaştırma, saklanacak fotoğraf önerisi ve çöp kutusuna güvenli silme.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | İhtiyaç / acı noktası | Başarı ölçütü |
|---|---|---|---|
| **P1 – Depolama sıkışık kullanıcı** | Telefon depolaması dolmuş, teknik bilgisi az olan genel kullanıcı. | Hızlıca yer açmak. Hangi fotoğrafın güvenle silinebileceğinden emin olmak. | Az adımda yer açar ve yanlışlıkla fotoğraf kaybetmez. |
| **P2 – Yoğun fotoğraf çeken kullanıcı** | Kütüphanesi büyüktür (on binlerce öğe). Aynı fotoğrafın farklı çözünürlükte/biçimde kopyaları vardır. | Toplu işlem yapmak. Hangi kopyanın daha kaliteli olduğunu (çözünürlük, boyut, biçim) görmek. | Tüm grupları birkaç dokunuşla temizler. Önerilen "saklanacak" öğeye güvenir. |
| **P3 – Gizlilik hassasiyeti yüksek kullanıcı** | Fotoğraflarının bir sunucuya gitmesini istemez. Erişimi sınırlı tutabilir. | Fotoğrafların cihazdan çıkmadığından emin olmak. Yalnızca seçtiği albüm/fotoğraflarda çalışmak. | Sınırlı erişimle de anlamlı sonuç alır. Mağaza gizlilik beyanı "veri toplanmıyor" der. |

Not: Persona'lar görevden ve rakip kullanıcı yorumlarından [K4][K5] türetilmiştir. Pazar payı veya kullanıcı sayısı verisine dayanmazlar (bkz. V-07).

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakipler ve yerleşik çözümler

| Çözüm | Platform | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler |
|---|---|---|---|---|
| **iOS Photos – Duplicates** (yerleşik) | iOS 16+ | Kopya fotoğraf ve videoları "Utilities" altındaki Duplicates albümünde topluyor [K1]. Bir grup "Merge" ile birleştiriliyor [K2]. | Ücretsiz ve sisteme gömülü. Ek izin gerektirmiyor. | Tespit için "iPhone to be locked and connected to power" şartı var. Süre "up to a few days" olabiliyor [K1]. Kullanıcı taramayı ne zaman başlatacağını seçemiyor. Apple'ın kılavuz sayfası hangi kopyanın saklandığını ve kopya kriterini belirtmiyor [K2]. |
| **Files by Google – Duplicate files** (Google'ın uygulaması) | Android | "Clean" altında "Duplicate files" kartı var. Orijinal dosya "Original" rozetiyle işaretleniyor ve Google bu dosyanın saklanmasını öneriyor. Seçilen dosyalar onay diyaloğuyla Trash'e taşınıyor [K3]. | Orijinal açıkça işaretli. Silme geri alınabilir (Trash). | Yardım sayfası eşleştirmenin birebir mi yoksa görsel benzerlik mi olduğunu söylemiyor [K3]. Yalnızca fotoğraflara özel bir karşılaştırma arayüzü belgelenmemiş. |
| **CleanMy®Phone (eski Gemini Photos), MacPaw** | iOS | "finds duplicate and similar shots". Gruplardaki en iyi fotoğrafı vurguluyor. Kaydırarak silme/saklama (Swipe Mode) sunuyor. Mağaza sayfasına göre "works fully on-device" [K4]. | 23K oyla 4,6 puan (ABD App Store, 2026-11-22). Benzer fotoğraf ve "en iyi kare" önerisi var [K4]. | Abonelik modeli: ABD mağazasında yıllık 36,99 $ ve aylık 7,99 $. Geliştiricinin kendi beyanına göre gizlilik etiketinde "Data used to track you: Identifiers, Usage Data" var. Kullanıcı yorumlarında seçimin takıldığı ve taramanın haftalarca bitmediği bildiriliyor [K4]. |
| **Remo Duplicate Photos Remover** | iOS | Birebir kopyaları siliyor, benzer görselleri buluyor. Gruplarda işaretleme/işaret kaldırma, silmeden önce önizleme ve tarih/çözünürlük/boyut gösterimi var. Mağaza sayfası "Automatically keeps one best copy safe in each group" diyor [K5]. | Ücretsiz. Geliştirici beyanı "does not collect any data". 14K oyla 4,6 puan [K5]. | Kullanıcı yorumları: başka uygulamaya geçilince tarama devam etmiyor; büyük kütüphanede tarama uzun süre duraklıyor; karşılaştırma ekranında dosya türü yok; ekran görüntülerini "lumps them all as similars" [K5]. |

**Ürün için çıkarımlar** (rakip bulgularından türetilmiş iş yorumu):
- Yerleşik iOS çözümünün koşullara bağlı ve yavaş tespiti [K1], istendiği anda tarama yapmayı değerli bir fark yapıyor (FR-03).
- Rakiplerde ortak beklentiler var: yan yana karşılaştırma, meta veri gösterimi ve otomatik "saklanacak" önerisi [K3][K5]. Bunlar M öncelikli (FR-07, FR-08).
- Rakip yorumlarındaki şikâyetler doğrudan gereksinime dönüştürüldü: arka plana geçince taramanın kesilmesi ve dosya türünün gösterilmemesi [K5]. Bkz. FR-07 ve FR-12.
- Ekran görüntülerinin "benzer" sayılması bir kullanıcı şikâyeti [K5]. Bu nedenle "birebir/neredeyse aynı" kopyalar ile "benzer" fotoğraflar ayrı tutuldu: FR-05 ile FR-15 ayrımı.
- Rakiplerden birinde izleme verisi beyanı var [K4], diğeri "veri toplanmıyor" diyor [K5]. Gizlilik bir farklılaşma alanı (NFR-01).

### 3.2 Alan standartları ve düzenlemeler

| Alan | Bulgu | Ürüne etkisi |
|---|---|---|
| **KVKK (6698 sayılı Kanun)** | **Kişisel veri:** "Kimliği belirli veya belirlenebilir gerçek kişiye ilişkin her türlü bilgi" (md. 3/d).<br>**İşleme:** kaydetme, depolama, değiştirme ve "sınıflandırılması" dahil her türlü işlem (md. 3/e).<br>**Özel nitelikli veri:** biyometrik veriler bu kapsamda (md. 6/1). İşlenmeleri kural olarak yasak, sınırlı istisnalar var (md. 6/3) [K6]. | Fotoğraflar kişisel veri içerebilir. İşleme yalnızca cihazda kalsa da geliştiricinin "veri sorumlusu" sayılıp sayılmayacağı hukuki bir sorudur (AS-05). Yüz tanımayla gruplama biyometrik veri riski taşıdığı için kapsam dışıdır (Bölüm 7). |
| **Erişilebilirlik – WCAG 2.2** | **1.4.3:** metin kontrastı "at least 4.5:1".<br>**2.5.8:** dokunma hedefi "at least 24 by 24 CSS pixels" (Seviye AA).<br>**2.5.7:** sürükleme gerektiren her işlev "by a single pointer without dragging" da yapılabilmeli.<br>**1.1.1:** metin dışı içeriğe metin alternatifi [K7]. | Kaydırarak silme gibi hareketlere buton alternatifi gerekir. Küçük resim ızgarasındaki seçim kutuları en az bu boyutta olmalıdır (NFR-06). WCAG bir web standardıdır; mobil uygulamaya uygulanması varsayımdır (V-03). |
| **GDPR / diğer ülke mevzuatı** | Bu oturumda birincil kaynak açılmadı. | Hedef pazar belli değil (AS-04). |

### 3.3 Platform ve mağaza politikaları

| Platform | Bulgu | İş etkisi |
|---|---|---|
| **Google Play – Fotoğraf/Video İzinleri** | Android 13+ hedefleyen uygulamalar `READ_MEDIA_IMAGES`/`READ_MEDIA_VIDEO` izinlerini yalnızca sistem seçicileri yetmiyorsa isteyebiliyor. Geniş erişim örneği olarak "Apps whose core function includes managing and maintaining all of users' photos/videos (such as "gallery apps")" veriliyor. Play Console'da beyan gerekiyor. Ayrıca "you must make a reasonable effort to accommodate users who do not grant broad access to media files" şartı var [K8]. | Ürünün ana işlevi tüm kütüphaneyi taramak, dolayısıyla beyan zorunlu. Politika dosya temizleyicileri açıkça anmıyor; beyanın reddedilme riski var (AS-02). Tam erişim vermeyen kullanıcıya da çalışan bir deneyim sunulmalı (FR-02). |
| **Android 14+ seçili fotoğraf erişimi** | Kullanıcı tüm kütüphane yerine belirli fotoğraflara erişim verebiliyor. Uygulama bu durumda kütüphaneyi boş varsaymamalı ve kullanıcının seçimi genişletmesine izin vermeli. Seçili öğelere erişim "expires eventually" [K9]. | Kısmi erişimde sonuçlar eksik olur. Kullanıcıya bu açıkça söylenmeli ve seçimi genişletme yolu sunulmalı (FR-02). |
| **Android 11+ silme** | Başka uygulamanın medyası toplu olarak `createTrashRequest()` ile çöp kutusuna, `createDeleteRequest()` ile kalıcı olarak silinebiliyor. İki durumda da sistem kullanıcıdan onay diyaloğuyla izin istiyor. Çöp kutusundaki öğeler "system-defined" bir süre sonra kalıcı siliniyor. Varsayılan galeri olmayan uygulama her değişiklik/silmede onay göstermek zorunda. Android 12+'da "media management" özel izni bu onayları kaldırabiliyor [K10]. | Silme akışında bir sistem onay ekranı olacak. Çöp kutusuna gönderme seçeneği kullanıcıya geri alma güvencesi veriyor (FR-09). Medya yönetimi özel izni bir ürün kararıdır (AS-08). |
| **Google Play – Data safety** | "Even developers with apps that do not collect any user data must complete this form." "Collect" = "transmitting data from your app off a user's device" [K11]. | Form her durumda doldurulmalı. Fotoğraflar cihazdan çıkmazsa "toplanan veri" sayılmaz (NFR-01). |
| **iOS – Sınırlı erişim** | `PHAuthorizationStatus.limited`: "The user authorized this app for limited photo library access." Sistem, seçim ekranını otomatik gösterebiliyor; bu davranış Info.plist anahtarıyla kapatılabiliyor [K12]. | iOS'ta da kısmi erişim senaryosu ele alınmalı (FR-02). |
| **App Store İnceleme Yönergeleri** | **5.1.1(ii):** amaç metinleri (purpose strings) "clearly and completely describe your use of the data".<br>**5.1.1(iii):** mümkünse tam erişim yerine sistem seçicisi kullanılmalı.<br>**5.1.1(iv):** izin vermeyen kullanıcıya mümkünse alternatif sunulmalı.<br>**5.1.2(iv):** Photos verisiyle kendi kullanımı için veritabanı oluşturmak yasak.<br>**3.1.2(a):** abonelik süresi en az 7 gün [K13]. | İzin açıklaması ürünün neden tam kütüphaneye ihtiyaç duyduğunu açıkça anlatmalı (FR-01). Fotoğraf verisi başka amaçla kullanılmamalı (NFR-01). Abonelik kararı verilirse 3.1.2 kuralları geçerli olur (AS-03). |

### 3.4 Yöntemler (kopya tespit algoritmaları) – iş açısından değerlendirme

Seçim ve parametreler TA'ya bırakılmıştır. Aşağıdaki tablo yalnızca iş etkisini özetler.

| Yöntem | Nasıl çalışır (özet) | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Birebir (bayt düzeyi/kriptografik) hash** | Kriptografik hash'te "avalanche effect" geçerlidir: girdideki küçük bir değişiklik çıktıyı tamamen değiştirir [K15]. | Yalnızca bayt bayt aynı dosyaları eşleştirir. Kullanıcıya "bunlar kesinlikle aynı" denebilir ve güven yüksektir (kaynaktaki tanımdan çıkarıldı). | Yeniden boyutlandırılmış, yeniden sıkıştırılmış veya biçimi değişmiş kopyaları kaçırır (kaynaktaki tanımdan çıkarıldı). |
| **aHash (ortalama hash)** | Algısal hash ailesindendir [K15]. | Yazarın 150.000 görsellik testinde hızlı çalıştı: yaklaşık 3,75 saat [K14]. | Aynı testte 32 eşleşme beklenen bir görsel "over 400 images" ile eşleşti [K14]. Bu, beklenenin 12 katından fazla (kaynaktaki verilerden hesaplandı). Yanlış eşleşme, kullanıcının yanlış fotoğrafı silme riskini artırır. |
| **dHash (fark hash'i)** | Görsel 9×8'e küçültülür ve griye çevrilir. Komşu pikseller karşılaştırılarak 64 bitlik bir hash çıkar. Hash'ler farklı bit sayısıyla (Hamming mesafesi) karşılaştırılır. Yazarın yorumuna göre 0 "likely a similar picture", 1–10 olası varyasyon, 10'dan fazlası farklı görseldir [K14]. | Aynı testte "as fast as aHash" ve "Very few false positives" [K14]. | Aynı testte iki gerçek eşleşmesi olan bir görsel 6 görselle eşleşti, bunların 4'ü yanlıştı [K14]. Hatalı öneri riski sıfır değil. |
| **pHash (DCT tabanlı algısal hash)** | Algısal hash, benzer içeriğe benzer çıktı veren yerelliğe duyarlı bir hash'tir [K15]. | Yazarın testinde "No false positives, no false negatives" [K14]. Coğrafi görüntü veri setleri üzerine bir çalışmada algısal hash ile AICrowd eğitim setindeki görsellerin yaklaşık %90'ının (~250k) birebir kopyası bulundu [K16]. Yöntem büyük ölçekte kullanılabiliyor. | Yazarın testinde "over 7 hours" sürdü. Bu, aHash'in yaklaşık 1,9 katı (kaynaktaki verilerden hesaplandı) [K14]. Büyük kütüphanede tarama süresi uzar. Kasıtlı manipülasyonla algısal hash'lerde çakışma üretilebildiği gösterildi [K15][K18]. Kişisel kütüphane için düşük risk olsa da kullanıcı onayı şarttır. |
| **Gömme vektörü (image embedding)** | Görsel sayısal bir vektöre çevrilir ve kosinüs benzerliği ile karşılaştırılır. Örneğin MediaPipe Image Embedder ImageNet ile eğitilmiş MobileNet-V3 kullanır ve Android, iOS ve Flutter örnekleri sunar [K17]. | Seri çekim gibi "benzer ama aynı olmayan" fotoğrafları da yakalayabilir. Pixel 6'da ortalama gecikme CPU'da 3,94 ms (small model) [K17]. | Bu yöntem "benzer" fotoğrafları da yakalar. Ekran görüntülerinin hep "benzer" sayılması bir kullanıcının Remo hakkındaki şikâyetiydi [K5]. Model dosyası uygulama boyutunu artırır (büyüklüğü doğrulanmadı). |

**İş açısından sonuç (karar değil, girdi):**
- İki katmanlı bir yaklaşım iş hedeflerine uyuyor:
  - "kesin kopya" (yüksek güven, otomatik öneriye uygun),
  - "neredeyse aynı kopya" (kullanıcı kontrolüyle).
- "Benzer fotoğraflar" (seri çekim) farklı bir kullanıcı niyetidir ve yanlış silme riski yüksektir. Bu yüzden C önceliğinde ve ayrı kategoride tutuldu.
- Yukarıdaki hız ve doğruluk bulguları tek bir yazarın kendi görsel setindeki testine [K14] ve farklı alanlardaki çalışmalara [K16] dayanıyor. Mobil kişisel kütüphaneler için doğrulanmadı (V-05).

---

## 4. Fonksiyonel gereksinimler

**M** öncelikli gereksinimler ilk sürümün (MVP) sınırını tanımlar: M maddelerinin tamamı yoksa ürün ilk sürümde amacına ulaşamaz. S ve C maddeleri kaynak durumuna göre ilk sürüme alınır veya ertelenir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Uygulama, sistem izin penceresini göstermeden önce fotoğraflara neden ihtiyaç duyduğunu ve fotoğrafların cihazdan çıkmadığını anlatan bir açıklama ekranı gösterir. İzin isteği bu ekranda kullanıcının bir butona basmasıyla başlar, uygulama açılışında otomatik başlamaz. | M | [K9][K13] |
| FR-02 | Uygulama üç erişim durumunu ayrı ayrı ele alır. **Tam erişim:** tüm kütüphane taranır. **Sınırlı erişim:** yalnızca erişilebilen fotoğraflar taranır, kullanıcıya sonuçların eksik olabileceği söylenir ve seçimi genişletme seçeneği sunulur. **Red:** uygulamanın neden çalışamadığı açıklanır ve ayarlara gitme yolu gösterilir. | M | [K8][K9][K12][K13] |
| FR-03 | Kullanıcı taramayı istediği anda başlatabilir. Tarama sırasında ilerleme (işlenen/toplam fotoğraf ve yüzde) gösterilir. Kullanıcı taramayı iptal edebilir. | M | [K1] |
| FR-04 | Uygulama bayt bayt aynı fotoğrafları "Kesin kopya" olarak gruplar. | M | [K15] |
| FR-05 | Uygulama, görsel olarak aynı olup çözünürlüğü, sıkıştırması veya biçimi farklı olan fotoğrafları "Neredeyse aynı" olarak gruplar ve bu grupları "Kesin kopya" gruplarından ayrı gösterir. | M | [K14][K15] |
| FR-06 | Sonuç ekranı grup sayısını, kopya fotoğraf sayısını ve önerilen öğeler silinirse açılacak tahmini alanı (MB/GB) gösterir. | M | [K5] |
| FR-07 | Grup detayında fotoğraflar yan yana önizlenir. Her fotoğraf için çekim tarihi, çözünürlük, dosya boyutu ve dosya biçimi/türü gösterilir. Kullanıcı bir fotoğrafı tam ekran açabilir. | M | [K5] |
| FR-08 | Uygulama her grupta bir fotoğrafı "Saklanacak" olarak önerir. Kural: önce en yüksek çözünürlük, eşitse en büyük dosya, o da eşitse en eski tarih. Kullanıcı öneriyi değiştirebilir. Uygulama bir grubun tüm öğelerinin silinmek üzere seçilmesine izin vermez. | M | [K3][K5] |
| FR-09 | Silinmek üzere seçilen fotoğraflar platformun geri alınabilir alanına gönderilir: Android'de çöp kutusu, iOS'ta Son Silinenler (bkz. V-04). Silmeden önce uygulama silinecek adedi ve açılacak alanı gösteren bir onay ekranı gösterir. Uygulama kalıcı silme yapmaz. | M | [K3][K10] |
| FR-10 | Silme tamamlanınca bir özet gösterilir: silinen adet, tahmini açılan alan ve alanın ancak çöp kutusu/Son Silinenler boşaltıldığında tamamen açılacağı bilgisi. | M | [K3][K10] |
| FR-11 | Kullanıcı "Neredeyse aynı" tespitinin hassasiyetini üç kademeden birine ayarlayabilir: Sıkı / Normal (varsayılan) / Gevşek. | S | [K14] |
| FR-12 | Tarama kesilirse (uygulama arka plana alınır veya kapatılırsa) bir sonraki açılışta kaldığı yerden devam eder. İlk taramadan sonraki taramalar yalnızca yeni veya değişen fotoğrafları işler. | S | [K5] |
| FR-13 | Kullanıcı "Tüm gruplarda öneriyi uygula" ile tek seferde toplu seçim yapabilir. Silme FR-09'daki onay ekranından geçer. | S | [K1][K5] |
| FR-14 | Kullanıcı bir grubu "Kopya değil" olarak işaretleyebilir. İşaretlenen grup sonraki taramalarda gösterilmez. Bu işaret ayarlardan geri alınabilir. | S | [K14] |
| FR-15 | Uygulama, aynı anda çekilmiş benzer fotoğrafları (seri çekim) "Benzer fotoğraflar" adlı ayrı bir kategoride gösterir. Bu kategoride otomatik "silinecek" seçimi yapılmaz. | C | [K4][K5][K17] |
| FR-16 | Kullanıcı taramayı belirli bir albüm veya tarih aralığıyla sınırlayabilir. | C | — (görevden türetildi) |

---

## 5. Fonksiyonel olmayan gereksinimler

| ID | Kategori | Gereksinim | Ölçüt / doğrulama |
|---|---|---|---|
| NFR-01 | Gizlilik | Fotoğraflar, küçük resimler ve fotoğraflardan türetilen veriler (hash, vektör) cihaz dışına gönderilmez. Fotoğraf verisi tespit dışında bir amaçla kullanılmaz. | Tarama ve silme akışında fotoğraf kaynaklı ağ trafiği 0 bayt olur (ağ izleme ile test edilir). Play Data safety formunda fotoğraflar "toplanan veri" olarak beyan edilmek zorunda kalmaz [K11][K13]. |
| NFR-02 | Performans | 10.000 fotoğraflık bir kütüphanenin ilk taraması, referans orta segment cihazda uygulama ön plandayken tamamlanır. | ≤ 10 dakika (hedef değer, doğrulanmadı; bkz. V-05, AS-07). |
| NFR-03 | Performans | Artımlı taramada (FR-12) 100 yeni fotoğraf işlenir. | ≤ 30 saniye (hedef değer, doğrulanmadı; V-05). |
| NFR-04 | Yanıt verme | Tarama sürerken arayüz donmaz. Kullanıcı ekranlar arasında gezinebilir ve taramayı iptal edebilir. | İptal butonuna basıldıktan sonra tarama ≤ 2 saniyede durur. |
| NFR-05 | Doğruluk | Varsayılan hassasiyette "Kesin kopya" ve "Neredeyse aynı" gruplarında yanlış eşleşme oranı düşük tutulur. | Ekibin hazırlayacağı etiketli bir test setinde (en az 1.000 fotoğraf; yeniden boyutlandırılmış, sıkıştırılmış ve biçimi değişmiş kopyalar dahil): "Kesin kopya" kesinliği %100; "Neredeyse aynı" kesinliği ≥ %98 ve duyarlılığı ≥ %90 (hedef değerler; V-06). |
| NFR-06 | Erişilebilirlik | WCAG 2.2 AA'dan uyarlanan kurallar [K7]: metin kontrastı ≥ 4.5:1; dokunma hedefleri ≥ 24×24 (platform birimi); kaydırma/sürükleme gerektiren her işlev için buton alternatifi; fotoğraf önizlemelerine ekran okuyucu için metin alternatifi (ör. "Fotoğraf, 12 Mart 2025, 4032×3024, 3,1 MB"). | Erişilebilirlik kontrol listesi ve platform ekran okuyucusu ile manuel test. |
| NFR-07 | Veri güvenliği / geri alınabilirlik | Uygulamanın yaptığı hiçbir silme, platformun geri alınabilir alanını atlamaz. | Test edilen tüm silmelerde öğeler Android çöp kutusunda / iOS Son Silinenler'de bulunur (%100). |
| NFR-08 | Uyumluluk | Uygulama iOS ve Android telefonlarda çalışır. Minimum işletim sistemi sürümleri AS-06 ile belirlenir. | Her iki platformda kabul testleri geçer. |
| NFR-09 | Mağaza uyumu | Google Play fotoğraf/video izni beyanı ve Data safety formu doldurulur [K8][K11]. iOS izin amaç metni veri kullanımını açık ve eksiksiz anlatır [K13]. | Mağaza incelemesinden izin kaynaklı ret almadan geçilir. |
| NFR-10 | Depolama | Uygulamanın kendi verisi (tarama sonuçları, önbellek) kütüphane büyüklüğüyle orantılı ve sınırlı kalır. | 10.000 fotoğraf için ≤ 50 MB (hedef değer, doğrulanmadı; V-05). |
| NFR-11 | Dil | Arayüz Türkçedir. Metinler başka dillere çevrilebilecek şekilde ayrı tutulur. | Tüm arayüz metinleri Türkçe (V-08). |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: İzin vermeden önce bilgilenme** (FR-01)
P3 olarak, fotoğraflarıma erişim izni vermeden önce uygulamanın bunları ne için kullanacağını öğrenmek istiyorum. Böylece güvenle izin verebilirim.
- AC-1: İlk açılışta sistem izin penceresinden önce bir açıklama ekranı görünür. Bu ekranda "fotoğraflar cihazdan çıkmaz" ifadesi yer alır.
- AC-2: Sistem izin penceresi yalnızca kullanıcı açıklama ekranındaki butona bastığında açılır.

**US-02: Sınırlı erişimle kullanım** (FR-02)
P3 olarak, yalnızca seçtiğim fotoğraflara erişim verdiğimde de uygulamayı kullanabilmek istiyorum.
- AC-1: Sınırlı erişimde tarama yalnızca erişilebilen fotoğraflarla tamamlanır ve sonuç ekranında "Sonuçlar yalnızca erişim verdiğiniz fotoğrafları kapsar" uyarısı görünür.
- AC-2: Uyarının yanındaki buton, kullanıcının erişim seçimini genişletmesini sağlar.
- AC-3: İzin reddedildiğinde kilitlenme olmaz. Uygulama nedenini açıklar ve ayarlara gitme butonunu gösterir.

**US-03: İstediğim anda tarama** (FR-03, FR-12)
P1 olarak, taramayı istediğim anda başlatıp ilerlemesini görmek istiyorum. Böylece ne kadar bekleyeceğimi bilirim.
- AC-1: "Taramayı başlat" butonuna basıldığında ilerleme göstergesi "işlenen/toplam" ve yüzdeyi gösterir.
- AC-2: "İptal" butonuna basıldığında tarama ≤ 2 saniyede durur (NFR-04).
- AC-3 (S): Tarama sırasında uygulama kapatılıp yeniden açıldığında tarama baştan değil, kaldığı yerden sürer.

**US-04: Kopya gruplarını görme** (FR-04, FR-05, FR-06)
P1 olarak, kopyaların gruplar halinde listelendiğini ve ne kadar yer açabileceğimi görmek istiyorum.
- AC-1: Bayt bayt aynı fotoğraflar "Kesin kopya" başlığı altında listelenir.
- AC-2: Aynı fotoğrafın küçültülmüş veya farklı biçimde kaydedilmiş kopyası "Neredeyse aynı" başlığı altında listelenir.
- AC-3: Sonuç ekranında grup sayısı, kopya sayısı ve tahmini açılacak alan gösterilir.

**US-05: Karşılaştırıp saklanacak fotoğrafı seçme** (FR-07, FR-08)
P2 olarak, bir gruptaki fotoğrafları yan yana karşılaştırıp hangisinin saklanacağını görmek ve gerekirse değiştirmek istiyorum.
- AC-1: Grup detayında her fotoğrafın tarihi, çözünürlüğü, dosya boyutu ve biçimi görünür.
- AC-2: En yüksek çözünürlüklü fotoğraf "Saklanacak" olarak işaretlidir. Çözünürlükler eşitse en büyük dosya işaretlidir.
- AC-3: Kullanıcı "Saklanacak" işaretini başka bir fotoğrafa taşıyabilir.
- AC-4: Kullanıcı bir gruptaki tüm fotoğrafları silinecek olarak seçmeye çalıştığında uygulama buna izin vermez ve nedenini açıklar.

**US-06: Güvenle silme** (FR-09, FR-10)
P1 olarak, yanlışlıkla sildiğim bir fotoğrafı geri alabilmek istiyorum.
- AC-1: "Sil" butonuna basıldığında silinecek adet ve açılacak alanla birlikte bir onay ekranı çıkar.
- AC-2: Onaydan sonra fotoğraflar Android'de çöp kutusuna, iOS'ta Son Silinenler'e gider (NFR-07).
- AC-3: Özet ekranı, alanın çöp kutusu/Son Silinenler boşaltılınca tamamen açılacağını belirtir.

**US-07: Toplu temizlik** (FR-13)
P2 olarak, tüm gruplardaki önerileri tek dokunuşla uygulamak istiyorum.
- AC-1: "Tüm gruplarda öneriyi uygula" seçildiğinde her grupta yalnızca "Saklanacak" dışındaki öğeler seçilir.
- AC-2: Silme işlemi US-06/AC-1'deki onay ekranından geçmeden yapılmaz.

**US-08: Hassasiyeti ayarlama** (FR-11)
P2 olarak, "Neredeyse aynı" tespitinin ne kadar katı olacağını seçmek istiyorum.
- AC-1: Ayarlarda Sıkı/Normal/Gevşek seçenekleri vardır ve varsayılan Normal'dir.
- AC-2: Ayar değiştiğinde kullanıcıya yeni ayarın bir sonraki taramada uygulanacağı bildirilir.

**US-09: Yanlış eşleşmeyi gizleme** (FR-14)
P2 olarak, kopya olmayan bir grubu işaretleyip bir daha görmemek istiyorum.
- AC-1: "Kopya değil" ile işaretlenen grup sonraki taramalarda listelenmez.
- AC-2: Ayarlardaki "Gizlenen gruplar" listesinden işaret kaldırılabilir.

**US-10: Benzer fotoğrafları ayıklama** (FR-15)
P2 olarak, seri çekimde oluşan benzer fotoğrafları ayrı bir yerde görmek istiyorum.
- AC-1: Benzer fotoğraflar "Kesin kopya" ve "Neredeyse aynı" gruplarından ayrı bir kategoride listelenir.
- AC-2: Bu kategoride hiçbir fotoğraf otomatik olarak silinecek diye seçilmez.

**US-11: Taramayı daraltma** (FR-16)
P1 olarak, yalnızca belirli bir albümü veya tarih aralığını taramak istiyorum.
- AC-1: Kullanıcı tarama başlamadan önce bir albüm veya tarih aralığı seçebilir.
- AC-2: Sonuçlar yalnızca seçilen kapsamdaki fotoğrafları içerir.

---

## 7. Kapsam dışı maddeler

| Madde | Neden |
|---|---|
| Video kopyalarının tespiti | Görev "Photo Library" diyor ve videolar açıkça istenmemiş. İlk sürüm fotoğraflarla sınırlı tutuldu (AS-01). |
| Yüz tanıma veya kişiye göre gruplama | Görevde yok. Biyometrik veri KVKK md. 6'ya göre özel nitelikli kişisel veridir [K6]. |
| Bulanık/karanlık/kötü fotoğraf tespiti, ekran görüntüsü temizliği | Rakiplerde var [K4] ama kopya bulma görevinin dışında. |
| Bulut hesaplarına (iCloud, Google Photos vb.) doğrudan bağlanıp tarama | Görev cihazdaki fotoğraf kütüphanesiyle ilgili (V-02). |
| Sunucu tarafı işleme, hesap/oturum açma | Ürün büyüklüğüne uygun değil ve NFR-01 ile çelişir. |
| Kalıcı silme (çöp kutusunu atlama) ve çöp kutusunu uygulama içinden boşaltma | Geri alınamaz kayıp riski taşır (NFR-07). |
| Tablet, web ve masaüstü sürümleri | Görev cep telefonlarını hedefliyor. |
| Kopyaları birleştirme (meta verileri tek fotoğrafta toplama) | iOS'taki "Merge" davranışının ayrıntısı doğrulanamadı [K2]. İlk sürümde yalnızca saklama/silme var. |

---

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| V-01 | Flutter ve hedef platformlar (iOS, Android) görevde verilmiş kısıtlardır. Bu doküman teknoloji seçimi yapmaz; algoritma ve kütüphane seçimi TA'ya aittir. |
| V-02 | "Photo Library", cihazın sistem fotoğraf kütüphanesi anlamına gelir (iOS Photos, Android MediaStore). Yalnızca bulutta duran öğelerin nasıl ele alınacağı belirsizdir (AS-09). |
| V-03 | WCAG 2.2 bir web standardıdır [K7]. Mobil uygulama için erişilebilirlik ölçütü olarak uyarlandı. Platformların kendi erişilebilirlik kılavuzları bu oturumda açılmadı. |
| V-04 | iOS'ta uygulama tarafından silinen fotoğrafların Son Silinenler'e gittiği ve sistemin silmeden önce kullanıcıdan onay istediği doğrulanamadı: Apple'ın `deleteAssets` belgesi bu konuda bir şey söylemiyor. Bu davranış TA tarafından doğrulanmalı. FR-09'daki uygulama içi onay her durumda geçerlidir. |
| V-05 | NFR-02, NFR-03 ve NFR-10'daki süre ve boyut hedefleri kaynaklı değildir, başlangıç hedefleridir. [K14]'teki süreler masaüstü ölçekte, tek bir yazarın kendi 150.000 görseliyle yaptığı teste aittir. Mobil cihazlara doğrudan aktarılamaz. |
| V-06 | NFR-05'teki doğruluk hedefleri kaynaklı değildir. Ekibin oluşturacağı etiketli test setiyle doğrulanmalıdır. |
| V-07 | Persona'lar pazar verisine değil, görev metnine ve rakip mağaza yorumlarına [K4][K5] dayanır. |
| V-08 | Hedef kullanıcı Türkiye'deki genel kullanıcıdır ve arayüz dili Türkçedir (AS-04). |
| V-09 | Ürün çocuklara yönelik değildir. Bu nedenle çocuk verisi düzenlemeleri ayrıca incelenmedi. |
| V-10 | iOS Photos'un kopya kriterleri ve "Merge" sırasında hangi kopyanın saklandığı Apple'ın birincil kaynağından doğrulanamadı [K2]. Bu konudaki ikincil kaynak iddiaları kullanılmadı. |
| V-11 | Bazı sayfalar açılamadı veya iddiayı desteklemedi; bu sayfalar kaynak gösterilmedi:<br>- MDPI'deki algısal hash karşılaştırma makalesi (HTTP 403)<br>- Apple `NSPhotoLibraryUsageDescription` belgesi (içerik gelmedi)<br>- kvkk.gov.tr özel nitelikli veri sayfası (sertifika hatası)<br>- bir Google Play rakip listesi (içerik gelmedi)<br>- CNET/AOL haberi (404) |

---

## 9. Açık sorular

| ID | Soru | İlgili |
|---|---|---|
| AS-01 | Videolar ilk sürümde veya sonraki bir sürümde kapsama alınacak mı? | Bölüm 7 |
| AS-02 | Google Play beyanında ürün "managing and maintaining all of users' photos/videos" kapsamına girecek mi [K8]? Beyan reddedilirse Android'de hangi alternatif deneyim kabul edilir (yalnızca sınırlı erişimle çalışma vb.)? | FR-02, NFR-09 |
| AS-03 | Gelir modeli nedir (ücretsiz, tek seferlik ücret, abonelik, reklam)? Abonelik seçilirse App Store 3.1.2 kuralları geçerli olur [K13]. Rakip yorumlarında "silmek için ödeme" uygulamasından şikâyet edildiği görülüyor [K5]. | — |
| AS-04 | Hedef pazar yalnızca Türkiye mi? AB de hedefleniyorsa GDPR değerlendirmesi gerekir. Arayüz hangi dillerde olacak? | NFR-11, V-08 |
| AS-05 | Tüm işleme cihazda kaldığında geliştirici KVKK açısından "veri sorumlusu" sayılır mı [K6]? Aydınlatma metni gerekli mi? (Hukuki görüş gerekir.) | NFR-01 |
| AS-06 | Minimum desteklenecek iOS ve Android sürümleri neler? Android 11 öncesinde toplu çöp kutusu isteği yoktur [K10]. | NFR-08, FR-09 |
| AS-07 | NFR-02 ve NFR-03 için hangi referans cihaz(lar) kullanılacak? | NFR-02, NFR-03 |
| AS-08 | Android'de her silmede çıkan sistem onayı yerine kullanıcıdan "media management" özel izni istenmeli mi [K10]? | FR-09 |
| AS-09 | Yalnızca bulutta duran fotoğraflar taramaya dahil edilecek mi? Dahil edilecekse mobil veri kullanımı konusunda kullanıcıya sorulacak mı? | V-02 |
| AS-10 | Düzenlenmiş fotoğraflar (kırpılmış, filtre uygulanmış) "Neredeyse aynı" sayılmalı mı? | FR-05 |
| AS-11 | Analitik veya çökme raporlama kullanılacak mı? Kullanılacaksa Data safety ve App Store gizlilik beyanları buna göre doldurulmalı [K11]. | NFR-01 |
| AS-12 | FR-08'deki "saklanacak" öneri kuralı (çözünürlük → boyut → tarih) paydaşlarca uygun mu? Favori veya albümdeki fotoğraf öncelikli tutulmalı mı? | FR-08 |

---

## 10. Kaynaklar

Tüm kaynaklar 2026-11-22 tarihinde WebFetch ile açılıp okundu.

| ID | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | Apple Support – If you can't find the Duplicates album for duplicate photos and videos on iPhone | https://support.apple.com/102260 | iOS 16+ kopya tespiti ve Duplicates albümü. "The detection process requires iPhone to be locked and connected to power." Süre: "take up to a few days to complete." |
| K2 | Apple iPhone User Guide – Merge duplicate photos and videos | https://support.apple.com/en-gb/guide/iphone/iph1978d9c23/ios | Duplicates koleksiyonu ve Merge adımları. Kopya kriteri ve hangi kopyanın saklandığı sayfada yazmıyor. |
| K3 | Files by Google Help – Delete duplicate files | https://support.google.com/files/answer/9764075?hl=en | "The original file is marked with an "Original" badge." "We recommend that you keep the original file." Silinenler onay diyaloğuyla Trash'e taşınıyor. |
| K4 | App Store – CleanMy®Phone: Cleanup Storage | https://apps.apple.com/US/app/id1277110040 | "finds duplicate and similar shots". "works fully on-device". Yıllık 36,99 $ / aylık 7,99 $ (ABD). 23K oyla 4,6 puan. Geliştirici beyanlı izleme verisi. Kullanıcı yorumları. |
| K5 | App Store – Remo Duplicate Photos Remover | https://apps.apple.com/app/id1066797785 | "Automatically keeps one best copy safe in each group". Tarih/çözünürlük/boyut gösterimi. "The developer does not collect any data". 14K oyla 4,6 puan. Kullanıcı yorumları: arka planda tarama durması, dosya türünün olmaması, ekran görüntülerinin "similars" sayılması. |
| K6 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (mevzuat.gov.tr) | https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf | Md. 3/d: "Kişisel veri: Kimliği belirli veya belirlenebilir gerçek kişiye ilişkin her türlü bilgiyi". Md. 3/e: işleme tanımı. Md. 6/1: "…biyometrik ve genetik verileri özel nitelikli kişisel veridir." Md. 6/3: işlenmeleri yasak, istisnalar var. |
| K7 | W3C – Web Content Accessibility Guidelines (WCAG) 2.2 | https://www.w3.org/TR/WCAG22/ | 1.4.3: "contrast ratio of at least 4.5:1". 2.5.8: "at least 24 by 24 CSS pixels". 2.5.7: "achieved by a single pointer without dragging". 1.1.1: metin alternatifi. |
| K8 | Google Play Console Help – Details on Google Play's Photo and Video Permissions policy | https://support.google.com/googleplay/android-developer/answer/14115180 | İzinler yalnızca sistem seçicileri yetmezse alınabiliyor. Örnek: "managing and maintaining all of users' photos/videos (such as "gallery apps")". "you must make a reasonable effort to accommodate users who do not grant broad access". Beyan zorunluluğu. |
| K9 | Android Developers – Grant partial access to photos and videos (Android 14) | https://developer.android.com/about/versions/14/changes/partial-photo-video-access | Android 14'te seçili fotoğraf erişimi. Kütüphane boş varsayılmamalı, kullanıcı seçimi genişletebilmeli. Seçili öğelere erişim "expires eventually". İzin, uygulama açılışında değil gerektiğinde istenmeli. |
| K10 | Android Developers – Access media files from shared storage | https://developer.android.com/training/data-storage/shared/media | Android 11+ `createTrashRequest`/`createDeleteRequest` ve kullanıcı onay diyaloğu. Çöp kutusundaki öğeler "after a system-defined time period" kalıcı siliniyor. "media management" özel izni (Android 12+). |
| K11 | Google Play Console Help – Provide information for Google Play's Data safety section | https://support.google.com/googleplay/android-developer/answer/10787469?hl=en | "Even developers with apps that do not collect any user data must complete this form." "Collect" = "transmitting data from your app off a user's device". |
| K12 | Apple Developer – PHAuthorizationStatus.limited | https://developer.apple.com/documentation/photos/phauthorizationstatus/limited.md | "The user authorized this app for limited photo library access." (iOS 14+). Sınırlı erişim seçicisinin otomatik gösterimi ve Info.plist ile kapatılabilmesi. |
| K13 | Apple – App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 5.1.1(ii) amaç metinleri. 5.1.1(iii) "Where possible, use the out-of-process picker… rather than requesting full access to protected resources like Photos". 5.1.1(iv) izin vermeyene alternatif. 5.1.2(iv) Photos verisiyle veritabanı yasağı. 3.1.2(a) abonelik ≥ 7 gün. |
| K14 | The Hacker Factor Blog – Kind of Like That (Neal Krawetz) | https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html | dHash adımları (9×8, gri ton, komşu piksel farkı, 64 bit) ve Hamming eşiklerine ilişkin yazar yorumu. Yazarın 150.000 görsellik testinde: aHash "about 3.75 hours", 32 yerine "over 400 images" eşleşme; pHash "No false positives, no false negatives", "over 7 hours"; dHash "as fast as aHash", "Very few false positives". |
| K15 | Wikipedia – Perceptual hashing | https://en.wikipedia.org/wiki/Perceptual_hashing | Algısal hash bir "locality-sensitive hash". Kriptografik hash ise "avalanche effect"e dayanıyor. Struppek ve diğerlerine göre küçük değişikliklerle hash çakışması üretilebiliyor. |
| K16 | arXiv 2304.02296 – Data Leakage Detection and De-duplication in Large Scale Geospatial Image Datasets | https://arxiv.org/abs/2304.02296 | Algısal hash ile AICrowd Mapping Challenge veri setinde "nearly 90% (about 250k) of the training split images had identical copies". |
| K17 | Google AI Edge – MediaPipe Image Embedder | https://developers.google.com/edge/mediapipe/solutions/vision/image_embedder | Görseli vektöre çeviriyor ve kosinüs benzerliği hesaplıyor. MobileNet-V3 (ImageNet ile eğitilmiş). Android/iOS/Flutter örnekleri. Pixel 6'da ortalama gecikme CPU'da 3,94 ms (small) ve 9,75 ms (large). |
| K18 | arXiv 2106.09820 – Adversarial Detection Avoidance Attacks: Evaluating the robustness of perceptual hashing-based client-side scanning | https://arxiv.org/abs/2106.09820 | Makalenin özetine göre DCT tabanlı algısal hash algoritmalarına yapılan saldırılarda "more than 99.9% of images successfully attacked while preserving the content of the image" (kasıtlı manipülasyon senaryosu). |

**Kaynak gösterilmeyen sayfalar:** Açılamayan veya iddiayı desteklemeyen sayfalar için bkz. V-11.