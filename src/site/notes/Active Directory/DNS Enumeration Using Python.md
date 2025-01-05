---
{"dg-publish":true,"permalink":"/active-directory/dns-enumeration-using-python/"}
---



Bir DNS servisinin bileşenleri şunlardan oluşur:
- `Name servers`
- `Zones`
- `Domain names`
- `IP addresses`

Name server'lar zone veya zone dosyaları olarak adlandırılan dosyalar içerir. Basit bir ifadeyle, zone dosyaları ilgili domain için kayıt listeleridir. Bunu belirli bir şehir için bir telefon rehberi gibi düşünebiliriz. Bu bölge dosyaları belirli domainlerin ve hostların IP adreslerini içerir. Böyle bir zone dosyası ister Linux ister Windows olsun her sistemde bulunur ve “hosts” dosyası olarak adlandırılır. Bunları Linux'ta “/etc/hosts” altında bulabiliriz. Bu dosya bizim local zone dosyamızdır.

Örnek olarak aşağıdaki tam nitelikli domain adını (FQDN) ele alalım:

- `www.domain-A.com`

Domain, bilgisayarların IP adreslerine gerçek isimler vermek ve aynı zamanda onları hiyerarşik bir yapıya bölmek için kullanılır. Bu hiyerarşik yapı, bir işletim sisteminin dizin ağacına benzer.

```shell-session
.
├── com.
│   ├── domain-A.
│   │   ├── blog.
│   │   ├── ns1.
│   │   └── www.
│   │ 
│   ├── domain-B.
│   │   └── ...
│   └── domain-C.
│       └── ...
│
└── net.
│   ├── domain-D.
│   │   └── ...
│   ├── domain-E.
│   │   └── ...
│   └── domain-F.
│       └── ...
└── ...
│   ├── ...
│   └── ...
```

![Pasted image 20241029143748.png](/img/user/resimler/Pasted%20image%2020241029143748.png)

Her bir domain en az iki bölümden oluşur:

1. `Top-Level Domain` (`TLD`)
2. `Domain Name`

Son örneğe göre, domain adı “inlanefreight” ve TLD de “com” olacaktır. “inlanefreight ”a bakarsak, bazı sözde subdomainler (dev, www, mail) içerdiğini görürüz. Bu subdomainler tek bir hostu veya sanal hostları (vHosts) temsil edebilir. DNS sunucuları dört farklı türe ayrılır:

- `Recursive resolvers` (`DNS Recursor`)
- `Root name server`
- `TLD name server`
- `Authoritative name servers`


### Recursive DNS Resolver
Recursive resolver, client ve name server arasında bir aracı görevi görür. Rekursif resolver bir web client'tan bir DNS sorgusu aldıktan sonra, bu sorguya cache'lenmiş verilerle yanıt verir veya bir root name server'a bir sorgu gönderir, ardından bir TLD name server'a bir sorgu gönderir ve son olarak yetkili bir name server'a son bir sorgu gönderir. Yetkili ad sunucusundan istenen IP adresini içeren bir yanıt aldıktan sonra, tekrarlamalı çözümleyici istemcinin yanıtını gönderir.

Bu işlem sırasında, rekürsif resolver yetkili isim sunucularından aldığı bilgileri önbelleğinde saklar. Bir istemci, yakın zamanda başka bir istemci tarafından talep edilen bir domain adının IP adresini talep ettiğinde, resolver name server'larla iletişimi atlayabilir ve talep edilen girdiyi önbelleğinden alıp istemciye gönderebilir.


### Root Name Server
IPv4 ve IPv6 adresleri altında on üç root isim sunucusuna ulaşılabilir. Kâr amacı gütmeyen uluslararası bir kuruluş olan Internet Corporation for Assigned Names and Numbers (ICANN) bu root name sunucularının bakımını yapmaktadır. Bunların zone dosyaları TLD'lerin tüm domain isimlerini ve IP adreslerini içerir. Her recursive resolver bu 13 root isim sunucusunu bilir. Bunlar, her rekürsif resolver için DNS girişleri aramasındaki ilk istasyonlardır. Bu root ad sunucularının her biri, bir domain adı içeren bir recursive resolver sorgusunu kabul eder. Domain uzantısı sorguyu yanıtlar ve recursive resolver'ı ilgili TLD name server'a yönlendirir. Root-servers.net alan adındaki 13 root ad sunucusunu, subdomain olarak karşılık gelen harfle birlikte buluyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ dig ns root-servers.net | grep NS | sort -u                                                                          

