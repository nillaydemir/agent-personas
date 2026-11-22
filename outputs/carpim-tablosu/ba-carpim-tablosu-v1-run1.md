# Çarpım Tablosu Öğrenme Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ba v1
- **Girdi:** "Basit bir çarpım tablosu öğrenme uygulaması yapacağız. Bu konuda araştırmalar yaparak mevcut çalışmaları da göz önünde bulundurarak iş gerekliliklerini belirle ve bize bir analiz dokümanı hazırla."

---

## 1. Özet

Bu ürün, ilkokul çağındaki çocukların çarpım tablosunu öğrenip ezberlemesine yardım eden **basit** bir uygulamadır. Görev metni "basit" dediği için kapsam küçük tutuldu. Uygulama yalnızca çarpma işlemini, tek bir çocuk kullanıcıyı ve dört temel işlevi kapsar:

1. Öğrenilecek tabloyu seçmek
2. Bir tabloyu görsel açıklamayla öğrenmek (tekrarlı toplama, eşit gruplar)
3. Anında geri bildirimli alıştırma yapmak
4. İlerlemeyi işlem bazında görmek ve zorlanılan işlemleri daha sık tekrar etmek

Araştırmadan çıkan başlıca bulgular:
- **Hatırlama pratiği işe yarıyor.** Cevabı akıldan çıkarmayı gerektiren çalışma (flash kart, quiz), sesli tekrar etmekten daha etkili bulundu. Bu etki bir hafta sonra da sürdü. Bkz. [K2].
- **Bazı işlemler daha çok tekrar istiyor.** 4, 5, 6 ve 7'li işlemler, 2 ve 3'lülerden anlamlı ölçüde daha fazla tekrar gerektiriyor. Matematikte zorlanan öğrencilerin de daha çok tekrara ihtiyacı var. Bkz. [K1].
- **Rakip uygulamalarda ortak bir özellik seti var.** Tablo bazlı dersler, karışık tekrar, süreli turlar, rozet ve seri gibi ödüller, işlem bazında akıcılık haritası ve ebeveyn alanı. Bkz. [K4], [K5], [K6].
- **MEB programı tekrarlı toplamayı öne çıkarıyor.** Çarpma, tekrarlı toplama ve eşit gruplama üzerinden anlatılıyor. 0 ve 1'in çarpmadaki etkisi ayrıca vurgulanıyor. Bkz. [K3].

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | İhtiyaç | Dayanak |
|---|---|---|---|
| **P1: Öğrenen çocuk (birincil)** | Yaklaşık 2–4. sınıf öğrencisi, okumayı yeni sökmüş olabilir. | Kısa, eğlenceli, sıkmayan alıştırma. Yanlışta cezalandırılmama, hemen doğruyu görme. | MEB İYEP içeriği çarpım tablosunu 3. sınıf temasında işliyor [K3]. Rakip bir uygulama da 2–4. sınıfları hedefliyor [K4]. |
| **P2: Ebeveyn (ikincil)** | Çocuğun ev çalışmasını takip eden yetişkin. | Çocuğun hangi tabloları bildiğini ve hangilerinde zorlandığını hızlıca görmek. | Rakip uygulamada kilitli ebeveyn alanı ve ilerleme görünümü var [K4]. |

Öğretmen ve sınıf yönetimi bu sürümde hedef kullanıcı değildir (bkz. Bölüm 7).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Uygulamalar

