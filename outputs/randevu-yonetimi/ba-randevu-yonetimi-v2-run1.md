# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ba v2
- **Girdi:** Flutter ile Android ve iOS cihazlarda çalışacak bir randevu yönetim uygulaması hazırlayacağız. Firebase kullanarak kullanıcı kaydı, randevu oluşturma, iptal etme, çalışan takvimleri ve bildirim özelliklerini değerlendireceğiz. Güzellik salonları ve kuaförler için mevcut uygulamaları araştırarak iş gerekliliklerini belirle ve bize bir iş analizi dokümanı hazırla.

---

## 1. Özet

Ürün, güzellik salonu ve kuaför işletmelerinin müşterilerinden mobil uygulama üzerinden randevu almasını sağlar. Android ve iOS'ta çalışır. Görevde istenen beş ana yetenek vardır: kullanıcı kaydı, randevu oluşturma, randevu iptali, çalışan takvimleri ve bildirimler. Flutter ve Firebase görevde ekip tarafından verilmiş kısıtlardır. Bu doküman bu kısıtlar üzerine ek bir teknoloji kararı vermez.

Araştırmadan çıkan başlıca bulgular:
- **Rakiplerde ortak çekirdek:** Fresha, Booksy, Vagaro ve Türkiye'deki salon uygulamaları aynı temel özellikleri sunuyor: 7/24 çevrimiçi randevu, çalışan bazlı takvim, otomatik hatırlatma ve iptal politikası. Ön ödeme, kapora ve iptal ücreti ise ücretli ödeme altyapısına dayanan ek özelliklerdir. Bu ilk sürümde kapsam dışı önerilmiştir.
- **Mağaza politikaları zorunlu iş gereksinimi doğuruyor:** Hesap oluşturan uygulamalar uygulama içinden hesap silme sunmak zorundadır (Apple 5.1.1(v); Google Play ayrıca web üzerinden silme bağlantısı istiyor). Pazarlama amaçlı push bildirimi açık onay gerektirir (Apple 4.5.4). Android 13 ve üzerinde bildirimler yeni kurulumda varsayılan olarak kapalıdır.
- **Mevzuat:** Uygulama müşteri ve çalışan kişisel verisi işler. Bu nedenle 6698 sayılı KVKK'daki aydınlatma, ilgili kişi hakları, silme ve 30 günlük başvuru cevap süresi yükümlülükleri geçerlidir. Tanıtım içermeyen randevu bildirimleri, Ticari Elektronik İletiler Yönetmeliği md. 6 kapsamında önceden onay gerektirmeyen ileti türüne yakın görünüyor. Bu değerlendirme hukuki teyit gerektirir.
- **Hatırlatmanın iş değeri:** Klinik randevularında SMS hatırlatmanın katılımı artırdığına dair meta-analiz kanıtı var. Salon sektörüne doğrudan aktarılabilirliği bir varsayımdır.

İlk sürümün (MVP) sınırı, Fonksiyonel gereksinimler bölümündeki M öncelikli maddelerle tanımlanmıştır.

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Tanım | Temel ihtiyaç | Başlıca acı noktası |
|---|---|---|---|
| **P-1 Müşteri (Elif, 32)** | Düzenli olarak kuaföre ve manikür hizmetine giden, telefonla aramak yerine uygulamadan randevu almak isteyen kişi | Uygun saati görüp hızlıca randevu almak, gerekirse iptal etmek, randevuyu unutmamak | Mesai saatinde salonu arayamamak, randevuyu unutmak |
| **P-2 Çalışan (Murat, kuaför)** | Salonda hizmet veren personel | Kendi günlük/haftalık randevularını görmek, izin ve mola günlerini tanımlamak, yeni randevu ve iptallerden haberdar olmak | Çakışan randevular, son dakika iptallerinden geç haberdar olmak |
| **P-3 İşletme sahibi veya yöneticisi (Ayşe, salon sahibi)** | İşletmeyi, hizmetleri ve çalışanları yöneten kişi | Hizmetleri ve süreleri tanımlamak, tüm çalışanların takvimini görmek, telefonla gelen randevuyu sisteme girmek, iptal kurallarını belirlemek | Boş kalan saatler, gelmeyen müşteriler (no-show), çift rezervasyon |

Rakip incelemesi, yöneticinin tüm çalışan takvimlerini görebildiği ve çalışanın yalnızca kendi randevusunu yönettiği rol ayrımının yaygın olduğunu gösteriyor (Booksy, bkz. Kaynaklar).

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Rakipler ve mevcut çözümler

