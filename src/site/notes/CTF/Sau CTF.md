---
{"dg-publish":true,"permalink":"/ctf/sau-ctf/"}
---


Sau, HackTheBox'tan kolay bir kutu. Bir web sitesindeki SSRF açığını bulup kullanacağım ve bunu iç Mailtrack web sitesindeki bir command injection'dan yararlanmak için kullanacağım. Oradan, Less pager'ın root olarak shell almak için systemctl ile nasıl çalıştığını kötüye kullanacağım.

## Box Info

|Name|[Sau](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fsau)[![Sau](https://0xdf.gitlab.io/icons/box-sau.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fsau)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fsau)|
|---|---|
|Release Date|[08 Jul 2023](https://twitter.com/hackthebox_eu/status/1676984520350785538)|
|Retire Date|06 Jan 2024|
|OS|Linux ![Linux](https://0xdf.gitlab.io/icons/Linux.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Sau](https://0xdf.gitlab.io/img/sau-diff.png)|
|Radar Graph|![Radar chart for Sau](https://0xdf.gitlab.io/img/sau-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:08:39[![celesian](https://www.hackthebox.com/badge/image/114435)](https://app.hackthebox.com/users/114435)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|00:11:40[![szymex73](https://www.hackthebox.com/badge/image/139466)](https://app.hackthebox.com/users/139466)|
|Creator|[![sau123](https://www.hackthebox.com/badge/image/201596)](https://app.hackthebox.com/users/201596)|

## Recon

### nmap

nmap, SSH (22) ve HTTP (55555) olmak üzere iki açık TCP portunun yanı sıra 80 ve 8338 olmak üzere iki filtrelenmiş port bulur:

![Pasted image 20250110234115.png](/img/user/Pasted%20image%2020250110234115.png)

OpenSSH [sürümlerine](https://packages.ubuntu.com/search?keywords=openssh-server) göre, host muhtemelen Ubuntu 20.04 focal çalıştırıyor.

![Pasted image 20250110234332.png](/img/user/Pasted%20image%2020250110234332.png)

```
nmap -p 22,55555 -sCV 10.10.11.224
Starting Nmap 7.94SVN ( https://nmap.org ) at 2025-01-10 15:41 EST
Stats: 0:00:01 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 50.00% done; ETC: 15:41 (0:00:01 remaining)
Nmap scan report for 10.10.11.224
Host is up (0.11s latency).

PORT      STATE SERVICE VERSION
22/tcp    open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed (RSA)
|   256 ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21 (ECDSA)
|_  256 b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36 (ED25519)
55555/tcp open  unknown
| fingerprint-strings: 
|   FourOhFourRequest: 
|     HTTP/1.0 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     X-Content-Type-Options: nosniff
|     Date: Fri, 10 Jan 2025 20:42:07 GMT
|     Content-Length: 75
|     invalid basket name; the name does not match pattern: ^[wd-_\.]{1,250}$
|   GenericLines, Help, Kerberos, LDAPSearchReq, LPDString, RTSPRequest, SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest: 
|     HTTP/1.0 302 Found
|     Content-Type: text/html; charset=utf-8
|     Location: /web
|     Date: Fri, 10 Jan 2025 20:41:35 GMT
|     Content-Length: 27
|     href="/web">Found</a>.
|   HTTPOptions: 
|     HTTP/1.0 200 OK
|     Allow: GET, OPTIONS
|     Date: Fri, 10 Jan 2025 20:41:36 GMT
|_    Content-Length: 0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port55555-TCP:V=7.94SVN%I=7%D=1/10%Time=678185FF%P=x86_64-pc-linux-gnu%
SF:r(GetRequest,A2,"HTTP/1\.0\x20302\x20Found\r\nContent-Type:\x20text/htm
SF:l;\x20charset=utf-8\r\nLocation:\x20/web\r\nDate:\x20Fri,\x2010\x20Jan\
SF:x202025\x2020:41:35\x20GMT\r\nContent-Length:\x2027\r\n\r\n<a\x20href=\
SF:"/web\">Found</a>\.\n\n")%r(GenericLines,67,"HTTP/1\.1\x20400\x20Bad\x2
SF:0Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection
SF::\x20close\r\n\r\n400\x20Bad\x20Request")%r(HTTPOptions,60,"HTTP/1\.0\x
SF:20200\x20OK\r\nAllow:\x20GET,\x20OPTIONS\r\nDate:\x20Fri,\x2010\x20Jan\
SF:x202025\x2020:41:36\x20GMT\r\nContent-Length:\x200\r\n\r\n")%r(RTSPRequ
SF:est,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/pla
SF:in;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Reque
SF:st")%r(Help,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20
SF:text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\
SF:x20Request")%r(SSLSessionReq,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\n
SF:Content-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r
SF:\n\r\n400\x20Bad\x20Request")%r(TerminalServerCookie,67,"HTTP/1\.1\x204
SF:00\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r
SF:\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%r(TLSSessionReq,6
SF:7,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20text/plain;\x
SF:20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x20Request")%
SF:r(Kerberos,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Type:\x20t
SF:ext/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\x20Bad\x
SF:20Request")%r(FourOhFourRequest,EA,"HTTP/1\.0\x20400\x20Bad\x20Request\
SF:r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nX-Content-Type-Opti
SF:ons:\x20nosniff\r\nDate:\x20Fri,\x2010\x20Jan\x202025\x2020:42:07\x20GM
SF:T\r\nContent-Length:\x2075\r\n\r\ninvalid\x20basket\x20name;\x20the\x20
SF:name\x20does\x20not\x20match\x20pattern:\x20\^\[\\w\\d\\-_\\\.\]{1,250}
SF:\$\n")%r(LPDString,67,"HTTP/1\.1\x20400\x20Bad\x20Request\r\nContent-Ty
SF:pe:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20close\r\n\r\n400\
SF:x20Bad\x20Request")%r(LDAPSearchReq,67,"HTTP/1\.1\x20400\x20Bad\x20Requ
SF:est\r\nContent-Type:\x20text/plain;\x20charset=utf-8\r\nConnection:\x20
SF:close\r\n\r\n400\x20Bad\x20Request");
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 105.97 seconds

```


### **1. SSH Servisi (Port 22):**

- Port **22** açık ve **OpenSSH 8.2p1** çalışıyor.
- **Versiyon bilgisi**: OpenSSH 8.2p1 Ubuntu 4ubuntu0.7 (Ubuntu Linux, protokol 2.0).
- **Host Key Algoritmaları**:
    - RSA (3072-bit): `aa:88:67:d7:13:3d:08:3a:8a:ce:9d:c4:dd:f3:e1:ed`
    - ECDSA (256-bit): `ec:2e:b1:05:87:2a:0c:7d:b1:49:87:64:95:dc:8a:21`
    - ED25519 (256-bit): `b3:0c:47:fb:a2:f2:12:cc:ce:0b:58:82:0e:50:43:36`

**Çıkarım**:

- Sistemde SSH erişimi mümkün.
- Versiyon bilgisi saldırı yüzeyini değerlendirmek için kullanılabilir. Örneğin, OpenSSH 8.2p1'in bilinen zafiyetleri araştırılabilir.

---

### **2. Bilinmeyen Servis (Port 55555):**

- Port **55555** açık, ancak tam olarak tanımlanamayan bir servise ev sahipliği yapıyor.
- Tarama sırasında alınan bazı çıktılar, bu portun HTTP tabanlı bir uygulamaya işaret ettiğini gösteriyor:
    - `HTTP/1.0 302 Found`: Bu, yönlendirme davranışı olduğunu gösteriyor.
    - `Location: /web`: `/web` dizinine yönlendirme yapıyor.
    - `HTTP/1.0 400 Bad Request`: Geçersiz isteklerde kötü talep hatası döndürüyor.
    - **Hata mesajı**: "invalid basket name; the name does not match pattern: ^[wd-_.]{1,250}$"

**Çıkarım**:

- Port **55555**'de çalışan servis, HTTP protokolüne dayalı bir uygulama olabilir.
- **Muhtemel uygulama**: Özel bir API, HTTP tabanlı bir servis veya uygulama sunucusu.




### Website - TCP 55555

#### Site

Site, HTTP isteklerini toplamak ve incelemek için kullanılan bir servisdir:

![Pasted image 20250110234713.png](/img/user/Pasted%20image%2020250110234713.png)

Create tıklandığında, gelecekte sepete erişmek için kullanılabilecek bir token döndürür:

![Pasted image 20250110234826.png](/img/user/Pasted%20image%2020250110234826.png)

Sepet açıldığında, nasıl doldurulacağı gösterilir:

![Pasted image 20250110234906.png](/img/user/Pasted%20image%2020250110234906.png)

Eğer http://10.10.11.224:55555/h5lgafg adresinde curl çalıştırırsam, sepette görünür:

![Pasted image 20250110235106.png](/img/user/Pasted%20image%2020250110235106.png)

![Pasted image 20250110235122.png](/img/user/Pasted%20image%2020250110235122.png)

#### Tech Stack

HTTP response başlıkları pek bir şey söylemiyor:

```
HTTP/1.1 200 OK
Content-Type: text/html; charset=utf-8
Date: Thu, 04 Jan 2024 15:31:26 GMT
Connection: close
Content-Length: 8700
```

Tüm URL'ler extension-less görünüyor ve dizin sayfasını indiren bir tane bile tahmin edemiyorum.

Ana sayfanın footer'ında “Powered by [request-baskets](https://github.com/darklynx/request-baskets) | Version: 1.2.1” yazıyor. Bu Go ile yazılmış bir yazılım.

![Pasted image 20250110235555.png](/img/user/Pasted%20image%2020250110235555.png)


#### Directory Brute Force

Feroxbuster'ı siteye karşı çalıştıracağım:

![Pasted image 20250111000338.png](/img/user/Pasted%20image%2020250111000338.png)
![Pasted image 20250111000356.png](/img/user/Pasted%20image%2020250111000356.png)
![Pasted image 20250111000416.png](/img/user/Pasted%20image%2020250111000416.png)
![Pasted image 20250111000500.png](/img/user/Pasted%20image%2020250111000500.png)

Bazı hatalar buluyor ama ilginç bir şey yok.

## Shell as puma

### Access Mailtrail

#### Find Exploit

“request-baskets exploit” araması [CVE-2023-27163](https://medium.com/@li_allouche/request-baskets-1-2-1-server-side-request-forgery-cve-2023-27163-2bab94f201f7) hakkındaki bu blog yazısına yönlendiriyor. Bu bir server-side request forgery (SSRF) açığıdır, yani sunucunun benim adıma istek göndermesini sağlayabilirim. Yazıda 1.2.1 sürümünün savunmasız olduğu belirtiliyor:

![Pasted image 20250111000750.png](/img/user/Pasted%20image%2020250111000750.png)

[[Mediıum Request-Baskets Türkçe\|Mediıum Request-Baskets Türkçe]]
