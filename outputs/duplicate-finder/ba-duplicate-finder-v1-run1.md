# İş Analizi: Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** "Flutter ile yazacağımız ve cep telefonlarında çalışacak bir Photo Library duplicate finder uygulaması hazırlayacağız. Gerekli algoritmaları araştırıp bana bir iş analizi çıkar."

---

## 1. Özet

Bu uygulama telefonun fotoğraf kütüphanesini tarar. Birebir aynı kopyaları ve gözle aynı görünen kopyaları (yeniden boyutlandırılmış, yeniden sıkıştırılmış, format değiştirilmiş) bulup gruplar. Her grupta saklanacak en iyi kareyi önerir. Kullanıcı onay verdikten sonra fazlalıkları silerek depolama alanı açar. Kaydırmalı (swipe) inceleme veya önizleme ekranı gibi ek etkileşimler ancak "Açık sorular" bölümündeki kararlardan sonra değerlendirilecektir.

Görevde Flutter verilmiştir. Bu, paydaşın koyduğu bir kısıttır ve BA'nın verdiği bir karar değildir. Görevden uygulamanın hem iOS hem Android'de çalışacağı anlaşılıyor. Bu varsayım "Varsayımlar" bölümünde ayrıca yazılıdır.

**Araştırmanın ana bulguları:**
- **Kopya bulma yöntemleri katman katmandır.** Dosya özeti (kriptografik hash) yalnızca bayt bayt aynı dosyaları yakalar. Algısal özet (perceptual hash: aHash, dHash, pHash) yeniden boyutlandırma ve sıkıştırmaya dayanıklıdır, ancak kırpma ve döndürmede zayıf kalır. Görüntü gömmeleri (embedding, CNN/CLIP) kırpılmış veya açısı değişmiş "benzer" kareleri de yakalar, ama hesaplama maliyeti yüksektir ve yanlış eşleşme riski taşır. Kaynaklar 3.2 bölümündedir.
- **Rakip uygulamalar** ayrımı genelde iki grupla yapar: "birebir aynı" ve "benzer". Otomatik en iyi kare seçimi, hızlı tarama ve inceleme ekranı öne çıkan özelliklerdir. Şikâyetler pahalı abonelik, yanlış gruplama, çökme ve gereksiz ek özellikler üzerinde toplanır.
- **İşletim sistemleri iki risk getirir.** Birincisi kullanıcı yalnızca seçili fotoğraflara izin verebilir (kısıtlı erişim). İkincisi Google Play, geniş fotoğraf erişim iznini beyan ve inceleme şartına bağlamıştır. Bu ürünün çekirdek değeri tüm kütüphaneye erişime dayandığı için bu iki konu iş riski olarak ele alınmalıdır.

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Profil | İhtiyaç / acı noktası | Başarı ölçütü |
|---|---|---|---|
| **P1: Alanı dolan günlük kullanıcı** | Teknik bilgisi düşük. Binlerce fotoğrafı var, WhatsApp vb. kaynaklardan aynı görselin kopyaları birikmiş. | "Depolama dolu" uyarısından kurtulmak, yanlışlıkla önemli bir fotoğrafı silmemek. | Birkaç dokunuşla güvenle yer açmak, ne kadar yer açıldığını görmek. |
| **P2: Seri çekim yapan ebeveyn veya hobi fotoğrafçısı** | Aynı anı için art arda çok kare çeker. | Benzer karelerden en net olanı tutmak, diğerlerini temizlemek. | Grubun içinden en iyi kareyi hızla seçmek. |
| **P3: Gizliliğe duyarlı kullanıcı** | Fotoğraflarının buluta gönderilmesinden çekinir, izinleri dikkatle verir. | Fotoğrafların cihazdan çıkmadığından emin olmak, kısıtlı izinle de uygulamayı deneyebilmek. | Uygulamanın internete fotoğraf göndermemesi, izinlerin neden istendiğinin açıkça anlatılması. |

Persona'lar görev metni ve rakip incelemesindeki kullanım senaryolarından türetilmiştir. Kullanıcı araştırmasıyla doğrulanmamıştır (bkz. Varsayımlar).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Uygulamalar ve yerleşik çözümler

