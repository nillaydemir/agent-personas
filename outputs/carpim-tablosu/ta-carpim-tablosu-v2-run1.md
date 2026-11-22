# Çarpım Tablosu Öğrenme Uygulaması: Teknik Gereksinim Dokümanı

- **Tarih:** 2026-11-22
- **Persona sürümü:** ta v2
- **Girdi:** /home/user/agent-personas/outputs/carpim-tablosu/ba-carpim-tablosu-v2-run1.md

---

## 1. Özet

Bu doküman, BA analiz dokümanındaki 13 fonksiyonel (FR-01–FR-13) ve 11 fonksiyonel olmayan (NFR-01–NFR-11) gereksinimi karşılayan teknik çözümü tanımlar.

Ürün küçük, tek kullanıcılı ve çevrimdışı çalışan bir öğrenme uygulamasıdır. Sunucu, hesap, ağ bağlantısı ve üçüncü taraf servis yoktur. Önerilen çözüm şudur:

- **Platform:** Tek kod tabanlı bir **Flutter** mobil uygulaması (iOS ve Android). Hedef platform BA'da açık bir soru (AS-02). Bu nedenle seçim **geçici** olarak işaretlenmiştir. Gerekçesi, NFR-10 (kalıcılık), NFR-02 (veri toplamama) ve BA'nın Bölüm 3.3'te incelediği iki mağazadır (Bölüm 3.1).
- **Veri saklama:** Cihazdaki SQLite veritabanı (`sqflite`). Olgu başına özet satırı tutulur. Her cevap ayrı bir işlemle (transaction) yazılır (NFR-10, FR-06).
- **Soru seçimi:** 3 kutulu, Leitner benzeri ve **deterministik kotalı** bir seçim algoritması. Kutu, olgunun son cevaplardaki doğru serisinden türetilir. Böylece "öğrenildi" tanımı (V-04) ile kutu mantığı aynı veriden hesaplanır (FR-07, FR-09).
- **Gizlilik:** Android sürüm derlemesinde INTERNET izni yoktur. Analitik, reklam ve satın alma SDK'sı da yoktur (NFR-02, NFR-03, NFR-08).
- **Erişilebilirlik:** Sistem metin ölçeğine uyum, en az 48×48 mantıksal piksel dokunma hedefleri ve otomatik kontrast ile dokunma hedefi testleri (NFR-04, NFR-05, NFR-06).

Belirsiz noktalar (zamanlayıcı seçenekleri, süre dolduğunda kayıt türü, 200% büyütmede "tek ekran" şartı, ×0 olgularının ilerleme tablosundaki yeri vb.) geçici kararlarla ilerletilmiş ve Bölüm 12'de BA'ya sorulmuştur.

## 2. Mimari genel bakış

Tek süreçte çalışan, katmanlı bir istemci uygulamasıdır. Arka uç (backend) yoktur.

```
┌──────────────────────────────────────────────────────────┐
│  Sunum katmanı (Flutter widget'ları, Türkçe metinler)     │
│  Ana ekran/Tablo seçimi · İnceleme · Pratik · Özet ·      │
│  İlerleme · Ayarlar · Ebeveyn kapısı · Sayısal tuş takımı │
└───────────────┬──────────────────────────────────────────┘
                │ (ChangeNotifier tabanlı durum nesneleri)
┌───────────────▼──────────────────────────────────────────┐
│  Alan (domain) katmanı, saf Dart, platformdan bağımsız    │
│  SoruSecici · OturumYoneticisi · OlguDurumHesaplayici ·   │
│  Zamanlayici · EbeveynKapisiUretici                       │
└───────────────┬──────────────────────────────────────────┘
                │ (depo arayüzleri)
┌───────────────▼──────────────────────────────────────────┐
│  Veri katmanı: OlguDeposu, AyarDeposu → SQLite (sqflite)  │
│  Cihaz içi tek dosya; ağ erişimi yok                      │
└──────────────────────────────────────────────────────────┘
```

**İlkeler ve bağlandıkları gereksinimler:**
- Tüm işlevler cihazda çalışır. Ağ katmanı yoktur (NFR-02, NFR-03, NFR-08).
- Alan katmanı saf Dart'tır ve rastgele sayı üreteci dışarıdan verilir. Seçim algoritması bu sayede birim testlerle deterministik olarak doğrulanabilir (FR-07, US-03).
- Her cevap, geri bildirim gösterilirken veritabanına yazılır. Geri bildirim yazmayı beklemez (NFR-09, NFR-10).
- Durum yönetimi için ek paket kullanılmaz. Flutter'ın yerleşik `ChangeNotifier` yapısı yeterlidir. Bağımlılık sayısını en aza indirmek NFR-02 ve NFR-03 denetimini kolaylaştırır (Bölüm 8).

## 3. Teknoloji seçimleri ve gerekçeleri

### 3.1 Kritik karar: Uygulama türü ve platform (geçici, AS-02'ye bağlı)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı gereksinim / risk |
|---|---|---|
| Ö1 | Cihazda tutulan ilerleme, uygulama ve cihaz yeniden başlatıldığında kaybolmamalı. Platform veriyi kendiliğinden silmemeli. | NFR-10, FR-06 |
| Ö2 | Uygulama bağlantısız tam çalışmalı. | NFR-08 |
| Ö3 | Ağ erişimi platform düzeyinde kapatılabilmeli; "veri toplanmıyor" beyanı teknik olarak desteklenmeli. | NFR-02, NFR-03 |
| Ö4 | BA'nın incelediği iki mağazaya (Google Play Aile, Apple Kids) dağıtılabilmeli. | BA Bölüm 3.3, AS-02, FR-11 |
| Ö5 | Ürün büyüklüğüne uygun sadelik: tek kod tabanı, az bağımlılık. | Kapsam kuralı |
| Ö6 | Sistem metin ölçeğine uyum. | NFR-06 |

**Karşılaştırma:**

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 | Ö6 |
|---|---|---|---|---|---|---|
| **A. Flutter (iOS + Android, tek kod tabanı)** | İyi. Veri uygulamanın kendi dosya alanındadır ve SQLite işlemleriyle yazılır. | İyi | İyi. Android şablonu INTERNET iznini içermez [S5]. | İyi, iki mağaza | İyi, tek kod tabanı | İyi. `MediaQuery.textScalerOf` platform ayarını izler [S8]. |
| B. Web uygulaması (PWA) | Zayıf. Tarayıcı depolaması "best effort"tur ve tarayıcı veriyi "at its discretion" silebilir. Safari, ana ekrana eklenmemiş sitelerde betikle yazılabilen depolamaya 7 günlük sınır uygular [S4]. | Service worker ile iyi | Orta. Ağ erişimi platform düzeyinde kapatılamaz. | Zayıf, mağaza dağıtımı yok | Çok iyi | İyi (tarayıcı yakınlaştırması) |
| C. Ayrı yerel uygulamalar (Swift + Kotlin) | İyi | İyi | İyi | İyi | Zayıf. İki kod tabanı, iki kat test. | İyi |

