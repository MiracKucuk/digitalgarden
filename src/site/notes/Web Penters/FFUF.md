---
{"dg-publish":true,"permalink":"/web-penters/ffuf/"}
---


* Dizinler için Fuzzing
* Dosyalar ve uzantılar için fuzzing
* Gizli vhost'ları belirleme
* PHP parametreleri için fuzzing
* Parametre değerleri için fuzzing

İpucu: Bu kelime listesine baktığımızda, başlangıçta kelime listesinin bir parçası olarak kabul edilebilecek ve sonuçları karıştırabilecek telif hakkı yorumları içerdiğini fark edeceğiz. Bu satırlardan kurtulmak için aşağıdaki komutu **-ic** bayrağı ile kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -h

HTTP OPTIONS:
  -H               Header `"Name: Value"`, separated by colon. Multiple -H flags are accepted.
  -X               HTTP method to use (default: GET)
  -b               Cookie data `"NAME1=VALUE1; NAME2=VALUE2"` for copy as curl functionality.
  -d               POST data
  -recursion       Scan recursively. Only FUZZ keyword is supported, and URL (-u) has to end in it. (default: false)
  -recursion-depth Maximum recursion depth. (default: 0)
  -u               Target URL
...SNIP...

MATCHER OPTIONS:
  -mc              Match HTTP status codes, or "all" for everything. (default: 200,204,301,302,307,401,403)
  -ms              Match HTTP response size
...SNIP...

FILTER OPTIONS:
  -fc              Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fs              Filter HTTP response size. Comma separated list of sizes and ranges
...SNIP...

INPUT OPTIONS:
...SNIP...
  -w               Wordlist file path and (optional) keyword separated by colon. eg. '/path/to/wordlist:KEYWORD'

OUTPUT OPTIONS:
  -o               Write output to file
...SNIP...

EXAMPLE USAGE:
  Fuzz file paths from wordlist.txt, match all responses but filter out those with content-size 42.
  Colored, verbose output.
    ffuf -w wordlist.txt -u https://example.org/FUZZ -mc all -fs 42 -c -v
...SNIP...
```


## Directory Fuzzing

Yukarıdaki örnekten de görebileceğimiz gibi, ana iki seçenek kelime listeleri için `-w` ve URL için `-u`'dur. Bir kelime listesini, fuzz yapmak istediğimiz yerde referans vermek için bir anahtar kelimeye atayabiliriz. Örneğin, kelime listemizi seçebilir ve arkasına :FUZZ ekleyerek FUZZ anahtar kelimesini atayabiliriz:

Acelemiz varsa, örneğin **-t 200** ile thread sayısını **200'e** çıkararak daha hızlı çalışmasını sağlayabiliriz, ancak bu, özellikle uzak bir sitede kullanıldığında, siteyi bozabileceğinden ve Denial of Service'e neden olabileceğinden veya ciddi durumlarda internet bağlantınızı çökertebileceğinden önerilmez.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://SERVER_IP:PORT/FUZZ
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

<SNIP>
blog                    [Status: 301, Size: 326, Words: 20, Lines: 10]
:: Progress: [87651/87651] :: Job [1/1] :: 9739 req/sec :: Duration: [0:00:09] :: Errors: 0 ::
```

Boş bir sayfa alırız, bu da dizinin özel bir sayfası olmadığını gösterir, ancak 404 Not Found veya 403 Access Denied HTTP kodu almadığımız için dizine erişimimiz olduğunu da gösterir. Bir sonraki bölümde, gerçekten boş mu yoksa gizli dosya ve sayfalara mı sahip olduğunu görmek için bu dizin altındaki sayfalara bakacağız.


Soru : Yukarıda bulduğumuz dizine ek olarak bulunabilecek başka bir dizin daha vardır. Nedir bu dizin?

Cevap : forum

![Pasted image 20241224055217.png](/img/user/resimler/Pasted%20image%2020241224055217.png)


# Page Fuzzing

## Extension Fuzzing

Önceki bölümde, **/blog** dizinine erişimimiz olduğunu, ancak dizinin boş bir sayfa döndürdüğünü ve herhangi bir bağlantıyı veya sayfayı manuel olarak bulamadığımızı gördük. Bu nedenle, dizinin herhangi bir gizli sayfa içerip içermediğini görmek için bir kez daha web fuzzing kullanacağız. Ancak, başlamadan önce, web sitesinin **.html, .aspx, .php** veya başka bir şey gibi ne tür sayfalar kullandığını öğrenmeliyiz.

