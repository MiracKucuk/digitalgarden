---
{"dg-publish":true,"permalink":"/active-directory-expert/kerberos-attacks/"}
---

Kerberos, kullanıcıların ağ üzerinde kimlik doğrulamasına ve kimlik doğrulaması yapıldıktan sonra hizmetlere erişmesine olanak tanıyan bir protokoldür. Kerberos varsayılan olarak 88 numaralı portu kullanır ve Windows 2000'den beri domain hesapları için varsayılan kimlik doğrulama protokolüdür. Bir kullanıcı bilgisayarında oturum açtığında, Kerberos onun kimliğini doğrulamak için kullanılır. Bir kullanıcı ağ üzerindeki bir servise erişmek istediğinde kullanılır. Kerberos sayesinde, bir kullanıcının sürekli olarak parolasını yazması gerekmez ve sunucunun her kullanıcının parolasını bilmesi gerekmez. Bu bir merkezi kimlik doğrulama örneğidir

Kerberos, biletlere dayalı stateless  bir kimlik doğrulama protokolüdür. Bir kullanıcının kimlik bilgilerini kullanılabilir kaynaklara yönelik taleplerinden etkili bir şekilde ayırır ve şifrelerinin ağ üzerinden iletilmemesini sağlar.[ Zero-knowledge](https://en.wikipedia.org/wiki/Zero-knowledge_proof) kanıtlı bir protokoldür. [Kerberos Key ](https://docs.microsoft.com/en-us/windows/win32/secauthn/key-distribution-center)Distribution Center (KDC) önceki işlemleri kaydetmez; bunun yerine Kerberos Ticket Granting Service (TGS) geçerli bir Ticket Granting Ticket'a (TGT) dayanır. Bir kullanıcı geçerli bir TGT'ye sahipse, kimliğini kanıtlamış olması gerektiğini varsayar.


### Basic Understanding

Çok yüksek bir seviyede, bir kullanıcı ağ üzerindeki mevcut kaynaklarla etkileşime geçmek istediğinde, aşağıdakiler gerçekleşir:
* İlk olarak merkezi bir sunucudan bir “identity card” isteyeceklerdir.
* Kullanıcı daha sonra kim olduğunu kanıtlamak zorunda kalacak ve karşılığında “kimlik kartını” veya [Ticket Granting Ticket'ı (TGT](https://docs.microsoft.com/en-us/windows/win32/secauthn/ticket-granting-tickets)) alacaktır.
* Bir servise erişmek istediklerinde bu TGT sunulacaktır. Böylece, bir hizmete her erişmek istediklerinde, bu kimliği sunacaklar ve eğer geçerliyse, merkezi sunucu istenen kaynağa sunmak için geçici bir bilet sağlayacaktır.
* Bu geçici bilet, kullanıcının adı, grup üyeliği vb. gibi tüm bilgilerini içerir.
* Kaynak daha sonra bu bileti alacak ve kullanıcının bunu yapmaya hakkı varsa hizmetlerine erişim izni verebilecektir.

Bu proses iki aşamada gerçekleşir. İlk olarak, bir kullanıcının TGT'sini tanımlamak için bir bilet talebi ve ardından bir Ticket Granting Service ( TGS ) bileti veya Service Ticket ( ST ) kullanarak hizmetlere erişim talebi yoluyla.

Not: Ticket Granting Service (TGS), hizmet biletlerinin düzenlenmesinden sorumlu olan Key Distribution Center'ın (KDC) bir bileşenidir.

Not: Modül boyunca, TGS bileti terimini kullandığımızda, sanki bir Service Ticket  (ST) atıfta bulunuyormuşuz gibi olacaktır.


### Kerberos Avantajları
Kerberos saldırılarından ve Golden Ticket saldırısının tehlikelerinden bahsedilirken, Kerberos'un zayıf bir kimlik doğrulama protokolü olduğunu düşünmek kolaydır. Kerberos'tan önce kimlik doğrulama SMB/NTLM üzerinden gerçekleşiyordu ve kullanıcının hash'i, kimlik doğrulama sırasında bellekte saklanıyordu. Hedef bir makine ele geçirildiğinde ve NTLM hash'i çalındığında, saldırgan Pass-The-Hash saldırısı ile kullanıcı hesabının erişim yetkisine sahip olduğu her şeye ulaşabiliyordu. Daha önce belirtildiği gibi, Kerberos ticket'ları kullanıcının şifresini içermez ve ticket'ın erişim sağladığı makineyi belirtir.

Bu nedenle, WinRM üzerinden makineler uzaktan erişilirken [Double Hop](https://posts.specterops.io/offensive-lateral-movement-1744ae62b14f?gi=f925425e7a42) Problemi ortaya çıkar. Bir makineye uzaktan erişmek için Kerberos dışı bir protokol kullanıldığında, NTLM şifre hash'i o oturuma bağlı olduğundan, kullanıcıdan tekrar kimlik doğrulama istenmeden bu bağlantıyı kullanarak diğer makinelere o kullanıcı olarak erişmek mümkündür. Kerberos kimlik doğrulaması ile her erişilmek istenen makine için kimlik bilgileri özel olmalıdır çünkü bir şifre bulunmamaktadır. Bu konu hakkında daha fazla bilgi için Active Directory Enumeration & Attacks modülündeki Kerberos ["Double Hop](https://academy.hackthebox.com/module/143/section/1573)" Problemi bölümüne bakabilirsiniz.

Diyelim ki aktif oturumları olan bir makine Kerberos ile kimlik doğrulandı ve ele geçirildi. Bu durumda, Pass-The-Ticket saldırısı gerçekleştirmek mümkündür ve bu, bu modülün ilerleyen bölümlerinde açıklanıp gösterilecektir. Ancak, Pass-The-Hash saldırısının aksine, saldırgan yalnızca mağdur kullanıcının kimlik doğrulaması yaptığı kaynaklarla sınırlı olacaktır. Ayrıca, bu ticket'ların bir ömrü vardır, yani saldırganın kaynaklara erişmek ve kalıcılık sağlamaya çalışmak için sınırlı bir zaman aralığı bulunur.

Aşağıdaki bölüm, Kerberos kimlik doğrulama sürecini ayrıntılı bir şekilde açıklayarak ticket'ların nasıl korunduğunu ve ne içerdiğini ele almaktadır. Bu süreci anlamak, Kerberos ile ilgili saldırılara geçmeden önce çok önemlidir. Kerberos, Active Directory ortamında lateral hareket, ayrıcalık yükseltme ve kalıcılık sağlama amacıyla kötüye kullanılabilecek birçok yönteme sahiptir. Bu tekniklerin birçoğuyla penetrasyon testleri ve red team değerlendirmelerimiz sırasında karşılaşacağız. Güvenlik uzmanları olarak, Kerberos'un nasıl çalıştığını ve nasıl kötüye kullanılabileceğini derinlemesine anlamamız gerekmektedir. Ayrıca, Kerberos saldırılarının nasıl çalıştığını, müşterilerin bu saldırılara karşı nasıl korunabileceğini ve Kerberos suistimalini tespit etmek için doğru izleme yapılandırmalarının nasıl yapılacağını açıklamak da kritik öneme sahiptir.


### Kerberos Authentication Process

Kerberos context'inde, bir kullanıcı bir servise erişmek istediğinde üç varlık vardır: user (kullanıcı), service (servis) ve Key Distribution Center (Anahtar Dağıtım Merkezi) veya KDC olarak da bilinen kimlik doğrulama sunucusu.

KDC, tüm hesapların kimlik bilgilerini bilen varlıktır.

![Pasted image 20241205212511.png](/img/user/resimler/Pasted%20image%2020241205212511.png)

Kerberos protokolünü tam olarak anlamak için nasıl çalıştığını ve ne için kullanıldığını yakınlaştıracağız.


### Why Kerberos?

Kendimize sorabileceğimiz ilk soru, bu protokolün ne için kullanıldığıdır.

Bir yandan, kimlik doğrulamanın merkezileştirilmesi için kullanılır, böylece tüm hizmetlerin her kullanıcının kimlik bilgilerini bilmesi gerekmez. Bu, şifre değişikliği, yeni bir kullanıcının eklenmesi veya bir kullanıcının devre dışı bırakılması ya da silinmesi gibi durumlarda kullanıcıların düzenli olarak güncellendiği bir bağlamda son derece pratiktir. Eğer tüm hizmetlerin tüm kullanıcıların durumunu bilmesi gerekseydi, bu büyük bir karmaşıklık yaratırdı. Bunun yerine, yalnızca bir varlık olan KDC'nin mevcut kullanıcıların güncel bir listesine sahip olması gerekir.

Öte yandan, bu protokol kullanıcıların ağ üzerinden bir şifre göndermeden hizmetlere karşı kimlik doğrulaması yapmasına olanak tanır. Bu, ortadaki adam (on-path olarak da bilinir) saldırılarına karşı koruma sağlamak için mükemmel bir güvenlik önlemidir.


### Üst düzey genel bakış

### Tickets

Her iki ihtiyacı karşılamak için Kerberos, private key'ler ve bir ticket (bilet) mekanizması kullanır. Private key'ler, pratikte, bir Active Directory ortamında, farklı hesapların şifreleri (ya da en azından bu şifrelerin bir hash'leri) olarak kullanılır.

Üst düzey bir bakış açısıyla, bir kullanıcının bir hizmete nasıl erişebileceği aşağıda açıklanmıştır.

1. Başlamak için, kullanıcı ilk ticket'ı anahtar sunucusundan (KDC) talep eder ve bununla kim olduğunu kanıtlar. Bu, kullanıcının KDC'ye kimlik doğrulaması yaptığı andır. Bu ticket, TGT (Ticket Granting Ticket) olarak adlandırılır ve kullanıcının kimlik kartıdır. Kullanıcı hakkında ad, hesap oluşturulma tarihi, güvenlik bilgileri, kullanıcının ait olduğu gruplar vb. tüm bilgileri içerir. Bu kimlik kartı, TGT, varsayılan olarak birkaç saatle sınırlıdır. Bu ticket, KDC'ye yapılacak diğer tüm talepler için sunulur.

2. Bu TGT alındıktan sonra, kullanıcı her hizmete erişmesi gerektiğinde bunu KDC'ye sunar. KDC, gönderilen TGT'nin geçerli olduğunu ve kullanıcının bunu sahtekarca oluşturmadığını doğrular, eğer geçerliyse, kullanıcıya Ticket Granting Service (TGS) ticket'ı veya Service Ticket (ST) döner. Kullanıcının TGT'deki bilgileri, TGS ticket'ına dahil edilir.
    
3. Şimdi kullanıcı, belirli bir hizmet için bir TGS ticket'ına sahip olduğuna göre, bu TGS ticket'ını hizmete sunar ve hizmet, bu ticket'ın geçerliliğini kontrol eder. Her şey yolunda ise, hizmet, kullanıcının bilgilerini okuyarak, kullanıcının talep edilen hizmeti kullanmaya yetkili olup olmadığını belirler. Bu nedenle, hizmet, kullanıcının erişim haklarını kontrol eden taraftır.

1. **Kullanıcı Kimlik Doğrulama Talebi:**
    
    - **Kullanıcı**: KDC (Key Distribution Center) sunucusundan ilk ticket'ı talep eder.
    - **KDC**: Kullanıcının kimliğini doğrulamak için şifre veya private key ile oturum açma işlemini gerçekleştirir.
2. **TGT (Ticket Granting Ticket) Alınması:**
    
    - **KDC**: Kullanıcının kimliğini doğruladıktan sonra, TGT verir. Bu ticket, kullanıcının kimlik bilgilerini ve erişim bilgilerini içerir.
    - **Kullanıcı**: TGT'yi alır ve hizmetlere erişim için kullanmaya hazır hale gelir.
3. **Hizmet Erişimi Talebi:**
    
    - **Kullanıcı**: TGT'yi, erişmek istediği hizmete sunar.
    - **KDC**: Gönderilen TGT'yi doğrular ve geçerli olduğunu onaylar. Ardından, kullanıcıya bir **TGS (Ticket Granting Service)** veya **Service Ticket (ST)** gönderir.
4. **Hizmet Erişimi:**
    
    - **Kullanıcı**: TGS veya ST'yi ilgili hizmete sunar.
    - **Hizmet**: Ticket'ı doğrular ve kullanıcının yetkilerini kontrol eder. Eğer geçerli ise, kullanıcıya erişim sağlanır.

![Pasted image 20241205213257.png](/img/user/resimler/Pasted%20image%2020241205213257.png)
