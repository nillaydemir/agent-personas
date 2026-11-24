# Küçük Kafeler İçin Dijital Sadakat Kartı Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ta v2
- **Girdi:** outputs/sadakat-karti/ba-sadakat-karti-v2-run1.md

---

## 1. Özet

Ürün, birçok kafeyi barındıran **tek bir web uygulaması** olarak tasarlanmıştır. Uygulama, ana ekrana eklenebilen bir web uygulamasıdır (PWA). Müşteri, kafe sahibi ve personel aynı uygulamayı rollerine göre kullanır. Arka uç olarak **yönetilen bir PostgreSQL hizmeti (Supabase)** kullanılır. Bu hizmet kimlik doğrulama, satır düzeyinde yetki (RLS) ve veritabanı fonksiyonlarını tek pakette sunar.

Temel kararlar:
- **Platform:** PWA, uygulama mağazası yok. Müşteri QR'ı okutup indirmeden katılır (FR-03, NFR-02, NFR-10, NFR-11).
- **Damga yöntemi:** Personel, müşterinin ekranındaki kart QR'ını kendi telefonunun kamerasıyla tarar. Bu, BA dokümanındaki Y-1 yöntemidir (FR-05, NFR-05, NFR-07).
- **Sayaç bütünlüğü:** Her damga, ödül ve geri alma işlemi tek bir veritabanı fonksiyonunda, kart satırı kilitlenerek işlenir. İşlem kaydı (ledger) ve kart bakiyesi aynı işlemde (transaction) yazılır. Yinelenen istekler idempotency anahtarıyla engellenir (FR-08, NFR-06).
- **Müşteri kimliği:** Varsayılan olarak anonim hesap kullanılır. Kart kurtarma için isteğe bağlı olarak e-posta bağlanabilir (FR-03, FR-11). Kesin karar BA'nın S-04 sorusunun yanıtına bağlıdır.
- **Kapsam:** MVP, M önceliğindeki tüm FR'leri karşılar. C önceliğindeki FR-16, FR-17 ve FR-19 için yalnızca veri modeli ve basit akış tasarlanmıştır. FR-18 (Wallet) bu sürümde ertelenmiştir (bkz. Bölüm 10, R-07).

Birkaç karar **geçici** olarak işaretlenmiştir. Bunlar BA'nın açık sorularına (S-01, S-03, S-04, S-06, S-09) bağlıdır ve Bölüm 12'de geri sorulmuştur.

---

## 2. Mimari genel bakış

```
 [Müşteri telefonu]           [Personel / kafe sahibi telefonu]
   PWA (müşteri ekranları)      PWA (personel + yönetim ekranları)
   - kart QR'ını gösterir       - kamera ile QR tarar (qr-scanner)
        │  HTTPS                        │  HTTPS
        ▼                               ▼
 ┌──────────────────────── Supabase projesi (AB bölgesi) ────────────────────────┐
 │  Auth: anonim müşteri, e-posta OTP ile personel/sahip                         │
 │  PostgreSQL: tablolar + RLS + SECURITY DEFINER RPC fonksiyonları              │
 │     add_stamps / redeem_reward / reverse_event / join_cafe / preview_rule     │
 │  Sunucu fonksiyonu (Edge Function): hesap silme (service_role ile)            │
 └───────────────────────────────────────────────────────────────────────────────┘
        ▲
        │ HTTPS (statik)
 [Statik barındırma: PWA dosyaları, aydınlatma metni, web silme talep formu]
```

Mimari ilkeler:
- **Tek kod tabanı, tek uygulama, çok kafe** ("picker" modeli). Bu model BA'nın S-01 sorusunda önerdiği seçenektir (FR-12, KD-09, NFR-11). S-01 yanıtı "her kafeye ayrı uygulama" olursa mimari yeniden değerlendirilir. Bu nedenle karar **geçicidir**.
- **İş kuralları veritabanında:** Damga, ödül ve geri alma kuralları istemcide değil, PostgreSQL fonksiyonlarında çalışır. İstemci yalnızca fonksiyonu çağırır. Bu sayede yetki (NFR-05) ve tutarlılık (NFR-06) tek yerde sağlanır.
- **Ayrı sunucu yok:** Silme dışında özel bir sunucu kodu yoktur. Bu, 1–5 çalışanlı tek şubeli kafeler için ürün büyüklüğüne uygundur (V-03).

---

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Kritik karar K-1: İstemci platformu

**Seçim ölçütleri:**
- Ö1: Uygulama indirmeden katılım, ≤ 60 sn ve ≤ 3 ekran (NFR-02, FR-03, S-03).
- Ö2: Tek çözümle iOS ve Android (NFR-10).
- Ö3: Mağaza politikası riski (NFR-11; App Store 4.2 ve 4.2.6, BA [K10]).
- Ö4: Personelin ek donanım olmadan kamera ile tarayabilmesi (NFR-07, FR-05).
- Ö5: Uygulama dışından silme talebi için web yolu (FR-14).
- Ö6: Bildirim desteği (FR-17, C önceliği).

| Ölçüt | A) PWA (web) | B) Çapraz platform yerel uygulama (mağazadan) | C) Yalnızca Wallet pass + web yönetim paneli |
|---|---|---|---|
| Ö1 | Karşılar: QR bağlantısı doğrudan açılır | Karşılamaz: indirme adımı var | Karşılar [BA K7, K9] |
| Ö2 | Karşılar (tarayıcı) | Karşılar | Kısmen: iki ayrı pass altyapısı (Apple ve Google) |
| Ö3 | Mağaza yok; risk yok | 4.2.6 ve 5.1.1(v) yükümlülükleri | Mağaza yok; ancak Apple ve Google hesap koşulları var [BA K8, K9] |
| Ö4 | Karşılar; `getUserMedia` HTTPS ve kullanıcı izni ister [T16] | Karşılar | Personel tarafı yine bir uygulama gerektirir |
| Ö5 | Doğal olarak web | Ayrıca web sayfası gerekir | Web panel ile karşılanır |
| Ö6 | iOS'ta yalnızca ana ekrana eklenmiş web uygulamasında, iOS 16.4 ve sonrasında [T17] | Tam destek | Pass güncellemesi ve bildirim var [BA K8] |

