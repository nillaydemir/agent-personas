# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ta v2
- **Girdi:** outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run3.md

---

## 1. Özet

Bu doküman, tek bir salona ait randevu uygulamasının teknik çözümünü tanımlar. Uygulama Android ve iOS'ta çalışır. Girdi olarak kullanılan BA dokümanında FR-01–FR-29, NFR-01–NFR-15 ve US-01–US-12 tanımlıdır.

**Temel kararlar:**
- **İstemci:** Tek bir Flutter uygulaması yazılır. Müşteri, çalışan ve yönetici ekranları rol bazlı açılır (FR-04, NFR-01, NFR-15).
- **Arka uç:** Firebase kullanılır.
  - Kimlik doğrulama: Authentication, geçici olarak e-posta ve şifreyle (AS-02).
  - Veri: Cloud Firestore.
  - Kritik işlemler: Cloud Functions. Bunlar randevu oluşturma, taşıma, iptal, gelmedi işareti, hesap silme ve bildirim gönderimidir.
  - Push: Cloud Messaging (FCM).
  - Web: Firebase Hosting, hesap silme talebi sayfası ve yasal metinler için kullanılır.
- **Çift rezervasyonun önlenmesi (FR-11, NFR-02):** Her çalışan ve gün için tek bir "gün doluluk" dokümanı tutulur. Bu doküman sunucu tarafı bir Firestore işleminde (transaction) okunup güncellenir. Standart sürümde sunucu kitaplıkları okunan dokümanı kilitler, bu yüzden aynı çalışanın aynı günü için yapılan istekler sırayla işlenir (T-3).
- **Müsait saatler (FR-09):** İstemcide hesaplanır, böylece NFR-03'teki 2 saniye hedefi gözetilir. Kayıt anında sunucu aynı algoritmayla yeniden doğrular.
- **Hatırlatmalar (FR-18 c, FR-24, NFR-04):**
  - Planlı hatırlatmalar Firestore'da kayıt olarak tutulur ve dakikada bir çalışan zamanlanmış bir fonksiyonla gönderilir (T-9, T-10).
  - Olay bildirimleri ve hemen gönderilecek hatırlatmalar bir "giden kutusu" (outbox) dokümanıyla tetiklenir ve hemen gönderilir.
- **Hesap silme (FR-03, FR-29, NFR-07, NFR-14):** Sunucu fonksiyonu Seçenek A'yı uygular. Gelecekteki randevuları iptal eder, geçmiş randevulardaki müşteri bilgisini çıkarır, kişisel dokümanları ve Auth kaydını siler. Normal akışta işlem aynı istekte tamamlanır. Tamamlanamazsa günlük yeniden deneme işi devreye girer.
- **Platform tabanı:** Android 7.0 (API 24) ve iOS 15. Hem Flutter'ın (T-8) hem FlutterFire'ın (T-7) desteklediği en düşük sürümler bunlardır.

**Bekleyen kararlar:** Veri konumu (AS-08) proje kurulmadan önce karara bağlanmalıdır, çünkü Firestore konumu sonradan değiştirilemez (T-1). AS-02, AS-03, AS-04, AS-05 ve AS-15 için geçici kararlarla ilerlenmiştir. Bunlar ilgili bölümlerde "geçici" olarak işaretlidir.

## 2. Mimari genel bakış

```
┌──────────────────────────── Flutter uygulaması (Android 7.0+ / iOS 15+) ───────────────────────────┐
│  Müşteri ekranları │ Çalışan ekranları │ Yönetici ekranları   (rol: members/{uid}.role)            │
│  Müsait saat hesaplayıcı (saf Dart)  ·  Firestore dinleyicileri (çevrimdışı önbellek)                │
│  Bildirim izni yöneticisi · FCM token kaydı · Ön planda uygulama içi bildirim şeridi               │
└──────────┬──────────────────────────────┬───────────────────────────────┬─────────────────────────┘
           │ okuma (güvenlik kuralları)   │ callable fonksiyonlar          │ FCM push
           ▼                              ▼                               │
┌───────────────────┐        ┌──────────────────────────────────────┐    │
│  Cloud Firestore  │◄──────►│ Cloud Functions (Node.js 22, TS)     │────┘
│  (tek bölge,      │  tx    │ createAppointment / reschedule /     │
│   AS-08 bekliyor) │        │ cancel / markNoShow / requestDeletion│
└───────────────────┘        │ onOutboxCreated (anlık bildirim)     │
           ▲                 │ reminderScanner (her dakika)         │
           │                 │ deletionRetry (günlük)               │
┌───────────────────┐        └──────────────────────────────────────┘
│ Firebase Hosting  │  Web: hesap silme talebi sayfası (Firebase JS SDK ile giriş + aynı callable),
│                   │  gizlilik politikası ve aydınlatma metni URL'leri
└───────────────────┘
```

**İlkeler:**
- **Okumalar** istemciden doğrudan Firestore'a yapılır ve güvenlik kurallarıyla sınırlanır (NFR-06). Mobilde çevrimdışı önbellek varsayılan olarak açıktır, bu NFR-13'ü karşılar (T-12).
- **Randevu durumunu değiştiren yazmalar** yalnızca callable fonksiyonlarla yapılır. Bunun üç nedeni var:
  - Mobil SDK işlemleri iyimser eşzamanlılıkla çalışır ve çevrimdışıyken başarısız olur (T-2, T-3).
  - Çakışma kontrolü kurallarla ifade edilemeyecek kadar karmaşıktır.
  - Çevrimdışı yazmalar kuyruğa alınıp sonradan eşitlenir (T-12). NFR-13 ise işlemin kuyruğa alınmamasını istiyor. Callable çağrısı çevrimdışıyken hata döndürür ve uygulama bunu açık bir mesajla gösterir.
- **Yönetici ayarları** (salon profili, hizmetler, çalışan saatleri, kurallar) kurallarla korunan doğrudan yazmalarla yapılır. Bunların çakışma riski yoktur.
- **Sahiplik (NFR-15):** Firebase projesi ve fatura hesabı salona aittir. Mağaza hesapları ve APNs anahtarı salonun Apple hesabında oluşturulur. Geliştirme ekibine yetkiyi salon verir.

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Platform ve servis seçimleri

| Karar | Seçim | Gerekçe | İlgili gereksinim |
|---|---|---|---|
| İstemci çatısı | Flutter (görev kısıtı) | Tek kod tabanıyla Android ve iOS. Flutter destek aralığı Android API 24–37, iOS 15–27 (T-8) | NFR-01 |
| Uygulama sayısı | Tek uygulama, rol bazlı ekranlar | Ölçütler ve karşılaştırma 6.1'de (K-1) | FR-04, NFR-15 |
| En düşük sürümler | Android 7.0 (API 24), iOS 15 | FlutterFire "iOS 15" ve "Target API level 24 (Nougat) or higher" istiyor (T-7). Flutter API 23 ve öncesini, iOS 14 ve öncesini desteklemiyor (T-8). FCM istemcisi Android 7.0 ve Google Play hizmetleri gerektiriyor (T-4) | NFR-01, AS-07 |
| Kimlik doğrulama | Firebase Authentication, e-posta ve şifre (**geçici**, AS-02) | Şifre sıfırlama e-postası hazır (`sendPasswordResetEmail`, T-23). Telefon doğrulaması SMS maliyeti getirir (BA AS-02). 50 bin aylık aktif kullanıcıya kadar ücretsiz (T-11) | FR-01, NFR-08 |
| Veritabanı | Cloud Firestore (Standard sürüm) | Mobilde çevrimdışı önbellek varsayılan olarak açık (T-12). Sunucu kitaplıklarında kötümser (pessimistic) kilitli işlemler var (T-3). Bölgesel konumun aylık erişilebilirlik taahhüdü %99,99 (T-1) | FR-11, NFR-02, NFR-12, NFR-13 |
| Sunucu mantığı | Cloud Functions 2. nesil, Node.js 22, TypeScript | Node.js 22 destekleniyor (T-29). Callable fonksiyonlara Auth bilgisi otomatik eklenir (T-19). Blaze planı gerekir, faturası salona aittir (T-11, NFR-15) | FR-11, FR-12, FR-13, FR-21, FR-03 |
| Push | Firebase Cloud Messaging | Ücretsiz (T-11). Google Play hizmetleri olmayan cihazlarda çalışmaz (T-4). Bu durum paydaşça kabul edildi (Yanıt 12) | FR-18, FR-19, FR-17 |
| Zamanlama | `onSchedule` + Cloud Scheduler, dakikada bir | Cron biçiminin en ince çözünürlüğü 1 dakika (T-10). İş başına aylık 0,10 $, hesap başına 3 iş ücretsiz (T-9). Bu tasarımda 2 iş var | NFR-04, FR-18, FR-24, NFR-07 |
| Web sayfaları | Firebase Hosting | Hesap silme talebi bağlantısı ve gizlilik politikası URL'si mağaza politikalarınca zorunlu (BA K-10, K-11). Emulator Suite Hosting'i de kapsıyor (T-27) | FR-02, FR-03 |
| Veri ve fonksiyon bölgesi | Tek bölgeli Firestore. Fonksiyonlar aynı bölgede. **Konum AS-08'e bağlı** | Türkiye bölgesi yok. Konum sonradan değiştirilemiyor (T-1). Fonksiyonların Firestore'a yakın bölgede olması öneriliyor, yoksa gecikme ve maliyet artabilir (T-25). Ölçütler 6.7'de (K-7) | AS-08, NFR-03, NFR-12 |

### 3.2 Kütüphane ve paket kontrolü

Aşağıdaki bilgiler, ilgili belge sayfası bu çalışmada açılarak kontrol edilmiştir. "Belgede yok" ifadesi, açılan sayfada bilgi bulunmadığı anlamına gelir.

