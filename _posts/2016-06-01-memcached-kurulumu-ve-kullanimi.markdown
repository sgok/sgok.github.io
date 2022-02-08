---
layout: post
title:  "Memcached Kurulumu ve Kullanımı"
date:   2016-06-01 16:03:00 +0300
categories: server
excerpt_separator: <!--more-->
---

Merhaba arkadaşlar, bir önceki yazımda site performansını arttırma ile alakalı bir yazı yazacağımı söylemiştim. Bunun tek bir yazıya sığmayacağını düşündüm ve bir performans serisi yapmaya karar verdim. Bu serinin ilk ayağı olan bu yazımda cachleme sistemlerinden `Memcached`'ı anlatmaya çalışacağım.
<!--more-->
Bilindiği üzere cachleme denilince genellikle _shared hosting_ sahibi olanların çözüm olarak kullandığı _html output_ yöntemi akla gelir. Bu yöntemde sayfa içindeki ögeler bir cache klasörüne html çıktısı olarak kaydedilir ve siteye giren kullanıcılara bu html çıktısı gösterilir. Zaten paylaşımlı hosting kullananların hemen hemen başka çaresi de yoktur. **Smarty** gibi template engine sistemi kullananlar bu `template engine`lerin kendi içinde bulunan cachleme yöntemlerinden faydalanırlar. Bu tarz bir sistem kullanmayanlar ise kendi yöntemlerini geliştirerek bir şekilde cachleme sistemi yaparlar.

Bugün anlatacağım konu ise _VPS - VDS - Dedicated Server_ kullananların işine yarayacak bir konu, shared hosting kullananlar için kendi cachleme yöntemlerini yapabilecekleri bir sistemi başka bir yazımda konu edeceğim.

Konuyu daha fazla dağıtmadan biraz **Memcached**'dan bahsedeyim.

### Memcached Nedir?

Memcached dünyaca ünlü **LiveJournal** sitesi için geliştirilmiş açık kaynak kodlarından oluşan bir önbellekleme sistemidir. Facebook yazılım mühendislerinin geliştirmekte olduğu bu sistemi şuan dünyaca ünlü bir çok web sitesi_(Wikipedia, Facebook, Flickr, Twitter gibi)_ kullanmaktadır. Temel amacı yüksek trafik alan web sitelerinin sunucu yükünü azaltarak hızlı ve performanslı bir kullanıcı deneyimi sunmaktır.

### Çalışma Prensibi

Memcached'in çalışma mantığı aslında çok basit bir yapıdan ibarettir. Veritabanından çekilen veriyi ya da herhangi bir veriyi **Key** ve **Value** ile RAM belleğe atarak buradan ulaşılmasını sağlamaktır. RAM belleklerin okuma ve yazma hızını bilmeyeniniz yoktur sanırım. Tabi ki bu da hız ve performansı çok olumlu bir yönde etkilemektedir. Eğer yeteri kadar RAM belleğiniz varsa kesinlikle yüksek trafik alan web siteleriniz için Memcached kullanmanızı öneririm.

Bu sistem TCP/IP alt yapısını kullanarak bir çok farklı platformdan erişilebilmeyi de sağlar. Sadece PHP olarak düşünmemek gerekir bir çok yazılım platformunun birlikte kullanımlarında ideal bir çözümdür. Örneğin _Session paylaşımı_ yaparkende memcached kesinlikle iyi bir araçtır.

### Kurulum ve Yapılandırılması

**Ubuntu 14.04 Üzerinde Kurulumu**

Öncelikle sistemi update ediyoruz ve sonra gerekli olan paketleri kuruyoruz._(Sisteminizde Apache, MySQL Server ve PHP kurulu olduğu varsayılarak anlatılmıştır.)_ SSH üzerinden aşağıdaki komutları uygulayınız.

    sudo apt-get update
    sudo apt-get install php5-memcached memcached

**PECL ile Kurmak İçin**

    pecl install memcache

Servisi başlatmak için.

    memcached -d -m 512 -l 127.0.0.1 -p 11211 -u nobody

Daha sonrasında Apache Servera bir restart atalım.

    sudo service apache2 restart

Şimdi kurulduğunu görmek için public_html dizininizde bir php dosyası oluşturup içine aşağıdaki kodu yazınız.

    <?php
    phpinfo();
    ?>

Böylece kurulu olup olmadığının kontrolünü yapınız. Eğer kurulduysa şöyle bir görüntüyle karşılacaksınız.

![Memcached](/assets/img/upload/memcached_ubuntu_phpinfos.png)

Şimdi sistemde aktif olup olmadığını anlamak için SSH üzerinde aşağıdaki komutu verelim.

    ps aux | grep memcached

Komutunu verdiğinizde şöyle bir görüntüyle karşılaşmalısınız.

