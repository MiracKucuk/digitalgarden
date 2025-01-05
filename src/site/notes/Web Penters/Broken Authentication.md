---
{"dg-publish":true,"permalink":"/web-penters/broken-authentication/"}
---


### What is Authentication

==Kimlik doğrulama (Authentication)==, RFC 4949'da “Bir sistem varlığının veya sistem kaynağının belirli bir öznitelik değerine sahip olduğu iddiasını doğrulama süreci” olarak tanımlanmaktadır

Öte yandan, ==yetkilendirme (authorization)==  “bir sistem kaynağına erişmesi için bir sistem varlığına verilen onaydır”;


### Kimlik Doğrulama (Authentication)

- **Kullanıcıların iddia ettikleri kişi olup olmadığını belirler.**
- **Kullanıcıdan kimlik doğrulamak için kimlik bilgilerini ister.** (Örneğin, şifreler, güvenlik sorularına yanıtlar veya yüz tanıma)
- **Genellikle yetkilendirmeden önce yapılır.**
- **Kullanıcının giriş bilgilerini gerektirir.**
- **Genellikle bilgileri bir Kimlik Jetonu (ID Token) aracılığıyla iletir.**

### Yetkilendirme (Authorization)

- **Kullanıcıların neleri yapabileceğini veya erişemeyeceğini belirler.**
- **Erişim izninin politikalara ve kurallara göre doğrulanmasını sağlar.**
- **Genellikle başarılı bir kimlik doğrulamadan sonra yapılır.**
- **Kullanıcının yetki seviyelerini veya güvenlik düzeylerini gerektirir.**
- **Genellikle bilgileri bir Erişim Jetonu (Access Token) aracılığıyla iletir.**

Web uygulamalarındaki en yaygın kimlik doğrulama yöntemi, kullanıcıların kimliklerini kanıtlamak için kullanıcı adı ve şifrelerini girdikleri oturum açma formlarıdır.

![Pasted image 20241208210833.png](/img/user/resimler/Pasted%20image%2020241208210833.png)


### Yaygın Kimlik Doğrulama Yöntemleri

Tipik olarak, bunlar aşağıdaki üç ana kategoriye ayrılabilir:

Knowledge-based authentication (Bilgi Tabanlı)
Ownership-based authentication (Sahiplik tabanlı )
Inherence-based authentication


### Knowledge
Bilgi faktörlerine dayalı kimlik doğrulama, kullanıcının kimliğini kanıtlamak için bildiği bir şeye dayanır. Kullanıcı parolalar, parolalar, PIN'ler veya güvenlik sorularının yanıtları gibi bilgileri sağlar.


### Ownership
Sahiplik faktörlerine dayalı kimlik doğrulama, kullanıcının sahip olduğu bir şeye dayanır.


### Inherence
Son olarak, kalıtım faktörlerine dayalı kimlik doğrulama, kullanıcının olduğu veya yaptığı bir şeye dayanır. Bu, parmak izi, yüz desenleri ve ses tanıma veya imzalar gibi biyometrik faktörleri içerir.


![Pasted image 20241208224229.png](/img/user/resimler/Pasted%20image%2020241208224229.png)


### Single-Factor Authentication vs Multi-Factor Authentication

Tek faktörlü kimlik doğrulama yalnızca tek bir yönteme dayanır. Örneğin, parola kimlik doğrulaması yalnızca parolanın bilinmesine dayanır. Bu nedenle, tek faktörlü bir kimlik doğrulama yöntemidir.

Öte yandan, çok faktörlü kimlik doğrulama (MFA) birden fazla kimlik doğrulama yöntemi içerir. Örneğin, bir web uygulaması bir parola ve zamana dayalı tek seferlik parola (TOTP) gerektiriyorsa, kimlik doğrulama için parola bilgisine ve TOTP cihazının sahipliğine dayanır. Tam olarak iki faktörün gerekli olduğu özel durumda, MFA genellikle 2 faktörlü kimlik doğrulama (2FA) olarak adlandırılır.



