---
{"dg-publish":true,"permalink":"/active-directory/nmap/"}
---

TCP-SYN taraması (-sS) en popüler yöntemdir. SYN bayrağı gönderilir, tam bağlantı kurulmaz:

- SYN-ACK cevabı: Port açık
- RST cevabı: Port kapalı
- Hiç cevap gelmezse: Filtrelenmiş

```shell-session
[!bash!]$ sudo nmap -sS localhost

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-11 22:50 UTC
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000010s latency).
Not shown: 996 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
5432/tcp open  postgresql
5901/tcp open  vnc-1

Nmap done: 1 IP address (1 host up) scanned in 0.18 seconds
```

# Host Discovery

```shell-session
sudo nmap 10.129.2.0/24 -sn -oA tnet | grep for | cut -d" " -f5

10.129.2.4
10.129.2.10
10.129.2.11
10.129.2.18
10.129.2.19
10.129.2.20
10.129.2.28
```

Bu tarama yöntemi yalnızca hostların güvenlik duvarları buna izin veriyorsa çalışır.

| **Scanning Options** | **Description**                                          |
| -------------------- | -------------------------------------------------------- |
| `10.129.2.0/24`      | Hedef ağ aralığı.                                        |
| `-sn`                | Port taramasını devre dışı bırakır.                      |
| `-oA tnet`           | Sonuçları 'tnet' adıyla başlayan tüm formatlarda saklar. |

## Scan Multiple IPs

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18 10.129.2.19 10.129.2.20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sn -oA tnet 10.129.2.18-20| grep for | cut -d" " -f5

10.129.2.18
10.129.2.19
10.129.2.20
```

Eğer port taramasını (`-sn`) devre dışı bırakırsak, Nmap otomatik olarak `ICMP Echo Requests (-PE)` ile ping taraması yapar. Böyle bir istek gönderildikten sonra, ping atan host canlıysa genellikle bir `ICMP reply` bekleriz. Daha da ilginci, önceki taramalarımız bunu yapmıyordu çünkü Nmap bir `ICMP echo` isteği göndermeden önce, bir `ARP reply` ile sonuçlanan bir `ARP pingi` gönderiyordu. Bunu “`--packet-trace`” seçeneği ile doğrulayabiliriz. ICMP echo isteklerinin gönderildiğinden emin olmak için, bunun için (-PE) seçeneğini de tanımlıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:08 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up (0.023s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

Nmap'in hedefimizi neden “LİVE” olarak işaretlediğini belirlemenin bir başka yolu da “--reason” seçeneğidir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:10 CEST
SENT (0.0074s) ARP who-has 10.129.2.18 tell 10.10.14.2
RCVD (0.0309s) ARP reply 10.129.2.18 is-at DE:AD:00:00:BE:EF
Nmap scan report for 10.129.2.18
Host is up, received arp-response (0.028s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.03 seconds
```

