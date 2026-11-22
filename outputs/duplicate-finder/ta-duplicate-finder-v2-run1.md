# Fotoğraf Kütüphanesi Kopya Bulucu: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v2
- **Girdi:** /home/user/agent-personas/outputs/duplicate-finder/ba-duplicate-finder-v2-run1.md

---

## 1. Özet

Ürün, sunucusu olmayan tek bir Flutter uygulaması olarak tasarlanmıştır. Bu karar NFR-01, NFR-08 ve BA dokümanının Bölüm 7'sine dayanır. Tüm işlem cihazda yapılır ve uygulama ağ kullanmaz.

**Fotoğraf erişimi ve silme:**
- Kütüphane erişimi, izinler, küçük resimler ve silme işlemleri `photo_manager` eklentisiyle yapılır.
- Android'de silme yalnızca `moveToTrash` ile, yani sistem çöp kutusuna yapılır. Bu yol Android 11 ve üstünde çalışır.
- iOS'ta silme PhotoKit'in `deleteAssets` yoluyla yapılır. Bu yolun öğeleri Son Silinenler'e gönderdiği belgede doğrulanamadı. Bu nedenle sürüm öncesinde bir cihaz testi zorunludur (Risk R2).

**Kopya tespiti (iki katman):**
- **Kesin kopya (FR-04):** Önce dosya boyutu eşit olan adaylar bulunur. Yalnızca bu adayların SHA-256 özeti hesaplanır.
- **Neredeyse aynı (FR-05):** Her fotoğraf için platformdan alınan JPEG küçük resimden iki değer hesaplanır: dHash ve pHash. İki fotoğraf ancak iki Hamming eşiğini birlikte geçerse eşleşir. Bu birleşimi kaynak yazar da önerir [T14].
- **Gruplama:** "Lider kümeleme" kullanılır. Gruplar FR-08'deki saklama sırasına göre kurulur. Her silinecek fotoğrafın, saklanacak fotoğrafa eşik içinde benzer olması garanti edilir (NFR-05).

**Veri ve arayüz:**
- Tarama durumu ve özetler `sqflite` (SQLite) veritabanında saklanır. Böylece kesilen tarama kaldığı yerden sürer ve sonraki taramalar artımlı yapılır (FR-12, NFR-03).
- Ağır işler (görsel çözme, hash, gruplama) ayrı bir isolate'te çalışır. Arayüz bu sırada donmaz (NFR-04).

**Başlıca açık noktalar:**
- `photo_manager`, Android 13'te fotoğraf izniyle birlikte video iznini de zorunlu tutuyor. Bu, Google Play beyanını etkiler (R1).
- Minimum işletim sistemi sürümleri belli değil (AS-06).
- iOS'taki silme davranışı doğrulanmadı (V-04).

Bu noktalar Bölüm 12'de BA'ya soruldu.

---

## 2. Mimari genel bakış

**Mimari tarzı:** Tek süreçli, katmanlı bir mobil istemci. Sunucu, hesap, analitik ve ağ katmanı yoktur. Gerekçeler NFR-01 ve BA Bölüm 7'deki "Sunucu tarafı işleme" maddesidir. Ürün küçük/orta ölçekli olduğu için yalnızca üç katman kullanılır (kapsam kuralları).

```
┌──────────────────────── Sunum katmanı (Flutter widget'ları) ────────────────────────┐
│ İzin açıklama │ Tarama/ilerleme │ Sonuç özeti │ Grup detayı │ Onay/Özet │ Ayarlar    │
└───────────────────────────────────────┬──────────────────────────────────────────────┘
                                        │ (durum akışları / Stream)
┌──────────────────────── Uygulama servisleri (ana isolate) ──────────────────────────┐
│ PermissionService │ ScanCoordinator │ KeeperRecommender │ DeletionPlanner           │
│ DeletionService   │ SettingsService │ DismissedGroupService                         │
└──────┬──────────────────────┬───────────────────────────────┬────────────────────────┘
       │                      │ (bayt dizileri, mesajlaşma)   │
┌──────▼───────────┐  ┌───────▼───────────────────────┐  ┌────▼──────────────────────┐
│ PhotoLibrary-    │  │ HashWorker (arka isolate)     │  │ ScanRepository (sqflite,  │
│ Adapter          │  │ - JPEG çözme (image)          │  │ ana isolate)              │
│ (photo_manager)  │  │ - dHash / pHash               │  │ - assets, scans, groups,  │
│ - izin, listeleme│  │ - SHA-256 (crypto, akışlı)    │  │   dismissed, settings     │
│ - küçük resim    │  │ GroupingEngine (arka isolate) │  └───────────────────────────┘
│ - çöp/silme      │  └───────────────────────────────┘
└──────────────────┘
```

**Tarama akışı (FR-03, FR-04, FR-05, FR-12):**
1. `PermissionService` erişim durumunu belirler: tam, sınırlı veya red (FR-02).
2. `ScanCoordinator` varlık listesini sayfalı olarak alır [T2]. Listeyi veritabanıyla karşılaştırır ve yeni ya da değişen öğeleri işleme kuyruğuna koyar (FR-12).
3. Her öğe için şunlar yapılır:
   - meta veri alınır: boyut, tarih, MIME türü, dosya boyutu [T3][T4],
   - JPEG küçük resim istenir [T5],
   - küçük resim dHash ve pHash hesabı için HashWorker'a gönderilir.
4. Sonuçlar küçük gruplar halinde tek bir veritabanı işlemiyle (transaction) yazılır. Böylece kontrol noktası oluşur (FR-12).
5. Tüm öğeler bitince şu adımlar uygulanır:
   - dosya boyutu çakışan adayların SHA-256 değeri hesaplanır (FR-04),
   - `GroupingEngine` kesin kopya, neredeyse aynı ve benzer gruplarını kurar (FR-04, FR-05, FR-15),
   - `KeeperRecommender` her grupta saklanacak fotoğrafı seçer (FR-08),
   - "Kopya değil" olarak işaretlenmiş gruplar elenir (FR-14).
6. Sonuç ekranı özet değerleri hesaplar (FR-06).

**Silme akışı (FR-09, FR-10, FR-13, NFR-07):**
1. `DeletionPlanner` seçimi doğrular: her grupta en az bir fotoğraf silinmeden kalmalıdır.
2. Uygulamanın kendi onay ekranı gösterilir.
3. `DeletionService` silmeyi platform yoluyla yapar:
   - Android: `moveToTrash`, tek çağrı,
   - iOS: `deleteWithIds`, tek değişiklik bloğu.
4. Sonuç doğrulanır ve özet gösterilir.

---

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Teknoloji seçimleri tablosu

