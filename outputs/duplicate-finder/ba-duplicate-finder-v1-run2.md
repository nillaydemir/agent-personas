# İş Analizi: Fotoğraf Kütüphanesi Kopya Bulucu (Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** "Flutter ile yazacağımız ve cep telefonlarında çalışacak bir Photo Library duplicate finder uygulaması hazırlayacağız. Gerekli algoritmaları araştırıp bana bir iş analizi çıkar."

---

## 1. Özet

Uygulama, telefondaki fotoğraf kütüphanesini tarar. Birebir aynı kopyaları (exact duplicate) ve görsel olarak çok benzer fotoğrafları (near-duplicate) gruplar halinde gösterir. Kullanıcı her grupta hangi fotoğrafı tutacağını onaylar, gerisini güvenli biçimde siler ve depolama alanı açar.

Pazarda üç tür çözüm var:
- **Platformun kendi araçları:** iOS Fotoğraflar uygulamasının "Duplicates" albümü ve Files by Google'ın "Duplicate files" kartı.
- **Ücretli üçüncü taraf uygulamalar:** Gemini Photos ve Tidy gibi.

Platform araçları ücretsizdir ama kapsamı dardır. Kaynaklara göre iOS aracı yalnızca birebir kopyaları yakalar ve seri çekimleri (burst) kopya saymaz. Üçüncü taraf uygulamalar benzer fotoğrafları da bulur ve "en iyi kare" önerir. Ancak abonelik isterler ve inceleyicilere göre farklı kişileri de "benzer" diye işaretleyebilirler.

Gerekli algoritmalar iki gruptadır:
1. **Birebir kopya tespiti:** İçeriğin kriptografik özeti (hash) karşılaştırılır.
2. **Benzer fotoğraf tespiti:**
   - Algısal özet (perceptual hash: aHash, dHash, pHash) ve Hamming uzaklığı, ya da
   - Makine öğrenmesi modeliyle çıkarılan görüntü vektörü (embedding) ve kosinüs benzerliği.

Çok sayıda fotoğrafı karşılaştırırken her fotoğrafı her fotoğrafla kıyaslamak yavaş kalır. BK-tree gibi uzaklık tabanlı indeksler bu sorunu azaltır. Yöntemlerin iş açısından artıları ve eksileri 3.2'de verilmiştir. Hangisinin seçileceğine Technical Architect (TA) karar verecektir.

Ürün küçük/orta ölçekli, tek kullanıcılı bir tüketici uygulamasıdır. Kapsam buna göre sınırlandı: yalnızca fotoğraflar, tüm işlemler cihazda, hesap ve bulut yok.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Endişe |
|---|---|---|---|
| **P1: Alanı dolan kullanıcı** | Telefonu "depolama dolu" uyarısı veren, teknik bilgisi orta düzeyde biri. | Hızlıca ve az çabayla alan açmak. | Yanlışlıkla önemli bir fotoğrafı silmek. |
| **P2: Çok çeken kullanıcı** | Ebeveyn, gezgin veya hobi fotoğrafçısı. Aynı sahneyi saniyeler arayla çok kez çeker, kütüphanesinde on binlerce fotoğraf vardır. | Benzer kareler arasından en iyisini tutup gerisini topluca temizlemek. | Uzun tarama süresi, gruplar arasında tek tek gezinmenin yorucu olması. |
| **P3: Gizlilik hassasiyeti olan kullanıcı** | Fotoğraflarının buluta ya da üçüncü tarafa gitmesini istemez. | Analizin cihazda yapıldığından emin olmak. | Tam kütüphane erişimi vermek. Kısmi erişimle kullanmak isteyebilir. |

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakipler

| Çözüm | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler |
|---|---|---|---|
| **iOS Fotoğraflar – Duplicates albümü** (iOS 16 ve sonrası) | Utilities bölümünde, yalnızca kopya varsa görünen bir albüm. Gruplar tarih sırasıyla listelenir, her grubun yanında "Merge" düğmesi vardır. "Select All" ile toplu birleştirme yapılabilir. Birleştirmede en yüksek kaliteli kopya ve ilgili veriler tutulur, diğer kopyalar "Recently Deleted" albümüne gider ve 30 gün orada kalır. [MacRumors], [Setapp] | Ücretsiz, sisteme gömülü. Yapılan işlem geri alınabilir. Kopyalar silinmez, birleştirilir ve meta veriler korunur. | Setapp'e göre yalnızca birebir kopyaları yakalar ve seri çekimleri kopya saymaz. MacRumors ise çok benzer karelerin de birleştirilebildiğini yazıyor. Kaynaklar burada çelişiyor (bkz. Açık sorular S-08). Tarama birkaç gün sürebilir. Benzerlik hassasiyeti ayarlanamaz. |
| **Files by Google – Duplicate files** (Android) | "Clean" bölümünde "Duplicate files" kartı bulunur. Orijinal kopya "Original" rozetiyle işaretlenir ve tutulması önerilir. Seçilen kopyalar onay alındıktan sonra çöp kutusuna (Trash) taşınır. [Google Files Yardım] | Ücretsiz. Hangi kopyanın orijinal olduğunu açıkça gösterir. Kalıcı silme yerine çöp kutusu kullanır. | Kopyaların nasıl tespit edildiği belgelenmemiş. Benzer fotoğraf desteğine dair bilgi bulunamadı. |
| **Gemini Photos** (MacPaw, iOS) | Bulguları üç kategoriye ayırır: Similar, Clutter (bulanık kareler, not fotoğrafları, ekran görüntüleri) ve Other. Benzer gruplarda bir "Best Result" önerir, gerisini silinmek üzere önceden seçer. Önerirken düzenlemeleri, favori işaretini ve yüz tespiti verisini dikkate alır. Favoriler hiçbir zaman otomatik seçilmez. Hiçbir şeyi kendiliğinden silmez, silinenler 30 gün Recently Deleted'da kalır. İşlemler cihazda, çevrimdışı yapılır. Videoları desteklemez. Kullanıcı bir grubu gizleyerek tekrar işaretlenmesini engelleyebilir. [MacPaw SSS], [iMore] | En iyi kare önerisi var, favoriler korunur, gizlilik ön planda. | Ücretli: 3 günlük denemeden sonra aylık 1,99 $, yıllık 11,99 $ ya da tek seferlik 14,99 $ (2018 inceleme verisi). İnceleyici, farklı kişilerin fotoğraflarını da kopya diye işaretleyebildiğini ve benzer karelerin her zaman gereksiz olmadığını belirtiyor. [iMore] |
| **Tidy: Photo & Gallery Cleaner** (iOS) | Yapay zekâ ile kopya ve neredeyse aynı fotoğrafları gruplar, videolarda da çalışır. Bulanık ve düşük kaliteli fotoğraf filtresi var. "Best-Shot Auto-Pick" en iyi kareyi yıldızlar. Yan yana yakınlaştırarak karşılaştırma modu ve tek albüm temizleme sunar. [App Store – Tidy] | Zengin özellik seti, karşılaştırma modu, 429 değerlendirmede 4,8 puan. | Abonelik veya ömür boyu satın alma gerekir (ömür boyu 49,99 $). Gelişmiş kopya tespiti ücretli. Gizlilik etiketinde takip amaçlı tanımlayıcı kullanımı beyan edilmiş. Düzenleyici, seriler (streaks) gibi kapsamı genişleten özellikler içeriyor. [App Store – Tidy] |

**Rakiplerden çıkan dersler:**
1. Hiçbir rakip kullanıcı onayı almadan silmiyor ve silinenler geri alınabilir bir alana gidiyor. Bu sektörde temel beklenti haline gelmiş.
2. "Benzer" ile "aynı" farklı şeyler. Kullanıcının grubu gözden geçirmesi ve bir grubu "kopya değil" diye işaretleyebilmesi gerekiyor.
3. Gizlilik ve cihazda işleme bir rekabet avantajı olarak öne çıkarılıyor.
4. Platform araçları ücretsiz ve birebir kopyalarda yeterli. Ürünün farkı benzer fotoğraf tespiti, ayarlanabilir hassasiyet ve şeffaf "tutulacak" önerisi olabilir.

### 3.2 İlgili yöntemler (algoritmalar)

Aşağıda yöntemler yalnızca iş etkisi açısından karşılaştırılmıştır. Hangi yöntemin seçileceğine ve nasıl uygulanacağına TA karar verecektir.

#### A) Birebir kopya tespiti

| Yöntem | Nasıl çalışır | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Kriptografik özet** (ör. SHA-256) | Dosya içeriğinden sabit uzunlukta bir özet hesaplanır. Özetler eşitse dosyalar aynı kabul edilir. Kriptografik özetler kopya veriyi tespit etmek ve dosyaları tanımlamak için kullanılır. [Wikipedia – Kriptografik özet] | Uygulamada yanlış pozitif yok denecek kadar az, kullanıcı güveni yüksek. | Çığ etkisi (avalanche effect) nedeniyle tek bitlik bir fark bile özeti tamamen değiştirir. Yeniden kaydedilmiş ya da boyutu değiştirilmiş bir kopya yakalanmaz. [Wikipedia – Algısal özet] Her dosyanın tamamının okunması gerekir. |

#### B) Benzer fotoğraf tespiti: algısal özetler

Algısal özet, benzer içeriğe benzer parmak izi üretir. İki özetin ne kadar farklı olduğu, farklı bit sayısıyla, yani Hamming uzaklığıyla ölçülür. [mojoauth], [Wikipedia – Algısal özet]

| Yöntem | Nasıl çalışır | Hız | Doğruluk ve dayanıklılık |
|---|---|---|---|
| **aHash** (ortalama özet) | 8×8 küçük resimde her piksel ortalama parlaklıkla kıyaslanır. | En basit ve hızlı yöntem. [mojoauth], [HackerFactor] | Çok sayıda yanlış pozitif üretir. 32 eşleşme beklenen bir testte 400'den fazla eşleşme bulmuş. [HackerFactor] |
| **dHash** (fark özeti) | 9×8 gri tonlamalı resimde her piksel sağdaki komşusuyla kıyaslanır, 64 bit özet çıkar. [HackerFactor] | aHash kadar hızlı. Testte yaklaşık 3,75 saat sürmüş, pHash ise 7 saatten fazla. [HackerFactor] | pHash'e yakın doğrulukta, az yanlış pozitif üretir. [HackerFactor] |
| **pHash** (DCT tabanlı) | 32×32 gri tonlamalı resme ayrık kosinüs dönüşümü (DCT) uygulanır ve düşük frekanslı katsayılar kullanılır. [mojoauth] | En yavaşı, çünkü DCT hesabı darboğaz oluşturuyor. [HackerFactor] | Testte en doğru sonucu vermiş. [HackerFactor] Ton ve filtre değişikliklerine en dayanıklı yöntem. [mojoauth] |

