# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: İş Analizi Dokümanı (Güncelleme)

- **Tarih:** 2026-11-24
- **Persona sürümü:** ba v2
- **Girdi:** Flutter ile Android ve iOS cihazlarda çalışacak bir randevu yönetim uygulaması hazırlayacağız. Firebase kullanarak kullanıcı kaydı, randevu oluşturma, iptal etme, çalışan takvimleri ve bildirim özelliklerini değerlendireceğiz. Güzellik salonları ve kuaförler için mevcut uygulamaları araştırarak iş gerekliliklerini belirle ve bize bir iş analizi dokümanı hazırla.
  - Önceki BA dokümanı: outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run1.md
  - TA'nın bu dokümana dair geri soruları: outputs/randevu-yonetimi/ta-randevu-yonetimi-v2-run1.md ("12. BA'ya geri sorular" bölümü)
  - Paydaşın (kullanıcının) yanıtları, kendi ifadesiyle:
    1. Tek salon mu, çok işletmeli platform mu (AS-01 / TA S-01)? → "tek salona ait olacak"
    2. Hesap silindiğinde geçmiş randevular anonimleştirilsin mi, tamamen silinsin mi (TA S-03)? → "hesap silindi şeklinde tutulabilir. hard delete olmamalı"
    3. İşletmenin girdiği iptal nedeni bildirimde gösterilsin mi (TA S-08, US-07 KK-1)? → "müşteriye bir şekilde bildirilmeli, eğer bildirimde olursa iyi olur"
    4. Hizmet notu alanı gerekiyor mu (TA S-09, V-08)? → "hayır"
    5. Randevu hatırlatma süresinden daha yakın bir zamanda alınırsa hatırlatma ne olsun (TA S-06)? → "hemen gönderilir"
  - Önceki dokümanı ve TA'nın sorularını okuyup bu yanıtları işleyerek güncellenmiş, eksiksiz (tek başına okunabilir) bir BA dokümanı üret. Yanıtların gereksinimlere, kabul kriterlerine, varsayımlara ve açık sorulara etkisini yansıt; cevaplanan soruları açık sorulardan çıkar. Mevcut FR/NFR/US ID'lerini koru (yeniden numaralandırma yapma); kaldırılan bir madde olursa ID'sini boş bırakıp nedenini belirt, yeni madde gerekirse sıradaki ID'yi kullan. Paydaş yanıtı bir mevzuat veya mağaza politikası yükümlülüğüyle çelişiyorsa (ör. "hard delete olmamalı" ile KVKK silme yükümlülüğü ve Apple/Google hesap silme kuralları) bunu açıkça belirt ve çözüm seçeneklerini iş açısından açıklayıp gerekiyorsa açık soru olarak yaz. Dokümanda bir "Değişiklik notu" bölümüyle önceki sürüme göre neyin hangi yanıt nedeniyle değiştiğini listele.

---

## 1. Özet

Ürün, **tek bir güzellik salonuna veya kuaföre ait** bir mobil randevu uygulamasıdır. Müşteriler bu salondan Android ve iOS uygulaması üzerinden randevu alır. Görevde istenen beş ana yetenek şunlardır: kullanıcı kaydı, randevu oluşturma, randevu iptali, çalışan takvimleri ve bildirimler. Flutter ve Firebase, ekibin görevde verdiği kısıtlardır. Bu doküman bunların üzerine ek bir teknoloji kararı vermez.

Bu sürüm, önceki BA dokümanını paydaşın 5 yanıtına göre günceller. Ayrıntılı değişiklik listesi 11. bölümdedir (Değişiklik notu). Başlıca etkiler:
- **Tek salon (Yanıt 1):** Çok işletmeli platform, işletme kaydı ve işletme seçimi kapsamdan çıktı. Apple'ın şablon uygulama kuralı (4.2.6) uygulamanın kimin adına yayımlanacağını bir iş sorusuna dönüştürüyor.
- **Hesap silme ve "hard delete olmamalı" (Yanıt 2):** Bu yanıt, KVKK ile Apple ve Google'ın hesap silme kurallarıyla **kısmen çelişiyor**. Apple yalnızca devre dışı bırakmayı açıkça "yetersiz" sayıyor. KVKK ise veriyi silmeyen veya anonim hâle getirmeyenler için ceza hükmü içeriyor. Bu dokümanın önerdiği çözüm şudur: Randevu **kayıtları** silinmez, "Hesap silindi" etiketiyle tutulur. Ancak kayıtlardaki **kişisel veriler** anonim hâle getirilir. Ayrıntı Bölüm 3.6'da, ilgili maddeler FR-29 ve NFR-14'te, hukuki teyit AS-13'tedir.
- **İptal nedeni (Yanıt 3):** İşletme iptal nedenini müşteriye her durumda uygulama içinde gösterir. Bildirimde ise yalnızca önceden tanımlı bir **neden kategorisi** gösterilir. Serbest metin açıklama, Apple 4.5.4'teki hassas bilgi uyarısı nedeniyle yalnızca uygulama içinde kalır.
- **Hizmet notu yok (Yanıt 4):** Serbest metin müşteri ve hizmet notu alanı kapsam dışıdır.
- **Geç alınan randevuda hatırlatma (Yanıt 5):** Hatırlatma zamanı geçmiş bir randevu için hatırlatma hemen gönderilir.

Araştırmadan çıkan başlıca bulgular önceki sürümle aynıdır. Kaynaklar bu çalışmada yeniden açılıp doğrulandı:
- **Sektördeki ortak çekirdek:** çevrimiçi randevu, çalışan bazlı takvim, otomatik hatırlatma ve iptal politikası.
- **Mağaza kuralları:** Apple ve Google, uygulama içinden hesap silmeyi zorunlu tutuyor. Google Play ayrıca web üzerinden silme talebi bağlantısı istiyor.
- **Bildirim izni:** Android 13 ve üzerinde, yeni kurulumda bildirimler varsayılan olarak kapalıdır.
- **KVKK:** Aydınlatma, ilgili kişi hakları ve 30 günlük başvuru cevap süresi yükümlülükleri geçerlidir.

İlk sürümün (MVP) sınırı, Fonksiyonel gereksinimler bölümündeki M öncelikli maddelerdir.

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Başlıca acı noktası |
|---|---|---|---|
| **P-1 Müşteri (Elif, 32)** | Salonun düzenli müşterisi. Telefonla aramak yerine uygulamadan randevu almak istiyor | Uygun saati görüp hızlıca randevu almak, gerekirse iptal etmek, randevuyu unutmamak, işletme iptal ederse nedenini öğrenmek | Mesai saatinde salonu arayamamak, randevuyu unutmak, salon iptal edince nedenini bilmemek |
| **P-2 Çalışan (Murat, kuaför)** | Salonda hizmet veren personel | Kendi günlük ve haftalık randevularını görmek, izin ve molalarını bilmek, yeni randevu ve iptallerden haberdar olmak | Çakışan randevular, son dakika iptallerinden geç haberdar olmak |
| **P-3 Salon sahibi veya yöneticisi (Ayşe)** | Salonu, hizmetleri ve çalışanları yöneten kişi | Hizmetleri ve süreleri tanımlamak, tüm çalışanların takvimini görmek, telefonla gelen randevuyu girmek, iptal kurallarını belirlemek, geçmiş randevu kaydını korumak | Boş kalan saatler, gelmeyen müşteriler (no-show), çift rezervasyon, hesabını silen müşteriyle birlikte geçmiş kaydın kaybolması |

Rakip incelemesi şu rol ayrımının yaygın olduğunu gösteriyor: Yönetici tüm çalışan takvimlerini görür, çalışan kendi randevularını yönetir. Booksy'de her çalışanın hizmet, çalışma saati ve kendi randevularını yönettiği bir profili vardır (K-3).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakipler ve mevcut çözümler

| Çözüm | Öne çıkan özellikler (kaynağa göre) | Güçlü yön | Zayıf yön / not |
|---|---|---|---|
| **Fresha** (uluslararası) | 7/24 çevrimiçi kendi kendine randevu. E-posta ve SMS ile otomatik hatırlatma. Esnek iptal politikası. Randevuda kapora veya tam ön ödeme (K-1). Fiyat: bağımsız (tek kişi) plan aylık 19,95 $, ekip planı ekip üyesi başına aylık 14,95 $. Ayda 100 ücretsiz SMS/WhatsApp mesajı (ekip planında üye başına). Sonrasında SMS başına 0,02 $, WhatsApp mesajı başına 0,04–0,11 $. Pazar yerinden gelen yeni müşteri için tek seferlik %20 komisyon, en az 6 $ (K-2) | Hatırlatma ve iptal politikası araçları hazır | SMS ve komisyon ek maliyet getiriyor. Ürün sayfası, işletmenin yaptığı iptalde müşterinin nasıl bilgilendirildiğini anlatmıyor. Ekip takvimi ayrıntısı sayfada yok (K-1) |
| **Booksy** (uluslararası) | Müşteri müsaitliği görüp 7/24 randevu alır. Her çalışanın hizmetlerini, çalışma saatlerini ve kendi randevularını yönettiği bir profili vardır. Sahip, web ve tablet uygulamasında çalışan takvimlerine erişir. Sınırsız otomatik hatırlatma. Bekleme listesi uyarısı. Fiyat: aylık 29,99 $, ek kullanıcı başına +20 $, vergi hariç (K-3). İşletme, "ücretsiz iptal veya yeniden planlama için ne kadar önceden haber verilmesi gerektiğini" belirler. Kayıtlı kartla geç iptal ve no-show ücreti, kapora ve güvenilir müşteri muafiyeti (K-4) | Çalışan ve yönetici rol ayrımı net. İptal penceresi kavramı açık | Pencere içindeki iptalin ücrete mi bağlandığı, yoksa tamamen mi engellendiği sayfada netleşmiyor. Metin ücret modelini ima ediyor (K-4) |
| **Vagaro** (uluslararası) | Web sitesi, Instagram, Facebook, Google ve Apple Haritalar üzerinden randevu. Otomatik hatırlatma. No-show ücreti veya kapora içeren iptal politikaları. İlk gelen, en yüksek değer veya özel kurala göre bildirim gönderen otomatik bekleme listesi. Özelleştirilebilir takvim. Tek şube için aylık 23,99 $'dan başlıyor (K-5) | Boş saatleri doldurmaya yönelik otomasyon | Sayfa hatırlatma kanallarını belirtmiyor |
| **Google Takvim randevu planları** (platform ekosistemindeki yerleşik çözüm) | Rezervasyon penceresi: en ileri tarih ve en kısa önceden bildirim süresi (varsayılan en az 4 saat). Randevular arası tampon süre. Günlük en fazla randevu sınırı. Belirli tarihler (ör. tatil) için müsaitlik değişikliği. En fazla 5 hatırlatma e-postası, metni düzenlenemez. Bazı özellikler uygun Workspace veya Google One aboneliği gerektiriyor (K-6) | Ek uygulama gerektirmiyor. Tampon ve önceden bildirim süresi gibi kavramlar iyi örnek | Sayfada çok çalışanlı salon ve hizmet-çalışan eşleştirmesi gibi sektöre özgü ihtiyaçlar yer almıyor |
| **Salon Randevu İşletme Yönetimi** (Türkiye) | Instagram, WhatsApp ve Google'da paylaşılan işletme bağlantısıyla çevrimiçi randevu. SMS ile randevu onayı ve hatırlatma. Adisyon, ürün ve paket satışı, gelir-gider, stok. Personel primi. Müşteri notları. Puan sistemi ve memnuniyet anketi. Mağaza açıklaması ücretsiz kullanım ve aylık ücret olmadığını söylüyor. Yeni üyeye ücretsiz Silver paket veriliyor (açıklamaya göre 100 SMS ve 50 randevu). Uygulama içinde aylık 4,99 $ ile 107,99 $ arasında ücretli paketler var. 1 değerlendirme (K-7) | Yerel dil. Geniş işletme işlevleri | Kullanıcı kanıtı zayıf (1 değerlendirme). "Ücretsiz" geliştiricinin tanıtım ifadesi; ücretli paketler de var |
| **Kuaförüm Yanımda** (Türkiye) | Otomatik onay ("Herhangi bir onaylamaya gerek kalmadan"). Toplu randevu onayı (sürüm 2.1.10). Tatil, personel izni ve yemek molası tanımlanarak çakışma önleniyor. Bildirim merkezi ve özelleştirilebilir bildirim içerikleri (sürüm 2.1.4). Müşteriler için "Hesap Sil" seçeneği (sürüm 2.1.4, 19 Ekim 2025). App Store (TR) puanı 4,6 (75 değerlendirme) (K-8) | Mola ve izin yönetimi, onay modları ve hesap silme, Türkiye pazarındaki beklentiyi gösteriyor | Mağaza sayfası iptal nedeninin müşteriye bildirilmesinden söz etmiyor. Yalnızca mağaza açıklaması incelendi |

