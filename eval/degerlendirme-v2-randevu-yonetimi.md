# Değerlendirme: Randevu yönetimi senaryosu (BA v2 / TA v2)

- **Tarih:** 2026-11-24
- **Senaryo:** `randevu-yonetimi` (Flutter + Firebase, güzellik salonu ve kuaför randevu uygulaması)
- **Persona sürümleri:** BA v2 / TA v2 (persona ve rules dosyaları değiştirilmedi)
- **Ölçüt:** `eval/kontrol-listesi.md` (BA-1 … BA-6, TA-1 … TA-5, T-1 … T-3; 1-5 puan)
- **Değerlendirilen dosyalar:**
  - `outputs/randevu-yonetimi/ba-randevu-yonetimi-v2-run{1,2,3}.md`
  - `outputs/randevu-yonetimi/ta-randevu-yonetimi-v2-run{1,2,3}.md`

## 0. Test yöntemi ve önemli fark

Bu senaryo, Aşama 2 ve 3'teki testlerden farklı yürütüldü. Puanlar bu farklar göz önünde bulundurularak okunmalıdır.

- **Çalıştırmalar bağımsız tekrar değil, ardışık iterasyondur.**
  - run1 yalnızca görev metniyle çalıştırıldı.
  - BA run2 ve run3'e ek girdi verildi: önceki BA dokümanı, TA'nın geri soruları ve paydaşın yanıtları.
  - Paydaş yanıtları: run2 için 5, run3 için 4 + 3 = 7.
  - TA her turda ilgili BA dokümanının dosya yoluyla çalıştırıldı.
  - Bu nedenle C bölümündeki tutarlılık kriterleri yalnızca **girdideki değişiklikle açıklanamayan farklara** bakılarak puanlandı.
- **Çağrı yöntemi:** Aşama 2'deki gibi ayrı `claude -p` oturumları kullanılmadı. Ajanlar ana oturumdan Agent aracıyla çağrıldı.
  - Subagent'a giden ilk mesaj oturum kaydından kontrol edildi.
  - run1'de BA'ya görev metni birebir gitti.
  - TA'lara yalnızca `BA dokümanı: <yol>` metni gitti.
- **Çıktıların kaydı:** Çıktılar subagent kaydındaki dönüş mesajından program ile alındı ve değiştirilmeden yeni dosyaya yazıldı. 6/6 dosya `# ...` başlığıyla başlıyor ve son bölümle bitiyor; doküman dışı not yok.
- **Süreç olayı (BA run3):** Paydaşın son 3 yanıtı, BA çalışmasını bitirdikten hemen sonra ona ulaştı.
  - İlk dönüş bu yanıtları içermiyordu, bu yüzden kaydedilmedi.
  - Ajan yeniden çalışınca yanıtları işleyerek ikinci bir dönüş üretti. Kaydedilen run3 bu ikinci dönüştür.
  - Ek talimat, run3'ün `Girdi` bloğunda "Ek talimat (aynı tur)" olarak yer alıyor.
- **Ana oturum hatası:** BA run1'in ilk kayıt denemesinde betik hatası boş bir dosya oluşturdu. Dosyanın 0 bayt ve izlenmeyen bir dosya olduğu kontrol edildi, ardından silinip yeniden yazıldı. Var olan bir çıktının üzerine yazılmadı.

| Çalıştırma | Read | WebSearch | WebFetch | Kelime | Kural dosyalarını okudu mu? |
|---|---|---|---|---|---|
| BA run1 | 4 | 5 | 29 | ~5.400 | Evet (3/3) |
| BA run2 | 5 | 1 | 20 | ~9.000 | Kısmen (yalnızca `documentation-standards.md`) |
| BA run3 | 5 | 3 | 22 | ~11.700 | Hayır (yalnızca önceki BA/TA dokümanları ve bir PDF) |
| TA run1 | 4 | 0 | 39 | ~6.700 | Evet (3/3) |
| TA run2 | 5 | 0 | 39 | ~7.700 | Evet (3/3) |
| TA run3 | 5 | 0 | 32 | ~8.400 | Evet (3/3) |

---

## 1. Puan özeti

### A. Business Analyst (/30)

| Kriter | run1 | run2 | run3 |
|---|---|---|---|
| BA-1 Araştırma | 5 | 5 | 5 |
| BA-2 Kaynaklar gerçek ve erişilebilir | 4 | 5 | 5 |
| BA-3 Gereksinimler ölçülebilir | 5 | 5 | 4 |
| BA-4 Kapsam uygun | 4 | 4 | 4 |
| BA-5 Şablon | 5 | 5 | 5 |
| BA-6 Açık sorular anlamlı | 5 | 5 | 5 |
| **Toplam** | **28** | **29** | **28** |

