# Persona Değişiklik Günlüğü

Persona'larda yapılan her değişiklik bu tabloya eklenir. Değişiklik yapıldığında
ilgili agent dosyasındaki persona sürümü artırılır.

| Tarih | Persona | Sürüm | Gözlenen sorun | Yapılan değişiklik | Sonuç |
|---|---|---|---|---|---|
| 2026-11-22 | BA | v1 | — (ilk sürüm) | Persona oluşturuldu. | Duman testi: bkz. CLAUDE.md |
| 2026-11-22 | TA | v1 | — (ilk sürüm) | Persona oluşturuldu. | Duman testi: bkz. CLAUDE.md |
| 2026-11-22 | BA + TA (ortak) | v2 | O-1: Yanıtın sonuna ana oturuma not ekleniyor; yanlış dosya adı öneriliyor, proje sürecine atıf yapılıyor (BA 4/6, TA 1/4). | `documentation-standards.md`'ye "Yanıt biçimi" bölümü: yanıt yalnızca doküman; başlıktan önce ve son bölümden sonra metin yok; CLAUDE.md süreç notlarına atıf yok. | test edilecek |
| 2026-11-22 | BA + TA (ortak) | v2 | O-5: Kaynak iddiaları bağlamı kaybolarak aktarılıyor (EdWeek'te koşul, PMC'de katılımcı ve grup, imagededup'ta veri seti). | `research-rules.md`'ye "İddiaların bağlamıyla aktarılması" bölümü: koşul ve örneklem, iddianın sahibi, hesaplanan değer işareti, kritik iddiaya alıntı, birincil kaynak tercihi, WebFetch özetine karşı yeniden sorgu. | test edilecek |
| 2026-11-22 | BA | v2 | BA-1: Kritik araştırma alanları bazı çalıştırmalarda eksik (duplicate run2'de Play politikası, çarpım run1'de olağan müfredat). | `business-analyst.md` çalışma adımlarına 4 zorunlu araştırma başlığı: mevcut çözümler (≥3, yerleşik dahil), alan standartları ve düzenlemeler, platform ve mağaza politikaları, yöntemler. Uygulanamayan başlık Varsayımlar'da gerekçelendirilir. | test edilecek |
| 2026-11-22 | BA | v2 | BA-2: FR önceliği tutarsız (duplicate 2/3, çarpım 0/3); MVP sınırı belirsiz. | `business-analyst.md` çıktı şablonuna zorunlu "Öncelik" sütunu ve MoSCoW tanımı (M/S/C; W maddeleri Kapsam dışı'na). | test edilecek |
| 2026-11-22 | TA | v2 | TA-1: Doğruluğu etkileyen algoritma kararları (birebir tespit, gruplama, eşikler) aynı girdide çalıştırmalar arasında değişiyor. | `technical-architect.md` çalışma adımlarına: kritik kararlarda ≥2 alternatif, FR/NFR veya riske bağlı seçim ölçütleriyle karşılaştırma tablosu. | test edilecek |
| 2026-11-22 | TA | v2 | TA-2: Seçilen kütüphanenin belgesindeki izin kısıtı gözden kaçtı (duplicate run1: Android 13 video izni). | `technical-architect.md` çalışma adımlarına: her dış kütüphane için izinler, en düşük platform sürümleri ve bilinen kısıtların kontrolü; çelişki varsa risk. | test edilecek |
| 2026-11-22 | Süreç (ana oturum) | — | Aşama 2'de ana oturumun betik hatası 4 BA dosyasının üzerine yazdı. | `CLAUDE.md`'ye kural: `outputs/` altındaki dosyaların üzerine yazılmaz; hedef dosya varsa işlem durur ve kullanıcıya sorulur. | test edilecek |
