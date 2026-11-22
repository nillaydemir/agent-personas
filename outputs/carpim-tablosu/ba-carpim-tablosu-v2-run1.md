# Çarpım Tablosu Öğrenme Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v2
- **Girdi:** Basit bir çarpım tablosu öğrenme uygulaması yapacağız. Bu konuda araştırmalar yaparak mevcut çalışmaları da göz önünde bulundurarak iş gerekliliklerini belirle ve bize bir analiz dokümanı hazırla.

---

## 1. Özet

Bu doküman, ilkokul çağındaki çocuklar için çarpım tablosunu öğretecek ve pekiştirecek bir uygulamanın iş gereksinimlerini tanımlar. Görev metni ürünü "basit" diye tanımlıyor. Bu yüzden kapsam küçük tutuldu. Okul ve öğretmen paneli, çok oyunculu mod, hesap sistemi ve ödeme gibi büyük ürün özellikleri kapsam dışında bırakıldı.

Araştırmanın ana bulguları:

- **Müfredat:** MEB'in 2026 tarihli İlkokul Matematik Dersi Öğretim Programında çarpım tablosu iki sınıfa yayılıyor. 2. sınıfta 5'e kadar (5 dâhil), 3. sınıfta 10'a kadar (10 dâhil) çarpım tablosu oluşturuluyor. Uygulamanın içerik aralığı bu nedenle **1–10 tabloları** olarak belirlendi.
- **Referans sınav:** İngiltere'nin resmî Multiplication Tables Check (MTC) çerçevesi akıcılığı şöyle ölçüyor: 25 soru, `n1 × n2 =` biçimi, soru başına 6 saniye. Bu çerçeve, akıcılık pratiği için kullanılabilecek bir referans sunuyor.
- **Yöntem:** Bir sınıf içi çalışma (48 ikinci sınıf öğrencisi) geri çağırma pratiğinin (flashcard) koro hâlinde tekrar etmeye göre daha güçlü akıcılık kazanımı sağladığını buldu. Bu sonuç, soru-cevap temelli bir pratik modunu destekliyor.
- **Rakipler:** Pazar lideri Times Tables Rock Stars uyarlanabilir sorular, gizlenebilir zamanlayıcı ve erişilebilirlik ayarları sunuyor. İncelenen iki Türkçe uygulama ise mağaza sayfalarında gizlilik beyanı vermiyor.
- **Platform politikaları ve mevzuat:** Google Play Aile politikaları, Apple App Store Kids kuralları ve KVKK birlikte değerlendirildi. Sonuç olarak ilk sürümde **kişisel veri toplanmaması, reklam ve üçüncü taraf analitik kullanılmaması** öneriliyor.

İlk sürüm (MVP) şunları kapsar: tablo seçimi, tabloyu inceleme, soru-cevap pratiği, anında geri bildirim, oturum özeti ve cihazda tutulan olgu bazında ilerleme.

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | İhtiyaçlar | Kaynak / dayanak |
|---|---|---|---|
| **P1 – Öğrenci Elif (7–8 yaş, 2. sınıf)** | Çarpmayı tekrarlı toplama olarak yeni öğreniyor. 1–5 tablolarıyla çalışıyor. | Az sayıda tablo, görsel olarak sade ekran, zaman baskısı olmayan pratik. | MEB 2. sınıf: "Çarpma işleminde 10'a kadar olan sayıları 1, 2, 3, 4 ve 5 ile çarpmaları istenir." [K1] |
| **P2 – Öğrenci Kerem (8–9 yaş, 3. sınıf)** | 1–10 tablolarını ezberlemesi bekleniyor. Bazı olgularda (ör. 7×8) takılıyor. | Hata yaptığı olguları daha sık görmek, hızını artırmak. | MEB 3. sınıf: "10'a kadar (10 dâhil) çarpım tablosu oluşturulur." [K1] |
| **P3 – Veli Ayşe** | Çocuğunun evde kısa süre pratik yapmasını istiyor. | Reklamsız ve güvenli ortam, hangi tabloların öğrenildiğini görme, kurulumun kolay olması. | Platform çocuk politikaları [K7][K8]. Veli ihtiyaçları varsayımdır (bkz. V-01). |

Öğretmen, sınıf yönetimi ihtiyacı olan ayrı bir persona olarak bu sürümde ele alınmadı (bkz. Kapsam dışı maddeler).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Doğrudan rakipler

