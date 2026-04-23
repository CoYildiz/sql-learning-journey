- **SQL (Structured Query Language):** İlişkisel veritabanlarında verileri sorgulamak, yönetmek ve manipüle etmek için kullanılan bir programlama dilidir.
- **PostgreSQL:** Genellikle SQL öğreniminde ve pratiğinde kullanılan ücretsiz, açık kaynaklı bir ilişkisel veritabanı sistemidir.

Query:
```sql
SELECT DISTINCT attack_category AS "Saldırı Türü"
FROM network_traffic_logs
LIMIT 5;
```
- **DISTINCT:** sütundan yalnızca benzersiz değerleri döndürür. Milyonlarca log kaydı içinde belki sadece 8 farklı saldırı türü (DDoS, Phishing, vb.) var. `DISTINCT` sayesinde aynı saldırı türünün milyonlarca kez listelenmesini engeller, sadece mevcut kategorileri tekil olarak çekeriz.
- **AS(ALIAS) :** Tabloya veya sütuna geçici bir isim atar. Orijinal veritabanında sütunun adı `attack_category` olsa da, biz onu ekranda veya analiz raporunda daha okunabilir olması için `"Saldırı Türü"` olarak adlandırdık.
- **LIMIT:** PostgreSQL'de döndürülen satır sayısını kısıtlar. Eğer tablon çok büyükse, verinin neye benzediğini görmek için tüm tabloyu çekmek sistemi yorar ve bekletir. `LIMIT 5` diyerek "bana sadece ilk 5 benzersiz sonucu getir" demiş oluyoruz.
- **LIMIT and/or OFFSET clause:** LIMIT 5 OFFSET 10 diyerek de sorgunun(query) ilk 10 satirini atla ve sonrasinda ki 5 satiri yaz anlamina geliyor.

Query ex:
```sql
SELECT *
FROM employees
ORDER BY salary DESC
LIMIT 5 OFFSET 10;
```
Output: 11 -> 12 -> 13 -> 14 -> 15 satirlari


---
- **WHERE Clause:** Belirli koşulları karşılayan kayıtları filtrelemek için kullanılan bir ifadedir.

- **BETWEEN:** Sonuçları belirli bir değer aralığında filtrelemek için kullanılan bir anahtar kelimedir.

- **IN:** Değerlerin belirtilen bir listedeki herhangi bir değerle eşleşip eşleşmediğini kontrol ederek kayıtları filtrelemek için kullanılan bir anahtar kelimedir.

- **LIKE / NOT LIKE:** Genellikle joker karakterlerle birlikte, metinlerde desen eşleştirme (pattern matching) için kullanılan anahtar kelimelerdir.

- **Wildcard Character:** LIKE operatörü ile birlikte dize eşleştirmesinde bir veya daha fazla karakteri temsil etmek için kullanılan sembollerdir (metinde % ve _ olarak geçer).

- **Null Value:** SQL'de bir kayıttaki eksik veya bilinmeyen verileri gösteren bir işarettir.

- **IS NULL / IS NOT NULL:** Bir veri kümesindeki eksik değerleri test etmek için kullanılan operatörlerdir.

##### Ornek PostreSQL sorgusu

UNSW-NB15 gibi bir ağ sızma tespiti veri seti üzerinde çalıştığını varsayalım. Sadece TCP ve UDP protokollerindeki, belirli bir süre aralığında gerçekleşen ve içinde "DoS" (Denial of Service) kelimesi geçen, aynı zamanda kaynak portu eksik (Null) olmayan paketleri filtrelemek istiyoruz.

Query:
```sql
SELECT source_ip, dest_ip, protocol, attack_category
FROM unsw_nb15_traffic_logs
WHERE protocol IN ('tcp', 'udp')
  AND duration BETWEEN 0.5 AND 2.0
  AND attack_category LIKE '%DoS%'
  AND source_port IS NOT NULL;
```
Sorgunun Adım Adım İncelemesi:

    WHERE: Tüm bu filtreleme işlemlerini başlatacağımızı veritabanına bildiren ana komuttur.

- IN ('tcp', 'udp'): Protokol sütunu sadece 'tcp' veya 'udp' olanları getirir. Tek tek protocol = 'tcp' OR protocol = 'udp' yazmak yerine kodu çok daha temiz hale getirir.

- BETWEEN 0.5 AND 2.0: Bağlantı süresi (duration) 0.5 ile 2.0 saniye arasında olanları seçer.

- LIKE '%DoS%': Saldırı kategorisi içinde nerede olursa olsun "DoS" geçenleri bulur. Buradaki % sembolü (Wildcard Character), "DoS kelimesinden önce veya sonra herhangi bir sayıda karakter olabilir" anlamına gelir. Örneğin "DDoS" veya "DoS_Attack" değerlerini de başarıyla yakalar.