| Paket / SDK | Kontrol edilen sürüm | Amaç | (a) İzinler / beyanlar | (b) En düşük platform | (c) Belgedeki bilinen kısıtlar | Gereksinim |
|---|---|---|---|---|---|---|
| `firebase_core` (FlutterFire) | Sürüm pub.dev'de kontrol edilmedi. Diğer FlutterFire paketleriyle uyumlu sürüm kullanılır | Firebase başlatma | Belgede yok | iOS 15; Android API 24 / Android 7.0 (T-7) | Android emülatörü Google Play imajı kullanmalı (T-7) | NFR-01 |
| `firebase_auth` | 7.1.0, yayıncı firebase.google.com (T-15) | Kayıt, giriş, şifre sıfırlama, yeniden kimlik doğrulama | Belgede yok | T-7 ile aynı | Hesap silme gibi hassas işlemler yakın tarihli giriş ister, aksi hâlde `requires-recent-login` hatası verilir (T-23) | FR-01, FR-03 |
| `cloud_firestore` | 7.2.0, yayıncı firebase.google.com (T-14) | Okuma, dinleyiciler, yönetici ayar yazmaları | Belgede yok | T-7 ile aynı | Mobil SDK işlemleri iyimser eşzamanlılıkla çalışır (T-3) ve çevrimdışı başarısız olur (T-2). Çevrimdışı yazmalar kuyruğa alınır, aynı dokümanda son yazma kazanır (T-12). Önbellekte olmayan tek doküman okuması çevrimdışı hata verir (T-12) | NFR-06, NFR-13 |
| `cloud_functions` | 7.1.0, yayıncı firebase.google.com (T-16) | Callable çağrıları | Belgede yok | T-7 ile aynı | Varsayılan bölge us-central1, farklı bölge başlatmada ayarlanmalı. İstemci zaman aşımı varsayılan 70 saniye. Dart ile yazılan fonksiyon desteği deneysel (T-19) | FR-10–FR-13, FR-21, FR-23, FR-03 |
| `firebase_messaging` | 17.1.0, yayıncı firebase.google.com (T-13) | Push token ve bildirim alma | Android 13+: bildirim izni istenmeli (T-6; izin adı `POST_NOTIFICATIONS`, BA K-12). iOS: Push Notifications ve Background Modes (Background fetch, Remote notifications) yetenekleri, APNs kimlik doğrulama anahtarı, method swizzling (T-5) | Android 7.0 + Google Play hizmetleri (T-4, T-5); iOS 15 (T-7) | Google Play hizmetleri olmayan cihazda çalışmaz (T-4). Android 13+'ta kullanıcının izin konusunda karar verip vermediği belirlenemez (T-6). Ön plandaki bildirim varsayılan olarak görünmez (T-6). iOS'ta Cloud Messaging testi için fiziksel cihaz gerekir (T-7). UIScene kullanan iOS uygulamalarında `configureNotificationCenterDelegate()` çağrılmalı (T-13). APNs token'ı API isteklerinden önce hazır olmayabilir (T-5) | FR-16, FR-17, FR-18, FR-19 |
| `intl` | 0.20.3, yayıncı dart.dev (T-24) | Türkçe tarih ve saat biçimi (`dd.MM.yyyy`, `HH:mm`) | Belgede yok | Android, iOS desteklenir (T-24) | Tarihlerde saat dilimi desteği yok. `initializeDateFormatting` çağrılmalı (T-24) | NFR-10 |
| `timezone` | 0.11.2, yayıncı tools.dart.dev (T-26) | Salonun saat dilimine (`Europe/Istanbul`) göre slot üretimi ve gösterim | Belgede yok | Android, iOS desteklenir (T-26) | Cihaz saat dilimini algılamaz. Yerel konum `setLocalLocation` çağrılana kadar UTC'dir. Veritabanı betikle güncellenir (sürüm 2025c). "10y" varyantı yalnızca ±5 yılı kapsar (T-26). Bu yüzden varsayılan varyant kullanılır, geçmiş randevular da doğru gösterilir | FR-09, NFR-10 |
| Firebase JS SDK (Hosting sayfası) | Callable için en az 9.7.0 (T-19) | Web'de hesap silme talebi | Belgede yok | Tarayıcı | Web'de çevrimdışı kalıcılık varsayılan olarak kapalı (T-12). Bu sayfa için sorun değil | FR-03 |
| `firebase-functions` / `firebase-admin` (Node.js) | Node.js 22 çalışma ortamı (T-29) | Callable, Firestore tetikleyici, zamanlanmış fonksiyon, Auth yönetimi, FCM gönderimi | Proje IAM yetkisi (salon verir, NFR-15) | Node.js 20 veya 22. Node.js 18 kullanımdan kaldırılıyor, 14 ve 16 dağıtılamıyor (T-29) | Callable hataları `HttpsError` ile döndürülmeli, diğer hatalar istemciye `INTERNAL` olarak gider (T-19). `deleteUsers` toplu silme `onDelete` tetikleyicisini çalıştırmaz (T-20); bu tasarım o tetikleyiciyi kullanmaz | FR-03, FR-11, FR-18 |
| Luxon (Node.js) | Belgedeki saat dilimi bölümü kontrol edildi (T-18) | Sunucuda salon saat dilimine göre slot ve hatırlatma zamanı hesabı | Yok | Intl API'si olan ortam gerekir (T-18) | Ortam saat dilimini desteklemezse geçersiz tarih ("unsupported zone") döner. Yaz saati boşluğundaki saatler bir saat ileri kayar, belirsiz saatlerde davranış tanımsızdır (T-18). Önlem: fonksiyon açılışında saat dilimi doğrulanır (R-09) | FR-09, FR-18, FR-24 |

**Uygulama izinleri (özet):**
- Android'de yalnızca bildirim izni istenir (FR-16).
- iOS'ta yalnızca bildirim yetkisi istenir. Kullanıcıya sistem penceresi gösterilir (T-6).
- Konum, kişiler, kamera veya takvim izni istenmez.
- FR-28 (C) seçilirse takvim izni gerekebilir. Bu, ilgili paket seçildiğinde ayrıca kontrol edilecektir (AS-06).

### 3.3 Erişilebilirlik için mobil eşdeğerler (NFR-09)

| Ölçüt | BA hedefi | Mobil karar | Dayanak |
|---|---|---|---|
| Dokunma hedefi | 24×24 CSS piksel (WCAG 2.5.8) | Her iki platformda en az 48×48 mantıksal piksel (dp) | Android en az 48dp×48dp öneriyor (T-28). iOS için Apple'ın değeri bu çalışmada açılmadı. Android değeri iki platformda da uygulanır, WCAG hedefinden büyüktür |
| Kontrast | 4.5:1 | 4.5:1. 18sp ve üzeri (kalın yazıda 14sp ve üzeri) metinde 3:1 | T-28, BA K-15 |
| Yazı büyütme | %200 | Sistem yazı ölçeğine uyulur, metin kesilmez. En büyük ayarda ekran görüntüsü testi yapılır | NFR-09 |
| Ekran okuyucu | Ana ekranlar | Etkileşimli öğelere anlamlı etiketler verilir. TalkBack ve VoiceOver ile el ile test yapılır | NFR-09 |

### 3.4 Kaynaklar

