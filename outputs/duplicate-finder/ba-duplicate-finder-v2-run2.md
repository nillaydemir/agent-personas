# Photo Library Duplicate Finder: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v2
- **Girdi:** "Flutter ile yazacağımız ve cep telefonlarında çalışacak bir Photo Library duplicate finder uygulaması hazırlayacağız. Gerekli algoritmaları araştırıp bana bir iş analizi çıkar."

---

## 1. Özet

Ürün, telefonun fotoğraf kütüphanesini cihaz üzerinde tarayan bir mobil uygulamadır. Birebir aynı (exact) ve birbirine çok benzeyen (near-duplicate) fotoğrafları gruplar. Kullanıcı grupları inceler, hangi kopyanın kalacağına karar verir ve gereksiz kopyaları silerek yer açar. Görevde verilen kısıtlar şunlardır: geliştirme Flutter ile yapılacak ve uygulama cep telefonlarında çalışacak. Bu doküman iki mobil platformu (iOS ve Android) hedef kabul eder (bkz. VS-01).

Araştırmanın başlıca bulguları:

- **Yerleşik çözümler sınırlı.**
  - iOS Fotoğraflar uygulamasının "Duplicates" (Kopyalar) koleksiyonu kopyaları birleştirmeye (merge) izin verir. Ancak Apple'ın MacRumors'a yaptığı açıklamaya göre tespit için telefonun kilitli ve şarjda olması gerekir ve işlem birkaç gün sürebilir.
  - Files by Google yalnızca "Duplicate files" (kopya dosyalar) kartı sunar.
  - Google Photos'un "Photo stacks" özelliği benzer fotoğrafları gruplar, ancak Google'ın yardım sayfasına göre bu gruplama depolama alanını değiştirmez.
- **Üçüncü taraf uygulamalar** benzerlik seviyesi ayarı ve "her grupta bir kopyayı bırakıp gerisini işaretleme" (automark) sunar. Bir kullanıcı yorumu ise bu otomatik işaretlemenin istenmeyen silmelere yol açabildiğini gösteriyor. Güven ve geri alınabilirlik bu yüzden önemli bir farklılaşma alanıdır.
- **Platform politikaları ürün için en büyük iş riskidir.**
  - Google Play, tüm fotoğraflara geniş erişim izinlerini (`READ_MEDIA_IMAGES`) yalnızca "temel işlevi fotoğraflara geniş erişim etrafında dönen" uygulamalara verir ve Play Console'da beyan ister.
  - iOS ve Android kullanıcıları yalnızca seçili fotoğraflara erişim de verebilir. Uygulama bu durumda da çalışabilmelidir.
- **Yöntemler:** Birebir kopyalar için içerik özeti (kriptografik hash), benzer kopyalar için algısal hash (aHash, dHash, pHash) veya derin öğrenme vektörleri (embedding) kullanılabilir. Yayımlanmış karşılaştırmalarda dHash ve pHash, aHash'ten daha az yanlış eşleşme üretmiş, dHash ise en hızlı yöntem olmuştur. Yanlış pozitif riski her yöntemde vardır. Bu nedenle benzer gruplarda silmeden önce kullanıcı onayı zorunlu tutulmuştur. Yöntem seçimi TA'ya bırakılmıştır.

İlk sürüm (MVP) şunları kapsar: izin yönetimi, cihaz üzerinde tarama, birebir ve benzer kopyaların gruplanması, inceleme ekranı, önerilen "kalacak kopya" ve onaylı, geri alınabilir silme.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Başarı ölçütü |
|---|---|---|---|
| **P1: Yer sıkıntısı çeken genel kullanıcı** | Telefonu dolu, teknik bilgisi orta düzeyde, mesajlaşma uygulamalarından gelen ve tekrar kaydedilen görselleri biriktirmiş. | Hızlıca ve güvenle yer açmak. | Kaç fotoğrafın silindiğini ve ne kadar yer açıldığını görmek, yanlışlıkla önemli bir fotoğrafı kaybetmemek. |
| **P2: Yoğun çekim yapan kullanıcı** | Seri çekim yapan, aynı kareyi küçük farklarla çok kez çeken, düzenlenmiş ve düzenlenmemiş sürümleri birlikte saklayan kullanıcı. | Benzer kareler arasından en iyisini seçip gerisini silmek. | Benzer grupları hızlı incelemek, toplu işlem yapabilmek. |
| **P3: Gizlilik hassasiyeti olan kullanıcı** | Fotoğraflarının bir sunucuya gitmesinden çekinen, uygulamaya yalnızca seçili fotoğraflara erişim verebilen kullanıcı. | Fotoğraflarının cihazdan çıkmadığından emin olmak, sınırlı izinle de uygulamayı kullanabilmek. | Ağ bağlantısı olmadan çalışma, açık gizlilik beyanı. |

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Yerleşik ve doğrudan rakip çözümler

