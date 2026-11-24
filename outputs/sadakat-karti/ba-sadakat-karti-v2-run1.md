# Küçük Kafeler İçin Dijital Sadakat Kartı Uygulaması: İş Analizi Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ba v2
- **Girdi:** "Küçük kafeler için cep telefonundan kullanılan bir dijital sadakat kartı uygulaması yapacağız (10 kahvede 1 bedava gibi). Mevcut çözümleri araştırarak iş gereksinimlerini belirle ve bir analiz dokümanı hazırla."

---

## 1. Özet

Ürün, küçük ve bağımsız kafelerin kâğıt damga kartının (ör. "10 kahveye 1 bedava") yerine telefondan kullanılan dijital bir kart sunmasını sağlar. Kullanıcı tarafı iki gruptan oluşur:
- **Müşteri:** Kartını telefonda taşır ve ilerlemesini görür.
- **Kafe:** Sahibi ve personeli damga verir, ödülü kullandırır ve basit sonuçları izler.

Araştırmada incelenen çözümler:
- uluslararası ürünler: Loopy Loyalty, Stamp Me, Square Loyalty,
- Türkiye'de yayımlanan uygulamalar: Yine Bekleriz, Kupa,
- platformların yerleşik çözümleri: Apple Wallet ve Google Wallet sadakat kartları.

Bu çözümlerde öne çıkan ortak beklentiler şunlardır:
- müşterinin hızlı katılması (QR veya bağlantıyla, çoğu zaman uygulama indirmeden),
- personelin özel donanım olmadan telefonla damga vermesi,
- sıranın yavaşlamaması.

Ürünün Türkiye'de sunulacağı varsayılmıştır. Bu durumda uygulanacak düzenlemeler:
- **KVKK (6698):** aydınlatma, silme ve veri güvenliği.
- **6563 sayılı Kanun:** ticari elektronik ileti için önceden onay ve ret hakkı.
- **Mağaza politikaları:** hesap silme, veri güvenliği beyanı, şablon uygulama kuralı.

Ürünün iş modelini belirleyen en önemli açık nokta şudur: Ürün, birçok kafeyi barındıran tek bir uygulama mı olacak, yoksa her kafeye özel markalı uygulama mı? Apple'ın 4.2.6 kuralı ikinci seçeneği kısıtlar (bkz. Bölüm 3.3 ve Açık sorular S-01).

İlk sürüm (MVP) şu döngüyü kapsar:
1. Kafe kuralını tanımlar.
2. Müşteri karta katılır.
3. Personel damga verir.
4. Ödül kullandırılır.
5. İşlem geçmişi tutulur.

Bu döngüye yasal zorunluluklar (aydınlatma, hesap silme) eklenir.

---

## 2. Hedef kullanıcılar ve persona'lar

| Persona | Rol | İhtiyaç / hedef | Sıkıntı noktası |
|---|---|---|---|
| **P-1 Kafe sahibi** (tek şubeli, 1–5 çalışanlı kafe) | Programı kurar, personeli yetkilendirir, sonuçları izler | Müdavim sayısını artırmak; kurulumun dakikalar içinde bitmesi; ek donanım ve yüksek aylık maliyet olmaması | Kâğıt kartların kaybolması ve sahte damga; müşteriyi tanıyamamak |
| **P-2 Barista / personel** | Kasada damga verir ve ödül kullandırır | İşlemin birkaç saniyede bitmesi, sıranın yavaşlamaması | Yoğun saatte karmaşık ekranlar; yanlış damga verip geri alamamak |
| **P-3 Müdavim müşteri** | Haftada birkaç kez gelen müşteri | Kartını kaybetmemek, ödüle ne kadar kaldığını görmek | Kâğıt kartı unutmak; her kafe için ayrı uygulama indirmek |
| **P-4 Ara sıra gelen / yeni müşteri** | İlk kez gelen müşteri | Kişisel veri vermeden, hızlıca katılmak | Uzun kayıt formları; istenmeyen pazarlama mesajları |

Hedef kitlenin yetişkin olduğu varsayılmıştır (bkz. Varsayımlar V-05).

---

## 3. Mevcut çözümler / rakip incelemesi

### 3.1 Doğrudan rakipler ve mevcut çözümler

