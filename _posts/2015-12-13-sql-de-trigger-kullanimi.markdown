---
layout: post
title:  "SQL'de Trigger Kullanımı"
date:   2015-12-13 20:22:16 +0300
categories: sql
excerpt_separator: <!--more-->
---
Merhaba arkadaşlar bugün sizlere `SQL` de nasıl trigger kullanılır bunları anlatacağım. Konuya geçmeden önce temel SQL bilginiz olduğunu varsayarak konuyu anlatacağım.

Yaptığımız projelerde zaman zaman veritabanıyla alakalı ihtiyaç duyduğumuz işlevler oluyor. Bu işlevlerden bana göre ihtiyaç duyulan bir işlev olan `trigger`e geçmeden önce triggerin ne olduğunu öğrenelim.

<!--more-->

### Trigger Nedir?

Trigger kelimesi Türkçe de `tetiklemek` ya da `tetikleyici` anlamına gelmektedir. "Peki ya ne işe yarar bu tetikleyici neyi tetikler?" dediğinizi duyar gibiyim. Örnek verecek olursak bir tablomuz var ve bu tablomuzda yaptığımız işlemler başka bir tabloda da bir etki yaratsın istiyorsak trigger kullanmalıyız.

"Ne diyorsun sen be adam!" demeden önce bir daha düşünün lütfen :). Daha da açıklayacak olursak **kategoriler** adında bir tablomuz olsun. Bu tablonun içinden örneğin PHP kategorisini sileceğiz. PHP kategorisini sildiğimiz zaman istiyoruz ki bu kategoriye ait konular da silinsin. Tek tek elle bütün konular tablosunu aramayalım istiyoruz. İşte burada yardımımıza trigger koşuyor ve süper kahraman edasıyla bu işi ben çözerim diyor.

### Nasıl Yapılır?

Yukarıda verdiğim örnekten gidelim bir adet kategoriler tablomuz ve bir de konular tablomuz olsun. PHP kategorisini sileceğiz ve o kategoriye ait olan tüm konuları sileceğiz. Bu kadar teori yeter artık biraz pratiğe geçip kod yazalım.

Şimdi kendi veritabanı yöneticimizi açalım phpMyadmin veya hangisini kullanıyorsanız SQL kodu yazma bölümüne gelip aşağıdaki kodları yazalım
```sql
    DELIMITER $$

    CREATE
    	DEFINER = root@localhost /* Tanımlayıcı */
    	TRIGGER konulari_sil /* Trigger adı */
    	AFTER DELETE /* Trigger Zamanı (AFTER) Trigger Olayı (DELETE) */
    	ON kategoriler /* kategoriler tablomu seçtim */
    	FOR EACH ROW /* Buradan sonra trigger tanımlamamıza geçebiliriz */
    	BEGIN
    	DELETE FROM konular WHERE konu_kategori_id = OLD.kategori_id; /* konular tablomda, konu_kategori_id  eşitse kategoriler tablomda son silinen kategori_id ye işlemi yap. */
    	END$$

    DELIMITER ;
```

Yukarıda gördüğünüz üzere `konulari_sil` adında bir trigger adı verdik. Sonrasında `AFTER DELETE` diyerek silme işlemi gerçekleştikten sonra yap dedik. Daha sonra `kategoriler` tablomuzu seçtik. `BEGIN` ve `END$$` belirteçlerimiz arasına yapılacak işlemi tanımlıyoruz. `konu_kategori_id` eşitse `OLD.kategori_id` ye yani son silinen kategorinin id'sine eşitse konular tablosundaki konu_kategori_id bu işlemi gerçekleştir ve konuyu sil diyoruz.

Yukarıda ki trigger tanımlamamızı yaptıktan sonra (tabi siz kendi tablo ve sütunlarınıza göre isimlerini değiştirebilirsiniz.) artık kategori sildiğimiz zaman otomatik olarak o kategoriye ait konumuzun silinmesi tetiklenmiş olacak ve bizi büyük bir külfetten kurtaracak.

Bu işlemi siz kendi projenize veya veritabanınıza göre şekillendirip farklılaştırabilirsiniz, ben örnek olması açısından böyle bir örnek gösterdim. Tabi ki triggerlerin işlevi sadece `DELETE` işlemiyle bitmiyor `INSERT, UPDATE` gibi işlemlerde yapılıyor. Bu işlemlere `BEFORE, AFTER` gibi zamanlar verip gerçekleşecek olan işlemin önce mi yoksa sonra mı olacağına karar verebiliyoruz.

### Farklı Örnekler

Farklı bir örnek olması açısından başka bir işlem gerçekleştirelim. Örneğin `malidurum` adında bir tablomuz var ve bu tabloda güncelleme yapıldığında otomatik olarak `kayitlar` tabloma yansısın istiyorum ki yapılan değişiklikleri kayitlar tablosunda izleyebilelim.
```sql
    DELIMITER $$

    CREATE
    	DEFINER = root@localhost /* Tanımlayıcı */
    	TRIGGER tetikleyici_adi /* Trigger adı */
    	BEFORE UPDATE /* Trigger Zamanı (BEFORE) Trigger Olayı (UPDATE) */
    	ON malidurum /* malidurum tablomu seçtim */
    	FOR EACH ROW /* Buradan sonra trigger tanımlamamıza geçebiliriz */
    	BEGIN
    	INSERT INTO kayitlar SET kayit_cari = OLD.cari_durum, guncelleme_tarihi = OLD.cari_tarih; /* kayitlar tabloma yapılan değişikliği eklettiriyorum. */
    	END$$

    DELIMITER ;
```

Yukarıda ki örnekte malidurum tablosunda bir güncelleme yapılırsa bu UPDATE işleminden önce yapılan değişiklikleri kayıtlar adlı tabloya ekle dedim ve değerlerini bildirdim.

Trigger konusunda elbette daha farklı yapılan işlevler vardır ancak ben en çok işe yarayabilecek konuları anlatmak istedim. En azından temel olarak tetikleyiciler ne işe yarar öğrenmiş olduk.
