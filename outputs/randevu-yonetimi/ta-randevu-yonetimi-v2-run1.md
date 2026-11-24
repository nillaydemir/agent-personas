# Güzellik Salonu ve Kuaför Randevu Yönetim Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-24
- **Persona sürümü:** ta v2
- **Girdi:** outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run1.md

---

## 1. Özet

Bu doküman, BA analiz dokümanındaki 28 fonksiyonel gereksinim (FR-01–FR-28) ve 13 fonksiyonel olmayan gereksinimi (NFR-01–NFR-13) karşılayan teknik çözümü tanımlar. Flutter ve Firebase görev metninde verilmiş kısıtlardır (BA V-02). Bu doküman yalnızca hangi Firebase servislerinin nasıl kullanılacağına karar verir.

Ana kararlar:
- **Tek Flutter kod tabanı** (Android + iOS), arkasında yalnızca yönetilen Firebase servisleri kullanılır: Authentication, Cloud Firestore, Cloud Functions (2. nesil, TypeScript), Cloud Messaging (FCM), App Check ve Hosting. Ayrı bir sunucu yoktur (NFR-01, kapsam kuralı).
- **Randevu yazma işlemleri yalnızca sunucuda yapılır.** Oluşturma, iptal, taşıma ve no-show işaretleme, Cloud Functions'taki çağrılabilir (callable) fonksiyonlarla yapılır. İstemcinin randevu koleksiyonuna doğrudan yazması güvenlik kurallarıyla engellenir (FR-11, NFR-02, NFR-06).
- **Çift rezervasyon önleme:** Her çalışan ve gün için bir "doluluk" (occupancy) dokümanı tutulur. Randevu bu doküman üzerinden, sunucu tarafı Firestore işlemi (transaction) içinde kaydedilir (FR-11, NFR-02; Bölüm 6.2).
- **Müsait saatler sunucuda hesaplanır.** Hesaplama 15 dakikalık sabit ızgara üzerinde, hizmet süresi ve tampon dikkate alınarak yapılır. Aynı kod kayıt anındaki doğrulamada da kullanılır (FR-09, FR-22; Bölüm 6.1).
- **Hatırlatmalar:** Dakikada bir çalışan zamanlanmış bir fonksiyon, zamanı gelmiş hatırlatma kayıtlarını tarar ve FCM ile gönderir (FR-18, FR-24, NFR-04; Bölüm 6.3).
- **Rol yetkisi**, işletme altındaki üyelik dokümanlarıyla yönetilir. Özel talepler (custom claims) kullanılmaz (FR-04, NFR-06; Bölüm 6.4).

BA'nın açık sorularına bağlı kararlar "geçici" olarak işaretlenmiştir. Bu kararlar Bölüm 12'de BA'ya geri sorulmuştur. En kritik bekleyen karar, Firestore veri konumudur (AS-08). Konum sonradan değiştirilemez.

## 2. Mimari genel bakış

```
┌──────────────────────────── Flutter uygulaması (Android/iOS) ────────────────────────────┐
│ Sunum: Müşteri akışı | Çalışan takvimi | Yönetici paneli | Ayarlar (hesap silme, KVKK)    │
│ Uygulama katmanı: AuthService, BookingRepository, ScheduleRepository, NotificationService │
│ Veri erişimi: firebase_auth, cloud_firestore (okuma + önbellek), cloud_functions (yazma), │
│               firebase_messaging, firebase_app_check, intl                                │
└───────────────┬───────────────────────────┬───────────────────────────┬──────────────────┘
                │ okuma (kurallarla sınırlı) │ callable (Auth + App Check)│ FCM push
                ▼                           ▼                           ▲
        ┌───────────────┐        ┌───────────────────────────┐   ┌──────────────┐
        │ Cloud         │◄──────►│ Cloud Functions (2. nesil) │──►│ FCM          │
        │ Firestore     │ trans. │ - callable: getSlots,      │   └──────────────┘
        │ (tek bölge)   │        │   createAppointment, ...   │
        │               │──trig.►│ - Firestore tetikleyicileri│
        └───────────────┘        │ - onSchedule: hatırlatma   │
                ▲                └───────────────────────────┘
                │                ┌───────────────────────────┐
                └── Auth ────────│ Firebase Authentication   │
                                 └───────────────────────────┘
        Firebase Hosting: statik gizlilik politikası, aydınlatma metni, hesap silme talebi sayfası
```

**Akış ilkeleri:**
1. **Okuma:** İstemci kendi profilini, randevularını ve (rolüne göre) işletme verisini Firestore'dan doğrudan okur. Çevrimdışı önbellek bu okumalarda kullanılır (NFR-13).
2. **Yazma:** Randevu durumunu değiştiren her işlem callable fonksiyonla yapılır. İşletme ayarları ve çalışan takvimi tanımları ise yönetici tarafından doğrudan Firestore'a yazılır ve kurallarla doğrulanır (FR-05–FR-08).
3. **Yan etkiler:** Bildirimler, hatırlatma kayıtları ve no-show sayaçları sunucuda üretilir. İstemciye güvenilmez (NFR-06, FR-18, FR-19).
4. **Çok işletmeli yapı (geçici, BA V-01 / AS-01):** Tüm işletme verisi `businesses/{bizId}` altında tutulur. Müşteri işletmeye bağlantı veya kodla katılır. Tek salonlu sürüm seçilirse aynı model tek bir `bizId` ile çalışır. Model değişmez.

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Platform ve servisler

| Katman | Seçim | Gerekçe | Gereksinim |
|---|---|---|---|
| İstemci | Flutter, tek kod tabanı | Görev kısıtı. Android ve iOS'ta aynı işlev | NFR-01 |
| Kimlik | Firebase Authentication, e-posta + şifre (**geçici**, AS-02) | Yalnızca ad, e-posta ve şifre istenir (3 alan). Telefon doğrulaması Blaze planında SMS başına ücretlidir [T-6]. Şifre sıfırlama `sendPasswordResetEmail()` ile, e-posta doğrulama `sendEmailVerification()` ile yapılır [T-13] | FR-01, NFR-08 |
| Veritabanı | Cloud Firestore (Standard) | Bileşik filtre ve sıralama sorguları gerekir (çalışan + tarih aralığı). Firebase yeni projeler için Firestore'u öneriyor. Realtime Database'de bir sorgu aynı anda hem sıralama hem filtreleme yapamaz [T-2] | FR-09, FR-14, FR-15 |
| Sunucu mantığı | Cloud Functions 2. nesil, TypeScript (Node.js) | Callable fonksiyonlarda Auth token'ı otomatik eklenir ve doğrulanır [T-9]. Dart ile yazılan fonksiyonlar deneysel durumdadır ve istemci SDK'larından adla çağrılamaz [T-9]. Bu nedenle TypeScript seçildi | FR-09–FR-13, FR-21, NFR-02 |
| Push | Firebase Cloud Messaging | Ücretsiz [T-6]. Flutter eklentisi resmî | FR-16–FR-19 |
| Zamanlanmış iş | Cloud Functions `onSchedule` (Cloud Scheduler) | Bölüm 6.3'teki karşılaştırma | FR-18, FR-24, NFR-04 |
| Kötüye kullanım koruması | Firebase App Check | Callable uç noktalarına yalnızca uygulamanın erişmesine yardım eder [T-9] | NFR-06 |
| Statik web | Firebase Hosting | Statik içerik, yerleşik SSL ve ücretsiz `web.app` alt alan adı [T-17]. Hesap silme talebi sayfası ve gizlilik politikası için kullanılır | FR-02, FR-03 |
| Tarih ve saat | İstemci: `intl`. Sunucu: Luxon | `intl` saat dilimlerini desteklemez [T-15]. Bu nedenle işletme saat dilimindeki etiketler sunucuda üretilir. Luxon IANA bölgelerini ortamın Intl API'si üzerinden destekler [T-16] | NFR-10, FR-09 |

**Faturalama planı:** Cloud Functions, Spark planında "Not applicable" olarak listelenir. Bu nedenle proje **Blaze** planında açılmalıdır (fiyat sayfasındaki tablodan çıkarıldı) [T-6]. Zamanlanmış fonksiyonlarda Google hesabı başına 3 iş ücretsizdir, sonraki her iş aylık 0,10 $'dır [T-5]. Bu tasarım tek bir zamanlanmış iş kullanır.