| Çözüm | Müşteri tarafı | Damga verme yöntemi | Öne çıkan özellikler | Güçlü yön | Zayıf yön / risk |
|---|---|---|---|---|---|
| **Loopy Loyalty** [K1][K2] | Uygulama indirmeden QR veya bağlantıyla katılım; kart Apple/Google Wallet'a eklenir. Telefon değişince kart damgalarıyla birlikte geri yüklenebilir | Personel "Stamper" uygulaması; üreticiye göre "Any phone or tablet can be used" | Mesajlaşma ("Smart Messaging"), gerçek zamanlı analitik. Üretici "No POS integration, no special hardware" diyor | Uygulama indirme engeli yok; donanım gerekmiyor | Ücretli. Fiyat sayfasına göre Starter planı $25/ay: 1 kart tasarımı, 1 lokasyon, en fazla 3 Stamper kullanıcısı. Kafe sayfasında kötüye kullanım önlemleri anlatılmıyor; güvenlik sayfası açılamadı (404) |
| **Stamp Me** [K3] | Web uygulaması veya yerel uygulama; üreticiye göre uygulama indirmek zorunlu değil. Üye kodu Wallet'a eklenebilir | Birden çok yöntem: tezgâh cihazı (StampPod, "tap & go"), personelin taşıdığı etiket (Stamp Tag), tek kullanımlık kodlar (OneStamps), personelin üye kodunu taraması (Stamp Business), web portalından elle damga | Push/SMS, doğum günü ödülü, oyunlaştırma, CRM entegrasyonu. Üreticiye göre "50+ countries"; SOC 2 Type 2 rozeti | Yöntem çeşitliliği. Tek kullanımlık kodlar, üretici ifadesiyle "issue one stamp then expire" | Özellik kapsamı küçük bir kafe için geniş. Fiyat ana sayfada yok. Tanıtım iddiaları üreticiye ait |
| **Square Loyalty** [K4] | Kasada telefon numarasıyla katılım. Bir işletme sahibinin sayfadaki ifadesi: "type in their phone number and get points automatically" | Square'in satış noktası (POS), fatura ve online mağaza ürünleriyle yapılan alışverişlerde puan. Sayfaya göre puan "even when they use cash" kazanılır | Ziyaret, tutar veya ürün bazlı puan; VIP seviyeleri; Apple Wallet kartı; otomatik bilgilendirme SMS'leri | Satışla bütünleşik, personel ayrıca damga vermiyor | Sayfada Square POS'un zorunlu olduğu açıkça yazmıyor. Ancak listelenen tüm puan kanalları Square ürünleri üzerinden işliyor. Bu nedenle Square kullanmayan kafeye uygun değil (kaynaktaki bilgiden çıkarıldı) |
| **Yine Bekleriz** (Türkiye) [K5] | Ücretsiz müşteri uygulaması; birçok işletmenin kartı tek uygulamada | Müşteri kartın QR kodunu gösterir veya telefonunu NFC etiketine yaklaştırır ("QR kodunu göster ya da telefonunu NFC etiketine yaklaştır") | Ödüle kalan damgayı gösterme, ödül talebi, tüm kartları birlikte izleme | Türkçe; çok işletmeli tek uygulama modeli. Mağaza sayfasında üretici "4.000'den fazla işletme" diyor | Mağaza sayfasında yalnızca 1 değerlendirme var. İşletme sayısı üreticinin iddiası, doğrulanmadı |
| **Kupa** (Türkiye) [K6] | Ücretsiz müşteri uygulaması; kafe keşfi ve sadakat bir arada | Mağaza sayfası damga yöntemini açıklamıyor ("Kahve içtikçe puan topla"); sürüm notlarında NFC anılıyor | Yakındaki kafeleri keşif, hediye kahve, kampanya bildirimleri | Keşif özelliği yeni müşteri getirebilir | Kafe tarafı araçları mağaza sayfasında anlatılmıyor. Mağaza sayfasında 32 değerlendirmede ortalama 4,4 |
| **Platformun yerleşik çözümü: Apple Wallet** [K7][K8] | Wallet'taki "store card" türü pass; uygulama, web, SMS, e-posta, NFC etiketi ve QR ile dağıtılabilir | QR veya barkod; NFC ile otomatik tanınma ise Apple'ın sertifikalı terminal ve POS koşullarına bağlı | Apple'a göre pass güncellenebilir ("real-time updates based on customer activity right on the pass") ve bildirim gönderilebilir | Müşteri uygulama indirmeden kartı taşır | Pass'ın kendisi damga verme veya kafe yönetimi sağlamaz; bunun için ayrı bir sistem gerekir. NFC için "must already accept Apple Pay and be certified for VAS" koşulu var |
| **Platformun yerleşik çözümü: Google Wallet** [K9] | Sadakat kartı pass'ı. Google'a göre pass'lar uygulamada ve "anywhere hyperlinks work" (web, e-posta, SMS) dağıtılabilir | Sayfa barkod/NFC ayrıntısı vermiyor | Puan ve ödülü telefonda gösterme ("immediate visibility to points and rewards") | Android'de uygulama indirmeden kart | Kart vermek için Google Wallet API Issuer hesabı gerekir. Damga verme ve yönetim yine ayrı bir sistemin işidir |

### 3.2 Ortak örüntüler ve çıkarımlar (BA değerlendirmesi)
- **Uygulama indirmeden katılım** üç çözümde ön planda: Loopy, Stamp Me ve Wallet pass'ları [K1][K3][K7][K9]. Bu, P-4 için giriş engelini düşürür. Ürünün müşteriyi uygulama indirmeye zorlayıp zorlamayacağı bir iş kararıdır (bkz. S-03).
- **Ek donanım gerektirmemek** küçük işletmeye yönelik ürünlerde satış argümanı olarak öne çıkıyor [K1]. NFC ile Wallet'ta otomatik tanınma ise sertifikalı terminal ister [K8]. Bu nedenle küçük kafe için ilk sürümde uygun değildir (Kapsam dışı KD-05).
- **Damga yetkisi** personelde: Personel ya müşterinin kodunu tarar [K3][K5] ya da müşteri kafeye ait bir kodu veya etiketi okutur [K3][K5]. Kötüye kullanım önlemleri incelenen sayfalarda ayrıntılandırılmamış. Tek açık örnek, Stamp Me'nin süresi dolan tek kullanımlık kodlarıdır [K3].
- **Çok işletmeli tek uygulama** modeli Türkiye'deki iki örnekte de var [K5][K6]. Bu model Apple 4.2.6'nın "picker" seçeneğiyle de uyumludur [K10].

### 3.3 Alan standartları ve düzenlemeler

| Düzenleme / standart | İlgili hüküm | Ürüne iş etkisi |
|---|---|---|
| **6698 sayılı KVKK** [K11] | Md. 4: verilerin "İşlendikleri amaçla bağlantılı, sınırlı ve ölçülü" olması. Md. 5: açık rıza ve istisnalar, ör. "Bir sözleşmenin kurulması veya ifasıyla doğrudan doğruya ilgili olması". Md. 7: silme. Md. 10: aydınlatma yükümlülüğü (kimlik, amaç, aktarım, yöntem ve hukuki sebep, haklar). Md. 12: güvenlik tedbirleri ve ihlal halinde "en kısa sürede" bildirim. Md. 13: başvurunun "en geç otuz gün içinde" sonuçlandırılması | Toplanan veri en aza indirilmeli. Katılımda aydınlatma metni sunulmalı. Silme ve başvuru süreci tanımlanmalı (FR-14, NFR-04) |
| **6563 sayılı Kanun** [K12] | Md. 6: ticari elektronik iletiler "ancak önceden onayları alınmak kaydıyla gönderilebilir". Aynı fıkraya göre, iletişim bilgisini kendisi veren alıcıya mal veya hizmete ilişkin "değişiklik, kullanım ve bakıma yönelik" iletiler için ayrıca onay alınmaz. Md. 8: alıcı "hiçbir gerekçe belirtmeksizin" reddedebilir; gönderim talepten itibaren "üç iş günü içinde" durdurulur | Kampanya mesajları ayrı onaya bağlanmalı ve kolay ret sunulmalı (FR-16). Push bildiriminin bu kanun kapsamındaki "ticari elektronik ileti" sayılıp sayılmadığı netleştirilmeli (S-07) |
| **WCAG 2.2** [K13] | 1.4.3 (AA): metin kontrastı "at least 4.5:1", büyük metinde 3:1. 2.5.8 (AA): dokunma hedefi "at least 24 by 24 CSS pixels" (istisnalar var) | Müşteri ve personel ekranları için ölçülebilir erişilebilirlik hedefi (NFR-03) |