| Çözüm | Öne çıkan özellikler | Güçlü yön | Zayıf yön / risk | Kaynak |
|---|---|---|---|---|
| **Simply Times Tables: Practice** (iOS) | 1×1'den 12×12'ye 144 işlem. Her tablo için bir "ülke" haritası. Eşit gruplar ve atlayarak sayma ile görsel ipuçları. Bir dakikalık hız turları (1–9 veya 1–12). Karışık tekrar, günlük seri, işlem başına çıkartma. Kilitli ebeveyn alanı, günlük ders sınırı, çoklu çocuk profili. Çevrimdışı çalışıyor, reklam ve hesap yok. | Pedagojik akış (önce ders, sonra hız) ve çocuk güvenliği (reklamsız, veri toplamıyor). | Mağazadaki tek yorum, cevap gönderilince uygulamanın kapandığını söylüyor (1/5). Fiyat bilgisi çelişkili ($3.00 ve $0.99). Erişilebilirlik özelliği beyan edilmemiş. | [K4] |
| **Çarpım Tablosu-Times Table** (Hız Yayınları, iOS) | Kolay, Orta ve Zor seviyeler. Her blokta 12 soru ve 30 saniye süre. Çocuğun tek başına kullanabileceği sade arayüz. Ücretsiz. | Türkçe, sade, kısa oturumlar. | Öğretici ve görsel açıklama modu yok. Kişisel ilerleme takibinden bahsedilmiyor. Yeterli yorum yok. Gizlilik ve erişilebilirlik beyanı yok. Boyutu 91,6 MB. | [K5] |
| **Times Tables Rock Stars** (Birleşik Krallık, okul odaklı) | Süresiz mod (Jamming), 1–5 dakikalık süreli modlar, uyarlanabilir mod (Garage), MTC benzeri sınav modu (Soundcheck), çok oyunculu modlar, coin ödülleri. **Heatmap:** her işlem için son 10 süreli cevabın ortalama süresi. Koyu yeşil 1 saniyenin altı, koyu kırmızı 10 saniyenin üstü. | İşlem bazında akıcılık ölçümü ve görselleştirme. Süresiz ve süreli modların bir arada sunulması. | Okul ve öğretmen ekosistemine bağlı, kapsamı geniş. Basit bir ev uygulaması için fazla karmaşık. | [K6], [K7] |

### 3.2 Yöntemler ve standartlar

- **Hatırlama pratiği ve aralıklı tekrar:** 48 ikinci sınıf öğrencisiyle yapılan bir deneyde flash kart grubu ortalama yaklaşık 8 işlem, sesli tekrar grubu yaklaşık 6 işlem öğrendi. Fark bir hafta sonra da sürdü. Önerilenler: hafta içine yayılmış kısa oturumlar, anında geri bildirim ve doğru cevabı gördükten sonra yeniden deneme. Örneklem küçük. [K2]
- **Tekrar ihtiyacı:** 15.402 öğrencinin (3–5. sınıf) verisine göre 4, 5, 6 ve 7'li işlemler, 2 ve 3'lülerden daha fazla tekrar gerektiriyor. Düşük beceri düzeyindeki öğrencilerin daha çok tekrara ihtiyacı var. [K1]
- **MEB yaklaşımı (İYEP, 3. sınıf):** Çarpma, tekrarlı toplama ve ritmik sayma ile anlamlandırılıyor. Nesneler ikişerli, üçerli, dörderli ve beşerli gruplanıyor. 10'a kadar olan sayılar 1–5 ile çarpılıyor. Çarpanların yer değiştirmesinin sonucu değiştirmediği gösteriliyor. 5'e kadar çarpım tablosu yüzlük tablo ile oluşturuluyor. 0 ve 1'in etkisi vurgulanıyor. [K3]
- **İngiltere MTC (referans sınav formatı):** 25 soru, soru başına 6 saniye, 12'ye kadar tablolar. Asıl sınavdan önce 3 deneme sorusu var, geçme notu yok ve sınav ortalama 5 dakika sürüyor. [K8]
- **Erişilebilirlik (WCAG 2.2):** Asgari dokunma hedefi 24×24 CSS piksel (AA). Geliştirilmiş hedef 44×44 CSS piksel. Normal metin için kontrast en az 4.5:1, büyük metin için en az 3:1 (AA). [K9], [K10]