; EDNS: version: 0, flags:; udp: 4096
;; ANSWER SECTION:
;; flags: qr rd ra; QUERY: 1, ANSWER: 13, AUTHORITY: 0, ADDITIONAL: 1
;root-servers.net.		IN	NS
root-servers.net.	6882	IN	NS	a.root-servers.net.
root-servers.net.	6882	IN	NS	b.root-servers.net.
root-servers.net.	6882	IN	NS	c.root-servers.net.
root-servers.net.	6882	IN	NS	d.root-servers.net.
root-servers.net.	6882	IN	NS	e.root-servers.net.
root-servers.net.	6882	IN	NS	f.root-servers.net.
root-servers.net.	6882	IN	NS	g.root-servers.net.
root-servers.net.	6882	IN	NS	h.root-servers.net.
root-servers.net.	6882	IN	NS	i.root-servers.net.
root-servers.net.	6882	IN	NS	j.root-servers.net.
root-servers.net.	6882	IN	NS	k.root-servers.net.
root-servers.net.	6882	IN	NS	l.root-servers.net.
root-servers.net.	6882	IN	NS	m.root-servers.net.
```

Bu 13 root name server, 13 farklı root name server türünü temsil etmektedir. Bu sadece 13 host üzerinde yayıldığı anlamına gelmez, dünya çapında bu root name server'ların 600'den fazla kopyası vardır.


### TLD Name Server
Bir TLD name server, aynı TLD'ye sahip tüm domain adları hakkındaki bilgileri yönetir. Bu TLD ad sunucuları İnternet Tahsisli Sayılar Kurumu'nun (IANA) sorumluluğundadır ve onun tarafından yönetilir. Bu, “.com” TLD'si altındaki tüm alan adlarının ilgili TLD ad sunucusu tarafından yönetildiği anlamına gelir. Bu TLD altında kayıtlı bir domain aradığımızda, recursive resolver, root name server'dan bir yanıt aldıktan sonra, o TLD'den sorumlu TLD name server'a bir sorgu gönderir.


### Authoritative Name Server
Yetkili ad sunucuları, domainler için DNS kayıt bilgilerini depolar. Bu sunucular, bir web sayfasının IP adresini ve diğer DNS girişlerini içeren ad sunucularından gelen isteklere yanıt vermekten sorumludur, böylece web sayfası istemci tarafından adreslenebilir ve erişilebilir. Bir özyinelemeli çözümleyici bir TLD ad sunucusu yanıtı aldığında, yanıt onu yetkili bir ad sunucusuna yönlendirir. Yetkili ad sunucusu, bir IP adresi almak için son adımdır.

# DNS Zones

### Primary DNS Server
Birincil DNS sunucusu, bir domain için tüm yetkili bilgileri içeren ve bu bölgenin yönetiminden sorumlu olan bölge dosyasının sunucusudur. Bir bölgenin DNS kayıtları yalnızca birincil DNS sunucusunda düzenlenebilir ve bu da ikincil DNS sunucularını günceller.


### Secondary DNS Server
İkincil DNS sunucuları, birincil DNS sunucusundaki zone dosyasının salt okunur kopyalarını içerir. Bu sunucular verilerini düzenli aralıklarla birincil DNS sunucusuyla karşılaştırır ve böylece yedek sunucu olarak hizmet verir. Birincil ad sunucusunun arızalanması, ad çözümlemesi olmayan bağlantıların artık mümkün olmadığı anlamına geldiğinden yararlıdır. Yine de bağlantı kurmak için kullanıcının bağlantı kurulan sunucuların IP adreslerini bilmesi gerekir. Bununla birlikte, bu bir kural değildir.


### Zone Files
Zone dosyalarının belirli bir domain için ilgili kayıtları ve bilgileri içerdiğini zaten biliyoruz. Burada aşağıdakiler arasında ayrım yapıyoruz:

- `Primary zone` (`master zone`)
- `Secondary zone` (`slave zone`)

Sekonder DNS sunucusundaki sekonder zone, birincil DNS sunucusuna erişilememesi durumunda birincil DNS sunucusundaki birincil bölgenin yerine geçer. Birincil bölge kopyasının oluşturulması ve birincil DNS sunucusundan ikincil DNS sunucusuna aktarılmasına “ zone transferi” denir.

zone dosyalarının güncelleştirilmesi yalnızca birincil DNS sunucusunda yapılabilir ve bu sunucu daha sonra ikincil DNS sunucusunu güncelleştirir. Her zone dosyası yalnızca bir birincil DNS sunucusuna ve sınırsız sayıda ikincil DNS sunucusuna sahip olabilir.

Ayrıca “.com” TLD'sini de düşünebiliriz, örneğin belirli bir departman için sadece ekip liderlerini tanıyan ve yöneten bir şirket yöneticisi (Bölge 1). Her ekip lideri de sadece kendi yöneticisini ve ekip üyelerini tanır (Bölge 2 ve Bölge 3).

#### Zone 1
```shell-session
.
└── com.
    ├── domain-A.
    ├── domain-B.
    └── domain-C.
```

#### Zone 2
```shell-session
com.
└── domain-A.
    ├── blog.
    ├── ns1.
    └── www.
```


#### Zone 3
```shell-session
com.
└── domain-B.
    └── www2.
```

## Zone Transfer
İki farklı türde zone transferi vardır.
- `AXFR` - Asynchronous Full Transfer Zone
- `IXFR` - Incremental Zone Transfer

AXFR, zone dosyasının tüm girişlerinin eksiksiz bir aktarımıdır. Full asenkron aktarımın aksine, bir IXFR için zone dosyasının yalnızca değişen ve yeni DNS kayıtları ikincil DNS sunucularına aktarılır.

DNS sunucuları ve zone aktarımları ile ilgili sorun, kimlik doğrulama gerektirmemesi ve herhangi bir istemci tarafından talep edilebilmesidir. Yönetici, bu bölgeleri alma izni olan DNS sunucuları için “Güvenilir Hostlar/IP adresleri” ayarlamamışsa, tüm bölge dosyasını içeriğiyle birlikte de sorgulayabiliriz. Bu bize ilgili hostlarla birlikte tüm IP adreslerini verecek ve eğer anlaşma kapsamımızı sınırlamıyorsa saldırı vektörümüzü önemli ölçüde artıracaktır. Bugün bile, DNS sunucularının yanlış yapılandırılması ve buna izin vermesi oldukça yaygındır.


### DNS Kayıtları ve Sorguları
DNS birçok farklı kayıtla çalışır. DNS kayıtları, yetkili DNS sunucularında bulunan ve bir domain hakkında bilgi içeren talimatlardır. Bu kayıtlar, DNS sunucularına uygun talimatları veren DNS sözdiziminde yazılır. Sızma Testlerimiz sırasında karşılaşacağımız en yaygın DNS kayıtları şunlardır:
![Pasted image 20241029145841.png](/img/user/resimler/Pasted%20image%2020241029145841.png)


### NS Servers
Oluşturduğumuz araçla bu güvenlik açığını hızlı bir şekilde kontrol etmek için yanlış yapılandırılmış DNS sunucularındaki olası zone transferleri için numaralandırma işlemini otomatikleştirmek istiyoruz. Bazı NS sunucularını numaralandırarak ve bir DNS zone transferinin mümkün olup olmadığını kontrol ederek ve eğer mümkünse, zone transferinden subdomainler hakkında bilgi alarak başlayalım.


### NS Records
NS kayıtları name server anlamına gelir. Bu kayıtlar, gerçek DNS kayıtlarını içeren ilgili domain için hangi DNS sunucusunun yetkili sunucu olduğunu belirtir. Genellikle bir domainin, o domain için birincil ve ikincil ad sunucularını belirleyebilen birkaç NS kaydı vardır. Artık hedef domainimizi bildiğimize göre, hala etkileşimde bulunacağımız DNS sunucularına ihtiyacımız var. Bunun için, domain için hangi DNS sunucularının sorumlu olduğunu bulmalıyız ve bunun için dig adlı aracı kullanabiliriz. Bu örnekte, inlanefreight.com adlı domaini kullanıyoruz

#### DIG - NS Queries
```shell-session
M1R4CKCK@htb[/htb]$ dig NS inlanefreight.com

