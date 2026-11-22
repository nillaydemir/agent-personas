# Teknik Gereksinim Dokümanı: Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v1
- **Girdi:** /home/user/agent-personas/outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md

---

## 1. Özet

Bu doküman, BA analizindeki 16 fonksiyonel gereksinimi (FR-01 ile FR-16) ve 11 fonksiyonel olmayan gereksinimi (NFR-01 ile NFR-11) karşılayan teknik çözümü tanımlıyor. Çözüm iOS ve Android'de tek kod tabanıyla çalışan, sunucusu olmayan bir Flutter uygulamasıdır.

**Ana kararlar:**
- **Mimari:** Tek süreçli, katmanlı, yalnızca cihaz üzerinde çalışan bir uygulama. Sunucu, hesap ve ağ bileşeni yok (NFR-01). Ağır hesaplama ana isolate dışındaki işçi isolate'lerde yapılır (NFR-06).
- **Platform erişimi:** İzin, listeleme, küçük resim, favori bilgisi ve silme için `photo_manager` eklentisi kullanılır (FR-02, FR-09, FR-12). Eklenti tek bir adaptör sınıfının arkasında tutulur.
- **Birebir kopya (FR-04):** Önce algısal özetle aday çiftler bulunur. Adaylar için orijinal dosyanın SHA-256 özeti hesaplanır ve özetler eşitse kopya kabul edilir. Böylece kütüphanenin tamamını okumaya gerek kalmaz.
- **Benzer fotoğraf (FR-05):** dHash ve pHash birlikte kullanılır. Bir çiftin eşleşmesi için iki mesafenin de eşiğin altında kalması gerekir. Aday arama Hamming metriği üzerinde bir BK-ağacıyla (BK-tree) yapılır. Gruplama, saklanacak fotoğrafı merkez alan yıldız kümeleme (leader clustering) ile yapılır. Gömme vektörü (embedding) yöntemi ilk sürümde kullanılmaz (gerekçe Bölüm 6).
- **Kalıcılık:** Tarama indeksi, aday kenarlar ve "Kopya değil" işaretleri yerel bir SQLite veritabanında (`sqflite`) tutulur. Bu yapı artımlı taramayı (FR-15), kaldığı yerden devam etmeyi (FR-16) ve yeniden tarama gerektirmeyen hassasiyet değişikliğini (FR-07) aynı mekanizmayla sağlar.
- **Silme:** Silme her zaman sistem onayıyla ve geri alınabilir alana taşınarak yapılır. Android'de `moveToTrash` (Android 11 ve üzeri), iOS'ta tek bir kütüphane değişiklik bloğu kullanılır (FR-12, NFR-07).

Sayısal performans ve doğruluk hedefleri (NFR-02, NFR-03, NFR-04) ancak prototip ölçümüyle doğrulanabilir. Bunun nedeni, BA'nın V-04 varsayımında belirttiği gibi mobil cihazlar için doğrulanmış bir ölçüm kaynağının bulunmamasıdır. Bu yüzden eşik değerleri ve iş parçacığı sayıları kalibre edilecek parametre olarak tasarlandı.

---

## 2. Mimari genel bakış

### 2.1 Mimari tarzı

Tek cihazlı ve tek kullanıcılı bir ürün olduğu için **katmanlı, monolitik bir mobil uygulama** seçildi. Sunucu, senkronizasyon ve hesap bileşeni yok. BA dokümanında bunlar kapsam dışında (Bölüm 7) ve NFR-01 ile de çelişirler.

```
┌──────────────────────────────────────────────────────────────────────┐
│ Sunum katmanı (Flutter widget'ları, ana isolate)                     │
│  Bilgilendirme · İzin · Tarama · Sonuçlar (Birebir / Benzer sekmesi) │
│  Grup detayı ve karşılaştırma · Silme onayı · Silme özeti · Ayarlar  │
└───────────────▲──────────────────────────────────────────────────────┘
                │ durum (ChangeNotifier / ValueNotifier)
┌───────────────┴──────────────────────────────────────────────────────┐
│ Uygulama servisleri (ana isolate, yalnızca orkestrasyon ve G/Ç)      │
│  PermissionService · ScanOrchestrator · GroupingService              │
│  RecommendationService · DeletionService · SettingsService           │
└──────┬───────────────────────┬───────────────────────┬───────────────┘
       │                       │ mesajlaşma            │
┌──────▼─────────────┐  ┌──────▼──────────────────┐  ┌─▼──────────────────┐
│ PhotoLibraryGateway│  │ İşçi isolate havuzu     │  │ IndexRepository    │
│ (photo_manager     │  │  HashWorker: decode,    │  │ (sqflite, SQLite)  │
│  adaptörü)         │  │  dHash, pHash, SHA-256  │  │                    │
│  iOS: PhotoKit     │  │  GroupingWorker: BK-tree│  │                    │
│  Android: MediaStore│ │  aday kenar, kümeleme   │  │                    │
└────────────────────┘  └─────────────────────────┘  └────────────────────┘
```

### 2.2 Tarama hattı (pipeline)

| Faz | İş | Çalıştığı yer | İlgili gereksinim |
|---|---|---|---|
| A. Listeleme | Kütüphanedeki fotoğraf varlıkları sayfalar halinde listelenir (kimlik, değiştirilme tarihi, en, boy, çekim tarihi, favori). Liste indeksle karşılaştırılır ve yeni, değişmiş ve silinmiş varlıklar ayrılır. | Ana isolate (eklenti çağrısı) | FR-03, FR-15, FR-16 |
| B. Algısal özet | İşlenecek her varlık için işletim sisteminden küçük bir küçük resim alınır. Küçük resim işçi isolate'te çözülür, gri tona çevrilir, ardından dHash ve pHash hesaplanır. Sonuçlar indekse toplu olarak yazılır. | Küçük resim ana isolate'te, hesap işçide | FR-04, FR-05, NFR-02 |
| C. Aday arama | Yeni özetler BK-ağacında, en gevşek eşikle sorgulanır. Bulunan çiftler dHash ile doğrulanır ve mesafeleriyle birlikte aday kenar olarak saklanır. | İşçi isolate | FR-05, FR-07, FR-15 |
| D. Birebir doğrulama | Boyutları aynı olan ve algısal mesafesi çok küçük olan aday çiftlerin orijinal dosyaları SHA-256 ile özetlenir. Bu adımda dosya boyutu da kaydedilir. | Dosya yolu ana isolate'te alınır, akışlı okuma ve özet işçide yapılır | FR-04, NFR-04(a) |
| E. Gruplama | Birebir gruplar özet eşitliğiyle kurulur. Benzer gruplar, seçili hassasiyetin eşiğiyle aday kenarlardan yıldız kümeleme ile kurulur. | İşçi isolate | FR-06, FR-07, FR-09, FR-14 |
| F. Görüntü verisi | Grup üyelerinin dosya boyutu ve meta verisi sonuç ekranı için tamamlanır. | Ana isolate ve işçi | FR-06, FR-08 |

### 2.3 Temel mimari ilkeler

1. **Ana isolate'te hesap yapılmaz.** Flutter dokümanı, karede gecikmeye (jank) yol açan büyük hesaplar için isolate kullanılmasını söyler ve örnek olarak fotoğraf işlemeyi verir ([Flutter: Concurrency and isolates](https://docs.flutter.dev/perf/isolates)). Bu ilke NFR-06 içindir.
2. **Platform eklentisi çağrıları ana isolate'te kalır.** Arka plan isolate'lerinde eklenti kullanmak `BackgroundIsolateBinaryMessenger` ile kayıt gerektirir. Ayrıca bu isolate'ler `dart:ui` işi yapamaz ([Flutter: Concurrency and isolates](https://docs.flutter.dev/perf/isolates)). Sadelik için eklenti çağrıları ana isolate'te asenkron yapılır ve işçilere yalnızca bayt dizisi ya da dosya yolu gönderilir (NFR-06, NFR-08).
3. **Tek silme yolu.** Fotoğraf silebilen tek bileşen `DeletionService`'tir (NFR-07).
4. **Tarama ön planda yapılır.** NFR-02 ön plan süresini tanımlıyor. Arka plan işi (background task) altyapısı kurulmaz. Uygulama arka plana geçince tarama duraklatılır ve FR-16 mekanizmasıyla devam ettirilir.

---

## 3. Teknoloji seçimleri ve gerekçeleri