**Karar:** A (Flutter). B, Ö1'de (NFR-10'un "%0 kayıp" ölçütü) ve Ö4'te zayıf kalıyor. C, A ile aynı sonucu iki kat maliyetle veriyor. Seçim AS-02'nin cevabına bağlı ve **geçicidir**. Yalnızca web istenirse bu karar ve Bölüm 3.2 yeniden yapılır (bkz. Riskler R-01).

### 3.2 Kritik karar: Kalıcı veri saklama

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı gereksinim / risk |
|---|---|---|
| Ö1 | Yazma kalıcılığı garantisi: dönen yazma diske ulaşmış olmalı. | NFR-10, FR-06 |
| Ö2 | Tek adımda bütün ilerlemeyi silebilme (silme yolu). | FR-11, US-07 |
| Ö3 | Ek izin istememe, ağ erişimi olmama | NFR-02 |
| Ö4 | Geri bildirimi bekletmeme (yazma işi arayüz iş parçacığını engellemez). | NFR-09 |
| Ö5 | Sadelik | Kapsam kuralı |

**Karşılaştırma:**

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| **A. SQLite (`sqflite`)** | İyi. İşlem (transaction) tabanlı. | İyi. Tek işlemde `DELETE`. | İyi. Belgede izin şartı yok [S2]. | İyi: "SQL runs in a background thread … never blocking the UI" [S2] | Orta. Küçük şema gerekir. |
| B. `shared_preferences` | **Yetersiz.** Belgeye göre "there is no guarantee that writes will be persisted to disk after returning", kritik veri için önerilmiyor [S3]. | İyi | İyi | İyi | Çok iyi |
| C. JSON dosyası (geçici dosyaya yaz ve yeniden adlandır) | Orta. Atomiklik bizim kodumuza ve dosya sistemine bağlı, doğrulanmadı. | İyi | `path_provider` eklentisi gerekir | İyi | İyi |

**Karar:** A (`sqflite`). B, Ö1'de belgenin kendi uyarısıyla eleniyor. C'de atomiklik garantisi kanıtlanmış bir bileşene değil, bizim uygulamamıza dayanıyor. Ayarlar da aynı veritabanında tutulur, böylece `shared_preferences` bağımlılığı eklenmez.

### 3.3 Teknoloji seçimleri tablosu

| Teknoloji | Amaç | Gereksinim | (a) İzinler | (b) En düşük platform | (c) Belgedeki bilinen kısıtlar | Kaynak |
|---|---|---|---|---|---|---|
| **Flutter SDK (Dart)** | Tek kod tabanlı iOS ve Android uygulaması | NFR-08, NFR-10, NFR-06, AS-02 (geçici) | Android şablonu INTERNET iznini içermez. Geliştirme sırasında Flutter araçlarıyla iletişim için internet erişimi sağlanır, uygulamanın ihtiyacı varsa izin elle eklenir [S5]. | Android API 24 (desteklenen aralık 24–37). iOS 15 (desteklenen aralık 15–27) [S1]. | Android API 23 ve altı ile iOS 14 ve altı desteklenmez [S1]. | [S1], [S5] |
| **`flutter_localizations`** (SDK ile gelir) | Material bileşenlerinin yerleşik metinlerini (geri butonu etiketi, diyalog butonları vb.) Türkçeleştirmek | NFR-01 | Belgede izin şartı yok. | Flutter SDK ile aynı | Paket eklenmezse "Flutter only provides US English localizations" [S9]. | [S9] |
| **`sqflite`** (belgedeki güncel sürüm 2.4.6) | Olgu özeti ve ayarların kalıcı saklanması | FR-06, FR-11, NFR-10, NFR-09 | Belgede izin şartı belirtilmiyor [S2]. | Android: eklenti `minSdk` 19, ancak Flutter'ın en düşük sürümü geçerlidir, yani etkin değer API 24. iOS 12, Flutter nedeniyle etkin değer iOS 15 [S1][S2]. | Yalnızca Android, iOS ve macOS. Web desteği ayrı paketle "experimental". Eşzamanlı okuma yok, işlem süresince diğer çağrılar bekler. İşlem içinde `db` yerine `txn` kullanılmazsa kilitlenme olur [S2]. | [S2] |
| **`sqflite_common_ffi`** (yalnızca test bağımlılığı) | Veri katmanının masaüstü ve CI ortamında birim testi | NFR-10, FR-11 (test) | Uygulamaya girmez. | Dart VM | Belgede "Unit tests and Dart VM" için önerilir [S2]. | [S2] |
| **`flutter_test`** (SDK ile gelir) | Widget ve erişilebilirlik testleri (`meetsGuideline`) | NFR-04, NFR-05 | Uygulamaya girmez. | — | Belgede kılavuz eşik değerleri yazmıyor (bkz. R-07) [S7]. | [S7] |

Durum yönetimi, analitik, reklam, çökme raporlama veya ağ paketi **eklenmez** (NFR-02, NFR-03, Ö5).

## 4. Bileşenler ve sorumlulukları