**Seçim: A) PWA (geçici, S-03'e bağlı).** Ö1, Ö3 ve Ö5'te tek tam karşılayan seçenektir. Ö6'daki zayıflık kabul edilebilir: FR-17 C önceliğindedir ve bildirim gönderimi isteğe bağlıdır, uygulama içi gösterim yeterlidir. S-03 yanıtı "yerel uygulama zorunlu" olursa B seçeneğine geçilir. Bu durumda NFR-11 kontrol listesi devreye girer.

### 3.2 Kritik karar K-2: Arka uç ve veri saklama

**Seçim ölçütleri:**
- Ö1: Damga ve ödül sayacının atomik güncellenmesi ve işlem kaydıyla tutarlılığı (NFR-06, FR-08).
- Ö2: Yetki kurallarının istemciden bağımsız uygulanması (NFR-05, FR-07).
- Ö3: Dönemsel toplam göstergeler için sorgu kolaylığı (FR-13).
- Ö4: Veri konumu ve yurt dışı aktarım (NFR-04, S-09).
- Ö5: İşletme yükünün ürün büyüklüğüne uygunluğu (kapsam kuralı, V-03).
- Ö6: Anonim katılım ve sonradan kimlik bağlama (FR-03, FR-11).

| Ölçüt | A) Supabase (yönetilen Postgres + Auth) | B) Firebase (Firestore + Auth) | C) Kendi sunucumuz + Postgres (Türkiye'de barındırma) |
|---|---|---|---|
| Ö1 | Postgres fonksiyonu + `SELECT FOR UPDATE` ile satır kilidi [T10] | Transaction çakışmada yeniden çalıştırılır ve çevrimdışıyken başarısız olur [T11] | Ö1'de A ile aynı (Postgres) |
| Ö2 | RLS politikaları veritabanında çalışır [T5] | Güvenlik kuralları (bu dokümanda incelenmedi) | Uygulama kodunda yazılır |
| Ö3 | SQL ile toplama sorguları | Belge veritabanında toplama sorguları daha sınırlı (bu dokümanda doğrulanmadı) | SQL |
| Ö4 | AB bölgeleri var, Türkiye bölgesi yok; bölge sonradan değiştirilemez [T3] | Avrupa konumları var, Türkiye yok; konum sonradan değiştirilemez [T12] | Türkiye'de tutulabilir |
| Ö5 | Düşük: yönetilen hizmet | Düşük | Yüksek: sunucu, yedek ve güvenlik yamaları ekibin sorumluluğu |
| Ö6 | Anonim giriş var; sonradan e-posta veya OAuth kimliği bağlanabilir [T1][T2] | (bu dokümanda incelenmedi) | Elle yazılır |

**Seçim: A) Supabase, Frankfurt (`eu-central-1`) bölgesi (geçici, S-09'a bağlı).** Ö1, Ö2, Ö3, Ö5 ve Ö6'da en iyi dengeyi sağlar. Ö4 açıktır: veriler yurt dışında (AB) işlenir. KVKK Md. 9 koşullarının nasıl karşılanacağı BA'nın S-09 sorusudur. Yanıt "Türkiye'de tutulmalı" olursa C seçeneğine geçilir. Şema, fonksiyonlar ve RLS standart PostgreSQL olduğu için taşınabilir. Bölge seçimi sonradan değiştirilemediği için proje açılmadan önce S-09 netleşmelidir [T3].

### 3.3 Kritik karar K-3: Damga verme yöntemi

**Seçim ölçütleri:**
- Ö1: Yetkisiz damganın engellenmesi (NFR-05: test senaryolarında başarı oranı %0).
- Ö2: Ek donanım gerektirmemesi (NFR-07).
- Ö3: İşlem süresi ≤ 3 sn, %95 (NFR-01).
- Ö4: Öğrenilebilirlik (NFR-08).
- Ö5: Tekrar kullanım ve sahte damga riski (FR-10, BA Y-2 değerlendirmesi).

| Ölçüt | Y-1: Personel müşterinin kart QR'ını tarar | Y-2: Müşteri kafedeki sabit QR'ı okutur | Y-3: Tek kullanımlık kod |
|---|---|---|---|
| Ö1 | İşlemi yetkili personel oturumu başlatır | İşlemi müşteri başlatır; personel onayı yok | Kod üreten personel yetkili |
| Ö2 | Personel telefonunun kamerası yeterli | Basılı QR yeterli | Kodu göstermek için ekran gerekir |
| Ö3 | 1 tarama + 1 dokunma | Personel adımı yok | Kod üretme + müşteri okutma, iki adım |
| Ö4 | Tek ana düğme ("Tara") | Personel eğitimi gerekmez | Ek adım öğrenilir |
| Ö5 | Düşük: damga kararı personelde | Yüksek: kod fotoğraflanıp tekrar kullanılabilir (BA değerlendirmesi) | Düşük |

**Seçim: Y-1 (geçici, S-05'e bağlı).** Ö1 ve Ö5'i karşılayan seçenekler arasında Ö3 ve Ö4'te daha sadedir. Kamera açılamazsa (izin reddi, bozuk kamera) personel, kart QR'ının altında görünen kısa kodu elle girebilir. Bu yedek yol NFR-01 ve NFR-08'in hata durumunda da sağlanması içindir. Yeni bir gereksinim değildir; aynı FR-05 işlemini çağırır.

### 3.4 Kütüphane ve hizmet kontrol tablosu

| Bileşen | Kullanım | (a) İzinler | (b) En düşük platform sürümü | (c) Belgedeki bilinen kısıtlar | Bağlı gereksinim |
|---|---|---|---|---|---|
| **Supabase Auth: anonim giriş** [T1] | Müşterinin kişisel veri vermeden katılması | Tarayıcı depolaması (oturum) | Belgede yok | Çıkış yapan, tarayıcı verisini silen veya başka cihaz kullanan anonim kullanıcı hesabına erişemez. Anonim kullanıcıların otomatik temizliği yok. IP başına saatte 30 istek varsayılan sınırı var (panelden değiştirilebilir). CAPTCHA "strongly recommended". RLS politikaları önceden gözden geçirilmeli. | FR-03, FR-11, NFR-02 |
| **Supabase Auth: kimlik bağlama** [T1][T2] | Kurtarma için e-posta veya OAuth bağlama | Belgede yok | Belgede yok | Manuel bağlama projede ayrıca etkinleştirilmeli ve "beta" olarak işaretli. Kimlik ayırma için en az 2 bağlı kimlik gerekir. | FR-11 |
| **Supabase Auth: CAPTCHA** [T7] | Anonim kayıt kötüye kullanımına karşı | Üçüncü taraf hizmet (hCaptcha veya Cloudflare Turnstile) | Belgede yok | Belge yalnızca sign-in, sign-up ve parola sıfırlama formlarını anar; anonim giriş [T1]'de ayrıca önerilir. Yerel testte alan adı izin listesi gerekir. | NFR-05 |
| **Supabase Postgres: RLS** [T5] | Satır düzeyinde yetki | — | — | Açık şemadaki RLS'siz tablo, yetkisi olan her rol tarafından okunup yazılabilir. RLS açık ve politika yoksa erişim reddedilir. Oturumsuz istekte `auth.uid()` null döner. | NFR-05, FR-07 |
| **Supabase Postgres: veritabanı fonksiyonları (RPC)** [T4] | Damga, ödül ve geri alma işlemleri | — | — | `security definer` fonksiyonda `search_path` ayarlanmalı (öneri: boş). Panelden oluşturulan fonksiyonların sahibi `postgres` olduğu için RLS'yi atlar; yetki kontrolü fonksiyon gövdesinde yapılmalı ve çalıştırma yetkileri daraltılmalı. | FR-05, FR-06, FR-09, NFR-05 |
| **Supabase Auth Admin: deleteUser** [T6] | Hesap silme | `service_role` anahtarı | — | Yalnızca sunucuda çağrılmalı; anahtar tarayıcıya verilmemeli. Belge, diğer tablolardaki satırların silinip silinmediğini söylemiyor; uygulama verisi ayrıca silinmeli. `shouldSoftDelete` geri alınamaz. | FR-14 |
| **Supabase Edge Functions** [T8] | `deleteUser` çağrısı için sunucu ortamı | Proje secret'ları (ortam değişkeni) | — | Soğuk başlatma olabilir; kısa ve idempotent işler önerilir. İstekler kullanıcıya en yakın bölgesel düğümde çalışır (bkz. R-02). | FR-14 |
| **Supabase bölgesi** [T3] | Veri konumu | — | — | Türkiye bölgesi yok; bölge sonradan değiştirilemez. | NFR-04, S-09 |
| **qr-scanner (nimiq)** [T13], MIT | Personel ekranında QR okuma | Kamera izni (kullanıcıya sorulur); HTTPS zorunlu | Belgede sürüm yok. Normal derleme ES2017 ister; eski tarayıcılar için ES2015 uyumlu "legacy" derleme var. | Varsa tarayıcının yerel `BarcodeDetector`'ını kullanır, işlem WebWorker'da çalışır. Kamera tercihi (ör. arka kamera) garanti edilemez. Eski string döndüren API kaldırılacak. | FR-05, NFR-01, NFR-07 |
| **Tarayıcı `getUserMedia`** [T16] | Kamera erişimi | Her zaman kullanıcı izni; yalnızca güvenli bağlam (HTTPS) | — | Güvensiz bağlamda `navigator.mediaDevices` tanımsızdır. İzin isteği hiç yanıtlanmayabilir; promise ne çözülür ne reddedilir. | FR-05, NFR-08 |
| **Barcode Detection API** [T15] | qr-scanner'ın hızlı yolu (dolaylı) | Güvenli bağlam | MDN "Limited availability" ve "Experimental" diyor; uyumluluk tablosu sayfa özetinde okunamadı | Doğrudan kullanılmaz; qr-scanner yoksa kendi çözücüsüne geçer. | NFR-01 |
| **qrcode (node-qrcode)** [T14], MIT | Kart QR'ı ve kafe katılım QR'ı üretimi | Yok | Hazır paket: "Internet Explorer 10+, Safari 5.1+, and all evergreen browsers" | Çok baytlı karakterler Byte modunda kodlanır; ECI desteği tam değil. QR içeriği yalnızca ASCII URL ve token olacağı için sorun yok. | FR-03, FR-05, US-01 AC-3 |
| **Web Push (iOS)** [T17] | FR-17 için isteğe bağlı bildirim | Kullanıcı izni; izin isteği doğrudan kullanıcı etkileşimine yanıt olarak yapılmalı | iOS/iPadOS 16.4 | Yalnızca ana ekrana eklenmiş web uygulamalarında çalışır. | FR-17 |

Arayüz çatısı (UI framework) seçimi kritik bir karar değildir: doğruluk, güvenlik veya veri bütünlüğü iş kurallarına değil, veritabanı fonksiyonlarına bağlıdır. Dil TypeScript'tir. Seçilen çatı uygulama aşamasında bu tabloya aynı ölçütlerle eklenir.

---

## 4. Bileşenler ve sorumlulukları

| ID | Bileşen | Sorumluluk | Karşıladığı gereksinim |
|---|---|---|---|
| B-01 | **PWA: müşteri ekranları** | Katılım ekranı (aydınlatma bağlantısı, isteğe bağlı kampanya onayı), kart listesi, kart ekranı (n/N, "ödül hazır", kart QR'ı ve kısa kod), kart geçmişi, hesap ayarları (e-posta bağlama, silme) | FR-03, FR-04, FR-08, FR-11, FR-12, FR-14, FR-15, FR-16, FR-17, NFR-02, NFR-03, NFR-09 |
| B-02 | **PWA: personel ekranı** | "Tara" düğmesi, kamera ile QR okuma, kısa kod girişi, adet seçimi, onay, FR-10 uyarısı, ödül kullandırma | FR-05, FR-06, FR-10, NFR-01, NFR-07, NFR-08 |
| B-03 | **PWA: kafe sahibi paneli** | Kafe kaydı, kural tanımı ve değişiklik önizlemesi, personel ekleme ve kaldırma, işlem kaydı, geri alma, göstergeler, katılım QR'ını indirme ve yazdırma, kafe silme | FR-01, FR-02, FR-07, FR-08, FR-09, FR-13, FR-14, FR-19 |
| B-04 | **Kimlik doğrulama (Supabase Auth)** | Müşteri: anonim oturum ve isteğe bağlı e-posta bağlama. Sahip ve personel: e-posta ile tek kullanımlık kod (OTP) girişi. | FR-03, FR-07, FR-11, NFR-05 |
| B-05 | **Veritabanı ve RLS** | Tablolar (Bölüm 5), salt okuma politikaları; doğrudan yazmaya kapalı | FR-08, NFR-05, NFR-06 |
| B-06 | **İşlem fonksiyonları (RPC)** | `join_cafe`, `add_stamps`, `redeem_reward`, `reverse_event`, `preview_rule_change`, `get_metrics` (Bölüm 6) | FR-02, FR-03, FR-05, FR-06, FR-09, FR-10, FR-13, FR-19, NFR-06 |
| B-07 | **Silme fonksiyonu (Edge Function)** | Kimliği doğrulanmış müşteri veya kafe sahibi için uygulama verisini silme ve `deleteUser` çağrısı | FR-14, NFR-04 |
| B-08 | **Statik sayfalar** | Aydınlatma metni (sürümlü), uygulama dışı silme talep formu | FR-14, FR-15, NFR-04 |
| B-09 | **Silme talepleri takibi** | Web formundan gelen talepleri 30 günlük son tarihle kaydeder; kafe sahibi veya ürün sahibi tarafından işlenir (sorumlu S-09'a bağlı) | FR-14, NFR-04 |

Ekran akışı (US-02, NFR-02): Kafe QR'ı okutulur → (1) katılım ekranı: kafe adı, ödül, aydınlatma bağlantısı, işaretsiz kampanya onayı, "Karta katıl" → (2) kart ekranı "0/10". Toplam 2 ekran.

---

## 5. Veri modeli

PostgreSQL şeması. Bütün tablolarda RLS açıktır [T5]. İstemcinin doğrudan yazma yetkisi yoktur; yazmalar yalnızca B-06 fonksiyonlarıyla yapılır.

| Tablo | Alanlar (özet) | Not | Gereksinim |
|---|---|---|---|
| `cafe` | `id`, `name`, `logo_url`, `description`, `stamps_required` (CHECK 1–20, varsayılan 10), `reward_text`, `max_stamps_per_tx` (≥ 1), `duplicate_window_min` (varsayılan 5), `welcome_stamps` (CHECK 0 ≤ x ≤ `stamps_required` − 1, varsayılan 0), `join_token`, `created_at` | `name` ve `reward_text` NOT NULL (US-01 AC-1) | FR-01, FR-02, FR-05, FR-10, FR-19 |
| `cafe_member` | `cafe_id`, `user_id`, `role` (`owner` / `staff`), `display_name`, `active`, `created_at`, `revoked_at` | Yetki kaldırılınca `active = false`; kayıt silinmez, geçmiş işlemlerde personel adı görünür | FR-07, FR-08 |
| `card` | `id`, `cafe_id`, `customer_user_id`, `scan_token` (128 bit rastgele, benzersiz), `short_code` (8 karakter, kafe içinde benzersiz), `balance` (CHECK ≥ 0), `last_stamp_at`, `notice_version`, `created_at` | UNIQUE(`cafe_id`, `customer_user_id`): müşteri başına kafe başına tek kart (V-04) | FR-03, FR-04, FR-05, FR-12, FR-15 |
| `card_event` | `id`, `card_id`, `cafe_id`, `type` (`stamp` / `redeem` / `reversal` / `welcome`), `delta` (işaretli tam sayı), `actor_user_id`, `reason`, `reverses_event_id` (UNIQUE), `idempotency_key` (UNIQUE), `balance_after`, `created_at` | Ekleme dışında değiştirilemez (append-only). Her olay en fazla bir kez geri alınabilir. | FR-06, FR-08, FR-09, FR-13, NFR-06 |
| `marketing_consent` | `card_id`, `granted`, `granted_at`, `revoked_at`, `consent_text_version` | Varsayılan yok (onay yok). Yalnızca FR-16 uygulanırsa kullanılır. | FR-16 |
| `privacy_notice` | `version`, `content`, `published_at` | Müşteriye gösterilen sürüm `card.notice_version`'a yazılır | FR-15 |
| `deletion_request` | `id`, `channel` (`app` / `web`), `subject_ref` (kısa kod veya bağlı e-posta), `status`, `received_at`, `due_at` (= `received_at` + 30 gün), `completed_at` | NFR-04(b) takibi | FR-14, NFR-04 |

**Toplanan kişisel veri envanteri** (NFR-04(a) ve Google Play Data safety beyanı [BA K15] için temel):
- Müşteri: anonim kullanıcı kimliği, kartlar, işlem geçmişi, bağlanırsa e-posta, kampanya onayı. Ad, telefon ve konum **toplanmaz**.
- Personel ve sahip: e-posta, görünen ad, rol.
- Teknik: Supabase Auth'un oturum ve IP tabanlı hız sınırı için işlediği veriler [T1]. Ayrıntısı Supabase belgesinden ayrıca doğrulanmalıdır (bkz. R-02).

Kart bakiyesi (`balance`) bir önbellek değil, işlem kaydıyla aynı transaction'da güncellenen asıl değerdir. Değişmez kural: her kart için `balance = SUM(card_event.delta)`. Bu kural testlerde ve günlük bir denetim sorgusuyla kontrol edilir (NFR-06).

---

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Kritik karar K-4: Sayaç güncelleme ve tutarlılık

**Seçim ölçütleri:**
- Ö1: Kayıp veya çift sayım olmaması (NFR-06).
- Ö2: İşlem kaydı ile kart sayısının birebir tutarlılığı (FR-08, NFR-06, US-07 AC-2).
- Ö3: İşlem süresi (NFR-01).
- Ö4: Eşzamanlı işlemlerde doğruluk. Risk: iki personel aynı kartı aynı anda tarar.

| Ölçüt | A) Bakiye + olay aynı transaction'da, satır kilidi, idempotency anahtarı | B) Yalnızca olay kaydı; bakiye her okumada SUM ile hesaplanır | C) İstemci tarafında oku-hesapla-yaz |
|---|---|---|---|
| Ö1 | Yeniden denemede aynı anahtar ikinci olay oluşturmaz | Yeniden denemede çift olay oluşabilir (anahtar ayrıca gerekir) | Ağ hatasında yarım yazma riski |
| Ö2 | Aynı transaction'da yazıldığı için birlikte başarılı olur ya da birlikte başarısız olur | Tanım gereği tutarlı | Garanti yok |
| Ö3 | Tek RPC çağrısı | Okumada toplama maliyeti (kart başına küçük) | Birden fazla gidiş-dönüş |
| Ö4 | `SELECT … FOR UPDATE` ikinci işlemi bekletir ve güncel satırla devam ettirir [T10] | Ödül kontrolünde yarış durumu olabilir | Yarış durumu var |

**Seçim: A.** Dört ölçütü de karşılar. Ödül kullandırmada "bakiye ≥ gereken" kontrolü kilitli satır üzerinde yapıldığı için iki personelin aynı ödülü iki kez kullandırması engellenir (Ö4).

Transaction sınırı hakkında not: PostgreSQL belgesine göre COMMIT ve ROLLBACK yalnızca `CALL` ile çağrılan prosedürlerde ve `DO` bloklarında yapılabilir. Bir SELECT içinden çağrılan fonksiyon transaction'ı bitiremez [T9]. Buradan, fonksiyonun çağıran ifadenin transaction'ı içinde çalıştığı çıkarılmıştır (kaynaktan çıkarıldı; belge bunu açıkça yazmıyor). PostgREST'in her isteği tek transaction'a sarıp sarmadığına ilişkin belge açılamadı (HTTP 429). Bu nedenle atomiklik bir kabul testiyle doğrulanır (Bölüm 9, T-04).

### 6.2 `add_stamps(scan_token | short_code, qty, idempotency_key, confirm_duplicate)` (FR-05, FR-07, FR-08, FR-10)

1. `idempotency_key` ile kayıtlı bir olay varsa önceki sonucu döndür (yeni yazma yapılmaz).
2. Kartı `scan_token` ile bul ve `FOR UPDATE` ile kilitle. Bulunamazsa açık hata döndür.
3. Çağıranın (`auth.uid()`) kartın kafesinde `active = true` üyesi olduğunu doğrula. Değilse reddet (NFR-05).
4. `1 ≤ qty ≤ cafe.max_stamps_per_tx` kontrolü (US-03 AC-5).
5. `last_stamp_at` şu andan `duplicate_window_min` dakika öncesinden yeniyse ve `confirm_duplicate = false` ise yazmadan `needs_confirmation` döndür (FR-10).
6. `card_event(type='stamp', delta=+qty, balance_after=balance+qty)` ekle, `balance` ve `last_stamp_at` alanlarını güncelle.
7. Yeni bakiyeyi ve "ödül hazır" durumunu (`balance ≥ stamps_required`) döndür.

İstemci her kullanıcı eylemi için bir UUID üretir ve ağ hatasında aynı anahtarla yeniden dener. Yanıt alınamazsa personele "işlem doğrulanamadı, tekrar deneyin" mesajı gösterilir. Tekrar deneme aynı anahtarı kullandığı için çift sayım olmaz (NFR-06: sessiz kayıp yok). Fonksiyon `security definer`, `search_path = ''` ve tam şema adlarıyla yazılır; `EXECUTE` yetkisi yalnızca `authenticated` rolüne verilir [T4].

### 6.3 `redeem_reward(card, idempotency_key)` (FR-06, US-04)

Kilitli kartta `balance ≥ stamps_required` değilse reddedilir (US-04 AC-3). Kart doluysa `delta = −stamps_required` ile olay yazılır. Fazla damga devreder: 12 damgalı kart 10 gereken kuralda 2/10 olur (US-04 AC-2). Her çağrı tek ödül kullandırır ("Ödül 1 adet ile sınırlıdır", FR-02).

### 6.4 `reverse_event(event_id, reason)` (FR-09, US-05)

- Yalnızca `owner` rolü çağırabilir (US-05 AC-3). `reason` boş olamaz (US-05 AC-1).
- Ters işaretli `reversal` olayı yazılır. `reverses_event_id` UNIQUE olduğu için aynı olay iki kez geri alınamaz.
- Sonuç bakiye negatife düşecekse işlem reddedilir. Örnek: ödülden sonra eski bir damga geri alınmak istenirse. **Geçici karar**, BA'ya soruldu (G-05).

### 6.5 `join_cafe(join_token, marketing_opt_in)` (FR-03, FR-15, FR-16, FR-19)

Anonim oturum yoksa istemci önce `signInAnonymously()` çağırır [T1]. Fonksiyon kartı oluşturur (varsa mevcut kartı döndürür). `cafe.welcome_stamps > 0` ise `type='welcome'` olayı ve gerekçe metni ("Hoş geldin damgası") yazılır (FR-19, US-11 AC-2). Gösterilen aydınlatma sürümü `notice_version` alanına kaydedilir.

### 6.6 Kural değişikliği önizlemesi `preview_rule_change(new_required)` (FR-02)

Kaydetmeden önce sahibe şunlar gösterilir:
- etkilenen kart sayısı,
- yeni kuralla hemen "ödül hazır" olacak kart sayısı (`balance ≥ new_required`),
- ödülden uzaklaşacak kart sayısı.

**Geçici varsayılan:** Kart bakiyeleri değişmez, yeni eşik bütün kartlara uygulanır. Kesin davranış S-06'ya bağlıdır (G-04). `welcome_stamps` yeni eşiğin altında kalacak şekilde doğrulanır.

### 6.7 Göstergeler `get_metrics(cafe, period)` (FR-13, US-07)

Hepsi `card_event` üzerinden ve `Europe/Istanbul` saat dilimine göre (V-01) hesaplanır:
- verilen damga: `stamp` olaylarının toplamı eksi geri alınanlar,
- kullandırılan ödül: `redeem` sayısı eksi geri alınanlar,
- kart sahibi müşteri: dönem sonunda var olan kart sayısı,
- aktif müşteri: dönemde en az 1 net damga alan farklı kart sayısı.

Geri alınan işlemlerin göstergelerden düşülmesi geçici bir karardır (G-06). Veri hacmi küçük olduğu için `(cafe_id, created_at)` indeksiyle anlık sorgu yeterlidir; ayrı bir raporlama deposu kurulmaz.

### 6.8 Kritik karar K-5: Müşteri kimliği ve kart kurtarma

**Seçim ölçütleri:**
- Ö1: Zorunlu kişisel veri ≤ 1 alan (NFR-02, FR-03, KVKK Md. 4 [BA K11]).
- Ö2: Telefon değişiminde kartların geri alınması (FR-11, US-06 AC-1).
- Ö3: Kurtarmanın başkasının kartına erişim vermemesi (US-06 AC-2, NFR-05).
- Ö4: Katılım hızı (NFR-02).

| Ölçüt | A) Anonim hesap + isteğe bağlı e-posta bağlama | B) Zorunlu e-posta veya telefon ile kayıt | C) Hesapsız, yalnızca cihazdaki token |
|---|---|---|---|
| Ö1 | 0 zorunlu alan | 1 zorunlu alan | 0 |
| Ö2 | Yalnızca e-posta bağlayanlar kurtarabilir; anonim kullanıcı tarayıcı verisini silerse erişimi kaybeder [T1] | Herkes kurtarabilir | Kurtarma yok |
| Ö3 | Doğrulanmış e-posta ile [T1][T2] | Doğrulanmış e-posta veya SMS ile | — |
| Ö4 | En hızlı | Doğrulama adımı ekler | Hızlı |

