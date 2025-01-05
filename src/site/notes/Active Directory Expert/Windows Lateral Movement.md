---
{"dg-publish":true,"permalink":"/active-directory-expert/windows-lateral-movement/"}
---

Lateral movement, ilk erişimi elde ettikten sonra bir ağda ilerlemek için kullandığımız teknikleri ifade eder. Lateral hareketi anlayarak, saldırganlar ve savunucular ağlarda daha iyi gezinebilir ve güvenli hale gelebilir. Bu bilgi, savunucuların daha etkili güvenlik önlemleri uygulamasına olanak tanır ve saldırganların ağ savunmalarındaki zayıflıkları belirleyip bunlardan yararlanmasına yardımcı olarak sonuçta daha sağlam ve dirençli bir güvenlik duruşu sağlar.

### Lateral Movement Tanımı
Lateral hareket, genellikle ayrıcalıkları yükseltmek veya hassas verilere erişmek amacıyla bir ağ içinde bir sistemden diğerine geçmeyi içerir. Lateral hareket tekniklerini kullanarak, kimlik bilgileri, hassas veriler ve diğer yüksek değerli varlıkları aramak için bir ağın derinliklerine inebiliriz.

Lateral bir hareket gerçekleştirmek için parolalar, hash'ler, biletler, SSH anahtarları ve session cookie'ler de dahil olmak üzere her türlü kimlik bilgisine ihtiyacımız vardır. Ağdaki remote bir bilgisayara bağlanmak için bunlardan yararlanabiliriz. Etkili bir lateral hareket, ağ mimarilerini derinlemesine anlamayı ve uzak sistemlerde kod çalıştırmak için kullanabileceğimiz servisleri ve protokolleri belirleme becerisini gerektirir


### MITRE ATT&CK Framework
MITRE ATT&CK framework lateral hareketi bir ağ üzerindeki uzak sistemlere girmek ve kontrol etmek için kullanılan teknikler olarak tanımlar. Bu genellikle ağı keşfetmeyi, birden fazla sistem ve hesap arasında geçiş yapmayı ve uzaktan erişim araçlarını ya da local araçlarla meşru kimlik bilgilerini kullanmayı içerir

MITRE ATT&CK, lateral hareket için aşağıdakiler de dahil olmak üzere çeşitli teknikler listeler:

![Pasted image 20241205215403.png](/img/user/resimler/Pasted%20image%2020241205215403.png)
![Pasted image 20241205215413.png](/img/user/resimler/Pasted%20image%2020241205215413.png)
![Pasted image 20241205215427.png](/img/user/resimler/Pasted%20image%2020241205215427.png)

Bu teknikler, bir ağ içindeki uzak sistemlerde gezinmek ve bunları kontrol etmek için kullanabileceğimiz çeşitli yöntemleri göstermektedir.


### Networks & Systems
Ağların ve sistemlerin nasıl çalıştığını anlamak, yanal hareket gerçekleştirmek için çok önemlidir. İlk adımımız, hedef alabileceğimiz ağ cihazlarını belirlemek veya haritalamaktır; bunu port taraması, ping taraması veya Active Directory bilgilerini kullanarak yapabiliriz.

Ağı anladıktan sonra, ağ segmentasyonu veya güvenlik duvarı kısıtlamaları nedeniyle bazı sistemlere erişilemeyebileceğinin farkında olmamız gerekir. Bu gibi durumlarda, bu hizmetlere erişmek için kutunun dışında düşünmemiz gerekir. Bu senaryoları doğrudan lateral hareket ve dolaylı lateral hareket olarak ikiye ayıralım


### Direct Lateral Movement

Direkt lateral hareket, komutları doğrudan hedef makinede yürütebildiğimiz ve hedef makineyi bize geri bağlanmaya zorlayabildiğimiz yerdir. Örneğin, SRV01'i ele geçirirsek ve SRV02'ye yanal olarak geçmemiz gerekirse, SRV02'de komutları yürütmek ve SRV02'de bir oturum veya shell elde etmek için SRV01'den PSExec'i kullanabiliriz.

![Pasted image 20241205215604.png](/img/user/resimler/Pasted%20image%2020241205215604.png)


### Indirect Lateral Movement

İndirekt lateral hareket, başka bir sistemden talimat aldığında hedef makinede komutların çalıştırılmasını içerir. Örneğin, bir ağ güvenlik duvarı kısıtlaması nedeniyle SRV01'den SRV02'ye doğrudan ulaşamadığımızı, ancak SRV02'nin Windows Update Server'a (WSUS) bağlanabildiğini varsayalım. Bu durumda, WSUS sunucusunu ele geçirir ve istediğimiz komutu çalıştıran sahte bir Windows Güncellemesi oluşturursak, SRV02 güncellemeyi aldığında, kötü amaçlı güncellememizi çalıştıracak ve SRV02 üzerinde bir shell elde etmemize izin verecektir.

Aşağıdaki bölümde, her iki senaryoyu da test etmek için bazı örnekler oluşturacağız.


