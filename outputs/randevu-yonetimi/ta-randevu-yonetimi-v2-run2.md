# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ta v2
- **Girdi:** outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run2.md

---

## 1. Özet

Ürün tek bir salona ait bir mobil uygulamadır (FR-05, V-01). Müşteri, çalışan ve yönetici aynı Flutter uygulamasını rol bazlı ekranlarla kullanır (FR-04, NFR-01). Arka uçta yalnızca Firebase servisleri kullanılır:
- Kimlik doğrulama için Firebase Authentication.
- Veri için Cloud Firestore.
- İş kuralları için Cloud Functions (TypeScript, Node.js 22).
- Bildirim için Firebase Cloud Messaging (FCM).
- Gizlilik metinleri ve web üzerinden hesap silme sayfası için Firebase Hosting.

Ayrı bir sunucu, kuyruk servisi veya yönetim paneli yoktur.

Temel kararlar:
- **Randevu yazma yolu (K-2):** Randevu oluşturan, değiştiren veya silen tüm işlemler istemciden doğrudan Firestore'a yazılmaz. Bu işlemler çağrılabilir (callable) Cloud Functions üzerinden yapılır. Böylece çift rezervasyon kontrolü (FR-11, NFR-02) tek yerde yapılır. Çevrimdışı yazmalar da kuyruğa girmez (NFR-13).
- **Çakışma önleme (K-3):** Her çalışan ve gün için tek bir "gün takvimi" belgesi (`staffDays`) tutulur. Randevu bu belge üzerinde bir Firestore işlemiyle (transaction) eklenir.
- **Müsait saat üretimi (K-4):** Başlangıç saatleri sabit bir ızgarada üretilir (geçici olarak 15 dk, AS-17). Uygunluk ise hizmetin gerçek süresine ve tampon süreye göre kontrol edilir. Hesap yalnızca sunucuda yapılır.
- **Hatırlatma (K-5):** Dakikada bir çalışan zamanlanmış bir fonksiyon, zamanı gelen hatırlatmaları gönderir. Cloud Tasks seçilmedi, çünkü en fazla 30 gün ileriye görev planlanabiliyor. Randevu ise 60 gün sonrasına alınabiliyor (FR-22).
- **Hesap silme (K-6):** Kullanıcı hesabı ve profili silinir. Gelecekteki randevular iptal edilir. Geçmiş randevu kayıtları kişisel alanları temizlenip "Hesap silindi" etiketiyle tutulur (FR-03, FR-29, NFR-14).

Proje oluşturulmadan önce yanıtlanması gereken iki konu var:
- **Firestore konumu sonradan değiştirilemiyor.** Listede Türkiye bölgesi de yok [T-11]. Bu nedenle KVKK yurt dışı aktarım sorusunun (AS-08) proje kurulmadan önce yanıtlanması gerekir (Soru S-01).
- **Cloud Functions, kullandıkça öde (Blaze) planını gerektiriyor [T-14].** Fatura sahibinin belirlenmesi gerekir (Soru S-15).

## 2. Mimari genel bakış

```
┌──────────────────────── Flutter uygulaması (Android + iOS) ────────────────────────┐
│ Müşteri ekranları │ Çalışan ekranları │ Yönetici ekranları   (rol: users/{uid}.role) │
│  - Firestore okuma (canlı dinleyici + çevrimdışı önbellek)                           │
│  - Yazma işlemleri yalnızca callable fonksiyonlarla                                  │
│  - FCM jetonu kaydı, bildirim izni (bağlamında)                                      │
└───────────────┬───────────────────────────────┬─────────────────────────────────────┘
                │ okuma (Security Rules)        │ callable (Auth jetonu otomatik)
                ▼                               ▼
        ┌──────────────┐              ┌──────────────────────────────┐
        │ Cloud        │◄────────────►│ Cloud Functions (Node.js 22, │
        │ Firestore    │  Admin SDK   │ TypeScript, tek bölge)       │
        └──────────────┘              │  - getAvailableSlots         │
                ▲                     │  - createAppointment / ...   │
                │                     │  - deleteAccount             │
                │                     │  - reminderTick (her dakika) │
                │                     └──────────────┬───────────────┘
                │                                    │ FCM gönderimi
┌───────────────┴──────────┐                         ▼
│ Firebase Hosting          │                 Cihazlar (FCM / APNs)
│ - Gizlilik politikası,    │
│   aydınlatma metni        │
│ - Web hesap silme sayfası │── Firebase JS SDK (Auth + Functions) ──► deleteAccount
└───────────────────────────┘
```

**Ürün büyüklüğüne uygunluk:** Tek salon, az sayıda çalışan ve tek mobil uygulama söz konusu. Mikroservis, ayrı API sunucusu, mesaj kuyruğu veya analitik altyapısı önerilmez (kapsam kuralları). Tablet ve masaüstü panel kapsam dışıdır (BA Bölüm 7). Bu nedenle yönetici ekranları da mobil uygulamanın içindedir.

**Tek uygulama ve rol bazlı arayüz (FR-04):** Müşteri ve personel için ayrı uygulama yayımlamak mağaza ve bakım yükünü ikiye katlar. Rol, Firestore'daki kullanıcı belgesinden okunur. Arayüz bu role göre açılır. Yetki kontrolü ise arayüzde değil, Security Rules ve fonksiyonlarda yapılır (NFR-06).

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Seçim tablosu

Flutter ve Firebase görevde verilmiş kısıtlardır (V-02). Aşağıdaki tablo, bunların hangi bileşenlerinin ve hangi ek kütüphanelerin kullanılacağını gösterir. "İzinler", "En düşük sürüm" ve "Bilinen kısıtlar" sütunları ilgili belgeden alınmıştır. Belgede yazmayan bilgi "belgede yok" olarak işaretlenmiştir.

| Teknoloji / paket | Kullanım ve gerekçe | İzinler | En düşük platform sürümü | Bilinen kısıtlar (belgeye göre) | Gereksinim |
|---|---|---|---|---|---|
| Flutter (3.50 belgesine göre) | Tek kod tabanıyla Android ve iOS. Görev kısıtı | Belgede yok | Android: "Supported 24 to 37", 23 ve öncesi desteklenmiyor. iOS: "Supported 15 to 27", 14 ve öncesi desteklenmiyor [T-10] | Android'de API 23 ve altı, iOS'ta 14 ve altı desteklenmiyor | NFR-01 |
| `firebase_core` 5.2.0 | Firebase başlatma | Belgede yok | Firebase Android: Android 7.0 ve üzeri ("Uses Android 7.0 or higher", API 24) [T-8]. Firebase Apple: "iOS 15" [T-9] | Yayıncı: firebase.google.com (doğrulanmış) [T-21] | NFR-01 |
| `firebase_auth` 7.1.0 | Kayıt, giriş, şifre sıfırlama (`sendPasswordResetEmail()`), yeniden kimlik doğrulama [T-20, T-22] | Belgede yok | Firebase tabanıyla aynı [T-8, T-9] | Hesap silme gibi hassas işlemlerde "requires-recent-login" hatası verir, yeniden kimlik doğrulama gerekir [T-22] | FR-01, FR-03 |
| `cloud_firestore` 7.2.0 | Veri okuma, canlı dinleyici, çevrimdışı önbellek [T-18] | Belgede yok | Firebase tabanıyla aynı | Mobilde çevrimdışı kalıcılık varsayılan olarak açıktır. Çevrimdışı yazmalar yerelde tutulup bağlantı gelince senkronize edilir [T-12]. Bu nedenle randevu yazmaları istemciden yapılmaz (K-2) | FR-14, FR-15, NFR-13 |
| `cloud_functions` 7.1.0 | Callable fonksiyon çağrıları [T-19] | Belgede yok | Firebase tabanıyla aynı | İstemci SDK'sında varsayılan zaman aşımı 70 sn [T-25]. Çevrimdışı davranış belgede yok (Risk R-06) | FR-09–FR-13, FR-21, FR-03 |
| `firebase_messaging` 17.1.0 | Push bildirimi, izin isteği [T-5] | Android 13+: `POST_NOTIFICATIONS` çalışma zamanı izni (BA K-12). iOS: Xcode'da "push notifications and background modes" açılır, APNs kimlik doğrulama anahtarı Firebase'e yüklenir [T-6] | FCM istemcisi: "Android 7.0 or higher that also have Google Play services installed" [T-6]. iOS en düşük sürüm belgede yok, Firebase tabanı iOS 15 [T-9] | Google Play hizmetleri olmayan cihazda FCM çalışmaz [T-6]. Ön plandaki bildirim mesajı Android ve iOS'ta varsayılan olarak görünür bildirim göstermez [T-7]. iOS'ta API çağrılarından önce APNs jetonu gerekir [T-6]. UIScene yaşam döngüsünde `configureNotificationCenterDelegate()` çağrısı gerekir [T-5] | FR-16–FR-19 |
| Cloud Firestore (sunucu) | Belge veritabanı (karşılaştırma K-1) | — | — | Konum sonradan değiştirilemez, Türkiye bölgesi yok [T-11]. Kurallarda belge erişimi tek istekte 10, işlem ve toplu yazmada 20 çağrıyla sınırlı [T-24]. İşlem kilit süresi 20 sn [T-1] | FR-11, NFR-02, NFR-06 |
| Cloud Functions for Firebase (Node.js 22, TypeScript) | Randevu iş kuralları, bildirim, hatırlatma, hesap silme | — | Node.js 22 ve 20 destekleniyor, 18 kullanımdan kaldırılmış [T-23] | Blaze planı gerekir [T-14]. Soğuk başlatma `minInstances` ile azaltılabilir, ama boşta çalışan örnekler de ücretlendirilir [T-23] | FR-09–FR-13, FR-18, FR-19, NFR-03 |
| `firebase-functions`, `firebase-admin` (npm) | Fonksiyon tanımları ve Admin SDK | — | Sürüm doğrulanamadı (npm sayfası 403 döndü) | Kuruluma girerken sürüm `package.json`'a sabitlenir (Risk R-08) | Yukarıdakiyle aynı |
| `luxon` (npm, MIT lisanslı) [T-17] | Salonun IANA saat dilimiyle (`Europe/Istanbul`) yerel saat ve UTC dönüşümü | Yok (sunucu tarafı) | Belgede yok. Intl API'ye dayanır ("abusing built-in Intl APIs") [T-16] | Desteklenmeyen bölge geçersiz DateTime döndürür. Yaz saati boşluğundaki saat bir saat ileri kaydırılır. Tekrarlanan saatte davranış "undefined" [T-16] (Risk R-04). Sürüm doğrulanamadı (npm 403) | FR-09, NFR-10 |
| Firebase Hosting | Gizlilik politikası, aydınlatma metni, web üzerinden hesap silme sayfası. "Zero-configuration SSL" [T-26] | — | — | Statik içerik. Dinamik iş Cloud Functions ile yapılır [T-26] | FR-02, FR-03 |
| Firebase JS SDK (modüler; yalnızca Auth ve Functions) | Web silme sayfasında giriş ve `deleteAccount` çağrısı [T-27] | Belgede yok | Auth ve Functions; Edge, Firefox, Chrome, iOS Safari ve Safari'de destekleniyor [T-28] | Bazı ortamlarda telefonla kimlik doğrulama ve açılır pencere veya yönlendirmeli OAuth desteklenmiyor. İçerik güvenlik politikası (CSP) varsa gerekli betiklere izin verilmeli [T-28] | FR-03 |
| Cloud Scheduler (zamanlanmış fonksiyon aracılığıyla) | Dakikada bir hatırlatma taraması | — | — | İş başına 0,10 $/ay. Google hesabı başına 3 iş ücretsiz [T-2] | FR-18 (c), FR-24, NFR-04 |
| `flutter_localizations` (Flutter SDK ile gelir) | Türkçe arayüz metinleri ve takvim gün adları | — | Flutter tabanı | Dış paket değildir. Ayrı belge kontrolü yapılmadı | NFR-10 |