**Seçim: A (geçici, S-04'e bağlı).** Ö1 ve Ö4'ü karşılar. Ö2'yi isteğe bağlı olarak karşılar. FR-11 ile en az veri ilkesi arasındaki çelişki BA'nın S-04 sorusudur. Kart ekranında "Kartlarını kaybetmemek için e-posta ekle" önerisi gösterilir; bu bir gereksinim değil, FR-11'i karşılayan yoldur. Manuel kimlik bağlama projede etkinleştirilmelidir ve beta durumundadır [T2] (R-04).

### 6.9 Kritik karar K-6: Silme yolu (FR-14, NFR-04)

**Seçim ölçütleri:**
- Ö1: Uygulama içinden ve web'den silme (FR-14; Google Play [BA K14]).
- Ö2: 30 gün içinde sonuç (NFR-04(b), US-08 AC-3).
- Ö3: `service_role` anahtarının istemciye çıkmaması [T6].
- Ö4: Kafenin işlem kaydı ve göstergelerinin tutarlılığı (FR-08, FR-13 AC-2).

| Ölçüt | A) Müşteri verisini tamamen silme (kart, olaylar, onay, auth kullanıcısı) | B) Kartı ve auth kullanıcısını silme; olayları müşteriden koparıp saklama |
|---|---|---|
| Ö1 | Her ikisinde de B-07 ve B-08 ile | Aynı |
| Ö2 | Uygulama içi silme anında; web talebi B-09 takibiyle | Aynı |
| Ö3 | Edge Function içinde | Aynı |
| Ö4 | Kafenin geçmiş göstergeleri değişir (silinen olaylar sayılmaz) | Geçmiş göstergeler korunur, ancak verinin gerçekten anonim sayılıp sayılmayacağı hukuki bir sorudur |

