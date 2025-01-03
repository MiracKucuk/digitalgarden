---
{"dg-publish":true,"permalink":"/portswigger/sql-injection/","tags":["gardenEntry"]}
---



**SQL Enjeksiyonu Tespit Yöntemleri:**

1. **Manuel Test:**
    
    - `'` (tek tırnak) ile hata veya anormallik kontrolü.
    - Boolean koşulları: `OR 1=1`, `OR 1=2` ile yanıt farklarını gözlemleme.
    - Zaman gecikmesi tetikleyen payload’lar ile yanıt süresini analiz etme.
    - Out-of-band ağ etkileşimleri tetikleyen OAST payload’ları kullanma.

2. **Otomatik Test:**

    - **Burp Scanner** gibi araçlar kullanarak hızlı ve etkili tarama yapabilirsiniz.


**SQL Enjeksiyonu Sorgu Konumları:**

- **WHERE Cümlesi:** En yaygın enjeksiyon noktası, genellikle `SELECT` sorgularında.
- **UPDATE:** Güncellenen değerler veya WHERE cümlesinde.
- **INSERT:** Eklenen değerlerde.
- **SELECT:** Tablo/sütun adı veya `ORDER BY` ifadesinde.

SQL enjeksiyonu sorgunun farklı bölümlerinde ve sorgu türlerinde ortaya çıkabilir.



**SQL Enjeksiyonu Örnekleri:**

1. **Gizli Veri Alma:** Sorguyu değiştirerek ek veriler döndürmek.
2. **Mantık Altüst Etme:** Uygulama mantığını değiştirmek için sorguyu manipüle etme.
3. **UNION Saldırıları:** Farklı tabloların verilerini birleştirerek elde etme.
4. **Blind SQL Enjeksiyonu:** Sorgu sonuçları doğrudan görünmese bile saldırı gerçekleştirme.



### Gizli verileri alma

Ürünleri farklı kategorilerde görüntüleyen bir alışveriş uygulaması düşünün. Kullanıcı Gifts kategorisine tıkladığında, tarayıcısı URL'yi ister:

https://insecure-website.com/products?category=Gifts

Bu, uygulamanın ilgili ürünlerin ayrıntılarını veritabanından almak için bir SQL sorgusu yapmasına neden olur:

`SELECT * FROM products WHERE category = 'Gifts' AND released = 1`

[[Bağlantılar/released acıklama\|released acıklama]]

Bu SQL sorgusu, veritabanından şunları istemektedir:

- Tüm detaylar (*),
- **products** tablosundan,
- **category** değeri **Gifts** olan,
- Ve **released** değeri **1** olan kayıtlar.


released = 1 kısıtlaması, piyasaya sürülmemiş ürünleri gizlemek için kullanılmaktadır. Yayınlanmamış ürünler için yayınlanmış = 0 olduğunu varsayabiliriz.

Uygulama SQL enjeksiyon saldırılarına karşı herhangi bir savunma uygulamamaktadır. Bu, bir saldırganın örneğin aşağıdaki saldırıyı oluşturabileceği anlamına gelir:

https://insecure-website.com/products?category=Gifts'--

Bu, SQL sorgusuyla sonuçlanır:

`SELECT * FROM products WHERE category = 'Gifts'--' AND released = 1`

Önemli olarak, -- işaretinin SQL'de bir yorum göstergesi olduğunu unutmayın. Bu, sorgunun geri kalanının bir yorum olarak yorumlanacağı ve etkili bir şekilde kaldırılacağı anlamına gelir. Bu örnekte, bu, sorgunun artık AND released = 1 içermediği anlamına gelir. Sonuç olarak, henüz piyasaya sürülmemiş olanlar da dahil olmak üzere tüm ürünler görüntülenir.

Benzer bir saldırı kullanarak, kullanıcıların bilmediği kategoriler dahil olmak üzere herhangi bir kategorideki tüm ürünleri uygulamanın görüntülemesini sağlayabilirsiniz.

https://insecure-website.com/products?category=Gifts'+OR+1=1--

Bu, SQL sorgusuyla sonuçlanır:

`SELECT * FROM products WHERE category = 'Gifts' OR 1=1--' AND released = 1`

Değiştirilmiş sorgu, kategorinin Gifts olduğu veya 1'in 1'e eşit olduğu tüm öğeleri döndürür. 1=1 her zaman doğru olduğundan, sorgu tüm öğeleri döndürür.