| Seçim | Sürüm (belgedeki) | Amaç / bağlı gereksinim | (a) İzinler | (b) En düşük platform | (c) Belgedeki bilinen kısıtlar |
|---|---|---|---|---|---|
| **Flutter / Dart** | — | Görevde verilmiş kısıt (BA V-01). NFR-08. | — | AS-06'ya bağlı (bkz. 3.3) | Isolate'lerde `dart:ui` ve widget işi yapılamaz. Arka plan isolate'leri platformdan kendiliğinden gelen mesajları alamaz [T12]. |
| **photo_manager** | 3.12.0 [T1] | İzin, sınırlı erişim, listeleme, meta veri, küçük resim, silme. FR-01, FR-02, FR-03, FR-07, FR-09, FR-16. | iOS: `NSPhotoLibraryUsageDescription`. Android 13: "READ_MEDIA_IMAGES and READ_MEDIA_VIDEO are both required whether requesting images or videos" [T2]. Android 14: sınırlı erişim için `READ_MEDIA_VISUAL_USER_SELECTED`. Android 12+: `MANAGE_MEDIA` isteğe bağlıdır ve onay diyaloglarını kaldırır [T1]. | "Android 16, iOS 9.0" (API düzeyi). Kotlin 1.7.22, AGP 7.2.2 [T1]. | `moveToTrash` "supports Android 11 and above" ve daha eski sürümlerde istisna fırlatır [T1][T2]. `deleteWithIds` "will delete the asset completely from your device" [T1][T2]. Silme API'leri deneysel bölümdedir ve semantik sürümlemeye uymadan değişebilir [T1]. İptal (`PMCancelToken`, `cancelAllRequest`) yalnızca iOS/macOS'ta vardır [T1]. HEIC her platformda tam desteklenmez [T1]. iOS, dosyaları uygulama kapsayıcısında önbelleğe alır. Yalnızca iCloud'da duran öğeler indirme gerektirebilir [T1]. iOS 16+ gizli albümlere erişilemez [T1]. `requestLegacyExternalStorage` kullanan uygulamalar Google Play'den reddedilir [T1]. |
| **photo_manager_image_provider** | 2.2.0 [T6] | Grup detayında ve ızgarada önizleme. FR-07. | Belgede izin yazmıyor [T6]. | Belgede yazmıyor. Platformlar: Android, iOS, macOS [T6]. | Belgede kısıt listesi yok. Son yayın yaklaşık iki yıl önce [T6] (bakım riski: R12). |
| **image** (saf Dart) | 4.13.1 [T9] | Platformdan gelen JPEG küçük resmi isolate içinde piksel dizisine çevirmek. FR-05, NFR-04. | Belgede izin yok [T9]. | Belgede yazmıyor. Android/iOS listeleniyor [T9]. | Çözülebilen biçimler arasında HEIC/HEIF yok [T9]. Bu yüzden küçük resim platformdan JPEG olarak istenir [T5]. |
| **crypto** | 3.0.8 [T8] | SHA-256 ile bayt bayt kopya tespiti. FR-04. | Belgede izin yok [T8]. | Belgede yazmıyor. Android/iOS listeleniyor [T8]. | "This library has not been reviewed or vetted by security professionals." [T8] Burada güvenlik amacıyla değil, yalnızca eşitlik kontrolü için kullanılır (R11). Parça parça hesaplama (`startChunkedConversion`) destekleniyor [T8]. |
| **sqflite** | 2.4.6 [T7] | Tarama durumu, hash'ler, gruplar, gizlenen gruplar, ayarlar. FR-11, FR-12, FR-14, NFR-03, NFR-10. | Belgede izin yok [T7]. | Android: "the plugin's `minSdk` is 19". iOS 12 [T7]. | "Prefer the main isolate." Eşzamanlı okuma yok. İşlem (transaction) içinde `db` çağrısı kilitlenmeye yol açar. Desteklenen türler: `int`, `num`, `String`, `Uint8List`, `null` [T7]. Birim testleri `sqflite_common_ffi` ile Dart VM'de koşulabilir [T7]. |
| **flutter_localizations + intl (gen-l10n, ARB)** | intl 0.20.3 [T16] | Türkçe arayüz ve çevrilebilir metinler. NFR-11. | Belgede izin yok [T16]. | Belgede yazmıyor [T16]. | intl'de saat dilimi desteği yok, sayı ayrıştırma yok [T16]. iOS'ta dillerin Xcode Info > Localizations bölümüne de eklenmesi gerekir [T15]. |

**Bilinçli olarak eklenmeyenler:** izin paketi (permission_handler vb.), ayarlar paketi, ML modeli, analitik/çökme raporlama ve arka plan görev paketi.
- `photo_manager` hem izin isteğini hem de `openSetting()` çağrısını kendisi sunuyor [T1]. Ayrı paket gerekmez (FR-02).
- ML modeli gerekmiyor (bkz. 6.4).
- Analitik ve çökme raporlama NFR-01 ile çelişebilir (AS-11, Bölüm 12).
- Arka plan görev paketi gerekmiyor. FR-12 yalnızca "bir sonraki açılışta devam" istiyor, NFR-02 de ön planda ölçülüyor.

### 3.2 Kritik karar: depolama

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim |
|---|---|---|
| Ö1 | Kesilmede tutarlılık: yarım yazılmış veri kalmamalı | FR-12 |
| Ö2 | Artımlı taramada yalnızca değişeni bulabilme | FR-12, NFR-03 |
| Ö3 | Boyut sınırı | NFR-10 |
| Ö4 | Bağımlılık ve karmaşıklık | Kapsam kuralı |

| Alternatif | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **sqflite (SQLite)** | Transaction ve batch var. Hata olursa geri alınır [T7]. | İndeksli sorgu yapılabilir (id, modified_at). | Satır başına yaklaşık 150 bayt. 10.000 öğe için yaklaşık 1,5 MB (tahmin, doğrulanmadı). | Bir paket eklenir. Ana isolate kısıtı var [T7]. |
| JSON dosyası (dart:io) | Atomik yazım kendimiz kurulmalı. Her kontrol noktasında tüm dosya yeniden yazılır. | Tüm dosya belleğe okunur. | Benzer boyut. | Paket gerekmez, ama tutarlılık kodu yazılmalı. |

**Karar:** sqflite seçildi. Ö1 ve Ö2, FR-12 ve NFR-03'ü doğrudan etkiliyor. JSON dosyasında kesilmeye dayanıklılığı elle kurmak, getirdiği sadelikten daha büyük bir risk.

### 3.3 Minimum platform sürümleri (geçici; AS-06)

| Platform | Önerilen en düşük sürüm | Gerekçe |
|---|---|---|
| Android | API 30 (Android 11) | Çöp kutusuna silme yolu (`moveToTrash`) Android 11'den önce yok ve istisna fırlatıyor [T1]. FR-09 ve NFR-07 kalıcı silmeyi yasaklıyor. |
| iOS | 14 | Sınırlı erişim ve `presentLimited` iOS 14+ ile geliyor [T1] (FR-02). sqflite iOS 12, photo_manager iOS 9 istiyor [T7][T1]. Belirleyici olan FR-02. |

Bu sürümler geçicidir. BA'nın AS-06 cevabına bağlıdır (Bölüm 12, S-02).

---

## 4. Bileşenler ve sorumlulukları

