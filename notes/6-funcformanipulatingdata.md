# 6 Veri Manipulasyonu

### 1: Veri Tipleri ve Tablo Yönetimi (Data Types & Table Management)



#### 1. Kavramlar ve Tanımlar

* **INT (integer):** Ondalıklı bileşeni olmayan tam sayıları depolayan sayısal bir veri tipidir.
* **DECIMAL (numeric):** Para birimi ve diğer kesin hesaplamalar için kullanışlı olan, kullanıcı tanımlı hassasiyet ve ölçeğe sahip tam sabit noktalı sayıları depolayan sayısal bir veri tipidir.
* **DATE:** Zaman bileşeni olmadan yalnızca takvim tarihini (yıl, ay, gün) saklayan bir veri tipidir.
* **TIME:** İlişkili bir tarih olmadan yalnızca günün saati değerini (saat, dakika, saniye) saklayan bir veri tipidir.
* **TIMESTAMP:** Mikrosaniye hassasiyetinde hem takvim tarihini hem de günün saatini saklayan, zaman dilimi bilgisiyle veya bilgisi olmadan temsil edilebilen bir veri tipidir.
* **Text data types (CHAR, VARCHAR, TEXT):** Karakter dizelerini depolamak için kullanılan tiplerdir; CHAR sabit uzunluklu, VARCHAR limitli değişken uzunluklu ve TEXT ise efektif olarak limitsiz değişken uzunlukludur.
* **ARRAY:** Herhangi bir desteklenen temel türün sıralı listelerini (çok boyutlu diziler dahil) tutan, 1 tabanlı indeksler ve dizi notasyonu ile erişilen bir kolon tipidir.
* **Casting (CAST/:: operator):** Bir değeri standart `CAST(ifade AS tip)` sözdizimini veya PostgreSQL'in çift sütun (`::`) kısayolunu kullanarak bir veri tipinden diğerine dönüştürme işlemidir.
* **CREATE TABLE:** Bir veritabanında yeni bir tabloyu, sütun adlarını ve veri tiplerini tanımlamak için kullanılan SQL komutudur.
* **INSERT statement:** Bir tabloya yeni satırlar (kayıtlar) ekleyen ve bir veya daha fazla sütun için değerler sağlayan SQL komutudur.

---

#### 2. Senaryo ve Tablo Yapısı

Ağ güvenliği projeniz kapsamında, ağdaki cihazların envanterini ve son görülme zamanlarını takip eden bir sistem tasarlayalım. Bu tablo; cihaz id'si (INT), cihaz adı (VARCHAR), IP adresleri listesi (ARRAY), cihazın maliyeti (DECIMAL) ve sisteme kaydedildiği tarih (DATE) gibi farklı veri tiplerini içerecektir.

**Tablo: `cihaz_envanteri`**

| cihaz_id | cihaz_adi | ip_adresleri | maliyet | kayit_tarihi |
| :--- | :--- | :--- | :--- | :--- |
| 1 | Sunucu_A | {"192.168.1.1", "10.0.0.5"} | 1500.50 | 2026-04-23 |

---

#### 3. Sorgu Örneği (Tablo Oluşturma, Ekleme ve Casting)

Önce tabloyu oluşturuyoruz, ardından veri ekliyoruz. Son olarak, sayısal bir veriyi metne dönüştürerek (casting) raporlama amaçlı kullanıyoruz.

```sql
-- 1. Tablo Oluşturma
CREATE TABLE cihaz_envanteri (
    cihaz_id INT,
    cihaz_adi VARCHAR(50),
    ip_adresleri TEXT[], -- Metin dizisi (Array)
    maliyet DECIMAL(10, 2),
    kayit_tarihi DATE
);

-- 2. Veri Ekleme
INSERT INTO cihaz_envanteri (cihaz_id, cihaz_adi, ip_adresleri, maliyet, kayit_tarihi)
VALUES (1, 'Sunucu_A', ARRAY['192.168.1.1', '10.0.0.5'], 1500.50, '2026-04-23');

-- 3. Veri Sorgulama ve Casting (Tip Dönüştürme)
SELECT 
    cihaz_adi,
    -- PostgreSQL'e özgü :: operatörü ile casting
    maliyet::TEXT || ' TL' AS maliyet_etiketi,
    -- Standart CAST kullanımı
    CAST(kayit_tarihi AS TIMESTAMP) AS detayli_zaman
FROM cihaz_envanteri;
```

