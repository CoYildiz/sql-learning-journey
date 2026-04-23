# 3 - Joining Data

### 1: Veritabanı Temelleri ve İlişki Türleri

#### 1. Veritabanı Temelleri

- **Table (Tablo):** Verileri satırlar ve sütunlar halinde depolayan veritabanı nesnesidir; burada her satır bir kayıt ve her sütun belirli bir veri tipine sahip bir alandır.
    
- **Column (Sütun):** Bir tablodaki isimlendirilmiş bir alandır; her kayıt için belirli bir veri tipinde değerler tutar ve genellikle koşullar, seçim ve toplama işlemleri için kullanılır.
    
- **Key / Primary Key (Anahtar / Birincil Anahtar):** Tablodaki her bir satırı benzersiz şekilde tanımlayan, benzersizliği sağlamak ve tabloları güvenilir bir şekilde birleştirmek için kullanılan sütun veya sütun kümesidir.
    

**Örnek Tablo Yapısı:**

Aşağıdaki `kullanicilar` tablosunda `kullanici_id` bir Primary Key'dir (Birincil Anahtar) ve her satırı benzersiz kılar. `isim` ve `yas` ise birer sütundur (Column).

|**kullanici_id (Primary Key)**|**isim (Column)**|**yas (Column)**|
|---|---|---|
|1|Can|25|
|2|Ayşe|30|

#### 2. İlişki Türleri (Relationships)

- **One-to-one relationship (Bire-bir ilişki):** Bir tablodaki her bir kaydın, diğer tablodaki tam olarak bir kayda karşılık geldiği, iki tablo arasında eşleştirilmiş benzersizlik anlamına gelen ilişkidir.
    
- **One-to-many relationship (Bire-çok ilişki):** Bir tablodaki tek bir kaydın, diğer tablodaki birden fazla kayıtla (örneğin bir yazar ve yazdığı kitaplar gibi) ilişkilendirildiği yapıdır.
    
- **Many-to-many relationship (Çoka-çok ilişki):** Bir tablodaki birden fazla kaydın diğer tablodaki birden fazla kayıtla ilişkili olduğu durumdur; genellikle iki tabloyu bağlayan bir "join tablosu" (ara tablo) ile temsil edilir.
    

**Senaryo ve Query Örneği (One-to-many / Bire-çok İlişki):**

Bir yazarın birden fazla kitabı olabilir. `yazarlar` tablosundaki bir kayıt, `kitaplar` tablosundaki birden fazla kayıtla eşleşebilir.

**Tablo 1: `yazarlar`**

|**yazar_id (Primary Key)**|**yazar_isim**|
|---|---|
|10|Sabahattin Ali|
|20|Orhan Pamuk|

**Tablo 2: `kitaplar`**

|**kitap_id (Primary Key)**|**kitap_adi**|**yazar_id (Foreign Key)**|
|---|---|---|
|101|Kürk Mantolu Madonna|10|
|102|İçimizdeki Şeytan|10|
|103|Kar|20|

**Sorgu (Query):**

Belirli bir yazarın tüm kitaplarını getiren SQL sorgusu:

SQL

```
SELECT 
    y.yazar_isim, 
    k.kitap_adi
FROM yazarlar AS y
JOIN kitaplar AS k 
    ON y.yazar_id = k.yazar_id
WHERE y.yazar_id = 10;
```

**Sorgu Çıktısı (Output):**

|**yazar_isim**|**kitap_adi**|
|---|---|
|Sabahattin Ali|Kürk Mantolu Madonna|
|Sabahattin Ali|İçimizdeki Şeytan|


-**Senaryo ve Query Örneği Many-to-many relationship (Çoka-çok ilişki):**
**Senaryo ve Tablo Yapısı:**

