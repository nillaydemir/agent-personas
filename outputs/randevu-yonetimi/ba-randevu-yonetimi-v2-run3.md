# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: İş Analizi Dokümanı (Güncelleme 2)

- **Tarih:** 2026-11-24
- **Persona sürümü:** ba v2
- **Girdi:** Flutter ile Android ve iOS cihazlarda çalışacak bir randevu yönetim uygulaması hazırlayacağız. Firebase kullanarak kullanıcı kaydı, randevu oluşturma, iptal etme, çalışan takvimleri ve bildirim özelliklerini değerlendireceğiz. Güzellik salonları ve kuaförler için mevcut uygulamaları araştırarak iş gerekliliklerini belirle ve bize bir iş analizi dokümanı hazırla.
  - Önceki BA dokümanı: outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run2.md
  - TA'nın bu dokümana dair geri soruları: outputs/randevu-yonetimi/ta-randevu-yonetimi-v2-run2.md ("12. BA'ya geri sorular" bölümü)
  - Paydaşın (kullanıcının) yeni yanıtları, kendi ifadesiyle ("A seçeneği olsun, kategoriler uygun, salon, ayarlanabilir"):
    1. AS-13 (hesap silmede çözüm seçeneği) → "A seçeneği olsun" (kaydı tut, kişisel veriyi anonimleştir)
    2. AS-14 (önerilen 4 iptal nedeni kategorisi ve zorunlu olması) → "kategoriler uygun"
    3. AS-16 (uygulama mağazalarda kimin geliştirici hesabıyla yayımlanacak) → "salon"
    4. AS-17 (müsait saat başlangıç aralığı sabit mi, salon tarafından ayarlanabilir mi) → "ayarlanabilir"
  - Önceki dokümanı ve TA'nın sorularını okuyup bu yanıtları işleyerek güncellenmiş, eksiksiz (tek başına okunabilir) bir BA dokümanı üret. Yanıtların gereksinimlere, kabul kriterlerine, varsayımlara ve açık sorulara etkisini yansıt; cevaplanan soruları açık sorulardan çıkar. TA'nın geri sorularından BA'nın kendi yetkisiyle netleştirebileceği belirsizlikleri (ör. ifade tutarsızlıkları, tanımsız kalan atıflar) gider; paydaş kararı gerektirenleri açık sorulara yaz. Paydaşın bu turda yanıtlamadığı konuları (ör. iptal açıklamasının karakter sınırı) cevaplanmış sayma. Mevcut FR/NFR/US ID'lerini koru (yeniden numaralandırma yapma); kaldırılan bir madde olursa ID'sini boş bırakıp nedenini belirt, yeni madde gerekirse sıradaki ID'yi kullan. Dokümanın "Değişiklik notu" bölümünü önceki sürüme (run2) göre neyin hangi yanıt veya TA sorusu nedeniyle değiştiğini listeleyecek şekilde yeniden yaz.
  - Ek talimat (aynı tur): Paydaş bu tur için 3 yanıt daha verdi; bunları da aynı dokümana (run3) işle. Paydaşın kendi ifadesi: "fatura salon, 200 karakter uygun, sadece uygulama içi yeterli".
    1. Firebase projesinin ve faturanın (Blaze planı) sahibi kim olacak (TA S-15)? → "salon"
    2. İptal açıklaması için 200 karakter üst sınırı uygun mu (TA S-16)? → "200 karakter uygun"
    3. Google Play hizmetleri olmayan Android cihazlarda FCM çalışmıyor; bu kullanıcıların bilgiyi yalnızca uygulama içinde görmesi kabul edilebilir mi (TA S-10)? → "sadece uygulama içi yeterli"
    - Önceki talimattaki "iptal açıklamasının karakter sınırı yanıtlanmadı" notu artık geçersiz. Bu yanıtları gereksinimlere, kabul kriterlerine, varsayımlara, açık sorulara ve Değişiklik notuna yansıt.

---

## 1. Özet

Ürün, **tek bir güzellik salonuna veya kuaföre ait** bir mobil randevu uygulamasıdır. Müşteriler bu salondan Android ve iOS uygulaması üzerinden randevu alır. Görevde istenen beş ana yetenek şunlardır: kullanıcı kaydı, randevu oluşturma, randevu iptali, çalışan takvimleri ve bildirimler. Flutter ve Firebase ekibin görevde verdiği kısıtlardır. Bu doküman bunların üzerine ek bir teknoloji kararı vermez.

**Yanıt numaralandırması:** Önceki turdaki paydaş yanıtları dokümanda **Yanıt 1–5** olarak geçer: tek salon, hesap silindi olarak tutma, iptal nedeninin bildirilmesi, hizmet notu olmaması, hatırlatmanın hemen gönderilmesi. Bu turun yanıtları **Yanıt 6–12** olarak numaralandırılmıştır:

| Yanıt | Soru | Paydaşın ifadesi |
|---|---|---|
| Yanıt 6 | AS-13: hesap silmede çözüm seçeneği | "A seçeneği olsun" |
| Yanıt 7 | AS-14: iptal nedeni kategorileri ve zorunluluk | "kategoriler uygun" |
| Yanıt 8 | AS-16: mağazalarda kimin geliştirici hesabıyla yayın | "salon" |
| Yanıt 9 | AS-17: başlangıç aralığı sabit mi, ayarlanabilir mi | "ayarlanabilir" |
| Yanıt 10 | TA S-15: Firebase projesi ve faturanın sahibi | "salon" ("fatura salon") |
| Yanıt 11 | TA S-16: iptal açıklaması için 200 karakter sınırı | "200 karakter uygun" |
| Yanıt 12 | TA S-10: Google Play hizmetleri olmayan cihazlarda yalnızca uygulama içi bilgi | "sadece uygulama içi yeterli" |

Bu sürüm, önceki BA dokümanını paydaşın 7 yeni yanıtına ve TA'nın geri sorularına göre günceller. Ayrıntılı liste 11. bölümdedir (Değişiklik notu). Başlıca etkiler:
- **Hesap silmede Seçenek A onaylandı (Yanıt 6):** Randevu kayıtları "Hesap silindi" etiketiyle tutulur, kişisel veriler anonim hâle getirilir (FR-29). Açık kalan konu, anonimliğin hukuken yeterli olup olmadığı ve yeterli bulunmazsa ne yapılacağıdır (AS-13).
- **İptal nedeni ve açıklaması (Yanıt 7, Yanıt 11):** Dört kategori ve kategori seçiminin zorunlu olması onaylandı. İsteğe bağlı açıklama en fazla 200 karakterdir (FR-13, US-07). Serbest metnin bildirimde gösterilmemesi kararının açık onayı beklenmektedir (AS-14).
- **Yayıncı ve sahiplik salonda (Yanıt 8, Yanıt 10):** Uygulama salonun geliştirici hesaplarıyla yayımlanır. Firebase projesi ve faturası da salona aittir (NFR-15). Bu durum Apple 4.2.6'daki şablon uygulama riskini azaltır. Ancak salonun hukuki yapısı açılacak hesap türünü belirler:
  - Apple, şahıs işletmesinin bireysel hesapla kaydolmasını istiyor. Bu durumda mağazada satıcı olarak kişinin yasal adı görünür (K-20).
  - Google Play'de kurumsal hesap D-U-N-S numarası gerektiriyor ve bu numaranın alınması 30 güne kadar sürebiliyor (K-21).
  - 13 Kasım 2023'ten sonra açılan bireysel Google hesapları, yayından önce en az 12 test kullanıcısıyla 14 günlük kapalı test yapmak zorunda (K-22).
  - Bu konular AS-16'da açıktır.
- **Başlangıç saati aralığı ayarlanabilir (Yanıt 9):** Salon, müsait saatlerin başlangıç aralığını seçer. Varsayılan 15 dakikadır (FR-09, FR-22, US-11 KK-3).
- **Push alamayan cihazlar (Yanıt 12):** Google Play hizmetleri olmayan Android cihazlardaki kullanıcılar bilgiyi yalnızca uygulama içinden alır. Bu cihazlar için yedek kanal gerekmez (FR-17, V-19).
- **TA sorularıyla giderilen tutarsızlıklar:**
  - FR-18'deki "salon tarafından değiştirildi" bildirimine karşılık gelen bir FR yoktu. Bu ifade çıkarıldı. Salonun randevuyu değiştirmesi kapsam dışına ve açık sorulara (AS-20) yazıldı.
  - FR-29 ve NFR-14'teki tanımsız "rapor, arama, dışa aktarım" atıfları mevcut ekranlara bağlandı.
  - Müşterinin randevu taşıması FR-19'a eklendi. Böylece US-10 KK-2 bir FR'ye dayanıyor.

Araştırmanın başlıca bulguları önceki sürümle aynıdır. Kaynaklar bu çalışmada yeniden açılıp doğrulandı:
- **Sektördeki ortak çekirdek:** çevrimiçi randevu, çalışan bazlı takvim, otomatik hatırlatma ve iptal politikası.
- **Hesap silme:** Apple ve Google uygulama içinden hesap silmeyi zorunlu tutuyor. Google Play ayrıca web üzerinden silme talebi bağlantısı istiyor.
- **Bildirim izni:** Android 13 ve üzerinde yeni kurulumda bildirimler varsayılan olarak kapalıdır.
- **KVKK:** Aydınlatma, ilgili kişi hakları ve 30 günlük cevap süresi yükümlülükleri geçerlidir.

İlk sürümün (MVP) sınırı, Fonksiyonel gereksinimler bölümündeki M öncelikli maddelerdir.

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Başlıca acı noktası |
|---|---|---|---|
| **P-1 Müşteri (Elif, 32)** | Salonun düzenli müşterisi. Telefonla aramak yerine uygulamadan randevu almak istiyor | Uygun saati görüp hızlıca randevu almak, gerekirse iptal etmek, randevuyu unutmamak, salon iptal ederse nedenini öğrenmek | Mesai saatinde salonu arayamamak, randevuyu unutmak, salon iptal edince nedenini bilmemek |
| **P-2 Çalışan (Murat, kuaför)** | Salonda hizmet veren personel | Kendi günlük ve haftalık randevularını görmek, izin ve molalarını bilmek, yeni randevu, taşıma ve iptallerden haberdar olmak | Çakışan randevular, son dakika değişikliklerinden geç haberdar olmak |
| **P-3 Salon sahibi veya yöneticisi (Ayşe)** | Salonu, hizmetleri ve çalışanları yöneten kişi. Uygulama salonun adına mağazalarda yayımlanır, altyapı faturası da salona aittir (Yanıt 8, Yanıt 10) | Hizmetleri ve süreleri tanımlamak, tüm çalışanların takvimini görmek, telefonla gelen randevuyu girmek, iptal ve randevu kurallarını (başlangıç aralığı dahil) belirlemek, geçmiş randevu kaydını korumak | Boş kalan saatler, gelmeyen müşteriler (no-show), çift rezervasyon, hesabını silen müşteriyle birlikte geçmiş kaydın kaybolması |

Rakip incelemesi şu rol ayrımının yaygın olduğunu gösteriyor: Yönetici tüm çalışan takvimlerini görür, çalışan kendi randevularını yönetir. Booksy'de her çalışanın hizmetleri ve çalışma saatleri için bir profili vardır. Çalışan bu profille kendi randevularını görüp yönetir (K-3).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakipler ve mevcut çözümler

