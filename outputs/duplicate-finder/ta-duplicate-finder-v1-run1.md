# Teknik Gereksinim Dokümanı: Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v1
- **Girdi:** /home/user/agent-personas/outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md

---

## 1. Özet

Bu doküman, BA analizindeki FR-01..FR-16 ve NFR-01..NFR-11 gereksinimlerinin teknik çözümünü tanımlar.

**Ürün büyüklüğüne uygun çözüm:** Sunucusu, hesabı ve ağ bağımlılığı olmayan tek bir Flutter uygulaması (V-03, NFR-01). Bütün işlem cihazda yapılır. Kalıcı durum, uygulamaya özel tek bir SQLite dosyasında tutulur.

**Ana teknik kararlar:**
- **Platform erişimi:** `photo_manager` eklentisi kullanılır: izin durumları, varlık listeleme, küçük resim, favori bilgisi, Android çöpe taşıma ve iOS silme (FR-02, FR-08, FR-09, FR-12).
- **Birebir kopya (FR-04, NFR-04a):** Önce dosya boyutuyla aday ön elemesi yapılır. Ardından akış (stream) halinde SHA-256 hesaplanır ve özetler eşleşince bayt bayt doğrulanır. Böylece yanlış pozitif oranı yapısal olarak %0 olur.
- **Benzer (FR-05):** İşletim sisteminin ürettiği küçük resimden dHash ve pHash (64 bit) hesaplanır. Adaylar BK-ağacı (BK-tree) ile bulunur ve iki özetin de eşik altında kalması şartı aranır. Gömme vektörü (embedding) modeli v1'de kullanılmaz, çünkü FR-05 kırpma veya döndürme istemiyor ve bu durumlar kapsam dışında.
- **Hassasiyet (FR-07):** Aday kenarlar en gevşek eşikte bir kez hesaplanıp saklanır. Seviye değişince yalnızca saklanan kenarlar süzülür ve birleşim-bul (union-find) ile yeniden gruplanır. Yeniden tarama gerekmez.
- **Artımlı ve devam ettirilebilir tarama (FR-15, FR-16):** Varlık kimliği ve değişiklik zamanına göre fark çıkarılır. Her varlığın işlenme durumu ve parti (batch) sonu kontrol noktaları veritabanında tutulur.
- **Arayüz yanıt verebilirliği (NFR-06):** Ağır hesaplamalar uzun ömürlü işçi izolatlarında (worker isolate) yapılır.
- **Silme (FR-12, NFR-07):** Android'de yalnızca `moveToTrash` (sistem çöpü ve sistem onayı) kullanılır. iOS'ta silme, tek bir değişiklik bloğuyla yapılır ve sistem uyarısı gösterilir. Onay diyaloğunu atlatan `MANAGE_MEDIA` izni istenmez.

**Kritik açık konular:** Android'de en düşük sürüm önerisi API 30 (Android 11) ve S-03 ile teyit edilmeli. Çöpteki fotoğraflar yer kaplamaya devam ettiği için FR-13'teki "kazanılan alan" ifadesi yanıltıcı olabilir. US-06 kabul ölçütü, gruplama yönteminden bağımsız olarak matematiksel olarak garanti edilemiyor. Ayrıntılar Bölüm 12'de.

---

## 2. Mimari genel bakış

Tek süreçli, katmanlı bir mobil uygulama. Ağ katmanı yok (NFR-01).

```
┌──────────────────────────── Flutter uygulaması (ana izolat) ───────────────────────────┐
│  Sunum katmanı (UI)                                                                   │
│   Bilgilendirme/İzin · Tarama/İlerleme · Sonuç sekmeleri (Birebir|Benzer) ·           │
│   Grup detayı/Karşılaştırma · Onay özeti · Silme özeti · Ayarlar                      │
│                 │ (durum akışları)                                                    │
│  Uygulama servisleri                                                                  │
│   PermissionService · ScanCoordinator · GroupingService · KeeperPolicy ·              │
│   DeletionService · SettingsService                                                   │
│        │                    │                         │                               │
│  MediaLibraryGateway   LocalStore (sqflite)     WorkerPool (2-3 uzun ömürlü izolat)   │
│  (photo_manager)       tek SQLite dosyası        HashWorker: dHash/pHash, SHA-256,    │
│        │                                         bayt doğrulama, BK-ağacı, union-find │
└────────┼──────────────────────────────────────────────────────────────────────────────┘
         ▼
 iOS PhotoKit / Android MediaStore  (izin, listeleme, küçük resim, çöp/silme diyaloğu)
```

**Tarama akışı (özet):**
1. **Listeleme ve fark:** Erişilebilen tüm görüntü varlıkları sayfalı olarak listelenir. Sonuç veritabanıyla karşılaştırılır ve yeni, değişmiş ve kaybolmuş varlıklar bulunur (FR-15).
2. **Algısal özet:** İşlenmemiş her varlık için küçük resim alınır ve işçi izolatında dHash ile pHash hesaplanır (FR-05).
3. **Benzerlik kenarları:** Yeni varlıklar BK-ağacında sorgulanır. Gevşek eşiği geçen çiftler kenar olarak kaydedilir (FR-05, FR-07).
4. **Birebir doğrulama:** Aynı boyuttaki varlıklar için SHA-256 hesaplanır, özetler eşleşirse bayt bayt karşılaştırılır (FR-04).
5. **Gruplama ve öneri:** Seçili hassasiyetle gruplar oluşturulur ve saklanacak fotoğraf seçilir (FR-06, FR-09, FR-14).

Her parti sonunda ilerleme veritabanına yazılır. Kesintide 2.–4. adımlar kaldığı yerden devam eder (FR-16).

---

## 3. Teknoloji seçimleri ve gerekçeleri

