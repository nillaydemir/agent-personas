# İş Analizi: Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** "Flutter ile yazacağımız ve cep telefonlarında çalışacak bir Photo Library duplicate finder uygulaması hazırlayacağız. Gerekli algoritmaları araştırıp bana bir iş analizi çıkar."

---

## 1. Özet

Uygulama, telefonun fotoğraf kütüphanesini tarar. Birebir aynı ve görsel olarak neredeyse aynı fotoğrafları gruplar, her grupta saklanacak fotoğrafı önerir ve kullanıcının onayıyla fazlalıkları işletim sisteminin geri alınabilir çöp kutusuna taşır. Amaç depolama alanı kazandırmak ve kütüphaneyi sadeleştirmek.

**Ürün büyüklüğü:** Küçük-orta ölçekli, tek kullanıcılı bir mobil uygulama. Sunucu tarafı ya da hesap sistemi beklenmiyor. Kapsam buna göre dar tutuldu: yalnızca fotoğraf kopyaları ele alınıyor. Bulanık fotoğraf, ekran görüntüsü ve video temizliği gibi rakiplerde bulunan ek özellikler kapsam dışında bırakıldı.

**Araştırmadan çıkan ana noktalar:**
- **Yöntemler:** Birebir kopyalar kriptografik özet (hash) ile güvenilir biçimde bulunabilir. Neredeyse aynı fotoğraflar için algısal özetleme (perceptual hashing: aHash, dHash, pHash) ve yapay öğrenme tabanlı gömme vektörleri (embedding) kullanılır. Algısal özetler hızlıdır ama yanlış eşleşme üretebilir. Kırpma, döndürme ve aynalamada zayıftırlar. Yöntem seçimi TA'ya bırakılmıştır. Bölüm 3.2'de iş açısından artıları ve eksileri verildi.
- **Rakipler:** iOS 16 ile gelen yerleşik "Kopyalar" (Duplicates) albümü, Android'de Files by Google'ın "Duplicate files" kartı ve çok sayıda ücretli temizleyici uygulama var. Rakiplerde ortak şikâyet, farklı kişilerin "benzer" diye gruplanması gibi yanlış pozitifler.
- **Platform kısıtları:** Android'de geniş fotoğraf erişim izni Google Play politikasıyla sınırlandırılmış. Başka uygulamaların fotoğraflarını silmek kullanıcı onay diyaloğu gerektiriyor. iOS'ta kullanıcı yalnızca seçili fotoğraflara erişim (limited) verebiliyor.

**Not:** Flutter, paydaşın verdiği bir kısıt olarak kayda alındı. Bu dokümanda başka bir teknoloji kararı verilmemiştir.

---

## 2. Hedef kullanıcılar ve persona'lar

Aşağıdaki persona'lar tasarım amaçlıdır ve kullanıcı araştırmasına dayanmaz (bkz. Varsayımlar V-01).

| Persona | Profil | İhtiyaç / acı noktası | Başarı ölçütü |
|---|---|---|---|
| **P1: Depolaması dolan kullanıcı** | iPhone veya Android kullanıcısı. Binlerce fotoğrafı var, teknik bilgisi orta düzeyde. | "Depolama dolu" uyarısı alıyor. Neyi sileceğini bilmiyor, yanlışlıkla önemli bir fotoğrafı silmekten korkuyor. | Birkaç dakikada güvenle yer açmak ve ne kadar yer kazandığını görmek. |
| **P2: Seri çeken kullanıcı** | Aynı sahneyi art arda birkaç kez çeken, çocuk veya evcil hayvan fotoğrafçısı ebeveyn. | Neredeyse aynı onlarca kare birikiyor ve en iyisini elle seçmek zahmetli. | Her seride en iyi kareyi kolayca seçip kalanları temizlemek. |
| **P3: Gizliliğe duyarlı kullanıcı** | Fotoğraflarının buluta gönderilmesini istemeyen kullanıcı. | Temizleyici uygulamaların fotoğrafları nereye gönderdiğinden emin değil. | Fotoğrafların cihazdan çıkmadığından emin olmak. Sınırlı erişim verse bile uygulamayı kullanabilmek. |

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakip uygulamalar

