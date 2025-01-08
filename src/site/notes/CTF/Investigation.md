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