**Seçim: A (geçici).** Ö1–Ö3'te eşit oldukları için hukuki belirsizliği olmayan seçenek seçilmiştir. Ö4 etkisi BA'ya sorulmuştur (G-07). Uygulama içi silme akışı:
1. Kullanıcı onay verir.
2. B-07 JWT'den kullanıcıyı doğrular.
3. Uygulama tablolarındaki satırlar tek transaction'da silinir.
4. `deleteUser` çağrılır [T6].

Web talebinde kimlik, kartın kısa kodu veya bağlı e-postaya gönderilen doğrulama koduyla tespit edilir (G-08). Kafe sahibinin silme talebi kafeyi, üyelikleri, kartları ve olayları zincirleme siler.

---

## 7. Performans

| Hedef | Tasarım tedbiri | Ölçüm |
|---|---|---|
| NFR-01: kart tanıma → onay ≤ 3 sn (%95) | Tarama tek RPC çağrısıdır. QR çözme WebWorker'da çalışır ve varsa yerel `BarcodeDetector` kullanılır [T13]. Personel ekranı kamerayı açık tutar. Kafe ve kart verisi önceden yüklenir. | İstemci, QR çözüldüğü andan RPC yanıtına kadar geçen süreyi kaydeder. Gerçek kafe Wi-Fi'ı ve mobil veriyle en az 100 işlemlik testte %95'lik değer ölçülür (T-08). |
| NFR-02: katılım ≤ 60 sn, ≤ 3 ekran | 2 ekranlı akış (Bölüm 4). Anonim giriş arka planda yapılır. PWA kabuğu küçük tutulur. | Kullanılabilirlik testinde süre ve ekran sayısı ölçülür (T-09). |
| FR-13: göstergeler | İndeksli toplama sorgusu | Örnek veriyle (ör. 1 yıllık, kafe başına binlerce olay) sorgu süresi ölçülür. |