| Çözüm | Öne çıkan özellikler (kaynağa göre) | Güçlü yön | Zayıf yön / not |
|---|---|---|---|
| **Fresha** (uluslararası) | 7/24 çevrimiçi kendi kendine randevu; sınırsız randevu, müşteri, şube ve ekip için takvim; e-posta ve SMS ile otomatik hatırlatma; esnek iptal politikası; kapora veya ön ödeme (ürün sayfası). Fiyatlandırma: tek kişi için aylık 19,95 $, ekip planında takvimde randevu alınabilen kişi başına aylık 14,95 $. Ayda 100 ücretsiz otomatik SMS/WhatsApp mesajı (ekip planında kişi başına), sonrası mesaj başına 0,02 $. Pazar yerinden gelen yeni müşteride tek seferlik %20 komisyon (fiyat sayfası) | Kapsamlı. Hatırlatma ve politika araçları hazır | SMS ve pazar yeri komisyonu ek maliyet getiriyor. Ödeme özellikleri ek ücretli |
| **Booksy** (uluslararası) | Müşteri müsaitliği görüp 7/24 randevu alır. Her çalışanın kendi profili ve takvimi var. Sahip, web ve tablette çalışan takvimlerine erişir. Sınırsız otomatik hatırlatma. Bekleme listesi uyarısı. İptal penceresi, kayıtlı kart üzerinden iptal ücreti, kapora ve "güvenilir müşteri" muafiyeti. Fiyat: aylık 29,99 $ + vergi, ek kullanıcı başına +20 $ | Çalışan ve yönetici rol ayrımı net. İptal penceresi kavramı açık | Bulunan kaynaklarda iptal penceresinin iptali tamamen engelleyip engellemediği netleşmedi. Booksy'nin kendi metni pencere içindeki iptalin ücrete bağlandığını ima ediyor |
| **Vagaro** (uluslararası) | Web sitesi, Instagram, Facebook, Google ve Apple Haritalar üzerinden randevu; özelleştirilebilir takvim ayarları; otomatik hatırlatma; no-show ücreti veya kapora içeren iptal politikaları; boşlukları kurala göre (ilk gelen veya en yüksek değer) dolduran otomatik bekleme listesi | Boş saatleri doldurmaya yönelik otomasyon | Sayfada hatırlatma kanalları (SMS, e-posta, push) belirtilmiyor |
| **Google Takvim randevu planları** (platform ekosistemindeki yerleşik çözüm) | Uygun Workspace hesaplarında otomatik rezervasyon sayfası; genel müsaitlik ve saat dilimi; rezervasyon penceresi (en ileri tarih ve en kısa önceden bildirim süresi); belirli tarihler için müsaitlik değişikliği; randevular arası tampon süre; günlük en fazla randevu sınırı; randevu başına en fazla 5 hatırlatma e-postası (metin düzenlenemez). Bazı özellikler ücretli Workspace veya Google One aboneliği gerektiriyor | Ek uygulama gerektirmiyor. Tampon ve önceden bildirim süresi gibi kavramlar iyi örnek | Çok çalışanlı salon, hizmet süresi ve çalışan eşleştirmesi gibi sektöre özgü ihtiyaçlar sayfada yer almıyor |
| **Salon Randevu İşletme Yönetimi** (Türkiye) | Instagram, WhatsApp ve Google'da paylaşılan işletme bağlantısıyla çevrimiçi randevu; randevu hatırlatma SMS'i; personel yönetimi ve prim hesaplama; stok ve gelir-gider takibi. Açıklamada "Ücretsiz kullanım, aylık ödeme yok" deniyor, uygulama içi satın alımlar ise ücretli paketler içeriyor | Yerel dil. Muhasebe ve prim gibi geniş işletme işlevleri | App Store'da (ABD mağazası) yalnızca 1 değerlendirme var, kullanıcı kanıtı zayıf. Ücretsiz iddiası geliştiricinin tanıtım ifadesidir |
| **Kuaförüm Yanımda** (Türkiye) | Randevuların onay adımı olmadan takvimde kesinleşmesi (otomatik onay); toplu randevu onayı; tatil, personel izni ve yemek molası tanımlayarak çakışmayı önleme; bildirim merkezi; özelleştirilebilir SMS ve bildirim içeriği; müşteriler için hesap silme seçeneği (sürüm 2.1.4). App Store (TR) puanı 4,6 (75 değerlendirme) | Mola ve izin yönetimi, onay modları ve hesap silme Türkiye pazarındaki beklentiyi gösteriyor | Yalnızca mağaza açıklaması incelendi, kullanım testi yapılmadı |

### 3.2 Çıkarımlar (BA yorumu)
- **Sektörde temel beklenti:** çevrimiçi randevu, çalışan bazlı takvim, mola ve izin tanımı, otomatik hatırlatma ve iptal kuralı. Bu ürün de bu özellikleri M önceliğinde ele alır.
- **Ek modüller:** Kapora, iptal ücreti, pazar yeri, muhasebe ve stok olgun rakiplerde var. Ancak görev metninde istenmedikleri için kapsam dışı bırakılmıştır.
- **Hatırlatma kanalı maliyeti:** Rakiplerde SMS ücretli veya kotalı (Fresha). Görevdeki "bildirim" ifadesi bu dokümanda uygulama bildirimi (push) olarak yorumlanmıştır. SMS yedek kanal olarak C önceliğindedir (bkz. Açık sorular).

### 3.3 Alan standartları ve düzenlemeler
- **KVKK (6698 sayılı Kanun):**
  - md. 5/2-c, sözleşmenin kurulması veya ifasıyla doğrudan ilgili verinin açık rıza aranmadan işlenmesine izin verir. Randevu verisi bu şarta dayanabilir; nihai değerlendirme hukuki teyit gerektirir.
  - md. 10, veri elde edilirken aydınlatma yapılmasını zorunlu kılar.
  - md. 11, ilgili kişiye bilgi talep etme, düzeltme ve silme gibi haklar tanır.
  - md. 7, işleme sebebi ortadan kalkınca verinin silinmesini ister.
  - md. 13/2, başvuruların "en geç otuz gün içinde" sonuçlandırılmasını öngörür.
  - md. 9, kişisel verinin yurt dışına aktarımını şartlara bağlar. İncelenen metin kanunun 2016 tarihli kabul metnidir. Sonraki değişiklikler, özellikle md. 9'daki değişiklikler, bu çalışmada doğrulanmadı (bkz. Açık sorular).
- **Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik md. 6:**
  - Üyelik, satın alma ve "benzeri durumlara ilişkin bildirimler" için önceden onay aranmaz. Şart, bu bildirimlerde herhangi bir mal veya hizmetin özendirilmemesi ya da tanıtılmamasıdır.
  - Bu kapsamdaki iletilerde İYS kontrolü yapılmaz (md. 6/5).
  - Randevu hatırlatmasının bu muafiyete girdiği BA yorumudur. Yönetmelikte "randevu" adıyla geçmiyor.
- **Erişilebilirlik (WCAG 2.2):**
  - Metin kontrastı en az 4.5:1 olmalı (1.4.3).
  - Metin, içerik veya işlev kaybı olmadan %200'e kadar büyütülebilmeli (1.4.4).
  - Dokunma hedefi en az 24×24 CSS piksel olmalı (2.5.8).
  - WCAG web içeriği için yazılmıştır. Mobil uygulamaya hedef olarak uyarlanması bir varsayımdır.

### 3.4 Platform ve mağaza politikaları (iş etkisi)