### 3.4 Platform ve mağaza politikaları (iş etkisi düzeyinde)
Görevde yalnızca "cep telefonu" belirtilmiş, platform belirtilmemiştir. Olası platformlar iOS ve Android'dir.

| Politika | Hüküm | İş etkisi |
|---|---|---|
| **App Store 4.2.6** [K10] | "Apps created from a commercialized template or app generation service will be rejected unless submitted directly by the provider of the app's content". Kabul edilen alternatif: "a single binary to host all client content in an aggregated or 'picker' model" | Her kafeye ayrı markalı uygulama yayımlamak, ürün sahibi tarafından yapılırsa reddedilme riski taşır. Çok kafeli tek uygulama modeli bu riski önler (S-01) |
| **App Store 4.2** [K10] | Uygulama "elevate it beyond a repackaged website" olmalı | Uygulama mağazası yolu seçilirse yalnızca web sayfasını saran bir uygulama reddedilebilir |
| **App Store 5.1.1(v)** [K10] | Önemli hesap özelliği yoksa giriş zorunlu tutulmamalı. Hesap oluşturuluyorsa "you must also offer account deletion within the app". Kişisel veri yalnızca çekirdek işlevle doğrudan ilgiliyse istenebilir | Katılımda zorunlu kişisel veri en aza indirilmeli (NFR-02). Hesap varsa uygulama içi silme gerekir (FR-14) |
| **Google Play hesap silme** [K14] | Hesap oluşturan uygulamalar "an in-app path to delete their app accounts and associated data" ve ayrıca "a web link resource" sunmalı | Uygulama içi silme ile birlikte web üzerinden silme talebi de gerekir (FR-14) |
| **Google Play Data safety** [K15] | Yayımlanan tüm uygulamalar formu doldurur; veri toplama, paylaşım, aktarımda şifreleme ve silme beyan edilir. Beyanın doğruluğundan geliştirici sorumludur | Toplanan veri listesi baştan net olmalı; ürün davranışı beyanla uyumlu tutulmalı (NFR-04, NFR-05) |
| **Apple / Google Wallet** [K8][K9] | Apple'da NFC ile otomatik tanınma için sertifika, VAS uyumlu terminal ve POS gerekir. Google Wallet'ta kart vermek için Issuer hesabı gerekir | Wallet desteği (FR-18) ek hesap ve süreç gerektirir; NFC kapsam dışıdır (KD-05) |

### 3.5 Yöntemler: damga verme ve kart taşıma seçenekleri (iş açısından)
Yöntem seçimi TA'ya bırakılmıştır. Aşağıdaki artı/eksi değerlendirmeleri, kaynakla işaretlenmedikçe BA'nın iş değerlendirmesidir.

| Yöntem | Örnek | İş açısından artı | İş açısından eksi |
|---|---|---|---|
| **Y-1** Müşteri kodunu gösterir, personel telefonla tarar | Stamp Me Stamp Business [K3], Yine Bekleriz QR [K5] | Damga kararı personelde; kötüye kullanım riski düşük; ek donanım yok | Personelin elinde yetkili bir cihaz olmalı; tarama kasada birkaç saniye ekler |
| **Y-2** Kafedeki sabit QR'ı müşteri okutur | (İncelenen sayfalarda açıkça bu biçimde anlatılmadı) | Personel iş yükü en az | Sabit kod fotoğraflanıp tekrar kullanılabilir. Ek kontrol olmadan sahte damga riski yüksek (BA değerlendirmesi) |
| **Y-3** Tek kullanımlık veya süreli kod | Stamp Me OneStamps [K3] | Kod tekrar kullanılamaz | Kod üretme ve dağıtma adımı ekler |
| **Y-4** NFC etiketi veya tezgâh cihazı | Yine Bekleriz [K5], StampPod [K3] | Hızlı "dokun ve geç" deneyimi | Etiket veya cihaz maliyeti. Wallet ile NFC sertifikalı terminal ister [K8] |
| **Y-5** Telefon numarasıyla kasada eşleştirme | Square [K4] | Müşteri tarafında uygulama gerekmez | Telefon numarası kişisel veridir, KVKK yükü artar [K11]. POS entegrasyonu olmadan personel elle girer |
| **Y-6** Yönetim panelinden elle damga | Stamp Me Loyalty Portal [K3] | Hata düzeltme ve istisna durumları için yedek | Kötüye kullanıma açık; kayıt ve yetki gerekir |

Kartın taşınma biçimi için seçenekler şunlardır. Seçim yine TA'ya aittir; iş tercihi S-03'te sorulmuştur.
- **Yerel uygulama:** Bildirim ve marka deneyimi sağlar, ancak indirme engeli vardır.
- **Wallet pass'ı:** İndirme gerektirmez [K7][K9], ancak yönetim için ayrı bir sistem ister.
- **Web:** İndirme gerektirmez, ancak App Store 4.2 açısından yalnızca web sarmalayıcı uygulama riski vardır [K10].

**Başlangıç damgası etkisi (endowed progress):** Wharton'daki habere göre Nunes ve Drèze, bir oto yıkamacının 300 müşterisiyle deney yaptı:
- standart kart: 8 alışveriş,
- deney kartı: 10 alışveriş, 2 damga baştan verilmiş.