| Bileşen | Sorumluluk | İlgili gereksinim |
|---|---|---|
| **PermissionIntroScreen** | Açıklama metnini gösterir ("fotoğraflar cihazdan çıkmaz"). Sistem izin isteği yalnızca bu ekrandaki butona basılınca başlar; açılışta otomatik izin isteği yapılmaz. iOS'ta `PHPhotoLibraryPreventAutomaticLimitedAccessAlert = YES` [T1] ayarlanır. Böylece sınırlı erişim seçici penceresini yalnızca uygulama açar. | FR-01, US-01 |
| **PermissionService** | `requestPermissionExtend` sonucunu üç duruma çevirir: `authorized`, `limited`, `denied` [T1]. Sınırlı erişimde uyarı ve "seçimi genişlet" (`presentLimited`) sunar. Redde açıklama ve `openSetting()` sunar [T1]. Her uygulama açılışında ve her taramadan önce durumu yeniden okur, çünkü Android 14'te seçili öğelere erişim bir süre sonra sona eriyor (BA [K9]). | FR-02, US-02 |
| **PhotoLibraryAdapter** | `photo_manager` için ince bir sarmalayıcıdır. Görevleri: sayfalı listeleme (`getAssetListPaged`) [T2]; albüm listeleme (FR-16); meta veri okuma (`width`, `height`, `createDateTime`, `modifiedDateTime`, `mimeType`, `fileSize`) [T3][T4]; JPEG küçük resim alma (`thumbnailDataWithSize`) [T5]; özgün dosya alma (`originFile`, yalnızca SHA-256 adayları için) [T3]; yerel bulunurluk kontrolü (`isLocallyAvailable`) [T3]. Silme API'leri yalnızca bu katmandan çağrılır. Böylece deneysel API değişikliği tek bir yere dokunur (R3). | FR-03, FR-04, FR-05, FR-07, FR-09, FR-16 |
| **ScanCoordinator** | Taramayı başlatır, duraklatır, sürdürür ve iptal eder. Kuyruğu, ilerleme akışını (işlenen/toplam, %) ve kontrol noktalarını yönetir. Aynı anda en fazla 4 küçük resim isteği gönderir (geçici değer, ölçümle ayarlanır: Bölüm 7). Uygulama arka plana geçince taramayı duraklatır (`AppLifecycleState`). Açılışta yarım kalmış bir tarama varsa kaldığı yerden sürdürür. | FR-03, FR-12, NFR-02, NFR-03, NFR-04 |
| **HashWorker** (uzun ömürlü arka isolate) | JPEG küçük resmi çözer (`image`) ve griye çevirir. dHash ve pHash hesaplar [T13][T14]. SHA-256 hesabını parçalar halinde yapar [T8]. Platform kanalı kullanmaz: baytlar ana isolate'ten mesajla gelir, sonuçlar mesajla döner. Bu sayede isolate kısıtlarına takılmaz [T12]. | FR-04, FR-05, NFR-04 |
| **GroupingEngine** (arka isolate) | Kesin kopya, neredeyse aynı ve benzer grupları kurar (Bölüm 6). Çıktısı deterministiktir: aynı girdi her zaman aynı grupları verir. | FR-04, FR-05, FR-11, FR-15, NFR-05 |
| **KeeperRecommender** | FR-08 kuralını uygular: en yüksek çözünürlük (genişlik × yükseklik), sonra en büyük dosya, sonra en eski tarih, son olarak `asset_id` sırası (yalnızca deterministik eşitlik bozucu). Kullanıcı seçimini değiştirebilir. | FR-08, US-05 |
| **DeletionPlanner** | Seçimi doğrular. Kural: tüm gruplar genelinde her grupta en az bir öğe silinmeden kalmalıdır (bkz. 6.3, çakışan üyelik). "Tüm gruplarda öneriyi uygula" seçeneği, her grupta saklanacak öğe dışındaki tüm öğeleri seçer (FR-13). "Benzer fotoğraflar" kategorisinde hiçbir öğe otomatik seçilmez (FR-15). Silinecek adedi ve açılacak alanı hesaplar. Bir öğe birden fazla grupta olsa bile alanı bir kez sayılır. | FR-06, FR-08, FR-09, FR-13, FR-15 |
| **DeletionService** | Android'de tek bir `moveToTrash(list)` çağrısı yapar [T2]. iOS'ta tek bir `deleteWithIds` çağrısı yapar. Tek çağrı gerekir, çünkü iOS her `performChanges` çağrısında ayrı izin uyarısı gösterir; çok değişiklik tek blokta birleştirilmelidir [T11]. Kalıcı silme API'si Android'de hiç çağrılmaz. Silmeden sonra öğelerin gerçekten kaldırıldığını yeniden sorgulayarak doğrular. Özetteki adet, bu doğrulamadan gelir. | FR-09, FR-10, NFR-07 |
| **ResultScreens** | Sonuç özeti, kategoriler, grup detayı (yan yana önizleme, meta veri, tam ekran), onay ekranı ve silme özeti. Özet ekranı "alan, çöp kutusu/Son Silinenler boşaltılınca tamamen açılır" bilgisini içerir. | FR-06, FR-07, FR-09, FR-10, NFR-06 |
| **SettingsService** | Hassasiyet seviyesi (Sıkı/Normal/Gevşek, varsayılan Normal) ve "Gizlenen gruplar" listesi. Hassasiyet değişirse kullanıcıya "bir sonraki taramada uygulanır" bilgisi verilir. | FR-11, FR-14, US-08, US-09 |
| **DismissedGroupService** | "Kopya değil" işaretlerini saklar, tarama sonuçlarından eler ve işaretin geri alınmasını sağlar. | FR-14 |
| **ScopeSelector** (C) | Tarama kapsamını albüm (`AssetPathEntity` sayfalı listeleme [T2]) veya tarih aralığıyla daraltır. Tarih süzgeci `createDateTime` üzerinden uygulama tarafında uygulanır. | FR-16 |
| **L10n** | gen-l10n ve ARB dosyaları kullanılır. Şablon dil Türkçe (`app_tr.arb`) [T15]. | NFR-11 |

---

## 5. Veri modeli

Tüm veriler uygulamanın özel dizinindeki tek bir SQLite dosyasında tutulur (NFR-01, NFR-10). Fotoğraf pikseli veya küçük resim **saklanmaz**, yalnızca türetilmiş özetler saklanır (NFR-10).

| Tablo | Alanlar | Not / gereksinim |
|---|---|---|
| `assets` | `asset_id TEXT PK`, `modified_at INTEGER`, `created_at INTEGER`, `width INTEGER`, `height INTEGER`, `file_size INTEGER`, `mime_type TEXT`, `dhash INTEGER`, `phash INTEGER`, `sha256 BLOB NULL`, `status TEXT` (`pending`/`hashed`/`skipped_cloud`/`error`), `last_seen_scan INTEGER` | 64 bitlik hash'ler `int` olarak saklanır (sqflite türleri [T7]). `(modified_at)` ve `(file_size)` üzerinde indeks vardır. FR-04, FR-05, FR-07, FR-12 |
| `scans` | `scan_id INTEGER PK`, `started_at`, `finished_at NULL`, `status` (`running`/`paused`/`cancelled`/`done`), `total`, `processed`, `access_level` (`full`/`limited`), `sensitivity`, `scope_json NULL` | Kontrol noktası ve sürdürme (FR-12). Sınırlı erişim uyarısı (FR-02). Kapsam (FR-16). |
| `groups` | `group_id INTEGER PK`, `scan_id`, `kind` (`exact`/`near`/`similar`), `keeper_asset_id`, `signature TEXT` | `signature`: sıralı üye kimliklerinin SHA-256 değeri. FR-04, FR-05, FR-08, FR-14, FR-15 |
| `group_members` | `group_id`, `asset_id`, `selected_for_delete INTEGER` | Kullanıcının seçimi ve saklanacak öğe değişikliği. FR-08, FR-13 |
| `dismissed_groups` | `signature TEXT PK`, `kind`, `member_ids TEXT`, `dismissed_at` | "Kopya değil" işareti (FR-14). Eşleşme kuralı geçicidir (bkz. S-05). |
| `settings` | `key TEXT PK`, `value TEXT` | Hassasiyet (FR-11). |

**Yaşam döngüsü kuralları:**
- Bir taramada görülmeyen öğeler `assets` tablosundan silinir (`last_seen_scan`). Bu öğeler başka yerde silinmiş, erişimi sona ermiş veya sınırlı erişimden çıkarılmış olabilir (FR-02, NFR-10).
- `modified_at` değeri değişen öğe yeniden işlenir (FR-12).
- iOS'ta SHA-256 için alınan `originFile` kopyası, hash hesaplandıktan hemen sonra silinir. Gerekçe: iOS dosyaları uygulama kapsayıcısında önbelleğe alıyor [T1] (NFR-10).

---

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Kesin kopya tespiti (FR-04)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim |
|---|---|---|
| Ö1 | Kesinlik %100 | NFR-05 |
| Ö2 | Tarama süresi: tam dosya okuma maliyeti | NFR-02 |
| Ö3 | Bulutta duran öğeleri indirmeme | NFR-01, AS-09 |
| Ö4 | Sadelik | Kapsam kuralı |

| Alternatif | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| A) Tüm dosyaların SHA-256 değeri | Kriptografik hash'te küçük değişiklik çıktıyı tamamen değiştiriyor (BA [K15]). Pratikte %100. | Her dosya tamamen okunur. 10.000 fotoğrafta en pahalı seçenek. | Her öğe için özgün dosya gerekir. | Basit. |
| **B) Önce dosya boyutu eşitliği, sonra yalnızca çakışanların SHA-256 değeri** | A ile aynı: son karar yine SHA-256 eşitliği. | Yalnızca aynı boyuttaki adaylar okunur. | Yalnızca adaylar için özgün dosya gerekir. | Bir ön adım eklenir. |
| C) Boyut ve meta veri eşitliği (hash yok) | Bayt eşitliğini kanıtlamaz. Ö1'i karşılamaz. | En hızlısı. | İndirme yok. | En basiti. |

**Karar:** B seçildi. Ö1'i A kadar karşılıyor, Ö2 ve Ö3'te A'dan iyi. C, Ö1'i karşılamadığı için elendi.

**Uygulama ayrıntıları:**
- Dosya boyutu `fileSize` ile okunur [T4]. Belge bu değerin nasıl elde edildiğini açıklamıyor; maliyeti performans testinde ölçülür (R5).
- SHA-256 değeri `originFile` akışından `startChunkedConversion` ile parça parça hesaplanır [T8]. Böylece büyük dosyalar belleğe tamamen yüklenmez.
- `isLocallyAvailable(isOrigin: true)` [T3] false dönerse öğe `skipped_cloud` olarak işaretlenir. Bu geçici bir karardır ve AS-09'a bağlıdır.

