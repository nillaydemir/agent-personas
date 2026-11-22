# Teknik Gereksinim Dokümanı: Fotoğraf Kütüphanesi Kopya Bulucu (Photo Library Duplicate Finder) Mobil Uygulaması

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v2
- **Girdi:** /home/user/agent-personas/outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md

---

## 1. Özet

Bu doküman, BA analizindeki 16 fonksiyonel (FR-01 – FR-16) ve 11 fonksiyonel olmayan (NFR-01 – NFR-11) gereksinimi karşılayan teknik çözümü tanımlar.

**Çözüm:** Sunucusu, hesabı ve ağ bağımlılığı olmayan tek bir Flutter uygulaması (iOS ve Android). Flutter, paydaşın verdiği kısıttır (BA V-03). Ana kararlar şunlar:

| Karar | Seçim | Dayandığı gereksinim |
|---|---|---|
| Fotoğraf erişimi, izin, silme | `photo_manager` eklentisi. Android'de `moveToTrash`, iOS'ta `deleteWithIds`. Android'de `deleteWithIds` **kullanılmaz**. | FR-02, FR-12, NFR-07, NFR-09 |
| Birebir kopya | Aday ön-filtresi (aynı en×boy ve küçük algısal mesafe) + dosya uzunluğu + akışlı SHA-256 | FR-04, NFR-04(a), NFR-02 |
| Benzer kopya | dHash **ve** pHash (64 bit), ikisi de eşiğin altındaysa eşleşme. Özetler işletim sisteminin ürettiği küçük resimden hesaplanır. | FR-05, FR-07, NFR-04(b) |
| Aday arama | dHash üzerinde BK-ağacı, en gevşek eşikle | NFR-02, NFR-05 |
| Gruplama | Deterministik açgözlü tam bağlantı (complete-link) | NFR-04(b), FR-07 |
| Kalıcı veri | Yerel SQLite (`sqflite`) | FR-14, FR-15, FR-16 |
| Eşzamanlılık | Ağır hesaplar uzun ömürlü arka plan isolate'inde | NFR-06, FR-03 |

