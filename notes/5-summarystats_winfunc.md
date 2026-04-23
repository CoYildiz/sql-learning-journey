# 5 - PostgreSQL Summary Stats and Window Functions 

### 1: Gelişmiş Gruplama ve Pivot İşlemleri (Advanced Grouping & Pivoting)

#### 1. Kavramlar ve Tanımlar

- **ROLLUP:** Belirtilen gruplama sütunları için hiyerarşik alt toplam satırları üreten ve grup düzeyinde ile genel toplamları kısa bir şekilde oluşturmak için yararlı olan bir GROUP BY değiştiricisidir.
    
- **CUBE:** Listelenen sütunlar için grup düzeyindeki toplamaların tüm olası kombinasyonlarını üreten, böylece her alt toplamı ve genel toplamı veren bir GROUP BY değiştiricisidir.
    
- **COALESCE:** Argüman listesindeki ilk NULL olmayan değeri döndüren, genellikle ROLLUP/CUBE veya LAG/LEAD'den gelen NULL'ları varsayılan bir değerle değiştirmek için kullanılan skaler bir fonksiyondur.
    
- **STRING_AGG:** Belirtilen bir ayırıcıyı (separator) kullanarak birden fazla satırdaki değerleri tek bir dizede birleştiren, listeleri tek bir satıra sıkıştırmak için kullanışlı bir toplulaştırma (aggregate) fonksiyondur.
    
- **CREATE EXTENSION:** Bir uzantıyı (tablefunc gibi) etkinleştiren, böylece ek fonksiyonlarının ve özelliklerinin veritabanında kullanılabilir hale gelmesini sağlayan bir PostgreSQL komutudur.
    
- **CROSSTAB:** Bir sütunun farklı değerlerini çıktı sütunlarına dönüştürerek satırları sütunlara çeviren (pivot) ve çapraz tablolanmış bir sonuç üreten bir PostgreSQL tablefunc uzantısı (extension) fonksiyonudur.
    

---

#### 2. ROLLUP, CUBE ve COALESCE Örneği

**Senaryo ve Tablo:**

Bir şirketin bölgelere ve yıllara göre gelirlerini analiz etmek istiyoruz.

**Tablo: `bolge_gelirleri`**

|**bolge**|**yil**|**gelir**|
|---|---|---|
|Marmara|2025|100|
|Marmara|2026|150|
|Ege|2026|200|

**Sorgu 1 (ROLLUP ve COALESCE):**

ROLLUP, veriyi `bolge` -> `yil` hiyerarşisine göre toplayarak ilerler. Hiyerarşinin üst basamaklarındaki toplamları gösterirken ilgili sütunlara NULL atar. `COALESCE` ile bu NULL değerlerini anlamlı metinlere çeviririz.

SQL

```sql
SELECT 
    COALESCE(bolge, 'Tüm Bölgeler') AS bolge,
    COALESCE(CAST(yil AS VARCHAR), 'Tüm Yıllar') AS yil,
    SUM(gelir) AS toplam_gelir
FROM bolge_gelirleri
GROUP BY ROLLUP(bolge, yil);
```

**Çıktı 1:**

|**bolge**|**yil**|**toplam_gelir**|
|---|---|---|
|Marmara|2025|100|
|Marmara|2026|150|
|Marmara|Tüm Yıllar|250|
|Ege|2026|200|
|Ege|Tüm Yıllar|200|
|Tüm Bölgeler|Tüm Yıllar|450|

_(ROLLUP hiyerarşik çalışır: Önce her bölge+yıl detayını verir, sonra Marmara'nın kendi içindeki toplamını verir, sonra Ege'nin toplamını verir, en sonda tüm veritabanının genel toplamını (450) verir)._

---

#### 3. STRING_AGG Örneği

**Senaryo ve Tablo:**

Departmanlardaki çalışan isimlerini tek bir satırda, aralarına virgül koyarak listelemek istiyoruz.

**Tablo: `departman_calisanlari`**

