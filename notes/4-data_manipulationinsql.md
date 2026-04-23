# 4 - Data Manipulation in SQL


### 1: Koşullu Mantık (Conditional Logic)

#### 1. Kavramlar ve Tanımlar

- **CASE statement:** SQL'de bir veya daha fazla `WHEN` koşulunu değerlendiren ve karşılık gelen `THEN` değerini veya bir `ELSE` değerini döndüren, sorgu sonucunda tek bir sütun üreten koşullu bir ifadedir.
    
- **WHEN clause:** Değerlendirilecek mantıksal bir testi belirten `CASE` ifadesinin bir bileşenidir; test doğru (true) ise `CASE`, ilişkili `THEN` sonucunu üretir.
    
- **THEN clause:** Bir `CASE` ifadesinin, kendisinden önceki `WHEN` koşulu doğru olarak değerlendirildiğinde döndürülen çıktı değerini tanımlayan kısmıdır.
    
- **ELSE clause:** `CASE` ifadesinin isteğe bağlı son dalıdır ve `WHEN` koşullarının hiçbiri doğru olmadığında kullanılacak varsayılan bir değer sağlar.
    
- **END (CASE end) and Alias:** `END`, bir `CASE` ifadesini sonlandırır; `Alias` (takma ad) ise `CASE` sonucuna atanan kullanıcı tanımlı bir sütun adıdır, böylece diğer herhangi bir sütun gibi referans gösterilebilir.
    

#### 2. Senaryo ve Tablo Yapısı

Öğrencilerin sınav notlarının tutulduğu bir veritabanı tablomuz bulunmaktadır. Amacımız, bu sayısal notları belirli koşullara göre (80 ve üzeri, 50 ve üzeri, 50 altı) değerlendirip metin tabanlı bir "Başarı Durumu" sütunu oluşturmaktır.

**Tablo: `ogrenci_notlari`**

|**ogrenci_id**|**isim**|**sinav_notu**|
|---|---|---|
|1|Demir|85|
|2|Asya|45|
|3|Bulut|70|
|4|Selin|92|

#### 3. Sorgu (Query)

Mantıksal sıralama önemlidir; SQL `CASE` ifadesini yukarıdan aşağıya doğru okur ve doğru olan ilk `WHEN` koşulunu yakaladığında diğerlerine bakmadan sonucu yazdırır.

SQL

```
SELECT 
    isim, 
    sinav_notu,
    CASE 
        WHEN sinav_notu >= 80 THEN 'Pekiyi'
        WHEN sinav_notu >= 50 THEN 'Geçti'
        ELSE 'Kaldı' 
    END AS basari_durumu
FROM ogrenci_notlari;
```

#### 4. Çıktı (Output)

|**isim**|**sinav_notu**|**basari_durumu**|
|---|---|---|
|Demir|85|Pekiyi|
|Asya|45|Kaldı|
|Bulut|70|Geçti|
|Selin|92|Pekiyi|
###  2: Alt Sorgular (Subqueries)

#### 1. Kavramlar ve Tanımlar

- **Simple subquery (Basit Alt Sorgu):** Başka bir sorgunun içine yerleştirilmiş, bağımsız olarak yürütülebilen ve dış sorguya bir değer, liste veya tablo sağlamak için bir kez değerlendirilen bağımsız sorgudur.
    
- **Scalar subquery (Skaler Alt Sorgu):** Tek bir değer (bir satır ve bir sütun) döndüren ve tipik olarak SELECT veya WHERE ifadelerinde bir toplama veya karşılaştırmalı sabit sağlamak için kullanılan alt sorgudur.
    
- **Correlated subquery (İlişkili Alt Sorgu):** Dış sorgudan sütunlara referans veren ve dış sonuç kümesinin her satırı için yeniden yürütülen, bu nedenle dış sorgunun geçerli satırına bağımlı olan alt sorgudur.
    
- **Subquery in FROM / derived table (FROM İçinde Alt Sorgu / Türetilmiş Tablo):** FROM ifadesine yerleştirilen ve bir takma ad (alias) ile geçici bir tablo döndüren, birleştirme veya seçme işleminden önce karmaşık yeniden şekillendirme veya ön toplama yapılmasına olanak tanıyan alt sorgudur.
    

#### 2. Senaryo ve Tablo Yapısı

Şirket çalışanlarının maaşlarını ve bağlı oldukları departmanları tutan iki tablomuz bulunmaktadır. Alt sorguları kullanarak şirket ortalaması ve departman ortalaması gibi dinamik metrikler üzerinden filtrelemeler yapacağız.