### 6.2 Neredeyse aynı tespiti (FR-05, FR-11)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim / risk |
|---|---|---|
| Ö1 | Kesinlik ≥ %98 | NFR-05 |
| Ö2 | Duyarlılık ≥ %90: yeniden boyutlandırma, sıkıştırma ve biçim değişikliğine dayanıklılık | NFR-05, FR-05 |
| Ö3 | Hız | NFR-02, NFR-03 |
| Ö4 | Uygulama boyutu ve depolama | NFR-10 |
| Ö5 | "Benzer" fotoğrafları "neredeyse aynı" saymama | FR-05 ile FR-15 ayrımı |
| Ö6 | Hassasiyetin üç kademeye bağlanabilmesi | FR-11 |

| Alternatif | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 | Ö6 |
|---|---|---|---|---|---|---|
| aHash | Yazarın 150.000 görsellik testinde: "about 10x more false-positives", 32 yerine "over 400 images" [T14]. Ölçütü karşılamaz. | Ölçeğe dayanıklı. Gama veya histogram değişikliğinde eşleşmeyi kaçırabiliyor [T13]. | Hızlı [T14]. | 8 bayt. | Zayıf. | Eşik ile ayarlanabilir. |
| dHash tek başına | Aynı testte "very few false positives". Bir örnekte 6 eşleşmenin 4'ü yanlış [T14]. | Aynı testte beklenen eşleşmeler bulundu [T14]. | "as fast as aHash" [T14]. | 8 bayt. | Orta. | Eşik ile ayarlanabilir. |
| pHash tek başına | Aynı testte "No false positives, no false negatives" [T14]. | Gama ve histogram değişikliğine dayanıklı. Yazara göre resmin %25'inden azının değiştiği küçük değişikliklere toleranslı [T13]. | Aynı testte "over 7 hours" (aHash yaklaşık 3,75 saat) [T14]. DCT nedeniyle daha pahalı. | 8 bayt. | İyi. | Eşik ile ayarlanabilir. |
| **dHash + pHash (ikisi de eşik içinde olmalı)** | pHash'in kesinliğine ek bir filtre (çıkarım; NFR-05 test setiyle doğrulanır). | pHash ile aynı. AND koşulu duyarlılığı biraz düşürebilir; eşikler kalibre edilir. | Yazarın önerisi: "use the really fast dHash as a fast filter", sonra pHash [T14]. | 16 bayt. | İyi. | İki eşik çifti üç kademeye bağlanır. |
| Gömme vektörü (MediaPipe) | Benzer fotoğrafları da yakalar (BA [K17]). Ö5 ile çelişir. | Yüksek. | Pixel 6 CPU'da 3,94 ms (BA [K17]). | Model dosyası gerekir. Boyutu doğrulanmadı (BA 3.4). | Zayıf: seri çekimleri "aynı" sayma riski. | Eşik ile ayarlanabilir. |

**Karar:** dHash + pHash seçildi. Ö1 (kesinlik) ve Ö5 (benzer/aynı ayrımı) belirleyici ölçütler. pHash bu iki ölçütte en iyi kaynaklı sonuca sahip [T14]. dHash ise ek ve ucuz bir filtre. Gömme vektörü Ö5 ve Ö4 nedeniyle elendi.

**Hız notu:** [T14]'teki maliyet farkı masaüstünde, tam boy görseller üzerinde ölçülmüştü. Bu tasarımda iki hash de aynı küçük resimden hesaplanır. Mobilde asıl maliyetin küçük resim alma ve çözme olması beklenir (tahmin, doğrulanmadı; Bölüm 7).

**Hesaplama adımları:**
1. Platformdan, en uzun kenarı 64 piksel olan ve **özgün en-boy oranını koruyan** bir JPEG küçük resim istenir [T5]. Kare istenmez, çünkü iOS'taki `fill` modunun kırpma davranışı belgede açıklanmıyor (R7).
2. Küçük resim `image` ile çözülür ve gri tona çevrilir.
3. **dHash:** görsel 9×8 boyutuna küçültülür. Her satırda sol pikselin sağdakinden parlak olup olmadığına göre bit atanır. Sonuç 64 bittir [T14].
4. **pHash:** görsel 32×32 boyutuna küçültülür, 32×32 DCT uygulanır. Sol üst 8×8 düşük frekans katsayıları alınır. DC terimi hariç ortalama hesaplanır ve her katsayının ortalamaya göre konumu bir bit olur. Sonuç 64 bittir [T13].
5. İki hash'in Hamming mesafesi (farklı bit sayısı) hesaplanır.

**Hassasiyet eşikleri (FR-11):** Değerler **geçici başlangıç değerleridir**. NFR-05 test setiyle kalibre edilecektir (BA V-06).

| Kademe | dHash eşiği | pHash eşiği | Dayanak |
|---|---|---|---|
| Sıkı | ≤ 3 | ≤ 2 | Yazarın testinde pHash eşleşmelerinin hepsi "a score of 2 or less" [T14]. |
| Normal (varsayılan) | ≤ 6 | ≤ 4 | Yazarın yorumu: 5'te "a few things may be different" ama büyük olasılıkla benzer [T13]. |
| Gevşek | ≤ 10 | ≤ 8 | Yazarın yorumu: 10'dan büyük değer "likely a different image" [T14]. |

Kaynak eşikleri yazarın kendi görsel seti üzerindeki yorumlarıdır. Mobil kişisel kütüphaneler için doğrulanmamıştır (BA V-05).

### 6.3 Gruplama (FR-04, FR-05, FR-08)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim / risk |
|---|---|---|
| Ö1 | Silinecek her fotoğraf, saklanacak fotoğrafa eşik içinde benzer olmalı (yanlış silme riski) | NFR-05, NFR-07 ruhu, FR-08 |
| Ö2 | Determinizm: aynı girdi aynı grupları vermeli | FR-14 (imza kararlılığı), test edilebilirlik |
| Ö3 | Hesaplama maliyeti | NFR-02, NFR-03 |
| Ö4 | Sadelik | Kapsam kuralı |

| Alternatif | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| Bağlı bileşenler (union-find) | **Zincirleme riski:** A~B ve B~C ise A ile C uzak olsa da aynı gruba düşer. Ö1 garanti edilmez. | Kenar kümesi aynıysa determinist. | Kenar sayısıyla orantılı. | Basit. |
| Tam bağlantı (her çift eşik içinde) | Garanti eder. | Sıraya bağlıdır; ek kural gerekir. | Grup içinde çiftlerin karesi kadar kontrol. | Daha karmaşık. |
| **Lider kümeleme: FR-08 sırasıyla sıralanmış listede her öğe, eşik içindeki ilk lidere katılır; yoksa kendisi lider olur** | Garanti eder: her üye lidere eşik içinde. Lider, FR-08'e göre önerilen saklanacak öğedir. | Sıralama FR-08 kuralına ve `asset_id` sırasına göre tamamen belirli. | Öğe sayısı × lider sayısı kadar kontrol. En kötü durumda 10.000 öğe için yaklaşık 50 milyon 64 bitlik XOR ve bit sayımı (hesaplandı). Arka isolate'te çalışır. | Basit. |

**Karar:** Lider kümeleme seçildi. Ö1 ve Ö2'yi aynı anda karşılayan en basit yöntem bu. Ayrıca FR-08'deki öneri kuralı, gruplamanın doğal bir sonucu oluyor.

Kullanıcı saklanacak öğeyi değiştirirse yeni saklanacak öğe ile diğer üyeler arasındaki mesafe en fazla iki eşik olabilir. Kullanıcının bilinçli seçimi olduğu için bu kabul edilir. Ölçüm Ö3'ü karşılamazsa, aynı sonucu veren bir BK-tree arama yapısına geçilir (R9).

