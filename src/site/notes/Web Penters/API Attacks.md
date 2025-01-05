---
{"dg-publish":true,"permalink":"/web-penters/api-attacks/"}
---

API'ler, modern yazılım geliştirmede kritik bir rol oynar ve özellikle web API'leri, farklı sistemler arasında iletişim ve veri alışverişi sağlar. Tanımlı kurallar ve protokollerle veri biçimlendirme, kaynak erişimi ve yanıt yapıları belirlenir. API'ler, dış erişime açık **public** ya da belirli gruplara özel **private** olarak sınıflandırılır.


## API Building Styles

Web API'ler, REST, SOAP, GraphQL ve gRPC gibi çeşitli mimari stillerle oluşturulabilir. Her birinin kendine özgü avantajları ve kullanım alanları vardır:

- **Representational State Transfer (REST)**: En yaygın kullanılan API tarzıdır. Client-server modeli ile çalışır ve standart HTTP yöntemleri (GET, POST, PUT, DELETE) kullanılarak sunucudaki kaynaklara istek gönderir. RESTful API'ler stateless'tir, yani her istek, sunucunun işleyebilmesi için gerekli tüm bilgileri içerir. Yanıtlar genellikle JSON veya XML olarak serileştirilir.
- **Simple Object Access Protocol (SOAP)**: Sistemler arasında XML tabanlı mesajlaşmayı kullanır. SOAP API'leri, güvenlik, işlemler ve hata yönetimi için kapsamlı özellikler sunar ancak RESTful API'lere kıyasla daha karmaşıktır.
- **GraphQL**: Daha esnek ve verimli veri alımı ve güncellemesi sağlayan bir alternatif stildir. Sabit bir veri seti döndürmek yerine, GraphQL istemcilerin tam olarak hangi veriye ihtiyaç duyduğunu belirtmesine olanak tanır. Bu, fazla veya eksik veri alımını önler. Tek bir endpoint kullanır ve güçlü bir şekilde tiplenmiş bir sorgu dili ile veri alınır.
- **gRPC**: Protobuf (Protocol Buffers) kullanarak mesaj serileştirme sağlar ve sistemler arasında yüksek performanslı, verimli iletişim için kullanılır. gRPC API'leri birçok programlama diliyle geliştirilebilir ve özellikle mikro hizmetler ve dağıtık sistemler için uygundur.

Bu modülde, bir RESTful web API'ye yönelik saldırılara odaklanılacaktır. Ancak, gösterilen güvenlik açıkları diğer mimari stillerle oluşturulmuş API'lerde de bulunabilir.


# API Attacks

Farklı sistemler arasında veri alışverişini ve iletişimi kolaylaştıran API'lerin doğası, Hassas Verilerin Açığa Çıkması, Kimlik Doğrulama ve Yetkilendirme Sorunları, Yetersiz Hız Sınırlaması, Uygun Olmayan Hata İşleme ve diğer çeşitli güvenlik yanlış yapılandırmaları gibi güvenlik açıklarını ortaya çıkarır.

## OWASP Top 10 API Security Risks

API'lerin karşılaşabileceği güvenlik açıklarını ve yanlış yapılandırmaları kategorize etmek ve standartlaştırmak için OWASP, özellikle API'lerle ilgili en kritik güvenlik risklerinin kapsamlı bir listesi olan [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)'u hazırladı:

| Risk                                                                                                                                                                     | Açıklama                                                                                                                                                    |
| ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [**API1:2023 - Broken Object Level Authorization**](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)                             | API, kimlik doğrulaması yapılmış kullanıcıların yetkilendirilmediği verilere erişmesine izin verir.                                                         |
| [**API2:2023 - Broken Authentication**](https://owasp.org/API-Security/editions/2023/en/0xa2-broken-authentication/)                                                     | API'nin kimlik doğrulama mekanizmaları atlatılabilir veya geçersiz kılınabilir, bu da yetkisiz erişime yol açar.                                            |
| [**API3:2023 - Broken Object Property Level Authorization**](https://owasp.org/API-Security/editions/2023/en/0xa3-broken-object-property-level-authorization/)           | API, yetkilendirilmiş kullanıcılara erişmemeleri gereken hassas verileri gösterir veya bu kullanıcıların hassas özellikleri manipüle etmelerine izin verir. |
| [**API4:2023 - Unrestricted Resource Consumption**](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)                             | API, kullanıcıların tüketeceği kaynakları sınırlamaz.                                                                                                       |
| [**API5:2023 - Broken Function Level Authorization**](https://owasp.org/API-Security/editions/2023/en/0xa5-broken-function-level-authorization/)                         | API, yetkisiz kullanıcıların yetkilendirilmiş işlemleri gerçekleştirmesine izin verir.                                                                      |
| [**API6:2023 - Unrestricted Access to Sensitive Business Flows**](https://owasp.org/API-Security/editions/2023/en/0xa6-unrestricted-access-to-sensitive-business-flows/) | API, hassas iş akışlarını açığa çıkarır, bu da potansiyel finansal kayıplara ve diğer zararlara yol açabilir.                                               |
| [**API7:2023 - Server Side Request Forgery**](https://owasp.org/API-Security/editions/2023/en/0xa7-server-side-request-forgery/)                                         | API, istekleri yeterince doğrulamaz, bu da saldırganların kötü niyetli istekler göndermesine ve dahili kaynaklarla etkileşime girmesine olanak tanır.       |
| [**API8:2023 - Security Misconfiguration**](https://owasp.org/API-Security/editions/2023/en/0xa8-security-misconfiguration/)                                             | API, yanlış yapılandırmalar nedeniyle güvenlik açıklarına sahiptir, bu da Enjeksiyon Saldırıları gibi zafiyetlere yol açabilir.                             |
| [**API9:2023 - Improper Inventory Management**](https://owasp.org/API-Security/editions/2023/en/0xa9-improper-inventory-management/)                                     | API, sürüm envanterini düzgün ve güvenli bir şekilde yönetmez.                                                                                              |
| [**API10:2023 - Unsafe Consumption of APIs**](https://owasp.org/API-Security/editions/2023/en/0xaa-unsafe-consumption-of-apis/)                                          | API, başka bir API'yi güvensiz bir şekilde tüketir, bu da potansiyel güvenlik risklerine yol açar.                                                          |

Bu modül, tüm bu güvenlik risklerinden yararlanmaya ve bunların nasıl önleneceğini anlamaya odaklanacaktır.


# Introduction to Lab
Modül boyunca ilerledikçe, bu güvenlik açıklarını tam olarak anlamak için RESTful web API kullanarak OWASP API Top 10 Güvenlik Risklerinin her birini tanımlama ve kullanma pratiği yapacağız.

## Inlanefreight E-Commerce Marketplace

- **Inlanefreight**, **Inlanefreight E-Commerce Marketplace** ile e-ticaret dünyasına adım atmıştır.
- Platform, müşterilerin tedarikçi ürünlerini inceleyip satın almasını sağlar.
- Pazar yeri, her satın alınan ürün için ücret alarak gelir elde eder.
- **Inlanefreight**, çok kiracılı bir **web API** geliştirmiştir ve **Rol Tabanlı Erişim Kontrolü (RBAC)** kullanmaktadır.
- Kullanıcıların kimlik bilgileri:
    - **pentestercompany.com**: Tedarikçi hesapları
    - **hackthebox.com**: Müşteri hesapları
- Kullanıcıların rolleri yöneticiler tarafından atanır ve roller, ilgili endpointlerle aynı ada sahiptir.
    - Örneğin, **Suppliers_GetAll** rolü, kullanıcının `/api/v1/suppliers` endpointine erişim yetkisine sahip olduğunu gösterir.

Amacımız, bulunan tüm güvenlik açıklarını Inlanefreight E-Ticaret Pazaryeri yöneticisine bildirmektir. Keşfedilen tüm güvenlik açıklarının ayrıntılı bir raporu, API'nin güvenliğini sağlamak için uygun önlemleri almada yöneticiye yardımcı olacaktır. Her güvenlik açığı, ilgili [CWE](https://cwe.mitre.org/data/index.html) zayıflığıyla eşleştirilecektir.


### Swagger API User Interface

Inlanefreight E-Ticaret Pazaryeri'nin frontend'i hala aktif geliştirme aşamasında olmasına rağmen, web API'sine /swagger yolundaki bir [Swagger](https://swagger.io/tools/swagger-ui/) kullanıcı arayüzü aracılığıyla erişilebilir (bunu, oluşturulan hedef makinenin portundan sonra eklediğinizden emin olun). Modül boyunca bu arayüzü, 60'tan fazla endpoint içeren pazaryeri API'sinin güvenliğini keşfetmek ve değerlendirmek için kullanacağız:

![Pasted image 20241222212116.png](/img/user/resimler/Pasted%20image%2020241222212116.png)

Pazar yerinin kapsadığı temel varlıklar arasında Customers, Product, Supplier-Companies ve Suppliers yer almaktadır. Bölümlerde ilerledikçe diğer varlıklarla da etkileşime geçeceğiz.


Soru :  Herhangi bir endpoint ile etkileşime geçin ve response header'ları inceleyin; web API'sinin kullandığı sunucunun adı nedir?

Cevap : 

1- ![Pasted image 20241222212525.png](/img/user/resimler/Pasted%20image%2020241222212525.png)


Soru : Roles grubuna ait yalnızca bir endpoint vardır. Yolunu gönderin

Cevap : 

![Pasted image 20241222213133.png](/img/user/resimler/Pasted%20image%2020241222213133.png)


# Broken Object Level Authorization

Web API'leri, kullanıcıların Globally Unique Identifiers (GUIDs) olarak da bilinen Universally Unique Identifiers (UUIDs) gibi benzersiz tanımlayıcılar ve tamsayı kimlikleri de dahil olmak üzere çeşitli parametreler göndererek veri veya kayıt talep etmelerine olanak tanır. Ancak, bir kullanıcının nesne düzeyinde yetkilendirme mekanizmaları aracılığıyla belirli bir kaynağı görüntülemek için sahipliğe ve izne sahip olduğunu düzgün ve güvenli bir şekilde doğrulamamak, verilerin açığa çıkmasına ve güvenlik açıklarına neden olabilir.

Bir web API endpoint'i, (kaynak kod seviyesinde uygulanan) yetkilendirme kontrolleri, kimliği doğrulanmış bir kullanıcının belirli verileri talep etmek ve görüntülemek veya belirli işlemleri gerçekleştirmek için yeterli izinlere veya ayrıcalıklara sahip olmasını doğru bir şekilde sağlayamazsa, Secure Direct Object Reference (IDOR) olarak da bilinen Broken Object Level Authorization'a (BOLA) karşı savunmasızdır.

### Kullanıcı Kontrollü Anahtar Aracılığıyla Yetkilendirme Bypass'ı

Alıştırma yapacağımız endpoint [CWE-639: Kullanıcı Kontrollü Anahtar Üzerinden Yetkilendirme Bypass'ına karşı savunmasızdır.](https://cwe.mitre.org/data/definitions/639.html)

### Scenario

Inlanefreight E-Ticaret Pazaryeri'nin yöneticisi bize htbpentester1@pentestercompany.com:HTBPentester1 kimlik bilgilerini sağladı ve kullanıcının atanan rolleriyle hangi API güvenlik açıklarından yararlanabileceğini değerlendirmemizi istedi.

Hesap bir Tedarikçiye ait olduğundan, oturum açmak ve bir JWT elde etmek için /api/v1/authentication/suppliers/sign-in endpoint'ini kullanacağız:

![Authenitcation_Suppliers.gif](/img/user/resimler/Authenitcation_Suppliers.gif)

JWT'yi kullanarak kimlik doğrulaması yapmak için yanıtı kopyalayacağız ve Authorize düğmesine tıklayacağız. Şu anda kilidi açık olan ve kimliğimizin doğrulanmadığını gösteren kilit simgesine dikkat edin. Ardından, JWT'yi Kullanılabilir yetkilendirmeler açılır penceresindeki Değer text alanına yapıştıracağız ve Authorize (Yetkilendir) butonuna tıklayacağız. Tamamlandığında, kilit simgesi tamamen kilitlenecek ve kimlik doğrulamamızı onaylayacaktır:

![Authentication_Suppliers_2.gif](/img/user/resimler/Authentication_Suppliers_2.gif)

Suppliers grubu içindeki endpoint'leri incelerken (en sağlarında kimlik doğrulamanın gerekli olduğunu gösteren bir kilit olduğuna dikkat edin), /api/v1/suppliers/current-user adında bir tane fark edeceğiz:

![Pasted image 20241222214340.png](/img/user/resimler/Pasted%20image%2020241222214340.png)

Yollarında current-user içeren endpoint'ler, belirtilen işlemi gerçekleştirmek için o anda kimliği doğrulanmış kullanıcının JWT'sini kullandıklarını gösterir; bu durumda bu işlem mevcut kullanıcının verilerini almaktır. Endpoint'i çağırdıktan sonra, mevcut kullanıcımızın şirket ID'si olan b75a7c76-e149-4ca7-9c55-d9fc4ffa87be'yi bir Guid değeri olarak alacağız:

![Pasted image 20241222214411.png](/img/user/resimler/Pasted%20image%2020241222214411.png)

Daha sonra mevcut kullanıcımızın rollerini alalım. api/v1/roles/current-user endpoint'ini çağırdıktan sonra, SupplierCompanies_GetYearlyReportByID rolü ile yanıt verir:

![Pasted image 20241222214438.png](/img/user/resimler/Pasted%20image%2020241222214438.png)

Supplier-Companies grubunda, bir GET parametresi kabul eden SupplierCompanies_GetYearlyReportByID rolüyle ilgili bir endpoint buluyoruz: /api/v1/supplier-companies/yearly-reports/{ID}:

![Pasted image 20241222214530.png](/img/user/resimler/Pasted%20image%2020241222214530.png)

Bunu genişletirken, SupplierCompanies_GetYearlyReportByID rolünü gerektirdiğini ve ID parametresini bir Guid olarak değil bir tamsayı olarak kabul ettiğini fark edeceğiz:

![Pasted image 20241222214628.png](/img/user/resimler/Pasted%20image%2020241222214628.png)

Kimlik olarak 1 kullanırsak, f9e58492-b594-4d82-a4de-16e4f230fce1 kimliğine sahip bir şirkete ait yıllık bir rapor alırız, ki bu bizim ait olduğumuz b75a7c76-e149-4ca7-9c55-d9fc4ffa87be değildir:

![Pasted image 20241222214655.png](/img/user/resimler/Pasted%20image%2020241222214655.png)

Diğer kimlikleri denerken, diğer tedarikçi şirketlerin yıllık raporlarına erişebiliyoruz ve bu da potansiyel olarak hassas iş verilerine erişmemizi sağlıyor:

![Pasted image 20241222214712.png](/img/user/resimler/Pasted%20image%2020241222214712.png)

Ayrıca, BOLA güvenlik açığını toplu olarak kötüye kullanabilir ve tedarikçi şirketlerin ilk 20 yıllık raporlarını getirebiliriz:

![BOLA_Mass_Abuse.gif](/img/user/resimler/BOLA_Mass_Abuse.gif)


Swagger arayüzünden kopyalanan cURL komutunda yapmamız gereken tek değişiklik, değişken enterpolasyonlu bir Bash for-loop kullanmak, -w “\n” bayrağını kullanarak her yanıttan sonra yeni bir satır eklemek, -s bayrağını kullanarak ilerlemeyi durdurmak ve çıktıyı jq'ya aktarmaktır:

```shell-session
M1R4CKCK@htb[/htb]$ for ((i=1; i<= 20; i++)); do
curl -s -w "\n" -X 'GET' \
  'http://94.237.49.212:43104/api/v1/supplier-companies/yearly-reports/'$i'' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjFAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOiJTdXBwbGllckNvbXBhbmllc19HZXRZZWFybHlSZXBvcnRCeUlEIiwiZXhwIjoxNzIwMTg1NzAwLCJpc3MiOiJodHRwOi8vYXBpLmlubGFuZWZyZWlnaHQuaHRiIiwiYXVkIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiJ9.D6E5gJ-HzeLZLSXeIC4v5iynZetx7f-bpWu8iE_pUODlpoWdYKniY9agU2qRYyf6tAGdTcyqLFKt1tOhpOsWlw' | jq
done

{
  "supplierCompanyYearlyReport": {
    "id": 1,
    "companyID": "f9e58492-b594-4d82-a4de-16e4f230fce1",
    "year": 2020,
    "revenue": 794425112,
    "commentsFromCLevel": "Superb work! The Board is over the moon! All employees will enjoy a dream vacation!"
  }
}
{
  "supplierCompanyYearlyReport": {
    "id": 2,
    "companyID": "f9e58492-b594-4d82-a4de-16e4f230fce1",
    "year": 2022,
    "revenue": 339322952,
    "commentsFromCLevel": "Excellent performance! The Board is exhilarated! Prepare for a special vacation adventure!"
  }
}
{
  "supplierCompanyYearlyReport": {
    "id": 3,
    "companyID": "058ac1e5-3807-47f3-b546-cc069366f8f9",
    "year": 2020,
    "revenue": 186208503,
    "commentsFromCLevel": "Phenomenal performance! The Board is deeply impressed! Everyone will be treated to a deluxe vacation!"
  }
}

<SNIP>
```


### Önleme

BOLA güvenlik açığını azaltmak için /api/v1/supplier-companies/yearly-reports endpoint'i, yetkili kullanıcıların yalnızca bağlı oldukları şirketle ilişkili yıllık raporlara erişebilmesini sağlamak için bir doğrulama adımı (kaynak kodu düzeyinde) uygulamalıdır. Bu doğrulama, raporun companyID alanının kimliği doğrulanmış tedarikçinin companyID'si ile karşılaştırılmasını içerir. Erişim yalnızca bu değerler eşleşirse verilmelidir; aksi takdirde talep reddedilmelidir. Bu yaklaşım, tedarikçi-şirketlerin yıllık raporları arasındaki veri ayrımını etkin bir şekilde korur.

Soru: Başka bir Broken Object Level Authorization güvenlik açığından yararlanın ve bayrağı gönderin.

Cevap : 

1- Öncelikle verilen kimlik bilgileriyle jwt token alalım .

![Pasted image 20241222215801.png](/img/user/resimler/Pasted%20image%2020241222215801.png)

![Pasted image 20241222220135.png](/img/user/resimler/Pasted%20image%2020241222220135.png)

![Pasted image 20241222223400.png](/img/user/resimler/Pasted%20image%2020241222223400.png)

![Pasted image 20241222223423.png](/img/user/resimler/Pasted%20image%2020241222223423.png)

![Pasted image 20241222224942.png](/img/user/resimler/Pasted%20image%2020241222224942.png)

![Pasted image 20241222224954.png](/img/user/resimler/Pasted%20image%2020241222224954.png)

![Pasted image 20241222225014.png](/img/user/resimler/Pasted%20image%2020241222225014.png)

![Pasted image 20241222225142.png](/img/user/resimler/Pasted%20image%2020241222225142.png)

Execute Edelim 

![Pasted image 20241222225728.png](/img/user/resimler/Pasted%20image%2020241222225728.png)

![Pasted image 20241222230850.png](/img/user/resimler/Pasted%20image%2020241222230850.png)

![Pasted image 20241222231109.png](/img/user/resimler/Pasted%20image%2020241222231109.png)

Daha sonra mevcut kullanıcımızın rollerini alalım. api/v1/roles/current-user endpoint'ini çağırdıktan sonra, SupplierCompanies_GetYearlyReportByID rolü ile yanıt verir:

Get quarterlyReportBy ID
![Pasted image 20241222231235.png](/img/user/resimler/Pasted%20image%2020241222231235.png)

Supplier-Companies grubunda, bir GET parametresi kabul eden SupplierCompanies_GetYearlyReportByID rolüyle ilgili bir endpoint buluyoruz: /api/v1/supplier-companies/yearly-reports/{ID}:

![Pasted image 20241222231447.png](/img/user/resimler/Pasted%20image%2020241222231447.png)

Bunu genişletirken, SupplierCompanies_GetYearlyReportByID rolünü gerektirdiğini ve ID parametresini bir Guid olarak değil bir tamsayı olarak kabul ettiğini fark edeceğiz:

![Pasted image 20241222233300.png](/img/user/resimler/Pasted%20image%2020241222233300.png)

Kimlik olarak 1 kullanırsak, f9e58492-b594-4d82-a4de-16e4f230fce1 kimliğine sahip bir şirkete ait yıllık bir rapor alırız, ki bu bizim ait olduğumuz b75a7c76-e149-4ca7-9c55-d9fc4ffa87be değildir:

![Pasted image 20241222233243.png](/img/user/resimler/Pasted%20image%2020241222233243.png)

Diğer kimlikleri denerken, diğer tedarikçi şirketlerin yıllık raporlarına erişebiliyoruz ve bu da potansiyel olarak hassas iş verilerine erişmemizi sağlıyor:

Ayrıca, BOLA güvenlik açığını toplu olarak kötüye kullanabilir ve tedarikçi şirketlerin ilk 20 yıllık raporlarını getirebiliriz:

![Pasted image 20241222231235.png](/img/user/resimler/Pasted%20image%2020241222231235.png)

Ama soru çözümü anlatılanlardan farklı olarak GetQuarterlyReportByID ye odaklanacağız geri kalan zaten her şey aynı . 

![Pasted image 20241222234117.png](/img/user/resimler/Pasted%20image%2020241222234117.png)

![Pasted image 20241222234133.png](/img/user/resimler/Pasted%20image%2020241222234133.png)

Curl komutunu alalım .

![Pasted image 20241222234212.png](/img/user/resimler/Pasted%20image%2020241222234212.png)

```
for ((i=1; i<=20; i++)); do
curl -s -w "\n" -X 'GET' \
  'http://83.136.254.158:47230/api/v1/suppliers/quarterly-reports/'$i'' \
  -H 'accept: application/json' \
  -H 'Authorization: Bearer eyJhbGciOiJIUzUxMiIsInR5cCI6IkpXVCJ9.eyJodHRwOi8vc2NoZW1hcy54bWxzb2FwLm9yZy93cy8yMDA1LzA1L2lkZW50aXR5L2NsYWltcy9uYW1laWRlbnRpZmllciI6Imh0YnBlbnRlc3RlcjJAcGVudGVzdGVyY29tcGFueS5jb20iLCJodHRwOi8vc2NoZW1hcy5taWNyb3NvZnQuY29tL3dzLzIwMDgvMDYvaWRlbnRpdHkvY2xhaW1zL3JvbGUiOlsiU3VwcGxpZXJDb21wYW5pZXNfR2V0WWVhcmx5UmVwb3J0QnlJRCIsIlN1cHBsaWVyc19HZXRRdWFydGVybHlSZXBvcnRCeUlEIl0sImV4cCI6MTczNDkwMDU2OSwiaXNzIjoiaHR0cDovL2FwaS5pbmxhbmVmcmVpZ2h0Lmh0YiIsImF1ZCI6Imh0dHA6Ly9hcGkuaW5sYW5lZnJlaWdodC5odGIifQ.qYTdWZ8ODIsWs8q33KlUuc_KTsfi_1vLjTBj4stbo5VTP43AmqbTtsudlh8nl358DLgDVEoc97mWi1uuHEYwoA' | jq
done
```

![Pasted image 20241222234433.png](/img/user/resimler/Pasted%20image%2020241222234433.png)


# Broken Authentication

[Authentication](https://cheatsheetseries.owasp.org/cheatsheets/Authentication_Cheat_Sheet.html), web API güvenliğinin temel bir ayağıdır. Web API'leri veri gizliliğini sağlamak için çeşitli kimlik doğrulama mekanizmaları kullanır. Bir API, kimlik doğrulama mekanizmalarından herhangi biri atlanabilir veya atlatılabilirse Broken Authentication'dan muzdarip olur.


## Improper Restriction of Excessive Authentication Attempts

Alıştırma yapacağımız endpoint [CWE-307: Improper Restriction of Excessive Authentication Attempts'e](https://cwe.mitre.org/data/definitions/307.html) karşı savunmasızdır.

### Scenario

Inlanefreight E-Ticaret Pazaryeri'nin yöneticisi bize htbpentester3@hackthebox.com:HTBPentester3 kimlik bilgilerini sağladı ve kullanıcının atanan rolleriyle hangi API güvenlik açıklarından yararlanabileceğini değerlendirmemizi istedi.

Hesap bir müşteriye ait olduğundan, bir JWT elde etmek ve ardından bununla kimlik doğrulaması yapmak için /api/v1/authentication/customers/sign-in endpoint'ini kullanacağız:

![Pasted image 20241222235428.png](/img/user/resimler/Pasted%20image%2020241222235428.png)

api/v1/customers/current-user endpoint'ini çağırırken, o anda kimliği doğrulanmış kullanıcımızın bilgilerini geri alırız:

![Pasted image 20241222235553.png](/img/user/resimler/Pasted%20image%2020241222235553.png)

api/v1/roles/current-user endpoint, kullanıcıya üç rol atandığını ortaya koymaktadır: Customers_UpdateByCurrentUser, Customers_Get ve Customers_GetAll:

![Pasted image 20241222235713.png](/img/user/resimler/Pasted%20image%2020241222235713.png)

Customers_GetAll, tüm müşterilerin kayıtlarını döndüren /api/v1/customers endpoint'ini kullanmamızı sağlar:

![Pasted image 20241222235744.png](/img/user/resimler/Pasted%20image%2020241222235744.png)

Endpoint, e-posta, phoneNumber ve birthDate gibi diğer müşterilerle ilgili hassas bilgileri açığa çıkardığı için Broken Object Property Level Authorization (bir sonraki bölümde ele alacağız) sorununu yaşasa da, başka bir hesabı ele geçirmemize doğrudan izin vermez.

api/v1/customers/current-user PATCH endpoint'ini genişlettiğimizde, hesabın şifresi de dahil olmak üzere bilgi alanlarımızı güncellememize izin verdiğini keşfederiz:

![Pasted image 20241222235821.png](/img/user/resimler/Pasted%20image%2020241222235821.png)

Eğer 'pass' gibi zayıf bir parola girersek API, parolaların en az altı karakter uzunluğunda olması gerektiğini belirterek güncellemeyi reddeder:

![Pasted image 20241222235847.png](/img/user/resimler/Pasted%20image%2020241222235847.png)

Doğrulama mesajı, API'nin kriptografik olarak güvenli şifreleri zorunlu kılmayan zayıf bir şifre politikası kullandığını ortaya çıkararak değerli bilgiler sağlar. Parolayı '123456' olarak ayarlamayı denersek, API'nin artık başarı durumu için true değerini döndürdüğünü ve güncellemeyi gerçekleştirdiğini fark ederiz:

![Pasted image 20241222235914.png](/img/user/resimler/Pasted%20image%2020241222235914.png)

API'nin zayıf bir parola politikası kullandığı göz önüne alındığında, diğer müşteri hesapları kayıt olurken kriptografik olarak güvenli olmayan parolalar kullanmış olabilir. Bu nedenle, ffuf kullanarak müşterilere karşı brute-forcing  zorlaması gerçekleştireceğiz.

İlk olarak, /api/v1/authentication/customers/sign-in endpoint'inin yanlış kimlik bilgileri sağlandığında döndürdüğü (başarısız) mesajı almamız gerekiyor, bu durumda bu mesaj 'Geçersiz Kimlik Bilgileri'dir:

![Pasted image 20241223000006.png](/img/user/resimler/Pasted%20image%2020241223000006.png)

Inlanefreight E-Ticaret Pazaryeri yöneticisi, 107 müşterinin tamamına saldırmak yerine, bize üç yüksek değerli hedefin e-postalarını sağladı (bunları bir dosyaya kaydetmemiz gerekiyor):

* OlawaleJones@yandex.com
* IsabellaRichardson@gmail.com
* WenSalazar@zoho.com

Parola kelime listesi için SecLists'ten [xato-net-10-million-passwords-10000](https://github.com/danielmiessler/SecLists/blob/master/Passwords/xato-net-10-million-passwords-10000.txt) kullanacağız.

Aynı anda iki parametreyi fuzzing yaptığımız için (bunlar e-posta ve paroladır), ffuf'un -w bayrağını kullanmamız ve müşteri e-postaları ve parolaları kelime listelerine sırasıyla EMAIL ve PASS anahtar kelimelerini atamamız gerekir. ffuf tamamlandığında, IsabellaRichardson@gmail.com şifresinin qwerasdfzxcv olduğunu keşfedeceğiz:

```shell-session
[!bash!]$ ffuf -w /opt/useful/seclists/Passwords/xato-net-10-million-passwords-10000.txt:PASS -w customerEmails.txt:EMAIL -u http://94.237.59.63:31874/api/v1/authentication/customers/sign-in -X POST -H "Content-Type: application/json" -d '{"Email": "EMAIL", "Password": "PASS"}' -fr "Invalid Credentials" -t 100

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://94.237.59.63:31874/api/v1/authentication/customers/sign-in
 :: Wordlist         : PASS: /opt/useful/seclists/Passwords/xato-net-10-million-passwords-10000.txt
 :: Wordlist         : EMAIL: /home/htb-ac-413848/customerEmails.txt
 :: Header           : Content-Type: application/json
 :: Data             : {"Email": "EMAIL", "Password": "PASS"}
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 100
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
 :: Filter           : Regexp: Invalid Credentials
________________________________________________

[Status: 200, Size: 393, Words: 1, Lines: 1, Duration: 81ms]
    * EMAIL: IsabellaRichardson@gmail.com
    * PASS: qwerasdfzxcv

:: Progress: [30000/30000] :: Job [1/1] :: 1275 req/sec :: Duration: [0:00:24] :: Errors: 0 ::
```

Artık şifreyi brute-forced yaptığımıza göre, IsabellaRichardson@gmail.com:qwerasdfzxcv kimlik bilgileriyle /api/v1/authentication/customers/sign-in endpoint'ini kullanarak Isabella olarak bir JWT elde edebilir ve Inlanefreight E-Commerce Marketplace'teki tüm gizli bilgilerini görüntüleyebiliriz.


### OTP'leri ve Güvenlik Sorularının Cevaplarını Brute-Force ile Deneme  
Uygulamalar, kullanıcıların şifrelerini sıfırlamalarına, sahip oldukları bir cihaza gönderilen bir Kerelik Şifre (One Time Password - OTP) talep ederek veya kayıt sırasında seçtikleri bir güvenlik sorusunun cevabını vererek izin verir. Güçlü şifre politikaları nedeniyle şifreleri brute-force yöntemiyle kırmak mümkün değilse, düşük entropiye sahip olmaları veya tahmin edilebilir olmaları durumunda OTP'leri veya güvenlik sorularının cevaplarını brute-force ile denemeyi düşünebiliriz (özellikle de hız sınırlaması uygulanmıyorsa).


### Prevention

Broken Authentication güvenlik açığını azaltmak için, /api/v1/authentication/customers/sign-in endpoint, brute-force saldırılarını önlemek için hız sınırlaması uygulamalıdır. Bu, belirli bir zaman dilimi içinde tek bir IP adresinden veya kullanıcı hesabından giriş denemelerinin sayısını sınırlayarak gerçekleştirilebilir.

Ayrıca, web API'si hem kayıt hem de güncellemeler sırasında kullanıcı kimlik bilgileri (müşteriler ve tedarikçiler dahil) için sağlam bir parola politikası uygulamalı ve yalnızca kriptografik olarak güvenli parolalara izin vermelidir. Bu politika şunları içermelidir:

- Minimum şifre uzunluğu (örneğin, en az 12 karakter)
- Karmaşıklık gereksinimleri (örneğin, büyük ve küçük harflerin, sayıların ve özel karakterlerin karışımı)
- Yaygın olarak kullanılan veya kolay tahmin edilebilir şifrelerin yasaklanması (örneğin, sızdırılmış şifre veritabanlarında bulunanlar)
- Şifre geçmişinin zorunlu tutulması, böylece yakın zamanda kullanılan şifrelerin yeniden kullanımının önlenmesi
- Düzenli şifre süresi dolumu ve zorunlu şifre değişiklikleri

Buna ek olarak, web API endpoint, kullanıcıları tam olarak kimlik doğrulamadan önce bir OTP talep ederek ek güvenlik sağlamak için Çok Faktörlü Kimlik Doğrulama (Multi-Factor Authentication - MFA) uygulamalıdır.