|**departman**|**isim**|
|---|---|
|IT|Ali|
|IT|Veli|
|İK|Ayşe|

**Sorgu:**

SQL

```sql
SELECT 
    departman, 
    STRING_AGG(isim, ', ') AS calisan_listesi
FROM departman_calisanlari
GROUP BY departman;
```

**Çıktı:**

|**departman**|**calisan_listesi**|
|---|---|
|IT|Ali, Veli|
|İK|Ayşe|

---

#### 4. CREATE EXTENSION ve CROSSTAB Örneği

**Senaryo ve Tablo:**

Aylık bazda satılan ürün miktarları dikey (satır satır) durmaktadır. Analiz kolaylığı için her ayı ayrı bir sütun olacak şekilde yatay formata (pivot) çevirmek istiyoruz. Bu işlemi yapabilmek için önce `tablefunc` eklentisini veritabanına kurmalıyız.

**Tablo: `aylik_satislar`**

|**urun_adi**|**ay**|**miktar**|
|---|---|---|
|Laptop|Ocak|10|
|Laptop|Subat|15|
|Telefon|Ocak|30|

**Sorgu:**

SQL

```sql
-- Eklentiyi aktif etme (Sadece bir kez çalıştırılır)
CREATE EXTENSION IF NOT EXISTS tablefunc;

-- CROSSTAB kullanımı
SELECT * FROM CROSSTAB(
    'SELECT urun_adi, ay, miktar 
     FROM aylik_satislar 
     ORDER BY 1, 2'
) AS donusturulmus_tablo (
    urun_adi VARCHAR, 
    "Ocak" INT, 
    "Subat" INT
);
```

**Çıktı:**

|**urun_adi**|**Ocak**|**Subat**|
|---|---|---|
|Laptop|10|15|
|Telefon|30|NULL|

_(Satır düzeyindeki "ay" verileri, dinamik olarak sütun başlıklarına dönüştürülmüştür)._


### 2: Pencere Fonksiyonlarına Giriş (Window Functions Basics)

#### 1. Kavramlar ve Tanımlar

- **Window function (Pencere Fonksiyonu):** Geçerli satırla ilgili bir dizi tablo satırı üzerinde hesaplama yapan ve satırları daraltmadan (collapsing rows) her giriş satırı için ayrı bir sonuç üreten bir SQL fonksiyonudur.
    
- **OVER clause:** Bir fonksiyonu pencere fonksiyonu olarak tanımlayan ve PARTITION BY, ORDER BY ile çerçeve (frame) ifadeleri gibi alt ifadeleri (subclauses) belirterek pencereyi oluşturan ifadedir.
    
- **PARTITION BY:** Pencere fonksiyonunun her bir bölüme bağımsız olarak uygulanabilmesi için satırları ayrı bölümlere (gruplara) bölen bir OVER alt ifadesidir.
    
- **ORDER BY (within OVER):** Pencere içindeki satırların sıralamasını tanımlayan ve ROW_NUMBER, RANK gibi fonksiyonların veya çerçeve tabanlı toplamaların (frame-based aggregates) nasıl uygulanacağını belirleyen OVER alt ifadesidir.
    

#### 2. Senaryo ve Tablo Yapısı

Bir fabrikadaki iki farklı üretim hattının günlük üretim miktarlarını takip ettiğimiz bir veritabanı tablomuz bulunmaktadır. Normal bir `GROUP BY` işlemi satırları birleştirip tek satıra düşürür. Ancak amacımız, günlük kayıtların orijinal satırlarını bozmadan (daraltmadan), her satırın yanına o hattın **toplam üretim miktarını** ve o günün üretim miktarının **kendi hattı içindeki sıralamasını** yazdırmaktır.

**Tablo: `gunluk_uretim`**

|**uretim_id**|**hat_adi**|**tarih**|**miktar**|
|---|---|---|---|
|1|Hat A|2026-05-01|100|
|2|Hat A|2026-05-02|150|
|3|Hat A|2026-05-03|120|
|4|Hat B|2026-05-01|300|
|5|Hat B|2026-05-02|250|