Tüm kaynaklar bu çalışmada WebFetch ile açılıp okunmuştur. BA dokümanındaki K-xx kaynaklarıyla karışmaması için T-xx numaraları kullanılmıştır.

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| T-1 | Cloud Firestore locations | https://firebase.google.com/docs/firestore/locations | Bölge listesi, Türkiye bölgesi yok. "once you provision a database instance, you cannot change its location setting". Aylık erişilebilirlik taahhüdü: çok bölgeli %99,999, bölgesel %99,99 |
| T-2 | Firestore: Transactions and batched writes | https://firebase.google.com/docs/firestore/manage-data/transactions | "Transactions will fail when the client is offline." Kilit süresi 20 sn, işlem süresi sınırı 270 sn, okumalar yazmalardan önce gelmeli. Kurallarda atomik işlemde 20 doküman erişim çağrısı sınırı |
| T-3 | Firestore: Transaction data contention | https://firebase.google.com/docs/firestore/transaction-data-contention | "Standard edition uses pessimistic concurrency controls"; sunucu kitaplıklarında "transactions place locks on the documents they read". Mobil ve web SDK'ları iyimser eşzamanlılık kullanır. Çekişmede otomatik yeniden deneme, sonunda "ABORTED: Too much contention" |
| T-4 | Set up a Firebase Cloud Messaging client app on Android | https://firebase.google.com/docs/cloud-messaging/android/client | FCM istemcisi Android 7.0 veya üstü ve Google Play Store uygulaması yüklü cihaz gerektirir. "should always check the device for a compatible Google Play services APK" |
| T-5 | FCM: Flutter client setup | https://firebase.google.com/docs/cloud-messaging/flutter/client | "upload your APNs authentication key to Firebase"; push ve background modes yetenekleri; "method swizzling is required"; APNs token'ı istek öncesi hazır olmayabilir; Android 7.0 ve Google Play hizmetleri |
| T-6 | FCM: Receive messages in a Flutter app | https://firebase.google.com/docs/cloud-messaging/flutter/receive | İzin "On iOS, macOS, web and Android 13 (or newer)" için gerekli. "On Android versions 13 and higher, there is no way to determine if the user has chosen whether to grant/deny permission." Ön plandaki bildirim "won't display a visible notification by default" |
| T-7 | Add Firebase to your Flutter app | https://firebase.google.com/docs/flutter/setup | En düşük sürümler: "iOS 15"; Android "Target API level 24 (Nougat) or higher". Cloud Messaging için fiziksel Apple cihazı ve APNs anahtarı |
| T-8 | Flutter: Supported deployment platforms | https://docs.flutter.dev/reference/supported-platforms | Flutter 3.50 itibarıyla Android 24–37 ve iOS 15–27 destekleniyor. Android 23 ve öncesi ile iOS 14 ve öncesi desteklenmiyor |
| T-9 | Schedule functions | https://firebase.google.com/docs/functions/schedule-functions | `onSchedule` (firebase-functions/v2/scheduler). Scheduler işi aylık 0,10 $, hesap başına 3 iş ücretsiz |
| T-10 | Cloud Scheduler: Cron job format | https://docs.cloud.google.com/scheduler/docs/configuring/cron-job-schedules | "`* * * * *`: the job runs every minute". Saat dilimi ayarlanabilir, varsayılan UTC |
| T-11 | Firebase Pricing | https://firebase.google.com/pricing | Cloud Functions Spark planında yok, Blaze'de ayda 2 milyon çağrı ücretsiz. Firestore'da günlük 50 bin okuma, 20 bin yazma, 20 bin silme ve 1 GiB depolama ücretsiz. FCM ücretsiz. Auth'ta 50 bin aylık aktif kullanıcı ücretsiz |
| T-12 | Access data offline (Firestore) | https://firebase.google.com/docs/firestore/manage-data/enable-offline | "For Android and Apple platforms, offline persistence is enabled by default." Web'de kapalı. Çevrimdışı yazmalar yerelde tutulur ve bağlantı gelince eşitlenir |
| T-13 | pub.dev: firebase_messaging | https://pub.dev/packages/firebase_messaging | 17.1.0, firebase.google.com. UIScene için `configureNotificationCenterDelegate()` notu |
| T-14 | pub.dev: cloud_firestore | https://pub.dev/packages/cloud_firestore | 7.2.0, firebase.google.com |
| T-15 | pub.dev: firebase_auth | https://pub.dev/packages/firebase_auth | 7.1.0, firebase.google.com |
| T-16 | pub.dev: cloud_functions | https://pub.dev/packages/cloud_functions | 7.1.0, firebase.google.com |
| T-17 | Control access with custom claims | https://firebase.google.com/docs/auth/admin/custom-claims | Özel talepler (custom claims) yalnızca Admin SDK ile atanır, 1000 bayt sınırı var. İstemciye ID token yenilenince ulaşır: yeniden giriş, token süresinin dolması veya `getIdToken(true)` |
| T-18 | Luxon: Time zones and offsets | https://github.com/moment/luxon/blob/master/docs/zones.md | IANA bölgeleri için "abusing built-in Intl APIs". Desteklenmeyen bölgede 'unsupported zone'. Yaz saati boşluğu ve belirsiz saat davranışı |
| T-19 | Call functions from your app (callable) | https://firebase.google.com/docs/functions/callable | Auth token'ları otomatik eklenir. Hatalar `HttpsError` ile döner. Varsayılan bölge us-central1. İstemci zaman aşımı 70 sn. Web SDK'sı en az 9.7.0. Dart desteği deneysel |
| T-20 | Manage users (Admin SDK) | https://firebase.google.com/docs/auth/admin/manage-users | `updateUser(uid, {disabled: true})`, `deleteUser(uid)`. `deleteUsers` toplu silmesi `onDelete()` tetiklemez |
| T-21 | Writing conditions for Security Rules | https://firebase.google.com/docs/firestore/security/rules-conditions | `get()`/`exists()` sınırı: "10 for single-document requests and query requests", "20 for multi-document reads, transactions, and batched writes". Önbellekteki çağrılar sayılmaz |
| T-22 | Security Rules and Firebase Authentication | https://firebase.google.com/docs/rules/rules-and-auth | Özel taleplere kurallarda `auth.token` ile erişilir. Rol bazlı örnek |
| T-23 | Manage Users in Firebase (Flutter) | https://firebase.google.com/docs/auth/flutter/manage-users | `sendPasswordResetEmail()`. Hesap silme yakın tarihli giriş ister: `requires-recent-login` ve `reauthenticateWithCredential()` |
| T-24 | pub.dev: intl | https://pub.dev/packages/intl | 0.20.3, dart.dev. "Time zones are not supported in dates". `initializeDateFormatting` gerekli |
| T-25 | Cloud Functions locations | https://firebase.google.com/docs/functions/locations | Avrupa ve Orta Doğu'daki 2. nesil bölgeler. Firestore'dan farklı konumda "increased latency and billing costs" olabilir. Bölgeyi açıkça belirtme önerisi |
| T-26 | pub.dev: timezone | https://pub.dev/packages/timezone | 0.11.2, tools.dart.dev. `tz.getLocation('Europe/Istanbul')`. Cihaz saat dilimini algılamaz. Veritabanı varyantları ve güncelleme yöntemi |
| T-27 | Firebase Local Emulator Suite | https://firebase.google.com/docs/emulator-suite | Firestore, Auth, Hosting, Functions (beta) emülasyonu. "built for accuracy, not performance or security". FCM listede yok. İşlem ve eşzamanlılık davranışından söz edilmiyor |
| T-28 | Android: Make apps more accessible | https://developer.android.com/guide/topics/ui/accessibility/apps | "touch target size, of at least 48dpx48dp". 18sp altı metinde 4.5:1, diğerlerinde 3:1 kontrast |
| T-29 | Manage functions | https://firebase.google.com/docs/functions/manage-functions | Node.js 22 ve 20 destekleniyor, 18 kullanımdan kaldırılıyor. "A minimum number of instances kept running incur billing costs at idle rates". Belgedeki örnek: 256MiB fonksiyonda en az örnek sayısıyla aylık yaklaşık 3 $ |

**Açılmayan veya kullanılmayan kaynaklar:**
- Apple'ın dokunma hedefi değeri (Human Interface Guidelines) açılmadı. Bu yüzden 3.3'te iOS için ayrı bir değer yazılmadı.
- Cloud Functions ve FCM'nin hizmet düzeyi taahhütleri (SLA) bu çalışmada açılmadı (bkz. R-11, AS-10).
- Admin SDK'da token iptali (revoke) ve devre dışı bırakmanın mevcut ID token'larına etkisi T-20'de yer almıyor. Bu konu doğrulanmadı ve tasarımda kurallarla önlem alındı (R-06).

## 4. Bileşenler ve sorumlulukları

### 4.1 İstemci (Flutter)

| Bileşen | Sorumluluk | Gereksinim |
|---|---|---|
| `AuthModule` | Kayıt (en fazla 3 zorunlu alan: ad, e-posta, şifre), giriş, şifre sıfırlama (T-23). Kayıt ekranında aydınlatma metni ve gizlilik politikası bağlantıları gösterilir | FR-01, FR-02, NFR-08, US-01 |
| `RoleRouter` | `members/{uid}` dokümanındaki rol (`customer`, `staff`, `admin`) ile ilgili ekran setini açar. Durum `deleting` ise oturumu kapatır | FR-04, NFR-06 |
| `BookingFlow` | Akış dört adımdır: Hizmet → Çalışan ("Fark etmez" dahil) → Tarih ve saat → Onay. İşletme seçme adımı yoktur | FR-10, NFR-05, US-05 |
| `SlotCalculator` (saf Dart) | Müsait başlangıç saatlerini hesaplar (6.3). Sunucudaki TypeScript sürümüyle ortak test vektörlerini kullanır | FR-09, FR-22, NFR-03 |
| `MyAppointments` | Yaklaşan ve geçmiş randevular. Görünen durumlar: aktif, iptal, tamamlandı, gelmedi. Salon iptalinde kategori ve açıklama gösterilir. Çevrimdışıyken son önbellek gösterilir | FR-14, FR-17, NFR-13, US-07 KK-3 |
| `CancelReschedule` | Müşteri iptali ve taşıma. İptal süresi dolmuşsa iptal butonu yerine salonun iletişim bilgisi gösterilir (**geçici**, AS-03) | FR-12, FR-21, US-06, US-10 |
| `StaffCalendar` | Çalışan için kendi günlük ve haftalık takvimi. Yönetici için çalışan sütunlu görünüm. "Hesap silindi" etiketi gösterilir | FR-15, FR-29, US-04, US-12 |
| `AdminSettings` | Salon profili, kapalı günler, hizmetler, çalışanlar ve davet, çalışma saatleri, molalar, izinler, randevu kuralları, hatırlatma zamanları, onay modu | FR-05–FR-08, FR-22, FR-24, FR-25 |
| `SalonCancelDialog` | Sabit 4 kategoriden seçim zorunludur. İsteğe bağlı açıklama en fazla 200 karakterdir, kalan karakter sayacı ve sağlık bilgisi uyarısı gösterilir | FR-13, US-07 KK-1/4/5 |
| `PhoneBooking` | Hesapsız müşteri için ad ve telefonla randevu girişi (yönetici ve çalışan) | FR-20, US-09 |
| `NoShowAction` | "Gelmedi" işareti. Yönetici müşteri detayında gelmeme sayısını görür | FR-23, US-11 |
| `NotificationPermission` | İzin ilk randevu onayından sonra, gerekçe ekranıyla istenir. İznin istendiği `users/{uid}.notifPromptedAt` alanına yazılır, çünkü Android 13+'ta karar durumu belirlenemez (T-6). Push kullanılamıyorsa (izin yok veya token alınamıyor) "Bildirimleri aç" yolu ya da yalnızca uygulama içi bilgi gösterilir | FR-16, FR-17, US-08 KK-1/3/7 |
| `PushHandler` | FCM token'ını `users/{uid}/devices/{id}` altına kaydeder. Ön planda gelen bildirimi uygulama içi şeritle gösterir (T-6). Bildirime dokunulunca randevu detayını açar | FR-18, FR-19 |
| `AccountDeletion` | Silinecek ve anonim tutulacak veriler ile en geç tamamlanma tarihi gösterilir. Kullanıcı şifresiyle yeniden kimlik doğrular (T-23), ardından `requestAccountDeletion` çağrılır ve onay ekranı gösterilir | FR-03, NFR-07, US-02 |
| `LegalTexts` | Aydınlatma metni ve gizlilik politikası uygulama paketinde gömülü olarak bulunur. Aynı metinler aynı sürüm numarasıyla Hosting'de yayımlanır | FR-02 |