Bunu belirlemenin yaygın bir yolu, HTTP yanıt başlıkları aracılığıyla sunucu türünü bulmak ve uzantıyı tahmin etmektir. Örneğin, sunucu **apache** ise .**php** olabilir veya **IIS** ise **.asp** veya **.aspx** olabilir vb. Ancak bu yöntem çok pratik değildir. Bu nedenle, dizinleri fuzzladığımıza benzer şekilde uzantıyı fuzzlamak için yine ffuf kullanacağız. FUZZ anahtar kelimesini dizin adının olduğu yere yerleştirmek yerine, uzantının .FUZZ olacağı yere yerleştireceğiz ve yaygın uzantılar için bir kelime listesi kullanacağız. Uzantılar için SecLists'te aşağıdaki kelime listesini kullanabiliriz:

Fuzzing işlemine başlamadan önce, bu uzantının hangi dosyanın sonunda olacağını belirtmeliyiz! Her zaman iki kelime listesi kullanabilir ve her biri için benzersiz bir anahtar kelimeye sahip olabiliriz ve ardından her ikisini de fuzzlamak için **FUZZ_1.FUZZ_2** yapabiliriz. Bununla birlikte, çoğu web sitesinde her zaman bulabileceğimiz bir dosya vardır, bu da **index.*,** bu yüzden dosyamız olarak kullanacağız ve üzerindeki uzantıları fuzzlayacağız.

Not: Seçtiğimiz kelime listesi zaten bir nokta (.) içeriyor, bu nedenle fuzzing işlemimizde "index" kelimesinden sonra nokta eklememiz gerekmeyecek.

Şimdi, FUZZ anahtar sözcüğümüzü index'ten sonra uzantının olacağı yere dikkatlice yerleştirerek komutumuzu yeniden çalıştırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt:FUZZ -u http://SERVER_IP:PORT/blog/indexFUZZ


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://SERVER_IP:PORT/blog/indexFUZZ
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/web-extensions.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 5
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

.php                    [Status: 200, Size: 0, Words: 1, Lines: 1]
.phps                   [Status: 403, Size: 283, Words: 20, Lines: 10]
:: Progress: [39/39] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

Birkaç hit alıyoruz, ancak yalnızca .php bize 200 koduyla yanıt veriyor. Harika! Artık PHP dosyalarını fuzzing yapmaya başlamak için bu web sitesinin PHP ile çalıştığını biliyoruz.

## Page Fuzzing

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/blog/FUZZ.php


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://SERVER_IP:PORT/blog/FUZZ.php
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

index                   [Status: 200, Size: 0, Words: 1, Lines: 1]
REDACTED                [Status: 200, Size: 465, Words: 42, Lines: 15]
:: Progress: [87651/87651] :: Job [1/1] :: 5843 req/sec :: Duration: [0:00:15] :: Errors: 0 ::
```

![Pasted image 20241224055557.png](/img/user/resimler/Pasted%20image%2020241224055557.png)


Soru : Bu bölümde öğrendiklerinizi '/blog' dizinini fuzz'lamak ve tüm sayfaları bulmak için kullanmaya çalışın. İçlerinden biri bir bayrak içermelidir. Bayrak nedir?

Cevap : `HTB{bru73_f0r_c0mm0n_p455w0rd5}`

![Pasted image 20241224060037.png](/img/user/resimler/Pasted%20image%2020241224060037.png)

![Pasted image 20241224060010.png](/img/user/resimler/Pasted%20image%2020241224060010.png)


# Recursive Fuzzing

Tekrarlı tarama yaptığımızda, ana web sitesini ve tüm alt dizinlerini fuzz'layıncaya kadar sayfalarında olabilecek yeni tanımlanmış dizinler altında otomatik olarak başka bir tarama başlatır

ffuf'ta -**recursion** bayrağı ile recursive taramayı etkinleştirebilir ve **-recursion-depth** bayrağı ile derinliği belirleyebiliriz. Eğer ==**-recursion-depth 1**== olarak belirtirsek, sadece ana dizinleri ve onların doğrudan alt dizinlerini fuzzlayacaktır. Herhangi bir alt-alt-dizin tanımlanmışsa **(/login/user** gibi, sayfalar için onları Fuzzlamayacaktır). ffuf'ta recursion kullanırken, uzantımızı ==**-e .php**== ile belirtebiliriz

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://SERVER_IP:PORT/FUZZ -recursion -recursion-depth 1 -e .php -v


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://SERVER_IP:PORT/FUZZ
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt
 :: Extensions       : .php 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

[Status: 200, Size: 986, Words: 423, Lines: 56] | URL | http://SERVER_IP:PORT/
    * FUZZ: 

[INFO] Adding a new job to the queue: http://SERVER_IP:PORT/forum/FUZZ
[Status: 200, Size: 986, Words: 423, Lines: 56] | URL | http://SERVER_IP:PORT/index.php
    * FUZZ: index.php

[Status: 301, Size: 326, Words: 20, Lines: 10] | URL | http://SERVER_IP:PORT/blog | --> | http://SERVER_IP:PORT/blog/
    * FUZZ: blog

<...SNIP...>
[Status: 200, Size: 0, Words: 1, Lines: 1] | URL | http://SERVER_IP:PORT/blog/index.php
    * FUZZ: index.php

[Status: 200, Size: 0, Words: 1, Lines: 1] | URL | http://SERVER_IP:PORT/blog/
    * FUZZ: 

<...SNIP...>
```