Ağ gecikmesi için rakamsal bir öngörü yazılmamıştır; Frankfurt bölgesine Türkiye'den gidiş-dönüş süresi bu dokümanda kaynakla doğrulanmadı. Hedef, T-08 ölçümüyle doğrulanır. Ölçüm hedefi karşılamazsa ilk adım bölge kararının (K-2) yeniden ele alınmasıdır.

---

## 8. Güvenlik ve gizlilik

- **Yetki (NFR-05, FR-07):**
  - Tüm tablolarda RLS açıktır; politikasız tabloda erişim reddedilir [T5].
  - Müşteri yalnızca kendi kartlarını ve olaylarını okur.
  - Kafe üyesi yalnızca kendi kafesinin kayıtlarını okur; personel kaydı görmez, yalnızca tarar (FR-08 kayıt görüntüleme sahibe aittir).
  - Bütün yazmalar, yetkiyi gövdede denetleyen `security definer` fonksiyonlarla yapılır. Fonksiyonlarda `search_path = ''` kullanılır [T4].
  - Yetkisi kaldırılan personelin (`active = false`) sonraki çağrıları reddedilir (US-03 AC-2).
- **Anonim kayıt kötüye kullanımı:** Supabase, anonim girişte CAPTCHA'yı "strongly recommended" olarak belirtir [T1]. CAPTCHA bir üçüncü taraf hizmettir ve KVKK açısından veri aktarımı doğurur. Etkinleştirme kararı G-09'a bağlıdır. IP başına saatlik varsayılan sınır kafe Wi-Fi'ında sorun yaratabilir (R-03).
- **Taşıma şifrelemesi (NFR-05):** Uygulama yalnızca HTTPS ile sunulur. Kamera erişimi zaten güvenli bağlam gerektirir [T16].
- **Token tasarımı:** `scan_token` tahmin edilemez 128 bit rastgele değerdir ve sıralı kimlik içermez. `short_code` yalnızca yetkili personel oturumunda ve hız sınırıyla kullanılabilir.
- **Gizli anahtarlar:** `service_role` anahtarı yalnızca Edge Function secret'ında tutulur [T6][T8].
- **Gizlilik (NFR-04, FR-15):** Veri envanteri Bölüm 5'tedir. Aydınlatma metni sürümlüdür ve katılım ekranından, sonra hesap ayarlarından erişilebilir. Metnin içeriği ve veri sorumlusu S-09'a bağlıdır.
- **Kampanya onayı (FR-16):** Varsayılan olarak işaretsizdir (US-02 AC-4). Geri alma tek dokunuşla yapılır ve `revoked_at` yazılır. Gönderim kanalı MVP'de yoktur (G-10).
- **İhlal bildirimi (NFR-04(c)):** Bu teknik bir bileşen değil, bir süreç belgesidir. Teknik katkı olarak işlem kayıtları ve Supabase erişim günlükleri inceleme için saklanır. Süreç sahibi S-09'a bağlıdır.