Burada Nmap'in gerçekten de sadece ARP request ve ARP reply ile hostun canlı olup olmadığını tespit ettiğini görüyoruz. ARP isteklerini devre dışı bırakmak ve hedefimizi istediğimiz ICMP echo istekleri ile taramak için “--disable-arp-ping” seçeneğini ayarlayarak ARP pinglerini devre dışı bırakabiliriz. Daha sonra hedefimizi tekrar tarayıp gönderilen ve alınan paketlere bakabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.18 -sn -oA host -PE --packet-trace --disable-arp-ping 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 00:12 CEST
SENT (0.0107s) ICMP [10.10.14.2 > 10.129.2.18 Echo request (type=8/code=0) id=13607 seq=0] IP [ttl=255 id=23541 iplen=28 ]
RCVD (0.0152s) ICMP [10.129.2.18 > 10.10.14.2 Echo reply (type=0/code=0) id=13607 seq=0] IP [ttl=128 id=40622 iplen=28 ]
Nmap scan report for 10.129.2.18
Host is up (0.086s latency).
MAC Address: DE:AD:00:00:BE:EF
Nmap done: 1 IP address (1 host up) scanned in 0.11 seconds
```

Host keşfi hakkında daha fazla stratejiyi şu adreste bulabilirsiniz:

https://nmap.org/book/host-discovery-strategies.html


Soru  : Son sonuca göre, hangi işletim sistemine ait olduğunu bulun. Sonuç olarak işletim sisteminin adını gönderin.

Cevap : Windows (ttl değeri 128 olduğu için)


# Host and Port Scanning

| **State**            | **Description**                                                                                                                                                                         |
| -------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **open**             | Taranan porta bağlantının kurulduğunu gösterir. Bu bağlantılar TCP bağlantıları, UDP datagramları veya SCTP birleşimleri olabilir.                                                      |
| **closed**           | Portun kapalı olarak gösterilmesi, TCP protokolünün aldığımız paketin RST bayrağı içerdiğini belirttiği anlamına gelir. Hedefin aktif olup olmadığını belirlemek için kullanılabilir.   |
| **filtered**         | Nmap, taranan portun açık mı yoksa kapalı mı olduğunu doğru bir şekilde tespit edemez çünkü ya hedeften port için bir yanıt alınmaz ya da hedeften bir hata kodu alırız.                |
| **unfiltered**       | Bu port durumu yalnızca ==TCP-ACK== taraması sırasında ortaya çıkar. Porta erişilebilir, ancak açık mı yoksa kapalı mı olduğunun belirlenemediği anlamına gelir.                        |
| **open\|filtered**   | Belirli bir port için yanıt alınamadığında Nmap bu durumu ayarlar. Bu, portun bir güvenlik duvarı veya paket filtresi tarafından korunduğunu gösterebilir.                              |
| **closed\|filtered** | Bu durum yalnızca ==IP ID idle== taramalarında ortaya çıkar. Taranan portun kapalı mı yoksa bir güvenlik duvarı tarafından filtrelenip filtrelenmediğinin belirlenemediğini ifade eder. |


## Discovering Open TCP Ports

Varsayılan olarak, Nmap en çok kullanılan 1000 TCP portunu SYN taraması (`-sS`) ile tarar. Bu SYN taraması, yalnızca ==root== kullanıcısı olarak çalıştırıldığında varsayılan olarak ayarlanır, çünkü raw TCP paketleri oluşturmak için gereken soket izinlerine ihtiyaç duyar. Aksi takdirde, varsayılan olarak TCP taraması (==-sT==) gerçekleştirilir. Bu, eğer portları ve tarama yöntemlerini tanımlamazsak, bu parametrelerin otomatik olarak ayarlandığı anlamına gelir. Portları tek tek tanımlayabiliriz (-p 22,25,80,139,445), aralık olarak belirtebiliriz (-p 22-445), Nmap veritabanında en sık kullanılan olarak işaretlenmiş üst portları tarayabiliriz (--top-ports=10), tüm portları tarayabiliriz (-p-) veya hızlı port taraması yapabiliriz, ki bu da en çok kullanılan 100 portu içerir (-F).

- **Root değilseniz:** Nmap varsayılan olarak **TCP Connect Scan** (-sT) yapar.
- **Root iseniz:** Nmap varsayılan olarak **SYN Scan** (-sS) yapar.


#### Scanning Top 10 TCP Ports

```shell-session
sudo nmap 10.129.2.28 --top-ports=10 
```



#### Nmap - Trace the Packets

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 21 --packet-trace -Pn -n --disable-arp-ping

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:39 CEST
SENT (0.0429s) TCP 10.10.14.2:63090 > 10.129.2.28:21 S ttl=56 id=57322 iplen=44  seq=1699105818 win=1024 <mss 1460>
RCVD (0.0573s) TCP 10.129.2.28:21 > 10.10.14.2:63090 RA ttl=64 id=0 iplen=40  seq=0 win=0
Nmap scan report for 10.11.1.28
Host is up (0.014s latency).

PORT   STATE  SERVICE
21/tcp closed ftp
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.07 seconds
```


|                      |                                      |
| -------------------- | ------------------------------------ |
| `-n`                 | DNS resolution'ı devre dışı bırakır. |
| `--disable-arp-ping` | ARP ping'i devre dışı bırakır.       |

**SENT** satırından, hedefimize (10.129.2.28) SYN bayraklı (S) bir TCP paketi gönderdiğimizi (10.10.14.2) görebiliriz. Sonraki **RCVD** satırında, hedefin RST ve ACK bayrakları (RA) içeren bir TCP paketiyle yanıt verdiğini görebiliriz. RST ve ACK bayrakları, TCP paketinin alındığını onaylamak (ACK) ve TCP oturumunu sonlandırmak (RST) için kullanılır.