### 4.2 Sunucu (Cloud Functions, TypeScript)

| Fonksiyon | Tür | Sorumluluk | Gereksinim |
|---|---|---|---|
| `createAppointment` | Callable | Rol ve kural kontrolü, işlem içinde çakışma kontrolü (6.2), "Fark etmez" ataması (6.4), hatırlatma kayıtları ve outbox. İstemci tekrar denediğinde çift kayıt olmaması için idempotency anahtarı kullanılır | FR-10, FR-11, FR-20, FR-18 a/c, FR-19, NFR-02 |
| `rescheduleAppointment` | Callable | Tek işlemde eski ve yeni gün doluluk dokümanlarını okur. Yeni aralık uygunsa randevuyu yerinde günceller ve eski aralığı bırakır. Hatırlatmaları yeniden planlar | FR-21, US-10 |
| `cancelAppointment` | Callable | Müşteri iptalinde süre kontrolü yapılır. Salon iptalinde kategori zorunludur, açıklama ≤200 karakterdir. Aralığı bırakır, bekleyen hatırlatmaları iptal eder, outbox'a yazar | FR-12, FR-13, FR-18 b, FR-19 |
| `markNoShow` | Callable | Durumu `no_show` yapar. Kayıtlı müşteride `customerStats/{uid}.noShowCount` değerini bir artırır | FR-23 |
| `onOutboxCreated` | Firestore tetikleyici | Olay bildirimlerini ve hemen gönderilecek hatırlatmaları FCM ile gönderir. Geçersiz token'ları temizler, işlenen outbox dokümanını siler | FR-18, FR-19, NFR-04, NFR-11 |
| `reminderScanner` | Zamanlanmış, `* * * * *` (T-10) | `status=pending` ve `dueAt<=now` olan hatırlatmaları gönderir ve `sent` olarak işaretler | FR-18 c, FR-24, NFR-04 |
| `requestAccountDeletion` | Callable (uygulama ve web) | Silme ve anonimleştirme akışı (6.6) | FR-03, FR-29, NFR-07, NFR-14 |
| `deletionRetry` | Zamanlanmış, günlük | Tamamlanmamış silme taleplerini yeniden işler | NFR-07 |
| `inviteStaff` | Callable | Çalışan davetini oluşturur. Davet kabul edilince `members/{uid}` dokümanını `staff` rolüne bağlar | FR-07 |

### 4.3 Web (Firebase Hosting)

- `/hesap-sil`: Kullanıcı Firebase JS SDK ile giriş yapar ve `requestAccountDeletion` çağrılır. Şifresini unutan kullanıcı sıfırlama yolunu kullanır. Bu bağlantı mağaza sayfasına yazılır (FR-03, US-02 KK-3).
- `/gizlilik` ve `/aydinlatma`: Mağaza beyanları için URL'ler (FR-02, NFR-08).

## 5. Veri modeli

Tüm zaman damgaları UTC olarak `Timestamp` türünde saklanır. Gösterim ve slot üretimi salonun saat dilimiyle yapılır (`salon/main.timeZone = "Europe/Istanbul"`, NFR-10).

| Koleksiyon / doküman | Alanlar | Kişisel veri | Kim okur / yazar (kurallar) | Gereksinim |
|---|---|---|---|---|
| `salon/main` | name, address, phone, timeZone, weeklyHours, closedDates[], settings {minLeadMinutes, maxAdvanceDays=60, bufferMinutes, cancelCutoffMinutes=120, slotIntervalMinutes ∈ {5,10,15,20,30,60} varsayılan 15, reminderOffsetsMinutes=[1440], approvalMode="auto"} | Yok | Okuma: girişli kullanıcı. Yazma: admin | FR-05, FR-09, FR-12, FR-22, FR-24, FR-25 |
| `services/{id}` | name, durationMinutes, priceInfo, staffIds[], active. **Not alanı yok** | Yok | Okuma: girişli. Yazma: admin | FR-06, NFR-08 |
| `staff/{id}` | displayName, uid?, weeklyHours, breaks[], leaveDates[] (neden alanı yok), active | Çalışan adı | Okuma: girişli. Yazma: admin (**geçici**, AS-05) | FR-07, FR-08 |
| `members/{uid}` | role, staffId?, status (`active` / `deleting`) | Kullanıcı kimliği | Okuma: kendisi. Yazma: yalnızca sunucu | FR-04, NFR-06 |
| `users/{uid}` | name, email, phone?, createdAt, notifPromptedAt | Ad, e-posta, telefon | Okuma ve yazma: kendisi. Okuma: admin | FR-01, FR-16 |
| `users/{uid}/devices/{id}` | fcmToken, platform, updatedAt | Cihaz token'ı | Yazma: kendisi. Okuma: sunucu | FR-18, FR-19 |
| `customerStats/{uid}` | noShowCount | Kullanıcı kimliğiyle bağlı | Okuma: admin. Yazma: sunucu | FR-23 |
| `appointments/{id}` | staffId, serviceId, serviceName, durationMinutes, start, end, status (`active`, `pending_approval`, `cancelled_customer`, `cancelled_salon`, `no_show`), customer {uid?, name, phone?, email?} veya `null`, customerDeleted (bool), cancelCategory (4 sabit değerden biri), cancelNote (≤200), createdBy, idempotencyKey, createdAt, updatedAt, previousStart? | `customer` alanı | Okuma: müşteri kendi kaydını (`customer.uid == auth.uid`), çalışan kendi `staffId` kayıtlarını, admin tümünü. Yazma: yalnızca sunucu | FR-10–FR-14, FR-20, FR-21, FR-23, FR-29 |
| `staffDays/{staffId}_{YYYYMMDD}` | intervals [{apptId, startMin, endMin}] (salonun yerel gününde dakika) | Yok | Okuma: girişli (slot hesabı için). Yazma: yalnızca sunucu | FR-09, FR-11, NFR-02 |
| `reminders/{id}` | apptId, recipientUid, dueAt, status (`pending` / `sent` / `cancelled`) | Kullanıcı kimliği | Yalnızca sunucu | FR-18 c, FR-24 |
| `outbox/{id}` | type, recipientUids[], template, fields {salon, hizmet, tarih, saat, çalışan, kategori?}, apptId | Kullanıcı kimliği (gönderimden sonra silinir) | Yalnızca sunucu | FR-18, FR-19, NFR-11 |
| `deletionRequests/{id}` | uid (tamamlanınca silinir), status, requestedAt, dueBy (+30 gün), completedAt, anonymizedCount, cancelledCount | İşlem süresince kullanıcı kimliği | Yalnızca sunucu | FR-03, NFR-07, NFR-14 |

**Tasarım notları:**
- **"Tamamlandı" durumu saklanmaz.** Bitiş zamanı geçmiş `active` randevu istemcide "tamamlandı" olarak gösterilir. Böylece durumu güncelleyen ayrı bir iş gerekmez (FR-14).
- **Müşteri bilgisi randevuya kopyalanır** (`customer`). Çalışan ekranında ek okuma veya kurallarda ek `get()` gerekmez. Anonimleştirmede tek alan `null` yapılır (FR-29). Kopyalanan alanlar NFR-06'nın çalışana izin verdiği alanlardır: ad, iletişim, hizmet.
- **`staffDays` dokümanında kişisel veri yoktur.** Müşteriler bu dokümanı okuyarak slot hesaplar. Diğer müşterilerin kimliği görünmez (NFR-06).
- **İptal kategorileri kodda sabit bir sözlüktür.** Yönetici düzenleyemez (FR-13, US-07 KK-4).
- **Bileşik indeksler:** `appointments(customer.uid, start)`, `appointments(staffId, start)`, `reminders(status, dueAt)`, `reminders(apptId)`.

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 K-1: Uygulama yapısı

**Seçim ölçütleri:** (Ö1) Tek mağaza kaydı ve salonun hesabıyla yayın kolaylığı (NFR-15, Apple 4.3(a), BA K-9). (Ö2) Rol ayrımının güvenliği (FR-04, NFR-06). (Ö3) Ürün büyüklüğüne uygunluk (kapsam kuralı).

| Seçenek | Ö1 | Ö2 | Ö3 |
|---|---|---|---|
| **A. Tek uygulama, rol bazlı ekranlar** | Her mağazada tek kayıt | Yetki sunucu kurallarıyla uygulanır, ekran gizlemeye dayanmaz | Tek kod tabanı ve tek yayın süreci |
| B. Müşteri ve salon için iki ayrı uygulama | İki kayıt, iki inceleme süreci | Aynı (kurallar yine gerekli) | Yayın ve bakım işi iki katına çıkar |

**Karar: A.**

### 6.2 K-2: Çift rezervasyonun önlenmesi

**Seçim ölçütleri:**
- (Ö1) Eşzamanlı isteklerde en fazla bir kayıt (FR-11, NFR-02).
- (Ö2) Herhangi bir hizmet süresi ve değişebilen başlangıç aralığıyla çalışması (FR-06, FR-22, US-11 KK-3).
- (Ö3) Taşımanın atomik olması: yeni saat alınmadan eski saat bırakılmaz (FR-21).
- (Ö4) Basitlik ve test edilebilirlik.

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **A. Çalışan-gün doluluk dokümanı (`staffDays`), sunucu işleminde oku-kontrol et-yaz** | Standart sürümde sunucu işlemi okunan dokümanı kilitler (T-3). Aynı çalışan ve gün için istekler sırayla işlenir | Aralıklar dakika olarak saklanır, süre ve aralıktan bağımsızdır | İki gün dokümanı aynı işlemde okunur ve yazılır | Tek doküman, tek kural |
| B. Birim zaman kilit dokümanları (ör. çalışan başına her 5 dakika için bir doküman, `create` ile) | Var olan dokümanı yeniden oluşturma hatası çakışmayı yakalar | Süre birimin katı değilse yuvarlama gerekir. Aralık ayarı değişince birim uyumsuzlaşabilir | 60 dakikalık bir randevu 12 doküman demektir, taşıma çok sayıda doküman ister | Doküman sayısı ve temizleme yükü artar |
| C. İşlem içinde çakışan randevu sorgusu | Sorgu sonucunda olmayan yeni kayıtların (phantom) kilitlenip kilitlenmediği belgede açık değil (T-3) | Uygun | Uygun | Doğruluk belgeyle kanıtlanamıyor |