**Bilerek eklenmeyen paketler:**
- Takvim widget paketi eklenmedi. FR-15'teki günlük ve haftalık görünümler basit liste ve sütun düzeniyle yazılır, böylece bağımlılık sayısı düşük kalır.
- Yerel bildirim paketi eklenmedi. Ön planda gelen mesaj, uygulama içi bir şeritle gösterilir (FR-17).
- İzin yönetim paketi eklenmedi. İzin isteği `firebase_messaging` ile yapılır.
- SMS sağlayıcısı (FR-27) ve takvime ekleme paketi (FR-28) seçilmedi. İkisi de C öncelikli ve AS-06'ya bağlı, karar ertelendi.

### 3.2 Platform tabanı (NFR-01, AS-07)
Teknik taban **Android 7.0 (API 24)** ve **iOS 15**'tir. Flutter 3.50 [T-10], Firebase Android [T-8], Firebase Apple [T-9] ve FCM istemcisi [T-6] daha düşük sürümleri desteklemiyor. İş açısından bu tabanın kabul edilip edilmediği BA'ya sorulur (S-03).

### 3.3 Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| T-1 | Firestore: Transactions and batched writes | https://firebase.google.com/docs/firestore/manage-data/transactions | "Transactions will fail when the client is offline." Eşzamanlı düzenlemede "Cloud Firestore runs the entire transaction again". Kilit süresi 20 sn, en fazla 270 sn |
| T-2 | Schedule functions | https://firebase.google.com/docs/functions/schedule-functions | `onSchedule`. Crontab ve App Engine söz dizimi. "each Cloud Scheduler job costs $0.10 (USD) per month". "allowance of three jobs per Google account, at no charge" |
| T-3 | Enqueue functions with Cloud Tasks | https://firebase.google.com/docs/functions/task-functions | Görevler `scheduleTime` ile ileri bir zamana planlanabilir. "automatically retried up to 5 times" |
| T-4 | Cloud Tasks quotas | https://docs.cloud.google.com/tasks/docs/quotas | En fazla planlama süresi: "30 days from current date and time". Saklama süresi 31 gün |
| T-5 | firebase_messaging (pub.dev) | https://pub.dev/packages/firebase_messaging | Sürüm 17.1.0. Android, iOS, macOS, web. UIScene için `configureNotificationCenterDelegate()` gerekir |
| T-6 | FCM Flutter client setup | https://firebase.google.com/docs/cloud-messaging/flutter/client | "FCM clients require devices running Android 7.0 or higher that also have Google Play services installed". APNs anahtarı ve Xcode yetenekleri. iOS'ta geçici (provisional) izin seçeneği |
| T-7 | FCM Flutter: receive messages | https://firebase.google.com/docs/cloud-messaging/flutter/receive | Ön plandaki bildirim mesajları "won't display a visible notification by default, on both Android and iOS". `onMessage` akışı |
| T-8 | Add Firebase to your Android project | https://firebase.google.com/docs/android/setup | "Uses Android 7.0 or higher". API 24 hedefi. Google Play hizmetleri gereksinimi |
| T-9 | Add Firebase to your Apple project | https://firebase.google.com/docs/ios/setup | "Your project must target these platform versions or later: iOS 15" |
| T-10 | Flutter supported platforms | https://docs.flutter.dev/reference/supported-platforms | Flutter 3.50: Android "Supported 24 to 37", iOS "Supported 15 to 27" |
| T-11 | Firestore locations | https://firebase.google.com/docs/firestore/locations | Avrupa ve Orta Doğu bölgeleri listeleniyor, Türkiye bölgesi yok. "once you provision a database instance, you cannot change its location setting" |
| T-12 | Access data offline (Firestore) | https://firebase.google.com/docs/firestore/manage-data/enable-offline | "For Android and Apple platforms, offline persistence is enabled by default." Çevrimdışı yazmalar yeniden bağlanınca senkronize edilir |
| T-13 | Control access with custom claims | https://firebase.google.com/docs/auth/admin/custom-claims | Yalnızca Admin SDK ile atanır. 1000 bayt sınırı. Değişiklik yeni ID jetonuyla yansır. "Use custom claims to store data for controlling user access only." |
| T-14 | Firebase pricing | https://firebase.google.com/pricing | Cloud Functions yalnızca Blaze planında, "No-cost up to 2M/month" çağrı. Firestore "20K writes/day", "50K reads/day" ücretsiz. Telefon doğrulamada SMS "Billed per SMS sent". FCM ücretsiz |
| T-15 | Firestore pricing | https://firebase.google.com/docs/firestore/pricing | "There is a minimum charge of one document read for each query that you perform, even if the query returns no results." |
| T-16 | Luxon docs: Time zones (GitHub) | https://github.com/moment/luxon/blob/master/docs/zones.md | IANA bölgeleri, Intl API'ye dayanır. Yaz saati boşluğundaki saat ileri alınır. Tekrarlanan saatte "Luxon's behavior here is undefined" |
| T-17 | Luxon (GitHub depo) | https://github.com/moment/luxon | MIT lisansı. "Native time zone and Intl support (no locale or tz files)" |
| T-18 | cloud_firestore (pub.dev) | https://pub.dev/packages/cloud_firestore | Sürüm 7.2.0, yayıncı firebase.google.com |
| T-19 | cloud_functions (pub.dev) | https://pub.dev/packages/cloud_functions | Sürüm 7.1.0, yayıncı firebase.google.com |
| T-20 | firebase_auth (pub.dev) | https://pub.dev/packages/firebase_auth | Sürüm 7.1.0, yayıncı firebase.google.com |
| T-21 | firebase_core (pub.dev) | https://pub.dev/packages/firebase_core | Sürüm 5.2.0, yayıncı firebase.google.com |
| T-22 | Manage users in Firebase (Flutter) | https://firebase.google.com/docs/auth/flutter/manage-users | `delete()`. `requires-recent-login`. "You can send a password reset email to a user with the `sendPasswordResetEmail()` method." |
| T-23 | Manage functions | https://firebase.google.com/docs/functions/manage-functions | Node.js 22 ve 20 destekleniyor. `minInstances` soğuk başlatmayı azaltır. "A minimum number of instances kept running incur billing costs at idle rates". Örnek maliyet yaklaşık 3–8 $/ay (belgedeki örnek koşullarda) |
| T-24 | Writing conditions for Security Rules | https://firebase.google.com/docs/firestore/security/rules-conditions | `get()` ve `exists()` ile rol kontrolü. Belge erişim sınırı 10 ve 20. "you will be billed for reading documents even if your rules reject the request" |
| T-25 | Call functions from your app (callable) | https://firebase.google.com/docs/functions/callable | Auth jetonları istekle otomatik gönderilir. `HttpsError`. İstemcide varsayılan zaman aşımı 70 sn |
| T-26 | Firebase Hosting | https://firebase.google.com/docs/hosting | "Zero-configuration SSL is built into Firebase Hosting". Cloud Functions ile birlikte kullanılabilir |
| T-27 | Add Firebase to your JavaScript project | https://firebase.google.com/docs/web/setup | npm ile modüler API önerisi |
| T-28 | Firebase JS SDK supported environments | https://firebase.google.com/support/guides/environments_js-sdk | Auth ve Functions'ın tarayıcı desteği. CSP notu |
| T-29 | Choose a database: Firestore or Realtime Database | https://firebase.google.com/docs/database/rtdb-vs-firestore | Firestore: "Indexed queries with compound sorting and filtering", işlemler "any part of the database". RTDB: "Queries can sort *or* filter on a property, but not both", işlemler tek alt ağaçta |

