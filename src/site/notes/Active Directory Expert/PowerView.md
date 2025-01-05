---
{"dg-publish":true,"permalink":"/active-directory-expert/power-view/"}
---


### Active Directory PowerView

### AD Enumeration Toolkit
Active Directory LDAP modülünde tartışıldığı gibi, derinlemesine numaralandırma, herhangi bir güvenlik değerlendirmesinin tartışmasız en önemli aşamasıdır. Saldırganlar AD'yi kötüye kullanmak ve saldırmak için yeni (ve eski) teknikler ve metodolojiler bulmaya devam etmektedir. AD'de bu aşama, OU'ların, kullanıcıların, grupların, bilgisayarların, ACL'lerin ve diğer AD nesnelerinin sayısı ve bir AD ortamını oluşturan yüzlerce ve binlerce ilişki dahil olmak üzere “araziyi tanımamıza” ve dahili ağın tasarımını anlamamıza yardımcı olur. İşimiz, ilgili verileri çeşitli formatlarda toplayarak ve ağın içinde saklanan kusurları ve yanlış yapılandırmaları ortaya çıkarmamıza yardımcı olacak şekilde düzenleyerek bu genellikle çok karmaşık ilişkileri çözmektir.

Active Directory LDAP modülü Active Directory'ye genel bir bakış sağladı, AD numaralandırması yaparken son derece yararlı olabilecek çeşitli yerleşik araçları tanıttı ve belki de en önemlisi, bu yerleşik araçlarla birleştirildiğinde bize AD'nin karmaşıklıklarını incelemek ve saldırganlardan önce ince ama ciddi yanlış yapılandırmaları keşfetmek için güçlü bir cephanelik sağlayan LDAP ve AD arama filtrelerini ele aldı. Değerlendirme yaparken “arazide yaşayabilmek” bizim için önemli olsa da, AD'yi numaralandırmak ve AD'ye saldırmak için kullanabileceğimiz çok çeşitli üçüncü taraf açık kaynak araçlarını anlamak da aynı derecede önemlidir. Bu modülde ele alacağımız araçların her biri AD numaralandırmasını biraz farklı şekillerde gerçekleştirir. Genellikle değerlendirme boyunca birçoğundan yinelemeli olarak veri toplamamız, analiz etmemiz ve yorumlamamız gerekir. Yerleşik araçları ve üçüncü taraf araçları etkin bir şekilde kullanma bilgisi ve becerisi, bizi diğer değerlendiricilerden ayırabilecek şeydir.

Yürüttüğümüz görevin türüne bağlı olarak, AD numaralandırması yapmak için kullanabileceğimiz çeşitli araçlar vardır. Etkin bir şekilde kullanabilmemiz için en önemlilerinden bazıları şunlardır:

![Pasted image 20241204143203.png](/img/user/resimler/Pasted%20image%2020241204143203.png)![Pasted image 20241204143214.png](/img/user/resimler/Pasted%20image%2020241204143214.png)
![Pasted image 20241204143222.png](/img/user/resimler/Pasted%20image%2020241204143222.png)

Bu modül, çeşitli AD numaralandırma tekniklerini kapsamak için PowerView ve SharpView araçlarına odaklanacaktır. Sızma testi uzmanları olarak, çok çeşitli araçlara sahip olmak ve beklenen sonuçları alamadığımızda sorun gidermek için nasıl çalıştıklarını anlamak önemlidir. Bu araçların her birini bir görevde kullanmasak da, nasıl çalıştıklarını, birbirlerini nasıl tamamladıklarını ve değerlendirmenin hedeflerine bağlı olarak hedef AD ortamının mümkün olan en derin kapsamını sağlamak için nasıl birleştirilebileceklerini anlamak önemlidir. Yukarıda listelenen araçlar diğer modüllerde ele alınacaktır.


### PowerView/SharpView Genel Bakış ve Kullanım
SharpView, artık kullanımdan kaldırılan PowerSploit offensive PowerShell araç setinde yer alan birçok araçtan biri olan PowerView'in bir .NET portudur. Bu [Read the Docs ](https://powersploit.readthedocs.io/en/latest/Recon/)sayfası fonksiyon adlandırma şemasını açıklar ve her fonksiyona aktarılabilecek çeşitli parametreler hakkında bilgi verir.

Not: Bu modülü yazdığımızdan beri BC-Security'nin Empire projesinin bir parçası olarak PowerView için güncellemeler yayınlamaya başladığını fark ettik. Bu kurs hala PowerSploit'in GitHub'ındaki Geliştirme PowerView modülünü kullanıyor, ancak yıl sonuna kadar bunu [Empire](https://github.com/BC-SECURITY/Empire)'ın kullandığı sürüme taşımayı planlıyoruz.

Geçmişte PowerShell, saldırı araçları için tercih edilen komut dosyası diliydi, ancak hem tüketici hem de kurumsal düzeydeki uç nokta koruma ürünleri için daha iyi algılama optikleri ile daha fazla güvenlik şeffaf hale geldi. Bu nedenle, saldırgan güvenlik uygulayıcıları, gelişmiş güvenlik izleme yeteneklerini hafifletmek için ticari araçlarını geliştirdiler ve araçlarını daha az güvenlik şeffaflığı olan C# inline'a taşıdılar. PowerView artık resmi olarak sürdürülmese de, hala son derece güçlü bir AD numaralandırma aracıdır ve gizliliğin bir gereklilik olmadığı angajmanlar gerçekleştirirken yararlı olabilir. Ayrıca, AD ortamlarını daha iyi anlamak isteyen savunucular için de yararlı olmaya devam etmektedir. PowerView'in tarihçesini ve genel kullanımını ele alacağız, ancak bu modül (ve ilgili modüller) gerçek hayattaki modern angajmanlara daha uygulanabilir olması için mevcut .NET ticaretine uygun olarak SharpView'e odaklanacaktır. Bu modülde genel PowerView ve SharpView kullanımını ele alacağız çünkü duruma bağlı olarak her ikisinin de hala kullanım alanları vardır

Her iki araç da numaralandırma yapabilir, durumsal farkındalık kazanabilir ve bir Windows domain'i içinde saldırılar gerçekleştirebilir. PowerView, PowerShell AD hook'larını ve Win32 API işlevlerini kullanır ve diğer fonksiyonların yanı sıra, yerleşik Windows araçları tarafından çağrılan çeşitli net komutlarının yerini alır. SharpView, tüm PowerView fonksiyonlarını ve argümanlarını bir .NET derlemesinde sağlayan bir .NET portudur. PowerView ve SharpView arasındaki önemli bir fark, komutları pipe etme yeteneğidir. SharpView, PowerShell nesneleri yerine stringler kullanır. Bu nedenle, çıktıyı ayrıştırmak veya belirli AD nesnelerini kolayca seçmek için Select veya Select-Object kullanarak özellikleri belirtemeyiz

![Pasted image 20241204144358.png](/img/user/resimler/Pasted%20image%2020241204144358.png)
![Pasted image 20241204144404.png](/img/user/resimler/Pasted%20image%2020241204144404.png)

Burada, net hesaplarına benzer bir komutun PowerView veya SharpView komutu Get-DomainPolicy ile gerçekleştirilebileceğini görebiliriz.

![Pasted image 20241204144428.png](/img/user/resimler/Pasted%20image%2020241204144428.png)