- IS NOT NULL: Makine öğrenmesi modelleri eksik verileri (NaN/Null) olmamali. Bu komutla, kaynak portu kaydedilmemiş, yani eksik olan sorunlu satırları daha SQL aşamasındayken veri setimizden dışlamış (drop etmiş) oluruz.

---
#### 4. Fonksiyonlar ve Hesaplamalar

- **Aggregate Functions (Kümeleme/Toplama Fonksiyonları):** Değerleri özetleyen fonksiyonlardır; örneğin `SUM()` (toplam), `AVG()` (ortalama), `MIN()` (minimum), `MAX()` (maksimum).

- **COUNT(\*):** Null (eksik) değerlere sahip olanlar dahil olmak üzere bir tablodaki tüm satırları sayan bir fonksiyondur.

- **COUNT(column_name):** Belirli bir sütundaki null olmayan (dolu) değerleri sayan bir fonksiyondur.

- **ROUND(col_name, int):** Sayısal değerleri belirtilen ondalık basamak sayısına yuvarlayan bir fonksiyondur.

- **Arithmetic Operators (Aritmetik Operatörler):** SQL hesaplamalarında kullanılan sembollerdir (+, -, *, /).

##### Örnek PostgreSQL Sorgusu

İleri Veri Analitiği için sınıflandırma metotlarını (örneğin Naive Bayes veya KNN) test etmek üzere UCI öğrenci ayrılma (student dropout) veri seti üzerinde çalıştığını düşünelim. Modele veriyi vermeden önce, öğrencilerin not ortalamalarını ve veri setindeki eksik not durumlarını hızlıca özetlemek istiyoruz.


Query:
```sql
SELECT 
    COUNT(*) AS total_records,
    COUNT(admission_grade) AS valid_grades_count,
    ROUND(AVG(admission_grade), 2) AS average_admission_grade,
    MAX(admission_grade) - MIN(admission_grade) AS grade_range
FROM uci_student_dropout_data;
```

- **COUNT(\*):** Tablodaki toplam satır (öğrenci/gözlem) sayısını verir. Eğer veri setinde 4424 satır varsa, sonuç tam olarak 4424 olacaktır.

- **COUNT(admission_grade):** Sadece giriş notu girilmiş olan (null olmayan) satırları sayar. Eğer bu değer `COUNT(*)` sonucundan düşükse, aradaki fark kadar eksik verin (missing value) var demektir ve bu satırları imputation (doldurma) işleminden geçirmen gerektiğini hemen anlarsın.

- **AVG() ve ROUND(col_name, int):** `AVG` fonksiyonu notların ortalamasını alır. Ancak sonuç çok uzun ondalıklı bir sayı (örn: 12.456789) çıkabileceği için, `ROUND(..., 2)` kullanarak sonucu virgülden sonra 2 basamak olacak şekilde (12.46) yuvarlarız ve okunabilirliği artırırız.

- **Aritmetik Operatör (-):** En yüksek nottan (`MAX`) en düşük notu (`MIN`) çıkararak notların dağılım aralığını (range) hesaplarız.

### Bu İşlemlerin En Çok Kullanıldığı Alanlar

1. **Tanımlayıcı İstatistikler (Descriptive Statistics):** Veri setinin genel eğilimini (ortalama, maksimum, minimum) veri tabanı seviyesinde hesaplamak, veriyi Python ortamına çekmeden önce büyük bir zaman kazandırır.
2. **Eksik Veri (Missing Data) Tespiti:** `COUNT(*)` ile `COUNT(sütun_adı)` arasındaki farkı incelemek, veri ön işleme (preprocessing) adımlarının en hızlı kalite kontrol yöntemidir.
3. **Yeni Öznitelik Üretme (Feature Extraction):** Aritmetik operatörler kullanılarak mevcut sütunlardan yenileri türetilebilir. Örneğin, `(gelir - gider)` şeklinde bir SQL hesaplaması yapılarak model için doğrudan "net_kar" adında yeni ve daha anlamlı bir öznitelik (feature) yaratılabilir.



#### 5. Gruplama ve Sıralama

Bu komutlar, satır satır olan veriyi kategorik olarak birleştirip, makine öğrenmesi modelleri için anlamlı istatistikler çıkarmamızı sağlar.

- **GROUP BY:** Toplama işlevlerinin (aggregate functions) uygulanabilmesi için ortak bir özelliği paylaşan satırları gruplandıran bir ifadedir.
    
- **HAVING Clause:** Çoğunlukla GROUP BY ile kombinasyon halinde, birleştirilmiş (gruplanmış) verileri filtrelemek için kullanılan bir ifadedir.
    
- **ORDER BY:** Sorgu sonuçlarını bir veya daha fazla sütuna göre sıralamak için kullanılan bir anahtar kelimedir.
    
- **ASC / DESC:** Sonuçları artan (ascending) veya azalan (descending) sırada düzenlemek için kullanılan seçeneklerdir.
    

---

### Örnek PostgreSQL Sorgusu

