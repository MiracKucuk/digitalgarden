---
{"dg-publish":true,"permalink":"/ctf/htb-carpe-diem/"}
---

[hackthebox](https://0xdf.gitlab.io/tags#hackthebox) [htb-carpediem](https://0xdf.gitlab.io/tags#htb-carpediem) [ctf](https://0xdf.gitlab.io/tags#ctf) [nmap](https://0xdf.gitlab.io/tags#nmap) [feroxbuster](https://0xdf.gitlab.io/tags#feroxbuster) [wfuzz](https://0xdf.gitlab.io/tags#wfuzz) [vhosts](https://0xdf.gitlab.io/tags#vhosts) [php](https://0xdf.gitlab.io/tags#php) [trudesk](https://0xdf.gitlab.io/tags#trudesk) [html-file](https://0xdf.gitlab.io/tags#html-file) [upload](https://0xdf.gitlab.io/tags#upload) [burp](https://0xdf.gitlab.io/tags#burp) [burp-repeater](https://0xdf.gitlab.io/tags#burp-repeater) [webshell](https://0xdf.gitlab.io/tags#webshell) [docker](https://0xdf.gitlab.io/tags#docker) [container](https://0xdf.gitlab.io/tags#container) [pivot](https://0xdf.gitlab.io/tags#pivot) [chisel](https://0xdf.gitlab.io/tags#chisel) [mongo](https://0xdf.gitlab.io/tags#mongo) [mongoexport](https://0xdf.gitlab.io/tags#mongoexport) [bcrypt](https://0xdf.gitlab.io/tags#bcrypt) [python](https://0xdf.gitlab.io/tags#python) [api](https://0xdf.gitlab.io/tags#api) [source-code](https://0xdf.gitlab.io/tags#source-code) [voip](https://0xdf.gitlab.io/tags#voip) [zoiper](https://0xdf.gitlab.io/tags#zoiper) [voicemail](https://0xdf.gitlab.io/tags#voicemail) [backdrop-cms](https://0xdf.gitlab.io/tags#backdrop-cms) [wireshark](https://0xdf.gitlab.io/tags#wireshark) [tcpdump](https://0xdf.gitlab.io/tags#tcpdump) [tls-decryption](https://0xdf.gitlab.io/tags#tls-decryption) [weak-tls](https://0xdf.gitlab.io/tags#weak-tls) [backdrop-plugin](https://0xdf.gitlab.io/tags#backdrop-plugin) [docker-escape](https://0xdf.gitlab.io/tags#docker-escape) [cgroups](https://0xdf.gitlab.io/tags#cgroups) [cve-2022-0492](https://0xdf.gitlab.io/tags#cve-2022-0492) [htb-ready](https://0xdf.gitlab.io/tags#htb-ready)

CarpeDiem, pivoting yaparak küçük bir Docker container ağı üzerinden ilerlemeyi içeren zor bir Linux makinesidir. Başlangıçta, bir web sitesinde admin erişimi elde edeceğim ve bir upload özelliğini kullanarak bir webshell yükleyip bu container'a bir foothold kazanacağım. Daha sonra ağı enumerate ederek bir **Trudesk** instance'ı bulacağım. Buradan, yeni bir çalışan hakkında, kimlik bilgilerinin voicemail üzerinden iletileceğini belirten bir ticket okuyacağım. Bu ticket'taki talimatları takip ederek voicemail'e erişim sağlayıp SSH parolasını alacağım.

SSH ile erişim sağladıktan sonra, bir **Backdrop CMS** instance'ına pivoting yapacağım. Burada kimlik bilgilerini elde edip kötü niyetli bir plugin yükleyerek ilerleyeceğim. Bu noktadan sonra, container içinde root erişimi elde edeceğim ve ardından **CVE-2022-0492** güvenlik açığını kötüye kullanarak host üzerinde root yetkilerine ulaşacağım.


## Box Info

|Name|[CarpeDiem](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)[![CarpeDiem](https://0xdf.gitlab.io/icons/box-carpediem.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fcarpediem)|
|---|---|
|Release Date|[25 Jun 2022](https://twitter.com/hackthebox_eu/status/1539624282686464005)|
|Retire Date|03 Dec 2022|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Hard [40]|
|Rated Difficulty|![Rated difficulty for CarpeDiem](https://0xdf.gitlab.io/img/carpediem-diff.png)|
|Radar Graph|![Radar chart for CarpeDiem](https://0xdf.gitlab.io/img/carpediem-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|01:23:15[![jazzpizazz](https://www.hackthebox.com/badge/image/87804)](https://app.hackthebox.com/users/87804)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|02:20:39[![jazzpizazz](https://www.hackthebox.com/badge/image/87804)](https://app.hackthebox.com/users/87804)|
|Creators|[![ctrlzero](https://www.hackthebox.com/badge/image/168546)](https://app.hackthebox.com/users/168546)  <br>[![TheCyberGeek](https://www.hackthebox.com/badge/image/114053)](https://app.hackthebox.com/users/114053)|

## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

```
oxdf@hacky$ nmap -p- --min-rate 10000 10.10.11.167
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-28 13:59 UTC
Nmap scan report for 10.10.11.167
Host is up (0.087s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.25 seconds


oxdf@hacky$ nmap -p 22,80 -sCV 10.10.11.167
Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-28 14:49 UTC
Nmap scan report for 10.10.11.167
Host is up (0.086s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Comming Soon
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.93 seconds
```


[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) sürümüne göre, host muhtemelen Ubuntu focal 20.04 çalıştırıyor.


### Website - TCP 80

#### Site

Site, “ coming soon” geri sayımı için çok uzun bir zamanlayıcı dışında fazla bilgi sunmuyor:

![Pasted image 20250107234202.png](/img/user/Pasted%20image%2020250107234202.png)

Sanal makinemdeki /etc/hosts dosyasına ekleyeceğim bir domain adı veriyor.

“ Subscribe” butonu “Your email address” kısmına girilen verileri bile göndermiyor, bunun yerine sadece /?


#### Tech Stack

HTTP headers NGINX (nmap ile eşleşiyor) dışında pek bir şey göstermiyor:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 28 Nov 2022 14:51:36 GMT
Content-Type: text/html
Last-Modified: Thu, 07 Apr 2022 22:54:58 GMT
Connection: close
ETag: W/"624f6bc2-b3b"
Content-Length: 2875
```

HTML kaynağına hızlı bir bakış sadece bir bootstrap template'i gösteriyor. Çok ilginç başka bir şey yok.

#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştırıyorum ve ilginç bir şey bulamıyor:

```
oxdf@hacky$ feroxbuster -u http://carpediem.htb

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://carpediem.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
200      GET       58l      161w     2875c http://carpediem.htb/
301      GET        7l       12w      178c http://carpediem.htb/scripts => http://carpediem.htb/scripts/
301      GET        7l       12w      178c http://carpediem.htb/img => http://carpediem.htb/img/
301      GET        7l       12w      178c http://carpediem.htb/styles => http://carpediem.htb/styles/
[####################] - 54s   150000/150000  0s      found:4       errors:0
[####################] - 53s    30000/30000   565/s   http://carpediem.htb
[####################] - 53s    30000/30000   564/s   http://carpediem.htb/
[####################] - 52s    30000/30000   566/s   http://carpediem.htb/scripts
[####################] - 53s    30000/30000   565/s   http://carpediem.htb/img
[####################] - 53s    30000/30000   565/s   http://carpediem.htb/styles
```


### Subdomain Fuzz

Carpediem.htb domain adının kullanımı göz önüne alındığında, farklı bir sayfa döndürebilecek diğer subdomainleri arayacağım. Host başlığını fuzzlamak için wfuzz kullanacağım ve 2875 bayt uzunluğundaki varsayılan sayfayı filtreleyeceğim:

```
oxdf@hacky$ wfuzz -u http://carpediem.htb -H "Host: FUZZ.carpediem.htb" -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt --hh 2875
********************************************************
* Wfuzz 2.4.5 - The Web Fuzzer                         *
********************************************************

Target: http://carpediem.htb/
Total requests: 4989

===================================================================
ID           Response   Lines    Word     Chars       Payload
===================================================================

000000048:   200        462 L    2174 W   31090 Ch    "portal"

Total time: 44.55443
Processed Requests: 4989
Filtered Requests: 4988
Requests/sec.: 111.9753
```


### portal.carpediem.htb - TCP 80

#### Site

Bu site motosikletler hakkında:

![Pasted image 20250107234612.png](/img/user/Pasted%20image%2020250107234612.png)


“About” (Hakkında) bağlantısı sadece bazı lorem ipsum metinleri verir. “ Categories” (Kategoriler) ve “Brand” (Marka), filtreleme yapabilen açılır pencereler sağlar:

![Pasted image 20250107234658.png](/img/user/Pasted%20image%2020250107234658.png)


#### With Account

“ Login ” tıklandığında bir form açılır penceresi görüntülenir:

![Pasted image 20250107234727.png](/img/user/Pasted%20image%2020250107234727.png)

“ Create Account” (Hesap Oluştur) bağlantısı bir kayıt formu sunmaktadır:

![Pasted image 20250107235040.png](/img/user/Pasted%20image%2020250107235040.png)

Formu doldurduktan sonra, “Login” ifadesi “Hi, 0xdf!” ve bir çıkış (logout) simgesiyle değiştirilir:

![Pasted image 20250107235112.png](/img/user/Pasted%20image%2020250107235112.png)

“Hello, 0xdf!” bölümünden hesabımla ilgili bir sayfa olması dışında diğer her şey aynı:

![Pasted image 20250107235141.png](/img/user/Pasted%20image%2020250107235141.png)

“ Manage Account” bir şeyleri değiştirmek için bir sayfaya yönlendirir:

![Pasted image 20250107235209.png](/img/user/Pasted%20image%2020250107235209.png)

Her motosiklet görüntülendiğinde, bir “Book this Bike” düğmesi bulunur. Giriş yapılmamışsa, giriş formu açılır. Aksi takdirde, bir motosiklet ayırtma formu açılır:

![Pasted image 20250107235258.png](/img/user/Pasted%20image%2020250107235258.png)


Gönderirken gösteriyor:

![Pasted image 20250107235312.png](/img/user/Pasted%20image%2020250107235312.png)

Artık tüm rezervasyonlar “My Bookings” sayfasında görünmektedir:

![Pasted image 20250107235335.png](/img/user/Pasted%20image%2020250107235335.png)

#### Tech Stack

Bu subdomain için HTTP header'ları sitenin PHP çalıştırdığını gösterir:

```
HTTP/1.1 200 OK
Server: nginx/1.18.0 (Ubuntu)
Date: Mon, 28 Nov 2022 15:13:32 GMT
Content-Type: text/html; charset=UTF-8
Content-Length: 31090
Connection: close
X-Powered-By: PHP/7.4.25
Expires: Thu, 19 Nov 1981 08:52:00 GMT
Cache-Control: no-store, no-cache, must-revalidate
Pragma: no-cache
Vary: Accept-Encoding
```

Bunu, `/index.php'nin` aynı sayfayı yüklediğini, ancak `index.html` veya diğer yolların yüklemediğini göstererek doğrulayabilirim.

URL yapısı da oldukça ilginç. “Home” sayfası dışındaki her sayfa bir `?p=[name]` değişkeni belirtiyor. Örneğin, “About” sayfası için `p=about` kullanılıyor. “All Categories” (Tüm Kategoriler) sayfasını görüntülemek için `p=view_categories` ayarlanıyor. Honda motosikletlerini filtrelemek için `p=bikes&s=c81e728d9d4c2f636f067f89cc14862c` kullanılıyor; burada `s`, Honda için bir ID’yi temsil ediyor. Kategori filtresi ise benzer şekilde çalışıyor, ancak bu durumda `c` değişkeni kullanılıyor.

Bu noktada, p değişkenine göre sayfaları ya dahil eden ya da dallandıran bir PHP sayfası görebiliyorum. p değerini `index` olarak ayarlamayı deneyeceğim ve ne olduğunu göreceğim. Sayfa çöküyor:

![Pasted image 20250107235649.png](/img/user/Pasted%20image%2020250107235649.png)

Bu, sayfanın `index.php` dosyasını dahil etmeye çalışması, ardından `index.php` dosyasının tekrar kendini dahil etmeye çalışması ve bu döngünün belleğin tükenmesine kadar devam etmesi durumudur. Aynı zamanda, disk üzerindeki web dizininin tam yolunu da ortaya çıkarır.


---

Bu durumda, web uygulaması, URL'de verilen `p` parametresine göre dosyaları dahil ediyor gibi görünüyor. Örneğin, `p=about` parametresi ile `about.php` gibi bir dosya dahil edilebilir. Ancak burada, `p=index` parametresini verdiğinizde şöyle bir durum oluşuyor:


1. **`index.php` Kendini Dahil Ediyor:**  

* Sayfa, `index.php` dosyasını dahil etmeye çalışıyor. Ancak, bu dosyanın içinde aynı `index.php` dosyasını tekrar dahil eden bir kod bulunuyor. Örneğin:

```
include($_GET['p'] . '.php');
```

Bu durumda, `p=index` olduğunda, `index.php` kendi içine `index.php` dosyasını dahil etmeye çalışıyor.


2. **Sonsuz Döngü Oluşuyor:**  
* `index.php` dosyası, bir kez daha kendi kendini dahil etmeye çalışıyor. Bu işlem sonsuz bir döngü yaratıyor, çünkü her dahil edilen `index.php` bir başka `index.php` daha dahil etmeye çalışıyor.

3. **Bellek Tükeniyor:**  
* PHP'nin bir işlemi gerçekleştirmek için kullanabileceği belirli bir bellek sınırı vardır. Bu döngü, sürekli olarak yeni bir `index.php` dosyası dahil ettiği için bellek sınırını aşar ve program çökmesine neden olur.

4. **Tam Dizin Ortaya Çıkıyor:**  
* PHP çökme sırasında hata mesajı verirken genellikle bir "include" hatası gösterir. Örneğin:

```
Fatal error: Allowed memory size of 134217728 bytes exhausted (tried to allocate 1234 bytes) in /var/www/html/index.php on line 10
```

Bu mesajda, sunucuda **`/var/www/html/`** gibi bir dizin yolu belirtilir. Bu, saldırganlar için faydalı bir bilgi olabilir, çünkü web uygulamasının dosya sistemindeki tam konumunu öğrenmiş olurlar.

### Özet

Kısaca, `p=index` parametresi, `index.php` dosyasının kendini tekrar tekrar dahil etmeye çalışmasına neden oluyor, bu da sonsuz bir döngü oluşturuyor ve bellek sınırını aşarak çökme yaratıyor. Hata mesajı ise web uygulamasının dosya sistemi yolunu açığa çıkarıyor, bu da güvenlik açısından bir risk oluşturabilir.


---



#### Directory Brute Force

feroxbuster -x php ile bir ton şey döndürür (okunabilirlik için çoğu burada kesilmiştir):

```
oxdf@hacky$ feroxbuster -u http://portal.carpediem.htb -x php

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.7.1
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://portal.carpediem.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /usr/share/seclists/Discovery/Web-Content/raft-medium-directories.txt
 👌  Status Codes          │ [200, 204, 301, 302, 307, 308, 401, 403, 405, 500]
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.7.1
 💲  Extensions            │ [php]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
 🎉  New Version Available │ https://github.com/epi052/feroxbuster/releases/latest
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
301      GET        9l       28w      330c http://portal.carpediem.htb/plugins => http://portal.carpediem.htb/plugins/
301      GET        9l       28w      328c http://portal.carpediem.htb/admin => http://portal.carpediem.htb/admin/
200      GET       75l      135w     2963c http://portal.carpediem.htb/login.php
200      GET      462l     2174w        0c http://portal.carpediem.htb/
302      GET        0l        0w        0c http://portal.carpediem.htb/logout.php => ./
301      GET        9l       28w      326c http://portal.carpediem.htb/inc => http://portal.carpediem.htb/inc/
301      GET        9l       28w      330c http://portal.carpediem.htb/uploads => http://portal.carpediem.htb/uploads/
301      GET        9l       28w      329c http://portal.carpediem.htb/assets => http://portal.carpediem.htb/assets/
...[snip]...
```

En ilginç bulgu /admin. Bunu ziyaret etmeye çalışmak bir açılır pencere döndürür:

 ![Pasted image 20250108000621.png](/img/user/Pasted%20image%2020250108000621.png)


## Shell as www-data in Portal Container

### Fails

#### Failed Source Read

Yukarıdaki analize dayanarak, `index.php` sayfasının büyük ihtimalle şu şekilde bir şey çağırdığından oldukça eminim: `include $_GET['p'] . '.php'`. Çünkü uzantıyı ekliyor, bu nedenle PHP olmayan dosyaları okumayı deneyemem.

PHP filtrelerini kullanarak dosyaların kaynak kodlarını base64 ile encode etmeyi deneyebilirim. Ne yazık ki, bu işlem başarısız oluyor:

```
?p=php://filter/convert.base64-encode/resource=index
```

![Pasted image 20250108000829.png](/img/user/Pasted%20image%2020250108000829.png)

En iyi tahminim, filtreye izin vermemek için bir şekilde filtreleme yaptığıdır. Beyond Root'ta neler olduğunu göstereceğim.


#### Failed XSS

Bir motosiklet rezervasyonu geri aldığımda, mesajda “The management will contact you as soon they sees your request for confirmation (Yönetim, onay talebinizi görür görmez sizinle iletişime geçecektir.)” ifadesi bulunuyor. Bu, yönetimin başvurumu inceleyeceğini ima ediyor ve bu da belki bir Cross-Site Scripting (XSS) açığının olabileceğini düşündürüyor.

Bir başvuru gönderdiğimde neler olduğunu incelediğimde, aslında iki POST isteği yapıldığını görüyorum, bunlar **/classes/Master.php** yoluna gönderiliyor:


![Pasted image 20250108001156.png](/img/user/Pasted%20image%2020250108001156.png)


