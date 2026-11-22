# Çarpım Tablosu Öğrenme Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v1
- **Girdi:** /home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v1-run2.md

---

## 1. Özet

Bu doküman, BA analiz dokümanındaki 17 fonksiyonel gereksinimi (FR-01 – FR-17) ve 10 fonksiyonel olmayan gereksinimi (NFR-01 – NFR-10) karşılayan teknik çözümü tanımlar.

Ana kararlar:

1. **Tarayıcıda çalışan, sunucusu olmayan tek sayfalı bir web uygulaması.** Bütün mantık ve veri cihazda kalır. Sunucu yalnızca statik dosyaları verir. Böylece kişisel veri toplanmaz (NFR-07), reklam ve satın alma altyapısı gerekmez (NFR-08). Uygulama telefon, tablet ve bilgisayar tarayıcısında aynı kodla çalışır. Platform kararı BA'nın açık sorusu 1'e bağlı olduğu için bu seçim, cevaba göre yeniden değerlendirilecek bir karar olarak işaretlenmiştir.
2. **TypeScript, UI framework'ü yok, derleme Vite ile.** Ürün altı ekranlı ve küçük. Ek bir framework'ün getireceği bağımlılık ve karmaşıklık bu ölçek için gereksiz.
3. **Veriler `localStorage` içinde, tek bir JSON kaydında tutulur.** Kayıt her cevaptan sonra, geri bildirim gösterilmeden önce eşzamanlı olarak yazılır (FR-15, NFR-10).
4. **Soru planlayıcı.** Fisher–Yates karıştırması kullanılır. Karışık modda zayıf sorular oturumun en az yarısını oluşturur (FR-09) ve aynı soru art arda gelmez (FR-05). Yanlış cevaplanan soru, arada en az 2 soru olacak biçimde kuyruğa yeniden eklenir (FR-08).
5. **Soru bazında öğrenme durumu.** Her soru için yalnızca "ardışık doğru sayısı" saklanır, durum bundan türetilir (FR-10).