| Alan | Seçim | Gerekçe | Değerlendirilen alternatif | Gereksinim |
|---|---|---|---|---|
| Uygulama çatısı | **Flutter (Dart)** | Paydaşın verdiği kısıt (BA V-03). | Yok. Kısıt olduğu için değerlendirilmedi. | NFR-08 |
| Fotoğraf kütüphanesi erişimi | **`photo_manager`** eklentisi ve görüntüleme için **`photo_manager_image_provider`** | Android, iOS ve macOS'u destekler. `requestPermissionExtend()` ile `PermissionState` döner (sınırlı erişim dahil). iOS 14 ve üzerinde seçimi yeniden açan `presentLimited()` ve ayarlara yönlendiren `openSetting()` vardır. Android 11 ve üzeri için `editor.android.moveToTrash` sunar. Değişiklik bildirimi, sayfalı listeleme (`getAssetListPaged`), iCloud durumunu soran `isLocallyAvailable` ve iptal için `PMCancelToken` bulunur ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). `AssetEntity`; `id`, `isFavorite`, `width`, `height`, `createDateTime`, `modifiedDateTime`, `originFile` ve `thumbnailDataWithSize` üyelerini sağlar ([AssetEntity API](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html)). | Her platform için ayrı yerel kod (PhotoKit ve MediaStore) ile method channel. Daha fazla kontrol verir ama iki platform için bakım yükü getirir. Yalnızca eklentinin yetmediği yerlerde (Risk R-04, R-06) dar kapsamlı bir yerel uzantı açık seçenek olarak tutulur. | FR-02, FR-03, FR-08, FR-09, FR-10, FR-12, FR-15 |
| Görüntü çözme ve küçültme | **`image`** paketi (saf Dart), işçi isolate içinde | Saf Dart'tır, yerel bağımlılığı yoktur. JPEG ve PNG çözer, yeniden boyutlandırma yapar ([pub.dev: image](https://pub.dev/packages/image)). Spawn edilen isolate'ler `dart:ui` kullanamadığı için işçide çözme yapabilen saf Dart bir kütüphane gerekir ([Flutter isolates](https://docs.flutter.dev/perf/isolates)). HEIC'i çözemez ([pub.dev: image](https://pub.dev/packages/image)), ancak çözülen veri orijinal dosya değil, işletim sisteminin ürettiği küçük resim olduğu için bu bir sorun değildir. | `dart:ui` ile çözme. İşçi isolate'te kullanılamaz. | FR-05, NFR-06 |
| Kriptografik özet | **`crypto`** paketi, SHA-256 | dart.dev yayıncısının paketidir. SHA-256'yı ve parça parça (chunked) girişi `startChunkedConversion` ile destekler ([pub.dev: crypto](https://pub.dev/packages/crypto)). Büyük dosyalar belleğe tümüyle alınmadan özetlenebilir (NFR-05). | MD5 veya SHA-1. Aynı pakette var, ancak burada hız kazancı birebir doğruluk garantisinden (NFR-04a) önemli değil. | FR-04, NFR-04, NFR-05 |
| Yerel kalıcılık | **`sqflite`** (SQLite) | Android, iOS ve macOS'ta çalışır. SQL işlemleri yerel tarafta arka plan iş parçacığında yürür ve arayüzü bloke etmez. Transaction ve `Batch` desteği vardır. Ana isolate'ten kullanılması önerilir ([pub.dev: sqflite](https://pub.dev/packages/sqflite)). Toplu yazma, devam ettirilebilir tarama (FR-16) ve tutarlı güncelleme için transaction gereklidir. | Anahtar-değer depoları. Çift tabanlı sorgular (FR-14) ve kenar tabloları için uygun değil. | FR-07, FR-14, FR-15, FR-16, NFR-05 |
| Eşzamanlılık | **Dart isolate'leri**, uzun ömürlü küçük bir işçi havuzu (`Isolate.spawn`, `SendPort`/`ReceivePort`) | Aynı hesap binlerce kez tekrarlandığı için her seferinde isolate başlatmanın maliyetinden kaçınılır. Flutter bu durum için uzun ömürlü işçiler önerir ([Flutter isolates](https://docs.flutter.dev/perf/isolates)). | Her iş için `Isolate.run`. Basit ama 10.000 iş için ek yük getirir. | NFR-02, NFR-06 |
| Durum yönetimi | Flutter'ın yerleşik `ChangeNotifier` / `ValueNotifier` yapıları | Ekran sayısı az ve akış doğrusal. Ek bir durum yönetimi kütüphanesi bu ürün büyüklüğü için gereksiz (kapsam kuralı). | Üçüncü taraf durum yönetimi paketleri. Gerekçesiz karmaşıklık getirir. | FR-03, FR-06, FR-07 |
| Yerelleştirme | **`flutter_localizations`** ve **gen-l10n** (ARB dosyaları) | Flutter dokümanının ana iş akışıdır. Her dil için bir ARB dosyası (ör. `app_tr.arb`) kullanılır ve `AppLocalizations` sınıfı üretilir ([Flutter: Internationalization](https://docs.flutter.dev/ui/internationalization)). Ek diller (S-10) yalnızca yeni ARB dosyası gerektirir. | Kod içi metin haritaları. Ölçeklenmez. | NFR-11 |
| Analitik, çökme raporlama, reklam SDK'sı | **Kullanılmaz** | NFR-01 cihaz dışına veri gönderilmemesini ve çevrimdışı çalışmayı istiyor. Ağ kullanan her SDK bu gereksinimi doğrulamayı zorlaştırır. | Yok (bkz. BA'ya soru Q-11, gelir modeli). | NFR-01 |

---

## 4. Bileşenler ve sorumlulukları

| Bileşen | Sorumluluk | Karşıladığı gereksinim |
|---|---|---|
| **OnboardingScreen** | İlk açılışta, izin istenmeden önce erişim gerekçesini ve "fotoğraflar cihazdan çıkmaz" bilgisini gösterir. İzin isteği yalnızca bu ekrandaki "Devam" eyleminden tetiklenir. iOS'ta sistemin otomatik sınırlı erişim uyarısı `PHPhotoLibraryPreventAutomaticLimitedAccessAlert` Info.plist anahtarıyla bastırılır ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Böylece sınırlı erişim mesajı uygulamanın kendi akışında gösterilir. | FR-01, FR-02, US-01 |
| **PermissionService** | `requestPermissionExtend()` ile izin durumunu alır ve tam, sınırlı veya reddedilmiş olarak sınıflar. Sınırlı izinde erişilebilen fotoğraf sayısını ve "seçimi genişlet" eylemini (`presentLimited()`) sağlar. Reddedilmiş izinde `openSetting()` ile ayarlara yönlendirir ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Uygulama ön plana döndüğünde izin durumunu yeniden okur. | FR-02, NFR-09, US-01 |
| **PhotoLibraryGateway** | `photo_manager` için tek adaptör. İşlevleri: sayfalı listeleme, küçük resim baytları, orijinal dosya yolu, favori bilgisi, iCloud yerellik kontrolü, silme. Test için sahte (fake) uygulaması yazılabilir. Android'de kalıcı silme API'si (`deleteWithIds`) bu adaptörde **hiç sunulmaz**. | FR-03, FR-04, FR-08, FR-12, FR-15, NFR-07, NFR-08 |
| **ScanOrchestrator** | Tarama oturumunu yönetir: Faz A ile F'yi sırayla yürütür, işleri işçi havuzuna dağıtır ve sonuçları toplu olarak indekse yazar. İlerlemeyi (işlenen/toplam, yüzde) en fazla 500 ms aralıkla yayınlar. İptal belirtecini (cancellation token) yönetir. Uygulama arka plana geçince duraklar. Açılışta yarım kalmış oturumu algılayıp "Taramaya devam et" seçeneği sunar. | FR-03, FR-15, FR-16, NFR-02, NFR-03, NFR-05, US-02, US-09 |
| **HashWorker** (işçi isolate) | Küçük resim baytlarını çözer, gri tona çevirir, dHash (9×8) ve pHash (32×32 DCT) hesaplar. Aday çiftler için orijinal dosyayı akışlı okuyarak SHA-256 ve bayt boyutunu hesaplar. | FR-04, FR-05, NFR-04, NFR-05, NFR-06 |
| **GroupingWorker** (işçi isolate) | pHash değerlerini BK-ağacında tutar ve yarıçap sorgusu yapar. dHash ile doğrulanan aday kenarları üretir. "Kopya değil" çiftlerini kenarlardan çıkarır. Birebir grupları (SHA-256 eşitliği) ve Benzer grupları (yıldız kümeleme) kurar. Hassasiyet değiştiğinde yalnızca kümelemeyi yeniden çalıştırır. | FR-04, FR-05, FR-06, FR-07, FR-14, NFR-04 |
| **RecommendationService** | Her grupta saklanacak tek fotoğrafı seçer ve diğerlerini önceden işaretler. Favorileri hiçbir zaman önceden işaretlemez. Bir grubun bütün üyeleri işaretlendiğinde uyarı üretir. Toplu silme için sayı ve alan özetini hesaplar. | FR-09, FR-10, FR-11, US-04, US-05 |
| **ResultsScreen** | "Birebir kopya" ve "Benzer" sekmelerini, her sekmenin grup sayısını ve kazanılabilir alanı gösterir. Hassasiyet seçicisi (Sıkı, Normal, Gevşek) bu ekrandadır. | FR-06, FR-07, US-03, US-06 |
| **GroupDetailScreen / CompareView** | Grubun küçük resimlerini, çözünürlüğü, dosya boyutunu ve çekim tarihini gösterir. Tam ekranda kaydırmalı (`PageView`) veya yan yana karşılaştırma ve işaret değiştirme sunar. "Kopya değil" eylemi burada yer alır. Görseller `AssetEntityImage` / `AssetEntityImageProvider` ile gösterilir ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). | FR-08, FR-10, FR-14, US-04, US-07 |
| **DeletionService** | Uygulamadaki tek silme noktasıdır. Önce uygulama içi onay özetini gösterir. Ardından Android'de `moveToTrash`, iOS'ta tek bir değişiklik bloğu içinde toplu silme çağırır. Silme sonrası varlıkların hâlâ var olup olmadığını sorgular, gerçekten silinenleri belirler ve indeksi günceller. `MANAGE_MEDIA` izni istemez. | FR-11, FR-12, FR-13, NFR-07, US-03, US-08 |
| **DeletionSummaryScreen** | Silinen sayıyı, kazanılan alanı ve geri alma yerini (iOS: Son Silinenler, Android: sistem çöpü) gösterir. Kullanıcı sistem diyaloğunu reddederse "hiçbir fotoğraf silinmedi" bilgisini verir. | FR-13, US-08 |
| **SettingsScreen / SettingsService** | Hassasiyet seviyesini saklar ve "Kopya değil işaretlerini sıfırla" eylemini sunar. | FR-07, FR-14 |
| **IndexRepository** | SQLite şemasını (Bölüm 5) yönetir. İşlemleri transaction ve batch ile toplu yapar. | FR-07, FR-14, FR-15, FR-16, NFR-05 |

---

## 5. Veri modeli

Tüm veriler uygulamanın kendi sandbox'ındaki tek bir SQLite dosyasında tutulur. **Görüntü, küçük resim ya da vektör saklanmaz.** Yalnızca 64 bitlik özetler, SHA-256 özetleri ve meta veriler saklanır (NFR-01).

### 5.1 Tablolar

**`asset_index`**: Her taranmış fotoğraf için bir satır.

| Alan | Tür | Açıklama | Gereksinim |
|---|---|---|---|
| `asset_id` | TEXT, PK | Platformun varlık kimliği (`AssetEntity.id`). | FR-15 |
| `modified_at` | INTEGER | Son değiştirilme zamanı. Değişiklik tespiti için kullanılır. | FR-15 |
| `created_at` | INTEGER | Çekim tarihi. | FR-08, FR-09 |
| `width`, `height` | INTEGER | Piksel boyutları. | FR-04, FR-08, FR-09 |
| `is_favorite` | INTEGER (0/1) | Her listelemede yenilenir. | FR-09 |
| `phash` | INTEGER (64 bit) | DCT tabanlı algısal özet. | FR-05 |
| `dhash` | INTEGER (64 bit) | Fark özeti. | FR-05 |
| `sha256` | BLOB, NULL olabilir | Yalnızca birebir adaylar için hesaplanır. | FR-04 |
| `byte_size` | INTEGER, NULL olabilir | Grup üyeleri ve adaylar için doldurulur. | FR-06, FR-08, FR-09, FR-11, FR-13 |
| `status` | TEXT | `hashed`, `skipped_unavailable` (ör. yalnızca bulutta), `error`. | FR-03, FR-16 |
| `scan_session_id` | INTEGER | Satırı en son işleyen oturum. | FR-16 |

**`similarity_edge`**: En gevşek eşiğin içinde kalan aday çiftler.

| Alan | Tür | Açıklama |
|---|---|---|
| `a_id`, `b_id` | TEXT, birleşik PK (`a_id < b_id`) | Çiftin iki üyesi. |
| `phash_dist`, `dhash_dist` | INTEGER | Hamming mesafeleri. Hassasiyet değiştiğinde yeniden hesap gerekmez (FR-07). |

**`not_duplicate_pair`**: "Kopya değil" işaretleri (FR-14).

| Alan | Tür | Açıklama |
|---|---|---|
| `a_id`, `b_id` | TEXT, birleşik PK (`a_id < b_id`) | İşaretlenen grubun her üye çifti. |
| `created_at` | INTEGER | İşaretleme zamanı. |

**`scan_session`**: Tarama oturumları (FR-03, FR-16).

| Alan | Tür | Açıklama |
|---|---|---|
| `id` | INTEGER, PK | |
| `kind` | TEXT | `full` veya `incremental`. |
| `state` | TEXT | `running`, `paused`, `completed`, `cancelled`. |
| `total`, `processed` | INTEGER | İlerleme sayaçları. |
| `started_at`, `updated_at` | INTEGER | |

**`settings`**: Anahtar-değer tablosu. Örneğin `sensitivity` = `strict` / `normal` / `loose` (FR-07).

### 5.2 Bellekte tutulan ve kalıcı olmayan veriler

- **Gruplar ve kullanıcı işaretleri** (sakla / sil): Gruplar kenar tablosundan her an yeniden üretilebildiği için saklanmaz. BA dokümanı, kullanıcının işaretlerinin oturumlar arasında korunmasını istemiyor.
- **BK-ağacı:** Açılışta `asset_index.phash` değerlerinden kurulur. 50.000 adet 64 bitlik değer bellekte küçük yer kaplar (NFR-05).

### 5.3 Yaşam döngüsü kuralları

- Listelemede kütüphanede artık bulunmayan `asset_id`'ler `asset_index`'ten ve ilgili kenarlardan silinir (FR-15). `not_duplicate_pair` satırları da temizlenir.
- `modified_at` değeri değişen varlığın özetleri ve kenarları yeniden hesaplanır (FR-15).
- Başarılı silmeden sonra silinen varlıklar indeksten çıkarılır (FR-13).
- "İşaretleri sıfırla" eylemi `not_duplicate_pair` tablosunu boşaltır (FR-14).

---

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Benzer tespiti: dHash ve pHash birlikte (FR-05, NFR-04b)

**Hesaplama** ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)):
- İşletim sisteminden 64×64 boyutunda küçük resim istenir (`thumbnailDataWithSize`). Biçim olarak kayıpsız PNG tercih edilir. Bu biçimin eklenti tarafından desteklendiği prototipte doğrulanacak.
- **dHash:** Gri ton ve 9×8 boyutuna küçültme yapılır. Her bit, komşu iki pikselin parlaklığının arttığını ya da azaldığını gösterir (64 bit).
- **pHash:** Gri ton ve 32×32 boyutuna küçültme yapılır, ardından 2B DCT uygulanır. Sol üstteki 8×8 düşük frekans bloğunun her katsayısı bloğun medyanıyla karşılaştırılır (64 bit).
- **Mesafe:** Hamming mesafesi, iki değerin XOR'undaki 1 bitlerinin sayısıdır (popcount) ([Wikipedia: Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance)).

**Eşleşme kuralı:** Bir çift ancak `phash_dist ≤ Tp` **ve** `dhash_dist ≤ Td` olduğunda eşleşmiş sayılır. İki özetin birlikte istenmesi, kaynakta en yüksek kesinlik (precision) için önerilen yöntemdir ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). NFR-04(b)'deki %95 kesinlik hedefi bu seçimin ana gerekçesidir.

**Hassasiyet seviyeleri (FR-07): başlangıç değerleri**

| Seviye | Tp (pHash) | Td (dHash) | Dayanak |
|---|---|---|---|
| Sıkı | 5 | 5 | Kaynakta 0–5 arası mesafe "genellikle aynı görüntü" olarak verilir. |
| Normal (varsayılan) | 8 | 8 | Sıkı ve Gevşek arasında bir ara değer. Kaynağı yok, kalibrasyonla belirlenecek. |
| Gevşek | 10 | 10 | Kaynakta "başlangıç eşiği olarak yaklaşık 10" önerilir. |

Kaynak, standart bir eşik olmadığını ve eşiğin kendi veriyle ayarlanması gerektiğini belirtir ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). Bu nedenle tablodaki değerler **başlangıç parametresidir**. Kesin değerler Bölüm 9.3'teki kalibrasyonla, NFR-04 hedeflerini sağlayacak şekilde belirlenir. Kalibrasyonda Sıkı ≤ Normal ≤ Gevşek sıralaması korunur. Bu sıralama, "Sıkı'da Normal'e göre eşit ya da daha az grup" koşulunu (US-06) sağlamaya yardım eder. Kesin garanti için Bölüm 6.4'teki kümeleme notuna bakın.