Ağ sızma tespiti projeniz için modelleri eğitmeden önce, UNSW-NB15 veri setindeki saldırı sınıflarının dengesini (class imbalance) ve ortalama sürelerini incelemek üzere toplandığınızı düşünelim. Sadece 1000'den fazla gerçekleşen saldırı türlerini görmek istiyorsunuz:

Query:
```sql
SELECT 
    attack_category,
    COUNT(*) AS total_attacks,
    ROUND(AVG(duration), 2) AS avg_duration
FROM unsw_nb15_traffic_logs
GROUP BY attack_category
HAVING COUNT(*) > 1000
ORDER BY avg_duration DESC;
```

### Sorgunun Adım Adım İncelemesi:

- **GROUP BY attack_category:** Tüm veri setini saldırı türlerine göre (örneğin; Fuzzers, Analysis, Backdoors) paketlere ayırır. Artık `COUNT` veya `AVG` fonksiyonları tüm veritabanı için değil, her bir saldırı paketi (grubu) için ayrı ayrı hesaplanır.
    
- **HAVING COUNT(\*) > 1000:**  `WHERE` komutu satırları gruplanmadan önce filtrelerken; `HAVING` komutu veriler **gruplandıktan sonra** ortaya çıkan sonuçları filtreler. Yani "bana sadece içinde 1000'den fazla kayıt olan saldırı gruplarını getir" demiş oluyoruz.
    
- **ORDER BY avg_duration DESC:** Oluşan bu özet tabloyu, ortalama saldırı süresine (`avg_duration`) göre büyükten küçüğe (`DESC`, azalan) doğru sıralar. En uzun süren saldırı türü listenin en tepesinde çıkar. Eğer küçükten büyüğe sıralamak isteseydik `ASC` (artan) kullanırdık.
    

---

### Bu İşlemlerin En Çok Kullanıldığı Alanlar

1. **Sınıf Dengesizliği (Class Imbalance) Analizi:** Model eğitiminde GNN, SVM veya MLP fark etmeksizin veri setindeki etiketlerin (labels) dağılımını görmek çok önemlidir. Hangi sınıftan kaç örnek olduğunu `GROUP BY` ile anında görürsün.
    
2. **Agregasyon Bazlı Öznitelik Mühendisliği (Feature Engineering):** Bazen her bir kullanıcının veya IP adresinin "ortalama işlem süresi" veya "toplam bağlantı sayısı" gibi grup bazlı metrikleri modele yeni bir öznitelik (feature) olarak eklenir.
    
3. **Anomali Tespiti:** Çok nadir gerçekleşen olayları (örneğin `HAVING COUNT(*) < 5` diyerek) veritabanı seviyesinde hızlıca tespit etmek için kullanılır.


#### 6. Sorgu Çalıştırma Sırası (Query Execution Order)

Bir SQL sorgusunu yazarken her zaman `SELECT` komutuyla başlarız, ancak veritabanı motoru (örneğin PostgreSQL) kodu okumaya en üstten başlamaz.

- **Query Execution Order (Sorgu Çalıştırma Sırası):** SQL ifadelerinin yürütülme (çalıştırılma) sırasıdır. Bu sıra genel olarak şöyledir: FROM / JOIN → WHERE → GROUP BY → HAVING → SELECT → DISTINCT -> ORDER BY → LIMIT AND/OR OFFSET CLAUSE.
    

Peki bu sıralama mantığı tam olarak nasıl işliyor? Az önce yazdığımız ağ trafiği sorgusu üzerinden adım adım gidelim:

1. **`FROM` (Veriyi Bul):** Sistem önce "Hangi veritabanına ve tabloya gidiyorum?" der. Çalıştığın devasa ağ trafiği veri setini (`unsw_nb15_traffic_logs`) bulur ve hafızaya alır.
    
2. **`WHERE` (Satırları Filtrele):** Tabloyu bulduktan sonra, istenmeyen satırları hemen atar (örneğin "sadece TCP olanları bırak"). Bu işlem, sistemin yükünü erkenden hafifletir.
    
3. **`GROUP BY` (Paketle):** Kalan temiz satırları, istenen özelliğe (örneğin `attack_category`) göre alt kümelere ayırır.
    
4. **`HAVING` (Grupları Filtrele):** Oluşan bu yeni paketleri filtreler (örneğin "1000'den az kaydı olan paketleri çöpe at").
    
5. **`SELECT` (Görüntüle):** Sistem ancak bu aşamaya geldiğinde "Peki kullanıcı benden hangi sütunları veya hesaplamaları ekranda görmemi istiyordu?" diye sorar ve sütunları seçer.
    
6. **`ORDER BY` (Sırala):** Elinde kalan nihai tabloyu istenen düzene göre (büyükten küçüğe vb.) sıralar.
    
7. **`LIMIT` (Sınırla):** En son aşamada, ekrana basılacak sonuç sayısını kırpar (örneğin "sadece ilk 5 sonucu göster").