| Çözüm | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler / riskler | Kaynak |
|---|---|---|---|---|
| **Times Tables Rock Stars (TTRS)**, İngiltere | Firmanın beyanları: 8 oyun modu (çok oyunculu modlar dahil), soruların "her 60 saniyelik oyundan sonra" uyarlanması, avatar ve jeton ödülleri, MTC'yi taklit eden "Soundcheck" modu, öğretmen istatistikleri, anonim "Rock Name"ler, öğretmenin oyun saatini kısıtlayabildiği "curfew". | Uyarlanabilirlik. Erişilebilirlik ayarları ("colour schemes and font size to cater for visual impairments"). Kaygıyı azaltmak için zamanlayıcıyı gizleme. | Okul aboneliği yıllık £116.80, aile aboneliği yıllık £7.20 (en çok 3 çocuk). Türkçe içerik veya MEB uyumu beyanı görülmedi. Tüm iddialar firmanın kendi sayfalarından alındı, bağımsız doğrulanmadı. | [K5], [K6] |
| **Çarpım Tablosu – Times Table** (Hız Yayınları; Speed UP ve Ledu GO ile), iOS | Kolay, Orta ve Zor olmak üzere 3 seviye. 12 soruluk bloklar. "her oyun 30 saniyede çözülmelidir". Yaş sınırı 4+. | Türkçe içerik. Ücretsiz. | Zorunlu süre sınırı var, kapatılabildiğine dair bilgi yok. Gizlilik ayrıntıları mağazada verilmemiş. Değerlendirme sayısı yetersiz. | [K9] |
| **Çarpma Makinesi** (bağımsız geliştirici), iOS ve iPadOS | "Eğitim", "Kendini test" ve "Oyun" bölümleri. Türkçe ve İngilizce. Ücretsiz. 4+. | Öğrenme, test ve oyun ayrımı basit ve anlaşılır. | Geliştirici gizlilik ayrıntısı ve erişilebilirlik bilgisi vermemiş. 1 değerlendirmeyle 4,0 puan. | [K10] |

**Platformun yerleşik çözümü:** Yapılan aramalarda iOS veya Android'in yerleşik bir çarpım tablosu öğrenme aracına rastlanmadı (bkz. V-02).

**Çıkarımlar (BA yorumu):**

- İncelenen iki Türkçe uygulama mağazada gizlilik beyanı vermiyor. Erişilebilirlik ve süre esnekliği konusunda da bilgi yok. "Reklamsız, veri toplamayan, süresi ayarlanabilir, MEB aralığıyla uyumlu" bir ürün bu alanda ayırt edici olabilir.
- TTRS'nin zamanlayıcıyı gizleme ve görsel ayar seçenekleri, kaygılı veya görme güçlüğü olan çocuklar için sektörde örnek alınan bir uygulamadır.

### 3.2 Alan standartları ve düzenlemeler

| Standart / düzenleme | Ürüne etkisi | Kaynak |
|---|---|---|
| **MEB İlkokul Matematik Dersi Öğretim Programı (Türkiye Yüzyılı Maarif Modeli, 2026)** | 2. sınıf, MAT.2.2.4: 10'a kadar sayılar 1–5 ile çarpılır, "5'e kadar (5 dâhil) çarpım tablosu oluşturulur". 1 ve 0'ın çarpmadaki etkisi örneklerle vurgulanır. 3. sınıf, MAT.3.2.3: "Çarpma işleminde 10'a kadar olan sayıları 9'a kadar olan rakamlarla çarpmaları istenir" ve "10'a kadar (10 dâhil) çarpım tablosu oluşturulur." Program, çarpanların yer değiştirmesinin sonucu değiştirmediğinin (değişme özelliği) kavratılmasını da istiyor. | [K1] |
| **İngiltere Multiplication Tables Check (MTC) değerlendirme çerçevesi** (4. sınıf, zorunlu) | 25 soru, `n1 × n2 =` biçimi, soru başına 6 saniye, sorular arasında 3 saniye ara. Sorular 2–12 tablolarından seçiliyor. Çerçeveye göre 6, 7, 8, 9 ve 12 tabloları "the most difficult" olarak belirlenmiş. 6 saniyelik sınır, STA'nın 1124 öğrenciyle 3 farklı süre sınırını denediği araştırmaya dayanıyor. Türkiye'de geçerli değil, akıcılık ölçümü için referans olarak kullanılıyor. | [K2], [K3] |
| **WCAG 2.2** | 1.4.3: metin kontrastı en az 4.5:1. 1.4.4: metin %200'e kadar büyütülebilmeli. 2.2.1: her süre sınırı kapatılabilmeli, ayarlanabilmeli veya uzatılabilmeli. 2.5.8: dokunma hedefi en az 24×24 CSS piksel. | [K11] |
| **6698 sayılı KVKK** | Kişisel veri, "kimliği belirli veya belirlenebilir gerçek kişiye ilişkin her türlü bilgi" olarak tanımlanıyor (md. 3). İşleme kural olarak açık rızaya bağlı (md. 5). Veri sorumlusunun aydınlatma yükümlülüğü var (md. 10). Okunan bölümlerde çocuklara özgü bir madde görülmedi. | [K12] |

### 3.3 Platform ve mağaza politikaları

Görevde platform belirtilmedi. Bu yüzden olası iki mağaza yalnızca iş etkisi düzeyinde incelendi.