<SNIP>
;; ANSWER SECTION:
inlanefreight.com.	60	IN	NS	ns2.inlanefreight.com.
inlanefreight.com.	60	IN	NS	ns1.inlanefreight.com.

<SNIP>
```

#### DIG - SOA Queries
```shell-session
M1R4CKCK@htb[/htb]$ dig SOA inlanefreight.com

<SNIP>
;; ANSWER SECTION:
inlanefreight.com.	879	IN	SOA	ns-161.awsdns-20.com. awsdns-hostmaster.amazon.com. 1 7200 900 1209600 86400

<SNIP>
```


#### NSLOOKUP - SPF
```shell-session
M1R4CKCK@htb[/htb]$ nslookup -type=SPF inlanefreight.com

Non-authoritative answer:
inlanefreight.com	rdata_99 = "v=spf1 include:_spf.google.com include:mail1.inlanefreight.com include:google.com ~all"
```
`nslookup -type=SPF inlanefreight.com` komutu, `inlanefreight.com` domaininin **SPF (Sender Policy Framework)** kaydını sorgulamak için kullanılır.

- **SPF Kaydı**: E-posta gönderiminde, hangi sunucuların bu domain adına e-posta göndermeye yetkili olduğunu belirten bir kayıt türüdür. Bu, sahte e-posta gönderimini önlemeye yardımcı olur.
- **Sonuç**: Bu komutun çıktısı, `inlanefreight.com` domaininin SPF ayarlarını içerir ve hangi IP adreslerinin veya domainlerin e-posta göndermeye yetkili olduğunu gösterir.

Yani, bu sorguyla `inlanefreight.com` domaininin e-posta gönderim politikalarını öğrenirsiniz.


#### NSLOOKUP - DMARC
```shell-session
M1R4CKCK@htb[/htb]$ nslookup -type=txt _dmarc.inlanefreight.com

