# Değerlendirme: Business Analyst v1 (Aşama 2)

- **Tarih:** 2026-11-22
- **Değerlendirilen persona:** BA v1 (persona ve rules dosyaları bu aşamada değiştirilmedi)
- **Ölçüt:** `eval/kontrol-listesi.md` (BA-1 … BA-6, T-1 … T-2; 1-5 puan)
- **Değerlendirilen dosyalar:**
  - `outputs/carpim-tablosu/ba-carpim-tablosu-v1-run{1,2,3}.md`
  - `outputs/duplicate-finder/ba-duplicate-finder-v1-run{1,2,3}.md`

## 0. Test yöntemi

- Her çalıştırma, depoda açılan **ayrı ve yeni** bir `claude -p` oturumunda tek bir `business-analyst` çağrısıyla yapıldı. Önceki çalıştırmaların çıktısı hiçbir çağrıya verilmedi. 6 çalıştırma paralel koştu.
- **Görev metninin birebir verildiği** oturum kayıtlarından doğrulandı: subagent'a giden `prompt` alanı 6 çalıştırmanın 6'sında görev metniyle karakter karakter aynı.
- **Çıktılar değiştirilmeden kaydedildi:** subagent'ın döndürdüğü metin oturum kaydından program ile alınıp dosyaya yazıldı. Bu metin, subagent'ın son mesajıyla 6/6 çalıştırmada birebir aynı. Doküman dışına eklenen notlar da talimat gereği dosyada bırakıldı (bkz. Sorun A-1).
- **Dosya yazma:** hiçbir çalıştırmada agent dosya yazmadı. Agent'lar Write aracına sahip değil; `git status` ile de doğrulandı.

| Çalıştırma | Read | WebSearch | WebFetch | Kural dosyalarını okudu mu? |
|---|---|---|---|---|
| çarpım run1 | 3 | 6 | 12 | Evet (3/3) |
| çarpım run2 | 4 | 5 | 21 (1 proxy hatası) | Evet (3/3) |
| çarpım run3 | 3 | 6 | 12 | Evet (3/3) |
| duplicate run1 | 4 | 7 | 23 | Evet (3/3) |
| duplicate run2 | 3 | 9 | 20 | Evet (3/3) |
| duplicate run3 | 5 | 8 | 16 | Evet (3/3) |

Not: 3'ün üzerindeki Read çağrıları, WebFetch'in büyük sayfaları kaydettiği geçici dosyaların okunmasıdır. Aşama 1'deki "rules dosyaları Read ile açılmıyor" gözlemi bu aşamada tekrarlanmadı.

**Kaynak kontrolü yöntemi:** Her dokümandan 4 link (toplam 24) seçildi. Seçimde sayısal veya özgül iddia taşıyan linkler tercih edildi. Linkler WebFetch ile ayrı `claude -p` oturumlarında açıldı. Her iddia için sayfadan alıntı istendi. Bu oturumdaki `web-fetch` yardımcı agent'larının WebFetch aracı DNS hatası verdiği için (24 linkten 1'i açıldı) kontrol bu yolla tekrarlandı. Ayrıca bir iddia (imagededup hız oranı) `curl` ile sayfa tablosundan elle hesaplanarak doğrulandı.

Kaynak kontrolündeki "Doğru / Kısmen / Yanlış" sonuçları BA dokümanındaki **Türkçe ifadeye** göre verildi. Bazı durumlarda kontrol için yazılan İngilizce özet ifade BA'nın ifadesinden farklıydı. Bu durumlar ayrıca belirtildi.

---

## 1. Senaryo 1: Çarpım tablosu

### 1.1 Kriter puanları

| Kriter | run1 | run2 | run3 |
|---|---|---|---|
| BA-1 Araştırma | 4 | 5 | 5 |
| BA-2 Kaynaklar gerçek ve destekleyici | 5 | 4 | 4 |
| BA-3 Ölçülebilirlik | 4 | 5 | 4 |
| BA-4 Kapsam | 5 | 4 | 4 |
| BA-5 Şablona uyum | 4 | 5 | 4 |
| BA-6 Açık sorular | 5 | 5 | 5 |
| **Toplam (/30)** | **27** | **28** | **26** |

### 1.2 Gerekçeler

**run1 (27/30)**
- **BA-1 = 4:** 10 kaynak var: 2 araştırma, 3 rakip, MEB, MTC ve WCAG. Ancak müfredat dayanağı olarak yalnızca İYEP sayfası kullanıldı. İYEP bir yetiştirme (telafi) programı, olağan 3. sınıf programı değil. 3. sınıfta tablonun 10'a kadar olduğu bulunamadı ve aralık 1–10 "doğrulanmamış varsayım" olarak kaldı (A2). Rakip sayısı da az (3).
- **BA-2 = 5:** Kontrol edilen 4 linkin 4'ü açıldı ve iddiaları destekliyor:
  - PSU: 15.402 öğrenci; 4–7'li işlemler daha fazla tekrar gerektiriyor; düşük beceri düzeyi daha çok tekrar istiyor.
  - TTRS heatmap: son 10 cevap; <1 sn ve >10 sn eşikleri.
  - WCAG: 24×24 AA; 44×44 eşiği SC 2.5.5'e ait.
  - İYEP: 3. sınıf, tekrarlı toplama, 1–5 ile çarpma, 0/1 etkisi, yer değiştirme.
  - Küçük bir nüans: TTRS ortalaması yalnızca doğru cevaplardan hesaplanıyor. Doküman bunu belirtmiyor.