---

#### 4. Çıktı (Output)

| cihaz_adi | maliyet_etiketi | detayli_zaman |
| :--- | :--- | :--- |
| Sunucu_A | 1500.50 TL | 2026-04-23 00:00:00 |

*(Bu örnekte `maliyet` sütunu metne dönüştürülüp yanına birim eklenmiş, `DATE` tipi ise `TIMESTAMP` tipine dönüştürülerek saat bilgisiyle birlikte gösterilmiştir).*


### 2: Tarih ve Zaman Fonksiyonları (Date & Time Functions)

---

#### 1. Kavramlar ve Tanımlar

- **NOW() / CURRENT_TIMESTAMP:** Mevcut tarih ve saati zaman dilimi bilgisiyle birlikte `TIMESTAMP` formatında döndüren fonksiyonlardır.
    
- **AGE():** İki zaman damgası (timestamp) arasındaki farkı hesaplayarak sonucu yıl, ay, gün, saat gibi birimlerden oluşan bir `INTERVAL` (zaman aralığı) olarak döndüren fonksiyonudur.
    
- **DATE_TRUNC():** Bir zaman damgasını veya aralığını belirtilen bir hassasiyete (örneğin ayın başlangıcı, yılın başlangıcı veya günün başlangıcı) göre kırparak o periyodun başlangıç değerine yuvarlayan fonksiyondur.
    
- **INTERVAL:** Tarih ve saatler üzerinde toplama veya çıkarma gibi aritmetik işlemler yapılmasına olanak tanıyan, belirli bir zaman dilimini (örneğin '2 months' veya '3 days') temsil eden veri tipidir.
    

---

#### 2. Senaryo ve Tablo Yapısı

Bir sunucunun bakım kayıtlarını ve çalışma sürelerini (uptime) takip ettiğimiz bir senaryoyu ele alalım. Amacımız sunucunun ne kadar süredir çalıştığını hesaplamak, kayıtları ayın başına göre gruplamak ve bir sonraki bakım tarihini belirlemektir.

**Tablo: `sunucu_loglari`**

|**sunucu_id**|**baslatma_zamani**|**son_kontrol_zamani**|
|---|---|---|
|101|2026-01-15 08:00:00|2026-04-20 14:00:00|
|102|2026-03-10 10:30:00|2026-04-23 09:00:00|

---

#### 3. Sorgu (Query)

Bu sorguda sunucuların çalışma süresini `AGE` ile bulacağız, `DATE_TRUNC` ile kayıtları ay bazında normalize edeceğiz ve `INTERVAL` kullanarak 1 yıl sonrasına bir "Garanti Bitiş" tarihi atayacağız.

Query:
```sql
SELECT 
    sunucu_id,
    -- 1. Çalışma süresini hesapla
    AGE(son_kontrol_zamani, baslatma_zamani) AS calisma_suresi,
    -- 2. Kaydı ayın başına yuvarla
    DATE_TRUNC('month', baslatma_zamani) AS baslatma_ayi,
    -- 3. Mevcut zamana 1 yıl ekleyerek garanti tarihi bul
    NOW() + INTERVAL '1 year' AS gelecek_yil_bugun
FROM sunucu_loglari;
```

---

#### 4. Çıktı (Output)

|**sunucu_id**|**calisma_suresi**|**baslatma_ayi**|**gelecek_yil_bugun**|
|---|---|---|---|
|101|3 mons 5 days 06:00:00|2026-01-01 00:00:00|2027-04-23 23:31:00|
|102|1 mon 12 days 22:30:00|2026-03-01 00:00:00|2027-04-23 23:31:00|

- **Analiz:** Sunucu 101 için çalışma süresi tam olarak 3 ay, 5 gün ve 6 saat olarak hesaplanmıştır. `DATE_TRUNC` işlemi 15 Ocak olan tarihi o ayın ilk anına (1 Ocak) çekmiştir. `INTERVAL` kullanımı ise bugünün tarihini tam bir yıl sonrasına başarıyla taşımıştır.