| Çözüm | Öne çıkan özellikler (kaynağa göre) | Güçlü yön | Zayıf yön / not |
|---|---|---|---|
| **Fresha** (uluslararası) | Müşteriler 7/24 çevrimiçi kendi randevusunu alır. E-posta ve SMS ile otomatik hatırlatma. Esnek iptal politikası. Randevu anında kapora veya tam ön ödeme (K-1). Fiyat: bağımsız plan aylık 19,95 $ (tek kişi), ekip planı ekip üyesi başına aylık 14,95 $. Ayda 100 ücretsiz SMS/WhatsApp (ekip planında üye başına). Kota sonrası SMS başına 0,02 $, WhatsApp mesajı başına 0,04–0,11 $. Pazar yerinden gelen yeni müşteri için tek seferlik %20 komisyon, en az 6 $ (K-2) | Hatırlatma ve iptal politikası araçları hazır | SMS ve komisyon ek maliyet getiriyor. Ürün sayfası, işletmenin yaptığı iptalde müşterinin nasıl bilgilendirildiğini anlatmıyor (K-1) |
| **Booksy** (uluslararası) | Müşteri müsaitliği görüp 7/24 randevu alır. Her çalışanın hizmetleri ve çalışma saatleri için bir profili vardır, çalışan kendi randevularını görüp yönetir. Sahip, web ve tablet sürümünde çalışan takvimlerine erişir. Sınırsız otomatik hatırlatma. Bekleme listesi bildirimi. Fiyat: aylık 29,99 $ + vergi, ek kullanıcı başına aylık 20 $ (K-3). Ücretsiz iptal veya yeniden planlama için gereken önceden bildirim süresini işletme belirler. Kayıtlı kartla geç iptal ve no-show ücreti, kapora, güvenilir müşteri muafiyeti (K-4) | Çalışan ve yönetici rol ayrımı net. İptal penceresi kavramı açık | Pencere içindeki iptalin ücrete bağlandığı görülüyor. Tamamen engellenip engellenmediği sayfada yok (K-4) |
| **Vagaro** (uluslararası) | Web sitesi, Instagram, Facebook, Google ve Apple Haritalar üzerinden randevu. Otomatik hatırlatma. No-show ücreti veya zorunlu kapora içeren iptal politikaları. Doğru müşterilere otomatik bildirim gönderen bekleme listesi. Özelleştirilebilir takvim. Tek şube için aylık 23,99 $ taban ücret (K-5) | Boş saatleri doldurmaya yönelik otomasyon | Sayfa hatırlatma kanallarını belirtmiyor |
| **Google Takvim randevu planları** (platform ekosistemindeki yerleşik çözüm) | Rezervasyon penceresi: en ileri tarih ve en kısa önceden bildirim süresi (varsayılan 4 saat). Randevular arası tampon süre. Günlük en fazla randevu sınırı. Belirli tarihler (ör. tatil) için müsaitlik değişikliği. En fazla 5 hatırlatma e-postası, metni düzenlenemez. Bazı özellikler uygun Workspace veya Google One aboneliği gerektiriyor (K-6) | Ek uygulama gerektirmiyor. Tampon ve önceden bildirim süresi gibi kavramlar iyi örnek | Sayfada çok çalışanlı salon ve hizmet-çalışan eşleştirmesi gibi sektöre özgü ihtiyaçlar yok |
| **Salon Randevu İşletme Yönetimi** (Türkiye) | Instagram, WhatsApp ve Google'da paylaşılan işletme bağlantısıyla çevrimiçi randevu. SMS ile onay ve hatırlatma. Müşteri notları. Personel primi, stok, gelir-gider. Mağaza açıklamasına göre aylık ücret yok. Yeni üyeye 100 SMS ve 50 randevu içeren ücretsiz Silver paket veriliyor. Uygulama içi satın almalar 4,99 $ (ör. Barkod eki) ile 949,00 $ (yıllık Diamond üyelik) arasında. 1 değerlendirme (K-7) | Yerel dil. Geniş işletme işlevleri | Kullanıcı kanıtı zayıf (1 değerlendirme). "Ücretsiz" geliştiricinin tanıtım ifadesi, ücretli paketler de var |
| **Kuaförüm Yanımda** (Türkiye) | Otomatik onay ("Herhangi bir onaylamaya gerek kalmadan"). Toplu randevu onayı (sürüm 2.1.10). Tatil, personel izni ve yemek molası tanımlanarak çakışma önleniyor. Sürüm 2.1.4'te (19.10.2025) eklenenler: bildirim merkezi, özelleştirilebilir bildirim içerikleri, müşteriler için "Hesap Sil" ve "Müşterilerin iptal yetkisi salon tarafından yönetilebiliyor". App Store (TR) puanı 4,6 (75 değerlendirme) (K-8) | Mola ve izin yönetimi, onay modları, hesap silme ve salonun iptal yetkisini yönetmesi Türkiye pazarındaki beklentiyi gösteriyor | Mağaza sayfası iptal nedeninin müşteriye bildirilmesinden söz etmiyor. Yalnızca mağaza açıklaması incelendi |

### 3.2 Çıkarımlar (BA yorumu)
- **Sektördeki temel beklenti:** çevrimiçi randevu, çalışan bazlı takvim, mola ve izin tanımı, otomatik hatırlatma ve iptal kuralı. Bu ürün bunları M önceliğinde ele alır.
- **Ek modüller:** Kapora, iptal ücreti, pazar yeri, muhasebe ve stok olgun rakiplerde var. Görevde istenmedikleri için kapsam dışıdır. Salon Randevu'daki müşteri notları benzeri bir alan, paydaş kararıyla (Yanıt 4) bu üründe yer almaz.
- **İşletme iptalinde müşteri bilgilendirmesi:** İncelenen rakip sayfaları iptal nedeninin müşteriye nasıl iletildiğini anlatmıyor (K-1, K-8). FR-13 ve FR-18 bu nedenle rakip pratiğine değil, paydaş yanıtlarına (Yanıt 3, Yanıt 7, Yanıt 11) ve mağaza kurallarına dayanır.
- **İptal yetkisinin salonda olması:** Kuaförüm Yanımda'da müşterinin iptal yetkisini salon yönetiyor (K-8). Booksy'de ücretsiz iptal için gereken süreyi işletme belirliyor (K-4). FR-12'deki salonca ayarlanan iptal süresi bu pratikle uyumludur.
- **Hatırlatma kanalı maliyeti:** Rakiplerde SMS ücretli veya kotalı (K-2, K-7). "Bildirim" bu dokümanda uygulama bildirimi (push) olarak yorumlanmıştır. SMS, C öncelikli bir yedek kanaldır. Push alamayan cihazlar için uygulama içi bilgi paydaşça yeterli bulunmuştur (Yanıt 12).

### 3.3 Alan standartları ve düzenlemeler
- **KVKK (6698 sayılı Kanun, 24/3/2016 kabul metni; K-14):**
  - md. 3/1-b: Anonim hâle getirme, kişisel verilerin "başka verilerle eşleştirilerek dahi hiçbir surette kimliği belirli veya belirlenebilir bir gerçek kişiyle ilişkilendirilemeyecek hâle getirilmesi"dir.
  - md. 4/2-d: Veriler, "ilgili mevzuatta öngörülen veya işlendikleri amaç için gerekli olan süre kadar" muhafaza edilir.
  - md. 5/2: Açık rıza aranmayan şartlar sayılır. Bunlar arasında sözleşmenin ifası (c), hukuki yükümlülük (ç) ve bir hakkın tesisi, kullanılması veya korunması (e) vardır. Randevu verisinin (c) şartına dayanabileceği bir BA yorumudur (V-07).
  - md. 6: Sağlık verisi özel nitelikli kişisel veridir.
  - md. 7/1: İşleme sebepleri ortadan kalkınca veriler resen veya ilgili kişinin talebiyle "silinir, yok edilir veya anonim hâle getirilir". md. 7/2: Diğer kanunlardaki hükümler saklıdır.
  - md. 9: Yurt dışına aktarım şartları. İncelenen metin 2016 kabul metnidir, sonraki değişiklikler doğrulanmadı.
  - md. 10: aydınlatma yükümlülüğü.
  - md. 11: ilgili kişinin hakları (11/1-e: silinmesini veya yok edilmesini isteme).
  - md. 13/2: Başvurular "en geç otuz gün içinde ücretsiz olarak" sonuçlandırılır.
  - md. 16: Veri Sorumluları Sicili.
  - md. 17/2: 7. madde hükmüne aykırı olarak "kişisel verileri silmeyen veya anonim hâle getirmeyenler" TCK 138. maddeye göre cezalandırılır.
- **Kişisel Verilerin Silinmesi, Yok Edilmesi veya Anonim Hale Getirilmesi Hakkında Yönetmelik (Resmî Gazete, 28.10.2017; K-19):**
  - md. 4: İmha; silme, yok etme veya anonim hâle getirmeyi kapsar.
  - md. 5: Sicile kayıtlı veri sorumluları "kişisel veri saklama ve imha politikası hazırlamakla yükümlüdür".
  - md. 8/1: Silme, verinin "hiçbir şekilde erişilemez ve tekrar kullanılamaz hale getirilmesi"dir.
  - md. 10: Anonim hâle getirme.
  - md. 11: Periyodik imha süresi "her halde altı ayı geçemez". Politika yükümlülüğü olmayan veri sorumlularında bu süre üç aydır (md. 11/3).
  - md. 12: İşleme şartlarının tamamı ortadan kalkmışsa talep "en geç otuz gün içinde" sonuçlandırılır. Şartlar tamamen ortadan kalkmamışsa talep gerekçesi açıklanarak reddedilebilir.
  - Yönetmelik 2019'da değiştirilmiştir. İncelenen metin 2017 tarihli ilk yayımdır, güncel konsolide metin doğrulanmadı.
- **Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik md. 6 (K-13):**
  - md. 6/2: Devam eden abonelik veya üyelik ile satın alma, teslimat veya benzeri durumlara ilişkin bildirimler için "önceden onay alma zorunluluğu aranmaz". Şart: "Ancak bu tür bildirimlerde herhangi bir mal veya hizmet özendirilemez veya bunların tanıtımı yapılamaz."
  - md. 6/5: Bu iletiler için "İYS üzerinden kontrol yapılmaz".
  - Randevu hatırlatması ve iptal bildiriminin bu kapsama girdiği bir BA yorumudur, yönetmelikte "randevu" adıyla geçmiyor (V-06).
- **Erişilebilirlik (WCAG 2.2):**
  - 1.4.3: Metin kontrastı en az 4.5:1, büyük metinde 3:1 (K-15).
  - 1.4.4: Metin %200'e kadar içerik veya işlev kaybı olmadan büyütülebilmelidir (K-15).
  - 2.5.8: Dokunma hedefi en az 24×24 CSS piksel (istisnalarla), AA düzeyi (K-16).
  - WCAG web içeriği için yazılmıştır, mobil uygulamaya uyarlanması bir varsayımdır (V-10).

### 3.4 Platform ve mağaza politikaları (iş etkisi)

| Politika | İçerik | Ürüne iş etkisi |
|---|---|---|
| Apple 5.1.1(v) (K-9) | "If your app supports account creation, you must also offer account deletion within the app." | Hesap silme M öncelikli (FR-03). Kayıtta yalnızca gerekli alanlar istenir (NFR-08) |
| Apple, "Offering account deletion in your app" (K-18) | Silme, hesabı geliştiricinin kayıtlarından ve "yasal olarak tutması gerekmeyen" ilişkili verilerden çıkarır. "Only offering to temporarily deactivate or disable an account is insufficient." Yasal saklama varsa kullanıcıya neyin tutulup neyin tutulmadığı söylenir. Silmenin ne kadar süreceği bildirilir ve silme tamamlanınca onay verilir | Paydaş Seçenek A'yı seçti (Yanıt 6): kişisel veri silinir, kayıt anonim tutulur (Bölüm 3.6). Silme ekranı tutulan ve silinen verileri ve süreyi söyler. Tamamlanınca kullanıcıya onay iletilir (FR-03, US-02) |
| Apple 5.1.1(i) (K-9) | Gizlilik politikası "Explain its data retention/deletion policies" ve kullanıcının silmeyi nasıl talep edeceğini açıklar | Anonim tutulan randevu kayıtları gizlilik politikasında anlatılır (FR-02) |
| Apple 4.5.4 (K-9) | "Push Notifications must not be required for the app to function, and should not be used to send sensitive personal or confidential information." | Bildirim alınmadan da akış çalışır (FR-17). İptal açıklamasının serbest metni bildirimde gösterilmez, yalnızca onaylı kategori gösterilir (FR-18, NFR-11) |
| Apple 4.8 (K-9) | Birincil hesap için üçüncü taraf veya sosyal giriş kullanan uygulama, belirtilen özelliklere sahip eşdeğer başka bir giriş seçeneği de sunmalıdır | Sosyal giriş seçilirse ek giriş seçeneği gerekir (AS-02) |
| Apple 4.2.6 ve 4.3(a) (K-9) | 4.2.6: "Apps created from a commercialized template or app generation service will be rejected unless they are submitted directly by the provider of the app's content." 4.3(a): Aynı uygulamanın birden çok Bundle ID ile gönderilmemesi | Uygulama salonun hesabıyla yayımlanacak (Yanıt 8). İçerik sağlayıcısı salon olduğu için 4.2.6 riski düşer (NFR-15). Aynı uygulamanın ileride başka salonlara çoğaltılması 4.2.6 ve 4.3 açısından ayrıca değerlendirilmeli (AS-16) |
| Apple Developer Program kurumsal kayıt (K-20) | Kurum "legal entity" olmalı. "We don't accept DBAs, fictitious businesses, trade names, or branches." D-U-N-S numarası gerekir. Kaydı yapan kişi kurumu bağlayacak yetkiye sahip olmalı. Kuruma ait alan adında çalışan bir web sitesi ve iş e-postası istenir. Yıllık ücret 99 USD. "If you are a sole proprietor/single-person business, you must join as an individual." Bireysel hesapta satıcı olarak kişinin yasal adı, kurumsal hesapta tüzel kişinin adı görünür | Salon tüzel kişiyse kurumsal hesap açar; bunun için D-U-N-S, web sitesi ve iş e-postası hazırlanır. Şahıs işletmesiyse App Store'da satıcı olarak salon sahibinin kişisel adı görünür (AS-16, NFR-15) |
| Google Play kurumsal geliştirici hesabı (K-21) | "You can't create an organization developer account without a valid D-U-N-S number." D&B'den alınması "can take up to 30 days." Ödeme profilindeki yasal ad ve adres D&B kaydıyla birebir eşleşmeli. Google Play'de yasal ad, D-U-N-S, yasal adres, geliştirici e-postası ve telefonu gösterilir. Geliştirici e-postası genel veya kişisel olamaz | Kurumsal hesap için yayın takvimine 30 güne kadar D-U-N-S süresi eklenmeli. Salonun adres ve iletişim bilgileri mağazada herkese açık görünür (AS-16, NFR-15) |
| Google Play yeni bireysel hesaplarda test şartı (K-22) | 13 Kasım 2023'ten sonra açılan bireysel hesaplar, üretime geçmeden önce "a minimum of 12 testers who have been opted in continuously for at least 14 days" ile kapalı test yapmalı. Sayfa kurumsal hesaplardan söz etmiyor | Salon bireysel hesap açarsa yayından önce 12 test kullanıcısı ve en az 14 günlük kapalı test planlanmalı (AS-16, NFR-15) |
| Google Play hesap silme (K-10) | Uygulama içi silme yolu ve "a web link resource where users can request app account deletion and associated data deletion" zorunlu. "You must also delete the user data associated with that app account." Güvenlik, dolandırıcılığı önleme veya mevzuat için tutulan veride "you must clearly inform users about your data retention practices" | Web üzerinden silme talebi kanalı (FR-03). Anonimleştirilen kayıtların neden tutulduğu gizlilik politikasında açıklanır (FR-02) |
| Google Play Data safety (K-11) | Tüm geliştiriciler formu doldurur, veri toplamasa bile. Üçüncü taraf kodun topladığı veriler de beyan edilir. Gizlilik politikası zorunlu. Form, kullanıcıya silme talep etme yolu sunulup sunulmadığını sorar. Veriyi 90 gün içinde otomatik silen veya anonimleştiren uygulama da silme beyanına uygun olabilir | Veri envanteri ile silme ve anonimleştirme davranışı yayın öncesinde hazırlanır (NFR-08, NFR-14) |
| Android 13+ bildirim izni (K-12) | `POST_NOTIFICATIONS` çalışma zamanı izni. Yeni kurulumda "notifications are off by default". İzin işlevle ilgili bağlamda istenmeli (sayfadaki örneklerden biri: "The user submits an order for food delivery") | İzin ilk randevu oluşturulurken gerekçesiyle istenir (FR-16). İzin vermeyen kullanıcı randevu bilgisini ve iptal nedenini uygulama içinde görür (FR-14, FR-17) |