| Çözüm | Platform | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler / sınırlar |
|---|---|---|---|---|
| **Apple Fotoğraflar – Duplicates koleksiyonu** (yerleşik) | iOS | Apple'ın kullanım kılavuzuna göre Fotoğraflar uygulaması kopya fotoğraf ve videoları "Duplicates" koleksiyonunda toplar. Kullanıcı bir seti "Merge" ile birleştirir ([Apple iPhone Kullanım Kılavuzu](https://support.apple.com/en-gb/guide/iphone/iph1978d9c23/)). | Ek uygulama gerekmez, sisteme entegredir. Toplu birleştirme yapılabilir. | Apple'ın MacRumors'a yaptığı açıklamaya göre tespit "telefonun kilitli ve şarja bağlı olmasını gerektirir" ve süreç "birkaç gün sürebilir". Kopya yoksa koleksiyon görünmez ([MacRumors, 2022-09-28](https://www.macrumors.com/2022/09/28/apple-explains-why-photos-duplicates-no-show/)). Hangi kopyanın tutulduğu ve fazla kopyaların nereye gittiği kılavuzun okunan bölümünde yazmıyor. MacRumors sayfasındaki bir okur yorumu fazla kopyaların "Recently Deleted" albümüne taşındığını aktarıyor. Bu bilgi Apple belgesinden doğrulanamadı. |
| **Files by Google – Duplicate files** (yerleşik veya Google'a ait) | Android | Menü > Clean > "Duplicate files" kartı. Orijinal dosya "Original" rozetiyle gösterilir. Seçilen dosyalar kalıcı silinmez, çöp kutusuna (Bin) taşınır ([Files by Google Yardım](https://support.google.com/files/answer/9764075?hl=en-GB)). | Orijinali açıkça işaretler ve silme geri alınabilir. | Yardım sayfası yalnızca "kopya dosyaları" anlatır. Benzer (birebir olmayan) fotoğraflardan söz etmez. |
| **Google Photos – Photo stacks** (yerleşik veya Google'a ait) | Android / iOS (Google Photos uygulaması) | "Kısa bir zaman aralığında birlikte çekilmiş, aynı konunun neredeyse özdeş fotoğraflarını" otomatik gruplar ve bir "Top pick" (en iyi kare) seçer. Kullanıcı bir fotoğrafı bırakıp gerisini silebilir ([Google Photos Yardım](https://support.google.com/photos/answer/14169846)). | Benzer kareleri zaman yakınlığıyla yakalar ve en iyi kareyi önerir. | Google'ın yardım sayfasına göre gruplama "depolama alanını değiştirmez". Özellik yalnızca yedeklenmiş fotoğraflarda ve yalnızca Google Photos uygulamasında çalışır. |
| **Duplicate Photos Fixer** (App Store id966475595) | iOS | App Store sayfasına göre üç tarama türü vardır: Normal, Aggressive ve Custom. Benzerlik seviyesi ayarlanabilir, piksel, zaman farkı ve GPS ölçütleri seçilebilir. "Magic wand" ile her grupta biri hariç tüm kopyalar işaretlenir. İşaretlenenler "kalıcı olarak" silinir. Fiyatı 7,99 USD'dir ([App Store](https://apps.apple.com/us/app/-/id966475595)). | Ayarlanabilir benzerlik ve toplu otomatik işaretleme sunar. | Kullanıcı yorumları karışık. Bir kullanıcı uygulamanın kopyaları bulamadığını ve "bir tanesi yerine tüm benzer fotoğrafları sildiğini" yazmış (tek kullanıcı yorumu). |
| **Remo Duplicate Photos Remover** (App Store id1066797785) | iOS | App Store sayfasına göre birebir aynı ve "neredeyse benzer" görselleri siler. Her grupta "en iyi bir kopyayı otomatik olarak korur" ve silmeden önce önizleme sunar. Ücretsizdir. 14 bin değerlendirmeyle puanı 4,6'dır. Geliştiricinin beyanına göre veri toplamaz, bu beyan Apple tarafından doğrulanmamıştır ([App Store](https://apps.apple.com/app/id1066797785)). | Hem birebir hem benzer tespit yapar ve en iyi kopyayı önerir. | 2020 tarihli bir kullanıcı yorumu silme işlemi için abonelik gerektiğini söylüyor (tek kullanıcı yorumu, güncelliği doğrulanamadı). |

### 3.2 Rakip incelemesinden çıkan iş bulguları

1. **Silmenin geri alınabilir olması sektörde beklenen davranıştır.** Files by Google silinenleri çöp kutusuna taşır. Android, toplu silmede sistem çöp kutusuna taşıma isteği (trash request) sunar. Kalıcı silme yapan bir üçüncü taraf uygulama, kullanıcı yorumlarında güven sorunu yaşamıştır. → FR-09 ve NFR-06.
2. **"Orijinal / kalacak kopya" önerisi standart bir beklentidir** (Files by Google'daki "Original" rozeti, Google Photos'taki "Top pick", Remo'daki "best copy"). → FR-08.
3. **Benzerlik hassasiyeti ayarı** üçüncü taraf uygulamalarda yaygındır. → FR-11.
4. **Yerleşik çözümlerde boşluklar var.** iOS'ta tespit gecikmeli ve koşullu çalışıyor. Google Photos stacks yer açmıyor ve yerel fotoğraflarda çalışmıyor. Ürün bu boşluklara konumlanabilir: anında, cihaz üzerinde ve yer açmaya odaklı tarama.

### 3.3 Alan standartları ve düzenlemeler

| Alan | Bulgu | Ürüne etkisi |
|---|---|---|
| **Kişisel veri (KVKK, Türkiye)** | 6698 sayılı Kanun md. 4(2)(ç): kişisel veriler "İşlendikleri amaçla bağlantılı, sınırlı ve ölçülü olma" ilkesine uygun işlenir. Md. 4(2)(d): veriler amaç için gerekli süre kadar saklanır. Md. 28(1)(a), gerçek kişilerin kendi kişisel veya ev içi faaliyetleri kapsamındaki işlemelerini, "üçüncü kişilere verilmemek ve veri güvenliğine ilişkin yükümlülüklere uyulmak kaydıyla" kanun kapsamı dışında tutar ([Lexpera – 6698 sayılı KVKK](https://www.lexpera.com.tr/mevzuat/kanunlar/kisisel-verilerin-korunmasi-kanunu-6698)). | Analiz cihaz üzerinde yapılır ve hiçbir veri cihazdan çıkmaz (NFR-01). Analiz verisi kullanıcı tarafından silinebilir (FR-18). Hukuki niteleme açık sorudur (AS-05). |
| **Kişisel veri (GDPR, AB)** | Madde 5(1)(c) veri minimizasyonu: veriler "amaç için yeterli, ilgili ve gerekli olanla sınırlı" olmalıdır. Madde 5(1)(e) saklama sınırlaması: veriler gerekli olandan uzun süre saklanmaz ([GDPR Art. 5](https://gdpr-info.eu/art-5-gdpr/)). | NFR-01 ve FR-18 ile aynı yöndedir. |
| **Erişilebilirlik (WCAG 2.2)** | 1.4.3: metin kontrastı en az 4,5:1, büyük metinde 3:1. 2.5.8: dokunma hedefi en az 24×24 CSS pikseli ([W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/)). | NFR-08. WCAG bir web standardıdır. Mobil uygulamaya referans olarak uygulanması bir varsayımdır (VS-06). |

### 3.4 Platform ve mağaza politikaları

| Platform | Bulgu | Ürüne iş etkisi |
|---|---|---|
| **Google Play – Fotoğraf ve Video İzinleri politikası** | Android 13+ hedefleyen uygulamalar `READ_MEDIA_IMAGES`/`READ_MEDIA_VIDEO` izinlerini yalnızca sistem seçicisi (photo picker) temel işlevi karşılamadığında isteyebilir. Politikadan alıntı: "Apps whose core functionality revolves around broad access to Photos and Videos may use the permissions above." Bu uygulamalar Play Console'da beyan (declaration) vermek zorundadır. Sayfaya göre uzatma alan geliştiriciler için tam uyum tarihi 28 Mayıs 2025'tir, uyumsuz uygulamalar kaldırılabilir. Örnek olarak galeri uygulamaları anılır. Kopya bulucu uygulamalar sayfada adıyla geçmez. Geniş erişim vermeyen kullanıcılara "makul çaba" ile hizmet verilmelidir ([Google Play Console Yardım](https://support.google.com/googleplay/android-developer/answer/14115180)). | **Yüksek risk:** Beyan reddedilirse uygulamanın Android sürümü temel işlevini kaybeder. Mağaza metni ve beyan, "tüm kütüphanenin taranmasının temel işlev olduğunu" açıkça anlatmalıdır (NFR-11, AS-03). Sınırlı erişimde de çalışma zorunludur (FR-02). |
| **Android 14+ seçili fotoğraf erişimi** | Kullanıcı tam erişim yerine yalnızca belirli fotoğraf ve videolara erişim verebilir (`READ_MEDIA_VISUAL_USER_SELECTED`). Android belgesi şunları önerir: uygulama izin durumunu kalıcı olarak saklamamalı, "tam erişim varsaymamalı", kullanıcının seçimini yönetmesine izin vermeli ve izni tekrar istemeden önce kullanıcının bastığı bir arayüz öğesi göstermeli ([Android Developers – Partial photo access](https://developer.android.com/about/versions/14/changes/partial-photo-video-access)). | FR-01 ve FR-02. Uygulama her açılışta izin durumunu yeniden kontrol eder. |
| **Android toplu silme** | Android 11+ cihazlarda başka uygulamaların oluşturduğu medya `createTrashRequest()` veya `createDeleteRequest()` ile toplu olarak çöp kutusuna taşınabilir ya da silinebilir. Bu çağrılardan sonra kullanıcı, işlemi onaylayan bir sistem iletişim kutusu görür. Belgeye göre çöp kutusundaki öğeler "sistem tarafından belirlenen bir süre sonra" kalıcı olarak silinir ([Android Developers – MediaStore](https://developer.android.com/training/data-storage/shared/media)). | FR-09: silme işlemi, uygulamanın kendi onayına ek olarak bir sistem onayı da içerir. Kullanıcı deneyimi iki onay adımını hesaba katmalıdır. Silinen öğeler çöp kutusundan geri alınabilir. |
| **Android photo picker** | Sistem seçicisi, uygulamanın tüm kütüphaneye erişmeden kullanıcının seçtiği öğelere ulaşmasını sağlar ve depolama izni gerektirmez ([Android Developers – Photo picker](https://developer.android.com/training/data-storage/shared/photopicker)). | Kalıcı erişim için bir uygulamanın aynı anda en fazla 5.000 medya iznine sahip olabileceği belirtiliyor. Bu nedenle seçici tek başına tüm kütüphanenin taranması için uygun görünmüyor. Bu çıkarım TA ile doğrulanmalıdır (AS-03). |
| **iOS sınırlı kütüphane erişimi** | iOS 14'ten beri kullanıcı "Select Photos" seçeneğiyle uygulamaya yalnızca seçtiği öğeleri gösterebilir. Bu modda kullanıcı albümleri okunamaz ve paylaşılan bulut albümlerine erişilemez. Apple, kullanıcıya seçimini uygulama içinden değiştirme olanağı verilmesini önerir ([WWDC20 – Handle the Limited Photos Library](https://developer.apple.com/videos/play/wwdc2020/10641/)). | FR-02. Sınırlı modda albüm bazlı filtre (FR-16) çalışmayabilir. |
| **App Store İnceleme Kuralları** | 5.1.1(i): uygulama içinde ve mağazada gizlilik politikası bağlantısı zorunludur. 5.1.1(ii): izin amaç metinleri (purpose string) kullanımı "açık ve eksiksiz" anlatmalıdır. 5.1.1(iii): uygulama yalnızca temel işleviyle ilgili verilere erişim istemelidir, mümkünse Fotoğraflar'a tam erişim yerine seçici kullanılmalıdır. 2.3.1(a): yanıltıcı pazarlama yasaktır. 3.1.2: abonelik isteyen uygulama, abonelik öncesinde kullanıcının ne alacağını açıkça anlatmalıdır ([App Store Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)). | NFR-11. Mağaza metni gerçek dışı yer açma vaadi içermemelidir. Monetizasyon modeli açık sorudur (AS-04). |
| **Google Play Veri Güvenliği (Data safety)** | Formu her uygulama doldurmak zorundadır. "Toplama" (collect), verinin cihazdan dışarı iletilmesi anlamına gelir. Yalnızca cihaz üzerinde işlenen veri toplanmış sayılmaz ([Play Console Yardım – Data safety](https://support.google.com/googleplay/android-developer/answer/10787469)). | NFR-01 sağlanırsa beyan "veri toplanmıyor" olabilir. Üçüncü taraf analiz veya reklam SDK'sı eklenirse bu durum değişir (AS-04). |

### 3.5 Yöntemler (algoritmalar): iş açısından değerlendirme

Aşağıdaki tablo seçenekleri iş açısından karşılaştırır. Hangi yöntemin veya kombinasyonun kullanılacağına TA karar verir.

| Yöntem | Ne yakalar | Kaynaktaki bulgu (koşuluyla) | İş açısından artı | İş açısından eksi |
|---|---|---|---|---|
| **İçerik özeti / kriptografik hash** (birebir kopya) | Bayt düzeyinde aynı dosyalar | Kriptografik hash'ler "çığ etkisi" (avalanche effect) gösterir: girdideki küçük bir değişiklik çıktıyı tamamen değiştirir. Bu yüzden yakın kopyaları eşleştiremez ([Wikipedia – Perceptual hashing](https://en.wikipedia.org/wiki/Perceptual_hashing)). | "Birebir aynı" etiketi kullanıcıya yüksek güvenle sunulabilir ve toplu silmeye uygundur. | Yeniden sıkıştırılmış, boyutu değiştirilmiş veya mesajlaşma uygulamasından tekrar kaydedilmiş kopyaları kaçırır. |
| **aHash** (ortalama hash) | Benzer görseller | Coğrafi uydu görüntüsü benchmark'ında (10.000 görüntü; birebir kopya, 90°/180°/270° döndürme, yatay/dikey çevirme varyantları) F1 0,9429; 664 yanlış pozitif, 218 yanlış negatif ([Adimoolam vd., arXiv 2304.02296](https://arxiv.org/html/2304.02296v2)). BSDS500 üzerinde yaklaşık %0,5 FPR için seçilen eşikte, Kaggle meme veri setinde FPR %6,4 ([Ref. arXiv 2206.00282](https://arxiv.org/html/2206.00282)). | Basit ve hızlıdır. | İncelenen iki çalışmada da diğer hash'lere göre daha çok yanlış eşleşme üretmiştir. Bu, kullanıcıya gereksiz inceleme yükü ve yanlış silme riski demektir. |
| **pHash** (DCT tabanlı algısal hash) | Benzer görseller | Aynı coğrafi benchmark'ta, 64 bit hash ve Hamming eşiği 0 ile yazarların pHash tabanlı yöntemi kesinlik 1,0000, duyarlılık 0,9997 ve F1 0,9999 elde etmiştir (0 yanlış pozitif, 2 yanlış negatif) ([arXiv 2304.02296](https://arxiv.org/html/2304.02296v2)). Meme veri setinde, yukarıdaki eşik koşulunda FPR %0,90 ([arXiv 2206.00282](https://arxiv.org/html/2206.00282)). Yüz görüntüsü veri setleri çalışmasında pHash, kırpmaya dayanıklı hash'ten daha fazla kopya bulmuştur. Yazarlar yanlış pozitif grupların "manuel incelemede kolayca farklı yüzler olarak ayırt edilebilecek görüntüler" içerebildiğini bildirmiş ve ikinci bir doğrulama adımı kullanmıştır ([Rusak vd., arXiv 2401.14088](https://arxiv.org/html/2401.14088)). | Yeniden sıkıştırılmış ve boyutu değiştirilmiş kopyalarda iyi sonuç verir. | Yanlış pozitif riski vardır. Bulgular kişisel fotoğraf kütüphanesinde değil, uydu, meme ve yüz veri setlerinde elde edilmiştir. Gerçek kullanımda doğruluk kendi test setimizle ölçülmelidir (NFR-05). |
| **dHash** (fark hash'i) | Benzer görseller | Makalenin bulgusuna göre dHash güçlü döndürme, eğme (shear) ve kırpma dışında "diğer tüm algoritmalara eşit ya da onlardan iyi" performans göstermiştir. "En hesaplama verimli algoritma" olarak raporlanmıştır. Meme veri setinde, yukarıdaki eşik koşulunda FPR %0,05 ([arXiv 2206.00282](https://arxiv.org/html/2206.00282)). | Hızlıdır. Büyük kütüphanelerde tarama süresini ve pil tüketimini düşürmeye aday bir yöntemdir. | Kırpılmış veya döndürülmüş kopyaları kaçırabilir. |
| **Derin öğrenme vektörleri** (ör. SimCLR v2) | Anlamsal benzerlik (farklı açı, kırpma) | Makaleye göre SimCLR v2 ResNet'ler "mükemmel performans" göstermiştir. Ancak SimCLRv2 ResNet50 2x, dHash'ten "yaklaşık 10 kat yavaştır" ([arXiv 2206.00282](https://arxiv.org/html/2206.00282)). | Daha geniş "benzer fotoğraf" kapsamı (P2 persona'sı için seri ve açı farkı olan kareler) sağlayabilir. | Telefonda tarama süresi ve pil maliyeti daha yüksektir, uygulama boyutu büyüyebilir. Benzerlik ölçütü genişledikçe kullanıcının "kopya değil" diyeceği gruplar artabilir. |
| **Zaman ve meta veri yakınlığı** (seri çekim gruplama) | Kısa sürede çekilmiş benzer kareler | Google Photos, "kısa bir zaman aralığında birlikte çekilmiş" neredeyse özdeş fotoğrafları gruplamak için bu ölçütü kullanır ([Google Photos Yardım](https://support.google.com/photos/answer/14169846)). Duplicate Photos Fixer zaman farkı ve GPS ölçütleri sunar ([App Store](https://apps.apple.com/us/app/-/id966475595)). | Görüntü analizine ek olarak ucuz bir ön eleme veya gruplama ölçütü olabilir. | Tek başına "kopya" kanıtı değildir. Aynı anda çekilmiş farklı kareleri aynı gruba koyabilir. |

**Ortak risk:** Algısal hash'ler hatasız değildir. Wikipedia'da aktarılan bir örnekte gerçek bir kadın portresi ile suluboya bir kelebek resmi birbirine benzer hash'ler üretmiştir. Ayrıca küçük değişikliklerle farklı görüntüler arasında çakışma (collision) üretilebildiği bildirilmiştir ([Wikipedia – Perceptual hashing](https://en.wikipedia.org/wiki/Perceptual_hashing)).

**İş sonucu:** Ürün iki güven seviyesi tanımlar: "Birebir aynı" ve "Benzer".
- "Benzer" gruplarda hiçbir öğe kullanıcı incelemesi olmadan silinmez (FR-10).
- Toplu "öneriyi uygula" işlemi yalnızca birebir gruplarda sunulur (FR-14).

---

## 4. Fonksiyonel gereksinimler

**M (Must) önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S ve C maddeleri ertelenebilir. Bu sürümde yapılmayacak maddeler bölüm 7'dedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Uygulama, fotoğraf erişim iznini sistem iletişim kutusundan **önce** bir açıklama ekranıyla ister. İzni ilk açılışta değil, kullanıcı "Taramayı başlat" düğmesine bastığında ister. Bu ekran tam erişimin neden gerektiğini ve fotoğrafların cihazdan çıkmadığını anlatır. İzin reddedilirse ayarlara yönlendiren bir ekran gösterilir. | M | Android 14 partial access, WWDC20 10641, App Review 5.1.1(ii) |
| FR-02 | Kullanıcı yalnızca seçili fotoğraflara erişim verdiyse (iOS sınırlı erişim veya Android seçili fotoğraf erişimi) uygulama yalnızca erişilebilir fotoğrafları tarar. Sonuç ekranında taramanın kısmi olduğunu belirtir (ör. "Kütüphanenizin yalnızca seçtiğiniz N fotoğrafı tarandı") ve seçimi genişletme veya tam erişim verme seçeneği sunar. İzin durumu her uygulama açılışında ve ön plana dönüşte yeniden kontrol edilir. | M | Google Play politikası ("makul çaba"), Android 14 partial access, WWDC20 10641 |
| FR-03 | Uygulama, erişilebilir tüm fotoğrafları (yalnızca görseller) cihaz üzerinde tarar. Tarama sırasında ilerleme (taranan/toplam), tahmini kalan süre ve "İptal" seçeneği gösterilir. | M | Görev, persona P1 |
| FR-04 | Uygulama, içeriği birebir aynı olan fotoğrafları tespit eder ve bunları "Birebir aynı" etiketli gruplarda gösterir. | M | Bölüm 3.5 (kriptografik hash) |
| FR-05 | Uygulama, birebir aynı olmasa da görsel olarak neredeyse özdeş fotoğrafları (yeniden boyutlandırılmış, yeniden sıkıştırılmış veya biçimi değiştirilmiş kopyalar) tespit eder ve bunları "Benzer" etiketli gruplarda gösterir. | M | Bölüm 3.5 (algısal hash), rakipler |
| FR-06 | Sonuç ekranı grup sayısını, toplam kopya sayısını ve öneriler uygulanırsa açılacak tahmini alanı (MB/GB) "Birebir aynı" ve "Benzer" kategorileri için ayrı ayrı gösterir. | M | Persona P1, rakipler |
| FR-07 | Grup inceleme ekranı gruptaki her fotoğrafı küçük resim olarak gösterir. Her fotoğraf tam ekran açılabilir. Her fotoğraf için çekim tarihi, çözünürlük ve dosya boyutu gösterilir. | M | Remo ve Duplicate Photos Fixer önizleme özellikleri |
| FR-08 | Uygulama her grupta bir fotoğrafı "Kalacak" olarak önerir ve geri kalanları silinmek üzere işaretler. Öneri kuralı belgelenir ve kullanıcıya gösterilir (ör. "En yüksek çözünürlüklü olan"). Kullanıcı öneriyi tek dokunuşla değiştirebilir. Bir grupta tüm fotoğrafların silinmek üzere işaretlenmesi engellenmez, ancak bu durumda ayrıca uyarı gösterilir. | M | Files by Google "Original" rozeti, Google Photos "Top pick", Remo "best copy", AS-02 |
| FR-09 | Silme yalnızca kullanıcının açık onayıyla yapılır. Onay ekranı silinecek fotoğraf sayısını ve açılacak alanı gösterir. Silinen fotoğraflar, platform izin verdiği ölçüde sistemin geri alınabilir çöp kutusuna veya "Son Silinenler" albümüne gider. Silme sonrası özet ekranında silinen öğe sayısı, açılan alan ve geri alma yolu (sistem çöp kutusu) gösterilir. | M | Files by Google (Bin), Android `createTrashRequest`, Duplicate Photos Fixer kullanıcı yorumu |
| FR-10 | "Benzer" gruplarda silme işareti yalnızca öneri niteliğindedir. Kullanıcı bir "Benzer" grubu açıp incelemeden o grubun öğeleri silinmez. Uygulama hiçbir koşulda kullanıcı onayı olmadan otomatik silme yapmaz. | M | arXiv 2401.14088 (yanlış pozitifler), Wikipedia (çakışmalar) |
| FR-11 | Kullanıcı "Benzer" tespiti için en az 3 hassasiyet seviyesi (ör. Sıkı, Normal, Geniş) arasından seçim yapabilir. Varsayılan seviye "Normal"dir. Değişiklik sonuçlara yeniden tarama gerektirmeden yansır. | S | Duplicate Photos Fixer (iOS ve Mac sayfaları) |
| FR-12 | Sonraki taramalarda yalnızca son taramadan sonra eklenen veya değişen fotoğraflar analiz edilir. Silinen fotoğraflar sonuçlardan çıkarılır. | S | Android MediaStore değişiklik takibi, WWDC20 (değişiklik bildirimi), NFR-02 |
| FR-13 | Tarama kesintiye uğrarsa (uygulama kapanması, arka plana alınma) kullanıcı bir sonraki açılışta taramaya kaldığı yerden devam edebilir. | S | NFR-02 |
| FR-14 | Kullanıcı "Birebir aynı" grupların tamamına öneriyi tek işlemle uygulayabilir. Bu işlem de FR-09'daki onay ekranından geçer. | S | Apple Duplicates "Select All", Duplicate Photos Fixer "Magic wand" |
| FR-15 | Kullanıcı bir grubu "Kopya değil" olarak işaretleyebilir. Bu grup sonraki taramalarda tekrar gösterilmez. Kullanıcı ayarlardan bu işaretleri sıfırlayabilir. | S | Bölüm 3.5 (yanlış pozitif riski) |
| FR-16 | Kullanıcı taramayı tarih aralığına veya albüme göre sınırlandırabilir. Sınırlı erişim modunda albüm filtresi gösterilmez. | C | Duplicate Photos Fixer (zaman aralığı filtresi), WWDC20 (sınırlı modda albümler okunamaz) |
| FR-17 | Kısa zaman aralığında çekilmiş benzer kareler "Seri çekimler" adlı ayrı bir kategoride gösterilir. | C | Google Photos Photo stacks |
| FR-18 | Kullanıcı ayarlardan uygulamanın cihazda sakladığı analiz verilerini (tarama sonuçları, "Kopya değil" işaretleri) tek işlemle silebilir. Bu işlem fotoğrafları etkilemez. | S | KVKK md. 4(2)(d), GDPR Art. 5(1)(e) |

---

## 5. Fonksiyonel olmayan gereksinimler

Aşağıdaki sayısal hedefler kaynaktan alınmamıştır. Bunlar iş hedefi önerileridir ve VS-03 ile VS-04'te varsayım olarak belirtilmiştir.

| ID | Kategori | Gereksinim | Ölçüm yöntemi |
|---|---|---|---|
| NFR-01 | Gizlilik | Fotoğraflar, küçük resimler, hash veya vektör değerleri ve fotoğraf meta verileri cihazdan dışarı gönderilmez. Tarama ve gruplama ağ bağlantısı olmadan (uçak modunda) eksiksiz çalışır. | Uçak modunda uçtan uca tarama testi. Tarama sırasında ağ trafiği izlenir, fotoğraf verisi içeren istek sayısı 0 olmalıdır. |
| NFR-02 | Performans | Referans orta segment cihazda 10.000 fotoğraflık kütüphanenin ilk tam taraması ≤ 10 dakikada tamamlanır. İlk sonuçlar (en az bir grup) tarama başladıktan sonra ≤ 60 saniyede görünür. Artımlı taramada (FR-12) 100 yeni fotoğraf ≤ 30 saniyede işlenir. | Her platformda bir referans cihazda zaman ölçümü (referans cihaz: AS-06). |
| NFR-03 | Yanıt verebilirlik | Tarama sırasında arayüz kullanılabilir kalır. "İptal" ve gezinme dokunuşlarına ≤ 500 ms içinde tepki verilir. | Tarama sırasında manuel veya otomatik arayüz testi. |
| NFR-04 | Doğruluk (birebir) | "Birebir aynı" etiketli gruplarda yanlış pozitif oranı %0'dır: bu gruptaki her öğe içerik olarak aynıdır. | Bilinen kopyalar içeren etiketli test seti üzerinde doğrulama. |
| NFR-05 | Doğruluk (benzer) | Varsayılan hassasiyette, ekipçe hazırlanan ve kişisel fotoğraf kütüphanesini temsil eden en az 1.000 görsellik etiketli test setinde "Benzer" tespiti için grup düzeyinde kesinlik (precision) ≥ %95, duyarlılık (recall) ≥ %85 olmalıdır. Test seti şunları içerir: yeniden sıkıştırma, boyut değişikliği, mesajlaşma uygulamasından tekrar kaydetme ve seri çekim. | Etiketli test setiyle ölçüm. Sonuçlar her sürümde raporlanır. |
| NFR-06 | Veri güvenliği | Kullanıcının onay ekranında görmediği hiçbir fotoğraf silinmez. Uygulamanın çökmesi veya kesinti fotoğraf kaybına yol açmaz. Silme işlemi, platformun sunduğu geri alınabilir yöntemle (çöp kutusu veya Son Silinenler) yapılır. | Silme senaryoları için test: onaylanan liste ile silinen liste birebir aynı olmalıdır. Silme sırasında uygulama zorla kapatılarak kesinti testi yapılır. |
| NFR-07 | Kararlılık | Referans cihazda 50.000 fotoğraflık kütüphane taranırken bellek yetersizliği nedeniyle çökme yaşanmaz. | 50.000 fotoğraflık test kütüphanesiyle 3 ardışık tarama yapılır, çökme sayısı 0 olmalıdır. |
| NFR-08 | Erişilebilirlik | Metin kontrastı en az 4,5:1 (büyük metinde 3:1) olmalıdır. Dokunma hedefleri en az 24×24 birim olmalıdır (WCAG 2.2 2.5.8 referansı). Tüm düğme ve fotoğraf öğeleri platform ekran okuyucusunda anlamlı etiketle okunur (ör. "Fotoğraf, 12 Mart 2025, 4032×3024, Kalacak"). | Kontrast ölçüm aracı ve ekran okuyucu (VoiceOver ve TalkBack) ile manuel test. |
| NFR-09 | Platform kapsamı | iOS ve Android sürümleri aynı M ve S önceliğindeki fonksiyonları sunar. Platforma özgü farklar (ör. silme onayının sistem iletişim kutusu) yalnızca platform kısıtından kaynaklanabilir. | Her iki platformda aynı kabul testleri çalıştırılır. |
| NFR-10 | Dil | Arayüz en az Türkçe olarak sunulur. Tüm metinler çevrilebilir yapıdadır (ek diller: AS-07). | Metin envanteri incelemesi. |
| NFR-11 | Mağaza uyumu | Uygulama içinde ve mağaza sayfalarında gizlilik politikası bağlantısı bulunur. Google Play'de fotoğraf izinleri beyanı ve Data safety formu, App Store'da izin amaç metinleri (purpose string) bulunur ve gerçek kullanımı anlatır. Mağaza metni ölçülmemiş yer açma vaadi içermez. | Yayın öncesi kontrol listesi: App Review 5.1.1, 2.3.1(a) ve Google Play politikası maddeleri. |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

Kabul kriterleri "Verilen / Eğer / O zaman" (Given/When/Then) biçimindedir.

### US-01: İzin verme
**Bir kullanıcı olarak**, uygulamanın fotoğraflarıma neden eriştiğini önceden anlamak istiyorum, **böylece** güvenerek izin verebilirim.
İlgili: FR-01, NFR-01, NFR-11

- **AC-1:** Verilen: uygulama ilk kez açıldı. Eğer: kullanıcı "Taramayı başlat"a basarsa. O zaman: sistem izin iletişim kutusundan önce, fotoğrafların cihazdan çıkmadığını belirten açıklama ekranı gösterilir.
- **AC-2:** Verilen: kullanıcı izni reddetti. Eğer: "Taramayı başlat"a tekrar basarsa. O zaman: ayarlara yönlendiren bir ekran gösterilir ve uygulama çökmez.
- **AC-3:** Verilen: uygulama ilk kez açıldı. Eğer: kullanıcı hiçbir düğmeye basmazsa. O zaman: sistem izin iletişim kutusu gösterilmez.

### US-02: Sınırlı erişimle kullanım
**Gizlilik hassasiyeti olan bir kullanıcı olarak**, yalnızca seçtiğim fotoğraflara erişim verip yine de kopya bulmak istiyorum.
İlgili: FR-02, FR-16

- **AC-1:** Verilen: kullanıcı yalnızca 200 fotoğrafa erişim verdi. Eğer: tarama tamamlanırsa. O zaman: yalnızca bu 200 fotoğraf taranır ve sonuç ekranında "Yalnızca seçtiğiniz 200 fotoğraf tarandı" bilgisi ve "Seçimi genişlet" seçeneği gösterilir.
- **AC-2:** Verilen: kullanıcı ayarlardan erişimi tamdan sınırlıya değiştirdi. Eğer: uygulamaya geri dönerse. O zaman: izin durumu yeniden okunur ve erişilemeyen fotoğraflar sonuçlardan kaldırılır.

### US-03: Kütüphaneyi tarama
**Yer sıkıntısı çeken bir kullanıcı olarak**, kütüphanemi tarayıp ne kadar yer açabileceğimi görmek istiyorum.
İlgili: FR-03, FR-06, NFR-02, NFR-03, NFR-07

- **AC-1:** Verilen: tam erişim var. Eğer: tarama başlarsa. O zaman: taranan/toplam sayısı ve tahmini kalan süre gösterilir ve "İptal" düğmesi ≤ 500 ms içinde tepki verir.
- **AC-2:** Verilen: tarama tamamlandı. Eğer: sonuç ekranı açılırsa. O zaman: "Birebir aynı" ve "Benzer" kategorileri için grup sayısı, kopya sayısı ve tahmini açılacak alan ayrı ayrı gösterilir.
- **AC-3:** Verilen: kütüphanede kopya yok. Eğer: tarama biterse. O zaman: "Kopya bulunamadı" mesajı gösterilir.

### US-04: Birebir kopyaları silme
**Bir kullanıcı olarak**, birebir aynı kopyaları önerilen orijinali koruyarak silmek istiyorum.
İlgili: FR-04, FR-07, FR-08, FR-09, NFR-04, NFR-06

- **AC-1:** Verilen: 3 öğeli bir "Birebir aynı" grup. Eğer: kullanıcı grubu açarsa. O zaman: bir öğe "Kalacak", iki öğe "Silinecek" olarak işaretli görünür ve öneri kuralı ekranda yazar.
- **AC-2:** Verilen: kullanıcı "Kalacak" önerisini başka bir öğeye taşıdı. Eğer: "Sil"e basarsa. O zaman: onay ekranı 2 öğe ve açılacak alanı gösterir. Onaydan sonra yalnızca bu 2 öğe silinir.
- **AC-3:** Verilen: silme tamamlandı. Eğer: özet ekranı açılırsa. O zaman: silinen öğe sayısı, açılan alan ve öğelerin sistem çöp kutusundan veya Son Silinenler'den geri alınabileceği bilgisi gösterilir.
- **AC-4:** Verilen: kullanıcı onay ekranında veya sistem onay iletişim kutusunda "Vazgeç" seçti. O zaman: hiçbir öğe silinmez.

### US-05: Benzer fotoğrafları inceleme
**Yoğun çekim yapan bir kullanıcı olarak**, birbirine çok benzeyen fotoğraflardan en iyisini seçip gerisini silmek istiyorum.
İlgili: FR-05, FR-07, FR-08, FR-10, NFR-05

- **AC-1:** Verilen: aynı fotoğrafın orijinali ile mesajlaşma uygulamasından kaydedilmiş küçültülmüş bir kopyası. Eğer: tarama biterse. O zaman: iki fotoğraf aynı "Benzer" grupta gösterilir.
- **AC-2:** Verilen: kullanıcının henüz açmadığı "Benzer" gruplar var. Eğer: kullanıcı ana ekrandan toplu silme yapmaya çalışırsa. O zaman: açılmamış "Benzer" grupların öğeleri silme listesine eklenmez.
- **AC-3:** Verilen: bir "Benzer" grup açık. Eğer: kullanıcı bir fotoğrafa dokunursa. O zaman: fotoğraf tam ekran açılır, çekim tarihi, çözünürlük ve boyut bilgisi gösterilir.

### US-06: Toplu işlem
**Bir kullanıcı olarak**, yüzlerce birebir kopya grubunu tek tek açmadan temizlemek istiyorum.
İlgili: FR-14, FR-09

- **AC-1:** Verilen: 120 "Birebir aynı" grup. Eğer: kullanıcı "Tüm birebir gruplara öneriyi uygula"ya basarsa. O zaman: onay ekranı toplam silinecek öğe sayısını ve alanı gösterir. "Benzer" gruplar bu işleme dahil edilmez.

### US-07: Hassasiyet ayarı
**Bir kullanıcı olarak**, benzerlik hassasiyetini değiştirerek daha az veya daha çok grup görmek istiyorum.
İlgili: FR-11

- **AC-1:** Verilen: tarama tamamlandı, hassasiyet "Normal". Eğer: kullanıcı "Sıkı"yı seçerse. O zaman: "Benzer" grup sayısı azalır veya aynı kalır ve yeniden tarama yapılmaz.

### US-08: Yeni fotoğrafları tarama
**Düzenli kullanan biri olarak**, bir sonraki taramada yalnızca yeni fotoğrafların işlenmesini istiyorum.
İlgili: FR-12, FR-13, NFR-02

- **AC-1:** Verilen: önceki tam tarama tamamlandı ve 100 yeni fotoğraf eklendi. Eğer: kullanıcı yeniden tarama başlatırsa. O zaman: yalnızca yeni fotoğraflar işlenir ve süre NFR-02'deki sınırı aşmaz.
- **AC-2:** Verilen: tarama %40'ta iken uygulama kapatıldı. Eğer: uygulama yeniden açılırsa. O zaman: "Taramaya devam et" seçeneği sunulur ve tarama baştan başlamaz.

### US-09: "Kopya değil" işareti
**Bir kullanıcı olarak**, yanlışlıkla gruplanmış fotoğrafların bir daha karşıma çıkmamasını istiyorum.
İlgili: FR-15

- **AC-1:** Verilen: kullanıcı bir grubu "Kopya değil" olarak işaretledi. Eğer: yeniden tarama yapılırsa. O zaman: bu grup gösterilmez.
- **AC-2:** Verilen: ayarlarda "İşaretleri sıfırla" seçildi. Eğer: yeniden tarama yapılırsa. O zaman: grup tekrar gösterilir.

### US-10: Gizlilik ve veri silme
**Gizlilik hassasiyeti olan bir kullanıcı olarak**, fotoğraflarımın cihazdan çıkmadığından emin olmak ve uygulamanın sakladığı analiz verisini silebilmek istiyorum.
İlgili: NFR-01, FR-18

- **AC-1:** Verilen: cihaz uçak modunda. Eğer: kullanıcı tam tarama yaparsa. O zaman: tarama ve gruplama eksiksiz tamamlanır.
- **AC-2:** Verilen: kullanıcı ayarlardan "Analiz verilerini sil"i seçti. O zaman: tarama sonuçları ve işaretler silinir, kütüphanedeki fotoğraflar etkilenmez.

---

## 7. Kapsam dışı maddeler

Aşağıdaki maddeler bu sürümde yapılmayacaktır (MoSCoW: W):

1. **Video kopyalarının tespiti.** Görev "Photo" kütüphanesinden söz ediyor. Videolar ayrı bir analiz gerektirir.
2. **Yalnızca bulutta bulunan fotoğrafların taranması** (iCloud'da optimize edilmiş, Google Photos'ta yalnızca yedekte olan öğeler). AS-01'e bağlıdır.
3. **Apple'ın "Merge" davranışı gibi meta veri birleştirme** (favori, açıklama, albüm üyeliklerinin kalan kopyaya aktarılması).
4. **Yapay zekâ ile kalite puanlama** (bulanıklık, kapalı göz, en iyi kare seçimi). FR-08'deki öneri kuralı basit ve belgelenmiş ölçütlere dayanır.
5. **Ekran görüntüsü, bulanık fotoğraf ve büyük dosya temizliği gibi genel "telefon temizleyici" özellikleri.**
6. **Kişiler, dosyalar ve müzik gibi fotoğraf dışı içerik.**
7. **Hesap, oturum açma, bulut yedekleme ve cihazlar arası eşitleme.**
8. **Tablet, web ve masaüstü sürümleri.**
9. **Otomatik veya zamanlanmış arka plan taraması ve bildirimler.**

---

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| VS-01 | "Cep telefonları" iOS ve Android telefonları anlamına gelir. Her iki platform da ilk sürümde hedeflenir. |
| VS-02 | Ürün genel tüketiciye yöneliktir. Kurumsal veya çocuklara yönelik bir kullanım hedeflenmez. Bu nedenle çocukların kişisel verilerine ilişkin özel düzenlemeler bu analizde incelenmemiştir. |
| VS-03 | NFR-02, NFR-03 ve NFR-07'deki süre ve kütüphane boyutu hedefleri kaynaktan alınmamıştır. Ürün beklentisine göre konmuş iş hedefleridir ve referans cihaz belirlendikten sonra gözden geçirilmelidir. |
| VS-04 | NFR-05'teki doğruluk hedefleri (%95 / %85) kaynaktan alınmamıştır. Bölüm 3.5'teki çalışmalar kişisel fotoğraf kütüphanesi değil uydu, meme ve yüz veri setleri kullanmıştır. Bu nedenle sonuçları doğrudan aktarılamaz ve hedef kendi test setimizle doğrulanmalıdır. |
| VS-05 | iOS'ta uygulama üzerinden yapılan silmenin bir sistem onayı gerektirdiği ve silinen öğelerin "Son Silinenler" albümüne gittiği varsayılmıştır. Apple'ın ilgili geliştirici belgesi (`deleteAssets`) bu oturumda içeriğiyle açılamadığı için doğrulanamadı. Android davranışı resmî belgeyle doğrulanmıştır (Bölüm 3.4). |
| VS-06 | WCAG 2.2 bir web standardıdır. Mobil uygulamada erişilebilirlik için referans olarak kullanılması bu dokümanın varsayımıdır. Platformların kendi tasarım kılavuzlarındaki dokunma hedefi değerleri bu oturumda araştırılmadı. |
| VS-07 | Kopya bulucu uygulamaların Google Play'de geniş fotoğraf erişimi için uygun olduğu varsayılmıştır. Politika sayfası kopya bulucuları adıyla anmıyor, yalnızca "temel işlevi fotoğraflara geniş erişim etrafında dönen" uygulamalardan ve galeri örneğinden söz ediyor. Kesin durum beyan sürecinde netleşir (AS-03). |
| VS-08 | Android photo picker'ın tek başına tüm kütüphane taramasına uygun olmadığı, sayfadaki "aynı anda en fazla 5.000 medya izni" bilgisinden çıkarılmıştır. Bu bir çıkarımdır, TA tarafından doğrulanmalıdır. |
| VS-09 | **Açılamayan veya kullanılamayan kaynaklar:** Google Play mağaza sayfaları (Remo Duplicate Photos Remover, Similar Photo Cleaner AI) içerik döndürmediği için Android tarafındaki üçüncü taraf rakipler doğrulanamadı. Bu yüzden rakip tablosundaki üçüncü taraf uygulamalar iOS sürümleridir. Diğer açılamayan veya kullanılamayan kaynaklar: mevzuat.gov.tr KVKK PDF'i (metin okunamadı; KVKK metni Lexpera'dan alındı), kvkk.gov.tr (sertifika hatası), arXiv 2304.02296 PDF sürümü (HTML sürümü kullanıldı), Apple `deleteAssets` ve `performChanges` belge sayfaları (gövde metni gelmedi). "Duplicate Photos Fixer Pro" (App Store id951695705) sayfası açıldı ancak yalnızca Mac sürümü olduğu için rakip olarak kullanılmadı. |
| VS-10 | **Uygulanamayan alt başlıklar:** Eğitim müfredatı ve sektör sertifikasyonu gibi alan standartları bu ürün için geçerli değildir, çünkü ürün kişisel kullanıma yönelik bir yardımcı araçtır. Bu başlıkta kişisel veri ve erişilebilirlik ele alınmıştır. |

---

## 9. Açık sorular

| ID | Soru | Neden önemli |
|---|---|---|
| AS-01 | iCloud Fotoğrafları veya Google Photos'ta yalnızca bulutta duran (cihaza indirilmemiş) fotoğraflar taranacak mı? Taranacaksa orijinallerin indirilmesi kabul ediliyor mu? | Tarama süresini, veri kullanımını ve NFR-01'in yorumunu etkiler. iCloud senkronizasyonu nedeniyle silmenin diğer cihazlara yansıması kullanıcıya ayrıca anlatılmalı mı? |
| AS-02 | "Kalacak" kopya önerisinin kuralı ne olmalı? Örneğin önce en yüksek çözünürlük, sonra en büyük dosya, sonra en eski tarih. Düzenlenmiş sürümler veya favoriler önceliklendirilmeli mi? | FR-08'in kabul testini belirler. |
| AS-03 | Google Play fotoğraf izinleri beyanı reddedilirse Android sürümü nasıl konumlanacak? Yalnızca seçili fotoğraflarla çalışan sınırlı bir sürüm kabul edilebilir mi? | Android'de iş modelini ve yayın takvimini doğrudan etkileyen bir risktir. |
| AS-04 | Monetizasyon modeli ne olacak (ücretsiz, tek seferlik ücret, abonelik, ücretsiz kullanım limiti, reklam)? | Rakiplerin bir kısmı ücretli veya limitli. Reklam veya analiz SDK'sı eklenirse "veri toplanmıyor" beyanı (NFR-01, Data safety) değişebilir. Abonelik App Review 3.1.2 kurallarına tabidir. |
| AS-05 | Uygulama hiçbir veriyi cihazdan çıkarmadığında geliştirici KVKK ve GDPR açısından nasıl nitelendirilir (veri sorumlusu olup olmadığı, aydınlatma metni gerekip gerekmediği)? | Hukuki değerlendirme gerektirir. Gizlilik politikası metnini belirler. |
| AS-06 | Performans hedefleri için her platformda hangi "referans orta segment cihaz" ve en düşük işletim sistemi sürümü esas alınacak? | NFR-02, NFR-03 ve NFR-07'nin ölçülebilmesi için gereklidir. |
| AS-07 | Türkçe dışında hangi diller desteklenecek? Hedef pazarlar hangileri? | NFR-10 ve mağaza metinlerini etkiler. |
| AS-08 | "Benzer" kapsamı ne kadar geniş olmalı: yalnızca yeniden sıkıştırılmış veya boyutu değiştirilmiş kopyalar mı, yoksa kırpılmış veya döndürülmüş kopyalar ve seri çekim kareleri de mi? | Bölüm 3.5'e göre bazı yöntemler kırpma ve döndürmede zayıf, bazıları ise daha yavaştır. Bu karar yöntem seçimini (TA) ve NFR-02/NFR-05 dengesini belirler. |
| AS-09 | HEIC/HEIF, RAW, Live Photo ve GIF gibi biçimler tarama kapsamına dahil mi? Bir Live Photo'nun fotoğraf bileşeni tek başına mı değerlendirilecek? | FR-03 kapsamını ve test setini (NFR-05) etkiler. |

---

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | Apple Explains Why Duplicates Album in Photos Not Showing for Some Users (MacRumors, 2022-09-28) | https://www.macrumors.com/2022/09/28/apple-explains-why-photos-duplicates-no-show/ | Apple'ın açıklaması (haberde aktarıldığı şekliyle): tespit "requires your iPhone to be locked and connected to power", süreç "could happen quickly or take up to a few days". Fazla kopyaların Recently Deleted'a gittiği bilgisi yalnızca bir okur yorumunda geçiyor. |
| 2 | Merge duplicate photos and videos on iPhone (Apple iPhone Kullanım Kılavuzu) | https://support.apple.com/en-gb/guide/iphone/iph1978d9c23/ | "The Photos app identifies duplicate photos and videos in your photo library in the Duplicates collection." Merge'in yer açma amacı ve adımları. |
| 3 | Delete duplicate files (Files by Google Yardım) | https://support.google.com/files/answer/9764075?hl=en-GB | "The original file is marked with an 'Original' badge." Seçilen dosyaların Bin'e taşınması. |
| 4 | Organize your Photos view (Google Photos Yardım) | https://support.google.com/photos/answer/14169846 | Photo stacks: "Nearly identical photos of the same subject that were taken together within a short time frame". "doesn't change your available storage". "only available for backed up photos". Top pick. |
| 5 | Duplicate Photos Fixer (App Store, id966475595) | https://apps.apple.com/us/app/-/id966475595 | Normal/Aggressive/Custom tarama, benzerlik seviyesi, Magic wand ("All copies except one image in each group will be marked"), kalıcı silme, 7,99 USD, karışık kullanıcı yorumları. |
| 6 | Remo Duplicate Photos Remover (App Store, id1066797785) | https://apps.apple.com/app/id1066797785 | Birebir ve benzer tespit, "Automatically keeps one best copy safe in each group", önizleme, ücretsiz, 4,6/14K, geliştirici gizlilik beyanı, 2020 abonelik yorumu. |
| 7 | Details on Google Play's Photo and Video Permissions policy | https://support.google.com/googleplay/android-developer/answer/14115180 | "Apps whose core functionality revolves around broad access to Photos and Videos may use the permissions above." Play Console beyan zorunluluğu, 28 Mayıs 2025 uyum tarihi (uzatma alanlar için), "makul çaba" yükümlülüğü. |
| 8 | Provide information for Google Play's Data safety section | https://support.google.com/googleplay/android-developer/answer/10787469 | Formun her uygulama için zorunlu olması. "Collect" tanımı (cihaz dışına iletim). Yalnızca cihazda işlenen verinin toplama sayılmaması. |
| 9 | Grant partial access to photos and videos (Android 14) | https://developer.android.com/about/versions/14/changes/partial-photo-video-access | `READ_MEDIA_VISUAL_USER_SELECTED`; "Don't assume full access to photos and videos"; izin durumunun kalıcı saklanmaması; kullanıcının seçimini yönetebilmesi. |
| 10 | Photo picker (Android Developers) | https://developer.android.com/training/data-storage/shared/photopicker | Seçicinin depolama izni gerektirmemesi. "An app can only have up to 5,000 media grants at any one time." |
| 11 | Access media files from shared storage (Android Developers) | https://developer.android.com/training/data-storage/shared/media | `createTrashRequest()` / `createDeleteRequest()`; "users see a dialog that requests their consent for your app to update or delete the specified media files". Çöp kutusundaki öğelerin sistem tanımlı süre sonra kalıcı silinmesi. |
| 12 | Handle the Limited Photos Library in your app (WWDC20, oturum 10641) | https://developer.apple.com/videos/play/wwdc2020/10641/ | iOS 14 sınırlı erişim; "User albums however cannot be fetched, nor created"; seçimin uygulama içinden yönetilmesi önerisi. |
| 13 | App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 5.1.1(i) gizlilik politikası; 5.1.1(ii) "Ensure your purpose strings clearly and completely describe your use of the data"; 5.1.1(iii) mümkünse tam Fotoğraflar erişimi yerine seçici; 2.3.1(a) yanıltıcı pazarlama; 3.1.2 abonelik şeffaflığı. |
| 14 | Art. 5 GDPR – Principles relating to processing of personal data | https://gdpr-info.eu/art-5-gdpr/ | 5(1)(c) "adequate, relevant and limited to what is necessary"; 5(1)(e) saklama sınırlaması. |
| 15 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (Lexpera) | https://www.lexpera.com.tr/mevzuat/kanunlar/kisisel-verilerin-korunmasi-kanunu-6698 | Md. 4(2)(ç) "İşlendikleri amaçla bağlantılı, sınırlı ve ölçülü olma"; md. 4(2)(d) saklama süresi; md. 28(1)(a) ev içi faaliyet istisnası. |
| 16 | Web Content Accessibility Guidelines (WCAG) 2.2 (W3C) | https://www.w3.org/TR/WCAG22/ | 1.4.3 "contrast ratio of at least 4.5:1"; 2.5.8 "at least 24 by 24 CSS pixels". |
| 17 | Adimoolam, Poullis, Averkiou – Data Leakage Detection and De-duplication in Large Scale Geospatial Image Datasets (arXiv 2304.02296v2) | https://arxiv.org/html/2304.02296v2 | 10.000 görüntülü coğrafi benchmark (birebir kopya, döndürme ve çevirme varyantları); 64 bit hash, Hamming eşiği 0; Tablo 6: pHash P 1,0000 / R 0,9997 / F1 0,9999 (FP 0, FN 2), aHash P 0,9164 / R 0,9709 / F1 0,9429 (FP 664, FN 218). |
| 18 | Needle In A Haystack, Fast: Benchmarking Image Perceptual Similarity Metrics At Scale (arXiv 2206.00282) | https://arxiv.org/html/2206.00282 | BSDS500, ImageNet ve Kaggle meme veri setleri. dHash "matching or exceeding all other algorithms" (güçlü döndürme, eğme ve kırpma hariç); "the most computationally efficient algorithm"; SimCLRv2 ResNet50 2x "about 10 times slower than Dhash"; memelerde FPR dHash %0,05, pHash %0,90, aHash %6,4 (BSDS500'de yaklaşık %0,5 FPR için seçilen eşikte). |
| 19 | Double Trouble? Impact and Detection of Duplicates in Face Image Datasets (arXiv 2401.14088) | https://arxiv.org/html/2401.14088 | pHash'in kırpmaya dayanıklı hash'ten fazla kopya bulması; yanlış pozitif grupların "can even consist of images that a manual inspection can easily identify as different faces"; ikinci aşama doğrulama. |
| 20 | Perceptual hashing (Wikipedia) | https://en.wikipedia.org/wiki/Perceptual_hashing | Kriptografik hash'teki çığ etkisi ile algısal hash'in farkı; algısal hash'lerin hata ve çakışma riskleri ("Perceptual hashes are messy", Kuederle). |