En büyük teknik risk şu: Safari, kullanıcının 7 gün etkileşim kurmadığı sitelerin betikle oluşturulmuş verisini siliyor. Bu, ilerlemenin korunmasını (FR-15) tehdit ediyor (bkz. Bölüm 10, R1). Ayrıca BA dokümanındaki birkaç belirsizlik (FR-08 oturum sonu davranışı, sıralı mod ile oturum uzunluğu ilişkisi, FR-12'deki "toplam soru" tanımı vb.) Bölüm 12'de soruldu. Cevap gelene kadar uygulanacak geçici teknik davranışlar ilgili yerlerde **"geçici"** diye işaretlendi.

## 2. Mimari genel bakış

```
┌──────────────────────────── Tarayıcı (tek origin) ────────────────────────────┐
│                                                                               │
│  Ekranlar (UI)                                                                │
│  E1 Ana / Tablo seçimi   E2 Tablo inceleme   E3 Alıştırma   E4 Oturum özeti   │
│  E5 İlerleme ızgarası    E6 Ayarlar          + Tuş takımı, Yetişkin kapısı    │
│            │                         │                                        │
│            ▼                         ▼                                        │
│  Oturum motoru ──► Soru planlayıcı   Zamanlayıcı (isteğe bağlı)               │
│            │                                                                  │
│            ▼                                                                  │
│  İlerleme servisi (durum makinesi, yıldızlar)                                 │
│            │                                                                  │
│            ▼                                                                  │
│  Depolama adaptörü ──► localStorage (tek JSON kaydı)                          │
│                                                                               │
│  Metin kataloğu (tr)                                                          │
└───────────────────────────────────────────────────────────────────────────────┘
            ▲
            │ yalnızca ilk yüklemede statik dosyalar (HTML/JS/CSS), HTTPS
   Statik dosya barındırma (sunucu tarafı mantık yok)
```

**Katmanlar:**

- **UI katmanı:** Ekranları DOM üzerinde çizer ve kullanıcı girdisini alır. İş kuralı içermez.
- **Alan (domain) katmanı:** Oturum motoru, soru planlayıcı, ilerleme servisi ve zamanlayıcı. Saf TypeScript modülleridir; DOM'a ve tarayıcı API'lerine doğrudan bağlı değildir. Rastgele sayı üreteci ve saat dışarıdan verilir, böylece testte deterministik çalışır (Bölüm 9).
- **Altyapı katmanı:** Depolama adaptörü. `localStorage` erişimi yalnızca buradan yapılır.

**Neden bu kadar sade?** Kapsam kuralı ürün büyüklüğüne uygun çözüm istiyor. BA dokümanında hesap, senkronizasyon, çok oyunculu mod ve sınıf yönetimi kapsam dışı (BA Bölüm 7). Kişisel veri de toplanmayacak (NFR-07). Bu yüzden sunucu, veritabanı, kimlik doğrulama ve API katmanına ihtiyaç yok.

**Bu dokümandaki teknik varsayımlar** (gereksinim değildir, BA cevaplarına göre değişebilir):

- TV-1: Bir oturumun kendisi (soru kuyruğu, sıradaki soru) kalıcı saklanmaz. Yalnızca cevapların ilerlemeye etkisi saklanır. FR-15 de "o ana kadar verilen cevapların ilerlemeye işlenmesini" istiyor, oturumun kaldığı yerden devam etmesini istemiyor.
- TV-2: Tablo seçimi (FR-01) uygulama kapatılınca hatırlanmaz. Kalıcı saklanması gereken veriler FR-15 ve FR-17'de sayılmış, tablo seçimi aralarında yok.
- TV-3: "a × b" sorusunda a tablo numarasıdır, b ise 1–10 arasındadır (FR-02 örneği "3×1 … 3×10"). 3×4 ile 4×3 ayrı sorulardır. Bu, FR-11'deki 10×10 = 100 hücreli ızgarayla tutarlıdır (bkz. Bölüm 12, S-10).

## 3. Teknoloji seçimleri ve gerekçeleri

| # | Karar | Seçenekler | Seçim ve gerekçe | İlgili gereksinim |
|---|---|---|---|---|
| T1 | Platform | (a) Tarayıcı web uygulaması, (b) yerel mobil uygulama, (c) ikisi birden | **(a) Web uygulaması (geçici).** Platform BA'da açık (Açık soru 1). Web, telefon, tablet ve bilgisayarda tek kodla çalışır. NFR-05'teki ölçüt de CSS pikseli cinsinden ([W3C SC 2.5.8](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)). Hesap ve sunucu gerekmiyor (NFR-07). Cevap "yalnızca mağaza uygulaması" olursa T1 ve T4 yeniden değerlendirilir. Safari'nin veri silme riski de bu kararla ilişkili (R1). | NFR-03, NFR-05, NFR-07, FR-15 |
| T2 | Dil | JavaScript, TypeScript | **TypeScript.** Veri modeli (Bölüm 5) ve durum geçişleri (FR-10) tip kontrolünden yararlanır. Kaynak gerektirmeyen bir mühendislik tercihidir. | FR-10, FR-15 |
| T3 | UI framework'ü | Framework'süz DOM, bileşen framework'ü | **Framework'süz.** 6 ekran ve tek bir özel bileşen (tuş takımı) var. Framework bağımlılığı bu ölçekte bakım yükü getirir ve fayda sağlamaz (kapsam kuralı: ürün büyüklüğüne uygunluk). | FR-01 – FR-17 (UI) |
| T4 | Derleme aracı | Vite (`vanilla-ts` şablonu) | Vite `vanilla-ts` şablonunu destekliyor. `vite build` çıktısı üretime hazır **statik dosyalardır** ([Vite rehberi](https://vite.dev/guide/)). Bu, sunucusuz mimari (T1) için yeterli. | NFR-07, NFR-08 |
| T5 | Kalıcı depolama | `localStorage`, IndexedDB | **`localStorage`.** Veri tarayıcı oturumları arasında korunur ve son kullanma süresi yoktur ([MDN localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). İşlemler **eşzamanlıdır**; MDN, performans sorununun büyük veri kümelerinde ortaya çıktığını belirtiyor ([MDN Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)). Bizim veri ~3–5 KB olduğundan (Bölüm 7) eşzamanlı yazma bir avantaj: cevap, geri bildirim gösterilmeden önce diske yazılmış olur (NFR-10). Kota origin başına yaklaşık 5 MiB ([MDN kota ve silme kriterleri](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)); ihtiyacın çok üstünde. IndexedDB'nin eşzamansız API'si bu veri boyutunda karmaşıklıktan başka bir şey getirmez. | FR-15, FR-16, FR-17, NFR-10 |
| T6 | Kalıcılık isteği | `navigator.storage.persist()` | İlk anlamlı etkileşimden sonra çağrılır. Kalıcılık izni verilen origin'ler depolama baskısı nedeniyle silinmez. Safari ve Chromium bu isteği kullanıcıya sormadan, site etkileşim geçmişine bakarak kabul ya da reddeder ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Bu nedenle bir garanti değil, risk azaltıcıdır (R1). | FR-15, NFR-10 |
| T7 | Cevap girişi | (a) `<input inputmode="numeric">` ile cihaz klavyesi, (b) ekran üstü özel tuş takımı | **(b) Özel tuş takımı.** `inputmode` yalnızca bir ipucudur ve girdiyi doğrulamaz ([MDN inputmode](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode)). Cihaz klavyesinin tuş boyutu kontrol edilemez (NFR-05) ve klavye her cihazda farklı görünür. Özel tuş takımı (0–9, Sil, Tamam) yalnızca rakam kabul eder, 3 hane sınırını uygular (FR-04) ve tuş boyutunu bizim belirlememizi sağlar. Bilgisayarda fiziksel klavyedeki rakam, Backspace ve Enter tuşları da aynı işlevlere bağlanır. | FR-04, NFR-05 |
| T8 | Test çalıştırıcı | Vitest | Vitest, Vite yapılandırmasını varsayılan olarak okur; ek yapılandırma gerekmez ([Vitest rehberi](https://vitest.dev/guide/)). Alan katmanı saf TypeScript olduğu için birim testleri DOM gerektirmez. | Bölüm 9 |
| T9 | Dış kaynak politikası | CSP `default-src 'self'` (`<meta>` ile) | `default-src 'self'` yalnızca aynı origin'den kaynak yüklenmesine izin verir. İstemci tarafında çizilen, yalnızca statik kaynaklardan oluşan uygulamalarda CSP `<meta http-equiv>` ile verilebilir ([MDN CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)). Üçüncü taraf betik, reklam, analitik veya font yüklenmesi teknik olarak engellenmiş olur. | NFR-07, NFR-08 |
| T10 | Yazı tipi | Sistem yazı tipleri | Dış font servisi kullanılmaz (T9 ile tutarlı). Türkçe karakter desteği hedef cihazlarda kontrol edilir (Bölüm 9). | NFR-01, NFR-07 |

**Bilerek seçilmeyenler:** Sunucu, veritabanı, kimlik doğrulama, analitik/telemetri, servis çalışanı (service worker) ve çevrimdışı önbellek. Çevrimdışı çalışma BA'da açık soru (Açık soru 2). Gereksinime dönüşmediği için mimariye eklenmedi. Cevap "evet" olursa statik yapı servis çalışanı eklemeye uygundur (bkz. Bölüm 12, S-1).

## 4. Bileşenler ve sorumlulukları

| ID | Bileşen | Sorumluluk | İlgili gereksinim |
|---|---|---|---|
| K1 | **Uygulama kabuğu ve ekran yöneticisi** | Açılışta depolamadan veriyi yükler (K7), ekranlar arasında geçişi yönetir. İlk açılışta tanıtım ekranı yoktur; doğrudan E1 açılır. | NFR-02, FR-15 |
| K2-E1 | **Ana ekran / tablo seçimi** | 1–10 tablolarını açılıp kapanan düğmeler olarak gösterir. Kazanılmış yıldızları (FR-13) düğme üzerinde gösterir. Hiç tablo seçili değilse "Başla" pasiftir. Diğer ekranlara (İnceleme, İlerleme, Ayarlar) giriş buradandır. Oturuma başlamak için en fazla 2 dokunuş gerekir: bir tablo seçmek ve "Başla". | FR-01, FR-13, NFR-02 |
| K2-E2 | **Tablo inceleme** | Seçilen tablonun 10 satırını tekrarlı toplama karşılığıyla gösterir (örn. "4×3 = 4+4+4 = 12"). Satır gerekirse alt satıra kayar; ekran dikey kaydırılabilir. | FR-02, FR-03 |
| K2-E3 | **Alıştırma** | "a × b = ?" sorusunu, cevap alanını ve tuş takımını (K3) gösterir. Geri bildirimi simge, metin ve renk birlikte kullanarak verir. Süre açıksa geri sayımı gösterir. Uygulamadan çıkışa izin verir; çıkışta o ana kadarki cevaplar zaten kaydedilmiştir. | FR-04, FR-07, FR-14, NFR-06, NFR-09 |
| K2-E4 | **Oturum özeti** | Doğru/toplam sayısını, yanlış yapılan soruları ve bu oturumda "Öğrenildi"ye geçen soruları listeler. Yeni kazanılan yıldız varsa tebriki gösterir. | FR-12, FR-13 |
| K2-E5 | **İlerleme ızgarası** | 10×10 ızgarada her hücrenin durumunu gösterir. Durum, renkle birlikte simge ve erişilebilir metinle de ayırt edilir (Yeni: boş daire, Çalışılıyor: yarım daire, Öğrenildi: onay işareti). "İlerlemeyi sıfırla" girişi buradadır. | FR-11, FR-16, NFR-06 |
| K2-E6 | **Ayarlar** | Soru sırası (sıralı/karışık, varsayılan karışık), oturum uzunluğu (5/10/20, varsayılan 10), süre (kapalı/10/7/5 sn, varsayılan kapalı) ve ses (yalnızca ses efekti varsa, bkz. S-8) ayarlarını içerir. Ayarları değiştirmek zorunlu değildir; varsayılanlarla oturum başlar (NFR-02). | FR-05, FR-06, FR-14, FR-17 |
| K3 | **Tuş takımı** | 0–9, Sil ve Tamam tuşlarını içerir. En fazla 3 hane kabul eder; boş cevapta Tamam pasiftir. Tuşlar en az 24×24 CSS pikseldir. Çocuklar için daha büyük değer BA/tasarım kararı bekliyor (S-11). Fiziksel klavye eşlemesi vardır (T7). | FR-04, NFR-05 |
| K4 | **Oturum motoru** | Planlayıcıdan (K5) soru listesini alır ve soru kuyruğunu yönetir. Cevabı değerlendirir, FR-08 tekrarlarını kuyruğa ekler, zamanlayıcıyı (K8) başlatır ve durdurur. Her cevapta K6'yı çağırır. Oturum sonunda özet verisini üretir. | FR-04, FR-06, FR-07, FR-08, FR-12, FR-14, NFR-09 |
| K5 | **Soru planlayıcı** | Seçilen tablolar, oturum uzunluğu, sıra modu ve mevcut ilerlemeden N soruluk bir plan üretir (algoritma: Bölüm 6.1–6.3). | FR-01, FR-05, FR-06, FR-09 |
| K6 | **İlerleme servisi** | Cevap sonrası soru durumunu günceller (Bölüm 6.5). Tablo yıldızı koşulunu denetler (FR-13). Güncel durumu K7 üzerinden yazar. Sıfırlama işlemini yürütür. | FR-10, FR-11, FR-13, FR-15, FR-16 |
| K7 | **Depolama adaptörü** | Tek anahtarlı JSON kaydını okur ve yazar. Şema sürümünü denetler. Bozuk veya okunamayan kayıtta varsayılan boş duruma döner ve bozuk kaydı üzerine yazmadan önce ayrı bir anahtara yedekler. `QuotaExceededError` ve depolamanın kapalı olması durumlarını `try…catch` ile yakalar ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)); uygulama bellekte çalışmaya devam eder ve kısa bir uyarı gösterir. T6'daki kalıcılık isteğini yapar. | FR-15, FR-16, FR-17, NFR-10 |
| K8 | **Zamanlayıcı** | Süre açıksa soru ekrana geldiğinde bir son tarih belirler. Kalan süreyi monoton saat (`performance.now()`) ile hesaplar; görsel geri sayımı yaklaşık 250 ms'de bir günceller. Süre dolunca K4'e "zaman aşımı" bildirir. Saat dışarıdan verildiği için test edilebilir. | FR-14 |
| K9 | **Yetişkin doğrulama kapısı** | Sıfırlama öncesinde, uygulamanın 1–10 aralığının dışında kalan bir işlem sorar (örn. rastgele iki basamaklı × tek basamaklı bir çarpım). Sorudan sonra ayrı bir onay adımı gelir. Doğrulama veya onay başarısızsa hiçbir veri silinmez. Kapının gücü BA'ya soruldu (S-7). | FR-16 |
| K10 | **Metin kataloğu (tr)** | Tüm arayüz metinleri tek bir dosyadadır. Yönerge cümleleri ayrı bir anahtar grubunda (`yonerge.*`) tutulur; böylece 8 kelime sınırı otomatik test edilebilir. Geri bildirim metinleri suçlayıcı olmayan bir dille yazılır (örn. "Neredeyse! 6×7 = 42"). | NFR-01, FR-07, NFR-09 |
| K11 | **Ses (koşullu)** | Yalnızca BA ses efektlerini kapsama alırsa uygulanır (S-8). Uygulanırsa olumsuz veya cezalandırıcı ses kullanılmaz; ayar K7'de saklanır. | FR-17, NFR-09 |

## 5. Veri modeli

### 5.1 Kalıcı kayıt (`localStorage`, anahtar: `carpim-tablosu`)

Tek bir anahtar kullanılır. Soru durumları ve yıldızlar aynı `setItem` çağrısıyla yazıldığından birbirleriyle her zaman tutarlıdır.

```json
{
  "schemaVersion": 1,
  "facts": {
    "7x8": { "streak": 2 },
    "3x4": { "streak": 3 }
  },
  "stars": [5],
  "settings": {
    "order": "mixed",
    "sessionLength": 10,
    "timerSeconds": null,
    "soundOn": true
  }
}
```

| Alan | Tip | Açıklama | Gereksinim |
|---|---|---|---|
| `schemaVersion` | sayı | İleride yapılacak alan değişiklikleri için sürüm. | FR-15 |
| `facts` | nesne | Anahtar `"<a>x<b>"`, a ve b 1–10 arası. **Yalnızca en az bir kez sorulmuş sorular** yer alır. | FR-10, FR-11 |
| `facts[k].streak` | 0–3 tam sayı | Son ardışık doğru cevap sayısı. Doğru cevapta `min(streak+1, 3)` olur; yanlış cevapta veya zaman aşımında 0 olur. | FR-10, FR-14 |
| `stars` | sayı dizisi | Yıldız kazanılmış tablolar. Bir kez eklenen yıldız, sıfırlama dışında silinmez ("kalıcı işaret"; bkz. S-9). | FR-13 |
| `settings.order` | `"mixed"` \| `"sequential"` | Varsayılan `"mixed"`. | FR-05 |
| `settings.sessionLength` | 5 \| 10 \| 20 | Varsayılan 10. | FR-06 |
| `settings.timerSeconds` | `null` \| 10 \| 7 \| 5 | `null` = süre kapalı (varsayılan). | FR-14 |
| `settings.soundOn` | boolean | Yalnızca K11 uygulanırsa kullanılır. | FR-17 |

### 5.2 Türetilmiş durum (saklanmaz)

| Durum | Koşul |
|---|---|
| Yeni | `facts` içinde anahtar yok (soru hiç sorulmadı). |
| Çalışılıyor | Anahtar var ve `streak < 3`. |
| Öğrenildi | `streak = 3` (son 3 cevap doğru). |

Durumun saklanmayıp türetilmesi, "durum" ile "son cevaplar" arasında tutarsızlık oluşmasını yapısal olarak engeller. "Son 3 cevabın doğru olması", "ardışık doğru sayısı ≥ 3" ile aynı anlama gelir (FR-10).

### 5.3 Oturum durumu (yalnızca bellekte)

- `plan`: N soruluk başlangıç listesi (K5).
- `queue`: Sıradaki sorular. Her öğe `{ fact, isRepeat }` biçimindedir.
- `answers`: `{ fact, given, correct, timedOut, isRepeat }` kayıtları (FR-12 özeti için).
- `statusBefore`: Oturum başında her sorunun durumu. Bu oturumda "Öğrenildi"ye geçenleri bulmak için kullanılır (FR-12).

### 5.4 Sıfırlama

FR-16 sıfırlamasında `facts` boşaltılır ve `stars` silinir. `settings` korunur, çünkü ayarlar "ilerleme" değildir (geçici; bkz. S-12).

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Karıştırma: Fisher–Yates (Durstenfeld sürümü)

Dizi sondan başa doğru dolaşılır. Her i için 0 ≤ j ≤ i aralığından rastgele bir j seçilir ve a[i] ile a[j] yer değiştirir. Algoritma O(n) sürede çalışır ve rastgele kaynak yansız olduğunda her permütasyon eşit olasılıklıdır. j'nin yanlış aralıktan seçilmesi (örn. j < i ya da her adımda tüm dizi) yanlı sonuç üretir ([Wikipedia: Fisher–Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)). Bu nedenle j aralığı birim testle doğrulanır (Bölüm 9).

**Neden:** FR-05 karışık sıra istiyor. Basit ve yansız bir karıştırma, bazı soruların sistematik olarak öne çıkmasını önler. Rastgele sayı fonksiyonu (`rng: () => number`) dışarıdan verilir: üretimde `Math.random`, testte tohumlu (seeded) bir üreteç kullanılır.

**Yardımcı fonksiyon:** `draw(S, k)`, S kümesinden k öğe üretir. S karıştırılır ve ilk k öğe alınır. k > |S| ise yeni karıştırmalar arka arkaya eklenir; böylece her öğe olabildiğince eşit sayıda tekrar eder.

### 6.2 Plan üretimi: karışık mod (FR-01, FR-05, FR-06, FR-09)

```
P  = seçilen tabloların tüm soruları  (|P| = 10 × seçili tablo sayısı, en az 10)
W  = P içinde durumu "Öğrenildi" olmayanlar (Yeni + Çalışılıyor)
L  = P \ W
N  = oturum uzunluğu (5, 10 veya 20)

W boşsa:            liste = draw(P, N)
L boşsa:            liste = draw(W, N)
aksi halde:         kW = ⌈N / 2⌉ ;  kL = N − kW
                    liste = draw(W, kW) + draw(L, kL)
liste = shuffle(liste)
liste = ardışıkTekrarıGider(liste)
```

- **FR-09:** W boş değilse sorulardan en az ⌈N/2⌉ tanesi W'den gelir. Bu, %50 eşiğini karşılar.
- **L küçükse** (örn. L'de 1 soru, kL = 5) L'den tekrarlı çekilir. W'ye taşma yapılmaz, çünkü yapılırsa tek bir zayıf sorunun oturumun yarısından fazlasını kaplaması mümkün olur ve bu durumda FR-05'teki "art arda gelmeme" koşulu sağlanamayabilir.
- **Uygulanabilirlik:** Bu kuralla hiçbir soru ⌈N/2⌉ kereden fazla görünmez. Bu sınır, aynı öğenin yan yana gelmediği bir dizilim için yeter koşuldur.
- **Not:** W'de tek soru varsa (örn. N=20 için) o soru 10 kez sorulur. Bu FR-09'a uygundur ama pedagojik olarak uygun olup olmadığı BA'ya soruldu (S-5).

**`ardışıkTekrarıGider` (FR-05):** Karıştırılmış listede `liste[i] = liste[i−1]` olan her i için, i'den sonra gelen ve komşularıyla çakışmayan bir öğe bulunup yer değiştirilir. 20 denemede çözülemezse deterministik bir dizilime geçilir: öğeler sıklığa göre sıralanır ve önce çift, sonra tek konumlara yerleştirilir. Yukarıdaki sınır sayesinde bu dizilim her zaman bulunur. Liste en fazla 20 öğe olduğu için maliyet ihmal edilebilir.

### 6.3 Plan üretimi: sıralı mod (FR-05, FR-06), geçici

```
P sıralı: tablo artan, ardından b = 1..10   (3×1, 3×2, …, 3×10, 4×1, …)
liste = P'nin ilk N öğesi;  N > |P| ise P baştan tekrarlanır
```

FR-09 BA dokümanında yalnızca karışık mod için tanımlı. Bu yüzden sıralı modda zayıf soru önceliği uygulanmaz. **Geçici davranış:** N < |P| olduğunda (örn. iki tablo, 10 soru) yalnızca ilk tablonun soruları sorulur; N > |P| olduğunda (örn. tek tablo, 20 soru) tablo iki kez sorulur. Doğru davranış BA'ya soruldu (S-4).

### 6.4 Oturum içi tekrar: FR-08

```
Kuyruktaki konumu i olan ve isRepeat = false olan soru yanlış cevaplanırsa
(zaman aşımı dahil, FR-14):
  hedef = i + 3          # soru ile tekrarı arasında en az 2 soru
  hedef ≤ kuyruk sonu ise: tekrarı 'hedef' konumuna ekle (isRepeat = true)
  değilse:               tekrarı kuyruğun sonuna ekle
```

- **Aralık:** "En az 2 soru sonra" ifadesinin iki yorumu var: tekrar soru 2. sırada da gelebilir, araya 2 soru da girebilir. Araya en az 2 soru koyan yorum iki okumayı da karşılar.
- **Tekrar sayısı:** Tekrar sorusunun yeniden yanlış cevaplanması yeni bir tekrar doğurmaz. FR-08 "en az 1 kez" istiyor; bu kural oturumu sınırlı tutar (en fazla 2N soru). Bu kısıtın doğruluğu BA'ya soruldu (S-2).
- **Oturum sonu çatışması (geçici):** Yanlış cevap son iki sorudan birine verilmişse arada 2 soru bırakmak mümkün olmayabilir. US-01 oturumun N soru ve FR-08 tekrarlarıyla bittiğini söylüyor, dolgu soru eklemeye izin vermiyor. Bu durumda tekrar kuyruğun sonuna eklenir ve aralık koşulu mümkün olduğu kadar sağlanır. En kötü durumda (son soru yanlış, bekleyen başka tekrar yok) tekrar hemen ardından gelir. FR-08 ile US-01 arasındaki bu çelişki S-3'te soruldu. Kural tek bir fonksiyonda tutulduğu için cevaba göre kolayca değiştirilebilir.
- Tekrar soruları da ilerlemeyi günceller (FR-10, "son 3 cevap").

### 6.5 Öğrenme durumu güncellemesi: FR-10

```
cevap doğru:              streak = min(streak + 1, 3)
cevap yanlış / zaman aşımı: streak = 0
(ilk cevapta kayıt { streak: 0 } ile oluşturulur, ardından kural uygulanır)
```

Bu kural Bölüm 5.2'deki türetimle birlikte BA'nın tanımını birebir karşılar:

- Yeni → (doğru) Çalışılıyor (streak 1)
- 3 ardışık doğru → Öğrenildi
- Öğrenildi → (yanlış) Çalışılıyor

"Son 3 cevap" eşiği BA'nın ürün kararıdır (BA Varsayım 5). Kodda tek bir sabit olarak tutulur.

### 6.6 Yıldız kontrolü: FR-13

Her cevaptan sonra, yalnızca cevaplanan sorunun tablosu (a) için kontrol yapılır: a×1 … a×10 sorularının hepsi Öğrenildi ise ve a `stars` içinde yoksa a eklenir ve "yeni yıldız" olayı üretilir. Kontrol O(10)'dur.

**Geçici:** Tebrik, oturumu bölmemek için oturum özetinde (E4) gösterilir. Tebrikin anında gösterilip gösterilmeyeceği S-9'da soruldu.

### 6.7 Cevap işleme sırası: NFR-10, NFR-03

```
1. Cevabı değerlendir                       (O(1))
2. K6: durumu güncelle, yıldızı kontrol et  (O(10))
3. K7: tek JSON kaydını eşzamanlı yaz       (~3–5 KB)
4. Geri bildirimi çiz
5. Gerekirse FR-08 tekrarını kuyruğa ekle
```

Kayıt, geri bildirim gösterilmeden **önce** yazılır. Böylece "son onaylanan cevap" uygulama beklenmedik biçimde kapansa bile kaydedilmiş olur (NFR-10, FR-15).

### 6.8 Zamanlayıcı: FR-14

- Soru ekrana geldiğinde `bitis = simdi + timerSeconds × 1000` olarak belirlenir.
- Cevap onaylandığında zamanlayıcı durur.
- `simdi ≥ bitis` olduğunda cevap "zaman aşımı" ile yanlış sayılır, doğru cevap gösterilir ve sonraki soruya geçilir (US-06). Zaman aşımı da 6.4'teki tekrarı tetikler.
- **Geçici:** Sayfa arka plana geçtiğinde (`visibilitychange`) geri sayım duraklar. Uygulama kapalıyken çocuğun yanlış sayılmaması NFR-09'un baskısız tonuyla uyumludur. Ancak BA dokümanında açıkça yazmadığı için S-6'da soruldu.

## 7. Performans

| Konu | Hedef (BA) | Teknik yaklaşım | Gereksinim |
|---|---|---|---|
| Cevap → geri bildirim | ≤ 0,5 sn | Ağ çağrısı yok. Cevap yolu O(1) değerlendirme, O(10) yıldız kontrolü, ~3–5 KB eşzamanlı yazma ve tek bir DOM güncellemesinden oluşur (Bölüm 6.7). Büyük veri gerektirmediği için eşzamanlı depolamanın MDN'de belirtilen performans riski burada geçerli değil ([MDN Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)). Geri bildirim animasyonu varsa ilk karede görünür ve sonra animasyon başlar. | NFR-03, FR-07 |
| Sonraki sorunun görünmesi | ≤ 1 sn | Plan oturum başında bir kez üretilir (≤ 20 öğe). Sonraki soru kuyruktan O(1) ile alınır. **Geçici:** Doğru cevapta yaklaşık 0,8 sn sonra otomatik geçilir. Yanlış cevapta çocuk doğru cevabı okuyabilsin diye "Devam" düğmesi beklenir ve 1 sn ölçümü düğmeye basıldığı andan başlar (bkz. S-13). | NFR-03 |
| Veri boyutu | (BA'da yok) | En fazla 100 soru kaydı ve 10 yıldız, tahmini ≤ 5 KB. Origin başına yaklaşık 5 MiB kotanın çok altında ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). | FR-15 |
| Ölçüm yöntemi | Hedef cihaz sınıfı | Test derlemesinde `performance.now()` işaretleri kullanılır: "onay" → "geri bildirim DOM'da" ve "devam" → "sonraki soru DOM'da". Hedef cihaz sınıfı BA'da açık (Açık soru 1, S-1); cevap gelene kadar ölçüm, test ekibinin erişebildiği en düşük donanımlı telefonda yapılır. | NFR-03 |
| İlk yükleme | (BA'da yok) | BA'da hedef yok; yeni hedef eklenmedi. Framework'süz yapı (T3) ve statik derleme (T4) yükü küçük tutar. | (T3, T4'ün yan etkisi) |

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Kişisel veri | Hiçbir ekranda ad, e-posta, konum vb. alan yok. Hesap yok. Saklanan tek veri soru durumları, yıldızlar ve ayarlardır (Bölüm 5); bunlar kişiyi tanımlamaz. Veri yalnızca cihazdaki tarayıcı depolamasında, uygulamanın kendi origin'inde tutulur ([MDN localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). | NFR-07 |
| Ağ trafiği | İlk yüklemedeki statik dosyalar dışında ağ isteği yapılmaz. Analitik, hata raporlama ve telemetri yoktur. CSP `default-src 'self'` üçüncü taraf betik ve bağlantıları engeller ([MDN CSP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP)). | NFR-07, NFR-08 |
| Reklam ve satın alma | Reklam SDK'sı ve ödeme altyapısı yoktur; CSP bunların sonradan fark edilmeden eklenmesini de zorlaştırır. | NFR-08 |
| Çerez | Kullanılmaz. | NFR-07 |
| Taşıma güvenliği | Statik dosyalar HTTPS üzerinden sunulur. `localStorage` protokole özgü olduğundan (http ve https ayrı depolama, [MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)) uygulama yalnızca tek bir HTTPS adresinden yayınlanır; aksi halde ilerleme iki adres arasında bölünür. | FR-15 |
| Yanlışlıkla sıfırlama | K9 yetişkin kapısı ve ardından onay adımı. Bu bir güvenlik kontrolü değil, kaza önleyici bir mekanizmadır; kararlı bir çocuk aşabilir (R4). | FR-16 |
| Yasal uyum (KVKK, mağaza kuralları) | BA'da araştırılmamış (Açık soru 12). Bu doküman, veri toplanmayan mimariyle uyumu kolaylaştırır ama uyum değerlendirmesi yapmaz. S-14'e bakın. | NFR-07 |

## 9. Test stratejisi

### 9.1 Birim testleri (Vitest, DOM gerektirmez)

Alan katmanı saf olduğu için (Bölüm 2) testler tohumlu rastgele üreteç ve sahte saat ile deterministik çalışır.

| Test | Doğrulanan | Gereksinim |
|---|---|---|
| Planlayıcı yalnızca seçili tablolardan soru üretir (1.000 tohum, rastgele tablo kombinasyonları). | FR-01, US-01 |
| Boş tablo seçiminde plan üretilmez, "Başla" pasif kalır. | FR-01, US-01 |
| Plan uzunluğu N'dir (5/10/20). | FR-06, US-01 |
| Karışık modda ardışık aynı soru yoktur (FR-08 tekrarları hariç); 1.000 tohum ve uç durumlar (W=1, L=1, tek tablo + N=20). | FR-05 |
| W boş değilse W'den gelen soru oranı ≥ %50'dir. | FR-09, US-04 |
| Sıralı mod sırası 3×1 … 3×10'dur (geçici davranışla). | FR-05 |
| Fisher–Yates: j aralığı ve dağılım (küçük dizide permütasyon frekansları kabaca eşit). | FR-05 |
| Yanlış cevapta tekrar ile soru arasında ≥ 2 soru vardır; oturum sonu davranışı geçici kurala uyar. | FR-08, US-04 |
| Durum makinesi: Yeni→Çalışılıyor, 3 doğru→Öğrenildi, Öğrenildi+yanlış→Çalışılıyor (tablo güdümlü test). | FR-10, US-04 |
| 10 sorunun tümü Öğrenildi olunca yıldız eklenir; tekrar eklenmez. | FR-13, US-05 |
| Özet: doğru/toplam, yanlışlar, yeni öğrenilenler. | FR-12, US-05 |
| Sahte saat ile süre dolunca yanlış sayılır, doğru cevap döner, tekrar kuyruğa girer. Süre kapalıyken zamanlayıcı hiç başlamaz. | FR-14, US-06 |
| Depolama: yaz-oku tutarlılığı, bozuk JSON, şema sürümü, `QuotaExceededError` ve depolama kapalı senaryoları. | FR-15, FR-17, NFR-10 |
| Sıfırlama: doğrulama başarısızsa veya onay verilmezse veri değişmez; onayda `facts` ve `stars` silinir. | FR-16, US-07 |
| Metin kataloğu: tüm metinler Türkçe katalogdan gelir; `yonerge.*` metinleri ≤ 8 kelimedir. | NFR-01 |
| Renk paleti: tüm metin/arka plan çiftleri WCAG kontrast formülüyle ((L1+0,05)/(L2+0,05)) hesaplanır ve normal metinde ≥ 4,5:1, büyük metinde ≥ 3:1 olduğu doğrulanır ([W3C SC 1.4.3](https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html)). | NFR-04 |
| FR-03 metin üretimi: "4×3 = 4+4+4 = 12" biçimi; a×1 için tek terim. | FR-02, FR-03, US-02 |

### 9.2 Cihaz üzerinde manuel testler

| Test | Yöntem | Gereksinim |
|---|---|---|
| Yanıt süresi | Test derlemesinde `performance.now()` ölçümü, hedef cihaz sınıfında 50 cevap; tüm değerler eşik altında olmalı. | NFR-03, US-03 |
| Dokunma alanı | Tarayıcı geliştirici araçlarıyla tüm etkileşimli öğelerin boyutu ölçülür (≥ 24×24 CSS px, [W3C SC 2.5.8](https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html)). | NFR-05 |
| Renge bağımlılık | Ekranlar gri tonlamada incelenir; doğru/yanlış ve ızgara durumları ayırt edilebilmeli. | NFR-06, FR-11 |
| Ağ trafiği | Ağ panelinde ilk yüklemeden sonra hiçbir istek görülmemeli; kişisel veri isteyen alan olmamalı. | NFR-07, NFR-08, US-07 |
| Beklenmedik kapanma | Oturum ortasında sekme/uygulama zorla kapatılır; yeniden açılınca son onaylanan cevap ilerlemede görünmeli. | NFR-10, FR-15, US-05 |
| Pedagojik ton | Yanlış cevap ve zaman aşımı akışlarında olumsuz ses, ceza veya oturum sonlandırma olmadığı kontrol edilir. | NFR-09, US-03 |
| Türkçe karakter | Hedef tarayıcılarda sistem yazı tipiyle ı, ğ, ş, İ vb. karakterlerin doğru göründüğü kontrol edilir. | NFR-01 |

### 9.3 Kullanılabilirlik testi

BA'nın ölçütü uygulanır: en az 5 ikinci sınıf öğrencisinden en az 4'ü, ilk açılıştan sonra yardımsız ve ≤ 3 dokunuşla oturum başlatabilmeli (NFR-02). Aynı oturumda US-01 – US-07 kabul kriterleri bir kontrol listesiyle doğrulanır.

## 10. Riskler

| ID | Risk | Etki | Olasılık | Azaltma | Gereksinim |
|---|---|---|---|---|---|
| R1 | **Safari'nin 7 günlük silmesi:** Takip önleme açıkken, son 7 günlük tarayıcı kullanımında etkileşim olmayan origin'in betikle oluşturduğu veri siliniyor ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Bir hafta ara veren çocuğun tüm ilerlemesi kaybolabilir. Ana ekrana eklenen web uygulamalarının bu kuraldan muaf olup olmadığını MDN sayfası teyit etmiyor. | Yüksek | Orta (iOS/Safari kullanımına bağlı) | `navigator.storage.persist()` istenir (T6), ancak Safari bunu kendi kriteriyle onaylar veya reddeder. Kalıcı çözüm platform kararına bağlı: hedef kitle ağırlıklı iOS ise yerel uygulama seçeneği (T1) yeniden değerlendirilmeli. S-1'de soruldu. | FR-15, NFR-10 |
| R2 | Gizli/özel tarama modunda veri son gizli sekme kapanınca siliniyor ([MDN](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). | Orta | Düşük | Depolama yazılamıyorsa K7 kısa bir uyarı gösterir; uygulama bellekte çalışır. | FR-15 |
| R3 | Kullanıcı tarayıcı verilerini temizlerse ilerleme kaybolur. Yedek veya senkronizasyon yok (BA kapsam dışı: hesap ve senkronizasyon). | Orta | Düşük–orta | Kapsam gereği kabul edilen risk. Ebeveyne bilgi verilip verilmeyeceği S-1 ile birlikte değerlendirilmeli. | FR-15, NFR-07 |
| R4 | Yetişkin kapısı, iki basamaklı çarpım yapabilen büyük çocuklar (3.–4. sınıf) tarafından aşılabilir. | Düşük | Orta | Onay adımı ikinci bir engel oluşturur. Daha güçlü bir kapı S-7'de soruldu. | FR-16 |
| R5 | FR-08 ile US-01 arasında oturum sonu çatışması (Bölüm 6.4). | Düşük | Kesin (uç durumda) | Geçici kural uygulanır; kural tek bir fonksiyonda tutulur. S-3. | FR-08, FR-06 |
| R6 | Hedef cihaz sınıfı bilinmediği için NFR-03 ölçümü temsil gücü düşük olabilir. | Orta | Orta | En düşük donanımlı test cihazında ölçüm yapılır; S-1 cevabıyla güncellenir. | NFR-03 |
| R7 | Çocuk kullanıcılar için 24×24 CSS px tuş boyutu küçük kalabilir (BA da daha büyük boyut öneriyor). | Orta | Orta | Tuş boyutu tek bir CSS değişkeniyle yönetilir; S-11 cevabıyla güncellenir. | NFR-05 |
| R8 | Platform cevabı "yalnızca mağaza uygulaması" olursa T1, T4, T5 ve T6 yeniden değerlendirilir. | Yüksek (iş yükü) | Bilinmiyor | Alan katmanı (K4–K6, K8) tarayıcıdan bağımsız tutulduğu için yeniden kullanılabilir; değişiklik UI ve depolama adaptörüyle sınırlı kalır. | NFR-03, NFR-05, FR-15 |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Tablo seçimi (1–10, tek tek veya birlikte; seçim yoksa başlatılamaz) | K2-E1, K5 (P kümesi) | 4, 6.2, 9.1 |
| FR-02 | Tablonun 10 satırını görüntüleme | K2-E2 | 4, 9.1 |
| FR-03 | Tekrarlı toplama karşılığı gösterimi | K2-E2, metin üretim fonksiyonu | 4, 9.1 |
| FR-04 | "a × b = ?" sorusu, sayısal tuş takımı, en fazla 3 hane | K2-E3, K3, T7 | 3, 4 |
| FR-05 | Sıralı/karışık sıra, varsayılan karışık, ardışık tekrar yok | K5, K2-E6, Fisher–Yates, `ardışıkTekrarıGider`, sıralı plan (geçici) | 6.1, 6.2, 6.3 |
| FR-06 | Oturum uzunluğu 5/10/20, varsayılan 10 | K2-E6, K5, `settings.sessionLength` | 4, 5.1, 6.2 |
| FR-07 | Anında, suçlayıcı olmayan geri bildirim ve doğru cevap | K2-E3, K4, K10, cevap işleme sırası | 4, 6.7, 7 |
| FR-08 | Yanlış soru en az 2 soru sonra oturumda tekrar sorulur | K4, tekrar ekleme algoritması | 6.4 |
| FR-09 | Karışık modda zayıf sorulara ≥ %50 öncelik | K5, W/L ayrımı ve kW = ⌈N/2⌉ | 6.2 |
| FR-10 | Soru bazında Yeni/Çalışılıyor/Öğrenildi durumu | K6, `facts[k].streak`, türetilmiş durum | 5.1, 5.2, 6.5 |
| FR-11 | 10×10 ilerleme ızgarası, renk ve simge | K2-E5, türetilmiş durum | 4, 5.2 |
| FR-12 | Oturum sonu özeti | K2-E4, K4, `answers`, `statusBefore` | 4, 5.3 |
| FR-13 | Tablo tamamlanınca tebrik ve kalıcı yıldız | K6, `stars`, yıldız kontrolü | 5.1, 6.6 |
| FR-14 | İsteğe bağlı soru başına süre (varsayılan kapalı; 10/7/5 sn) | K8, K2-E6, `settings.timerSeconds` | 4, 6.8 |
| FR-15 | İlerlemenin kapanıp açılmada korunması | K7, T5, T6, cevap başına eşzamanlı yazma | 3, 5.1, 6.7, 10 (R1–R3) |
| FR-16 | Yetişkin doğrulamalı ve onaylı sıfırlama | K9, K6, sıfırlama kuralı | 4, 5.4, 8 |
| FR-17 | Ses efektlerini kapatma, ayarın korunması | K11 (koşullu), `settings.soundOn` | 4, 5.1 |
| NFR-01 | Türkçe arayüz, yönergeler ≤ 8 kelime | K10, `yonerge.*` testi, T10 | 4, 9.1 |
| NFR-02 | ≤ 3 dokunuşla oturum başlatma | K1 (tanıtım ekranı yok), K2-E1 (2 dokunuş), varsayılan ayarlar | 4, 9.3 |
| NFR-03 | Geri bildirim ≤ 0,5 sn, sonraki soru ≤ 1 sn | Ağsız mimari (T1), O(1) cevap yolu, ölçüm yöntemi | 6.7, 7, 9.2 |
| NFR-04 | Kontrast 4,5:1 / 3:1 | Palet ve otomatik kontrast testi | 9.1 |
| NFR-05 | Dokunma alanı ≥ 24×24 CSS px | K3 özel tuş takımı (T7), ölçüm testi | 3, 4, 9.2 |
| NFR-06 | Durumlar yalnızca renge dayanmaz | K2-E3, K2-E5 (simge ve metin), gri tonlama testi | 4, 9.2 |
| NFR-07 | Kişisel veri istenmez/toplanmaz, hesap yok | Sunucusuz mimari (T1), CSP (T9), yerel depolama | 2, 3, 8 |
| NFR-08 | Reklam ve uygulama içi satın alma yok | Reklam/ödeme altyapısı yok, CSP (T9) | 3, 8 |
| NFR-09 | Hatada olumsuz ses, ceza veya sonlandırma yok | K4 (hata oturumu kesmez), K10 (ton), K11 kısıtı, zamanlayıcı duraklatma (geçici) | 4, 6.8, 9.2 |
| NFR-10 | Beklenmedik kapanmada son onaylı cevaba kadar veri korunur | Kaydın geri bildirimden önce eşzamanlı yazılması, K7 hata yönetimi | 6.7, 9.2 |

## 12. BA'ya geri sorular

Her soru için, cevap gelene kadar uygulanacak geçici teknik davranış belirtilmiştir. Bu davranışlar gereksinim değildir; cevaba göre değiştirilecektir.

| ID | Soru | İlgili | Geçici teknik davranış |
|---|---|---|---|
| S-1 | **Platform ve çevrimdışı çalışma** (BA Açık soru 1 ve 2): Hedef kitle hangi cihazları kullanıyor, iOS/Safari payı ne? Uygulamanın internetsiz çalışması gerekiyor mu? Safari'nin 7 gün etkileşimsizlikte veriyi silmesi (R1) FR-15 için kabul edilebilir mi? | FR-15, NFR-03, NFR-05, NFR-10 | Web uygulaması; çevrimdışı önbellek yok; `persist()` isteği yapılıyor. |
| S-2 | FR-08: Tekrar sorusu da yanlış cevaplanırsa yeniden kuyruğa eklenmeli mi? Eklenecekse bir üst sınır var mı? | FR-08 | Tekrar sorusunun yanlışı yeni tekrar doğurmaz. |
| S-3 | FR-08 ile US-01 çelişkisi: Yanlış cevap oturumun son iki sorusundan birine verilirse "en az 2 soru sonra" koşulu dolgu sorusu eklemeden sağlanamaz. Hangisi öncelikli: aralık koşulu mu (dolgu soru eklenir), oturum uzunluğu mu (aralık gevşer)? | FR-06, FR-08 | Aralık gevşer; tekrar oturum sonuna eklenir. |
| S-4 | FR-05 sıralı mod ile FR-06 nasıl birleşir? Örn. iki tablo seçili ve 10 soruda yalnızca ilk tablo mu sorulmalı? Tek tablo ve 20 soruda tablo iki kez mi tekrarlanmalı? FR-09 sıralı modda da geçerli mi? | FR-05, FR-06, FR-09 | İlk N soru alınır, gerekirse baştan tekrarlanır; FR-09 sıralı modda uygulanmaz. |
| S-5 | FR-09: Zayıf soru sayısı çok azsa (örn. 1) %50 kuralı aynı sorunun bir oturumda 10 kez sorulmasına yol açar. Bu kabul edilebilir mi, yoksa soru başına bir üst sınır var mı? %50 oranı FR-08 tekrarlarını da kapsıyor mu? | FR-09 | Üst sınır yok; oran yalnızca planlanan N soru üzerinden hesaplanır. |
| S-6 | FR-14: Süre açıkken uygulama arka plana alınırsa (çağrı, uygulama değiştirme) süre işlemeye devam etmeli mi? | FR-14, NFR-09 | Sayfa görünmezken geri sayım duraklar. |
| S-7 | FR-16: Yetişkin doğrulamasının gücü ne olmalı? Örnekteki "basit yetişkin sorusu" 3.–4. sınıf öğrencilerince çözülebilir. | FR-16 | Uygulama aralığı dışında bir çarpım sorusu ve ardından onay adımı. |
| S-8 | FR-17 "ses efektleri varsa" diyor: Üründe ses efekti olacak mı? Olacaksa hangi olaylarda (doğru cevap, yıldız vb.)? | FR-17, NFR-09 | Ses efekti uygulanmaz; K11 ve ses ayarı devre dışı. |
| S-9 | FR-13: Yıldız kazanıldıktan sonra o tablodaki bir soru tekrar "Çalışılıyor"a düşerse yıldız korunmalı mı? Tebrik anında mı, oturum sonunda mı gösterilmeli? | FR-13 | Yıldız korunur (sıfırlama hariç); tebrik oturum özetinde gösterilir. |
| S-10 | 3×4 ve 4×3 ayrı sorular mı (FR-11'deki 100 hücreli ızgara bunu ima ediyor), yoksa biri öğrenilince diğeri de öğrenilmiş mi sayılmalı? | FR-10, FR-11, FR-13 | Ayrı sorular (TV-3). |
| S-11 | NFR-05 / BA Açık soru 11: Çocuklar için tuş takımı tuşlarının kesin minimum boyutu nedir? | NFR-05 | 24×24 CSS px; tek CSS değişkeniyle ayarlanabilir. |
| S-12 | FR-16 "tüm ilerleme" ayarları (oturum uzunluğu, süre, ses) da kapsıyor mu? | FR-16 | Ayarlar korunur; yalnızca soru durumları ve yıldızlar silinir. |
| S-13 | NFR-03 "bir sonraki sorunun görünmesi ≤ 1 sn" hangi andan ölçülür? Yanlış cevapta çocuk doğru cevabı okumak için zamana ihtiyaç duyar. Sonraki soruya geçiş otomatik mi, "Devam" düğmesiyle mi olmalı? | NFR-03, FR-07 | Doğru cevapta ~0,8 sn sonra otomatik geçiş; yanlışta "Devam" düğmesi, ölçüm düğmeden başlar. |
| S-14 | BA Açık soru 12 (KVKK, mağaza kuralları): Bu konu kimin tarafından ve ne zaman değerlendirilecek? Sonuç teknik kararları (ör. barındırma yeri, gizlilik metni) etkileyebilir. | NFR-07 | Veri toplanmayan mimari; ek bir yasal metin veya mekanizma yok. |
| S-15 | FR-12 "doğru sayısı / toplam soru": Toplam, FR-08 tekrarlarını da içeriyor mu? | FR-12 | Toplam, tekrarlar dahil sorulan tüm sorulardır; tekrarlar özette ayrıca işaretlenir. |
| S-16 | FR-01 – FR-06: Tablo seçimi ile soru sırası ve oturum uzunluğu ayarları uygulama kapatılınca hatırlanmalı mı? (FR-15 ve FR-17 yalnızca öğrenme durumu, yıldız ve ses ayarını sayıyor.) | FR-01, FR-05, FR-06, FR-15 | Sıra ve uzunluk ayarları saklanır (ayar ekranında oldukları için); tablo seçimi saklanmaz (TV-2). |

---

## Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | MDN: Storage quotas and eviction criteria | https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria | Web Storage kotası (origin başına yaklaşık 5 MiB), `QuotaExceededError`, best-effort ve kalıcı depolama, `navigator.storage.persist()` ve Safari/Chromium'un isteği sormadan onaylaması, Safari'nin 7 günlük silme kuralı, ana ekran web uygulamaları için muafiyetin teyit edilmemesi. |
| 2 | MDN: Window.localStorage | https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage | Verinin oturumlar arası korunması ve son kullanma süresinin olmaması, değerlerin metin olarak saklanması, origin ve protokole özgülük, gizli modda silinme. |
| 3 | MDN: Web Storage API | https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API | `localStorage`'ın eşzamanlı ve bloklayıcı olması; performans riskinin büyük veride ortaya çıkması; IndexedDB'nin eşzamansız alternatif olması. |
| 4 | MDN: inputmode | https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode | `inputmode="numeric"`in yalnızca sanal klavye ipucu olması ve girdiyi doğrulamaması. |
| 5 | MDN: Content Security Policy (CSP) | https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/CSP | `default-src 'self'` ile yalnızca aynı origin'den kaynak yüklenmesi; statik tek sayfalı uygulamalarda CSP'nin `<meta>` ile verilebilmesi. |
| 6 | Vite: Getting Started | https://vite.dev/guide/ | `vanilla-ts` şablonu; `vite build` çıktısının üretime hazır statik dosyalar olması. |
| 7 | Vitest: Getting Started | https://vitest.dev/guide/ | Vitest'in Vite yapılandırmasını varsayılan olarak okuması. |
| 8 | Wikipedia: Fisher–Yates shuffle | https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle | Durstenfeld sürümünün adımları, O(n) karmaşıklık, yansız permütasyon ve yanlış j aralığının yol açtığı yanlılık. |
| 9 | W3C: Understanding SC 1.4.3 Contrast (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/contrast-minimum.html | Kontrast oranı formülü (L1+0,05)/(L2+0,05); büyük metin tanımı; 4,5:1 ve 3:1 eşikleri. |
| 10 | W3C: Understanding SC 2.5.8 Target Size (Minimum) | https://www.w3.org/WAI/WCAG22/Understanding/target-size-minimum.html | Dokunma hedeflerinin en az 24×24 CSS piksel olması ve aralık istisnası. |