| Çözüm | Öne çıkan özellikler | Güçlü yön | Zayıf yön | Kaynak |
|---|---|---|---|---|
| **Apple Fotoğraflar: Kopyalar (Duplicates) albümü** | iOS 16 ile geldi. Kopyaları otomatik bulur ve "Yardımcı Öğeler" altında toplar. "Birleştir" (Merge) en yüksek kaliteli sürümü tutar, diğerini "Son Silinenler"e taşır. | Ücretsiz ve yerleşik. Birleştirme en yüksek detayı ve en fazla meta veriyi korur. | İndeksleme için cihazın kilitli ve şarjda olması gerekir, birkaç gün sürebilir. Algılama kuralı açıklanmamıştır. | [Apple 102260](https://support.apple.com/102260), [PopSci](https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios), [MacRumors](https://www.macrumors.com/how-to/remove-duplicates-photo-library-ios/) |
| **Files by Google** (Android) | "Temizle" sekmesinden kopyaları tek tek veya toptan siler, yalnızca kopyaları kaldırır. | Ücretsiz ve basit. | Kaynakta yalnızca dosya kopyalarından söz ediliyor. Görsel benzerlik desteği belirtilmemiş. | [PopSci](https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios) |
| **Samsung My Files** (yalnızca Galaxy) | "Depolamayı yönet > Kopya dosyalar". Kopyaları orijinalin yanında gösterir. | Yan yana inceleme. | Yalnızca Samsung cihazlarda var. | [PopSci](https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios) |
| **Google Fotoğraflar** | Aynı dosyanın buluta tekrar yüklenmesini engeller. | Bulutta kopya birikmez. | Cihazdaki yerel kopyaları temizlemez. Benzer fotoğraflar yine görünebilir. | [PopSci](https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios) |
| **Clever Cleaner** (iOS) | Benzer fotoğraf algılama, 10.000'den fazla fotoğrafı 5 dakikanın altında tarama, kaydırmalı inceleme, kategori görünümleri. | Testte en yüksek puanı aldı (20/5×4). Reklamsız ve cömert ücretsiz sürüm. | iPad'de uyumluluk modunda çalışıyor. | [CleverFiles incelemesi](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html) |
| **Cleaner Guru** (iOS) | Birebir ve benzer kopya, otomatik en iyi kare seçimi, büyük kütüphane desteği. | İyi algılama. | Pahalı (haftalık 7,99 USD), karmaşık arayüz, gereksiz ek özellikler. | aynı |
| **CleanMy®Phone** (iOS) | Tek dokunuşla temizlik, yapay zekâ ile gruplama, bulanık fotoğraf tespiti, durdurup sürdürülebilen tarama. | Hızlı tarama, duraklat/sürdür. | Farklı kişileri veya pozları zaman zaman aynı gruba koyuyor, açılacak toplam alanı tek ekranda göstermiyor, pahalı. | aynı |
| **Boost Cleaner** (iOS) | Temkinli algılama, tarih ve konum filtreleri, yerel işleme. | Güçlü yerel gizlilik. | Bazı kopyaları kaçırıyor, çökme bildirimleri var. | aynı |
| **Remo Duplicate Photos Remover** (iOS) | Sonuçları "Birebir" ve "Benzer" diye ayırır, kopyaları otomatik işaretler. | Tamamen ücretsiz ve reklamsız. | Çok yavaş tarama, eski arayüz, birebir kopyaları zaman zaman yanlış sınıflıyor. | aynı, [PopSci](https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios) |

**Rakiplerden çıkan dersler:**
- CleverFiles incelemesinde en önemli kriter **algılama doğruluğudur**. Ardından tarama hızı, ücretsiz kullanım ve kullanılabilirlik gelir. Video sıkıştırma, rehber temizliği gibi ek özellikler zorunlu değil, "bonus" sayılmıştır ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)). Bu, kapsamı küçük tutmayı destekler.
- "Birebir" ve "Benzer" ayrımı, otomatik işaretleme ve son kararın kullanıcıda kalması ortak örüntüdür.
- Yanlış gruplama (farklı kişi veya poz) ve "silme sorunları" güveni en çok zedeleyen hatalardır.

### 3.2 Kopya bulma yöntemleri (iş açısından)

Bu bölüm yöntemleri yalnızca iş açısından karşılaştırır. Hangi yöntemin, hangi eşikle ve hangi teknolojiyle uygulanacağına TA karar verir.