### B. Technical Architect (/25)

| Kriter | run1 | run2 | run3 |
|---|---|---|---|
| TA-1 Her BA gereksinimi karşılanmış | 5 | 5 | 5 |
| TA-2 Uydurma gereksinim yok (5 = hiç yok) | 4 | 5 | 5 |
| TA-3 Kararlar gerekçeli | 5 | 5 | 5 |
| TA-4 Mimari ürün büyüklüğüne uygun | 5 | 5 | 4 |
| TA-5 İzlenebilirlik tablosu eksiksiz | 5 | 5 | 5 |
| **Toplam** | **24** | **25** | **24** |

### C. Tutarlılık (/15)

| Kriter | Puan | Not |
|---|---|---|
| T-1 Yapı benzer mi? | 5 | BA'da 10 bölüm ve TA'da 12 bölüm aynı sırada. BA run2 ve run3'teki 11. bölüm ("Değişiklik notu") ana oturumun talebiyle eklendi. |
| T-2 Kapsam benzer mi? | 5 | FR 28 → 29 → 29, NFR 13 → 14 → 15, US 11 → 12 → 12. Değişikliklerin tamamı paydaş yanıtlarına bağlanmış; ID'ler korunmuş, yeniden numaralandırma yok. |
| T-3 Temel kararlar benzer mi? (TA) | 4 | Aynı kalan kararlar: yığın, sunucu tarafı randevu yazma, çalışan-gün doluluk dokümanı ve dakikalık zamanlanmış hatırlatma. Ancak run3'te müsait saat hesabı sunucudan istemciye taşındı (gerekçe NFR-03). Girdideki değişiklikle açıklanmıyor. |
| **Toplam** | **14** | |

---

## 2. Kontroller

### 2.1 Biçim ve ID'ler (program ile)
- 6/6 doküman başlık bloğu (tarih, persona sürümü, girdi) ile başlıyor ve Türkçe.
- BA'da 10 şablon bölümü, TA'da 12 şablon bölümü doğru sırada.
- FR, NFR ve US ID'leri her dokümanda 01'den başlayarak boşluksuz ilerliyor.
- BA run2 ve run3'te yeni maddeler sıradaki ID'yi aldı: FR-29, NFR-14, NFR-15, US-12.

### 2.2 İzlenebilirlik (program ile + elle örneklem)
| TA | BA'da tanımlı FR/NFR | TA 11. bölümde satır | Eksik | BA'da olmayan ID |
|---|---|---|---|---|
| run1 | 41 | 41 | 0 | 0 |
| run2 | 43 | 43 | 0 | 0 |
| run3 | 44 | 44 | 0 | 0 |

- TA'nın belirsiz bulduğu noktalar "geçici" işaretiyle "BA'ya geri sorular" bölümüne yazıldı. Bölümdeki soru sayısı run1'den run3'e 13, 16 ve 9 (+8 bekleyen).
- Örnek: TA run2, BA'da tanımı olmayan "salonun randevuyu değiştirmesi" bildirimini fark etti (S-04). Bunu gereksinime çevirmedi, BA'ya sordu. BA run3 bunu AS-20 olarak açık soruya taşıdı.

### 2.3 Kaynaklar
- **Açılıp okunmuş mu (oturum kaydından):** Her dokümandaki kaynak URL'lerinin tamamı aynı çalıştırmada WebFetch ile açılmış. Açılmadan kaynak gösterilen link sayısı 0.
  - BA: 17, 19 ve 22 link.
  - TA: 32, 29 ve 29 link.
- **Bugün erişilebilir mi (`curl`):** Altı dokümandaki 71 benzersiz URL'nin 69'u HTTP 200 döndü. Kalan 2 URL GitHub'daki Luxon sayfalarıdır (TA). Bunlar bu ortamın proxy'sinde 403 verdi; bu, sayfanın kendisinden değil ortamdan kaynaklanıyor.
- **İçerik örneklemi:** Karar etkileyen 15 iddia, sayfa metni `curl` ile alınarak kontrol edildi.