### 3.2 Çıkarımlar (BA yorumu)
- **Sektördeki temel beklenti:** çevrimiçi randevu, çalışan bazlı takvim, mola ve izin tanımı, otomatik hatırlatma ve iptal kuralı. Bu ürün bunları M önceliğinde ele alır.
- **Ek modüller:** Kapora, iptal ücreti, pazar yeri, muhasebe ve stok olgun rakiplerde var. Görevde istenmedikleri için kapsam dışıdır. Salon Randevu'daki "müşteri notları" benzeri alan, paydaş kararıyla (Yanıt 4) bu üründe yer almaz.
- **İşletme iptalinde müşteri bilgilendirmesi:** İncelenen rakip sayfaları iptal nedeninin müşteriye nasıl iletildiğini anlatmıyor (K-1, K-8). Bu konudaki gereksinim (FR-13, FR-18) paydaş yanıtına ve mağaza kurallarına dayanır, rakip pratiğine değil.
- **Hatırlatma kanalı maliyeti:** Rakiplerde SMS ücretli veya kotalı (K-2, K-7). "Bildirim" bu dokümanda uygulama bildirimi (push) olarak yorumlanmıştır. SMS yedek kanal olarak C önceliğindedir.

### 3.3 Alan standartları ve düzenlemeler
- **KVKK (6698 sayılı Kanun, 2016 kabul metni; K-14):**
  - md. 3/1-b: Anonim hâle getirme, verinin "başka verilerle eşleştirilerek dahi hiçbir surette kimliği belirli veya belirlenebilir bir gerçek kişiyle ilişkilendirilemeyecek hâle getirilmesi"dir.
  - md. 4/2-d: Veriler, ilgili mevzuatta öngörülen veya işlendikleri amaç için gerekli olan süre kadar muhafaza edilir.
  - md. 5/2: Açık rıza aranmayan şartlar sayılır. Bunlar arasında sözleşmenin ifasıyla doğrudan ilgili olma (c), hukuki yükümlülük (ç) ve bir hakkın tesisi, kullanılması veya korunması (e) vardır. Randevu verisinin (c) şartına dayanabileceği bir BA yorumudur (V-07).
  - md. 6: Sağlık verisi özel nitelikli kişisel veridir.
  - md. 7/1: İşleme sebepleri ortadan kalkınca veri resen veya ilgili kişinin talebiyle "silinir, yok edilir veya anonim hâle getirilir". md. 7/2: Diğer kanunlardaki hükümler saklıdır.
  - md. 9: Yurt dışına aktarım şartları. İncelenen metin 2016 kabul metnidir; sonraki değişiklikler doğrulanmadı.
  - md. 10: aydınlatma yükümlülüğü. md. 11: ilgili kişinin hakları (11/1-e: silme veya yok edilmesini isteme).
  - md. 13/2: Başvurular "en geç otuz gün içinde ücretsiz olarak" sonuçlandırılır.
  - md. 16: Veri Sorumluları Sicili.
  - md. 17/2: 7. madde hükmüne aykırı olarak "kişisel verileri silmeyen veya anonim hâle getirmeyenler" TCK 138. maddeye göre cezalandırılır.
- **Kişisel Verilerin Silinmesi, Yok Edilmesi veya Anonim Hale Getirilmesi Hakkında Yönetmelik (Resmî Gazete, 28.10.2017; K-19):**
  - md. 4: İmha; silme, yok etme veya anonim hâle getirmeyi kapsar.
  - md. 5: Sicile kayıtlı veri sorumluları kişisel veri saklama ve imha politikası hazırlamakla yükümlüdür.
  - md. 8/1: Silme, verinin "ilgili kullanıcılar için hiçbir şekilde erişilemez ve tekrar kullanılamaz hale getirilmesi"dir.
  - md. 11: Periyodik imha süresi "her halde altı ayı geçemez".
  - md. 12: İşleme şartlarının tamamı ortadan kalkmışsa talep üzerine imha en geç 30 gün içinde yapılır. Şartlar tamamen ortadan kalkmamışsa talep gerekçesi bildirilerek reddedilebilir.
  - Yönetmelik 2019'da değiştirilmiştir. İncelenen metin 2017 tarihli ilk yayımdır; güncel konsolide metin bu çalışmada doğrulanmadı.
- **Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik md. 6 (K-13):**
  - md. 6/2: Abonelik, üyelik, satın alma, teslimat "veya benzeri durumlara ilişkin bildirimleri içeren iletiler" için önceden onay aranmaz. Şart: "bu tür bildirimlerde herhangi bir mal veya hizmet özendirilemez veya bunların tanıtımı yapılamaz."
  - md. 6/5: Bu iletiler için İYS kontrolü yapılmaz.
  - Randevu hatırlatması ve iptal bildiriminin bu kapsama girdiği bir BA yorumudur. Yönetmelikte "randevu" adıyla geçmiyor (V-06).
- **Erişilebilirlik (WCAG 2.2):**
  - 1.4.3: Metin kontrastı en az 4.5:1 (K-15).
  - 1.4.4: Metin %200'e kadar içerik veya işlev kaybı olmadan büyütülebilmeli (K-15).
  - 2.5.8: Dokunma hedefi en az 24×24 CSS piksel, AA düzeyi (K-16).
  - WCAG web içeriği için yazılmıştır. Mobil uygulamaya uyarlanması bir varsayımdır (V-10).

### 3.4 Platform ve mağaza politikaları (iş etkisi)

| Politika | İçerik | Ürüne iş etkisi |
|---|---|---|
| Apple 5.1.1(v) (K-9) | "If your app supports account creation, you must also offer account deletion within the app." Temel işlevle ilgisiz kişisel veri istenemez | Hesap silme M öncelikli (FR-03). Kayıtta yalnızca gerekli alanlar istenir (NFR-08) |
| Apple, "Offering account deletion in your app" (K-18) | Silme, hesabı geliştiricinin kayıtlarından ve "yasal olarak tutması gerekmeyen" ilişkili verilerden çıkarır. "Only offering to temporarily deactivate or disable an account is insufficient." Yasal saklama varsa kullanıcıya neyin tutulup neyin tutulmadığı söylenir. Silme manuel veya zaman alan bir süreçse ne kadar süreceği bildirilir | "Hard delete olmamalı" yanıtı, kişisel veri silinmeden yalnızca devre dışı bırakma olarak uygulanamaz (Bölüm 3.6). Silme ekranı, tutulan ve silinen verileri ve süreyi söylemelidir (FR-03, US-02) |
| Apple 5.1.1(i) (K-9) | Gizlilik politikası, veri saklama ve silme politikalarını ve kullanıcının nasıl silme talep edeceğini açıklamalıdır | Anonim tutulan randevu kayıtları gizlilik politikasında anlatılmalı (FR-02) |
| Apple 4.5.4 (K-9) | Push bildirimi uygulamanın çalışması için zorunlu olamaz. Push "should not be used to send sensitive personal or confidential information". Promosyon için açık onay ve vazgeçme yolu gerekir | Bildirim izni olmadan da akış çalışır (FR-17). İptal nedeninin serbest metni bildirimde gösterilmez, yalnızca kategori gösterilir (FR-18, NFR-11) |
| Apple 4.8 (K-9) | Birincil hesap için üçüncü taraf veya sosyal giriş kullanılırsa, veri toplamayı ad ve e-postayla sınırlayan ve e-postayı gizleme imkânı veren eşdeğer bir giriş seçeneği de sunulmalıdır | Sosyal giriş seçilirse ek giriş seçeneği gerekir (AS-02) |
| Apple 4.2.6 ve 4.3 (K-9) | Ticari bir şablon veya uygulama üretim servisiyle oluşturulan uygulamalar, içerik sağlayıcısı tarafından doğrudan gönderilmedikçe reddedilir. 4.3: Aynı uygulamanın birden çok Bundle ID ile gönderilmemesi | Tek salona ait uygulama (Yanıt 1), salonun kendi geliştirici hesabından yayımlanmalı. Aynı uygulama ileride başka salonlara da çoğaltılırsa ret riski doğar (AS-16) |
| Google Play hesap silme (K-10) | Uygulama içi silme yolu ve "a web link resource where users can request app account deletion and associated data deletion" zorunlu. "You must also delete the user data associated with that app account." Güvenlik, dolandırıcılığı önleme veya mevzuata uyum için tutulan veri kullanıcıya açıkça bildirilir | Web üzerinden silme talebi kanalı (FR-03). Anonimleştirilen kayıtların neden tutulduğu gizlilik politikasında açıklanır |
| Google Play Data safety (K-11) | Tüm uygulamalar formu doldurur. Üçüncü taraf kütüphane ve SDK'ların verileri de beyan edilir. Gizlilik politikası bağlantısı zorunlu. Form, kullanıcıya silme talep etme yolu sunulup sunulmadığını sorar. Veriyi 90 gün içinde otomatik silen veya anonimleştiren uygulama da silme mekanizması beyan edebilir | Veri envanteri ve silme ve anonimleştirme davranışı yayın öncesi hazırlanır (NFR-08, NFR-14) |
| Android 13+ bildirim izni (K-12) | `POST_NOTIFICATIONS` çalışma zamanı izni. Yeni kurulumda "notifications are off by default". İzin, işlevle ilgili bağlamda istenmeli (sayfadaki örneklerden biri: kullanıcının yemek siparişi vermesi) | İzin ilk randevu oluşturulurken gerekçesiyle istenir (FR-16). İzin vermeyen kullanıcı randevu bilgisini ve iptal nedenini uygulama içinde görür (FR-14, FR-17) |

