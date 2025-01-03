---
{"dg-publish":true,"permalink":"/web-penters/using-web-proxies/"}
---


### CA Sertifikası Yükleme
Tarayıcımızla Burp Proxy/ZAP kullanırken bir diğer önemli adım da web proxy'sinin CA Sertifikalarını yüklemektir. Bu adımı yapmazsak, bazı HTTPS trafiği düzgün bir şekilde yönlendirilemeyebilir veya Firefox'un her HTTPS isteği göndermesi gerektiğinde kabul et'e tıklamamız gerekebilir.

Foxy Proxy'de proxy olarak Burp'ü seçtikten sonra http://burp adresine giderek Burp'ün sertifikasını yükleyebilir ve CA Sertifikasına tıklayarak sertifikayı buradan indirebiliriz:

![Pasted image 20241106114205.png](/img/user/resimler/Pasted%20image%2020241106114205.png)

Bundan sonra, Authorities (Yetkililer) sekmesini seçebilir ve ardından import (içe aktar) seçeneğine tıklayıp indirilen CA sertifikasını seçebiliriz:

![Pasted image 20241106114327.png](/img/user/resimler/Pasted%20image%2020241106114327.png)

Son olarak, Web sitelerini tanımlamak için bu CA'ya güven ve e-posta kullanıcılarını tanımlamak için bu CA'ya güven öğelerini seçmeli ve ardından Tamam'ı tıklatmalıyız:

![Pasted image 20241106114345.png](/img/user/resimler/Pasted%20image%2020241106114345.png)
Sertifikayı yükledikten ve Firefox proxy'sini yapılandırdıktan sonra, tüm Firefox web trafiği web proxy'miz üzerinden yönlendirilmeye başlayacaktır.


Soru 1 : Yukarıda gösterilen sunucudaki ping isteğini yakalamayı deneyin ve gönderi verilerini bu bölümde yaptığımıza benzer şekilde değiştirin. Komutu 'flag.txt' okuyacak şekilde değiştirin
![Pasted image 20241106114955.png](/img/user/resimler/Pasted%20image%2020241106114955.png)


### Intercepting Responses
Bazı durumlarda, sunucudan gelen HTTP yanıtlarını tarayıcıya ulaşmadan önce yakalamamız gerekebilir. Bu, belirli bir web sayfasının nasıl göründüğünü değiştirmek istediğimizde, belirli devre dışı alanları etkinleştirmek veya belirli gizli alanları göstermek gibi, sızma testi faaliyetlerimizde bize yardımcı olabilecek yararlı olabilir.


### Burp
Burp'ta, (Proxy>Options) seçeneğine gidip Intercept Server Responses altında Intercept Response seçeneğini etkinleştirerek yanıt durdurmayı etkinleştirebiliriz:

![Pasted image 20241106115728.png](/img/user/resimler/Pasted%20image%2020241106115728.png)

Bundan sonra, istek engellemeyi bir kez daha etkinleştirebilir ve tarayıcımızda [CTRL+SHIFT+R] ile sayfayı yenileyebiliriz (tam bir yenilemeye zorlamak için). Burp'a geri döndüğümüzde, durdurulan isteği görmeliyiz ve forward'e tıklayabiliriz. İsteği ilettiğimizde, ele geçirilen yanıtımızı göreceğiz:

![Pasted image 20241106115829.png](/img/user/resimler/Pasted%20image%2020241106115829.png)

Şimdi 27. satırdaki type=“number” ifadesini type=“text” olarak değiştirmeyi deneyelim, bu sayede istediğimiz değeri yazabileceğiz. Ayrıca maxlength=“3” değerini maxlength=“100” olarak değiştireceğiz, böylece daha uzun girdi girebileceğiz:

```html
<input type="text" id="ip" name="ip" min="1" max="255" maxlength="100"
    oninput="javascript: if (this.value.length > this.maxLength) this.value = this.value.slice(0, this.maxLength);"
    required>
```

Now, once we click on `forward` again, we can go back to Firefox to examine the edited response:

![Pasted image 20241106115951.png](/img/user/resimler/Pasted%20image%2020241106115951.png)

Gördüğümüz gibi, sayfanın tarayıcı tarafından işlenme şeklini değiştirebilir ve artık istediğimiz herhangi bir değeri girebiliriz. Aynı tekniği, HTML kodlarını değiştirerek devre dışı bırakılmış HTML düğmelerini kalıcı olarak etkinleştirmek için de kullanabiliriz.


### Otomatik Modifikasyon
Belirli durumlarda tüm giden HTTP isteklerine veya tüm gelen HTTP yanıtlarına belirli değişiklikler uygulamak isteyebiliriz. Bu durumlarda, belirlediğimiz kurallara dayalı otomatik değişiklikleri kullanabiliriz, böylece web proxy araçları bunları otomatik olarak uygular.


### Automatic Request Modification
Otomatik istek değiştirme örneği ile başlayalım. İsteklerimizdeki herhangi bir metni, istek başlığında veya istek gövdesinde eşleştirmeyi seçebilir ve ardından bunları farklı bir metinle değiştirebiliriz. Gösterim amacıyla, User-Agent'ımızı HackTheBox Agent 1.0 ile değiştirelim; bu, belirli User-Agent'ları engelleyen filtrelerle uğraşabileceğimiz durumlarda kullanışlı olabilir.

### Burp Eşleştirme ve Değiştirme
(Proxy>Options>Match and Replace) kısmına gidip Add in Burp seçeneğine tıklayabiliriz. Aşağıdaki ekran görüntüsünde gösterildiği gibi, aşağıdaki seçenekleri ayarlayacağız:
![Pasted image 20241106133828.png](/img/user/resimler/Pasted%20image%2020241106133828.png)

![Pasted image 20241106134613.png](/img/user/resimler/Pasted%20image%2020241106134613.png)