### Enumerating Users
Kullanıcı numaralandırma güvenlik açıkları, bir web uygulaması kimlik doğrulama endpointleri için kayıtlı/geçerli ve geçersiz girdilere farklı yanıt verdiğinde ortaya çıkar. Kullanıcı numaralandırma güvenlik açıkları sıklıkla kullanıcı girişi, kullanıcı kaydı ve parola sıfırlama gibi kullanıcının kullanıcı adına dayalı işlevlerde ortaya çıkar.

Web geliştiricileri, kullanıcı adları gibi bilgilerin gizli olmadığını varsayarak kullanıcı numaralandırma vektörlerini sıklıkla göz ardı etmektedir. Ancak, kullanıcı adları web uygulamalarında kimlik doğrulama için gereken birincil tanımlayıcı ise gizli olarak kabul edilebilir. Dahası, kullanıcılar web uygulamaları dışında FTP, RDP ve SSH gibi çeşitli hizmetlerde de aynı kullanıcı adını kullanma eğilimindedir. Birçok web uygulaması kullanıcı adlarını tanımlamamıza izin verdiğinden, geçerli kullanıcı adlarını listeleyebilir ve bunları kimlik doğrulamaya yönelik başka saldırılar için kullanabiliriz. Bu genellikle mümkündür çünkü web uygulamaları genellikle bir kullanıcı adını veya kullanıcının e-posta adresini kullanıcıların birincil tanımlayıcısı olarak kabul eder.


## User Enumeration Theory

Kullanıcı adı numaralandırma saldırılarına karşı korumanın kullanıcı deneyimi üzerinde etkisi olabilir. Bir kullanıcı adının var olup olmadığını gösteren bir web uygulaması, meşru bir kullanıcının kullanıcı adını doğru yazamadığını tespit etmesine yardımcı olabilir. Yine de aynı durum geçerli kullanıcı adlarını belirlemeye çalışan bir saldırgan için de geçerlidir. WordPress gibi iyi bilinen ve olgun uygulamalar bile varsayılan olarak kullanıcı numaralandırmasına izin verir. Örneğin, WordPress'e geçersiz bir kullanıcı adıyla giriş yapmaya çalışırsak aşağıdaki hata mesajını alırız:

![Pasted image 20241208224815.png](/img/user/resimler/Pasted%20image%2020241208224815.png)


Öte yandan, geçerli bir kullanıcı adı farklı bir hata mesajıyla sonuçlanır:

![Pasted image 20241208224828.png](/img/user/resimler/Pasted%20image%2020241208224828.png)

Gördüğümüz gibi, kullanıcı numaralandırma, bir web uygulamasının bir servis sağlamak için kasıtlı olarak kabul ettiği bir güvenlik riski olabilir. Başka bir örnek olarak, kullanıcıların başkalarıyla sohbet etmesini sağlayan bir sohbet uygulaması düşünün. Bu uygulama, kullanıcıları kullanıcı adlarına göre aramak için bir işlevsellik sağlayabilir. Bu işlev platformdaki tüm kullanıcıları listelemek için kullanılabilirken, aynı zamanda web uygulaması tarafından sağlanan hizmet için de gereklidir. Bu nedenle, kullanıcı numaralandırma her zaman bir güvenlik açığı değildir. Bununla birlikte, mümkünse derinlemesine savunma önlemi olarak bundan kaçınılmalıdır. Örneğin, örneğimizdeki web uygulamasında, oturum açma sırasında kullanıcı adı yerine bir e-posta adresi kullanılarak kullanıcı numaralandırmadan kaçınılabilir.


### Kullanıcıları Farklı Hata Mesajları ile Numaralandırma