### 3.3 Ürüne çıkarımlar
1. Alıştırma, cevabı **akıldan çıkarmaya** dayalı olmalı. Yanlış cevaptan sonra doğrusu gösterilmeli ve işlem tekrar sorulmalı [K2].
2. Zorlanılan işlemler daha sık sorulmalı [K1], [K6].
3. Öğretme kısmı tekrarlı toplama ve eşit gruplar üzerinden kurulmalı. 0 ve 1 kuralları ayrıca gösterilmeli [K3], [K4].
4. Süre baskısı isteğe bağlı olmalı. Rakipler süresiz ve süreli modları ayrı sunuyor [K4], [K7].
5. Çocuk güvenliği (reklamsız, hesapsız, veri toplamayan yapı) rakiplerde bir farklılaştırıcı olarak kullanılıyor [K4].

## 4. Fonksiyonel gereksinimler

| ID | Gereksinim | Ölçüt / not |
|---|---|---|
| **FR-01** | Kullanıcı çalışmak istediği bir veya birden fazla tabloyu seçebilmelidir. | Tablo aralığı varsayılan olarak 1–10'dur (bkz. Varsayım A2, Açık soru S1). En az bir tablo seçilmeden alıştırma başlatılamaz. |
| **FR-02** | Her tablo için bir **öğrenme ekranı** bulunmalıdır. Bu ekran işlemleri tekrarlı toplama ve eşit gruplar ile görsel olarak göstermelidir. | Örnek: 3×4 için "4+4+4" ve 3 grup halinde 4'er nesne [K3]. |
| **FR-03** | Öğrenme ekranında 0 ve 1 ile çarpma kuralı ve çarpanların yer değiştirmesi (3×4 = 4×3) ayrıca gösterilmelidir. | MEB vurgusu [K3]. |
| **FR-04** | **Alıştırma modu** seçilen tablolardan rastgele sırada sorular sormalı, cevap sayısal klavye ile girilmelidir. | Bir turdaki soru sayısı varsayılan 10'dur ve kullanıcı 10, 20 veya 30 arasında seçim yapabilir (TTRS Jamming soru sayılarına benzer [K7]). |
| **FR-05** | Her cevaptan hemen sonra doğru veya yanlış geri bildirimi verilmelidir. Yanlışta doğru cevap gösterilmelidir. | [K2] |
| **FR-06** | Yanlış cevaplanan işlem aynı tur içinde en az bir kez daha sorulmalıdır. | Doğru cevabı gördükten sonra yeniden deneme [K2]. |
| **FR-07** | Alıştırma modunda süre sınırı **isteğe bağlı** olmalıdır. Varsayılan olarak süre yoktur. Süreli seçenekte soru başına süre gösterilmelidir. | Süreli seçenek için varsayılan 6 saniye önerilir (MTC formatı [K8]); kesin değer Açık soru S4'tedir. |
| **FR-08** | Tur sonunda bir **özet ekranı** gösterilmelidir: doğru/yanlış sayısı, yanlış yapılan işlemlerin listesi ve doğru cevapları. | |
| **FR-09** | Uygulama her işlem için (ör. 7×8) doğru/yanlış geçmişini ve cevap süresini kaydetmelidir. Bu kayıt uygulama kapatılıp açıldığında korunmalıdır. | TTRS son 10 cevabın ortalamasını kullanıyor [K6]. |
| **FR-10** | Bir **ilerleme ekranı**, işlemleri tablo ızgarası (1–10 × 1–10) üzerinde renklerle "öğrenildi", "çalışılıyor" ve "henüz denenmedi" olarak göstermelidir. | "Öğrenildi" tanımı Varsayım A4'tedir. Heatmap benzeri gösterim [K6]. |
| **FR-11** | Alıştırma modunda soru seçimi, "çalışılıyor" durumundaki (son denemelerde yanlış veya yavaş cevaplanmış) işlemleri, "öğrenildi" durumundakilerden **daha sık** sormalıdır. | Basit ağırlıklandırma yeterlidir. Algoritma seçimi TA'ya bırakılmıştır [K1], [K2]. |
| **FR-12** | Bir **karışık tekrar** seçeneği, daha önce çalışılmış tüm tablolardan soru sormalıdır. | [K4] |
| **FR-13** | Doğru cevap ve tur tamamlama için basit bir ödül gösterilmelidir (ör. yıldız). Ödüller yanlış cevapta puan düşürmemelidir. | Rozet, coin ve çıkartma rakiplerde yaygın [K4], [K7]. Kapsam küçük tutuldu. |
| **FR-14** | Kullanıcı ses efektlerini açıp kapatabilmelidir. | Rakipte "daha sakin" ayar var [K4]. |
| **FR-15** | Kullanıcı ilerleme verisini, onay adımı olan bir işlemle sıfırlayabilmelidir. | |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt |
|---|---|---|
| **NFR-01** | **Dil:** Arayüz tamamen Türkçe olmalıdır. Matematik terimleri MEB terminolojisiyle uyumlu olmalıdır (çarpan, çarpım, tekrarlı toplama). | Arayüzde İngilizce metin sayısı 0. Terimler [K3] ile uyumlu. |
| **NFR-02** | **Kullanılabilirlik (çocuk):** Ana ekrandan alıştırmayı başlatmak en fazla 3 dokunuş gerektirmelidir. | Ana ekran → tablo seçimi → başlat ≤ 3 dokunuş. |
| **NFR-03** | **Oturum süresi:** Varsayılan bir tur (10 soru, süresiz) tipik bir çocuk için yaklaşık 5 dakikayı aşmamalıdır. | Kısa ve aralıklı oturum önerisi [K2]. MTC 25 soruyu yaklaşık 5 dakikada tamamlatıyor [K8]. |
| **NFR-04** | **Dokunma hedefleri:** Tüm etkileşimli öğeler (rakam tuşları dahil) en az 44×44 CSS piksel olmalıdır. | WCAG 2.5.5 eşiği [K9]. Hedef kitle küçük çocuk olduğu için asgari 24×24 yerine geliştirilmiş eşik seçildi. |
| **NFR-05** | **Kontrast:** Metin kontrastı normal metinde en az 4.5:1, büyük metinde en az 3:1 olmalıdır. | WCAG AA [K10]. |
| **NFR-06** | **Renge bağımlı olmama:** İlerleme ızgarasındaki durumlar (FR-10) yalnızca renkle değil, simge veya metinle de ayırt edilebilmelidir. | Ekran görüntüsü gri tonlamada incelendiğinde 3 durum ayırt edilebilir olmalı. |
| **NFR-07** | **Geri bildirim hızı:** Cevap gönderildikten sonra doğru/yanlış geri bildirimi kullanıcıya gecikmesiz görünmelidir. | Hedef ≤ 200 ms (varsayım, bkz. A6). |
| **NFR-08** | **Gizlilik ve çocuk güvenliği:** Uygulama reklam içermemeli, hesap açtırmamalı ve kişisel veri toplamamalıdır. İlerleme verisi yalnızca uygulama işlevi için tutulmalıdır. | Reklam: 0, zorunlu kayıt: 0. Rakip uygulamada farklılaştırıcı [K4]. Yasal gereklilikler Açık soru S6'dadır. |
| **NFR-09** | **Kararlılık:** Cevap gönderme akışında çökme olmamalıdır. | Kabul testinde 100 ardışık cevap gönderiminde 0 çökme. Rakipte bu konuda olumsuz yorum var [K4]. |
| **NFR-10** | **Veri kalıcılığı:** FR-09 kayıtları uygulama yeniden başlatıldığında kaybolmamalıdır. | Kayıp oranı 0 (uygulama kaldırılmadığı sürece). |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Tablo seçme** (FR-01)
Bir çocuk olarak hangi tabloyu çalışacağımı seçmek istiyorum, böylece bilmediğim tabloya odaklanabilirim.
- KK1: 1–10 tabloları seçilebilir olarak listelenir.
- KK2: Birden fazla tablo seçilebilir.
- KK3: Hiç tablo seçilmemişse "Başlat" pasiftir.

