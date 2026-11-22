# Çarpım Tablosu Öğrenme Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v1
- **Girdi:** /home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v1-run2.md

---

## 1. Özet

Bu doküman, BA analizinde tanımlanan basit çarpım tablosu uygulamasının (FR-01–FR-17, NFR-01–NFR-10) teknik çözümünü anlatır.

Önerilen çözüm, sunucusu olmayan tek sayfalık bir **web uygulamasıdır** (Progressive Web App olarak kurulabilir). Uygulama tarayıcıda çalışır ve tüm verisini cihazda (`localStorage`) tutar. Ana kararlar şunlardır:

- **Sunucu, hesap ve veritabanı yok.** NFR-07 kişisel veri toplanmamasını, NFR-08 reklam ve satın alma olmamasını istiyor. Bu iki gereksinim sunucusuz mimariyle doğal olarak karşılanır. Ayrıca gönderilecek veri olmadığı için NFR-03'teki yanıt süreleri ağdan etkilenmez.
- **Tek kod tabanı.** Platform hâlâ açık bir soru (BA Açık soru 1). Web uygulaması telefon, tablet ve bilgisayar tarayıcısında aynı koddan çalışır. NFR-05'teki ölçüt de zaten CSS pikseli cinsindendir.
- **UI çatısı (framework) kullanılmıyor.** Uygulama yaklaşık 6 ekrandan oluşuyor; TypeScript ve Vite yeterli.
- **Soru seçimi ve öğrenme durumu saf fonksiyonlardan oluşan bir "Soru Motoru"nda toplanıyor.** Bu motor FR-05, FR-08, FR-09 ve FR-10'u karşılar ve birim testlerle doğrulanır.

En önemli teknik risk şudur: Safari'de izleme önleme açıkken, yedi gün etkileşim olmayan sitelerin script ile yazılmış verisi silinebilir. Bu durum FR-15 ve NFR-10'u tehdit eder (bkz. Bölüm 10). BA dokümanındaki bazı belirsizlikler algoritmayı doğrudan etkiliyor. Bunlar için geçici tasarım kararları alındı ve hepsi Bölüm 12'de BA'ya soruldu.

## 2. Mimari genel bakış

```
┌──────────────────────────── Tarayıcı (telefon / tablet / bilgisayar) ────────────────────────────┐
│                                                                                                  │
│  UI Katmanı (ekranlar)                                                                           │
│  ┌───────────┐ ┌──────────┐ ┌────────────┐ ┌────────────┐ ┌──────────┐ ┌──────────┐              │
│  │Ana ekran /│ │İnceleme  │ │Alıştırma   │ │Oturum sonu │ │İlerleme  │ │Ayarlar   │              │
│  │tablo seçim│ │(FR-02/03)│ │+ tuş takımı│ │özeti       │ │ızgarası  │ │(FR-14/16/│              │
│  │(FR-01/05/ │ │          │ │(FR-04/07/14│ │(FR-12)     │ │(FR-11/13)│ │ 17)      │              │
│  │ 06)       │ │          │ │)           │ │            │ │          │ │          │              │
│  └─────┬─────┘ └────┬─────┘ └─────┬──────┘ └─────┬──────┘ └────┬─────┘ └────┬─────┘              │
│        └────────────┴──────┬──────┴──────────────┴─────────────┴────────────┘                    │
│                            ▼                                                                     │
│  Uygulama Durumu (tek durum nesnesi + ekran yönlendirici)                                         │
│        │                    │                    │                     │                         │
│        ▼                    ▼                    ▼                     ▼                         │
│  ┌──────────────┐   ┌────────────────┐   ┌──────────────┐   ┌──────────────────────┐             │
│  │Soru Motoru   │   │Öğrenme Durumu  │   │Zamanlayıcı   │   │Ses Yöneticisi        │             │
│  │(FR-05/06/08/ │   │(FR-10/13)      │   │(FR-14)       │   │(FR-17)               │             │
│  │ 09) saf fonk.│   │saf fonk.       │   │              │   │                      │             │
│  └──────────────┘   └───────┬────────┘   └──────────────┘   └──────────────────────┘             │
│                             ▼                                                                    │
│                     ┌────────────────┐        ┌──────────────────────┐                           │
│                     │Depolama Katmanı│───────►│ localStorage (cihaz) │                           │
│                     │(FR-15/16,      │        └──────────────────────┘                           │
│                     │ NFR-10)        │                                                           │
│                     └────────────────┘                                                           │
└──────────────────────────────────────────────────────────────────────────────────────────────────┘
        ▲
        │ yalnızca ilk yüklemede statik dosyalar (HTML/JS/CSS); kullanıcı verisi gönderilmez
┌───────┴────────┐
│Statik barındırma│
└────────────────┘
```

Mimari ilkeler:

- **İstemci tarafında tek katman.** Arka uç, API ve kullanıcı yönetimi yoktur (NFR-07, NFR-08). Kapsam dışı maddelerdeki "hesap, senkronizasyon, öğretmen paneli" de sunucuyu gerektirmiyor.
- **İş mantığı ile UI ayrı.** Soru Motoru ve Öğrenme Durumu, DOM'a ve depolamaya bağımlı olmayan saf fonksiyonlardır. Rastgele sayı üreteci dışarıdan verilir. Böylece FR-05, FR-08, FR-09 ve FR-10 kuralları belirlenimci (deterministik) birim testlerle doğrulanabilir.
- **Her onaylanan cevaptan sonra kalıcı yazma.** FR-15 ve NFR-10 için (bkz. Bölüm 6.5).
- **Çevrimdışı çalışma kapsama alınmadı.** BA Açık soru 2 yanıtlanmadığı için service worker eklenmedi. Mimari buna engel değildir: tüm iş mantığı ve veri zaten cihazdadır. Gerekirse yalnızca bir service worker ve önbellek eklenir (bkz. Bölüm 12, S-01).

## 3. Teknoloji seçimleri ve gerekçeleri