| Yöntem | Ne yakalar | Ne kaçırır / riski | İş açısından artı | İş açısından eksi | Kaynak |
|---|---|---|---|---|---|
| **Dosya özeti** (MD5/SHA-256) | Bayt bayt aynı kopyalar. | Yeniden boyutlandırılmış, sıkıştırılmış veya meta verisi değişmiş kopyalar. | Yanlış eşleşme pratikte yoktur, bu yüzden güvenle "birebir" etiketi verilebilir. | WhatsApp vb. üzerinden yeniden sıkıştırılmış kopyaları bulamaz. | [dev.to (Kencho)](https://dev.to/kencho/duplicate-image-detection-how-to-find-and-remove-duplicate-photos-5dod) |
| **aHash** (ortalama özet) | Ölçek değişikliği, orta düzey renk ve parlaklık değişimi. | Gama ve histogram ayarlarında kaçırır, çok sayıda yanlış eşleşme üretir. 150.000 görsellik testte 32 olması gereken eşleşme 400'ün üzerine çıktı. | Çok hızlı. | Yanlış eşleşme yüzünden kullanıcının güveni sarsılır. | [HackerFactor: Looks Like It](https://www.hackerfactor.com/blog/index.php?/archives/432-Looks-Like-It.html), [HackerFactor: Kind of Like That](https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html) |
| **dHash** (fark özeti) | Ölçek ve en-boy oranı değişimi, parlaklık, kontrast, renk ve gama değişimi. | Kırpma, döndürme. Eşik yükseldikçe kesinlik düşer (imagededup: eşik 10'da kesinlik 0,223). | En hızlı özetlerden biri. Eşik 0'da birebir kopyalarda kesinlik ve duyarlılık 1,0. Büyük ölçekli bir karşılaştırmada iyi performans gösterdiği ve iyi ölçeklendiği bildirildi. | Gevşek eşikte yanlış eşleşme artar. | [HackerFactor](https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html), [imagededup benchmark](https://idealo.github.io/imagededup/user_guide/benchmarks/), [arXiv 2206.00282](https://arxiv.org/abs/2206.00282) |
| **pHash** (DCT tabanlı) | dHash'in yakaladıklarına ek olarak küçük düzenlemeler (resmin yaklaşık %25'inden azı). | Kırpma ve döndürme gibi geometrik dönüşümler. | Testte en doğru özet oldu (yanlış pozitif ve yanlış negatif yok). Eşik 10'da bile kesinlik 0,980. | Hesaplaması daha pahalı. Bir testte dHash'in yaklaşık iki katı süre aldı. | [HackerFactor](https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html), [imagededup benchmark](https://idealo.github.io/imagededup/user_guide/benchmarks/) |
| **Görüntü gömmeleri** (embedding: CNN, CLIP vb.) | Kırpma, döndürme, filtre, farklı açıdan çekilmiş aynı sahne (seri çekim). | Benzer ama farklı nesneleri aşırı eşleştirebilir. | "Benzer fotoğraflar" kategorisi için gereklidir. Eşik 0,9'da kesinlik yaklaşık 1,0. | Özetlemeden 10-15 kat yavaş (GPU'suz testte). Dönüşüme uğramış görsellerde duyarlılık yalnızca 0,384. Cihazda pil ve süre maliyeti var. | [imagededup benchmark](https://idealo.github.io/imagededup/user_guide/benchmarks/), [imagededup](https://idealo.github.io/imagededup/), [dev.to](https://dev.to/kencho/duplicate-image-detection-how-to-find-and-remove-duplicate-photos-5dod) |
| **Katmanlı yaklaşım** | Önce dosya özeti, sonra algısal özet, en son gömme. Her ucuz adım kolay vakaları eler. | — | Hız ile doğruluğu dengeler. HackerFactor da hızlı dHash ile ön eleme yapıp yalnızca adaylarda pHash çalıştırmayı önerir. | Daha fazla bileşen, daha fazla ayar demektir. | [dev.to](https://dev.to/kencho/duplicate-image-detection-how-to-find-and-remove-duplicate-photos-5dod), [HackerFactor](https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html) |
| **Bulanıklık skoru** (Laplace varyansı) | "En iyi kare" önerisi için netlik ölçümü. | Eşik içeriğe bağlıdır ve veri setine göre ayarlanmalıdır. | Seri çekimlerde en net kareyi önermeye yardım eder. | Yanlış eşik, net kareyi bulanık sayabilir. | [PyImageSearch](https://pyimagesearch.com/2015/09/07/blur-detection-with-opencv/) |

**Eşikler hakkında kaynakların söyledikleri:** 64 bitlik algısal özetlerde iki görselin farkı Hamming uzaklığıyla ölçülür.
- HackerFactor'a göre 0 büyük olasılıkla aynı resim, 1-10 olası varyasyon, 10'un üstü büyük olasılıkla farklı resimdir.
- Bir uygulayıcı yazısı üretimde genellikle 5 ve altının kullanıldığını söyler ([dev.to](https://dev.to/kencho/duplicate-image-detection-how-to-find-and-remove-duplicate-photos-5dod)).
- Kaynaklar eşiklerin veri setine göre ayarlanması gerektiğinde birleşir. Ürün bu yüzden FR-13 ile duyarlılık ayarı sunar.

### 3.3 Platform kısıtları (iş riski olarak)

| Konu | Bulgu | İş etkisi | Kaynak |
|---|---|---|---|
| **iOS kısıtlı erişim** | Kullanıcı "Fotoğrafları Seç" diyerek yalnızca seçtiği öğelere izin verebilir. Bu modda paylaşılan bulut öğelerine erişilemez. Seçim sonradan değişebilir. Apple, uygulama içinde seçimi değiştirme düğmesi sunulmasını önerir. | Kısıtlı izinde tarama eksik kalır. Kullanıcıya bu açıkça gösterilmelidir. | [WWDC20 10641](https://developer.apple.com/videos/play/wwdc2020/10641/) |
| **Android 14 ve sonrası seçili fotoğraf erişimi** | Kullanıcı tam, kısmi veya hiç erişim verebilir. İzin, uygulama açıkken bile değişebilir. Kısmi erişimde uygulama kütüphanenin boş olduğunu varsaymamalı ve kullanıcıya daha fazla dosya verme imkânı tanımalıdır. İzin, başlangıçta değil ihtiyaç anında istenmelidir. | İzin akışı ve "eksik tarama" durumu tasarlanmalıdır. | [Android: partial photo access](https://developer.android.com/about/versions/14/changes/partial-photo-video-access) |
| **Google Play izin politikası** | READ_MEDIA_IMAGES ve READ_MEDIA_VIDEO kısıtlı izinlerdir. Yalnızca çekirdek işlevi fotoğraf ve videolara geniş erişim gerektiren uygulamalar beyan sunarak kullanabilir. Google her başvuruyu inceler. Adıyla anılan tek örnek galeri uygulamalarıdır. Kopya temizleyiciler açıkça anılmaz. | **Yayın riski:** beyan reddedilirse ürünün çekirdek işlevi Android'de çalışmaz. | [Play politika 14115180](https://support.google.com/googleplay/android-developer/answer/14115180), [Play 15800983](https://support.google.com/googleplay/android-developer/answer/15800983), [Android MediaStore](https://developer.android.com/training/data-storage/shared/media) |
| **Android'de silme onayı** | Android 11 ve üstünde, varsayılan galeri olmayan uygulamalar başka uygulamaların dosyalarını değiştirir veya silerken sistem bir onay penceresi gösterir. Toplu çöp kutusuna taşıma ve kalıcı silme için ayrı istekler vardır. Çöpteki öğeler sistemin belirlediği sürenin sonunda kalıcı silinir. Android 12 ve üstünde kullanıcı "medya yönetimi" özel iznini verirse her işlemde onay gerekmez. | Silme akışına sistem penceresi girer. Çöp kutusu kullanılırsa geri alma imkânı doğar. | [Android MediaStore](https://developer.android.com/training/data-storage/shared/media) |
| **iOS'ta silinenlerin saklanması** | "Son Silinenler" albümündeki öğeler 30 gün saklanır, sonra kalıcı olarak silinir. | Kullanıcıya "30 gün içinde geri alabilirsiniz" mesajı verilebilir. Bunun için uygulamanın sildiği öğelerin de bu albüme gittiği doğrulanmalıdır (bkz. Açık sorular). | [Apple 124460](https://support.apple.com/124460) |

## 4. Fonksiyonel gereksinimler

Öncelikler MoSCoW yöntemiyle verilmiştir: **M** = Olmazsa olmaz (Must), **S** = Olmalı (Should), **C** = Olsa iyi olur (Could).

| ID | Gereksinim | Öncelik |
|---|---|---|
| **FR-01** | Uygulama fotoğraf erişim iznini ilk açılışta değil, kullanıcı taramayı başlatmak istediğinde ister. İzinden önce iznin neden gerektiğini ve fotoğrafların cihazdan çıkmadığını anlatan bir açıklama ekranı gösterir. | M |
| **FR-02** | Uygulama üç erişim durumunu ayırt eder ve her biri için farklı ekran gösterir: tam, kısıtlı/seçili ve reddedilmiş. Kısıtlı erişimde "yalnızca seçtiğiniz N fotoğraf tarandı" uyarısı ve erişimi genişletme düğmesi bulunur. Reddedilmiş durumda ayarlara yönlendiren bir düğme bulunur. | M |
| **FR-03** | Uygulama erişilebilen tüm fotoğrafları tarar ve **birebir kopyaları** (içerik olarak bayt bayt aynı dosyalar) "Birebir kopyalar" kategorisinde gruplar. | M |
| **FR-04** | Uygulama **görsel olarak aynı kopyaları** bulur ve "Kopyalar" kategorisinde gruplar. Bunlar farklı çözünürlük, sıkıştırma veya format nedeniyle dosyası farklı ama görüntüsü aynı olan fotoğraflardır. | M |
| **FR-05** | Uygulama **benzer fotoğrafları** (seri çekim, art arda çekilmiş, hafif farklı kareler) kopyalardan ayrı bir "Benzer fotoğraflar" kategorisinde gruplar. Bu kategoride hiçbir fotoğraf otomatik olarak silinmek üzere işaretlenmez. | S |
| **FR-06** | Her grup için fotoğraf sayısı ve silinmesi önerilen fotoğrafların toplam boyutu gösterilir. Ana ekranda tüm kategoriler için toplam açılabilecek alan tek bir değer olarak gösterilir. | M |
| **FR-07** | Uygulama her grupta **saklanacak bir fotoğraf önerir** ve önerinin nedenini gösterir (örn. "en yüksek çözünürlük", "en net", "favori"). Önerilen fotoğraf varsayılan olarak saklanır, diğerleri silinmek üzere işaretlenir. Öneri kriterleri ve öncelik sırası Açık sorular bölümündedir. | M |
| **FR-08** | Kullanıcı bir grubu açıp fotoğrafları büyütülmüş önizlemeyle görebilir. Her fotoğrafın çekim tarihi, çözünürlüğü ve dosya boyutu görünür. Kullanıcı saklanacak ve silinecek seçimini değiştirebilir. Bir grupta en az bir fotoğrafın saklanması zorunludur. | M |
| **FR-09** | Kullanıcı seçili fotoğrafları tek bir işlemle toplu silebilir. Silmeden önce silinecek fotoğraf sayısını ve açılacak alanı gösteren bir onay ekranı çıkar. Silinen fotoğraflar, platform destekliyorsa sistemin geri alınabilir çöp kutusuna veya "Son Silinenler" alanına gider. | M |
| **FR-10** | Silme tamamlandıktan sonra silinen fotoğraf sayısı, açılan alan ve fotoğrafların nereden ve kaç gün içinde geri alınabileceği gösterilir. | M |
| **FR-11** | Tarama sırasında ilerleme (taranan/toplam sayı, yüzde) gösterilir. Kullanıcı taramayı iptal edebilir. Tarama yarıda kalırsa (uygulama kapanması vb.) sonraki açılışta kalınan yerden devam eder. | M |
| **FR-12** | İlk taramadan sonraki taramalarda yalnızca yeni eklenen veya değişen fotoğraflar işlenir. Silinmiş fotoğraflar sonuçlardan çıkarılır. | S |
| **FR-13** | Kullanıcı kopya algılama duyarlılığını üç seviyeden biri olarak seçebilir: Sıkı, Dengeli (varsayılan), Gevşek. Seviye değişince sonuçlar yeniden hesaplanır. | S |
| **FR-14** | Kullanıcı bir grubu "kopya değil" olarak işaretleyebilir. Bu grup sonraki taramalarda tekrar önerilmez. | S |
| **FR-15** | Kullanıcının favori olarak işaretlediği fotoğraflar hiçbir zaman otomatik olarak silinmek üzere işaretlenmez. | S |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt / doğrulama |
|---|---|---|
| **NFR-01** | **Gizlilik:** Fotoğraflar ve fotoğraflardan türetilen veriler (özetler, gömmeler) cihaz dışına gönderilmez. Tüm analiz cihazda yapılır. | Ağ trafiği izlemesinde tarama sırasında fotoğraf veya türetilmiş veri içeren istek sayısı 0. |
| **NFR-02** | **Çevrimdışı çalışma:** Tarama, inceleme ve silme internet bağlantısı olmadan tamamlanabilir. | Uçak modunda uçtan uca senaryo başarılı. |
| **NFR-03** | **Tarama süresi:** Cihazda yerel olarak bulunan 10.000 fotoğraflık kütüphanenin ilk taraması, referans orta segment cihazda (bkz. Açık sorular) **5 dakika veya daha kısa** sürer. Hedef, rakip kıyasından alınmıştır ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)). | Referans cihazda 3 ölçümün ortalaması. |
| **NFR-04** | **Artımlı tarama süresi:** İlk taramadan sonra 100 yeni fotoğrafın taranması 30 saniyeyi aşmaz. | Referans cihazda ölçüm. |
| **NFR-05** | **Arayüz akıcılığı:** Tarama sürerken arayüz donmaz. Kullanıcı etkileşimlerine 200 ms içinde yanıt verilir ve sonuç listesinin kaydırılması takılmaz. | Referans cihazda profil ölçümü, 200 ms'yi aşan yanıt sayısı 0. |
| **NFR-06** | **Birebir kopya doğruluğu:** "Birebir kopyalar" kategorisinde yanlış eşleşme oranı %0'dır (kesinlik = 1,0). | Etiketli test setinde (bkz. Varsayımlar) ölçüm. |
| **NFR-07** | **Görsel kopya doğruluğu:** "Kopyalar" kategorisinde Dengeli seviyede kesinlik en az **%98**, duyarlılık en az **%90**'dır (yeniden boyutlandırma, yeniden sıkıştırma, format dönüşümü içeren kopyalar için). | Etiketli test setinde ölçüm. Hedefler, imagededup benchmark'ında düşük eşikli özet yöntemlerinin birebir kopyalarda ulaştığı değerlere göre ayarlanmıştır ([imagededup](https://idealo.github.io/imagededup/user_guide/benchmarks/)). |
| **NFR-08** | **Silme güvenliği:** Kullanıcının açık onayı (FR-09) olmadan hiçbir fotoğraf silinmez veya çöpe taşınmaz. Bir grubun tüm üyeleri aynı işlemde silinemez (FR-08). | Test senaryolarında onaysız silme sayısı 0. |
| **NFR-09** | **Kararlılık:** Tarama veya silme sırasında çökmesiz oturum oranı en az %99,5'tir. | Sürüm sonrası çökme raporlaması. |
| **NFR-10** | **Kaynak kullanımı:** Uygulamanın kendi tuttuğu tarama verisi (indeks), 10.000 fotoğraf için 50 MB'ı aşmaz. Yer açmaya çalışan bir uygulamanın kendisi yer kaplamamalıdır. | Cihazda uygulama veri boyutu ölçümü. |
| **NFR-11** | **Platform kapsamı:** Uygulama iOS ve Android telefonlarda aynı fonksiyonel kapsamla çalışır. Minimum işletim sistemi sürümleri Açık sorular bölümündedir. | Her iki platformda kabul testleri. |
| **NFR-12** | **Mağaza uyumu:** Uygulama App Store ve Google Play izin politikalarına uyar. Google Play için geniş medya erişimi beyanı hazırlanır ([Play politika](https://support.google.com/googleplay/android-developer/answer/14115180)). | Mağaza incelemesinden geçiş. |
| **NFR-13** | **Erişilebilirlik:** Tüm etkileşimli öğeler ekran okuyucu etiketine sahiptir. Sistem yazı boyutu büyütüldüğünde metinler kesilmez. | Platform erişilebilirlik denetimiyle kontrol. |
| **NFR-14** | **Dil:** Arayüz en az Türkçe sunulur. Ek diller Açık sorular bölümündedir. | Tüm ekranlarda çevrilmemiş metin sayısı 0. |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: İzni anlayarak vermek** (P3). *İlgili: FR-01, FR-02, NFR-01*
> Gizliliğe önem veren bir kullanıcı olarak, uygulamanın neden fotoğraflarıma erişmek istediğini ve fotoğraflarımın cihazdan çıkmayacağını izin vermeden önce öğrenmek istiyorum, çünkü bilinçli karar vermek istiyorum.
- **AC1:** İzin penceresi açılmadan önce açıklama ekranı gösterilir.
- **AC2:** Kullanıcı izni reddederse ayarlara yönlendiren düğmeli bir ekran gösterilir ve uygulama çökmez.
- **AC3:** Kısıtlı izin verilirse taranan fotoğraf sayısı ve "erişimi genişlet" düğmesi görünür.

**US-02: Kütüphaneyi taramak** (P1). *İlgili: FR-03, FR-04, FR-11, NFR-03, NFR-05*
> Alanı dolan bir kullanıcı olarak, tek bir düğmeyle tüm fotoğraflarımı taramak ve ilerlemeyi görmek istiyorum, çünkü işlemin ne kadar süreceğini bilmek istiyorum.
- **AC1:** Tarama sırasında taranan/toplam sayı ve yüzde güncellenir.
- **AC2:** Kullanıcı "İptal"e bastığında tarama durur ve o ana kadar bulunan sonuçlar kaybolmaz.
- **AC3:** Uygulama tarama sırasında kapatılıp açılırsa tarama kaldığı yerden devam eder.
- **AC4:** 10.000 fotoğraflık kütüphanede tarama referans cihazda 5 dakika veya daha kısa sürer.

**US-03: Ne kadar yer kazanacağımı görmek** (P1). *İlgili: FR-06*
> Bir kullanıcı olarak, temizlik yapmadan önce ne kadar yer açabileceğimi görmek istiyorum, çünkü uğraşmaya değip değmeyeceğine karar vermek istiyorum.
- **AC1:** Sonuç ekranında kategori bazında grup sayısı ve açılabilecek alan gösterilir.
- **AC2:** Toplam açılabilecek alan, kullanıcı seçim değiştirdikçe güncellenir.

**US-04: Önerilen en iyi kareyi gözden geçirmek** (P2). *İlgili: FR-07, FR-08, FR-15*
> Seri çekim yapan bir kullanıcı olarak, her grupta hangi fotoğrafın neden saklanması önerildiğini görmek ve gerekirse değiştirmek istiyorum, çünkü en iyi karemi kaybetmek istemiyorum.
- **AC1:** Her grupta bir fotoğraf "Saklanacak" etiketiyle ve öneri nedeniyle gösterilir.
- **AC2:** Kullanıcı başka bir fotoğrafı saklanacak olarak seçebilir. Gruptaki son saklanan fotoğrafın işareti kaldırılamaz.
- **AC3:** Favori fotoğraflar hiçbir zaman otomatik olarak silinmek üzere işaretlenmez.
- **AC4:** Büyütülmüş önizlemede tarih, çözünürlük ve boyut görünür.

**US-05: Güvenle toplu silmek** (P1). *İlgili: FR-09, FR-10, NFR-08*
> Bir kullanıcı olarak, seçtiğim kopyaları tek seferde silmek ve yanlışlık yaparsam geri alabileceğimi bilmek istiyorum.
- **AC1:** "Sil"e basınca silinecek sayı ve açılacak alan içeren bir onay ekranı çıkar.
- **AC2:** Onay verilmeden hiçbir fotoğraf silinmez.
- **AC3:** Silme sonrası özet ekranı açılan alanı ve geri alma yolunu (yer ve süre) gösterir.
- **AC4:** Kullanıcı sistem onay penceresini reddederse hiçbir fotoğraf silinmez ve kullanıcıya nedeni açıklanır.

**US-06: Benzer fotoğrafları ayrı incelemek** (P2). *İlgili: FR-05*
> Bir kullanıcı olarak, birebir kopyalar ile "benzer" kareleri ayrı listelerde görmek istiyorum, çünkü benzer kareleri tek tek değerlendirmem gerekir.
- **AC1:** Benzer fotoğraflar ayrı bir kategoride listelenir.
- **AC2:** Bu kategoride varsayılan olarak hiçbir fotoğraf silinmek üzere işaretlenmez.

**US-07: Sonuçların hassasiyetini ayarlamak** (P2). *İlgili: FR-13, FR-14*
> Bir kullanıcı olarak, uygulama ilgisiz fotoğrafları kopya sanıyorsa duyarlılığı düşürmek veya o grubu "kopya değil" diye işaretlemek istiyorum.
- **AC1:** Duyarlılık değiştirildiğinde sonuç listesi yeniden hesaplanır.
- **AC2:** "Kopya değil" olarak işaretlenen grup sonraki taramalarda gösterilmez.

**US-08: Yeni fotoğraflar için hızlı tekrar tarama** (P1). *İlgili: FR-12, NFR-04*
> Düzenli kullanan biri olarak, uygulamayı tekrar açtığımda yalnızca yeni fotoğrafların taranmasını istiyorum, çünkü her seferinde uzun süre beklemek istemiyorum.
- **AC1:** İkinci taramada yalnızca yeni veya değişen fotoğraflar işlenir.
- **AC2:** Uygulama dışında silinmiş fotoğraflar sonuçlarda görünmez.
- **AC3:** 100 yeni fotoğraf için tarama 30 saniye veya daha kısa sürer.

## 7. Kapsam dışı maddeler

- **Video kopyaları:** Görev "Photo Library" diyor. Videoların kapsama alınıp alınmayacağı Açık sorular bölümündedir.
- Ekran görüntüsü, bulanık fotoğraf veya Live Photo gibi kategori bazlı temizlik. Bulanıklık ölçümü yalnızca en iyi kare önerisinde (FR-07) yardımcı ölçüt olarak kullanılabilir.
- Rakiplerde görülen video ve fotoğraf sıkıştırma, rehber ve e-posta temizliği, gizli klasör, widget. CleverFiles incelemesi bunları zorunlu değil "bonus" sayar.
- Bulut depolamadaki (iCloud, Google Fotoğraflar) fotoğrafların bulut tarafında taranması veya bulut hesabına bağlanma.
- Kullanıcı hesabı, cihazlar arası senkronizasyon, tablet veya masaüstü sürüm.
- Fotoğrafların analiz için sunucuya gönderilmesi (NFR-01 ile çelişir).
- Gruptaki fotoğrafların meta verisini birleştirme (Apple'ın "Birleştir" özelliğinin karşılığı). Gerekirse Açık sorular bölümüne bakılabilir.
- Gelir modeli (abonelik, reklam) ve ödeme akışları. Bu konu Açık sorular bölümündedir.

## 8. Varsayımlar

- **V-01:** "Cep telefonları" ifadesi hem iOS hem Android anlamına gelir.
- **V-02:** Ürün küçük veya orta ölçeklidir. Tek geliştirme ekibi, MVP önceliklidir ve kurumsal altyapı gerekmez.
- **V-03:** Hedef kütüphane büyüklüğü 1.000-50.000 fotoğraftır. NFR-03'teki 10.000 fotoğraf temsili ölçüm noktasıdır.
- **V-04:** Persona'lar rakip incelemesinden türetilmiştir. Gerçek kullanıcı görüşmesi yapılmamıştır.
- **V-05:** NFR-07 ve NFR-10'daki sayısal hedefler başlangıç değerleridir. Ekipçe oluşturulacak, telefon fotoğraflarından oluşan etiketli bir test setinde doğrulanıp ayarlanmalıdır. Kaynaklardaki benchmark'lar farklı veri setleriyle ve masaüstü/sunucu donanımında yapılmıştır. Mobil cihazdaki süreler doğrulanmamıştır.
- **V-06:** "Kopya" ile "benzer" ayrımının kullanıcı için anlamlı olduğu ve rakiplerdeki "Birebir / Benzer" ayrımına benzer bir yapının beklendiği varsayılmıştır.
- **V-07:** iOS'ta uygulamanın sildiği fotoğrafların "Son Silinenler"e gittiği ve sistemin bir onay penceresi gösterdiği varsayılmıştır. Bu oturumda resmî kaynaktan **doğrulanamamıştır** (ilgili Apple geliştirici sayfasının içeriği okunamadı).
- **V-08:** Uygulama ücretsiz veya gelir modelinden bağımsız olarak tasarlanmaktadır.

## 9. Açık sorular

1. **Video:** Video kopyaları ilk sürümde yer alacak mı?
2. **Benzer fotoğraflar (FR-05):** Bu kategori MVP'de mi olacak, yoksa sonraki bir sürümde mi? Bu karar tarama süresini ve cihaz yükünü belirgin biçimde etkiler (bkz. 3.2, gömme yöntemlerinin maliyeti).
3. **Google Play beyanı:** Geniş medya erişimi beyanı reddedilirse Android için alternatif plan nedir? Örneğin kullanıcının seçtiği albüm veya fotoğraflarla sınırlı tarama. Yayın takvimi bu riske göre planlanmalı mıdır?
4. **En iyi kare önceliği (FR-07):** Öneri kriterlerinin sırası ne olmalı? Örneğin favori > çözünürlük > netlik > dosya boyutu > en eski veya en yeni tarih.
5. **Buluttaki orijinaller:** Cihazda yalnızca küçük önizlemesi bulunan, orijinali bulutta duran fotoğraflar (ör. iCloud'da optimize depolama) taranacak mı? Taranacaksa veri ve pil kullanımı kabul edilebilir mi? Bu oturumda araştırılmamıştır.
6. **Minimum işletim sistemi sürümleri:** iOS ve Android için en düşük sürüm ne olacak? Bu karar kısıtlı erişim ve toplu silme davranışlarını etkiler.
7. **Referans cihaz:** NFR-03, NFR-04 ve NFR-05 ölçümleri hangi iOS ve Android cihazlarda yapılacak?
8. **Android silme davranışı:** Silinenler kalıcı silme yerine sistem çöp kutusuna mı gidecek? Kullanıcıdan her işlemde onay yerine "medya yönetimi" özel izni istenecek mi?
9. **iOS silme davranışı:** V-07'deki varsayım (Son Silinenler'e gitme ve sistem onayı) TA tarafından doğrulanmalıdır.
10. **Diller:** Türkçe dışında hangi diller desteklenecek?
11. **Gelir modeli:** Ücretsiz mi, tek seferlik ücret mi, abonelik mi? Rakip incelemesinde pahalı haftalık abonelik ana şikâyetlerden biridir.
12. **Albüm üyeliği:** Silinecek kopya bir albümdeyse ve saklanan fotoğraf o albümde değilse ne yapılmalı? Bu konu Apple'ın "Birleştir" özelliği hakkında kullanıcı kaygısı olarak forumlarda geçiyor, ancak bu oturumda kaynağı açılıp doğrulanmamıştır.

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | HackerFactor: Looks Like It | https://www.hackerfactor.com/blog/index.php?/archives/432-Looks-Like-It.html | aHash ve pHash adımları. Hamming eşik yorumları (0 / yaklaşık 5 / 10 ve üstü). aHash'in gama ve histogram değişikliklerinde zayıflığı, pHash'in küçük düzenlemelere dayanıklılığı. |
| 2 | HackerFactor: Kind of Like That | https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html | dHash algoritması. 150.000 görsellik testte aHash, dHash ve pHash doğruluk ve süre kıyası. dHash ile ön eleme, pHash ile doğrulama önerisi. Eşik 10. |
| 3 | Needle In A Haystack, Fast (arXiv 2206.00282) | https://arxiv.org/abs/2206.00282 | dHash ve SimCLR v2'nin iyi performans gösterdiği, iyi ölçeklendiği ve verimli olduğu. Klasik yöntemlerin her zaman en iyi olmadığı. |
| 4 | imagededup dokümantasyonu | https://idealo.github.io/imagededup/ | Desteklenen yöntemler (CNN, PHash, DHash, WHash, AHash). Benzer kopyalar için CNN önerisi. En hızlı yöntemin dHash olması. |
| 5 | imagededup Benchmarks | https://idealo.github.io/imagededup/user_guide/benchmarks/ | Birebir ve dönüşüme uğramış kopyalarda kesinlik ve duyarlılık değerleri. Özetlemenin CNN'den 10-15 kat hızlı olması. Eşik etkileri. |
| 6 | Duplicate Image Detection (dev.to, Kencho) | https://dev.to/kencho/duplicate-image-detection-how-to-find-and-remove-duplicate-photos-5dod | Dosya özeti, algısal özet ve gömmeden oluşan katmanlı yaklaşım. Her katmanın yakaladığı ve kaçırdığı durumlar. Eşik aralıkları. |
| 7 | PyImageSearch: Blur detection with OpenCV | https://pyimagesearch.com/2015/09/07/blur-detection-with-opencv/ | Laplace varyansıyla bulanıklık skoru ve eşiğin veri setine bağlı olduğu. |
| 8 | CleverFiles: Best apps to delete duplicate photos on iPhone | https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html | Yedi rakip uygulamanın özellikleri, fiyatları, artı ve eksileri. Değerlendirme kriterleri. 10.000'den fazla fotoğrafı 5 dakikanın altında tarama kıyası. |
| 9 | PopSci: How to find and delete duplicate photos on Android or iOS | https://www.popsci.com/diy/find-and-delete-duplicate-photos-android-ios | Files by Google, Samsung My Files, Google Fotoğraflar, Apple Kopyalar albümü ve Remo özellikleri ile sınırlamaları. |
| 10 | Apple Destek 102260: Duplicates albümü bulunamazsa | https://support.apple.com/102260 | iOS 16 ile Kopyalar albümü. İndeksleme için kilitli ve şarjda olma şartı, birkaç güne kadar sürebilmesi. |
| 11 | MacRumors: Remove duplicates photo library iOS | https://www.macrumors.com/how-to/remove-duplicates-photo-library-ios/ | Apple "Birleştir" özelliğinin en yüksek detayı ve en fazla meta veriyi koruması, toplu seçim. |
| 12 | Apple Destek 124460: Silinen fotoğrafları kurtarma | https://support.apple.com/124460 | "Son Silinenler" albümünde 30 gün saklama. |
| 13 | Apple WWDC20 10641: Limited Photos Library | https://developer.apple.com/videos/play/wwdc2020/10641/ | iOS kısıtlı erişim davranışı, seçim değişikliği, paylaşılan bulut öğelerine erişilememesi, seçimi değiştirme düğmesi önerisi. |
| 14 | Android: Grant partial access to photos and videos | https://developer.android.com/about/versions/14/changes/partial-photo-video-access | Android 14 seçili fotoğraf erişimi. Tam, kısmi ve reddedilmiş durumlar. İzni ihtiyaç anında isteme. Kısmi erişimde kütüphanenin boş varsayılmaması. |
| 15 | Android: Access media files from shared storage | https://developer.android.com/training/data-storage/shared/media | READ_MEDIA izinlerinin Play politikasıyla kısıtlı olması. Toplu çöpe taşıma ve silme istekleri. Sistem onay penceresi. Çöpün sistem süresi sonunda kalıcı silinmesi. Medya yönetimi özel izni. |
| 16 | Google Play: Photo and Video Permissions politikası | https://support.google.com/googleplay/android-developer/answer/14115180 | Geniş fotoğraf erişiminin yalnızca çekirdek işlevi bunu gerektiren uygulamalara açık olması. Örnek olarak galeri uygulamaları. Beyan zorunluluğu. |
| 17 | Google Play: Declaration / deadlines yardım sayfası | https://support.google.com/googleplay/android-developer/answer/15800983 | Play Console'da beyan sunma ve Google incelemesi süreci. |

---

**Not:** Dokümanı `outputs/<senaryo-adi>/ba-<senaryo-adi>-v1-run1.md` adıyla kaydetmenizi öneririm. Senaryo adını önermem: `photo-duplicate-finder`. Kural dosyalarını Read ile açıp okudum:
- `/home/user/agent-personas/.claude/rules/documentation-standards.md`
- `/home/user/agent-personas/.claude/rules/research-rules.md`
- `/home/user/agent-personas/.claude/rules/scope-rules.md`

İki sayfa kaynak olarak kullanılmadı: MDPI makalesi 403 hatası verdi, Apple deleteAssets sayfasının içeriği okunamadı.