| Çözüm | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler |
|---|---|---|---|
| **Apple Fotoğraflar: Kopyalar albümü** (iOS 16+, macOS Ventura+) | Albümler > Araçlar altında yalnızca kopya varsa görünür. Setler kronolojik sıralanır, her setin yanında "Birleştir" (Merge) düğmesi var. Toplu seçim ve silme yapılabilir. Birleştirme en yüksek detayı ve en fazla meta veriyi tek fotoğrafta toplar. Tarama sürekli yapılır, kopyalar zamanla görünür. ([MacRumors iOS](https://www.macrumors.com/how-to/remove-duplicates-photo-library-ios/), [MacRumors Mac](https://www.macrumors.com/how-to/find-duplicate-photos-mac)) | Ücretsiz ve yerleşik. Meta veriyi birleştiriyor, yalnızca silmiyor. | Yalnızca Apple cihazlarında. Eşleştirme kriterleri açıklanmıyor. Tarama zamanı kullanıcının kontrolünde değil. Hassasiyet ayarı yok. |
| **Files by Google: Duplicate files** (Android) | Temizle > "Duplicate files" kartı. Her sette orijinal dosya "Original" rozetiyle işaretlenir ve saklanması önerilir. Seçilenler Çöp'e taşınır ve onay diyaloğu gösterilir. ([Google Destek](https://support.google.com/files/answer/9764075?hl=en)) | Ücretsiz ve basit. Orijinal dosyayı açıkça işaretliyor. Silme geri alınabilir (Çöp). | Tespit yöntemi ve dosya türleri belgelenmemiş. Neredeyse aynı fotoğrafları bulup bulmadığı belirtilmiyor. |
| **Gemini Photos** (MacPaw, iPhone) | "Benzer" grupları oluşturur, her grupta "En İyi Sonuç" seçer ve kalanları silinmek üzere önceden işaretler. Seçimde düzenleme, favori durumu ve yüz verisi dikkate alınır. Favoriler hiçbir zaman otomatik seçilmez. Bulanık fotoğraf, not fotoğrafı ve ekran görüntüsü kategorileri var. İşleme cihazda yapılır ve internet gerekmez. Silinenler Son Silinenler'de 30 gün kalır. Videolar taranmaz. ([MacPaw SSS](https://macpaw.com/support/gemini-photos/faq)) Bir sete "gizle" denince o set tekrar işaretlenmez. Ücretlendirme: 3 günlük deneme, ardından abonelik veya tek seferlik ödeme (2018 fiyatları). ([iMore inceleme](https://imore.com/gemini-photos-iphone-clears-out-clutter)) | En iyi kare önerisi. Favori koruması. Cihazda işleme. Yanlış eşleşmeyi tekrar göstermeme. | Farklı kişileri kopya diye gruplayan yanlış pozitifler var. Sonuçların elle kontrolü gerekiyor. ([iMore](https://imore.com/gemini-photos-iphone-clears-out-clutter)) |
| **Remo Duplicate Photos Remover** (iPhone) | "Birebir" ve "Benzer" kategorileri, önizleme, görsel detayları ve elle işaretleme. ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)) | Tamamen ücretsiz ve reklamsız. Basit. | Taraması çok yavaş, arayüzü eski. Bazen birebir kopyaları "benzer" olarak etiketliyor. ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)) |
| **CleanMy®Phone, Cleaner Guru, Hyper Cleaner, Boost Cleaner, Cleanup** (iPhone temizleyicileri) | Birebir ve benzer tespitinin yanında kaydırarak inceleme (swipe), video sıkıştırma, kişi birleştirme, e-posta temizliği gibi ek araçlar sunuyorlar. ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)) | Geniş özellik seti, hızlı tarama. | Pahalı haftalık abonelikler ($3.99 ile $11.99/hafta arası). Yanlış gruplama ya da az kopya yakalama. Çökme ve silme sorunu şikâyetleri. ([CleverFiles](https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html)) |

**Kaynak notu:** CleverFiles karşılaştırmasını, listedeki uygulamalardan birinin (Clever Cleaner) yayıncısı olan şirket yazmıştır. Sıralamalar tarafsız kabul edilmemelidir.

**Rakiplerden çıkan dersler:**
1. Silme işlemleri geri alınabilir olmalı (Son Silinenler veya Çöp) ve kullanıcı onayı gerektirmeli. Bütün güvenilir çözümler böyle çalışıyor.
2. "Birebir" ve "Benzer" sonuçlarını ayrı göstermek, kullanıcının güven düzeyine göre karar vermesini kolaylaştırıyor.
3. Saklanacak fotoğrafı otomatik önermek temel bir beklenti. Favorilerin korunması güven artırıyor.
4. En sık şikâyet yanlış pozitifler. "Bu kopya değil" deyip bir daha göstermeme özelliği bunu hafifletiyor.

### 3.2 Kopya tespit yöntemleri (iş açısından değerlendirme)

Yöntem seçimi TA'ya aittir. Aşağıdaki tablo yalnızca iş etkisini özetler.