Öğrenciler ve kayıt oldukları dersler arasındaki ilişki çoka-çok bir ilişkidir. Bir öğrenci birden fazla derse kayıt olabilir, bir dersi de birden fazla öğrenci alabilir. Bu ilişkiyi veritabanında kurabilmek için `ogrenciler` ve `dersler` tablolarını birbirine bağlayan üçüncü bir ara tabloya (`ogrenci_ders_kayitlari`) ihtiyaç vardır.

**Tablo 1: `ogrenciler`**

|**ogrenci_id (Primary Key)**|**ogrenci_isim**|
|---|---|
|1|Efe|
|2|Naz|

**Tablo 2: `dersler`**

|**ders_id (Primary Key)**|**ders_adi**|
|---|---|
|101|Matematik|
|102|Fizik|

**Tablo 3 (Join Tablosu): `ogrenci_ders_kayitlari`**

Bu tablo, hangi öğrencinin hangi dersi aldığını eşleştiren yabancı anahtarları (Foreign Key) barındırır.

|**kayit_id (Primary Key)**|**ogrenci_id (Foreign Key)**|**ders_id (Foreign Key)**|
|---|---|---|
|50|1|101|
|51|1|102|
|52|2|101|

**Sorgu (Query):**

Hangi öğrencinin hangi dersleri aldığını listeleyen, üç tabloyu birbirine bağlayan SQL sorgusu:

SQL

```
SELECT 
    o.ogrenci_isim, 
    d.ders_adi
FROM ogrenciler AS o
JOIN ogrenci_ders_kayitlari AS k 
    ON o.ogrenci_id = k.ogrenci_id
JOIN dersler AS d 
    ON k.ders_id = d.ders_id;
```

**Sorgu Çıktısı (Output):**

|**ogrenci_isim**|**ders_adi**|
|---|---|
|Efe|Matematik|
|Efe|Fizik|
|Naz|Matematik|



### 2: Tablo Birleştirmeye Giriş (Join Basics)

#### 1. Temel Mekanizma ve Sorgu Anatomisi

- **Join:** İki veya daha fazla tablodaki sütunları, satırları belirtilen bir koşula göre eşleştirerek birleştiren ve böylece ilişkili verileri birlikte sorgulamanıza olanak tanıyan bir SQL işlemidir.
    
- **FROM clause:** Bir SQL sorgusunda veri seçilecek tabloları veya alt sorguları listeleyen ve birleştirmeler (joins) için sol/sağ bağlamını tanımlayan bölümdür.
    
- **ON clause:** JOIN ifadesinin bir parçası olup, birleştirilen tablolar arasındaki satırları eşleştirmek için kullanılan koşulu (bir veya daha fazla sütun karşılaştırması) belirtir.
    
- **USING clause:** Birleştirme sütunu her iki tabloda da aynı ada sahip olduğunda JOIN işlemi için kullanılan bir kısayoldur; bu sütunu tablo adlarıyla belirtmek yerine parantez içinde bir kez listelemene olanak tanır.
    
- **Alias (Tablo veya Sütun Takma Adı):** Referansları kısaltmak veya çıktının okunabilirliğini artırmak için AS kelimesi kullanılarak (veya sadece boşluk bırakılarak) bir tabloya veya sütuna verilen geçici isimdir.
    

#### 2. Senaryo ve Tablo Yapısı

Bir e-ticaret veritabanında ürünleri ve bu ürünlerin ait olduğu kategorileri birleştirmek istiyoruz. Her iki tabloda da ortak olan bağlantı sütunu `kategori_id` sütunudur.

**Tablo 1: `kategoriler`**

|**kategori_id (Primary Key)**|**kategori_adi**|
|---|---|
|10|Elektronik|
|20|Giyim|

**Tablo 2: `urunler`**

|**urun_id (Primary Key)**|**urun_adi**|**kategori_id (Foreign Key)**|**fiyat**|
|---|---|---|---|
|101|Laptop|10|25000|
|102|T-Shirt|20|300|

#### 3. Query Örneği (ON Clause ve Alias Kullanımı)

Bu sorguda tablolar `ON` ifadesiyle bağlanmakta ve `Alias` kullanılarak (`k` ve `u`) isimler kısaltılmaktadır.