#### Request
| **Mesaj**                                                     | **Açıklama**                                                                             |
| ------------------------------------------------------------- | ---------------------------------------------------------------------------------------- |
| **SENT (0.0429s)**                                            | Nmap'in SENT işlemini gösterir, hedefe bir paket gönderir.                               |
| **TCP**                                                       | Hedef portla etkileşimde kullanılan protokolü gösterir.                                  |
| **10.10.14.2:63090 >**                                        | Nmap'in paketleri göndermek için kullandığı IPv4 adresimizi ve kaynak portu temsil eder. |
| **10.129.2.28:21**                                            | Hedefin IPv4 adresini ve hedef portu gösterir.                                           |
| **S**                                                         | Gönderilen TCP paketinin SYN bayrağını belirtir.                                         |
| **ttl=56 id=57322 iplen=44 seq=1699105818 win=1024 mss 1460** | TCP başlığına ait ek parametreleri gösterir.                                             |


#### Response

| **Mesaj**                            | **Açıklama**                                                                    |
| ------------------------------------ | ------------------------------------------------------------------------------- |
| **RCVD (0.0573s)**                   | Hedeften alınan bir paketi gösterir.                                            |
| **TCP**                              | Kullanılan protokolü gösterir.                                                  |
| **10.129.2.28:21 >**                 | Hedefin IPv4 adresini ve yanıt vermek için kullanılan kaynak portu temsil eder. |
| **10.10.14.2:63090**                 | Bizim IPv4 adresimizi ve yanıtın gönderileceği portu gösterir.                  |
| **RA**                               | Gönderilen TCP paketinin RST ve ACK bayraklarını belirtir.                      |
| **ttl=64 id=0 iplen=40 seq=0 win=0** | TCP başlığına ait ek parametreleri gösterir.                                    |

### Connect Scan 

Nmap TCP Connect Scan (-sT), hedef bir sistemde belirli bir portun açık veya kapalı olup olmadığını belirlemek için TCP üçlü el sıkışmasını (three-way handshake) kullanır. Tarama, hedef porta bir SYN paketi gönderir ve bir yanıt bekler. Eğer hedef port bir SYN-ACK paketi ile yanıt verirse, port açık olarak kabul edilir. Eğer RST paketi ile yanıt verirse, port kapalıdır.

Connect Scan (tam TCP bağlantı taraması olarak da bilinir), TCP tree handshake tamamladığı için oldukça doğru sonuçlar verir ve bir portun durumunu (açık, kapalı veya filtrelenmiş) kesin olarak belirlememizi sağlar. Ancak, bu yöntem en gizli tarama tekniklerinden biri değildir. Hatta, Connect Scan en az gizli tarama yöntemlerinden biri olarak kabul edilir çünkü tam bir bağlantı kurar ve bu da çoğu sistemde log kayıtları oluşturur.

Ayrıca, hedef sistemde gelen paketleri engelleyen ancak giden paketlere izin veren kişisel bir güvenlik duvarı (firewall) varsa, Connect Scan bu firewall'ı atlayabilir ve hedef portların durumunu doğru bir şekilde belirleyebilir. Ancak, Connect Scan diğer tarama türlerine göre daha yavaştır çünkü tarayıcının, gönderdiği her paketten sonra hedeften bir yanıt beklemesi gerekir. Bu, hedef meşgul veya yanıt vermiyorsa biraz zaman alabilir.

SYN taraması (yarı açık tarama olarak da bilinir) gibi taramalar genellikle daha gizli kabul edilir çünkü full handshake tamamlamaz ve başlangıçta gönderilen SYN paketinden sonra bağlantıyı tamamlamaz.

#### Connect Scan on TCP Port 443

```shell-session
sudo nmap 10.129.2.28 -p 443 --packet-trace --disable-arp-ping -Pn -n --reason -sT
```


