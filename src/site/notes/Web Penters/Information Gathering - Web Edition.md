---
{"dg-publish":true,"permalink":"/web-penters/information-gathering-web-edition/"}
---

### Active Reconnaissance
- Port Scanning
- Vulnerability Scanning
- Network Mapping
- Banner Grabbing
- OS Fingerprinting
- Service Enumeration
- Web Spidering


### Passive Reconnaissance
* Search Engine Queries
* WHOIS Lookups
* DNS
* Web Archive Analysis
* Social Media Analysis
* Code Repositories

# WHOIS

```shell-session
M1R4CKCK@htb[/htb]$ whois inlanefreight.com

[...]
Domain Name: inlanefreight.com
Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
Registrar WHOIS Server: whois.registrar.amazon
Registrar URL: https://registrar.amazon.com
Updated Date: 2023-07-03T01:11:15Z
Creation Date: 2019-08-05T22:43:09Z
[...]
```


Her WHOIS kaydı tipik olarak aşağıdaki bilgileri içerir:

==Domain Adı==: Domain adının kendisi (örneğin, example.com)
==Registrar==: Domainin kaydedildiği şirket (örn. GoDaddy, Namecheap)
==Registrant Contact==: Domain'i kaydettiren kişi veya kuruluş.
==Administrative Contact==: Domainin yönetiminden sorumlu kişi.
==Technical Contact (Teknik İlgili Kişi)==: Domain ile ilgili teknik sorunları ele alan kişi.
==Creation and Expiration Dates==: Domainin ne zaman kaydedildiği ve ne zaman sona ereceği.
==Name Servers== : Domain adını bir IP adresine çeviren sunucular.