Üç yöntemin ortak sınırları:
- **Kırpma, döndürme ve aynalama:** Üç yöntem de bunlara karşı zayıftır. Örnekte her kenardan %10 kırpılınca dHash uzaklığı 20, pHash uzaklığı 24 çıkmış. [mojoauth]
- **Yanlış eşleşme riski:** Birbiriyle ilgisiz görüntüler de benzer özet üretebilir. [Wikipedia – Algısal özet] Bu yüzden kullanıcı onayı şarttır.

Eşik değeri konusunda kaynaklar farklı şeyler söylüyor:

| Kaynak | Önerilen eşik |
|---|---|
| [HackerFactor] | 0: büyük olasılıkla aynı resim, 1–10: varyasyon olabilir, 10'un üstü: farklı resim. |
| [mojoauth] | Başlangıç noktası olarak yaklaşık 10, sonra kendi verinize göre ayarlanmalı. |
| [Ben Hoyt] | 128 bit dHash'te eşik 2 bit seçilmiş. 4–5 bit eşiklerde farklı resimler de eşleşmiş (yanlış pozitif). |

**İş sonucu:** Eşik doğrudan doğruluğu belirliyor ve tek bir doğru değeri yok. Bu nedenle kullanıcıya hassasiyet ayarı sunulmalı (FR-05) ve varsayılan değer etiketli bir test setiyle belirlenmeli (NFR-03).

#### C) Benzer fotoğraf tespiti: makine öğrenmesi vektörleri (embedding)

| Yöntem | Nasıl çalışır | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Görüntü vektörü + kosinüs benzerliği** (ör. MediaPipe Image Embedder, MobileNet V3) | Bir model her görüntü için sayısal bir vektör üretir. İki vektör kosinüs benzerliğiyle karşılaştırılır, sonuç -1 ile 1 arasındadır. Android, iOS, web ve Python platformlarında sunuluyor, Flutter da listede. Pixel 6'da görüntü başına ortalama gecikme CPU'da Small model için 3,94 ms, Large model için 9,75 ms. [MediaPipe] | Açı farkı, kadraj gibi "aynı sahne" durumlarını yakalama potansiyeli daha yüksek. Bir akademik karşılaştırma, klasik yöntemlerin her zaman en iyi seçim olmadığını, dHash ile SimCLR v2 tabanlı yaklaşımların ikisinin de çok iyi sonuç verdiğini bildiriyor. [arXiv 2206.00282] | Uygulama boyutu, model dosyası yüzünden büyüyebilir. "Aynı sahne"yi fazla kolay eşleştirirse farklı anları da gruplayabilir. iMore'un Gemini için bildirdiği "farklı kişiler eşleşti" sorunu buna örnek. [iMore] |

#### D) Gruplama ve ölçek

- **Hepsini hepsiyle karşılaştırma:** Karşılaştırma sayısı fotoğraf sayısının karesiyle artar (O(N²)). 100–150 bin fotoğrafta bu yaklaşım çok yavaş kalmış. [Ben Hoyt]
- **BK-tree:** Hamming uzaklığı gibi bir metrik üzerinde, belirli bir uzaklık içindeki öğeleri tüm listeyi taramadan bulur. Ortalama durumda logaritmik, kötü durumda doğrusal maliyetlidir. [Wikipedia – BK-tree] Hoyt'un gözlemine göre gerçek maliyet seçilen eşiğe bağlı olarak log N ile N arasında bir yerde. [Ben Hoyt]
- **Çekim zamanı ön filtresi:** Gemini'ye göre benzer fotoğraflar çoğunlukla saniyeler arayla çekiliyor. [MacPaw SSS] Çekim zamanına göre ön gruplama karşılaştırma sayısını azaltabilir. Ancak zaman damgası farklı olan kopyalar (ör. mesajlaşma uygulamasından tekrar kaydedilmiş bir fotoğraf) kaçabilir. Bu takas TA'ya bırakılmıştır.

#### E) Platform erişim kısıtları (iş etkisi)

- **iOS:** Kullanıcı "Select Photos" seçeneğiyle yalnızca bazı fotoğraflara erişim verebilir. Bu durumda uygulama yalnızca o fotoğrafları görür, paylaşılan bulut albümlerine de erişemez. Apple, kullanıcının seçimini uygulama içinden değiştirebileceği bir arayüz sunulmasını öneriyor. [WWDC20 10641]
- **Android 14 ve sonrası:** Kullanıcı yalnızca seçtiği fotoğraf ve videolara erişim verebilir. Google, uygulamanın tam, kısmi veya reddedilmiş erişim durumunu kontrol edip arayüzü buna göre güncellemesini, kullanıcıya seçimini genişletme imkânı vermesini ve izni gerektiği anda, uygulama açılışında değil, istemesini öneriyor. [Android 14 kısmi erişim]
- **İş sonucu:** Kısmi erişimde uygulama kütüphanenin yalnızca bir bölümünü tarayabilir. Kullanıcıya bu durum açıkça anlatılmalı (FR-01).

---

## 4. Fonksiyonel gereksinimler