Yazarların sonucuna göre bu çerçeveleme "increases the likelihood of task completion and decreases completion time" [K16]. Habere göre her iki kartta da gereken gerçek alışveriş sayısı sekizdir. Haber kesin oranları vermiyor. Orijinal makale açılamadığı için (HTTP 503) oranlar doğrulanamadı. Bu bulgu yalnızca isteğe bağlı bir özelliğe (FR-19) dayanak yapılmıştır.

---

## 4. Fonksiyonel gereksinimler

**M (Must) önceliğindeki gereksinimler ilk sürümün (MVP) sınırını tanımlar.** S ve C maddeleri ilk sürüme kaynak kalırsa alınır. Bu sürümde yapılmayacak maddeler Bölüm 7'dedir.

| ID | Gereksinim | Öncelik | Dayanak |
|---|---|---|---|
| FR-01 | Kafe sahibi kafesini kaydedebilmeli ve kafe adını, logosunu ve kısa açıklamasını girebilmelidir. | M | Görev; [K1][K3] |
| FR-02 | Kafe sahibi sadakat kuralını tanımlayabilmelidir: ödül için gereken damga sayısı (1–20 arası, varsayılan 10) ve ödülün metni (ör. "1 bedava kahve"). Ödül 1 adet ile sınırlıdır. Kural değişikliği mevcut kartlara nasıl uygulanacak, kafe sahibine kaydetmeden önce açıkça gösterilmelidir. | M | Görev ("10 kahvede 1 bedava") |
| FR-03 | Müşteri, kafede gördüğü bir QR kodunu veya bağlantıyı açarak o kafenin kartına katılabilmelidir. Katılım için zorunlu kişisel veri istenmez ya da yalnızca kartın kurtarılması için gereken en az veri istenir (bkz. S-04). | M | [K1][K3][K10 5.1.1(v)][K11 Md. 4] |
| FR-04 | Müşteri kartında kafe adını, mevcut damga sayısını, gereken toplam damgayı (ör. "7/10") ve ödülün hazır olup olmadığını görebilmelidir. | M | Görev; [K5][K9] |
| FR-05 | Yetkili personel, müşterinin kartını tanımlayarak karta damga ekleyebilmelidir. Tek işlemde 1'den fazla damga eklenebilir (ör. 3 kahve). Tek işlemdeki üst sınırı kafe sahibi belirler. | M | [K1][K3][K5] |
| FR-06 | Kart dolduğunda yetkili personel ödülü kullandırabilmelidir. Kullandırmadan sonra kartın damga sayısı sıfırlanır. Fazla damga varsa yeni karta aktarılır. | M | Görev |
| FR-07 | Kafe sahibi personel hesabı veya yetkisi ekleyip kaldırabilmelidir. Yalnızca yetkili personel damga verebilir ve ödül kullandırabilir. | M | [K2: planlarda Stamper kullanıcı sayısı][K11 Md. 12] |
| FR-08 | Her damga, ödül ve düzeltme işlemi kaydedilmelidir: tarih-saat, kafe, kart, işlemi yapan personel, adet. Kafe sahibi bu kaydı görebilmeli, müşteri ise kendi kartının geçmişini görebilmelidir. | M | Anlaşmazlık çözümü ve kötüye kullanım kontrolü (BA değerlendirmesi) |
| FR-09 | Kafe sahibi yanlış verilmiş bir damgayı veya ödül kullandırmasını gerekçe yazarak geri alabilmelidir. Geri alma da FR-08 kaydına işlenir. | S | [K3: elle müdahale portalı]; BA değerlendirmesi |
| FR-10 | Aynı karta, kafe sahibinin belirlediği süre içinde (varsayılan 5 dakika) ikinci bir damga verilmek istendiğinde personele uyarı gösterilmeli ve onay istenmelidir. | S | Kötüye kullanım önleme (BA değerlendirmesi; incelenen kaynaklarda ayrıntı yok) |
| FR-11 | Müşteri telefonunu değiştirdiğinde veya uygulamayı yeniden kurduğunda kartlarını damgalarıyla birlikte geri alabilmelidir. | S | [K1: "restored with its collected stamps and rewards"] |
| FR-12 | Müşteri birden çok kafenin kartını tek yerde görebilmelidir. (Ürün modeli S-01'de "çok kafeli tek uygulama" olarak onaylanırsa geçerlidir.) | S | [K5][K10 4.2.6] |
| FR-13 | Kafe sahibi seçilen dönem (gün, hafta, ay) için şu temel göstergeleri görebilmelidir: verilen damga sayısı, kullandırılan ödül sayısı, kartı olan müşteri sayısı ve dönemde en az 1 damga alan müşteri sayısı. | S | [K1][K3: analitik] |
| FR-14 | Müşteri hesabı varsa müşteri, hesabını ve ilişkili verileri uygulama içinden silebilmeli ve aynı talebi uygulama dışında bir web bağlantısından da iletebilmelidir. Kafe sahibi de kafe hesabını ve verilerini silebilmelidir. | M | [K10 5.1.1(v)][K14][K11 Md. 7] |
| FR-15 | Müşteriye katılım anında ve sonrasında erişilebilir biçimde KVKK aydınlatma metni sunulmalıdır. Metin şunları içerir: veri sorumlusu, amaç, aktarım, yöntem ve hukuki sebep, haklar. | M | [K11 Md. 10] |
| FR-16 | Kafe kampanya ve tanıtım mesajı gönderecekse müşteriden bu mesajlar için ayrı ve isteğe bağlı onay alınmalıdır. Müşteri onayını istediği zaman kolayca geri alabilmelidir. Onay olmadan kampanya mesajı gönderilemez. | C | [K12 Md. 6, Md. 8][K1][K3] |
| FR-17 | Kart dolduğunda müşteriye "ödülünüz hazır" bilgisi gösterilmelidir. Uygulama içinde gösterim zorunludur; bildirim olarak gönderilmesi isteğe bağlıdır. | C | [K4: otomatik bilgilendirme mesajları][K8] |
| FR-18 | Müşteri kartını telefonunun cüzdan uygulamasına (Apple Wallet / Google Wallet) ekleyebilmelidir. Kartta damga sayısı güncel görünmelidir. | C | [K1][K7][K8][K9] |
| FR-19 | Kafe sahibi yeni katılan müşteriye gerekçesiyle birlikte (ör. "hoş geldin damgası") başlangıç damgası tanımlayabilmelidir (0 ile damga sayısı−1 arası, varsayılan 0). | C | [K16] (oranlar doğrulanamadı) |

---

## 5. Fonksiyonel olmayan gereksinimler

| ID | Gereksinim | Ölçüt / doğrulama | Dayanak |
|---|---|---|---|
| NFR-01 | **İşlem hızı:** Damga verme sıranın akışını yavaşlatmamalıdır. | Personel ekranı açıkken, kartın tanımlanmasından damganın onaylanmasına kadar geçen süre normal bağlantıda işlemlerin %95'inde ≤ 3 saniye olmalıdır. | [K1: "no slowing down the queue"] |
| NFR-02 | **Katılım kolaylığı:** Yeni müşteri karta hızlı katılabilmelidir. | QR'ı okutmaktan kartı görmeye kadar ≤ 60 saniye ve ≤ 3 ekran. Zorunlu kişisel veri alanı sayısı ≤ 1 olmalıdır (S-04'e göre kesinleşir). | [K1][K10 5.1.1(v)][K11 Md. 4] |
| NFR-03 | **Erişilebilirlik:** Müşteri ve personel ekranları WCAG 2.2 AA düzeyini hedeflemelidir. | Metin kontrastı ≥ 4.5:1 (büyük metin ≥ 3:1); dokunma hedefleri ≥ 24×24 CSS pikseli veya karşılığı. Erişilebilirlik denetiminde bu iki ölçütte 0 ihlal olmalıdır. | [K13] |
| NFR-04 | **KVKK uyumu:** | (a) Yalnızca bu dokümanda tanımlı amaçlar için veri toplanır; toplanan alanların listesi belgelenir. (b) Silme ve başvuru talepleri en geç 30 gün içinde sonuçlanır. (c) Veri ihlali halinde ilgili kişiyi ve Kurulu bildirme sürecinin sahibi ve adımları yazılı olarak tanımlanır. | [K11 Md. 4, 7, 12, 13] |
| NFR-05 | **Güvenlik ve yetki:** | Damga verme, ödül kullandırma ve geri alma işlemleri yalnızca yetkili kullanıcılarca yapılabilir. Yetkisiz denemelerin test senaryolarında başarı oranı %0 olmalıdır. Veri cihaz dışına aktarılırken şifrelenmelidir (mağaza veri güvenliği beyanında belirtilecek). | [K11 Md. 12][K15] |
| NFR-06 | **Veri doğruluğu:** Damga ve ödül sayıları kaybolmamalı ve çift sayılmamalıdır. | Kabul testlerinde FR-08 kaydı ile kart üzerindeki sayı arasında 0 tutarsızlık olmalıdır. Bağlantı hatasında işlem ya tamamlanır ya da personele açık bir hata mesajı gösterilir; sessiz kayıp olmaz. | BA değerlendirmesi |
| NFR-07 | **Ek donanım gerekmemesi:** | Kafenin ilk sürümü kullanabilmesi için mevcut bir akıllı telefon veya tablet dışında donanım (POS, NFC terminali, yazıcı) gerekmez. | [K1][K8] |
| NFR-08 | **Öğrenilebilirlik:** | Daha önce görmemiş bir personel, en fazla 1 dakikalık açıklamayla ilk damgayı yardımsız verebilmelidir. Kullanılabilirlik testinde katılımcıların ≥ %90'ı başarmalıdır. Kafe sahibi FR-01 ve FR-02'yi ≤ 10 dakikada tamamlayabilmelidir. | [K1][K3: kurulumun "minutes" sürmesi iddiası] |
| NFR-09 | **Dil:** | Tüm müşteri ve kafe ekranları ile aydınlatma metni Türkçe olmalıdır. | Varsayım V-01 |
| NFR-10 | **Platform kapsamı:** | Müşteriler hem iOS hem Android telefonlardan karta erişebilmelidir. Desteklenecek en düşük işletim sistemi sürümleri TA tarafından belirlenir. | Görev ("cep telefonu"); V-02 |
| NFR-11 | **Mağaza uyumu:** | Uygulama mağazası yolu seçilirse yayın öncesi kontrol listesi şunları kapsamalıdır: App Store 4.2, 4.2.6 ve 5.1.1(v); Google Play hesap silme ve Data safety. Kontrol listesinde 0 açık madde ile yayına çıkılır. | [K10][K14][K15] |