---

## 9. Test stratejisi

| ID | Test | Tür | Gereksinim |
|---|---|---|---|
| T-01 | Kural doğrulama: damga sayısı 0, 21 ve boş ad reddedilir | Birim (SQL CHECK + arayüz) | FR-01, FR-02, US-01 AC-1/2 |
| T-02 | Yetki matrisi: anonim, müşteri, başka kafenin personeli, yetkisi kaldırılmış personel ve sahip için her RPC'nin sonucu | Veritabanı entegrasyon testi | NFR-05 (%0 başarı), FR-07, US-03 AC-2, US-05 AC-3 |
| T-03 | Aynı idempotency anahtarıyla 2 çağrı → tek olay | Entegrasyon | NFR-06 |
| T-04 | Fonksiyon ortasında hata üretilir → ne olay ne bakiye değişir (atomiklik) | Entegrasyon | NFR-06 |
| T-05 | Eşzamanlı 2 `redeem_reward` aynı dolu kartta → biri başarılı, biri reddedilir | Eşzamanlılık | NFR-06, FR-06 |
| T-06 | Her testten sonra `balance = SUM(delta)` denetimi → 0 tutarsızlık | Değişmez kural kontrolü | NFR-06, FR-08 |
| T-07 | Devretme (12 → 2), dolmamış kartta ödül reddi, FR-10 uyarısı (5 dk içinde) | Birim | FR-06, FR-10, US-04 |
| T-08 | Gerçek cihazlarda (iOS Safari ve Android Chrome) 100 tarama süresi, %95 ≤ 3 sn | Performans | NFR-01, NFR-10 |
| T-09 | Kullanılabilirlik: yeni personel 1 dk açıklamayla ilk damga (≥ %90); sahip kurulumu ≤ 10 dk; müşteri katılımı ≤ 60 sn | Kullanılabilirlik | NFR-02, NFR-08 |
| T-10 | Kontrast ≥ 4.5:1 ve dokunma hedefi ≥ 24×24 için otomatik ve manuel denetim | Erişilebilirlik | NFR-03 |
| T-11 | Uygulama içi silme ve web talebi: satırlar ve auth kullanıcısı silinir; talep 30 günlük takipte görünür | Entegrasyon | FR-14, NFR-04 |
| T-12 | Gösterge değerleri olay kaydından bağımsız hesaplanan değerlerle birebir aynı | Entegrasyon | FR-13, US-07 AC-2 |
| T-13 | E-posta bağlanmış hesapla yeni cihazdan giriş → tüm kartlar aynı sayılarla görünür; başka e-postayla erişim yok | Uçtan uca | FR-11, US-06 |
| T-14 | Kamera izni reddedildiğinde kısa kod yolu çalışır | Uçtan uca | FR-05, NFR-08 |
| T-15 | Tüm metinlerin Türkçe olduğu ekran incelemesi | Manuel | NFR-09 |

---

## 10. Riskler

| ID | Risk | Etki | Önlem | Bağlı |
|---|---|---|---|---|
| R-01 | S-03 yanıtı "yerel uygulama zorunlu" olursa PWA kararı geçersiz kalır | Platform değişir; NFR-11 kontrol listesi devreye girer | Veritabanı ve iş kuralları istemciden bağımsız tasarlandı; yalnızca istemci değişir | K-1, NFR-11 |
| R-02 | Veriler AB'de işlenir (Türkiye bölgesi yok [T3]). Edge Function istekleri kullanıcıya en yakın düğümde çalışır [T8]. Bu nedenle işleme konumu tek bir bölgeyle sınırlı değildir. | KVKK Md. 9 yükümlülükleri (S-09) | Proje açılmadan S-09 netleştirilir. Gerekirse K-2'de C seçeneğine geçilir. | K-2, NFR-04 |
| R-03 | Anonim girişte IP başına saatte 30 istek varsayılan sınırı var [T1]. Aynı kafe Wi-Fi'ını kullanan müşteriler aynı IP'yi paylaşabilir. | Yoğun saatte katılım engellenir (NFR-02) | Sınır panelden artırılır [T1]; CAPTCHA ile birlikte değerlendirilir (G-09) | NFR-02, NFR-05 |
| R-04 | Manuel kimlik bağlama beta durumunda [T2] | FR-11 akışı değişebilir | T-13 her sürüm yükseltmesinde tekrarlanır | FR-11 |
| R-05 | Anonim kullanıcı tarayıcı verisini silerse veya cihaz değiştirirse e-posta bağlamadıysa kartlarını kaybeder [T1] | FR-11 kısmen karşılanır | Kart ekranında e-posta ekleme önerisi; S-04 kararı | FR-11, NFR-02 |
| R-06 | iOS'ta Web Push yalnızca ana ekrana eklenmiş web uygulamasında ve iOS 16.4 ve sonrasında çalışır [T17] | FR-17 bildirimi bazı kullanıcılara ulaşmaz | FR-17'de uygulama içi gösterim zorunlu, bildirim isteğe bağlı; tasarım bununla uyumlu | FR-17 |
| R-07 | FR-18 (Wallet) için Apple tarafında ayrı imzalama ve güncelleme altyapısı, Google tarafında Issuer hesabı gerekir [BA K8, K9] | MVP kapsamını büyütür | FR-18 (C) bu sürümde ertelendi. Veri modeli (`card.balance`) pass güncellemesine kaynak olabilecek şekilde tutuldu | FR-18 |
| R-08 | `BarcodeDetector` sınırlı destekli ve deneysel [T15]. qr-scanner kamera tercihini garanti etmez [T13]. | Bazı cihazlarda tarama yavaşlayabilir | qr-scanner'ın kendi çözücüsü ve kısa kod yolu; T-08 gerçek cihazlarda | NFR-01 |
| R-09 | Yönetimden oluşturulan `security definer` fonksiyonları RLS'yi atlar [T4] | Yanlış yazılmış bir fonksiyon yetki açığı yaratır | Yetki kontrolü her fonksiyonun ilk adımı; T-02 yetki matrisi zorunlu | NFR-05 |
| R-10 | Anonim kullanıcıların otomatik temizliği yok [T1] | Kullanılmayan hesaplar birikir; en az veri ilkesine aykırı olabilir | Saklama süresi BA'ya soruldu (G-11) | NFR-04 |
| R-11 | PostgREST transaction belgesi doğrulanamadı (HTTP 429) | Atomiklik varsayımı belgeyle desteklenmedi | T-04 ile test edilerek doğrulanır | NFR-06 |

