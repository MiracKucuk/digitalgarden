---
{"dg-publish":true,"permalink":"/active-directory/active-directory/"}
---


### Neden Active Directory?
Active Directory (AD), Windows ağ ortamları için bir ==dizin== hizmetidir. Kullanıcılar, bilgisayarlar, gruplar, ağ aygıtları, dosya paylaşımları, group  policy, aygıtlar ve güvenler dahil olmak üzere bir kuruluşun kaynaklarının merkezi olarak yönetilmesini sağlayan dağıtılmış, ==hiyerarşik== bir yapıdır. AD, bir Windows domain ortamında ==kimlik doğrulama ve yetkilendirme== işlevleri sağlar. 

Geriye dönük uyumlu olacak şekilde tasarlanmıştır ve birçok özelliği tartışmalı bir şekilde “varsayılan olarak güvenli” değildir ve kolayca yanlış yapılandırılabilir. Bu zayıflık, bir ağ içinde lateral ve dikey olarak hareket etmek ve yetkisiz erişim elde etmek için kullanılabilir. AD esasen, ayrıcalık düzeylerine bakılmaksızın domain içindeki tüm kullanıcıların erişebildiği büyük bir ==salt okunur veritabanıdır==. Ek ayrıcalıkları olmayan temel bir AD kullanıcı hesabı, AD içindeki çoğu object'i numaralandırabilir.

"==Geriye dönük uyumlu olacak şekilde tasarlanmıştır==" ifadesi, Active Directory (AD) gibi sistemlerin, eski sürümleriyle veya eski sistemlerle sorunsuz çalışabilmesi için tasarlandığını belirtir. Yani, AD'nin daha eski Windows sürümleriyle, eski protokollerle veya eski altyapılarla uyumlu çalışması sağlanmıştır. Bu tür tasarımlar, eski sistemlerde çalışan yazılımların ve yapılandırmaların yeni sürümlerde de çalışmaya devam etmesini amaçlar"

![Pasted image 20240927121557.png](/img/user/resimler/Pasted%20image%2020240927121557.png)

Fortune 500 şirketlerinin yaklaşık %95'inin Active Directory kullandığı tahmin edilmektedir, bu da AD'yi saldırganlar için önemli bir odak noktası haline getirmektedir. 

Saldırganın AD ortamına standart bir domain kullanıcısı olarak girmesini sağlayan **==phishing==** gibi başarılı bir saldırı, domainin haritasını çıkarmaya ve saldırı yollarını aramaya başlamak için yeterli erişim sağlayacaktır.

Ransomware saldırganlar, saldırı yollarının önemli bir parçası olarak Active Directory'yi giderek daha fazla hedef almaktadır. Dünya çapında 400'den fazla saldırıda kullanılan Conti Fidye Yazılımının, PrintNightmare (CVE-2021-34527) ve Zerologon (CVE-2020-1472) gibi son zamanlarda ortaya çıkan kritik Active Directory açıklarından yararlanarak ayrıcalıkları artırdığı ve hedef ağda yanal olarak hareket ettiği görülmüştür. Active Directory'nin yapısını ve işlevini anlamak, bu tür açıkları saldırganlardan önce bulmak ve önlemek için bir kariyer yolundaki ilk adımdır. 


# Active Directory Structure
Active Directory (AD), Windows ağ ortamları için bir dizin hizmetidir. Kullanıcılar, bilgisayarlar, gruplar, ağ aygıtları ve dosya paylaşımları, grup policy'ler, sunucular ve workstationlar ve trustlar dahil olmak üzere bir kuruluşun kaynaklarının ==merkezi== olarak yönetilmesini sağlayan dağıtılmış, hiyerarşik bir yapıdır. AD, bir Windows domain ortamında kimlik doğrulama ve yetkilendirme işlevleri sağlar. Active Directory Domain Services (AD DS) gibi bir dizin hizmeti, bir kuruluşa dizin verilerini depolama ve aynı ağdaki hem standart kullanıcılar hem de yöneticiler için kullanılabilir hale getirme yolları sunar. AD DS (Active Directory Domain Services), kullanıcı adları ve parolalar gibi bilgileri depolar ve yetkili kullanıcıların bu bilgilere erişmesi için gereken hakları yönetir. İlk olarak Windows Server 2000 ile birlikte gönderilmiştir; son yıllarda artan saldırılara maruz kalmıştır. Geriye dönük olarak uyumlu olacak şekilde tasarlanmıştır ve birçok özelliği tartışmalı bir şekilde “varsayılan olarak güvenli” değildir. Özellikle kolayca yanlış yapılandırılabileceği büyük ortamlarda düzgün bir şekilde yönetilmesi zordur.

Active Directory kusurları ve yanlış yapılandırmaları genellikle bir dayanak noktası (“internal access”) elde etmek, bir ağ içinde yanal ve dikey olarak hareket etmek ve veritabanları, dosya paylaşımları, kaynak kodu ve daha fazlası gibi korunan kaynaklara yetkisiz erişim elde etmek için kullanılabilir.AD esasen, ayrıcalık düzeylerine bakılmaksızın domain içindeki tüm kullanıcıların erişebildiği büyük bir veritabanıdır. Ek ayrıcalıkları olmayan temel bir AD kullanıcı hesabı, bunlarla sınırlı olmamak üzere AD'de bulunan objectlerin çoğunu numaralandırmak için kullanılabilir:

|                          |                             |
| ------------------------ | --------------------------- |
| Domain Computers         | Domain Users                |
| Domain Group Information | Organizational Units (OUs)  |
| Default Domain Policy    | Functional Domain Levels    |
| Password Policy          | Group Policy Objects (GPOs) |
| Domain Trusts            | Access Control Lists (ACLs) |

Active Directory hiyerarşik bir tree yapısında düzenlenmiştir; en üstte bir veya daha fazla domain içeren bir ==forest== bulunur ve bu domainler kendi içlerinde subdomain'lere sahip olabilirler. Forest, tüm objectlerin yönetim kontrolü altında olduğu güvenlik sınırıdır. Bir forest birden fazla domain içerebilir ve bir domain başka subdomainler  içerebilir. Domain, içerdiği objectlerin (kullanıcılar, bilgisayarlar ve gruplar) erişilebilir olduğu bir yapıdır. ==Domain Controllers, Users, Computers== gibi birçok built-in Organizasyonel Birime (OU) sahiptir ve gerektiğinde yeni OU'lar oluşturulabilir. OU'lar, farklı group policy'nin atanmasına olanak tanıyan objectler ve alt OU'lar içerebilir.

Çok (basit) bir üst düzeyde, bir AD yapısı aşağıdaki gibi görünebilir:
  ```shell-session
INLANEFREIGHT.LOCAL/
├── ADMIN.INLANEFREIGHT.LOCAL
│   ├── GPOs
│   └── OU
│       └── EMPLOYEES
│           ├── COMPUTERS
│           │   └── FILE01
│           ├── GROUPS
│           │   └── HQ Staff
│           └── USERS
│               └── barbara.jones
├── CORP.INLANEFREIGHT.LOCAL
└── DEV.INLANEFREIGHT.LOCAL
```
Burada ==INLANEFREIGHT.LOCAL=='in root domain olduğunu ve subdomainleri (alt veya tree root domainleri) ==ADMIN.INLANEFREIGHT.LOCAL, CORP.INLANEFREIGHT.LOCAL== ve ==DEV.INLANEFREIGHT.LOCAL=='in yanı sıra aşağıda ayrıntılı olarak göreceğimiz gibi kullanıcılar, gruplar, bilgisayarlar ve daha fazlası gibi bir domain oluşturan diğer objectleri içerdiğini söyleyebiliriz. Çok sayıda satın alma işlemi gerçekleştiren kuruluşlarda güven ilişkileri yoluyla birbirine bağlanmış birden fazla domain (veya forest) görmek yaygındır. Mevcut domain'deki tüm yeni kullanıcıları yeniden oluşturmaktansa başka bir domain/forest ile güven ilişkisi oluşturmak genellikle daha hızlı ve kolaydır. Daha sonraki konularda göreceğimiz gibi, domain güvenleri uygun şekilde yönetilmediği takdirde bir dizi güvenlik sorununa yol açabilir.

![Pasted image 20240927123304.png](/img/user/resimler/Pasted%20image%2020240927123304.png)

Aşağıdaki grafik ==INLANEFREIGHT.LOCAL== ve ==FREIGHTLOGISTICS.LOCAL== olmak üzere iki forest göstermektedir. İki yönlü ok, iki forest arasında çift yönlü bir güveni temsil eder, yani INLANEFREIGHT.LOCAL'daki kullanıcılar FREIGHTLOGISTICS.LOCAL'daki kaynaklara erişebilir ve bunun tersi de geçerlidir. Ayrıca her root domain altında birden fazla subdomain görebiliriz. Bu örnekte, root domain'in subdomain'lerin her birine güvendiğini görebiliriz, ancak A forest'teki subdomain'lerin B forest'teki subdomain'lerle kurulmuş güvenleri olması gerekmez. Bu, ==admin.dev.freightlogistics.local'in== bir parçası olan bir kullanıcının, üst düzey inlanefreight.local ve freightlogistics.local domain'leri arasında çift yönlü bir güven olmasına rağmen varsayılan olarak ==wh.corp.inlanefreight.local== domain'indeki makinelere kimlik doğrulaması YAPAMAYACAĞI anlamına gelir. admin.dev.freightlogistics.local ve wh.corp.inlanefreight.local adreslerinden doğrudan iletişime izin vermek için başka bir güven oluşturulması gerekir.

![Pasted image 20240927123430.png](/img/user/resimler/Pasted%20image%2020240927123430.png)

Hangi Active Directory yapısı bir veya daha fazla domain içerebilir? (Forest)

Doğru veya Yanlış; Güven ilişkileri ile birbirine bağlanmış birden fazla domain görmek yaygın olabilir mi?  (Yes)

Active Directory, bir Windows domain ortamında kimlik doğrulama ve <____> sağlar. (authorization (yetkilendirme))


### Active Directory Terminolojisi

#### Object (object)
Bir obje, Active Directory ortamında bulunan OU'lar, yazıcılar, kullanıcılar, domain controllerlar vb. gibi HERHANGİ bir kaynak olarak tanımlanabilir.