| Bileşen | Katman | Sorumluluk | Gereksinim |
|---|---|---|---|
| **AnaEkran / TabloSecimEkrani** | Sunum | Uygulama bu ekranla açılır. 1–10 tablo düğmeleri çoklu seçim yapar. "2. sınıf" ön ayarı 1–5'i, "3. sınıf" ön ayarı 1–10'u tek dokunuşla seçer. "İncele" ve "Pratik" butonları bu ekrandadır. Son seçim `ayar` tablosunda saklanır ve sonraki açılışta hazır gelir. | FR-01, FR-10, NFR-11 |
| **IncelemeEkrani** | Sunum | Seçili tablonun n×1…n×10 satırları cevaplarıyla listelenir. Birden çok tablo seçiliyse her tablo ayrı sayfada gösterilir ve sayfalar arasında kaydırılarak geçilir (geçici, bkz. BA-S02). | FR-02 |
| **PratikEkrani** | Sunum | `n1 × n2 = ?` sorusu, uygulama içi sayısal tuş takımı (0–9, sil, onayla, en çok 3 hane), doğru/yanlış geri bildirimi ve yanlışta "7 × 8 = 56" gösterimi. Zamanlayıcı açıksa geri sayım ve ekrandan zamanlayıcıyı kapatma düğmesi gösterilir. | FR-03, FR-04, FR-08, NFR-07, NFR-09 |
| **SayisalTusTakimi** | Sunum | Sistem klavyesi yerine uygulama içi tuşlar. Her tuş en az 48×48 mantıksal pikseldir. | FR-03, NFR-05 |
| **OzetEkrani** | Sunum | Toplam soru, doğru sayısı, yanlış cevaplanan olgular ve oturum yıldızı. Mağaza veya satın alma bağlantısı yoktur. | FR-05, FR-13, NFR-03 |
| **IlerlemeEkrani** | Sunum | 10×10 ızgarada her olgunun üç durumu ("henüz çalışılmadı", "çalışılıyor", "öğrenildi"). Durumlar renk yanında simge ve metin etiketiyle de ayrılır, böylece yalnızca renge dayanılmaz ve kontrast şartı korunur. | FR-09, NFR-04 |
| **AyarlarEkrani** | Sunum | Zamanlayıcı aç/kapa ve süre seçimi (6, 10, 15 sn, geçici, bkz. BA-S04), ×0 olguları aç/kapa, "İlerlemeyi sıfırla" (ebeveyn kapısından geçer). | FR-08, FR-12, FR-11, NFR-07 |
| **EbeveynKapisi** | Sunum + Alan | Rastgele iki basamaklı × iki basamaklı çarpma sorusu. Cevap tuş takımıyla girilir. Ardından ayrı bir onay diyaloğu gelir (Bölüm 6.3). | FR-11, US-07 |
| **OturumYoneticisi** | Alan | Oturumu başlatır (`SoruSecici` ile 20 soruluk liste). Cevabı değerlendirir, geri bildirim üretir, sonucu depoya iletir ve özeti hesaplar. | FR-03, FR-04, FR-05 |
| **SoruSecici** | Alan | Kutu durumlarına göre deterministik kotalı seçim yapar ve aynı olgunun art arda gelmediği bir sıralama üretir (Bölüm 6.1). | FR-07, FR-12, US-03 |
| **OlguDurumHesaplayici** | Alan | Doğru serisinden kutuyu (K1, K2, K3) ve ilerleme durumunu türetir (Bölüm 6.2). | FR-07, FR-09, V-04 |
| **Zamanlayici** | Alan | Monoton saatle (`Stopwatch`) soru başına geri sayım yapar. Süre dolunca sonuç `sure_doldu` olur. Uygulama arka plana geçince duraklar (geçici, bkz. BA-S10). | FR-08, NFR-07 |
| **OlguDeposu** | Veri | Cevap sonucunu tek işlemde yazar, tüm olgu özetlerini okur ve tek işlemde siler. | FR-06, FR-11, NFR-10 |
| **AyarDeposu** | Veri | Anahtar-değer ayarlarını okur ve yazar. | FR-08, FR-12, NFR-11 |
| **Metinler (tr)** | Sunum | Tüm ekran metinleri tek bir Türkçe metin dosyasında tutulur. `flutter_localizations` ile `tr` yerel ayarı tek desteklenen yerel ayar olarak verilir. | NFR-01 |

## 5. Veri modeli

### 5.1 Kritik karar: Geçmişin saklanma biçimi

**Seçim ölçütleri:** FR-06 (olgu bazında doğru ve yanlış geçmişi), FR-07 ve V-04 (kutu ile "öğrenildi" hesabı için son cevaplardaki doğru serisi gerekir), NFR-10 (yazmanın tutarlılığı), FR-11 (eksiksiz silme), sadelik.

| Seçenek | FR-06 | FR-07 / V-04 | NFR-10 tutarlılık | FR-11 | Sadelik |
|---|---|---|---|---|---|
| **A. Olgu başına özet satırı** (doğru sayısı, yanlış sayısı, süre doldu sayısı, son doğru serisi) | Sayılar düzeyinde karşılar | Seri doğrudan saklanır | Cevap başına tek satır güncellenir | Tek `DELETE` | İyi. En çok 110 satır. |
| B. Her cevap için ayrı kayıt (olay günlüğü) | Tam geçmiş | Seri her açılışta hesaplanır | Ekleme tek satırdır | Tek `DELETE` | Orta. Tablo zamanla büyür, okuma maliyeti artar. |

**Karar:** A. Hiçbir FR, tek tek cevap kayıtlarını (zaman, verilen cevap) kullanmıyor. V-04'ün "son 3 denemede doğru" şartı, "son doğru serisi ≥ 3" ile birebir aynıdır. Ayrıntılı günlük gerekip gerekmediği BA'ya sorulmuştur (BA-S06). Gerekirse B tablosu şemaya eklenebilir.

### 5.2 Şema (SQLite, sürüm 1)

**Tablo `olgu`**: olgu başına özet. Satır, olgu ilk kez cevaplandığında oluşur. Satırı olmayan olgu "henüz çalışılmadı" sayılır.

