---
{"dg-publish":true,"permalink":"/ctf/forest-htb-oscp-like/"}
---


HTB'nin en güzel yanlarından biri, daha önce karşılaştığım hiçbir CTF'ye benzemeyen Windows konseptlerini ortaya çıkarması. Forest bunun harika bir örneği. RPC üzerinden kullanıcıları listelememe, AS-REP Roasting ile Kerberos'a saldırı yapmama ve bir shell almak için Win-RM kullanmama izin veren bir domain controller. Daha sonra DCSycn yeteneklerini elde etmek için bu kullanıcının izinlerinden ve erişimlerinden faydalanabilirim, böylece admin kullanıcısı için hash'leri dump edebilir ve admin olarak bir shell elde edebilirim. Beyond Root bölümünde, DCSync'in kablo üzerinde nasıl göründüğüne ve izinleri temizleyen otomatik göreve bakacağım. "Beyond Root" kısmında, DCSync'in ağ trafiğinde nasıl göründüğünü inceleyeceğim ve izinleri temizleyen otomatik görevi analiz edeceğim.

## Box Info

|Name|[Forest](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)[![Forest](https://0xdf.gitlab.io/icons/box-forest.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Fforest)|
|---|---|
|Release Date|[12 Oct 2019](https://twitter.com/hackthebox_eu/status/1215300155341266951)|
|Retire Date|21 Mar 2020|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Forest](https://0xdf.gitlab.io/img/forest-diff.png)|
|Radar Graph|![Radar chart for Forest](https://0xdf.gitlab.io/img/forest-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|00:20:45[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|01:23:31[![cube0x0](https://www.hackthebox.com/badge/image/9164)](https://app.hackthebox.com/users/9164)|
|Creators|[![egre55](https://www.hackthebox.com/badge/image/1190)](https://app.hackthebox.com/users/1190)  <br>[![mrb3n](https://www.hackthebox.com/badge/image/2984)](https://app.hackthebox.com/users/2984)|



## Recon

### nmap

nmap, güvenlik duvarı olmayan Windows makinelerine özgü çok sayıda port gösterir:

```
root@kali# nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.161
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:22 EDT
Warning: 10.10.10.161 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.161
Host is up (0.031s latency).
Not shown: 64742 closed ports, 769 filtered ports

PORT STATE SERVICE
53/tcp open domain
88/tcp open kerberos-sec
135/tcp open msrpc
139/tcp open netbios-ssn
389/tcp open ldap
445/tcp open microsoft-ds
464/tcp open kpasswd5
593/tcp open http-rpc-epmap
636/tcp open ldapssl
3268/tcp open globalcatLDAP
3269/tcp open globalcatLDAPssl
5985/tcp open wsman
9389/tcp open adws
47001/tcp open winrm
49664/tcp open unknown
49665/tcp open unknown
49666/tcp open unknown
49667/tcp open unknown
49669/tcp open unknown
49670/tcp open unknown
49671/tcp open unknown
49678/tcp open unknown
49697/tcp open unknown
49898/tcp open unknown

Nmap done: 1 IP address (1 host up) scanned in 20.35 seconds
```


```
root@kali# nmap -sC -sV -p 53,88,135,139,389,445,464,593,636,3268,3269,5985,9389 -oA scans/nmap-tcpscripts 10.10.10.161

Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:24 EDT
Nmap scan report for 10.10.10.161
Host is up (0.030s latency).

PORT     STATE SERVICE      VERSION
53/tcp   open  domain?
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|_    bind
88/tcp   open  kerberos-sec Microsoft Windows Kerberos (server time: 2019-10-14 18:32:33Z)
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn  Microsoft Windows netbios-ssn
389/tcp  open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds Windows Server 2016 Standard 14393 microsoft-ds (workgroup: HTB)
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http   Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap         Microsoft Windows Active Directory LDAP (Domain: htb.local, Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
5985/tcp open  http         Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf       .NET Message Framing
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.80%I=7%D=10/14%Time=5DA4BD82%P=x86_64-pc-linux-gnu%r(DNS
SF:VersionBindReqTCP,20,"\0\x1e\0\x06\x81\x04\0\x01\0\0\0\0\0\0\x07version
SF:\x04bind\0\0\x10\0\x03");
Service Info: Host: FOREST; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 2h27m32s, deviation: 4h02m30s, median: 7m31s
| smb-os-discovery: 
|   OS: Windows Server 2016 Standard 14393 (Windows Server 2016 Standard 6.3)
|   Computer name: FOREST
|   NetBIOS computer name: FOREST\x00
|   Domain name: htb.local
|   Forest name: htb.local
|   FQDN: FOREST.htb.local
|_  System time: 2019-10-14T11:34:51-07:00
| smb-security-mode: 
|   account_used: <blank>
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: required
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2019-10-14T18:34:52
|_  start_date: 2019-10-14T09:52:45

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 281.19 seconds

root@kali# nmap -sU -p- --min-rate 10000 -oA scans/nmap-alludp 10.10.10.161
Starting Nmap 7.80 ( https://nmap.org ) at 2019-10-14 14:30 EDT
Warning: 10.10.10.161 giving up on port because retransmission cap hit (10).
Nmap scan report for 10.10.10.161
Host is up (0.091s latency).
Not shown: 65457 open|filtered ports, 74 closed ports
PORT      STATE SERVICE
123/udp   open  ntp
389/udp   open  ldap
58399/udp open  unknown
58507/udp open  unknown

Nmap done: 1 IP address (1 host up) scanned in 73.74 seconds
```


Nmap Çıktsınından Tespit Ettiklerimiz 

### **1. Hedef Sistem ve Genel Bilgiler**

- **IP Adresi:** 10.10.10.161
- **İşletim Sistemi:** Windows Server 2016 Standard 14393
    - NetBIOS bilgisayar adı: `FOREST`
    - Domain: `htb.local`
    - Forest : `htb.local`
    - Zaman farkı: Sistem saatinde 2 saat 27 dakika sapma gözlemlenmiş.

Bu, hedefin bir Windows Active Directory (AD) ortamına ait olduğunu ve domain yönetimi için kullanılan bir sunucu olduğunu gösteriyor.


### **2. TCP Taraması (Belirli Portlar)**

#### **Açık TCP Portları ve Servisler**

| **Port** | **Durum** | **Hizmet**           | **Versiyon/Detaylar**                                                      |
| -------- | --------- | -------------------- | -------------------------------------------------------------------------- |
| **53**   | Open      | Domain (DNS?)        | DNS sunucusu çalışıyor, ancak servis tam tespit edilmedi.                  |
| **88**   | Open      | Kerberos             | Windows Kerberos kullanılıyor (zaman eşleşmesi: 2019-10-14 18:32:33Z).     |
| **135**  | Open      | Microsoft RPC        | Remote prosedür çağrıları için Microsoft RPC çalışıyor.                    |
| **139**  | Open      | NetBIOS-SSN          | NetBIOS oturumu servisi çalışıyor.                                         |
| **389**  | Open      | LDAP                 | Active Directory için LDAP servisi çalışıyor.                              |
| **445**  | Open      | Microsoft-DS         | SMB (dosya paylaşımı) için kullanılan bir port.                            |
| **464**  | Open      | kpasswd5             | Kerberos ile ilişkili, kullanıcı parolalarını değiştirmek için kullanılır. |
| **593**  | Open      | RPC over HTTP        | Microsoft RPC’nin HTTP üzerinden çalıştığını gösteriyor.                   |
| **636**  | Open      | TCP Wrapped          | Güvenli LDAP (LDAPS) olabilir ama paket analizine ihtiyaç var.             |
| **3268** | Open      | LDAP                 | Global Katalog (Global Catalog) sunucusu çalışıyor.                        |
| **3269** | Open      | TCP Wrapped          | Güvenli Global Katalog olabilir.                                           |
| **5985** | Open      | HTTP                 | Microsoft HTTPAPI kullanıyor, muhtemelen PowerShell Remoting.              |
| **9389** | Open      | .NET Message Framing | Windows’un .NET Message Framework servisi çalışıyor.                       |

#### **Önemli Notlar:**

- **SMB Security Modu:**
    - Mesaj imzalama zorunlu.
    - Kullanıcı doğrulaması gerekli.

Bu, SMB üzerinden güvenlik açıkları aramanın zor olabileceğini gösteriyor. Ancak, doğru kimlik bilgileriyle SMB üzerinden daha fazla bilgi toplanabilir.


### **3. UDP Taraması**

#### **Açık UDP Portları**

|**Port**|**Durum**|**Hizmet**|
|---|---|---|
|**123/udp**|Open|NTP (Network Time Protocol)|
|**389/udp**|Open|LDAP|
|**58399**|Open|Bilinmeyen (Unknown)|
|**58507**|Open|Bilinmeyen (Unknown)|

- **123/udp:** Sistem saatini eşitlemek için kullanılıyor olabilir. Zaman sapmasının dikkatle incelenmesi gerekebilir.
- **389/udp:** LDAP üzerinden iletişim için kullanılabilir.
- **Bilinmeyen UDP Portlar:** Paket analiziyle (Wireshark vb.) bu portların trafiği analiz edilerek hizmet tespiti yapılabilir.


### **4. Potansiyel Saldırı Yüzeyleri**

1. **Active Directory ve LDAP (389/3268 TCP/UDP):**
    
    - LDAP üzerinden bilgi toplama yapılabilir. Örneğin:

```
ldapsearch -h 10.10.10.161 -p 389 -x -b "dc=htb,dc=local"
```

- Kullanıcı ve grup bilgileri elde edilebilir.
- Kimlik bilgileriyle LDAP üzerinden daha fazla yetki artırımı denenebilir.


2. **SMB (139/445):**

- SMB bağlantısı kurarak paylaşılan dosyalar araştırılabilir:

```
smbclient -L //10.10.10.161 -U ''
```

* Null oturumlarla (anonim bağlantılar) bilgi alınabilir.


3. **Kerberos (88/464):**

- Kerberos kimlik doğrulama açıkları araştırılabilir.
- **ASREPRoast ve Kerberoasting** gibi teknikler uygulanabilir:

```
impacket-GetNPUsers htb.local/ -no-pass
```


4. **RPC (135/593):**

- RPC üzerinden servis sorgulamaları yapılabilir:

```
rpcclient -U '' 10.10.10.161
```



5. **NTP (123/udp):**

- NTP hizmetinin kötü yapılandırılmış olup olmadığı kontrol edilebilir.


### **5. Genel Yorum**

- Hedef, bir Active Directory ortamında yer alan bir Windows Server 2016 makinesi.
- LDAP, Kerberos ve SMB servisleri ön planda. Bu servisler, bilgi toplama ve istismar için değerlendirilebilir.
- Özellikle LDAP, SMB ve Kerberos üzerinden bilgi toplama yapılabilir.
- Daha fazla analiz için hedef üzerinde paket analizi (Wireshark) ve kimlik doğrulama denemeleri yapılabilir.



Bu bir domain controller'a benziyor. Ayrıca TCP/5985'i de göreceğim, yani bir kullanıcının kimlik bilgilerini bulabilirsem, WinRM üzerinden bir shell alabilirim.



### DNS - UDP/TCP 53

Bu DNS sunucusundan ==htb.local== ve ==forest.htb.local== adreslerini çözümleyebiliyorum:


```
root@kali# dig  @10.10.10.161 htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> @10.10.10.161 htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 62514
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: bbee567cd8172763 (echoed)
;; QUESTION SECTION:
;htb.local.                     IN      A

;; ANSWER SECTION:
htb.local.              600     IN      A       10.10.10.161

;; Query time: 30 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Mon Oct 14 14:34:17 EDT 2019
;; MSG SIZE  rcvd: 66




root@kali# dig  @10.10.10.161 forest.htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> @10.10.10.161 forest.htb.local
; (1 server found)
;; global options: +cmd
;; Got answer:
;; WARNING: .local is reserved for Multicast DNS
;; You are currently testing what happens when an mDNS query is leaked to DNS
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 12842
;; flags: qr aa rd ra; QUERY: 1, ANSWER: 1, AUTHORITY: 0, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 4000
; COOKIE: ca9fa59dce2451be (echoed)
;; QUESTION SECTION:
;forest.htb.local.              IN      A

;; ANSWER SECTION:
forest.htb.local.       3600    IN      A       10.10.10.161

;; Query time: 150 msec
;; SERVER: 10.10.10.161#53(10.10.10.161)
;; WHEN: Mon Oct 14 14:35:19 EDT 2019
;; MSG SIZE  rcvd: 73
```

Ama zone transfer yapmama izin vermiyor:

```
root@kali# dig axfr  @10.10.10.161 htb.local

; <<>> DiG 9.11.5-P4-5.1+b1-Debian <<>> axfr @10.10.10.161 htb.local
; (1 server found)
;; global options: +cmd
; Transfer failed.
```



### SMB - TCP 445

Ne smbmap ne de smbclient parola olmadan paylaşımları listelememe izin vermiyor:


```
root@kali# smbmap -H 10.10.10.161
[+] Finding open SMB ports....
[+] User SMB session establishd on 10.10.10.161...
[+] IP: 10.10.10.161:445        Name: 10.10.10.161                                      
        Disk                                                    Permissions
        ----                                                    -----------
[!] Access Denied


root@kali# smbmap -H 10.10.10.161 -u 0xdf -p 0xdf
[+] Finding open SMB ports....
[!] Authentication error occured
[!] SMB SessionError: STATUS_LOGON_FAILURE(The attempted logon is invalid. This is either due to a bad username or authentication information.)
[!] Authentication error on 10.10.10.161


root@kali# smbclient -N -L //10.10.10.161
Anonymous login successful

        Sharename       Type      Comment
        ---------       ----      -------
smb1cli_req_writev_submit: called for dialect[SMB3_11] server[10.10.10.161]
Error returning browse list: NT_STATUS_REVISION_MISMATCH
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to 10.10.10.161 failed (Error NT_STATUS_RESOURCE_NAME_NOT_FOUND)
Failed to connect with SMB1 -- no workgroup available
```


### RPC - TCP 445

Kullanıcıları listelemek için RPC üzerinden kontrol etmeyi deneyebilirim. [BlackHills'in bu konuda](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/) iyi bir yazısı var. Yazının türkçelşetirilmiş hali --> [[BlackHills-T\|BlackHills-T]]

Null auth ile bağlanacağım:
