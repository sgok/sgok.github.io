---
layout: post
title:  "SQL'de View ve Stored Procedure Kullanımı"
date:   2016-05-12 20:56:16 +0300
categories: sql
excerpt_separator: <!--more-->
---

Merhaba arkadaşlar bugün anlatacağım konular `Stored Procedure`(Saklı Yordamlar) ve `View`(Görünümler) olacak. Konuya geçmeden önce temel SQL bilginiz olduğunu varsayıyorum. Bir önceki konuda bahsettiğim gibi([SQL'de Trigger Kullanımı](https://sgok.github.io/sql/2015/12/13/sql-de-trigger-kullanimi.html "SQL'de Trigger Kullanımı")) SQL kullanırken onun özelliklerinden tam anlamıyla yararlanmalıyız. SQL işlevlerini kullanırken çok amaçlı işler yapabilirsiniz. Ben temel anlamda(giriş seviyede) konuyu anlatacağım.
<!--more-->
### View (Görünümler)

View'ler nedir önce ondan bahsedeyim. Listeleme işlemlerimizi yaparken bize hız kazandıran ve SQL sunucuyu daha az yoran bir işlevdir. Genellikle `view`lerde `SELECT` sorgusu kullanılır.

Özetleyecek olursak bizim normalde PHP kodu içersinde kullandığımız SQL sorgularımızı, view denilen fiziksel olarak bulunmayan sanal tablolardan çalıştırırız. Tablolar veritabanında fiziksel olarak bulunurlar ancak viewler sql sorgularından oluşurlar.

Şimdi bunu örneklendirelim ve konuyu daha iyi anlayalım.
```sql
    CREATE 
    	VIEW onayliYorum /* View adı*/
    	AS
    	SELECT * FROM yorumlar AS yo /* Sorgumuz */
    	INNER JOIN konular AS ko ON ko.konu_id = yo.yorum_konu_id 
    	WHERE yorum_onay = 1;
```
Şeklinde oluşturulurlar. Bu örneğimizde yorumlar tablomuzdan konulara ait onaylanmış yorumları listelettik. Şimdi bunu PHP tarafında nasıl kullanacağımızı anlatayım.
```php
    <?php

    	$mysqli = new mysqli("localhost", "root", "sifre", "dbadi"); // DB bağlantısını yapıyoruz.

    	if( $yorumListelet = $mysqli->query("SELECT * FROM onayliYorum") ) { // onayliYorum View'ini çağırdık. ve ne yapmak istiyorsak yaptık.
    		foreach ( $yorumListelet as yorumListele ) {
    			print_r($yorumListele);		
    		}
    	}
    ?>
```

Viewleri silmek için;
```sql
    DROP VIEW onayliYorum
```
Yazmak yeterlidir.

PHP üzerinde kullanımı yukarıda ki şekildedir. Tabi ki bunu genişletip sorgularınızı değiştirebilirsiniz. Basit ve hızlıdır, viewleri projelerinizde rahatlıkla kullanabilirsiniz.

### Stored Procedure (Saklı Yordamlar)

SP'ler nedir kısaca bahsetmek gerekirse veritabanında bulunan derlenmiş SQL sorgularıdır diyebilirim. Yani siz SP'leri çağırdığınızda hazır olarak derlenmiş sonuçları alırsınız bu da veritabanı trafiğini oldukça azaltır. SQL sorgularından çok daha hızlı çalışırlar. SP'lerin iyi yazılmış olması çok önemlidir. Performans kaybı yaşamamak için optimize edilmiş SQL kodları kullanılmalıdır. SP'ler aynı zamanda bize veri güvenliği de sağlarlar. Stored Procedure'ye kimlerin erişeceğini belirtebiliriz, böylece güvenliği sağlamış oluruz. Şimdi düşünün büyük bir projeniz var ve sürekli aynı sql sorgularıyla çalışıyorsunuz bir çok yerde aynı sorguyu tek tek yazıyorsunuz. İşte bunun önüne geçmek için bir SP oluşturup tek bir sefer kodu çalıştırmanız hem performans anlamında hemde hızlı kod yazma anlamında size büyük avantaj sağlayacaktır.

O kadar çok teori anlattık ki artık basit şekilde bir SP oluşturalım ve ne olduğunu pratikte görelim.
```sql
    DELIMITER $$
       CREATE
    	PROCEDURE yorumGoster()
    	BEGIN
    	   SELECT * FROM yorumlar WHERE yorum_onay = 1;
    	END$$
    DELIMITER ;
```
View'lerde gördüğümüz örneğe benzer bir şekilde göstermek istedim en azından nasıl çalıştığını anlayabilmeniz açısından.

Şimdi bu SP yi nasıl çalıştırabileceğimize bakalım.
```php
    <?php

    	$mysqli = new mysqli("localhost", "root", "sifre", "dbadi"); // DB bağlantısını yapıyoruz.

    	if ( $yorumGoster = $mysqli->query("CALL yorumGoster()") ) { // SP'yi çağırıyoruz.
    		foreach ($yorumGoster as $yorumListele) { 
    			print_r($yorumListele);	
    		}
    	}
    ?>
```
`CALL yorumGoster()`Şeklinde çağrılabilir.

Şimdi birde parametreli örnek yapalım.
```sql
    DELIMITER $$
       CREATE
    	PROCEDURE yorumGoster(sp_onay INT, sp_kimin_yorumu VARCHAR(100))
    	BEGIN
    	   SELECT * FROM yorumlar WHERE yorum_onay = sp_onay AND yorum_yapan = sp_kimin_yorumu;
    	END$$
    DELIMITER ;
```
Yukarıda parametreli bir SP oluşturduk. Burada sp_onay değişkeninin integer değer alacağını belirttim ve sp_kimin_yorumu adlı değişkende ise varchar veri türü olabileceğini belirttim. Şimdi bunu nasıl çağıracacağız onu göstereyim.
```sql
    CALL yorumGoster(1, 'w133')
```
Şeklinde parametre verilerek çalıştırılır. Birinci değerimiz integer olduğu için değeri sayısal şekilde gönderdik ve ikincil değişkenimiz varchar olduğu için onu string şeklinde gönderdik. Biraz daha karmaşık örneklere geçelim.
```sql
    DELIMITER $$
       CREATE
    	CREATE DEFINER= w133@localhost /* SP yi sadece w133 kullanıcısı kullanabilir diye tanımlıyorum */
    	PROCEDURE yorumEkle(sp_yorum_ekleyen VARCHAR(100), sp_yorum_eposta VARCHAR(100), sp_yorum_icerik TEXT, sp_yorum_tarih DATETIME)
    	BEGIN
    	   INSERT INTO yorumlar 
    	   SET yorum_yapan = sp_yorum_ekleyen, 
    	   yorum_email = sp_yorum_eposta, 
    	   yorum_icerik = sp_yorum_icerik, 
    	   yorum_tarih = sp_yorum_tarih;
    	END$$
    DELIMITER ;	
```
Kullanımına gelecek olursak yine benzer şekilde parametreli kullanılır.
```php
    <?php

    	$mysqli = new mysqli("localhost", "root", "sifre", "dbadi"); // DB bağlantısını yapıyoruz.
    	$yorum_ekleyen = $_POST["yorum_ekleyen"];
    	$yorum_email = $_POST["yorum_email"];
    	$yorum_icerik = $_POST["yorum_icerik"];
    	$yorum_tarih = date("Y-m-d H:m:s");

    	if ( $yorumEkleme = $mysqli->query("CALL yorumEkle('".$yorum_ekleyen."', '".$yorum_email."', '".$yorum_icerik."', ".$yorum_tarih.")") ) { // SP'yi çağırıyoruz.
    		echo "Yorum başarıyle eklendi";
    		}
    	}
    ?>
```
Ben anlaşılması açısından ve örnek olması için basit bir örnek verdim. Geliştirmek yine size kalmış.

Yine farklı bir örnek verelim.
```sql
    DELIMITER $$
       CREATE
    	PROCEDURE konuGetir(IN p_konu_id INT, OUT f_name VARCHAR(45))
    	BEGIN
    	   SELECT konu_adi INTO f_name FROM konular WHERE konu_id = p_konu_id;
    	END$$
    DELIMITER ;
```
Önceki örneklerde tekli parametre görmüştük şimdi bu örnekte in/out lu parametreler alıyor. "Yahu bu da neyin nesi?" demeden önce size açıklayayım. IN ile daha önceki örneklerde gördüğümüz giriş parametresinin tipini belirtmiş oluyoruz. OUT ile bir çıkış değişkeni oluşturuyoruz aslında. Daha iyi anlamak için bu SP'yi çağıralım.
```sql
    CALL konuGetir(3, @kullanici);
```
Out ile @kullanici değişkenine aktardığımız sorguyu artık sadece;
```sql
    SELECT @kullanici
```
Diye çağırarak 3 numaralı konuyu çağırabiliriz.

### IF Else Kullanımı

Stored Procedureler aslında fonksiyon oldukları için T-SQL komutları da rahatlıkla kullanılıp programlanabilirler. Şimdi IF ve ELSE şart cümleciklerini kullanarak bir örnek yapalım
```sql
    DELIMITER $$

    CREATE PROCEDURE MusteriSeviyesi(
        in  p_musteriNumarasi int(11), 
        out p_musteriSeviyesi  varchar(10))
    BEGIN
        DECLARE kredilim double; /* kredilim diye bir değişken tanımladık */

        SELECT kredilimiti INTO kredilim
        FROM musteriler
        WHERE musteriNumarasi = p_musteriNumarasi;

        IF kredilim > 100 THEN /* Kredi 100 den Büyükse */
            SET p_musteriSeviyesi = 'ALTIN'; /* Müşteri ALTIN üye olsun.*/
        ELSEIF kredilim <= 100  THEN
            SET p_musteriSeviyesi = 'GUMUS';
        ELSEIF kredilim < 50 THEN
            SET p_musteriSeviyesi = 'BRONZ';
        END IF;

    END$$

    DELIMITER ;
```
### When Case Kullanımı

WHEN CASE koşulunuda Procedurlerimizde kullanabiliriz örneğin
```sql
    DELIMITER $$

    CREATE PROCEDURE MusteriSeviyesi(
     in  p_musteriNumarasi int(11), 
     out p_MusteriSeviyesi  varchar(10))
    BEGIN
        DECLARE kredilim double;

        SELECT kredilimiti INTO kredilim
     FROM musteriler
     WHERE musteriNumarasi = p_musteriNumarasi;

        CASE  
     WHEN kredilim > 100 THEN 
        SET p_musteriSeviyesi = 'ALTIN';
     WHEN kredilim <= 100 THEN
        SET p_musteriSeviyesi = 'GUMUS';
     WHEN kredilim < 50 THEN
        SET p_musteriSeviyesi = 'BRONZ';
     END CASE;

    END$$

    DELIMITER ;
```
IF ve ELSE kullanımına benzer bir şekilde WHEN CASE koşullarını da SP'lerde rahatlıkla kullanabiliriz. İyi güzel bunları yaptıkta şimdi nasıl çağıracağız diye düşünebilirsiniz. Şimdi nasıl çağrıldığına bakalım.
```sql
    CALL MusteriSeviyesi(112,@seviye); /* 112 numaralı müşteri */
    SELECT @seviye AS 'Musteri Seviyesi'; /* Artık Musteri Seviyesi takma adımızla rahatlıkla tabloyu izleyebileceğiz. */
```
### While Döngüsü Kullanımı

Aslında programlama dillerinde neler yapılabiliyorsa hemen hemen benzeri şeyler SP içinde de yapılıyor döngüler bile! İsterseniz basit bir döngü yazalım.
```sql
    DELIMITER $$
     DROP PROCEDURE IF EXISTS donguWhile$$
        CREATE PROCEDURE donguWhile(IN baslangic INT(10), IN bitis INT(10))
        BEGIN
         DECLARE isim VARCHAR(100);
         SET isim = '';
         WHILE baslangic < bitis DO
          SET isim = concat(isim, baslangic, ',');
          SET baslangic = baslangic + 1;
         END WHILE;
        END$$
    DELIMITER ;
```
Şeklinde While döngüsü kullanılabilir.

Şimdi biraz daha farklı bir konu olan Stored Functions örneği yapalım.

### Stored Functions
```sql
    DELIMITER $$
       CREATE
    	FUNCTION cemberAlaniHesapla(sayi INT)
    	RETURNS FLOAT
    	BEGIN
    	   RETURN PI() * sayi * sayi;
    	END$$
    DELIMITER ;
```
Şeklinde tanımlanır kullanımı ise;
```sql
    SELECT cemberAlaniHesapla(3);
```
Görüldüğü üzere çember alanını bulan bir Stored Functions tanımladık.

Daha çok çeşitli örnekler yapılabilir. Umarım View'ler, Stored Procedure ve Stored Functions konularını anlatabilmişimdir.