---

## 6. Kullanıcı hikâyeleri ve kabul kriterleri

**US-01: Programı kurma** (P-1) — FR-01, FR-02, NFR-08
> Bir kafe sahibi olarak, kafemi kaydedip "10 damgaya 1 bedava kahve" kuralını tanımlamak istiyorum; böylece kâğıt kartı bırakabilirim.
- AC-1: Kafe adı ve ödül metni girilmeden kayıt tamamlanamaz.
- AC-2: Damga sayısı 1–20 dışında girilirse kaydedilmez ve hata mesajı gösterilir.
- AC-3: Kural kaydedildiğinde müşterilerin katılacağı QR kodu veya bağlantı oluşur.
- AC-4: İlk kez kullanan bir kafe sahibi bu adımları ≤ 10 dakikada tamamlar.

**US-02: Karta katılma** (P-4) — FR-03, FR-04, FR-15, NFR-02
> Yeni bir müşteri olarak, kasadaki QR kodu okutup hemen kart sahibi olmak istiyorum; böylece ilk kahvemden itibaren damga toplarım.
- AC-1: QR okutulduktan sonra en fazla 3 ekranda kart "0/10" olarak görünür.
- AC-2: Katılım ekranında aydınlatma metnine erişim bağlantısı bulunur.
- AC-3: S-04'te kararlaştırılan alan dışında zorunlu kişisel veri istenmez.
- AC-4: Kampanya mesajı onayı (varsa) varsayılan olarak işaretsizdir.