**Sıralama ve üyelik kuralları:**
1. Kesin kopya grupları SHA-256 eşitliğiyle kurulur. Bir grubun lideri FR-08 sırasıyla seçilir.
2. Neredeyse aynı kümelemesine şunlar girer: kesin kopya grubunda olmayan öğeler ve kesin kopya gruplarının liderleri. Bir kesin kopya grubunun yalnızca lideri girer, böylece aynı baytlar tekrar gruplanmaz.
3. Bu kuralla bir öğe en fazla bir kesin kopya grubunda ve bir neredeyse aynı grubunda yer alabilir. İkisinde birden yer alırsa, her iki grupta da lider (saklanacak) konumundadır.
4. DeletionPlanner, kullanıcının saklanacak öğeyi değiştirmesinden sonra da şunu doğrular: **gruplar genelinde** her grupta en az bir öğe silinmeden kalmalıdır (FR-08, US-05/AC-4).
5. Öneri kuralı değişirse (AS-12), yalnızca sıralama karşılaştırıcısı değişir.

### 6.4 Benzer fotoğraflar (FR-15, C önceliği)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim |
|---|---|---|
| Ö1 | Seri çekimi yakalama | FR-15 |
| Ö2 | Ekran görüntüleri gibi alakasız öğeleri "benzer" saymama | BA [K5] kullanıcı şikâyeti, FR-15 |
| Ö3 | Uygulama boyutu | NFR-10 |
| Ö4 | Sadelik | Kapsam kuralı |

| Alternatif | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **Çekim zamanı penceresi + gevşek pHash eşiği** | "Aynı anda çekilmiş" tanımına doğrudan uyar. | Zaman penceresi farklı zamanlardaki ekran görüntülerini dışarıda bırakır. | Model yok. | Mevcut hash'lerle yapılır. |
| Gömme vektörü (MediaPipe) | Yüksek (BA [K17]). | Zayıf: içerik benzerliği zamana bakmaz. | Model dosyası gerekir. | Yeni bağımlılık ve model. |

**Karar:** Zaman penceresi ve gevşek pHash eşiği seçildi (Ö2, Ö3 ve Ö4'e göre).

**Kurallar:**
- Aday çift koşulları: `created_at` farkı en fazla 10 saniye (geçici) ve pHash mesafesi en fazla 16 (geçici).
- Kesin kopya veya neredeyse aynı gruplarında zaten birlikte bulunan çiftler aday sayılmaz.
- Gruplama yine lider kümelemeyle yapılır. `kind = similar` olarak saklanır.
- Bu gruplarda `selected_for_delete` hiçbir zaman otomatik olarak ayarlanmaz. "Tüm gruplarda öneriyi uygula" bu kategoriyi atlar.
- "Aynı anda" ifadesinin tanımı BA'ya soruldu (S-06).

### 6.5 Silme yolu (FR-09, NFR-07)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı olduğu gereksinim |
|---|---|---|
| Ö1 | Geri alınabilirlik %100 | NFR-07 |
| Ö2 | Kullanıcı onayı | FR-09 |
| Ö3 | Ek izin ve beyan yükü | NFR-09, AS-08 |

| Alternatif | Ö1 | Ö2 | Ö3 |
|---|---|---|---|
| **Android: `moveToTrash(list)`** | Çöp kutusuna gider (BA [K10]). Android 11+ [T1]. | Sistem onay diyaloğu + uygulama onayı. | Ek izin yok. |
| Android: `deleteWithIds` | "delete the asset completely" [T2]. NFR-07 ile çelişir. | Sistem onayı. | Ek izin yok. |
| Android: `MANAGE_MEDIA` + `moveToTrash` | Çöp kutusuna gider. | Sistem onayı kalkar [T1]; yalnızca uygulama onayı kalır. | Özel izin ve ayar ekranı (AS-08). |
| **iOS: `deleteWithIds` (PhotoKit `deleteAssets`)** | Son Silinenler'e gittiği Apple belgesinde yazmıyor [T10] (BA V-04). **Doğrulanmadı.** | iOS her `performChanges` çağrısında izin uyarısı gösteriyor [T11]. | Ek izin yok. |

**Karar:**
- Android'de yalnızca `moveToTrash` kullanılır. `MANAGE_MEDIA` istenmez; bu karar AS-08 cevaplanana kadar geçerlidir.
- iOS'ta `deleteWithIds` tek çağrıyla kullanılır. iOS silme özelliği, ancak gerçek cihazda Son Silinenler testi geçerse (Bölüm 9, T-DEL-iOS) sürüme alınır (R2).
- Kalıcı silme kodu yolu Android'de hiç çağrılmaz. Bu kural bir statik kontrolle test edilir.

---

## 7. Performans

| Hedef | Tasarım kararı | Doğrulama |
|---|---|---|
| NFR-02: 10.000 fotoğraf ≤ 10 dk | Bütçe fotoğraf başına ortalama 60 ms (hesaplandı: 600 s / 10.000). Bu bütçe şu kararlarla korunur: küçük resim 64 piksel ve JPEG; en fazla 4 eşzamanlı istek; çözme ve hash uzun ömürlü bir isolate'te [T12]; SHA-256 yalnızca boyutu çakışan adaylarda (6.1); veritabanı yazımı batch ve transaction ile [T7]. | İlk geliştirme adımında bir ölçüm prototipi yapılır. Referans cihaz AS-07'ye bağlıdır. Bütçe aşılırsa sırayla şunlar denenir: eşzamanlılık ayarı, küçük resim boyutu, `fileSize` maliyetinin ölçülmesi (R5). |
| NFR-03: 100 yeni fotoğraf ≤ 30 s | Yalnızca yeni veya `modified_at` değeri değişen öğeler işlenir. Gruplama, saklanan hash'lerle bellekte yeniden yapılır. Görseller yeniden işlenmez. | Ölçüm testi (Bölüm 9). |
| NFR-04: arayüz donmaz; iptal ≤ 2 s | Ana isolate'te yalnızca platform kanalı istekleri ve sqflite çalışır. sqflite için ana isolate önerilir [T7]. İptal bayrağı her öğeden önce kontrol edilir. iOS'ta bekleyen küçük resim istekleri `cancelAllRequest` ile iptal edilir [T1]. Android'de bu API yok [T1]; en fazla 4 uçuştaki isteğin bitmesi beklenir. | İptal süresi testi (Bölüm 9). |
| NFR-10: 10.000 fotoğraf ≤ 50 MB | Küçük resimler saklanmaz. `assets` satırı yaklaşık 150 bayt, 10.000 satır yaklaşık 1,5 MB (tahmin). iOS `originFile` geçici kopyaları hemen silinir [T1]. | Tarama sonrası uygulama veri dizini boyutu ölçülür. |

Bu bölümdeki süreler kaynaklı değildir, tahmindir. BA da hedefleri doğrulanmamış olarak işaretlemiştir (V-05).

---

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Ağ | Uygulamada ağ istemcisi, analitik veya çökme raporlama yoktur. Android sürüm manifestinde `INTERNET` izni bildirilmez. Bunun bir bağımlılık tarafından eklenip eklenmediği, birleştirilmiş manifest denetimiyle doğrulanır. iOS'ta bu tür bir izin kısıtı yok; ağ trafiği izleme testiyle doğrulanır. | NFR-01, NFR-09 |
| Bulut öğeleri | Yalnızca iCloud'da duran öğeler için özgün dosya indirilmez (`isLocallyAvailable` kontrolü [T3]). Geçici karar, AS-09'a bağlı. | NFR-01, V-02 |
| Veri amacı | Hash'ler yalnızca kopya tespitinde kullanılır, cihaz dışına çıkmaz. App Store 5.1.2(iv) metni "build a contact database" ifadesini kullanıyor; 5.1.2(vi) ise Photo API verisinin pazarlama, reklam ve veri madenciliğinde kullanımını yasaklıyor [T19]. Yerel hash önbelleği bu iki maddeyle çelişmiyor (TA yorumu; BA'nın aktarımındaki fark: S-11). | NFR-01, NFR-09 |
| Yerel depolama | Veritabanı uygulamanın özel dizininde tutulur. Piksel verisi ve küçük resim saklanmaz. | NFR-01, NFR-10 |
| İzin metinleri | iOS'ta `NSPhotoLibraryUsageDescription` kullanılır ve metin tam kütüphane taramasının nedenini açıkça anlatır (BA [K13]). Android'de Play fotoğraf/video izni beyanı ve Data safety formu doldurulur (BA [K8][K11]). Manifestte `READ_MEDIA_VIDEO` da bildirilmek zorunda [T2] (R1). | FR-01, NFR-09 |
| Yasaklı bayrak | `requestLegacyExternalStorage` kullanılmaz [T1]. | NFR-09 |
| Hash çakışma saldırıları | Algısal hash'lere kasıtlı çakışma üretilebiliyor (BA [K15][K18]). Önlemler: otomatik silme yok; her silme kullanıcı onayından ve platform onayından geçiyor; saklanacak öğe asla silinmiyor; silinen öğe geri alınabilir alana gidiyor. | NFR-05, NFR-07 |

