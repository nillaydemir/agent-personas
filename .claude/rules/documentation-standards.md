# Doküman Standartları

Bu kurallar tüm persona'ların ürettiği tüm dokümanlar için geçerlidir.

## Dil ve format
- Tüm dokümanlar **Türkçe** ve **markdown** formatında yazılır.
- Teknik terimlerin yaygın İngilizce karşılıkları parantez içinde verilebilir.

## Gereksinim ID'leri
Gereksinimler her zaman ID'li yazılır ve ID'ler doküman içinde sıralı, benzersiz olur:

| Önek | Anlamı | Örnek |
|---|---|---|
| `FR-xx` | Fonksiyonel gereksinim | FR-01 |
| `NFR-xx` | Fonksiyonel olmayan gereksinim | NFR-01 |
| `US-xx` | Kullanıcı hikâyesi | US-01 |

Başka bir dokümandaki gereksinime atıf yapılırken aynı ID kullanılır (ID değiştirilmez, yeniden numaralandırılmaz).

## Dosya adlandırma
```
<rol>-<senaryo>-v<persona-sürümü>-run<n>.md
```
- `<rol>`: `ba` veya `ta`
- `<senaryo>`: küçük harf, Türkçe karakter yok, tire ile ayrılmış (örn. `carpim-tablosu`)
- `<persona-sürümü>`: dokümanı üreten persona'nın sürümü (örn. `v1`)
- `<n>`: aynı senaryo ve sürüm için çalıştırma numarası

Örnek: `ba-carpim-tablosu-v1-run1.md`

## Doküman başlığı
Her doküman şu bilgi bloğuyla başlar:

```markdown
# <Doküman başlığı>

- **Tarih:** YYYY-MM-DD
- **Persona sürümü:** <rol> v<n>
- **Girdi:** <görev metni aynen> veya <kaynak doküman yolu>
```