| Alan | Tür | Kısıt | Açıklama | Gereksinim |
|---|---|---|---|---|
| `carpan1` | INTEGER | 1–10, birincil anahtarın parçası | Tablo numarası (ör. 7×8'de 7) | FR-01, FR-06 |
| `carpan2` | INTEGER | 0–10, birincil anahtarın parçası | İkinci çarpan (0, yalnızca FR-12 açıkken) | FR-06, FR-12 |
| `dogru_sayisi` | INTEGER | ≥ 0 | Toplam doğru cevap | FR-06 |
| `yanlis_sayisi` | INTEGER | ≥ 0 | Toplam yanlış cevap | FR-06 |
| `sure_doldu_sayisi` | INTEGER | ≥ 0 | Süresi dolan sorular | FR-08, US-04 KK-2 |
| `seri` | INTEGER | ≥ 0 | Son cevaplardaki kesintisiz doğru sayısı. Yanlış veya süre dolunca 0 olur. | FR-07, FR-09, V-04 |

Birincil anahtar `(carpan1, carpan2)` ikilisidir. 7×8 ve 8×7 ayrı olgulardır. Bu ayrım, FR-09'daki 10×10 ızgaradan ve US-01 KK-1'deki "6× olguları" ifadesinden çıkarılmıştır (geçici, bkz. BA-S12).

**Tablo `ayar`**: anahtar-değer.

| Anahtar | Değer | Varsayılan | Gereksinim |
|---|---|---|---|
| `zamanlayici_acik` | 0 veya 1 | 0 | FR-08, US-04 KK-1 |
| `zamanlayici_sn` | 6, 10 veya 15 | 6 | FR-08 (geçici değerler) |
| `sifir_carpimi` | 0 veya 1 | 0 | FR-12 |
| `secili_tablolar` | ör. "6,7" | boş | NFR-11 |

**Bellekte tutulan (kalıcı olmayan) yapılar:** `Oturum` (soru listesi, sıradaki soru, cevap sonuçları) ve `OturumOzeti`. Bunlar FR-05 içindir. Yarıda kesilen oturumun özeti saklanmaz, ancak verilen her cevap `olgu` tablosuna yazılmış olur (BA-S11).

**Yazma kuralı:** Her cevap tek işlemde işlenir: `INSERT OR IGNORE` ile satır oluşturulur, ardından `UPDATE` ile sayaç ve seri güncellenir. Böylece SQLite sürümüne bağlı UPSERT sözdizimine ihtiyaç kalmaz. Belgedeki uyarı gereği işlem içinde yalnızca `txn` nesnesi kullanılır [S2] (NFR-10).

**Silme kuralı (FR-11):** `DELETE FROM olgu` tek işlemde çalışır. `ayar` tablosu korunur (geçici, bkz. BA-S09). Silmeden sonra ilerleme ekranındaki tüm olgular "henüz çalışılmadı" görünür (US-07 KK-2).

**Şema sürümü:** `sqflite`'ın `version` parametresi ile tutulur. İleride BA-S06'nın cevabına göre eklenecek tablo bir göç (migration) adımıyla eklenir.

## 6. Algoritmalar ve seçim gerekçeleri

### 6.1 Kritik karar: Soru seçimi (FR-07)

**Seçim ölçütleri:**

| # | Ölçüt | Bağlı gereksinim |
|---|---|---|
| Ö1 | Yanlış cevaplanan olgu, doğru cevaplanandan daha sık sorulur; bu oturum başına **test edilebilir** olmalıdır. | FR-07, US-03 KK-1 |
| Ö2 | Aynı olgu art arda sorulmaz. | FR-07, US-03 KK-2 |
| Ö3 | "Öğrenildi" tanımıyla (V-04) aynı veriden çalışır, iki ayrı durum tutulmaz. | FR-09, V-04 |
| Ö4 | Veli ve çocuk tarafından anlaşılır, basit kurallar (Leitner). | FR-07 dayanağı (BA K15), kapsam kuralı |
| Ö5 | Hesaplama maliyeti ihmal edilebilir düzeyde. | NFR-09 |

**Karşılaştırma:**

| Seçenek | Ö1 | Ö2 | Ö3 | Ö4 | Ö5 |
|---|---|---|---|---|---|
| A. Hata ağırlıklı rastgele seçim (ağırlık = 1 + yanlış sayısı) | Zayıf. Yalnızca beklenen değerde geçerli, tek oturumda garanti yok. Eski hatalar sonsuza dek ağırlık taşır. | Ek kural gerekir | Zayıf, ayrı mantık | Orta | İyi |
| B. Leitner kutuları + kutu ağırlıklı rastgele seçim | Orta. Beklenen değerde geçerli, tek oturumda garanti yok. | Ek kural gerekir | İyi | İyi | İyi |
| **C. Leitner kutuları + deterministik öncelik kotası** | **İyi.** Koşul sağlandığında (aşağıda) oturum başına garanti verir. | Sıralama adımıyla | İyi | İyi | İyi |
| D. SM-2 gibi gün aralıklı aralıklı tekrar | Orta | Ek kural gerekir | Zayıf, kolaylık katsayısı gibi ek durumlar gerekir | Zayıf, açıklaması zor | İyi |

**Karar:** C. Rastgele seçeneklerde (A, B) US-03 KK-1 yalnızca istatistiksel olarak sağlanabilir. D, oturum tabanlı ve takvimsiz bir ürün için gereğinden karmaşıktır.

**Kutu tanımı** (Bölüm 6.2'deki seriden türetilir). Leitner kuralına göre doğru cevap kartı bir sonraki kutuya taşır, yanlış cevap ilk kutuya geri gönderir. Kaynakta 3 kutulu örnek de vardır [S10].

| Kutu | Koşul | Turdaki talep (kopya) |
|---|---|---|
| K1 | `seri = 0` ve en az bir deneme var (son cevap yanlış veya süre doldu) | 2 |
| Yeni | `olgu` satırı yok | 1 |
| K2 | `seri` 1 veya 2 | 1 |
| K3 | `seri ≥ 3` ("öğrenildi") | 1 |

**Seçim adımları** (oturum başında bir kez çalışır, oturum uzunluğu L = 20, V-03):
1. Havuz: seçili her tablo n için n×1…n×10. `sifir_carpimi` açıksa n×0 da eklenir (FR-01, FR-12).
2. Bir **tur listesi** oluşturulur. Önce K1 olgularının 2'şer kopyası, sonra Yeni, sonra K2, sonra K3 olguları gelir. Her grubun kendi içindeki sırası enjekte edilen rastgele üreteçle karıştırılır.
3. Tur listesinden baştan L öğe alınır. Liste L'den kısaysa tur yeniden (grup içi sıra yeniden karıştırılarak) eklenir ve L'ye ulaşılana kadar devam edilir.
4. **Sıralama (Ö2):** Seçilen çoklu küme, art arda tekrar içermeyecek şekilde sıralanır. Her adımda önceki olgudan farklı adaylar arasından rastgele seçim yapılır. Ancak bir olgunun kalan kopya sayısı kalan soru sayısının yarısını aşıyorsa o olgu zorunlu olarak seçilir. Havuz en az 10 olgudur (bir tablo) ve bir olguya turda en çok 2 kopya düşer. Bu yüzden bir olgunun payı her zaman yarının altında kalır ve sıralama her durumda mümkündür.

**Garanti koşulu (US-03 KK-1):** K1 olgu sayısının 2 katı L'yi aşmıyorsa (20 soruda en çok 10 hatalı olgu), her K1 olgusu her K2 ve K3 olgusundan kesin olarak daha çok sorulur. KK-1 örneği (6 ve 7 değil 7 ve 2 tabloları, 20 olgu): 7×8 (K1) 2 kez sorulur. 18 yeni olgu birer kez sorulur. 2×3 (K2) bu oturumda sorulmaz. Sonuç 2 > 0. 10'dan fazla hatalı olgu varsa garanti bozulabilir (BA-S07).

**Sınır:** Kural yalnızca oturum içindeki ardışıklığı kapsar. Bir oturumun son sorusu ile sonraki oturumun ilk sorusu aynı olabilir (BA-S07).

### 6.2 Olgu durumu ve "öğrenildi" hesabı (FR-09, V-04)

- Doğru cevapta `seri += 1` ve `dogru_sayisi += 1`.
- Yanlış cevapta `seri = 0` ve `yanlis_sayisi += 1`.
- Süre dolduğunda `seri = 0` ve `sure_doldu_sayisi += 1`. Kutu açısından yanlış ile aynı sayılır (geçici, BA-S03).
- İlerleme durumu: satır yoksa "henüz çalışılmadı"; `seri ≥ 3` ise "öğrenildi" (V-04: son 3 deneme doğru); diğer durumlarda "çalışılıyor".
- AS-06 (hız şartı) cevaplanırsa `seri` yalnızca süre içinde verilen doğru cevaplarda artırılarak genişletilebilir. Bu, şu an uygulanmayan bir seçenektir.

### 6.3 Kritik karar: Ebeveyn kapısı (FR-11)

**Seçim ölçütleri:** FR-11 ve US-07 KK-1 (çocuğun kazara sıfırlamasını önleme), NFR-02 (ek izin ve kişisel veri yok), Apple'ın "adult-level tasks" tanımı [S12], sadelik.

| Seçenek | Çocuğu engelleme | NFR-02 | Sadelik |
|---|---|---|---|
| **A. Hedef sınıfların üstünde aritmetik soru** (iki basamaklı × iki basamaklı, ör. 34 × 12, serbest giriş) | Orta-iyi. BA'nın aktardığı müfredatta 3. sınıf çarpması "10'a kadar olan sayıları 9'a kadar olan rakamlarla" sınırlı (BA K1). Apple örnekleri arasında matematik içeren kapı var [S12]. | İyi, izin yok | İyi |
| B. Cihaz kimlik doğrulaması (biyometri veya PIN eklentisi) | İyi | Zayıf. Ek eklenti ve biyometri izin beyanı gerekir. | Orta |
| C. Basılı tutma (ör. 3 sn) | Zayıf, çocuk kolayca keşfeder. | İyi | Çok iyi |

**Karar:** A. Her açılışta yeni soru üretilir. Yanlış cevapta yeni soru gelir. Kapı geçilince ayrıca "Tüm ilerleme silinecek. Emin misiniz?" onay diyaloğu gösterilir (US-07 KK-1). Zorluk düzeyinin yeterliliği BA'ya sorulmuştur (BA-S09).

### 6.4 Zamanlayıcı (FR-08, NFR-07)

- Varsayılan durum kapalıdır (US-04 KK-1).
- Süre `Stopwatch` ile ölçülür (monoton saat). Cihaz saatinin değişmesinden etkilenmez.
- Pratik ekranındaki "Süreyi kapat" düğmesi ve Ayarlar'daki anahtar aynı ayarı değiştirir. Değişiklik bir sonraki sorudan itibaren uygulanır (US-04 KK-3). Böylece süre sınırı, WCAG 2.2.1'in istediği gibi her an kapatılabilir.
- Uygulama arka plana geçtiğinde geri sayım duraklar, ön plana dönünce devam eder (geçici, BA-S10).

## 7. Performans

| Konu | Tasarım | Gereksinim |
|---|---|---|
| Geri bildirim gecikmesi | Cevap kontrolü bellekte tek çarpma ve karşılaştırmadır. Geri bildirim aynı karede çizilir, veritabanı yazması beklenmez. `sqflite` SQL'i arka plan iş parçacığında çalıştırır [S2]. | NFR-09 (≤ 0,5 sn) |
| Açılış | `olgu` tablosu en çok 110 satırdır (10 tablo × 11 çarpan). Açılışta tek sorguyla belleğe alınır. Oturum kullanımı satır sayısını büyütmez (Bölüm 5.1, seçenek A). | NFR-11, FR-09 |
| Soru seçimi | Havuz (≤ 110) ve L (20) ile doğrusal bir iştir. Oturum başında bir kez çalışır. | FR-07, NFR-09 |
| Yazma sıklığı | Cevap başına tek kısa işlem. Belgeye göre bir işlem diğer çağrıları bekletir [S2]. İşlemler kısa olduğu için birikme beklenmez. | NFR-10, NFR-09 |
| Ölçüm | NFR-09, entegrasyon testinde "onayla" dokunuşu ile geri bildirim widget'ının çizildiği kare arasındaki süre olarak, desteklenen en düşük sürümlü fiziksel cihazlarda (Android API 24, iOS 15) ölçülür. | NFR-09 |

Sayısal başarım iddiası kaynakla doğrulanmadığı için hedef değer yalnızca BA'nın NFR-09 ölçütüdür. Karşılandığı testle doğrulanır (Bölüm 9).

## 8. Güvenlik ve gizlilik

| Önlem | Açıklama | Gereksinim |
|---|---|---|
| Ağ izni yok (Android) | Sürüm manifestine `android.permission.INTERNET` eklenmez. Flutter şablonu bu izni içermez, ağ erişimi yalnızca geliştirme sırasında araçlar için açılır [S5]. Sürüm APK/AAB'sinde izin listesi CI'da denetlenir. | NFR-02, NFR-03, NFR-08 |
| Ağ kodu yok (iOS ve ortak) | HTTP, soket, analitik, reklam veya çökme raporlama paketi eklenmez. Bağımlılık izin listesi: Flutter SDK, `flutter_localizations`, `sqflite` ve geçişli bağımlılıkları. Yeni bağımlılık kod incelemesinde reddedilir. | NFR-02, NFR-03 |
| Kişisel veri alanı yok | Veri modelinde ad, doğum tarihi, e-posta, konum veya cihaz tanımlayıcısı alanı yoktur (Bölüm 5.2). Konum veya başka çalışma zamanı izni istenmez. | NFR-02 |
| Mağaza beyanları | Google Play'e göre "collect", veriyi cihaz dışına iletmektir; yalnızca cihazda işlenen veri beyan gerektirmez [S11]. Bu tasarımda uygulama veri iletmez. Apple Kids kategorisi üçüncü taraf analitik ve reklam içermemeyi ister [S6], bu tasarımda ikisi de yoktur. | NFR-02, NFR-03 |
| Satın alma ve dış bağlantı yok | Uygulama içi satın alma, mağaza bağlantısı veya dış web bağlantısı yoktur. Apple 1.3'ün kapı gerektiren durumları oluşmaz [S6]. Kapı yalnızca sıfırlama için kullanılır (Bölüm 6.3). | NFR-03, FR-11, FR-13 |
| Silme yolu | Tek işlemde tüm `olgu` satırları silinir, işlem yarıda kalırsa geri alınır (Bölüm 5.2). | FR-11 |
| Cihaz yedeği | Uygulama verisinin işletim sisteminin yedeklemesine (Android otomatik yedekleme, iCloud) dahil edilmesi platform varsayılanında bırakılmıştır. Google'ın Data safety sayfası yedeklemeyi ayrıca ele almıyor [S11]. Konu BA'ya sorulmuştur (BA-S08). | NFR-02, NFR-10 |

## 9. Test stratejisi

| Test türü | Kapsam | Araç | Gereksinim / US |
|---|---|---|---|
| Birim: SoruSecici | KK-1 senaryosu (7×8 K1, 2×3 K2 → 7×8 sayısı > 2×3 sayısı). Garanti koşulunun sınırı (10 ve 11 K1 olgusu). Sabit tohumla deterministik çıktı. 10.000 rastgele oturumda art arda tekrar olmadığı (özellik tabanlı test). ×0 açıkken havuzda n×0 bulunması, kapalıyken bulunmaması. Seçilen tablolar dışından soru gelmemesi. | `package:test` | FR-01, FR-07, FR-12, US-01 KK-1, US-03 KK-1/KK-2, US-08 |
| Birim: OlguDurumHesaplayici | Seri geçişleri. Son 3 doğru → "öğrenildi". Yanlış veya süre dolması → seri 0. Satır yok → "henüz çalışılmadı". | `package:test` | FR-09, V-04, US-05 KK-2 |
| Birim: Zamanlayici | Süre dolunca `sure_doldu`. Kapatıldığında bir sonraki soruda süre yok. Arka planda duraklama. | sahte saat | FR-08, NFR-07, US-04 |
| Veri katmanı | Cevap yazma ve okuma. Kapatıp yeniden açtıktan sonra değerlerin aynı olması. İşlem ortasında hata → değişiklik yok. Sıfırlamadan sonra tablonun boş, ayarların korunmuş olması. | `sqflite_common_ffi` [S2] | FR-06, FR-11, NFR-10, US-03 KK-3, US-07 KK-2 |
| Widget: akışlar | Tablo seçimi ve sınıf ön ayarları. İnceleme ekranında 10 satır. Yanlış cevapta "7 × 8 = 56". 20. sorudan sonra özet ve yıldız. Ebeveyn kapısından önce sıfırlanamaması. | `flutter_test` | FR-01–FR-05, FR-10, FR-11, FR-13, US-01, US-02, US-06, US-07, US-09 |
| Widget: erişilebilirlik | Tüm ekranlarda `androidTapTargetGuideline`, `iOSTapTargetGuideline`, `labeledTapTargetGuideline`, `textContrastGuideline` [S7]. Ayrıca tasarım paletinin 4.5:1 oranı ayrı bir kontrast hesabıyla doğrulanır (bkz. R-07). Metin ölçeği 2,0 ile her ekranda taşma (overflow) hatası olmaması. | `flutter_test` | NFR-04, NFR-05, NFR-06 |
| Widget: dil | Ekran metinlerinde İngilizce varsayılan metin kalmaması. Uygulamanın `tr` yerel ayarıyla açılması. | `flutter_test` | NFR-01 |
| Entegrasyon (fiziksel cihaz) | Uygulamayı zorla kapatıp açma ve cihazı yeniden başlatma sonrası ilerlemenin korunması. Uçak modunda tüm akışlar. NFR-09 süre ölçümü. | `integration_test`, elle test | NFR-08, NFR-09, NFR-10 |
| Derleme denetimi | Sürüm APK/AAB izin listesinde INTERNET ve başka izin olmaması. Bağımlılık listesinin izin listesiyle karşılaştırılması. | CI betiği | NFR-02, NFR-03 |
| Kullanılabilirlik | En az 5 çocukla açılış ekranından ilk soruya ≤ 3 dokunuş. Tasarımdaki yol: ilk kurulumda tablo seç (1), "Pratik" (2). Önceki seçim varsa yalnızca "Pratik" (1). | elle, BA ile | NFR-11 |

## 10. Riskler

| ID | Risk | Etki | Önlem |
|---|---|---|---|
| R-01 | AS-02'nin cevabı "yalnızca web" olursa Flutter ve SQLite kararları geçersizleşir. `sqflite` web desteği yalnızca deneysel bir ek paketle sağlanır [S2]. Web depolaması "best effort"tur [S4]. | Bölüm 3.1, 3.2 yeniden yapılır. | Alan katmanı saf Dart ve depo arayüzü soyuttur. Algoritmalar korunur. |
| R-02 | Flutter, Android API 23 ve altını, iOS 14 ve altını desteklemez [S1]. Eski aile cihazlarına kurulamaz. | Erişim kaybı | BA'ya bilgi olarak iletilir (BA-S01). |
| R-03 | NFR-05 ölçütü CSS pikseli cinsinden. Mobilde mantıksal piksel kullanılır ve iki birimin eşdeğer sayılması bir varsayımdır. | Ölçüt yorumu | 48×48 mantıksal piksel hedefi [S6 değil, S13] 24×24 asgarisinin iki katıdır, bu fark varsayımın etkisini sınırlar. |
| R-04 | FR-02 "tek ekranda" şartı ile NFR-06 "%200 büyütme" küçük ekranlarda çelişebilir: 10 satır %200 metinle sığmayabilir. | Gereksinim çelişkisi | Geçici: liste kaydırılabilir yapılır (BA-S02). |
| R-05 | Geri bildirim gösterildikten sonra ve veritabanı yazması bitmeden uygulama öldürülürse son cevap kaybolabilir. | NFR-10'a küçük sapma | Yazma, geri bildirimle aynı anda başlatılır ve sonraki soruya geçiş yazma bitince açılır. |
| R-06 | Ebeveyn kapısı, büyük kardeşler veya ileri düzeydeki çocuklar tarafından çözülebilir. | Kazara sıfırlama | Ek onay diyaloğu. Zorluk BA'ya soruldu (BA-S09). |
| R-07 | `meetsGuideline` belgesi kılavuzların eşik değerlerini yazmıyor [S7]. Otomatik kontrast testinin tam olarak 4.5:1'i denetlediği doğrulanamadı. | Test boşluğu | Palet renkleri için ayrı kontrast oranı birim testi yazılır. |
| R-08 | Üçüncü taraf çökme raporlama yok (NFR-03). Sahadaki hatalar ancak mağaza geri bildirimiyle öğrenilir. | Hata tespiti gecikir | Entegrasyon testleri desteklenen en düşük sürümlerde çalıştırılır. |
| R-09 | Uygulama verisinin cihaz yedeğine girmesinin NFR-02 açısından durumu belirsiz. Google'ın Data safety belgesi yedeklemeyi ele almıyor [S11]. | Gizlilik beyanı | BA-S08 cevabına göre yedekleme kapatılabilir. |
| R-10 | US-03 KK-1 garantisi, 20 soruluk oturumda 10'dan fazla hatalı olgu olduğunda sağlanamaz (Bölüm 6.1). | Kabul kriteri | BA-S07 |
| R-11 | Geliştirme derlemelerinde internet erişimi açıktır [S5]. Sürüm derlemesinde izin olmadığı varsayılırsa yanlışlıkla eklenen izin gözden kaçabilir. | NFR-02 | CI'da sürüm derlemesinin izin listesi denetlenir (Bölüm 9). |

Not: R-03 satırındaki 48×48 hedefi Flutter erişilebilirlik kontrol listesine dayanır ("All tappable targets should be at least 48x48 pixels") [S6].

## 11. İzlenebilirlik tablosu

| Gereksinim ID | Gereksinim özeti | Karşılayan bileşen / karar | İlgili bölüm |
|---|---|---|---|
| FR-01 | 1–10 tablolarından bir veya birkaçını seçme, karışık sorular | TabloSecimEkrani, SoruSecici havuzu | 4, 6.1 |
| FR-02 | İnceleme modu: n×1…n×10 tek ekranda | IncelemeEkrani (çoklu tabloda sayfa başına bir tablo, geçici) | 4, 10 (R-04) |
| FR-03 | Pratik modu: `n1 × n2 = ?`, ekrandaki tuş takımı, 20 soru | PratikEkrani, SayisalTusTakimi, OturumYoneticisi (L = 20) | 4, 6.1 |
| FR-04 | Anında doğru/yanlış geri bildirimi, yanlışta doğru sonuç | PratikEkrani, OturumYoneticisi | 4, 7 |
| FR-05 | Oturum özeti: toplam, doğru, yanlış olgular | OzetEkrani, bellekte `OturumOzeti` | 4, 5.2 |
| FR-06 | Olgu bazında geçmişin cihazda kalıcı saklanması | `olgu` tablosu, OlguDeposu, SQLite kararı | 3.2, 5 |
| FR-07 | Yanlış olguların daha sık sorulması, art arda tekrar olmaması | SoruSecici: Leitner + deterministik kota + sıralama | 6.1 |
| FR-08 | İsteğe bağlı süre sınırı, varsayılan kapalı, ≥ 3 seçenek, biri 6 sn | Zamanlayici, AyarlarEkrani, PratikEkrani kapatma düğmesi (6/10/15 sn, geçici) | 4, 6.4 |
| FR-09 | 10×10 ilerleme görünümü, üç durum | IlerlemeEkrani, OlguDurumHesaplayici (`seri ≥ 3`) | 4, 6.2 |
| FR-10 | 2. ve 3. sınıf ön ayarları | TabloSecimEkrani ön ayar düğmeleri | 4 |
| FR-11 | Ebeveyn kapısı ve onaydan sonra ilerlemeyi sıfırlama | EbeveynKapisi (aritmetik kapı), onay diyaloğu, tek işlemde `DELETE` | 5.2, 6.3, 8 |
| FR-12 | Ayardan açılırsa ×0 olguları | `sifir_carpimi` ayarı, SoruSecici havuzu, `carpan2` 0–10 | 5.2, 6.1 |
| FR-13 | Oturum sonunda yıldız veya rozet, mağaza yok | OzetEkrani | 4, 8 |
| NFR-01 | Arayüz %100 Türkçe | Türkçe metin dosyası, `flutter_localizations` (`tr`) | 3.3, 4, 9 |
| NFR-02 | Kişisel veri toplanmaz, hesap yok | Ağ izni yok, ağ paketi yok, kişisel veri alanı yok | 3.1, 5.2, 8 |
| NFR-03 | Reklam, üçüncü taraf analitik ve satın alma yok | Bağımlılık izin listesi, CI denetimi | 3.3, 8, 9 |
| NFR-04 | Kontrast ≥ 4.5:1 | Palet kontrast testi, `textContrastGuideline`, durumların simge ve metinle de ayrılması | 4, 9, 10 (R-07) |
| NFR-05 | Dokunma hedefi ≥ 24×24 | En az 48×48 mantıksal piksel, dokunma hedefi testleri | 4, 9, 10 (R-03) |
| NFR-06 | %200 metin büyütme | `MediaQuery.textScalerOf` ile sistem ölçeğine uyum, ölçek 2,0'da taşma testi | 3.1, 9 |
| NFR-07 | Süre sınırları kullanıcı denetiminde | Zamanlayici: varsayılan kapalı, ekranda ve ayarlarda kapatma | 6.4 |
| NFR-08 | İnternet olmadan tüm işlevler | Ağ katmanı yok, tüm veri cihazda | 2, 3.1, 8 |
| NFR-09 | Geri bildirim ≤ 0,5 sn | Bellek içi kontrol, yazmanın beklenmemesi, arka plan SQL iş parçacığı | 7 |
| NFR-10 | Yeniden başlatmada ilerleme kaybı %0 | SQLite işlemleri, cevap başına yazma | 3.2, 5.2, 10 (R-05) |
| NFR-11 | İlk soruya ≤ 3 dokunuş | Açılış ekranı = tablo seçimi, son seçimin saklanması | 4, 9 |

## 12. BA'ya geri sorular

| ID | Soru | İlgili gereksinim | Geçici karar |
|---|---|---|---|
| BA-S01 | Hedef platform nedir (AS-02)? Yalnızca web istenirse platform ve saklama kararları değişir. Android API 24 ve iOS 15 altındaki cihazların desteklenmemesi kabul edilebilir mi? | NFR-02, NFR-08, NFR-10 | Flutter, iOS + Android |
| BA-S02 | FR-02'deki "tek ekranda" ifadesi, %200 metin büyütmede (NFR-06) kaydırılabilir bir listeyi de kapsıyor mu? Birden fazla tablo seçiliyken inceleme modu nasıl davranmalı? | FR-02, NFR-06 | Kaydırılabilir liste, tablo başına bir sayfa |
| BA-S03 | US-04 KK-2'deki "yanlış veya cevapsız" ifadesinden hangisi geçerli? Süresi dolan soru özet ekranında yanlışlar arasında mı, ayrı bir başlıkta mı gösterilmeli? | FR-08, FR-05, FR-07 | Ayrı sayılır, kutu ve seri açısından yanlış gibi işlenir, özette yanlışlar arasında listelenir. |
| BA-S04 | FR-08'deki süre seçeneklerinden 6 sn dışındaki değerler neler olmalı? | FR-08 | 6, 10, 15 sn |
| BA-S05 | FR-12 açıkken ×0 olguları FR-09'daki 10×10 ilerleme tablosunda gösterilmeli mi? Gösterilecekse nasıl? | FR-09, FR-12 | Geçmiş saklanır, ızgarada gösterilmez. |
| BA-S06 | FR-06'daki "geçmiş" için olgu başına sayılar ve son doğru serisi yeterli mi, yoksa her cevabın ayrı kaydı (zaman, verilen cevap) gerekiyor mu? | FR-06 | Özet satırı (Bölüm 5.1) |
| BA-S07 | US-03 KK-1 yalnızca bir oturumdaki hatalı olgu sayısı 10'u (L/2) aşmadığında kesin olarak sağlanabiliyor. Bu koşul kabul kriterine eklenebilir mi? "Art arda iki kez sorulmama" kuralı oturumlar arası geçişi de kapsamalı mı? | FR-07, US-03 | Koşullu garanti, yalnızca oturum içi |
| BA-S08 | İlerleme verisi işletim sisteminin cihaz yedeğine (Android otomatik yedekleme, iCloud) dahil edilmeli mi, hariç tutulmalı mı? | NFR-02, NFR-10 | Platform varsayılanı |
| BA-S09 | Ebeveyn kapısı olarak iki basamaklı × iki basamaklı çarpma sorusu yeterli mi? Sıfırlama ayarları (zamanlayıcı, ×0, seçili tablolar) da kapsamalı mı? | FR-11, US-07 | Aritmetik kapı, ayarlar korunur. |
| BA-S10 | Zamanlayıcı açıkken uygulama arka plana geçerse soru süresi duraklamalı mı, devam mı etmeli? | FR-08, NFR-07 | Duraklar |
| BA-S11 | Yarıda bırakılan bir oturumda (uygulama kapatıldı) özet gösterilmeli mi? Yıldız verilmeli mi? | FR-05, FR-13 | Verilen cevaplar kaydedilir, özet ve yıldız yalnızca tamamlanan oturumda gösterilir. |
| BA-S12 | 7×8 ve 8×7 ayrı olgular mı (FR-09'daki 10×10 ızgara bunu düşündürüyor), yoksa değişme özelliği gereği tek olgu mu sayılmalı? | FR-06, FR-07, FR-09 | Ayrı olgular |
| BA-S13 | NFR-05'teki CSS pikseli ölçütü mobil uygulamada mantıksal piksel olarak yorumlanabilir mi? AS-05'e göre daha büyük bir değer belirlenecek mi? | NFR-05 | 48×48 mantıksal piksel |

## Kaynaklar

| # | Başlık | URL | Alınan bilgi |
|---|---|---|---|
| S1 | Flutter – Supported deployment platforms | https://docs.flutter.dev/reference/supported-platforms | Android desteklenen aralık "24 to 37", "23 and earlier" desteklenmez. iOS desteklenen aralık "15 to 27", "14 and earlier" desteklenmez. |
| S2 | pub.dev – sqflite | https://pub.dev/packages/sqflite | Android, iOS, macOS desteği. Android `minSdk` 19 ("your Flutter version's minimum applies"), iOS 12, macOS 10.14. Web için deneysel ek paket. "SQL runs in a background thread (Android, iOS, macOS), never blocking the UI". "A transaction blocks the other calls on the database until it ends". "There are no concurrent reads". İşlem içinde `db` kullanımı kilitlenme yaratır. Testler için `sqflite_common_ffi`. Güncel sürüm 2.4.6. Belgede izin şartı yok. |
| S3 | pub.dev – shared_preferences | https://pub.dev/packages/shared_preferences | "there is no guarantee that writes will be persisted to disk after returning". Kritik veri için önerilmez. |
| S4 | web.dev – Storage for the web | https://web.dev/articles/storage-for-the-web | "the browser may evict site data at its discretion". Safari'de "a seven-day cap on all script writable storage", "does not apply to installed PWAs that have been added to the home screen". |
| S5 | Flutter – Build and release an Android app | https://docs.flutter.dev/deployment/android | "The standard template doesn't include this tag but allows Internet access during development to enable communication between Flutter tools and a running app." INTERNET izni, uygulama internete ihtiyaç duyarsa eklenir. |
| S6 | Flutter – Accessibility | https://docs.flutter.dev/ui/accessibility | "All tappable targets should be at least 48x48 pixels". "contrast ratio of at least 4.5:1". "The UI should remain legible and usable at very large scale factors for text size". |
| S7 | Flutter API – meetsGuideline | https://api.flutter.dev/flutter/flutter_test/meetsGuideline.html | "Asserts that the currently rendered widget meets the provided accessibility guideline". Kılavuzlar: `androidTapTargetGuideline`, `iOSTapTargetGuideline`, `textContrastGuideline`, `labeledTapTargetGuideline`. Eşik değerleri sayfada yazmıyor. |
| S8 | Flutter API – MediaQueryData.textScaler | https://api.flutter.dev/flutter/widgets/MediaQueryData/textScaler.html | `MediaQueryData.fromView` ile oluşturulduğunda "reflects the platform's preferred text scaling strategy". Widget'lar `MediaQuery.textScalerOf` ile kullanır. |
| S9 | Flutter – Internationalizing Flutter apps | https://docs.flutter.dev/ui/internationalization | "By default, Flutter only provides US English localizations". `flutter_localizations` Material ve Cupertino kitaplıklarını yerelleştirir. `sdk: flutter` ile eklenir. |
| S10 | Wikipedia – Leitner system (ikincil kaynak) | https://en.wikipedia.org/wiki/Leitner_system | "If they succeed, they send the card to the next group. If they fail, they send it back to the first group." Üç kutulu örnek: Box 1 sık yanlış yapılan kartlar, Box 3 iyi bilinen kartlar. |
| S11 | Google Play Console Yardım – Data safety bölümü | https://support.google.com/googleplay/android-developer/answer/10787469?hl=en | "'Collect' means transmitting data from your app off a user's device." Yalnızca cihazda işlenen veri "does not need to be disclosed". Android yedeklemesinden söz edilmiyor. |
| S12 | Apple – Kids apps | https://developer.apple.com/app-store/kids-apps/ | Ebeveyn kapıları "adult-level tasks that must be completed in order to continue". Örnekler arasında "a parental gate task that requires math" var. |
| S13 | Apple – App Store Review Guidelines | https://developer.apple.com/app-store/review/guidelines/ | 1.3: "must not include links out of the app, purchasing opportunities, or other distractions to kids unless reserved for a designated area behind a parental gate". "Apps in the Kids Category should not include third-party analytics or third-party advertising." |

**Kaynak numaralandırma düzeltmesi:** Bölüm 8'deki "Apple Kids kategorisi … [S6]" ve "Apple 1.3 … [S6]" atıfları **[S13]** olarak okunmalıdır. Bölüm 10 R-03'teki 48×48 dayanağı ise **[S6]**'dır. Bu satırın nedeni, aynı dokümanda Flutter erişilebilirlik sayfasının (S6) ve Apple inceleme kurallarının (S13) iki farklı numarayla geçmesidir.