---

## 9. Test stratejisi

| Düzey | Kapsam | Araç / yöntem | Gereksinim |
|---|---|---|---|
| Birim | dHash ve pHash bit üretimi: sabit sentetik görsellerle "altın" çıktılar. Hamming mesafesi. SHA-256 akışı. | `flutter_test` | FR-04, FR-05 |
| Birim | Gruplama determinizmi: aynı girdi aynı grupları ve imzaları vermeli. Lider kümeleme garantisi: her üye lidere eşik içinde olmalı. Kesin kopya liderinin neredeyse aynı grubuna katılması. | `flutter_test` | FR-05, FR-14, NFR-05 |
| Birim | FR-08 sıralaması ve eşitlik bozucu. Grup genelinde "en az bir öğe kalır" kuralı. FR-13 toplu seçim. FR-15 gruplarında otomatik seçim olmaması. | `flutter_test` | FR-08, FR-13, FR-15, US-05/AC-4, US-07, US-10 |
| Birim | Repository: kontrol noktası, artımlı fark bulma, gizlenen grup eleme. | `sqflite_common_ffi` ile Dart VM [T7] | FR-12, FR-14 |
| Doğruluk | Ekibin hazırlayacağı etiketli set (en az 1.000 fotoğraf; BA NFR-05). Aynı cihaz hattıyla (platform küçük resmi) çalışan bir doğruluk testi. Her kademe için kesinlik ve duyarlılık raporlanır. Eşikler bu sonuçlara göre son halini alır (6.2). | Cihaz üzerinde entegrasyon testi | NFR-05, FR-11 |
| Performans | 10.000 ve 100 fotoğraflık senaryolar. Referans cihaz AS-07'ye bağlı. | Cihaz üzerinde zaman ölçümü | NFR-02, NFR-03, NFR-10 |
| Yanıt verme | Tarama sırasında gezinme. İptalden durmaya kadar geçen süre ≤ 2 s. Arka plana alma ve yeniden açmada sürdürme. | Entegrasyon testi | NFR-04, FR-03, FR-12, US-03 |
| İzin | Tam, sınırlı ve red durumları. iOS 14+ ve Android 14+ sınırlı erişim. Seçimi genişletme. Ayarlara gitme. İzin penceresinin yalnızca butonla açılması. | Gerçek cihazlarda manuel ve entegrasyon testi | FR-01, FR-02, US-01, US-02 |
| Silme (Android) | Her silmeden sonra öğelerin sistem çöp kutusunda bulunması (%100). Sistem diyaloğunda "iptal" seçilirse hiçbir şey silinmemesi. | Gerçek cihaz | FR-09, FR-10, NFR-07 |
| **Silme (iOS): T-DEL-iOS** | `deleteWithIds` sonrası öğelerin Son Silinenler albümünde bulunması. Sistem uyarısının bir kez görünmesi. **Sürüm kapısıdır:** geçmezse iOS silme özelliği yayımlanmaz. | Gerçek cihaz | FR-09, NFR-07, BA V-04 |
| Gizlilik | Tarama ve silme sırasında ağ trafiği 0 bayt. Birleştirilmiş Android manifestinde `INTERNET` izni yok. | Proxy veya ağ izleme, manifest denetimi | NFR-01 |
| Erişilebilirlik | `androidTapTargetGuideline` (48×48), `iOSTapTargetGuideline` (44×44), `labeledTapTargetGuideline`, `textContrastGuideline` [T18]. Flutter'ın kontrast kılavuzu büyük metin için 3:1 öneriyor [T18], bu yüzden 4,5:1 hedefi ayrıca manuel kontrol edilir. TalkBack ve VoiceOver ile manuel test [T17]. Kaydırma/sürükleme işlevlerinin buton alternatifi kontrol edilir. | `flutter_test` ve manuel test | NFR-06 |
| Yerelleştirme | Kodda sabit metin olmaması. Tüm metinlerin ARB dosyasında olması. | Statik kontrol | NFR-11 |
| Uyumluluk | Hedef minimum sürümler (3.3) ve güncel sürümlerde kabul testleri. | Cihaz matrisi | NFR-08 |
| Mağaza | Play izin beyanı ve Data safety; iOS amaç metni için kontrol listesi. | Manuel | NFR-09 |

---

## 10. Riskler

| ID | Risk | Etki | Önlem | İlgili |
|---|---|---|---|---|
| R1 | `photo_manager`, Android 13'te fotoğraf istense bile `READ_MEDIA_IMAGES` ve `READ_MEDIA_VIDEO` iznini birlikte zorunlu tutuyor [T2]. Video kapsam dışında (BA Bölüm 7). Play beyanının videoyu da kapsaması gerekebilir ve beyanın reddedilme riski artar (BA [K8], AS-02). | Yüksek | BA'ya soruldu (S-01). Gerekirse yalnızca görsel izniyle çalışan bir platform katmanı değerlendirilir (ek iş). | NFR-09, FR-01, FR-02 |
| R2 | iOS'ta `deleteAssets` sonrası öğelerin Son Silinenler'e gittiği Apple belgesinde yazmıyor [T10]. `photo_manager` belgesi ise `deleteWithIds` için "delete the asset completely" diyor [T2]. | Yüksek | T-DEL-iOS sürüm kapısı. Geçmezse iOS silme yayımlanmaz ve BA'ya dönülür (S-03). | NFR-07, FR-09 |
| R3 | Silme API'leri `photo_manager`'ın deneysel bölümünde ve semantik sürümlemeye uymadan değişebilir [T1]. | Orta | Paket sürümü sabitlenir. Silme çağrıları yalnızca PhotoLibraryAdapter'da yapılır. Silme testleri her sürüm yükseltmesinde koşulur. | FR-09, NFR-07 |
| R4 | Android 11'den önce çöp kutusu yolu yok [T1]. | Orta | Önerilen minimum API 30 (AS-06, S-02). | NFR-08, FR-09 |
| R5 | `fileSize` değerinin nasıl elde edildiği belgelenmemiş [T4]. Dosyayı okuyorsa NFR-02 bütçesini aşabilir. | Orta | İlk ölçüm prototipinde ölçülür. | NFR-02, FR-04, FR-06 |
| R6 | iOS'ta `originFile` dosyaları uygulama kapsayıcısında önbelleğe alıyor. Bulut öğeleri indirme gerektirebilir [T1]. | Orta | Yalnızca adaylar için alınır. Hash'ten sonra silinir. Yerel olmayan öğe atlanır. | NFR-10, NFR-01, AS-09 |
| R7 | iOS küçük resimlerinde `ResizeContentMode.fill` kullanılıyor [T5]. Kırpma davranışı belgede yok. | Düşük–Orta | En-boy oranını koruyan boyut istenir. Doğruluk testinde iOS ve Android ayrı ölçülür. | FR-05, NFR-05 |
| R8 | Performans hedefleri ve eşikler kaynaklı değil (BA V-05, V-06). | Orta | Ölçüm prototipi ve etiketli test seti. | NFR-02, NFR-03, NFR-05 |
| R9 | Lider kümeleme en kötü durumda öğe sayısının karesi kadar karşılaştırma yapar. Büyük kütüphanede (P2) gruplama süresi uzayabilir. | Düşük–Orta | Arka isolate'te çalışır. Gerekirse BK-tree kullanılır (aynı sonuç). | NFR-02, NFR-03 |
| R10 | Android'de küçük resim isteği için iptal API'si yok [T1]. | Düşük | Eşzamanlılık en fazla 4 tutulur. İptal süresi ölçülür. | NFR-04 |
| R11 | `crypto` paketi güvenlik uzmanlarınca denetlenmemiş [T8]. | Düşük | Yalnızca eşitlik kontrolü için kullanılıyor, güvenlik amacı yok. | FR-04 |
| R12 | `photo_manager_image_provider`'ın son yayını yaklaşık iki yıl önce [T6]. Yeni Flutter sürümleriyle uyumsuzluk çıkabilir. | Düşük | Gerekirse aynı işi `thumbnailDataWithSize` [T5] ile `Image.memory` yapar. | FR-07 |
| R13 | sqflite'ta eşzamanlı okuma yok ve sqflite ana isolate'te çalışmalı [T7]. Büyük yazımlar arayüzü yavaşlatabilir. | Düşük | Küçük batch'ler halinde, transaction içinde yazılır. | NFR-04 |
| R14 | iOS 16+ gizli albüm öğelerine erişilemez [T1]. Bu öğeler taramaya girmez. | Düşük | Sonuç ekranındaki "eksik olabilir" bilgisinin bu durumu kapsayıp kapsamayacağı BA'ya soruldu (S-09). | FR-02 |

