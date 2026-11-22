# Çarpım Tablosu Öğrenme Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v2
- **Girdi:** Basit bir çarpım tablosu öğrenme uygulaması yapacağız. Bu konuda araştırmalar yaparak mevcut çalışmaları da göz önünde bulundurarak iş gerekliliklerini belirle ve bize bir analiz dokümanı hazırla.

---

## 1. Özet

Bu ürün, ilkokul çağındaki çocukların çarpım tablosunu öğrenmesi ve akıcı biçimde hatırlaması için tasarlanmış küçük bir uygulamadır. Görev metni ürünü "basit" diye tanımlıyor. Bu nedenle kapsam dar tutuldu: tablo seçimi, tabloyu inceleme, soru-cevap alıştırması, anında geri bildirim, yanlışların tekrarı ve cihazda saklanan ilerleme.

Araştırmanın ana bulguları şunlar:

- **Müfredat:** MEB'in Türkiye Yüzyılı Maarif Modeli (TYMM) programında çarpım tablosu iki aşamada kuruluyor. 2. sınıfta 5'e kadar (5 dâhil), 3. sınıfta 10'a kadar (10 dâhil) ([TYMM 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164), [TYMM 3. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)). İncelenen yabancı rakipler ise genellikle 1–12 aralığını kullanıyor.
- **Öğretim yöntemi:** 2. sınıf öğrencileriyle yapılan bir sınıf içi çalışmada, kartlarla geri getirme pratiği, olguları sesli tekrar etmeye göre hatırlama akıcılığını hem kısa hem uzun vadede daha çok artırdı ([Ophuis-Cox vd., 2023](https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/)). Bu yüzden ürünün merkezine "soru sor, çocuk cevabı hatırlasın" mantığı konuldu.
- **Çocuk verisi ve mağaza kuralları:** Çocuk verisi ve mağaza politikaları ürün kararlarını doğrudan etkiliyor. Apple Kids kategorisinde üçüncü taraf analitik ve reklam genel olarak yasak ([Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)). Google Play'de ise yalnızca çocuklara yönelik uygulamalar reklam kimliği (AAID) gibi tanımlayıcıları gönderemiyor ([Google Play Families Policy](https://support.google.com/googleplay/android-developer/answer/9893335)). Bu nedenle ilk sürüm hesap gerektirmeyecek, kişisel veri toplamayacak ve çevrimdışı çalışacak şekilde tanımlandı.

