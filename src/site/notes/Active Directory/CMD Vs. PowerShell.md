---
{"dg-publish":true,"permalink":"/active-directory/cmd-vs-power-shell/"}
---



Bu noktaya kadar, built-in Windows Command-line interpreter cmd.exe'yi ele aldık. İleride, Window'un CMD'nin modern halefi PowerShell'e bakacağız. Bu bölümde [PowerShell](https://learn.microsoft.com/en-us/powershell/scripting/overview?view=powershell-7.2)'in ne olduğu, PowerShell ve CMD arasındaki farklar, CLI içinde nasıl yardım alınacağı ve CLI içinde temel gezinme konuları ele alınacaktır.


### Farklılıklar
PowerShell ve CMD her Windows hostunda yerel olarak bulunur, bu nedenle kendimize “Neden birini diğerine tercih edeyim?” diye sorabiliriz. Bunu hızlıca ele alalım. Aşağıda PowerShell ve CMD arasındaki bazı farkları içeren bir tablo bulunmaktadır.


### PowerShell ve CMD Karşılaştırıldı
![Pasted image 20241008003901.png](/img/user/resimler/Pasted%20image%2020241008003901.png)

En önemlisi, PowerShell genişletilebilir olacak ve gerektiğinde diğer birçok araç ve fonksiyon ile entegre olacak şekilde inşa edilmiştir. Çoğu kişi bunu sadece başka bir CLI olarak düşünür, ancak çok daha fazlasıdır. Aynı zamanda bir komut dosyası dili olduğunu biliyor muydunuz? CMD yalnızca Windows host'lar için varsayılan komut satırı yorumlayıcısı iken, PowerShell açık kaynaklı bir proje olarak piyasaya sürülmüştür ve Linux tabanlı sistemlerde de kullanımını destekleyen kapsamlı yeteneklere sahiptir. .NET framework'ün kullanılması [PowerShell](https://github.com/PowerShell/PowerShell)'in sadece metin tabanlı yerine nesne tabanlı bir etkileşim ve çıktı modeli kullanabilmesini de sağlamıştır.


### Neden cmd.exe Yerine PowerShell'i Seçmelisiniz?
PowerShell BT yöneticileri, Saldırgan ve Savunmacı Infosec uzmanları için neden önemlidir?

[PowerShell](https://docs.microsoft.com/en-us/powershell/), BT ve Infosec profesyonelleri arasında giderek daha fazla öne çıkmaktadır. Sistem Yöneticileri, Sızma Test Uzmanları, SOC Analistleri ve Windows sistemlerinin yönetildiği diğer birçok teknik disiplin için yaygın bir kullanım alanına sahiptir. Windows sunucuları, masaüstü bilgisayarlar (Windows 10 & 11), Azure ve Microsoft 365 bulut tabanlı uygulamalardan oluşan BT ortamlarını yöneten BT yöneticilerini ve Windows sistem yöneticilerini düşünün. Birçoğu günlük olarak gerçekleştirmeleri gereken görevleri otomatikleştirmek için PowerShell kullanıyor. Bu görevlerden bazıları şunlardır:

* Sunucuları hazırlama ve sunucu rollerini yükleme
* Yeni çalışanlar için Active Directory kullanıcı hesapları oluşturma
* Active Directory grup izinlerini yönetme
* Active Directory kullanıcı hesaplarını devre dışı bırakma ve silme
* Dosya paylaşım izinlerini yönetme
* Azure AD ve [Azure](https://azure.microsoft.com/en-us/) VM'leri ile etkileşim kurma
* Dizin ve dosya oluşturma, silme ve izleme
* Workstation'lar ve sunucular hakkında bilgi toplama
* Kullanıcılar için Microsoft Exchange e-posta gelen kutularını ayarlama (bulutta ve/veya şirket içinde)

PowerShell'i BT yöneticisi açısından kullanmanın sayısız yolu vardır ve bu konuda dikkatli olmak sızma testçileri ve hatta savunmacılar olarak bizim için yararlı olabilir. Bir sysadmin olarak PowerShell bize CMD'den çok daha fazla yetenek sağlayabilir. Genişletilebilir, otomasyon ve komut dosyası oluşturma için üretilmiştir, çok daha sağlam bir güvenlik uygulamasına sahiptir ve CMD'nin yapamayacağı birçok farklı görevi yerine getirebilir. Bir pentester olarak, birçok iyi bilinen yetenek PowerShell'de yerleşiktir. PowerShell'in modül içe aktarma özelliği, araçlarımızı ortama getirmeyi ve çalışacaklarından emin olmayı kolaylaştırır. Bununla birlikte, gizli bir bakış açısıyla, PowerShell'in günlük kaydı ve geçmiş özelliği güçlüdür ve host ile etkileşimlerimizin daha fazlasını günlüğe kaydedecektir. Bu nedenle, PowerShell'in yeteneklerine ihtiyacımız yoksa ve daha gizli olmak istiyorsak, CMD'yi kullanmalıyız.


## Calling PowerShell
PowerShell'e lokal makineye bağlı çevre birimleri aracılığıyla doğrudan bir host üzerinden veya çeşitli yöntemlerle ağ üzerinden RDP aracılığıyla erişebiliriz.

1.Windows Arama'yı Kullanma

PowerShell uygulamasını ve komut konsolunu bulmak ve başlatmak için Windows Arama'ya PowerShell yazabiliriz.

2.Windows Terminal Uygulamasını Kullanma

[Windows Terminal](https://github.com/Microsoft/Terminal), Windows komut satırını kullanan herkese tek bir uygulama aracılığıyla birden fazla farklı komut satırı arayüzüne, sisteme ve alt sisteme erişmenin bir yolunu sunmak için Microsoft tarafından geliştirilen daha yeni bir terminal emülatörü uygulamasıdır. Bu uygulama muhtemelen Windows işletim sistemlerinde varsayılan terminal emülatörü haline gelecektir.


 3.Using Windows `PowerShell ISE`

[Windows PowerShell Integrated Scripting Environment (ISE),](https://learn.microsoft.com/en-us/powershell/scripting/windows-powershell/ise/introducing-the-windows-powershell-ise?view=powershell-7.2) PowerShell için bir IDE gibidir. Oluşturduğumuz PowerShell komut dosyalarını geliştirmeyi, hata ayıklamayı ve test etmeyi kolaylaştırabilir. PowerShell ISE'yi kullanmak PowerShell öğrenirken inanılmaz derecede faydalı olabilir.

![Pasted image 20241010130359.png](/img/user/resimler/Pasted%20image%2020241010130359.png)

4.Using PowerShell in `CMD`

PowerShell'i CMD içinden de başlatabiliriz. Bu eylem önemsiz görünebilir, ancak hiç şüphesiz CMD aracılığıyla savunmasız bir Windows hedefinin CLI'sında bir shell elde edebileceğimiz ve host üzerinde ve ağ üzerinde erişimimizi ilerletmek için PowerShell'i kullanmaya çalışmaktan yararlanacağımız bir zaman gelecektir.

![Pasted image 20241010130512.png](/img/user/resimler/Pasted%20image%2020241010130512.png)


### Shell'e Bir Bakış
Lokal veya remote bir sistemde PowerShell'e erişirken ilk inceleyebileceğimiz şeylerden biri komut istemidir.

## PowerShell Prompt
```powershell-session
PS C:\Users\htb-student> ipconfig 

Ethernet adapter VMware Network Adapter VMnet8:

   Connection-specific DNS Suffix  . :
   Link-local IPv6 Address . . . . . : fe80::adb8:3c9:a8af:114%25
   IPv4 Address. . . . . . . . . . . : 172.16.110.1
   Subnet Mask . . . . . . . . . . . : 255.255.255.0
   Default Gateway . . . . . . . . . :
```

Komut istemi CMD'de gördüğümüzle neredeyse aynıdır.
* PS, PowerShell'in kısaltmasıdır ve ardından geçerli çalışma dizini C:\Users\htb-student> gelir.
* Bunu çalıştırmak istediğimiz cmdlet veya string takip eder, ipconfig.
* Son olarak, bunun altında, komutumuzun çıktı sonuçlarını görüyoruz.

Yine CMD'ye benzer şekilde, PowerShell bize kullanmamız için birçok komut ve cmdlet sunar. 


### Get-Help
Yardım fonksiyonunu kullanma. Belirli bir cmdlet ile kullanabileceğimiz seçenekleri ve fonksiyonları görmek istiyorsak [Get-Help](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-help?view=powershell-7.2) cmdlet'ini kullanabiliriz.


#### Using Get-Help
```powershell-session
PS C:\Users\htb-student> Get-Help Test-Wsman

NAME
    Test-WSMan

SYNTAX
    Test-WSMan [[-ComputerName] <string>] [-Authentication {None | Default | Digest | Negotiate | Basic | Kerberos |
    ClientCertificate | Credssp}] [-Port <int>] [-UseSSL] [-ApplicationName <string>] [-Credential <pscredential>]
    [-CertificateThumbprint <string>]  [<CommonParameters>]


ALIASES
    None


REMARKS
    Get-Help cannot find the Help files for this cmdlet on this computer. It is displaying only partial help.
        -- To download and install Help files for the module that includes this cmdlet, use Update-Help.
        -- To view the Help topic for this cmdlet online, type: "Get-Help Test-WSMan -Online" or
           go to https://go.microsoft.com/fwlink/?LinkId=141464.
```
Get-Help bir cmdlet hakkında yararlı bilgiler verebilir. Syntax çıktısının bize mevcut birkaç seçeneği ve her seçenekle birlikte kullanılabilecek ek anahtar sözcükleri gösterdiğine dikkat edin. Komutlarımız için daha kısa adlar olan ==takma adlardan (Allies==) da bahsedilmektedir. Takma adları bu bölümün ilerleyen kısımlarında daha ayrıntılı olarak ele alacağız. ==Remarks== çıktısı bize cmdlet hakkında daha fazla bilgi ve hatta cmdlet hakkında daha fazla bilgi edinmek için kullanabileceğimiz ek seçenekler sağlar. Bu ek seçeneklerden biri olan ==-online==, host'un İnternet erişimi varsa ilgili cmdlet için bir Microsoft docs web sayfası açacaktır.

![Pasted image 20241010131848.png](/img/user/resimler/Pasted%20image%2020241010131848.png)

Windows sistemindeki her cmdlet için en güncel bilgilere sahip olduğumuzdan emin olmak için [Update-Help](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/update-help?view=powershell-7.2) adlı yararlı bir cmdlet de kullanabiliriz.


#### Using Update-Help

```powershell-session
PS C:\Windows\system32> Update-Help
```

Update-Help'i çalıştırdıktan sonra Test-Wsman ile ilgili ne kadar çok bilginin doldurulduğuna dikkat edin. Bu çıktıyı daha önce Get-Help'i ilk kez ele aldığımızda gösterilen çıktı ile karşılaştırmaktan çekinmeyin.

![Pasted image 20241010131958.png](/img/user/resimler/Pasted%20image%2020241010131958.png)

```powershell-session
PS C:\Windows\system32> Get-Help  Test-Wsman

NAME
    Test-WSMan

SYNOPSIS
    Tests whether the WinRM service is running on a local or remote computer.


SYNTAX
    Test-WSMan [[-ComputerName] <System.String>] [-ApplicationName <System.String>]
    [-Authentication {None | Default | Digest | Negotiate | Basic | Kerberos |
    ClientCertificate | Credssp}] [-CertificateThumbprint <System.String>]
    [-Credential <System.Management.Automation.PSCredential>] [-Port <System.Int32>]
    [-UseSSL] [<CommonParameters>]


DESCRIPTION
    The `Test-WSMan` cmdlet submits an identification request that determines
    whether the WinRM service is running on a local or remote computer. If the
    tested computer is running the service, the cmdlet displays the WS-Management
    identity schema, the protocol version, the product vendor, and the product
    version of the tested service.


RELATED LINKS
    Online Version: https://docs.microsoft.com/powershell/module/microsoft.wsman.mana
    gement/test-wsman?view=powershell-5.1&WT.mc_id=ps-gethelp
    Connect-WSMan
    Disable-WSManCredSSP
    Disconnect-WSMan
    Enable-WSManCredSSP
    Get-WSManCredSSP
    Get-WSManInstance
    Invoke-WSManAction
    New-WSManInstance
    New-WSManSessionOption
    Remove-WSManInstance
    Set-WSManInstance
    Set-WSManQuickConfig

REMARKS
    To see the examples, type: "get-help Test-WSMan -examples".
    For more information, type: "get-help Test-WSMan -detailed".
    For technical information, type: "get-help Test-WSMan -full".
    For online help, type: "get-help Test-WSMan -online"
```

### PowerShell'de Dolaşmak
PowerShell'in ne olduğunu ve built-in yardım özelliklerinin temellerini ele aldığımıza göre, şimdi PowerShell'in temel gezinme ve kullanımına geçelim.


### Neredeyiz?
Sadece nerede olduğumuzu biliyorsak hareket edebiliriz, değil mi? ==Get-Location== cmdlet'ini kullanarak mevcut çalışma dizinimizi (ana sisteme göre) belirleyebiliriz.

#### Get-Location
```powershell-session
PS C:\Users\BEN 10> Get-Location

Path
----
C:\Users\BEN 10


PS C:\Users\BEN 10> pwd

Path
----
C:\Users\BEN 10
```

O anda çalıştığımız dizinin tam yolunu yazdırdığını görebiliriz; bu durumda bu C:\Users\DLarusso olacaktır. Artık yönümüzü bulduğumuza göre, bu dizin içinde hangi nesnelerin ve dosyaların bulunduğuna bakalım.


### List the Directory
==Get-ChildItem== cmdlet'i geçerli dizinimizin veya belirttiğimiz dizinin içeriğini görüntüleyebilir.


#### Get-ChildItem
```powershell-session
PS C:\Users\BEN 10> Get-ChildItem


    Directory: C:\Users\BEN 10


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---        25.09.2024     21:47                3D Objects
d-r---        25.09.2024     21:47                Contacts
d-r---        10.10.2024     12:47                Desktop
d-r---         2.10.2024     23:45                Documents
d-r---        10.10.2024     12:47                Downloads
d-r---        25.09.2024     21:47                Favorites
d-r---        25.09.2024     21:47                Links
d-r---        25.09.2024     21:47                Music
d-r---        25.09.2024     21:48                OneDrive
d-r---        25.09.2024     21:48                Pictures
d-r---        25.09.2024     21:47                Saved Games
d-r---        25.09.2024     21:48                Searches
d-----        28.09.2024     19:54                source
d-r---        26.09.2024     18:39                Videos


PS C:\Users\BEN 10> ls


    Directory: C:\Users\BEN 10


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-r---        25.09.2024     21:47                3D Objects
d-r---        25.09.2024     21:47                Contacts
d-r---        10.10.2024     12:47                Desktop
d-r---         2.10.2024     23:45                Documents
d-r---        10.10.2024     12:47                Downloads
d-r---        25.09.2024     21:47                Favorites
d-r---        25.09.2024     21:47                Links
d-r---        25.09.2024     21:47                Music
d-r---        25.09.2024     21:48                OneDrive
d-r---        25.09.2024     21:48                Pictures
d-r---        25.09.2024     21:47                Saved Games
d-r---        25.09.2024     21:48                Searches
d-----        28.09.2024     19:54                source
d-r---        26.09.2024     18:39                Videos

```
Mevcut çalışma dizinimiz içinde birkaç başka dizin görebiliriz. Bir tanesini inceleyelim.

### Yeni Bir Dizine Geçme
Konumumuzu değiştirmek basittir; bunu ==Set-Location== cmdlet'ini kullanarak yapabiliriz.


#### Set-Location
```powershell-session
PS C:\htb>  Set-Location .\Documents\

PS C:\Users\tru7h\Documents> Get-Location

Path
----
C:\Users\DLarusso\Documents
```

Set-Location cmdlet'ine .\Documents\ parametrelerini girdik ve PowerShell'e geçerli çalışma dizinimizde bulunan Belgeler dizinine geçiş yapmak istediğimizi söyledik. Bu şekilde tam dosya yolunu da verebilirdik:

```powershell
Set-Location C:\Users\DLarusso\Documents 
```


### Dosya İçeriğini Görüntüleme
Şimdi, bir dosyanın içeriğini görmek istersek Get-Content'i kullanabiliriz. Belgeler dizinine baktığımızda Readme.md adında bir dosya görüyoruz. Hadi kontrol edelim.
```powershell-session
PS C:\htb> Get-Content Readme.md  

# ![logo][] PowerShell

Welcome to the PowerShell GitHub Community!
PowerShell Core is a cross-platform (Windows, Linux, and macOS) automation and configuration tool/framework that works well with your existing tools and is optimized
for dealing with structured data (e.g., JSON, CSV, XML, etc.), REST APIs, and object models.
It includes a command-line shell, an associated scripting language and a framework for processing cmdlets. 

<SNIP> 
```

Readme dosyası PowerShell GitHub sayfasından alınmış gibi görünüyor. ==Get-Content== cmdlet'ini kullanmak bu kadar basittir. PowerShell CLI içinde gezinmek oldukça basittir. Artık bu beceriye sahip olduğumuza göre, CLI kullanımını daha da sorunsuz hale getirebilecek birkaç yararlı ipucu ve püf noktasına bakalım.


### PowerShell Kullanımı için İpuçları ve Püf Noktaları

### Get-Command
Get-Command, tam kullanmamız gereken anda hafızamızdan kaybolabilecek sinir bozucu bir komutu bulmak için harika bir yoldur. PowerShell'in cmdlet'ler için ==verb-noun== kuralını kullanmasıyla, herhangi birinde arama yapabiliriz.


#### Get-Command Usage

```powershell-session
PS C:\htb> Get-Command

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Add-AppPackage                                     2.0.1.0    Appx
Alias           Add-AppPackageVolume                               2.0.1.0    Appx
Alias           Add-AppProvisionedPackage                          3.0        Dism
Alias           Add-ProvisionedAppPackage                          3.0        Dism
Alias           Add-ProvisionedAppxPackage                         3.0        Dism
Alias           Add-ProvisioningPackage                            3.0        Provisioning
Alias           Add-TrustedProvisioningCertificate                 3.0        Provisioning
Alias           Apply-WindowsUnattend                              3.0        Dism
Alias           Disable-PhysicalDiskIndication                     2.0.0.0    Storage
Alias           Disable-StorageDiagnosticLog                       2.0.0.0    Storage
Alias           Dismount-AppPackageVolume                          2.0.1.0    Appx
Alias           Enable-PhysicalDiskIndication                      2.0.0.0    Storage
Alias           Enable-StorageDiagnosticLog                        2.0.0.0    Storage
Alias           Flush-Volume                                       2.0.0.0    Storage
Alias           Get-AppPackage                                     2.0.1.0    Appx

<SNIP>  
```

Yukarıdaki çıktı, ekran alanından tasarruf etmek için kısaltılmıştır. Get-Command'ı ek düzenleyiciler olmadan kullanmak, o anda PowerShell oturumuna yüklenmiş olan her cmdlet'in tam bir çıktısını gerçekleştirecektir. Cmdlet'in fiil veya isim kısmını filtreleyerek bunu daha da azaltabiliriz.


#### Get-Command (verb)
```powershell-session
PS C:\htb> Get-Command -verb get

<SNIP>
Cmdlet          Get-Acl                                            3.0.0.0    Microsoft.Pow...
Cmdlet          Get-Alias                                          3.1.0.0    Microsoft.Pow...
Cmdlet          Get-AppLockerFileInformation                       2.0.0.0    AppLocker
Cmdlet          Get-AppLockerPolicy                                2.0.0.0    AppLocker
Cmdlet          Get-AppvClientApplication                          1.0.0.0    AppvClient  
<SNIP>  
```

-verb düzenleyicisini kullanarak ve adında get terimi geçen herhangi bir cmdlet, alias veya fonksiyonu arayarak, PowerShell'in şu anda bildiği her şeyin ayrıntılı bir listesini elde ederiz. Aynı aramayı -verb get yerine ==get*== filtresini kullanarak da gerçekleştirebiliriz. Get-Command cmdlet'i * işaretini joker karakter olarak tanır ve get(anything) ifadesinin her bir varyantını gösterir. İsim üzerinde de arama yaparak benzer bir şey yapabiliriz.


#### Get-Command (noun)
```powershell-session
PS C:\htb> Get-Command -noun windows*  

CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           Apply-WindowsUnattend                              3.0        Dism
Function        Get-WindowsUpdateLog                               1.0.0.0    WindowsUpdate
Cmdlet          Add-WindowsCapability                              3.0        Dism
Cmdlet          Add-WindowsDriver                                  3.0        Dism
Cmdlet          Add-WindowsImage                                   3.0        Dism
Cmdlet          Add-WindowsPackage                                 3.0        Dism
Cmdlet          Clear-WindowsCorruptMountPoint                     3.0        Dism
Cmdlet          Disable-WindowsErrorReporting                      1.0        WindowsErrorR...
Cmdlet          Disable-WindowsOptionalFeature                     3.0        Dism
Cmdlet          Dismount-WindowsImage                              3.0        Dism
Cmdlet          Enable-WindowsErrorReporting                       1.0        WindowsErrorR...
Cmdlet          Enable-WindowsOptionalFeature                      3.0        Dism
```

Yukarıdaki çıktıda, ==-noun== modifier'ını kullandık, filtreyi bir adım daha ileri götürdük ve ismin windows* içeren herhangi bir bölümünü aradık, böylece sonuçlarımız oldukça spesifik oldu. İsim kısmında windows ile başlayan ve ardından başka bir şey gelen her şey bu filtreyle eşleşecektir. Bunlar Get-Command cmdlet'inin ne kadar güçlü olabileceğinin sadece birkaç göstergesiydi. Get-Help cmdlet'i ile eşleştirildiğinde, bunlar bize doğrudan PowerShell tarafından sağlanan güçlü yardım işlevleri olabilir. Bir sonraki ipucumuz PowerShell oturum Geçmişimizi inceliyor.


### History
PowerShell, çalıştırılan komutların geçmişini iki farklı şekilde tutar. Bunlardan ilki, her konsol oturumunun başında ve sonunda uygulanan ve silinen built-in oturum geçmişidir. Diğeri ise PSReadLine modülüdür. ==PSReadLine== modülü, diğer birçok özelliğin yanı sıra, host üzerindeki tüm oturumlarda kullanılan PowerShell komutlarının geçmişini izler. Varsayılan olarak, PowerShell girilen son 4096 komutu tutar, ancak bu ayar ==$MaximumHistoryCount== değişkeni değiştirilerek değiştirilebilir.


#### Get-History (Session)
```powershell-session
PS C:\htb> Get-History

 Id CommandLine
  -- -----------
   1 Get-Command
   2 clear
   3 get-command -verb set
   4 get-command set*
   5 clear
   6 get-command -verb get
   7 get-command -noun windows
   8 get-command -noun windows*
   9 get-module
  10 clear
  11 get-history
  12 clear
  13 ipconfig /all
  14 arp -a
  15 get-help
  16 get-help get-module
```

Varsayılan olarak, Get-History yalnızca bu etkin oturum sırasında çalıştırılan komutları gösterecektir. Komutların nasıl numaralandırıldığına dikkat edin; bu komutu tekrar çalıştırmak için r takma adını ve ardından numarayı kullanarak bu komutları geri çağırabiliriz. Örneğin, arp -a komutunu yeniden çalıştırmak istersek, r 14 komutunu verebiliriz ve PowerShell bu komutu çalıştıracaktır. Shell penceresini kapatırsak veya komut ve kontrol yoluyla uzak bir shell örneğinde, çalıştırdığımız oturumu veya işlemi öldürdüğümüzde, PowerShell geçmişimizin kaybolacağını unutmayın. Ancak PSReadLine ile durum böyle değildir. PSReadLine her şeyi $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine adresinde bulunan $($host.Name)_history.txt adlı bir dosyada saklar.

```powershell-session
PS C:\Users\BEN 10\Desktop> Get-History
>>

  Id CommandLine
  -- -----------
   1 Get-Location
   2 pwd
   3 Get-ChildItem
   4 ls
   5 Get-Location .\Documents\
   6 Set-Localtion .\Documents\
   7 Set-Location .\Documents\
   8 cd ..
   9 cd .\Documents\
  10 Set-Location 'C:\Users\Default User\'
  11 ls
  12 ls
  13 cd ..
  14 cd ..
  15 ls
  16 Get-Content .\vfcompat.dll
  17 clear
  18 ls
  19 cd .\Users\
  20 ls
  21 cd '.\BEN 10\'
  22 ls
  23 cd .\Desktop\
  24 ls
  25 Get
  26 clear
  27 ls
  28 Get-Content .\History
  29 clear
  30 Get-Content '.\not defteri .txt'
  31 cat '.\not defteri .txt'
  32 Get-Command
  33 clear
  34 Get-Command -verb get
  35 clear
  36 Get-History...

  r 33 yazarsak clear kodunu kullanır ve ekranı temizler.
```



### PSReadLine Geçmişini Görüntüleme

Kullanıcı düzeyinde arama.

```powershell-session
PS C:\htb> get-content C:\Users\DLarusso\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt

get-module
Get-ChildItem Env: | ft Key,Value
Get-ExecutionPolicy
clear
ssh administrator@10.172.16.110.55
powershell -nop -c "iex(New-Object Net.WebClient).DownloadString('https://download.sysinternals.com/files/PSTools.zip')"
Get-ExecutionPolicy

<SNIP>
```



```powershell-session
PS C:\Windows\system32> Get-Content $env:APPDATA\Microsoft\Windows\PowerShell\PSReadLine\ConsoleHost_history.txt
Get-Help Test-Wsman -Online
Update-Help
Get-Help  Test-Wsman
Update-Help
Get-Location
pwd
Get-ChildItem
ls
Get-Location .\Documents\
Set-Localtion .\Documents\
Set-Location .\Documents\
cd ..
cd .\Documents\
Set-Location 'C:\Users\Default User\'
ls
cd ..
ls
Get-Content .\vfcompat.dll
clear
ls
cd .\Users\
ls
cd '.\BEN 10\'
ls
cd .\Desktop\
ls
Get
clear
```

Yukarıdaki komutu çalıştırsaydık ve CLI'yı sık kullanan biri olsaydık, sıralamak için kapsamlı bir geçmiş dosyamız olurdu. Yukarıdaki çıktı, zamandan ve ekran alanından tasarruf etmek için kısaltılmıştır. PSReadline'ın yönetici perspektifinden harika bir özelliği, dizeleri içeren tüm girişleri otomatik olarak filtrelemeye çalışmasıdır:
- `password`
- `asplaintext`
- `token`
- `apikey`
- `secret`

Bu davranış, PSReadLine geçmiş dosyasındaki anahtarlar, kimlik bilgileri veya diğer hassas bilgileri içeren tüm girdileri temizlemeye yardımcı olacağından, yöneticiler olarak bizim için mükemmeldir. Yerleşik oturum geçmişi bunu yapmaz.


### Hotkeys (Kısayol Tuşları)
Bir GUI ortamından CLI'da çalışmadığımız sürece, faremiz genellikle çalışmayacaktır. Örneğin, bir pentest sırasında bir host'a bir shell indirdiğimizi varsayalım. Bu shell'den CMD veya PowerShell'e erişimimiz olacak, ancak GUI'yi kullanamayacağız. Bu yüzden sadece bir klavye kullanarak rahat olmamız gerekir. Kısayol tuşları, genellikle fare gerektiren daha karmaşık eylemleri sadece tuşlarımızla gerçekleştirmemizi sağlayabilir. Aşağıda daha kullanışlı kısayol tuşlarından bazılarının hızlı bir listesi bulunmaktadır.

#### Hotkeys
![Pasted image 20241010140725.png](/img/user/resimler/Pasted%20image%2020241010140725.png)

Bu liste PowerShell'de kullanabileceğimiz tüm işlevleri değil, kendimizi en çok kullanırken bulduklarımızı içermektedir.


### Sekme Tamamlama
PowerShell'in en iyi işlevlerinden biri komutların sekme ile tamamlanması olmalıdır. Yazdığımız komutu tamamlayabilecek seçenekler arasında gezinmek için tab ve SHIFT+tab tuşlarını kullanabiliriz.
![Pasted image 20241010140845.png](/img/user/resimler/Pasted%20image%2020241010140845.png)


### Aliases
Bahsetmemiz gereken son ipucumuz Alias'lar. PowerShell takma adı, bir cmdlet, komut veya çalıştırılabilir dosya için başka bir addır. [Get-Alias](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-alias?view=powershell-7.2) cmdlet'ini kullanarak varsayılan takma adların bir listesini görebiliriz. Çoğu yerleşik takma ad, cmdlet'in kısaltılmış sürümleridir, bu da hatırlamayı kolaylaştırır ve kullanımı hızlandırır.


#### Using Get-Alias

```powershell-session
PS C:\Windows\system32> Get-Alias

CommandType     Name                                               Version    Source
                                                                              
-----------     ----                                               -------    -----
Alias           % -> ForEach-Object
Alias           ? -> Where-Object
Alias           ac -> Add-Content
Alias           asnp -> Add-PSSnapin
Alias           cat -> Get-Content
Alias           cd -> Set-Location
Alias           CFS -> ConvertFrom-String                          3.1.0.0    Mi...
Alias           chdir -> Set-Location
Alias           clc -> Clear-Content
Alias           clear -> Clear-Host
Alias           clhy -> Clear-History
Alias           cli -> Clear-Item
Alias           clp -> Clear-ItemProperty
Alias           cls -> Clear-Host
Alias           clv -> Clear-Variable
Alias           cnsn -> Connect-PSSession
Alias           compare -> Compare-Object
Alias           copy -> Copy-Item
Alias           cp -> Copy-Item
Alias           cpi -> Copy-Item
Alias           cpp -> Copy-ItemProperty
Alias           curl -> Invoke-WebRequest
Alias           cvpa -> Convert-Path
Alias           dbp -> Disable-PSBreakpoint
Alias           del -> Remove-Item
Alias           diff -> Compare-Object
Alias           dir -> Get-ChildItem

<SNIP>
```

Takma adları gerçek cmdlet, komut veya çalıştırılabilir dosyanın adından daha kısa yapmak mükemmel bir uygulamadır. Get-Alias cmdlet'inin bile aşağıdaki klipte görüldüğü gibi varsayılan takma adı gal'dir.

PS C:\Users\BEN 10> gal

```powershell-session
CommandType     Name                                               Version    Source
-----------     ----                                               -------    ------
Alias           % -> ForEach-Object
Alias           ? -> Where-Object
Alias           ac -> Add-Content
Alias           asnp -> Add-PSSnapin
Alias           cat -> Get-Content
Alias           cd -> Set-Location
Alias           CFS -> ConvertFrom-String                          3.1.0.0    Microsoft.PowerShell.Utility
Alias           chdir -> Set-Location
Alias           clc -> Clear-Content
Alias           clear -> Clear-Host
Alias           clhy -> Clear-History
Alias           cli -> Clear-Item
Alias           clp -> Clear-ItemProperty
Alias           cls -> Clear-Host
Alias           clv -> Clear-Variable
Alias           cnsn -> Connect-PSSession
Alias           compare -> Compare-Object
Alias           copy -> Copy-Item
Alias           cp -> Copy-Item
Alias           cpi -> Copy-Item
Alias           cpp -> Copy-ItemProperty
Alias           curl -> Invoke-WebRequest
Alias           cvpa -> Convert-Path
Alias           dbp -> Disable-PSBreakpoint
Alias           del -> Remove-Item
Alias           diff -> Compare-Object
Alias           dir -> Get-ChildItem
Alias           dnsn -> Disconnect-PSSession
Alias           ebp -> Enable-PSBreakpoint
```


[Set-Alias](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/set-alias?view=powershell-7.2) kullanarak belirli bir cmdlet için bir takma ad da ayarlayabiliriz. Get-Help cmdlet'i için bir takma ad oluşturarak bu konuda pratik yapalım.


#### Using Set-Alias

```powershell-session
PS C:\Windows\system32> Set-Alias -Name gh -Value Get-Help
```
![Pasted image 20241010141209.png](/img/user/resimler/Pasted%20image%2020241010141209.png)

Set-Alias kullanırken, takma adın adını (-Name gh) ve ilgili cmdlet'i (-Value Get-Help) belirtmemiz gerekir.

Aşağıda, en yararlı olduğunu düşündüğümüz birkaç takma adın bir listesini de ekliyoruz. Bazı komutların birden fazla takma adı da vardır. Yararlı bulabileceğiniz diğer takma adlar için listenin tamamına baktığınızdan emin olun.


#### Helpful Aliases
BASH'a aşina olanlar için, takma adların çoğunun Linux dağıtımlarında yaygın olarak kullanılan komutlarla eşleştiğini fark etmiş olabilirsiniz. Bu bilgi yararlı olabilir ve öğrenme eğrisini kolaylaştırmaya yardımcı olabilir.

Bu bölüm biraz uzun oldu ve bunun iyi bir nedeni var. PowerShell ustalığına giden yolda bizi ilerletecek tüm temel konuları ele aldık. Buradan itibaren PowerShell modülleri ve cmdlet'lerine derinlemesine dalacağız.

![Pasted image 20241010141328.png](/img/user/resimler/Pasted%20image%2020241010141328.png)
BASH'a aşina olanlar için, takma adların çoğunun Linux dağıtımlarında yaygın olarak kullanılan komutlarla eşleştiğini fark etmiş olabilirsiniz. Bu bilgi yararlı olabilir ve öğrenme eğrisini kolaylaştırmaya yardımcı olabilir.


### Cmdlet'ler ve Modüller Hakkında Her Şey
Bu bölümde aşağıdakileri ele alacağız:
* Cmdlet'ler ve Modüller nedir?
* Onlarla nasıl etkileşim kurarız?
* Web'den yeni modülleri nasıl yükleriz ve yükleriz?

PowerShell'i hem sysadmin hem de pentester olarak kullanırken bu soruları anlamak çok önemlidir. PowerShells'in modüler ve genişletilebilir olma özelliği, onu kitimizde bulunması gereken güçlü bir araç haline getirir. Şimdi cmdlet'lerin ve modüllerin ne olduğunu inceleyelim.


### Cmdlets
Microsoft tarafından tanımlandığı şekliyle bir [cmdlet](https://learn.microsoft.com/en-us/powershell/scripting/lang-spec/chapter-13?view=powershell-7.2):

“PowerShell'deki nesneleri manipüle eden tek özellikli bir komut.”

Cmdlet'ler, genellikle herhangi bir cmdlet'in ne yaptığını anlamamızı kolaylaştıran bir Verb-Noun yapısını takip eder. Test-WSMan ile fiilin Test ve ismin Wsman olduğunu görebiliriz. Fiil ve isim bir tire (-) ile ayrılmıştır. Fiil ve isimden sonra, istenen eylemi gerçekleştirmek için belirli bir cmdlet ile kullanabileceğimiz seçenekleri kullanırız. Cmdlet'ler PowerShell kodunda veya diğer programlama dillerinde kullanılan fonksiyonlara benzer ancak önemli bir farkları vardır. Cmdlet'ler PowerShell'de yazılmaz. C# veya başka bir dilde yazılırlar ve daha sonra kullanım için derlenirler. Son bölümde gördüğümüz gibi, mevcut uygulamaları, cmdlet'leri ve fonksiyonları görüntülemek için ==Get-Command== cmdlet'ini ve türünü belirlememize yardımcı olabilecek “CommandType” etiketli bir özelliği kullanabiliriz.

Belirli bir cmdlet ile kullanabileceğimiz seçenekleri ve fonksiyonları görmek istiyorsak [Get-Help](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-help?view=powershell-7.2) cmdlet'inin yanı sıra ==Get-Member== cmdlet'ini de kullanabiliriz.


## PowerShell Modules

Bir [PowerShell modülü](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/understanding-a-windows-powershell-module?view=powershell-7.2), kullanımı ve paylaşımı kolay hale getirilmiş yapılandırılmış PowerShell kodudur. Resmi Microsoft dokümanlarında belirtildiği gibi, bir modül aşağıdakilerden oluşabilir:
* Cmdlets
* Script files
* Functions
* Assemblies
* İlgili kaynaklar (manifestolar ve yardım dosyaları)

Bu bölümde, PowerView projesini kullanarak bir modülü neyin oluşturduğunu ve onlarla nasıl etkileşim kurulacağını inceleyeceğiz. PowerView.ps1, [PowerShellMafia](https://github.com/PowerShellMafia/PowerSploit) tarafından oluşturulan [PowerSploit](https://github.com/PowerShellMafia/PowerSploit) adlı bir projede düzenlenen PowerShell modülleri koleksiyonunun bir parçasıdır ve sızma test uzmanlarına Windows Domain/Active Directory ortamlarını test ederken kullanabilecekleri birçok değerli araç sağlar. Bu projenin arşivlendiğini fark etsek de, dahil edilen araçların çoğu bugün hala pen-test ile ilgili ve kullanışlıdır (Ağustos 2022'de yazılmıştır). Bu modülde PowerSploit'in kullanımını ve uygulanmasını kapsamlı bir şekilde ele almayacağız. Sadece PowerShell'i daha iyi anlamak için bir referans olarak kullanacağız. Windows Domain ortamlarını Listelemek ve Saldırmak için PowerSploit kullanımı, [Active Directory Listeleme ve Saldırılar](https://academy.hackthebox.com/course/preview/active-directory-enumeration--attacks) modülünde derinlemesine ele alınmıştır.
![Pasted image 20241010152303.png](/img/user/resimler/Pasted%20image%2020241010152303.png)

### PowerSploit.psd1

PowerShell veri dosyası (.psd1) bir[ Modül manifestosu dosyasıdır](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_module_manifests?view=powershell-7.2). Bir manifesto dosyasında sıklıkla bulabiliriz:
* İşlenecek olan modüle referans
* Büyük değişiklikleri takip etmek için sürüm numaraları
* GUID
* Modülün Yazarı
* Telif Hakkı
* PowerShell uyumluluk bilgileri
* Modüller ve cmdlet'ler dahildir
* Metadata


#### PowerSploit.psd1
![Pasted image 20241010152441.png](/img/user/resimler/Pasted%20image%2020241010152441.png)

### PowerSploit.psm1
PowerShell komut dosyası modülü dosyası (==.psm1==) basitçe PowerShell kodu içeren bir komut dosyasıdır. Bunu bir modülün eti olarak düşünün.

### PowerSploit.psm1 dosyasının içeriği
```powershell-session
Get-ChildItem $PSScriptRoot | ? { $_.PSIsContainer -and !('Tests','docs' -contains $_.Name) } | % { Import-Module $_.FullName -DisableNameChecking }
```
Get-ChildItem cmdlet'i geçerli dizindeki öğeleri alır ($PSScriptRoot otomatik değişkeni ile temsil edilir) ve Where-Object cmdlet'i (“?” karakteri olarak takılır) bunları yalnızca klasör olan ve “Tests” veya “docs” adlarına sahip olmayan öğelere kadar filtreler. Son olarak, ForEach-Object cmdlet'i (“%” karakteriyle gösterilir) kalan öğelerin her birine karşı Import-Module cmdlet'ini çalıştırır ve modülün geçerli oturumdaki cmdlet'ler veya fonksiyonlarla aynı adlara sahip cmdlet'ler veya fonksiyonlar içermesi durumunda hataları önlemek için DisableNameChecking parametresini geçirir.

## Using PowerShell Modules
Hangi PowerShell modülünü kullanmak istediğimize karar verdikten sonra, onu nasıl ve nereden çalıştıracağımızı belirlememiz gerekecektir. Ayrıca, seçilen modülün ve komut dosyalarının hostta zaten olup olmadığını veya bunları host'a almamız gerekip gerekmediğini de düşünmeliyiz. Get-Module, hangi modüllerin zaten yüklü olduğunu belirlememize yardımcı olabilir.

#### Get-Module
```powershell-session
PS C:\htb> Get-Module 

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     0.0        chocolateyProfile                   {TabExpansion, Update-SessionEnvironment, refreshenv}
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Con...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}
Script     0.7.3.1    posh-git                            {Add-PoshGitToProfile, Add-SshKey, Enable-GitColors, Expan...
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...
```


### List-Available
```powershell-session
PS C:\htb> Get-Module -ListAvailable 

 Directory: C:\Users\tru7h\Documents\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     1.1.0      PSSQLite                            {Invoke-SqliteBulkCopy, Invoke-SqliteQuery, New-SqliteConn...


    Directory: C:\Program Files\WindowsPowerShell\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Script     1.0.1      Microsoft.PowerShell.Operation.V... {Get-OperationValidation, Invoke-OperationValidation}
Binary     1.0.0.1    PackageManagement                   {Find-Package, Get-Package, Get-PackageProvider, Get-Packa...
Script     3.4.0      Pester                              {Describe, Context, It, Should...}
Script     1.0.0.1    PowerShellGet                       {Install-Module, Find-Module, Save-Module, Update-Module...}
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Set-PSReadLineKeyHandler, Remov...
```

==-ListAvailable== modifier'ı bize yüklediğimiz ancak oturumumuza yüklenmemiş tüm modülleri gösterecektir.

İstediğimiz modülü veya komut dosyalarını hedef Windows hostuna zaten aktardık. Daha sonra bunları çalıştırmamız gerekecektir. ==Import-Module== cmdlet'ini kullanarak başlayabiliriz.


#### Using Import-Module
[Import-Module cmdlet'i,](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/import-module?view=powershell-7.2) geçerli PowerShell oturumuna bir modül eklememizi sağlar.

```powershell-session
PS C:\Users\htb-student> Get-Help Import-Module

NAME
    Import-Module

SYNOPSIS
    Adds modules to the current session.


SYNTAX
    Import-Module [-Assembly] <System.Reflection.Assembly[]> [-Alias <System.String[]>] [-ArgumentList
    <System.Object[]>] [-AsCustomObject] [-Cmdlet <System.String[]>] [-DisableNameChecking] [-Force] [-Function
    <System.String[]>] [-Global] [-NoClobber] [-PassThru] [-Prefix <System.String>] [-Scope {Local | Global}]
    [-Variable <System.String[]>] [<CommonParameters>]

    Import-Module [-Name] <System.String[]> [-Alias <System.String[]>] [-ArgumentList <System.Object[]>]
    [-AsCustomObject] [-CimNamespace <System.String>] [-CimResourceUri <System.Uri>] -CimSession
    <Microsoft.Management.Infrastructure.CimSession> [-Cmdlet <System.String[]>] [-DisableNameChecking] [-Force]
    [-Function <System.String[]>] [-Global] [-MaximumVersion <System.String>] [-MinimumVersion <System.Version>]
    [-NoClobber] [-PassThru] [-Prefix <System.String>] [-RequiredVersion <System.Version>] [-Scope {Local | Global}]
    [-Variable <System.String[]>] [<CommonParameters>]

<SNIP>
```

Modülü mevcut PowerShell oturumumuza import etme fikrini anlamak için PowerSploit'in bir parçası olan bir cmdlet'i (Get-NetLocalgroup) çalıştırmayı deneyebiliriz. Bir modülü import etmeden bunu yapmaya çalıştığımızda bir hata mesajı alacağız. PowerSploit modülünü başarıyla import ettiğimizde (bizim kullanımımız için hedef hostun Masaüstüne yerleştirilmiştir), Get-NetLocalgroup da dahil olmak üzere birçok cmdlet kullanılabilir olacaktır. Aşağıdaki klipte bunu çalışırken görün:

#### Importing PowerSploit.psd1
```powershell-session
PS C:\Users\htb-student\Desktop\PowerSploit> Import-Module .\PowerSploit.psd1
PS C:\Users\htb-student\Desktop\PowerSploit> Get-NetLocalgroup

ComputerName GroupName                           Comment
------------ ---------                           -------
WS01         Access Control Assistance Operators Members of this group can remotely query authorization attributes a...
WS01         Administrators                      Administrators have complete and unrestricted access to the compute...
WS01         Backup Operators                    Backup Operators can override security restrictions for the sole pu...
WS01         Cryptographic Operators             Members are authorized to perform cryptographic operations.
WS01         Distributed COM Users               Members are allowed to launch, activate and use Distributed COM obj...
WS01         Event Log Readers                   Members of this group can read event logs from local machine
WS01         Guests                              Guests have the same access as members of the Users group by defaul...
WS01         Hyper-V Administrators              Members of this group have complete and unrestricted access to all ...
WS01         IIS_IUSRS                           Built-in group used by Internet Information Services.
WS01         Network Configuration Operators     Members in this group can have some administrative privileges to ma...
WS01         Performance Log Users               Members of this group may schedule logging of performance counters,...
WS01         Performance Monitor Users           Members of this group can access performance counter data locally a...
WS01         Power Users                         Power Users are included for backwards compatibility and possess li...
WS01         Remote Desktop Users                Members in this group are granted the right to logon remotely
WS01         Remote Management Users             Members of this group can access WMI resources over management prot...
WS01         Replicator                          Supports file replication in a domain
WS01         System Managed Accounts Group       Members of this group are managed by the system.
WS01         Users                               Users are prevented from making accidental or intentional system-wi...
```

![Pasted image 20241010153559.png](/img/user/resimler/Pasted%20image%2020241010153559.png)
Klibin başında Get-NetLocalgroup'un nasıl tanınmadığına dikkat edin. Bu olay, varsayılan modül yolunda yer almadığı için gerçekleşmiştir. Varsayılan modül yolunun nerede olduğunu PSModulePath ortam değişkenini listeleyerek görebiliriz.

#### Viewing PSModulePath
```powershell-session
PS C:\Users\htb-student> $env:PSModulePath

C:\Users\htb-student\Documents\WindowsPowerShell\Modules;C:\Program Files\WindowsPowerShell\Modules;C:\Windows\system32\WindowsPowerShell\v1.0\Modules
```
PowerSploit.psd1 modülü import edildiğinde, Get-NetLocalgroup fonksiyonu tanınır. Bunun nedeni, PowerSploit.psd1 yüklendiğinde birkaç modülün dahil edilmesidir. Dosyaları PSModulePath'te başvurulan dizinlere ekleyerek bir modülü veya birkaç modülü kalıcı olarak eklemek mümkündür. Bu eylem, birincil saldırı hostumuz olarak bir Windows işletim sistemi kullanıyorsak mantıklıdır, ancak bir etkileşimde, zamanımız sadece belirli komut dosyalarını saldırı hostuna aktarmak ve gerektiğinde içe aktarmak için daha iyi olacaktır.


## Execution Policy
PowerShell komut dosyalarını ve modüllerini kullanmaya çalışırken göz önünde bulundurulması gereken önemli bir faktör [PowerShell'in execution policy](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_execution_policies?view=powershell-7.2)'sidir. Microsoft'un resmi belgelerinde belirtildiği gibi, execution policy bir güvenlik denetimi değildir. BT yöneticilerine kendileri için parametreler ve güvenlik önlemleri belirleyebilecekleri bir araç sunmak üzere tasarlanmıştır.


### Execution Policy's Etkisi
```powershell-session
PS C:\Users\htb-student\Desktop\PowerSploit> Import-Module .\PowerSploit.psd1

Import-Module : File C:\Users\Users\htb-student\PowerSploit.psm1
cannot be loaded because running scripts is disabled on this system. For more information, see
about_Execution_Policies at https:/go.microsoft.com/fwlink/?LinkID=135170.
At line:1 char:1
+ Import-Module .\PowerSploit.psd1
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : SecurityError: (:) [Import-Module], PSSecurityException
    + FullyQualifiedErrorId : UnauthorizedAccess,Microsoft.PowerShell.Commands.ImportModuleCommand
```

Host'un execution policy'si script'imizi çalıştıramamamıza neden oluyor. Ancak bunu aşabiliriz. İlk olarak, execution policy ayarlarımızı kontrol edelim.


### Execution Policy Durumunu Kontrol Etme
```powershell-session
PS C:\htb> Get-ExecutionPolicy 

Restricted  
```

Mevcut ayarımız kullanıcının yapabileceklerini kısıtlar. Ayarı değiştirmek istiyorsak, bunu Set-ExecutionPolicy cmdlet'i ile yapabiliriz.

#### Setting Execution Policy
```powershell-session
PS C:\htb> Set-ExecutionPolicy undefined 
```
Politikayı tanımsız olarak ayarlayarak, PowerShell'e etkileşimlerimizi sınırlamak istemediğimizi söylüyoruz. Şimdi script'imizi import edebilmeli ve çalıştırabilmeliyiz.


#### Testing It Out
```powershell-session
PS C:\htb> Import-Module .\PowerSploit.psd1

Import-Module .\PowerSploit.psd1
PS C:\Users\htb> get-module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Check...
Manifest   3.0.0.0    Microsoft.PowerShell.Security       {ConvertFrom-SecureString, Conver...
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Vari...
Script     3.0.0.0    PowerSploit                         {Add-Persistence, Add-ServiceDacl...
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PS...
```
Yüklü modüllerimize baktığımızda, PowerSploit'i başarıyla yüklediğimizi görebiliriz. Artık araçları gerektiği gibi kullanabiliriz.

Not: Bir sysadmin olarak, bu tür değişiklikler yaygındır ve işimiz bittiğinde her zaman geri alınmalıdır. Bir pentester olarak, böyle bir değişiklik yapmamız ve bunu geri almamamız, bir savunmacıya hostun ele geçirildiğini gösterebilir. Eylemlerimizden sonra temizlik yaptığımızı kontrol ettiğinizden emin olun. Execution policy'yi atlamanın ve kalıcı bir değişiklik bırakmamanın bir başka yolu da -scope kullanarak proses seviyesinde değişiklik yapmaktır.



#### Change Execution Policy By Scope
```powershell-session
PS C:\htb> Set-ExecutionPolicy -scope Process 
PS C:\htb> Get-ExecutionPolicy -list

Scope ExecutionPolicy
        ----- ---------------
MachinePolicy       Undefined
   UserPolicy       Undefined
      Process          Bypass
  CurrentUser       Undefined
 LocalMachine          Bypass  
```
Process seviyesinde değiştirdiğimizde, PowerShell oturumunu kapattığımızda değişikliğimiz geri dönecektir. Komut dosyaları ve yeni modüllerle çalışırken execution policy'yi aklınızda bulundurun. Elbette, kullanım için güvenli olduklarından emin olmak için önce yüklemeye çalıştığımız komut dosyalarına bakmak isteriz. Sızma testi uzmanları olarak, bir hostta Execution Policy'yi nasıl bypass edeceğimiz konusunda yaratıcı olmamız gereken zamanlarla karşılaşabiliriz. Bu [blog](https://www.netspi.com/blog/technical/network-penetration-testing/15-ways-to-bypass-the-powershell-execution-policy/) yazısında, gerçek dünyadaki görevlerde büyük başarı ile kullandığımız bazı yaratıcı yöntemler yer almaktadır.


### Cmdlet'leri ve Fonksiyonları Modül İçinden Çağırma
İmport edilen bir modülün oturuma hangi takma adları, cmdlet'leri ve fonksiyonları getirdiğini görmek istersek, bizi aydınlatması için ==Get-Command -Module modulename== komutunu kullanabiliriz.


#### Using Get-Command
![Pasted image 20241010154536.png](/img/user/resimler/Pasted%20image%2020241010154536.png)
Şimdi PowerSploit tarafından nelerin yüklendiğini görebiliriz. Bu noktadan sonra, komut dosyalarını ve fonksiyonları gerektiği gibi kullanabiliriz. Bu işin kolay kısmı, fonksiyonu seçin ve çalışmasına izin verin.


### Derin Dalış: PowerShell Gallery ve GitHub'dan Modül Bulma ve Yükleme
Günümüzde bilgi paylaşmak son derece kolay. Bu, çözümler ve yeni yaratımlar için de geçerli. PowerShell modülleri söz konusu olduğunda, [PowerShell Galerisi](https://www.powershellgallery.com/) bunun için en iyi yerdir. Microsoft ve diğer kullanıcılar tarafından oluşturulan PowerShell komut dosyalarını, modüllerini ve daha fazlasını içeren bir depodur. Bunlar, kullanıcı öznitelikleriyle ilgilenmek kadar basit bir şeyden karmaşık bulut depolama sorunlarını çözmeye kadar değişebilir.


#### PowerShell Gallery
![Pasted image 20241010154845.png](/img/user/resimler/Pasted%20image%2020241010154845.png)
Bizim için uygun bir şekilde, PowerShell'de PowerShellGet adlı PowerShell Galerisi ile etkileşime girmemize yardımcı olacak yerleşik bir modül zaten var. Şimdi ona bir göz atalım:


#### PowerShellGet
![Pasted image 20241010154906.png](/img/user/resimler/Pasted%20image%2020241010154906.png)
Bu modül, galerideki mevcut modüllerle çalışmamıza ve bunları indirmemize ve kendi modüllerimizi oluşturmamıza ve yüklememize yardımcı olacak birçok farklı işleve sahiptir. Fonksiyon listemizden Find-Module'ü bir deneyelim. Sistem yöneticileri için son derece yararlı olacak bir modül de [AdminToolbox](https://www.powershellgallery.com/packages/AdminToolbox/11.0.8) modülüdür. Active Directory yönetimi, Microsoft Exchange, sanallaştırma ve bir yöneticinin herhangi bir günde ihtiyaç duyacağı diğer birçok görev için araçlar içeren diğer birkaç modülün bir koleksiyonudur.

#### Find-Module
```powershell-session
PS C:\htb> Find-Module -Name AdminToolbox 

Version    Name                                Repository           Description
-------    ----                                ----------           -----------
11.0.8     AdminToolbox                        PSGallery            Master module for a col...
```

Diğer birçok PowerShell cmdlet'inde olduğu gibi, joker karakterleri kullanarak da arama yapabiliriz. Kullanmak istediğimiz bir modül bulduğumuzda, onu yüklemek Install-Module kadar kolaydır. Modülleri bu şekilde yüklemek için yönetici hakları gerektiğini unutmayın.

#### Install-Module

![Pasted image 20241010155015.png](/img/user/resimler/Pasted%20image%2020241010155015.png)
Yukarıdaki resimde, her iki eylemi de aynı anda gerçekleştirmek için Find-Module ile Install-Module'u zincirledik. Bu örnek PowerShell'in Pipeline işlevinden faydalanmaktadır. Bunu başka bir bölümde daha derinlemesine ele alacağız, ancak şimdilik modülü tek bir komut dizesiyle bulup yüklememizi sağladı. Modern PowerShell örneklerinin, içinden bir cmdlet veya fonksiyon çalıştırdığımızda ilk kez yüklenen bir modülü otomatik olarak import edeceğini unutmayın, bu nedenle modülü yükledikten sonra import etmeye gerek yoktur. Bu, özel modüllerden veya host'a getirdiğimiz modüllerden (örneğin GitHub'dan) farklıdır. PowerShell Profilimizi değiştirmediğimiz sürece her kullanmak istediğimizde manuel olarak import etmemiz gerekecektir. Her bir PowerShell profili için konumları [burada bulabiliriz](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_profiles?view=powershell-7.2). Kendi modüllerimizi ve komut dosyalarımızı oluşturmanın veya PowerShell Galerisi'nden içe aktarmanın yanı sıra, Github'dan ve BT topluluğunun harici olarak ortaya çıkardığı tüm harika içeriklerden de yararlanabiliriz. Şimdilik Git ve Github'ı kullanmak, başka uygulamaların yüklenmesini ve henüz ele almadığımız diğer kavramlar hakkında bilgi sahibi olmayı gerektiriyor, bu nedenle bunu modülün ilerleyen bölümlerine saklayacağız.


### Farkında Olmanız Gereken Araçlar
Aşağıda, sızma test uzmanları ve sistem yöneticileri olarak farkında olmamız gereken birkaç PowerShell modülünü ve projesini hızlıca listeleyeceğiz. Bu araçların her biri PowerShell içinde kullanılmak üzere yeni bir yetenek getiriyor. Elbette, listemizden çok daha fazlası var; bunlar sadece kendimizi her görevde geri dönerken bulduğumuz birkaçı.

[AdminToolbox](https://www.powershellgallery.com/packages/AdminToolbox/11.0.8): AdminToolbox, sistem yöneticilerinin Active Directory, Exchange, Ağ yönetimi, dosya ve depolama sorunları ve daha fazlası gibi şeylerle ilgili herhangi bir sayıda eylem gerçekleştirmesine olanak tanıyan yararlı modüllerden oluşan bir koleksiyondur.

[ActiveDirectory](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps): Bu modül, Active Directory ile ilgili her şey için lokal ve remote yönetim araçlarının bir koleksiyonudur. Bununla kullanıcıları, grupları, izinleri ve çok daha fazlasını yönetebiliriz.

[Empire / Durumsal Farkındalık](https://github.com/BC-SECURITY/Empire/tree/main/empire/server/data/module_source/situational_awareness): PowerShell modülleri ve komut dosyalarından oluşan bir koleksiyondur ve bize bir host ve içinde bulundukları domain hakkında durumsal farkındalık sağlayabilir. Bu proje BC Security tarafından Empire Framework'ün bir parçası olarak sürdürülmektedir.

[Inveigh](https://github.com/Kevin-Robertson/Inveigh): Inveigh, ağ sahtekarlığı ve ortadaki adam saldırıları gerçekleştirmek için oluşturulmuş bir araçtır.

[BloodHound / SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors): Bloodhound/Sharphound, C# ve PowerShell ile yazılmış grafik analiz araçları ve veri toplayıcıları kullanarak bir Active Directory Ortamını görsel olarak haritalandırmamızı sağlar.

PowerShell modülleri ve cmdlet'leri ile çalışmak sezgiseldir ve hızlı bir şekilde ustalaşmak kolaydır. PowerShell'de modülleri ve cmdlet'leri yüklememizi, içe aktarmamızı veya incelememizi gerektirebilecek çeşitli araçlar ve konularla ilgileneceğimiz için bu beceri bu modülün geri kalanında kullanışlı olacaktır. Eğer takılırsanız, bu bölüme geri döndüğünüzden emin olun. Şimdi User and Group yönetimine geçme zamanı.


Soru : Oturumumuza yüklenen modülleri bulmamıza hangi cmdlet yardımcı olabilir?
cevap : Get-Module

Soru : Hangi modül bize PowerShell Galerisi'nden paket kurulumunu yönetmek için oluşturulmuş cmdlet'leri sağlar?
Cevap : PowerShellGet



# User and Group Management
Bir system administrator olarak, user ve Group yönetimi önemli bir beceridir çünkü kullanıcılarımız genellikle yönetmemiz gereken ana varlıklarımızdır ve genellikle bir kuruluşun en büyük saldırı vektörüdür. Pentesterlar olarak, kullanıcıların ve grupların nasıl numaralandırılacağını, yorumlanacağını ve bunlardan nasıl yararlanılacağını anlamak, bir pentest çalışması sırasında erişim elde etmenin ve ayrıcalıklarımızı yükseltmenin en kolay yollarından biridir. Bu bölümde kullanıcıların ve grupların ne olduğu, PowerShell ile nasıl yönetileceği ve kısaca Active Directory domainleri ve domain kullanıcıları kavramı ele alınacaktır.


### User Accounts nedir?
User hesapları, personelin bir hostun kaynaklarına erişmesi ve bunları kullanması için bir yoldur. Belirli durumlarda sistem, eylemleri gerçekleştirmek için özel olarak sağlanmış bir kullanıcı hesabını da kullanacaktır. Hesaplar hakkında düşünürken, genellikle dört farklı türle karşılaşırız:
- Service Accounts
- Built-in accounts
- Local users
- Domain users

### Default Local User Accounts
İşletim sistemi yüklendiğinde, host yönetimi ve temel kullanıma yardımcı olmak için her Windows örneğinde çeşitli hesaplar oluşturulur. Aşağıda standart built-in hesapların bir listesi bulunmaktadır.

#### Built-In Accounts

==Administrator==: Bu hesap lokal host üzerinde yönetimsel görevleri yerine getirmek için kullanılır.

==Default Account (Varsayılan Hesap)== : Varsayılan hesap, sistem tarafından Xbox yardımcı programı gibi çok kullanıcılı kimlik doğrulama uygulamalarını çalıştırmak için kullanılır.

==Guest Account :== Bu hesap, normal kullanıcı hesabı olmayan kullanıcıların host'a erişmesine izin veren sınırlı haklara sahip bir hesaptır. Varsayılan olarak devre dışıdır ve bu şekilde kalmalıdır.

==WDAGUtility Account	:== Bu hesap, uygulama oturumlarını korumalı alana alabilen Defender Application Guard için kullanılır.



### Active Directory'ye Kısa Giriş
Özetle, Active Directory (AD), kullanıcılar, bilgisayarlar, gruplar, ağ cihazları, dosya paylaşımları, grup politikaları, cihazlar ve diğer kuruluşlarla olan güven ilişkileri için merkezi bir yönetim noktası sağlayan Windows ortamlarına yönelik bir dizin hizmetidir. Bunu bir kurumsal ortam için kapı bekçisi olarak düşünün. Domain'in bir parçası olan herkes kaynaklara serbestçe erişebilirken, olmayan herkesin aynı kaynaklara erişimi reddedilir veya en azından ziyaretçi merkezinde beklemek zorunda kalır.

Bu bölümde, kullanıcılar ve gruplar bağlamında AD ile ilgileniyoruz. ActiveDirectory Modülünü kullanarak domain'e katılmış herhangi bir host üzerinde PowerShell'den bunları yönetebiliriz. Active Directory'ye derinlemesine bir dalış yapmak birden fazla bölüm alacaktır, bu nedenle burada denemeyeceğiz. 

### Local vs. Domain Joined Users
Aralarındaki fark nedir?

Domain kullanıcıları, kullanıcı ve grup üyeliğine bağlı olarak dosya sunucuları, yazıcılar, intranet host'ları ve diğer nesneler gibi kaynaklara erişmek için domain'den haklara sahip olmaları bakımından local kullanıcılardan farklıdır. Domain kullanıcı hesapları domain'deki herhangi bir host'ta oturum açabilirken, local kullanıcı sadece oluşturulduğu host'a erişim iznine sahiptir.

Bireysel bir Windows sisteminde ve bir domain ağında çeşitli hesapların birlikte nasıl çalıştığını daha iyi anlamak için [hesaplarla](https://learn.microsoft.com/en-us/windows/security/identity-protection/access-control/local-accounts) ilgili belgelere bakmaya değer. Bunları incelemek ve aralarındaki farkları anlamak için biraz zaman ayırın. Kullanımlarını ve her bir hesap türünün faydasını anlamak, bir sızma testi sırasında ayrıcalıklı erişim veya lateral hareket girişiminde bulunan bir pentesters'ı yapabilir veya bozabilir.


### User Groups Nedir?
Gruplar, kullanıcı hesaplarını mantıksal olarak sıralamanın ve bunu yaparken her bir kullanıcıyı manuel olarak yönetmek zorunda kalmadan kaynaklara ayrıntılı izinler ve erişim sağlamanın bir yoludur. Örneğin, belirli bir dizine veya paylaşıma erişimi kısıtlayabiliriz, böylece yalnızca erişime ihtiyacı olan kullanıcılar dosyaları görüntüleyebilir. Tekil bir hostta bu bizim için pek bir şey ifade etmez. Ancak mantıksal gruplama, binlerce olmasa da yüzlerce kullanıcıdan oluşan bir domain içinde uygun bir güvenlik duruşu sağlamak için gereklidir. Domain perspektifinden bakıldığında, yalnızca kullanıcıları değil, PC'ler, yazıcılar ve hatta diğer gruplar gibi uç cihazları da tutabilen birkaç farklı grup türüne sahibiz. Bu kavram bu modül için çok derin bir konudur. Ancak şimdilik grupların nasıl yönetileceği hakkında konuşacağız. 


#### Get-LocalGroup
```powershell-session
PS C:\htb> get-localgroup

Name                                Description
----                                -----------
__vmware__                          VMware User Group
Access Control Assistance Operators Members of this group can remotely query authorization attributes and permission...
Administrators                      Administrators have complete and unrestricted access to the computer/domain
Backup Operators                    Backup Operators can override security restrictions for the sole purpose of back...
Cryptographic Operators             Members are authorized to perform cryptographic operations.
Device Owners                       Members of this group can change system-wide settings.
Distributed COM Users               Members are allowed to launch, activate and use Distributed COM objects on this ...
Event Log Readers                   Members of this group can read event logs from local machine
Guests                              Guests have the same access as members of the Users group by default, except for...
Hyper-V Administrators              Members of this group have complete and unrestricted access to all features of H...
IIS_IUSRS                           Built-in group used by Internet Information Services.
Network Configuration Operators     Members in this group can have some administrative privileges to manage configur...
Performance Log Users               Members of this group may schedule logging of performance counters, enable trace...
Performance Monitor Users           Members of this group can access performance counter data locally and remotely
Power Users                         Power Users are included for backwards compatibility and possess limited adminis...
Remote Desktop Users                Members in this group are granted the right to logon remotely
Remote Management Users             Members of this group can access WMI resources over management protocols (such a...
Replicator                          Supports file replication in a domain
System Managed Accounts Group       Members of this group are managed by the system.
Users                               Users are prevented from making accidental or intentional system-wide changes an...  
```

Yukarıda, bağımsız bir host için lokal gruplara bir örnek verilmiştir. Administrators ve guest hesapları gibi basit şeyler için grupların yanı sıra sanallaştırma uygulamaları için administrators, remote users, vb. gibi belirli roller için gruplar olduğunu görebiliriz. Şimdi onları anladığımıza göre kullanıcılar ve gruplarla etkileşime geçelim.


### Adding/Removing/Editing User Accounts & Groups
PowerShell'deki diğer çoğu şey gibi, user ve grupları bulmak, oluşturmak ve değiştirmek için ==get==, ==new== ve ==set== fiillerini kullanırız. Lokal user ve gruplarla uğraşıyorsanız, ==localuser & localgroup== bunu başarabilir. Domain varlıkları için ==aduser & adgroup== işimizi görür. Emin değilsek, nelere erişimimiz olduğunu görmek için her zaman ==Get-Command user== cmdlet'ini kullanabiliriz. Bunlardan birkaçını deneyelim.


### Lokal Kullanıcıların Belirlenmesi
```powershell-session
PS C:\htb> Get-LocalUser  
  
Name               Enabled Description
----               ------- -----------
Administrator      False   Built-in account for administering the computer/domain
DefaultAccount     False   A user account managed by the system.
DLarusso           True    High kick specialist.
Guest              False   Built-in account for guest access to the computer/domain
sshd               True
WDAGUtilityAccount False   A user account managed and used by the system for Windows Defender A...
```

Get-LocalUser hostumuzdaki kullanıcıları gösterecektir. Bu kullanıcıların yalnızca bu host'a erişimi vardır. Diyelim ki JLawrence adında yeni bir lokal kullanıcı oluşturmak istiyoruz. Bu görevi New-LocalUser kullanarak gerçekleştirebiliriz. Doğru sözdiziminden emin değilsek, lütfen Get-Help komutunu kullanmayı unutmayın. Yeni bir lokal kullanıcı oluştururken, syntax açısından tek gerçek gereksinim bir isim girmek ve bir parola belirtmektir (veya -NoPassword). Açıklama veya hesabın sona ermesi gibi diğer tüm ayarlar isteğe bağlıdır.

#### Creating A New User
```powershell-session
PS C:\htb>  New-LocalUser -Name "JLawrence" -NoPassword

Name      Enabled Description
----      ------- -----------
JLawrence True
```
Yukarıda, JLawrence kullanıcısını oluşturduk ve bir parola belirlemedik. Yani bu hesap aktif ve parola olmadan oturum açılabilir. Kullandığımız Windows sürümüne bağlı olarak, bir Parola belirlemeyerek Windows'a bunun bir Microsoft canlı hesabı olduğunu belirtiriz ve local bir parola kullanmak yerine bu şekilde oturum açmaya çalışır.

Eğer bir kullanıcıyı değiştirmek istersek, Set-LocalUser cmdlet'ini kullanabiliriz. Bu örnek için, JLawrence'ı değiştireceğiz ve hesabında bir parola ve açıklama belirleyeceğiz.

#### Modifying a User
```powershell-session
PS C:\htb> $Password = Read-Host -AsSecureString
****************
PS C:\htb> Set-LocalUser -Name "JLawrence" -Password $Password -Description "CEO EagleFang"
PS C:\htb> Get-LocalUser  

Name               Enabled Description
----               ------- -----------
Administrator      False   Built-in account for administering the computer/domain
DefaultAccount     False   A user account managed by the system.
demo               True
Guest              False   Built-in account for guest access to the computer/domain
JLawrence          True    CEO EagleFang
```
Kullanıcı oluşturmak ve değiştirmek ise yukarıda gördüğümüz kadar basittir. Şimdi, grupları kontrol etmeye geçelim. Eğer biraz eko gibi geldiyse...evet, öyle. Komutların kullanımı benzerdir.

#### Get-LocalGroup
```powershell-session
PS C:\htb> Get-LocalGroup  

Name                                Description
----                                -----------
Access Control Assistance Operators Members of this group can remotely query authorization attr...
Administrators                      Administrators have complete and unrestricted access to the...
Backup Operators                    Backup Operators can override security restrictions for the...
Cryptographic Operators             Members are authorized to perform cryptographic operations.
Device Owners                       Members of this group can change system-wide settings.
Distributed COM Users               Members are allowed to launch, activate and use Distributed...
Event Log Readers                   Members of this group can read event logs from local machine
Guests                              Guests have the same access as members of the Users group b...
Hyper-V Administrators              Members of this group have complete and unrestricted access...
IIS_IUSRS                           Built-in group used by Internet Information Services.
Network Configuration Operators     Members in this group can have some administrative privileg...
Performance Log Users               Members of this group may schedule logging of performance c...
Performance Monitor Users           Members of this group can access performance counter data l...
Power Users                         Power Users are included for backwards compatibility and po...
Remote Desktop Users                Members in this group are granted the right to logon remotely
Remote Management Users             Members of this group can access WMI resources over managem...
Replicator                          Supports file replication in a domain
System Managed Accounts Group       Members of this group are managed by the system.
Users                               Users are prevented from making accidental or intentional s...

PS C:\Windows\system32> Get-LocalGroupMember -Name "Users"

ObjectClass Name                             PrincipalSource
----------- ----                             ---------------
User        DESKTOP-B3MFM77\demo             Local
User        DESKTOP-B3MFM77\JLawrence        Local
Group       NT AUTHORITY\Authenticated Users Unknown
Group       NT AUTHORITY\INTERACTIVE         Unknown
```

Yukarıdaki çıktıda, host üzerindeki her grubun bir çıktısını almak için Get-LocalGroup cmdlet'ini çalıştırdık. İkinci komutta, Users grubunu incelemeye ve söz konusu gruba kimlerin üye olduğunu görmeye karar verdik. Bunu Get-LocalGroupMember komutu ile yaptık. Şimdi, bir gruba başka bir grup veya kullanıcı eklemek istersek, Add-LocalGroupMember komutunu kullanabiliriz. Aşağıdaki örnekte JLawrence'ı Remote Desktop Users grubuna ekleyeceğiz.


#### Adding a Member To a Group
```powershell-session
PS C:\htb> Add-LocalGroupMember -Group "Remote Desktop Users" -Member "JLawrence"
PS C:\htb> Get-LocalGroupMember -Name "Remote Desktop Users" 

ObjectClass Name                      PrincipalSource
----------- ----                      ---------------
User        DESKTOP-B3MFM77\JLawrence Local
```
Komutu çalıştırdıktan sonra grup üyeliğini kontrol ettik ve kullanıcımızın gerçekten de Remote Desktop Users grubuna eklendiğini gördük. Lokal kullanıcı ve grupların bakımı basittir ve harici modüller gerektirmez. Active Directory Kullanıcılarını ve Gruplarını yönetmek biraz daha fazla çalışma gerektirir.




### Domain Kullanıcılarını ve Gruplarını Yönetme
İhtiyacımız olan cmdlet'lere erişebilmemiz ve Active Directory ile çalışabilmemiz için ActiveDirectory PowerShell Modülünü yüklememiz gerekir. AdminToolbox'ı yüklediyseniz, AD modülü zaten hostunuzda olabilir. Değilse, AD modüllerini hızlıca alabilir ve çalışmaya başlayabiliriz. Bir gereklilik, isteğe bağlı ==Remote System Administration Tools== özelliğinin yüklü olmasıdır. Bu özellik, resmi ActiveDirectory PowerShell modülünü edinmenin tek yoludur. AdminToolbox ve diğer Modüllerdeki sürüm yeniden paketlenmiştir, bu nedenle dikkatli olun.


#### Installing RSAT
```powershell-session
PS C:\htb> Get-WindowsCapability -Name RSAT* -Online | Add-WindowsCapability -Online

Path          :  
Online        : True  
RestartNeeded : False  
```
Yukarıdaki komut Microsoft Catalog'daki TÜM RSAT özelliklerini yükleyecektir. Hafif kalmak istiyorsak, Rsat.ActiveDirectory.DS-LDS.Tools ~ ~ ~ ~0.0.1.0 adlı paketi yükleyebiliriz. Şimdi ActiveDirectory modülünü yüklemiş olmalıyız. Kontrol edelim.


### AD Modülünün Yerinin Belirlenmesi
```powershell-session
PS C:\htb> Get-Module -Name ActiveDirectory -ListAvailable 

    Directory: C:\Windows\system32\WindowsPowerShell\v1.0\Modules


ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAccount, Add-ADDomainControllerPasswordReplicationPolicy, Add-A...
```

Güzel. Artık modülümüz olduğuna göre, AD Kullanıcı ve Grup yönetimine başlayabiliriz. Belirli bir kullanıcıyı bulmanın en kolay yolu Get-ADUser cmdlet'i ile arama yapmaktır.


#### Get-ADUser
```powershell-session
PS C:\htb> Get-ADUser -Filter *

DistinguishedName : CN=user14,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         :
Name              : user14
ObjectClass       : user
ObjectGUID        : bef9787d-2716-4dc9-8e8f-f8037a72c3d9
SamAccountName    : user14
SID               : S-1-5-21-1480833693-1324064541-2711030367-1110
Surname           :
UserPrincipalName :

DistinguishedName : CN=sshd,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         :
Name              : sshd
ObjectClass       : user
ObjectGUID        : 7a324e98-00e4-480b-8a1a-fa465d558063
SamAccountName    : sshd
SID               : S-1-5-21-1480833693-1324064541-2711030367-1112
Surname           :
UserPrincipalName :

DistinguishedName : CN=TSilver,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         :
Name              : TSilver
ObjectClass       : user
ObjectGUID        : a19a6c8a-000a-4cbf-aa14-0c7fca643c37
SamAccountName    : TSilver
SID               : S-1-5-21-1480833693-1324064541-2711030367-1602
Surname           :
UserPrincipalName :  

<SNIP>
```

-Filtre * parametresi Active Directory içindeki tüm kullanıcıları almamızı sağlar. Kuruluşumuzun büyüklüğüne bağlı olarak, bu bir ton çıktı üretebilir. Bir kullanıcı için ayırt edici ad, GUID, objectSid veya SamAccountName ile daha spesifik bir arama yapmak için -Identity parametresini kullanabiliriz. Bu seçenekler size anlamsız geliyorsa endişelenmeyin; sorun değil. Bunların ayrıntıları şu anda önemli değil; konuyla ilgili daha fazla bilgi için bu [makaleye](https://learn.microsoft.com/en-us/windows/win32/ad/naming-properties) veya Active Directory'ye Giriş modülüne göz atın. Şimdi TSilver kullanıcısını arayacağız.


### Belirli Bir Kullanıcıyı Al
```powershell-session
PS C:\htb>  Get-ADUser -Identity TSilver


DistinguishedName : CN=TSilver,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         :
Name              : TSilver
ObjectClass       : user
ObjectGUID        : a19a6c8a-000a-4cbf-aa14-0c7fca643c37
SamAccountName    : TSilver
SID               : S-1-5-21-1480833693-1324064541-2711030367-1602
Surname           :
UserPrincipalName :
```

Çıktıdan kullanıcı hakkında aşağıdakiler de dahil olmak üzere çeşitli bilgiler görebiliriz:

* ==Object Class (Nesne Sınıfı):== nesnenin kullanıcı mı, bilgisayar mı yoksa başka bir nesne türü mü olduğunu belirtir.
* ==DistinguishedName==: Nesnenin AD şeması içindeki göreli yolunu belirtir.
* ==Enabled (Etkin)==: Kullanıcının etkin olup olmadığını ve oturum açıp açamayacağını söyler.
* ==SamAccountName==: ActiveDirectory host'larında oturum açmak için kullanılan kullanıcı adının temsilidir.
* ==ObjectGUID==: Kullanıcı nesnesinin benzersiz tanımlayıcısıdır.

Kullanıcıların birçok farklı özniteliği vardır (hepsi burada gösterilmemiştir) ve hepsi onları tanımlamak ve gruplamak için kullanılabilir. Bunları belirli öznitelikleri filtrelemek için de kullanabiliriz. Örneğin, kullanıcının E-posta adresini filtreleyelim.


#### Searching On An Attribute
```powershell-session
PS C:\htb> Get-ADUser -Filter {EmailAddress -like '*greenhorn.corp'}


DistinguishedName : CN=TSilver,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         :
Name              : TSilver
ObjectClass       : user
ObjectGUID        : a19a6c8a-000a-4cbf-aa14-0c7fca643c37
SamAccountName    : TSilver
SID               : S-1-5-21-1480833693-1324064541-2711030367-1602
Surname           :
UserPrincipalName :
```

Çıktımızda, adlandırma bağlamımız greenhorn.corp ile eşleşen bir e-posta adresine sahip bir kullanıcı için yalnızca bir sonucumuz olduğunu görebiliriz. Bu, filtreleyebileceğimiz özniteliklere yalnızca bir örnektir. Daha ayrıntılı bir liste için, varsayılan ve genişletilmiş kullanıcı nesnesi özelliklerini kapsayan bu [Technet Makalesine](https://social.technet.microsoft.com/wiki/contents/articles/12037.active-directory-get-aduser-default-and-extended-properties.aspx) göz atın.

Greenhorn'a yeni katılan Mori Tanaka adlı bir çalışan için yeni bir kullanıcı oluşturmamız gerekiyor. New-ADUser cmdlet'ini bir deneyelim.

#### New ADUser
```powershell-session
PS C:\htb> New-ADUser -Name "MTanaka" -Surname "Tanaka" -GivenName "Mori" -Office "Security" -OtherAttributes @{'title'="Sensei";'mail'="MTanaka@greenhorn.corp"} -Accountpassword (Read-Host -AsSecureString "AccountPassword") -Enabled $true 

AccountPassword: ****************
PS C:\htb> Get-ADUser -Identity MTanaka -Properties * | Format-Table Name,Enabled,GivenName,Surname,Title,Office,Mail

Name    Enabled GivenName Surname Title  Office   Mail
----    ------- --------- ------- -----  ------   ----
MTanaka    True Mori      Tanaka  Sensei Security MTanaka@greenhorn.corp
```

Tamam, burada çok şey oluyor. Göz korkutucu görünebilir ama gelin inceleyelim. Yukarıdaki çıktının ilk kısmı kullanıcımızı oluşturuyor:

* ==New-ADUser -Name “MTanaka”== : New-ADUser komutunu veriyoruz ve kullanıcının SamAccountName'ini MTanaka olarak ayarlıyoruz.
* ==-Surname “Tanaka” -GivenName “Mori”==: Bu kısım kullanıcımızın Soyadını ve Adını ayarlar.
* ==-Office “Security”==: Office'in genişletilmiş özelliğini Security olarak ayarlar.
* ==-OtherAttributes @{'title'=“Sensei”;'mail'=“MTanaka@greenhorn.corp”}==: Burada başlık ve E-posta-Adres gibi diğer genişletilmiş özellikleri ayarlarız.
* ==-Accountpassword (Read-Host -AsSecureString “AccountPassword”)==: Bu kısımla, shell'in bizden yeni bir şifre girmemizi istemesini sağlayarak kullanıcının şifresini ayarlıyoruz. (bunu aşağıdaki satırda yıldızlarla görebiliriz)
* ==-Enabled $true==: Hesabı kullanım için etkinleştiriyoruz. Bu \$False olarak ayarlanmışsa kullanıcı oturum açamaz.

İkincisi, oluşturduğumuz kullanıcının ve ayarladığımız özelliklerin var olduğunu doğrulamaktır:

* ==Get-ADUser -Identity MTanaka -Properties== *: Burada, MTanaka kullanıcısının özelliklerini arıyoruz.
* ==|== : Bu Pipe sembolüdür. Başka bir bölümde daha fazla incelenecek, ancak şimdilik Get-ADUser'dan çıktımızı alıyor ve aşağıdaki komuta gönderiyor.
* ==Format-Table Name,Enabled,GivenName,Surname,Title,Office,Mail==: Burada, PowerShell'e sonuçlarımızı listelenen varsayılan ve genişletilmiş özellikleri içeren bir tablo olarak Formatlamasını söylüyoruz.

Komutların bu şekilde parçalara ayrıldığını görmek stringlerin gizemini çözmeye yardımcı olur. Peki ya bir kullanıcıyı değiştirmemiz gerekirse? Set-ADUser bizim çözümümüzdür. Daha önce incelediğimiz filtrelerin çoğu burada da geçerlidir. Listelenen niteliklerden herhangi birini değiştirebilir veya ayarlayabiliriz. Bu örnek için, Bay Tanaka'ya bir Açıklama ekleyelim.


### Kullanıcı Özelliklerini Değiştirme
```powershell-session
PS C:\htb> Set-ADUser -Identity MTanaka -Description " Sensei to Security Analyst's Rocky, Colt, and Tum-Tum"  

PS C:\htb> Get-ADUser -Identity MTanaka -Property Description


Description       :  Sensei to Security Analyst's Rocky, Colt, and Tum-Tum
DistinguishedName : CN=MTanaka,CN=Users,DC=greenhorn,DC=corp
Enabled           : True
GivenName         : Mori
Name              : MTanaka
ObjectClass       : user
ObjectGUID        : c19e402d-b002-4ca0-b5ac-59d416166b3a
SamAccountName    : MTanaka
SID               : S-1-5-21-1480833693-1324064541-2711030367-1603
Surname           : Tanaka
UserPrincipalName :
```
AD sorgulandığında, belirlediğimiz açıklamanın Bay Tanaka'nın özniteliklerine eklendiğini görebiliriz. Kullanıcı ve grup yönetimi, sysadmin olarak kendimizi yaparken bulabileceğimiz ortak bir görevdir. Ancak, bir pentester olarak bunu neden önemsemeliyiz?

### Kullanıcıları ve Grupları Numaralandırmak Neden Önemlidir?
Kullanıcılar ve gruplar, bir Windows ortamının Pentesting'i ile ilgili zengin fırsatlar sunar. Kullanıcıların sıklıkla yanlış yapılandırıldığını görürüz. Onlara aşırı izinler verilmiş, gereksiz gruplara eklenmiş ya da zayıf/parolaları belirlenmemiş olabilir. Gruplar da aynı derecede değerli olabilir. Çoğu zaman gruplar iç içe üyeliğe sahip olur ve kullanıcıların ihtiyaç duymayabilecekleri ayrıcalıkları elde etmelerine olanak tanır. Bu yanlış yapılandırmalar Bloodhound gibi araçlarla kolayca bulunabilir ve görselleştirilebilir. Kullanıcıları ve Grupları numaralandırmaya ayrıntılı bir bakış için Windows Privilege Escalation modülüne göz atın.



Soru : Hangi PowerShell Cmdlet'i bir host üzerindeki tüm LOCAL kullanıcıları görüntüler?
Cevap : Get-LocalUser

Soru : 



# Working with Files and Directories
Sadece PowerShell kullanarak host üzerinde nasıl gezineceğimizi, kullanıcıları ve grupları nasıl yöneteceğimizi zaten biliyoruz; şimdi sıra dosyaları ve dizinleri keşfetmeye geldi. Bu bölümde, dosya ve dizin oluşturma, değiştirme ve silme işlemlerinin yanı sıra dosya izinlerine ve bunların nasıl numaralandırılacağına hızlı bir giriş yapacağız. Şimdiye kadar, diğerlerinin yanı sıra Get, Set, New fiillerine aşina olmalıyız, bu nedenle birkaç komutu tek bir shell oturumunda birleştirerek örneklerimizle bunu hızlandıracağız.


## Creating/Moving/Deleting Files & Directories
Bu bölümde tartışacağımız cmdlet'lerin çoğu dosya ve klasörlerle çalışmak için de geçerlidir, bu nedenle daha verimli çalışmak için bazı eylemlerimizi birleştireceğiz (her iyi pentester veya sysadmin'in yapması gerektiği gibi). Aşağıdaki tabloda PowerShell'de nesnelerle çalışırken yaygın olarak kullanılan cmdlet'ler listelenmektedir.
![Pasted image 20241012203759.png](/img/user/resimler/Pasted%20image%2020241012203759.png)

Senaryo: Greenhorn'un yeni Güvenlik Şefi Bay Tanaka, kendisi için bir dizi dosya ve klasör oluşturulmasını talep etti. Bunları Güvenlik ekibi için SOP dokümantasyonu için kullanmayı planlıyor. Host erişimine yeni sahip olduğu için, dosya ve klasör yapısını onun için ayarlamayı kabul ettik. Aşağıdaki örneklerle birlikte takip etmek isterseniz, lütfen çekinmeyin. Pratik yapmanız için, aşağıda tartışılan klasörleri ve dosyaları kaldırdık, böylece bunları yeniden oluşturabilirsiniz.

İlk olarak, istediği klasör yapısı ile başlayacağız. . adında üç klasör oluşturacağız:
- `SOPs`
    - `Physical Sec`
    - `Cyber Sec`
    - `Training`

Klasör yapımızı oluşturmak için Get-Item, Get-ChildItem ve New-Item komutlarını kullanacağız. Hadi başlayalım. Öncelikle hostta nerede olduğumuzu belirlememiz ve ardından Bay Tanaka'nın Documents klasörüne geçmemiz gerekiyor.


### Yerimizi Bulmak
```powershell-session
PS C:\htb> Get-Location

Path
----
C:\Users\MTanaka

PS C:\Users\MTanaka> cd Documents
PS C:\Users\MTanaka\Documents>
```

Artık doğru dizinde olduğumuza göre, işe koyulma zamanı geldi. Daha sonra, SOPs klasörünü oluşturmamız gerekiyor. Bunu gerçekleştirmek için New-Item Cmdlet'i kullanılabilir.


### New-Item
```powershell-session
PS C:\Users\MTanaka\Documents>  new-item -name "SOPs" -type directory


    Directory: C:\Users\MTanaka\Documents


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/5/2022  12:20 PM                SOPs
```
Harika. Ana dizinimiz artık mevcut. İç içe geçmiş Physical Sec, Cyber Sec ve Training klasörlerimizi oluşturalım. Geçen seferki komutun aynısını ya da mkdir takma adını kullanabiliriz. İlk olarak, SOPs Dizinine geçmemiz gerekiyor.


#### Making More Directories
```powershell-session
PS C:\Users\MTanaka\Documents> cd SOPs 

PS C:\Users\MTanaka\Documents\SOPs> mkdir "Physical Sec"

    Directory: C:\Users\MTanaka\Documents\SOPs


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/5/2022   4:30 PM                Physical Sec

PS C:\Users\MTanaka\Documents\SOPs> mkdir "Cyber Sec"

    Directory: C:\Users\MTanaka\Documents\SOPs


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/5/2022   4:30 PM                Cyber Sec

PS C:\Users\MTanaka\Documents\SOPs> mkdir "Training"

    Directory: C:\Users\MTanaka\Documents\SOPs


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----         10/5/2022   4:31 PM                Training  

PS C:\Users\MTanaka\Documents\SOPs> Get-ChildItem 

Directory: C:\Users\MTanaka\Documents\SOPs


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        10/5/2022   9:08 AM                Cyber Sec
d-----        11/5/2022   9:09 AM                Physical Sec
d-----        11/5/2022   9:08 AM                Training
```

Artık dizin yapımız yerinde olduğuna göre. Gerekli dosyaları doldurmaya başlamanın zamanı geldi. Bay Tanaka her klasörde şu şekilde bir Markdown dosyası istedi:

- `SOPs` > ReadMe.md
    - `Physical Sec` > Physical-Sec-draft.md
    - `Cyber Sec` > Cyber-Sec-draft.md
    - `Training` > Employee-Training-draft.md

Her dosyada, en üstte bu başlığı talep etmiştir:
- Title: Insert Document Title Here
- Date: x/x/202x
- Author: MTanaka
- Version: 0.1 (Draft)


New-Item cmdlet'ini ve Add-Content cmdlet'ini kullanarak bunu hızlıca halledebilmeliyiz.


#### Making Files
```powershell-session
PS C:\htb> PS C:\Users\MTanaka\Documents\SOPs> new-Item "Readme.md" -ItemType File

    Directory: C:\Users\MTanaka\Documents\SOPs

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:12 AM              0 Readme.md

PS C:\Users\MTanaka\Documents\SOPs> cd '.\Physical Sec\'
PS C:\Users\MTanaka\Documents\SOPs\Physical Sec> ls
PS C:\Users\MTanaka\Documents\SOPs\Physical Sec> new-Item "Physical-Sec-draft.md" -ItemType File

    Directory: C:\Users\MTanaka\Documents\SOPs\Physical Sec

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:14 AM              0 Physical-Sec-draft.md

PS C:\Users\MTanaka\Documents\SOPs\Physical Sec> cd ..
PS C:\Users\MTanaka\Documents\SOPs> cd '.\Cyber Sec\'

PS C:\Users\MTanaka\Documents\SOPs\Cyber Sec> new-Item "Cyber-Sec-draft.md" -ItemType File

    Directory: C:\Users\MTanaka\Documents\SOPs\Cyber Sec

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:14 AM              0 Cyber-Sec-draft.md

PS C:\Users\MTanaka\Documents\SOPs\Cyber Sec> cd ..
PS C:\Users\MTanaka\Documents\SOPs> cd .\Training\
PS C:\Users\MTanaka\Documents\SOPs\Training> ls
PS C:\Users\MTanaka\Documents\SOPs\Training> new-Item "Employee-Training-draft.md" -ItemType File

    Directory: C:\Users\MTanaka\Documents\SOPs\Training

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:15 AM              0 Employee-Training-draft.md

PS C:\Users\MTanaka\Documents\SOPs\Training> cd ..
PS C:\Users\MTanaka\Documents\SOPs> tree /F
Folder PATH listing
Volume serial number is F684-763E
C:.
│   Readme.md
│
├───Cyber Sec
│       Cyber-Sec-draft.md
│
├───Physical Sec
│       Physical-Sec-draft.md
│
└───Training
        Employee-Training-draft.md
```

Artık dosyalarımız olduğuna göre, içlerine içerik eklememiz gerekiyor. Bunu Add-Content cmdlet'i ile yapabiliriz.


#### Adding Content

```powershell-session
PS C:\htb> Add-Content .\Readme.md "Title: Insert Document Title Here
>> Date: x/x/202x
>> Author: MTanaka
>> Version: 0.1 (Draft)"  
  
PS C:\Users\MTanaka\Documents\SOPs> cat .\Readme.md
Title: Insert Document Title Here
Date: x/x/202x
Author: MTanaka
Version: 0.1 (Draft)
```

Readme.md için yaptığımız bu işlemin aynısını Bay Tanaka için oluşturduğumuz diğer tüm dosyalarda da uygulayacaktık. Bu senaryo biraz sıkıcı geldi, değil mi? Dosyaları tekrar tekrar elle oluşturmak yorucu olabilir. İşte bu noktada otomasyon ve komut dosyası devreye giriyor. Şu an için biraz zor ama bu modülün ilerleyen bölümlerinde, işleri kolaylaştırmak için değişkenleri kullanarak ve komut dosyaları yazarak nasıl hızlı bir PowerShell Modülü oluşturacağımızı tartışacağız.

Senaryo Devam Ediyor..: Bay Tanaka, `Cyber-Sec-draft.md` dosyasının adını `Infosec-SOP-draft.md` olarak değiştirmemizi istedi.

Rename-Item cmdlet'ini kullanarak bu görevi hızlıca halledebiliriz. Hadi bir deneyelim:

### Renaming An Object
```powershell-session
PS C:\Users\MTanaka\Documents\SOPs\Cyber Sec> ls

    Directory: C:\Users\MTanaka\Documents\SOPs\Cyber Sec

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:14 AM              0 Cyber-Sec-draft.md

PS C:\Users\MTanaka\Documents\SOPs\Cyber Sec> Rename-Item .\Cyber-Sec-draft.md -NewName Infosec-SOP-draft.md
PS C:\Users\MTanaka\Documents\SOPs\Cyber Sec> ls

    Directory: C:\Users\MTanaka\Documents\SOPs\Cyber Sec

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/10/2022   9:14 AM              0 Infosec-SOP-draft.md
```
Yukarıda tek yapmamız gereken Rename-Item cmdlet'ini çalıştırmak, ona değiştirmek istediğimiz orijinal dosya adını (Cyber-Sec-draft.md) vermek ve ardından -NewName (Infosec-SOP-draft.md) parametresiyle yeni adımızı söylemekti. Basit görünüyor değil mi? Bunu daha da ileri götürebilir ve bir dizin içindeki tüm dosyaları yeniden adlandırabilir veya dosya türünü ya da birkaç farklı eylemi değiştirebiliriz. Aşağıdaki örneğimizde, Bay Tanakas Masaüstündeki tüm metin dosyalarının adlarını file.txt'den file.md'ye değiştireceğiz.


### Files1-5.txt MTanaka'nın Masaüstünde
```powershell-session
PS C:\Users\MTanaka\Desktop> ls

    Directory: C:\Users\MTanaka\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/13/2022   1:05 PM              0 file-1.txt
-a----        10/13/2022   1:05 PM              0 file-2.txt
-a----        10/13/2022   1:06 PM              0 file-3.txt
-a----        10/13/2022   1:06 PM              0 file-4.txt
-a----        10/13/2022   1:06 PM              0 file-5.txt

PS C:\Users\MTanaka\Desktop> get-childitem -Path *.txt | rename-item -NewName {$_.name -replace ".txt",".md"}
PS C:\Users\MTanaka\Desktop> ls

    Directory: C:\Users\MTanaka\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/13/2022   1:05 PM              0 file-1.md
-a----        10/13/2022   1:05 PM              0 file-2.md
-a----        10/13/2022   1:06 PM              0 file-3.md
-a----        10/13/2022   1:06 PM              0 file-4.md
-a----        10/13/2022   1:06 PM              0 file-5.md
```
Yukarıda gördüğümüz gibi, Masaüstünde beş metin dosyamız vardı. Nesneleri seçmek için get-childitem -Path yıldız.txt komutunu kullanarak bunları .md dosyalarına dönüştürdük ve bu nesneleri rename-item -NewName {dolar_.name -replace “.txt”,“.md”} cmdlet'ine göndermek için | komutunu kullanarak her şeyi orijinal adından (dolar_.name) yeniden adlandırdık ve addaki .txt'yi .md ile değiştirdik. Bu, dosyalarla etkileşim kurmanın ve toplu eylemler gerçekleştirmenin çok daha hızlı bir yoludur. Sayın Tanakas'ın tüm isteklerini yerine getirdiğimize göre, şimdi bir an için Dosya ve Dizin izinlerini tartışalım.

### Dosya ve Dizin İzinleri Nedir?
Basitleştirilmiş haliyle izinler, host'umuzun belirli bir nesneye kimin erişebileceğini ve bu nesneyle neler yapabileceğini belirleme yöntemidir. Bu izinler, uygun bir güvenlik duruşu sağlamak için nesnelerimiz üzerinde ayrıntılı güvenlik kontrolü uygulamamıza olanak tanır. Birden fazla departmanı (İK, BT, Satış vb.) olan büyük kuruluşlar gibi ortamlarda, bilgi erişimini “bilmesi gerekenler” temelinde tuttuklarından emin olmak isterler. Bu, dışarıdan birinin verileri bozamamasını veya kötüye kullanamamasını sağlar. Windows dosya sistemi birçok temel ve gelişmiş izne sahiptir. Temel izin türlerinden bazıları şunlardır:



### İzin Türlerinin Açıklanması
* ==Full Control (Tam Kontrol)==: Tam Kontrol, belirtilen kullanıcı veya grubun dosyayla uygun gördüğü şekilde etkileşime girmesine olanak tanır. Bu, aşağıdaki her şeyi, izinleri değiştirmeyi ve dosyanın sahipliğini almayı içerir.
* ==Modify (Değiştir)==: Dosya ve klasörlerin okunmasına, yazılmasına ve silinmesine izin verir.
* ==List Folder Contents:== Bu, dosyaları yürütmenin yanı sıra klasörleri ve alt klasörleri görüntülemeyi ve listelemeyi mümkün kılar. Bu yalnızca klasörler için geçerlidir.
* ==Read and Execute (Oku ve Çalıştır)==: Kullanıcıların dosyaların içindeki içeriği görüntülemesine ve yürütülebilir dosyaları (.ps1, .exe, .bat, vb.) çalıştırmasına olanak tanır.
* ==Write (Yaz)==: Yazma: Kullanıcının yeni dosyalar ve alt klasörler oluşturabilmesinin yanı sıra dosyalara içerik ekleyebilmesini sağlar.
* ==Read (Oku):== Klasörlerin ve alt klasörlerin görüntülenmesine ve listelenmesine ve bir dosyanın içeriğinin görüntülenmesine izin verir.
* ==Traverse Folder==: Traverse, bir kullanıcıya bir ağaç içindeki dosyalara veya alt klasörlere erişme yeteneği vermemizi sağlar, ancak üst düzey klasörün içeriğine erişemez. Bu, güvenlik açısından seçici erişim sağlamanın bir yoludur.

Windows (genel olarak NTFS) bir üst dizinde izinleri ayarlamamıza ve bu izinlerin o dizinde bulunan her dosya ve klasörü doldurmasına izin verir. Bu, içinde bulunan her nesnenin izinlerini manuel olarak ayarlamaya kıyasla bize tonlarca zaman kazandırır. Bu miras, belirli dosyalar, klasörler ve alt klasörler için gerektiğinde devre dışı bırakılabilir. Bu yapılırsa, etkilenen dosyalarda istediğimiz izinleri manuel olarak ayarlamamız gerekecektir. İzinlerle çalışmak karmaşık bir görev olabilir ve sadece CLI'dan yapmak için biraz fazla olabilir, bu nedenle izinlerle oynamayı Windows Temelleri Modülüne bırakacağız.

Dosyalar ve Dizinlerle çalışmak bazen biraz sıkıcı olsa da basittir. İleride, CLI temelimize başka bir katman ekleyeceğiz ve host üzerindeki dosyalar içindeki içeriği nasıl bulabileceğimize ve filtreleyebileceğimize bakacağız.

Soru : Hangi Cmdlet'in takma adı “cat” ?
Cevap : Get-Content

Soru : Yeni dosya ve klasörler oluşturmak için hangi Cmdlet'i kullanabiliriz?
Cevap : New-Item


### Finding & Filtering Content
Aradığımız şey için içerik arayabilmek, bulabilmek ve filtreleyebilmek, CLI kullanan herhangi bir kullanıcı için mutlak bir gerekliliktir (hangi shell veya işletim sistemi olursa olsun). Bununla birlikte, bunu PowerShell'de nasıl yaparız? Bu soruyu yanıtlamak için, bu bölümde PowerShell'in Objects'i nasıl kullandığını, Properties ve içeriğe dayalı olarak nasıl filtreleme yapabileceğimizi ve PowerShell Pipeline gibi bileşenleri daha ayrıntılı olarak açıklayacağız.


### PowerShell Çıktısının Açıklanması (Açıklanan Objeler)
PowerShell'de her şey Bash veya cmd'deki gibi genel bir text dizesi değildir. PowerShell'de her şey bir Object'tir. Ancak, Object nedir? Bu kavramı daha ayrıntılı inceleyelim:

Object nedir? Object, PowerShell içindeki bir class'ın bireysel bir örneğidir. Objemiz olarak bir bilgisayar örneğini kullanalım. Her şeyin toplamı (parçalar, zaman, tasarım, yazılım, vb.) bir bilgisayarı bilgisayar yapar.

Class nedir? Class, bir şeyin (obje) ve özelliklerinin toplamının onu nasıl tanımladığının şeması veya 'benzersiz temsilidir. Bilgisayarın nasıl bir araya getirilmesi gerektiğini ve içindeki her şeyin ne olduğunu ortaya koymak için kullanılan plan bir Class olarak kabul edilebilir.

Properties (Özellikler) nedir? Properties (Özellikler), PowerShell'de bir obje ile ilişkilendirilmiş verilerdir. Bilgisayar örneğimizde, bilgisayarı oluşturmak için bir araya getirdiğimiz tek tek parçalar onun özellikleridir. Her parça bir amaca hizmet eder ve obje içinde benzersiz bir kullanıma sahiptir.

Metotlar Nedir? Basitçe söylemek gerekirse, metotlar objemizin sahip olduğu tüm fonksiyonlardır. Bilgisayarımız veri işlememizi, internette gezinmemizi, yeni beceriler öğrenmemizi vb. sağlar. Tüm bunlar objemiz için metotlardır.

Şimdi, bu terimleri tanımladık, böylece daha sonra bakacağımız tüm farklı özellikleri ve objelerle hangi etkileşim yöntemlerine sahip olduğumuzu anlayabiliriz. PowerShell'in objelere nasıl yorum getirdiğini ve Class'ları nasıl kullandığını anlayarak kendi obje tiplerimizi tanımlayabiliriz. İleride, PowerShell CLI aracılığıyla objeleri nasıl filtreleyebileceğimize ve bulabileceğimize bakacağız.


### Objeleri Bulma ve Filtreleme
Buna bir user object (kullanıcı nesnesi) bağlamında bakalım. Bir user dosyalara erişmek, uygulamaları çalıştırmak ve veri input/output etmek gibi şeyler yapabilir. Peki kullanıcı nedir? Nelerden oluşur?

### Bir Obje ( User) ve Özelliklerini/Metotlarını Almak
```powershell-session
PS C:\htb> Get-LocalUser administrator | get-member

   TypeName: Microsoft.PowerShell.Commands.LocalUser

Name                   MemberType Definition
----                   ---------- ----------
Clone                  Method     Microsoft.PowerShell.Commands.LocalUser Clone()
Equals                 Method     bool Equals(System.Object obj)
GetHashCode            Method     int GetHashCode()
GetType                Method     type GetType()
ToString               Method     string ToString()
AccountExpires         Property   System.Nullable[datetime] AccountExpires {get;set;}
Description            Property   string Description {get;set;}
Enabled                Property   bool Enabled {get;set;}
FullName               Property   string FullName {get;set;}
LastLogon              Property   System.Nullable[datetime] LastLogon {get;set;}
Name                   Property   string Name {get;set;}
ObjectClass            Property   string ObjectClass {get;set;}
PasswordChangeableDate Property   System.Nullable[datetime] PasswordChangeableDate {get;set;}
PasswordExpires        Property   System.Nullable[datetime] PasswordExpires {get;set;}
PasswordLastSet        Property   System.Nullable[datetime] PasswordLastSet {get;set;}
PasswordRequired       Property   bool PasswordRequired {get;set;}
PrincipalSource        Property   System.Nullable[Microsoft.PowerShell.Commands.PrincipalSource] PrincipalSource {ge...
SID                    Property   System.Security.Principal.SecurityIdentifier SID {get;set;}
UserMayChangePassword  Property   bool UserMayChangePassword {get;set;}
```

Artık bir kullanıcının tüm özelliklerini görebildiğimize göre, bu özelliklerin PowerShell tarafından çıktılandığında nasıl göründüğüne bakalım. [Select-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-object?view=powershell-7.2) cmdlet'i bunu başarmamıza yardımcı olacaktır. Bu şekilde, artık bir user nesnesini neyin oluşturduğunu anlıyoruz.

### Property Output (All)
```powershell-session
PS C:\htb> Get-LocalUser administrator | Select-Object -Property *


AccountExpires         :
Description            : Built-in account for administering the computer/domain
Enabled                : False
FullName               :
PasswordChangeableDate :
PasswordExpires        :
UserMayChangePassword  : True
PasswordRequired       : True
PasswordLastSet        :
LastLogon              : 1/20/2021 5:39:14 PM
Name                   : Administrator
SID                    : S-1-5-21-3916821513-3027319641-390562114-500
PrincipalSource        : Local
ObjectClass            : User
```

Bir kullanıcı gerçekçi olarak küçük bir objedir, ancak özellikle büyük listeler veya tablolar gibi öğelerin çıktısına bu şekilde bakmak çok fazla olabilir. Peki ya bu içeriği filtrelemek ya da bize daha kesin bir şekilde göstermek istersek? Görmek istemediğimiz bir nesnenin özelliklerini, istediğimiz birkaç tanesini seçerek filtreleyebiliriz. Kullanıcılarımıza bakalım ve hangilerinin yakın zamanda bir parola belirlediğini görelim.


### Properties Üzerinde Filtreleme
```powershell-session
PS C:\htb> Get-LocalUser * | Select-Object -Property Name,PasswordLastSet

Name               PasswordLastSet
----               ---------------
Administrator
DefaultAccount
Guest
MTanaka              1/27/2021 2:39:55 PM
WDAGUtilityAccount 1/18/2021 7:40:22 AM
```

Ayrıca objelerimizi bu özelliklere göre sıralayabilir ve gruplayabiliriz.

### Sıralama ve Gruplama
```powershell-session
PS C:\htb> Get-LocalUser * | Sort-Object -Property Name | Group-Object -property Enabled

Count Name                      Group
----- ----                      -----
    4 False                     {Administrator, DefaultAccount, Guest, WDAGUtilityAccount}
    1 True                      {MTanaka}
```

Sort-Object ve Group-Object cmdlet'lerini kullanarak tüm kullanıcıları bulduk, ada göre sıraladık ve ardından Enabled özelliklerine göre gruplandırdık. Çıktıdan, birkaç kullanıcının devre dışı bırakıldığını ve etkileşimli oturum açma için kullanılmadığını görebiliriz. Bu, PowerShell objeleriyle neler yapılabileceğine ve her bir objede depolanan bilgi miktarına dair sadece hızlı bir örnektir. PowerShell'de daha derinlere indikçe ve Windows işletim sistemi içinde araştırma yaptıkça, birçok objenin arkasındaki class'ların kapsamlı olduğunu ve sıklıkla paylaşıldığını fark edeceğiz. Onlarla daha fazla çalışırken bunları aklınızda bulundurun.


### Sonuçlarımızı Neden Filtrelememiz Gerekiyor?
Bu tanıtım için bir get-service örneği kullanarak konuyu değiştiriyoruz. Temel kullanıcılara ve bilgilere bakmak çok fazla sonuç üretmez, ancak diğer nesneler olağanüstü miktarda veri içerir. Aşağıda Get-Service çıktısından sadece bir parçanın örneği bulunmaktadır:


#### Too Much Output
```powershell-session
PS C:\htb> Get-Service | Select-Object -Property *

Name                : AarSvc_1ca8ea
RequiredServices    : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
DisplayName         : Agent Activation Runtime_1ca8ea
DependentServices   : {}
MachineName         : .
ServiceName         : AarSvc_1ca8ea
ServicesDependedOn  : {}
ServiceHandle       :
Status              : Stopped
ServiceType         : 224
StartType           : Manual
Site                :
Container           :

Name                : AdobeARMservice
RequiredServices    : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : True
DisplayName         : Adobe Acrobat Update Service
DependentServices   : {}
MachineName         : .
ServiceName         : AdobeARMservice
ServicesDependedOn  : {}
ServiceHandle       :
Status              : Running
ServiceType         : Win32OwnProcess
StartType           : Automatic
Site                :
Container           :

Name                : agent_ovpnconnect
RequiredServices    : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : True
DisplayName         : OpenVPN Agent agent_ovpnconnect
DependentServices   : {}
MachineName         : .
ServiceName         : agent_ovpnconnect
ServicesDependedOn  : {}
ServiceHandle       :
Status              : Running
ServiceType         : Win32OwnProcess
StartType           : Automatic
Site                :
Container           :

<SNIP>
```

Bu, elemek için çok fazla veri demek, değil mi? Bunu daha da parçalayalım ve bu verileri bir liste olarak biçimlendirelim. Çıktımızı aşağıdaki gibi değiştirmek için ==get-service | Select-Object -Property DisplayName,Name,Status | Sort-Object DisplayName | fl== komut dizesini kullanabiliriz:

```powershell-session
PS C:\htb> get-service | Select-Object -Property DisplayName,Name,Status | Sort-Object DisplayName | fl 

<SNIP>
DisplayName : ActiveX Installer (AxInstSV)
Name        : AxInstSV
Status      : Stopped

DisplayName : Adobe Acrobat Update Service
Name        : AdobeARMservice
Status      : Running

DisplayName : Adobe Genuine Monitor Service
Name        : AGMService
Status      : Running
<SNIP>
```

Bu hala bir ton çıktıdır, ancak biraz daha okunabilirdir. Burada kendimize bu çıktıların hepsine ihtiyacımız var mı gibi sorular sormaya başlıyoruz. Bu objelerin tümüyle mi yoksa sadece belirli bir alt kümesiyle mi ilgileniyoruz? Ya belirli bir servisin çalışıp çalışmadığını belirlemek istiyorsak, ancak belirli bir Name'i bulmamız gerekiyorsa? [Where-Object](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/where-object?view=powershell-7.2), kendisine iletilen nesneleri ve bunların belirli özellik değerlerini değerlendirerek ihtiyacımız olan bilgileri arayabilir. Bu senaryoyu düşünün:


Senaryo: Güvenli olmayan bir protokol aracılığıyla bir host'a ilk shell'i indirdik ve host'u herkese açık hale getirdik. Daha ileri gitmeden önce, hostu değerlendirmemiz ve herhangi bir savunma hizmeti veya uygulamasının çalışıp çalışmadığını belirlememiz gerekir. İlk olarak, çalışan herhangi bir `Windows Defender` hizmeti örneği ararız.

Where-Object (takma ad olarak where) ve -like ile eşleşen parametreyi kullanmak, özellikte “Defender” olan herhangi bir şeyi arayarak devam etmek için güvenli olup olmadığımızı belirlememizi sağlayacaktır. Bu örnekte, Get-Service tarafından alınan tüm objelerin DisplayName özelliğini kontrol ediyoruz.


### Windows Defender için Avlanma
```powershell-session
PS C:\htb>  Get-Service | where DisplayName -like '*Defender*'

Status   Name               DisplayName
------   ----               -----------
Running  mpssvc             Windows Defender Firewall
Stopped  Sense              Windows Defender Advanced Threat Pr...
Running  WdNisSvc           Microsoft Defender Antivirus Networ...
Running  WinDefend          Microsoft Defender Antivirus Service
```

Gördüğümüz gibi, sonuçlarımız Defender Güvenlik Duvarı, Advanced Threat Protection ve daha fazlası dahil olmak üzere çalışan birkaç servis döndürdü. Bu bizim için hem iyi hem de kötü bir haber. Savunma hizmetleri tarafından tespit edilme olasılığımız yüksek olduğu için hemen içeri dalıp bir şeyler yapmaya başlayamayız, ancak onları tespit etmiş olmamız iyi bir şeydir ve şimdi yeniden gruplanabilir ve alınacak savunma amaçlı kaçınma eylemleri için bir plan yapabiliriz. Hızlı bir örnek senaryo olmasına rağmen, bu pentesterlar olarak sık sık karşılaşacağımız bir durumdur ve savunma önlemlerinin ne zaman alındığını tespit edebilmeliyiz. Ancak bu örnek, aramalarımızı değiştirmenin ilginç bir yolunu ortaya koymaktadır. Değerlendirme değerleri amacımız için faydalı olabilir. Onları daha fazla kontrol edelim.


### Hedef Değerlerin Değerlendirilmesi
Where ve diğer birçok cmdlet, nesneleri ve verileri, bu nesnelerin ve özelliklerinin içerdiği değerlere göre değerlendirebilir. Yukarıdaki çıktı, - like Comparison operatörünü kullanan mükemmel bir örnektir. İfade edilen değerlerle eşleşen her şeyi arar ve * gibi joker karakterler içerebilir. Aşağıda, kullanabileceğimiz diğer yararlı ifadelerin hızlı bir listesi (her şeyi kapsamamaktadır) yer almaktadır:


### Karşılaştırma Operatörleri
![Pasted image 20241013175626.png](/img/user/resimler/Pasted%20image%2020241013175626.png)

Elbette, NotEqual gibi büyüktür, küçüktür ve negatifler gibi kullanabileceğimiz başka birçok [karşılaştırma operatörü](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_comparison_operators?view=powershell-7.2) vardır, ancak bu tür bir aramada bunlar yaygın olarak kullanılmayabilir. Şimdi bu operatörlerin bize daha önce olduğundan daha fazla nasıl yardımcı olabileceğini -GTE bir anlayışla (orada ne yaptığımı görün), Defender hizmetlerini araştırmaya geri dönelim. Şimdi yine <bir şey>Defender<bir şey> gibi bir DisplayName'e sahip hizmet nesnelerini arayacağız.

### Defender Özellikleri
```powershell-session
PS C:\htb> Get-Service | where DisplayName -like '*Defender*' | Select-Object -Property *

Name                : mpssvc
RequiredServices    : {mpsdrv, bfe}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
DisplayName         : Windows Defender Firewall
DependentServices   :
MachineName         : .
ServiceName         : mpssvc
ServicesDependedOn  : {mpsdrv, bfe}
ServiceHandle       :
Status              : Running
ServiceType         : Win32ShareProcess
StartType           : Automatic
Site                :
Container           :

Name                : Sense
RequiredServices    : {}
CanPauseAndContinue : False
CanShutdown         : False
CanStop             : False
DisplayName         : Windows Defender Advanced Threat Protection Service
<SNIP>
```

Yukarıdaki sonuçlarımız artık Windows Defender ile ilişkili her hizmeti filtrelemekte ve her eşleşmenin tam özellik listesini görüntülemektedir. Artık servislere bakabilir, çalışıp çalışmadıklarını ve hatta mevcut izin seviyemizle bu servislerin durumunu etkileyip etkileyemeyeceğimizi (kapatma, devre dışı bırakma, vb.) belirleyebiliriz. Son birkaç bölümde verdiğimiz komutların çoğunda, genellikle ayrı ayrı verdiğimiz birden fazla komutu birleştirmek için | sembolünü kullandık. Aşağıda bunun ne olduğunu ve bizim için nasıl çalıştığını tartışacağız.


## What is the PowerShell Pipeline? ( | )
En basit haliyle PowerShell'deki [Pipeline](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_pipelines?view=powershell-7.2), son kullanıcıya komutları birbirine zincirlemenin bir yolunu sağlar. Bu zincire Pipeline adı verilir ve pipe veya piping commands together olarak da adlandırılır. PowerShell nesneleri bu şekilde ele aldığından, bir komut verebilir ve ardından ortaya çıkan obje çıktısını eylem için başka bir komuta pipe (|) edebiliriz. Pipeline, komutları soldan sağa doğru birer birer yorumlayacak ve yürütecektir. Bunu önceki bölümlerde birkaç örnekte yaptık, bu nedenle burada daha derine iniyoruz. Örnek olarak, komutları bir araya getirmek için Pipeline'ı kullanmak şöyle görünebilir:


#### Piping Commands
```powershell-session
PS C:\htb> Command-1 | Command-2 | Command-3

Output from the result of 1+2+3 
```

OR

```powershell-session
PS C:\htb> 
Command-1 |
  Command-2 |
    Command-3  

Output result from Pipeline
```

OR

```powershell-session
PS C:\htb> Get-Process | Where-Object CPU | Where-Object Path
    | Get-Item |  

Output result from Pipeline  
```

Her iki yol da komutları bir araya getirmenin mükemmel kabul edilebilir bir yoludur. PowerShell, string içindeki (|) işaretinin konumuna göre ne istediğinizi yorumlayabilir. Bize eyleme geçirilebilir veriler sağlamak için pipeline kullanımının bir örneğini görelim. Aşağıda Get-Process cmdlet'ini çalıştıracağız, elde edilen verileri sıralayacağız ve ardından hostumuzda kaç tane benzersiz process çalıştığını ölçeceğiz.


### Benzersiz Örnekleri Saymak için Pipeline'ı Kullanma
```powershell-session
PS C:\htb> get-process | sort | unique | measure-object

Count             : 113  
```
Sonuç olarak, pipeline o anda çalışan benzersiz processes'lerin toplam sayısını (113) verir. Pipeline'u herhangi bir noktada parçalara ayırdığımızı varsayalım. Bu durumda, proses çıktısını sıralanmış, benzersiz örnekler için filtrelenmiş (yinelenen adlar yok) veya ==Measure-Object== cmdlet'inden yalnızca bir sayı çıktısı olarak görebiliriz. Gerçekleştirdiğimiz görev nispeten basitti. Ancak, yeni günlük girdilerini sıralamak, belirli olay günlüğü kodları için filtreleme yapmak veya belirli dizeleri arayan büyük miktarda veriyi (örneğin bir veritabanı ve tüm girdileri) işlemek gibi daha karmaşık bir şey için bundan yararlanabilseydik ne olurdu? İşte bu noktada Pipeline üretkenliğimizi artırabilir ve aldığımız çıktıyı düzene sokabilir, bu da onu herhangi bir sistem yöneticisi veya pentester için hayati bir araç haline getirir.


### Pipeline Chain Operatörleri ( && ve || )
Şu anda, Windows PowerShell 5.1 ve daha eski sürümler bu şekilde kullanılan Pipeline Chain Operators'ı desteklememektedir. Hata görürseniz, Windows PowerShell'in yanında PowerShell 7'yi de yüklemeniz gerekir. İkisi aynı şey değildir.

Yeni ve güncellenmiş özelliklerin çoğunu kullanabilmeniz için PowerShell 7'yi [yüklemenin](https://www.thomasmaurer.ch/2019/07/how-to-install-and-update-powershell-7/) harika bir örneğini burada bulabilirsiniz. PowerShell, Chain operatörlerinin kullanımı ile pipeline'ların koşullu olarak yürütülmesini sağlar. Bu operatörler ( && ve || ) iki ana işleve hizmet eder:

* &&: Geçerli komut düzgün bir şekilde tamamlanırsa, PowerShell'in bir sonraki komutu satır içi olarak yürüteceği bir koşul belirler.

* ||: Geçerli komut başarısız olursa PowerShell'in sonraki komutu satır içi olarak yürüteceği bir koşul ayarlar.

Bu operatörler, bir hedef veya koşul karşılandığında çalıştırılan komut dosyaları için koşullar belirlememize yardımcı olabilir. Örneğin:

Senaryo: Diyelim ki bir dosyanın içeriğini almak ve ardından bir host'a ping atmak istediğimiz bir komut zinciri yazıyoruz. Bunu, && ile ilk komut başarılı olursa host'a ping atacak şekilde veya yalnızca komut başarısız olursa çalışacak şekilde ayarlayabiliriz ||. Her ikisini de görelim.

Bu çıktıda, ping komutumuzun sonuçlarıyla birlikte test.txt dosyasının çıktısını konsola yazdırdığımız için her iki komutun da başarılı bir şekilde yürütüldüğünü görebiliriz.


#### Successful Pipeline
```powershell-session
PS C:\htb> Get-Content '.\test.txt' && ping 8.8.8.8
pass or fail

Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=23ms TTL=118
Reply from 8.8.8.8: bytes=32 time=28ms TTL=118
Reply from 8.8.8.8: bytes=32 time=28ms TTL=118
Reply from 8.8.8.8: bytes=32 time=21ms TTL=118

Ping statistics for 8.8.8.8:
    Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
Approximate round trip times in milli-seconds:
    Minimum = 21ms, Maximum = 28ms, Average = 25ms
```
Bu çıktı ile pipeline'ımızın ilk komuttan sonra kendini kapattığını, çünkü dosyanın çıktısını konsola yazdırarak yeterli şekilde çalıştığını görebiliriz.


#### Stop Unless Failure
```powershell-session
PS C:\htb>  Get-Content '.\test.txt' || ping 8.8.8.8

pass or fail
```

Burada pipeline'ımızın tamamen çalıştığını görebiliriz. İlk komutumuz dosya adı yanlış yazıldığı için başarısız oldu ve PowerShell bunu istediğimiz dosyanın mevcut olmadığı şeklinde algıladı. İlk komut başarısız olduğu için ikinci komutumuz çalıştırıldı.


#### Success in Failure
```powershell-session
PS C:\htb> Get-Content '.\testss.txt' || ping 8.8.8.8

Get-Content: Cannot find path 'C:\Users\MTanaka\Desktop\testss.txt' because it does not exist.

Pinging 8.8.8.8 with 32 bytes of data:
Reply from 8.8.8.8: bytes=32 time=20ms TTL=118
Reply from 8.8.8.8: bytes=32 time=37ms TTL=118
Reply from 8.8.8.8: bytes=32 time=19ms TTL=118

<SNIP>
```

Kullandığımız pipeline ve operatörler, nesneleri ve verileri bir görevden diğerine hızlı bir şekilde aktarabilmenin yanı sıra zaman tasarrufu açısından da bizim için faydalıdır. Birden fazla komutu sırayla vermek, her bir komutu manuel olarak vermekten çok daha etkilidir. Peki ya dosya ve dizinlerin içeriğindeki stringleri ya da verileri aramak istersek? Bu, birçok pentesterin erişim sağladıkları bir hostu numaralandırırken gerçekleştirecekleri yaygın bir görevdir. Hostta yerel olarak bulunanlarla arama yapmak, gizliliğimizi korumak ve kullanıcı ortamına araçlar getirerek yeni riskler getirmediğimizden emin olmak için harika bir yoldur.


### İçerik İçinde Veri Bulma
Snaffler, Winpeas ve benzerleri gibi ilginç dosyaları ve stringleri arayabilen bazı araçlar mevcuttur, ancak ya host üzerine yeni bir araç getiremiyorsak? Kimlik bilgileri, anahtarlar vb. gibi hassas bilgileri nasıl avlayabiliriz? Select-String ve where gibi yeni cmdlet'lerle eşleştirilmiş önceki bölümlerde uyguladığımız cmdlet'leri birleştirmek, bir dosya sisteminde root yapmamız için mükemmel bir yoldur.

Linux CLI kullanmaya daha aşina olanlar için [Select-String](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/select-string?view=powershell-7.2) (takma ad olarak sls), Grep'in yaptığı gibi veya Windows Komut İstemi içindeki findstr.exe ile aynı şekilde çalışır. Girdi stringleri, dosya içerikleri ve daha fazlasını düzenli ifade (regex) desen eşleştirmesine dayalı olarak değerlendirir. Bir eşleşme bulunduğunda, Select-String eşleşen satırı, dosyanın adını ve varsayılan olarak bulunduğu satır numarasını çıktılayacaktır. Genel olarak, herkesin araç kutusunda bulunması gereken esnek ve yararlı bir cmdlet'tir. Aşağıda, bir hostu numaralandırırken dikkat edilmesi gereken bazı ilginç dosya ve dizinlerdeki bilgileri ararken yeni cmdlet'imizi bir test sürüşüne çıkaracağız.


### Bir Dizin İçindeki İlginç Dosyaları Bulma
İlginç dosyalar ararken, günlük olarak kullandığımız en yaygın dosya türlerini düşünün ve oradan başlayın. Belirli bir günde text dosyaları, biraz Markdown, biraz Python, PowerShell ve daha pek çok şey yazabiliriz. Kullanıcıların ve yöneticilerin en çok etkileşimde bulunacağı yer olduğu için bir hostta arama yaparken bunları aramak isteriz. ==Get-ChildItem== ile başlayabilir ve bir klasörde özyinelemeli arama yapabiliriz. Bunu test edelim.



### Avın Başlangıcı
```powershell-session
PS C:\htb> Get-ChildItem -Path C:\Users\MTanaka\ -File -Recurse 

 Directory: C:\Users\MTanaka\Desktop\notedump\NoteDump

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---           4/26/2022  1:47 PM           1092 demo notes.md
-a---           4/22/2022  2:20 PM           1074 noteDump.py
-a---           4/22/2022  2:55 PM          61440 plum.sqlite
-a---           4/22/2022  2:20 PM            375 README.md
<SNIP>
```

Hızlı bir şekilde çok fazla bilgi döndürdüğünü fark edeceğiz. Belirtilen yoldaki her klasördeki her dosya konsolumuza çıktı olarak verildi. Bunu biraz azaltmamız gerekiyor. Belirli dosya türü uzantıları için isme bakma koşulunu kullanalım. Bunu yapmak için, ==Get-ChildItem== çıktısını where cmdlet'inden geçirerek çıktımızı filtreleyeceğiz. İlk olarak ==*.txt== dosya türü uzantısını arayarak test edelim.


### Aramamızı Daraltıyoruz
```powershell-session
PS C:\htb> Get-Childitem –Path C:\Users\MTanaka\ -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.txt")}

Directory: C:\Users\MTanaka\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          10/11/2022  3:32 PM            183 demo-notes.txt
-a---            4/4/2022  9:37 AM            188 q2-to-do.txt
-a---          10/12/2022 11:26 AM             14 test.txt
-a---            1/4/2022 11:23 PM            310 Untitled-1.txt

    Directory: C:\Users\MTanaka\Desktop\win-stuff

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---           5/19/2021 10:12 PM           7831 wmic.txt

    Directory: C:\Users\MTanaka\Desktop\Workshop\

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-----            1/7/2022  4:39 PM            945 info.txt
```

Bu çok daha verimli çalıştı. Filtremizin ==$_.Name== niteliği nedeniyle yalnızca txt dosya türüyle eşleşen dosyaları döndürdük. Artık çalıştığını bildiğimize göre, where filtresi içinde bir -or ifadesi kullanarak arayacağımız diğer dosya türlerini ekleyebiliriz.


### Hazine Avımızı Genişletmek İçin Or'u Kullanmak
```powershell-session
PS C:\htb> Get-Childitem –Path C:\Users\MTanaka\ -File -Recurse -ErrorAction SilentlyContinue | where {($_.Name -like "*.txt" -or $_.Name -like "*.py" -or $_.Name -like "*.ps1" -or $_.Name -like "*.md" -or $_.Name -like "*.csv")}

 Directory: C:\Users\MTanaka\Desktop

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          10/11/2022  3:32 PM            183 demo-notes.txt
-a---          10/11/2022 10:22 AM           1286 github-creds.txt
-a---            4/4/2022  9:37 AM            188 q2-to-do.txt
-a---           9/18/2022 12:35 PM             30 notes.txt
-a---          10/12/2022 11:26 AM             14 test.txt
-a---           2/14/2022  3:40 PM           3824 remote-connect.ps1
-a---          10/11/2022  8:22 PM            874 treats.ps1
-a---            1/4/2022 11:23 PM            310 Untitled-1.txt

    Directory: C:\Users\MTanaka\Desktop\notedump\NoteDump

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---           4/26/2022  1:47 PM           1092 demo.md
-a---           4/22/2022  2:20 PM           1074 noteDump.py
-a---           4/22/2022  2:20 PM            375 README.md
```

Stringimiz çalıştı ve artık Get-ChildItem'dan birden fazla dosya türü alabiliyoruz! Artık elimizde ilginç dosyaların listesi olduğuna göre, bu nesneleri başka bir cmdlet'e (Select-String) aktararak içeriklerinde ilginç stringler ve anahtar kelimeler ya da ifadeler arayabiliriz. Bunu iş başında görelim.


#### Basic Search Query

```powershell-session
PS C:\htb> Get-ChildItem -Path C:\Users\MTanaka\ -Filter "*.txt" -Recurse -File | sls "Password","credential","key"

CFP-Notes.txt:99:Lazzaro, N. (2004). Why we play games: Four keys to more emotion without story. Retrieved from:
notes.txt:3:- Password: F@ll2022!
wmic.txt:67:  wmic netlogin get name,badpasswordcount
wmic.txt:69:Are the screensavers password protected? What is the timeout? good use: see that all systems are
complying with policy evil use: find systems to walk up and use (assuming physical access is an option)
```
Select-string'in varsayılan olarak büyük/küçük harfe duyarlı olmadığını unutmayın. Eğer öyle olmasını istersek, ona -CaseSensitive değiştiricisini ekleyebiliriz. Şimdi orijinal dosya aramamızı içerik filtremizle birleştireceğiz.


### Aramaların Birleştirilmesi
```powershell-session
PS C:\htb> Get-Childitem –Path C:\Users\MTanaka\ -File -Recurse -ErrorAction SilentlyContinue | where {($_. Name -like "*.txt" -or $_. Name -like "*.py" -or $_. Name -like "*.ps1" -or $_. Name -like "*.md" -or $_. Name -like "*.csv")} | sls "Password","credential","key","UserName"

New-PC-Setup.md:56:  - getting your vpn key
CFP-Notes.txt:99:Lazzaro, N. (2004). Why we play games: Four keys to more emotion without story. Retrieved from:
notes.txt:3:- Password: F@ll2022!
wmic.txt:54:  wmic computersystem get username
wmic.txt:67:  wmic netlogin get name,badpasswordcount
wmic.txt:69:Are the screensavers password protected? What is the timeout? good use: see that all systems are
complying with policy evil use: find systems to walk up and use (assuming physical access is an option)
wmic.txt:83:  wmic netuse get Name,username,connectiontype,localname
```
Pipeline'daki komutlarımız uzuyor, ancak görünümümüzü okunabilir hale getirmek için kolayca temizleyebiliriz. Yine de sonuçlarımıza baktığımızda, dosya listesi sonuçlarımızı anahtar kelime aramamızla beslemek çok daha sorunsuz bir süreç oldu. Komut satırımıza birkaç yeni ekleme yapıldığına dikkat edin. Bir hata oluştuğunda komutun devam etmesi için bir satır ekledik                 (==-ErrorAction SilentlyContinue==). Bu, okuyamadığı bir dosya veya dizinle karşılaştığında tüm pipeline'ımızın bozulmadan kalmasını sağlamamıza yardımcı olur. İçerik bulma ve filtreleme başlı başına ilginç bir bulmaca olabilir. Hangi kelimelerin ve stringlerin en iyi sonuçları vereceğini belirlemek sürekli gelişen bir görevdir ve genellikle müşteriye göre değişir.


### Kontrol Edilecek Yararlı Rehberler
Değerli dosyaları ve diğer içerikleri ararken, birçok farklı yerde daha birçok değerli dosyayı kontrol edebiliriz. Aşağıdaki liste ganimet arayışımızda kullanabileceğimiz sadece birkaç ipucu ve püf noktası içermektedir.
* Users \AppData\ klasörüne bakmak başlamak için harika bir yerdir. Birçok uygulama yapılandırma dosyalarını, belgelerin geçici kayıtlarını ve daha fazlasını depolar.
* Kullanıcıların ana klasörü C:\Users\User\ yaygın bir depolama yeridir; VPN anahtarları, SSH anahtarları ve daha fazlası gibi şeyler saklanır. Genellikle gizli klasörlerde. (Get-ChildItem -Hidden)
* Host tarafından tutulan Konsol Geçmişi dosyaları, özellikle de bir yöneticinin host'unda bulunuyorsanız, sonsuz bir bilgi kuyusudur. İki farklı noktayı kontrol edebilirsiniz:
- `C:\Users\<USERNAME>\AppData\Roaming\Microsoft\Windows\Powershell\PSReadline\ConsoleHost_history.txt`
- `Get-Content (Get-PSReadlineOption).HistorySavePath`
- Bir kullanıcının panosunu kontrol etmek de yararlı bilgiler sağlayabilir. Bunu ==Get-Clipboard== ile yapabilirsiniz
* Zamanlanmış görevlere bakmak da yardımcı olabilir.

Bunlar kontrol edilmesi gereken ilginç yerlerden sadece birkaçı. Becerileriniz ve deneyimleriniz arttıkça kendi kontrol listenizi oluşturmak ve sürdürmek için bunu bir başlangıç noktası olarak kullanın.

CLI Kung Fu'muzu hızla büyütüyoruz ve bir sonraki mücadeleye geçme zamanı geldi. İlerledikçe, nelerin yapılabileceğini ve bunları nasıl değiştirebileceğinizi anlamak için lütfen gösterilen örnekleri kendi başınıza deneyin. Bir sonraki dersimizde Servisler ve proseslerle çalışmaya başlayacağız.

Soru : Objelerimizin sahip olduğu fonksiyonları ne tanımlar?
Cevap : Methods

Soru : Hangi Cmdlet bize bir objenin özelliklerini ve metotlarını gösterebilir?
cevap : Get-Member

Soru : Bir dizinde ve tüm alt dizinlerde bir şey aramak istersek, Get-ChildItem Cmdlet'i ile hangi modifieri kullanırız?
Cevap : -Recurse



### Servislerle Çalışma
Önceki bölümümüzde, bir pentester'ın gözünden host üzerindeki servisleri bulma örneğini kullanarak filtreleme ve pipeline'dan bahsetmiştik. Bu bölümde, bu konuyu daha derinlemesine inceleyeceğiz ve tersine çevireceğiz. Konuya bir yöneticinin gözünden bakacağız.

Senaryo: Bay Tanaka Yardım Masasına bir mesaj göndererek günün erken saatlerinde bir pencere açıldığını fark ettiğini ve pencerede birçok bilgi yanıp söndüğü için bunun sadece Windows güncellemelerinin çalışması olduğunu düşündüğünü belirtti. Ancak şimdi Defender'ın kapalı olduğunu belirten uyarıların da açıldığını ve hostunun yavaş çalıştığını bildiriyor. Bunu incelememiz, Defender ile ilgili hangi servislerin kapatıldığını belirlememiz ve mümkünse bunları tekrar etkinleştirmemiz gerekiyor. Daha sonra olay günlüklerine bakacağız ve neler olduğunu göreceğiz.

Servis yönetimi, hostları yönetmek ve güvenlik duruşumuzun değişmeden kalmasını sağlamak için çok önemlidir. Bu bölümde servislerin ve izinlerinin gerektiği gibi nasıl sorgulanacağı, başlatılacağı, durdurulacağı ve düzenleneceği anlatılacaktır. Ayrıca bu servislerle lokal ve uzaktan etkileşime geçmenin yollarını da tartışacağız. Bir sonraki CLI Kung-Fu Kuşağımızı edinmenin ve dalmanın zamanı geldi.


### Servisler Nedir ve Powershell Kullanarak Onlarla Nasıl Etkileşim Kurarız?
Windows İşletim sistemindeki servisler temelde, host üzerinde kullanılan uygulamalar için prosesleri ve diğer gerekli bileşenleri yöneten ve sürdüren arka planda çalışan bir bileşenin tekil örnekleridir. Servisler genellikle kullanıcı etkileşimi gerektirmez ve kullanıcının etkileşim kurabileceği somut bir arayüze sahip değildir. Ayrıca host üzerinde servisin tekil bir örneği olarak bulunurken, bir servis bir prosesin birden fazla örneğini tutabilir. Bir proses, bir kullanıcı veya uygulamanın görevleri yerine getirmesi için geçici bir konteyner olarak düşünülebilir. Windows'un üç servis kategorisi vardır: Lokal Servisler, Ağ Servisleri ve Sistem Servisleri. Birçok farklı servis (Windows işletim sistemindeki temel bileşenler de dahil olmak üzere) aynı anda birden fazla proses örneğini idare eder. PowerShell bize Servislerle etkileşim için çeşitli cmdlet'ler içeren ==Microsoft.PowerShell.Management== modülünü sağlar. PowerShell'deki her şeyde olduğu gibi, nereden başlayacağınızdan veya hangi cmdlet'e ihtiyacınız olduğundan emin değilseniz, başlamanız için built-in yardımdan yararlanın.


#### Getting Help (Services)
```powershell-session
PS C:\htb> Get-Help *-Service  

Name                              Category  Module                    Synopsis
----                              --------  ------                    --------
Get-Service                       Cmdlet    Microsoft.PowerShell.Man… …
New-Service                       Cmdlet    Microsoft.PowerShell.Man… …
Remove-Service                    Cmdlet    Microsoft.PowerShell.Man… …
Restart-Service                   Cmdlet    Microsoft.PowerShell.Man… …
Resume-Service                    Cmdlet    Microsoft.PowerShell.Man… …
Set-Service                       Cmdlet    Microsoft.PowerShell.Man… …
Start-Service                     Cmdlet    Microsoft.PowerShell.Man… …
Stop-Service                      Cmdlet    Microsoft.PowerShell.Man… …
Suspend-Service                   Cmdlet    Microsoft.PowerShell.Man… …
```
Şimdi Bay Tanaka'nın host'unun incelemesine (triage) başlayalım ve neler olup bittiğini görelim.

Not: Çalışan sorgular dışındaki servisleri yönetmek veya değiştirmek için, bunu yapmak için doğru izinlere sahip olmamız gerektiğini unutmayın. Bu, kullanıcımızın ideal olarak host üzerinde lokal yönetici olması veya üyesi olduğu domain gruplarından bu izinlerin verilmiş olması gerektiği anlamına gelir. PowerShell'i yönetici bağlamında açmak da işe yarayacaktır.


#### Çalışan Servislerin İncelenmesi
Öncelikle hedef hostumuzdan hizmetlerin hızlı çalışan bir listesini almamız gerekir. Servislerin durumu Running (Çalışıyor), Stopped (Durduruldu) veya Paused (Duraklatıldı) olarak ayarlanabilir ve manuel olarak (kullanıcı etkileşimi), otomatik olarak (sistem başlangıcında) veya sistem açılışından sonra gecikmeli olarak başlatılmak üzere ayarlanabilir. Yönetici ayrıcalıklarına sahip kullanıcılar genellikle servisleri oluşturabilir, değiştirebilir ve silebilir. Servis izinleriyle ilgili yanlış yapılandırmalar Windows sistemlerinde yaygın bir ayrıcalık yükseltme vektörüdür.


#### Get-Service
```powershell-session
PS C:\htb> Get-Service | ft DisplayName,Status 

DisplayName                                                                         Status
-----------                                                                         ------

Adobe Acrobat Update Service                                                       Running
OpenVPN Agent agent_ovpnconnect                                                    Running
Adobe Genuine Monitor Service                                                      Running
Adobe Genuine Software Integrity Service                                           Running
Application Layer Gateway Service                                                  Stopped
Application Identity                                                               Stopped
Application Information                                                            Running
Application Management                                                             Stopped
App Readiness                                                                      Stopped
Microsoft App-V Client                                                             Stopped
AppX Deployment Service (AppXSVC)                                                  Running
AssignedAccessManager Service                                                      Stopped
Windows Audio Endpoint Builder                                                     Running
Windows Audio                                                                      Running
ActiveX Installer (AxInstSV)                                                       Stopped
GameDVR and Broadcast User Service_172433                                          Stopped
BitLocker Drive Encryption Service                                                 Running
Base Filtering Engine                                                              Running
<SNIP> 

PS C:\htb> Get-Service | measure  

Count             : 321
```

Çalıştırmayı biraz daha net hale getirmek için, servis listemizi format-table'a aktardık ve konsolumuzda görüntülenecek DisplayName ve Status özelliklerini seçtik. Verilen ikinci komutta, kaç tane servisle çalıştığımıza dair bir fikir edinmek için listede görünen servis sayısını ölçtük. 321 servisi bir kerede kaydırmak ve üzerinde çalışmak için çok fazla, bu yüzden biraz daha azaltmamız gerekiyor. Bay Tanaka'nın isteği üzerine, Windows Defender ile ilgili olası bir sorundan bahsetti, bu nedenle bununla ilgili olmayan tüm hizmetleri filtreleyelim.


### Defender'a Hassas Bakış
```powershell-session
PS C:\htb> Get-Service | where DisplayName -like '*Defender*' | ft DisplayName,ServiceName,Status

DisplayName                                             ServiceName  Status
-----------                                             -----------  ------
Windows Defender Firewall                               mpssvc      Running
Windows Defender Advanced Threat Protection Service     Sense       Stopped
Microsoft Defender Antivirus Network Inspection Service WdNisSvc    Running
Microsoft Defender Antivirus Service                    WinDefend   Stopped
```

Şimdi sadece Defender ile ilgili servisleri görebiliyoruz ve bir nedenden dolayı Microsoft Defender Antivirüs Servisi'nin (WinDefend) gerçekten de kapalı olduğunu görebiliyoruz. Şimdilik, Bay Tanaka'nın host'unun korunmasını sağlamak için, Start-Service cmdlet'ini kullanarak tekrar açmayı deneyelim.


#### Resume / Start / Restart a Service

```powershell-session
PS C:\htb> Start-Service WinDefend
```

Start-Service cmdlet'ini çalıştırdığımızda “ParserError: Bu script zararlı içerik içeriyor ve antivirüs yazılımınız tarafından engellendi.” gibi bir hata mesajı almadığımız sürece komut başarıyla çalışmıştır. Servisi sorgulayarak tekrar kontrol edebiliriz.


#### Checking Our Work

```powershell-session
PS C:\htb>  get-service WinDefend

Status   Name               DisplayName
------   ----               -----------
Running  WinDefend          Microsoft Defender Antivirus Service
```

DisplayName'deki herhangi bir şey yerine servisi başlatmak ve sorgulamak için servis Name'i nasıl kullandığımıza dikkat edin. Şimdilik Defender tekrar çalışır durumda, yani ilk görev tamamlandı. Hazır buradayken etrafa bir göz atalım ve başka neler olduğuna bakalım. Orada ne olduğunu görmek için servislere biraz daha baktığımızda, garip bir DisplayName'e sahip bir Servis fark ediyoruz.

```powershell-session
PS C:\htb> get-service 

Stopped  SmsRouter          Microsoft Windows SMS Router Service.
Stopped  SNMPTrap           SNMP Trap
Stopped  spectrum           Windows Perception Service
Running  Spooler            Totally still used for Print Spooli...
Stopped  sppsvc             Software Protection
Running  SSDPSRV            SSDP Discovery
```

Bu özel servis hakkında herhangi bir bilgi bulamıyoruz ve DisplayName'inin değişmesi garip, bu yüzden güvende olmak için şimdilik servisi durduracağız ve güvenlik ekibindeki ekip üyelerimizden birinin bunu araştırmasına izin vereceğiz.


#### Stopping a Service
```powershell-session
PS C:\htb> Stop-Service Spooler 

PS C:\htb> Get-Service Spooler 

Status   Name               DisplayName
------   ----               -----------
Stopped  spooler            Totally still used for Print Spooli...
```

Şimdi Stop-Service kullanarak Spooler servisinin çalışma durumunu durdurduğumuzu görebiliriz. Artık servisi durdurduğumuza göre, daha fazla araştırma yapılana kadar servisin başlangıç türünü Otomatik'ten Devre Dışı'na ayarlayalım.

#### Set-Service
```powershell-session
PS C:\htb> get-service spooler | Select-Object -Property Name, StartType, Status, DisplayName

Name    StartType  Status DisplayName
----    ---------  ------ -----------
spooler Automatic Stopped Totally still used for Print Spooling...


PS C:\htb> Set-Service -Name Spooler -StartType Disabled

PS C:\htb> Get-Service -Name Spooler | Select-Object -Property StartType 

StartType
---------
 Disabled
```

Tamam, şimdi Spooler servisimiz durduruldu ve Startup'ı şimdilik Disabled olarak değiştirildi. Çalışan bir servisi değiştirmek oldukça basittir. Herhangi bir değişiklik yapmaya çalışırsanız, host veya domain için bir Administrator olduğunuzdan emin olun. PowerShell'de servisleri kaldırmak şu anda zordur. Remove-Service cmdlet'i yalnızca PowerShell sürüm 7 kullanıyorsanız çalışır. Varsayılan olarak, hostlarımız PowerShell sürüm 5.1'i açacak ve çalıştıracaktır. Şimdilik, bir servisi ve girdilerini kaldırmak istiyorsanız ==sc.exe== aracını kullanın.


### PowerShell Kullanarak Remote Services ile Nasıl Etkileşim Kurarız?
Artık servislerle nasıl çalışacağımızı bildiğimize göre, remote hostlarla nasıl etkileşim kurabileceğimize bakalım. Bay Tanaka'nın host'u bir domain içinde olduğu için, diğer host'lardaki çalışan servisleri kolayca sorgulayabilir ve kontrol edebiliriz. ==-ComputerName== parametresi, remote bir host'u sorgulamak istediğimizi belirtmemizi sağlar.



### Remotely Sorgulama Servisleri
```powershell-session
PS C:\htb> get-service -ComputerName ACADEMY-ICL-DC

Status   Name               DisplayName
------   ----               -----------
Running  ADWS               Active Directory Web Services
Stopped  AppIDSvc           Application Identity
Stopped  AppMgmt            Application Management
Stopped  AppReadiness       App Readiness
Stopped  AppXSvc            AppX Deployment Service (AppXSVC)
Running  BFE                Base Filtering Engine
Stopped  BITS               Background Intelligent Transfer Ser...
<SNIP>  
```


#### Filtering our Output
```powershell-session
PS C:\htb> Get-Service -ComputerName ACADEMY-ICL-DC | Where-Object {$_.Status -eq "Running"}

Status   Name               DisplayName
------   ----               -----------
Running  ADWS               Active Directory Web Services
Running  BFE                Base Filtering Engine
Running  COMSysApp          COM+ System Application
Running  CoreMessagingRe... CoreMessaging
Running  CryptSvc           Cryptographic Services
Running  DcomLaunch         DCOM Server Process Launcher
Running  Dfs                DFS Namespace
Running  DFSR               DFS Replication
```

Burada dikkat edilmesi gereken ilginç bir nokta, PowerShell her şeyi bir obje olarak ele aldığından, remote bir komutun çıktısı bile olsa, bir objenin özelliklerini Where-Object ile incelemek için PowerShell pipeline'ı kullanabiliriz. Sonuçlarımız yalnızca çalıştırıldığında durumu Running (Çalışıyor) olan servisleri döndürdü. Bu kombinasyonları birçok şey için kullanabiliriz. Harika bir örnek, hosts'larımızı belirli bir özellik için sorgulamak olabilir, örneğin durumun Running olup olmadığı, DisplayName'in belirli bir şeye ayarlanıp ayarlanmadığı vb. Remote etkileşimlerle ilgili olarak ==Invoke-Command== cmdlet'ini de kullanabiliriz. Birden fazla host'u sorgulamayı ve UserManager servisinin durumunu görmeyi deneyelim.


### Invoke-Command
```powershell-session
PS C:\htb> invoke-command -ComputerName ACADEMY-ICL-DC,LOCALHOST -ScriptBlock {Get-Service -Name 'windefend'}

Status   Name               DisplayName                            PSComputerName
------   ----               -----------                            --------------
Running  windefend          Microsoft Defender Antivirus Service   LOCALHOST
Running  windefend          Windows Defender Antivirus Service     ACADEMY-ICL-DC
```


Şimdi bunu açıklayalım:
* ==Invoke-Command==: PowerShell'e lokal veya remote bir bilgisayarda bir komut çalıştırmak istediğimizi söylüyoruz.
* ==Computername==: Sorgulanacak bilgisayar adlarının virgülle tanımlanmış bir listesini sağlıyoruz.
* ==ScriptBlock {çalıştırılacak komutlar}==: Bu kısım, bilgisayarda çalıştırmak istediğimiz kapalı komuttur. Çalışması için {} içine alınmasına ihtiyacımız var.

Ev sahipleriyle bu şekilde etkileşim kurmak işimizi büyük ölçüde hızlandırabilir.

Senaryo: Bu bölümün başlarında, değiştirilmiş bir DisplayName'e sahip bir servis (Spooler) gördük. Bu, ortamımızdaki bir sorun hakkında bize ipucu verebilir. Ortamımızdaki tüm hostları sorgulamak ve DisplayName özelliklerini kontrol etmek için -ComputerName parametresini veya Invoke-Command cmdlet'ini kullanarak başka bir hostun etkilenip etkilenmediğini görebiliriz. Bir yönetici olarak, bu tür bir güce erişmek paha biçilmezdir ve genellikle bir tehdidin hostda bulunduğu süreyi azaltmaya ve sorunun önüne geçmeye ve tehdidi ortadan kaldırmak için çalışmaya yardımcı olabilir.


Servisleri anlamak ve onları bir host üzerinde yönetmek bir yönetici ve pentester açısından çok önemlidir. Onlarla ayrıcalık yükseltmeden kalıcılığa ve daha fazlasına kadar pek çok şey yapabiliriz. Bir sonraki bölümde, Windows Registry'yi ve bir Windows hostunda onunla nasıl etkileşim kuracağımızı tanıtacağız.

Soru : Hangi Cmdlet bize bir hostun mevcut hizmetlerini gösterir?
Cevap : Get-Service 

Soru : Windows Defender Hizmetini başlatmak istersek hangi komutu kullanırız?
Cevap : Start-Service WinDefend

Soru : Hangi Cmdlet uzaktaki bir hostta bir komut çalıştırmamıza izin verir? 
Cevap : invoke-command


### Registry ile Çalışma

Bu noktada CLI ile rahat olmalıyız. Becerilerimizi bir üst seviyeye çıkarmanın ve Windows işletim sisteminin daha karmaşık yönlerinden biri olan Registry'yi ele almanın zamanı geldi. Bu bölümde Registry'nin ne olduğu, Registry'de nasıl gezinileceği ve key/value çiftlerinin nasıl okunacağı ve gerektiğinde nasıl değişiklik yapılacağı anlatılacaktır.


### Windows Registry Nedir?
Registry, özünde iki temel unsur içeren hiyerarşik bir ağaç olarak düşünülebilir: Key'ler ve Value'lar. Bu ağaç, işletim sistemi ve alt ağaçlar altında çalışmak üzere yüklenen yazılımlar için gerekli tüm bilgileri depolar (bunları bir ağacın dalları olarak düşünün). Bu bilgiler, ayarlardan kurulum dizinlerine ve her şeyin nasıl çalışacağını belirleyen belirli seçeneklere ve değerlere kadar her şey olabilir. Pentesters olarak Registry yararlı bilgiler, tesis kalıcılığı ve daha fazlasını bulmak için harika bir noktadır. [MITRE](https://attack.mitre.org/techniques/T1112/), bir tehdit aktörünün bir hostun registry kovanına erişimiyle ( lokal ya da uzaktan) neler yapabileceğine dair birçok harika örnek sunmaktadır.

### Key Nedir ? 
Keys, özünde, PC'nin belirli bir bileşenini temsil eden konteynerlerdir. Key'ler veri olarak başka key'ler ve değerler içerebilir. Bu girdiler birçok şekilde olabilir ve adlandırma bağlamları yalnızca bir Key'in alfanümerik (yazdırılabilir) karakterler kullanılarak adlandırılmasını gerektirir ve büyük/küçük harfe duyarlı değildir. Anahtarlara görsel bir örnek olarak, aşağıdaki resme bakarsak, Yeşil dikdörtgen içindeki her klasör bir Anahtardır ve alt anahtarlar içerir.

#### Keys (Green)
![Pasted image 20241013224113.png](/img/user/resimler/Pasted%20image%2020241013224113.png)


### Registry Key Files
Bir host sisteminin Registry ==root anahtarları== birkaç farklı dosyada saklanır ve ==C:\Windows\System32\Config==\ adresinden erişilebilir. Bu Key dosyalarının yanı sıra, registry kovanları host boyunca çeşitli diğer yerlerde tutulur.

#### Root Registry Keys
```powershell-session
PS C:\htb> Get-ChildItem C:\Windows\System32\config\

    Directory: C:\Windows\System32\config

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d----           12/7/2019  4:14 AM                Journal
d----           12/7/2019  4:14 AM                RegBack
d----           4/28/2021 11:43 AM                systemprofile
d----           9/18/2021 12:22 AM                TxR
-a---          10/12/2022 10:06 AM         786432 BBI
-a---           1/20/2021  5:13 PM          28672 BCD-Template
-a---          10/18/2022 11:14 AM       38273024 COMPONENTS
-a---          10/12/2022 10:06 AM        1048576 DEFAULT
-a---          10/15/2022  9:33 PM       13463552 DRIVERS
-a---           1/27/2021  2:54 PM          32768 ELAM
-a---          10/12/2022 10:06 AM         131072 SAM
-a---          10/12/2022 10:06 AM          65536 SECURITY
-a---          10/12/2022 10:06 AM      168034304 SOFTWARE
-a---          10/12/2022 10:06 AM       29884416 SYSTEM
-a---          10/12/2022 10:06 AM           1623 VSMIDK
```
Tüm Registry Kovanlarının ve işletim sistemi içindeki destekleyici dosyalarının ayrıntılı bir listesi için [BURAYA](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-hives) bakabiliriz. Şimdi Registry içindeki Values (Değerler) konusunu ele alalım.


### Values Nedir ? 

Values (Değerler), belirli bir Key (Anahtar) ile ilgili objeler biçimindeki verileri temsil eder. Bu değerler bir ad, bir tür belirtimi ve ne için olduğunu tanımlamak için gerekli verilerden oluşur. Aşağıdaki görüntü, Values'i (Değerler) Turuncu çizgiler arasındaki veriler olarak görsel olarak temsil etmektedir. Bu değerler Installer anahtarının içine yerleştirilmiştir ve bu anahtar da başka bir anahtarın içindedir.

#### Values
![Pasted image 20241013224415.png](/img/user/resimler/Pasted%20image%2020241013224415.png)
Registry Key Values (Kayıt Defteri Anahtarı Değerleri) listesinin tamamına [BURADAN](https://learn.microsoft.com/en-us/windows/win32/sysinfo/registry-value-types) ulaşabilirsiniz. Toplamda, yapılandırılabilecek 11 farklı değer türü vardır.


### Registry Hives (Kovan)
Her Windows host'u, host'u ve kullanım için gerekli ayarları koruyan bir dizi önceden tanımlanmış Registry anahtarına sahiptir. Aşağıda her bir kovanın ve içinde referans olarak nelerin bulunabileceğinin bir dökümü yer almaktadır.


### Hive Breakdown

![Pasted image 20241013224653.png](/img/user/resimler/Pasted%20image%2020241013224653.png)
[HKLM](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2003/cc739525(v=ws.10)) 

Registry için önceden tanımlanmış başka anahtarlar da vardır, ancak bunlar Windows'un belirli sürümlerine ve bölgesel ayarlarına özgüdür. Bu girdiler ve genel olarak Registry anahtarları hakkında daha fazla bilgi için [Microsoft](https://learn.microsoft.com/en-us/windows/win32/sysinfo/predefined-keys) tarafından sağlanan belgelere göz atın

### Registry'de Saklanan Bilgiler Neden Önemlidir?
Bir pentester olarak Registry, görevlerimizi ilerletmemize yardımcı olabilecek bir bilgi hazinesi olabilir. Hangi yazılımın yüklü olduğu, mevcut işletim sistemi revizyonu, ilgili güvenlik ayarları, Defender'ın kontrolü ve daha fazlasına kadar her şey Registry'de bulunabilir. Tüm bu bilgileri başka yerlerde de bulabilir miyiz? Evet. Ancak tüm bunları bulmak ve aynı anda ana bilgisayarda geniş çaplı değişiklikler yapmak için daha iyi bir tek nokta yoktur. Saldırı perspektifinden bakıldığında, Kayıt Defteri Savunucular için korunması zor bir yerdir. Kovanlar çok büyüktür ve yüzlerce girişle doludur. Kovanlar arasında tekil bir değişiklik ya da ekleme bulmak samanlıkta iğne aramaya benzer (eğer yapılandırmalarının ve host durumlarının sağlam yedeklerini tutmuyorlarsa). Registry ve anahtar değerlerin nerede olduğu hakkında genel bir anlayışa sahip olmak, daha hızlı harekete geçmemize ve savunucular için herhangi bir sorunu daha erken tespit etmemize yardımcı olabilir.

### Bilgiye Nasıl Ulaşırız?
CLI'dan Registry'ye erişmek ve key'lerimizi yönetmek için birkaç seçeneğimiz var. Bunlardan ilki [reg.exe](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/reg) kullanmaktır. Reg, Registry ayarlarını yönetmek için özel olarak hazırlanmış bir dos çalıştırılabiliridir. İkincisi, Key ve değerleri okumak için Get-Item ve Get-ItemProperty cmdlet'lerini kullanmaktır. Eğer bir değişiklik yapmak istersek, New-ItemProperty kullanımı işimizi görecektir.


### Registry Girdilerini Sorgulama
İlk olarak ==Get-Item== ve ==Get-ChildItem== kullanımına bakacağız. Aşağıda Get-Item kullanımının çıktısını görebilir ve sonucu Select-Object'e aktarabiliriz.


#### Get-Item
```powershell-session
PS C:\htb> Get-Item -Path Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run | Select-Object -ExpandProperty Property  

SecurityHealth
RtkAudUService
WavesSvc
DisplayLinkTrayApp
LogiOptions
Acrobat Assistant 8.0
(default)
Focusrite Notifier
AdobeGCInvoker-1.0
```
Bu basit bir çıktıdır ve bize yalnızca o anda çalışan servislerin/uygulamaların adını gösterir. Bir kovan içindeki her bir key ve objeyi  görmek istersek, ==Get-ChildItem'ı -Recurse== parametresiyle birlikte aşağıdaki gibi de kullanabiliriz:


### Recursive Search
```powershell-session
PS C:\htb> Get-ChildItem -Path HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion -Recurse

Hive: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths
<SNIP>
Name                           Property
----                           --------
7zFM.exe                       (default) : C:\Program Files\7-Zip\7zFM.exe
                               Path      : C:\Program Files\7-Zip\
Acrobat.exe                    (default) : C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrobat.exe
                               Path      : C:\Program Files\Adobe\Acrobat DC\Acrobat\
AcrobatInfo.exe                (default) : C:\Program Files\Adobe\Acrobat DC\Acrobat\AcrobatInfo.exe
                               Path      : C:\Program Files\Adobe\Acrobat DC\Acrobat\
AcroDist.exe                   Path      : C:\Program Files\Adobe\Acrobat DC\Acrobat\
                               (default) : C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrodist.exe
Ahk2Exe.exe                    (default) : C:\Program Files\AutoHotkey\Compiler\Ahk2Exe.exe
AutoHotkey.exe                 (default) : C:\Program Files\AutoHotkey\AutoHotkey.exe
chrome.exe                     (default) : C:\Program Files\Google\Chrome\Application\chrome.exe
                               Path      : C:\Program Files\Google\Chrome\Application
cmmgr32.exe                    CmNative          : 2
                               CmstpExtensionDll : C:\Windows\System32\cmcfg32.dll
CNMNSST.exe                    (default) : C:\Program Files (x86)\Canon\IJ Network Scanner Selector EX\CNMNSST.exe
                               Path      : C:\Program Files (x86)\Canon\IJ Network Scanner Selector EX
devenv.exe                     (default) : "C:\Program Files\Microsoft Visual
                               Studio\2022\Community\common7\ide\devenv.exe"
dfshim.dll                     UseURL : 1
excel.exe                      (default) : C:\Program Files\Microsoft Office\Root\Office16\EXCEL.EXE
                               Path      : C:\Program Files\Microsoft Office\Root\Office16\
                               UseURL    : 1
                               SaveURL   : 1
fsquirt.exe                    DropTarget : {047ea9a0-93bb-415f-a1c3-d7aeb3dd5087}
IEDIAG.EXE                     (default) : C:\Program Files\Internet Explorer\IEDIAGCMD.EXE
                               Path      : C:\Program Files\Internet Explorer;
IEDIAGCMD.EXE                  (default) : C:\Program Files\Internet Explorer\IEDIAGCMD.EXE
                               Path      : C:\Program Files\Internet Explorer;
IEXPLORE.EXE                   (default) : C:\Program Files\Internet Explorer\IEXPLORE.EXE
                               Path      : C:\Program Files\Internet Explorer;
install.exe                    BlockOnTSNonInstallMode : 1
javaws.exe                     (default) : C:\Program Files\Java\jre1.8.0_341\bin\javaws.exe
                               Path      : C:\Program Files\Java\jre1.8.0_341\bin
licensemanagershellext.exe     (default) : C:\Windows\System32\licensemanagershellext.exe
mip.exe                        (default) : C:\Program Files\Common Files\Microsoft Shared\Ink\mip.exe
mpc-hc64.exe                   (default) : C:\Program Files (x86)\K-Lite Codec Pack\MPC-HC64\mpc-hc64.exe
                               Path      : C:\Program Files (x86)\K-Lite Codec Pack\MPC-HC64
mplayer2.exe                   (default) : "C:\Program Files\Windows Media Player\wmplayer.exe"
                               Path      : C:\Program Files\Windows Media Player
MSACCESS.EXE                   (default) : C:\Program Files\Microsoft Office\Root\Office16\MSACCESS.EXE
                               Path      : C:\Program Files\Microsoft Office\Root\Office16\
                               UseURL    : 1
msedge.exe                     (default) : C:\Program Files (x86)\Microsoft\Edge\Application\msedge.exe
                               Path      : C:\Program Files (x86)\Microsoft\Edge\Application

    Hive: HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\App Paths\msedge.exe

Name                           Property
----                           --------
SupportedProtocols             http  :
                               https :
<SNIP>  
```

Şimdi, HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion key içindeki her bir key'i ve ilişkili value'ları genişlettiği ve gösterdiği için çıktıyı kısalttık. Get-ItemProperty cmdlet'ini kullanarak çıktımızı daha kolay okunur hale getirebiliriz. Aynı sorguyu Get-ItemProperty ile deneyelim.,


#### Get-ItemProperty
```powershell-session
PS C:\htb> Get-ItemProperty -Path Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run

SecurityHealth        : C:\Windows\system32\SecurityHealthSystray.exe
RtkAudUService        : "C:\Windows\System32\DriverStore\FileRepository\realtekservice.inf_amd64_85cff5320735903
                        d\RtkAudUService64.exe" -background
WavesSvc              : "C:\Windows\System32\DriverStore\FileRepository\wavesapo9de.inf_amd64_d350b8504310bbf5\W
                        avesSvc64.exe" -Jack
DisplayLinkTrayApp    : "C:\Program Files\DisplayLink Core Software\DisplayLinkTrayApp.exe" -basicMode
LogiOptions           : C:\Program Files\Logitech\LogiOptions\LogiOptions.exe /noui
Acrobat Assistant 8.0 : "C:\Program Files\Adobe\Acrobat DC\Acrobat\Acrotray.exe"
(default)             :
Focusrite Notifier    : "C:\Program Files\Focusriteusb\Focusrite Notifier.exe"
AdobeGCInvoker-1.0    : "C:\Program Files (x86)\Common Files\Adobe\AdobeGCClient\AGCInvokerUtility.exe"
PSPath                : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Curren
                        tVersion\Run
PSParentPath          : Microsoft.PowerShell.Core\Registry::HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\Curren
                        tVersion
PSChildName           : Run
PSProvider            : Microsoft.PowerShell.Core\Registry
```
Şimdi bunu parçalara ayıralım. Get-ItemProperty komutunu verdik, yolu Registry'ye bakmak olarak belirttik ve HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run anahtarını belirttik. Çıktı bize başlatılan servislerin adını ve onları çalıştırmak için kullanılan değeri (çalıştırıldıkları yol) sağlar. Bu Registry key, bir kullanıcı hostta oturum açtığında hizmetleri/uygulamaları başlatmak için kullanılır. Bir sızma test uzmanı olarak görünürlüğe sahip olmak ve akılda tutmak için harika bir anahtardır. Bu anahtarın bu bölümde biraz sonra tartışacağımız birkaç versiyonu vardır. Get-ItemProperty kullanmak Get-Item kullanmaktan çok daha okunabilirdir. Bilgi sorgulama söz konusu olduğunda Reg.exe'yi de kullanabiliriz. Şimdi bunun çıktısına bir göz atalım. Bu örnek için daha basit bir key'e bakacağız.

#### Reg.exe
```powershell-session
PS C:\htb> reg query HKEY_LOCAL_MACHINE\SOFTWARE\7-Zip

HKEY_LOCAL_MACHINE\SOFTWARE\7-Zip
    Path64    REG_SZ    C:\Program Files\7-Zip\
    Path    REG_SZ    C:\Program Files\7-Zip\
```
HKEY_LOCAL_MACHINE\SOFTWARE\7-Zip anahtarını Reg.exe ile sorguladık, bu da bize ilişkili değerleri sağladı. Path ve Path64 olmak üzere iki değerin ayarlandığını, ValueType'ın bir Unicode veya ASCII dizesi içerdiğini belirten bir Reg_SZ değeri olduğunu ve bu değerin 7-Zip C:\Program Files\7-Zip\'in yolu olduğunu görebiliriz.


### Registry'de Bilgi Bulma
Pentesterlar ve yöneticiler olarak bizler için Registry'de veri bulmak olmazsa olmaz bir beceridir. Reg.exe'nin bizim için gerçekten parladığı yer burasıdır. Key ve value isimleri ya da içerdiği veriler aracılığıyla Password ve Username gibi anahtar kelimeleri ve stringleri aramak için kullanabiliriz. Kullanmaya başlamadan önce, Reg Query'nin kullanımını inceleyelim. REG QUERY HKCU /F “password” /t REG_SZ /S /K komut dizisine bakacağız.

* ==Reg query==: Reg.exe'yi çağırıyoruz ve verileri sorgulamak istediğimizi belirtiyoruz.
* ==HKCU==: Bu kısım arama yapılacak yolu ayarlıyor. Bu örnekte, HKey_Current_User'ın tamamına bakıyoruz.
* ==/f “password”==: /f aradığımız kalıbı ayarlar. Bu örnekte, “ Password” arıyoruz.
* ==/t REG_SZ==: /t aranacak value türünü ayarlıyor. Eğer belirtmezsek, reg sorgusu her tipte arama yapacaktır.
* ==/s==: /s tüm alt anahtarları ve değerleri özyinelemeli olarak aramayı söyler.
* ==/k==: /k sadece Anahtar isimleri üzerinden arama yapacak şekilde daraltır.



### Reg Sorgusu ile Arama
```powershell-session
PS C:\htb>  REG QUERY HKCU /F "Password" /t REG_SZ /S /K

HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\Winlogon\PasswordExpiryNotification
    NotShownErrorTime    REG_SZ    08::23::24, 2022/10/19
    NotShownErrorReason    REG_SZ    GetPwdResetInfoFailed

End of search: 2 match(es) found.
```

Bu sorgudan elde ettiğimiz sonuçlar daha heyecan verici olabilir, ancak yine de bir göz atmaya ve Username, Credentials ve Keys gibi diğer anahtar kelimeler ve ifadeler için benzer bir arama yapmaya değer. Bulduklarımız bizi şaşırtabilir. Gördüğümüz gibi, registry anahtarlarını ve değerlerini sorgulamak nispeten kolaydır. Peki ya yeni bir value ayarlamak ya da yeni bir key oluşturmak istersek?


### Registry Keys and Values Oluşturma ve Değiştirme
Yeni key ve value'ların değiştirilmesi veya oluşturulması ile uğraşırken, New-Item, Set-Item, New-ItemProperty ve Set-ItemProperty gibi standart PowerShell cmdlet'lerini kullanabilir veya ihtiyacımız olan değişiklikleri yapmak için yine Reg.exe'den yararlanabiliriz. Aşağıda yeni bir Registry Key oluşturmayı deneyelim. Örneğimiz için, RunOnce kovanında HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce içinde TestKey adında yeni bir test key'i oluşturacağız. Key ve value RunOnce'a yerleştirildiğinde, çalıştırıldıktan sonra silinecektir.

==Senaryo==: Bir host'a yerleştik ve kalıcılık için yeni bir registry anahtarı ekleyebiliriz. TestKey adında bir anahtar ve C:\Users\htb-student\Downloads\payload.exe değerini ayarlamamız gerekir; bu değer RunOnce'a, kullanıcı bir sonraki oturum açışında host üzerinde bıraktığımız payload'umuzu çalıştırmasını söyler. Bu, host yeniden başlatılırsa veya erişimi kaybedersek, kullanıcı bir sonraki oturum açışında yeni bir shell almamızı sağlayacaktır.


#### New Registry Key
```powershell-session
PS C:\htb> New-Item -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\ -Name TestKey

    Hive: HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce

Name                           Property
----                           --------
TestKey   
```

Artık RunOnce key içinde yeni bir key'imiz var. Path parametresini belirterek, shell'deki konumumuzu Registry'de bir key eklemek istediğimiz yere değiştirmekten kaçınıyoruz ve mutlak yolu belirttiğimiz sürece herhangi bir yerden çalışmamıza izin veriyoruz. Şimdi bir Property ve bir değer belirleyelim.


### New Registry Item Özelliğini Ayarla
```powershell-session
PS C:\htb>  New-ItemProperty -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\TestKey -Name  "access" -PropertyType String -Value "C:\Users\htb-student\Downloads\payload.exe"

access       : C:\Users\htb-student\Downloads\payload.exe
PSPath       : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\
               TestKey
PSParentPath : Microsoft.PowerShell.Core\Registry::HKEY_CURRENT_USER\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce
PSChildName  : TestKey
PSDrive      : HKCU
PSProvider   : Microsoft.PowerShell.Core\Registry
```
Access adlı değerimizi ayarlamak için New-ItemProperty'yi kullandıktan ve değeri C:\Users\htb-student\Downloads\payload.exe olarak belirttikten sonra, sonuçlarda değerimizin başarıyla oluşturulduğunu ve yol konumu ve Key adı gibi ilgili bilgileri görebiliriz. Key'imizin oluşturulduğunu göstermek için, GUI Registry editöründen aşağıdaki resimde new key'i ve değerlerini görebiliriz.

![Pasted image 20241013230139.png](/img/user/resimler/Pasted%20image%2020241013230139.png)

Aynı key/value çiftini Reg.exe kullanarak eklemek isteseydik, bunu şu şekilde yapardık:

```powershell
reg add "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce\TestKey" /v access /t REG_SZ /d "C:\Users\htb-student\Downloads\payload.exe"  
```

Şimdi gerçek bir pentestte, host üzerinde çalıştırılabilir bir payload bırakırdık ve hostun yeniden başlatılması veya kullanıcının oturum açması durumunda, C2'mize yeni bir shell elde ederdik. Bu değer şu anda bizim için pek bir şey ifade etmiyor, bu yüzden onu silme alıştırması yapalım.


### Reg özelliklerini silme

```powershell-session
PS C:\htb> Remove-ItemProperty -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\TestKey -Name  "access"

PS C:\htb> Get-ItemProperty -Path HKCU:\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\TestKey
```

Herhangi bir hata penceresi açılmadıysa, anahtar/değer çiftimiz başarıyla silinmiştir. Ancak, bu son derece dikkatli olmanız gereken şeylerden biridir. Windows Kayıt Defteri'nden girdileri kaldırmak host'u ve nasıl çalıştığını olumsuz etkileyebilir. Daha önce neyi kaldırdığınızı bildiğinizden emin olun. Ben Amca'nın bilge sözleriyle, “Büyük güç büyük sorumluluk getirir.”


### İleriye doğru
Artık Kayıt Defteri yönetimini öğrendiğimize göre, PowerShell aracılığıyla Olay Günlüklerini işlemeye geçme zamanı geldi.


Soru : Bir kayıt defteri girdisi iki parçadan oluşur, bir 'Anahtar' ve ' ' . İkinci parça nedir?
Cevpa : values

Soru : "HKey_Current_User” için kısaltma nedir."
Cevap : HKCU




### Windows Event Log ile Çalışma
Bir SOC Analisti veya BT Yöneticisinin bakış açısından, ağdaki tüm makinelerde meydana gelen olayları izlemek, toplamak ve kategorize etmek, ağlarını proaktif olarak analiz eden ve şüpheli faaliyetlerden koruyan savunucular için paha biçilmez bir bilgi kaynağıdır. Öte yandan, saldırganlar bunu hedef ortam hakkında bilgi edinmek, bilgi akışını bozmak ve izlerini gizlemek için bir fırsat olarak görebilirler. Daha sonraki modüllerde göreceğimiz gibi, bazen bir sızma testi sırasında tehlikeye attığımız bir hedef sistem olarak olay günlüklerinde saklanan kimlik bilgileri gibi ilginç bilgiler bulabiliriz. Diğer zamanlarda, olay günlüklerini numaralandırmak ortamdaki günlük kaydı seviyesini anlamamıza yardımcı olabilir (sadece varsayılanlar mı mevcut, yoksa hedef kuruluş daha ayrıntılı günlük kaydı yapılandırdı mı?) Bu bölümde aşağıdakileri tartışacağız:

* Windows Event Log nedir?
* Hangi bilgileri günlüğe kaydeder ve bu bilgileri nerede saklar?
* wevtutil komut satırı yardımcı programı aracılığıyla Event Log ile etkileşim kurma
* PowerShell cmdlet'lerini kullanarak Event Log ile etkileşim kurma

## What is the Windows Event Log?

Event Log'un net bir şekilde anlaşılması bilgi güvenliğinde başarı için çok önemlidir. Windows Event Log'u kapsamlı bir şekilde anlama yolculuğumuza başlamak için, dalmadan önce tanımlamamız gereken birkaç temel kavram vardır. Bu kavramlar, diğer her şeyin üzerine inşa edileceği temel olacaktır. Açıklanması gereken ilk kavram “event” tanımıdır. Basitçe ifade etmek gerekirse event, bir sistemin donanımı veya yazılımı tarafından tanımlanabilen ve sınıflandırılabilen herhangi bir eylem veya oluşumdur. Event'lar aşağıdakilerden bazıları da dahil olmak üzere çeşitli farklı yollarla oluşturulabilir veya tetiklenebilir:

User Tarafından Oluşturulan Etkinlikler
* Bir farenin hareketi, bir klavyede yazma, diğer kullanıcı kontrollü çevre birimleri vb.

Uygulama Tarafından Oluşturulan Olaylar
* Uygulama güncellemeleri, çökmeler, bellek kullanımı/tüketimi vb.

Sistem Tarafından Oluşturulan Olaylar
* Sistem çalışma süresi, sistem güncellemeleri, sürücü yükleme/boşaltma, kullanıcı girişi vb.

Çeşitli kaynaklardan farklı zaman aralıklarında meydana gelen bu kadar çok event varken, bir Windows sistemi bunların hepsini nasıl takip eder ve kategorize eder? İşte olay günlüğü olarak bilinen ikinci anahtar kavramımız burada devreye giriyor.

Microsoft tarafından tanımlanan [Event Logging](https://learn.microsoft.com/en-us/windows/win32/eventlog/event-logging):

==“...uygulamaların (ve işletim sisteminin) önemli yazılım ve donanım olaylarını kaydetmesi için standart, merkezi bir yol sağlar.”==

Bu tanım soruyu oldukça güzel özetliyor. Ancak, bunu biraz daha açmaya çalışalım. Daha önce de tartıştığımız gibi, bir sistemde eş zamanlı olarak tetiklenen veya üretilen çok sayıda olay vardır. Her olayın, olayın arkasındaki bilgileri ve ayrıntıları kendi formatında sağlayan kendi kaynağı olacaktır. Peki tüm bu bilgiler nasıl işlenir?

Windows bu sorunu, Windows Event Log olarak bilinen bir servis aracılığıyla event ve event bilgilerini kaydetmek, saklamak ve yönetmek için standartlaştırılmış bir yaklaşım sağlayarak çözmeye çalışır. Adından da anlaşılacağı gibi Event Log olayları ve event logları yönetir, ancak bu fonksiyonelliğe ek olarak uygulamaların kendi ayrı loglarını tutmalarına ve yönetmelerine olanak tanıyan özel bir API de açar. Windows Temelleri modülünde, günlüklerdeki hizmetleri Windows Servisleri ve Prosesleri bölümünde daha ayrıntılı olarak ele aldık, ancak Event Log'un sistemin başlatılmasıyla başlayan ve kendi başına değil başka bir yürütülebilir dosya bağlamında çalışan gerekli bir Windows servisi olduğunu anlamak önemlidir.


### Event Log Kategorileri ve Türleri
Dört ana log kategorisi uygulama, security, setup ve system'dir. İletilen olaylar adı verilen başka bir kategori türü de mevcuttur.
![Pasted image 20241014093621.png](/img/user/resimler/Pasted%20image%2020241014093621.png)


## Event Types
Windows sistemlerinde günlüğe kaydedilebilecek beş tür olay vardır:
![Pasted image 20241014093728.png](/img/user/resimler/Pasted%20image%2020241014093728.png)


### Event Önem Seviyeleri
Her log, bir sayı ile gösterilen beş önem düzeyinden birine sahip olabilir:
![Pasted image 20241014093853.png](/img/user/resimler/Pasted%20image%2020241014093853.png)


### Windows Event Log Unsurları
Windows Event Log (Windows Olay Günlüğü), bir Windows sistemindeki donanım ve yazılım olayları hakkında bilgi sağlar. Tüm event log'lar standart bir formatta saklanır ve aşağıdaki unsurları içerir:

* ==Log name (Günlük adı)==: Yukarıda belirtildiği gibi, olayların yazılacağı logun adıdır. Varsayılan olarak, olaylar system, security ve uygulamalar için loglanır.
* ==Event date/time==: Olayın gerçekleştiği tarih ve saat
* ==Task Category==: Kaydedilen olay günlüğünün türü
* ==Event ID==: Sistem yöneticilerinin günlüğe kaydedilen belirli bir olayı tanımlaması için benzersiz bir tanımlayıcı
* ==Source (Kaynak):== Günlüğün nereden kaynaklandığı, tipik olarak bir programın veya yazılım uygulamasının adı
* ==Level (Seviye)==: Olayın önem düzeyi. Bu bilgi, hata, ayrıntılı, uyarı, kritik olabilir
* ==User==: Olay gerçekleştiğinde hostta oturum açan kişinin kullanıcı adı
* ==Computer (Bilgisayar)==: Olayın kaydedildiği bilgisayarın adı

Bir kuruluşun çeşitli sorunları tespit etmek için izleyebileceği birçok Event ID vardır. Active Directory ortamında,[ bu liste](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/appendix-l--events-to-monitor) bir tehlikenin işaretlerini aramak için izlenmesi önerilen önemli olayları içerir. Bu aranabilir Event ID veritabanı, bir Windows sisteminde mümkün olan günlük kaydının derinliğini anlamak için [incelenmeye](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/) değerdir.


### Windows Event Log Teknik Detayları
Windows Event Log EventLog servisleri tarafından yönetilir. Bir Windows sisteminde, servisin görünen adı Windows Event Log'dur ve servis host prosesi [svchost.exe](https://en.wikipedia.org/wiki/Svchost.exe) içinde çalışır. Varsayılan olarak sistem açılışında otomatik olarak başlayacak şekilde ayarlanmıştır. Birden fazla bağımlılık servisi olduğu için EventLog servisini durdurmak zordur. Durdurulursa, büyük olasılıkla önemli sistem kararsızlığına neden olacaktır. Varsayılan olarak, Windows Event Logs ==.evtx== dosya uzantısı ile ==C:\Windows\System32\winevt\logs== içinde saklanır.


```powershell-session
PS C:\htb> ls C:\Windows\System32\winevt\logs

    Directory: C:\Windows\System32\winevt\logs


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/16/2022   2:19 PM        7409664 Application.evtx
-a----         6/14/2022   8:20 PM          69632 HardwareEvents.evtx
-a----         6/14/2022   8:20 PM          69632 Internet Explorer.evtx
-a----         6/14/2022   8:20 PM          69632 Key Management Service.evtx        
-a----         8/23/2022   7:01 PM          69632 Microsoft-Client-License-Flexible-P
                                                  latform%4Admin.evtx
-a----        11/16/2022   2:19 PM        1052672 Microsoft-Client-Licensing-Platform
                                                  %4Admin.evtx


<SNIP>
```

[Windows Event Viewer](https://en.wikipedia.org/wiki/Event_Viewer) GUI uygulamasını kullanarak komut satırı yardımcı programı [wevtutil](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) veya [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.4&viewFallbackFrom=powershell-7.3) PowerShell cmdlet'ini kullanarak Windows Event log ile etkileşime geçebiliriz. Hem wevtutil hem de Get-WinEvent, cmd.exe veya PowerShell aracılığıyla hem lokal hem de remote Windows sistemlerindeki Event Log'ları sorgulamak için kullanılabilir.


### Windows Event Log ile Etkileşim - wevtutil
[Wevtutil](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/wevtutil) komut satırı yardımcı programı, event logları hakkında bilgi almak için kullanılabilir. Diğer komutların yanı sıra günlükleri dışa aktarmak, arşivlemek ve temizlemek için de kullanılabilir.


### Parametreler olmadan Wevtutil
```cmd-session
C:\htb> wevtutil /?

Windows Events Command Line Utility.

Enables you to retrieve information about event logs and publishers, install
and uninstall event manifests, run queries, and export, archive, and clear logs.

Usage:

You can use either the short (for example, ep /uni) or long (for example,
enum-publishers /unicode) version of the command and option names. Commands,
options and option values are not case-sensitive.

Variables are noted in all upper-case.

wevtutil COMMAND [ARGUMENT [ARGUMENT] ...] [/OPTION:VALUE [/OPTION:VALUE] ...]

Commands:

el | enum-logs          List log names.
gl | get-log            Get log configuration information.
sl | set-log            Modify configuration of a log.
ep | enum-publishers    List event publishers.
gp | get-publisher      Get publisher configuration information.
im | install-manifest   Install event publishers and logs from manifest.
um | uninstall-manifest Uninstall event publishers and logs from manifest.
qe | query-events       Query events from a log or log file.
gli | get-log-info      Get log status information.
epl | export-log        Export a log.
al | archive-log        Archive an exported log.
cl | clear-log          Clear a log.

<SNIP>
```

Bir Windows sisteminde bulunan tüm logların isimlerini listelemek için "==el==" parametresini kullanabiliriz.


#### Enumerating Log Sources
```cmd-session
C:\htb> wevtutil el

AMSI/Debug
AirSpaceChannel
Analytic
Application
DirectShowFilterGraph
DirectShowPluginControl
Els_Hyphenation/Analytic
EndpointMapper
FirstUXPerf-Analytic
ForwardedEvents
General Logging
HardwareEvents

<SNIP>
```

==gl== parametresiyle, belirli bir log için yapılandırma bilgilerini, özellikle logun etkin olup olmadığını, maksimum boyutu, izinleri ve logun sistemde nerede depolandığını görüntüleyebiliriz.


#### Log Bilgilerinin Toplanması
```cmd-session
C:\htb> wevtutil gl "Windows PowerShell"

name: Windows PowerShell
enabled: true
type: Admin
owningPublisher:
isolation: Application
channelAccess: O:BAG:SYD:(A;;0x2;;;S-1-15-2-1)(A;;0x2;;;S-1-15-3-1024-3153509613-960666767-3724611135-2725662640-12138253-543910227-1950414635-4190290187)(A;;0xf0007;;;SY)(A;;0x7;;;BA)(A;;0x7;;;SO)(A;;0x3;;;IU)(A;;0x3;;;SU)(A;;0x3;;;S-1-5-3)(A;;0x3;;;S-1-5-33)(A;;0x1;;;S-1-5-32-573)
logging:
  logFileName: %SystemRoot%\System32\Winevt\Logs\Windows PowerShell.evtx
  retention: false
  autoBackup: false
  maxSize: 15728640
publishing:
  fileMax: 1
```

==gli== parametresi bize günlük veya günlük dosyası hakkında oluşturma zamanı, son erişim ve yazma zamanları, dosya boyutu, günlük kayıtlarının sayısı ve daha fazlası gibi belirli durum bilgileri verecektir.

```cmd-session
C:\htb> wevtutil gli "Windows PowerShell"

creationTime: 2020-10-06T16:57:38.617Z
lastAccessTime: 2022-10-26T19:05:21.533Z
lastWriteTime: 2022-10-26T19:05:21.533Z
fileSize: 11603968
attributes: 32
numberOfLogRecords: 9496
oldestRecordNumber: 1
```

Eventler için sorgulama yapabileceğimiz birçok yol vardır. Örneğin, Güvenlik logundaki en son 5 olayı metin biçiminde görüntülemek istediğimizi varsayalım. Bu komut için local admin erişimi gereklidir.


### Event'leri Sorgulama
```cmd-session
C:\htb> wevtutil qe Security /c:5 /rd:true /f:text

Event[0]
  Log Name: Security
  Source: Microsoft-Windows-Security-Auditing
  Date: 2022-11-16T14:54:13.2270000Z
  Event ID: 4799
  Task: Security Group Management
  Level: Information
  Opcode: Info
  Keyword: Audit Success
  User: N/A
  User Name: N/A
  Computer: ICL-WIN11.greenhorn.corp
  Description: 
A security-enabled local group membership was enumerated.

Subject:
        Security ID:            S-1-5-18
        Account Name:           ICL-WIN11$
        Account Domain:         GREENHORN
        Logon ID:               0x3E7

Group:
        Security ID:            S-1-5-32-544
        Group Name:             Administrators
        Group Domain:           Builtin

Process Information:
        Process ID:             0x56c
        Process Name:           C:\Windows\System32\svchost.exe

Event[1]
  Log Name: Security
  Source: Microsoft-Windows-Security-Auditing
  Date: 2022-11-16T14:54:13.0160000Z
  Event ID: 4672
  Task: Special Logon
  Level: Information
  Opcode: Info
  Keyword: Audit Success
  User: N/A
  User Name: N/A
  Computer: ICL-WIN11.greenhorn.corp
  Description:
Special privileges assigned to new logon.

Subject:
        Security ID:            S-1-5-21-4125911421-2584895310-3954972028-1001        
        Account Name:           htb-student
        Account Domain:         ICL-WIN11
        Logon ID:               0x8F211

Privileges:             SeSecurityPrivilege
                        SeTakeOwnershipPrivilege
                        SeLoadDriverPrivilege
                        SeBackupPrivilege
                        SeRestorePrivilege
                        SeDebugPrivilege
                        SeSystemEnvironmentPrivilege
                        SeImpersonatePrivilege
                        SeDelegateSessionUserImpersonatePrivilege

Event[2]
  Log Name: Security
  Source: Microsoft-Windows-Security-Auditing
  Date: 2022-11-16T14:54:13.0160000Z
  Event ID: 4624
  Task: Logon
  Level: Information
  Opcode: Info
  Keyword: Audit Success
  User: N/A
  User Name: N/A
  Computer: ICL-WIN11.greenhorn.corp
  Description:
An account was successfully logged on.

Subject:
        Security ID:            S-1-5-18
        Account Name:           ICL-WIN11$
        Account Domain:         GREENHORN
        Logon ID:               0x3E7


<SNIP>
```

Çevrimdışı proses için belirli bir logdaki eventleri de dışarı aktarabiliriz. Bu dışa aktarma işlemini gerçekleştirmek için lokal admin de gereklidir.


### Exporting Events (Eventleri Dışa Aktarma)
```cmd-session
C:\htb> wevtutil epl System C:\system_export.evtx
```



### Windows Event Log ile Etkileşim - PowerShell
Benzer şekilde, [Get-WinEvent](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.4&viewFallbackFrom=powershell-7.3) PowerShell cmdlet'ini kullanarak Windows Event Logs ile etkileşim kurabiliriz. Wevtutil örneklerinde olduğu gibi, bazı komutlar lokal yönetici seviyesinde erişim gerektirir.

Başlangıç olarak, bilgisayardaki tüm logları listeleyebilir ve bize her logdaki kayıt sayısını verebiliriz.


### PowerShell - Tüm Logları Listeleme
```powershell-session
PS C:\htb> Get-WinEvent -ListLog *

LogMode   MaximumSizeInBytes RecordCount LogName
-------   ------------------ ----------- -------
Circular            15728640         657 Windows PowerShell
Circular            20971520       10713 System
Circular            20971520       26060 Security
Circular            20971520           0 Key Management Service
Circular             1052672           0 Internet Explorer
Circular            20971520           0 HardwareEvents
Circular            20971520        6202 Application
Circular             1052672             Windows Networking Vpn Plugin Platform/Op...
Circular             1052672             Windows Networking Vpn Plugin Platform/Op... 
Circular             1052672           0 SMSApi
Circular             1052672          61 Setup
Circular            15728640          24 PowerShellCore/Operational
Circular             1052672          99 OpenSSH/Operational
Circular             1052672          46 OpenSSH/Admin

<SNIP>
```

Belirli bir log hakkındaki bilgileri de listeleyebiliriz. Burada ==Security== logunun boyutunu görebiliriz.

#### Security Log Details
```powershell-session
PS C:\htb> Get-WinEvent -ListLog Security

LogMode   MaximumSizeInBytes RecordCount LogName
-------   ------------------ ----------- -------
Circular            20971520       26060 Security
```

==MaxEvents== parametresini kullanarak özellikle son beş olayı arayarak son X sayıda event için sorgulama yapabiliriz. Burada Security günlüğüne kaydedilen son beş olayı listeleyeceğiz. Varsayılan olarak, en yeni günlükler ilk olarak listelenir. Önce daha eski günlükleri almak istiyorsak,  ==-Oldest== parametresini kullanarak en eskileri önce listelemek için sırayı tersine çevirebiliriz.


### Son Beş Olayı Sorgulama

```powershell-session
PS C:\htb> Get-WinEvent -LogName 'Security' -MaxEvents 5 | Select-Object -ExpandProperty Message

An account was logged off.

Subject:
        Security ID:            S-1-5-111-3847866527-469524349-687026318-516638107-1125189541-6052
        Account Name:           sshd_6052
        Account Domain:         VIRTUAL USERS
        Logon ID:               0x8E787

Logon Type:                     5

This event is generated when a logon session is destroyed. It may be positively correlated with a logon event using the Logon ID value. Logon IDs are only unique between reboots on the same computer.
Special privileges assigned to new logon.

Subject:
        Security ID:            S-1-5-18
        Account Name:           SYSTEM
        Account Domain:         NT AUTHORITY
        Logon ID:               0x3E7

Privileges:             SeAssignPrimaryTokenPrivilege
                        SeTcbPrivilege
                        SeSecurityPrivilege
                        SeTakeOwnershipPrivilege
                        SeLoadDriverPrivilege
                        SeBackupPrivilege
                        SeRestorePrivilege
                        SeDebugPrivilege
                        SeAuditPrivilege
                        SeSystemEnvironmentPrivilege
                        SeImpersonatePrivilege
                        SeDelegateSessionUserImpersonatePrivilege
An account was successfully logged on.

<SNIP>
```

Daha derine inebilir ve belirli günlüklerdeki belirli olay kimliklerine bakabiliriz. Diyelim ki yalnızca Security günlüğündeki oturum açma hatalarına bakmak istiyoruz ve[ Event ID 4625'i](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/auditing/event-4625) kontrol ediyoruz: Bir hesap oturum açamadı. Buradan -ExpandProperty parametresini kullanarak belirli olayları daha derinlemesine inceleyebilir, günlükleri en eskiden en yeniye doğru listeleyebilir vb.


### Oturum Açma Hataları için Filtreleme
```powershell-session
PS C:\htb> Get-WinEvent -FilterHashTable @{LogName='Security';ID='4625 '}

   ProviderName: Microsoft-Windows-Security-Auditing

TimeCreated                      Id LevelDisplayName Message
-----------                      -- ---------------- -------
11/16/2022 2:53:16 PM          4625 Information      An account failed to log on....  
11/16/2022 2:53:16 PM          4625 Information      An account failed to log on.... 
11/16/2022 2:53:12 PM          4625 Information      An account failed to log on.... 
11/16/2022 2:50:36 PM          4625 Information      An account failed to log on.... 
11/16/2022 2:50:29 PM          4625 Information      An account failed to log on.... 
11/16/2022 2:50:21 PM          4625 Information      An account failed to log on....

<SNIP>
```

Yalnızca belirli bir bilgi düzeyine sahip olaylara da bakabiliriz. Tüm System günlüklerini yalnızca bilgi düzeyi 1 olan ==kritik== olaylar için kontrol edelim. Burada sistemin temiz bir şekilde yeniden başlatılmadığı tek bir günlük girdisi görüyoruz.

```powershell-session
PS C:\htb> Get-WinEvent -FilterHashTable @{LogName='System';Level='1'} | select-object -ExpandProperty Message

The system has rebooted without cleanly shutting down first. This error could be caused if the system stopped responding, crashed, or lost power unexpectedly.
```

Logları arama konusunda daha rahat olmak için wevtutil ve Get-WinEvent ile daha fazla pratik yapın. Microsoft Get-WinEvent için bazı [örnekler](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent?view=powershell-7.4&viewFallbackFrom=powershell-7.3) sağlarken, [bu site](https://www.thewindowsclub.com/what-is-wevtutil-and-how-do-you-use-it) wevtutil için örnekler gösterir ve [bu sitede](https://4sysops.com/archives/search-the-event-log-with-the-get-winevent-powershell-cmdlet/) Get-WinEvent kullanımı için bazı ek örnekler vardır.


### Devam Ediyoruz
Bu bölümde, daha sonraki modüllerde çok daha derinlemesine inceleyeceğimiz geniş bir konu olan Windows Event Log tanıtıldı. Bu bölümdeki çeşitli örnekleri deneyin ve belirli bilgileri sorgulamak için her iki aracı da rahatça kullanın. Daha sonraki modüllerde, Event Log'larda bazen parolalar gibi hassas verileri nasıl bulabileceğimizi göreceğiz. Windows'ta günlük kaydı düzgün yapılandırıldığında çok güçlüdür. Her sistem çok büyük miktarda günlük oluşturur ve tüm olası Event ID'lerde gördüğümüz gibi, tam olarak neyi günlüğe kaydetmeyi seçtiğimiz konusunda oldukça ayrıntılı olabiliriz. Tüm bu verilerin tek başına sürekli olarak sorgulanması çok zordur ve Kerberoasting, Password Spraying veya diğer daha az yaygın saldırılar gibi bir saldırının göstergesi olabilecek belirli Event ID'ler hakkında uyarılar ayarlamak için kullanılabilecek bir SIEM aracına iletildiğinde en etkilidir. Sızma testi uzmanları olarak Windows Olay Günlüğünü, ortam hakkında bilgi edinmek ve hatta bazen hassas verileri ayıklamak için nasıl kullanabileceğimizi bilmeliyiz. Mavi ekip çalışanları için Windows Olay Günlüğü hakkında derinlemesine bilgi sahibi olmak ve etkili uyarı ve izleme için bundan nasıl yararlanılacağını bilmek kritik önem taşır.

Bir sonraki bölümde, bir Windows sisteminde komut satırından ağ işlemleri ile çalışmayı ele alacağız.



### CLI'dan Networking Yönetimi
PowerShell, Networking ayarları, uygulamalar ve daha fazlası ile uğraşırken Windows işletim sistemi içindeki yeteneklerimizi genişletti. Bu bölümde IP adresleri, adaptör ayarları ve DNS ayarları gibi ağ ayarlarınızı nasıl kontrol edeceğiniz ele alınacaktır. Ayrıca WinRM ve SSH kullanarak remote host erişimini nasıl etkinleştireceğimizi ve yöneteceğimizi de ele alacağız.

Senaryo: Bay Tanaka'nın hostunun düzgün çalıştığından ve BT ofisinden uzaktan yönetebildiğimizden emin olmak için hızlı bir kontrol gerçekleştireceğiz, host ayarlarını doğrulayacağız ve host için remote management'ı etkinleştireceğiz.


### Windows Ağı İçinde Networking Nedir?
Windows host'larla ağ oluşturma, diğer Linux veya Unix tabanlı host'lar gibi çalışır. TCP/IP stack, kablosuz protokoller ve diğer uygulamalar çoğu cihaza aynı şekilde davranır, bu nedenle öğrenilecek yeni bir şey yoktur. Bu modül, temel ağ protokollerini ve tipik ağ trafiğinin İnternet'i nasıl geçtiğini bildiğinizi varsayar. Ağ iletişimi hakkında daha fazla bilgi edinmek isterseniz Ağ İletişimine Giriş modülüne göz atabilir ya da ağ trafiğinin daha derinlemesine incelenmesi için Ağ Trafiği Analizine Giriş modülünü inceleyebilirsiniz. İşlerin biraz farklılaştığı nokta Windows hostlarının birbirleriyle, domainlerle ve diğer Linux hostlarıyla nasıl iletişim kurduklarıdır. Aşağıda, Windows host'ları yönetirken veya pentest yaparken karşılaşabileceğiniz bazı standart protokolleri hızlıca ele alacağız.

SMB : [SMB](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-smb2/4287490c-602c-41c0-a23e-140a1f137832), Windows hostlarına kaynakları, dosyaları paylaşma yeteneği ve kaynaklara erişime izin verilip verilmediğini belirlemek için hostlar arasında kimlik doğrulamanın standart bir yolunu sağlar. Diğer dağıtımlar için SAMBA açık kaynak seçeneğidir.

Netbios : [NetBios](https://www.ietf.org/rfc/rfc1001.txt)'un kendisi doğrudan bir servis veya protokol değil, ağlarda yaygın olarak kullanılan bir bağlantı ve konuşma mekanizmasıdır. SMB için orijinal aktarım mekanizmasıydı, ancak o zamandan beri bu değişti. Şimdi DNS başarısız olduğunda alternatif bir tanımlama mekanizması olarak hizmet vermektedir. NBT-NS (NetBIOS name service) olarak da bilinir.

LDAP : [LDAP](https://www.rfc-editor.org/rfc/rfc4511), çeşitli dizin hizmetleri ile kimlik doğrulama ve yetkilendirme için kullanılan open source bir çapraz platform protokolüdür. Modern ağlardaki birçok farklı cihaz Active Directory gibi büyük dizin yapısı hizmetleriyle bu şekilde iletişim kurabilir.

LLMNR: [LLMNR](https://www.rfc-editor.org/rfc/rfc4795), DNS tabanlı bir name resolution hizmeti sağlar ve DNS mevcut değilse veya çalışmıyorsa çalışır. Bu protokol bir çok noktaya yayın protokolüdür ve bu nedenle yalnızca lokal bağlantılarda çalışır (normal bir yayın alanı içinde, katman üç bağlantıları arasında değil).

DNS : [DNS](https://datatracker.ietf.org/doc/html/rfc1034), İnternet genelinde ve çoğu modern ağ türünde kullanılan yaygın bir adlandırma standardıdır. DNS, hostlara IP adresleri yerine benzersiz bir isimle referans vermemizi sağlar. Bu sayede bir web sitesine “8.8.8.8” yerine “WWW.google.com” şeklinde referans verebiliriz. İçsel olarak bu, bir ağdan kaynakları ve erişimi nasıl talep ettiğimizdir.

HTTP/HTTPS : [HTTP/S HTTP ve HTTPS](https://www.rfc-editor.org/rfc/rfc2818), İnternet üzerinden kaynak talep etme ve kullanma yöntemlerimizin güvenli ve güvensiz yoludur. Bu protokoller web sunucuları gibi kaynaklara erişmek ve bunları kullanmak, uzak kaynaklardan veri göndermek ve almak ve çok daha fazlası için kullanılır.

Kerberos : [Kerberos](https://web.mit.edu/kerberos/) ağ düzeyinde bir kimlik doğrulama protokolüdür. Modern zamanlarda, client'lar domain kaynaklarını kullanmak için yetkilendirme biletleri talep ettiklerinde Active Directory kimlik doğrulaması ile uğraşırken görmemiz muhtemeldir.

WinRM: [WinRM WS-Management](https://learn.microsoft.com/en-us/windows/win32/winrm/portal) protokolünün bir uygulamasıdır. Hostların donanım ve yazılım fonksiyonlarını yönetmek için kullanılabilir. Esas olarak BT yönetiminde kullanılır, ancak host numaralandırma ve komut dosyası motoru olarak da kullanılabilir.

RDP : [RDP](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/rds-plan-access-from-anywhere), kullanıcılara bir ağ bağlantısı üzerinden hostlara erişmek için bir Grafik arayüz sağlayan bir ağ UI hizmetleri protokolünün Windows uygulamasıdır. Bu, klavye ve fare girdisinin remote host'a aktarılmasını içerecek şekilde tam UI kullanımına izin verir.

SSH : [SSH](https://datatracker.ietf.org/doc/html/rfc4251), güvenli host erişimi, dosya transferi ve ağ hostları arasında genel iletişim için kullanılabilen güvenli bir protokoldür. Güvensiz ağlar üzerinden hostlara ve hizmetlere güvenli bir şekilde erişmek için bir yol sağlar.

Elbette, bu liste her şeyi kapsamamaktadır, ancak Windows host'larıyla iletişim kurarken tipik olarak göreceğimiz şeylerin mükemmel bir genel başlangıcıdır. Şimdi lokal erişime karşı remote erişimi tartışalım.

## Local vs. Remote Access?


### Local Access
Lokal host erişimi, şu anda bilgisayarınızdan yaptığınız gibi doğrudan terminalde olup kaynaklarını kullandığımız durumdur. Genellikle bu, ağa bağlı hostlardan kaynak talep ettiğimiz veya İnternet'e erişmeye çalıştığımız durumlar dışında herhangi bir özel erişim protokolü kullanmamızı gerektirmez. Aşağıda, hosts'umuzdaki ağ ayarlarını kontrol etmek ve doğrulamak için bazı cmdlet'leri ve diğer yolları göstereceğiz.

### Ağ Ayarlarını Sorgulama
Başka bir şey yapmadan önce, Bay Tanaka'nın hostundaki ağ ayarlarını doğrulayalım. IPConfig komutunu çalıştırarak başlayacağız. Bu bir PowerShell yerel komutu değildir, ancak uyumludur.

#### IPConfig
```powershell-session
PS C:\htb> ipconfig 

Windows IP Configuration

Ethernet adapter Ethernet0:

   Connection-specific DNS Suffix  . : .htb
   Link-local IPv6 Address . . . . . : fe80::c5ca:594d:759d:e0c1%11
   IPv4 Address. . . . . . . . . . . : 10.129.203.105
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:b9fc%11
                                       10.129.0.1
```

Gördüğümüz gibi, ipconfig bize ağ arayüzünüzün temel ayarlarını gösterecektir. IPv4/6 adreslerini, ağ geçidimizi, alt ağ maskelerini ve ayarlanmışsa DNS sonekini çıktı olarak alıyoruz. ipconfig komutuna aşağıdaki gibi /all değiştiricisini ekleyerek tüm ağ ayarlarının çıktısını alabiliriz:

Artık eskisinden çok daha fazla bilgi görebiliyoruz. Birden fazla adaptör, Host ayarları, IP adreslerimizin manuel olarak mı ayarlandığı yoksa DHCP kiralamaları mı olduğu, bu kiralamaların ne kadar sürdüğü ve daha fazlası hakkında daha fazla ayrıntı içeren çıktılar sunuluyor. Görünüşe göre Bay Tanaka'nın hostu düzgün bir IP adresi yapılandırmasına sahip. Dikkat çeken ve özellikle pentester olarak bizim için ilginç olan, bu hostun dual-homed olması. Yani ayrı ağlara bağlı birden fazla ağ arayüzüne sahip. Bu durum, ağda bir yer edinmek istiyorsak ve ağlar arasında geçiş yapmanın bir yolunu bulmak istiyorsak Bay Tanaka'nın hostunu harika bir hedef haline getiriyor.

Arp ayarlarına bakalım ve host'unun ağdaki diğerleriyle iletişim kurup kurmadığını görelim. Tekrar hatırlatmak gerekirse ARP, IP adreslerini Fiziksel adreslere çevirmek için kullanılan bir protokoldür. Fiziksel adres, iletişim için OSI/TCP-IP modellerinin daha düşük seviyelerinde kullanılır. Hostun mevcut ARP girişlerini görüntülemesini sağlamak için -a anahtarını kullanacağız.


#### ARP
```powershell-session
PS C:\htb> arp -a

Interface: 10.129.203.105 --- 0xb
  Internet Address      Physical Address      Type
  10.129.0.1            00-50-56-b9-b9-fc     dynamic
  10.129.204.58         00-50-56-b9-5f-41     dynamic
  10.129.255.255        ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
  255.255.255.255       ff-ff-ff-ff-ff-ff     static

Interface: 172.16.5.100 --- 0xe
  Internet Address      Physical Address      Type
  172.16.5.155          00-50-56-b9-e2-30     dynamic
  172.16.5.255          ff-ff-ff-ff-ff-ff     static
  224.0.0.22            01-00-5e-00-00-16     static
  224.0.0.251           01-00-5e-00-00-fb     static
  224.0.0.252           01-00-5e-00-00-fc     static
  239.255.255.250       01-00-5e-7f-ff-fa     static
```

Arp -a çıktısı oldukça basittir. Ağ bağdaştırıcılarımızdan, farkında olduğu ya da yakın zamanda iletişim kurduğu hostlar hakkında girdiler alıyoruz. Şaşırtıcı olmayan bir şekilde, bu host oldukça yeni olduğundan, henüz çok fazla host ile iletişim kurmamıştır. Sadece ağ geçitleri, remote hostumuz ve Greenhorn.corp için Domain Controller olan 172.16.5.155 hostu. Burada görülecek çılgınca bir şey yok. Şimdi DNS yapılandırmamızın düzgün çalıştığını doğrulayalım. Greenhorn domain controller'ın IP adresini / DNS adını çözümlemeye çalışmak için yerleşik bir DNS sorgulama aracı olan nslookup'ı kullanacağız.

#### Nslookup
```powershell-session
PS C:\htb> nslookup ACADEMY-ICL-DC

DNS request timed out.
    timeout was 2 seconds.
Server:  UnKnown
Address:  172.16.5.155

Name:    ACADEMY-ICL-DC.greenhorn.corp
Address:  172.16.5.155
```

Şimdi Bay Tanakas'ın DNS ayarlarını doğruladığımıza göre, host üzerindeki açık portları kontrol edelim. Bunu netstat -an kullanarak yapabiliriz. Netstat hostumuza olan mevcut ağ bağlantılarını gösterecektir. -an anahtarı tüm bağlantıları ve dinleme portlarını yazdıracak ve bunları sayısal formda yerleştirecektir.


#### Netstat
```powershell-session
PS C:\htb> netstat -an 

netstat -an

Active Connections

  Proto  Local Address          Foreign Address        State
  TCP    0.0.0.0:22             0.0.0.0:0              LISTENING
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING
  TCP    0.0.0.0:3389           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:5040           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49671          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49673          0.0.0.0:0              LISTENING
  TCP    0.0.0.0:49674          0.0.0.0:0              LISTENING
  TCP    10.129.203.105:22      10.10.14.19:32557      ESTABLISHED
  TCP    172.16.5.100:139       0.0.0.0:0              LISTENING
  TCP    [::]:22                [::]:0                 LISTENING
  TCP    [::]:135               [::]:0                 LISTENING
  TCP    [::]:445               [::]:0                 LISTENING
  TCP    [::]:3389              [::]:0                 LISTENING
  TCP    [::]:5985              [::]:0                 LISTENING
  TCP    [::]:47001             [::]:0                 LISTENING
  TCP    [::]:49664             [::]:0                 LISTENING
  TCP    [::]:49665             [::]:0                 LISTENING
  TCP    [::]:49666             [::]:0                 LISTENING
  TCP    [::]:49667             [::]:0                 LISTENING
  TCP    [::]:49668             [::]:0                 LISTENING
  TCP    [::]:49671             [::]:0                 LISTENING
  TCP    [::]:49673             [::]:0                 LISTENING
  TCP    [::]:49674             [::]:0                 LISTENING
  UDP    0.0.0.0:123            *:*
<SNIP>
  UDP    172.16.5.100:137       *:*
  UDP    172.16.5.100:138       *:*
  UDP    172.16.5.100:1900      *:*
  UDP    172.16.5.100:54453     *:*
```

Şimdi, ağ trafiğine bakma veya standart portlar ve protokolleri anlama konusunda bir arka plan edinmeniz gerekebilir, aksi takdirde yukarıdakiler anlamsız görünebilir. Yine de sorun değil. Yukarıya baktığımızda, hangi portların açık olduğunu ve herhangi bir aktif bağlantımız olup olmadığını görebiliriz. Yukarıdaki çıktıya göre, açık olan portların hepsi Windows ortamlarında yaygın olarak kullanılan ve beklenen portlardır. Çoğu Active Directory hizmetleri ve SSH ile ilgilidir. Bağlantılara baktığımızda, şu anda aktif olan yalnızca bir oturum görüyoruz: TCP port 22 üzerinden kendi SSH bağlantımız.

Bu noktaya kadar çalıştığımız bu komutların çoğu Windows built-in executables'dır ve bir host hakkında hızlı bilgi edinmek için faydalıdır, ancak daha fazlası için değil. Aşağıda, ağ bağlantılarımızı ayrıntılı olarak yönetmemize olanak tanıyan PowerShell'den eklenmiş birkaç cmdlet'i ele alacağız.


### PowerShell Net Cmdlets
PowerShell, ağ hizmetlerini ve yönetimini ele almak için yapılmış birçok güçlü yerleşik cmdlet'e sahiptir. NetAdapter, NetConnection ve NetTCPIP modülleri bugün pratik yapacağımız modüllerden sadece birkaçıdır.

#### Net Cmdlets
* Get-NetIPInterface : Görünür tüm ağ bağdaştırıcısı özelliklerini getirir.
* Get-NetIPAddress : Her adaptörün IP konfigürasyonlarını getirir. IPConfig'e benzer.
* Get-NetNeighbor : Önbellekten komşu girişlerini alır. Arp -a'ya benzer.
* Get-Netroute : Mevcut rota tablosunu yazdırır. IPRoute'a benzer.
* Set-NetAdapter : VLAN kimliği, açıklama ve MAC-Adresi gibi temel bağdaştırıcı özelliklerini Layer-2 düzeyinde ayarlar.
* Set-NetIPInterface : DHCP durumu, MTU ve diğer ölçümleri içerecek şekilde bir arayüzün ayarlarını değiştirir.
* New-NetIPAddress : Bir IP adresi oluşturur ve yapılandırır.
* Set-NetIPAddress : Bir ağ adaptörünün yapılandırmasını değiştirir.
* Disable-NetAdapter : Ağ adaptörü arayüzlerini devre dışı bırakmak için kullanılır.
* Enable-NetAdapter : Ağ adaptörlerini tekrar açmak ve ağ bağlantılarına izin vermek için kullanılır.
* Restart-NetAdapter: Bir adaptörü yeniden başlatmak için kullanılır. Adaptör ayarlarında yapılan değişiklikleri itmeye yardımcı olmak için yararlı olabilir.
* test-NetConnection : Bir bağlantı üzerinde tanılama denetimlerinin çalıştırılmasına izin verir. Ping, tcp, rota izleme ve daha fazlasını destekler.

Her cmdlet'i kullanımda göstermeyeceğiz, ancak kullanımınız için hızlı bir referans sağlamak akıllıca olacaktır. İlk olarak Get-NetIPInterface ile başlayacağız.

#### Get-NetIPInterface
```powershell-session
PS C:\htb> get-netIPInterface

ifIndex InterfaceAlias                  AddressFamily NlMtu(Bytes) InterfaceMetric Dhcp     ConnectionState PolicyStore
------- --------------                  ------------- ------------ --------------- ----     --------------- -----------
20      Ethernet 3                      IPv6                  1500              25 Enabled  Disconnected    ActiveStore
14      VMware Network Adapter VMnet8   IPv6                  1500              35 Enabled  Connected       ActiveStore
8       VMware Network Adapter VMnet2   IPv6                  1500              35 Enabled  Connected       ActiveStore
10      VMware Network Adapter VMnet1   IPv6                  1500              35 Enabled  Connected       ActiveStore
17      Local Area Connection* 2        IPv6                  1500              25 Enabled  Disconnected    ActiveStore
21      Bluetooth Network Connection    IPv6                  1500              65 Disabled Disconnected    ActiveStore
15      Local Area Connection* 1        IPv6                  1500              25 Disabled Disconnected    ActiveStore
25      Wi-Fi                           IPv6                  1500              40 Enabled  Connected       ActiveStore
7       Local Area Connection           IPv6                  1500              25 Enabled  Disconnected    ActiveStore
1       Loopback Pseudo-Interface 1     IPv6            4294967295              75 Disabled Connected       ActiveStore
20      Ethernet 3                      IPv4                  1500              25 Enabled  Disconnected    ActiveStore
14      VMware Network Adapter VMnet8   IPv4                  1500              35 Disabled Connected       ActiveStore
8       VMware Network Adapter VMnet2   IPv4                  1500              35 Disabled Connected       ActiveStore
10      VMware Network Adapter VMnet1   IPv4                  1500              35 Disabled Connected       ActiveStore
17      Local Area Connection* 2        IPv4                  1500              25 Disabled Disconnected    ActiveStore
21      Bluetooth Network Connection    IPv4                  1500              65 Enabled  Disconnected    ActiveStore
15      Local Area Connection* 1        IPv4                  1500              25 Enabled  Disconnected    ActiveStore
25      Wi-Fi                           IPv4                  1500              40 Enabled  Connected       ActiveStore
7       Local Area Connection           IPv4                  1500               1 Disabled Disconnected    ActiveStore
1       Loopback Pseudo-Interface 1     IPv4            4294967295              75 Disabled Connected       ActiveStore
```

Bu liste bize host üzerindeki mevcut arayüzlerimizi biraz karmaşık bir şekilde gösterir. Bize çok sayıda metrik sağlanır, ancak adaptörler AddressFamily'ye göre ayrılır. Dolayısıyla, söz konusu arayüzde IPv4 ve IPv6 etkinleştirilmişse, her adaptör için iki kez giriş görürüz. ifindex ve InterfaceAlias özellikleri özellikle kullanışlıdır. Bu özellikler, NetTCPIP modülü tarafından sağlanan diğer cmdlet'leri kullanmamızı kolaylaştırır. [Get-NetIPAddress](https://learn.microsoft.com/en-us/powershell/module/nettcpip/get-netipaddress?view=windowsserver2022-ps) cmdlet'ini kullanarak ifIndex 25'teki Wi-Fi bağlantımız için Adapter bilgilerini alalım.

#### Get-NetIPAddress
```powershell-session
PS C:\htb> Get-NetIPAddress -ifIndex 25

IPAddress         : fe80::a0fc:2e3d:c92a:48df%25
InterfaceIndex    : 25
InterfaceAlias    : Wi-Fi
AddressFamily     : IPv6
Type              : Unicast
PrefixLength      : 64
PrefixOrigin      : WellKnown
SuffixOrigin      : Link
AddressState      : Preferred
ValidLifetime     : Infinite ([TimeSpan]::MaxValue)
PreferredLifetime : Infinite ([TimeSpan]::MaxValue)
SkipAsSource      : False
PolicyStore       : ActiveStore

IPAddress         : 192.168.86.211
InterfaceIndex    : 25
InterfaceAlias    : Wi-Fi
AddressFamily     : IPv4
Type              : Unicast
PrefixLength      : 24
PrefixOrigin      : Dhcp
SuffixOrigin      : Dhcp
AddressState      : Preferred
ValidLifetime     : 21:35:36
PreferredLifetime : 21:35:36
SkipAsSource      : False
PolicyStore       : ActiveStore
```

Bu cmdlet de oldukça fazla bilgi döndürdü. Bilgi istemek için ifIndex numarasını nasıl kullandığımıza dikkat ettiniz mi? Aynı şeyi InterfaceAlias ile de yapabiliriz. Bu cmdlet, indeks, takma ad, DHCP durumu, arayüz türü ve diğer ölçümler gibi oldukça fazla bilgi döndürür. Bu, komut isteminden IPconfig çalıştırılabilir dosyasını yayınladığımızda göreceğimiz şeylerin çoğunu yansıtır. Peki ya arayüzdeki bir ayarı değiştirmek istersek? Bunu [Set-NetIPInterface](https://learn.microsoft.com/en-us/powershell/module/nettcpip/set-netipinterface?view=windowsserver2022-ps) ve [Set-NetIPAddress ](https://learn.microsoft.com/en-us/powershell/module/nettcpip/set-netipaddress?view=windowsserver2022-ps)cmdlet'lerini kullanarak yapabiliriz. Bu örnekte, diyelim ki arayüzün DHCP durumunu etkin'den devre dışı'ya değiştirmek ve IP'yi DHCP tarafından otomatik olarak atanan bir IP'den manuel olarak ayarladığımız bir IP'ye değiştirmek istiyoruz. Bunu şu şekilde gerçekleştirebiliriz:

#### Set-NetIPInterface
```powershell-session
PS C:\htb> Set-NetIPInterface -InterfaceIndex 25 -Dhcp Disabled
```

Set-NetIPInterface cmdlet'i ile DHCP özelliğini devre dışı bırakarak, artık manuel IP Adresimizi ayarlayabiliriz. Bunu Set-NetIPAddress cmdlet'i ile yapıyoruz.

#### Set-NetIPAddress
```powershell-session
PS C:\htb> Set-NetIPAddress -InterfaceIndex 25 -IPAddress 10.10.100.54 -PrefixLength 24

PS C:\htb> Get-NetIPAddress -ifindex 20 | ft InterfaceIndex,InterfaceAlias,IPAddress,PrefixLength

InterfaceIndex InterfaceAlias IPAddress                   PrefixLength
-------------- -------------- ---------                   ------------
            20 Ethernet 3     fe80::7408:bbf:954a:6ae5%20           64
            20 Ethernet 3     10.10.100.54                          24

PS C:\htb> Get-NetIPinterface -ifindex 20 | ft ifIndex,InterfaceAlias,Dhcp

ifIndex InterfaceAlias     Dhcp
------- --------------     ----
     20 Ethernet 3     Disabled
     20 Ethernet 3     Disabled
```
Yukarıdaki komut şimdi IP adresimizi 10.10.100.54 ve PrefixLength'i (alt ağ maskesi olarak da bilinir) 24 olarak ayarlar. Kontrollerimize baktığımızda, bu ayarların yerinde olduğunu görebiliriz. Güvende olmak için ağ bağdaştırıcımızı yeniden başlatalım ve bağlantımızın devam edip etmediğini görmek için test edelim.

#### Restart-NetAdapter
```powershell-session
PS C:\htb> Restart-NetAdapter -Name 'Ethernet 3'
```

Hiçbir şey ters gitmediği sürece çıktı almayacaksınız. Yani Restart-NetAdapter söz konusu olduğunda, hiçbir haber iyi haber değildir. Cmdlet'e hangi arayüzü yeniden başlatacağını söylemenin en kolay yolu, çalıştırdığımız önceki komutlardaki InterfaceAlias ile aynı olan Name özelliğidir. Şimdi, hala bir bağlantımız olduğundan emin olmak için Test-NetConnection cmdlet'ini kullanabiliriz.


#### Test-NetConnection
```powershell-session
PS C:\htb> Test-NetConnection

ComputerName           : <snip>msedge.net
RemoteAddress          : 13.107.4.52
InterfaceAlias         : Ethernet 3
SourceAddress          : 10.10.100.54
PingSucceeded          : True
PingReplyDetails (RTT) : 44 ms
```

[Test-NetConnection](https://learn.microsoft.com/en-us/powershell/module/nettcpip/test-netconnection?view=windowsserver2022-ps), başka bir host'a ulaşıp ulaşamayacağımızı belirlemek için temel ağ bağlantısının ötesinde testler yapabilen güçlü bir cmdlet'tir. Bize TCP sonuçlarımız, ayrıntılı metrikler, rota tanılama ve daha fazlası hakkında bilgi verebilir. Microsoft'un Test-NetConnection hakkındaki bu makalesine göz atmanız faydalı olacaktır. Görevimizi tamamladığımıza ve Bay Tanaka'nın hostundaki ağ ayarlarını doğruladığımıza göre, şimdi biraz uzaktan erişim bağlantısını tartışalım.

### Remote Access
Windows sistemlerine erişemediğimizde veya hostları uzaktan yönetmemiz gerektiğinde, işimizi gerçekleştirmek için diğer araçların yanı sıra PowerShell, SSH ve RDP'yi kullanabiliriz. Uzaktan erişimi etkinleştirebileceğimiz ve kullanabileceğimiz ana yolları ele alalım. İlk olarak, SSH'yi tartışacağız.


### Remote Access Nasıl Etkinleştirilir? (SSH, PSSessions, vb.)


### SSH Erişimini Etkinleştirme
Ağ üzerinden bir Windows sistemindeki PowerShell'e erişmek için SSH kullanabiliriz. 2018'den itibaren, OpenSSH client ve server uygulamaları aracılığıyla SSH erişilebilir hale geldi ve tüm Windows Server ve Client sürümlerine dahil edildi. Bu, idari kullanımımız için kullanımı kolay ve genişletilebilir bir iletişim mekanizması sağlar. OpenSSH'ı hostlarımızda kurmak basittir. Hadi bir deneyelim. SSH üzerinden bir host'a uzaktan erişmek için SSH Server bileşenini ve client uygulamasını kurmalıyız.


### Windows Hedefinde SSH Kurulumu
[Add-WindowsCapability](https://learn.microsoft.com/en-us/powershell/module/dism/add-windowscapability?view=windowsserver2022-ps) cmdlet'ini kullanarak bir Windows hedefinde bir SSH sunucusu kurabilir ve [Get-WindowsCapability](https://learn.microsoft.com/en-us/powershell/module/dism/get-windowscapability?view=windowsserver2022-ps) cmdlet'ini kullanarak başarıyla kurulduğunu onaylayabiliriz.

```powershell-session
PS C:\Users\htb-student> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent

PS C:\Users\htb-student> Add-WindowsCapability -Online -Name OpenSSH.Client~~~~0.0.1.0

Path          :
Online        : True
RestartNeeded : False

PS C:\Users\htb-student> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : NotPresent

PS C:\Users\htb-student> Add-WindowsCapability -Online -Name OpenSSH.Server~~~~0.0.1.0

Path          :
Online        : True
RestartNeeded : False

PS C:\Users\htb-student> Get-WindowsCapability -Online | Where-Object Name -like 'OpenSSH*'

Name  : OpenSSH.Client~~~~0.0.1.0
State : Installed

Name  : OpenSSH.Server~~~~0.0.1.0
State : Installed
```


### SSH Hizmetini Başlatma ve Başlangıç Türünü Ayarlama
SSH'nin yüklendiğini onayladıktan sonra, SSH servisini başlatmak için [Start-Service](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/start-service?view=powershell-7.2) cmdlet'ini kullanabiliriz. İstersek SSH servisinin başlangıç ayarlarını yapılandırmak için [Set-Service](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.management/set-service?view=powershell-7.2) cmdlet'ini de kullanabiliriz.

```powershell-session
PS C:\Users\htb-student> Start-Service sshd  
  
PS C:\Users\htb-student> Set-Service -Name sshd -StartupType 'Automatic'  
```

Not: Remote access hizmetlerinin ilk kurulumu, bu modülde zorluk sorularını tamamlamak için bir gereklilik olmayacaktır. Bu modüldeki zorlukların her birinde, remote access zaten kurulmuş ve yapılandırılmıştır. Ancak, modül boyunca ele alınan kavramların nasıl bağlanacağının ve uygulanacağının anlaşılması gerekecektir. Kurulum ve yapılandırma adımları, yaygın yapılandırma hatalarının ve bazı durumlarda en iyi güvenlik uygulamalarının anlaşılmasına yardımcı olmak için sağlanmıştır. Kendi kişisel sanal makinenizde bazı kurulum adımlarını denemekten çekinmeyin.


### PowerShell'e SSH üzerinden erişme
Bir Windows hedefinde SSH yüklü ve çalışır durumdayken, bir SSH istemcisi ile ağ üzerinden bağlanabiliriz.


### Windows'tan Bağlanma
```powershell-session
PS C:\Users\administrator> ssh htb-student@10.129.224.248

htb-student@10.129.224.248 password:
```

Varsayılan olarak, bu bizi bir CMD oturumuna bağlayacaktır, ancak bu bölümde daha önce belirtildiği gibi bir PowerShell oturumuna girmek için powershell yazabiliriz.

```powershell-session
WS01\htb-student@WS01 C:\Users\htb-student> powershell

Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved. 

PS C:\Users\htb-student>
```

Linux kullanarak SSH üzerinden bir Windows hedefine bağlanma adımlarının Windows'tan bağlanma adımlarıyla aynı olduğunu fark edeceğiz.

#### Connecting from Linux
```shell-session
PS C:\Users\administrator> ssh htb-student@10.129.224.248

htb-student@10.129.224.248 password:

WS01\htb-student@WS01 C:\Users\htb-student> powershell

Windows PowerShell
Copyright (C) Microsoft Corporation. All rights reserved. 

PS C:\Users\htb-student>
```

SSH konusunu ele aldığımıza göre şimdi de uzaktan erişim ve yönetim için WinRM'yi etkinleştirme ve kullanma konusuna biraz zaman ayıralım.



### WinRM'yi Etkinleştirme
[Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (WinRM) özel PowerShell cmdlet'leri kullanılarak yapılandırılabilir ve bir PowerShell etkileşimli oturumuna girebilir ve remote Windows hedef(ler)inde komutlar verebiliriz. WinRM'nin Windows Server işletim sistemlerinde daha yaygın olarak etkinleştirildiğini fark edeceğiz, böylece BT yöneticileri bir veya birden fazla host üzerinde görevler gerçekleştirebilir. Windows Server'da varsayılan olarak etkindir.

Windows sistemlerindeki görevleri uzaktan yönetme ve otomatikleştirme becerisine yönelik artan talep nedeniyle, WinRM'nin giderek daha fazla Windows masaüstü işletim sisteminde (Windows 10 ve Windows 11) de etkinleştirildiğini göreceğiz. WinRM bir Windows hedefinde etkinleştirildiğinde, 5985 ve 5986 numaralı mantıksal portları dinler.


### WinRM'yi Etkinleştirme ve Yapılandırma
WinRM, aşağıdaki komutlar kullanılarak bir Windows hedefinde etkinleştirilebilir:
```powershell-session
PS C:\WINDOWS\system32> winrm quickconfig

WinRM service is already running on this machine.
WinRM is not set up to allow remote access to this machine for management.
The following changes must be made:

Enable the WinRM firewall exception.
Configure LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.

Make these changes [y/n]? y

WinRM has been updated for remote management.

WinRM firewall exception enabled.
Configured LocalAccountTokenFilterPolicy to grant administrative rights remotely to local users.
```

Yukarıdaki çıktıda görülebileceği gibi, bu komutu çalıştırmak otomatik olarak gerekli tüm yapılandırmaların yerinde olmasını sağlayacaktır:

* WinRM hizmetini etkinleştirin
* Windows Defender Güvenlik Duvarı üzerinden WinRM'ye izin ver (Gelen ve Giden)
* Lokal kullanıcılara uzaktan yönetici hakları verme


Sisteme erişim için kimlik bilgileri bilindiği sürece, bu komut çalıştırıldıktan sonra ağ üzerinden hedefe ulaşabilen herkes bağlanabilir. BT yöneticileri, özellikle sisteme İnternet üzerinden uzaktan erişilecekse, bu WinRM yapılandırmalarını güçlendirmek için başka adımlar atmalıdır. Bu sertleştirme seçeneklerinden bazıları şunlardır:

* TrustedHosts'u yalnızca uzaktan yönetim için kullanılacak IP adreslerini/hostları adlarını içerecek şekilde yapılandırın
* Aktarım için HTTPS'yi yapılandırma
* Windows sistemlerini Active Directory Domain Ortamına Ekleme ve Kerberos Kimlik Doğrulamasını Zorlama

### PowerShell Remote Access'i Test Etme
WinRM'yi etkinleştirip yapılandırdıktan sonra, [Test-WSMan](https://docs.microsoft.com/en-us/powershell/module/microsoft.wsman.management/test-wsman?view=powershell-7.2) PowerShell cmdlet'ini kullanarak uzaktan erişimi test edebiliriz.

#### Testing Unauthenticated Access
```powershell-session
PS C:\Users\administrator> Test-WSMan -ComputerName "10.129.224.248"

wsmid           : http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd
ProtocolVersion : http://schemas.dmtf.org/wbem/wsman/1/wsman.xsd
ProductVendor   : Microsoft Corporation
ProductVersion  : OS: 0.0.0 SP: 0.0 Stack: 3.0
```
Bu cmdlet'i çalıştırmak, WinRM hizmetinin çalışıp çalışmadığını kontrol eden bir istek gönderir. Bunun kimliği doğrulanmamış olduğunu unutmayın, bu nedenle kimlik bilgileri kullanılmaz, bu nedenle işletim sistemi sürümü algılanmaz. Bu bize WinRM hizmetinin hedef üzerinde çalıştığını gösterir.


#### Kimliği Doğrulanmış Erişimin Test Edilmesi
```powershell-session
PS C:\Users\administrator> Test-WSMan -ComputerName "10.129.224.248" -Authentication Negotiate


wsmid           : http://schemas.dmtf.org/wbem/wsman/identity/1/wsmanidentity.xsd
ProtocolVersion : http://schemas.dmtf.org/wbem/wsman/1/wsman.xsd
ProductVendor   : Microsoft Corporation
ProductVersion  : OS: 10.0.17763 SP: 0.0 Stack: 3.0
```
Aynı komutu -Authentication Negotiate seçeneği ile çalıştırarak WinRM'nin kimliğinin doğrulanıp doğrulanmadığını test edebiliriz ve işletim sistemi sürümünü (10.0.11764) alırız.

### PowerShell Remote Sessions
Ayrıca, bir Windows hedefiyle PowerShell oturumu oluşturmak için [Enter-PSSession](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/enter-pssession?view=powershell-7.2) cmdlet'ini kullanma seçeneğimiz de vardır.

### PowerShell Session'u Oluşturma
```powershell-session
PS C:\Users\administrator> Enter-PSSession -ComputerName 10.129.224.248 -Credential htb-student -Authentication Negotiate
[10.129.5.129]: PS C:\Users\htb-student\Documents> $PSVersionTable 
  
Name                           Value
----                           -----
PSVersion                      5.1.17763.592
PSEdition                      Desktop
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}
BuildVersion                   10.0.17763.592
CLRVersion                     4.0.30319.42000
WSManStackVersion              3.0
PSRemotingProtocolVersion      2.3
SerializationVersion           1.1.0.1
```

Aynı işlemi PowerShell core yüklü Linux tabanlı bir saldırı hostundan da gerçekleştirebiliriz (Pwnbox'ta olduğu gibi). PowerShell'in Windows'a özel olmadığını ve artık diğer işletim sistemlerinde de çalışacağını unutmayın.

#### Using Enter-PSSession from Linux
```shell-session
M1R4CKCK@htb[/htb]$ [PS]> Enter-PSSession -ComputerName 10.129.224.248 -Credential htb-student -Authentication Negotiate

PowerShell credential request
Enter your credentials.
Password for user htb-student: ***************

[10.129.224.248]: PS C:\Users\htb-student\Documents> $PSVersionTable

Name                           Value                                           
----                           -----                                           
PSVersion                      5.1.19041.1                                     
PSEdition                      Desktop                                         
PSCompatibleVersions           {1.0, 2.0, 3.0, 4.0...}                         
BuildVersion                   10.0.19041.1                                    
CLRVersion                     4.0.30319.42000                                 
WSManStackVersion              3.0                                             
PSRemotingProtocolVersion      2.3                                             
SerializationVersion           1.1.0.1
```

İşletim sisteminden bağımsız olmanın yanı sıra, artık host'larla uzaktan etkileşim kurmak için kullanabileceğimiz tonlarca farklı araç var. Hostlarımızı uzaktan yönetmek için bir araç seçmek, çoğunlukla neyle rahat olduğunuza ve etkileşime veya ortamınızın güvenlik ayarlarına bağlı olarak neyi kullanabileceğinize bağlıdır.

Networking, Windows hostlarda yönetilmesi oldukça basit bir görevdir. Ortamlarınız bulut sunucuları, birden fazla domain ve geniş coğrafi mesafelerdeki birden fazla site ile daha karmaşık hale geldikçe, ağ yönetimi de sıkıcı hale gelebilir. Neyse ki biz sadece lokal hostumuza ve tekil bir hostun nasıl yönetileceğine odaklanıyoruz. İleride, PowerShell kullanarak web ile nasıl etkileşim kurabileceğimize bakacağız.


Soru : Hangi PowerShell cmdlet'i bize host ağ adaptörlerinin IP konfigürasyonlarını gösterecektir.
Cevap : Get-NetIPAddress

Soru : Bir host üzerinde Windows Remote Management'ı hangi komut etkinleştirebilir ve yapılandırabilir?
Cevap  : winrm quickconfig


### Web ile Etkileşim
Bir administrator olarak, PowerShell aracılığıyla araçlar ve cmdlet'lerle uzaktan güncellemeleri nasıl gerçekleştireceğimizi, uygulamaları nasıl yükleyeceğimizi ve çok daha fazlasını otomatikleştirebiliriz. Bu sayede host'larda ihtiyaç duyduğumuz yazılımları, güncellemeleri ve diğer nesneleri GUI üzerinden manuel olarak taramadan lokal ve uzaktan alabiliriz. Bu bize zaman kazandıracak ve klavyenin başında oturmak ya da RDP'ye girmek yerine hostları uzaktan yönetebilmemizi sağlayacaktır. Bir pentester olarak bu, ihtiyaç duyduğumuz araçları ve diğer öğeleri ortama sokmanın ve gönderecek altyapımız varsa veri sızdırmanın hızlı bir yoludur. Bu bölüm web ile nasıl etkileşim kurduğumuzu ele alacak ve bu amaca hizmet etmek için PowerShell'i kullanmanın çeşitli yollarını gösterecektir.

### PowerShell Kullanarak Web ile Nasıl Etkileşim Kurarız?
PowerShell aracılığıyla web ile etkileşim söz konusu olduğunda, [Invoke-WebRequest ](https://learn.microsoft.com/bs-latn-ba/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-5.1)cmdlet'i bizim şampiyonumuzdur. Temel HTTP/HTTPS isteklerini (GET ve POST gibi) gerçekleştirmek, HTML sayfalarını ayrıştırmak, dosya indirmek, kimlik doğrulaması yapmak ve hatta bir siteyle oturumu sürdürmek için kullanabiliriz. Çok yönlüdür ve komut dosyası oluşturma ve otomasyonda kullanımı kolaydır. Takma adları tercih ederseniz, Invoke-WebRequest cmdlet'i wget, iwr ve curl ile takma ad olarak kullanılır. Linux Temelleri'ne aşina olanlar, Linux dağıtımlarında komut satırından dosya indirmek için kullanıldıkları için cURL ve wget'e aşina olabilirler. Bir dakikalığına Invoke-WebRequest'in yardımına bakalım.


#### Invoke-WebRequest Help
```powershell-session
PS C:\Windows\system32> Get-Help Invoke-Webrequest
```

Get-Help çıktısının özetinde şu ifadeye dikkat edin:

“İnternet üzerindeki bir web sayfasından içerik alır.”

Temel işlev bu olsa da, aynı ağ ortamındaki web sunucularında barındırdığımız içeriği almak için de kullanabiliriz. Bu konudan bahsettik ve şimdi Invoke-WebRequest kullanarak basit bir web isteği yapmayı deneyelim.


### Basit Bir Web İsteği
Aşağıda görüldüğü gibi Invoke-WebRequest cmdlet'i ile -Method GET değiştiricisini kullanarak bir web sitesinin temel bir Get isteğini gerçekleştirebiliriz. Bu örnek için URI'yi https://web.ics.purdue.edu/~gchopra/class/public/pages/webdesign/05_simple.html olarak belirteceğiz. Ayrıca nesnenin çıktı yöntemlerini ve özelliklerini incelemek için Get-Member'a göndereceğiz.


#### Get Request with Invoke-WebRequest
```powershell-session
PS C:\htb> Invoke-WebRequest -Uri "https://web.ics.purdue.edu/~gchopra/class/public/pages/webdesign/05_simple.html" -Method GET | Get-Member


   TypeName: Microsoft.PowerShell.Commands.HtmlWebResponseObject

----              ---------- ----------
Dispose           Method     void Dispose(), void IDisposable.Dispose()
Equals            Method     bool Equals(System.Object obj)
GetHashCode       Method     int GetHashCode()
GetType           Method     type GetType()
ToString          Method     string ToString()
AllElements       Property   Microsoft.PowerShell.Commands.WebCmdletElementCollection AllElements...
BaseResponse      Property   System.Net.WebResponse BaseResponse {get;set;}
Content           Property   string Content {get;}
Forms             Property   Microsoft.PowerShell.Commands.FormObjectCollection Forms {get;}
Headers           Property   System.Collections.Generic.Dictionary[string,string] Headers {get;}
Images            Property   Microsoft.PowerShell.Commands.WebCmdletElementCollection Images {get;}
InputFields       Property   Microsoft.PowerShell.Commands.WebCmdletElementCollection InputFields...
Links             Property   Microsoft.PowerShell.Commands.WebCmdletElementCollection Links {get;}
ParsedHtml        Property   mshtml.IHTMLDocument2 ParsedHtml {get;}
RawContent        Property   string RawContent {get;set;}
RawContentLength  Property   long RawContentLength {get;}
RawContentStream  Property   System.IO.MemoryStream RawContentStream {get;}
Scripts           Property   Microsoft.PowerShell.Commands.WebCmdletElementCollection Scripts {get;}
StatusCode        Property   int StatusCode {get;}
StatusDescription Property   string StatusDescription {get;}
```

Bu sitenin sahip olduğu tüm farklı özelliklere dikkat edin. Artık sitenin yalnızca bir bölümünü göstermek istiyorsak bunları filtreleyebiliriz. Örneğin, sadece sitedeki resimlerin bir listesini görmek istersek ne olur? Bunu, isteği gerçekleştirerek ve ardından aşağıdaki gibi yalnızca Görüntüler için filtreleme yaparak yapabiliriz:


#### Gelen İçeriği Filtreleme
```powershell-session
PS C:\htb> Invoke-WebRequest -Uri "https://web.ics.purdue.edu/~gchopra/class/public/pages/webdesign/05_simple.html" -Method GET | fl Images

Images : {@{innerHTML=; innerText=; outerHTML=<IMG alt="Pretty Picture"
         src="example/prettypicture.jpg">; outerText=; tagName=IMG; alt=Pretty Picture;
         src=example/prettypicture.jpg}, @{innerHTML=; innerText=; outerHTML=<IMG alt="Pretty
         Picture" src="example/prettypicture.jpg" align=top>; outerText=; tagName=IMG; alt=Pretty
         Picture; src=example/prettypicture.jpg; align=top}}
```

Bu sitenin sahip olduğu tüm farklı özelliklere dikkat edin. Artık sitenin yalnızca bir bölümünü göstermek istiyorsak bunları filtreleyebiliriz. Örneğin, sadece sitedeki resimlerin bir listesini görmek istersek ne olur? Bunu, isteği gerçekleştirerek ve ardından aşağıdaki gibi yalnızca Görüntüler için filtreleme yaparak yapabiliriz:


#### Raw Content
```powershell-session
PS C:\htb> Invoke-WebRequest -Uri "https://web.ics.purdue.edu/~gchopra/class/public/pages/webdesign/05_simple.html" -Method GET | fl RawContent

RawContent : HTTP/1.1 200 OK
             Strict-Transport-Security: max-age=16070400
             X-Content-Type-Options: nosniff
             X-XSS-Protection: 1; mode=block
             X-Frame-Options: SAMEORIGIN
             Accept-Ranges: bytes
             Content-Length: 1807
             Content-Type: text/html
             Date: Thu, 10 Nov 2022 16:25:07 GMT
             ETag: "70f-529340fa7b28d"
             Last-Modified: Wed, 13 Jan 2016 09:47:41 GMT
             Server: Apache/2.4.6 () OpenSSL/1.0.2k-fips

             <html>

             <head>
             <title>A very simple webpage</title>
             <basefont size=4>
             </head>

             <body bgcolor=FFFFFF>

             <h1>A very simple webpage. This is an "h1" level header.</h1>

             <h2>This is a level h2 header.</h2>

             <h6>This is a level h6 header.  Pretty small!</h6>

             <p>This is a standard paragraph.</p>

             <p align=center>Now I've aligned it in the center of the screen.</p>

             <p align=right>Now aligned to the right</p>

             <p><b>Bold text</b></p>

             <p><strong>Strongly emphasized text</strong>  Can you tell the difference vs. bold?</p>

             <p><i>Italics</i></p>

             <p><em>Emphasized text</em>  Just like Italics!</p>

             <p>Here is a pretty picture: <img src=example/prettypicture.jpg alt="Pretty
             Picture"></p>
<SNIP>
```

Talepteki her şeye bir kerede bakmak yerine bu sitenin “ raw content” ini çıkarabiliriz. Okumanın ne kadar kolay olduğunu fark ettiniz mi? Bir web sitesini yeniden keşfetmenin veya isimler, adresler ve e-postalar gibi önemli bilgileri çekmenin hızlı bir yolu olarak, bundan daha kolay olamaz. Invoke-WebRequest'in kullanışlı olduğu yer, CLI aracılığıyla dosya indirme yeteneğidir. Şimdi dosyaları indirmeye bakalım.

## Downloading Files using PowerShell
Sistem yöneticisi, pentesting görevi veya felaket kurtarma ile ilgili görevleri yerine getirirken, her türlü dosyanın kaçınılmaz olarak bir Windows hostuna indirilmesi gerekecektir. Bir pentesting çalışmasında, bir hedefi ele geçirmiş olabiliriz ve ortamı daha fazla numaralandırmak ve diğer hostlara ve ağlara ulaşmanın yollarını belirlemek için araçları bu host üzerine aktarmak isteyebiliriz. PowerShell bize bunu yapmak için bazı yerleşik seçenekler sunar. Bu modül için Invoke-WebRequest'e odaklanacağız, ancak web istekleri ve indirmeleri gerçekleştirebileceğimiz birçok farklı yol olduğunu biliyoruz (bazıları bunun içindir, diğerleri araç yaratıcıları tarafından kasıtsızdır).


### Downloading PowerView.ps1 from GitHub
[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) adında birçok pentester tarafından kullanılan popüler bir aracı indirerek Invoke-WebRequest kullanma pratiği yapabiliriz.


#### Download To Our Host
```powershell-session
PS C:\> Invoke-WebRequest -Uri "https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1" -OutFile "C:\PowerView.ps1"

PS C:\> dir


    Directory: C:\


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----          6/5/2021   5:10 AM                PerfLogs
d-r---         7/25/2022   7:36 AM                Program Files
d-r---          6/5/2021   7:37 AM                Program Files (x86)
d-r---         7/30/2022  10:21 AM                Users
d-----         7/21/2022  11:28 AM                Windows
-a----         8/10/2022   9:12 AM        7299504 PowerView.ps1
```

Invoke-WebRequest'i kullanmak basittir; cmdlet'i ve indirmek istediğimiz kaynağın tam URL'sini belirtiriz:

==-Uri “https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1”==

URL'den sonra, indirdiğimiz kaynağın Windows sistemindeki konumunu ve dosya adını belirtiriz:

==-OutFile “C:\PowerView.ps1”==

Local ağdaki web sunucularından veya Windows hedefinden erişilebilen bir ağdan dosya indirmek için Invoke-WebRequest'i de kullanabiliriz. Saldırı hostumuzdan bir Windows hedefine dosya aktarma ihtiyacı duymak yaygındır. Bunu yapmanın bir faydası, bir pentest sırasında hedeflerimizden biri mümkün olduğunca gizli kalmaksa, ağ güvenlik cihazlarının ağın kenarında tespit edebileceği İnternet'e istek oluşturmamız gerekmeyebilir. PowerView.ps1 dosyasını saldırı hostumuzda depolamış olsaydık, PowerView.ps1 dosyasını barındırmak ve hedeften indirmek için basit bir python web sunucusu kullanabilirdik.


### Tools'u Ortama Getirmek için Örnek Yol
PowerView.ps1 dosyasının saldırı hostumuzda zaten kayıtlı olması durumunda, PowerView.ps1 dosyasını barındırmak ve hedeften indirmek için basit bir Python web sunucusu kullanabiliriz. Saldırı hostundan, dosyanın zaten mevcut olduğunu veya indirmemiz gerektiğini doğrulamak istiyoruz. Bu örnekte, gösterim amacıyla dosyanın zaten saldırı hostunda olduğunu varsayabiliriz.


### Dosyayı Görüntülemek için ls Kullanımı (Saldırı Hostu)
```shell-session
M1R4CKCK@htb[/htb]$ ls

Dictionaries            Get-HttpStatus.ps1                    Invoke-Portscan.ps1          PowerView.ps1  Recon.psd1
Get-ComputerDetail.ps1  Invoke-CompareAttributesForClass.ps1  Invoke-ReverseDnsLookup.ps1  README.md      Recon.psm1
```
PowerView.ps1'in bulunduğu dizinde basit bir python web sunucusu başlatıyoruz.


### Python Web Sunucusunu Başlatma (Saldırı Hostu)
```shell-session
M1R4CKCK@htb[/htb]$ python3 -m http.server 8000

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Ardından, barındırılan dosyayı Invoke-WebRequest kullanarak saldırı hostundan indiririz.


### PowerView.ps1'i Web Sunucusundan İndirme (Saldırı Hostundan Hedef Hosta)
```powershell-session
Invoke-WebRequest -Uri "http://10.10.14.169:8000/PowerView.ps1" -OutFile "C:\PowerView.ps1"
```
Daha önce tartışıldığı gibi, remote host'lara komut göndermek için Invoke-WebRequest cmdlet'ini kullanabiliriz. Bu, özellikle bir Windows hedefinde komutları çalıştırmamıza izin veren ancak etkileşimli bir shell veya remote masaüstü oturumu aracılığıyla erişemeyeceğimiz güvenlik açıklarını keşfettiğimizde oldukça faydalı olabilir. Bu, hedef host'a dosya indirmemize ve bu hedefe erişimimizi ilerletmemize ve ağdaki diğerlerine geçmemize izin verebilir. 

### Invoke-WebRequest'i Kullanamazsak Ne Olur?
Peki, herhangi bir nedenle[ Invoke-WebRequest'i](https://learn.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-7.0) kullanmamız kısıtlanırsa ne olur? Korkmayın, Windows web client'larla etkileşim için birkaç farklı yöntem sunar. İlk ve daha zorlu etkileşim yolu .Net.WebClient sınıfını kullanmaktır. Bu kullanışlı sınıf, Windows .Net kullandığı ve anladığı için kullanabileceğimiz bir .Net çağrısıdır. Bu sınıf, bir URI (github.com/project/tool.ps1 gibi web adresleri) aracılığıyla kaynaklarla etkileşim için standart system.net yöntemlerini içerir. Bir örneğe bakalım:


#### Net.WebClient Download
```powershell-session
PS C:\htb> (New-Object Net.WebClient).DownloadFile("https://github.com/BloodHoundAD/BloodHound/releases/download/4.2.0/BloodHound-win32-x64.zip", "Bloodhound.zip")

PS C:\htb> ls

    Directory: C:\Users\MTanaka

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a---          11/10/2022 10:45 AM      108511752 Bloodhound.zip
-a---           6/14/2022  8:22 AM           4418 passwords.kdbx
-a---            9/9/2020  4:54 PM         696576 Start.exe
-a---           9/11/2021 12:58 PM              0 sticky.gpr
-a---          11/10/2022 10:44 AM      108511752 test.zip
```

Yani işe yaradı. Şimdi ne yaptığımızı inceleyelim:

* İlk olarak Download cradle (New-Object Net.WebClient).DownloadFile()'a sahibiz, bu şekilde ona isteğimizi yerine getirmesini söylüyoruz.
* Daha sonra, indirmek istediğimiz dosyanın URI'sini ()'ye ilk parametre olarak eklememiz gerekir. Bu örnek için bu “https://github.com/BloodHoundAD/BloodHound/releases/download/4.2.0/BloodHound-win32-x64.zip” idi.
* Son olarak, komuta dosyanın nereye yazılmasını istediğimizi ikinci parametre olan “BloodHound.zip” ile söylememiz gerekir.

Yukarıdaki komut dosyayı Bloodhound.zip olarak çalıştığımız dizine indirirdi. Terminalimize baktığımızda, komutun başarıyla çalıştığını görebiliriz çünkü Bloodhound.zip dosyası artık çalışma dizinimizde bulunmaktadır. Eğer başka bir yere yerleştirmek isteseydik, tam yolu belirtmemiz gerekirdi. Buradan araçları çıkarabilir ve uygun gördüğümüz şekilde çalıştırabiliriz. Bunun gürültülü olduğunu unutmayın çünkü dosya okuma ve yazmalarıyla birlikte ağınıza giren ve çıkan web istekleri olacaktır, bu nedenle günlükler bırakacaktır. Örneğin, aktarımlarınız yerel olarak, yalnızca hosttan host'a yapılıyorsa, yalnızca bu hostlarda günlükler bırakırsınız; bu günlükleri elemek biraz daha zordur ve müşteri sınırında giriş / çıkış günlükleri yazmadığımız için daha az iz bırakır.


### Toparlıyoruz
Bu bölüm, web ile etkileşim kurarken PowerShell ile neler yapabileceğimizin sadece yüzeyini çizdi. Biraz zaman ayırdığınızdan ve gönderebileceğiniz farklı istek türlerini ve hatta aldığınız bilgileri filtreleyip kullanabileceğiniz birçok farklı yolu uyguladığınızdan emin olun. Bu noktadan sonra, PowerShell ile otomasyon ve bunun bize nasıl fayda sağlayabileceği hakkında konuşmaya devam edeceğiz.


# PowerShell Scripting and Automation
PowerShell ne kadar inanılmaz olsa da, yalnızca onu nasıl kullandığımız kadar iyidir. PowerShell dilinin ve işlevselliğinin çoğu otomatik bir şekilde kullanılmaya uygundur. PowerShell'de bizim için komut dosyaları ve modüller oluşturma yeteneğine sahip olmak (ne kadar basit veya karmaşık olursa olsun) idari yükümüzü hafifletebilir veya pentesters olarak tabağımızdan bazı kolay görevleri temizleyebilir. Bu modülde bir PowerShell script ve modülünü oluşturan parçalar ve bölümler ele alınacaktır. Sonunda, kendi kullanımı kolay ve özelleştirilebilir modülümüzü oluşturmuş olacağız.


## Understanding PowerShell Scripting
PowerShell, doğası gereği modülerdir ve kullanımı ile önemli miktarda kontrol sağlar. Komut dosyası oluşturma ile ilgili geleneksel düşünce, oluşturulduğu dilde bizim için görevleri yerine getiren bir tür yürütülebilir dosya yazdığımızdır. PowerShell ile bu hala geçerlidir, ancak birkaç farklı dilden ve dosya türünden girdiyi işleyebilir ve birçok farklı nesne türünü işleyebilir. Tekil komut dosyalarını .\script sözdizimini kullanarak çağırarak ve Import-Module cmdlet'ini kullanarak modülleri içe aktararak olağan şekilde kullanabiliriz. Şimdi biraz komut dosyaları ve modüller hakkında konuşalım.

### Scripts vs. Modules
Bunu düşünmenin en kolay yolu, bir komut dosyasının PowerShell cmdlet'lerini ve fonksiyonlarını içeren çalıştırılabilir bir metin dosyası olduğu, bir modülün ise sadece basit bir komut dosyası veya birden fazla komut dosyası, manifesto ve fonksiyonun bir araya getirildiği bir koleksiyon olabileceğidir. Diğer temel fark ise kullanımlarıdır. Genellikle bir komut dosyasını doğrudan çalıştırarak çağırırsınız, oysa bir modülü ve ilişkili tüm komut dosyalarını ve fonksiyonları istediğiniz zaman çağırmak üzere içe aktarabilirsiniz. Bu bölümün iyiliği için, bunları aynı terimi kullanarak tartışacağız ve bir modül dosyasında bahsettiğimiz her şey standart bir PowerShell scriptinde çalışır. İlk olarak dosya uzantıları ve bizim için ne anlama geldikleri.


### File Extensions
PowerShell scriptleri ve modülleri ile çalışırken karşılaşacağımız bazı dosya uzantılarına aşina olmak için, uzantıları ve açıklamalarını içeren küçük bir tablo hazırladık.


#### PowerShell Extensions

* ==ps1== : *.ps1 dosya uzantısı, çalıştırılabilir PowerShell komut dosyalarını temsil eder.
* ==psm1== : *.psm1 dosya uzantısı bir PowerShell modül dosyasını temsil eder. Modülün ne olduğunu ve içinde ne bulunduğunu tanımlar.
* ==psd1== : *.psd1, bir PowerShell modülünün içeriğini key/value çiftleri tablosunda detaylandıran bir PowerShell veri dosyasıdır.

Bunlar şu anda ilgilendiğimiz ana uzantılardır. Gerçekte, PowerShell modülleri çeşitli uzantılara sahip birçok farklı eşlik eden dosyaya sahip olabilir, ancak bunlar yapmaya çalıştığımız şey için gereklilik değildir. PowerShell script dosyaları ve yardım dosyaları hakkında daha derinlemesine bilgi edinmek isterseniz [şu yazıya](https://learn.microsoft.com/en-us/powershell/scripting/developer/module/writing-a-windows-powershell-module?view=powershell-7.2) göz atabilirsiniz

### Modül Oluşturma
Şimdi işe koyulalım. Bu noktadan itibaren, bir PowerShell modülünün bileşenlerini, neleri içerdiklerini ve nasıl oluşturulacaklarını ele alacağız. Bu proses basittir. Sadece biraz ön planlama gerektirir. Bu senaryoyu düşünün:

Senaryo: Hostları yönetirken kendimizi aynı kontrolleri tekrar tekrar yaparken bulduk. Bu nedenle, görevlerimizi hızlandırmak için, kontrolleri bizim için çalıştıracak ve ardından istediğimiz bilgilerin çıktısını verecek bir PowerShell modülü oluşturacağız. Modülümüz kullanıldığında, hostun bilgisayar adının, IP adresinin ve temel alan bilgilerinin çıktısını vermeli ve bize C:\Users\ dizininin çıktısını sağlamalıdır, böylece hangi kullanıcıların bu hostta etkileşimli olarak oturum açtığını görebiliriz.

Artık modülümüze ne gireceğini bildiğimize göre, onu oluşturmaya başlamanın zamanı geldi.


### Modül Bileşenleri
Bir modül dört temel bileşenden oluşur:
1. Gerekli tüm dosyaları ve içeriği içeren bir dizin, $env:PSModulePath içinde bir yere kaydedilir.
* Bu, PowerShell oturumunuza veya Profilinize aktarmaya çalıştığınızda, nerede olduğunu belirtmek yerine otomatik olarak bulunabilmesi için yapılır.

2. Modül ve fonksiyonu hakkındaki tüm dosyaları ve ilgili bilgileri listeleyen bir manifesto dosyası.
* Bu, ilişkili komut dosyalarını, bağımlılıkları, yazarı, örnek kullanımı vb. içerebilir.

3. Bir kod dosyası - genellikle bir PowerShell komut dosyası (.ps1) veya komut dosyası fonksiyonlarımızı ve diğer bilgileri içeren bir (.psm1) modül dosyası.

4. Yardım dosyaları, komut dosyaları ve diğer destekleyici belgeler gibi modülün ihtiyaç duyduğu diğer kaynaklar.

Bu kurulum standart bir uygulamadır ancak kesinlikle gerekli değildir. Modülümüzün, manifesto ve diğer yardımcı dosyaları atlayarak komut dosyalarımızı ve bağlamımızı içeren yalnızca bir *.psm1 dosyası olmasını sağlayabiliriz. PowerShell her iki durumda da ne yapılacağını yorumlayabilir ve anlayabilir. Özelliğin iyiliği için, manifesto dosyası ve bazı yerleşik yardım işlevleri de dahil olmak üzere standart bir PowerShell modülü oluşturmaya çalışacağız.


### Modülümüzü Tutmak İçin Bir Dizin Oluşturmak
Daha önceki bölümlerde tartışıldığı gibi bir dizin oluşturmak çok basittir. Daha ileri gitmeden önce, modülümüzü tutacak dizini oluşturmamız gerekir. Bu dizin $env:PSModulePath içindeki yollardan birinde olmalıdır. Bu yolların ne olduğundan emin değilseniz, en iyi yerin neresi olacağını görmek için değişkeni çağırabilirsiniz. Bu yüzden quick-recon adında bir klasör oluşturacağız.

#### Mkdir
```powershell-session
PS C:\htb> mkdir quick-recon  

    Directory: C:\Users\MTanaka\Documents\WindowsPowerShell\Modules


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
d-----        10/31/2022   7:38 AM                quick-recon
```
Artık dizinimiz olduğuna göre modülü oluşturabiliriz. Bir saniyeliğine modül manifesto dosyasını tartışalım.


### Module Manifest
Modül manifestosu, bir hash tablosu içeren basit bir .psd1 dosyasıdır. Hash tablosundaki key ve values aşağıdaki fonksiyonları yerine getirir:
* Modülün içeriğini ve niteliklerini açıklayın.
* Önkoşulları tanımlayın. (modülün dışından belirli modüller, değişkenler, fonksiyonlar, vb.)
* Bileşenlerin nasıl işlendiğini belirleyin.

Modül klasörüne bir manifesto dosyası eklerseniz, manifestoya başvurarak birden fazla dosyaya tek bir birim olarak başvurabilirsiniz. Manifesto aşağıdaki bilgileri açıklar:

* Meta veriler, modül sürüm numarası, yazar ve açıklama gibi modül hakkında.
* Modülü içe aktarmak için gereken Windows PowerShell sürümü, ortak dil çalışma zamanı (CLR) sürümü ve gerekli modüller gibi ön koşullar.
* İşlenecek komut dosyaları, biçimler ve türler gibi işleme yönergeleri.
* Dışa aktarılacak diğer adlar, işlevler, değişkenler ve cmdlet'ler gibi dışa aktarılacak modül üyeleriyle ilgili kısıtlamalar.

New-ModuleManifest kullanarak ve nereye yerleştirilmesini istediğimizi belirterek hızlı bir şekilde bir manifesto dosyası oluşturabiliriz.


#### New-ModuleManifest
```powershell-session
PS C:\htb> New-ModuleManifest -Path C:\Users\MTanaka\Documents\WindowsPowerShell\Modules\quick-recon\quick-recon.psd1 -PassThru

# Module manifest for module 'quick-recon'
#
# Generated by: MTanaka
#
# Generated on: 10/31/2022
#

@{

# Script module or binary module file associated with this manifest.
# RootModule = ''

# Version number of this module.
ModuleVersion = '1.0'

<SNIP>
```

Yukarıdaki komutu vererek, varsayılan hususlarla doldurulmuş yeni bir manifesto dosyası hazırladık. PassThru değiştiricisi dosyada ve konsolda ne yazdırıldığını görmemizi sağlar. Artık içeri girebilir ve istediğimiz bölümleri ilgili bilgilerle doldurabiliriz. ModuleVersion satırı hariç manifesto dosyalarındaki tüm satırların isteğe bağlı olduğunu unutmayın. Manifestoyu düzenlemek en kolay şekilde bir metin düzenleyici veya VSCode gibi bir IDE kullanabileceğiniz bir GUI'den yapılacaktır. Bu modül için manifesto dosyamızı şimdi tamamlayacak olsaydık, aşağıdaki gibi bir şey görünürdü:

#### Örnek Manifesto
```powershell
# Module manifest for module 'quick-recon'
#
# Generated by: MTanaka
#
# Generated on: 10/31/2022
#

@{

# Script module or binary module file associated with this manifest.
# RootModule = 'C:\Users\MTanaka\WindowsPowerShell\Modules\quick-recon\quick-recon.psm1'

# Version number of this module.
ModuleVersion = '1.0'

# ID used to uniquely identify this module
GUID = '0a062bb1-8a1b-4bdb-86ed-5adbe1071d2f'

# Author of this module
Author = 'MTanaka'

# Company or vendor of this module
CompanyName = 'Greenhorn.Corp.'

# Copyright statement for this module
Copyright = '(c) 2022 Greenhorn.Corp. All rights reserved.'

# Description of the functionality provided by this module
Description = 'This module will perform several quick checks against the host for Reconnaissance of key information.'

# Functions to export from this module, for best performance, do not use wildcards and do not delete the entry, use an empty array if there are no functions to export.
FunctionsToExport = @()

# Cmdlets to export from this module, for best performance, do not use wildcards and do not delete the entry, use an empty array if there are no cmdlets to export.
CmdletsToExport = @()

# Variables to export from this module
VariablesToExport = '*'

# Aliases to export from this module, for best performance, do not use wildcards and do not delete the entry, use an empty array if there are no aliases to export.
AliasesToExport = @()

# List of all modules packaged with this module
# ModuleList = @()

# List of all files packaged with this module
# FileList = @()  
}
```
Bildirime daha sonra geri dönebilir ve dışa aktarmaya izin vermek istediğimiz fonksiyonları, cmdlet'leri ve değişkenleri ekleyebiliriz. Önce komut dosyasını oluşturmamız ve bitirmemiz gerekir.


### Create Our Script File
Dosyamızı oluşturmak için New-Item (ni) cmdlet'ini kullanabiliriz.

#### New-Item
```powershell-session
PS C:\htb>  ni quick-recon.psm1 -ItemType File


    Directory: C:\Users\MTanaka\Documents\WindowsPowerShell\Modules\quick-recon


Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        10/31/2022   9:07 AM              0 quick-recon.psm1
```

Yeterince kolay, değil mi? Şimdi bu canavarı doldurmak için.


### İhtiyacınız Olan Modülleri İçe Aktarma
Yeni PowerShell'imizin düzgün çalışması için içinden başka modüller veya cmdlet'ler gerekiyorsa, kod dosyamızın başına bir Import-Module dizesi yerleştiririz. Bu şekilde Import-Module kullanımı, shell içinden verdiğimizde olduğu gibi fonksiyon görür; script'imizi çalıştırmadan önce ihtiyacımız olan modülleri çağırır ve yükler. Bu modül için belirlediğimiz hedeflere ulaşmak amacıyla, cmdlet'lerin ve fonksiyonların çoğu PowerShell'de zaten yerleşik olarak bulunmaktadır. Ancak ActiveDirectory PowerShell modülünden bir tanesine ihtiyacımız var. Bu yüzden ActiveDirectory modülü için bir içe aktarma satırı ekleyelim.

### Modülümüze İçe Aktarın
```powershell
Import-Module ActiveDirectory 
```

Oldukça basit, değil mi? Şimdi quick-recon.psm1 modül kod dosyamız var ve içine bir import-module ifadesi ekledik. Şimdi dosyanın özüne, yani fonksiyonlarımıza geçebiliriz.


### Fonksiyonlar ve Powershell ile iş yapma
Bu modülle dört ana şey yapmamız gerekiyor:
* Host Bilgisayar Adını al
* Host IP yapılandırmasını al
* Temel domain bilgilerini alma
* “C:\Users” dizininin bir çıktısını alın

Başlamak için ComputerName çıktısına odaklanalım. Bunu çeşitli cmdlet'ler, modüller ve DOS komutları ile birçok şekilde elde edebiliriz. Komut dosyamız, çıktı için host adını almak üzere ortam değişkenini (env:ComputerName) kullanacaktır. Çıktımızın daha sonra okunmasını kolaylaştırmak için, ortam değişkeninden gelen çıktıyı saklamak üzere hostname adlı başka bir değişken kullanacağız. Etkin host adaptörlerinin IP adresini yakalamak için IPConfig kullanacağız ve bu bilgiyi IP değişkeninde saklayacağız. Temel domain bilgileri için Get-ADDomain kullanacağız ve çıktıyı Domain değişkeninde saklayacağız. Son olarak, Get-ChildItem ile C:\Users\ içindeki kullanıcı klasörlerinin bir listesini alacağız ve bunu Users içinde saklayacağız. Değişkenlerimizi oluşturmak için önce ($Hostname) gibi bir isim belirlemeli, “=” sembolünü eklemeli ve ardından tutmasını istediğimiz eylem veya değerleri eklemeliyiz. Örneğin, ihtiyacımız olan ilk değişken, $Hostname, şu şekilde görünecektir: ($Hostname = $env:ComputerName). Şimdi içeri dalalım ve geri kalan değişkenlerimizi kullanım için oluşturalım.

#### Variables
```powershell
Import-Module ActiveDirectory 

$Hostname = $env:ComputerName
$IP = ipconfig 
$Domain = Get-ADDomain  
$Users = Get-ChildItem C:\Users\ 
  
```
Değişkenlerimiz artık tekil komutları veya fonksiyonları çalıştıracak ve gerekli çıktıyı alacak şekilde ayarlanmıştır. Şimdi bu verileri biçimlendirelim ve kendimize güzel bir çıktı verelim. Bunu New-Item ve Add-Content kullanarak sonucu bir dosyaya yazarak yapabiliriz. İşleri kolaylaştırmak için, bu çıktı işlemini Get-Recon adlı çağrılabilir bir fonksiyon haline getireceğiz.

### Bilgilerimizin Çıktısı
```powershell
Import-Module ActiveDirectory

function Get-Recon {  

    $Hostname = $env:ComputerName  

    $IP = ipconfig

    $Domain = Get-ADDomain 

    $Users = Get-ChildItem C:\Users\

    new-Item ~\Desktop\recon.txt -ItemType File 

    $Vars = "***---Hostname info---***", $Hostname, "***---Domain Info---***", $Domain, "***---IP INFO---***",  $IP, "***---USERS---***", $Users

    Add-Content ~\Desktop\recon.txt $Vars
  } 
```

New-Item önce bizim için çıktı dosyamızı oluşturur, ardından çıktımızı biçimlendirmek için bir değişkeni ($Vars) daha nasıl kullandığımıza dikkat edin. Her bir değişkeni çağırıyoruz ve her birinin arasına açıklayıcı bir satır ekliyoruz. Son olarak, Add-Content cmdlet'i topladığımız verileri $Vars sonuçlarını yazarak recon.txt adlı bir dosyaya ekler. Fonksiyonumuz artık şekilleniyor. Daha sonra, başkalarının neyi başarmaya çalıştığımızı ve neden bu şekilde yaptığımızı anlayabilmesi için dosyamıza bazı yorumlar eklememiz gerekir.


### Senaryo İçindeki Yorumlar
(#) PowerShell'e satırın kod veya modül dosyanız içinde bir yorum içerdiğini söyleyecektir. Yorumlarınız birkaç satırı kapsayacaksa, aşağıda görüldüğü gibi birkaç satırı tek bir büyük yorum olarak sarmak için <# ve #> kullanabilirsiniz:

### Yorum Blokları
```powershell
# This is a single-line comment.  

<# This line and the following lines are all wrapped in the Comment specifier. 
Nothing with this window will be ready by the script as part of a function.
This text exists purely for the creator and us to convey pertinent information.

#>  
```


### Yorumlar Eklendi
```powershell
Import-Module ActiveDirectory

function Get-Recon {  
    # Collect the hostname of our PC.
    $Hostname = $env:ComputerName  
    # Collect the IP configuration.
    $IP = ipconfig
    # Collect basic domain information.
    $Domain = Get-ADDomain 
    # Output the users who have logged in and built out a basic directory structure in "C:\Users\".
    $Users = Get-ChildItem C:\Users\
    # Create a new file to place our recon results in.
    new-Item ~\Desktop\recon.txt -ItemType File 
    # A variable to hold the results of our other variables. 
    $Vars = "***---Hostname info---***", $Hostname, "***---Domain Info---***", $Domain, "***---IP INFO---***",  $IP, "***---USERS---***", $Users
    # It does the thing 
    Add-Content ~\Desktop\recon.txt $Vars
  } 
```
Bu kadar basit. Yorumlarla ilgili çok çılgınca bir şey yok. Şimdi, başkalarının modülümüzü nasıl kullanacaklarını anlayabilmeleri için biraz yardım sözdizimi eklememiz gerekiyor.

### Destek İçeriği
PowerShell, kod veya modül için ihtiyacınız olan her şeyi yerleştirmek için bir tür Yorum tabanlı yardım kullanır. Yardım bölümünü oluşturmak ve hatta daha sonra Get-Help kullanarak çağırmak için tanınan anahtar kelimelerle birlikte yukarıda tartıştığımız gibi yorum bloklarını kullanabiliriz. Yerleşim söz konusu olduğunda, burada iki seçeneğimiz var. Yardımı fonksiyonun içine ya da kod içinde fonksiyonun dışına yerleştirebiliriz. Fonksiyonun içine yerleştirmek istersek, fonksiyonun başında, fonksiyonun açılış satırından hemen sonra veya fonksiyonun sonunda, fonksiyonun son eyleminden bir satır sonra olmalıdır. Kodun içine ancak fonksiyonun dışına yerleştirirsek, yardım ile fonksiyon arasında en fazla bir satır olacak şekilde fonksiyonumuzun üzerine yerleştirmeliyiz. PowerShell'de yardım konusunu daha derinlemesine incelemek için bu [makaleye](https://learn.microsoft.com/en-us/powershell/scripting/developer/help/writing-help-for-windows-powershell-scripts-and-functions?view=powershell-7.2) göz atın. Şimdi yardım bölümümüzü tanımlayalım. Şimdilik bunu kodun üst kısmındaki fonksiyonun dışına yerleştireceğiz.


#### Module Help
```powershell
Import-Module ActiveDirectory

<# 
.Description  
This function performs some simple recon tasks for the user. We import the module and issue the 'Get-Recon' command to retrieve our output. Each variable and line within the function and script are commented for our understanding. Right now, this module will only work on the local host from which you run it, and the output will be sent to a file named 'recon.txt' on the Desktop of the user who opened the shell. Remote Recon functions are coming soon!  

.Example  
After importing the module run "Get-Recon"
'Get-Recon


    Directory: C:\Users\MTanaka\Desktop


Mode                 LastWriteTime         Length Name                                                                                                                                        
----                 -------------         ------ ----                                                                                                                                        
-a----         11/3/2022  12:46 PM              0 recon.txt '

.Notes  
Remote Recon functions coming soon! This script serves as our initial introduction to writing functions and scripts and making PowerShell modules.  

#>

function Get-Recon {  
<SNIP>  
```

Anahtar kelime kullanımımıza dikkat edin. Yorum bloğu içinde bir anahtar kelime belirtmek için .keyword sözdizimini kullanırız ve ardından açıklama metnini altına yerleştiririz. Biz sadece Açıklama, Örnek ve Notlar'ı belirttik, ancak yardım bloğuna birkaç anahtar kelime daha yerleştirilebilir. Mevcut tüm anahtar sözcükleri görmek için [Comment Based Help Keywords](https://learn.microsoft.com/en-us/powershell/scripting/developer/help/comment-based-help-keywords?view=powershell-7.2) hakkındaki bu makaleye bakın. Her şeyi güzel PowerShell Modülü dosyamızın içine yerleştirmeden önce tartışacağımız son kısım, fonksiyonları Dışa Aktarma ve Korumadır.


### Protecting Functions
Komut dosyalarımıza, PowerShell içindeki diğer komut dosyaları veya prosesler tarafından erişilmesini, dışa aktarılmasını veya kullanılmasını istemediğimiz fonksiyonlar ekleyebiliriz. Bir işlevin dışa aktarılmasını önlemek veya dışa aktarma için açıkça ayarlamak için, Export-ModuleMember bu iş için cmdlet'tir. Bunu kod modüllerimizin dışında bırakırsak içerik dışa aktarılabilir. Dosyaya yerleştirir ancak aşağıdaki gibi boş bırakırsak:

### Export Dışında Bırak
```powershell
Export-ModuleMember  
```
Modülün değişkenlerinin, takma adlarının ve fonksiyonlarının export edilememesini sağlar. Neyin export edileceğini belirtmek istersek, bunları komut dizesine aşağıdaki gibi ekleyebiliriz:

### Export'a Özel Fonksiyonlar ve Değişkenler
```powershell
Export-ModuleMember -Function Get-Recon -Variable Hostname 
```

Alternatif olarak, örneğin yalnızca tüm fonksiyonları ve belirli bir değişkeni dışa aktarmak istiyorsanız, -Function komutundan sonra * komutunu verebilir ve ardından dışa aktarılacak Değişkenleri açıkça belirtebilirsiniz. Şimdi Export-ModuleMember cmdlet'ini kodumuza ekleyelim ve Get-Recon fonksiyonumuzun ve Hostname değişkenimizin dışa aktarılabilmesine izin vermek istediğimizi belirtelim.

### Export Line Addition
```powershell
<SNIP>  
function Get-Recon {  
    # Collect the hostname of our PC
    $Hostname = $env:ComputerName  
    # Collect the IP configuration
    $IP = ipconfig
    # Collect basic domain information
    $Domain = Get-ADDomain 
    # Output the users who have logged in and built out a basic directory structure in "C:\Users"
    $Users = Get-ChildItem C:\Users\
    # Create a new file to place our recon results in
    new-Item ~\Desktop\recon.txt -ItemType File 
    # A variable to hold the results of our other variables 
    $Vars = "***---Hostname info---***", $Hostname, "***---Domain Info---***", $Domain, "***---IP INFO---***",  $IP, "***---USERS---***", $Users
    # It does the thing 
    Add-Content ~\Desktop\recon.txt $Vars
  } 

Export-ModuleMember -Function Get-Recon -Variable Hostname  
```


### Scope
Komut dosyaları, PowerShell oturumu ve Komut Satırında bir şeylerin nasıl tanındığı ile uğraşırken, Scope kavramı devreye girer. Scope, özünde, PowerShell'in oturum içindeki nesneleri yetkisiz erişime veya değişikliğe karşı nasıl tanıdığı ve koruduğudur. PowerShell şu anda üç farklı Scope seviyesi kullanmaktadır:

#### Scope Levels
![Pasted image 20241014225147.png](/img/user/resimler/Pasted%20image%2020241014225147.png)

Bu, script'i çalıştırdığımız scope dışındaki herhangi bir şeyin içeriğine erişmesini istemiyorsak bizim için önemlidir. Ek olarak, ana scope'lar içinde oluşturulan alt scope'lara sahip olabiliriz. Örneğin, bir komut dosyasını çalıştırdığınızda, komut dosyası scope'u örneklenir ve ardından çağrılan herhangi bir fonksiyon, bu fonksiyonu ve içerdiği değişkenleri çevreleyen bir alt scope da oluşturabilir. Söz konusu fonksiyonun içeriğine script'in geri kalanının veya PowerShell oturumunun kendisinin erişememesini sağlamak istiyorsak, fonksiyonun scope'unu değiştirebiliriz. Bu karmaşık bir konudur ve şu anda bu modülün seviyesinin üzerinde bir konudur, ancak bahsetmeye değer olduğunu düşündük. PowerShell'de Scope hakkında daha fazla bilgi için [buradaki](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_scopes?view=powershell-7.2) belgelere göz atın.


### Hepsini Bir Araya Getirmek
Parçalarımızı ve kısımlarımızı gözden geçirip oluşturduğumuza göre şimdi hepsini bir arada görelim.

#### Final Product
```powershell
import-module ActiveDirectory

<# 
.Description  
This function performs some simple recon tasks for the user. We import the module and then issue the 'Get-Recon' command to retrieve our output. Each variable and line within the function and script are commented for your understanding. Right now, this only works on the local host from which you run it, and the output will be sent to a file named 'recon.txt' on the Desktop of the user who opened the shell. Remote Recon functions coming soon!  

.Example  
After importing the module run "Get-Recon"
'Get-Recon


    Directory: C:\Users\MTanaka\Desktop


Mode                 LastWriteTime         Length Name                                                                                                                                        
----                 -------------         ------ ----                                                                                                                                        
-a----         11/3/2022  12:46 PM              0 recon.txt '

.Notes  
Remote Recon functions coming soon! This script serves as our initial introduction to writing functions and scripts and making PowerShell modules.  

#>
function Get-Recon {  
    # Collect the hostname of our PC
    $Hostname = $env:ComputerName  
    # Collect the IP configuration
    $IP = ipconfig
    # Collect basic domain information
    $Domain = Get-ADDomain 
    # Output the users who have logged in and built out a basic directory structure in "C:\Users"
    $Users = Get-ChildItem C:\Users\
    # Create a new file to place our recon results in
    new-Item ~\Desktop\recon.txt -ItemType File 
    # A variable to hold the results of our other variables 
    $Vars = "***---Hostname info---***", $Hostname, "***---Domain Info---***", $Domain, "***---IP INFO---***",  $IP, "***---USERS---***", $Users
    # It does the thing 
    Add-Content ~\Desktop\recon.txt $Vars
  } 

Export-ModuleMember -Function Get-Recon -Variable Hostname 
```

Ve işte tam modül dosyamız. Yorum tabanlı yardım, fonksiyonlar, değişkenler ve içerik koruması kullanımımız dinamik ve okunması kolay bir komut dosyası oluşturuyor. Buradan bu dosyayı oluşturduğumuz Modül dizinimize kaydedebilir ve kullanmak için PowerShell içinden içe aktarabiliriz.


### Modülü Kullanım İçin İçe Aktarma
```powershell-session
PS C:\htb> Import-Module 'C:\Users\MTanaka\Documents\WindowsPowerShell\Modules\quick-recon.psm1`

PS C:\Users\MTanaka\Documents\WindowsPowerShell\Modules\quick-recon> get-module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Management     {Add-Computer, Add-Content, Checkpoint-Computer, Clear-Con...
Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...
Script     0.0        quick-recon                         Get-Recon
```

Mükemmel. Modülümüzün Import-Module cmdlet'i kullanılarak içe aktarıldığını görebiliyoruz ve oturumumuza yüklendiğinden emin olmak için Get-Module cmdlet'ini çalıştırdık. Bu bize quick-recon modülümüzün içe aktarıldığını ve dışa aktarılabilecek Get-Recon komutuna sahip olduğunu gösterdi. Ayrıca Get-Help'i modülümüzde çalıştırmayı deneyerek Yorum tabanlı yardımı test edebiliriz.


#### Help Validation
```powershell-session
PS C:\htb> get-help get-recon

NAME
    Get-Recon

SYNOPSIS


SYNTAX
    Get-Recon [<CommonParameters>]


DESCRIPTION
    This function performs some simple recon tasks for the user. We simply import the module and then issue the
    'Get-Recon' command to retrieve our output. Each variable and line within the function and script are commented for
    your understanding. Right now, this only works on the local host from which you run it, and the output will be sent
    to a file named 'recon.txt' on the Desktop of the user who opened the shell. Remote Recon functions coming soon!


RELATED LINKS

REMARKS
    To see the examples, type: "get-help Get-Recon -examples."
    For more information, type: "get-help Get-Recon -detailed."
    For technical information, type: "get-help Get-Recon -full."
```
Yardımımız da çalışıyor. Yani artık kullanımımız için tamamen fonksiyonel bir modülümüz var. Bunu daha sonra oluşturacağımız herhangi bir şey için temel olarak kullanabiliriz ve hatta bunu gelecekte daha fazla keşif fonksiyonunu kapsayacak şekilde değiştirebiliriz.

Bu, PowerShell ile otomasyon perspektifinden neler yapılabileceğinin basit bir örneğiydi, ancak inşa edildiğini ve kullanıldığını görmenin harika bir yoluydu. Modül oluşturma ve komut dosyası yazmayı kendi avantajımıza kullanabilir ve ilerledikçe süreçlerimizi basitleştirebiliriz. Nihayetinde zamandan tasarruf etmek, operatörler olarak daha fazlasını yapmamızı ve dikkatimizi vermemiz gereken diğer görevlere zaman ayırmamızı sağlar.


### Bu Modülün Ötesinde
Genç bir sızma test uzmanı veya sistem yöneticisi olarak, bu modüldeki görevlerin harekete geçmeniz için günlük görevler olması beklenebilir. Bazen doğrudan rehberlik ve gözetim altında, bazen de değil. Windows'un, CLI'sının ve bunlardan ne elde edebileceğimizin (erişim ve numaralandırma açısından) derinlemesine anlaşılması, rolün görevlerini yerine getirmek için çok önemlidir. Beceri değerlendirmesinin sonunda, GUI'ye erişim olmadan bir Windows ortamına erişme ve bu ortamda çalışma konusunda kendimizi rahat hissetmeliyiz. İdari görevlerle ilgilenmek için CLI kullanmak iş akışımızı artırabilir ve daha fazla dikkat gerektiren şeyler üzerinde çalışmaya daha fazla zaman ayırmamızı sağlayabilir. Windows'un geniş dünyasında öğrenme yolculuğumuza devam etmek için bundan sonra ne yapabiliriz?

## What's Next?
Kendi özel oluşturulmuş Windows hostlarınızı nasıl yapılandıracağınızı ve otomatikleştireceğinizi daha iyi anlamak için Kurulum modülüne göz atın. Ayrıca, AD ortamlarının nasıl çalıştığını anlamak için Active Directory'ye Giriş modülüne göz atın. Bir yönetici ve pentester olarak, birçok kuruluşun ortamında dizin hizmetleri için bir standart olduğundan, nerede olursanız olun şüphesiz Active Directory ile tekrar tekrar karşılaşacaksınız.


### Daha Fazla Windows Öğrenme Fırsatı