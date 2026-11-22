# Karşılaştırma: Persona v1 → v2 (Aşama 3)

- **Tarih:** 2026-11-22
- **Uygulanan değişiklikler:** `eval/degisiklik-gunlugu.md` (O-1, O-5, BA-1, BA-2, TA-1, TA-2 ve CLAUDE.md üzerine yazma kuralı)
- **Ölçüt:** `eval/kontrol-listesi.md`

## 0. Test düzeni

| Grup | Persona | Girdi | Çalıştırma | Çıktı |
|---|---|---|---|---|
| BA v1 (Aşama 2) | ba v1 | Görev metni | 3 × 2 senaryo | `ba-<senaryo>-v1-run{1,2,3}.md` |
| TA v1 (Aşama 2) | ta v1 | BA v1 (çarpım run2, duplicate run3) | 2 × 2 senaryo | `ta-<senaryo>-v1-run{1,2}.md` |
| **BA v2** | ba v2 | Aynı görev metni (birebir) | 2 × 2 senaryo | `ba-<senaryo>-v2-run{1,2}.md` |
| **TA v2, v1 girdi** | ta v2 | Aşama 2'deki **aynı** BA v1 dosyaları | 1 × 2 senaryo | `ta-<senaryo>-v2-v1girdi-run1.md` |
| **TA v2, v2 girdi** | ta v2 | En iyi BA v2 (çarpım run1, duplicate run1) | 1 × 2 senaryo | `ta-<senaryo>-v2-run1.md` |

**Doğrulamalar (8/8 v2 çalıştırması):**
- Her çalıştırma ayrı ve yeni bir `claude -p` oturumunda, tek bir subagent çağrısıyla yapıldı.
- Subagent'a giden görev metni birebir aynı. Çıktı, subagent'ın son mesajıyla aynı.
- Doküman başlığında `ba v2` / `ta v2` yazıyor; yani v2 yüklenmiş, yeni oturum açmaya gerek kalmadı.
- Kayıt `'x'` (yalnızca yeni dosya) moduyla yapıldı; mevcut hiçbir dosyanın üzerine yazılmadı (`git diff HEAD` boş).

**Süreç olayları:**
1. İlk BA v2 başlatmasında çalıştırma betiğine çalıştırma izni verilmemişti. Hiçbir agent çalışmadı, hiçbir dosya oluşmadı; izin verilip yeniden başlatıldı.
2. Çalışma sırasında kullanım limitine takıldık. Limit, subagent'lar işini bitirdikten sonra ana oturumun yalnızca son "TAMAM" adımına denk geldi. Altı subagent'ın altısı da `completed` durumundaydı, çıktılar etkilenmedi.

**En iyi BA v2 seçimi ((c) adımı için):**