**Seçilmeyen yöntemler:**

| Yöntem | Seçilmeme gerekçesi | Gereksinim |
|---|---|---|
| aHash | Üç algısal özet arasında en az dayanıklı olarak tanımlanır ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). | NFR-04 |
| Yalnızca dHash | Az detaylı görüntülerde yanlış pozitif sık görülür ([MojoAuth](https://mojoauth.com/security-guides/phash-vs-dhash)). %95 kesinlik hedefi için tek başına riskli. | NFR-04 |
| Gömme vektörü (embedding) | Kırpma, döndürme ve aynalamanın yakalanması kapsam dışı (BA Bölüm 7, S-06). Anlamsal benzerlik "aynı sahne, farklı an" fotoğraflarını da yakalar ve yanlış pozitif riskini artırır (BA 3.2). Model dosyası uygulama boyutunu büyütür ve mobil hız verisi doğrulanmamış (S-08). S-06 "gerekli" olarak yanıtlanırsa yeniden değerlendirilir. | FR-05, NFR-02, NFR-04 |

### 6.2 Birebir tespiti: aday filtresi ve SHA-256 (FR-04, NFR-04a)

**Neden kriptografik özet?** İçerik bayt düzeyinde aynıysa özet de aynıdır. En küçük değişiklik ise özeti tamamen değiştirir ([Wikipedia: Perceptual hashing](https://en.wikipedia.org/wiki/Perceptual_hashing)). FR-04'ün "bayt düzeyinde aynı" tanımına doğrudan karşılık gelir ve NFR-04(a)'daki %0 yanlış pozitif hedefini sağlar.

**Neden tüm kütüphane özetlenmiyor?** Kütüphanedeki her orijinal dosyanın tamamını okumak, 10.000 fotoğraflık ilk taramada (NFR-02) en pahalı adım olur. `photo_manager` belgelerinde bayt cinsinden dosya boyutu özelliği yer almıyor ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). `AssetEntity.size` piksel boyutunu verir ([AssetEntity API](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html)). Bu yüzden "önce dosya boyutuna göre grupla" yöntemi eklentiyle doğrudan uygulanamaz.

**Seçilen yöntem: iki aşamalı doğrulama**
1. **Aday:** `width` ve `height` değerleri aynı ve `phash_dist ≤ Tx` olan çiftler. Tx başlangıçta 4'tür. Bayt düzeyinde aynı iki dosyanın küçük resimleri aynı hattan geçtiği için özetleri aynı olmalıdır. Tx > 0 seçilerek işletim sisteminin küçük resim üretimindeki olası farklara pay bırakılır.
2. **Doğrulama:** Adayların orijinal dosyaları (`originFile`) işçide parça parça okunur, SHA-256 özeti `startChunkedConversion` ile hesaplanır ([pub.dev: crypto](https://pub.dev/packages/crypto)) ve bayt boyutu kaydedilir. Özetleri eşit olan varlıklar bir "Birebir kopya" grubu oluşturur.

**Ek koruma:** iOS'ta düzenlenmiş bir fotoğrafın orijinal kaynağı, düzenlenmemiş kopyasıyla bayt düzeyinde aynı olabilir. Bu durumda iki fotoğraf ekranda farklı görünür. Küçük resim düzenlenmiş hali yansıttığı için, aday koşulundaki algısal mesafe şartı bu çiftleri büyük olasılıkla eler. Bu davranış test setinde doğrulanacak (Risk R-03, Soru Q-07).

**Doğrulama ölçütü:** "Aynı dosyanın iki kopyası her zaman aynı gruba düşer" kabul ölçütü, aday filtresinin bayt düzeyinde aynı hiçbir çifti kaçırmamasına bağlıdır. Bölüm 9.3'teki test setinde bayt düzeyinde aynı her çiftin `phash_dist` değeri ölçülür. Bir tanesi bile Tx'i aşarsa geri dönüş yöntemi uygulanır: aynı piksel boyutundaki tüm varlıklar SHA-256 ile özetlenir (Risk R-02).

### 6.3 Aday arama: Hamming metriği üzerinde BK-ağacı (FR-05, FR-15, NFR-02, NFR-03, NFR-05)

- **Sorun:** 50.000 fotoğraf için tüm çiftleri karşılaştırmak yaklaşık 1,25 milyar karşılaştırma demektir. Artımlı taramada da her yeni fotoğrafın tüm kütüphaneyle karşılaştırılması gerekir.
- **Seçim:** pHash değerleri bir BK-ağacında tutulur. BK-ağacı ayrık metrik uzaylar için tasarlanmış bir ağaçtır ve arama sırasında üçgen eşitsizliğiyle dalları budar. Ortalama ekleme ve arama süresi Θ(log n), en kötü durum Θ(n)'dir ([Wikipedia: BK-tree](https://en.wikipedia.org/wiki/BK-tree)). Hamming mesafesi sabit uzunluklu sözcükler üzerinde bir metriktir ve üçgen eşitsizliğini sağlar ([Wikipedia: Hamming distance](https://en.wikipedia.org/wiki/Hamming_distance)). Bu yüzden BK-ağacına uygundur. Kaynak doğrudan "en yakın öğe" aramasını anlatıyor. "Belirli yarıçap içindeki tüm öğeler" sorgusu, aynı budama kuralının bir uzantısı olarak uygulanır.
- **Kullanım:** Her yeni özet önce Gevşek eşik (Tp) yarıçapıyla sorgulanır, sonra ağaca eklenir. Bulunan adaylar dHash ile doğrulanır ve kenar olarak saklanır. Artımlı taramada yalnızca yeni ve değişmiş varlıklar sorgulanır. Bu, NFR-03'ün temel dayanağıdır.
- **Risk:** Yarıçap büyüdükçe budama zayıflar ve süre doğrusal davranışa yaklaşabilir ([Wikipedia: BK-tree](https://en.wikipedia.org/wiki/BK-tree)). Bu nedenle 50.000 öğeyle prototip ölçümü yapılacak (Risk R-01).

### 6.4 Gruplama: saklanacak fotoğrafı merkez alan yıldız kümeleme (FR-05, FR-06, FR-07, FR-09, NFR-04)

- **Sorun:** Bağlı bileşen (union-find) yöntemi zincirleme etkisi yaratır. A, B'ye ve B, C'ye benzer olduğunda A ile C hiç benzemese bile aynı gruba düşer. Bu, NFR-04(b)'deki kesinlik hedefini tehlikeye atar.
- **Seçim: yıldız (leader) kümeleme.**
  1. Birebir gruplar kurulur. Her birebir grup, "Benzer" kümelemesine yalnızca saklanması önerilen üyesiyle girer. Böylece bir fotoğraf iki sekmede birden görünmez (FR-06).
  2. Kalan varlıklar FR-09 öncelik sırasına göre dizilir: en yüksek çözünürlük, en büyük dosya boyutu, en eski tarih. Dosya boyutu yalnızca kenarı olan varlıklar için okunur. Ayrıca dizilim hassasiyet seviyesinden bağımsızdır.
  3. Sırayla, henüz bir gruba atanmamış her varlık bir lider olur. Seçili seviyede bu lidere kenarla bağlı olan, atanmamış ve lideriyle `not_duplicate_pair` kaydı bulunmayan varlıklar onun grubuna katılır.
- **Sonuç:** Her gruptaki her fotoğraf, saklanması önerilen fotoğrafa doğrudan benzerdir. "Silinmesi önerilen her fotoğraf saklanacak fotoğrafa benziyor" anlamı kullanıcıya açıkça sunulabilir.
- **US-06 notu:** Daha sıkı eşikte kenar kümesi daha gevşek eşikteki kenar kümesinin alt kümesidir. Ancak yıldız kümelemede grup sayısı kenar sayısıyla birebir azalmayabilir: bir lider kenarlarını kaybedince başka bir varlık lider olabilir. US-06'daki "eşit ya da daha az grup" ölçütü test setinde doğrulanır. Gerekirse grup sayısı yerine "gruplanan fotoğraf sayısı" ölçütü önerilir (Soru Q-06).

### 6.5 Hassasiyet değişikliği (FR-07, S-09)

Kenarlar mesafeleriyle birlikte en gevşek eşikte saklandığı için, seviye değiştiğinde yalnızca kenar filtresi ve 6.4'teki kümeleme işçide yeniden çalışır. Bu işlemin maliyeti kenar sayısıyla doğrusaldır ve görüntü verisi okumaz. 5 saniye hedefi 50.000 fotoğraflık kütüphanede prototipte ölçülecek.

### 6.6 Saklanacak fotoğraf önerisi (FR-09, FR-10, FR-11)

1. Grup üyeleri FR-09 sırasına göre dizilir: en yüksek `width × height`, en büyük `byte_size`, en eski `created_at`. Eşitlik kalırsa sıralama `asset_id` ile belirlenir (deterministik sonuç).
2. İlk üye "Sakla" olarak önerilir. Her grupta tam bir öneri bulunur.
3. Diğer üyelerden `is_favorite = 1` olmayanlar silinmeye önceden işaretlenir. Favoriler hiçbir koşulda önceden işaretlenmez. Bu kural `RecommendationService` içinde tek bir yerde uygulanır ve birim testle korunur.
4. Kullanıcı bir grubun tüm üyelerini işaretlerse uyarı gösterilir (FR-10).
5. Toplu kabulde (FR-11) silinecek sayı ve `byte_size` toplamı onay özetinde gösterilir.

Kriter seti S-05'e göre değişebileceği için sıralama, yapılandırılabilir bir karşılaştırıcı (comparator) listesi olarak tasarlandı.

### 6.7 Artımlı tarama ve devam ettirme (FR-15, FR-16, NFR-03)

- **Fark hesabı:** Listeleme (Faz A) kütüphanedeki `asset_id` ve `modified_at` değerlerini indeksle karşılaştırır. Sonuç üç küme olur: yeni, değişmiş ve kaybolmuş varlıklar. Yalnızca ilk iki küme Faz B ile D'ye girer.
- **Devam ettirme:** Faz B sonuçları her N varlıkta bir (başlangıç değeri 200) tek bir transaction içinde toplu olarak yazılır. Uygulama kapanırsa bir sonraki açılışta `scan_session.state = running/paused` bulunur ve "Taramaya devam et" seçeneği sunulur. Devam ettirme, artımlı taramayla aynı fark hesabını kullanır: indekste olan varlıklar yeniden işlenmez. Böylece FR-15 ve FR-16 için tek bir mekanizma yeterli olur.
- **Platform notu:** Android dokümanı, değişiklik tespitinde `getGeneration()` yönteminin `DATE_MODIFIED` gibi tarih sütunlarından daha sağlam olduğunu belirtir. Tarih sütunları sistem saati değişince ya da `setLastModified()` çağrılınca değişebilir. Ayrıca medya deposu sürümü (`getVersion()`) değişince tam eşitleme yapılması önerilir ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). `photo_manager` belgelerinde bu değerler yer almıyor. İlk sürümde `modified_at` kullanılır. Risk R-06'da tanımlanan koşul oluşursa Android'e özel dar bir yerel uzantı eklenir.

### 6.8 "Kopya değil" işaretleri (FR-14)

Grup bir bütün olarak değil, **üye çiftleri** olarak saklanır. Kümelemede bu çiftler arasındaki kenarlar yok sayılır. Gerekçe şudur: grup kimliği her taramada üye listesine göre değişebilir. Çift tabanlı kayıt ise aynı fotoğrafların aynı grupta yeniden görünmesini kalıcı olarak engeller. Gruba sonradan yeni bir fotoğraf eklenirse ne olacağı Q-05'te soruldu.

### 6.9 Tarama iptali (FR-03)

İptal belirteci her varlıktan önce kontrol edilir. Bekleyen küçük resim istekleri `PMCancelToken` (iOS/macOS) veya `PhotoManager.cancelAllRequest()` ile iptal edilir ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). İşçi başına kuyruktaki iş sayısı küçük tutulur (başlangıçta 8). Böylece iptal, sırada bekleyen işler bitmeden etkili olur. 2 saniye hedefi prototipte ölçülür. Tamamlanmış işlerin sonuçları indekste kalır ve sonraki tarama bunları yeniden işlemez.

---

## 7. Performans

### 7.1 Bütçe

| Hedef | Bütçe | Tasarımdaki karşılığı |
|---|---|---|
| NFR-02: 10.000 fotoğraf ≤ 10 dk | Varlık başına uçtan uca ortalama ≤ 60 ms (tüm işçiler birlikte). | Orijinal dosya yerine 64×64 küçük resim okunur. SHA-256 yalnızca adaylar için hesaplanır (6.2). Hesap paralel işçilerde yapılır. Yazma işlemleri toplu yapılır. |
| NFR-03: 100 yeni fotoğraf ≤ 30 sn | Listeleme, 100 küçük resim ve 100 BK-ağacı sorgusu. | Yalnızca fark kümesi işlenir (6.7). BK-ağacı aramada tüm kütüphaneyi taramaz (6.3). |
| NFR-05: 50.000 fotoğrafta çökme olmaması | Bellekte görüntü biriktirilmez. | Küçük resim baytları işlendikten sonra bırakılır. Orijinal dosyalar parça parça okunur. Listeleme sayfalıdır (`getAssetListPaged`). Kenarlar yalnızca gevşek eşik içinde saklanır. |
| NFR-06: etkileşim ≤ 200 ms | Ana isolate'te 16 ms'yi aşan senkron iş olmaz. | Çözme, özet ve kümeleme işçide yapılır. SQLite işlemleri yerel tarafta arka plan iş parçacığında yürür ([pub.dev: sqflite](https://pub.dev/packages/sqflite)). İlerleme güncellemeleri en fazla 500 ms'de bir yapılır. |
| FR-07: seviye değişikliği ≤ 5 sn | Yalnızca kümeleme yeniden çalışır. | 6.5. |
| FR-03: ilerleme ≥ 1/sn, iptal ≤ 2 sn | | 6.9 ve ScanOrchestrator. |

### 7.2 Ayarlanacak parametreler

İşçi sayısı (başlangıç: işlemci sayısı − 1, en fazla 3), küçük resim boyutu (64×64), toplu yazma boyutu (200), işçi kuyruk derinliği (8) ve eşikler (Tp, Td, Tx). Bu değerler referans cihazda (S-07) ölçülerek sabitlenir. V-04 gereği bu dokümanda hiçbir hız değeri ölçülmüş gibi sunulmaz.

### 7.3 Bulutta duran fotoğraflar

iOS'ta `isLocallyAvailable` üç durumu ayırt eder: yerelde ve önbellekte, yerelde ama henüz önbellekte değil, yalnızca iCloud'da ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). S-02 yanıtlanana kadar yalnızca iCloud'da bulunan varlıklar indirilmez ve `skipped_unavailable` olarak işaretlenir. Bu seçimin gerekçeleri NFR-01'deki çevrimdışı çalışma koşulu ve NFR-02 süresidir. Atlanan sayı tarama sonunda gösterilir (Soru Q-01).

---

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Ağ erişimi | Uygulama hiçbir ağ çağrısı yapmaz. Ağ kullanan SDK (analitik, çökme raporu, reklam) eklenmez. Android'in release manifest'inde `INTERNET` izni bulunmaz. Bu durum, derleme sonucu birleştirilmiş (merged) manifest incelenerek doğrulanır. | NFR-01 |
| Türetilmiş veri | Yalnızca özetler ve meta veri saklanır. Küçük resim ya da görüntü diske yazılmaz. iOS'ta `originFile` ile oluşabilecek geçici kopyalar SHA-256 hesabından sonra silinir. | NFR-01, NFR-05 |
| Cihaz yedeği | İndeks veritabanının cihaz/bulut yedeğine dahil edilmemesi önerilir. Bunun NFR-01 kapsamına girip girmediği Q-10'da soruldu. | NFR-01 |
| İzinler, en az yetki | Android'de yalnızca `READ_MEDIA_IMAGES` ve Android 14 kısmi erişim için `READ_MEDIA_VISUAL_USER_SELECTED` istenir. `photo_manager`, Android 14 sınırlı erişimi için bu iznin gerektiğini belirtir ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Video izni ve `MANAGE_MEDIA` istenmez. `MANAGE_MEDIA` silme onay diyaloglarını bastırır ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)), bu da NFR-07 ile çelişir. `ACCESS_MEDIA_LOCATION` istenmez, çünkü konum meta verisi kullanılmıyor. | NFR-07, NFR-09 |
| Google Play beyanı | `READ_MEDIA_IMAGES` belirli kullanım durumlarıyla sınırlıdır ve Android dokümanı onun yerine fotoğraf seçici öneriyor ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). Beyan gerekçesi "çekirdek işlev, tüm kütüphanede kopya taraması" olarak hazırlanır. Reddedilen veya sınırlı erişimde FR-02 akışı çalışır. Beyan reddedilirse izlenecek yol S-14'e bağlıdır. | NFR-09, FR-02 |
| Onaysız silme engeli | Silme yalnızca `DeletionService` üzerinden yapılır. Önce uygulama içi özet onayı (FR-11), ardından işletim sisteminin onayı gelir. Android 11 ve üzerinde `createTrashRequest()` kullanıcıya onay diyaloğu gösterir ve öğeler sistem tanımlı süre sonunda kalıcı silinir ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). `photo_manager` bunu `moveToTrash` olarak sunar. iOS'ta kütüphane değişikliği isteyen her çağrı için sistem bir izin uyarısı gösterir. Apple birden çok değişikliğin tek değişiklik bloğunda birleştirilmesini önerir ([Apple: performChanges](https://developer.apple.com/tutorials/data/documentation/photos/phphotolibrary/performchanges(_:completionhandler:).json)). Bu bilgi BA'nın V-05 varsayımındaki onay diyaloğu kısmını doğrular. Toplu silme bu yüzden tek çağrıda yapılır. | NFR-07, FR-11, FR-12 |
| Kalıcı silme yasağı | `deleteWithIds` Android'de kalıcı siler ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Bu yüzden Android'de adaptörde hiç sunulmaz. Android 10 ve altında `moveToTrash` hata fırlatır. Bu sürümlerde silme özelliği kapalı tutulur ve S-03 yanıtı beklenir (Soru Q-02). | FR-12, NFR-07, NFR-08 |
| İçerik değişikliği | Uygulama fotoğrafa yazma, düzenleme veya meta veri değiştirme API'si çağırmaz. `createWriteRequest` ve favori değiştirme kullanılmaz. | NFR-07 |

---

## 9. Test stratejisi

### 9.1 Birim testleri (saf Dart, cihaz gerektirmez)

| Konu | Test | Gereksinim |
|---|---|---|
| dHash, pHash | Sabit test görsellerinde bilinen (golden) özet değerleri. Görsel yeniden boyutlandırıldığında ve yeniden sıkıştırıldığında mesafenin eşik altında kaldığının kontrolü. | FR-05 |
| Hamming ve BK-ağacı | Rastgele 64 bitlik değerlerle BK-ağacının yarıçap sorgusu, kaba kuvvet (brute force) sonucuyla birebir karşılaştırılır (özellik tabanlı test). | FR-05, FR-15 |
| Yıldız kümeleme | Zincirleme senaryosu (A~B, B~C, A≁C): A ile C aynı gruba düşmemeli. Her fotoğraf tek bir grupta ve tek bir sekmede yer almalı. | FR-06, NFR-04 |
| Öneri kuralları | Her grupta tam bir "Sakla" önerisi. Favori hiçbir durumda önceden işaretli değil. Tüm üyeler işaretlenince uyarı. Eşitlik durumunda deterministik sonuç. | FR-09, FR-10 |
| Fark hesabı | Yeni, değişmiş ve kaybolmuş varlık kümeleri. "Kopya değil" çiftlerinin korunması ve sıfırlanması. | FR-14, FR-15 |
| Devam ettirme | Faz B'nin ortasında süreç kesildiğinde indeksteki varlıklar yeniden işlenmemeli. | FR-16 |
| SHA-256 akışı | Parça parça hesaplanan özet, tek seferde hesaplanan özetle aynı olmalı. | FR-04 |

### 9.2 Bileşen ve entegrasyon testleri

- `PhotoLibraryGateway` için sahte (fake) uygulama kullanılır. Senaryolar: tam, sınırlı ve reddedilmiş izin (FR-02); silme diyaloğunun reddi ve kısmi başarı (US-08); iptal (FR-03).
- Widget testleri: izin diyaloğu bilgilendirme ekranından önce tetiklenmemeli (FR-01). Sekme sayıları ve alan özetleri doğru olmalı (FR-06). Onay özeti doğru olmalı (FR-11). Silme özeti doğru olmalı (FR-13).
- Statik kontrol: Android kod yolunda `deleteWithIds` ve `MANAGE_MEDIA` bulunmamalı (NFR-07).

### 9.3 Doğruluk ve eşik kalibrasyonu (NFR-04, FR-07)

- BA'nın tanımladığı en az 500 etiketli çiftten oluşan set kullanılır (NFR-04). Çift türleri: birebir, sıkıştırılmış, boyutu değişmiş, ardışık çekim, farklı sahne, iOS'ta düzenlenmiş. Görseller iki platformun gerçek kütüphane hattından geçirilir.
- Aşırı uyumu (overfitting) önlemek için set **kalibrasyon** ve **değerlendirme** bölümlerine ayrılır. Eşikler kalibrasyon bölümünde seçilir, hedefler yalnızca değerlendirme bölümünde raporlanır.
- Ölçümler: "Birebir kopya" yanlış pozitif sayısı 0 olmalı. Bayt düzeyinde aynı her çift için `phash_dist ≤ Tx` kontrol edilir (6.2). "Benzer" sekmesinde Normal seviyede kesinlik ≥ %95 ve duyarlılık ≥ %85 olmalı. US-06 için seviyeler arası grup sayısı karşılaştırılır.

### 9.4 Cihaz üzerinde performans ve kararlılık testleri (S-07'deki referans cihazda)

| Test | Ölçüt | Gereksinim |
|---|---|---|
| 10.000 fotoğrafla ilk tarama | ≤ 10 dk, uygulama ön planda | NFR-02 |
| Taramadan sonra 100 fotoğraf ekleyip artımlı tarama | ≤ 30 sn | NFR-03 |
| 50.000 fotoğrafla 3 ardışık tam tarama | 0 çökme, bellek kaynaklı sonlanma yok | NFR-05 |
| Tarama sırasında kaydırma ve iptal | Yanıt ≤ 200 ms, iptal ≤ 2 sn, ilerleme ≥ 1/sn | NFR-06, FR-03 |
| 50.000 fotoğrafta hassasiyet değişikliği | ≤ 5 sn | FR-07 |
| Uçak modunda uçtan uca tarama ve silme. Bir proxy üzerinden ağ trafiği incelemesi. Birleştirilmiş manifest'te `INTERNET` izni olmadığının kontrolü. | Görüntü verisi içeren istek sayısı 0 | NFR-01 |
| Silinen her fotoğrafın sistem galerisinden geri yüklenmesi | iOS: Son Silinenler, Android 11 ve üzeri: Çöp | FR-12 |
| Platform matrisi | S-03 ile belirlenecek en düşük sürümler ve Android 14'te kısmi erişim | NFR-08, FR-02 |

### 9.5 Erişilebilirlik ve dil

- TalkBack ve VoiceOver ile tüm ana akışlar tamamlanmalı. Her kontrol, dokunulduğunda anlaşılır biçimde seslendirilmeli. Arayüz büyük yazı ölçeklerinde okunabilir ve kullanılabilir kalmalı ([Flutter: Accessibility](https://docs.flutter.dev/ui/accessibility)) (NFR-10).
- CI'da her ARB dosyasının şablondaki tüm anahtarları içerdiği kontrol edilir (NFR-11).

### 9.6 Mağaza

İç test kanalında Play Console beyanı gönderilir ve sonuç beklenir (NFR-09).

---

## 10. Riskler

| ID | Risk | Olasılık / Etki | Azaltma | Gereksinim |
|---|---|---|---|---|
| R-01 | BK-ağacı gevşek yarıçapta (10/64) iyi budayamaz ve aday arama doğrusala yaklaşır ([Wikipedia: BK-tree](https://en.wikipedia.org/wiki/BK-tree)). | Orta / Yüksek | Prototipte 50.000 öğeyle ölçüm yapılır. Gerekirse en gevşek seviyenin eşiği düşürülür (FR-07 seviyeleri korunur) veya başka bir indeksleme yöntemi araştırılır. | NFR-02, NFR-03, FR-07 |
| R-02 | Bayt düzeyinde aynı iki dosyanın küçük resim özetleri Tx'i aşar ve birebir aday filtresi kopyayı kaçırır. | Düşük / Yüksek | Test setiyle ölçülür (9.3). Gerekirse aynı piksel boyutundaki tüm varlıklar SHA-256 ile özetlenir. Bu, NFR-02 süresini artırır. | FR-04 |
| R-03 | iOS'ta düzenlenmiş fotoğraflarda `originFile` orijinal kaynağı döndürür ve görsel olarak farklı iki fotoğraf "birebir" görünür. | Orta / Yüksek | Aday koşulundaki algısal mesafe şartı ve özel test çiftleri kullanılır. Davranış Q-07 ile netleştirilir. | FR-04, NFR-04(a) |
| R-04 | Bayt cinsinden dosya boyutu yalnızca `originFile` ile alınabiliyor ([AssetEntity API](https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html)). iOS'ta bu işlem geçici kopya ve süre maliyeti getirebilir. | Orta / Orta | Boyut yalnızca grup üyeleri için okunur. Maliyet yüksek çıkarsa dar bir yerel uzantı yazılır (Android'de MediaStore'un `SIZE` sütunu var ([Android Developers](https://developer.android.com/training/data-storage/shared/media))). | FR-06, FR-08, FR-09, NFR-02 |
| R-05 | `photo_manager` belgesine göre Android 13'te görsel ya da video için hem `READ_MEDIA_IMAGES` hem `READ_MEDIA_VIDEO` gerekiyor ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Gereksiz video izni Play beyanını zorlaştırabilir. | Orta / Yüksek | Prototipte yalnızca görsel türüyle izin isteği denenir. Gerekirse izin akışı dar bir yerel uzantıyla yapılır. | NFR-09 |
| R-06 | `modified_at` tabanlı değişiklik tespiti Android'de sistem saati değişikliği gibi durumlarda yanılabilir ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). | Düşük / Orta | Kaybolan ve yeni kimlik tespiti tarihten bağımsızdır. Değişiklik kaçırma testte gözlenirse `getVersion()` / `getGeneration()` için Android'e özel uzantı eklenir. | FR-15 |
| R-07 | Toplu silmede çok sayıda öğenin tek sistem isteğine sığıp sığmayacağı doğrulanmadı. | Bilinmiyor / Orta | Prototipte büyük partiler denenir. Gerekirse istek parçalara bölünür ve kullanıcıya her parça için sistem onayı gösterilir. | FR-11, FR-12 |
| R-08 | Eşik başlangıç değerleri NFR-04 hedeflerini karşılamaz (V-04). | Orta / Yüksek | Kalibrasyon (9.3). Hedef tutmazsa BA ile hedef ya da yöntem yeniden değerlendirilir (S-06, S-08). | NFR-04, FR-07 |
| R-09 | `thumbnailDataWithSize` için kayıpsız biçim desteği ve en-boy oranı davranışı (kırpma ya da sığdırma) belgelenmemiş ([pub.dev: photo_manager](https://pub.dev/packages/photo_manager)). Platformlar arasında tutarsız özet üretebilir. | Orta / Orta | Prototipte doğrulanır. Özet hattına sabit bir "kare ölçekleme" adımı eklenir. Kalibrasyon her iki platformda ayrı ölçülür. | FR-05, NFR-04, NFR-08 |
| R-10 | Google Play beyanı reddedilir (BA V-07). | Orta / Yüksek | FR-02 akışı hazır tutulur. Alternatif ürün akışı S-14 yanıtına bağlı. | NFR-09 |

---

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | İzin öncesi bilgilendirme ekranı | OnboardingScreen. İzin isteği yalnızca bu ekrandan tetiklenir. iOS otomatik sınırlı erişim uyarısı bastırılır. | 4, 9.2 |
| FR-02 | Tam, sınırlı ve reddedilmiş izin durumlarının ele alınması | PermissionService (`requestPermissionExtend`, `presentLimited`, `openSetting`). Android 14 için `READ_MEDIA_VISUAL_USER_SELECTED`. | 3, 4, 8 |
| FR-03 | Elle başlatılan tarama, ilerleme gösterimi ve iptal | ScanOrchestrator. İlerleme en fazla 500 ms aralıkla yayınlanır. İptal belirteci ve `PMCancelToken` kullanılır. | 2.2, 4, 6.9, 7 |
| FR-04 | Bayt düzeyinde aynı fotoğrafların "Birebir kopya" olarak gruplanması | Aday filtresi (aynı boyut ve pHash ≤ Tx), ardından akışlı SHA-256 (`crypto`). | 6.2 |
| FR-05 | Görsel olarak neredeyse aynı fotoğrafların "Benzer" olarak gruplanması | dHash ve pHash birlikte. BK-ağacıyla aday arama. Yıldız kümeleme. | 6.1, 6.3, 6.4 |
| FR-06 | İki ayrı sekme, grup sayısı, kazanılabilir alan, bir fotoğrafın tek sekmede olması | ResultsScreen. Birebir grubun yalnızca saklanacak üyesi Benzer kümelemesine girer. | 4, 6.4 |
| FR-07 | 3 hassasiyet seviyesi ve yeniden tarama olmadan ≤ 5 sn güncelleme | Eşik tablosu. Kenarlar mesafeleriyle saklanır ve yalnızca kümeleme yeniden çalışır. | 5.1, 6.1, 6.5 |
| FR-08 | Küçük resim, çözünürlük, dosya boyutu ve çekim tarihi | GroupDetailScreen. `width`/`height`, `byte_size`, `created_at` alanları. | 4, 5.1 |
| FR-09 | Saklanacak fotoğraf önerisi ve favori koruması | RecommendationService. Yapılandırılabilir sıralama. Favori kuralı tek yerde uygulanır. | 6.6 |
| FR-10 | İşaret değiştirme, tam ekran karşılaştırma, tümü işaretlenince uyarı | CompareView ve RecommendationService uyarısı. | 4, 6.6 |
| FR-11 | Tek işlemle toplu kabul ve silme özeti | RecommendationService özeti. DeletionService tek çağrılık toplu silme. | 4, 6.6, 8 |
| FR-12 | Yalnızca onaylı ve geri alınabilir silme | DeletionService. Android 11 ve üzeri `moveToTrash`, iOS tek değişiklik bloğu. Android'de `deleteWithIds` yok. | 4, 8 |
| FR-13 | Silme sonrası özet | DeletionSummaryScreen. Silme sonrası varlık sorgusuyla gerçekten silinenler belirlenir. | 4, 5.3 |
| FR-14 | "Kopya değil" işareti ve sıfırlanması | `not_duplicate_pair` tablosu ve çift tabanlı kenar dışlama. SettingsService ile sıfırlama. | 5.1, 6.8 |
| FR-15 | Artımlı tarama | `asset_index` ile fark hesabı. Yalnızca yeni ve değişmiş varlıklar BK-ağacında sorgulanır. | 5, 6.3, 6.7 |
| FR-16 | Kesilen taramanın devam ettirilmesi | `scan_session` ve toplu transaction yazma. Devam ettirme artımlı fark hesabını kullanır. | 5.1, 6.7 |
| NFR-01 | Veri cihazdan çıkmaz, çevrimdışı çalışır | Ağ bileşeni ve SDK yok. Release manifest'inde `INTERNET` izni yok. Görüntü saklanmaz. iCloud'dan indirme yapılmaz. | 3, 5, 7.3, 8, 9.4 |
| NFR-02 | 10.000 fotoğrafın ilk taraması ≤ 10 dk | Küçük resim tabanlı özet, aday filtreli SHA-256, işçi havuzu, toplu yazma. | 6.2, 7 |
| NFR-03 | 100 fotoğraflık artımlı tarama ≤ 30 sn | Fark hesabı ve BK-ağacı sorgusu. | 6.3, 6.7, 7 |
| NFR-04 | Birebir %0 yanlış pozitif; Benzer'de kesinlik ≥ %95 ve duyarlılık ≥ %85 | SHA-256 doğrulaması. İki özetin birlikte eşleşmesi. Yıldız kümeleme. Ayrılmış setle kalibrasyon. | 6.1, 6.2, 6.4, 9.3 |
| NFR-05 | 50.000 fotoğrafta çökme olmaması | Sayfalı listeleme, akışlı okuma, bellekte görüntü tutulmaması, küçük BK-ağacı. | 5.2, 7.1 |
| NFR-06 | Etkileşim yanıtı ≤ 200 ms | Hesap işçi isolate'lerde. Ana isolate yalnızca asenkron G/Ç yapar. İlerleme güncellemeleri sınırlandırılır. | 2.3, 3, 7.1 |
| NFR-07 | Onaysız silme ya da değişiklik yok | Tek silme yolu (DeletionService). Çift onay. `MANAGE_MEDIA` ve yazma API'leri kullanılmaz. Statik kontrol. | 8, 9.2 |
| NFR-08 | iOS ve Android desteği, Android 11 ve üzerinde toplu çöp | Flutter ve `photo_manager`. Android 10 ve altında silme kapalı (S-03 bekleniyor). | 3, 8, 9.4 |
| NFR-09 | Google Play izin politikasına uyum | En az izin. Beyan gerekçesi. FR-02 alternatif akışı. R-05 ve R-10 azaltımları. | 8, 10 |
| NFR-10 | Ekran okuyucu etiketleri ve sistem yazı boyutu | TalkBack ve VoiceOver testleri, yazı ölçeği kontrolü. | 9.5 |
| NFR-11 | Arayüzün en az Türkçe olması | `flutter_localizations` ve gen-l10n (`app_tr.arb`). CI'da ARB eksiksizlik kontrolü. | 3, 9.5 |

---

## 12. BA'ya geri sorular

BA'nın açık soruları S-02, S-03, S-05, S-06, S-07, S-09, S-11, S-12 ve S-14 bu tasarımı doğrudan etkiliyor. Tasarım bu sorular yanıtlanmadan da çalışacak şekilde parametrik kuruldu. Aşağıdaki sorular bunlara ek olarak teknik çalışma sırasında çıktı.

- **Q-01 (S-02, NFR-01, NFR-02):** Tasarım, iCloud'da olup cihazda olmayan fotoğrafları indirmeden atlıyor ve atlanan sayıyı gösteriyor. Bu davranış kabul edilebilir mi? İndirme istenirse NFR-01'deki "internet olmadan çalışır" koşulu bu fotoğraflar için geçerliliğini yitirir.
- **Q-02 (S-03, FR-12, NFR-08):** Android 10 ve altı desteklenecekse, bu sürümlerde geri alınabilir çöp yok. Bu sürümlerde silme kapalı mı olmalı, yoksa FR-12'nin geri alınabilirlik koşulundan vazgeçilerek kalıcı silmeye mi izin verilmeli? Tasarım şu an silmeyi kapalı tutuyor.
- **Q-03 (FR-09, S-05):** Bir grupta favori fotoğraf var ama öncelik sırasında ilk değilse, "Sakla" önerisi favori fotoğrafa mı kaymalı? Ayrıca bir grupta birden çok favori varsa birden çok fotoğraf işaretsiz kalır. Bu durum "tam 1 öneri" ölçütüyle uyumlu kabul ediliyor mu?
- **Q-04 (FR-06, FR-11):** "Kazanılabilir toplam alan" nasıl tanımlanmalı? Önceden silinmeye işaretlenmiş fotoğrafların boyut toplamı mı (favoriler hariç), yoksa saklanacak fotoğraf dışındaki tüm üyelerin toplamı mı?
- **Q-05 (FR-14):** "Kopya değil" olarak işaretlenmiş bir gruba sonraki taramada yeni bir benzer fotoğraf eklenirse ne olmalı? Tasarım, işaretli çiftleri bir daha eşleştirmiyor. Yeni fotoğraf, eski üyelerden biriyle yeni bir grup olarak görünebilir. Bu davranış kabul ediliyor mu?
- **Q-06 (US-06, FR-07):** US-06'daki "Sıkı'da grup sayısı Normal'e göre eşit ya da daha az" ölçütü, yıldız kümelemede her durumda garanti edilemeyebilir (6.4). Ölçüt "gruplanan toplam fotoğraf sayısı eşit ya da daha az" olarak güncellenebilir mi?
- **Q-07 (FR-04, NFR-04a):** iOS'ta düzenlenmiş bir fotoğraf ile düzenlenmemiş kopyası nasıl ele alınmalı? Orijinal dosyaları bayt düzeyinde aynı olabilir ama ekranda farklı görünürler. "Birebir" tanımı orijinal dosyaya mı, görünen sürüme mi dayanmalı?
- **Q-08 (NFR-02):** Tarama sırasında uygulama arka plana geçerse tasarım taramayı duraklatıyor ve dönüşte devam ediyor. Arka planda devam etmesi bekleniyor mu? Bu, platformların arka plan iş kısıtları nedeniyle ek bileşen gerektirir.
- **Q-09 (NFR-04):** 500 çiftlik etiketli test setini kim hazırlayacak? Set, iki platformun gerçek fotoğraf hattından (iOS HEIC, Android JPEG) örnek içermeli mi?
- **Q-10 (NFR-01):** Türetilmiş verilerin (özetler) işletim sisteminin cihaz veya bulut yedeğine girmesi "cihaz dışına gönderme" sayılıyor mu? Tasarım, veritabanını yedek dışı tutmayı öneriyor. Bu durumda yeni cihaza geçen kullanıcı ilk taramayı yeniden yapar.
- **Q-11 (S-12, NFR-01):** Gelir modeli reklam ya da sunucu doğrulamalı abonelik olursa ağ erişimi gerekir ve NFR-01'in "ağ trafiğinde görüntü verisi yok" ölçütü yeniden tanımlanmalı. Gelir modeli netleşene kadar tasarım ağ erişimi içermiyor.
- **Q-12 (FR-13):** Android'de çöpteki öğeler sistem tanımlı bir süre sonunda kalıcı silinir ([Android Developers](https://developer.android.com/training/data-storage/shared/media)). Silme özetinde süre belirtilmeli mi? Süre sistem tarafından belirlendiği için kesin gün sayısı gösterilemeyebilir.
- **Q-13 (FR-02, FR-03):** Sınırlı erişimde ilerleme göstergesindeki "toplam" yalnızca erişilebilen fotoğrafları mı göstermeli? Tasarım bu şekilde. Kullanıcı tarama sırasında seçimi genişletirse kapsam bir sonraki artımlı taramada güncelleniyor.

---

## Ek: Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | photo_manager, pub.dev | https://pub.dev/packages/photo_manager | Platform desteği. İzin API'leri (`requestPermissionExtend`, `presentLimited`, `openSetting`, `PHPhotoLibraryPreventAutomaticLimitedAccessAlert`). Android 13 ve 14 izin notları. `deleteWithIds` ile kalıcı silme, `moveToTrash` (Android 11 ve üzeri), `MANAGE_MEDIA` etkisi. `isLocallyAvailable`, `PMCancelToken`, `cancelAllRequest`. Bayt cinsinden boyut özelliğinin belgelenmemiş olması. |
| 2 | AssetEntity class, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html | `id`, `isFavorite`, `width`, `height`, `createDateTime`, `modifiedDateTime`, `originFile`, `thumbnailDataWithSize`. `size` üyesinin piksel boyutu olması. |
| 3 | AssetPathEntity class, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetPathEntity-class.html | `assetCountAsync`, `getAssetListPaged`, `isAll`: sayfalı listeleme. |
| 4 | Concurrency and isolates, Flutter docs | https://docs.flutter.dev/perf/isolates | Gecikmeye yol açan hesaplar için isolate kullanımı. Uzun ömürlü işçiler. Arka plan isolate'te eklenti kullanımı (`BackgroundIsolateBinaryMessenger`) ve `dart:ui` kısıtı. |
| 5 | image, pub.dev | https://pub.dev/packages/image | Saf Dart, JPEG ve PNG çözme, yeniden boyutlandırma, HEIC desteğinin olmaması. |
| 6 | crypto, pub.dev | https://pub.dev/packages/crypto | SHA-256 desteği, `startChunkedConversion` ile parça parça özet, dart.dev yayıncısı. |
| 7 | sqflite, pub.dev | https://pub.dev/packages/sqflite | Platform desteği, transaction ve batch, SQL'in yerel arka plan iş parçacığında çalışması, ana isolate'ten kullanım önerisi. |
| 8 | BK-tree, Wikipedia | https://en.wikipedia.org/wiki/BK-tree | Ayrık metrik uzaylar için ağaç, üçgen eşitsizliğiyle budama, ortalama Θ(log n) ve en kötü Θ(n) karmaşıklık. |
| 9 | Hamming distance, Wikipedia | https://en.wikipedia.org/wiki/Hamming_distance | Hamming mesafesinin metrik olması (üçgen eşitsizliği). İkili dizilerde XOR ve popcount ile hesaplanması. |
| 10 | pHash vs dHash, MojoAuth | https://mojoauth.com/security-guides/phash-vs-dhash | dHash (9×8) ve pHash (32×32 DCT, 8×8 medyan) adımları. Eşik önerileri (0–5 aynı görüntü, yaklaşık 10 başlangıç, standart yok). En yüksek kesinlik için iki özetin birlikte kullanılması. dHash ve aHash zayıflıkları. |
| 11 | Perceptual hashing, Wikipedia | https://en.wikipedia.org/wiki/Perceptual_hashing | Kriptografik özetin en küçük değişiklikte tamamen değişmesi ve algısal özetten farkı. |
| 12 | Access media files from shared storage, Android Developers | https://developer.android.com/training/data-storage/shared/media | `READ_MEDIA_IMAGES` kısıtı ve fotoğraf seçici önerisi. `createTrashRequest` ve `createDeleteRequest` ile onay diyaloğu. Çöpün sistem tanımlı süre sonunda silinmesi. `getVersion()` ve `getGeneration()` yöntemlerinin tarih sütunlarından daha sağlam olması. MediaStore `SIZE` sütunu. |
| 13 | PHPhotoLibrary.performChanges(_:completionHandler:), Apple Developer | https://developer.apple.com/tutorials/data/documentation/photos/phphotolibrary/performchanges(_:completionhandler:).json | Her değişiklik çağrısında iOS'un izin uyarısı göstermesi, değişikliklerin tek blokta birleştirilmesi önerisi, asenkron çalışma. |
| 14 | Internationalizing Flutter apps, Flutter docs | https://docs.flutter.dev/ui/internationalization | `flutter_localizations`, gen-l10n, ARB dosyaları, yeni dil ekleme. |
| 15 | Accessibility, Flutter docs | https://docs.flutter.dev/ui/accessibility | TalkBack ve VoiceOver ile test, büyük yazı ölçeklerinde kullanılabilirlik kontrol maddeleri. |