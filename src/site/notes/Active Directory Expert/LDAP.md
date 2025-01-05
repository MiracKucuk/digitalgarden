---
{"dg-publish":true,"permalink":"/active-directory-expert/ldap/"}
---

AD esasen, ayrıcalık düzeylerine bakılmaksızın domain içindeki tüm kullanıcıların erişebildiği büyük bir veritabanıdır. Ek ayrıcalıkları olmayan temel bir AD kullanıcı hesabı, aşağıdakiler dahil ancak bunlarla sınırlı olmamak üzere AD'de bulunan nesnelerin çoğunu numaralandırmak için kullanılabilir:

* Domain Computers 
* Domain Users 
* Domain Group Information 
* Default Domain Policy 
* Domain Functional Levels 
* Password Policy 
* Group Policy Objects (GPOs) 
* Kerberos Delegation 
* Domain Trusts 
* Access Control Lists (ACLs)

AD'deki yanlış yapılandırmalardan, kötü uygulamalardan veya kötü yönetimden yararlanan birçok saldırı mevcuttur, örneğin:

* Kerberoasting / ASREPRoasting 
* NTLM Relaying 
* Network traffic poisoning 
* Password spraying
* Kerberos delegation abuse
* Domain trust abuse
* Credential theft 
* Object control

 
### Active Directory Structure

Active Directory hiyerarşik bir ağaç yapısında düzenlenmiştir; en üstte bir veya daha fazla domain içeren ve kendileri de iç içe geçmiş subdomainler içerebilen bir forest bulunur. Forest, tüm nesnelerin yönetim kontrolü altında olduğu güvenlik sınırıdır. Bir forest birden fazla domain içerebilir ve bir domain başka alt veya alt domainler içerebilir. Domain, içerdiği nesnelere (kullanıcılar, bilgisayarlar ve gruplar) erişilebilen bir yapıdır. Nesneler AD'deki en temel veri birimidir

“ Domain Controllers”, “Users” ve “Computers” gibi birçok yerleşik Organizasyonel Birim ( OU) içerir ve gerektiğinde yeni OU'lar oluşturulabilir. OU'lar, farklı grup ilkelerinin atanmasına olanak tanıyan nesneler ve alt OU'lar içerebilir.

![Pasted image 20241203213234.png](/img/user/resimler/Pasted%20image%2020241203213234.png)

Bu yapıyı bir Domain Controller üzerinde Active Directory Users and Computers'ı açarak grafiksel olarak görebiliriz. Laboratuvar domainimiz INLANEFREIGHT.LOCAL'de Admin, Employees, Servers, Workstations gibi çeşitli OU'lar görürüz. Bu OU'ların birçoğu, Employees altındaki Mail Room OU gibi iç içe geçmiş OU'lara sahiptir. Bu, Active Directory içinde net ve tutarlı bir yapının korunmasına yardımcı olur; bu, domain genelinde ayarları zorlamak için özellikle Grup Politikası Nesneleri (GPO'lar) eklediğimizde önemlidir.

![Pasted image 20241203213424.png](/img/user/resimler/Pasted%20image%2020241203213424.png)

Active Directory'nin yapısını anlamak, doğru numaralandırma yapmak ve bazen uzun yıllar boyunca bir ortamda gözden kaçan kusurları ve yanlış yapılandırmaları ortaya çıkarmak için çok önemlidir


### Module Exercises

Bu modül boyunca, alıştırmaları tamamlamak için Remote Desktop Protocol (RDP) aracılığıyla çeşitli hedef hostlara bağlanacaksınız. Gerekli kimlik bilgileri her alıştırma ile birlikte verilecektir ve RDP bağlantısı Pwnbox'tan xfreerdp aracılığıyla aşağıdaki şekilde yapılabilir:

![Pasted image 20241203213519.png](/img/user/resimler/Pasted%20image%2020241203213519.png)

`/cert-ignore`, bu tür sertifika hatalarını yok sayar ve bağlantının devam etmesini sağlar. Bu, özellikle test ortamlarında veya bilinmeyen sertifikalarla çalışan sistemlerde faydalıdır.

Gerekli araçlar, hedef hostta oturum açtıktan sonra c:\tools dizininde bulunabilir


### Başlarken

Bir AD ortamında yerimizi aldıktan sonra, aşağıdakiler dahil ancak bunlarla sınırlı olmamak üzere birkaç temel bilgi toplayarak işe başlamalıyız:

* Domain fonksiyonel seviyesi
* Domain parola ilkesi
* AD kullanıcılarının tam bir envanteri
* AD bilgisayarlarının tam bir envanteri
* AD gruplarının ve üyeliklerinin tam bir envanteri
* Domain güven ilişkileri
* Object ACL'ler
* Grup İlkesi Nesneleri (GPO) bilgileri
* Uzaktan erişim hakları

Elimizdeki bu bilgilerle, mevcut kullanıcımızın veya tüm Domain Users grubunun bir veya daha fazla host'a RDP ve/veya local administrator erişimi olması gibi “hızlı kazanımlar” arayabiliriz. Bu, büyük ortamlarda birçok nedenden dolayı yaygındır; bunlardan biri jump host'ların yanlış kullanımı, diğeri ise Citrix sunucusu Remote Desktop Services (RDS) yanlış yapılandırmalarıdır. Mevcut kullanıcımızın domain içinde hangi haklara sahip olduğunu da kontrol etmeliyiz. Herhangi bir ayrıcalıklı gruba üye mi? Devredilmiş herhangi bir özel hakları var mı? Kullanıcı, bilgisayar veya GPO gibi başka bir domain nesnesi üzerinde herhangi bir kontrole sahipler mi?


### Rights and Privileges in AD

AD, üyelerine güçlü haklar ve ayrıcalıklar veren birçok grup içerir. Bunların birçoğu, bir domain içindeki ayrıcalıkları yükseltmek ve nihayetinde bir Domain Controller (DC) üzerinde Domain Admin veya SYSTEM ayrıcalıkları elde etmek için kötüye kullanılabilir. Bu gruplardan bazıları aşağıda listelenmiştir.

![Pasted image 20241203215938.png](/img/user/resimler/Pasted%20image%2020241203215938.png)
![Pasted image 20241203215947.png](/img/user/resimler/Pasted%20image%2020241203215947.png)
![Pasted image 20241203215959.png](/img/user/resimler/Pasted%20image%2020241203215959.png)



### “Schema Admins” üyeleri
![Pasted image 20241203220102.png](/img/user/resimler/Pasted%20image%2020241203220102.png)
![Pasted image 20241203220118.png](/img/user/resimler/Pasted%20image%2020241203220118.png)
![Pasted image 20241203220242.png](/img/user/resimler/Pasted%20image%2020241203220242.png)



### Kullanıcı Hakları Ataması