| # | Karar | Seçim | Gerekçe | Değerlendirilen alternatif | İlgili gereksinim |
|---|---|---|---|---|---|
| T-01 | Platform | Web uygulaması, ana ekrana kurulabilir PWA | MDN'e göre bir PWA "tek kod tabanından birden çok platform ve cihazda çalışabilir" ve cihaza kurulabilir ([MDN: Progressive web apps](https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps)). Platform açık bir soru olduğu için (BA Açık soru 1) telefon, tablet ve tarayıcıyı tek kodla kapsamak en az riskli seçim. NFR-05 ölçütü de CSS pikseli cinsinden verilmiş. | Yerel (native) iOS/Android uygulaması. Mağaza politikaları ve iki ayrı kod tabanı küçük bir ürün için ağır kalır. BA hedef cihazın mağaza uygulaması olmasını isterse karar yeniden ele alınır (S-02). | NFR-05, NFR-07, NFR-08, FR-15 |
| T-02 | Arka uç | Yok; yalnızca statik dosya barındırma | Gereksinimlerin hiçbiri sunucu tarafında veri istemiyor. NFR-07 kişisel veri toplanmamasını şart koşuyor. Vite'in `vite build` çıktısı `dist` klasöründeki statik dosyalardır ve "herhangi bir platforma" yüklenebilir ([Vite: Deploying a Static Site](https://vite.dev/guide/static-deploy)). | Küçük bir API ile bulutta ilerleme saklama. Hesap gerektirir; NFR-07 ve kapsam dışı maddelerle çelişir. | NFR-07, NFR-08, NFR-03 |
| T-03 | Dil | TypeScript | Öğrenme durumu, ayarlar ve depolama şeması gibi veri yapılarının türleri derleme anında denetlenir. Bu, FR-10 durum geçişlerindeki ve FR-15 şemasındaki hataları azaltır. | Düz JavaScript. Daha az araç gerektirir, ancak şema hatalarını yakalamaz. | FR-10, FR-15 |
| T-04 | UI çatısı | Çatısız (DOM API + küçük bir ekran yönlendirici) | Yaklaşık 6 ekran ve tek bir durum nesnesi var. Kapsam kuralı küçük ürün için gereksiz karmaşıklığı yasaklıyor. | React veya Vue. Ekran sayısı artarsa ya da BA kapsamı genişletirse yeniden değerlendirilir. | FR-01–FR-17 (tüm ekranlar) |
| T-05 | Derleme aracı | Vite | Statik çıktı üretir (T-02 ile aynı kaynak). Birim test aracı Vitest ile aynı yapılandırmayı paylaşır (T-08). | Paketleyicisiz düz ES modülleri. TypeScript derlemesi için yine de bir araç gerekir. | NFR-07 (statik çıktı), test stratejisi |
| T-06 | Kalıcı depolama | `localStorage` + tek JSON kaydı; ilk cevapta `navigator.storage.persist()` isteği | `localStorage` verisi "tarayıcı oturumları arasında saklanır" ve süresi yoktur ([MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). Sınır köken (origin) başına yaklaşık 5 MiB ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)); bu üründe veri birkaç KB'tır (Bölüm 5). `persist()` verilirse veri yalnızca kullanıcı isterse silinir (aynı kaynak). | IndexedDB. 100 kayıtlık veri için asenkron API ve şema yönetimi gereksiz karmaşıklık getirir. | FR-10, FR-13, FR-15, FR-17, NFR-10 |
| T-07 | Cevap girişi | Uygulama içi ekran tuş takımı (0–9, Sil, Onayla); fiziksel klavyede rakam tuşları da kabul edilir | `inputmode="numeric"` yalnızca sayısal sanal klavye ister, "geçerlilik kuralı uygulamaz" ([MDN: inputmode](https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode)). FR-04'teki "yalnızca rakam, en fazla 3 hane" kuralını ve NFR-05'teki tuş boyutunu kendi tuş takımımızla tam denetleyebiliriz. Sistem klavyesinin boyutu ise cihaza bağlıdır. | `<input type="text" inputmode="numeric">`. Klavye ve tuş boyutu cihaza göre değişir. | FR-04, NFR-05, NFR-02 |
| T-08 | Birim test | Vitest | Vitest, Vite üzerine kurulu bir test çatısıdır ve varsayılan olarak `vite.config.*` dosyasını okur ([Vitest Guide](https://vitest.dev/guide/)). | Jest. Ayrı yapılandırma ister. | FR-05, FR-08, FR-09, FR-10, FR-13 |
| T-09 | Uçtan uca (E2E) ve erişilebilirlik testi | Playwright + `@axe-core/playwright` | Playwright Chromium, Firefox ve WebKit motorlarında test çalıştırır ([Playwright: Browsers](https://playwright.dev/docs/browsers)). Telefon ve tablet profilleriyle cihaz öykünmesi yapar ([Playwright: Emulation](https://playwright.dev/docs/emulation)). axe motorunu testlerin içinde çalıştırabilir ([Playwright: Accessibility testing](https://playwright.dev/docs/accessibility-testing)). | Yalnızca elle test. US kabul kriterlerinin tekrar edilebilir doğrulamasını sağlamaz. | US-01–US-07, NFR-03, NFR-04, NFR-05, NFR-10 |

## 4. Bileşenler ve sorumlulukları

| # | Bileşen | Sorumluluk | Karşıladığı gereksinimler |
|---|---|---|---|
| B-01 | **Ana ekran / Tablo seçimi** | 1–10 tablolarını çoklu seçimle gösterir. Sıralı/karışık seçimini (varsayılan karışık) ve 5/10/20 soru seçimini (varsayılan 10) sunar. Hiç tablo seçilmemişse "Başla" pasif kalır. Tablo seçilip "Başla"ya basılınca oturum 2 dokunuşta başlar. Tablo kartlarında yıldız işareti görünür. | FR-01, FR-05, FR-06, FR-13, NFR-02 |
| B-02 | **İnceleme ekranı** | Seçilen tablonun 10 satırını "a×b = a+a+…+a = c" biçiminde gösterir. Tekrarlı toplama dizgesi koddan üretilir (b adet a). | FR-02, FR-03 |
| B-03 | **Alıştırma ekranı** | "a × b = ?" sorusunu gösterir. Ekran tuş takımını (T-07) içerir. Cevabı onaylar, geri bildirimi simge ve metinle gösterir. Yanlışta doğru cevabı da gösterir. Süre açıksa geri sayımı görüntüler. Hata oturumu kesmez. | FR-04, FR-07, FR-14, NFR-06, NFR-09 |
| B-04 | **Soru Motoru** (saf modül) | Oturumun ana soru listesini oluşturur (sıralı/karışık, zayıf soru kotası). Bir sonraki soruyu seçer ve oturum içi tekrar kuyruğunu yönetir. Oturumun bitişine karar verir. | FR-05, FR-06, FR-08, FR-09 |
| B-05 | **Öğrenme Durumu** (saf modül) | Her cevaptan sonra olgunun (fact) kaydını günceller ve durumu (Yeni / Çalışılıyor / Öğrenildi) hesaplar. Tablonun yıldız koşulunu denetler. Oturumda "Öğrenildi"ye geçenleri raporlar. | FR-10, FR-12, FR-13 |
| B-06 | **Zamanlayıcı** | Süre açıksa soru başına 10/7/5 sn geri sayar. Geçen süreyi `performance.now()` ile ölçer; bu değer monoton bir saattir ve sistem saati değişikliklerinden etkilenmez ([MDN: performance.now](https://developer.mozilla.org/en-US/docs/Web/API/Performance/now)). Süre dolunca cevabı "zaman aşımı = yanlış" olarak motora bildirir. | FR-14 |
| B-07 | **Oturum sonu ekranı** | Doğru/toplam sayısını, yanlış yapılan soruların tekil listesini ve bu oturumda öğrenilen soruları gösterir. | FR-12 |
| B-08 | **İlerleme ekranı** | 10×10 ızgarada satır a, sütun b, hücre a×b'yi gösterir. Her hücrede durum rengi, simge ve erişilebilir metin etiketi bulunur (örn. "7 çarpı 8: Öğrenildi"). Satır başında tablo yıldızı yer alır. Ebeveyn de aynı ekranı kullanır. | FR-11, FR-13, NFR-06, US-07 |
| B-09 | **Ayarlar ekranı** | Süre sınırı (Kapalı / 10 / 7 / 5 sn, varsayılan Kapalı), ses açma/kapama ve "İlerlemeyi sıfırla" seçeneklerini içerir. | FR-14, FR-16, FR-17 |
| B-10 | **Yetişkin Doğrulama** | Sıfırlamadan önce yetişkin sorusu sorar. Geçici tasarım: rastgele "iki basamaklı × tek basamaklı" bir çarpma (örn. 23 × 7), yani 10×10 tablosu dışında bir işlem. Ardından ayrı bir onay adımı gelir. Doğrulama başarısızsa ya da iptal edilirse hiçbir veri silinmez. | FR-16, US-07 |
| B-11 | **Ses Yöneticisi** | Ses efektleri ürüne eklenirse bunları çalar. Ayardaki açma/kapama durumuna uyar. Yanlış cevapta olumsuz ses çalmaz. | FR-17, NFR-09 |
| B-12 | **Depolama Katmanı** | JSON kaydını okur ve yazar. Şema sürümünü denetler. Her onaylanan cevaptan sonra geri bildirim gösterilmeden önce yazar. Yazma hatasını yakalar (Bölüm 6.5). Sıfırlamada ilerleme alanlarını siler. | FR-15, FR-16, FR-17, NFR-10 |
| B-13 | **Metin kaynağı** (`tr.ts`) | Tüm arayüz metinleri tek dosyada tutulur. Geri bildirim metinleri suçlayıcı olmayan bir dille yazılır. | NFR-01, FR-07, NFR-09 |
| B-14 | **Tema / tasarım belirteçleri** | Renk paleti ve en küçük dokunma boyutu merkezi değişkenlerde tutulur. Durum renkleri 4,5:1 ve 3:1 kontrast ölçütüne göre seçilir. Tuş takımı boyutu tek bir değişkendir; BA Açık soru 11 yanıtlanınca güncellenir. | NFR-04, NFR-05, NFR-06 |

## 5. Veri modeli

### 5.1 Kalıcı kayıt (`localStorage`, tek anahtar: `carpim-tablosu`)

```ts
type FactKey = `${number}x${number}`;        // "7x8": 7'ler tablosunun 8. satırı (a=7, b=8)

interface FactRecord {
  attempts: number;        // toplam cevap sayısı; 0 ise durum "Yeni"
  correctStreak: number;   // art arda doğru sayısı; yanlış veya zaman aşımı 0'a çeker
}

interface StoredData {
  schemaVersion: 1;
  facts: Partial<Record<FactKey, FactRecord>>;   // en fazla 100 kayıt (1..10 × 1..10)
  stars: number[];                               // yıldız kazanılan tablolar, örn. [2, 5]
  settings: {
    timerSeconds: 0 | 5 | 7 | 10;                // 0 = kapalı (varsayılan)
    soundOn: boolean;                            // FR-17
  };
}
```

- **Durum türetilir, saklanmaz** (FR-10). `attempts == 0` ise **Yeni**; `correctStreak >= 3` ise **Öğrenildi**; diğer tüm durumlar **Çalışılıyor**. "Son 3 cevabın doğru olması", art arda doğru sayısının 3 veya üstünde olmasıyla aynı şeydir. Durumu ayrıca saklamak tutarsızlık riski yaratacağı için türetme tercih edildi.
- **a×b ve b×a ayrı olgulardır.** FR-11'deki 10×10 ızgara ve FR-13'teki "bir tablonun 10 sorusu" ifadesi bunu gerektiriyor (doğrulama için bkz. S-07).
- **Boyut:** 100 olgu × yaklaşık 40 bayt + ayarlar, birkaç KB eder. Bu, `localStorage` için verilen yaklaşık 5 MiB sınırının çok altındadır ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)).
- **`schemaVersion`:** Uygulama güncellendiğinde eski kaydın okunabilmesi ve ilerlemenin korunması için tutulur (FR-15).
- **Saklanmayanlar:** Ad, yaş ve cihaz kimliği gibi hiçbir kişisel veri tutulmaz (NFR-07). Tablo seçimi, sıra ve oturum uzunluğu da saklanmaz, çünkü BA dokümanında bunların korunması istenmiyor (bkz. S-10).

### 5.2 Oturum durumu (yalnızca bellekte)

```ts
interface SessionState {
  mainQueue: FactKey[];                          // B-04'ün oluşturduğu N soruluk ana liste
  mainIndex: number;
  step: number;                                  // sorulan toplam soru sayısı (tekrarlar dâhil)
  reAskQueue: { fact: FactKey; dueStep: number }[];
  reAskCount: Partial<Record<FactKey, number>>;
  lastFact: FactKey | null;
  results: { fact: FactKey; correct: boolean; timedOut: boolean; isReAsk: boolean }[];
  learnedThisSession: Set<FactKey>;              // FR-12
}
```

Oturum durumu kalıcı yazılmaz. Yarıda bırakılan oturumda yalnızca verilen cevaplar `facts` alanına zaten işlenmiş olur (FR-15). Oturumun kendisi kaldığı yerden devam ettirilmez, çünkü BA bunu istemiyor.

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Karıştırma: Fisher–Yates (Durstenfeld)

Karışık sıra (FR-05) ve zayıf soru seçimi (FR-09) için Fisher–Yates karıştırması kullanılır. Bu algoritma, her permütasyonun eşit olasılıkla çıktığı yansız bir sonuç üretir ve O(n) sürede çalışır. Yaygın hata, takas indeksini yanlış aralıktan seçmektir. Bu hata Sattolo algoritmasına dönüşür ve yanlılık yaratır ([Wikipedia: Fisher–Yates shuffle](https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle)). Uygulamada indeks `0..i` aralığından seçilir ve bu bir birim testle sabitlenir. Rastgele sayı üreteci parametre olarak verilir, böylece testlerde tohumlu (seeded) üreteç kullanılabilir.

### 6.2 Oturum oluşturma (FR-05, FR-06, FR-09)

```
oturumOlustur(tablolar, mod, N, kayitlar, rng):
  P = [ "a x b" | a ∈ tablolar (artan), b ∈ 1..10 ]
  eğer mod == SIRALI:
      ana = P'yi sırayla, N soruya ulaşana kadar baştan dolaşarak al         // geçici, bkz. S-03
  değilse (KARIŞIK):
      W = [ f ∈ P | durum(f) ≠ Öğrenildi ]          // "Yeni" + "Çalışılıyor" (FR-09)
      eğer W boş:
          ana = karıştır(P), gerekirse dolaşarak N'e tamamla
      değilse:
          k   = tavan(N / 2)                       // FR-09: en az %50
          zayıf = karıştır(W)'den k adet, W yetmezse dolaşarak (tekrarlı)   // geçici, bkz. S-04
          havuz = P \ W; boşsa havuz = W
          diğer = karıştır(havuz)'dan N−k adet, yetmezse dolaşarak
          ana = karıştır(zayıf + diğer)
      ana = bitişikleriAyır(ana)                   // FR-05: aynı soru art arda gelmez
  dön ana
```

- `bitişikleriAyır`, art arda gelen iki aynı olgudan ikincisini listede ondan farklı ilk olguyla takas eder. Tek tablo ve 20 soru seçildiğinde (10 olgu, 20 soru) tekrar kaçınılmazdır. Bu yordam tekrarların bitişik olmamasını sağlar.
- **Gerekçe:** Kota tabanlı basit bir yaklaşım seçildi. Leitner kutuları veya aralıklı tekrar zamanlaması gibi daha karmaşık yöntemler BA gereksinimlerinde yok. BA Varsayım 6, %50 oranının bir ürün kararı olduğunu belirtiyor. Oran tek bir sabitte (`WEAK_RATIO = 0.5`) tutulur.

### 6.3 Sonraki soruyu seçme ve oturum içi tekrar (FR-08)

```
sonrakiSoru(s):
  r = s.reAskQueue içinde dueStep ≤ s.step ve fact ≠ s.lastFact olan ilk öğe
  eğer r varsa: kuyruktan çıkar, dön r.fact (isReAsk = true)
  eğer s.mainIndex < N:
      f = s.mainQueue[s.mainIndex++]
      eğer f == s.lastFact: ana listedeki sonraki farklı öğeyle takas et
      dön f
  eğer reAskQueue boş değilse:
      // ana liste bitti, ama vadesi gelmemiş tekrar var
      dön araSoru(P \ {s.lastFact, bekleyen tekrarlar})   // sayıma girmez; geçici, bkz. S-05
  dön BİTTİ

cevapİşle(s, f, doğru):
  eğer !doğru ve s.reAskCount[f] < MAX_REASK (geçici: 2):
      s.reAskQueue.ekle({ fact: f, dueStep: s.step + 3 })   // arada en az 2 başka soru
      s.reAskCount[f]++
  s.step++; s.lastFact = f
```

- **"En az 2 soru sonra" yorumu:** Tekrar sorusundan önce en az 2 başka soru sorulur (`step + 3`). Bu, iki olası yorumun (1 ya da 2 ara soru) ikisini de karşılayan daha güvenli seçenektir (S-05).
- **Oturum sayacı:** US-01'e göre N sayısına FR-08 tekrarları dâhil değildir. Oturum, ana liste bittiğinde ve tekrar kuyruğu boşaldığında biter.
- **Sonsuz döngü koruması:** Aynı soru tekrar sorulduğunda yine yanlış cevaplanırsa yeniden kuyruğa girer, ama en fazla `MAX_REASK` kez. Bu sınır BA dokümanında yok. Oturumun bitmesini garanti etmek için konmuş bir tasarım parametresidir ve onay için BA'ya soruldu (S-06).

### 6.4 Öğrenme durumu ve yıldız (FR-10, FR-13, FR-14)

```
güncelle(kayıt, doğru):               // zaman aşımı → doğru = false (FR-14)
  kayıt.attempts++
  kayıt.correctStreak = doğru ? kayıt.correctStreak + 1 : 0
yıldızDenetle(t):
  eğer t ∉ stars ve ∀ b ∈ 1..10: durum("t x b") == Öğrenildi → stars.ekle(t); tebrik göster
```

- Oturum içi tekrar cevapları da sayılır, çünkü FR-10 yalnızca "son 3 cevap" diyor. Bunun öğrenme ölçütünü gevşetip gevşetmediği S-08'de soruldu.
- Yıldız bir kez verilince kalıcıdır. Yalnızca FR-16 sıfırlamasıyla silinir; tablodaki bir soru sonradan "Çalışılıyor"a dönse de yıldız kalır (FR-13'teki "kalıcı" ifadesine dayanır, S-09).
- Tebrik, yıldızı kazandıran cevabın geri bildiriminin ardından engellemeyen bir bildirim olarak gösterilir ve oturum sonu özetinde tekrarlanır.

### 6.5 Kalıcı yazma ve hata yönetimi (FR-15, NFR-10)

1. Cevap onaylanır, `güncelle()` ve `yıldızDenetle()` çalışır.
2. Tüm kayıt `JSON.stringify` ile `localStorage.setItem` üzerinden yazılır.
3. Ardından geri bildirim gösterilir.

Böylece uygulama geri bildirim ekranında kapansa bile son onaylanan cevap kaydedilmiş olur. `setItem`, depolama politikası nedeniyle `SecurityError` verebilir ([MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). Kota dolduğunda da hata fırlatılabilir; bu nedenle MDN yazma çağrılarını `try…catch` içine almayı önerir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Hata durumunda alıştırma kesilmez, ilerlemenin kaydedilemediği bir kez bildirilir. Bu bildirimin metni için bkz. S-11.

Uygulama ilk cevap kaydedildiğinde `navigator.storage.persist()` çağrısı yapar. İzin verilirse veri yalnızca kullanıcı isterse silinir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)).

### 6.6 Zamanlayıcı (FR-14)

Soru ekrana geldiğinde `t0 = performance.now()` alınır. Geri sayım göstergesi yaklaşık 250 ms aralıklarla güncellenir. Kalan süre her güncellemede `performance.now() − t0` farkından hesaplanır, böylece birikmiş zamanlayıcı kayması oluşmaz. Süre dolunca cevap "yanlış" olarak işlenir (6.4), doğru cevap gösterilir ve sonraki soruya geçilir. Süre kapalıyken (varsayılan) geri sayım öğesi hiç oluşturulmaz (US-06).

### 6.7 Yetişkin doğrulaması (FR-16)

Her açılışta rastgele bir "iki basamaklı (11–39) × tek basamaklı (3–9)" işlem sorulur ve cevap sayısal olarak girilir. Bu işlem, uygulamanın kapsadığı 10×10 tablosunun dışındadır. Amaç, FR-16'da yazıldığı gibi yanlışlıkla sıfırlamayı önlemektir, kesin güvenlik sağlamak değildir. Yöntemin yeterliliği S-12'de soruldu.

## 7. Performans

- **NFR-03 (geri bildirim ≤ 0,5 sn, sonraki soru ≤ 1 sn).** Cevaptan sonraki işlemler tamamen cihazda yapılır: en fazla 100 kayıt üzerinde güncelleme, birkaç KB'lık bir JSON yazma ve DOM güncellemesi. Ağ çağrısı yoktur (T-02). Bu nedenle hedefler, sınırın çok altında kalacak biçimde tasarlanmıştır.
- **Ölçüm:** Playwright E2E testlerinde onay tıklaması ile geri bildirim öğesinin görünmesi arasındaki süre ölçülür. Testler telefon ve tablet öykünmesiyle Chromium, Firefox ve WebKit motorlarında çalışır (T-09). Öykünme gerçek donanım hızını yansıtmaz. Kesin ölçüm, BA Açık soru 1 yanıtlandıktan sonra belirlenecek hedef cihaz sınıfından en az bir gerçek cihazda elle yapılır.
- **Sonraki soru gecikmesi:** Geri bildirim ile sonraki soru arasında bilinçli bir bekleme olacaksa (çocuğun doğru cevabı okuyabilmesi için) bu süre NFR-03'teki 1 sn bütçesinin içinde kalmalıdır. Yanlış cevapta doğru cevabın ne kadar süre gösterileceği bu bütçeyle çelişebilir (S-13).
- **İlk yükleme süresi:** BA dokümanında bununla ilgili bir gereksinim yok, bu nedenle hedef konmadı. Çatısız yaklaşım (T-04) paket boyutunu zaten küçük tutar.

## 8. Güvenlik ve gizlilik

| Konu | Karar | Gereksinim |
|---|---|---|
| Kişisel veri | Hiçbir ekranda ad, e-posta, konum veya yaş istenmez. Hesap yoktur. Depolanan veri yalnızca soru sayaçları, yıldızlar ve iki ayardır (Bölüm 5.1). | NFR-07 |
| Veri iletimi | Uygulama ilk yüklemeden sonra hiçbir sunucuya istek atmaz. Analitik, hata izleme veya çerez kullanılmaz. | NFR-07 |
| Üçüncü taraf içerik | Yazı tipi, simge ve script gibi tüm varlıklar uygulamayla birlikte paketlenir; CDN kullanılmaz. Barındırmada yalnızca kendi kökene izin veren bir İçerik Güvenliği Politikası (Content-Security-Policy) başlığı tanımlanır. Bu, sonradan reklam veya izleme kodu eklenmesini de teknik olarak engeller. | NFR-07, NFR-08 |
| Reklam ve satın alma | Reklam SDK'sı veya ödeme entegrasyonu yoktur. | NFR-08 |
| Barındırma kayıtları | Statik barındırma sağlayıcısı istek kayıtlarında IP adresini tutabilir. Bunun KVKK açısından değerlendirilmesi BA Açık soru 12'ye bağlıdır (S-14). | NFR-07 |
| Yanlışlıkla sıfırlama | Yetişkin doğrulaması ve ayrı bir onay adımı (B-10, 6.7). | FR-16 |
| Gizli sekme | Gizli sekmede `localStorage` verisi son gizli sekme kapanınca silinir ([MDN: localStorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage)). Bu bir gizlilik sorunu değil, bir risktir (Bölüm 10, R-03). | FR-15 |

## 9. Test stratejisi

| Düzey | Araç | Kapsam | Gereksinim / US |
|---|---|---|---|
| Birim | Vitest | **Soru Motoru:** yalnızca seçili tabloların sorulması; sıralı ve karışık sıra; varsayılan değerler; N ve tekrarlar dışındaki sayım; aynı sorunun art arda gelmemesi; zayıf soru oranı ≥ %50 (tohumlu üreteçle 1.000 rastgele senaryo); tekrar vadesi ≥ 3 adım; `MAX_REASK` sonrası oturumun bitmesi. | FR-01, FR-05, FR-06, FR-08, FR-09; US-01, US-04 |
| Birim | Vitest | **Öğrenme Durumu:** Yeni → Çalışılıyor → Öğrenildi geçişleri; Öğrenildi durumundayken yanlış cevap; zaman aşımının yanlış sayılması; yıldızın verilmesi ve kalıcılığı; "bu oturumda öğrenilenler" listesi. | FR-10, FR-12, FR-13, FR-14; US-04, US-05 |
| Birim | Vitest | **Fisher–Yates:** indeks aralığı, permütasyon dağılımı (küçük n için ki-kare testi). | FR-05, FR-09 |
| Birim | Vitest | **Depolama:** şema okuma/yazma, bozuk JSON, `setItem` hatası (taklit nesne ile), sıfırlamanın yalnızca onaydan sonra silmesi. | FR-15, FR-16, NFR-10 |
| Birim | Vitest | **Metinler:** `tr.ts` içindeki yönerge metinlerinin en fazla 8 kelime olması (otomatik sayım). Geri bildirim metinleri ise elle gözden geçirilir. | NFR-01, NFR-09 |
| E2E | Playwright (Chromium, Firefox, WebKit; telefon ve tablet profilleri) | US-01–US-07 kabul kriterlerinin her biri ayrı bir test olur. İnceleme ekranındaki toplama gösterimi de test edilir. | US-01–US-07; FR-02, FR-03, FR-04, FR-07, FR-11, FR-14, FR-17 |
| E2E | Playwright | **Kalıcılık:** cevap onayından hemen sonra sayfa kapatılıp yeniden açılır; ızgara, yıldızlar ve ayarlar karşılaştırılır. | FR-15, FR-17, NFR-10 |
| E2E | Playwright | **Yanıt süresi:** onay ile geri bildirim arası ≤ 500 ms, geri bildirim ile sonraki soru arası ≤ 1000 ms. | NFR-03 |
| E2E | Playwright | **Dokunuş sayısı:** ilk açılıştan "Başla"ya en fazla 3 tıklama. | NFR-02 (otomatik kısım) |
| Erişilebilirlik | `@axe-core/playwright` + elle denetim | Her ekranda WCAG etiketli kurallarla tarama (kontrast dâhil). Dokunma hedefi boyutları DOM ölçümüyle ≥ 24×24 CSS px olarak doğrulanır. Durumların renk olmadan ayırt edilebilirliği gri tonlamalı ekran görüntüsüyle elle denetlenir. Playwright belgeleri otomatik testlerin tüm WCAG ihlallerini bulamayacağını, bu yüzden elle değerlendirme ve kullanıcı testiyle birleştirilmesi gerektiğini belirtir ([Playwright: Accessibility testing](https://playwright.dev/docs/accessibility-testing)). | NFR-04, NFR-05, NFR-06 |
| Kullanılabilirlik | Elle, en az 5 çocukla | NFR-02'de tanımlanan doğrulama: en az 4 çocuğun yardımsız başarması. Gözlem sırasında geri bildirim tonu ve süre ayarı da izlenir. | NFR-02, NFR-09 |
| Gerçek cihaz | Elle | Hedef cihaz sınıfında yanıt süresi ölçümü ve Safari'de ana ekran kurulumu ile `persist()` davranışı. | NFR-03, FR-15 |
| İnceleme | Elle (kod + ağ izleme) | Tarayıcı geliştirici araçlarında ilk yüklemeden sonra hiç ağ isteği olmadığı ve üçüncü taraf kaynak yüklenmediği denetlenir. | NFR-07, NFR-08 |

## 10. Riskler

| # | Risk | Etki | Olasılık | Azaltma | İlgili gereksinim |
|---|---|---|---|---|---|
| R-01 | **Safari'de proaktif silme.** Siteler arası izleme önleme açıkken, bir köken yedi gün boyunca kullanıcı etkileşimi almazsa script ile oluşturulmuş verisi silinir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Bir hafta ara veren çocuğun ilerlemesi kaybolabilir. | Yüksek | Orta (iOS/Safari kullanımına bağlı) | `persist()` isteği. WebKit bu isteği "sitenin ana ekran web uygulaması olarak açılıp açılmadığı gibi" sezgilere göre onaylar ([WebKit: Updates to Storage Policy](https://webkit.org/blog/14403/updates-to-storage-policy/)). Bu nedenle iOS'ta ana ekrana kurulum önerilir. Ancak okunan kaynaklar, ana ekran uygulamalarının yedi günlük kuraldan muaf olduğunu açıkça doğrulamıyor. Bu risk kabul edilemezse yerel uygulama veya dışa aktarma gibi bir yol gerekir, ki bu da BA kararıdır (S-02, S-15). | FR-15, NFR-10 |
| R-02 | **Depolama sıkışması.** Kalıcı olmayan (best-effort) kökenler depolama baskısında en az kullanılan önce silinir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). | Yüksek | Düşük (aynı kaynağa göre tarayıcılar veriyi çok nadiren siliyor) | `persist()` isteği; küçük veri boyutu. | FR-15 |
| R-03 | **Gizli sekme veya depolamanın engellenmesi.** Veri sekme kapanınca silinir ya da `SecurityError` oluşur. | Orta | Düşük | Hata yakalanır, alıştırma sürer ve kullanıcı bir kez bilgilendirilir (6.5). | FR-15, NFR-10 |
| R-04 | **Firefox'ta `persist()` izin penceresi.** Firefox bu istekte kullanıcıya bir açılır pencere gösterir ([MDN: Storage quotas](https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria)). Çocuk pencereyi anlamayabilir. | Düşük | Orta | İstek ilk cevaptan sonra yapılır. Reddedilmesi uygulamayı etkilemez. Kullanılabilirlik testinde gözlemlenir. | FR-15, NFR-02 |
| R-05 | **Platform kararı değişebilir.** BA hedefin mağaza uygulaması olduğunu belirtirse T-01 yeniden ele alınır. | Yüksek | Bilinmiyor | İş mantığı (B-04, B-05) DOM'dan bağımsız tutulduğu için taşınabilir. | NFR-03, NFR-05 |
| R-06 | **FR-09 kotası ile küçük havuzlar.** Az sayıda zayıf soru varken %50 kotası aynı soruların sık tekrarına yol açar. Bu da Can personasındaki sıkılma sorununu yeniden yaratabilir. | Orta | Orta | Oran ve dolaşma davranışı parametrik tutuldu. Davranış BA'ya soruldu (S-04). | FR-09 |
| R-07 | **Yetişkin doğrulamasının aşılması.** İleri yaştaki bir çocuk işlemi çözebilir. | Düşük | Orta | Amaç yalnızca yanlışlıkla sıfırlamayı önlemek (FR-16). Ek olarak ayrı bir onay adımı var. | FR-16 |
| R-08 | **Kullanıcının tarayıcı verisini temizlemesi.** İlerleme silinir. | Orta | Düşük | Cihaz dışına yedekleme kapsam dışı. Kabul edilen bir risk olarak BA'ya bildirildi (S-15). | FR-15 |

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | Tablo seçimi (1–10, çoklu, seçimsiz başlatılamaz) | B-01; B-04'te soru havuzu P | 4, 6.2 |
| FR-02 | Tabloyu inceleme (10 satır) | B-02 | 4 |
| FR-03 | Tekrarlı toplama karşılığı gösterimi | B-02 (toplama dizgesi koddan üretilir) | 4 |
| FR-04 | "a × b = ?" sorusu, sayısal tuş takımı, en fazla 3 hane | B-03; T-07 ekran tuş takımı | 3, 4 |
| FR-05 | Sıralı / karışık sıra, varsayılan karışık, art arda aynı soru yok | B-01, B-04; Fisher–Yates; `bitişikleriAyır` | 6.1, 6.2, 6.3 |
| FR-06 | Oturum uzunluğu 5/10/20, varsayılan 10 | B-01, B-04 | 6.2, 6.3 |
| FR-07 | Anında geri bildirim, yanlışta doğru cevap, suçlayıcı olmayan metin | B-03, B-13; yerel işlem (T-02) | 4, 7 |
| FR-08 | Yanlışın oturum içinde ≥ 2 soru sonra tekrarı | B-04; tekrar kuyruğu (`step + 3`), `MAX_REASK` | 6.3 |
| FR-09 | Karışık modda zayıf sorulara ≥ %50 öncelik | B-04; zayıf soru kotası `WEAK_RATIO` | 6.2 |
| FR-10 | Soru bazında Yeni / Çalışılıyor / Öğrenildi durumu | B-05; `FactRecord` ve türetilmiş durum | 5.1, 6.4 |
| FR-11 | 10×10 ilerleme ızgarası, renk + simge | B-08, B-14 | 4 |
| FR-12 | Oturum sonu özeti | B-07; `SessionState.results`, `learnedThisSession` | 4, 5.2 |
| FR-13 | Tablo tamamlanınca tebrik ve kalıcı yıldız | B-05 `yıldızDenetle`; `stars` alanı; B-01, B-08 gösterimi | 5.1, 6.4 |
| FR-14 | İsteğe bağlı süre (Kapalı / 10 / 7 / 5 sn), dolunca yanlış | B-06, B-09; `timerSeconds`; `performance.now()` | 4, 6.6 |
| FR-15 | İlerlemenin korunması, yarıda kalan oturumun işlenmesi | B-12; T-06 `localStorage` + `persist()`; her cevapta yazma | 3, 5, 6.5, 10 |
| FR-16 | Yetişkin doğrulamalı ve onaylı sıfırlama | B-09, B-10, B-12 | 4, 6.7, 8 |
| FR-17 | Ses efektlerini kapatma, ayarın korunması | B-09, B-11; `settings.soundOn` | 4, 5.1 |
| NFR-01 | Arayüz Türkçe, yönergeler en fazla 8 kelime | B-13; kelime sayısı birim testi | 4, 9 |
| NFR-02 | En fazla 3 dokunuşla oturum başlatma | B-01 (varsayılanlarla 2 dokunuş); E2E ve kullanılabilirlik testi | 4, 9 |
| NFR-03 | Geri bildirim ≤ 0,5 sn, sonraki soru ≤ 1 sn | T-02 (ağsız işlem); E2E ve gerçek cihaz ölçümü | 3, 7, 9 |
| NFR-04 | Kontrast 4,5:1 / 3:1 | B-14; axe taraması | 4, 9 |
| NFR-05 | Dokunma hedefi ≥ 24×24 CSS px | T-07, B-14; DOM ölçüm testi | 3, 4, 9 |
| NFR-06 | Durumlar yalnızca renge dayanmaz | B-03, B-08, B-14 (simge + metin + erişilebilir etiket) | 4, 9 |
| NFR-07 | Kişisel veri yok, hesap yok | T-02 sunucusuz mimari; Bölüm 5.1 veri kapsamı; CSP | 3, 5, 8 |
| NFR-08 | Reklam ve uygulama içi satın alma yok | T-02; SDK yok; CSP | 3, 8 |
| NFR-09 | Hatada ceza yok, oturum kesilmez | B-03, B-11, B-13 | 4, 6.3 |
| NFR-10 | Beklenmedik kapanmada son onaylı cevaba kadar ilerleme korunur | B-12; geri bildirimden önce yazma | 6.5, 9 |

## 12. BA'ya geri sorular

Aşağıdaki sorular BA dokümanındaki eksik, belirsiz veya çelişkili noktalardır. Belirtilen geçici kararlar yalnızca tasarımın ilerleyebilmesi içindir; yanıta göre değiştirilecektir.

1. **S-01 (BA Açık soru 2, çevrimdışı çalışma):** Uygulama internetsiz açılabilmeli mi? Evet ise yalnızca bir service worker ve önbellek eklenir. Mimarinin geri kalanı değişmez.
2. **S-02 (BA Açık soru 1, platform):** Hedef cihaz ve dağıtım şekli nedir? Uygulama mağazasından indirilmesi bir iş gereksinimi mi? Geçici karar: kurulabilir web uygulaması (T-01).
3. **S-03 (FR-05 + FR-06, sıralı mod):** Birden çok tablo seçildiğinde ve soru sayısı toplam olgu sayısından farklıysa (örn. 3 tablo = 30 olgu, oturum 10 soru) sıralı modda hangi sorular sorulmalı? Geçici karar: tablolar artan sırada, ×1'den başlanarak N soru alınır ve gerekirse başa dönülür. Bu kararla 3 tablo seçen bir kullanıcı 10 soruluk oturumda yalnızca ilk tabloyu görür.
4. **S-04 (FR-09, küçük zayıf soru havuzu):** Öğrenilmemiş soru sayısı oturumun yarısından azsa (örn. 2 zayıf soru, 10 soruluk oturum) %50 kotası aynı soruların tekrarıyla mı doldurulmalı, yoksa mevcut zayıf soruların tamamını bir kez sormak yeterli mi? Geçici karar: tekrarla doldurulur, aynı soru art arda gelmez.
5. **S-05 (FR-08, tekrar aralığı ve oturum sonu):** "En az 2 soru sonra" ifadesi, arada en az 2 başka soru olması mı demek? Yanlış cevap oturumun son sorularında verilirse oturum ara sorularla mı uzatılmalı, yoksa tekrar beklemeden mi sorulmalı? Geçici karar: arada en az 2 soru olur; gerekirse N sayımına girmeyen ara sorular eklenir.
6. **S-06 (FR-08, tekrarda yine yanlış):** Tekrar sorulan soru yine yanlış cevaplanırsa yeniden kuyruğa mı girmeli? Bir üst sınır olmalı mı? Geçici karar: oturum başına soru başına en fazla 2 tekrar.
7. **S-07 (FR-10 / FR-11, olgu tanımı):** 3×4 ve 4×3 ayrı sorular mı? Geçici karar: ayrı (10×10 ızgara ve FR-13'teki "bir tablonun 10 sorusu" ifadesine dayanarak).
8. **S-08 (FR-10 + FR-08, öğrenildi ölçütü):** Doğru cevabın gösterilmesinden yalnızca 2–3 soru sonra gelen oturum içi tekrar cevapları "son 3 cevap" sayımına dâhil mi? Dâhilse bir soru tek oturumda kısa süreli bellekle "Öğrenildi"ye geçebilir. Geçici karar: tüm cevaplar sayılır.
9. **S-09 (FR-13, yıldızın kalıcılığı):** Yıldız kazanılan bir tablodaki soru sonradan "Çalışılıyor"a dönerse yıldız kalmalı mı? Geçici karar: kalır; yalnızca sıfırlamayla silinir.
10. **S-10 (NFR-02 / FR-01, seçimlerin hatırlanması):** Son seçilen tablolar, sıra ve oturum uzunluğu uygulama yeniden açıldığında hatırlanmalı mı? BA Açık soru 5'teki varsayılan tablo seti ile birlikte ilk açılıştaki dokunuş sayısını etkiler. Geçici karar: hatırlanmaz (yalnızca FR-17 ayarı ve süre ayarı saklanır).
11. **S-11 (FR-15, kaydetme hatası):** Cihaz depolamaya izin vermediğinde (gizli sekme, engellenmiş depolama) kullanıcıya bir bildirim gösterilmeli mi? Gösterilecekse metni ne olmalı? Geçici karar: alıştırma sürer, kısa bir bilgi metni bir kez gösterilir.
12. **S-12 (FR-16, yetişkin sorusu):** "Basit yetişkin sorusu" için iki basamaklı × tek basamaklı çarpma (örn. 23 × 7) kabul edilebilir mi? Hedef kitlede 4. sınıf öğrencileri de olduğundan bazı çocuklar bunu çözebilir.
13. **S-13 (FR-07 + NFR-03, doğru cevabın gösterim süresi):** Yanlış cevapta doğru cevap ekranda ne kadar kalmalı? Otomatik geçiş mi olmalı, yoksa çocuk "Devam"a mı basmalı? NFR-03'teki "sonraki soru ≤ 1 sn" ölçütü, otomatik geçişte doğru cevabın okunması için 1 saniyeden az süre bırakır. Bu ölçüt "Devam"a basıldıktan sonrası için mi geçerli?
14. **S-14 (BA Açık soru 12, yasal uyum ve barındırma):** Barındırma sağlayıcısının istek kayıtlarında IP adresi tutması KVKK açısından NFR-07 ile uyumlu mu? Barındırma sağlayıcısı veya bölgesi için bir kısıt var mı?
15. **S-15 (FR-15, ilerleme kaybı riski):** Safari'de uzun süre kullanılmayınca veya tarayıcı verisi temizlenince ilerlemenin silinme riski (R-01, R-08) kabul edilebilir mi? Kabul edilemezse ilerlemeyi dosyaya dışa/içe aktarma gibi bir gereksinim mi eklenmeli, yoksa platform kararı mı değişmeli?
16. **S-16 (FR-17, ses efektlerinin varlığı):** Üründe ses efekti olacak mı? FR-17 "varsa" diyor. Olacaksa hangi olaylarda (doğru cevap, yıldız) çalmalı? NFR-09 gereği yanlış cevapta olumsuz ses olmayacak. Ses yoksa FR-17 ayarı gösterilmez.
17. **S-17 (BA Açık soru 7, cevap biçimi):** Çoktan seçmeli cevap kapsama alınırsa yeni bir FR gerekir ve B-03 ile B-04 değişir. Mevcut tasarım yalnızca yazarak cevabı kapsar.
18. **S-18 (BA Açık soru 11, tuş boyutu):** Tuş takımı için kesin en küçük boyut nedir? B-14'te tek bir değişken olarak tutuluyor ve şimdilik 24×24 CSS px alt sınırı uygulanıyor.

## Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| 1 | MDN: Storage quotas and eviction criteria | https://developer.mozilla.org/en-US/docs/Web/API/Storage_API/Storage_quotas_and_eviction_criteria | `localStorage` için köken başına yaklaşık 5 MiB sınırı; best-effort ve kalıcı depolama farkı; depolama baskısında en az kullanılanın silinmesi; Safari'nin yedi gün etkileşimsiz kökenlerin script verisini silmesi; `persist()` davranışı (Firefox'ta açılır pencere); `QuotaExceededError` için `try…catch` önerisi. |
| 2 | MDN: Window.localStorage | https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage | Verinin oturumlar arasında süresiz saklanması; yalnızca dize saklanması; `SecurityError` koşulları; gizli sekmede verinin silinmesi. |
| 3 | WebKit: Updates to Storage Policy | https://webkit.org/blog/14403/updates-to-storage-policy/ | WebKit kotaları ve silme koşulları; `persist()` isteğinin ana ekran web uygulaması olma gibi sezgilerle onaylanması. |
| 4 | MDN: Progressive web apps | https://developer.mozilla.org/en-US/docs/Web/Progressive_web_apps | PWA'nın tek kod tabanından birden çok platformda çalışması, kurulabilmesi ve service worker ile çevrimdışı çalışabilmesi. |
| 5 | MDN: inputmode | https://developer.mozilla.org/en-US/docs/Web/HTML/Global_attributes/inputmode | `inputmode="numeric"` özniteliğinin yalnızca sanal klavye ipucu vermesi, geçerlilik kuralı uygulamaması. |
| 6 | Vite: Deploying a Static Site | https://vite.dev/guide/static-deploy | `vite build` çıktısının `dist` klasöründe statik dosyalar olması ve statik barındırmalara yüklenebilmesi. |
| 7 | Vitest: Getting Started | https://vitest.dev/guide/ | Vitest'in Vite üzerine kurulu olması ve `vite.config.*` dosyasını okuması. |
| 8 | Wikipedia: Fisher–Yates shuffle | https://en.wikipedia.org/wiki/Fisher%E2%80%93Yates_shuffle | Yansız permütasyon, O(n) süre, indeks aralığı hatasının Sattolo algoritmasına dönüşerek yanlılık yaratması. |
| 9 | MDN: Performance.now() | https://developer.mozilla.org/en-US/docs/Web/API/Performance/now | Yüksek çözünürlüklü ve monoton zaman ölçümü; sistem saati değişikliklerinden etkilenmemesi. |
| 10 | Playwright: Browsers | https://playwright.dev/docs/browsers | Chromium, Firefox ve WebKit motorlarının desteklenmesi. |
| 11 | Playwright: Emulation | https://playwright.dev/docs/emulation | Telefon ve tablet profilleriyle cihaz öykünmesi (viewport, dokunma). |
| 12 | Playwright: Accessibility testing | https://playwright.dev/docs/accessibility-testing | `@axe-core/playwright` entegrasyonu; otomatik testlerin tüm WCAG ihlallerini bulamaması ve elle değerlendirme önerisi. |