**US-02: Tabloyu öğrenme** (FR-02, FR-03)
Bir çocuk olarak bir tablonun ne anlama geldiğini görmek istiyorum, böylece ezberlemeden önce anlayabilirim.
- KK1: Seçilen tablonun her işlemi tekrarlı toplama ifadesi ve eşit grup görseliyle gösterilir.
- KK2: 0 ve 1 ile çarpma kuralı ayrıca açıklanır.
- KK3: Yer değiştirme örneği (ör. 3×4 = 4×3) gösterilir.

**US-03: Alıştırma yapma** (FR-04, FR-05, FR-06)
Bir çocuk olarak soruları cevaplayıp hemen doğru mu yanlış mı olduğunu görmek istiyorum, böylece hatamı hemen öğrenebilirim.
- KK1: Sorular seçilen tablolardan gelir ve sıraları her turda farklıdır.
- KK2: Yanlış cevaptan sonra doğru cevap gösterilir.
- KK3: Yanlış cevaplanan işlem aynı turda tekrar sorulur.
- KK4: Soru sayısı 10, 20 veya 30 olarak seçilebilir.

**US-04: Süreli meydan okuma** (FR-07)
Bir çocuk olarak isteğe bağlı olarak süreye karşı oynamak istiyorum, böylece hızımı test edebilirim.
- KK1: Varsayılan modda süre sayacı görünmez.
- KK2: Süreli mod açıldığında kalan süre gösterilir. Süre dolduğunda soru "yanlış" sayılır ve doğru cevap gösterilir.

