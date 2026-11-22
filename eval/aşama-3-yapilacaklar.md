# Aşama 3: Yapılacaklar (öneri listesi)

- **Tarih:** 2026-11-22
- **Dayanak:** `eval/degerlendirme-v1-ba.md`, `eval/degerlendirme-v1-ta.md`
- **Durum:** Yalnızca öneri. Henüz hiçbir persona veya rules dosyası değiştirilmedi. Uygulanan her değişiklik `eval/degisiklik-gunlugu.md` dosyasına işlenecek ve ilgili persona sürümü artırılacak (BA v2, TA v2).

## Uygulama durumu (Aşama 3)

| Öneri | Öncelik | Durum |
|---|---|---|
| O-1, O-5, BA-1, BA-2, TA-1, TA-2 | Y | **Uygulandı (v2)** (bkz. `eval/degisiklik-gunlugu.md`) |
| O-2, O-3, O-4, BA-3, BA-4, BA-5, TA-3, TA-4 | O | **Sonraya bırakıldı** |
| O-6, BA-6, TA-5, TA-6 | D | **Sonraya bırakıldı** |
| E-1 … E-4 (değerlendirme süreci) | — | **Sonraya bırakıldı.** E-4'ün karşılığı olarak `CLAUDE.md`'ye üzerine yazma yasağı eklendi. |

Not: O-1 öneri metnindeki "CLAUDE.md süreç notlarına atıf yapılmaz" cümlesi O-2'nin bir parçasıydı. Doküman dışı notu kapattığı için O-1 kapsamında `documentation-standards.md`'ye eklendi. O-2'nin CLAUDE.md uyarı satırı uygulanmadı.

---

