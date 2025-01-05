---
{"dg-publish":true,"permalink":"/active-directory-expert/adcs-attacks/"}
---

Dijital güvenliğin dinamik dünyasında, **Active Directory Certificate Services (ADCS)** temel bir teknoloji olarak öne çıkar. **ADCS**, kuruluşların kendi **Public Key Infrastructure (PKI)** yapılarını kurmalarını ve yönetmelerini sağlar. Bu, güvenli iletişim, kullanıcı kimlik doğrulama ve veri koruma için bir temel oluşturur. Bu giriş, **ADCS** dünyasına bir kapı aralayarak **Certificate Authority (CA)**, dijital sertifikalar, **PKI architecture** ve modern ağ güvenliğinin temelini güçlendirmedeki rollerini kapsayan ana unsurları ele alır.

İlk üç bölümde, ADCS ile ilgili temel kavramları ve terminolojiyi keşfetmeye odaklanacağız. Bunu takiben, yanlış yapılandırılmış ADCS servislerinin nasıl numaralandırılacağını ve bunlara nasıl saldırılacağını inceleyeceğiz.


### Public Key Infrastructure (PKI)

Public Key Infrastructure (PKI), İnternet gibi güvenli olmayan ağlar üzerinden güvenli iletişim sağlamak için dijital sertifikalar ve public key kriptografi kullanan bir sistemdir. PKI, elektronik belgelerin, e-posta mesajlarının ve diğer çevrimiçi iletişim biçimlerinin dijital imzalarını, şifrelenmesini ve kimlik doğrulamasını sağlar.

Dijital sertifika, bir public key'i bir kişiye, kuruluşa, cihaza veya hizmete bağlayan elektronik bir belgedir. Sertifika sahibinin kimliğini ve pbulic anahtarın bütünlüğünü doğrulayan güvenilir bir Certificate Authority (CA) tarafından düzenlenir ve imzalanır. Sertifika, public anahtarı, öznenin adını, sertifikayı verenin adını, geçerlilik süresini ve diğer nitelikleri içerir.

PKI'nın Faydaları:

* Gizlilik (Confidentiality): PKI, depolanan veya iletilen verileri şifrelemenize olanak tanır

* Bütünlük (Integrity): Dijital imza, veri iletilirken verinin değiştirilip değiştirilmediğini tanımlar.

* **Authenticity:** Bir mesaj özeti, gönderenin **private key**i kullanılarak dijital olarak imzalanır. Bu özet yalnızca gönderenin karşılık gelen **public key**i ile çözülebileceğinden, mesajın yalnızca gönderen kullanıcıdan gelebileceğini kanıtlar (**inkâr edilemezlik**).


ADCS'nin PKI'ya göre avantajları:

* Active Directory kullanan kurumsal kuruluşlarda sertifika yönetimini ve kimlik doğrulamayı basitleştiren AD DS ile sıkı entegrasyon

* Certificate Revocation List (CRL) ve Online Certificate Status Protocol (OCSP) kullanarak sertifika iptali için built-in destek.

* Adminlerin AD CS tarafından verilen sertifikaların özniteliklerini, uzantılarını ve politikalarını tanımlamasına olanak tanıyan özel sertifika şablonları desteği.

* Birden fazla CA'nın bir hiyerarşide veya load-balanced bir kümede dağıtılmasına olanak tanıyan ölçeklenebilirlik ve yedeklilik.


### What is ADCS?

Active Directory Certificate Services (AD CS), kuruluşların kendi Public Key Infrastructure'larını (PKI) kurmalarını ve yönetmelerini sağlayan bir Windows sunucu rolüdür.

AD CS, bir Windows ağındaki kullanıcıların, bilgisayarların, grupların ve diğer objelerin merkezi bir veritabanı olan Active Directory Domain Services (AD DS) ile bütünleşir.

AD CS, Secure Socket Layer/Transport Layer Security (SSL/TLS), Virtual Private Network (VPN), Remote Desktop Services (RDS) ve Wireless LAN (WLAN) gibi çeşitli ağ hizmetlerinin güvenliğini sağlamak için kullanılabilir. Ayrıca, kullanıcıların ağ kaynaklarına kimliklerini doğrulamak için kullanılabilecek akıllı kartlar ve diğer fiziksel tokenlar için de sertifika verebilir. Akıllı kartta veya tokenda saklanan private key daha sonra kullanıcının ağa kimlik doğrulamasını yapmak için kullanılır.

Active Directory Certificate Services şunları içerir:

1. Digital Certificates
2. Certificate Authority
3. Bağımsız CA veya Kurumsal CA
4. Root CA veya Subordinate CA
5. Sertifika Şablonları
6. Anahtar Çifti Oluşturma
7. Sertifika İptali
8. Güvenli İletişim
9. Dijital İmzalar
10. Şifreleme ve Şifre Çözme
11. Geliştirilmiş Güvenlik ve Kimlik Yönetimi


### Temel ADCS Terminolojisi

Active Directory Certificate Services (ADCS), modern güvenliğin temelini oluşturan kriptografik karmaşıklıkların bir senfonisini düzenler. Bu teknoloji, kuruluşların Public Key Infrastructure (PKI) kurmalarını ve yönetmelerini sağlayarak güvenli iletişim, veri bütünlüğü ve kullanıcı kimlik doğrulamasını kolaylaştırır.