### 3: Metin Manipülasyonu (String Manipulation)

#### 1. Kavramlar ve Tanımlar

- **String concatenation / CONCAT():** İki veya daha fazla metni (string) tek bir metin haline getirme işlemidir; bu işlem `||` operatörü veya birden fazla argüman kabul eden `CONCAT()` fonksiyonu ile gerçekleştirilir.
    
- **TRIM / LTRIM / RTRIM:** Bir metnin başındaki ve/veya sonundaki istenmeyen karakterleri (varsayılan olarak boşluk) kaldıran fonksiyonlardır; `TRIM` her iki ucu, `LTRIM` sadece sol tarafı, `RTRIM` ise sadece sağ tarafı temizler.
    
- **Substring functions (LEFT, RIGHT, SUBSTRING, SUBSTR):** Bir metnin belirli kısımlarını ayıklayan fonksiyonlardır; `LEFT` ve `RIGHT` baştan/sondan belirtilen sayıda karakteri alırken, `SUBSTRING` veya `SUBSTR` başlangıç konumu ve uzunluk verilerek belirli bir parçayı çeker.
    

---

#### 2. Senaryo ve Tablo Yapısı

Ağ trafiğini izleyen bir sistemde, kullanıcıların tam isimlerinin başında/sonunda gereksiz boşluklar olduğunu ve cihaz kodlarının karmaşık formatlarda (örneğin `CIHAZ-123-TR`) saklandığını varsayalım. Bu verileri raporlama için temizlemek ve parçalamak istiyoruz.

**Tablo: `kullanici_kayitlari`**

|**kayit_id**|**tam_isim**|**cihaz_kodu**|
|---|---|---|
|1|" Ahmet Yilmaz "|"CIHAZ-101-TR"|
|2|" Buse Demir"|"CIHAZ-202-EN"|

---

#### 3. Sorgu (Query)

Bu sorguda isimleri temizleyecek, metinleri birleştirecek ve cihaz kodlarından sadece model numarasını ve ülke kodunu ayıklayacağız.

SQL

```sql
SELECT 
    -- 1. Boşlukları temizle ve metin birleştir
    'Kullanıcı: ' || TRIM(tam_isim) AS temiz_isim_etiketi,
    -- 2. Cihaz kodunun başından ilk 5 karakteri al (CIHAZ)
    LEFT(cihaz_kodu, 5) AS cihaz_tipi,
    -- 3. Cihaz kodunun sonundan 2 karakteri al (Ülke Kodu)
    RIGHT(cihaz_kodu, 2) AS ulke_kodu,
    -- 4. Belirli bir konumdan parça al (101, 202 gibi sayısal kısım)
    SUBSTRING(cihaz_kodu FROM 7 FOR 3) AS model_no
FROM kullanici_kayitlari;
```

---

#### 4. Çıktı (Output)

|**temiz_isim_etiketi**|**cihaz_tipi**|**ulke_kodu**|**model_no**|
|---|---|---|---|
|Kullanıcı: Ahmet Yilmaz|CIHAZ|TR|101|
|Kullanıcı: Buse Demir|CIHAZ|EN|202|

- **Analiz:** `TRIM` fonksiyonu isimlerin etrafındaki çift boşlukları başarıyla temizledi. `LEFT` ve `RIGHT` fonksiyonları sabit uzunluktaki kısımları ayıklarken, `SUBSTRING` fonksiyonu 7. karakterden başlayarak 3 karakter uzunluğundaki model numarasını doğru şekilde çekti.


### 4: Desen Eşleştirme ve Arama (Pattern Matching & Search)

---

#### 1. Kavramlar ve Tanımlar

- **LIKE operator and wildcards:** Dizeler için bir desen eşleştirme operatörüdür; `%` karakteri sıfır veya daha fazla karakterle eşleşirken, `_` karakteri tam olarak bir karakterle eşleşir ve eşleşme varsayılan olarak büyük/küçük harfe duyarlıdır.
    
