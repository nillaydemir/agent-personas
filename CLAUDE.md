# Agent Personas — Proje Hafızası

## Projenin amacı
Claude Code subagent'ları ile iki "agent persona" tasarlamak, senaryolarla test
etmek ve sonuçlara göre iyileştirmek:

- **Business Analyst (BA)**
- **Technical Architect (TA)**

Bu projede **uygulama kodu yazılmaz**; agent'lar yalnızca doküman üretir. Agent'lar
yalnızca geliştirme sırasında kullanılan yardımcılardır; çalışan bir uygulamada
karşılıkları yoktur.

## Agent'lar ve akış
```
Görev metni ──► business-analyst ──► Analiz dokümanı (FR / NFR / US)
                                          │
                                          ▼
                               technical-architect ──► Teknik gereksinim dokümanı
```
1. **BA** görev metnini alır, web'de araştırma yapar ve kaynaklı bir analiz dokümanı üretir.
2. **TA**, BA dokümanını (dosya yolu verilerek) girdi olarak alır ve her teknik kararı
   bir BA gereksinimine bağlayan teknik gereksinim dokümanı üretir.

## Klasör yapısı
| Yol | Amaç |
|---|---|
| `CLAUDE.md` | Her oturumda okunan proje hafızası (bu dosya). |
| `.claude/agents/` | Persona tanımları: yalnızca role özgü görev, araç ve sınırlar. |
| `.claude/rules/` | İki agent'ın da uyduğu ortak kurallar (doküman standardı, araştırma, kapsam). |
| `eval/` | Değerlendirme kontrol listesi ve persona değişiklik günlüğü. |
| `outputs/` | Test çıktıları. Henüz yok, testler sırasında `outputs/<senaryo-adi>/` olarak oluşur. |

## Ortak kurallar
Ortak kurallar agent dosyalarında tekrar edilmez, şu dosyalardadır:
- @.claude/rules/documentation-standards.md
- @.claude/rules/research-rules.md
- @.claude/rules/scope-rules.md

## Çıktıların kaydı
- Agent'ların **Write aracı yoktur**; dokümanı tam metin olarak döndürürler.
- Ana oturum, dönen metni `outputs/<senaryo-adi>/` altına,
  `documentation-standards.md` içindeki adlandırma kuralıyla kaydeder
  (örn. `outputs/carpim-tablosu/ba-carpim-tablosu-v1-run1.md`).