Dijital güvenliğin dinamik ortamında ADCS, güven ve şifreleme konularını sorunsuz bir şekilde bir araya getiren önemli bir oyuncu olarak hizmet vermektedir. Özünde, dijital sertifikaları düzenleyen ve yöneten bir nöbetçi olan Certificate Authority (CA) kavramı yatmaktadır. Bu sertifikalar, bir ağ içindeki kullanıcıların, cihazların veya servislerin gerçekliğine ilişkin kefil olan dijital pasaportlar rolünü oynar.

ADCS, dijital sertifikaların ve private key'lerin verileri güvende ve değiştirilmemiş tutmak için birlikte çalıştığı karmaşık bir koruma prosesini düzenler. Bu teknoloji bir güven ağı oluşturarak farklı tarafların kimliklerinin doğrulandığını ve konuşmalarının yetkisiz gözlemcilerden gizli tutulduğunu bilerek güvenle iletişim kurmalarına olanak tanır.

Dijital güvenlik ortamında gezinmek, Active Directory Sertifika Servisleri (ADCS) temellerini sağlam bir şekilde kavramayı gerektirir. Bu keşif, ADCS kavramlarının gizemini çözmeyi amaçlamaktadır. Bu temel terimleri incelemek, ADCS'yi nasıl kullanacağımızı daha iyi özümsemek için gerekli bileşenleri aydınlatacaktır.



### ADCS'deki Temel Terminolojiler

* `Certificate Templates (Sertifika Şablonları) :` Bu önceden tanımlanmış yapılandırmalar, AD CS tarafından verilen sertifikaların özelliklerini ve kullanımını belirler. Sertifika amacı, anahtar boyutu, geçerlilik süresi ve verme politikaları gibi ayarları kapsar. AD CS standart şablonlar (örneğin, Web Server, Code Signing) sunarken, adminlere belirli iş gereksinimlerini karşılayan özel şablonlar oluşturma yetkisi verir.