- **Full-text search (to_tsvector, to_tsquery, tsvector, lexeme):** Metni normalleştirilmiş aranabilir belirteçlere (`tsvector` ve `lexemes`) dönüştüren ve kök bulma (stemming), sıralama ile büyük/küçük harf duyarsız eşleştirme kullanarak doğal dil sorguları gerçekleştiren bir özelliktir.
    

---

#### 2. Senaryo ve Tablo Yapısı

Ağ güvenliği ve sızma tespiti (Intrusion Detection) çalışmalarında, sistem loglarındaki belirli hata mesajlarını veya şüpheli aktiviteleri bulmak için gelişmiş arama teknikleri kullanılır. Basit aramalar için `LIKE`, daha karmaşık ve dil bazlı aramalar için **Full-text search** tercih edilir.

**Tablo: `guvenlik_loglari`**

|**log_id**|**hata_mesaji**|**risk_seviyesi**|
|---|---|---|
|1|"Unauthorized access attempt detected from IP 192.168.1.5"|Kritik|
|2|"System update completed successfully"|Düşük|
|3|"Multiple failed login attempts for user: admin"|Yüksek|
|4|"Access denied for unauthorized user"|Orta|

---

#### 3. Sorgu (Query)

Bu sorguda, önce `LIKE` kullanarak "access" ile başlayan mesajları bulacağız. Ardından, Full-text search kullanarak "unauthorized" kelimesini içeren mesajları (kelime köküne bakarak) daha profesyonel bir şekilde sorgulayacağız.

SQL

```sql
-- 1. LIKE ile basit desen eşleştirme
SELECT hata_mesaji 
FROM guvenlik_loglari
WHERE hata_mesaji LIKE 'Access%'; -- 'Access' ile başlayanlar

-- 2. Full-text search ile doğal dil araması
SELECT hata_mesaji, risk_seviyesi
FROM guvenlik_loglari
-- Mesajı tsvector'a, aranacak kelimeyi tsquery'ye dönüştürür
WHERE to_tsvector('english', hata_mesaji) @@ to_tsquery('english', 'unauthorized');
```

---

#### 4. Çıktı (Output)

|**hata_mesaji**|**risk_seviyesi**|
|---|---|
|"Unauthorized access attempt detected from IP 192.168.1.5"|Kritik|
|"Access denied for unauthorized user"|Orta|

- **Analiz:** `LIKE` sorgusu sadece 4 numaralı satırı (büyük/küçük harf duyarlılığına dikkat ederek) getirirken, **Full-text search** "unauthorized" kelimesini hem cümlenin başında hem de sonunda doğru bir şekilde yakalamıştır. Ayrıca Full-text search, "unauthorized" ve "unauthorizedly" gibi farklı türevleri de kök analizi sayesinde eşleştirebilir.

### 5: Benzerlik ve Bulanık Arama (String Similarity & Fuzzy Match)

#### 1. Kavramlar ve Tanımlar

- **Levenshtein distance (fuzzystrmatch):** Bir dizeyi diğerine dönüştürmek için gereken minimum tek karakterli düzenleme (ekleme, silme, değiştirme) sayısını döndüren, `fuzzystrmatch` eklentisi tarafından sağlanan bir dize metriğidir.
    
- **Trigram and similarity (pg_trgm):** Dizeleri üç karakterli örtüşen dizilere (trigram) ayıran ve dize benzerliğini ölçmek için 0 ile 1 arasında bir benzerlik puanı hesaplayan, `pg_trgm` eklentisi tarafından sağlanan bir yaklaşımdır.
    

---

#### 2. Senaryo ve Tablo Yapısı

Ağ envanterinizde cihaz isimlerinin elle girildiğini ve bu süreçte "Firewall" kelimesinin farklı şekillerde (yazım hatalarıyla) kaydedildiğini varsayalım. Amacımız, "Firewall" kelimesine en yakın olan hatalı kayıtları bulmaktır.

**Tablo: `envanter_hatalari`**

|**cihaz_id**|**girilen_isim**|
|---|---|
|1|"Firewall_Core"|
|2|"Firwall_Core"|
|3|"Freewall_Main"|
|4|"Switch_Layer3"|

---

#### 3. Sorgu (Query)