**US-05: Tur sonucunu görme** (FR-08, FR-13)
Bir çocuk olarak turun sonunda nasıl yaptığımı görmek istiyorum, böylece başarımı ve hatalarımı bilirim.
- KK1: Özet ekranı doğru ve yanlış sayısını gösterir.
- KK2: Yanlış yapılan işlemler doğru cevaplarıyla listelenir.
- KK3: Kazanılan ödül gösterilir. Yanlışlar ödülden düşülmez.

**US-06: Zorlandığım işlemleri daha çok çalışma** (FR-09, FR-11, FR-12)
Bir çocuk olarak zorlandığım işlemlerin daha sık sorulmasını istiyorum, böylece onları da öğrenebilirim.
- KK1: Bir işlem art arda yanlış cevaplandığında sonraki turlarda "öğrenildi" durumundaki işlemlerden daha sık sorulur. Bu, 50 soruluk bir örneklemde ölçülebilir olmalıdır.
- KK2: Karışık tekrar modu daha önce çalışılmış tüm tablolardan soru içerir.

**US-07: İlerlemeyi görme** (FR-09, FR-10, FR-15)
Bir ebeveyn olarak çocuğumun hangi işlemleri bildiğini görmek istiyorum, böylece onu doğru yönlendirebilirim.
- KK1: İlerleme ızgarası her işlemi 3 durumdan biriyle gösterir.
- KK2: Uygulama kapatılıp açıldığında ilerleme korunur.
- KK3: "İlerlemeyi sıfırla" onay istendikten sonra tüm durumları "henüz denenmedi" yapar.

**US-08: Sesi kapatma** (FR-14)
Bir ebeveyn olarak ses efektlerini kapatabilmek istiyorum, böylece çocuk sessiz ortamda da çalışabilir.
- KK1: Ses ayarı kapatıldığında hiçbir ses efekti çalmaz ve ayar kalıcıdır.

## 7. Kapsam dışı maddeler