Gömme vektörü (embedding) yöntemi ilk sürüm için **seçilmedi**. Kırpılmış ve döndürülmüş kopyalar kapsam dışında ve S-06 hâlâ açık (gerekçe Bölüm 6.1'de).

Bazı kararlar BA'nın açık sorularına (S-02, S-03, S-05, S-11) bağlı. Bunlar **"geçici"** diye işaretlendi ve Bölüm 12'de BA'ya soruldu.

---

## 2. Mimari genel bakış

Tek süreçli, katmanlı bir istemci uygulaması. Ürün küçük-orta ölçekli ve tek kullanıcılı olduğu için sunucu, bulut, durum yönetimi çatısı ya da modüler paket yapısı önerilmez (kapsam kuralları, NFR-01).

```
┌──────────────────────────── Ana isolate (UI) ────────────────────────────┐
│  Ekranlar: Bilgilendirme/İzin │ Tarama │ Sonuç sekmeleri │ Grup │ Özet  │
│        ▲ durum (ChangeNotifier)                                          │
│  ScanController ── PermissionService ── DeletionService ── ResultService │
│        │  (photo_manager platform çağrıları: listeleme, küçük resim,    │
│        │   izin, silme)                       │                          │
│        ▼                                       ▼                          │
│   Repository (sqflite, toplu işlem/transaction)                          │
└────────┬─────────────────────────────────────────────────────────────────┘
         │ SendPort / ReceivePort (küçük resim baytları, dosya yolları)
┌────────▼──────────── Arka plan isolate'i (HashWorker) ────────────────────┐
│  JPEG çözme (image) → gri ton → dHash / pHash                            │
│  Akışlı SHA-256 (crypto, dart:io)                                        │
│  BK-ağacı aday araması → aday çiftleri                                   │
└───────────────────────────────────────────────────────────────────────────┘
```

**Tarama hattı (pipeline):**
1. **Listeleme:** Erişilebilen tüm görseller ve meta verileri (id, en, boy, tarih, değişiklik bilgisi, favori, yerelde olma) okunur. Veritabanıyla karşılaştırılıp yeni, değişen ve silinen kayıtlar bulunur (FR-15).
2. **Algısal özet:** Her yeni veya değişen fotoğraf için işletim sisteminden küçük bir JPEG küçük resim istenir ve worker'da dHash ile pHash hesaplanır (FR-05).
3. **Aday arama:** BK-ağacıyla en gevşek eşikteki aday çiftleri bulunur ve iki mesafeyle birlikte kaydedilir (FR-05, FR-07).
4. **Birebir doğrulama:** Birebir aday çiftleri dosya uzunluğu ve SHA-256 ile doğrulanır (FR-04).
5. **Gruplama:** Seçili hassasiyetle bellekte yapılır. Saklanacak fotoğraf önerisi hesaplanır (FR-06, FR-07, FR-09).

Her aşama kontrol noktası (checkpoint) yazar, böylece tarama kesilse de kaldığı yerden sürer (FR-16).

---

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Teknoloji seçimleri tablosu

| Teknoloji / paket | Kullanım amacı | Gerekçe (gereksinim) | (a) İzinler | (b) En düşük platform | (c) Belgedeki bilinen kısıtlar |
|---|---|---|---|---|---|
| **Flutter / Dart** | Tek kod tabanı | Paydaş kısıtı (V-03), NFR-08 | – | Flutter'ın kendi alt sınırları bu oturumda kontrol edilmedi. Bkz. Riskler R-09. | Isolate'ler bellek paylaşmaz, yalnızca mesajla haberleşir. Spawn edilen isolate'te UI işi yapılamaz. Arka plan isolate'i platformdan "istenmemiş mesaj" alamaz. [K15] |
| **photo_manager** 3.12.0 | Listeleme, meta veri, küçük resim, orijinal dosya, izin durumları, sınırlı erişim seçicisi, silme ve çöpe taşıma, favori bilgisi | FR-02, FR-03, FR-08, FR-09, FR-12, FR-15 | Android 13+: `READ_MEDIA_IMAGES` **ve** `READ_MEDIA_VIDEO` beyanı; belgeye göre ikisi de yalnızca görsel istense bile gerekli. Android 14+: `READ_MEDIA_VISUAL_USER_SELECTED`. İsteğe bağlı: `ACCESS_MEDIA_LOCATION`, `MANAGE_MEDIA`. iOS: `NSPhotoLibraryUsageDescription`, isteğe bağlı `PHPhotoLibraryPreventAutomaticLimitedAccessAlert`. [K1] | Belgede "Android 16, iOS 9.0" yazıyor (Android için API düzeyi olduğu anlaşılıyor). [K1] | `presentLimited()` yalnızca iOS 14+ ve sınırlı durumda çalışır. `moveToTrash` yalnızca Android 11+ (API 30+), daha eskisinde hata fırlatır. `deleteWithIds` Android 30+'da çöpe taşımak yerine **kalıcı siler** [K4]; taşıma kılavuzu ise bunun tersini söylüyor [K5], yani belgeler çelişkili. iOS'ta `isLocallyAvailable` false olan varlıklar genellikle yalnızca iCloud'dadır. iOS'ta dosya boyutu sütunu yok [K3]. `isLivePhoto` yalnızca iOS/macOS'ta geçerli. [K1][K2] |
| **photo_manager_image_provider** 2.2.0 | Grup ekranında küçük resim gösterimi (`AssetEntityImage`) | FR-08, FR-10 | Belgede izin yok. | Android, iOS, macOS. photo_manager `>=2.6.0 <4.0.0`. [K6] | Belgede kısıt listelenmemiş. Son yayın 2 yıl önce, bakım riski var (R-10). [K6] |
| **image** 4.13.1 | Küçük resim JPEG'ini çözme, gri tona çevirme, yeniden boyutlandırma (dHash ve pHash girdisi) | FR-05 | Belgede izin yok. | Saf Dart, Android ve iOS dahil. [K7] | Desteklenen biçimler arasında **HEIC yok**; JPEG, PNG, WebP vb. var. Bu nedenle orijinal dosya değil, işletim sisteminin JPEG küçük resmi çözülür (Bölüm 6.2). [K7] |
| **crypto** 3.0.8 | SHA-256, büyük dosyalar için parça parça (`startChunkedConversion`) | FR-04, NFR-04(a), NFR-05 | Belgede izin yok. | Android ve iOS dahil. [K8] | "best effort" destek. Güvenlik uzmanlarınca incelenmemiş. Bu kullanımda (bütünlük karşılaştırması) etkisi düşük. [K8] |
| **sqflite** 2.4.6 | Kalıcı tarama durumu, özetler, aday çiftler, "Kopya değil" işaretleri | FR-14, FR-15, FR-16, FR-07 | Belgede izin yok. | Android minSdk 19, iOS 12. [K9] | Ana isolate tercih edilmeli. Transaction içinde `db` yerine `txn` kullanılmazsa kilitlenir. Eşzamanlı okuma yok. Batch hata alırsa tamamen geri alınır. [K9] |

**Bilinçli olarak eklenmeyenler:** Ağ, analitik ve çökme raporlama paketleri (NFR-01); ayrı izin paketi (izinleri `photo_manager` yönetiyor); durum yönetimi çatısı (Flutter'ın yerleşik `ChangeNotifier`'ı yeterli); ML modeli (Bölüm 6.1).

### 3.2 Kritik karar: kalıcı veri saklama

**Seçim ölçütleri:**
- Ö1: Kesilme sonrası tutarlı devam, transaction desteği (FR-16).
- Ö2: Artımlı tarama için id'ye göre hızlı arama (FR-15, NFR-03).
- Ö3: 50.000 fotoğraf ve aday çiftlerini bellek sorunu olmadan tutabilmek (NFR-05).
- Ö4: En sade çözüm, yani ek kod üretimi veya altyapı yok (kapsam kuralı).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **sqflite** (SQLite) | Var, transaction ve batch [K9] | İndeksli sorgu | Disk tabanlı | Doğrudan SQL, ek araç yok |
| drift (SQLite üstü) | Var | İndeksli sorgu | Disk tabanlı | Tablolardan tip güvenli kod üretir [K10]; bu ürün için gereksiz katman |
| Tek JSON dosyası | Yok; her kayıtta dosyanın tamamı yeniden yazılır | Tümü belleğe yüklenmeli | 50 bin kayıt + çiftler tek blokta | Çok sade |

**Karar: sqflite.** Ö1–Ö3'ü karşılayan seçenekler arasında Ö4'te en sade olanı. drift'in isolate'ler arası erişim desteği [K10] gerekli değil: worker sonuçları ana isolate'e mesajla gönderir, yazma ana isolate'te toplu batch'lerle yapılır. Bu, sqflite'ın "ana isolate tercih edilmeli" kısıtıyla da uyumlu [K9].

### 3.3 Kritik karar: silme yolu

**Seçim ölçütleri:**
- Ö1: Silinen fotoğraf işletim sisteminin geri alınabilir alanına gider (FR-12).
- Ö2: Her silmede kullanıcıya işletim sisteminin onayı gösterilir (NFR-07, FR-12).
- Ö3: Toplu silme desteklenir (FR-11).
- Ö4: Ek izin veya mağaza riski yok (NFR-09).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **Android: `photo_manager` `moveToTrash`** (API 30+) | Çöp; sistemin belirlediği süre sonunda kalıcı silinir (`createTrashRequest`) [K1][K4][K11] | Android 11+'da varsayılan galeri olmayan uygulama her değişiklik ve silmede onay göstermek zorunda [K11] | Liste alır | Ek izin yok |
| Android: `deleteWithIds` | **Hayır:** 2.8.0'dan beri API 30+'da kalıcı silme [K4]; taşıma kılavuzu çelişiyor [K5] | Var | Var | Ek izin yok |
| Android: `MANAGE_MEDIA` + `moveToTrash` | Çöp | **Hayır:** onay diyaloğunu kaldırır [K1][K11] | Var | Özel erişim izni ve ayarlar ekranı |
| **iOS: `deleteWithIds`** (`PHAssetChangeRequest.deleteAssets`) | Son Silinenler'e gider; bu yalnızca MacPaw SSS'sinde doğrulandı (BA V-05) | photo_manager sürüm notunda "native deletion confirmation dialogs on iOS" ifadesi geçiyor [K12]. Apple belgesinde doğrulanamadı. | Liste alır | Ek izin yok |

**Karar:** Android'de yalnızca `moveToTrash`, iOS'ta `deleteWithIds`. `MANAGE_MEDIA` beyan edilmez. Android'de `deleteWithIds` çağrısı kod incelemesinde yasaklı API olarak işaretlenir (NFR-07). Android 11 altı için Bölüm 12, G-01'e bakın.

---

## 4. Bileşenler ve sorumlulukları

| Bileşen | Sorumluluk | Gereksinim |
|---|---|---|
| **OnboardingScreen** | İzin istemeden önce gizlilik ve erişim gerekçesini gösterir. İzin isteği yalnızca bu ekrandaki "Devam" ile tetiklenir. | FR-01, NFR-01 |
| **PermissionService** | `photo_manager` izin durumunu tam, sınırlı veya reddedildi olarak eşler. Sınırlı izinde erişilen fotoğraf sayısını gösterir ve "seçimi genişlet" çağrısı yapar (iOS 14+: `presentLimited()`, Android 14+: `READ_MEDIA_VISUAL_USER_SELECTED` seçicisi). Reddedilmiş izinde ayarları açar. iOS'ta `PHPhotoLibraryPreventAutomaticLimitedAccessAlert=YES` ile sistemin kendiliğinden açtığı seçici kapatılır; genişletme uygulamanın kendi düğmesiyle yapılır. [K1] | FR-02, NFR-09 |
| **ScanController** (ana isolate) | Tarama durum makinesini yönetir: `listeleme → özet → aday → birebir doğrulama → tamam`. İlerlemeyi en fazla 500 ms aralıkla yayınlar. İptal bayrağını her fotoğraf arasında kontrol eder ve bekleyen küçük resim isteklerini `PMCancelToken` ile iptal eder [K2]. Kontrol noktalarını Repository'ye yazar. | FR-03, FR-15, FR-16, NFR-06 |
| **AssetEnumerator** | Sayfa sayfa varlık listesi çıkarır: id, en, boy, `createDateTime`, `modifiedDateTime`, `isFavorite`, `isLocallyAvailable`. Android'de `generationModified` ve `size` sütunlarını da okur [K2][K3]. Veritabanıyla farkı hesaplar: yeni, değişen ve kütüphanede artık olmayan kayıtlar. | FR-15, FR-08 |
| **HashWorker** (uzun ömürlü arka plan isolate'i) | Gelen küçük resim baytlarından dHash ve pHash hesaplar. Verilen dosya yolundan akışlı SHA-256 hesaplar. BK-ağacını kurar, sorgular ve aday çiftleri döndürür. Kısa ömürlü isolate'lerin başlatma ve kopyalama yükünden kaçınmak için tek uzun ömürlü worker kullanılır [K15]. | FR-04, FR-05, NFR-02, NFR-06 |
| **Repository** | sqflite erişimi. Sonuçlar her N fotoğrafta bir (geçici olarak N=200) tek batch transaction'ında yazılır. | FR-15, FR-16, FR-14 |
| **GroupingService** | Seçili hassasiyetteki eşikle aday çiftleri filtreler. "Kopya değil" çiftlerini çıkarır ve deterministik gruplama yapar (Bölüm 6.5). Birebir gruplardaki fotoğrafları "Benzer" sekmesinden dışlar (geçici, bkz. G-04). Grup sayısını ve kazanılabilir alanı hesaplar. | FR-06, FR-07, FR-14 |
| **KeeperSelector** | Her grupta saklanacak tek fotoğrafı seçer ve diğerlerini önceden işaretler. Favoriler asla önceden silinmeye işaretlenmez (Bölüm 6.6). | FR-09, NFR-07 |
| **ResultScreens** | "Birebir kopya" ve "Benzer" sekmeleri, hassasiyet seçici (Sıkı/Normal/Gevşek, varsayılan Normal), grup listesi, grup detayı, tam ekran yan yana veya ardışık karşılaştırma. Grubun tüm fotoğrafları silinmeye işaretlenirse uyarı gösterir. | FR-06, FR-07, FR-08, FR-10 |
| **DeletionService** | Toplu öneri kabulünde önce silinecek sayıyı ve alanı özetler. Onay alınınca Android'de `moveToTrash`, iOS'ta `deleteWithIds` çağırır. Dönen id listesiyle gerçekleşen silmeyi hesaplar. Boş liste veya ret durumunda "hiçbir fotoğraf silinmedi" bilgisini gösterir. | FR-11, FR-12, FR-13, NFR-07 |
| **SummaryScreen** | Silinen sayı, kazanılan alan ve geri alma yeri (iOS: Fotoğraflar > Son Silinenler, Android: Galeri > Çöp). | FR-13 |
| **SettingsScreen** | "Kopya değil" işaretlerini sıfırlama. | FR-14 |
| **Localization** | Tüm arayüz metinleri kod dışında tek bir çeviri kaynağında tutulur; ilk dil Türkçe. | NFR-11 |

---

## 5. Veri modeli

Tek SQLite veritabanı, uygulamanın özel dizininde. Yalnızca türetilmiş veriler (id, meta veri, özetler) saklanır; fotoğraf ya da küçük resim kopyası saklanmaz (NFR-01).

### 5.1 Tablolar

**`asset`**: her fotoğraf için bir satır (FR-15, FR-16)

| Sütun | Tür | Açıklama |
|---|---|---|
| `id` | TEXT PK | Platform varlık id'si (`AssetEntity.id`) |
| `width`, `height` | INTEGER | Çözünürlük (FR-08, FR-09) |
| `created_at` | INTEGER | Çekim veya oluşturma zamanı, epoch ms (FR-08, FR-09) |
| `modified_at` | INTEGER | Değişiklik zamanı (FR-15) |
| `generation_modified` | INTEGER NULL | Yalnızca Android (FR-15, Bölüm 6.7) |
| `file_size` | INTEGER NULL | Bayt. Android'de listelemede okunur. iOS'ta gerektiğinde (grup üyeleri ve birebir adaylar) orijinal dosya uzunluğundan doldurulur. |
| `is_favorite` | INTEGER (0/1) | FR-09 |
| `is_local` | INTEGER (0/1) | iOS `isLocallyAvailable`. 0 ise atlanır (geçici, S-02). |
| `dhash`, `phash` | INTEGER NULL | 64 bitlik özet. SQLite INTEGER işaretli 64 bit olduğundan bit deseni aynen saklanır. |
| `sha256` | BLOB NULL | Yalnızca birebir adaylar için hesaplanır |
| `status` | TEXT | `pending` / `hashed` / `skipped_not_local` / `error` |
| `last_seen_scan` | INTEGER | Kütüphaneden çıkan kayıtları silmek için tarama sayacı (FR-15) |

İndeksler: `(width, height)`, `status`.

**`candidate_pair`**: en gevşek eşikte bulunan çiftler (FR-05, FR-07)

| Sütun | Tür | Açıklama |
|---|---|---|
| `a_id`, `b_id` | TEXT, PK(a_id, b_id), `a_id < b_id` | Sıralı çift |
| `d_dist`, `p_dist` | INTEGER | dHash ve pHash Hamming mesafesi |
| `exact` | INTEGER (0/1/NULL) | SHA-256 doğrulama sonucu (FR-04) |

**`not_duplicate_pair`**: "Kopya değil" işaretleri (FR-14)

| Sütun | Tür | Açıklama |
|---|---|---|
| `a_id`, `b_id` | TEXT PK, `a_id < b_id` | Kullanıcının "Kopya değil" dediği grubun tüm iç çiftleri |
| `created_at` | INTEGER | |

**`scan_state`**: tek satır (FR-03, FR-16)

| Sütun | Açıklama |
|---|---|
| `scan_no` | Tarama sayacı |
| `phase` | `enumerate` / `hash` / `candidates` / `exact_verify` / `done` |
| `total`, `processed` | İlerleme (FR-03) |
| `android_media_version` | `MediaStore.getVersion()` değeri. Değişirse tam senkronizasyon yapılır [K11]. Bu değerin photo_manager üzerinden okunup okunamadığı doğrulanmadı (R-07). |
| `started_at`, `updated_at` | |

**`setting`**: anahtar ve değer çiftleri (`sensitivity`: `strict|normal|loose`, varsayılan `normal`) (FR-07)

### 5.2 Bellekte tutulan (kalıcı olmayan) veri
- Gruplar ve kullanıcının "Sakla" / "Sil" işaretleri ekran oturumu boyunca bellekte tutulur. Gruplar her açılışta ve hassasiyet değişiminde `candidate_pair` tablosundan yeniden hesaplanır (FR-07).
- BK-ağacı yalnızca tarama sırasında worker belleğinde tutulur. Artımlı taramada `hashed` kayıtlardan yeniden kurulur (Bölüm 7).

---

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Kritik karar: benzer fotoğraf tespit yöntemi

**Seçim ölçütleri:**
- Ö1: Yeniden boyutlandırma, sıkıştırma, küçük parlaklık ve renk farkına dayanıklılık (FR-05).
- Ö2: Yanlış pozitifi az, yani yüksek kesinlik; hedef ≥ %95 (NFR-04(b)).
- Ö3: Cihazda 10.000 fotoğrafı 10 dakikada işleyebilecek hız (NFR-02).
- Ö4: Cihaz dışına veri çıkmaması ve uygulama boyutunu büyütmemesi (NFR-01, kapsam).
- Ö5: Yeniden tarama olmadan hassasiyet değişebilmesi (FR-07).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| Yalnızca dHash | Yeniden boyutlandırma ve yeniden sıkıştırmayı iyi tolere eder [K13] | Az detaylı görüntülerde yanlış pozitif sık (BA 3.2) | En hızlısı: "resize and compare only" [K13] | Model yok | Mesafe saklanırsa var |
| Yalnızca pHash | Kontrast, gama ve filtre değişikliklerinde "tends to hold up better" [K13] | Tek özet | DCT maliyeti ekler [K13] | Model yok | Var |
| **dHash + pHash, ikisi de eşik altında** | İkisinin ortak gücü | Kaynağa göre "Highest precision: compute both and require both distances under their thresholds" [K13] | dHash aday filtreler, pHash yalnızca hesaplanır ve karşılaştırılır | Model yok | Var (iki mesafe saklanır) |
| Gömme vektörü + kosinüs | Kırpma ve döndürmeye de dayanıklı [K13] | "Aynı sahne, farklı an" fotoğraflarını da yakalar; yanlış pozitif riski (BA 3.2) | Mobil cihaz hız verisi yok (BA S-08) | Model dosyası boyutu artırır (BA 3.2) | Var |

**Karar: dHash + pHash birlikte.** Ö1, Ö2 ve Ö4'te en uygun seçenek. Ö3 açısından ek maliyet yalnızca pHash hesabı. Kırpma ve döndürme BA'da kapsam dışında (Bölüm 7) ve iki algısal özet de bunu yakalamaz: "neither handles crops or rotation" [K13]. Gömme vektörü S-06 ve S-08 yanıtlanana kadar ertelenir.

### 6.2 Özet hesabının girdisi

**Seçim ölçütleri:** HEIC dahil tüm galeri biçimlerinin işlenmesi (BA V-08, FR-05), hız (NFR-02), bellek (NFR-05).

| Seçenek | Biçim kapsamı | Hız / bellek |
|---|---|---|
| Orijinal dosyayı `image` paketiyle çözmek | `image` paketinde HEIC yok [K7] | Tam çözünürlükte çözme ağır, bellek yüksek |
| **İşletim sisteminden küçük resim (`thumbnailDataWithSize`, varsayılan biçim JPEG) almak ve `image` ile çözmek** | Çözmeyi işletim sistemi yapar; Dart'a JPEG gelir [K1][K2] | Küçük girdi |

**Karar:** Küçük resim yolu. Geçici boyut 256×256; dHash için 9×8, pHash için 32×32'ye indirilir, bu yüzden daha büyüğü gereksiz. Yönelim (EXIF orientation) ve küçük resim belirlenimciliği (determinism) belgede doğrulanamadı; test kapsamına alındı (R-04).

**Hesaplama (worker'da):**
- **dHash:** Gri ton, 9×8'e yeniden boyutlandırma, her satırda soldaki ve sağdaki komşu pikselin parlaklık karşılaştırmasıyla 64 bit [K13].
- **pHash:** Gri ton, 32×32, 2B DCT, sol üst 8×8 blok; her katsayı blok medyanından büyükse 1, değilse 0 [K13].
- **Mesafe:** 64 bitlik XOR sonrası bit sayımı (Hamming).

### 6.3 Kritik karar: hassasiyet eşikleri

Kaynağa göre: "Distances of 0-5 out of 64 usually mean the same image, and around 10 is a common starting threshold to tune" ve "There is no standard value" [K13]. Başka bir uygulamada, 200.000'i aşkın görüntülük kendi setlerinde 128 bitlik dHash için eşik 2 seçilmiş; 4–5 denendiğinde yanlış pozitifler başlamış (yazarın deneyimi, ölçüm verisi yok) [K14].

| Seviye | dHash eşiği | pHash eşiği | Durum |
|---|---|---|---|
| Sıkı | ≤ 4 | ≤ 4 | **Geçici** |
| Normal (varsayılan) | ≤ 8 | ≤ 8 | **Geçici** |
| Gevşek | ≤ 12 | ≤ 12 | **Geçici**; aday arama yarıçapı |

Eşik kuralı: çift ancak **iki mesafe de** seviyenin eşiği altındaysa eşleşir. Eşikler monoton olduğu için Sıkı seviyedeki grup kenarları Normal'in alt kümesidir; bu US-06'daki "eşit ya da daha az grup" beklentisini destekler. Gruplama tam bağlantılı olduğu için bunun test edilmesi gerekir (Bölüm 9).

Nihai eşikler NFR-04 test setiyle kalibre edilir. Kural: **Normal seviye, kesinlik ≥ %95 sağlanırken duyarlılığı en yüksek yapan eşik çiftidir.** Kalibrasyon sonucu sabit değer olarak koda yazılır; aynı test seti aynı eşiği verir.

### 6.4 Kritik karar: aday arama

**Seçim ölçütleri:** Ö1: 50.000 fotoğrafta makul süre (NFR-05, NFR-02). Ö2: Eşik içindeki hiçbir çiftin atlanmaması, yani tam arama (NFR-04(b) duyarlılık). Ö3: Artımlı ekleme (FR-15, NFR-03). Ö4: Sadelik.

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| Kaba kuvvet (tüm çiftler) | 50.000 fotoğrafta n(n−1)/2 ≈ 1,25×10⁹ karşılaştırma (kaynaktaki verilerden değil, n=50.000'den hesaplandı) | Tam | 100 yeni × 50.000 = 5×10⁶ karşılaştırma (hesaplandı) | En sade |
| **BK-ağacı (dHash, Hamming)** | Benzer bir kopya bulucuda kaba kuvvet "With a hundred thousand images, that's way too slow" denip BK-ağacına geçilmiş; karmaşıklık eşiğe bağlı olarak "between log N and N" (yazarın 2020 düzeltmesi) [K14] | Hamming bir metrik olduğu için tam arama | Ekleme kolay | Orta |
| Zaman penceresi (yalnızca yakın tarihliler) | Hızlı | **Eksik:** günler sonra yeniden kaydedilen veya indirilen kopyaları kaçırır (FR-05) | Kolay | Sade |

**Karar: BK-ağacı**, Gevşek eşik (12) yarıçapıyla. Bulunan her çift için pHash mesafesi de hesaplanır; yalnızca `p_dist ≤ 12` olanlar `candidate_pair`'e yazılır. Hızın eşik arttıkça düştüğü kaynakta belirtiliyor [K14]. Gevşek yarıçapta referans cihazda ölçülmesi gerekiyor (R-03). Ölçüm yetersiz çıkarsa geri dönüş yolu: kaba kuvvetin worker'da parçalara bölünmesi.

### 6.5 Kritik karar: gruplama

**Seçim ölçütleri:** Ö1: Grup içinde birbirine benzemeyen fotoğrafların bir araya gelmemesi (NFR-04(b) kesinlik, rakiplerdeki yanlış pozitif şikâyeti). Ö2: Aynı girdiyle aynı grupların oluşması (FR-07, test edilebilirlik). Ö3: 5 saniye içinde yeniden hesap (FR-07). Ö4: "Kopya değil" çiftlerine saygı (FR-14).

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| Bağlı bileşenler (union-find) | **Zayıf:** A~B ve B~C ise A ile C benzemese de aynı gruba düşer (zincirleme) | Var | Çok hızlı | Kenar çıkarılarak |
| **Açgözlü tam bağlantı** (grubun her üyesiyle eşik altı) | Güçlü | Sabit sıralama ile var | Çiftler önceden hesaplı olduğundan hızlı | Kenar çıkarılarak |
| Temsilci (lider) tabanlı | Orta: yalnızca liderle karşılaştırılır | Var | Hızlı | Var |

**Karar: Açgözlü tam bağlantı.**
1. Seviyenin eşiğindeki kenarlar alınır; `not_duplicate_pair`'deki kenarlar çıkarılır.
2. Fotoğraflar `(created_at, id)` sırasına dizilir.
3. Her fotoğraf, mevcut gruplardan tüm üyeleriyle kenarı olan ilk gruba eklenir. Uygun grup yoksa yeni grup açılır.
4. Tek üyeli gruplar atılır.

**Birebir sekmesi:** `exact = 1` kenarlarının bağlı bileşenleri kullanılır. SHA-256 eşitliği geçişli (transitive) olduğundan burada zincirleme sorunu yoktur.

**Sekme ayrımı (FR-06):** Birebir bir gruba giren fotoğraflar "Benzer" gruplamasına alınmaz (geçici, G-04).

### 6.6 Saklanacak fotoğraf önerisi (FR-09)

Deterministik sıralama: (1) `width×height` büyükten küçüğe, (2) `file_size` büyükten küçüğe, (3) `created_at` eskiden yeniye, (4) `id` (eşitlik bozucu). Kural seti S-05'e bağlı olduğu için **geçici**.

- İlk sıradaki fotoğraf "Sakla" olur; diğerleri "Sil" diye önceden işaretlenir.
- **İstisna:** `is_favorite = 1` olan fotoğraflar hiçbir zaman "Sil" ile önceden işaretlenmez, işaretsiz gelir. Böylece her grupta tam 1 öneri bulunur ve favoriler korunur.
- iOS'ta `file_size` bilinmiyorsa grup gösterilmeden önce grup üyeleri için doldurulur (R-05).

### 6.7 Birebir kopya doğrulaması (FR-04, NFR-04(a))

**Seçim ölçütleri:** Ö1: Yanlış pozitif %0 (NFR-04(a)). Ö2: "Aynı dosyanın iki kopyası her zaman aynı gruba düşer" (FR-04). Ö3: Tüm dosyaları okumama, yani hız (NFR-02).

| Seçenek | Ö1 | Ö2 | Ö3 |
|---|---|---|---|
| Her dosyanın SHA-256'sı | Var | Var | **Zayıf:** kütüphanenin tamamı okunur |
| Aday çiftlerde bayt bayt karşılaştırma | Var | Var | Var; ama sonuç saklanamaz, her taramada tekrar okunur (FR-15) |
| **Aday ön-filtresi + uzunluk + akışlı SHA-256** | Var | Var (ön-filtre gerekçesi aşağıda) | Var; özet saklanır |

**Karar: Aday ön-filtresi + uzunluk + akışlı SHA-256.**
- **Ön-filtre:** Bayt bayt aynı iki dosya aynı pikselleri ve aynı boyutu üretir. Bu yüzden aday koşulu `width, height` eşit ve `d_dist ≤ 4` (Sıkı) olarak alınır. Sıfır yerine küçük tolerans, küçük resim üretimindeki olası belirlenimsizliğe karşı güvenlik payıdır (R-04).
- **Doğrulama:** Önce dosya uzunluğu karşılaştırılır. Eşitse her iki dosyanın SHA-256'sı worker'da `startChunkedConversion` ile parça parça okunarak hesaplanır [K8]. Böylece dosya tamamen belleğe alınmaz (NFR-05). Android'de dosyaya photo_manager `originFile` ile erişilir [K2].
- **Not:** Android 10+'da `ACCESS_MEDIA_LOCATION` izni olmadan konum EXIF'i karartılmış (redacted) olarak okunur [K11]. Bu durumda yalnızca konum verisi farklı iki dosya aynı SHA-256'yı verebilir (R-06, G-06). Geçici karar: izin istenmez (NFR-01, NFR-09'da izin sayısını az tutmak için).

### 6.8 Artımlı tarama (FR-15) ve devam (FR-16)

- **Değişiklik tespiti:** Kayıt `id` ile eşlenir. Android'de `generation_modified` artmışsa, iOS'ta `modified_at` değişmişse kayıt "değişti" sayılır ve özetleri yeniden hesaplanır. Android belgesine göre `getGeneration()`, `DATE_MODIFIED` gibi tarih sütunlarından daha güvenilirdir ve medya deposu sürümü değişince tam senkronizasyon gerekir [K11].
- **Silinen kayıtlar:** Listelemede görülmeyen (`last_seen_scan < scan_no`) kayıtlar ve bunların çiftleri silinir.
- **"Kopya değil" işaretleri:** Silinmez (FR-15).
- **Devam:** Her batch commit'inde `scan_state.processed` ve `phase` güncellenir. Açılışta `phase ≠ done` ise "Taramaya devam et" sunulur ve yalnızca `status = pending` kayıtlar işlenir (US-02).

---

## 7. Performans

Mobil cihazda dHash, pHash veya SHA-256 hızı için doğrulanmış bir ölçüm kaynağı bulunamadı (BA V-04). Aşağıdaki bütçeler hesap ve tasarım hedefidir; erken prototipte referans cihazda ölçülmeleri gerekir (S-07).

| Hedef | Bütçe / tasarım tedbiri |
|---|---|
| **NFR-02:** 10.000 fotoğraf ≤ 10 dk | Fotoğraf başına ortalama ≤ 60 ms (600 s / 10.000, hesaplandı). Tedbirler: (1) tam dosya yerine 256 px küçük resim; (2) SHA-256 yalnızca birebir adaylarda; (3) küçük resim istekleri en fazla 4 eşzamanlı (geçici), worker ile boru hattı biçiminde; (4) BK-ağacı. |
| **NFR-03:** 100 yeni fotoğraf ≤ 30 sn | Yalnızca yeni ve değişen kayıtlar özetlenir. BK-ağacı veritabanındaki özetlerden yeniden kurulur (diskten okuma, görüntü işleme yok). Yalnızca 100 sorgu yapılır. |
| **NFR-05:** 50.000 fotoğraf, çökme yok | Listeleme sayfalı yapılır. Bellekte tam görüntü tutulmaz. SHA-256 akışlı. Veritabanına batch yazılır. BK-ağacındaki düğümler yalnızca 64 bit özet ve id içerir. |
| **NFR-06:** Etkileşim ≤ 200 ms | Çözme, özet, SHA-256 ve BK-ağacı arka plan isolate'inde. Flutter belgesine göre isolate kullanımının tek kesin kuralı ağır hesabın arayüzde takılma (jank) yaratmasıdır [K15]. İlerleme güncellemeleri en fazla 2/sn ile sınırlanır. |
| **FR-03:** İptal ≤ 2 sn | Her fotoğraf arasında iptal bayrağı. Bekleyen küçük resim istekleri `PMCancelToken` ile iptal edilir [K2]. SHA-256 her parçadan sonra bayrağı kontrol eder. |
| **FR-07:** Hassasiyet değişimi ≤ 5 sn | Yeniden tarama yok. `candidate_pair` tablosundan seviyeye göre filtreleme ve bellekte gruplama yapılır. |

**Ön plan şartı:** NFR-02 "uygulama ön plandayken" diye tanımlı. Arka planda kesintisiz çalışma tasarlanmadı; askıya alınma durumunda FR-16'daki devam mekanizması devreye girer.

---

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Veri cihazda kalır | Ağ, analitik veya reklam paketi yok. Android release manifestinde `INTERNET` izni bulunmaz (iOS'ta eşdeğer bir kapatma yok, bu yüzden ağ trafiği testiyle doğrulanır). | NFR-01 |
| iCloud yalnız varlıklar | `isLocallyAvailable = false` olanlar atlanır ve sayıları kullanıcıya gösterilir; ağ indirmesi tetiklenmez (geçici, S-02, G-03). | NFR-01 |
| Saklanan veri | Yalnızca id, meta veri ve özetler, uygulamanın özel dizininde. Küçük resim veya fotoğraf kopyası saklanmaz. | NFR-01 |
| Onaysız silme yok | Silme yalnızca kullanıcının özet ekranındaki onayından sonra ve işletim sisteminin onay diyaloğuyla yapılır. `MANAGE_MEDIA` beyan edilmez. Android'de `deleteWithIds` yasak (Bölüm 3.3). Uygulama fotoğraf içeriğine yazmaz; `createWriteRequest` veya düzenleme API'si kullanılmaz. | NFR-07, FR-12 |
| İzinlerin en aza indirilmesi | Android: `READ_MEDIA_IMAGES`, `READ_MEDIA_VISUAL_USER_SELECTED` ve photo_manager'ın zorunlu tuttuğu `READ_MEDIA_VIDEO` (R-01). `ACCESS_MEDIA_LOCATION` yok (geçici, G-06). iOS: `NSPhotoLibraryUsageDescription`. | NFR-09, FR-02 |
| Mağaza beyanı | Play Console'da geniş erişim gerekçesi verilir. Reddedilmiş veya sınırlı erişimde uygulama erişilebilen fotoğraflarla çalışır (FR-02). | NFR-09 |

---

## 9. Test stratejisi

| Test | Kapsam | Gereksinim |
|---|---|---|
| **Birim: özetler** | dHash ve pHash için sabit girdi görsellerinde beklenen bit desenleri (altın değerler), Hamming fonksiyonu, BK-ağacının kaba kuvvetle aynı çiftleri bulması (rastgele 5.000 özetle karşılaştırma) | FR-05, NFR-04 |
| **Birim: gruplama** | Zincirleme örneği (A~B, B~C, A≁C) iki gruba ayrılır. Aynı girdiyle aynı gruplar oluşur. Sıkı ≤ Normal ≤ Gevşek grup sayısı ilişkisi. "Kopya değil" çiftleri. Bir fotoğraf iki sekmede birden görünmez. | FR-06, FR-07, FR-14, US-06 |
| **Birim: öneri** | Her grupta tam 1 "Sakla". Favoriler hiçbir koşulda "Sil" ile önceden işaretlenmez (birden fazla favori olan gruplar dahil). Tüm fotoğraflar "Sil" işaretlenince uyarı çıkar. | FR-09, FR-10, US-05 |
| **Doğruluk seti** | ≥ 500 etiketli çift: birebir, yeniden sıkıştırılmış, boyutu değişmiş, ardışık çekim, farklı sahne; HEIC ve EXIF yönelimli örnekler dahil. Birebir sekmesinde yanlış pozitif 0; Normal seviyede kesinlik ≥ %95, duyarlılık ≥ %85. Eşik kalibrasyonu bu setle yapılır (Bölüm 6.3). | NFR-04, FR-04, FR-05 |
| **Cihaz entegrasyonu** | İzin matrisi (tam, sınırlı, reddedildi; iOS ve Android 14+). Silme onayı kabul ve ret senaryoları. Silinenlerin Son Silinenler veya Çöp'ten geri yüklenebilmesi. Tarama sırasında uygulamanın öldürülüp devam ettirilmesi. Artımlı taramada önceden işlenmiş fotoğrafın yeniden özetlenmemesi (sayaçla doğrulanır). | FR-01, FR-02, FR-12, FR-13, FR-15, FR-16, US-08 |
| **Performans** | Referans cihazda 10.000 fotoğraf ≤ 10 dk, +100 fotoğraf ≤ 30 sn, 50.000 fotoğrafta 3 ardışık tam tarama ve 0 çökme. Tarama sırasında etkileşim gecikmesi ≤ 200 ms. İptal ≤ 2 sn. Hassasiyet değişimi ≤ 5 sn. | NFR-02, NFR-03, NFR-05, NFR-06, FR-03, FR-07 |
| **Gizlilik** | Uçak modunda uçtan uca tarama ve silme. Ağ trafiğinde görüntü verisi içeren istek sayısı 0. | NFR-01 |
| **Güvenlik** | Kod incelemesi kontrol listesi: `deleteWithIds` Android yolunda yok; `MANAGE_MEDIA` yok; onaysız silme yolu yok. | NFR-07 |
| **Erişilebilirlik ve dil** | VoiceOver ve TalkBack ile tüm ana akışlar; büyük sistem yazı boyutu; çevrilmemiş metin taraması. | NFR-10, NFR-11 |
| **Platform matrisi** | S-03 ile belirlenen en düşük ve en güncel sürümler. | NFR-08 |

---

## 10. Riskler

| # | Risk | Etki | Azaltma |
|---|---|---|---|
| R-01 | photo_manager belgesine göre Android 13+'da yalnızca görsel istense bile `READ_MEDIA_VIDEO` beyanı zorunlu [K1]. Video işlevi olmayan bir uygulamada bu beyan Google Play politikasında sorun yaratabilir. | NFR-09 | Mağaza beyanında gerekçe hazırlanır. Prototipte yalnızca `READ_MEDIA_IMAGES` ile çalışıp çalışmadığı denenir. Gerekirse BA'ya iletilir (G-07). |
| R-02 | `deleteWithIds` davranışı belgelerde çelişkili: changelog 2.8.0 "delete instead of moving to Trash on Android 30+" [K4], taşıma kılavuzu "only move assets to the trash on Android 30 and above" [K5]. | FR-12, NFR-07 | Android'de yalnızca `moveToTrash`. Cihaz testinde silinenlerin Çöp'te göründüğü doğrulanır. |
| R-03 | BK-ağacının Gevşek yarıçapta (12) hız avantajı azalabilir [K14]. Mobil ölçüm yok. | NFR-02, NFR-05 | Erken prototip ölçümü. Yetersizse Gevşek eşiğin düşürülmesi veya parçalı kaba kuvvet. |
| R-04 | İşletim sisteminin küçük resminin yönelimi uygulayıp uygulamadığı ve aynı dosya için bayt bayt aynı küçük resmi üretip üretmediği belgede doğrulanamadı. | FR-04, FR-05 | Birebir ön-filtresinde tolerans (≤ 4). Test setine yönelimli ve kopyalanmış dosyalar eklenir. |
| R-05 | iOS'ta dosya boyutu sütunu yok [K3]. Boyut orijinal dosyaya erişerek alınmalı; bunun maliyeti ve iCloud indirmesini tetikleyip tetiklemediği doğrulanmadı. | FR-06, FR-08, FR-09, NFR-01, NFR-02 | Boyut yalnızca grup üyeleri ve birebir adaylar için alınır ve yalnızca `is_local = 1` kayıtlarda istenir. Prototipte ölçülür. |
| R-06 | `ACCESS_MEDIA_LOCATION` olmadan Android'de konum EXIF'i karartılır [K11]; yalnızca konumu farklı iki dosya "birebir" sayılabilir. | NFR-04(a) | Teste özel durum olarak eklenir; BA'ya sorulur (G-06). |
| R-07 | `MediaStore.getVersion()` ve `generationModified` değerlerinin photo_manager üzerinden okunabildiği yalnızca sütun adının varlığıyla görüldü [K3]; davranışı ve sürüm şartı doğrulanmadı. | FR-15 | Okunamazsa Android'de de `modified_at` kullanılır; ayrıca her taramada id kümesi karşılaştırılır. |
| R-08 | iOS silmede sistem onay diyaloğu ve Son Silinenler davranışı Apple belgesinde doğrulanamadı (BA V-05). Yalnızca photo_manager sürüm notu "native deletion confirmation dialogs on iOS" ifadesini içeriyor [K12]. | FR-12, NFR-07 | Cihaz testinde doğrulanır. |
| R-09 | Flutter'ın desteklediği en düşük iOS ve Android sürümleri bu oturumda kontrol edilmedi. sqflite iOS 12 istiyor [K9], `presentLimited` iOS 14+ [K1], `moveToTrash` Android 11+ [K1]. | NFR-08 | En düşük sürümler S-03 yanıtıyla birlikte belirlenir. |
| R-10 | `photo_manager_image_provider` son 2 yıldır yayın almamış [K6]. | FR-08 | Gerekirse `thumbnailDataWithSize` + `Image.memory` ile değiştirilir; bileşen arayüzü buna göre ince tutulur. |
| R-11 | Eşikler kalibrasyon öncesinde NFR-04(b)'yi sağlamayabilir; Normal seviyede kesinlik ve duyarlılık hedefleri aynı anda tutmayabilir. | NFR-04 | Kalibrasyon kuralı (Bölüm 6.3). Tutmazsa sonuç BA'ya raporlanır. |

---

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | İzinden önce bilgilendirme ekranı | OnboardingScreen; izin isteği yalnızca bu ekrandan tetiklenir | 4, 8 |
| FR-02 | Tam, sınırlı ve reddedilmiş izin durumları | PermissionService; `presentLimited` (iOS 14+), `READ_MEDIA_VISUAL_USER_SELECTED` (Android 14+), ayarlara yönlendirme | 3.1, 4, 8 |
| FR-03 | Elle başlatma, ilerleme, iptal | ScanController; ≤ 500 ms ilerleme, iptal bayrağı + `PMCancelToken` | 4, 7 |
| FR-04 | Bayt düzeyinde birebir gruplama | Aday ön-filtresi + uzunluk + akışlı SHA-256; bağlı bileşenler | 6.5, 6.7 |
| FR-05 | Görsel olarak neredeyse aynı fotoğrafları gruplama | dHash + pHash, küçük resim girdisi, BK-ağacı, tam bağlantılı gruplama | 6.1, 6.2, 6.4, 6.5 |
| FR-06 | Birebir ve Benzer sekmeleri, grup sayısı, alan | GroupingService, ResultScreens; birebir üyeleri Benzer'den dışlanır (geçici) | 4, 6.5 |
| FR-07 | 3 hassasiyet seviyesi, ≤ 5 sn güncelleme | `candidate_pair` tablosunda iki mesafe; bellekte yeniden gruplama; `setting.sensitivity` | 5, 6.3, 7 |
| FR-08 | Küçük resim, çözünürlük, boyut, tarih | AssetEnumerator, `photo_manager_image_provider`, `asset` tablosu | 3.1, 4, 5 |
| FR-09 | Saklanacak fotoğraf önerisi, favori koruması | KeeperSelector; deterministik sıralama (geçici), favori istisnası | 6.6 |
| FR-10 | İşaret değiştirme, tam ekran karşılaştırma, tümü silinirse uyarı | ResultScreens | 4 |
| FR-11 | Toplu öneri kabulü ve özet | DeletionService ön-özeti | 4 |
| FR-12 | Yalnızca onaylı ve geri alınabilir silme | Android `moveToTrash`, iOS `deleteWithIds`; Android'de `deleteWithIds` yasak; `MANAGE_MEDIA` yok | 3.3, 8 |
| FR-13 | Silme sonrası özet | SummaryScreen; dönen id listesiyle hesap | 4 |
| FR-14 | "Kopya değil" işareti ve sıfırlama | `not_duplicate_pair` tablosu, GroupingService, SettingsScreen | 5, 6.5 |
| FR-15 | Artımlı tarama | AssetEnumerator farkı; `generation_modified` / `modified_at`; `last_seen_scan` | 6.8, 7 |
| FR-16 | Kesilen taramaya devam | `scan_state`, batch commit kontrol noktaları, `status = pending` | 5, 6.8 |
| NFR-01 | Veri cihazdan çıkmaz, çevrimdışı çalışır | Ağ paketi yok, `INTERNET` izni yok (Android), yerel SQLite, iCloud yalnız varlıklar atlanır | 8 |
| NFR-02 | 10.000 fotoğraf ≤ 10 dk | Küçük resim girdisi, aday ön-filtresi, BK-ağacı, boru hattı | 6.4, 6.7, 7 |
| NFR-03 | 100 yeni fotoğraf ≤ 30 sn | Artımlı tarama, kayıtlı özetlerden BK-ağacı | 6.8, 7 |
| NFR-04 | Birebirde %0 yanlış pozitif; Benzer'de kesinlik ≥ %95, duyarlılık ≥ %85 | SHA-256 doğrulaması; dHash VE pHash; tam bağlantı; eşik kalibrasyonu | 6.1, 6.3, 6.5, 6.7, 9 |
| NFR-05 | 50.000 fotoğrafta çökme yok | Sayfalı listeleme, akışlı SHA-256, sqflite batch, ince BK düğümleri | 3.2, 7 |
| NFR-06 | Etkileşim ≤ 200 ms | Uzun ömürlü HashWorker isolate'i | 4, 7 |
| NFR-07 | Onaysız silme veya değişiklik yok | DeletionService akışı, işletim sistemi onayı, yasaklı API kontrolü | 3.3, 8, 9 |
| NFR-08 | iOS ve Android; Android 11+ toplu çöp | `moveToTrash` (API 30+); en düşük sürümler S-03 ile (geçici) | 3.1, 10 (R-09), 12 |
| NFR-09 | Google Play Fotoğraf ve Video İzinleri politikası | En az izin, beyan gerekçesi, sınırlı erişimde çalışma | 8, 10 (R-01) |
| NFR-10 | Ekran okuyucu etiketleri, sistem yazı boyutu | Tüm etkileşimli öğelere semantik etiket; ölçeklenen metin; VoiceOver/TalkBack testi | 9 |
| NFR-11 | En az Türkçe arayüz | Localization bileşeni, kod dışı çeviri kaynağı | 4, 9 |

---

## 12. BA'ya geri sorular

- **G-01 (S-03, NFR-08, FR-12):** Android 10 ve altında sistem çöpü API'si yok; `moveToTrash` Android 11+ istiyor [K1]. Geçici karar en düşük Android sürümünü 11 (API 30) yapmak. Alternatif: Android 10'da tarama olsun ama silme devre dışı kalsın. Hangisi kabul edilir? En düşük iOS sürümü için de, sınırlı erişim seçicisi iOS 14+ olduğundan, iOS 14 önerilir.
- **G-02 (S-05, FR-09):** Geçici öncelik sırası: çözünürlük, dosya boyutu, en eski tarih, id. Bir grupta favori varsa favori mi "Sakla" önerisi olmalı, yoksa kurala göre seçilip favori yalnızca işaretsiz mi bırakılmalı? Birden fazla favori olan grupta beklenen davranış nedir?
- **G-03 (S-02, NFR-01):** iCloud'da olup cihazda olmayan fotoğraflar geçici olarak atlanıyor ve sayıları gösteriliyor. iCloud'dan indirme kullanıcıya gelen bir trafik olsa da NFR-01'in "internet olmadan çalışır" koşuluyla nasıl bağdaştırılmalı?
- **G-04 (FR-06):** "Bir fotoğraf iki sekmede birden gösterilmez" kuralı için geçici karar, birebir gruptaki fotoğrafları Benzer sekmesinden tamamen çıkarmak. Bu durumda A, A′ (birebir) ve B (A'ya benzer) için B tek başına kalır ve gösterilmez. Birebir grubun saklanan fotoğrafı Benzer gruplamasına katılmalı mı?
- **G-05 (FR-14):** "Kopya değil" denilen grup, içindeki tüm çiftler eşleşme dışı sayılarak saklanıyor. Sonraki taramada bu gruptaki bir fotoğraf yeni bir fotoğrafla yeni bir grup oluşturursa gösterilmeli mi? Geçici karar: gösterilir.
- **G-06 (NFR-04(a), FR-04):** Android'de `ACCESS_MEDIA_LOCATION` izni olmadan yalnızca konum EXIF'i farklı iki dosya "birebir" görünebilir [K11]. Bu durum "birebir kopya" sayılabilir mi, yoksa ek çalışma zamanı izni istenmeli mi?
- **G-07 (NFR-09):** photo_manager, Android 13+'da `READ_MEDIA_VIDEO` beyanını da zorunlu tutuyor [K1]; video işlevi kapsam dışı (S-01). Mağaza beyanı için bu kabul edilebilir mi, yoksa eklenti dışında bir erişim yolu araştırılmalı mı?
- **G-08 (S-11, V-08):** Live Photo (iOS'ta `isLivePhoto` ile ayırt edilebiliyor [K2]), seri çekim ve RAW+JPEG çiftleri için karar verilene kadar bunlar sıradan fotoğraf gibi işleniyor. Bu geçici yaklaşım uygun mu?
- **G-09 (S-07, NFR-02, NFR-03, NFR-04):** Referans cihaz ve NFR-04 test seti kim tarafından ne zaman sağlanacak? Eşik kalibrasyonu (Bölüm 6.3) bu sete bağlı.

---

## 13. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| K1 | photo_manager, pub.dev | https://pub.dev/packages/photo_manager | Sürüm 3.12.0. "Minimum platform versions: Android 16, iOS 9.0". "`READ_MEDIA_IMAGES` and `READ_MEDIA_VIDEO` are both required whether requesting images or videos." `READ_MEDIA_VISUAL_USER_SELECTED` (Android 14). `MANAGE_MEDIA` onay diyaloğunu kaldırır. `presentLimited()` yalnızca iOS 14+. `moveToTrash` "supports Android 11 and above versions". `deleteWithIds`: "This method will delete the asset completely from your device." `isLocallyAvailable` false ise genellikle yalnızca iCloud'da. Küçük resim biçimi varsayılanı JPEG. |
| K2 | AssetEntity class, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AssetEntity-class.html | `isFavorite`, `width`, `height`, `createDateTime`, `modifiedDateTime`, `isLocallyAvailable`, `originFile` ("keeps all EXIF metadata"), `thumbnailDataWithSize(..., cancelToken)`; `isLivePhoto` "Only valid on iOS/macOS". |
| K3 | AndroidMediaColumns ve DarwinColumns, photo_manager API | https://pub.dev/documentation/photo_manager/latest/photo_manager/AndroidMediaColumns-class.html , https://pub.dev/documentation/photo_manager/latest/photo_manager/DarwinColumns-class.html | Android sütunları: `size`, `generationModified`, `isFavorite`, `modifiedDate`. Darwin sütunlarında dosya boyutu yok. |
| K4 | photo_manager 2.8.1 changelog | https://pub.dev/packages/photo_manager/versions/2.8.1/changelog | 2.8.0: `deleteWithIds` "The behavior changes to delete instead of moving to Trash on Android 30+." `moveToTrash` eklendi, "the method only support Android 30+." |
| K5 | photo_manager MIGRATION_GUIDE | https://raw.githubusercontent.com/fluttercandies/flutter_photo_manager/main/MIGRATION_GUIDE.md | "`PhotoManager.editor.deleteWithIds` only move assets to the trash on Android 30 and above." (K4 ile çelişiyor.) |
| K6 | photo_manager_image_provider, pub.dev | https://pub.dev/packages/photo_manager_image_provider | `AssetEntityImage`, `AssetEntityImageProvider`; photo_manager ">=2.6.0 <4.0.0"; Android/iOS/macOS; 2.2.0, "Published 2 years ago". |
| K7 | image, pub.dev | https://pub.dev/packages/image | 4.13.1; desteklenen biçimler (JPEG, PNG, WebP vb.), HEIC listede yok; "written in pure Dart"; isolate komut API'si. |
| K8 | crypto, pub.dev | https://pub.dev/packages/crypto | 3.0.8; SHA-256; "Invoke the `startChunkedConversion` method to create a sink for the input data"; "has not been reviewed or vetted by security professionals". |
| K9 | sqflite, pub.dev | https://pub.dev/packages/sqflite | 2.4.6; "minSdk is 19", "iOS 12"; ana isolate tercihi; transaction'da `txn`; batch geri alma davranışı. |
| K10 | drift, pub.dev | https://pub.dev/packages/drift | "built on top of SQLite"; "generates type-safe code based on your tables"; isolate'ler arası yerleşik destek. |
| K11 | Access media files from shared storage, Android Developers | https://developer.android.com/training/data-storage/shared/media | `createTrashRequest`: "Items in the trash are permanently deleted after a system-defined time period." Android 11+'da varsayılan galeri olmayan uygulama "must show a confirmation dialog to the user each time your app attempts to modify or delete a file". `getGeneration()` "is more robust than the dates in media columns". `getVersion()` değişirse tam senkronizasyon. `ACCESS_MEDIA_LOCATION` ve karartılmış EXIF. |
| K12 | photo_manager changelog (güncel) | https://pub.dev/packages/photo_manager/changelog | Sürüm notu: "Fix delayed native deletion confirmation dialogs on iOS by elevating the dispatch queue priority of ...". İki sorguda sürüm numarası farklı döndü (3.10.0 / 3.12.0); sürüm numarası doğrulanamadı. 3.11.0'da `restoreFromTrash` ve `isTrashed` eklendi. |
| K13 | pHash vs dHash: Perceptual Image Hashing, MojoAuth | https://mojoauth.com/security-guides/phash-vs-dhash | dHash 9×8 ve pHash 32×32 DCT adımları. dHash "Faster (resize and compare only)". "pHash tends to hold up better under contrast, gamma and filter changes". "neither handles crops or rotation". "Distances of 0-5 out of 64 usually mean the same image, and around 10 is a common starting threshold to tune"; "There is no standard value". "Highest precision: compute both and require both distances under their thresholds." |
| K14 | Duplicate image detection with perceptual hashing in Python, Ben Hoyt | https://benhoyt.com/writings/duplicate-image-detection/ | Yazarın deneyimi (ölçüm verisi yok): yaklaşık 150.000 fotoğrafta "With a hundred thousand images, that's way too slow" (kaba kuvvet); BK-ağacı karmaşıklığı "between log N and N, depending on the distance threshold" (2020 düzeltmesi); 200.000'i aşkın görüntülük kendi setlerinde 128 bit dHash eşiği 2, 4–5'te yanlış pozitif başlamış. |
| K15 | Concurrency and isolates, Flutter docs | https://docs.flutter.dev/perf/isolates | "the only one hard rule ... when large computations are causing your Flutter application to experience UI jank"; uzun ömürlü isolate'ler; isolate'ler bellek paylaşmaz; arka plan isolate'inde UI işi yapılamaz ve platformdan istenmemiş mesaj alınamaz. |

**Not:** Arama sonuçlarında görülen ama açılmayan sayfalar (Go ve Node BK-ağacı kütüphaneleri, medikal görüntüleme makalesi, photo_manager GitHub PR'ı) kaynak gösterilmedi. `pub.dev` üzerindeki `AndroidEditor` ve `Editor.deleteWithIds` API sayfaları 404 döndü; silme davranışı K1, K4, K5 ve K12'den alındı.