![Memcached PS AUX](/assets/img/upload/memcached_ubuntu_ps_auxs.png)

Buradan anlaşıldığı üzere memcached 11211 numaralı porttan çalışmaktadır.

Daha sonra memcached servisine bir restart atalım.

    sudo service memcached restart

Artık elimizde hazır bir memcached servisi var. Kurulumu bu kadar kolay işte siz ne sanmıştınız? :)

**Centos Üzerinde Kurulumu**

Öncelikle gerekli olan bağımlılıkları kuralım.

    yum install libevent libevent-devel

Daha sonrasında /usr/local/src dizinine gidip gerekli olan memcached dosyasını indiriyoruz.

    cd /usr/local/src
    wget http://memcached.googlecode.com/files/memcached-1.4.15.tar.gz

Arşiv dosyasını extract ediyoruz ve açılan dizinin içine giriyoruz.

    tar xvzf memcached-1.4.15.tar.gz
    cd memcached-1.4.15

Şimdi sıra geldi programı derlemeye. _(Sunucunuzun 64 Bit olduğu varsayılmıştır.)_ Ama öncesinde **gcc** ve **make** derleme araçlarının bulunduğundan emin olun eğer yoksa aşağıdaki komutları veriniz.

    yum install gcc
    yum install make

Daha sonra ise derleme işlemini başlatalım.

    ./configure --enable-64bit

Eğer işlemciniz çoklu çekirdeğe sahipse(Multi-Core) Threading özelliğini aktif edebilirsiniz derleme esnasında.

    ./configure --enable-threads --enable-64bit

Komutlarını veriniz ve sonra yükleme işlemine geçelim.

    make && make install

Eğer bir hata almadıysanız muhtemelen memcached serverı yüklenmiştir. Şimdi serverı başlatmak için aşağıdaki komutları verelim.

    memcached -d -u nobody -m 512 -p 11211 127.0.0.1

Garanti olması açısından memcached serverına ve apache sunucusuna restart atmakta fayda var.

**Windows Üzerinde Xampp'a Kurulumu**

Öncelikle Xampp'ın kurulu olduğu dizine gidiyoruz ve php klasörüne giriyoruz. Sonrasında php.ini dosyasını açıp aşağıdaki kodu aratıyoruz.

    ;extension=php_memcache.dll

Bulduysak eğer yanındaki noktalı virgülü kaldırıp modülün aktif olmasını sağlıyoruz. Eğer yoksa aşağıdaki kodları dosyanın en altına yapıştırıp kaydediyoruz.

    extension=php_memcache.dll
    [Memcache]
    memcache.allow_failover = 1
    memcache.max_failover_attempts=20
    memcache.chunk_size =8192
    memcache.default_port = 11211