Her iki aracın işlevselliği farklı “buckets” halinde gruplandırılabilir. Bu bölümde her bir fonksiyonu ele almayacak olsak da, her birinin altındaki en önemli fonksiyonlardan bazılarını ele alacağız. Her iki araç da aynı işlevleri ve argümanları kullanır, ancak çıktı farklı olabilir. Bu [Read the Docs ](https://powersploit.readthedocs.io/en/latest/Recon/)dokümantasyonu, her bir fonksiyonun ve komut sözdiziminin derinlemesine bir açıklamasını ve fonksiyonların nasıl kullanılabileceğine dair çeşitli örnekler sunmaktadır.


### Çeşitli Fonksiyonlar
Çeşitli fonksiyonlar UAC değerlerini dönüştürme, SID dönüştürme, kullanıcı kimliğine bürünme, Kerberoasting ve daha fazlası gibi çeşitli faydalı araçlar sunar. Araç dokümantasyonundan açıklamalarla birlikte tüm fonksiyon listesi aşağıdaki gibidir:


- **Export-PowerViewCSV**
    
    - **Açıklama:** Birden fazla thread ile çalışırken, bir CSV dosyasına güvenli bir şekilde ekleme yapar.
- **Resolve-IPAddress**
    
    - **Açıklama:** Bir hostname'i IP adresine çözer.
- **ConvertTo-SID**
    
    - **Açıklama:** Belirtilen kullanıcı/grup adını, bir güvenlik tanımlayıcısına (SID) dönüştürür.
- **Convert-ADName**
    
    - **Açıklama:** Nesne adlarını çeşitli formatlar arasında dönüştürür.
- **ConvertFrom-UACValue**
    
    - **Açıklama:** Bir UAC (Kullanıcı Hesabı Denetimi) tam sayı değerini okunabilir bir forma dönüştürür.
- **Add-RemoteConnection**
    
    - **Açıklama:** Belirtilen kimlik bilgisi nesnesini kullanarak, bir uzak yolu "taklit" ederek bağlanır.
- **Remove-RemoteConnection**
    
    - **Açıklama:** `NewRemoteConnection` komutuyla oluşturulan bağlantıyı kaldırır.
- **Invoke-UserImpersonation**
    
    - **Açıklama:** "runas /netonly" benzeri bir oturum açar ve belirli bir token'i taklit eder.
- **Invoke-RevertToSelf**
    
    - **Açıklama:** Mevcut bir token taklidini iptal eder.
- **Get-DomainSPNTicket**
    
    - **Açıklama:** Belirtilen bir hizmet ana ismi (SPN) için Kerberos bileti talep eder.
- **Invoke-Kerberoast**
    
    - **Açıklama:** Kerberoast'a uygun hesaplar için hizmet biletleri talep eder ve çıkarılan bilet hash'lerini döndürür.
- **Get-PathAcl**
    
    - **Açıklama:** Yerel/uzak bir dosya yolunun ACL'lerini (Erişim Kontrol Listesi) ve isteğe bağlı olarak grup sorgulamalarını alır.


Bir kullanıcı adını ilgili [SID](https://en.wikipedia.org/wiki/Security_Identifier)'ye dönüştürmek için SharpView veya PowerView kullanabiliriz.

![Pasted image 20241204145823.png](/img/user/resimler/Pasted%20image%2020241204145823.png)

Ve tam tersi:

![Pasted image 20241204145847.png](/img/user/resimler/Pasted%20image%2020241204145847.png)

useraccountcontrol değerini kullanarak UAC değerlerini listelediğimizde, değerler bize insan tarafından okunabilir bir biçimde değil, sayısal değerler olarak geri gösterilir. ConvertFrom-UACValue fonksiyonunu kullanabiliriz. Eğer -showall özelliğini eklersek, tüm ortak UAC değerleri gösterilir ve kullanıcı için ayarlanmış olanlar + ile işaretlenir. Bu, gelecekteki etkileşimler için bir hile sayfasına referans olarak kaydedilebilir.

![Pasted image 20241204145920.png](/img/user/resimler/Pasted%20image%2020241204145920.png)
![Pasted image 20241204145938.png](/img/user/resimler/Pasted%20image%2020241204145938.png)


### Domain/LDAP Functions

- **Get-DomainDNSZone**
    
    - **Açıklama:** Belirtilen bir etki alanı için Active Directory DNS bölgelerini listelemek için kullanılır.
- **Get-DomainDNSRecord**
    
    - **Açıklama:** Belirtilen bir DNS bölgesi için Active Directory DNS kayıtlarını listeler.
- **Get-Domain**
    
    - **Açıklama:** Mevcut (veya belirtilen) etki alanı için etki alanı nesnesini döndürür.
- **Get-DomainController**
    
    - **Açıklama:** Mevcut (veya belirtilen) etki alanı için etki alanı denetleyicilerini döndürür.
- **Get-Forest**
    
    - **Açıklama:** Mevcut (veya belirtilen) orman için orman nesnesini döndürür.
- **Get-ForestDomain**
    
    - **Açıklama:** Mevcut (veya belirtilen) orman içindeki tüm etki alanlarını döndürür.
- **Get-ForestGlobalCatalog**
    
    - **Açıklama:** Mevcut (veya belirtilen) orman içindeki tüm global katalogları döndürür.
- **Find-DomainObjectPropertyOutlier**
    
    - **Açıklama:** Active Directory'de kullanıcı, grup veya bilgisayar nesneleri üzerinde "farklı" (standart dışı) özelliklere sahip olanları bulur.
- **Get-DomainUser**
    
    - **Açıklama:** Active Directory'deki tüm kullanıcıları veya belirli kullanıcı nesnelerini döndürür.
- **New-DomainUser**
    
    - **Açıklama:** Yeni bir etki alanı kullanıcısı oluşturur (uygun izinler varsa) ve kullanıcı nesnesini döndürür.
- **Set-DomainUserPassword**
    
    - **Açıklama:** Belirtilen bir kullanıcı için şifreyi değiştirir ve kullanıcı nesnesini döndürür.
- **Get-DomainUserEvent**
    
    - **Açıklama:** Hesap giriş olaylarını (ID 4624) ve açık kimlik bilgileriyle giriş olaylarını listeler.
- **Get-DomainComputer**
    
    - **Açıklama:** Active Directory'deki tüm bilgisayarları veya belirli bilgisayar nesnelerini döndürür.
- **Get-DomainObject**
    
    - **Açıklama:** Active Directory'deki tüm (veya belirtilen) nesneleri döndürür.
- **Set-DomainObject**
    
    - **Açıklama:** Belirtilen bir Active Directory nesnesi için bir özelliği değiştirir.
- **Get-DomainObjectAcl**
    
    - **Açıklama:** Belirli bir Active Directory nesnesiyle ilişkili ACL'leri (Erişim Kontrol Listesi) döndürür.
- **Add-DomainObjectAcl**
    
    - **Açıklama:** Belirli bir Active Directory nesnesi için bir ACL ekler.
- **Find-InterestingDomainAcl**
    
    - **Açıklama:** Mevcut (veya belirtilen) etki alanındaki, varsayılan olmayan nesnelere değişiklik hakları tanımlanmış ACL'leri bulur.
- **Get-DomainOU**
    
    - **Açıklama:** Active Directory'deki tüm organizasyon birimlerini (OU) veya belirli OU nesnelerini arar.
- **Get-DomainSite**
    
    - **Açıklama:** Active Directory'deki tüm siteleri veya belirli site nesnelerini arar.
- **Get-DomainSubnet**
    
    - **Açıklama:** Active Directory'deki tüm alt ağları veya belirli alt ağ nesnelerini arar.
- **Get-DomainSID**
    
    - **Açıklama:** Mevcut veya belirtilen etki alanı için SID'yi döndürür.
- **Get-DomainGroup**
    
    - **Açıklama:** Active Directory'deki tüm grupları veya belirli grup nesnelerini döndürür.
- **New-DomainGroup**
    
    - **Açıklama:** Yeni bir etki alanı grubu oluşturur (uygun izinler varsa) ve grup nesnesini döndürür.
- **Get-DomainManagedSecurityGroup**
    
    - **Açıklama:** Mevcut (veya hedef) etki alanındaki bir yönetici atanmış tüm güvenlik gruplarını döndürür.
- **Get-DomainGroupMember**
    
    - **Açıklama:** Belirli bir etki alanı grubunun üyelerini döndürür.
- **Add-DomainGroupMember**
    
    - **Açıklama:** Belirtilen bir etki alanı grubuna kullanıcı (veya grup) ekler (uygun izinler varsa).
- **Get-DomainFileServer**
    
    - **Açıklama:** Dosya sunucusu olarak işlev gören sunucuların bir listesini döndürür.
- **Get-DomainDFSShare**
    
    - **Açıklama:** Mevcut (veya belirtilen) etki alanı için tüm hata toleranslı dağıtılmış dosya sistemlerini listeler.

LDAP fonksiyonları bize çok sayıda faydalı komut sağlar. Get-Domain işlevi bize domain hakkında isim, alt domainler, domain controller listesi, domain controller rolleri ve daha fazlası gibi bilgiler sağlayacaktır.

![Pasted image 20241204150708.png](/img/user/resimler/Pasted%20image%2020241204150708.png)
![Pasted image 20241204150714.png](/img/user/resimler/Pasted%20image%2020241204150714.png)

Get-DomainOU fonksiyonu ile arazinin yapısını öğrenmeye başlayabilir ve domain yapısının haritasını çıkarmamıza yardımcı olabilecek tüm Organizasyonel Birimlerin (OU'lar) adlarını döndürebiliriz. Bu isimleri SharpView ile numaralandırabiliriz.

![Pasted image 20241204150746.png](/img/user/resimler/Pasted%20image%2020241204150746.png)
![Pasted image 20241204150755.png](/img/user/resimler/Pasted%20image%2020241204150755.png)
![Pasted image 20241204150803.png](/img/user/resimler/Pasted%20image%2020241204150803.png)


Get-DomainUser fonksiyonu ile domain kullanıcıları hakkında bilgi toplayabilir ve PreauthNotRequired gibi özellikler belirleyerek saldırıları planlamayı deneyebiliriz.

![Pasted image 20241204150842.png](/img/user/resimler/Pasted%20image%2020241204150842.png)
![Pasted image 20241204150849.png](/img/user/resimler/Pasted%20image%2020241204150849.png)
![Pasted image 20241204150856.png](/img/user/resimler/Pasted%20image%2020241204150856.png)

GetDomainComputer fonksiyonunu kullanarak tek tek host'lar hakkında bilgi toplamaya da başlayabiliriz

![Pasted image 20241204150915.png](/img/user/resimler/Pasted%20image%2020241204150915.png)


### GPO functions

- **Get-DomainGPO**
    
    - **Açıklama:** Active Directory'deki tüm Grup İlkesi Nesnelerini (GPO) veya belirli GPO nesnelerini döndürür.
- **Get-DomainGPOLocalGroup**
    
    - **Açıklama:** Etki alanındaki yerel grup üyeliklerini "Kısıtlı Gruplar" veya Grup İlkesi tercihleri aracılığıyla değiştiren tüm GPO'ları döndürür.
- **Get-DomainGPOUserLocalGroupMapping**
    
    - **Açıklama:** Belirli bir etki alanı kullanıcı/grubunun, GPO ilişkisi aracılığıyla hangi makinelerde belirli bir yerel grubun üyesi olduğunu listeler.
- **Get-DomainGPOComputerLocalGroupMapping**
    
    - **Açıklama:** Belirli bir bilgisayar (veya GPO) nesnesi için, GPO ilişkisi aracılığıyla belirli bir yerel grupta hangi kullanıcıların/grupların bulunduğunu belirler.
- **Get-DomainPolicy**
    
    - **Açıklama:** Mevcut (veya belirtilen) etki alanı/etki alanı denetleyicisi için varsayılan etki alanı ilkesini veya etki alanı denetleyici ilkesini döndürür.


GPO fonksiyonlarına geçecek olursak, tüm Group Policy Objects (GPO) adlarını döndürmek için Get-DomainGPO'yu kullanabiliriz

![Pasted image 20241204151241.png](/img/user/resimler/Pasted%20image%2020241204151241.png)

Hangi GPO'ların hangi hostlara geri eşlendiğini de belirleyebiliriz

![Pasted image 20241204151314.png](/img/user/resimler/Pasted%20image%2020241204151314.png)


### Computer Enumeration Functions

- **Get-NetLocalGroup**
    
    - **Açıklama:** Yerel (veya uzak) makinedeki yerel grupları listeler.
- **Get-NetLocalGroupMember**
    
    - **Açıklama:** Yerel (veya uzak) makinedeki belirli bir yerel grubun üyelerini listeler.
- **Get-NetShare**
    
    - **Açıklama:** Yerel (veya uzak) makinedeki açık paylaşımları döndürür.
- **Get-NetLoggedon**
    
    - **Açıklama:** Yerel (veya uzak) makinede oturum açmış kullanıcıları döndürür.
- **Get-NetSession**
    
    - **Açıklama:** Yerel (veya uzak) makine için oturum bilgilerini döndürür.
- **Get-RegLoggedOn**
    
    - **Açıklama:** Uzaktan kayıt defteri anahtarlarını inceleyerek yerel (veya uzak) makinede oturum açmış kullanıcıları döndürür.
- **Get-NetRDPSession**
    
    - **Açıklama:** Yerel (veya uzak) makine için uzak masaüstü/oturum bilgilerini döndürür.
- **Test-AdminAccess**
    
    - **Açıklama:** Mevcut kullanıcının yerel (veya uzak) makinede yönetici erişimine sahip olup olmadığını test eder.
- **Get-NetComputerSiteName**
    
    - **Açıklama:** Yerel (veya uzak) makinenin bulunduğu Active Directory sitesini döndürür.
- **Get-WMIRegProxy**
    
    - **Açıklama:** Mevcut kullanıcı için proxy sunucusunu ve WPAD içeriğini listeler.
- **Get-WMIRegLastLoggedOn**
    
    - **Açıklama:** Yerel (veya uzak) makinede oturum açan son kullanıcıyı döndürür.
- **Get-WMIRegCachedRDPConnection**
    
    - **Açıklama:** Yerel (veya uzak) makineden yapılan RDP bağlantılarına dair bilgileri döndürür.
- **Get-WMIRegMountedDrive**
    
    - **Açıklama:** Yerel (veya uzak) makine için kaydedilmiş ağ sürücüsü bağlantılarını listeler.
- **Get-WMIProcess**
    
    - **Açıklama:** Yerel (veya uzak) makinedeki işlemleri ve bunların sahiplerini listeler.
- **Find-InterestingFile**
    
    - **Açıklama:** Belirtilen yolda, belirtilen kriterlere uygun dosyaları arar.


Bilgisayar numaralandırma fonksiyonları kullanıcı oturumları hakkında bilgi toplayabilir, local admin erişimini test edebilir, dosya paylaşımlarını ve ilginç dosyaları arayabilir ve daha fazlasını yapabilir. TestAdminAccess fonksiyonu, mevcut kullanıcımızın herhangi bir remote host üzerinde local admin haklarına sahip olup olmadığını kontrol edebilir.


![Pasted image 20241204151514.png](/img/user/resimler/Pasted%20image%2020241204151514.png)

Remote bir bilgisayardaki açık paylaşımları listelemek için Net-Share fonksiyonunu kullanabiliriz. Paylaşımlar çok sayıda bilgi barındırabilir ve dosya paylaşımlarını numaralandırmanın önemi göz ardı edilmemelidir

![Pasted image 20241204151541.png](/img/user/resimler/Pasted%20image%2020241204151541.png)


### Threaded 'Meta'-Functions

- **Find-DomainUserLocation**
    
    - **Açıklama:** Belirli kullanıcıların oturum açtığı etki alanı makinelerini bulur.
- **Find-DomainProcess**
    
    - **Açıklama:** Belirli işlemlerin çalıştığı etki alanı makinelerini bulur.
- **Find-DomainUserEvent**
    
    - **Açıklama:** Belirtilen kullanıcılar için mevcut (veya uzak) etki alanında oturum açma olaylarını bulur.
- **Find-DomainShare**
    
    - **Açıklama:** Etki alanı makinelerinde erişilebilir paylaşımları bulur.
- **Find-InterestingDomainShareFile**
    
    - **Açıklama:** Etki alanındaki okunabilir paylaşımlarda belirli kriterlere uyan dosyaları arar.
- **Find-LocalAdminAccess**
    
    - **Açıklama:** Mevcut kullanıcının yerel yönetici erişimine sahip olduğu etki alanı makinelerini bulur.
- **Find-DomainLocalGroupMember**
    
    - **Açıklama:** Etki alanındaki makinelerde belirtilen yerel grupların üyelerini listeler.


'Meta' işlevleri, domain kullanıcılarının nerede oturum açtığını bulmak, remote host'lardaki belirli prosesleri aramak, domain paylaşımlarını bulmak, domain paylaşımlarındaki dosyaları bulmak ve mevcut kullanıcımızın local admin haklarına sahip olduğu yerleri test etmek için kullanılabilir. Find-DomainUserLocation fonksiyonunu, kullanıcıların oturum açtığı domain makinelerini bulmak için kullanabiliriz.

![Pasted image 20241204151717.png](/img/user/resimler/Pasted%20image%2020241204151717.png)


### Domain Trust Functions

- **Get-DomainTrust**
    
    - **Açıklama:** Mevcut etki alanı veya belirtilen bir etki alanı için tüm etki alanı güven ilişkilerini döndürür.
- **Get-ForestTrust**
    
    - **Açıklama:** Mevcut orman veya belirtilen bir orman için tüm orman güven ilişkilerini döndürür.
- **Get-DomainForeignUser**
    
    - **Açıklama:** Kullanıcının etki alanı dışındaki gruplarda bulunan kullanıcıları listeler.
- **Get-DomainForeignGroupMember**
    
    - **Açıklama:** Kendi etki alanı dışındaki kullanıcıları içeren grupları listeler ve her yabancı üyenin bilgilerini döndürür.
- **Get-DomainTrustMapping**
    
    - **Açıklama:** Mevcut etki alanı için tüm güven ilişkilerini ve ardından bulduğu her etki alanının güven ilişkilerini sıralar.


Domain trust fonksiyonları bize cross-trust saldırıları düzenlemek için kullanılabilecek bilgileri listelemek için ihtiyaç duyduğumuz araçları sağlar. Bu komutların en temeli olan GetDomainTrust, mevcut domainimiz için tüm domain güvenlerini döndürür

![Pasted image 20241204151829.png](/img/user/resimler/Pasted%20image%2020241204151829.png)


### Kapanış Düşünceleri
PowerView / SharpView, Kerberoasting ve ASREPRoasting saldırılarını gerçekleştirmek ve daha sonraki modüllerde ele alınacak olan Kerberos delegasyonunu kötüye kullanmak için de kullanılabilir. 

PowerView token kimliğine bürünme özelliğinden yararlanabilir. Yeni bir proses oluşturmak yerine, kullanıcıyı geçici olarak taklit ederek ve ardından mevcut kullanıcıya geri dönerek komutları başka bir kullanıcı olarak çalıştırmayı sağlar. Kimlik bilgileri -Credential bayrağı kullanılarak belirtilebilir. 

Not: Gizli kalmaya çalışılıyorsa, kullanıcı kimliğine bürünme çağrısı, yönetici düzeyi veya eşdeğer ayrıcalıklara sahip hassas bir hesap kullanılıyorsa uyarı oluşturabilecek bir oturum açma olayı oluşturur.


### Enumerating AD Users
Bir AD ortamında numaralandırmaya başlarken, tartışmasız en önemli nesneler domain kullanıcılarıdır. Kullanıcıların bilgisayarlara erişimi vardır ve domain genelinde çeşitli işlevleri gerçekleştirmek için izinler atanır. Değerlendirme hedefine ulaşmak için kullanıcı hesaplarının bir ağ içinde yanal ve dikey olarak hareket etmesini kontrol etmemiz gerekir.


### Temel AD Kullanıcı Veri Noktaları
PowerView ve SharpView'u AD kullanıcıları hakkında çok sayıda bilgiyi listelemek için kullanabiliriz. Hedef domain'de kaç kullanıcı olduğunu sayarak başlayabiliriz. Bu kursta iki araç arasında sık sık geçiş yapacağız çünkü ikisini de bilmek önemlidir. Bazen powershell komutlarını çalıştıramayabilirsiniz ve diğer zamanlarda çalıştırılabilir dosyaları çalıştıramayabilirsiniz. SharpView kullanımını gördüğünüzde, sadece SharpView.exe'yi kaldırarak bunu PowerView'de yapmak mümkün olmalıdır. SharpView en son PowerSploit özelliklerine sahip değildir, bu nedenle SharpView içinde PowerView komutlarını çalıştırmak mümkün olmayabilir.

![Pasted image 20241204193601.png](/img/user/resimler/Pasted%20image%2020241204193601.png)


Şimdi Get-DomainUser fonksiyonunu inceleyelim. Herhangi bir SharpView fonksiyonuna -Help bayrağını sağlarsak, fonksiyonun kabul ettiği tüm parametreleri görebiliriz.

![Pasted image 20241204193632.png](/img/user/resimler/Pasted%20image%2020241204193632.png)
![Pasted image 20241204193638.png](/img/user/resimler/Pasted%20image%2020241204193638.png)

Aşağıda, domain kullanıcıları hakkında toplanması gereken en önemli özelliklerden bazıları verilmiştir. harry.jones kullanıcısına bir göz atalım

![Pasted image 20241204193822.png](/img/user/resimler/Pasted%20image%2020241204193822.png)
Bu özellikleri TÜM domain kullanıcıları için listelemek ve çevrimdışı işleme için bir CSV dosyasına aktarmak yararlıdır.

![Pasted image 20241204193914.png](/img/user/resimler/Pasted%20image%2020241204193914.png)


Tüm kullanıcılar hakkında bilgi topladıktan sonra, Kerberos ön kimlik doğrulaması gerektirmeyen ve ASREPRoast saldırısına maruz kalabilecek kullanıcıların bir listesini elde ederek daha spesifik kullanıcı numaralandırması yapmaya başlayabiliriz.

![Pasted image 20241204193936.png](/img/user/resimler/Pasted%20image%2020241204193936.png)

Kerberos kısıtlı delegasyonu olan kullanıcılar hakkında da bilgi toplayalım

![Pasted image 20241204194025.png](/img/user/resimler/Pasted%20image%2020241204194025.png)


Bunu yaparken, kısıtlanmamış yetkilendirmeye izin veren kullanıcıları arayabiliriz.

![Pasted image 20241204194053.png](/img/user/resimler/Pasted%20image%2020241204194053.png)
![Pasted image 20241204194103.png](/img/user/resimler/Pasted%20image%2020241204194103.png)


Ayrıca, açıklama alanında saklanan parola gibi hassas verilere sahip domain kullanıcılarını da kontrol edebiliriz


![Pasted image 20241204194132.png](/img/user/resimler/Pasted%20image%2020241204194132.png)


Ardından, Kerberoasting saldırısına maruz kalabilecek Service Principal Names'e (SPN'ler) sahip tüm kullanıcıları listeleyelim.

![Pasted image 20241204194204.png](/img/user/resimler/Pasted%20image%2020241204194204.png)
![Pasted image 20241204194214.png](/img/user/resimler/Pasted%20image%2020241204194214.png)
![Pasted image 20241204194220.png](/img/user/resimler/Pasted%20image%2020241204194220.png)

Son olarak, mevcut domainimizdeki herhangi bir grup içinde grup üyeliği olan diğer (yabancı) domainlerdeki kullanıcıları listeleyebiliriz. FREIGHTLOGISTICS.LOCAL domain'inden harry.jones kullanıcısının mevcut domain'imizin administrators grubunda olduğunu görebiliriz. Geçerli domain'i tehlikeye atarsak, NTDS veritabanından bu kullanıcının kimlik bilgilerini alabilir ve FREIGHTLOGISTICS.LOCAL domain'inde kimlik doğrulaması yapabiliriz.

![Pasted image 20241204194504.png](/img/user/resimler/Pasted%20image%2020241204194504.png)

![Pasted image 20241204194516.png](/img/user/resimler/Pasted%20image%2020241204194516.png)

Bir başka yararlı komut da, tüm kullanıcıların bir trust genelinde kimlik doğrulamasına izin veren forest-wide authentication veya belirli kullanıcıların kimlik doğrulamasına izin veren selectiveauthentication kurulumu ile gelen veya çift yönlü trust ilişkileri yoluyla kimlik doğrulaması yapabileceğimiz diğer domainlerde ayarlanmış Service Principal Names (SPNs) olan kullanıcıları kontrol etmektir. Burada, FREIGHTLOGISTICS.LOCAL domaininde, forest trust genelinde Kerberoast için kullanılabilecek bir hesap görebiliriz.

![Pasted image 20241204194706.png](/img/user/resimler/Pasted%20image%2020241204194706.png)
![Pasted image 20241204194710.png](/img/user/resimler/Pasted%20image%2020241204194710.png)


### Şifre Ayarlama Zamanları
Password Set zamanlarını analiz etmek, password spreyleri gerçekleştirirken inanılmaz derecede önemlidir. Kuruluşların tüm hesaplarda otomatik bir parola spreyi bulma olasılığı, küçük bir hesap grubuna yönelik birkaç tahminden çok daha yüksektir.

* Aynı anda birkaç parolanın belirlendiğini görürseniz, bu Help Desk tarafından belirlendiklerini ve aynı olabileceklerini gösterir. Password Lockout Politikaları nedeniyle, on beş dakika içinde dört başarısız parolayı geçemeyebilirsiniz. Ancak, parolanın 20 hesapta aynı olduğunu düşünüyorsanız, bir kullanıcı için “Password2020” farklı bir kullanım için “Freight2020!” gibi şirket adını kullanabilirsiniz.

* Ayrıca, parolanın Temmuz 2019'da belirlendiğini görürseniz; normalde “2020 ‘yi parola tahminlerinizden hariç tutabilirsiniz ve muhtemelen ’Winter2019” gibi mantıklı olmayan varyasyonları tahmin etmemelisiniz.

* Eğer 2 yıl önce belirlenmiş eski bir parola görüyorsanız, büyük ihtimalle bu parola zayıftır ve ayrıca büyük bir Parola Spreyi başlatmadan önce parolasını tahmin etmenizi önereceğim ilk hesaplardan biridir.

* Çoğu kuruluşta yöneticilerin birden fazla hesabı vardır. Yöneticinin “kullanıcı hesabını” “Yönetici Hesabı” ile aynı zamanda değiştirdiğini görürseniz, büyük olasılıkla her iki hesap için de aynı parolayı kullanıyorlardır.

Aşağıdaki komut tüm şifre ayarlama zamanlarını gösterecektir.

![Pasted image 20241204195427.png](/img/user/resimler/Pasted%20image%2020241204195427.png)

Yalnızca belirli bir tarihten önce ayarlanmış parolaları göstermek istiyorsanız:

![Pasted image 20241204195441.png](/img/user/resimler/Pasted%20image%2020241204195441.png)

Mavi takım ipucu: Ne zaman bir tehlikeyle uğraşsanız veya bir Sızma Testi tamamlasanız. Tüm parolaların döndürüldüğünü doğrulamak için yukarıdaki komutu kullanmak her zaman iyi bir fikirdir!

Artık domain kullanıcıları hakkında çok sayıda bilgi topladığımıza göre, domainin haritasını çıkarmak için grup üyeliklerine bakalım


### Enumerating AD Groups
Domain kullanıcı bilgileriyle birlikte, bir grubun üyelerinin hangi ayrıcalıklara sahip olabileceğini görmek ve hatta istenmeyen haklara yol açabilecek iç içe geçmiş grupları veya grup üyeliğiyle ilgili sorunları bulmak için AD grup bilgilerini toplamak önemlidir


### Domain Groups
 Hızlı bir kontrol, hedef domainimiz INLANEFREIGHT.LOCAL'in 72 gruba sahip olduğunu gösteriyor
 ![Pasted image 20241204201944.png](/img/user/resimler/Pasted%20image%2020241204201944.png)
![Pasted image 20241204201956.png](/img/user/resimler/Pasted%20image%2020241204201956.png)
![Pasted image 20241204202005.png](/img/user/resimler/Pasted%20image%2020241204202005.png)
![Pasted image 20241204202009.png](/img/user/resimler/Pasted%20image%2020241204202009.png)

Grup adlarının tam bir listesini alalım. Bunların çoğu yerleşik, standart AD gruplarıdır. Bazı grupların varlığı bize Microsoft Exchange'in ortamda mevcut olduğunu gösterir. Bir Exchange yüklemesi AD'ye çeşitli gruplar ekler; bunlardan Exchange Trusted Subsystem ve Exchange Windows Permissions gibi bazıları, bu gruplara üyeliğin bir kullanıcıya veya bilgisayara verdiği izinler nedeniyle [yüksek değerli hedefler](https://github.com/gdedrouas/Exchange-AD-Privesc) olarak kabul edilir. Protected Users , LAPS Admins , Help Desk ve Security Operations gibi diğer gruplar da incelenmek üzere not edilmelidir.

Herhangi bir gruptaki grup üyeliğini incelemek için Get-DomainGroupMember'ı kullanabiliriz. Yine, bunun için SharpView fonksiyonunu kullanırken, bu fonksiyonun kabul ettiği tüm parametreleri görmek için -Help bayrağını geçebiliriz.

![Pasted image 20241204202136.png](/img/user/resimler/Pasted%20image%2020241204202136.png)

Help Desk grubunun hızlı bir incelemesi bize iki üye olduğunu gösteriyor

![Pasted image 20241204202156.png](/img/user/resimler/Pasted%20image%2020241204202156.png)
![Pasted image 20241204202205.png](/img/user/resimler/Pasted%20image%2020241204202205.png)
![Pasted image 20241204202211.png](/img/user/resimler/Pasted%20image%2020241204202211.png)


### Protected Groups
Ardından, AdminCount özniteliği 1 olarak ayarlanmış tüm AD gruplarını arayabiliriz, bu da bunun korumalı bir grup olduğunu gösterir.

![Pasted image 20241204202812.png](/img/user/resimler/Pasted%20image%2020241204202812.png)
![Pasted image 20241204202823.png](/img/user/resimler/Pasted%20image%2020241204202823.png)
![Pasted image 20241204202833.png](/img/user/resimler/Pasted%20image%2020241204202833.png)
![Pasted image 20241204202841.png](/img/user/resimler/Pasted%20image%2020241204202841.png)
![Pasted image 20241204202847.png](/img/user/resimler/Pasted%20image%2020241204202847.png)

Bir diğer önemli kontrol de yönetilen security gruplarını aramaktır. Bu gruplar, yönetici olmayan kişilere AD güvenlik gruplarına ve "[dağıtım gruplarına](https://learn.microsoft.com/en-us/exchange/recipients-in-exchange-online/manage-distribution-groups/manage-distribution-groups)"üye ekleme yetkisi verir ve managedBy özniteliği değiştirilerek ayarlanır. Bu denetim, bir grubun yönetici ayarlı olup olmadığına ve kullanıcının gruba kullanıcı ekleyip ekleyemeyeceğine bakar. Bu, bize ek kaynaklara erişim sağlayarak yanal hareket için yararlı olabilir. İlk olarak, yönetilen güvenlik grupları listesine bir göz atalım

![Pasted image 20241204203003.png](/img/user/resimler/Pasted%20image%2020241204203003.png)

Ardından, Security Operations grubuna bakalım ve grubun bir yönetici ayarlayıp ayarlamadığını görelim. joe.evans kullanıcısının grup yöneticisi olarak ayarlandığını görebiliriz.
![Pasted image 20241204203037.png](/img/user/resimler/Pasted%20image%2020241204203037.png)
![Pasted image 20241204203045.png](/img/user/resimler/Pasted%20image%2020241204203045.png)
![Pasted image 20241204203059.png](/img/user/resimler/Pasted%20image%2020241204203059.png)

Bu grup üzerinde ayarlanan ACL'leri sıraladığımızda, bu kullanıcının GenericWrite ayrıcalıklarına sahip olduğunu görebiliriz, yani bu kullanıcı grup üyeliğini değiştirebilir (kullanıcı ekleyebilir veya kaldırabilir). Bu kullanıcı hesabının kontrolünü ele geçirirsek, bu hesabı veya kontrol ettiğimiz başka herhangi bir hesabı gruba ekleyebilir ve domain'de sahip olduğu tüm ayrıcalıkları devralabiliriz.

![Pasted image 20241204203129.png](/img/user/resimler/Pasted%20image%2020241204203129.png)

![Pasted image 20241204203137.png](/img/user/resimler/Pasted%20image%2020241204203137.png)


### Local Groups

Lokal grup üyeliğini kontrol etmek de önemlidir. Mevcut kullanıcımız local admin mi yoksa herhangi bir hosttaki local grupların bir parçası mı? GetNetLocalGroup kullanarak bir host üzerindeki local grupların bir listesini alabiliriz

![Pasted image 20241204203225.png](/img/user/resimler/Pasted%20image%2020241204203225.png)
![Pasted image 20241204203231.png](/img/user/resimler/Pasted%20image%2020241204203231.png)

Ayrıca GetNetLocalGroupMember fonksiyonunu kullanarak herhangi bir host üzerindeki local grup üyelerini listeleyebiliriz

![Pasted image 20241204203302.png](/img/user/resimler/Pasted%20image%2020241204203302.png)
![Pasted image 20241204203352.png](/img/user/resimler/Pasted%20image%2020241204203352.png)
![Pasted image 20241204203410.png](/img/user/resimler/Pasted%20image%2020241204203410.png)

Local administrators grubunda RID 500 olmayan bir kullanıcı görüyoruz ve SID'yi dönüştürmek ve harry.jones kullanıcısını ortaya çıkarmak için ConvertSidToName işlevini kullanıyoruz.

![Pasted image 20241204203435.png](/img/user/resimler/Pasted%20image%2020241204203435.png)

Aynı fonksiyonu, belirli bir kullanıcının local yönetici erişimine sahip olduğu tüm hostları kontrol etmek için kullanırız, ancak bu modülde daha sonra ele alacağımız başka bir PowerView / SharpView fonksiyonu ile bu işlem çok daha hızlı yapılabilir.

![Pasted image 20241204203508.png](/img/user/resimler/Pasted%20image%2020241204203508.png)


### Gruba Eklenen Kullanıcı Tarihini Çekme

PowerView, bir kullanıcının bir gruba eklendiği tarihi çekemez, ancak burada grupları listelediğimiz için bunu dahil etmek istedik. Bu bilgi bir saldırgan için çok yararlı değildir. Yine de, Olay Müdahalesine yardımcı olabilecek bilgiler eklemek, raporunuzu öne çıkaracak ve umarım tekrar iş yapmanızı sağlayacaktır. Bu bilgilere sahip olarak, bir grubun parçası olarak garip bir kullanıcı fark ederseniz, savunucular kullanıcıyı kimin eklediğini öğrenmek için o tarihteki Olay Kimliği 4728/ 4738'i veya herhangi birinin oturum açıp açmadığını görmek için eklendiği tarihten bu yana Olay Kimliği 4624'ü arayabilir.

Bu bilgileri çekmek için genellikle kullandığımız modül Get-ADGroupMemberDate olarak adlandırılır ve [buradan](https://raw.githubusercontent.com/proxb/PowerShell_Scripts/master/Get-ADGroupMemberDate.ps1) indirilebilir. Bu modülü PowerView ile aynı şekilde yükleyin.

Ardından Get-ADGroupMemberDate -Group “Help Desk” -DomainController DC01.INLANEFREIGHT.LOCAL komutunu çalıştırın, çekmek istediğiniz belirli bir kullanıcı varsa Get-ADGroupMemberDate -Group “Help Desk” -DomainController DC01.INLANEFREIGHT.LOCAL | ? { ($_.Username -match 'harry.jones') -And ($_.State -NotMatch 'ABSENT') }

Şimdi AD kullanıcıları ve grupları konusunu ele aldık. Şimdi bir şeyleri bir araya getirmeye başlayalım ve domain bilgisayarları ile ilgili bazı temel numaralandırma tekniklerine göz atalım.


### Enumerating AD Computers

Artık kullanıcı ve grup bilgilerini topladığımıza göre, hedef kullanıcılarımızın oturum açabileceği çeşitli hostlar hakkında bilgi edinmemiz ve herhangi bir hostta SYSTEM erişimi elde etmenin farklı saldırı yolları açıp açmayacağını öğrenmemiz gerekir.


### Domain Computer Information
Domain bilgisayarları hakkında birçok ayrıntıyı listelemek için Get-DomainComputer fonksiyonunu kullanabiliriz.

![Pasted image 20241204203819.png](/img/user/resimler/Pasted%20image%2020241204203819.png)

Toplayabileceğimiz en yararlı bilgilerden bazıları host adı, işletim sistemi ve Kullanıcı Hesabı Denetimi (UAC) öznitelikleridir

![Pasted image 20241204203855.png](/img/user/resimler/Pasted%20image%2020241204203855.png)

PowerView kullanarak kayıtlarımız için bu verileri bir CSV'ye kaydedelim.

![Pasted image 20241204203909.png](/img/user/resimler/Pasted%20image%2020241204203909.png)


### İstismar Edilebilir Makineleri Bulma
Yukarıdaki ekran görüntüsündeki en belirgin şey “ User Account Control” ayarıdır ve bu konuya birazdan değineceğiz. Bununla birlikte, Bloodhound gibi araçlar bu ayarı hızlı bir şekilde işaret edecektir ve düzenli sızma testleri yapılan kuruluşlarda bulunması nadir hale gelebilir. Saldırıların ortaya çıkmasına yardımcı olmak için aşağıdaki bayraklar birleştirilebilir:

* LastLogonTimeStamp : Bu alan, yöneticilerin eski makineleri bulmasını sağlamak için vardır. Bu alan bir makine için 90 günlükse, makine açılmamış ve hem işletim sistemi hem de uygulama yamaları eksik demektir. Bu nedenle, yöneticiler bu alan 90 günlük yaşa ulaştığında makineleri otomatik olarak devre dışı bırakmak isteyebilir. Saldırganlar, hedefleri belirlemek için bu alanı İşletim Sistemi veya Oluşturulduğu Zaman gibi diğer alanlarla birlikte kullanabilir.

* OperatingSystem : Bu, İşletim Sistemini listeler. Bariz saldırı yolu, hala aktif olan bir Windows 7 kutusu bulmak (LastLogonTimeStamp) ve Eternal Blue gibi saldırıları denemektir. Eternal Blue uygulanabilir olmasa bile, Windows'un eski sürümleri çalışmak için ideal noktalardır, çünkü eski Windows'larda daha az günlük kaydı / antivirüs özelliği vardır. Windows sürümleri arasındaki farkları bilmek de önemlidir. Örneğin, Windows 10 Enterprise, varsayılan olarak “Credential Guard” (Mimikatz'ın Parolaları Çalmasını Önler) Etkinleştirilmiş olarak gelen tek sürümdür. Yöneticilerin Windows 10 Professional ve Windows 10 Enterprise'da oturum açtığını görürseniz, Professional kutusu hedeflenmelidir.

* WhenCreated : Bu alan, bir makine Active Directory'ye katıldığında oluşturulur. Kutu ne kadar eskiyse, “Standart Yapı ”dan sapma olasılığı o kadar yüksektir. Eski workstationlarda daha zayıf local administration passwords, daha fazla local admins, vulnerable software, daha fazla data vs. olabilir.


### Computer Attacks

Domain'deki herhangi bir bilgisayarın [unconstrained delegation'a](https://adsecurity.org/?p=1667) (kısıtlanmamış temsilci) izin verecek şekilde yapılandırılıp yapılandırılmadığını görebilir ve standart olan bir domain controller bulabiliriz.

![Pasted image 20241204204232.png](/img/user/resimler/Pasted%20image%2020241204204232.png)

Son olarak, [kısıtlı temsilciliğe](https://docs.microsoft.com/en-us/windows-server/security/kerberos/kerberos-constrained-delegation-overview#:~:text=Constrained%20delegation%20gives%20service%20administrators,to%20their%20back%2Dend%20services.) izin vermek için ayarlanmış herhangi bir host olup olmadığını kontrol edebiliriz.

![Pasted image 20241204204305.png](/img/user/resimler/Pasted%20image%2020241204204305.png)


### Yukarı ve İleriye
Şimdi, elde ettiğimiz herhangi bir erişimden nasıl daha fazla yararlanabileceğimizi görmek için domain'de kurulan erişim kontrol listelerini (ACL'ler) inceleyelim.



### Enumerating Domain ACLs


### Access Control Lists (ACLs)
[Access Control List (ACL)](https://docs.microsoft.com/en-us/windows/win32/secauthz/access-control-lists) ayarlarının kendilerine Access Control Entries (ACEs) denir. Her ACE bir kullanıcıya, gruba veya işleme (güvenlik sorumlusu) geri döner ve sorumlunun haklarını tanımlar

İki tür ACL vardır.

![Pasted image 20241204204754.png](/img/user/resimler/Pasted%20image%2020241204204754.png)

ACL (yanlış) yapılandırmaları zincirleme nesneden nesneye kontrole izin verebilir. Bir AD saldırı zincirinden yararlanarak yönetici hakları elde edebilen türemiş yöneticiler olarak adlandırılan hedef grupların kaydırılmamış üyeliğini görselleştirebiliriz.

AD Saldırı zincirleri aşağıdaki bileşenleri içerebilir:

* ACL (yanlış)- Üye sunucular veya workstation'lar üzerinde yönetici erişimine sahip "Yetkisiz" kullanıcılar (shadow admins)
* Bu workstationlarda ve üye sunucularda oturum açma oturumuna sahip ayrıcalıklı kullanıcılar. 
* Nesneden nesneye kontrolün diğer biçimleri arasında parola değişikliğine zorlama, grup üyesi ekleme, sahibi değiştirme, ACE yazma ve tam kontrol yer alır.

Aşağıda, bir kullanıcı nesnesi üzerinde ayarlanabilecek ACL'lerden sadece bazılarına bir örnek verilmiştir.

![Pasted image 20241204204948.png](/img/user/resimler/Pasted%20image%2020241204204948.png)


### ACL İstismarı
ACL'leri neden önemsiyoruz? ACL kötüye kullanımı, sızma testi uzmanları olarak bizim için güçlü bir saldırı vektörüdür. Bu tür yanlış yapılandırmalar kurumsal ortamlarda genellikle fark edilmez çünkü izlenmesi ve kontrol edilmesi zor olabilir. Bir kuruluş, biz onları keşfetmeden önce (umarız) yıllarca aşırı izin verilen ACL ayarlarının farkında olmayabilir. Aşağıda örnek Active Directory nesne güvenlik izinlerinden bazıları verilmiştir (BloodHound tarafından desteklenir ve SharpView / PowerView ile kötüye kullanılabilir):

* ForceChangePassword Set-DomainUserPassword ile kötüye kullanıldı
* Add-DomainGroupMember ile kötüye kullanılan Üyeleri Ekleme
* GenericAll Set-DomainUserPassword veya Add-DomainGroupMember ile kötüye kullanıldı
* Set-DomainObject ile kötüye kullanılan GenericWrite
* WriteOwner Set-DomainObjectOwner ile kötüye kullanıldı
* WriteDACL, Add-DomainObjectACL ile kötüye kullanıldı
* Set-DomainUserPassword veya AddDomainGroupMember ile kötüye kullanılan AllExtendedRights


### ACL'leri Yerleşik Cmdlet'lerle Numaralandırma
ACL'leri numaralandırmak için yerleşik Get-ADUser cmdlet'ini kullanabiliriz. Örneğin, bu komutla tek bir domain kullanıcısı olan daniel.carter için ACL'ye bakabiliriz.

![Pasted image 20241204205149.png](/img/user/resimler/Pasted%20image%2020241204205149.png)

Hedef kullanıcı üzerinde WriteProperty veya GenericAll haklarına sahip tüm kullanıcıları bulmak için bu kullanıcıyı daha ayrıntılı inceleyebiliriz.

![Pasted image 20241204205232.png](/img/user/resimler/Pasted%20image%2020241204205232.png)
![Pasted image 20241204205243.png](/img/user/resimler/Pasted%20image%2020241204205243.png)
![Pasted image 20241204205249.png](/img/user/resimler/Pasted%20image%2020241204205249.png)


### PowerView ve SharpView ile ACL'leri numaralandırma
Bir önceki komutu çok daha hızlı gerçekleştirmek için PowerView / SharpView kullanabiliriz. Örneğin, Get-DomainObjectACL benzer verileri döndürmek için bir kullanıcı üzerinde kullanılabilir.

![Pasted image 20241204205324.png](/img/user/resimler/Pasted%20image%2020241204205324.png)
![Pasted image 20241204205335.png](/img/user/resimler/Pasted%20image%2020241204205335.png)
![Pasted image 20241204205343.png](/img/user/resimler/Pasted%20image%2020241204205343.png)

Belirli kullanıcılar üzerindeki ACL'leri arayabilir ve Active Directory LDAP modülünde ele alınan çeşitli AD filtrelerini kullanarak sonuçları filtreleyebiliriz. Domain'de built-in olmayan nesneler üzerinde değişiklik haklarına sahip nesneleri aramak için FindInterestingDomainAcl komutunu kullanabiliriz. Bu komut da büyük miktarda veri üretir ve belirli nesneler hakkında bilgi almak için filtrelenebilir ya da çevrimdışı olarak incelenmek üzere kaydedilebilir.

![Pasted image 20241204205428.png](/img/user/resimler/Pasted%20image%2020241204205428.png)
![Pasted image 20241204205440.png](/img/user/resimler/Pasted%20image%2020241204205440.png)
![Pasted image 20241204205446.png](/img/user/resimler/Pasted%20image%2020241204205446.png)
Kullanıcılardan ve bilgisayarlardan Aside'ı arayabiliriz, ayrıca dosya paylaşımlarında ayarlanan ACL'lere de bakmalıyız. Bu bize hangi kullanıcıların belirli bir paylaşıma erişebileceği veya belirli bir paylaşımda izinlerin çok gevşek ayarlandığı hakkında bilgi sağlayabilir, bu da hassas verilerin ifşasına veya diğer saldırılara yol açabilir.

![Pasted image 20241204210056.png](/img/user/resimler/Pasted%20image%2020241204210056.png)

![Pasted image 20241204210104.png](/img/user/resimler/Pasted%20image%2020241204210104.png)
![Pasted image 20241204210111.png](/img/user/resimler/Pasted%20image%2020241204210111.png)
![Pasted image 20241204210116.png](/img/user/resimler/Pasted%20image%2020241204210116.png)


Belirli kullanıcıların ve bilgisayarların bize tam kontrol veya başka izinler verebilecek ACL'lerinin yanı sıra, domain nesnesinin ACL'sini de kontrol etmeliyiz. [DCSync](https://adsecurity.org/?p=1729) adı verilen yaygın bir saldırı, bir kullanıcıya aşağıdaki üç hakkın bir kombinasyonunun verilmesini gerektirir:

* Dizin Değişikliklerini Çoğaltma (DS-Replication-Get-Changes)
* Dizin Değişikliklerinin Tümünü Çoğaltma (DS-Replication-Get-Changes-All)
* Filtrelenmiş Kümedeki Dizin Değişikliklerini Çoğaltma (DS-Replication-Get-Changes-In-FilteredSet)

bu haklara sahip tüm kullanıcıları aramak için Get-ObjectACL işlevini kullanabiliriz.

![Pasted image 20241204210241.png](/img/user/resimler/Pasted%20image%2020241204210241.png)

SID'leri aldıktan sonra, hangi hesapların bu haklara sahip olduğunu görmek ve bunun amaçlanıp amaçlanmadığını ve/veya bu hakları kötüye kullanıp kullanamayacağımızı belirlemek için SID'yi kullanıcıya geri dönüştürebiliriz.

![Pasted image 20241204210257.png](/img/user/resimler/Pasted%20image%2020241204210257.png)
![Pasted image 20241204210301.png](/img/user/resimler/Pasted%20image%2020241204210301.png)

Bu, bu hakka sahip tüm kullanıcıları listelemek için hızlı bir şekilde yapılabilir.


![Pasted image 20241204210314.png](/img/user/resimler/Pasted%20image%2020241204210314.png)


### ACL'lerden Yararlanma

Bu bölümde görüldüğü gibi, AD içinde çeşitli ACE girişleri ayarlanabilir. Yöneticiler, bir nesne veya nesne kümesi üzerinde ayrıntılı ayrıcalıklar vermek için bazılarını kasıtlı olarak ayarlayabilir. Buna karşılık, diğerleri yanlış yapılandırmalardan veya varsayılan olarak domain içinde ACL'lerde birçok değişiklik yapan Exchange gibi bir hizmetin yüklenmesinden kaynaklanabilir.

Bir kullanıcı ya da grup üzerinde GenericWrite özelliğine sahip bir kullanıcıyı tehlikeye atabilir ve bundan yararlanarak bir kullanıcının parolasını değiştirmeye zorlayabilir ya da hesabımızı belirli bir gruba ekleyerek erişimimizi artırabiliriz. Bu gibi değişiklikler dikkatlice not edilmeli ve nihai raporda belirtilmelidir, böylece müşteri değerlendirme süresi boyunca yapamazsak değişikliklerin geri alındığından emin olabilir. Ayrıca, bir kullanıcının şifresini değiştirmek gibi “yıkıcı” bir eylem, aksaklıkları önlemek için idareli bir şekilde kullanılmalı ve müşteriyle koordine edilmelidir.

Bir nesne üzerinde WriteDacl ayrıcalıklarına sahip bir kullanıcı, grup veya bilgisayar bulursak, bundan çeşitli şekillerde yararlanabiliriz. Örneğin, Exchange Trusted Subsystem gibi Exchange ile ilgili bir grubun bir üyesini ele geçirebilirsek, muhtemelen domain nesnesinin kendisi üzerinde WriteDacl ayrıcalıklarına sahip olacağız ve kontrol ettiğimiz bir hesaba Replicating Directory Changes ve Replicating Directory Change izinlerini verebilecek ve seçtiğimiz herhangi bir hesap için kullanıcı NTLM parola karmalarını almak üzere bir Domain Controller'ı taklit ederek domain'i tamamen ele geçirmek için bir DCSync saldırısı gerçekleştirebileceğiz.

Kendimizi bir hedef kullanıcı üzerinde GenericAll / GenericWrite ayrıcalıklarına sahip bulursak, hesapta sahte bir SPN ayarlamak ve hedefli bir Kerberoasting saldırısı gerçekleştirmek veya hesabın userAccountControl'ünü Kerberos ön kimlik doğrulaması gerektirmeyecek şekilde değiştirmek ve hedefli bir ASREPRoasting saldırısı gerçekleştirmek daha az yıkıcı bir saldırı olacaktır. Bu örnekler, hesabın Hashcat gibi bir araç kullanılarak çevrimdışı olarak minimum çabayla kırılabilecek zayıf bir parola kullanmasını gerektirir, ancak bir kullanıcının parolasını değiştirmekten çok daha az yıkıcıdır ve fark edilmeme olasılığı daha yüksektir.

Bir kullanıcının parolasını değiştirmek gibi yıkıcı bir eylem gerçekleştirirseniz ve domainin güvenliğini tehlikeye atabilirseniz, DCSync yapabilir, hesabın parola geçmişini elde edebilir ve LSADUMP::ChangeNTLM veya LSADUMP::SetNTLM kullanarak hesabı önceki parolaya sıfırlamak için Mimikatz'ı kullanabilirsiniz.

Kendimizi bir bilgisayarda GenericAll / GenericWrite ile bulursak, Kerberos Resource-based Constrained Delegation saldırısı gerçekleştirebiliriz.

Bazen bir kullanıcıya veya hatta tüm Domain Users grubuna belirli bir grup ilkesi nesnesi üzerinde yazma izinleri verildiğini görürüz. Bu tür bir yanlış yapılandırma bulursak ve GPO bir veya daha fazla kullanıcı veya bilgisayarla bağlantılıysa, bir kullanıcıya ek ayrıcalıklar sağlamak (hedeflenen kimlik bilgisi hırsızlığını gerçekleştirebilmek için SeDebugPrivilege veya hassas bir dosya veya dosya paylaşımı üzerinde kontrol elde etmek için SeTakeOwnershipPrivilege gibi), hedef ana bilgisayara yerel yönetici olarak kontrol ettiğimiz bir kullanıcı eklemek, bilgisayar başlatma komut dosyası eklemek ve daha fazlası gibi eylemleri gerçekleştirmek üzere hedef GPO'yu değiştirmek için [SharpGPOAbuse](https://github.com/FSecureLABS/SharpGPOAbuse) gibi bir araç kullanabiliriz. Yukarıda tartışıldığı gibi, bu değişiklikler müşteri ile koordinasyon içinde dikkatlice yapılmalı ve aksaklıkları en aza indirmek için nihai raporda belirtilmelidir.

Bu, ACL'leri kötüye kullanmak için sahip olduğumuz birçok seçeneğin bir özetidir. Bu konu daha sonraki modüllerde daha derinlemesine ele alınacaktır.


### Özet
ACL'ler AD güvenliğinin genellikle göz ardı edilen bir alanıdır, ancak burada gördüğümüz gibi domain ortamındaki nesneler üzerinde güçlü amaçlanmış ve amaçlanmamış haklar sağlayabilirler. Küçük bir AD ağında bile binlerce ACL vardır, bu nedenle yararlı verileri ortaya çıkarmak için aramalarımızı hedefe yönelik yapmalıyız. Daha sonra, Group Policy Objects'e (GPO'lar) bir göz atacağız.


### Enumerating Group Policy Objects (GPOs)