- **`outputs/` altındaki dosyaların üzerine asla yazılmaz.** Kaydetmeden önce hedef
  dosyanın var olmadığı kontrol edilir. Dosya varsa işlem durur ve kullanıcıya sorulur.
  (Aşama 2'de bir betik hatası 4 BA dosyasının üzerine yazmıştı.)
- TA çalıştırılırken ilgili BA dokümanının **dosya yolu** verilir.
- Her değerlendirme `eval/kontrol-listesi.md` ile puanlanır; persona
  değişiklikleri `eval/degisiklik-gunlugu.md` dosyasına işlenir.

## Geliştirme aşamaları
1. **Yapı ve persona'lar**: klasör yapısı, kurallar, agent tanımları, eval şablonları, duman testi.
2. **İlk testler ve değerlendirme**: senaryoların çalıştırılması, çıktıların kaydı ve puanlanması.
3. **İyileştirme ve tekrar test**: gözlenen sorunlara göre persona'ların güncellenmesi (sürüm artırılır) ve tekrar test.

Her aşamadan sonra bu dosya güncellenir.

## Durum
- Persona sürümleri: BA v1, TA v1

### Aşama 1 tamamlandı (2026-11-22)
- Yapı, ortak kurallar, BA/TA persona'ları (v1) ve eval şablonları oluşturuldu.
- **Duman testi** (yeni bir Claude Code oturumunda; çıktılar kaydedilmedi):
  - İki agent da tanındı ve Agent aracıyla çağrılabildi.
  - BA: WebSearch, WebFetch ve Read kullandı; dosya yazmadı. 3 ID'li FR üretti.
    Açamadığı sayfaları kaynak göstermedi, araştırma kuralına uydu.
  - TA: BA dokümanını Read ile okudu; dosya yazmadı. 2 kararı FR ID'lerine bağladı
    ve "BA'ya geri sorular" yazdı. Yeni gereksinim eklemedi.
- **Çözülen kısıt:** Duman testi sırasında bulut ortamının ağ politikası "limited"
  olduğu için WebFetch dış sitelere (wikipedia, todoist, microsoft vb.) erişemedi.
  Ağ erişimi 2026-11-22'de "full" yapıldı; bu siteler artık erişilebilir.
  BA'nın kaynaklı araştırması Aşama 2'de doğrulanacak.
- **Gözlem (Aşama 2'de izlenecek):** İki agent da "Başlamadan önce" adımındaki
  rules dosyalarını ayrıca Read ile açmadı; kurallar CLAUDE.md importları üzerinden
  uygulanmış görünüyor. Agent'lar ayrıca doküman dışına ana oturum için not ekledi.
  Kayıtta yalnızca doküman gövdesinin alınması gerekir.
- Sonraki adım: **Aşama 2: İlk testler ve değerlendirme.**

### Aşama 2 tamamlandı (2026-11-22)
- **TA testleri:** TA v1, seçilen BA dokümanlarıyla (çarpım **run2**, duplicate **run3**)
  2'şer kez çalıştırıldı. Değerlendirme: `eval/degerlendirme-v1-ta.md`.
- **TA puanları (/25):** çarpım 23 / 24; duplicate 24 / 25.
- **İzlenebilirlik:** 4/4 çalıştırmada eksiksiz (27/27); uydurma FR/NFR yok.
  Belirsizlikler "geçici" işaretiyle BA'ya soruluyor.
- **Mimari:** Ürün büyüklüğüne uygun.
  - Çarpım: sunucusuz web uygulaması, TypeScript, `localStorage`.
  - Duplicate: tek Flutter uygulaması, `photo_manager`, SQLite, dHash + pHash.
  - Flutter kısıtına tam uyuldu.
- **Başlıca sorunlar:**
  - Doküman dışı not hem BA'da (4/6) hem TA'da (1/4).
  - ID ve kaynak bölümü biçimi tutarsız.
  - BA'da kritik araştırma alanları bazen eksik.
  - BA'da kaynak aktarımında nüans kaybı.
  - TA'da duplicate algoritma kararları çalıştırmalar arasında değişiyor.
  - TA'da kütüphane izin kısıtı bir kez gözden kaçtı.
- **Aşama 3 önerileri:** `eval/aşama-3-yapilacaklar.md` (henüz uygulanmadı;
  `degisiklik-gunlugu.md`'ye değişiklik yazılmadı).
- **Süreç notu:** TA çıktıları kaydedilirken ana oturumun betik hatası 4 BA dosyasının
  üzerine yazdı. Dosyalar git'ten geri yüklendi ve HEAD ile aynı oldukları doğrulandı.
  TA girdileri etkilenmedi.
- Sonraki adım: **Aşama 3: İyileştirme ve tekrar test.**

#### Aşama 2 ara notu (BA testleri)
- BA v1, iki senaryo için 3'er kez çalıştırıldı (`outputs/carpim-tablosu/`,
  `outputs/duplicate-finder/`). Değerlendirme: `eval/degerlendirme-v1-ba.md`.
- **Test yöntemi:** Her çalıştırma depoda açılan yeni bir `claude -p` oturumunda
  tek bir subagent çağrısıyla yapıldı. Görev metninin birebir gittiği ve çıktının
  değiştirilmeden kaydedildiği oturum kaydından doğrulandı.
- **Puanlar (/30):** çarpım 27 / 28 / 26; duplicate 28 / 28 / 29.
- **Önemli bulgular:**
  - 24 kaynak linkinin 24'ü açıldı; uydurma yok. 4 iddia ise kaynakta yalnızca
    kısmen destekleniyor.
  - 6 çalıştırmanın 4'ünde doküman dışına ana oturum notu eklendi.
  - Varsayım/soru ID'leri ve kabul kriteri biçimi çalıştırmadan çalıştırmaya değişiyor.
  - Duplicate run2'de Google Play politikası riski eksik.
- **Ortam notu:** Bu oturumdaki `web-fetch` yardımcı agent'ının WebFetch aracı
  DNS hatası veriyor. Web erişimi gereken kontroller yeni `claude -p` oturumlarıyla
  yapılmalı.
- TA'ya giden girdiler (kullanıcı onayladı): çarpım **run2**, duplicate **run3**.
- Persona ve rules dosyaları bu aşamada değiştirilmedi. Önerilen değişiklikler
  Aşama 3'e bırakıldı.