#### Filtered Ports

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 139 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:45 CEST
SENT (0.0381s) TCP 10.10.14.2:60277 > 10.129.2.28:139 S ttl=47 id=14523 iplen=44  seq=4175236769 win=1024 <mss 1460>
SENT (1.0411s) TCP 10.10.14.2:60278 > 10.129.2.28:139 S ttl=45 id=7372 iplen=44  seq=4175171232 win=1024 <mss 1460>
Nmap scan report for 10.129.2.28
Host is up.

PORT    STATE    SERVICE
139/tcp filtered netbios-ssn
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

|Tarama Seçenekleri|Açıklama|
|---|---|
|**10.129.2.28**|Belirtilen hedefi tarar.|
|**-p 139**|Sadece belirtilen portu tarar.|
|**--packet-trace**|Gönderilen ve alınan tüm paketleri gösterir.|
|**-n**|DNS çözümlemesini devre dışı bırakır.|
|**--disable-arp-ping**|ARP ping işlemini devre dışı bırakır.|
|**-Pn**|ICMP Echo isteklerini devre dışı bırakır.|

Son taramada Nmap'in SYN bayrağı ile iki TCP paketi gönderdiğini görüyoruz. Taramanın süresinden (2.06s), öncekilerden (~0.05s) çok daha uzun sürdüğünü anlayabiliriz. Firewall paketleri reddediyorsa durum farklıdır. Bunun için, firewall'un böyle bir kuralı tarafından uygun şekilde işlenen TCP port 445'e bakıyoruz.


```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -p 445 --packet-trace -n --disable-arp-ping -Pn

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 15:55 CEST
SENT (0.0388s) TCP 10.129.2.28:52472 > 10.129.2.28:445 S ttl=49 id=21763 iplen=44  seq=1418633433 win=1024 <mss 1460>
RCVD (0.0487s) ICMP [10.129.2.28 > 10.129.2.28 Port 445 unreachable (type=3/code=3) ] IP [ttl=64 id=20998 iplen=72 ]
Nmap scan report for 10.129.2.28
Host is up (0.0099s latency).

PORT    STATE    SERVICE
445/tcp filtered microsoft-ds
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.05 seconds
```

Bir yanıt olarak, **type 3** ve **code 3** olan bir ICMP yanıtı alıyoruz. Bu, istenen portun erişilemez olduğunu belirtir. Ancak, eğer host'un aktif olduğunu biliyorsak, bu durumda ilgili port üzerinde bir **firewall**'ın paketleri reddettiğini güçlü bir şekilde varsayabiliriz ve bu portu daha detaylı incelememiz gerekecektir.


### Discovering Open UDP Ports

Bazı sistem yöneticileri, TCP portlarını filtrelemeye ek olarak UDP portlarını filtrelemeyi bazen unutabilir. UDP, **stateless** (durumsuz) bir protokol olduğu için TCP gibi **three-way handshake** gerektirmez. Bu nedenle, herhangi bir **acknowledgment** (onay) almayız. Sonuç olarak, zaman aşımı süresi çok daha uzundur ve bu durum tüm UDP taramasını (**-sU**) TCP taramasına (**-sS**) kıyasla çok daha yavaş hale getirir.

Şimdi bir UDP taramasının (**-sU**) nasıl göründüğüne ve bize hangi sonuçları verdiğine dair bir örneğe bakalım.

#### UDP Port Scan

```shell-session
sudo nmap 10.129.2.28 -F -sU
```

|      |                      |
| ---- | -------------------- |
| `-F` | Scans top 100 ports. |

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 137 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:15 CEST
SENT (0.0367s) UDP 10.10.14.2:55478 > 10.129.2.28:137 ttl=57 id=9122 iplen=78
RCVD (0.0398s) UDP 10.129.2.28:137 > 10.10.14.2:55478 ttl=64 id=13222 iplen=257
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.0031s latency).

