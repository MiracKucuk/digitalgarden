---
{"dg-publish":true,"permalink":"/active-directory-expert/blood-hound/"}
---


## BloodHound Setup and Installation

BloodHound, verileri grafik olarak temsil etmek, depolamak ve sorgulamak için tasarlanmış bir **graf veri tabanı yönetim sistemi** olan [**Neo4j**](https://neo4j.com/) kullanır. Neo4j, **NoSQL** tabanlı bir veri tabanıdır ve verileri **graf veri modeli** kullanarak depolar. Bu modelde **`nodes`** verileri, **`edges`** ise bu veriler arasındaki ilişkileri temsil eder. Bu yaklaşım, Neo4j'nin **geleneksel ilişkisel veri tabanlarına kıyasla** **daha karmaşık ve bağlantılı veri yapılarının** daha sezgisel ve verimli bir şekilde temsil edilmesini sağlar.

Neo4j, **Java** ile yazılmıştır ve çalıştırılabilmesi için **Java Virtual Machine (JVM)** gerektirir.

**BloodHound**, **Windows, Linux ve macOS** üzerinde kurulabilir. Bunun için öncelikle **Java ve Neo4j** yüklememiz, ardından **BloodHound GUI**'yi indirmemiz gerekir. BloodHound GUI'yi kaynak koddan derlemek de mümkündür, ancak bu bölümde o adıma değinmeyeceğiz. Eğer kaynak koddan derlemek isterseniz, **[BloodHound official documentation](https://bloodhound.readthedocs.io/en/latest/index.html)**'ı inceleyebilirsiniz.

**Kurulum 3 adımda gerçekleştirilir:**

1. **Java** yükleme
2. **Neo4j** yükleme
3. **BloodHound** yükleme

Neo4j database'i başlatmak için:

```
net start neo4j
```

BloodHound GUI'yi çalıştırmak için:

```
C:\Tools\bloodhound.exe
```


### Windows Installation

Öncelikle **[Java Oracle JDK 11](https://www.oracle.com/java/technologies/javase-jdk11-downloads.html)**'i indirip yüklememiz gerekiyor. **Java**'yı indirmeden önce web sitesinde bir hesap oluşturmamız gerekmektedir. **Kurulum dosyasını** indirdikten sonra, aşağıdaki komutu kullanarak **sessiz kurulum** gerçekleştirebiliriz:


#### Install Java Silently

```
PS C:\htb> .\jdk-11.0.17_windows-x64_bin.exe /s
```

Sonraki adımda **Neo4j**'yi kurmamız gerekiyor. **[Neo4j Download Center](https://neo4j.com/download-center/#community)** üzerinden mevcut sürümlerin tam listesini görüntüleyebiliriz. Burada **[Neo4j 4.4](https://go.neo4j.com/download-thanks.html?edition=community&release=4.4.16&flavour=winzip)** sürümünü kullanacağız. Bu yazının yazıldığı tarihteki en güncel sürüm **Neo4j 4.4.16**'dır.

**İndirme işlemi tamamlandıktan sonra**, **yönetici yetkileriyle çalışan** **PowerShell**'i açarak dosyanın içeriğini çıkartabiliriz:


#### Unzip Neo4j

```
PS C:\htb> Expand-Archive .\neo4j-community-4.4.16-windows.zip .
```

**Not:** **Neo4j 5**, en son sürüm olmasına rağmen ciddi **performans gerileme sorunları** yaşadığı için bu sürümü kullanmıyoruz. Daha fazla bilgi için **BloodHound Resmi Dokümantasyonu**'nu ziyaret edebilirsiniz.

Daha sonra, Neo4j'yi yüklememiz gerekiyor. Bir servis olarak yüklemek için .`\neo4jcommunity-*\bin\` dizinine gitmemiz ve aşağıdaki `neo4j.bat installservice komutunu` çalıştırmamız gerekiyor:


#### Install Neo4j Service

```
PS C:\htb> .\neo4j-community-4.4.16\bin\neo4j.bat install-service Neo4j service installed.
```


**Not: Bu noktada, Java'nın bulunamadığına veya yanlış Java sürümünün çalıştırıldığına dair bir hata görebiliriz. **JAVA_HOME** ortam değişkeninin JDK klasörüne ayarlandığından emin olun (örneğin: `C:\Program Files\Java\jdk-11.0.17`). Bu işlem kurulumdan sonra otomatik olarak yapılır, ancak kurulum başarısız olursa her şeyin doğru şekilde yapılandırıldığından emin olmamız gerekir.

Servis kurulduktan sonra, servisi başlatabiliriz:

#### Start Service

```
PS C:\htb> net start neo4j
The Neo4j Graph Database - neo4j service is starting..
The Neo4j Graph Database - neo4j service was started successfully.
```



### Configure Neo4j Database

Neo4j veritabanını yapılandırmak için bir web tarayıcısı açın ve Neo4j web konsoluna şu adresten gidin: [http://localhost:7474/](http://localhost:7474/)

![Pasted image 20250219001317.png](/img/user/Pasted%20image%2020250219001317.png)

Web konsolunda Neo4j'ye, kullanıcı adı olarak `neo4j` ve şifre olarak `neo4j` ile kimlik doğrulaması yapın, veritabanını boş bırakın ve şifre değiştirmeniz istendiğinde şifreyi değiştirin.

![Pasted image 20250219001346.png](/img/user/Pasted%20image%2020250219001346.png)


### Download BloodHound GUI

1. En son sürüm BloodHound GUI'yi Windows için [https://github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases) adresinden indirin.

![Pasted image 20250219001429.png](/img/user/Pasted%20image%2020250219001429.png)

Not: Tarayıcıdan veya antivirüs yazılımından dosyanın zararlı olduğuna dair bir uyarı alabiliriz. Uyarıyı görmezden gelip indirmeye izin verin.

1. Klasörü açın ve BloodHound.exe dosyasına çift tıklayın.
2. neo4j için ayarladığınız kimlik bilgileriyle kimlik doğrulaması yapın.


![Pasted image 20250219001521.png](/img/user/Pasted%20image%2020250219001521.png)


### Linux Installation

İlk olarak, Java Oracle JDK 11'i indirmemiz ve kurmamız gerekiyor. Doğru paketi kurmak için apt kaynaklarımızı güncelleyeceğiz:


#### Updating APT sources to install Java

```
[!bash!]# echo "deb http://httpredir.debian.org/debian stretch-backports main" | sudo tee -a /etc/apt/sources.list.d/stretch-backports.list

[!bash!]# sudo apt-get update
...SNIP...
```

Bu güncelleme ile, eğer Java kurulu değilse, Neo4j'yi kurmaya çalıştığımızda otomatik olarak Neo4j kurulumu ile birlikte yüklenecektir. Şimdi Neo4j kurulumu için apt kaynaklarını ekleyelim:


#### Updating APT sources to install Neo4j

```
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -

echo 'deb https://debian.neo4j.com stable 4.4' | sudo tee -a /etc/apt/sources.list.d/neo4j.list

sudo apt-get update 
...SNIP...
```

Neo4j'yi kurmadan önce, apt ile `apt-transport-https` paketini kurmamız gerekiyor.


#### Installing required packages

```
sudo apt-get install apt-transport-https
...SNIP...
```

Şimdi Neo4j'yi kurabiliriz. İlk olarak mevcut seçenekleri listeleyelim ve en son 4.4.X sürümünü seçelim.


#### Installing Neo4j

```
sudo apt list -a neo4j
sudo apt list -a neo4j
Listing... Done
neo4j/stable 1:5.3.0 all [upgradable from: 1:4.4.12]
neo4j/stable 1:5.2.0 all
neo4j/stable 1:5.1.0 all
neo4j/stable 1:4.4.16 all
neo4j/stable 1:4.4.15 all
neo4j/stable 1:4.4.14 all
neo4j/stable 1:4.4.13 all
neo4j/stable,now 1:4.4.12 all [installed,upgradable to: 1:5.3.0]
neo4j/stable 1:4.4.11 all
neo4j/stable 1:4.4.10 all
neo4j/stable 1:4.4.9 all
...SNIP...

```

Yazıldığı sırada en son sürüm Neo4j 4.4.16'dır, bu sürümü aşağıdaki komutla kurabiliriz:


#### Installing Neo4j 4.4.X

```
sudo apt install neo4j=1:4.4.16 -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
 neo4j
1 upgraded, 0 newly installed, 0 to remove, and 236 not upgraded.
Need to get 106 MB of archives.
After this operation, 1,596 kB of additional disk space will be used.
Get:1 https://debian.neo4j.com stable/4.4 amd64 neo4j all 1:4.4.16 [106
MB]
Fetched 106 MB in 2s (55.9 MB/s)
...SNIP...

```

Sonraki adımda, Java 11 kullandığından emin olmamız gerekiyor. İşletim sistemimizin hangi Java sürümünü kullanacağını aşağıdaki komutla güncelleyebiliriz:


#### Change Java version to 11

```
sudo update-alternatives --config java
There are 2 choices for the alternative java (providing /usr/bin/java).
 Selection Path Priority
Status
------------------------------------------------------------
 0 /usr/lib/jvm/java-13-openjdk-amd64/bin/java 1311
auto mode
* 1 /usr/lib/jvm/java-11-openjdk-amd64/bin/java 1111
manual mode
 2 /usr/lib/jvm/java-13-openjdk-amd64/bin/java 1311
manual mode
Press <enter> to keep the current choice[*], or type selection number: 1
```

Not: Seçenek 1, Java 11'e karşılık gelir. Seçenek, sisteminizde farklı olabilir. Neo4j'yi hata almadan başlatıp başlatmadığını doğrulamak için bir konsol uygulaması olarak başlatabiliriz:


#### Running Neo4j as console

```
cd /usr/bin
sudo ./neo4j console
Directories in use:
home: /var/lib/neo4j
config: /etc/neo4j
logs: /var/log/neo4j
plugins: /var/lib/neo4j/plugins
import: /var/lib/neo4j/import
data: /var/lib/neo4j/data
certificates: /var/lib/neo4j/certificates
licenses: /var/lib/neo4j/licenses
run: /var/lib/neo4j/run
Starting Neo4j.
2023-01-05 20:04:26.679+0000 INFO Starting...
2023-01-05 20:04:27.369+0000 INFO This instance is ServerId{fb3f5e13}
(fb3f5e13-5dfd-49ee-b068-71ad7f5ce997)
2023-01-05 20:04:29.103+0000 INFO ======== Neo4j 4.4.16 ========
2023-01-05 20:04:30.562+0000 INFO Performing postInitialization step for
component 'security-users' with version 3 and status CURRENT
2023-01-05 20:04:30.562+0000 INFO Updating the initial password in
component 'security-users'
2023-01-05 20:04:30.862+0000 INFO Bolt enabled on localhost:7687.
2023-01-05 20:04:31.881+0000 INFO Remote interface available at
http://localhost:7474/
2023-01-05 20:04:31.887+0000 INFO id:
613990AF56F6A7BDDA8F79A02F0ACED758E04015C5B0809590687C401C98A4BB
2023-01-05 20:04:31.887+0000 INFO name: system
2023-01-05 20:04:31.888+0000 INFO creationDate: 2022-12-12T15:59:25.716Z
2023-01-05 20:04:31.888+0000 INFO Started.

```

Servisi başlatmak ve durdurmak için aşağıdaki komutları kullanabiliriz:


#### Start Neo4j

```
sudo systemctl start neo4j
```


#### Stop Neo4j

```
sudo systemctl stop neo4j
```

Not: Neo4j'u genellikle bir Linux sisteminde barındırmak, ancak BloodHound GUI'yi farklı bir sistemde kullanmak yaygındır. Neo4j, varsayılan olarak yalnızca local bağlantılara izin verir. Remote olarak bağlantılara izin vermek için, `/etc/neo4j/neo4j.conf` dosyasını açın ve bu satırı düzenleyin:

```
#dbms .default_listen_address=0.0.0.0
```

Satırın başındaki `#` karakterini kaldırarak yorumu açın. Dosyayı kaydedin, ardından neo4j'yi tekrar başlatın.


### Configure Neo4j Database

Neo4j veritabanarını yapılandırmak için Windows'ta yaptığımız aynı adımları uygulayacağız:  

Bir web tarayıcısı açın ve Neo4j web konsoluna [http://localhost:7474/](http://localhost:7474/) adresinden erişin.

![Pasted image 20250219002538.png](/img/user/Pasted%20image%2020250219002538.png)

Neo4j varsayılan kimlik bilgilerini değiştirin. Web konsolunda `neo4j` kullanıcı adı ve `neo4j` şifresiyle oturum açın, veritabanını boş bırakın ve şifre değiştirmeniz istendiğinde yeni şifreyi belirleyin.

![Pasted image 20250219002601.png](/img/user/Pasted%20image%2020250219002601.png)


### Download BloodHound GUI

1. BloodHound GUI'nin en son sürümünü Linux için [https://github.com/BloodHoundAD/BloodHound/releases](https://github.com/BloodHoundAD/BloodHound/releases) adresinden indirin.

![Pasted image 20250219002716.png](/img/user/Pasted%20image%2020250219002716.png)

1. Klasörü zipten çıkarın, ardından BloodHound'u --no-sandbox bayrağı ile çalıştırın:


#### Unzip BloodHound

```
unzip BloodHound-linux-x64.zip
Archive: BloodHound-linux-x64.zip
 creating: BloodHound-linux-x64/
 inflating: BloodHound-linux-x64/BloodHound
 ...SNIP...

```


#### Execute BloodHound

```
cd BloodHound-linux-x64/
./BloodHound --no-sandbox
```

1. Neo4j için ayarladığınız kimlik bilgileri ile giriş yapın.

![Pasted image 20250219002907.png](/img/user/Pasted%20image%2020250219002907.png)


### MacOS Install

BloodHound'u MacOS'a yüklemek için [BloodHound resmi belgelerinde](https://bloodhound.readthedocs.io/en/latest/index.html) verilen adımları takip edebiliriz.


### Updating BloodHound requirements (Linux)

BloodHound'u zaten yüklediysek ve en son sürümü destekleyecek şekilde güncellememiz gerekiyorsa, aşağıdaki komutlarla Neo4j ve Java'yı güncelleyebiliriz:


#### Update Neo4j

```
wget -O - https://debian.neo4j.com/neotechnology.gpg.key | sudo apt-key add -

echo 'deb https://debian.neo4j.com stable 4.4' | sudo tee -a /etc/apt/sources.list.d/neo4j.list

sudo apt-get update
...SNIP...
```



#### Install Neo4j 4.4.X

```
sudo apt install neo4j=1:4.4.16 -y
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following packages will be upgraded:
 neo4j
1 upgraded, 0 newly installed, 0 to remove, and 236 not upgraded.
Need to get 106 MB of archives.
After this operation, 1,596 kB of additional disk space will be used.
Get:1 https://debian.neo4j.com stable/4.4 amd64 neo4j all 1:4.4.16 [106
MB]
Fetched 106 MB in 2s (55.9 MB/s)
...SNIP...

```

Not: Kurulum adımlarında belirtildiği gibi Java sürümünü 11 olarak değiştirdiğinizden emin olun.


### Recovering Neo4j Credentials

Eğer Neo4j veritabanına varsayılan kimlik bilgileriyle erişemiyorsak, varsayılan kimlik bilgilerini sıfırlamak için şu adımları izleyebiliriz:

1. Neo4j'yi çalışıyorsa durdurun

```
sudo systemctl stop neo4j
```

1. /etc/neo4j/neo4j.conf dosyasını düzenleyin ve `dbms.security.auth_enabled=false` satırını yorumdan çıkarın.
2. Neo4j konsolunu başlatın:

```
sudo neo4j console
Directories in use:
home: /var/lib/neo4j
config: /etc/neo4j
logs: /var/log/neo4j
plugins: /var/lib/neo4j/plugins
import: /var/lib/neo4j/import
data: /var/lib/neo4j/data
certificates: /var/lib/neo4j/certificates
licenses: /var/lib/neo4j/licenses
run: /var/lib/neo4j/run
Starting Neo4j.
2023-01-05 20:49:46.214+0000 INFO Starting
...SNIP...

```

1. [http://localhost:7474/](http://localhost:7474/) adresine gidin ve kimlik bilgisi olmadan giriş yapmak için "Connect" seçeneğine tıklayın.
2. Neo4j hesabı için yeni bir şifre ayarlamak için şu sorguyu çalıştırın: `ALTER USER neo4j SET PASSWORD 'Password123';`

![Pasted image 20250219003322.png](/img/user/Pasted%20image%2020250219003322.png)

1. Neo4j servisini durdurun.
2. `/etc/neo4j/neo4j.conf` dosyasını düzenleyin ve `dbms.security.auth_enabled=false` satırını yorum satırı haline getirin.
3. Neo4j'i başlatın ve yeni şifreyi kullanın.


### Next Steps

Aşağıdaki bölümlerde, BloodHound ile çalışmaya başlayacak ve Active Directory'den veri toplamak için SharpHound'u kullanacağız.


## BloodHound Overview

### Active Directory Access Management

Active Directory'de erişim yönetimi karmaşıktır ve günlük yapılandırmalarda zafiyetler veya kötü uygulamalar kolayca ortaya çıkabilir.

Hem saldırganlar hem de savunmacılar, bir kullanıcının tüm erişimlerini keşfetmede veya denetlemede ve bu erişimlerin nasıl birbirine bağlı olduğunu anlamada genellikle zorluk yaşar. Bu bağlantılar, kullanıcıya aslında sahip olmaması gereken ayrıcalıkları kazandırabilir.

Active Directory ortamlarında saldırı yüzeyi ve üretilen veri miktarı oldukça karmaşıktır ve sürekli gelişmektedir. Bu nedenle, bu verilerin toplanmasını ve analiz edilmesini otomatikleştirecek bir yönteme ihtiyaç duyuldu. İşte bu noktada, [@_wald0](https://www.twitter.com/_wald0), [@harmj0y](https://twitter.com/harmj0y) ve [@CptJesus](https://twitter.com/CptJesus) tarafından [BloodHound](https://github.com/BloodHoundAD/BloodHound) geliştirildi.


### BloodHound Overview

BloodHound, saldırganlar ve savunmacılar tarafından Active Directory domain güvenliğini analiz etmek için kullanılan açık kaynaklı bir araçtır. Bu araç, Active Directory domaininden büyük miktarda veri toplar. Geleneksel enumarasyon yöntemleriyle tespit edilmesi zor veya imkânsız olan domain saldırı yollarını belirlemek için **graf teorisini** kullanarak objeler arasındaki ilişkileri görsel olarak temsil eder.

BloodHound, **4.0 sürümünden itibaren Azure desteği** de sunmaktadır. Bu modülün ana odağı **Active Directory** olacak olsa da, [Azure Enumeration](https://academy.hackthebox.com/module/69/section/2070) bölümünde **AzureHound**’a da giriş yapacağız.

BloodHound tarafından kullanılacak veri, **[SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors)** **collector** kullanılarak elde edilir. **PowerShell** ve **C#** ile kullanılabilen bu **collector**, veri toplama işlemini gerçekleştirmek için kullanılır. Veri toplama sürecini ilerleyen bölümlerde ele alacağız.


### BloodHound Graph Theory & Cypher Query Language

**BloodHound**, [**Graph Theory**](https://en.wikipedia.org/wiki/Graph_theory)'yi kullanır; bu, **object**'ler arasındaki ikili ilişkileri modellemek için kullanılan matematiksel yapılardır. Bu bağlamda bir **graph**, **[node](https://bloodhound.readthedocs.io/en/latest/data-analysis/nodes.html)**'lardan (**Active Directory object**'leri, örneğin **user**, **group**, **computer**, vb.) oluşur ve **[edge](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html)**'ler ile bağlanır (**object**'ler arasındaki ilişkiler, örneğin **MemberOf**, **AdminTo**, vb.). **Node** ve **edge** kavramlarını ilerleyen bölümlerde daha ayrıntılı ele alacağız, ancak **BloodHound**'un nasıl çalıştığını görmek için bir örnek yapalım.

Araç, ilişkileri analiz etmek için **[Cypher Query Language](https://neo4j.com/docs/getting-started/current/cypher-intro/)** kullanır. **Cypher**, **Neo4j**'in **graph query language**'idir ve **graph**'tan veri çekmemizi sağlar. **SQL**'e benzer şekilde tasarlandığından, sorguların nasıl çalıştığına değil, hangi veriyi çekmek istediğimize odaklanmamıza olanak tanır. Öğrenilmesi en kolay **graph query language** olarak kabul edilir, çünkü diğer dillerle benzerliği ve sezgiselliği sayesinde anlaşılması kolaydır. **Cypher query**'leri ile ilgili detayları ilerleyen bölümlerde ele alacağız.

Aşağıdaki **diagram**, iki **node** (**A** ve **B**) içermektedir. Bu örnekte yalnızca **node A**'dan **node B**'ye gidebiliriz; tersi mümkün değildir.

![Pasted image 20250219005238.png](/img/user/Pasted%20image%2020250219005238.png)

Bu, **A**'yı **user** **Grace**, **B**'yi ise **group** **SQL Admins** olarak simüle edebilir. İkisi arasındaki çizgi **edge**'i temsil eder ve bu durumda **MemberOf** ilişkisidir. Aşağıdaki **graphic**, **BloodHound**'da **user** **Grace**'in **SQL Admins** **group**'unun bir üyesi olduğunu gösterir.

![Pasted image 20250219005354.png](/img/user/Pasted%20image%2020250219005354.png)

Hadi **nodes** arasındaki daha karmaşık bir ilişkiyi inceleyelim. Aşağıdaki **graphic**, sekiz (**8**) **node** ve on (**10**) **edge** içeriyor. **Node H**, **node G**'ye ulaşabilir, ancak hiçbir **node**'un **node H**'ye doğrudan bir yolu yoktur. **Node A**'dan **node C**'ye gitmek için önce **node G**'ye, ardından **node F**'ye ve son olarak **node C**'ye geçebiliriz, ancak bu en kısa yol değildir. **BloodHound**'un yeteneklerinden biri de en kısa yolu bulmaktır. Bu örnekte, **node A**'dan **node C**'ye en kısa yol, **node B** üzerinden tek bir sıçrama yapmaktır.

![Pasted image 20250219005635.png](/img/user/Pasted%20image%2020250219005635.png)

Önceki örnekte, **BloodHound**'u kullanarak **Grace**'in **SQL Admins** grubunun bir üyesi olduğunu keşfettik, ki bu oldukça basit bir bilgidir. Aynı bilgiyi **Active Directory Users and Computers GUI** veya `net user grace /domain` komutuyla da elde edebiliriz. Yalnızca bu bilgiye dayanarak, **Grace**'in **Domain Admins** grubuna doğrudan bir yolu olmadığını düşünebiliriz. Ancak, **BloodHound** burada devreye girerek **nodes** arasındaki keşfedilmesi zor ilişkileri bulmamıza yardımcı olur.

Şimdi **BloodHound**'u bir harita gezgini olarak kullanıp, **Grace**'in **Domain Admins** grubuna nasıl ulaşabileceğini soralım. İşte sonuç:

![Pasted image 20250219005741.png](/img/user/Pasted%20image%2020250219005741.png)

Bu, **`Grace`**'in **`SQL Admins`** grubunun bir üyesi olarak **`Peter`**'ın şifresini değiştirme yetkisine sahip olduğu anlamına gelir. **`Peter`**'ın yeni şifresiyle kimlik doğrulaması yaparak **`Domain Admins`** grubunun bir üyesi gibi işlem gerçekleştirebiliriz. **`Peter`** doğrudan bir üye olmasa da, **Domain Admins** grubunun üyesi olan bir grubun içinde yer almaktadır.


### BloodHound for Enterprise