**Açılamayan veya kullanılmayan kaynaklar (kaynak gösterilmedi):**
- Firestore SLA sayfası (cloud.google.com/firestore/sla): içerik kesik döndü, erişilebilirlik yüzdesi doğrulanamadı (NFR-12, Risk R-07).
- Apple "Asking permission to use notifications": sayfa içeriği gelmedi. iOS izin davranışı için yalnızca T-6 kullanıldı. BA'nın V-09 varsayımı hâlâ doğrulanmadı.
- npm'deki `firebase-functions`, `firebase-admin` ve `luxon` sayfaları 403 döndü, bu paketlerin güncel sürümleri doğrulanamadı.
- Luxon resmi belge sitesi (moment.github.io/luxon) içerik döndürmedi. Yerine GitHub'daki belge kullanıldı (T-16).

## 4. Bileşenler ve sorumlulukları

### 4.1 Mobil uygulama (Flutter)

| Bileşen | Sorumluluk | Gereksinim |
|---|---|---|
| Kimlik modülü | E-posta ve şifreyle kayıt ve giriş (geçici karar, AS-02 / S-02). Şifre sıfırlama. Kayıtta en fazla 3 zorunlu alan. Kayıt ekranında aydınlatma metni ve gizlilik politikası bağlantıları (Hosting) | FR-01, FR-02, NFR-08, US-01 |
| Rol yönlendirici | `users/{uid}.role` alanına göre müşteri, çalışan veya yönetici ana ekranını açar | FR-04 |
| Müşteri randevu akışı | Hizmet seçimi, çalışan seçimi ("fark etmez" dahil), tarih ve saat seçimi, onay. Uygulama açılınca doğrudan salonun hizmetleri gelir, işletme seçimi yoktur. En fazla 5 adım | FR-09, FR-10, NFR-05, US-05 |
| Randevularım | Firestore dinleyicisiyle yaklaşan ve geçmiş randevular (aktif, iptal, tamamlandı, gelmedi). Salon iptalinde kategori ve açıklama. Çevrimdışıyken önbellekten son liste gösterilir | FR-14, FR-17, NFR-13, US-07 |
| İptal ve taşıma ekranları | İptal süresi dolmamışsa "İptal et" ve "Taşı" seçenekleri görünür. Süre dolmuşsa salon iletişim bilgisi ve iptal kuralı gösterilir | FR-12, FR-21, US-06, US-10 |
| Çalışan takvimi | Günlük ve haftalık liste veya sütun görünümü. Yönetici için çalışan sütunları | FR-15, US-04 |
| Yönetici ekranları | Salon profili, hizmetler (not alanı yok), çalışanlar ve davet kodu, çalışma saatleri, mola ve izinler, randevu kuralları, hatırlatma zamanları, iptal (kategori zorunlu), telefonla gelen randevu, gelmedi işaretleme | FR-05–FR-08, FR-13, FR-20, FR-22–FR-24 |
| Bildirim modülü | İlk randevu onayından sonra gerekçe ekranı ve `requestPermission`. FCM jetonunu `users/{uid}/devices` altına yazma. Ön planda `onMessage` ile uygulama içi şerit. İzin yoksa ayarlardan açma yönergesi | FR-16, FR-17, US-08 |
| Hesap silme ekranı | Silinecek ve anonim tutulacak verilerin açıklaması, en geç tamamlanma süresi, şifreyle yeniden kimlik doğrulama, `deleteAccount` çağrısı. En fazla 3 adım | FR-03, NFR-07, US-02 |
| Bağlantı durumu | Callable çağrılarında istemci zaman aşımı geçici olarak 15 sn. Hata veya bağlantı yoksa "İşlem yapılamadı, internet bağlantınızı kontrol edin" mesajı gösterilir, işlem kuyruğa alınmaz | NFR-13 |

### 4.2 Cloud Functions (tek bölge, Node.js 22, TypeScript)

| Fonksiyon | Tür | Sorumluluk | Gereksinim |
|---|---|---|---|
| `getAvailableSlots` | callable | Tarih, hizmet ve çalışan (veya "fark etmez") için müsait başlangıç saatleri (Algoritma A-1) | FR-09, FR-22, NFR-03 |
| `createAppointment` | callable | Müsaitliği işlem içinde yeniden kontrol eder (A-2). Kaydeder, hatırlatmaları planlar (A-3), bildirim gönderir. "Fark etmez" seçilirse atama yapar (A-4) | FR-10, FR-11, FR-18 (a, c), FR-19, NFR-02 |
| `cancelAppointment` | callable | Müşteri iptali: iptal süresi kontrolü. Salon iptali: kategori zorunlu, açıklama isteğe bağlı. Aralığı serbest bırakır, hatırlatmaları iptal eder, bildirim gönderir | FR-12, FR-13, FR-18 (b), FR-19 |
| `rescheduleAppointment` | callable | Tek işlemde yeni aralığı alır ve eskisini bırakır. Hatırlatmaları yeniden planlar | FR-21, FR-18 (c) |
| `staffCreateAppointment` | callable | Yönetici veya çalışan, hesabı olan ya da olmayan (ad ve telefon) müşteri adına randevu girer | FR-20 |
| `markNoShow` | callable | Durumu "gelmedi" yapar | FR-23 |
| `redeemStaffInvite` | callable | Tek kullanımlık davet kodunu doğrular, kullanıcıyı çalışan kaydına bağlar ve `role=staff` atar | FR-07 |
| `updateSalonSettings` ve diğer yönetim yazmaları | callable | Salon, hizmet, çalışan, saat, mola, izin ve kural yazmaları. Yönetici rolü kontrolü yapılır | FR-05–FR-08, FR-22, FR-24 |
| `deleteAccount` | callable | Hesap silme (A-5) | FR-03, FR-29, NFR-07, NFR-14 |
| `reminderTick` | zamanlanmış, her dakika | Zamanı gelen hatırlatmaları gönderir. Yarım kalmış silme işlerini yeniden dener | FR-18 (c), FR-24, NFR-04, NFR-07 |
| `notify` (iç modül) | — | Bildirim şablonları (izin verilen alan listesiyle), FCM gönderimi, geçersiz jetonların temizlenmesi | FR-18, FR-19, NFR-11 |
| `availability` (iç modül) | — | A-1 ve A-2'nin ortak kodu. Tek uygulama, tek test kümesi | FR-09, FR-11 |

### 4.3 Firebase Hosting
- `/gizlilik` ve `/aydinlatma`: statik metinler. Metnin içeriği BA ve hukuk tarafından sağlanır (FR-02).
- `/hesap-sil`: Kullanıcı e-posta ve şifreyle giriş yapar ve aynı `deleteAccount` fonksiyonunu çağırır. Bu bağlantı mağaza sayfasında ve gizlilik politikasında verilir (FR-03, US-02 KK-3).

### 4.4 Kurulum betiği
İlk yönetici hesabı ve `salon/main` belgesi, kurulum sırasında Admin SDK ile çalışan tek seferlik bir betikle oluşturulur. Bu bir müşteri akışı değildir (V-01, FR-05).

## 5. Veri modeli

Tüm zamanlar UTC `Timestamp` olarak saklanır. Görüntüleme için sunucu, salonun yerel saatini ayrıca metin alanı olarak yazar (`startLocal`, ör. `2026-12-01T14:30`). Böylece istemci, cihazın saat diliminden bağımsız olarak salon saatini gösterir ve istemcide saat dilimi kütüphanesi gerekmez (NFR-10).