---

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Kafe kaydı: ad, logo, açıklama | B-03, `cafe` tablosu | 4, 5 |
| FR-02 | Kural: 1–20 damga, ödül metni, değişiklik etkisinin gösterilmesi | `cafe` CHECK kısıtları, `preview_rule_change` (geçici, S-06) | 5, 6.6 |
| FR-03 | QR veya bağlantıyla, en az veriyle katılım | K-1 PWA, K-5 anonim hesap, `join_cafe` | 3.1, 6.5, 6.8 |
| FR-04 | Kartta n/N ve ödül durumu | B-01 kart ekranı, `card.balance` | 4, 5 |
| FR-05 | Yetkili personelin çoklu damga vermesi, üst sınır | K-3 Y-1, qr-scanner, `add_stamps` | 3.3, 6.2 |
| FR-06 | Ödül kullandırma, sıfırlama ve devretme | `redeem_reward` | 6.3 |
| FR-07 | Personel ekleme ve kaldırma; yalnızca yetkili işlem | `cafe_member`, RLS, fonksiyon içi yetki kontrolü | 5, 8 |
| FR-08 | İşlem kaydı; sahip ve müşteri görüntüler | `card_event` (append-only), RLS okuma politikaları | 5, 6.1, 8 |
| FR-09 | Gerekçeli geri alma | `reverse_event` (yalnızca sahip) | 6.4 |
| FR-10 | Süre içinde ikinci damgada uyarı ve onay | `add_stamps` 5. adım, `duplicate_window_min` | 6.2 |
| FR-11 | Telefon değişiminde kartların geri alınması | K-5: isteğe bağlı e-posta bağlama (geçici, S-04) | 6.8 |
| FR-12 | Birden çok kafenin kartı tek yerde | Çok kafeli tek PWA, B-01 kart listesi | 2, 4 |
| FR-13 | Dönemsel göstergeler | `get_metrics` | 6.7 |
| FR-14 | Uygulama içi ve web üzerinden silme; kafe silme | K-6, B-07, B-08, B-09 | 6.9 |
| FR-15 | Katılımda ve sonrasında aydınlatma metni | B-08, `privacy_notice`, `card.notice_version` | 5, 8 |
| FR-16 | Kampanya için ayrı, isteğe bağlı onay ve geri alma | `marketing_consent`, işaretsiz onay kutusu | 5, 8 |
| FR-17 | "Ödül hazır" gösterimi, isteğe bağlı bildirim | B-01 uygulama içi gösterim; Web Push isteğe bağlı | 3.4, 10 (R-06) |
| FR-18 | Wallet'a ekleme ve güncel sayı | Bu sürümde ertelendi; veri modeli hazır | 10 (R-07) |
| FR-19 | Başlangıç damgası | `cafe.welcome_stamps`, `join_cafe` | 5, 6.5 |
| NFR-01 | Damga ≤ 3 sn (%95) | Tek RPC, worker'da QR çözme, T-08 | 6.2, 7 |
| NFR-02 | Katılım ≤ 60 sn, ≤ 3 ekran, ≤ 1 zorunlu alan | 2 ekranlı akış, anonim hesap | 4, 6.8, 7 |
| NFR-03 | WCAG 2.2 AA kontrast ve dokunma hedefi | Tasarım kuralı ve T-10 | 9 |
| NFR-04 | KVKK: veri envanteri, 30 gün, ihlal süreci | Bölüm 5 envanteri, `deletion_request.due_at`, bölge kararı (S-09) | 5, 6.9, 8 |
| NFR-05 | Yalnızca yetkili işlem; şifreli aktarım | RLS, `security definer` + yetki kontrolü, HTTPS, T-02 | 8, 9 |
| NFR-06 | Kayıp ve çift sayım yok; açık hata | K-4: kilit + idempotency + aynı transaction | 6.1, 6.2, 9 |
| NFR-07 | Ek donanım gerekmez | Personel telefon kamerası (Y-1) | 3.3 |
| NFR-08 | Öğrenilebilirlik | Tek düğmeli personel ekranı, kısa kod yedeği, T-09 | 4, 9 |
| NFR-09 | Türkçe arayüz ve metin | Tek dilli arayüz, T-15 | 9 |
| NFR-10 | iOS ve Android | PWA; en düşük sürüm önerisi G-12'de | 3.1, 12 |
| NFR-11 | Mağaza uyumu (mağaza yolu seçilirse) | K-1: mağaza yolu seçilmedi; R-01 durumunda kontrol listesi uygulanır | 3.1, 10 |

---

## 12. BA'ya geri sorular