#### 3. Sorgu (Query)

- İlk pencere fonksiyonunda (`hat_toplam_uretim`), `PARTITION BY hat_adi` kullanarak veriyi sanal olarak Hat A ve Hat B pencerelerine ayırıyoruz ve `SUM` ile sadece o pencerenin toplamını alıyoruz.
    
- İkinci pencere fonksiyonunda (`hat_ici_siralama`), yine hattına göre bölüyoruz ancak bu kez `ORDER BY miktar DESC` ekleyerek her pencereyi kendi içinde en yüksek üretimden en düşüğe doğru sıralıyoruz.
    

SQL

```SQL
SELECT 
    hat_adi, 
    tarih, 
    miktar,
    SUM(miktar) OVER(PARTITION BY hat_adi) AS hat_toplam_uretim,
    ROW_NUMBER() OVER(PARTITION BY hat_adi ORDER BY miktar DESC) AS hat_ici_siralama
FROM gunluk_uretim;
```

#### 4. Çıktı (Output)

|**hat_adi**|**tarih**|**miktar**|**hat_toplam_uretim**|**hat_ici_siralama**|
|---|---|---|---|---|
|Hat A|2026-05-02|150|370|1|
|Hat A|2026-05-03|120|370|2|
|Hat A|2026-05-01|100|370|3|
|Hat B|2026-05-01|300|550|1|
|Hat B|2026-05-02|250|550|2|

_(Görüldüğü üzere toplam 5 satırlık veri korunmuştur. Hat A'nın toplamı (100+150+120=370) sadece Hat A satırlarına, Hat B'nin toplamı (300+250=550) sadece Hat B satırlarına yazılmıştır. Sıralama da yine hatların kendi içinde bağımsız olarak yapılmıştır)._


### 3: Sıralama ve Dilimleme (Ranking & Paging)

#### 1. Kavramlar ve Tanımlar

- **ROW_NUMBER:** Belirtilen sıralamaya kesinlikle bağlı kalarak pencere içindeki her satıra benzersiz ardışık bir tam sayı atayan bir pencere fonksiyonudur.
    
- **RANK:** Eşit değerlere eşit sıralar atayan ve sonraki sıralarda boşluklar bırakan (örneğin, 2. sıradaki eşitliklerin bir sonraki sıranın 4 olmasına neden olduğu) bir sıralama pencere fonksiyonudur.
    
- **DENSE_RANK:** Eşit değerlere eşit sıralar atayan ancak boşluk bırakmayan, böylece sıraların eşitliklerden sonra ardışık olarak arttığı bir sıralama pencere fonksiyonudur.
    
- **NTILE:** Sayfalama (paging) veya kantil (quantile) sınıflandırması için sıralı satırları "n" adet yaklaşık olarak eşit sepete (bucket) bölen ve her satıra bir sepet numarası atayan pencere fonksiyonudur.
    

#### 2. Senaryo ve Tablo Yapısı

Makine öğrenmesi alanında çeşitli algoritmaları başarı metriklerine (F1 Skoru) göre test ettiğimizi varsayalım. Bu modelleri sıralamak ve aynı skoru alan modeller arasındaki sıralama farklarını (ROW_NUMBER, RANK, DENSE_RANK) görmek ve ayrıca en başarılı ile en başarısız modelleri 2 ayrı gruba (NTILE) ayırmak istiyoruz.

**Tablo: `model_skorlari`**

|**algoritma**|**f1_skoru**|
|---|---|
|SVM|0.95|
|Random Forest|0.95|
|Gradient Boosting|0.92|
|KNN|0.88|
|Naive Bayes|0.85|

_(Not: SVM ve Random Forest modelleri 0.95 ile tamamen aynı skora sahiptir)._

#### 3. Sorgu (Query)