Bu işlemleri yapabilmek için önce ilgili PostgreSQL eklentilerini aktif etmemiz gerekir. Ardından, hem düzenleme mesafesini (Levenshtein) hem de benzerlik oranını (Trigram) hesaplayacağız.

SQL

```sql
-- 1. Eklentileri aktif etme
CREATE EXTENSION IF NOT EXISTS fuzzystrmatch;
CREATE EXTENSION IF NOT EXISTS pg_trgm;

-- 2. Benzerlik analizi
SELECT 
    girilen_isim,
    -- 'Firewall' kelimesine olan uzaklık (Küçük sayı = Daha benzer)
    levenshtein(girilen_isim, 'Firewall_Core') AS duzenleme_mesafesi,
    -- Benzerlik oranı (1'e yakın = Daha benzer)
    similarity(girilen_isim, 'Firewall_Core') AS benzerlik_skoru
FROM envanter_hatalari
WHERE similarity(girilen_isim, 'Firewall_Core') > 0.3
ORDER BY benzerlik_skoru DESC;
```

---

#### 4. Çıktı (Output)

|**girilen_isim**|**duzenleme_mesafesi**|**benzerlik_skoru**|
|---|---|---|
|"Firewall_Core"|0|1.0|
|"Firwall_Core"|1|0.82|
|"Freewall_Main"|6|0.35|

- **Analiz:** "Firwall_Core" ismi, hedef kelimeden sadece 1 karakter eksik olduğu için Levenshtein mesafesi 1 çıktı ve yüksek bir benzerlik skoru aldı. "Freewall_Main" ise hem yazım hatası hem de farklı bir son ek ("Main") içerdiği için mesafesi daha yüksek, benzerlik skoru ise daha düşük çıktı. "Switch_Layer3" ise benzerlik eşiğinin (0.3) altında kaldığı için sonuç listesine girmedi.


### 6: Şema Denetimi ve Özel Fonksiyonlar (Metadata & Custom Logic)

---

#### 1. Kavramlar ve Tanımlar

- **INFORMATION_SCHEMA:** Veritabanı nesnelerini (tablolar, sütunlar, veri tipleri vb.) tanımlayan ve kullanıcıların şema ayrıntılarını programlı olarak incelemesine olanak tanıyan, meta veri tablolarını içeren standart bir sistem şemasıdır.
    
- **User-defined function (CREATE FUNCTION):** Yeniden kullanılabilir hesaplamaları veya işlemleri gerçekleştirmek için SQL veya prosedürel kodu bir araya getiren, `CREATE FUNCTION` ile oluşturulan bir veritabanı nesnesidir.
    

---

#### 2. Senaryo ve Tablo Yapısı

Büyük bir veritabanında çalışırken hangi tabloda hangi veri tiplerinin kullanıldığını programlı olarak kontrol etmeniz gerekebilir. Ayrıca, ağ paketlerinin boyutuna göre otomatik bir "Risk Durumu" belirleyen bir fonksiyona ihtiyaç duyduğunuzu varsayalım.

**Tablo: `ag_paketleri`**

|**paket_id**|**protokol**|**boyut_kb**|**kaynak_ip**|
|---|---|---|---|
|1|TCP|150|192.168.1.10|
|2|UDP|1200|10.0.0.5|

---

#### 3. Sorgu (Query)

Bu sorguda önce `INFORMATION_SCHEMA` kullanarak tablomuzun teknik kolon yapılarını listeleyeceğiz. Ardından, paket boyutuna göre risk hesaplayan özel bir fonksiyon tanımlayıp bunu `SELECT` sorgusu içinde kullanacağız.

SQL

```sql
-- 1. Meta Veri İnceleme: Tablonun kolonlarını ve veri tiplerini listele
SELECT column_name, data_type 
FROM information_schema.columns 
WHERE table_name = 'ag_paketleri';

-- 2. Özel Fonksiyon Oluşturma: Boyuta göre risk kategorisi ata
CREATE FUNCTION belirle_risk_seviyesi(boyut INT) 
RETURNS TEXT AS $$
BEGIN
    IF boyut > 1000 THEN
        RETURN 'Kritik (Büyük Paket)';
    ELSE
        RETURN 'Normal';
    END IF;
END;
$$ LANGUAGE plpgsql;

-- 3. Fonksiyonu Kullanarak Veri Çekme
SELECT 
    paket_id, 
    boyut_kb, 
    belirle_risk_seviyesi(boyut_kb) AS risk_analizi
FROM ag_paketleri;
```

