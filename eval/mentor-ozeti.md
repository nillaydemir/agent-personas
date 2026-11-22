# Mentor Özeti: BA ve TA Agent Persona'ları

**Tarih:** 2026-11-22 · **Son sürümler:** BA v2, TA v2 · **Ayrıntı:** `eval/karsilastirma-v1-v2.md`

## Persona'ların son hâli
- **Business Analyst (v2):** Görev metninden web araştırmasıyla kaynaklı bir iş analizi üretir. 4 zorunlu araştırma başlığını işler: mevcut çözümler, alan standartları, platform/mağaza politikaları, yöntemler. FR'leri MoSCoW ile önceliklendirir. Teknoloji kararı vermez.
- **Technical Architect (v2):** BA dokümanının dosya yolunu alır ve teknik gereksinim dokümanı üretir. Kritik kararları seçim ölçütlü tablolarla verir. Her kütüphanenin izin, sürüm ve kısıtlarını kontrol eder. Her kararı bir FR/NFR'ye bağlar; yeni gereksinim eklemez, belirsizliği BA'ya sorar.
- **Ortak kurallar** `.claude/rules/` altında: doküman standardı (yanıt yalnızca doküman), araştırma (kaynaksız iddia yok, iddia bağlamıyla aktarılır), kapsam.
- **Test düzeni:** Her çalıştırma ayrı bir `claude -p` oturumunda yapıldı. Görev metninin birebir gittiği ve çıktının değiştirilmeden kaydedildiği oturum kaydından doğrulandı.

## v1'de bulunan başlıca sorunlar
1. Yanıtın sonuna ana oturuma not ekleniyordu: BA'da 4/6, TA'da 1/4 çalıştırma. Bazı notlarda yanlış dosya adı vardı.
2. Kaynak iddiaları bağlamı kaybolarak aktarılıyordu: 24 linkten 4'ünde kısmi destek. Uydurma link ise yoktu.
3. BA'da kritik araştırma alanları bazı çalıştırmalarda eksikti: Play politikası, olağan müfredat.
4. FR önceliği tutarsızdı (2/6), MVP sınırı belirsizdi.
5. TA'da duplicate algoritma kararları çalıştırmalar arasında değişiyordu; bir kütüphane izin kısıtı gözden kaçmıştı.
6. Süreç: ana oturumun betik hatası 4 çıktı dosyasının üzerine yazdı. Dosyalar git'ten geri yüklendi.

## Yapılan değişiklikler (yalnızca yüksek öncelikli 6 öneri ve 1 süreç kuralı)
Yanıt yalnızca doküman · İddialar bağlamıyla · BA zorunlu araştırma başlıkları · BA MoSCoW · TA seçim ölçütlü kritik kararlar · TA kütüphane kontrolü · `outputs/` üzerine yazma yasağı.

## v2 sonuçları
| | v1 | v2 |
|---|---|---|
| BA puanı (ort., /30) | çarpım 27,0 · duplicate 28,3 | çarpım 29,0 · duplicate 29,0 |
| TA puanı (ort., /25) | çarpım 23,5 · duplicate 24,5 | çarpım 23,5 · duplicate 24,5 |
| Doküman dışı not | BA 4/6, TA 1/4 | **0/8** |
| Kaynak aktarım hatası | 24 linkte 4 kısmi | 8 iddiada 0 hata, 2 küçük nüans |
| FR önceliği | 2/6 | **4/4** |
| Kütüphane izin kısıtı (Android 13 video) | 1/2 TA yakaladı | **2/2** |
| Kör değerlendirme (v1 ↔ v2, A/B maskeli) | — | v2: 3 kazanç, 1 eşitlik, 0 kayıp |

- 7 değişikliğin 5'i sorunu çözdü, 1'i büyük ölçüde çözdü (bağlamla aktarma), 1'i kısmen çözdü (TA kararlarının tutarlılığı).
- TA'daki kazanım puandan çok niteliksel:
  - Kararlar yapılandırılmış gerekçeyle veriliyor.
  - Kaynak doğruluğu arttı. Örnek: v1'in açık bıraktığı Safari veri silme riski, v2'de WebKit birincil kaynağıyla kapatıldı.

## Hâlâ açık sorunlar
1. **TA karar tutarlılığı:** Duplicate'te gruplama yöntemi ve eşik değerleri hâlâ çalıştırmaya göre değişiyor. Ölçütleri aynı derecede sağlayan seçenekler arasında seçim sabit değil.
2. **Platform kararı BA içeriğine aşırı duyarlı:** BA v2 mağaza politikalarını işleyince TA çarpımda web yerine Flutter seçti. Platform BA'da hâlâ açık soru.
3. **Biçim tutarsızlığı:** Varsayım, soru ve kaynak ID'leri ile kaynak bölümü adı çalıştırmadan çalıştırmaya değişiyor. Bir TA'da atıf numaralandırma hatası oluştu.
4. **Doküman uzunluğu ve maliyet:** BA v2 dokümanları %33–42 daha uzun, WebFetch çağrısı daha fazla.
5. **Bir çalıştırmada performans tasarımı zayıfladı:** Kritik karar ölçütleri tüm ilgili NFR'leri kapsamadı.
6. TA, BA'ya sormadan küçük davranışlar eklemeye devam ediyor; test stratejisi ürün ölçeğine göre ayarlanmıyor.

## Sonraki adım önerileri
1. **Orta öncelikli önerileri uygula** (`eval/aşama-3-yapilacaklar.md`):
   - Ortak ID ve kaynak gösterimi (O-3, O-4).
   - BA'nın mimariyi belirleyen açık sorularda iş açısından varsayılan önermesi (BA-5) ve TA'nın iki seçenekli karşılaştırması (TA-4).
   - TA şablonuna "Teknik varsayımlar" ve "Kaynaklar" bölümleri (TA-3).
2. **TA-1'i güçlendir:** Kritik karar tablosunda ilgili tüm NFR'lerin (özellikle performans) ölçüt olması zorunlu olsun. Ölçütleri eşit sağlayan seçenekler için bir eşitlik bozma kuralı tanımlansın (ör. "en sade olan").
3. **Tutarlılığı doğru ölç:** TA v3'ü aynı girdiyle en az 3 kez çalıştır. Bu aşamada v2 TA'nın iki çalıştırmasının girdisi farklıydı, bu yüzden kanıt sınırlı.
4. **Maliyet sınırı:** "Basit" ürünlerde doküman uzunluğu ve kaynak sayısı için ölçeğe uygun bir üst sınır tanımla.
5. **Değerlendirme:** Kontrol listesine TA kaynak doğrulama kriteri ekle. Kör değerlendirmeyi kalıcı yöntem olarak sürdür; v1 puanlarında kendi değerlendirmemden daha katıydı ve iki yerde haklı çıktı.