iOS bildirim izni için Apple'ın ilgili dokümantasyonu bu çalışmada açılmadı (V-09).

### 3.5 Yöntemler (iş açısından artı/eksi, seçim TA'ya bırakılır)

| Konu | Seçenek | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Müsait saat başlangıç aralığı** (Yanıt 9 ile karara bağlandı: salon ayarlar) | Kısa aralık (ör. 5–10 dk) | Takvimdeki boşluklar daha iyi doldurulur | Müşteriye uzun ve düzensiz bir saat listesi gösterilir |
| | Uzun aralık (ör. 30–60 dk) | Liste kısa ve kolay anlaşılır | Hizmet süresiyle uyumsuz kısa boşluklar kalabilir |
| **Saat hesaplama** | Hizmet süresi ve tampon süreye göre uygunluk kontrolü (Google Takvim'deki tampon kavramına benzer, K-6) | Molaya veya kapanışa taşan randevu önlenir | Yönetici ayarları ve test yükü artar |
| **Çakışma önleme** | Kayıt anında müsaitliğin yeniden kontrolü | Çift rezervasyon önlenir | Yarışı kaybeden müşteri "saat doldu" mesajı görür |
| **Randevu onayı** | Otomatik onay (Kuaförüm Yanımda'da var, K-8) | Müşteri anında kesin yanıt alır | Salon istenmeyen randevuyu engelleyemez |
| | Manuel onay | Salonun kontrolü artar | Müşteri bekler |
| **Hatırlatma kanalı** | Push | Mesaj başına maliyet yok (BA varsayımı) | İzin yoksa veya cihaz desteklemiyorsa ulaşmaz. Android 13+ yeni kurulumda kapalı (K-12) |
| | SMS | İzinden bağımsız ulaşır | Mesaj başına maliyet (ör. Fresha'da kota sonrası 0,02 $, K-2). Telefon numarası gerekir |
| | Yalnızca uygulama içi bilgi (push alamayan cihazlar için; Yanıt 12 ile karara bağlandı) | Ek maliyet ve ek kanal yok | Kullanıcı uygulamayı açmadıkça hatırlatma veya iptal bilgisini görmez |
| **Geç alınan randevuda hatırlatma** (Yanıt 5 ile karara bağlandı) | Hemen gönder | Müşteri yakın randevu için de hatırlatma alır | Oluşturma bildirimiyle neredeyse aynı anda ikinci bir bildirim gider (AS-15) |
| **İptal nedeninin iletilmesi** (Yanıt 3, Yanıt 7 ve Yanıt 11 ile karara bağlandı) | Bildirimde onaylı kategori, en fazla 200 karakterlik serbest açıklama yalnızca uygulama içinde | Müşteri nedeni bildirimde görür, hassas bilgi riski düşer | Ayrıntı için uygulamayı açmak gerekir |
| | Bildirimde serbest metin | Müşteri nedeni anında tam görür | Kilit ekranında kişisel veya hassas bilgi görünebilir. Apple 4.5.4 hassas bilginin push ile gönderilmemesini söylüyor (K-9). Bu seçenek uygulanmaz (AS-14) |
| **İptal kuralı** | Randevudan X saat öncesine kadar serbest iptal | Son dakika iptali riskini azaltır (Booksy'de ücretle birlikte kullanılıyor, K-4) | Ücret olmadan pencere içinde iptali engellemek, müşteriyi haber vermeden gelmemeye itebilir |

**Hatırlatmanın etkisi:** Health Services Research'te 2012'de yayımlanan meta-analiz 18 çalışmayı inceledi: 8 randomize kontrollü çalışma (RKÇ) ve 10 kontrollü gözlemsel çalışma. Çalışmaların ortamı çoğunlukla hastane poliklinikleri (10) ve birinci basamak klinikleri (6). Yalnızca 8 RKÇ birleştirildiğinde, SMS hatırlatma randevuya katılım olasılığını artırıyor (olasılık oranı OR 1,48). %95 güven aralığı özette 1,23–1,72, sonuçlar bölümünde 1,33–1,72 olarak geçiyor (K-17). Bu bulgu sağlık kliniklerine aittir. Salon müşterilerine ve push kanalına aktarılabilirliği bir varsayımdır (V-05).

### 3.6 Hesap silmede paydaş kararı ve mevzuat ile mağaza yükümlülükleri (Yanıt 2 ve Yanıt 6)

**Paydaş yanıtları:**
- Önceki tur: "hesap silindi şeklinde tutulabilir. hard delete olmamalı" (Yanıt 2).
- Bu tur: "A seçeneği olsun" (Yanıt 6).

**İlgili yükümlülükler:**
- **KVKK:** Veriler amaç için gerekli süre kadar tutulur (md. 4/2-d). Sebep ortadan kalkınca silinir, yok edilir veya anonim hâle getirilir (md. 7/1). İlgili kişi silme isteyebilir (md. 11/1-e). Silmeyen veya anonim hâle getirmeyenler için ceza hükmü vardır (md. 17/2). Talep üzerine imha en geç 30 gün içinde yapılır (Yönetmelik md. 12) (K-14, K-19).
- **Apple:** Yalnızca geçici devre dışı bırakma "insufficient" sayılır. Hesap kaydı, yasal olarak tutulması gerekmeyen ilişkili verilerle birlikte silinir (K-18).
- **Google Play:** Hesapla ilişkili kullanıcı verisi silinir. Saklanan veri kullanıcıya açıkça bildirilir (K-10).

**Karar:** Seçenek A uygulanır. Hesap, profil ve cihaz kayıtları silinir. Gelecekteki randevular iptal edilir. Geçmiş randevu kayıtları tarih, saat, hizmet, çalışan ve durumla kalır. Müşteri alanında "Hesap silindi" yazar. Ad, e-posta, telefon, kullanıcı kimliği ve müşteri bazlı gelmeme sayısı kayıttan çıkarılır (FR-03, FR-29, NFR-14).

**Seçeneklerin özeti (karar kaydı için):**

| Seçenek | Tanım | Durum |
|---|---|---|
| **A. Kaydı tut, kişisel veriyi anonimleştir** | Yukarıdaki karar | **Paydaş tarafından seçildi (Yanıt 6)** |
| **B. Süreli saklama, sonra anonimleştirme** | Yasal bir saklama yükümlülüğü varsa ilgili kişisel veri o süre boyunca erişimi kısıtlı tutulur, sonra anonimleştirilir | Yalnızca hukuk danışmanı bir yükümlülük tespit ederse A'ya ek olarak uygulanır (AS-13 c). Görev kapsamında (ödeme ve fatura yok) böyle bir yükümlülük tespit edilmedi |
| **C. Yalnızca "silindi" işareti (soft delete), kişisel veri kalır** | Hesap kapatılır, veri olduğu gibi kalır | Uygulanmaz. Apple bunu yetersiz sayıyor (K-18), Google ilişkili verinin silinmesini istiyor (K-10), KVKK md. 7 ve 17/2 riski var |

**Kalan risk (BA yorumu):** Küçük bir salonda tarih, saat ve çalışan bilgisi başka bilgilerle birleşince kişiyi tanımlatabilir. KVKK tanımı ise "başka verilerle eşleştirilerek dahi" tanınamamayı istiyor (md. 3/1-b). Bu nedenle anonimliğin yeterliliği hukuken teyit edilmelidir. Yeterli bulunmazsa uygulanacak yol paydaş kararıdır (AS-13).

## 4. Fonksiyonel gereksinimler

**M önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S maddelerinin ilk sürümde olması beklenir ama ertelenebilir. C maddeleri kaynak kalırsa yapılır. Bu sürümde yapılmayacaklar (W) "Kapsam dışı maddeler" bölümündedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kullanıcı uygulamada hesap oluşturabilmeli ve giriş yapabilmelidir. Kayıtta yalnızca randevu için gerekli bilgiler istenir: ad, iletişim bilgisi (e-posta ve/veya telefon, bkz. AS-02) ve şifre ya da doğrulama kodu. Şifre sıfırlama veya yeniden doğrulama yolu bulunur. | M | Görev; Apple 5.1.1(v) |
| FR-02 | Kayıt sırasında KVKK aydınlatma metni ve gizlilik politikası gösterilmeli, bu metinlere uygulama içinden her zaman erişilebilmelidir. Aydınlatma metni şunları içerir: veri sorumlusunun kimliği, işleme amacı, aktarım alıcıları, toplama yöntemi ve hukuki sebep, md. 11 hakları. Gizlilik politikası ayrıca saklama ve silme politikasını açıklar. Buna hesap silindikten sonra randevu kayıtlarının anonim olarak tutulduğu bilgisi de dahildir. | M | KVKK md. 10; Apple 5.1.1(i); Google Play hesap silme ve Data safety; Yanıt 2, Yanıt 6 |
| FR-03 | Kullanıcı hesabını uygulama içinden silebilmelidir. Uygulama dışından (web bağlantısı) silme talebi de gönderilebilmelidir. Silmenin sonuçları: hesap ve profil bilgileri silinir, gelecekteki randevular iptal edilir, geçmiş randevu kayıtları FR-29'a göre anonimleştirilerek tutulur. Silme ekranı, silinen ve anonim tutulan verileri ve işlemin en geç ne zaman tamamlanacağını söyler. Silme tamamlandığında kullanıcıya bir onay iletilir (ör. silme talebinde verilen e-posta adresine). Yalnızca devre dışı bırakma seçeneği sunulamaz. | M | Apple 5.1.1(v) ve hesap silme rehberi; Google Play hesap silme; KVKK md. 7; Yanıt 2, Yanıt 6 |
| FR-04 | Sistem üç rolü desteklemelidir: Müşteri, Çalışan, Salon yöneticisi. Her rol yalnızca yetkili olduğu verileri görür ve değiştirir (bkz. NFR-06). | M | Görev; Booksy rol ayrımı |
| FR-05 | Salon yöneticisi **salonun tek işletme profilini** tanımlayabilmelidir: ad, adres, iletişim, haftalık çalışma saatleri ve kapalı günler (resmî tatil dahil). Uygulama yalnızca bu salona hizmet verir. Birden çok işletme veya işletme kaydı yoktur. | M | Görev; Kuaförüm Yanımda; Yanıt 1 |
| FR-06 | Salon yöneticisi hizmetleri tanımlayabilmelidir: ad, süre (dakika), bilgi amaçlı fiyat ve hizmeti verebilen çalışanlar. Hizmete serbest metin not alanı eklenmez. | M | Rakip incelemesi; Yanıt 4 |
| FR-07 | Yönetici çalışan ekleyip çıkarabilmeli, çalışanı uygulamaya davet edebilmelidir. | M | Görev (çalışan takvimleri) |
| FR-08 | Her çalışan için haftalık çalışma saatleri, mola aralıkları ve izin günleri tanımlanabilmelidir. Yönetici tüm çalışanlar için bu tanımı yapar. Çalışanın kendi takvimini düzenleyebilmesi açık sorudur (AS-05). | M | Görev; Kuaförüm Yanımda; Booksy |
| FR-09 | Sistem, seçilen hizmet ve çalışan için müsait başlangıç saatlerini göstermelidir. Hesaplamada salon çalışma saatleri, çalışan saatleri, mola ve izinler, mevcut randevular ve hizmet süresi dikkate alınır. Hizmet süresi molaya veya kapanış saatine taşan saat gösterilmez. Başlangıç saatleri, salonun belirlediği **başlangıç aralığına** göre üretilir. Varsayılan aralık 15 dakikadır. Ayar FR-22'dedir; FR-22 ertelenirse ilk sürümde varsayılan değer kullanılır. | M | Yöntemler 3.5; Yanıt 9 |
| FR-10 | Müşteri hizmeti, çalışanı ("fark etmez" seçeneği dahil) ve müsait saati seçerek randevu oluşturabilmelidir. Tek salon olduğu için işletme seçimi adımı yoktur. "Fark etmez" seçilirse randevu, hizmeti verebilen müsait çalışanlardan birine atanır (atama kuralı AS-04). | M | Görev; Yanıt 1 |
| FR-11 | Aynı çalışana aynı zaman aralığında birden fazla randevu kaydedilmemelidir. İki kullanıcı aynı slotu aynı anda seçerse yalnızca biri başarılı olur, diğerine güncel müsait saatler gösterilir. | M | Yöntemler 3.5; Kuaförüm Yanımda |
| FR-12 | Müşteri kendi randevusunu iptal edebilmelidir. Randevuya salonun belirlediği iptal süresinden (varsayılan 2 saat, FR-22 ile ayarlanabilir) daha az kaldıysa uygulama, iptal yerine salonla iletişime geçme bilgisini gösterir (bkz. AS-03). İptal edilen saat yeniden müsait olur. | M | Görev; Booksy iptal penceresi; Kuaförüm Yanımda (salonun iptal yetkisini yönetmesi) |
| FR-13 | Yönetici ve ilgili çalışan randevuyu iptal edebilmelidir. İptalde bir **iptal nedeni kategorisi** seçilir ve kategori seçimi **zorunludur**. Kategoriler paydaş tarafından onaylanmıştır (Yanıt 7): "Çalışan müsait değil", "Salon kapalı / beklenmeyen durum", "Müşterinin talebiyle", "Diğer". İlk sürümde liste bu dört kategoriyle sabittir. Yönetici kategori ekleyemez veya düzenleyemez; böylece listede sağlık veya kişisel durum bilgisi yer almaz. İsteğe bağlı olarak **en fazla 200 karakterlik** bir serbest metin açıklama girilebilir (Yanıt 11). Açıklama alanında sağlık veya kişisel bilgi yazılmaması gerektiğini belirten bir uyarı gösterilir (V-08). | M | Görev; Yanıt 3; Yanıt 7; Yanıt 11; Apple 4.5.4; KVKK md. 6 |
| FR-14 | Müşteri yaklaşan ve geçmiş randevularını durumlarıyla (aktif, iptal, tamamlandı, gelmedi) listeleyebilmelidir. Salon tarafından iptal edilen randevunun detayında iptal nedeni kategorisi ve varsa açıklama gösterilir. | M | Görev; Yanıt 3 |
| FR-15 | Çalışan kendi randevularını günlük ve haftalık takvim görünümünde görebilmelidir. Yönetici tüm çalışanların takvimini çalışan bazında (ör. sütunlar hâlinde) görebilmelidir. | M | Görev; Booksy |
| FR-16 | Uygulama bildirim iznini, ilk randevu oluşturma gibi işlevle ilgili bir anda ve gerekçesini açıklayarak istemelidir. İzin uygulama açılışında istenmez. | M | Android bildirim izni rehberi |
| FR-17 | Bildirim alınamasa bile tüm randevu işlemleri çalışmalıdır. Bu durum iznin verilmemesini ve cihazın push bildirimi alamamasını (ör. Google Play hizmetleri olmayan Android cihazlar) kapsar. Randevu bilgisi, hatırlatma içeriği ve iptal nedeni uygulama içinde görülebilmelidir. Bildirimleri kapalı kullanıcıya uygulama içinde bildirimleri açma yolu gösterilir. Push alamayan cihazlar için paydaş kararıyla uygulama içi bilgi yeterlidir; bu cihazlara özel yedek kanal gerekmez (Yanıt 12). | M | Apple 4.5.4; TA S-10; Yanıt 12 |
| FR-18 | Müşteriye şu olaylarda bildirim gönderilmelidir:<br>(a) Randevu oluşturuldu.<br>(b) Randevu salon tarafından iptal edildi. Bildirim iptal nedeni **kategorisini** içerir. Serbest metin açıklama bildirimde yer almaz; bildirim uygulamadaki randevu detayına yönlendirir.<br>(c) Randevu hatırlatması (varsayılan olarak randevudan 24 saat önce). Randevu, hatırlatma zamanı geçmiş olacak kadar yakın bir zamanda oluşturulur veya taşınırsa (ör. 24 saatlik hatırlatmada randevudan 3 saat önce) hatırlatma **hemen** gönderilir.<br>Bildirim içeriği tanıtım veya kampanya içermez. | M | Görev; Yanıt 3; Yanıt 5; Yanıt 7; Ticari Elektronik İletiler Yönetmeliği md. 6/2 |
| FR-19 | İlgili çalışana ve yöneticiye şu olaylarda bildirim gönderilmelidir: yeni randevu, müşteri iptali ve müşterinin randevuyu başka bir saate taşıması (FR-21). Hesap silme nedeniyle yapılan iptaller de müşteri iptali gibi bildirilir. | M | Görev; Booksy; Yanıt 2; TA S-04 (US-10 KK-2 tutarlılığı) |
| FR-20 | Yönetici veya çalışan, telefonla veya salonda alınan randevuyu müşteri adına sisteme girebilmelidir. Müşterinin uygulama hesabı olmayabilir; bu durumda ad ve telefon girilir. Bu kişilerin verilerinin aydınlatma, saklama ve silme yolu açık sorudur (AS-18). | S | Rakip incelemesi (takvimin tek kaynak olması) |
| FR-21 | Müşteri, iptal süresi dolmamış randevusunu başka bir saate taşıyabilmelidir (yeniden planlama). Eski randevu, ancak yeni saat başarıyla alındığında serbest bırakılır. | S | Booksy (yeniden planlama) |
| FR-22 | Yönetici şu randevu kurallarını ayarlayabilmelidir: en kısa önceden randevu süresi, en ileri randevu tarihi (ör. 60 gün), hizmetler arası tampon süre, iptal süresi ve **müsait saat başlangıç aralığı**. Başlangıç aralığı salonun tamamı için tek bir değerdir. Seçenekler 5, 10, 15, 20, 30 ve 60 dakikadır (BA önerisi), varsayılan değer 15 dakikadır. Aralık değişikliği yalnızca bundan sonra gösterilen müsait saatleri etkiler; mevcut randevuların saati değişmez. | S | Google Takvim randevu planları; Yanıt 9 |
| FR-23 | Yönetici veya çalışan, gelmeyen müşterinin randevusunu "gelmedi" (no-show) olarak işaretleyebilmelidir. Yönetici müşteri bazında gelmeme sayısını görebilmelidir. Hesabı silinen müşterinin gelmeme sayısı FR-29 kapsamında artık hiçbir ekranda görünmez. | S | Rakiplerde no-show odaklı özellikler; Yanıt 2 |
| FR-24 | Hatırlatma zamanı salon tarafından ayarlanabilmelidir (ör. 24 saat ve/veya 2 saat önce). Birden çok hatırlatmanın zamanı randevu oluşturulurken geçmişse yalnızca bir hatırlatma hemen gönderilir. Zamanı henüz gelmemiş hatırlatmalar zamanında gider (BA önerisi, V-14). | S | Google Takvim (çoklu hatırlatma); Yanıt 5 |
| FR-25 | Yönetici randevu onay modunu seçebilmelidir: otomatik onay (varsayılan) veya manuel onay. Manuel modda müşteri "onay bekliyor" durumunu görür, onay veya ret olduğunda bildirim alır. | C | Kuaförüm Yanımda |
| FR-26 | Dolu bir saat için müşteri bekleme listesine yazılabilmeli, saat boşalırsa bildirim alabilmelidir. | C | Booksy; Vagaro |
| FR-27 | Push bildirimine izin vermeyen müşteriye SMS veya e-posta ile yedek hatırlatma gönderilebilmelidir. Google Play hizmetleri olmayan cihazlar için bu yedek kanal gerekli değildir (Yanıt 12). | C | Fresha; Salon Randevu; maliyet AS-06; Yanıt 12 |
| FR-28 | Müşteri randevusunu cihazının takvimine ekleyebilmelidir. | C | BA önerisi (kaynaksız; kullanıcı kolaylığı) |
| FR-29 | Hesabı silinen müşterinin geçmiş randevu kayıtları silinmez. Bu kayıtlar yöneticinin ve çalışanların takvim ve randevu ekranlarında (FR-15) **"Hesap silindi"** etiketiyle görünür. Kayıtlardan müşteriye ait kişisel tanımlayıcılar çıkarılır: ad, e-posta, telefon, kullanıcı kimliği ve müşteri bazlı gelmeme sayısı (FR-23). Kayıtta tarih, saat, hizmet, çalışan ve durum kalır (Bölüm 3.6, Seçenek A; paydaş onaylı, Yanıt 6). İleride eklenecek liste, arama, dışa aktarım veya rapor işlevleri de bu kurala uyar. Anonimliğin hukuken yeterli bulunmaması veya yasal bir saklama yükümlülüğü çıkması durumunda yapılacak değişiklik AS-13'e bağlıdır. | M | Yanıt 2; Yanıt 6; KVKK md. 7/1, md. 3/1-b; Apple hesap silme rehberi; Google Play hesap silme; TA S-05 |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt |
|---|---|---|
| NFR-01 | **Platform:** Uygulama Android ve iOS telefonlarda aynı işlevleri sunar. | Tüm M gereksinimleri iki platformda kabul testinden geçer. Desteklenen en düşük işletim sistemi sürümleri açık sorudur (AS-07). |
| NFR-02 | **Çift rezervasyon olmaması** | En az 50 eşzamanlı deneme içeren testlerde aynı çalışan ve zaman aralığına kaydedilen randevu sayısı en fazla 1'dir. |
| NFR-03 | **Yanıt süresi** | 4G bağlantıda, çalışan veya tarih seçiminden sonra müsait saat listesi 95. persentilde 2 saniye içinde görüntülenir. Randevu oluşturma onayı 3 saniye içinde gösterilir. |
| NFR-04 | **Bildirim zamanlaması** | Olay bildirimleri (oluşturma, iptal, taşıma) ve hemen gönderilen hatırlatmalar (FR-18 c) olaydan sonra 1 dakika içinde gönderilir. Planlı hatırlatmalar planlanan zamandan en fazla ±5 dakika sapmayla gönderilir. Ölçüm gönderim anına göre yapılır; cihaza teslim işletim sistemine bağlıdır. |
| NFR-05 | **Kullanılabilirlik** | Kayıtlı bir müşteri, uygulamayı açtıktan sonra en fazla 5 ekran veya adımda randevu oluşturabilir. İlk kez kullanan 5 test katılımcısının en az 4'ü randevu oluşturmayı yardımsız 3 dakikada tamamlar. |
| NFR-06 | **Yetkilendirme ve gizlilik** | Müşteri yalnızca kendi randevu ve profil verisini görür. Çalışan, müşterinin yalnızca randevu için gerekli bilgilerini (ad, iletişim, hizmet) görür. Yetkisiz erişim testlerinde başarılı erişim sayısı 0'dır. |
| NFR-07 | **KVKK başvuruları ve hesap silme süresi** | İlgili kişinin bilgi, düzeltme ve silme talepleri en geç 30 gün içinde sonuçlandırılır (KVKK md. 13/2; Yönetmelik md. 12). Uygulama içi silme talebinden sonra hesap hemen kullanılamaz hâle gelir. Kişisel verilerin silinmesi ve anonimleştirilmesi (FR-29) en geç 30 gün içinde tamamlanır. Silme ekranında bu süre kullanıcıya gösterilir. Tamamlanınca kullanıcıya onay iletilir (Apple hesap silme rehberi). |
| NFR-08 | **Veri en aza indirme** | Kayıtta zorunlu alan sayısı en fazla 3'tür (ad, iletişim bilgisi, şifre veya doğrulama). Ürün serbest metin müşteri veya hizmet notu toplamaz. Tek serbest metin alanı olan iptal açıklaması en fazla 200 karakterdir (FR-13). Mağaza veri beyanları (Data safety ve App Store gizlilik bilgileri) üçüncü taraf kütüphanelerin topladığı veriler dahil yayın öncesinde hazırlanır. |
| NFR-09 | **Erişilebilirlik** | Metin kontrastı en az 4.5:1'dir. Sistem yazı boyutu en büyük ayardayken (veya %200 büyütmede) içerik kesilmez. Dokunma hedefleri WCAG 2.5.8 hedefinden (24×24 CSS piksel) küçük olmaz; mobil eşdeğerini TA belirler. Ana ekranlar ekran okuyucuyla gezilebilir. |
| NFR-10 | **Dil, saat ve tarih** | Arayüz Türkçedir. Saatler salonun yerel saat diliminde (varsayılan Türkiye saati) 24 saat biçiminde gösterilir. Tarih biçimi GG.AA.YYYY'dir. |
| NFR-11 | **Bildirim içeriği** | Bildirimler yalnızca şu bilgileri içerir: salon, hizmet, tarih, saat, çalışan ve iptal bildiriminde onaylı iptal nedeni kategorisi (FR-13). Serbest metin alanı (ör. iptal açıklaması) bildirim gövdesinde yer almaz. Tanıtım veya kampanya içeriği yoktur. Test: Tanımlı tüm bildirim türlerinin metinlerinde serbest metin veya kampanya içeriği sayısı 0'dır. |
| NFR-12 | **Erişilebilirlik süresi (uptime)** | Randevu alma hizmeti aylık en az %99,5 erişilebilir olur. Hedefin niteliği (iç hedef mi, sözleşme mi) AS-10'da teyit edilecektir. |
| NFR-13 | **Bağlantı kesintisi** | İnternet yokken uygulama son görüntülenen randevu listesini gösterir. Yeni randevu veya iptal işleminin yapılamadığını açık bir mesajla bildirir. Çevrimdışıyken işlem kuyruğa alınmaz (BA önerisi; çakışma riski nedeniyle). |
| NFR-14 | **Anonimleştirmenin doğrulanması** | Hesap silme tamamlandıktan sonra, silinen kullanıcının adı, e-postası, telefonu veya kullanıcı kimliği yönetici ve çalışan ekranlarında (takvim, randevu detayı, müşteri detayı ve gelmeme sayısı) aranır; sonuç 0 eşleşmedir. Aynı kontrol, ürünün sakladığı randevu ve kullanıcı verilerinin tamamında da yapılır. Aynı kullanıcının silme öncesindeki geçmiş randevu sayısı, salon takviminde "Hesap silindi" etiketli kayıt sayısına eşittir. |
| NFR-15 | **Yeni. Mağaza yayını, altyapı ve fatura sahipliği (Yanıt 8, Yanıt 10)** | Uygulama App Store'da ve Google Play'de **salonun** geliştirici hesabıyla yayımlanır. Altyapı projesi (Firebase) ve faturası da **salona** aittir. Ölçütler:<br>- Her iki mağaza sayfasındaki satıcı ve geliştirici adı, AS-16'da belirlenen yasal adla birebir aynıdır. Bu ad salonun veya, salon şahıs işletmesiyse, sahibinin yasal adıdır (K-20, K-21).<br>- Altyapı projesinin sahibi ve fatura hesabı salon adınadır. Geliştirme ekibinin erişimi salonun verdiği yetkiyle sağlanır ve salon bu yetkiyi geri alabilir (V-20).<br>- Yayın planı, hesap türüne göre gereken ön koşul sürelerini içerir: kurumsal hesapta D-U-N-S numarası (Google'a göre 30 güne kadar, K-21); yeni bireysel Google Play hesabında en az 12 test kullanıcısıyla 14 günlük kapalı test (K-22). |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Kayıt ve giriş** (FR-01, FR-02)
Müşteri olarak hesap oluşturmak istiyorum; böylece randevularımı uygulamadan yönetebilirim.
- **KK-1:** Kayıt ekranında aydınlatma metni ve gizlilik politikası bağlantısı görünür. Kayıt, zorunlu alanlar (en fazla 3) doldurulunca tamamlanır.
- **KK-2:** Kayıtlı kullanıcı uygulamayı yeniden açtığında oturumu sürer veya giriş yapabilir.
- **KK-3:** Şifresini unutan kullanıcı sıfırlama veya doğrulama yoluyla hesabına erişir.

**US-02: Hesap silme** (FR-03, FR-19, FR-29, NFR-07, NFR-14)
Müşteri olarak hesabımı ve kişisel verilerimi silmek istiyorum.
- **KK-1:** Profil ayarlarında "Hesabı sil" seçeneği vardır ve talep en fazla 3 adımda tamamlanır. Yalnızca "hesabı dondur" veya "devre dışı bırak" seçeneği sunulmaz.
- **KK-2:** Silme talebinden sonra hesapla giriş yapılamaz. Gelecekteki randevular iptal edilir ve salona iptal bildirimi gider (FR-19). Geçmiş randevular salon kayıtlarında "Hesap silindi" etiketiyle, ad ve iletişim bilgisi olmadan görünür (FR-29).
- **KK-3:** Uygulamaya erişimi olmayan kullanıcı, mağaza sayfasındaki web bağlantısından silme talebi gönderebilir.
- **KK-4:** Onay ekranı şunları açıkça yazar: hangi verilerin silineceği, randevu kayıtlarının anonim olarak tutulacağı ve işlemin en geç ne zaman tamamlanacağı (en fazla 30 gün). Silme tamamlandığında kullanıcıya bir onay iletilir.
- **KK-5:** Silme tamamlandıktan sonra NFR-14'teki aramalar 0 eşleşme döndürür. Müşterinin gelmeme sayısı artık görünmez.

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
- **KK-2:** Varsayılan ayarda (15 dakika) başlangıç saatleri 15 dakikalık aralıklarla listelenir (ör. 10:00, 10:15, 10:30).
- **KK-3:** Randevu oluşturulunca onay ekranı gösterilir. Bildirim izni varsa onay bildirimi 1 dakika içinde gönderilir (NFR-04).
- **KK-4:** İki müşteri aynı slotu aynı anda onaylarsa yalnızca biri başarılı olur. Diğeri "Bu saat az önce doldu" mesajı ve güncel saat listesi görür (NFR-02).
- **KK-5:** "Fark etmez" seçeneğinde randevu, hizmeti verebilen ve o saatte müsait bir çalışana atanır. Atanan çalışanın adı onayda gösterilir.

**US-06: Randevu iptali (müşteri)** (FR-12, FR-14, FR-19)
Müşteri olarak gidemeyeceğim randevuyu iptal etmek istiyorum.
- **KK-1:** İptal süresi dolmamış randevuda "İptal et" seçeneği çalışır. Randevu durumu "iptal" olur ve saat diğer müşterilere müsait görünür.
- **KK-2:** İlgili çalışana ve yöneticiye iptal bildirimi 1 dakika içinde gönderilir.
- **KK-3:** İptal süresi dolmuşsa uygulama, iptal butonu yerine salonun iletişim bilgisini ve iptal kuralını gösterir.

**US-07: Salon tarafından iptal** (FR-13, FR-14, FR-17, FR-18, NFR-11)
Yönetici olarak, bir çalışan gelemediğinde randevuları iptal edip müşterileri nedeniyle birlikte bilgilendirmek istiyorum.
- **KK-1:** Yönetici veya ilgili çalışan, listeden bir iptal nedeni kategorisi seçmeden iptali tamamlayamaz. Kısa açıklama girmek isteğe bağlıdır.
- **KK-2:** Bildirim izni olan müşteriye 1 dakika içinde iptal bildirimi gönderilir. Bildirimde salon, hizmet, tarih, saat ve iptal nedeni kategorisi yer alır (ör. "İptal nedeni: Çalışan müsait değil"). Serbest metin açıklama bildirimde yer almaz.
- **KK-3:** Müşterinin randevu listesinde randevu "salon tarafından iptal edildi" olarak görünür. Detayda kategori ve varsa açıklama gösterilir. Bildirim alamayan müşteri de bu bilgiyi uygulama içinde görür. Buna izin vermeyen ve cihazı push desteklemeyen müşteriler dahildir.
- **KK-4:** Kategori listesi tam olarak onaylı dört kategoriden oluşur: "Çalışan müsait değil", "Salon kapalı / beklenmeyen durum", "Müşterinin talebiyle", "Diğer". Yönetici ekranında kategori ekleme veya düzenleme seçeneği yoktur.
- **KK-5:** Açıklama alanı en fazla 200 karakter kabul eder. Kalan karakter sayısı alanda gösterilir. 200 karakterlik açıklama müşterinin randevu detayında kesilmeden görünür. 201. karakter girilemez veya kayıt engellenir. Alanda sağlık veya kişisel bilgi yazılmaması gerektiğini belirten uyarı görünür.

**US-08: Hatırlatma ve bildirim izni** (FR-16, FR-17, FR-18, FR-24)
Müşteri olarak randevumu unutmamak için hatırlatma almak istiyorum.
- **KK-1:** Bildirim izni uygulama ilk açıldığında değil, ilk randevu oluşturulurken gerekçesiyle istenir.
- **KK-2:** İzin veren müşteri randevudan 24 saat önce (±5 dakika) hatırlatma alır. Hatırlatma salon, hizmet, tarih, saat ve çalışan bilgisini içerir ve kampanya içermez.
- **KK-3:** İzin vermeyen müşteri randevu oluşturabilir ve randevusunu listede görür. Uygulamada bildirimleri açma yolu gösterilir.
- **KK-4:** İptal edilen randevu için hatırlatma gönderilmez.
- **KK-5:** Hatırlatma süresi 24 saat iken randevudan 3 saat önce oluşturulan randevu için hatırlatma, oluşturmadan sonra 1 dakika içinde gönderilir. Aynı kural, randevu yakın bir saate taşındığında da geçerlidir (FR-21).
- **KK-6:** Salon 24 saat ve 2 saat önce hatırlatma ayarlamışsa: Randevudan 3 saat önce oluşturulan randevuya 24 saatlik hatırlatma hemen, 2 saatlik hatırlatma zamanında gider. Randevudan 1 saat önce oluşturulan randevuya yalnızca bir hatırlatma hemen gider.
- **KK-7:** Google Play hizmetleri olmayan bir Android cihazda müşteri randevu oluşturabilir, iptal edebilir ve yaklaşan randevusunu, durumunu ve salon iptalinin nedenini uygulama içinde görür. Bu cihaza SMS veya e-posta ile yedek bildirim gönderilmesi beklenmez (Yanıt 12).

**US-09: Telefonla gelen randevuyu girme** (FR-20)
Yönetici olarak telefonla arayan müşterinin randevusunu sisteme girmek istiyorum; böylece takvim tek kaynak olur.
- **KK-1:** Yönetici, hesabı olmayan müşteri için ad ve telefonla randevu oluşturabilir. Slot diğer müşterilere dolu görünür.
- **KK-2:** Çakışan bir saate randevu girilmek istendiğinde sistem engeller (FR-11).

**US-10: Yeniden planlama** (FR-21, FR-19, FR-18)
Müşteri olarak randevumu iptal edip yeniden almak yerine başka bir saate taşımak istiyorum.
- **KK-1:** Yeni saat başarıyla alınmadan eski randevu iptal edilmez.
- **KK-2:** Taşımadan sonra ilgili çalışana ve yöneticiye 1 dakika içinde değişiklik bildirimi gider (FR-19). Taşıma bir çalışandan başka bir çalışana yapılmışsa iki çalışan da bildirim alır.
- **KK-3:** Hatırlatmalar yeni saate göre planlanır. Hatırlatma zamanı geçmişse US-08 KK-5 uygulanır.

**US-11: Randevu kuralları ve no-show** (FR-22, FR-23, FR-09)
Yönetici olarak son dakika randevularını, saat listesinin düzenini ve gelmeyen müşterileri yönetmek istiyorum.
- **KK-1:** En kısa önceden randevu süresi 2 saat olarak ayarlandığında, şu andan itibaren 2 saatten yakın slotlar müşteriye gösterilmez.
- **KK-2:** "Gelmedi" olarak işaretlenen randevu müşterinin gelmeme sayısına eklenir. Yönetici bu sayıyı müşteri detayında görür.
- **KK-3:** Yönetici başlangıç aralığını yalnızca tanımlı seçeneklerden (5, 10, 15, 20, 30, 60 dakika) seçebilir. Aralık 30 dakika yapıldığında müşteriye yalnızca saat başı ve buçuk başlangıçlar gösterilir (ör. 10:00, 10:30). Ayar değişikliğinden önce alınmış 10:15 randevusu aynı saatte kalır ve takvimde görünmeye devam eder.

**US-12: Hesabı silinen müşterinin geçmiş kayıtları** (FR-29, NFR-14)
Salon yöneticisi olarak, müşteri hesabını silse de geçmiş takvimimin eksiksiz kalmasını istiyorum; böylece geçmiş günlerin doluluğunu ve çalışanların yaptığı işleri görebilirim.
- **KK-1:** Hesabı silinen müşterinin geçmiş randevuları takvimde tarih, saat, hizmet, çalışan ve durumla görünmeye devam eder. Müşteri alanında "Hesap silindi" yazar.
- **KK-2:** Bu kayıtlarda ad, telefon, e-posta veya müşteri detayına giden bağlantı yoktur. Yönetici ve çalışan ekranlarında silinen müşterinin adıyla veya telefonuyla yapılan aramada sonuç çıkmaz.
- **KK-3:** Silme öncesinde ve sonrasında, ilgili tarih aralığındaki toplam randevu sayısı aynıdır.

## 7. Kapsam dışı maddeler

Bu sürümde yapılmayacak maddeler (MoSCoW: W):
- **Çok işletmeli platform:** birden çok salonun aynı uygulamayı kullanması, işletme kaydı ve onayı, işletme seçimi veya katılım kodu (Yanıt 1).
- **Aynı uygulamanın başka salonlar için çoğaltılması.** Planlanırsa Apple 4.2.6 ve 4.3 açısından ayrıca değerlendirilir (AS-16).
- **Çok şubeli salon yönetimi** ve şubeler arası raporlama.
- **Serbest metin müşteri ve hizmet notu alanı** (Yanıt 4). Salon Randevu gibi rakiplerde müşteri notu var (K-7). Bu ürünün notu olmaması sağlık verisi toplanması riskini de azaltır.
- **Hesap silmede yalnızca "silindi" işaretiyle kişisel verinin tutulması** (Bölüm 3.6, Seçenek C). Mağaza politikalarıyla çeliştiği için uygulanmaz.
- **Bildirimde serbest metin iptal açıklaması.** Açıklama yalnızca uygulama içinde gösterilir (FR-18). Paydaşın bu karara açık onayı AS-14'te beklenmektedir.
- **200 karakteri aşan iptal açıklaması** (Yanıt 11).
- **Yöneticinin iptal nedeni kategorilerini eklemesi veya düzenlemesi.** İlk sürümde onaylı dört kategori sabittir (FR-13).
- **Google Play hizmetleri olmayan cihazlar için SMS, e-posta veya başka bir yedek bildirim kanalı.** Paydaş kararına göre bu cihazlarda uygulama içi bilgi yeterlidir (Yanıt 12).
- **Salonun mevcut bir randevunun saatini veya çalışanını değiştirmesi.** İlk sürümde salon randevuyu iptal eder (FR-13), müşteri yeni randevu alır veya yönetici yeni randevuyu girer (FR-20). Paydaş bu işlevi isterse AS-20'ye bakınız.
- **Hizmete veya çalışana göre ayrı başlangıç aralığı.** Başlangıç aralığı salonun tamamı için tek değerdir (FR-22).
- **Çevrimiçi ödeme, kapora, ön ödeme ve iptal ücreti** (Fresha, Booksy ve Vagaro'da var). Görevde istenmedi. Ödeme altyapısı ve ek mevzuat getirir.
- **Pazar yeri ve keşif** (Fresha pazar yeri modeli).
- **Pazarlama ve kampanya bildirimleri, toplu SMS.** Ticari Elektronik İletiler Yönetmeliği md. 6/2'deki onay istisnası tanıtım içeren iletileri kapsamaz (K-13). Bu tür iletiler İYS ve onay süreçleri gerektirir.
- **Muhasebe, adisyon, stok, personel primi** (Salon Randevu'da var).
- **Müşteri değerlendirme ve puanlama, sadakat ve puan sistemi.**
- **Web üzerinden randevu sayfası ve sosyal medya entegrasyonları** (Vagaro'da var). Hesap silme için gereken web bağlantısı (FR-03) bunun istisnasıdır.
- **Tablet ve masaüstü yönetim paneli.**
- **Gelişmiş raporlama, analitik, arama ve dışa aktarım işlevleri.** İleride eklenirse FR-29 ve NFR-14'teki anonimlik kuralına uyarlar.

## 8. Varsayımlar

- **V-01:** Uygulama tek bir salona aittir (Yanıt 1). Müşteri uygulamayı mağazadan indirir ve doğrudan bu salonun hizmetlerini görür. İşletme bağlantısı, kodu veya seçimi yoktur. İlk salon yöneticisi hesabının kurulum sırasında oluşturulacağı varsayılmıştır. Bu bir müşteri akışı değildir.
- **V-02:** Flutter ve Firebase ekibin verdiği kısıtlardır. Firebase'in hangi servislerinin kullanılacağı TA'nın kararıdır.
- **V-03:** "Bildirim" uygulama içi ve push bildirimi olarak yorumlanmıştır. SMS ve e-posta, yalnızca bildirime izin vermeyen kullanıcılar için C öncelikli yedek kanaldır (FR-27).
- **V-04:** Varsayılan değerler (24 saatlik hatırlatma, 2 saatlik iptal süresi, 60 günlük ileri tarih, 15 dakikalık başlangıç aralığı) BA önerisidir. Bu değerler rakip pratiklerinden esinlenmiştir, kesin değerler doğrulanmadı. İptal açıklamasının 200 karakter sınırı ise paydaş kararıdır (Yanıt 11).
- **V-05:** Hatırlatmanın no-show'u azaltacağı varsayımı, sağlık kliniklerindeki SMS meta-analizine dayanır (8 RKÇ, OR 1,48). Salon sektörüne ve push kanalına aktarılabilirliği doğrulanmadı.
- **V-06:** Randevu hatırlatmaları ve iptal bildirimlerinin Ticari Elektronik İletiler Yönetmeliği md. 6/2 kapsamında önceden onay gerektirmediği bir BA yorumudur. Hukuki teyit gerekir.
- **V-07:** Randevu verisinin KVKK md. 5/2-c (sözleşmenin ifası) şartına dayanarak açık rıza olmadan işlenebileceği bir BA yorumudur. Hukuki teyit gerekir.
- **V-08:** Paydaş hizmet notu alanını istemedi (Yanıt 4). Bu nedenle üründe serbest metin müşteri veya hizmet notu yoktur ve sağlık verisi (KVKK md. 6) toplanması öngörülmez. Üründe kalan tek serbest metin alanı, salonun isteğe bağlı ve en fazla 200 karakterlik iptal açıklamasıdır (FR-13). Bu alana sağlık bilgisi yazılmaması için arayüzde uyarı gösterilmesi BA önerisidir.
- **V-09:** iOS'ta bildirim izninin Android'e benzer şekilde kullanıcı onayı gerektirdiği varsayılmıştır. Apple'ın ilgili sayfası bu çalışmada açılmadığı için doğrulanmadı.
- **V-10:** WCAG 2.2 web içeriği için yazılmıştır. Mobil uygulamada eşdeğer hedef olarak kullanılması bir varsayımdır.
- **V-11:** Platformun yerleşik çözümü olarak Google Takvim randevu planları incelendi. iOS ve Android'de salon randevusuna özgü yerleşik bir çözüm için ayrıca kaynak aranmadı.
- **V-12:** Çocuklara yönelik özel düzenlemeler değerlendirilmedi. Hedef kitle yetişkin müşteriler ve salon çalışanlarıdır.
- **V-13 (Yanıt 6 ile güncellendi):** Önceki sürümde Yanıt 2 ("hard delete olmamalı"), kişisel verinin değil randevu **kayıtlarının** silinmemesi olarak yorumlanmıştı. Paydaş bu yorumu Seçenek A'yı seçerek onayladı. Bu nedenle FR-29 artık varsayıma değil, paydaş kararına dayanır. Hukuki yeterlilik AS-13'te açıktır.
- **V-14 (Yanıt 5):** "Hemen gönderilir" şöyle yorumlanmıştır: Hatırlatma, randevu oluşturulduktan veya taşındıktan sonra 1 dakika içinde (NFR-04) ve oluşturma bildiriminden ayrı olarak gönderilir. Manuel onay modunda (FR-25) "hemen", onay anı olarak kabul edilmiştir. Birden çok hatırlatmanın zamanı geçmişse yalnızca birinin gönderilmesi BA önerisidir. Amaç müşteriye art arda aynı içerikte bildirim gitmesini önlemektir.
- **V-15 (Yanıt 3, Yanıt 7 ve Yanıt 11 ile güncellendi):** "Müşteriye bir şekilde bildirilmeli" ifadesi, iptal nedeninin uygulama içinde her durumda gösterilmesi olarak uygulanmıştır. "Bildirimde olursa iyi olur" ifadesi ise bildirimde kategori gösterilmesi olarak uygulanmıştır. Kategori listesi, zorunluluk ve 200 karakter sınırı paydaşça onaylandı. Listenin ilk sürümde sabit olması (yönetici düzenleyemez) BA önerisidir. Amaç, onaylanan listenin dışına çıkılmasını ve sağlık veya kişisel durum içeren bir kategori eklenmesini önlemektir.
- **V-16 (Yanıt 1; Yanıt 8 ve Yanıt 10 ile desteklendi):** Tek salon modelinde veri sorumlusunun salon işletmesi olacağı tahmin edilmektedir. Uygulamanın salonun hesabıyla yayımlanması (Yanıt 8) ve altyapı projesinin ve faturasının salona ait olması (Yanıt 10) bu tahmini destekler. Hukuki teyit AS-09'dadır.
- **V-17 (Yanıt 9):** "Ayarlanabilir" yanıtı, başlangıç aralığının salonun tamamı için tek bir ayar olması şeklinde yorumlanmıştır. Seçenek listesi (5, 10, 15, 20, 30, 60 dakika) ve varsayılan 15 dakika BA önerisidir. Ayar değişikliğinin mevcut randevuları etkilememesi BA kararıdır; amaç müşteriye haber verilmeden randevu saatinin değişmesini önlemektir. Hizmete veya çalışana göre ayrı aralık istenmediği varsayılmıştır.
- **V-18 (Yanıt 8):** "Salon" yanıtı, uygulamanın App Store'da ve Google Play'de salonun kendi adına açılmış geliştirici hesaplarıyla yayımlanacağı şeklinde yorumlanmıştır. Geliştirme ekibinin bu hesaplara salonun izniyle ekip üyesi olarak erişeceği varsayılmıştır. Hesap açma ve doğrulama işlerini kimin yürüteceği AS-16'dadır.
- **V-19 (Yanıt 12 ile onaylandı):** TA dokümanına göre push altyapısı Google Play hizmetleri olmayan Android cihazlarda çalışmıyor. Paydaş bu cihazlardaki kullanıcıların bilgiyi yalnızca uygulama içinde görmesini yeterli buldu ("sadece uygulama içi yeterli"). Bu kullanıcılar FR-17 kapsamındadır ve FR-27'deki yedek kanal bu cihazlar için gerekmez. Bu cihazlarda hatırlatmanın kullanıcıya ancak uygulama açıldığında ulaşacağı kabul edilmiştir.
- **V-20 (yeni, Yanıt 10):** "Salon" yanıtı şöyle yorumlanmıştır: Firebase projesi salonun hesabında açılır ve kullandıkça öde planının faturası salona kesilir. Geliştirme ekibi projeye salonun verdiği yetkiyle erişir. Aylık bütçe ve maliyet uyarı eşiği bu turda belirlenmedi (AS-19).

## 9. Açık sorular

Bu turda tamamen cevaplanan sorular listeden çıkarıldı: AS-17 (başlangıç aralığı, Yanıt 9). Kısmen cevaplanan AS-13, AS-14, AS-16 ve AS-19 kalan kısımlarıyla güncellendi. AS-19'un ilk kısmı (proje ve fatura sahibi) Yanıt 10 ile cevaplandı, bütçe kısmı açık kaldı. Önceki sürümdeki diğer ID'ler korunmuştur. Daha önce cevaplanan AS-01 listede yer almaz.

- **AS-02:** Müşteri kaydında birincil kimlik e-posta mı, telefon numarası mı olacak? Telefon doğrulaması SMS maliyeti getirir. Sosyal giriş (Google, Apple) isteniyor mu? İstenirse Apple 4.8 eşdeğer bir giriş seçeneği gerektirir. (TA S-02: TA geçici olarak e-posta ve şifreyle ilerliyor.)
- **AS-03:** İptal süresi dolduktan sonra müşteri iptali tamamen engellenmeli mi, yoksa iptal edilip salona "geç iptal" olarak mı işaretlenmeli? (TA S-13: TA geçici olarak engelleme ile ilerliyor.)
- **AS-04:** "Fark etmez" seçildiğinde çalışan nasıl atanmalı: en az randevulu çalışan mı, sırayla mı, ilk müsait çalışan mı? (TA S-06: TA'nın geçici kuralı "o gün en az randevusu olan çalışan".)
- **AS-05:** Çalışan kendi çalışma saatlerini ve izinlerini kendisi düzenleyebilmeli mi, yoksa yalnızca yönetici mi? Booksy'de çalışan kendi randevularını görüp yönetebiliyor (K-3). (TA S-14.)
- **AS-06:** SMS yedek hatırlatma (FR-27, yalnızca bildirime izin vermeyen kullanıcılar için) ve takvime ekleme (FR-28) ilk sürüm planına alınacak mı? SMS isteniyorsa maliyeti kim üstlenecek, aylık bütçe nedir?
- **AS-07:** Desteklenen en düşük Android ve iOS sürümleri nelerdir? TA'nın teknik tabanı Android 7.0 (API 24) ve iOS 15'tir. TA, daha düşük sürümlerin seçilen altyapıda desteklenmediğini bildiriyor (TA S-03). Bu taban iş açısından kabul edilebilir mi?
- **AS-08 (acil):** Kişisel veriler Türkiye dışında barındırılacaksa KVKK md. 9 kapsamında hangi hukuki dayanak kullanılacak? İncelenen kanun metni 2016 kabul metnidir. Güncel md. 9 hükmü doğrulanmadı ve hukuk danışmanıyla teyit edilmelidir. TA'ya göre veri konumu proje kurulduktan sonra değiştirilemiyor ve seçeneklerde Türkiye bölgesi yok. Proje salonun hesabında açılacağı için (Yanıt 10) bu soru salon proje kurulmadan önce yanıtlanmalıdır (TA S-01).
- **AS-09:** Tek salon modelinde veri sorumlusu salon işletmesi mi olacak (V-16)? Uygulamayı geliştiren veya işleten taraf veri işleyen mi olacak? Salonun Veri Sorumluları Siciline kayıt yükümlülüğü (KVKK md. 16) ve buna bağlı saklama ve imha politikası yükümlülüğü (Yönetmelik md. 5) var mı?
- **AS-10:** Erişilebilirlik süresi hedefi (NFR-12, %99,5) ve yanıt süresi hedefleri (NFR-03) salonun beklentisine uygun mu? %99,5 bir sözleşme yükümlülüğü mü, iç hedef mi? Ölçümden kim sorumlu? (TA S-11: TA altyapının hizmet düzeyi taahhüdünü doğrulayamadı.)
- **AS-11:** Bir hizmet birden çok çalışan veya kaynak gerektiriyor mu (ör. belirli bir koltuk veya oda)? İlk sürümde tek çalışan ve tek hizmet varsayılmıştır.
- **AS-12:** Müşteri bir randevuda birden çok hizmet seçebilmeli mi (ör. kesim + fön)?
- **AS-13 (Yanıt 6 ile daraltıldı):** Seçenek A onaylandı. Kalan sorular:
  - (b) Hukuk danışmanı, tutulan kaydın (tarih, saat, hizmet, çalışan) küçük bir salonda KVKK md. 3/1-b anlamında anonim sayılıp sayılmadığını teyit etmeli.
  - (c) Salonun bu kayıtlar için yasal bir saklama yükümlülüğü var mı? Varsa süre ve kapsam nedir (Seçenek B)? Bu çalışmada böyle bir yükümlülük tespit edilmedi.
  - (d) Hukuk danışmanı kaydı yeterince anonim bulmazsa hangi yol izlenecek? Örnek seçenekler (TA S-12):
    - Saat bilgisi gün düzeyine indirilir. Günlük doluluk korunur, saatlik geçmiş kaybolur.
    - Çalışan bilgisi çıkarılır. Çalışanın yaptığı işlerin geçmişi kaybolur.
    - Kayıt belirli bir süre sonra tamamen silinir.
  - Bu seçeneklerin her biri US-12 KK-1'i değiştirir.
- **AS-14 (Yanıt 7 ve Yanıt 11 ile daraltıldı):** Kategoriler, zorunluluk ve 200 karakter sınırı onaylandı. Kalan soru:
  - (c) Serbest metin açıklamanın bildirimde gösterilmemesi kararı (FR-18 b) paydaş tarafından açıkça onaylanıyor mu? Bu kararın gerekçesi, Apple 4.5.4'teki hassas bilgi uyarısıdır.
- **AS-15 (Yanıt 5 ve TA S-08):**
  - (a) Hemen gönderilen hatırlatma, oluşturma bildirimiyle tek bildirimde birleştirilsin mi, yoksa ayrı mı gitsin? Bu doküman ayrı gönderimi varsaymıştır (V-14).
  - (b) Salon hatırlatma zamanını değiştirdiğinde mevcut randevuların hatırlatmaları da yeni ayara göre güncellensin mi?
- **AS-16 (Yanıt 8 ile daraltıldı):** Uygulama salonun geliştirici hesabıyla yayımlanacak. Kalan sorular:
  - (a) Salonun hukuki yapısı nedir: tüzel kişi (ör. limited veya anonim şirket) mi, şahıs işletmesi mi?
    - Apple şahıs işletmesinin bireysel hesap açmasını istiyor. Bu durumda App Store'da satıcı olarak salon sahibinin kişisel yasal adı görünür (K-20). Bu kabul edilebilir mi?
    - Tüzel kişi için Apple'da ve Google Play'de D-U-N-S numarası gerekir (K-20, K-21). Apple ayrıca kuruma ait alan adında çalışan bir web sitesi ister (K-20).
  - (b) Hesapların açılması, D-U-N-S başvurusu ve doğrulama işlerini kim yürütecek? Yayın takvimine şu süreler eklenmeli:
    - D-U-N-S numarasının alınması 30 güne kadar sürebilir (K-21).
    - Bireysel bir Google Play hesabı yeni açılırsa en az 12 test kullanıcısıyla 14 günlük kapalı test gerekir (K-22).
  - (c) Aynı uygulamanın ileride başka salonlar için çoğaltılması planlanıyor mu? Planlanıyorsa Apple 4.2.6 ve 4.3 ret riski ayrıca değerlendirilmelidir.
- **AS-18 (TA S-09):** Hesapsız müşteri adına girilen randevularda (FR-20) bu müşterinin ad ve telefonu için aydınlatma nasıl yapılacak? Saklama süresi nedir? Bu kişilerin silme talepleri hangi kanaldan alınacak? Yönetici ekranında "bu müşterinin kişisel verisini anonimleştir" gibi bir işlev isteniyor mu?
- **AS-19 (Yanıt 10 ile daraltıldı):** Firebase projesinin ve faturasının sahibi salon olacak. Kalan sorular:
  - (b) Altyapı için aylık bütçe nedir? Maliyet belirli bir tutarı aşınca salona uyarı gitmesi isteniyor mu, isteniyorsa eşik nedir?
  - (c) Salon ile geliştirme ekibi arasındaki veri sorumlusu ve veri işleyen ilişkisi (AS-09) bu sahiplik modeline göre yazılı olarak tanımlanacak mı?
- **AS-20 (TA S-04):** Salonun mevcut bir randevunun saatini veya çalışanını değiştirebilmesi isteniyor mu? Bu sürümde bu işlev kapsam dışıdır; salon iptal eder, müşteri yeniden randevu alır. İstenirse müşteriye "randevunuz değiştirildi" bildirimi ve müşterinin değişikliği kabul veya ret yolu tanımlanmalıdır.

## 10. Kaynaklar

Tüm kaynaklar bu çalışmada açılıp okunmuştur. Önceki sürümün K-1–K-19 numaraları korunmuştur.

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K-1 | Fresha for business | https://www.fresha.com/for-business | "get your clients self-booking online 24/7"; "we send automated booking reminders via email and text"; "implement a flexible cancellation policy"; randevu anında kapora veya tam ön ödeme. İşletme iptalinde müşteri bilgilendirmesi sayfada yok |
| K-2 | Fresha Pricing | https://www.fresha.com/pricing | Bağımsız plan 19,95 $/ay; ekip üyesi başına 14,95 $/ay; ayda 100 ücretsiz SMS/WhatsApp (ekipte üye başına); SMS başına 0,02 $, WhatsApp 0,04–0,11 $; yeni müşteride %20 tek seferlik komisyon, en az 6 $ |
| K-3 | Booksy Features | https://biz.booksy.com/en-us/features | "Clients can view your availability and conveniently book appointments 24/7"; çalışan profili, çalışan kendi randevularını "view and manage their own appointments"; "Access staff calendars"; "Unlimited automatic client appointment reminders are included"; bekleme listesi; 29,99 $/ay + vergi, ek kullanıcı başına 20 $/ay |
| K-4 | Booksy No-Show Protection | https://biz.booksy.com/en-us/features/no-show-protection | Ücretsiz iptal veya yeniden planlama için "how much advance notice" gerektiğini işletme belirler; "charge a fee if a client cancels too late or ghosts you"; "Charge a deposit at the time of booking"; güvenilir müşteri muafiyeti |
| K-5 | Vagaro Salon Software | https://www.vagaro.com/pro/salon-software | "on your website, Instagram, Facebook, Google, or Apple Maps"; "Send automatic reminders"; "cancellation policies with no-show fees or required deposits"; "Smart waitlists automatically notify the right clients"; "base of $23.99 monthly cost" (tek şube) |
| K-6 | Google Calendar Help: Appointment schedules | https://support.google.com/calendar/answer/10729749 | "By default, there is a 4-hour minimum lead time to schedule an appointment"; "Add time between appointments on your calendar"; günlük en fazla randevu; en fazla 5 hatırlatma, "You can't edit the text of the reminder emails"; "Some appointment schedule features require an eligible Google Workspace or Google One subscription" |
| K-7 | Salon Randevu İşletme Yönetimi (App Store, ABD) | https://apps.apple.com/us/app/id1413695973 | Paylaşılan bağlantıyla çevrimiçi randevu; SMS ile onay ve hatırlatma; müşteri notları; prim, stok, gelir-gider; mağaza açıklamasına göre aylık ücret yok; ücretsiz Silver paket (100 SMS, 50 randevu); uygulama içi satın almalar 4,99 $ ile 949,00 $ arası; 1 değerlendirme |
| K-8 | Kuaförüm Yanımda (App Store, TR) | https://apps.apple.com/tr/app/kuaf%C3%B6r%C3%BCm-yan%C4%B1mda/id1516248277?l=tr | "Herhangi bir onaylamaya gerek kalmadan"; "Toplu Randevu Onaylama" (2.1.10); tatil, personel izni ve yemek molasıyla çakışma önleme; 2.1.4 (19.10.2025): bildirim merkezi, özelleştirilebilir bildirim içerikleri, "Hesap Sil", "Müşterilerin iptal yetkisi salon tarafından yönetilebiliyor"; 4,6 puan, 75 değerlendirme; iptal nedeni bildirimi sayfada yok |
| K-9 | App Review Guidelines (Apple) | https://developer.apple.com/app-store/review/guidelines/ | 5.1.1(v): "If your app supports account creation, you must also offer account deletion within the app"; 5.1.1(i): "Explain its data retention/deletion policies"; 4.5.4: "Push Notifications must not be required for the app to function, and should not be used to send sensitive personal or confidential information"; 4.8 eşdeğer giriş seçeneği; 4.2.6: "Apps created from a commercialized template or app generation service will be rejected unless they are submitted directly by the provider of the app's content"; 4.3(a): "Don't create multiple Bundle IDs of the same app" |
| K-10 | Google Play: Understanding account deletion requirements | https://support.google.com/googleplay/android-developer/answer/13327111 | "provide users with an in-app path to delete their app accounts and associated data"; "provide a web link resource where users can request app account deletion and associated data deletion"; "you must also delete the user data associated with that app account"; saklanan veride "you must clearly inform users about your data retention practices" |
| K-11 | Google Play: Data safety section | https://support.google.com/googleplay/android-developer/answer/10787469 | Tüm geliştiriciler formu doldurur; üçüncü taraf kodun verisi beyan edilir; gizlilik politikası zorunlu; silme talebi yolu sorusu; veriyi 90 gün içinde otomatik silen veya anonimleştiren uygulama silme beyanına uygun olabilir |
| K-12 | Android Developers: Notification runtime permission | https://developer.android.com/develop/ui/views/notifications/notification-permission | Android 13 (API 33) `POST_NOTIFICATIONS`; yeni kurulumda "notifications are off by default"; "Request the permission in context"; örnek: "The user submits an order for food delivery" |
| K-13 | Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik (Lexpera) | https://www.lexpera.com.tr/mevzuat/yonetmelikler/ticari-iletisim-ve-ticari-elektronik-iletiler-hakkinda-yonetmelik | md. 6/2: abonelik, üyelik, satın alma, teslimat veya benzeri bildirimlerde "önceden onay alma zorunluluğu aranmaz"; "Ancak bu tür bildirimlerde herhangi bir mal veya hizmet özendirilemez veya bunların tanıtımı yapılamaz."; md. 6/5: "İYS üzerinden kontrol yapılmaz" |
| K-14 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (Adalet Bakanlığı, 24/3/2016 kabul metni PDF) | https://mgm.adalet.gov.tr/Resimler/SayfaDokuman/51120191031036698%20KVKK.pdf | md. 3/1-b: "başka verilerle eşleştirilerek dahi hiçbir surette kimliği belirli veya belirlenebilir bir gerçek kişiyle ilişkilendirilemeyecek hâle getirilmesi"; md. 4/2-d; md. 5/2 (c), (ç), (e); md. 6; md. 7/1: "silinir, yok edilir veya anonim hâle getirilir"; md. 7/2; md. 9; md. 10; md. 11/1-e; md. 13/2: "en geç otuz gün içinde ücretsiz olarak sonuçlandırır"; md. 16; md. 17/2: "kişisel verileri silmeyen veya anonim hâle getirmeyenler" TCK 138'e göre cezalandırılır. WebFetch PDF'i metin olarak çözemedi; doğrulama indirilen dosyanın sayfaları okunarak yapıldı |
| K-15 | WCAG 2.2 (W3C) | https://www.w3.org/TR/WCAG22/ | 1.4.3: "a contrast ratio of at least 4.5:1" (büyük metinde 3:1); 1.4.4: "text can be resized without assistive technology up to 200 percent without loss of content or functionality" |
| K-16 | Understanding SC 2.5.8 Target Size (Minimum) (W3C) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | "The size of the target for pointer inputs is at least 24 by 24 CSS pixels, except when:" (beş istisna); AA düzeyi |
| K-17 | How Effective Are Short Message Service Reminders at Increasing Clinic Attendance? A Meta-Analysis and Systematic Review (Health Services Research, 2012) | https://pmc.ncbi.nlm.nih.gov/articles/PMC3419880/ | 18 çalışma (8 RKÇ, 10 kontrollü gözlemsel); hastane poliklinikleri (10) ve birinci basamak (6) ağırlıklı; RKÇ'lerde birleştirilmiş OR 1,48; güven aralığı özette 1,23–1,72, sonuçlarda alt sınır 1,33; yazarlar: "substantially increase the likelihood of attending clinic appointments" |
| K-18 | Apple Developer: Offering account deletion in your app | https://developer.apple.com/support/offering-account-deletion-in-your-app/ | "Deleting an account removes the account from the developer's records, along with any data associated with the account that the developer isn't legally required to maintain."; "only offering to temporarily deactivate or disable an account is insufficient."; "If you're legally required to retain some user information, let people know what information will and won't be retained."; "Inform the person how long it will take to delete the account and provide a confirmation when the deletion is complete." |
| K-19 | Kişisel Verilerin Silinmesi, Yok Edilmesi veya Anonim Hale Getirilmesi Hakkında Yönetmelik (Resmî Gazete, 28.10.2017, ilk yayım metni) | https://www.resmigazete.gov.tr/eskiler/2017/10/20171028-10.htm | md. 4 imha tanımı; md. 5: "kişisel veri saklama ve imha politikası hazırlamakla yükümlüdür"; md. 8/1: "hiçbir şekilde erişilemez ve tekrar kullanılamaz hale getirilmesi"; md. 10 anonimleştirme; md. 11: "Bu süre her halde altı ayı geçemez", politika yükümlülüğü olmayanlarda üç ay (11/3); md. 12: "talebini en geç otuz gün içinde sonuçlandırır", şartlar tamamen ortadan kalkmamışsa "gerekçesi açıklanarak reddedilebilir". 2019 değişiklikleri sonrası konsolide metin doğrulanmadı |
| K-20 | Apple Developer: Program enrollment (Account Help) | https://developer.apple.com/help/account/membership/program-enrollment/ | "your organization must be a legal entity... We don't accept DBAs, fictitious businesses, trade names, or branches."; kurumsal kayıtta D-U-N-S gerekir, çoğu ülkede ücretsiz; kurumu bağlama yetkisi; kuruma ait alan adıyla web sitesi ve iş e-postası; yıllık ücret 99 USD; "If you are a sole proprietor/single-person business, you must join as an individual."; bireyselde "your personal legal name will be listed as the seller", kurumsalda "the legal entity name will be listed as the seller" |
| K-21 | Google Play Console Help: Organization developer account requirements | https://support.google.com/googleplay/android-developer/answer/13628312?hl=en | "You can't create an organization developer account without a valid D-U-N-S number."; D&B başvurusu ücretsiz ve "can take up to 30 days"; ödeme profilindeki yasal ad ve adres D&B kaydıyla birebir eşleşmeli; Google Play'de yasal ad, D-U-N-S, yasal adres, geliştirici e-postası ve telefonu gösterilir; geliştirici e-postası genel veya kişisel olamaz |
| K-22 | Google Play Console Help: App testing requirements for new personal developer accounts | https://support.google.com/googleplay/android-developer/answer/14151465?hl=en | "Developers with personal accounts created after November 13, 2023, must run a closed test for their app"; en az 12 test kullanıcısı, "continuously for at least 14 days"; sayfa kurumsal hesaplardan söz etmiyor |

**Açılamayan veya kullanılmayan kaynaklar (kaynak gösterilmedi):**
- Apple'ın "Asking permission to use notifications" sayfası bu çalışmada açılmadı (V-09).
- Apple D-U-N-S yardım sayfası ve Google'ın 2023 tarihli "Boosting trust and transparency in Google Play" blog yazısı yalnızca arama sonucunda görüldü, açılmadı. Bunların yerine birincil yardım sayfaları (K-20, K-21, K-22) kullanıldı.
- Google Play geliştirici hesabı kayıt ücreti ve Firebase ücretlendirmesi bu çalışmada doğrulanmadı (AS-19).
- WCAG 2.2 ana sayfası 2.5.8 maddesine kadar okunamadı. Bu madde için K-16 kullanıldı.
- KVKK Kurumu (kvkk.gov.tr) sayfaları açılmadı. Silme yönetmeliği için Resmî Gazete metni (K-19) kullanıldı.
- TA dokümanından aktarılan teknik bilgiler BA tarafından ayrıca doğrulanmadı. Bunlar: veri konumunun değiştirilememesi, platform tabanı, kullandıkça öde planı gereksinimi ve Google Play hizmetleri olmayan cihazlarda push desteğinin olmaması. Bu bilgiler metinde "TA'ya göre" diye belirtilmiştir.

## 11. Değişiklik notu

Önceki sürüme (outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run2.md) göre yapılan değişiklikler aşağıdadır. Hiçbir FR, NFR veya US ID'si kaldırılmadı ya da yeniden numaralandırılmadı. Yeni madde olarak NFR-15 eklendi. Açık sorulardan AS-17 kaldırıldı. AS-19 eklendi ve Yanıt 10 ile daraltıldı. AS-20 eklendi.

| # | Neden | Değişiklik | Etkilenen ID / bölüm |
|---|---|---|---|
| D-01 | Yanıt 6 (AS-13 → "A seçeneği olsun") | Bölüm 3.6 öneri metninden karar kaydına dönüştürüldü. Seçenek A "paydaş tarafından seçildi" olarak işaretlendi, B'nin koşulu ve C'nin reddi kayda geçti | Bölüm 3.6 |
| D-02 | Yanıt 6 | FR-29'un dayanağına paydaş onayı eklendi. FR-02 ve FR-03 dayanaklarına Yanıt 6 eklendi | FR-02, FR-03, FR-29 |
| D-03 | Yanıt 6 | V-13 "yorum" yerine "paydaşça onaylandı" olarak yeniden yazıldı | V-13 |
| D-04 | Yanıt 6 ve TA S-12 | AS-13'ten (a) maddesi çıkarıldı. Kalan (b) hukuki yeterlilik ve (c) saklama yükümlülüğü korundu. Yeterli bulunmazsa izlenecek yol için yeni (d) maddesi eklendi (TA S-12) | AS-13 |
| D-05 | Yanıt 7 (AS-14 → "kategoriler uygun") | FR-13'te kategoriler ve zorunluluk "BA önerisi" yerine paydaş onaylı yazıldı. Listenin ilk sürümde sabit olduğu ve yöneticinin düzenleyemediği eklendi. Bu son kural BA önerisidir (V-15) | FR-13, V-15 |
| D-06 | Yanıt 7 | US-07 KK-4 onaylı dört kategoriyle ve düzenleme seçeneğinin olmamasıyla test edilebilir hâle getirildi. Kapsam dışına "kategori ekleme veya düzenleme" eklendi | US-07, Bölüm 7 |
| D-07 | Yanıt 7 ve Yanıt 11 (TA S-16) | AS-14'ten kategori onayı (Yanıt 7) ve karakter sınırı (Yanıt 11) çıkarıldı. Yalnızca serbest metnin bildirimde gösterilmemesi kararının açık onayı (c) açık kaldı | AS-14 |
| D-08 | Yanıt 8 (AS-16 → "salon") | Bölüm 3.4'teki Apple 4.2.6 satırının iş etkisi güncellendi. Apple kurumsal kayıt (K-20), Google Play kurumsal hesap (K-21) ve yeni bireysel hesapta test şartı (K-22) satırları eklendi. Üç yeni kaynak araştırıldı | Bölüm 3.4, K-20, K-21, K-22 |
| D-09 | Yanıt 8 | Yeni NFR-15 eklendi: salonun hesabıyla yayın, satıcı adı ve yayın ön koşul süreleri | NFR-15 |
| D-10 | Yanıt 8 | AS-16 kalan sorularla yeniden yazıldı: salonun hukuki yapısı ve hesap türü (a), hesap ve D-U-N-S işlerinin sahibi ve takvim etkisi (b), çoğaltma planı (c). V-18 eklendi. V-16 Yanıt 8 ile desteklendi. Persona P-3 ve kapsam dışı ("başka salonlara çoğaltma") güncellendi | AS-16, V-16, V-18, Bölüm 2, 7 |
| D-11 | Yanıt 9 (AS-17 → "ayarlanabilir") | FR-22'ye salon geneli başlangıç aralığı ayarı eklendi (seçenekler 5–60 dk, varsayılan 15 dk, mevcut randevulara etkisiz). FR-09'a aralığa göre üretim ve FR-22 ertelenirse varsayılanın kullanılacağı eklendi | FR-09, FR-22 |
| D-12 | Yanıt 9 | US-05'e yeni KK-2 (varsayılan aralık) eklendi; eski KK-2–KK-4 sırasıyla KK-3–KK-5 oldu. US-11'e KK-3 (aralık ayarı ve mevcut randevuya etkisi) eklendi. Bölüm 3.5'teki müsait saat satırları aralık seçeneklerine göre yeniden yazıldı. V-04'e varsayılan 15 dk eklendi, V-17 eklendi. AS-17 açık sorulardan çıkarıldı. Kapsam dışına "hizmete veya çalışana göre ayrı aralık" eklendi | US-05, US-11, Bölüm 3.5, V-04, V-17, AS-17, Bölüm 7 |
| D-13 | TA S-04 (tanımsız "değiştirildi" bildirimi) | FR-18 (b)'den "veya değiştirildi" çıkarıldı. Salonun randevu değiştirmesi kapsam dışına alındı. Paydaş kararı için AS-20 eklendi | FR-18, Bölüm 7, AS-20 |
| D-14 | TA S-04 kapsamında tespit edilen tutarsızlık | US-10 KK-2'deki çalışana değişiklik bildirimi bir FR'ye dayanmıyordu. Yapılan düzeltmeler:<br>- FR-19'a müşterinin randevu taşıması eklendi.<br>- US-10'un FR atıflarına FR-18 ve FR-19 eklendi.<br>- US-10 KK-2 ölçülebilir hâle getirildi, KK-3 (hatırlatmaların yeniden planlanması) eklendi.<br>- NFR-04 olay listesine "taşıma" eklendi | FR-19, US-10, NFR-04 |
| D-15 | TA S-05 (tanımsız "rapor, arama, dışa aktarım" atıfları) | FR-29'daki "raporlar" yerine mevcut takvim ve randevu ekranları (FR-15) yazıldı. İleride eklenecek işlevler için koşul cümlesi eklendi. NFR-14'teki ölçüt mevcut ekranlara ve saklanan verinin tamamına bağlandı. US-12 KK-2 buna göre güncellendi. Kapsam dışındaki gelişmiş raporlama maddesine arama ve dışa aktarım eklendi | FR-29, NFR-14, US-12, Bölüm 7 |
| D-16 | Yanıt 12 (TA S-10 → "sadece uygulama içi yeterli") | Yapılan değişiklikler:<br>- FR-17 push alamayan cihazları açıkça kapsayacak şekilde genişletildi, bu cihazlar için yedek kanal gerekmediği eklendi.<br>- FR-27'nin kapsamı yalnızca bildirime izin vermeyen kullanıcılarla sınırlandı.<br>- V-19 paydaş onaylı olarak yeniden yazıldı, V-03 güncellendi.<br>- US-07 KK-3 güncellendi, US-08'e KK-7 eklendi.<br>- Bölüm 3.5'e "yalnızca uygulama içi bilgi" seçeneği eklendi. Bölüm 3.2 güncellendi.<br>- Kapsam dışına "push alamayan cihazlar için yedek kanal" eklendi | FR-17, FR-27, V-03, V-19, US-07, US-08, Bölüm 3.2, 3.5, 7 |
| D-17 | Yanıt 10 (TA S-15 → "salon") | Yapılan değişiklikler:<br>- NFR-15 altyapı projesi ve fatura sahipliğini kapsayacak şekilde genişletildi.<br>- V-20 eklendi. V-16 ve persona P-3 güncellendi.<br>- AS-19 eklendi; proje ve fatura sahibi sorusu cevaplandığı için yalnızca bütçe ve maliyet uyarısı (b) ile veri sorumlusu ilişkisi (c) açık bırakıldı.<br>- AS-08'e projenin salon hesabında kurulacağı notu eklendi | NFR-15, V-16, V-20, AS-08, AS-19, Bölüm 2 |
| D-18 | Yanıt 11 (TA S-16 → "200 karakter uygun") | Yapılan değişiklikler:<br>- FR-13'e en fazla 200 karakterlik açıklama ve uyarı metni eklendi.<br>- NFR-08'e tek serbest metin alanının sınırı eklendi.<br>- US-07'ye KK-5 (karakter sınırı testi) eklendi.<br>- V-04, V-08 ve V-15 güncellendi.<br>- Bölüm 3.5'teki iptal nedeni satırı ve kapsam dışı ("200 karakteri aşan açıklama") güncellendi | FR-13, NFR-08, US-07, V-04, V-08, V-15, Bölüm 3.5, 7 |
| D-19 | TA S-01, S-02, S-03, S-06, S-08, S-09, S-11, S-13, S-14 (paydaş kararı gerektiren sorular) | İlgili AS'lere TA'nın geçici kararları ve gerekçeleri not edildi: AS-02, AS-03, AS-04, AS-05, AS-07 (Android 7.0 / iOS 15 tabanı), AS-08 ("acil" işareti), AS-10, AS-15, AS-18 ("anonimleştir" işlevi sorusu). FR-20'ye AS-18 atfı eklendi | AS-02–AS-05, AS-07, AS-08, AS-10, AS-15, AS-18, FR-20 |
| D-20 | Kaynakların yeniden doğrulanması | Tüm kaynaklar bu çalışmada yeniden açıldı. Güncellenen bilgiler: <br>- Salon Randevu'nun uygulama içi satın alma aralığı 4,99–107,99 $'dan 4,99–949,00 $'a güncellendi (K-7).<br>- Kuaförüm Yanımda'da salonun müşteri iptal yetkisini yönetmesi eklendi (K-8); Bölüm 3.2 ve FR-12 dayanağına işlendi.<br>- Apple hesap silme rehberindeki "provide a confirmation when the deletion is complete" ifadesi FR-03, NFR-07 ve US-02 KK-4'e eklendi (K-18).<br>- Yönetmelik md. 11/3 eklendi: politika yükümlülüğü olmayanlarda üç ay (K-19).<br>- WCAG 1.4.3'te büyük metin için 3:1 istisnası eklendi (K-15).<br>- Booksy'de çalışanın kendi saatlerini yönetmesi ifadesi, kaynaktaki "view and manage their own appointments" ifadesine göre düzeltildi (Bölüm 2, 3.1, AS-05).<br>- Kapsam dışındaki pazarlama bildirimleri gerekçesi K-13'e bağlandı | Bölüm 2, 3.1, 3.2, 3.3, 7, 10; FR-03, FR-12, NFR-07, US-02, AS-05 |
| D-21 | Bütünlük | Bölüm 1'e yanıt numaralandırma tablosu eklendi (önceki tur Yanıt 1–5, bu tur Yanıt 6–12). Girdi bloğuna aynı turda gelen ek talimat eklendi. Kaynaklar bölümüne TA dokümanından aktarılan, BA tarafından doğrulanmamış bilgiler için not eklendi | Doküman başlığı, Bölüm 1, 10 |