Grup üyeliğine ve Grup İlkesi aracılığıyla atanan ayrıcalıklar gibi diğer faktörlere bağlı olarak, kullanıcıların hesaplarına çeşitli haklar atanabilir. [Kullanıcı Hakları Ataması](https://docs.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/user-rights-assignment) hakkındaki bu Microsoft makalesi, Windows'ta ayarlanabilen kullanıcı haklarının her biri hakkında ayrıntılı bir açıklama sağlar.

whoami /priv komutunu yazdığınızda, mevcut kullanıcınıza atanan tüm kullanıcı haklarının bir listesini alırsınız. Bazı haklar yalnızca yönetici kullanıcılar tarafından kullanılabilir ve yalnızca yükseltilmiş bir cmd veya PowerShell oturumu çalıştırıldığında listelenebilir/alınabilir. Bu yükseltilmiş haklar ve [Kullanıcı Hesabı Denetimi (UAC)](https://docs.microsoft.com/en-us/windows/security/identity-protection/user-account-control/how-user-account-control-works) kavramları, Windows Vista ile getirilen ve uygulamaların kesinlikle gerekli olmadıkça tam izinlerle çalışmasını kısıtlayan güvenlik özellikleridir. Yükseltilmemiş bir konsolda ve yükseltilmiş bir konsolda yönetici olarak kullanabileceğimiz hakları karşılaştırırsak, bunların büyük ölçüde farklı olduğunu görürüz. Bunu laboratuvar makinesinde htb-student kullanıcısı olarak deneyelim.

Aşağıda bir Domain Admin kullanıcısının sahip olduğu haklar yer almaktadır.


### Kullanıcı Hakları Yükseltilmemiş
Yükseltilmemiş bir konsolda aşağıdakileri görebiliriz:
![Pasted image 20241203220613.png](/img/user/resimler/Pasted%20image%2020241203220613.png)
![Pasted image 20241203220620.png](/img/user/resimler/Pasted%20image%2020241203220620.png)


### Kullanıcı Hakları Yükseltildi
Yükseltilmiş bir komut çalıştırırsak (htb-student kullanıcımız iç içe grup üyeliği yoluyla local admin haklarına sahiptir; Domain Users grubu local Administrators grubundadır), kullanabileceğimiz hakların tam listesini görebiliriz:

![Pasted image 20241203220740.png](/img/user/resimler/Pasted%20image%2020241203220740.png)
![Pasted image 20241203220751.png](/img/user/resimler/Pasted%20image%2020241203220751.png)
![Pasted image 20241203220802.png](/img/user/resimler/Pasted%20image%2020241203220802.png)

Buna karşın, standart bir domain kullanıcısı çok daha az hakka sahiptir.


### Domain User Rights
![Pasted image 20241203220827.png](/img/user/resimler/Pasted%20image%2020241203220827.png)
![Pasted image 20241203220833.png](/img/user/resimler/Pasted%20image%2020241203220833.png)

Kullanıcı hakları, yerleştirildikleri gruplara ve/veya atanan ayrıcalıklara bağlı olarak artar. Aşağıda Backup Operators grubundaki kullanıcılara verilen haklara bir örnek verilmiştir. Bu gruptaki kullanıcılar şu anda UAC tarafından kısıtlanan başka haklara da sahiptir. Yine de, bu komuttan SeShutdownPrivilege'a sahip olduklarını görebiliriz, bu da bir domain controller'da local olarak oturum açmaları durumunda (RDP veya WinRM aracılığıyla değil) büyük bir hizmet kesintisine neden olabilecek bir domain controller'ı kapatabilecekleri anlamına gelir.


### Backup Operator Rights
![Pasted image 20241203220938.png](/img/user/resimler/Pasted%20image%2020241203220938.png)

Saldırganlar ve savunmacılar olarak bu grupların üyeliklerini gözden geçirmemiz gerekir. Domain'e daha fazla erişim sağlamak veya domain'i tehlikeye atmak için kullanılabilecek bu gruplardan birine veya daha fazlasına eklenmiş, görünüşte düşük ayrıcalıklı kullanıcılar bulmak nadir değildir.


### Microsoft Remote Server Yönetim Araçları (RSAT)

### **RSAT Background**

Remote Server Administration Tools ( RSAT ), Windows 2000 günlerinden beri Windows'un bir parçasıdır. RSAT, sistem yöneticilerinin Windows 10, Windows 8.1, Windows 7 veya Windows Vista çalıştıran bir workstation'dan Windows Server rollerini ve özelliklerini uzaktan yönetebilmelerini sağlar. RSAT yalnızca Windows'un Professional veya Enterprise sürümlerine kurulabilir. Kurumsal bir ortamda RSAT, Active Directory, DNS ve DHCP'yi uzaktan yönetebilir. RSAT ayrıca yüklü sunucu rollerini ve özelliklerini, Dosya Hizmetlerini ve Hyper-V'yi yönetmemizi sağlar. RSAT ile birlikte gelen araçların tam listesi:

![Pasted image 20241203221356.png](/img/user/resimler/Pasted%20image%2020241203221356.png)
![Pasted image 20241203221422.png](/img/user/resimler/Pasted%20image%2020241203221422.png)

Bu [komut](https://gist.github.com/dually8/558fcfa9156f59504ab36615dfc4856a) dosyası RSAT'ı Windows 10 1809, 1903 ve 1909'a yüklemek için kullanılabilir. Windows'un diğer sürümleri için kurulum talimatlarının yanı sıra RSAT hakkında ek bilgileri [burada](https://support.microsoft.com/en-us/help/2693643/remote-server-administration-tools-rsat-for-windows-operating-systems) bulabilirsiniz. RSAT, PowerShell ile de kolayca kurulabilir.

PowerShell kullanarak varsa hangi RSAT araçlarının yüklü olduğunu kontrol edebiliriz.


### PowerShell - Mevcut RSAT Araçları
![Pasted image 20241203222032.png](/img/user/resimler/Pasted%20image%2020241203222032.png)

Buradan, aşağıdaki komutu kullanarak mevcut tüm araçları yüklemeyi seçebiliriz:

### PowerShell - Mevcut Tüm RSAT Araçlarını Yükleyin
![Pasted image 20241203222119.png](/img/user/resimler/Pasted%20image%2020241203222119.png)

Gerektiğinde aletleri teker teker de kurabiliriz.


### PowerShell - RSAT Aracı Yükleme
![Pasted image 20241203222355.png](/img/user/resimler/Pasted%20image%2020241203222355.png)
Kurulduktan sonra, tüm araçlar Control Panel'deki Administrative Tools (Yönetimsel Araçlar) altında bulunacaktır.

![Pasted image 20241203222418.png](/img/user/resimler/Pasted%20image%2020241203222418.png)

![Pasted image 20241203222643.png](/img/user/resimler/Pasted%20image%2020241203222643.png)


### Numaralandırma için Domain Context

Birçok araçta kimlik bilgisi ve context parametreleri eksiktir ve bunun yerine bu değerleri doğrudan geçerli context'ten alırlar. Bir parolaya veya hash'e erişiminiz varsa, Windows'ta bir kullanıcının context'ini değiştirmenin birkaç yolu vardır, örneğin:

Yerleşik [runas.exe](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) komut satırı aracından yararlanmak için “ runas /netonly ” kullanımı


### CMD - Runas User
![Pasted image 20241203222929.png](/img/user/resimler/Pasted%20image%2020241203222929.png)

Daha sonraki modüllerde tartışacağımız Rubeus ve mimikatz gibi diğer araçlara açık metin kimlik bilgileri veya bir NTLM parola hash'i aktarılabilir.

`runas` komutu, Windows'ta bir uygulamayı başka bir kullanıcı hesabıyla çalıştırmak için kullanılır. Bu komut, özellikle farklı yetkilere sahip kullanıcı hesaplarıyla işlem yapmak gerektiğinde oldukça faydalıdır. Örneğin, bir programı yönetici yetkileriyle çalıştırabilir veya başka bir kullanıcı hesabı altında bir uygulama başlatabilirsiniz.

![Pasted image 20241203223718.png](/img/user/resimler/Pasted%20image%2020241203223718.png)

- **`/user:<kullanıcı_adı>`**: Hangi kullanıcıyla çalıştırmak istediğinizi belirtir.
- **`<uygulanacak_komut>`**: Çalıştırılacak komut veya uygulama.


### CMD - Rubeus.exe Açık Metin Kimlik Bilgileri
![Pasted image 20241203223139.png](/img/user/resimler/Pasted%20image%2020241203223139.png)


### CMD - Mimikatz.exe Cleartext Kimlik Bilgileri
![Pasted image 20241203223543.png](/img/user/resimler/Pasted%20image%2020241203223543.png)


### RSAT ile numaralandırma
Domain'e bağlı bir sistemi tehlikeye atarsak (veya bir müşteri workstationlarda birinden AD değerlendirmesi yapmanızı isterse), AD'yi numaralandırmak için RSAT'tan yararlanabiliriz. RSAT, Active Directory Users and Computers ve ADSI Edit gibi GUI araçlarını kullanımımıza sunacak olsa da, bu modül boyunca gördüğümüz en önemli araç PowerShell Active Directory modülüdür.

Alternatif olarak, komut satırından “ runas ” kullanarak herhangi bir RSAT ek bileşenini başlatarak domain'e katılmamış bir hosttan (bir domain controller ile iletişim kuran bir alt ağda olması şartıyla) domain'i numaralandırabiliriz. Bu, özellikle bir iç değerlendirme gerçekleştiriyorsak, geçerli AD kimlik bilgilerine sahipsek ve bir Windows sanal makinesinden numaralandırma yapmak istiyorsak kullanışlıdır.

![Pasted image 20241203224028.png](/img/user/resimler/Pasted%20image%2020241203224028.png)

MMC Konsolunu, aşağıdaki komut sözdizimini kullanarak domain'e bağlı olmayan bir bilgisayardan da açabiliriz:


### CMD - MMC Domain Kullanıcısı Olarak Çalıştır

![Pasted image 20241203224114.png](/img/user/resimler/Pasted%20image%2020241203224114.png)

![Pasted image 20241203224127.png](/img/user/resimler/Pasted%20image%2020241203224127.png)

RSAT ek bileşenlerinden herhangi birini ekleyebilir ve freightlogistics.local domainindeki hedef kullanıcı sally.jones context'inde hedef domain'i numaralandırabiliriz. Ek bileşenleri ekledikten sonra, “belirtilen domain ya mevcut değil ya da iletişime geçilemedi” şeklinde bir hata mesajı alacağız. Buradan, Active Directory Kullanıcıları ve Bilgisayarları ek bileşenine (veya seçilen başka bir ek bileşene) sağ tıklayıp Domain Değiştir'i seçmemiz gerekir.

![Pasted image 20241203225518.png](/img/user/resimler/Pasted%20image%2020241203225518.png)

Domain değiştir diyalog kutusuna hedef domain'i yazın, burada freightlogistics.local . Buradan, artık AD RSAT ek bileşenlerinden herhangi birini kullanarak domaini serbestçe numaralandırabiliriz.

![Pasted image 20241203225553.png](/img/user/resimler/Pasted%20image%2020241203225553.png)

Bu grafiksel araçlar kullanışlı ve kullanımı kolay olsa da, büyük bir domain'i numaralandırmaya çalışırken çok verimsizdirler. Önümüzdeki birkaç bölümde, PowerShell kullanarak AD'yi numaralandırmak için kullanabileceğimiz LDAP ve çeşitli arama filtreleri türlerini tanıtacağız. Bu bölümlerde ele aldığımız konular, AD'nin nasıl çalıştığını ve bilgilerin verimli bir şekilde nasıl aranacağını daha iyi anlamamıza yardımcı olacak ve sonuçta sonraki iki AD Numaralandırma modülünde ele alacağımız daha “otomatik” araçların ve komut dosyalarının kullanımı konusunda bizi daha iyi bilgilendirecektir.


### The Power of NT AUTHORITY\SYSTEM

[LocalSystem](https://learn.microsoft.com/en-us/windows/win32/services/localsystem-account) hesabı NT AUTHORITY\SYSTEM, Windows işletim sistemlerinde servis kontrol yöneticisi tarafından kullanılan built-in bir hesaptır. İşletim sistemindeki en yüksek erişim düzeyine sahiptir (ve Trusted Installer ayrıcalıklarıyla daha da güçlü hale getirilebilir). Bu hesap lokal bir yönetici hesabından daha fazla ayrıcalığa sahiptir ve çoğu Windows hizmetini çalıştırmak için kullanılır. Üçüncü taraf hizmetleri varsayılan olarak bu hesap bağlamında çalıştırmak da çok yaygındır. SYSTEM hesabı aşağıdaki [ayrıcalıklara](https://docs.microsoft.com/en-us/windows/win32/secauthz/privilege-constants) sahiptir:

![Pasted image 20241203225943.png](/img/user/resimler/Pasted%20image%2020241203225943.png)
![Pasted image 20241203225955.png](/img/user/resimler/Pasted%20image%2020241203225955.png)
![Pasted image 20241203230002.png](/img/user/resimler/Pasted%20image%2020241203230002.png)

LocalSystem hesabı NT AUTHORITY\SYSTEM, Windows işletim sistemlerinde servis kontrol yöneticisi tarafından kullanılan built-in bir hesaptır. İşletim sistemindeki en yüksek erişim düzeyine sahiptir (ve Trusted Installer ayrıcalıklarıyla daha da güçlü hale getirilebilir). Bu hesap lokal bir yönetici hesabından daha fazla ayrıcalığa sahiptir ve çoğu Windows hizmetini çalıştırmak için kullanılır. Üçüncü taraf hizmetleri varsayılan olarak bu hesap bağlamında çalıştırmak da çok yaygındır. SYSTEM hesabı aşağıdaki ayrıcalıklara sahiptir:

Bir host üzerinde SYSTEM düzeyinde erişim elde etmenin, bunlarla sınırlı olmamak üzere, çeşitli yolları vardır:

* EternalBlue veya BlueKeep gibi uzak Windows açıkları.
* SYSTEM hesabı bağlamında çalışan bir hizmetin kötüye kullanılması.
* Eski Windows sistemlerine karşı [RottenPotatoNG](https://github.com/breenmachine/RottenPotatoNG), [Juicy Potato](https://github.com/ohpe/juicy-potato) veya [Windows 10/Windows Server 2019'u](https://itm4n.github.io/printspoofer-abusing-impersonate-privileges/) hedefliyorsa [PrintSpoofer](https://github.com/itm4n/PrintSpoofer) kullanarak SeImpersonate ayrıcalıklarını kötüye kullanma.
* [Windows 10 Task Scheduler](https://blog.0patch.com/2019/06/another-task-scheduler-0day-another.html) 0day gibi Windows işletim sistemlerinde local privilege escalation kusurları.
* PsExec -s bayrağı ile

Domain ile birleştirilmiş bir hostlarda SYSTEM düzeyinde erişim elde ederek şunları yapabileceğiz:

* BloodHound ve PowerView / SharpView kullanarak domain'i numaralandırın ve domain kullanıcıları ve grupları, local administrator erişimi, domain trust'ları, ACL'ler, kullanıcı ve bilgisayar özellikleri vb. hakkında bilgi toplayın.
* Kerberoasting / ASREPRoasting saldırıları gerçekleştirin.
* Net-NTLM-v2 hash'lerini toplamak veya relay saldırıları gerçekleştirmek için [Inveigh](https://github.com/Kevin-Robertson/Inveigh) gibi araçları çalıştırın
* Ayrıcalıklı bir domain kullanıcı hesabını ele geçirmek için token kimliğine bürünme gerçekleştirin.
* ACL saldırıları gerçekleştirme

### LDAP'ye Genel Bakış
[Lightweight Directory Access Protocol ( LDAP )](https://en.wikipedia.org/wiki/Lightweight_Directory_Access_Protocol) Active Directory'nin (AD) ayrılmaz bir parçasıdır. En son LDAP spesifikasyonu RFC [4511 ](https://datatracker.ietf.org/doc/html/rfc4511)olarak yayınlanan Sürüm 3'tür. LDAP'ın AD ortamında nasıl çalıştığını anlamak hem saldırganlar hem de savunmacılar için çok önemlidir

LDAP, çeşitli dizin hizmetlerine (AD gibi) karşı kimlik doğrulama için kullanılan açık kaynaklı ve platformlar arası bir protokoldür. Önceki bölümde tartışıldığı gibi AD, kullanıcı hesap bilgilerini ve parolalar gibi güvenlik bilgilerini depolar ve bu bilgilerin ağdaki diğer cihazlarla paylaşılmasını kolaylaştırır. LDAP, uygulamaların dizin hizmetleri de sağlayan diğer sunucularla iletişim kurmak için kullandığı dildir. Başka bir deyişle, LDAP ağ ortamındaki sistemlerin AD ile “konuşabilmesinin” bir yoludur.

Bir LDAP oturumu ilk olarak Dizin Sistemi Aracısı olarak da bilinen bir LDAP sunucusuna bağlanarak başlar. AD'deki Domain Controller, güvenlik kimlik doğrulama istekleri gibi LDAP isteklerini aktif olarak dinler.

![Pasted image 20241204010231.png](/img/user/resimler/Pasted%20image%2020241204010231.png)

AD ve LDAP arasındaki ilişki Apache ve HTTP ile karşılaştırılabilir. Apache'nin HTTP protokolünü kullanan bir web sunucusu olması gibi, Active Directory de LDAP protokolünü kullanan bir dizin sunucusudur.

Nadiren de olsa, bir değerlendirme yaparken AD'ye sahip olmayan ancak LDAP'ye sahip olan kuruluşlarla karşılaşabilirsiniz, bu da büyük olasılıkla [OpenLDAP](https://en.wikipedia.org/wiki/OpenLDAP) gibi başka bir LDAP sunucusu kullandıkları anlamına gelir


### AD LDAP Authentication

LDAP, bir LDAP oturumu için kimlik doğrulama durumunu ayarlamak üzere bir “BIND” işlemi kullanarak kimlik bilgilerini AD'ye karşı doğrulamak üzere ayarlanmıştır. İki tür LDAP kimlik doğrulaması vardır.

==Simple Authentication==: Bu, anonim kimlik doğrulama, kimliği doğrulanmamış kimlik doğrulama ve kullanıcı adı/parola kimlik doğrulamasını içerir. Basit kimlik doğrulama, bir kullanıcı adı ve parolanın LDAP sunucusuna kimlik doğrulaması yapmak için bir BIND isteği oluşturması anlamına gelir

==SASL Kimlik Doğrulaması==: [Basit Kimlik Doğrulama ve Güvenlik Katmanı](https://en.wikipedia.org/wiki/Simple_Authentication_and_Security_Layer) (SASL) framework'ü, LDAP sunucusuna bağlanmak için Kerberos gibi diğer kimlik doğrulama hizmetlerini kullanır ve ardından LDAP'ye kimlik doğrulaması yapmak için bu kimlik doğrulama hizmetini (bu örnekte Kerberos) kullanır. LDAP sunucusu, yetkilendirme hizmetine bir LDAP mesajı göndermek için LDAP protokolünü kullanır ve bu da başarılı ya da başarısız kimlik doğrulama ile sonuçlanan bir dizi meydan okuma/yanıt mesajı başlatır. SASL, kimlik doğrulama yöntemlerinin uygulama protokollerinden ayrılması nedeniyle daha fazla güvenlik sağlayabilir.

LDAP kimlik doğrulama mesajları varsayılan olarak açık metin olarak gönderilir, böylece herhangi biri dahili ağdaki LDAP mesajlarını koklayabilir. Bu bilgileri aktarım sırasında korumak için TLS şifrelemesi veya benzerinin kullanılması önerilir.


### LDAP Sorguları

Hizmetten bilgi istemek için LDAP sorgularını kullanarak dizin hizmetiyle iletişim kurabiliriz. Örneğin, bir ağdaki tüm workstation'ları (objectCategory=computer) bulmak için aşağıdaki sorgu kullanılabilirken, tüm domain controller'ları bulmak için şu sorgu kullanılabilir: (&(objectCategory=Computer) (userAccountControl:1.2.840.113556.1.4.803:=8192)) .

LDAP sorguları, tüm kullanıcıları arayan “ (& (objectCategory=person)(objectClass=user)) ‘ gibi kullanıcıyla ilgili aramaların yanı sıra tüm grupları döndüren ’ (objectClass=group) ” gibi grupla ilgili aramaları gerçekleştirmek için kullanılabilir. Aşağıda “ Get-ADObject ‘ cmdlet'ini ve ’ LDAPFilter parametresini ” kullanarak tüm AD gruplarını bulmak için basit bir sorgu örneği verilmiştir.


### LDAP Sorgusu - Kullanıcı ile İlgili Arama
![Pasted image 20241204011540.png](/img/user/resimler/Pasted%20image%2020241204011540.png)

Daha ayrıntılı aramalar yapmak için LDAP sorgularını da kullanabiliriz. Bu sorgu, domain'de yönetimsel (administratively) olarak devre dışı bırakılmış tüm hesapları arar.


### LDAP Sorgusu - Detaylı Arama
![Pasted image 20241204011649.png](/img/user/resimler/Pasted%20image%2020241204011649.png)
![Pasted image 20241204011657.png](/img/user/resimler/Pasted%20image%2020241204011657.png)

AD için temel ve daha gelişmiş LDAP sorgularına ilişkin daha fazla örnek aşağıdaki bağlantılarda bulunabilir:

* AD [bilgisayarlarıyla](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20Computer%20Related%20LDAP%20Query) ilgili LDAP sorguları
* AD [kullanıcılarıyla](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20User%20Related%20Searches) ilgili LDAP sorguları
* AD [gruplarıyla](https://ldapwiki.com/wiki/Wiki.jsp?page=Active%20Directory%20Group%20Related%20Searches) ilgili LDAP sorguları

LDAP sorguları, Active Directory'yi sorgulamak için son derece güçlü araçlardır. Çok çeşitli bilgileri toplamak, AD ortamının haritasını çıkarmak ve yanlış yapılandırmaları avlamak için güçlerinden yararlanabiliriz. LDAP sorguları, daha da ayrıntılı aramalar gerçekleştirmek için filtrelerle birleştirilebilir. Sonraki iki bölüm, bizi sonraki modüllerde çeşitli AD numaralandırma araçlarını tanıtmaya hazırlamak için hem AD hem de LDAP arama filtrelerini derinlemesine ele alacaktır.


### Active Directory Arama Filtreleri
Sonraki iki bölümde [ActiveDirectory PowerShell modülü cmdlet'leri](https://learn.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps) tarafından kullanılan Filter ve LDAPFilter parametreleri ele alınacaktır. PowerShell kullanarak Active Directory'yi sorgulamak için uygun filtre sözdiziminin nasıl oluşturulacağını bilmek önemlidir. Bu bilgi, PowerView gibi araçlarımızın kaputun altında nasıl çalıştığını ve Active Directory'yi numaralandırırken güçlerinden nasıl daha fazla yararlanabileceğimizi daha iyi anlamamızı sağlar. Ayrıca, bir değerlendirme sırasında araçlarınızdan herhangi biri olmadan kendinizi bir durumda bulursanız filtreleri nasıl formüle edeceğinizi anlamak da yararlıdır. Bu bilgilerle donanmış olarak, etkili bir şekilde “arazide yaşayabilecek” ve numaralandırma görevlerinizi gerçekleştirmek için yerleşik PowerShell cmdlet'lerini kullanabileceksiniz (bu modülde ele alacağımız araçların çoğunu kullanmaktan daha yavaş olsa da).


### PowerShell Filters
PowerShell'deki filtreler, piped çıktısını daha verimli bir şekilde işlemenize ve bir komuttan tam olarak ihtiyacınız olan bilgileri almanıza olanak tanır. Filtreler, büyük bir sonuçtaki belirli verileri daraltmak veya daha sonra başka bir komuta aktarılabilecek verileri almak için kullanılabilir.

Filter parametresi ile filtreleri kullanabiliriz. Temel bir örnek, yüklü yazılım için bir bilgisayarın sorgulanmasıdır:


### PowerShell - Yüklü Yazılımı Filtreleme

![Pasted image 20241204013749.png](/img/user/resimler/Pasted%20image%2020241204013749.png)
![Pasted image 20241204013757.png](/img/user/resimler/Pasted%20image%2020241204013757.png)

Yukarıdaki komut hatırı sayılır bir çıktı sağlayabilir. Tüm Microsoft yazılımlarını filtrelemek için Filter parametresini notlike operatörüyle birlikte kullanabiliriz (bu, yerel ayrıcalık yükseltme vektörleri için bir sistemi numaralandırırken yararlı olabilir).

### PowerShell - Microsoft Yazılımlarını Filtreleme

![Pasted image 20241204013834.png](/img/user/resimler/Pasted%20image%2020241204013834.png)


### Operators
Filtre işleci, arama sonuçlarını daraltmaya veya büyük miktarda komut çıktısını daha sindirilebilir bir şeye indirgemeye yardımcı olabilecek en az bir operatörü gerektirir. Özellikle geniş ortamları numaralandırırken ve komut çıktısında çok özel bilgiler ararken düzgün filtreleme yapmak önemlidir. Aşağıdaki operatörler Filter parametresiyle birlikte kullanılabilir:
![Pasted image 20241204014441.png](/img/user/resimler/Pasted%20image%2020241204014441.png)


### Filtre Örnekleri: AD Nesne Özellikleri

Filtre, çeşitli AD nesnesi özelliklerini karşılaştırmak, hariç tutmak, aramak vb. için operatörlerle birlikte kullanılabilir. Filtreler küme parantezleri, tek tırnak, parantez veya çift tırnak içine alınabilir. Örneğin, Sally Jones kullanıcısı hakkında bilgi bulmak için Get-ADUser kullanan aşağıdaki basit arama filtresi aşağıdaki gibi yazılabilir:


### PowerShell - Filter Examples

![Pasted image 20241204014546.png](/img/user/resimler/Pasted%20image%2020241204014546.png)
![Pasted image 20241204014551.png](/img/user/resimler/Pasted%20image%2020241204014551.png)

Yukarıda görüldüğü gibi, özellik değeri (burada, sally jones ) tek veya çift tırnak içine alınabilir. Yıldız işareti ( * ), sorgular gerçekleştirilirken [joker](https://ss64.com/ps/syntax-wildcards.html) karakter olarak kullanılabilir. Get-ADUser -filter {name -like “joe*”} komutu bir joker karakter kullanarak, adı joe (joe, joel, vb.) ile başlayan tüm domain kullanıcılarını döndürür. Filtreler kullanılırken, belirli karakterlerin öncelenmesi gerekir:

![Pasted image 20241204014729.png](/img/user/resimler/Pasted%20image%2020241204014729.png)

INLANEFREIGHT.LOCAL domain'ini listelemek için bu filtrelerden bazılarını deneyelim. İlginç host adları için tüm domain bilgisayarlarını arayabiliriz. SQL sunucuları, dahili değerlendirmelerde özellikle ilgi çekici bir hedeftir. Aşağıdaki komut Get-ADComputer kullanarak domain'deki tüm hostları arar ve SQL kelimesini içeren DNSHostName özelliğini filtreler.


### PowerShell - Filter For SQL
 ![Pasted image 20241204014934.png](/img/user/resimler/Pasted%20image%2020241204014934.png)
![Pasted image 20241204014938.png](/img/user/resimler/Pasted%20image%2020241204014938.png)

Ardından, yönetici gruplarını arayalım. Bunu adminCount özniteliğinde filtreleme yaparak yapabiliriz. Bu özniteliğin 1 olarak ayarlandığı gruplar [AdminSDHolder](https://docs.microsoft.com/en-us/windows-server/identity/ad-ds/plan/security-best-practices/appendix-c--protected-accounts-and-groups-in-active-directory) tarafından korunur ve korumalı gruplar olarak bilinir. AdminSDHolder, Domain Admins grubuna aittir. Active Directory'deki nesnelerin izinlerini değiştirme ayrıcalıklarına sahiptir. Yukarıda tartışıldığı gibi, filtrelenmiş komut çıktısını pipe edebilir ve yalnızca grup adlarını seçebiliriz.


### PowerShell - Yönetici Gruplarını Filtreleme

![Pasted image 20241204015655.png](/img/user/resimler/Pasted%20image%2020241204015655.png)

Filtreleri de birleştirebiliriz. DoesNotRequirePreAuth özniteliği ayarlanmış tüm yönetici kullanıcıları arayalım, yani ASREPRoasted olabilirler (bu saldırı daha sonraki modüllerde derinlemesine ele alınacaktır). Burada tüm domain kullanıcılarını seçiyoruz ve -eq operatörü ile iki koşul belirtiyoruz


### PowerShell - Filter Administrative Users
![Pasted image 20241204015803.png](/img/user/resimler/Pasted%20image%2020241204015803.png)
![Pasted image 20241204015812 1.png](/img/user/resimler/Pasted%20image%2020241204015812%201.png)

Son olarak, istediğimiz bilgileri bulmak için filtreleri birleştirmenin ve çıktıyı birden çok kez piping etmenin bir örneğini görelim. Aşağıdaki komut, “ servicePrincipalName ” öznitelik kümesine sahip tüm yönetici kullanıcıları bulmak için kullanılabilir, bu da muhtemelen bir Kerberoasting saldırısına maruz kalabilecekleri anlamına gelir. Bu örnek, adminCount özniteliği 1 olarak ayarlanmış hesapları bulmak için Filter parametresini uygular, bir Service Principal Name (SPN) ile tüm hesapları bulmak için bu çıktıyı pipe eder ve son olarak hesap adı, grup üyeliği ve SPN dahil olmak üzere hesaplarla ilgili birkaç özniteliği seçer.


### PowerShell - ServicePrincipalName ile Yönetici Kullanıcıları Bulma


![Pasted image 20241204015927.png](/img/user/resimler/Pasted%20image%2020241204015927.png)

Yukarıdaki komutların birçok kombinasyonunu kullanarak bir Active Directory ortamını numaralandırmak son derece uzun zaman alacaktır. Bu son örnek PowerView veya Rubeus gibi araçlarla hızlı ve kolay bir şekilde gerçekleştirilebilir. Bununla birlikte, PowerView gibi araçlardan elde edilen çıktılar bize kesin sonuçlar sağlamak için daha da filtrelenebileceğinden, AD'yi numaralandırırken filtreleri yetkin bir şekilde uygulamak önemlidir.


### LDAP Search Filters

### Basic LDAP Filter Syntax and Operators

Aynı cmdlet'lere sahip LDAPFilter parametresi, bilgi ararken LDAP arama filtrelerini kullanmamızı sağlar. Bu filtrelerin sözdizimi [RFC 4515 - Lightweight Directory Access Protocol (LDAP) içinde tanımlanmıştır: Arama Filtrelerinin String Gösterimi.](https://datatracker.ietf.org/doc/html/rfc4515)

LDAP filtrelerinin bir veya daha fazla ölçütü olmalıdır. Birden fazla kriter varsa, bunlar mantıksal AND veya OR operatörleri kullanılarak birleştirilebilir. Bu operatörler her zaman kriterlerin (operandların) önüne yerleştirilir ve bu da [Polish Notation](https://en.wikipedia.org/wiki/Polish_notation) olarak adlandırılır.

Filtre kuralları parantez içine alınır ve grup parantez içine alınarak ve aşağıdaki karşılaştırma operatörlerinden biri kullanılarak gruplandırılabilir:

![Pasted image 20241204021027.png](/img/user/resimler/Pasted%20image%2020241204021027.png)

Bazı örnek AND ve OR işlemleri aşağıdaki gibidir:

AND Operation:
* Bir kriter: (& (..C1..) (..C2..))
* İkiden fazla kriter: (& (..C1..) (..C2..) (..C3..))

OR Operation:
* Bir kriter: (| (..C1..) (..C2..))
* İkiden fazla kriter: (| (..C1..) (..C2..) (..C3..)) 

Ayrıca iç içe işlemlerimiz de olabilir, örneğin “ (|(& (..C1..) (..C2..))(& (..C3..) (..C4..)) “ ifadesi “ (C1 VE C2) VEYA (C3 VE C4) ” ifadesine dönüşür.


### Arama Kriterleri
Bir LDAP arama filtresi yazarken, söz konusu LDAP özniteliği için bir kural gereksinimi belirtmemiz gerekir (örneğin “ (displayName=william) ”). Arama kriterlerimizi belirtmek için aşağıdaki kurallar kullanılabilir:

![Pasted image 20241204021238.png](/img/user/resimler/Pasted%20image%2020241204021238.png)
![Pasted image 20241204021247.png](/img/user/resimler/Pasted%20image%2020241204021247.png)

Bu [bağlantı](https://docs.bmc.com/docs/fpsc121/ldap-attributes-and-associated-fields-495323340.html) User Attributes'in geniş bir listesini içerir ve aşağıda tüm Base Attributes'in bir listesi yer almaktadır.

Temel Niteliklerin tam listesi

![Pasted image 20241204021451.png](/img/user/resimler/Pasted%20image%2020241204021451.png)
.... 10 sayfa 



### Object Identifiers (OIDs)

Microsoft'un bu [Search Filter Syntax](https://docs.microsoft.com/en-us/windows/win32/adsi/search-filter-syntax) belgesinde listelendiği gibi LDAP filtreleriyle eşleşen kural [Object Identifiers](https://ldapwiki.com/wiki/Wiki.jsp?page=OID) (OID'ler) de kullanabiliriz:

![Pasted image 20241204022225.png](/img/user/resimler/Pasted%20image%2020241204022225.png)
https://ldapwiki.com/wiki/Wiki.jsp?page=1.2.840.113556.1.4.804
https://ldapwiki.com/wiki/Wiki.jsp?page=1.2.840.113556.1.4.803

1-Bir eşleşme yalnızca özniteliğin tüm bitleri değerle eşleşirse bulunur. Bu kural, bitsel AND operatörüne eşdeğerdir
2-Öznitelikteki herhangi bir bit değerle eşleşirse bir eşleşme bulunur. Bu kural, bitsel VEYA operatörüne eşdeğerdir.

![Pasted image 20241204022318.png](/img/user/resimler/Pasted%20image%2020241204022318.png)

Bu kural, DN'ye uygulanan filtrelerle sınırlıdır. Bu, bir eşleşme bulana kadar nesnelerdeki soy zincirini köke kadar yürüten özel bir “genişletilmiş” eşleşme işlecidir.

Yukarıdaki OID'leri bazı örneklerle açıklayabiliriz. Aşağıdaki LDAP sorgusunu ele alalım:
![Pasted image 20241204022350.png](/img/user/resimler/Pasted%20image%2020241204022350.png)

Bu sorgu, yönetimsel olarak devre dışı bırakılmış tüm kullanıcı hesaplarını veya [ACCOUNTDISABLE (2)](https://ldapwiki.com/wiki/Wiki.jsp?page=ACCOUNTDISABLE) döndürür. Bu sorguyu, hedef domainimize karşı “ Get-ADUser ” cmdlet'i ile bir LDAP arama filtresi olarak birleştirebiliriz. LDAP sorgusu aşağıdaki gibi kısaltılabilir:


### LDAP Sorgusu - Devre Dışı Kullanıcı Hesaplarını Filtreleme

![Pasted image 20241204022440.png](/img/user/resimler/Pasted%20image%2020241204022440.png)

Şimdi genişletilebilir eşleşme kuralı “ 1.2.840.113556.1.4.1941 ” için bir örneğe bakalım. Aşağıdaki sorguyu ele alalım:

![Pasted image 20241204023732.png](/img/user/resimler/Pasted%20image%2020241204023732.png)

Bu eşleştirme kuralı, Harry Jones (” CN=Harry Jones,OU=Network Ops,OU=IT,OU=Employees,DC=INLANEFREIGHT,DC=LOCAL ”) kullanıcısının üyesi olduğu tüm grupları bulacaktır. Bu filtreyi “ Get-ADGroup ” cmdlet'i ile kullanmak bize aşağıdaki çıktıyı verir:


### LDAP Query - Find All Groups
![Pasted image 20241204023758.png](/img/user/resimler/Pasted%20image%2020241204023758.png)


### Filter Types, Item Types & Escaped Characters
LDAP arama [filtreleri](https://ldapwiki.com/wiki/Wiki.jsp?page=LDAP%20SearchFilters) ile aşağıdaki dört filtre türüne sahibiz:
![Pasted image 20241204023837.png](/img/user/resimler/Pasted%20image%2020241204023837.png)

Ve dört item tipimiz var:

![Pasted image 20241204023857.png](/img/user/resimler/Pasted%20image%2020241204023857.png)

Son olarak, bir LDAP filtresinde kullanıldığında aşağıdaki karakterlerin öncelenmesi gerekir:

![Pasted image 20241204023910.png](/img/user/resimler/Pasted%20image%2020241204023910.png)


### Example LDAP Filters

Test domainimizde kullanmak üzere birkaç LDAP filtresi daha oluşturalım

“ (&(objectCategory=user)(description=*)) filtresini kullanabiliriz. “ tuşuna basarak boş bir açıklama alanı olmayan tüm kullanıcı hesaplarını bulabilirsiniz. Bu, her dahili ağ değerlendirmesinde gerçekleştirilmesi gereken yararlı bir aramadır, çünkü AD'deki kullanıcı açıklaması özniteliğinde depolanan (tüm AD kullanıcıları tarafından okunabilen) kullanıcıların parolalarını bulmak nadir değildir.

Bunu “ Get-ADUser ” cmdlet'i ile birleştirerek, boş bir açıklama alanına sahip olmayan tüm domain kullanıcılarını arayabilir ve bu durumda bir hizmet hesabı şifresi bulabiliriz!


### LDAP Sorgusu - Açıklama Alanı

![Pasted image 20241204024026.png](/img/user/resimler/Pasted%20image%2020241204024026.png)
![Pasted image 20241204024030.png](/img/user/resimler/Pasted%20image%2020241204024030.png)

Bu filtre “ (userAccountControl:1.2.840.113556.1.4.803:=524288) ”, Kerberos Saldırıları ile ilgili daha sonraki bir modülde ele alınacak olan yetkilendirme veya kısıtlanmamış yetkilendirme için güvenilir olarak işaretlenmiş tüm kullanıcıları veya bilgisayarları bulmak için kullanılabilir. Bu LDAP filtresi yardımıyla kullanıcıları numaralandırabiliriz:


### LDAP Sorgusu - Güvenilir Kullanıcıları Bulma
![Pasted image 20241204024107.png](/img/user/resimler/Pasted%20image%2020241204024107.png)

Bu ayar ile bilgisayarları da numaralandırabiliriz:

### LDAP Sorgusu - Güvenilir Bilgisayarları Bulma

![Pasted image 20241204024143.png](/img/user/resimler/Pasted%20image%2020241204024143.png)

Son olarak, “ adminCount ‘ özniteliği 1 olarak ayarlanmış ve ’ useraccountcontrol ‘ özniteliği ’ PASSWD_NOTREQD ” bayrağıyla ayarlanmış, yani hesabın boş bir parolaya sahip olabileceği tüm kullanıcıları arayalım. Bunu yapmak için, iki LDAP arama filtresini aşağıdaki gibi birleştirmeliyiz:

![Pasted image 20241204024203.png](/img/user/resimler/Pasted%20image%2020241204024203.png)

### LDAP Sorgusu - Boş Parolalı Kullanıcılar

![Pasted image 20241204024220.png](/img/user/resimler/Pasted%20image%2020241204024220.png)
Nadiren de olsa, zaman zaman parola belirlenmemiş hesaplar buluyoruz, bu nedenle PASSWD_NOTREQD bayrağı ayarlanmış hesapları listelemek ve gerçekten bir parola belirlenip belirlenmediğini kontrol etmek her zaman önemlidir. Bu durum kasıtlı olarak (belki de zaman kazanmak için) ya da bu bayrağa sahip bir kullanıcının komut satırı üzerinden parolasını değiştirmesi ve parolayı yazmadan önce yanlışlıkla enter tuşuna basması durumunda kazara meydana gelebilir. Tüm kuruluşlar periyodik hesap denetimleri gerçekleştirmeli ve bu bayrağın ayarlanması için geçerli bir iş nedeni olmayan tüm hesaplardan bu bayrağı kaldırmalıdır.

Kendi filtrelerinizi oluşturmayı deneyin. Bu kılavuz Active Directory: LDAP [Sözdizimi](https://social.technet.microsoft.com/wiki/contents/articles/5392.active-directory-ldap-syntax-filters.aspx) Filtreleri harika bir başlangıç noktasıdır.


### Recursive Match

“ RecursiveMatch “ parametresini, ‘ 1.2.840.113556.1.4.1941 ’ eşleştirme kuralı OID'sini kullandığımıza benzer şekilde kullanabiliriz. Bunun iyi bir örneği, bir AD kullanıcısının hem doğrudan hem de dolaylı olarak parçası olduğu tüm grupları bulmaktır. Bu aynı zamanda “iç içe grup üyeliği” olarak da bilinir. Örneğin, bob.smith kullanıcısı Domain Admins grubunun doğrudan bir üyesi olmayabilir ancak Security Operations grubu Domain Admins grubunun bir üyesi olduğu için türetilmiş Domain Admin haklarına sahiptir. Bunu grafiksel olarak Active Directory Computers and Users'a bakarak görebiliriz.

![Pasted image 20241204024444.png](/img/user/resimler/Pasted%20image%2020241204024444.png)

Bunu PowerShell ile birkaç şekilde numaralandırabiliriz, bir yol “ GetADGroupMember ” cmdlet'idir.

### PowerShell - Members Of Security Operations

![Pasted image 20241204024509.png](/img/user/resimler/Pasted%20image%2020241204024509.png)

Yukarıda gördüğümüz gibi, Security Operations grubu gerçekten de Domain Admins grubunun içinde “ iç içe geçmiştir”. Bu nedenle, üyelerinden herhangi biri Domain Admins'tir.

Get-ADUser kullanarak bir kullanıcının grup üyeliğini memberof özelliğine odaklanarak aramak bu bilgiyi doğrudan göstermeyecektir.


### PowerShell - User's Group Membership

![Pasted image 20241204024558.png](/img/user/resimler/Pasted%20image%2020241204024558.png)

Aşağıdaki örneklerde görüldüğü gibi, eşleşen kural OID'si ve RecursiveMatch parametresiyle iç içe grup üyeliğini bulabiliriz. İlk örnekte, harry.jones kullanıcısının üyesi olduğu tüm grupları özyinelemeli olarak sorgulamak için bir AD filtresi ve RecursiveMatch gösterilmektedir.


### PowerShell - Tüm Kullanıcı Grupları
![Pasted image 20241204024630.png](/img/user/resimler/Pasted%20image%2020241204024630.png)

Aynı bilgiyi döndürmenin başka bir yolu da bir LDAPFilter ve eşleşen kural OID'si kullanmaktır


### LDAP Sorgusu - Tüm Kullanıcı Grupları
![Pasted image 20241204024655.png](/img/user/resimler/Pasted%20image%2020241204024655.png)

Yukarıdaki örneklerde gösterildiği gibi, AD'de özyinelemeli olarak arama yapmak, standart arama sorgularının göstermediği bilgileri sıralamamıza yardımcı olabilir. İç içe grup üyeliğinin numaralandırılması çok önemli. Hedef AD ortamında, özellikle AD'de binlerce nesne bulunan büyük kuruluşlarda, aksi takdirde fark edilmeyecek ciddi yanlış yapılandırmaları ortaya çıkarabiliriz. Bu bilgileri numaralandırmanın başka yollarını ve hatta grafik biçiminde sunmanın yollarını göreceğiz, ancak RecursiveMatch göz ardı edilmemesi gereken güçlü bir arama parametresidir.


### SearchBase ve SearchScope Parametreleri
Küçük Active Directory ortamları bile binlerce olmasa da yüzlerce nesne içerebilir. Active Directory, kullanıcılar, gruplar, bilgisayarlar, OU'lar vb. eklendikçe ve ACL'ler kuruldukça çok hızlı bir şekilde büyüyebilir ve bu da giderek daha karmaşık bir ilişkiler ağı oluşturur. Kendimizi 10-20 yıllık, 10 binlerce nesneye sahip geniş bir ortamda da bulabiliriz. Bu ortamları numaralandırmak hantal bir görev haline gelebilir, bu nedenle aramalarımızı hassaslaştırmamız gerekir.

“ SearchBase “ parametresini kullanarak aramalarımızı kapsamlandırarak numaralandırma komutlarımızın ve komut dosyalarımızın performansını artırabilir ve döndürülen nesnelerin hacmini azaltabiliriz. Bu parametre, altında arama yapılacak bir Active Directory yolunu belirtir ve belirli bir OU'daki bir kullanıcı hesabını aramaya başlamamızı sağlar. “ SearchBase “ parametresi ‘OU=Employees,DC=INLANEFREIGHT,DC=LOCAL’ gibi bir OU'nun ayırt edici adını (DN) kabul eder.

” SearchScope “ OU hiyerarşisinin ne kadar derininde arama yapmak istediğimizi tanımlamamızı sağlar. Bu parametrenin üç seviyesi vardır:

![Pasted image 20241204024933.png](/img/user/resimler/Pasted%20image%2020241204024933.png)

AD'yi “ SearchScope ‘ kullanarak sorgularken adı veya sayıyı belirtebiliriz (örneğin, SearchScope Onelevel ’ SearchScope 1 ” ile aynı şekilde yorumlanır).

![Pasted image 20241204024959.png](/img/user/resimler/Pasted%20image%2020241204024959.png)

Yukarıdaki örnekte, SearchBase OU=Employees,DC=INLANEFREIGHT,DC=LOCAL olarak ayarlandığında, Base olarak ayarlanan bir SearchScope OU nesnesinin (Employees) kendisini sorgulamaya çalışacaktır. OneLevel olarak ayarlanan bir SearchScope yalnızca Employees OU içinde arama yapar. Son olarak, SubTree olarak ayarlanmış bir SearchScope, Employees OU'sunu ve altındaki Muhasebe, Yükleniciler vb. gibi tüm OU'ları sorgulayacaktır. Bu OU'ların altındaki OU'lar (alt kapsayıcılar)


### SearchBase ve Arama Kapsamı Parametreleri Örnekleri
Base, OneLevel ve Subtree arasındaki farkı göstermek için bazı örneklere bakalım. Bu örnekler için Employees OU'ya odaklanacağız. Aşağıdaki Active Directory Kullanıcıları ve Bilgisayarları ekran görüntüsünde Employees Base'dir ve Get-ADUser ile belirtilmesi hiçbir şey döndürmeyecektir. OneLevel sadece Amelia Matthews kullanıcısını döndürür ve SubTree, Employees konteyneri altındaki tüm alt konteynerlerdeki tüm kullanıcıları döndürür.
![Pasted image 20241204025101.png](/img/user/resimler/Pasted%20image%2020241204025101.png)

Bu sonuçları PowerShell kullanarak doğrulayabiliriz. Referans amacıyla, 970 kullanıcı gösteren Employees OU altındaki tüm AD kullanıcılarının bir sayısını alalım.

### PowerShell - Tüm AD Kullanıcılarının Sayısı

![Pasted image 20241204025126.png](/img/user/resimler/Pasted%20image%2020241204025126.png)
Beklendiği gibi, SearchScope'un Base olarak belirtilmesi hiçbir şey döndürmeyecektir.


### PowerShell - SearchScope Base
![Pasted image 20241204025147.png](/img/user/resimler/Pasted%20image%2020241204025147.png)

Ancak, “ Get-ADObject ‘ ile ’ Base ” belirtirsek, bize yalnızca nesne (Employees OU) döndürülür.


### PowerShell - SearchScope Temel OU Nesnesi

![Pasted image 20241204025234.png](/img/user/resimler/Pasted%20image%2020241204025234.png)
![Pasted image 20241204025239.png](/img/user/resimler/Pasted%20image%2020241204025239.png)

SearchScope olarak OneLevel belirtirsek, yukarıdaki resimde beklendiği gibi bize bir kullanıcı döndürülür


### PowerShell - Searchscope OneLevel

![Pasted image 20241204025305.png](/img/user/resimler/Pasted%20image%2020241204025305.png)

Yukarıda belirtildiği gibi, SearchScope değerleri birbirinin yerine kullanılabilir, bu nedenle SearchScope değeri olarak 1 belirtildiğinde aynı sonuç döndürülür.


### PowerShell - Searchscope 1

![Pasted image 20241204025327.png](/img/user/resimler/Pasted%20image%2020241204025327.png)

Son olarak, SearchBase olarak Subtree'yi belirtirsek, yukarıda belirlediğimiz kullanıcı sayısıyla eşleşen tüm alt konteynerlerdeki tüm nesneleri alırız.


### PowerShell - Searchscope Subtree

![Pasted image 20241204025350.png](/img/user/resimler/Pasted%20image%2020241204025350.png)

Bu bölüm ve PowerShell Filtreleri bölümü, “arazide yaşayarak” numaralandırmamızı geliştirmek için yerleşik AD cmdlet'leriyle birlikte arama filtrelerini kullanabileceğimiz birçok yolu ele aldı. Daha sonraki bölümlerde, numaralandırmayı çok daha hızlı ve kolay hale getiren ve daha da güçlü olmak için filtrelerle birleştirilebilen araçları ele alacağız. Yerleşik araçlar, özel komut dosyaları veya üçüncü taraf araçlar kullanıyor olmamızdan bağımsız olarak, ne yaptıklarını anlamak ve hedefimize ulaşmamıza yardımcı olmak için numaralandırmamızın çıktısını anlayabilmek ve kullanabilmek önemlidir.


### Enumerating Active Directory with Built-in Tools
Doğru numaralandırma, tüm sızma testleri ve kırmızı ekip değerlendirmeleri için kilit öneme sahiptir. AD'yi, özellikle de çok sayıda host, kullanıcı ve hizmet içeren büyük kurumsal ortamları numaralandırmak oldukça göz korkutucu bir görev olabilir ve çok fazla miktarda veri sağlayabilir. AD'yi numaralandırmak için sistem yöneticileri ve pentesterler tarafından çeşitli yerleşik Windows araçları kullanılabilir. Aynı numaralandırma tekniklerine dayalı olarak açık kaynak araçları da oluşturulmuştur. Bu araçların birçoğu (SharpView, BloodHound ve PingCastle gibi) numaralandırma sürecini hızlandırmak ve verileri tüketilebilir ve eyleme geçirilebilir bir formatta doğru bir şekilde sunmak için kullanılabilir. Bir değerlendirmede arazide yaşamanız gerekiyorsa veya belirli araçlar için tespitler mevcutsa, birden fazla araç ve “ ayrıntılı suç ” bilgisi önemlidir


### User-Account-Control (UAC) Attributes

User-Account-Control Öznitelikleri domain hesaplarının davranışını kontrol eder. Bu değerler Windows User Account Control teknolojisi ile karıştırılmamalıdır. Bu UAC özniteliklerinin çoğu güvenlikle ilgilidir:
![Pasted image 20241204025553.png](/img/user/resimler/Pasted%20image%2020241204025553.png)


### PowerShell - Built-in AD Cmdlets

![Pasted image 20241204025610.png](/img/user/resimler/Pasted%20image%2020241204025610.png)
![Pasted image 20241204025614.png](/img/user/resimler/Pasted%20image%2020241204025614.png)

Yine de useraccountcontrol değerlerini yorumlamak için karşılık gelen bayraklara dönüştürmemiz gerekir. Bu, bu [script](https://academy.hackthebox.com/storage/resources/Convert-UserAccountControlValues.zip) ile yapılabilir. Örnek olarak useraccountcontrol değeri 4260384 olan Jenna Smith kullanıcısını ele alalım.


### PowerShell - UAC Values
![Pasted image 20241204025704.png](/img/user/resimler/Pasted%20image%2020241204025704.png)

Bu değerleri listelemek için PowerView'i de kullanabiliriz (sonraki modüllerde ayrıntılı olarak ele alınacaktır). Bazı kullanıcıların varsayılan 512 veya Normal_Account değeriyle eşleştiğini, diğerlerinin ise dönüştürülmesi gerektiğini görebiliriz. jenna.smith için değer, dönüştürme scriptimizin sağladığı değerle eşleşiyor.

PowerView, hedef host üzerindeki c:\tools dizininde bulunabilir. Aracı yüklemek için bir PowerShell konsolu açın, araçlar dizinine gidin ve Import-Module .\PowerView.ps1 komutunu kullanarak PowerView'i içe aktarın.


### PowerView - Domain Accounts
![Pasted image 20241204025751.png](/img/user/resimler/Pasted%20image%2020241204025751.png)
![Pasted image 20241204025801.png](/img/user/resimler/Pasted%20image%2020241204025801.png)


### Enumeration Using Built-In Tools
PowerShell AD Modülü, Sysinternals Suite ve AD DS Araçları gibi sistem yöneticilerinin kendilerinin de kullanabileceği araçların beyaz listeye alınması ve özellikle daha olgun ortamlarda radarın altında kalması muhtemeldir. AD numaralandırması için aşağıdakiler de dahil olmak üzere çeşitli yerleşik araçlardan yararlanılabilir:

DS Tools tüm modern Windows işletim sistemlerinde varsayılan olarak mevcuttur ancak numaralandırma faaliyetlerini gerçekleştirmek için domain bağlantısı gerekir.


### DS Tools
![Pasted image 20241204025852.png](/img/user/resimler/Pasted%20image%2020241204025852.png)
![Pasted image 20241204025859.png](/img/user/resimler/Pasted%20image%2020241204025859.png)

PowerShell Active Directory modülü, Active Directory'yi yönetmek için kullanılan bir grup cmdlet'tir. AD PowerShell modülünün yüklenmesi yönetici erişimi gerektirir.


### AD PowerShell Module
![Pasted image 20241204025930.png](/img/user/resimler/Pasted%20image%2020241204025930.png)

Windows Yönetim Araçları (WMI) Active Directory'deki nesnelere erişmek ve bunları sorgulamak için de kullanılabilir. Birçok komut dosyası dili WMI AD sağlayıcısı ile etkileşime girebilir, ancak PowerShell bunu çok kolaylaştırır.


### Windows Management Instrumentation (WMI)
![Pasted image 20241204025958.png](/img/user/resimler/Pasted%20image%2020241204025958.png)
...


Active Directory Service Interfaces (ADSI), Active Directory'yi sorgulayabilen bir dizi COM arayüzüdür. PowerShell yine onunla etkileşim kurmak için kolay bir yol sağlar.


### AD Service Interfaces (ADSI)

![Pasted image 20241204030043.png](/img/user/resimler/Pasted%20image%2020241204030043.png)



### LDAP Anonymous Bind
LDAP (Lightweight Directory Access Protocol), dizin hizmetlerine erişmek için kullanılan bir protokoldür.


### LDAP Anonim Bağlamadan Yararlanma
LDAP anonim bağlamaları, kimliği doğrulanmamış saldırganların domain'den kullanıcıların, grupların, bilgisayarların, kullanıcı hesabı özniteliklerinin ve domain parola ilkesinin tam listesi gibi bilgileri almasına olanak tanır. LDAP'nin açık kaynak sürümlerini çalıştıran Linux host'ları ve Linux vCenter gereçleri genellikle anonim bağlamalara izin verecek şekilde yapılandırılır.

Bir LDAP sunucusu anonim taban bağlamalarına izin verdiğinde, bir saldırganın domain'den önemli miktarda bilgi sorgulamak için bir base nesnesini bilmesi gerekmez. Bu aynı zamanda bir parola püskürtme saldırısı düzenlemek veya hesap açıklama alanlarında saklanan parolalar gibi bilgileri okumak için de kullanılabilir. [Windapsearch](https://github.com/ropnop/windapsearch) ve [ldapsearch](https://linux.die.net/man/1/ldapsearch) gibi araçlar, anonim bir LDAP bind aracılığıyla domain bilgilerini numaralandırmak için kullanılabilir. Anonim bir LDAP bağlantısından elde ettiğimiz bilgiler, bir parola püskürtme veya AS-REPRoasting saldırısı düzenlemek, hesap açıklama alanlarında depolanan parolalar gibi bilgileri okumak için kullanılabilir.

Kimlik bilgileri olmadan LDAP ile etkileşime girip giremeyeceğimizi hızlıca kontrol etmek için Python'u kullanabiliriz.

![Pasted image 20241204030349.png](/img/user/resimler/Pasted%20image%2020241204030349.png)
![Pasted image 20241204030355.png](/img/user/resimler/Pasted%20image%2020241204030355.png)


### Using Ldapsearch
Anonim LDAP bağlantısını ldapsearch ile onaylayabilir ve tüm AD nesnelerini LDAP'tan alabiliriz.
![Pasted image 20241204030418.png](/img/user/resimler/Pasted%20image%2020241204030418.png)


### Using Windapsearch
Windapsearch, LDAP sorgularını kullanarak AD kullanıcılarının, gruplarının ve bilgisayarlarının anonim ve kimliği doğrulanmış LDAP numaralandırmasını gerçekleştirmek için kullanılan bir Python betiğidir. Özel LDAP sorguları oluşturmanızı gerektiren ldapsearch gibi araçlara bir alternatiftir. LDAP NULL oturum kimlik doğrulamasını onaylamak için kullanabiliriz, ancak -u “” ile boş bir kullanıcı adı sağlayabilir ve domain fonksiyonel seviyesini onaylamak için --functionality ekleyebiliriz.
![Pasted image 20241204030457.png](/img/user/resimler/Pasted%20image%2020241204030457.png)

Parola püskürtme saldırısında kullanmak üzere tüm domain kullanıcılarının bir listesini çıkarabiliriz.

![Pasted image 20241204030514.png](/img/user/resimler/Pasted%20image%2020241204030514.png)

Tüm domain bilgisayarları hakkında bilgi edinebiliriz.

![Pasted image 20241204030527.png](/img/user/resimler/Pasted%20image%2020241204030527.png)
![Pasted image 20241204030534.png](/img/user/resimler/Pasted%20image%2020241204030534.png)

Bu proses, grup bilgilerini ve kısıtlanmamış kullanıcılar ve bilgisayarlar, GPO bilgileri, kullanıcı ve bilgisayar öznitelikleri gibi daha ayrıntılı bilgileri çekmek için tekrarlanabilir

### Other Tools
LDAP'tan bilgi almak için başka birçok araç ve yardımcı betik vardır. Bu ldapsearch-ad.py betiği windapsearch'e benzer.

![Pasted image 20241204030610.png](/img/user/resimler/Pasted%20image%2020241204030610.png)
![Pasted image 20241204030616.png](/img/user/resimler/Pasted%20image%2020241204030616.png)

Domain bilgilerini çekmek ve bir NULL bind'ı onaylamak için kullanabiliriz. Bu özel araç, ek numaralandırma gerçekleştirmek için geçerli domain kullanıcı kimlik bilgileri gerektirir.

![Pasted image 20241204030637.png](/img/user/resimler/Pasted%20image%2020241204030637.png)



### Kimlik Bilgisine Sahip LDAP Numaralandırma
SMB'de olduğu gibi, domain kimlik bilgilerine sahip olduğumuzda, LDAP'tan kullanıcı, grup, bilgisayar, güven, GPO bilgisi, domain şifre politikası vb. dahil olmak üzere çok çeşitli bilgileri çıkarabiliriz. ldapsearch-ad.py ve windapsearch bu numaralandırmayı gerçekleştirmek için kullanışlıdır.

### Windapsearch
![Pasted image 20241204030726.png](/img/user/resimler/Pasted%20image%2020241204030726.png)
![Pasted image 20241204030733.png](/img/user/resimler/Pasted%20image%2020241204030733.png)
![Pasted image 20241204030741.png](/img/user/resimler/Pasted%20image%2020241204030741.png)

Kısıtlanmamış yetkilendirme ile kullanıcıları ve bilgisayarları çekmek de dahil olmak üzere bazı ek kullanışlı seçenekler.

![Pasted image 20241204030803.png](/img/user/resimler/Pasted%20image%2020241204030803.png)


### Ldapsearch-ad
Bu araç, tüm standart numaralandırma işlemlerini ve işleri basitleştirmek için birkaç yerleşik aramayı gerçekleştirebilir. Parola politikasını hızlı bir şekilde elde edebiliriz

![Pasted image 20241204030829.png](/img/user/resimler/Pasted%20image%2020241204030829.png)

Kerberoasting saldırısına maruz kalabilecek kullanıcıları arayabiliriz.

![Pasted image 20241204030841.png](/img/user/resimler/Pasted%20image%2020241204030841.png)

Ayrıca, ASREPRoasted olabilecek kullanıcıları hızlı bir şekilde alır

![Pasted image 20241204030852.png](/img/user/resimler/Pasted%20image%2020241204030852.png)


### LDAP Özeti
LDAP kullanarak önemli miktarda AD numaralandırması yapmak için bu bölümde gösterilen iki araç gibi araçları kullanabiliriz. Araçlar, aramayı basitleştirmek ve bize en yararlı ve eyleme geçirilebilir verileri sağlamak için birçok yerleşik sorguya sahiptir. Bu araçları modülün başlarında öğrendiğimiz özel LDAP arama filtreleriyle de birleştirebiliriz. Bunlar cephaneliğimizde bulundurmamız gereken harika araçlardır, özellikle de AD değerlendirmesinin çoğunun bir Linux saldırı kutusundan yapılması gereken bir konumda olduğumuzda.


### Active Directory LDAP - Skills Assessment
INLANEFREIGHT kuruluşu tarafından, standart bir Domain User hesabıyla dahili ağ erişimi elde eden bir saldırgan tarafından istismar edilebilecek kusurların neler olduğunu değerlendirmek üzere bir Active Directory güvenlik değerlendirmesi yapmak üzere görevlendirildiniz. Hedef host'a bağlanın ve bu modülü tamamlamak için aşağıda listelenen numaralandırma görevlerini gerçekleştirin. 