Bu sefer görebileceğimiz gibi, tarama çok daha uzun sürdü, neredeyse altı kat daha fazla istek gönderildi ve kelime listesinin boyutu iki katına çıktı (.php ile bir kez ve bir kez olmadan). Yine de, daha önce belirlediğimiz tüm sonuçlar da dahil olmak üzere, tek bir komut satırıyla çok sayıda sonuç elde ettik.


Soru : Daha fazla dosya/dizin bulmak için şimdiye kadar öğrendiklerinizi tekrarlamaya çalışın. Bunlardan biri size bir bayrak vermelidir. Bayrağın içeriği nedir?

Cevap : `HTB{fuzz1n6_7h3_w3b!}`

![Pasted image 20241224061407.png](/img/user/resimler/Pasted%20image%2020241224061407.png)

![Pasted image 20241224061451.png](/img/user/resimler/Pasted%20image%2020241224061451.png)

![Pasted image 20241224061613.png](/img/user/resimler/Pasted%20image%2020241224061613.png)


# DNS Records

Blog altındaki sayfaya eriştiğimizde, Admin  panelinin academy.htb adresine taşındığını belirten bir mesaj aldık. Web sitesini tarayıcımızda ziyaret edersek, www.academy.htb adresindeki sunucuya bağlanamıyoruz:

![Pasted image 20241224124145.png](/img/user/resimler/Pasted%20image%2020241224124145.png)

```shell-session
$ sudo sh -c 'echo "SERVER_IP  academy.htb" >> /etc/hosts'
```

![Pasted image 20241224124226.png](/img/user/resimler/Pasted%20image%2020241224124226.png)

Testlerimizi bu IP üzerinde çalıştırdığımızda, hedefimiz üzerinde tam bir recursive tarama yaptığımızda bile admin veya paneller hakkında hiçbir şey bulamadık. Bu durumda, `'*.academy.htb'` altında sub-domain aramaya başlayacağız ve bir şey bulup bulamayacağımıza bakacağız, ki bunu bir sonraki bölümde deneyeceğiz.


# Sub-domain Fuzzing

Bu bölümde, herhangi bir web sitesinin sub-domain'lerini (yani, *.website.com) tanımlamak için ffuf'u nasıl kullanacağımızı öğreneceğiz.

## Sub-domains

Hedefimize gelince, hedefimiz olarak inlanefreight.com'u kullanacağız ve taramamızı bunun üzerinde yapacağız. Ffuf kullanalım ve FUZZ anahtar kelimesini subdomainlerinin yerine yerleştirelim ve herhangi bir hit alıp almadığımıza bakalım:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u https://FUZZ.inlanefreight.com/


        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : https://FUZZ.inlanefreight.com/
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
________________________________________________

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 381ms]
    * FUZZ: support

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 385ms]
    * FUZZ: ns3

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 402ms]
    * FUZZ: blog

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 180ms]
    * FUZZ: my

[Status: 200, Size: 22266, Words: 2903, Lines: 316, Duration: 589ms]
    * FUZZ: www

