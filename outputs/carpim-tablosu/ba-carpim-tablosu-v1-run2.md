# Çarpım Tablosu Öğrenme Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** Basit bir çarpım tablosu öğrenme uygulaması yapacağız. Bu konuda araştırmalar yaparak mevcut çalışmaları da göz önünde bulundurarak iş gerekliliklerini belirle ve bize bir analiz dokümanı hazırla.

---

## 1. Özet

Bu doküman, ilkokul çağındaki çocukların çarpım tablosunu öğrenmesi ve pekiştirmesi için yapılacak **basit** bir uygulamanın iş gereksinimlerini tanımlar.

Araştırmadan çıkan üç ana bulgu var:

1. **Hatırlamaya dayalı alıştırma, ezberlemeden daha etkili.** Çocuğun cevabı kendisinin hatırlamaya çalıştığı alıştırmalar (geri çağırma pratiği), tabloyu yüksek sesle tekrar etmekten daha etkili bulunmuş. Bu alıştırmalar kısa oturumlar halinde güne yayıldığında ve hemen geri bildirim verildiğinde etkisi artıyor ([KQED MindShift](http://www.kqed.org/mindshift/62663/flash-cards-prevail-over-repetition-in-memorizing-multiplication-tables)).
2. **Süre baskısı bazı çocuklarda kaygı yaratabilir.** Süreli testlerin bazı çocuklarda matematik kaygısına yol açabileceği belirtiliyor ([YouCubed](https://www.youcubed.org/evidence/fluency-without-fear/)). Rakip uygulamaların bir kısmı da süreyi isteğe bağlı sunuyor.
3. **Rakiplerin ortak özellikleri belli.** Mevcut uygulamalar sıralı ve karışık soru, süreli ve süresiz mod, rozet ve oyunlaştırma, ilerleme takibi sunuyor. Ücretli kilit açma veya abonelik de yaygın.

Önerilen kapsam bilerek dar tutuldu:

- tablo seçimi,
- tabloyu inceleme,
- cevabı yazarak alıştırma,
- anında geri bildirim,
- yanlış yapılan soruların tekrar sorulması,
- basit ilerleme görünümü.

Çok oyunculu mod, hesap ve sınıf yönetimi, abonelik gibi özellikler kapsam dışında bırakıldı.

## 2. Hedef kullanıcılar ve persona'lar

**Birincil kullanıcı: Öğrenen çocuk.** MEB 2. sınıf programında çarpım tablosu **5'e kadar (5 dâhil)** oluşturuluyor ([TYMM 2. Sınıf Matematik, 3. Tema](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164)). Bu nedenle ana hedef kitle 2. sınıftan başlayan ilkokul öğrencileri olarak kabul edilmiştir (bkz. Varsayımlar).

**İkincil kullanıcı: Ebeveyn veya öğretmen.** Çocuğun ilerlemesine bakar ve ayarları yapar.

| Persona | Profil | İhtiyaç | Zorluk |
|---|---|---|---|
| **Elif (8, 2. sınıf)** | 2'ler–5'ler tablolarını yeni öğreniyor. Okuma hızı sınırlı. | Kısa, eğlenceli ve baskısız alıştırma; doğru mu yanlış mı hemen görmek. | Süre baskısında hata yapıyor, yanlış yapınca motivasyonu düşüyor. |
| **Can (9, 3. sınıf)** | Tabloların çoğunu biliyor ama 6'lar–9'larda takılıyor. | Bilmediği soruları hedefleyen tekrar; hangi soruları öğrendiğini görmek. | Bildiği soruları tekrar tekrar çözmekten sıkılıyor. |
| **Ayşe Hanım (ebeveyn)** | Çocuğuna evde destek olmak istiyor. Teknik bilgisi orta. | Çocuğun hangi tablolarda zorlandığını hızlıca görmek; reklam ve veri toplama olmaması. | Zamanı kısıtlı; karmaşık ayar ekranları istemiyor. |

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Uygulamalar

| Çözüm | Öne çıkan özellikler | Güçlü yönler | Zayıf yönler / not | Kaynak |
|---|---|---|---|---|
| **Times Tables Rock Stars** | Çocuğa göre uyarlanan soru tabanlı oyunlar; çok oyunculu canlı oyunlar ve skor tabloları; ödül olarak rozet, çıkartma ve kupa; öğretmen ve ebeveyn için ilerleme takibi; akıcılığa ne kadar yakın olunduğunu gösteren ısı haritası; okul dışında varsayılan 20 dakikalık günlük oyun sınırı. | Uyarlanabilir sorular, görsel ilerleme haritası, dijital sağlık sınırı. | Abonelikle çalışıyor; kapsamı okul ölçeğinde. | [ttrockstars.com](https://ttrockstars.com/) |
| **Multiplication Tables! (3MB Solutions)** | 2–20 arası tablolar; sıralı veya karışık mod; çarpma ve bölme; isteğe bağlı süreli test modu; çoktan seçmeli ve tuş takımıyla cevap; rozet ve jeton; test geçmişi ve istatistik; kapatılabilir ses efektleri; doğrulamayla korunan ebeveyn ayarları. 6–8 yaş için işaretli, yaklaşık 5,9 bin değerlendirmede 4,8 puan. | Süre isteğe bağlı, iki cevap biçimi var, ebeveyn ayarları korumalı. | Aralıklı tekrar yok; bir kullanıcı bunu talep etmiş. Erişilebilirlik özellikleri belirtilmemiş. | [App Store](https://apps.apple.com/us/app/id1451984868) |
| **Times Tables Multiplication (Learning Apps)** | 1–10 arası tablolar; tabloları adım adım ve sesli anlatım dinleyerek öğrenme; ezberi kontrol eden kısa sınavlar. Tüm tabloların kilidi 2,99 $'lık uygulama içi satın almayla açılıyor. 106 değerlendirmede 3,9 puan. | Sade arayüz; dinleyerek öğrenme. | Tabloların kilidi ücretle açılıyor; bir kullanıcı 11–20 tablolarını istemiş. Erişilebilirlik bilgisi yok. | [App Store](https://apps.apple.com/us/app/id1089191129) |
| **Times Tables and Friends** | Oyun, öğrenme, meydan okuma, antrenman ve çok oyunculu olmak üzere beş mod; 12×12'ye kadar 12 seviye. Meydan okuma modu zamanlı; antrenman modu zayıf olgulara odaklanan süresiz çalışma. Rozet, başarı panosu, 12 dilde sesli anlatım. Fiyatı 4,99 $; 6–8 yaş. | Süresiz hedefli çalışma modu; çok dilli ses. | Oyun modunda ikiden fazla yanlış oyunu bitiriyor. Bir kullanıcı ses tanımanın yanlış sayıyı da kabul ettiğini bildirmiş. Son güncelleme 2019. | [App Store](https://apps.apple.com/us/app/-/id1341586204) |
| **Multiplication: Times Tables x (Mobilaxy)** | 20×20'ye kadar; oyun (20 dünya), meydan okuma (zamanlı, skor tablosu), iki oyunculu, öğrenme (çok duyulu), antrenman (zorlanılan sorular) modları. Abonelik: aylık 3,99 $, yıllık 9,99 $. | Zorlanılan sorulara hedefli antrenman. | Abonelik modeli; yalnızca 6 değerlendirme; ayrıntılı ilerleme paneli yok. | [App Store](https://apps.apple.com/us/app/-/id1498747525) |

### 3.2 Öğrenme yöntemleri

- **Hatırlayarak alıştırma (geri çağırma pratiği).** Hollanda'da 48 ikinci sınıf öğrencisiyle yapılan çalışmada iki yöntem karşılaştırıldı: kartla hatırlayarak çalışma ve tabloyu yüksek sesle tekrar etme. Her grup haftada 3 kez 5'er dakika çalıştı.
  - Kartla çalışanlar ortalama 8, tekrar edenler 6 doğruya ulaştı (başlangıç: 3).
  - Kartla çalışanların üstünlüğü bir hafta sonra da sürdü.
  - Çalışma küçük ölçekli. Hatırlamanın ardından hemen geri bildirim ve yeni deneme verilmesi etkili bulunuyor.
  - Kaynak: [KQED MindShift](http://www.kqed.org/mindshift/62663/flash-cards-prevail-over-repetition-in-memorizing-multiplication-tables)
- **Güne yayılmış çalışma ve geri bildirim.** Yetişkinlerle yapılan bir aritmetik olgu deneyinde en çok kazanımı, üç ayrı günde yapılan ve her cevaba geri bildirim verilen hatırlama oturumları sağladı. Yazarlar, çalışmanın günlere yayılmasının kazanıma katkı yapmış olabileceğini belirtiyor ([Schrift vd., 2022, npj Science of Learning](https://pmc.ncbi.nlm.nih.gov/articles/PMC9385657)). Katılımcılar yetişkin olduğundan sonuç çocuklara doğrudan genellenemez.
- **Süre baskısı ve kaygı.**
  - Kaynağa göre süreli testler matematik kaygısını tetikleyebiliyor; öğrencilerin yaklaşık üçte biri bu kaygıyı yaşıyor ve kaygı 5 yaşında bile görülebiliyor.
  - Önerilen yaklaşım: hıza vurgu yapmamak; sayı hissini geliştirmek (örn. 7×8'i 70−14 üzerinden bulmak); stratejiye dayalı öğrenme.
  - Kaynak: [YouCubed: Fluency Without Fear](https://www.youcubed.org/evidence/fluency-without-fear/)
  - Bu kaynak ezbere dayalı tekrar karşıtı bir bakış sunuyor; KQED kaynağındaki hatırlama pratiği bulgusuyla gerilim içinde. Bu doküman iki görüşü şöyle uzlaştırır: hatırlama pratiği yapılır, ancak süre baskısı olmadan ve süre varsayılan olarak kapalı.
- **Müfredat uyumu.** MEB 2. sınıf öğrenme çıktıları ([TYMM](https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164)):
  - MAT.2.2.4: çarpmayı toplama ile ilişkilendirme.
  - MAT.2.2.5: çarpma sonuçlarını tahmin ve zihinden işlemle muhakeme etme.
  - Çarpım tablosu 5'e kadar (5 dâhil) oluşturuluyor.
  - 3. sınıf ve sonrasında beklenen tablo aralığı bu oturumda doğrulanamadı (bkz. Açık sorular).

### 3.3 Çıkarımlar

1. Temel değer hatırlayarak alıştırma ve anında geri bildirimden geliyor; oyunlaştırma ikincil.
2. Süre varsayılan olarak kapalı, isteğe bağlı açık olmalı.
3. Rakiplerde yaygın ama basit üründe gereksiz olanlar kapsam dışında: çok oyunculu mod, abonelik, 20×20 kapsamı, ses tanıma.
4. Rakiplerin çoğunda erişilebilirlik bilgisi yok. Bu, ürünün farklılaşabileceği bir alan.

## 4. Fonksiyonel gereksinimler

| ID | Gereksinim | Ölçüt / ayrıntı |
|---|---|---|
| **FR-01** | **Tablo seçimi:** Kullanıcı çalışmak istediği tabloları seçebilmelidir. | 1'den 10'a kadar her tablo tek tek veya birlikte seçilebilir. En az bir tablo seçilmeden alıştırma başlatılamaz. |
| **FR-02** | **Tabloyu inceleme (öğrenme görünümü):** Seçilen bir tablonun tüm satırları görüntülenebilmelidir. | Örn. 3'ler için 3×1=3 … 3×10=30 olmak üzere 10 satır, tek ekranda veya kaydırarak gösterilir. |
| **FR-03** | **Toplama ile ilişki gösterimi:** İnceleme görünümünde her çarpım, tekrarlı toplama karşılığıyla birlikte gösterilebilmelidir. | Örn. "3×4 = 3+3+3+3 = 12". Müfredattaki toplama–çarpma ilişkisine destek olur (MAT.2.2.4). |
| **FR-04** | **Alıştırma oturumu:** Uygulama seçilen tablolardan "a × b = ?" biçiminde sorular sormalı, çocuk cevabı sayısal tuş takımıyla girmelidir. | Cevap alanı yalnızca rakam kabul eder, en fazla 3 hane. |
| **FR-05** | **Soru sırası:** Kullanıcı sıralı (×1'den ×10'a) veya karışık soru sırasını seçebilmelidir. | Varsayılan karışık. Karışık modda aynı soru art arda iki kez sorulmaz (FR-08 kapsamındaki tekrar hariç). |
| **FR-06** | **Oturum uzunluğu:** Bir oturumdaki soru sayısı seçilebilmelidir. | Seçenekler 5, 10 ve 20 soru; varsayılan 10. |
| **FR-07** | **Anında geri bildirim:** Her cevaptan sonra doğru veya yanlış olduğu gösterilmelidir. | Cevap onaylandıktan sonra ≤ 0,5 saniyede gösterilir (bkz. NFR-03). Yanlış cevapta doğru cevap gösterilir. Geri bildirim metni suçlayıcı değildir (örn. "Neredeyse! 6×7 = 42"). |
| **FR-08** | **Yanlışları tekrar sorma (oturum içi):** Yanlış cevaplanan soru aynı oturumda tekrar sorulmalıdır. | Yanlış cevaplanan soru, oturum bitmeden en az 1 kez ve hemen ardından değil, en az 2 soru sonra tekrar sorulur. |
| **FR-09** | **Zayıf sorulara öncelik (oturumlar arası):** Karışık modda, geçmişte yanlış cevaplanan veya henüz öğrenilmemiş sorular daha sık sorulmalıdır. | Seçilen tablolarda öğrenilmemiş soru varsa, bir oturumdaki soruların en az %50'si bunlardan seçilir. "Öğrenildi" tanımı FR-10'dadır. |
| **FR-10** | **Soru bazında öğrenme durumu:** Her soru (örn. 7×8) için öğrenme durumu tutulmalıdır. | Üç durum: "Yeni" (hiç sorulmadı), "Çalışılıyor", "Öğrenildi". "Öğrenildi" = o soruya verilen son 3 cevabın doğru olması. Yanlış cevap durumu "Çalışılıyor"a döndürür. |
| **FR-11** | **İlerleme tablosu:** 10×10'luk bir ızgarada her sorunun durumu renk ve simgeyle gösterilmelidir. | Rakiplerdeki ısı haritası yaklaşımına benzer. Durum yalnızca renkle değil, simge veya metinle de ayırt edilir (bkz. NFR-06). |
| **FR-12** | **Oturum sonu özeti:** Oturum bitince sonuç özeti gösterilmelidir. | Doğru sayısı / toplam soru, yanlış yapılan sorular ve bu oturumda "Öğrenildi"ye geçen sorular. |
| **FR-13** | **Basit motivasyon ögesi:** Bir tablonun 10 sorusunun tamamı "Öğrenildi" olduğunda tebrik gösterilmeli ve o tablo için kalıcı bir işaret (örn. yıldız) verilmelidir. | Puan, jeton, mağaza veya skor tablosu yoktur. |
| **FR-14** | **İsteğe bağlı süre:** Alıştırma oturumunda soru başına süre sınırı ayarlardan açılabilmelidir. | Varsayılan kapalı. Açıkken seçenekler soru başına 10, 7 ve 5 saniye. Süre dolunca soru "yanlış" sayılır ve doğru cevap gösterilir. |
| **FR-15** | **İlerlemenin korunması:** Öğrenme durumu (FR-10) ve tablo işaretleri (FR-13) uygulama kapatılıp açıldığında korunmalıdır. | Bir oturum yarıda bırakılırsa o ana kadar verilen cevaplar ilerlemeye işlenir. |
| **FR-16** | **İlerlemeyi sıfırlama:** Tüm ilerleme sıfırlanabilmelidir. | Sıfırlama öncesinde onay istenir. Çocuğun yanlışlıkla sıfırlamasını önlemek için onay adımı yetişkin doğrulaması içerir (örn. basit bir yetişkin sorusu). |
| **FR-17** | **Ses kontrolü:** Ses efektleri varsa ayarlardan kapatılabilmelidir. | Ayar uygulama yeniden açıldığında korunur. |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Kategori | Gereksinim ve ölçüt |
|---|---|---|
| **NFR-01** | Dil | Tüm arayüz metinleri Türkçedir. Ekranlardaki yönerge cümleleri en fazla 8 kelimedir. |
| **NFR-02** | Kullanılabilirlik | 2. sınıf düzeyindeki bir çocuk, ilk açılıştan sonra yetişkin yardımı olmadan **en fazla 3 dokunuşla** bir alıştırma oturumu başlatabilir. Doğrulama: en az 5 çocukla yapılan kullanılabilirlik testinde en az 4'ü başarır. |
| **NFR-03** | Yanıt süresi | Cevap onayından geri bildirimin görünmesine kadar geçen süre **≤ 0,5 sn**, bir sonraki sorunun görünmesi **≤ 1 sn**. Ölçüm hedef cihaz sınıfında yapılır (bkz. Açık sorular). |
| **NFR-04** | Erişilebilirlik: kontrast | Metin ve arka plan kontrastı normal metinde en az **4,5:1**, büyük metinde en az **3:1** olmalıdır (WCAG 2.2 SC 1.4.3, AA seviyesi) ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)). |
| **NFR-05** | Erişilebilirlik: dokunma alanı | Etkileşimli ögeler en az **24×24 CSS piksel** olmalıdır (WCAG 2.2 SC 2.5.8, AA) ([W3C](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)). Çocuk kullanıcılar nedeniyle tuş takımı tuşları için daha büyük boyut önerilir; kesin değer Açık sorular'dadır. |
| **NFR-06** | Erişilebilirlik: renk | Doğru/yanlış geri bildirimi ve ilerleme durumları yalnızca renge dayanmaz; her durumda simge veya metin de bulunur. |
| **NFR-07** | Gizlilik | Uygulama çocuktan ad, e-posta, konum gibi kişisel veri **istemez ve toplamaz**. Hesap oluşturma yoktur. |
| **NFR-08** | Reklam ve satın alma | Uygulamada **reklam ve uygulama içi satın alma yoktur**. Rakiplerin çoğu ücretli kilit veya abonelik kullanıyor; bu fark bilerek tercih edilmiştir (bkz. Açık sorular: iş modeli). |
| **NFR-09** | Pedagojik ton | Hata durumunda olumsuz ses, ceza ya da "oyun bitti" gibi sonlandırma kullanılmaz. Hata yapmak oturumu kesmez. |
| **NFR-10** | Güvenilirlik | Uygulama beklenmedik biçimde kapanırsa, son onaylanan cevaba kadarki ilerleme kaybolmaz. |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Tablo seçip alıştırma yapma**
*Bir öğrenci olarak, çalışmak istediğim tabloları seçip alıştırmaya başlamak istiyorum, böylece yalnızca o tablolara odaklanabilirim.*
İlgili: FR-01, FR-04, FR-05, FR-06, NFR-02
- **Verilen** ana ekranda 3'ler ve 4'ler seçilmişken, **olduğunda** "Başla"ya basılınca, **o zaman** yalnızca 3×n ve 4×n (n=1..10) soruları sorulur.
- **Verilen** hiçbir tablo seçili değilken, **o zaman** "Başla" düğmesi pasiftir.
- **Verilen** oturum uzunluğu 10 seçiliyken, **o zaman** oturum (FR-08 tekrarları hariç) 10 soru sonra biter.

**US-02: Tabloyu inceleme**
*Bir öğrenci olarak, alıştırmadan önce tabloyu toplama karşılığıyla görmek istiyorum, böylece çarpmanın ne anlama geldiğini anlayabilirim.*
İlgili: FR-02, FR-03
- **Verilen** 4'ler tablosu açıldığında, **o zaman** 4×1'den 4×10'a kadar 10 satır görünür.
- **O zaman** her satırda tekrarlı toplama karşılığı yer alır (örn. 4×3 = 4+4+4 = 12).

**US-03: Anında geri bildirim almak**
*Bir öğrenci olarak, cevabımın doğru olup olmadığını hemen görmek istiyorum, böylece yanlışımı hemen düzeltebilirim.*
İlgili: FR-07, NFR-03, NFR-06, NFR-09
- **Verilen** 6×7 sorusuna "42" girilip onaylandığında, **o zaman** en geç 0,5 sn içinde "doğru" geri bildirimi simge ve metinle gösterilir.
- **Verilen** "48" girildiğinde, **o zaman** "yanlış" geri bildirimiyle birlikte doğru cevap "6×7 = 42" gösterilir ve oturum devam eder.

**US-04: Yanlışlarımı tekrar çalışmak**
*Bir öğrenci olarak, yanlış yaptığım soruların bana tekrar sorulmasını istiyorum, böylece onları da öğrenebilirim.*
İlgili: FR-08, FR-09, FR-10
- **Verilen** 7×8 oturumda yanlış cevaplandığında, **o zaman** aynı oturumda en az 2 soru sonra tekrar sorulur.
- **Verilen** seçilen tablolarda "Öğrenildi" olmayan sorular varken, **o zaman** karışık modda bir oturumdaki soruların en az %50'si bu sorulardan gelir.
- **Verilen** bir soru art arda 3 kez doğru cevaplandığında, **o zaman** durumu "Öğrenildi" olur; ardından gelen bir yanlış durumu "Çalışılıyor"a döndürür.

**US-05: İlerlememi görmek**
*Bir öğrenci olarak, hangi soruları öğrendiğimi görmek istiyorum, böylece ne kadar ilerlediğimi bilirim.*
İlgili: FR-11, FR-12, FR-13, FR-15
- **Verilen** ilerleme ekranı açıldığında, **o zaman** 10×10 ızgarada her hücre "Yeni", "Çalışılıyor" veya "Öğrenildi" durumunu renk ve simgeyle gösterir.
- **Verilen** oturum bittiğinde, **o zaman** doğru sayısı / toplam soru, yanlış yapılan sorular ve yeni öğrenilen sorular listelenir.
- **Verilen** 5'ler tablosunun 10 sorusu da "Öğrenildi" olduğunda, **o zaman** tebrik gösterilir ve 5'ler tablosu kalıcı bir yıldız işareti alır.
- **Verilen** uygulama kapatılıp yeniden açıldığında, **o zaman** ızgara ve yıldızlar aynı kalır.

**US-06: Baskı olmadan çalışmak veya kendimi zorlamak**
*Bir öğrenci olarak, varsayılan olarak süre baskısı olmadan çalışmak ama istersem süreli denemek istiyorum, böylece kendi hızımda ilerleyebilirim.*
İlgili: FR-14, NFR-09
- **Verilen** uygulama ilk açıldığında, **o zaman** süre sınırı kapalıdır ve sorularda geri sayım görünmez.
- **Verilen** ayarlardan soru başına 7 sn seçildiğinde, **o zaman** her soruda geri sayım görünür. Süre dolunca soru yanlış sayılır, doğru cevap gösterilir ve sonraki soruya geçilir.

**US-07: Ebeveyn olarak ilerlemeyi takip etmek ve yönetmek**
*Bir ebeveyn olarak, çocuğumun hangi tablolarda zorlandığını görmek ve gerekirse ilerlemeyi sıfırlamak istiyorum, böylece ona doğru yerde destek olabilirim.*
İlgili: FR-11, FR-16, FR-17, NFR-07, NFR-08
- **Verilen** ilerleme ekranı açıldığında, **o zaman** "Çalışılıyor" durumundaki sorular ızgarada ayırt edilebilir.
- **Verilen** "İlerlemeyi sıfırla" seçildiğinde, **o zaman** yetişkin doğrulaması ve onay istenir. Onay verilmezse hiçbir veri silinmez.
- **Verilen** uygulama kullanılırken, **o zaman** hiçbir ekranda kişisel veri istenmez, reklam veya satın alma teklifi görünmez.

## 7. Kapsam dışı maddeler

Aşağıdakiler rakiplerde görülen veya akla gelen fakat "basit" ürün tanımına uymayan maddelerdir:

- Bölme işlemi alıştırması ([3MB Solutions](https://apps.apple.com/us/app/id1451984868) uygulamasında var).
- 10'dan büyük tablolar (11–20).
- Çok oyunculu, iki oyunculu mod ve skor tabloları.
- Hikâyeli oyun dünyaları, mini oyunlar, jeton ve mağaza sistemleri.
- Ses tanıma ile sözlü cevap.
- Kullanıcı hesabı, birden çok çocuk profili, cihazlar arası senkronizasyon.
- Öğretmen veya sınıf yönetim paneli.
- Abonelik, reklam ve uygulama içi satın alma.
- Günlük kullanım süresi sınırı (Times Tables Rock Stars'ta var). Gerekirse Açık sorular'da ele alınır.
- Sayı hissi ve strateji oyunları (YouCubed önerileri). Değerli, ancak ayrı bir ürün kapsamı.

## 8. Varsayımlar

1. Hedef yaş yaklaşık 7–10 (2.–4. sınıf) olarak kabul edilmiştir. Bu, 2. sınıf müfredat bulgusuna ve rakiplerin "6–8 yaş" etiketine dayanır, paydaş tarafından doğrulanmamıştır.
2. Tablo aralığı **1–10** (10×10) olarak varsayılmıştır. 3. sınıf ve sonrası müfredat aralığı bu oturumda doğrulanamamıştır.
3. Uygulama Türkiye'deki kullanıcılar için Türkçe hazırlanacaktır.
4. Bir cihazda tek çocuğun kullanacağı varsayılmıştır (profil yönetimi yok).
5. "Öğrenildi" tanımındaki "son 3 cevap doğru" eşiği ürün kararı olarak önerilmiştir; araştırma kaynaklı bir standart değildir.
6. FR-09'daki "%50 öncelik" oranı ürün kararıdır; bilimsel olarak optimize edilmiş bir değer değildir.
7. Oturum uzunlukları (5/10/20 soru) ve süre seçenekleri (10/7/5 sn) ürün kararıdır. KQED'de aktarılan çalışmadaki 5 dakikalık kısa oturumlar ölçek için yol gösterici kabul edilmiştir.
8. KQED çalışmasının (48 öğrenci) ve yetişkinlerle yapılan PMC çalışmasının bulgularının bu ürünün kullanıcılarına genellenebileceği varsayılmıştır. Her iki çalışmanın da sınırlılıkları var.
9. Ürün ücretsiz ve ticari olmayan bir proje olarak varsayılmıştır (NFR-08).

## 9. Açık sorular

1. **Platform:** Ürün hangi cihazlarda kullanılacak (telefon, tablet, bilgisayar tarayıcısı)? Bu, NFR-03 ölçüm koşullarını ve NFR-05 boyutlarını etkiler. Seçim TA'ya bırakılmıştır; iş açısından cevaplanması gereken soru, hedef kullanıcının hangi cihaza eriştiğidir.
2. **İnternetsiz çalışma:** Uygulama internet bağlantısı olmadan kullanılabilmeli mi?
   - Fayda: evde veya yolda bağlantı olmadan kesintisiz kullanım, veri toplanmamasıyla uyum.
   - Maliyet: içerik ve ilerlemenin tamamen cihazda tutulması gerekir, cihazlar arası taşıma olmaz.
3. **Tablo aralığı:** 1–10 yeterli mi, yoksa 12×12 veya 20×20 isteniyor mu? Rakiplerde 10, 12 ve 20 sınırları var.
4. **Müfredat aralığı:** 2. sınıfta 5'e kadar olduğu doğrulandı. 3. sınıfta beklenen tablo aralığı nedir? Bu oturumda doğrulanamadı; MEB programı ile teyit edilmeli.
5. **Varsayılan tablo seti:** İlk açılışta 2. sınıf müfredatına uygun olarak yalnızca 1–5 tabloları mı önerilmeli?
6. **Sesli anlatım:** Okuma güçlüğü çeken küçük çocuklar için soruların sesli okunması isteniyor mu? Rakiplerin bir kısmında var. Kapsama alınırsa yeni bir FR gerekir.
7. **Cevap biçimi:** Yalnızca yazarak cevap mı, yoksa başlangıç seviyesi için çoktan seçmeli seçenek de mi? Hatırlama pratiği açısından yazarak cevap daha güçlü bir geri çağırma gerektirir. Çoktan seçmeli seçenek küçük yaş için erişimi kolaylaştırır.
8. **Ebeveyn özellikleri:** Ebeveyn için ilerleme ızgarası (FR-11) yeterli mi, yoksa ayrı bir özet ekranı isteniyor mu?
9. **Günlük kullanım sınırı:** Times Tables Rock Stars'taki gibi bir günlük süre sınırı isteniyor mu?
10. **İş modeli:** Ürün tamamen ücretsiz mi kalacak? NFR-08 buna göre belirlendi.
11. **Dokunma alanı:** Çocuk kullanıcılar için tuş takımı tuşlarının kesin minimum boyutu nedir? WCAG AA en az 24×24 CSS piksel diyor; daha büyük bir değer kararı paydaş ve tasarımla verilmeli.
12. **Yasal uyum:** Kişisel veri toplanmasa da çocuklara yönelik uygulamalar için uygulama mağazası veya mevzuat gereksinimleri (örn. KVKK) var mı? Bu oturumda araştırılmadı.

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | KQED MindShift: Flash Cards Prevail Over Repetition in Memorizing Multiplication Tables | http://www.kqed.org/mindshift/62663/flash-cards-prevail-over-repetition-in-memorizing-multiplication-tables | 48 öğrencilik çalışma; kartla hatırlayarak çalışma ile yüksek sesle tekrar karşılaştırması (8'e 6 doğru); haftada 3×5 dk; anında geri bildirim; sonucun bir hafta sonra da sürmesi; çalışmanın küçük ölçeği. |
| 2 | Schrift, Dotan, Censor (2022), npj Science of Learning (PMC9385657) | https://pmc.ncbi.nlm.nih.gov/articles/PMC9385657 | Yetişkinlerde aritmetik olgu öğrenimi; üç ayrı günde geri bildirimli hatırlama oturumlarının en çok kazanımı sağlaması; günlere yaymanın olası katkısı. |
| 3 | YouCubed: Fluency Without Fear | https://www.youcubed.org/evidence/fluency-without-fear/ | Süreli testlerin matematik kaygısıyla ilişkisi (öğrencilerin yaklaşık üçte biri, 5 yaştan itibaren); hıza vurgu yapmama ve sayı hissi önerileri. |
| 4 | Times Tables Rock Stars | https://ttrockstars.com/ | Uyarlanabilir sorular, çok oyunculu oyunlar, rozet ve ödüller, ilerleme ısı haritası, 20 dakikalık günlük sınır, abonelik modeli. |
| 5 | App Store: Multiplication Tables! (3MB Solutions) | https://apps.apple.com/us/app/id1451984868 | 2–20 tablolar, sıralı ve karışık mod, isteğe bağlı süre, çoktan seçmeli ve tuş takımı, rozetler, ebeveyn ayarları, aralıklı tekrar olmaması, 4,8 puan. |
| 6 | App Store: Times Tables Multiplication (Learning Apps) | https://apps.apple.com/us/app/id1089191129 | 1–10 tablolar, sesli anlatım, kısa sınavlar, 2,99 $ kilit açma, 3,9 puan, 11–20 talebi. |
| 7 | App Store: Times Tables and Friends | https://apps.apple.com/us/app/-/id1341586204 | Beş mod (zamanlı meydan okuma, süresiz antrenman dâhil), 12×12, ikiden fazla yanlışta oyunun bitmesi, ses tanıma şikâyeti, 4,99 $. |
| 8 | App Store: Multiplication: Times Tables x (Mobilaxy) | https://apps.apple.com/us/app/-/id1498747525 | 20×20, beş mod, zorlanılan sorulara antrenman, abonelik fiyatları. |
| 9 | TYMM: İlkokul Matematik 2. Sınıf, 3. Tema "İşlemlerden Cebirsel Düşünmeye" | https://tymm.meb.gov.tr/ilkokul-matematik-dersi/unite/164 | MAT.2.2.4 ve MAT.2.2.5 öğrenme çıktıları; 2. sınıfta çarpım tablosunun 5'e kadar (5 dâhil) oluşturulması. |
| 10 | W3C: Understanding SC 1.4.3 Contrast (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html | Normal metinde 4,5:1, büyük metinde 3:1 kontrast; AA seviyesi. |
| 11 | W3C: Understanding SC 2.5.8 Target Size (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | Dokunma hedeflerinin en az 24×24 CSS piksel olması; AA seviyesi. |