| # | Seçim | Gerekçe | Bağlı gereksinimler |
|---|---|---|---|
| T-01 | **Flutter (tek kod tabanı, iOS + Android)** | Paydaş kısıtı (V-03). Tek uygulama iki platformu kapsar. | NFR-08 |
| T-02 | **Sunucu, hesap ve ağ kütüphanesi yok.** Analitik veya çökme raporlama SDK'sı eklenmez. | Fotoğraflar ve türetilmiş veriler cihazdan çıkmamalı. Tarama ağ olmadan çalışmalı. Ağ kodunun hiç olmaması, bu gereksinimin en basit ve doğrulanabilir karşılığıdır. | NFR-01 |
| T-03 | **`photo_manager` (3.12.0)** | Android ve iOS'u destekler. Sınırlı erişim için `PermissionState.limited` durumunu ve iOS 14+ için seçimi güncelleyen `PhotoManager.presentLimited()` çağrısını sunar. `AssetEntity` üzerinden `isFavorite`, `width`, `height`, `createDateTime`, `modifiedDateSecond`, `isLocallyAvailable`, `thumbnailDataWithSize(..., cancelToken)` ve `originFile` gelir. Android 11+ için `PhotoManager.editor.android.moveToTrash` vardır. Sayfalı listeleme `getAssetListPaged` ile yapılır. Değişiklik bildirimi alınabilir. ([pub.dev photo_manager](https://pub.dev/packages/photo_manager), [AssetEntity](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html), [AssetPathEntity](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetPathEntity-class.html), [PermissionState](https://pub.dev/documentation/photo_manager/latest/photo_manager/PermissionState.html)) Tek eklentiyle tüm platform ihtiyaçları karşılanır, ayrı yerel (native) kod yazma ihtiyacı en aza iner. | FR-01, FR-02, FR-03, FR-08, FR-09, FR-12, FR-15 |
| T-04 | **`photo_manager_image_provider`** (yalnızca küçük resim gösterimi) | `photo_manager` sayfası, varlıkları göstermek için `AssetEntityImage` / `AssetEntityImageProvider` kullanılmasını öneriyor ([pub.dev photo_manager](https://pub.dev/packages/photo_manager)). Küçük resimler diske kopyalanmadan gösterilir. | FR-08, FR-10, NFR-06 |
| T-05 | **`sqflite` (2.4.6)**, tek SQLite dosyası | Android ve iOS'ta çalışır. SQL işlemleri arka plan iş parçacığında yürür. İşlem (transaction) ve toplu işlem (batch) desteği vardır. Bu, parti sonu kontrol noktalarını atomik yazmak için gerekir ([pub.dev sqflite](https://pub.dev/packages/sqflite)). İlişkisel sorgular (kenarları eşiğe göre süzme, `GROUP BY` ile birebir gruplar) için uygundur. Daha ağır bir ORM gerekmez. | FR-07, FR-14, FR-15, FR-16, NFR-05 |
| T-06 | **`crypto` (dart.dev, 3.0.8)**: SHA-256 | Doğrulanmış yayıncı. SHA-256 destekler. `startChunkedConversion` ile parça parça (chunked) özet hesaplanabilir ([pub.dev crypto](https://pub.dev/packages/crypto)). Dosya belleğe tamamen yüklenmez. | FR-04, NFR-04, NFR-05 |
| T-07 | **`image` (4.13.1)**: küçük resmi çözme, gri tona çevirme, yeniden boyutlandırma | Saf Dart, yerel bağımlılık yok. Böylece işçi izolatında çalışabilir. JPEG ve PNG çözer, yeniden boyutlandırma yapar. **HEIC çözmez** ([pub.dev image](https://pub.dev/packages/image)). Bu nedenle özetler orijinal dosyadan değil, işletim sisteminin JPEG olarak verdiği küçük resimden hesaplanır (`thumbnailDataWithSize`, varsayılan biçim JPEG). HEIC dahil tüm biçimler bu sayede aynı yoldan geçer (V-08). | FR-05, NFR-02 |
| T-08 | **Dart izolatları: uzun ömürlü işçi havuzu** | Bir kare süresini aşan hesaplamalar arayüzü takılmaya (jank) uğratır. Bu iş yardımcı izolata taşınmalıdır. Aynı hesaplamanın tekrar tekrar yapıldığı durumlarda uzun ömürlü izolat (`Isolate.spawn`, `SendPort`/`ReceivePort`) önerilir. Arka plan izolatları `BackgroundIsolateBinaryMessenger` ile platform kanalına istek gönderebilir ([Flutter: Isolates](https://docs.flutter.dev/perf/isolates)). | NFR-06, NFR-02, FR-03 |
| T-09 | **`flutter_localizations` + `intl` + `gen-l10n` (ARB dosyaları)** | Flutter'ın önerdiği yerelleştirme yolu. `untranslated-messages-file` seçeneği çevrilmemiş mesajları bir dosyaya döker ([Flutter: Internationalization](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization)). Bu dosya NFR-11'in ölçümünde kullanılır. Şablon dil Türkçe (`app_tr.arb`) olur. | NFR-11 |
| T-10 | **Erişilebilirlik:** anlamsal etiketler ve `flutter_test` erişilebilirlik kılavuz kontrolleri | `meetsGuideline(labeledTapTargetGuideline)`, `androidTapTargetGuideline`, `iOSTapTargetGuideline` ve `textContrastGuideline` testleri hazır ([Flutter: Accessibility testing](https://docs.flutter.dev/ui/accessibility/accessibility-testing)). TalkBack ve VoiceOver ile test ve çok büyük yazı ölçeğinde kullanılabilirlik, Flutter'ın yayın kontrol listesinde yer alıyor ([Flutter: Accessibility](https://docs.flutter.dev/ui/accessibility-and-internationalization/accessibility)). | NFR-10 |
| T-11 | **Gömme vektörü (embedding) modeli v1'de kullanılmaz** | FR-05 yeniden boyutlandırma, sıkıştırma ve küçük parlaklık/renk farklarını istiyor. Kırpma, döndürme ve aynalama kapsam dışı (BA Bölüm 7, S-06). Bu farkları algısal özetler karşılıyor (Bölüm 6). Model, uygulama boyutunu büyütür ve "aynı sahne, farklı an" yanlış pozitif riskini artırır (BA 3.2). Mobil performansı da doğrulanmamış (S-08). S-06 "gerekli" yanıtı alırsa yeniden değerlendirilir. | FR-05, NFR-04 |
| T-12 | **Ek durum yönetimi (state management) paketi eklenmez** | Ekran sayısı az ve durum akışları basit (ilerleme, sonuç listeleri, işaretler). Ek paket karmaşıklık getirir, ama karşıladığı bir gereksinim yok. | FR-03, FR-06, FR-10 |

**Platform yapılandırma kararları:**

| # | Karar | Gerekçe | Bağlı gereksinimler |
|---|---|---|---|
| P-01 | Android'de **yalnızca `READ_MEDIA_IMAGES`** (Android 13+), `READ_MEDIA_VISUAL_USER_SELECTED` (Android 14+ kısmi erişim) ve gerekiyorsa eski sürümler için `READ_EXTERNAL_STORAGE` (`maxSdkVersion` ile sınırlı) istenir. `READ_MEDIA_VIDEO` istenmez. | Yalnızca `READ_MEDIA_IMAGES` istenirse seçim diyaloğunda yalnızca görüntüler seçilebilir. Kısmi erişimde yeniden seçim, izin tekrar istenerek yapılır ([Android 14 kısmi erişim](https://developer.android.com/about/versions/14/changes/partial-photo-video-access)). Video kapsam dışı (S-01). İzinleri en aza indirmek Play beyanını sadeleştirir. Birleştirilmiş manifest (merged manifest), eklentilerin ek izin eklemediğinden emin olmak için CI'da kontrol edilir. | FR-02, NFR-09 |
| P-02 | **`MANAGE_MEDIA` istenmez.** | Bu izin silme, çöpe taşıma ve favori işlemlerinde sistem onay diyaloğunu kaldırır ([Android Developers](https://developer.android.com/training/data-storage/shared/media), [pub.dev photo_manager](https://pub.dev/packages/photo_manager)). FR-12 ve NFR-07 ise her silmede kullanıcı onayını şart koşuyor. | FR-12, NFR-07 |
| P-03 | **`ACCESS_MEDIA_LOCATION` istenmez.** | Bu izin yalnızca EXIF konum verisi için gerekir ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). Hiçbir gereksinim konum verisi kullanmıyor. | NFR-01, NFR-09 |
| P-04 | Android sürüm yayın manifestinde `INTERNET` izni bildirilmez. Birleştirilmiş manifest CI'da kontrol edilir. | Ağ erişiminin teknik olarak engellenmesi, NFR-01'in kod incelemesine ek bir güvencesidir. | NFR-01 |
| P-05 | **Android en düşük SDK önerisi: 30 (Android 11).** Kesin değer S-03 ile belirlenir. | Geri alınabilir toplu çöpe taşıma (`createTrashRequest`, `moveToTrash`) Android 11+ gerektirir. Daha eski sürümlerde `moveToTrash` hata fırlatır ([pub.dev photo_manager](https://pub.dev/packages/photo_manager), [Android Developers](https://developer.android.com/training/data-storage/shared/media)). Android 10 ve altında FR-12'yi (geri alınabilir silme) karşılayan bir yol yok. | FR-12, NFR-08 |
| P-06 | **iOS en düşük sürüm önerisi: 14.** Kesin değer S-03 ile belirlenir. | Sınırlı erişimde seçimi genişletme (`presentLimited`) iOS 14+ ile mümkün ([pub.dev photo_manager](https://pub.dev/packages/photo_manager)). | FR-02, NFR-08 |

---

## 4. Bileşenler ve sorumlulukları

| Bileşen | Sorumluluk | Karşıladığı gereksinimler |
|---|---|---|
| **OnboardingScreen** | Gizlilik ve erişim gerekçesi ekranı. İzin isteği bu ekrandaki düğmeye basılmadan **hiç** tetiklenmez. Uygulamanın kod yolunda izin istemeye tek giriş noktası `PermissionService.request()` olur ve yalnızca bu ekrandan çağrılır. | FR-01, NFR-09 |
| **PermissionService** | `PermissionState` değerlerini (authorized, limited, denied/restricted) üç arayüz durumuna eşler. Limited durumda iOS'ta `presentLimited()` çağrılır, Android 14+'ta izin yeniden istenir. Denied durumda sistem ayarları açılır. İzin durumu **kalıcı olarak saklanmaz**, her ön plana dönüşte yeniden okunur. Android belgesi bunu açıkça öneriyor ([Android 14 kısmi erişim](https://developer.android.com/about/versions/14/changes/partial-photo-video-access)). | FR-02, NFR-09 |
| **MediaLibraryGateway** | `photo_manager` üzerinde ince bir soyutlama. İşlevleri: varlıkları sayfalı listeleme (`RequestType.image`), meta veri okuma (boyutlar, tarihler, favori, yerel kullanılabilirlik), küçük resim baytları (iptal belirteciyle), orijinal dosya akışı, dosya boyutu, çöpe taşıma veya silme isteği ve değişiklik bildirimi. Platform farklılıkları burada gizlenir. Testte sahte (fake) bir uygulamasıyla değiştirilebilir. | FR-02, FR-03, FR-08, FR-09, FR-12, FR-15 |
| **ScanCoordinator** | Tarama oturumunu yönetir: listeleme ve fark, algısal özet, kenar üretimi, birebir doğrulama. Partiler halinde işçi havuzuna iş dağıtır. İlerlemeyi en fazla saniyede bir yayınlar. İptal bayrağını her varlık arasında kontrol eder ve bekleyen küçük resim isteklerini `PMCancelToken` ile iptal eder. Her parti sonunda tek bir veritabanı işlemiyle kontrol noktası yazar. Uygulama arka plana geçince taramayı duraklatır ve kontrol noktası yazar. | FR-03, FR-15, FR-16, NFR-02, NFR-03, NFR-05 |
| **WorkerPool / HashWorker** | 2–3 uzun ömürlü izolat. Görevleri: JPEG küçük resmi çözme ve dHash/pHash hesaplama, SHA-256 akışı, bayt karşılaştırma, BK-ağacı kurma ve sorgulama, union-find gruplama. Ana izolata yalnızca küçük sonuç mesajları döner. | FR-04, FR-05, FR-07, NFR-02, NFR-06 |
| **LocalStore** | SQLite şeması, sürüm geçişleri (migration), parti yazımları, eşiğe göre kenar sorguları. | FR-07, FR-14, FR-15, FR-16 |
| **GroupingService** | Birebir grupları (doğrulanmış özet eşitliği) ve seçili hassasiyetteki Benzer grupları oluşturur. "Kopya değil" çiftlerini dışlar. Bir fotoğrafın iki sekmede birden görünmemesini sağlar. Grup sayısını ve kazanılabilir alanı hesaplar. Seviye başına sonucu bellekte önbellekler. | FR-04, FR-05, FR-06, FR-07, FR-14 |
| **KeeperPolicy** | Saklanacak fotoğraf önerisini verir ve diğerlerini önceden işaretler. Sıralama: çözünürlük, boyut, en eski tarih, kararlı eşitlik bozucu. Favoriler asla önceden işaretlenmez. Kural seti değiştirilebilir bir yapılandırma olarak tutulur (S-05). | FR-09 |
| **ResultsUI** (sekmeler, grup detayı, tam ekran karşılaştırma) | İki sekme, grup kartları, öğe meta verisi (çözünürlük, boyut, tarih), işaret değiştirme, yan yana veya ardışık karşılaştırma. Bir gruptaki tüm öğeler silinmeye işaretlenince uyarı gösterir. "Kopya değil" eylemi buradadır. | FR-06, FR-08, FR-10, FR-14 |
| **DeletionService** | Onay özeti (sayı ve alan), platforma göre geri alınabilir silme isteği, sonucun doğrulanması (varlıklar yeniden sorgulanır), veritabanının güncellenmesi, silme özeti ekranı verisi. Sistem diyaloğu reddedilirse hiçbir şey değişmez. Kalıcı silme API'si (`deleteWithIds`) Android'de **çağrılmaz**. | FR-11, FR-12, FR-13, NFR-07 |
| **SettingsService** | Hassasiyet seviyesi (varsayılan Normal), "Kopya değil" işaretlerini sıfırlama. | FR-07, FR-14 |
| **Localization (gen-l10n)** ve **Semantics** | Bütün metinler ARB dosyalarından gelir. Etkileşimli öğelerin hepsinde anlamsal etiket bulunur. Düzenler sistem yazı ölçeğine göre esner (sabit yükseklikli metin kapları kullanılmaz). | NFR-10, NFR-11 |

---

## 5. Veri modeli

Tek SQLite dosyası, uygulamanın özel alanında tutulur. **Görüntü veya küçük resim saklanmaz**, yalnızca kimlikler, meta veri ve özetler saklanır (NFR-01).

**`assets`**: taranan her görüntü varlığı için bir satır.

| Alan | Tür | Açıklama | İlgili gereksinim |
|---|---|---|---|
| `asset_id` | TEXT, PK | Platform varlık kimliği. | FR-15 |
| `modified_at` | INTEGER | Platformdan gelen değişiklik zamanı (`modifiedDateSecond`). Fark çıkarmada kullanılır. | FR-15 |
| `created_at` | INTEGER | Çekim tarihi. | FR-08, FR-09 |
| `width`, `height` | INTEGER | Çözünürlük. | FR-08, FR-09 |
| `file_size` | INTEGER, NULL | Bayt cinsinden dosya boyutu. | FR-04 (ön eleme), FR-06, FR-08, FR-09 |
| `mime_type` | TEXT | Bilgi ve hata ayıklama amaçlı. | V-08 |
| `is_favorite` | INTEGER (0/1) | Her listelemede yenilenir. | FR-09 |
| `is_local` | INTEGER (0/1) | Cihazda yerel olarak var mı? | S-02 |
| `dhash`, `phash` | 8 baytlık değer, NULL | 64 bitlik algısal özetler. | FR-05 |
| `sha256` | BLOB, NULL | Yalnızca aynı boyutlu adaylar için hesaplanır. | FR-04 |
| `exact_group_key` | TEXT, NULL | Bayt doğrulamasından geçmiş birebir grubun anahtarı. | FR-04, FR-06 |
| `phash_state`, `exact_state` | INTEGER | 0 = bekliyor, 1 = tamamlandı, 2 = atlandı veya hata. | FR-15, FR-16 |
| `last_seen_scan_id` | INTEGER | Son listelemede görüldüğü tarama. | FR-15 |

İndeksler: `file_size`, `sha256`, `exact_group_key`, `last_seen_scan_id`.

**`similar_edges`**: Gevşek eşiği geçen her benzer çift için bir satır.

| Alan | Tür | Açıklama |
|---|---|---|
| `a_id`, `b_id` | TEXT (`a_id < b_id`), birleşik PK | Çiftin iki varlığı. |
| `d_dist`, `p_dist` | INTEGER (0–64) | dHash ve pHash Hamming mesafeleri. |

İndeks: `(d_dist, p_dist)`. Kenarlar, varlık silinince birlikte silinir. Bağlı gereksinimler: FR-05, FR-07.

**`not_duplicate_pairs`**: "Kopya değil" denen grubun **tüm çiftleri** (`a_id`, `b_id`, `created_at`). Bağlı gereksinim: FR-14. Ayarlardan sıfırlama tabloyu boşaltır. Anlam sorusu için bkz. Bölüm 12, G-03.

**`scan_sessions`**: `id`, `kind` (full, incremental), `status` (running, paused, completed, cancelled), `phase`, `total`, `processed`, `started_at`, `finished_at`. Bağlı gereksinimler: FR-03, FR-16. Açılışta `running` veya `paused` durumda bir oturum varsa "Taramaya devam et" seçeneği sunulur.

**`settings`**: anahtar-değer çiftleri (`sensitivity` = strict | normal | loose). Bağlı gereksinim: FR-07.

**Saklanmayanlar:** Grup sonuçları kalıcı olarak saklanmaz. Kenarlardan ve özet eşitliğinden her seferinde birkaç saniye içinde yeniden hesaplanır. Bu tek doğruluk kaynağı (single source of truth) sağlar ve FR-07 ile FR-14 değişikliklerinde tutarsızlık riskini kaldırır. Kullanıcının grup içi işaret değişiklikleri yalnızca oturum belleğinde tutulur. Kalıcı olması gereken bir gereksinim yok.

---

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Listeleme ve fark çıkarma (FR-15, FR-16, FR-02)
1. Erişilebilen görüntü varlıkları `getAssetListPaged` ile sayfa sayfa (ör. 500'lük) okunur ([AssetPathEntity](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetPathEntity-class.html)). Her varlık için `(asset_id, modified_at)` veritabanıyla karşılaştırılır:
   - **Yeni:** Satır eklenir. Özet durumları "bekliyor" olur.
   - **Değişmiş** (`modified_at` farklı): Özetler ve kenarlar silinir, durum "bekliyor"a döner.
   - **Aynı:** Yalnızca `is_favorite`, `is_local` ve `last_seen_scan_id` güncellenir. Yeniden analiz edilmez (FR-15 kabul ölçütü).
2. Listeleme bitince bu taramada görülmeyen varlıklar ve onların kenarları silinir. Bunun **istisnası** sınırlı erişim durumudur. Sınırlı erişimde görülmeyen varlıklar silinmez, yalnızca sonuçlarda gösterilmez (bkz. Bölüm 12, G-06).
3. **Bilinen sınır:** Android, değişiklik tespiti için `getGeneration()`'ı `DATE_MODIFIED`'dan daha güvenilir buluyor ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). `photo_manager` belgesinde bu çağrı bulunamadı. v1'de tarih tabanlı fark kullanılır. Prototipte yanlış veya eksik tespit görülürse küçük bir yerel kanal eklenir (Risk R-06).

### 6.2 Birebir kopya tespiti (FR-04, NFR-04a)
1. **Ön eleme:** Bayt düzeyinde aynı iki dosyanın boyutu da aynıdır. Yalnızca `file_size` değeri en az bir başka varlıkla paylaşılan varlıklar aday olur. Bu adım, büyük çoğunluğun dosyasını hiç okumadan eler.
2. **Özet:** Adayların orijinal dosyası işçi izolatında parça parça okunup SHA-256 ile özetlenir (`startChunkedConversion`) ([pub.dev crypto](https://pub.dev/packages/crypto)). Bellek kullanımı parça boyutuyla sınırlıdır (NFR-05).
3. **Doğrulama:** Aynı özete sahip varlıklar ilk varlığa karşı bayt bayt karşılaştırılır. Yalnızca birebir eşleşenler aynı `exact_group_key`'i alır. Bu adım yalnızca eşleşen nadir gruplarda çalışır. Özet çakışması ihtimali ne olursa olsun, "Birebir" sekmesinde yanlış pozitifin olmamasını yapısal olarak garanti eder (NFR-04a).

**Neden bu yöntem:** FR-04 "her zaman aynı gruba düşer" ve NFR-04a "%0 yanlış pozitif" istiyor. Boyut eşitliği, aynı dosya için zorunlu bir koşul olduğundan gerçek kopyayı kaçırmaz. Bayt doğrulaması da kesinliği garanti eder. Algısal özet eşitliğini ön eleme olarak kullanmak daha ucuz olurdu, ama işletim sisteminin küçük resim üretiminin aynı dosyalar için her zaman aynı sonucu verdiği doğrulanmadı. Bu nedenle FR-04 için tercih edilmedi.

### 6.3 Algısal özetler: dHash ve pHash (FR-05)
- **Girdi:** İşletim sisteminden JPEG küçük resim alınır (ör. 64×64 sınır, `thumbnailDataWithSize`) ve `image` paketiyle işçi izolatında çözülür. Orijinal dosya çözülmez. HEIC, `image` paketince desteklenmediği için de bu yol zorunludur ([pub.dev image](https://pub.dev/packages/image)).
- **dHash:** Görüntü gri tona çevrilir ve 9×8'e küçültülür. Her satırda komşu pikseller arası parlaklık artışı veya azalışı 1 bit olarak kaydedilir, toplam 64 bit.
- **pHash:** Görüntü gri tona çevrilir ve 32×32'ye küçültülür. 2B DCT uygulanır. Sol üst 8×8 katsayı bloğundaki her katsayı, bloğun medyanından büyükse bit 1 olur ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)).
- **Karşılaştırma:** Hamming mesafesi kullanılır (farklı bit sayısı, 0–64).

**Neden ikisi birlikte:** dHash hızlıdır, ama az detaylı görüntülerde yanlış pozitif üretmeye yatkındır. pHash ton ve filtre değişikliklerine daha dayanıklıdır. İkisinin de eşik altında olması şartı en yüksek kesinliği verir ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). NFR-04b'deki %95 kesinlik hedefi bu yüzden birleşik kuralı gerektiriyor. Bir algısal özet hesabının maliyeti yalnızca küçük resmi çözmek ve küçük bir DCT'dir. Varlık başına bir kez hesaplanır ve saklanır.

### 6.4 Aday çift arama: BK-ağacı (FR-05, FR-07, FR-15, NFR-02, NFR-03, NFR-05)
- Bütün `dhash` değerleri bellekte bir BK-ağacına yerleştirilir. Her yeni varlık için ağaçta `r_max` yarıçaplı sorgu yapılır. `r_max`, Gevşek seviyenin dHash eşiğidir. Bulunan her aday için pHash mesafesi hesaplanır. İki mesafe de Gevşek eşikler içindeyse kenar kaydedilir.
- BK-ağacı, ayrık bir metrik uzayda üçgen eşitsizliğini kullanarak dalları budar. Ortalama arama maliyeti yaklaşık logaritmiktir, kötü durumda doğrusala iner ([Wikipedia: BK-tree](https://en.wikipedia.org/wiki/BK-tree)). Hamming mesafesi bir metriktir: a ile c'nin farklı olduğu her bit, ya a–b ya da b–c çiftinde farklı olmak zorundadır. Bu nedenle d(a,c) ≤ d(a,b) + d(b,c) sağlanır.
- **Artımlı tarama:** Yalnızca yeni veya değişmiş varlıklar sorgulanır ve ağaca eklenir. Önceden işlenmiş çiftler yeniden hesaplanmaz (FR-15, NFR-03).
- **Bellek:** 50.000 varlık için ağaç yalnızca kimlik ve 64 bitlik özet tutar (NFR-05).
- **Neden kaba kuvvet değil:** 50.000 varlıkta tüm çiftleri karşılaştırmak yaklaşık 1,25 milyar karşılaştırma demektir. BK-ağacı bu sayıyı büyük ölçüde azaltır. Budamanın verimi `r_max` büyüdükçe düşer. Bu nedenle verim prototipte ölçülür (Risk R-03).

### 6.5 Gruplama ve hassasiyet (FR-06, FR-07, FR-14)
**Başlangıç eşikleri:** Yayınlanmış standart bir eşik yok. 0–5 arası mesafe genellikle aynı görüntü, yaklaşık 10 ise yaygın bir başlangıç değeri ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). Başlangıç değerleri aşağıdadır. Kesin değerler 6.7'deki kalibrasyonla belirlenir.

| Seviye | dHash ≤ | pHash ≤ |
|---|---|---|
| Sıkı | 4 | 4 |
| Normal (varsayılan) | 8 | 8 |
| Gevşek (`r_max`) | 12 | 12 |

**Benzer gruplar:**
1. Seviye eşiklerini sağlayan kenarlar SQL ile seçilir.
2. `not_duplicate_pairs` içindeki çiftler çıkarılır (FR-14).
3. Herhangi bir birebir gruba dahil olan varlıklara dokunan kenarlar çıkarılır, böylece bir fotoğraf iki sekmede görünmez (FR-06). Birebir kopyalar temizlenince kalan fotoğraf, bir sonraki gruplamada tarama gerekmeden Benzer değerlendirmesine girer. Bu yaklaşımın onayı için bkz. G-02.
4. Kalan kenarlar işçi izolatında union-find ile bağlı bileşenlere ayrılır. İki ve daha fazla öğeli her bileşen bir gruptur.
5. Grup anahtarı, sıralı üye kimliklerinden türetilir.

**Birebir gruplar:** `exact_group_key` ile gruplanır. İçinden "Kopya değil" çiftleri çıkarılır.

**Hassasiyet değişimi:** Seviye değişince yalnızca 1–4. adımlar tekrarlanır. Yeniden tarama ve yeniden özet hesabı gerekmez. Union-find, kenar sayısıyla neredeyse doğrusal çalışır. Bu, FR-07'deki 5 saniye hedefinin dayanağıdır. Ölçümü Bölüm 7'de.

**Neden bağlı bileşen:** Basit, deterministik ve hızlıdır. Sonuç da kenar kümesinin doğrudan bir fonksiyonudur. Bilinen zayıflığı zincirlenmedir: A~B ve B~C olunca A ile C benzemese de aynı gruba düşer. Bu, NFR-04b kesinlik testinde ölçülür. Gerekirse grup merkezine (önerilen fotoğrafa) mesafe şartı eklenir (Risk R-04).

### 6.6 Saklanacak fotoğraf önerisi (FR-09)
- Grup öğeleri şu sırayla dizilir: `width×height` azalan, `file_size` azalan, `created_at` artan, en son `asset_id` (eşitliği kararlı biçimde bozmak için). İlk öğe "Sakla" önerisidir, böylece her grupta tam olarak bir öneri olur.
- Diğer öğeler silinmeye işaretlenir. Bunun tek istisnası `is_favorite = 1` olan öğelerdir. Favoriler hiçbir koşulda önceden işaretlenmez, öneri olmasalar bile.
- Kural seti, S-05 yanıtına göre değiştirilebilmesi için tek bir sıralama fonksiyonunda toplanır.

### 6.7 Eşik kalibrasyonu (NFR-04)
NFR-04'teki etiketli test seti (en az 500 çift) masaüstünde aynı Dart kodu ve aynı küçük resim boyutuyla çalıştırılır. Normal seviye için kesinlik ≥ %95 ve duyarlılık ≥ %85 koşulunu sağlayan eşik çiftleri arasından, kesinliği en yüksek olan seçilir. Sıkı ve Gevşek bu noktanın iki yanına yerleştirilir ve Sıkı ≤ Normal ≤ Gevşek sırası korunur. Eşikler kodda tek bir yapılandırma tablosundadır. `r_max` değişirse yalnızca kenarlar yeniden üretilir, özetler yeniden hesaplanmaz.

### 6.8 Geri alınabilir silme (FR-11, FR-12, FR-13, NFR-07)
1. **Uygulama içi onay özeti:** silinecek sayı ve alan (FR-11).
2. **Android 11+:** `PhotoManager.editor.android.moveToTrash(list)` çağrılır. Altta `createTrashRequest` kullanılır. Kullanıcı sistem onay diyaloğunu görür ve çöpteki öğeler sistemin belirlediği süre sonunda kalıcı olarak silinir ([Android Developers](https://developer.android.com/training/data-storage/shared/media), [pub.dev photo_manager](https://pub.dev/packages/photo_manager)). Kalıcı silen `deleteWithIds` Android'de kullanılmaz.
3. **iOS:** Silme, tek bir `performChanges` bloğu içinde `PHAssetChangeRequest.deleteAssets` ile istenir. Apple belgesine göre her `performChanges` çağrısında Fotoğraflar kullanıcıdan izin isteyen bir uyarı gösterir. Birden çok değişiklik tek blokta birleştirilmelidir ([Apple: Requesting Changes to the Photo Library](https://developer.apple.com/documentation/photokit/requesting-changes-to-the-photo-library.md)). Bu bilgi BA'nın V-05 varsayımının onay diyaloğu kısmını doğruluyor. `photo_manager`'ın `deleteWithIds` çağrısının listeyi tek blokta gönderdiği prototipte doğrulanmalı (Risk R-02). Son Silinenler'e taşınma ve 30 günlük saklama süresi bu oturumda Apple kaynağıyla doğrulanamadı. BA'nın kaynağına (V-05) dayanır.
4. **Sonuç doğrulama:** Diyalogdan sonra ilgili kimlikler yeniden sorgulanır. Yalnızca gerçekten kaybolanlar silindi sayılır, veritabanından ve kenarlardan çıkarılır, özete yansıtılır (FR-13). Diyalog reddedilirse hiçbir kayıt değişmez ve kullanıcıya bilgi verilir (US-08).
5. **Zaman aşımı:** iOS'ta `deleteAssets` çağrısının diyalog göstermeden ve tamamlanma bildirmeden askıda kaldığı bildirimleri var. Apple bu sorunu biliyor (FB18720533) ve önerilen bir geçici çözüm yok. Bir raporda tetikleyicinin önizlemesiz 48 MP ProRAW varlıklar olduğu belirtiliyor ([Apple Developer Forums 806349](https://developer.apple.com/forums/thread/806349)). DeletionService bu nedenle bir zaman aşımı uygular (ör. 60 sn). Süre dolunca kullanıcıya "işlem tamamlanamadı" bilgisi verilir ve hiçbir şey silindi sayılmaz. Gerçek durum bir sonraki listelemede uzlaştırılır (NFR-07).

---

## 7. Performans

**Bütçe:** NFR-02 hedefi 10.000 fotoğraf için 10 dakikadır, yani varlık başına ortalama 60 ms. Bu süreye küçük resim isteği, çözme, iki özet, BK-ağacı sorgusu ve payına düşen SHA-256 dahildir. Bu rakamlar bir iş hedefidir ve mobil ölçüm kaynağıyla doğrulanmadı (V-04). Referans cihaz da henüz belli değil (S-07). Aşağıdaki tasarım bütçeyi tutturmaya yöneliktir. Kesin sonuç prototip ölçümüyle alınır.

| Önlem | Etki | Gereksinim |
|---|---|---|
| Özetler orijinal dosyadan değil, işletim sistemi küçük resminden hesaplanır. | Tam çözünürlüklü çözme maliyeti olmaz. | NFR-02, NFR-05 |
| SHA-256 yalnızca aynı boyutlu adaylarda hesaplanır, parça parça okunur. | Disk okuması büyük ölçüde azalır ve bellek sabit kalır. | NFR-02, NFR-05 |
| 2–3 işçi izolatı, partiler halinde (ör. 100 varlık) çalışır. Platform kanalı istekleri eşzamanlı ama sınırlı sayıda yapılır. | Çok çekirdekten yararlanılır. | NFR-02 |
| BK-ağacıyla aday arama, artımlı taramada yalnızca yeni varlıklar için yapılır. | Karesel karşılaştırma maliyetinden kaçınılır. 100 yeni varlıkta maliyet yaklaşık 100 sorgu olur. | NFR-02, NFR-03 |
| Ana izolatta yalnızca arayüz çalışır. İlerleme mesajları en fazla saniyede bir gönderilir. | Kaydırma ve iptal gibi etkileşimler gecikmez. | NFR-06, FR-03 |
| İptal bayrağı her varlık arasında kontrol edilir. Bekleyen küçük resim istekleri `PMCancelToken` ile iptal edilir. | İptal 2 saniye içinde etkili olur. | FR-03 |
| Hassasiyet değişince yalnızca SQL süzme ve union-find çalışır (bellekte önbellekli). | Yeniden tarama yapılmaz. Sonuç 5 saniye içinde güncellenir. | FR-07 |
| Sayfalı listeleme yapılır, görüntü baytları bellekte biriktirilmez. Parti sonunda bellek serbest bırakılır. | 50.000 varlıkta bellek kaynaklı sonlanma önlenir. | NFR-05 |
| Küçük resim ızgarası `AssetEntityImageProvider` ile tembel (lazy) yüklenir. | Sonuç ekranında kaydırma akıcı kalır. | NFR-06, FR-08 |

**Ölçüm planı:** Profil derlemesinde (profile build) referans cihazda şu ölçümler alınır:

| Ölçüm | Gereksinim |
|---|---|
| 10.000 varlıkla toplam süre ve aşama süreleri | NFR-02 |
| 100 yeni varlıkla artımlı tarama süresi | NFR-03 |
| 50.000 varlıkla 3 ardışık tarama: çökme sayısı ve tepe bellek | NFR-05 |
| Tarama sırasında kare süreleri ve iptal gecikmesi | NFR-06, FR-03 |
| Seviye değişiminde gruplama süresi | FR-07 |

Ölçüm zamanlamaları yalnızca cihazda, yerel günlükte tutulur ve dışarı gönderilmez (NFR-01). Aşama süreleri bütçeyi aşarsa ilk ayar noktaları sırasıyla işçi sayısı, küçük resim boyutu ve parti boyutudur.

---

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Veri cihazda kalır | Ağ kütüphanesi, analitik ve çökme raporlama SDK'sı yok. Android yayın manifestinde `INTERNET` izni yok (P-04). Bağımlılık listesi her sürümde ağ erişimi açısından incelenir. | NFR-01 |
| Saklanan türetilmiş veri | Yalnızca kimlikler, meta veri ve özetler saklanır. Küçük resim veya görüntü kopyası diske yazılmaz. Veritabanı uygulamanın özel alanında durur ve kaldırılınca silinir. `originFile` geçici dosya üretiyorsa SHA-256 sonrası silinir. Bu davranış prototipte doğrulanır (R-07). | NFR-01 |
| iCloud veya bulutta kalan varlıklar | `isLocallyAvailable` false olan varlıklar için orijinal veri indirilmez. S-02 yanıtlanana kadar bu varlıklar atlanır ve sayıları kullanıcıya gösterilir. Bu, uygulamanın ağ olmadan çalışma şartıyla da uyumludur. Küçük resim isteğinin ağ erişimi tetikleyip tetiklemediği prototipte ağ izlemesiyle doğrulanır (R-07, G-07). | NFR-01, S-02 |
| En az izin | Yalnızca görüntü okuma izni istenir (P-01). `MANAGE_MEDIA`, `ACCESS_MEDIA_LOCATION` ve video izinleri istenmez (P-02, P-03). | NFR-09, NFR-07 |
| İzin öncesi bilgilendirme | İzin isteği yalnızca bilgilendirme ekranındaki eylemle tetiklenir. Android belgesi de sistem diyaloğunun bir arayüz öğesine basıldıktan sonra gösterilmesini öneriyor ([Android 14 kısmi erişim](https://developer.android.com/about/versions/14/changes/partial-photo-video-access)). | FR-01 |
| Onaysız değişiklik yok | Uygulama yalnızca okuma ve geri alınabilir silme API'lerini kullanır. `createWriteRequest`, favori değiştirme ve içerik düzenleme gibi yazma API'leri kullanılmaz. Silme iki onaydan geçer: uygulama içi özet ve sistem diyaloğu. Kod incelemesi kuralı: Android yolunda `deleteWithIds` çağrısı yoktur. Bu kural statik bir aramayla CI'da denetlenir. | NFR-07, FR-12 |
| Play politikası | Geniş erişim gerekçesi çekirdek işleve (tüm kütüphanenin taranması) dayanır. Kısmi erişim ve reddedilen erişim akışları FR-02 ile sağlanır. Ret riski V-07 ve S-14'te. | NFR-09 |

---

## 9. Test stratejisi

| Düzey | Kapsam | Araç / yöntem | Gereksinim |
|---|---|---|---|
| **Birim** | dHash/pHash: altın (golden) görüntülerle sabit çıktı. Aynı görüntünün JPEG kalitesi veya boyutu değişmiş sürümünde küçük mesafe. Farklı görüntülerde büyük mesafe. | `flutter_test`, saf Dart | FR-05 |
| **Birim** | SHA-256 akışı ve tek seferlik hesaplamanın aynı sonucu vermesi. Bayt doğrulamasının yapay çakışmayı (aynı özet, farklı içerik, test dublörüyle) reddetmesi. | `flutter_test` | FR-04, NFR-04a |
| **Birim / özellik** | BK-ağacı aralık sorgusunun rastgele 64 bitlik veri kümelerinde kaba kuvvetle **aynı** sonucu vermesi. | Özellik tabanlı test (rastgele tohumlu) | FR-05, FR-15 |
| **Birim** | Union-find gruplama, "Kopya değil" çiftlerinin dışlanması, birebir grup üyelerinin Benzer'den dışlanması (iki sekmede aynı fotoğraf olmaması). | `flutter_test` | FR-06, FR-07, FR-14 |
| **Birim** | KeeperPolicy: her grupta tam bir öneri, favorilerin hiçbir senaryoda önceden işaretlenmemesi, eşitlik durumlarında kararlı sonuç. | Tablo güdümlü testler | FR-09 |
| **Birim** | Fark çıkarma (yeni, değişmiş, kaybolan) ve kesilen taramanın kontrol noktasından devamı. | Sahte `MediaLibraryGateway` + bellek içi SQLite (`sqflite_common_ffi`) ([pub.dev sqflite](https://pub.dev/packages/sqflite)) | FR-15, FR-16 |
| **Widget** | İzin durumlarına göre ekranlar. İzin isteğinin bilgilendirme ekranından önce çağrılmaması. Tümü işaretlenince uyarı. Onay ve silme özetleri. | `flutter_test`, sahte servisler | FR-01, FR-02, FR-10, FR-11, FR-13 |
| **Erişilebilirlik** | Etiketli dokunma hedefleri, dokunma hedef boyutu, kontrast. Ayrıca elle VoiceOver ve TalkBack ile ana akışlar ve en büyük yazı ölçeği. | `meetsGuideline(...)` ([Flutter: Accessibility testing](https://docs.flutter.dev/ui/accessibility/accessibility-testing)) | NFR-10 |
| **Yerelleştirme** | Çevrilmemiş mesaj dosyasının boş olması, CI'da kontrol edilir. | `untranslated-messages-file` ([Flutter i18n](https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization)) | NFR-11 |
| **Entegrasyon (gerçek cihaz)** | Hazırlanmış test kütüphanesinde uçtan uca tarama, sonuçlar, silme ve özet akışı. Sistem diyaloğunun kabul ve ret yolları. Android 11+ ve iOS'ta silinenlerin çöpten veya Son Silinenler'den geri yüklenebilmesi. | `integration_test`, fiziksel cihaz ([Flutter: Integration tests](https://docs.flutter.dev/testing/integration-tests)) ve elle doğrulama | FR-03, FR-11, FR-12, FR-13, NFR-07, NFR-08 |
| **Doğruluk** | En az 500 etiketli çiftlik test setinde kesinlik ve duyarlılık. "Birebir" sekmesinde yanlış pozitif sayısı 0. | Masaüstünde aynı Dart koduyla toplu değerlendirme (6.7) | NFR-04 |
| **Performans** | Bölüm 7'deki ölçüm planı. | Profil derlemesi, referans cihaz | NFR-02, NFR-03, NFR-05, NFR-06, FR-03, FR-07 |
| **Gizlilik** | Uçak modunda uçtan uca akış. Ağ izleyiciyle (ör. cihaz proxy'si) görüntü verisi içeren istek sayısının 0 olması. Birleştirilmiş manifestte `INTERNET` izninin olmaması. | Elle test ve CI manifest kontrolü | NFR-01 |
| **Platform matrisi** | S-03 ile belirlenecek en düşük sürümler ve güncel sürümlerde izin, tarama ve silme. Android 14+ kısmi erişim ve iOS sınırlı erişim. | Cihaz ve emülatör matrisi | NFR-08, FR-02 |
| **Mağaza** | Play Console Fotoğraf ve Video İzinleri beyanının hazırlanması ve onayı. | Süreç kontrolü | NFR-09 |

---

## 10. Riskler

| ID | Risk | Etki | Olasılık | Azaltma | İlgili gereksinim |
|---|---|---|---|---|---|
| R-01 | Dosya boyutunu ucuza almanın yolu belirsiz. `AssetEntity` belgesinde dosya boyutu alanı yok ([AssetEntity](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html)). iOS'ta `originFile` ile boyut almak dosya kopyalama maliyeti doğurabilir. | Tarama süresi bütçesi aşılır. Birebir ön elemesi pahalılaşır. | Orta | Prototipte ilk iş olarak ölçülür. Gerekirse `MediaLibraryGateway.getFileSizes()` için küçük bir yerel kanal yazılır. | FR-04, FR-08, NFR-02 |
| R-02 | iOS'ta `deleteWithIds`'in listeyi tek değişiklik bloğunda göndermemesi, kullanıcıya birden çok sistem uyarısı gösterilmesi. | Toplu silme deneyimi bozulur. | Düşük-orta | Prototipte doğrulanır. Gerekirse tek blokta silen küçük bir yerel kanal yazılır. | FR-11, FR-12 |
| R-03 | Gevşek seviyede `r_max` büyüdükçe BK-ağacı budamasının zayıflaması. | İlk tarama yavaşlar. | Orta | Ölçülür. Gerekirse `r_max` düşürülür veya çoklu indeksli özet araması gibi alternatif bir indeks denenir. | NFR-02, FR-07 |
| R-04 | Bağlı bileşen gruplamasında zincirlenme (az detaylı görüntülerde dHash yanlış pozitif eğilimiyle birlikte) ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). | NFR-04b kesinlik hedefi tutmaz. | Orta | İki özet şartı uygulanır, eşikler kalibre edilir. Gerekirse grup merkezine mesafe şartı eklenir. | FR-05, NFR-04 |
| R-05 | iOS'ta silme isteğinin diyalog göstermeden askıda kalması ([Apple Developer Forums 806349](https://developer.apple.com/forums/thread/806349)). | Silme tamamlanmaz. Aynı süreçteki sonraki değişiklik istekleri de bloklanabilir. | Düşük | Zaman aşımı, kullanıcı bilgilendirmesi, silindi sayılmama ve sonraki listelemede uzlaştırma (6.8). | FR-12, FR-13, NFR-07 |
| R-06 | Tarih tabanlı değişiklik tespitinin bazı durumlarda değişikliği kaçırması. Android, `getGeneration()` kullanılmasını öneriyor ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). | Değişmiş fotoğraf yeniden analiz edilmez. | Düşük | Gerekirse Android'de nesil (generation) bilgisini okuyan yerel kanal eklenir. Kullanıcı tam yeniden taramayı yalnızca hata kurtarma amacıyla başlatabilir. Ayarlarda yeni bir özellik olarak sunulmaz. | FR-15 |
| R-07 | Küçük resim veya orijinal dosya isteğinin iCloud'dan indirme tetiklemesi ya da geçici dosya bırakması. | Gizlilik testi başarısız olur, depolama dolar. | Orta | Önce `isLocallyAvailable` kontrol edilir, yerel olmayanlar atlanır. Prototipte ağ izlemesi yapılır ve geçici dosyalar temizlenir. | NFR-01, S-02 |
| R-08 | `photo_manager`'ın Android 14+ kısmi erişimi `PermissionState.limited`'e eşlediğinin belgede açık olmaması ([PermissionState](https://pub.dev/documentation/photo_manager/latest/photo_manager/PermissionState.html)). | Sınırlı erişim mesajı Android'de gösterilmez. | Düşük-orta | Android 14 cihazında test edilir. Gerekirse PermissionService içinde platforma özel kontrol eklenir. | FR-02 |
| R-09 | Google Play beyanının reddedilmesi (V-07). | Android yayını gecikir. | Orta | Beyan, çekirdek işlev gerekçesiyle erkenden hazırlanır. Alternatif akış S-14 yanıtına bağlı. | NFR-09 |
| R-10 | Çok büyük silme listelerinde sistem isteklerinin sınırları (Android `createTrashRequest` ve iOS tek blok için belgelenmiş bir üst sınır bulunamadı). | Toplu silme başarısız olabilir. | Düşük | Entegrasyon testinde binlerce öğeyle denenir. Gerekirse parçalara bölünür, kullanıcıya kaç onay diyaloğu çıkacağı söylenir. | FR-11 |

---

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | İzinden önce gizlilik ve erişim bilgilendirmesi | OnboardingScreen, PermissionService (tek giriş noktası) | 4, 8 |
| FR-02 | Tam, sınırlı ve reddedilmiş izin durumları; seçimi genişletme; ayarlara yönlendirme | PermissionService, `PermissionState`, `presentLimited` (iOS), `READ_MEDIA_VISUAL_USER_SELECTED` (Android 14+), P-01, P-06 | 3, 4, 6.1, 10 (R-08) |
| FR-03 | Elle başlatılan tarama, ilerleme gösterimi (≥1/sn), 2 sn içinde iptal | ScanCoordinator, WorkerPool, `PMCancelToken`, ilerleme kısıtlama | 4, 7 |
| FR-04 | Bayt düzeyinde aynı fotoğrafları "Birebir" olarak gruplama | Boyut ön elemesi, SHA-256 akışı (`crypto`), bayt doğrulama | 3 (T-06), 6.2 |
| FR-05 | Boyut, sıkıştırma ve küçük ton farklarına rağmen "Benzer" gruplama | dHash ve pHash (işletim sistemi küçük resmi + `image`), BK-ağacı, birleşik eşik | 3 (T-07, T-11), 6.3, 6.4 |
| FR-06 | İki sekme, grup sayısı, kazanılabilir alan; bir fotoğraf iki sekmede olmaz | GroupingService (birebir üyelerini Benzer'den dışlama), ResultsUI | 4, 6.5 |
| FR-07 | En az 3 hassasiyet seviyesi, varsayılan Normal, 5 sn içinde güncelleme | `similar_edges` (Gevşek eşikte saklanır), SQL süzme ve union-find, `settings` | 5, 6.5, 7 |
| FR-08 | Küçük resim, çözünürlük, dosya boyutu, çekim tarihi | ResultsUI, `photo_manager_image_provider`, `assets` alanları | 3 (T-04), 4, 5 |
| FR-09 | Saklama önerisi, ön işaretleme, favori koruması | KeeperPolicy | 4, 6.6 |
| FR-10 | İşaret değiştirme, tam ekran karşılaştırma, tümü işaretlenince uyarı | ResultsUI | 4 |
| FR-11 | Tek işlemle toplu silme ve öncesinde özet | DeletionService (uygulama içi onay özeti, tek sistem isteği) | 4, 6.8 |
| FR-12 | Yalnızca onayla, geri alınabilir alana taşıyarak silme | Android `moveToTrash` (`createTrashRequest`), iOS tek `performChanges` bloğu; `deleteWithIds` Android'de yok; `MANAGE_MEDIA` yok; minSdk 30 önerisi | 3 (P-02, P-05), 6.8, 8 |
| FR-13 | Silme sonrası sayı, alan ve geri alma yeri özeti | DeletionService (sonuç doğrulama), özet ekranı | 4, 6.8 |
| FR-14 | "Kopya değil" işareti, sonraki taramalarda gizleme, sıfırlama | `not_duplicate_pairs`, GroupingService, SettingsService | 5, 6.5 |
| FR-15 | Artımlı tarama, işaretlerin korunması, kaybolanların çıkarılması | Fark çıkarma (`asset_id`, `modified_at`), varlık başına durum, artımlı BK-ağacı sorgusu | 5, 6.1, 6.4 |
| FR-16 | Kesilen taramanın kaldığı yerden devamı | `scan_sessions`, parti sonu kontrol noktaları, varlık durum alanları | 4, 5 |
| NFR-01 | Veri cihazdan çıkmaz, çevrimdışı çalışır | T-02, P-03, P-04; küçük resim saklanmaz; yerel olmayan varlıklar atlanır | 3, 5, 8, 9 |
| NFR-02 | 10.000 fotoğrafta ilk tarama ≤ 10 dk | Küçük resimden özet, boyut ön elemesi, işçi havuzu, BK-ağacı | 6.2–6.4, 7 |
| NFR-03 | 100 yeni fotoğrafta artımlı tarama ≤ 30 sn | Fark çıkarma, yalnızca yeni varlık sorgusu | 6.1, 6.4, 7 |
| NFR-04 | Birebir %0 yanlış pozitif; Benzer kesinlik ≥ %95, duyarlılık ≥ %85 | Bayt doğrulama; iki özet şartı; eşik kalibrasyonu | 6.2, 6.3, 6.7, 9 |
| NFR-05 | 50.000 fotoğrafta çökme ve bellek sonlanması olmaması | Sayfalı listeleme, parça parça SHA-256, küçük bellek izi, parti işleme | 6.2, 6.4, 7 |
| NFR-06 | Tarama sırasında etkileşimlere ≤ 200 ms yanıt | İşçi izolatları (T-08), ilerleme kısıtlama, tembel küçük resim yükleme | 3, 4, 7 |
| NFR-07 | Onaysız silme veya değişiklik yok | Çift onay, yazma API'lerinin kullanılmaması, `MANAGE_MEDIA` yok, zaman aşımı ve sonuç doğrulama, CI statik kontrolü | 3 (P-02), 6.8, 8 |
| NFR-08 | iOS ve Android desteği; Android 11+'ta toplu onaylı çöpe taşıma | T-01, P-05 (minSdk 30 önerisi), P-06 (iOS 14 önerisi), platform matrisi testi | 3, 9 |
| NFR-09 | Google Play Fotoğraf ve Video İzinleri politikasına uyum | P-01 (en az izin), FR-02 akışları, beyan hazırlığı | 3, 8, 10 (R-09) |
| NFR-10 | Ekran okuyucu etiketleri, sistem yazı boyutuna uyum | Semantics, `meetsGuideline` testleri, VoiceOver ve TalkBack ile elle test | 3 (T-10), 4, 9 |
| NFR-11 | Arayüz en az Türkçe, çevrilmemiş metin yok | gen-l10n, `app_tr.arb`, `untranslated-messages-file` kontrolü | 3 (T-09), 9 |

---

## 12. BA'ya geri sorular

- **G-01 (US-06, FR-07):** US-06, "Sıkı" seviyede grup sayısının "Normal"e göre eşit ya da daha az olmasını istiyor. Eşik sıkılaştıkça bir grup ikiye bölünebilir. Örneğin 4 öğeli bir grup iki çifte ayrılırsa grup sayısı artar. Bu durum hangi gruplama yöntemi seçilirse seçilsin mümkündür. Kabul ölçütü "Benzer gruplarındaki toplam fotoğraf sayısı eşit ya da daha az olur" şeklinde değiştirilebilir mi? Bu ölçüt her zaman sağlanır.
- **G-02 (FR-06):** Bir fotoğraf hem birebir kopyası hem de benzeri olan bir kümede yer alabilir. Tasarım, birebir gruptaki fotoğrafları "Benzer" sekmesinden dışlıyor. Birebir temizlik yapılınca kalan fotoğraf yeniden tarama gerekmeden "Benzer"e giriyor. Bu davranış kabul edilebilir mi?
- **G-03 (FR-14):** "Kopya değil" denen gruptaki fotoğraflardan birine sonradan yeni bir benzer fotoğraf eklenirse ne olmalı? Tasarım işareti grubun çiftleri düzeyinde saklıyor: işaretlenen fotoğraflar birbirleriyle bir daha gruplanmıyor, ama yeni gelen fotoğrafla yeni bir grup oluşturabiliyor. Beklenen davranış bu mu?
- **G-04 (FR-13, US-08):** Android çöpündeki ve iOS Son Silinenler'deki fotoğraflar, kalıcı olarak silinene kadar depolama alanı kaplamaya devam eder. FR-13'teki "kazanılan alan" ifadesi "çöp boşaltılınca kazanılacak alan" olarak mı gösterilmeli? Bu ifade kullanıcıyı yanıltmamak için önemli.
- **G-05 (FR-09, S-05):** Grupta bir veya daha fazla favori varsa, saklama önerisi favori fotoğrafa mı verilmeli? Şu anki kural yalnızca favorinin silinmeye işaretlenmemesini garanti ediyor, öneri yine çözünürlük ve boyuta göre seçiliyor.
- **G-06 (FR-02, FR-15):** Sınırlı erişimde veya erişim daraltıldığında görünmeyen fotoğraflar "kütüphanede artık olmayan" sayılıp sonuçlardan ve önbellekten silinmeli mi? Yoksa yalnızca gizlenip erişim genişleyince yeniden işlenmeden geri mi gelmeli? Tasarım şu an ikinci yolu izliyor.
- **G-07 (S-02, NFR-01):** Yalnızca bulutta duran fotoğraflar, S-02 yanıtlanana kadar tasarımda atlanıyor ve sayıları gösteriliyor. NFR-01'deki "görüntü verisi içeren istek sayısı 0" ölçütü, iCloud veya Google Fotoğraflar'dan **indirme** trafiğini de kapsıyor mu? Kapsıyorsa bulut fotoğrafları kalıcı olarak kapsam dışı olur.
- **G-08 (S-03, NFR-08, FR-12):** Android 10 ve altında geri alınabilir toplu silme yolu yok. En düşük Android sürümü olarak API 30 (Android 11) önerisi kabul edilebilir mi? iOS için de en düşük sürüm olarak iOS 14 önerisi kabul edilebilir mi? iOS 14, sınırlı erişimde seçimi genişletmek için gerekiyor.
- **G-09 (V-05):** Bilgi notu: Apple belgesi, uygulamanın her kütüphane değişikliği isteğinde Fotoğraflar'ın kullanıcıdan izin isteyen bir uyarı gösterdiğini doğruluyor. V-05'in bu kısmı güncellenebilir. Son Silinenler'de 30 gün saklama ise bu oturumda Apple kaynağıyla doğrulanamadı.
- **G-10 (FR-03):** Artımlı taramada ilerleme ekranındaki "toplam fotoğraf sayısı", işlenecek yeni ve değişmiş fotoğraf sayısı mı olmalı, yoksa kütüphanenin tamamı mı?
- **G-11 (NFR-02, FR-16):** NFR-02 "uygulama ön plandayken" diyor. Kullanıcı taramayı başlatıp uygulamayı arka plana alırsa taramanın duraklaması ve öne dönünce devam etmesi kabul edilebilir mi? Tasarım arka planda çalışmayı öngörmüyor.
- **G-12 (NFR-04):** En az 500 çiftlik etiketli test setini kim hazırlayacak ve etiketleyecek? Kesinlik ve duyarlılık çift düzeyinde mi, grup düzeyinde mi ölçülecek? Bu, eşik kalibrasyonunun (6.7) temelini oluşturuyor.
- **G-13 (S-11, FR-04/FR-05):** Seri çekimler (burst) ve RAW+JPEG çiftleri için karar verilene kadar bu varlıklar diğer fotoğraflarla aynı şekilde işleniyor. Ayrı ele alınacaklarsa listeleme ve gruplama kuralları değişir.
- **G-14 (S-07, FR-07):** FR-07'deki 5 saniye ve FR-03'teki süre hedefleri hangi referans cihazda ölçülecek? S-07 ile birlikte yanıtlanmalı.

---

## Ek A: Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | photo_manager, pub.dev | https://pub.dev/packages/photo_manager | Sürüm 3.12.0, platformlar; `PermissionState.limited`, `presentLimited` (iOS 14+); `deleteWithIds`'in kalıcı silmesi; `moveToTrash` (Android 11+); `MANAGE_MEDIA`'nın onay diyaloğunu kaldırması; `isLocallyAvailable`; değişiklik bildirimi; `AssetEntityImageProvider` önerisi; HEIC uyarısı. |
| 2 | AssetEntity class, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html | `isFavorite`, `modifiedDateSecond`, `createDateTime`, `width`/`height`, `originFile`/`originBytes`, `thumbnailDataWithSize` (varsayılan JPEG, `cancelToken`); dosya boyutu alanının olmaması. |
| 3 | AssetPathEntity class, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetPathEntity-class.html | `getAssetListPaged`, `getAssetListRange`, `assetCountAsync`. |
| 4 | PermissionState enum, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/PermissionState.html | İzin durumu değerleri. Android 14 kısmi erişim eşlemesinin belgede açık olmaması. |
| 5 | crypto, pub.dev | https://pub.dev/packages/crypto | SHA-256 desteği, `startChunkedConversion`, dart.dev yayıncısı, sürüm 3.0.8. |
| 6 | sqflite, pub.dev | https://pub.dev/packages/sqflite | Android/iOS desteği, işlem ve toplu işlem, arka plan iş parçacığı, izolatta kullanım, testte `sqflite_common_ffi`. |
| 7 | image, pub.dev | https://pub.dev/packages/image | Saf Dart, JPEG/PNG çözme, yeniden boyutlandırma, HEIC desteğinin olmaması. |
| 8 | Concurrency and isolates, Flutter | https://docs.flutter.dev/perf/isolates | Takılmayı önlemek için yardımcı izolat, uzun ömürlü izolatlar, `BackgroundIsolateBinaryMessenger`, izolat sınırlamaları. |
| 9 | Access media files from shared storage, Android Developers | https://developer.android.com/training/data-storage/shared/media | `createTrashRequest`/`createDeleteRequest` ve onay diyaloğu (Android 11+), `MANAGE_MEDIA`, `getVersion`/`getGeneration`'ın tarih sütunlarından güvenilir olması, `ACCESS_MEDIA_LOCATION`'ın amacı. |
| 10 | Grant partial access to photos and videos (Android 14), Android Developers | https://developer.android.com/about/versions/14/changes/partial-photo-video-access | `READ_MEDIA_VISUAL_USER_SELECTED`, yeniden seçim, izin durumunu saklamama, izni bir arayüz öğesinden sonra isteme, yalnızca görüntü izniyle seçim diyaloğu. |
| 11 | Requesting Changes to the Photo Library, Apple Developer | https://developer.apple.com/documentation/photokit/requesting-changes-to-the-photo-library.md | Her `performChanges` çağrısında kullanıcıdan izin isteyen uyarı. Değişikliklerin tek blokta birleştirilmesi. |
| 12 | PHPhotoLibrary.performChanges completionHandler not called when deleting assets, Apple Developer Forums | https://developer.apple.com/forums/thread/806349 | `deleteAssets`'in diyalog göstermeden askıda kalması, ProRAW tetikleyicisi, Apple'ın bilinen sorun kaydı (FB18720533). |
| 13 | pHash vs dHash: Perceptual Image Hashing, MojoAuth | https://mojoauth.com/security-guides/phash-vs-dhash | dHash ve pHash adımları, Hamming eşik rehberi (0–5, ~10), iki özetin birlikte kullanımıyla kesinlik, dHash'in az detaylı görüntülerdeki zayıflığı. |
| 14 | BK-tree, Wikipedia | https://en.wikipedia.org/wiki/BK-tree | BK-ağacı yapısı, üçgen eşitsizliğiyle budama, ortalama ve kötü durum karmaşıklığı. |
| 15 | Internationalizing Flutter apps, Flutter | https://docs.flutter.dev/ui/accessibility-and-internationalization/internationalization | `flutter_localizations`, `intl`, `gen-l10n`, ARB, `untranslated-messages-file`. |
| 16 | Accessibility, Flutter | https://docs.flutter.dev/ui/accessibility-and-internationalization/accessibility | TalkBack ve VoiceOver ile test, büyük yazı ölçeğinde kullanılabilirlik, yayın kontrol listesi. |
| 17 | Accessibility testing, Flutter | https://docs.flutter.dev/ui/accessibility/accessibility-testing | `meetsGuideline` ile dokunma hedefi, etiket ve kontrast kontrolleri. |
| 18 | Integration testing, Flutter | https://docs.flutter.dev/testing/integration-tests | `integration_test` paketi, fiziksel cihazda çalıştırma. |

## Ek B: Teknik varsayımlar

- **TV-01:** Sürüm numaraları (photo_manager 3.12.0, sqflite 2.4.6, crypto 3.0.8, image 4.13.1) 2026-11-22 itibarıyla pub.dev'de görülenlerdir. Uygulama başlarken güncel sürümler yeniden kontrol edilir.
- **TV-02:** Eşik değerleri (Bölüm 6.5), işçi sayısı, parti boyutu, küçük resim boyutu ve silme zaman aşımı başlangıç değerleridir. Ölçüm ve kalibrasyonla değiştirilecektir.
- **TV-03:** BA'nın kapsam dışı bıraktığı maddeler (video, kırpılmış veya döndürülmüş kopyalar, birleştirme, bulut taraması) tasarıma dahil edilmedi. S-01, S-04 ve S-06 yanıtları değişirse Bölüm 3 (T-11) ve Bölüm 6 yeniden değerlendirilir.

---

Analiz şu girdi dokümanına dayanıyor: /home/user/agent-personas/outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md

Teknik dokümanın tam metni yukarıda. Hiçbir dosyaya yazılmadı; `ta-duplicate-finder-v1-run1.md` adıyla kaydedilmesi bekleniyor.

- **Kapsam:** 16 FR ve 11 NFR'nin hepsi izlenebilirlik tablosunda.
- **Kaynaklar:** Yalnızca bu oturumda açıp okuduğum 18 sayfa kaynak olarak gösterildi. Yalnızca arama sonuçlarında başlığını gördüğüm sayfalar kaynak listesine alınmadı.