Tek bir sorguda tüm sıralama algoritmalarını `OVER(ORDER BY f1_skoru DESC)` penceresi üzerinden uyguluyoruz. `NTILE(2)` ifadesi satırları 2 eşit (veya yaklaşığı) dilime böler.

SQL

```sql
SELECT 
    algoritma,
    f1_skoru,
    ROW_NUMBER() OVER(ORDER BY f1_skoru DESC) AS row_num,
    RANK() OVER(ORDER BY f1_skoru DESC) AS rank_no,
    DENSE_RANK() OVER(ORDER BY f1_skoru DESC) AS dense_rank_no,
    NTILE(2) OVER(ORDER BY f1_skoru DESC) AS dilim_no
FROM model_skorlari;
```

#### 4. Çıktı (Output)

|**algoritma**|**f1_skoru**|**row_num**|**rank_no**|**dense_rank_no**|**dilim_no**|
|---|---|---|---|---|---|
|SVM|0.95|1|1|1|1|
|Random Forest|0.95|2|1|1|1|
|Gradient Boosting|0.92|3|3|2|1|
|KNN|0.88|4|4|3|2|
|Naive Bayes|0.85|5|5|4|2|

**Farkların Özeti:**

- `ROW_NUMBER` eşitliklere bakmaksızın rastgele birini 1. diğerini 2. seçti ve ardışık devam etti.
    
- `RANK` eşit skorlulara 1 verdi. Ancak 2. sırayı "yaktığı" için bir sonraki gelen modele 3 numarasını atadı.
    
- `DENSE_RANK` eşit skorlulara 1 verdi, sıralamada boşluk bırakmadı ve bir sonraki gelen modele 2 numarasını atadı.
    
- `NTILE(2)` 5 satırlık bu veriyi iki gruba bölerken (tek sayı olduğu için) ilk gruba 3, ikinci gruba 2 eleman atayarak gruplandırdı.


### 4: Göreceli Veri Çekme (Fetching Functions)

#### 1. Kavramlar ve Tanımlar

- **LAG:** Geçerli satırdan _n_ satır önceki bir sütunun değerini döndüren, o önceki satır yoksa NULL döndüren göreceli bir veri çekme pencere fonksiyonudur.
    
- **LEAD:** Geçerli satırdan _n_ satır sonraki bir sütunun değerini döndüren, sonraki satır yoksa NULL döndüren göreceli bir veri çekme pencere fonksiyonudur.
    
- **LAST_VALUE:** Pencere veya bölümdeki son değeri döndüren mutlak bir veri çekme pencere fonksiyonudur; gerçek son değeri elde etmek için genellikle çerçevenin genişletilmesini (örneğin, `RANGE/ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` şeklinde) gerektirir.
    

#### 2. Senaryo ve Tablo Yapısı

Geliştirdiğimiz bir makine öğrenmesi modelinin her bir eğitim döngüsündeki (epoch) hata payını (loss) izlediğimiz bir log tablosu düşünelim. Amacımız, geçerli epoch'taki loss değerini **bir önceki epoch** ile karşılaştırmak, **bir sonraki epoch'ta** ne olacağını aynı satırda görmek ve modelin eğitim sonundaki **final loss değerini** referans olarak her satıra eklemektir.

**Tablo: `model_egitim_loglari`**

|**epoch_no**|**loss_degeri**|
|---|---|
|1|0.80|
|2|0.65|
|3|0.50|
|4|0.45|

#### 3. Sorgu (Query)

- `LAG(loss_degeri, 1)`: Mevcut satırdan 1 adım geriye gider.
    
- `LEAD(loss_degeri, 1)`: Mevcut satırdan 1 adım ileriye gider.
    
- `LAST_VALUE(loss_degeri)`: Varsayılan olarak pencere çerçevesi o anki satırda bittiği için, gerçek son değeri bulmak adına `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` komutuyla çerçeveyi tablonun en sonuna kadar genişletiyoruz.
    

SQL