| Yöntem | Nasıl çalışır (özet) | Kullanıcıya faydası | Maliyeti / riski |
|---|---|---|---|
| **Kriptografik özet** (ör. SHA-256) | İçerik bayt bayt aynıysa özet de aynıdır. Algısal özetin tersine, en küçük değişiklik özeti tamamen değiştirir. ([Wikipedia](https://en.wikipedia.org/wiki/Perceptual_hashing), [MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | "Birebir kopya" için yanlış pozitif riski pratikte yoktur. Kullanıcı bu sonuçlara gönül rahatlığıyla güvenir. | Yeniden kaydedilmiş, sıkıştırılmış ya da boyutu değişmiş kopyaları yakalamaz. Dosyanın tamamının okunması gerekir. |
| **aHash** (ortalama özet) | 8×8 küçük resimde her piksel ortalama parlaklıkla karşılaştırılır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | Hızlı ve basit. | Üçü arasında en az dayanıklı olarak tanımlanır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) |
| **dHash** (fark özeti) | Gri tonlu 9×8 görüntüde komşu pikseller arasındaki parlaklık artışı veya azalışı kaydedilir (64 bit). ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | En hızlısı. Yeniden boyutlandırma, JPEG sıkıştırma ve küçük parlaklık farklarını tolere eder. Büyük kütüphaneler için başlangıç önerisi. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | Gökyüzü veya boş belge gibi az detaylı görüntülerde yanlış pozitif sık görülür. Kırpma, döndürme ve aynalamada zayıftır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) |
| **pHash** (DCT tabanlı algısal özet) | 32×32 gri ton görüntüye 2B DCT uygulanır, düşük frekanslı 8×8 katsayı medyanla karşılaştırılır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | Parlaklık, kontrast, gama ve sıkıştırma değişikliklerine en dayanıklı olanı. Filtrelenmiş veya yeniden kodlanmış kopyaları yakalar. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | dHash'ten biraz yavaştır. Kırpma ve döndürmede yine zayıftır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) |
| **İki algısal özetin birlikte kullanılması** | dHash ve pHash birlikte hesaplanır. İkisi de eşiğin altındaysa eşleşme sayılır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | En yüksek kesinlik (precision), yani daha az yanlış pozitif. | İşlem süresi artar, bazı gerçek kopyalar kaçabilir. |
| **Gömme vektörü (image embedding) ve kosinüs benzerliği** | Bir model görüntüyü anlamsal bir özellik vektörüne çevirir. Vektörler kosinüs benzerliğiyle karşılaştırılır: −1 ile 1 arası, 1 aynı demek. Görüntüler arası benzerlik anında hesaplanabilir ve mobil uygulamalara gömülmek üzere sunulur. ([Google LiteRT Image Embedder](https://developers.google.com/edge/lite/libraries/task_library/image_embedder)) | Kırpılmış, döndürülmüş veya ekran görüntüsü alınmış kopyaları yakalamada önerilen yaklaşım. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) | Model dosyası uygulama boyutunu büyütür, işlem daha ağırdır. Anlamsal benzerlik "aynı sahne, farklı an" fotoğraflarını da yakalar, bu da yanlış pozitif riski demektir. Mobil cihazda hız ve doğruluk için doğrulanmış kaynak bulunamadı (bkz. Açık sorular S-08). |

**Eşik değerleri ile ilgili bulgular:** Algısal özetler 64 bitlik değerler olarak Hamming mesafesiyle karşılaştırılır. Standart bir eşik yoktur. 0 ile 5 arası mesafe genellikle aynı görüntüyü gösterir. Yaklaşık 10, başlangıç eşiği olarak kullanılıp kendi veri setiyle ayarlanmalıdır. ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)) Algısal özetler ayrıca birbirine hiç benzemeyen iki görüntüye benzer özet üretebilir ("Perceptual hashes are messy"). ([Wikipedia](https://en.wikipedia.org/wiki/Perceptual_hashing))

**İş sonucu:** Yanlış pozitif riskinin kullanıcıya şeffaf gösterilmesi gerekir. Bunun için iki seviyeli sonuç (Birebir ve Benzer), ayarlanabilir hassasiyet ve zorunlu kullanıcı onayı öngörüldü (FR-06, FR-07, FR-12).

### 3.3 Platform kısıtları (gereksinimleri etkileyen)

| Platform | Bulgu | Kaynak |
|---|---|---|
| Android | `READ_MEDIA_IMAGES` izni Google Play politikasıyla kısıtlıdır. Yalnızca çekirdek işlevi fotoğraflara geniş erişim gerektiren ve sistem seçicisinin (Photo Picker) yetmediği uygulamalar isteyebilir. Play Console'da beyan zorunludur. Uyum 28 Mayıs 2025'ten beri zorunludur ve uymayan uygulamalar kaldırılabilir. Kullanıcı geniş erişimi reddederse uygulamanın makul bir alternatif deneyim sunması beklenir. | [Google Play politikası](https://support.google.com/googleplay/android-developer/answer/14115180), [Android Developers](https://developer.android.com/training/data-storage/shared/media) |
| Android | Kapsamlı depolama (scoped storage) altında bir uygulama başka uygulamaların medya dosyalarını doğrudan silemez. Android 11 ve üzerinde dosya grupları için `createTrashRequest()` (sistem tanımlı süre sonunda kalıcı silinen çöp) veya `createDeleteRequest()` (anında kalıcı silme) kullanılır. Her ikisinde de kullanıcı bir onay diyaloğu görür. | [Android Developers](https://developer.android.com/training/data-storage/shared/media) |
| Android | Uygulama, medya deposunun değişip değişmediğini sürüm ve nesil (generation) bilgisiyle tespit edebilir. Bu, artımlı tarama için gereklidir. | [Android Developers](https://developer.android.com/training/data-storage/shared/media) |
| iOS | Kullanıcı sınırlı erişim (`limited`) verebilir. Uygulama, kullanıcının seçimi güncelleyebileceği sistem seçicisini açabilir. | [Apple PHAuthorizationStatus.limited](https://developer.apple.com/tutorials/data/documentation/photos/phauthorizationstatus/limited.json) |
| iOS | Fotoğraf silme işlemi bir fotoğraf kütüphanesi değişiklik bloğu içinde istenir. | [Apple PHAssetChangeRequest.deleteAssets](https://developer.apple.com/tutorials/data/documentation/photos/phassetchangerequest/deleteassets(_:).json) |
| iOS | Üçüncü taraf bir uygulamanın sildiği fotoğraflar Fotoğraflar uygulamasında "Son Silinenler"e gider ve 30 gün kalır (Gemini Photos örneği). | [MacPaw SSS](https://macpaw.com/support/gemini-photos/faq) |

---

## 4. Fonksiyonel gereksinimler

| ID | Gereksinim | Kabul ölçütü (özet) |
|---|---|---|
| **FR-01** | Uygulama, fotoğraf erişim izni istemeden önce neden erişim gerektiğini ve fotoğrafların cihazdan çıkmadığını anlatan bir bilgilendirme ekranı gösterir. | İzin diyaloğu bilgilendirme ekranından önce hiç çıkmaz. |
| **FR-02** | Uygulama tam, sınırlı ve reddedilmiş izin durumlarını ayrı ayrı ele alır. Sınırlı izinde yalnızca erişilebilen fotoğrafları tarar, bunu kullanıcıya açıkça söyler ve seçimi genişletme yolu sunar. Reddedilmiş izinde ayarlara yönlendirir. | Üç durumda da uygulama çökmez ve kullanıcıya bir sonraki adım gösterilir. |
| **FR-03** | Kullanıcı kütüphane taramasını elle başlatır. Tarama sırasında işlenen ve toplam fotoğraf sayısı ile yüzde ilerleme gösterilir. Kullanıcı taramayı iptal edebilir. | İlerleme en az 1 saniyede bir güncellenir. İptal 2 saniye içinde etkili olur. |
| **FR-04** | Uygulama, içeriği bayt düzeyinde aynı olan fotoğrafları "Birebir kopya" olarak gruplar. | Aynı dosyanın iki kopyası her zaman aynı gruba düşer. |
| **FR-05** | Uygulama, yeniden boyutlandırılmış, yeniden sıkıştırılmış veya küçük parlaklık ya da renk farkı olan görsel olarak neredeyse aynı fotoğrafları "Benzer" olarak gruplar. | Bölüm 5'teki NFR-04 doğruluk hedefleri sağlanır. |
| **FR-06** | Sonuçlar "Birebir kopya" ve "Benzer" sekmelerinde ayrı listelenir. Her sekmede grup sayısı ve kazanılabilir toplam alan (MB veya GB) gösterilir. | Bir fotoğraf aynı anda iki sekmede gösterilmez. |
| **FR-07** | Kullanıcı "Benzer" tespiti için en az 3 hassasiyet seviyesi arasında seçim yapabilir (ör. Sıkı, Normal, Gevşek). Varsayılan "Normal"dir. | Seviye değişince "Benzer" sonuçları yeni tarama gerekmeden en geç 5 saniyede güncellenir (bkz. S-09). |
| **FR-08** | Her grup küçük resimleriyle gösterilir. Her fotoğraf için çözünürlük, dosya boyutu ve çekim tarihi görülebilir. | Bu alanlar grubun her öğesinde görünür. |
| **FR-09** | Uygulama her grupta saklanacak bir fotoğraf önerir ve diğerlerini silinmek üzere önceden işaretler. Varsayılan öncelik sırası şöyledir: en yüksek çözünürlük, en büyük dosya boyutu, en eski çekim tarihi (kural seti bkz. S-05). Favori olarak işaretli fotoğraflar hiçbir zaman otomatik olarak silinmeye işaretlenmez. | Her grupta tam olarak 1 öneri bulunur. Favoriler hiçbir koşulda önceden işaretli gelmez. |
| **FR-10** | Kullanıcı herhangi bir fotoğrafın işaretini değiştirebilir ve grup içi fotoğrafları tam ekranda yan yana veya ardışık karşılaştırabilir. | Bir grubun tüm fotoğrafları silinmeye işaretlenirse kullanıcı uyarılır. |
| **FR-11** | Kullanıcı tek işlemle tüm gruplardaki önerileri kabul edip toplu silme başlatabilir. | Silmeden önce silinecek fotoğraf sayısı ve kazanılacak alan özetlenir. |
| **FR-12** | Silme yalnızca kullanıcı onayıyla ve işletim sisteminin geri alınabilir alanına taşınarak yapılır (iOS: Son Silinenler, Android: sistem çöpü). Uygulama kendi akışında kalıcı silme sunmaz. | Silinen her fotoğraf işletim sisteminin galerisinden geri yüklenebilir. |
| **FR-13** | Silme sonrasında silinen fotoğraf sayısı, kazanılan alan ve fotoğrafların nereden geri alınabileceği gösterilir. | Özet ekranı her başarılı silmeden sonra görünür. |
| **FR-14** | Kullanıcı bir grubu "Kopya değil" olarak işaretleyebilir. Bu grup sonraki taramalarda gösterilmez. Bu işaretler ayarlardan sıfırlanabilir. | İşaretli grup yeniden taramada listelenmez. |
| **FR-15** | İlk taramadan sonraki taramalar artımlıdır: yalnızca yeni eklenen veya değişen fotoğraflar işlenir. Önceki sonuçlar ve FR-14 işaretleri korunur. Kütüphanede artık olmayan fotoğraflar sonuçlardan çıkarılır. | Artımlı taramada önceden işlenmiş fotoğraflar yeniden analiz edilmez. |
| **FR-16** | Uygulama kapanır veya tarama kesilirse, bir sonraki açılışta tarama kaldığı yerden devam ettirilebilir. | Devam edilen tarama baştan başlamaz. |

---

## 5. Fonksiyonel olmayan gereksinimler

NFR-02, NFR-03 ve NFR-04'teki sayısal hedefler araştırmaya değil iş hedefine dayanan önerilerdir. Paydaşla teyit edilmelidir (bkz. V-04, S-07).

| ID | Gereksinim | Ölçüm yöntemi |
|---|---|---|
| **NFR-01 Gizlilik** | Fotoğraflar ve fotoğraflardan türetilen veriler (özet, vektör, küçük resim) cihaz dışına gönderilmez. Tarama ve sonuçların gösterimi internet bağlantısı olmadan çalışır. | Uçak modunda uçtan uca tarama ve silme testi. Ağ trafiği incelemesinde görüntü verisi içeren istek sayısı 0 olmalı. |
| **NFR-02 İlk tarama süresi** | 10.000 fotoğraflık bir kütüphanenin ilk taraması referans orta segment cihazda (bkz. S-07) uygulama ön plandayken en fazla 10 dakika sürer. | Referans cihazda 10.000 fotoğraflık test kütüphanesiyle süre ölçümü. |
| **NFR-03 Artımlı tarama süresi** | 100 yeni fotoğraf eklendikten sonraki artımlı tarama en fazla 30 saniye sürer. | Referans cihazda süre ölçümü. |
| **NFR-04 Doğruluk** | (a) "Birebir kopya" sekmesinde yanlış pozitif oranı %0'dır. (b) "Benzer" sekmesinde varsayılan hassasiyette, etiketli test setinde kesinlik (precision) en az %95, duyarlılık (recall) en az %85 olur. | En az 500 etiketli fotoğraf çiftinden oluşan test seti (birebir, sıkıştırılmış, boyutu değişmiş, ardışık çekim ve farklı sahne çiftleri). |
| **NFR-05 Ölçek ve kararlılık** | 50.000 fotoğraflık kütüphanede tarama çökme veya bellek nedeniyle sonlanma olmadan tamamlanır. | Referans cihazda 3 ardışık tam tarama ve 0 çökme. |
| **NFR-06 Arayüz yanıt verebilirliği** | Tarama sürerken kullanıcı etkileşimlerine (kaydırma, iptal) 200 ms içinde yanıt verilir. | Referans cihazda etkileşim gecikmesi ölçümü. |
| **NFR-07 Veri güvenliği** | Hiçbir fotoğraf kullanıcının açık onayı olmadan silinmez ya da değiştirilmez. Uygulama fotoğrafların içeriğini değiştirmez. | Kod incelemesi ve onaysız silme senaryosu testleri. |
| **NFR-08 Platform desteği** | Uygulama iOS ve Android telefonlarda çalışır. Desteklenen en düşük işletim sistemi sürümleri S-03 ile netleşecektir. Toplu ve onaylı çöpe taşıma Android'de Android 11 ve üzerinde sağlanır. | Hedef sürüm matrisi üzerinde test. |
| **NFR-09 Mağaza uyumu** | Uygulama Google Play Fotoğraf ve Video İzinleri politikasına uyar. Geniş erişim izni için gerekçe beyanı hazırlanır ve kullanıcı geniş erişimi reddederse alternatif bir akış sunulur (FR-02). | Play Console beyanının onaylanması. Mağaza incelemesinde ret alınmaması. |
| **NFR-10 Erişilebilirlik** | Tüm etkileşimli öğelerin ekran okuyucu etiketi vardır. Arayüz sistem yazı boyutu ayarına uyar. | iOS VoiceOver ve Android TalkBack ile tüm ana akışların tamamlanabilmesi. |
| **NFR-11 Dil** | Arayüz en az Türkçe sunulur. Ek diller için bkz. S-10. | Tüm ekranlarda çevrilmemiş metin bulunmaması. |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Güvenli izin verme** (FR-01, FR-02, NFR-01)
P3 olarak, izin vermeden önce fotoğraflarımın cihazdan çıkmayacağını bilmek istiyorum. Böylece uygulamaya güvenebilirim.
- *Verilen* uygulama ilk kez açıldığında, *o zaman* izin diyaloğundan önce gizlilik açıklaması gösterilir.
- *Verilen* kullanıcı sınırlı erişim verdiğinde, *o zaman* tarama yalnızca seçili fotoğraflarda yapılır ve "Yalnızca X fotoğrafa erişim var, seçimi genişlet" mesajı gösterilir.
- *Verilen* kullanıcı izni reddettiğinde, *o zaman* ayarlara giden bir düğme gösterilir ve uygulama çökmez.

**US-02: Kütüphaneyi tarama** (FR-03, FR-16, NFR-02, NFR-05, NFR-06)
P1 olarak, tüm fotoğraflarımı tarayıp ilerlemeyi görmek istiyorum. Böylece ne kadar bekleyeceğimi bilirim.
- *Verilen* tarama başladığında, *o zaman* işlenen ve toplam sayı ile yüzde en az saniyede bir güncellenir.
- *Verilen* kullanıcı "İptal"e bastığında, *o zaman* tarama 2 saniye içinde durur.
- *Verilen* tarama sırasında uygulama kapandığında, *o zaman* bir sonraki açılışta "Taramaya devam et" seçeneği sunulur.

**US-03: Birebir kopyaları temizleme** (FR-04, FR-06, FR-08, FR-09, FR-11, FR-12, NFR-04)
P1 olarak, birebir aynı fotoğrafları tek dokunuşla temizlemek istiyorum. Böylece hızlıca yer açarım.
- *Verilen* tarama bittiğinde, *o zaman* "Birebir kopya" sekmesinde grup sayısı ve kazanılabilir alan gösterilir.
- *Verilen* kullanıcı "Önerileri uygula"ya bastığında, *o zaman* silinecek sayı ve alanı içeren bir onay özeti gösterilir.
- *Verilen* kullanıcı onayladığında, *o zaman* her grupta tam olarak bir fotoğraf kalır ve silinenler işletim sisteminin geri alınabilir alanına taşınır.

**US-04: Benzer çekimlerden en iyisini seçme** (FR-05, FR-08, FR-09, FR-10)
P2 olarak, ardışık çektiğim benzer fotoğrafları yan yana görüp en iyisini seçmek istiyorum.
- *Verilen* bir "Benzer" grubu açıldığında, *o zaman* önerilen fotoğraf "Sakla" olarak işaretli gelir ve diğerleri silinmeye işaretli gelir.
- *Verilen* kullanıcı bir fotoğrafa dokunduğunda, *o zaman* tam ekran karşılaştırma açılır ve işaret değiştirilebilir.
- *Verilen* gruptaki tüm fotoğraflar silinmeye işaretlendiğinde, *o zaman* uyarı gösterilir.

**US-05: Favorileri koruma** (FR-09, NFR-07)
P1 olarak, favori olarak işaretlediğim fotoğrafların yanlışlıkla silinmemesini istiyorum.
- *Verilen* bir grupta favori fotoğraf varsa, *o zaman* bu fotoğraf hiçbir zaman önceden silinmeye işaretli gelmez.

**US-06: Hassasiyeti ayarlama** (FR-07, NFR-04)
P2 olarak, "Benzer" sonuçlarının ne kadar sıkı olacağını ayarlamak istiyorum. Böylece yanlış eşleşmeleri azaltırım.
- *Verilen* kullanıcı hassasiyeti "Sıkı" yaptığında, *o zaman* "Benzer" grup sayısı "Normal"e göre eşit ya da daha az olur ve sonuçlar 5 saniye içinde güncellenir.

**US-07: Yanlış eşleşmeyi gizleme** (FR-14, FR-15)
P2 olarak, aslında farklı olan fotoğrafları "Kopya değil" diye işaretlemek istiyorum. Böylece aynı grubu tekrar görmem.
- *Verilen* kullanıcı bir grubu "Kopya değil" olarak işaretlediğinde, *o zaman* grup listeden kalkar ve sonraki taramalarda tekrar görünmez.
- *Verilen* kullanıcı ayarlardan sıfırlama yaptığında, *o zaman* gizlenen gruplar bir sonraki taramada tekrar değerlendirilir.

**US-08: Geri alınabilir silme ve özet** (FR-12, FR-13, NFR-07)
P1 olarak, yanlışlıkla sildiğim bir fotoğrafı geri alabilmek ve ne kadar yer kazandığımı görmek istiyorum.
- *Verilen* silme tamamlandığında, *o zaman* silinen sayı, kazanılan alan ve "Fotoğraflar > Son Silinenler" veya "Galeri > Çöp" yönlendirmesi gösterilir.
- *Verilen* kullanıcı sistemin silme onay diyaloğunu reddettiğinde, *o zaman* hiçbir fotoğraf silinmez ve kullanıcıya bilgi verilir.

**US-09: Sonraki taramaların hızlı olması** (FR-15, NFR-03)
P1 olarak, birkaç hafta sonra tekrar taradığımda yalnızca yeni fotoğrafların işlenmesini istiyorum.
- *Verilen* önceki taramadan sonra 100 fotoğraf eklendiğinde, *o zaman* tarama en fazla 30 saniyede tamamlanır ve önceki "Kopya değil" işaretleri korunur.

---

## 7. Kapsam dışı maddeler

Rakiplerde bulunan ama görevde istenmeyen özellikler ürün büyüklüğüne uygun olmadığı için kapsam dışında bırakıldı:
- Video kopyalarının tespiti (bkz. S-01).
- Bulanık fotoğraf, ekran görüntüsü ve not fotoğrafı gibi "dağınıklık" kategorileri (Gemini Photos ve CleanMy®Phone'da var).
- Kaydırarak inceleme (swipe), video veya fotoğraf sıkıştırma, Live Photo dönüştürme.
- Kişi birleştirme, e-posta temizliği, gizli klasör gibi depolama dışı araçlar.
- Apple tarzı "Birleştir" (Merge): kopyaların meta verilerini tek fotoğrafta toplamak. Ürün yalnızca saklama ve silme önerir (bkz. S-04).
- Bulutta (iCloud Fotoğrafları, Google Fotoğraflar) bulunan ama cihazda olmayan fotoğrafların ayrıca taranması (bkz. S-02).
- Cihazlar arası veya hesaplar arası kopya karşılaştırması. Kullanıcı hesabı ve sunucu altyapısı.
- Otomatik veya zamanlanmış arka plan silme.
- Kırpılmış, döndürülmüş veya aynalanmış kopyaların garanti altında yakalanması (bkz. S-06).
- Tablet için özel arayüz.

---

## 8. Varsayımlar

- **V-01:** Persona'lar kullanıcı araştırmasına değil, rakip incelemelerinde görülen şikâyet ve özelliklere dayanır.
- **V-02:** Hedef kullanıcı tek bir telefonun kendi fotoğraf kütüphanesini temizlemek istiyor. Paylaşılan albümler kapsamda değil.
- **V-03:** Flutter, paydaşın verdiği bir teknoloji kısıtıdır. Platform erişiminin (izin, silme) nasıl sağlanacağı TA kararıdır.
- **V-04:** NFR-02, NFR-03 ve NFR-04'teki sayısal hedefler iş hedefi önerisidir. Mobil cihazda algısal özet ya da gömme vektörü performansı için doğrulanmış bir ölçüm kaynağı bulunamadı.
- **V-05:** iOS'ta üçüncü taraf uygulamanın silme isteği için sistemin kendi onay diyaloğunu gösterdiği varsayıldı. Bu bilgi okunan Apple sayfasında doğrulanamadı. Son Silinenler'e taşınma ve 30 gün saklama yalnızca MacPaw SSS'sinden doğrulandı.
- **V-06:** Android 10 ve altında toplu çöpe taşıma API'si yok. Bu sürümlerin desteklenip desteklenmeyeceği S-03'e bağlı.
- **V-07:** Uygulama gerekçesi ("çekirdek işlev tüm kütüphanenin taranması") Google Play beyanında kabul görecek. Politika sayfası bu kategoriyi açıkça örnek vermiyor, bu yüzden ret riski var.
- **V-08:** HEIC, JPEG, PNG gibi işletim sistemi galerisinin gösterebildiği tüm fotoğraf biçimleri taranacak. RAW ve Live Photo davranışı S-11'de soruldu.
- **V-09:** Kütüphane boyutu için tipik üst sınır 50.000 fotoğraf kabul edildi (NFR-05).

---

## 9. Açık sorular

- **S-01:** Videolar ilk sürümde taranmalı mı?
- **S-02:** iCloud veya Google Fotoğraflar'da olup cihazda yalnızca küçük resmi bulunan fotoğraflar nasıl ele alınmalı? Taranmalı mı, indirme izni istenmeli mi, atlanmalı mı?
- **S-03:** Desteklenecek en düşük iOS ve Android sürümleri neler? (Android 11 altı toplu silmeyi etkiler.)
- **S-04:** Silinen kopyanın albüm üyelikleri ve meta verileri (konum, açıklama) kaybolabilir. Apple'ın "Birleştir" özelliği bunları korur. Bu kayıp kabul edilebilir mi, yoksa meta veri aktarımı istenir mi?
- **S-05:** Saklanacak fotoğraf önerisinde hangi kriterler hangi öncelikle kullanılmalı (çözünürlük, dosya boyutu, tarih, düzenlenmiş olma, keskinlik, yüz ifadesi)?
- **S-06:** Kırpılmış, döndürülmüş veya ekran görüntüsü alınmış kopyaların yakalanması iş açısından gerekli mi? Bu, yöntem seçimini ve uygulama boyutunu doğrudan etkiler (bkz. 3.2).
- **S-07:** Performans hedefleri için referans cihaz(lar) hangileri? 10 dakika ve 30 saniye hedefleri kabul edilebilir mi?
- **S-08:** Gömme vektörü tabanlı yöntem için mobil cihazda hız ve doğruluk verisi bulunamadı. Prototip ölçümü yapılmalı mı?
- **S-09:** Hassasiyet değişince yeniden tarama yapılmadan sonuç güncellenmesi zorunlu mu?
- **S-10:** Türkçe dışında hangi diller desteklenecek?
- **S-11:** Live Photo, seri çekim (burst) ve RAW+JPEG çiftleri kopya mı sayılmalı, ayrı mı ele alınmalı?
- **S-12:** Gelir modeli ne olacak (ücretsiz, tek seferlik ödeme, abonelik, reklam)? Rakiplerin çoğu abonelik kullanıyor, Remo tamamen ücretsiz.
- **S-13:** iOS 16 ve üzerinde yerleşik "Kopyalar" albümü ücretsiz olarak var. Ürünün iOS'taki farklılaştırıcı değeri ne olacak (hassasiyet ayarı, şeffaf eşleşme sebebi, Android ile tutarlı deneyim vb.)?
- **S-14:** Google Play beyanı reddedilirse (V-07) alternatif akış ne olmalı? Örneğin kullanıcının sistem seçicisiyle seçtiği fotoğraflar arasında kopya arama.

---

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | Perceptual hashing, Wikipedia | https://en.wikipedia.org/wiki/Perceptual_hashing | Algısal özetin tanımı, kriptografik özetten farkı, yanlış eşleşme ve manipülasyon riskleri. |
| 2 | pHash vs dHash: Perceptual Image Hashing, MojoAuth | https://mojoauth.com/security-guides/phash-vs-dhash | aHash, dHash ve pHash adımları, hız ve dayanıklılık karşılaştırması, Hamming eşik önerileri, kullanım önerileri (birlikte kullanım, embedding, SHA-256). |
| 3 | Image Embedder, Google AI Edge (LiteRT) | https://developers.google.com/edge/lite/libraries/task_library/image_embedder | Görüntü gömme vektörü kavramı, anında benzerlik hesaplama, kosinüs benzerliği aralığı, mobil uygulamaya gömme. |
| 4 | How to remove duplicates in Photos on iOS, MacRumors | https://www.macrumors.com/how-to/remove-duplicates-photo-library-ios/ | iOS 16 Kopyalar albümü: konum, benzer fotoğrafları gruplama, birleştirme davranışı, toplu işlem, sürekli tarama. |
| 5 | How to Find, Merge, and Delete Duplicate Photos in macOS, MacRumors | https://www.macrumors.com/how-to/find-duplicate-photos-mac | macOS Ventura Kopyalar albümü, birleştirmenin en yüksek detay ve meta veriyi koruması, albümün yeniden belirmesi. |
| 6 | Gemini Photos FAQ, MacPaw | https://macpaw.com/support/gemini-photos/faq | Benzer ve Dağınıklık kategorileri, en iyi fotoğraf seçimi kriterleri, favori koruması, cihazda işleme, Son Silinenler'de 30 gün, video desteği olmaması. |
| 7 | Gemini Photos Review, iMore | https://imore.com/gemini-photos-iphone-clears-out-clutter | Yanlış pozitifler (farklı kişilerin gruplanması), gizleme özelliği, fiyatlandırma, artılar ve eksiler. |
| 8 | Best Apps to Delete Duplicate Photos on iPhone, CleverFiles | https://www.cleverfiles.com/howto/best-apps-to-delete-duplicate-photos-on-iphone.html | Clever Cleaner, Cleaner Guru, CleanMy®Phone, Hyper Cleaner, Boost Cleaner, Cleanup ve Remo'nun özellikleri, fiyatları, güçlü ve zayıf yönleri. Yayıncının çıkar çatışması var. |
| 9 | Delete duplicate files, Files by Google Yardım | https://support.google.com/files/answer/9764075?hl=en | Android'de Duplicate files kartı, "Original" rozeti, Çöp'e taşıma ve onay diyaloğu. |
| 10 | Access media files from shared storage, Android Developers | https://developer.android.com/training/data-storage/shared/media | READ_MEDIA_IMAGES kısıtı, kapsamlı depolamada silme, Android 11+ createTrashRequest ve createDeleteRequest ile onay diyaloğu, medya deposu sürüm ve nesil takibi. |
| 11 | Photo and Video Permissions policy, Google Play Console Yardım | https://support.google.com/googleplay/android-developer/answer/14115180 | Geniş fotoğraf erişimi için çekirdek işlev şartı, beyan zorunluluğu, 28 Mayıs 2025 uyum tarihi, reddedilen erişimde alternatif deneyim beklentisi. |
| 12 | PHAuthorizationStatus.limited, Apple Developer Documentation | https://developer.apple.com/tutorials/data/documentation/photos/phauthorizationstatus/limited.json | iOS sınırlı fotoğraf erişimi ve seçimin sistem seçicisiyle güncellenebilmesi. |
| 13 | PHAssetChangeRequest.deleteAssets(_:), Apple Developer Documentation | https://developer.apple.com/tutorials/data/documentation/photos/phassetchangerequest/deleteassets(_:).json | iOS'ta silme isteğinin bir kütüphane değişiklik bloğu içinde yapılması. |

**Notlar:**
- Kaynak 12 ve 13 Apple belgelerinin JSON sürümlerinden okundu. Bu sürümler onay diyaloğu bilgisini içermiyordu (V-05).
- Towards Data Science'taki "image duplicate finder" rehberi (HTTP 403) ve Apple'ın "Delivering an Enhanced Privacy Experience" sayfası (içerik gelmedi) açılamadı. Bu sayfalar kaynak gösterilmedi ve onlardan bilgi kullanılmadı.