**Tablo 1: `calisanlar`**

|**id**|**isim**|**departman_id**|**maas**|
|---|---|---|---|
|1|Ali|10|50000|
|2|Ayşe|10|60000|
|3|Can|20|40000|
|4|Ece|20|45000|

**Tablo 2: `departmanlar`**

|**departman_id**|**departman_adi**|
|---|---|
|10|IT|
|20|İK|

#### 3. Scalar Subquery / Simple Subquery (Skaler / Basit Alt Sorgu)

Amacımız, **tüm şirketin maaş ortalamasından** daha fazla maaş alan çalışanları listelemektir. Alt sorgu (parantez içindeki kısım) bağımsız olarak çalışır, tek bir değer (şirket ortalaması olan 48750) üretir ve ana sorgu bu sabite göre filtreleme yapar.

**Sorgu:**

SQL

```
SELECT isim, maas
FROM calisanlar
WHERE maas > (
    SELECT AVG(maas) 
    FROM calisanlar
);
```

**Çıktı:**

|**isim**|**maas**|
|---|---|
|Ali|50000|
|Ayşe|60000|
|_(Şirket ortalaması 48750 olduğu için sadece bu değerin üzerindeki çalışanlar listelenmiştir)._||

#### 4. Correlated Subquery (İlişkili Alt Sorgu)

Amacımız, tüm şirketin değil, **sadece kendi departmanının maaş ortalamasından** daha yüksek maaş alan çalışanları bulmaktır. Bu durumda alt sorgu bağımsız çalışamaz; dış sorgudaki her bir çalışan satırı için (`c1.departman_id = c2.departman_id` şartıyla) tekrar tekrar hesaplanır.

**Sorgu:**

SQL

```
SELECT c1.isim, c1.maas
FROM calisanlar AS c1
WHERE c1.maas > (
    SELECT AVG(c2.maas)
    FROM calisanlar AS c2
    WHERE c1.departman_id = c2.departman_id
);
```

**Çıktı:**

|**isim**|**maas**|
|---|---|
|Ayşe|60000|
|Ece|45000|
|_(IT departmanının ortalaması 55000'dir, Ayşe bu ortalamayı geçer. İK departmanının ortalaması 42500'dür, Ece bu ortalamayı geçer)._||

#### 5. Subquery in FROM / Derived Table (FROM İçinde Alt Sorgu)

Amacımız, departman isimlerini ve o departmanın ortalama maaşını yan yana yazdırmaktır. Önce FROM içerisinde bir alt sorgu yazarak departmanlara göre ortalama maaşları hesaplayıp `ort_tablo` adında sanal/türetilmiş bir tablo oluştururuz. Ardından bu sanal tabloyu ana `departmanlar` tablosuyla birleştiririz (JOIN).

**Sorgu:**

SQL

```
SELECT 
    d.departman_adi, 
    ort_tablo.ortalama_maas
FROM departmanlar AS d
JOIN (
    SELECT departman_id, AVG(maas) AS ortalama_maas
    FROM calisanlar
    GROUP BY departman_id
) AS ort_tablo 
    ON d.departman_id = ort_tablo.departman_id;
```

**Çıktı:**

|**departman_adi**|**ortalama_maas**|
|---|---|
|IT|55000.00|
|İK|42500.00|
### 3: Ortak Tablo İfadeleri (Common Table Expressions)

#### 1. Kavramlar ve Tanımlar

- **Common Table Expression (CTE):** Bir sorgunun başlangıcında WITH komutu ile tanımlanan, sorgunun okunabilirliğini artıran ve ana sorgu içerisinde normal bir tablo gibi referans gösterilebilen adlandırılmış geçici bir sonuç kümesidir.
    
- **WITH clause:** Ana sorgudan önce bir veya daha fazla CTE tanımlamak için kullanılan, sıralı ve adlandırılmış alt sorgulara izin vererek karmaşık mantığın çok daha kolay düzenlenmesini sağlayan SQL sözdizimidir.
    

#### 2. Senaryo ve Tablo Yapısı

Alt sorguların (subqueries) karmaşıklaştığı durumlarda kodun daha düzenli okunmasını sağlamak için CTE kullanılır. Satış personellerinin yaptığı günlük işlem tutarlarını içeren bir tablomuz bulunmaktadır. Amacımız, personellerin toplam satışlarını hesaplamak ve sadece toplam satışı belirli bir hedefin (örneğin 15.000 TL) üzerinde olan "Yıldız Personelleri" listelemektir.

**Tablo: `gunluk_satislar`**