PORT    STATE SERVICE    REASON
137/udp open  netbios-ns udp-response ttl 64
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 0.04 seconds
```


|            |                                                               |
| ---------- | ------------------------------------------------------------- |
| `--reason` | Bir portun belirli bir durumda olmasının nedenini görüntüler. |
Error code 3 (port unreachable) ile bir ICMP response alırsak, portun gerçekten kapalı olduğunu biliriz.

```shell-session
sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 100 --reason 
```

Diğer tüm ICMP yanıtları için, taranan portlar ((open|filtered)) olarak işaretlenir.

**`open|filtered`**, Nmap çıktısında, bir portun açık (**open**) veya bir firewall ya da filtre tarafından engellendiği (**filtered**) durumunu ayırt edemediği anlamına gelir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -sU -Pn -n --disable-arp-ping --packet-trace -p 138 --reason 

Starting Nmap 7.80 ( https://nmap.org ) at 2020-06-15 16:32 CEST
SENT (0.0380s) UDP 10.10.14.2:52341 > 10.129.2.28:138 ttl=50 id=65159 iplen=28
SENT (1.0392s) UDP 10.10.14.2:52342 > 10.129.2.28:138 ttl=40 id=24444 iplen=28
Nmap scan report for 10.129.2.28
Host is up, received user-set.

PORT    STATE         SERVICE     REASON
138/udp open|filtered netbios-dgm no-response
MAC Address: DE:AD:00:00:BE:EF (Intel Corporate)

Nmap done: 1 IP address (1 host up) scanned in 2.06 seconds
```

#### Version Scan

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.2.28 -Pn -n --disable-arp-ping --packet-trace -p 445 --reason  -sV

Starting Nmap 7.80 ( https://nmap.org ) at 2022-11-04 11:10 GMT
SENT (0.3426s) TCP 10.10.14.2:44641 > 10.129.2.28:445 S ttl=55 id=43401 iplen=44  seq=3589068008 win=1024 <mss 1460>
RCVD (0.3556s) TCP 10.129.2.28:445 > 10.10.14.2:44641 SA ttl=63 id=0 iplen=44  seq=2881527699 win=29200 <mss 1337>
NSOCK INFO [0.4980s] nsock_iod_new2(): nsock_iod_new (IOD #1)
NSOCK INFO [0.4980s] nsock_connect_tcp(): TCP connection requested to 10.129.2.28:445 (IOD #1) EID 8
NSOCK INFO [0.5130s] nsock_trace_handler_callback(): Callback: CONNECT SUCCESS for EID 8 [10.129.2.28:445]
Service scan sending probe NULL to 10.129.2.28:445 (tcp)
NSOCK INFO [0.5130s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 6000ms) EID 18
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: READ TIMEOUT for EID 18 [10.129.2.28:445]
Service scan sending probe SMBProgNeg to 10.129.2.28:445 (tcp)
NSOCK INFO [6.5190s] nsock_write(): Write request for 168 bytes to IOD #1 EID 27 [10.129.2.28:445]
NSOCK INFO [6.5190s] nsock_read(): Read request from IOD #1 [10.129.2.28:445] (timeout: 5000ms) EID 34
NSOCK INFO [6.5190s] nsock_trace_handler_callback(): Callback: WRITE SUCCESS for EID 27 [10.129.2.28:445]
NSOCK INFO [6.5320s] nsock_trace_handler_callback(): Callback: READ SUCCESS for EID 34 [10.129.2.28:445] (135 bytes)
Service scan match (Probe SMBProgNeg matched with SMBProgNeg line 13836): 10.129.2.28:445 is netbios-ssn.  Version: |Samba smbd|3.X - 4.X|workgroup: WORKGROUP|
NSOCK INFO [6.5320s] nsock_iod_delete(): nsock_iod_delete (IOD #1)
Nmap scan report for 10.129.2.28
Host is up, received user-set (0.013s latency).

PORT    STATE SERVICE     REASON         VERSION
445/tcp open  netbios-ssn syn-ack ttl 63 Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
Service Info: Host: Ubuntu

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.55 seconds
```

Port tarama teknikleri hakkında daha fazla bilgiyi şu adreste bulabilirsiniz: https://nmap.org/book/man-port-scanning-techniques.html


Soru : Hedefinizdeki tüm TCP portlarını bulun. Bulunan TCP portlarının toplam sayısını cevap olarak gönderin.

Cevap : 7

![Pasted image 20250131012651.png](/img/user/resimler/Pasted%20image%2020250131012651.png)

Soru : Hedefinizin host adını numaralandırın ve cevap olarak gönderin. (büyük/küçük harfe duyarlı)

Cevap : 

![Pasted image 20250131012721.png](/img/user/resimler/Pasted%20image%2020250131012721.png)

![Pasted image 20250131012711.png](/img/user/resimler/Pasted%20image%2020250131012711.png)