Öncelikler: **M** = Must (MVP için zorunlu), **S** = Should (güçlü öneri), **C** = Could (kaynak kalırsa).

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Uygulama fotoğraf kütüphanesi iznini ilk taramadan hemen önce ister, açılışta istemez. İzin durumunu "tam", "kısmi" ve "reddedildi" olarak ayırt eder. Kısmi erişimde "Yalnızca seçtiğiniz X fotoğraf taranabilir" bilgisini ve seçimi genişletme düğmesini gösterir. Erişim reddedildiyse cihaz ayarlarına yönlendiren bir açıklama gösterir. | M | 3.2-E |
| FR-02 | Kullanıcı taramayı tek dokunuşla başlatır. Tarama sırasında taranan ve toplam fotoğraf sayısı ile yüzde ilerleme gösterilir. Kullanıcı taramayı istediği zaman iptal edebilir. | M | Görev, P1, P2 |
| FR-03 | Uygulama içeriği birebir aynı olan fotoğrafları "Birebir kopya" grubu olarak tespit eder. | M | 3.2-A |
| FR-04 | Uygulama görsel olarak çok benzer fotoğrafları "Benzer" grubu olarak tespit eder: yeniden sıkıştırılmış, yeniden boyutlandırılmış, parlaklığı biraz değiştirilmiş kopyalar ve saniyeler arayla çekilmiş kareler. Birebir kopya grupları ile benzer gruplar arayüzde ayrı etiketlenir. | M | 3.1, 3.2-B/C |
| FR-05 | Kullanıcı benzerlik hassasiyetini en az 3 seviyeden (Sıkı / Dengeli / Gevşek) seçebilir. Varsayılan seviye "Dengeli"dir. Seviye değişince sonuçlar yeniden hesaplanır. | S | 3.2-B, eşik belirsizliği |
| FR-06 | Sonuç ekranı grupları listeler. Her grup için tür (birebir/benzer), fotoğraf sayısı ve temizlenirse kazanılacak tahmini alan gösterilir. Ekranın üstünde toplam grup sayısı ve toplam kazanılabilir alan yer alır. | M | P1 |
| FR-07 | Her grupta bir fotoğraf "Tutulacak" olarak önerilir, diğerleri silinmek üzere önceden seçilir. Varsayılan sıralama: (1) favori işaretli fotoğraf, (2) en yüksek çözünürlük, (3) en büyük dosya boyutu, (4) en eski çekim tarihi. Favori fotoğraflar hiçbir zaman otomatik olarak silinmek üzere seçilmez. | M | [MacPaw SSS], [Google Files Yardım], [MacRumors] |
| FR-08 | Kullanıcı bir grubu açtığında fotoğrafları yan yana veya sırayla tam ekran ve yakınlaştırılabilir biçimde görür. Her fotoğrafın çekim tarihi, çözünürlüğü ve dosya boyutu da gösterilir. | M | [App Store – Tidy] |
| FR-09 | Kullanıcı grup içindeki seçimleri değiştirebilir (tutulacak/silinecek). Bir grubu "Kopya değil" olarak işaretleyebilir. İşaretlenen grup sonraki taramalarda gösterilmez ve kullanıcı bu işareti ayarlardan geri alabilir. | M | [iMore], [MacPaw SSS] |
| FR-10 | Silme işlemi her zaman bir onay diyaloğundan geçer. Diyalogda silinecek fotoğraf sayısı ve kazanılacak alan yer alır. Silinen fotoğraflar platformun geri alınabilir çöp alanına gönderilir (iOS'ta "Son Silinenler", Android'de çöp kutusu). Uygulama kalıcı silme yapmaz. | M | 3.1 dersleri |
| FR-11 | Kullanıcı birden fazla grubu, ya da sonuç ekranındaki tüm grupları, tek işlemle temizleyebilir. Bu işlem de FR-10'daki onaydan geçer. | S | [MacRumors], P2 |
| FR-12 | Temizlik bittikten sonra silinen fotoğraf sayısı ve kazanılan alanı gösteren bir özet ekranı açılır. Ekranda fotoğrafların çöp alanından nasıl geri alınacağı bir cümleyle anlatılır. | S | P1 |
| FR-13 | Sonraki taramalarda yalnızca son taramadan sonra eklenen, değişen ya da silinen fotoğraflar işlenir (artımlı tarama). Kullanıcı isterse tam yeniden tarama başlatabilir. | S | P2, NFR-02 |
| FR-14 | Kullanıcı taramayı tüm kütüphane yerine tek bir albümle sınırlayabilir. | C | [App Store – Tidy] |

---

## 5. Fonksiyonel olmayan gereksinimler

Hedef sayılar varsayımdır (bkz. Varsayımlar V-04, V-05). Paydaş onayından sonra kesinleşir.

| ID | Gereksinim | Ölçüt / doğrulama |
|---|---|---|
| NFR-01 | **Gizlilik:** Fotoğrafların içeriği ve bunlardan türetilen veriler (özetler, vektörler) cihaz dışına gönderilmez. Tüm analiz cihazda yapılır. | Tarama ve temizlik sırasında ağ trafiği izlenir. Fotoğraf içeriği ya da türetilmiş veri taşıyan istek sayısı 0 olmalıdır. Mağaza gizlilik etiketi bununla tutarlı olmalıdır. |
| NFR-02 | **Tarama süresi:** 10.000 fotoğraflık bir kütüphane, referans orta segment cihazda ilk taramada en fazla 10 dakikada, artımlı taramada (100 yeni fotoğraf) en fazla 30 saniyede taranır. | Referans cihazlarda (V-05) zamanlı test yapılır. |
| NFR-03 | **Doğruluk:** Birebir kopya tespitinde yanlış pozitif oranı %0 olmalıdır. Benzer tespitinde, varsayılan "Dengeli" seviyede, en az 500 çiftlik etiketli bir test setinde kesinlik (precision) en az %95, duyarlılık (recall) en az %85 olmalıdır. | Etiketli test setiyle ölçülür. Test seti kırpma, döndürme, farklı kişiler ve seri çekim örnekleri içermelidir (3.2-B, [iMore]). |
| NFR-04 | **Ölçeklenebilirlik:** Fotoğraf sayısı iki katına çıktığında tarama süresi en fazla 2,5 katına çıkar. 50.000 fotoğraflık kütüphane çökmeden ve bellek hatası vermeden taranır. | 10.000 / 20.000 / 50.000 fotoğraflık test kütüphaneleriyle ölçülür (3.2-D). |
| NFR-05 | **Arayüz akıcılığı:** Tarama sürerken arayüz donmaz. İlerleme göstergesi en geç 2 saniyede bir güncellenir. İptal komutundan sonra tarama en geç 2 saniye içinde durur. | Manuel ve otomatik arayüz testleriyle ölçülür. |
| NFR-06 | **Veri güvenliği:** Kullanıcı onayı olmadan hiçbir fotoğraf silinmez ya da değiştirilmez. Silme işlemi yarıda kesilirse (uygulama kapanırsa, izin geri alınırsa) hiçbir fotoğraf kalıcı olarak kaybolmaz. | Kesinti senaryolarıyla test edilir. Silinen her öğe platformun çöp alanında bulunmalıdır. |
| NFR-07 | **Platform:** Uygulama iOS ve Android telefonlarda çalışır. Flutter, görev metninde paydaş tarafından verilmiş bir kısıttır. Desteklenecek minimum işletim sistemi sürümleri TA ve paydaş tarafından belirlenecektir (bkz. S-03). | Her iki platformda kabul testleri geçer. |
| NFR-08 | **Kısmi erişim uyumu:** Uygulama iOS sınırlı erişim modunda ve Android 14'ün seçili fotoğraf erişim modunda hata vermeden çalışır. Yalnızca erişilebilen fotoğrafları tarar. | Kısmi erişim senaryolarıyla test edilir ([WWDC20 10641], [Android 14 kısmi erişim]). |
| NFR-09 | **Erişilebilirlik:** Tüm etkileşimli öğelerin ekran okuyucu etiketi vardır. Sistem yazı boyutu büyütüldüğünde metinler kesilmez. | Platform erişilebilirlik denetim araçlarıyla kontrol edilir, kritik hata sayısı 0 olmalıdır. |
| NFR-10 | **Kararlılık:** Kullanıcı oturumlarının en az %99,5'i çökmesiz tamamlanır. | Yayın sonrası çökme raporlarıyla izlenir. Rapor toplama, NFR-01 ile uyumlu olacak şekilde fotoğraf verisi içermemelidir. |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: İzin ve kısmi erişim** (FR-01, NFR-08)
Gizliliğine önem veren bir kullanıcı olarak, uygulamanın neden fotoğraf erişimi istediğini anlamak ve istersem yalnızca bazı fotoğraflara izin vermek istiyorum. Böylece kontrolün bende olduğunu bilirim.
- Uygulama ilk açıldığında izin istenmez. Kullanıcı "Taramayı başlat"a dokunduğunda istenir.
- Kısmi erişim verildiyse ekranda "Yalnızca seçtiğiniz N fotoğraf taranabilir" mesajı ve "Seçimi genişlet" düğmesi görünür.
- Erişim reddedildiyse uygulama çökmez. Cihaz ayarlarına giden bir açıklama gösterilir.

**US-02: Kütüphaneyi tarama** (FR-02, NFR-02, NFR-05)
Alanı dolan bir kullanıcı olarak, tek dokunuşla tarama başlatıp ne kadar kaldığını görmek istiyorum. Böylece beklerken telefonu kullanmaya devam edebilirim.
- Tarama sırasında "X / Y fotoğraf, %Z" bilgisi en geç 2 saniyede bir güncellenir.
- "İptal"e dokunulduğunda tarama en geç 2 saniye içinde durur ve o ana kadar bulunan gruplar gösterilir. Bulunan grup yoksa bu bilgi verilir.
- 10.000 fotoğraflık kütüphane referans cihazda en fazla 10 dakikada taranır.

**US-03: Birebir kopyaları görme** (FR-03, FR-06)
Bir kullanıcı olarak, aynı fotoğrafın birebir kopyalarını ayrı bir başlık altında görmek istiyorum. Böylece bunları gönül rahatlığıyla silebilirim.
- Byte düzeyinde aynı iki fotoğraf aynı "Birebir kopya" grubunda görünür.
- Tek bir piksel bile farklı olan fotoğraflar bu türdeki bir grupta yer almaz.
- Her grupta fotoğraf sayısı ve kazanılacak alan gösterilir.

**US-04: Benzer fotoğrafları görme ve hassasiyeti ayarlama** (FR-04, FR-05, NFR-03)
Aynı sahneyi çok kez çeken bir kullanıcı olarak, benzer kareleri gruplanmış görmek ve gerekirse eşleşmeyi daha sıkı ya da daha gevşek yapmak istiyorum. Böylece yalnızca gerçekten gereksiz kareleri temizlerim.
- Yeniden boyutlandırılmış veya yeniden sıkıştırılmış bir kopya, "Dengeli" seviyede orijinaliyle aynı "Benzer" grubunda görünür.
- Seviye "Sıkı"ya alındığında grup sayısı "Dengeli"deki sayıdan fazla olmaz.
- Etiketli test setinde "Dengeli" seviyede kesinlik en az %95, duyarlılık en az %85 olur.

**US-05: Önerilen tutulacak fotoğraf** (FR-07, FR-08)
Bir kullanıcı olarak, her grupta hangi fotoğrafın tutulmasının önerildiğini ve nedenini görmek istiyorum. Böylece tek tek karşılaştırmak zorunda kalmam.
- Her grupta tam olarak bir fotoğraf "Tutulacak" etiketiyle işaretlidir.
- Grupta favori işaretli bir fotoğraf varsa, bu fotoğraf silinmek üzere önceden seçilmez.
- Kullanıcı bir fotoğrafa dokunduğunda tam ekran, yakınlaştırılabilir görünüm açılır. Çekim tarihi, çözünürlük ve dosya boyutu gösterilir.

**US-06: Seçimi düzeltme ve "Kopya değil" işareti** (FR-09)
Bir kullanıcı olarak, yanlış gruplanmış fotoğrafları "Kopya değil" diye işaretlemek istiyorum. Böylece aynı grup her taramada karşıma çıkmaz.
- Kullanıcı herhangi bir fotoğrafın tutulacak/silinecek durumunu değiştirebilir.
- "Kopya değil" olarak işaretlenen grup sonraki taramada gösterilmez.
- Bu işaret ayarlardaki listeden geri alınabilir.

**US-07: Güvenli silme ve toplu temizlik** (FR-10, FR-11, FR-12, NFR-06)
Alanı dolan bir kullanıcı olarak, seçtiğim kopyaları topluca silmek, ama yanlışlık yaparsam geri alabilmek istiyorum.
- Silmeden önce "N fotoğraf silinecek, yaklaşık X MB alan açılacak" yazan bir onay diyaloğu gösterilir.
- Onay verilmeden hiçbir fotoğraf silinmez.
- Silinen fotoğraflar platformun çöp alanında (iOS'ta Son Silinenler, Android'de çöp kutusu) bulunur ve oradan geri alınabilir.
- "Tümünü temizle" işlemi, tüm grupların seçili fotoğraflarını tek onayla siler.
- İşlem sonunda silinen sayı ve kazanılan alan gösterilir.

**US-08: Yeniden tarama** (FR-13)
Düzenli temizlik yapan bir kullanıcı olarak, ikinci taramanın yalnızca yeni fotoğrafları işlemesini istiyorum. Böylece her seferinde uzun süre beklemem.
- Önceki taramadan sonra 100 yeni fotoğraf eklendiyse artımlı tarama referans cihazda en fazla 30 saniye sürer.
- Önceki taramadan sonra silinmiş fotoğraflar sonuçlarda görünmez.
- Ayarlarda "Tam yeniden tara" seçeneği vardır.

**US-09: Tek albüm tarama** (FR-14)
Bir kullanıcı olarak, yalnızca belirli bir albümü taramak istiyorum. Böylece hızlıca odaklı bir temizlik yapabilirim.
- Tarama başlatılırken "Tüm kütüphane" ya da tek bir albüm seçilebilir.
- Albüm seçildiyse sonuçlarda yalnızca o albümdeki fotoğraflar yer alır.

---

## 7. Kapsam dışı maddeler

| Madde | Neden kapsam dışı |
|---|---|
| Video kopyalarını bulma | Görev metni "Photo Library" diyor. Gemini de videoları desteklemiyor. [MacPaw SSS] Video desteği ileride ayrıca değerlendirilebilir (S-01). |
| Bulanık fotoğraf, ekran görüntüsü, not fotoğrafı gibi "Clutter" temizliği | Rakiplerde var [MacPaw SSS], [App Store – Tidy], ancak görev yalnızca kopya bulmayı istiyor. |
| Yapay zekâ ile estetik "en iyi kare" puanlama (yüz ifadesi, gözler açık mı vb.) | FR-07'deki kural tabanlı öneri MVP için yeterli. Bu özellik ürün büyüklüğünü aşıyor (S-06). |
| Fotoğraf düzenleme, filtreler, seriler (streaks), "Bu gün" anıları | Tidy'de var [App Store – Tidy], ancak kopya bulma işleviyle ilgisi yok. |
| Bulut hesaplarındaki (iCloud, Google Photos) yalnızca bulutta duran fotoğrafları tarama veya bulut tarafında işlem yapma | NFR-01'deki cihazda işleme ilkesiyle ve ürün büyüklüğüyle uyumlu değil. iOS sınırlı modda paylaşılan bulut albümlerine zaten erişilemiyor. [WWDC20 10641] |
| Kullanıcı hesabı, giriş yapma, cihazlar arası senkronizasyon | Ürün tek cihazda çalışan bir yardımcı uygulama. |
| Fotoğrafları birleştirme (merge): meta verileri tek fotoğrafta toplama | iOS'un kendi aracı bunu yapıyor [MacRumors], ancak platformlar arasında aynı davranışı sağlamak belirsiz. MVP'de "tut + sil" yeterli kabul edildi (S-07). |
| Kalıcı silme (çöp alanını atlama) | NFR-06 gereği güvenlik önceliği var. |
| Algoritma, kütüphane ve veri saklama yönteminin seçimi | Mimari karar olduğu için TA'ya bırakıldı (kapsam kuralları). |