Bu maddeler "basit" ürün tanımı ve kapsam kuralı gereği bu sürüme alınmadı:
- Bölme, toplama, çıkarma ve kesir gibi diğer işlemler. Rakiplerde var [K4], [K7] ama görevde istenmedi.
- Çok oyunculu ve çevrim içi yarışma modları (Festival, Arena, Rock Slam benzeri [K7]).
- Öğretmen ve sınıf yönetimi paneli, okul hesapları.
- Birden fazla çocuk profili (rakipte var [K4]). Talep edilirse ileride değerlendirilebilir, bkz. S5.
- Hatırlatma ve bildirimler, günlük seri (streak) mekaniği.
- Sesli soru okuma (text-to-speech) ve çoklu dil desteği.
- Ödül mağazası, avatar özelleştirme gibi kapsamlı oyunlaştırma.
- Uygulama içi satın alma ve abonelik.
- 10'un üzerindeki tablolar (11, 12), bkz. S1.

## 8. Varsayımlar

- **A1:** Hedef kullanıcı 2–4. sınıf düzeyindeki, Türkçe konuşan ve okuyabilen çocuklardır.
- **A2:** Tablo aralığı 1–10'dur. Türkiye'deki yaygın uygulamaya göre varsayıldı ve doğrulanmadı. MEB İYEP sayfası 3. sınıfta 5'e kadar tabloyu gösteriyor [K3]. İngiltere'de 12'ye kadar tablo kullanılıyor [K8].
- **A3:** Uygulama tek bir çocuk tarafından tek cihazda kullanılır. Hesap ve senkronizasyon gerekmez.
- **A4:** Bir işlem, son 3 denemesinin tamamı doğru olduğunda "öğrenildi" sayılır. Bu eşik keyfidir ve paydaş onayı gerekir. TTRS süre temelli ölçüm (son 10 cevap) kullanıyor [K6].
- **A5:** Ebeveyn alanı için ayrı bir kilit gerekmez. İlerleme ekranı herkese açıktır.
- **A6:** NFR-07'deki 200 ms hedefi kaynağa dayanmaz, "gecikmesiz algı" için yaklaşık bir değerdir.
- **A7:** Ürün ücretsiz ve ticari amaç taşımayan bir proje olarak düşünülmüştür.

## 9. Açık sorular

