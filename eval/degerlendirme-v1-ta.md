# Değerlendirme: Technical Architect v1 (Aşama 2)

- **Tarih:** 2026-11-22
- **Değerlendirilen persona:** TA v1 (persona ve rules dosyaları bu aşamada değiştirilmedi)
- **Ölçüt:** `eval/kontrol-listesi.md` (TA-1 … TA-5, T-1 … T-3; 1-5 puan)
- **TA girdileri (kullanıcı onayıyla):**
  - Çarpım tablosu: `outputs/carpim-tablosu/ba-carpim-tablosu-v1-run2.md` (17 FR, 10 NFR, 7 US)
  - Duplicate finder: `outputs/duplicate-finder/ba-duplicate-finder-v1-run3.md` (16 FR, 11 NFR, 9 US)
- **Değerlendirilen dosyalar:**
  - `outputs/carpim-tablosu/ta-carpim-tablosu-v1-run{1,2}.md`
  - `outputs/duplicate-finder/ta-duplicate-finder-v1-run{1,2}.md`

## 0. Test yöntemi

- Her çalıştırma depoda açılan **ayrı ve yeni** bir `claude -p` oturumunda tek bir `technical-architect` çağrısıyla yapıldı. 4 çalıştırma paralel koştu.
- Görev metni: `<mutlak dosya yolu> dosyasındaki analiz dokümanından teknik gereksinim dokümanını çıkar.` Subagent'a giden `prompt` alanının bu metinle birebir aynı olduğu 4/4 çalıştırmada oturum kaydından doğrulandı.
- Çıktılar oturum kaydından program ile alınıp değiştirilmeden kaydedildi. Kaydedilen metin, subagent'ın son mesajıyla 4/4 aynı. Agent'lar dosya yazmadı.

| Çalıştırma | Read | WebSearch | WebFetch | Kelime |
|---|---|---|---|---|
| çarpım run1 | 4 | 0 | 13 | ~5.500 |
| çarpım run2 | 4 | 0 | 10 | ~6.000 |
| duplicate run1 | 7 | 1 | 22 | ~6.800 |
| duplicate run2 | 7 | 0 | 15 | ~7.000 |

Read çağrıları: BA dokümanı, 3 kural dosyası ve büyük WebFetch sayfalarının geçici kayıtları. Dört çalıştırmada da BA dokümanı ilk iş olarak okundu.