**Veri konumu (geçici, AS-08):** Firestore konum listesinde Türkiye bölgesi yoktur. Avrupa'da örneğin `europe-west3` (Frankfurt), `europe-west1` (Belçika) ve çok bölgeli `eur3` bulunur [T-4]. Konum, veritabanı oluşturulduktan sonra değiştirilemez: "once you provision a database instance, you cannot change its location setting" [T-4]. Fonksiyonlar Firestore ile aynı bölgeye kurulur. Firebase, Firestore ve fonksiyon arasındaki mesafenin ciddi gecikme yaratabileceğini belirtiyor [T-11]. Geçici öneri tek bölgeli `europe-west3`'tür. Hukuki dayanak (KVKK md. 9) netleşmeden üretim veritabanı **oluşturulmaz**.

### 3.2 Flutter paketleri: izin, sürüm ve kısıt kontrolü

FlutterFire'ın en düşük hedef sürümleri, kurulum belgesine göre **iOS 15** ve **Android API 24 (Android 7.0)**'tür [T-7]. Uygulamanın en düşük sürümleri bunlardır (geçici, AS-07).

| Paket (pub.dev sürümü) | (a) İzinler | (b) En düşük platform | (c) Belgedeki bilinen kısıtlar | Gereksinim |
|---|---|---|---|---|
| `firebase_auth` 7.1.0 [T-14] | Paket sayfasında izin belirtilmemiş | FlutterFire: iOS 15, API 24 [T-7] | Paket sayfasında kısıt listelenmemiş. Hesap silme yakın zamanda giriş gerektirir. Aksi halde `requires-recent-login` hatası döner [T-13] | FR-01, FR-03 |
| `cloud_firestore` 7.2.0 [T-14] | Belirtilmemiş | FlutterFire: iOS 15, API 24 | Paket sayfasında kısıt yok. Belgeye göre istemci işlemleri çevrimdışıyken başarısız olur. Toplu yazmalar ise çevrimdışıyken kuyruğa alınır [T-1] | FR-14, FR-15, NFR-13 |
| `cloud_functions` 7.1.0 [T-14] | Belirtilmemiş | FlutterFire: iOS 15, API 24 | `us-central1` dışındaki bölgeler istemcide açıkça belirtilmelidir [T-9] | FR-09–FR-13, FR-21 |
| `firebase_messaging` 17.1.0 [T-3] | Paket sayfasında izin listelenmemiş. FCM belgesine göre iOS'ta ve Android 13+'ta mesaj alınmadan önce kullanıcı izni gerekir [T-8]. Android manifest izni (`POST_NOTIFICATIONS`) BA K-12'de anılıyor | FlutterFire: iOS 15, API 24. iOS teslimi için fiziksel cihaz, APNs kimlik doğrulama anahtarı ve Xcode'da Push Notifications gerekir [T-7] | UIScene yaşam döngüsünde `configureNotificationCenterDelegate()` çağrısı gerekir [T-3]. Android 13+'ta izin kararının verilip verilmediği belirlenemez, uygulama sorduğunu kendisi kaydetmelidir [T-8]. Kullanıcı uygulamayı zorla kapatırsa, uygulama yeniden açılana kadar mesajlar gelmez [T-8] | FR-16–FR-19, NFR-04 |
| `firebase_app_check` 0.5.2 [T-14] | Belirtilmemiş | App Attest iOS 14.0+ (iOS 15 tabanı bunu karşılar). Android'de Play Integrity [T-10] | Kısa token ömrü kotayı hızlı tüketir. Belgede sayısal kota verilmiyor [T-10] | NFR-06 |
| `intl` 0.20.3 [T-15] | Yok | Android, iOS dahil tüm platformlar | "Time zones are not yet supported." Yerel ayar verisi `initializeDateFormatting` ile yüklenmelidir [T-15] | NFR-10 |
| `firebase_core` ^5.1.0 | Yukarıdaki tüm paketlerin ortak bağımlılığı [T-3][T-14]. Ayrı sayfası açılmadı | FlutterFire: iOS 15, API 24 | Kurulum `flutterfire configure` ile yapılır. Bu komut Crashlytics ve Performance için Gradle eklentisi ekleyebilir [T-7] (bkz. Bölüm 8, NFR-08) | NFR-01 |

**Sunucu bağımlılıkları:** `firebase-functions`, `firebase-admin` ve Luxon [T-16]. Luxon'da yaz saati geçişindeki çift anlamlı yerel saatlerin davranışı tanımsızdır [T-16]. Bu durum Riskler bölümünde R-06 olarak ele alınmıştır.

**Bilinçli olarak kullanılmayan paket:** `flutter_local_notifications` hatırlatma için değerlendirildi ve seçilmedi (Bölüm 6.3).