- **S1:** Tablo aralığı ne olmalı: 1–5 (MEB 3. sınıf İYEP [K3]), 1–10 veya 1–12 [K8]? 0 tablosu ayrı bir tablo olarak dahil edilmeli mi?
- **S2:** Hangi platform(lar) hedefleniyor: telefon, tablet, web? Bu karar TA'ya girdi olacaktır.
- **S3:** Uygulamanın internet olmadan çalışması gerekli mi? İş açısından artısı, çocuğun her yerde çalışabilmesi ve veri paylaşımının olmaması (rakipte de bu yaklaşım var [K4]). Eksisi, ilerlemenin cihaz değişince taşınamaması. Seçim paydaş ihtiyacına ve TA'ya bağlıdır.
- **S4:** Süreli modda soru başına süre ne olmalı: 6 saniye (MTC [K8]) mi, daha uzun mu? 2. sınıf için ayrı bir değer gerekir mi?
- **S5:** Bir cihazda birden fazla çocuk (kardeşler) kullanacak mı? Kullanacaksa profil desteği kapsama alınmalı mı?
- **S6:** Çocuklara yönelik uygulamalar için hangi yasal gereklilikler (ör. KVKK) geçerli? Bu konu bu çalışmada araştırılmadı, hukuki görüş gerekir.
- **S7:** Görseller ve sesler için marka, karakter veya maskot beklentisi var mı?
- **S8:** "Öğrenildi" kriteri doğruluğa mı, hıza mı, yoksa ikisine birden mi dayanmalı (bkz. A4)?

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | Number of repetitions required to retain single-digit multiplication math facts for elementary students (Penn State Research Portal) | https://pure.psu.edu/en/publications/number-of-repetitions-required-to-retain-single-digit-multiplicat/ | 15.402 öğrenci verisi. 4–7'li işlemlerin daha çok tekrar gerektirmesi. Düşük beceri düzeyinde daha çok tekrar ihtiyacı. |
| K2 | Flash Cards Prevail Over Repetition in Memorizing Multiplication Tables (KQED MindShift) | https://www.kqed.org/mindshift/62663/flash-cards-prevail-over-repetition-in-memorizing-multiplication-tables | Hatırlama pratiğinin sesli tekrardan üstünlüğü (48 öğrenci). Aralıklı kısa oturumlar, anında geri bildirim, yeniden deneme. |
| K3 | MEB TYMM: İlkokullarda Yetiştirme Programı (İYEP) ünite sayfası | https://tymm.meb.gov.tr/ilkokullarda-yetistirme-programi-iyep/unite/1831 | Tekrarlı toplama ve eşit gruplama. 1–5 ile çarpma, 5'e kadar tablo. 0 ve 1'in etkisi, yer değiştirme. |
| K4 | Simply Times Tables: Practice (App Store) | https://apps.apple.com/app/id1472813038 | Rakip özellikleri (dersler, hız turları, karışık tekrar, ebeveyn alanı, çevrimdışı, reklamsız), hedef sınıflar ve olumsuz yorum. |
| K5 | Çarpım Tablosu-Times Table, Hız Yayınları (App Store) | https://apps.apple.com/us/app/id1513006155 | Türkçe rakip: 3 seviye, 12 soru ve 30 saniyelik bloklar, ücretsiz, gizlilik ve erişilebilirlik beyanı yok. |
| K6 | Heatmaps (TTRS) – Times Tables Rock Stars Yardım Merkezi | https://intercom.help/times-tables-rock-stars/en/articles/3871753-heatmaps-ttrs | İşlem bazında akıcılık haritası: son 10 süreli cevabın ortalaması, renk eşikleri (1 saniyenin altı ve 10 saniyenin üstü). |
| K7 | Game types for schools (TTRS) – Times Tables Rock Stars Yardım Merkezi | https://intercom.help/times-tables-rock-stars/en/articles/1616983-game-types-for-schools-ttrs | Oyun modları, süreli ve süresiz modlar, 10/20/30 soru seçeneği, Soundcheck formatı. |
| K8 | Multiplication tables check: information for parents (GOV.UK) | https://gov.uk/government/publications/multiplication-tables-check-information-for-parents/multiplication-tables-check-information-for-parents-text-version--2 | MTC formatı: 25 soru, 6 saniye, 12'ye kadar tablolar, 3 deneme sorusu, geçme notu yok, yaklaşık 5 dakika. |
| K9 | Understanding SC 2.5.8: Target Size (Minimum) (W3C WCAG 2.2) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | Asgari dokunma hedefi 24×24 CSS piksel (AA). Geliştirilmiş eşik 44×44 CSS piksel. |
| K10 | Understanding SC 1.4.3: Contrast (Minimum) (W3C WCAG 2.2) | https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html | Kontrast oranları: normal metin 4.5:1, büyük metin 3:1 (AA). |

---

**Ana oturum için not (dokümanın parçası değildir):** Dokümanın tamamı yukarıda; hiçbir dosya yazılmadı. Kayıt yolu `/home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v1-run1.md` olmalı. Bu not satırı kayda alınmamalı.

- Üç kural dosyası, Aşama 1'de izlenmesi gereken konu olarak not edilen davranışın aksine, bu sefer Read ile ayrıca açıldı.
- Yalnızca bu oturumda WebFetch ile açılıp okunan 10 sayfa kaynak gösterildi.
- Açılamayan Türkçe App Store sayfası (404) kaynak gösterilmedi. Yalnızca arama sonucunda görülen sayfalar da kaynak gösterilmedi.