Öncelik: **Y** = Yüksek (çıktı kalitesini veya TA'ya giden girdiyi doğrudan etkiliyor), **O** = Orta (tutarlılık), **D** = Düşük (biçim).

## 1. Ortak öneriler (her iki persona)

| # | Sorun (kaynak) | Önerilen kural | Dosya | Öncelik |
|---|---|---|---|---|
| O-1 | Yanıtın sonuna ana oturuma yönelik not ekleniyor. Not yanlış dosya adı veya senaryo adı önerebiliyor ya da proje sürecine atıf yapıyor (BA B-1, B-2: 4/6; TA-B1: 1/4). | "Yanıt yalnızca dokümandan oluşur. Doküman başlığından önce veya `Kaynaklar` bölümünden sonra metin yazılmaz. Dosya adı, kayıt yolu, araç kullanımı veya proje aşaması hakkında not eklenmez. Açılamayan kaynaklar gibi dokümanla ilgili bilgiler `Varsayımlar` veya `Kaynaklar` bölümüne yazılır." | `.claude/rules/documentation-standards.md` (yeni "Yanıt biçimi" başlığı) | Y |
| O-2 | Proje hafızasındaki süreç notları (Aşama 1 gözlemi gibi) çıktıya sızıyor (BA B-2). CLAUDE.md subagent'lara da yükleniyor. | O-1'deki kurala ek olarak: "CLAUDE.md'deki proje durumu ve değerlendirme notları görev girdisi değildir; dokümanda bunlara atıf yapılmaz." Ayrıca CLAUDE.md'deki "Durum" bölümünün başına bu bölümün yalnızca ana oturum için olduğunu belirten tek satırlık bir uyarı eklenmesi. | `.claude/rules/documentation-standards.md`, `CLAUDE.md` | O |
| O-3 | Varsayım, açık soru, kabul kriteri ve TA'ya özgü öğelerin ID biçimleri çalıştırmadan çalıştırmaya değişiyor (BA B-3, TA-B3). | ID önek tablosunun genişletilmesi: `V-xx` (varsayım), `S-xx` (BA açık soru), `R-xx` (risk), `T-xx` (teknoloji kararı), `K-xx` (bileşen), `G-xx` (TA'nın BA'ya geri sorusu, BA'nın `S-xx`'iyle çakışmaması için). Numaralar iki hanelidir. | `.claude/rules/documentation-standards.md` | O |
| O-4 | Kaynak gösterim biçimi değişiyor ([K1], satır içi, [Kısaltma]). TA şablonunda "Kaynaklar" bölümü yok ama 4/4 çalıştırmada farklı adlarla eklenmiş (TA-B2). | "Metin içinde kaynak `[K-xx]` biçiminde gösterilir; dokümanın son bölümü her zaman `Kaynaklar` tablosudur (#, Başlık, URL, Alınan bilgi)." Kaynaklar bölümü TA şablonuna da 13. bölüm olarak eklenmeli (bkz. TA-3). | `.claude/rules/documentation-standards.md`, `.claude/rules/research-rules.md` | O |
| O-5 | Kaynaktan aktarımda bağlam veya nüans kayboluyor: veri seti, karşılaştırma koşulu, iddianın kime ait olduğu (BA A-1, A-2, B-6; TA-A2). Olası neden: WebFetch tam metin yerine özet döndürüyor. | "Sayısal veya bulgusal bir iddia aktarılırken kaynağın koşulu (örneklem, veri seti, karşılaştırılan gruplar, tarih) birlikte yazılır. Karar etkileyen iddialarda kaynağın kendi ifadesinden kısa bir alıntı `Kaynaklar` tablosundaki 'Alınan bilgi' sütununa konur. İkincil haber yerine mümkünse birincil kaynak (makale, resmî belge) açılır. Bir iddianın yazar mı, muhabir mi, kullanıcı mı tarafından söylendiği belirtilir." | `.claude/rules/research-rules.md` | Y |
| O-6 | Satıcı kaynaklarındaki çıkar çatışması tutarsız biçimde belirtiliyor (BA B-7). | "Bir ürünün üreticisi veya satıcısı tarafından yazılmış karşılaştırma kaynakları kullanılırsa bu durum metinde belirtilir." | `.claude/rules/research-rules.md` | D |

## 2. Business Analyst

| # | Sorun (kaynak) | Önerilen kural | Dosya | Öncelik |
|---|---|---|---|---|
| BA-1 | Kritik araştırma alanları bazı çalıştırmalarda hiç ele alınmıyor: duplicate run2'de Google Play politikası ve silme davranışı, çarpım run1'de olağan müfredat (B-5). | Çalışma adımlarına zorunlu bir **araştırma kontrol listesi** eklenmesi: (1) en az 3 doğrudan rakip veya mevcut çözüm, yerleşik platform çözümleri dahil; (2) alanın resmî standartları veya düzenlemeleri (ör. müfredat, erişilebilirlik); (3) hedef platformun mağaza ve izin politikaları; (4) ilgili yöntemler veya algoritmalar. Bir başlık uygulanamıyorsa `Varsayımlar` bölümünde neden yazılır. | `.claude/agents/business-analyst.md` (Çalışma adımları) | Y |
| BA-2 | FR önceliği tutarsız: duplicate'te 2/3 çalıştırmada MoSCoW var, çarpımda hiç yok. MVP sınırı belirsiz kalıyor (B-4). | "Her FR bir öncelik taşır: M (MVP için zorunlu), S (güçlü öneri), C (kaynak kalırsa). FR tablosunda 'Öncelik' sütunu zorunludur." | `.claude/agents/business-analyst.md` (Çıktı şablonu), öncelik tanımı için `.claude/rules/documentation-standards.md` | Y |
| BA-3 | Kabul kriteri biçimi değişiyor (KK, AC, Verilen/O zaman) (B-3). | "Kabul kriterleri `Verilen … olduğunda … o zaman …` biçiminde yazılır; her US en az bir FR'ye atıf yapar." | `.claude/agents/business-analyst.md` (Çıktı şablonu) | O |
| BA-4 | Kaynaksız sayısal hedefler çalıştırmadan çalıştırmaya değişiyor (5 ↔ 10 dk, %98 ↔ %95). Doğru işaretleniyorlar, ama TA'ya giden değer rastgele (B-8). | "Kaynağa dayanmayan sayısal hedefler `NFR` satırında '(öneri, V-xx)' etiketiyle yazılır ve ilgili varsayımda hedefin nasıl türetildiği bir cümleyle açıklanır (rakip değeri, yaygın kullanım, sezgi)." Değişkenliği tamamen ortadan kaldırmaz ama gerekçeyi görünür kılar. | `.claude/agents/business-analyst.md` | O |
| BA-5 | BA'nın açık bıraktığı "mimariyi belirleyen" sorular (platform, çevrimdışı çalışma) TA'yı geçici bir platform seçmeye zorluyor (TA-B5). | "Ürün türünü veya platformu belirleyen bir bilgi görevde yoksa BA bunu `S-xx` olarak sorar **ve** iş açısından önerdiği varsayılanı (gerekçesiyle) `V-xx` olarak yazar. Böylece TA'nın girdisinde açık bir varsayılan olur." Kapsam kuralıyla çelişmez: BA teknoloji değil, hedef cihaz ve kullanım bağlamı önerir. | `.claude/agents/business-analyst.md`, `.claude/rules/scope-rules.md` (BA bölümü) | O |
| BA-6 | BA zaman zaman platform API adlarına ve belirli kütüphane/model örneklerine giriyor (A-3, belirsiz). | `scope-rules`'a açıklama: "BA platform kısıtlarını kullanıcı ve iş etkisi düzeyinde yazar (ör. 'silinenler 30 gün geri alınabilir'); API, sınıf, kütüphane veya model adı yazmaz. Bu ayrıntılar kaynak tablosunda kalabilir." | `.claude/rules/scope-rules.md` (BA bölümü) | D |

## 3. Technical Architect

| # | Sorun (kaynak) | Önerilen kural | Dosya | Öncelik |
|---|---|---|---|---|
| TA-1 | Doğruluğu etkileyen algoritma kararları (birebir tespit, gruplama) aynı girdide çalıştırmalar arasında değişiyor (TA-B4). | "Doğruluğu, güvenliği veya veri bütünlüğünü etkileyen her karar için en az 2 alternatif, bir **seçim ölçütleri** satırıyla (hangi gereksinim, hangi risk) karşılaştırılır. Seçim bu ölçütlere göre gerekçelendirilir." Bu, kararları tekrarlanabilir kılmayı amaçlar. | `.claude/agents/technical-architect.md` (Çalışma adımları) | Y |
| TA-2 | Seçilen kütüphane belgesindeki izin ve sürüm kısıtı gözden kaçabiliyor (Android 13 video izni) (TA-A2, TA-B7). | "Seçilen her dış kütüphane veya eklenti için belgesinden şunlar kontrol edilip `Teknoloji seçimleri` tablosuna yazılır: gerektirdiği izinler, desteklenen en düşük platform sürümleri, bilinen kısıtlar. Tasarım bu kısıtlarla çelişiyorsa risk olarak yazılır." | `.claude/agents/technical-architect.md` (Çalışma adımları) | Y |
| TA-3 | Şablonda olmayan ek bölümler farklı adlarla ekleniyor ("Kaynaklar", "Ek: Kaynaklar", "Ek B: Teknik varsayımlar") (TA-B2). | TA çıktı şablonuna iki bölüm eklenmesi: **13. Teknik varsayımlar** (`TV-xx`) ve **14. Kaynaklar**. Başka ek bölüm açılmaz. | `.claude/agents/technical-architect.md` (Çıktı şablonu) | O |
| TA-4 | BA'nın açık bıraktığı mimariyi belirleyen soruda (platform) TA tek bir geçici seçim yapıp her şeyi onun üzerine kuruyor (TA-B5). | "BA dokümanında mimariyi belirleyen bir açık soru varsa (platform, çevrimdışı, çok kullanıcılı), TA önce en fazla 2 seçeneği kısa bir tabloyla karşılaştırır, birini 'geçici varsayılan' olarak seçer ve diğer seçenekte değişecek bileşenleri `Riskler` bölümünde listeler." BA-5 uygulanırsa bu durum seyrekleşir. | `.claude/agents/technical-architect.md` | O |
| TA-5 | Test stratejisi ürün ölçeğine göre ayarlanmıyor (çarpım run1'de 3 motorlu E2E + otomatik erişilebilirlik hattı) (TA-B6). | `scope-rules` genel bölümüne: "Ürün büyüklüğüne uygunluk kuralı test stratejisi, araç seçimi ve süreç önerileri için de geçerlidir. Her test düzeyi en az bir FR/NFR veya US doğrulamasına bağlanır." | `.claude/rules/scope-rules.md` | D |
| TA-6 | BA'da olmayan küçük davranışlar (fiziksel klavye girişi) geri sorulmadan ekleniyor (TA-A1, belirsiz). | Mevcut kuralın açıklaştırılması: "Gereksinimde tanımlanmamış bir kullanıcı etkileşimi veya giriş yolu eklenmez. Gerekli görülürse 'geçici' işaretiyle tasarıma konur ve `G-xx` ile BA'ya sorulur." | `.claude/rules/scope-rules.md` (TA bölümü) | D |

## 4. Değerlendirme süreci (persona dışı)

| # | Sorun | Öneri | Dosya |
|---|---|---|---|
| E-1 | Bu oturumdaki `web-fetch` yardımcı agent'ının WebFetch aracı DNS hatası veriyor; link kontrolü yeni `claude -p` oturumlarıyla yapıldı. | Aşama 3'te link kontrolü için doğrudan aynı yöntem kullanılmalı (CLAUDE.md'de not var). | — |
| E-2 | TA kaynakları bu aşamada sistematik kontrol edilmedi (yalnızca 4 kritik iddia). | Aşama 3 kontrol listesine TA için "kaynaklar gerçek ve iddiayı destekliyor mu?" kriteri (TA-6) eklenmesi. | `eval/kontrol-listesi.md` |
| E-3 | Kontrol listesinde BA için "şablon dışı / doküman dışı metin" ayrı bir kriter değil; şimdilik BA-5'e gömülü. | BA-5 açıklamasına "doküman dışı metin yok" maddesi eklenmesi; TA için de aynı madde. | `eval/kontrol-listesi.md` |
| E-4 | Ana oturumun ayıklama betiğindeki yol hatası BA dosyalarının üzerine yazdı (geri yüklendi). | Kayıt betiği hedef dosya zaten varsa yazmayı reddetmeli (üzerine yazma koruması). | (scratchpad betiği; depo dışı) |

## 5. Aşama 3 için önerilen sıra

1. Y öncelikli ortak kurallar: O-1, O-5.
2. Y öncelikli persona değişiklikleri: BA-1, BA-2, TA-1, TA-2.
3. O ve D öncelikli maddeler (biçim ve tutarlılık).
4. Persona sürümlerini artırma (BA v2, TA v2) ve `eval/degisiklik-gunlugu.md` kaydı.
5. Tekrar test: aynı iki senaryo, BA 3'er + TA 2'şer çalıştırma. v1 ile karşılaştırma için aynı kontrol listesi kullanılacak.