### 3.3 Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| T-1 | Firestore: Transactions and batched writes | https://firebase.google.com/docs/firestore/manage-data/transactions | "Transactions will fail when the client is offline."; "Batched writes execute even when the user's device is offline."; eşzamanlı düzenlemede işlem tümüyle yeniden çalıştırılır, "retried a finite number of times"; okumalar yazmalardan önce yapılmalı; 20 saniyelik kilit süresi |
| T-2 | Firestore vs Realtime Database | https://firebase.google.com/docs/database/rtdb-vs-firestore | Firestore: "Indexed queries with compound sorting and filtering". RTDB'de bir sorgu aynı anda hem sıralama hem filtreleme yapamaz. Firestore tek dokümana yazma hızını sınırlar. Yeni müşterilere Firestore öneriliyor |
| T-3 | pub.dev: firebase_messaging | https://pub.dev/packages/firebase_messaging | Sürüm 17.1.0; platformlar; UIScene için `configureNotificationCenterDelegate()` gerekliliği |
| T-4 | Firestore locations | https://firebase.google.com/docs/firestore/locations | Avrupa ve Orta Doğu bölgeleri; Türkiye bölgesi yok; "once you provision a database instance, you cannot change its location setting" |
| T-5 | Schedule functions | https://firebase.google.com/docs/functions/schedule-functions | `onSchedule`, Cloud Scheduler; "an allowance of three jobs per Google account, at no charge", sonrası iş başına aylık 0,10 $; en kısa çalışma aralığı sayfada belirtilmiyor |
| T-6 | Firebase pricing | https://firebase.google.com/pricing | Spark'ta Cloud Functions "Not applicable"; Spark Firestore kotaları (günde 50K okuma, 20K yazma); Phone Auth Blaze'de "Billed per SMS sent"; FCM ücretsiz |
| T-7 | Add Firebase to your Flutter app | https://firebase.google.com/docs/flutter/setup | iOS 15; "Target API level 24 (Nougat) or higher"; Apple'da Cloud Messaging için fiziksel cihaz ve APNs anahtarı gerekli; `flutterfire configure` |
| T-8 | FCM Flutter: Receive messages | https://firebase.google.com/docs/cloud-messaging/flutter/receive | "before FCM payloads can be received on your device, you must first ask the user's permission" (iOS, Android 13+); Android'de "there is no way to determine if the user has chosen whether to grant/deny permission"; zorla kapatılan uygulama elle yeniden açılmalı |
| T-9 | Call functions from your app (callable) | https://firebase.google.com/docs/functions/callable?gen=2nd | Auth token'ları "automatically included in requests"; App Check önerisi; `us-central1` dışı bölge istemcide belirtilmeli; Dart fonksiyonları deneysel; `HttpsError` |
| T-10 | App Check Flutter default providers | https://firebase.google.com/docs/app-check/flutter/default-providers | Android'de Play Integrity, Apple'da DeviceCheck veya App Attest; "App Attest provider is only available on iOS 14.0+"; kısa TTL kotayı hızlı tüketir |
| T-11 | Cloud Firestore triggers | https://firebase.google.com/docs/functions/firestore-events | "Events are delivered at least once, but a single event may result in multiple function invocations."; "Ordering is not guaranteed."; fonksiyonu Firestore ile aynı bölgeye kurma önerisi |
| T-12 | Transaction serializability and isolation (data contention) | https://firebase.google.com/docs/firestore/transaction-data-contention | Mobil ve web SDK'ları "emulate optimistic concurrency transactions using write preconditions on document versions". Sunucu kütüphanelerinde karamsar (pessimistic) modda "transactions place locks on the documents they read". Standard sürümde varsayılan mod karamsardır |
| T-13 | Firebase Auth Flutter: Manage users | https://firebase.google.com/docs/auth/flutter/manage-users | `delete()`; `requires-recent-login`; `reauthenticateWithCredential`; `sendPasswordResetEmail()`; `sendEmailVerification()` |
| T-14 | pub.dev: firebase_auth, cloud_firestore, cloud_functions, firebase_app_check | https://pub.dev/packages/firebase_auth ; https://pub.dev/packages/cloud_firestore ; https://pub.dev/packages/cloud_functions ; https://pub.dev/packages/firebase_app_check | Sürümler 7.1.0 / 7.2.0 / 7.1.0 / 0.5.2; Android ve iOS desteği; `firebase_core` ^5.1.0 bağımlılığı; izin veya kısıt listelenmemiş |
| T-15 | pub.dev: intl | https://pub.dev/packages/intl | Sürüm 0.20.3; yerel ayarlı `DateFormat`; `initializeDateFormatting`; "Time zones are not yet supported." |
| T-16 | Luxon: Time zones and offsets | https://github.com/moment/luxon/blob/master/docs/zones.md | IANA bölgeleri yerleşik Intl API'leri üzerinden destekleniyor; yaz saati boşluğundaki saat ileri kaydırılıyor; çift anlamlı saatlerde "Luxon's behavior here is undefined" |
| T-17 | Firebase Hosting | https://firebase.google.com/docs/hosting | "optimized for static and single-page web apps"; "Zero-configuration SSL"; ücretsiz `web.app` alt alan adı |
| T-18 | Firestore security rules conditions | https://firebase.google.com/docs/firestore/security/rules-conditions | `get()`/`exists()` ile rol kontrolü; tek doküman isteğinde 10 erişim çağrısı sınırı, işlem ve toplu yazmada toplam 20; erişim çağrıları okuma olarak faturalanır; "security rules are not filters" |
| T-19 | Control access with custom claims | https://firebase.google.com/docs/auth/admin/custom-claims | "The payload must not exceed 1000 bytes."; yeni talepler "will propagate to the user's ID token the next time a new one is issued" |
| T-20 | Cloud Tasks quotas and limits | https://docs.cloud.google.com/tasks/docs/quotas | En ileri zamanlama: "30 days from current date and time" |
| T-21 | Enqueue functions with Cloud Tasks | https://firebase.google.com/docs/functions/task-functions | `scheduleTime` / `scheduleDelaySeconds`; kuyruk düzeyinde yeniden deneme; `roles/cloudtasks.enqueuer` ve `roles/cloudfunctions.invoker` IAM gereksinimleri |
| T-22 | pub.dev: flutter_local_notifications | https://pub.dev/packages/flutter_local_notifications | iOS'ta "it will only keep the 64 notifications that were last set"; bazı OEM'lerde "scheduled notifications may not work when the application is in the background"; `SCHEDULE_EXACT_ALARM`, `USE_EXACT_ALARM` ve `RECEIVE_BOOT_COMPLETED` izinleri |
| T-23 | Firestore offline data | https://firebase.google.com/docs/firestore/manage-data/enable-offline | "For Android and Apple platforms, offline persistence is enabled by default."; `fromCache` true ise veri önbellekten gelmiştir |
| T-24 | Firestore pricing | https://firebase.google.com/docs/firestore/pricing | "There is a minimum charge of one document read for each query that you perform, even if the query returns no results." |
| T-25 | Best practices for FCM registration management | https://firebase.google.com/docs/cloud-messaging/manage-tokens | Kayıtları zaman damgasıyla saklama; varsayılan bayatlık süresi bir ay; `UNREGISTERED` (404) ve `INVALID_ARGUMENT` (400) yanıtlarında kaydı silme |
| T-26 | Google Play: account deletion requirements | https://support.google.com/googleplay/android-developer/answer/13327111 | Web kaynağı şu yollardan biri olabilir: "an additional link that initiates account deletion, a customer service email or a form". Sayfada uygulama veya geliştirici adı yer almalı. Saklanan veriler bildirilmeli |
| T-27 | Flutter accessibility / accessibility testing | https://docs.flutter.dev/ui/accessibility ; https://docs.flutter.dev/ui/accessibility/accessibility-testing | "All tappable targets should be at least 48x48 pixels"; 4.5:1 kontrast önerisi; `androidTapTargetGuideline` (48×48), `iOSTapTargetGuideline` (44×44), `labeledTapTargetGuideline`, `textContrastGuideline`; TalkBack ve VoiceOver testi |
| T-28 | Test security rules (rules-unit-testing) | https://firebase.google.com/docs/rules/unit-tests | Emülatörde `initializeTestEnvironment`, `authenticatedContext`, `assertSucceeds` ve `assertFails` |

## 4. Bileşenler ve sorumlulukları

### 4.1 İstemci (Flutter)

| Bileşen | Sorumluluk | Gereksinim |
|---|---|---|
| `AuthService` | Kayıt (ad, e-posta, şifre), giriş, şifre sıfırlama, e-posta doğrulama ve silme öncesi yeniden kimlik doğrulama | FR-01, FR-03, NFR-08 |
| `ConsentScreen` / `LegalLinks` | Kayıtta aydınlatma metni ve gizlilik politikası bağlantısı gösterilir. Ayarlar ekranından bu metinlere her zaman erişilir. Metinler Hosting'de barındırılır | FR-02 |
| `AccountDeletionFlow` | En fazla 3 adım: Ayarlar → "Hesabı sil" → onay ve gerekirse yeniden giriş. Ardından `deleteMyAccount` çağrılır | FR-03, NFR-07 |
| `BusinessJoin` | Müşterinin işletmeye bağlantı veya kodla katılması (geçici, AS-01) | FR-10 |
| `BookingFlow` | 1) İşletme ana sayfası → 2) Hizmet → 3) Çalışan veya "Fark etmez" → 4) Tarih + saat (`getAvailableSlots`) → 5) Onay (`createAppointment`) | FR-09, FR-10, NFR-05 |
| `MyAppointments` | Yaklaşan ve geçmiş randevuların durumlarıyla listelenmesi, iptal ve taşıma. Önbellekten okuma ve çevrimdışı uyarısı | FR-12, FR-14, FR-21, NFR-13 |
| `StaffCalendar` | Çalışan için günlük ve haftalık görünüm. Yönetici için çalışan sütunlu gün görünümü. Kaynak: `appointments` sorgusu (çalışan + tarih aralığı) | FR-15 |
| `AdminSettings` | İşletme profili, çalışma saatleri, kapalı günler, hizmetler, çalışanlar ve davet, randevu kuralları, hatırlatma zamanları, onay modu | FR-05–FR-08, FR-22, FR-24, FR-25 |
| `ManualBooking` | Yönetici veya çalışanın hesapsız müşteri adına (ad + telefon) randevu girmesi | FR-20 |
| `NotificationService` | İzin isteğini ilk başarılı randevudan sonra gerekçe diyaloğuyla yapar. "İzin istendi" bayrağı yerelde saklanır [T-8]. FCM kaydı `devices` alt koleksiyonuna yazılır. İzin yoksa "Bildirimleri aç" yolu gösterilir | FR-16, FR-17 |
| `ConnectivityBanner` | Önbellekten gelen veride (`fromCache`) [T-23] ve callable `unavailable` hatasında "İnternet yok, randevu işlemi yapılamaz" mesajı gösterilir | NFR-13 |

### 4.2 Sunucu (Cloud Functions, TypeScript, Firestore ile aynı bölge)