| Senaryo | Seçim | Neden |
|---|---|---|
| Çarpım | run1 | 13 FR ile en dar MVP. Bilgiler birincil kaynaktan, sayfa numaralı alıntıyla alınmış (MEB program PDF'i, STA MTC çerçevesi). run2 de 29 puan alıyor, ama sesli okuma ve seri gibi C önceliğindeki ekler yüzünden 16 FR'ye çıkıyor. |
| Duplicate | run1 | 16 FR, dar MVP, platform politikaları eksiksiz. run2 bağlamla aktarmada daha zengin, ama 18 FR'si var ve kapsamı genişleten FR-18 (analiz verisini silme) içeriyor. |

---

## 1. Puanlar (kendi değerlendirmem)

### 1.1 BA (/30)

| Senaryo | v1 run1 | v1 run2 | v1 run3 | **v1 ort.** | v2 run1 | v2 run2 | **v2 ort.** |
|---|---|---|---|---|---|---|---|
| Çarpım | 27 | 28 | 26 | **27,0** | 29 | 29 | **29,0** |
| Duplicate | 28 | 28 | 29 | **28,3** | 29 | 29 | **29,0** |

**v2 kriter gerekçeleri (kısa):**
- **Çarpım run1 (29):**
  - BA-1 = 5: 4 zorunlu başlığın hepsi var; MEB PDF ve STA çerçevesi birincil kaynak.
  - BA-2 = 5: MTC 4/4 doğrulandı; Hechinger'de "haftada 3 kez" ifadesi küçük bir nüans.
  - BA-3 = 4: FR-07'deki "daha sık sorulmalı" sayısal değil.
  - BA-4, BA-5, BA-6 = 5.
- **Çarpım run2 (29):**
  - BA-1 = 5: COPPA, KVKK ve akademik makale dahil.
  - BA-2 = 5: EdWeek ve TAAD doğrulandı.
  - BA-3 = 5.
  - BA-4 = 4: 16 FR; sesli okuma ve rozet C önceliğinde.
  - BA-5, BA-6 = 5.
- **Duplicate run1 (29):**
  - BA-2 = 4: arXiv'deki "%90" bulgusu pHash satırına bağlanmış, ama özette algısal hash yalnızca ima ediliyor.
  - Diğer kriterler 5.
- **Duplicate run2 (29):**
  - BA-4 = 4: 18 FR; FR-18 ek bir özellik.
  - Diğer kriterler 5. Bağlamla aktarma en iyi bu dokümanda: veri seti, eşik ve koşul her iddiada yazılı.

### 1.2 TA (/25)

| Senaryo | v1 run1 | v1 run2 | v2 (v1 girdi) | v2 (v2 girdi) |
|---|---|---|---|---|
| Çarpım | 23 | 24 | 23 | 24 |
| Duplicate | 24 | 25 | 25 | 24 |

**v2 kriter gerekçeleri:**
- **Çarpım v2 (v1 girdi):**
  - TA-2 = 4: fiziksel klavye girişi BA'ya sorulmadan hâlâ ekleniyor.
  - TA-4 = 4: Playwright ile 3 tarayıcı motorunda uçtan uca test, v1 run1'deki gibi ağır.
  - Diğer kriterler 5.
- **Çarpım v2 (v2 girdi):**
  - TA-4 = 4: geçici platform kararı Flutter + SQLite. Gerekçeli (Safari'nin veri silme riski, ebeveyn kapısı, mağaza çocuk politikaları), ama "basit" ürün için web çözümünden daha ağır.
  - Diğer kriterler 5. Ancak bkz. yeni sorun YS-1.
- **Duplicate v2 (v1 girdi):** Tüm kriterler 5. Not: kör değerlendirici bu sürüme TA-1 = 4 verdi (bkz. 4.3).
- **Duplicate v2 (v2 girdi):** TA-1 = 4. NFR-05 (50.000 fotoğraf) karşılanıyor ama riskli: aday araması BK-ağacı olmadan, en kötü durumda O(n²) (bkz. YS-5). Diğer kriterler 5.

**Yorum:** TA toplam puanı neredeyse değişmedi. v1 TA zaten 23–25 aralığındaydı. v2'nin etkisi puandan çok **gerekçe yapısında** ve **kaynak doğruluğunda** görülüyor (bkz. 2.5 ve 2.6).

---

## 2. Uygulanan değişikliklerin etkisi

| # | Değişiklik | v1'deki sorun | v2'de çözüldü mü? | Kanıt |
|---|---|---|---|---|
| O-1 | Yanıt yalnızca doküman | BA 4/6, TA 1/4 çalıştırmada doküman dışı not; yanlış dosya adı önerisi; Aşama 1 notuna atıf | **Evet** | Program kontrolü: 8/8 v2 çıktısı `# ` başlığıyla başlıyor ve son bölümle bitiyor. Doküman dışı not 0/8. Açılamayan kaynaklar artık `Kaynaklar` veya `Varsayımlar` bölümünün içinde (ör. `ba-duplicate-finder-v2-run1.md` V-11). Kör değerlendirici de v2 dosyalarının hiçbirinde doküman dışı not bulmadı; bulduğu tek not v1 TA'ya aitti. Dosya adı önerisi ve proje sürecine atıf 0/8. |
| O-5 | İddiaların bağlamıyla aktarılması | 24 linkten 4'ünde kısmi destek: EdWeek'te karşılaştırma koşulu yanlış, PMC'de katılımcı ve grup bilgisi kaymış, imagededup'ta veri seti yok | **Büyük ölçüde evet** | 8 kaynak iddiası kontrol edildi (2 PDF dahil, PDF'ler `pdftotext` ile okundu). **Olgusal hata 0.** 2 küçük nüans kaldı: (1) Hechinger: "haftada 3 kez 5 dk" ifadesi aslında tek haftada toplam 3 oturum; (2) arXiv 2304.02296'daki %90 bulgusunun algısal hashle ilişkisi özette yalnızca ima ediliyor. İşaretlerin kullanımı: "(kaynaktaki verilerden hesaplandı)" (duplicate run1: aHash "12 katından fazla", pHash "yaklaşık 1,9 katı"), "firma beyanı", "muhabirin aktarımı", "tek kullanıcı yorumu", "yazarın kendi testinde". Koşullu aktarım örneği: duplicate run2 "64 bit hash, Hamming eşiği 0 ile… FP 0, FN 2"; tablo değerleri kaynakla birebir doğrulandı. Birincil kaynak tercihi: MEB program PDF'i, STA çerçeve PDF'i, arXiv HTML tam metinleri, KVKK kanun metni. |
| BA-1 | Zorunlu araştırma başlıkları | Duplicate run2'de Google Play politikası ve silme davranışı yoktu; çarpım run1'de olağan müfredat bulunamamıştı | **Evet** | 4/4 BA v2'de 4 başlık ayrı alt bölüm: mevcut çözümler, alan standartları, platform/mağaza politikaları, yöntemler. **Çarpım** v1'de mağaza politikası ve kişisel veri hiç ele alınmıyordu; v2'nin ikisinde de Apple Kids (1.3, 5.1.4), Google Play Families ve KVKK var (run2'de COPPA da). **Duplicate** 4/4'te Play izin politikası, Android silme, iOS sınırlı erişim ve KVKK var. Yerleşik çözüm kontrolü yapılmış: çarpımda yerleşik çözüm bulunamadığı V-02 ve V-07'de gerekçelendirilmiş. Müfredat 2/2 çarpımda MEB 2. ve 3. sınıf olarak birincil kaynaktan doğrulanmış. |
| BA-2 | FR'lerde MoSCoW | v1'de öncelik 2/6 (yalnızca duplicate) | **Evet** | 4/4 BA v2'de FR tablosunda "Öncelik" sütunu var; önceliksiz FR 0. Dağılım M/S/C: çarpım 6/5/2 ve 7/5/4, duplicate 10/4/2 ve 10/6/2. 4/4'te bölüm başında "M = MVP sınırı" cümlesi var. Duplicate run2'de W maddeleri açıkça "Kapsam dışı (MoSCoW: W)" olarak yazılmış. Kör değerlendirici de MoSCoW'u v2 BA'ların ayırt edici üstünlüğü olarak saydı. |
| TA-1 | Kritik kararlarda seçim ölçütlü karşılaştırma | Duplicate'te birebir tespit, gruplama ve eşikler çalıştırmalar arasında değişiyordu | **Kısmen** | **Yapı oluştu:** 4/4 TA v2'de kritik kararlar "Seçim ölçütleri (Ö1…Ön) → seçenek tablosu → Karar" biçiminde. Çarpımda 5'er, duplicate'te 6–7 kritik karar var; her ölçüt bir FR/NFR ID'sine bağlı. Kör değerlendirici bunu çarpımda TA-3 = 5'in nedeni olarak gösterdi. **Sonuç değerleri hâlâ değişken** (bkz. 3.2): gruplama tam bağlantı ↔ lider kümeleme; eşikler 4/8/12 ↔ dHash 3/6/10 ve pHash 2/4/8; birebir ön filtre algısal mesafe ↔ dosya boyutu. Yine de v2'nin **iki çalıştırması da** union-find'ı aynı ölçütle ("grupta birbirine benzemeyen fotoğraf olmamalı", NFR-04b) reddetti. Yani gerekçe yakınsadı, ama ölçütü sağlayan birden fazla seçenek olduğunda seçim hâlâ değişiyor. **Sınırlılık:** v2 TA'nın iki çalıştırmasının girdisi farklıydı (v1 ve v2 BA). Aynı girdiyle 2 v2 çalıştırması yapılmadığı için tutarlılık kanıtı sınırlı. |
| TA-2 | Kütüphane izin, sürüm ve kısıt kontrolü | Duplicate v1 run1 Android 13'te video izni gerekliliğini kaçırmıştı | **Evet** | 2/2 duplicate TA v2'de `READ_MEDIA_IMAGES` + `READ_MEDIA_VIDEO` birlikte gerekliliği yakalanmış ve risk olarak yazılmış (v2-v1girdi R-01, v2-run1 R1). Teknoloji tablolarında İzinler / En düşük sürüm / Bilinen kısıtlar sütunları var. **v1'de görülmeyen yeni tespitler:** `deleteWithIds` için photo_manager changelog ile taşıma kılavuzu arasında çelişki (v1 girdili sürüm); sqflite iOS 12 / minSdk 19 ve transaction kısıtları; `image` paketinde HEIC olmaması; `photo_manager_image_provider`'ın 2 yıldır güncellenmemesi; iOS'ta dosya boyutu sütununun olmaması. Çarpımda da Vitest/Playwright Node gereksinimleri ve MDN kısıtları tabloya girmiş. |
| CLAUDE.md | `outputs/` üzerine yazma yasağı | Aşama 2'de 4 BA dosyası yanlışlıkla üzerine yazılmıştı | **Evet** | Kayıt betiği hedef dosya varsa durdu (`'x'` modu). 8 kayıtta üzerine yazma 0; her kayıttan sonra `git diff HEAD` boş. |

### 2.1 Örnek kanıt: kaynakla doğrulanan v2 tespitleri

| Tespit | v1 | v2 | Kaynak kontrolü |
|---|---|---|---|
| Safari'de ana ekrana eklenen web uygulamalarının 7 günlük silme sayacından muaf olması | İki v1 TA da "kaynak teyit etmiyor" diyerek riski açık bıraktı | Çarpım TA v2 (v1 girdi) WebKit blogunu bulup riski kaynağa bağladı | ✅ webkit.org/blog/10218: "Web applications added to the home screen are not part of Safari and thus have their own counter of days of use." |
| photo_manager'ın Android 13'te video iznini de istemesi | 1/2 TA'da vardı | 2/2 TA'da var | ✅ pub.dev: "READ_MEDIA_IMAGES and READ_MEDIA_VIDEO are both required whether requesting images or videos." |
| MTC'de 6 saniyelik sınırın dayanağı | v1 BA'larda yok | Çarpım BA v2 run1: "1124 öğrenci, 3 süre sınırı denendi" | ✅ STA çerçeve PDF'i: "1124 pupils took part and 3 time limits were trialled." |

---

## 3. Tutarlılık

### 3.1 BA (aynı görev, farklı çalıştırmalar)

| Boyut | v1 (3 çalıştırma) | v2 (2 çalıştırma) |
|---|---|---|
| Bölüm yapısı | 10/10 aynı | 10/10 aynı; bölüm 3 dört alt başlıkla da aynı |
| FR önceliği | 2/6 | 6/6 → **4/4** |
| Platform ve mağaza politikası (çarpım) | 0/3 | **2/2** |
| Play politikası (duplicate) | 2/3 | **2/2** |
| Müfredat (çarpım) | Tutarsız: İYEP, 2. sınıf, 2.+3. sınıf | **2/2** MEB 2. ve 3. sınıf |
| FR sayısı | Çarpım 14–17, duplicate 14–16 | Çarpım 13–16, duplicate 16–18 |
| Varsayım ve soru ID'leri | A1/numaralı/V-01; S1/numaralı/S-01 | V-01/VS-01; AS-01/S-01. Hâlâ tutarsız (O-3 uygulanmadı). |
| Kabul kriteri biçimi | KK / AC / Verilen-O zaman | 4/4 "Verilen/Eğer/O zaman" ya da benzeri; etiket adı değişiyor (KK, AC) |
| Kaynak gösterimi | [K1] / satır içi / [Kısaltma] | [K-x] (2) / satır içi (2). Hâlâ tutarsız (O-4 uygulanmadı). |

| Kriter | v1 | v2 | Not |
|---|---|---|---|
| T-1 Yapı | 4 / 4 | **5 / 5** | Bölüm 3 alt başlıkları ve öncelik sütunu artık sabit. ID biçimleri hâlâ farklı ama bölüm düzeyinde fark yok. |
| T-2 Kapsam | 4 / 3 | **5 / 5** | Kritik başlıkların hiçbiri eksik değil. FR sayısındaki fark C önceliğindeki eklerde. |

### 3.2 TA, duplicate finder: kritik kararlar

| Karar | TA v1 run1 | TA v1 run2 | TA v2 (v1 girdi) | TA v2 (v2 girdi) |
|---|---|---|---|---|
| Algısal özet | dHash + pHash (ikisi de) | dHash + pHash | dHash + pHash | dHash + pHash |
| Eşik Sıkı/Normal/Gevşek | 4 / 8 / 12 (ikisi için) | 5 / 8 / 10 | 4 / 8 / 12 (geçici) | dHash 3/6/10, pHash 2/4/8 (kaynaktan türetilmiş) |
| Gruplama | Union-find | Yıldız (lider) | **Tam bağlantı** | **Lider kümeleme** |
| Union-find reddedildi mi? | — | Evet (zincirleme) | Evet, ölçüt tablosuyla | Evet, ölçüt tablosuyla |
| Birebir ön filtre | Dosya boyutu → SHA-256 → bayt | Piksel boyutu + pHash ≤ 4 → SHA-256 | Piksel boyutu + dHash ≤ 4 → uzunluk → SHA-256 | Dosya boyutu → SHA-256 |
| Aday arama | BK-ağacı (dHash) | BK-ağacı (pHash) | BK-ağacı (dHash) | BK-ağacı yok; lider kümeleme doğrudan, en kötü durumda O(n²) (R9: "gerekirse BK-tree"). Zaman penceresi yalnızca seri çekim kategorisi (FR-15) için. |
| Embedding | Yok | Yok | Yok | Yok |
| Video izni (Android 13) | **Kaçırılmış** | Var | Var | Var |
| Kararlar seçim ölçütüyle mi? | Hayır | Kısmen ("seçilmeyenler" tablosu) | **Evet (7 karar)** | **Evet (6 karar)** |

| Kriter | v1 (2 çalıştırma) | v2 (2 çalıştırma, farklı girdi) |
|---|---|---|
| T-3 Temel kararlar | 4 | **4**: çatı, paketler, özet türleri, silme yolu ve embedding kararı 4/4 aynı. Gruplama ve eşikler değişken. |

**Sonuç:** Duplicate'teki tutarlılık sorunu **çözülmedi**. Sorun daraldı: v2'de değişkenlik "ölçütü sağlayan eşdeğer seçenekler arasındaki seçim" ile sınırlı. Eşiklerdeki değişkenlik ise BA'daki sayısal hedeflerin ve kaynakların farklı olmasından da etkileniyor. v2 girdili TA, eşikleri BA v2'nin kaynağından (HackerFactor) türetmiş.

### 3.3 TA, çarpım tablosu: platform kararı

| | TA v1 run1 | TA v1 run2 | TA v2 (v1 girdi) | TA v2 (v2 girdi) |
|---|---|---|---|---|
| Platform (geçici) | Web (PWA) | Web | Web | **Flutter mobil** |
| Depolama | `localStorage` | `localStorage` | `localStorage` | SQLite |

v2 girdili TA'nın platformu değiştirmesi persona değişikliğinden değil, **girdi değişikliğinden** geliyor. BA v2, BA-1 kuralı gereği App Store ve Google Play çocuk politikalarını, ebeveyn kapısını ve KVKK'yı işledi. TA bunu "mağaza uygulaması" sinyali olarak okudu. Platform BA'da hâlâ açık soru (AS-02), yani bu bir çelişki değil. Ama BA'daki tek bir araştırma başlığının TA'nın en büyük kararını çevirebildiğini gösteriyor (bkz. YS-3).

---

## 4. Kör değerlendirme

### 4.1 Yöntem

- **Değerlendirici:** Ayrı bir `general-purpose` agent. Depo **dışında** açılan bir `claude -p` oturumundan çağrıldı, bu yüzden `CLAUDE.md` ve proje geçmişi yüklenmedi. Görev metni agent'a birebir gitti (oturum kaydından doğrulandı).
- **Okuduğu dosyalar:** Yalnızca kör klasördeki 11 dosya. 10 kaynak linkini WebFetch ile açtı.
- **Dosyalar ve maskeleme:**
  - Senaryo başına bir v1 ve bir v2 BA, bir v1 ve bir v2 TA, A/B adlarıyla verildi.
  - Kopyalarda "Persona sürümü" satırı, dosya adları ve yol izleri maskelendi; asıllara dokunulmadı.
  - A/B sırası senaryodan senaryoya karıştırıldı.
- **TA kıyasının adilliği:** v2 TA olarak **v1 girdili** çıktı verildi; böylece iki TA aynı girdiyi kullanıyor.
- **Eşleme anahtarı** (değerlendirme bittikten sonra açıldı):

| Kör ad | Gerçek dosya |
|---|---|
| carpim-BA-A / -B | ba-carpim-tablosu-**v2**-run1 / ba-carpim-tablosu-**v1**-run2 |
| duplicate-BA-A / -B | ba-duplicate-finder-**v1**-run3 / ba-duplicate-finder-**v2**-run1 |
| carpim-TA-A / -B | ta-carpim-tablosu-**v1**-run1 / ta-carpim-tablosu-**v2**-v1girdi-run1 |
| duplicate-TA-A / -B | ta-duplicate-finder-**v2**-v1girdi-run1 / ta-duplicate-finder-**v1**-run1 |

### 4.2 Kör puanlar (v1/v2 olarak çözülmüş)

| Karşılaştırma | v1 | v2 | Kör değerlendiricinin kazanan ve gerekçesi |
|---|---|---|---|
| Çarpım BA | 26 | **29** | v2. Müfredatı birincil MEB PDF'inden doğruluyor; mağaza politikaları ve KVKK gereksinime bağlanmış; MoSCoW ve MVP sınırı var. |
| Duplicate BA | 28 | **29** | v2 (biraz). Daha çok birincil kaynak ve ölçüm verisi; MoSCoW var. |
| Çarpım TA | 23 | **24** | v2 (biraz). Her kritik karar ölçüt ve seçenek tablosuyla; Safari riski kaynakla kapatılmış. |
| Duplicate TA | 24 | 24 | Eşit. v2 gruplamada ve kararları belgelemede daha iyi; v1 birebir doğrulamada (FR-04) daha iyi; v1'de doküman dışı not var. |

**Kör değerlendirmenin toplamı:** 4 karşılaştırmada v2 → 3 kazanç, 1 eşitlik, 0 kayıp.

**Kör değerlendiricinin doküman dışı not tespiti:** Notu yalnızca **v1** TA'da (duplicate v1 run1) buldu. v2 dosyalarının hiçbirinde bulmadı. Bu, O-1'in bağımsız bir teyidi.

### 4.3 Kendi değerlendirmemle çeliştiği yerler

| # | Konu | Kör değerlendirme | Benim değerlendirmem | Çözüm / yorum |
|---|---|---|---|---|
| Ç-1 | Duplicate TA v2 (v1 girdi), TA-1 | **4.** Birebir ön filtre algısal mesafeye dayandığı için FR-04'teki "her zaman aynı gruba düşer" garanti edilmiyor. | 5 | **Kör değerlendirme haklı bir zayıflığı gösteriyor.** Aşama 2'de aynı yaklaşımı kullanan v1 run2'ye de 5 vermiştim; tutarlılık için 5'i koruyorum. Ancak bu bir **ortak zayıflık**: TA-1 kuralı (seçim ölçütleri) bu durumu önlemedi. TA bu seçeneğe Ö2 için "Var" yazıp gerekçeyi risk R-04'e bıraktı. Puan konusunda **belirsiz**. |
| Ç-2 | Duplicate TA v1 run1, `READ_MEDIA_VIDEO` | İki TA çelişiyor; hangisinin doğru olduğu **belirsiz** (kaynak açılmadı). | v2 doğru, v1 yanlış | **Çözüldü:** pub.dev photo_manager belgesiyle doğrulandı, v2 doğru. Kör değerlendirme bu noktayı v1 aleyhine puana yansıtmadı. Doğrulanınca v1'in TA-3'ü 4'te kalır, fark v2 lehine büyür. |
| Ç-3 | Çarpım TA, TA-4 (ölçek) | v1 ve v2: 5 | v1 run1 ve v2 v1girdi: 4 | İkisinde de Playwright ile 3 tarayıcı motorunda uçtan uca test hattı var. Ben bunu küçük ürün için ağır buldum, kör değerlendirici bulmadı. Fark iki sürüme eşit uygulandığı için v1–v2 karşılaştırmasını etkilemiyor. |
| Ç-4 | Çarpım BA v1 run2, toplam | 26 | 28 (Aşama 2) | Kör değerlendirici "mağaza politikası ve KVKK araştırılmadı", "öncelik yok" ve "3. sınıf doğrulanamadı" gerekçesiyle BA-1, BA-3, BA-4 ve BA-6'dan puan kırdı. Aşama 2'de ben bu eksikleri "persona eksikliği" sayıp puandan daha az düşmüştüm. v2 ile yan yana okununca eksikler daha görünür hale geliyor. Kör puan daha katı ama savunulabilir. |
| Ç-5 | Duplicate BA v1 run3, BA-1 | 4: algoritma bölümü tek bir firma bloguna (MojoAuth) dayanıyor | 5 | Kör değerlendirme haklı: run3'ün algoritma kaynakları büyük ölçüde MojoAuth. v2 run1 ve run2 ise HackerFactor ölçümleri ve arXiv tam metinlerini kullanıyor. Benim v1 puanım cömertti. |
| Ç-6 | Çarpım TA v2, Safari muafiyeti | Alıntıyı açmadı | — | **Doğrulandı:** WebKit blogunda birebir geçiyor (2.1). |

**Genel uyum:** Yön açısından kör değerlendirme ile benim değerlendirmem uyumlu: v2 BA'da açıkça, TA'da hafifçe daha iyi. Kör puanlar v1 BA'ya benimkinden daha düşük; bu da v1→v2 farkını benimkinden büyük gösteriyor.

---

## 5. Yeni ve süren sorunlar

### 5.1 v2'de ortaya çıkan yeni sorunlar

| # | Sorun | Nerede | Olası neden |
|---|---|---|---|
| YS-1 | **Atıf numaralandırma hatası.** Doküman sonunda "Kaynak numaralandırma düzeltmesi: [S6] … [S13] olarak okunmalıdır" notu var. Notun kendisi doküman içinde, ama gövdedeki atıflar yanlış numaraya gidiyor. | Çarpım TA v2 (v2 girdi) | Uzun doküman ve çok kaynak. Kaynak ID biçimi kuralı (O-4) yok; agent atıfları sonradan yeniden numaralandırmaya çalışmış. |
| YS-2 | **Doküman uzunluğu ve maliyet arttı.** BA v2 4.300–6.100 kelime (v1 2.800–4.400). BA v2'de WebFetch 20–35 çağrı (v1 12–23). TA uzunlukları benzer. | Tüm BA v2 | Zorunlu araştırma başlıkları ve bağlam/alıntı kuralı. Kalite arttı ama okuma yükü de arttı. "Basit" ürün için doküman boyutunun da ölçeğe uygun olması kuralı yok. |
| YS-3 | **TA'nın en büyük kararı BA içeriğine aşırı duyarlı.** Çarpımda platform kararı web → Flutter mobile döndü, çünkü BA v2 mağaza politikalarını işledi (3.3). | Çarpım TA v2 (v2 girdi) | BA'nın platform sorusu (AS-02) açık; BA "olası platformlar"ı iş etkisi düzeyinde incelemek zorunda (BA-1). TA bunu sinyal olarak okuyor. Aşama 2'deki BA-5 ve TA-4 önerileri (mimariyi belirleyen açık soru için varsayılan önerme / iki seçenekli karşılaştırma) uygulanmadığı için bu açık kaldı. |
| YS-4 | **Kaynak ID'leri BA'nın ID'leriyle çakışıyor.** Çarpım TA v2 (v2 girdi) kaynakları `S1…S13` diye numaralandırmış; BA'da `S-xx` geri soru öneki olarak kullanılabiliyor. Duplicate TA v2'de `T1…T19` kullanılmış; bu, teknoloji kararı önekiyle karışabilir. | TA v2 | O-3/O-4 (ID önekleri, kaynak gösterimi) uygulanmadı. |

| YS-5 | **Büyük kütüphane performans tasarımı zayıfladı.** Duplicate TA v2 (v2 girdi), lider kümelemeyi BK-ağacı aday araması olmadan doğrudan yapıyor. En kötü durum O(n²) olduğu için 50.000 fotoğraf hedefi (BA NFR) riskli. TA bunu R9 olarak kabul ediyor ve BK-ağacını yalnızca yedek plan olarak bırakıyor. Diğer 3 duplicate TA'da aday arama BK-ağacıyla yapılıyordu. | Duplicate TA v2 (v2 girdi) | Kritik karar tablosunda (TA-1) gruplama ölçütleri kesinlik ve belirlenimcilik üzerine kurulmuş; performans ölçütü (NFR'ye bağlı) bu kararda yok. TA-1 kuralı, bir kararın tüm ilgili NFR'leri ölçüt olarak almasını zorlamıyor. |

### 5.2 Süren (v2 değişikliklerinin kapsamı dışında kalan) sorunlar

- **ID ve atıf biçimi tutarsızlığı:** O-3 ve O-4 orta öncelikliydi, uygulanmadı.
- **TA'nın BA'ya sormadan küçük davranış eklemesi:** Fiziksel klavye girişi çarpım TA v2'de de var (TA-6, düşük öncelik, uygulanmadı).
- **Test stratejisinin ürün ölçeğine göre ayarlanmaması:** Çarpımda Playwright ile 3 motorlu test hattı (TA-5, düşük öncelik).
- **Kritik algoritma değerlerinin çalıştırmadan çalıştırmaya değişmesi** (3.2). TA-1 bunu yalnızca kısmen çözdü.

---

## 6. Sonuç

- **Net etki:** 7 değişikliğin 5'i hedeflediği sorunu çözdü (O-1, BA-1, BA-2, TA-2, üzerine yazma kuralı). O-5 büyük ölçüde çözdü: olgusal aktarım hatası 0, iki küçük nüans kaldı. TA-1 kısmen çözdü: karar gerekçeleri yapılandırıldı, sonuç değerleri hâlâ değişiyor.
- **Puan:**
  - BA ortalaması 27,0 → 29,0 (çarpım) ve 28,3 → 29,0 (duplicate).
  - TA ortalaması 23,5 → 23,5 (çarpım) ve 24,5 → 24,5 (duplicate). TA puanında değişim yok; kazanım gerekçe yapısında ve kaynak doğruluğunda.
  - Kör değerlendirme: 4 karşılaştırmada v2'ye 3 kazanç, 1 eşitlik.
- **Bedel:** Daha uzun dokümanlar ve daha fazla araç çağrısı. Bir atıf numaralandırma hatası. Platform kararının BA içeriğine duyarlılığı. Bir çalıştırmada büyük kütüphane performans tasarımının zayıflaması (YS-5).