İlk sürümün (MVP) sınırı, "M" öncelikli 7 fonksiyonel gereksinimle (FR-01, FR-02, FR-04 – FR-08) çizildi.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Kaynak / dayanak |
|---|---|---|---|
| **P1: Öğrenen çocuk (Elif, 8 yaş, 3. sınıf)** | Çarpım tablosunu okulda öğreniyor. Okuma becerisi gelişiyor, dikkat süresi kısa. | Kısa, anlaşılır alıştırma; hatasında ne olduğunu görmek; başarı hissi. | MEB 3. sınıf: 10'a kadar çarpım tablosu ([TYMM 3. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)). Rakipler 2.–4. sınıfı hedefliyor ([Simply Times Tables](https://apps.apple.com/app/id1472813038)). |
| **P2: Başlangıç düzeyindeki çocuk (Can, 7 yaş, 2. sınıf)** | Çarpmayı tekrarlı toplama olarak yeni öğreniyor. | Önce 1–5 tablolarını görsel destekle anlamak. | MEB 2. sınıf: 5'e kadar çarpım tablosu, çarpmanın toplama işlemine dayalı çözümlenmesi ([TYMM 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164)). |
| **P3: Ebeveyn (Ayşe, 38 yaş)** | Çocuğunun evde pratik yapmasını istiyor. Reklam, satın alma ve veri paylaşımı konusunda hassas. | Çocuğun hangi tablolarda zorlandığını görmek; güvenli, reklamsız ortam. | Rakiplerde ebeveyn alanı ve ilerleme görünümü var ([Simply Times Tables](https://apps.apple.com/app/id1472813038), [Times Tables Quiz](https://apps.apple.com/app/id1281267400)). |

Öğretmen persona'sı ilk sürüme alınmadı (bkz. Kapsam dışı maddeler, Açık sorular S-03).

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakip uygulamalar

| Çözüm | Öne çıkan özellikler | Güçlü yön | Zayıf yön / risk |
|---|---|---|---|
| **Simply Times Tables: Practice** (iOS) ([kaynak](https://apps.apple.com/app/id1472813038)) | 1×1–12×12 arası 144 olgu. 12 bölgelik macera haritası, hız turları (1–9 veya 1–12), eşit gruplar ve ritmik sayma ile görsel ipuçları, karışık tekrar, seri ve çıkartma ödülleri, ebeveyn ilerleme görünümü, ayrı çocuk profilleri. Geliştirici beyanına göre çevrimdışı çalışıyor, reklam ve abonelik yok, veri toplamıyor. | Görsel ipucu ile strateji desteği; reklamsız ve verisiz model. | Fiyat bilgisi sayfada çelişkili ($3,00 listelenmiş, ayrıca $0,99'luk "Unlock All Features"). Mağazada tek değerlendirme var (1,0/5) ve bu kullanıcı yorumu uygulamanın cevap gönderilirken kapandığını söylüyor. |
| **Times Tables Quiz: Multiply** (iOS) ([kaynak](https://apps.apple.com/app/id1281267400)) | 1–12 tabloları, tek tablo veya özel tablo seçimi, 12 soruluk görevler. "Normal" modda dikkat gerektiren tablolara odaklanan uyarlanabilir sınavlar. Olguları sesli okuma, sesli geri bildirim (kapatılabilir). Geri sayım yok; geri bildirim ve açıklama ekranda kalıyor. Hata tekrarı, sonraki tablo önerisi, doğruluk, seri ve rozet takibi. İlerleme cihazda; hesap, reklam ve izleme yok. | Süre baskısı olmaması; zayıf tablolara uyarlama; hesapsız gizlilik modeli. | Mağazada yalnızca 2 değerlendirme var (ortalama 2,0/5). Kullanıcı memnuniyeti doğrulanamıyor. |
| **Multiplication: Fun Times Tables for Kids** (iOS, Sierra SkyWare) ([kaynak](https://apps.apple.com/ca/app/multiplication/id304569626)) | "Teach Me" (dönen sayı çarkları), "Test Me" (tek soru, anında görsel geri bildirim, doğru/yanlış sayacı), "Compete" (Game Center üzerinden gerçek zamanlı çevrim içi yarışma). Çarpan bazlı karne, yanlışların kaydı, karakter özelleştirme, liderlik tablosu. | Çarpan bazlı karne ve yanlışların ayrıca kaydedilmesi. | $0,99 fiyatın üstüne uygulama içi coin satışı var. Küresel çevrim içi yarışma, küçük çocuklar için ek gizlilik ve içerik riskleri getiriyor. Kapsanan tablo aralığı sayfada belirtilmemiş. |
| **Times Tables Rock Stars (TTRS)** (web + mobil, abonelik) ([kaynak](https://www.ttrockstars.com/)) | Ürün sayfasına göre 15.000'den fazla okulda kullanılıyor ve 6 yaş ve üzerine uygun. Çocuğun ihtiyacına uyarlanan soru tabanlı oyunlar, akıcılığa doğru ilerlemeyi gösteren ısı haritaları, canlı çok oyunculu oyunlar ve liderlik tabloları, ödül mağazası, özel eğitim gereksinimli öğrenciler için erişilebilirlik ayarları. Okul saatleri dışında varsayılan 20 dakikalık günlük oynama sınırı. | Uyarlanabilirlik, ilerlemenin ısı haritasıyla görünür olması, ekran süresi sınırı. | Abonelik modeli (fiyat sayfada yok). Okul odaklı ve kapsamı "basit uygulama" hedefinin çok üzerinde. |

**Platformun yerleşik çözümü:** iOS veya Android'de çarpım tablosu öğretmeye yönelik yerleşik bir çözüm bulunamadı (bkz. Varsayımlar V-07).

### 3.2 Alan standartları ve düzenlemeler

| Alan | Bulgu | Ürüne etkisi |
|---|---|---|
| **MEB öğretim programı (TYMM), 2. sınıf** | MAT.2.2.4: "Çarpma ve bölme işlemlerini toplama ve çıkarma işlemlerine dayalı olarak çözümleyebilme". Uygulama notunda: "5'e kadar (5 dâhil) çarpım tablosu oluşturulur" ([TYMM 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164)). | 1–5 tabloları ayrı seçilebilmeli; tekrarlı toplama görseli değerli. |
| **MEB öğretim programı (TYMM), 3. sınıf** | MAT.3.2.3: "Çarpma ve bölme işlemlerinin sonuçlarını tahminde bulunarak ve zihinden işlem yaparak muhakeme edebilme". Uygulama notunda: "10'a kadar (10 dâhil) çarpım tablosu oluşturulur" ([TYMM 3. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)). Bu sayfada "ezber" ve "akıcılık" ifadeleri geçmiyor; zihinden işlem vurgulanıyor. | Varsayılan aralık 1–10. Rakiplerdeki 1–12 aralığı Türkiye müfredatında yok (bkz. S-01). |
| **Kişisel veri: Türkiye (KVKK, 6698)** | Madde 3: kişisel veri "Kimliği belirli veya belirlenebilir gerçek kişiye ilişkin her türlü bilgi"dir. Madde 5(1): "Kişisel veriler ilgili kişinin açık rızası olmaksızın işlenemez." Madde 4(2)(ç): veriler "İşlendikleri amaçla bağlantılı, sınırlı ve ölçülü" olmalıdır ([KVKK metni](https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf)). Karataş'ın (2025) akademik makalesine göre Türk hukukunda çocukların kişisel verilerinin işlenmesine ve yaş sınırına ilişkin özel bir düzenleme yok ([TAAD](https://taad.taa.gov.tr/dergiler/makale/taad-64-9)). | Ad, yaş veya hesap gibi kişisel veri toplanmazsa rıza ve aydınlatma yükü en aza iner. Çocuk rızası konusunda hukuki belirsizlik var (bkz. S-05). |
| **Kişisel veri: ABD (COPPA)** | FTC SSS'ye göre COPPA "applies only to those websites and online services that collect, use, or disclose personal information from children". Kişisel veri, kalıcı tanımlayıcıları (çerezdeki müşteri numarası, IP adresi, cihaz seri numarası) da kapsıyor. Uygulamanın cihazdaki kişisel bilgiyle yalnızca etkileşmesi "collecting" sayılmıyor ([FTC COPPA SSS](https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions)). | Veriyi cihazda tutan, sunucuya kişisel veri göndermeyen model, ABD pazarına açılma durumunda uyum yükünü azaltır. |
| **Erişilebilirlik (WCAG 2.2)** | 1.4.3: metin kontrastı en az 4.5:1. 1.4.1: renk, bilgi aktarmanın tek görsel yolu olamaz. 2.2.1: içerikteki süre sınırları ayarlanabilir veya kapatılabilir olmalı. 2.5.8 (AA): dokunma hedefi en az 24×24 CSS pikseli ([WCAG 2.2](https://www.w3.org/TR/WCAG22/)). | Doğru/yanlış geri bildirimi yalnızca renkle (yeşil/kırmızı) verilmemeli. Süreli mod varsa kapatılabilmeli. |

### 3.3 Platform ve mağaza politikaları

Görevde platform belirtilmemiş. Bu bölüm olası platformlar (iOS App Store, Google Play) için yalnızca iş etkisi düzeyinde yazıldı.

| Politika | Bulgu | İş etkisi |
|---|---|---|
| **Apple: Kids kategorisi (1.3) ve Kids (5.1.4)** | "Apps in the Kids Category should not include third-party analytics or third-party advertising." Uygulama dışı bağlantılar ve satın alma fırsatları yalnızca "behind a parental gate" olabilir. Kids kategorisindeki uygulamalar "may not send personally identifiable information or device information to third parties." Kişisel veri toplayan uygulamalar gizlilik politikası içermeli ([Apple App Review Guidelines](https://developer.apple.com/app-store/review/guidelines/)). Kural 2.3.8'e göre "For Kids" gibi ifadeler Kids kategorisine ayrılmış. | Reklamla gelir modeli Kids kategorisiyle büyük ölçüde uyumsuz. Ebeveyn alanı için ebeveyn kapısı gerekli. Kids kategorisine girmeme kararı, mağaza metninde çocuk odaklı ifadeleri sınırlar. |
| **Google Play: Families politikası** | Hedef yaş grubu Play Console'da beyan edilir ve Google bu beyanı değiştirebilir. Yalnızca çocuklara yönelik uygulamalar AAID, IMEI veya IMSI gibi tanımlayıcıları gönderemez ve konum izni isteyemez. Reklam varsa yalnızca "Families Self-Certified Ads SDKs" kullanılabilir. Çocuklardan toplanan veriler, üçüncü taraf SDK'lar dâhil, beyan edilmeli. Uyumlu uygulamalar "Teacher Approved" değerlendirmesine başvurabilir, ancak dâhil edilme garanti değil ([Google Play Families Policy](https://support.google.com/googleplay/android-developer/answer/9893335)). | Reklamsız ve SDK'sız model, beyan ve inceleme riskini azaltır. Hiçbir cihaz izni (konum, kamera, mikrofon vb.) gerekmemeli. |

### 3.4 Yöntemler (iş açısından artı/eksi)

| Yöntem | Kanıt / kaynak | Artı (iş açısından) | Eksi / risk |
|---|---|---|---|
| **Geri getirme pratiği** (soru sorup cevabı hatırlatmak) vs. **tekrar okuma/koro** | 48 ikinci sınıf öğrencisiyle yapılan, denek içi desenli bir sınıf çalışmasında (her strateji için 1 ders ve 3 aralıklı alıştırma oturumu), kartlarla geri getirme pratiği sesli koro tekrarına göre "a stronger short‐term and long‐term increase" sağladı. Ölçümler 5 dakika ve 1 hafta sonra yapıldı ([Ophuis-Cox, Catrysse ve Camp, 2023](https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/)). | Ürünün çekirdek etkileşimi için kanıta dayalı gerekçe. Basit ve düşük maliyetli. | Örneklem küçük (48 öğrenci) ve tek bir okulda yapılmış. Etkinin ürün ortamında aynı olacağı garanti değil. |
| **Az sayıda olguyla başlama / artımlı tekrar** | EdWeek makalesinde psikolog Nicole McNeil: "You can't become fluent with all of them at once." Rastgele tüm kombinasyonlar yerine 2–4 olguluk küçük gruplar öneriyor. Makale, yeni olgunun bilinen olgularla birlikte çalışıldığı "incremental rehearsal" yöntemini anlatıyor ([EdWeek, 2023](https://www.edweek.org/teaching-learning/kids-need-to-know-their-math-facts-what-schools-can-do-to-help/2023/05)). | Çocuğun başarı hissini korur; tablo seçimiyle doğal biçimde uygulanabilir. | Doğru grup büyüklüğünü ayarlamak ek tasarım çalışması gerektirir. |
| **Aralıklı tekrar** (yanlışları sonraki güne taşıma) | Aynı makalede öğretmen Dylan Kane, kaçırılan olguları ertesi günün alıştırmasına geri getiren bir program kullandığını söylüyor ve "low-dose practice throughout the year" eksikliğine dikkat çekiyor ([EdWeek, 2023](https://www.edweek.org/teaching-learning/kids-need-to-know-their-math-facts-what-schools-can-do-to-help/2023/05)). | Uzun vadeli kalıcılığı destekler. Cihazda saklanan olgu geçmişiyle hesapsız uygulanabilir. | Bu kanıt bir öğretmenin uygulamasına dayanıyor, kontrollü çalışma değil. Olgu bazında geçmiş tutulmasını gerektirir. |
| **Uyarlanabilir soru seçimi** (zayıf olgulara ağırlık) | Times Tables Quiz'in "Normal" modu dikkat gerektiren tablolara odaklanıyor ([kaynak](https://apps.apple.com/app/id1281267400)). TTRS soruları çocuğun ihtiyacına göre ayarlıyor ([kaynak](https://www.ttrockstars.com/)). | Pratik süresinin verimli kullanımı; rakiplerle başa baş özellik. | Bunlar ürün beyanı; etkinlik kanıtı sunulmuyor. Çocuk için "neden hep aynı soru geliyor" algısı yaratabilir. |
| **Süreli alıştırma** | EdWeek makalesine göre McNeil süreli çalışmayı destekliyor ve çocukların akranlarıyla değil kendi süreleriyle yarışmasını öneriyor. Uzman Jill Milton ise süreli testlere karşı çıkıyor. Makale, "many teachers—and some scholars" süreli alıştırmaların kaygıyı artırdığını savunduğunu ve bu konuda kesin bir araştırma olmadığını aktarıyor ([EdWeek, 2023](https://www.edweek.org/teaching-learning/kids-need-to-know-their-math-facts-what-schools-can-do-to-help/2023/05)). Times Tables Quiz bilinçli olarak geri sayım kullanmıyor ([kaynak](https://apps.apple.com/app/id1281267400)). WCAG 2.2.1 süre sınırlarının kapatılabilmesini istiyor ([WCAG 2.2](https://www.w3.org/TR/WCAG22/)). | Akıcılığı (hız) ölçmeyi ve "kendi rekorunu kırma" motivasyonunu sağlar. | Kaygı riski tartışmalı. Varsayılan açık olursa erişilebilirlik sorunu doğar. Bu yüzden isteğe bağlı ve varsayılan kapalı önerildi. |
| **Strateji ve görsel destek** (tekrarlı toplama, eşit gruplar, ritmik sayma) | MEB 2. sınıf çıktısı çarpmayı toplamaya dayalı çözümletiyor ([TYMM 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164)). Simply Times Tables eşit gruplar ve ritmik sayma görselleri sunuyor ([kaynak](https://apps.apple.com/app/id1472813038)). EdWeek'te Bushart ve McNeil ileri sayma ve ikiye katlama gibi yedek stratejiler öneriyor ([EdWeek, 2023](https://www.edweek.org/teaching-learning/kids-need-to-know-their-math-facts-what-schools-can-do-to-help/2023/05)). | Müfredatla uyum; ezber yerine anlamaya destek. | Görsel içerik üretimi ek emek getirir. |
| **Rekabet / çok oyunculu oyun** | Sierra SkyWare uygulaması Game Center üzerinden küresel yarışma sunuyor ([kaynak](https://apps.apple.com/ca/app/multiplication/id304569626)); TTRS canlı çok oyunculu oyunlar ve liderlik tabloları sunuyor ([kaynak](https://www.ttrockstars.com/)). | Bazı çocuklar için motivasyon. | Çevrim içi bağlantı, hesap ve çocuk verisi gerektirir; "basit" kapsamın dışında. Kapsam dışı bırakıldı. |

---

## 4. Fonksiyonel gereksinimler

**M (Must) önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S ve C maddeleri ilk sürüm için önerilir ama ertelenebilir. Bu sürümde yapılmayacak maddeler `Kapsam dışı maddeler` bölümündedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kullanıcı, alıştırma veya inceleme için 1'den 10'a kadar tablolardan bir veya birden fazlasını seçebilmelidir. "1–5" ve "1–10" hazır seçim kısayolları bulunmalıdır. | M | MEB 2. sınıf 5'e kadar, 3. sınıf 10'a kadar ([TYMM 2](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164), [TYMM 3](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)); rakiplerde tek veya özel tablo seçimi var. |
| FR-02 | Kullanıcı, seçtiği tablonun tüm olgularını (ör. 7×1 … 7×10) sonuçlarıyla birlikte bir inceleme ekranında görebilmelidir. | M | Rakiplerdeki "Teach Me" / öğrenme modları; MEB'de tablonun oluşturulması. |
| FR-03 | İnceleme ekranında her olgu için çarpmanın tekrarlı toplama veya eşit gruplar olarak görsel gösterimi açılabilmelidir (ör. 3×4 = 4+4+4). | S | MEB MAT.2.2.4; Simply Times Tables görsel ipuçları. |
| FR-04 | Kullanıcı, seçili tablolardan oluşturulan bir alıştırma oturumu başlatabilmelidir. Oturumda olgular "a × b = ?" biçiminde tek tek sorulur ve kullanıcı cevabı sayısal olarak girer. Varsayılan oturum uzunluğu 10 sorudur. | M | Geri getirme pratiği ([Ophuis-Cox vd., 2023](https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/)). |
| FR-05 | Her cevaptan sonra anında doğru/yanlış geri bildirimi verilmelidir. Yanlış cevapta doğru sonuç gösterilmeli ve kullanıcı devam etmeyi seçene kadar ekranda kalmalıdır. | M | Times Tables Quiz ve Sierra SkyWare "Test Me" anında geri bildirim. |
| FR-06 | Oturumda yanlış cevaplanan olgular aynı oturumun sonunda en az bir kez yeniden sorulmalıdır. | M | Rakiplerdeki hata tekrarı; EdWeek'teki kaçırılan olguların geri getirilmesi. |
| FR-07 | Oturum sonunda bir özet gösterilmelidir: toplam soru, doğru sayısı, yanlış cevaplanan olguların listesi. | M | Times Tables Quiz doğruluk takibi; Sierra SkyWare doğru/yanlış sayacı. |
| FR-08 | Uygulama her olgu (a×b) için doğru ve yanlış cevap geçmişini cihazda kalıcı olarak saklamalıdır. | M | FR-09, FR-10 ve FR-11 için ön koşul; rakiplerde olgu ve tablo bazlı ilerleme takibi. |
| FR-09 | Kullanıcı, 10×10 olgu tablosu (ısı haritası) üzerinde her olgunun durumunu en az üç seviyede görebilmelidir: "henüz çalışılmadı", "çalışılıyor", "öğrenildi". Durumlar yalnızca renkle değil, simge veya metinle de ayırt edilmelidir. | S | TTRS ısı haritaları; WCAG 1.4.1. |
| FR-10 | Alıştırma soruları seçilirken geçmişte yanlış cevaplanan veya henüz "öğrenildi" durumuna geçmemiş olgulara, öğrenilmiş olgulardan daha yüksek ağırlık verilmelidir. | S | Times Tables Quiz uyarlanabilir modu; TTRS. |
| FR-11 | Önceki günlerde yanlış cevaplanan olgular sonraki günün ilk alıştırma oturumuna dâhil edilmelidir. | S | EdWeek'te Kane'in uygulaması (aralıklı tekrar). |
| FR-12 | Ebeveyn alanı (ayarlar, ilerleme sıfırlama) çocuğun kolayca geçemeyeceği bir ebeveyn kapısının (parental gate) arkasında olmalıdır. | S | Apple 1.3: bağlantı ve satın almalar ebeveyn kapısı arkasında; Times Tables Quiz "Grown-ups area". |
| FR-13 | Ebeveyn alanından oturum uzunluğu (ör. 5, 10, 20 soru) değiştirilebilmelidir. | C | EdWeek: "relatively short but consistent" alıştırma önerisi. |
| FR-14 | İsteğe bağlı bir süreli mod sunulmalıdır. Varsayılan olarak kapalı olmalı, açıldığında oturum süresi ölçülmeli ve kullanıcıya yalnızca kendi önceki en iyi süresiyle karşılaştırma gösterilmelidir. | C | EdWeek (McNeil: kendi süresini geçme); WCAG 2.2.1. |
| FR-15 | Soruların ve sonuçların sesli okunması açılıp kapatılabilmelidir. | C | Times Tables Quiz sesli olgular ve sesli geri bildirim. |
| FR-16 | Kullanıcı, art arda gün pratiği (seri) ve tamamlanan tablolar için basit rozetler görebilmelidir. | C | Simply Times Tables seri ve çıkartma; Times Tables Quiz rozet ve seri. |

---

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt | Dayanak |
|---|---|---|---|
| NFR-01 | **Kişisel veri toplanmaz.** Uygulama hesap, ad, yaş, e-posta, konum veya cihaz tanımlayıcısı istemez ve hiçbir veriyi cihaz dışına göndermez. | Ağ trafiği incelemesinde uygulama kaynaklı dış istek sayısı 0. Kayıt veya giriş ekranı sayısı 0. | KVKK md. 4–5; FTC COPPA SSS; Apple 1.3 / 5.1.4; Google Families. |
| NFR-02 | **Üçüncü taraf reklam ve analitik yok.** | Uygulamada üçüncü taraf reklam veya analitik bileşeni sayısı 0. | Apple 1.3 / 5.1.4; Google Families. |
| NFR-03 | **Cihaz izni istenmez** (konum, kamera, mikrofon, kişiler vb.). | Kurulum ve kullanım sırasında gösterilen izin isteği sayısı 0. | Google Families: konum izni yasağı; KVKK md. 4(2)(ç) ölçülülük. |
| NFR-04 | **Çevrimdışı çalışma.** Tüm M öncelikli işlevler internet bağlantısı olmadan çalışır. | Uçak modunda FR-01, FR-02 ve FR-04 – FR-08 test senaryolarının %100'ü geçer. | Rakipler çevrimdışı çalışıyor ([Simply Times Tables](https://apps.apple.com/app/id1472813038), [Times Tables Quiz](https://apps.apple.com/app/id1281267400)). |
| NFR-05 | **Veri kalıcılığı.** İlerleme, uygulama kapatılıp açıldığında veya cihaz yeniden başlatıldığında korunur. | 20 kapatma-açma döngüsünde ilerleme kaybı 0. | FR-08. |
| NFR-06 | **Hızlı geri bildirim.** Cevap gönderildikten sonra geri bildirim gecikmesi düşüktür. | Hedef cihazlarda gecikmenin %95'i ≤ 200 ms. | Varsayım V-04. |
| NFR-07 | **Kararlılık.** | Oturumların en az %99,5'i çökme olmadan tamamlanır (iç testlerde ölçülür). | Bir rakibin tek kullanıcı yorumunda bildirilen cevap gönderirken çökme sorunu ([Simply Times Tables](https://apps.apple.com/app/id1472813038)); hedef değer varsayım (V-04). |
| NFR-08 | **Erişilebilirlik (WCAG 2.2 AA esas alınarak).** Metin kontrastı ≥ 4.5:1; dokunma hedefleri ≥ 24×24 CSS pikseli (çocuk kullanıcılar için daha büyük hedef önerilir, bkz. V-05); doğru/yanlış bilgisi renkle birlikte simge veya metinle de verilir; süre sınırları kapatılabilir. | Erişilebilirlik denetiminde bu dört maddede hata 0. | [WCAG 2.2](https://www.w3.org/TR/WCAG22/) 1.4.3, 2.5.8, 1.4.1, 2.2.1. |
| NFR-09 | **Dil ve okunabilirlik.** Arayüz Türkçedir; metinler kısa ve 2. sınıf okuma düzeyine uygundur. Matematik terimleri MEB terminolojisiyle uyumludur (çarpan, çarpım). | Her arayüz metni en fazla 8 kelime (talimat ekranları hariç). Terimler TYMM ünite sayfalarıyla uyumlu. | [TYMM 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164); varsayım V-03. |
| NFR-10 | **Kolay başlangıç.** Çocuk, yetişkin yardımı olmadan ana ekrandan ilk soruya ulaşabilir. | Ana ekrandan ilk soruya en fazla 3 dokunuş. 7–9 yaş arası en az 5 çocukla yapılan kullanılabilirlik testinde çocukların en az %80'i yardımsız başlar. | Hedef kitle P1/P2; ölçüt değerleri varsayım (V-04). |
| NFR-11 | **Dış bağlantı ve satın alma.** Çocuk alanında uygulama dışı bağlantı veya satın alma yoktur; varsa yalnızca ebeveyn kapısı arkasındadır. | Çocuk ekranlarında dış bağlantı ve satın alma öğesi sayısı 0. | Apple 1.3. |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Tablo seçme** (FR-01)
Bir öğrenci olarak çalışmak istediğim tabloları seçmek istiyorum, böylece okulda öğrendiğim tablolara odaklanabilirim.
- KK-01.1: 1'den 10'a kadar her tablo tek tek seçilip bırakılabilir.
- KK-01.2: "1–5" kısayolu 1, 2, 3, 4 ve 5 tablolarını; "1–10" kısayolu tüm tabloları seçer.
- KK-01.3: Hiç tablo seçilmemişken alıştırma başlatılamaz ve kullanıcıya bir tablo seçmesi söylenir.

**US-02: Tabloyu inceleme** (FR-02, FR-03)
Çarpmayı yeni öğrenen bir öğrenci olarak bir tablonun tamamını ve nasıl oluştuğunu görmek istiyorum, böylece soruları cevaplamadan önce tabloyu anlayabilirim.
- KK-02.1: Seçilen tablo için 10 olgunun tamamı sonuçlarıyla listelenir (ör. 7×1=7 … 7×10=70).
- KK-02.2 (FR-03): Bir olguya dokunulduğunda tekrarlı toplama gösterimi açılır (ör. 3×4 → 4+4+4 = 12).

**US-03: Alıştırma yapma** (FR-04, FR-05)
Bir öğrenci olarak bana sorular sorulsun ve cevabımın doğru olup olmadığını hemen öğreneyim istiyorum, böylece tabloyu hatırlamayı pratik edebilirim.
- KK-03.1: Oturum varsayılan olarak 10 sorudan oluşur ve sorular yalnızca seçili tablolardan gelir.
- KK-03.2: Cevap gönderildiğinde doğru/yanlış bilgisi hem renk hem simge veya metinle gösterilir (NFR-08).
- KK-03.3: Yanlış cevapta doğru sonuç gösterilir ve kullanıcı "Devam" diyene kadar ekranda kalır.
- KK-03.4: Sayı dışı girişler kabul edilmez; boş cevap gönderilemez.

**US-04: Yanlışlarımı tekrar etme** (FR-06, FR-11)
Bir öğrenci olarak yanlış yaptığım soruları tekrar görmek istiyorum, böylece onları da öğrenebilirim.
- KK-04.1: Oturumda yanlış cevaplanan her olgu, aynı oturumun sonunda en az bir kez yeniden sorulur.
- KK-04.2 (FR-11): Bir gün önce yanlış cevaplanan olgular, ertesi günün ilk oturumunda en az bir kez sorulur.

**US-05: Oturum sonucumu görme** (FR-07)
Bir öğrenci olarak oturum bitince nasıl yaptığımı görmek istiyorum, böylece neyi bildiğimi ve neye çalışmam gerektiğini anlayabilirim.
- KK-05.1: Özet ekranında toplam soru, doğru sayısı ve yanlış cevaplanan olgular yer alır.
- KK-05.2: Özet ekranından aynı tablolarla yeni oturum başlatılabilir.

**US-06: İlerlemenin korunması** (FR-08)
Bir öğrenci olarak uygulamayı kapatıp açtığımda ilerlememin kaybolmamasını istiyorum, böylece kaldığım yerden devam edebilirim.
- KK-06.1: Uygulama kapatılıp yeniden açıldığında olgu bazındaki doğru/yanlış geçmişi korunur (NFR-05).
- KK-06.2: İlerleme internet bağlantısı olmadan kaydedilir (NFR-04).

**US-07: Zayıf olguları görme** (FR-09, FR-10)
Bir ebeveyn olarak çocuğumun hangi olgularda zorlandığını görmek istiyorum, böylece ona doğru yerde destek olabilirim.
- KK-07.1: 10×10 tablosunda her olgunun durumu ("henüz çalışılmadı", "çalışılıyor", "öğrenildi") renk ve simgeyle ayırt edilir.
- KK-07.2 (FR-10): Alıştırmalarda, "öğrenildi" durumunda olmayan olgular öğrenilmiş olgulardan daha sık sorulur. Doğrulama: 100 soruluk test oturumunda öğrenilmemiş olguların sorulma oranı, olgu sayısındaki paylarından yüksek çıkar.

**US-08: Ebeveyn ayarları** (FR-12, FR-13)
Bir ebeveyn olarak ayarları ve ilerleme sıfırlamayı çocuğumun yanlışlıkla değiştiremeyeceği bir yerde tutmak istiyorum, böylece çocuğun verisi kazara silinmez.
- KK-08.1: Ebeveyn alanına giriş bir ebeveyn kapısı gerektirir.
- KK-08.2: İlerleme sıfırlama, ek bir onay adımıyla yapılır.
- KK-08.3 (FR-13): Oturum uzunluğu 5, 10 veya 20 soru olarak ayarlanabilir.

**US-09: Kendi süremi geçme** (FR-14)
Tabloları iyi bilen bir öğrenci olarak ne kadar hızlı cevap verdiğimi görmek istiyorum, böylece kendi rekorumu geçmeye çalışabilirim.
- KK-09.1: Süreli mod varsayılan olarak kapalıdır ve ayarlardan açılır.
- KK-09.2: Oturum sonunda yalnızca kullanıcının kendi önceki en iyi süresiyle karşılaştırma gösterilir; başka kullanıcılarla karşılaştırma yapılmaz.

**US-10: Sesli destek ve ödüller** (FR-15, FR-16)
Okumada zorlanan veya motivasyona ihtiyaç duyan bir öğrenci olarak soruların sesli okunmasını ve emeğimin karşılığını görmek istiyorum.
- KK-10.1 (FR-15): Sesli okuma ayarı açıldığında her soru sesli okunur; kapatıldığında ses çıkmaz.
- KK-10.2 (FR-16): Art arda pratik yapılan gün sayısı ve tamamlanan tablolar için rozetler gösterilir.

---

## 7. Kapsam dışı maddeler

| Madde | Neden kapsam dışı |
|---|---|
| Kullanıcı hesabı, giriş ve bulut senkronizasyonu | Kişisel veri toplama ve çocuk verisi yükümlülükleri doğurur (NFR-01). "Basit" ürün kapsamına uymaz. |
| Çevrim içi çok oyunculu oyun, liderlik tablosu | Bağlantı ve hesap gerektirir; çocuk güvenliği ve gizlilik riski var (bkz. Bölüm 3.4). |
| Öğretmen ve sınıf yönetimi paneli | Görevde istenmemiş; TTRS gibi okul ürünlerinin ölçeği (bkz. S-03). |
| Bölme işlemi alıştırmaları | Görev yalnızca çarpım tablosunu kapsıyor. Bazı rakipler bölmeyi de sunuyor. |
| 11 ve 12 tabloları (veya üstü) | MEB programında çarpım tablosu 10'a kadar (bkz. S-01). |
| Çok basamaklı çarpma (ör. üç basamaklı × bir basamaklı) | MEB 3. sınıf programında var ([TYMM 3](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)), ancak bu çarpım tablosu değil. |
| Eksik çarpanlı soru biçimleri (ör. "? × 4 = 20") | Görevde istenmemiş; ileri sürüm adayı. |
| Reklam ve uygulama içi satın alma | Gelir modeli belirsiz (S-04); mağaza politikalarında ek kısıtlar getiriyor. |
| Birden fazla çocuk profili | Rakiplerde var, ancak basit kapsam için gerekli görülmedi (S-02). |

---

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| V-01 | Hedef kitle Türkiye'deki ilkokul 2.–3. sınıf öğrencileri ve ebeveynleridir. Görevde hedef ülke veya yaş belirtilmedi; bu varsayım MEB programına dayanılarak yapıldı. |
| V-02 | Hedef platform belirtilmediği için iOS ve Android mağaza politikaları yalnızca iş etkisi düzeyinde değerlendirildi. Web veya başka platformlar için ayrıca politika incelemesi yapılmadı. |
| V-03 | Arayüz dili yalnızca Türkçedir. |
| V-04 | NFR-06 (≤ 200 ms), NFR-07 (%99,5) ve NFR-10'daki (3 dokunuş, %80) hedef değerler kaynaktan alınmadı; ekip için başlangıç hedefi olarak önerildi ve paydaş onayı gerekiyor. |
| V-05 | WCAG 2.2 web içeriği için yazılmış bir standarttır. Bu dokümanda mobil uygulama için de referans ölçüt olarak alındı. 24×24 CSS piksel değeri minimum sınırdır; küçük çocuklar için daha büyük hedef önerisi kaynakla desteklenmiş bir değer değil, analist önerisidir. |
| V-06 | "Öğrenildi" durumunun tanımı (ör. bir olgunun art arda kaç kez doğru cevaplanması gerektiği) kaynaklarda bulunamadı. Eşik değer paydaşla belirlenmelidir (S-06). |
| V-07 | iOS veya Android'de çarpım tablosu öğretmeye yönelik yerleşik bir çözüm bulunamadı. Platformların yerleşik hesap makineleri öğretim amaçlı olmadığı için rakip olarak değerlendirilmedi. Bu tespit bir aramaya değil, araştırma sırasında böyle bir ürünle karşılaşılmamasına dayanıyor. |
| V-08 | MEB TYMM programının birleşik PDF'leri (2024 onaylı program ve 1–4. sınıf Matematik DÖP) bu çalışmada açılamadı. Müfredat bilgisi TYMM'nin 2. ve 3. sınıf ünite web sayfalarından alındı. Programın en güncel sürümüyle tutarlılık kontrol edilmelidir. |
| V-09 | Rakip özellikleri, uygulama mağazalarındaki geliştirici beyanlarına dayanıyor. Uygulamalar indirilip denenmedi. Veri toplamama beyanları Apple tarafından doğrulanmamış (mağaza sayfasındaki ifadeye göre). |
| V-10 | Ophuis-Cox vd. (2023) çalışmasının yalnızca özet sayfası okundu; tam metin incelenmedi. |
| V-11 | Bu ürünün kişisel veri işlemeyeceği (NFR-01) varsayıldığı için KVKK kapsamında açık rıza ve aydınlatma akışları gereksinim olarak yazılmadı. Kapsam değişirse bu varsayım yeniden değerlendirilmelidir. |

---

## 9. Açık sorular

| ID | Soru | Etkilediği gereksinim |
|---|---|---|
| S-01 | Tablo aralığı MEB'e uygun olarak 1–10 mu kalmalı, yoksa yabancı rakipler gibi 1–12'ye mi genişletilmeli? | FR-01, FR-09 |
| S-02 | Aynı cihazda birden fazla çocuk kullanacak mı? Kullanacaksa çocuk profilleri ilk sürüme alınmalı mı? | FR-08, Kapsam dışı |
| S-03 | Okullarda veya öğretmenlerce kullanım hedefleniyor mu? | Kapsam dışı (öğretmen paneli) |
| S-04 | Gelir modeli nedir (ücretsiz, tek seferlik ücret, reklam, abonelik)? Reklam veya satın alma, Apple Kids ve Google Families kurallarıyla ek kısıt getirir. | NFR-02, NFR-11, FR-12 |
| S-05 | Hedef platform(lar) ve pazar(lar) hangileri? Uygulama Apple Kids kategorisine veya Google Play'de "yalnızca çocuklar" hedef kitlesine mi beyan edilecek? | NFR-01 – NFR-03, Bölüm 3.3 |
| S-06 | Bir olgu ne zaman "öğrenildi" sayılmalı (ör. art arda 3 doğru cevap, belirli bir yanıt süresi)? | FR-09, FR-10 |
| S-07 | Süreli mod (FR-14) ilk sürümde olmalı mı? Kaygı riski konusunda pedagojik bir danışman görüşü alınacak mı? | FR-14 |
| S-08 | Sesli okuma için hangi ses tercih edilir (kayıtlı insan sesi veya cihazın konuşma sentezi)? Bu, maliyet ve kaliteyi etkiler. | FR-15 |
| S-09 | Ürünün etkisi (öğrenme kazanımı) ölçülecek mi? Ölçülecekse kişisel veri toplamadan nasıl ölçüleceği belirlenmeli. | NFR-01 |

---

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | T.C. MEB TYMM: İlkokul Matematik Dersi, 2. sınıf, 3. Tema | https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164 | MAT.2.2.4 "Çarpma ve bölme işlemlerini toplama ve çıkarma işlemlerine dayalı olarak çözümleyebilme"; uygulama notu: "5'e kadar (5 dâhil) çarpım tablosu oluşturulur." |
| K2 | T.C. MEB TYMM: İlkokul Matematik Dersi, 3. sınıf, 3. Tema (İşlemlerden Cebirsel Düşünmeye) | https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264 | MAT.3.2.3 "…tahminde bulunarak ve zihinden işlem yaparak muhakeme edebilme"; "10'a kadar (10 dâhil) çarpım tablosu oluşturulur"; 10'a kadar sayıların 9'a kadar rakamlarla çarpılması; üç basamaklı × bir basamaklı çarpma. Sayfada "ezber" ve "akıcılık" ifadeleri geçmiyor. |
| K3 | Ophuis-Cox, Catrysse ve Camp (2023), "The effect of retrieval practice on fluently retrieving multiplication facts…", Applied Cognitive Psychology 37(6) (özet sayfası) | https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/ | 48 ikinci sınıf öğrencisi, denek içi desen, kartlarla geri getirme vs. koro tekrar; "retrieval practice led to a stronger short‐term and long‐term increase" (5 dk ve 1 hafta sonraki son testler). |
| K4 | EdWeek (Stephen Sawchuk), "Kids Need to Know Their Math Facts. What Schools Can Do to Help" (Mayıs 2023) | https://www.edweek.org/teaching-learning/kids-need-to-know-their-math-facts-what-schools-can-do-to-help/2023/05 | McNeil: "You can't become fluent with all of them at once", 2–4 olguluk gruplar, kendi süresini geçme. Kane: kaçırılan olguları ertesi güne taşıma, "low-dose practice". Makale: süreli testlerin kaygıyı artırdığı "many teachers—and some scholars" tarafından savunuluyor, kesin araştırma yok. Bushart ve McNeil: yedek stratejiler. |
| K5 | App Store: Simply Times Tables: Practice | https://apps.apple.com/app/id1472813038 | 144 olgu (1–12), hız turları, görsel ipuçları, seri ve çıkartma, ebeveyn görünümü, çevrimdışı, "No ads or subscriptions", veri toplamama beyanı. Fiyat bilgisi çelişkili. 1 değerlendirme (1,0/5); kullanıcı yorumunda cevap gönderirken kapanma bildirimi. |
| K6 | App Store: Times Tables Quiz: Multiply | https://apps.apple.com/app/id1281267400 | 1–12, uyarlanabilir "Normal" mod, sesli olgular ve sesli geri bildirim, geri sayım yok, hata tekrarı, cihazda ilerleme, hesap, reklam ve izleme yok. 2 değerlendirme (2,0/5). |
| K7 | App Store: Multiplication: Fun Times Tables for Kids (Sierra SkyWare) | https://apps.apple.com/ca/app/multiplication/id304569626 | Teach Me / Test Me / Compete (Game Center) modları, çarpan bazlı karne, $0,99 fiyat ve coin satın alımları, veri toplamama beyanı (Apple tarafından doğrulanmamış). |
| K8 | Times Tables Rock Stars ana sayfası | https://www.ttrockstars.com/ | 15.000'den fazla okul (ürün beyanı), 6 yaş ve üzeri, uyarlanabilir oyunlar, ısı haritaları, çok oyunculu oyun ve liderlik tabloları, okul saatleri dışında varsayılan 20 dakikalık günlük sınır, abonelik. |
| K9 | Apple App Store Review Guidelines (1.3, 5.1.4, 2.3.8) | https://developer.apple.com/app-store/review/guidelines/ | "Apps in the Kids Category should not include third-party analytics or third-party advertising"; bağlantı ve satın almalar "behind a parental gate"; kişisel veri toplayan uygulamalarda gizlilik politikası zorunluluğu; "For Kids" ifadesinin Kids kategorisine ayrılması. |
| K10 | Google Play Console Yardım: Families Policy | https://support.google.com/googleplay/android-developer/answer/9893335 | Hedef yaş beyanı; yalnızca çocuklara yönelik uygulamalarda AAID, IMEI ve IMSI gönderimi ile konum izni yasağı; "Families Self-Certified Ads SDKs"; Teacher Approved'a dâhil edilmenin garanti olmaması. |
| K11 | FTC: Complying with COPPA: Frequently Asked Questions | https://www.ftc.gov/business-guidance/resources/complying-coppa-frequently-asked-questions | COPPA "applies only to those websites and online services that collect, use, or disclose personal information from children"; kalıcı tanımlayıcılar kişisel veri sayılıyor; cihazdaki bilgiyle yalnızca etkileşmek toplama sayılmıyor (F.5). |
| K12 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (mevzuat.gov.tr) | https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf | Md. 3 kişisel veri ve açık rıza tanımları; md. 4(2)(ç) "sınırlı ve ölçülü olma"; md. 5(1) "Kişisel veriler ilgili kişinin açık rızası olmaksızın işlenemez." |
| K13 | Karataş, E. (2025), "Çocukların Kişisel Verilerinin İşlenmesinde Açık Rıza ve Yaş Sınırına İlişkin Mukayeseli Bir İnceleme", TAAD 16(64) | https://taad.taa.gov.tr/dergiler/makale/taad-64-9 | Yazarın tespiti: "Türk hukukunda ise çocukların kişisel verilerinin işlenmesine ve yaş sınırına ilişkin herhangi bir düzenleme bulunmamaktadır". COPPA'da yaş sınırı 13, GDPR'de 16 (üye devletler 13'e indirebilir). |
| K14 | W3C: Web Content Accessibility Guidelines (WCAG) 2.2 | https://www.w3.org/TR/WCAG22/ | 1.4.3 kontrast "at least 4.5:1"; 1.4.1 renk tek görsel yol olamaz; 2.2.1 süre sınırı ayarlanabilir veya kapatılabilir olmalı; 2.5.8 hedef boyutu "at least 24 by 24 CSS pixels". |

**Açılamadığı veya kullanılmadığı için kaynak gösterilmeyen sayfalar:**
- MEB TYMM 2024 onaylı matematik programı PDF'i: HTTP 500 hatası verdi (bkz. V-08).
- Springer'deki aralıklı uygulama ve problem seti büyüklüğü makalesi: kimlik doğrulama yönlendirmesi nedeniyle açılamadı.
- ExploreLearning Reflex "multiplication fact fluency" sayfası: HTTP 403 hatası verdi.
- kvkk.gov.tr kanun sayfası: sertifika hatası verdi; kanun metni mevzuat.gov.tr'den okundu.
- FTC COPPA kural sayfası: açıldı ancak yükümlülükleri ayrıntılı vermediği için kullanılmadı.
- Arama sonuçlarında görülen KQED, Hechinger Report ve Khan Academy topluluk sayfaları: açılmadığı için kaynak gösterilmedi.