**Karar: A.**
- İşlem sunucuda (Admin SDK) çalışır. İstemci SDK'sı iyimser çalıştığı ve çevrimdışı başarısız olduğu için kullanılmaz (T-2, T-3).
- Çekişmede kitaplık otomatik olarak yeniden dener. Yeniden denemeler tükenirse `ABORTED` hatası alınır (T-3). İstemci bu durumda "Bu saat az önce doldu" mesajını ve güncel listeyi gösterir (US-05 KK-4).
- Tek salon ölçeğinde bir çalışanın bir gününe gelen eşzamanlı istek sayısı düşüktür. Yine de bu, NFR-02 testindeki 50 eşzamanlı denemeyle doğrulanır (bölüm 9).

**İşlem adımları (`createAppointment`):**
1. `salon/main`, `services/{id}`, `staff/{id}` ve `staffDays/{staff}_{gün}` okunur. "Fark etmez" seçilmişse hizmeti verebilen tüm çalışanların gün dokümanları okunur.
2. 6.3'teki algoritmayla istenen başlangıç saatinin geçerliliği doğrulanır. Kontrol edilenler: çalışma saati, mola, izin, kapalı gün, en kısa önceden süre, en ileri tarih, başlangıç aralığına hizalanma ve çakışma.
3. Yazılanlar: `appointments` (onay modu otomatikse `active`, manuelse `pending_approval`), `staffDays` aralık eklemesi, `reminders` (6.5), `outbox` (müşteriye "oluşturuldu", çalışana ve yöneticilere "yeni randevu").

### 6.3 K-3: Müsait saat hesabı ve hesaplama yeri

**Algoritma** (salonun saat diliminde, gün `d`, çalışan `s`, hizmet süresi `D`, tampon `B`, başlangıç aralığı `I`):
1. Gün kapalıysa veya çalışan izinliyse sonuç boş listedir.
2. Çalışma pencereleri hesaplanır: salon saatleri ∩ çalışan saatleri, molalar çıkarılır.
3. Dolu aralıklar: her randevu için `[start, end + B)` (**geçici yorum**, S-03).
4. Aday saatler, gece yarısından itibaren `I`'nın katlarıdır. Böylece 30 dakikalık aralıkta yalnızca :00 ve :30 üretilir (US-11 KK-3).
5. Bir aday `t` şu koşullarda geçerlidir:
   - `[t, t+D)` tamamen bir çalışma penceresinin içindedir. Molaya veya kapanışa taşan saat gösterilmez (FR-09, US-05 KK-1).
   - `[t, t+D+B)` hiçbir dolu aralıkla kesişmez.
   - `t ≥ şimdi + minLead` (US-11 KK-1).
   - `d ≤ bugün + maxAdvanceDays`.
6. "Fark etmez" seçilmişse sonuç, uygun çalışanların listelerinin birleşimidir.

Karmaşıklık: Gün başına aday sayısı (≤ 1440/I) × dolu aralık sayısıdır. Tek salon için önemsizdir.

**Hesaplama yeri. Seçim ölçütleri:** (Ö1) 4G'de %95 ile 2 sn (NFR-03). (Ö2) Kayıt doğruluğu (FR-11). (Ö3) Diğer müşterilerin verisinin görünmemesi (NFR-06). (Ö4) Çevrimdışı davranış (NFR-13). (Ö5) Maliyet (AS-19).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. İstemcide hesap, sunucuda yeniden doğrulama** | Veri dinleyicilerle önceden yüklenir. Seçimden sonra ağ çağrısı yok, soğuk başlatma riski yok | Nihai karar sunucu işleminde verilir | `staffDays` kişisel veri içermez | Önbellekteki veriyle liste görünür. Kayıt yine engellenir | Fonksiyon çağrısı yalnızca kayıtta yapılır |
| B. Her listeleme için sunucu callable'ı | Her seçimde ağ çağrısı yapılır, soğuk başlatma olabilir (T-29) | Aynı | Uygun | Çevrimdışıyken liste yok | Listeleme başına çağrı |

**Karar: A.**
- Algoritma Dart ve TypeScript'te iki kez yazılır. Dart ile fonksiyon yazma desteği deneysel olduğu için tek dil seçeneği kullanılmaz (T-19).
- Tutarlılık, iki uygulamanın da çalıştırdığı ortak JSON test vektörleriyle sağlanır (R-02).
- İstemci listesinde görünüp sunucuda reddedilen saat için "Bu saat az önce doldu" akışı kullanılır.

### 6.4 "Fark etmez" ataması (**geçici**, AS-04)

- O gün en az randevusu olan uygun çalışan seçilir. Eşitlikte çalışan kimliğinin alfabetik sırası kullanılır, böylece sonuç belirlenimli olur.
- Atama sunucu işlemi içinde yapılır, istemcinin tahmini bağlayıcı değildir. Atanan çalışan onayda gösterilir (US-05 KK-5).
- AS-04 farklı yanıtlanırsa yalnızca bu seçim fonksiyonu değişir.

### 6.5 K-4: Hatırlatma ve bildirim gönderimi

**Seçim ölçütleri:**
- (Ö1) Planlı hatırlatmada en fazla ±5 dk sapma, olay ve anlık hatırlatmada 1 dk (NFR-04).
- (Ö2) İptal ve taşımada eski hatırlatmanın gitmemesi (US-08 KK-4, US-10 KK-3).
- (Ö3) Salon iptali gibi başka cihazdan yapılan değişikliklerin yansıması (FR-18 b).
- (Ö4) Maliyet (AS-19).
- (Ö5) Bildirim içeriğinin sunucuda denetlenmesi (NFR-11).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. Firestore'da hatırlatma kaydı + dakikada bir tarayıcı; olaylar için outbox tetikleyicisi** | Tarayıcı dakikada bir çalışır (T-10), sapma ±5 dk sınırının çok altında kalır | Kayıt `cancelled` yapılır, tarayıcı göndermez | Sunucu tek kaynaktır | Ayda 43.200 çağrı (60×24×30, kaynaktaki verilerden hesaplandı). Blaze'deki 2 milyon ücretsiz çağrının içinde (T-11). 1 Scheduler işi ücretsiz kotada (T-9) | Şablon sunucuda |
| B. Cihazda yerel bildirim planlama | Cihaz saatine bağlı | Salonun yaptığı iptal, uygulama açılana kadar cihaza yansımaz. İptal edilmiş randevuya hatırlatma gidebilir (US-08 KK-4 ihlali) | Yansımaz | Sunucu maliyeti yok | İçerik istemcide |

**Karar: A.** Seçenek B Ö2 ve Ö3'ü karşılamadığı için elendi.

**Hatırlatma planlama kuralı** (oluşturma ve taşımada; FR-18 c, FR-24, V-14):
1. Salonun `reminderOffsetsMinutes` listesinden her ofset için `dueAt = start − ofset` hesaplanır.
2. `dueAt > şimdi` olanlar `pending` kaydı olarak yazılır.
3. `dueAt ≤ şimdi` olanlar varsa yalnızca **bir** anlık hatırlatma outbox'a yazılır. Bu, oluşturma bildiriminden ayrı bir bildirimdir (V-14, US-08 KK-5/6).
4. Taşımada eski kayıtlar `cancelled` yapılır ve plan yeni saate göre yeniden kurulur (US-10 KK-3).
5. Manuel onay modunda (FR-25) planlama onay anında yapılır (V-14).
6. Salon hatırlatma ayarını değiştirdiğinde mevcut kayıtlar değişmez (**geçici**, AS-15 b).

**Bildirim içeriği (NFR-11):**
- Şablonlar sunucuda sabittir. Kullanılan alanlar: salon, hizmet, tarih (GG.AA.YYYY), saat (SS:dd), çalışan ve iptalde kategori etiketi.
- `cancelNote` hiçbir şablona bağlanmaz. Veri yükünde yalnızca `apptId` gider ve bildirime dokunulunca detay ekranı açılır.
- Kampanya içeriği yoktur.

**Push alamayan cihazlar (FR-17, Yanıt 12):** Ayrı bir kanal kurulmaz. Randevu detayı ve yaklaşan randevu kartı, hatırlatma içeriğinin aynısını uygulama içinde gösterir.

### 6.6 K-5: Hesap silme ve anonimleştirme

**Seçim ölçütleri:**
- (Ö1) Randevu kayıtlarının kalıp kişisel tanımlayıcıların çıkarılması (FR-29, Seçenek A).
- (Ö2) Tüm kişisel verinin doğrulanabilir şekilde kaldırılması (NFR-14).
- (Ö3) Gelecekteki randevuların iptali ve salona bildirim (FR-19, US-02 KK-2).
- (Ö4) 30 gün içinde tamamlanma ve onay (NFR-07, FR-03).
- (Ö5) Uygulamaya erişimi olmayan kullanıcının web'den talep gönderebilmesi (FR-03, US-02 KK-3).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. Sunucu fonksiyonu (`requestAccountDeletion`), adım adım ve tekrar çalıştırılabilir (idempotent) + günlük yeniden deneme** | Alan düzeyinde `customer=null` | Tek fonksiyonda listelenmiş adımlar test edilebilir | Aynı işlem iptal mantığını kullanır | Normalde anlık. Hata olursa yeniden deneme, son tarih `dueBy` | Web sayfası aynı callable'ı çağırır |
| B. İstemciden doğrudan silme | Kurallar diğer kullanıcıların randevularına yazmayı engeller, ya da gevşetilmesi gerekir (NFR-06 riski) | İstemci yarıda kalırsa veri kalır | Çakışma mantığı istemcide olmaz | Yeniden deneme yok | Web'de ayrıca yazılmalı |