**US-03: Damga verme** (P-2) — FR-05, FR-07, FR-08, FR-10, NFR-01, NFR-05
> Bir barista olarak, müşterinin kartına birkaç saniyede damga vermek istiyorum; böylece sıra yavaşlamaz.
- AC-1: Yetkili personel kartı tanımlayıp adet seçtiğinde damga ≤ 3 saniyede karta yansır (%95).
- AC-2: Yetkisi kaldırılmış bir personel damga veremez.
- AC-3: Her damga FR-08 kaydında personel adı ve saatiyle görünür.
- AC-4: Aynı karta 5 dakika içinde ikinci damga verilmek istenirse uyarı ve onay ekranı çıkar (FR-10, S önceliği).
- AC-5: Tek işlem adedi kafe sahibinin belirlediği üst sınırı aşamaz.

**US-04: Ödülü kullanma** (P-3, P-2) — FR-04, FR-06, FR-08, FR-17
> Bir müdavim olarak, kartım dolduğunda bedava kahvemi almak istiyorum.
- AC-1: Kart 10/10 olduğunda müşteri ekranında "ödül hazır" gösterilir.
- AC-2: Personel ödülü kullandırdığında kart 0/10'a döner. Fazla damga varsa (ör. 12) kart 2/10 olur.
- AC-3: Dolmamış bir kartta ödül kullandırma işlemi yapılamaz.
- AC-4: Kullandırma FR-08 kaydına işlenir.

**US-05: Hatalı işlemi düzeltme** (P-1) — FR-08, FR-09
> Bir kafe sahibi olarak, yanlışlıkla verilen damgayı geri almak istiyorum; böylece kayıtlar doğru kalır.
- AC-1: Geri alma için gerekçe girilmesi zorunludur.
- AC-2: Geri alma sonrası kart sayısı ve FR-08 kaydı tutarlıdır (NFR-06).
- AC-3: Personel rolündeki kullanıcı geri alma yapamaz.

**US-06: Kartımı kaybetmemek** (P-3) — FR-11, FR-12
> Bir müdavim olarak, telefonumu değiştirdiğimde damgalarımın kaybolmamasını ve tüm kafe kartlarımı tek yerde görmeyi istiyorum.
- AC-1: Kurtarma işleminden sonra tüm kartlar önceki damga sayılarıyla görünür.
- AC-2: Kurtarma, başka bir kişinin kartlarına erişim sağlamaz.
- AC-3: (S-01 onaylanırsa) Birden çok kafenin kartları tek listede, kafe adıyla görünür.

**US-07: Sonuçları izleme** (P-1) — FR-13
> Bir kafe sahibi olarak, programın işe yarayıp yaramadığını görmek istiyorum.
- AC-1: Seçilen dönem için verilen damga, kullandırılan ödül, kart sahibi müşteri ve aktif müşteri sayıları gösterilir.
- AC-2: Gösterilen sayılar FR-08 kaydından hesaplananla birebir aynıdır.

**US-08: Verilerimi silme** (P-3, P-4) — FR-14, NFR-04
> Bir müşteri olarak, artık kullanmadığımda hesabımı ve verilerimi silmek istiyorum.
- AC-1: Silme seçeneği uygulama içinde hesap ayarlarından bulunabilir.
- AC-2: Silme talebi uygulamaya girmeden bir web bağlantısıyla da iletilebilir.
- AC-3: Talep en geç 30 gün içinde sonuçlanır ve müşteriye sonuç bildirilir.
- AC-4: Yasal nedenle saklanan bir veri varsa bu durum müşteriye açıkça belirtilir.

**US-09: Kampanya mesajı tercihi** (P-3, P-1) — FR-16
> Bir müşteri olarak, kampanya mesajı alıp almayacağıma kendim karar vermek istiyorum.
- AC-1: Onay vermemiş müşteriye kampanya mesajı gönderilemez.
- AC-2: Müşteri onayını tek adımda geri alabilir. Geri alma talebinden sonra en geç 3 iş günü içinde kampanya mesajı gönderimi durur.

**US-10: Cüzdana ekleme** (P-3) — FR-18
> Bir müşteri olarak, kartımı telefonumun cüzdanına eklemek istiyorum; böylece ayrı uygulama açmam gerekmez.
- AC-1: Kart cüzdana eklendiğinde kafe adı ve damga sayısı görünür.
- AC-2: Yeni damga verildiğinde cüzdandaki sayı güncellenir.

**US-11: Hoş geldin damgası** (P-1) — FR-19
> Bir kafe sahibi olarak, yeni müşterilere başlangıç damgası vermek istiyorum; böylece kartı tamamlama isteği artar.
- AC-1: Başlangıç damgası 0 ile (damga sayısı−1) arasında ayarlanabilir.
- AC-2: Başlangıç damgası müşteriye gerekçesiyle gösterilir (ör. "Hoş geldin damgası").

---

## 7. Kapsam dışı maddeler

| ID | Madde | Neden |
|---|---|---|
| KD-01 | POS / kasa sistemi entegrasyonu ve satışla otomatik damga | Küçük kafede ek sistem bağımlılığı getirir; görevde istenmedi [K4] |
| KD-02 | Harcama tutarına göre puan, VIP seviyeleri, puan çarpanları | Görev damga temelli ("10 kahvede 1"); Square tarzı puanlama ürünü büyütür [K4] |
| KD-03 | Uygulama içi ödeme, ön ödemeli bakiye, hediye kartı | Görevde yok; ek düzenleme ve risk getirir |
| KD-04 | Çok şubeli zincir yönetimi | Hedef küçük kafeler (V-03) |
| KD-05 | Apple Wallet / Google Wallet ile NFC otomatik tanınma, NFC etiketi veya tezgâh cihazı | Sertifikalı terminal veya donanım gerektirir [K8][K3]; NFR-07 ile çelişir |
| KD-06 | SMS veya e-posta kampanya araçları, CRM entegrasyonu, doğum günü ödülü | Rakiplerde var [K3][K4] ama görevde istenmedi |
| KD-07 | Oyunlaştırma (kazı-kazan vb.) | [K3]'te var; görevde yok |
| KD-08 | Kafe keşfi, yorum ve sosyal özellikler | [K6]'da var; görevde yok |
| KD-09 | Her kafeye ayrı markalı uygulama yayımlama | Apple 4.2.6 riski [K10]; S-01 ile kesinleşir |