Yukarıdaki seçenekleri girip Tamam'a tıkladığımızda, yeni Match and Replace seçeneğimiz eklenip etkinleştirilecek ve isteklerimizdeki User-Agent başlığını otomatik olarak yeni User-Agent'ımızla değiştirmeye başlayacaktır. Bunu, önceden yapılandırılmış Burp tarayıcısını kullanarak herhangi bir web sitesini ziyaret ederek ve ele geçirilen isteği inceleyerek doğrulayabiliriz. User-Agent'ımızın gerçekten de otomatik olarak değiştirildiğini göreceğiz:

![Pasted image 20241106134711.png](/img/user/resimler/Pasted%20image%2020241106134711.png)



## Proxy History
![Pasted image 20241106135226.png](/img/user/resimler/Pasted%20image%2020241106135226.png)

Soru  : Komutları hızlı bir şekilde test edebilmek için istek yinelemeyi kullanmayı deneyin. Bununla birlikte, diğer bayrağı aramayı deneyin.
Cevap : 
![Pasted image 20241106135705.png](/img/user/resimler/Pasted%20image%2020241106135705.png)

![Pasted image 20241106140017.png](/img/user/resimler/Pasted%20image%2020241106140017.png)


### Encoding/Decoding

* Spaces: Kodlanmamışsa istek verisinin sonunu gösterebilir
* &: Aksi takdirde parametre sınırlayıcısı olarak yorumlanır
* #: Aksi takdirde bir fragman tanımlayıcısı olarak yorumlanır

Çok sayıda özel karakter içeren istekler için yararlı olabilecek Tam URL Kodlama veya Unicode URL kodlama gibi başka URL kodlama türleri de vardır.

* HTML
* Unicode
* Base64
* ASCII hex


Soru :Ekteki dosyada bulunan dize, çeşitli kodlayıcılarla birkaç kez kodlanmıştır. Kodunu çözmek ve bayrağı almak için tartıştığımız kod çözme araçlarını kullanmayı deneyin.
Cevap  : HTB{3nc0d1n6_n1nj4}
* 3 kere base64 1 kerede url encode ile kodlanmış.

# Proxying Tools

Proxychains'i kullanmak için öncelikle /etc/proxychains.conf dosyasını düzenlememiz, son satırı yorumsuz bırakmamız ve sonuna aşağıdaki satırı eklememiz gerekir:

```shell-session
#socks4         127.0.0.1 9050
http 127.0.0.1 8080
```

Ayrıca quiet_mode yorumunu kaldırarak gürültüyü azaltmak için Quiet Mode'u etkinleştirmeliyiz. Bunu yaptıktan sonra, herhangi bir komuta proxychains ekleyebiliriz ve bu komutun trafiği proxychains (yani web proxy'miz) üzerinden yönlendirilmelidir. Örneğin, önceki alıştırmalarımızdan birinde cURL kullanmayı deneyelim:

```shell-session
M1R4CKCK@htb[/htb]$ proxychains curl http://SERVER_IP:PORT

ProxyChains-3.1 (http://proxychains.sf.net)
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <title>Ping IP</title>
    <link rel="stylesheet" href="./style.css">
</head>
...SNIP...
</html>    
```

ProxyChains üzerinden yönlendirildiğini not etmek için başlangıçta ek ProxyChains-3.1 satırı ile normalde olduğu gibi çalıştığını görüyoruz. Web proxy'mize (bu durumda Burp) geri dönersek, isteğin gerçekten de onun üzerinden geçtiğini göreceğiz:

![Pasted image 20241107015900.png](/img/user/resimler/Pasted%20image%2020241107015900.png)

### Nmap

Gördüğümüz gibi --proxies bayrağını kullanabiliriz. Ayrıca host keşfini atlamak için -Pn bayrağını da eklemeliyiz (man sayfasında önerildiği gibi). Son olarak, bir nmap script taramasının ne yaptığını incelemek için -sC bayrağını da kullanacağız:

```shell-session
M1R4CKCK@htb[/htb]$ nmap --proxies http://127.0.0.1:8080 SERVER_IP -pPORT -Pn -sC

Starting Nmap 7.91 ( https://nmap.org )
Nmap scan report for SERVER_IP
Host is up (0.11s latency).

PORT      STATE SERVICE
PORT/tcp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 0.49 seconds
```

Bir kez daha, web proxy aracımıza gidersek, nmap tarafından yapılan tüm istekleri proxy geçmişinde göreceğiz:

![Pasted image 20241107015956.png](/img/user/resimler/Pasted%20image%2020241107015956.png)

Not: Nmap'in built-in proxy'si, kılavuzunda (man nmap) belirtildiği gibi hala deneysel aşamasındadır, bu nedenle tüm fonksiyonlar veya trafik proxy üzerinden yönlendirilmeyebilir. Bu durumlarda, daha önce yaptığımız gibi proxychains'e başvurabiliriz.

## Metasploit
Metasploit içindeki herhangi bir exploit için bir proxy ayarlamak için set PROXIES bayrağını kullanabiliriz. Örnek olarak robots_txt tarayıcısını deneyelim ve önceki alıştırmalarımızdan birine karşı çalıştıralım:

```shell-session
M1R4CKCK@htb[/htb]$ msfconsole

msf6 > use auxiliary/scanner/http/robots_txt
msf6 auxiliary(scanner/http/robots_txt) > set PROXIES HTTP:127.0.0.1:8080

PROXIES => HTTP:127.0.0.1:8080


msf6 auxiliary(scanner/http/robots_txt) > set RHOST SERVER_IP

RHOST => SERVER_IP


msf6 auxiliary(scanner/http/robots_txt) > set RPORT PORT

RPORT => PORT


msf6 auxiliary(scanner/http/robots_txt) > run

[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Bir kez daha, tercih ettiğimiz web proxy aracına geri dönebilir ve gönderilen tüm istekleri görüntülemek için proxy geçmişini inceleyebiliriz:

![Pasted image 20241107020101.png](/img/user/resimler/Pasted%20image%2020241107020101.png)

Soru : Trafiği Burp üzerinden yönlendirirken, herhangi bir web sitesinde Metasploit'te 'auxiliary/scanner/http/http_put' çalıştırmayı deneyin. Gönderilen istekleri görüntülediğinizde, istekteki son satır nedir?
cevap : msf test file
![Pasted image 20241107020527.png](/img/user/resimler/Pasted%20image%2020241107020527.png)


# Burp Intruder

### Payloadlar
Üçüncü sekme olan 'Payloads'ta Payload'larımızı/wordlist'lerimizi seçer ve özelleştiririz. Bu payload/wordlist, üzerinde yinelenecek olan şeydir ve her bir öğesi/satırı, daha önce seçtiğimiz Payload Konumuna tek tek yerleştirilecek ve test edilecektir. Yapılandırmamız gereken dört ana şey var:

* Payload Setleri
* Payload Seçenekleri
* Payload İşleme
* Payload Kodlaması

##### Payload Sets
Yapılandırmamız gereken ilk şey Payload Set'tir. Payload seti, Payload Position Pointers'da kullandığımız saldırı türüne ve Payload sayısına bağlı olarak Payload sayısını tanımlar:

![Pasted image 20241107163142.png](/img/user/resimler/Pasted%20image%2020241107163142.png)

Bu durumda, sadece bir payload pozisyonu ile 'Sniper' Saldırı tipini seçtiğimiz için sadece bir Payload Setimiz var. Örneğin ' Cluster Bomb' saldırı türünü seçmiş ve birkaç faydalı payload pozisyonu eklemiş olsaydık, aralarından seçim yapabileceğimiz daha fazla payload seti elde eder ve her biri için farklı seçenekler seçerdik. Bizim durumumuzda, payload seti için 1'i seçeceğiz.

Ardından, kullanacağımız payloadların/sözcük listelerinin türü olan Payload Tipini seçmemiz gerekiyor. Burp, her biri belirli bir şekilde hareket eden çeşitli Payload Tipleri sağlar. Ardından, kullanacağımız payloadların/sözcük listelerinin türü olan Payload Tipini seçmemiz gerekir:

* Simple List (Basit Liste): Temel ve en temel türdür. Bir kelime listesi sağlarız ve Intruder bu listedeki her satır üzerinde yineleme yapar.

* Runtime file: Basit Listeye benzer, ancak Burp tarafından aşırı bellek kullanımını önlemek için tarama çalışırken satır satır yüklenir.

* Character Substitution: Bir karakter listesi ve bunların yerlerini belirtmemizi sağlar ve Burp Intruder tüm olası permütasyonları dener.

Her biri kendi seçeneklerine sahip olan ve birçoğu her saldırı için özel kelime listeleri oluşturabilen başka birçok Payload Türü vardır. Her bir Payload Türü hakkında daha fazla bilgi edinmek için Payload Setlerinin yanındaki ? işaretine ve ardından Payload Türüne tıklamayı deneyin. Bizim durumumuzda, temel bir Simple List ile devam edeceğiz.


### Payload Options
Daha sonra, Payload Set'lerinde seçtiğimiz her Payload Tipi için farklı olan Payload Seçeneklerini belirtmeliyiz. Bir Simple List için, bir kelime listesi oluşturmamız veya yüklememiz gerekir. Bunu yapmak için, kelime listemizi anında oluşturacak olan Ekle'ye tıklayarak her bir öğeyi manuel olarak girebiliriz. Daha yaygın olan diğer seçenek ise Yükle'ye tıklamak ve ardından Burp Intruder'a yüklenecek bir dosya seçmektir.

Biz kelime listemiz olarak /opt/useful/seclists/Discovery/Web-Content/common.txt dosyasını seçeceğiz. Burp Intruder'ın kelime listemizin tüm satırlarını Payload Options tablosuna yüklediğini görebiliriz:

![Pasted image 20241107163536.png](/img/user/resimler/Pasted%20image%2020241107163536.png)

Başka bir kelime listesi ekleyebilir veya manuel olarak birkaç öğe ekleyebiliriz ve bunlar aynı öğe listesine eklenir. Bunu birden fazla kelime listesini birleştirmek veya özelleştirilmiş kelime listeleri oluşturmak için kullanabiliriz. Burp Pro'da, Listeden ekle menü seçeneğinden seçim yaparak Burp'ta bulunan mevcut kelime listelerinin bir listesinden de seçim yapabiliriz.

İpucu: Çok büyük bir kelime listesi kullanmak istiyorsanız, Basit Liste yerine Runtime dosyasını Payload Türü olarak kullanmak en iyisidir, böylece Burp Intruder'ın tüm kelime listesini önceden yüklemesi gerekmez, bu da bellek kullanımını azaltabilir.

### Payload Processing
Uygulayabileceğimiz bir diğer seçenek ise yüklenen kelime listesi üzerinden fuzzing kuralları belirlememizi sağlayan Payload Processing'dir. Örneğin, payload öğemizden sonra bir uzantı eklemek istersek veya kelime listesini belirli kriterlere göre filtrelemek istersek, bunu payload işleme ile yapabiliriz.

. ile başlayan tüm satırları atlayan bir kural eklemeyi deneyelim (daha önceki kelime listesi ekran görüntüsünde gösterildiği gibi). Bunu, Add (Ekle) düğmesine tıklayıp Skip if matches regex (regex ile eşleşirse atla) seçeneğini seçerek yapabiliriz; bu da atlamak istediğimiz öğeler için bir regex kalıbı sağlamamıza olanak tanır. Ardından, . ile başlayan satırlarla eşleşen bir regex deseni sağlayabiliriz: ^\..*$:

![Pasted image 20241107163954.png](/img/user/resimler/Pasted%20image%2020241107163954.png)

Kuralımızın eklendiğini ve etkinleştirildiğini görebiliriz:

![Pasted image 20241107164013.png](/img/user/resimler/Pasted%20image%2020241107164013.png)


### Örnek Durum

Örneğin, elinde şöyle bir kelime listesi olduğunu düşün:

![Pasted image 20241107164151.png](/img/user/resimler/Pasted%20image%2020241107164151.png)

Bu durumda, `^\..*$` regex deseni ile **“.admin”** ve **“.test”** satırları atlanacak, yalnızca **“user1”**, **“admin”** ve **“testuser”** gibi “. (nokta)” ile başlamayan satırlar payload olarak kullanılacaktır.

Bu şekilde, belirli desenlere göre payloadları işleyebilir veya filtreleyebilirsin.


### Payload Encoding
Uygulayabileceğimiz dördüncü ve son seçenek, Payload URL kodlamasını etkinleştirmemizi veya devre dışı bırakmamızı sağlayan Payload Encoding'dir.

![Pasted image 20241107164849.png](/img/user/resimler/Pasted%20image%2020241107164849.png)

Etkin bırakacağız.

### Options
Son olarak, saldırı seçeneklerimizi Options (Seçenekler) sekmesinden özelleştirebiliriz. Saldırımız için özelleştirebileceğimiz (veya varsayılan olarak bırakabileceğimiz) birçok seçenek var. Örneğin, başarısızlık durumunda yeniden deneme sayısını ve yeniden denemeden önce duraklatma sayısını 0 olarak ayarlayabiliriz.

Bir başka kullanışlı seçenek de, yanıtlarına bağlı olarak belirli istekleri işaretlememizi sağlayan Grep - Match'tir. Web dizinlerini fuzzing yaptığımız için, yalnızca HTTP kodu 200 OK olan yanıtlarla ilgileniyoruz. Bu nedenle, önce etkinleştireceğiz ve ardından mevcut listeyi temizlemek için Temizle'ye tıklayacağız. Bundan sonra, bu dizgiye sahip tüm istekleri eşleştirmek için 200 OK yazabilir ve yeni kuralı eklemek için Ekle'ye tıklayabiliriz. Son olarak, aradığımız şey HTTP başlığında olduğu için HTTP Header'larını hariç tut seçeneğini de devre dışı bırakacağız:

![Pasted image 20241107165011.png](/img/user/resimler/Pasted%20image%2020241107165011.png)

Ayrıca, HTTP yanıtları uzunsa ve yanıtın yalnızca belirli bir kısmıyla ilgileniyorsak yararlı olan Grep - Extract seçeneğini de kullanabiliriz. Yani, bu bize yanıtın yalnızca belirli bir bölümünü göstermede yardımcı olur. İçeriğinden bağımsız olarak yalnızca HTTP Kodu 200 OK olan yanıtları arıyoruz, bu nedenle bu seçeneği tercih etmeyeceğiz.

Not: Intruder'ın ne kadar ağ kaynağı kullanacağını belirtmek için Resource Pool sekmesini de kullanabiliriz, bu çok büyük saldırılar için yararlı olabilir. Örneğimiz için bunu varsayılan değerlerinde bırakacağız.


### Attack
![Pasted image 20241107165155.png](/img/user/resimler/Pasted%20image%2020241107165155.png)

Options sekmesinde belirlediğimiz 200 OK grep değeriyle eşleşen istekleri gösteren 200 OK sütununu da görebiliriz. Eşleşen sonuçları en üstte olacak şekilde sıralamak için üzerine tıklayabiliriz. Aksi takdirde, duruma veya Uzunluğa göre sıralayabiliriz. Taramamız tamamlandığında, bir /admin isabeti aldığımızı görüyoruz:

![Pasted image 20241107165246.png](/img/user/resimler/Pasted%20image%2020241107165246.png)

Soru : Bayrağı içeren bir dosya bulmak için /admin dizini altındaki '.html' dosyalarını bulanıklaştırmak için Burp Intruder'ı kullanın.
Cevap /usr/share/dirb/wordlists/common.txt

![Pasted image 20241107170746.png](/img/user/resimler/Pasted%20image%2020241107170746.png)

![Pasted image 20241107170811.png](/img/user/resimler/Pasted%20image%2020241107170811.png)


# ZAP Fuzzer

Soru : Yukarıda bulduğumuz dizin, (misafir) kullanıcı için istekte md5 cookie'sini görebildiğimiz gibi, çerezi kullanıcı adının md5 karmasına ayarlar. Çerezli bir istek almak için '/skills/' adresini ziyaret edin, ardından bayrağı almak için farklı md5 karması kullanıcı adları için çerezi bulanıklaştırmak için ZAP Fuzzer'ı kullanmayı deneyin. Seclists'ten “top-usernames-shortlist.txt” kelime listesini kullanın.

1- verilen wordlisteki kelime listesini md5 hashlere dönüştürdük. 
2- burp ile fuzzladık .
3- yüksek lenght olan değeri bulduk ve bayrağı aldık . 



### Burp Scanner
Web proxy araçlarının önemli bir özelliği de web tarayıcılarıdır. Burp Suite, web sitesi yapısını oluşturmak için bir Crawler ve pasif ve aktif tarama için Tarayıcı kullanarak çeşitli web güvenlik açıkları için güçlü bir tarayıcı olan Burp Scanner ile birlikte gelir.

Burp Scanner Pro-Only bir özelliktir ve Burp Suite'in ücretsiz Topluluk sürümünde mevcut değildir. Bununla birlikte, Burp Scanner'ın kapsadığı geniş kapsam ve içerdiği gelişmiş özellikler göz önüne alındığında, onu kurumsal düzeyde bir araç haline getirir ve bu nedenle ücretli bir özellik olması beklenir.


### Target Scope
Burp Suite'te bir tarama başlatmak için aşağıdaki seçeneklere sahibiz:
* Proxy History'den belirli bir istek üzerine tarama başlatma
* Bir dizi hedef üzerinde yeni bir tarama başlatın
* Scope dahilindeki öğeler üzerinde bir tarama başlatın

Proxy Geçmişi'nden belirli bir istek üzerinde tarama başlatmak için, geçmişte bulduktan sonra üzerine sağ tıklayabilir ve ardından taramayı çalıştırmadan önce yapılandırabilmek için Tara'yı seçebilir veya varsayılan yapılandırmalarla hızlı bir şekilde tarama başlatmak için Pasif/Aktif Tarama'yı seçebiliriz:

![Pasted image 20241107175554.png](/img/user/resimler/Pasted%20image%2020241107175554.png)

Ayrıca, bir dizi özel hedef üzerinde bir tarama yapılandırmak için Dashboard sekmesindeki New Scan (Yeni Tarama) düğmesine tıklayarak New Scan (Yeni Tarama) yapılandırma penceresini açabiliriz. Sıfırdan özel bir tarama oluşturmak yerine, Target Scope'u kullanarak taramalarımıza neyin dahil edileceğini/hariç tutulacağını doğru bir şekilde tanımlamak için kapsamı nasıl kullanabileceğimizi görelim. Target Scope, işlenecek özel bir hedef kümesi tanımlamak için tüm Burp özellikleriyle birlikte kullanılabilir. Burp ayrıca, kapsam dışı URL'leri yok sayarak kaynakları korumak için Burp'u kapsam içi öğelerle sınırlamamıza olanak tanır.

Not: Bir sonraki bölümün sonunda bulunan alıştırmadaki web uygulamasını tarayacağız. Burp Pro kullanmak için bir lisans alırsanız, bir sonraki bölümün sonundaki hedefi oluşturabilir ve burada takip edebilirsiniz.

( Target>Site haritası)'na gidersek, burp'un proxy'sinden geçen çeşitli isteklerde tespit ettiği tüm dizinlerin ve dosyaların bir listesini gösterecektir:

![Pasted image 20241107175759.png](/img/user/resimler/Pasted%20image%2020241107175759.png)

Bir öğeyi kapsamımıza eklemek için üzerine sağ tıklayabilir ve Kapsama ekle'yi seçebiliriz:

![Pasted image 20241107175816.png](/img/user/resimler/Pasted%20image%2020241107175816.png)

Not: Kapsamınıza ilk öğeyi eklediğinizde, Burp size özelliklerini yalnızca kapsam içi öğelerle kısıtlama ve kapsam dışı öğeleri yok sayma seçeneği sunacaktır.

Ayrıca, taranması tehlikeli olabilecek veya oturumumuzu ' logout fonksiyonu gibi' sonlandırabilecek birkaç öğeyi kapsam dışında bırakmamız gerekebilir. Bir öğeyi kapsamımızdan çıkarmak için, kapsam dahilindeki herhangi bir öğeye sağ tıklayabilir ve Kapsamdan çıkar'ı seçebiliriz. Son olarak, kapsamımızın ayrıntılarını görüntülemek için ( Target>Scope) seçeneğine gidebiliriz. Burada, diğer öğeleri de ekleyebilir/çıkarabilir ve dahil edilecek/hariç tutulacak regex kalıplarını belirtmek için gelişmiş kapsam kontrolünü kullanabiliriz.

![Pasted image 20241107175918.png](/img/user/resimler/Pasted%20image%2020241107175918.png)

### Crawler
Kapsamımızı hazırladıktan sonra, Dashboard sekmesine gidebilir ve kapsam içi öğelerimizle otomatik olarak doldurulacak olan taramamızı yapılandırmak için New Scan (Yeni Tarama) seçeneğine tıklayabiliriz:
![Pasted image 20241107180002.png](/img/user/resimler/Pasted%20image%2020241107180002.png)

Burp'ün bize iki tarama seçeneği sunduğunu görüyoruz: Crawl ve Audit and Crawl. Bir Web Tarayıcısı, sayfalarında bulunan tüm bağlantılara erişerek, tüm formlara erişerek ve web sitesinin kapsamlı bir haritasını oluşturmak için yaptığı tüm istekleri inceleyerek bir web sitesinde gezinir. Sonunda, Burp Scanner bize hedefin bir haritasını sunar ve herkese açık tüm verileri tek bir yerde gösterir. Crawl and Audit'i seçersek, Burp tarayıcısını Crawler'dan sonra çalıştıracaktır (daha sonra göreceğimiz gibi).

Not: Bir Crawl taraması yalnızca belirttiğimiz sayfada bulunan bağlantıları ve bu sayfada bulunan tüm sayfaları takip eder ve eşler. Dirbuster ya da ffuf'un yaptığı gibi hiç referans verilmeyen sayfaları belirlemek için bir bulanıklaştırma taraması yapmaz. Bu, Burp Intruder veya Content Discovery ile yapılabilir ve ardından gerekirse kapsama eklenebilir.

Başlangıç olarak Crawl'ı seçelim ve taramamızı yapılandırmak için Tarama yapılandırması sekmesine gidelim. Buradan, tarama hızı veya sınırı, Burp'un herhangi bir oturum açma formunda oturum açmaya çalışıp çalışmayacağı ve diğer birkaç yapılandırma gibi yapılandırmaları ayarlamamıza olanak tanıyan özel bir yapılandırma oluşturmak için Yeni'ye tıklamayı seçebiliriz. Basitlik adına, bize seçebileceğimiz birkaç önceden ayarlanmış yapılandırma (veya daha önce tanımladığımız özel yapılandırmalar) veren Kitaplıktan seç düğmesine tıklayacağız:

![Pasted image 20241107180211.png](/img/user/resimler/Pasted%20image%2020241107180211.png)

`Crawl strategy - fastest` seçeneğini seçeceğiz ve `Application login` sekmesine devam edeceğiz. Bu sekmede, Burp'ün bulabileceği herhangi bir Login formunda/alanında denemesi için bir dizi kimlik bilgisi ekleyebiliriz. Ayrıca, önceden yapılandırılmış tarayıcıda manuel bir oturum açma gerçekleştirerek bir dizi adım kaydedebiliriz, böylece Burp bir oturum açma oturumu kazanmak için hangi adımları izleyeceğini bilir. Bu, taramamızı kimliği doğrulanmış bir kullanıcı kullanarak yürütüyorsak önemli olabilir, bu da web uygulamasının Burp'ün başka türlü erişemeyeceği bölümlerini kapsamamıza izin verir. Herhangi bir kimlik bilgimiz olmadığı için boş bırakacağız.  

Bununla birlikte, Tarama taramamızı başlatmak için `Tamam` düğmesine tıklayabiliriz. Taramamız başladığında, ilerlemesini `Dashboard` sekmesinde `Tasks (Görevler`) altında görebiliriz:

![Pasted image 20241107180317.png](/img/user/resimler/Pasted%20image%2020241107180317.png)

Ayrıca, çalışan tarama hakkında daha fazla ayrıntı görmek için görevlerdeki `View details (Ayrıntıları` görüntüle) düğmesine tıklayabilir veya tarama yapılandırmalarımızı daha da özelleştirmek için dişli simgesine tıklayabiliriz. Son olarak, taramamız tamamlandığında, görev bilgilerinde `Tarama` Tamamlandı ifadesini göreceğiz ve ardından güncellenmiş site haritasını görüntülemek için (`Hedef>Site` haritası ) seçeneğine geri dönebiliriz:

![Pasted image 20241107180349.png](/img/user/resimler/Pasted%20image%2020241107180349.png)

## **Passive Scanner**  

Artık site haritası tamamen oluşturulduğuna göre, bu hedefi olası güvenlik açıklarına karşı taramayı seçebiliriz. `New Scan` iletişim kutusunda `Crawl and Audit` seçeneğini seçtiğimizde, Burp iki tür `tarama` gerçekleştirecektir: `Pasif Güvenlik Açığı Taraması` ve `Aktif Güvenlik Açığı Taraması`.  

Aktif Taramanın aksine, Pasif Tarama herhangi bir yeni istek göndermez, ancak hedefte/kapsamda zaten ziyaret edilen sayfaların kaynağını analiz eder ve ardından `olası` güvenlik açıklarını belirlemeye çalışır. Bu, eksik HTML etiketleri veya potansiyel DOM tabanlı XSS güvenlik açıkları gibi belirli bir hedefin hızlı bir analizi için çok kullanışlıdır. Bununla birlikte, bu güvenlik açıklarını test etmek ve doğrulamak için herhangi bir istek göndermeden, Pasif Tarama yalnızca potansiyel güvenlik açıklarının bir listesini önerebilir. Yine de Burp Pasif Tarayıcı, tespit edilen her güvenlik açığı için bir `Güven` düzeyi sağlar ve bu da potansiyel güvenlik açıklarının önceliklendirilmesinde yardımcı olur.  

Sadece bir Pasif Tarama gerçekleştirmeyi deneyerek başlayalım. Bunu yapmak için, bir kez daha (`Target>Site haritası`) 'ndaki hedefi veya Burp Proxy Geçmişindeki bir isteği seçebilir, ardından üzerine sağ tıklayabilir ve `Passive scan` ( `Pasif tarama yap` ) veya `Passively scan this target (Bu hedefi` `pasif` `olarak` `tara` ) seçeneğini seçebiliriz. Pasif Tarama çalışmaya başlayacaktır ve görevi Gösterge `Tablosu` sekmesinde de görülebilir. Tarama bittiğinde, belirlenen güvenlik açıklarını incelemek için `Ayrıntıları Görüntüle` 'ye tıklayabilir ve ardından `Issue activity (Sorun etkinliği` ) sekmesini seçebiliriz:

![Pasted image 20241107180526.png](/img/user/resimler/Pasted%20image%2020241107180526.png)

Alternatif olarak, tanımlanan tüm sorunları `Dashboard` sekmesindeki `Issue activity` bölmesinde görüntüleyebiliriz. Gördüğümüz gibi, potansiyel güvenlik açıklarının listesini, önem derecelerini ve güvenirliklerini gösterir. Genellikle, `Yüksek` önem derecesine ve `Belirli` güvene sahip güvenlik açıklarını aramak isteriz. Ancak, çok hassas web uygulamaları için, özellikle `Yüksek` önem derecesi ve `Gizli/Firma` güvenine odaklanarak, tüm önem derecesi ve güven düzeylerini dahil etmeliyiz.


## **Active Scanner**

Sonunda Burp Scanner'ın en güçlü kısmı olan Aktif Güvenlik Açığı Tarayıcısına ulaşıyoruz. Aktif bir tarama, Pasif Tarama'dan daha kapsamlı bir taramayı aşağıdaki gibi çalıştırır:  

1. Tüm olası sayfaları tanımlamak için bir Crawl ve bir web fuzzer (dirbuster/ffuf gibi) çalıştırarak başlar  
    
2. Tanımlanan tüm sayfalarda bir Pasif Tarama çalıştırır  
    
3. Pasif Tarama'dan tanımlanan güvenlik açıklarının her birini kontrol eder ve bunları doğrulamak için talepler gönderir  
    
4. Daha fazla potansiyel güvenlik açığını belirlemek için bir JavaScript analizi gerçekleştirir  
    
5. XSS, Komut Enjeksiyonu, SQL Enjeksiyonu ve diğer yaygın web güvenlik açıklarını aramak için çeşitli tanımlanmış ekleme noktalarını ve parametreleri bulanıklaştırır

Burp Active tarayıcısı bu alandaki en iyi araçlardan biri olarak kabul edilir ve Burp araştırma ekibi tarafından yeni tanımlanan web güvenlik açıklarını taramak için sık sık güncellenir.  

Burp Proxy Geçmişindeki bir istek üzerine sağ tıklama menüsünden Active Scan `yap` seçeneğini seçerek Pasif Tarama başlattığımıza benzer şekilde Aktif Tarama başlatabiliriz. Alternatif olarak, aktif taramamızı yapılandırmamıza olanak tanıyan `Dashboard` sekmesindeki `Yeni` Tarama düğmesiyle kapsamımızda bir tarama çalıştırabiliriz. Bu kez, yukarıdaki tüm noktaları ve şu ana kadar tartıştığımız her şeyi gerçekleştirecek olan `Crawl and Audit` seçeneğini seçeceğiz.  

Ayrıca Tarama yapılandırmalarını (daha önce tartıştığımız gibi) ve Denetim yapılandırmalarını da ayarlayabiliriz. Denetim yapılandırmaları, taramak istediğimiz güvenlik açıklarının türünü (varsayılan olarak hepsi), tarayıcının payloadlarını nereye eklemeye çalışacağını ve diğer birçok yararlı yapılandırmayı seçmemizi sağlar. Bir kez daha, `Select from library` (Kitaplıktan seç) düğmesi ile bir yapılandırma ön ayarı seçebiliriz. Testimiz için, backend sunucusu üzerinde kontrol sahibi olmamızı sağlayabilecek `Yüksek` güvenlik açıklarıyla ilgilendiğimizden, `Denetim kontrolleri - yalnızca kritik sorunlar` seçeneğini seçeceğiz. Son olarak, daha önce Crawl yapılandırmalarında gördüğümüz gibi oturum açma ayrıntılarını ekleyebiliriz.  

Yapılandırmalarımızı seçtikten sonra, taramayı başlatmak için `Tamam` düğmesine tıklayabiliriz ve etkin tarama görevi Gösterge `Tablosu` sekmesindeki `Görevler` bölmesine eklenmelidir:

![Pasted image 20241107180810.png](/img/user/resimler/Pasted%20image%2020241107180810.png)

Tarama yukarıda belirtilen tüm adımları çalıştıracaktır, bu nedenle seçtiğimiz yapılandırmalara bağlı olarak daha önceki taramalarımızdan çok daha uzun sürecektir. Tarama devam ederken, `Ayrıntıları` görüntüle düğmesine tıklayıp `Logger` sekmesini seçerek ya da Burp'teki `Logger` sekmesine giderek Burp tarafından yapılan ya da geçen tüm istekleri görüntüleyebiliriz:

![Pasted image 20241107180834.png](/img/user/resimler/Pasted%20image%2020241107180834.png)

Tarama tamamlandığında, şu ana kadar tespit edilen tüm sorunları görüntülemek ve filtrelemek için `Dashboard` sekmesindeki `Issue activity` bölmesine bakabiliriz. Sonuçların üzerindeki filtreden `High` ve `Certain` 'i seçelim ve filtrelenmiş sonuçlarımızı görelim:

![Pasted image 20241107180903.png](/img/user/resimler/Pasted%20image%2020241107180903.png)

Burp'ün `High` severity ve `Firm` confidence ile derecelendirilen bir `OS komut enjeksiyonu` güvenlik açığı tespit ettiğini görüyoruz. Burp bu ciddi güvenlik açığının varlığından kesin olarak emin olduğundan, güvenlik açığından yararlanılıp yararlanılamayacağını veya web sunucusunda nasıl bir tehdit oluşturduğunu öğrenebilmek için üzerine tıklayarak ve gösterilen danışmayı okuyarak ve gönderilen isteği ve alınan yanıtı görüntüleyerek güvenlik açığı hakkında bilgi edinebiliriz:

![Pasted image 20241107180936.png](/img/user/resimler/Pasted%20image%2020241107180936.png)

## **Reporting**  

Son olarak, tüm taramalarımız tamamlandıktan ve tüm olası sorunlar belirlendikten sonra, (`Target>Site map`)'e gidebilir, hedefimize sağ tıklayabilir ve (`Issue>Report issues for this host`)'u seçebiliriz. Rapor için dışa aktarma türünü ve rapora hangi bilgileri dahil etmek istediğimizi seçmemiz istenecektir. Raporu dışa aktardıktan sonra, ayrıntılarını görüntülemek için herhangi bir web tarayıcısında açabiliriz:

![Pasted image 20241107181007.png](/img/user/resimler/Pasted%20image%2020241107181007.png)

Gördüğümüz gibi, Burp'un raporu çok düzenli ve yalnızca önem derecesine / güvene göre belirli sorunları içerecek şekilde özelleştirilebilir. Ayrıca, güvenlik açığından nasıl yararlanılacağına dair kavram kanıtı ayrıntılarını ve güvenlik açığının nasıl giderileceğine ilişkin bilgileri de gösterir. Bu raporlar, bir web sızma testi gerçekleştirirken müşterilerimiz veya web uygulaması geliştiricileri için hazırladığımız ayrıntılı raporlar için tamamlayıcı veriler olarak kullanılabilir veya ileride başvurmak üzere saklanabilir. Hiçbir zaman sadece herhangi bir sızma aracından bir raporu dışa aktarmamalı ve nihai çıktı olarak müşteriye sunmamalıyız. Bunun yerine, araçlar tarafından üretilen raporlar ve veriler, iyileştirme çalışmaları için ham tarama verilerine ihtiyaç duyabilecek müşteriler için ek veriler olarak veya bir izleme panosuna aktarmak için yararlı olabilir.


### Skill assesment : 

Soru : /lucky.php sayfasında devre dışı gibi görünen bir düğme vardır. Düğmeyi etkinleştirmeyi deneyin ve ardından bayrağı almak için tıklayın.
Cevap.:  
1- script dosyasını disable seçeneğini silip isteği 10 kere yolladıktan sonra geliyor. 


Soru : admin.php sayfası birden çok kez kodlanmış bir cookie kullanır. 31-karakterli bir değer elde edene kadar cookie'nin kodunu çözmeye çalışın. Değeri cevap olarak gönderin.
CEvap : 

1- Önce hex decode --> sonra base64 decode

Soru : Cookie'nin kodunu çözdüğünüzde, sadece 31 karakter uzunluğunda olduğunu fark edeceksiniz, bu da son karakteri eksik bir md5 hash'i gibi görünüyor. Bu nedenle, her isteği yukarıda tanımladığınız kodlama yöntemleriyle kodlarken, çözülen md5 çerezinin son karakterini tüm alfa sayısal karakterlerle fuzz etmeye çalışın. ( Payload için Seclist'teki "alphanum-case.txt" kelime listesini kullanabilirsiniz)

1- Önceki bulduğumuz 31 uzunluğundaki md5 hash ' i sonuna 66 karakterin hepsini içeren alphanum txt dosyasını ekledik ve 32 karakter yaptık . 

2- ayrıca base64 encode yapıp ve hex encode yapıp yollamamız lazım .
![Pasted image 20241107192138.png](/img/user/resimler/Pasted%20image%2020241107192138.png)

Soru : Metasploit içindeki 'auxiliary/scanner/http/coldfusion_locale_traversal' aracını kullanıyorsunuz, ancak sizin için düzgün çalışmıyor. Metasploit tarafından gönderilen isteği yakalamaya karar veriyorsunuz, böylece manuel olarak doğrulayabilir ve tekrarlayabilirsiniz. İsteği yakaladığınızda, '/XXXXX/administrator/...' içinde çağrılan 'XXXXX' dizini nedir?

> msf6 >**search coldfusion_locale_traversa**l
> 
> msf6 >**use 0**
> 
> msf6 >**set RHOSTS 167.71.140.137**
> 
> msf6 >**set RPORT 30650**
> 
> msf6 >**set PROXIES HTTP:127.0.0.1:8080**
> 
> msf6 >**exploit**



# **Extensions**

Hem Burp hem de ZAP, Burp kullanıcıları topluluğunun Burp için herkesin kullanabileceği uzantılar geliştirebileceği şekilde uzantı yeteneklerine sahiptir. Bu tür uzantılar, örneğin yakalanan herhangi bir istek üzerinde belirli eylemler gerçekleştirebilir veya kod çözme ve güzelleştirme gibi yeni özellikler ekleyebilir. Burp, `Extender` özelliği ve [BApp Mağazası](https://portswigger.net/bappstore) aracılığıyla genişletilebilirliğe izin verirken, ZAP yeni eklentiler yüklemek için ZAP [Marketplace](https://www.zaproxy.org/addons/) 'e sahiptir.


## **BApp Store**

Mevcut tüm uzantıları bulmak için Burp içindeki `Extender` sekmesine tıklayabilir ve `BApp Store` alt sekmesini seçebiliriz. Bunu yaptığımızda, bir dizi uzantı göreceğiz. Bunları `Popülerliğe` göre sıralayabiliriz, böylece kullanıcıların hangilerini en kullanışlı bulduklarını biliriz:

![Pasted image 20241108004555.png](/img/user/resimler/Pasted%20image%2020241108004555.png)

Not: Bazı uzantılar yalnızca Pro kullanıcılar içindir, diğerleri ise herkes tarafından kullanılabilir.  

Birçok yararlı uzantı görüyoruz, bunları gözden geçirmek ve sizin için en yararlı olanları görmek için biraz zaman ayırın ve ardından bunları yüklemeyi ve test etmeyi deneyin. `Decoder Improved` uzantısını yüklemeyi deneyelim:

![Pasted image 20241108004620.png](/img/user/resimler/Pasted%20image%2020241108004620.png)

Not: Bazı uzantılar, `Jython` gibi Linux/macOS/Windows'ta varsayılan olarak genellikle yüklü olmayan gereksinimlere sahiptir, bu nedenle uzantıyı yükleyebilmek için önce bunları yüklemeniz gerekir.  

`Decoder Improved`'ı yükledikten sonra, Burp'a yeni sekmesinin eklendiğini göreceğiz. Her uzantının farklı bir kullanımı vardır, bu nedenle daha fazla bilgi edinmek için `BApp Store` 'daki herhangi bir uzantının belgelerine tıklayabilir veya kullanımı hakkında daha fazla bilgi için GitHub sayfasını ziyaret edebiliriz. Bu uzantıyı tıpkı Burp'ün Decoder'ını kullandığımız gibi kullanabiliriz, ancak birçok ek kodlayıcıya sahip olmanın avantajıyla. Örneğin, `MD5` ile hash edilmesini istediğimiz metni girebilir ve `Hash With>MD5`'i seçebiliriz:

![Pasted image 20241108004658.png](/img/user/resimler/Pasted%20image%2020241108004658.png)

Benzer şekilde, diğer kodlama ve hashing türlerini de gerçekleştirebiliriz. Burp'ün işlevselliğini daha da genişletmek için kullanılabilecek başka birçok Burp Uzantısı vardır.

Göz atmaya değer bazı uzantılar bunlarla sınırlı olmamakla birlikte şunları içerir:

![Pasted image 20241108004725.png](/img/user/resimler/Pasted%20image%2020241108004725.png)