<...SNIP...>
```

Birkaç isabet aldığımızı görüyoruz. Şimdi, aynı şeyi academy.htb üzerinde çalıştırmayı deneyebilir ve herhangi bir isabet alıp almadığımızı görebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://FUZZ.academy.htb/


        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : https://FUZZ.academy.htb/
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

:: Progress: [4997/4997] :: Job [1/1] :: 131 req/sec :: Duration: [0:00:38] :: Errors: 4997 ::
```

Herhangi bir geri dönüş almadığımızı görüyoruz. Bu, academy.htb altında sub-domain olmadığı anlamına mı geliyor? - Hayır.

Bu, daha önce de belirtildiği gibi herkese açık bir DNS kaydına sahip olmadığı için academy.htb altında herkese açık sub-domainler olmadığı anlamına gelir. academy.htb dosyasını /etc/hosts dosyamıza eklemiş olsak da, yalnızca ana domain'i ekledik, bu nedenle ffuf diğer sub-domain'leri aradığında, bunları /etc/hosts dosyasında bulamayacak ve public DNS'e soracaktır, ki bu DNS'in de bu domainlere sahip olmayacağı açıktır.


Soru : Bir müşteri sub domain portalı bulmak için 'inlanefreight.com' üzerinde bir sub-domain fuzzing testi çalıştırmayı deneyin. Tam domain adı nedir?

Cevap : `customer.inlanefreight.com`

![Pasted image 20241224125836.png](/img/user/resimler/Pasted%20image%2020241224125836.png)
![Pasted image 20241224125848.png](/img/user/resimler/Pasted%20image%2020241224125848.png)


# Vhost Fuzzing

VHost'lar ve sub-domainler arasındaki temel fark, bir VHost'un temelde aynı sunucuda sunulan ve aynı IP'ye sahip bir 'subdomain' olmasıdır, öyle ki tek bir IP iki veya daha fazla farklı web sitesine hizmet verebilir.


### **VHost'ların herkese açık DNS kayıtları olabilir veya olmayabilir.**

Birçok durumda, birçok web sitesi aslında herkese açık olmayan subdomain adlarına sahiptir ve bunları herkese açık DNS kayıtlarında yayınlamaz ve bu nedenle onları bir tarayıcıda ziyaret edersek, herkese açık DNS IP'lerini bilmeyeceğinden bağlanamayız. Bir kez daha, subdomain fuzzing'i kullanırsak, yalnızca herkese açık subdomainleri belirleyebiliriz, ancak herkese açık olmayan herhangi bir subdomain belirleyemeyiz.

Bu, zaten sahip olduğumuz bir IP üzerinde VHosts Fuzzing'i kullandığımız yerdir. Aynı IP üzerinde bir tarama yapacağız ve taramaları test edeceğiz ve ardından hem public hem de public olmayan  subdomain alanları ve VHost'ları belirleyebileceğiz.


## Vhosts Fuzzing

VHost'ları taramak için, tüm kelime listesini /etc/hosts dosyamıza manuel olarak eklemeden, HTTP başlıklarını, özellikle de Host: başlığını fuzzing yapacağız. Bunu yapmak için, bir başlık belirtmek için -H bayrağını kullanabiliriz ve aşağıdaki gibi içinde FUZZ anahtar kelimesini kullanacağız:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb'


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://academy.htb:PORT/
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
________________________________________________