> **Süreç olayı (ana oturum hatası, agent'larla ilgisi yok):** TA çıktılarını kaydederken ana oturumun ayıklama betiğindeki bir yol hatası yüzünden 4 BA dosyası (çarpım run1-2, duplicate run1-2) TA metniyle üzerine yazıldı. Dosyalar hemen `git checkout` ile son commit'ten geri yüklendi ve HEAD ile birebir aynı oldukları doğrulandı. Üzerine yazma TA çalıştırmaları **bittikten sonra** oldu; TA'lar okuma sırasında bozulmamış BA dosyalarını gördü. Etkilenen dosyalardan hiçbiri TA girdisi değildi.

**Kaynak kontrolü:** Kullanıcı bu aşamada TA için link kontrolü istemedi. Yine de karar etkileyen 4 iddia ayrıca kontrol edildi (bkz. 3.4).

---

## 1. Puan özeti

| Kriter | çarpım run1 | çarpım run2 | duplicate run1 | duplicate run2 |
|---|---|---|---|---|
| TA-1 Her BA gereksinimi karşılanmış mı? | 5 | 5 | 5 | 5 |
| TA-2 Uydurma gereksinim yok mu? (5 = hiç yok) | 4 | 4 | 5 | 5 |
| TA-3 Kararlar gerekçeli mi? | 5 | 5 | 4 | 5 |
| TA-4 Mimari ürün büyüklüğüne uygun mu? | 4 | 5 | 5 | 5 |
| TA-5 İzlenebilirlik tablosu eksiksiz mi? | 5 | 5 | 5 | 5 |
| **Toplam (/25)** | **23** | **24** | **24** | **25** |

---

## 2. İzlenebilirlik kontrolü

Kontrol iki şekilde yapıldı:
1. **Program ile:** BA'daki her FR/NFR ID'si TA'nın 11. bölümündeki tabloda satır olarak var mı? TA metninde BA'da olmayan bir FR/NFR/US ID'si geçiyor mu?
2. **Elle:** Her satırda gösterilen bileşen veya karar, TA dokümanında gerçekten tanımlanmış mı?

### 2.1 Çarpım tablosu (BA run2 → TA run1, run2)

| BA ID | BA gereksinimi (özet) | TA run1 karşılığı | TA run2 karşılığı |
|---|---|---|---|
| FR-01 | Tablo seçimi (1–10, seçimsiz başlatılamaz) | ✅ B-01, B-04 soru havuzu | ✅ K2-E1, K5 |
| FR-02 | Tablonun 10 satırını görme | ✅ B-02 | ✅ K2-E2 |
| FR-03 | Tekrarlı toplama gösterimi | ✅ B-02 (koddan üretim) | ✅ K2-E2 + metin üretim testi |
| FR-04 | "a × b = ?", sayısal tuş takımı, ≤3 hane | ✅ B-03, T-07 | ✅ K2-E3, K3, T7 |
| FR-05 | Sıralı/karışık, art arda aynı soru yok | ✅ Fisher–Yates, `bitişikleriAyır` | ✅ Fisher–Yates, `ardışıkTekrarıGider` |
| FR-06 | Oturum uzunluğu 5/10/20 | ✅ B-01, B-04 | ✅ K2-E6, K5 |
| FR-07 | Anında, suçlayıcı olmayan geri bildirim | ✅ B-03, B-13 | ✅ K2-E3, K4, K10 |
| FR-08 | Yanlış ≥2 soru sonra tekrar | ✅ tekrar kuyruğu (`step+3`) | ✅ tekrar ekleme (`i+3`) |
| FR-09 | Zayıf sorulara ≥%50 | ✅ `WEAK_RATIO` kotası | ✅ W/L ayrımı, ⌈N/2⌉ |
| FR-10 | Yeni/Çalışılıyor/Öğrenildi | ✅ `FactRecord`, türetilmiş durum | ✅ `streak`, türetilmiş durum |
| FR-11 | 10×10 ızgara, renk + simge | ✅ B-08, B-14 | ✅ K2-E5 |
| FR-12 | Oturum sonu özeti | ✅ B-07 | ✅ K2-E4 |
| FR-13 | Tebrik ve kalıcı yıldız | ✅ `yıldızDenetle`, `stars` | ✅ K6, `stars` |
| FR-14 | İsteğe bağlı süre 10/7/5 sn | ✅ B-06, `performance.now()` | ✅ K8, monoton saat |
| FR-15 | İlerlemenin korunması | ✅ `localStorage` + `persist()` | ✅ `localStorage` + `persist()` |
| FR-16 | Yetişkin doğrulamalı sıfırlama | ✅ B-10 | ✅ K9 |
| FR-17 | Ses kapatma | ✅ B-11 | ✅ K11 (koşullu) |
| NFR-01 | Türkçe, yönerge ≤8 kelime | ✅ B-13 + kelime testi | ✅ K10 + `yonerge.*` testi |
| NFR-02 | ≤3 dokunuşla başlatma | ✅ 2 dokunuş tasarımı + E2E | ✅ 2 dokunuş + kullanılabilirlik testi |
| NFR-03 | Geri bildirim ≤0,5 sn, sonraki ≤1 sn | ✅ ağsız işlem + ölçüm | ✅ O(1) yol + ölçüm |
| NFR-04 | Kontrast | ✅ B-14 + axe | ✅ palet + formül testi |
| NFR-05 | Dokunma ≥24×24 | ✅ T-07, B-14 | ✅ K3 |
| NFR-06 | Renge bağımlı olmama | ✅ simge + metin | ✅ simge + metin |
| NFR-07 | Kişisel veri yok | ✅ sunucusuz + CSP | ✅ sunucusuz + CSP |
| NFR-08 | Reklam/satın alma yok | ✅ SDK yok + CSP | ✅ SDK yok + CSP |
| NFR-09 | Pedagojik ton | ✅ B-03, B-11, B-13 | ✅ K4, K10, K11 |
| NFR-10 | Kapanmada veri korunur | ✅ geri bildirimden önce yazma | ✅ eşzamanlı yazma |

**Sonuç:** 27/27 gereksinim iki çalıştırmada da karşılanmış. **Karşılığı olmayan BA gereksinimi yok.** TA metinlerinde BA'da olmayan bir FR/NFR/US ID'si yok.

### 2.2 Duplicate finder (BA run3 → TA run1, run2)

| BA ID | BA gereksinimi (özet) | TA run1 karşılığı | TA run2 karşılığı |
|---|---|---|---|
| FR-01 | İzin öncesi bilgilendirme | ✅ OnboardingScreen, tek giriş noktası | ✅ OnboardingScreen + iOS otomatik uyarının bastırılması |
| FR-02 | Tam/sınırlı/reddedilmiş izin | ✅ PermissionService, `presentLimited` | ✅ PermissionService, `requestPermissionExtend` |
| FR-03 | Tarama, ilerleme, iptal | ✅ ScanCoordinator, `PMCancelToken` | ✅ ScanOrchestrator, iptal belirteci |
| FR-04 | Birebir kopya | ✅ boyut ön elemesi + SHA-256 + bayt doğrulama | ✅ boyut + pHash aday filtresi + SHA-256 |
| FR-05 | Benzer kopya | ✅ dHash + pHash, BK-ağacı | ✅ dHash + pHash, BK-ağacı |
| FR-06 | İki sekme, tek sekme kuralı | ✅ birebir üyeleri Benzer'den dışlama | ✅ birebir grubun yalnızca saklanan üyesi |
| FR-07 | 3 hassasiyet, ≤5 sn güncelleme | ✅ kenarlar gevşek eşikte saklanır | ✅ aynı yaklaşım |
| FR-08 | Küçük resim + meta veri | ✅ ResultsUI | ✅ GroupDetailScreen |
| FR-09 | Saklama önerisi, favori koruma | ✅ KeeperPolicy | ✅ RecommendationService |
| FR-10 | İşaret değiştirme, karşılaştırma, uyarı | ✅ ResultsUI | ✅ CompareView |
| FR-11 | Toplu silme | ✅ DeletionService | ✅ DeletionService |
| FR-12 | Onaylı ve geri alınabilir silme | ✅ `moveToTrash`, iOS tek blok | ✅ aynı |
| FR-13 | Silme özeti | ✅ sonuç doğrulama | ✅ DeletionSummaryScreen |
| FR-14 | "Kopya değil" | ✅ `not_duplicate_pairs` | ✅ `not_duplicate_pair` |
| FR-15 | Artımlı tarama | ✅ `asset_id` + `modified_at` farkı | ✅ aynı |
| FR-16 | Taramaya devam | ✅ `scan_sessions` + kontrol noktası | ✅ `scan_session` + toplu yazma |
| NFR-01 | Gizlilik, çevrimdışı | ✅ ağ yok, `INTERNET` izni yok | ✅ aynı + yedek dışı tutma önerisi |
| NFR-02 | 10 bin ≤10 dk | ✅ bütçe 60 ms/varlık | ✅ aynı bütçe |
| NFR-03 | Artımlı ≤30 sn | ✅ | ✅ |
| NFR-04 | Doğruluk %0 / %95 / %85 | ✅ bayt doğrulama + kalibrasyon | ✅ SHA-256 + kalibrasyon/değerlendirme ayrımı |
| NFR-05 | 50 bin, çökme yok | ✅ | ✅ |
| NFR-06 | ≤200 ms yanıt | ✅ işçi izolatları | ✅ işçi isolate'leri |
| NFR-07 | Onaysız değişiklik yok | ✅ çift onay, `MANAGE_MEDIA` yok | ✅ aynı |
| NFR-08 | iOS + Android | ✅ minSdk 30 / iOS 14 önerisi | ✅ Android ≤10'da silme kapalı |
| NFR-09 | Play politikası | ✅ en az izin + beyan | ✅ en az izin + beyan + R-05 |
| NFR-10 | Erişilebilirlik | ✅ `meetsGuideline` + elle test | ✅ elle test |
| NFR-11 | Türkçe | ✅ gen-l10n + çevrilmemiş mesaj kontrolü | ✅ gen-l10n + ARB kontrolü |

**Sonuç:** 27/27 gereksinim iki çalıştırmada da karşılanmış. **Karşılığı olmayan BA gereksinimi yok.** TA metinlerinde BA'da olmayan bir FR/NFR/US ID'si yok.

### 2.3 TA'da olup BA'da olmayanlar (uydurma gereksinim kontrolü)

Yeni bir FR veya NFR ID'si hiçbir çalıştırmada üretilmemiş. Aşağıdaki tablo, BA'da açıkça olmayan ama TA'nın tasarıma koyduğu **davranışları** listeler. Her birinin işaretlenip işaretlenmediği belirtildi.

| Çalıştırma | TA'nın eklediği davranış | BA'ya soruldu mu / "geçici" işareti var mı? | Değerlendirme |
|---|---|---|---|
| çarpım run1, run2 | Fiziksel klavyede rakam tuşlarıyla cevap girişi | Hayır | **Küçük kapsam genişlemesi.** FR-04 "sayısal tuş takımı" diyor. Masaüstü tarayıcıyı da kapsayan platform kararının doğal sonucu, ama yeni bir giriş yolu. |
| çarpım run1 | `MAX_REASK = 2` (tekrar sorusunun en fazla 2 kez yeniden sorulması) | Evet (S-06) | Kabul edilebilir: sonlanma garantisi, açıkça soruldu. |
| çarpım run1 | Ana liste bitince N sayımına girmeyen "ara sorular" eklenmesi | Evet (S-05) | Kabul edilebilir: FR-08 ile US-01 çatışmasını çözmek için, açıkça soruldu. |
| çarpım run2 | Doğru cevapta ~0,8 sn sonra otomatik geçiş; yanlışta "Devam" düğmesi | Evet (S-13) | Kabul edilebilir: geçici, soruldu. Ancak UI akışı belirleyen bir karar. |
| çarpım run2 | Uygulama arka plana geçince süre sayacının durması | Evet (S-6) | Kabul edilebilir: geçici, soruldu. |
| çarpım run2 | Bozuk kaydın ayrı anahtara yedeklenmesi | — | NFR-10'un uygulama ayrıntısı; gereksinim değil. |
| duplicate run1 | iOS silmede 60 sn zaman aşımı | — | Belgelenmiş bir platform hatasına karşı önlem (kaynak doğrulandı, 3.4). Gereksinim değil. |
| duplicate run1, run2 | Uygulama arka plana geçince taramanın duraklaması | Evet (G-11 / Q-08) | Kabul edilebilir: NFR-02 "ön planda" diyor; soruldu. |
| duplicate run2 | Android 10 ve altında silme özelliğinin kapalı tutulması | Evet (Q-02) | Kabul edilebilir ama işlev daraltıyor; soruldu. |
| duplicate run2 | İndeks veritabanının cihaz yedeğine dahil edilmemesi | Evet (Q-10) | Kabul edilebilir: öneri olarak sunulmuş, soruldu. |

**Sonuç:** Uydurma FR/NFR yok. Çarpım tablosunda fiziksel klavye girişi BA'ya sorulmadan eklenmiş tek davranış (TA-2 = 4). Diğer bütün ekler ya uygulama ayrıntısı ya da "geçici" işaretiyle BA'ya sorulmuş.

---

## 3. Senaryoya özel kontroller

### 3.1 Çarpım tablosu: ölçek kontrolü

| Konu | run1 | run2 | Değerlendirme |
|---|---|---|---|
| Mimari tarz | Sunucusuz tek sayfa web uygulaması (PWA), tek katman | Sunucusuz tek sayfa web uygulaması, 3 katman (UI / alan / altyapı) | ✅ Uygun. Sunucu, veritabanı, hesap ve API yok; gerekçesi NFR-07/08 ve BA kapsam dışı listesi. |
| UI çatısı | Yok (DOM API + küçük yönlendirici) | Yok | ✅ Uygun. "6 ekran için framework gereksiz" gerekçesi açık. |
| Dil ve derleme | TypeScript + Vite | TypeScript + Vite | ✅ Hafif. |
| Depolama | `localStorage`, tek JSON kaydı (birkaç KB) | Aynı | ✅ Uygun. IndexedDB açıkça gerekçeyle reddedilmiş. |
| Algoritmalar | Fisher–Yates + kota tabanlı seçim; Leitner ve aralıklı tekrar bilinçli olarak alınmamış | Aynı | ✅ Uygun: "BA'da yok" gerekçesiyle sade tutulmuş. |
| Test araçları | Vitest + **Playwright (3 tarayıcı motoru, telefon/tablet öykünmesi) + axe** + gerçek cihaz + kullanılabilirlik | Vitest + elle cihaz testleri + kullanılabilirlik | run1 **biraz ağır:** küçük bir ürün için 3 motorlu E2E ve otomatik erişilebilirlik hattı. run2 daha orantılı. |
| Ek güvenlik | CSP, `persist()` | CSP, `persist()`, sistem fontu | ✅ Hafif ve gerekçeli. |

**Sonuç:** İki çalıştırmada da mimari ürün büyüklüğüne uygun; aşırı karmaşık değil. run1'in yalnızca test hattı küçük bir ürün için fazla (TA-4 = 4). Not: İki çalıştırma da BA'nın açık bıraktığı platform sorusunu "geçici: web uygulaması" diye cevaplamış ve S-01/S-02 olarak geri sormuş. Bu TA'nın yetki alanında ve doğru işaretlenmiş. Ancak BA'nın cevabı "mağaza uygulaması" olursa mimarinin büyük kısmı değişir (her iki çalıştırma da bunu risk olarak yazmış).

### 3.2 Duplicate finder: algoritma seçimleri ve Flutter kısıtı

| Konu | run1 | run2 | Gerekçeli mi? |
|---|---|---|---|
| **Flutter kısıtı** | T-01: "Paydaş kısıtı (V-03)". Tüm paketler Flutter/Dart: `photo_manager`, `sqflite`, `crypto`, `image`, isolate'ler, gen-l10n. | Aynı; alternatif "Kısıt olduğu için değerlendirilmedi". | ✅ İkisi de kısıta uymuş; başka çatı önerilmemiş. Yerel kod yalnızca "eklenti yetmezse dar uzantı" olarak risk planında. |
| **Algısal özet türü** | dHash + pHash birlikte; aHash ve embedding gerekçeyle reddedilmiş. | Aynı; aHash, yalnızca dHash ve embedding için ayrı "seçilmeme gerekçesi" tablosu. | ✅ İkisi de NFR-04b (%95 kesinlik) hedefine ve kaynağa bağlı. |
| **Benzerlik eşiği** | Sıkı / Normal / Gevşek = 4 / 8 / 12 (her iki özet) | 5 / 8 / 10 | ✅ İkisi de "başlangıç değeri, kalibre edilecek" diyor. run2 her değerin dayanağını yazmış ("Normal: kaynağı yok"). run2 ayrıca kalibrasyon ve değerlendirme setini ayırmış (aşırı uyum önlemi). |
| **Birebir kopya** | Dosya boyutu ön elemesi → SHA-256 → bayt bayt doğrulama | Aynı piksel boyutu + pHash ≤ Tx aday filtresi → SHA-256 | run1 en güçlü doğruluk garantisini veriyor. Ancak **iç tutarsızlık var:** ön eleme bayt cinsinden dosya boyutuna dayanıyor, oysa R-01 bu bilginin `photo_manager`'da ucuza alınamadığını kabul ediyor. Ana yol, belirsiz bir yeteneğe bağlı (TA-3 = 4). run2 bu kısıtı görüp tasarımı ona göre kurmuş, kaçırma riskini (R-02) ve geri dönüş yolunu yazmış. Ayrıca iOS'ta düzenlenmiş fotoğrafın orijinali ile kopyasının bayt düzeyinde aynı görünme riskini (Q-07) **yalnızca run2** yakalamış. |
| **Gruplama** | Union-find (bağlı bileşen); zincirleme riski R-04'te | Yıldız (lider) kümeleme; zincirlemeyi yapısal olarak önlüyor | ✅ İkisi de gerekçeli. run2'nin seçimi NFR-04b kesinliğiyle daha uyumlu. İkisi de US-06'nın her durumda garanti edilemeyeceğini fark etmiş. |
| **Büyük kütüphanede performans** | BK-ağacı (dHash), artımlı sorgu, işçi havuzu, 60 ms/varlık bütçesi, ölçüm planı | BK-ağacı (pHash), aynı bütçe, ayarlanacak parametre listesi | ✅ İkisi de O(N²)'yi gerekçeyle reddetmiş, budamanın gevşek eşikte zayıflayacağını risk olarak yazmış. Sayısal değer ölçülmüş gibi sunulmamış. |
| **Cihaz üzerinde işleme** | Özet OS küçük resminden (JPEG); HEIC sorunu bu yolla çözülüyor | Aynı; kayıpsız PNG tercihi prototipte doğrulanacak | ✅ |
| **İzinler** | Yalnızca `READ_MEDIA_IMAGES` (+ kısmi erişim izni); `MANAGE_MEDIA` ve `ACCESS_MEDIA_LOCATION` yok | Aynı + R-05: photo_manager Android 13'te video iznini de istiyor | run1'de **olgusal eksik:** photo_manager belgesi Android 13'te `READ_MEDIA_IMAGES` ve `READ_MEDIA_VIDEO` ikisinin de gerektiğini söylüyor (3.4'te doğrulandı). run1 bunu gözden kaçırmış. run2 doğru tespit edip Play beyanı riskine bağlamış. |
| **Gizlilik** | Ağ kütüphanesi yok, `INTERNET` izni yok (CI kontrolü), iCloud'dan indirme yok | Aynı + yedek dışı tutma önerisi | ✅ |
| **Silme** | `moveToTrash`, `deleteWithIds` Android'de yasak, iOS tek blok, zaman aşımı (forum kaynağı) | Aynı | ✅ İkisi de BA V-05'in onay diyaloğu kısmını Apple kaynağıyla doğrulamış. |

