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