| Fonksiyon | Tür | Sorumluluk | Gereksinim |
|---|---|---|---|
| `getAvailableSlots` | callable | Bölüm 6.1 algoritması. İşletme saat diliminde `HH:mm` etiketleri ve UTC anları döndürür | FR-09, FR-10, FR-22, NFR-03, NFR-10 |
| `createAppointment` | callable | Kuralları yeniden doğrular, "fark etmez" ise çalışan atar, doluluk dokümanını işlem içinde günceller, randevu ve hatırlatma kayıtlarını yazar. Müşteri, yönetici veya çalışan tarafından çağrılabilir (FR-20) | FR-10, FR-11, FR-20, FR-22, FR-24, FR-25, NFR-02 |
| `cancelAppointment` | callable | Rol ve iptal süresi kontrolü, doluluktan aralığın düşülmesi, hatırlatmaların `cancelled` yapılması | FR-12, FR-13 |
| `rescheduleAppointment` | callable | Tek işlemde yeni aralığı alır, eski aralığı bırakır ve hatırlatmaları yeniden üretir | FR-21 |
| `markNoShow` / `markCompleted` | callable | Durum değişikliği ve müşteri no-show sayacının artırılması | FR-14, FR-23 |
| `approveAppointment` / `rejectAppointment` | callable | Manuel onay modu (C) | FR-25 |
| `inviteEmployee` / `acceptInvite` / `removeEmployee` | callable | Tek kullanımlık davet kodu, üyelik dokümanı oluşturma ve silme | FR-04, FR-07 |
| `deleteMyAccount` | callable | Bölüm 8.3'teki silme akışı | FR-03, NFR-07 |
| `onAppointmentWrite` | Firestore tetikleyicisi | Durum geçişinden olay türünü çıkarır. Müşteriye, çalışana ve yöneticiye FCM gönderir. `notificationLog/{eventId}` kaydıyla aynı olay için tekrar göndermez (en az bir kez teslim [T-11]). Bekleme listesini (C) tetikler | FR-18, FR-19, FR-26, NFR-04, NFR-11 |
| `sendDueReminders` | `onSchedule("* * * * *")` | Bölüm 6.3 | FR-18, FR-24, NFR-04 |
| `pruneStaleDevices` | Aynı zamanlanmış iş içinde günde bir kez çalışan dal | Bir aydan eski FCM kayıtlarını siler [T-25]. Ayrı bir zamanlanmış iş açılmaz | FR-18, FR-19 |

### 4.3 Diğer

- **Firestore güvenlik kuralları:** Bölüm 6.4 ve Bölüm 8.
- **Hosting:** `/gizlilik`, `/aydinlatma`, `/hesap-silme` statik sayfaları. Hesap silme sayfasında uygulama adı, silinen ve saklanan veriler ve talep e-posta adresi yer alır. Play, e-posta yoluyla talep almayı kabul ediyor [T-26] (FR-02, FR-03).
- **Operatör betiği:** `scripts/` altındaki Admin SDK betikleri şu işler için kullanılır: işletme ve ilk yönetici oluşturma (geçici, AS-01), web üzerinden gelen silme talebini `deleteMyAccount` ile aynı kodla işleme, KVKK bilgi talebi için kullanıcı verisini dışa aktarma (NFR-07).

## 5. Veri modeli

Tüm zaman anları UTC `Timestamp` olarak saklanır. Haftalık saatler yerel `"HH:mm"` olarak ve işletmenin IANA saat dilimiyle (`timeZone`, varsayılan `Europe/Istanbul`) birlikte tutulur (NFR-10).

| Yol | Alanlar (özet) | Okuma / yazma | Gereksinim |
|---|---|---|---|
| `users/{uid}` | `name`, `email`, `createdAt`, `joinedBusinessIds[]` | Okuma ve yazma yalnızca sahibi. Silme yalnızca sunucuda | FR-01, NFR-06, NFR-08 |
| `users/{uid}/devices/{installId}` | `token`, `platform`, `updatedAt` | Yazma sahibi. Okuma yalnızca sunucu | FR-16–FR-19 |
| `businesses/{bizId}` | `name`, `address`, `phone`, `timeZone`, `weeklyHours{mon..sun:[{start,end}]}`, `closedDates[]`, `rules{minNoticeMin, maxAdvanceDays, bufferMin, cancelWindowMin(=120), reminderOffsetsMin[](=[1440]), approvalMode}`, `joinCode` | Okuma işletmeye katılmış kullanıcı. Yazma `admin` | FR-05, FR-12, FR-22, FR-24, FR-25 |
| `businesses/{bizId}/members/{uid}` | `role: admin\|employee\|customer`, `employeeId?` | Okuma sahibi ve `admin`. Yazma yalnızca sunucu | FR-04, FR-07 |
| `businesses/{bizId}/employees/{empId}` | `displayName`, `uid?`, `active`, `weeklyHours`, `breaks[{day,start,end}]`, `leaves[{from,to}]`, `serviceIds[]` | Okuma işletme üyeleri (yalnızca görünen ad ve hizmet bilgisi). Yazma `admin`; çalışan kendi kaydı için yazabilir mi, AS-05'e bağlı | FR-06, FR-07, FR-08 |
| `businesses/{bizId}/services/{svcId}` | `name`, `durationMin`, `price` (bilgi amaçlı), `employeeIds[]`, `active` | Okuma üyeler. Yazma `admin` | FR-06 |
| `businesses/{bizId}/appointments/{apptId}` | `customerUid\|null`, `customerName`, `customerPhone?` (yalnızca FR-20), `employeeId`, `serviceId`, `serviceName`, `durationMin`, `start`, `end`, `status: pending\|active\|cancelled_by_customer\|cancelled_by_business\|completed\|no_show\|rescheduled`, `cancelReason?`, `createdBy`, `rescheduledTo?`, `createdAt`, `updatedAt` | Okuma: müşteri `customerUid == uid` olan kayıtlar (collection group sorgusu), çalışan `employeeId` kendisi olan kayıtlar, `admin` tümü. Yazma yalnızca sunucu | FR-10–FR-15, FR-20, FR-21, FR-23, NFR-06 |
| `businesses/{bizId}/occupancy/{empId}_{yyyyMMdd}` | `intervals[{apptId, startMin, endMin}]` (işletme yerel günündeki dakika; kişisel veri içermez) | İstemci erişimi yok. Yalnızca sunucu | FR-11, NFR-02 |
| `businesses/{bizId}/customers/{customerKey}` | `noShowCount`, `displayName`. `customerKey`: `uid` veya hesapsız müşteri için normalize edilmiş telefon | Okuma `admin`. Yazma yalnızca sunucu | FR-23 |
| `reminders/{apptId}_{offsetMin}` | `bizId`, `apptId`, `dueAt`, `status: pending\|sent\|cancelled`, `sentAt?` | Yalnızca sunucu | FR-18, FR-24 |
| `notificationLog/{eventId}` | `createdAt` | Yalnızca sunucu | NFR-04 (tekrarlı tetiklemeye karşı) |
| `businesses/{bizId}/waitlist/{id}` (C) | `customerUid`, `employeeId?`, `date`, `serviceId`, `createdAt` | Yazma sahibi (kendi kaydı). Okuma sahibi ve `admin` | FR-26 |

**Dizinler:** Şu sorgular için bileşik dizin gerekir:
- `appointments`: (`employeeId`, `start`) ve (`status`, `start`).
- `appointments` collection group: (`customerUid`, `start`).
- `reminders`: (`status`, `dueAt`).

**Hizmet notu alanı yoktur.** BA V-08, sağlık verisi girilmemesini öngörüyor. Hiçbir FR serbest metin not istemediği için model bu alanı içermez (NFR-11, NFR-08). Bu konu BA'ya ayrıca sorulmuştur (S-09).

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Müsait saat üretimi (FR-09, FR-10, FR-22)

**Seçim ölçütleri:**
- Ö1: Süre, mola ve kapanışa taşan saatlerin gösterilmemesi (FR-09, US-05 KK-1).
- Ö2: Müşterinin anlayabileceği saatler ve en fazla 5 adım (NFR-05).
- Ö3: Tampon, en kısa önceden bildirim ve en ileri tarih kurallarının uygulanması (FR-22).
- Ö4: Doğrulanabilirlik ve test yükü (risk: yanlış slot gösterme).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| A. Sabit 15 dakikalık ızgara + uygunluk filtresi | Karşılar | Saatler düzenli (10:00, 10:15…) | Karşılar | Basit ve deterministik |
| B. Dinamik başlangıçlar (ızgara + her doluluğun bitiş anı) | Karşılar | 10:40 gibi düzensiz saatler çıkabilir | Karşılar | Test durumu sayısı artar |