| # | Doküman | İddia | Sonuç |
|---|---|---|---|
| 1 | BA run1 | Fresha: tek kişi 19,95 $, ekip kişi başı 14,95 $, 100 ücretsiz mesaj, "sonrası mesaj başına 0,02 $", pazar yeri %20 (en az 6 $) | **Kısmen.** SMS 0,02 $ doğru, ancak WhatsApp 0,04–0,11 $. BA run1 "SMS/WhatsApp" için tek fiyat verdi. run3 yalnızca SMS için 0,02 $ yazarak düzeltti. |
| 2 | BA run1-3 | Booksy 29,99 $ + ek kullanıcı başı 20 $ | Doğru |
| 3 | BA run1-3 | Apple 5.1.1(v): "you must also offer account deletion within the app" | Doğru |
| 4 | BA run2-3 | Apple 4.2.6: şablon uygulamalar içerik sağlayıcısınca gönderilmeli | Doğru |
| 5 | BA run2-3 | Apple hesap silme rehberi: yalnızca devre dışı bırakma "insufficient" | Doğru |
| 6 | BA run3 | Google Play: D-U-N-S alımı "can take up to 30 days" | Doğru |
| 7 | BA run3 | 13 Kasım 2023 sonrası bireysel hesaplarda 12 test kullanıcısı, 14 gün | Doğru |
| 8 | BA run1-3 | Meta-analiz: 8 RKÇ, OR 1,48 (GA 1,23–1,72) | Doğru (PMC OAI kaydından) |
| 9 | BA run1 | Kuaförüm Yanımda 4,6 puan, 75 değerlendirme | Doğru |
| 10 | TA run1-3 | Firestore konumu sonradan değiştirilemez | Doğru |
| 11 | TA run2 | Cloud Tasks en fazla 30 gün ileriye planlar | Doğru |
| 12 | TA run1, 3 | Standard sürümde varsayılan mod karamsar (pessimistic), sunucu işlemi okunan dokümanı kilitler | Doğru |
| 13 | TA run1-3 | FlutterFire: iOS 15, API 24 | Doğru |
| 14 | TA run1 | Cloud Scheduler: hesap başına 3 iş ücretsiz, iş başı 0,10 $/ay | Doğru |
| 15 | TA run1 | `firebase_messaging` 17.1.0, `intl` 0.20.3, "Time zones are not yet supported." | Doğru |

Sonuç: 15 iddianın 14'ü doğru, 1'i kısmen doğru. Uydurma kaynak yok.

---

## 3. Gerekçeler

### BA
- **BA-1 (5/5/5):**
  - Rakip incelemesi 6 çözümü kapsıyor: 3 uluslararası rakip, Google Takvim randevu planları ve 2 Türkiye uygulaması.
  - Araştırma mağaza politikaları (Apple, Google Play), KVKK, Ticari Elektronik İletiler Yönetmeliği, WCAG ve bir meta-analizi de içeriyor.
  - run2'de hesap silme çatışması için, run3'te mağaza yayıncı hesabı için (D-U-N-S, test kullanıcısı şartı) hedefli yeni araştırma yapıldı.
- **BA-2 (4/5/5):** Linklerin tamamı açılmış. run1'deki Fresha mesaj fiyatı nüans kaybı (2.3 #1) nedeniyle 1 puan düşüldü.
- **BA-3 (5/5/4):**
  - NFR'lerin çoğunda sayısal eşik var (2 sn p95, ±5 dk, %99,5, 30 gün, 50 eşzamanlı deneme). Kabul kriterleri test edilebilir.
  - run3'te NFR-15 (mağaza yayını, Firebase ve fatura sahipliği) kalite özelliği değil, bir iş ve organizasyon kısıtı. NFR olarak yazılması ve teknoloji adı (Firebase) içermesi nedeniyle 1 puan düşüldü. Firebase görev kısıtı olduğundan bu bir kapsam ihlali sayılmadı.
- **BA-4 (4/4/4):**
  - Ödeme, pazar yeri ve muhasebe doğru biçimde kapsam dışı bırakıldı.
  - Ancak görevde istenmeyen bazı C maddeleri (bekleme listesi FR-26, takvime ekleme FR-28) gereksinim listesine eklendi. FR-28 dokümanda "kaynaksız BA önerisi" diye işaretli. Kapsam kuralına göre bunlar "Kapsam dışı" veya "Açık sorular" bölümünde anılmalıydı.
- **BA-5 (5/5/5):** Şablona uyuldu. run2 ve run3'teki 11. bölüm ana oturumun talimatıyla eklendi, persona hatası sayılmadı.
- **BA-6 (5/5/5):**
  - Açık sorular kararları gerçekten etkiliyor.
  - run2, paydaşın "hard delete olmamalı" yanıtının KVKK ve mağaza kurallarıyla çeliştiğini açıkça gösterdi. Çatışmayı A/B/C seçenekleriyle karar sorusuna dönüştürdü; bu, paydaşın Seçenek A'yı seçmesini sağladı.
  - Cevaplanan sorular listeden çıkarıldı, kısmen cevaplananlar daraltıldı.