mail2                   [Status: 200, Size: 900, Words: 423, Lines: 56]
dns2                    [Status: 200, Size: 900, Words: 423, Lines: 56]
ns3                     [Status: 200, Size: 900, Words: 423, Lines: 56]
dns1                    [Status: 200, Size: 900, Words: 423, Lines: 56]
lists                   [Status: 200, Size: 900, Words: 423, Lines: 56]
webmail                 [Status: 200, Size: 900, Words: 423, Lines: 56]
static                  [Status: 200, Size: 900, Words: 423, Lines: 56]
web                     [Status: 200, Size: 900, Words: 423, Lines: 56]
www1                    [Status: 200, Size: 900, Words: 423, Lines: 56]
<...SNIP...>
```

Kelime listesindeki tüm kelimelerin 200 OK döndürdüğünü görüyoruz! Bu beklenen bir durum, çünkü http://academy.htb:PORT/ adresini ziyaret ederken sadece header'ı değiştiriyoruz. Yani, her zaman 200 OK alacağımızı biliyoruz. Ancak, VHost mevcutsa ve başlıkta doğru bir tane gönderirsek, farklı bir yanıt boyutu almalıyız, çünkü bu durumda, muhtemelen farklı bir sayfa gösterecek olan VHost'lardan sayfa alacağız.


# Filtering Results

Şimdiye kadar, ffuf'umuzda herhangi bir filtreleme kullanmadık ve sonuçlar varsayılan olarak HTTP kodlarına göre otomatik olarak filtreleniyor, bu da 404 NOT FOUND kodunu filtreliyor ve geri kalanını tutuyor. Ancak, önceki ffuf çalıştırmamızda gördüğümüz gibi, 200 kodlu birçok yanıt alabiliriz. Bu durumda, sonuçları bu bölümde öğreneceğimiz başka bir faktöre göre filtrelememiz gerekecektir.


## Filtering

Ffuf, belirli bir HTTP kodunu, yanıt boyutunu veya kelime miktarını eşleştirme veya filtreleme seçeneği sunar. Bunu ffuf -h ile görebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -h
...SNIP...
MATCHER OPTIONS:
  -mc              Match HTTP status codes, or "all" for everything. (default: 200,204,301,302,307,401,403)
  -ml              Match amount of lines in response
  -mr              Match regexp
  -ms              Match HTTP response size
  -mw              Match amount of words in response

FILTER OPTIONS:
  -fc              Filter HTTP status codes from response. Comma separated list of codes and ranges
  -fl              Filter by amount of lines in response. Comma separated list of line counts and ranges
  -fr              Filter regexp
  -fs              Filter HTTP response size. Comma separated list of sizes and ranges
  -fw              Filter by amount of words in response. Comma separated list of word counts and ranges
<...SNIP...>
```

Bu durumda, diğer VHost'lardan gelen yanıt boyutunun ne olacağını bilmediğimiz için eşleştirmeyi kullanamayız. Yukarıdaki testten de görüldüğü gibi, yanlış sonuçların yanıt boyutunun 900 olduğunu biliyoruz ve bunu -fs 900 ile filtreleyebiliriz. Şimdi, aynı önceki komutu tekrarlayalım, yukarıdaki bayrağı ekleyelim ve ne elde ettiğimize bakalım:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt:FUZZ -u http://academy.htb:PORT/ -H 'Host: FUZZ.academy.htb' -fs 900


       /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://academy.htb:PORT/
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/DNS/subdomains-top1million-5000.txt
 :: Header           : Host: FUZZ.academy.htb
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: 900
________________________________________________

<...SNIP...>
admin                   [Status: 200, Size: 0, Words: 1, Lines: 1]
:: Progress: [4997/4997] :: Job [1/1] :: 1249 req/sec :: Duration: [0:00:04] :: Errors: 0 ::
```

Sayfayı ziyaret ederek ve bağlanıp bağlanamayacağımızı görerek bunu doğrulayabiliriz:

![Pasted image 20241224130810.png](/img/user/resimler/Pasted%20image%2020241224130810.png)

Not 1: “admin.academy.htb” adresini “/etc/hosts” dosyasına eklemeyi unutmayın.

Not 2: Alıştırmanız yeniden başlatıldıysa, web sitesini ziyaret ederken hala doğru porta sahip olduğunuzdan emin olun.

Sayfaya erişebildiğimizi görüyoruz, ancak academy.htb ile elde ettiğimizden farklı olarak boş bir sayfa alıyoruz, bu nedenle bunun gerçekten farklı bir VHost olduğunu doğruluyoruz. Hatta https://admin.academy.htb:PORT/blog/index.php adresini ziyaret edebiliriz ve 404 SAYFA BULUNAMADI hatası alacağımızı görürüz, bu da şu anda gerçekten farklı bir VHost'ta olduğumuzu teyit eder.

admin.academy.htb üzerinde recursive bir tarama çalıştırmayı deneyin ve hangi sayfaları tanımlayabileceğinizi görün.


Soru : 'academy.htb' üzerinde bir VHost fuzzing taraması çalıştırmayı deneyin ve başka hangi VHost'ları elde ettiğinize bakın. Başka hangi VHost'ları elde ettiniz?

Cevap : ==test.academy.htb==

![Pasted image 20241224131218.png](/img/user/resimler/Pasted%20image%2020241224131218.png)

![Pasted image 20241224131330.png](/img/user/resimler/Pasted%20image%2020241224131330.png)
![Pasted image 20241224131338.png](/img/user/resimler/Pasted%20image%2020241224131338.png)

# Parameter Fuzzing - GET

admin.academy.htb adresinde rekursive ffuf taraması yaparsak http://admin.academy.htb:PORT/admin/admin.php adresini bulmamız gerekir. Bu sayfaya erişmeye çalışırsak aşağıdakileri görürüz:

![Pasted image 20241224132136.png](/img/user/resimler/Pasted%20image%2020241224132136.png)

Bu, bayrağı okuma erişimine sahip olup olmadıklarını doğrulamak için kullanıcıları tanımlayan bir şey olması gerektiğini gösterir. Giriş yapmadık ya da backend'de doğrulanabilecek herhangi bir cookie'ye sahip değiliz. Dolayısıyla, belki de bayrağı okumak için sayfaya iletebileceğimiz bir anahtar vardır. Bu tür anahtarlar genellikle bir GET ya da POST HTTP isteği kullanılarak parametre olarak aktarılır. Bu bölümde, sayfa tarafından kabul edilebilecek bir parametre belirleyene kadar bu tür parametreler için nasıl fuzz yapılacağı tartışılacaktır.

İpucu: Parametreleri fuzzing yapmak, herkese açık olan yayınlanmamış parametreleri ortaya çıkarabilir. Bu tür parametreler daha az test edilmiş ve daha az güvenli olma eğilimindedir, bu nedenle bu tür parametreleri diğer modüllerde tartıştığımız web güvenlik açıkları için test etmek önemlidir.

## GET Request Fuzzing

Bir web sitesinin çeşitli bölümlerini nasıl fuzzing yaptığımıza benzer şekilde, parametreleri numaralandırmak için ffuf kullanacağız. İlk olarak GET istekleri için fuzzing ile başlayalım, bunlar genellikle URL'den hemen sonra bir ? sembolü ile geçilir:

- `http://admin.academy.htb:PORT/admin/admin.php?param1=key`.