**Karar: A.** BA doküman 3.5, B'nin doluluğu artırdığını ama kurgu ve test yükünü artırdığını belirtiyor. A, Ö2 ve Ö4'te üstündür. Ö1 ve Ö3'te ikisi eşittir. Izgara adımı (`SLOT_STEP_MIN = 15`) bir sabittir. Ayarlanabilir olması gerekip gerekmediği BA'ya sorulmuştur (S-05).

**Algoritma** (işletme yerel gününde dakika cinsinden; G = gün, E = çalışan, S = hizmet):
1. G kapalıysa, E izinliyse veya G > bugün + `maxAdvanceDays` ise boş liste döndürülür.
2. `açık = işletmeSaatleri(G) ∩ çalışanSaatleri(E, G)`.
3. `açık = açık − molalar(E, G)`.
4. `dolu` = `occupancy/{E}_{G}.intervals` içindeki her aralığın `[start − bufferMin, end + bufferMin]` biçimine genişletilmiş hali. `açık = açık − dolu`.
5. `açık` içindeki her serbest aralık `[a, b]` için: `t = ceil(a / 15) × 15`. `t + S.durationMin ≤ b` olduğu sürece, `t ≥ şimdi + minNoticeMin` ise `t` slot olarak eklenir ve `t += 15` yapılır.
6. "Fark etmez" seçeneğinde 1–5. adımlar `S.employeeIds` içindeki her aktif çalışan için çalıştırılır ve sonuçların birleşimi döndürülür.
7. Çıktı her slot için `{startUtc, label: "HH:mm", employeeIds[]}` biçimindedir. Yerel saat ↔ UTC dönüşümü Luxon ile işletmenin `timeZone` değeri kullanılarak yapılır [T-16].

**Neden sunucuda?** İstemci başka müşterilerin randevularını okuyamaz (NFR-06). `intl` saat dilimini desteklemez [T-15]. Aynı fonksiyon `createAppointment` içinde doğrulama için tekrar çalıştırılır. Böylece kuralların tek uygulaması vardır (Ö4).

### 6.2 Çift rezervasyon önleme (FR-11, NFR-02)

**Seçim ölçütleri:**
- Ö1: Eşzamanlı 50 denemede en fazla 1 başarı (NFR-02).
- Ö2: Farklı süreli, birbiriyle kısmen çakışan randevuların yakalanması (FR-09, FR-11).
- Ö3: İstemci manipülasyonuna dayanıklılık (NFR-06).
- Ö4: İptal ve taşımada atomik bırakma (FR-12, FR-21).
- Ö5: Gecikme (NFR-03, 3 saniye).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| A. Çalışan-gün **doluluk dokümanı** + sunucu (Admin SDK) işlemi | Karşılar. Aynı dokümanı okuyan işlemler sıralanır. Standard sürüm sunucu işlemlerinde varsayılan karamsar kilit kullanılır [T-12] | Aralık çakışması kodla kontrol edilir | Callable; kurallar doğrudan yazmayı engeller | Tek işlemde aralık eklenir veya çıkarılır | 1 doküman okuma + 3–4 yazma |
| B. Zaman dilimi başına kilit dokümanları (ör. 15 dakikalık her dilim için `create()`) | Karşılar (`create` mevcut dokümanda başarısız olur) | Karşılar, ama 45 dakikalık randevu 3 kilit dokümanı ister ve tampon değişince dilim sayısı değişir | Kurallarla da yapılabilir | İptalde birden çok doküman silinir | Daha çok yazma |
| C. İstemci SDK işlemi + güvenlik kuralları | Mobil SDK'lar sürüm ön koşuluyla iyimser çalışır [T-12]. Aralık çakışmasını kurallarla doğrulamak pratik değil | Kurallarla zor | Zayıf: iş kuralları istemcide | — | Callable soğuk başlangıcı yok |

**Karar: A.** Ö2, Ö3 ve Ö4'ü en sade biçimde karşılar. Ayrıca doluluk dokümanı kişisel veri içermediği için müsaitlik hesabında da okunur (6.1, adım 4).

**`createAppointment` işlem adımları:**
1. Oku: `occupancy/{E}_{G}`, `businesses/{bizId}`, `employees/{E}`. Okumalar yazmalardan önce yapılır [T-1].
2. Algoritma 6.1'i yalnızca istenen `t` için çalıştır. Uygun değilse `HttpsError('failed-precondition', 'SLOT_TAKEN')` döndür.
3. Yaz: `intervals` dizisine yeni aralığı ekle. `appointments/{new}` oluştur (onay modu `approvalMode = manual` ise `status: pending`, aksi halde `active`). Her `reminderOffsetsMin` için `reminders/{apptId}_{offset}` oluştur.
4. İstemci `SLOT_TAKEN`, `aborted` veya `deadline-exceeded` aldığında "Bu saat az önce doldu" mesajını gösterir ve `getAvailableSlots`'u yeniden çağırır (US-05 KK-3). İşlemler sınırlı sayıda yeniden denenir [T-1]. Bu nedenle yoğun çakışmada kaybeden istek hata ile dönebilir. Doğruluk bozulmaz.

**"Fark etmez" ataması (geçici, AS-04):** O saatte uygun çalışanlar arasından o gün en az randevusu olan seçilir. Eşitlikte `empId` sırasına bakılır (deterministik). Seçilen çalışanın doluluk dokümanı işlem içinde okunur. Atanan çalışanın adı yanıtta döner (US-05 KK-4).

**Taşıma (FR-21):** Eski ve yeni gün dokümanları aynı işlemde okunur. Yeni aralık eklenir, eski aralık çıkarılır. Eski randevu `rescheduled` olur, `rescheduledTo` alanı doldurulur. Yeni aralık alınamazsa hiçbir değişiklik uygulanmaz, çünkü işlemler kısmen uygulanmaz [T-1] (US-10 KK-1).

**İptal (FR-12, FR-13):** Müşteri çağrısında `start − şimdi < cancelWindowMin` ise `failed-precondition: CANCEL_WINDOW` döner ve istemci işletme iletişim bilgisini gösterir (geçici, AS-03). Yönetici ve ilgili çalışan için bu sınır uygulanmaz. Aralık doluluktan çıkarılır, ilgili hatırlatmalar `cancelled` yapılır.

### 6.3 Hatırlatma zamanlaması (FR-18, FR-24, NFR-04)