### Uyarı
Bir SQL sorgusuna OR 1=1 koşulunu enjekte ederken dikkatli olun. Enjekte ettiğiniz ortamda zararsız gibi görünse bile, uygulamaların tek bir istekten gelen verileri birden fazla farklı sorguda kullanması yaygındır. Örneğin, koşulunuz bir UPDATE veya DELETE deyimine ulaşırsa, yanlışlıkla veri kaybına neden olabilir.


### Laboratuvar: WHERE cümlesinde gizli verilerin alınmasına izin veren SQL enjeksiyonu açığı

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Kullanıcı bir kategori seçtiğinde, uygulama aşağıdaki gibi bir SQL sorgusu gerçekleştirir:

SELECT * FROM products WHERE category = 'Gifts' AND released = 1

Laboratuvarı çözmek için, uygulamanın bir veya daha fazla yayınlanmamış ürünü görüntülemesine neden olan bir SQL enjeksiyon saldırısı gerçekleştirin.


Çözüm : 

* Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.
* Kategori parametresini değiştirerek '+OR+1=1--' değerini verin.
* Talebi gönderin ve yanıtın artık bir veya daha fazla yayınlanmamış ürün içerdiğini doğrulayın.

![Pasted image 20241212140202.png](/img/user/resimler/Pasted%20image%2020241212140202.png)



### Uygulama mantığını alt üst etme

Kullanıcıların bir kullanıcı adı ve parola ile oturum açmasına izin veren bir uygulama düşünün. Bir kullanıcı wiener kullanıcı adını ve bluecheese parolasını gönderirse, uygulama aşağıdaki SQL sorgusunu gerçekleştirerek kimlik bilgilerini kontrol eder:

`SELECT * FROM users WHERE username = 'wiener' AND password = 'bluecheese'`

Sorgu bir kullanıcının ayrıntılarını döndürürse, oturum açma işlemi başarılı olur. Aksi takdirde reddedilir.

Bu durumda, bir saldırgan parolaya ihtiyaç duymadan herhangi bir kullanıcı olarak oturum açabilir. Bunu, sorgunun WHERE cümlesinden parola kontrolünü kaldırmak için SQL yorum dizisini -- kullanarak yapabilirler. Örneğin, administrator'-- kullanıcı adını ve boş bir parolayı göndermek aşağıdaki sorguyla sonuçlanır:

`SELECT * FROM users WHERE username = 'administrator'--' AND password = ''`

Bu sorgu, kullanıcı adı administrator olan kullanıcıyı döndürür ve saldırganı bu kullanıcı olarak başarıyla oturum açar.


### Laboratuvar: Oturum açma atlamasına izin veren SQL enjeksiyonu güvenlik açığı

Bu laboratuvar, oturum açma işlevinde bir SQL enjeksiyonu güvenlik açığı içerir.

Laboratuvarı çözmek için, uygulamada administrator kullanıcısı olarak oturum açan bir SQL enjeksiyon saldırısı gerçekleştirin.

![Pasted image 20241212141439.png](/img/user/resimler/Pasted%20image%2020241212141439.png)


![Pasted image 20241212141413.png](/img/user/resimler/Pasted%20image%2020241212141413.png)



### SQL enjeksiyonu UNION saldırıları
Bir uygulama SQL enjeksiyonuna karşı savunmasız olduğunda ve sorgunun sonuçları uygulamanın yanıtları içinde döndürüldüğünde, veritabanındaki diğer tablolardan veri almak için UNION anahtar sözcüğünü kullanabilirsiniz. Bu genellikle SQL enjeksiyonu UNION saldırısı olarak bilinir.

UNION anahtar sözcüğü, bir veya daha fazla ek SELECT sorgusu yürütmenizi ve sonuçları orijinal sorguya eklemenizi sağlar. Örneğin:

`SELECT a, b FROM table1 UNION SELECT c, d FROM table2`

Bu SQL sorgusu, tablo1'deki a ve b sütunlarından ve tablo2'deki c ve d sütunlarından değerler içeren iki sütunlu tek bir sonuç kümesi döndürür.

UNION sorgusunun çalışması için iki temel gereksinimin karşılanması gerekir:

* Ayrı sorgular aynı sayıda sütun döndürmelidir.
* Her sütundaki veri türleri, ayrı sorgular arasında uyumlu olmalıdır.

Bir SQL enjeksiyonu UNION saldırısı gerçekleştirmek için, saldırınızın bu iki gereksinimi karşıladığından emin olun. Bu normalde şunları bulmayı içerir:

* Orijinal sorgudan kaç sütun döndürüldüğü.
* Orijinal sorgudan döndürülen hangi sütunların, enjekte edilen sorgunun sonuçlarını tutmak için uygun bir veri türüne sahip olduğu.


### Gerekli sütun sayısını belirleme

Bir SQL enjeksiyonu UNION saldırısı gerçekleştirdiğinizde, orijinal sorgudan kaç sütun döndürüldüğünü belirlemek için iki etkili yöntem vardır.

Yöntemlerden biri, bir dizi ORDER BY cümlesinin enjekte edilmesini ve bir hata oluşana kadar belirtilen sütun dizininin artırılmasını içerir. Örneğin, enjeksiyon noktası orijinal sorgunun WHERE cümlesinde tırnak içine alınmış bir string ise, gönderirsiniz:

`' ORDER BY 1--` 
`' ORDER BY 2--` 
`' ORDER BY 3--` 
`etc.`

Bu payload serisi, sonuçları sonuç kümesindeki farklı sütunlara göre sıralamak için orijinal sorguyu değiştirir. ORDER BY cümlesindeki sütun, indeksiyle belirtilebilir, bu nedenle herhangi bir sütunun adını bilmeniz gerekmez. Belirtilen sütun dizini sonuç kümesindeki gerçek sütun sayısını aştığında, veritabanı aşağıdaki gibi bir hata döndürür:

ORDER BY pozisyon numarası 3, seçim listesindeki öğe sayısı aralığının dışındadır.

Uygulama aslında HTTP yanıtında veritabanı hatasını döndürebilir, ancak genel bir hata yanıtı da verebilir. Diğer durumlarda, hiçbir sonuç döndürmeyebilir. Her iki durumda da, yanıtta bazı farklılıklar tespit edebildiğiniz sürece, sorgudan kaç sütun döndürüldüğünü çıkarabilirsiniz.

İkinci yöntem, farklı sayıda null değer belirten bir dizi UNION SELECT payload'u göndermeyi içerir:

`' UNION SELECT NULL--` 
`' UNION SELECT NULL,NULL--` 
`' UNION SELECT NULL,NULL,NULL--` 
`etc.`

Null sayısı sütun sayısıyla eşleşmezse, veritabanı aşağıdaki gibi bir hata döndürür:

UNION, INTERSECT veya EXCEPT operatörü kullanılarak birleştirilen tüm sorgular, hedef listelerinde eşit sayıda ifadeye sahip olmalıdır.

Enjekte edilen SELECT sorgusundan döndürülen değerler olarak NULL kullanırız çünkü her sütundaki veri türlerinin orijinal ve enjekte edilen sorgular arasında uyumlu olması gerekir. NULL her yaygın veri türüne dönüştürülebilir, bu nedenle sütun sayısı doğru olduğunda payload'un başarılı olma şansını en üst düzeye çıkarır.

ORDER BY tekniğinde olduğu gibi, uygulama HTTP yanıtında veritabanı hatasını gerçekten döndürebilir, ancak genel bir hata döndürebilir veya hiçbir sonuç döndüremeyebilir. Null sayısı sütun sayısıyla eşleştiğinde, veritabanı sonuç kümesinde her sütunda null değerler içeren ek bir satır döndürür. HTTP yanıtı üzerindeki etki uygulamanın koduna bağlıdır. Şanslıysanız, yanıtta bir HTML tablosunda fazladan bir satır gibi bazı ek içerikler görürsünüz. Aksi takdirde, null değerler NullPointerException gibi farklı bir hatayı tetikleyebilir. En kötü durumda, yanıt yanlış sayıda null değerin neden olduğu bir yanıtla aynı görünebilir. Bu da bu yöntemi etkisiz hale getirir.



### Lab: SQL enjeksiyonu UNION saldırısı, sorgu tarafından döndürülen sütun sayısını belirleme

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, bu nedenle diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz. Böyle bir saldırının ilk adımı, sorgu tarafından döndürülen sütun sayısını belirlemektir. Daha sonra bu tekniği sonraki laboratuvarlarda tam saldırıyı oluşturmak için kullanacaksınız.

Laboratuvarı çözmek için, null değerler içeren ek bir satır döndüren bir SQL enjeksiyon UNION saldırısı gerçekleştirerek sorgu tarafından döndürülen sütun sayısını belirleyin.

1. Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.
2. Kategori parametresini değiştirerek '+UNION+SELECT+NULL-- değerini verin. Bir hata oluştuğunu gözlemleyin.
3. Null değeri içeren ek bir sütun eklemek için kategori parametresini değiştirin:

`'+UNION+SELECT+NULL,NULL--`

4. Hata ortadan kalkana ve yanıt null değerleri içeren ek içerik içerene kadar null değerleri eklemeye devam edin

![Pasted image 20241212150525.png](/img/user/resimler/Pasted%20image%2020241212150525.png)



### Database-specific syntax

Oracle'da her SELECT sorgusu FROM anahtar sözcüğünü kullanmalı ve geçerli bir tablo belirtmelidir. Oracle'da bu amaçla kullanılabilecek dual adında built-in bir tablo vardır. Dolayısıyla, Oracle'da enjekte edilen sorguların aşağıdaki gibi görünmesi gerekir:

`' UNION SELECT NULL FROM DUAL--`

Payloadlarda tanımlanan yöntemler, enjeksiyon noktasını takip eden orijinal sorgunun geri kalanını yorum satırı haline getirmek için çift çizgi yorum dizisini `--` kullanır. MySQL'de, çift çizgi dizisinden sonra bir boşluk gelmelidir. Alternatif olarak, bir yorumu belirtmek için kare işareti `#` kullanılabilir.