| ID | Soru | İlgili gereksinim / BA sorusu | Geçici karar |
|---|---|---|---|
| G-01 | S-03 yanıtlandı mı? PWA (indirmesiz) kabul edilebilir mi? | FR-03, NFR-02, NFR-11 / S-03 | PWA |
| G-02 | S-01 yanıtı "çok kafeli tek uygulama" mı? | FR-12, NFR-11 / S-01 | Çok kafeli tek uygulama |
| G-03 | S-09: Veriler AB'de (Frankfurt) işlenebilir mi, yoksa Türkiye'de tutulması şart mı? Veri sorumlusu kim? | NFR-04, FR-15 / S-09 | AB bölgesi; proje açılmadan kesinleşmeli |
| G-04 | S-06: Kural değişince mevcut kartların bakiyesi korunup yeni eşik mi uygulanacak? Damga veya ödül süre sonu olacak mı? | FR-02, FR-06 / S-06 | Bakiye korunur, yeni eşik uygulanır, süre sonu yok |
| G-05 | Geri alma bakiyeyi negatife düşürecekse (ör. ödülden sonra eski damganın geri alınması) ne olmalı? | FR-09, NFR-06 | Reddedilir |
| G-06 | FR-13 göstergelerinde geri alınan damga ve ödüller düşülmeli mi? "Aktif müşteri" net damgaya göre mi sayılmalı? | FR-13 | Düşülür; net damga |
| G-07 | Müşteri hesabını silince kafenin geçmiş işlem kaydı ve göstergeleri değişebilir mi, yoksa olaylar müşteriden koparılıp saklanmalı mı? | FR-14, FR-08, FR-13 / S-09 | Tamamen silinir |
| G-08 | Web üzerinden silme talebinde anonim müşterinin kimliği nasıl doğrulanmalı (kartın kısa kodu, bağlı e-posta)? | FR-14, US-08 AC-2 | Kısa kod veya bağlı e-posta |
| G-09 | Anonim katılımda üçüncü taraf CAPTCHA (hCaptcha / Turnstile) kullanımı KVKK ve aydınlatma metni açısından kabul edilebilir mi? | NFR-05, FR-15 | Yalnızca IP hız sınırı |
| G-10 | FR-16 için kampanya mesajı hangi kanalla gönderilecek (uygulama içi, bildirim, e-posta)? MVP'de yalnızca onay toplamak yeterli mi? | FR-16 / S-07 | Yalnızca onay kaydı |
| G-11 | Kullanılmayan anonim hesaplar ve kartlar ne kadar süre saklanmalı? | NFR-04 | Belirlenmedi |
| G-12 | NFR-10 için en düşük sürüm önerisi: iOS 16.4 (Web Push eşiği [T17]) ve güncel Android Chrome. Bu, hedef kitle için kabul edilebilir mi? | NFR-10, FR-17 | iOS 16.4+, Android Chrome güncel |
| G-13 | FR-18 (Wallet) MVP sonrasına ertelenebilir mi? | FR-18 | Ertelendi |
| G-14 | S-10: Bağlantı yokken damga verme gerekiyor mu? Tasarım şu an çevrimiçi işlem ve açık hata mesajı varsayar. | NFR-06 / S-10 | Çevrimdışı damga yok |
| G-15 | S-05: Y-1 (personelin müşteri QR'ını taraması) iş tercihine uygun mu? | FR-05, FR-10 / S-05 | Y-1 |

---

## 13. Kaynaklar

| No | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| T1 | Supabase Docs: Anonymous Sign-Ins | https://supabase.com/docs/guides/auth/auth-anonymous | `signInAnonymously()` kişisel veri olmadan kullanıcı oluşturur. Anonim kullanıcı "can't access their account if they sign out, clear browsing data" (veya başka cihaz kullanırsa). E-posta veya OAuth kimliği bağlanarak kalıcı kullanıcıya dönüşür. "Automatic cleanup of anonymous users is currently not available." "An IP-based rate limit is enforced at 30 requests per hour" (panelden değiştirilebilir). CAPTCHA "strongly recommended". `is_anonymous` JWT iddiası. |
| T2 | Supabase Docs: Identity Linking | https://supabase.com/docs/guides/auth/auth-identity-linking | Manuel bağlama projede etkinleştirilmeli; beta olarak işaretli. Ayırma için en az 2 kimlik gerekir. Self-hosting için ortam değişkeni anılıyor. |
| T3 | Supabase Docs: Regions | https://supabase.com/docs/guides/platform/regions | AB bölgeleri (Frankfurt `eu-central-1` dahil); Türkiye bölgesi listelenmiyor. "Once you select a region for your project, you cannot change it." |
| T4 | Supabase Docs: Database Functions | https://supabase.com/docs/guides/database/functions | `rpc()` ile çağrı. "When you use `security definer`, you must set the `search_path`." Panel veya migration ile oluşturulan fonksiyonun sahibi `postgres`'tir ve RLS'yi atlar. Boş `search_path` ve çalıştırma yetkisinin daraltılması önerilir. Transaction kapsamı bu sayfada yazmıyor. |
| T5 | Supabase Docs: Row Level Security | https://supabase.com/docs/guides/database/postgres/row-level-security | "Enable RLS on every table in an exposed schema." Politika yoksa API üzerinden veri erişilemez. Oturumsuz istekte `auth.uid()` null döner. |
| T6 | Supabase JS Reference: auth.admin.deleteUser | https://supabase.com/docs/reference/javascript/auth-admin-deleteuser | "Requires a `service_role` key"; "This function should only be called on a server". Soft delete geri alınamaz. Diğer tablolardaki satırlar belgede anılmıyor. |
| T7 | Supabase Docs: CAPTCHA | https://supabase.com/docs/guides/auth/auth-captcha | hCaptcha ve Cloudflare Turnstile desteği; sign-in, sign-up ve parola sıfırlama formları; yerel test için alan adı ayarı. |
| T8 | Supabase Docs: Edge Functions | https://supabase.com/docs/guides/functions | Deno tabanlı TypeScript fonksiyonları. İstekler "regionally-distributed Edge Runtime node closest to the user" üzerinde çalışır. Soğuk başlatma olabilir. Kimlik bilgileri secret ve ortam değişkeninde tutulur. |
| T9 | PostgreSQL Docs: Transaction Management (PL/pgSQL) | https://www.postgresql.org/docs/current/plpgsql-transactions.html | COMMIT ve ROLLBACK yalnızca `CALL` ile çağrılan prosedürlerde ve `DO` bloklarında. Araya SELECT giren çağrı zincirinde transaction kontrolü yapılamaz. Fonksiyonun çağıranın transaction'ında çalıştığı buradan çıkarılmıştır. |
| T10 | PostgreSQL Docs: Transaction Isolation | https://www.postgresql.org/docs/current/transaction-iso.html | "Read Committed is the default isolation level." Eşzamanlı güncellemede ikinci güncelleyici bekler, ardından WHERE koşulunu güncel satır üzerinde yeniden değerlendirir. `SELECT FOR UPDATE` güncel satırı kilitler ve döndürür. |
| T11 | Firebase Docs: Firestore Transactions | https://firebase.google.com/docs/firestore/manage-data/transactions | Çakışmada "Cloud Firestore runs the entire transaction again"; "Transactions will fail when the client is offline"; okumalar yazmalardan önce yapılmalı. |
| T12 | Firebase Docs: Firestore Locations | https://firebase.google.com/docs/firestore/locations | Avrupa konumları (`eur3` ve bölgesel konumlar); Türkiye yok; konum sonradan değiştirilemez. |
| T13 | GitHub: nimiq/qr-scanner | https://github.com/nimiq/qr-scanner | ZXing tabanlı. "Uses the browser's native BarcodeDetector if available"; WebWorker'da çalışır. "your page must be served via HTTPS"; kamera izni sorulur. ES2017 ve legacy derleme. MIT. Kamera tercihi garanti değil. |
| T14 | GitHub: soldair/node-qrcode | https://github.com/soldair/node-qrcode | Tarayıcıda canvas, SVG ve data URL üretimi. Hazır paket "Internet Explorer 10+, Safari 5.1+, and all evergreen browsers". L/M/Q/H düzeltme seviyeleri. MIT. ECI desteği tam değil. |
| T15 | MDN: Barcode Detection API | https://developer.mozilla.org/en-US/docs/Web/API/Barcode_Detection_API | "Limited availability" ve "Experimental". Güvenli bağlam gerekir. Uyumluluk tablosu sayfa özetinde okunamadı. |
| T16 | MDN: MediaDevices.getUserMedia() | https://developer.mozilla.org/en-US/docs/Web/API/MediaDevices/getUserMedia | Yalnızca güvenli bağlamda kullanılabilir. "user permission is always required". Promise hiç sonuçlanmayabilir. |
| T17 | WebKit Blog: Web Push for Web Apps on iOS and iPadOS | https://webkit.org/blog/13878/web-push-for-web-apps-on-ios-and-ipados/ | "with iOS and iPadOS 16.4, we are adding support for Web Push to Home Screen web apps". İzin isteği "in response to direct user interaction" yapılmalı. |

BA dokümanındaki kaynaklara [BA Kn] biçiminde atıf yapılmıştır; bu kaynaklar bu oturumda yeniden açılmamıştır.

**Açılamayan veya kullanılmayan kaynaklar:**
- PostgREST Docs, Transactions (`docs.postgrest.org`, stable ve v12): HTTP 429 hatası verdi. Her RPC isteğinin tek transaction'da çalıştığı belgeyle doğrulanamadı (R-11, T-04).