---

#### 4. Çıktı (Output)

**Meta Veri Sorgu Sonucu:**

|**column_name**|**data_type**|
|---|---|
|paket_id|integer|
|protokol|character varying|
|boyut_kb|integer|
|kaynak_ip|character varying|

**Risk Analizi Sonucu:**

|**paket_id**|**boyut_kb**|**risk_analizi**|
|---|---|---|
|1|150|Normal|
|2|1200|Kritik (Büyük Paket)|

- **Analiz:** `INFORMATION_SCHEMA` sorgusu, tablonun fiziksel yapısını görmemizi sağlayarak veri entegrasyonu süreçlerinde hata yapmamızı önler. `CREATE FUNCTION` ile oluşturduğumuz mantık ise SQL içinde karmaşık `CASE` ifadelerini tekrar tekrar yazmak yerine, merkezi ve temiz bir çözüm sunar.


### 7: Veri Kültürü ve Yardımcı Araçlar

#### 1. Kavramlar ve Tanımlar

- **Dual coding (İkili Kodlama):** Hedef kitlenin mesajları daha etkili bir şekilde işlemesini ve hatırlamasını sağlamak için tamamlayıcı sözel ve görsel bilgileri eşleştirme uygulamasıdır.
    
- **Flash Fill (Hızlı Doldurma):** Kullanıcı tarafından girilen örneklerden bir kalıbı algılayarak değerleri otomatik olarak dolduran, hücreler arasındaki metni çıkarmak veya birleştirmek için kullanışlı bir Excel özelliğidir.
    

#### 2. Senaryo

Bir veri mühendisi olarak, elinizde ham metin (plain text) halinde bulunan kullanıcı loglarını SQL veritabanına `INSERT` etmeden önce Excel'de hızlıca yapılandırılmış bir tabloya dönüştürmek istiyorsunuz. Ardından, bu verilerden elde ettiğiniz analizi bir rapor haline getirirken bilgilerin akılda kalıcılığını artırmak için **Dual Coding** prensiplerinden yararlanacaksınız.

#### 3. Uygulama: Excel Flash Fill ile Veri Hazırlama

Elinizde `USER_LOGIN_20260423_ISTANBUL` gibi karmaşık metin dizileri olduğunu varsayalım. Excel'de ilk satırı manuel olarak parçaladığınızda, Flash Fill geri kalan yüzlerce satırı otomatik olarak algılar.

|**Ham Veri (A Sütunu)**|**Kullanıcı (B Sütunu)**|**Tarih (C Sütunu)**|**Şehir (D Sütunu)**|
|---|---|---|---|
|USER_LOGIN_20260423_ISTANBUL|USER|2026-04-23|ISTANBUL|
|ADMIN_LOGIN_20260424_ANKARA|**ADMIN**|**2026-04-24**|**ANKARA**|
|GUEST_LOGIN_20260425_IZMIR|**GUEST**|**2026-04-25**|**IZMIR**|

(Flash Fill, ilk satırdaki mantığı (alt çizgileri ayraç olarak kullanma) saniyeler içinde tüm tabloya uygular ).

#### 4. Analiz ve Sunum: Dual Coding Uygulaması

Verileri SQL'e aktarıp analiz ettikten sonra, bulgularınızı sunarken sadece ham verileri veya sadece metinleri kullanmak yerine ikili kodlamayı kullanırsınız:

- **Sözel Kanal:** "Nisan ayının son haftasında İstanbul kaynaklı giriş denemelerinde %20 artış gözlemlenmiştir".
    
- **Görsel Kanal:** [İstanbul'daki artışı gösteren kırmızı bir trend grafiği].
    

Bu yöntem, paydaşlarınızın (veya proje ekibinizin) veritabanından gelen teknik sonuçları çok daha hızlı ve kalıcı bir şekilde anlamasını sağlar.