SQL

```
SELECT 
    u.urun_adi, 
    k.kategori_adi, 
    u.fiyat
FROM urunler AS u
JOIN kategoriler AS k 
    ON u.kategori_id = k.kategori_id;
```

#### 4. Query Örneği (USING Clause Kullanımı)

Eğer birleştirme yapılacak sütun adı her iki tabloda da tamamen aynıysa (`kategori_id`), `ON` yerine `USING` kısayolu kullanılabilir.

SQL

```
SELECT 
    u.urun_adi, 
    k.kategori_adi, 
    u.fiyat
FROM urunler AS u
JOIN kategoriler AS k 
    USING (kategori_id);
```

#### 5. Sorgu Çıktısı (Output)

Her iki sorgunun da üreteceği ortak çıktı tablosu aşağıdaki gibidir:

|**urun_adi**|**kategori_adi**|**fiyat**|
|---|---|---|
|Laptop|Elektronik|25000|
|T-Shirt|Giyim|300|

### 3: Yönlü ve Özel Birleştirmeler (Advanced Joins)

#### 1. Yönlü ve Özel Birleştirme Türleri

- **LEFT JOIN:** Sol tablodaki tüm satırları ve sağ tablodaki eşleşen satırları döndüren, eşleşme bulunmadığında ise sağ tablo sütunları için NULL değerlerini kullanan birleştirme işlemidir.
    
- **RIGHT JOIN:** Sağ tablodaki tüm satırları ve sol tablodaki eşleşen satırları döndüren, eşleşme olmadığında sol tablo sütunları için NULL değerlerini kullanan birleştirme işlemidir.
    
- **CROSS JOIN:** İki tablonun Kartezyen çarpımını üreten, birinci ve ikinci tablodaki satırların her olası kombinasyonunu döndüren birleştirme işlemidir.
    
- **SELF JOIN:** Bir tablonun, aynı tablo içindeki satırları karşılaştırmak veya ilişkilendirmek amacıyla genellikle takma adlar (aliases) kullanılarak kendisiyle birleştirildiği işlemdir.
    

#### 2. Senaryo ve Tablo Yapısı

Veritabanımızda şirket çalışanlarını ve ait oldukları departmanları tutan iki tablo bulunmaktadır. "Finans" departmanının henüz çalışanı yoktur ve "Cem" isimli çalışan henüz bir departmana atanmamıştır.

**Tablo 1: `calisanlar`**

|**calisan_id**|**isim**|**departman_id**|**yonetici_id**|
|---|---|---|---|
|1|Aslı|10|3|
|2|Berk|20|3|
|3|Cem|NULL|NULL|

**Tablo 2: `departmanlar`**

|**departman_id**|**departman_adi**|
|---|---|
|10|IT|
|20|İK|
|30|Finans|

#### 3. LEFT JOIN (Sol Birleştirme)

Amacımız şirketteki tüm çalışanları listelemek ve eğer bir departmanları varsa yanına yazdırmaktır.

**Sorgu:**

SQL

```
SELECT 
    c.isim, 
    d.departman_adi
FROM calisanlar AS c
LEFT JOIN departmanlar AS d 
    ON c.departman_id = d.departman_id;
```

**Çıktı:**

|**isim**|**departman_adi**|
|---|---|
|Aslı|IT|
|Berk|İK|
|Cem|NULL|

#### 4. RIGHT JOIN (Sağ Birleştirme)

Amacımız şirketteki tüm departmanları listelemek ve içindeki çalışanları görmektir.

**Sorgu:**

SQL

```
SELECT 
    c.isim, 
    d.departman_adi
FROM calisanlar AS c
RIGHT JOIN departmanlar AS d 
    ON c.departman_id = d.departman_id;
```

**Çıktı:**

|**isim**|**departman_adi**|
|---|---|
|Aslı|IT|
|Berk|İK|
|NULL|Finans|