* `Public Key Infrastructure (PKI)` : Dijital sertifikaların oluşturulması, yönetilmesi, dağıtılması ve iptal edilmesi için donanım, yazılım, politika ve prosedürleri entegre eden kapsamlı bir sistem. Public Key Cryptography (Açık Anahtar Kriptografisi) aracılığıyla elektronik işlemlerde yer alan varlıkları doğrulayan Certification Authorities (CA'lar) ve kayıt yetkililerini barındırır.

* `Certificate Authority (CA)` : Bu bileşen, sertifika geçerlilik yönetimini denetlerken kullanıcılara, bilgisayarlara ve servislere sertifika verir.

* `Certificate Enrollment (Sertifika Kaydı)`: Kuruluşlar CA'lardan sertifika talep eder ve sertifika verilmeden önce talep edenin kimliğinin doğrulanması gerekir.

* `Certificate Manager (Sertifika Yöneticisi)` : Sertifika verilmesi, yönetimi ve kayıt ve iptal taleplerinin yetkilendirilmesinden sorumludur.

* `Digital Sertifika` : Kullanıcı veya kuruluş adları gibi kimlik bilgilerini ve bunlara karşılık gelen bir public anahtarı barındıran elektronik bir belge. Bu sertifikalar, bir kişinin veya cihazın kimliğini kanıtlayarak kimlik doğrulama işlevi görür

* `Certificate Revocation (Sertifika İptali)` : ADCS, tehlikeye girmeleri veya artık geçerli olmamaları durumunda sertifikaların iptal edilmesini destekler. İptal, Sertifika İptal Listeleri (CRL'ler) veya Online Sertifika Durum Protokolü (OCSP) aracılığıyla yönetilebilir

* `Key Management` : ADCS, private key'leri yönetmek, güvenliklerini ve doğru kullanımlarını sağlamak için mekanizmalar sağlar.

* `Backup Operator (Yedekleme Operatörü)` : Backup operatörü dosyaları ve dizinleri yedekler ve geri yükler. Backup operatörleri Active Directory Kullanıcıları ve Bilgisayarları veya Computer Management kullanılarak atanır. CA bilgileri de dahil olmak üzere sistem durumunu yedekleyebilir ve geri yükleyebilir, AD CS servisini başlatabilir ve durdurabilir, sistem yedekleme kullanıcı hakkına sahip olabilir ve CA veritabanındaki kayıtları ve yapılandırma bilgilerini okuyabilirler.

* `Standalone CA & Enterprise CA` : Standalone CA'lar Active Directory olmadan bağımsız olarak çalışır ve manuel veya web tabanlı sertifika taleplerine izin verir. Buna karşılık, Active Directory'ye bağlı olan Enterprise CA'lar, bir kuruluş içindeki kullanıcılar, cihazlar ve sunucular için sertifikalar düzenler ve Group Policy veya Certificate Enrollment Web Services kullanarak işlemleri otomatikleştirir.

* `Certificate Signing Requests (Sertifika İmzalama İstekleri)` : Sertifika İmzalama İstekleri (CSR'ler), bir sertifika almak için kullanıcılar veya aygıtlar tarafından bir ADCS CA'ya gönderilen isteklerdir. CSR, kullanıcının veya aygıtın public anahtarını ve sertifikanın konu adı ve kullanım amacı gibi diğer tanımlayıcı bilgileri içerir. Bir CSR bir CA'ya gönderildiğinde, CA talep sahibinin kimliğini doğrular ve CSR'nin bütünlüğünü ve geçerliliğini sağlamak için çeşitli kontroller gerçekleştirir. CSR onaylanırsa CA, talep sahibinin public anahtarını kimliğine ve kullanım amacına bağlayan bir dijital sertifika düzenler

* `Certificate Revocation List (Sertifika İptal Listesi) :` İptal edilen sertifikaları kataloglayan bir CA tarafından yayınlanan dijital olarak imzalanmış bir envanter. CRL, CA tarafından geçersiz kılınan sertifikaların ayrıntılarını içerir ve kuruluşların belirli sertifikaların iptal durumunu doğrulayabilmesini sağlar.

* `Extended/Enhanced Key Usages (Genişletilmiş/Geliştirilmiş Anahtar Kullanımları)` : Bir sertifika için yetkili kullanımları tanımlayan sertifika uzantıları. EKU'lar yöneticilerin sertifika kullanımını kod imzalama, e-posta şifreleme veya akıllı kart oturum açma gibi tanımlanmış uygulamalar veya senaryolarla sınırlamasına olanak tanır. AD CS, Server Authentication, Client Authentication ve Code Signing gibi önceden oluşturulmuş EKU'lar sağlayarak yöneticilerin belirli iş gereksinimlerine uygun özel EKU'lar oluşturmasına olanak tanır.



### ADCS Attack Scenario Examples:

Kurumsal bir ortamda, AD CS güvenli iletişim için hayati bir bileşendir. Saldırganlar AD CS'deki güvenlik açıklarından yararlanarak yetkisiz erişim elde edebilir ve kritik kaynakları tehlikeye atabilir. AD CS temel güvenlik servisleri sağlar. Saldırganlar, bütünlüğünü zayıflatmak için yanlış yapılandırmalardan veya zayıf güvenlik uygulamalarından yararlanabilir

- **Senaryo 1: Sertifika Hırsızlığı ve Kötü Amaçlı Kaydetme İşlemleri**
- **Senaryo 2: Hatalı Yapılandırılmış Şablon Üzerinden Yetki Yükseltme**
- **Senaryo 3: Yetkisiz Sertifika Otoritesinin Ele Geçirilmesi**
- **Senaryo 4: Kötü Amaçlı Sertifika Otoritesi Sunucusunun Tanıtılması**
- **Senaryo 5: Zayıf CA Yöneticisi Şifresi**
- **Senaryo 6: CA Sunucusunun Ele Geçirilmesi**



### Introduction to ADCS Attacks

ADCS yanlış yapılandırma yolculuğu, SpectreOps'un ESC1'den ESC8'e kadar değişen yanlış yapılandırmayı inceleyen [Certified Pre-Owned - Abusing Active Directory Certificate Services](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf) başlıklı çığır açan White Paper'ı ile başladı. Bu temel üzerine inşa edilen ek araştırmacılar, güvenlik açığı keşiflerinin kapsamını genişletti. [Certipy](https://github.com/ly4k/Certipy)'nin arkasındaki beyin olan [Oliver Lyak](https://twitter.com/ly4k_), ESC9 ve ESC10 olarak bilinen iki yanlış yapılandırmayı ortaya çıkardı. Daha sonra [Sylvain Heiniger](https://twitter.com/sploutchy) ve [@compasssecurity](https://twitter.com/compasssecurity) ekibi ESC11 olarak adlandırılan başka bir yanlış yapılandırma tespit etti

Bu bölümde sertifikaların ne olduğunu ve kimlik doğrulama için nasıl kullanılabileceğini genel hatlarıyla açıklayacağız.


### Certificates

Sertifika, şifreleme, mesaj imzalama ve kimlik doğrulama gibi amaçlara hizmet eden X.509 formatında dijital olarak imzalanmış bir belgedir. Birden fazla anahtar alanından oluşur:

`Subject` : Sertifika sahibinin kimliği.
`Public Key` : Subject'i ayrı bir private key'e bağlar.
`NotBefore ve NotAfter dates`: Sertifikanın geçerlilik süresini tanımlar.
`Serial Number (Seri Numarası)` : Sertifikayı veren CA tarafından atanan benzersiz bir tanımlayıcıdır.
`Issuer` : Genellikle bir CA olan sertifikayı veren kuruluşu tanımlar.
`SubjectAlternativeName` : Subject ile ilişkili alternatif adları belirtir.
`Basic Constraints (Temel Kısıtlamalar) :` Sertifikanın CA mı yoksa end entity mi olduğunu kullanım kısıtlamalarıyla birlikte belirtir.
`Extended Key Usages (EKUs)` : Sertifika için belirli kullanım senaryolarını tanımlayan obje tanımlayıcıları. Yaygın EKU'lar kod imzalama, dosya sistemlerini şifreleme, güvenli e-posta, client ve server authentication ve smart card logon gibi işlevleri kapsar.
`Signature Algorithm and Signature :` Sertifikayı imzalamak için kullanılan algoritmayı ve sertifika sahibinin imza oluşturma verisiyle atılan imzayı belirtir

Sertifikanın içeriği, bir kimliği (**Subject**) bir anahtar çiftiyle ilişkilendirir ve uygulamaların bu anahtar çiftini işlemlerde kullanarak kullanıcının kimliğinin kanıtı olarak hizmet etmesini sağlar.



### Certificate Authorities

**Certificate Authority (CA)** 'ler, sertifikaların oluşturulmasından sorumlu temel otoriteler olarak görev yapar. Bu sertifikalar, dijital kimliklerin doğrulanmasında, güvenli iletişimlerin sağlanmasında ve ağlar içinde güvenin tesis edilmesinde kritik bir rol oynar.

**Root CA** sertifikası, CA tarafından kendi **private key**i kullanılarak yeni bir sertifika imzalanarak oluşturulur, bu da root CA sertifikasının **self-signed** olduğu anlamına gelir. **ADCS**, sertifikanın **Subject** ve **Issuer** alanlarını CA'nın adıyla, **Basic Constraints** alanını ise **Subject Type=CA** olarak ayarlar. Ayrıca, **NotBefore** ve **NotAfter** alanları varsayılan olarak beş yıl olacak şekilde belirlenir. Bu işlem tamamlandıktan sonra, sistemler **root CA** sertifikasını kendi trust deposuna ekleyerek CA ile güven ilişkisi kurabilir.


**ADCS**, güvenilen root CA sertifikalarını **CN=Public Key Services,CN=Services,CN=Configuration,DC=,DC=** kapsayıcısı altında dört konumda depolar. Bunlar şunlardır:

1. **Certification Authorities Container:**  
    Bu bölüm, AD CS ortamlarında güvenin temelini oluşturan üst düzey root CA sertifikalarını tanımlar. **certificationAuthority** **objectClass** ile temsil edilen **AD objects** olarak her CA'nın sertifika verisi bu kapsayıcıda bulunur. Windows makineleri, bu root CA sertifikalarını evrensel olarak **Trusted Root Certification Authorities** deposuna dahil eder ve bu, sertifika güven doğrulamasının temelini oluşturur.
    
2. **Enrollment Services Container:**  
    **ADCS** içinde etkinleştirilmiş **Enterprise CA**lere ayrılmıştır. Bu bölümde, her **Enterprise CA** için **AD objects** barındırılır. Bu nesneler, **pKIEnrollmentService objectClass**, **cACertificate** verisi, CA'nın DNS'ini tanımlayan **dNSHostName** ve sertifika yapılandırmalarını belirten **certificateTemplates** gibi temel nitelikleri içerir. AD'deki istemciler, sertifika talep etmek için bu **Enterprise CA**lerle etkileşime girer ve sertifika şablonlarında belirtilen ayarlara uyar. **Enterprise CA**ler tarafından verilen sertifikalar, Windows makinelerindeki **Intermediate Certification Authorities** deposuna dağıtılır.
    
3. **NTAuthCertificates AD Object:**  
    Bu öğe, AD'ye kimlik doğrulamada kritik öneme sahip CA sertifikalarını tanımlar. **certificationAuthority objectClass** ile tanımlanan bu nesne, bir dizi güvenilir CA sertifikasını tanımlayan **cACertificate** özelliklerini içerir. AD ağlarındaki Windows cihazları, bu CA'ları **Intermediate Certification Authorities** deposuna entegre eder. AD'ye sertifikalar kullanılarak kimlik doğrulaması yapılması, istemci sertifikalarının **NTAuthCertificates** içinde listelenen CA'lardan biri tarafından imzalanmasını gerektirir.
    
4. **AIA (Authority Information Access) Container:**  
    Orta düzey ve çapraz CA'ları temsil eden **AD objects** barındırır ve sertifika zincirlerinin doğrulanmasına yardımcı olur. Her CA, **certificationAuthority objectClass** ile belirtilir ve kendi sertifikasını temsil eden **cACertificate** verilerini içerir. Bu orta düzey CA'lar, Windows makinelerindeki **Intermediate Certification Authorities** deposuna dağıtılır ve PKI hiyerarşisi içinde sorunsuz sertifika zinciri doğrulaması için kritik öneme sahiptir.



### Certificate Templates

**AD CS Enterprise CA**ler, sertifika ayarlarını oluşturmak için **certificate templates** kullanır. Bu ayarlar, kayıt politikaları, geçerlilik süresi, amaçlanan kullanım, konu (subject) özellikleri ve talep edenin uygunluğunu içerir. Sertifika şablonları, **Certificate Templates** özelliği üzerinden yönetilir ve **pKICertificateTemplate** **objectClass**ına sahip **AD objects** olarak depolanır. Sertifika şablonlarının ayarları, nitelikler aracılığıyla tanımlanırken, kayıt izinleri ve şablon düzenlemeleri güvenlik tanımlayıcıları ile kontrol edilir.

**AD certificate template** nesnesindeki **pKIExtendedKeyUsage** niteliği, sertifikanın izin verilen kullanımlarını etkileyen etkin **OID (Object Identifier)** kümesini barındırır. Bu **EKU OID**leri, **Encrypting File System**, **Code Signing**, **Smart Card Logon**, ve **Client Authentication** gibi işlevsellikleri kapsar. **Microsoft** tarafından sağlanan **PKI Solutions** EKU OID açıklamasında bu işlevsellikler detaylandırılmıştır.

**SpecterOps** araştırmaları, bir sertifikada mevcut olduklarında AD'ye kimlik doğrulama sağlayan **EKU**lere odaklanmıştır. İlk başta yalnızca **Client Authentication OID (1.3.6.1.5.5.7.3.2)**nin bu yeteneği sunduğu düşünülürken, araştırma sonucunda başka OID'lerin de bu işlevi desteklediği tespit edilmiştir:

|**Açıklama**|**OID**|
|---|---|
|**Client Authentication**|1.3.6.1.5.5.7.3.2|
|**PKINIT Client Authentication***|1.3.6.1.5.2.3.4|
|**Smart Card Logon**|1.3.6.1.4.1.311.20.2.2|
|**Any Purpose**|2.5.29.37.0|
|**SubCA (EKU bulunmuyor)**||

**Not:** **1.3.6.1.5.2.3.4 OID**, AD CS dağıtımlarında manuel olarak eklenmesi gerekir, ancak istemci kimlik doğrulaması için kullanılabilir.  

Daha fazla bilgi için **Certificate Templates** bölümünde [**Certified Pre-Owned**](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf) kaynağına başvurabilirsiniz.



### **Enrollment Process**  

**ADCS** üzerinden sertifika almak isteyen istemcilerin bir **enrollment** sürecinden geçmesi gerekir:

1. **Enterprise CA Bulma:**  
    İstemciler ilk adımda, **Enrollment Services container** içindeki nesnelere dayalı olarak bir **Enterprise CA** bulmalıdır.
    
2. **Public-Private Anahtar Çifti Oluşturma ve CSR Hazırlama:**  
    İstemciler bir **public-private key pair** oluşturur ve bir **Certificate Signing Request (CSR)** mesajı hazırlar. Bu mesaj, **public key** ile birlikte sertifika şablonu adı ve sertifikanın konusu gibi çeşitli diğer bilgileri içerir.
    
3. **CSR'yi Private Key ile İmzalama ve Enterprise CA'ya Gönderme:**  
    İstemciler, **CSR**yi kendi **private key**leriyle imzalar ve **Enterprise CA** sunucusuna gönderir.
    
4. **CA, İstemcinin Sertifika Talep Etmeye Yetkili Olduğunu Kontrol Eder:**  
    **CA** sunucusu, istemcinin sertifika talep etmeye yetkili olup olmadığını kontrol eder. Eğer yetkiliyse, **CSR**de belirtilen **certificate template AD object**i arar ve sertifika verilip verilmeyeceğini belirlemek için inceler. **CA**, sertifika şablonu **AD object**inin izinlerinin kimlik doğrulaması yapan hesabın sertifika almasına izin verip vermediğini kontrol eder.
    
5. **CA Sertifikayı Oluşturur, İmzalar ve İstemciye Gönderir:**  
    Eğer izinler uygunsa, **CA**, sertifika şablonunda tanımlı olan **blueprint** ayarlarını (örneğin, **EKU**, kriptografi ayarları ve issuance gereksinimleri) kullanarak bir sertifika oluşturur. Sertifika şablon ayarlarının izin verdiği durumlarda, **CA**, **CSR** içinde sağlanan diğer bilgileri kullanır, sertifikayı kendi **private key**i ile imzalar ve istemciye gönderir.
    
6. **İstemci Sertifikayı Alır:**  
    İstemci, sertifikayı **Windows Certificate Store**a kaydeder ve ilgili işlemlerde kullanır.


### **Sertifika Verme Gereklilikleri**  

Sertifika şablonu ve **Enterprise CA** erişim kontrollerindeki varsayılan kısıtlamaların yanı sıra, sertifika kayıt işlemini yönetmek için sıkça kullanılan iki ek ayar bulunur. Bunlar, **issuance requirements** olarak bilinir ve şunları içerir:

- Yönetici onayı (**manager approval**)
- Yetkilendirilmiş imza sayısı ve uygulama politikası ayarları

Bu ayarlara ADCS sunucusunda erişmek için aşağıdaki adımlar izlenebilir:

1. **Certification Authority** konsolunu çalıştırmak için **certsrv.msc** komutunu çalıştırın.
2. **Certificate Templates** üzerine sağ tıklayın ve **Manage** seçeneğini seçin. Bu işlem, **Certificate Template Console** penceresini açacaktır.
3. İstediğiniz bir şablonun üzerine sağ tıklayın ve **Properties** seçeneğini seçin.
4. Açılan pencerede **Issuance Requirements** sekmesini seçin.

Bu sekmede, sertifika kayıt işlemi için gereken onay ve imza ayarlarını yapılandırabilirsiniz.

![Pasted image 20250101231343.png](/img/user/resimler/Pasted%20image%2020250101231343.png)

**"CA certificate manager approval"** kısıtlaması, AD nesnesinin **msPKI-EnrollmentFlag** özniteliğinde **CT_FLAG_PEND_ALL_REQUESTS (0x2)** bitinin yapılandırılmasını tetikler. Bu durum, şablon temel alınarak yapılan tüm sertifika taleplerinin bekleme durumuna alınmasına neden olur ve bu talepler **certsrv.msc**'deki **"Pending Requests"** bölümünde görülebilir. Sertifikanın verilmesinden önce bir sertifika yöneticisinin onayı veya reddi gereklidir.

İkinci kısıtlama grubu, **This number of authorized signatures** ve **Application policy** ayarlarından oluşur:

- **This number of authorized signatures**: Sertifika talebinin (CSR) **CA** tarafından kabul edilmesi için gereken imza sayısını belirler.
- **Application policy**: CSR imzalama sertifikası için gerekli olan belirli **EKU OID**lerini tanımlar.

### **Sonuç** 

ADCS yanlış yapılandırmalarının keşfi ve analizi, sistemin zayıflıklarını anlamak için daha fazla araştırma ve keşfe yol açtı. Bu modülde, kolay anlaşılabilir bir şekilde kategorize edilmiş geniş bir yanlış yapılandırma yelpazesini ele alacağız.


* ***Sertifika Şablonlarının Kötüye Kullanılması:** Bu kategori, **ESC1**, **ESC2**, **ESC3**, **ESC9** ve **ESC10**'u kapsar ve sertifika şablonlarındaki yanlış yapılandırmalara odaklanır.

* ***CA Yapılandırmasının Kötüye Kullanılması (ESC6):** Bu kategori, **Certificate Authority** yapılandırmasındaki zayıflıkların istismarını içerir.

* ***Erişim Kontrolünün Kötüye Kullanılması (ESC4, ESC5, ESC7):** **Access Control** ile ilgili yanlış yapılandırmaları anlamak önemlidir ve bu yanlış yapılandırmalar, ADCS içindeki potansiyel zayıflıkları vurgular.

* ***NTLM Relay (ESC8, ESC11):** **NTLM relay** yanlış yapılandırmalarının istismar edilmesi kritik bir konudur. **ESC8** ve **ESC11**, ADCS içindeki bu konuyla ilgili detaylara odaklanır.

* ***Çeşitli ADCS Saldırıları:** Bu kategori, **Certifried** ve **PKINIT not Supported** gibi diğer önemli güvenlik açıklarını kapsar ve ADCS içindeki çeşitli saldırı vektörlerine genel bir bakış sunar.

Bu modül, ADCS içindeki bu yanlış yapılandırmayı tanımlamak, kullanmak ve hafifletmek için bilgi ve teknikler kazanmaya odaklanacaktır.


### ADCS Enumeration

Bir kuruluşun altyapısını denetlerken, **Active Directory Certificate Services (AD CS)**'nin varlığını ve yapılandırmasını belirlemek kritik önem taşır. Bazı kuruluşlar **AD CS**'yi dağıtmayı tercih ederken, bazıları tamamen bu hizmetten vazgeçebilir. Bu değişkenlik, denetlenen **Domain** içinde **ADCS** hizmetinin var olup olmadığını araştırmayı zorunlu kılar.

Laboratuvar ortamları veya belirli **Capture The Flag (CTF)** senaryoları gibi bazı ortamlarda, **ADCS** sunucusu **Domain Controller** üzerinde kurulmuş olabilir. Ancak, çoğu durumda kuruluşlar bu hizmeti bağımsız bir sunucuya kurmayı tercih eder. Bununla birlikte, istisnalar da bulunabilir; bu nedenle **ADCS**'nin varlığını ve hangi sunucuda barındırıldığını kesinleştirmek önemlidir.



### Enumeration From Windows

Bir **ADCS** kurulumu için belirleyici bir faktör, built-in **Cert Publishers** grubunun varlığıdır. Bu grup, genellikle **Certificate Authorities**'nin sertifikaları dizine yayınlamasına yetki verir ve genellikle bir **ADCS** sunucusunun varlığını gösterir. Bu, **ADCS** sunucusunun bu grubun üyesi olacağı anlamına gelir. Bunu doğrulamak için **net group**, **net localgroup** veya diğer grup numaralandırma araçlarını kullanabiliriz.


### Cert Publishers grup üyeliğini sorgulama

![Pasted image 20250101231811.png](/img/user/resimler/Pasted%20image%2020250101231811.png)

Alternatif olarak, Public Key Services konteyner yapısını keşfetmek sadece ADCS'nin varlığını ortaya çıkarmakla kalmaz, aynı zamanda yapılandırmasını da detaylandırır. ADCS ile ilgili tüm konteynerler, ==Public Key Services konteynerinin== altındaki yapılandırma adlandırma context'inde bulunur:

![Pasted image 20250101231907.png](/img/user/resimler/Pasted%20image%2020250101231907.png)

Ayrıca, [Certified Pre-Owned - Abusing Active Directory](https://specterops.io/wp-content/uploads/sites/3/2022/06/Certified_Pre-Owned.pdf) Certificate Services adlı White Paper'ı yayınlayan [SpecterOps](https://specterops.io/) ekibi, ADCS üzerinde ESC1 ila ESC8 olarak etiketlenen sekiz saldırı türünün ana hatlarını çizmiştir. Ayrıca, Active Directory Certificate Services (AD CS) içindeki yanlış yapılandırmaları listelemek ve bunlardan yararlanmak için tasarlanmış bir C# aracı olan [Certify](https://github.com/GhostPack/Certify)'ı geliştirdiler.

Certify çalıştırılabilir dosyasını oluşturmak için, [Certify Github](https://github.com/GhostPack/Certify)'daki kodu derlememiz gerekir veya [Flangvik SharpCollection](https://github.com/Flangvik/SharpCollection/blob/master/NetFramework_4.7_x64/Certify.exe) deposunda derlenen binary'yi kullanabiliriz.

Certify.exe kullanarak numaralandırma yapmak için sadece bir domain kullanıcısı ile kimliği doğrulanmış bir oturumdan Certify.exe find dosyasını çalıştırmamız gerekir:


### Enumerate ESC9 from Windows

![Pasted image 20250101232306.png](/img/user/resimler/Pasted%20image%2020250101232306.png)
![Pasted image 20250101232318.png](/img/user/resimler/Pasted%20image%2020250101232318.png)
![Pasted image 20250101232329.png](/img/user/resimler/Pasted%20image%2020250101232329.png)
![Pasted image 20250101232345.png](/img/user/resimler/Pasted%20image%2020250101232345.png)
![Pasted image 20250101232351.png](/img/user/resimler/Pasted%20image%2020250101232351.png)

Not: **Certify.exe** genellikle mevcut oturum contex'inden kimlik bilgilerini alır, bu da belirli kullanıcı ayrıcalıkları gerektiren senaryolarda kullanışlı veya sorunlu olabilir.


### Enumeration from Linux

Linux'tan, ADCS modülünü kullanarak Domain'de ADCS sunucuları olup olmadığını belirlemek için [NetExec](https://github.com/Pennyw0rth/NetExec)'i kullanabiliriz:


### NetExec ADCS enumeration

![Pasted image 20250101233047.png](/img/user/resimler/Pasted%20image%2020250101233047.png)

Ayrıca, **Certify.exe**'nin Linux karşılığı **[Certipy](https://github.com/ly4k/Certipy)**'dir; bu, [Oliver Lyak](https://twitter.com/ly4k_) tarafından oluşturulan bir Python aracıdır ve bir dizi saldırı ve numaralandırma işlemi gerçekleştirmek için kullanılabilir. Bugüne kadar, bu, **[BloodHound](https://github.com/BloodHoundAD/BloodHound)** desteği, davranışı üzerinde kapsamlı kontrol ve birçok (hatta tüm) saldırı senaryosunu destekleyen en iyi numaralandırma (ve istismar) aracıdır.

Certipy'yi kurmak için pip kullanabiliriz:


### Install Certipy with pip

![Pasted image 20250101233230.png](/img/user/resimler/Pasted%20image%2020250101233230.png)

**Certipy**'yi kullanmak için bir domain kullanıcısının kimlik bilgilerini sağlamamız gerekir. Ayrıca, domain IP'sini de ekleyeceğiz, ancak domain ile DNS çözümlemesi yapabiliyorsak bu adımı atlayabiliriz. Son olarak, numaralandırmanın sonucunu görüntülemek için **-stdout** seçeneğini kullanacağız


### ADCS Servisini numaralandırmak için Certipy kullanma

![Pasted image 20250102000104.png](/img/user/resimler/Pasted%20image%2020250102000104.png)
![Pasted image 20250102000119.png](/img/user/resimler/Pasted%20image%2020250102000119.png)
![Pasted image 20250102000133.png](/img/user/resimler/Pasted%20image%2020250102000133.png)
![Pasted image 20250102000142.png](/img/user/resimler/Pasted%20image%2020250102000142.png)
![Pasted image 20250102000156.png](/img/user/resimler/Pasted%20image%2020250102000156.png)
![Pasted image 20250102000218.png](/img/user/resimler/Pasted%20image%2020250102000218.png)
![Pasted image 20250102000230.png](/img/user/resimler/Pasted%20image%2020250102000230.png)
![Pasted image 20250102000236.png](/img/user/resimler/Pasted%20image%2020250102000236.png)


### ESC1

ESC1 ( Escalation 1 ) domain escalation senaryolarının ilkidir; yanlış yapılandırılmış AD CS sertifika templates (şablonları) kullanan bir grup escalation senaryosuna aittir.


### Understanding ESC1

Bu domain yükseltme senaryosunun temel yanlış yapılandırması, sertifika talebinde alternatif bir kullanıcı belirtebilme olasılığına dayanır. Bu, eğer bir sertifika şablonu, sertifika talebini (Certificate Signing Request - **CSR**) yapan kullanıcıdan farklı bir **subjectAltName (SAN)** eklenmesine izin veriyorsa, herhangi bir kullanıcı adına sertifika talep etmemize olanak tanıyabilir.

Örneğin, domain hesabı **BlWasp**'ı ele geçirdiğimizi varsayalım. Bu hesabı kullanarak, sertifika otoritesinin (**CA**) sertifika şablonlarını enumerate edebilir ve alternatif adların (**SAN**) eklenmesine izin verenleri arayabiliriz. Eğer bu tür şablonlar mevcutsa, ele geçirilmiş **BlWasp** hesabının kimlik bilgilerini kullanarak bir sertifika talebinde bulunabilir ve **SAN** alanına istediğimiz başka bir kullanıcıyı (örneğin, **Administrator**) dahil edebiliriz.

Sertifika başarıyla oluşturulduğunda, **ADCS** sunucusu sertifikayı bize geri gönderir. Bu sertifikayı, **SAN** alanında belirtilen kullanıcı olarak kimlik doğrulamak için kullanabiliriz. Böylece, elde edilen sertifikayı kimlik bilgisi olarak kullanarak yetkisiz erişim ve ayrıcalık yükseltmesi gerçekleştirilebilir.

Not: Bu modüldeki çoğu örnekte **ADCS** hizmetinin bir **Domain Controller (DC)** üzerinde barındırıldığı varsayılmıştır, ancak bu hizmetin DC haricinde başka bir sunucuda da konuşlandırılabileceğini unutmayın.

---
 
Daha Detaylı

- **Yanlış Yapılandırma ve Temel Sorun:**  
    Domain yükseltme senaryosunun temelinde, sertifika talebinde bulunan kullanıcının yerine başka bir kullanıcı belirtmeye izin veren yanlış yapılandırma yatmaktadır. Eğer bir sertifika şablonu, **Certificate Signing Request (CSR)** yapan kullanıcıdan farklı bir **subjectAltName (SAN)** eklenmesine izin veriyorsa, bu durum kötüye kullanılabilir. Başka bir deyişle, bir kullanıcı, kendisi dışında başka bir kullanıcı adına sertifika talep edebilir.
    
- **Ele Geçirilmiş Hesabın Kullanılması:**  
    Örneğin, domain üzerindeki bir hesabı (örneğin, **BlWasp**) ele geçirdiğimizi düşünelim. Bu hesap, domain üzerinde belirli haklara sahip bir hesaptır ve bu hakları kullanarak işlem yapabilir. Ele geçirilen bu hesabın kimlik bilgileriyle, domain üzerindeki sertifika otoritesine (**Certificate Authority - CA**) erişim sağlanabilir.
    
- **CA Sertifika Şablonlarının İncelenmesi:**  
    CA üzerinde, mevcut sertifika şablonlarını incelemek için bir **enumeration** işlemi yapılır. Bu işlemde, özellikle **SAN** alanında alternatif adların eklenmesine izin veren sertifika şablonları aranır. Eğer bu tür bir şablon bulunursa, bu bir güvenlik açığı anlamına gelir.
    
- **Sertifika Talebinde Bulunma:**  
    Ele geçirilmiş **BlWasp** hesabının kimlik bilgileri kullanılarak bir sertifika talebi yapılır. Talep sırasında, **SAN** alanına başka bir kullanıcı, örneğin **Administrator**, eklenir. Bu, sertifikanın aslında **Administrator** adına oluşturulmasını sağlar.
    
- **Sertifikanın Geri Gönderilmesi:**  
    Sertifika otoritesi (**ADCS**) tarafından talep edilen sertifika başarıyla oluşturulur ve talebi yapan kişiye geri gönderilir. Bu noktada, ele geçirilen kullanıcı artık bu sertifikaya sahiptir.
    
- **Yetkisiz Erişim ve Ayrıcalık Yükseltmesi:**  
    Geri alınan sertifika, kimlik doğrulama amacıyla kullanılabilir. Bu sertifika, **SAN** alanında belirtilen kullanıcı adına (**Administrator**) kimlik doğrulamak için kullanılır. Bu işlem sonucunda, elde edilen sertifika bir kimlik bilgisi gibi davranır ve saldırgan, yüksek yetkilere sahip kullanıcı olarak sisteme erişim sağlar.
    
- **ADCS'nin Yerleştirildiği Ortamın Önemi:**  
    ADCS genellikle bir **Domain Controller (DC)** üzerinde çalıştırılır, ancak her zaman böyle olmak zorunda değildir. ADCS, farklı bir sunucuda da konuşlandırılabilir. Bu nedenle, saldırının hedefi, sadece DC değil, ADCS hizmetinin çalıştığı herhangi bir sunucu olabilir.

---


### ESC1 Kötüye Kullanım Gereksinimleri

ESC1'i kötüye kullanmak için aşağıdaki koşulların karşılanması gerekir:

1. Enterprise CA, düşük ayrıcalıklı kullanıcılara sertifika kaydı yapma yetkisi verir.
2. Yönetici onayı devre dışı bırakılmalıdır (sosyal mühendislik taktikleri bu güvenlik önlemlerini aşabilir).
3. Herhangi bir yetkili imza gerekmemelidir.
4. Sertifika şablonunun güvenlik tanımlayıcısı, düşük ayrıcalıklı kullanıcıların sertifika talep etmesine izin verecek şekilde aşırı izinli olmalıdır.
5. Sertifika şablonu, kimlik doğrulamayı mümkün kılan EKU'ları tanımlar.
6. Sertifika şablonu, talep sahiplerinin **Certificate Signing Request (CSR)** içinde bir **subjectAltName (SAN)** belirtmesine izin verir.


### ESC1 Enumeration and Attack

ESC1'in Linux ve Windows'tan nasıl numaralandırılacağını ve kötüye kullanılacağını tartışacağız