#### Attributes (Nitelikler)
**Active Directory (AD)**, ağdaki tüm **object**'leri (örneğin kullanıcılar, gruplar, bilgisayarlar, vb.) saklayan ve yöneten bir hizmettir. Her **object** (örneğin bir bilgisayar veya kullanıcı), o **object**'in özelliklerini tanımlayan bir dizi **[attribute](https://learn.microsoft.com/en-us/windows/win32/adschema/attributes-all)** ile tanımlanır.

Örneğin:

- Bir **computer object** (bilgisayar object'i) Active Directory'de, bilgisayarın adı (hostname) veya DNS adı gibi **attribute**'lara sahip olabilir.
- Bir **user object** (kullanıcı object'i) ise, o kullanıcının adı, soyadı, e-posta adresi gibi bilgilerle tanımlanır.

Bu **attributes** (özellikler), **object**'in tanımlanmasını ve yönetilmesini sağlar.

#### LDAP ve AD Attributes:

**LDAP** (Lightweight Directory Access Protocol), Active Directory ile iletişim kurmak için kullanılan bir protokoldür. **LDAP sorguları** kullanılarak AD'deki **object**'lere ve bu **object**'lerin **attribute**'lerine erişilebilir.

Her **attribute** (özellik), bir **LDAP adı** ile ilişkilidir. LDAP adı, o **attribute**'in tanımını yapar. Yani, **Active Directory**'deki her özellik, **LDAP sorgularında** kullanılabilen bir **LDAP adı** ile eşleşir.

#### Örnekler:

- **Full Name** (Tam İsim) için, Active Directory'de kullanılan **attribute** **`displayName`**'dir.
- **First Name** (İlk İsim) için ise, Active Directory'de kullanılan **attribute** **`givenName`**'dir.

**displayName** ve **givenName** gibi LDAP adları, bu **attribute**'leri sorgulamak veya değiştirmek için kullanılır.

#### Özet:

Her **object**, özelliklerini tanımlayan **attributes**'lere sahiptir ve bu **attributes**'lar, LDAP ile erişilebilir. Bu sayede AD'deki **object**'lerin özelliklerine kolayca ulaşılabilir ve yönetilebilir.


### Schema
**Active Directory Schema**, **Active Directory**'deki **object**'lerin nasıl yapılandırıldığını ve birbirleriyle nasıl ilişkilendirildiklerini tanımlayan bir **plan** gibidir. Bu **Schema**, ağdaki tüm **object**'lerin türlerini, bu **object**'lerin hangi özelliklere sahip olduklarını (attributes) ve her bir **object**'in nasıl düzenleneceğini belirler.

#### Schema Ne İşe Yarar?

**Schema**, **Active Directory**'nin temel yapı taşlarını oluşturur. Yani, **Active Directory**'de hangi **object** türlerinin (örneğin kullanıcılar, bilgisayarlar, gruplar) bulunduğunu ve bu **object**'lerin sahip olduğu özellikleri tanımlar.

**Örneğin:**

- **Kullanıcılar** (users), **user** sınıfına aittir.
- **Bilgisayarlar** (computers), **computer** sınıfına aittir.
- **Gruplar** (groups), **group** sınıfına aittir.

Her **object**'in belirli **attributes** (özellikleri) vardır. Bu **attributes**, o **object**'in çeşitli bilgilerini içerir. Örneğin:

- **User** object'leri için, **first name** (ilk isim), **last name** (soy isim), **email address** (e-posta adresi) gibi **attributes** vardır.
- **Computer** object'leri için, **hostname** (ana bilgisayar adı), **IP address** (IP adresi), **operating system** (işletim sistemi) gibi **attributes** bulunur.

#### Sınıflar ve Object Örneklemesi (Instantiation)

**Schema**, sadece **object**'lerin türlerini (sınıflarını) değil, aynı zamanda her **object**'in nasıl oluşturulacağını ve hangi özelliklere sahip olması gerektiğini de tanımlar.

- **Sınıf** (Class): **Schema** içinde tanımlanan bir kategori ya da türdür. Örneğin, **user** ve **computer** birer **sınıf**tır.
- **Object örneği (Instance)**: Bir **sınıf**tan oluşturulmuş bir nesnedir. Yani, bir **sınıf**'ın belirli bir örneği **object**'tir.

Örneğin:

- **computer** sınıfı, **bilgisayar object**'lerini tanımlar.
- **RDS01** adlı bir bilgisayar, **computer** sınıfının bir örneğidir.

Bu durumda, **RDS01** bilgisayarı bir **object**'tir ve **computer** sınıfının bir örneğidir. **Schema**, bu **object**'in hangi özelliklere (attributes) sahip olacağını belirler.

#### Özet:

- **Active Directory Schema**, **object**'lerin türlerini (sınıflarını) ve her bir **object**'in sahip olması gereken **attributes**'ları tanımlar.
- **Sınıf**: **Object**'lerin türüdür (örneğin **user** veya **computer**).
- **Object örneği** (Instance): Bir **sınıf**tan oluşturulmuş gerçek bir nesnedir (örneğin, **RDS01** bilgisayar **computer** sınıfının bir örneğidir).

Bu şekilde **Schema**, Active Directory'deki tüm **object**'lerin nasıl oluşturulacağı ve nasıl düzenleneceği konusunda bir kılavuz işlevi görür.


### Domain
Domain, bilgisayarlar, kullanıcılar, OU'lar, gruplar vb. gibi objectlerden oluşan mantıksal bir gruptur. Her bir domain'i bir eyalet veya ülke içindeki farklı bir şehir olarak düşünebiliriz. Domain'ler birbirlerinden tamamen bağımsız olarak çalışabilir veya güven ilişkileri yoluyla birbirlerine bağlanabilir.


### Forest
Forest, Active Directory domain'lerinden oluşan bir koleksiyondur. En üst kapsayıcıdır ve domainler, kullanıcılar, gruplar, bilgisayarlar ve Group Policy objectleri dahil ancak bunlarla sınırlı olmamak üzere aşağıda tanıtılan tüm AD objectlerini içerir. Bir forest bir veya birden fazla domain içerebilir ve ABD'deki bir eyalet veya AB'deki bir ülke gibi düşünülebilir. Her forest bağımsız olarak çalışır ancak diğer forest'larla çeşitli güven ilişkilerine sahip olabilir.


### Tree
**Active Directory Tree (Ağaç)**, tek bir **root domain**'den başlayan ve birbiriyle ilişkili olan **domain**'lerin oluşturduğu bir koleksiyondur. Başka bir deyişle, **domain**'ler bir araya gelerek bir **tree**'yi (ağaç yapısını) oluşturur.

Bir **Forest** ise, birden fazla **tree**'nin bir araya gelerek oluşturduğu daha büyük bir yapıdır. Yani, **forest** birden fazla **tree**'yi barındıran bir koleksiyondur.

#### Domain'ler Arasındaki İlişki:

Bir **tree**'deki her **domain**, diğer **domain**'lerle bir sınır paylaşır. Bu, domain'lerin birbirine bağlanmasını sağlar. Eğer bir **domain**, başka bir **domain**'in altına eklenirse, bu iki **domain** arasında bir **parent-child** (ebeveyn-çocuk) güven ilişkisi oluşur. Yani, bir **parent domain**'i, **child domain**'in üzerinde denetim sağlar.

#### Farklı **Tree**'ler ve **Subdomain**'ler:

Bir **forest**'ın içinde birden fazla **tree** olabilir. Fakat, aynı **forest** içindeki iki **tree**, aynı **ad** alanını paylaşamaz. Yani her **tree** kendine ait benzersiz bir **ad alanı** (domain adı) kullanır.

Örneğin, bir **forest**'ta iki **tree** olduğunu varsayalım:

- **inlanefreight.local** (ilk **tree**)
- **ilfreight.local** (ikinci **tree**)

Bu durumda:

- **inlanefreight.local** tree'inin **subdomain**'i **corp.inlanefreight.local** olabilir.
- **ilfreight.local** tree'inin **subdomain**'i ise **corp.ilfreight.local** olabilir.

Yani, her **tree** farklı bir **ad alanı**na sahip olsa da, her **tree**'de kendi içindeki **subdomain**'ler **parent-child** ilişkisini takip eder.

#### Global Katalog:

Bir **tree**'deki tüm **domain**'ler, o **tree**'ye ait **object**'ler hakkında tüm bilgileri içeren ortak bir **Global Katalog** paylaşır. Bu katalog, **tree** içindeki tüm **domain**'lerdeki **object** bilgilerini merkezi bir şekilde saklar ve bu bilgilere kolayca erişilmesini sağlar.

#### Özet:

- **Tree**, bir **root domain**'den başlayan ve birbirleriyle ilişkili **domain**'lerin oluşturduğu koleksiyondur.
- **Forest**, birden fazla **tree**'yi barındıran yapıdır.
- Her **tree**, kendi benzersiz **ad alanı**na sahiptir ve **parent-child** ilişkileri ile güven bağları kurar.
- Bir **Global Katalog**, bir **tree** içindeki tüm **domain**'ler ve **object** bilgilerini merkezi olarak depolar.

### Container
**Container** (Konteyner) **object**'leri, **Active Directory**'de **diğer object**'leri tutan, yani içinde başka **object**'lerin barındığı bir yapıdır. **Container object**'leri, **dizin** (directory) yapısındaki **alt tree** hiyerarşisinde belirli bir konumda yer alır.

#### Container Nedir?

**Container object**'leri, adından da anlaşılacağı gibi, bir tür "kutudur" ve başka **object**'leri bu kutunun içinde saklar. Bu **object**'ler genellikle kullanıcılar, bilgisayarlar, gruplar ve diğer kaynaklar olabilir.

Örnek:

- **Organizational Unit (OU)**: Bu, **container object**'lerinin en yaygın örneklerinden biridir. Bir **OU**, kullanıcılar, gruplar, bilgisayarlar gibi **object**'leri organize etmek ve yönetmek için kullanılır. Yani bir **OU**, içinde farklı türdeki **object**'leri barındıran bir **container**'dır.
    
- **Domain**: Bir **domain** de bir **container**'dır, çünkü bir **domain** içinde birçok farklı **object** (kullanıcılar, bilgisayarlar, gruplar, vb.) bulunur.
    

#### Container ile Diğer Object'ler Arasındaki Fark:

- **Container**: Diğer **object**'leri tutar ve hiyerarşik yapıda yer alır.
- **Object**: Kendine ait özellikleri ve bilgileri olan gerçek bir nesnedir. Örneğin, bir **user object** veya **computer object**.

#### Container'ın Hiyerarşideki Yeri:

**Container object**'leri, **Active Directory**'nin hiyerarşisinde belirli bir konumda bulunur ve altında başka **object**'ler yer alır. Bu yapılar, **Active Directory**'deki **tree** veya **forest** yapılarının düzenli ve mantıklı bir şekilde organize edilmesini sağlar.

Örnek:

- **Active Directory Tree** içinde bir **domain**, bir **container object** olabilir ve bu **domain** içinde **organizational units (OU)** gibi daha küçük **container**'lar olabilir.
- Bir **OU**, içinde birden çok **user** ve **computer object**'i barındırabilir.

#### Özet:

**Container** object'leri, **Active Directory**'de diğer **object**'leri içinde barındıran ve hiyerarşinin bir parçası olan yapılardır. Bir **container**, diğer **object**'lerin düzenli bir şekilde gruplandırılmasını sağlar ve bu şekilde yönetilebilir hale gelir.


### Leaf
**Leaf** (Yaprak) **object**'leri, **Active Directory**'de başka **object**'leri **içermeyen**, yani kendi başlarına tek başına var olan **object**'lerdir. Bu **object**'ler, **Active Directory**'deki hiyerarşinin **son** noktasında yer alır, yani altlarında başka bir **object** barındırmazlar.

#### Leaf Nedir?

**Leaf object**'leri, diğer **object**'leri içermeyen ve genellikle aktif olarak kullanılan **nesneler**dir. Bu **object**'ler, **container**'lardan (kapsayıcılar) farklı olarak başka **object**'leri **içermez**. **Leaf object**'leri, kendi başlarına **sadece bir varlık** olarak bulunur ve **Active Directory**'de belirli bir işlevi yerine getirir.

#### Leaf ve Container Arasındaki Fark:

- **Container**: Diğer **object**'leri içeren ve bir hiyerarşide yer alan **object**'tir (örneğin **Organizational Unit (OU)**).
- **Leaf**: **Başka object**'leri içermeyen ve genellikle tek başına var olan **object**'tir (örneğin, bir kullanıcı, bir bilgisayar veya bir grup).

### Leaf Object Örnekleri:

- **User Object**: Bir **kullanıcı** **leaf object**'idir. Çünkü bir kullanıcı **başka object**'leri içermez ve kendi başına bir varlıktır.
- **Computer Object**: Bir **bilgisayar** da bir **leaf object**'idir. Yine başka **object**'leri içermez ve sadece bilgisayarı tanımlar.
- **Group Object**: Bir **grup** da bir **leaf object**'idir. Bu grup, üyelerini içeriyor olabilir, ancak grup kendisi başka **object**'leri barındırmaz, sadece grup olarak bir varlık olarak bulunur.

#### Leaf'in Hiyerarşideki Yeri:

**Leaf object**'leri, **container**'ların altında yer alır ve altlarında başka bir **object** bulunmaz. Örneğin, bir **container** olan **Organizational Unit (OU)** içinde **user**, **computer**, **group** gibi **leaf object**'ler bulunabilir.

#### Özet:

- **Leaf object**'leri, **Active Directory**'de başka **object**'leri içermeyen ve genellikle hiyerarşinin en alt seviyesinde bulunan **object**'lerdir.
- **Container object**'leri, diğer **object**'leri içerebilirken, **leaf object**'leri tek başlarına varlıklarını sürdürürler ve başka **object**'lere sahip değildir.


### Global Unique Identifier (Küresel Benzersiz Tanımlayıcı) (GUID)

#### 1. **GUID Nedir?**

- **[GUID](https://docs.microsoft.com/en-us/windows/win32/adschema/a-objectguid)**, "Globally Unique Identifier" (Küresel Benzersiz Tanımlayıcı) anlamına gelir.
- Bu, **Active Directory**'de her bir **object**'e (kullanıcı, grup, bilgisayar, domain, vb.) atanan **128 bitlik benzersiz bir değerdir**.
- **GUID**, tıpkı **MAC adresi** gibi kuruluş genelinde **benzersizdir**, yani her **object** için farklı ve bir daha tekrarlanmayan bir değerdir.

#### 2. **GUID Nerelerde Kullanılır?**

- Her **Active Directory object**'i (kullanıcı, grup, bilgisayar, domain, vb.) oluşturulduğunda bir **GUID** atanır.
- Bu **GUID**, **ObjectGUID** adı verilen bir **öznitelikte** (attribute) saklanır.
- **GUID**, **Active Directory** tarafından **object**'leri tanımlamak için kullanılır ve her **object**'in **eşsiz** olarak tanımlanmasını sağlar.

#### 3. **GUID ve Diğer Özellikler:**

- **ObjectGUID**, **Active Directory**'deki **object**'in kimliğini belirleyen **değişmez** bir özelliktir.
- Bir **object**, **Active Directory**'de var olduğu sürece, her zaman aynı **GUID**'yi taşır. Bu değer **asla değişmez**.

#### 4. **GUID'nin Kullanım Amacı:**

- **GUID**, bir **object**'in **benzersiz kimliği** olarak kullanılır.
- Bu, özellikle büyük **Active Directory** ortamlarında, **benzer isimlere sahip object**'ler olabileceği için, **GUID** değeri sayesinde **object**'i **kesin olarak tanımlayabiliriz**.
- **GUID** kullanarak arama yapmak, özellikle benzer adlar veya çakışmalar olduğunda **en doğru** ve **kesin sonuçları** almak için çok faydalıdır.

#### 5. **PowerShell ile GUID Arama:**

- PowerShell kullanarak **Active Directory**'de bir **object**'i sorgularken, **ObjectGUID** değerini sorgulayabilirsiniz.
- **ObjectGUID**'yi sorgulamak, **object**'i tam olarak bulmak için **en güvenilir** yöntemlerden biridir.
- **PowerShell** ile sorgularken, **objectGUID**, **SID** (Security Identifier), **SAM hesap adı** gibi farklı özellikler kullanarak **Active Directory object**'ini arayabilirsiniz.

#### 6. **GUID'nin Değişmezliği:**

- Bir **object**'e atanan **GUID**, o **object**'in **ömrü boyunca değişmez**. Yani bir kez oluşturulmuş bir **GUID**, **Active Directory**'deki **object**'le sürekli ilişkilendirilir.
- Bu özellik, **object**'in tanımlanmasını ve takip edilmesini kolaylaştırır.

#### 7. **Active Directory'de Arama Yaparken GUID'nin Önemi:**

- **GUID** kullanarak arama yapmak, özellikle **Global Katalog**'da **benzer isimler** varsa, **object**'i bulmada en doğru yol olabilir.
- **GUID**, **object**'i **kesin bir şekilde tanımladığı** için, adının benzer olduğu başka **object**'lerle karışma riskini ortadan kaldırır.

#### Özet:

- **GUID**, her **Active Directory object**'ine atanan **benzersiz** ve **değişmeyen** bir değerdir.
- **ObjectGUID**, **object**'in kimliğini belirler ve **Active Directory**'deki tüm **object**'ler için bu değer kesindir.
- **PowerShell** gibi araçlar ile **GUID** değeri kullanılarak **object**'lere en doğru şekilde ulaşılabilir ve arama yapılabilir.


### Security principals
#### 1. **Security Principals (Güvenlik İlkeleri) Nedir?**

- **Security principals**, **Active Directory** ve işletim sistemi içindeki **kimlik doğrulaması** yapılabilen her şeydir.
- Bu **principals**, genellikle **kullanıcı hesapları**, **bilgisayar hesapları** ve hatta **servis hesapları** gibi özel hesaplar olabilir.
- Örneğin, bir **domain içindeki** bir **servis hesabı** (örneğin **Tomcat** gibi bir uygulamanın çalıştığı hesap) da bir **security principal**'dır.

#### 2. **Security Principals'ın Tanımladığı Kimlikler:**

- **User Accounts (Kullanıcı Hesapları)**: İnsan kullanıcıları temsil eder. Bu hesaplar, kullanıcıların sisteme giriş yapmalarını ve kaynaklara erişimlerini sağlar.
- **Computer Accounts (Bilgisayar Hesapları)**: **Domain**'e katılan bilgisayarların hesaplarıdır. Bu hesaplar, bilgisayarların **Active Directory**'de tanımlanmasını sağlar.
- **Thread'ler ve Process'ler**: Bir **kullanıcı** veya **bilgisayar hesabı** çerçevesinde çalışan threadler veya processes de **security principal** olarak kabul edilir.
    - Örneğin, **Tomcat** gibi bir uygulamanın **domain içindeki** çalıştığı **hizmet hesabı** bir **security principal**'dir.

#### 3. **Active Directory'deki Security Principles:**

- **AD'deki Security Principles, domain içindeki kaynaklara** erişimi yönetmek için kullanılan **object'ler**'dir.
- Bu **security principal**'ler, **Active Directory**'de tanımlanan **kullanıcılar** ve **gruplar** gibi nesneler olabilir.
- **Active Directory**, bu **security principal**'lerin kimlik doğrulamasını yaparak, hangi **domain kaynaklarına** erişebileceğine karar verir.

#### 4. **Local Kullanıcı Hesapları ve Güvenlik Grupları:**

- **local kullanıcı hesapları** ve **güvenlik grupları**, sadece belirli bir **bilgisayar** üzerindeki kaynaklara erişim kontrolünü yönetir.
- Bu hesaplar ve gruplar, **Active Directory** tarafından değil, **Security Accounts Manager (SAM)** tarafından yönetilir.
    - **SAM**, Windows işletim sisteminin bir parçasıdır ve local güvenlik bilgilerini depolar.
    - **SAM**, sadece bilgisayarın kendi kaynaklarına erişim için kullanılan hesapları yönetir, **Active Directory**'nin dışında çalışır.

#### 5. **Security Accounts Manager (SAM):**

- **SAM**, Windows işletim sistemi içindeki bir bileşendir.
- **SAM**, **Active Directory**'nin dışında kalan ve **local** olarak yönetilen kullanıcı hesapları ve güvenlik gruplarının yönetiminden sorumludur.
- **SAM**'deki hesaplar, sadece o bilgisayar üzerindeki local kaynaklara erişim sağlar.

#### Özet:

- **Security principals**, **Active Directory** veya işletim sistemi içindeki **kullanıcılar**, **bilgisayar hesapları**, **servis hesapları** gibi kimlik doğrulaması yapılabilen her şeydir.
- **AD Security Principles, **domain içindeki kaynaklara** erişimi yönetmek için kullanılan **object'ler**'dir.
- **local kullanıcı hesapları** ve **güvenlik grupları**, **SAM** tarafından yönetilir ve sadece local bilgisayarın kaynaklarına erişimi kontrol eder. **Active Directory** ise daha geniş bir ağ içindeki kaynakları yönetir.


### Security Identifier (Güvenlik Tanımlayıcısı) (SID)
#### 1. **SID Nedir?**

- **Security Identifier (SID)**, bir security principal veya **security grubu** için **benzersiz** bir tanımlayıcıdır.
- Her **hesap**, **grup** veya **process** için, **Active Directory** ortamında, **domain controller** tarafından verilen ve **güvenli bir veritabanında** saklanan **kendine özgü** bir SID bulunur.

#### 2. **SID'in Benzersizliği:**

- Bir SID **bir kez kullanılır** ve bir daha **tekrar kullanılmaz**. Yani her SID, sadece bir security policy veya **grup** ile ilişkilidir.
- Security policy silinse bile, o SID **asla başka bir kullanıcı veya grup için kullanılmaz**. Bu, SID'in her zaman **benzersiz** olduğunu ve her bir SID'nin belirli bir güvenlik ilkesine ait olduğunu garanti eder.

#### 3. **SID ve Erişim Token'ları:**

- Bir **kullanıcı** **oturum açtığında**, sistem, kullanıcının **SID**'sini içeren bir **access token'ı** oluşturur.
- Bu token, kullanıcının **haklarını** (permissions) ve kullanıcının **üyesi olduğu tüm grupların SID'lerini** içerir.
- Bu **access token'ı**, kullanıcının bilgisayarda gerçekleştirdiği **eylemler sırasında** haklarını kontrol etmek için kullanılır.

#### 4. **SID'in Rolü ve Kullanımı:**

- **SID**, kullanıcı veya grup **kimliğini tanımlamak** için kullanılır.
- **Erişim kontrolü** sağlamak için, sistem **SID**'yi kullanarak hangi kullanıcının veya grubun hangi kaynaklara erişebileceğini belirler.
- Bu, özellikle **güvenlik politikaları** ve **kimlik doğrulama** süreçlerinde çok önemlidir.

#### 5. **İyi Bilinen SID'ler:**

- **İyi bilinen SID'ler**, **genel** kullanıcıları ve **grupları** tanımlamak için kullanılır.
- Bu SID'ler, tüm **işletim sistemlerinde** aynıdır ve **standart** SID'lerdir.
    - Örnek: **Everyone** grubu.
- **Everyone** grubu, **Windows işletim sistemlerinde** varsayılan olarak bulunan ve tüm kullanıcılar tarafından erişilebilen bir grup olduğundan, **her ortamda aynı SID'yi** kullanır.

#### 6. **SID'in Güvenlikteki Önemi:**

- SID'ler, **sistem yönetimi** ve **erişim kontrolü** açısından kritik öneme sahiptir.
- **Erişim kontrol listeleri (ACL)** gibi yapılar, **SID'leri** kullanarak hangi kullanıcıların veya grupların belirli bir kaynağa erişebileceğini belirler.
- **SID**'ler, sistemdeki **herhangi bir değişikliği** izlemek ve güvenlik ihlallerini tespit etmek için de kullanılır.

#### Özet:

- **SID**, her **hesap**, **grup** veya **işlem** için benzersiz bir tanımlayıcıdır ve sadece bir kez kullanılır.
- **Erişim token'ları**, kullanıcının SID'sini ve haklarını içererek, kullanıcının bilgisayarda gerçekleştirdiği işlemlerde **hak kontrolü** yapar.
- **İyi bilinen SID'ler**, genel grupları tanımlar ve tüm işletim sistemlerinde aynıdır (örneğin, **Everyone** grubu).
- SID'ler, **erişim kontrolü** ve **güvenlik yönetimi** açısından çok önemlidir.


### Distinguished Name (Ayırt Edici Ad) (DN)
#### 1. **Distinguished Name (DN) Nedir?**

- [**Distinguished Name](https://docs.microsoft.com/en-us/previous-versions/windows/desktop/ldap/distinguished-names) (DN)**, **Active Directory**'deki bir **object**'in **tam yolunu** tanımlar.
- Bu yol, object'in bulunduğu **yerin tam adresi** gibi düşünülebilir ve AD içerisindeki hiyerarşiyi takip eder.
- Örneğin, **cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local** şeklinde bir DN örneği, **bjones** kullanıcısının yerini tanımlar.

#### 2. **DN İçindeki Bileşenler:**

- **CN (Common Name):** Bu, object'in **adı** veya **ortak adı** olarak bilinir. Örneğin, `cn=bjones` kısmı, **bjones** kullanıcısını tanımlar.
- **OU (Organizational Unit):** Bu, object'in **bulunduğu birim** veya **organizasyonel birim**'i belirtir. `ou=IT` ve `ou=Employees` gibi bölümler, kullanıcının hangi organizasyonel birimde yer aldığını gösterir.
- **DC (Domain Component):** Bu, **domain**'in bileşenlerini tanımlar. `dc=inlanefreight, dc=local`, Inlanefreight şirketinin **domain**'ine ait bilgileri belirtir.

#### 3. **DN'nin Kullanımı:**

- **Distinguished Name**, **object'in** AD içinde aranabilmesi veya erişilebilmesi için **tam yol** sağlar.
- Bu yol, **Active Directory**'deki her object'e özgüdür ve bir object’in **kesin kimliğini** tanımlar.

#### 4. **Örnek:**

- **cn=bjones, ou=IT, ou=Employees, dc=inlanefreight, dc=local**:
    - **bjones**: Kullanıcı adı (Common Name).
    - **IT**: Kullanıcının **IT departmanı** içinde olduğu belirtiliyor.
    - **Employees**: Kullanıcıların bulunduğu **organizasyonel birim** (OU).
    - **inlanefreight.local**: Domain adı, Inlanefreight şirketine ait.

#### 5. **Distinguished Name ve Erişim:**

- DN, object'in **tam ve benzersiz** bir şekilde bulunmasını sağlar. AD içinde aynı isme sahip birden fazla object olabilir, ancak her object'in **DN**'i benzersizdir.
- Bu, **LDAP sorguları** veya **Active Directory yönetimi** sırasında doğru object'e erişim sağlamak için kullanılır.

#### Özet:

- **Distinguished Name (DN)**, **Active Directory**'deki bir object'in **tam yolunu** tanımlar ve **benzersiz** bir kimlik sağlar.
- **DN**'deki bileşenler (CN, OU, DC) object'in **bulunduğu yer** hakkında bilgi verir.
- **DN**, object'lerin **kesin bir şekilde tanımlanmasını** ve **erişilmesini** sağlar


### Relative Distinguished Name (RDN)
[Relative Distinguished Name](https://docs.microsoft.com/en-us/windows/win32/ad/object-names-and-identities) (RDN), objectyi adlandırma hiyerarşisinde geçerli düzeydeki diğer objectlerden benzersiz olarak tanımlayan Distinguished Name'in tek bir bileşenidir. Örneğimizde, bjones objectnin Relative Distinguished Name (Göreli Ayırt Edici Ad) öğesidir. AD, aynı üst kapsayıcı altında aynı ada sahip iki objectye izin vermez, ancak farklı DN'lere sahip oldukları için domainde yine de benzersiz olan aynı RDN'lere sahip iki object olabilir. Örneğin, cn=bjones,dc=dev,dc=inlanefreight,dc=local objectsi cn=bjones,dc=inlanefreight,dc=local objectsinden farklı olarak tanınır.

![Pasted image 20240927132125.png](/img/user/resimler/Pasted%20image%2020240927132125.png)

### sAMAccountName
[sAMAccountName](https://learn.microsoft.com/en-us/windows/win32/ad/naming-properties#samaccountname) kullanıcının oturum açma adıdır. Burada sadece bjones olacaktır. Benzersiz bir değer ve 20 veya daha az karakter olmalıdır.

### userPrincipalName
[userPrincipalName](https://social.technet.microsoft.com/wiki/contents/articles/52250.active-directory-user-principal-name.aspx) özniteliği, AD'deki kullanıcıları tanımlamanın başka bir yoludur. Bu öznitelik, bjones@inlanefreight.local biçiminde bir önek (kullanıcı hesabı adı) ve bir sonekten ( domain adı) oluşur. Bu öznitelik zorunlu değildir.


### FSMO Rolleri
AD'nin ilk günlerinde, bir ortamda birden fazla DC'niz varsa, hangi DC'nin değişiklik yapacağı konusunda kavga ederlerdi ve bazen değişiklikler düzgün yapılmazdı. Microsoft daha sonra “son yazan kazanır” uygulamasını hayata geçirdi, bu da son değişiklik işleri bozarsa kendi sorunlarını ortaya çıkarabilirdi. Daha sonra tek bir “ master” DC'nin domain'e değişiklikleri uygulayabildiği, diğerlerinin ise sadece kimlik doğrulama isteklerini yerine getirdiği bir model getirdiler. Bu hatalı bir tasarımdı çünkü master DC devre dışı kalırsa, geri yüklenene kadar ortamda hiçbir değişiklik yapılamıyordu. Bu tek hata noktası modelini çözmek için Microsoft, bir DC'nin sahip olabileceği çeşitli sorumlulukları [Flexible Single Master Operation](https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/fsmo-roles) (FSMO) rollerine ayırdı. Bunlar Domain Controller'lara (DC) kesintisiz olarak kullanıcıların kimliklerini doğrulamaya ve izinler vermeye devam etme (yetkilendirme ve kimlik doğrulama) olanağı verir. Beş FSMO rolü vardır: Schema Master ve Domain Naming Master (forest başına birer tane), Relative ID (RID) Master ( domain başına bir tane), Primary Domain Controller (PDC) Emulator (domain başına bir tane) ve Infrastructure Master (domain başına bir tane). Beş rolün tümü, yeni bir AD forest'ında forest root domain'indeki ilk DC'ye atanır. Bir forest'a her yeni domain eklendiğinde, yeni domain'e yalnızca RID Master, PDC Emulator ve Infrastructure Master rolleri atanır. FSMO rolleri genellikle domain controller'lar oluşturulduğunda ayarlanır, ancak gerekirse sysadmins bu rolleri aktarabilir. Bu roller AD'de çoğaltmanın sorunsuz çalışmasına yardımcı olur ve kritik hizmetlerin doğru şekilde çalışmasını sağlar. Bu rollerin her birini bu bölümün ilerleyen kısımlarında ayrıntılı olarak inceleyeceğiz.


### Global Catalog
[Global katalog](https://learn.microsoft.com/en-us/windows/win32/ad/global-catalog) (GC), Active Directory forest'taki TÜM objectlerin kopyalarını depolayan bir domain controller'dır. GC, geçerli domain'deki tüm objectlerin tam bir kopyasını ve forest'taki diğer domain'lere ait objectlerin kısmi bir kopyasını saklar. Standart domain controller'lar kendi domain'ine ait objectlerin tam bir kopyasını tutar ancak forest'taki farklı domain'lere ait objectlerin kopyasını tutmaz. GC, hem kullanıcıların hem de uygulamaların forest'ındaki HERHANGİ bir domain'deki herhangi bir object hakkında bilgi bulmasını sağlar. GC, bir domain controller üzerinde etkinleştirilen bir özelliktir ve aşağıdaki fonksiyonları yerine getirir:
* Authentication (bir access token oluşturulduğunda dahil edilen, bir kullanıcı hesabının ait olduğu tüm gruplar için sağlanan yetkilendirme)
* Obje arama (bir forest içindeki dizin yapısını şeffaf hale getirerek, bir obje hakkında sadece bir öznitelik sağlayarak bir forest içindeki tüm domainler arasında arama yapılmasına izin verir).


### Read-Only Domain Controller (RODC)
[Read-Only Domain Controller](https://learn.microsoft.com/en-us/windows/win32/ad/rodc-and-active-directory-schema) (RODC) sadece okunabilir bir Active Directory veritabanına sahiptir. Bir RODC'de hiçbir AD hesabı parolası önbelleğe alınmaz (RODC bilgisayar hesabı ve RODC KRBTGT parolaları dışında.) Bir RODC'nin AD veritabanı, SYSVOL veya DNS aracılığıyla hiçbir değişiklik dışarı aktarılmaz. RODC'ler ayrıca salt okunur bir DNS sunucusu içerir, yönetici rolü ayrımına izin verir, ortamdaki replikasyon trafiğini azaltır ve SYSVOL değişikliklerinin diğer DC'lere replike edilmesini önler.


### Replication
[Replikasyon](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/get-started/replication/active-directory-replication-concepts), AD objectleri güncelleştirildiğinde ve bir Domain Controller'dan diğerine aktarıldığında AD'de gerçekleşir. Bir DC eklendiğinde, bunlar arasındaki çoğaltmayı yönetmek için bağlantı objectleri oluşturulur. Bu bağlantılar, tüm DC'lerde bulunan Knowledge Consistency Checker (KCC) hizmeti tarafından yapılır. Replikasyon, değişikliklerin bir forestdaki diğer tüm DC'lerle senkronize edilmesini sağlar ve bir domain controller'ın arızalanması durumunda bir yedek oluşturulmasına yardımcı olur.


### Service Principal Name  (SPN)
[Service Principal Name](https://learn.microsoft.com/en-us/windows/win32/ad/service-principal-names) (SPN) bir hizmet örneğini benzersiz bir şekilde tanımlar. Kerberos kimlik doğrulaması tarafından bir hizmet örneğini bir oturum açma hesabıyla ilişkilendirmek için kullanılırlar ve bir istemci uygulamanın hesap adını bilmesine gerek kalmadan bir hesabın kimliğini doğrulamak için hizmetten talepte bulunmasına olanak tanırlar.


### Group Policy Object (GPO)
[Group Policy objectleri (GPO'lar)](https://learn.microsoft.com/en-us/previous-versions/windows/desktop/policy/group-policy-objects) ilke ayarlarının sanal koleksiyonlarıdır. Her GPO'nun benzersiz bir GUID'si vardır. Bir GPO local dosya sistemi ayarlarını veya Active Directory ayarlarını içerebilir. GPO ayarları hem kullanıcı hem de bilgisayar objectlerine uygulanabilir. Domain içindeki tüm kullanıcılara ve bilgisayarlara uygulanabilir veya OU düzeyinde daha ayrıntılı olarak tanımlanabilirler.


### Access Control List (ACL)
[Erişim Kontrol Listesi (ACL)](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-lists), bir object için geçerli olan Erişim Kontrol Girişlerinin (ACE'ler) sıralı koleksiyonudur.

### Access Control Entries (ACEs)
Bir ACL'deki her [Erişim Denetimi Girişi ](https://learn.microsoft.com/en-us/windows/win32/secauthz/access-control-entries)(ACE) bir mutemedi (kullanıcı hesabı, grup hesabı veya oturum açma oturumu) tanımlar ve söz konusu mutemet için izin verilen, reddedilen veya denetlenen erişim haklarını listeler.

### Discretionary Access Control List (DACL)
DACL'ler hangi güvenlik ilkelerine bir objectye erişim izni verildiğini veya reddedildiğini tanımlar; ACE'lerin bir listesini içerir. Bir işlem güvenli bir objectye erişmeye çalıştığında, sistem erişim izni verip vermeyeceğini belirlemek için objectnin DACL'sindeki ACE'leri kontrol eder. Bir objectnin DACL'si yoksa, sistem herkese tam erişim izni verir, ancak DACL'de ACE girişi yoksa, sistem tüm erişim girişimlerini reddeder. DACL'deki ACE'ler, istenen haklara izin veren bir eşleşme bulunana kadar veya erişim reddedilene kadar sırayla kontrol edilir.

### Sistem Erişim Kontrol Listeleri (SACL)
Yöneticilerin güvenli objectlere yapılan erişim denemelerini günlüğe kaydetmesini sağlar. ACE'ler, sistemin güvenlik olay günlüğünde bir kayıt oluşturmasına neden olan erişim girişimlerinin türlerini belirtir.


### Fully Qualified Domain Name (FQDN)
FQDN, belirli bir bilgisayar veya ana bilgisayar için tam addır. Ana bilgisayar adı ve etki alanı adı ile [ana bilgisayar adı].[etki alanı adı].[tld] biçiminde yazılır. Bu, bir objectnin DNS tree hiyerarşisindeki konumunu belirtmek için kullanılır. FQDN, IP adresini bilmeden Active Directory'deki ana bilgisayarları bulmak için kullanılabilir, tıpkı ilişkili IP adresini yazmak yerine google.com gibi bir web sitesine göz atarken olduğu gibi. Örnek olarak INLANEFREIGHT.LOCAL etki alanındaki DC01 ana bilgisayarı verilebilir. Buradaki FQDN DC01.INLANEFREIGHT.LOCAL olacaktır.


### Tombstone
[Tombstone](https://ldapwiki.com/wiki/Wiki.jsp?page=Tombstone), AD'de silinen AD objectlerini tutan bir kapsayıcı objectsidir. Bir object AD'den silindiğinde, object Tombstone Lifetime olarak bilinen belirli bir süre boyunca kalır ve isDeleted özniteliği TRUE olarak ayarlanır. Bir object Tombstone Lifetime süresini aştığında, tamamen kaldırılır. Microsoft, yedeklemelerin kullanışlılığını artırmak için 180 günlük bir mezar taşı ömrü önerir, ancak bu değer ortamlar arasında farklılık gösterebilir. DC işletim sistemi sürümüne bağlı olarak, bu değer varsayılan olarak 60 veya 180 gün olacaktır. AD Recycle Bin'i olmayan bir domain'de bir object silinirse, bu object bir tombstone objectsi haline gelir. Bu durumda, object özniteliklerinin çoğundan arındırılır ve tombstoneLifetime süresi boyunca Silinen objectler kapsayıcısına yerleştirilir. Kurtarılabilir, ancak kaybolan öznitelikler artık kurtarılamaz.


### AD Recycle Bin 
[AD Geri Dönüşüm Kutusu](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/the-ad-recycle-bin-understanding-implementing-best-practices-and/ba-p/396944), silinen AD objectlerinin kurtarılmasını kolaylaştırmak için ilk olarak Windows Server 2008 R2'de kullanıma sunulmuştur. Bu, sistem yöneticilerinin objectleri geri yüklemesini kolaylaştırarak yedeklerden geri yükleme, Active Directory Etki Alanı Hizmetlerini (AD DS) yeniden başlatma veya Domain Controller'ı yeniden başlatma ihtiyacını ortadan kaldırdı. AD Geri Dönüşüm Kutusu etkinleştirildiğinde, silinen objectler belirli bir süre korunur ve gerektiğinde geri yüklemeyi kolaylaştırır. Sistem yöneticileri bir objectnin silinmiş ve kurtarılabilir durumda ne kadar süre kalacağını ayarlayabilir. Bu belirtilmezse, object varsayılan değer olan 60 gün boyunca geri yüklenebilir. AD Geri Dönüşüm Kutusu'nu kullanmanın en büyük avantajı, silinen bir objectnin özniteliklerinin çoğunun korunmasıdır; bu da silinen bir objectyi önceki durumuna tamamen geri yüklemeyi çok daha kolay hale getirir.


### SYSVOL
[SYSVOL](https://social.technet.microsoft.com/wiki/contents/articles/8548.active-directory-sysvol-and-netlogon.aspx) klasörü veya paylaşımı, domain'deki sistem ilkeleri, Group Policy ayarları, oturum açma/kapatma komut dosyaları gibi genel dosyaların kopyalarını saklar ve genellikle AD ortamında çeşitli görevleri gerçekleştirmek için yürütülen diğer komut dosyalarını içerir. SYSVOL klasörünün içeriği, File Replication Services (FRS) kullanılarak ortamdaki tüm DC'lere çoğaltılır. SYSVOL yapısı hakkında daha fazla bilgiyi [buradan](https://networkencyclopedia.com/sysvol-share/#Components-and-Structure) okuyabilirsiniz.


### AdminSDHolder
[AdminSDHolder](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory) objectsi, ayrıcalıklı olarak işaretlenmiş AD'deki built-in grupların üyeleri için ACL'leri yönetmek için kullanılır. Korumalı grupların üyelerine uygulanan Security Descriptor'ı tutan bir kapsayıcı görevi görür. SDProp (SD Propagator) işlemi, PDC Emulator Domain Controller üzerinde bir zamanlama ile çalışır. Bu işlem çalıştığında, korunan grupların üyelerini kontrol ederek onlara doğru ACL'nin uygulandığından emin olur. Varsayılan olarak her saat çalışır. Örneğin, bir saldırganın bir kullanıcıya Domain Admins grubunun bir üyesi üzerinde belirli haklar vermek için kötü amaçlı bir ACL girişi oluşturabildiğini varsayalım. Bu durumda, AD'deki diğer ayarları değiştirmedikleri sürece, SDProp işlemi belirlenen aralıkta çalıştığında bu haklar kaldırılacaktır (ve elde etmeyi umdukları kalıcılığı kaybedeceklerdir).


### dsHeuristics
[dsHeuristics](https://learn.microsoft.com/en-us/windows/win32/adschema/a-dsheuristics) özniteliği, forest çapında birden fazla yapılandırma ayarı tanımlamak için kullanılan Directory Service objectsinde ayarlanan bir string değeridir. Bu ayarlardan biri, built-in grupların [Protected Groups](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory) listesinden çıkarılmasıdır. Bu listedeki gruplar AdminSDHolder objectsi aracılığıyla değiştirilmeye karşı korunur. Bir grup dsHeuristics özniteliği aracılığıyla hariç tutulursa, SDProp işlemi çalıştığında onu etkileyen herhangi bir değişiklik geri alınmaz.


### adminCount
[adminCount](https://learn.microsoft.com/en-us/windows/win32/adschema/a-admincount) özniteliği SDProp işleminin bir kullanıcıyı koruyup korumadığını belirler. Değer 0 olarak ayarlanırsa veya belirtilmezse, kullanıcı korunmaz. Öznitelik değeri value olarak ayarlanırsa, kullanıcı korunur. Saldırganlar genellikle dahili bir ortamda hedef almak için adminCount özniteliği 1 olarak ayarlanmış hesapları ararlar. Bunlar genellikle ayrıcalıklı hesaplardır ve daha fazla erişime veya etki alanının tamamen ele geçirilmesine yol açabilir.

### Active Directory Kullanıcıları ve Bilgisayarları (ADUC)
ADUC, AD'deki kullanıcıları, grupları, bilgisayarları ve kişileri yönetmek için yaygın olarak kullanılan bir GUI konsoludur. ADUC'de yapılan değişiklikler PowerShell aracılığıyla da yapılabilir.

### ADSI Edit
ADSI Edit, AD'deki objectleri yönetmek için kullanılan bir GUI aracıdır. ADUC'de bulunandan çok daha fazlasına erişim sağlar ve bir object üzerinde bulunan herhangi bir özniteliği ayarlamak veya silmek, object eklemek, kaldırmak ve taşımak için kullanılabilir. Kullanıcının AD'ye çok daha derin bir düzeyde erişmesini sağlayan güçlü bir araçtır. Buradaki değişiklikler AD'de büyük sorunlara neden olabileceğinden, bu aracı kullanırken çok dikkatli olunmalıdır.

### sIDHistory
[Bu](https://learn.microsoft.com/en-us/defender-for-identity/security-assessment-unsecure-sid-history-attribute) öznitelik, bir objectye daha önce atanmış olan tüm SID'leri tutar. Genellikle geçişlerde kullanılır, böylece bir kullanıcı bir domain'den diğerine geçtiğinde aynı erişim seviyesini koruyabilir. Bu öznitelik, güvenli olmayan bir şekilde ayarlanırsa potansiyel olarak kötüye kullanılabilir ve SID Filtreleme (veya bir kullanıcının erişim belirtecinden başka bir etki alanından gelen ve yükseltilmiş erişim için kullanılabilecek SID'lerin kaldırılması) etkinleştirilmezse bir saldırganın bir hesabın geçişten önce sahip olduğu yükseltilmiş erişimi elde etmesine olanak tanır.


### NTDS.DIT
NTDS.DIT dosyası Active Directory'nin kalbi olarak kabul edilebilir. Bir Domain Controller üzerinde C:\Windows\NTDS\ adresinde depolanır ve kullanıcı ve grup objectleri, grup üyeliği ve saldırganlar ve sızma testçileri için en önemlisi domain'deki tüm kullanıcıların parola hash'leri gibi AD verilerini depolayan bir veritabanıdır. Domain'in tam olarak ele geçirilmesinden sonra, bir saldırgan bu dosyayı alabilir, hash'leri çıkarabilir ve bunları bir pass-the-hash saldırısı gerçekleştirmek için kullanabilir ya da domain'deki ek kaynaklara erişmek için Hashcat gibi bir araç kullanarak çevrimdışı olarak kırabilir. [Parolayı tersine çevrilebilir şifreleme ile sakla ayarı etkinleştirilmişse](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-10/security/threat-protection/security-policy-settings/store-passwords-using-reversible-encryption), NTDS.DIT bu ilke ayarlandıktan sonra oluşturulan veya parolasını değiştiren tüm kullanıcıların açık metin parolalarını da saklar. Nadiren de olsa, bazı kuruluşlar kimlik doğrulama için kullanıcının mevcut parolasını (Kerberos'u değil) kullanması gereken uygulamalar veya protokoller kullanıyorsa bu ayarı etkinleştirebilir.


### MSBROWSE
MSBROWSE, Windows tabanlı local alan ağlarının (LAN) ilk sürümlerinde tarama hizmetleri sağlamak için kullanılan bir Microsoft ağ protokolüdür. Ağda bulunan paylaşılan yazıcılar ve dosyalar gibi kaynakların bir listesini tutmak ve kullanıcıların bu kaynaklara kolayca göz atmasına ve erişmesine izin vermek için kullanılırdı.

Windows'un eski sürümlerinde Master Browser'ı aramak için nbtstat -A ip-adresini kullanabilirdik. Eğer MSBROWSE görürsek bu Master Browser olduğu anlamına gelir. Ayrıca nltest yardımcı programını kullanarak bir Windows Master Browser'ı Domain Controller'ların isimleri için sorgulayabilirdik.

Günümüzde MSBROWSE büyük ölçüde eskimiştir ve artık yaygın olarak kullanılmamaktadır. Modern Windows tabanlı LAN'lar dosya ve yazıcı paylaşımı için Server Message Block (SMB) protokolünü ve tarama hizmetleri için Common Internet File System (CIFS) protokolünü kullanmaktadır.


Bir Active Directory ortamının “Blueprint ”i olarak ne bilinir? (==Schema==)

Bir Service örneğini benzersiz olarak tanımlayan nedir? (tam ad, boşluk bırakılmış, kısaltılmamış) (==Service Principal Name==)

Doğru veya Yanlış; Group Policy objectleri kullanıcı ve bilgisayar objectlerine uygulanabilir. (==Evet==)

AD'de silinen objectleri hangi kapsayıcı tutar? (==Tombstone==)

Bir domain'deki tüm kullanıcıların parolalarının hash'lerini içeren dosya hangisidir? (==NTDS.DIT==)



#### Active Directory Objects
AD'den bahsederken sık sık “ objeler” terimini göreceğiz. Obje nedir? object, Active Directory ortamında bulunan OU'lar, yazıcılar, kullanıcılar, domain denetleyicileri gibi HERHANGİ bir kaynak olarak tanımlanabilir.
![Pasted image 20240927135340.png](/img/user/resimler/Pasted%20image%2020240927135340.png)

### Users
Bunlar kuruluşun AD ortamındaki kullanıcılardır. Kullanıcılar yaprak (leaf) objectler olarak kabul edilir; bu da içlerinde başka objectler barındıramayacakları anlamına gelir. Yaprak objectye bir başka örnek de Microsoft Exchange'deki posta kutusudur. Bir kullanıcı objectsi bir güvenlik sorumlusu olarak kabul edilir ve bir güvenlik tanımlayıcısına (SID) ve bir genel benzersiz tanımlayıcıya (GUID) sahiptir. Kullanıcı [objectleri](http://www.kouti.com/tables/userattributes.htm), görünen adları, son oturum açma zamanı, son parola değiştirme tarihi, e-posta adresi, hesap açıklaması, yönetici, adres ve daha fazlası gibi birçok olası özniteliğe sahiptir. Belirli bir Active Directory ortamının nasıl kurulduğuna bağlı olarak, burada ayrıntılı olarak [açıklandığı](https://www.easy365manager.com/how-to-get-all-active-directory-user-object-attributes/) gibi TÜM olası öznitelikler hesaba katıldığında 800'den fazla olası kullanıcı özniteliği olabilir. Bu örnek, çoğu ortamda standart bir kullanıcı için tipik olarak doldurulanların çok ötesine geçer, ancak Active Directory'nin büyüklüğünü ve karmaşıklığını gösterir. Düşük ayrıcalıklı bir kullanıcıya erişim elde etmek bile birçok objectye ve kaynağa erişim sağlayabileceğinden ve tüm domainin (veya forest'in) ayrıntılı bir şekilde numaralandırılmasına izin verebileceğinden, saldırganlar için çok önemli bir hedeftir.


...




# Kerberos, DNS, LDAP, MSRPC
Windows işletim sistemleri iletişim kurmak için çeşitli protokoller kullanırken, Active Directory özellikle [Lightweight Directory Access Protocol](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) (LDAP), Microsoft'un [Kerberos](https://en.wikipedia.org/wiki/Kerberos_(protocol)) sürümü, kimlik doğrulama ve iletişim için [DNS](https://en.wikipedia.org/wiki/Domain_Name_System) ve istemci-sunucu modeli tabanlı uygulamalar için kullanılan bir işlemler arası iletişim tekniği olan [Remote Procedure Call'un](https://en.wikipedia.org/wiki/Remote_procedure_call) (RPC) Microsoft uygulaması olan MSRPC gerektirir.

- ==LDAP (Lightweight Directory Access Protocol):== Active Directory, dizin servislerine erişim ve yönetim için bu protokolü kullanır.

- ==Kerberos:== Microsoft'un Active Directory'ye özel uyarladığı bu protokol, kimlik doğrulama işlemlerini güvenli ve hızlı bir şekilde gerçekleştirir.

- ==DNS (Domain Name System):== Active Directory, domain controllerın bulunması ve name resolution işlemleri için DNS protokolüne dayanır. 

	* NetBIOS, WINS (Windows Internet Name Service), LLMNR (Link-Local Multicast Name Resolution), mDNS (Multicast DNS) gibi protokoller, DNS'in olmadığı veya tercih edilmediği durumlarda name resolution işlemleri için kullanılır.
	
- ==MSRPC (Microsoft Remote Procedure Call)==: Client-Server tabanlı uygulamalar arasında processler arası iletişim sağlayarak kaynaklara erişim ve yönetim süreçlerini kolaylaştırır.

## Kerberos
Kerberos, Windows 2000'den beri domain hesapları için varsayılan ==kimlik doğrulama== protokolüdür. Kerberos açık bir standarttır ve aynı standardı kullanan diğer sistemlerle birlikte çalışabilirlik sağlar. Bir kullanıcı bilgisayarında oturum açtığında, Kerberos karşılıklı kimlik doğrulama yoluyla kimliklerini doğrulamak için kullanılır veya hem kullanıcı hem de server kimliklerini doğrular. Kerberos, kullanıcı parolalarını ağ üzerinden iletmek yerine ==biletlere (ticket)== dayanan ==stalles== bir kimlik doğrulama protokolüdür. Active Directory Domain Services'in (AD DS) bir parçası olarak, Domain Controller'larda bilet veren bir ==Kerberos Key Distribution Center (KDC)== bulunur. Bir kullanıcı bir sisteme oturum açma isteği başlattığında, kimlik doğrulaması için kullandığı client KDC'den bir bilet ister ve isteği kullanıcının parolasıyla şifreler. KDC kullanıcının parolasını kullanarak isteğin (AS-REQ) şifresini çözebilirse, bir Ticket Granting Ticket (TGT) oluşturur ve bunu kullanıcıya iletir. Kullanıcı daha sonra, ilişkili servisin NTLM parola hash'i ile şifrelenmiş bir Ticket Granting Service (TGS) bileti talep etmek için TGT'sini bir Domain Controller'a sunar. Son olarak, istemci TGS'yi şifre hash'iyle şifresini çözen uygulama veya service sunarak gerekli servise erişim talebinde bulunur. Tüm proses uygun şekilde tamamlanırsa, kullanıcının istenen hizmete veya uygulamaya erişmesine izin verilir.

Kerberos kimlik doğrulaması, kullanıcıların kimlik bilgilerini tüketilebilir kaynaklara yaptıkları taleplerden etkili bir şekilde ayırarak parolalarının ağ üzerinden iletilmemesini sağlar (örneğin, internal bir SharePoint intranet sitesine erişim). Kerberos Key Distribution Centre (KDC) önceki işlemleri kaydetmez. Bunun yerine, Kerberos Ticket Granting Service bileti (TGS) geçerli bir Ticket Granting Ticket'a (TGT) dayanır. Kullanıcı geçerli bir TGT'ye sahipse, kimliğini kanıtlamış olması gerektiğini varsayar. Aşağıdaki şemada bu süreç yüksek seviyede anlatılmaktadır.


---

#### Genel Özet 

##### 1. **Kerberos Nedir?**

- **Kerberos**, Windows 2000'den beri **domain hesapları** için varsayılan **kimlik doğrulama** protokolüdür.
- **Açık bir standart**tır ve aynı standardı kullanan diğer sistemlerle uyumluluk sağlar.

##### 2. **Kerberos'un Temel Özellikleri**

- **Karşılıklı kimlik doğrulama:**  
    Kullanıcı bilgisayarında oturum açtığında, Kerberos hem **kullanıcı** hem de **server** kimliklerini doğrular.
- **Ticket Tabanlı Kimlik Doğrulama:**  
    Kerberos, **parolaları** ağ üzerinden iletmek yerine ==ticket== dayalı **stateless** bir kimlik doğrulama protokolüdür.
- **Ağ Üzerinde Parola İletimi Yok:**  
    Kerberos, kullanıcıların **kimlik bilgilerini** erişim sağladıkları kaynaklardan etkili bir şekilde ayırarak **parolalarının ağ üzerinden iletilmesini engeller** (örneğin, internal bir SharePoint intranet sitesine erişim).



##### 3. **Kerberos'un Çalışma Mekanizması**

###### A. **Kerberos Key Distribution Center (KDC):**

- **Domain Controller**’larda bulunan **KDC**, ticket veren merkezi bir bileşendir.
- Kullanıcı, kimlik doğrulaması için KDC'den **Ticket Granting Ticket (TGT)** alır.
- **KDC, önceki işlemleri kaydetmez**; her işlem biletler üzerinden doğrulanır.

###### B. **Kullanıcı Oturum Açma Süreci:**

1. **Authentication Service Request (AS-REQ):**
    
    - Kullanıcı, bir sisteme oturum açmak için **client**'ından **KDC**'ye bir kimlik doğrulama isteği gönderir.
    - Bu istek **kullanıcının parolasıyla şifrelenir.**
    
2. **Ticket Granting Ticket (TGT):**
    
    - KDC, kullanıcının parolasını kullanarak isteğin **şifresini çözer**.
    - Eğer şifre çözülürse, **Ticket Granting Ticket (TGT)** oluşturulur ve kullanıcıya iletilir.

3. **Ticket Granting Service Request (TGS-REQ):**
    
    - Kullanıcı, TGT’sini sunarak **TGS bileti** talep eder.
    - TGT, **NTLM parola hash’i** ile şifrelenmiş olarak iletilir.

4. **Service'e Erişim:**
    
    - Kullanıcı, TGS biletini hedef **servis** veya uygulamaya sunar.
    - Server bu ticket'ın şifresini çözer ve kullanıcının kimliğini doğrular.
    - Kimlik doğrulama başarılıysa, kullanıcı istenen service'e erişim sağlar.



##### 4. **Bilet Sistemi Kullanımı**

- **Güvenlik:** Parolalar ağ üzerinde iletilmez.
- **Stateless:** Kerberos, **stateless** bir kimlik doğrulama sistemidir.
- **Hız:** Biletlerin kullanılması, parolaların her seferinde doğrulanmasını engeller.



##### 5. **Kerberos'un Avantajları**

- **Güvenlik:** Şifrelerin ağ üzerinde taşınmaması, kimlik avı gibi saldırılara karşı koruma sağlar.
- **Performans:** Ticket tabanlı işlem, her seferinde parola doğrulama yapılmadığı için daha hızlıdır.
- **Uyumluluk:** Farklı sistemlerle birlikte çalışabilir, esneklik sağlar.



##### 6. **Ekstra Bilgiler: Kerberos'un Kimlik Doğrulama Süreci**

- **TGS ve TGT'nin Rolü:**  
    **Kerberos Key Distribution Centre (KDC)** önceki işlemleri kaydetmez. Bunun yerine, **Kerberos Ticket Granting Service (TGS)** bileti, geçerli bir **Ticket Granting Ticket (TGT)**’ye dayanır.
- **Geçerli TGT:**  
    Kullanıcı, geçerli bir TGT'ye sahipse, kimliğini kanıtlamış olması gerektiği varsayılır.

--- 

### Kerberos Authentication Process

1. Kullanıcı oturum açar ve parolası, TGT biletini şifrelemek için kullanılan bir NTLM hash'ine dönüştürülür. Bu, kullanıcının kimlik bilgilerini kaynaklara yapılan isteklerden ayırır.

	* **Kullanıcı Girişi**: Kullanıcı `password123` gibi bir parola girer. 
	* **NTLM Hash'ine Dönüşüm**: Parola, NTLM algoritmasıyla şifrelenir ve bir hash değeri oluşturulur.
	* Örnek: `password123` parolasının NTLM hash'ine dönüşmesi sonucu bir dizi sayı ve harf (hash değeri) oluşur, örneğin: `8846f7eaee8fb117ad06bdd830b7586c`.
	* NTLM hash'i, TGT (Ticket Granting Ticket) biletini şifrelemek için kullanılır. Bu aşamada, sistem kaynaklarına erişim sağlamak amacıyla kullanıcı kimlik bilgileri şifrelenmiş olur.

2. DC'deki KDC service'i kimlik doğrulama servisi talebini (AS-REQ) kontrol eder, kullanıcı bilgilerini doğrular ve kullanıcıya teslim edilen bir Ticket Granting Ticket (TGT) oluşturur.

3. Kullanıcı TGT'yi DC'ye sunarak belirli bir sevice için Ticket Granting Service (TGS) ticket'ı talep eder. Bu TGS-REQ'dir. TGT başarıyla doğrulanırsa, verileri bir TGS bileti oluşturmak için kopyalanır.

4. TGS, context'inde service  örneğinin çalıştığı servisin veya bilgisayar hesabının NTLM parola hash'i ile şifrelenir ve TGS_REP içinde kullanıcıya teslim edilir.

	* **Kullanıcı parolası (password123)**, önce NTLM hash'ine dönüştürülüp, TGT şifrelenmesinde kullanılıyor.
	- **TGS**, servisin NTLM hash'i ile şifreleniyor ve kullanıcıya teslim ediliyor. 

5. Kullanıcı TGS'yi service'e sunar ve geçerli olması durumunda kullanıcının kaynağa bağlanmasına izin verilir (AP_REQ).


![Pasted image 20240927153822.png](/img/user/resimler/Pasted%20image%2020240927153822.png)


- Kullanıcı şifresi, **NTLM hash**'ine dönüştürülür ve bu, kimlik doğrulama bileti (TGT) isteğini şifreler.
    
- **DC (KDC)**, kimlik doğrulama servisi isteğini (**AS_REQ**) kontrol eder, kullanıcı bilgilerini doğrular ve **Ticket-Granting Ticket (TGT)** oluşturur. **TGT**, kullanıcıya teslim edilir (**AS_REP**).
    
- Kullanıcı, belirli bir service'i örneği için **Ticket Granting Service (TGS)** bileti talep ettiğinde (**TGS_REQ**), **TGT**'yi **DC**'ye sunar. **TGT**, **DC** doğrulamasını geçerse, verileri kopyalanarak bir **TGS bileti** oluşturulur.
    
- **TGS**, servisin çalıştığı servis/bilgisayar hesabının **NTLM şifre** hash'i ile şifrelenir ve kullanıcıya teslim edilir (**TGS_REP**).
    
- Kullanıcı, **TGS**'yi servisi sunar ve geçerli ise, servis örneğine bağlanabilir               (**AP_REQ**).


---
- **KRB_AS_REQ (Authentication Service Request)**:
    
    - **Açıklama**: Kullanıcı, kimlik doğrulama için **KDC'ye** (Key Distribution Center) bir istek gönderir. Bu istekte kullanıcının kimlik bilgileri yer alır.
    - **Amaç**: Kullanıcı, kimlik doğrulama sürecine başlamak için **TGT** (Ticket Granting Ticket) almak amacıyla KDC'ye başvurur.

- **KRB_AS_REP (Authentication Service Reply)**:
    
    - **Açıklama**: KDC, gelen **KRB_AS_REQ** isteğini doğrular ve eğer kimlik doğrulama başarılıysa, bir **TGT** ve bir **şifreli mesaj** ile kullanıcıya cevap verir.
    - **Amaç**: KDC, kullanıcının kimliğini doğruladıktan sonra ona güvenli bir **TGT** verir, böylece kullanıcı diğer hizmetlere erişim için bu bileti kullanabilir.

- **KRB_TGS_REQ (Ticket Granting Service Request)**:
    
    - **Açıklama**: Kullanıcı, sahip olduğu **TGT'yi** kullanarak belirli bir servis için **TGS** (Ticket Granting Service) talep eder.
    - **Amaç**: Kullanıcı, belirli bir servise erişmek için gerekli olan **TGS** biletini almak için **TGT** ile başvurur.

- **KRB_TGS_REP (Ticket Granting Service Reply)**:
    
    - **Açıklama**: KDC, kullanıcının **TGT'sini** doğrular ve başarılı bir doğrulama sonrası, **TGS** biletini kullanıcıya gönderir. Bu bilet, kullanıcıyı belirli bir servise bağlamak için kullanılacaktır.
    - **Amaç**: Kullanıcıya, erişmek istediği servise bağlanabilmesi için gerekli olan **TGS** bileti verilir.

---


Kerberos protokolü 88 numaralı portu kullanır (hem TCP hem de UDP). Bir Active Directory ortamını numaralandırırken, ==Nmap gibi bir araç kullanarak açık port 88'i arayan port taramaları gerçekleştirerek Domain Controllers'ın yerini sıklıkla tespit edebiliriz.==


## DNS
Active Directory Domain Services (AD DS), client'ların ( workstation'lar, server'lar ve domain ile iletişim kuran diğer sistemler) Domain Controllers'ı bulmalarını ve dizin hizmetini barındıran Domain Controllers'ın kendi aralarında iletişim kurmalarını sağlamak için DNS kullanır. DNS, host adlarını IP adreslerine çözümlemek için kullanılır ve iç ağlar ve internet genelinde yaygın olarak kullanılır. Private iç ağlar, serverlar, clientlar ve peerlar arasındaki iletişimi kolaylaştırmak için Active Directory DNS namespaces kullanır. AD, ağ üzerinde servis record'ları (SRV) şeklinde çalışan servislerin bir veritabanını tutar. Bu servis record'ları AD ortamındaki Client'ların dosya sunucusu, yazıcı veya Domain Controller gibi ihtiyaç duydukları servisleri bulmalarını sağlar. Dinamik DNS, bir sistemin IP adresinin değişmesi durumunda DNS veritabanında otomatik olarak değişiklik yapmak için kullanılır. Bu girişleri manuel olarak yapmak çok zaman alır ve hataya yer bırakır. DNS veritabanında bir host için doğru IP adresi yoksa, client'lar bu host'u ağ üzerinde bulamaz ve onunla iletişim kuramazlar. Bir client ağa katıldığında, DNS servisine bir sorgu göndererek, DNS veritabanından bir SRV record alarak ve Domain Controller'ın hostname'ini client'a ileterek Domain Controller'ı bulur. Client daha sonra Domain Controller'ın IP adresini almak için bu hostname'i kullanır. DNS, TCP ve UDP port 53'ü kullanır. UDP port 53 varsayılandır, ancak artık iletişim kurulamadığında ve DNS mesajları 512 bayttan büyük olduğunda TCP'ye geri döner.

##### Özet : 

- Active Directory Domain Services (AD DS), client'ların Domain Controllers'ı bulmasını ve Domain Controllers'ın kendi aralarında iletişim kurmasını sağlamak için DNS kullanır.

- DNS, host adlarını IP adreslerine çözümlemek için kullanılır.

- Active Directory DNS namespaces, private iç ağlar, server'lar, client'lar ve peer'lar arasındaki iletişimi kolaylaştırır.

- AD, servislerin bir veritabanını tutar ve bu servisler SRV record'ları şeklinde çalışır.

- Client'lar, AD ortamındaki ihtiyaç duydukları servisleri (dosya sunucusu, yazıcı, Domain Controller vb.) SRV record'ları ile bulurlar.

- Dinamik DNS, bir sistemin IP adresi değiştiğinde DNS veritabanını otomatik olarak günceller.

- Manuel DNS güncellemeleri zaman alıcı ve hata yapmaya açıktır.

- Bir client ağa katıldığında, DNS servisine bir sorgu gönderir ve DNS veritabanından SRV record alır.

- SRV record, Domain Controller'ın hostname'ini içerir ve client bu hostname'i kullanarak Domain Controller'ın IP adresini alır.

- DNS, TCP ve UDP port 53'ü kullanır.

- UDP port 53 varsayılandır, ancak iletişim kurulamadığında veya DNS mesajları 512 bayttan büyük olduğunda TCP'ye geçilir.



![Pasted image 20240927213721.png](/img/user/resimler/Pasted%20image%2020240927213721.png)


### Forward DNS Lookup ( Yönlendirmeli DNS Arama)
Bir örneğe bakalım. Domain adı için bir nslookup gerçekleştirebilir ve bir domain'deki tüm Domain Controller'ların IP adreslerini alabiliriz.
![Pasted image 20241001152006.png](/img/user/resimler/Pasted%20image%2020241001152006.png)


### Reverse DNS Lookup
IP adresini kullanarak tek bir hostun DNS adını elde etmek istersek, bunu aşağıdaki şekilde yapabiliriz:

![Pasted image 20241001152049.png](/img/user/resimler/Pasted%20image%2020241001152049.png)


### Bir Hostun IP Adresini Bulma
Tek bir hostun IP adresini bulmak istersek, bunu tersten yapabiliriz. Bunu FQDN belirterek veya belirtmeden yapabiliriz.
![Pasted image 20241001152629.png](/img/user/resimler/Pasted%20image%2020241001152629.png)


## LDAP
Active Directory, dizin aramaları için[ Lightweight Directory Access Protocol](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol)'ü (LDAP) destekler. LDAP, çeşitli dizin servislerine (AD gibi) yönelik kimlik doğrulama için kullanılan açık kaynaklı ve platformlar arası bir protokoldür. En son LDAP spesifikasyonu RFC 4511 olarak yayınlanan [Versiyon 3](https://tools.ietf.org/html/rfc4511)'tür. LDAP'ın AD ortamında nasıl çalıştığını anlamak saldırganlar ve savunmacılar için çok önemlidir. LDAP 389 numaralı portu kullanır ve SSL üzerinden LDAP (LDAPS) 636 numaralı port üzerinden iletişim kurar.

AD, kullanıcı hesap bilgilerini ve parolalar gibi güvenlik bilgilerini depolar ve bu bilgilerin ağ üzerindeki diğer cihazlarla paylaşılmasını kolaylaştırır. LDAP, uygulamaların dizin servislerini sağlayan diğer sunucularla iletişim kurmak için kullandığı dildir. Başka bir deyişle, LDAP ağ ortamındaki sistemlerin AD ile “==konuşabilmesini==” sağlar.

Bir LDAP oturumu ilk olarak Directory System Agent olarak da bilinen bir LDAP sunucusuna bağlanarak başlar. AD'deki Domain Controller, güvenlik kimlik doğrulama istekleri gibi LDAP isteklerini etkin bir şekilde dinler.

![Pasted image 20241001155009.png](/img/user/resimler/Pasted%20image%2020241001155009.png)
- **Client Application** , kullanıcı bilgileri için bir request gönderir.
- Request ==**API Gateway**== üzerinden geçer.
- API Gateway, **LDAP** sorguları aracılığıyla **Active Directory**'ye bu bilgileri iletir.
- **Active Directory** kullanıcı bilgilerini bulur ve yanıt olarak **API Gateway**'e geri gönderir.
- API Gateway, kullanıcı bilgilerini tekrar **Client Application**'a ileterek isteği tamamlar.

AD ve LDAP arasındaki ilişki Apache ve HTTP ile karşılaştırılabilir. Apache'nin HTTP protokolünü kullanan bir web sunucusu olması gibi, Active Directory de LDAP protokolünü kullanan bir dizin sunucusudur.

Nadiren de olsa, bir değerlendirme yaparken AD'ye sahip olmayan ancak LDAP kullanan kuruluşlarla karşılaşabilirsiniz, bu da büyük olasılıkla [OpenLDAP](https://en.wikipedia.org/wiki/OpenLDAP) gibi başka bir LDAP sunucusu kullandıkları anlamını taşır.


### AD LDAP Authentication (Kimlik Doğrulama)
LDAP, bir LDAP oturumu için kimlik doğrulama durumunu ayarlamak üzere bir “BIND” işlemi kullanarak kimlik bilgilerini AD'ye karşı doğrulamak üzere ayarlanmıştır. İki tür LDAP kimlik doğrulaması vardır. 

**BIND**, LDAP client'inin sunucuya bağlanıp kimlik doğrulaması yapması için kullanılan bir mekanizmadır.

* ==Simple Authentication==: Bu, anonim kimlik doğrulama, kimliği doğrulanmamış kimlik doğrulama ve kullanıcı adı/parola kimlik doğrulamasını içerir. Basit kimlik doğrulama, bir kullanıcı adı ve parolanın LDAP sunucusuna kimlik doğrulamak için bir BIND isteği oluşturması anlamına gelir.

* ==SASL Authentication==: [Basit Kimlik Doğrulama ve Security Layer](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) (SASL) framework'ü LDAP sunucusuna bağlanmak için Kerberos gibi diğer kimlik doğrulama servislerini kullanır ve ardından LDAP'ye kimlik doğrulaması yapmak için bu kimlik doğrulama servisini (bu örnekte Kerberos) kullanır. LDAP sunucusu, yetkilendirme servisinde bir LDAP mesajı göndermek için LDAP protokolünü kullanır ve bu da başarılı ya da başarısız kimlik doğrulamasıyla sonuçlanan bir dizi challenge/response mesajı başlatır. SASL, kimlik doğrulama yöntemlerinin uygulama protokollerinden ayrılması nedeniyle ek güvenlik sağlayabilir.

LDAP kimlik doğrulama mesajları varsayılan olarak açık metin olarak gönderilir, böylece herhangi biri iç ağdaki LDAP mesajlarını koklayabilir. Bu bilgileri aktarım sırasında korumak için TLS şifrelemesi veya benzerinin kullanılması önerilir.

---
LDAP'taki **BIND**, bir LDAP oturumunda kimlik doğrulama durumunu belirlemek için kullanılan bir işlemdir. Bu işlem, client ile server arasında bir oturum başlatır ve client'in kimliğini doğrulamak için kullanılır. Başka bir deyişle, BIND, LDAP üzerinden erişim sağlamak için kimlik bilgilerini doğrulama sürecidir.

### BIND İşleminde Ne Olur?

1. **Bağlantı Kurulması**: client, LDAP sunucusuna bir bağlantı kurar.
2. **Kimlik Doğrulama Gönderimi**: Client, kimlik bilgilerini (kullanıcı adı ve şifre gibi) BIND işlemi aracılığıyla LDAP sunucusuna gönderir.
3. **Kimlik Doğrulama Sonucu**: Sunucu, kimlik bilgilerini doğrular ve başarılı/başarısız sonucunu döner.

### LDAP Kimlik Doğrulama Türleri

1. **Simple Bind**:
    
    - Düz metin olarak kullanıcı adı ve şifre gönderir.
    - Güvensizdir ve yalnızca şifreleme (TLS/SSL) kullanılarak güvenli hale getirilebilir.
    
2. **SASL Bind** (Simple Authentication and Security Layer):
    
    - Gelişmiş bir kimlik doğrulama mekanizmasıdır.
    - Örneğin, Kerberos gibi protokollerle entegre çalışabilir.

### Özetle

- **BIND**, LDAP client'in server'a bağlanıp kimlik doğrulaması yapması için kullanılan bir mekanizmadır.
- Güvenlik sağlamak için genellikle TLS veya SSL ile kullanılır.
- Eğer kimlik doğrulama başarılı olursa, client LDAP dizininde sorgulamalar ve değişiklikler yapabilir.

---


### MSRPC
Yukarıda belirtildiği gibi MSRPC, Microsoft'un client-server modeli tabanlı uygulamalar için kullanılan bir prosesler arası iletişim tekniği olan Remote Procedure Call (RPC) uygulamasıdır. Windows sistemleri, dört temel RPC interface'ini kullanarak Active Directory'deki sistemlere erişmek için MSRPC'yi kullanır.

==lsarpc== : Bir bilgisayardaki lokal security politikasını yöneten, denetim politikasını kontrol eden ve etkileşimli kimlik doğrulama servisleri sağlayan [Local Security Authority](https://networkencyclopedia.com/local-security-authority-lsa/) (LSA) sistemine yönelik bir dizi RPC çağrısı. LSARPC, domain güvenlik politikaları üzerinde yönetim gerçekleştirmek için kullanılır. 

* [[Bağlantılar/lsarpc hakkında daha fazla bilgi\|lsarpc hakkında daha fazla bilgi]]

==netlogon== : Netlogon, domain ortamındaki kullanıcıların ve diğer servislerin kimliklerini doğrulamak için kullanılan bir Windows prosesidir. Arka planda sürekli çalışan bir servisdir.

* [[Bağlantılar/netlogon hakkında daha fazla bilgi\|netlogon hakkında daha fazla bilgi]]

==samr== : Remote SAM (samr), user'lar ve gruplar hakkında bilgi depolayan domain hesap veritabanı için yönetim fonksiyonelliği sağlar. BT yöneticileri, yöneticilerin security policy'leri hakkında bilgi oluşturmasını, okumasını, güncellemesini ve silmesini sağlayarak kullanıcıları, grupları ve bilgisayarları yönetmek için bu protokolü kullanır. Saldırganlar (ve pentesterlar) samr protokolünü kullanarak [BloodHound](https://github.com/BloodHoundAD/) gibi araçlarla iç domain hakkında keşif yapabilir, AD ağının görsel haritasını çıkarabilir ve “saldırı yolları” oluşturarak administrative erişiminin ya da tüm domainin ele geçirilmesinin nasıl başarılabileceğini görsel olarak gösterebilirler. Kuruluşlar, varsayılan olarak kimliği doğrulanmış tüm domain kullanıcıları AD domain'i hakkında önemli miktarda bilgi toplamak için bu sorguları yapabileceğinden, bir Windows registry key'ini yalnızca yöneticilerin remote SAM sorguları yapmasına izin verecek şekilde değiştirerek bu tür keşiflere karşı [koruma](Organizations can [protect](https://stealthbits.com/blog/making-internal-reconnaissance-harder-using-netcease-and-samri1o/) against this type of reconnaissance by changing a Windows registry key to only allow a) sağlayabilir.

* [[samr hakkında daha fazla bilgi \|samr hakkında daha fazla bilgi ]]

==drsuapi== : drsuapi, multi-DC ortamında Domain Controller'lar arasında çoğaltma ile ilgili görevleri gerçekleştirmek için kullanılan Directory Replication Service (DRS) Remote Protocol'ü uygulayan Microsoft API'sidir. Saldırganlar drsuapi'yi kullanarak Active Directory domain veritabanı (NTDS.dit) [dosyasının bir kopyasını oluşturabilir](https://attack.mitre.org/techniques/T1003/003/) ve domain'deki tüm hesaplar için parola hash'lerini alabilir; bu hash'ler daha sonra daha fazla sisteme erişmek için Pass-the-Hash saldırıları gerçekleştirmek için kullanılabilir veya Remote Desktop (RDP) ve WinRM gibi remote  yönetim protokollerini kullanarak sistemlerde oturum açmak için açık metin parolasını elde etmek üzere Hashcat gibi bir araç kullanarak çevrimdışı olarak kırılabilir.

* [[drsuapi hakkında daha fazla bilgi \|drsuapi hakkında daha fazla bilgi ]]

Soru : Kerberos hangi ağ portunu kullanıyor? (88)

Soru : Adları IP adreslerine çevirmek için hangi protokol kullanılır? (kısaltma) (DNS)

Soru : RFC 4511 hangi protokolü belirtir? (kısaltma)  (LDAP)


### NTLM Authentication
Kerberos ve LDAP'ın yanı sıra Active Directory, AD'deki uygulamalar ve servisler tarafından kullanılabilen (ve kötüye kullanılabilen) birkaç başka kimlik doğrulama yöntemi kullanır. Bunlar LM, NTLM, NTLMv1 ve NTLMv2'yi içerir. Buradaki LM ve NTLM hash adlarıdır ve NTLMv1 ve NTLMv2, LM veya NT hash'ini kullanan kimlik doğrulama protokolleridir. Aşağıda bu hash'ler ve protokoller arasında hızlı bir karşılaştırma yer almaktadır; bu da bize hiçbir şekilde mükemmel olmasa da Kerberos'un mümkün olan her yerde tercih edilen kimlik doğrulama protokolü olduğunu göstermektedir. Hash türleri ve bunları kullanan protokoller arasındaki farkı anlamak çok önemlidir.


### Hash Protokolü Karşılaştırması
![Pasted image 20241001184942.png](/img/user/resimler/Pasted%20image%2020241001184942.png)


### LM
LAN Manager (LM veya LANMAN) hash'leri Windows işletim sistemi tarafından kullanılan en eski parola depolama mekanizmasıdır. LM 1987 yılında OS/2 işletim sisteminde kullanılmaya başlanmıştır. Eğer kullanılıyorsa, bir Windows host üzerindeki SAM veritabanında ve bir Domain Controller üzerindeki ==NTDS.DIT== veritabanında saklanırlar. LM hash'leri için kullanılan hash algoritmasındaki önemli güvenlik zayıflıkları nedeniyle, Windows Vista/Server 2008'den bu yana varsayılan olarak kapatılmıştır. Ancak, özellikle eski sistemlerin hala kullanıldığı büyük ortamlarda hala yaygın olarak karşılaşılmaktadır. LM kullanan parolalar en fazla 14 karakterle sınırlıdır. Parolalar büyük/küçük harfe duyarlı değildir ve hash değer oluşturulmadan önce büyük harfe dönüştürülür, bu da anahtar alanını toplam 69 karakterle sınırlayarak Hashcat gibi bir araç kullanarak bu hashleri kırmayı nispeten kolaylaştırır.

Hashing işleminden önce, 14 karakterlik bir parola ilk olarak iki adet yedi karakterlik parçaya bölünür. Parola on dört karakterden azsa, doğru değere ulaşmak için NULL karakterlerle doldurulacaktır. Her parçadan iki DES anahtarı oluşturulur. Bu parçalar daha sonra ==KGS!@#$%== stringi kullanılarak şifrelenir ve iki adet 8 baytlık şifreli metin değeri oluşturulur. Bu iki değer daha sonra birleştirilerek bir LM hash'i elde edilir. Bu hash algoritması, bir saldırganın on dört karakterin tamamı yerine yalnızca yedi karakteri iki kez brute force yapması gerektiği anlamına gelir, bu da bir veya daha fazla GPU'lu bir sistemde LM hash'lerini kırmayı hızlı hale getirir. Bir parola yedi karakter veya daha az ise, LM hash'inin ikinci yarısı her zaman aynı değerde olacaktır ve hatta Hashcat gibi araçlara bile gerek kalmadan görsel olarak belirlenebilir. LM hash'lerinin kullanımına [Group Policy ](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-do-not-store-lan-manager-hash-value-on-next-password-change)kullanılarak izin verilmeyebilir. Bir LM hash'i 299bd128c1101fd6 biçimini alır.

Not: Windows Vista ve Windows Server 2008'den önceki Windows işletim sistemleri (Windows NT4, Windows 2000, Windows 2003, Windows XP) varsayılan olarak bir kullanıcının parolasının hem LM hash'ini hem de NTLM hash'ini depolar.


### NTHash (NTLM)
==NT LAN Manager (NTLM)== hash'leri modern Windows sistemlerinde kullanılır. Bu bir challenge-response kimlik doğrulama protokolüdür ve kimlik doğrulamak için üç mesaj kullanır: bir client ilk olarak sunucuya bir ==NEGOTIATE_MESSAGE== gönderir, sunucunun cevabı ise client'ın kimliğini doğrulamak için bir ==CHALLENGE_MESSAGE=='dır. Son olarak, client bir ==AUTHENTICATE_MESSAGE== ile yanıt verir. Bu hash'ler local olarak SAM veritabanında veya bir Domain Controller üzerindeki NTDS.DIT veritabanı dosyasında saklanır. Protokolün kimlik doğrulama gerçekleştirmek için seçebileceği iki hashlenmiş parola değeri vardır: LM hash (yukarıda tartışıldığı gibi) ve parolanın little-endian UTF-16 değerinin MD4 hash'i olan NT hash. Algoritma şu şekilde görselleştirilebilir: ==MD4(UTF-16-LE(password)==).

---

###### ÖZET 

1. **NTLM Kimlik Doğrulama Protokolü**:
    
    - NTLM, modern Windows sistemlerinde kullanılan bir **challenge-response** (meydan okuma-cevap) kimlik doğrulama protokolüdür.
    - Kimlik doğrulama sürecinde üç mesaj kullanılır:
        - **NEGOTIATE_MESSAGE**: İlk olarak, client sunucuya bu mesajı gönderir.
        - **CHALLENGE_MESSAGE**: Sunucu, client'ın kimliğini doğrulamak için bu mesajı gönderir.
        - **AUTHENTICATE_MESSAGE**: Son olarak, client bu mesajla yanıt verir.

2. **NTLM Hash'lerinin Saklanması**:
    
    - **LM Hash** ve **NT Hash** olmak üzere iki tür hash değeri vardır.
        - Bu hash'ler, **SAM (Security Account Manager)** veritabanı veya **NTDS.DIT** (Domain Controller üzerindeki veritabanı dosyasında) saklanır.

3. **Hash Türleri**:
    
    - **LM Hash**: Eski bir hash türüdür, ancak genellikle zayıf bir güvenlik sağlar ve modern sistemlerde nadiren kullanılır.
    - **NT Hash**: Kullanıcının parolasının **little-endian UTF-16** formatında şifrelenmiş haliyle, **MD4** algoritmasıyla oluşturulan hash değeridir. Bu değer daha güvenlidir.

4. **NT Hash Algoritması**:
    
    - **NT Hash** algoritması şu şekilde çalışır:
        - **UTF-16-LE** (little-endian) formatında şifrelenmiş parola alınır.
        - Ardından bu şifreli parola, **MD4** algoritması ile hashlenir.
        - Sonuçta elde edilen değer, **NT Hash** olarak saklanır.

5. **Kimlik Doğrulama Adımları**:
    
    - Kullanıcı, bu hash'leri kullanarak kimlik doğrulama işlemine girer.
    - **NTLM** kimlik doğrulaması, bu hash'leri kullanarak, password doğrulaması yapılmadan önce **challenge-response** mesajları aracılığıyla kimlik doğrulaması sağlar.

Bu protokol, şifreler üzerine yapılabilecek saldırılara karşı daha güvenli olmakla birlikte, günümüzde daha güvenli alternatifler (Kerberos gibi) tercih edilmektedir.

---



#### NTLM Authentication Request

![Pasted image 20241001185648.png](/img/user/resimler/Pasted%20image%2020241001185648.png)
LM hash'lerinden (65.536 karakterlik Unicode karakter setinin tamamını destekler) çok daha güçlü olsalar da, Hashcat gibi bir araç kullanılarak nispeten hızlı bir şekilde çevrimdışı olarak brute-forced edilebilirler. GPU saldırıları, 8 karakterlik NTLM anahtar uzayının tamamının ==3 saatten== kısa bir sürede brute-forced edilebileceğini göstermiştir. Daha uzun NTLM hash'lerinin kırılması, seçilen parolaya bağlı olarak daha zor olabilir ve uzun parolalar (15+ karakter) bile kurallarla birlikte çevrimdışı bir sözlük saldırısı kullanılarak kırılabilir. NTLM aynı zamanda ==pass-the-hash== saldırısına karşı da savunmasızdır, bu da bir saldırganın parolanın açık metin değerini bilmesine gerek kalmadan kullanıcının local admin olduğu hedef sistemlerde kimlik doğrulaması yapmak için yalnızca NTLM hash'ini (başka bir başarılı saldırı yoluyla elde ettikten sonra) kullanabileceği anlamına gelir.

Bir NT hash'i, tam NTLM hash'inin ikinci yarısı olan b4b9b02e6f09a9bd760f388b67351e2b şeklini alır. Bir NTLM hash'i şu şekilde görünür:

![Pasted image 20241001185851.png](/img/user/resimler/Pasted%20image%2020241001185851.png)

Yukarıdaki hash'e baktığımızda, NTLM hash'ini tek tek parçalarına ayırabiliriz:

* `Rachel` is the username
*  ==500== Relative Identifier'dır (Bağıl Tanımlayıcı) (RID). 500, ==administrator== hesabı için bilinen RID'dir
* ==aad3c435b514a4eeaad3b935b51304fe== LM hash'idir ve LM hash'leri sistemde devre dışı bırakılmışsa hiçbir şey için kullanılamaz
* ==e46b9e548fa0d122de7f59fb6d48eaa2== NT hash'idir. Bu hash, açık metin değerini ortaya çıkarmak için çevrimdışı olarak kırılabilir (parolanın uzunluğuna/gücüne bağlı olarak) veya bir pass-the-hash saldırısı için kullanılabilir. Aşağıda [CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) aracı kullanılarak yapılan başarılı bir pass-the-hash saldırısı örneği yer almaktadır:

![Pasted image 20241001190136.png](/img/user/resimler/Pasted%20image%2020241001190136.png)

Artık NTLM'nin yeteneklerini ve yapısını anladığımıza göre, protokolün NTLMv1 ve NTLMv2 üzerinden ilerleyişini inceleyelim.

Not: Ne LANMAN (LM) ne de NTLM bir ==salt== kullanmaz.


### NTLMv1 (Net-NTLMv1)
NTLM protokolü NT hash'ini kullanarak server ve client arasında bir challenge/response gerçekleştirir. NTLMv1 hem NT hem de LM hash'ini kullanır, bu da [Responder](https://github.com/lgandx/Responder) gibi bir aracı kullanarak veya bir[ NTLM relay saldırısı](https://byt3bl33d3r.github.io/practical-guide-to-ntlm-relaying-in-2017-aka-getting-a-foothold-in-under-5-minutes.html) yoluyla bir hash yakaladıktan sonra çevrimdışı “kırmayı” kolaylaştırabilir (her ikisi de bu modülün kapsamı dışındadır ve Lateral Movement ile ilgili daha sonraki modüllerde ele alınacaktır). Bu protokol ağ kimlik doğrulaması için kullanılır ve Net-NTLMv1 hash'inin kendisi bir challenge/response algoritmasından oluşturulur. Server client'e 8 baytlık rastgele bir sayı (challenge) gönderir ve client 24 baytlık bir response döndürür. Bu hash'ler pass-the-hash saldırıları için KULLANILAMAZ. Algoritma aşağıdaki gibi görünür:


### V1 Challenge & Response Algorithm
![Pasted image 20241001190701.png](/img/user/resimler/Pasted%20image%2020241001190701.png)
Tam bir NTLMv1 hash örneği aşağıdaki gibi görünür:


#### NTLMv1 Hash Example
```shell-session
u4-netntlm::kNS:338d08f8e26de93300000000000000000000000000000000:9526fb8c23a90751cdd619b6cea564742e1e4bf33006ba41:cb8086049ec4736c
```

NTLMv1, modern NTLM kimlik doğrulamasının yapı taşıdır. Her protokol gibi kusurları vardır ve kırılmaya ve diğer saldırılara karşı hassastır. Şimdi devam edelim ve NTLMv2'ye bir göz atalım ve birinci sürümün belirlediği temeli nasıl geliştirdiğini görelim.



### NTLMv2 (Net-NTLMv2)
NTLMv2 protokolü ilk olarak Windows NT 4.0 SP4'te tanıtılmış ve NTLMv1'e daha güçlü bir alternatif olarak oluşturulmuştur. 2000 yılından beri Windows'ta varsayılan olarak kullanılmaktadır. NTLMv1'in duyarlı olduğu belirli spoofing  saldırılarına karşı güçlendirilmiştir. NTLMv2, server tarafından alınan 8 baytlık challenge'a iki response gönderir. Bu response'lar, challenge'ın 16 baytlık bir HMAC-MD5 hash'ini, client'tan rastgele oluşturulmuş bir challenge'ı ve kullanıcının kimlik bilgilerinin bir HMAC-MD5 hash'ini içerir. Geçerli saati, 8 baytlık rastgele bir değeri ve alan adını içeren değişken uzunlukta bir istemci meydan okuması kullanılarak ikinci bir yanıt gönderilir. Algoritma aşağıdaki gibidir:

---

NTLMv2 protokolünü daha ayrıntılı ve madde madde açıklayayım:

1. **NTLMv2'nin Tanıtımı ve Geliştirilmesi:**
    
    - **NTLMv2**, Windows NT 4.0 SP4 ile tanıtıldı.
    - **NTLMv2**, daha önceki protokol olan **NTLMv1**'in güvenlik açıklarını kapatmak için geliştirilmiş bir alternatif olarak tasarlandı.
    - **Windows Server 2000**'den itibaren **varsayılan kimlik doğrulama protokolü** olarak kullanıma sunuldu.

2. **NTLMv2'nin Güçlendirilmiş Güvenliği:**
    
    - **NTLMv1**'in karşılaştığı bazı **sahtecilik (spoofing)** saldırılarına karşı **NTLMv2** daha dayanıklıdır.
    - Yani NTLMv2, güvenlik açıklarını kapatarak daha güvenli bir kimlik doğrulama sağlar.

3. **NTLMv2'nin Responsları:**
    
    - NTLMv2, sunucudan aldığı **8 baytlık bir challenge**  ile iki farklı **yanıt** gönderir.

4. **Birinci Yanıt:**
    
    - Bu yanıt, **HMAC-MD5** (Hash-based Message Authentication Code - MD5) algoritması ile oluşturulmuş bir **16 baytlık hash** içerir.
    - Bu hash, aşağıdaki bileşenlerin birleşiminden türetilir:
        - **Sunucudan alınan challenge**.
        - **Client tarafından rastgele oluşturulan challenge**.
        - **Kullanıcının kimlik bilgileri** (şifre veya parola).

5. **İkinci Yanıt:**
    
    - İkinci yanıt, **değişken uzunluktaki client challenge'ı** içerir.
    - Bu challenge, aşağıdaki bileşenleri içerir:
        - **Mevcut zaman** (istek gönderildiği andaki zaman).
        - **8 baytlık rastgele bir değer**.
        - **Doamin adı ** (domain name).

6. **Genel Algoritma:**
    
    - NTLMv2 protokolü, bu iki yanıtla birlikte kimlik doğrulaması gerçekleştirilir. Hem client hem de server, bu yanıtları doğrulamak için aynı algoritmayı kullanır.
    - NTLMv2'nin güvenliği, kullanıcının kimlik bilgilerini ve çeşitli rastgele verileri içerdiği için NTLMv1'e göre daha güçlüdür.

Kısaca, NTLMv2'nin yapısı daha güvenli olup, kimlik doğrulama için hem sunucu hem de client tarafından oluşturulan karmaşık veriler ve HMAC-MD5 hash'leri kullanır. Bu sayede daha güvenli bir doğrulama ve sahtecilik engellemesi sağlanır.

![Pasted image 20250104234858.png](/img/user/resimler/Pasted%20image%2020250104234858.png)

---


#### V2 Challenge & Response Algorithm
![Pasted image 20241001194949.png](/img/user/resimler/Pasted%20image%2020241001194949.png)

NTLMv2 hash'ine bir örnek:

#### NTLMv2 Hash Example
```shell-session
admin::N46iSNekpT:08ca45b7d7ea58ee:88dcbe4446168966a153a0064958dac6:5c7830315c7830310000000000000b45c67103d07d7b95acd12ffa11230e0000000052920b85f78d013c31cdb3b92f5d765c783030
```

Geliştiricilerin NTLMv2'nin kırılmasını zorlaştırarak ve birden fazla aşamadan oluşan daha sağlam bir algoritma sunarak v1'i geliştirdiklerini görebiliriz. Devam etmeden önce tartışmamız gereken bir kimlik doğrulama mekanizması daha var. Bu yöntem, çalışmak için kalıcı bir ağ bağlantısı gerektirmediği için bizim için önemlidir.


### Domain Cached Credentials (MSCache2)
Bir AD ortamında, bu bölümde ve bir önceki bölümde bahsedilen kimlik doğrulama yöntemleri, erişmeye çalıştığımız hostun ağın “beyni” olan Domain Controller ile iletişim kurmasını gerektirir. [Microsoft MS Cache v1 ve v2](https://webstersprodigy.net/lander) algoritmasını (==Domain Cached Credentials== (DCC) olarak da bilinir), domain'e bağlı bir host'un bir domain controller ile iletişim kuramaması (örneğin, bir ağ kesintisi veya başka bir teknik sorun nedeniyle) ve dolayısıyla NTLM/Kerberos kimlik doğrulamasının söz konusu host'a erişmek için çalışmaması gibi olası sorunları çözmek için geliştirmiştir. Host'lar, makinede başarıyla oturum açan tüm domain kullanıcıları için son ==10== hash'i ==HKEY_LOCAL_MACHINE\SECURITY\Cache== registy key (anahtarına) kaydeder. Bu hash'ler pass-the-hash saldırılarında kullanılamaz. Ayrıca, son derece güçlü bir GPU kırma donanımı kullanıldığında bile Hashcat gibi bir araçla hash kırmak çok yavaştır, bu nedenle bu hashleri kırma girişimlerinin genellikle son derece hedefli olması veya kullanımda olan çok zayıf bir parolaya dayanması gerekir. Bu hashler bir saldırgan veya pentester tarafından bir hosta local admin erişimi sağlandıktan sonra elde edilebilir ve aşağıdaki formata sahiptir: ==$DCC2$10240#bjones#e4e938d12fe5974dc42a90120bd9c90f==. Sızma testi uzmanları olarak, bir AD ortamını değerlendirirken karşılaşabileceğimiz çeşitli hash türlerini, bunların güçlü ve zayıf yönlerini, nasıl kötüye kullanılabileceklerini (cleartext, pass-the-hash veya relayed'e kırma) ve bir saldırının ne zaman boşa çıkabileceğini (örneğin, bir dizi Domain Cached Credentials'ı kırmaya çalışmak için günler harcamak) anlamamız çok önemlidir.

Kimlik doğrulama protokollerini ve ilgili parola hash'lerini ele aldığımıza göre şimdi hem penetration tester hem de attackerlar için genellikle en önemli hedef olan Active Directory'deki kullanıcılara ve gruplara bakalım. Çeşitli ayrıcalıklara sahip olabilirler ve bir ortamda lateral olarak hareket etmek veya korunan kaynaklara erişim sağlamak için kullanılabilirler.


Soru : Hangi Hashing protokolü simetrik ve asimetrik kriptografi yeteneğine sahiptir? (Kerberos)

Soru : NTLM kimlik doğrulaması için üç mesaj kullanır; Negotiate, Challenge ve <__>. Eksik mesaj nedir? (boşluğu doldurun) (authenticate)

Soru : Domain Cached Credentials mekanizması varsayılan olarak bir host'a kaç tane hash kaydeder? (10)


### User and Machine Accounts

User hesapları hem lokal sistemlerde (AD'ye bağlı olmayan) hem de Active Directory'de bir kişiye veya bir programa (sistem servisi gibi) bir bilgisayarda oturum açma ve haklarına göre kaynaklara erişme yeteneği vermek için oluşturulur. Bir kullanıcı oturum açtığında, sistem parolasını doğrular ve bir access token  oluşturur. Bu token bir prosesin veya thread'in güvenlik içeriğini tanımlar ve kullanıcının güvenlik kimliğini ve grup üyeliğini içerir. Bir kullanıcı bir prosesle etkileşime girdiğinde, bu token sunulur. User accounts (kullanıcı hesapları), çalışanların/işverenleri bir bilgisayarda oturum açmasına ve kaynaklara erişmesine, programları veya servisleri belirli bir güvenlik contextında çalıştırmasına (örneğin, bir ağ servisi hesabı yerine yüksek ayrıcalıklı bir kullanıcı olarak çalıştırma) ve ağ dosya paylaşımları, dosyalar, uygulamalar vb. gibi objectlere ve bunların özelliklerine erişimi yönetmesine izin vermek için kullanılır. Kullanıcılar, bir veya daha fazla üye içerebilen gruplara atanabilir. Bu gruplar kaynaklara erişimi kontrol etmek için de kullanılabilir. Bir admin için ayrıcalıkları her bir kullanıcıya birçok kez atamak yerine bir gruba (tüm grup üyelerinin devraldığı) bir kez atamak daha kolay olabilir. Bu, yönetimi basitleştirmeye yardımcı olur ve kullanıcı haklarının verilmesini ve iptal edilmesini kolaylaştırır.

* Kısaca : Kullanıcı hesapları, oturum açma ve kaynaklara erişimi sağlar; access token ile güvenlik contex'i tanımlanır, gruplar ise erişim ve hak yönetimini kolaylaştırır.

Standart kullanıcı hesaplarının sağlanması ve yönetilmesi Active Directory'nin temel unsurlarından biridir. Tipik olarak, karşılaştığımız her şirkette kullanıcı başına en az bir AD kullanıcı hesabı sağlanır. Bazı kullanıcıların iş rollerine göre (örneğin, bir BT admin veya Help Desk üyesi) iki veya daha fazla hesabı olabilir. Belirli bir kullanıcıya bağlı standart kullanıcı ve admin hesaplarının yanı sıra, arka planda belirli bir uygulamayı veya servisi çalıştırmak veya domain ortamında diğer hayati fonksiyonları yerine getirmek için kullanılan birçok servis hesabı görürüz. 1.000 çalışanı olan bir kuruluşun 1.200 veya daha fazla aktif kullanıcı hesabı olabilir! Ayrıca eski çalışanlardan, geçici/mevsimlik çalışanlardan, stajyerlerden vb. yüzlerce devre dışı bırakılmış hesabı olan kuruluşlar da görebiliriz. Bazı şirketler denetim amacıyla bu hesapların kayıtlarını tutmak zorundadır, bu nedenle çalışan işten çıkarıldığında bu hesapları devre dışı bırakırlar (ve umarız tüm ayrıcalıkları kaldırırlar), ancak silmezler. FORMER EMPLOYEES gibi devre dışı bırakılmış birçok hesap içeren bir OU görmek yaygındır.

![Pasted image 20241001201747.png](/img/user/resimler/Pasted%20image%2020241001201747.png)
Bu modülde daha sonra göreceğimiz gibi, kullanıcı hesaplarına Active Directory'de birçok hak verilebilir. Temel olarak ortamın çoğuna okuma erişimi olan salt okunur kullanıcılar (standart bir Domain User'ın aldığı izinler), Enterprise Admin'e ( domain'deki her objectnin tam kontrolüne sahip) ve aradaki sayısız kombinasyona kadar yapılandırılabilirler. Kullanıcılara bu kadar çok hak atanabildiği için, nispeten kolay bir şekilde yanlış yapılandırılabilir ve bir saldırganın veya sızma test uzmanının yararlanabileceği istenmeyen haklar verilebilir. Kullanıcı hesapları muazzam bir saldırı yüzeyi sunar ve genellikle bir sızma testi sırasında bir dayanak noktası elde etmek için kilit bir odak noktasıdır. Kullanıcılar genellikle herhangi bir kuruluştaki en zayıf halkadır. İnsan davranışını yönetmek ve her kullanıcının zayıf veya paylaşılan parolalar seçmesini, yetkisiz yazılımlar yüklemesini veya yöneticilerin dikkatsiz hatalar yapmasını veya hesap yönetimi konusunda aşırı müsamahakâr davranmasını hesaba katmak zordur. Bununla mücadele etmek için, bir kuruluşun kullanıcı hesapları etrafında ortaya çıkabilecek sorunlarla mücadele etmek için politikalara ve prosedürlere sahip olması ve kullanıcıların domain'e getirdiği doğal riski azaltmak için derinlemesine savunmaya sahip olması gerekir.

Kullanıcılarla ilgili yanlış yapılandırmalar ve saldırılarla ilgili ayrıntılar bu modülün kapsamı dışındadır. Yine de, kullanıcıların herhangi bir Active Directory ağında sahip olabileceği büyük etkiyi anlamak ve karşılaşabileceğimiz farklı kullanıcı/hesap türleri arasındaki farkları anlamak önemlidir.


### Local Accounts

Local hesaplar belirli bir sunucu veya workstation üzerinde local olarak saklanır. Bu hesaplara o host üzerinde tek tek ya da grup üyeliği yoluyla haklar atanabilir. Atanan tüm haklar yalnızca söz konusu host için verilebilir ve domain genelinde çalışmaz. Lokal kullanıcı hesapları güvenlik sorumluları olarak kabul edilir ancak yalnızca bağımsız bir host üzerindeki kaynaklara erişimi yönetebilir ve bunların güvenliğini sağlayabilir. Bir Windows sisteminde oluşturulan birkaç varsayılan lokal kullanıcı hesabı vardır:

* ==Administrator==: Bu hesap `SID S-1-5-domain-500`'e sahiptir ve yeni bir Windows kurulumunda oluşturulan ilk hesaptır. Sistemdeki hemen hemen her kaynak üzerinde tam denetime sahiptir. Silinemez veya kilitlenemez, ancak devre dışı bırakılabilir veya yeniden adlandırılabilir. Windows 10 ve Server 2016 host'ları varsayılan olarak built-in administrator hesabını `devre dışı bırakır` ve kurulum sırasında lokal administrator grubunda başka bir lokal hesap oluşturur.

* ==Guest (Misafir)==: bu hesap varsayılan olarak devre dışıdır. Bu hesabın amacı, bilgisayarda hesabı olmayan kullanıcıların sınırlı erişim haklarıyla geçici olarak oturum açmasına izin vermektir. Varsayılan olarak boş bir parolaya sahiptir ve bir host'a anonim erişime izin vermenin güvenlik riski nedeniyle genellikle devre dışı bırakılması önerilir.

* ==SYSTEM==: Bir Windows host üzerindeki SYSTEM (veya NT AUTHORITY\SYSTEM) hesabı, işletim sistemi tarafından iç fonksiyonlarının çoğunu gerçekleştirmek için kurulan ve kullanılan varsayılan hesaptır. Linux'taki Root hesabının aksine, SYSTEM bir servis hesabıdır ve normal bir kullanıcıyla tamamen aynı contextda çalışmaz. Bir host üzerinde çalışan proseslerin ve servislerin çoğu SYSTEM context'i altında çalıştırılır. Bu hesapla ilgili dikkat edilmesi gereken bir nokta, bu hesap için bir profilin mevcut olmamasıdır, ancak host üzerindeki neredeyse her şey üzerinde izinlere sahip olacaktır. User Manager'da görünmez ve herhangi bir gruba eklenemez. SYSTEM hesabı, bir Windows hostunda ulaşılabilecek en yüksek izin düzeyidir ve varsayılan olarak bir Windows sistemindeki tüm dosyalar için Full Control izinlerine sahiptir.

* ==Network Service==: Bu, Windows servislerini çalıştırmak için Service Control Manager (SCM) tarafından kullanılan önceden tanımlanmış bir lokal hesaptır. Bir servis bu özel hesap contextında çalıştığında, remote servislere kimlik bilgilerini sunacaktır.

	* Bir servis **Network Service** hesabı context'iinde çalıştırıldığında, remote servislere erişim gerektiğinde **kimlik bilgilerini (credentials)** kullanarak kendini doğrular. Yani, bu özel hesap remote sistemlerde işlem yapmak için bir kimlik doğrulama mekanizması sunar. Ancak, **Network Service** hesabının kimlik bilgileri, genellikle sistemin makine hesabı (örneğin, `MACHINE$`) olarak temsil edilir ve bu bilgiler, remote kaynaklara erişim sağlamak için kullanılır.

Kısaca, servis, başka bir bilgisayardaki bir kaynağa erişmesi gerektiğinde, kendi kimliğini (örneğin, makinenin adı ve security context) göndererek doğrulama yapar.

*  ==Local Service==: Bu, Windows servislerini çalıştırmak için Service Control Manager (SCM) tarafından kullanılan önceden tanımlanmış başka bir lokal hesaptır. Bilgisayarda minimum ayrıcalıklarla yapılandırılır ve ağa anonim kimlik bilgileri sunar.

Çeşitli hesapların tek bir Windows sisteminde ve bir domain ağında birlikte nasıl çalıştığını daha iyi anlamak için Microsoft'un [lokal varsayılan hesaplarla](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/local-accounts) ilgili belgelerini derinlemesine incelemeye değer. Bunları incelemek ve aralarındaki farkları anlamak için biraz zaman ayırın.


### Domain Users
Domain kullanıcılarının local kullanıcılardan farkı, user hesaplarına veya hesabın üyesi olduğu gruba verilen izinlere bağlı olarak dosya sunucuları, yazıcılar, intranet host'ları ve diğer objectler gibi kaynaklara erişmek için domain'den haklara sahip olmalarıdır. Domain kullanıcı hesapları, lokal kullanıcıların aksine, domain içindeki herhangi bir hostta oturum açabilir. Birçok farklı Active Directory hesap türü hakkında daha fazla bilgi için bu [bağlantıya](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-accounts) göz atın. Ancak akılda tutulması gereken bir hesap ==KRBTGT== hesabıdır. Bu, AD altyapısında built-in olarak bulunan bir local hesap türüdür. Bu hesap, domain kaynakları için kimlik doğrulama ve erişim sağlayan Key Distribution servisi için bir servis hesabı olarak görev yapar. Bu hesap birçok saldırganın ortak hedefidir çünkü kontrol veya erişim elde etmek bir saldırganın domain'e sınırsız erişime sahip olmasını sağlar. [Golden Ticket](https://attack.mitre.org/techniques/T1558/001/) saldırısı gibi saldırılar yoluyla bir domain'de ayrıcalık yükseltme ve kalıcılık için kullanılabilir.


### User Naming Attributes
Active Directory'de güvenlik, oturum açma adı veya kimliği gibi kullanıcı objectlerini tanımlamaya yardımcı olmak için bir dizi kullanıcı adlandırma attribute kullanılarak geliştirilebilir. Aşağıda AD'deki birkaç önemli Adlandırma Özniteliği verilmiştir:

| **Özellik**                 | **Açıklama**                                                                                                                                             |
| --------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **UserPrincipalName (UPN)** | Kullanıcının birincil oturum açma adı. Geleneksel olarak, kullanıcıya ait e-posta adresi formatında kullanılır.                                          |
| **ObjectGUID**              | Kullanıcının benzersiz tanımlayıcısıdır. AD'de değişmez ve kullanıcı silinse bile benzersizliğini korur.                                                 |
| **SAMAccountName**          | Eski Windows clientlerde ve sunucuları için desteklenen bir oturum açma adıdır.                                                                          |
| **objectSID**               | Kullanıcının **Security Identifier (SID)** kimliğidir. Güvenlik etkileşimlerinde kullanıcı ve grup üyeliklerini tanımlamak için kullanılır.              |
| **sIDHistory**              | Kullanıcının önceki SID'lerini içerir. Genellikle domain taşımalarında görülür. Yeni domain'deki SID **objectSID** olurken, eski SID'ler buraya eklenir. |

### Ortak Kullanıcı Attribute'leri

```powershell-session
PS C:\htb Get-ADUser -Identity htb-student

DistinguishedName : CN=htb student,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
Enabled           : True
GivenName         : htb
Name              : htb student
ObjectClass       : user
ObjectGUID        : aa799587-c641-4c23-a2f7-75850b4dd7e3
SamAccountName    : htb-student
SID               : S-1-5-21-3842939050-3880317879-2865463114-1111
Surname           : student
UserPrincipalName : htb-student@INLANEFREIGHT.LOCAL
```

Kullanıcı objectsi attribute'larına daha derin bir bakış için bu [sayfaya](https://learn.microsoft.com/en-us/windows/win32/ad/user-object-attributes) göz atın. AD'deki herhangi bir object için birçok attribute ayarlanabilir. Birçok object hiçbir zaman kullanılmayacak veya güvenlik uzmanları olarak bizleri ilgilendirmeyecektir. Yine de, hassas veriler içerebilecek veya bir saldırıya yardımcı olabilecek en yaygın ve daha belirsiz olanları tanımak önemlidir.


### Domain-joined ve Domain-joined Olmayan Makineler
Bilgisayar kaynakları söz konusu olduğunda, tipik olarak yönetildikleri birkaç yol vardır. Aşağıda, bir domain'e bağlı bir host ile yalnızca bir workgroup'ta bulunan bir host arasındaki farkları tartışacağız.


##### Domain joined
Bir domain'e bağlanan host'lar kurum içinde daha kolay bilgi paylaşımına ve kaynakları, politikaları ve güncellemeleri toplayabilecekleri merkezi bir yönetim noktasına (DC) sahip olurlar. Bir domain'e bağlanan bir host, domain'in Group Policy'si aracılığıyla gerekli tüm konfigürasyonları veya değişiklikleri edinecektir. Buradaki avantaj, domain'deki bir kullanıcının oturum açabilmesi ve sadece üzerinde çalıştığı hosttan değil, domain'e bağlı herhangi bir hosttan kaynaklara erişebilmesidir. Bu, kurumsal ortamlarda göreceğiniz tipik kurulumdur.


##### Non-domain joined
Domain'e bağlı olmayan bilgisayarlar veya bir workgroup'taki bilgisayarlar domain politikası tarafından yönetilmez. Bunu göz önünde bulundurarak, local ağınızın dışındaki kaynakları paylaşmak, bir domain üzerinde olacağından çok daha karmaşıktır. Bu, ev kullanımı amaçlı bilgisayarlar veya aynı LAN üzerindeki küçük işletme kümeleri için uygundur. Bu kurulumun avantajı, bireysel kullanıcıların hostlarında yapmak istedikleri her türlü değişiklikten sorumlu olmalarıdır. Bir workgroup bilgisayarındaki tüm kullanıcı hesapları yalnızca o hostta bulunur ve profiller workgroup içindeki diğer hostlara taşınmaz.

AD ortamındaki bir makine hesabının (==NT AUTHORITY\SYSTEM== düzeyi erişim) standart bir domain kullanıcı hesabıyla aynı hakların çoğuna sahip olacağına dikkat etmek önemlidir. Bu önemlidir çünkü bir domain'i numaralandırmaya ve saldırmaya başlamak için (daha sonraki modüllerde göreceğimiz gibi) her zaman tek bir kullanıcının hesabı için bir dizi geçerli kimlik bilgisi edinmemiz gerekmez. Başarılı bir remote code execution exploit yoluyla veya bir host üzerinde ayrıcalıkları yükselterek domain-joined bir Windows hostuna SYSTEM seviyesinde erişim elde edebiliriz. Bu erişim genellikle yalnızca belirli bir host üzerindeki hassas verileri (parolalar, SSH anahtarları, hassas dosyalar vb.) yağmalamak için yararlı olduğu için göz ardı edilir. Gerçekte, SYSTEM hesabı contextında erişim, domain içindeki verilerin çoğuna okuma erişimi sağlar ve AD ile ilgili uygulanabilir saldırılara geçmeden önce domain hakkında mümkün olduğunca fazla bilgi toplamak için harika bir başlangıç noktasıdır.

Soru : Doğru veya Yanlış; Lokal bir kullanıcı hesabı, domain'e bağlı herhangi bir host'ta oturum açmak için kullanılabilir. (Yanlış)

Soru : Hangi varsayılan kullanıcı hesabı “S-1-5-domain-500” SID'sine sahiptir? (Administrator)

Soru : Bir Windows host'unda mümkün olan en yüksek izin düzeyine sahip hesap hangisidir? (System)

Soru : Hangi kullanıcı adlandırma özelliği kullanıcıya özgüdür ve hesap silinse bile öyle kalacaktır? (ObjectGUID)


### Active Directory Grupları
User'lardan sonra gruplar Active Directory'deki bir diğer önemli objectdir. Benzer kullanıcıları bir araya getirebilir ve toplu olarak hak ve erişim atayabilirler. Gruplar saldırganlar ve sızma testçileri için bir başka önemli hedeftir, çünkü üyelerine verdikleri haklar kolayca görülemeyebilir, ancak doğru şekilde ayarlanmadığında kötüye kullanılabilecek aşırı (ve hatta istenmeyen) ayrıcalıklar verebilir. Active Directory'de [birçok built-in grup](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#about-active-directory-groups) vardır ve çoğu kuruluş ayrıca hakları ve ayrıcalıkları tanımlamak için kendi gruplarını oluşturarak domain içindeki erişimi daha da yönetir. Bir AD ortamındaki grupların sayısı çığ gibi büyüyerek hantal hale gelebilir ve kontrol edilmediği takdirde istenmeyen erişimlere yol açabilir. Farklı grup türlerini kullanmanın etkisini anlamak ve her kuruluşun kendi domain'inde hangi grupların bulunduğunu, bu grupların üyelerine verdiği ayrıcalıkları periyodik olarak denetlemesi ve bir kullanıcının günlük işlerini gerçekleştirmesi için gerekli olanın ötesinde aşırı grup üyeliği olup olmadığını kontrol etmesi çok önemlidir. İleride, var olan farklı grup türlerini ve atanabilecekleri kapsamları tartışacağız.

Gruplar (Groups) ve Organizational Units (OUs) arasındaki fark, sıkça sorulan bir sorudur. Modülde daha önce tartışıldığı gibi, **OUs**, kullanıcıları, grupları ve bilgisayarları gruplandırarak yönetimi kolaylaştırmak ve domain'deki belirli objectlere **Group Policy** ayarlarını uygulamak için kullanılır.

**Gruplar (Groups)** ise genellikle kaynaklara erişim izinlerini atamak için kullanılır. Ayrıca, **OUs**, bir kullanıcıya ek yönetici yetkileri (örneğin, grup üyeliği aracılığıyla devralınabilecek haklar) vermeden şifre sıfırlama veya kullanıcı hesaplarının kilidini açma gibi belirli yönetim görevlerini devretmek için de kullanılabilir.


### Grup Türleri
Daha basit bir ifadeyle, gruplar kullanıcıları, bilgisayarları ve iletişim objectlerini, izinler üzerinde yönetim kolaylığı sağlayan ve yazıcılar ve dosya paylaşım erişimi gibi kaynakların atanmasını kolaylaştıran yönetim birimlerine yerleştirmek için kullanılır. Örneğin, bir yöneticinin bir departmanın 50 üyesine yeni bir paylaşım sürücüsüne erişim ataması gerekiyorsa, her kullanıcının hesabını tek tek eklemek zaman alıcı olacaktır. İzinlerin bu şekilde verilmesi, kaynaklara kimin erişimi olduğunu denetlemeyi ve izinleri temizlemeyi/iptal etmeyi de zorlaştıracaktır. Bunun yerine, bir sistem yöneticisi mevcut bir grubu kullanabilir ya da yeni bir grup oluşturabilir ve bu gruba kaynak üzerinde belirli izinler verebilir. Buradan itibaren, gruptaki her kullanıcı gruptaki üyeliklerine göre izinleri devralacaktır. Bir veya daha fazla kullanıcı için izinlerin değiştirilmesi veya iptal edilmesi gerekirse, bu kullanıcılar gruptan çıkarılabilir ve diğer kullanıcılar bundan etkilenmez ve izinlerine dokunulmaz.

Active Directory'deki grupların iki temel özelliği vardır: ==type== ve ==scope== . ==Grup type'ı== grubun amacını tanımlarken, ==grup scope=='u grubun domain veya forest içinde nasıl kullanılabileceğini gösterir. Yeni bir grup oluştururken, bir grup türü seçmeliyiz. İki ana tür vardır: ==security== (güvenlik) ve ==distribution== (dağıtım) grupları.


### Group Type And Scope

![Pasted image 20241001223322.png](/img/user/resimler/Pasted%20image%2020241001223322.png)

==Security groups (Güvenlik grupları)== türü, öncelikle izinleri ve hakları teker teker atamak yerine bir kullanıcı topluluğuna atamayı kolaylaştırmak içindir. Belirli bir kaynak için izinler ve haklar atarken yönetimi basitleştirir ve ek yükü azaltırlar. Bir security grubuna eklenen tüm kullanıcılar, gruba atanan tüm izinleri devralır, bu da grubun izinlerini değiştirmeden kullanıcıları grupların içine ve dışına taşımayı kolaylaştırır.

==Distribution groups (Dağıtım grupları)== türü, Microsoft Exchange gibi e-posta uygulamaları tarafından mesajları grup üyelerine dağıtmak için kullanılır. Posta listeleri gibi çalışırlar ve Microsoft Outlook'ta bir e-posta oluştururken “Kime” alanına e-postaların otomatik olarak eklenmesini sağlarlar. Bu grup type'ı, bir domain ortamındaki kaynaklara izin atamak için kullanılamaz.


### Group Scopes

Yeni bir grup oluştururken atanabilecek üç farklı grup kapsamı vardır.

* Domain Local Group
* Global Group
* Universal Group

### Domain Local Group
Domain lokal grupları yalnızca oluşturulduğu domain'deki domain kaynaklarına yönelik izinleri yönetmek için kullanılabilir. Lokal gruplar diğer domainlerde kullanılamaz ancak DİĞER domainlerden kullanıcıları içerebilir. Lokal gruplar diğer lokal grupların içine yerleştirilebilir (dahil edilebilir) ancak global grupların içine YERLEŞTİRİLEMEZ.


### Global Group

Global gruplar, başka bir domain'deki kaynaklara erişim izni vermek için kullanılabilir. Bir global grup yalnızca oluşturulduğu domain'deki hesapları içerebilir. Global gruplar hem diğer global gruplara hem de lokal gruplara eklenebilir.


### Universal Group

Universal grup kapsamı, birden fazla domain'e dağıtılmış kaynakları yönetmek için kullanılabilir ve aynı forest içindeki herhangi bir objectye izinler verilebilir. Bir kuruluş içindeki tüm domainler tarafından kullanılabilirler ve herhangi bir domainin kullanıcılarını içerebilirler. Domain lokal ve global gruplarının aksine, evrensel gruplar Global Katalog'da (GC) saklanır ve evrensel bir gruba object eklenmesi veya gruptan object çıkarılması forest çapında replikasyonu tetikler. Yöneticilerin diğer grupları (global gruplar gibi) universal grupların üyesi olarak tutmaları önerilir çünkü universal gruplar içindeki global grup üyeliğinin global gruplardaki bireysel kullanıcı üyeliğinden daha az değişme olasılığı vardır. Replikasyon yalnızca bir kullanıcı genel bir gruptan çıkarıldığında bireysel domain düzeyinde tetiklenir. Evrensel gruplar içinde tek tek kullanıcılar ve bilgisayarlar ( global gruplar yerine) tutulursa, her değişiklik yapıldığında forest çapında replikasyon tetiklenir. Bu da çok fazla ağ yükü ve sorun potansiyeli yaratabilir. Aşağıda AD'deki grupların ve scope ayarlarının bir örneği bulunmaktadır. Lütfen bazı kritik gruplara ve kapsamlarına dikkat edin. (Örneğin, Domain Adminlere kıyasla Enterprise ve Schema adminleri)


### AD Group Scope Examples

![Pasted image 20241001223939.png](/img/user/resimler/Pasted%20image%2020241001223939.png)

Grup skopları değiştirilebilir, ancak birkaç uyarı vardır:

* Bir Global Grup ancak başka bir Global Grubun parçası DEĞİLSE bir Universal Gruba dönüştürülebilir.

* Bir Domain Local Group, yalnızca Domain Local Group üye olarak başka bir Domain Local Group içermiyorsa Universal Group'a dönüştürülebilir.

* Bir Universal Grup herhangi bir kısıtlama olmaksızın bir Domain Local Gruba dönüştürülebilir.

* Bir Universal Grup yalnızca üye olarak başka Universal Grupları İÇERMİYORSA bir Global Gruba dönüştürülebilir.


## Built-in vs. Custom Groups

Bir domain oluşturulduğunda, Domain Local Group kapsamına sahip birkaç built-in security grubu oluşturulur. Bu gruplar, belirli yönetim amaçları için kullanılır ve bir sonraki bölümde daha detaylı olarak ele alınacaktır. Önemli bir nokta, bu built-in gruplara yalnızca kullanıcı hesaplarının eklenebileceğidir, çünkü grup içi gruplaşmayı (gruplar içinde gruplar) desteklemezler.

Örneklerden biri, yalnızca kendi domainindeki hesapları içerebilen **Domain Admins** adlı Global security grubudur. Eğer bir organizasyon, domain B'deki bir hesabın domain A'daki bir domain controller'ında yönetimsel işlemler yapmasına izin vermek istiyorsa, bu hesap **Administrators** adlı local gruba eklenmelidir, çünkü bu grup bir Domain Local grubudur.

Active Directory, birçok grup ile önceden doldurulmuş olarak gelir, ancak çoğu organizasyon, kendi amaçları için ek gruplar (hem security hem de distribution grupları) oluşturmayı yaygın olarak tercih eder. AD ortamına yapılan değişiklikler ve eklemeler, yeni grupların oluşturulmasına da yol açabilir. Örneğin, Microsoft Exchange bir domain'e eklendiğinde, domain'e çeşitli security grupları ekler; bunlardan bazıları yüksek ayrıcalıklara sahip olup, doğru yönetilmezse domain içinde ayrıcalıklı erişim sağlamak için kullanılabilir.


### Nested Group Membership

İç içe grup üyeliği AD'de önemli bir kavramdır. Daha önce de belirtildiği gibi, bir Domain Local Group aynı domain içindeki başka bir Domain Local Group'un üyesi olabilir. Bu üyelik sayesinde, bir kullanıcı doğrudan kendi hesabına veya hatta doğrudan üyesi olduğu gruba değil, grubunun üyesi olduğu gruba atanan ayrıcalıkları devralabilir. Bu durum bazen bir kullanıcıya domain'i derinlemesine değerlendirmeden ortaya çıkarılması zor olan istenmeyen ayrıcalıklar verilmesine yol açabilir. BloodHound gibi araçlar, bir kullanıcının bir veya daha fazla grup iç içe geçmesi yoluyla devralabileceği ayrıcalıkların ortaya çıkarılmasında özellikle yararlıdır. Bu, sızma testi uzmanları için ince ayrıntıları yanlış yapılandırmaları ortaya çıkarmak için önemli bir araçtır ve aynı zamanda sistem yöneticileri ve benzerleri için domainlerinin güvenlik duruşu hakkında derinlemesine (görsel olarak) bilgi edinmek için son derece güçlüdür.

Aşağıda, iç içe grup üyeliği yoluyla miras alınan ayrıcalıkların bir örneği verilmiştir. **DCorner**, **Helpdesk Level 1** grubunun doğrudan bir üyesi olmasa da, **Help Desk** grubuna üyeliği sayesinde **Helpdesk Level 1** grubunun tüm üyelerinin sahip olduğu ayrıcalıkları kazanır. Bu durumda, bu ayrıcalık, **Tier 1 Admins** grubuna bir üye eklemelerine (GenericWrite) olanak tanıyacaktır. Eğer bu grup, domain içinde herhangi bir yükseltilmiş ayrıcalık sağlıyorsa, bu muhtemelen bir sızma testi uzmanı için önemli bir hedef olacaktır. Burada, kullanıcımızı gruba ekleyip, **Tier 1 Admins** grubunun üyelerine sağlanan ayrıcalıklara (örneğin, bir veya birden fazla hosta yerel yönetici erişimi) sahip olabiliriz, bu da daha fazla erişim sağlamak için kullanılabilir.

### BloodHound ile İç İçe Grupları İnceleme

![Pasted image 20241001224539.png](/img/user/resimler/Pasted%20image%2020241001224539.png)


## Important Group Attributes
Kullanıcılar gibi grupların da birçok [attribute'ler](http://www.selfadsi.org/group-attributes.htm) vardır. En [önemli grup özelliklerinden](https://learn.microsoft.com/en-us/windows/win32/ad/group-objects) bazıları şunlardır:

==cn==: cn veya Common-Name, Active Directory Domain Services'daki grubun adıdır.

==member==: Hangi kullanıcı, grup ve kişi objectlerinin grubun üyesi olduğu.

==groupType==: Grup türünü ve kapsamını belirten bir tamsayıdır.

==memberOf==: Grubu üye olarak içeren tüm grupların bir listesi (iç içe grup üyeliği).

==objectSid==: Bu, grubu bir güvenlik sorumlusu olarak tanımlamak için kullanılan benzersiz değer olan grubun güvenlik tanımlayıcısı veya SID'sidir.


Gruplar, AD'de diğer objectleri bir arada gruplandırmak ve hakların ve erişimin yönetimini kolaylaştırmak için kullanılabilen temel objectlerdir. Grup type'ları ve scope'ları arasındaki farkları incelemek için zaman ayırın. Bu bilgi, AD'yi yönetmenin yanı sıra aynı ve farklı domainlerdeki gruplar arasındaki ilişkileri ve bir sızma testinin keşif aşamasında hangi bilgilerin numaralandırılabileceğini anlamak için de faydalıdır. Tek bir domain içinde ve güven sınırları ötesinde saldırılar gerçekleştirmek için farklı grup türlerinin nasıl kullanılabileceğini anlamak, sahip olunması gereken mükemmel bir bilgidir. Bu bölümde Grupları derinlemesine inceledik, şimdi Haklar ve Ayrıcalıklar  (`Rights` and `Privileges`.) arasındaki farkları inceleyelim.

Soru : Kullanıcılara izin ve hak atamak için en iyi hangi grup türü kullanılır?
Cevap : Security

Soru : Doğru veya Yanlış; Bir “Global Grup” yalnızca oluşturulduğu domain'deki hesapları içerebilir.
Cevap : True

Soru :  Bir Universal grubu bir Domain Local grubuna dönüştürülebilir mi? (evet veya hayır)
Cevap : yes


# Active Directory Rights and Privileges

Rights (haklar) ve privileges AD yönetiminin temel taşlarıdır ve yanlış yönetildikleri takdirde saldırganlar veya penetrasyon testçileri tarafından kolayca kötüye kullanılabilirler. Access Rights ve privileges AD'de (ve genel olarak bilgi güvenliğinde) iki önemli konudur ve aralarındaki farkı anlamamız gerekir. Right'lar genellikle ==kullanıcılara veya gruplara== atanır ve dosya gibi bir objeye erişim izinleriyle ilgilenirken, ==privilege'ler kullanıcıya bir programı çalıştırma, sistemi kapatma, parolaları sıfırlama vb.== gibi bir eylemi gerçekleştirme izni verir. Privilege'lar kullanıcılara bireysel olarak atanabilir veya built-in ya da özel grup üyeliği yoluyla verilebilir. Windows bilgisayarlarda ==User Rights Assignment (Kullanıcı Hakları Ataması)== adı verilen bir kavram vardır ve bunlar haklar olarak adlandırılsa da aslında bir kullanıcıya verilen privilege türleridir. Bunları bu bölümün ilerleyen kısımlarında tartışacağız. Daha geniş anlamda rights ve privileges arasındaki farkları ve bunların bir AD ortamına tam olarak nasıl uygulandığını kesin olarak kavramamız gerekir.


## Built-in AD Groups

AD birçok [varsayılan veya built-in security grubu](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups) içerir, bunlardan bazıları üyelerine bir domain içindeki privilege'leri yükseltmek ve nihayetinde bir Domain Controller (DC) üzerinde Domain Admin veya SYSTEM ayrıcalıkları elde etmek için kötüye kullanılabilecek güçlü rights and privilege'lar verir. Bu grupların birçoğuna üyelik sıkı bir şekilde yönetilmelidir çünkü aşırı grup üyeliği/ayrıcalıkları birçok AD ağında saldırganların kötüye kullanmaya çalıştığı yaygın bir kusurdur. En yaygın built-in gruplardan bazıları aşağıda listelenmiştir.


| **Grup Adı**                           | **Açıklama**                                                                                                                                                                                                                                                                                                                                                                           |
| -------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Account Operators**                  | Üyeler, kullanıcılar, local gruplar ve global gruplar dahil olmak üzere çoğu türdeki hesapları oluşturabilir ve değiştirebilir, ayrıca domain controller'lara local olarak giriş yapabilirler. Administrator hesabını, admin kullanıcı hesaplarını veya Administrators, Server Operators, Account Operators, Backup Operators veya Print Operators gruplarının üyelerini yönetemezler. |
| **Administrators**                     | Üyeler, bir bilgisayar veya bir domain üzerindeki tüm kaynaklara tam ve sınırsız erişime sahiptirler. Domain controller üzerinde bu grupta iseler, domaini yönetebilirler.                                                                                                                                                                                                             |
| **Backup Operators**                   | Üyeler, bir bilgisayardaki tüm dosyaları, dosyaların izinlerine bakılmaksızın yedekleyebilir ve geri yükleyebilirler. Backup Operators, bilgisayara giriş yapabilir ve bilgisayarı kapatabilir. Ayrıca, DC'lere local olarak giriş yapabilirler ve Domain Admins olarak kabul edilmelidirler. SAM/NTDS veritabanının yedeklerini alabilirler.                                          |
| **DnsAdmins**                          | Üyeler, ağ DNS bilgilerine erişime sahiptir. Bu grup yalnızca DNS sunucusu rolü domain controller'da yüklendiğinde veya daha önce yüklendiğinde oluşturulur.                                                                                                                                                                                                                           |
| **Domain Admins**                      | Üyeler, domaini yönetmek için tam erişime sahiptir ve tüm domain’e bağlı makinelerde local administrator grubunun üyeleridirler.                                                                                                                                                                                                                                                       |
| **Domain Computers**                   | Domain içinde oluşturulan tüm bilgisayarlar (domain controller’lar hariç) bu gruba eklenir.                                                                                                                                                                                                                                                                                            |
| **Domain Controllers**                 | Domain içindeki tüm domain controller'ları içerir. Yeni DC'ler otomatik olarak bu gruba eklenir.                                                                                                                                                                                                                                                                                       |
| **Domain Guests**                      | Bu grup, domainin built-in Guest hesabını içerir. Üyeler, bir domain'e bağlı bilgisayara local misafir olarak giriş yaptıklarında domain profili oluşturulmasını sağlar.                                                                                                                                                                                                               |
| **Domain Users**                       | Bu grup, domaindeki tüm kullanıcı hesaplarını içerir. Domainde yeni bir kullanıcı hesabı oluşturulduğunda otomatik olarak bu gruba eklenir.                                                                                                                                                                                                                                            |
| **Enterprise Admins**                  | Bu gruptaki üyelik, domain içinde tam yapılandırma erişimi sağlar. Bu grup yalnızca bir AD forest'ının root domaininde bulunur. Bu gruptaki üyeler, bir child domaini eklemek veya bir trust oluşturmak gibi forest çapında değişiklik yapabilme yeteneğine sahiptir. Root domainin Administrator hesabı, bu gruptaki tek üyedir.                                                      |
| **Event Log Readers**                  | Üyeler, local bilgisayarlarda event loglarını okuyabilirler. Bu grup, bir host domain controller olarak yükseltildiğinde oluşturulur.                                                                                                                                                                                                                                                  |
| **Group Policy Creator Owners**        | Üyeler, domain içinde Group Policy Nesnelerini (GPO) oluşturabilir, düzenleyebilir veya silebilirler.                                                                                                                                                                                                                                                                                  |
| **Hyper-V Administrators**             | Üyeler, Hyper-V'nin tüm özelliklerine tam ve sınırsız erişime sahiptirler. Domain içinde virtual DC'ler varsa, Hyper-V Administrators gibi sanallaştırma yöneticileri Domain Admins olarak kabul edilmelidirler.                                                                                                                                                                       |
| **IIS_IUSRS**                          | Bu, Internet Information Services (IIS) tarafından kullanılan built-in bir gruptur, IIS 7.0 ve sonrasında kullanılmaktadır.                                                                                                                                                                                                                                                            |
| **Pre–Windows 2000 Compatible Access** | Bu grup, Windows NT 4.0 ve önceki sürümleri çalıştıran bilgisayarlar için geriye uyumluluk sağlamak amacıyla vardır. Bu gruptaki üyelik genellikle eski bir yapılandırmanın kalıntısıdır. Bu, ağdaki herkesin geçerli bir AD kullanıcı adı ve şifresi olmadan AD'den bilgi okumasına neden olabilir.                                                                                   |
| **Print Operators**                    | Üyeler, domain controller’lara bağlı yazıcıları yönetebilir, oluşturabilir, paylaşabilir ve silebilirler. Ayrıca AD içindeki yazıcı nesnelerini yönetebilirler. Üyeler, DC'lere local olarak giriş yapabilirler ve kötü amaçlı bir yazıcı driver'ı yükleyerek domain içinde ayrıcalıkları yükseltebilirler.                                                                            |
| **Protected Users**                    | Bu [grubun](https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/active-directory-security-groups#protected-users) üyelerine, kimlik bilgisi çalınmasına ve Kerberos kötüye kullanımına karşı ek korumalar sağlanır.                                                                                                                                   |
| **Read-only Domain Controllers**       | Domain içindeki tüm salt okunur domain controller'ları içerir.                                                                                                                                                                                                                                                                                                                         |
| **Remote Desktop Users**               | Bu grup, kullanıcılara ve gruplara, bir hosta Remote Desktop (RDP) ile bağlanma izni vermek için kullanılır. Bu grup yeniden adlandırılamaz, silinemez veya taşınamaz.                                                                                                                                                                                                                 |
| **Remote Management Users**            | Bu grup, kullanıcılara [Windows Remote Management](https://docs.microsoft.com/en-us/windows/win32/winrm/portal) (WinRM) ile bilgisayarlara remote erişim sağlamak için kullanılabilir.                                                                                                                                                                                                 |
| **Schema Admins**                      | Üyeler, Active Directory şemasını değiştirebilirler; bu, AD içindeki tüm objelerin nasıl tanımlandığını belirler. Bu grup yalnızca AD forest'ının root domaininde bulunur. Root domainin Administrator hesabı, bu gruptaki tek üyedir.                                                                                                                                                 |
| **Server Operators**                   | Bu grup yalnızca domain controller’larda bulunur. Üyeler, domain controller’larda servisleri değiştirebilir, SMB paylaşımlarına erişebilir ve dosya yedekleme işlemleri yapabilirler. Varsayılan olarak, bu grubun üyesi yoktur.                                                                                                                                                       |
Aşağıda, domain adminleri ve server operatörleri ile ilgili bazı çıktılar verilmiştir.


### Server Operators Group Details

```powershell-session
PS C:\htb>  Get-ADGroup -Identity "Server Operators" -Properties *

adminCount                      : 1
CanonicalName                   : INLANEFREIGHT.LOCAL/Builtin/Server Operators
CN                              : Server Operators
Created                         : 10/27/2021 8:14:34 AM
createTimeStamp                 : 10/27/2021 8:14:34 AM
Deleted                         : 
Description                     : Members can administer domain servers
DisplayName                     : 
DistinguishedName               : CN=Server Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
dSCorePropagationData           : {10/28/2021 1:47:52 PM, 10/28/2021 1:44:12 PM, 10/28/2021 1:44:11 PM, 10/27/2021 
                                  8:50:25 AM...}
GroupCategory                   : Security
GroupScope                      : DomainLocal
groupType                       : -2147483643
HomePage                        : 
instanceType                    : 4
isCriticalSystemObject          : True
isDeleted                       : 
LastKnownParent                 : 
ManagedBy                       : 
MemberOf                        : {}
Members                         : {}
Modified                        : 10/28/2021 1:47:52 PM
modifyTimeStamp                 : 10/28/2021 1:47:52 PM
Name                            : Server Operators
nTSecurityDescriptor            : System.DirectoryServices.ActiveDirectorySecurity
ObjectCategory                  : CN=Group,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
ObjectClass                     : group
ObjectGUID                      : 0887487b-7b07-4d85-82aa-40d25526ec17
objectSid                       : S-1-5-32-549
ProtectedFromAccidentalDeletion : False
SamAccountName                  : Server Operators
sAMAccountType                  : 536870912
sDRightsEffective               : 0
SID                             : S-1-5-32-549
SIDHistory                      : {}
systemFlags                     : -1946157056
uSNChanged                      : 228556
uSNCreated                      : 12360
whenChanged                     : 10/28/2021 1:47:52 PM
whenCreated                     : 10/27/2021 8:14:34 AM
```

Yukarıda gördüğümüz gibi, Server Operators grubunun varsayılan durumu hiçbir üyeye sahip olmamaktır ve varsayılan olarak bir domain local grubudur. Buna karşılık, aşağıda görülen Domain Admins grubunun birkaç üyesi ve kendisine atanmış servis hesapları vardır. Domain Admins ayrıca domain local yerine Global gruplardır. Grup üyeliği hakkında daha fazla bilgiyi bu modülün ilerleyen bölümlerinde bulabilirsiniz. Bu gruplara kimlerin erişimine izin verdiğiniz konusunda dikkatli olun. Bir saldırgan, bu gruplara atanmış bir kullanıcıya erişim sağlarsa kuruluşun anahtarlarını kolayca elde edebilir.


#### Domain Admins Group Membership

```powershell-session
PS C:\htb>  Get-ADGroup -Identity "Domain Admins" -Properties * | select DistinguishedName,GroupCategory,GroupScope,Name,Members

DistinguishedName : CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
GroupCategory     : Security
GroupScope        : Global
Name              : Domain Admins
Members           : {CN=htb-student_adm,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=sharepoint
                    admin,CN=Users,DC=INLANEFREIGHT,DC=LOCAL, CN=FREIGHTLOGISTICSUSER,OU=Service
                    Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=PROXYAGENT,OU=Service
                    Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL...}
```


#### User Rights Ataması

Mevcut grup üyeliklerine ve administrator'ların Group Policy (GPO) aracılığıyla atayabileceği privilege'lar gibi diğer faktörlere bağlı olarak, kullanıcılar hesaplarına atanmış çeşitli haklara sahip olabilirler. [User Rights Assignment](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment) (Kullanıcı Hakları Ataması) hakkındaki bu Microsoft makalesi, Windows'ta ayarlanabilen kullanıcı haklarının her biri hakkında ayrıntılı bir açıklama sunmaktadır. Burada listelenen her right (hak) penetrasyon testçileri veya savunucular olarak güvenlik açısından bizim için önemli değildir, ancak bir hesaba verilen bazı rightlar privilege escalation (ayrıcalık yükseltme) veya hassas dosyalara erişim gibi istenmeyen sonuçlara yol açabilir. Örneğin, kontrol ettiğimiz bir veya daha fazla kullanıcıyı içeren bir OU'ya uygulanan bir Group Policy Object (GPO) üzerinde yazma erişimi elde edebileceğimizi varsayalım. Bu örnekte, bir kullanıcıya hedeflenen hakları atamak için [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) gibi bir araçtan yararlanabiliriz. Bu yeni haklarla erişimimizi ilerletmek için domain'de birçok eylem gerçekleştirebiliriz. Birkaç örnek şunlardır:

| **Yetki (Privilege)**             | **Açıklama**                                                                                                                                                                                                                                 |
| --------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **SeRemoteInteractiveLogonRight** | Hedef kullanıcıya Remote Desktop (RDP) aracılığıyla bir host'a giriş yapma hakkı verir. Bu hak, hassas verilerin ele geçirilmesi veya ayrıcalıkların yükseltilmesi için kullanılabilir.                                                      |
| **SeBackupPrivilege**             | Kullanıcıya sistem yedeklemeleri oluşturma yetkisi tanır. Bu yetki, SAM ve SYSTEM **Registry** Registry hives ile **NTDS.dit** gibi dosyaların ele geçirilmesi için kullanılabilir. Bu dosyalar, şifrelerin geri alınmasında kullanılabilir. |
| **SeDebugPrivilege**              | Kullanıcının bir **process**in belleğini hata ayıklama ve düzenleme yeteneği kazanmasını sağlar. Bu yetkiyle, **Mimikatz** gibi araçlarla **LSASS** belleğinden kimlik bilgileri alınabilir.                                                 |
| **SeImpersonatePrivilege**        | Ayrıcalıklı bir hesabın (ör. **NT AUTHORITY\SYSTEM**) **token**ını taklit etme yetkisi verir. Bu, **JuicyPotato**, **RogueWinRM** veya **PrintSpoofer** ile ayrıcalık yükseltmek için kullanılabilir.                                        |
| **SeLoadDriverPrivilege**         | Cihaz driverlarını yükleme ve kaldırma yetkisi verir. Bu yetki, sistemin ele geçirilmesi veya ayrıcalıkların yükseltilmesi için kullanılabilir.                                                                                              |
| **SeTakeOwnershipPrivilege**      | Bir **process**in bir **object**in sahipliğini ele geçirmesine izin verir. Bu yetki, erişim izni olmayan dosya veya paylaşılan kaynaklara erişim sağlamak için kullanılabilir.                                                               |

Kullanıcı haklarını kötüye kullanmak için [burada](https://blog.palantir.com/windows-privilege-abuse-auditing-detection-and-defense-3078a403d74e) ve [burada](https://book.hacktricks.xyz/windows/windows-local-privilege-escalation/privilege-escalation-abusing-tokens) ayrıntıları verilen birçok teknik mevcuttur. Bu modülün kapsamı dışında olsa da, bir kullanıcı hesabına yanlış ayrıcalık atamanın Active Directory'de yaratabileceği etkiyi anlamak önemlidir. Küçük bir admin hatası tüm sistemin veya kurumun tehlikeye girmesine yol açabilir.


### Domain Admin Rights Yükseltilmemiş

Yükseltilmemiş bir konsolda aşağıdakileri görebiliriz, bu da standart domain kullanıcısı için mevcut olandan daha fazlası gibi görünmemektedir. Bunun nedeni, CMD veya PowerShell konsolunu yükseltilmiş bir contexte çalıştırmadığımız sürece Windows sistemlerinin varsayılan olarak bize tüm hakları etkinleştirmemesidir. Bu, her uygulamanın mümkün olan en yüksek ayrıcalıklarla çalışmasını önlemek içindir. Bu, Windows Privilege Escalation modülünde derinlemesine ele alınan [User Account Control (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) adı verilen bir şey tarafından kontrol edilir.

```powershell-session
PS C:\htb> whoami /priv

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


#### Domain Admin Rights Elevated (Yükseltilmiş)

Aynı komutu yükseltilmiş bir PowerShell konsolundan girersek, kullanabileceğimiz hakların tam listesini görebiliriz:

```powershell-session
PS C:\htb> whoami /priv

PRIVILEGES INFORMATION
----------------------

Privilege Name                            Description                                                        State
========================================= ================================================================== ========
SeIncreaseQuotaPrivilege                  Adjust memory quotas for a process                                 Disabled
SeMachineAccountPrivilege                 Add workstations to domain                                         Disabled
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
SeDebugPrivilege                          Debug programs                                                     Enabled
SeSystemEnvironmentPrivilege              Modify firmware environment values                                 Disabled
SeChangeNotifyPrivilege                   Bypass traverse checking                                           Enabled
SeRemoteShutdownPrivilege                 Force shutdown from a remote system                                Disabled
SeUndockPrivilege                         Remove computer from docking station                               Disabled
SeEnableDelegationPrivilege               Enable computer and user accounts to be trusted for delegation     Disabled
SeManageVolumePrivilege                   Perform volume maintenance tasks                                   Disabled
SeImpersonatePrivilege                    Impersonate a client after authentication                          Enabled
SeCreateGlobalPrivilege                   Create global objects                                              Enabled
SeIncreaseWorkingSetPrivilege             Increase a process working set                                     Disabled
SeTimeZonePrivilege                       Change the time zone                                               Disabled
SeCreateSymbolicLinkPrivilege             Create symbolic links                                              Disabled
SeDelegateSessionUserImpersonatePrivilege Obtain an impersonation token for another user in the same session Disabled
```


User rights (Kullanıcı hakları), yerleştirildikleri gruplara veya atanan ayrıcalıklara göre artar. Aşağıda bir ==Backup Operators== grup üyesine verilen hakların bir örneği yer almaktadır. Bu gruptaki kullanıcılar şu anda UAC tarafından kısıtlanan diğer haklara sahiptir (güçlü ==SeBackupPrivilege== gibi ek haklar standart bir konsol oturumunda varsayılan olarak etkinleştirilmez). Yine de, bu komuttan ==SeShutdownPrivilege=='a sahip olduklarını görebiliriz, bu da bir domain controller'ı kapatabilecekleri anlamına gelir. Bu ayrıcalık tek başına hassas verilere erişim sağlamak için kullanılamaz, ancak bir domain controller'da local olarak oturum açmaları durumunda (RDP veya WinRM aracılığıyla remote olarak değil) büyük bir servis kesintisine neden olabilir.


#### Backup Operator Rights

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

Saldırganlar ve savunmacılar olarak, Active Directory'deki built-in security gruplarına üyelik yoluyla kullanıcılara verilen hakları anlamamız gerekir. Domain'e daha fazla erişim sağlamak veya domain'i tehlikeye atmak için kullanılabilecek bu gruplardan birine veya daha fazlasına eklenmiş, görünüşte düşük ayrıcalıklı kullanıcılar bulmak nadir değildir. Bu gruplara erişim sıkı bir şekilde kontrol edilmelidir. Bu grupların çoğunu boş bırakmak ve bir gruba yalnızca tek seferlik bir eylem gerçekleştirilmesi veya tekrarlayan bir görevin ayarlanması gerektiğinde bir hesap eklemek genellikle en iyi uygulamadır. Bu bölümde tartışılan gruplardan birine eklenen veya ekstra ayrıcalıklar verilen tüm hesaplar sıkı bir şekilde kontrol edilmeli ve izlenmeli, çok güçlü bir parola veya parola atanmalı ve bir sistem yöneticisi tarafından günlük görevlerini yerine getirmek için kullanılan bir hesaptan ayrı olmalıdır.

AD'de kullanıcı ayrıcalıkları ve built-in grup üyeliğiyle ilgili bazı güvenlik konularına değinmeye başladığımıza göre, şimdi bir Active Directory kurulumunun güvenliğini sağlamak için bazı kritik noktaların üzerinden geçelim.


Soru : Hangi built-in grubu bir kullanıcıya bir bilgisayara tam ve sınırsız erişim sağlar?

Cevap : Administrators

Soru : Hangi kullanıcı hakkı bir kullanıcıya bir sistemin yedeklerini alma yetkisi verir?

Cevap : SeBackupPrivilege

Soru : Hangi Windows komutu bize mevcut kullanıcıya atanmış tüm kullanıcı haklarını gösterebilir?

Cevap : whoami /priv




### Active Directory'de Güvenlik