| Politika | İş etkisi | Kaynak |
|---|---|---|
| **Google Play Aile (Families) politikaları** | Hedef kitle beyanı Play Console'da yapılmalı. Gizlilik politikası, veri toplama uygulamalarını doğru açıklamalı. Yalnızca çocuklara yönelik uygulamalar AAID veya IMEI gibi kalıcı tanımlayıcıları göndermemeli ve konum izni istememeli. Reklam gösterilecekse yalnızca "Families Self-Certified Ads SDKs" kullanılabilir. **Sonuç:** Reklamsız ve veri toplamayan bir ürün, uyum yükünü en aza indirir. | [K7] |
| **Apple App Store İnceleme Kuralları 1.3 ve 5.1.4** | Kids kategorisindeki uygulamalarda uygulama dışına bağlantı veya satın alma fırsatı yalnızca ebeveyn kapısının (parental gate) arkasında bulunabilir. Kids kategorisindeki uygulamalar "should not include third-party analytics or third-party advertising". Kişisel veri toplanıyorsa gizlilik politikası zorunlu. **Sonuç:** Ayarlar ve sıfırlama gibi veli işlemleri için basit bir ebeveyn kapısı gerekebilir. | [K8] |

### 3.4 Yöntemler (iş açısından)

| Yöntem | Açıklama | İş açısından artıları | İş açısından eksileri | Kaynak |
|---|---|---|---|---|
| **Geri çağırma pratiği (soru-cevap / flashcard)** | Öğrenci cevabı hatırlayıp verir, sonra doğru cevabı görür. | Hollanda'da 4 ikinci sınıfta 48 öğrenciyle yapılan çalışmada (2022, deneklerin kendi içinde karşılaştırma) geri çağırma pratiği, koro hâlinde tekrar etmeye göre hem 5 dakika sonra hem 1 hafta sonra daha güçlü akıcılık artışı sağladı. Hechinger Report muhabirine göre doğru sayısının ön test ortalaması yaklaşık 3 olgu. Son testte bu sayı tekrar grubunda yaklaşık 6'ya, flashcard grubunda yaklaşık 8'e çıktı. | Çalışma küçük ölçekli, kâğıt flashcard ile yapıldı ve dijital uygulamayı test etmedi. Muhabir de zamanlı testlerle karşılaştırma yapılmadığını not ediyor. | [K13], [K14] |
| **Aralıklı tekrar / Leitner sistemi** | Kartlar kutulara ayrılır. Doğru cevaplanan kart daha seyrek tekrar edilen kutuya geçer, yanlış cevaplanan kart ilk kutuya döner. | Zor olgulara daha çok zaman ayrılır. Wikipedia'ya göre toplam çalışma süresini azaltabilir. Kurallar velinin ve çocuğun anlayabileceği kadar basittir. | Kutu sayısı ve aralıklar için tek bir doğru değer yoktur, ürün kararı gerektirir. Kaynak ansiklopedik nitelikte (ikincil). | [K15] |
| **Zamanlı akıcılık pratiği** | Soru başına süre sınırı uygulanır (MTC'de 6 sn). | Hesaplayarak bulmayı değil, hatırlamayı ölçer. MTC çerçevesine göre hız şartı "key to assessing fluent recall". | Kaygı yaratabilir: TTRS kaygıyı azaltmak için zamanlayıcıyı gizleme seçeneği sunuyor. WCAG 2.2.1 her süre sınırının kapatılabilmesini veya ayarlanabilmesini istiyor. | [K3], [K5], [K11] |
| **Uyarlanabilir soru seçimi** | Sorular öğrencinin performansına göre değiştirilir. | TTRS bunu temel özellik olarak sunuyor (firma beyanı: "adapt after every 60 secs of gameplay"). | Uyarlanabilirliğin öğrenmeye etkisine dair bağımsız bir kaynak bu araştırmada açılmadı. Fayda, firma beyanına dayanıyor. | [K6] |
| **Oyunlaştırma (avatar, jeton, liderlik tablosu)** | Pratik ödülle desteklenir. | TTRS'de motivasyon aracı olarak kullanılıyor. | Liderlik tablosu ve çok oyunculu mod sosyal özellik ve veri yükü getirir. "Basit" ürün kapsamını aşar. | [K5], [K6] |

## 4. Fonksiyonel gereksinimler

**M (Must) önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S ve C maddeleri ilk sürümde hedeflenir ama ertelenebilir. Bu sürümde yapılmayacak maddeler Bölüm 7'dedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kullanıcı 1–10 tablolarından bir veya birden fazlasını seçerek çalışabilmelidir. Birden fazla tablo seçildiğinde sorular seçilen tabloların karışımından gelir. | M | MEB 2. ve 3. sınıf aralığı [K1] |
| FR-02 | **İnceleme modu:** Seçilen tablonun tüm çarpımları (n×1 … n×10) cevaplarıyla birlikte tek ekranda gösterilmelidir. | M | MEB'de tablonun "oluşturulması" [K1]. Rakiplerde "Eğitim" bölümü [K10]. |
| FR-03 | **Pratik modu:** Sorular `n1 × n2 = ?` biçiminde tek tek sorulmalıdır. Cevap ekrandaki sayısal tuş takımıyla girilmelidir. Varsayılan oturum 20 sorudur (bkz. V-03). | M | Geri çağırma pratiği [K13]. MTC soru biçimi [K3]. |
| FR-04 | Her cevaptan hemen sonra cevabın doğru mu yanlış mı olduğu gösterilmelidir. Yanlış cevapta doğru sonuç (ör. "7 × 8 = 56") gösterilmelidir. | M | Geri çağırma pratiğinde geri bildirim [K14] |
| FR-05 | Oturum bitiminde bir özet ekranı gösterilmelidir: toplam soru, doğru sayısı ve yanlış cevaplanan olguların listesi. | M | — (ürün gereksinimi) |
| FR-06 | Her çarpım olgusu (ör. 7×8) için doğru ve yanlış cevap geçmişi cihazda saklanmalıdır. Uygulama kapatılıp açıldığında bu geçmiş korunmalıdır. | M | FR-07 ve FR-09'un ön koşulu |
| FR-07 | **Uyarlanabilir tekrar:** Yanlış cevaplanan olgular, doğru cevaplananlardan daha sık sorulmalıdır (Leitner benzeri). Aynı olgu art arda iki kez sorulmamalıdır. Yöntemin ayrıntısı TA'ya bırakılır. | S | Leitner sistemi [K15]. TTRS uyarlanabilirliği [K6]. |
| FR-08 | **İsteğe bağlı süre sınırı:** Pratikte zamanlayıcı varsayılan olarak kapalı olmalıdır. Açıldığında soru başına süre kullanıcı tarafından seçilebilmelidir: en az 3 seçenek, biri 6 sn. Zamanlayıcı her zaman kapatılabilmelidir. | S | WCAG 2.2.1 [K11]. MTC 6 sn [K3]. TTRS zamanlayıcı gizleme [K5]. |
| FR-09 | **İlerleme görünümü:** 10×10'luk bir tablo üzerinde her olgu üç durumdan biriyle gösterilmelidir: "henüz çalışılmadı", "çalışılıyor", "öğrenildi". "Öğrenildi" eşiği V-04'te tanımlıdır. | S | FR-06'nın görünür hâle gelmesi. Veli ihtiyacı (P3). |
| FR-10 | **Sınıf seviyesi ön ayarı:** "2. sınıf" seçildiğinde 1–5 tabloları, "3. sınıf" seçildiğinde 1–10 tabloları tek dokunuşla seçilmelidir. | S | MEB MAT.2.2.4 ve MAT.3.2.3 [K1] |
| FR-11 | **İlerlemeyi sıfırlama:** Tüm ilerleme, onay adımından sonra sıfırlanabilmelidir. Bu işlem basit bir ebeveyn kapısının arkasında olmalıdır. | S | Apple 1.3 ebeveyn kapısı [K8] |
| FR-12 | **0 ile çarpma:** Ayarlardan açılırsa ×0 olguları da (ör. 6×0) sorulmalıdır. | C | MEB'de 0'ın çarpmadaki etkisi [K1] |
| FR-13 | **Basit ödül:** Tamamlanan her oturum için yıldız veya rozet gösterilmelidir. Sanal mağaza ve satın alma olmayacaktır. | C | Oyunlaştırma [K5] |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt | Dayanak |
|---|---|---|---|
| NFR-01 | Arayüz dili Türkçedir. | Tüm ekran metinlerinin %100'ü Türkçe. | Hedef kitle (MEB) |
| NFR-02 | Uygulama kişisel veri toplamaz. Hesap veya giriş gerektirmez. | Ad, doğum tarihi, e-posta, konum ve cihaz tanımlayıcısı istenmez ve cihaz dışına gönderilmez. Mağaza veri beyanında "veri toplanmıyor" beyan edilebilir. | KVKK md. 3, 5, 10 [K12]. Google Play Aile [K7]. Apple 5.1.4 [K8]. |
| NFR-03 | Reklam, üçüncü taraf analitik ve uygulama içi satın alma bulunmaz. | Bu üç unsurdan hiçbiri yok. | Apple 1.3 ve 5.1.4 [K8]. Google Play Aile [K7]. |
| NFR-04 | Metin ve arka plan arasında yeterli kontrast sağlanır. | Tüm metinlerde kontrast oranı ≥ 4.5:1. | WCAG 2.2, 1.4.3 [K11] |
| NFR-05 | Dokunma hedefleri yeterli büyüklüktedir. | Tüm etkileşimli öğeler ≥ 24×24 CSS piksel (WCAG asgarisi; çocuklar için daha büyük hedef: AS-05). | WCAG 2.2, 2.5.8 [K11] |
| NFR-06 | Metin büyütülebilir. | İçerik veya işlev kaybı olmadan %200 büyütme. | WCAG 2.2, 1.4.4 [K11] |
| NFR-07 | Tüm süre sınırları kullanıcı denetimindedir. | Süre sınırı içeren her ekranda sınır kapatılabilir veya ayarlanabilir (FR-08). | WCAG 2.2, 2.2.1 [K11] |
| NFR-08 | Uygulama internet bağlantısı olmadan kullanılabilir. | FR-01–FR-13'teki işlevlerin tamamı çevrimdışı çalışır. | NFR-02 ile tutarlılık. Çocuk kullanımı (V-05). |
| NFR-09 | Geri bildirim hızlıdır. | Cevap onaylandıktan sonra geri bildirim ≤ 0,5 sn içinde görünür. | Varsayım (V-06) |
| NFR-10 | İlerleme verisi kalıcıdır. | Uygulama kapatılıp yeniden başlatıldığında ve cihaz yeniden başlatıldığında ilerleme kaybı %0. | FR-06 |
| NFR-11 | Kullanım kolaydır. | 7–8 yaşındaki bir çocuk, açılış ekranından ilk pratik sorusuna yardım almadan en çok 3 dokunuşla ulaşır. Bu ölçüt en az 5 çocukla yapılacak kullanılabilirlik testinde doğrulanır. | Varsayım (V-07) |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Tablo seçip pratik yapma** (FR-01, FR-03, FR-04, FR-05)
*3. sınıf öğrencisi olarak 6 ve 7 tablolarını seçip soru-cevapla pratik yapmak istiyorum, böylece bu tabloları ezberleyebilirim.*
- **KK-1:** Verilen: tablo seçim ekranı açık. Eğer: 6 ve 7 seçilip pratik başlatılırsa. O zaman: tüm sorular 6× veya 7× olgularından gelir.
- **KK-2:** Verilen: pratik sürüyor. Eğer: yanlış cevap girilirse. O zaman: yanlış işareti ve doğru sonuç (ör. "7 × 8 = 56") gösterilir.
- **KK-3:** Verilen: varsayılan oturum. Eğer: 20. soru cevaplanırsa. O zaman: toplam soru, doğru sayısı ve yanlış cevaplanan olgular özet ekranında listelenir.

**US-02: Tabloyu inceleme** (FR-02)
*2. sınıf öğrencisi olarak pratikten önce 4 tablosunun tamamını görmek istiyorum, böylece sonuçları önce tanıyabilirim.*
- **KK-1:** Verilen: 4 tablosu seçili. Eğer: inceleme modu açılırsa. O zaman: 4×1'den 4×10'a kadar 10 satır cevaplarıyla tek ekranda görünür.

**US-03: Zorlandığım olguları daha sık görme** (FR-06, FR-07)
*Öğrenci olarak yanlış yaptığım çarpımların bana daha sık sorulmasını istiyorum, böylece zayıf olduğum yerleri çalışabilirim.*
- **KK-1:** Verilen: 7×8 son oturumda yanlış, 2×3 doğru cevaplanmış. Eğer: aynı tablolarla yeni bir oturum başlatılırsa. O zaman: oturumda 7×8, 2×3'ten daha sık sorulur.
- **KK-2:** Verilen: herhangi bir oturum. Eğer: bir olgu sorulursa. O zaman: hemen ardından gelen soru aynı olgu olmaz.
- **KK-3:** Verilen: geçmişi olan bir kullanıcı. Eğer: uygulama kapatılıp açılırsa. O zaman: olgu geçmişi aynen korunur.

**US-04: Süre baskısı olmadan çalışma** (FR-08, NFR-07)
*Zaman baskısında gerilen bir öğrenci olarak süresiz pratik yapmak, hazır olduğumda süreyi açmak istiyorum.*
- **KK-1:** Verilen: ilk kurulum. Eğer: pratik başlatılırsa. O zaman: ekranda zamanlayıcı yoktur ve soru süresiz bekler.
- **KK-2:** Verilen: ayarlar ekranı. Eğer: zamanlayıcı açılıp 6 sn seçilirse. O zaman: her soru için 6 sn geri sayım gösterilir ve süre dolunca soru yanlış veya cevapsız olarak kaydedilir.
- **KK-3:** Verilen: zamanlayıcı açık. Eğer: kullanıcı zamanlayıcıyı kapatırsa. O zaman: bir sonraki sorudan itibaren süre sınırı uygulanmaz.

**US-05: Çocuğun ilerlemesini görme** (FR-06, FR-09)
*Veli olarak çocuğumun hangi çarpımları öğrendiğini görmek istiyorum, böylece ona hangi tablolarda destek olacağımı bilirim.*
- **KK-1:** Verilen: birkaç oturum tamamlanmış. Eğer: ilerleme ekranı açılırsa. O zaman: 10×10 tabloda her olgu "henüz çalışılmadı", "çalışılıyor" veya "öğrenildi" durumuyla görünür.
- **KK-2:** Verilen: bir olgu son 3 denemede doğru cevaplanmış (V-04). Eğer: ilerleme ekranı açılırsa. O zaman: olgu "öğrenildi" görünür.

**US-06: Sınıfa uygun hızlı başlangıç** (FR-10)
*Veli olarak çocuğumun sınıfını seçip uygun tabloları tek dokunuşla ayarlamak istiyorum.*
- **KK-1:** Verilen: tablo seçim ekranı. Eğer: "2. sınıf" seçilirse. O zaman: yalnızca 1–5 tabloları seçili olur.
- **KK-2:** Eğer: "3. sınıf" seçilirse. O zaman: 1–10 tabloları seçili olur.

**US-07: İlerlemeyi sıfırlama** (FR-11)
*Veli olarak cihazı başka bir çocuğa verirken ilerlemeyi sıfırlamak istiyorum, ama çocuğun bunu yanlışlıkla yapmasını istemiyorum.*
- **KK-1:** Verilen: ayarlar ekranı. Eğer: "İlerlemeyi sıfırla" seçilirse. O zaman: önce ebeveyn kapısı, sonra onay adımı gösterilir.
- **KK-2:** Eğer: ebeveyn kapısı geçilip onay verilirse. O zaman: tüm olgu geçmişi silinir ve ilerleme ekranındaki tüm olgular "henüz çalışılmadı" görünür.

**US-08: Çarpmada 0'ı çalışma** (FR-12)
*2. sınıf öğrencisi olarak 0 ile çarpmayı da çalışmak istiyorum.*
- **KK-1:** Verilen: ayarlarda "0 ile çarpma" açık. Eğer: pratik başlatılırsa. O zaman: soru havuzunda seçili tabloların ×0 olguları da bulunur.

**US-09: Oturum ödülü** (FR-13)
*Öğrenci olarak oturumu bitirdiğimde bir ödül görmek istiyorum, böylece yeniden çalışmaya istekli olurum.*
- **KK-1:** Eğer: bir oturum tamamlanırsa. O zaman: özet ekranında bir yıldız veya rozet gösterilir. Satın alma veya mağaza bağlantısı bulunmaz.

## 7. Kapsam dışı maddeler

| # | Madde | Neden |
|---|---|---|
| KD-01 | 11 ve 12 tabloları | MEB programı 10'a kadar (10 dâhil) tabloyu kapsıyor [K1]. 12'ye kadar aralık İngiltere MTC'ye özgü [K3]. Bkz. AS-01. |
| KD-02 | Bölme pratiği | Görev "çarpım tablosu" ile sınırlı. TTRS bölmeyi de içeriyor, ancak bu kapsam genişlemesi olur. |
| KD-03 | Öğretmen paneli, sınıf yönetimi, sınıf raporları | Okul ürünü ölçeğinde (TTRS örneği [K6]). "Basit" ürün tanımına uymuyor. |
| KD-04 | Kullanıcı hesabı, bulut eşitleme, birden çok cihaz | Kişisel veri işlemeyi gerektirir (KVKK [K12], platform politikaları [K7][K8]). |
| KD-05 | Çok oyunculu mod, liderlik tablosu, sohbet | Sosyal özellikler çocuk politikalarında ek yükümlülük getirir [K7]. Kapsamı büyütür. |
| KD-06 | Reklam ve uygulama içi satın alma | NFR-03 ile çelişir. Gelir modeli AS-03'te sorulur. |
| KD-07 | Avatar ve sanal mağaza gibi gelişmiş oyunlaştırma | Basit ödül (FR-13) yeterli görüldü. |
| KD-08 | Birden fazla çocuk profili | Aile kullanımında yararlı olabilir (TTRS aile aboneliğinde 3 çocuk hesabı var [K5]). İlk sürüm için AS-04'te soruluyor. |
| KD-09 | Sesli soru okuma ve konuşarak cevap verme | Görevde istenmedi. Erişilebilirlik açısından ileride değerlendirilebilir. |
| KD-10 | Resmî MTC simülasyonu (25 soru, 6 sn, 3 sn ara) | Türkiye'de geçerli bir sınav değil. Benzer pratik FR-08 ile sağlanabiliyor. |

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| V-01 | Birincil kullanım ortamı ev, birincil kullanıcı 2. ve 3. sınıf öğrencisi. Veli ihtiyaçları (reklamsızlık, ilerleme takibi) kullanıcı araştırmasıyla doğrulanmadı. |
| V-02 | Aramalarda iOS veya Android'in yerleşik bir çarpım tablosu öğrenme aracına rastlanmadı. Bu nedenle "platformun yerleşik çözümü" başlığı rakip tablosuna eklenemedi. Böyle bir çözümün olmadığı ayrıca kesin olarak doğrulanmadı. |
| V-03 | Varsayılan oturum uzunluğu 20 soru olarak seçildi. Dayanak: MTC'nin 25 soruluk ve 5 dakikadan kısa biçimi [K2] ile sınıf çalışmasındaki 5 dakikalık pratik seansları [K14]. 20 sayısı bu kaynaklardan türetilmiş bir ürün tercihidir, kaynakta yazmaz. |
| V-04 | "Öğrenildi" eşiği: olgunun son 3 denemede doğru cevaplanması. Bu bir ürün varsayımıdır, kaynağa dayanmaz. Zamanlayıcı açıkken hız da eşiğe dahil edilebilir (AS-06). |
| V-05 | Çocukların internet erişimi her zaman olmayabilir. Çevrimdışı çalışma (NFR-08) bu varsayıma ve veri toplamama hedefine dayanır. |
| V-06 | 0,5 sn geri bildirim süresi (NFR-09) kaynağa dayanmayan bir kullanılabilirlik hedefidir. |
| V-07 | "3 dokunuşla ilk soru" ve "5 çocukla test" (NFR-11) kaynağa dayanmayan ürün hedefleridir. |
| V-08 | Görevde platform belirtilmedi. Mağaza politikaları (Bölüm 3.3) yalnızca iş etkisi düzeyinde ele alındı. Platform ve teknoloji seçimi TA'ya bırakıldı. |
| V-09 | Uyarlanabilirliğin öğrenmeye katkısı yalnızca TTRS'nin kendi beyanıyla destekleniyor. FR-07'nin S önceliği, bağımsız bir kanıta değil Leitner yönteminin mantığına [K15] dayanıyor. |
| V-10 | KVKK'nın okunan maddelerinde (md. 1–12) çocuklara özgü bir hüküm görülmedi. Bu, çocuk verisi için ek bir düzenleme olmadığı anlamına gelmez. Hukuki görüş alınmadı. |

## 9. Açık sorular

| ID | Soru | Etkilenen maddeler |
|---|---|---|
| AS-01 | Tablolar 1–10 ile mi sınırlı kalmalı, yoksa 11 ve 12 de eklenmeli mi? | FR-01, KD-01 |
| AS-02 | Hedef platform nedir: iOS, Android, web veya bunların birkaçı? Mağazada "Kids" veya "Aile" kategorisi hedefleniyor mu? | NFR-02, NFR-03, FR-11 |
| AS-03 | Gelir modeli nedir: ücretsiz, tek seferlik ücret veya abonelik? Reklam veya satın alma isteniyorsa NFR-03 ve çocuk politikaları yeniden değerlendirilmelidir. | NFR-03, KD-06 |
| AS-04 | İlk sürümde tek cihazda birden fazla çocuk profili gerekli mi? | KD-08, FR-06 |
| AS-05 | Çocuklar için WCAG asgarisinin (24×24 CSS piksel) üzerinde bir dokunma hedefi büyüklüğü belirlenmeli mi? | NFR-05 |
| AS-06 | "Öğrenildi" tanımına hız da dahil edilmeli mi (ör. zamanlayıcı açıkken 6 sn içinde doğru)? | FR-09, V-04 |
| AS-07 | Uygulama öğretmenler tarafından sınıfta da kullanılacak mı? Kullanılacaksa hangi asgari ihtiyaçlar karşılanmalı? | KD-03 |
| AS-08 | Pedagojik doğrulama için bir sınıf öğretmeninden veya matematik eğitimi uzmanından içerik incelemesi alınacak mı? | FR-02, FR-10 |

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | MEB – İlkokul Matematik Dersi Öğretim Programı (1, 2, 3 ve 4. Sınıflar), Türkiye Yüzyılı Maarif Modeli, 2026 | https://mufredat.meb.gov.tr/Dosyalar/20268149102521-Matematik%20(1-4)%20D%C3%96P.pdf | 2. sınıf MAT.2.2.4: "Çarpma işleminde 10'a kadar olan sayıları 1, 2, 3, 4 ve 5 ile çarpmaları istenir … 5'e kadar (5 dâhil) çarpım tablosu oluşturulur. Çarpma işleminde 1 ve 0'ın etkisi çeşitli örnekler verilerek vurgu yapılır" (s. 69). 3. sınıf MAT.3.2.3: "Çarpma işleminde 10'a kadar olan sayıları 9'a kadar olan rakamlarla çarpmaları istenir … 10'a kadar (10 dâhil) çarpım tablosu oluşturulur" (s. 103). Tema ve sınıf dağılımı (s. 10–11). |
| K2 | GOV.UK – Multiplication tables check assessment framework | https://www.gov.uk/government/publications/multiplication-tables-check-assessment-framework | Çerçevenin 4. sınıf MTC için olduğu, 2019'dan itibaren geçerli olduğu, 2026–2027 düzenlemelerinin değişmediği bilgisi. Çerçeve PDF'ine bağlantı. |
| K3 | STA – Multiplication tables check assessment framework (2018, 2022'de güncellendi) | https://assets.publishing.service.gov.uk/media/62384958e90e0779a5e700c3/2018_MTC_assessment_framework_PDFA_updated_for_2022.pdf | "Each form consists of 25 questions worth one mark each." "Pupils will have 6 seconds to enter a response." Sorular arasında 3 sn ara. Sorular 2–12 tablolarından (121 olgu) seçiliyor, 1 tablosu dahil değil. "emphasis on the 6, 7, 8, 9 and 12 multiplication tables because these have been determined to be the most difficult". 6 sn sınırı, STA'nın 1124 öğrenciyle 3 süre sınırını denediği araştırmaya dayanıyor. "The requirement to answer at speed is key to assessing fluent recall". Uygulama 5 dakikadan kısa sürüyor. |
| K4 | — | — | Kullanılmadı (numara boş bırakıldı). |
| K5 | Times Tables Rock Stars – Families | https://ttrockstars.com/families | Firma beyanı: "eight different game modes … including multiplayer modes", "automatically adapt to each child's unique learning needs", "De-clutter the screen or hide the timer to help minimise anxiety", ödül puanları ve avatar, aile aboneliği "£7.20 PER YEAR" (3 çocuk hesabına kadar), "aged 6 years and upwards". |
| K6 | Times Tables Rock Stars – Schools | https://ttrockstars.com/schools/ | Firma beyanı: "colour schemes and font size to cater for visual impairments", "hide the timer (to help reduce maths anxiety)", "set curfews", "anonymous Rock Names", "adapt after every 60 secs of gameplay", Soundcheck'in MTC'yi taklit etmesi, öğretmen istatistikleri, "£116.80 per year". |
| K7 | Google Play Console Yardım – Families policies | https://support.google.com/googleplay/android-developer/answer/9893335?hl=en | "Ensure your privacy policy accurately explains your app's data collection and handling practices". "Don't collect personal data persistent identifiers (like AAID or IMEI) from children". Yalnızca çocuklara yönelik uygulamalarda konum izni yasağı. "Only use Google Play Families Self-Certified Ads SDKs". Hedef yaş grubunun Play Console'da beyan edilmesi. |
| K8 | Apple – App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 1.3: Kids kategorisinde dış bağlantı ve satın alma yalnızca ebeveyn kapısının arkasında. "Apps in the Kids Category should not include third-party analytics or third-party advertising." 5.1.4: çocuk verisinde COPPA, GDPR gibi yasalara uyum. Kişisel veri toplayan uygulamalar için gizlilik politikası zorunlu. |
| K9 | App Store – Çarpım Tablosu-Times Table | https://apps.apple.com/us/app/id1513006155 | Kolay, Orta ve Zor olmak üzere 3 seviye, 12 soruluk bloklar, "her oyun 30 saniyede çözülmelidir", 4+, ücretsiz, geliştirici gizlilik ayrıntısı vermemiş, yetersiz değerlendirme. |
| K10 | App Store – Çarpma Makinesi | https://apps.apple.com/us/app/id1458975066 | Eğitim, kendini test ve oyun bölümleri, Türkçe ve İngilizce, ücretsiz, 4+, geliştirici gizlilik ve erişilebilirlik bilgisi vermemiş, 1 değerlendirmeyle 4,0 puan. |
| K11 | W3C – Web Content Accessibility Guidelines (WCAG) 2.2 | https://www.w3.org/TR/WCAG22/ | 1.4.3: "contrast ratio of at least 4.5:1". 1.4.4: metin "up to 200 percent" büyütülebilir. 2.2.1: süre sınırı kapatılabilir, ayarlanabilir veya uzatılabilir olmalı ("The user is allowed to turn off the time limit before encountering it"). 2.5.8: "at least 24 by 24 CSS pixels". |
| K12 | Mevzuat Bilgi Sistemi – 6698 sayılı Kişisel Verilerin Korunması Kanunu | https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf | md. 3/d: "Kişisel veri: Kimliği belirli veya belirlenebilir gerçek kişiye ilişkin her türlü bilgi". md. 5/1: "Kişisel veriler ilgili kişinin açık rızası olmaksızın işlenemez." md. 10: aydınlatma yükümlülüğü. md. 12: veri güvenliği. (md. 1–12 okundu.) |
| K13 | Open Universiteit – The effect of retrieval practice on fluently retrieving multiplication facts in an authentic elementary school setting (Ophuis-Cox, Catrysse, Camp; Applied Cognitive Psychology, 37(6), 2023) | https://research.ou.nl/en/publications/the-effect-of-retrieval-practice-on-fluently-retrieving-multiplic/ | 48 ikinci sınıf öğrencisi. Deneklerin kendi içinde karşılaştırma: flashcard ile geri çağırma pratiği ve koro hâlinde tekrar. Ön test, 5 dakika sonra ve 1 hafta sonra son test. Bulgu: "retrieval practice led to a stronger short‐term and long‐term increase in the fluency of retrieving multiplication facts". |
| K14 | The Hechinger Report – Proof Points: Flashcards prevail over repetition in memorizing multiplication tables (Jill Barshay) | https://hechingerreport.org/proof-points-flashcards-prevail-over-repetition-in-memorizing-multiplication-tables/ | Muhabirin aktarımı: Hollanda'da 4 ikinci sınıf, 2022. Haftada 3 kez 5'er dakika pratik. Doğru olgu sayısı ön testte yaklaşık 3, son testte tekrar grubunda yaklaşık 6, flashcard grubunda yaklaşık 8. Bir hafta sonra flashcard grubunun üstünlüğü sürüyor. Muhabire göre küçük bir çalışma. Zamanlı testlerle karşılaştırma yapılmamış. |
| K15 | Wikipedia – Leitner system | https://en.wikipedia.org/wiki/Leitner_system | Kartlar kutulara ayrılır. Doğru cevaplananlar daha seyrek tekrar edilen kutuya geçer, yanlışlar ilk kutuya döner. "cards are reviewed at increasing intervals". Toplam çalışma süresini azaltabileceği belirtiliyor. (İkincil kaynak.) |

**Açılamayan veya kullanılmayan kaynaklar:**

- Aşağıdaki sayfalar açılamadı ve kaynak gösterilmedi. Bu sayfalardaki bilgiler doğrulanmamış sayılır:
  - MEB tymm.meb.gov.tr 2024 program PDF'i (HTTP 500)
  - Internet Matters TTRS sayfası (403)
  - ExploreLearning Reflex sayfaları (403)
  - Topmarks "Hit the Button" (403)
  - Google Play "Tumla" sayfası (içerik kesik)
- Common Sense Education Reflex incelemesi açıldı, ancak sayfada inceleme yerine incelemelerin durdurulduğuna dair bir açıklama vardı. Bu yüzden kullanılmadı.
- EdWeek makalesi yalnızca arama sonucunda görüldü, açılmadığı için kaynak gösterilmedi.