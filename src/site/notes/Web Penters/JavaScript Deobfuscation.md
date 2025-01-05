---
{"dg-publish":true,"permalink":"/web-penters/java-script-deobfuscation/"}
---

* JavaScript kodunu bulma
* Code Obfuscation'a Giriş
* JavaScript kodu nasıl deobfuscate edilir
* Kodlanmış mesajlar nasıl çözülür?
* Basit Kod Analizi
* Temel HTTP isteklerini gönderme


![Pasted image 20241225101537.png](/img/user/resimler/Pasted%20image%2020241225101537.png)

Bunu, web sitesinin kaynak görünümünü açacak olan [CTRL + U] tuşlarına basarak yapabiliriz:

![Pasted image 20241225101600.png](/img/user/resimler/Pasted%20image%2020241225101600.png)

CSS kodu ya aynı HTML dosyası içinde `<style>` öğeleri arasında dahili olarak tanımlanır ya da ayrı bir .css dosyasında harici olarak tanımlanır ve HTML kodu içinde referans verilir.

```html
    <style>
        *,
        html {
            margin: 0;
            padding: 0;
            border: 0;
        }
        ...SNIP...
        h1 {
            font-size: 144px;
        }
        p {
            font-size: 64px;
        }
    </style>
```

Bir sayfa CSS stili externally olarak tanımlanmışsa, harici .css dosyasına HTML başlığı içinde `<link>` etiketi ile aşağıdaki gibi atıfta bulunulur:

```html
<head>
    <link rel="stylesheet" href="style.css">
</head>
```


## JavaScript

Aynı kavram JavaScript için de geçerlidir. `<script>` öğeleri arasına dahili olarak yazılabilir veya ayrı bir .js dosyasına yazılabilir ve HTML kodu içinde referans verilebilir.

HTML kaynağımızda .js dosyasına externally olarak referans verildiğini görebiliriz:

Bizi doğrudan scriptin içine götürecek olan secret.js'ye tıklayarak scripti kontrol edebiliriz. Kodu ziyaret ettiğimizde, kodun çok karmaşık olduğunu ve anlaşılamadığını görüyoruz:

```javascript
eval(function (p, a, c, k, e, d) { e = function (c) { '...SNIP... |true|function'.split('|'), 0, {}))
```

Bunun arkasındaki neden kod gizlemedir. Nedir bu? Nasıl yapılır? Nerede kullanılır?


Soru : Bu bölümde öğrendiklerinizi tekrarlayın ve gizli bir bayrak bulmalısınız, nedir o?
Cevap : 

![Pasted image 20241225102616.png](/img/user/resimler/Pasted%20image%2020241225102616.png)

![Pasted image 20241225102709.png](/img/user/resimler/Pasted%20image%2020241225102709.png)


# Code Obfuscation

Deobfuscation, obfuscation ile karmaşıklaştırılmış kodun okunabilir ve anlaşılabilir hale getirilmesi işlemidir. Obfuscation ise, kodun yapısını ve mantığını gizlemek için bilerek karmaşıklaştırılmasıdır.


### **Obfuscation nedir?**  
Obfuscation, bir scriptin insan tarafından okunmasını zorlaştıran, ancak teknik açıdan aynı şekilde çalışmasını sağlayan bir tekniktir. Performans biraz yavaşlayabilir, ancak işlevsellik korunur. Genellikle, bir obfuscation aracı kullanılarak otomatik olarak gerçekleştirilir. Bu araç, kodu girdi olarak alır ve tasarımına bağlı olarak okunması çok daha zor bir hale getirmeye çalışır.

Örneğin, kod obfuscator'ları genellikle kodda kullanılan tüm kelimeleri ve sembolleri bir sözlük halinde düzenler ve ardından bu sözlükteki her kelime ve sembole referans vererek, orijinal kodu çalıştırma sırasında yeniden oluşturur. Aşağıda basit bir JavaScript kodunun obfuscate edilmiş bir örneği verilmiştir:

![Pasted image 20241225102945.png](/img/user/resimler/Pasted%20image%2020241225102945.png)

Python, PHP ve JavaScript gibi birçok dilde yazılan kodlar yorumlanan dillerde derlenmeden yayınlanır ve çalıştırılır. Python ve PHP genellikle sunucu tarafında bulunur ve bu nedenle son kullanıcılardan gizlenirken, JavaScript genellikle client tarafındaki tarayıcılarda kullanılır ve kod kullanıcıya gönderilir ve açık metin olarak çalıştırılır. Bu nedenle obfuscation JavaScript ile çok sık kullanılır.