---

## 8. Varsayımlar

- **V-01:** Hedef pazar ve dil tanımlanmadı. Arayüz dilinin en az Türkçe olacağı varsayıldı (S-02).
- **V-02:** Uygulamanın ücretli mi, reklamlı mı, ücretsiz mi olacağı bilinmiyor. Gereksinimler gelir modelinden bağımsız yazıldı (S-04).
- **V-03:** "Benzer fotoğraf" kapsamına yeniden sıkıştırılmış ya da boyutu değiştirilmiş kopyalar ve saniyeler arayla çekilmiş kareler girer. Belirgin biçimde kırpılmış, döndürülmüş veya aynalanmış kopyaların yakalanması zorunlu değil. Algısal özetler bu durumlarda zayıf [mojoauth], bu yüzden beklenti paydaşla netleştirilmeli (S-05).
- **V-04:** NFR-02, NFR-03, NFR-04 ve NFR-10'daki sayısal hedefler (10 dakika, %95/%85, 2,5 kat, %99,5) analist önerisidir, kaynaktan alınmamıştır. Paydaş onayı ve referans cihaz testleriyle kesinleşmelidir.
- **V-05:** "Referans orta segment cihaz" tanımlanmadı. Her platformdan en az bir orta segment cihazın belirleneceği varsayıldı (S-03).
- **V-06:** Kullanıcıların tipik kütüphane boyutunun 1.000 ile 50.000 fotoğraf arasında olduğu varsayıldı. Bu konuda kaynaklı bir istatistik bulunamadı.
- **V-07:** Uygulamanın, platformun izin verdiği ölçüde, kullanıcının onayladığı fotoğrafları silebileceği ve silinenlerin platformun çöp alanına gideceği varsayıldı. İki platformda bunun teknik olarak nasıl sağlanacağını TA doğrulamalıdır.
- **V-08:** Kopya tespiti için fotoğrafın çekim tarihi, çözünürlük, dosya boyutu ve favori bilgisi gibi meta verilere erişilebildiği varsayıldı (FR-07, FR-08).

---

## 9. Açık sorular

- **S-01:** Videolar ileride kapsama alınacak mı? Alınacaksa MVP'nin veri modeli buna göre hazırlanmalı mı?
- **S-02:** Hedef pazar ve desteklenecek diller hangileri?
- **S-03:** Desteklenecek minimum iOS ve Android sürümleri ile referans test cihazları hangileri?
- **S-04:** Gelir modeli ne olacak (ücretsiz, tek seferlik ödeme, abonelik, reklam)? Rakipler abonelik kullanıyor ([iMore], [App Store – Tidy]). Reklam ya da analitik eklenirse NFR-01'deki gizlilik vaadi nasıl korunacak?
- **S-05:** Kırpılmış veya döndürülmüş kopyaların yakalanması iş açısından önemli mi? Önemliyse algısal özet yerine ya da onunla birlikte görüntü vektörü yaklaşımı gerekebilir. Bu, uygulama boyutunu ve tarama süresini etkiler (3.2-C).
- **S-06:** "Tutulacak" fotoğraf önerisinde FR-07'deki kural sırası paydaş için uygun mu? Örneğin "en yeni" ya da "düzenlenmiş olan" öncelikli olsun mu? Gemini düzenlemeleri dikkate alıyor. [MacPaw SSS]
- **S-07:** Silme yerine iOS'taki gibi meta verileri birleştiren bir "merge" davranışı isteniyor mu?
- **S-08:** iOS'un kendi Duplicates aracının benzer fotoğrafları kapsayıp kapsamadığı konusunda kaynaklar çelişiyor ([Setapp] ve [MacRumors]). Ürünün iOS'taki farkını netleştirmek için bu, cihaz üzerinde denenerek doğrulanmalı.
- **S-09:** Seri çekim (burst) fotoğrafları taramaya dahil edilecek mi? iOS aracı bunları kopya saymıyor [Setapp], Gemini ise taramaya almıyor. [MacPaw SSS]
- **S-10:** Tarama uygulama arka plandayken de devam etmeli mi, yoksa yalnızca uygulama açıkken mi çalışmalı?

---

## 10. Kaynaklar