---

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | İzin öncesi açıklama ekranı; izin yalnızca butonla istenir | PermissionIntroScreen; `requestPermissionExtend` butondan çağrılır; `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` | 3.1, 4, 8 |
| FR-02 | Tam / sınırlı / red durumlarının ayrı ele alınması | PermissionService; `PermissionState.limited`, `presentLimited`, `openSetting` | 3.1, 3.3, 4 |
| FR-03 | İstenen anda tarama, ilerleme, iptal | ScanCoordinator; ilerleme akışı; iptal bayrağı | 2, 4, 7 |
| FR-04 | Bayt bayt aynı fotoğraflar "Kesin kopya" | Boyut ön süzgeci + SHA-256 (`crypto`) | 6.1 |
| FR-05 | Neredeyse aynı fotoğraflar ayrı kategoride | dHash + pHash (AND) + lider kümeleme; `kind = near` | 6.2, 6.3 |
| FR-06 | Grup sayısı, kopya sayısı, tahmini alan | DeletionPlanner alan hesabı (öğe bir kez sayılır); ResultScreens | 4 |
| FR-07 | Yan yana önizleme, meta veri, tam ekran | ResultScreens; `photo_manager_image_provider`; AssetEntity meta verisi | 3.1, 4, 5 |
| FR-08 | Saklanacak önerisi; kural; tüm öğeler silinemez | KeeperRecommender; lider = FR-08 sırası; DeletionPlanner grup geneli kural | 4, 6.3 |
| FR-09 | Geri alınabilir alana silme, onay ekranı, kalıcı silme yok | DeletionService: Android `moveToTrash`, iOS `deleteWithIds` + T-DEL-iOS kapısı | 6.5, 9, 10 (R2) |
| FR-10 | Silme özeti ve çöp kutusu bilgisi | DeletionService doğrulaması; ResultScreens özet | 4 |
| FR-11 | Üç kademeli hassasiyet | SettingsService; kademe başına eşik çifti | 6.2 |
| FR-12 | Kaldığı yerden devam; artımlı tarama | ScanCoordinator kontrol noktaları; `assets.modified_at`, `scans.status` | 2, 5, 7 |
| FR-13 | Tüm gruplarda öneriyi uygula | DeletionPlanner toplu seçim; onay ekranı zorunlu | 4 |
| FR-14 | "Kopya değil" işareti, gizleme ve geri alma | DismissedGroupService; `dismissed_groups.signature` | 4, 5 |
| FR-15 | Benzer fotoğraflar ayrı kategoride, otomatik seçim yok | Zaman penceresi + gevşek pHash; `kind = similar`; otomatik seçim yok | 6.4 |
| FR-16 | Albüm veya tarih aralığıyla sınırlama | ScopeSelector; `AssetPathEntity` sayfalı listeleme; `created_at` süzgeci | 4 |
| NFR-01 | Fotoğraf verisi cihazdan çıkmaz | Sunucusuz mimari; ağ istemcisi yok; `INTERNET` izni yok; bulut öğesi indirilmez | 2, 8, 9 |
| NFR-02 | 10.000 fotoğraf ≤ 10 dk | Küçük resim boyutu, eşzamanlılık, isolate, boyut ön süzgeci; ölçüm prototipi | 6.1, 7 |
| NFR-03 | 100 yeni fotoğraf ≤ 30 s | Artımlı işleme; saklanan hash'lerle yeniden gruplama | 5, 7 |
| NFR-04 | Arayüz donmaz; iptal ≤ 2 s | HashWorker ve GroupingEngine arka isolate'te; iptal bayrağı; iOS `cancelAllRequest` | 4, 7 |
| NFR-05 | Doğruluk hedefleri | dHash + pHash AND; lider kümeleme garantisi; etiketli setle kalibrasyon | 6.2, 6.3, 9 |
| NFR-06 | Erişilebilirlik (kontrast, dokunma hedefi, buton alternatifi, metin alternatifi) | Flutter Guideline testleri (48/44 ≥ 24); önizlemelere anlamsal etiket; kaydırmaya buton alternatifi; manuel ekran okuyucu testi | 9 |
| NFR-07 | Hiçbir silme geri alınabilir alanı atlamaz | Kalıcı silme yolu yok (Android); iOS sürüm kapısı | 6.5, 9, 10 |
| NFR-08 | iOS ve Android uyumluluğu | Flutter; önerilen minimum iOS 14 / Android API 30 (geçici) | 3.3 |
| NFR-09 | Mağaza uyumu | İzin metinleri; Play beyanı ve Data safety; `requestLegacyExternalStorage` yok; R1 | 8, 10 |
| NFR-10 | Uygulama verisi ≤ 50 MB / 10.000 fotoğraf | Yalnızca özet veri; küçük resim saklanmaz; geçici dosyalar silinir | 5, 7 |
| NFR-11 | Türkçe arayüz, çevrilebilir metinler | gen-l10n + ARB (`app_tr.arb`); Xcode Localizations | 3.1, 4 |

---

## 12. BA'ya geri sorular

| ID | Soru | İlgili |
|---|---|---|
| S-01 | Seçilen `photo_manager` eklentisi, Android 13'te yalnızca fotoğraf istense bile `READ_MEDIA_VIDEO` iznini de zorunlu tutuyor [T2]. Video kapsam dışı (Bölüm 7, AS-01). Play beyanının video iznini de kapsaması kabul edilebilir mi? Kabul edilemezse yalnızca görsel izniyle çalışan özel bir platform katmanı gerekir (ek iş). | NFR-09, AS-01, AS-02 |
| S-02 | AS-06 için öneri: iOS 14 ve Android 11 (API 30). Gerekçe: FR-02 sınırlı erişim ve FR-09 çöp kutusu yolu. Android 10 ve altı desteklenmeyecek mi? Yoksa bu sürümlerde silme özelliği kapalı mı olacak? | NFR-08, FR-09 |
| S-03 | iOS'ta silinen öğelerin Son Silinenler'e gittiği doğrulanamadı (V-04) [T10]. Cihaz testi bunu doğrulamazsa iOS'ta silme özelliği çıkarılabilir mi, yoksa başka bir ürün kararı mı gerekir? | FR-09, NFR-07 |
| S-04 | Silmede iki onay olacak: uygulamanın onay ekranı (FR-09) ve sistemin onay diyaloğu (Android [K10]; iOS her değişiklik bloğu için [T11]). Bu çift onay kabul edilebilir mi? | FR-09, AS-08 |
| S-05 | "Kopya değil" olarak işaretlenmiş bir gruba sonradan yeni bir fotoğraf eklenirse grup yeniden gösterilmeli mi? Geçici karar: üye kümesi değişen grup yeniden gösterilir (imza eşitliği). | FR-14 |
| S-06 | FR-15'teki "aynı anda çekilmiş" ifadesi için kabul edilen süre penceresi nedir? Geçici değer 10 saniye. | FR-15 |
| S-07 | Bir kesin kopya grubunun saklanacak öğesi, aynı zamanda bir neredeyse aynı grubunun saklanacak öğesi olabilir (6.3). Aynı fotoğrafın iki kategoride görünmesi kabul edilebilir mi? | FR-04, FR-05, FR-06 |
| S-08 | FR-12: yarım kalan tarama açılışta otomatik mi sürdürülmeli, yoksa kullanıcıya "Devam et" mi sorulmalı? Geçici karar: otomatik sürdürme, iptal edilebilir. | FR-12, US-03 |
| S-09 | iOS 16+ gizli albüm öğelerine erişilemiyor [T1]. Sonuç ekranında bu durum için de "sonuçlar eksik olabilir" bilgisi verilmeli mi? | FR-02 |
| S-10 | AS-09 cevaplanana kadar yalnızca bulutta duran öğeler kesin kopya hesabında atlanıyor (`skipped_cloud`). Bu öğe sayısı kullanıcıya gösterilmeli mi? | FR-04, NFR-01, AS-09 |
| S-11 | BA dokümanı App Store 5.1.2(iv) maddesini "Photos verisiyle kendi kullanımı için veritabanı oluşturmak yasak" diye aktarıyor (BA 3.3 ve [K13]). Kılavuzun metni ise "to build a contact database" diyor [T19]. Yerel hash önbelleğinin bu maddeyle çelişmediği yorumu (Bölüm 8) BA tarafından da kabul ediliyor mu? Gerekirse BA dokümanındaki aktarım düzeltilmeli. | NFR-01, NFR-09 |
| S-12 | NFR-05 test setinde "neredeyse aynı olmayan ama birbirine çok benzeyen" negatif örnekler (seri çekimler, ekran görüntüleri) de olacak mı? Kesinlik ölçümünün anlamlı olması için bu örnekler gerekir. Seti kim, ne zaman hazırlayacak? | NFR-05, V-06 |
| S-13 | Düzenlenmiş fotoğraflar (AS-10) için şu davranış kabul edilebilir mi? Küçük resim, fotoğrafın görüntülenen (düzenlenmiş) halinden alınır; bu yüzden kırpılmış veya filtreli bir fotoğraf eşiklere göre eşleşmeyebilir. | FR-05, AS-10 |
| S-14 | AS-11: Çökme raporlama istenirse NFR-01 ve Data safety beyanları değişir. Tasarım şu an hiçbir ağ bileşeni içermiyor. Bu doğru kabul mü? | NFR-01, NFR-09 |

