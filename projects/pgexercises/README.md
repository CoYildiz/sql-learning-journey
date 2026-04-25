# SQL Challenges: Club Management System
Bu depo, pgexercises.com uzerinde yer alan SQL problemlerinin kapsamli cozumlerini ve veri analizlerini icermektedir. Proje, iliskisel veritabani sorgulama, veri manipulasyonu ve karmasik raporlama tekniklerini sergilemek amaciyla hazirlanmistir.

## Proje Icerigi
Cozumler pgexercises.com sitesindeki gibi ayni ana basliklar altinda gruplanmistir:
- Basic:
;;sonra ekle buraya diger kisma gecince




## Kurulum ve Calisma
Veri tabani PopOS(Ubuntu) ortaminda gelistirilmis olup, eger SQL istemcisi kullaniyorsaniz 1. adimi atlayabilirsiniz.

Veritabanini yerel ortamda kullanmak icin:
- 1. Veriyi yuklemek icin portun 5433 olup olmamasina dikkat edin ben GNU Guix kullandigim icin port PostgreSQL'in genel olarak kullandigi portan farkli olabilir.
Ayni sekilde exercises ve localhostu da kendi bilgisayar PostgreSQL ayarlariza gore duzenleyiniz.

```Bash
psql -h localhost -p 5433 -d exercises -f clubdata.sql
``` 
- 2. Sorgulari test etmek icin cozumlerin bulundugu exercises.md(veya .org) dosyasindaki sorgulari herhangi bir SQL istemcisi uzerinden calistirabilirsiniz.
