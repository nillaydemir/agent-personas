# Çarpım Tablosu Öğrenme Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v2
- **Girdi:** /home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v1-run2.md

---

## 1. Özet

BA dokümanında 17 fonksiyonel gereksinim (FR-01 – FR-17), 10 fonksiyonel olmayan gereksinim (NFR-01 – NFR-10) ve 7 kullanıcı hikâyesi (US-01 – US-07) var. Ürün küçük, tek kullanıcılı ve hesapsız. Sunucuda veri tutulmuyor (NFR-07).

Önerilen çözüm:

- **Sunucusuz, statik bir web uygulaması.** TypeScript ile yazılır ve Vite ile paketlenir. UI çatısı (framework) kullanılmaz.
- **Tüm mantık ve veri cihazda kalır.** İlerleme `localStorage` içinde tek bir JSON belgesinde tutulur. Her onaylanan cevaptan sonra eşzamanlı (synchronous) olarak yazılır (FR-15, NFR-10).
- **Web uygulama bildirimi (manifest) eklenir.** Uygulama ana ekrana eklenebilir. Amaç, Safari'de verinin proaktif silinme riskini azaltmaktır (bkz. Riskler R-01).
- **Saf (yan etkisiz) bir "alıştırma motoru" modülü** soru seçimini (FR-05, FR-06, FR-09), oturum içi tekrarı (FR-08), öğrenme durumunu (FR-10) ve tablo yıldızlarını (FR-13) yönetir. Rastgelelik dışarıdan verilir, bu sayede motor tekrarlanabilir biçimde test edilir.

**"Geçici" kararlar.** Platform (BA Açık soru 1) ve internetsiz çalışma (BA Açık soru 2) BA'da açık. Bu iki konudaki kararlar "geçici" işaretlidir ve 12. bölümde BA'ya geri sorulur. FR-08, FR-09 ve FR-13'teki bazı uç durumlar da yorum gerektiriyor. Bunlar için seçilen yorumlar "geçici" işaretlidir ve aynı bölümde sorulur.

## 2. Mimari genel bakış

```
┌──────────────────────────── Tarayıcı (cihaz) ────────────────────────────┐
│                                                                           │
│  UI katmanı (ekranlar, DOM)                                               │
│   Ana ekran │ İnceleme │ Alıştırma │ Oturum özeti │ İlerleme │ Ayarlar    │
│        │                    │                                             │
│        ▼                    ▼                                             │
│  Uygulama denetleyicisi (ekran geçişleri, zamanlayıcı, girdi)             │
│        │                    │                                             │
│        ▼                    ▼                                             │
│  Alıştırma motoru (saf TS)         Metin tablosu (tr)                     │
│   - soru havuzu ve seçim            - tüm arayüz metinleri (NFR-01)       │
│   - oturum içi tekrar kuyruğu                                             │
│   - öğrenme durumu ve yıldız                                              │
│        │                                                                  │
│        ▼                                                                  │
│  Depolama bağdaştırıcısı ──► localStorage (tek anahtar, sürümlü JSON)     │
│                                                                           │
└───────────────────────────────────────────────────────────────────────────┘
        ▲ yalnızca ilk yüklemede statik dosyalar (HTML/JS/CSS/ikon)
        │ HTTPS statik barındırma; arka uç, API ve analitik yok
```

- **Arka uç yok.** Uygulama ilk yüklemeden sonra ağa istek göndermez. Bu karar NFR-07'ye (veri toplanmaz) ve NFR-08'e (reklam ve satın alma yok) bağlıdır.
- **Tek sayfa, altı ekran.** Ekranlar FR'lerden türetilmiştir:
  - Ana ekran: FR-01, FR-05, FR-06, NFR-02
  - İnceleme: FR-02, FR-03
  - Alıştırma: FR-04, FR-07, FR-08, FR-14
  - Oturum özeti: FR-12, FR-13
  - İlerleme: FR-11, FR-13
  - Ayarlar: FR-14, FR-16, FR-17
- **Ana ekran ve NFR-02.** Tablo seçimi, soru sırası ve oturum uzunluğu ana ekranda varsayılan değerleriyle durur. Böylece bir oturum en az 2 dokunuşla başlatılabilir: bir tablo seçmek ve "Başla"ya basmak. NFR-02'nin üst sınırı 3 dokunuştur.
- **İnternetsiz çalışma (geçici).** Service worker yalnızca BA Açık soru 2'ye "evet" cevabı gelirse eklenir. Mimari buna hazırdır (bkz. Bileşen K-07). Hiçbir FR/NFR şu an internetsiz çalışmayı gerektirmiyor.

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Kritik karar: Platform ve dağıtım biçimi (geçici)

Seçim ölçütleri:

| # | Ölçüt | Bağlı gereksinim / risk |
|---|---|---|
| Ö1 | Hedef cihaz bilinmeden olabildiğince çok cihaza ulaşma | BA Açık soru 1, NFR-02 |
| Ö2 | Sunucu, hesap ve veri toplama olmadan çalışma | NFR-07, NFR-08 |
| Ö3 | İlerlemenin cihazda kalıcı tutulması | FR-15, NFR-10 |
| Ö4 | Küçük ürüne uygun sadelik ve az bağımlılık | Kapsam kuralı, NFR-08 |
| Ö5 | Geri bildirimin hızlı görünmesi | NFR-03 |

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. Statik web uygulaması (ana ekrana eklenebilir)** | Telefon, tablet ve bilgisayarda tek kod tabanı; mağaza gerekmez | Statik dosya sunumu yeterli | `localStorage` ile kalıcı. Safari'de 7 gün etkileşimsizlik silme kuralı var (R-01) | Tek dil, tek derleme aracı | Tüm mantık yerel |
| B. Çapraz platform yerel uygulama (mobil) | Yalnızca mobil; bilgisayar tarayıcısı ayrı iş | Sağlanır | Uygulama verisi alanında kalıcı | Mağaza yayını, imzalama ve mağaza politikaları ek iş getirir | Yerel |
| C. Platform başına yerel uygulama | En dar erişim, iki kod tabanı | Sağlanır | Kalıcı | En yüksek maliyet | Yerel |

**Karar: A (geçici).** Ö1 ve Ö4'te açık üstünlüğü var. Ö3'teki zayıflık R-01'de ele alınır.

Platform cevabı "yalnızca mobil uygulama mağazası" gelirse karar B'ye döner. Motor ve veri modeli (bölüm 5 ve 6) platformdan bağımsız tasarlandı, dolayısıyla taşınabilir.

### 3.2 Kritik karar: İlerleme verisinin saklanması

Seçim ölçütleri:

| # | Ölçüt | Bağlı gereksinim / risk |
|---|---|---|
| Ö1 | Son onaylanan cevap, uygulama çökse bile kaybolmamalı; yazma, geri bildirimden önce tamamlanmalı | NFR-10, FR-15 |
| Ö2 | Sıfırlamada tüm ilerlemenin tek adımda silinmesi | FR-16 |
| Ö3 | Veri boyutuna uygunluk (100 soru durumu, birkaç ayar) | FR-10, FR-13, FR-17 |
| Ö4 | Sadelik | Kapsam kuralı |

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 |
|---|---|---|---|---|
| **A. `localStorage`, tek anahtar, tek JSON belgesi** | Web Storage API eşzamanlıdır ([MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)). `setItem` döndüğünde yazma çağrısı tamamlanmış olur | Tek `setItem` çağrısı | Kaynak, köken (origin) başına 5 MiB sınır veriyor ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Bu veri birkaç KB'dir (çıkarım) | Bağımlılıksız, birkaç satır |
| B. IndexedDB | Eşzamansız; işlem tamamlanmadan geri bildirim gösterilirse çökme anında kayıp olabilir, bunu önlemek ek akış ister | İşlemle mümkün | Büyük veri için tasarlanmış, burada gereksiz | Daha fazla kod veya yardımcı kütüphane |
| C. Uzak sunucu | Ağa bağlı | Mümkün | Uygun | NFR-07 ile çelişir: hesap/kimlik gerekir. Elendi |

**Karar: A.** Her cevap onayında yazma sırası şöyledir:

1. Motor durumu günceller.
2. `localStorage.setItem` çağrılır.
3. Geri bildirim çizilir.

Böylece "son onaylanan cevaba kadar" olan ilerleme saklanmış olur (NFR-10).

### 3.3 Kritik karar: Cevap girişi (sayısal tuş takımı)

Seçim ölçütleri: FR-04 (yalnızca rakam, en fazla 3 hane), NFR-05 (dokunma alanı), NFR-02 (dokunuş sayısı), NFR-03 (yanıt süresi).

| Seçenek | FR-04 | NFR-05 | NFR-02 / NFR-03 |
|---|---|---|---|
| **A. Ekran üstü özel tuş takımı (0–9, Sil, Tamam) ve fiziksel klavye rakamları** | Yalnızca rakam tuşu var; 3 hane sınırı kodda | Tuş boyutu tasarımda doğrudan ayarlanır | Sistem klavyesinin açılma gecikmesi ve ekranı kapatması yok |
| B. `<input inputmode="numeric">` ve sistem klavyesi | `inputmode` yalnızca bir ipucudur ve geçerlilik denetimi yapmaz ([MDN: inputmode](https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inputmode)); ek doğrulama gerekir | Tuş boyutu cihaza bağlı, denetlenemez | Klavye cihaza göre değişir |

**Karar: A.** Masaüstünde fiziksel klavyedeki 0–9, Backspace ve Enter tuşları da aynı işlevlere bağlanır.

### 3.4 UI çatısı

Bu kritik bir karar değildir, kısaca gerekçelendirilmiştir. Ekran sayısı altıdır ve durum küçüktür. Bu nedenle bir UI çatısı (framework) yerine doğrudan DOM API'si kullanılır.

Gerekçeler:
- Kapsam kuralı: küçük ürüne sade mimari.
- NFR-07 ve NFR-08: üçüncü taraf çalışma zamanı kodu en aza iner.
- NFR-03: ek soyutlama katmanı yok.

### 3.5 Teknoloji seçimleri tablosu