#### 5. CROSS JOIN (Çapraz Birleştirme)

İki tablonun her bir satırını birbiriyle eşleştirerek tüm olası kombinasyonları oluşturmak istediğimizde kullanılır.

**Tablo A: `renkler`**

|**renk**|
|---|
|Siyah|
|Beyaz|

**Tablo B: `bedenler`**

|**beden**|
|---|
|M|
|L|

**Sorgu:**

SQL

```
SELECT 
    r.renk, 
    b.beden
FROM renkler AS r
CROSS JOIN bedenler AS b;
```

**Çıktı:**

|**renk**|**beden**|
|---|---|
|Siyah|M|
|Siyah|L|
|Beyaz|M|
|Beyaz|L|

#### 6. SELF JOIN (Öz Birleştirme)

Amacımız, `calisanlar` tablosunu kendisiyle birleştirerek her çalışanın isminin yanına kendi yöneticisinin ismini yazdırmaktır.

**Sorgu:**

SQL

```
SELECT 
    c.isim AS calisan_ismi, 
    y.isim AS yonetici_ismi
FROM calisanlar AS c
JOIN calisanlar AS y 
    ON c.yonetici_id = y.calisan_id;
```

**Çıktı:**

|**calisan_ismi**|**yonetici_ismi**|
|---|---|
|Aslı|Cem|
|Berk|Cem|


###  4: Küme İşlemleri (Set Operations)

#### 1. Küme İşlemleri ve Türleri

- **Set operation (Küme İşlemi):** Anahtarlar aracılığıyla sütunları birleştirmek yerine, iki sorgudan elde edilen tüm sonuç kümelerini (satırları) birleştiren bir SQL işlemidir.
    
- **UNION:** İki sorgunun sonuçlarını üst üste ekleyen ve kopyaları (tekrar edenleri) kaldırarak herhangi bir sonuç kümesinde bulunan tüm farklı satırları döndüren bir küme işlemidir.
    
- **UNION ALL:** İki sorgunun sonuçlarını kopyalar dahil olmak üzere tüm satırları döndürecek şekilde üst üste ekleyen ve tekrarlanan kayıtları koruyan bir küme işlemidir.
    
- **INTERSECT:** Yalnızca iki sorgunun sonuç kümesinde de ortak olarak bulunan satırları döndüren ve kopyaları kaldıran bir küme işlemidir.
    
- **EXCEPT:** İlk sorgudan gelen ancak ikinci sorguda görünmeyen satırları döndüren , bir sonuç kümesini diğerinden etkili bir şekilde çıkaran bir küme işlemidir.
    

#### 2. Senaryo ve Tablo Yapısı

Aynı sütun yapısına sahip iki farklı müşteri listesini karşılaştırdığımızı varsayalım. `kampanya_a` tablosunda A kampanyasına katılan müşteriler, `kampanya_b` tablosunda ise B kampanyasına katılan müşteriler yer almaktadır. Bazı müşteriler her iki kampanyaya da katılmış olabilir.

**Tablo 1: `kampanya_a`**

|**musteri_id**|**isim**|
|---|---|
|10|Ali|
|20|Veli|
|30|Can|

**Tablo 2: `kampanya_b`**

|**musteri_id**|**isim**|
|---|---|
|20|Veli|
|30|Can|
|40|Ece|

#### 3. UNION (Benzersiz Tüm Kayıtları Birleştirme)

Her iki kampanyaya katılan tüm müşterileri, tekrar edenleri tekilleştirerek listeler.

**Sorgu:**

SQL

```
SELECT musteri_id, isim FROM kampanya_a
UNION
SELECT musteri_id, isim FROM kampanya_b;
```

**Çıktı:**

|**musteri_id**|**isim**|
|---|---|
|10|Ali|
|20|Veli|
|30|Can|
|40|Ece|

#### 4. UNION ALL (Tekrarları Koruyarak Birleştirme)