Dolayısıyla, tek yapmamız gereken yukarıdaki örnekte param1'i FUZZ ile değiştirmek ve taramamızı yeniden çalıştırmaktır. Ancak başlamadan önce uygun bir kelime listesi seçmeliyiz. Bir kez daha, SecLists /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt içinde tam da bunu sunuyor. Bununla birlikte, taramamızı çalıştırabiliriz.

Bir kez daha, birçok sonuç alacağız, bu yüzden aldığımız varsayılan yanıt boyutunu filtreleyeceğiz.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key -fs xxx


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : GET
 :: URL              : http://admin.academy.htb:PORT/admin/admin.php?FUZZ=key
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: xxx
________________________________________________

<...SNIP...>                    [Status: xxx, Size: xxx, Words: xxx, Lines: xxx]
```

Bir geri dönüş alıyoruz. Sayfayı ziyaret etmeyi deneyelim ve bu GET parametresini ekleyelim ve şimdi bayrağı okuyup okuyamayacağımızı görelim:

![Pasted image 20241224132621.png](/img/user/resimler/Pasted%20image%2020241224132621.png)

Gördüğümüz gibi, geri aldığımız tek isabet kullanımdan kaldırılmış ve artık kullanılmıyor gibi görünüyor.


Soru : Bu bölümde öğrendiklerinizi kullanarak, bu sayfada bir parametre fuzzing taraması yapın. bu web sayfası tarafından kabul edilen parametre nedir?

Cevap : user

![Pasted image 20241224133213.png](/img/user/resimler/Pasted%20image%2020241224133213.png)
![Pasted image 20241224133137.png](/img/user/resimler/Pasted%20image%2020241224133137.png)


# Parameter Fuzzing - POST

POST istekleri ile GET istekleri arasındaki temel fark, POST isteklerinin URL ile birlikte iletilmemesi ve bir ? sembolünden sonra basitçe eklenememesidir. POST istekleri HTTP isteği içindeki veri alanında iletilir. 

Data alanını ffuf ile fuzzlamak için, daha önce ffuf -h çıktısında gördüğümüz gibi -d bayrağını kullanabiliriz. Ayrıca POST istekleri göndermek için -X POST eklememiz gerekir.

İpucu: PHP'de “POST” verisi “content-type” sadece “application/x-www-form-urlencoded” kabul edebilir. Dolayısıyla, bunu “ffuf” içinde “-H ‘Content-Type: application/x-www-form-urlencoded’” ile ayarlayabiliriz.

Öyleyse, daha önce yaptığımızı tekrarlayalım, ancak FUZZ keyword'ümüzü -d bayrağından sonra yerleştirelim:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'FUZZ=key' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.1.0-git
________________________________________________

 :: Method           : POST
 :: URL              : http://admin.academy.htb:PORT/admin/admin.php
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : FUZZ=key
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: xxx
________________________________________________

id                      [Status: xxx, Size: xxx, Words: xxx, Lines: xxx]
<...SNIP...>
```