```sql
SELECT 
    epoch_no,
    loss_degeri,
    LAG(loss_degeri, 1) OVER(ORDER BY epoch_no) AS onceki_loss,
    LEAD(loss_degeri, 1) OVER(ORDER BY epoch_no) AS sonraki_loss,
    LAST_VALUE(loss_degeri) OVER(
        ORDER BY epoch_no 
        ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING
    ) AS final_loss
FROM model_egitim_loglari;
```

#### 4. Çıktı (Output)

|**epoch_no**|**loss_degeri**|**onceki_loss**|**sonraki_loss**|**final_loss**|
|---|---|---|---|---|
|1|0.80|NULL|0.65|0.45|
|2|0.65|0.80|0.50|0.45|
|3|0.50|0.65|0.45|0.45|
|4|0.45|0.50|NULL|0.45|

_(İlk satırın öncesi olmadığı için `LAG` sonucu NULL döndü. Son satırın sonrası olmadığı için `LEAD` sonucu NULL döndü. `LAST_VALUE` ise çerçevenin genişletilmesi sayesinde her satırda doğru bir şekilde son epoch'un (0.45) değerini gösterdi)._

### 5: Çerçeveler ve Hareketli Hesaplamalar (Window Frames & Moving Calcs)

#### 1. Kavramlar ve Tanımlar

- **ROWS BETWEEN:** Mevcut satırdan önceki veya sonraki fiziksel satır sayısı cinsinden pencereyi tanımlayan bir çerçeve belirtimidir (örneğin, `ROWS BETWEEN 2 PRECEDING AND CURRENT ROW`).
    
- **RANGE BETWEEN:** Pencereyi `ORDER BY` ifadesindeki mantıksal değer aralıkları açısından tanımlayan ve yinelenen `ORDER BY` değerlerini tek bir aralık varlığı olarak işleyen bir çerçeve belirtimidir.
    
- **Aggregate window function:** Pencere veya bölümdeki satırlar üzerinde hesaplanan, satır başına bir sonuç döndüren geleneksel bir toplulaştırma (SUM, AVG, COUNT, MIN, MAX) fonksiyonunun pencere fonksiyonu olarak uygulanmış halidir.
    
- **Running total (cumulative sum):** Genellikle bölümün başından başlayan ve mevcut satırda sona eren bir çerçeve ve sıralama üzerinde kullanılan, kümülatif toplam üreten bir `SUM` pencere fonksiyonudur.
    
- **Moving average:** Kısa vadeli dalgalanmaları düzeltmek için mevcut ve önceki (veya sonraki) satırların belirli bir çerçevesi üzerinden değerin ortalamasını alan, zamana veya sıraya dayalı bir pencere hesaplamasıdır.
    

#### 2. Senaryo ve Tablo Yapısı

Ağ trafiği verilerini saniye bazında takip ettiğiniz bir Network Intrusion Detection (Ağ Sızma Tespiti) projesinde çalıştığınızı varsayalım. Gelen paket sayılarındaki ani sıçramaları (anomali) tespit etmek için hem kümülatif toplamı hem de son 3 saniyenin hareketli ortalamasını hesaplamanız gerekiyor.

**Tablo: `ag_trafigi`**

|**saniye**|**paket_sayisi**|
|---|---|
|1|50|
|2|60|
|3|100|
|4|70|
|5|80|

#### 3. Sorgu (Query)

Bu sorguda `Running Total` için tablonun en başından o anki satıra kadar olan çerçeveyi, `Moving Average` için ise sadece son 3 satırı kapsayan fiziksel çerçeveyi kullanıyoruz.

SQL

```sql
SELECT 
    saniye,
    paket_sayisi,
    -- Kümülatif Toplam (Running Total)
    SUM(paket_sayisi) OVER(
        ORDER BY saniye 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS kumulatif_paket,
    -- 3 Saniyelik Hareketli Ortalama (Moving Average)
    AVG(paket_sayisi) OVER(
        ORDER BY saniye 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS hareketli_ortalama
FROM ag_trafigi;
```

#### 4. Çıktı (Output)

|**saniye**|**paket_sayisi**|**kumulatif_paket**|**hareketli_ortalama**|
|---|---|---|---|
|1|50|50|50.0|
|2|60|110|55.0|
|3|100|210|70.0|
|4|70|280|76.6|
|5|80|360|83.3|

_(Saniye 3'teki hareketli ortalama (50+60+100)/3 = 70 olarak hesaplanmıştır. Saniye 4'te pencere kayar ve (60+100+70)/3 = 76.6 sonucunu verir)._


### 6: Ortak Tablo İfadeleri ve Veri Mimarisi (CTEs & Data Architecture)

#### 1. Kavramlar ve Tanımlar

- **CTE (Common Table Expression):** Sorguların okunabilirliğini artırmak ve karmaşık mantığı basitleştirmek için tek bir SQL ifadesi içinde referans verebileceğiniz, `WITH` ile tanımlanan adlandırılmış geçici bir sonuç kümesidir.
    
- **Data warehouse (Veri Ambarı):** Raporlama ve analiz için büyük miktarlardaki organizasyonel verileri toplamak, entegre etmek ve depolamak için tasarlanmış merkezi bir bilgisayar sistemidir.
    
- **AWS Lambda:** Olaylara (events) yanıt olarak kod çalıştıran, altta yatan bilgi işlem kaynaklarını otomatik olarak yöneten ve yalnızca fiili yürütme süresi için ücret alan sunucusuz (serverless) bir bilgi işlem servisidir.
    

#### 2. Senaryo ve Tablo Yapısı

Bir **Data Warehouse** (Veri Ambarı) ortamında çalıştığınızı varsayalım. **AWS Lambda** aracılığıyla her saat başı tetiklenen bir ETL (Extract-Transform-Load) süreciyle veritabanınıza ham satış verileri akıyor. Amacımız, bu karmaşık verileri pencere fonksiyonları ve CTE kullanarak temizlemek, ardından nihai raporu oluşturmaktır.

**Tablo: `ham_satis_verileri`**

|**satis_id**|**urun_kodu**|**satis_tutari**|**tarih**|
|---|---|---|---|
|101|A1|500|2026-05-23 10:00|
|102|A1|600|2026-05-23 11:00|
|103|B1|200|2026-05-23 10:30|

#### 3. Sorgu (Query)

Bu örnekte, önce `SatisAnalizi` adında bir CTE tanımlıyoruz. Bu CTE içerisinde pencere fonksiyonu kullanarak her ürünün kümülatif toplamını hesaplıyoruz. Ana sorguda ise bu CTE'yi sanki gerçek bir tabloymuş gibi kullanarak basit bir filtreleme yapıyoruz.

SQL

```sql
WITH SatisAnalizi AS (
    SELECT 
        urun_kodu,
        satis_tutari,
        tarih,
        -- Ürün bazlı kümülatif toplam (Running Total)
        SUM(satis_tutari) OVER(
            PARTITION BY urun_kodu 
            ORDER BY tarih
        ) AS urun_bazli_kumulatif
    FROM ham_satis_verileri
)
-- Ana sorgu CTE'yi referans alır
SELECT * FROM SatisAnalizi
WHERE urun_bazli_kumulatif > 300;
```

#### 4. Çıktı (Output)

|**urun_kodu**|**satis_tutari**|**tarih**|**urun_bazli_kumulatif**|
|---|---|---|---|
|A1|500|2026-05-23 10:00|500|
|A1|600|2026-05-23 11:00|1100|

_(B1 ürünü için kümülatif toplam sadece 200 olduğu için ana sorgudaki filtreye takılmış ve listelenmemiştir. CTE kullanımı sayesinde pencere fonksiyonu hesaplaması ile filtreleme işlemi mantıksal olarak birbirinden ayrılmış ve sorgu okunabilirliği artırılmıştır)._