Hangi kampanyadan kaç adet katılım olduğunu görmek için mükerrer kayıtları silmeden alt alta ekler.

**Sorgu:**

SQL

```
SELECT musteri_id, isim FROM kampanya_a
UNION ALL
SELECT musteri_id, isim FROM kampanya_b;
```

**Çıktı:**

|**musteri_id**|**isim**|
|---|---|
|10|Ali|
|20|Veli|
|30|Can|
|20|Veli|
|30|Can|
|40|Ece|

#### 5. INTERSECT (Kesişimleri Bulma)

Yalnızca her iki kampanyaya birden katılmış olan ortak müşterileri listeler.

**Sorgu:**

SQL

```
SELECT musteri_id, isim FROM kampanya_a
INTERSECT
SELECT musteri_id, isim FROM kampanya_b;
```

**Çıktı:**

|**musteri_id**|**isim**|
|---|---|
|20|Veli|
|30|Can|

#### 6. EXCEPT (Farkı Bulma)

Sadece A kampanyasına katılıp, B kampanyasına katılmayan (A'da olup B'de olmayan) müşterileri listeler.

**Sorgu:**

SQL

```
SELECT musteri_id, isim FROM kampanya_a
EXCEPT
SELECT musteri_id, isim FROM kampanya_b;
```

**Çıktı:**

|**musteri_id**|**isim**|
|---|---|
|10|Ali|

###  5: Alt Sorgular ve Gelişmiş Filtreleme (Subqueries & Filtering)

#### 1. Temel Filtreleme ve Tekilleştirme

- **WHERE clause:** Belirtilen koşullara göre satırları filtreleyen ve alt sorgular (subqueries) aracılığıyla semi join (yarı birleştirme) ile anti join işlemlerini uygulamak için yaygın olarak kullanılan bir SQL sorgusu bölümüdür.

- **DISTINCT:** Sorgu sonucundan tekrar eden kopyaları kaldıran ve yalnızca benzersiz (unique) kayıtları döndüren anahtar kelimedir.

#### 2. Alt Sorgular ve Desenler

- **Subquery (Alt Sorgu):** Dış sorgu tarafından kullanılan değerleri veya geçici bir tabloyu sağlayan, başka bir SQL ifadesinin (SELECT, FROM veya WHERE içinde) içine yerleştirilmiş sorgudur.
    
- **Semi join (Yarı Birleştirme):** İkinci tablodan sütunlar eklemeden, ilk tablodan yalnızca ikinci tabloda eşleşen değerlere sahip satırları döndüren (genellikle `WHERE IN` veya `EXISTS` kullanılarak uygulanan) sorgu desenidir.
    
- **Anti join:** İkinci tabloda eşleşen değerleri olmayan ilk tablodaki satırları döndüren (genellikle `WHERE NOT IN` veya `NOT EXISTS` ile uygulanan) sorgu desenidir.
    

#### 3. Senaryo ve Tablo Yapısı

Bir e-ticaret platformunda kayıtlı müşterilerimiz ve verdikleri siparişlerin tutulduğu iki ayrı tablo bulunmaktadır. Amacımız, alt sorguları kullanarak sipariş veren ve vermeyen müşterileri filtrelemektir.

**Tablo 1: `musteriler`**

|**musteri_id**|**isim**|
|---|---|
|1|Efe|
|2|Naz|
|3|Cem|

**Tablo 2: `siparisler`**