---

## 8. Varsayımlar

| ID | Varsayım |
|---|---|
| V-01 | Ürün Türkiye'de, Türkçe olarak sunulacaktır. Bu nedenle KVKK ve 6563 sayılı Kanun esas alınmıştır. |
| V-02 | Görevde platform belirtilmemiştir. "Cep telefonu" ifadesinden hem iOS hem Android kullanıcılarının hedeflendiği varsayılmıştır. Yerel uygulama, web veya Wallet seçimi TA'ya bırakılmıştır. |
| V-03 | Hedef kafeler tek şubeli ve 1–5 çalışanlıdır. Personel damga verirken kendi telefonunu veya kafeye ait bir telefon/tableti kullanır. |
| V-04 | Kafe başına tek bir sadakat kuralı (tek kart türü) yeterlidir. |
| V-05 | Hedef kitle yetişkindir. Çocuklara yönelik özel düzenlemeler bu nedenle uygulanmamıştır. 18 yaş altı kullanıcıların durumu S-08'de sorulmuştur. |
| V-06 | Rakip özellikleri, fiyatları ve işletme sayıları üreticilerin kendi sayfalarından alınmıştır ve tanıtım iddiası niteliğindedir. Bağımsız olarak doğrulanmamıştır. Fiyatlar araştırma tarihindeki değerlerdir. |
| V-07 | Başlangıç damgası etkisine ilişkin oranlar doğrulanamamıştır. Orijinal makale (USC sunucusu) HTTP 503 hatası verdi. Yalnızca Wharton haberindeki niteliksel sonuç kullanılmıştır. |
| V-08 | Kötüye kullanım önleme gereksinimleri (FR-09, FR-10) incelenen kaynaklarda ayrıntılı bulunamadığı için BA değerlendirmesine dayanır. Loopy Loyalty'nin güvenlik sayfası açılamadı (HTTP 404). |
| V-09 | KVKK ve 6563 sayılı Kanun hükümleri, mevzuat.gov.tr'deki kanun metinlerinden aktarılmıştır. Yönetmelik ve Kurul kararları incelenmemiştir. Hukuki yorum değil, iş gereksinimi girdisidir. |

---

## 9. Açık sorular

| ID | Soru | Etkilediği gereksinim |
|---|---|---|
| S-01 | Ürün birçok kafeyi barındıran tek bir uygulama ve hizmet mi olacak, yoksa her kafeye özel markalı uygulama mı? İkincisinde Apple 4.2.6 nedeniyle uygulamayı kafenin kendisinin yayımlaması gerekebilir. | FR-12, KD-09, NFR-11 |
| S-02 | İş modeli nedir: kafeye abonelik mi, ücretsiz mi? Rakiplerle fiyat konumlandırması için referans: Loopy Starter $25/ay. | Kapsam, FR-13 |
| S-03 | Müşterinin uygulama indirmesi kabul edilebilir mi, yoksa indirmeden katılım (web veya Wallet) bir iş zorunluluğu mu? | FR-03, FR-18, NFR-02 |
| S-04 | Katılımda müşteriden hangi veri istenecek: hiçbir şey, e-posta veya telefon? Kart kurtarma (FR-11) ile en az veri ilkesi arasındaki denge nasıl kurulacak? | FR-03, FR-11, NFR-02, NFR-04 |
| S-05 | Damga verme yöntemi için iş tercihi var mı? Ör. yalnızca personelin onayıyla (Y-1) mı olmalı, yoksa müşterinin kendi okuttuğu kod (Y-2/Y-3) da kabul edilebilir mi? | FR-05, FR-10 |
| S-06 | Damgaların veya ödülün geçerlilik süresi olacak mı (ör. 6 ay sonra silinme)? Kural değiştiğinde mevcut kartlar eski kuralla mı devam eder? | FR-02, FR-06 |
| S-07 | Uygulama içi push bildirimleri 6563 sayılı Kanun kapsamında "ticari elektronik ileti" sayılır mı? İleti Yönetim Sistemi kaydı gerekir mi? "Ödülünüz hazır" gibi bilgilendirmeler onay istisnasına girer mi? (Hukuk danışmanına sorulmalı.) | FR-16, FR-17 |
| S-08 | 18 yaş altı müşterilerin katılımına izin verilecek mi? | FR-03, FR-15 |
| S-09 | KVKK açısından veri sorumlusu kim olacak: kafe mi, ürün sahibi mi, ortak mı? Veriler yurt dışında işlenecekse Md. 9 koşulları nasıl karşılanacak? | FR-15, NFR-04 |
| S-10 | İnternet bağlantısı olmadığında damga verilebilmeli mi? Kafelerdeki bağlantı güvenilirliği hakkında bilgi var mı? | NFR-06 |
| S-11 | Kafe sahibi müşterilerini tek tek (ad veya iletişim bilgisiyle) görmek istiyor mu, yoksa toplu göstergeler (FR-13) yeterli mi? | FR-13, NFR-04 |

---

## 10. Kaynaklar