Veritabanına özgü sözdizimi hakkında daha fazla bilgi için SQL enjeksiyonu [hile sayfasına](https://portswigger.net/web-security/sql-injection/cheat-sheet) bakın.



### Kullanışlı Bir Veri Türüne Sahip Sütunları Bulma

Bir SQL enjeksiyonu UNION saldırısı, enjekte edilmiş bir sorgunun sonuçlarını almanızı sağlar. Elde etmek istediğiniz ilginç veriler genellikle string formundadır. Bu, orijinal sorgu sonuçlarında veri türü string olan veya string verilerle uyumlu olan bir ya da daha fazla sütun bulmanız gerektiği anlamına gelir.

Gerekli sütun sayısını belirledikten sonra, her bir sütunun string veri barındırıp barındıramayacağını test edebilirsiniz. Bunun için her sütuna sırayla bir string değer yerleştiren bir dizi UNION SELECT payload’u gönderebilirsiniz. Örneğin, sorgu dört sütun döndürüyorsa şu şekilde gönderirsiniz:

`' UNION SELECT 'a',NULL,NULL,NULL--`  
`' UNION SELECT NULL,'a',NULL,NULL--`  
`' UNION SELECT NULL,NULL,'a',NULL--`  
`' UNION SELECT NULL,NULL,NULL,'a'--`  

Eğer sütun veri türü string verilerle uyumlu değilse, enjekte edilmiş sorgu bir veritabanı hatasına neden olur, örneğin:

`Conversion failed when converting the varchar value 'a' to data type int.`

Eğer bir hata oluşmaz ve uygulamanın yanıtında, enjekte edilen string değeri içeren ek bir içerik görünürse, ilgili sütun string verileri elde etmek için uygundur.



### Laboratuvar: SQL enjeksiyonu UNION saldırısı, text içeren bir sütun bulma

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, bu nedenle diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz. Böyle bir saldırı oluşturmak için öncelikle sorgu tarafından döndürülen sütun sayısını belirlemeniz gerekir. Bunu daha önceki bir laboratuvarda öğrendiğiniz bir tekniği kullanarak yapabilirsiniz. Bir sonraki adım, string verileriyle uyumlu bir sütun belirlemektir.

Laboratuvar, sorgu sonuçları içinde görünmesini sağlamanız gereken rastgele bir değer sağlayacaktır. Laboratuvarı çözmek için, sağlanan değeri içeren ek bir satır döndüren bir SQL enjeksiyon UNION saldırısı gerçekleştirin. Bu teknik, hangi sütunların string verileriyle uyumlu olduğunu belirlemenize yardımcı olur.

Çözüm : 

1. Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.

2. Sorgu tarafından döndürülen sütun sayısını belirleyin. Kategori parametresinde aşağıdaki payload'u kullanarak sorgunun üç sütun döndürdüğünü doğrulayın:

 `'+UNION+SELECT+NULL,NULL,NULL--`
 
3. Her boş değeri laboratuvar tarafından sağlanan rastgele değerle değiştirmeyi deneyin, örneğin:

`'+UNION+SELECT+'abcdef',NULL,NULL--`

4. Bir hata oluşursa, bir sonraki null'a geçin ve bunun yerine onu deneyin.

![Pasted image 20241212203051.png](/img/user/resimler/Pasted%20image%2020241212203051.png)

Edelco'yu aramamızı söylüyor.

![Pasted image 20241212203251.png](/img/user/resimler/Pasted%20image%2020241212203251.png)



### İlginç verileri almak için SQL enjeksiyonu UNION saldırısı kullanma

Orijinal sorgunun döndürdüğü sütun sayısını ve hangi sütunların string veri barındırabildiğini belirledikten sonra, ilginç verileri elde etme aşamasına geçebilirsiniz.

Diyelim ki:

- Orijinal sorgu, her ikisi de string veri barındırabilen iki sütun döndürüyor.
- Enjeksiyon noktası, WHERE ifadesi içinde tırnak içine alınmış bir string.
- Veritabanında `users` adında bir tablo ve bu tabloda `username` ve `password` adında sütunlar bulunuyor.

Bu durumda, aşağıdaki girdiyi göndererek `users` tablosunun içeriğini elde edebilirsiniz:

`' UNION SELECT username, password FROM users--`

Bu saldırıyı gerçekleştirebilmek için, veritabanında `users` adında bir tablo ve bu tabloda `username` ve `password` adında iki sütun bulunduğunu bilmeniz gerekir. Bu bilgiye sahip olmadığınızda, tablo ve sütun isimlerini tahmin etmeniz gerekebilir. Modern veritabanlarının tamamı, veritabanı yapısını incelemek ve hangi tablo ve sütunları içerdiğini belirlemek için yöntemler sunar.


### Laboratuvar: SQL enjeksiyonu UNION saldırısı, diğer tablolardan veri alma

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, bu nedenle diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz. Böyle bir saldırı oluşturmak için önceki laboratuvarlarda öğrendiğiniz bazı teknikleri birleştirmeniz gerekir.

Veritabanı, users adında, username ve password adlı sütunlara sahip farklı bir tablo içerir.

Laboratuvarı çözmek için, tüm username ve password'leri alan bir SQL injection UNION saldırısı gerçekleştirin ve bu bilgileri administrator kullanıcısı olarak oturum açmak için kullanın.

1. Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.

2. Sorgu tarafından döndürülen sütun sayısını ve hangi sütunların metin verileri içerdiğini belirleyin. Sorgunun, kategori parametresinde aşağıdaki gibi bir payload kullanarak her ikisi de metin içeren iki sütun döndürdüğünü doğrulayın:

`'+UNION+SELECT+'abc','def'--`

3. Users tablosunun içeriğini almak için aşağıdaki payload'u kullanın:

`'+UNION+SELECT+username,+password+FROM+users--`

4. Uygulamanın yanıtının username ve passwords içerdiğini doğrulayın.


![Pasted image 20241212204247.png](/img/user/resimler/Pasted%20image%2020241212204247.png)

![Pasted image 20241212204313.png](/img/user/resimler/Pasted%20image%2020241212204313.png)

![Pasted image 20241212204331.png](/img/user/resimler/Pasted%20image%2020241212204331.png)

![Pasted image 20241212204426.png](/img/user/resimler/Pasted%20image%2020241212204426.png)



### Tek bir sütun içerisinde birden fazla değer alma

Bazı durumlarda, önceki örnekteki sorgu yalnızca tek bir sütun döndürebilir.

Değerleri birleştirerek bu tek sütun içinde birden fazla değeri birlikte alabilirsiniz. Birleştirilmiş değerleri ayırt edebilmeniz için bir ayırıcı ekleyebilirsiniz. Örneğin, Oracle'da girişi gönderebilirsiniz:

`' UNION SELECT username || '~' || password FROM users--`

Bu, Oracle'da bir string birleştirme operatörü olan çift borulu || dizisini kullanır. Enjekte edilen sorgu, ~ karakteriyle ayrılmış kullanıcı adı ve parola alanlarının değerlerini bir araya getirir.

Sorgudan elde edilen sonuçlar, örneğin tüm kullanıcı adlarını ve parolaları içerir:

...
administrator~s3cure
wiener~peter
carlos~montoya
...

Farklı veritabanları, string birleştirme işlemini gerçekleştirmek için farklı sözdizimi kullanır. Daha fazla ayrıntı için SQL enjeksiyonu [hile sayfasına](https://portswigger.net/web-security/sql-injection/cheat-sheet) bakın.



### Laboratuvar: SQL enjeksiyonu UNION saldırısı, tek bir sütunda birden fazla değer alma

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, böylece diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz.

Veritabanı, users adında, username ve password adlı sütunlara sahip farklı bir tablo içerir.

Laboratuvarı çözmek için, tüm kullanıcı adlarını ve parolaları alan bir SQL enjeksiyon UNION saldırısı gerçekleştirin ve bilgileri yönetici kullanıcı olarak oturum açmak için kullanın.

1. Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.

2. Sorgu tarafından döndürülen sütun sayısını ve hangi sütunların metin verileri içerdiğini belirleyin. Sorgunun, kategori parametresinde aşağıdaki gibi bir payload kullanarak yalnızca biri text içeren iki sütun döndürdüğünü doğrulayın:

`'+UNION+SELECT+NULL,'abc'--`

3. Users tablosunun içeriğini almak için aşağıdaki payload'u kullanın:

`'+UNION+SELECT+NULL,username||'~'||password+FROM+users--`

4. Uygulamanın yanıtının username ve passwords içerdiğini doğrulayın.


![Pasted image 20241212211816.png](/img/user/resimler/Pasted%20image%2020241212211816.png)

ilk sütun'a bir string girdiğimizde hata alıyoruz.

![Pasted image 20241212211856.png](/img/user/resimler/Pasted%20image%2020241212211856.png)

![Pasted image 20241212211916.png](/img/user/resimler/Pasted%20image%2020241212211916.png)

![Pasted image 20241212212018.png](/img/user/resimler/Pasted%20image%2020241212212018.png)



### SQL enjeksiyon saldırılarında veritabanının incelenmesi

SQL enjeksiyonu güvenlik açıklarından yararlanmak için genellikle veritabanı hakkında bilgi bulmak gerekir. Bu bilgiler şunları içerir:

* Veritabanı yazılımının türü ve sürümü.
* Veritabanının içerdiği tablolar ve sütunlar.


### Veritabanı türünü ve sürümünü sorgulama

Birinin çalışıp çalışmadığını görmek için sağlayıcıya özgü sorgular ekleyerek hem veritabanı türünü hem de sürümünü belirleyebilirsiniz

Aşağıda, bazı popüler veritabanı türleri için veritabanı sürümünü belirlemeye yönelik bazı sorgular yer almaktadır:

|   |   |
|---|---|
|Database type|Query|
|Microsoft, MySQL|`SELECT @@version`|
|Oracle|`SELECT * FROM v$version`|
|PostgreSQL|`SELECT version()`|
Örneğin, aşağıdaki girdi ile bir UNION saldırısı kullanabilirsiniz:

`' UNION SELECT @@version--`

Bu, aşağıdaki çıktıyı döndürebilir. Bu durumda, veritabanının Microsoft SQL Server olduğunu doğrulayabilir ve kullanılan sürümü görebilirsiniz:

`Microsoft SQL Server 2016 (SP2) (KB4052908) - 13.0.5026.0 (X64) Mar 18 2018 09:11:49 Copyright (c) Microsoft Corporation Standard Edition (64-bit) on Windows Server 2016 Standard 10.0 <X64> (Build 14393: ) (Hypervisor)`



### Lab: SQL enjeksiyon saldırısı, MySQL ve Microsoft'ta veritabanı türünü ve sürümünü sorgulama

Bu laboratuvar, ürün kategorisi filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Enjekte edilen bir sorgudan sonuçları almak için bir UNION saldırısı kullanabilirsiniz.

Laboratuvarı çözmek için veritabanı sürüm dizesini görüntüleyin.

1. Product category filtresini ayarlayan isteği kesmek ve değiştirmek için Burp Suite'i kullanın.

2. Sorgu tarafından döndürülen sütun sayısını ve hangi sütunların metin verileri içerdiğini belirleyin. Sorgunun, kategori parametresinde aşağıdaki gibi bir payload kullanarak her ikisi de metin içeren iki sütun döndürdüğünü doğrulayın:

`'+UNION+SELECT+'abc','def'#`

3. Veritabanı sürümünü görüntülemek için aşağıdaki payload'u kullanın:

`' UNION+SELECT @@version, NULL#`

![Pasted image 20241212221257.png](/img/user/resimler/Pasted%20image%2020241212221257.png)



### Veritabanının içeriğini listeleme

Çoğu veritabanı türü (Oracle hariç) information schema adı verilen bir view kümesine sahiptir. Bu, veritabanı hakkında bilgi sağlar.

Örneğin, veritabanındaki tabloları listelemek için information_schema.tables dosyasını sorgulayabilirsiniz:

`SELECT * FROM information_schema.tables`

Bu, aşağıdaki gibi bir çıktı döndürür:

![Pasted image 20241213034026.png](/img/user/resimler/Pasted%20image%2020241213034026.png)

Bu çıktı, Products (Ürünler), Users (Kullanıcılar) ve Feedback (Geri Bildirim) adında üç tablo olduğunu gösterir.

Daha sonra tek tek tablolardaki sütunları listelemek için information_schema.columns dosyasını sorgulayabilirsiniz:

`SELECT * FROM information_schema.columns WHERE table_name = 'Users'`

Bu, aşağıdaki gibi bir çıktı döndürür:

![Pasted image 20241213034158.png](/img/user/resimler/Pasted%20image%2020241213034158.png)

Bu çıktı, belirtilen tablodaki sütunları ve her bir sütunun veri türünü gösterir.


### Lab: SQL Enjeksiyon Saldırısı, Oracle Olmayan Veritabanlarında Veritabanı İçeriğini Listeleme

Bu laboratuvar, product category filtresinde bir SQL enjeksiyonu güvenlik açığı içerir. Sorgudan elde edilen sonuçlar uygulamanın yanıtında döndürülür, böylece diğer tablolardan veri almak için bir UNION saldırısı kullanabilirsiniz.

Uygulamanın bir oturum açma fonksiyonu vardır ve veritabanı kullanıcı adlarını ve parolaları tutan bir tablo içerir. Bu tablonun adını ve içerdiği sütunları belirlemeniz, ardından tüm kullanıcıların kullanıcı adı ve parolalarını elde etmek için tablonun içeriğini almanız gerekir.

Laboratuvarı çözmek için administrator kullanıcısı olarak oturum açın.

1. Sorgu tarafından döndürülen sütun sayısını ve hangi sütunların metin verileri içerdiğini belirleyin. Kategori parametresinde aşağıdaki gibi bir payload kullanarak sorgunun her ikisi de metin içeren iki sütun döndürdüğünü doğrulayın:

`'+UNION+SELECT+'abc','def'--`

2. Veritabanındaki tabloların listesini almak için aşağıdaki payload'u kullanın:

`'+UNION+SELECT+table_name,+NULL+FROM+information_schema.tables--`

3. Kullanıcı kimlik bilgilerini içeren tablonun adını bulun.

4. Tablodaki sütunların ayrıntılarını almak için aşağıdaki payload'u (tablo adını değiştirerek) kullanın:

`'+UNION+SELECT+column_name,+NULL+FROM+information_schema.columns+WHERE+table_name='users_abcdef'--`


5. Kullanıcı adlarını ve parolaları içeren sütunların adlarını bulun.

6. Tüm kullanıcıların kullanıcı adlarını ve parolalarını almak için aşağıdaki payload'u (tablo ve sütun adlarını değiştirerek) kullanın:

`'+UNION+SELECT+username_abcdef,+password_abcdef+FROM+users_abcdef--`


7. Administrator kullanıcısının parolasını bulun ve oturum açmak için bu parolayı kullanın.



![Pasted image 20241213035129.png](/img/user/resimler/Pasted%20image%2020241213035129.png)

![Pasted image 20241213035143.png](/img/user/resimler/Pasted%20image%2020241213035143.png)


![Pasted image 20241213035159.png](/img/user/resimler/Pasted%20image%2020241213035159.png)

![Pasted image 20241213035227.png](/img/user/resimler/Pasted%20image%2020241213035227.png)

![Pasted image 20241213035714.png](/img/user/resimler/Pasted%20image%2020241213035714.png)

![Pasted image 20241213035742.png](/img/user/resimler/Pasted%20image%2020241213035742.png)

![Pasted image 20241213035913.png](/img/user/resimler/Pasted%20image%2020241213035913.png)


## Blind SQL injection

Bu bölümde, blind SQL injection güvenlik açıklarını bulmaya ve kullanmaya yönelik teknikleri açıklayacağız.

Blind SQL enjeksiyonu, bir uygulama SQL enjeksiyonuna karşı savunmasız olduğunda, ancak HTTP yanıtları ilgili SQL sorgusunun sonuçlarını veya herhangi bir veritabanı hatasının ayrıntılarını içermediğinde ortaya çıkar.

UNION saldırıları gibi birçok teknik Blind SQL injection güvenlik açıklarında etkili değildir. Bunun nedeni, enjekte edilen sorgunun sonuçlarını uygulamanın yanıtları içinde görebilmeye dayanmalarıdır. Yetkisiz verilere erişmek için blind SQL injection'dan faydalanmak hala mümkündür, ancak farklı teknikler kullanılmalıdır.


### Koşullu yanıtları tetikleyerek blind SQL enjeksiyonundan yararlanma

Kullanıcı kullanım analizlerini toplamak için takip çerezleri kullanan bir uygulamayı ele alalım. Uygulamaya yapılan istekler, şu şekilde bir çerez (cookie) başlığı içerir:

`Cookie: TrackingId=u5YD3PapBcR4lN3e7Tj4`

TrackingId çerezi içeren bir istek işlendiğinde, uygulama bunun bilinen bir kullanıcı olup olmadığını belirlemek için bir SQL sorgusu kullanır:

`SELECT TrackingId FROM TrackedUsers WHERE TrackingId = 'u5YD3PapBcR4lN3e7Tj4'`

Bu sorgu SQL enjeksiyonuna karşı savunmasızdır, ancak sorgudan dönen sonuçlar kullanıcıya iletilmez. Ancak, sorgunun herhangi bir veri döndürüp döndürmediğine bağlı olarak uygulama farklı davranır. Eğer tanınan bir `TrackingId` gönderirseniz, sorgu veri döndürür ve yanıt olarak bir "Tekrar hoş geldiniz" mesajı alırsınız.

Bu davranış, blind SQL enjeksiyon zafiyetini sömürmek için yeterlidir. Enjekte edilen bir koşula bağlı olarak farklı yanıtlar tetikleyerek bilgi elde edebilirsiniz.

Bu exploit'in nasıl çalıştığını anlamak için, sırayla aşağıdaki TrackingId cookie değerlerini içeren iki istek gönderildiğini varsayalım:

`…xyz' AND '1'='1`

`…xyz' AND '1'='2`

* Bu değerlerden ilki sorgunun sonuç döndürmesine neden olur, çünkü enjekte edilen AND '1'='1 koşulu doğrudur. Sonuç olarak, “Tekrar hoş geldiniz” mesajı görüntülenir.

* İkinci değer, enjekte edilen koşul yanlış olduğu için sorgunun herhangi bir sonuç döndürmemesine neden olur. “Tekrar hoş geldiniz” mesajı görüntülenmez.

Bu, enjekte edilen herhangi bir koşulun yanıtını belirlememize ve verileri her seferinde bir parça çıkarmamıza olanak tanır.

Örneğin, Users (Kullanıcılar) adında, Username (Kullanıcı Adı) ve Password (Parola) sütunlarına sahip bir tablo ve Administrator (Yönetici) adında bir kullanıcı olduğunu varsayalım. Parolayı her seferinde bir karakter test etmek için bir dizi girdi göndererek bu kullanıcının parolasını belirleyebilirsiniz. Bunu yapmak için aşağıdaki girdi ile başlayın:

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 'm`

Bu, enjekte edilen koşulun doğru olduğunu ve dolayısıyla parolanın ilk karakterinin m'den büyük olduğunu belirten “Tekrar hoş geldiniz” mesajını döndürür.

Ardından, aşağıdaki girdiyi gönderiyoruz:

`xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) > 't`


Bu, enjekte edilen koşulun yanlış olduğunu ve dolayısıyla parolanın ilk karakterinin t'den büyük olmadığını gösteren “Tekrar hoş geldiniz” mesajını döndürmez.

Sonunda, “Tekrar hoş geldiniz” mesajını döndüren ve böylece parolanın ilk karakterinin s olduğunu doğrulayan aşağıdaki girdiyi gönderiyoruz:

xyz' AND SUBSTRING((SELECT Password FROM Users WHERE Username = 'Administrator'), 1, 1) = 's

Administrator kullanıcısının tam parolasını sistematik olarak belirlemek için bu işleme devam edebiliriz.


### Not
SUBSTRING fonksiyonu bazı veritabanı türlerinde SUBSTR olarak adlandırılır. Daha fazla ayrıntı için SQL enjeksiyonu hile sayfasına bakın.


### Laboratuvar: Koşullu yanıtlarla Blind SQL enjeksiyonu