Non-authoritative answer:
_dmarc.inlanefreight.com	text = "v=DMARC1; p=reject; rua=mailto:master@inlanefreight.com; ruf=mailto:master@inlanefreight.com; fo=1;"
```

`nslookup -type=txt _dmarc.inlanefreight.com` komutu, **DMARC** kaydını almak için kullanılır. Kısaca şöyle açıklayabiliriz:

- **DMARC Kaydı**: Bir domainin, e-posta güvenliği kurallarını içerir.
- Bu kayıt sayesinde, domainin e-postaları doğrulama yöntemi ve sahte e-postalarla nasıl başa çıkılacağı belirlenir.

Bu komut, `inlanefreight.com` domaini için DMARC politikasını gösterir, yani hangi e-postaların güvenilir sayılacağı ve sahte e-postalarla nasıl başa çıkılacağı bilgisini verir.


Sorular :
* Dig veya nslookup yardımıyla “inlanefreight.com” domainine ait tüm kayıtları araştırın ve cevap olarak çift tırnak içinde benzersiz bir kayıt gönderin.

1- dig inlanefreight.com TXT

* “inlanefreight.com” domainine karşılık gelen IPv6 adresini bulun ve cevap olarak gönderin.

2- dig inlanefreight.com AAAA



### DNS Security
DNS Threat Intelligence, diğer açık kaynak ve diğer tehdit istihbaratı feed'leriyle entegre edilebilir. EDR (Endpoint Detection and Response) ve SIEM (Security Information and Event Management) gibi analitik sistemleri, güvenlik durumunun bütünsel ve duruma dayalı bir görünümünü sağlayabilir. DNS Security Services, IOC'leri ( Compromise Indicators) ve IOA'ları ( Indicators of Attacks) güvenlik duvarları, ağ proxy'leri, endpoint security, Network Access Control (NACs) ve güvenlik açığı tarayıcıları gibi diğer güvenlik teknolojileriyle paylaşarak ve onlara bilgi sağlayarak olay müdahalesinin koordinasyonunu destekler.


### DNSSEC
DNS sunucularının güvenliği için kullanılan bir diğer feed, kaynak kayıtlarını dijital sertifikalarla güvence altına alarak Domain Name System aracılığıyla iletilen verilerin gerçekliğini ve bütünlüğünü sağlamak için tasarlanmış Domain Name System Security Extensions (DNSSEC)'dir. DNSSEC, DNS verilerinin manipüle edilmemesini ve başka bir kaynaktan gelmemesini sağlar. Resource Recors dijital olarak imzalamak için private key'ler kullanılır. Resource Records, örneğin zaman içinde süresi dolan anahtarları değiştirmek için farklı private key'lerle birkaç kez imzalanabilir


### Private Key
Güvenliği sağlanacak bir zonu yöneten DNS sunucusu, gönderilen resource kayıtlarını bilinen tek private key'ini kullanarak imzalar. Her zone'nin, her biri bir private ve bir public anahtardan oluşan zone anahtarları vardır. DNSSEC, RRSIG ile yeni bir resource record türü belirtir. İlgili DNS kaydının imzasını içerir ve kullanılan bu anahtarların belirli bir geçerlilik süresi vardır ve bir başlangıç ve bitiş tarihi ile sağlanır.


### Public Key
Public key, veri alıcılarının imzasını doğrulamak için kullanılabilir. DNSSEC güvenlik mekanizmaları için, DNS bilgilerinin sağlayıcısı ve talep eden istemci sistemi tarafından desteklenmesi gerekir. Talepte bulunan istemciler, DNS bölgesinin genel olarak bilinen public anahtarını kullanarak imzaları doğrular. Bir kontrol başarılı olursa, yanıtı manipüle etmek imkansızdır ve bilgi istenen kaynaktan gelir.


# DNS Enumeration
- `DNS Records`
- `Subdomains`/`Hosts`
- `DNS Security`

Bunun için kullanılabilecek çeşitli teknikler vardır. Bunlar şunları içerir:
- OSINT
- Certificate Transparency
- Zone transfer


### OSINT
![Pasted image 20241029200548.png](/img/user/resimler/Pasted%20image%2020241029200548.png)
https://www.recordedfuture.com/threat-intelligence-101/threat-analysis-techniques/google-dorks

Ayrıca, ilgili domain için bilinen girdileri okumak için VirusTotal, DNSdumpster, Netcraft ve diğerleri gibi genel servisleri kullanabiliriz. 


#### VirusTotal
![Pasted image 20241029200704.png](/img/user/resimler/Pasted%20image%2020241029200704.png)


#### DNSdumpster
![Pasted image 20241029200716.png](/img/user/resimler/Pasted%20image%2020241029200716.png)


#### Netcraft
![Pasted image 20241029200727.png](/img/user/resimler/Pasted%20image%2020241029200727.png)


### Certificate Transparency
Certificate Transparency (CT) günlükleri, katılımcı bir Sertifika Yetkilisi (CA) tarafından belirli bir etki alanı için verilen tüm sertifikaları içerir. Bu nedenle, web sunucularından alınan SSL/TLS sertifikaları domain adlarını, subdomain adlarını ve e-posta adreslerini içerir. Bu günlükler herkese açık ve herkes tarafından erişilebilir olduğundan, hedef şirketin altyapısını daha iyi anlamak için değerli bir kaynaktır. Hedef alanımız için farklı kaynaklardan gelen ve filtrelenmiş tüm CT loglarını çıktı olarak veren bir araç olan [ctfr.py](https://github.com/UnaPibaGeek/ctfr)'yi kullanabiliriz.

#### CTFR.py
```shell-session
M1R4CKCK@htb[/htb]$ python3 ctfr.py -d inlanefreight.com

          ____ _____ _____ ____  
         / ___|_   _|  ___|  _ \ 
        | |     | | | |_  | |_) |
        | |___  | | |  _| |  _ < 
         \____| |_| |_|   |_| \_\
	
    Made by Sheila A. Berta (UnaPibaGeek)
	
inlanefreight.com
vc.inlanefreight.com
wlan.inlanefreight.com
afdc0102.inlanefreight.com
autodiscover.inlanefreight.com
kfdcex07.inlanefreight.com
kfdcex08.inlanefreight.com
videoconf.inlanefreight.com
<SNIP>
```

## Zone Transfer
DNS'de Zone transferi, zone'ların diğer DNS sunucularına aktarılması anlamına gelir. Bu prosedüre daha önce öğrendiğimiz gibi Asynchronous Full Transfer Zone (AXFR) adı verilir. Bir DNS arızası genellikle bir şirket için ciddi sonuçlar doğurduğundan, zone dosyaları neredeyse istisnasız olarak birkaç nameserver'da aynı tutulur. Değişiklik olması durumunda, tüm sunucuların aynı veri stokuna sahip olması sağlanmalıdır. Zone transferi sadece dosyaların veya kayıtların transferini ve ilgili sunucuların veritabanlarındaki tutarsızlıkların tespitini içerir. 

```shell-session
M1R4CKCK@htb[/htb]$ dig axfr inlanefreight.com @10.129.2.67

; <<>> DiG 9.16.1-Ubuntu <<>> axfr inlanefreight.com @10.129.2.67
;; global options: +cmd
inlanefreight.com.	    3600	IN	SOA	ns1.inlanefreight.com. adm.inlanefreight.com. 8 3600 300 86400 600
inlanefreight.com.	    3600	IN	A	178.128.39.165
inlanefreight.com.   	3600	IN	A	206.189.119.186
inlanefreight.com.	    3600	IN	NS	ns1.inlanefreight.com.
inlanefreight.com.	    3600	IN	NS	ns2.inlanefreight.com.
adm.inlanefreight.com.	3600	IN	A	134.209.24.248
blog.inlanefreight.com.	3600	IN	A	134.209.24.248
<SNIP>
```

veya syntax'ı bu 

```shell-session
dig inlanefreight.com AXFR @ns1.inlanefreight.com
```

Soru 1 : Hedefinize karşı “inlanefreight.htb” domain'i için bir zone transferi gerçekleştirin ve şirketin kaç tane nameserver'ı olduğunu belirleyin. Toplam nameserver sayısını cevap olarak gönderin.

Cevap : 

```shell-session
dig inlanefreight.htb AXFR @10.129.146.26