| Kısaltma | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| [MacPaw SSS] | Gemini Photos FAQ (MacPaw) | https://macpaw.com/support/gemini-photos/faq | Similar / Clutter / Other kategorileri, Best Result kriterleri (düzenleme, favori, yüz), favorilerin otomatik seçilmemesi, Recently Deleted'da 30 gün, cihazda çevrimdışı işleme, video desteğinin olmaması, seri çekim ve gizli fotoğrafların dışlanması, saniyeler arayla çekilen kareler. |
| [iMore] | Gemini Photos Review (iMore) | https://imore.com/gemini-photos-iphone-clears-out-clutter | Fiyatlar (1,99 $/ay, 11,99 $/yıl, 14,99 $ tek seferlik, 3 gün deneme), farklı kişileri eşleştirebilmesi, benzer karelerin her zaman gereksiz olmaması, grup gizleme. |
| [MacRumors] | How to Remove Duplicates in Your Photo Library (MacRumors) | https://www.macrumors.com/how-to/remove-duplicates-photo-library-ios/ | iOS 16 Duplicates albümü, Merge ve Select All, en yüksek detay ve meta verinin tutulması, kademeli tarama, benzer kareler hakkındaki iddia. |
| [Setapp] | How to delete duplicate photos on iPhone (Setapp) | https://setapp.com/how-to/delete-duplicate-photos-on-iphone | iOS aracının yalnızca birebir kopyaları bulması, Recently Deleted'da 30 gün, seri çekimlerin kopya sayılmaması, taramanın birkaç gün sürebilmesi. |
| [Google Files Yardım] | Delete duplicate files (Files by Google Yardım) | https://support.google.com/files/answer/9764075?hl=en | "Duplicate files" kartı, "Original" rozeti, çöp kutusuna taşıma ve onay diyaloğu. Tespit yönteminin belgelenmemiş olması. |
| [App Store – Tidy] | Tidy: Photo & Gallery Cleaner (App Store) | https://apps.apple.com/us/app/-/id1330584027 | Kopya ve benzer gruplama, video desteği, bulanık fotoğraf filtresi, Best-Shot Auto-Pick, karşılaştırma modu, tek albüm temizleme, fiyatlar, 4,8 puan, gizlilik etiketi. |
| [mojoauth] | pHash vs dHash: Perceptual Image Hashing | https://mojoauth.com/security-guides/phash-vs-dhash.md | aHash, dHash ve pHash'in nasıl çalıştığı, Hamming uzaklığı, kırpma ve aynalama testlerindeki uzaklıklar, yaklaşık 10'luk başlangıç eşiği. |
| [HackerFactor] | Kind of Like That (Neal Krawetz, HackerFactor) | https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html | dHash adımları, üç yöntemin hız (3,75 saat ve 7+ saat) ve doğruluk karşılaştırması, aHash'in yanlış pozitifleri, 0 / 1–10 / 10+ eşik yorumu. |
| [Ben Hoyt] | Duplicate image detection with perceptual hashing in Python | https://benhoyt.com/writings/duplicate-image-detection/ | 128 bit dHash, 2 bitlik eşik, 4–5 bit eşiklerdeki yanlış pozitifler, O(N²) sorunu, BK-tree kullanımı ve gerçek maliyet gözlemi. |
| [Wikipedia – BK-tree] | BK-tree (Wikipedia) | https://en.wikipedia.org/wiki/BK-tree | BK-tree tanımı, üçgen eşitsizliğine dayalı budama, ortalama Θ(log n) ve kötü durumda Θ(n) karmaşıklık. |
| [Wikipedia – Algısal özet] | Perceptual hashing (Wikipedia) | https://en.wikipedia.org/wiki/Perceptual_hashing | Algısal ve kriptografik özet farkı (çığ etkisi), ilgisiz görüntülerin eşleşebilmesi. |
| [Wikipedia – Kriptografik özet] | Cryptographic hash function (Wikipedia) | https://en.wikipedia.org/wiki/Cryptographic_hash_function | Kriptografik özetin kopya veriyi tespit etmek ve dosyaları tanımlamak için kullanılması, SHA-256. |
| [arXiv 2206.00282] | Needle In A Haystack, Fast: Benchmarking Image Perceptual Similarity Metrics At Scale | https://arxiv.org/abs/2206.00282 | dHash ile SimCLR v2 ResNet yaklaşımlarının ikisinin de çok iyi sonuç vermesi, klasik yöntemlerin her zaman en iyi seçim olmaması (yalnızca özet okundu). |
| [MediaPipe] | Image embedding task guide (MediaPipe, Google) | https://developers.google.com/edge/mediapipe/solutions/vision/image_embedder | Görüntü vektörü ve kosinüs benzerliği, desteklenen platformlar (Flutter dahil), MobileNet V3 Small/Large, Pixel 6 gecikme değerleri. |
| [Android 14 kısmi erişim] | Grant partial access to photos and videos (Android Developers) | https://developer.android.com/about/versions/14/changes/partial-photo-video-access | Seçili fotoğraf erişimi, tam/kısmi/reddedilmiş durumların kontrolü, seçimi genişletme, iznin ihtiyaç anında istenmesi. |
| [WWDC20 10641] | Handle the Limited Photos Library in your app (WWDC20, Apple) | https://developer.apple.com/videos/play/wwdc2020/10641/ | iOS sınırlı erişim modu, yalnızca seçilen öğelerin görünmesi, paylaşılan bulut albümlerine erişilememesi, seçimi değiştirme arayüzü önerisi. |

---

Notlar:

- **Uygulanan kural:** Görev metnindeki Flutter, paydaşın verdiği bir kısıt olarak NFR-07'ye yazıldı. Algoritma, kütüphane ve veri saklama seçimleri yapılmadı. Yöntemler yalnızca iş açısından karşılaştırıldı ve karar TA'ya bırakıldı.
- **Kaynak gösterilmeyen sayfalar:** Üç sayfa açıldı ama dokümanda kaynak gösterilmedi:
  - Apple'ın `PHAuthorizationStatus.limited` dokümantasyon sayfası: içerik olarak yalnızca başlık geldi, yerine WWDC20 oturumu kullanıldı.
  - "Similar Photo Cleaner AI" Google Play sayfası: içerik gelmedi.
  - arXiv 2304.02296: özet sayfası algoritma karşılaştırması içermiyor.
- **Önerilen dosya adı:** `outputs/photo-duplicate-finder/ba-photo-duplicate-finder-v1-run1.md` (senaryo adı tarafınızca değiştirilebilir).