iOS bildirim izni için Apple'ın ilgili dokümantasyonu bu çalışmada açılmadı (V-09).

### 3.5 Yöntemler (iş açısından artı/eksi, seçim TA'ya bırakılır)

| Konu | Seçenek | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Müsait saat üretimi** | Sabit aralıklı slot (ör. 15 dakikada bir) | Müşteri için anlaşılır, tahmin edilebilir saatler | Hizmet süresiyle uyumsuz kısa boşluklar kalabilir |
| | Hizmet süresi ve tampon süreye göre dinamik slot (Google Takvim'deki tampon kavramına benzer, K-6) | Takvim doluluğu artar | Kurgu ve test yükü ile yönetici ayarları artar |
| **Çakışma önleme** | Kayıt anında müsaitliğin yeniden kontrolü | Çift rezervasyon önlenir | Kaybeden müşteri "saat doldu" mesajı görür |
| **Randevu onayı** | Otomatik onay (Kuaförüm Yanımda'da var, K-8) | Müşteri anında kesin yanıt alır | Salon istenmeyen randevuyu engelleyemez |
| | Manuel onay | Salonun kontrolü artar | Müşteri bekler |
| **Hatırlatma kanalı** | Push | Mesaj başına maliyet yok (BA varsayımı) | İzin yoksa ulaşmaz. Android 13+ yeni kurulumda kapalı (K-12) |
| | SMS | İzinden bağımsız ulaşır | Mesaj başına maliyet (ör. Fresha'da kota sonrası 0,02 $, K-2). Telefon numarası gerekir |
| **Geç alınan randevuda hatırlatma** (Yanıt 5 ile karara bağlandı) | Hemen gönder | Müşteri, yakın randevu için de bir hatırlatma alır | Oluşturma bildirimiyle neredeyse aynı anda ikinci bir bildirim gider (AS-15) |
| **İptal nedeninin iletilmesi** (Yanıt 3) | Bildirimde serbest metin | Müşteri nedeni anında tam olarak görür | Kilit ekranında kişisel veya hassas bilgi görünebilir (ör. çalışanın sağlık durumu). Apple 4.5.4 hassas bilginin push ile gönderilmemesini söylüyor (K-9) |
| | Bildirimde önceden tanımlı kategori, serbest açıklama uygulama içinde (öneri) | Müşteri nedeni bildirimde görür. Hassas bilgi riski düşer | Ayrıntıyı görmek için uygulamayı açmak gerekir. Kategori listesinin bakımı gerekir |
| | Yalnızca uygulama içinde | En düşük gizlilik riski | Paydaşın "bildirimde olursa iyi olur" beklentisini karşılamaz |
| **İptal kuralı** | Randevudan X saat öncesine kadar serbest iptal | Son dakika iptali riskini azaltır (Booksy'de ücretle birlikte kullanılıyor, K-4) | Ücret olmadan pencere içinde iptali engellemek, müşteriyi haber vermeden gelmemeye itebilir |

**Hatırlatmanın etkisi:** Health Services Research'te 2012'de yayımlanan meta-analiz 18 çalışmayı inceledi: 8 randomize kontrollü çalışma (RKÇ) ve 10 kontrollü gözlemsel çalışma. Birinci basamak ve hastane polikliniklerindeki randevularda yalnızca 8 RKÇ birleştirildi. Birleştirilen sonuca göre SMS hatırlatma katılım olasılığını artırıyor (olasılık oranı OR 1,48). %95 güven aralığı özette 1,23–1,72, sonuçlar bölümünde 1,33–1,72 olarak geçiyor (K-17). Bu bulgu sağlık kliniklerine aittir. Salon müşterilerine ve push kanalına aktarılabilirliği bir varsayımdır (V-05).

### 3.6 Paydaş yanıtı ile mevzuat ve mağaza yükümlülükleri arasındaki gerilim (Yanıt 2)

**Paydaşın yanıtı:** "hesap silindi şeklinde tutulabilir. hard delete olmamalı"

**Çelişen yükümlülükler:**
- **KVKK:** Veriler amaç için gerekli süre kadar tutulur (md. 4/2-d). Sebep ortadan kalkınca silinir, yok edilir veya anonim hâle getirilir (md. 7/1). İlgili kişi silme isteyebilir (md. 11/1-e). Silmeyen veya anonim hâle getirmeyenler için ceza hükmü vardır (md. 17/2). Talep üzerine imha en geç 30 gün içinde yapılır (Yönetmelik md. 12) (K-14, K-19).
- **Apple:** Yalnızca geçici devre dışı bırakma veya hesabı kapatma seçeneği sunmak "insufficient" sayılır. Hesap kaydı, yasal olarak tutulması gerekmeyen ilişkili verilerle birlikte silinmelidir (K-18).
- **Google Play:** Hesap silindiğinde ilişkili kullanıcı verisi de silinmelidir. Güvenlik, dolandırıcılık önleme veya mevzuat nedeniyle tutulan veri kullanıcıya açıkça bildirilir (K-10).

**Sonuç (BA yorumu):** Yanıtın "kullanıcı kaydını 'silindi' bayrağıyla, kişisel verisiyle birlikte tutmak" olarak uygulanması bu yükümlülüklerle çelişir. Yanıtın "randevu kayıtları (satırlar) silinmesin, 'Hesap silindi' olarak görünsün" olarak uygulanması ise çelişmez. Bunun koşulu, bu kayıtlardaki kişisel verilerin anonim hâle getirilmesidir. KVKK anonim hâle getirmeyi silmeye eşdeğer bir imha yolu olarak sayar (md. 7/1). Apple ve Google ise yalnızca kişisel ve ilişkili veriyi hedef alır. Bu doküman, yanıtın ikinci anlamda verildiğini varsayar (V-13).

**Çözüm seçenekleri (iş açısından):**

| Seçenek | Tanım | İş açısından artı | İş açısından eksi / risk | Öneri |
|---|---|---|---|---|
| **A. Kaydı tut, kişisel veriyi anonimleştir** | Hesap, profil ve cihaz kayıtları silinir. Geçmiş randevu kayıtları tarih, saat, hizmet, çalışan ve durumla kalır. Müşteri adı yerine "Hesap silindi" yazılır. Ad, e-posta, telefon ve kullanıcı kimliği kayıttan çıkarılır. Gelecekteki randevular iptal edilir | Salonun takvim geçmişi ve hizmet ve çalışan bazlı sayıları korunur. "Hard delete olmamalı" beklentisi kayıt düzeyinde karşılanır. KVKK, Apple ve Google ile uyumlu görünür | Müşteri bazlı geçmiş (ör. gelmeme sayısı) kaybolur. Küçük bir salonda tarih, saat ve çalışan bilgisi başka bilgilerle birleştirilip kişiyi tanımlatabilir. KVKK tanımı "başka verilerle eşleştirilerek dahi" tanınamamayı istiyor (md. 3/1-b). Anonimliğin yeterliliği hukuken teyit edilmeli (AS-13) | **Önerilen** (FR-29, NFR-14) |
| **B. Süreli saklama, sonra anonimleştirme** | Yasal bir saklama yükümlülüğü varsa ilgili kişisel veri o süre boyunca erişimi kısıtlı tutulur, süre sonunda anonimleştirilir | Yasal yükümlülük (KVKK md. 5/2-ç) karşılanır | Bu ürünün görev kapsamında (ödeme ve fatura yok) böyle bir yükümlülük tespit edilmedi. Hukuki dayanak, saklama ve imha politikası, kullanıcıya bildirim (Apple, Google) ve periyodik imha (en fazla 6 ay, Yönetmelik md. 11) gerekir | Yalnızca hukuk danışmanı bir yükümlülük tespit ederse (AS-13) |
| **C. Yalnızca "silindi" işareti (soft delete), kişisel veri kalır** | Hesap kapatılır, veri olduğu gibi durur | Uygulaması en kolay seçenek. Tüm geçmiş müşteri bazında korunur | Apple bunu açıkça yetersiz sayıyor (K-18). Google ilişkili verinin silinmesini istiyor (K-10). KVKK md. 7 ve 17/2 riski var. Mağaza reddi ve idari ve cezai yaptırım riski | **Önerilmez** |

## 4. Fonksiyonel gereksinimler

**M önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S maddelerinin ilk sürümde olması beklenir ama ertelenebilir. C maddeleri kaynak kalırsa yapılır. Bu sürümde yapılmayacaklar (W) "Kapsam dışı maddeler" bölümündedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kullanıcı uygulamada hesap oluşturabilmeli ve giriş yapabilmelidir. Kayıtta yalnızca randevu için gerekli bilgiler istenir: ad, iletişim bilgisi (e-posta ve/veya telefon, bkz. AS-02) ve şifre ya da doğrulama kodu. Şifre sıfırlama veya yeniden doğrulama yolu bulunur. | M | Görev; Apple 5.1.1(v) |
| FR-02 | Kayıt sırasında KVKK aydınlatma metni ve gizlilik politikası gösterilmeli, bu metinlere uygulama içinden her zaman erişilebilmelidir. Aydınlatma metni şunları içerir: veri sorumlusunun kimliği, işleme amacı, aktarım alıcıları, toplama yöntemi ve hukuki sebep, md. 11 hakları. Gizlilik politikası ayrıca saklama ve silme politikasını açıklar. Buna hesap silindikten sonra randevu kayıtlarının anonim olarak tutulduğu bilgisi de dahildir. | M | KVKK md. 10; Apple 5.1.1(i); Google Play Data safety; Yanıt 2 |
| FR-03 | Kullanıcı hesabını uygulama içinden silebilmelidir. Uygulama dışından (web bağlantısı) silme talebi de gönderilebilmelidir. Silme şu sonuçları doğurur: hesap ve profil bilgileri silinir, gelecekteki randevular iptal edilir, geçmiş randevu kayıtları FR-29'a göre anonimleştirilerek tutulur. Silme ekranı, silinen ve anonim olarak tutulan verileri ve işlemin ne kadar süreceğini kullanıcıya söyler. Yalnızca devre dışı bırakma seçeneği sunulamaz. | M | Apple 5.1.1(v) ve hesap silme rehberi; Google Play hesap silme; KVKK md. 7; Yanıt 2 |
| FR-04 | Sistem üç rolü desteklemelidir: Müşteri, Çalışan, Salon yöneticisi. Her rol yalnızca yetkili olduğu verileri görür ve değiştirir (bkz. NFR-06). | M | Görev; Booksy rol ayrımı |
| FR-05 | Salon yöneticisi **salonun tek işletme profilini** tanımlayabilmelidir: ad, adres, iletişim, haftalık çalışma saatleri ve kapalı günler (resmî tatil dahil). Uygulama yalnızca bu salona hizmet verir. Birden çok işletme veya işletme kaydı yoktur. | M | Görev; Kuaförüm Yanımda; Yanıt 1 |
| FR-06 | Salon yöneticisi hizmetleri tanımlayabilmelidir: ad, süre (dakika), bilgi amaçlı fiyat ve hizmeti verebilen çalışanlar. Hizmete serbest metin not alanı eklenmez. | M | Rakip incelemesi; Yanıt 4 |
| FR-07 | Yönetici çalışan ekleyip çıkarabilmeli, çalışanı uygulamaya davet edebilmelidir. | M | Görev (çalışan takvimleri) |
| FR-08 | Her çalışan için haftalık çalışma saatleri, mola aralıkları ve izin günleri tanımlanabilmelidir. Yönetici tüm çalışanlar için bu tanımı yapar. Çalışanın kendi takvimini düzenleyebilmesi açık sorudur (AS-05). | M | Görev; Kuaförüm Yanımda; Booksy |
| FR-09 | Sistem, seçilen hizmet ve çalışan için müsait başlangıç saatlerini göstermelidir. Hesaplamada salon çalışma saatleri, çalışan saatleri, mola ve izinler, mevcut randevular ve hizmet süresi dikkate alınır. Hizmet süresi molaya veya kapanış saatine taşan saat gösterilmez. | M | Yöntemler 3.5 |
| FR-10 | Müşteri hizmeti, çalışanı ("fark etmez" seçeneği dahil) ve müsait saati seçerek randevu oluşturabilmelidir. Tek salon olduğu için işletme seçimi adımı yoktur. "Fark etmez" seçilirse randevu, hizmeti verebilen müsait çalışanlardan birine atanır (atama kuralı AS-04). | M | Görev; Yanıt 1 |
| FR-11 | Aynı çalışana aynı zaman aralığında birden fazla randevu kaydedilmemelidir. İki kullanıcı aynı slotu aynı anda seçerse yalnızca biri başarılı olur, diğerine güncel müsait saatler gösterilir. | M | Yöntemler 3.5; Kuaförüm Yanımda |
| FR-12 | Müşteri kendi randevusunu iptal edebilmelidir. Randevuya salonun belirlediği iptal süresinden (varsayılan 2 saat, ayarlanabilir) daha az kaldıysa uygulama, iptal yerine salonla iletişime geçme bilgisini gösterir (bkz. AS-03). İptal edilen saat yeniden müsait olur. | M | Görev; Booksy iptal penceresi |
| FR-13 | Yönetici ve ilgili çalışan randevuyu iptal edebilmelidir. İptalde önceden tanımlı listeden bir **iptal nedeni kategorisi** seçilir. Kategori seçimi zorunludur (BA önerisi, AS-14). Önerilen kategoriler: "Çalışan müsait değil", "Salon kapalı / beklenmeyen durum", "Müşterinin talebiyle", "Diğer". Kategoriler sağlık veya kişisel durum bilgisi içermez. İsteğe bağlı olarak kısa bir serbest metin açıklama da girilebilir. | M | Görev; Yanıt 3; Apple 4.5.4; KVKK md. 6 |
| FR-14 | Müşteri yaklaşan ve geçmiş randevularını durumlarıyla (aktif, iptal, tamamlandı, gelmedi) listeleyebilmelidir. Salon tarafından iptal edilen randevunun detayında iptal nedeni kategorisi ve varsa açıklama gösterilir. | M | Görev; Yanıt 3 |
| FR-15 | Çalışan kendi randevularını günlük ve haftalık takvim görünümünde görebilmelidir. Yönetici tüm çalışanların takvimini çalışan bazında (ör. sütunlar hâlinde) görebilmelidir. | M | Görev; Booksy |
| FR-16 | Uygulama bildirim iznini, ilk randevu oluşturma gibi işlevle ilgili bir anda ve gerekçesini açıklayarak istemelidir. İzin uygulama açılışında istenmez. | M | Android bildirim izni rehberi |
| FR-17 | Bildirim izni verilmese bile tüm randevu işlemleri çalışmalıdır. Randevu bilgisi ve iptal nedeni uygulama içinde görülebilmelidir. Bildirimleri kapalı kullanıcıya uygulama içinde bildirimleri açma yolu gösterilir. | M | Apple 4.5.4 |
| FR-18 | Müşteriye şu olaylarda bildirim gönderilmelidir: (a) randevu oluşturuldu; (b) randevu salon tarafından iptal edildi veya değiştirildi. İptal bildirimi iptal nedeni **kategorisini** içerir. Serbest metin açıklama bildirimde yer almaz; bildirim uygulamadaki randevu detayına yönlendirir. (c) Randevu hatırlatması (varsayılan olarak randevudan 24 saat önce). Randevu, hatırlatma zamanı geçmiş olacak kadar yakın bir zamanda oluşturulur veya taşınırsa (ör. 24 saatlik hatırlatmada randevudan 3 saat önce) hatırlatma **hemen** gönderilir. Bildirim içeriği tanıtım veya kampanya içermez. | M | Görev; Yanıt 3; Yanıt 5; Ticari Elektronik İletiler Yönetmeliği md. 6/2 |
| FR-19 | İlgili çalışana ve yöneticiye şu olaylarda bildirim gönderilmelidir: yeni randevu ve müşteri iptali. Hesap silme nedeniyle yapılan iptaller de müşteri iptali gibi bildirilir. | M | Görev; Booksy; Yanıt 2 |
| FR-20 | Yönetici veya çalışan, telefonla veya salonda alınan randevuyu müşteri adına sisteme girebilmelidir. Müşterinin uygulama hesabı olmayabilir; bu durumda ad ve telefon girilir. | S | Rakip incelemesi (takvimin tek kaynak olması) |
| FR-21 | Müşteri, iptal süresi dolmamış randevusunu başka bir saate taşıyabilmelidir (yeniden planlama). Eski randevu, ancak yeni saat başarıyla alındığında serbest bırakılır. | S | Booksy (yeniden planlama) |
| FR-22 | Yönetici şu randevu kurallarını ayarlayabilmelidir: en kısa önceden randevu süresi, en ileri randevu tarihi (ör. 60 gün), hizmetler arası tampon süre ve iptal süresi. | S | Google Takvim randevu planları |
| FR-23 | Yönetici veya çalışan, gelmeyen müşterinin randevusunu "gelmedi" (no-show) olarak işaretleyebilmelidir. Yönetici müşteri bazında gelmeme sayısını görebilmelidir. Hesabı silinen müşterinin sayacı FR-29 kapsamında silinir. | S | Rakiplerde no-show odaklı özellikler; Yanıt 2 |
| FR-24 | Hatırlatma zamanı salon tarafından ayarlanabilmelidir (ör. 24 saat ve/veya 2 saat önce). Birden çok hatırlatma zamanının randevu oluşturulurken geçmiş olduğu durumda yalnızca bir hatırlatma hemen gönderilir. Zamanı henüz gelmemiş hatırlatmalar zamanında gider (BA önerisi, V-14). | S | Google Takvim (çoklu hatırlatma); Yanıt 5 |
| FR-25 | Yönetici randevu onay modunu seçebilmelidir: otomatik onay (varsayılan) veya manuel onay. Manuel modda müşteri "onay bekliyor" durumunu görür, onay veya ret olduğunda bildirim alır. | C | Kuaförüm Yanımda |
| FR-26 | Dolu bir saat için müşteri bekleme listesine yazılabilmeli, saat boşalırsa bildirim alabilmelidir. | C | Booksy; Vagaro |
| FR-27 | Push bildirimi alamayan müşteriye SMS veya e-posta ile yedek hatırlatma gönderilebilmelidir. | C | Fresha; Salon Randevu; maliyet AS-06 |
| FR-28 | Müşteri randevusunu cihazının takvimine ekleyebilmelidir. | C | BA önerisi (kaynaksız; kullanıcı kolaylığı) |
| FR-29 | **Yeni.** Hesabı silinen müşterinin geçmiş randevu kayıtları silinmez; salon takviminde ve raporlarında **"Hesap silindi"** etiketiyle tutulur. Bu kayıtlardan müşteriye ait kişisel tanımlayıcılar çıkarılır: ad, e-posta, telefon, kullanıcı kimliği ve müşteri bazlı gelmeme sayacı. Kayıtta tarih, saat, hizmet, çalışan ve durum kalır (Bölüm 3.6, Seçenek A). Yasal bir saklama yükümlülüğü tespit edilirse uygulanacak süre ve kapsam AS-13'e bağlıdır. | M | Yanıt 2; KVKK md. 7/1, md. 3/1-b; Apple hesap silme rehberi; Google Play hesap silme |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt |
|---|---|---|
| NFR-01 | **Platform:** Uygulama Android ve iOS telefonlarda aynı işlevleri sunar. | Tüm M gereksinimleri iki platformda kabul testinden geçer. Desteklenen en düşük işletim sistemi sürümleri açık sorudur (AS-07). |
| NFR-02 | **Çift rezervasyon olmaması** | En az 50 eşzamanlı deneme içeren testlerde aynı çalışan ve zaman aralığına kaydedilen randevu sayısı en fazla 1'dir. |
| NFR-03 | **Yanıt süresi** | 4G bağlantıda, çalışan veya tarih seçiminden sonra müsait saat listesi 95. persentilde 2 saniye içinde görüntülenir. Randevu oluşturma onayı 3 saniye içinde gösterilir. |
| NFR-04 | **Bildirim zamanlaması** | Olay bildirimleri (oluşturma, iptal) ve hemen gönderilen hatırlatmalar (FR-18 c) olaydan sonra 1 dakika içinde gönderilir. Planlı hatırlatmalar planlanan zamandan en fazla ±5 dakika sapmayla gönderilir. Ölçüm gönderim anına göre yapılır; cihaza teslim işletim sistemine bağlıdır. |
| NFR-05 | **Kullanılabilirlik** | Kayıtlı bir müşteri, uygulamayı açtıktan sonra en fazla 5 ekran veya adımda randevu oluşturabilir. İlk kez kullanan 5 test katılımcısının en az 4'ü randevu oluşturmayı yardımsız 3 dakikada tamamlar. |
| NFR-06 | **Yetkilendirme ve gizlilik** | Müşteri yalnızca kendi randevu ve profil verisini görür. Çalışan, müşterinin yalnızca randevu için gerekli bilgilerini (ad, iletişim, hizmet) görür. Yetkisiz erişim testlerinde başarılı erişim sayısı 0'dır. |
| NFR-07 | **KVKK başvuruları ve hesap silme süresi** | İlgili kişinin bilgi, düzeltme ve silme talepleri en geç 30 gün içinde sonuçlandırılır (KVKK md. 13/2; Yönetmelik md. 12). Uygulama içi silme talebinden sonra hesap hemen kullanılamaz hâle gelir. Kişisel verilerin silinmesi ve anonimleştirilmesi (FR-29) en geç 30 gün içinde tamamlanır. Silme ekranında bu süre kullanıcıya gösterilir (Apple hesap silme rehberi). |
| NFR-08 | **Veri en aza indirme** | Kayıtta zorunlu alan sayısı en fazla 3'tür (ad, iletişim bilgisi, şifre veya doğrulama). Ürün serbest metin müşteri veya hizmet notu toplamaz. Mağaza veri beyanları (Data safety ve App Store gizlilik bilgileri) üçüncü taraf kütüphanelerin topladığı veriler dahil yayın öncesi hazırlanır. |
| NFR-09 | **Erişilebilirlik** | Metin kontrastı en az 4.5:1'dir. Sistem yazı boyutu en büyük ayardayken (veya %200 büyütmede) içerik kesilmez. Dokunma hedefleri WCAG 2.5.8 hedefinden (24×24 CSS piksel) küçük olmaz; mobil eşdeğerini TA belirler. Ana ekranlar ekran okuyucuyla gezilebilir. |
| NFR-10 | **Dil, saat ve tarih** | Arayüz Türkçedir. Saatler salonun yerel saat diliminde (varsayılan Türkiye saati) 24 saat biçiminde gösterilir. Tarih biçimi GG.AA.YYYY'dir. |
| NFR-11 | **Bildirim içeriği** | Bildirimler yalnızca şu bilgileri içerir: salon, hizmet, tarih, saat, çalışan ve iptal bildiriminde iptal nedeni kategorisi (FR-13). Serbest metin alanı (ör. iptal açıklaması) bildirim gövdesinde yer almaz. Tanıtım veya kampanya içeriği yoktur. Test: tanımlı tüm bildirim türlerinin metinlerinde serbest metin veya kampanya içeriği sayısı 0'dır. |
| NFR-12 | **Erişilebilirlik süresi (uptime)** | Randevu alma hizmeti aylık en az %99,5 erişilebilir olur. Hedefin niteliği (iç hedef mi, sözleşme mi) AS-10'da teyit edilecektir. |
| NFR-13 | **Bağlantı kesintisi** | İnternet yokken uygulama son görüntülenen randevu listesini gösterir ve yeni randevu veya iptal işleminin yapılamadığını açık bir mesajla bildirir. Çevrimdışıyken işlem kuyruğa alınmaz (BA önerisi; çakışma riski nedeniyle). |
| NFR-14 | **Yeni. Anonimleştirmenin doğrulanması** | Hesap silme tamamlandıktan sonra, silinen kullanıcının adı, e-postası, telefonu veya kullanıcı kimliğiyle yönetici ve çalışan ekranlarında, aramada ve dışa aktarımda yapılan sorgular 0 eşleşme döndürür. Aynı kullanıcının silme öncesindeki geçmiş randevu sayısı, salon takviminde "Hesap silindi" etiketli kayıt sayısına eşittir. |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Kayıt ve giriş** (FR-01, FR-02)
Müşteri olarak hesap oluşturmak istiyorum; böylece randevularımı uygulamadan yönetebilirim.
- **KK-1:** Kayıt ekranında aydınlatma metni ve gizlilik politikası bağlantısı görünür. Kayıt, zorunlu alanlar (en fazla 3) doldurulunca tamamlanır.
- **KK-2:** Kayıtlı kullanıcı uygulamayı yeniden açtığında oturumu sürer veya giriş yapabilir.
- **KK-3:** Şifresini unutan kullanıcı sıfırlama veya doğrulama yoluyla hesabına erişir.

**US-02: Hesap silme** (FR-03, FR-29, NFR-07, NFR-14)
Müşteri olarak hesabımı ve kişisel verilerimi silmek istiyorum.
- **KK-1:** Profil ayarlarında "Hesabı sil" seçeneği vardır ve talep en fazla 3 adımda tamamlanır. Yalnızca "hesabı dondur" veya "devre dışı bırak" seçeneği sunulmaz.
- **KK-2:** Silme talebinden sonra hesapla giriş yapılamaz. Gelecekteki randevular iptal edilir ve salona iptal bildirimi gider (FR-19). Geçmiş randevular salon kayıtlarında "Hesap silindi" etiketiyle, ad ve iletişim bilgisi olmadan görünür (FR-29).
- **KK-3:** Uygulamaya erişimi olmayan kullanıcı, mağaza sayfasındaki web bağlantısından silme talebi gönderebilir.
- **KK-4:** Onay ekranı şunları açıkça yazar: hangi verilerin silineceği, randevu kayıtlarının anonim olarak tutulacağı ve işlemin en geç ne zaman tamamlanacağı (en fazla 30 gün).
- **KK-5:** Silme tamamlandıktan sonra NFR-14'teki aramalar 0 eşleşme döndürür. Müşterinin gelmeme sayacı artık görünmez.

**US-03: Salon ve hizmet tanımı** (FR-04, FR-05, FR-06, FR-07)
Salon yöneticisi olarak çalışma saatlerimi, hizmetlerimi ve çalışanlarımı tanımlamak istiyorum.
- **KK-1:** Yönetici bir hizmeti ad, süre ve fiyatla oluşturur ve en az bir çalışana atar. Hizmet müşteri tarafında görünür. Hizmet formunda serbest metin not alanı yoktur.
- **KK-2:** Kapalı olarak işaretlenen günde müşteriye müsait saat gösterilmez.
- **KK-3:** Çalışan rolündeki kullanıcı hizmet veya fiyat değiştiremez.
- **KK-4:** Uygulamada tek bir salon profili vardır. İkinci bir işletme oluşturma seçeneği yoktur.

**US-04: Çalışan takvimi, mola ve izin** (FR-08, FR-15)
Çalışan olarak çalışma saatlerimi, molalarımı ve izinlerimi görmek ve günlük randevularımı takip etmek istiyorum.
- **KK-1:** Çalışan için 12:30-13:30 molası tanımlandığında bu aralıkla çakışan hiçbir slot müşteriye gösterilmez.
- **KK-2:** İzin günü tanımlanan çalışan, o gün seçilebilir müsait çalışanlar arasında görünmez.
- **KK-3:** Çalışan günlük ve haftalık görünümünde yalnızca kendi randevularını görür. Yönetici tüm çalışanların randevularını çalışan bazında görür.

**US-05: Randevu oluşturma** (FR-09, FR-10, FR-11, FR-18)
Müşteri olarak uygun bir saati seçip randevu almak istiyorum.
- **KK-1:** Müşteri uygulamayı açınca doğrudan salonun hizmetlerini görür; işletme seçme adımı yoktur. 45 dakikalık bir hizmet seçtiğinde, kapanışa veya molaya 45 dakikadan az kalan başlangıç saatleri listelenmez.
- **KK-2:** Randevu oluşturulunca onay ekranı gösterilir. Bildirim izni varsa onay bildirimi 1 dakika içinde gönderilir (NFR-04).
- **KK-3:** İki müşteri aynı slotu aynı anda onaylarsa yalnızca biri başarılı olur. Diğeri "Bu saat az önce doldu" mesajı ve güncel saat listesi görür (NFR-02).
- **KK-4:** "Fark etmez" seçeneğinde randevu, hizmeti verebilen ve o saatte müsait bir çalışana atanır. Atanan çalışanın adı onayda gösterilir.

**US-06: Randevu iptali (müşteri)** (FR-12, FR-14, FR-19)
Müşteri olarak gidemeyeceğim randevuyu iptal etmek istiyorum.
- **KK-1:** İptal süresi dolmamış randevuda "İptal et" seçeneği çalışır. Randevu durumu "iptal" olur ve saat diğer müşterilere müsait görünür.
- **KK-2:** İlgili çalışana ve yöneticiye iptal bildirimi 1 dakika içinde gönderilir.
- **KK-3:** İptal süresi dolmuşsa uygulama, iptal butonu yerine salonun iletişim bilgisini ve iptal kuralını gösterir.

**US-07: Salon tarafından iptal** (FR-13, FR-14, FR-17, FR-18, NFR-11)
Yönetici olarak, bir çalışan gelemediğinde randevuları iptal edip müşterileri nedeniyle birlikte bilgilendirmek istiyorum.
- **KK-1:** Yönetici veya ilgili çalışan, listeden bir iptal nedeni kategorisi seçmeden iptali tamamlayamaz. Kısa açıklama girmek isteğe bağlıdır.
- **KK-2:** Bildirim izni olan müşteriye 1 dakika içinde iptal bildirimi gönderilir. Bildirim, salon, hizmet, tarih, saat ve iptal nedeni kategorisini içerir (ör. "İptal nedeni: Çalışan müsait değil"). Serbest metin açıklama bildirimde yer almaz.
- **KK-3:** Müşterinin randevu listesinde randevu "salon tarafından iptal edildi" olarak görünür. Detayda kategori ve varsa açıklama gösterilir. Bildirim izni olmayan müşteri de bu bilgiyi uygulama içinde görür.
- **KK-4:** Kategori listesinde sağlık durumu veya kişisel durum belirten bir seçenek yoktur.

**US-08: Hatırlatma ve bildirim izni** (FR-16, FR-17, FR-18, FR-24)
Müşteri olarak randevumu unutmamak için hatırlatma almak istiyorum.
- **KK-1:** Bildirim izni uygulama ilk açıldığında değil, ilk randevu oluşturulurken gerekçesiyle istenir.
- **KK-2:** İzin veren müşteri randevudan 24 saat önce (±5 dakika) hatırlatma alır. Hatırlatma salon, hizmet, tarih, saat ve çalışan bilgisini içerir ve kampanya içermez.
- **KK-3:** İzin vermeyen müşteri randevu oluşturabilir ve randevusunu listede görür. Uygulamada bildirimleri açma yolu gösterilir.
- **KK-4:** İptal edilen randevu için hatırlatma gönderilmez.
- **KK-5:** Hatırlatma süresi 24 saat iken randevudan 3 saat önce oluşturulan randevu için hatırlatma, oluşturmadan sonra 1 dakika içinde gönderilir. Aynı kural, randevu yakın bir saate taşındığında da geçerlidir (FR-21).
- **KK-6:** Salon 24 saat ve 2 saat önce hatırlatma ayarlamışsa: Randevudan 3 saat önce oluşturulan randevuya 24 saatlik hatırlatma hemen, 2 saatlik hatırlatma zamanında gider. Randevudan 1 saat önce oluşturulan randevuya yalnızca bir hatırlatma hemen gider.

**US-09: Telefonla gelen randevuyu girme** (FR-20)
Yönetici olarak telefonla arayan müşterinin randevusunu sisteme girmek istiyorum; böylece takvim tek kaynak olur.
- **KK-1:** Yönetici, hesabı olmayan müşteri için ad ve telefonla randevu oluşturabilir. Slot diğer müşterilere dolu görünür.
- **KK-2:** Çakışan bir saate randevu girilmek istendiğinde sistem engeller (FR-11).

**US-10: Yeniden planlama** (FR-21)
Müşteri olarak randevumu iptal edip yeniden almak yerine başka bir saate taşımak istiyorum.
- **KK-1:** Yeni saat başarıyla alınmadan eski randevu iptal edilmez.
- **KK-2:** Taşımadan sonra çalışana değişiklik bildirimi gider.

**US-11: Randevu kuralları ve no-show** (FR-22, FR-23)
Yönetici olarak son dakika randevularını ve gelmeyen müşterileri yönetmek istiyorum.
- **KK-1:** En kısa önceden randevu süresi 2 saat olarak ayarlandığında, şu andan itibaren 2 saatten yakın slotlar müşteriye gösterilmez.
- **KK-2:** "Gelmedi" olarak işaretlenen randevu müşterinin gelmeme sayısına eklenir. Yönetici bu sayıyı müşteri detayında görür.

**US-12: Hesabı silinen müşterinin geçmiş kayıtları** (FR-29, NFR-14) *(yeni)*
Salon yöneticisi olarak, müşteri hesabını silse de geçmiş takvimimin eksiksiz kalmasını istiyorum; böylece geçmiş günlerin doluluğunu ve çalışanların yaptığı işleri görebilirim.
- **KK-1:** Hesabı silinen müşterinin geçmiş randevuları takvimde tarih, saat, hizmet, çalışan ve durumla görünmeye devam eder. Müşteri alanında "Hesap silindi" yazar.
- **KK-2:** Bu kayıtlarda ad, telefon, e-posta veya müşteri detayına giden bağlantı yoktur.
- **KK-3:** Silme öncesinde ve sonrasında, ilgili tarih aralığındaki toplam randevu sayısı aynıdır.

## 7. Kapsam dışı maddeler

Bu sürümde yapılmayacak maddeler (MoSCoW: W):
- **Çok işletmeli platform:** birden çok salonun aynı uygulamayı kullanması, işletme kaydı ve onayı, işletme seçimi veya katılım kodu (Yanıt 1).
- **Çok şubeli salon yönetimi** ve şubeler arası raporlama.
- **Serbest metin müşteri ve hizmet notu alanı** (Yanıt 4). Salon Randevu gibi rakiplerde müşteri notu var (K-7). Bu üründe yoktur; sağlık verisi toplanması riski de böylece azalır.
- **Hesap silmede yalnızca "silindi" işaretiyle kişisel verinin tutulması** (Bölüm 3.6, Seçenek C). Mağaza politikalarıyla çeliştiği için uygulanmaz.
- **Bildirimde serbest metin iptal açıklaması.** Açıklama yalnızca uygulama içinde gösterilir (FR-18). Paydaş farklı karar verirse AS-14'e bakınız.
- **Çevrimiçi ödeme, kapora, ön ödeme ve iptal ücreti** (Fresha, Booksy ve Vagaro'da var). Görevde istenmedi. Ödeme altyapısı ve ek mevzuat getirir.
- **Pazar yeri ve keşif** (Fresha pazar yeri modeli).
- **Pazarlama ve kampanya bildirimleri, toplu SMS.** Apple 4.5.4'e göre açık onay ve vazgeçme yolu ister; Türkiye'de İYS süreçleri gerektirir.
- **Muhasebe, adisyon, stok, personel primi** (Salon Randevu'da var).
- **Müşteri değerlendirme ve puanlama, sadakat ve puan sistemi.**
- **Web üzerinden randevu sayfası ve sosyal medya entegrasyonları** (Vagaro'da var). Hesap silme için gereken web bağlantısı (FR-03) bunun istisnasıdır.
- **Tablet ve masaüstü yönetim paneli.**
- **Gelişmiş raporlama ve analitik.**

## 8. Varsayımlar

- **V-01 (Yanıt 1 ile güncellendi):** Uygulama tek bir salona aittir. Müşteri uygulamayı mağazadan indirir ve doğrudan bu salonun hizmetlerini görür. İşletme bağlantısı, kodu veya seçimi yoktur. İlk salon yöneticisi hesabının kurulum sırasında oluşturulacağı varsayılmıştır; bu bir müşteri akışı değildir.
- **V-02:** Flutter ve Firebase ekibin verdiği kısıtlardır. Firebase'in hangi servislerinin kullanılacağı TA'nın kararıdır.
- **V-03:** "Bildirim" uygulama içi ve push bildirimi olarak yorumlanmıştır. SMS ve e-posta yedek kanaldır (FR-27, C).
- **V-04:** Varsayılan değerler (24 saatlik hatırlatma, 2 saatlik iptal süresi, 60 günlük ileri tarih) BA önerisidir ve rakip pratiklerinden esinlenmiştir. Kesin değerler doğrulanmadı.
- **V-05:** Hatırlatmanın no-show'u azaltacağı varsayımı, sağlık kliniklerindeki SMS meta-analizine dayanır (8 RKÇ, OR 1,48). Salon sektörüne ve push kanalına aktarılabilirliği doğrulanmadı.
- **V-06:** Randevu hatırlatmaları ve iptal bildirimlerinin Ticari Elektronik İletiler Yönetmeliği md. 6/2 kapsamında önceden onay gerektirmediği bir BA yorumudur. Hukuki teyit gerekir.
- **V-07:** Randevu verisinin KVKK md. 5/2-c (sözleşmenin ifası) şartına dayanarak açık rıza olmadan işlenebileceği bir BA yorumudur. Hukuki teyit gerekir.
- **V-08 (Yanıt 4 ile güncellendi):** Paydaş hizmet notu alanını istemedi. Bu nedenle üründe serbest metin müşteri veya hizmet notu yoktur ve sağlık verisi (KVKK md. 6) toplanması öngörülmez. Üründe kalan tek serbest metin alanı, salonun isteğe bağlı iptal açıklamasıdır (FR-13). Bu alana sağlık bilgisi yazılmaması için arayüzde uyarı gösterilmesi BA önerisidir.
- **V-09:** iOS'ta bildirim izninin Android'e benzer şekilde kullanıcı onayı gerektirdiği varsayılmıştır. Apple'ın ilgili sayfası bu çalışmada açılmadığı için doğrulanmadı.
- **V-10:** WCAG 2.2 web içeriği için yazılmıştır. Mobil uygulamada eşdeğer hedef olarak kullanılması bir varsayımdır.
- **V-11:** Platformun yerleşik çözümü olarak Google Takvim randevu planları incelendi. iOS ve Android'de salon randevusuna özgü yerleşik bir çözüm için ayrıca kaynak aranmadı.
- **V-12:** Çocuklara yönelik özel düzenlemeler değerlendirilmedi. Hedef kitle yetişkin müşteriler ve salon çalışanlarıdır.
- **V-13 (yeni, Yanıt 2):** "Hesap silindi şeklinde tutulabilir. hard delete olmamalı" ifadesi, randevu **kayıtlarının** silinmemesi olarak yorumlanmıştır, **kişisel verinin** tutulması olarak değil. Kişisel veriler anonim hâle getirilir (Bölüm 3.6, Seçenek A). Paydaş kişisel verinin de tutulmasını kastediyorsa Bölüm 3.6'daki çelişki geçerlidir (AS-13).
- **V-14 (yeni, Yanıt 5):** "Hemen gönderilir" şöyle yorumlanmıştır: Hatırlatma, randevu oluşturulduktan veya taşındıktan sonra 1 dakika içinde (NFR-04) ve oluşturma bildiriminden ayrı olarak gönderilir. Manuel onay modunda (FR-25) "hemen", onay anı olarak kabul edilmiştir. Birden çok hatırlatmanın zamanı geçmişse yalnızca bir hatırlatma gönderilmesi BA önerisidir; amaç müşteriye art arda aynı içerikte bildirim gitmesini önlemektir.
- **V-15 (yeni, Yanıt 3):** "Müşteriye bir şekilde bildirilmeli" ifadesi, iptal nedeninin uygulama içinde her durumda gösterilmesi olarak uygulanmıştır. "Bildirimde olursa iyi olur" ifadesi ise bildirimde yalnızca kategori gösterilmesi olarak uygulanmıştır. Önerilen kategori listesi BA önerisidir.
- **V-16 (yeni, Yanıt 1):** Tek salon modelinde veri sorumlusunun salon işletmesi olacağı tahmin edilmektedir. Bu bir varsayımdır; teyit AS-09'dadır.

## 9. Açık sorular

Cevaplanan AS-01 (tek salon) bu listeden çıkarıldı. Önceki sürümdeki diğer ID'ler korunmuştur.

- **AS-02:** Müşteri kaydında birincil kimlik e-posta mı, telefon numarası mı olacak? Telefon doğrulaması SMS maliyeti getirir. Sosyal giriş (Google, Apple) isteniyor mu? İstenirse Apple 4.8 ek bir giriş seçeneği gerektirir.
- **AS-03:** İptal süresi dolduktan sonra müşteri iptali tamamen engellenmeli mi, yoksa iptal edilip salona "geç iptal" olarak mı işaretlenmeli?
- **AS-04:** "Fark etmez" seçildiğinde çalışan nasıl atanmalı: en az randevulu çalışan mı, sırayla mı, ilk müsait çalışan mı?
- **AS-05:** Çalışan kendi çalışma saatlerini ve izinlerini kendisi düzenleyebilmeli mi, yoksa yalnızca yönetici mi? (Booksy'de çalışan kendi saatlerini ve randevularını yönetebiliyor, K-3.)
- **AS-06:** SMS yedek hatırlatma (FR-27) ve takvime ekleme (FR-28) ilk sürüm planına alınacak mı? SMS isteniyorsa maliyeti kim üstlenecek, aylık bütçe nedir?
- **AS-07:** Desteklenen en düşük Android ve iOS sürümleri nelerdir? İş açısından daha yüksek bir taban kabul edilebilir mi?
- **AS-08:** Kişisel veriler Türkiye dışında barındırılacaksa KVKK md. 9 kapsamında hangi hukuki dayanak kullanılacak? İncelenen kanun metni 2016 kabul metnidir. Güncel md. 9 hükmü doğrulanmadı ve hukuk danışmanıyla teyit edilmelidir.
- **AS-09 (Yanıt 1 ile daraltıldı):** Tek salon modelinde veri sorumlusu salon işletmesi mi olacak (V-16)? Uygulamayı geliştiren veya işleten taraf veri işleyen mi olacak? Salonun Veri Sorumluları Siciline kayıt yükümlülüğü (KVKK md. 16) ve buna bağlı saklama ve imha politikası yükümlülüğü (Yönetmelik md. 5) var mı?
- **AS-10:** Erişilebilirlik süresi hedefi (NFR-12, %99,5) ve yanıt süresi hedefleri (NFR-03) salonun beklentisine uygun mu? %99,5 bir sözleşme yükümlülüğü mü, iç hedef mi, ölçümden kim sorumlu?
- **AS-11:** Bir hizmet birden çok çalışan veya kaynak gerektiriyor mu (ör. belirli bir koltuk veya oda)? İlk sürümde tek çalışan ve tek hizmet varsayılmıştır.
- **AS-12:** Müşteri bir randevuda birden çok hizmet seçebilmeli mi (ör. kesim + fön)?
- **AS-13 (yeni, Yanıt 2):** "Hard delete olmamalı" yanıtı için:
  - (a) Kayıtları tutup kişisel veriyi anonimleştirme yaklaşımı (Bölüm 3.6, Seçenek A) paydaş tarafından onaylanıyor mu?
  - (b) Hukuk danışmanı, tutulan kaydın (tarih, saat, hizmet, çalışan) küçük bir salonda KVKK md. 3/1-b anlamında anonim sayılıp sayılmadığını teyit etmeli.
  - (c) Salonun bu kayıtlar için yasal bir saklama yükümlülüğü var mı? Varsa süre ve kapsam nedir (Seçenek B)? Böyle bir yükümlülük bu çalışmada tespit edilmedi.
- **AS-14 (yeni, Yanıt 3):** Önerilen iptal nedeni kategorileri ("Çalışan müsait değil", "Salon kapalı / beklenmeyen durum", "Müşterinin talebiyle", "Diğer") ve kategorinin zorunlu olması uygun mu? Paydaş serbest metin açıklamanın da bildirimde görünmesini isterse, kilit ekranında hassas bilgi görünme riskini ve Apple 4.5.4'teki uyarıyı kabul ediyor mu?
- **AS-15 (yeni, Yanıt 5 ve TA S-06):**
  - (a) Hemen gönderilen hatırlatma, oluşturma bildirimiyle tek bildirimde birleştirilsin mi, yoksa ayrı mı gitsin? Bu doküman ayrı gönderimi varsaymıştır (V-14).
  - (b) Salon hatırlatma zamanını değiştirdiğinde mevcut randevuların hatırlatmaları da yeni ayara göre güncellensin mi?
- **AS-16 (yeni, Yanıt 1):** Uygulama App Store ve Google Play'de kimin geliştirici hesabıyla yayımlanacak? Apple 4.2.6'ya göre şablondan üretilen uygulamalar içerik sağlayıcısı (salon) tarafından gönderilmelidir. Aynı uygulamanın ileride başka salonlar için çoğaltılması planlanıyor mu? Bu durumda 4.2.6 ve 4.3 ret riski doğar.
- **AS-17 (yeni, TA S-05):** Müsait saatlerin başlangıç aralığı (ör. 15 dakika) salon tarafından ayarlanabilir olmalı mı, sabit mi kalmalı?
- **AS-18 (yeni, TA S-11):** Hesapsız müşteri adına girilen randevularda (FR-20) bu müşterinin telefon numarası için aydınlatma nasıl yapılacak? Saklama süresi nedir? Bu kişilerin silme talepleri hangi kanaldan alınacak?

## 10. Kaynaklar

Tüm kaynaklar bu çalışmada açılıp okunmuştur. Önceki sürümün K-1–K-17 numaraları korunmuştur.

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K-1 | Fresha for business | https://www.fresha.com/for-business | "get your clients self-booking online 24/7"; "automated booking reminders via email and text"; esnek iptal politikası; kapora veya tam ön ödeme. İşletme iptalinde müşteri bilgilendirmesi sayfada yok |
| K-2 | Fresha Pricing | https://www.fresha.com/pricing | Bağımsız plan 19,95 $/ay; ekip üyesi başına 14,95 $/ay; ayda 100 ücretsiz SMS/WhatsApp (ekipte üye başına); SMS başına 0,02 $, WhatsApp 0,04–0,11 $; yeni müşteride %20 tek seferlik komisyon, en az 6 $ |
| K-3 | Booksy Features | https://biz.booksy.com/en-us/features | "Clients can view your availability and conveniently book appointments 24/7"; çalışan profili ile hizmet, çalışma saati ve kendi randevularını yönetme; "Access staff calendars"; "Unlimited automatic client appointment reminders are included"; bekleme listesi; 29,99 $/ay + ek kullanıcı başına 20 $ |
| K-4 | Booksy No-Show Protection | https://biz.booksy.com/en-us/features/no-show-protection | "how much advance notice is required for no-fee cancellations or rescheduling"; kayıtlı kartla iptal ücreti; kapora; güvenilir müşteri muafiyeti |
| K-5 | Vagaro Salon Software | https://www.vagaro.com/pro/salon-software | Çok kanallı randevu; otomatik hatırlatma; no-show ücreti veya kapora içeren iptal politikaları; otomatik bekleme listesi; tek şube için 23,99 $/ay |
| K-6 | Google Calendar Help: Appointment schedules | https://support.google.com/calendar/answer/10729749 | Rezervasyon penceresi (varsayılan en kısa önceden bildirim süresi 4 saat); "Add time between appointments on your calendar"; günlük en fazla randevu; en fazla 5 hatırlatma e-postası, metni düzenlenemez; bazı özellikler ücretli abonelik gerektirir |
| K-7 | Salon Randevu İşletme Yönetimi (App Store, ABD) | https://apps.apple.com/us/app/id1413695973 | Bağlantıyla çevrimiçi randevu; SMS onay ve hatırlatma; müşteri notları; prim, stok, gelir-gider; geliştiriciye göre ücretsiz kullanım, aylık ücret yok; ücretsiz Silver paket (100 SMS, 50 randevu); uygulama içi ücretli paketler; 1 değerlendirme |
| K-8 | Kuaförüm Yanımda (App Store, TR) | https://apps.apple.com/tr/app/kuaf%C3%B6r%C3%BCm-yan%C4%B1mda/id1516248277?l=tr | "Herhangi bir onaylamaya gerek kalmadan"; toplu onay (2.1.10); izin ve molayla çakışma önleme; bildirim merkezi ve "Bildirim içerikleri özelleştirilebilir hale getirildi" (2.1.4); "Hesap Sil" (2.1.4, 19 Ekim 2025); 4,6 puan, 75 değerlendirme; iptal nedeni bildirimi sayfada yok |
| K-9 | App Review Guidelines (Apple) | https://developer.apple.com/app-store/review/guidelines/ | 5.1.1(v): "If your app supports account creation, you must also offer account deletion within the app"; 5.1.1(i): gizlilik politikası "Explain its data retention/deletion policies"; 4.5.4: "should not be used to send sensitive personal or confidential information"; 4.8 eşdeğer giriş seçeneği; 4.2.6: "Apps created from a commercialized template or app generation service will be rejected unless they are submitted directly by the provider of the app's content"; 4.3(a) birden çok Bundle ID yasağı |
| K-10 | Google Play: Understanding account deletion requirements | https://support.google.com/googleplay/android-developer/answer/13327111 | "provide users with an in-app path to delete their app accounts and associated data"; "provide a web link resource where users can request app account deletion and associated data deletion"; "you must also delete the user data associated with that app account"; güvenlik, dolandırıcılık önleme veya mevzuat nedeniyle saklanan veri kullanıcıya açıkça bildirilir |
| K-11 | Google Play: Data safety section | https://support.google.com/googleplay/android-developer/answer/10787469 | "All developers that have an app published on Google Play must complete the Data safety form"; üçüncü taraf kod verisi beyan edilir; gizlilik politikası zorunlu; "Does your app provide a way for users to request deletion of their data?"; veriyi 90 gün içinde silen veya anonimleştiren uygulama silme mekanizması beyan edebilir |
| K-12 | Android Developers: Notification runtime permission | https://developer.android.com/develop/ui/views/notifications/notification-permission | Android 13 (API 33) `POST_NOTIFICATIONS`; yeni kurulumda "notifications are off by default"; "Request the permission in context"; örnek: "The user submits an order for food delivery" |
| K-13 | Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik (Lexpera) | https://www.lexpera.com.tr/mevzuat/yonetmelikler/ticari-iletisim-ve-ticari-elektronik-iletiler-hakkinda-yonetmelik | md. 6/2: "satın alma ve teslimat veya benzeri durumlara ilişkin bildirimleri içeren iletiler" için onay aranmaz; "Ancak bu tür bildirimlerde herhangi bir mal veya hizmet özendirilemez veya bunların tanıtımı yapılamaz."; md. 6/5: "İYS üzerinden kontrol yapılmaz" |
| K-14 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (Adalet Bakanlığı, 2016 kabul metni PDF) | https://mgm.adalet.gov.tr/Resimler/SayfaDokuman/51120191031036698%20KVKK.pdf | md. 3/1-b anonim hâle getirme tanımı ("başka verilerle eşleştirilerek dahi"); md. 4/2-d; md. 5/2 (c), (ç), (e); md. 6; md. 7/1: "silinir, yok edilir veya anonim hâle getirilir"; md. 7/2; md. 9; md. 10; md. 11/1-e; md. 13/2: "en geç otuz gün içinde ücretsiz olarak sonuçlandırır"; md. 16; md. 17/2: "kişisel verileri silmeyen veya anonim hâle getirmeyenler" TCK 138'e göre cezalandırılır |
| K-15 | WCAG 2.2 (W3C) | https://www.w3.org/TR/WCAG22/ | 1.4.3: "a contrast ratio of at least 4.5:1"; 1.4.4: "resized without assistive technology up to 200 percent without loss of content or functionality" |
| K-16 | Understanding SC 2.5.8 Target Size (Minimum) (W3C) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | "The size of the target for pointer inputs is at least 24 by 24 CSS pixels" (istisnalarla), AA düzeyi |
| K-17 | How Effective Are Short Message Service Reminders at Increasing Clinic Attendance? A Meta-Analysis and Systematic Review (Health Services Research, 2012) | https://pmc.ncbi.nlm.nih.gov/articles/PMC3419880/ | 18 çalışma (8 RKÇ, 10 kontrollü gözlemsel); birinci basamak ve hastane poliklinikleri; RKÇ'lerde birleştirilmiş OR 1,48; güven aralığı özette 1,23–1,72, sonuçlarda 1,33–1,72; yazarlar: "substantially increase the likelihood of attending" |
| K-18 | Apple Developer: Offering account deletion in your app | https://developer.apple.com/support/offering-account-deletion-in-your-app/ | "Deleting an account removes the account from the developer's records, along with any data associated with the account that the developer isn't legally required to maintain."; "only offering to temporarily deactivate or disable an account is insufficient."; "If you're legally required to retain some user information, let people know what information will and won't be retained."; manuel veya zaman alan silmede "Inform the person how long it will take" |
| K-19 | Kişisel Verilerin Silinmesi, Yok Edilmesi veya Anonim Hale Getirilmesi Hakkında Yönetmelik (Resmî Gazete, 28.10.2017, 2017 ilk yayım metni) | https://www.resmigazete.gov.tr/eskiler/2017/10/20171028-10.htm | md. 4: imha tanımı (silme, yok etme veya anonim hâle getirme); md. 5: sicile kayıtlı veri sorumlusunun saklama ve imha politikası yükümlülüğü; md. 8/1: silme "ilgili kullanıcılar için hiçbir şekilde erişilemez ve tekrar kullanılamaz hale getirilmesi"; md. 10: anonimleştirme; md. 11: periyodik imha "her halde altı ayı geçemez"; md. 12: talep üzerine imha "en geç otuz gün içinde", şartlar tamamen ortadan kalkmamışsa gerekçeli ret. 2019 değişiklikleri sonrası konsolide metin doğrulanmadı |

**Açılamayan veya kullanılmayan kaynaklar (kaynak gösterilmedi):**
- Apple "Asking permission to use notifications" sayfası bu çalışmada açılmadı (V-09).
- KVKK Kurumu (kvkk.gov.tr) ve Lexpera'daki silme yönetmeliği sayfaları arama sonuçlarında görüldü. Açılmadıkları için kaynak gösterilmedi; yerine Resmî Gazete metni (K-19) kullanıldı.
- WCAG 2.2 ana sayfası 2.5.8 maddesine kadar okunamadı. Bu madde için K-16 kullanıldı.
- Önceki sürümde açılamayan Square Appointments, Booksy destek makalesi ve BMJ Open 2016 meta-analizi bu çalışmada yeniden denenmedi.

## 11. Değişiklik notu

Önceki sürüme göre (outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run1.md) yapılan değişiklikler aşağıdadır. Hiçbir FR/NFR/US ID'si kaldırılmadı veya yeniden numaralandırılmadı. Yeni maddeler sıradaki ID'yi aldı.

| # | Neden | Değişiklik | Etkilenen ID / bölüm |
|---|---|---|---|
| D-01 | Yanıt 1 (tek salon) | Ürün tek salona ait olarak tanımlandı. Özet ve persona'lar buna göre güncellendi | Bölüm 1, 2 |
| D-02 | Yanıt 1 | FR-05 tek işletme profili olarak yeniden yazıldı. FR-10'dan işletme seçimi çıkarıldı | FR-05, FR-10 |
| D-03 | Yanıt 1 | US-03'e KK-4 (tek salon profili) eklendi. US-05 KK-1'e işletme seçimi adımının olmaması eklendi | US-03, US-05 |
| D-04 | Yanıt 1 | V-01 yeniden yazıldı. V-16 eklendi. AS-01 açık sorulardan çıkarıldı. AS-09 tek salon modeline göre daraltıldı | V-01, V-16, AS-01, AS-09 |
| D-05 | Yanıt 1 | Çok işletmeli platform kapsam dışına eklendi. Apple 4.2.6 ve 4.3 politika satırı ve AS-16 eklendi | Bölüm 3.4, 7, AS-16 |
| D-06 | Yanıt 2 (hard delete olmamalı) | Mevzuat ve mağaza politikasıyla çelişki analizi ve A/B/C çözüm seçenekleri eklendi. Seçenek A önerildi | Bölüm 3.6 (yeni) |
| D-07 | Yanıt 2 | FR-03 güncellendi: geçmiş kayıtların anonim tutulması, silme ekranında süre ve kapsamın bildirilmesi, yalnızca devre dışı bırakmanın yetersizliği | FR-03 |
| D-08 | Yanıt 2 | Yeni FR-29 eklendi ("Hesap silindi" etiketli anonim kayıt, M). FR-02 (saklama ve silme politikası), FR-19 ve FR-23 (sayacın silinmesi) güncellendi | FR-29, FR-02, FR-19, FR-23 |
| D-09 | Yanıt 2 | NFR-07'ye 30 günlük silme ve anonimleştirme süresi ile kullanıcıya süre bildirimi eklendi. Yeni NFR-14 (anonimleştirme doğrulaması) eklendi | NFR-07, NFR-14 |
| D-10 | Yanıt 2 | US-02'ye KK-1 ek cümlesi, KK-2 güncellemesi, KK-4 ve KK-5 eklendi. Yeni US-12 eklendi | US-02, US-12 |
| D-11 | Yanıt 2 | V-13 ve AS-13 eklendi. Yeni kaynaklar K-18 (Apple hesap silme rehberi) ve K-19 (silme yönetmeliği) eklendi. K-14'e md. 3/1-b, 4/2-d, 7/2 ve 17/2 eklendi. K-9, K-10 ve K-11 yeni alıntılarla genişletildi | V-13, AS-13, Bölüm 3.3, 10 |
| D-12 | Yanıt 3 (iptal nedeni) | FR-13 önceden tanımlı ve zorunlu iptal nedeni kategorisi ile isteğe bağlı açıklama olarak yeniden yazıldı. FR-14'e iptal nedeninin uygulama içinde gösterimi, FR-17'ye izin yokken iptal nedeninin görünürlüğü eklendi | FR-13, FR-14, FR-17 |
| D-13 | Yanıt 3 | FR-18 (b): iptal bildiriminde kategori gösterilir, serbest metin gösterilmez. NFR-11 bu kurala ve ölçülebilir bir teste göre yeniden yazıldı | FR-18, NFR-11 |
| D-14 | Yanıt 3 | US-07'nin KK-1–KK-4 maddeleri yeniden yazıldı. Hikâye metninde "hastalandığında" ifadesi "gelemediğinde" yapıldı (sağlık verisi örneğinden kaçınmak için). Bölüm 3.5'e iptal nedeni seçenekleri, V-15 ve AS-14 eklendi | US-07, Bölüm 3.5, V-15, AS-14 |
| D-15 | Yanıt 4 (hizmet notu yok) | V-08 yeniden yazıldı. FR-06'ya not alanı olmadığı, NFR-08'e serbest metin not toplanmadığı eklendi. NFR-11'deki "hizmet notları" örneği çıkarıldı. US-03 KK-1 güncellendi. Kapsam dışına not alanı eklendi | V-08, FR-06, NFR-08, NFR-11, US-03, Bölüm 7 |
| D-16 | Yanıt 5 (hatırlatma hemen) | FR-18 (c)'ye geç alınan veya taşınan randevuda hatırlatmanın hemen gönderimi eklendi. FR-24'e çoklu hatırlatma kuralı eklendi. NFR-04'e hemen gönderilen hatırlatma için 1 dakika süresi eklendi | FR-18, FR-24, NFR-04 |
| D-17 | Yanıt 5 | US-08'e KK-5 ve KK-6 eklendi. Bölüm 3.5'e ilgili satır, V-14 ve AS-15 eklendi (AS-15 TA S-06'nın cevaplanmamış ikinci kısmını da içerir) | US-08, V-14, AS-15 |
| D-18 | TA'nın cevaplanmamış soruları | S-05, S-11 ayrı açık soru olarak eklendi (AS-17, AS-18). S-07, S-12 ve S-13 sırasıyla AS-10, AS-07 ve AS-06'ya eklendi. S-02, S-04 ve S-10 mevcut AS-02–AS-05 ve AS-08 olarak açık kalmaya devam ediyor | AS-06, AS-07, AS-10, AS-17, AS-18 |
| D-19 | Kaynakların yeniden doğrulanması | Tüm kaynaklar bu çalışmada yeniden açıldı. Güncellenen bilgiler: Vagaro fiyatı (23,99 $/ay) eklendi; Fresha'nın ekip takvimi ayrıntısı doğrulanamadığı için tablodan çıkarıldı; Fresha WhatsApp fiyatı ve en az 6 $ komisyon eklendi; Google Takvim'in varsayılan 4 saatlik en kısa önceden bildirim süresi eklendi; Booksy'de çalışanın kendi saatlerini yönetmesi eklendi (AS-05'e not); Salon Randevu'nun müşteri notu ve paket bilgileri eklendi | Bölüm 3.1, 10 |
| D-20 | Bütünlük | NFR-01 ölçütü "FR-01 ile FR-19 arasındaki M gereksinimleri" yerine "tüm M gereksinimleri" olarak değiştirildi (yeni FR-29 M önceliğinde olduğu için) | NFR-01 |