; <<>> DiG 9.20.0-Debian <<>> inlanefreight.htb AXFR @10.129.146.26
;; global options: +cmd
inlanefreight.htb.      3600    IN      SOA     ns1.inlanefreight.htb. adm.inlanefreight.htb. 8 3600 300 86400 600
inlanefreight.htb.      3600    IN      A       10.129.2.55
inlanefreight.htb.      3600    IN      A       10.129.2.58
inlanefreight.htb.      3600    IN      NS      ns1.inlanefreight.htb.
inlanefreight.htb.      3600    IN      NS      ns2.inlanefreight.htb.
admin.inlanefreight.htb. 3600   IN      A       134.209.24.248
app.inlanefreight.htb.  3600    IN      A       134.209.24.248
blog.inlanefreight.htb. 3600    IN      A       134.209.24.248
calvin.inlanefreight.htb. 3600  IN      A       134.209.24.248
cry0l1t3.inlanefreight.htb. 3600 IN     A       134.209.24.248
customers.inlanefreight.htb. 3600 IN    A       134.209.24.248
help.inlanefreight.htb. 3600    IN      CNAME   support.inlanefreight.htb.
jerry.inlanefreight.htb. 3600   IN      A       134.209.24.248
key.inlanefreight.htb.  3600    IN      TXT     "HTB{mgraRNhvDCPcKpzAXHA6cJUhW}"
key.inlanefreight.htb.  3600    IN      A       134.209.24.248
mrb3n.inlanefreight.htb. 3600   IN      A       134.209.24.248
ns1.inlanefreight.htb.  3600    IN      A       10.129.2.55
ns2.inlanefreight.htb.  3600    IN      A       10.129.2.58
vanny.inlanefreight.htb. 3600   IN      A       134.209.24.248
vpn.inlanefreight.htb.  3600    IN      A       134.209.24.248
inlanefreight.htb.      3600    IN      SOA     ns1.inlanefreight.htb. adm.inlanefreight.htb. 8 3600 300 86400 600
;; Query time: 163 msec
;; SERVER: 10.129.146.26#53(10.129.146.26) (TCP)
;; WHEN: Tue Oct 29 13:22:21 EDT 2024
;; XFR size: 21 records (messages 1, bytes 563)

```

4 Sorunun cevabı cıktıda mevcut. 

---

## Programming Guidelines

![Pasted image 20241029203435.png](/img/user/resimler/Pasted%20image%2020241029203435.png)


### PEP8
Özellikle Python için geliştirilen ve Python Geliştirme Önerisi ([PEP8](https://peps.python.org/pep-0008/)) olarak bilinen yönergeler vardır. PEP8, Python kodu yazmak için yönergeler ve en iyi uygulamaları içeren bir belgedir ve 2001 yılında Guido van Rossum, Barry Warsaw ve Nick Coghlan tarafından yazılmıştır. PEP8'in birincil odak noktası Python kodunun okunabilirliğini ve tutarlılığını geliştirmektir. Python kodu yazma konusunda daha fazla deneyime sahip olduğumuzda, başkalarıyla çalışmaya başlayabiliriz. Okunabilir kod yazmak çok önemlidir çünkü kodlama tarzımıza aşina olmayan diğer kişilerin kodumuzu okuması ve anlaması gerekir. Takip ettiğimiz ve tanıdığımız yönergelerimiz varsa, başkaları da kodumuzun okunmasını daha kolay bulacaktır.

![Pasted image 20241029203605.png](/img/user/resimler/Pasted%20image%2020241029203605.png)


### Python Modülleri
Aracımızı kodlamaya başlamadan önce, hangi modüllere ihtiyacımız olduğunu ve hangilerinin bunun için en uygun olduğunu belirlemeliyiz. Google'da hızlı bir arama bizi “dnspython” adlı modüle götürecektir. Amacımız bir zone transferi gerçekleştirmek ve buna göre DNS sunucuları ile iletişim kurmak için uygun sınıfları ve fonksiyonları bulmamız gerekiyor. Python 3 ile bu tür araçlar geliştirmek için kullanabileceğimiz bir başka eklenti de IPython'dur. Otomatik tamamlamayı destekler ve etkileşimli bir Python shell'inde ilgili class ve modüllerdeki farklı seçenekleri gösterir.

Modülü kurduktan sonra import edebiliriz. Bu modül bize ==Query==, ==Resolver== ve ==Zone== sınıflarını sunuyor. ==dns.zone== modülü bir “==from_xfr==” sınıfı içerir. Bunu Python'daki ==help()== fonksiyonunu kullanarak öğrenebiliriz.

#### Python Help()
```shell-session
M1R4CKCK@htb[/htb]$ ipython

In [1]: import dns.zone as dz