| No | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | Loopy Loyalty: Cafe loyalty program | https://loopyloyalty.com/industries/cafe-loyalty-program/ | Wallet'a eklenen kart, QR veya bağlantıyla katılım. Alıntılar: "Their digital stamp card is instantly added to Apple or Google Wallet"; "Any phone or tablet can be used to stamp loyalty cards"; "No POS integration, no special hardware, and no slowing down the queue"; "Their card can be restored with its collected stamps and rewards". Ayrıca Smart Messaging, analitik, 15 günlük deneme. |
| K2 | Loopy Loyalty: Pricing | https://loopyloyalty.com/pricing/ | Planlar: Starter $25/ay (1 tasarım, 1 lokasyon, en fazla 3 Stamper kullanıcısı); Growth $69/ay; Ultimate $95/ay. Tüm planlarda sınırsız müşteri; 15 gün ücretsiz deneme. |
| K3 | Stamp Me (ana sayfa) | https://stampme.com/ | Uygulama indirmeden web uygulamasıyla katılım. Damga yöntemleri: StampPod, Stamp Tag, OneStamps ("issue one stamp then expire"), Stamp Business, Loyalty Portal. Push/SMS, doğum günü, oyunlaştırma, CRM; "50+ countries" (üretici iddiası); SOC 2 Type 2 rozeti. |
| K4 | Square Loyalty | https://squareup.com/us/en/software/loyalty | Telefon numarasıyla katılım ("type in their phone number and get points automatically", bir işletme sahibinin ifadesi). Ziyaret, tutar veya ürün bazlı puan; puanlar "even when they use cash" kazanılır. VIP seviyeleri, Apple Wallet kartı, otomatik SMS. Kanallar Square ürünleri üzerinden. |
| K5 | App Store: Yine Bekleriz: Müşteri | https://apps.apple.com/us/app/awrd/id6446122448 | Türkiye'de çok işletmeli ücretsiz müşteri uygulaması. "QR kodunu göster ya da telefonunu NFC etiketine yaklaştır"; 4.000'den fazla işletme (üretici iddiası); 1 değerlendirme. |
| K6 | App Store: Kupa – Kahveci Sadakat | https://apps.apple.com/tr/app/kupa/id6753937573 | Kafe keşfi ve sadakat uygulaması: "Kahve içtikçe puan topla", hediye kahve, kampanya bildirimleri. 32 değerlendirmede 4,4. Kafe tarafı araçları anlatılmıyor. |
| K7 | Apple Developer: Wallet | https://developer.apple.com/wallet/ | Store card türü: "Store loyalty cards, discount cards, points cards, and gift cards"; bakiye güncellenebilir. |
| K8 | Apple Developer: Loyalty passes | https://developer.apple.com/wallet/loyalty-passes/ | Dağıtım kanalları: "app, App Clips, SMS, email, near field communication (NFC) tags, and QR codes". Güncelleme ve bildirim. NFC koşulu: "must already accept Apple Pay and be certified for VAS". |
| K9 | Google for Developers: Google Wallet loyalty cards | https://developers.google.com/wallet/retail/loyalty-cards | Pass'lar uygulamada ve "anywhere hyperlinks work" dağıtılabilir; "immediate visibility to points and rewards"; Google Wallet API Issuer hesabı gerekir. |
| K10 | Apple: App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 4.2: "elevate it beyond a repackaged website". 4.2.6: şablon uygulamalar "will be rejected unless submitted directly by the provider of the app's content"; "picker" modeli kabul edilir. 5.1.1(v): "you must also offer account deletion within the app"; giriş ve kişisel veri zorunluluğu sınırı. |
| K11 | 6698 sayılı Kişisel Verilerin Korunması Kanunu (mevzuat.gov.tr) | https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6698.pdf | Md. 4 "amaçla bağlantılı, sınırlı ve ölçülü". Md. 5 açık rıza ve sözleşme istisnası. Md. 7 silme. Md. 9 yurt dışına aktarım. Md. 10 aydınlatma unsurları. Md. 12 güvenlik ve ihlal bildirimi ("en kısa sürede"). Md. 13 "en geç otuz gün içinde". |
| K12 | 6563 sayılı Elektronik Ticaretin Düzenlenmesi Hakkında Kanun (mevzuat.gov.tr) | https://www.mevzuat.gov.tr/mevzuatmetin/1.5.6563.pdf | Md. 6: "ancak önceden onayları alınmak kaydıyla gönderilebilir"; değişiklik, kullanım ve bakım iletileri istisnası. Md. 8: "hiçbir gerekçe belirtmeksizin" ret; "üç iş günü içinde" durdurma. Md. 2: ticari elektronik ileti tanımı. |
| K13 | W3C: Web Content Accessibility Guidelines (WCAG) 2.2 | https://www.w3.org/TR/WCAG22/ | 1.4.3: "contrast ratio of at least 4.5:1", büyük metin 3:1. 2.5.8: "at least 24 by 24 CSS pixels" (istisnalarla). |
| K14 | Google Play Console Yardım: Hesap silme gereksinimleri | https://support.google.com/googleplay/android-developer/answer/13327111 | "an in-app path to delete their app accounts and associated data" ve "a web link resource" zorunluluğu; ilişkili verilerin silinmesi. |
| K15 | Google Play Console Yardım: Data safety bölümü | https://support.google.com/googleplay/android-developer/answer/10787469 | Tüm yayımlanan uygulamalar formu doldurur; veri toplama, paylaşım, aktarımda şifreleme ve silme beyanı; beyanın doğruluğundan geliştirici sorumludur. |
| K16 | Knowledge at Wharton: Nunes ve Drèze sadakat kartı deneyi haberi | https://knowledge.wharton.upenn.edu/?p=5630 | 300 oto yıkama müşterisi; 8 damgalı kart ile 2'si önceden verilmiş 10 damgalı kart karşılaştırması. Yazarların sonucu: "increases the likelihood of task completion and decreases completion time". Haberde oran verilmiyor. |

**Açılamayan veya kullanılmayan kaynaklar:**
- Nunes ve Drèze, "The Endowed Progress Effect" makalesi (USC sunucusu): HTTP 503 hatası verdi. Oranlar doğrulanamadı ve kullanılmadı.
- Loopy Loyalty güvenlik sayfası (`/features/security-reliability/`): HTTP 404 hatası verdi.
- Arama sonuçlarında yalnızca başlığı görülen sayfalar kaynak gösterilmemiştir: Capterra, Stamp Me blogları, Coffily, TechCrunch, EurekAlert vb.