[WhoisFreaks](https://whoisfreaks.com/), alan adı bilgileri, WHOIS sorguları ve IP adresleriyle ilgili kapsamlı veriler sağlayan bir platformdur.

# DNS

DNS, domain adlarını (örneğin [www.example.com](http://www.example.com)) IP adreslerine çevirerek internet üzerindeki doğru hedefe yönlendirir. Bu, kullanıcıların hatırlanması kolay domainlerle gezinmesini sağlar.

## How DNS Works

Check Cache -> IP Found ? --> (No) --> Sends DNS Query to Resolver --> Checks Cache --> (No) --> Recursive Lookup --> Root Name Server --> TLD Name Server --> Authoritative Name Server --> Returns Ip to Computer --> Connect Web site 

* Bilgisayarınız Yönlendirme İstediğinde (DNS Sorgusu): Domain yazdığınızda, bilgisayarınız önce geçmişteki ziyaretlerden IP adresini hatırlayıp hatırlamadığını görmek için belleğini (==cache==) kontrol eder. Eğer hatırlamıyorsa, genellikle internet servis sağlayıcınız (ISP) tarafından sağlanan bir DNS resolver'a başvurur.

* DNS Resolver Haritasını Kontrol Eder (Recursive Search): Resolver'ın de bir cache'i vardır ve IP adresini burada bulamazsa, DNS hiyerarşisi üzerinden bir yolculuğa çıkar. İlk olarak ==root name server=='a sorar, bu sunucu internetin kütüphanecisi gibidir.

* Root Name Sunucusu Yolu Gösterir: Root sunucusu tam adresi bilmez, ancak kimin bildiğini bilir – domain'in sonuyla (örneğin, .com, .org) sorumlu olan top level domain (TLD) sunucusu. Resolver'a doğru yönü gösterir.

* TLD Name Sunucusu Adresi Daraltır: TLD adı sunucusu, bir zone harita gibidir. İncelediğiniz belirli domain için sorumlu olan authorized name sunucusunun kim olduğunu bilir (örneğin, example.com) ve resolver'a oraya gönderir.

* Authorized name Sunucusu Adresi Sağlar: Authorized name sunucusu son duraktır. İncelemek istediğiniz web sitesinin sokak adresi gibidir. Doğru IP adresini tutar ve resolver'a geri gönderir.

* DNS Resolver Bilgiyi Geri Döner: Resolver IP adresini alır ve bilgisayarınıza iletir. Ayrıca, bir süreliğine (cache'de alarak) saklar, böylece yakında tekrar siteyi ziyaret etmek isterseniz.

* Bilgisayarınız Bağlantıyı Kurar: Artık bilgisayarınız IP adresini bildiğine göre, doğrudan web sunucusuna bağlanabilir ve web sitesini ziyaret etmeye başlayabilirsiniz.


### Hosts Dosyası
hosts dosyası, host adlarını IP adresleriyle eşlemek için kullanılan basit bir metin dosyasıdır ve DNS sürecini atlayan manuel bir domain adı resolver yöntemi sağlar. DNS domain adlarının IP adreslerine çevrilmesini otomatikleştirirken, hosts dosyası doğrudan, local geçersiz kılmalara izin verir. Bu özellikle geliştirme, sorun giderme veya web sitelerini engelleme için yararlı olabilir.

Hosts dosyası Windows'ta C:\Windows\System32\drivers\etc\hosts ve Linux ve MacOS'ta /etc/hosts içinde bulunur. Dosyadaki her satır şu formatı izler:

```txt
<IP Address>    <Hostname> [<Alias> ...]
```

```txt
127.0.0.1       localhost
192.168.1.10    devserver.local
```


Hosts dosyasını düzenlemek için, administrative/root ayrıcalıklarını kullanarak bir metin düzenleyicisi ile açın. Gerektiği gibi yeni girişler ekleyin ve ardından dosyayı kaydedin. Değişiklikler, sistemin yeniden başlatılmasını gerektirmeden hemen etkili olur.

Hosts dosyasını düzenlemek için, administrative/root ayrıcalıklarını kullanarak bir metin düzenleyicisi ile açın. Gerektiği gibi yeni girişler ekleyin ve ardından dosyayı kaydedin. Değişiklikler, sistemin yeniden başlatılmasını gerektirmeden hemen etkili olur.

Yaygın kullanımlar arasında bir domain'in geliştirme için lokal bir sunucuya yönlendirilmesi yer alır:

```txt
127.0.0.1       myapp.local
```

Bir IP adresi belirleyerek bağlantıyı test etme:

```txt
192.168.1.20    testserver.local
```

veya istenmeyen web sitelerini, domainlerini var olmayan bir IP adresine yönlendirerek engellemek:

```txt
0.0.0.0       unwanted-site.com
```


DNS sürecini bir bayrak yarışı olarak düşünün. Bilgisayarınız domain adı ile başlar ve bunu resolver'a iletir. Resolver daha sonra isteği root sunucusuna, TLD sunucusuna ve son olarak authoritative sunucusuna iletir ve her biri hedefe daha da yaklaşır. IP adresi bulunduğunda, zincirleme olarak bilgisayarınıza iletilir ve web sitesine erişmenizi sağlar.


### Temel DNS Kavramları

Domain Name System'de (DNS) zone, belirli bir kuruluşun veya yöneticinin yönettiği domain namespace'in farklı bir parçasıdır. Bunu bir dizi domain adı için sanal bir konteyner olarak düşünün. Örneğin, example.com ve tüm subdomainleri (mail.example.com veya blog.example.com gibi) genellikle aynı DNS zone'una ait olur.

DNS sunucusunda bulunan bir metin dosyası olan zone dosyası, bu zone içindeki resource kayıtlarını (aşağıda açıklanmıştır) tanımlar ve domain adlarının IP adreslerine çevrilmesi için önemli bilgiler sağlar.

Örnek olarak, example.com için bir zone dosyasının nasıl görünebileceğine dair basitleştirilmiş bir örnek aşağıda verilmiştir:

```dns-zone
$TTL 3600 ; Default Time-To-Live (1 hour)
@       IN SOA   ns1.example.com. admin.example.com. (
                2024060401 ; Serial number (YYYYMMDDNN)
                3600       ; Refresh interval
                900        ; Retry interval
                604800     ; Expire time
                86400 )    ; Minimum TTL

@       IN NS    ns1.example.com.
@       IN NS    ns2.example.com.
@       IN MX 10 mail.example.com.
www     IN A     192.0.2.1
mail    IN A     198.51.100.1
ftp     IN CNAME www.example.com.
```

Bu dosya, example.com domain alanındaki çeşitli hostlar için authoritative name servers (NS kayıtları), mail server (MX kaydı) ve IP adreslerini (A kayıtları) tanımlar.

DNS sunucuları, her biri domain adı çözümleme sürecinde belirli bir amaca hizmet eden çeşitli kaynak kayıtlarını saklar. Şimdi en yaygın DNS kavramlarından bazılarını inceleyelim:

| DNS Kavramı               | Açıklama                                                                            | Örnek                                                                                                                                 |
| ------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| Domain Name               | İnternetteki bir site ya da kaynağın insanlar tarafından okunabilir etiketidir.     | [www.example.com](http://www.example.com)                                                                                             |
| IP Address                | İnternete bağlı her cihaz için atanmış benzersiz sayısal tanımlayıcıdır.            | 192.0.2.1                                                                                                                             |
| DNS Resolver              | Domain adlarını IP adreslerine çeviren sunucudur.                                   | İnternet servis sağlayıcınızın DNS sunucusu ya da Google DNS gibi genel çözümleyiciler (8.8.8.8)                                      |
| Root Name Server          | DNS hiyerarşisindeki en üst seviyedeki sunuculardır.                                | Dünyada 13 kök sunucu vardır, A-M olarak adlandırılır: a.root-servers.net                                                             |
| TLD Name Server           | Belirli üst düzey alan adları (örneğin, .com, .org) için sorumlu olan sunuculardır. | [Verisign](https://en.wikipedia.org/wiki/Verisign) .com için, [PIR](https://en.wikipedia.org/wiki/Public_Interest_Registry) .org için |
| Authoritative Name Server | Domain için gerçek IP adresini tutan sunucudur.                                     | Genellikle barındırma sağlayıcıları veya domain kayıt şirketleri tarafından yönetilir.                                                |
| DNS Record Types          | DNS'te saklanan farklı türdeki bilgilerdir.                                         | A, AAAA, CNAME, MX, NS, TXT vb.                                                                                                       |

DNS bilgilerinin yapı taşları olan çeşitli record türlerini daha derinlemesine inceleyelim. Bu record'lar, her biri belirli bir amaca hizmet eden, domain adlarıyla ilişkili farklı veri türlerini depolar:


| Record Type | Full Name                 | Description                                                                                                                        | Zone File Example                                                                          |
| ----------- | ------------------------- | ---------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------ |
| A           | Address Record            | Bir host adını IPv4 adresine eşler.                                                                                                | [www.example.com](http://www.example.com). IN A 192.0.2.1                                  |
| AAAA        | IPv6 Address Record       | Bir host adını IPv6 adresine eşler.                                                                                                | [www.example.com](http://www.example.com). IN AAAA 2001:db8:85a3::8a2e:370:7334            |
| CNAME       | Canonical Name Record     | Bir host adı için takma ad oluşturur ve onu başka bir host adına yönlendirir.                                                      | blog.example.com. IN CNAME webserver.example.net.                                          |
| MX          | Mail Exchange Record      | Domain için e-posta işlemlerini yöneten mail sunucusunu belirtir.                                                                  | example.com. IN MX 10 mail.example.com.                                                    |
| NS          | Name Server Record        | Bir DNS zonunu belirli bir authorized name server'a yönlendirir.                                                                   | example.com. IN NS ns1.example.com.                                                        |
| TXT         | Text Record               | Arbitrar text bilgisi saklar, genellikle domain doğrulama veya güvenlik politikaları için kullanılır.                              | example.com. IN TXT "v=spf1 mx -all" (SPF kaydı)                                           |
| SOA         | Start of Authority Record | Bir DNS zonu hakkında yönetimsel bilgileri belirtir, main name server, sorumlu kişinin e-posta adresi ve diğer parametreler dahil. | example.com. IN SOA ns1.example.com. admin.example.com. 2024060301 10800 3600 604800 86400 |
| SRV         | Service Record            | Belirli hizmetler için host adı ve port numarasını tanımlar.                                                                       | _sip._udp.example.com. IN SRV 10 5 5060 sipserver.example.com.                             |
| PTR         | Pointer Record            | Reverse DNS sorgulamaları için kullanılır, bir IP adresini host adına eşler.                                                       | 1.2.0.192.in-addr.arpa. IN PTR [www.example.com](http://www.example.com).                  |

Örneklerdeki “IN”, “İnternet” anlamına gelir. DNS kayıtlarında protokol ailesini belirten bir sınıf alanıdır. Çoğu durumda, çoğu alan adı için kullanılan İnternet protokol paketini (IP) ifade ettiği için “IN” kullanıldığını görürsünüz. Diğer sınıf değerleri mevcuttur (örneğin, Chaosnet için CH, Hesiod için HS) ancak modern DNS yapılandırmalarında nadiren kullanılır.

Özünde, “IN” sadece kaydın bugün kullandığımız standart internet protokolleri için geçerli olduğunu gösteren bir kuraldır. Fazladan bir ayrıntı gibi görünse de, anlamını anlamak DNS kayıt yapısının daha iyi anlaşılmasını sağlar.


### Web Recon için DNS Neden Önemlidir?

DNS, yalnızca domain çevirisi için teknik bir protokol değildir; aynı zamanda hedefin altyapısının kritik bir bileşenidir.

**Varlıkları Ortaya Çıkarmak**: DNS kayıtları, domain adları, mail sunucuları ve name server kayıtları dahil olmak üzere birçok bilgi ortaya çıkarabilir. 

**Ağ Altyapısını Haritalamak**: DNS verilerini analiz ederek hedefin ağ altyapısının kapsamlı bir haritasını oluşturabilirsiniz. Örneğin, bir domain için name server (NS kayıtları) belirlemek, kullanılan barındırma sağlayıcısını ortaya çıkarabilirken, loadbalancer.example.com için bir A kaydı, bir load blancer'ı işaret edebilir. Bu, farklı sistemlerin nasıl bağlantılı olduğunu anlamanıza, trafik akışını belirlemenize ve penetrasyon testinde istismar edilebilecek potansiyel tıkanma noktalarını veya zayıflıkları belirlemenize yardımcı olur.

**Değişiklikleri İzlemek**: DNS kayıtlarını sürekli izlemek, hedefin altyapısındaki değişiklikleri zamanla ortaya çıkarabilir. Örneğin, yeni bir subdomainin (vpn.example.com) ani bir şekilde ortaya çıkması, ağa yeni bir giriş noktası eklenmiş olabileceğini gösterebilirken, `_1password=...` gibi bir değere sahip bir TXT kaydı, organizasyonun 1Password kullanıyor olabileceğini ve bunun sosyal mühendislik saldırıları veya hedeflenmiş phishing kampanyaları için kullanılabileceğini güçlü bir şekilde ima eder.




















## Reconnaissance Frameworks

Bu frameworkler, web keşifi için tamamlayıcı bir araç seti sunmayı hedefler:

- **FinalRecon:** Python tabanlı bir keşif aracıdır ve SSL sertifikası kontrolü, Whois bilgisi toplama, başlık analizi ve tarama gibi farklı görevler için bir dizi modül sunar. Modüler yapısı, özel ihtiyaçlar için kolayca özelleştirilebilir.

- **Recon-ng:** Python ile yazılmış güçlü bir framework olup, farklı keşif görevleri için çeşitli modüller sunar. DNS sorgulama, subdomain keşfi, port taraması, web tarama ve hatta bilinen güvenlik açıklarını istismar etme gibi görevleri yerine getirebilir.

- **theHarvester:** E-posta adresleri, subdomainler, hostlar, çalışan isimleri, açık portlar ve SHODAN veritabanı gibi farklı kamu kaynaklarından bilgiler toplayan, Python ile yazılmış bir komut satırı aracıdır.
-
- **SpiderFoot:** Bir hedef hakkında IP adresleri, domainleri, e-posta adresleri ve sosyal medya profilleri gibi bilgileri toplamak için çeşitli veri kaynaklarıyla entegre olan açık kaynaklı bir istihbarat otomasyon aracıdır. DNS sorgulamaları, web taramaları, port taramaları ve daha fazlasını yapabilir.

- **OSINT Framework:** Açık kaynak istihbarat toplama için çeşitli araçlar ve kaynaklardan oluşan bir koleksiyondur. Sosyal medya, arama motorları, kamu kayıtları ve daha birçok bilgi kaynağını kapsar.

**Keşif (Reconnaissance) Otomasyonu**

Manuel keşif etkili olabilir, ancak zaman alıcıdır ve insan hatalarına açıktır. Web keşif görevlerini otomatikleştirmek, verimliliği ve doğruluğu önemli ölçüde artırabilir, böylece daha hızlı bir şekilde bilgi toplayabilir ve potansiyel güvenlik açıklarını daha hızlı tespit edebilirsiniz.

### Keşif Neden Otomatikleştirilmeli?

Otomasyonun web keşifinde sağladığı birkaç önemli avantaj vardır:

- **Verimlilik:** Otomatik araçlar, tekrarlayan görevleri insanlardan çok daha hızlı gerçekleştirebilir, böylece analiz ve karar verme için değerli zaman kazandırır.
- **Ölçeklenebilirlik:** Otomasyon, keşif çabalarınızı çok sayıda hedef veya alan adı üzerinde ölçeklendirmeyi sağlar, bu da daha geniş bir bilgi kapsamı sağlar.
- **Tutarlılık:** Otomatik araçlar, önceden tanımlanmış kuralları ve prosedürleri takip eder, bu da tutarlı ve tekrar edilebilir sonuçlar sağlar ve insan hatası riskini minimize eder.
- **Kapsamlı Kapsama:** Otomasyon, DNS sorgulama, alt alan adı keşfi, web tarama, port tarama ve daha fazlası gibi keşif görevlerinin geniş bir yelpazesinde kullanılabilir, bu da potansiyel saldırı vektörlerinin kapsamlı bir şekilde değerlendirilmesini sağlar.
- **Entegrasyon:** Birçok otomasyon çerçevesi, keşiften zafiyet değerlendirmesine ve istismarına kadar kesintisiz bir iş akışı sağlamak için diğer araçlarla ve platformlarla kolayca entegrasyon sağlar.

### Keşif Çerçeveleri

Bu çerçeveler, web keşifi için tamamlayıcı bir araç seti sunmayı hedefler:

- **FinalRecon:** Python tabanlı bir keşif aracıdır ve SSL sertifikası kontrolü, Whois bilgisi toplama, başlık analizi ve tarama gibi farklı görevler için bir dizi modül sunar. Modüler yapısı, özel ihtiyaçlar için kolayca özelleştirilebilir.
- **Recon-ng:** Python ile yazılmış güçlü bir çerçeve olup, farklı keşif görevleri için çeşitli modüller sunar. DNS sorgulama, alt alan adı keşfi, port taraması, web tarama ve hatta bilinen güvenlik açıklarını istismar etme gibi görevleri yerine getirebilir.
- **theHarvester:** E-posta adresleri, alt alan adları, ana bilgisayarlar, çalışan isimleri, açık portlar ve SHODAN veritabanı gibi farklı kamu kaynaklarından bilgiler toplayan, Python ile yazılmış bir komut satırı aracıdır.
- **SpiderFoot:** Bir hedef hakkında IP adresleri, alan adları, e-posta adresleri ve sosyal medya profilleri gibi bilgileri toplamak için çeşitli veri kaynaklarıyla entegre olan açık kaynaklı bir istihbarat otomasyon aracıdır. DNS sorgulamaları, web taramaları, port taramaları ve daha fazlasını yapabilir.
- **OSINT Framework:** Açık kaynak istihbarat toplama için çeşitli araçlar ve kaynaklardan oluşan bir koleksiyondur. Sosyal medya, arama motorları, kamu kayıtları ve daha birçok bilgi kaynağını kapsar.

### FinalRecon

FinalRecon, keşif için oldukça kapsamlı bir bilgi sunar:

- **Header Bilgisi:** Sunucu ayrıntıları, kullanılan teknolojiler ve olası güvenlik yapılandırma hatalarını ortaya çıkarır.
- **Whois Sorgulama:** Domain kayıt bilgilerini, kayıt sahibi bilgileri ve iletişim detaylarını ortaya koyar.
- **SSL Sertifikası Bilgisi:** SSL/TLS sertifikasını geçerlilik, sağlayıcı ve diğer önemli detaylarla inceler.
- **Tarayıcı:**
    - **HTML, CSS, JavaScript:** Bu dosyalardan bağlantılar, kaynaklar ve potansiyel güvenlik açıkları çıkarır.
    - **İç ve Dış Bağlantılar:** Web sitesinin yapısını haritalar ve diğer domainlere olan bağlantıları tespit eder.
    - **Görseller, robots.txt, sitemap.xml:** Tarama izinli/izinsiz yollar ve web site yapısı hakkında bilgi toplar.
    - **JavaScript Bağlantıları, Wayback Machine:** Gizli bağlantıları ve geçmişteki web sitesi verilerini keşfeder.
- **DNS Sorgulama:** DMARC kayıtları da dahil olmak üzere 40'tan fazla DNS kaydını sorgular ve e-posta güvenliği değerlendirmesi yapar.
- **Subdomain Keşfi:** domain adı keşfi için birden fazla veri kaynağını (crt.sh, AnubisDB, ThreatMiner, CertSpotter, Facebook API, VirusTotal API, Shodan API, BeVigil API) kullanır.
- **Dizin Keşfi:** Özel kelime listeleri ve dosya uzantıları kullanarak gizli dizinler ve dosyalar tespit eder.
- **Wayback Machine:** Son beş yıl içinde URL'leri alarak web sitesi değişikliklerini ve potansiyel güvenlik açıklarını analiz eder.

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/thewhiteh4t/FinalRecon.git
M1R4CKCK@htb[/htb]$ cd FinalRecon
M1R4CKCK@htb[/htb]$ pip3 install -r requirements.txt
M1R4CKCK@htb[/htb]$ chmod +x ./finalrecon.py
M1R4CKCK@htb[/htb]$ ./finalrecon.py --help

usage: finalrecon.py [-h] [--url URL] [--headers] [--sslinfo] [--whois]
                     [--crawl] [--dns] [--sub] [--dir] [--wayback] [--ps]
                     [--full] [-nb] [-dt DT] [-pt PT] [-T T] [-w W] [-r] [-s]
                     [-sp SP] [-d D] [-e E] [-o O] [-cd CD] [-k K]

FinalRecon - All in One Web Recon | v1.1.6

optional arguments:
  -h, --help  show this help message and exit
  --url URL   Target URL
  --headers   Header Information
  --sslinfo   SSL Certificate Information
  --whois     Whois Lookup
  --crawl     Crawl Target
  --dns       DNS Enumeration
  --sub       Sub-Domain Enumeration
  --dir       Directory Search
  --wayback   Wayback URLs
  --ps        Fast Port Scan
  --full      Full Recon

Extra Options:
  -nb         Hide Banner
  -dt DT      Number of threads for directory enum [ Default : 30 ]
  -pt PT      Number of threads for port scan [ Default : 50 ]
  -T T        Request Timeout [ Default : 30.0 ]
  -w W        Path to Wordlist [ Default : wordlists/dirb_common.txt ]
  -r          Allow Redirect [ Default : False ]
  -s          Toggle SSL Verification [ Default : True ]
  -sp SP      Specify SSL Port [ Default : 443 ]
  -d D        Custom DNS Servers [ Default : 1.1.1.1 ]
  -e E        File Extensions [ Example : txt, xml, php ]
  -o O        Export Format [ Default : txt ]
  -cd CD      Change export directory [ Default : ~/.local/share/finalrecon ]
  -k K        Add API key [ Example : shodan@key ]
```


[[Bağlantılar/Sanal ortamada kurulabilir ayrıntı için tıkla\|Sanal ortamada kurulabilir ayrıntı için tıkla]]

Başlamak için önce git clone https://github.com/thewhiteh4t/FinalRecon.git adresini kullanarak GitHub'dan FinalRecon deposunu klonlayacaksınız. Bu, gerekli tüm dosyaları içeren “FinalRecon” adlı yeni bir dizin oluşturacaktır.

Ardından, cd FinalRecon ile yeni oluşturulan dizine gidin. İçeri girdikten sonra, pip3 install -r requirements.txt kullanarak gerekli Python bağımlılıklarını yükleyeceksiniz. Bu, FinalRecon'un düzgün çalışması için gereken tüm kütüphanelere ve modüllere sahip olmasını sağlar.

Ana script'in çalıştırılabilir olduğundan emin olmak için chmod +x ./finalrecon.py kullanarak dosya izinlerini değiştirmeniz gerekecektir. Bu, script'i doğrudan terminalinizden çalıştırmanıza olanak tanır.

Son olarak, FinalRecon'un doğru şekilde kurulduğunu doğrulayabilir ve ./finalrecon.py --help komutunu çalıştırarak mevcut seçeneklerine genel bir bakış elde edebilirsiniz. Bu, çeşitli modüller ve ilgili seçenekleri de dahil olmak üzere aracın nasıl kullanılacağına ilişkin ayrıntıları içeren bir yardım mesajı görüntüler:

|Seçenek|Argüman|Açıklama|
|---|---|---|
|-h, --help||Yardım mesajını göster ve çık.|
|--url|URL|Hedef URL'yi belirt.|
|--headers||Hedef URL için başlık bilgilerini al.|
|--sslinfo||Hedef URL için SSL sertifikası bilgilerini al.|
|--whois||Hedef domain için Whois sorgulaması yap.|
|--crawl||Hedef web sitesini tarar.|
|--dns||Hedef domain için DNS keşfi yapar.|
|--sub||Hedef domain için alt alan adı keşfi yapar.|
|--dir||Hedef web sitesinde dizin araması yapar.|
|--wayback||Wayback Machine URL'lerini alır.|
|--ps||Hedef için hızlı bir port taraması yapar.|
|--full||Hedef üzerinde tam keşif taraması yapar.|

Örneğin, FinalRecon'un header bilgilerini toplamasını ve inlanefreight.com için bir Whois araması yapmasını istiyorsak, ilgili bayrakları (--headers ve --whois) kullanırız, böylece komut şöyle olur:

```shell-session
M1R4CKCK@htb[/htb]$ ./finalrecon.py --headers --whois --url http://inlanefreight.com

 ______  __   __   __   ______   __
/\  ___\/\ \ /\ "-.\ \ /\  __ \ /\ \
\ \  __\\ \ \\ \ \-.  \\ \  __ \\ \ \____
 \ \_\   \ \_\\ \_\\"\_\\ \_\ \_\\ \_____\
  \/_/    \/_/ \/_/ \/_/ \/_/\/_/ \/_____/
 ______   ______   ______   ______   __   __
/\  == \ /\  ___\ /\  ___\ /\  __ \ /\ "-.\ \
\ \  __< \ \  __\ \ \ \____\ \ \/\ \\ \ \-.  \
 \ \_\ \_\\ \_____\\ \_____\\ \_____\\ \_\\"\_\
  \/_/ /_/ \/_____/ \/_____/ \/_____/ \/_/ \/_/

[>] Created By   : thewhiteh4t
 |---> Twitter   : https://twitter.com/thewhiteh4t
 |---> Community : https://twc1rcle.com/
[>] Version      : 1.1.6

[+] Target : http://inlanefreight.com

[+] IP Address : 134.209.24.248

[!] Headers :

Date : Tue, 11 Jun 2024 10:08:00 GMT
Server : Apache/2.4.41 (Ubuntu)
Link : <https://www.inlanefreight.com/index.php/wp-json/>; rel="https://api.w.org/", <https://www.inlanefreight.com/index.php/wp-json/wp/v2/pages/7>; rel="alternate"; type="application/json", <https://www.inlanefreight.com/>; rel=shortlink
Vary : Accept-Encoding
Content-Encoding : gzip
Content-Length : 5483
Keep-Alive : timeout=5, max=100
Connection : Keep-Alive
Content-Type : text/html; charset=UTF-8

[!] Whois Lookup : 

   Domain Name: INLANEFREIGHT.COM
   Registry Domain ID: 2420436757_DOMAIN_COM-VRSN
   Registrar WHOIS Server: whois.registrar.amazon.com
   Registrar URL: http://registrar.amazon.com
   Updated Date: 2023-07-03T01:11:15Z
   Creation Date: 2019-08-05T22:43:09Z
   Registry Expiry Date: 2024-08-05T22:43:09Z
   Registrar: Amazon Registrar, Inc.
   Registrar IANA ID: 468
   Registrar Abuse Contact Email: abuse@amazonaws.com
   Registrar Abuse Contact Phone: +1.2024422253
   Domain Status: clientDeleteProhibited https://icann.org/epp#clientDeleteProhibited
   Domain Status: clientTransferProhibited https://icann.org/epp#clientTransferProhibited
   Domain Status: clientUpdateProhibited https://icann.org/epp#clientUpdateProhibited
   Name Server: NS-1303.AWSDNS-34.ORG
   Name Server: NS-1580.AWSDNS-05.CO.UK
   Name Server: NS-161.AWSDNS-20.COM
   Name Server: NS-671.AWSDNS-19.NET
   DNSSEC: unsigned
   URL of the ICANN Whois Inaccuracy Complaint Form: https://www.icann.org/wicf/


[+] Completed in 0:00:00.257780

[+] Exported : /home/htb-ac-643601/.local/share/finalrecon/dumps/fr_inlanefreight.com_11-06-2024_11:07:59
```
