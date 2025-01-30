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