**Seçim ölçütleri:**
- Ö1: ±5 dakika sapma (NFR-04).
- Ö2: En ileri randevu tarihine kadar (örnek değer 60 gün, FR-22) zamanlanabilme.
- Ö3: İşletme tarafından yapılan iptal sonrasında hatırlatma gitmemesi (US-08 KK-4).
- Ö4: Ek izin ve platform kısıtı (FR-16, FR-17).
- Ö5: Kurulum ve işletim sadeliği.

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| A. Dakikada bir `onSchedule` + `reminders` taraması | Dakikalık çalışmada gecikme yaklaşık 1 dakika + çalışma süresi | Sınırsız (kayıt Firestore'da) | Gönderimden önce randevu durumu kontrol edilir | FCM izni dışında ek izin yok | Tek iş (3 ücretsiz iş içinde [T-5]) |
| B. Randevu oluşturulurken Cloud Tasks görevi (`scheduleTime`) [T-21] | Hassas | **En fazla 30 gün ileri** [T-20]. 60 günlük ufukla çelişir | İş anında kontrolle sağlanabilir | Ek IAM rolleri gerekir [T-21] | Orta |
| C. Cihazda yerel bildirim (`flutter_local_notifications`) | OEM'e bağlı; bazı cihazlarda arka planda çalışmayabilir [T-22] | iOS'ta yalnızca son 64 bildirim tutulur [T-22] | İşletme iptali cihaz uygulamayı açana kadar bilinmez | Tam zamanlı alarm izinleri (`SCHEDULE_EXACT_ALARM` / `USE_EXACT_ALARM`) [T-22] | Sunucu gerekmez |

**Karar: A.** Ö2 ve Ö3'ü koşulsuz karşılayan tek seçenektir.

**Çalışma şekli:**
1. Her dakika `reminders` koleksiyonunda `status == pending && dueAt <= şimdi` sorgusu çalıştırılır (sayfa başına 200 kayıt).
2. Her kayıt için ayrı bir işlem içinde şu adımlar yapılır: randevu `active` değilse kayıt `cancelled` yapılır. Aksi halde kayıt `sent` yapılır ve FCM gönderilir. Bu sıra, aynı hatırlatmanın iki kez gönderilmesini önler.
3. Boş sorgu da en az 1 okuma olarak faturalanır [T-24]. Bu, günde 1.440 okuma demektir (kaynaktaki verilerden hesaplandı).
4. `dueAt` geçmişte kalmış kayıtlar (ör. randevu 24 saatten kısa süre önce alındıysa) gönderilmez, `cancelled` yapılır. Bu davranış geçicidir ve BA'ya sorulmuştur (S-06).
5. Belge, `onSchedule` için en kısa çalışma aralığını belirtmiyor [T-5]. Dakikalık çalışma kabul testinde doğrulanacaktır (R-03).

### 6.4 Rol yetkilendirmesi (FR-04, NFR-06)

**Seçim ölçütleri:**
- Ö1: Rol değişikliğinin ve çalışan çıkarmanın hemen etkili olması (FR-07).
- Ö2: Bir kullanıcının birden çok işletmede farklı rolde olabilmesi (BA V-01).
- Ö3: Kural maliyeti ve sınırları.

| Seçenek | Ö1 | Ö2 | Ö3 |
|---|---|---|---|
| A. `businesses/{bizId}/members/{uid}` + kurallarda `get()` | Hemen etkili | Doğal olarak desteklenir | `get()` okuma olarak faturalanır. Tek istekte en fazla 10 erişim çağrısı [T-18] |
| B. Auth özel talepleri (custom claims) | Bir sonraki ID token'ına kadar gecikir [T-19] | İşletme sayısı arttıkça 1000 bayt sınırına yaklaşılır [T-19] | Ek okuma yok |

**Karar: A.** Ö1 ve Ö2 güvenlik ve doğruluk açısından belirleyicidir. Kurallar en fazla 2 `get()` kullanır, bu nedenle 10 sınırının altında kalır.

**Müşteri randevu sorgusu:** Kurallar filtre değildir [T-18]. Bu nedenle istemci sorguyu `where('customerUid', '==', uid)` ile yapar. Kural da aynı koşulu kontrol eder.

## 7. Performans

| Hedef | Tasarım önlemi | Ölçüm |
|---|---|---|
| NFR-03: Müsait saat listesi p95 ≤ 2 sn (4G) | `getAvailableSlots` en fazla (1 + çalışan sayısı × 2) doküman okur. Fonksiyon Firestore ile aynı bölgededir [T-11]. Tarih şeridi değiştikçe yalnızca seçilen gün hesaplanır | Hazırlık (staging) projesinde ağ kısıtlamalı cihazla 100 ölçüm, p95 |
| NFR-03: Randevu onayı ≤ 3 sn | Tek işlem, 1–2 doküman okuma, 3–4 yazma. FCM gönderimi tetikleyiciye bırakılır ve yanıtı bekletmez | Aynı ölçüm düzeneği |
| NFR-04: Olay bildirimleri ≤ 1 dk | `onAppointmentWrite` tetikleyicisi doğrudan FCM gönderir | Gönderim zaman damgası ile `updatedAt` farkı loglanır |
| NFR-04: Hatırlatma ±5 dk | Dakikalık tarama (6.3) | `sentAt − dueAt` dağılımı loglanır |
| NFR-02 altında çakışma | Çakışma yalnızca aynı çalışan-gün dokümanında olur. Firestore tek dokümana yazma hızını sınırlar [T-2], ancak bir salonda tek çalışanın bir günü için bu yük beklenmez | 50 eşzamanlı deneme testi (Bölüm 9) |

**Soğuk başlangıç** (callable'ın ilk çağrıda gecikmesi), NFR-03'ü riske atabilir. Bu tasarımda ölçülmemiştir ve R-02 olarak ele alınmıştır.

## 8. Güvenlik ve gizlilik

### 8.1 Erişim denetimi (FR-04, NFR-06)
- **Varsayılan kural:** Her şey reddedilir. Okuma ve yazma izinleri Bölüm 5 tablosuna göre açılır.
- `appointments`, `occupancy`, `reminders`, `customers`, `members` ve `notificationLog` koleksiyonlarına istemci yazamaz.
- Çalışan randevuda yalnızca ad, iletişim ve hizmet bilgisini görür (NFR-06). Randevu dokümanında bu bilgilerden fazlası yoktur.
- `users/{uid}` dokümanını yalnızca sahibi okuyabilir. Çalışan müşterinin profiline erişemez.
- Callable fonksiyonlar `request.auth.uid` ve üyelik dokümanıyla rol kontrolü yapar [T-9]. App Check zorunlu tutulur [T-10].

### 8.2 Veri en aza indirme ve mağaza beyanları (NFR-08, FR-02)
- Kayıtta zorunlu alanlar: ad, e-posta ve şifre (3 alan).
- Analytics, Crashlytics ve Performance SDK'ları MVP'ye eklenmez. `flutterfire configure` bu ürünler için Gradle eklentisi ekleyebilir [T-7]. Eklenirse kaldırılır ya da Data safety beyanına yazılır.
- Data safety ve App Store gizlilik beyanı için veri envanteri Bölüm 5'teki tablodur. FCM kayıt kimliği ve App Check cihaz doğrulaması da beyana eklenir.

### 8.3 Hesap silme (FR-03, NFR-07)
**Seçim ölçütleri:**
- Ö1: Talep anında hesabın kullanılamaz olması (NFR-07).
- Ö2: Gelecekteki randevuların iptal edilip işletmeye bildirilmesi (US-02 KK-2).
- Ö3: Kısmi silme kalmaması.

| Seçenek | Ö1 | Ö2 | Ö3 |
|---|---|---|---|
| A. İstemci `user.delete()` + sunucu tetikleyicisi ile veri temizliği | Hemen | Tetikleyiciye bağlı | Tetikleyici başarısız olursa yetim veri kalır |
| B. `deleteMyAccount` callable: önce hesabı devre dışı bırakır, sonra veriyi temizler, en son Auth kaydını siler | Hemen (ilk adım) | Aynı akışta | Adım adım ilerler. Her adım tekrar çalıştırılabilir (idempotent). Başarısızlıkta yeniden denenebilir |

**Karar: B.**
- İstemci önce yeniden kimlik doğrulaması yapar, çünkü silme yakın zamanda giriş gerektirir [T-13].
- Akış sırası:
  1. Auth hesabı devre dışı bırakılır.
  2. Gelecekteki randevular `cancelled_by_customer` yapılır. Bu adım işletme bildirimini tetikler.
  3. Geçmiş randevularda `customerName` "Silinmiş kullanıcı" yapılır, `customerUid` null yapılır.
  4. `users/{uid}`, `devices` kayıtları ve bekleme listesi kayıtları silinir.
  5. Auth kaydı silinir.
- Geçmiş randevuların anonimleştirilmiş halde saklanması **geçici** karardır (S-03).
- **Web kanalı:** Hosting'deki `/hesap-silme` sayfasında talep e-posta adresi bulunur [T-26]. Operatör, talebi aynı silme kodunu çalıştıran betikle 30 gün içinde işler (NFR-07, KVKK md. 13/2).
- Bilgi ve düzeltme talepleri de aynı kanaldan alınır. Operatör dışa aktarma betiğini kullanır.

### 8.4 Bildirim içeriği (NFR-11, FR-18)
- Bildirim gövdesi sunucuda sabit şablonla üretilir: `{işletme} – {hizmet}, {GG.AA.YYYY} {HH:mm}, {çalışan}`.
- Şablonda serbest metin alanı yoktur. Kampanya içeriği eklenemez.
- İptal nedeni (FR-13) serbest metindir. Bu nedenle bildirimde gösterilmez, yalnızca uygulama içinde gösterilir (S-08).

### 8.5 Veri konumu ve KVKK (AS-08, AS-09)
- Konum seçimi ve yurt dışına aktarım dayanağı BA ve hukuk teyidine bağlıdır (Bölüm 3.1).
- Aydınlatma metninin sahibi (veri sorumlusu) AS-09'a bağlıdır. Metin Hosting'de, sürüm numarasıyla yayımlanır.

## 9. Test stratejisi

| Düzey | Kapsam | Araç | Gereksinim |
|---|---|---|---|
| Birim (TypeScript) | Slot algoritması: mola, kapanış, tampon, en kısa önceden bildirim, en ileri tarih, kapalı gün, izin. US-04 KK-1, US-05 KK-1 ve US-11 KK-1 örnekleri birebir test edilir | Jest | FR-08, FR-09, FR-22 |
| Birim (Dart) | Tarih ve saat biçimi (`dd.MM.yyyy`, `HH:mm`, `tr`), durum etiketleri | `flutter_test` | NFR-10, FR-14 |
| Güvenlik kuralları | Rol başına izinli ve yasak okuma/yazma matrisi. Hedef: yetkisiz erişimde 0 başarı | Firestore emülatörü + `@firebase/rules-unit-testing` (`assertSucceeds` / `assertFails`) [T-28] | FR-04, NFR-06 |
| Entegrasyon | Callable fonksiyonlar + tetikleyiciler emülatörde: oluşturma, iptal (süre içi ve dışı), taşıma atomikliği, no-show sayacı, hesap silme akışı, hatırlatmanın iptalde gitmemesi | Firebase Emulator Suite | FR-10–FR-13, FR-18–FR-21, FR-23, FR-03, US-08 KK-4 |
| Eşzamanlılık | Aynı slot için 50 eşzamanlı `createAppointment`. Beklenen: tam 1 başarı. Emülatördeki kilit davranışının üretimle aynı olduğu doğrulanmadığı için test hazırlık projesinde de koşulur | Node betiği | NFR-02, FR-11 |
| Performans | Bölüm 7'deki ölçümler | Hazırlık projesi + ağ kısıtlamalı cihaz | NFR-03, NFR-04 |
| Erişilebilirlik | `meetsGuideline(androidTapTargetGuideline)`, `iOSTapTargetGuideline`, `labeledTapTargetGuideline`, `textContrastGuideline` [T-27]. En büyük metin ölçeğinde ekran görüntüsü testleri. TalkBack ve VoiceOver ile elle gezinme [T-27] | `flutter_test`, elle test | NFR-09 |
| Cihaz kabul | M gereksinimleri en düşük sürümlü cihazda (iOS 15, Android 7.0) ve güncel cihazda, Android ve iOS'ta ayrı ayrı. Bildirim izni reddedildiğinde akışın sürmesi. iOS push testi fiziksel cihazda [T-7] | Elle test kontrol listesi | NFR-01, FR-16, FR-17 |
| Çevrimdışı | Uçak modunda liste önbellekten gelir. Oluşturma ve iptal açık mesajla reddedilir, kuyruğa alınmaz | Elle test | NFR-13 |
| Kullanılabilirlik | 5 katılımcı, yardımsız randevu, 3 dakika. Hedef ≥ 4/5 | Moderasyonlu test | NFR-05 |

## 10. Riskler

| # | Risk | Etki | Önlem | Gereksinim |
|---|---|---|---|---|
| R-01 | Firestore konumu sonradan değiştirilemez [T-4]. KVKK md. 9 dayanağı belirsiz | Yeniden kurulum ve veri taşıma | Üretim veritabanı AS-08 yanıtından önce açılmaz | AS-08, NFR-07 |
| R-02 | Callable soğuk başlangıcı p95 gecikmeyi artırabilir | NFR-03 hedefi kaçabilir | Erken ölçüm yapılır. Aşılırsa en az bir sıcak örnek (ek maliyet) BA ve bütçe kararına götürülür | NFR-03 |
| R-03 | `onSchedule` için en kısa aralık belgede yazmıyor [T-5] | Dakikalık çalışma mümkün değilse hatırlatma sapması artar | Kabul testinde doğrulanır. Gerekirse 6.3-B (30 gün içindeki randevular için Cloud Tasks) ile birlikte kullanılır | NFR-04 |
| R-04 | Push teslimi işletim sistemine bağlı. Zorla kapatılan uygulamaya mesaj gelmez [T-8]. Android 13+'ta izin varsayılan olarak kapalı (BA K-12) | Hatırlatma ulaşmayabilir | Randevu her zaman uygulama içinde görünür (FR-17). Yedek kanal FR-27 (C, AS-06) | FR-17, FR-18 |
| R-05 | Aynı çalışan-gün dokümanında yoğun çakışmada işlem yeniden denemeleri tükenebilir [T-1] | Kullanıcı hata görür (doğruluk bozulmaz) | "Saat doldu" akışı ve güncel liste gösterilir | NFR-02 |
| R-06 | Luxon'da yaz saati geçişindeki çift anlamlı saatlerin davranışı tanımsız [T-16] | Yaz saati uygulayan bir saat diliminde yanlış slot gösterilebilir | `timeZone` ayarı varsayılan `Europe/Istanbul` dışına açılırsa geçiş günleri için test eklenir. Türkiye'nin yaz saati uygulayıp uygulamadığı bu çalışmada doğrulanmadı | NFR-10 |
| R-07 | NFR-12 (%99,5) yönetilen servislerin erişilebilirliğine bağlı. Firebase SLA değerleri bu çalışmada doğrulanmadı | Hedef garanti edilemez | BA'ya geri soru (S-07) | NFR-12 |
| R-08 | Blaze planı zorunlu [T-6]. Kötüye kullanımla maliyet artabilir | Bütçe aşımı | App Check, bütçe uyarısı | NFR-06 |
| R-09 | Hesapsız müşteri telefonu (FR-20) kişisel veridir ve silme kanalı yalnızca web/e-posta yoluyla mümkündür | KVKK talebi | Operatör betiği hesapsız kayıtları telefonla bulup anonimleştirir | FR-20, NFR-07 |
| R-10 | `flutterfire configure` beyan edilmemiş SDK eklentileri ekleyebilir [T-7] | Data safety uyumsuzluğu | Yayın öncesi bağımlılık listesi kontrol edilir | NFR-08 |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Kayıt, giriş, şifre sıfırlama | `AuthService`; Firebase Auth e-posta/şifre (geçici, AS-02) | 3.1, 4.1 |
| FR-02 | KVKK aydınlatma ve gizlilik politikası | `ConsentScreen`, Hosting sayfaları | 4.1, 4.3, 8.2 |
| FR-03 | Uygulama içi ve web üzerinden hesap silme | `AccountDeletionFlow`, `deleteMyAccount` (seçenek B), Hosting `/hesap-silme` | 4.2, 8.3 |
| FR-04 | Üç rol ve yetki ayrımı | `members` dokümanı + kurallar (6.4-A) | 5, 6.4, 8.1 |
| FR-05 | İşletme profili, saatler, kapalı günler | `AdminSettings`, `businesses/{bizId}` | 4.1, 5 |
| FR-06 | Hizmet tanımı | `services` koleksiyonu | 5 |
| FR-07 | Çalışan ekleme, çıkarma ve davet | `inviteEmployee` / `acceptInvite` / `removeEmployee` | 4.2, 6.4 |
| FR-08 | Çalışan saatleri, mola ve izin | `employees` koleksiyonu; çalışanın kendi düzenlemesi AS-05'e bağlı | 5, 6.1 |
| FR-09 | Müsait saatlerin hesaplanması | `getAvailableSlots`, 15 dakikalık ızgara algoritması (6.1-A) | 6.1 |
| FR-10 | Randevu oluşturma, "fark etmez" | `BookingFlow`, `createAppointment`, atama kuralı (geçici, AS-04) | 4, 6.2 |
| FR-11 | Çakışan randevu olmaması | Doluluk dokümanı + sunucu işlemi (6.2-A) | 6.2 |
| FR-12 | Müşteri iptali ve iptal süresi | `cancelAppointment`, `CANCEL_WINDOW` (geçici, AS-03) | 6.2 |
| FR-13 | Yönetici veya çalışan iptali, neden | `cancelAppointment` (rol kontrolü), `cancelReason` | 4.2, 6.2 |
| FR-14 | Randevu listesi ve durumlar | `MyAppointments`, `status` alanı, collection group sorgusu | 4.1, 5, 6.4 |
| FR-15 | Çalışan ve yönetici takvim görünümleri | `StaffCalendar`, (`employeeId`, `start`) dizini | 4.1, 5 |
| FR-16 | Bağlama uygun bildirim izni | `NotificationService` (ilk randevudan sonra, yerel bayrak) | 3.2, 4.1 |
| FR-17 | İzin olmadan da tam işlev | Randevu verisi uygulama içinde; "Bildirimleri aç" yolu | 4.1, 10 (R-04) |
| FR-18 | Müşteri bildirimleri ve 24 saat hatırlatma | `onAppointmentWrite`, `sendDueReminders` (6.3-A), sabit şablon | 4.2, 6.3, 8.4 |
| FR-19 | Çalışan ve yönetici bildirimleri | `onAppointmentWrite` | 4.2 |
| FR-20 | Hesapsız müşteri adına randevu | `ManualBooking`, `createAppointment` (`customerUid = null`) | 4, 5 |
| FR-21 | Yeniden planlama (atomik) | `rescheduleAppointment`, tek işlem | 6.2 |
| FR-22 | Randevu kuralları (önceden bildirim, ileri tarih, tampon, iptal süresi) | `businesses.rules`, algoritma 6.1 | 5, 6.1 |
| FR-23 | No-show işaretleme ve sayaç | `markNoShow`, `customers.noShowCount` | 4.2, 5 |
| FR-24 | Ayarlanabilir hatırlatma zamanı | `rules.reminderOffsetsMin`, `reminders` kayıtları | 5, 6.3 |
| FR-25 | Onay modu (C) | `approvalMode`, `pending` durumu, `approveAppointment` / `rejectAppointment` | 4.2, 5, 6.2 |
| FR-26 | Bekleme listesi (C) | `waitlist` koleksiyonu; iptalde `onAppointmentWrite` bildirim gönderir | 4.2, 5 |
| FR-27 | SMS/e-posta yedek hatırlatma (C) | Bu sürümde uygulanmaz. Bildirim gönderici kanal soyutlamasıyla yazılır. Sağlayıcı seçimi AS-06 yanıtına bağlıdır | 10 (R-04), 12 |
| FR-28 | Cihaz takvimine ekleme (C) | Bu sürümde uygulanmaz. Kütüphane seçimi ve izin kontrolü iş planlandığında yapılacak | 12 |
| NFR-01 | Android ve iOS'ta aynı işlev | Tek Flutter kod tabanı; en düşük sürümler iOS 15 / API 24 (geçici, AS-07) | 3.1, 3.2, 9 |
| NFR-02 | Çift rezervasyon olmaması (50 eşzamanlı deneme) | 6.2-A; eşzamanlılık testi | 6.2, 9 |
| NFR-03 | Yanıt süresi p95 2 sn / 3 sn | Aynı bölge, az okuma, bildirimin ayrıştırılması | 3.1, 7, 10 (R-02) |
| NFR-04 | Bildirim ≤ 1 dk, hatırlatma ±5 dk | Tetikleyici + dakikalık tarama; `notificationLog` ile tekrar önleme | 6.3, 7 |
| NFR-05 | ≤ 5 adım, 5 kişiden 4'ü 3 dk içinde | 5 adımlı `BookingFlow`; kullanılabilirlik testi | 4.1, 9 |
| NFR-06 | Yetkilendirme ve gizlilik | Kurallar, yalnızca sunucu yazımı, App Check, kural testleri | 6.4, 8.1, 9 |
| NFR-07 | KVKK talepleri 30 gün; silmede hesabın hemen kapanması | `deleteMyAccount` (önce devre dışı bırakma), operatör betikleri | 4.3, 8.3 |
| NFR-08 | Zorunlu alan ≤ 3; mağaza beyanları | 3 alanlı kayıt; analitik SDK'sı yok; veri envanteri | 8.2 |
| NFR-09 | Erişilebilirlik | Flutter Guideline API testleri; TalkBack ve VoiceOver; büyük metin testleri | 9 |
| NFR-10 | Türkçe, 24 saat, GG.AA.YYYY, işletme saat dilimi | Sunucuda Luxon ile etiket; istemcide `intl` `tr` | 3.1, 6.1 |
| NFR-11 | Bildirim içeriği sınırlı, kilit ekranında hassas veri yok | Sabit şablon; not alanı yok; iptal nedeni bildirimde yok | 5, 8.4 |
| NFR-12 | %99,5 erişilebilirlik | Yönetilen servisler; SLA doğrulanmadı | 10 (R-07), 12 |
| NFR-13 | Çevrimdışı davranış | Varsayılan Firestore önbelleği [T-23]; yazmalar callable olduğu için kuyruğa alınmaz; `ConnectivityBanner` | 4.1, 6.2 |

## 12. BA'ya geri sorular

- **S-01 (AS-01, V-01):** Tek salon mu, çok işletmeli platform mu? Çok işletmeli ise işletmeler kendileri mi kaydolacak? Tasarım şu an işletmelerin operatör betiğiyle açıldığını varsayıyor (geçici). Kendi kendine kayıt istenirse yeni bir FR gerekir.
- **S-02 (AS-02):** Birincil kimlik e-posta olarak (geçici) kabul edildi. Telefon girişi istenirse SMS başına ücret oluşur [T-6]. Sosyal giriş istenirse Apple 4.8 kapsamında ek giriş seçeneği gerekir. Onaylıyor musunuz?
- **S-03 (FR-03, AS-09):** Hesap silindiğinde geçmiş randevular işletmenin kayıtlarında anonimleştirilerek tutulsun mu, tamamen silinsin mi? FR-03'teki "mevzuat gereği saklanan veriler" listesi nedir?
- **S-04 (AS-03, AS-04, AS-05):** Geçici kararlar şunlardır: iptal süresi dolunca müşteri iptali engellenir; "fark etmez" seçiminde o gün en az randevusu olan çalışan atanır; çalışan kendi saatlerini düzenleyemez. Onaylıyor musunuz?
- **S-05 (FR-09):** Saat ızgarası adımı 15 dakika (sabit) seçildi. İşletmenin bu adımı değiştirebilmesi gerekiyor mu?
- **S-06 (FR-18, FR-24):** Randevu hatırlatma süresinden daha kısa bir süre kala alınırsa (ör. 3 saat önce alınan randevu için 24 saatlik hatırlatma) hatırlatma gönderilmesin mi, hemen mi gönderilsin? Ayrıca işletme hatırlatma zamanını değiştirdiğinde mevcut randevuların hatırlatmaları da güncellensin mi?
- **S-07 (NFR-12, AS-10):** %99,5 hedefi yönetilen Firebase servislerinin erişilebilirliğine bağlıdır. Bu hedef bir sözleşme yükümlülüğü mü, yoksa iç hedef mi? Ölçüm kimin sorumluluğunda?
- **S-08 (FR-13, NFR-11):** İşletmenin girdiği iptal nedeni serbest metindir ve kişisel veri içerebilir. Bu nedenle bildirimde değil, yalnızca uygulama içinde gösterilmesi önerildi. US-07 KK-1 ("varsa nedenle birlikte bildirim") ile çelişiyor. Hangisi geçerli?
- **S-09 (V-08):** Hiçbir FR serbest metin hizmet notu istemiyor, NFR-11 ise "hizmet notları"ndan söz ediyor. Not alanı gerekiyor mu? Gerekiyorsa hangi rol yazacak ve sağlık verisi girişi nasıl önlenecek?
- **S-10 (AS-08, AS-09):** Veri konumu (öneri: `europe-west3`) ve yurt dışına aktarım dayanağı hukuk tarafından ne zaman teyit edilecek? Üretim veritabanı bu yanıta kadar açılmayacak (R-01).
- **S-11 (FR-20):** Hesapsız müşterinin telefon numarası için aydınlatma nasıl yapılacak, saklama süresi nedir?
- **S-12 (AS-07):** En düşük sürümler, FlutterFire gereği iOS 15 ve Android 7.0 (API 24) olarak alındı [T-7]. İş açısından daha yüksek bir taban kabul edilebilir mi?
- **S-13 (FR-27, FR-28, AS-06):** C öncelikli SMS/e-posta yedeği ve takvime ekleme özellikleri ilk sürüm planına alınacak mı? Alınırsa sağlayıcı ve kütüphane seçimi ayrıca yapılacak.