### TA
- **TA-1, TA-2, TA-5:** 3/3 çalıştırmada eksiksiz. Uydurma FR/NFR yok.
- **TA-2 düzeltmesi (run1: 5 → 4, sonradan eklendi):** BA run1'de NFR-11 ile US-07 KK-1 ("varsa nedenle birlikte bildirim") çelişiyordu. TA run1 bu çelişkiyi S-08 ile BA'ya sordu. Ancak tasarımı "geçici" işareti koymadan kendi tercihine göre yaptı: "iptal nedeni bildirimde yok" (8.4 ve izlenebilirlik satırı NFR-11). Bu, kapsam kuralındaki "kendi varsayımıyla gereksinime dönüştürmez" ilkesine kısmen aykırıdır. ID kontrolü bu tür sapmayı yakalamıyor; kabul kriteri düzeyinde ayrıca bakılmalı. Tanımsız değerler (ızgara adımı, hatırlatma sayısı, atama kuralı) "geçici" etiketiyle soruya bağlandı.
- **TA-3 (5/5/5):**
  - Kararlar kaynaklı ve alternatifleriyle karşılaştırılmış. Örnekler:
    - Firestore ve Realtime Database karşılaştırması.
    - Dart ve TypeScript Cloud Functions arasındaki seçim.
    - Cloud Tasks ile zamanlanmış fonksiyon arasındaki seçim (30 gün sınırı).
    - `flutter_local_notifications` paketinin bilinçli olarak kullanılmaması.
  - Kütüphane izin, sürüm ve kısıt tablosu (persona v2 kuralı TA-2) 3/3 çalıştırmada var.
- **TA-4 (5/5/4):**
  - Yalnızca yönetilen Firebase servisleri ve tek bir zamanlanmış iş kullanılıyor. Ayrı sunucu yok; Analytics ve Crashlytics MVP'ye eklenmedi.
  - run3'te slot algoritması hem istemcide (Dart) hem sunucuda (TypeScript) iki ayrı uygulama olarak yazılıyor. Ortak test vektörleriyle güvenceye alınmış olsa da küçük ürün için ek bakım yükü getiriyor, bu nedenle 1 puan düşüldü.
  - Bu tasarımda `staffDays` dokümanı girişli tüm kullanıcılara okunur ve yalnızca dakika aralıkları içerir; kişisel veri yok.

---

## 4. Genel not ve gözlemler

- **Güçlü yönler:**
  - İzlenebilirlik 3/3 eksiksiz. Açılmadan kaynak gösterilen link yok. Doküman dışı not 0/6.
  - BA, paydaş yanıtı ile mevzuat arasındaki çatışmayı gizlemedi, karar sorusuna dönüştürdü.
  - BA ve TA arasındaki soru-cevap döngüsü çalışıyor: TA'nın soruları bir sonraki BA turunda ya düzeltildi ya da açık soruya taşındı.
- **Zayıf yönler / gözlenen sorunlar:**
  1. **Doküman uzunluğu:** BA 5,4 bin → 9,0 bin → 11,7 bin kelime. Güncelleme turları dokümanı tek başına okunur tutarken sürekli büyütüyor. Bu sorun Aşama 3'ten beri açık.
  2. **TA karar kararlılığı:** Müsait saat hesabının yeri girdide bir değişiklik olmadan değişti. Aşama 3'teki "TA karar değerlerinin tutarlılığı" sorunu bu senaryoda da görüldü.
  3. **BA güncelleme turlarında kural dosyaları okunmadı:** run2'de 1/3, run3'te 0/3 kural dosyası okundu. Çıktıda bir kural ihlali görülmedi, ama "Başlamadan önce" adımı atlanıyor.
  4. **Kapsam sınırı:** Görevde istenmeyen C maddeleri gereksinim olarak listeleniyor.
  5. **Nüans kaybı:** Fresha mesaj fiyatında SMS ile WhatsApp birleştirildi (run1).
- **Persona'da önerilen değişiklik:** Bu değerlendirmede persona değiştirilmedi ve `degisiklik-gunlugu.md` dosyasına yazılmadı. Aday öneriler:
  - BA'ya "güncelleme modu" kuralı: önceki doküman ve paydaş yanıtı verildiğinde de kural dosyaları okunur. Değişiklik notu standart bölüm olur. Uzunluk için hedef konur.
  - BA'da görevde istenmeyen fikirlerin C önceliğiyle bile FR'ye dönüştürülmemesi.
  - TA'da aynı BA içeriğiyle değişmemesi gereken kararların (hesaplamanın yeri gibi) gerekçe ve ölçütle sabitlenmesi.