**Sonuç:** Algoritma seçimleri iki çalıştırmada da gerekçeli, kaynaklı ve BA gereksinimlerine bağlı. Flutter kısıtına tam uyulmuş. run2 iki kritik platform ayrıntısında run1'den daha doğru: video izni gerekliliği ve dosya boyutunun alınamaması.

### 3.3 "BA'ya geri sorular" bölümünün kalitesi

| Çalıştırma | Soru sayısı | Güçlü örnekler | Zayıf yön |
|---|---|---|---|
| çarpım run1 | 18 | S-05 (FR-08 ile oturum sonu), S-08 (oturum içi tekrarın "Öğrenildi" ölçütünü gevşetmesi), S-13 (NFR-03 ile doğru cevabı okuma süresi çatışması) | Birkaçı BA'nın kendi açık sorularının tekrarı (S-01, S-02, S-17, S-18). Bağlantı kurulmuş ama yeni bilgi az. |
| çarpım run2 | 16 | S-3 (FR-08 ↔ US-01 çelişkisi açıkça adlandırılmış), S-15 (FR-12'deki "toplam"a tekrarlar dahil mi?), S-12 (sıfırlama ayarları kapsar mı?) | Tablo biçiminde, her sorunun "geçici teknik davranış"ı yazılı. Çok net. |
| duplicate run1 | 14 | G-01 (US-06 matematiksel olarak garanti edilemiyor, alternatif ölçüt önerisi), G-04 (çöpteki fotoğraf hâlâ yer kaplıyor, "kazanılan alan" yanıltıcı) | G-09 bir soru değil, bilgi notu. |
| duplicate run2 | 13 | Q-07 (iOS düzenlenmiş fotoğraf ile "birebir" tanımı), Q-10 (yedeğe girmek "cihaz dışı" mı?), Q-11 (gelir modeli NFR-01'i bozabilir) | — |

**Sonuç:** Dört çalıştırmada da bölüm anlamlı. Sorular BA dokümanındaki gerçek çelişkileri ve test edilemezlikleri buluyor (FR-08 ↔ US-01, US-06'nın garanti edilemezliği, "kazanılan alan" tanımı). Her soruya geçici teknik davranış eklenmiş, bu sayede tasarım soruları beklemeden ilerleyebiliyor. Kimlik biçimi farklı: çarpımda `S-xx`, duplicate'te BA'nın `S-xx`'iyle çakışmasın diye `G-xx` / `Q-xx`.

### 3.4 Karar etkileyen iddiaların kontrolü

| İddia | Çalıştırma | Kaynak | Sonuç |
|---|---|---|---|
| Safari, 7 gün etkileşimsiz kökenin script verisini siler; ana ekran muafiyeti teyit edilmiyor | çarpım run1, run2 | MDN Storage quotas | ✅ Doğrulandı ("last seven days of browser use"). Sayfada "home screen" geçmiyor; TA'ların "teyit edilmiyor" ifadesi doğru. |
| `photo_manager` Android 13'te `READ_MEDIA_IMAGES` ve `READ_MEDIA_VIDEO` ikisini birden istiyor | duplicate run2 (R-05) | pub.dev photo_manager | ✅ Doğrulandı: "READ_MEDIA_IMAGES and READ_MEDIA_VIDEO are both required whether requesting images or videos." **run1 P-01 bu bilgiyle çelişiyor.** |
| `moveToTrash` Android 11+; `MANAGE_MEDIA` onay diyaloğunu kaldırır | duplicate run1, run2 | pub.dev photo_manager | ✅ Doğrulandı. |
| iOS `deleteAssets` diyalog göstermeden askıda kalabiliyor; FB18720533; ProRAW tetikleyici; önerilen geçici çözüm yok | duplicate run1 | Apple Developer Forums 806349 | ✅ Doğrulandı. Not: ProRAW tetikleyicisi kullanıcı gözlemi, Apple onaylamamış; TA "bir raporda" diyerek doğru aktarmış. |

---

## 4. Tutarlılık (aynı girdinin 2 çalıştırması)

### 4.1 Çarpım tablosu

| Boyut | run1 | run2 |
|---|---|---|
| Bölüm yapısı | 12 şablon bölümü + Kaynaklar | 12 şablon bölümü + Kaynaklar |
| Mimari | Sunucusuz web SPA / PWA, TS, framework yok, Vite, `localStorage` | Aynı (PWA ifadesi yok, "web uygulaması") |
| Soru seçimi | Fisher–Yates + ⌈N/2⌉ zayıf kota | Aynı |
| Öğrenme durumu | `attempts` + `correctStreak` | Yalnızca `streak` (0–3); kayıt yoksa "Yeni" |
| FR-08 oturum sonu | Ara sorular eklenir (aralık korunur) | Aralık gevşer, dolgu yok |
| Tekrarın tekrarı | En fazla 2 (`MAX_REASK`) | Yok |
| Tebrik zamanı | Anında bildirim + özet | Yalnızca oturum özetinde |
| Ayarların saklanması | Sıra ve uzunluk saklanmaz | Sıra ve uzunluk saklanır |
| Sayaç arka planda | Belirtilmemiş | Duraklar |
| Test hattı | Vitest + Playwright (3 motor) + axe | Vitest + elle cihaz testleri |
| Bileşen ve soru ID biçimi | T-01, B-01, R-01, S-01 | T1, K1, R1, S-1 |
| Doküman dışı not | Yok | Yok |

| Kriter | Puan | Gerekçe |
|---|---|---|
| T-1 Yapı | 4 | Bölümler aynı; ID biçimleri farklı. |
| T-2 Kapsam | 5 | 27/27 aynı gereksinimler, aynı kapsam dışı kararları. |
| T-3 Temel kararlar | 5 | Platform, dil, çatı, depolama ve algoritmalar aynı. Farklar yalnızca BA'ya sorulan belirsiz noktaların geçici çözümlerinde. |

### 4.2 Duplicate finder

| Boyut | run1 | run2 |
|---|---|---|
| Bölüm yapısı | 12 bölüm + Ek A Kaynaklar + Ek B Teknik varsayımlar | 12 bölüm + Ek Kaynaklar |
| Ana teknoloji | Flutter, `photo_manager`, `sqflite`, `crypto`, `image`, uzun ömürlü isolate'ler | Aynı |
| Algısal özet | dHash + pHash (birlikte) | Aynı |
| BK-ağacı anahtarı | dHash | pHash |
| Eşikler (Sıkı/Normal/Gevşek) | 4 / 8 / 12 | 5 / 8 / 10 |
| Birebir kopya | Dosya boyutu → SHA-256 → bayt doğrulama | Piksel boyutu + pHash adayı → SHA-256 |
| Gruplama | Union-find | Yıldız (lider) kümeleme |
| Embedding | v1'de yok | v1'de yok |
| Android en düşük sürüm | API 30 önerisi | Karar yok; ≤10'da silme kapalı |
| Video izni (Android 13) | Gözden kaçmış | Risk olarak yazılmış |
| Soru ID biçimi | G-01… | Q-01… |
| Doküman dışı not | **Var** (kayıt adı ve kaynak sayısı notu) | Yok |

| Kriter | Puan | Gerekçe |
|---|---|---|
| T-1 Yapı | 4 | Bölümler aynı; ek bölümler ve ID biçimleri farklı; run1'de doküman dışı not var. |
| T-2 Kapsam | 5 | 27/27 aynı gereksinimler; ikisi de embedding, video ve bulut taramasını dışarıda bırakmış. |
| T-3 Temel kararlar | 4 | Çatı, paketler, özet türleri ve silme yaklaşımı aynı. Ancak **birebir tespit stratejisi** ve **gruplama algoritması** iki çalıştırmada farklı; bunlar doğruluğu doğrudan etkileyen kararlar. Eşikler de farklı. |

---

## 5. Gözlenen sorunlar

### A. Agent ihlali (mevcut bir kuralı veya talimatı çiğneme)

| # | Sorun | Çalıştırma | İhlal edilen kural |
|---|---|---|---|
| TA-A1 | Fiziksel klavye girişi, BA'da olmayan bir davranış olarak eklenmiş ve BA'ya sorulmamış. | çarpım run1, run2 | `scope-rules`: "TA, BA dokümanında olmayan gereksinim eklemez." **Belirsiz:** platform kararının doğal uzantısı sayılabilir. |
| TA-A2 | Kaynakta açıkça yazan bir platform kısıtı gözden kaçırılmış: Android 13'te video izni de gerekli. Tasarım (P-01) buna aykırı. | duplicate run1 | Doğrudan bir kural ihlali değil. Ancak `research-rules` gereği okunan kaynağın içeriği doğru aktarılmalı; aynı sayfa (pub.dev photo_manager) kaynak olarak gösterilmiş. **Belirsiz:** dikkat hatası. |

### B. Persona kaynaklı (persona'daki eksiklikten gelen)

| # | Sorun | Çalıştırma | Persona'da eksik olan |
|---|---|---|---|
| TA-B1 | Doküman dışına ana oturuma not eklenmiş (kayıt adı tahmini, kaynak sayısı). BA'daki B-1 sorunuyla aynı. | duplicate run1 | Yanıtın yalnızca dokümandan oluşacağı kuralı. |
| TA-B2 | Şablonda olmayan ek bölümler: "Kaynaklar" (4/4) ve "Ek B: Teknik varsayımlar" (1/4). Kaynaklar `research-rules` gereği zorunlu ama TA şablonunda yer almıyor; adı ve yeri çalıştırmaya göre değişiyor. | Tümü | TA şablonunda "Kaynaklar" ve "Teknik varsayımlar" bölümleri tanımlı değil. |
| TA-B3 | Kimlik biçimleri tutarsız: bileşen (B-01 / K1 / ad), karar (T-01 / T1), risk (R-01 / R1), geri soru (S-01 / S-1 / G-01 / Q-01). | Tümü | `documentation-standards` yalnızca FR/NFR/US'yi tanımlıyor. TA'ya özgü ID önekleri ve BA'nın S-xx'iyle çakışmayan bir geri soru öneki tanımlı değil. |
| TA-B4 | Doğruluğu doğrudan etkileyen algoritma kararları (birebir stratejisi, gruplama) aynı girdide çalıştırmadan çalıştırmaya değişiyor. | duplicate | Persona, kritik kararlarda alternatifleri karşılaştırıp **seçim ölçütlerini** açıkça yazmayı istemiyor. Karar kendiliğinden veriliyor; hangi ölçütün ağır bastığı tekrarlanabilir değil. |
| TA-B5 | Platform kararı BA'da açık soru olduğu hâlde TA "geçici" bir platform seçip bütün mimariyi onun üzerine kuruyor. Doğru işaretlenmiş, ama yanıt farklı gelirse doküman büyük ölçüde geçersiz olur. | çarpım run1, run2 | Persona, "mimariyi belirleyen" bir BA açık sorusu varken ne yapılacağını tanımlamıyor (ör. iki seçenek için kısa karşılaştırma ve önerilen varsayılan, ya da durup sorma). |
| TA-B6 | Test stratejisinin büyüklüğü ürün ölçeğine göre ayarlanmıyor: çarpım run1'de 3 motorlu E2E ve otomatik erişilebilirlik hattı var. | çarpım run1 | Kapsam kuralı mimari için yazılmış; test stratejisi ve araçlar için açıkça uygulanmıyor. |
| TA-B7 | Kullanılan kütüphanenin belgesindeki platform kısıtlarının (izin, sürüm, dosya boyutu erişimi) gözden kaçabilmesi (TA-A2). | duplicate run1 | Seçilen her eklenti veya kütüphane için "izinler, en düşük sürüm, bilinen kısıtlar" kontrol listesi yok. |

### C. Olumlu gözlemler

- **İzlenebilirlik:** 4/4 çalıştırmada eksiksiz (27/27); hiçbir uydurma ID yok.
- **Belirsizlik yönetimi:** Her belirsizlik "geçici" işaretiyle tasarıma konup BA'ya soruluyor. Bu, scope kuralının iyi uygulandığını gösteriyor.
- **Kaynak kullanımı:** Sürüm numaraları, API adları ve platform davranışları kaynaklı. Doğrulanamayan noktalar açıkça belirtilmiş ("Apple kaynağıyla doğrulanamadı", "MDN muafiyeti teyit etmiyor").
- **BA hatalarını düzeltme:** Duplicate run1 ve run2, BA'nın doğrulayamadığı V-05 varsayımını (iOS onay diyaloğu) Apple kaynağıyla doğrulamış ve BA'ya geri bildirmiş.
- **Flutter kısıtı:** İki çalıştırmada da tam uyum.

---

## Genel not ve gözlemler

- **Güçlü yönler:**
  - TA v1 izlenebilirlik, gerekçelendirme ve kapsam disiplininde çok güçlü.
  - Ürün ölçeğine uygun sade mimariler seçiliyor (çarpım: sunucusuz web; duplicate: tek Flutter uygulaması + SQLite).
  - BA dokümanındaki çelişkileri ve test edilemez kabul kriterlerini bulup geri soruyor.
- **Zayıf yönler:**
  - Doküman dışı not (1/4).
  - Şablon dışı ve adı değişen ek bölümler.
  - ID biçimi tutarsızlığı.
  - Duplicate'te kritik algoritma kararlarının çalıştırmalar arasında değişmesi.
  - Bir çalıştırmada kütüphane belgesindeki izin kısıtının gözden kaçması.
- **Persona'da önerilen değişiklikler:** `eval/aşama-3-yapilacaklar.md` dosyasında listelendi (henüz uygulanmadı).