**Karar: A.**

**Akış:**
1. İstemci kullanıcıdan şifresiyle yeniden kimlik doğrulamasını ister (T-23), sonra callable'ı çağırır.
2. Sunucu şunları yapar:
   - `deletionRequests` kaydını oluşturur (`dueBy = şimdi + 30 gün`).
   - `members/{uid}.status = deleting` yapar.
   - Auth kaydını `disabled: true` yapar (T-20). Bundan sonra giriş yapılamaz (US-02 KK-2). Mevcut token'lar için kurallar `deleting` durumunu reddeder (R-06).
3. Gelecekteki her `active` randevu 6.2'deki işlemle `cancelled_customer` yapılır, aralığı bırakılır, hatırlatmaları iptal edilir. Outbox'a çalışan ve yönetici için iptal bildirimi yazılır (FR-19).
4. Geçmiş randevular toplu yazmayla güncellenir: `customer=null`, `customerDeleted=true`. Sayısı `anonymizedCount` alanına yazılır.
5. Silinenler: `users/{uid}` ve `devices` alt koleksiyonu, `customerStats/{uid}`, `members/{uid}`, `recipientUid=uid` olan `reminders` ve `outbox` kayıtları.
6. `deleteUser(uid)` çağrılır (T-20).
7. `deletionRequests` güncellenir: `status=completed`, `completedAt` yazılır, `uid` alanı silinir.
8. Callable başarıyla döner ve istemci onay ekranını gösterir (FR-03, US-02 KK-4).

**Hata durumu:**
- Her adım tekrar çalıştırılabilir. Örneğin zaten `null` olan alan yeniden yazılır.
- `deletionRetry` günlük çalışır ve `dueBy` öncesinde tamamlanmamış talepleri bitirir.
- Asenkron tamamlanmada onayın kullanıcıya hangi kanaldan iletileceği açıktır (S-02).

**Kayıt dışı kopyalar:** Fonksiyon günlüklerine (log) ad, e-posta, telefon ve uid yazılmaz. Yalnızca `deletionRequests` kimliği yazılır (NFR-14).

### 6.7 K-6: Rol ve yetki kontrolü

**Seçim ölçütleri:** (Ö1) Çalışan çıkarılınca veya hesap silinince erişimin hemen kesilmesi (FR-07, NFR-06, NFR-07). (Ö2) Kural karmaşıklığı ve `get()` sınırları (T-21).

| Seçenek | Ö1 | Ö2 |
|---|---|---|
| **A. `members/{uid}` dokümanındaki rol, kurallarda `get()` ile okunur** | Değişiklik hemen etkili olur | İstek başına 1 `get()` çağrısı. Sınır tek doküman ve sorgu için 10, işlemler için 20 (T-21) |
| B. Auth özel talepleri (custom claims) | Değişiklik token yenilenene kadar istemciye ulaşmaz (T-17) | Ek okuma yok (T-22) |

**Karar: A.** Ö1 doğrudan NFR-06'ya bağlıdır. Ek `get()` maliyeti tek salon ölçeğinde düşüktür.

**Kural özeti:**
- Müşteri: yalnızca `customer.uid == auth.uid` olan randevuları okur.
- Çalışan: yalnızca `staffId == members.staffId` olan randevuları okur.
- Admin: tümünü okur.
- `appointments`, `staffDays`, `reminders`, `outbox`, `customerStats`, `members` koleksiyonlarına istemci yazamaz.

### 6.8 K-7: Veri konumu (AS-08 ile birlikte karara bağlanacak)

**Seçim ölçütleri:**
- (Ö1) KVKK md. 9 kapsamında yurt dışına aktarım dayanağı (AS-08, hukuki karar).
- (Ö2) %99,5 erişilebilirlik hedefi (NFR-12).
- (Ö3) Türkiye'den gecikme (NFR-03).
- (Ö4) Konumun sonradan değiştirilememesi (T-1).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **Tek bölge (ör. bir Avrupa veya Orta Doğu bölgesi)** | Türkiye bölgesi yok, her seçenek yurt dışıdır (T-1) | Taahhüt %99,99 (T-1) | Ölçülerek seçilir | Kalıcı |
| Çok bölgeli `eur3` | Aynı | Taahhüt %99,999 (T-1) | Ölçülerek seçilir | Kalıcı |

**Karar:** Tek bölge yeterlidir, çünkü iki seçenek de NFR-12'nin üstündedir ve fazlası ürün büyüklüğüne uygun değildir.
- Hangi bölgenin seçileceği AS-08'in hukuki yanıtından sonra, aday bölgelerde Türkiye'den yapılacak gecikme ölçümüyle belirlenir.
- Fonksiyonlar aynı bölgeye dağıtılır (T-25) ve istemci bu bölgeyle başlatılır (T-19).
- **Proje AS-08 yanıtlanmadan kurulmaz.**

## 7. Performans

| Hedef | Tasarım önlemi | Doğrulama | Gereksinim |
|---|---|---|---|
| Müsait saat listesi: 4G'de %95 ile ≤2 sn | Salon, hizmet ve çalışan verileri uygulama açılışında dinleyiciyle yüklenir. Seçilen gün için yalnızca ilgili `staffDays` dokümanları okunur. Hesap yerelde yapılır (6.3) | Gerçek cihazda 4G ağ profiliyle 100 ölçüm, %95 değeri raporlanır | NFR-03 |
| Randevu onayı ≤3 sn | Tek callable ve tek işlem. Fonksiyon ile Firestore aynı bölgededir (T-25). Soğuk başlatma riski için önce en az örnek sayısı 0 ile ölçülür. Hedef tutmazsa `createAppointment` için en az 1 örnek açılır. Bunun boşta faturalanan bir maliyeti vardır, belgedeki örnek aylık yaklaşık 3 $'dır (T-29) ve bütçe AS-19'a bağlıdır | Soğuk ve sıcak çağrı ölçümleri ayrı raporlanır | NFR-03, AS-19 |
| Olay bildirimi ≤1 dk, planlı ±5 dk | Outbox tetikleyicisi anında çalışır. Tarayıcı dakikada bir çalışır (T-10) | Gönderim anı ile olay ve `dueAt` farkı günlüklerden ölçülür. Cihaza teslim ölçüm dışıdır (NFR-04) | NFR-04 |
| Eşzamanlılık | 6.2'deki kilitli işlem | 50 eşzamanlı istek testi (bölüm 9) | NFR-02 |
| Kullanılabilirlik ≤5 adım | Akış 4 adımdır: Hizmet → Çalışan → Tarih ve saat → Onay | 5 katılımcıyla test | NFR-05 |
| Erişilebilirlik süresi %99,5 | Firestore bölgesel taahhüdü %99,99 (T-1). Cloud Functions ve FCM taahhütleri bu çalışmada doğrulanmadı | AS-10 yanıtına göre izleme sorumlusu belirlenir | NFR-12 |
| Maliyet | Ücretsiz kotalar: Firestore'da günlük 50 bin okuma ve 20 bin yazma, Functions'ta ayda 2 milyon çağrı (T-11), 3 Scheduler işi (T-9). Hacim tahmini BA dokümanında yok (S-05) | Bütçe uyarısı AS-19'a bağlı | NFR-15, AS-19 |

## 8. Güvenlik ve gizlilik

- **Yetkilendirme (NFR-06):** 6.7'deki kurallar uygulanır. Durum değiştiren tüm yazmalar sunucudadır ve her callable `auth.uid`, rol ve kaynak sahipliğini yeniden kontrol eder (T-19).
- **Veri en aza indirme (NFR-08):**
  - Kayıtta 3 alan istenir.
  - Hizmet ve müşteri için not alanı yoktur.
  - Tek serbest metin alanı `cancelNote`'tur. 200 karakter sınırı istemcide (sayaç ve giriş engeli), callable'da ve kuralda (`size() <= 200`) uygulanır (US-07 KK-5).
  - Çalışan izinlerinde neden alanı yoktur.
- **Bildirim gizliliği (NFR-11, Apple 4.5.4):** Serbest metin hiçbir şablonda yoktur (6.5).
- **Hesap silme ve anonimlik (FR-03, FR-29, NFR-14):**
  - 6.6'daki akış uygulanır. Kişisel veri yalnızca `users`, `devices`, `customerStats`, `members`, `appointments.customer`, `reminders.recipientUid`, `outbox.recipientUids` ve geçici olarak `deletionRequests.uid` alanlarında bulunur.
  - Bu liste NFR-14 testinin taranacak alan envanteridir. Yeni alan eklendiğinde test güncellenir.
  - İlk sürümde Firestore yedeği veya zaman içinde geri alma (PITR) açılmaz. Açılırsa NFR-14 kapsamı yeniden değerlendirilir (S-06).
- **Hesapsız müşteri verisi (FR-20):** Ad ve telefon `appointments.customer` içinde tutulur. Saklama süresi ve silme yolu AS-18'e bağlıdır. Model, ileride aynı anonimleştirme adımının bu kayıtlara uygulanmasına izin verir.
- **Sahiplik ve erişim (NFR-15):** Firebase projesi, fatura hesabı, Apple ve Google geliştirici hesapları ve APNs anahtarı salona aittir. Geliştirme ekibine rol bazlı proje yetkisini salon verir ve geri alabilir (V-20).
- **Mağaza beyanları (NFR-08, FR-02):** Data safety ve App Store gizlilik beyanında şu veriler yer alır: ad, e-posta, telefon, cihaz push token'ı ve randevu geçmişi. Silme davranışı ve anonim tutulan kayıtlar da beyan edilir. Kullanılan üçüncü taraf kodlar FlutterFire paketleridir.
- **Günlükler:** Fonksiyon günlüklerine kişisel veri yazılmaz. Hata günlüklerinde yalnızca doküman kimlikleri bulunur.