Daha sonra şu adresten [Windows Memcache DLL](http://windows.php.net/downloads/pecl/releases/memcache/3.0.8/php_memcache-3.0.8-5.5-ts-vc11-x86.zip "Memcache Library for Windows") zip dosyasını indiriyoruz. Zip dosyasını açıp içindeki "php_memecache.dll" dosyasını xampp\php\ext klasörünün içine atıyoruz. Sonrasında ise [Memcached Server](http://static.runoob.com/download/memcached-1.2.6-win32-bin.zip "Memcached Server for Windows") dosyasını indiriyoruz ve c: üzerinde memcached klasörü oluşturup indirdiğimiz dosyanın içinden çıkan .exe dosyasını `c:\memcached` klasörünün içine atıyoruz.

Sonra **CMD** programını **Yönetici olarak çalıştır** diyerek açıyoruz ve şu komutu veriyoruz.

    c:/memcached/memcached.exe -d install

Sonrasında

    net start "memcached server"

Komutunu vererek serverı çalıştırıyoruz.

Xampp'ı restart ediyoruz.

Memcached'i restart ediyoruz.

    C:\Windows\system32> net start “memcached”

    C:\Windows\system32> net stop  “memcached”

Komutlarını sırasıyla vererek restart işlemini tamamlamış oluyoruz.

### Memcache Kullanımı

Eğer hata almadıysanız hemen hemen popüler bir çok işletim sisteminde **Memcached Serverı** ve **Memcache Library** kurulumlarını tamamladık. Memcached ile Memcache arasındaki fark birisi serverın adı diğeri ise kütüphanenin adıdır, bu ayrıntıyı vererek konuya devam ediyorum.

Şimdi ufak bir test yapalım ve memcache çalışacak mı bakalım. public_html klasörümüzde bir php dosyası oluşturarak içine aşağıdaki kodları yazalım.
```php
    <?php
    $mem = new Memcache;
    $mem->connect('localhost', 11211) or die ("Bağlantı Hatası!");

    $version = $mem->getVersion();
    echo "Server's version: ".$version."\n";

    $result = $mem->get("data");

    if ($result) {
        echo $result;
    } else {
        echo "Cachelenmiş data bulunamadı ama sayfayı yenilerseniz cachlenecek!";
        $mem->set("data", "Ben datayım! Benim içeriğim burada yer alıyor.") or die("Memcached bir hatadan dolayı veriyi kaydedemedi!");
    }
    ?>
```
Yukarıdaki kodları "cache.php" diye kaydedip browserınızda çalıştırdığınızda ilk olarak karşınıza

> Cachelenmiş data bulunamadı ama sayfayı yenilerseniz cachlenecek!

uyarısı çıkacak ve sayfayı yenilediğinizde ise

> Ben datayım! Benim içeriğim burada yer alıyor.

yazacak böylece artık ilk cachlememizi yapmış olacağız.

Yukarıda bulunan örnekte çok basit bir yapıyla gösterdim tabi ki şimdi daha detaylı bir örnek yapacağım ama öncesinde methodlarını öğrenmekte fayda var.
```php
    $memcache->get('anahtar') // anahtar keyine ait olan value yi(değeri) getirir.
    $memcache->set('anahtar', 'saklayacağımız içerik buraya gelecek', false, 10) // Bir değeri set etmek için kullanılır buraya yazdığımız veriler RAM de saklanır. İlk parametre RAM bellekteki 'key' değeri ikincisi value yani RAM bellekte saklayacağımız içerik, üçüncü parametre sıkıştırma yapılıp yapılmayacağını belirten parametredir. True olursa zlib ile sıkıştırma yapar eğer false ise sıkıştırma yapmaz. Son parametre ise kaç saniye cachleneceğini belirten parametredir. Buradaki süre dolduğu zaman cache silinir.
    $memcache->flush() // Ram bellekteki tüm depolanan keyleri siler.
    $memcache->delete('key') // Belirtilen key'e ait veriyi siler.
    $memcache->connect('hostname', 11211) // Connect ise örnekte kullandığımız gibi memcached serverına bağlanmamızı sağlar.
```
Memcached default olarak 11211 portunu kullanır o yüzden örneklerde 11211 portu üzerinden gösterdim. Spesifik olarak değiştirilmek istenirse ona göre port numarasını değiştirip projelerinizde kullanabilirsiniz.

Şunu unutmamak gerekiyor RAM üzerinde cacheleyeceğimiz her içerik 1 MB boyutunu geçmemeli eğer geçerse cachelenemez. Verileriniz 1MB üzerinde olduğunu düşünüyorsanız sıkıştırma parametresini TRUE yaparak sıkıştırmayı etkinleştirmeliyiz. Bunun sebebi RAM bellek üzerindeki her bloğun 1024 Kbyte büyüklüğünde olmasından kaynaklanmaktadır.

Şimdi bir de veritabanından veri çeken bir örnek yaparak asıl kullanım amacına uygun birşeyler yapalım.
```php
    <?php

    try { 
    	$db = new PDO('mysql:host=localhost;dbname=dbname;charset=utf8', 'username', 'password');
    } catch (PDOException $e) { 
    	echo $e->getMessage();
    }

    $mem = new Memcache;
    $mem->connect('localhost', 11211) or die ("Bağlantı Hatası!");

    $query = $db->query('SELECT * FROM data WHERE data_id < ?');
    $query->execute(array("1000"));
    $islem	= $query->fetchAll(PDO::FETCH_ASSOC);

    $array = array();

    foreach ($islem as $row)
    {	
    	$array['data_name'] = $row['data_name'];
    }
    $data_name = $mem->get('dataName')
    if($data_name === true){
    	print_r($data_name);
    }else{
    	$mem->set('dataName', $array['data_name'], true, 60) or die('Memcached üzerinde cacheleme işlemi yapılamadı!');	
    }
    ?>
```
Gibi bir örnekle kullanımını noktalıyorum gerisi sizin hayal gücünüz ve programlama yeteneklerinize kalmış birşeydir.

Memcache üzerinde bulunan get(), set(), flush(), delete(), connect() methodlarını kullanarak özelleştirmek artık sizin kendi işiniz.

Umarım faydalı bir döküman olmuştur. Eğer makale içinde yaptığım hatalar var ise bunları yorum yaparak bana bildirirseniz sevinirim. Kurulumlarda ve kullanımda karşılaştığınız hataları da yazarsanız onlara da çözüm getirmeye çalışırım. Herkese kolay gelsin.

**Faydalandığım makaleler:**

*   [StackOverflow](http://stackoverflow.com/questions/34173231/how-to-install-memcache-on-xampp-windows-7-8-10 "Install Memcache For Windows")
*   [DigitalOcean](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-memcache-on-ubuntu-14-04 "Install Memcache For Ubuntu")
*   [LiquidWeb](http://www.liquidweb.com/kb/how-to-install-memcached-on-centos-6/ "Install Memcache For Centos")