# Basic Obfuscation

Kod obfuscation genellikle manuel yapılmaz; birçok otomatik araç ve kötü niyetli aktörlerce geliştirilen özel araçlar bu işlemi gerçekleştirir.


## Running JavaScript code

```javascript
console.log('HTB JavaScript Deobfuscation Module');
```

[JSConsole](https://jsconsole.com/)'a gidip kodu yapıştırabilir ve enter tuşuna basarak çıktısını görebiliriz:

![Pasted image 20241225103423.png](/img/user/resimler/Pasted%20image%2020241225103423.png)

Bu kod satırının HTB JavaScript Deobfuscation Module yazdırdığını ve bunun console.log() fonksiyonu kullanılarak yapıldığını görüyoruz.


## Minifying JavaScript code

Bir JavaScript kod parçacığını tamamen fonksiyonel tutarken okunabilirliğini azaltmanın yaygın bir yolu JavaScript minification'dır. Code minification, tüm kodun tek bir satırda (genellikle çok uzun) olması anlamına gelir. Kod küçültme daha uzun kodlar için daha kullanışlıdır, çünkü kodumuz yalnızca tek bir satırdan oluşsaydı, küçültüldüğünde çok farklı görünmezdi.

[Javascript-minifier](https://javascript-minifier.com/) gibi birçok araç JavaScript kodunu küçültmemize yardımcı olabilir. Basitçe kodumuzu kopyalıyoruz ve Minify'a tıklıyoruz ve sağ tarafta küçültülmüş çıktıyı alıyoruz:

![Pasted image 20241225103717.png](/img/user/resimler/Pasted%20image%2020241225103717.png)

Bir kez daha, küçültülmüş kodu [JSConsole](https://jsconsole.com/)'a kopyalayıp çalıştırabiliriz ve beklendiği gibi çalıştığını görürüz. Genellikle, küçültülmüş JavaScript kodu .min.js uzantısı ile kaydedilir.

Not: Code minification JavaScript'e özel değildir ve javascript-minifier'da görülebileceği gibi diğer birçok dile uygulanabilir.


### JavaScript kodunu paketleme

Şimdi, kod satırımızı daha karmaşık ve zor okunur hale getirmek için obfuscate edelim. İlk olarak, kodumuzu obfuscate etmek için [BeautifyTools](http://beautifytools.com/javascript-obfuscator.php)'u kullanacağız:

![Pasted image 20241225103954.png](/img/user/resimler/Pasted%20image%2020241225103954.png)

```javascript
eval(function(p,a,c,k,e,d){e=function(c){return c};if(!''.replace(/^/,String)){while(c--){d[c]=k[c]||c}k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--){if(k[c]){p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c])}}return p}('5.4(\'3 2 1 0\');',6,6,'Module|Deobfuscation|JavaScript|HTB|log|console'.split('|'),0,{}))
```

Kodumuzun çok daha karmaşık hale geldiğini ve okunmasının zorlaştığını görüyoruz. Bu kodu https://jsconsole.com adresine kopyalayarak hala ana işlevini yerine getirdiğini doğrulayabiliriz:

![Pasted image 20241225104019.png](/img/user/resimler/Pasted%20image%2020241225104019.png)

Not: Yukarıdaki gizleme türü “ packing” olarak bilinir ve genellikle “function(p,a,c,k,e,d)” başlangıç fonksiyonunda kullanılan altı fonksiyon argümanından tanınabilir.

Bir packer obfuscation aracı genellikle kodun tüm kelime ve sembollerini bir liste veya sözlüğe dönüştürmeye çalışır ve daha sonra yürütme sırasında orijinal kodu yeniden oluşturmak için (p,a,c,k,e,d) fonksiyonunu kullanarak bunlara başvurur. (p,a,c,k,e,d) bir paketleyiciden diğerine farklı olabilir. Bununla birlikte, genellikle orijinal kodun kelime ve sembollerinin yürütme sırasında nasıl sıralanacağını bilmek için paketlendiği belirli bir sırayı içerir.


# Advanced Obfuscation

Şimdiye kadar, kodumuzu obfuscate ederek daha zor okunur hale getirdik. Ancak, kod hâlâ açık metin şeklinde stringler içeriyor ve bu da orijinal işlevini ortaya çıkarabilir. Bu bölümde, kodu tamamen obfuscate ederek orijinal işlevine dair herhangi bir izi gizlemesi gereken birkaç aracı deneyeceğiz.


## Obfuscator

https://obfuscator.io adresini ziyaret edelim. Obfuscate'e tıklamadan önce, aşağıda görüldüğü gibi String Array Encoding'i Base64 olarak değiştireceğiz:

![Pasted image 20241225104459.png](/img/user/resimler/Pasted%20image%2020241225104459.png)

Şimdi kodumuzu yapıştırabilir ve obfuscate'e tıklayabiliriz:

![Pasted image 20241225104522.png](/img/user/resimler/Pasted%20image%2020241225104522.png)

Aşağıdaki kodu alıyoruz:

```javascript
var _0x1ec6=['Bg9N','sfrciePHDMfty3jPChqGrgvVyMz1C2nHDgLVBIbnB2r1Bgu='];(function(_0x13249d,_0x1ec6e5){var _0x14f83b=function(_0x3f720f){while(--_0x3f720f){_0x13249d['push'](_0x13249d['shift']());}};_0x14f83b(++_0x1ec6e5);}(_0x1ec6,0xb4));var _0x14f8=function(_0x13249d,_0x1ec6e5){_0x13249d=_0x13249d-0x0;var _0x14f83b=_0x1ec6[_0x13249d];if(_0x14f8['eOTqeL']===undefined){var _0x3f720f=function(_0x32fbfd){var _0x523045='abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789+/=',_0x4f8a49=String(_0x32fbfd)['replace'](/=+$/,'');var _0x1171d4='';for(var _0x44920a=0x0,_0x2a30c5,_0x443b2f,_0xcdf142=0x0;_0x443b2f=_0x4f8a49['charAt'](_0xcdf142++);~_0x443b2f&&(_0x2a30c5=_0x44920a%0x4?_0x2a30c5*0x40+_0x443b2f:_0x443b2f,_0x44920a++%0x4)?_0x1171d4+=String['fromCharCode'](0xff&_0x2a30c5>>(-0x2*_0x44920a&0x6)):0x0){_0x443b2f=_0x523045['indexOf'](_0x443b2f);}return _0x1171d4;};_0x14f8['oZlYBE']=function(_0x8f2071){var _0x49af5e=_0x3f720f(_0x8f2071);var _0x52e65f=[];for(var _0x1ed1cf=0x0,_0x79942e=_0x49af5e['length'];_0x1ed1cf<_0x79942e;_0x1ed1cf++){_0x52e65f+='%'+('00'+_0x49af5e['charCodeAt'](_0x1ed1cf)['toString'](0x10))['slice'](-0x2);}return decodeURIComponent(_0x52e65f);},_0x14f8['qHtbNC']={},_0x14f8['eOTqeL']=!![];}var _0x20247c=_0x14f8['qHtbNC'][_0x13249d];return _0x20247c===undefined?(_0x14f83b=_0x14f8['oZlYBE'](_0x14f83b),_0x14f8['qHtbNC'][_0x13249d]=_0x14f83b):_0x14f83b=_0x20247c,_0x14f83b;};console[_0x14f8('0x0')](_0x14f8('0x1'));
```

Bu kod açıkça daha karmaşıktır ve orijinal kodumuzdan hiçbir kalıntı göremiyoruz. Şimdi orijinal işlevini hala yerine getirdiğini doğrulamak için https://jsconsole.com adresinde çalıştırmayı deneyebiliriz. Daha da fazla obfuscation kod oluşturmak için https://obfuscator.io adresindeki obfuscation ayarlarıyla oynamayı deneyin ve ardından orijinal işlevini yerine getirdiğini doğrulamak için https://jsconsole.com adresinde yeniden çalıştırmayı deneyin.


## More Obfuscation

```javascript
[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]][([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(!
...SNIP...
[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+([][[]]+[])[+!+[]]+(![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[+!+[]]+([][[]]+[])[+[]]+([][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[][(![]+[])[+[]]+([![]]+[][[]])[+!+[]+[+[]]]+(![]+[])[!+[]+!+[]]+(!![]+[])[+[]]+(!![]+[])[!+[]+!+[]+!+[]]+(!![]+[])[+!+[]]])[+!+[]+[+[]]]+(!![]+[])[+!+[]]])[!+[]+!+[]+[+[]]]](!+[]+!+[]+[+[]])))()
```

Bu kodu hala çalıştırabiliriz ve orijinal işlevini yerine getirmeye devam eder:

![Pasted image 20241225104712.png](/img/user/resimler/Pasted%20image%2020241225104712.png)

Not: Yukarıdaki kod, tam kodun çok uzun olması nedeniyle kısaltılmıştır, ancak tam kod başarıyla çalışmalıdır.

Aynı [JSF](http://www.jsfuck.com/) aracını kullanarak kodu obfuscate edebilir ve ardından yeniden çalıştırabiliriz. Kodun çalışmasının biraz zaman alabileceğini fark ederiz; bu da daha önce belirtildiği gibi, kod obfuscation'ın performansı nasıl etkileyebileceğini gösterir.

[JJ Encode](https://utf-8.jp/public/jjencode.html) veya[ AA Encode](https://utf-8.jp/public/aaencode.html) gibi başka birçok JavaScript obfuscator aracı da bulunmaktadır. Ancak, bu tür obfuscator'lar genellikle kodun çalıştırılmasını/derlenmesini çok yavaş hale getirir, bu yüzden yalnızca web filtrelerini veya kısıtlamalarını aşmak gibi açık bir neden varsa kullanılması önerilir.


# Deobfuscation

## Beautify

Şu anda elimizdeki kodun tek bir satırda yazılmış olduğunu görüyoruz. Bu, Minified JavaScript kodu olarak bilinir. Kodu doğru bir şekilde formatlamak için kodumuzu **Beautify** etmemiz gerekir. Bunun en temel yöntemi, tarayıcı **Dev Tools** kullanarak yapılır.

Örneğin, Firefox kullanıyorsak, tarayıcı **Debugger**'ını [ CTRL+SHIFT+Z ] tuşlarıyla açabilir ve ardından script dosyamız olan **secret.js**'e tıklayabiliriz. Bu, script'i orijinal formatında gösterecektir. Ancak, aşağıdaki '{ }' düğmesine tıklayarak script'i düzgün JavaScript formatına **Pretty Print** edebiliriz.

![Pasted image 20241225111323.png](/img/user/resimler/Pasted%20image%2020241225111323.png)

Ayrıca, [Prettier](https://prettier.io/playground/) veya [Beautifier](https://beautifier.io/) gibi birçok çevrimiçi araç veya kod düzenleyici eklentisi kullanabiliriz. Secret.js scriptini kopyalayalım:

```javascript
eval(function (p, a, c, k, e, d) { e = function (c) { return c.toString(36) }; if (!''.replace(/^/, String)) { while (c--) { d[c.toString(a)] = k[c] || c.toString(a) } k = [function (e) { return d[e] }]; e = function () { return '\\w+' }; c = 1 }; while (c--) { if (k[c]) { p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]) } } return p }('g 4(){0 5="6{7!}";0 1=8 a();0 2="/9.c";1.d("e",2,f);1.b(3)}', 17, 17, 'var|xhr|url|null|generateSerial|flag|HTB|flag|new|serial|XMLHttpRequest|send|php|open|POST|true|function'.split('|'), 0, {}))
```

Her iki web sitesinin de kodu formatlama konusunda iyi bir iş çıkardığını görebiliyoruz:

![Pasted image 20241225111539.png](/img/user/resimler/Pasted%20image%2020241225111539.png)

![Pasted image 20241225111543.png](/img/user/resimler/Pasted%20image%2020241225111543.png)

Ancak kod hâlâ çok kolay okunabilir durumda değil. Bunun sebebi, üzerinde çalıştığımız kodun yalnızca minify edilmemiş, aynı zamanda obfuscate edilmiş olmasıdır. Bu nedenle, kodu yalnızca formatlamak veya güzelleştirmek yeterli olmayacaktır. Bunun için kodu deobfuscate etmek için araçlara ihtiyacımız olacak.

## Deobfuscate

JavaScript kodunu deobfuscate edip anlaşılabilir bir hâle getirmek için birçok kaliteli çevrimiçi araç bulabiliriz. Bunlardan biri **[UnPacker](https://matthewfl.com/unPacker.html)** aracıdır. Yukarıda obfuscate ettiğimiz kodu kopyalayıp, UnPacker'da **UnPack** düğmesine tıklayarak çalıştırmayı deneyelim.

**İpucu:** Script'in öncesinde boş satır bırakmadığınızdan emin olun, aksi takdirde deobfuscate işlemi etkilenebilir ve hatalı sonuçlar alabilirsiniz.

![Pasted image 20241225111839.png](/img/user/resimler/Pasted%20image%2020241225111839.png)


Bu aracın JavaScript kodunu deobfuscating konusunda çok daha iyi bir iş çıkardığını ve bize anlayabileceğimiz bir çıktı verdiğini görebiliriz:

```javascript
function generateSerial() {
  ...SNIP...
  var xhr = new XMLHttpRequest;
  var url = "/serial.php";
  xhr.open("POST", url, true);
  xhr.send(null);
};
```

Daha önce bahsedildiği gibi, yukarıda kullanılan obfuscation yöntemi bir **packing** yöntemidir. Bu tür kodları unpack etmenin bir başka yolu, kodun sonunda bulunan **return** değerini bulup, bu değeri çalıştırmak yerine **console.log** kullanarak yazdırmaktır.

## Reverse Engineering

Bu araçlar, kodu anlaşılabilir bir hale getirme konusunda şu ana kadar iyi bir iş çıkarıyor olsa da, kod daha fazla obfuscation ve encoding işlemine tabi tutulduğunda, otomatik araçların bu kodu temizlemesi çok daha zor hale gelir. Özellikle, kod özel bir obfuscation aracı kullanılarak obfuscate edilmişse bu durum daha da geçerlidir.

Böyle durumlarda, kodun nasıl obfuscate edildiğini ve işlevselliğini anlamak için manuel olarak kodu **tersine mühendislik** ile analiz etmemiz gerekir. Eğer ileri düzeyde JavaScript **Deobfuscation** ve **Tersine Mühendislik** hakkında daha fazla bilgi edinmek isterseniz, bu konuyu detaylı şekilde ele alan **Secure Coding 101** modülüne göz atabilirsiniz.


Soru : Bu bölümde öğrendiklerinizi kullanarak, bayrağın içeriğini elde etmek için 'secret.js' dosyasını deobfuscate etmeye çalışın. Bayrak nedir?

Cevap : `HTB{1_4m_7h3_53r14l_g3n3r470r!}`

Çözüm Chatgpt kullan

### Açıklama

1. **`flag` değişkeni:** Kod, "HTB{1_4m_7h3_53r14l_g3n3r470r!}" şeklinde bir bayrak içeriyor.
2. **`xhr` nesnesi:** Yeni bir `XMLHttpRequest` nesnesi oluşturuluyor.
3. **`url` değişkeni:** İstek "/serial.php" URL'sine yapılıyor.
4. **`xhr.open` ve `xhr.send`:** POST isteği açılıp, boş bir içerik gönderiliyor.

```
function generateSerial() {
var flag = "HTB{1_4m_7h3_53r14l_g3n3r470r!}";
var xhr = new XMLHttpRequest();
var url = "/serial.php";
xhr.open("POST", url, true);
xhr.send(null);
}
```

# Code Analysis

Artık kodu deobfuscate ettiğimize göre, kodu incelemeye başlayabiliriz:

```javascript
'use strict';
function generateSerial() {
  ...SNIP...
  var xhr = new XMLHttpRequest;
  var url = "/serial.php";
  xhr.open("POST", url, true);
  xhr.send(null);
};
```

secret.js dosyasının yalnızca bir fonksiyon içerdiğini görüyoruz: generateSerial.

## HTTP Requests

Şimdi generateSerial fonksiyonunun her bir satırına bakalım.

#### Code Variables

Fonksiyon, önce bir XMLHttpRequest nesnesi oluşturan `xhr` değişkenini tanımlayarak başlar. XMLHttpRequest'in JavaScript'te tam olarak ne yaptığını bilmiyorsak, "XMLHttpRequest" terimini Google'da arayarak ne amaçla kullanıldığını öğrenebiliriz.

Bu konuda araştırma yaptığımızda, XMLHttpRequest'in web isteklerini işleyen bir JavaScript fonksiyonu olduğunu görürüz.


#### Code Functions

İkinci olarak tanımlanan değişken, aynı domain üzerinde olduğu anlaşılan `/serial.php` yolunu içeren bir `URL` değişkenidir. Domain belirtilmediği için bu adresin mevcut domain altında olduğu varsayılabilir.

Sonraki kodda, `xhr.open` fonksiyonunun "POST" ve `URL` parametreleriyle çağrıldığını görürüz. Bu fonksiyonun ne yaptığını öğrenmek için yine Google'da arama yapabiliriz. Araştırma sonucunda, `xhr.open` fonksiyonunun bir HTTP isteğini (GET veya POST) belirli bir URL'ye açtığını, ardından `xhr.send` fonksiyonunun bu isteği gönderdiğini öğreniyoruz.

Bu durumda, `generateSerial` fonksiyonunun yaptığı tek şey, `/serial.php` adresine herhangi bir POST verisi eklemeden ve bir return almadan bir POST isteği göndermek.

Bu fonksiyonun, örneğin, bir "Generate Serial" butonuna tıklandığında bir seri numarası üretmek için kullanılması planlanmış olabilir. Ancak, seri numaraları üreten bir HTML elementine rastlamadığımızdan, bu fonksiyonun henüz kullanılmadığı ve gelecekte kullanılması için geliştiriciler tarafından bırakılmış olduğu anlaşılmaktadır.

**Sonuç ve Analiz**  
Kod deobfuscation ve kod analizi kullanarak bu fonksiyonu açığa çıkarabildik. Şimdi bu fonksiyonun fonksiyonunu çoğaltmayı deneyebilir ve bir POST isteği gönderildiğinde sunucu tarafında nasıl işlendiğini görebiliriz. Eğer bu fonksiyon sunucu tarafından işleniyorsa, kullanılmayan bir fonksiyon keşfedebiliriz. Bu tür fonksiyonlar genellikle hata ve güvenlik açıkları içerme eğilimindedir.


# HTTP Requests

Önceki bölümde, secret.js ana fonksiyonunun /serial.php adresine boş bir POST isteği gönderdiğini öğrendik. Bu bölümde, /serial.php'ye bir POST isteği göndermek için cURL kullanarak aynı şeyi yapmaya çalışacağız.

## POST Request

POST isteği göndermek için, komutumuza -X POST bayrağını eklemeliyiz ve bir POST isteği göndermelidir:

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://SERVER_IP:PORT/ -X POST
```

İpucu: response'u gereksiz verilerle karıştırmamak için “-s” bayrağını ekliyoruz

Ancak POST isteği genellikle POST verisi içerir. Veri göndermek için “`-d ‘param1=sample`’” bayrağını kullanabilir ve aşağıdaki gibi her parametre için verilerimizi ekleyebiliriz:


Soru : Bu bölümde öğrendiklerinizi '/serial.php' adresine bir 'POST' isteği göndererek uygulamayı deneyin. Aldığınız yanıt nedir?

Cevap : N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz

```
$ curl -s -X POST 94.237.54.116:56040/serial.php
N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz
```

# Decoding

Önceki bölümdeki alıştırmayı yaptıktan sonra, kodlanmış gibi görünen garip bir metin bloğu elde ettik:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://SERVER_IP:PORT/serial.php -X POST -d "param1=sample"

ZG8gdGhlIGV4ZXJjaXNlLCBkb24ndCBjb3B5IGFuZCBwYXN0ZSA7KQo=
```

Bu, Daha Fazla Obfuscation (Gelişmiş Obfuscation) bölümünde bahsettiğimiz obfuscation'ın bir başka önemli yönüdür. Birçok teknik, kodu daha da obfuscate ederek insan tarafından okunmasını zorlaştırabilir ve sistemler tarafından tespit edilmesini engelleyebilir. Bu nedenle, obfuscate edilmiş kodlarda sıkça, çalıştırıldığında çözülmek üzere şifrelenmiş metin blokları bulunur. En yaygın kullanılan 3 metin kodlama yöntemi şunlardır:

- base64
- hex
- rot13


## Base64

Base64 kodlama, genellikle özel karakterlerin kullanımını azaltmak için kullanılır, çünkü base64 ile kodlanmış herhangi bir karakter yalnızca alfa-nümerik karakterler ile birlikte + ve / karakterlerini kullanarak temsil edilir. Girdi ne olursa olsun, hatta ikili formatta bile olsa, ortaya çıkan base64 kodlu dize yalnızca bunları kullanır.

#### Base64 Tespit Etme

Base64 kodlu dizeler, yalnızca alfa-nümerik karakterler içerdiği için kolayca tespit edilebilir. Ancak, base64'ün en belirgin özelliği, = karakterleriyle yapılan dolgudur. Base64 kodlu dizelerin uzunluğu 4'ün katları olmalıdır. Örneğin, çıkan çıktı sadece 3 karakter uzunluğunda ise, dolgu olarak ekstra bir = eklenir ve bu şekilde devam eder.


#### Base64 Encode

Linux'ta herhangi bir metni base64'e kodlamak için, metni eko edebilir ve '|' ile base64'e borulayabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo https://www.hackthebox.eu/ | base64

aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K
```

#### Base64 Decode

Herhangi bir base64 kodlu stringin kodunu çözmek istiyorsak base64 -d komutunu aşağıdaki gibi kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo aHR0cHM6Ly93d3cuaGFja3RoZWJveC5ldS8K | base64 -d

https://www.hackthebox.eu/
```

## Hex

Diğer bir yaygın kodlama yöntemi de her karakteri ASCII tablosundaki hex sırasına göre kodlayan hex kodlamadır. Örneğin, a hex'te 61'dir, b 62'dir, c 63'tür ve bu böyle devam eder. ASCII tablosunun tamamını Linux'ta man ascii komutunu kullanarak bulabilirsiniz.


#### Spotting Hex

Onaltılı olarak kodlanmış herhangi bir dize yalnızca onaltılı karakterlerden oluşur ve bunlar yalnızca 16 karakterdir: 0-9 ve a-f. Bu, onaltılı kodlanmış dizeleri tespit etmeyi base64 kodlu dizeleri tespit etmek kadar kolay hale getirir.


#### Hex Encode

Linux'ta herhangi bir dizeyi hex olarak kodlamak için xxd -p komutunu kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo https://www.hackthebox.eu/ | xxd -p

68747470733a2f2f7777772e6861636b746865626f782e65752f0a
```

#### Hex Decode

Hex kodlu bir dizenin kodunu çözmek için xxd -p -r komutunu kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo 68747470733a2f2f7777772e6861636b746865626f782e65752f0a | xxd -p -r

https://www.hackthebox.eu/
```


## Caesar/Rot13

Bir diğer yaygın -ve çok eski- kodlama tekniği de her harfi sabit bir sayı kadar kaydıran Sezar şifresidir. Örneğin, 1 karakter kaydırma a'yı b yapar ve b c olur ve bu böyle devam eder. Sezar şifresinin birçok varyasyonu farklı sayıda kaydırma kullanır, bunlardan en yaygın olanı her karakteri 13 kez ileri kaydıran rot13'tür.

#### Spotting Caesar/Rot13

Bu kodlama yöntemi herhangi bir metnin rastgele görünmesine neden olsa da, her karakter belirli bir karakterle eşleştirildiği için onu tespit etmek yine de mümkündür. Örneğin, rot13'te http://www uggc://jjj haline gelir, bu da hala bazı benzerlikler taşır ve bu şekilde tanınabilir.


#### Rot13 Encode

Linux'ta rot13 kodlaması yapmak için özel bir komut yoktur. Ancak, karakter kaydırma işlemini yapmak için kendi komutumuzu oluşturmak oldukça kolaydır:

```shell-session
M1R4CKCK@htb[/htb]$ echo https://www.hackthebox.eu/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'

uggcf://jjj.unpxgurobk.rh/
```

#### Rot13 Decode

Rot13'ün kodunu çözmek için de aynı önceki komutu kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ echo uggcf://jjj.unpxgurobk.rh/ | tr 'A-Za-z' 'N-ZA-Mn-za-m'

https://www.hackthebox.eu/
```

Rot13'ü kodlamak/kodunu çözmek için başka bir seçenek de [rot13](https://rot13.com/) gibi çevrimiçi bir araç kullanmak olabilir.


### Diğer Kodlama Türleri

İnternette bulabileceğimiz yüzlerce başka kodlama yöntemi vardır. Bunlar en yaygın olanları olsa da, bazen tanımlamak ve çözmek için biraz deneyim gerektirebilecek başka kodlama yöntemleriyle de karşılaşabiliriz.

Benzer kodlama türleriyle karşılaşırsanız, önce kodlama türünü belirlemeye çalışın ve ardından bunu çözmek için çevrimiçi araçlar arayın.

[Cipher Identifier](https://www.boxentriq.com/code-breaking/cipher-identifier) gibi bazı araçlar kodlama türünü otomatik olarak belirlememize yardımcı olabilir. Kodlama yöntemini doğru bir şekilde belirleyip belirleyemediğini görmek için yukarıdaki kodlanmış stringleri Cipher Identifier ile deneyin.

Kodlamanın yanı sıra, birçok obfuscation aracı, bir anahtar kullanarak bir stringi kodlayan şifrelemeyi kullanır, bu da özellikle şifre çözme anahtarı kodun içinde saklanmıyorsa, obfuscated kodun tersine mühendisliğini ve deobfuscate edilmesini çok zorlaştırabilir.


Soru :  Bu bölümde öğrendiklerinizi kullanarak, önceki alıştırmada aldığınız string'de kullanılan kodlama türünü belirleyin ve kodunu çözün. Bayrağı almak için, 'serial.php' adresine bir 'POST' isteği gönderebilir ve veriyi “serial=YOUR_DECODED_OUTPUT” olarak ayarlayabilirsiniz.

Cevap : 

```
$ curl 94.237.54.116:30359/serial.php -X POST
N2gxNV8xNV9hX3MzY3IzN19tMzU1NGcz

Base64 decode yap 

$ curl 94.237.54.116:30359/serial.php -X POST -d "serial=7h15_15_a_s3cr37_m3554g3"

```


# Skills Assessment

Penetrasyon Testimiz sırasında JavaScript ve API'ler içeren bir web sunucusuyla karşılaştık. Müşterimizi nasıl olumsuz etkileyebileceğini anlamak için fonksiyonelliklerini belirlememiz gerekiyor.

Soru : Web sayfasının HTML kodunu incelemeye çalışın ve içinde kullanılan JavaScript kodunu belirleyin. Kullanılan JavaScript dosyasının adı nedir?

Cevap : ==api.min.js==

![Pasted image 20241225140448.png](/img/user/resimler/Pasted%20image%2020241225140448.png)


Soru : JavaScript kodunu bulduğunuzda, ilginç işlevler yapıp yapmadığını görmek için çalıştırmayı deneyin. Karşılığında bir şey aldınız mı?

Cevap : 

![Pasted image 20241225140605.png](/img/user/resimler/Pasted%20image%2020241225140605.png)


Soru : Fark etmiş olabileceğiniz gibi, JavaScript kodu gizlenmiştir. Kodu deobfuscate etmek ve 'flag' değişkenini almak için bu modülde öğrendiğiniz becerileri uygulamayı deneyin.

Cevap : 

![Pasted image 20241225143037.png](/img/user/resimler/Pasted%20image%2020241225143037.png)


```
function apiKeys() {

var flag = 'HTB{n3v3r_run_0bfu5c473d_c0d3!}';

var xhr = new XMLHttpRequest();

var _0x437f8b = '/keys.php';

xhr.open('POST', _0x437f8b, true);

xhr.send(null);

}

console.log('HTB{j4v45c1r1p7_3num3r4710n_15_k3y}');
```

[[Bağlantılar/Kod açıklaması 312357858\|Kod açıklaması 312357858]]

Soru : İlk olarak gizli anahtarı elde ettiğinizde, bunun hangi kodlama yöntemiyle kodlandığını belirlemeye çalışın ve ardından çözün. Ardından, çözülmüş anahtarı 'key=DECODED_KEY' olarak aynı önceki sayfaya bir 'POST' isteği gönderin. Elde ettiğiniz bayrak nedir?

Cevap : 

![Pasted image 20241225143247.png](/img/user/resimler/Pasted%20image%2020241225143247.png)


![Pasted image 20241225143350.png](/img/user/resimler/Pasted%20image%2020241225143350.png)

![Pasted image 20241225143431.png](/img/user/resimler/Pasted%20image%2020241225143431.png)


Soru : Private anahtarı aldıktan sonra, kodlama yöntemine karar vermeye çalışın ve kodunu çözün. Daha sonra aynı önceki sayfaya “key=DECODED_KEY” olarak kodu çözülmüş anahtar ile bir “POST” isteği gönderin. Aldığınız bayrak nedir?

Cevap : 

![Pasted image 20241225143608.png](/img/user/resimler/Pasted%20image%2020241225143608.png)