### Kaynaklar

Tüm kaynaklar 2026-11-22 tarihinde WebFetch ile açılıp okundu. "BA [Kn]" biçimindeki atıflar girdi olan BA dokümanının kaynaklarıdır; bu dokümanda yeniden açılmadılar.

| ID | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| T1 | pub.dev – photo_manager | https://pub.dev/packages/photo_manager | Sürüm 3.12.0. "Android 16, iOS 9.0" minimum. İzinler (iOS `NSPhotoLibraryUsageDescription`, Android 13/14 `READ_MEDIA_*`, `MANAGE_MEDIA`). `presentLimited` iOS 14+/Android 14+. `PHPhotoLibraryPreventAutomaticLimitedAccessAlert`. `openSetting()`. `moveToTrash` "supports Android 11 and above". `deleteWithIds` kalıcı siler. Deneysel API uyarısı. HEIC kısıtı. İptal yalnızca iOS/macOS. iOS önbellek ve iCloud indirme. Gizli albüm kısıtı. `requestLegacyExternalStorage` reddi. |
| T2 | photo_manager README (GitHub raw) | https://raw.githubusercontent.com/fluttercandies/flutter_photo_manager/main/README.md | "READ_MEDIA_IMAGES and READ_MEDIA_VIDEO are both required whether requesting images or videos." `moveToTrash(list)` imzası. `deleteWithIds`: "This method will delete the asset completely from your device." Sayfalı listeleme API'leri. |
| T3 | pub.dev API – AssetEntity | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html | `width`, `height`, `createDateTime`, `modifiedDateTime`, `mimeType`, `originFile`, `isLocallyAvailable`, `thumbnailDataWithSize`, `fileSize`. |
| T4 | pub.dev API – AssetEntity.fileSize | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity/fileSize.html | "The file size of the asset in bytes." Nasıl elde edildiği belgelenmemiş. |
| T5 | pub.dev API – AssetEntity.thumbnailDataWithSize | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity/thumbnailDataWithSize.html | `format` varsayılanı jpeg. `cancelToken`. iOS'ta `ResizeContentMode.fill`. |
| T6 | pub.dev – photo_manager_image_provider | https://pub.dev/packages/photo_manager_image_provider | Sürüm 2.2.0, yaklaşık iki yıl önce yayımlandı. `AssetEntityImage`, `AssetEntityImageProvider`. Platformlar Android/iOS/macOS. |
| T7 | pub.dev – sqflite | https://pub.dev/packages/sqflite | Sürüm 2.4.6. Android "minSdk is 19", iOS 12. "Prefer the main isolate." Eşzamanlı okuma yok. Transaction kilitlenme uyarısı. Desteklenen türler. Testler için `sqflite_common_ffi`. |
| T8 | pub.dev – crypto | https://pub.dev/packages/crypto | Sürüm 3.0.8. SHA-256. `startChunkedConversion`. "This library has not been reviewed or vetted by security professionals." |
| T9 | pub.dev – image | https://pub.dev/packages/image | Sürüm 4.13.1. Saf Dart. Desteklenen biçimler; HEIC/HEIF listede yok. Isolate'te çalıştırma önerisi. |
| T10 | Apple Developer – PHAssetChangeRequest.deleteAssets(_:) | https://developer.apple.com/documentation/photos/phassetchangerequest/deleteassets(_:).md | "Requests that the specified assets be deleted." Son Silinenler veya onay hakkında bilgi yok. |
| T11 | Apple Developer – PHPhotoLibrary.performChanges(_:completionHandler:) | https://developer.apple.com/documentation/photos/phphotolibrary/performchanges(_:completionhandler:).md | "For each call to this method, iOS shows an alert asking the user for permission to edit the contents of the photo library. If your app needs to submit several changes at once, combine them into a single change block." |
| T12 | Flutter – Concurrency and isolates | https://docs.flutter.dev/perf/isolates | `Isolate.run`/`compute`. Fotoğraf işleme için isolate önerisi. `BackgroundIsolateBinaryMessenger`. "you can't receive unsolicited messages from the host platform". `dart:ui` ve widget kısıtı. |
| T13 | The Hacker Factor Blog – Looks Like It (Neal Krawetz) | https://www.hackerfactor.com/blog/index.php?/archives/432-Looks-Like-It.html | aHash ve pHash adımları (32×32, DCT, 8×8, DC hariç ortalama). Yazarın eşik yorumu: 0 "likely a very similar picture", 5 "a few things may be different", 10+ "probably a very different picture". Yazara göre pHash "very tolerant of minor modifications" (<%25 değişiklik). |
| T14 | The Hacker Factor Blog – Kind of Like That (Neal Krawetz) | https://www.hackerfactor.com/blog/index.php?/archives/529-Kind-of-Like-That.html | dHash adımları (9×8, komşu piksel). Yazarın 150.000 görsellik testinde: aHash "about 3.75 hours" ve "about 10x more false-positives"; pHash "over 7 hours", "No false positives, no false negatives", "every match has a score of 2 or less"; dHash "as fast as aHash", 6 eşleşmeden "(4 false positives)". Yazarın önerisi: "use the really fast dHash as a fast filter", sonra pHash. |
| T15 | Flutter – Internationalizing Flutter apps | https://docs.flutter.dev/ui/internationalization | `flutter_localizations` + `intl`, `l10n.yaml`, ARB, gen-l10n. iOS'ta dillerin Xcode Localizations'a eklenmesi. |
| T16 | pub.dev – intl | https://pub.dev/packages/intl | Sürüm 0.20.3. Platformlar. Kısıtlar: saat dilimi ve sayı ayrıştırma yok. |
| T17 | Flutter – Accessibility | https://docs.flutter.dev/ui/accessibility | "All tappable targets should be at least 48x48 pixels." Kontrast için 4.5:1 önerisi. TalkBack ve VoiceOver ile test. |
| T18 | Flutter – Accessibility testing | https://docs.flutter.dev/ui/accessibility/accessibility-testing | `androidTapTargetGuideline` 48×48, `iOSTapTargetGuideline` 44×44, `labeledTapTargetGuideline`, `textContrastGuideline` ("3:1 for larger text"). |
| T19 | Apple – App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 5.1.2(iv): "Do not use information from Contacts, Photos, or other APIs that access user data to build a contact database for your own use…". 5.1.2(vi): Photo API verisi "may not be used for marketing, advertising or use-based data mining". |