## 9. Test stratejisi

| Katman | Kapsam | Araç / yöntem | Gereksinim / US |
|---|---|---|---|
| Birim (Dart ve TS) | Slot algoritması: molaya ve kapanışa taşma, izin, kapalı gün, tampon, en kısa önceden süre, 60 gün sınırı, aralık 5/15/30/60, aralık değişiminde eski randevunun korunması. **İki dilde aynı JSON vektörleri kullanılır** | Dart `test`, TS test çatısı | FR-09, FR-22, US-04 KK-1/2, US-05 KK-1/2, US-11 KK-1/3 |
| Birim (TS) | Hatırlatma planlama kuralı: 24 sa ve 2 sa ofsetleri, 3 sa ve 1 sa önce oluşturma, taşıma, iptal | TS testi | FR-18 c, FR-24, US-08 KK-4/5/6, US-10 KK-3 |
| Kural testleri | Rol matrisi: müşteri başkasının randevusunu okuyamaz, çalışan başka çalışanın randevusunu okuyamaz, istemci `appointments` koleksiyonuna yazamaz, `deleting` durumundaki kullanıcı reddedilir. Beklenen başarılı yetkisiz erişim sayısı 0 | Firebase Emulator Suite, Firestore ve Auth emülatörleri (T-27) | NFR-06, FR-04, US-03 KK-3 |
| Fonksiyon entegrasyonu | Oluşturma, iptal (kategori zorunlu, 201 karakter reddi), taşıma atomikliği, gelmedi işareti, silme akışının her adımı ve yeniden çalıştırılabilirliği | Emulator Suite, Functions emülatörü beta (T-27) | FR-10–FR-13, FR-21, FR-23, FR-03, US-06, US-07, US-09, US-10 |
| Eşzamanlılık | Aynı çalışan ve aynı slot için 50 eşzamanlı `createAppointment`. Sonuç: 1 başarı, 49 "doldu". Taşıma ile oluşturma yarışı | **Gerçek bir test projesinde** yapılır, çünkü emülatör belgesi eşzamanlılık davranışından söz etmiyor ve performans için tasarlanmadığını söylüyor (T-27) | NFR-02, FR-11, US-05 KK-4 |
| Anonimleştirme doğrulaması | Silme öncesi kullanıcının geçmiş randevu sayısı kaydedilir. Silme sonrası tüm koleksiyonlar Admin SDK ile dışa alınır ve 8. bölümdeki envanterde test kullanıcısının benzersiz adı, e-postası, telefonu ve uid'si aranır, beklenen eşleşme 0'dır. "Hesap silindi" etiketli kayıt sayısı ve tarih aralığındaki toplam sayı karşılaştırılır | Test projesinde betik | NFR-14, FR-29, US-02 KK-5, US-12 |
| Bildirim | İzin akışı (Android 13+ ve iOS fiziksel cihaz, T-7), ön plan şeridi, dokununca detay açılması, şablonlarda serbest metin ve kampanya içeriği sayısı 0 | El ile test + şablon birim testi. FCM emülatörde yok (T-27), gerçek projede denenir | FR-16, FR-18, FR-19, NFR-11, US-07 KK-2, US-08 KK-1/2 |
| Push'suz cihaz | Google Play hizmetleri olmayan Android cihaz veya emülatör imajında oluşturma, iptal ve salon iptali nedeninin görünmesi | El ile test | FR-17, US-08 KK-7, US-07 KK-3 |
| Çevrimdışı | Uçak modunda liste görünür, oluşturma ve iptal açık hata mesajı verir, kuyruk oluşmaz | El ile test | NFR-13 |
| Performans | 7. bölümdeki ölçümler | Gerçek cihaz, 4G profili | NFR-03, NFR-04 |
| Erişilebilirlik | Kontrast, en büyük yazı boyutu, 48dp hedefler, TalkBack ve VoiceOver | Android Accessibility Scanner (T-28), el ile test | NFR-09 |
| Kabul | Tüm M gereksinimleri iki platformda; Türkçe metinler, 24 saat biçimi, GG.AA.YYYY | US kabul kriterleri listesi | NFR-01, NFR-10, US-01–US-12 |
| Yayın kontrolü | Mağaza satıcı adı, proje ve fatura sahibi, D-U-N-S ve kapalı test süreleri | Yayın kontrol listesi | NFR-15 |

## 10. Riskler