| Politika | İçerik | Ürüne iş etkisi |
|---|---|---|
| Apple 5.1.1(v) | Hesap oluşturma sunan uygulama, uygulama içinden hesap silme sunmalı. Temel işlevle ilgisiz kişisel veri istenemez | Hesap silme M öncelikli (FR-03). Kayıtta yalnızca gerekli alanlar istenir |
| Apple 4.5.4 | Push bildirimi uygulamanın çalışması için zorunlu tutulamaz. Promosyon amaçlı push için açık onay ve uygulama içinden vazgeçme yolu gerekir | Bildirim izni reddedilse de randevu akışı çalışmalı (FR-17). Kampanya bildirimleri kapsam dışı |
| Apple 4.8 | Üçüncü taraf veya sosyal giriş (ör. Google ile giriş) birincil hesap için kullanılıyorsa, veri toplamayı ad ve e-postayla sınırlayan, e-postayı gizleme imkânı veren eşdeğer bir giriş seçeneği de sunulmalı | Sosyal giriş seçilirse ek giriş seçeneği gerekir (bkz. Açık sorular) |
| Google Play hesap silme | Uygulama içi silme yolu ve hesap ile ilişkili veriyi silme talebi için web bağlantısı zorunlu. Güvenlik veya mevzuat gereği saklanan veri kullanıcıya bildirilmeli | Web üzerinden silme talebi kanalı gerekir (FR-03) |
| Google Play Data safety | Tüm uygulamalar veri güvenliği formunu doldurmalı. Üçüncü taraf kütüphane ve SDK'ların topladığı veriler dahil beyan edilmeli. Gizlilik politikası bağlantısı zorunlu | Gizlilik politikası ve veri envanteri yayın öncesi hazır olmalı (FR-02, NFR-08) |
| Android 13+ bildirim izni | `POST_NOTIFICATIONS` çalışma zamanı izni gerekir. Yeni kurulumda bildirimler varsayılan olarak kapalıdır. İzin, işlevle ilgili bir bağlamda istenmesi önerilir (ör. kullanıcı bir sipariş verdiğinde) | İzin, ilk randevu oluşturulduğunda gerekçesiyle istenmeli (FR-16). İzin vermeyen kullanıcı da randevusunu uygulamada görebilmeli |

iOS bildirim izni için Apple'ın ilgili dokümantasyon sayfası bu çalışmada okunamadı. iOS'taki izin davranışı doğrulanmamış kabul edilmiştir (bkz. Varsayımlar).

### 3.5 Yöntemler (iş açısından artı/eksi, seçim TA'ya bırakılır)