- **BA-3 = 4:** NFR'lerin çoğu sayısal (≤3 dokunuş, ≤200 ms, 100 cevapta 0 çökme, 4.5:1). Ancak FR-11'deki "daha sık sorulmalı" için oran verilmemiş. US-06/KK1'deki "50 soruluk örneklemde ölçülebilir olmalı" ifadesi bir eşik tanımlamıyor.
- **BA-4 = 5:** "Basit" ifadesi açıkça gerekçe gösterilerek kapsam dar tutulmuş. Kapsam dışı listesi gerekçeli (9 madde). Teknoloji kararı yok. Platform ve çevrimdışı çalışma soruları iş açısından artı/eksiyle açık soruya bırakılmış.
- **BA-5 = 4:** 10 bölüm doğru sırada, başlık bloğu tam. Ancak dokümanın sonuna "Ana oturum için not" eklenmiş. Not, Aşama 1 gözlemine de atıf yapıyor (bkz. Sorun B-2).
- **BA-6 = 5:** 8 soru var: tablo aralığı, platform, çevrimdışı, süre, çoklu profil, KVKK, maskot, "öğrenildi" tanımı. Hepsi kararları doğrudan etkiliyor ve bazıları ilgili varsayıma bağlanmış.

**run2 (28/30)**
- **BA-1 = 5:** 21 WebFetch çağrısı ve 11 kaynak var. 5 rakip uygulama fiyat, puan ve şikâyetleriyle incelenmiş. Hatırlama pratiği, süre kaygısı ve MEB 2. sınıf programı araştırılmış. Kaynaklar arasındaki gerilim de açıkça uzlaştırılmış (KQED ↔ YouCubed).
- **BA-2 = 4:**
  - YouCubed: doğru (üçte bir, 5 yaş, 70−14 örneği).
  - MEB 2. sınıf: doğru (MAT.2.2.4/2.2.5, 5'e kadar).
  - App Store (3MB): büyük ölçüde doğru (2–20, 4,8 puan / 5,9 bin değerlendirme, 6–8 yaş). Ancak "aralıklı tekrar yok; bir kullanıcı bunu talep etmiş" ifadesi **kısmen** doğru. Yorumda "spaced repetition" geçmiyor; kullanıcı yalnızca yavaş çözülen soruların daha sık tekrarını istiyor.
  - PMC9385657: **kısmen** doğru. Katılımcılar yetişkin, ama öğrenilen şey gerçek aritmetik olgu değil, uydurma sayı olguları ("3 & 8 = 29"). Ayrıca yazarlar "günlere yayma katkı yapmış olabilir" yorumunu Standard Practice grubu için değil, Reactivations grubu için yapıyor. Doküman bu ayrımı yansıtmıyor.
- **BA-3 = 5:** En ölçülebilir doküman: FR-09'da "en az %50", FR-08'de "en az 2 soru sonra", NFR-02'de "5 çocuktan en az 4'ü". Kabul kriterleri Verilen/Olduğunda/O zaman biçiminde ve test edilebilir.
- **BA-4 = 4:** Kapsam genel olarak dar. Ancak 17 FR üç çalıştırmanın en fazlası. FR-16'daki "yetişkin doğrulaması" ve FR-17 "basit ürün" için ek yük getiriyor.
- **BA-5 = 5:** 10 bölüm sırada, doküman dışı not yok. Küçük bir tutarsızlık: Girdi satırında görev metni tırnaksız yazılmış.
- **BA-6 = 5:** 12 soru var. Müfredat aralığı, cevap biçimi ve iş modeli soruları kararları doğrudan etkiliyor. 3. sınıf aralığının doğrulanamadığını dürüstçe belirtmiş.

**run3 (26/30)**
- **BA-1 = 5:** MEB 2. ve 3. sınıf programı bulunmuş; varsayılan 1–10 aralığı tek dayanaklı olan çalıştırma bu. Hatırlama pratiğinde ikincil haber yerine asıl yayın kaynak gösterilmiş. Süre konusunda karşı görüş aranmış.
- **BA-2 = 4:**
  - MEB 3. sınıf: doğru.
  - TTRS Families: doğru (8 mod, gizlenebilir sayaç, £7.20, 6+).
  - OU yayını: doğru. "Sesli ezber", kaynaktaki "chanting" ile örtüşüyor.
  - Education Week: **kısmen yanlış.** Doküman "süreli ve süresiz koşul arasında kaygı farkı bulunmadı" diyor. Oysa çalışma *açık* ile *gizli* süreli görevleri karşılaştırmış; süresiz bir koşul yok. Ayrıca "kendi önceki sonuçlarıyla karşılaştırma" önerisi çalışmanın değil muhabirin cümlesi.
- **BA-3 = 4:** NFR'ler sayısal ve her birinin doğrulama yöntemi var. Ancak FR-07'deki ağırlıklandırma oranı açık soruya (S-05) bırakılmış, bu yüzden FR-07 şu haliyle tam test edilemiyor.
- **BA-4 = 4:** "Kişisel en iyi" (FR-12), "sakin mod" ve 2 kademeli yazı boyutu "basit" ürün için ek özellik. Hepsi kaynağa bağlanmış ama kapsamı genişletiyor.
- **BA-5 = 4:** Şablon tam. Ancak sona "Ana oturum için notlar" eklenmiş ve yanlış dosya adı önerilmiş (`...-run1.md`, oysa bu run3).
- **BA-6 = 5:** 11 soru var ve her biri "neden önemli" sütunuyla verilmiş. Çevrimdışı sorusu iş açısından artı/eksiyle yazılmış.

### 1.3 Kaynak kontrolü (çarpım tablosu)

| Çalıştırma | Link | Açıldı mı? | İddia destekleniyor mu? |
|---|---|---|---|
| run1 | pure.psu.edu (tekrar sayısı çalışması) | Evet | Evet (3/3 iddia) |
| run1 | intercom.help TTRS Heatmaps | Evet | Evet. Nüans: ortalama yalnızca doğru cevaplardan. |
| run1 | w3.org WCAG 2.5.8 | Evet | Evet (24×24 AA, 44×44 = SC 2.5.5) |
| run1 | tymm.meb.gov.tr İYEP 1831 | Evet | Evet. Sayfa olağan program değil, İYEP 3. sınıf. Doküman bunu doğru etiketlemiş. |
| run2 | youcubed.org Fluency Without Fear | Evet | Evet (4/4) |
| run2 | apps.apple.com id1451984868 | Evet | Büyük ölçüde. "Aralıklı tekrar talebi" **kısmen**. |
| run2 | tymm.meb.gov.tr ünite 164 | Evet | Evet (2. sınıf, MAT.2.2.4/2.2.5, 5'e kadar) |
| run2 | pmc.ncbi.nlm.nih.gov PMC9385657 | Evet | **Kısmen.** Uydurma sayı olguları; günlere yayma yorumu başka gruba ait. |
| run3 | tymm.meb.gov.tr ünite 264 | Evet | Evet (3. sınıf, 10'a kadar, 9'a kadar rakamlarla) |
| run3 | edweek.org timed tasks | Evet | **Kısmen yanlış.** Süresiz koşul yok; öneri muhabire ait. |
| run3 | ttrockstars.com/families | Evet | Evet (5/5) |
| run3 | research.ou.nl (Ophuis-Cox vd.) | Evet | Evet. "Sesli ezber" = chanting. |

**Özet:** 12 link açıldı (12/12). 9 linkte iddia tam destekleniyor, 3 linkte kısmen. Uydurma veya açılmayan link bulunmadı.

### 1.4 Tutarlılık (çarpım tablosu)

| Boyut | run1 | run2 | run3 |
|---|---|---|---|
| Bölüm yapısı | 10 bölüm, şablon sırası | 10 bölüm, şablon sırası | 10 bölüm, şablon sırası |
| FR / NFR / US | 15 / 10 / 8 | 17 / 10 / 7 | 14 / 9 / 8 |
| Kaynak sayısı / kelime | 10 / ~2.800 | 11 / ~3.300 | 8 / ~3.700 |
| İncelenen rakipler | Simply TT, Hız Yay., TTRS | TTRS, 3MB, Learning Apps, TT&Friends, Mobilaxy | Simply TT, TTRS, Hit the Button |
| Müfredat dayanağı | İYEP 3. sınıf (1–5) | MEB 2. sınıf (1–5); 3. sınıf doğrulanamadı | MEB 2. sınıf (1–5) + 3. sınıf (1–10) |
| Varsayılan tablo aralığı | 1–10 (varsayım) | 1–10 (varsayım) | 1–10 (kaynaklı) |
| Hedef yaş | 2.–4. sınıf | 7–10 yaş | 7–9 yaş |
| Süre | İsteğe bağlı, varsayılan kapalı; soru başına 6 sn | İsteğe bağlı, kapalı; soru başına 10/7/5 sn | İsteğe bağlı, kapalı; tur başına 60/120 sn + sayaç gizleme |
| "Öğrenildi" tanımı | Son 3 deneme doğru (işlem) | Son 3 cevap doğru (işlem) | Son 20'de ≥%90 ve her olgu ≥2 doğru (tablo) |
| Zayıf olgu önceliği | Var, oran yok | Var, ≥%50 | Var, oran açık soru |
| Öğretme ekranı (tekrarlı toplama) | Var (+0/1, yer değiştirme) | Var | Var (+ yanlışta ipucu) |
| Ödül | Yıldız | Tablo yıldızı | Yıldız + rozet + kişisel en iyi |
| Ek ayarlar | Ses | Ses + yetişkin doğrulaması | Ses + sakin mod + yazı boyutu |
| Varsayım / soru ID'leri | A1 / S1 | Numaralı / numaralı | V-01 / S-01 |
| Kabul kriteri biçimi | KK1… | Verilen/Olduğunda/O zaman | KK1… |
| Kaynak gösterim biçimi | [K1] + tablo | Satır içi link + tablo | Satır içi link + tablo |
| Doküman dışı not | Var | Yok | Var |

| Kriter | Puan | Gerekçe |
|---|---|---|
| T-1 Yapı benzer mi? | 4 | Bölüm yapısı ve sırası 3/3 aynı. ID biçimleri (varsayım, soru), kabul kriteri biçimi, kaynak gösterimi ve doküman dışı not çalıştırmadan çalıştırmaya değişiyor. |
| T-2 Kapsam benzer mi? | 4 | Çekirdek aynı: tablo seçimi, öğretme ekranı, alıştırma, anında geri bildirim, yanlış tekrarı, ilerleme ızgarası, isteğe bağlı süre, kapsam dışı listesi. Farklar ayrıntıda: süre modeli, "öğrenildi" tanımı, ödüller ve müfredat dayanağı. Müfredat farkı araştırma derinliğinden geliyor. |

---

## 2. Senaryo 2: Photo Library duplicate finder

### 2.1 Kriter puanları

| Kriter | run1 | run2 | run3 |
|---|---|---|---|
| BA-1 Araştırma | 5 | 4 | 5 |
| BA-2 Kaynaklar gerçek ve destekleyici | 4 | 5 | 5 |
| BA-3 Ölçülebilirlik | 5 | 5 | 5 |
| BA-4 Kapsam | 5 | 5 | 4 |
| BA-5 Şablona uyum | 4 | 4 | 5 |
| BA-6 Açık sorular | 5 | 5 | 5 |
| **Toplam (/30)** | **28** | **28** | **29** |

### 2.2 Gerekçeler

**run1 (28/30)**
- **BA-1 = 5:** 17 kaynak, en geniş araştırma bu çalıştırmada:
  - Algoritmalar: HackerFactor, imagededup benchmark, arXiv, bulanıklık ölçümü.
  - Rakipler: 9 çözüm, yerleşik araçlar dahil.
  - Platform kısıtları: iOS kısıtlı erişim, Android 14 kısmi erişim, Google Play izin politikası, Android silme onayı, iOS Son Silinenler.
- **BA-2 = 4:**
  - HackerFactor: doğru (150 bin görsel, 32 yerine 400+, pHash'te hata yok, dHash ile ön eleme, eşikler). "pHash yaklaşık iki kat süre" ifadesi sayfada yok; aHash 3,75 saat, pHash 7+ saat ve "dHash aHash kadar hızlı" bilgilerinden **çıkarım**.
  - imagededup: "dHash eşik 10'da kesinlik 0,223" ve "pHash 0,980" değerleri yalnızca *tam kopya* veri setine ait, ama doküman veri setini belirtmeden yazıyor (**kısmen**). "Özetleme CNN'den 10–15 kat hızlı" ifadesi sayfada metin olarak yok; ancak tablodaki sürelerden hesaplanınca doğru (dHash 18–36 sn, CNN 191–397 sn).
  - CleverFiles: büyük ölçüde doğru. "20/5×4" puanlama gösterimi ters yazılmış (4 kriter × 5). "Karmaşık arayüz" yerine sayfada "cluttered" geçiyor.
  - Apple 102260: doğru.
- **BA-3 = 5:**
  - NFR'lerin tamamı ölçüm yöntemli: ağda 0 istek, 10 bin fotoğraf ≤5 dk (rakip kıyasına dayalı), P ≥%98 / R ≥%90, indeks ≤50 MB.
  - Kabul kriterleri test edilebilir.
  - Hedeflerin mobil cihazda doğrulanmadığı V-05'te açıkça yazılmış.
- **BA-4 = 5:** MoSCoW önceliği var. "Benzer fotoğraflar" S olarak ayrılmış. Rakiplerin ek özellikleri gerekçeyle kapsam dışına alınmış (CleverFiles'ın "bonus" değerlendirmesi). Mimari kararı yok.
- **BA-5 = 4:** Şablon tam. Ancak sona ana oturuma yönelik not eklenmiş ve farklı bir senaryo adı önerilmiş (`photo-duplicate-finder`).
- **BA-6 = 5:** 12 soru var: Play beyanı reddedilirse plan, buluttaki orijinaller, iOS silme davranışının doğrulanması, albüm üyeliği. Hepsi gerçek belirsizlikler.

**run2 (28/30)**
- **BA-1 = 4:** Algoritma araştırması en derini: eşik karşılaştırma tablosu, BK-tree, O(N²), MediaPipe mobil gecikmesi. Ancak **Google Play izin politikası riski ve Android'de silme/çöp davranışı hiç araştırılmamış**. Diğer iki çalıştırmada bu konular kritik iş riski olarak yer alıyor. Rakip sayısı da az (4).
- **BA-2 = 5:**
  - MediaPipe: doğru. Flutter platform listesinde var; Pixel 6'da 3,94 / 9,75 ms.
  - Ben Hoyt: doğru (128 bit, 2 bit eşik, 4–5'te yanlış pozitif, O(N²), BK-tree).
  - MacPaw SSS: doğru (6/6).
  - Setapp: doğru. Tek küçük kusur: iOS satırındaki "30 gün" Setapp'te birleştirme bağlamında geçmiyor.
- **BA-3 = 5:** NFR'ler ölçüm yöntemli. NFR-04'te ölçeklenebilirlik bile sayısal (2× fotoğrafta ≤2,5× süre). Hedeflerin analist önerisi olduğu V-04'te yazılı.
- **BA-4 = 5:** MoSCoW önceliği var. Tek albüm tarama "C" olarak işaretlenmiş. Kapsam dışı tablosu gerekçeli. Algoritma seçimi açıkça TA'ya bırakılmış.
- **BA-5 = 4:** Şablon tam. Ancak sonda ana oturuma yönelik notlar ve farklı senaryo adı önerisi var.
- **BA-6 = 5:** 10 soru var. S-08'de kaynaklar arasındaki çelişki (Setapp ↔ MacRumors) açık soru yapılmış.

**run3 (29/30)**
- **BA-1 = 5:** Algoritmalar, rakipler (yerleşik araçlar ve Gemini dahil) ve platform kısıtları (Play politikası ve tarihi, Android silme, iOS sınırlı erişim ve silme) araştırılmış. CleverFiles kaynağındaki çıkar çatışmasını fark eden tek çalıştırma bu.
- **BA-2 = 5:**
  - Google Play politikası: doğru (çekirdek işlev şartı, beyan, 28 Mayıs 2025, alternatif deneyim, galeri örneği).
  - MojoAuth: doğru (5/5).
  - MacPaw: doğru.
  - Android MediaStore: doğru. "Her ikisinde de onay diyaloğu" ifadesi genel olarak doğru, ancak istisnalar (OEM galeri, medya yönetimi izni) belirtilmemiş.
- **BA-3 = 5:** FR tablosunda her gereksinimin yanında kabul ölçütü var. NFR'ler ölçüm yöntemli. Mobil performans için kaynak bulunamadığı açıkça yazılmış (V-04, S-08).
- **BA-4 = 4:** Kapsam dışı listesi iyi. Ancak öncelik (MVP ayrımı) yok. Platform bölümünde API adlarına (`createTrashRequest`, `PHAssetChangeRequest`, sürüm/nesil) girilmiş; bu iş analizinden çok teknik ayrıntı (bkz. 2.5).
- **BA-5 = 5:** Şablon tam. Sondaki notlar Kaynaklar bölümü içinde ve açılamayan sayfaları açıklıyor; doküman içeriğiyle ilgili, ana oturuma yönelik değil.
- **BA-6 = 5:** 14 soru var. S-13 (iOS'ta yerleşik araç varken ürünün farkı ne?) ve S-14 (Play reddedilirse ne olacak?) iş açısından en değerli sorular.

### 2.3 Kaynak kontrolü (duplicate finder)

| Çalıştırma | Link | Açıldı mı? | İddia destekleniyor mu? |
|---|---|---|---|
| run1 | hackerfactor.com "Kind of Like That" | Evet | Evet. "≈2 kat süre" sayfadan çıkarım. |
| run1 | idealo.github.io imagededup benchmarks | Evet | **Kısmen.** 0,223 / 0,980 yalnızca tam kopya setinde; 10–15 kat tablodan hesaplanınca doğru. |
| run1 | cleverfiles.com en iyi uygulamalar | Evet | Büyük ölçüde. Puan gösterimi ters; "karmaşık" ≈ "cluttered". |
| run1 | support.apple.com/102260 | Evet | Evet ("birkaç güne kadar") |
| run2 | developers.google.com MediaPipe Image Embedder | Evet | Evet (Flutter listede, gecikmeler doğru) |
| run2 | benhoyt.com duplicate image detection | Evet | Evet (4/4) |
| run2 | setapp.com | Evet | Evet. "30 gün" birleştirme bağlamında değil (küçük). |
| run2 | macpaw.com Gemini SSS | Evet | Evet (6/6) |
| run3 | support.google.com Play politika 14115180 | Evet | Evet (5/5) |
| run3 | mojoauth.com phash-vs-dhash | Evet | Evet (5/5) |
| run3 | developer.android.com shared/media | Evet | Evet. Onay diyaloğu istisnaları anılmamış. |
| run3 | macpaw.com Gemini SSS | Evet | Evet |

**Özet:** 12 link açıldı (12/12). 11 linkte iddia tam veya büyük ölçüde destekleniyor, 1 linkte kısmen (bağlam eksik). Uydurma URL bulunmadı. Agent'lar açamadıkları sayfaları (MDPI 403, TDS 403, boş Apple sayfaları) kaynak göstermedi ve bunu belirtti.

### 2.4 Tutarlılık (duplicate finder)

| Boyut | run1 | run2 | run3 |
|---|---|---|---|
| Bölüm yapısı | 10 bölüm, şablon sırası | 10 bölüm, şablon sırası | 10 bölüm, şablon sırası |
| FR / NFR / US | 15 / 14 / 8 | 14 / 10 / 9 | 16 / 11 / 9 |
| Kaynak sayısı / kelime | 17 / ~4.100 | 16 / ~4.400 | 13 / ~4.000 |
| Öncelik | MoSCoW | MoSCoW | Yok |
| Sonuç kategorileri | 3: Birebir, Kopyalar, Benzer | 2: Birebir, Benzer | 2: Birebir, Benzer |
| Benzer fotoğraflarda otomatik işaretleme | Yok | Var (önerilen hariç) | Var (önerilen hariç) |
| Hassasiyet ayarı / "Kopya değil" / artımlı tarama | Var / Var / Var | Var / Var / Var | Var / Var / Var |
| Yarıda kalan taramaya devam | Var | Yok | Var |
| Tek albüm tarama | Yok | Var (C) | Yok |
| Tarama süresi hedefi (10 bin fotoğraf) | ≤5 dk | ≤10 dk | ≤10 dk |
| Benzer doğruluk hedefi | P ≥%98, R ≥%90 | P ≥%95, R ≥%85 | P ≥%95, R ≥%85 |
| Google Play izin riski | Var (iş riski) | **Yok** | Var (iş riski + varsayım + soru) |
| Android silme / çöp davranışı | Var | Yok | Var (API adlarıyla) |
| İncelenen algoritmalar | Kripto özet, aHash/dHash/pHash, embedding, katmanlı, bulanıklık | Kripto özet, aHash/dHash/pHash, embedding, BK-tree, zaman ön filtresi | Kripto özet, aHash/dHash/pHash, ikili özet, embedding |
| Rakipler | Apple, Files, Samsung, Google Fotoğraflar, 5 iOS uygulaması | Apple, Files, Gemini, Tidy | Apple, Files, Gemini, Remo, 5 iOS temizleyici |
| Varsayım / soru ID'leri | V-01 / numaralı | V-01 / S-01 | V-01 / S-01 |
| Kabul kriteri biçimi | AC1… | Madde işaretli | Verilen/O zaman |
| Doküman dışı not | Var | Var | Yok (notlar Kaynaklar içinde) |

| Kriter | Puan | Gerekçe |
|---|---|---|
| T-1 Yapı benzer mi? | 4 | Bölüm yapısı 3/3 aynı. Öncelik kullanımı, kabul kriteri biçimi ve ID biçimleri değişiyor. |
| T-2 Kapsam benzer mi? | 3 | Çekirdek ortak: izin durumları, birebir/benzer ayrımı, öneri ve favori koruma, geri alınabilir silme, hassasiyet, "kopya değil", artımlı tarama, video kapsam dışı. Ancak **Play politikası ve Android silme riski run2'de tamamen eksik**; bu, yayın riskini etkileyen önemli bir kapsam farkı. Performans ve doğruluk hedefleri de çalıştırmalar arasında değişiyor (5 ↔ 10 dk, %98 ↔ %95). |

### 2.5 Senaryo 2'ye özel analiz: Flutter kısıtı ve algoritmaların anlatımı

| Soru | run1 | run2 | run3 |
|---|---|---|---|
| Flutter'ı kısıt olarak kabul etti mi? | Evet. Özet: "paydaşın koyduğu bir kısıttır, BA'nın verdiği bir karar değildir". | Evet. NFR-07'de "paydaş tarafından verilmiş bir kısıt". | Evet. Özet notu ve V-03. |
| Kendisi teknoloji veya mimari kararı verdi mi? | Hayır. "Hangi yöntem, eşik ve teknoloji: TA karar verir" (3.2). | Karar yok. Ancak embedding satırında örnek olarak belirli kütüphane/model (MediaPipe, MobileNet V3) anılmış ve Flutter desteği kontrol edilmiş. **Belirsiz:** bu, karar değil fizibilite bilgisi; ama teknoloji değerlendirmesine yaklaşıyor. | Karar yok. Ancak platform bölümünde API adları (`createTrashRequest`, `createDeleteRequest`, `PHAssetChangeRequest`, sürüm/nesil) ve "artımlı tarama için gereklidir" ifadesi var. **Belirsiz / hafif ihlal eğilimi:** iş analizi için gereğinden teknik. |
| Algoritmalar iş açısından mı anlatıldı? | Evet. "Ne yakalar / ne kaçırır / iş artı / iş eksi" tablosu. | Evet. Birebir için iş artı/eksi tablosu; algısal özetler için hız ve doğruluk tablosu (işlev açıklaması daha teknik). | Evet. "Kullanıcıya faydası / maliyeti, riski" tablosu. |
| Doğruluk | Kesinlik ve duyarlılık değerleri, yanlış eşleşmenin güvene etkisi | Yanlış pozitif örnekleri, eşik belirsizliği → hassasiyet ayarı | Yanlış pozitifin şeffaf gösterilmesi → iki seviyeli sonuç ve onay |
| Hız | Özet ↔ embedding maliyeti, rakip tarama süresi | aHash/dHash/pHash süreleri, O(N²) ve BK-tree, mobil gecikme | Göreli hız; mobil ölçüm verisi bulunamadığı belirtilmiş |
| Gizlilik | NFR-01: cihazda işleme, ağda 0 istek | NFR-01 aynı; Gemini'nin cihazda işleme vurgusu | NFR-01 aynı; uçak modu testi |
| Cihaz üzerinde çalışma | Pil ve süre maliyeti; masaüstü benchmark'larının mobile genellenemeyeceği (V-05) | Model boyutu, Pixel 6 gecikmesi | Model boyutu; mobil performans için prototip sorusu (S-08) |

**Sonuç:** Üç çalıştırma da Flutter'ı doğru biçimde kısıt olarak ele aldı. Hiçbiri Flutter paketi, mimari veya veri saklama kararı vermedi. Algoritmalar her üçünde de doğruluk, hız, gizlilik ve cihaz üzerinde çalışma boyutlarıyla, iş etkisi üzerinden anlatıldı. Sınır ihlaline en yakın olanlar run2 (belirli kütüphane/model örnekleri) ve run3 (platform API adları). Kesin bir ihlal sayılmadı; "belirsiz" olarak işaretlendi.

---

## 3. Gözlenen sorunlar

### A. Agent'ın mevcut bir kuralı ihlal ettiği durumlar

| # | Sorun | Çalıştırma(lar) | İhlal edilen kural |
|---|---|---|---|
| A-1 | Kaynak, iddiayı tam desteklemiyor: Education Week çalışmasında süresiz koşul yok ve öneri muhabire ait; PMC'de olgular uydurma ve günlere yayma yorumu başka gruba ait. | çarpım run3, run2 | `research-rules`: "içeriği iddiayı desteklemeyen bir sayfa kaynak gösterilmez" |
| A-2 | Sayısal değer bağlamı olmadan aktarılmış: imagededup kesinlik değerleri yalnızca tam kopya setine ait ama veri seti belirtilmemiş. | duplicate run1 | `research-rules` (kısmi). **Belirsiz:** değer doğru, eksik olan bağlam. |
| A-3 | Teknik ayrıntıya kayma: platform API adları (run3) ve belirli kütüphane/model örnekleri (run2). | duplicate run3, run2 | `scope-rules`: "BA teknoloji kararı vermez". **Belirsiz:** karar verilmedi, ama iş analizinin düzeyini aşıyor. |

Not: A-1 ve A-2'de link gerçek ve açılmış. Sorun uydurma değil, aktarım hassasiyeti. Olası neden: WebFetch tam metni değil, küçük bir modelin özetini döndürüyor ve agent özet üzerinden yazıyor (bkz. B-6).

### B. Persona'daki eksiklikten kaynaklananlar

| # | Sorun | Çalıştırma(lar) | Persona'da eksik olan |
|---|---|---|---|
| B-1 | Doküman dışına ana oturuma yönelik not ekleniyor: kayıt yolu önerisi, araç kullanımı, açılamayan sayfalar. Bazılarında **yanlış dosya adı** (run3 için "run1") veya **farklı senaryo adı** (`photo-duplicate-finder`) önerilmiş. | çarpım run1, run3; duplicate run1, run2 (4/6) | Yanıtın yalnızca dokümandan oluşacağı; dosya adı ve kayıt yolu önerilmeyeceği. Açılamayan kaynak bilgisinin doküman içinde nereye yazılacağı. |
| B-2 | Agent, proje hafızasındaki (CLAUDE.md) test sürecine atıf yapıyor ("Aşama 1'de izlenmesi gereken konunun aksine…"). Değerlendirme bağlamı doküman çıktısına sızıyor. | çarpım run1, run3 | Proje süreci ve değerlendirme notlarının çıktıya yansıtılmaması gerektiği. CLAUDE.md'nin subagent'a da yüklendiği hesaba katılmamış. |
| B-3 | Biçim tutarsızlıkları: varsayım ID'leri (A1 / numaralı / V-01), soru ID'leri (S1 / numaralı / S-01), kabul kriteri biçimi (KK / AC / Verilen-O zaman), kaynak gösterimi ([K1] / satır içi / [Kısaltma]). | Tüm çalıştırmalar | `documentation-standards` yalnızca FR/NFR/US ID'lerini tanımlıyor. Varsayım, açık soru, kabul kriteri ve atıf biçimi tanımlı değil. |
| B-4 | Öncelik kullanımı tutarsız: MoSCoW 3 duplicate çalıştırmasından 2'sinde var, çarpım tablosunda hiçbirinde yok. MVP sınırı okuyucuya göre değişiyor. | duplicate run3; çarpım tümü | FR'lerde öncelik belirtme zorunluluğu ve yöntemi. |
| B-5 | Kritik alanların araştırılması garanti değil: run2'de Google Play politikası ve Android silme davranışı yok; çarpım run1'de olağan müfredat bulunamadı. Araştırma kapsamı çalıştırmaya göre değişiyor. | duplicate run2, çarpım run1 | Zorunlu araştırma başlıkları: alan kısıtları, mevzuat ve mağaza politikaları, platform kısıtları, kullanıcı segmenti standartları (ör. müfredat). En az rakip ve kaynak sayısı. |
| B-6 | Kaynak aktarım hassasiyeti: WebFetch özet döndürdüğü için incelikler kaçıyor (A-1, A-2). | çarpım run2, run3; duplicate run1 | Kritik sayısal veya bulgusal iddialarda kaynağın kendi ifadesine sadık kalma (kısa alıntı veya bağlam: veri seti, örneklem, karşılaştırma koşulu). İkincil haber yerine birincil kaynağı tercih etme. |
| B-7 | Kaynak tarafsızlığı tutarsız değerlendiriliyor: CleverFiles'ın kendi ürününü ilk sıraya koyduğunu yalnızca duplicate run3 belirtmiş. | duplicate run1, run2 | Satıcı veya pazarlama kaynaklarında çıkar çatışmasının belirtilmesi. |
| B-8 | Sayısal hedeflerin büyüklüğü çalıştırmadan çalıştırmaya değişiyor (5 ↔ 10 dk; %98/%90 ↔ %95/%85; soru başına süre ↔ tur başına süre). Hepsi varsayım olarak işaretlenmiş, bu doğru; ama TA'ya giden hedef rastgele. | Her iki senaryo | Kaynaksız sayısal hedeflerin nasıl türetileceği ve nasıl işaretleneceği (ör. "öneri hedef, paydaş onayı bekliyor" etiketi). Bu kısmen doğal değişkenlik, kısmen persona eksikliği. **Belirsiz.** |

### C. Olumlu gözlemler (persona'nın iyi çalışan yanları)

- Uydurma URL yok. 24 linkin 24'ü açıldı. Açılamayan sayfalar kaynak gösterilmedi ve açıkça not edildi.
- Kaynaksız sayısal hedefler 6/6 çalıştırmada "Varsayımlar" bölümünde açıkça işaretlendi.
- Kapsam kuralı güçlü uygulandı. Her çalıştırmada gerekçeli bir kapsam dışı listesi var; çevrimdışı çalışma ve platform gibi konular iş açısından artı/eksiyle TA'ya bırakıldı.
- Kaynaklar arası çelişkiler gizlenmedi, açık soru yapıldı (YouCubed ↔ KQED, Setapp ↔ MacRumors, Boaler ↔ EdWeek).

---

## 4. TA'ya verilmesi önerilen çalıştırmalar

| Senaryo | Öneri | Neden | Alternatif |
|---|---|---|---|
| Çarpım tablosu | **run2** (`outputs/carpim-tablosu/ba-carpim-tablosu-v1-run2.md`, 28/30) | En ölçülebilir gereksinimler (ör. FR-09 ≥%50), test edilebilir kabul kriterleri, en geniş rakip incelemesi. Doküman dışı not yok, yani TA'ya temiz girdi. Zayıf yanları: 3. sınıf müfredatı doğrulanmamış (açık soru olarak duruyor), PMC iddiası kısmen doğru. İkisi de TA kararlarını doğrudan etkilemiyor. | run3: müfredat dayanağı daha güçlü, ama Education Week aktarımı hatalı ve doküman dışı not içeriyor. |
| Duplicate finder | **run3** (`outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md`, 29/30) | En yüksek puan. Kaynak kontrolünde 4/4 destekleniyor. Play politikası, Android/iOS silme ve sınırlı erişim gibi TA'nın bilmesi gereken platform riskleri eksiksiz. Doküman dışı not yok. Zayıf yanları: öncelik (MoSCoW) yok; platform bölümünde API ayrıntısına kayma var. | run1 (28/30): MoSCoW önceliği ve platform riskleri var, ama sonda ana oturuma not var ve imagededup değerleri bağlamsız. run2, Play/Android silme riskini içermediği için **önerilmez**. |

---

## Genel not ve gözlemler

- **Güçlü yönler:**
  - BA v1 araştırma kuralına yüksek oranda uyuyor: uydurma link yok, açılamayan kaynak kullanılmıyor, varsayımlar ayrı.
  - 10 bölümlük şablon 6/6 çalıştırmada eksiksiz ve aynı sırada.
  - Flutter kısıtı doğru yorumlandı; algoritmalar iş diliyle anlatıldı.
- **Zayıf yönler / gözlenen sorunlar:**
  - Doküman dışı notlar (4/6).
  - Biçim ve öncelik tutarsızlıkları.
  - Kritik araştırma başlıklarının bazı çalıştırmalarda eksik kalması (duplicate run2'de Play politikası).
  - Kaynak aktarımında bağlam ve nüans kaybı.
  - Proje hafızasının çıktıya sızması.
- **Persona'da önerilen değişiklik (Aşama 3 için, henüz uygulanmadı):**
  - B-1, B-2: Yanıt yalnızca doküman olsun; süreç veya kayıt notu eklenmesin.
  - B-3, B-4: `documentation-standards`'a varsayım (`V-xx`), açık soru (`S-xx`), kabul kriteri (Verilen/Olduğunda/O zaman) ve öncelik (MoSCoW) biçimleri eklensin.
  - B-5: BA çalışma adımlarına zorunlu araştırma başlıkları eklensin (alan standartları ve müfredat, platform ve mağaza politikaları, mevzuat).
  - B-6, B-7: `research-rules`'a birincil kaynak tercihi, sayısal iddialarda bağlam (veri seti, örneklem, koşul) ve satıcı kaynaklarında çıkar çatışması notu eklensin.
  - Bu değişiklikler Aşama 3'te `eval/degisiklik-gunlugu.md` dosyasına işlenecek.