Bu sefer görebildiğimiz gibi, GET ve id olan başka bir parametreyi fuzzing yaparken elde ettiğimizle aynı olan birkaç isabet aldık. Bakalım id parametresi ile bir POST isteği gönderirsek ne elde edeceğiz. Bunu curl ile aşağıdaki gibi yapabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=key' -H 'Content-Type: application/x-www-form-urlencoded'

<div class='center'><p>Invalid id!</p></div>
<...SNIP...>
```

Gördüğümüz gibi, mesaj artık Geçersiz kimlik diyor!


# Value Fuzzing

Çalışan bir parametreyi fuzz ettikten sonra, şimdi ihtiyacımız olan bayrak içeriğini döndürecek doğru değeri fuzz etmeliyiz. Bu bölümde, kelime listemizi geliştirdikten sonra parametreler için fuzzing'e oldukça benzer olması gereken parametre değerleri için fuzzing tartışılacaktır.


## Custom Wordlist

Kullanıcı adları gibi bazı parametreler için, potansiyel kullanıcı adları için önceden hazırlanmış bir kelime listesi bulabilir veya web sitesini potansiyel olarak kullanan kullanıcıları temel alarak kendi kelime listemizi oluşturabiliriz. Bu gibi durumlarda, seclists dizini altında çeşitli kelime listeleri arayabilir ve hedeflediğimiz parametreyle eşleşen değerleri içerebilecek bir tane bulmaya çalışabiliriz. Özel parametreler gibi diğer durumlarda, kendi kelime listemizi geliştirmemiz gerekebilir. Bu durumda, id parametresinin bir tür sayı girişi kabul edebileceğini tahmin edebiliriz. Bu id'ler özel bir formatta olabilir veya 1-1000 veya 1-1000000 gibi sıralı olabilir. 1-1000 arasındaki tüm sayıları içeren bir kelime listesi ile başlayacağız.

Bu kelime listesini oluşturmanın, ID'leri bir dosyaya elle yazmaktan Bash veya Python kullanarak komut dosyası oluşturmaya kadar birçok yolu vardır. En basit yol, Bash'te 1-1000 arasındaki tüm sayıları bir dosyaya yazan aşağıdaki komutu kullanmaktır:

```shell-session
M1R4CKCK@htb[/htb]$ for i in $(seq 1 1000); do echo $i >> ids.txt; done
```

```shell-session
M1R4CKCK@htb[/htb]$ cat ids.txt

1
2
3
4
5
6
<...SNIP...>
```


## Value Fuzzing

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w ids.txt:FUZZ -u http://admin.academy.htb:PORT/admin/admin.php -X POST -d 'id=FUZZ' -H 'Content-Type: application/x-www-form-urlencoded' -fs xxx


        /'___\  /'___\           /'___\
       /\ \__/ /\ \__/  __  __  /\ \__/
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/
         \ \_\   \ \_\  \ \____/  \ \_\
          \/_/    \/_/   \/___/    \/_/

       v1.0.2
________________________________________________

 :: Method           : POST
 :: URL              : http://admin.academy.htb:30794/admin/admin.php
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : id=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: xxx
________________________________________________

<...SNIP...>                      [Status: xxx, Size: xxx, Words: xxx, Lines: xxx]
```


Soru : 'ids.txt' kelime listesini oluşturmaya çalışın, bir fuzzing taraması ile kabul edilen değeri belirleyin ve ardından bayrağı toplamak için 'curl' ile bir 'POST' isteğinde kullanın. Bayrağın içeriği nedir?

Cevap :  `HTB{p4r4m373r_fuzz1n6_15_k3y!}`


![Pasted image 20241224134202.png](/img/user/resimler/Pasted%20image%2020241224134202.png)
![Pasted image 20241224134211.png](/img/user/resimler/Pasted%20image%2020241224134211.png)

![Pasted image 20241224134248.png](/img/user/resimler/Pasted%20image%2020241224134248.png)