Geçerli kullanıcıların bir listesini elde etmek için, bir saldırgan genellikle test etmek için kullanıcı adlarından oluşan bir kelime listesine ihtiyaç duyar. Kullanıcı adları genellikle parolalardan çok daha az karmaşıktır. E-posta adresi olmadıklarında nadiren özel karakterler içerirler. Ortak kullanıcılardan oluşan bir liste, bir saldırganın brute force saldırısının kapsamını daraltmasına veya destek çalışanlarına veya kullanıcılara karşı hedefli saldırılar (OSINT'ten yararlanarak) gerçekleştirmesine olanak tanır. Ayrıca, ortak bir parola geçerli hesaplara karşı kolayca püskürtülebilir ve genellikle hesabın başarılı bir şekilde ele geçirilmesine yol açar. Kullanıcı adlarını toplamanın diğer yolları, bir web uygulamasını taramak veya sosyal ağlardaki şirket profilleri gibi herkese açık bilgileri kullanmaktır. İyi bir başlangıç noktası [SecLists](https://github.com/danielmiessler/SecLists/tree/master/Usernames) kelime listesi koleksiyonudur. 

Laboratuvara abc gibi geçersiz bir kullanıcı adıyla giriş yapmaya çalıştığımızda aşağıdaki hata mesajını görebiliriz:

![Pasted image 20241208225030.png](/img/user/resimler/Pasted%20image%2020241208225030.png)

Öte yandan, htb-stdnt gibi kayıtlı bir kullanıcı ve geçersiz bir parola ile giriş yapmaya çalıştığımızda, farklı bir hata görebiliriz:

![Pasted image 20241208225052.png](/img/user/resimler/Pasted%20image%2020241208225052.png)

Dönen hata mesajlarındaki bu farktan yararlanalım ve SecLists'in xato-net-10-million-usernames.txt kelime listesini ffuf ile geçerli kullanıcıları numaralandırmak için kullanalım. Kelime listesini -w parametresiyle, POST verilerini -d parametresiyle ve geçerli kullanıcıları fuzzlamak için kullanıcı adında FUZZ anahtar kelimesini belirtebiliriz. Son olarak, Unknown user dizesini içeren yanıtları kaldırarak geçersiz kullanıcıları filtreleyebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://172.17.0.2/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user"

<SNIP>

[Status: 200, Size: 3271, Words: 754, Lines: 103, Duration: 310ms]
    * FUZZ: consuelo
```


Geçerli kullanıcı adı consuelo'yu başarıyla tespit ettik. 

### User Enumeration via Side-Channel Attacks
Web uygulamasının yanıtındaki farklılıklar geçerli kullanıcı adlarını listelemenin en basit ve en belirgin yolu olsa da, geçerli kullanıcı adlarını side channel (yan kanal) yoluyla da listeleyebiliriz. Side-channel saldırıları doğrudan web uygulamasının yanıtını değil, yanıttan elde edilebilecek veya çıkarılabilecek ekstra bilgileri hedef alır. Bir Side-channel  örneği, yanıt zamanlamasıdır, yani web uygulamasının yanıtının bize ulaşması için geçen süredir. Bir web uygulamasının yalnızca geçerli kullanıcı adları için veritabanı araması yaptığını varsayalım. Bu durumda, yanıt aynı olsa bile, yanıt süresindeki bir farkı ölçebilir ve geçerli kullanıcı adlarını bu şekilde listeleyebiliriz. Yanıt zamanlamasına dayalı kullanıcı numaralandırma Whitebox Attacks modülünde ele alınmıştır.


Soru : Web uygulamasında geçerli bir kullanıcıyı numaralandırın. Cevap olarak kullanıcı adını verin.

`ffuf -w /opt/useful/seclists/Usernames/xato-net-10-million-usernames.txt -u http://94.237.61.84:36274/index.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "username=FUZZ&password=invalid" -fr "Unknown user" -ic`

![Pasted image 20241209012712.png](/img/user/resimler/Pasted%20image%2020241209012712.png)