### Command Execution
Gördüğümüz gibi, lateral hareket ile çalışırken komut yürütme çok önemlidir. Komutları yürütme yeteneği, remote hizmetlere erişim sağlamamıza yardımcı olabilir. Bu modül boyunca, çeşitli güvenlik mekanizmaları kullanan ağlarla uğraşırken yardımcı olacak komutları veya payloads'ları çalıştırmak için farklı yöntemler kullanacağız.

Çoklu ağ segmentleri: Farklı departmanları veya güvenlik bölgelerini temsil eder.
Key altyapı bileşenleri: Domain controller, güncelleme sunucuları ve yönetim sunucuları.

Tartışılan teknikler ve savunmalar hakkındaki anlayışımızı pekiştirerek lateral hareket fırsatlarını belirleme ve kullanma pratiği yapacağız.


### Network Segmentation

Ağ segmentasyonunu anlamak, saldırganlar olarak yanal hareketi etkili bir şekilde gerçekleştirmek için çok önemlidir. Ağ segmentasyonu, bir saldırının yayılmasını sınırlamak için bir ağı daha küçük, izole edilmiş segmentlere bölmeyi içerir. Uygun ağ segmentasyonu şunları yapabilir:

İhlalleri kontrol altına alın: Hareketimizi kısıtlayın ve saldırı yüzeyini azaltın.

İzlemeyi geliştirin: Ağ trafiğinin daha odaklı ve etkili bir şekilde izlenmesine izin verin.

Erişim kontrolünü geliştirin: Farklı segmentler arasında sıkı erişim politikaları uygulayın.

![Pasted image 20241205215948.png](/img/user/resimler/Pasted%20image%2020241205215948.png)

Yukarıdaki resimde, ağ topolojisinin üst düzey bir genel görünümünü görebiliriz. Üç ağ segmenti vardır ve hangi ağın diğerine ulaşabileceğini belirleyen cihaz Switch Layer 3'tür. Diğer ağlarda bu cihaz bir yönlendirici, bir Linux sunucusu veya bir güvenlik duvarı olabilir. Bu cihazların segmentler arasındaki iletişimi nasıl kontrol ettiğini anlamak, lateral hareketi planlamak için çok önemlidir.

Test yoluyla, hangi iletişime izin verildiğini belirleyebiliriz, ancak bu durumda, etkileşime varsayılan bir ihlal senaryosundan başlayacağız.

Her bölümde tüm sunucular mevcut olmayacaktır; bazen farklı bir sunucudan başlayacağız. Bu değişkenlik, ağ segmentasyonunu ve bunun yanal hareket etme kabiliyetimiz üzerindeki etkisini anlamanın önemini vurgulamaktadır.

Bu modülün sonunda, Windows yanal hareketi konusunda sağlam bir temele sahip olacağız ve bize bu gelişmiş saldırıları gerçekleştirme ve bunlara karşı savunma bilgisi sağlayacağız.


### Introduction to Remote Services
Remote services, sistemlere uzaktan erişim ve yönetim sağlayan, işbirliğini kolaylaştıran ve verimliliği artıran, işletmeler ve BT departmanları için temel araçlardır. Bu hizmetler, yöneticilerin fiziksel erişime ihtiyaç duymadan sistemleri yönetmesine ve sorun gidermesine olanak tanır; bu da özellikle dağıtık ve büyük ölçekli ortamlarda değerlidir.


### Remote Services'in Amacı
Remote servisler, aşağıdakiler de dahil olmak üzere çeşitli avantajlar sağlar:
* Remote management : BT personelinin sistemleri her yerden yapılandırmasına, güncellemesine ve sorun gidermesine olanak tanıma
* Kaynak paylaşımı : Kullanıcıların ağ üzerinden paylaşılan dosyalara, yazıcılara ve uygulamalara erişmesini sağlamak.
* İşbirliği : Uzak ekipler arasında iletişim ve işbirliğini kolaylaştırma.
* Verimlilik : Fiziksel varlık ihtiyacının azaltılması, zaman ve seyahat maliyetlerinden tasarruf.

Saldırganlar olarak, bir ağ içinde yanal olarak hareket etmek, ayrıcalıkları yükseltmek ve kalıcılığı sürdürmek için bu hizmetleri kötüye kullanabiliriz.


### Types of Remote Services