![Pasted image 20241224134343.png](/img/user/resimler/Pasted%20image%2020241224134343.png)
![Pasted image 20241224134349.png](/img/user/resimler/Pasted%20image%2020241224134349.png)

![Pasted image 20241224134637.png](/img/user/resimler/Pasted%20image%2020241224134637.png)


# Skills Assessment - Web Fuzzing

Size bir çevrimiçi akademinin IP adresi verildi ancak web sitesi hakkında daha fazla bilgiye sahip değilsiniz. Bir Sızma Testi gerçekleştirmenin ilk adımı olarak, IP ve alan adlarını düzgün bir şekilde numaralandırmak için IP'lerine bağlı tüm sayfaları ve domainleri bulmanız beklenir.

Son olarak, herhangi birinin etkileşime girilebilecek herhangi bir parametreye sahip olup olmadığını görmek için belirlediğiniz sayfalarda biraz fuzzing yapmalısınız. Etkin parametreler bulursanız, bunlardan herhangi bir veri alıp alamayacağınıza bakın.


Soru 1 : Yukarıda gösterilen IP için `'*.academy.htb'` üzerinde bir subdomain/vhost fuzzing taraması çalıştırın. Tanımlayabildiğiniz tüm subdomainler nelerdir? (Sadece subdomain'i yazın)

Cevap : ==archive,faculty,test==

![Pasted image 20241224142125.png](/img/user/resimler/Pasted%20image%2020241224142125.png)

![Pasted image 20241224142136.png](/img/user/resimler/Pasted%20image%2020241224142136.png)


Soru 2 : Sayfa fuzzing taramanızı çalıştırmadan önce, öncelikle bir uzantı fuzzing taraması yapmalısınız. Domainler tarafından kabul edilen farklı uzantılar nelerdir?

Cevap : ==php,phps,php7==

![Pasted image 20241224144116.png](/img/user/resimler/Pasted%20image%2020241224144116.png)

![Pasted image 20241224144149.png](/img/user/resimler/Pasted%20image%2020241224144149.png)

![Pasted image 20241224144224.png](/img/user/resimler/Pasted%20image%2020241224144224.png)


Soru 3 :Tespit edeceğiniz sayfalardan birinde “Erişiminiz yok!” yazmalıdır. Tam sayfa URL'si nedir? (-e ye güvenme)

Cevap : 

1-test.academy.htb'de herhangi bir dizin tespit edemedi

![Pasted image 20241224144722.png](/img/user/resimler/Pasted%20image%2020241224144722.png)

2- archive.academy.htb 'de dizin tespit edildi ama boş 

![Pasted image 20241224145229.png](/img/user/resimler/Pasted%20image%2020241224145229.png)

![Pasted image 20241224145214.png](/img/user/resimler/Pasted%20image%2020241224145214.png)

3- faculty.academy.htb'de bir dizin tespit ettik. -e kullanma !!

![Pasted image 20241224145308.png](/img/user/resimler/Pasted%20image%2020241224145308.png)

![Pasted image 20241224152218.png](/img/user/resimler/Pasted%20image%2020241224152218.png)

![Pasted image 20241224152229.png](/img/user/resimler/Pasted%20image%2020241224152229.png)
![Pasted image 20241224152241.png](/img/user/resimler/Pasted%20image%2020241224152241.png)


Soru : Önceki sorudaki sayfada, sayfa tarafından kabul edilen birden fazla parametre bulabilmeniz gerekir. Bunlar nelerdir?

Cevap : 

1- Get 
![Pasted image 20241224153157.png](/img/user/resimler/Pasted%20image%2020241224153157.png)
![Pasted image 20241224153216.png](/img/user/resimler/Pasted%20image%2020241224153216.png)

2-POST 

![Pasted image 20241224153953.png](/img/user/resimler/Pasted%20image%2020241224153953.png)
![Pasted image 20241224153959.png](/img/user/resimler/Pasted%20image%2020241224153959.png)


Soru :  Çalışma değerleri için tanımladığınız parametreleri fuzzing yapmayı deneyin. Bunlardan biri bir bayrak döndürmelidir. Bayrağın içeriği nedir?

Cevap : 


![Pasted image 20241224193715.png](/img/user/resimler/Pasted%20image%2020241224193715.png)
![Pasted image 20241224193720.png](/img/user/resimler/Pasted%20image%2020241224193720.png)

![Pasted image 20241224193735.png](/img/user/resimler/Pasted%20image%2020241224193735.png)
