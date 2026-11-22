# Değerlendirme Kontrol Listesi

Her çalıştırma (run) için bu listenin bir kopyası doldurulur.

## Puan ölçeği
| Puan | Anlamı |
|---|---|
| 1 | Karşılanmamış / ciddi sorun var |
| 2 | Büyük ölçüde eksik |
| 3 | Kısmen karşılanmış, belirgin eksikler var |
| 4 | Büyük ölçüde karşılanmış, küçük eksikler var |
| 5 | Tam karşılanmış |

## Değerlendirme bilgileri
- **Tarih:**
- **Senaryo:**
- **Değerlendirilen dosya(lar):**
- **Persona sürümleri:** BA v_ / TA v_

---

## A. Business Analyst
| # | Kriter | Nasıl kontrol edilir | Puan (1-5) | Not |
|---|---|---|---|---|
| BA-1 | Araştırma yapılmış mı? | Rakip/yöntem incelemesi gerçek bulgular içeriyor mu, genel geçer bilgiyle mi yetinilmiş? | | |
| BA-2 | Kaynaklar gerçek ve erişilebilir mi? | Linkler açılıyor mu, içerikleri iddiayı destekliyor mu? (Örneklem alıp elle kontrol et.) | | |
| BA-3 | Gereksinimler ölçülebilir mi? | FR/NFR'ler test edilebilir mi, NFR'lerde sayısal eşik var mı, ID'ler doğru mu? | | |
| BA-4 | Kapsam uygun mu? | Ürün büyüklüğüne uygun mu, gereksiz özellik veya teknoloji kararı var mı? | | |
| BA-5 | Şablona uyulmuş mu? | 10 bölüm doğru sırada mı, doküman başlığı (tarih, sürüm, girdi) var mı, Türkçe mi? | | |
| BA-6 | Açık sorular anlamlı mı? | Sorular gerçek belirsizlikleri mi hedefliyor, yanıtı kararları etkiler mi? | | |
| | **BA toplam** | | **/30** | |

## B. Technical Architect
| # | Kriter | Nasıl kontrol edilir | Puan (1-5) | Not |
|---|---|---|---|---|
| TA-1 | Her BA gereksinimi karşılanmış mı? | BA'daki her FR/NFR için somut bir teknik karşılık var mı? | | |
| TA-2 | Uydurma gereksinim var mı? (5 = hiç yok) | BA dokümanında olmayan gereksinim/özellik eklenmiş mi? | | |
| TA-3 | Kararlar gerekçeli mi? | Teknoloji ve algoritma seçimleri gerekçeli ve gereksinime bağlı mı, alternatifler anılmış mı? | | |
| TA-4 | Mimari ürün büyüklüğüne uygun mu? | Aşırı mühendislik (gereksiz servis, altyapı, katman) var mı? | | |
| TA-5 | İzlenebilirlik tablosu eksiksiz mi? | Tabloda BA'daki tüm FR/NFR ID'leri var mı, eşlemeler doğru mu? | | |
| | **TA toplam** | | **/25** | |

## C. Tutarlılık (aynı görevin farklı çalıştırmaları)
| # | Kriter | Nasıl kontrol edilir | Puan (1-5) | Not |
|---|---|---|---|---|
| T-1 | Yapı benzer mi? | Bölümler, sıralama ve ID düzeni çalıştırmalar arasında aynı mı? | | |
| T-2 | Kapsam benzer mi? | Gereksinim sayısı ve içeriği belirgin biçimde değişiyor mu? | | |
| T-3 | Temel kararlar benzer mi? (TA) | Ana mimari ve teknoloji kararları çalıştırmalar arasında tutarlı mı? | | |
| | **Tutarlılık toplam** | | **/15** | |

---

## Genel not ve gözlemler
- **Güçlü yönler:**
- **Zayıf yönler / gözlenen sorunlar:**
- **Persona'da önerilen değişiklik:** (varsa `eval/degisiklik-gunlugu.md` dosyasına işle)