| # | Risk | Etki | Önlem | İlgili |
|---|---|---|---|---|
| R-01 | Firestore konumu sonradan değiştirilemiyor ve Türkiye bölgesi yok (T-1). AS-08 yanıtlanmadan proje kurulamaz | Geliştirme başlangıcı gecikir | Geliştirme emülatörde başlar. Proje AS-08'den sonra kurulur | AS-08, K-7 |
| R-02 | Slot algoritmasının Dart ve TS sürümleri farklılaşabilir | İstemci gösterdiği bir saati sunucuda kaybeder ya da tersi olur | Ortak test vektörleri kullanılır, sunucu tek yetkili karar vericidir | FR-09, NFR-03 |
| R-03 | Callable soğuk başlatması 3 sn hedefini aşabilir | NFR-03 karşılanmaz | Ölçüm yapılır, gerekirse en az 1 örnek açılır (maliyetli, T-29) | NFR-03, AS-19 |
| R-04 | Google Play hizmetleri olmayan cihazlar push alamaz (T-4) | Kullanıcı hatırlatmayı yalnızca uygulamayı açınca görür | Paydaşça kabul edildi (Yanıt 12). Uygulama içi bilgi eksiksiz tutulur | FR-17 |
| R-05 | Android 13+'ta izin kararı belirlenemiyor (T-6) | İzin isteği tekrar tekrar gösterilebilir | `notifPromptedAt` alanıyla izin yalnızca bir kez istenir, sonrasında ayarlara yönlendirilir | FR-16 |
| R-06 | Devre dışı bırakılan kullanıcının mevcut ID token'ının ne zamana kadar geçerli kaldığı doğrulanmadı | Silme sonrası kısa süreli okuma | Kurallar `members.status == deleting` olan kullanıcıyı reddeder. `members` dokümanı silindikten sonra kurallar rolsüz kullanıcıya erişim vermez | NFR-06, NFR-07 |
| R-07 | Anonimliğin hukuken yetersiz bulunması | Veri modeli değişir (ör. saatin gün düzeyine indirilmesi) | Anonimleştirme tek fonksiyonda toplandı, yeni adım eklenebilir. US-12 testleri güncellenir | AS-13, FR-29 |
| R-08 | Eşzamanlılık emülatörde doğrulanamıyor (T-27) | NFR-02 yanlış güvenceyle geçebilir | Test gerçek bir test projesinde yapılır | NFR-02 |
| R-09 | Luxon desteklenmeyen saat diliminde geçersiz tarih döndürür (T-18) | Yanlış slot veya hatırlatma zamanı | Fonksiyon açılışında `Europe/Istanbul` doğrulanır, başarısızsa hata verilir. Saat dilimi ayarı yalnızca geçerli IANA adlarını kabul eder | FR-09, NFR-10 |
| R-10 | Çekişme sonucu `ABORTED` hatası (T-3) | Kullanıcı hata görür | İstemci mesajı ve güncel liste gösterilir. Tek salon ölçeğinde düşük olasılık | FR-11 |
| R-11 | Cloud Functions ve FCM'nin hizmet düzeyi taahhüdü doğrulanmadı | %99,5 hedefinin dayanağı eksik kalır | AS-10 yanıtına göre izleme yapılır | NFR-12 |
| R-12 | iOS push için salonun Apple hesabında APNs anahtarı gerekir (T-5). Hesap açılışında D-U-N-S süresi 30 güne kadar uzayabilir (BA K-21) | iOS bildirim testleri gecikir | Hesap işleri geliştirmenin başında başlatılır | NFR-15, AS-16 |
| R-13 | UIScene kullanan iOS uygulamasında bildirim temsilcisi (delegate) ayarlanmazsa bildirim davranışı bozulabilir (T-13) | iOS'ta bildirim sorunları | `configureNotificationCenterDelegate()` kurulum adımına eklenir | FR-18 |
| R-14 | Asenkron tamamlanan silmede kullanıcıya onay iletilemeyebilir | FR-03'teki onay şartı karşılanmaz | S-02 yanıtlanana kadar normal akış eşzamanlı tutulur | FR-03, NFR-07 |
| R-15 | Yedek alınmaması | Veri kaybında geri dönüş yok | S-06'da soruldu | NFR-14 |
| R-16 | Blaze planı ve dakikada bir çalışan iş, bütçe tanımlı değilken maliyet doğurur | Beklenmeyen fatura | Kullanım ücretsiz kotalar içinde öngörülüyor (7. bölüm). Bütçe ve uyarı AS-19'a bağlı | NFR-15, AS-19 |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Kayıt, giriş, şifre sıfırlama | `AuthModule`; Firebase Auth e-posta ve şifre (geçici, AS-02) | 3.1, 4.1 |
| FR-02 | Aydınlatma metni ve gizlilik politikası | `LegalTexts`; Hosting `/gizlilik`, `/aydinlatma` | 4.1, 4.3, 8 |
| FR-03 | Uygulama içi ve web'den hesap silme, onay | `AccountDeletion`, `requestAccountDeletion`, `deletionRetry`, Hosting `/hesap-sil`; K-5 | 4, 6.6 |
| FR-04 | Üç rol | `members/{uid}.role`, `RoleRouter`; K-6 | 4.1, 5, 6.7 |
| FR-05 | Tek salon profili | `salon/main`, `AdminSettings` | 4.1, 5 |
| FR-06 | Hizmet tanımı, not alanı yok | `services/{id}` | 5 |
| FR-07 | Çalışan ekleme, çıkarma ve davet | `inviteStaff`, `staff/{id}`, `members`; anında yetki kesimi (K-6) | 4.2, 6.7 |
| FR-08 | Çalışan saatleri, mola ve izin | `staff/{id}` (yönetici yazar, geçici AS-05) | 5 |
| FR-09 | Müsait saat hesabı, başlangıç aralığı | `SlotCalculator` + sunucu doğrulaması; K-3 | 6.3 |
| FR-10 | Randevu oluşturma, "Fark etmez" | `BookingFlow`, `createAppointment`, atama kuralı (geçici AS-04) | 4, 6.2, 6.4 |
| FR-11 | Çift rezervasyon olmaması | `staffDays` + sunucu işlemi; K-2 | 6.2 |
| FR-12 | Müşteri iptali ve iptal süresi | `cancelAppointment` süre kontrolü (geçici AS-03) | 4.2 |
| FR-13 | Salon iptali, zorunlu kategori, ≤200 karakter açıklama | `SalonCancelDialog`, `cancelAppointment`, sabit kategori sözlüğü, kural `size() <= 200` | 4, 5, 8 |
| FR-14 | Randevu listesi ve durumlar | `MyAppointments`; "tamamlandı" türetilir | 4.1, 5 |
| FR-15 | Çalışan ve yönetici takvimi | `StaffCalendar` | 4.1 |
| FR-16 | Bağlamında bildirim izni | `NotificationPermission`, `notifPromptedAt` | 4.1, 10 (R-05) |
| FR-17 | Bildirim olmadan çalışma, push'suz cihazlar | Tüm bilgi Firestore'dan uygulama içinde gösterilir; yedek kanal yok | 6.5, 9 |
| FR-18 | Müşteri bildirimleri (oluşturma, salon iptali, hatırlatma, anlık hatırlatma) | Outbox + `onOutboxCreated`, `reminderScanner`; K-4 | 6.5 |
| FR-19 | Çalışan ve yönetici bildirimleri (yeni, iptal, taşıma, silme kaynaklı iptal) | Outbox; silme akışı 3. adım | 6.5, 6.6 |
| FR-20 | Hesapsız müşteri için randevu girişi | `PhoneBooking`, `appointments.customer` (uid yok) | 4.1, 5, 8 |
| FR-21 | Müşteri taşıması, atomik | `rescheduleAppointment`, iki gün dokümanında tek işlem | 4.2, 6.2 |
| FR-22 | Randevu kuralları ve başlangıç aralığı | `salon/main.settings`, enum doğrulaması | 5, 6.3 |
| FR-23 | Gelmedi işareti ve sayısı | `markNoShow`, `customerStats` | 4.2, 5 |
| FR-24 | Ayarlanabilir hatırlatma zamanları | `reminderOffsetsMinutes`, planlama kuralı | 6.5 |
| FR-25 (C) | Onay modu | `approvalMode`, `pending_approval` durumu; slot ayrılır (S-08) | 5, 6.2 |
| FR-26 (C) | Bekleme listesi | İlk sürümde tasarlanmadı. Önceliklendirilirse ayrı koleksiyon + iptal tetikleyicisi önerilecek | 12 (S-09) |
| FR-27 (C) | SMS veya e-posta yedek hatırlatma | Sağlayıcı seçilmedi, AS-06'ya bağlı. Push'suz cihazlar kapsam dışı | 12 |
| FR-28 (C) | Cihaz takvimine ekleme | Paket seçilmedi. Takvim izni seçildiğinde kontrol edilecek (AS-06) | 3.2 |
| FR-29 | Silinen hesabın kayıtlarının anonim tutulması | Silme akışı 4. adım: `customer=null`, `customerDeleted=true` | 6.6, 8 |
| NFR-01 | Android ve iOS'ta aynı işlevler | Tek Flutter uygulaması; Android 7.0+ / iOS 15+ | 3.1, 6.1 |
| NFR-02 | 50 eşzamanlı denemede en fazla 1 kayıt | K-2 + gerçek projede eşzamanlılık testi | 6.2, 9 |
| NFR-03 | Liste ≤2 sn (%95), onay ≤3 sn | İstemci tarafı slot hesabı; aynı bölge; gerekirse en az 1 örnek | 6.3, 7 |
| NFR-04 | Olay ≤1 dk, planlı ±5 dk | Outbox tetikleyicisi, dakikada bir tarayıcı | 6.5, 7 |
| NFR-05 | ≤5 adım, 5 kişiden 4'ü 3 dk'da | 4 adımlı `BookingFlow` | 4.1, 7, 9 |
| NFR-06 | Yetkilendirme ve gizlilik | Güvenlik kuralları + callable kontrolleri; K-6; `staffDays` kişisel veri içermez | 5, 6.7, 8 |
| NFR-07 | 30 günde silme, onay | Eşzamanlı silme, `dueBy`, `deletionRetry` | 6.6 |
| NFR-08 | Veri en aza indirme, 200 karakter, mağaza beyanları | 3 kayıt alanı, not yok, üç katmanda sınır, beyan listesi | 5, 8 |
| NFR-09 | Erişilebilirlik | 48dp hedef, kontrast, yazı ölçeği, ekran okuyucu | 3.3, 9 |
| NFR-10 | Türkçe, 24 saat, GG.AA.YYYY, salon saat dilimi | `intl` + `timezone` (istemci), Luxon (sunucu) | 3.2, 5 |
| NFR-11 | Bildirim içeriği | Sunucuda sabit şablonlar, serbest metin yok | 6.5, 8 |
| NFR-12 | %99,5 erişilebilirlik | Firestore bölgesel %99,99 (T-1); diğer servisler doğrulanmadı (R-11) | 6.8, 7 |
| NFR-13 | Çevrimdışı davranış, kuyruk yok | Firestore önbelleği (T-12), yazmalar callable üzerinden | 2, 9 |
| NFR-14 | Anonimleştirmenin doğrulanması | Kişisel veri envanteri + tarama betiği; günlüklerde kişisel veri yok | 6.6, 8, 9 |
| NFR-15 | Salonun hesabıyla yayın, proje ve fatura salonda | Proje, fatura, mağaza hesapları ve APNs anahtarı salonda; yayın kontrol listesi | 2, 8, 9, 10 (R-12) |

## 12. BA'ya geri sorular

**Yeni sorular:**
- **S-01 (AS-08, engelleyici):** Firestore konumu sonradan değiştirilemiyor ve Türkiye bölgesi yok (T-1). KVKK md. 9 dayanağı belirlenmeden proje kurulamaz. Hukuki yanıt hangi tarihe kadar beklenebilir? Yanıt gelene kadar geliştirme yalnızca emülatörde ilerleyecek.
- **S-02 (FR-03, NFR-07):** Silme normal akışta anında tamamlanıyor ve onay ekranda gösteriliyor. Nadiren işlem asenkron tamamlanırsa kullanıcı artık uygulamaya giremez. Bu durumda onay hangi kanaldan iletilmeli? Bu tasarımda e-posta gönderim bileşeni yok, e-posta istenirse ek bir servis gerekir. Ekranda "en geç şu tarihte tamamlanacak" bilgisi yeterli mi?
- **S-03 (FR-22):** "Hizmetler arası tampon süre" her randevunun sonuna mı ekleniyor? Günün son randevusundan sonra, kapanışa kadar tampon aranmalı mı? Geçici yorum: Tampon her randevunun sonuna eklenir. Kapanıştan önce yalnızca hizmet süresinin sığması yeterlidir.
- **S-04 (FR-20, FR-23):** Hesapsız müşteri (telefonla girilen) "gelmedi" olarak işaretlendiğinde gelmeme sayısı tutulmalı mı? Tutulacaksa telefon numarasına göre mi eşleştirilmeli? Geçici karar: Yalnızca kayıtlı müşteriler için sayılır.
- **S-05 (AS-19, NFR-03):** Beklenen ölçek nedir: çalışan sayısı, günlük randevu sayısı, kayıtlı müşteri sayısı? Maliyet tahmini ve en az örnek sayısı kararı bu bilgilere bağlıdır.
- **S-06 (NFR-14):** Veri yedeği veya geri alma beklentisi var mı? Varsa NFR-14'teki "ürünün sakladığı verilerin tamamı" ifadesi yedekleri de kapsıyor mu? Yedekteki kişisel verinin ne kadar sürede silinmesi kabul edilir?
- **S-07 (FR-24):** Salon hangi hatırlatma zamanlarını seçebilir ve en fazla kaç hatırlatma tanımlanabilir? Geçici karar: En fazla 2 hatırlatma, 1 ile 48 saat arasında tam saat değerleri.
- **S-08 (FR-25, C):** Manuel onay modunda "onay bekliyor" durumundaki randevu saati başkalarına dolu mu görünmeli? Yönetici belirli bir süre içinde yanıt vermezse ne olur? Geçici karar: Saat ayrılır, süre sınırı yok.
- **S-09 (FR-26, C):** Bekleme listesinde saat boşalınca bildirim kime gider: ilk yazılana mı, tüm listeye mi? Saati ilk alan mı kazanır? FR-26 önceliklendirilirse tasarım bu yanıta göre yapılacak.

**BA dokümanında açık olan ve geçici kararla ilerlenen sorular** (paydaş yanıtı bekleniyor, yeni soru değil):
- AS-02: E-posta ve şifreyle kayıt.
- AS-03: İptal süresi dolduktan sonra müşteri iptali engellenir.
- AS-04: "Fark etmez"de o gün en az randevusu olan çalışan atanır, eşitlikte kimlik sırası kullanılır.
- AS-05: Çalışan saatlerini yalnızca yönetici düzenler.
- AS-07: Android 7.0 (API 24) ve iOS 15. Daha düşük sürümler Flutter (T-8) ve FlutterFire (T-7) tarafından desteklenmiyor.
- AS-15 (a): Anlık hatırlatma ayrı bildirim olarak gider (V-14).
- AS-15 (b): Ayar değişikliği mevcut hatırlatmaları güncellemez.
- AS-18: Hesapsız müşteri verisi randevu kaydında tutulur, saklama süresi tanımsız.