| Konu | Seçenek | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Müsait saat (slot) üretimi** | Sabit aralıklı slot (ör. 15 dakikada bir başlangıç) | Müşteri için anlaşılır, tahmin edilebilir saatler | Hizmet süresiyle uyumsuz kısa boşluklar kalabilir |
| | Hizmet süresi + tampon süreye göre dinamik slot (Google Takvim'deki tampon ve önceden bildirim süresi kavramlarına benzer) | Takvim doluluğu artar. Hazırlık ve temizlik payı bırakılır | Kurgu ve test yükü artar. Yönetici için ayar sayısı artar |
| **Çakışma önleme** | Randevu kaydı anında müsaitliğin yeniden kontrol edilmesi | İki müşteri aynı slotu aynı anda seçerse yalnızca biri kazanır. Çift rezervasyon önlenir | Kaybeden müşteriye "saat doldu" hatası gösterilir, kullanıcı deneyimi kısa süreli bozulur |
| **Randevu onayı** | Otomatik onay (Kuaförüm Yanımda'da var) | Müşteri anında kesin yanıt alır. Salon iş yükü azalır | Salon istenmeyen randevuyu engelleyemez |
| | Manuel onay | Salon kontrolü artar | Müşteri bekler. Onay gecikirse randevu kaybı olabilir |
| **Hatırlatma kanalı** | Push bildirimi | Mesaj başına maliyet yok (BA varsayımı) | Kullanıcı izin vermezse ulaşmaz. Android 13+ yeni kurulumda varsayılan olarak kapalı |
| | SMS | İzinden bağımsız ulaşır. Klinik çalışmalarında katılımı artırdığı gösterilmiş (aşağıda) | Mesaj başına maliyet (ör. Fresha'da kota sonrası 0,02 $). Telefon numarası toplamayı gerektirir |
| | E-posta | Düşük maliyet | Fark edilme oranı hakkında kanıt toplanmadı |
| **İptal kuralı** | İptal penceresi: randevudan X saat öncesine kadar serbest iptal | No-show ve son dakika iptali riskini azaltır (Booksy'de bu pencere iptal ücretiyle birlikte kullanılıyor) | Pencere içinde iptali tamamen engellemek müşteriyi gelmemeye itebilir. Ücret olmadan yaptırımın etkisi belirsiz |

**Hatırlatmanın etkisi:** Health Services Research'te yayımlanan 2012 tarihli meta-analiz 18 çalışmayı incelemiştir (8 RKÇ ve 10 kontrollü gözlemsel çalışma). Birinci basamak ve hastane polikliniklerindeki randevularda yalnızca 8 RKÇ birleştirilmiş ve SMS hatırlatmanın katılım olasılığını artırdığı bulunmuştur (olasılık oranı, OR 1,48). Bu bulgu sağlık kliniklerine aittir. Salon müşterilerine aktarılabilirliği bir varsayımdır.

## 4. Fonksiyonel gereksinimler

**M önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S maddelerinin ilk sürümde olması beklenir ama ertelenebilir. C maddeleri kaynak kalırsa yapılır. Bu sürümde yapılmayacaklar "Kapsam dışı maddeler" bölümündedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kullanıcı uygulamada hesap oluşturabilmeli ve giriş yapabilmelidir. Kayıtta yalnızca randevu için gerekli bilgiler istenir: ad, iletişim bilgisi (e-posta ve/veya telefon, bkz. Açık sorular) ve şifre ya da doğrulama kodu. Şifre sıfırlama veya yeniden doğrulama yolu bulunur. | M | Görev; Apple 5.1.1(v) |
| FR-02 | Kayıt sırasında KVKK aydınlatma metni ve gizlilik politikası gösterilmeli, bu metinlere uygulama içinden her zaman erişilebilmelidir. Aydınlatma metni veri sorumlusunun kimliğini, işleme amacını, aktarım alıcılarını, toplama yöntemi ve hukuki sebebi ile md. 11 haklarını içerir. | M | KVKK md. 10; Google Play Data safety |
| FR-03 | Kullanıcı hesabını ve ilişkili kişisel verisini uygulama içinden silebilmelidir. Ayrıca uygulama dışından (web bağlantısı) silme talebi gönderilebilmelidir. Mevzuat gereği saklanan veriler kullanıcıya bildirilir. | M | Apple 5.1.1(v); Google Play hesap silme; KVKK md. 7 |
| FR-04 | Sistem üç rolü desteklemelidir: Müşteri, Çalışan, İşletme yöneticisi. Her rol yalnızca yetkili olduğu verileri görür ve değiştirir (bkz. NFR-06). | M | Görev; Booksy rol ayrımı |
| FR-05 | İşletme yöneticisi işletme profilini tanımlayabilmelidir: ad, adres, iletişim, haftalık çalışma saatleri ve kapalı günler (resmî tatil dahil). | M | Görev; Kuaförüm Yanımda |
| FR-06 | İşletme yöneticisi hizmetleri tanımlayabilmelidir: ad, süre (dakika), bilgi amaçlı fiyat ve hizmeti verebilen çalışanlar. | M | Rakip incelemesi |
| FR-07 | Yönetici çalışan ekleyip çıkarabilmeli, çalışanı uygulamaya davet edebilmelidir. | M | Görev (çalışan takvimleri) |
| FR-08 | Her çalışan için haftalık çalışma saatleri, mola aralıkları ve izin günleri tanımlanabilmelidir. Yönetici tüm çalışanlar için, çalışan kendi takvimi için bu tanımı yapar (çalışanın kendi takvimini düzenleyebilmesi açık sorudur). | M | Görev; Kuaförüm Yanımda; Booksy |
| FR-09 | Sistem, seçilen hizmet ve çalışan için müsait başlangıç saatlerini göstermelidir. Saatler işletme çalışma saatleri, çalışan saatleri, mola ve izin, mevcut randevular ve hizmet süresi dikkate alınarak hesaplanır. Hizmet süresi bir molaya veya kapanış saatine taşan saat gösterilmez. | M | Yöntemler 3.5 |
| FR-10 | Müşteri işletmeyi, hizmeti, çalışanı ("fark etmez" seçeneği dahil) ve müsait saati seçerek randevu oluşturabilmelidir. "Fark etmez" seçilirse randevu, hizmeti verebilen müsait çalışanlardan birine atanır (atama kuralı açık sorudur). | M | Görev |
| FR-11 | Aynı çalışana aynı zaman aralığında birden fazla randevu kaydedilmemelidir. İki kullanıcı aynı slotu aynı anda seçerse yalnızca biri başarılı olur, diğerine güncel müsait saatler gösterilir. | M | Yöntemler 3.5; Kuaförüm Yanımda (çakışma önleme) |
| FR-12 | Müşteri kendi randevusunu iptal edebilmelidir. Randevuya işletmenin belirlediği iptal süresinden (varsayılan 2 saat, ayarlanabilir) daha az kaldıysa uygulama iptal yerine işletmeyle iletişime geçme bilgisini gösterir (bkz. Açık sorular). İptal edilen saat yeniden müsait olur. | M | Görev; Booksy iptal penceresi |
| FR-13 | Yönetici ve ilgili çalışan randevuyu iptal edebilmeli, isteğe bağlı bir iptal nedeni girebilmelidir. | M | Görev |
| FR-14 | Müşteri yaklaşan ve geçmiş randevularını durumlarıyla (aktif, iptal, tamamlandı, gelmedi) listeleyebilmelidir. | M | Görev |
| FR-15 | Çalışan kendi randevularını günlük ve haftalık takvim görünümünde görebilmelidir. Yönetici tüm çalışanların takvimini çalışan bazında (ör. sütunlar halinde) görebilmelidir. | M | Görev; Fresha (takvim sütunları); Booksy |
| FR-16 | Uygulama bildirim iznini ilk randevu oluşturma gibi işlevle ilgili bir anda, gerekçesini açıklayarak istemelidir. İzin uygulama açılışında istenmez. | M | Android bildirim izni rehberi |
| FR-17 | Bildirim izni verilmese bile tüm randevu işlemleri çalışmalı. Randevu bilgisi uygulama içinde görülebilmeli. Bildirimleri kapalı kullanıcıya uygulama içinde bildirimleri açma yolu gösterilmelidir. | M | Apple 4.5.4 |
| FR-18 | Müşteriye şu olaylarda bildirim gönderilmelidir: randevu oluşturuldu, randevu işletme tarafından iptal edildi veya değiştirildi, randevu hatırlatması (varsayılan olarak randevudan 24 saat önce). Bildirim içeriği tanıtım veya kampanya içermez. | M | Görev; Ticari Elektronik İletiler Yönetmeliği md. 6/2; meta-analiz |
| FR-19 | İlgili çalışana ve yöneticiye şu olaylarda bildirim gönderilmelidir: yeni randevu ve müşteri iptali. | M | Görev; Booksy |
| FR-20 | Yönetici veya çalışan, telefonla veya salonda alınan randevuyu müşteri adına sisteme girebilmelidir. Müşterinin uygulama hesabı olmayabilir; bu durumda ad ve telefon girilir. | S | Rakip incelemesi (takvimin tek kaynak olması) |
| FR-21 | Müşteri, iptal süresi dolmamış randevusunu başka bir saate taşıyabilmelidir (yeniden planlama). Taşıma, eski randevuyu ancak yeni saat başarıyla alınınca serbest bırakır. | S | Booksy (yeniden planlama penceresi) |
| FR-22 | Yönetici şu randevu kurallarını ayarlayabilmelidir: en kısa önceden randevu süresi (ör. en az 1 saat önce), en ileri randevu tarihi (ör. 60 gün), hizmetler arası tampon süre ve iptal süresi. | S | Google Takvim randevu planları |
| FR-23 | Yönetici veya çalışan, gelmeyen müşterinin randevusunu "gelmedi" (no-show) olarak işaretleyebilmeli; yönetici müşteri bazında gelmeme sayısını görebilmelidir. | S | Rakiplerde no-show odaklı özellikler |
| FR-24 | Hatırlatma zamanı işletme tarafından ayarlanabilmeli (ör. 24 saat ve/veya 2 saat önce). | S | Google Takvim (çoklu hatırlatma); Salon Randevu |
| FR-25 | Yönetici randevu onay modunu seçebilmelidir: otomatik onay (varsayılan) veya manuel onay. Manuel modda müşteri "onay bekliyor" durumunu görür ve onay veya reddedildiğinde bildirim alır. | C | Kuaförüm Yanımda |
| FR-26 | Dolu bir saat için müşteri bekleme listesine yazılabilmeli; saat boşalırsa bildirim alabilmelidir. | C | Booksy; Vagaro |
| FR-27 | Push bildirimi alamayan müşteriye SMS veya e-posta ile yedek hatırlatma gönderilebilmelidir. | C | Fresha; Salon Randevu; maliyet bkz. Açık sorular |
| FR-28 | Müşteri randevusunu cihazının takvimine ekleyebilmelidir. | C | BA önerisi (kaynaksız; kullanıcı kolaylığı) |

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt |
|---|---|---|
| NFR-01 | **Platform:** Uygulama Android ve iOS telefonlarda aynı işlevleri sunar. | FR-01 ile FR-19 arasındaki M gereksinimlerinin tamamı iki platformda kabul testinden geçer. Desteklenen en düşük işletim sistemi sürümleri açık sorudur. |
| NFR-02 | **Çift rezervasyon olmaması** | Eşzamanlı aynı slot denemesi testlerinde (en az 50 eşzamanlı deneme senaryosu) aynı çalışan ve zaman aralığına kaydedilen randevu sayısı en fazla 1 olur. |
| NFR-03 | **Yanıt süresi** | 4G bağlantıda müsait saat listesi, kullanıcının çalışan veya tarih seçiminden sonra 95. persentilde 2 saniye içinde görüntülenir. Randevu oluşturma onayı 3 saniye içinde gösterilir. |
| NFR-04 | **Bildirim zamanlaması** | Olay bildirimleri (oluşturma ve iptal) olaydan sonra 1 dakika içinde gönderilir. Hatırlatmalar planlanan zamandan en fazla ±5 dakika sapmayla gönderilir. Ölçüm gönderim anına göredir; cihaza teslim işletim sistemine bağlıdır. |
| NFR-05 | **Kullanılabilirlik** | Kayıtlı bir müşteri, uygulamayı açtıktan sonra en fazla 5 ekran veya adımda randevu oluşturabilir. İlk kez kullanan 5 test katılımcısının en az 4'ü randevu oluşturmayı yardımsız 3 dakikada tamamlar. |
| NFR-06 | **Yetkilendirme ve gizlilik** | Müşteri yalnızca kendi randevu ve profil verisini görür. Çalışan müşterinin yalnızca randevu için gerekli bilgilerini (ad, iletişim, hizmet) görür. Yetkisiz erişim testlerinde başarılı erişim sayısı 0'dır. |
| NFR-07 | **KVKK başvuruları** | İlgili kişinin bilgi, düzeltme ve silme talepleri en geç 30 gün içinde sonuçlandırılır (KVKK md. 13/2). Uygulama içi hesap silme talebi başlatıldıktan sonra hesap kullanılamaz hale gelir. |
| NFR-08 | **Veri en aza indirme** | Kayıtta zorunlu alan sayısı en fazla 3'tür (ad, iletişim bilgisi, şifre veya doğrulama). Mağaza veri beyanları (Data safety ve App Store gizlilik bilgileri) üçüncü taraf kütüphanelerin topladığı veriler dahil yayın öncesi hazırlanır. |
| NFR-09 | **Erişilebilirlik** | Metin kontrastı en az 4.5:1. Sistem yazı boyutu en büyük ayarda (veya %200 büyütmede) içerik kesilmez. Dokunma hedefleri WCAG 2.5.8 hedefinden (24×24 CSS piksel) küçük değildir. Mobil eşdeğeri TA tarafından belirlenir. Ana ekranlar ekran okuyucuyla gezilebilir. |
| NFR-10 | **Dil, saat ve tarih** | Arayüz Türkçedir. Saatler işletmenin yerel saat diliminde (varsayılan Türkiye saati) 24 saat biçiminde gösterilir. Tarih biçimi GG.AA.YYYY'dir. |
| NFR-11 | **Bildirim içeriği** | Hatırlatma ve işlem bildirimleri yalnızca randevu bilgisini içerir: işletme, hizmet, tarih, saat ve çalışan. Tanıtım ve kampanya içermez. Kilit ekranında görünen bildirimde hassas kişisel bilgi (ör. hizmet notları) yer almaz. |
| NFR-12 | **Erişilebilirlik süresi (uptime)** | Randevu alma hizmeti aylık en az %99,5 erişilebilirdir. Hedef değer işletme beklentisine göre teyit edilecektir (bkz. Açık sorular). |
| NFR-13 | **Bağlantı kesintisi** | İnternet yokken uygulama son görüntülenen randevu listesini gösterir ve yeni randevu veya iptal işleminin yapılamadığını açık bir mesajla bildirir. Çevrimdışıyken işlem kuyruğa alınmaz; çakışma riski nedeniyle bu bir BA önerisidir. |

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Kayıt ve giriş** (FR-01, FR-02)
Müşteri olarak hesap oluşturmak istiyorum; böylece randevularımı uygulamadan yönetebilirim.
- **KK-1:** Kayıt ekranında aydınlatma metni ve gizlilik politikası bağlantısı görünür. Kayıt, zorunlu alanlar (en fazla 3) doldurulunca tamamlanır.
- **KK-2:** Kayıtlı kullanıcı uygulamayı yeniden açtığında oturumu sürer veya giriş yapabilir.
- **KK-3:** Şifresini unutan kullanıcı sıfırlama veya doğrulama yoluyla hesabına erişir.

**US-02: Hesap silme** (FR-03)
Müşteri olarak hesabımı ve verilerimi silmek istiyorum.
- **KK-1:** Profil ayarlarında "Hesabı sil" seçeneği vardır ve en fazla 3 adımda talep tamamlanır.
- **KK-2:** Silme sonrası hesapla giriş yapılamaz. Gelecekteki randevular iptal edilir ve işletmeye iptal bildirimi gider.
- **KK-3:** Uygulamaya erişimi olmayan kullanıcı, mağaza sayfasındaki web bağlantısından silme talebi gönderebilir.

**US-03: İşletme ve hizmet tanımı** (FR-04, FR-05, FR-06, FR-07)
İşletme yöneticisi olarak çalışma saatlerimi, hizmetlerimi ve çalışanlarımı tanımlamak istiyorum.
- **KK-1:** Yönetici bir hizmeti ad, süre ve fiyatla oluşturur ve en az bir çalışana atar. Hizmet müşteri tarafında görünür.
- **KK-2:** Kapalı olarak işaretlenen günde müşteriye müsait saat gösterilmez.
- **KK-3:** Çalışan rolündeki kullanıcı hizmet veya fiyat değiştiremez.

**US-04: Çalışan takvimi, mola ve izin** (FR-08, FR-15)
Çalışan olarak çalışma saatlerimi, molalarımı ve izinlerimi görmek ve günlük randevularımı takip etmek istiyorum.
- **KK-1:** Çalışan için 12:30-13:30 molası tanımlandığında bu aralıkla çakışan hiçbir slot müşteriye gösterilmez.
- **KK-2:** İzin günü tanımlanan çalışan o gün için seçilebilir çalışanlar arasında müsait görünmez.
- **KK-3:** Çalışan kendi günlük ve haftalık görünümünde yalnızca kendi randevularını, yönetici tüm çalışanların randevularını çalışan bazında görür.

**US-05: Randevu oluşturma** (FR-09, FR-10, FR-11, FR-18)
Müşteri olarak uygun bir saati seçip randevu almak istiyorum.
- **KK-1:** Müşteri 45 dakikalık bir hizmet seçtiğinde, kapanışa veya molaya 45 dakikadan az kalan başlangıç saatleri listelenmez.
- **KK-2:** Randevu oluşturulunca onay ekranı gösterilir. Bildirim izni varsa onay bildirimi 1 dakika içinde gönderilir (NFR-04).
- **KK-3:** İki müşteri aynı slotu aynı anda onaylarsa yalnızca biri başarılı olur. Diğeri "Bu saat az önce doldu" mesajı ve güncel saat listesiyle karşılaşır (NFR-02).
- **KK-4:** "Fark etmez" seçeneğinde randevu, hizmeti verebilen ve o saatte müsait bir çalışana atanır ve atanan çalışanın adı onayda gösterilir.

**US-06: Randevu iptali (müşteri)** (FR-12, FR-14, FR-19)
Müşteri olarak gidemeyeceğim randevuyu iptal etmek istiyorum.
- **KK-1:** İptal süresi dolmamış randevuda "İptal et" seçeneği çalışır. Randevu durumu "iptal" olur ve saat diğer müşterilere müsait görünür.
- **KK-2:** İlgili çalışana ve yöneticiye iptal bildirimi 1 dakika içinde gönderilir.
- **KK-3:** İptal süresi dolmuşsa uygulama iptal butonu yerine işletmenin iletişim bilgisini ve iptal kuralını gösterir.

**US-07: İşletme tarafından iptal** (FR-13, FR-18)
Yönetici olarak, çalışan hastalandığında randevuları iptal edip müşterileri bilgilendirmek istiyorum.
- **KK-1:** Yönetici iptal ettiğinde müşteriye, varsa nedenle birlikte bildirim gönderilir.
- **KK-2:** Müşterinin randevu listesinde randevu "işletme tarafından iptal edildi" olarak görünür.

**US-08: Hatırlatma ve bildirim izni** (FR-16, FR-17, FR-18, FR-24)
Müşteri olarak randevumu unutmamak için hatırlatma almak istiyorum.
- **KK-1:** Bildirim izni uygulama ilk açıldığında değil, ilk randevu oluşturulurken gerekçesiyle istenir.
- **KK-2:** İzin veren müşteri, randevudan 24 saat önce (±5 dakika) işletme, hizmet, tarih, saat ve çalışan bilgisini içeren hatırlatma alır. Hatırlatma kampanya içermez.
- **KK-3:** İzin vermeyen müşteri randevu oluşturabilir ve randevusunu listede görür. Uygulamada bildirimleri açma yolu gösterilir.
- **KK-4:** İptal edilen randevu için hatırlatma gönderilmez.

**US-09: Telefonla gelen randevuyu girme** (FR-20)
Yönetici olarak telefonla arayan müşterinin randevusunu sisteme girmek istiyorum; böylece takvim tek kaynak olur.
- **KK-1:** Yönetici, hesabı olmayan müşteri için ad ve telefonla randevu oluşturabilir. Slot diğer müşterilere dolu görünür.
- **KK-2:** Çakışan bir saate randevu girilmek istendiğinde sistem engeller (FR-11).

**US-10: Yeniden planlama** (FR-21)
Müşteri olarak randevumu iptal edip yeniden almak yerine başka bir saate taşımak istiyorum.
- **KK-1:** Yeni saat başarıyla alınmadan eski randevu iptal edilmez.
- **KK-2:** Taşıma sonrası çalışana değişiklik bildirimi gider.

**US-11: Randevu kuralları ve no-show** (FR-22, FR-23)
Yönetici olarak son dakika randevularını ve gelmeyen müşterileri yönetmek istiyorum.
- **KK-1:** En kısa önceden randevu süresi 2 saat ayarlandığında, şu andan itibaren 2 saatten yakın slotlar müşteriye gösterilmez.
- **KK-2:** "Gelmedi" olarak işaretlenen randevu, müşterinin gelmeme sayısına eklenir ve yönetici bu sayıyı müşteri detayında görür.

## 7. Kapsam dışı maddeler

Bu sürümde yapılmayacak maddeler (MoSCoW: W):
- **Çevrimiçi ödeme, kapora, ön ödeme ve iptal ücreti** (Fresha, Booksy ve Vagaro'da var). Görevde istenmedi. Ödeme altyapısı, mağaza ödeme kuralları ve ek mevzuat getirir.
- **Pazar yeri ve keşif:** müşterinin şehir veya kategoriye göre yeni salon araması (Fresha pazar yeri modeli).
- **Pazarlama ve kampanya bildirimleri, toplu SMS.** Apple 4.5.4'e göre açık onay ve vazgeçme yolu, Türkiye'de İYS süreçleri gerektirir.
- **Muhasebe, adisyon, stok, personel prim hesabı** (Salon Randevu'da var).
- **Müşteri değerlendirme ve puanlama, sadakat ve puan sistemi.**
- **Çok şubeli işletme yönetimi ve şubeler arası raporlama.**
- **Web üzerinden randevu sayfası ve sosyal medya entegrasyonları** (Vagaro'da var). Görev yalnızca Android ve iOS uygulamasını kapsıyor. Hesap silme için gereken web bağlantısı (FR-03) bu maddenin istisnasıdır.
- **Tablet ve masaüstü yönetim paneli.**
- **Gelişmiş raporlama ve analitik** (doluluk oranı, gelir raporları).
- **Yapay zekâ ile telefon karşılama veya asistan** (Fresha'da ücretli eklenti).

## 8. Varsayımlar

- **V-01:** Ürün birden çok işletmenin kullanabileceği bir uygulama olarak düşünülmüştür. Müşteri, randevu alacağı işletmeye işletmenin paylaştığı bağlantı veya kod ile ulaşır; pazar yeri araması yoktur. Tek işletmeye özel uygulama da olabilir (bkz. Açık sorular).
- **V-02:** Flutter ve Firebase ekip tarafından verilen kısıtlardır. Bu doküman bunlarla ilgili mimari karar vermez. Firebase'in hangi servislerinin kullanılacağı TA'nın kararıdır.
- **V-03:** Görevdeki "bildirim" ifadesi uygulama içi ve push bildirimi olarak yorumlanmıştır. SMS ve e-posta yedek kanaldır (FR-27, C).
- **V-04:** Varsayılan değerler (24 saatlik hatırlatma, 2 saatlik iptal süresi, 60 günlük ileri tarih) rakip pratiklerinden esinlenmiş BA önerileridir. İşletme tarafından ayarlanabilir olmaları önerilir; kesin değerler doğrulanmadı.
- **V-05:** Hatırlatmanın no-show'u azaltacağı varsayımı sağlık kliniklerindeki SMS meta-analizine dayanır (OR 1,48, 8 RKÇ). Salon sektörü ve push kanalı için aktarılabilirliği doğrulanmamıştır.
- **V-06:** Randevu hatırlatmalarının Ticari Elektronik İletiler Yönetmeliği md. 6/2 kapsamında önceden onay gerektirmediği BA yorumudur. Hukuki teyit gerekir.
- **V-07:** Randevu verisinin KVKK md. 5/2-c (sözleşmenin ifası) şartına dayanarak açık rıza olmadan işlenebileceği BA yorumudur. Hukuki teyit gerekir.
- **V-08:** Hizmet notlarına sağlık bilgisi girilmeyecektir (ör. alerji, cilt hastalığı). Bu tür bilgi KVKK md. 6'ya göre özel nitelikli kişisel veridir ve açık rıza gerektirir.
- **V-09:** iOS'ta bildirim izninin Android'e benzer şekilde kullanıcı onayı gerektirdiği varsayılmıştır. Apple'ın ilgili dokümantasyon sayfası bu çalışmada okunamadığı için doğrulanmadı.
- **V-10:** WCAG 2.2 web içeriği için yazılmıştır. Mobil uygulamada eşdeğer hedef olarak kullanılması bir varsayımdır.
- **V-11:** Platformun yerleşik çözümü olarak Google Takvim randevu planları incelendi. iOS ve Android işletim sistemlerinde salon randevusuna özgü yerleşik bir çözüm bulunamadı; bu konuda ayrıca kaynak aranmadı.
- **V-12:** Çocuk kullanıcılara yönelik özel düzenlemeler değerlendirilmedi. Ürünün hedef kitlesi yetişkin müşteriler ve işletme çalışanları olarak varsayıldı.

## 9. Açık sorular

- **AS-01:** Uygulama tek bir salona mı ait olacak, yoksa birçok salonun kullandığı bir platform mu? Çok işletmeli ise işletme kaydı ve onayı nasıl olacak?
- **AS-02:** Müşteri kaydında birincil kimlik e-posta mı, telefon numarası mı? Telefon doğrulaması SMS maliyeti getirir. Sosyal giriş (Google, Apple) isteniyor mu? Apple 4.8 ek gereksinim doğurabilir.
- **AS-03:** İptal süresi dolduktan sonra müşteri iptali tamamen engellenmeli mi, yoksa iptale izin verilip işletmeye "geç iptal" olarak mı işaretlenmeli?
- **AS-04:** "Fark etmez" seçildiğinde çalışan atama kuralı ne olmalı: en az randevulu çalışan mı, sıra ile mi, ilk müsait mi?
- **AS-05:** Çalışan kendi çalışma saatlerini ve izinlerini kendisi düzenleyebilmeli mi, yoksa yalnızca yönetici mi?
- **AS-06:** SMS yedek hatırlatma (FR-27) isteniyor mu? İsteniyorsa SMS maliyetini kim üstlenecek ve aylık bütçe nedir?
- **AS-07:** Desteklenen en düşük Android ve iOS sürümleri nelerdir?
- **AS-08:** Kişisel verilerin barındırılacağı konum Türkiye dışında olacaksa KVKK md. 9 (yurt dışına aktarım) kapsamında nasıl bir hukuki dayanak kullanılacak? İncelenen kanun metni 2016 kabul metnidir. Güncel md. 9 hükmü bu çalışmada doğrulanmadı ve hukuk danışmanıyla teyit edilmelidir.
- **AS-09:** Veri sorumlusu kim olacak: her salon mu, uygulama sağlayıcısı mı, ikisi birlikte mi? Bu, aydınlatma metninin içeriğini ve Veri Sorumluları Sicili yükümlülüğünü (KVKK md. 16) etkiler.
- **AS-10:** Erişilebilirlik süresi hedefi (NFR-12, %99,5) ve yanıt süresi hedefleri (NFR-03) işletme beklentisine uygun mu?
- **AS-11:** Bir hizmet birden çok çalışan veya kaynak gerektiriyor mu (ör. boya + fön için iki çalışan, belirli bir koltuk veya oda)? İlk sürümde tek çalışan ve tek hizmet varsayılmıştır.
- **AS-12:** Müşteri bir randevuda birden çok hizmet seçebilmeli mi (ör. kesim + fön)?

## 10. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K-1 | Fresha for business | https://www.fresha.com/for-business | Rakip özellikleri: "get your clients self-booking online 24/7"; "automated booking reminders via email and text"; "implement a flexible cancellation policy"; kapora ve ön ödeme seçenekleri |
| K-2 | Fresha Pricing | https://www.fresha.com/pricing | Tek kişi için 19,95 $/ay; ekip planında takvimde randevu alınabilen kişi başına 14,95 $/ay; ayda 100 ücretsiz otomatik SMS/WhatsApp, sonrası mesaj başına 0,02 $; pazar yerinden gelen yeni müşteride tek seferlik %20 komisyon (en az 6 $) |
| K-3 | Booksy Features | https://biz.booksy.com/en-us/features | "Clients can view your availability and conveniently book appointments 24/7"; "Unlimited automatic client appointment reminders are included"; çalışan profili ve takvim; "Access staff calendars"; 29,99 $/ay, "+$20 per add'l user" |
| K-4 | Booksy No-Show Protection | https://biz.booksy.com/en-us/features/no-show-protection | İptal penceresi: "how much advance notice is required for no-fee cancellations or rescheduling"; kayıtlı kartla iptal ücreti; kapora; güvenilir müşteri muafiyeti |
| K-5 | Vagaro Salon Software | https://www.vagaro.com/pro/salon-software | Çok kanallı randevu; "automatic reminders so clients never miss an appointment"; "cancellation policies with no-show fees or required deposits"; "Automated waitlists that fill gaps for you" |
| K-6 | Google Calendar Help: Appointment schedules | https://support.google.com/calendar/answer/10729749 | Platform ekosistemindeki çözüm: rezervasyon penceresi, tampon süre ("Add time between appointments on your calendar"), günlük en fazla randevu, en fazla 5 hatırlatma e-postası; bazı özellikler ücretli abonelik gerektiriyor |
| K-7 | Salon Randevu İşletme Yönetimi (App Store, ABD) | https://apps.apple.com/us/app/id1413695973 | Türkiye rakibi: "Müşterileriniz online randevu alsın"; "Randevu hatırlatma SMS mesajları"; personel primi; "Ücretsiz kullanım, aylık ödeme yok" (geliştiricinin ifadesi); uygulama içi ücretli paketler; 1 değerlendirme |
| K-8 | Kuaförüm Yanımda (App Store, TR) | https://apps.apple.com/tr/app/kuaf%C3%B6r%C3%BCm-yan%C4%B1mda/id1516248277?l=tr | Türkiye rakibi: "Otomatik Onay: Herhangi bir onaylamaya gerek kalmadan..."; izin günleri ve yemek saatleriyle çakışma önleme; bildirim merkezi; müşteri için hesap silme seçeneği; 4,6 puan, 75 değerlendirme |
| K-9 | App Review Guidelines (Apple) | https://developer.apple.com/app-store/review/guidelines/ | 5.1.1(v): "If your app supports account creation, you must also offer account deletion within the app"; 4.5.4: "Push Notifications must not be required for the app to function... should not be used for promotions or direct marketing purposes unless customers have explicitly opted in"; 4.8: üçüncü taraf girişte eşdeğer giriş seçeneği şartı |
| K-10 | Google Play: Understanding account deletion requirements | https://support.google.com/googleplay/android-developer/answer/13327111 | "provide users with an in-app path to delete their app accounts and associated data"; "provide a web link resource where users can request app account deletion"; 31 Mayıs 2024 sonrası uygulama yaptırımı |
| K-11 | Google Play: Data safety section | https://support.google.com/googleplay/android-developer/answer/10787469 | Tüm uygulamalar formu doldurur, üçüncü taraf kütüphane ve SDK verileri dahil beyan edilir; gizlilik politikası bağlantısı zorunlu |
| K-12 | Android Developers: Notification runtime permission | https://developer.android.com/develop/ui/views/notifications/notification-permission | Android 13 (API 33) `POST_NOTIFICATIONS`; yeni kurulumda "notifications are off by default"; izni işlevle ilgili bağlamda istemek ("Request the permission in context") |
| K-13 | Ticari İletişim ve Ticari Elektronik İletiler Hakkında Yönetmelik (Lexpera) | https://www.lexpera.com.tr/mevzuat/yonetmelikler/ticari-iletisim-ve-ticari-elektronik-iletiler-hakkinda-yonetmelik | md. 6/2: "Devam eden abonelik, üyelik veya ortaklık durumu ile ... satın alma ve teslimat veya benzeri durumlara ilişkin bildirimleri içeren iletiler" için önceden onay aranmaz; "Ancak bu tür bildirimlerde herhangi bir mal veya hizmet özendirilemez veya bunların tanıtımı yapılamaz."; md. 6/5: İYS kontrolü yapılmaz |
| K-14 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (Adalet Bakanlığı, 2016 kabul metni PDF) | https://mgm.adalet.gov.tr/Resimler/SayfaDokuman/51120191031036698%20KVKK.pdf | md. 5/2-c: sözleşmenin ifasıyla "doğrudan doğruya ilgili olması kaydıyla" açık rıza aranmaz; md. 6 özel nitelikli veri (sağlık dahil); md. 7 silme; md. 9 yurt dışına aktarım; md. 10 aydınlatma; md. 11 haklar; md. 13/2: "en geç otuz gün içinde ücretsiz olarak sonuçlandırır"; md. 16 sicil |
| K-15 | WCAG 2.2 (W3C) | https://www.w3.org/TR/WCAG22/ | 1.4.3: "contrast ratio of at least 4.5:1"; 1.4.4: "resized without assistive technology up to 200 percent" |
| K-16 | Understanding SC 2.5.8 Target Size (Minimum) (W3C) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | "The size of the target for pointer inputs is at least 24 by 24 CSS pixels" (istisnalarla) |
| K-17 | How Effective Are Short Message Service Reminders at Increasing Clinic Attendance? A Meta-Analysis and Systematic Review (Health Services Research, 2012) | https://pmc.ncbi.nlm.nih.gov/articles/PMC3419880/ | 18 çalışma (8 RKÇ, 10 gözlemsel). Birinci basamak ve hastane polikliniklerinde yalnızca 8 RKÇ birleştirildi; OR 1,48. Yazarların sonucu: SMS hatırlatma "substantially increase the likelihood of attending clinic appointments". Özette GA 1,23-1,72, sonuçlar bölümünde 1,33-1,72 olarak geçiyor |

**Açılamayan veya kullanılmayan kaynaklar (kaynak gösterilmedi):**
- Square Appointments özellik sayfası (HTTP 429).
- KVKK Kurumu sayfaları kvkk.gov.tr (HTTP 503).
- mevzuat.gov.tr kanun metni (403 veya metin yok).
- Booksy destek makalesi "How to set up my policies" (403).
- BMJ Open 2016 meta-analizi (403) ve PubMed özeti (çerez sayfası).
- Apple "Asking permission to use notifications" sayfası (içerik okunamadı).
- Arama sonuçlarında görülen diğer Türkiye uygulamaları (Randevu345 vb.) açılmadığı için kaynak gösterilmedi.
