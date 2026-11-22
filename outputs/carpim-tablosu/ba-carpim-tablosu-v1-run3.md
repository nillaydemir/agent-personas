# Çarpım Tablosu Öğrenme Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** "Basit bir çarpım tablosu öğrenme uygulaması yapacağız. Bu konuda araştırmalar yaparak mevcut çalışmaları da göz önünde bulundurarak iş gerekliliklerini belirle ve bize bir analiz dokümanı hazırla."

---

## 1. Özet

Bu doküman, ilkokul çağındaki çocukların çarpım tablosunu öğrenmesine ve pekiştirmesine yardım edecek **basit** bir uygulamanın iş gereksinimlerini tanımlar. Görev "basit" bir ürün istiyor. Bu yüzden kapsam dar tutuldu: tabloyu görsel desteklerle inceleme, soru-cevap pratiği, anında geri bildirim, hatalı olguların tekrarı ve basit ilerleme takibi. Okul/sınıf yönetimi, çok oyunculu mod, avatar ekonomisi ve bölme işlemi kapsam dışı bırakıldı.

Araştırmadan çıkan başlıca bulgular:

- **Müfredat:** MEB'in Türkiye Yüzyılı Maarif Modeli programında 2. sınıfta 5'e kadar, 3. sınıfta 10'a kadar (10 dahil) çarpım tablosu oluşturulması öngörülüyor ([MEB 2. sınıf ünitesi](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164), [MEB 3. sınıf ünitesi](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264)). Bu nedenle varsayılan aralık 1-10 olarak önerildi.
- **Öğrenme yöntemi:** Sınıf ortamında yapılan bir çalışmada, kartlarla hatırlama pratiği (retrieval practice) çarpım olgularını akıcı hatırlamada sesli ezberden daha etkili bulundu ve etki bir hafta sonra da sürüyordu ([Ophuis-Cox ve ark., 2023](https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/)). Bu yüzden uygulamanın çekirdeği "soru sor, çocuk cevabı hatırlasın" pratiği olarak tanımlandı.
- **Süre baskısı:** Bu konu tartışmalı. Boaler ve ekibi süreli testlerin kaygı yarattığını savunuyor ([Stanford GSE](https://ed.stanford.edu/news/learning-math-without-fear)). 2024'te yapılan bir çalışma ise süreli olmanın kaygıyı anlamlı biçimde artırmadığını buldu ve çocukların birbiriyle değil kendi önceki sonuçlarıyla karşılaştırılmasını önerdi ([Education Week, 2024](https://www.edweek.org/teaching-learning/do-timed-tasks-really-worsen-math-anxiety/2024/08)). Bu nedenle süre **isteğe bağlı ve varsayılan olarak kapalı** tutuldu. Karşılaştırma yalnızca çocuğun kendi geçmişiyle yapılır.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | İhtiyaçlar | Uygulamadaki rolü |
|---|---|---|---|
| **P1: Öğrenen çocuk ("Elif", 8 yaş, 3. sınıf)** | Çarpmayı okulda öğreniyor ve tabloyu ezberlemekte zorlanıyor. Kısa sürede sıkılabiliyor. | Kısa oturumlar, anlaşılır görsel ipuçları, yanlış yaptığında azarlanmadığı hissi, ilerlemesini görebilme. | Birincil kullanıcı |
| **P2: Ebeveyn ("Murat", 38 yaş)** | Çocuğunun evde pratik yapmasını istiyor. Teknik bilgisi orta düzeyde. | Çocuğun hangi tablolarda zorlandığını hızlıca görmek, güvenli ve reklamsız bir ortam. | İkincil kullanıcı: ilerlemeyi izler, ayarları yapar |
| **P3: Öğretmen** | Sınıfına ödev olarak önerebilir. | Müfredatla uyum (2. sınıf: 5'e kadar, 3. sınıf: 10'a kadar). | **Doğrudan kullanıcı değildir.** Öğretmen paneli kapsam dışıdır (bkz. Bölüm 7, Açık soru S-02). |

Hedef yaş aralığı **7-9 yaş (2.-3. sınıf)** varsayılmıştır (bkz. Varsayım V-01). Bu aralık müfredat bilgisine ve benzer ürünlerin hedeflediği yaşa dayanır: Simply Times Tables 2.-4. sınıfları ([App Store](https://apps.apple.com/app/id1472813038)), Hit the Button Maths 6-8 yaşı ([App Store](https://apps.apple.com/gb/app/hit-the-button-maths/id1001893116)), TTRS ise 6 yaş ve üstünü ([TTRS Families](https://ttrockstars.com/families)) hedefliyor.

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakip uygulamalar

| Ürün | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler / riskler | Kaynak |
|---|---|---|---|---|
| **Simply Times Tables** | 1×1'den 12×12'ye 144 olgu. Her tablo için bir "bölge" (kısa ders, hız turu, kontrol noktası). Bilinen tablolar atlanabiliyor. Öğrenilen tabloların karışık tekrarı var. 1 dakikalık hız turu (1-9 veya 1-12). Yanlış cevaptan sonra yardım eden maskot. Eşit gruplar ve ritmik sayma ile görsel ipuçları. Sesli okuma. "Daha sakin" ayar. Ebeveyn alanı, ayrı çocuk profilleri, günlük hedef ve hatırlatma. Yıldız, kupa, olgu başına çıkartma, günlük seri. | Pedagojik akış net: öğren, pratik yap, tekrar et. Görsel ipuçları var. "Reklam yok, hesap yok, veri toplanmıyor" vaadi. Çevrimdışı çalışıyor. | Mağazadaki tek değerlendirme (1 yıldız) cevap gönderilince uygulamanın kapandığını bildiriyor. Bu, kararlılığın kullanıcı deneyimi için kritik olduğunu gösteriyor. Fiyat bilgisi sayfada tutarsız. | [App Store](https://apps.apple.com/app/id1472813038) |
| **Times Tables Rock Stars (TTRS)** | Uyarlamalı pratik: sorular çocuğun ihtiyacına göre otomatik ayarlanıyor. Biri çok oyunculu olmak üzere 8 mod. Süresiz "Jamming" modu. Coin ve avatar özelleştirme. İlerleme istatistikleri ve öğrenme boşluklarının tespiti. Dokunarak gizlenebilen sayaç. Sade ekran ve erişilebilirlik seçenekleri. Aile aboneliği yıllık £7.20 (3 çocuk, 2 yetişkin). | Uyarlamalı tekrar. Kaygılı çocuklar için sayacı gizleme. Ebeveyne yönelik veri. | Abonelik modeli var. Kapsam basit bir ürün için geniş (çok oyunculu, avatar ekonomisi). Okul odaklı bir ekosistem. | [TTRS Families](https://ttrockstars.com/families) |
| **Hit the Button Maths (ücretli uygulama)** | Çarpım ve bölme (10 veya 12'ye kadar), kare sayılar, sayı bağları. 166 oyun modu, rastgele sorular. 1 dakikalık süreli oyunlar **veya** süresiz pratik. Cihaz başına 30 profil. Sonuç geçmişi. Puana göre bronz, gümüş ve altın yıldız ile kupalar. £3.49. Veriler cihazda tutuluyor. | Basit ve hızlı oynanış. Süreli ve süresiz seçenekleri var. Sınıf ortamında tek cihazda çoklu profil. Mağaza puanı 4.0 (255 değerlendirme). | Bir değerlendirmeye göre içerik "sınırlı" görünebiliyor. Çarpım tablosu dışındaki konularla kapsam dağınık. | [App Store](https://apps.apple.com/gb/app/hit-the-button-maths/id1001893116) |

### 3.2 Yöntem bulguları

| Konu | Bulgu | Ürüne etkisi | Kaynak |
|---|---|---|---|
| Hatırlama pratiği | 48 ikinci sınıf öğrencisiyle yapılan çalışmada kartlarla hatırlama pratiği, sesli ezbere göre hem hemen sonra hem bir hafta sonra daha fazla akıcılık kazandırdı. Her iki yöntem de 3 aralıklı oturumda uygulandı. | Çekirdek etkinlik "soru gör, cevabı kendin hatırla" olmalı. Tabloyu yalnızca okumak yeterli değil. | [Ophuis-Cox ve ark., 2023](https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/) |
| Sayı anlayışı ve görsel temsil | Olguların sayı ilişkileri anlaşılarak öğrenilmesi öneriliyor (örn. 7×8 = 10×7 − 2×7). Görsel ve sembolik temsilleri ilişkilendirmenin öğrenmeyi derinleştirdiği belirtiliyor. | İnceleme modunda tekrarlı toplama ve grup görseli gösterilir. Yanlış cevaptan sonra ipucu verilir. | [Stanford GSE, 2015](https://ed.stanford.edu/news/learning-math-without-fear) |
| Müfredat | 2. sınıf: çarpmanın toplamayla ilişkisi, 5'e kadar çarpım tablosu. 3. sınıf: 10'a kadar (10 dahil) çarpım tablosu, 10'a kadar sayıların 9'a kadar rakamlarla çarpılması. | Varsayılan aralık 1-10. Tablo seçimi serbest olduğu için 2. sınıf öğrencisi 1-5 ile sınırlı çalışabilir. | [MEB 2. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164), [MEB 3. sınıf](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264) |
| Süre baskısı ve kaygı | Bir görüşe göre süreli testler öğrencilerin en az üçte birinde kaygı yaratıyor (Boaler). Haberde bu oranın dayandığı veri verilmiyor. Karşı bulguda 113 öğrenciyle süreli ve süresiz koşul arasında kaygı farkı bulunmadı. Öneri: çocuklar birbiriyle karşılaştırılmamalı, kendi önceki süresini geçmeye odaklanmalı. | Sayaç isteğe bağlı ve varsayılan olarak kapalı. Liderlik tablosu yok. Sonuçlar yalnızca çocuğun kendi geçmişiyle karşılaştırılır. | [Stanford GSE](https://ed.stanford.edu/news/learning-math-without-fear), [Education Week, 2024](https://www.edweek.org/teaching-learning/do-timed-tasks-really-worsen-math-anxiety/2024/08) |

### 3.3 Çıkarımlar (ürün için)

1. Rakiplerin hepsinde ortak çekirdek şu: **tablo seçimi, soru-cevap pratiği, anında geri bildirim, ilerleme görünümü.** Bu ürünün MVP'si de bu çekirdekle sınırlı tutulmalı.
2. Zorlanılan olguların daha sık sorulması rakiplerde öne çıkan bir değer (TTRS uyarlamalı pratik, Simply Times Tables karışık tekrar). Basit bir biçimiyle kapsama alındı (FR-07).
3. Süreli ve süresiz modun birlikte sunulması (Hit the Button, TTRS) hem araştırmadaki tartışmayla hem de farklı çocuk profilleriyle uyumlu.
4. Reklamsız olmak ve kişisel veri toplamamak, çocuk ürünlerinde rakiplerin öne çıkardığı bir güven unsuru (Simply Times Tables).

---

## 4. Fonksiyonel gereksinimler

| ID | Gereksinim | Gerekçe / Kaynak |
|---|---|---|
| **FR-01** | Kullanıcı 1'den 10'a kadar tablolar arasından bir veya birden fazla tablo seçebilmelidir. Varsayılan seçim tüm tablolardır (1-10). | MEB 3. sınıf: 10'a kadar tablo. Rakiplerde tablo seçimi standart. |
| **FR-02** | **İnceleme modu:** Seçilen bir tablonun 10 satırı (örn. 3×1 … 3×10) sonuçlarıyla birlikte listelenmelidir. Her satır için tekrarlı toplama gösterimi (örn. 3×4 = 3+3+3+3 = 12) ve eşit grupları temsil eden bir görsel sunulmalıdır. | MEB: çarpmanın toplamayla ilişkilendirilmesi. Görsel ve sembolik temsil (Stanford GSE). Simply Times Tables görsel ipuçları. |
| **FR-03** | **Pratik modu:** Seçilen tablolardan rastgele sıralı "a × b = ?" soruları sorulmalıdır. Bir tur varsayılan olarak 10 sorudan oluşur. Aynı turda aynı soru, yanlış cevap tekrarı dışında art arda gelmemelidir. | Hatırlama pratiği (Ophuis-Cox ve ark., 2023). |
| **FR-04** | Kullanıcı cevabı 0-9 rakam tuşları, silme ve onay tuşlarından oluşan bir ekran tuş takımıyla girebilmelidir. Cevap en fazla 3 haneli olabilir. | Tablo sonuçları en fazla 100. Serbest giriş, çoktan seçmeliye göre gerçek hatırlamayı ölçer. Çoktan seçmeli seçeneği için bkz. S-04. |
| **FR-05** | Her cevaptan sonra anında geri bildirim verilmelidir. Doğru cevapta olumlu bir mesaj gösterilir. Yanlış cevapta doğru sonuç ve FR-02'deki tekrarlı toplama ipucu gösterilir. Geri bildirim dili cezalandırıcı olmamalıdır. | Rakiplerde anında geri bildirim ve yanlıştan sonra yardım standart (Simply Times Tables maskotu). |
| **FR-06** | Tur sonunda bir özet ekranı gösterilmelidir: doğru sayısı / toplam soru, yanlış yapılan olguların listesi ve doğru cevapları. | Rakiplerde puan ve hata incelemesi var (Hit the Button sonuç geçmişi). |
| **FR-07** | **Hata tekrarı:** Turda yanlış cevaplanan olgular aynı turun sonunda en az bir kez daha sorulmalıdır. Sonraki turlarda, son 3 denemesinde yanlış cevaplanmış olgular diğer olgulardan daha sık seçilmelidir (ağırlıklandırma oranı Açık soru S-05'te). | TTRS uyarlamalı pratik, Simply Times Tables karışık tekrar. Aralıklı hatırlama pratiği (Ophuis-Cox ve ark.). |
| **FR-08** | **İsteğe bağlı süre:** Kullanıcı veya ebeveyn pratik turu için bir süre sınırı açabilmelidir (seçenekler: kapalı / 60 sn / 120 sn). Varsayılan "kapalı"dır. Süre açıkken tur sırasında sayaç gizlenebilmelidir. | Süre tartışması (Stanford GSE ↔ Education Week 2024). TTRS sayacı gizleme, Hit the Button süresiz mod. |
| **FR-09** | **İlerleme görünümü:** Her tablo (1-10) için şu bilgiler gösterilmelidir: o tabloda cevaplanan toplam soru sayısı, son 20 cevaptaki doğruluk yüzdesi ve "ustalık" durumu (Öğreniliyor / Ustalaşıldı; eşik Varsayım V-05'te). | Rakiplerde tablo başına istatistik var. TTRS'de ebeveyn öğrenme boşluklarını veriyle tespit ediyor. |
| **FR-10** | **10×10 olgu haritası:** 100 olgunun her biri için durum (hiç sorulmadı / zorlanılıyor / öğrenildi) bir tablo görünümünde, renk **ve** simge ile gösterilmelidir. | Simply Times Tables'ta olgu başına çıkartma ile 144'lük "times square" var. |
| **FR-11** | **Basit ödül:** Turda %90 ve üzeri doğrulukta 1 yıldız verilmeli, bir tablo "Ustalaşıldı" durumuna geçtiğinde tablo rozeti gösterilmelidir. Ödüller yalnızca çocuğun kendi başarısına dayanır. Başka kullanıcılarla karşılaştırma yapılmaz. | Simply Times Tables: 10'da 9 doğruya yıldız. Hit the Button: yıldız ve kupa. Kendi önceki sonucuyla karşılaştırma önerisi (Education Week 2024). |
| **FR-12** | **Kişisel en iyi:** Süreli turlarda çocuğun aynı tablo seçimi için kendi en iyi skoru saklanmalı ve tur sonunda "önceki en iyin" olarak gösterilmelidir. | Education Week 2024: kendi önceki süresini geçmek motive edici. |
| **FR-13** | **Ayarlar:** Ses efektleri açılıp kapatılabilmeli ve "sakin mod" (daha az animasyon ve kutlama) seçilebilmelidir. | Simply Times Tables "calmer setting". TTRS sade ekran seçeneği. |
| **FR-14** | İlerleme verileri sıfırlanabilmelidir. Sıfırlama bir onay adımı gerektirmelidir. | Ebeveyn ihtiyacı (P2). Hit the Button'da misafir skorunu sıfırlama özelliği var. |

---

## 5. Fonksiyonel olmayan gereksinimler

| ID | Kategori | Gereksinim (ölçüt) |
|---|---|---|
| **NFR-01** | Kullanılabilirlik | Okuma bilen 7-9 yaş arası bir çocuk, ilk açılıştan sonra yetişkin yardımı olmadan **en fazla 3 dokunuşla** bir pratik turu başlatabilmelidir. Doğrulama yöntemi: hedef yaş grubundan en az 5 çocukla gözlemli kullanılabilirlik testi, başarı oranı ≥ %80. |
| **NFR-02** | Yanıt süresi | Cevap onaylandıktan sonra geri bildirim **en geç 0,5 saniye** içinde görünmelidir. Sonraki soru en geç 1 saniye içinde gelmelidir (kullanıcı bekleme animasyonu hariç). |
| **NFR-03** | Kararlılık | Pratik turu sırasında uygulama çökmesi oturumların **%0,5'inden az** olmalıdır. Gerekçe: rakipte cevap gönderiminde çökme, ürünün tek değerlendirmesini 1 yıldıza düşürmüş ([App Store](https://apps.apple.com/app/id1472813038)). |
| **NFR-04** | Veri sürekliliği | Uygulama kapatılıp açıldığında ilerleme verileri (FR-09 – FR-12) kaybolmamalıdır. Tamamlanan turların %100'ü kaydedilmiş olmalıdır. |
| **NFR-05** | Gizlilik / çocuk güvenliği | Uygulama ad-soyad, e-posta, konum veya fotoğraf gibi **kişisel veri istememeli**, reklam göstermemeli ve dış bağlantı / sosyal paylaşım içermemelidir. Uygulama içi satın alma varsa ebeveyn doğrulaması olmadan erişilememelidir (bkz. S-06). |
| **NFR-06** | Erişilebilirlik | Doğru/yanlış ve olgu durumu **yalnızca renkle** ifade edilmemeli, metin veya simge ile de belirtilmelidir. Sayı ve soru metni yazı boyutu ayarlanabilir olmalıdır (en az 2 kademe). Tuş takımındaki tuşlar çocuk parmağıyla hatasız basılabilecek büyüklükte olmalıdır. Hedef: kullanılabilirlik testinde yanlış tuşa basma oranı < %5. |
| **NFR-07** | Dil | Arayüz metinlerinin tamamı Türkçe olmalıdır. Metinler 2. sınıf okuma düzeyine uygun, kısa cümlelerden (en fazla ~8 kelime) oluşmalıdır. |
| **NFR-08** | Doğruluk | Uygulamadaki tüm çarpım sonuçları ve gösterilen tekrarlı toplama ifadeleri 1-10 aralığındaki 100 olgunun tamamı için %100 doğru olmalıdır. Yayın öncesi tüm olgular kontrol edilir. |
| **NFR-09** | Oturum süresi | Varsayılan bir tur (10 soru, süresiz) ortalama bir kullanıcı tarafından **yaklaşık 2-4 dakikada** tamamlanabilecek uzunlukta olmalıdır. Amaç kısa, sık oturumlardır. Doğrulama: kullanılabilirlik testinde ölçüm. |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Tablo seçip pratik yapma** (FR-01, FR-03, FR-04)
*Bir öğrenci olarak, yalnızca çalışmak istediğim tabloları seçip soru çözmek istiyorum ki okulda o hafta işlediğimiz tabloya odaklanabileyim.*
- KK1: 3 ve 4 tabloları seçildiğinde turdaki tüm sorularda çarpanlardan biri 3 veya 4'tür.
- KK2: Hiç tablo seçilmeden tur başlatılamaz ve kullanıcıya "En az bir tablo seç" mesajı gösterilir.
- KK3: Tur varsayılan olarak 10 sorudan oluşur.
- KK4: Cevap yalnızca ekran tuş takımıyla girilir ve en fazla 3 hane kabul edilir.

**US-02: Tabloyu anlayarak inceleme** (FR-02)
*Bir öğrenci olarak, bir tabloyu sonuçlarıyla ve resimli olarak görmek istiyorum ki çarpmanın tekrarlı toplama olduğunu anlayayım.*
- KK1: Seçilen tablonun 10 satırı sonuçlarıyla listelenir.
- KK2: Bir satıra dokunulduğunda tekrarlı toplama ifadesi (örn. 4×3 = 4+4+4 = 12) ve eşit grup görseli gösterilir.
- KK3: 1-10 aralığındaki 100 olgunun hepsi için gösterilen ifade doğrudur (NFR-08).

**US-03: Hatamdan öğrenme** (FR-05, FR-07)
*Bir öğrenci olarak, yanlış cevap verdiğimde doğrusunu ve nasıl bulunacağını görmek istiyorum ki aynı hatayı tekrarlamayayım.*
- KK1: Yanlış cevaptan sonra en geç 0,5 sn içinde doğru sonuç ve tekrarlı toplama ipucu gösterilir (NFR-02).
- KK2: Geri bildirim metinleri "yanlış/başarısız" gibi cezalandırıcı ifadeler içermez (örn. "Neredeyse! Doğrusu 56.").
- KK3: Turda yanlış cevaplanan her olgu, aynı turun sonunda en az bir kez tekrar sorulur.
- KK4: Bir olgu son 3 denemesinde yanlış cevaplandıysa, sonraki turlarda diğer olgulara göre daha sık seçilir.

**US-04: Tur sonucunu görme** (FR-06, FR-11)
*Bir öğrenci olarak, tur bitince kaç doğru yaptığımı ve yıldız kazanıp kazanmadığımı görmek istiyorum ki ilerlediğimi hissedeyim.*
- KK1: Özet ekranında "doğru/toplam" ve yanlış yapılan olgular doğru cevaplarıyla listelenir.
- KK2: Doğruluk ≥ %90 ise 1 yıldız verilir, değilse yıldız verilmez ve teşvik edici bir mesaj gösterilir.
- KK3: Özet ekranında başka kullanıcılarla karşılaştırma veya sıralama yer almaz.

**US-05: İsteğe bağlı süreli pratik** (FR-08, FR-12)
*Kendini zorlamak isteyen bir öğrenci olarak, süreli tur oynayıp kendi rekorumu geçmeye çalışmak istiyorum.*
- KK1: Süre ayarı varsayılan olarak "kapalı"dır. 60 sn ve 120 sn seçilebilir.
- KK2: Süre açıkken sayaç tur sırasında gizlenebilir. Gizlendiğinde süre işlemeye devam eder.
- KK3: Süre dolduğunda tur biter ve özet ekranında çocuğun aynı tablo seçimi ve süre için kendi önceki en iyi skoru gösterilir.
- KK4: Yeni skor önceki en iyiden yüksekse "Yeni rekor!" bilgisi gösterilir ve en iyi skor güncellenir.

**US-06: Ebeveynin ilerlemeyi izlemesi** (FR-09, FR-10)
*Bir ebeveyn olarak, çocuğumun hangi tablolarda ve hangi olgularda zorlandığını görmek istiyorum ki ona doğru yerde destek olayım.*
- KK1: İlerleme ekranında 1-10 tablolarının her biri için cevaplanan soru sayısı, son 20 cevaptaki doğruluk yüzdesi ve ustalık durumu görünür.
- KK2: 10×10 olgu haritasında her olgunun durumu hem renk hem simge ile gösterilir (NFR-06).
- KK3: Hiç sorulmamış olgular "hiç sorulmadı" olarak ayrı biçimde işaretlenir.

**US-07: Sakin ve erişilebilir kullanım** (FR-13)
*Kolay dikkati dağılan veya gürültüden rahatsız olan bir öğrencinin ebeveyni olarak, sesleri kapatıp animasyonları azaltmak istiyorum.*
- KK1: Ses kapatıldığında uygulama hiçbir ses çalmaz.
- KK2: Sakin mod açıkken kutlama animasyonları yerini sade bir onay göstergesine bırakır.
- KK3: Ayarlar uygulama yeniden açıldığında korunur (NFR-04).

**US-08: İlerlemeyi sıfırlama** (FR-14)
*Bir ebeveyn olarak, ilerlemeyi sıfırlayabilmek istiyorum ki cihaz başka bir çocuğa geçtiğinde ya da baştan başlamak istediğimizde temiz bir başlangıç yapılsın.*
- KK1: Sıfırlama, onay sorusu ("Tüm ilerleme silinecek. Emin misin?") kabul edilmeden gerçekleşmez.
- KK2: Sıfırlamadan sonra FR-09 – FR-12 kapsamındaki tüm veriler başlangıç durumuna döner. Ayarlar (FR-13) korunur.

---

## 7. Kapsam dışı maddeler

Aşağıdaki maddeler rakiplerde görülse de "basit uygulama" hedefiyle uyumlu olmadığı için ilk sürüme alınmamıştır:

- **Bölme işlemi ve karışık işlemler.** TTRS ve Hit the Button'da var, ancak görev yalnızca çarpım tablosunu kapsıyor.
- **11 ve 12 tabloları (12×12).** Rakipler 12'ye kadar gidiyor. MEB programı 10'a kadar öngörüyor (bkz. S-03).
- **Çok oyunculu mod, liderlik tabloları, sınıf içi sıralama.** Araştırma, çocukların birbirleriyle karşılaştırılmasının olumsuz etkisine işaret ediyor (Education Week 2024).
- **Avatar, sanal para (coin) ve mağaza ekonomisi** (TTRS'deki gibi).
- **Öğretmen/okul paneli, sınıf yönetimi, ödev atama** (bkz. S-02).
- **Kullanıcı hesabı, bulut senkronizasyonu, cihazlar arası aktarım.**
- **Macera haritası, hikâye ve mini oyunlar** (Simply Times Tables'taki gibi).
- **Çok dilli arayüz ve sesli okuma.** Simply Times Tables'ta 10 dil ve sesli okuma var. İlk sürüm yalnızca Türkçe (sesli okuma için bkz. S-07).
- **Bildirim/hatırlatıcı ve günlük seri (streak)** (bkz. S-08).
- **Basılı çalışma kâğıdı ve sertifika üretimi.**

---

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| **V-01** | Birincil hedef kitle 7-9 yaş (2.-3. sınıf) çocuklarıdır. Çocuklar basit Türkçe metinleri okuyabilir. |
| **V-02** | Uygulama ev ortamında, çocuk veya ebeveyn cihazında bireysel kullanım için tasarlanmıştır. Sınıf içi toplu kullanım hedeflenmemiştir. |
| **V-03** | İlk sürümde cihaz başına **tek bir öğrenen profili** vardır. Çoklu profil ihtiyacı S-01'de sorulmuştur. |
| **V-04** | Ürün reklamsız olacaktır. Gelir modeli henüz belirlenmemiştir (bkz. S-06). |
| **V-05** | "Ustalaşıldı" eşiği geçici olarak şöyle tanımlanmıştır: bir tablonun son 20 cevabında ≥ %90 doğruluk **ve** tablodaki 10 olgunun her birinin en az 2 kez doğru cevaplanmış olması. Bu eşik pedagojik bir kaynağa dayanmaz. Paydaş onayı gerekir. |
| **V-06** | Varsayılan tur uzunluğu (10 soru) ve süre seçenekleri (60/120 sn) rakiplerdeki değerlerden esinlenmiştir. Kullanıcı testleriyle ayarlanabilir. |
| **V-07** | NFR'lerdeki sayısal hedefler (0,5 sn, %0,5, %80, %5, 3 dokunuş) analist önerisidir. Dış bir standarda dayanmaz ve paydaş tarafından onaylanmalıdır. |
| **V-08** | Görsel ve ipucu yaklaşımı (tekrarlı toplama, eşit gruplar) MEB programındaki "toplama ile ilişkilendirme" yaklaşımına dayanır. Diğer stratejiler (örn. 7×8 = 10×7 − 2×7 gibi ayrıştırma) ilk sürümde zorunlu değildir. |

---

## 9. Açık sorular

| ID | Soru | Neden önemli |
|---|---|---|
| **S-01** | Aynı cihazı birden fazla çocuk (kardeşler) kullanacak mı? Çoklu profil gerekiyor mu? | Rakiplerde ayrı profiller var (Simply Times Tables, Hit the Button'da 30 profil). Çoklu profil kapsamı ve veri modelini büyütür. |
| **S-02** | Öğretmenler hedef kullanıcı mı? Sınıf veya ödev özellikleri ileride düşünülüyor mu? | Kapsamı belirgin biçimde büyütür. Şu an kapsam dışı. |
| **S-03** | Tablo aralığı 1-10 mu kalmalı, yoksa 11-12 (veya 1-12 seçeneği) de eklenmeli mi? | MEB 10'a kadar öngörüyor. Rakipler 12'ye kadar destekliyor. |
| **S-04** | Cevap girişi yalnızca tuş takımıyla mı olmalı, yoksa küçük yaşlar için çoktan seçmeli seçenek de istenir mi? | Çoktan seçmeli giriş daha kolaydır ama tanımaya dayanır. Serbest giriş gerçek hatırlamayı ölçer. |
| **S-05** | Zorlanılan olguların ne kadar daha sık sorulacağı (ağırlıklandırma oranı) ve "zorlanılıyor" tanımı paydaşça netleştirilmeli mi, yoksa ekip mi belirlemeli? | FR-07'nin test edilebilirliği bu tanıma bağlı. |
| **S-06** | Ürün ücretsiz mi, tek seferlik ücretli mi, uygulama içi satın almalı mı olacak? | Rakipler farklı modeller kullanıyor (tek seferlik, abonelik). NFR-05'teki ebeveyn doğrulaması ihtiyacını belirler. |
| **S-07** | Henüz akıcı okuyamayan çocuklar için soruların sesli okunması gerekli mi? | Simply Times Tables'ta var. Erişilebilirliği artırır ama içerik üretim maliyeti getirir. |
| **S-08** | Günlük hatırlatma veya günlük seri gibi düzenli kullanım teşvikleri istenir mi? | Araştırma aralıklı pratiği destekliyor (Ophuis-Cox ve ark.: 3 aralıklı oturum). Ancak bildirimler çocuk ürünlerinde ebeveyn izni ve gizlilik açısından ek değerlendirme gerektirir. |
| **S-09** | Uygulamanın internet bağlantısı olmadan da tam çalışması bir iş gereksinimi mi? | **İş açısından artısı:** Çocuk her yerde (araba, yolculuk) çalışabilir ve veri cihazdan çıkmadığı için gizlilik vaadi güçlenir (rakipte "Works offline" ve "No data collected" vurgusu var). **Eksisi:** Cihazlar arası ilerleme aktarımı ve ebeveynin uzaktan izlemesi mümkün olmaz. Seçim ve teknik karar TA'ya bırakılmıştır. |
| **S-10** | Hangi cihaz ve platformlar hedefleniyor (tablet, telefon, tarayıcı)? | Kullanılabilirlik testlerinin hangi cihazda yapılacağını belirler. Teknoloji seçimi TA'ya aittir. |
| **S-11** | Çocuklara yönelik ürünler için yasal uyum (örn. KVKK kapsamında çocuk verisi) hukuk birimince değerlendirilecek mi? | NFR-05 veri toplamamayı hedefliyor. Ancak yasal yükümlülükler bu analizde araştırılmadı ve doğrulanmadı. |

---

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | T.C. MEB, İlkokul Matematik Dersi, 2. Sınıf, 3. Tema "İşlemlerden Cebirsel Düşünmeye" | https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164 | MAT.2.2.4 ve MAT.2.2.5 öğrenme çıktıları. Çarpma/bölmenin toplama/çıkarma ile ilişkilendirilmesi. 5'e kadar çarpım tablosunun yüzlük tablo ve işlem tablolarıyla oluşturulması. |
| 2 | T.C. MEB, İlkokul Matematik Dersi, 3. Sınıf, 3. Tema "İşlemlerden Cebirsel Düşünmeye" | https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/264 | MAT.3.2.3 ve MAT.3.2.4. "10'a kadar (10 dâhil) çarpım tablosu oluşturulur". 10'a kadar sayıların 9'a kadar rakamlarla çarpılması. |
| 3 | Ophuis-Cox, Catrysse, Camp (2023), "The effect of retrieval practice on fluently retrieving multiplication facts in an authentic elementary school setting", *Applied Cognitive Psychology* 37(6) | https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/ | 48 ikinci sınıf öğrencisi. Kartlarla hatırlama pratiği, sesli ezbere göre hemen ve bir hafta sonra daha fazla akıcılık sağladı. 3 aralıklı oturum. |
| 4 | Stanford Graduate School of Education, "Research shows the best ways to learn math" (2015) | https://ed.stanford.edu/news/learning-math-without-fear | Boaler ve ekibinin süreli testler ve kaygı görüşü. Sayı anlayışı ve görsel/sembolik temsil önerisi. 7×8 ayrıştırma örneği. |
| 5 | Education Week, "Do Timed Tasks Really Worsen Math Anxiety?" (2024) | https://www.edweek.org/teaching-learning/do-timed-tasks-really-worsen-math-anxiety/2024/08 | 113 öğrenciyle yapılan çalışmada süreli koşul kaygıyı anlamlı biçimde artırmadı. Öğrencileri birbirleriyle karşılaştırmama ve kendi önceki süresini geçmeye odaklanma önerisi. Sınırlılıklar. |
| 6 | App Store, "Simply Times Tables: Practice" | https://apps.apple.com/app/id1472813038 | 144 olgu, bölge yapısı, karışık tekrar, 1 dk hız turu, 10'da 9 doğruya yıldız, görsel ipuçları, sakin ayar, ebeveyn alanı, profiller, reklamsız ve veri toplamama, çevrimdışı çalışma, 2.-4. sınıf hedefi, 1 yıldızlı çökme değerlendirmesi. |
| 7 | Times Tables Rock Stars, "Families" sayfası | https://ttrockstars.com/families | Uyarlamalı pratik, 8 mod (süresiz Jamming), coin ve avatar, ilerleme istatistikleri, gizlenebilir sayaç, sade ekran ve erişilebilirlik, yıllık £7.20 aile aboneliği, 6 yaş ve üstü. |
| 8 | App Store, "Hit the Button Maths" | https://apps.apple.com/gb/app/hit-the-button-maths/id1001893116 | 10/12'ye kadar tablolar, 166 mod, 1 dk süreli veya süresiz pratik, 30 profil, sonuç geçmişi, yıldız ve kupalar, £3.49, 6-8 yaş, 4.0 puan, "sınırlı" görünme eleştirisi. |

---

**Ana oturum için notlar** (dokümanın parçası değildir, kayda alınmamalıdır):

- **Araştırma kuralının uygulanması:** Açılamayan sayfalar (EdSource haberi ve Internet Matters için 403, PMC için tarayıcı doğrulaması, Topmarks Hit the Button web sayfası için 403) kaynak gösterilmedi. Hit the Button'ın web sürümüyle ilgili iddialar dokümana alınmadı.
- **Kural dosyaları:** Bu çalışmada üç kural dosyası, persona tanımının istediği gibi Read ile ayrıca açıldı.
- **Önerilen kayıt yolu:** `/home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v1-run1.md`