| Bileşen | Seçim | Gerekçe (ID) | İzinler | En düşük platform | Bilinen kısıtlar (belgeden) |
|---|---|---|---|---|---|
| Dil | TypeScript (geliştirme bağımlılığı, npm paketi `typescript`) | Motorun FR-05, FR-08, FR-09 ve FR-10 kurallarının tip güvenli yazılması | Yok (derleme zamanı aracı) | Çalıştırmak için Node.js gerekir; belgede sürüm yazmıyor ([typescriptlang.org/download](https://www.typescriptlang.org/download/)) | Tarayıcıya JavaScript olarak derlenerek gider |
| Derleme ve geliştirme sunucusu | Vite | Statik çıktı üretir, sunucusuz dağıtıma uygun (NFR-07) | Yok | Varsayılan üretim hedefi Chrome/Edge 111, Firefox 114, Safari 16.4 ([Vite: Building for Production](https://vite.dev/guide/build.html)) | "Vite only handles syntax transforms and does not cover polyfills." Vite `.ts` dosyalarında tip denetimi yapmaz ve `isolatedModules: true` ister ([Vite: Features](https://vite.dev/guide/features.html)). Bu yüzden derlemeye `tsc --noEmit` eklenir |
| Kalıcı depolama | `localStorage` (tarayıcı API'si) | FR-15, FR-16, FR-17, NFR-10 | Kullanıcı izni istenmez | Tüm hedef tarayıcılar | Köken başına 5 MiB; dolunca `QuotaExceededError` ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Kullanıcı kalıcı veriyi engellediyse veya `file:` kökeninde `SecurityError` verir. Gizli gezintide son gizli sekme kapanınca silinir ([MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). Safari'de izleme önleme açıkken 7 günlük kullanımda etkileşim olmayan kökenin betikle yazılmış verisi silinir (aynı MDN kaynağı) |
| Ana ekrana ekleme | Web uygulama bildirimi (`manifest.json`) | R-01 azaltımı: ana ekrandaki web uygulamaları Safari'nin gün sayacından ayrıdır (FR-15) | Yok | iOS 16.4 ve sonrasında Safari, Chrome, Edge, Firefox ve Orion'un Paylaş menüsünden kurulabilir; iOS 16.3 ve öncesinde yalnızca Safari ([MDN: Making PWAs installable](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable)) | HTTPS (veya `localhost`) gerekir. Chromium `name`/`short_name`, 192 ve 512 px `icons`, `start_url` ve `display` ister. iOS'ta özel kurulum düğmesi API'si (`beforeinstallprompt`) desteklenmez |
| İnternetsiz çalışma (koşullu, geçici) | Service worker ile statik dosya önbelleği | Yalnızca BA Açık soru 2 "evet" ise | Yok | "enabled by default in all modern browsers" ([MDN: Using Service Workers](https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers)) | Yalnızca HTTPS'te çalışır. Gizli gezinti veya çerez engelleme gibi ayarlarda kayıt başarısız olabilir. Güncellenen worker, eski sayfalar kapanana kadar etkinleşmez |
| İçerik güvenlik politikası | `<meta http-equiv="Content-Security-Policy">` | NFR-07, NFR-08: dış bağlantı ve dış betik engeli | Yok | Tüm hedef tarayıcılar | `<meta>` ile verilen politika "does not support all CSP features"; rapor modu `<meta>` ile verilemez ([MDN: CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)) |
| Birim testi | Vitest | Motorun ve kuralların doğrulanması (bölüm 9) | Yok | "Vitest requires Vite >=v6.4.0 and Node >=v22.12.0" ([Vitest](https://vitest.dev/guide/)) | Test dosya adında `.test.` veya `.spec.` olmalı |
| Uçtan uca test | Playwright | US kabul kriterleri; NFR-02, NFR-03, NFR-05, NFR-07, NFR-10 ölçümleri | Yok | Node 22.x, 24.x veya 26.x; Windows 11+, macOS 14+, Debian 12/13, Ubuntu 22.04/24.04/26.04 ([Playwright](https://playwright.dev/docs/intro)) | Chromium, WebKit ve Firefox; Android Chrome ve Mobile Safari için emülasyon. Emülasyon gerçek cihazın yerini tutmaz (çıkarım). Gerçek cihaz ölçümü ayrıca yapılır |
| Ses (koşullu) | `HTMLAudioElement` | Yalnızca ses efekti olacaksa (FR-17) | Yok | Tüm hedef tarayıcılar | Betikle başlatılan ses, kullanıcı etkileşimi olmadan genellikle engellenir. `play()` reddedilebilir ([MDN: Autoplay guide](https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay)). Bu yüzden ses yalnızca "Tamam" dokunuşunun işleyicisinde çalınır |

**Toplam en düşük platform (geçici):** Chrome/Edge 111, Firefox 114, Safari/iOS 16.4. Bu, Vite'ın varsayılan hedefidir. Daha eski cihazlar gerekiyorsa bkz. R-04 ve BA sorusu S-14.

**Geliştirme ortamı:** Node 22.12 veya üstü. Bu sürüm Vitest ve Playwright koşullarının kesişimidir.

## 4. Bileşenler ve sorumlulukları

| Kod | Bileşen | Sorumluluk | Gereksinimler |
|---|---|---|---|
| K-01 | **Alıştırma motoru** (`engine/`) | Saf fonksiyonlardan oluşur, DOM ve depolamaya erişmez. Rastgele sayı üreteci parametre olarak alınır. Görevleri: soru havuzu ve oturum planı (bölüm 6.1–6.3), oturum içi tekrar kuyruğu (6.4), cevap değerlendirme ve öğrenme durumu güncellemesi (6.5), tablo yıldızı kontrolü (6.6), oturum özeti verisi | FR-01, FR-04 – FR-10, FR-12, FR-13, FR-14 (süre dolması = yanlış) |
| K-02 | **Depolama bağdaştırıcısı** (`storage/`) | Sürümlü JSON'u okur ve yazar. Yükleme sırasında şemayı doğrular. Bozuk veriyi yedek anahtara taşır. `SecurityError` ve `QuotaExceededError` hatalarını yakalar. Sıfırlamayı yapar | FR-15, FR-16, FR-17, NFR-10 |
| K-03 | **Uygulama denetleyicisi** | Ekran geçişlerini yönetir. Cevap onayında sırayla motor güncellemesini, kaydı ve geri bildirim çizimini çağırır. Zamanlayıcıyı ve tuş takımı/klavye girdisini yönetir | FR-04, FR-07, FR-14, NFR-03, NFR-10 |
| K-04 | **Ekranlar** | Ana ekran: tablo düğmeleri 1–10, sıra seçimi, uzunluk seçimi; hiç tablo seçili değilse "Başla" pasiftir. İnceleme: 10 satır ve tekrarlı toplama. Alıştırma: soru, tuş takımı, geri bildirim, geri sayım. Özet. İlerleme: 10×10 ızgara ve yıldızlar. Ayarlar: süre, ses (koşullu), sıfırlama | FR-01 – FR-04, FR-07, FR-11 – FR-14, FR-16, FR-17, NFR-02, NFR-06, NFR-09 |
| K-05 | **Metin tablosu** (`i18n/tr.ts`) | Tüm arayüz metinleri tek dosyadadır. Geri bildirim metinleri suçlayıcı değildir (örn. "Neredeyse! 6×7 = 42") | NFR-01, FR-07, NFR-09 |
| K-06 | **Görsel tasarım belirteçleri** (CSS değişkenleri) | Renk paleti, en küçük dokunma boyutu ve durum simgeleri (Yeni / Çalışılıyor / Öğrenildi; doğru / yanlış) | NFR-04, NFR-05, NFR-06, FR-11 |
| K-07 | **Statik kabuk** | `index.html`, CSP `<meta>` etiketi, `manifest.json` ve ikonlar. Service worker yalnızca koşullu eklenir | NFR-07, NFR-08, FR-15 (R-01 azaltımı) |
| K-08 | **Yetişkin doğrulama** | Sıfırlama öncesinde, 10×10 tablosu dışında kalan iki basamaklı × iki basamaklı bir çarpma sorusu üretir (örn. 14×12). Cevap tuş takımıyla girilir. Ardından ayrı bir "Evet, sıfırla" onayı istenir. İptalde hiçbir veri değişmez (geçici; bkz. S-10) | FR-16, US-07 |

## 5. Veri modeli

### 5.1 Kalıcı veri

Kalıcı veri `localStorage` içinde tek bir anahtarda tutulur: `carpim-tablosu:durum`.

```ts
type Fakt = `${number}x${number}`;   // "7x8": satır a=7, sütun b=8; a,b ∈ 1..10

interface SoruDurumu {
  goruldu: boolean;       // en az bir kez cevaplandı mı
  seri: 0 | 1 | 2 | 3;    // art arda doğru cevap sayısı (3'te sabitlenir)
}

interface KaliciDurum {
  surum: 1;                               // şema sürümü
  sorular: Record<Fakt, SoruDurumu>;      // 100 kayıt (FR-10, FR-11)
  yildizlar: number[];                    // yıldız almış tablolar (FR-13)
  ayarlar: {
    soruSuresiSn: null | 10 | 7 | 5;      // FR-14; varsayılan null = kapalı
    ses?: boolean;                        // yalnızca ses efekti varsa (FR-17)
  };
}
```

- **Durum türetilir, saklanmaz (FR-10).**
  - `goruldu = false` ise "Yeni".
  - `seri >= 3` ise "Öğrenildi".
  - Diğer durumlarda "Çalışılıyor".
  - Bu yöntem FR-10'daki tanımla birebir aynıdır: "Öğrenildi" = son 3 cevap doğru. Sayaç dışında ayrıca bir durum alanı tutulmaz, bu yüzden ikisi çelişemez.
- **Boyut:** 100 kayıt yaklaşık birkaç KB tutar (çıkarım). 5 MiB sınırının çok altındadır.
- **İlk açılış:** Anahtar yoksa tüm sorular `{goruldu:false, seri:0}`, yıldız yok, süre kapalı (US-06).
- **Sıfırlama (FR-16, geçici):** `sorular` ve `yildizlar` ilk açılış değerine döner. `ayarlar` korunur, çünkü FR-16 "ilerleme" diyor. Yeni belge tek bir `setItem` ile yazılır (bkz. S-10).
- **Bozuk veya bilinmeyen sürüm:** JSON ayrıştırma ya da şema doğrulama başarısız olursa ham metin `carpim-tablosu:bozuk-yedek` anahtarına kopyalanır ve ilk açılış durumuyla devam edilir. Amaç, verinin sessizce yok edilmemesidir (NFR-10).

### 5.2 Oturum verisi (yalnızca bellekte)

```ts
interface OturumSorusu { a: number; b: number; tekrarMi: boolean; }

interface Oturum {
  plan: OturumSorusu[];          // sıradaki sorular (bölüm 6)
  sorulanlar: { soru: OturumSorusu; cevap: number | null; dogru: boolean }[];
  buOturumdaOgrenilen: Fakt[];   // FR-12
  yeniYildizlar: number[];       // FR-13 tebriği
}
```

- Oturum kalıcı olarak saklanmaz. Yarıda bırakılan oturum devam ettirilmez; bunu isteyen bir gereksinim yok.
- Her cevabın ilerlemeye etkisi ise cevap anında kalıcı veriye işlenir (FR-15).
- Soru sırası (FR-05) ve oturum uzunluğu (FR-06) her açılışta varsayılan değerle başlar: karışık, 10 soru. Bunlar kaydedilmez (geçici; bkz. S-12).

## 6. Algoritmalar ve seçim gerekçeleri

Tanımlar:
- `T`: seçili tablolar.
- `N ∈ {5, 10, 20}`: oturum uzunluğu.
- `P = {(a,b) : a ∈ T, b = 1..10}`: soru havuzu, büyüklüğü `|P| = 10·|T|`.
- `(a,b)` ve `(b,a)` ayrı sorulardır. FR-11'deki 10×10 ızgara ve FR-13'teki "bir tablonun 10 sorusu" tanımı bu yorumu gerektiriyor (geçici; bkz. S-08).

### 6.1 Soru sırası (FR-05)

- **Sıralı mod:** `P`, önce `a` sonra `b` artan sırada dizilir ve ilk `N` soru alınır. `N > |P|` olduğunda (tek tablo ve 20 soru) dizi baştan tekrarlanır.
  - Geçici yorum: çok tablolu seçimde `N < |P|` ise yalnızca ilk tablo(lar) sorulur. Örneğin 3'ler ve 4'ler seçiliyken 10 soruda yalnızca 3'ler gelir. Bu, US-01'in "3×n ve 4×n" beklentisiyle gerilim yaratıyor (bkz. S-03).
- **Karışık mod:** Bölüm 6.2'deki planlama.

### 6.2 Kritik karar: Karışık modda zayıf sorulara öncelik (FR-09)

Seçim ölçütleri:

| # | Ölçüt | Bağlı gereksinim / risk |
|---|---|---|
| Ö1 | Öğrenilmemiş soru varsa soruların en az %50'si bunlardan gelmeli | FR-09, US-04 |
| Ö2 | Yalnızca seçili tablolardan soru gelmeli | FR-01, US-01 |
| Ö3 | Aynı soru art arda gelmemeli | FR-05 |
| Ö4 | Kural deterministik olarak test edilebilmeli | Tekrarlanabilirlik, bölüm 9 |
| Ö5 | Sadelik; BA'da tanımlı olmayan öğrenme modeli eklenmemeli | Kapsam kuralı, BA Varsayım 5–6 |

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. Önce zayıf: zayıf kümeden en fazla N, kalan öğrenilmişten; sonra karıştırma** | Zayıf soru sayısı N/2 veya üstündeyse her zaman sağlanır | Sağlanır | Karıştırma sonrası düzeltme adımı | Tohumlu RNG ile birebir | Tek kural |
| B. Sabit kota: tam `⌈N/2⌉` zayıf + `⌊N/2⌋` öğrenilmiş | Sağlanır | Sağlanır | Aynı | Birebir | Sade. Ancak zayıf soru çokken de yarısını bilinen sorulara harcar; bu, persona Can'ın "bildiğim soruları çözmekten sıkılıyorum" sorununu sürdürür |
| C. Ağırlıklı rastgele seçim (örn. zayıf ağırlık 3, öğrenilmiş ağırlık 1) | %50 yalnızca beklenen değer olarak tutar, tek oturumda garanti yok | Sağlanır | Ek kontrol | Olasılıksal, test daha zor | Ek parametre |
| D. Aralıklı tekrar algoritması (Leitner, SM-2 vb.) | Dolaylı | Sağlanır | Ek kontrol | Zor | BA'da tanımlı olmayan bir öğrenme modeli ekler. BA Varsayım 5–6 eşikleri ürün kararı olarak sabitlemiş. Elendi |

**Karar: A.** Ö1'i ölçülebilir biçimde garanti eder ve Ö5'e en uygun seçenektir. Adımlar:

1. `Z = {p ∈ P : durum(p) ≠ Öğrenildi}`: hem "Yeni" hem "Çalışılıyor" soruları. FR-09'daki "geçmişte yanlış cevaplanan veya henüz öğrenilmemiş" ifadesinin geçici yorumudur (S-04). `O = P \ Z`.
2. `Z` ve `O` ayrı ayrı Fisher–Yates ile karıştırılır.
3. Taban liste = `Z`'den `min(N, |Z|)` soru, ardından `O`'dan gereken kadar soru.
4. `|P| < N` ise (yalnızca tek tablo ve N=20) eksik kısım, yeniden karıştırılmış `P` turlarıyla doldurulur.
5. Taban liste bir kez daha Fisher–Yates ile karıştırılır. Bu, zayıf soruların hep başta toplanmasını önler.
6. **Bitişiklik düzeltmesi (FR-05):** Liste baştan taranır. `liste[i] = liste[i−1]` ise, `liste[i−1]`'den farklı ve yer değiştirmesi yeni bir bitişiklik yaratmayan ilk `j > i` bulunur ve iki eleman yer değiştirir. Bitişik tekrar yalnızca 4. adımdaki tekrar turlarında oluşabilir.

**Bilinen uç durum.** `0 < |Z| < N/2` ise, tekrarsız seçimle %50 karşılanamaz. Bu durumda karar A tüm `Z`'yi alır, yani mümkün olan en yüksek oranı. Bu bir gereksinim yorumudur (S-04).

### 6.3 Karıştırma yöntemi

Modern Fisher–Yates (Durstenfeld) karıştırması kullanılır:
- Algoritma O(n) sürede çalışır.
- Rastgele kaynak yansızsa "every permutation is equally likely" ([Wikipedia: Fisher–Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)).
- Uygulamada `j ∈ [0, i]` aralığı kullanılır. Aynı kaynağa göre `j < i` seçmek, algoritmayı hiçbir elemanın yerinde kalamadığı Sattolo algoritmasına çevirir. Birim testinde bu hata ayrıca kontrol edilir.
- RNG motor parametresidir: üretimde `Math.random`, testte tohumlu üreteç. Seçim güvenlik amaçlı olmadığı için kriptografik üreteç gerekmez.

### 6.4 Kritik karar: Oturum içi tekrar (FR-08)

Seçim ölçütleri: FR-08 (en az 1 tekrar ve en az 2 soru sonra), FR-05 (art arda aynı soru yok), US-01 (tekrarlar N'ye sayılmaz), NFR-09 (oturum sonsuza uzamamalı ve hata cezalandırılmamalı).

| Seçenek | FR-08 aralığı | FR-05 | US-01 | NFR-09 |
|---|---|---|---|---|
| **A. Yanlışta tekrar kopyasını kalan planın 3. sırasına ekleme (araya 2 soru girer); tekrarda yeniden yanlışa yeni kopya yok** | Plan uzunsa sağlanır | Ekleme noktasında komşu kontrolü | Kopyalar `tekrarMi=true` ile N dışında kalır | Tekrarlar sonlu: en çok N kopya |
| B. Tüm tekrarları oturum sonunda toplama | Son sorularda aralık 2'nin altına düşebilir | Ardışık tekrarlar aynı soru olabilir | Sağlanır | Sonlu |
| C. Her yanlışta yeni kopya (tekrarda yanlışsa yine) | Sağlanır | Kontrol gerekir | Sağlanır | Oturum sınırsız uzayabilir. Elendi |

**Karar: A (geçici).** Adımlar:

1. Taban sorudaki yanlış cevap (süre dolması dahil, FR-14) bir kopya üretir: `{a, b, tekrarMi: true}`.
2. Kopya, kalan planda 2 sorudan sonraya konur.
3. Ekleme noktasının bir önceki veya bir sonraki elemanı aynı soruysa nokta bir ileri kaydırılır.
4. Kalan plan 2 sorudan kısaysa kopya planın sonuna eklenir.

**Bilinen çatışma.** Oturumun son sorusu yanlış cevaplanırsa, "en az 1 kez tekrar" ile "en az 2 soru sonra" aynı anda sağlanamaz. Karar A tekrarı yine sorar ("en az 1 kez" öncelikli) ve bu durumu S-05 ile BA'ya taşır.

### 6.5 Öğrenme durumu (FR-10)

| Seçenek | FR-10'a uygunluk | Sadelik |
|---|---|---|
| **A. Ardışık doğru sayacı (0–3, 3'te sabit) ve `goruldu` bayrağı** | "Son 3 cevap doğru" ile eşdeğer: son 3 cevap doğru ise ve ancak o zaman sayaç ≥ 3 olur | En sade |
| B. Son 3 cevabın geçmişi (dizi) | Eşdeğer | Fazladan veri tutar, ek bir yararı yok |

**Karar: A.** Kurallar:
- Doğru cevap: `seri = min(seri+1, 3)`, `goruldu = true`.
- Yanlış cevap veya süre dolması: `seri = 0`, `goruldu = true`. Durum "Çalışılıyor"a döner (FR-10, US-04).
- Tekrar kopyalarına verilen cevaplar da sayılır, çünkü FR-10 "o soruya verilen son 3 cevap" diyor.
- "Öğrenildi"ye geçiş anında soru `buOturumdaOgrenilen` listesine eklenir (FR-12).

### 6.6 Tablo yıldızı (FR-13)

- Her cevaptan sonra yalnızca ilgili `a` satırı kontrol edilir. `a×1 … a×10` sorularının tamamı "Öğrenildi" ise ve `a` henüz `yildizlar` içinde değilse, `a` listeye eklenir ve `yeniYildizlar` listesine yazılır.
- Tebrik oturum özetinde gösterilir. Böylece alıştırma akışı kesilmez (geçici; S-07).
- **Kalıcılık (geçici):** Yıldız, satırdaki bir soru sonradan "Çalışılıyor"a dönse de korunur. Yalnızca FR-16 sıfırlamasıyla silinir. FR-13'teki "kalıcı" ifadesinin yorumudur (S-07).

### 6.7 Süre sınırı (FR-14)

- Ayar açıksa her soru gösterildiğinde bir bitiş zamanı (`performance.now() + süre`) belirlenir.
- Geri sayım hem sayı hem çubukla gösterilir (NFR-06).
- Süre dolunca cevap `null` ve yanlış olarak işlenir. Doğru cevap gösterilir ve sonraki soruya geçilir (US-06).
- **Geçici:** Sayfa arka plana geçerse (`visibilitychange`) geri sayım duraklatılır. Amaç, çocuğun görmediği bir sürede soruyu kaybetmemesidir (NFR-09; S-13).

### 6.8 Tekrarlı toplama metni (FR-03)

- `a×b` ifadesi, BA örneğine birebir uyularak "a" sayısının `b` kez toplanmasıyla yazılır: `3×4 = 3+3+3+3 = 12`.
- `b = 10` için ifade 10 terimdir. Dar ekranlarda satır kaydırmalı gösterilir (R-09).

### 6.9 Oturum özeti (FR-12)

Özet şunları gösterir:
- Doğru sayısı / sorulan toplam soru. Geçici yorum: tekrar kopyaları da sayılır (S-06).
- Yanlış yapılan soruların tekrarsız listesi.
- `buOturumdaOgrenilen` listesi.
- Varsa yeni yıldızların tebriği.

## 7. Performans

- **Hedefler (NFR-03):** Onaydan geri bildirime ≤ 0,5 sn, sonraki soruya ≤ 1 sn.
- **Onay anındaki iş:**
  - Motor güncellemesi tek bir soru kaydına ve en çok bir satır kontrolüne (10 kayıt) dokunur.
  - Tekrar ekleme, kalan plan üzerinde O(N) iştir ve N ≤ 20'dir.
  - `JSON.stringify` ile `setItem` birkaç KB'lik bir belge yazar.
  - Bu sürelerin hedefin çok altında kalması beklenir (çıkarım). Bu bir kaynak iddiası değildir; aşağıdaki ölçümle doğrulanır.
- **Oturum planlama:** `|P| ≤ 100`; karıştırma ve seçim O(|P|). Yalnızca "Başla"da bir kez çalışır.
- **Ağ:** Alıştırma sırasında ağ isteği yoktur. Yanıt süresi bağlantıdan bağımsızdır.
- **Paket boyutu:** UI çatısı ve çalışma zamanı kütüphanesi yoktur. Yalnızca uygulama kodu ve CSS yüklenir.
- **Animasyon:** Geri bildirim animasyonu kullanılacaksa 0,5 sn bütçesini aşmaz. "Sonraki soru"ya geçiş, geri bildirim görünür kaldıktan sonra en geç 1 sn içinde gerçekleşir.
- **Ölçüm:** Playwright testleri, cevap onayından ilgili DOM öğesinin görünmesine kadar geçen süreyi `performance.now()` ile ölçer. Testler Chromium, WebKit ve Firefox'ta, mobil emülasyonla çalışır. Kesin sonuç hedef cihaz sınıfındaki gerçek cihazda alınır (BA Açık soru 1; S-01).

## 8. Güvenlik ve gizlilik

- **Kişisel veri yok (NFR-07):** Hiçbir ekranda ad, e-posta, konum vb. alan yoktur. Hesap yoktur. Saklanan veri yalnızca soru sayaçları, yıldızlar ve ayarlardır.
- **Ağ çıkışı yok (NFR-07, NFR-08):** `index.html` içinde şu CSP kullanılır:
  `default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self'; connect-src 'none'; object-src 'none'; base-uri 'self'`
  - `connect-src 'none'` ifadesi `fetch()`, `XMLHttpRequest`, `WebSocket`, `EventSource` ve `sendBeacon()` çağrılarını engeller ([MDN: connect-src](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/connect-src)).
  - Dış betik, reklam ağı, analitik, dış yazı tipi ve izleme pikseli yüklenemez.
  - `<meta>` ile verilen CSP bazı özellikleri desteklemez ([MDN: CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)). Barındırma HTTP başlığı destekliyorsa aynı politika başlıkla da verilir.
- **Barındırma:** HTTPS zorunludur. Ana ekrana ekleme ve service worker HTTPS ister (bkz. 3.5). Barındırma hizmetinde sunucu tarafı analitik ve çerez kapatılır (NFR-07).
- **Reklam ve satın alma yok (NFR-08):** Kodda ödeme veya reklam SDK'sı yoktur. E2E testi tüm ekranlarda kökendışı istek olmadığını doğrular.
- **Yetişkin doğrulama (FR-16):** Bir güvenlik denetimi değildir. Amacı, BA'nın tanımladığı gibi çocuğun yanlışlıkla sıfırlamasını önlemektir. Güçlü erişim denetimi gerekmez.
- **Yerel veri:** `localStorage` cihaza erişen herkes tarafından görülebilir. Veri kişisel olmadığı için ek şifreleme yapılmaz (NFR-07).
- **Yasal uyum:** Çocuklara yönelik mevzuat ve mağaza gereksinimleri BA Açık soru 12'de açık. Bu doküman yasal bir değerlendirme yapmaz.

## 9. Test stratejisi

| Katman | Araç | Kapsam | Gereksinimler |
|---|---|---|---|
| Birim (motor) | Vitest, tohumlu RNG | Aşağıda ayrıca listelenmiştir | FR-01, FR-05, FR-06, FR-08, FR-09, FR-10, FR-12, FR-13, FR-14 |
| Birim (depolama) | Vitest, sahte `Storage` | Aşağıda ayrıca listelenmiştir | FR-15, FR-16, FR-17, NFR-10 |
| Birim (metin ve tasarım) | Vitest | `tr.ts` içindeki yönerge metinlerinin her birinin ≤ 8 kelime olması. Palet çiftlerinin WCAG oranıyla hesaplanması (kontrast oranı `(L1+0,05)/(L2+0,05)`, sRGB göreli parlaklık tanımıyla; [W3C SC 1.4.3](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)). Normal metinde ≥ 4,5, büyük metinde ≥ 3. Oran yuvarlanmaz | NFR-01, NFR-04 |
| Uçtan uca | Playwright: Chromium, WebKit, Firefox, mobil emülasyon | Aşağıda ayrıca listelenmiştir | US-01 – US-07, NFR-02, NFR-03, NFR-05 – NFR-10 |
| Gerçek cihaz | El ile | Hedef cihaz sınıfında NFR-03 ölçümü; ana ekrana ekleme; Safari'de verinin korunması | NFR-03, FR-15 |
| Kullanılabilirlik | El ile | En az 5 çocukla; en az 4'ü yardımsız ve ≤ 3 dokunuşla oturum başlatmalı | NFR-02 |

**Birim testleri (motor):**
- 1 000 tohum üzerinde değişmezler kontrol edilir:
  - Yalnızca `T` tablolarından soru gelir.
  - Bitişik aynı soru yoktur.
  - `|Z| ≥ N/2` iken karışık modda zayıf soru oranı ≥ %50'dir.
  - Tekrar kopyası, plan yeterince uzunsa, yanlıştan en az 2 soru sonra gelir.
  - Taban soru sayısı N'dir.
- Seri sayacının geçişleri test edilir: 3 doğru sonrası "Öğrenildi", yanlış sonrası "Çalışılıyor".
- Yıldız koşulu test edilir.
- Süre dolması yanlış sayılmalıdır.
- Fisher–Yates'te her elemanın yerinde kalabildiği kontrol edilir (Sattolo hatasının yokluğu).

**Birim testleri (depolama):**
- Kaydedip yeniden yükleme.
- Bozuk JSON ve bilinmeyen sürümde yedekleme.
- `SecurityError` ve `QuotaExceededError` yakalama.
- Sıfırlamanın ayarları koruması.

**Uçtan uca testler:**
- US kabul kriterleri birebir senaryolaştırılır.
- Dokunuş sayısı ölçülür (NFR-02).
- Onay → geri bildirim ≤ 0,5 sn, onay → sonraki soru ≤ 1 sn (NFR-03).
- Tüm etkileşimli öğelerin sınır kutusu ≥ 24×24 CSS piksel (NFR-05; geçici hedef bkz. S-11).
- Geri bildirim ve ızgara hücrelerinde simge veya metin bulunması (NFR-06).
- Ağ dinleyicisiyle kökendışı istek olmaması ve reklam/satın alma öğesi bulunmaması (NFR-07, NFR-08).
- Yanlış cevapta "oyun bitti" olmaması ve oturumun sürmesi (NFR-09).
- Çökme benzetimi (NFR-10): cevap onayından hemen sonra tarayıcı bağlamı kapatılır, yeniden açılır ve son cevabın işlendiği doğrulanır.

## 10. Riskler

| Kod | Risk | Etki | Azaltım |
|---|---|---|---|
| R-01 | Safari'de izleme önleme açıkken, 7 günlük tarayıcı kullanımında etkileşim olmayan kökenin betikle yazılmış verisi silinir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Çocuk bir hafta kullanmazsa ilerleme kaybolabilir. Bu, tasarımın FR-15 ile çelişebileceği noktadır | FR-15 | WebKit'e göre ana ekrana eklenen web uygulamaları "are not part of Safari and thus have their own counter of days of use", ve "We do not expect the first-party in such a web application to have its website data deleted" ([WebKit blog](https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/)). Manifest bu nedenle eklenir. Kullanıcıya "ana ekrana ekleyin" bilgisi verilip verilmeyeceği yeni bir gereksinim olur (S-15). Platform B'ye dönerse risk ortadan kalkar |
| R-02 | Gizli gezintide veri son gizli sekme kapanınca silinir ([MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)) | FR-15 | Teknik çözüm yok. Uyarı gösterilmesi yeni gereksinim olur (S-15) |
| R-03 | Kullanıcı kalıcı veriyi engellemişse `localStorage` `SecurityError` verir | FR-15, NFR-10 | Bağdaştırıcı hatayı yakalar. Uygulama bellekte çalışmaya devam eder, ilerleme kalıcı olmaz (geçici; S-15) |
| R-04 | Vite'ın varsayılan hedefi Safari 16.4, Chrome/Edge 111, Firefox 114. Daha eski tabletler ve telefonlar desteklenmez | BA Açık soru 1 | Vite belgesine göre `@vitejs/plugin-legacy` ile eski tarayıcı desteği mümkün. Hedef cihaz cevabına göre karar verilir (S-14) |
| R-05 | Platform kararı geçici. Cevap "yalnızca mağaza uygulaması" olursa dağıtım değişir | Tüm FR'ler | Motor ve veri modeli platformdan bağımsız |
| R-06 | FR-08 son soru çatışması (bkz. 6.4) | FR-08 | Geçici yorum ve S-05 |
| R-07 | FR-09 oranı, `|Z| < N/2` olduğunda tekrarsız seçimle karşılanamaz | FR-09 | Geçici yorum ve S-04 |
| R-08 | Service worker olmadan uygulama ilk açılışta ağa ihtiyaç duyar. Çevrimdışıyken açılmayabilir | BA Açık soru 2 | Cevap "evet" ise K-07'ye service worker eklenir |
| R-09 | FR-03'te `a×10` ifadesi 10 terimdir ve dar ekranda taşabilir | FR-03, NFR-05 | Satır kaydırma ve E2E görsel kontrol |
| R-10 | Yetişkin doğrulaması, ileri düzey bir çocuk tarafından aşılabilir | FR-16 | Amaç yalnızca yanlışlıkla sıfırlamayı önlemek. Ayrıca bir onay adımı daha var (S-10) |
| R-11 | Çökme anında `setItem`'in yarım yazma davranışı okunan belgelerde tanımlı değil (doğrulanamadı) | NFR-10 | Tek anahtar ve tek çağrı. Yüklemede şema doğrulaması ve bozuk veri yedeği. E2E çökme benzetimi |
| R-12 | Vite tip denetimi yapmaz ([Vite: Features](https://vite.dev/guide/features.html)). Tip hataları derlemeden geçebilir | Kod kalitesi (tüm FR) | Derleme betiğinde `tsc --noEmit` çalıştırılır |
| R-13 | Emülasyon testleri gerçek cihaz performansını temsil etmeyebilir (çıkarım) | NFR-03 | Gerçek cihazda el ile ölçüm |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | 1–10 tablo seçimi; seçim yoksa başlatılamaz | K-04 Ana ekran ("Başla" pasif); K-01 havuz `P` | 4, 6 |
| FR-02 | Seçilen tablonun 10 satırını görme | K-04 İnceleme ekranı | 4 |
| FR-03 | Tekrarlı toplama karşılığı | K-04 İnceleme; 6.8 ifade üretimi | 6.8 |
| FR-04 | "a × b = ?" sorusu; yalnızca rakam, en fazla 3 hane | Karar 3.3 özel tuş takımı; K-03 girdi | 3.3, 4 |
| FR-05 | Sıralı / karışık; varsayılan karışık; art arda aynı soru yok | 6.1 sıralı plan; 6.2 adım 6 bitişiklik düzeltmesi; 6.4 komşu kontrolü | 6.1, 6.2, 6.4 |
| FR-06 | 5/10/20 soru; varsayılan 10 | K-04 Ana ekran; K-01 taban liste uzunluğu N | 5.2, 6.2 |
| FR-07 | ≤ 0,5 sn geri bildirim; doğru cevabı gösterme; suçlayıcı olmayan metin | K-03 onay akışı; K-05 metinler; 7 Performans | 4, 7 |
| FR-08 | Yanlış soru oturumda en az 1 kez, en az 2 soru sonra tekrar | Karar 6.4-A tekrar kuyruğu | 6.4 |
| FR-09 | Karışık modda öğrenilmemişlere en az %50 | Karar 6.2-A önce zayıf | 6.2 |
| FR-10 | Yeni / Çalışılıyor / Öğrenildi; son 3 cevap doğru | Karar 6.5-A seri sayacı; 5.1 türetilen durum | 5.1, 6.5 |
| FR-11 | 10×10 ilerleme ızgarası; renk ve simge | K-04 İlerleme; K-06 belirteçler | 4 |
| FR-12 | Oturum sonu özeti | 6.9; K-01 oturum verisi | 5.2, 6.9 |
| FR-13 | Tablo tamamen öğrenilince tebrik ve kalıcı yıldız | 6.6; 5.1 `yildizlar` | 5.1, 6.6 |
| FR-14 | İsteğe bağlı süre 10/7/5 sn; varsayılan kapalı; dolunca yanlış | 6.7; 5.1 `soruSuresiSn` | 5.1, 6.7 |
| FR-15 | İlerlemenin kalıcılığı; yarıda kalan oturumun işlenmesi | Karar 3.2-A `localStorage`; her onayda yazma; manifest (R-01) | 3.2, 5.1, 10 |
| FR-16 | Yetişkin doğrulamalı ve onaylı sıfırlama | K-08; K-02 sıfırlama | 4, 5.1 |
| FR-17 | Ses efektleri varsa kapatılabilir ve ayar korunur | Koşullu `ayarlar.ses`; ses yalnızca dokunuş işleyicisinde çalınır | 3.5, 5.1 |
| NFR-01 | Türkçe arayüz; yönergeler ≤ 8 kelime | K-05 metin tablosu; birim testi kelime sayımı | 4, 9 |
| NFR-02 | ≤ 3 dokunuşla oturum başlatma; 5 çocukta en az 4 başarı | Ana ekran tasarımı (2 dokunuş); kullanılabilirlik testi | 2, 9 |
| NFR-03 | Geri bildirim ≤ 0,5 sn; sonraki soru ≤ 1 sn | Yerel mantık; ağsız alıştırma; Playwright ve gerçek cihaz ölçümü | 7, 9 |
| NFR-04 | Kontrast 4,5:1 / 3:1 | K-06 palet; birim testi kontrast hesabı | 4, 9 |
| NFR-05 | Dokunma alanı ≥ 24×24 CSS px | Karar 3.3 tuş boyutu kontrolü; K-06; E2E sınır kutusu testi | 3.3, 9 |
| NFR-06 | Durumlar yalnızca renge dayanmaz | K-06 simgeler; geri sayımda sayı; E2E kontrol | 4, 6.7, 9 |
| NFR-07 | Kişisel veri istenmez ve toplanmaz; hesap yok | Arka uçsuz mimari; CSP `connect-src 'none'`; veri modeli | 2, 5.1, 8 |
| NFR-08 | Reklam ve uygulama içi satın alma yok | Dış betik engelleyen CSP; SDK yok; E2E kontrol | 8, 9 |
| NFR-09 | Ceza, olumsuz ses veya "oyun bitti" yok; hata oturumu kesmez | K-05 metinler; 6.4 sonlu tekrar; 6.7 arka planda duraklatma | 4, 6.4, 6.7 |
| NFR-10 | Çökmede son onaylanan cevaba kadar ilerleme korunur | 3.2 eşzamanlı yazma sırası; bozuk veri yedeği; çökme testi | 3.2, 5.1, 9 |

## 12. BA'ya geri sorular

| Kod | Soru | İlgili ID | Geçici karar |
|---|---|---|---|
| S-01 | Hedef cihazlar hangileri: telefon, tablet, bilgisayar? Uygulama mağazası şart mı? (BA Açık soru 1) | NFR-03, NFR-05 | Statik web uygulaması |
| S-02 | Uygulama internetsiz açılabilmeli mi? (BA Açık soru 2) | FR-15 | Service worker yok; mimari hazır |
| S-03 | Sıralı modda birden çok tablo seçiliyken `N < |P|` ise ne beklenir? Yalnızca ilk tablonun soruları mı gelir, yoksa her tablodan sırayla mı? US-01'deki "3×n ve 4×n" ifadesi her ikisinin de gelmesini mi gerektiriyor? Tek tablo ve 20 soruda tablo baştan tekrar mı eder? | FR-05, FR-06, US-01 | İlk N soru; gerekirse baştan tekrar |
| S-04 | FR-09'daki "henüz öğrenilmemiş" ifadesi "Yeni" soruları da kapsıyor mu? Zayıf soru sayısı N/2'den azsa %50 için aynı soru tekrarlanmalı mı, yoksa mevcut zayıf soruların tamamını sormak yeterli mi? | FR-09 | Yeni ve Çalışılıyor dahil; tekrarsız, tüm zayıf sorular |
| S-05 | FR-08'de "en az 2 soru sonra" ifadesi "arada en az 2 başka soru" mu demek? Son soru yanlışsa tekrar hemen mi sorulsun, yoksa tekrar atlansın mı? Tekrarda yine yanlış yapılırsa ikinci bir tekrar olmalı mı? | FR-08 | Araya 2 soru; son soruda yine sorulur; tekrardan yeni tekrar yok |
| S-06 | FR-12'deki "toplam soru" tekrar sorularını da içeriyor mu? | FR-12 | İçeriyor |
| S-07 | Yıldızlı bir tablodaki soru sonradan "Çalışılıyor"a dönerse yıldız korunmalı mı? Tebrik anında mı, oturum sonunda mı gösterilmeli? | FR-13 | Korunur; tebrik oturum sonunda |
| S-08 | 3×4 ile 4×3 ayrı sorular mı? 10×10 ızgara ayrı olduklarını ima ediyor | FR-10, FR-11, FR-13 | Ayrı |
| S-09 | Varsayılan tablo seçimi ne olmalı: hiçbiri mi, 1–5 mi? (BA Açık soru 5) | FR-01, NFR-02 | Hiçbiri seçili değil |
| S-10 | Yetişkin doğrulaması için iki basamaklı × iki basamaklı bir soru yeterli mi? Sıfırlama ayarları da sıfırlamalı mı? | FR-16 | Evet yeterli; ayarlar korunur |
| S-11 | Tuş takımı için kesin en küçük boyut nedir? (BA Açık soru 11) W3C, önemli denetimlerde 44×44 CSS px eşikli 2.5.5'i hedeflemeyi öneriyor ([W3C SC 2.5.8](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)) | NFR-05 | Tuş takımı 44×44, diğer öğeler en az 24×24 CSS px |
| S-12 | Soru sırası ve oturum uzunluğu seçimleri sonraki açılışta hatırlanmalı mı? | FR-05, FR-06 | Hatırlanmaz; varsayılana döner |
| S-13 | Süreli modda uygulama arka plana geçerse süre duraklamalı mı? | FR-14, NFR-09 | Duraklar |
| S-14 | Desteklenmesi gereken en eski cihaz ve tarayıcı sürümleri neler? Varsayılan hedef Safari/iOS 16.4, Chrome/Edge 111, Firefox 114 | NFR-03, R-04 | Varsayılan hedef |
| S-15 | Verinin korunamadığı durumlarda (gizli gezinti, depolamanın engellenmesi, Safari'de ana ekrana eklenmemiş kullanım) kullanıcıya uyarı veya "ana ekrana ekleyin" yönlendirmesi istenir mi? İstenirse yeni bir FR gerekir | FR-15, R-01 – R-03 | Uyarı yok; depolama engelliyse yalnızca bellekte çalışılır |
| S-16 | Ses efekti olacak mı? Hiçbir FR ses efekti tanımlamıyor; FR-17 koşullu | FR-17, NFR-09 | Ses efekti yok. Eklenirse yalnızca olumlu/nötr sesler ve kapatma ayarı |

## 13. Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | MDN: Storage quotas and eviction criteria | https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria | Web Storage sınırı: "Browsers can store up to 5 MiB of local storage, and 5 MiB of session storage per origin". Aynı sayfada 10 MiB değeri de geçiyor, sayfa farkı açıklamıyor. Sınır aşılınca `QuotaExceededError`. Safari: "If an origin has no user interaction, such as click or tap, in the last seven days of browser use, its data created from script will be deleted." |
| 2 | MDN: Window.localStorage | https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage | "the stored data is saved across browser sessions". Gizli gezintide son gizli sekme kapanınca silinme. `SecurityError` koşulları: `file:`/`data:` kökeni, kalıcı verinin kullanıcı tarafından engellenmesi |
| 3 | MDN: Using Service Workers | https://developer.mozilla.org/en-US/docs/Web/API/Service_Worker_API/Using_Service_Workers | Web Storage API'nin eşzamanlı olduğu ("is synchronous, so not allowed in service workers"). HTTPS zorunluluğu. Çevrimdışı önbellek. Güncelleme ve gizli gezinti kısıtları |
| 4 | WebKit: Full Third-Party Cookie Blocking and More | https://webkit.org/blog/10218/full-third-party-cookie-blocking-and-more/ | 7 günlük silme kuralının betikle yazılabilir tüm depolamayı kapsaması. Ana ekrana eklenen web uygulamaları için: "are not part of Safari and thus have their own counter of days of use" ve "We do not expect the first-party in such a web application to have its website data deleted." |
| 5 | MDN: Making PWAs installable | https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps/Guides/Making_PWAs_installable | Kurulabilirlik için HTTPS ve manifest alanları. "On iOS 16.4 and later, PWAs can be installed from the Share menu in Safari, Chrome, Edge, Firefox, and Orion". iOS'ta `beforeinstallprompt` desteklenmez |
| 6 | Vite: Building for Production | https://vite.dev/guide/build.html | Varsayılan hedef Chrome/Edge 111, Firefox 114, Safari 16.4. "Vite only handles syntax transforms and does not cover polyfills". `@vitejs/plugin-legacy` ile eski tarayıcı desteği |
| 7 | Vite: Features | https://vite.dev/guide/features.html | "Vite only performs transpilation on `.ts` files and does NOT perform type checking". `isolatedModules: true` gerekliliği |
| 8 | TypeScript: Download | https://www.typescriptlang.org/download/ | npm paketi `typescript`, geliştirme bağımlılığı olarak kurulur. Çalıştırmak için Node.js gerekir; sürüm belirtilmemiş |
| 9 | Vitest: Getting Started | https://vitest.dev/guide/ | "Vitest requires Vite >=v6.4.0 and Node >=v22.12.0". Test dosya adı kuralı |
| 10 | Playwright: Installation | https://playwright.dev/docs/intro | Node 22.x/24.x/26.x ve işletim sistemi gereksinimleri. Chromium, WebKit, Firefox desteği. Android Chrome ve Mobile Safari emülasyonu |
| 11 | MDN: inputmode | https://developer.mozilla.org/en-US/docs/Web/HTML/Reference/Global_attributes/inputmode | `inputmode` bir ipucudur: "doesn't cause any validity requirements to be enforced on input". "Devices may or may not show a minus key" |
| 12 | Wikipedia: Fisher–Yates shuffle | https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle | Modern algoritma O(n). "every permutation is equally likely" (yansız rastgele kaynak koşuluyla). Aralık hatasının Sattolo algoritmasına dönüşmesi |
| 13 | MDN: Autoplay guide | https://developer.mozilla.org/en-US/docs/Web/Media/Guides/Autoplay | Betikle başlatılan sesin kullanıcı etkileşimi olmadan genellikle engellenmesi; tıklama veya dokunuştan sonra izin verilmesi |
| 14 | MDN: CSP connect-src | https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers/Content-Security-Policy/connect-src | `connect-src` kapsamı: `fetch`, XHR, WebSocket, EventSource, `sendBeacon`. `'none'`: "No resources of this type may be loaded." |
| 15 | MDN: Content Security Policy (CSP) | https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP | `<meta>` ile CSP: istemci tarafında çalışan, yalnızca statik kaynaklı tek sayfalık uygulamalar için uygun, ancak "this option does not support all CSP features". Rapor modu `<meta>` ile verilemez |
| 16 | W3C: Understanding SC 1.4.3 Contrast (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html | 4,5:1 ve 3:1 eşikleri. Kontrast oranı `(L1 + 0.05) / (L2 + 0.05)`. sRGB göreli parlaklık formülü. Büyük metin: 18 pt veya 14 pt kalın. Oranlar yuvarlanmaz |
| 17 | W3C: Understanding SC 2.5.8 Target Size (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | "at least 24 by 24 CSS pixels". 2.5.5 "uses the larger 44 by 44 CSS pixel measure"; önemli denetimlerde bu daha katı eşiğin hedeflenmesi önerisi |