|**islem_id**|**personel_isim**|**islem_tutari**|
|---|---|---|
|1|Burak|8000|
|2|Burak|9000|
|3|Elif|5000|
|4|Elif|6000|
|5|Tarık|20000|

#### 3. CTE Kullanımı (WITH Clause)

Önce `WITH` ifadesiyle `ToplamSatislar` adında geçici bir tablo (CTE) oluşturulur. Bu yapı, ana sorgu (SELECT) çalışmadan hemen önce hafızada hesaplanır. Ardından gelen ana sorgu, sanki veritabanında gerçekten `ToplamSatislar` adında bir tablo varmış gibi bu geçici veriyi kullanır.

**Sorgu:**

SQL

```
WITH ToplamSatislar AS (
    SELECT 
        personel_isim, 
        SUM(islem_tutari) AS toplam_tutar
    FROM gunluk_satislar
    GROUP BY personel_isim
)
SELECT 
    personel_isim, 
    toplam_tutar
FROM ToplamSatislar
WHERE toplam_tutar > 15000;
```

#### 4. Çıktı (Output)

|**personel_isim**|**toplam_tutar**|
|---|---|
|Burak|17000|
|Tarık|20000|
|_(Elif'in toplam satışı 11.000 TL olduğu için CTE üzerinden yapılan WHERE filtresine takılmış ve listelenmemiştir)._||

### 4: Toplulaştırma ve Gruplama (Aggregation)

#### 1. Kavramlar ve Tanımlar

- **Aggregate function (Toplulaştırma Fonksiyonu):** COUNT, SUM, AVG, MIN veya MAX gibi, bir dizi satır üzerinden tek bir özet değer hesaplayan ve genellikle GROUP BY ile veya pencere fonksiyonları (window functions) içinde kullanılan bir fonksiyondur.
    
- **GROUP BY:** Toplulaştırma fonksiyonlarının grup başına özet istatistikler hesaplayabilmesi için belirtilen sütunlarda aynı değerleri paylaşan satırları gruplandıran bir ifadedir.
    
- **ROUND:** Toplanan veya ortalaması alınan sonuçların okunabilirliğini artırmak için sayısal bir ifadeyi belirtilen ondalık basamak sayısına yuvarlayan sayısal bir fonksiyondur.
    

#### 2. Senaryo ve Tablo Yapısı

Farklı makine öğrenmesi modellerinin eğitim sürelerini (saniye cinsinden ondalıklı olarak) tutan bir log tablomuz bulunmaktadır. Amacımız, her bir model tipinin ortalama eğitim süresini hesaplamak ve bu sonucu daha okunaklı olması için virgülden sonra iki basamak olacak şekilde yuvarlamaktır.

**Tablo: `model_egitim_loglari`**

|**log_id**|**model_tipi**|**egitim_suresi_sn**|
|---|---|---|
|1|SVM|45.456|
|2|SVM|50.123|
|3|MLP|120.789|
|4|MLP|125.456|
|5|GNN|300.123|

#### 3. Sorgu (Query)

`GROUP BY` ifadesi ile aynı model tipine sahip satırları bir araya getiriyoruz. `AVG` (ortalama) toplulaştırma fonksiyonunu kullanarak bu grupların ortalamasını alıyoruz. Son olarak, çıkan karmaşık ondalıklı sayıyı `ROUND` fonksiyonu ile iki basamağa sabitliyoruz.

SQL

```
SELECT 
    model_tipi, 
    ROUND(AVG(egitim_suresi_sn), 2) AS ort_egitim_suresi
FROM model_egitim_loglari
GROUP BY model_tipi;
```

#### 4. Çıktı (Output)

|**model_tipi**|**ort_egitim_suresi**|
|---|---|
|SVM|47.79|
|MLP|123.12|
|GNN|300.12|
|_(Aynı model tipine sahip satırlar tek bir satıra indirgendi ve süre ortalamaları başarıyla yuvarlandı)._||


### 5: Pencere Fonksiyonları Temelleri (Window Functions)

#### 1. Kavramlar ve Tanımlar

- **Window function (Pencere Fonksiyonu):** Satırları daraltmadan (collapsing rows) geçerli satırla ilişkili bir dizi satır ("pencere") üzerinde hesaplamalar yapan ve hareketli toplamlar, sıralamalar ve hareketli ortalamalar sağlayan bir fonksiyondur.
    
- **OVER clause:** Pencere fonksiyonları için gerekli olan ve fonksiyonun üzerinde çalışacağı pencereyi tanımlayan, ORDER BY ve PARTITION BY özelliklerini içerebilen ifadedir.
    
- **PARTITION BY:** Sonuç kümesini gruplara (bölümlere) ayıran ve böylece pencere fonksiyonunun her bir bölüm için ayrı değerler hesaplamasını sağlayan bir OVER ifadesi seçeneğidir.
    
- **RANK:** Bağlı (eşit) değerlerin aynı sırayı aldığı ve numaralandırmada boşlukların (gaps in numbering) oluştuğu, bir veya daha fazla sütunun sıralamasına dayalı olarak bir bölümdeki her satıra bir sıra numarası atayan pencere fonksiyonudur.
    

#### 2. Senaryo ve Tablo Yapısı

Farklı problem alanlarında (domain) eğitilmiş modellerin doğruluk oranlarını (accuracy) tutan bir tablomuz bulunmaktadır. Amacımız, satırları GROUP BY ile tek bir satıra indirgemeden (daraltmadan), her bir modelin **kendi alanı (domain) içerisinde** kaçıncı sırada olduğunu bulmaktır.

**Tablo: `model_sonuclari`**

|**model_id**|**domain**|**algoritma**|**dogruluk_orani**|
|---|---|---|---|
|1|Görüntü (Vision)|ResNet|0.88|
|2|Görüntü (Vision)|YOLO|0.88|
|3|Görüntü (Vision)|CNN|0.85|
|4|Metin (NLP)|GPT|0.95|
|5|Metin (NLP)|BERT|0.92|

#### 3. Sorgu (Query)

`PARTITION BY domain` ifadesi ile modelleri kullanım alanlarına göre sanal pencerelere ayırıyoruz. `ORDER BY dogruluk_orani DESC` ile her pencerenin kendi içinde en yüksek skordan en düşüğe doğru sıralanmasını sağlıyoruz ve `RANK()` fonksiyonu ile bu sıralamaya bir numara atıyoruz.

SQL

```
SELECT 
    domain,
    algoritma,
    dogruluk_orani,
    RANK() OVER(PARTITION BY domain ORDER BY dogruluk_orani DESC) AS basari_sirasi
FROM model_sonuclari;
```

#### 4. Çıktı (Output)

|**domain**|**algoritma**|**dogruluk_orani**|**basari_sirasi**|
|---|---|---|---|
|Görüntü (Vision)|ResNet|0.88|1|
|Görüntü (Vision)|YOLO|0.88|1|
|Görüntü (Vision)|CNN|0.85|3|
|Metin (NLP)|GPT|0.95|1|
|Metin (NLP)|BERT|0.92|2|
|_(Görüntü alanında ResNet ve YOLO aynı doğruluk oranına (0.88) sahip olduğu için ikisi de 1. sırayı aldı. RANK fonksiyonunun özelliği gereği numaralandırmada boşluk bırakılarak CNN modeline 2 yerine 3. sıra atandı. Metin alanı ise kendi içinde ayrı olarak sıralandı)._||||



### 6: İleri Düzey Pencere Fonksiyonları (Advanced Window Functions)

#### 1. Kavramlar ve Tanımlar

- **Sliding window (Kayan Pencere):** Geçerli satıra göre hareket eden bir çerçeve (`ROWS BETWEEN` ile tanımlanır) kullanarak değerleri hesaplayan ve genellikle çalışan toplamlar (running totals) ile hareketli ortalamalar (moving averages) için kullanılan bir pencere fonksiyonu desenidir.
    
- **Window frame (Pencere Çerçevesi / ROWS BETWEEN):** Kayan hesaplamalar (sliding calculations) için pencereyi sınırlamak amacıyla kullanılan ve geçerli satıra göre başlangıç ve bitiş satırlarını (örneğin, `UNBOUNDED PRECEDING`, `1 PRECEDING`, `CURRENT ROW`) tanımlayan, `OVER` içindeki bir belirtimdir.
    

#### 2. Senaryo ve Tablo Yapısı

Bir API servisine gün gün yapılan istek sayılarını (request count) tutan bir tablomuz bulunmaktadır. Amacımız, her gün için o güne kadar yapılan **kümülatif toplam istek sayısını (running total)** ve son 2 günün **hareketli ortalamasını (moving average)** hesaplamaktır.

**Tablo: `api_kullanim_loglari`**

|**tarih**|**gunluk_istek**|
|---|---|
|2026-04-20|100|
|2026-04-21|150|
|2026-04-22|200|
|2026-04-23|250|

#### 3. Sorgu (Query)

İlk pencere fonksiyonunda (`kumulatif_toplam`), tablonun en başından (`UNBOUNDED PRECEDING`) mevcut satıra (`CURRENT ROW`) kadar olan tüm satırları toplayarak ilerliyoruz.

İkinci pencere fonksiyonunda (`hareketli_ortalama`), sadece bir önceki satırdan (`1 PRECEDING`) mevcut satıra (`CURRENT ROW`) kadar olan 2 günlük kayan dar bir pencerenin ortalamasını alıyoruz.

SQL

```
SELECT 
    tarih, 
    gunluk_istek,
    SUM(gunluk_istek) OVER(
        ORDER BY tarih 
        ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW
    ) AS kumulatif_toplam,
    AVG(gunluk_istek) OVER(
        ORDER BY tarih 
        ROWS BETWEEN 1 PRECEDING AND CURRENT ROW
    ) AS hareketli_ortalama
FROM api_kullanim_loglari;
```

#### 4. Çıktı (Output)

|**tarih**|**gunluk_istek**|**kumulatif_toplam**|**hareketli_ortalama**|
|---|---|---|---|
|2026-04-20|100|100|100.0|
|2026-04-21|150|250|125.0|
|2026-04-22|200|450|175.0|
|2026-04-23|250|700|225.0|

_(İlk günün kendisinden önceki bir gün olmadığı için hareketli ortalaması kendi değeridir. 21 Nisan'ın ortalaması (100+150)/2 = 125; 22 Nisan'ın ortalaması (150+200)/2 = 175 olarak hesaplanır ve bu 2 günlük dar pencere aşağıya doğru kayarak devam eder)._



###  7: Ekstra Veri ve Analitik Kavramları

#### 1. Kavramlar ve Tanımlar

- **SQL:** SELECT, INSERT, UPDATE ve DELETE gibi bildirimsel ifadeler (declarative statements) yazarak ilişkisel veritabanlarında depolanan verileri sorgulamak, değiştirmek ve yönetmek için kullanılan alana özgü (domain-specific) bir dildir.
    
- **Homogeneity (in arrays) (Dizilerde Homojenlik):** Bir NumPy dizisindeki tüm elemanların aynı veri tipini (data type) paylaşması özelliğidir; bu durum eleman başına düşen işlem yükünü azaltır ve hız ile bellek verimliliği için bitişik (contiguous) bellek düzenlerine olanak tanır.
    
- **Knowledge base (FAQ) (Bilgi Tabanı):** Yapay zeka temsilcilerinin (agents) doğru, güncel yanıtlar sağlamak amacıyla (genellikle bir arama / lookup aracı aracılığıyla) başvurabileceği, alana özgü yapılandırılmış bilgi veya soru-cevap çiftleri kümesidir.
    

#### 2. Senaryo ve Tablo Yapısı

Şirket içindeki geliştiricilerin (developer) yararlandığı bir Sıkça Sorulan Sorular (FAQ) bilgi tabanı (Knowledge Base) veritabanımız bulunmaktadır. SQL sütunları da tıpkı dizilerdeki (arrays) homojenlik (homogeneity) prensibinde olduğu gibi katı veri tiplerine sahiptir; örneğin `id` sütunu sadece tam sayı (integer), `kategori` sütunu sadece metin (string) alabilir.

Amacımız, bu bilgi tabanını sorgulayarak veri yapıları ve veritabanı ile ilgili kritik kavramların tanımlarını çekmektir.

**Tablo: `knowledge_base_faq`**

|**faq_id**|**kavram**|**aciklama**|**kategori**|
|---|---|---|---|
|1|SQL|İlişkisel veritabanlarını yönetmek için kullanılan dildir.|Veritabanı|
|2|Homogeneity|Tüm elemanların aynı veri tipini paylaşması özelliğidir.|Veri Yapıları|
|3|CTE|Adlandırılmış geçici sonuç kümesidir.|Veritabanı|

#### 3. Sorgu (Query)

Bu tablo üzerinden SQL dilini kullanarak, spesifik olarak veri yapılarıyla ilgili olan veya adı doğrudan 'SQL' olan kayıtları filtreleyip listeliyoruz.

SQL

```
SELECT 
    kavram, 
    aciklama
FROM knowledge_base_faq
WHERE kategori = 'Veri Yapıları' 
   OR kavram = 'SQL';
```

#### 4. Çıktı (Output)

|**kavram**|**aciklama**|
|---|---|
|SQL|İlişkisel veritabanlarını yönetmek için kullanılan dildir.|
|Homogeneity|Tüm elemanların aynı veri tipini paylaşması özelliğidir.|
