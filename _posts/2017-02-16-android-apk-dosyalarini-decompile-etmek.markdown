---
layout: post
title:  "Android APK Dosyalarını Decompile Etmek"
date:   2017-02-16 18:17:43 +0300
categories: android
excerpt_separator: <!--more-->
---

APK dosyalarını gerek zararlı yazılım tespiti gerekse belli başlı özellikleri irdelemek amaçlı **decompile** etme ihtiyacı duyabiliriz. Şimdi bu iş nasıl yapılır konuya girelim.
<!--more-->

#### **Başlayalım;**

1.  Öncelikle **APK** dosyamızı indiriyoruz, sonrasında uzantısını .apk'dan .zip'e çeviriyoruz. (Eğer APK dosyası yoksa sadece Google Play üzerinde yüklü ise **[bu siteye](https://apk-dl.com)** girip programın adını yazarak aratıyoruz ve arama sonucunda göründüyse tıklayıp **Download** seçeneğine tıklıyoruz.)

3.  Daha sonra herhangi bir arşiv programıyla(Windows için: winrar, winzip vs. Linux için: `unzip dosyaadi.zip`) .zip uzantılı yaptığımız dosyayı extract ediyoruz.

5.  Buraya kadar her şey tamamsa şimdi  **[dex2jar](https://sourceforge.net/projects/dex2jar/) **programını indiriyoruz ve zip dosyasını extract ediyoruz.

7.  Az önce apk dosyasını açtığımız klasöre girip classes.dex dosyasını dex2jar klasörünün içine kopyalıyoruz.

9.  Şimdi Windows için:**CMD**'yi açıp şu komutu veriyoruz `cd dex2jar_klasor_yolu` . Linux için: Terminali açıp `cd dex2jar_klasor_yolu` aynı komutu veriyoruz.

11.  Yukarıda ki komut ile dex2jar klasorune geldiyseniz şimdi (JAVA kütüphanelerinin bilgisayarınızda yüklü olduğunu varsayıyorum, eğer yüklü değilse öncelikle JAVA SDK'larını yükleyin.) Windows için: `d2j-dex2jar.bat classes.dex` Linux için: `chmod u+x d2j-dex2jar.sh`  ardından `./d2j-dex2jar.sh classes.dex` sonra komutunu verdikten sonra dosyayı .jar uzantısına çevirmesini bekliyoruz.

13.  **[jdgui](http://jd.benow.ca/)** programını indiriyoruz ve programı açıyoruz sonra programda **File** sekmesinden **Open File** diyerek <u>classes-dex2jar.jar</u> (veya sizin bilgisayarınızda adı her ne olduysa) dosyasını seçiyoruz ve karşınızda artık decompile edilmiş source dosyaları var. (Bu program java ile yazıldığı için cross platform çalışma özelliğine sahip yani Windows'da da Linux'da da aynı şekilde açabilirsiniz.)

15.  jdgui programından yine File sekmesine tıklayıp **Save All Sources **diyerek dosyaları bir yere kaydediyoruz ve artık herhangi bir kod düzenleyici editör yardımıyla dosyalarımızın içeriğinde ne arıyorsak bakmaya başlıyoruz.

İşte herşey bu kadar artık hangi amaçla kullanmak istiyorsanız decompile edilmiş kodları kullanabilirsiniz.
