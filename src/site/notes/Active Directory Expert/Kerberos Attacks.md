---
{"dg-publish":true,"permalink":"/active-directory-expert/kerberos-attacks/"}
---

## Kerberos Overview

[Kerberos](https://docs.microsoft.com/en-us/windows/win32/secauthn/microsoft-kerberos), kullanıcıların ağ üzerinde **authenticate** olmasına ve bir kez **authenticate** olduktan sonra servislere erişmesine olanak tanıyan bir protokoldür. Kerberos varsayılan olarak port 88’i kullanır ve Windows 2000’den beri **domain** hesapları için varsayılan **authentication** protokolü olmuştur. Bir kullanıcı bilgisayarına giriş yaptığında, Kerberos onu **authenticate** etmek için kullanılır. Kullanıcı ağ üzerindeki bir servise erişmek istediğinde de Kerberos devreye girer. Kerberos sayesinde, bir kullanıcının sürekli olarak şifresini girmesine gerek kalmaz ve sunucunun her kullanıcının şifresini bilmesine gerek olmaz. Bu, **centralized authentication** örneğidir.

Kerberos, **stateless authentication** sağlayan ve **ticket** tabanlı bir protokoldür. Kullanıcının kimlik bilgilerini, erişmeye çalıştığı kaynaklardan etkili bir şekilde ayırarak şifresinin ağ üzerinden iletilmesini önler. Bu, bir [**Zero-knowledge proof**](https://en.wikipedia.org/wiki/Zero-knowledge_proof) protokolüdür. [Kerberos **Key Distribution Center (KDC)**](https://docs.microsoft.com/en-us/windows/win32/secauthn/key-distribution-center) önceki işlemleri kaydetmez; bunun yerine, **Kerberos Ticket Granting Service (TGS)** geçerli bir **Ticket Granting Ticket (TGT)** üzerine çalışır. Eğer bir kullanıcının geçerli bir **TGT**'si varsa, kimliğini kanıtlamış olduğu varsayılır.

"**Zero-knowledge proof**, bir tarafın (kanıtlayan) belirli bir bilgiyi karşı tarafa (doğrulayan) ifşa etmeden bildiğini kanıtlamasına olanak tanıyan bir kriptografik yöntemdir. Bu sayede doğrulayıcı, bilginin doğru olduğuna emin olur ancak bilginin kendisini öğrenmez."


### Basic Understanding

Çok üst düzeyde bakıldığında, bir kullanıcı ağdaki mevcut kaynaklarla etkileşime geçmek istediğinde aşağıdaki işlemler gerçekleşir:

- Öncelikle, merkezi bir sunucudan bir **"identity card"** talep eder.
- Kullanıcı, kim olduğunu kanıtlamak zorundadır ve bunun karşılığında **"identity card"**, yani [**Ticket Granting Ticket (TGT)**](https://docs.microsoft.com/en-us/windows/win32/secauthn/ticket-granting-tickets) alır.
- Kullanıcı bir servise erişmek istediğinde bu **TGT**'yi sunar. Bu nedenle, her servis erişim isteğinde bu kimliği gösterir ve eğer geçerliyse, merkezi sunucu, istenen kaynağa sunulacak geçici bir **ticket** sağlar.
- Bu geçici **ticket**, kullanıcının adı, grup üyelikleri vb. gibi tüm bilgilerini içerir.
- Kaynak, bu **ticket**'i alır ve eğer kullanıcı yetkiliyse, servislerine erişim izni verir.

Bu süreç iki aşamada gerçekleşir. İlk olarak, kullanıcının **TGT**'sini belirlemek için bir **ticket** talebi yapılır, ardından **Ticket Granting Service (TGS) ticket** veya **Service Ticket (ST)** kullanılarak servislere erişim isteğinde bulunulur.

Not: **Ticket Granting Service (TGS)**, **Key Distribution Center (KDC)**'nin bir bileşenidir ve servis **ticket**'larını dağıtmaktan sorumludur.

Not: Bu konu boyunca **TGS ticket** terimini kullandığımızda, aslında **Service Ticket (ST)**'den bahsediyormuşuz gibi düşünebilirsiniz.


### Kerberos Avantajları

Kerberos saldırıları ve **Golden Ticket attack** gibi tehditler hakkında konuşulduğunda, Kerberos'un zayıf bir **authentication** protokolü olduğu düşünülebilir. Ancak, Kerberos'tan önce **authentication**, `SMB / NTLM` üzerinden gerçekleşiyordu ve kullanıcının **hash** değeri, **authenticate** olduktan sonra bellekte saklanıyordu. Hedef makine ele geçirilip **NTLM hash** çalındığında, saldırgan `Pass-The-Hash attack` ile kullanıcının yetkili olduğu her şeye erişebilirdi. Daha önce belirtildiği gibi, **Kerberos ticket**'ları kullanıcının şifresini içermez ve **ticket**'ın erişim verdiği makineyi belirtir.

Bu nedenle, **WinRM** üzerinden makineler uzaktan erişildiğinde **Double Hop Problem** ortaya çıkar. **Non-Kerberos** bir protokol kullanılarak bir makineye uzaktan erişildiğinde, **NTLM password hash** oturuma bağlı olduğundan, aynı bağlantı kullanılarak yeniden **authenticate** olmadan başka makinelere erişmek mümkündür. Ancak, **Kerberos authentication** ile her erişilmek istenen makine için kimlik bilgileri özel olmalıdır, çünkü herhangi bir şifre bulunmaz. Bu konu hakkında daha fazla bilgi için **Active Directory Enumeration & Attacks** modülündeki **Kerberos "Double Hop" Problem** bölümüne bakabilirsiniz.

Bu nedenle, **`WinRM`** üzerinden makineler **remote** erişildiğinde [**Double Hop Problem**](https://posts.specterops.io/offensive-lateral-movement-1744ae62b14f?gi=f925425e7a42) ortaya çıkar. **Non-Kerberos** bir protokol kullanılarak bir makineye **remote** erişildiğinde, **NTLM password hash** oturuma bağlı olduğundan, aynı bağlantı kullanılarak yeniden **authenticate** olmadan başka makinelere erişmek mümkündür. Ancak, **Kerberos authentication** ile her erişilmek istenen makine için kimlik bilgileri özel olmalıdır, çünkü herhangi bir şifre bulunmaz. 

Diyelim ki, aktif oturumlara sahip bir makine **Kerberos** ile **authenticate** edilmiştir. Bu durumda, **`Pass-The-Ticket attack`** gerçekleştirmek mümkündür. Ancak, **`Pass-The-Hash`**'ten farklı olarak, saldırgan sadece hedef kullanıcının **authenticate** olduğu kaynaklara erişimle sınırlıdır. Ayrıca, bu **ticket**'ların bir ömrü vardır, yani saldırganın kaynağa erişmek ve süreklilik sağlamak için sınırlı bir zaman penceresi vardır.

Aşağıdaki bölüm, **Kerberos authentication** sürecini ayrıntılı olarak açıklar, **ticket**'ların nasıl korunduğu ve içerdiği bilgiler hakkında bilgi verir. Bu süreci anlamak, **Kerberos** ile ilgili saldırılara geçmeden önce çok önemlidir. **Kerberos**'un bir **Active Directory** ortamında **lateral movement**, **privilege escalation** ve **persistence** sağlama için kötüye kullanılabileceği birçok yol vardır. Bu tekniklerin çoğuyla, **penetration test** ve **red team** değerlendirmelerimiz sırasında karşılaşacağız. **Security practitioners** olarak, **Kerberos**'un nasıl çalıştığını ve nasıl faydamıza kötüye kullanılabileceğini derinlemesine anlamalıyız. Ayrıca, **Kerberos** saldırılarının nasıl çalıştığını ve müşterilerin bunlardan nasıl korunabileceklerini, ayrıca **Kerberos abuse**'u tespit etmek için doğru **monitoring**'i nasıl kuracaklarını açıklamak da çok önemlidir.


### Kerberos Authentication Process

**Kerberos context**'inde, bir kullanıcı bir servise erişmek istediğinde üç varlık vardır: `user`, `servis` ve **`authentication server`** olarak da bilinen **Key Distribution Center (KDC)**.

**KDC**, tüm hesapların kimlik bilgilerini bilen varlıktır.

![Pasted image 20250215171009.png](/img/user/Pasted%20image%2020250215171009.png)

**Kerberos protocol**'ünün nasıl çalıştığını tam olarak anlamak ve ne için kullanıldığını görmek üzere detaylara ineceğiz.


### Why Kerberos?

Kendimize sorabileceğimiz ilk soru, bu protokol ne için kullanılır?

Bir yandan, **authentication**'ı merkezi hale getirerek, tüm servislerin her kullanıcının kimlik bilgilerini bilmesini engellemeye yarar. Bu, kullanıcıların düzenli olarak güncellenmesi gereken bir contex'de son derece pratik bir çözümdür; örneğin, şifre değişikliği, yeni bir kullanıcının eklenmesi veya bir kullanıcının devre dışı bırakılması ya da silinmesi gibi durumlar. Eğer tüm servislerin tüm kullanıcıların durumunu bilmesi gerekseydi, bu büyük bir karmaşıklık yaratırdı. Bunun yerine, yalnızca bir varlık, **KDC**, mevcut kullanıcıların güncel bir listesini tutmalıdır.

Diğer yandan, bu protokol, kullanıcıların **password** göndermeden servislere **authenticate** olmalarını sağlar. Bu, **man-in-the-middle** (diğer adıyla **on-path**) saldırılarına karşı mükemmel bir güvenlik önlemidir.


## High-level Genel Bakış 

### Tickets

Her iki ihtiyacı karşılamak için **Kerberos**, private key'ler ve bir **ticket** mekanizması kullanır. private key'ler, pratikte **Active Directory** ortamında, farklı hesapların şifreleri (ya da en azından bu şifrelerin **hash**) olarak kullanılır.

Yüksek seviyede bir perspektiften, bir kullanıcının bir servise nasıl erişebileceği şu şekildedir:

1. İlk olarak, kullanıcı, kim olduğunu kanıtlamak için **key server**'dan (yani **KDC**) ilk **ticket**'ı talep eder. Bu, **client**'ın **KDC**'ye **`authenticate`** olduğu andır. Bu **ticket**, **TGT** (Ticket Granting Ticket), kullanıcının kimlik kartıdır. Kullanıcı hakkında, isim, hesap oluşturulma tarihi, güvenlik bilgileri, kullanıcının ait olduğu gruplar gibi tüm bilgileri içerir. Bu kimlik kartı, yani **TGT**, varsayılan olarak birkaç saat ile sınırlıdır. Bu **ticket**, **KDC**'ye yapılan tüm diğer isteklerde sunulur.
    
2. **TGT** alındıktan sonra, kullanıcı, her seferinde bir servise erişmesi gerektiğinde bunu **KDC**'ye sunar. **KDC**, sunulan **TGT**'nin geçerliliğini ve kullanıcının bunu sahte olarak oluşturmadığını doğrular ve eğer doğruysa, kullanıcıya bir **Ticket Granting Service (TGS)** **ticket**'ı ya da **Service Ticket (ST)** verir. Kullanıcının **TGT**'deki bilgileri, **TGS** **ticket**'ında yer alır.
    
3. Artık kullanıcı, belirli bir servis için bir **TGS ticket**'ına sahip olduğunda, bu **TGS ticket**'ını servise sunar. Servis, bu **ticket**'ın geçerliliğini kontrol eder ve her şey yolundaysa, kullanıcının bilgilerini okuyarak, kullanıcının talep edilen servisi kullanma yetkisi olup olmadığını belirler. Dolayısıyla, `service, kullanıcının erişim haklarını kontrol eder`.

![Pasted image 20250215171650.png](/img/user/Pasted%20image%2020250215171650.png)


### Ticket Protection

Bu diyagram sayesinde, farklı değişimlerin nasıl gerçekleştiğini iyi bir şekilde anlıyoruz. Ancak, yukarıda açıklandığı gibi, **TGT** ve **TGS ticket**'ı, kullanıcıya ait tüm bilgileri içerir. **Service**, bu bilgiyi kullanarak kullanıcının erişim haklarını doğrular.

KDC tarafından sağlanan bu bilgilerin korunması gerekir. Kullanıcı, bu bilgiyi sahte olarak oluşturamamalıdır. İşte burada **encryption keys** devreye girer.

Her hesabın bir şifresi veya gizlisi vardır, bu da bir **encryption** ve **decryption key** olarak işlev görür. **KDC**, tüm kullanıcıların **keys**'ini bilir. **Tickets**'ları korumak için bu **keys**'ler şu şekilde kullanılır:

1. **KDC**'nin kullanıcıya gönderdiği **`TGT`**, sadece `**KDC**'nin bildiği private key` ile şifrelenir. Böylece kullanıcı, kendi hakkında olan bilgileri okuyamaz veya değiştiremez. **KDC** kendisi bu bilgiyi korur.
    
2. **KDC**'nin kullanıcıya gönderdiği **`TGS ticket`**, `service'in private key'`iyle şifrelenir. Aynı şekilde, kullanıcı **service key**'ini bilmediği için, **`TGS ticket`**'ındaki bilgileri değiştiremez. Diğer taraftan, bu **TGS ticket**'ını servise gönderdiğinde, servis, **ticket**'ın içeriğini şifre çözerek kullanıcının bilgilerini okuyabilir.

![Pasted image 20250215172532.png](/img/user/Pasted%20image%2020250215172532.png)


### Teknik Detaylar

Şimdi, kimlik doğrulama sürecinin nasıl bir araya geldiğini ve karşılaştığı çeşitli saldırılara karşı kullanılan koruma mekanizmalarını anlamak için detaylara ineceğiz. Bir servise erişimin üç aşamada gerçekleştirildiğini gördük. Bu aşamalar şu şekilde adlandırılır:

1. **TGT** isteği: **Authentication Service (AS)**
2. **TGS** isteği: **Ticket-Granting Service (TGS)**
3. **Service** isteği: **Application Request (AP)**

**Client** her aşamada bir istek gönderir ve sunucu yanıt verir. Bu üç değişimin nasıl çalıştığını, **tickets**'ların nasıl korunduğunu ve hangi **key** ile korunduğunu açıklayacağız.



### Authentication Service (AS)

#### Request (AS-REQ)

İlk olarak, kullanıcı bir **TGT** (ya da kimlik kartı) isteği yapar. Bu isteğe **`AS-REQ`** denir. Ancak **TGT** almak için, kimliklerini kanıtlamaları gerekmektedir. Bu istek, **KDC**'ye yapılır (bu, **Active Directory** ortamında **Domain Controller**'dır). **KDC**, tüm kullanıcı anahtarlarını tutar.

Kimliklerini kanıtlamak için kullanıcı, bir **`authenticator`** gönderir. Bu, kullanıcının kendi **private key**'iyle şifreleyeceği mevcut **time stamp**'tir. Kullanıcı adı da açık metin olarak gönderilir, böylece **KDC** kiminle işlem yapıldığını anlayabilir.

![Pasted image 20250215173404.png](/img/user/Pasted%20image%2020250215173404.png)

Bu isteği aldıktan sonra, **KDC** kullanıcı adını alacak, ilişkili anahtarı dizininde arayacak ve **authenticator**'ı şifre çözüp çözmediğini kontrol edecektir. Eğer başarılı olursa, bu, kullanıcının kendi **private key**'ini veritabanında kayıtlı olanla aynı şekilde kullandığı anlamına gelir ve kimlik doğrulaması yapılmış olur. Aksi takdirde, kimlik doğrulaması başarısız olur.

Bu adım,  `pre-authentication` (**önceden kimlik doğrulama**) olarak adlandırılır, zorunlu değildir ancak tüm hesaplar `varsayılan olarak` bunu yapmak zorundadır. Ancak, bir **administrator**'ın önceden kimlik doğrulamayı devre dışı bırakabileceğini belirtmek gerekir. Bu durumda, **client** artık bir **authenticator** göndermek zorunda değildir. **KDC** her durumda **TGT**'yi gönderecektir. Bunu başka bir bölümde ele alacağız.


### Response (AS-REP)

**KDC**, dolayısıyla **client**'ın **TGT** talebini almıştır. Eğer **KDC** **authenticator**'ı başarıyla çözebiliyorsa (veya eğer **client** için önceden kimlik doğrulama devre dışı bırakılmışsa), kullanıcıya **AS-REP** adlı bir yanıt gönderir.

Kalan tüm değiş tokuşları korumak için, **KDC** kullanıcıya yanıt vermeden önce `geçici bir session key` oluşturur. **Client**, bu anahtarı sonraki değiş tokuşlar için kullanacaktır. **KDC**, tüm bilgileri kullanıcının anahtarı ile şifrelemekten kaçınır. Daha önce belirttiğimiz gibi, **Kerberos** stateless bir protokoldür, bu yüzden **KDC** bu **session key**'i herhangi bir yerde saklamaz.

**AS-REP** yanıtında bulacağımız iki öğe vardır:

1. İlk olarak, kullanıcının talep ettiği **TGT**'yi bekliyoruz. Bu, tüm kullanıcının bilgilerini içerir ve **KDC**'nin anahtarı ile korunur, böylece kullanıcı bunu değiştiremez. Ayrıca, `oluşturulan` **`session key`**'in bir `kopyasını` da içerir.
    
2. İkinci olarak, **`session key`** vardır, ancak bu sefer `kullanıcının anahtarı` ile korunmuş bir şekilde.

![Pasted image 20250215173834.png](/img/user/Pasted%20image%2020250215173834.png)

Bu nedenle, bu **session key** yanıt içinde iki kez yer alır—bir versiyonu **KDC**'nin anahtarı ile korunur, diğeri ise kullanıcının anahtarı ile korunur.


### Ticket-Granting Service (TGS)

**Ticket-Granting Service (TGS)**, **Key Distribution Center (KDC)**'nin bir bileşenidir ve **service** **tickets**'lerini vermekten sorumludur.

Genellikle, **Active Directory** alanındaki bir **domain controller** üzerinde barındırılır. Bir **client** veya bilgisayar bir **service ticket** talep ettiğinde, istek **KDC**'nin **TGS** bileşenine gönderilir. **TGS**, kullanıcının veya bilgisayarın kimliğini doğrular ve istenilen kaynağa erişim iznini kontrol eder. Ardından, kaynağa erişim sağlamak için kullanılabilecek bir **service ticket** verir.


### Request (TGS-REQ)

**Client**, artık **TGT** talebine karşılık olarak **server**'dan bir yanıt almıştır. Bu yanıt, **KDC**'nin anahtarıyla korunmuş olan **TGT** ve **client**'ın/kullanıcının anahtarıyla korunmuş bir **session key** içerir. **Client**, bu bilgiyi çözebilir ve geçici **session key**'i çıkarabilir.

Kullanıcının bir sonraki adımı, bir **Service Ticket (ST)** veya **TGS ticket** talep etmek için **TGS-REQ** mesajı göndermektir. Bunu yapmak için **KDC**'ye üç şey iletecektir:

1. Erişmek istedikleri **service**'in adı (**SERVICE/HOST**, bu **Service Principal Name (SPN)** temsilidir)
2. Önceden aldıkları **TGT**, içinde kullanıcı bilgileri ve **session key**'in bir kopyası bulunan
3. Bu noktada **session key** kullanılarak şifrelenmiş bir **authenticator**

![Pasted image 20250215175149.png](/img/user/Pasted%20image%2020250215175149.png)


### Response (TGS-REP)

**KDC**, bu **TGS** talebini aldığında, **Kerberos** bir **stateless** protokol olduğu için, daha önce hangi bilgilerin değiş tokuş edildiği hakkında hiçbir fikri yoktur. Yine de, **TGS** talebinin geçerliliğini doğrulamalıdır. Bunu yapmak için **authenticator**'ın doğru **session key** ile şifrelendiğini doğrulamalıdır. Peki, **KDC** kullanılan **session key**'in doğru olduğunu nasıl bilecek? Hatırlayın ki, **TGT** içinde bir **session key** kopyası vardı. **KDC**, **TGT**'yi çözerek (bu sırada **TGT**'nin doğruluğunu kontrol ederek) **session key**'i çıkaracak. Bu **session key** ile **authenticator**'ın geçerliliğini doğrulayabilecektir.

Eğer bu işlemler doğru şekilde yapılırsa, **KDC** sadece talep edilen **service**'i okumalı ve **TGS-REP** mesajı ile kullanıcıya yanıt vermelidir. Daha önce **user** ile **KDC** arasındaki değiş tokuşlar için bir **session key** üretildiğini görmüştük. İşte burada da aynı şey geçerlidir. Kullanıcı ile **service** arasındaki gelecekteki değiş tokuşlar için yeni bir **session key** oluşturulur. Ve daha önce olduğu gibi, bu **session key** **KDC**'nin kullanıcıya gönderdiği yanıtta iki yerde bulunacaktır. İşte **KDC** tarafından gönderilen tüm öğeler:

Bir **service ticket** veya **TGS ticket** içeren üç öğe:

1. Talep edilen **service**'in adı (SPN'si)
2. **TGT**'de bulunan kullanıcının bilgileri kopyası. **Service**, bu bilgileri okuyarak kullanıcının bu **service**'i kullanmaya yetkili olup olmadığını belirleyecektir.
3. **Session key** kopyası

Tüm bu bilgiler, **user/KDC session key** ile şifrelenmiştir. Bu şifreli yanıt içinde, kullanıcının bilgileri ve **user/service session key** kopyası, **service key** ile şifrelenmiştir. Bu durumu daha iyi anlamak için bir diyagram yardımcı olacaktır.


## Application Request (AP)

### Request (AP-REQ)

Kullanıcı, şimdi bu yanıtı çözerek **user/service session key**'i ve **TGS ticket**'ı çıkarabilir, ancak **TGS ticket**'ı **service key** ile korunmaktadır. Kullanıcı bu **TGS ticket**'ı değiştiremez, bu yüzden haklarını değiştiremez, tıpkı **TGT**'de olduğu gibi.

Kullanıcı sadece bu **TGS ticket**'ı **service**'e iletecek ve tıpkı **TGS request**'te olduğu gibi bir **authenticator** ekleyecektir. Kullanıcı bu **authenticator**'ı neyle şifreleyecek? Tahmin ettiğiniz gibi, yeni çıkardığı **user/service session key** ile. Bu süreç, önceki **TGS request**'iyle oldukça benzerdir.

![Pasted image 20250215175652.png](/img/user/Pasted%20image%2020250215175652.png)



### Response (AP-REP)

Service nihayet **TGS ticket**'ını ve **user/service session key** ile şifrelenmiş bir **authenticator**'ı alır. Bu **TGS ticket**'ı **service key** ile korunmaktadır, böylece service bunu çözebilir. Unutmayın ki **user/service session key**'in bir kopyası **TGS ticket**'ına gömülü olduğu için, bu anahtarı çıkarabilir ve **authenticator**'ın geçerliliğini bu oturum anahtarıyla kontrol edebilir.

Her şey düzgün giderse, service nihayet kullanıcının bilgilerini okuyabilir, hangi gruplara ait olduğunu öğrenebilir ve erişim kurallarına göre kullanıcıya service'e erişim izni verebilir veya reddedebilir. Kimlik doğrulama başarılı olursa, service, **AP-REP** mesajıyla client'e yanıt verir ve **timestamp**'i çıkarılan oturum anahtarıyla şifreler. Client, bu mesajın service'ten geldiğini doğrulayabilir ve service isteklerini göndermeye başlayabilir.


Gördüğünüz gibi, tüm süreç paylaşılan anahtarlara dayanır ve üçlü bir iş birliğine dayanır. Bu, kullanıcıları ve servisleri **ticket** çalınması ve yeniden oynatılmasına karşı korur, çünkü saldırganlar geçerli **authenticator**'lar oluşturmak için anahtarları bilmeyeceklerdir. Ancak, yine de Kerberos'u saldırmak için kullanabileceğimiz bazı zayıflıklar ve yanlış yapılandırmalar vardır, bunları sonraki bölümlerde ele alacağız.  

Kerberos protokolü, işleyişi ve bileşenleri hakkında daha fazla açıklama yapmak isterseniz, [ATTL4S](https://twitter.com/DaniLJ94)'in [blogundaki](https://attl4s.github.io/) ve YouTube kanalındaki videolarını gözden geçirebilirsiniz: 

[You Do (Not) Understand Kerberos.](https://www.youtube.com/watch?v=4LDpb1R3Ghg&list=PLwb6et4T42wyb8dx-LQAA0PzLjw6ZlrXh)


## Kerberos Attacks Genel Bakış

Artık Kerberos'un temel ilkelerini ele aldığımıza göre, bu protokolün sunduğu belirli zayıflıkların veya fırsatların sömürülmesine ve analizine geçeceğiz.

Örneğin, **ticket** talepleri, **ticket** sahteciliği ve delegasyonla ilgili saldırıları vurgulayacağız. Ayrıca, bu protokolü kullanarak kullanıcı keşfi yapmanın ve hatta bazı hesapların parolalarını bulmak için **password spraying** yapmanın mümkün olduğunu göreceğiz.


### Ticket Request Attacks

Bir **ticket** talep etmenin iki yolu vardır:

- **TGT** talebi, yani **AS-REQ**, buna karşılık olarak **KDC** bir **AS-REP** yanıtı gönderir.
- **TGS** talebi, yani **TGS-REQ**, buna karşılık olarak **KDC** bir **TGS-REP** yanıtı gönderir.



### AS-REQ Roasting

Bir **TGT** talep ederken (**AS-REQ**), varsayılan olarak bir kullanıcının kimlik doğrulaması, kendi **secret** anahtarıyla şifrelenmiş bir **authenticator** ile yapılmalıdır. Ancak, eğer bir kullanıcının **preauthentication** özelliği devre dışı bırakılmışsa, o kullanıcı için kimlik doğrulama verisi talep edebiliriz ve **KDC** bir **AS-REP** mesajı döndürecektir. Bu mesajın bir kısmı (paylaşılan geçici oturum anahtarı) kullanıcının şifresiyle şifrelenmiş olduğundan, kullanıcının şifresini elde etmeye çalışmak için çevrimdışı bir **brute-force** saldırısı gerçekleştirmek mümkündür.


### Kerberoasting

Benzer şekilde, bir kullanıcı bir **TGT**'ye sahip olduğunda, mevcut herhangi bir **servis** için bir **Service Ticket** talep edebilir. **KDC** yanıtı (**TGS-REP**), **servis** hesabının **secret** anahtarıyla şifrelenmiş bilgileri içerir. Eğer **servis** hesabının şifresi zayıfsa, aynı çevrimdışı saldırıyı gerçekleştirerek o hesabın şifresini elde etmek mümkündür.


### **Kerberos Delegation Saldırıları**  

Kerberos Delegation, bir servisin bir kullanıcıyı taklit ederek başka bir kaynağa erişmesine izin verir. Kimlik doğrulama devredilir ve nihai kaynak, servise ilk kullanıcının haklarına sahipmiş gibi yanıt verir. Farklı türlerde delegasyon vardır ve her birinin, bir saldırganın (bazen keyfi) kullanıcıları taklit etmesine ve diğer servislere erişim sağlamak için kullanmasına olanak tanıyabilecek zayıflıkları bulunmaktadır. Kerberos'u kötüye kullanan şu saldırıları inceleyeceğiz: **`unconstrained delegation`**, **`constrained delegation`** ve **`resource-based constrained delegation`**.


### **Ticket Sahteciliği Saldırıları**  

Ticket'lar, keyfi ticket'ların sahteciliğini engellemek için secret anahtarlarla korunur (TGT, KDC anahtarıyla ve TGS ticket'ı servis hesabı anahtarıyla korunur). Bir saldırgan bu anahtarlardan birine sahip olursa, keyfi ticket'lar oluşturarak, keyfi haklarla servislere erişebilir. Aşağıdaki bölümler, bu iki saldırıyı açıklar: **Silver Ticket** (TGS sahteciliği) ve **Golden Ticket** (TGT sahteciliği).

### **Recon ve Password Spraying**  

Son olarak, Kerberos protokolü kullanılarak kullanıcılar üzerinde keşif yapılabilir ve şifreler test edilebilir. Belirli bir kullanıcı için bir TGT talep edersek, sunucu, kullanıcının veritabanında olup olmamasına bağlı olarak farklı bir şekilde yanıt verecektir. Aynı şekilde, authenticator'ı farklı şifrelerle şifreleyerek, seçilen kullanıcı için şifrenin geçerli olup olmadığını kontrol etmek mümkündür.


Kerberos kimlik doğrulama sürecinin ayrıntılarına ve buna karşı yapılabilecek saldırılara kısa bir genel bakış yaptıktan sonra, her bir saldırıyı tek tek inceleyelim ve bunları ekli laboratuvarlarda deneyelim.


### AS-REPRoasting

**AS-REPRoasting**, en temel Kerberos saldırısıdır ve **Pre-Authentication** mekanizmasını hedef alır. Bu durum bir organizasyonda nadir görülse de, herhangi bir ön kimlik doğrulama gerektirmeyen az sayıdaki Kerberos saldırılarından biridir. Saldırganın ihtiyacı olan tek bilgi, hedef aldığı kullanıcının **username** bilgisidir ve bu bilgi, diğer keşif teknikleriyle de bulunabilir.

Saldırgan **username** bilgisine sahip olduğunda, KDC'ye (**Key Distribution Center**) özel bir **AS_REQ** (**Authentication Service Request**) paketi göndererek ilgili kullanıcı gibi davranır. KDC ise bir **AS_REP** yanıtı döndürür. Bu yanıt, kullanıcının **password** bilgisinden türetilmiş bir anahtar ile şifrelenmiş bir veri içerir. Saldırgan, bu veriyi çevrimdışı olarak brute-force veya wordlist kullanarak kırarak kullanıcının **password** bilgisine ulaşabilir.


### Nasıl Çalışıyor?

**TGT** (**Ticket Granting Ticket**) talepleri, mevcut **timestamp** ve hesabın **password** bilgisi ile şifrelenir. **Domain Controller**, doğru **password** kullanıldığını doğrulamak için bu veriyi çözer. Eğer doğrulama başarılı olursa, **AS-REP** yanıtı ile kullanıcıya bir **TGT** verilir ve bu ticket, domaine yönelik sonraki kimlik doğrulama isteklerinde kullanılır.

Ayrıca, **TGT** ile birlikte bir **session key** de sağlanır ve bu anahtar kullanıcının **password** bilgisi ile şifrelenmiş şekilde iletilir.

![Pasted image 20250215192322.png](/img/user/Pasted%20image%2020250215192322.png)

Eğer bir hesapta **pre-authentication** devre dışı bırakılmışsa, bir saldırgan herhangi bir ön kimlik doğrulama gerektirmeden ilgili hesap için şifrelenmiş bir **TGT** elde edebilir. Bu **ticket**'lar, **Hashcat** veya **John the Ripper** gibi araçlar kullanılarak **offline password attack**'lara karşı savunmasızdır.

Özetle, "**Do not require Kerberos preauthentication**" ayarının etkin olduğu herhangi bir hesap için **Ticket Granting Ticket (TGT)** elde etmek mümkündür.

![Pasted image 20250215192409.png](/img/user/Pasted%20image%2020250215192409.png)

Birçok **vendor installation guide**, **service account**'ın bu şekilde yapılandırılmasını belirtir. **Authentication service reply (AS-REP)**, hesabın şifresiyle şifrelenir ve ağdaki herhangi biri bunu talep edebilir.

**`AS-REPRoasting`**, **`Kerberoasting`** ile benzerdir, ancak **`TGS-REP`** yerine **`AS-REP`**'e yönelik bir saldırıdır.

Bu ayar, **Impacket**, **PowerView** veya **PowerShell AD module** gibi **built-in tool**'lar kullanılarak **enumerate** edilebilir.

Saldırı, **[Impacket](https://github.com/SecureAuthCorp/impacket)**, **[Rubeus](https://github.com/GhostPack/Rubeus) toolkit** ve diğer araçlar kullanılarak hedef hesaba ait **ticket**'ı elde etmek için gerçekleştirilebilir. Daha önce belirtildiği gibi, bu ayarın etkin olduğu hesaplara rastlamak nispeten nadirdir. Ancak, zaman zaman değerlendirmelerimizde bu durumu görebiliriz. Yine de, **Service Principal Name (SPN)** içeren hesaplar daha yaygın olup, daha sonra bu modülde ele alacağımız **Kerberoasting** saldırısına daha sık maruz kalır.

Bu saldırıyı farklı şekillerde de kullanabiliriz. Örneğin, bir saldırganın bir hesap üzerinde **`GenericWrite`** veya **`GenericAll`** izinleri varsa, bu **attribute**'u etkinleştirip **AS_REP ticket** elde ederek şifreyi **offline cracking** ile kırabilir ve ardından bu ayarı tekrar devre dışı bırakabilir. Bu teknik, "**targeted AS-REPRoasting attack**" olarak da adlandırılır. Burada, ayarı etkinleştirerek **AS-REPRoasting** gerçekleştirebiliriz, ancak başarının sağlanması, kullanıcının nispeten zayıf bir şifreye sahip olmasına bağlıdır.

Şimdi, bu saldırıyı bir **Windows host** üzerinden nasıl gerçekleştireceğimizi bazı örneklerle inceleyelim.


### Enumeration

**[PowerView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1)**, **[UserAccountControl](https://learn.microsoft.com/en-us/troubleshoot/windows-server/identity/useraccountcontrol-manipulate-account-properties) (UAC)** özelliği **DONT_REQ_PREAUTH** olarak ayarlanmış kullanıcıları **enumerate** etmek için kullanılabilir.


#### PowerShell Enumeration of accounts with DONT_REQ_PREAUTH

```
PS C:\Tools> Import-Module .\PowerView.ps1
PS C:\Tools> Get-DomainUser -UACFilter DONT_REQ_PREAUTH
logoncount : 0
badpasswordtime : 12/31/1600 7:00:00 PM
distinguishedname : CN=Jenna Smith,OU=Server
Team,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
objectclass : {top, person, organizationalPerson, user}
displayname : Jenna Smith
userprincipalname : jenna.smith@inlanefreight
name : Jenna Smith
objectsid : S-1-5-21-2974783224-3764228556-2640795941-
1999
samaccountname : jenna.smith
admincount : 1
codepage : 0
samaccounttype : USER_OBJECT
accountexpires : NEVER
countrycode : 0
whenchanged : 8/3/2020 8:51:43 PM
instancetype : 4
usncreated : 19711
objectguid : ea3c930f-aa8e-4fdc-987c-4a9ee1a75409
sn : smith
lastlogoff : 12/31/1600 7:00:00 PM
objectcategory :
CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata : {7/30/2020 6:28:24 PM, 7/30/2020 3:09:16
AM, 7/30/2020 3:09:16 AM, 7/28/2020 1:45:00
 AM...}
givenname : jenna
memberof : CN=Schema
Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
lastlogon : 12/31/1600 7:00:00 PM
badpwdcount : 0
cn : Jenna Smith
useraccountcontrol : PASSWD_NOTREQD, NORMAL_ACCOUNT,
DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
whencreated : 7/27/2020 7:35:57 PM
primarygroupid : 513
pwdlastset : 7/27/2020 3:35:57 PM
msds-supportedencryptiontypes : 0
usnchanged : 89508
```


**Rubeus** aracı, **[preauthscan](https://github.com/GhostPack/Rubeus#preauthscan)** aksiyonuyla **pre-authentication** gerektirmeyen hesapları bulmak için de kullanılabilir.

Not: **`Rubeus.exe asreproast /format:hashcat`** komutu, **`DONT_REQ_PREAUTH`** bayrağına sahip tüm hesapları **enumerate** etmek için kullanılabilir.


### Saldırının Gerçekleştirilmesi

Bu bilgilerle, **Rubeus** aracı **AS-REP**’i offline **hash cracking** için uygun formatta almak amacıyla kullanılabilir. Bu saldırı herhangi bir **domain user context** gerektirmez ve **Kerberos pre-authentication** etkin olmayan bir hesabın yalnızca adını bilerek gerçekleştirilebilir.

```
PS C:\Tools> .\Rubeus.exe asreproast /user:jenna.smith /domain:inlanefreight.local /dc:dc01.inlanefreight.local /nowrap /outfile:hashes.txt
 ______ _
 (_____ \ | |
 _____) )_ _| |__ _____ _ _ ___
 | __ /| | | | _ \| ___ | | | |/___)
 | | \ \| |_| | |_) ) ____| |_| |___ |
 |_| |_|____/|____/|_____)____/(___/
 v1.5.0
[*] Action: AS-REP roasting
[*] Target User : jenna.smith
[*] Target Domain : inlanefreight.local
[*] Target DC : dc01.inlanefreight.local
[*] Using domain controller: dc01.inlanefreight.local
(fe80::c872:c68d:a355:e6f3%11)
[*] Building AS-REQ (w/o preauth) for: 'inlanefreight.local\jenna.smith'
[+] AS-REQ w/o preauth successful!
[*] AS-REP hash:
 [email protected]
:9369076320<SNIP>


```


### Hash Cracking