In [2]: help(dz.  #[1x Tab]

                 BadZone       dns           from_text()   generators    NoSOA         PY3           string_types  text_type     Zone
                 BytesIO       from_file()   from_xfr()    NoNS          os          
```

#### Dns.Zone.From_XFR()
```shell-session
In [2]: help(dz.from_xfr) 
```

#### From_XFR() - Documentation
```shell-session
from_xfr(xfr, zone_factory=<class 'dns.zone.Zone'>, relativize=True, check_origin=True)
    Convert the output of a zone transfer generator into a zone object.

    @param xfr: The xfr generator
    @type xfr: generator of dns.message.Message objects
    @param relativize: should names be relativized?  The default is True.
    It is essential that the relativize setting matches the one specified
    to dns.query.xfr().
    @type relativize: bool
    @param check_origin: should sanity checks of the origin node be done?
    The default is True.
    @type check_origin: bool
    @raises dns.zone.NoSOA: No SOA RR was found at the zone origin
    @raises dns.zone.NoNS: No NS RRset was found at the zone origin
    @rtype: dns.zone.Zone object
```

Bunun için gerekli dokümantasyonu from_xfr() fonksiyonu için gerekli parametreleri açıklayan [dokümantasyon sayfasında](https://dnspython.readthedocs.io/en/latest/zone-make.html) da bulabiliriz. “xfr” parametresinden dns.query sınıfına da ihtiyacımız olacak. Bu yüzden daha sonra kullanmak üzere bu sınıfı da not etmeliyiz.


#### Notes
```python
# Notes
import dns.zone as dz
import dns.query as dq

# axfr = dz.from_xfr(dq.xfr())
```

Hangi parametrelerin gerekli olduğunu belirlemek için dns.query.xfr() fonksiyonuna daha yakından bakmamız gerekir.

#### Dns.Query.XFR()
```shell-session
In [3]: help(dq.xfr) 
```

#### XFR() - Documentation
```shell-session
Help on function xfr in module dns.query:

xfr(where, zone, rdtype=252, rdclass=1, timeout=None, port=53, keyring=None, keyname=None, relativize=True, af=None, lifetime=None, source=None, source_port=0, serial=0, use_udp=False, keyalgorithm=<DNS name HMAC-MD5.SIG-ALG.REG.INT.>)
    Return a generator for the responses to a zone transfer.

    *where*.  If the inference attempt fails, AF_INET is used.  This
    parameter is historical; you need never set it.
<SNIP>
```

Burada sadece iki değişkenin değeri olmadığını ve bu nedenle bu fonksiyonu gerçekleştirmek için belirli parametrelere ihtiyaç duyduğunu görebiliriz.
- `where`
- `zone`

Fonksiyonu çalıştırarak gerekli parametreleri de öğrenebiliriz. Çünkü herhangi bir parametre çıktısı vermezsek, hangi parametrelerin gerekli olduğunu söyleyen bir açıklama ile birlikte bir hata oluşacaktır.

#### Required Parameters
```shell-session
In [4]: dq.xfr()

---------------------------------------------------------------------------
TypeError                                 Traceback (most recent call last)
<ipython-input-14-672b143b8cfc> in <module>
----> 1 dq.xfr()

TypeError: xfr() missing 2 required positional arguments: 'where' and 'zone'
```

Bu durumda, “where” değişkeni DNS sunucusunu, “zone” değişkeni ise domain'i temsil eder. Bunu da not ediyoruz.


#### Notes
```python
# Notes
import dns.zone as dz
import dns.query as dq

# axfr = dz.from_xfr(dq.xfr(nameserver, domain))
```

### DNS Requests
Belirli DNS sunucularını kullanarak isteklerimizi nasıl çözümleyeceğimizi bulmalıyız. Gerekli sınıf ile en kolay yol “[dns.resolver](https://dnspython.readthedocs.io/en/latest/resolver-class.html)” olacaktır. Dokümantasyonda, isteklerimizi göndermek istediğimiz DNS sunucularını belirtmemize izin veren “Resolver” sınıfını bulabiliriz. Elbette, bu DNS sunucularını otomatik olarak da bulabiliriz, böylece bunları manuel olarak değiştirmek zorunda kalmayız. Bununla birlikte, şirketler genellikle test etme iznine sahip olmadığımız üçüncü taraf sağlayıcıların DNS sunucularını kullandıkları için dikkatli olmalıyız. Bu nedenle, bunları manuel olarak ve tercihen kodumuz yerine komut satırında belirtmemizi öneririz. Bunu mümkün kılmak için “argparse” adlı başka bir modülü içe aktarıyoruz. Buna göre, bu bilgiyi notlarımıza da ekliyoruz.

#### Notes
```python
# Notes
import dns.zone as dz
import dns.query as dq
import dns.resolver as dr
import argparse

# axfr = dz.from_xfr(dq.xfr(nameserver, domain))

# NS = dr.Resolver()
# NS.nameservers = ['ns1', 'ns2']
```


Şimdi bu domain için “NS” kayıtlarını bulmamız gerekiyor ve bunu dig aracını kullanmak yerine Python modüllerimizle yapıyoruz. Bu örnekte, hala domain adını kullanıyoruz:

- `inlanefreight.com`

İlgili NS sunucularını aşağıdaki kodu kullanarak bulduk:

#### NS Records - DNS.Resolver
```shell-session
M1R4CKCK@htb[/htb]$ python3

>>> import dns.resolver
>>> 
>>> nameservers = dns.resolver.query('inlanefreight.com', 'NS')
>>> for ns in nameservers:
...    	print('NS:', ns)
...
NS: ns1.inlanefreight.com.
NS: ns2.inlanefreight.com.
```

Özetle, şu anda aşağıdaki bilgilere sahibiz:

```python
Domain = 'inlanefreight.com'
DNS Servers = ['ns1.inlanefreight.com', 'ns2.inlanefreight.com']
```

Şimdi tüm bilgilerimizi özetleyebilir ve kodumuzun ilk satırlarını yazabiliriz.


#### DNS-AXFR.py
```python
#!/usr/bin/env python3

# Dependencies:
# python3-dnspython

# Used Modules:
import dns.zone as dz
import dns.query as dq
import dns.resolver as dr
import argparse

# Initialize Resolver-Class from dns.resolver as "NS"
NS = dr.Resolver()

# Target domain
Domain = 'inlanefreight.com'

# Set the nameservers that will be used
NS.nameservers = ['ns1.inlanefreight.com', 'ns2.inlanefreight.com']

# List of found subdomains
Subdomains = []
```


### AXFR Function
Bireysel prosesleri birbirinden ayırmak ve bunun dışında tek bir fonksiyon oluşturmak her zaman avantajlıdır. Bu, daha sonra hata oluştuğunda hata ayıklamayı kolaylaştırır ve ana işlevi daha bağımsız hale getirir. İlk başta bu kadar küçük bir araçla alakasız görünebilir, ancak kodu genişletmeye başlarsak ve sonunda aracımız 1000 satırdan fazla kod boyutuna ulaşırsa, bu avantajı göreceğiz. Süreci aşağıdaki bölümlere ayırabiliriz:

1.Şimdi, verilen domain ve DNS sunucularını kullanarak bir zone transferi gerçekleştirmeye çalışan bir fonksiyon oluşturmak istiyoruz.

2.Zone transferi başarılı olursa, bulunan subdomainlerin doğrudan görüntülenmesini ve listemizde saklanmasını istiyoruz.

3.Bir hata oluşması durumunda, hata hakkında bilgilendirilmek istiyoruz.


### Fonksiyonlar
Fonksiyonlar için mümkün olduğunca az sayıda argüman kullanmaya çalışmalıyız. Bu nedenle üçten fazla argüman olmamalıdır, çünkü aksi takdirde yüksek bir hata olasılığı olabilir. Bir sonraki örnekte, zone transferi için ihtiyacımız olan iki argüman olan domain ve nameserver'ı kullanıyoruz.

Yine, fonksiyonları nasıl tanımlayacağımızı belirlemeli ve bunu standart tutmalıyız. Bu durumda, fonksiyonları tüm harflerini büyük yazarak tanımlıyoruz. Birçok kişi bu büyük harf kullanımını sınıflar için de kullanmaktadır. Daha sonra fonksiyondaki her adımı parçalara ayırmak için fonksiyona yorumlar eklemeliyiz.


#### DNS-AXFR.py - Functions
```python
<SNIP>
# List of found subdomains
Subdomains = []

# Define the AXFR Function
def AXFR(domain, nameserver):

        # Try zone transfer for given domain and namerserver
		# Perform the zone transfer
        # If zone transfer was successful
        # Add found subdomains to global 'Subdomain' list
        # If zone transfer fails

# Main
if __name__=="__main__":

        # For each nameserver
        # Try AXFR
        # Print the results
        # Print each subdomain
```

## **Try-Except**  

Bir statement veya expression sözdizimi açısından doğru yazılmışsa, yürütme sırasında hatalar oluşabilir. Yürütme sırasında ortaya çıkan hatalara `exception` denir ve ciddi olmaları gerekmez. Bir `try` deyimi, farklı istisnalar için farklı eylemler tanımlamak üzere birden fazla `except` bloğu içerebilir. En fazla bir `except` bloğu yürütülür. Bir blok yalnızca ilgili `try` bloğunda meydana gelen exception'ları işleyebilir, ancak aynı `try` deyiminin başka bir except bloğunda meydana gelen exception'ları işleyemez.


#### DNS-AXFR.py - Try-Except
```python
<SNIP>
# List of found subdomains
Subdomains = []

# Define the AXFR Function
def AXFR(domain, nameserver):

        # Try zone transfer for given domain and namerserver
        try:
				# Perform the zone transfer
                axfr = dz.from_xfr(dq.xfr(nameserver, domain))
				
                # If zone transfer was successful
                # Add found subdomains to global 'Subdomain' list
				
        # If zone transfer fails
        except Exception as error:
                print(error)
                pass
```

## **If-Else**  

`if-else` deyimlerinde, aynı anda kaç argüman veya değeri kontrol etmek istediğimize bağlıdır. En iyi durumda, bir seferde kontrol edilecek yalnızca bir değer veya argüman olmalıdır. Ancak, birden fazla argümanı kontrol etmemiz gerekiyorsa ve satır çok uzun olabiliyorsa, parantez içindeki her argümanı yeni bir satırda yazmanız önerilir.

#### If-Else - Few Arguments
```python
# Few arguments
if (arg1 and arg2):
	# Perform specified actions
else:
	pass
```

#### If-Else - Many Arguments
```python
# Many arguments
if (arg1 
	and arg2
	and arg3
	and arg4):
	
	# Perform specified actions
else:
	pass
```

`DNS.py` scriptimizde yalnızca bir test değeri kullanıyoruz ve bu nedenle birden fazla satıra ihtiyacımız yok.

#### DNS-AXFR.py - If-Else
```python
<SNIP>
# List of found subdomains
Subdomains = []

# Define the AXFR Function
def AXFR(domain, nameserver):

        # Try zone transfer for given domain and namerserver
        try:
				# Perform the zone transfer
                axfr = dz.from_xfr(dq.xfr(nameserver, domain))

                # If zone transfer was successful
                if axfr:
                        print('[*] Successful Zone Transfer from {}'.format(nameserver))

                        # Add found subdomains to global 'Subdomain' list

        # If zone transfer fails
        except Exception as error:
                print(error)
                pass
```


## **For-Loop**  

`For` döngüleri her zaman mümkün olduğunca basit tutulmalıdır, çünkü geçiş sayısında hatalara neden olabilirler ve bu durumda döngüde neyin yanlış gittiğini anlamak için her giriş için manuel olarak hata ayıklamamız gerekir. Bu nedenle, şimdi bulduğumuz subdomain'leri saklamak için her bir "`record`"u önceden tanımlanmış "`Subdomains`" listemize ekleyeceğiz.

#### DNS-AXFR.py
```python
<SNIP>
# List of found subdomains
Subdomains = []

# Define the AXFR Function
def AXFR(domain, nameserver):

        # Try zone transfer for given domain and namerserver
        try:
				# Perform the zone transfer
                axfr = dz.from_xfr(dq.xfr(nameserver, domain))

                # If zone transfer was successful
                if axfr:
                        print('[*] Successful Zone Transfer from {}'.format(nameserver))

                        # Add found subdomains to global 'Subdomain' list
                        for record in axfr:
                                Subdomains.append('{}.{}'.format(record.to_text(), domain))

        # If zone transfer fails
        except Exception as error:
                print(error)
                pass
```

Artık sadece domain ve ilgili name server'a argüman olarak ihtiyaç duyan fonksiyonumuz var. Daha sonra, argümanları `AXFR fonksiyonuna` geçiren, onu çalıştıran ve bize sonuçları gösteren `Main fonksiyonuna` ihtiyacımız var.


# **Main Function**  

Main fonksiyonu, kodun main programının çalıştığı kısmıdır. Aracımıza daha sonra büyük olasılıkla daha fazla fonksiyon ekleyeceğimiz için bunu mümkün olduğunca açık tutmak çok önemlidir. Bu durumda, aracımızın belirlediğimiz her DNS sunucusunda bir zone transferi denemesini istiyoruz. Bulunan subdomainlerin `AXFR()` fonksiyonundaki global subdomain listesine ekleneceğini biliyoruz. Dolayısıyla, bu liste boş değilse, tüm subdomainleri görmek istiyoruz.


## **Main**

Bir Python programını bir py dosyasını çağırarak başlatırsak ve orada `“__name__”` ifadesini çağırırsak, sistem bunu sistem değişkenine `“__main__`” tanımlayıcısını atar. Ancak, bu dosyayı başka bir modüle aktarırsak, dosya adına karşılık gelen bir tanımlayıcı alır. Bu yapı, bir Python dosyasını bağımsız bir program olarak kullanır, bu dosyanın tek tek öğelerini içe aktarılabilir hale getirir ve karmaşık modül testleri uygular. `AXFR` fonksiyonunda yaptığımız gibi, buna göre ilerliyoruz ve atılması gereken kaba adımları yorum olarak tanımlıyoruz.


#### DNS-AXFR.py
```python
<SNIP>
# Main
if __name__=="__main__":

        # For each nameserver
        # Try AXFR
        # Print the results
        # Print each subdomain
```


## For-Loop - Try AXFR
`AXFR` fonksiyonumuz “domain” ve “nameserver” olmak üzere iki parametre gerektirdiğinden, belirtilen her nameserver için `AXFR` fonksiyonunu çalıştıran bir `For döngüsü` oluşturuyoruz. Bulunan subdomain'lerin çıktısını alacak alanda `If-Else` deyimiyle birleştirilmiş başka bir `For` döngüsü kullanıyoruz. AXFR fonksiyonunun bulunan subdomainleri depoladığı subdomain listesi boş değilse, subdomain listesindeki her subdomain girişi yazdırılmalıdır.

#### DNS-AXFR.py
```python
<SNIP>
# Main
if __name__=="__main__":

        # For each nameserver
        for nameserver in NS.nameservers:

                #Try AXFR
                AXFR(Domain, nameserver)

        # Print the results
        if Subdomains is not None:
                print('-------- Found Subdomains:')

                # Print each subdomain
                for subdomain in Subdomains:
                        print('{}'.format(subdomain))

        else:
                print('No subdomains found.')
                exit()
```

## DNS-AXFR.py - Complete Code
```python
#!/usr/bin/env python3

# Dependencies:
# python3-dnspython

# Used Modules:
import dns.zone as dz
import dns.query as dq
import dns.resolver as dr
import argparse

# Initialize Resolver-Class from dns.resolver as "NS"
NS = dr.Resolver()

# Target domain
Domain = 'inlanefreight.com'

# Set the nameservers that will be used
NS.nameservers = ['ns1.inlanefreight.com', 'ns2.inlanefreight.com']

# List of found subdomains
Subdomains = []

# Define the AXFR Function
def AXFR(domain, nameserver):

        # Try zone transfer for given domain and namerserver
        try:
				# Perform the zone transfer
                axfr = dz.from_xfr(dq.xfr(nameserver, domain))

                # If zone transfer was successful
                if axfr:
                        print('[*] Successful Zone Transfer from {}'.format(nameserver))

                        # Add found subdomains to global 'Subdomain' list
                        for record in axfr:
                                Subdomains.append('{}.{}'.format(record.to_text(), domain))

        # If zone transfer fails
        except Exception as error:
                print(error)
                pass

# Main
if __name__=="__main__":

        # For each nameserver
        for nameserver in NS.nameservers:

                #Try AXFR
                AXFR(Domain, nameserver)

        # Print the results
        if Subdomains is not None:
                print('-------- Found Subdomains:')

                # Print each subdomain
                for subdomain in Subdomains:
                        print('{}'.format(subdomain))

        else:
                print('No subdomains found.')
                exit()                
```


## **Sonuçlar**  

#### **Komut Dosyasını Çalıştırılabilir Yapın ve Çalıştırın**
```shell-session
M1R4CKCK@htb[/htb]$ chmod +x dns-axfr.py
M1R4CKCK@htb[/htb]$ ./dns-axfr.py

[*] Successful Zone Transfer from ns1.inlanefreight.com
-------- Found Subdomains:
@.inlanefreight.com
admin.inlanefreight.com
app.inlanefreight.com
blog.inlanefreight.com
<SNIP>
```


Soru : "inlanefreight.htb" domain'i için hedefinize karşı DNS-AXFR.py script'ini kullanarak bir zone transferi gerçekleştirin ve bulunan toplam benzersiz subdomain sayısını gönderin.

Cevap : 