| Koleksiyon / belge | Alanlar | Okuma (Rules) | Yazma | Gereksinim |
|---|---|---|---|---|
| `salon/main` (tek belge) | `name`, `address`, `phone`, `timeZone` (`Europe/Istanbul`), `weeklyHours`, `closedDates[]`, `settings` (`cancelWindowMin`=120, `minLeadMin`, `maxAdvanceDays`=60, `bufferMin`=0, `slotStepMin`=15 geçici, `reminderOffsetsMin`=[1440], `approvalMode`=`auto`), `cancelReasonCategories[]` (`id`, `label`) | Giriş yapmış herkes | Yalnızca fonksiyon (yönetici) | FR-05, FR-12, FR-13, FR-22, FR-24, FR-25 |
| `services/{id}` | `name`, `durationMin`, `price`, `staffIds[]`, `active`. **Not alanı yok** | Giriş yapmış herkes | Fonksiyon (yönetici) | FR-06, NFR-08 |
| `staff/{id}` | `displayName`, `uid` (davet kabul edilene kadar null), `active`, `serviceIds[]`, `weeklyHours`, `breaks[]` | Giriş yapmış herkes (kişisel iletişim bilgisi yok) | Fonksiyon (yönetici; AS-05'e göre çalışan da) | FR-07, FR-08 |
| `staff/{id}/timeOff/{id}` | `startDate`, `endDate` | Çalışanın kendisi ve yönetici | Fonksiyon | FR-08 |
| `staffInvites/{id}` | `staffId`, `codeHash`, `expiresAt`, `usedAt` | Yok | Fonksiyon | FR-07 |
| `users/{uid}` | `role` (`customer`, `staff` veya `manager`), `name`, `email`, `phone` (AS-02'ye göre), `createdAt` | Kendisi ve yönetici | Kendisi yalnızca `name` ve `phone` alanlarını değiştirebilir, `role` alanını değiştiremez. Diğer alanları fonksiyon yazar | FR-01, FR-04, NFR-06, NFR-08 |
| `users/{uid}/devices/{tokenId}` | `fcmToken`, `platform`, `updatedAt` | Kendisi | Kendisi | FR-18, FR-19 |
| `appointments/{id}` | `staffId`, `staffUid`, `serviceId`, `serviceName`, `durationMin`, `startAt`, `endAt`, `startLocal`, `status` (`active`, `pending`, `cancelled`, `completed`, `noShow`), `source` (`app` veya `staff`), `customerUid` (null olabilir), `customer` {`name`, `phone`, `email`} veya null, `customerLabel` ("Hesap silindi" veya null), `cancelledBy` (`customer`, `salon` veya `accountDeletion`), `cancelReasonCategory`, `cancelNote` (en fazla 200 karakter, geçici), `createdAt`, `updatedAt` | Müşteri: `customerUid == uid`. Çalışan: `staffUid == uid`. Yönetici: tümü | Yalnızca fonksiyon | FR-10–FR-14, FR-20, FR-21, FR-23, FR-29, NFR-06 |
| `staffDays/{staffId}_{YYYY-MM-DD}` | `intervals[]`: {`appointmentId`, `start`, `end`} (UTC). Müşteri bilgisi yok | Yok (istemci okumaz) | Yalnızca fonksiyon (işlem içinde) | FR-09, FR-11, NFR-02 |
| `reminders/{appointmentId}_{offsetMin}` | `appointmentId`, `dueAt`, `status` (`scheduled`, `sending`, `sent`, `skipped`, `cancelled`), `immediate` | Yok | Yalnızca fonksiyon | FR-18 (c), FR-24 |
| `deletionJobs/{jobId}` | `uid`, `requestedAt`, `step`, `attempts` | Yok | Yalnızca fonksiyon. İş tamamlanınca belge silinir | NFR-07 |

**Notlar:**
- **Müşteri bilgisinin randevuda kopyalanması (NFR-06):** Çalışan, müşterinin yalnızca randevu için gereken ad, iletişim ve hizmet bilgisini görmelidir. Bu bilgi randevu belgesine `customer` alanı olarak kopyalanır. Böylece çalışana `users` koleksiyonunu okuma izni verilmez. Bunun karşılığında anonimleştirmede (FR-29) bu kopyaların da temizlenmesi gerekir (A-5).
- **Gelmeme sayısı (FR-23):** Ayrı bir sayaç tutulmaz. Sayı, yönetici ekranında `customerUid == X && status == noShow` sorgusundan hesaplanır. Anonimleştirmede `customerUid` null yapıldığı için sayı kendiliğinden kaybolur (FR-29, US-02 KK-5). Ayrıca silinmesi gereken bir sayaç olmaz.
- **İptal nedeni kategorileri (FR-13, AS-14):** Kategoriler `salon/main` belgesinde yapılandırılır ve geçici olarak BA'nın önerdiği 4 kategoriyle başlar. Sağlık veya kişisel durum içeren bir kategori eklenmemesi bir içerik kuralıdır, kod bunu denetleyemez.
- **Serbest metin alanları:** Üründe tek serbest metin alanı `cancelNote`'tur (V-08). Giriş alanında "Sağlık veya kişisel bilgi yazmayınız" uyarısı gösterilir (V-08'deki BA önerisi).
- **Firestore konumu:** Geçici olarak `europe-west3` seçildi. Konum sonradan değiştirilemediği [T-11] için kesin karar AS-08 / S-01'e bağlıdır. Fonksiyonlar da aynı bölgeye kurulur.

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Kritik kararlar ve karşılaştırmalar

**K-1: Veri saklama: Cloud Firestore ve Realtime Database**

Seçim ölçütleri:
- Ö1: Birden çok belgeye dokunan atomik işlem (FR-11, FR-21, NFR-02).
- Ö2: Aynı sorguda hem filtreleme hem sıralama (FR-14 durum + tarih, FR-15 çalışan + tarih, hatırlatma taraması).
- Ö3: Mobilde çevrimdışı önbellek (NFR-13).
- Ö4: Bölge seçeneği (AS-08).

| Ölçüt | Cloud Firestore | Realtime Database |
|---|---|---|
| Ö1 | "Transactions can atomically read and write data from any part of the database" [T-29] | "Transactions are atomic on a specific data subtree" [T-29] |
| Ö2 | "Indexed queries with compound sorting and filtering" [T-29] | "Queries can sort *or* filter on a property, but not both" [T-29] |
| Ö3 | Android ve Apple'da varsayılan açık [T-12] | Apple ve Android'de var [T-29] |
| Ö4 | Bölgesel ve çok bölgeli seçenekler [T-11, T-29] | "Realtime Database is a regional solution" [T-29] |

**Seçim: Cloud Firestore.** Ö1 ve Ö2'de açık üstünlüğü var. Taşıma işlemi (FR-21) iki farklı gün belgesine aynı işlemde dokunur. Randevu listeleri de durum ve tarih koşulunu birlikte kullanır.

**K-2: Randevu yazma yolu**

Seçim ölçütleri:
- Ö1: Çift rezervasyonun kesin önlenmesi (FR-11, NFR-02).
- Ö2: Çevrimdışıyken işlemin kuyruğa alınmaması (NFR-13).
- Ö3: Yetkisiz değişikliğin engellenmesi (NFR-06).
- Ö4: Bildirim ve hatırlatmanın her yazmada tutarlı tetiklenmesi (FR-18, FR-19).

| Ölçüt | A. İstemciden doğrudan Firestore yazması + Security Rules | B. Tüm yazmalar callable fonksiyonla, istemci yalnızca okur |
|---|---|---|
| Ö1 | İstemci işlemiyle mümkün. Ancak çakışma kuralı hem Dart'ta hem kurallarda yazılmalı | Tek yerde (Admin SDK işlemi) |
| Ö2 | Çevrimdışı yazmalar yerelde tutulur ve bağlantı gelince senkronize edilir [T-12]. NFR-13 ile çelişir. İşlemler çevrimdışıyken başarısız olur [T-1], ama diğer yazmalar kuyruğa girer | Çağrı sunucuya ulaşamazsa hata döner, kuyruk yoktur |
| Ö3 | Durum geçişleri (ör. iptal süresi) kurallarda ifade edilmeli. Belge erişim sınırları var [T-24] | İş kuralı kodda. Kurallar yalnızca okuma için |
| Ö4 | Firestore tetikleyicisi gerekir | Aynı fonksiyon içinde |

**Seçim: B.** Bedeli, her yazmanın bir fonksiyon çağrısı olması ve soğuk başlatma riskidir (NFR-03, Bölüm 7).

**K-3: Çakışma önleme yöntemi**

Seçim ölçütleri:
- Ö1: İki eşzamanlı istekten yalnızca birinin başarılı olması (FR-11, NFR-02).
- Ö2: Değişken hizmet süresi ve tampon süre (FR-09, FR-22).
- Ö3: Taşımanın atomik olması (FR-21).
- Ö4: Müsaitlik hesabında okuma sayısı (NFR-03).

| Ölçüt | A. Izgara kilit belgeleri (`slotLocks/{staff}_{15dk}`), randevu başına N belge | B. İşlem içinde çalışanın günlük randevularını sorgulama | C. Çalışan-gün belgesi (`staffDays`), içinde aralık listesi |
|---|---|---|---|
| Ö1 | Belge oluşturma çakışmasıyla sağlanır | Sağlanır (işlem yeniden çalıştırılır [T-1]) | Tek belge üzerinde okuma-değiştirme-yazma, eşzamanlı düzenlemede işlem yeniden çalışır [T-1] |
| Ö2 | Tampon ve süre ızgaraya yuvarlanmalı | Doğal | Doğal (aralık çakışma kontrolü) |
| Ö3 | 2N belge, kural sınırlarına yaklaşabilir | Sorgulu işlem | 2 belge |
| Ö4 | Çok sayıda belge | Sorgu | Çalışan başına günde 1 belge |

**Seçim: C.** Tek salon ölçeğinde aynı çalışan-gün belgesine düşen eşzamanlı istek sayısı düşüktür. Bu yüzden belge başına seri işlem sorun yaratmaz. Aralık kontrolü A-1 ile aynı kodu kullanır.

**K-4: Müsait saat üretimi**

Seçim ölçütleri:
- Ö1: Süresi molaya veya kapanışa taşan saatin gösterilmemesi (FR-09, US-05 KK-1).
- Ö2: Müşteri için anlaşılır saatler (NFR-05, BA 3.5).
- Ö3: Doluluk (BA 3.5).
- Ö4: Tek uygulama, tek test kümesi (FR-09 ile FR-11'in tutarlılığı).
- Ö5: Yanıt süresi (NFR-03).

| Ölçüt | A. Sabit ızgara, uygunluk ızgara birimiyle | B. Tamamen dinamik (her boşluğun başı) | C. Başlangıçlar ızgarada, uygunluk gerçek süre + tamponla (hibrit) |
|---|---|---|---|
| Ö1 | Süre yuvarlanırsa hata riski var | Sağlanır | Sağlanır |
| Ö2 | Anlaşılır | Düzensiz saatler (ör. 10:35) | Anlaşılır |
| Ö3 | Düşük | Yüksek | Orta |
| Ö4 / Ö5 | — | — | — |

Ö4 ve Ö5 ayrı bir alt karara göre ele alınır. Hesap yalnızca sunucuda (`getAvailableSlots`) yapılır. Alternatifi, Dart'ta istemci tarafı hesap ve TS'de sunucu doğrulamasıdır. Bu alternatif iki ayrı uygulama ve tutarsızlık riski getirir. Sunucu tarafında ise soğuk başlatma riski vardır. Bu risk `minInstances=1` ile azaltılır ve NFR-03 ölçümüyle doğrulanır.

**Seçim: C, sunucu tarafında.** Izgara adımı geçici olarak 15 dk'dır ve `settings.slotStepMin` ile ayarlanabilir tutulur. Kesin karar AS-17'ye bağlıdır.

**K-5: Hatırlatma zamanlama yöntemi**

Seçim ölçütleri:
- Ö1: Planlı gönderimde ±5 dk (NFR-04).
- Ö2: 60 gün ilerideki randevu (FR-22, V-04).
- Ö3: İptal veya taşımada hatırlatmanın kaldırılması (US-08 KK-4, FR-21).
- Ö4: Ayar değişince yeniden planlama (FR-24, AS-15 b).
- Ö5: Maliyet ve sadelik.

| Ölçüt | A. Dakikada bir zamanlanmış tarama (`reminders` sorgusu) | B. Randevu başına Cloud Tasks görevi (`scheduleTime`) |
|---|---|---|
| Ö1 | En fazla yaklaşık 1 dk gecikme | Planlanan anda |
| Ö2 | Sınır yok | En fazla "30 days from current date and time" [T-4]. 60 günlük randevu için ara yeniden planlama gerekir |
| Ö3 | Belge durumu `cancelled` yapılır | Görev silme API'si gerekir [T-3] |
| Ö4 | `dueAt` güncellenir | Görevler silinip yeniden oluşturulur |
| Ö5 | Ayda 43.200 çağrı, ücretsiz kota 2M/ay içinde [T-14] (kaynaktaki verilerden hesaplandı). Boş sorgu da en az 1 okuma sayılır [T-15], günde yaklaşık 1.440 okuma, ücretsiz kota 50K/gün içinde [T-14] (hesaplandı). Scheduler işi 3 işlik ücretsiz kota içinde [T-2] | Cloud Tasks ücretleri [T-3]. Ek kuyruk yapılandırması |

**Seçim: A.** Ö2 tek başına belirleyicidir. Ö1'deki yaklaşık 1 dk'lık gecikme ±5 dk toleransı içindedir.

**K-6: Hesap silmede veri yolu**

Seçim ölçütleri:
- Ö1: Hesap ve kişisel veri silinir, yalnızca devre dışı bırakma yapılmaz (FR-03, BA K-18).
- Ö2: Geçmiş kayıt sayısı korunur (FR-29, US-12 KK-3).
- Ö3: Silinen kullanıcının kimlik bilgileriyle 0 eşleşme (NFR-14).
- Ö4: 30 gün içinde tamamlanma (NFR-07).

| Ölçüt | A. Seçenek A: hesap ve profil silinir, randevu kayıtları kişisel alanlar temizlenerek tutulur | B. Randevu kayıtları dahil tamamen silme | C. Silindi bayrağı (soft delete), veri kalır |
|---|---|---|---|
| Ö1 | Sağlanır | Sağlanır | Sağlanmaz (BA 3.6 Seçenek C) |
| Ö2 | Sağlanır | Sağlanmaz ("hard delete olmamalı") | Sağlanır |
| Ö3 | Sağlanır (uid dahil tüm tanımlayıcılar temizlenir, takma kimlik tutulmaz) | Sağlanır | Sağlanmaz |
| Ö4 | Eşzamanlı yapılır | Eşzamanlı yapılır | — |

**Seçim: A** (BA'nın önerdiği Seçenek A, FR-29). Anonimliğin hukuken yeterliliği teknik olarak çözülemez, AS-13 (b)'ye bağlıdır (S-12).

**K-7: Rol ve yetki bilgisinin kaynağı**

Seçim ölçütleri:
- Ö1: Yetkisiz erişimin 0 olması (NFR-06).
- Ö2: Çalışan çıkarıldığında yetkinin hemen düşmesi (FR-07).
- Ö3: Okuma maliyeti.

| Ölçüt | A. Özel talepler (custom claims) | B. `users/{uid}.role` + kurallarda `get()` |
|---|---|---|
| Ö1 | Sağlanır | Sağlanır |
| Ö2 | Değişiklik ancak yeni ID jetonuyla yansır [T-13]. Gecikme olabilir | Sonraki istekte geçerli olur |
| Ö3 | Ek okuma yok [T-13] | Kural başına faturalı okuma, sınır 10 [T-24] |

**Seçim: B.** Salon ölçeğinde ek okuma maliyeti düşüktür. Ö2 güvenlik açısından önceliklidir. Fonksiyonlar da rolü aynı belgeden okur.

### 6.2 A-1: Müsait başlangıç saatleri (`getAvailableSlots`) (FR-09, FR-22, FR-08, FR-05)

Girdi: yerel tarih `D`, hizmet `s` (süre `d`), çalışan `e` veya "fark etmez", ayarlar (`g`=adım, `b`=tampon, `minLead`, `maxAdvanceDays`).

1. `D`, bugünden `maxAdvanceDays` günden daha ileriyse veya salonun kapalı günüyse boş liste döner (FR-05, FR-22, US-03 KK-2).
2. Aday çalışanlar: `e` seçildiyse yalnızca o çalışan, değilse `s.staffIds` içindeki aktif çalışanlar. O gün izinli olan çalışan çıkarılır (US-04 KK-2).
3. Her çalışan için çalışma aralıkları hesaplanır: salon saatleri ∩ çalışan saatleri − molalar. Yerel saatler Luxon ile `salon.timeZone`'a göre UTC'ye çevrilir.
4. Dolu aralıklar `staffDays` belgesinden okunur.
5. Aday başlangıçlar: açılıştan itibaren `g` adımlı her `t` (yerel saat). `t` şu koşulları sağlarsa uygundur:
   - `[t, t+d)` tamamen bir çalışma aralığının içindedir (molaya ve kapanışa taşmaz, US-05 KK-1, US-04 KK-1).
   - `[t−b, t+d+b)` hiçbir dolu aralıkla kesişmez (FR-22 tampon).
   - `t ≥ şimdi + minLead` (US-11 KK-1).
6. "Fark etmez" seçildiyse çalışanların sonuçlarının birleşimi döner. Her saat için uygun çalışan listesi sunucuda kalır.

Çalışan başına aralık sayısı küçüktür. Sıralı aralıklar üzerinde doğrusal tarama yeterlidir.

### 6.3 A-2: Randevu kaydı (`createAppointment`) (FR-10, FR-11, NFR-02)

1. Tek Firestore işlemi içinde `staffDays/{staff}_{D}` belgesi okunur.
2. A-1'in 5. adımındaki koşullar aynı fonksiyonla yeniden kontrol edilir.
3. Koşul tutmazsa `HttpsError('already-exists')` döner. İstemci "Bu saat az önce doldu" mesajını ve güncel listeyi gösterir (US-05 KK-3).
4. Koşul tutarsa aralık eklenir, `appointments` belgesi yazılır ve `reminders` belgeleri yazılır (A-3). Bunların hepsi aynı işlemdedir.
5. İşlem başarılı olursa bildirimler gönderilir: müşteriye "oluşturuldu" (FR-18 a), çalışana ve yöneticilere "yeni randevu" (FR-19).

Eşzamanlı düzenlemede Firestore işlemi yeniden çalıştırır [T-1]. Bu nedenle ikinci istek güncel aralıkları görür ve reddedilir.

**Taşıma (FR-21):** Eski ve yeni `staffDays` belgeleri aynı işlemde okunur. Önce yeni aralık eklenir, sonra eskisi kaldırılır. Yeni aralık alınamazsa hiçbir değişiklik yapılmaz (US-10 KK-1).

### 6.4 A-3: Hatırlatma planlama (FR-18 c, FR-24, US-08 KK-5/KK-6)

Hatırlatmalar randevu onaylandığı anda planlanır: otomatik onayda oluşturma veya taşıma anında, manuel onayda (FR-25) onay anında (V-14). Her `o ∈ reminderOffsetsMin` için `dueAt = startAt − o` hesaplanır.
- `dueAt > şimdi` olanlar `scheduled` olarak yazılır.
- `dueAt ≤ şimdi` olanlardan **yalnızca biri** (en büyük `o`) `immediate=true` olarak hemen gönderilir. Diğerleri `skipped` yazılır.

Örnekler:
- 24 sa ve 2 sa ayarı, randevudan 3 sa önce oluşturma: 24 sa hatırlatması hemen, 2 sa hatırlatması zamanında gider.
- Aynı ayar, randevudan 1 sa önce oluşturma: yalnızca bir hatırlatma hemen gider.

Hemen gönderilen hatırlatma, oluşturma bildiriminden ayrı bir mesajdır (V-14). Bu karar geçicidir, AS-15 (a)'ya bağlıdır.

İptal veya taşımada `scheduled` durumdaki hatırlatmalar `cancelled` yapılır (US-08 KK-4).

**`reminderTick`:**
- Her dakika `status=='scheduled' && dueAt<=şimdi` sorgusunu çalıştırır (bileşik dizin).
- Her belgeyi işlem içinde `sending` yapar, gönderir ve `sent` yapar. Böylece iki çalışma aynı belgeyi göndermez.
- Gönderim hatasında belge `scheduled` durumuna geri alınır ve sonraki dakikada tekrar denenir.

Salon hatırlatma ayarını değiştirdiğinde mevcut hatırlatmalar geçici olarak değiştirilmez. Bu karar AS-15 (b)'ye bağlıdır (S-08).

### 6.5 A-4: "Fark etmez" ataması (FR-10, AS-04)

Geçici kural: Seçilen saatte uygun olan çalışanlardan o gün en az randevusu olan seçilir. Eşitlikte `staffId` sırası kullanılır (belirlenimci). Atanan çalışanın adı onayda gösterilir (US-05 KK-4). Kesin kural AS-04'e bağlıdır (S-06).

### 6.6 A-5: Hesap silme (`deleteAccount`) (FR-03, FR-29, NFR-07, NFR-14)

1. İstemci önce kullanıcıyı şifreyle yeniden doğrular. Hassas işlemlerde `requires-recent-login` gerekir [T-22]. Bu adım onay ekranında yapılır, böylece US-02 KK-1'deki en fazla 3 adım korunur.
2. `deletionJobs` belgesi oluşturulur.
3. Gelecekteki aktif randevular iptal edilir: `cancelledBy=accountDeletion`, `staffDays` aralığı serbest bırakılır, hatırlatmalar iptal edilir, çalışana ve yöneticilere müşteri iptali bildirimi gider (FR-19, US-02 KK-2).
4. Geçmiş randevular anonimleştirilir:
   - `customerUid=null`, `customer=null`, `customerLabel="Hesap silindi"`.
   - Tarih, saat, hizmet, çalışan ve durum alanları değişmez (FR-29, US-12).
   - Kullanıcının daha önce iptal edilmiş randevuları da aynı şekilde anonimleştirilir.
5. `users/{uid}` belgesi ve `devices` alt koleksiyonu silinir.
6. Firebase Auth kullanıcısı Admin SDK ile silinir. Bu andan sonra giriş yapılamaz (US-02 KK-2).
7. `deletionJobs` belgesi silinir.

Adımlar yeniden çalıştırılabilir (idempotent) yazılır. Bir adım başarısız olursa `reminderTick`, bekleyen işleri yeniden dener. 30 gün yerine pratikte dakikalar içinde tamamlanır (NFR-07). Onay ekranında ise BA'nın belirlediği üst sınır ("en geç 30 gün") gösterilir (US-02 KK-4).

Hesapsız müşteri (FR-20) kayıtlarının silinmesi için bir akış tanımlı değildir (S-09).

## 7. Performans

| Hedef | Tasarım önlemi | Doğrulama |
|---|---|---|
| NFR-03: müsait saat listesi p95 ≤ 2 sn, oluşturma onayı ≤ 3 sn (4G) | A-1 sabit sayıda belge okur: `salon`, `service`, aday çalışan belgeleri, izinler ve çalışan başına 1 `staffDays`. `getAvailableSlots` ve `createAppointment` için `minInstances=1`. Belgeye göre boşta örnek de ücretlendirilir. Belgedeki örnek koşullarda maliyet yaklaşık 3–8 $/ay [T-23]. Fonksiyon ve Firestore aynı bölgededir | Bölüm 9, P-1 |
| NFR-04: olay bildirimleri ≤ 1 dk, planlı hatırlatma ±5 dk | Olay bildirimleri aynı fonksiyon çağrısında gönderilir. Planlı hatırlatmada en fazla yaklaşık 1 dk tarama gecikmesi olur (K-5) | Bölüm 9, P-2. Gönderim anı kaydedilir. Cihaza teslim ölçülmez (NFR-04 notu) |
| NFR-02: 50 eşzamanlı deneme | K-3 C. İşlem kilit süresi 20 sn [T-1] sınırının çok altında kısa işlemler | Bölüm 9, T-3 |
| Maliyet (BA'da ölçüt yok, bilgi amaçlı) | Firestore ve FCM kullanımı tek salon ölçeğinde ücretsiz kotalara yakın kalır [T-14]. Tarama maliyeti K-5'te hesaplandı | Blaze bütçe uyarısı kurulur |

NFR-03 ölçümü ilk sürümde `minInstances=1` olmadan da yapılır. Hedef bu durumda da tutuyorsa ayar kapatılabilir ve maliyet sıfıra iner.

## 8. Güvenlik ve gizlilik

- **Yetkilendirme (NFR-06, FR-04):**
  - Okuma kuralları rolü `users/{uid}.role` alanından alır (K-7). Kurallar Bölüm 5'teki tabloya göre yazılır.
  - Müşteri yalnızca `customerUid == request.auth.uid` olan randevuları okur. Çalışan yalnızca `staffUid == request.auth.uid` olanları okur. Yönetici tümünü okur.
  - İstemci sorguları da aynı filtreyi taşımalıdır.
  - `appointments`, `staffDays`, `reminders`, `staffInvites` ve `deletionJobs` koleksiyonlarına istemci yazması tamamen kapalıdır.
  - Fonksiyonlar her çağrıda `request.auth` [T-25] ve rol kontrolü yapar.
- **Rol yükseltme koruması:** Kullanıcı `users/{uid}` belgesinde yalnızca `name` ve `phone` alanlarını değiştirebilir. `role` alanını yalnızca fonksiyonlar yazar: davet kodu (`redeemStaffInvite`) ve kurulum betiği.
- **Davet kodu (FR-07):** Tek kullanımlıktır, geçici olarak 7 gün geçerlidir. Sunucuda özeti (hash) saklanır.
- **Veri en aza indirme (NFR-08):**
  - Kayıtta en fazla 3 zorunlu alan vardır.
  - Not alanı yoktur (FR-06).
  - Çalışan belgesinde kişisel iletişim bilgisi tutulmaz.
  - Mağaza veri beyanı için uygulamanın kendi topladığı veriler şunlardır: ad, e-posta ve/veya telefon, randevu verisi, FCM jetonu. Firebase SDK'larının ayrıca topladığı veriler bu çalışmada araştırılmadı. Yayın öncesinde Firebase belgelerinden doğrulanmalıdır (Risk R-09).
- **Bildirim içeriği (NFR-11, FR-18 b):**
  - `notify` modülünde her bildirim türü için yalnızca izin verilen alanlardan (salon, hizmet, tarih, saat, çalışan, iptal kategorisi) üretilen sabit şablonlar vardır.
  - `cancelNote` ve kampanya metni şablona hiçbir yoldan giremez.
  - Bildirim yükü (data) yalnızca `appointmentId` taşır. Uygulama detayı Firestore'dan okur.
- **Hesap silme ve anonimleştirme (FR-03, FR-29, NFR-14):** A-5. Anonimleştirmeden sonra randevu belgelerinde, kullanıcı belgelerinde ve cihaz belgelerinde silinen kullanıcının uid, ad, e-posta ve telefon bilgisi kalmaz. Takma kimlik veya özet (hash) tutulmaz.
- **Gizlilik metinleri (FR-02):** Metinler Hosting'de HTTPS üzerinden yayımlanır [T-26] ve uygulama içinden bağlantıyla açılır. Metnin içeriği teknik kapsam dışıdır.
- **Veri konumu (AS-08):** Firestore'da Türkiye bölgesi yok [T-11]. Seçilecek her konum yurt dışında olacaktır. Hukuki dayanak BA ve hukuk tarafında netleşmelidir (S-01).
- **Bildirim izni (FR-16, FR-17):**
  - İzin, ilk başarılı randevu onayından sonra bir gerekçe ekranıyla istenir.
  - İzin verilmezse tüm akış çalışır. Randevu ve iptal bilgisi uygulama içinde görünür.
  - iOS'ta reddedilen iznin yeniden istenip istenemeyeceği doğrulanmadı (Apple sayfası açılamadı). Bu nedenle izin yoksa ayarlardan açma yönergesi gösterilir.

## 9. Test stratejisi

| # | Test | Yöntem | Gereksinim |
|---|---|---|---|
| T-1 | A-1 birim testleri | TS birim testleri. Kapsanan durumlar: mola 12:30–13:30 (US-04 KK-1), 45 dk hizmet ve kapanış (US-05 KK-1), izinli çalışan (US-04 KK-2), kapalı gün (US-03 KK-2), `minLead` 2 sa (US-11 KK-1), tampon, `maxAdvanceDays` | FR-08, FR-09, FR-22 |
| T-2 | A-3 birim testleri | 24 sa / 3 sa, 24+2 sa / 3 sa, 24+2 sa / 1 sa senaryoları. İptalde hatırlatmanın iptal edilmesi | FR-18 (c), FR-24, US-08 KK-4–KK-6 |
| T-3 | Eşzamanlılık testi | Ayrı bir test Firebase projesinde aynı çalışan ve saate 50 paralel `createAppointment` çağrısı. Beklenen: `staffDays`'te ve `appointments`'ta en fazla 1 aktif kayıt. Taşıma ve iptalle karışık senaryo da denenir | FR-11, FR-21, NFR-02 |
| T-4 | Güvenlik kuralı testleri | Her rol için izinli ve izinsiz okuma ve yazma matrisi. Beklenen başarılı yetkisiz erişim: 0. `role` alanını değiştirme denemesi | FR-04, NFR-06 |
| T-5 | Bildirim şablon testi | Tüm şablonlar `cancelNote` ve kampanya metni verilerek üretilir. Çıktıda serbest metin ve kampanya içeriği sayısı 0 olmalı | NFR-11, FR-18 (b) |
| T-6 | Hesap silme testi | Geçmiş, gelecek ve gelmedi randevuları olan bir kullanıcı silinir. Kontroller: ad, e-posta, telefon ve uid ile tüm koleksiyonlarda 0 eşleşme; "Hesap silindi" etiketli kayıt sayısı = silme öncesindeki geçmiş randevu sayısı; tarih aralığındaki toplam randevu sayısı değişmemiş; gelecek randevular iptal edilmiş ve bildirim gitmiş; giriş yapılamıyor. Web silme sayfasından da aynı test | FR-03, FR-19, FR-23, FR-29, NFR-07, NFR-14, US-02, US-12 |
| T-7 | Çevrimdışı testi | Uçak modunda randevu listesi önbellekten görünür. Oluşturma ve iptal 15 sn içinde açık bir hata mesajı verir. Bağlantı gelince kendiliğinden işlem yapılmamış olmalı | NFR-13 |
| P-1 | Yanıt süresi | Gerçek cihaz, 4G, en az 100 ölçüm. Müsait saat listesi p95 ≤ 2 sn, oluşturma ≤ 3 sn. Soğuk ve sıcak örnek ayrı raporlanır | NFR-03 |
| P-2 | Bildirim zamanlaması | Gönderim anı ile olay veya `dueAt` arasındaki fark kaydedilir. Olayda ≤ 1 dk, planlıda ±5 dk | NFR-04 |
| U-1 | Kullanılabilirlik | 5 ilk kez kullanan katılımcı. En az 4'ü yardımsız ve 3 dk içinde randevu oluşturmalı. Adım sayısı ≤ 5 | NFR-05 |
| A-1t | Erişilebilirlik | TalkBack ve VoiceOver ile ana ekranlarda gezinme. En büyük sistem yazı boyutunda kesilme kontrolü. Kontrast ölçümü (≥ 4.5:1). Dokunma hedefi ≥ 24×24 mantıksal piksel | NFR-09 |
| M-1 | Platform matrisi | Android 7.0 (API 24) ve güncel Android, iOS 15 ve güncel iOS üzerinde tüm M gereksinimlerinin kabul testleri. Android 13+ üzerinde izin akışı. Google Play hizmetleri olmayan bir cihazda FCM'siz akış | NFR-01, FR-16, FR-17 |
| L-1 | Yerelleştirme | Cihaz saat dilimi salonunkinden farklıyken salon saatinin 24 saat biçiminde ve tarihin GG.AA.YYYY biçiminde gösterimi | NFR-10 |

## 10. Riskler

| # | Risk | Etki | Önlem | İlgili |
|---|---|---|---|---|
| R-01 | Firestore konumu sonradan değiştirilemez [T-11]. AS-08 yanıtlanmadan proje kurulursa veri taşıma gerekir | Yüksek | Proje kurulumu S-01 yanıtına kadar bekletilir | AS-08 |
| R-02 | FCM, Google Play hizmetleri olmayan Android cihazlarda çalışmaz [T-6] | Orta | Bu kullanıcılar bilgiyi uygulama içinde görür (FR-17). Kapsam BA'ya sorulur (S-10) | FR-17, FR-18 |
| R-03 | Ön plandaki bildirimler varsayılan olarak görünmez [T-7] | Düşük | `onMessage` ile uygulama içi şerit gösterilir | FR-18 |
| R-04 | Luxon'da yaz saati geçişindeki tekrarlanan saatte davranış tanımsız [T-16] | Saat diliminde yaz saati varsa yanlış slot oluşabilir | Saat dilimi `salon.timeZone` ile yapılandırılır. A-1 testlerine geçiş günü senaryosu eklenir | FR-09, NFR-10 |
| R-05 | Anonimleştirilmiş kayıt (tarih, saat, çalışan) küçük salonda başka bilgilerle kişiyi tanımlatabilir. Teknik çözüm bunu tek başına gideremez | Hukuki | AS-13 (b) yanıtına göre ek önlem (S-12) | FR-29, NFR-14 |
| R-06 | Callable fonksiyonların çevrimdışı davranışı belgede yok. Varsayılan zaman aşımı 70 sn [T-25] | Kullanıcı uzun süre bekler | İstemcide zaman aşımı 15 sn. T-7 testi | NFR-13 |
| R-07 | Firestore ve Functions SLA yüzdeleri doğrulanamadı (sayfa açılamadı). %99,5 hedefinin platform tarafından karşılanıp karşılanmadığı bilinmiyor | Orta | AS-10 yanıtından sonra SLA yeniden incelenir (S-11) | NFR-12 |
| R-08 | `firebase-functions`, `firebase-admin` ve `luxon` sürümleri doğrulanamadı (npm 403) | Düşük | Kurulumda sürümler sabitlenir ve ayrıca kontrol edilir | — |
| R-09 | Firebase SDK'larının topladığı veriler araştırılmadı. Data safety beyanı eksik kalabilir | Mağaza reddi | Yayın öncesinde Firebase'in veri açıklama belgeleri incelenir | NFR-08 |
| R-10 | Blaze planı zorunlu [T-14]. Fatura sahibi belirsiz | Kurulum engeli | S-15 | — |
| R-11 | Hatırlatma taraması dakikada bir çalışır. Ölçeklenme gerekirse ek çalışma gerekebilir | Tek salon için düşük | Gönderim gecikmesi P-2 ile izlenir | NFR-04 |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Kayıt, giriş, şifre sıfırlama, gerekli minimum alanlar | `firebase_auth` (e-posta ve şifre, geçici), kimlik modülü, `users/{uid}` | 3.1, 4.1, 5 |
| FR-02 | KVKK aydınlatma metni ve gizlilik politikası, her zaman erişilebilir | Hosting `/gizlilik`, `/aydinlatma`; kayıt ekranı bağlantıları | 4.3, 8 |
| FR-03 | Uygulama içi ve web üzerinden hesap silme, geçmişin anonim tutulması | `deleteAccount` (A-5), K-6, Hosting `/hesap-sil` + JS SDK | 4.2, 4.3, 6.6 |
| FR-04 | Üç rol ve yetki sınırları | `users.role` + Rules (K-7), rol yönlendirici | 4.1, 6.1, 8 |
| FR-05 | Tek salon profili, saatler, kapalı günler | `salon/main`, kurulum betiği | 4.4, 5 |
| FR-06 | Hizmet tanımı, not alanı yok | `services/{id}` | 5 |
| FR-07 | Çalışan ekleme, çıkarma, davet | `staff`, `staffInvites`, `redeemStaffInvite` | 4.2, 5, 8 |
| FR-08 | Çalışan saatleri, molalar, izinler | `staff.weeklyHours`, `breaks`, `timeOff`; A-1 | 5, 6.2 |
| FR-09 | Müsait başlangıç saatleri | `getAvailableSlots`, A-1, K-4 | 6.1, 6.2 |
| FR-10 | Randevu oluşturma, "fark etmez" | `createAppointment`, A-2, A-4 | 6.3, 6.5 |
| FR-11 | Çift rezervasyon olmaması | K-2, K-3 (`staffDays` işlemi), A-2 | 6.1, 6.3 |
| FR-12 | Müşteri iptali ve iptal süresi | `cancelAppointment`, `settings.cancelWindowMin` | 4.2, 5 |
| FR-13 | Salon iptali, zorunlu kategori, isteğe bağlı açıklama | `cancelAppointment`, `cancelReasonCategories`, `cancelNote` | 4.2, 5 |
| FR-14 | Randevu listesi ve iptal nedeni gösterimi | Randevularım ekranı, Firestore dinleyicisi | 4.1 |
| FR-15 | Çalışan ve yönetici takvim görünümleri | Çalışan takvimi bileşeni | 4.1 |
| FR-16 | Bildirim izninin bağlamında istenmesi | Bildirim modülü, `firebase_messaging` | 3.1, 4.1, 8 |
| FR-17 | İzinsiz çalışma, uygulama içi bilgi | Firestore dinleyicisi, uygulama içi şerit, ayar yönergesi | 4.1, 8 |
| FR-18 | Müşteri bildirimleri (oluşturma, salon iptali/değişikliği, hatırlatma, hemen hatırlatma) | `notify`, A-2, A-3, `reminderTick` (K-5). "Değiştirildi" olayı S-04'e bağlı | 4.2, 6.3, 6.4, 8 |
| FR-19 | Çalışan ve yönetici bildirimleri | `notify`; A-2, `cancelAppointment`, A-5 | 4.2, 6.3, 6.6 |
| FR-20 | Hesapsız müşteri adına randevu | `staffCreateAppointment`, `appointments.customer` | 4.2, 5 |
| FR-21 | Yeniden planlama | `rescheduleAppointment`, iki belgeli işlem | 6.3 |
| FR-22 | Randevu kuralları (minLead, maxAdvance, tampon, iptal süresi) | `salon.settings`, A-1 | 5, 6.2 |
| FR-23 | Gelmedi işaretleme ve müşteri bazlı sayı | `markNoShow`, sorguyla hesaplanan sayı | 4.2, 5 |
| FR-24 | Ayarlanabilir ve çoklu hatırlatma | `reminderOffsetsMin`, A-3 | 5, 6.4 |
| FR-25 | Onay modu (C) | `settings.approvalMode`, `status=pending`, onay anında A-3 | 5, 6.4 |
| FR-26 | Bekleme listesi (C) | Tasarlanmadı. C öncelikli, ilk sürüm planına alınırsa ayrıca ele alınır | 3.1 |
| FR-27 | SMS veya e-posta yedek hatırlatma (C) | Karar ertelendi (AS-06) | 3.1 |
| FR-28 | Cihaz takvimine ekleme (C) | Karar ertelendi (AS-06) | 3.1 |
| FR-29 | "Hesap silindi" etiketli anonim kayıt | A-5, K-6, `customerLabel` | 5, 6.1, 6.6 |
| NFR-01 | Android ve iOS'ta aynı işlevler | Tek Flutter uygulaması; taban Android 7.0 (API 24) ve iOS 15 | 3.2, 9 (M-1) |
| NFR-02 | 50 eşzamanlı denemede en fazla 1 kayıt | K-3, A-2 | 6.1, 9 (T-3) |
| NFR-03 | p95 ≤ 2 sn liste, ≤ 3 sn onay | A-1'in sabit okumaları, `minInstances=1`, aynı bölge | 7, 9 (P-1) |
| NFR-04 | Bildirim zamanlaması | K-5, olay içi gönderim | 6.1, 7, 9 (P-2) |
| NFR-05 | ≤ 5 adım, kullanılabilirlik testi | Müşteri randevu akışı | 4.1, 9 (U-1) |
| NFR-06 | Yetkilendirme ve gizlilik | Rules, K-2, K-7, randevudaki müşteri kopyası | 5, 8, 9 (T-4) |
| NFR-07 | 30 gün içinde başvuru ve silme | A-5 (eşzamanlı), `deletionJobs` ile yeniden deneme | 6.6 |
| NFR-08 | Veri en aza indirme, mağaza beyanları | En fazla 3 kayıt alanı, not alanı yok, veri envanteri | 5, 8 |
| NFR-09 | Erişilebilirlik | Flutter Semantics, test A-1t | 9 |
| NFR-10 | Türkçe, 24 saat, GG.AA.YYYY, salon saat dilimi | `startLocal`, Luxon, `flutter_localizations` | 3.1, 5 |
| NFR-11 | Bildirim içeriği kısıtı | `notify` şablonları, T-5 | 8, 9 |
| NFR-12 | Aylık %99,5 erişilebilirlik | Yönetilen Firebase servisleri. SLA doğrulanamadı (R-07, S-11) | 10 |
| NFR-13 | Bağlantı kesintisi davranışı | Firestore önbelleği [T-12], K-2 (kuyruk yok), 15 sn zaman aşımı | 4.1, 6.1, 9 (T-7) |
| NFR-14 | Anonimleştirmenin doğrulanması | A-5, T-6 | 6.6, 9 |

## 12. BA'ya geri sorular

- **S-01 (AS-08, acil):** Firestore konumu sonradan değiştirilemiyor ve listede Türkiye bölgesi yok [T-11]. Proje kurulmadan önce yurt dışı aktarım için hukuki dayanak ve tercih edilen bölge (veya bölge grubu) belirlenebilir mi? Geçici seçim `europe-west3`.
- **S-02 (AS-02):** Birincil kimlik e-posta mı, telefon mu? Telefonla doğrulamada SMS gönderim başına ücretlendiriliyor [T-14]. Geçici karar: e-posta ve şifre.
- **S-03 (AS-07):** Teknik taban Android 7.0 (API 24) ve iOS 15. Daha düşük sürümler Flutter ve Firebase tarafından desteklenmiyor [T-8, T-9, T-10]. Bu taban iş açısından kabul edilebilir mi?
- **S-04 (FR-18 b):** "Randevu salon tarafından değiştirildi" bildirimi tanımlı, ama salonun randevuyu değiştirmesini (saat veya çalışan değişikliği) tanımlayan bir FR yok. Salonun hangi değişiklikleri yapabileceği belirtilebilir mi? Tanımlanana kadar yalnızca salon iptali bildirimi uygulanır.
- **S-05 (FR-29, NFR-14):** NFR-14'te "arama ve dışa aktarım", FR-29'da "raporlar" geçiyor. Ancak arama, dışa aktarım veya rapor tanımlayan bir FR yok ve gelişmiş raporlama kapsam dışı. Bu ifadeler yalnızca ileride eklenecek işlevler için bir koşul mu, yoksa ilk sürümde bir arama veya dışa aktarım işlevi mi bekleniyor?
- **S-06 (AS-04):** "Fark etmez" atama kuralı ne olmalı? Geçici kural: o gün en az randevusu olan çalışan, eşitlikte sabit sıra.
- **S-07 (AS-17):** Başlangıç saati aralığı sabit mi, salon tarafından ayarlanabilir mi? Geçici: 15 dk, ayar alanı hazır.
- **S-08 (AS-15):** Hemen gönderilen hatırlatma oluşturma bildirimiyle birleştirilsin mi? Salon hatırlatma ayarını değiştirdiğinde mevcut randevular güncellensin mi? Geçici: ayrı gönderim, mevcut randevular değişmez.
- **S-09 (AS-18, FR-20):** Hesapsız müşteri için girilen ad ve telefon ne kadar saklanacak, silme talebi hangi kanaldan alınacak? Yönetici ekranında "bu müşterinin kişisel verisini anonimleştir" işlevi gerekiyor mu? Şu an bunu karşılayan bir FR yok.
- **S-10 (FR-17, FR-18):** Google Play hizmetleri olmayan Android cihazlarda FCM çalışmıyor [T-6]. Bu kullanıcıların bilgiyi yalnızca uygulama içinde görmesi kabul edilebilir mi?
- **S-11 (AS-10, NFR-12):** %99,5 hedefinin niteliği nedir? Firebase SLA bu çalışmada doğrulanamadı. Hedef sözleşmeye bağlıysa SLA ayrıca incelenmeli.
- **S-12 (AS-13 b):** Hukuk danışmanı, anonimleştirilmiş kaydın (tarih, saat, hizmet, çalışan) yeterli olmadığına karar verirse ne yapılacak? Seçenekler: zamanın gün düzeyine indirilmesi veya çalışan bilgisinin çıkarılması. Hangisi tercih edilir? Her biri US-12 KK-1'i etkiler.
- **S-13 (AS-03):** İptal süresi dolduktan sonra müşteri iptali tamamen mi engellenecek? Geçici: engellenir, iletişim bilgisi gösterilir (FR-12).
- **S-14 (AS-05):** Çalışan kendi saatlerini ve izinlerini düzenleyebilecek mi? Geçici: yalnızca yönetici düzenler. Yetki kuralı çalışana da açılabilecek şekilde tasarlandı.
- **S-15 (AS-16 ile ilişkili):** Cloud Functions için Blaze planı gerekiyor [T-14]. Firebase projesinin ve faturanın sahibi kim olacak? Bu taraf, mağaza geliştirici hesabını tutan tarafla aynı mı?
- **S-16 (AS-14):** İptal nedeni kategorileri ve zorunluluk onaylanıyor mu? Geçici olarak BA'nın önerdiği 4 kategori yapılandırıldı. İptal açıklaması için 200 karakter üst sınırı uygun mu?
