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

**BloodHound**'u geliştiren **[SpecterOps](https://specterops.io/)** ekibi, ayrıca **[BloodHound Enterprise](https://bloodhoundenterprise.io/)**'ı da oluşturdu. Bu, **Active Directory Attack Path**'lerini sürekli olarak haritalayan ve ölçen bir **Attack Path Management** çözümüdür. **On-premises** ve **cloud** ortamlarındaki farklı **attack path** türlerini sürekli izlemek, önceliklendirmek, çözüm rehberliği almak ve güvenlik duruşlarını ölçmek isteyen kuruluşlar için idealdir.

Bu projenin iyi tarafı, **BloodHound for Enterprise** ekibinin **ticari** ve **FOSS (Free and Open Source Software)** projeleri arasında ortak bir kütüphane kullanması ve **[SharpHound Common](https://github.com/BloodHoundAD/SharpHoundCommon)**'ı tanıtmasıdır. **[FOSS](https://en.wikipedia.org/wiki/Free_and_open-source_software) SharpHound** ve **SharpHound Enterprise**, aynı kod tabanından oluşturulmaktadır. Bu kod tabanı, diğer avantajlarının yanı sıra aşağıdakileri sağlamaktadır:

* Geliştirilmiş [belgeler](https://bloodhoundad.github.io/SharpHoundCommon/index.html).  
* ***SharpHound**'un kalitesini ve stabilitesini herkes için iyileştirir.  

Not: Daha fazla bilgi edinmek için şunu okuyabilirsiniz: **[Introducing BloodHound 4.1 — The Three Headed Hound.](https://posts.specterops.io/introducing-bloodhound-4-1-the-three-headed-hound-be3c4a808146)**


Şimdi, grafik teorisini ve **BloodHound**'un **nodes**  ve **edges**  ile en kısa yolları nasıl bulduğunu incelediğimize göre, biraz veri toplayıp bunları **BloodHound**'a aktaralım ve işlemeye başlayalım.


### RDP aracılığıyla bağlanma

```
xfreerdp /v: /u:username /p:
```

Aşağıdaki bölümlerde, Windows ve Linux sistemlerinden **Active Directory** bilgilerini nasıl toplayacağımızı ve topladığımız bilgileri nasıl analiz edeceğimizi göreceğiz.



## SharpHound - Data Collection from Windows

**[SharpHound](https://github.com/BloodHoundAD/SharpHound)**, **[BloodHound](https://github.com/BloodHoundAD/BloodHound)** için resmi veri toplama aracıdır, C# ile yazılmıştır ve .NET framework'ü yüklü Windows sistemlerinde çalıştırılabilir. Araç, **Active Directory**'den veri toplamak için yerel Windows API işlevleri ve LDAP sorguları gibi çeşitli teknikler kullanır. SharpHound tarafından toplanan veriler, **Active Directory** ortamında güvenlik zayıflıklarını belirlemek, saldırı düzenlemek veya düzeltme planları yapmak için kullanılabilir.

Bu bölümde, SharpHound kullanarak **Active Directory**'yi nasıl envanterleyeceğimizi ve bu işlemin temel işlevlerini keşfedeceğiz.


### Basic **Enumeration**

Varsayılan olarak, SharpHound herhangi bir seçenek olmadan çalıştırıldığında, onu çalıştıran kullanıcının ait olduğu domaini belirleyecek ve varsayılan verileri toplayacaktır. Şimdi SharpHound'u herhangi bir seçenek olmadan çalıştıralım.

#### SharpHound'u herhangi bir seçenek olmadan çalıştırma

```
C:\tools> SharpHound.exe

2023-01-10T09:10:27.5517894-06:00|INFORMATION|This version of SharpHound
is compatible with the 4.2 Release of BloodHound
2023-01-10T09:10:27.6678232-06:00|INFORMATION|Resolved Collection Methods:
Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps,
DCOM, SPNTargets, PSRemote
2023-01-10T09:10:27.6834781-06:00|INFORMATION|Initializing SharpHound at
9:10 AM on 1/10/2023
2023-01-10T09:11:12.0547392-06:00|INFORMATION|Flags: Group, LocalAdmin,
Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets,
PSRemote
2023-01-10T09:11:12.2081156-06:00|INFORMATION|Beginning LDAP search for
INLANEFREIGHT.HTB
2023-01-10T09:11:12.2394159-06:00|INFORMATION|Producer has finished,
closing LDAP channel
2023-01-10T09:11:12.2615280-06:00|INFORMATION|LDAP channel closed, waiting
for consumers
2023-01-10T09:11:42.6237001-06:00|INFORMATION|Status: 0 objects finished
(+0 0)/s -- Using 35 MB RAM
2023-01-10T09:12:12.6416076-06:00|INFORMATION|Status: 0 objects finished
(+0 0)/s -- Using 37 MB RAM
2023-01-10T09:12:42.9758511-06:00|INFORMATION|Status: 0 objects finished
(+0 0)/s -- Using 37 MB RAM
2023-01-10T09:12:43.2077516-06:00|INFORMATION|Consumers finished, closing
output channel
2023-01-10T09:12:43.2545768-06:00|INFORMATION|Output channel closed,
waiting for output task to complete
Closing writers
2023-01-10T09:12:43.3771345-06:00|INFORMATION|Status: 94 objects finished
(+94 1.032967)/s -- Using 42 MB RAM
2023-01-10T09:12:43.3771345-06:00|INFORMATION|Enumeration finished in
00:01:31.1684392
2023-01-10T09:12:43.4617976-06:00|INFORMATION|Saving cache with stats: 53
ID to type mappings.
53 name to SID mappings.
1 machine sid mappings.
2 sid to domain mappings.
0 global catalog mappings.
2023-01-10T09:12:43.4617976-06:00|INFORMATION|SharpHound Enumeration
Completed at 9:12 AM on 1/10/2023! Happy Graphing!

```

Yukarıdaki çıktının 2. satırı, varsayılan olarak kullanılan toplama yöntemini belirtir: **`Resolved Collection Methods`: `Group, LocalAdmin`, `Session, Trusts`, `ACL`, `Container`, `RDP`, `ObjectProps`, `DCOM`, `SPNTargets`, `PSRemote`**. Bu yöntemler, aşağıdaki bilgilerin toplanacağı anlamına gelir:

1. Users and Computers.
2. Active Directory security group membership.
3. Domain trusts.
4. AD objeleri üzerinde kötüye kullanılabilir permissions.
5. OU tree structure.
6. Group Policy links.
7. En fazla ilgili AD objesi özellikleri.
8. Domain-joined Windows sistemlerinden local gruplar ve RDP, DCOM ve PSRemote gibi local ayrıcalıklar.
9. User sessions.
10. All SPN (Service Principal Names).

Lokal gruplar ve oturumlar hakkında bilgi almak için SharpHound, topladığı bilgisayar listesinden domain-joined Windows bilgisayarlarının her birine bağlanmayı deneyecektir. SharpHound'un çalıştığı kullanıcının remote bilgisayarda ayrıcalıkları varsa, aşağıdaki bilgileri toplayacaktır::

1. **Local Administrators**, **Remote Desktop**, **Distributed COM**, ve **Remote Management** gruplarının üyeleri.
2. Kullanıcıların etkileşimli olarak oturum açtığı sistemlerle ilişkilendirilebilecek **Active sessions**.


Not: **Domain-joined** makinelerden bilgi toplamak, örneğin local grup üyelikleri ve aktif oturumlar, yalnızca SharpHound'un çalıştırıldığı kullanıcı oturumunun hedef bilgisayar üzerinde **Administrator** yetkilerine sahip olması durumunda mümkündür.

SharpHound sonlandığında, varsayılan olarak, ismi mevcut tarih ile başlayıp **BloodHound** ile biten bir zip dosyası oluşturacaktır. Bu zip arşivi, bir grup **JSON** dosyası içerir.

![Pasted image 20250219013310.png](/img/user/Pasted%20image%2020250219013310.png)


#### Importing Data into BloodHound

1. Start the neo4j database service:


#### Start Service

```
PS C:\htb> net start neo4j
The Neo4j Graph Database - neo4j service is starting..
The Neo4j Graph Database - neo4j service was started successfully.
```

1. C:\Tools\BloodHound\BloodHound.exe dosyasını başlatın ve aşağıdaki kimlik bilgileriyle oturum açın:

```
Username: neo4j
Password: Password123
```

![Pasted image 20250219013440.png](/img/user/Pasted%20image%2020250219013440.png)

En sağdaki **upload** butonuna tıklayın, zip dosyasını bulun ve yükleyin. Yükleme yüzdesinin tamamlanma durumunu gösteren bir durum bilgisi göreceksiniz.

![Pasted image 20250219013503.png](/img/user/Pasted%20image%2020250219013503.png)

Not: İstediğimiz kadar zip dosyası yükleyebiliriz. **BloodHound**, verileri çoğaltmaz, ancak veritabanında bulunmayan verileri ekler.

1. Yükleme tamamlandığında, verileri analiz edebiliriz. **Domain** hakkında bilgi görmek istiyorsak, arama kutusuna **Domain:INLANEFREIGHT.HTB** yazabiliriz. Bu, domain ismiyle bir ikon gösterecektir. İkona tıkladığınızda, **node** (domain) hakkında, kaç kullanıcı, grup, bilgisayar, **OU** vb. olduğuna dair bilgi görüntülenir.


![Pasted image 20250219063444.png](/img/user/Pasted%20image%2020250219063444.png)

Şimdi, **BloodHound** içinde bilgileri analiz etmeye başlayabilir ve hedeflerimize giden yolları bulabiliriz.

Not: Dosyaları içe aktarırken bilgisayar adları görünmezse, hatayı düzeltmek için dosyayı tekrar içe aktarabiliriz.

Aşağıdaki bölümlerde SharpHound'un Active Directory verilerini toplamak için sunduğu diğer seçenekler ele alınacaktır.


# SharpHound - Data Collection from Windows (Part 2)

**SharpHound**, hangi bilgileri toplamak istediğimizi belirlememize ve nasıl toplayacağımıza yardımcı olan birçok kullanışlı seçeneğe sahiptir. Bu bölümde, **SharpHound** içinde kullanabileceğimiz en yaygın seçeneklerden bazılarını inceleyeceğiz ve tüm **SharpHound** seçenekleri için resmi dokümantasyona bağlantılar vereceğiz.

## SharpHound Options

Tüm **SharpHound** seçeneklerini listelemek için `--help` kullanabiliriz. Aşağıdaki liste, **1.1.0** sürümüne aittir:


#### SharpHound Options

```
C:\Tools> SharpHound.exe --help
2023-01-10T13:08:39.2519248-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
SharpHound 1.1.0
Copyright (C) 2023 SpecterOps

  -c, --collectionmethods      (Default: Default) Collection Methods: Group, LocalGroup, LocalAdmin, RDP, DCOM, PSRemote, Session, Trusts, ACL, Container,
                               ComputerOnly, GPOLocalGroup, LoggedOn, ObjectProps, SPNTargets, Default, DCOnly, All

  -d, --domain                 Specify domain to enumerate

  -s, --searchforest           (Default: false) Search all available domains in the forest

  --stealth                    Stealth Collection (Prefer DCOnly whenever possible!)

  -f                           Add an LDAP filter to the pregenerated filter.

  --distinguishedname          Base DistinguishedName to start the LDAP search at

  --computerfile               Path to file containing computer names to enumerate

  --outputdirectory            (Default: .) Directory to output file too

  --outputprefix               String to prepend to output file names

  --cachename                  Filename for cache (Defaults to a machine specific identifier)

  --memcache                   Keep cache in memory and don't write to disk

  --rebuildcache               (Default: false) Rebuild cache and remove all entries

  --randomfilenames            (Default: false) Use random filenames for output

  --zipfilename                Filename for the zip

  --nozip                      (Default: false) Don't zip files

  --zippassword                Password protects the zip with the specified password

  --trackcomputercalls         (Default: false) Adds a CSV tracking requests to computers

  --prettyprint                (Default: false) Pretty print JSON

  --ldapusername               Username for LDAP

  --ldappassword               Password for LDAP

  --domaincontroller           Override domain controller to pull LDAP from. This option can result in data loss

  --ldapport                   (Default: 0) Override port for LDAP

  --secureldap                 (Default: false) Connect to LDAP SSL instead of regular LDAP

  --disablecertverification    (Default: false) Disables certificate verification when using LDAPS

  --disablesigning             (Default: false) Disables Kerberos Signing/Sealing

  --skipportcheck              (Default: false) Skip checking if 445 is open

  --portchecktimeout           (Default: 500) Timeout for port checks in milliseconds

  --skippasswordcheck          (Default: false) Skip check for PwdLastSet when enumerating computers

  --excludedcs                 (Default: false) Exclude domain controllers from session/localgroup enumeration (mostly for ATA/ATP)

  --throttle                   Add a delay after computer requests in milliseconds

  --jitter                     Add jitter to throttle (percent)

  --threads                    (Default: 50) Number of threads to run enumeration with

  --skipregistryloggedon       Skip registry session enumeration

  --overrideusername           Override the username to filter for NetSessionEnum

  --realdnsname                Override DNS suffix for API calls

  --collectallproperties       Collect all LDAP properties from objects

  -l, --Loop                   Loop computer collection

  --loopduration               Loop duration (Defaults to 2 hours)

  --loopinterval               Delay between loops

  --statusinterval             (Default: 30000) Interval in which to display status in milliseconds

  -v                           (Default: 2) Enable verbose output

  --help                       Display this help screen.

  --version                    Display version information.
```


## [Collection Methods](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound-all-flags.html#collectionmethod)

`-collectionmethod` veya `-c` seçeneği, hangi tür verileri toplamak istediğimizi belirlememizi sağlar. Yukarıdaki **help** menüsünde, kullanılabilecek **collection** yöntemlerinin listesini görebiliriz. Daha önce ele almadığımız bazı yöntemleri açıklayalım:

- **`All`**: **GPOLocalGroup** hariç tüm **collection** yöntemlerini uygular.
- **`DCOnly`**: Yalnızca **Domain Controller** üzerinden veri toplar ve **domain-joined** Windows cihazlardan veri almaya çalışmaz. **Users**, **computers**, **security groups** üyelikleri, **domain trusts**, **Active Directory objects** üzerindeki kötüye kullanılabilir izinler, **OU structure**, **Group Policy** ve en önemli **Active Directory object** özelliklerini toplar. **Group Policy** ile zorunlu kılınan **local groups** ile etkilenen bilgisayarları eşleştirmeye çalışır.
- **`ComputerOnly`**: **DCOnly** seçeneğinin tersidir. Yalnızca **domain-joined** bilgisayarlardan veri toplar; **user sessions** ve **local groups** gibi bilgileri içerir.

Bulunduğumuz senaryoya bağlı olarak, ihtiyaçlarımıza en uygun **collection** yöntemini seçmeliyiz. Şu kullanım senaryosunu inceleyelim:

2000 bilgisayardan oluşan bir ortamdayız ve bu sistemlerde **SOC** tarafından izlenen bazı **network monitoring** araçları bulunuyor. **Default** **collection** yöntemini kullanıyoruz, ancak **SharpHound**'u çalıştırdığımız bilgisayarı unutuyoruz. Bu bilgisayar, **domain** içindeki tüm bilgisayarlara bağlanmaya çalışıyor.

**Attack host**'umuz tüm **workstation**'lara trafik göndermeye başladığında, **SOC** sistemimizde şüpheli hareketlilik tespit edip makinemizi karantinaya alıyor.

Bu senaryoda, **All** veya **Default** yerine **DCOnly** kullanmalıyız, çünkü bu yöntem yalnızca **Domain Controller** ile trafik oluşturur. Daha sonra en ilginç hedef makineleri belirleyip bir listeye ekleyebiliriz (örn: `computers.txt`). Ardından, **SharpHound**'u **ComputerOnly** **collection** yöntemiyle ve `--computerfile` seçeneği ile çalıştırarak sadece `computers.txt` dosyasındaki bilgisayarları **enumerate** etmeye çalışabiliriz.

Yöntemleri ve bunların etkilerini bilmek oldukça önemlidir. **[SadProcessor](https://twitter.com/SadProcessor)** tarafından oluşturulan aşağıdaki tablo, her yöntemin kullandığı **communication protocols**, her teknik hakkında bilgiler ve diğer bazı detaylar için genel bir referans sunmaktadır:

![Pasted image 20250219064142.png](/img/user/Pasted%20image%2020250219064142.png)

Not: Bu tablo, **SharpHound**'un eski bir sürümü için oluşturulmuştur. Bazı seçenekler artık mevcut değil veya değiştirilmiş olabilir, ancak yine de **collection** yöntemleri ve bunların etkileri hakkında genel bir bakış sunmaktadır. Daha fazla bilgi için **BloodHound** [dokümantasyon](https://bloodhound.readthedocs.io/en/latest/data-collection/sharphound-all-flags.html) sayfasını ziyaret edebilirsiniz.


## Common used flags

Eğer **SharpHound**'u çalıştırdığımız **context** dışında bir kullanıcının kimlik bilgilerini ele geçirirsek, `--ldapusername` ve `--ldappassword` seçeneklerini kullanarak **SharpHound**'u bu kimlik bilgileriyle çalıştırabiliriz.

Kullanışlı bir diğer seçenek ise `-d` veya `--domain` bayrağıdır. Bu seçenek varsayılan olarak atanmış olsa da, birden fazla **domain** bulunan bir ortamda çalışıyorsak, **SharpHound**'un belirttiğimiz **domain** üzerinden veri topladığından emin olmak için bu seçeneği kullanabiliriz.

**SharpHound**, **Domain Controller**'ı otomatik olarak algılar. Ancak belirli bir **DC**'yi hedeflemek istiyorsak, `--domaincontroller` seçeneğini kullanarak hedef **Domain Controller**'ın **IP** veya **FQDN** değerini belirtebiliriz. Bu seçenek, genellikle unutulmuş veya ikincil bir **domain**'i hedeflemek için faydalı olabilir. İkincil **Domain Controller**, genellikle birincil **DC** kadar sıkı güvenlik önlemlerine veya izleme araçlarına sahip olmayabilir.

Bu bayrağın bir başka kullanım senaryosu ise **port forward** işlemi yaparken belirli bir **IP** ve **port** hedeflemektir. Bunun için `--ldapport` bayrağını kullanarak bir **port** belirtebiliriz.


## Randomize and hide SharpHound Output

**SharpHound**'un varsayılan olarak farklı **`.json`** dosyaları oluşturduğu, ardından bunları bir **zip** dosyasında sakladığı bilinmektedir. Ayrıca, gerçekleştirdiği sorguların önbelleğine karşılık gelen, rastgele adlandırılmış bir **`.bin`** uzantılı dosya da üretir. **Defense** ekipleri, bu kalıpları kullanarak **BloodHound**'u tespit edebilir.

Bu izleri gizlemeye çalışmanın bir yolu, aşağıdaki seçeneklerden bazılarını birleştirmektir:

| **Seçenek**         | **Açıklama**                                                                                |
| ------------------- | ------------------------------------------------------------------------------------------- |
| `--memcache`        | Önbelleği bellekte tutar ve diske yazmaz.                                                   |
| `--randomfilenames` | Çıktı dosyaları için, **zip** dosyası da dahil olmak üzere rastgele dosya adları oluşturur. |
| `--outputprefix`    | Çıktı dosya adlarının başına eklenmesini istediğimiz bir **string** belirtir.               |
| `--outputdirectory` | Çıktı dosyalarının kaydedileceği dizini belirler.                                           |
| `--zipfilename`     | **Zip** dosyası için özel bir dosya adı belirler.                                           |
| `--zippassword`     | **Zip** dosyasını belirtilen **password** ile korur.                                        |


#### Paylaşılan klasörü kullanıcı adı ve şifre ile başlat

```
sudo impacket-smbserver share ./ -smb2support -user test -password test
Impacket v0.10.1.dev1+20220720.103933.3c6713e3 - Copyright 2022 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```

Şimdi paylaşılan klasöre bağlanalım ve **SharpHound** çıktısını oraya kaydedelim:


#### Connect to the shared folder with username and password

```
C:\htb> net use \\10.10.14.33\share /user:test test
The command completed successfully.
```


#### SharpHound'u çalıştırma ve çıktıyı paylaşılan bir klasöre kaydetme

```
C:\htb> C:\Tools\SharpHound.exe --memcache --outputdirectory \\10.10.14.33\share\ --zippassword HackTheBox --outputprefix HTB --randomfilenames

2023-01-11T11:31:43.4459137-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-11T11:31:43.5998704-06:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-11T11:31:43.6311043-06:00|INFORMATION|Initializing SharpHound at 11:31 AM on 1/11/2023
2023-01-11T11:31:55.0551988-06:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-11T11:31:55.2710788-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T11:31:55.3089182-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T11:31:55.3089182-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T11:32:25.7331485-06:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 35 MB RAM
2023-01-11T11:32:41.7321172-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T11:32:41.7633662-06:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-11T11:32:52.2310202-06:00|INFORMATION|Status: 94 objects finished (+94 1.678571)/s -- Using 42 MB RAM
2023-01-11T11:32:52.2466171-06:00|INFORMATION|Enumeration finished in 00:00:56.9776773
2023-01-11T11:33:09.4855302-06:00|INFORMATION|SharpHound Enumeration Completed at 11:33 AM on 1/11/2023! Happy Graphing!
```


#### Unzipping the file

```
unzip ./HTB_20230111113143_5yssigbd.w3f
Archive:  ./HTB_20230111113143_5yssigbd.w3f
[./HTB_20230111113143_5yssigbd.w3f] HTB_20230111113143_hjclkslu.2in password:
  inflating: HTB_20230111113143_hjclkslu.2in
  inflating: HTB_20230111113143_hk3lxtz3.1ku
  inflating: HTB_20230111113143_kghttiwp.jbq
  inflating: HTB_20230111113143_kdg5svst.4sc
  inflating: HTB_20230111113143_qeugxqep.lya
  inflating: HTB_20230111113143_xsxzlxht.awa
  inflating: HTB_20230111113143_51zkhw0e.bth
```

Şimdi verilerimizi **BloodHound**'a yükleyebiliriz:

![Pasted image 20250219064934.png](/img/user/Pasted%20image%2020250219064934.png)

Not: Eğer **zip** dosyasına bir şifre koyarsak, önce şifreyi çözmemiz gerekecek, ancak şifre koymadıysak, dosyayı rastgele adı ve uzantısıyla olduğu gibi içe aktarabiliriz ve yine de dosya içe aktarılacaktır.


## Session Loop Collection Method

Bir kullanıcı, remote bir bilgisayara bağlantı kurduğunda, bir oturum oluşturur. Oturum bilgileri, kullanıcı adını ve bağlantının hangi bilgisayar veya **IP** üzerinden yapıldığını içerir. Oturum aktifken bağlantı bilgisayarda kalır, ancak kullanıcı bağlantıyı kestikten sonra oturum boşta kalır ve birkaç dakika içinde kaybolur. Bu, oturumları ve kullanıcıların aktif olduğu yerleri belirlemek için küçük bir zaman dilimimiz olduğu anlamına gelir.

Not: **Active Directory** ortamlarında, kullanıcıların hangi bilgisayarlara bağlandığını anlamak önemlidir, çünkü bu, hedeflerimize ulaşmak için hangi bilgisayarları ele geçireceğimizi anlamamıza yardımcı olur.

Hedef makinede bir komut clientini açalım ve **net session** komutunu yazalım, böylece herhangi bir aktif oturum olup olmadığını belirleyebiliriz:


#### Looking for Active Sessions

```
C:\htb> net session
There are no entries in the list.
```

Aktif oturum yok, bu da şu an **SharpHound**'u çalıştırırsak, bilgisayarımızda herhangi bir oturum bulamayacağımız anlamına gelir. **SharpHound**'u varsayılan **collection method** ile çalıştırdığımızda, **Session collection method**'u da içerir. Bu yöntem, hedef bilgisayarlardan bir session collection işlemi yapar. Eğer bu collection sırasında bir session bulursa, onu toplar; ancak oturum süresi dolarsa, böyle bir bilgiye sahip olamayız. Bu nedenle, **SharpHound**'da `--loop` seçeneği bulunur. **SharpHound**'da döngü ile kullanabileceğimiz birkaç seçenek vardır:

|**Seçenek**|**Açıklama**|
|---|---|
|`--Loop`|Bilgisayar toplama işlemini döngüye sokar.|
|`--loopduration`|Döngü işleminin süresi (Varsayılan: 02:00:00).|
|`--loopinterval`|Döngüler arasında uyuma süresi (Varsayılan: 00:00:30).|
|`--stealth`|"Stealth" veri toplama işlemi yapar. Yalnızca kullanıcı oturumu verisi olma olasılığı en yüksek olan sistemlere dokunulur.|

Eğer bir sonraki saat boyunca oturumları aramak ve her bilgisayarı her dakika sorgulamak istiyorsak, **SharpHound**'u şu şekilde kullanabiliriz:


#### Session Loop

```
C:\Tools> SharpHound.exe -c Session --loop --loopduration 01:00:00 --loopinterval 00:01:00

2023-01-11T14:15:48.9375275-06:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-11T14:15:49.1382880-06:00|INFORMATION|Resolved Collection Methods: Session
2023-01-11T14:15:49.1695244-06:00|INFORMATION|Initializing SharpHound at 2:15 PM on 1/11/2023
2023-01-11T14:16:00.4571231-06:00|INFORMATION|Flags: Session
2023-01-11T14:16:00.6108583-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T14:16:00.6421492-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T14:16:00.6421492-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T14:16:00.7268495-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T14:16:00.7424755-06:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-11T14:16:00.9587384-06:00|INFORMATION|Status: 4 objects finished (+4 Infinity)/s -- Using 35 MB RAM
2023-01-11T14:16:00.9587384-06:00|INFORMATION|Enumeration finished in 00:00:00.3535475
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Creating loop manager with methods Session
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Starting looping
2023-01-11T14:16:01.0434611-06:00|INFORMATION|Waiting 30 seconds before starting loop
2023-01-11T14:16:31.0598479-06:00|INFORMATION|Looping scheduled to stop at 01/11/2023 15:16:31
2023-01-11T14:16:31.0598479-06:00|INFORMATION|01/11/2023 14:16:31 - 01/11/2023 15:16:31
2023-01-11T14:16:31.0598479-06:00|INFORMATION|Starting loop 1 at 2:16 PM on 1/11/2023
2023-01-11T14:16:31.0754340-06:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-11T14:16:31.0754340-06:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-11T14:16:31.0754340-06:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-11T14:16:42.1893741-06:00|INFORMATION|Consumers finished, closing output channel
2023-01-11T14:16:42.1893741-06:00|INFORMATION|Output channel closed, waiting for output task to complete
...SNIP...
```

**SpecterOps** ekibinin **[How BloodHound’s session collection works](https://www.youtube.com/watch?v=q86VgM2Tafc)** başlıklı videosunu izleyerek bu toplama yöntemi hakkında daha derin bir açıklama alabilirsiniz. İşte **Sven Defatsch** tarafından yazılmış, **Compass Security**'den oturum numaralandırma ile ilgili başka bir mükemmel [blog](https://blog.compass-security.com/2022/05/bloodhound-inner-workings-part-2/) yazısı.

Not: **BloodHound** videosu, Microsoft'un session verisi toplamak için yönetici olma gereksinimini getirmesinden önce kaydedilmiştir.


## Running from Non-Domain-Joined Systems

Bazen **SharpHound**'u bir domain üyesi olmayan bir bilgisayardan çalıştırmamız gerekebilir, örneğin **HackTheBox** saldırısı veya yalnızca ağ erişimiyle yapılan internal
penetrasyon testi sırasında.

Bu senaryolarda, belirli kullanıcı kimlik bilgileriyle uygulamayı çalıştırmak için `runas /netonly /user:<DOMAIN>\<username> <app>` komutunu kullanabiliriz. `/netonly` bayrağı, sağlanan kimlik bilgileriyle ağ erişimi sağlar.

Bunu yapmak için domain üyesi olmayan bir bilgisayar kullanalım ve şu adımları takip edelim:

Hedef **IP**'ye ve **13389** portuna aşağıdaki kimlik bilgileriyle **RDP** üzerinden bağlanın: **haris:hacking123**.


#### RDP aracılığıyla şu adrese bağlanın

```
xfreerdp /v:10.129.204.207:13389 /u:haris /p:Hackthebox /dynamic-resolution /drive:.,linux
[12:13:14:635] [173624:173625] [INFO][com.freerdp.core] - freerdp_connect:freerdp_set_last_error_ex resetting error state
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpdr
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx rdpsnd
[12:13:14:635] [173624:173625] [INFO][com.freerdp.client.common.cmdline] - loading channelEx cliprd
...SNIP...
```

**SharpHound**'u kullanmadan önce, hedef domainin **DNS** adlarını çözümleyebilmemiz gerekir ve eğer domainin **DNS** sunucusuna ağ erişimimiz varsa, ağ kartımızın **DNS** ayarlarını bu sunucuya yapılandırabiliriz. Eğer bu mümkün değilse, [**hosts** dosyamızı ](https://en.wikiversity.org/wiki/Hosts_file/Edit)düzenleyerek domain denetleyicisinin **DNS** adlarını ekleyebiliriz.

Not: **Hosts** dosyasındaki **DNS** adlarını kullanırken, dosyada bulunmayan veya yanlış yapılandırılmış adlar nedeniyle bazı hatalar meydana gelebilir.

![Pasted image 20250219070043.png](/img/user/Pasted%20image%2020250219070043.png)

**cmd.exe**'yi çalıştırın ve ardından aşağıdaki komutu girin, bu komut başka bir **cmd.exe** penceresi başlatacak ve **htb-student** kimlik bilgileriyle giriş yapmanızı isteyecektir. Şifre **HTBRocks!**'tır:

```
C:\htb> runas /netonly /user:INLANEFREIGHT\htb-student cmd.exe
Enter the password for INLANEFREIGHT\htb-student:
Attempting to start cmd.exe as user "INLANEFREIGHT\htb-student" ...
```

![Pasted image 20250219070113.png](/img/user/Pasted%20image%2020250219070113.png)

Not: **runas /netonly**, kimlik bilgilerini doğrulamaz ve yanlış kimlik bilgilerini kullanırsak, ağ üzerinden bağlantı kurmaya çalışırken bunu fark ederiz.

Başarılı bir şekilde kimlik doğrulaması yaptığımızı doğrulamak için şu komutu çalıştırın:

```
C:\htb> net view \\inlanefreight.htb\
Shared resources at \\inlanefreight.htb\

Share name  Type  Used as  Comment.

-------------------------------------------------------------------------------
NETLOGON    Disk           Logon server share
SYSVOL      Disk           Logon server share
The command completed successfully.
```

**SharpHound.exe**'u şu seçenekle çalıştırın:

```
C:\Tools> SharpHound.exe -d inlanefreight.htb
2023-01-12T09:25:21.5040729-08:00|INFORMATION|This version of SharpHound is compatible with the 4.2 Release of BloodHound
2023-01-12T09:25:21.6603414-08:00|INFORMATION|Resolved Collection Methods: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-12T09:25:21.6760332-08:00|INFORMATION|Initializing SharpHound at 9:25 AM on 1/12/2023
2023-01-12T09:25:22.0197242-08:00|INFORMATION|Flags: Group, LocalAdmin, Session, Trusts, ACL, Container, RDP, ObjectProps, DCOM, SPNTargets, PSRemote
2023-01-12T09:25:22.2541585-08:00|INFORMATION|Beginning LDAP search for INLANEFREIGHT.HTB
2023-01-12T09:25:22.3010985-08:00|INFORMATION|Producer has finished, closing LDAP channel
2023-01-12T09:25:22.3010985-08:00|INFORMATION|LDAP channel closed, waiting for consumers
2023-01-12T09:25:52.3794310-08:00|INFORMATION|Status: 0 objects finished (+0 0)/s -- Using 36 MB RAM
2023-01-12T09:26:21.3792883-08:00|INFORMATION|Consumers finished, closing output channel
2023-01-12T09:26:21.4261266-08:00|INFORMATION|Output channel closed, waiting for output task to complete
Closing writers
2023-01-12T09:26:21.4885564-08:00|INFORMATION|Status: 94 objects finished (+94 1.59322)/s -- Using 44 MB RAM
2023-01-12T09:26:21.4885564-08:00|INFORMATION|Enumeration finished in 00:00:59.2357019
2023-01-12T09:26:21.5665717-08:00|INFORMATION|Saving cache with stats: 53 ID to type mappings.
 53 name to SID mappings.
 1 machine sid mappings.
 2 sid to domain mappings.
 0 global catalog mappings.
2023-01-12T09:26:21.5822432-08:00|INFORMATION|SharpHound Enumeration Completed at 9:26 AM on 1/12/2023! Happy Graphing!
```


SharpHound'un Windows üzerindeki bazı kullanım senaryolarını ve saldırdığımız domain'den nasıl bilgi toplayabileceğimizi keşfedeceğiz.

Sonraki bölümde, Linux'tan nasıl bilgi toplayabileceğimizi inceleyeceğiz.


# BloodHound.py - Data Collection from Linux

[SharpHound](https://github.com/BloodHoundAD/SharpHound)'un Linux için resmi bir veri toplayıcı aracı yoktur. Ancak, [Dirk-jan Mollema](https://twitter.com/_dirkjan), BloodHound için Impacket tabanlı bir Python toplayıcı olan **[BloodHound.py](https://github.com/fox-it/BloodHound.py)**'yı yaratarak, Linux'tan BloodHound için Active Directory bilgilerini toplamamıza olanak sağlamıştır.


## Installation

**[BloodHound.py](https://github.com/fox-it/BloodHound.py)**'yı şu komutla **pip** kullanarak ya da deposunu klonlayıp **python setup.py install** komutunu çalıştırarak kurabiliriz. Çalıştırmak için **impacket**, **ldap3** ve **dnspython** gereklidir. Aracı **pip** ile şu komutla kurabiliriz:


#### Install BloodHound.py

```
pip install bloodhound
Defaulting to user installation because normal site-packages is not writeable
Collecting bloodhound
  Downloading bloodhound-1.6.0-py3-none-any.whl (80 kB)
     ━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━ 81.0/81.0 kB 1.8 MB/s eta 0:00:00
Requirement already satisfied: ldap3!=2.5.0,!=2.5.2,!=2.6,>=2.5 in /home/plaintext/.local/lib/python3.9/site-packages (from bloodhound) (2.9.1)
Requirement already satisfied: pyasn1>=0.4 in /usr/local/lib/python3.9/dist-packages (from bloodhound) (0.4.6)
Requirement already satisfied: impacket>=0.9.17 in /home/plaintext/.local/lib/python3.9/site-packages (from bloodhound) (0.10.1.dev1+20220720.103933.3c6713e3)
Requirement already satisfied: future in /usr/lib/python3/dist-packages (from bloodhound) (0.18.2)
Requirement already satisfied: dnspython in /usr/local/lib/python3.9/dist-packages (from bloodhound) (2.2.1)
Requirement already satisfied: charset-normalizer in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (2.0.12)
Requirement already satisfied: six in /usr/local/lib/python3.9/dist-packages (from impacket>=0.9.17->bloodhound) (1.12.0)
Requirement already satisfied: pycryptodomex in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (3.9.7)
Requirement already satisfied: ldapdomaindump>=0.9.0 in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (0.9.3)
Requirement already satisfied: dsinternals in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (1.2.4)
Requirement already satisfied: flask>=1.0 in /usr/lib/python3/dist-packages (from impacket>=0.9.17->bloodhound) (1.1.2)
Requirement already satisfied: pyOpenSSL>=21.0.0 in /home/plaintext/.local/lib/python3.9/site-packages (from impacket>=0.9.17->bloodhound) (22.1.0)
Requirement already satisfied: cryptography<39,>=38.0.0 in /home/plaintext/.local/lib/python3.9/site-packages (from pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (38.0.3)
Requirement already satisfied: cffi>=1.12 in /usr/local/lib/python3.9/dist-packages (from cryptography<39,>=38.0.0->pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (1.12.3)
Requirement already satisfied: pycparser in /usr/local/lib/python3.9/dist-packages (from cffi>=1.12->cryptography<39,>=38.0.0->pyOpenSSL>=21.0.0->impacket>=0.9.17->bloodhound) (2.19)
Installing collected packages: bloodhound
Successfully installed bloodhound-1.6.0
```

Kaynaklardan kurmak için, [**BloodHound.py**'nın GitHub deposunu](https://github.com/fox-it/BloodHound.py) klonlayabiliriz ve ardından şu komutu çalıştırabiliriz:


#### Install BloodHound.py from the source

```
git clone https://github.com/fox-it/BloodHound.py -q
cd BloodHound.py/
sudo python3 setup.py install
running install
running bdist_egg
running egg_info
creating bloodhound.egg-info
writing bloodhound.egg-info/PKG-INFO
writing dependency_links to bloodhound.egg-info/dependency_links.txt
writing entry points to bloodhound.egg-info/entry_points.txt
writing requirements to bloodhound.egg-info/requires.txt
writing top-level names to bloodhound.egg-info/top_level.txt
writing manifest file 'bloodhound.egg-info/SOURCES.txt'
reading manifest file 'bloodhound.egg-info/SOURCES.txt'
writing manifest file 'bloodhound.egg-info/SOURCES.txt'
installing library code to build/bdist.linux-x86_64/egg
running install_lib
running build_py
creating build
creating build/lib
creating build/lib/bloodhound
copying bloodhound/__init__.py -> build/lib/bloodhound
copying bloodhound/__main__.py -> build/lib/bloodhound
...SNIP...
```


## BloodHound.py Options

Tüm **BloodHound.py** seçeneklerini listelemek için **--help** komutunu kullanabiliriz. Aşağıdaki liste, **1.6.0** sürümüne karşılık gelmektedir:


#### BloodHound.py options

```
python3 bloodhound.py
usage: bloodhound.py [-h] [-c COLLECTIONMETHOD] [-d DOMAIN] [-v] [-u USERNAME] [-p PASSWORD] [-k] [--hashes HASHES] [-no-pass] [-aesKey hex key]
                     [--auth-method {auto,ntlm,kerberos}] [-ns NAMESERVER] [--dns-tcp] [--dns-timeout DNS_TIMEOUT] [-dc HOST] [-gc HOST]
                     [-w WORKERS] [--exclude-dcs] [--disable-pooling] [--disable-autogc] [--zip] [--computerfile COMPUTERFILE]
                     [--cachefile CACHEFILE]

Python based ingestor for BloodHound
For help or reporting issues, visit https://github.com/Fox-IT/BloodHound.py

optional arguments:
  -h, --help            show this help message and exit
  -c COLLECTIONMETHOD, --collectionmethod COLLECTIONMETHOD
                        Which information to collect. Supported: Group, LocalAdmin, Session, Trusts, Default (all previous), DCOnly (no computer
                        connections), DCOM, RDP,PSRemote, LoggedOn, Container, ObjectProps, ACL, All (all except LoggedOn). You can specify more
                        than one by separating them with a comma. (default: Default)
  -d DOMAIN, --domain DOMAIN
                        Domain to query.
  -v                    Enable verbose output

authentication options:
  Specify one or more authentication options.
  By default Kerberos authentication is used and NTLM is used as fallback.
  Kerberos tickets are automatically requested if a password or hashes are specified.

  -u USERNAME, --username USERNAME
                        Username. Format: username[@domain]; If the domain is unspecified, the current domain is used.
  -p PASSWORD, --password PASSWORD
                        Password
  -k, --kerberos        Use kerberos
  --hashes HASHES       LM:NLTM hashes
  -no-pass              don't ask for password (useful for -k)
  -aesKey hex key       AES key to use for Kerberos Authentication (128 or 256 bits)
  --auth-method {auto,ntlm,kerberos}
                        Authentication methods. Force Kerberos or NTLM only or use auto for Kerberos with NTLM fallback

collection options:
  -ns NAMESERVER, --nameserver NAMESERVER
                        Alternative name server to use for queries
  --dns-tcp             Use TCP instead of UDP for DNS queries
  --dns-timeout DNS_TIMEOUT
                        DNS query timeout in seconds (default: 3)
  -dc HOST, --domain-controller HOST
                        Override which DC to query (hostname)
  -gc HOST, --global-catalog HOST
                        Override which GC to query (hostname)
  -w WORKERS, --workers WORKERS
                        Number of workers for computer enumeration (default: 10)
  --exclude-dcs         Skip DCs during computer enumeration
  --disable-pooling     Don't use subprocesses for ACL parsing (only for debugging purposes)
  --disable-autogc      Don't automatically select a Global Catalog (use only if it gives errors)
  --zip                 Compress the JSON output files into a zip archive
  --computerfile COMPUTERFILE
                        File containing computer FQDNs to use as allowlist for any computer based methods
  --cachefile CACHEFILE
                        Cache file (experimental)
```


**BloodHound.py** impacket kullandığı için, **hashes**, **ccachefiles**, **aeskeys**, **kerberos** gibi seçenekler de **BloodHound.py** için mevcuttur.


## Using BloodHound.py

BloodHound.py'yi Linux'ta kullanabilmek için **--domain** ve **--collectionmethod** seçeneklerine ve kimlik doğrulama yöntemine ihtiyacımız olacak. Kimlik doğrulama, bir kullanıcı adı ve parola, NTLM hash’i, AES anahtarı veya ccache dosyası olabilir. **BloodHound.py**, varsayılan olarak Kerberos kimlik doğrulama yöntemini kullanmaya çalışacak ve eğer başarısız olursa, NTLM'ye geçecektir.

Diğer önemli bir konu ise **domain** adı çözümlemesidir. Eğer DNS sunucumuz hedef **domain**'in DNS sunucusu değilse, **--nameserver** seçeneğini kullanarak sorgular için alternatif bir ad sunucusu belirleyebiliriz.


#### Running BloodHound.py

```
bloodhound-python -d inlanefreight.htb -c DCOnly -u htb-student -p HTBRocks! -ns 10.129.204.207 -k
INFO: Found AD domain: inlanefreight.htb
INFO: Getting TGT for user
WARNING: Failed to get Kerberos TGT. Falling back to NTLM authentication. Error: [Errno Connection error (inlanefreight.htb:88)] [Errno -2] Name or service not known
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 6 users
INFO: Found 52 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 3 computers
INFO: Found 0 trusts
INFO: Done in 00M 11S
```

Not: Kerberos kimlik doğrulaması, host'un **domain** FQDN'sini çözümlemesini gerektirir. Bu, **--nameserver** seçeneğinin Kerberos kimlik doğrulaması için yeterli olmadığı anlamına gelir, çünkü Kerberos'un çalışabilmesi için host'umuzun KDC DNS adını çözümlemesi gerekir. Eğer Kerberos kimlik doğrulaması kullanmak istiyorsak, DNS Sunucusu'nu hedef makineye ayarlamamız veya DNS kaydını hosts dosyamıza eklememiz gerekir.

Şimdi hosts dosyamıza DNS kaydını ekleyelim:

#### Setting up the /etc/hosts file

```
echo -e "\n10.129.204.207 dc01.inlanefreight.htb dc01 inlanefreight inlanefreight.htb" | sudo tee -a /etc/hosts

10.129.204.207 dc01.inlanefreight.htb dc01 inlanefreight inlanefreight.htb
```

Kerberos kimlik doğrulaması ile BloodHound.py kullanın:

#### Using BloodHound.py with Kerberos authentication

```
bloodhound-python -d inlanefreight.htb -c DCOnly -u htb-student -p HTBRocks! -ns 10.129.204.207 --kerberos
INFO: Found AD domain: inlanefreight.htb
INFO: Getting TGT for user
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 1 domains
INFO: Found 1 domains in the forest
INFO: Connecting to LDAP server: dc01.inlanefreight.htb
INFO: Found 6 users
INFO: Found 52 groups
INFO: Found 2 gpos
INFO: Found 1 ous
INFO: Found 19 containers
INFO: Found 3 computers
INFO: Found 0 trusts
INFO: Done in 00M 11S
```

Not: Kerberos Kimlik Doğrulaması, Windows için varsayılan kimlik doğrulama yöntemidir. Bu yöntemi NTLM yerine kullanmak, trafiğimizin daha normal görünmesini sağlar.


## BloodHound.py Output files

Toplama işlemi tamamlandığında, JSON dosyalarını oluşturacaktır, ancak varsayılan olarak içeriği SharpHound gibi zip dosyasına koymaz. İçeriğin bir zip dosyasına yerleştirilmesini istiyorsak, --zip seçeneğini kullanmamız gerekir.

#### List All Collections of BloodHound.py

```
ls
20230112171634_computers.json   20230112171634_domains.json  20230112171634_groups.json  20230112171634_users.json
20230112171634_containers.json  20230112171634_gpos.json     20230112171634_ous.json
```

## Using the Data

Şimdi, INLANEFREIGHT.HTB domaini için verileri toplamanın ve BloodHound GUI'sine aktarmanın çeşitli yollarını gördük, sonuçları analiz edelim ve bazı yaygın sorunlara göz atalım.

# Navigating the BloodHound GUI

BloodHound GUI, SharpHound kullanarak topladığımız verileri analiz etmek için birincil yerdir. Hızlı bir şekilde ihtiyacımız olan verilere erişebilmek için kullanıcı dostu bir arayüze sahiptir.

BloodHound grafiksel arayüzünü keşfedecek ve nasıl faydalanabileceğimizi göreceğiz.

Not: Daha fazla bilgi için [BloodHound Resmi Belgeleri](https://bloodhound.readthedocs.io/en/latest/data-analysis/bloodhound-gui.html#the-bloodhound-gui) sayfasına bakın.


## Getting Access to BloodHound Interface

BloodHound Kurulum ve Yükleme bölümünde, BloodHound GUI'ye bağlanmayı tartışıyoruz. Neo4j veritabanının çalıştığından emin olmamız, BloodHound uygulamasını çalıştırmamız ve tanımladığımız kullanıcı adı ve şifresiyle giriş yapmamız gerekiyor.

Başarıyla giriş yaptıktan sonra, BloodHound şu sorguyu gerçekleştirecek: `Find all Domain Admins`  Tüm Domain Admin'lerini bul ve gruba ait kullanıcıları görüntüle.

![Pasted image 20250219074252.png](/img/user/Pasted%20image%2020250219074252.png)

Bu alan, BloodHound'un **node**'ları ve bunların ilişkilerini **edge**'lerle gösterdiği Grafik Çizim Alanı olarak bilinir. Grafiği etkileşimli olarak inceleyebilir, objeleri taşıyabilir, yakınlaştırabilir veya uzaklaştırabiliriz (fare kaydırma tekerleği veya sağ alt köşedeki düğmelerle), **node**'lara tıklayarak daha fazla bilgi görebiliriz veya farklı işlemler yapmak için sağ tıklayabiliriz.

![indir.gif](/img/user/indir.gif)

Bu seçenekleri açıklayalım:

| Komut                                 | Açıklama                                                                                                                                                                                                                                   |
| ------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **Set as Starting Node**              | Bu **node**'u yol bulma aracında başlangıç noktası olarak ayarlar. Bunu tıkladığınızda, bu **node**'un adı arama çubuğunda görünecektir. Ardından, yol bulma düğmesine tıklamadan önce hedef almak için başka bir **node** seçebilirsiniz. |
| **Set as Ending Node**                | Bu **node**'u yol bulma aracında hedef **node** olarak ayarlar.                                                                                                                                                                            |
| **Shortest Paths to Here**            | Veritabanındaki herhangi bir **node**'dan bu **node**'a kadar olan en kısa yolları bulmak için bir sorgu çalıştırır. Bu işlem, neo4j'de uzun bir sorgu süresine ve BloodHound GUI'de daha uzun bir render süresine yol açabilir.           |
| **Shortest Paths to Here from Owned** | Sahip olduğunuz herhangi bir **node**'dan bu **node**'a saldırı yollarını bulur.                                                                                                                                                           |
| **Edit Node**                         | Bu, mevcut **node** üzerindeki özellikleri düzenleyebileceğiniz veya **node**'a özel özellikler ekleyebileceğiniz **node** düzenleme modalını açar.                                                                                        |
| **Mark Group as Owned**               | Bu, **node**'u içsel olarak neo4j veritabanında sahip olarak ayarlar ve ardından “Sahip Olunanlardan Buraya En Kısa Yollar” gibi diğer sorgularla kullanabilirsiniz.                                                                       |
| **Mark/Unmark Group as High Value**   | Bazı **node**'lar, varsayılan olarak "yüksek değerli" olarak işaretlenmiştir, örneğin **domain admins** grubu ve **enterprise admin** grubu. Bunu, "yüksek değerli varlıklara en kısa yollar" gibi diğer sorgularla kullanabilirsiniz.     |
| **Delete Node**                       | **Node**'u neo4j veritabanından siler.                                                                                                                                                                                                     |

**Graph Drawing Area** ayrıca **Edges** ile etkileşimde bulunmamıza da olanak tanır. **Edges**, iki **node** arasındaki bağlantıyı temsil eder ve bir **object**'ten diğerine nasıl geçeceğimizi anlamamıza yardımcı olur. BloodHound'da birçok **edge** olduğundan, her birini nasıl kötüye kullanabileceğimizi takip etmek zor olabilir. BloodHound ekibi, her bir **edge**'i nasıl kötüye kullanabileceğimize dair bilgi, örnekler ve referanslar görebileceğimiz bir yardım menüsü ekledi.

![indir (1).gif](/img/user/indir%20(1).gif)

## Search Bar

**Arama çubuğu**, BloodHound'un en çok kullanacağımız öğelerinden biridir. Belirli **objects**'leri adlarına veya türlerine göre arayabiliriz. Aradığımız bir **node**'a tıkladığımızda, o **node**'un bilgileri **node info** sekmesinde görüntülenir.

Belirli bir tür aramak istiyorsak, aramamızı **node type** ile önceden belirtebiliriz, örneğin **user:peter** veya **group:domain**. Bunu nasıl yapacağımızı şimdi görelim:

![indir (2).gif](/img/user/indir%20(2).gif)

İşte aramamıza ekleyebileceğimiz tüm **node types**  tam listesi:

**Active Directory**

- Group
- Domain
- Computer
- User
- OU
- GPO
- Container

**Azure**

- AZApp
- AZRole
- AZDevice
- AZGroup
- AZKeyVault
- AZManagementGroup
- AZResourceGroup
- AZServicePrincipal
- AZSubscription
- AZTenant
- AZUser
- AZVM


## Pathfinding

Arama çubuğundaki bir diğer harika özellik ise **Pathfinding** (Yol Bulma). Bu özellik, iki belirli **edge** (kenar) arasında bir saldırı yolu bulmamıza olanak tanır.

Örneğin, **ryan** ile **Domain Admins** arasında bir saldırı yolu arayabiliriz. Yol sonucu şöyle olur:

**Ryan > Mark > HelpDesk > ITSecurity > Domain Admins**

Bu yol, **ForceChangePassword** **edge**'ini içeriyor; diyelim ki kullanıcıların şifrelerini değiştirmeyi istemiyoruz. **Filtre** seçeneğini kullanarak **ForceChangePassword**'ı işaretini kaldırabiliriz ve bu **edge** olmadan yolu arayabiliriz. Sonuç şöyle olur:

**Ryan > AddSelf > Tech_Support > Testinggroup > HelpDesk > ITSecurity > Domain Admins**

![indir (3).gif](/img/user/indir%20(3).gif)

## Upper Right Menu

Sağ üst köşede etkileşimde bulunabileceğimiz çeşitli seçenekler bulacağız. Şimdi bu seçeneklerden bazılarını açıklayalım:

![indir (4).gif](/img/user/indir%20(4).gif)

**Refresh**: Önceki sorguyu yeniden çalıştırır ve sonuçları gösterir.

**Export Graph**: Geçerli grafiği JSON formatında kaydeder, böylece daha sonra içeri aktarılabilir. Ayrıca, geçerli grafiği PNG formatında bir resim olarak kaydedebiliriz.

**Import Graph**: Export ettiğimiz yaptığımız JSON formatındaki grafiği görüntüleyebiliriz.

![indir (5) 1.gif](/img/user/indir%20(5)%201.gif)

**Upload Data**: SharpHound, BloodHound.py veya AzureHound verilerini Neo4j'ye yükler. Yükleme verisini yükleme butonuyla seçebiliriz veya JSON ya da zip dosyasını doğrudan BloodHound penceresine sürükleyip bırakabiliriz.

![indir (6).gif](/img/user/indir%20(6).gif)

**Not:** Yükleme sırasında, BloodHound yeni verileri ekleyecek ancak yinelenen verileri görmezden gelecektir.

**Not:** Zip dosyaları, parola korumalı olarak yüklenemez.

* **Change Layout Type**: Hiyerarşik veya kuvvet yönlendirilmiş düzenler arasında geçiş yapar.

* **Settings**: Node ve edge'lerin görüntü ayarlarını ayarlamamıza olanak tanır, sorgu debug modu, düşük detay modu ve karanlık mod gibi seçenekleri etkinleştirir.

	* **Node Collapse Threshold**: Sadece bir ilişkiye sahip olan yol sonlarındaki node'ları çökertecek değeri ayarlar. 0, devre dışı bırakır, varsayılan 5.

	* **Edge Label Display**: Edge taglarını ne zaman görüntüleyeceğimizi ayarlar. Her zaman Görüntüle seçeneği, MemberOf, Contains gibi edgeleri her zaman gösterir.

	* **Node Label Display**: Node taglarını ne zaman görüntüleyeceğimizi ayarlar. Her zaman Görüntüle seçeneği, node adlarını, kullanıcı adlarını, bilgisayar adlarını vb. her zaman gösterir.

	* **Query Debug Mode**: raw sorgular raw Sorgu Box'da görüntülenir. Bu konu, Cypher Queries bölümünde daha ayrıntılı şekilde ele alınacaktır.

	* **Low Detail Mode**: Performansı artırmak için grafiksel düzenlemeler yapar.

	* **Dark Mode**: Arayüz için Karanlık modu etkinleştirir.

![Pasted image 20250219091220.png](/img/user/Pasted%20image%2020250219091220.png)


About (Hakkında): Yazılımın yazarı ve sürümü hakkında bilgi gösterir.


## Shortcuts

İşte faydalanabileceğimiz dört (4) kısayol:

| Kısayol       | Açıklama                                                                                                                                                                                                                   |
| ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **CTRL**      | CTRL tuşuna basmak, üç farklı node tag gösterim ayarını döngüsel olarak geçirecektir: varsayılan, her zaman göster, her zaman gizle.                                                                                       |
| **Spacebar**  | Spacebar tuşuna basmak, şu anda çizilen tüm nodeların listelendiği "spotlight" penceresini açar. Listeden bir öğeye tıklayarak, GUI'nin o node'a yakınlaştırma yapmasını ve kısa bir süreliğine vurgulamasını sağlarsınız. |
| **Backspace** | Backspace tuşuna basmak, önceki grafik sonucu görüntüsüne geri dönmenizi sağlar. Bu, arama çubuğundaki Geri butonuna tıklamakla aynı işlevi görür.                                                                         |
| **S**         | S harfine basmak, arama çubuğundaki Daha Fazla Bilgi butonuna tıklamakla aynı işlevi gören bilgi panelinin genişletilmesini veya daraltılmasını sağlar.                                                                    |


## Database Info

BloodHound Veritabanı Bilgisi sekmesi, kullanıcılara mevcut BloodHound veritabanı durumları hakkında hızlı bir genel bakış sağlar. Burada, kullanıcılar veritabanının oturum sayısını, ilişkilerini, ACL'lerini, Azure objelerini ve ilişkilerini ve Active Directory Objelerini görüntüleyebilirler.

Ayrıca, veritabanını yönetmek için birkaç seçenek mevcuttur

| Seçenek                    | Açıklama                                                                                                                                                                                                        |
| -------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Clear Database**         | Kullanıcıların tüm **node**'ları, ilişkileri ve özellikleri içeren veritabanını tamamen temizlemelerini sağlar. Bu, yeni bir değerlendirmeye başlarken veya eski verilere karşılık geldiğinde faydalı olabilir. |
| **Clear Sessions**         | Kullanıcıların veritabanından tüm kaydedilen sessionları temizlemelerini sağlar.                                                                                                                                |
| **Refresh Database Stats** | Görüntülenen istatistikleri, veritabanında yapılan herhangi bir değişikliği yansıtacak şekilde günceller.                                                                                                       |
| **Warming Up Database**    | Veritabanının tamamını belleğe yükleyen bir processdir, bu da sorguları önemli ölçüde hızlandırabilir. Ancak bu process, veritabanı büyükse tamamlanması zaman alabilir.                                        |

Bu bölümde, BloodHound'un grafiksel arayüzünü ve en kullanışlı özelliklerini nasıl kullanacağımızı öğrendik. Bir sonraki bölümde ise Active Directory için farklı BloodHound `node`'larını ayrıntılı olarak inceleyeceğiz.