[T1021 - MITRE ATT&CK framework](https://attack.mitre.org/techniques/T1021/) tekniğine dayanarak, yanal hareket için kullanabileceğimiz çeşitli remote service türleri vardır. Bu bölümde, aşağıdakilere odaklanacağız:

* Remote Desktop Protocol (RDP) :
* SMB / Windows Paylaşımları 
* Windows Management Instrumentation (WMI)
* Windows Remote Management (WinRM) :
* Windows Uzaktan Yönetimi (WinRM) : WinRM, web hizmetlerini kullanarak local ve remote bilgisayarlarla iletişim kurmak için güvenli bir yol sağlayan WS-Management protokolünün Microsoft uygulamasıdır. Genellikle uzaktan yönetim görevleri için kullanılır.
* Distributed Component Object Model (DCOM) : DCOM, yazılım bileşenlerinin doğrudan bir ağ üzerinden iletişim kurmasını sağlayan bir Microsoft teknolojisidir. Farklı bilgisayarlardaki nesneler arasındaki iletişimi desteklemek için Component Object Model'i (COM) genişletir.
* Secure Shell (SSH) 



### Enumeration Methods
Bu remote servicelerden faydalanmak için öncelikle onları ağ içinde tanımlamamız gerekir. Numaralandırma, mevcut hizmetlerin keşfedilmesini ve bunlar hakkında bilgi toplanmasını içerir. Yaygın numaralandırma yöntemleri şunları içerir:

* Port tarama : Açık portları ve üzerlerinde çalışan hizmetleri tanımlama
* Servis banner'ları : Sürüm ve yapılandırma bilgilerini toplamak için servis banner'larını yakalama ve analiz etme.
* Active Directory : Sistemler ve hizmetleri hakkında bilgi almak için Active Directory'yi sorgulama.


Elde ettiğimiz kimlik bilgilerini kullanarak bu servislere kimlik doğrulaması yapabilir ve lateral hareket girişiminde bulunabiliriz. Kullanabileceğimiz farklı kimlik bilgileri türleri şunlardır:

- **Şifreler (Passwords):** Kullanıcının kimliğini doğrulamak için gizli bir kelime veya ifade sağladığı geleneksel bir kimlik doğrulama yöntemi.
- **NTLM Hash'leri:** Windows ortamlarında kimlik doğrulama için kullanılan şifrelerin kriptografik temsilleri. Bu hash'ler, "Pass the Hash" adlı bir teknikle gerçek şifreye ihtiyaç duymadan kimlik doğrulama yapmak için kullanılabilir.
- **NTLMv2 Hash'leri:** Daha iyi güvenlik sağlayan NTLM hash'lerinin geliştirilmiş bir versiyonu. Bu hash'ler de Windows ortamlarında kimlik doğrulama için kullanılır. Ağdaki bu hash'leri kötüye kullanarak "NTLM Relay" saldırılarıyla lateral hareket gerçekleştirilebilir.
- **AES256 Anahtarları (AES256 Keys):** 256 bit anahtarlı Gelişmiş Şifreleme Standardı (AES), verileri şifrelemek için kullanılır. Bazı durumlarda, bu anahtarlar Windows sistemlerinde **Rubeus** veya **Mimikatz** kullanılarak kimlik doğrulama için kullanılabilir.
- **Biletler (Tickets - Kerberos):** Kerberos, düğümlerin kimliklerini güvenli bir şekilde kanıtlamalarına olanak tanıyan bir kimlik doğrulama protokolüdür. Ağ içinde lateral hareket yapmak ve kimlik doğrulama sağlamak için biletler oluşturulabilir veya ele geçirilebilir ve ardından kullanılabili


İlerleyen bölümlerde, bu hizmetleri nasıl kötüye kullanacağımızı ve uzak hizmetlere erişim sağlamak ve ağ içinde yanal olarak hareket etmek için farklı kimlik bilgileriyle nasıl kimlik doğrulaması yapacağımızı inceleyeceğiz.


### Remote Desktop Service (RDP)

Remote Desktop Protocol (RDP), Microsoft tarafından geliştirilen ve bir kullanıcıya ağ bağlantısı üzerinden başka bir bilgisayara bağlanmak için grafik bir arayüz sağlayan tescilli bir protokoldür. RDP remote yönetim, teknik destek ve workstation ve sunuculara uzaktan erişim için yaygın olarak kullanılır. RDP, bant genişliğine bağlı olarak küçültülebilen yüksek çözünürlüklü grafiklerle uzaktan ses, pano, yazıcılar ve dosya aktarımları dahil olmak üzere eksiksiz bir masaüstü deneyimini destekler. RDP varsayılan olarak iletişim için TCP port 3389'u kullanır


### RDP Rights
RDP'ye bağlanmak için gerekli haklar yapılandırmaya bağlıdır; varsayılan olarak, yalnızca Administrators veya Remote Desktop Users gruplarının üyeleri RDP aracılığıyla bağlanabilir. Ayrıca, bir yönetici belirli kullanıcılara veya gruplara RDP'ye bağlanma hakkı verebilir. Bu haklar local olarak ayarlandığından, bunları listelemenin tek yolu hedef bilgisayarda Administrative haklarına sahip olmamızdır.


### RDP Enumeration
Lateral hareket için RDP'yi kullanmak için, test ettiğimiz ortamda RDP'nin mevcut olup olmadığının farkında olmamız gerekir, 3389 numaralı portu aramak için NMAP veya başka bir ağ numaralandırma aracı kullanabiliriz ve hedeflerin bir listesini aldıktan sonra, birden fazla kimlik bilgisini test etmek için NetExec gibi araçlarla bu listeyi kullanabiliriz.

Not: RDP varsayılan olarak TCP portu 3389'u kullanır, ancak yöneticiler bunu başka herhangi bir portta yapılandırabilir.

RDP'ye karşı kimlik bilgilerini test etmek için netexec kullanacağız. Protokolü rdp, hesabı Helen ve şifreyi RedRiot88 seçelim:

![Pasted image 20241205224031.png](/img/user/resimler/Pasted%20image%2020241205224031.png)

Helen'in SRV01 üzerinde RDP haklarına sahip olduğunu doğruluyoruz. Unutmayın ki (Pwn3d!) hedef makinede yönetici haklarına sahip olduğumuz anlamına gelmez, ancak RDP'ye bağlanma haklarına sahip olduğumuz anlamına gelir.


### Windows'tan Lateral Movement 
Windows'tan RDP'ye bağlanmak için Run, Cmd veya PowerShell üzerinde mstsc çalıştırılarak erişilebilen varsayılan Windows Remote Desktop Connection istemcisini kullanabiliriz:

![Pasted image 20241205224241.png](/img/user/resimler/Pasted%20image%2020241205224241.png)

Bu, hedef IP adresini veya domain adını belirleyebileceğimiz bir client açacak ve Connect'e tıkladığımızda bizden kimlik bilgilerini isteyecektir:

![Pasted image 20241205224303.png](/img/user/resimler/Pasted%20image%2020241205224303.png)

RDP kullanılarak verimli bir şekilde yürütülebilecek bazı eylemler şunlardır

- **Dosya Transferi (File Transfer):** Dosyaları yerel ve uzak bilgisayarlar arasında sürükle-bırak yöntemiyle veya kopyala-yapıştır kullanarak aktarın.
- **Uygulama Çalıştırma (Running Applications):** Uzak bilgisayarda uygulamalar çalıştırın. Bu, yalnızca uzak makinede yüklü olan yazılımlara erişmek için faydalıdır.
- **Yazdırma (Printing):** Uzak bilgisayardaki belgeleri yerel bilgisayara bağlı bir yazıcıya yazdırın.
- **Ses ve Video Akışı (Audio and Video Streaming):** Uzak bilgisayardan yerel makineye ses ve video akışı gerçekleştirin. Bu, multimedya uygulamaları için faydalıdır.
- **Pano Paylaşımı (Clipboard Sharing):** Yerel ve uzak bilgisayarlar arasında panoyu paylaşarak metin ve görüntüleri makineler arasında kopyalayıp yapıştırmanıza olanak tanır.


### Linux'tan Lateral Movement 

Linux'tan RDP'ye bağlanmak için xfreerdp komut satırı aracını kullanabiliriz. İşte nasıl kullanılacağına dair bir örnek:

![Pasted image 20241205224617.png](/img/user/resimler/Pasted%20image%2020241205224617.png)

In this command:

/u:Helen kullanıcı adını belirtir.
/p:'RedRiot88' parolayı belirtir.
/d:inlanefreight.local domain alanını belirtir.
/v:10.129.229.244 hedef Windows makinesinin IP adresini belirtir.
/dynamic-resolution pencereleri dinamik olarak yeniden boyutlandırmamızı sağlayan dinamik çözünürlük ayarını etkinleştirir.
/drive:.,linux lokal dosya sistemini uzak oturuma yönlendirerek uzak Windows makinesinden erişilebilir hale getirir.

Bu komutu terminalde çalıştırarak, belirtilen Windows makinesine bir RDP bağlantısı kurabilir ve Windows Remote Desktop Connection istemcisini kullanarak yaptığımız benzer işlemleri gerçekleştirebiliriz.


### Düşük Gecikmeli Ağlar veya Proxy Bağlantıları için xfreerdp'yi Optimize Etme

Bir proxy üzerinden veya yavaş ağ bağlantısı ile xfreerdp kullanıyorsanız, aşağıdaki ek seçenekleri kullanarak oturum hızını artırabiliriz:

![Pasted image 20241205224800.png](/img/user/resimler/Pasted%20image%2020241205224800.png)

In this command:

- **/bpp:8:** Renk derinliğini piksel başına 8 bite düşürür, bu da iletilen veri miktarını azaltır.
- **/compression:** Ağ üzerinden gönderilen veri miktarını azaltmak için sıkıştırmayı etkinleştirir.
- **-themes:** Masaüstü temalarını devre dışı bırakarak grafiksel veri miktarını azaltır.
- **-wallpaper:** Masaüstü duvar kağıdını devre dışı bırakarak grafiksel veri miktarını daha da azaltır.
- **/clipboard:** Yerel ve uzak makineler arasında pano paylaşımını etkinleştirir.
- **/audio-mode:0:** Bant genişliğinden tasarruf etmek için ses yönlendirmesini devre dışı bırakır.
- **/auto-reconnect:** Bağlantı kesildiğinde otomatik olarak yeniden bağlanır, bu da oturumun kararlılığını artırır.
- **-glyph-cache:** Yazı karakterlerini (metin karakterlerini) önbelleğe almayı etkinleştirir, bu da metin işleme için gönderilen veri miktarını azaltır.

Bu seçeneklerin kullanılması, RDP oturumunun performansını optimize etmeye yardımcı olarak ideal olmayan ağ koşullarında bile daha sorunsuz bir deneyim sağlar.

### Kısıtlı Admin Modu
Restricted Admin Mode, RDP bağlantıları üzerinden kimlik bilgilerinin çalınması riskini azaltmak için Microsoft tarafından sunulan bir güvenlik özelliğidir. Etkinleştirildiğinde, etkileşimli oturum açma yerine bir ağ oturumu açar ve uzak sistemde kimlik bilgilerinin önbelleğe alınmasını önler. Bu mod yalnızca yöneticiler için geçerlidir, bu nedenle yönetici olmayan bir hesapla remote bir bilgisayarda oturum açtığınızda kullanılamaz.

Bu mod kimlik bilgilerinin önbelleğe alınmasını engellese de, etkinleştirilirse, yanal hareket için Pass the Hash veya Pass the Ticket'ın yürütülmesine izin verir.

Restricted Admin Mode'un etkin olup olmadığını doğrulamak için aşağıdaki kayıt defteri anahtarını sorgulayabiliriz:

![Pasted image 20241205225426.png](/img/user/resimler/Pasted%20image%2020241205225426.png)

DisableRestrictedAdmin değeri, Restricted Admin Mode (Kısıtlı Yönetici Modu) durumunu gösterir:
* Değer 0 ise, Restricted Admin Mode (Kısıtlı Yönetici Modu) etkinleştirilir.
* Değer 1 ise, Restricted Admin Mode (Kısıtlı Yönetici Modu) devre dışıdır.

Key mevcut değilse, bu devre dışı olduğu anlamına gelir ve aşağıdaki hata mesajını görürüz:

![Pasted image 20241205225529.png](/img/user/resimler/Pasted%20image%2020241205225529.png)

Ayrıca, Restricted Admin Mode'u etkinleştirmek için DisableRestrictedAdmin değerini 0 olarak ayarlarız. İşte bunu etkinleştirmek için komut:

![Pasted image 20241205225604.png](/img/user/resimler/Pasted%20image%2020241205225604.png)

Ve Restricted Admin Mode'u devre dışı bırakmak için DisableRestrictedAdmin değerini 1 olarak ayarlayın:

![Pasted image 20241205225631.png](/img/user/resimler/Pasted%20image%2020241205225631.png)

Not: Yalnızca Administrators grubunun üyeleri Restricted Admin Mode'u kötüye kullanabilir


### Pivoting

Lateral hareket gerçekleştirmek için pivotlama kullanmamız gerekebilir, Pivotlama, Tünelleme ve Port Yönlendirme modülünde pivotlama hakkında bilmemiz gereken her şeyi açıklıyoruz

Bu laboratuvarda tek bir host'a erişimimiz var. Linux saldırı hostumuzdan diğer makinelere bağlanmak için bir pivot yöntemi kurmamız gerekecek; bu durumda chisel kullanacağız.

etc/proxychains.conf dosyasında 1080 portunda bir socks5 SOCKS proxy yapılandırmamız gerekecek:


![Pasted image 20241205225934.png](/img/user/resimler/Pasted%20image%2020241205225934.png)

Daha sonra, Linux makinemizde reverse port forwarding sunucusunu başlatacağız:

![Pasted image 20241205225947.png](/img/user/resimler/Pasted%20image%2020241205225947.png)

Daha sonra SRV01'de aşağıdaki komutla sunucuya bağlanacağız chisel.exe client VPN IP R:socks :

![Pasted image 20241205230255.png](/img/user/resimler/Pasted%20image%2020241205230255.png)

Not: Modül sırasında proxychains kullanımını gördüğümüzde bu adımlar her zaman gereklidir. Alternatif olarak, PwnBox kullanıyorsanız tavsiye edilen [Ligolo-ng](https://github.com/nicocha30/ligolo-ng) gibi araçları da kullanabiliriz.


### Pass the Hash and Pass the Ticket for RDP

Restricted Admin Mode'un etkin olduğunu onayladıktan sonra veya etkinleştirebilirsek, RDP ile Pass the Hash veya Pass the Ticket saldırılarını gerçekleştirmeye devam edebiliriz.

Bir Linux makineden Pass the Hash işlemini gerçekleştirmek için, bir hash kullanmak ve RDP'ye bağlanmak için /pth seçeneği ile xfreerdp'yi kullanabiliriz. İşte örnek bir komut:

![Pasted image 20241205230817.png](/img/user/resimler/Pasted%20image%2020241205230817.png)

Pass the Ticket için Rubeus'u kullanabiliriz. Helen'in hash'ini kullanarak bir bilet oluşturacağız. Öncelikle createnetonly seçeneği ile bir kurban prosesi başlatmamız gerekiyor:

![Pasted image 20241205231004.png](/img/user/resimler/Pasted%20image%2020241205231004.png)
* ***.\Rubeus.exe**: Rubeus adlı aracı çalıştırır. Kerberos tabanlı güvenlik testleri için kullanılır.

* ***createnetonly**: "Net-Only" modunda bir oturum oluşturur. Yerel kimlik doğrulaması olmadan oturum açar.

* ***/program:powershell.exe**: "Net-Only" modunda çalıştırılacak programı belirler. Burada PowerShell başlatılır.

* ***/show**: Oturum bilgilerini (örneğin, ID) ekranda gösterir.

* ***Amaç**: Oturuma manuel olarak Kerberos biletleri veya kimlik bilgileri eklemek ve bu oturumu testlerde kullanmak.

Yeni PowerShell penceresinde Helen'in hash'ini kullanarak bir Ticket-Granting ticket (TGT) oluşturacağız:

![Pasted image 20241205231317.png](/img/user/resimler/Pasted%20image%2020241205231317.png)
![Pasted image 20241205231323.png](/img/user/resimler/Pasted%20image%2020241205231323.png)
![Pasted image 20241205231332.png](/img/user/resimler/Pasted%20image%2020241205231332.png)

[[Bağlantılar/Chatgpt\|Chatgpt]] 

Bileti içe aktardığımız pencereden mstsc /restrictedAdmin komutunu kullanabiliriz:

![Pasted image 20241205231637.png](/img/user/resimler/Pasted%20image%2020241205231637.png)

O anda oturum açmış kullanıcı olarak bir pencere açacaktır. İsmin taklit etmeye çalıştığımız hesapla aynı olmaması önemli değildir.

![Pasted image 20241205231657.png](/img/user/resimler/Pasted%20image%2020241205231657.png)


Girişe tıkladığımızda, hash kullanarak RDP'ye bağlanmamıza izin verecektir:

![Pasted image 20241205231712.png](/img/user/resimler/Pasted%20image%2020241205231712.png)

### Basit Özet:

1. **İlk Komut:** Bir "Net-Only" PowerShell oturumu başlatır.
2. **İkinci Komut:** Helen’in hash’ini kullanarak bir Kerberos bileti oluşturur ve bu bileti oturuma yükler.
3. **Üçüncü Komut:** Helen'in yetkileriyle bir uzak masaüstü bağlantısı başlatır.

### SharpRDP

SharpRDP, RDP client'ları tarafından kullanılan mstscax.dll kütüphanesinden yararlanarak RDP üzerinden grafiksel olmayan, kimliği doğrulanmış uzaktan komut yürütmeye izin veren bir .NET aracıdır. Bu araç, GUI client veya SOCKS proxy'ye ihtiyaç duymadan bağlanma, kimlik doğrulama, komutları yürütme ve bağlantıyı kesme gibi eylemleri gerçekleştirebilir

SharpRDP terminal servisleri kütüphanesine ( mstscax.dll ) dayanır ve gerekli DLL'leri ( MSTSCLib.DLL ve AxMSTSCLib.DLL ) mstscax.dll'den üretir. Terminal hizmetleri bağlantı nesnesi örneklemesini işlemek ve yanal hareket için gereken eylemleri gerçekleştirmek için görünmez bir Windows formu kullanır.

Hedef makinede komutları çalıştırmak için Metasploit ve PowerShell kullanacağız. Linux makinemizde 8888 numaralı portu dinlemek için Metasploit'i çalıştıracağız:

![Pasted image 20241206072400.png](/img/user/resimler/Pasted%20image%2020241206072400.png)

Daha sonra PowerShell Reflection kullanarak msfvenom ile bir payload oluşturacağız:

![Pasted image 20241206072453.png](/img/user/resimler/Pasted%20image%2020241206072453.png)

Daha sonra payload'umuzu barındırmak için python http sunucusunu kullanırız:

Şimdi payload'umuzu çalıştırmak ve bir oturum sağlamak üzere bir powershell komutu çalıştırmak için SharpRDP'yi kullanabiliriz:

![Pasted image 20241206072605.png](/img/user/resimler/Pasted%20image%2020241206072605.png)
![Pasted image 20241206072613.png](/img/user/resimler/Pasted%20image%2020241206072613.png)

Not: SharpRDP komutlarının yürütülmesi 259 karakterle sınırlıdır

SharpRDP komutları yürütmek için Microsoft run kullanır, tüm komut kayıtlarını temizlemek için [CleanRunMRU'yu](https://github.com/0xthirteen/CleanRunMRU) kullanabiliriz. Aracı derlemek için yerleşik Microsoft [csc](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/compiler-options/) derleyici aracını kullanabiliriz. İlk olarak CleanRunMRU'nun Program.cs dosyasını saldırı hostumuzdan hedef bilgisayara aktaralım:

![Pasted image 20241206073000.png](/img/user/resimler/Pasted%20image%2020241206073000.png)

Şimdi derlemek için csc.exe dosyasını kullanabiliriz:

![Pasted image 20241206073033.png](/img/user/resimler/Pasted%20image%2020241206073033.png)

Şimdi tüm komutları temizlemek için CleanRunMRU.exe'yi kullanabiliriz:

![Pasted image 20241206073148.png](/img/user/resimler/Pasted%20image%2020241206073148.png)


### Lateral Hareket için RDP'nin avantajları

RDP, lateral hareket için çeşitli avantajlar sağlar ve bu da onu belirli senaryolarda saldırganlar için tercih edilen bir yöntem haline getirir. Temel avantajlardan bazıları şunlardır:

* Algılamadan Kaçınma : RDP trafiği iş ortamlarında yaygındır, bu da şüphe uyandırma olasılığını azaltır.

* Yönetici Olmayan Erişim : RDP erişimi mutlaka yönetici hakları gerektirmez; yönetici olmayan bir kullanıcı da RDP erişimine sahip olabilir

* Kalıcı Erişim: Bir dayanak noktası oluşturulduktan sonra, RDP ağa kalıcı erişim sağlayabilir.


RDP, lateral hareket için güçlü bir araçtır ve kullanım kolaylığı ve tespitten kaçma gibi çeşitli avantajlar sunar. RDP'nin nasıl etkili bir şekilde kullanılacağını ve kötüye kullanılacağını anlamak, yanal hareket yeteneklerimizi önemli ölçüde artırabilir. Bir sonraki bölümde, yanal hareket için en yaygın ve yaygın olarak kullanılan yöntemlerden biri olan SMB'yi inceleyeceğiz.



### Server Message Block (SMB)

Server Message Block (SMB), bir ağ içindeki bilgisayarlar arasında dosya, yazıcı ve diğer kaynakların paylaşımını kolaylaştıran bir ağ iletişim protokolüdür. Kullanıcıların ve uygulamaların dosyaları okuyup yazmasına, dizinleri yönetmesine ve remote sunucularda local sunuculardaymış gibi farklı işlevler gerçekleştirmesine olanak tanır. Ayrıca işlemler arası iletişim için işlem protokollerini de destekler. SMB öncelikle Windows sistemlerinde çalışır, ancak diğer işletim sistemleriyle uyumludur, bu da onu ağa bağlı ortamlar için önemli bir protokol haline getirir.


### SMB Rights

Başarılı bir SMB lateral hareketi için, hedef bilgisayarda Administrators grubunun üyesi olan bir hesaba ihtiyacımız var. TCP 445 ve TCP 139 portlarının açık olması da çok önemlidir. İsteğe bağlı olarak, TCP 135 portunun da açık olması gerekebilir çünkü bazı araçlar iletişim için bunu kullanır.


### UAC remote kısıtlamaları
UAC, uzaktan kod yürütme elde etmemizi engelleyebilir, ancak bu kısıtlamaları anlamak, Windows'un farklı sürümlerinde UAC sınırlamalarında gezinirken bu araçlardan etkili bir şekilde yararlanmak için çok önemlidir, bu kısıtlamalar birkaç önemli noktaya işaret eder:

* Local admin ayrıcalıkları gereklidir.
* RID 500 olmayan local admin hesapları Windows Vista ve sonraki sürümlerde PsExec gibi araçları çalıştıramaz.
* Bir makinede admin haklarına sahip domain kullanıcıları PsExec gibi araçları çalıştırabilir.
* RID 500 local admin hesapları makinelerde PsExec gibi araçları kullanabilir


### SMB Named Pipes
TCP port 445 üzerinden IPC$ paylaşımı aracılığıyla erişilen SMB'deki named pipe'lar, bir ağ içinde lateral hareket için hayati önem taşır. NULL oturum context'lerinden local yönetici ayrıcalıkları gerektirenlere kadar bir dizi işlemi mümkün kılarlar. Örneğin, svcctl, Impacket'in psexec.py ve smbexec.py gibi araçlarında görüldüğü gibi, komutları yürütmek için servisleri remote olarak oluşturmayı, başlatmayı ve durdurmayı kolaylaştırır. atsvc, Impacket'in atexec.py'si tarafından kullanılan komut yürütme için zamanlanmış görevlerin remote olarak oluşturulmasını destekler. Bu named pipe'lar yanal hareket operasyonlarının etkin bir şekilde yürütülmesi ve yönetilmesi için çok önemlidir. winreg Windows kayıt defterine uzaktan erişim sağlayarak registry anahtarlarının ve değerlerinin sorgulanmasına ve değiştirilmesine olanak tanıyarak kötü amaçlı payload'ların kalıcılığına ve yapılandırılmasına yardımcı olur.



### SMB Enumeration
Lateral hareket işlemine başlamadan önce, SMB'nin hedef hostta çalıştığından emin olmamız gerekir. Bunu başarmak için NMAP kullanacağız.

SMB'nin hedefte çalışıp çalışmadığını doğrulamak için hedef hostta bir port taraması yapmalıyız. SMB varsayılan olarak TCP 139 ve TCP 445 portlarını kullanır.

![Pasted image 20241206173307.png](/img/user/resimler/Pasted%20image%2020241206173307.png)



### Lateral Movement From Windows

Windows'tan lateral hareket gerçekleştirmek için çeşitli araçlar ve teknikler kullanılabilir. Bu bölümde, PSExec , SharpNoPSExec , NimExec ve Reg.exe'yi göstereceğiz. Helen'in kimlik bilgilerini kullanarak RDP aracılığıyla SRV01'e bağlanalım:

![Pasted image 20241206173512.png](/img/user/resimler/Pasted%20image%2020241206173512.png)

### PSExec

[PsExec](https://learn.microsoft.com/en-us/sysinternals/downloads/psexec), yöneticilere sistem yönetimi görevlerinde yardımcı olmak için tasarlanmış bir araç koleksiyonu olan Microsoft'un [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/sysinternals-suite) paketine dahildir. Bu araç, uzaktan komut yürütmeyi kolaylaştırır ve TCP port 445 ve TCP port 139 üzerinde çalışan SMB protokolünü kullanarak bir named pipe üzerinden çıktı alır.

Varsayılan olarak, PSExec aşağıdaki eylemi gerçekleştirir:

* SMB aracılığıyla remote sistemdeki C:\Windows dizinine karşılık gelen hidden ADMIN$ paylaşımına bir bağlantı kurar
* PsExecsvc servisini başlatmak ve remote sistemde named pipe kurmak için Service Control Manager'ı (SCM) kullanır
* Interactive komut yürütme için konsolun girdi ve çıktısını oluşturulan named pipe üzerinden yönlendirir.

Not: PsExec [[Bağlantılar/double-hop\|double-hop]] sorununu ortadan kaldırır çünkü kimlik bilgileri komutla birlikte iletilir ve etkileşimli bir oturum açma oturumu oluşturur (Tip 2).

Remote bir host'a bağlanmak ve komutları interaktif olarak çalıştırmak için PsExec'i kullanabiliriz. Bağlandığımız bilgisayarı veya hedefi \\SRV02 , interactive shell için -i seçeneğini, -u <kullanıcı> seçeneği ile administrator oturum açma kimlik bilgilerini ve -p parola parolasını ve çalıştırılacak uygulamayı belirtmek için cmd'yi belirtmeliyiz:

![Pasted image 20241206174137.png](/img/user/resimler/Pasted%20image%2020241206174137.png)

Not: cmd veya powershell gibi uygulamaları çalıştırabiliriz ancak çalıştırmak için bir komut da belirtebiliriz.

Payload'umuzu NT AUTHORITY\SYSTEM olarak çalıştırmak istiyorsak -s seçeneğini belirtmemiz gerekir, bu da SYSTEM ayrıcalıklarıyla çalışacağı anlamına gelir:

![Pasted image 20241206174228.png](/img/user/resimler/Pasted%20image%2020241206174228.png)


### SharpNoPSExec

SharpNoPSExec, hedef sistemde mevcut servisleri kullanarak disk yazımı veya yeni servis oluşturma gibi işlemler yapmadan lateral movement (yatay hareket) gerçekleştirmek için tasarlanmış bir araçtır. Bu sayede tespit edilme riski minimuma indirilir. Araç, hedef makinedeki tüm servisleri sorgular ve aşağıdaki özelliklere sahip olanları belirler:

- **Başlatma Türü:** Disabled (devre dışı) veya manual (elle).
- **Mevcut Durum:** Stopped (durdurulmuş).
- **Çalışma Yetkisi:** LocalSystem ayrıcalıklarıyla çalışan servisler.

Araç, bu servislerden birini rastgele seçer ve geçici olarak servis binary yolunu (binary path) saldırganın seçtiği bir payload’a işaret edecek şekilde değiştirir. Çalıştırma işleminden sonra SharpNoPSExec yaklaşık 5 saniye bekler ve ardından servisin orijinal yapılandırmasını geri yükleyerek önceki durumuna döndürür.

Bu yaklaşım, bir shell sağlamanın yanı sıra, yeni bir servis oluşturulmadığı için güvenlik izleme sistemleri tarafından tespit edilme olasılığını da azaltır.

Aracı parametreler olmadan çalıştırdığımızda bazı yardım ve kullanım bilgileri göreceğiz

![Pasted image 20241206174624.png](/img/user/resimler/Pasted%20image%2020241206174624.png)
![Pasted image 20241206174637.png](/img/user/resimler/Pasted%20image%2020241206174637.png)

SharpNoPSExec ile lateral hareket gerçekleştirmek için bir listener'a ihtiyacımız olacak, çünkü bu araç sadece makinede kod çalıştırmamıza izin verecek, ancak PsExec'in yaptığı gibi bize etkileşimli bir shell vermeyecektir. Netcat ile dinlemeye başlayabiliriz:

![Pasted image 20241206174733.png](/img/user/resimler/Pasted%20image%2020241206174733.png)

SharpNoPSExec, komutu çalıştırdığımız konsolun kimlik bilgilerini kullanır, bu nedenle doğru kimlik bilgilerine sahip bir konsoldan başlattığımızdan emin olmamız gerekir. Alternatif olarak, --username , --password ve --domain argümanlarını kullanabiliriz. Ek olarak, hedef IP adresini veya domain adını --target= <IP/Domain> , ve çalıştırmak istediğimiz komutu sağlamamız gerekir. Komut için, reverse shell'imizi ayarlamak üzere yardım menüsünde gösterilen payload'u kullanabiliriz -- payload="c:\windows\system32\cmd.exe /c "reverseShell" . Reverse Shell payload'unu https://www.revshells.com veya favori C2'mizi kullanarak oluşturabiliriz:

![Pasted image 20241206174840.png](/img/user/resimler/Pasted%20image%2020241206174840.png)
![Pasted image 20241206174850.png](/img/user/resimler/Pasted%20image%2020241206174850.png)

Saldırı kutusuna baktığımızda, reverse shell bağlantısının başarıyla kurulduğunu görebiliriz:

![Pasted image 20241206174906.png](/img/user/resimler/Pasted%20image%2020241206174906.png)



### NimExec

sayfa22