|**siparis_id**|**musteri_id**|**tutar**|
|---|---|---|
|1001|1|250|
|1002|1|150|
|1003|2|300|
|_(Not: Cem'in henüz hiçbir siparişi yoktur, Efe ise iki kez sipariş vermiştir)._|||

#### 4. Subquery ve Semi Join (WHERE IN)

Amacımız, en az bir sipariş vermiş olan aktif müşterilerin isimlerini bulmaktır. İç içe geçmiş (nested) bir sorgu yazarak, önce siparişi olan ID'leri bulup ana tabloyu bu ID'lere göre filtreleriz.

**Sorgu:**

SQL

```
SELECT isim 
FROM musteriler
WHERE musteri_id IN (
    SELECT musteri_id 
    FROM siparisler
);
```

**Çıktı:**

|**isim**|
|---|
|Efe|
|Naz|
|_(İkinci tablodan herhangi bir sütun eklenmedi, sadece eşleşen müşteriler listelendi. Efe iki sipariş vermesine rağmen eşleşme mantığı gereği tekil olarak döndü)._|

#### 5. Anti Join (WHERE NOT IN)

Amacımız, sisteme kayıt olmuş ancak henüz hiçbir sipariş vermemiş olan pasif müşterileri bulmaktır. İlk tablodan, ikinci tabloda var olan kayıtları dışlarız.

**Sorgu:**

SQL

```
SELECT isim 
FROM musteriler
WHERE musteri_id NOT IN (
    SELECT musteri_id 
    FROM siparisler
);
```

**Çıktı:**

|**isim**|
|---|
|Cem|
|_(Alt sorgu 1 ve 2 ID'lerini getirdi. `NOT IN` koşulu, bu listede olmayan 3 ID'li Cem'i sonuç olarak döndürdü)._|

#### 6. DISTINCT Kullanımı

Amacımız, `siparisler` tablosuna doğrudan bakarak hangi müşteri ID'lerinin işlem yaptığını tekil olarak listelemektir. Efe'nin (1 numaralı ID) mükerrer kaydını tekilleştireceğiz.

**Sorgu:**

SQL

```
SELECT DISTINCT musteri_id 
FROM siparisler;
```

**Çıktı:**

|**musteri_id**|
|---|
|1|
|2|


###  6: Analitik ve Veri Kültürü

#### 1. Kavramlar ve Tanımlar

- **Data leader (Veri Lideri):** Resmi unvanı ne olursa olsun, verilerin kullanımını savunan, veri odaklı davranışlara örnek olan ve başkalarının verileri etkili bir şekilde kullanmayı öğrenmesine yardımcı olan herhangi bir kişidir.
    
- **LOWESS:** Tanısal grafiklerdeki kalıntı (residual) desenlerini görselleştirmek için sıkça kullanılan, doğrusal olmayan eğilimleri ortaya çıkarmak adına noktalar arasından pürüzsüz bir eğri geçiren, yerel olarak ağırlıklandırılmış bir dağılım grafiği düzleştirme tekniğidir.
    

#### 2. Senaryo ve Tablo Yapısı

SQL doğrudan LOWESS grafikleri çizemez; ancak bir **Data Leader** (Veri Lideri), analitik ekiplerinin Python veya R gibi araçlarda LOWESS eğrisi çizebilmesi için gerekli olan ham veriyi SQL kullanarak hazırlar. Amacımız, algoritmaların eğitim süresi ile başarı oranları arasındaki doğrusal olmayan (non-linear) ilişkiyi incelemektir.

**Tablo 1: `model_performanslari`**

|**model_id**|**egitim_suresi_sn**|**basari_orani**|
|---|---|---|
|1|45.5|0.82|
|2|120.0|0.89|
|3|300.5|0.91|
|4|800.0|0.92|

#### 3. Veri Lideri İçin Hazırlık Sorgusu (Query)

Eğitim süresi çok kısa olan hatalı kayıtları filtreleyerek veri setini görselleştirmeye (LOWESS analizi için dışa aktarmaya) hazır hale getiren sorgudur.

**Sorgu:**

SQL

```
SELECT 
    model_id, 
    egitim_suresi_sn, 
    basari_orani
FROM model_performanslari
WHERE egitim_suresi_sn > 10.0
ORDER BY egitim_suresi_sn ASC;
```

**Çıktı:**

|**model_id**|**egitim_suresi_sn**|**basari_orani**|
|---|---|---|
|1|45.5|0.82|
|2|120.0|0.89|
|3|300.5|0.91|
|4|800.0|0.92|


