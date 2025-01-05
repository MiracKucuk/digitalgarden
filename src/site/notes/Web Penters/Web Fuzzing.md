---
{"dg-publish":true,"permalink":"/web-penters/web-fuzzing/"}
---



### Why Fuzz Web Applications?

- **Gizli Güvenlik Açıkları Tespit Edilir:** Fuzzing, beklenmeyen ve geçersiz girdilerle uygulamayı test ederek kodda gizli hataları ortaya çıkarır.
- **Güvenlik Testleri Otomatize Edilir:** Otomatik test girdileri oluşturarak zaman ve kaynak tasarrufu sağlar.
- **Gerçek Saldırılar Simüle Edilir:** Saldırgan tekniklerini taklit ederek açıkları önceden tespit eder.
- **Girdi Doğrulama Güçlendirilir:** SQL Injection ve XSS gibi açıkları önlemek için doğrulama zayıflıklarını belirler.
- **Kod Kalitesi Artar:** Hataları ortaya çıkararak geliştiricilerin daha sağlam ve güvenli kod yazmasını sağlar.
- **Sürekli Güvenlik Sağlanır:** CI/CD süreçlerine entegre edilerek düzenli testlerle açıklar erken tespit edilir.


|**Kavram**|**Açıklama**|**Örnek**|
|---|---|---|
|**Wordlist**|Fuzzing sırasında giriş olarak kullanılan, kelimeler, ifadeler, dosya adları, dizin adları veya parametre değerlerinden oluşan bir liste.|**Genel:** admin, login, password, backup, config **Uygulamaya özel:** productID, addToCart, checkout|
|**Payload**|Fuzzing sırasında web uygulamasına gönderilen veri. Basit bir string, sayısal değer veya karmaşık bir veri yapısı olabilir.|`' OR 1=1 --` (SQL injection için)|
|**Response Analysis**|Fuzzing sırasında web uygulamasının yanıtlarını (örneğin, yanıt kodları, hata mesajları) inceleyerek güvenlik açıklarını gösterebilecek anormallikleri tespit etme.|**Normal:** 200 OK **Hata (potansiyel SQLi):** 500 Internal Server Error ve veritabanı hata mesajı|
|**Fuzzer**|Payload'ları otomatik olarak oluşturup gönderen ve web uygulamasının yanıtlarını analiz eden bir yazılım aracı.|**Örnek araçlar:** ffuf, wfuzz, Burp Suite Intruder|
|**False Positive**|Fuzzer tarafından yanlışlıkla güvenlik açığı olarak tanımlanan bir sonuç.|Var olmayan bir dizin için **404 Not Found** hatası.|
|**False Negative**|Web uygulamasında mevcut olan ancak fuzzer tarafından tespit edilemeyen bir güvenlik açığı.|Ödeme işleme fonksiyonundaki ince bir mantık hatası.|
|**Fuzzing Scope**|Fuzzing işlemlerinde hedef alınan web uygulamasının belirli bölümleri.|Sadece giriş sayfasını veya belirli bir API endpoint'ini hedeflemek.|



# Tooling

Bu modülde, web uygulaması keşfi ve güvenlik açığı değerlendirmesi için tasarlanmış dört güçlü araç kullanacağız. 


### Installing Go, Python and PIPX

Bu araçlar için Go ve Python'un kurulu olması gerekir. Henüz yüklemediyseniz bunları aşağıdaki gibi yükleyin.

pipx, Python uygulamalarının kurulumunu ve yönetimini basitleştirmek için tasarlanmış bir komut satırı aracıdır. Her uygulama için izole sanal ortamlar oluşturarak süreci kolaylaştırır ve bağımlılıkların çakışmamasını sağlar. Bu, uyumluluk sorunları hakkında endişelenmeden birden fazla Python uygulaması yükleyebileceğiniz ve çalıştırabileceğiniz anlamına gelir. pipx ayrıca uygulamaları yükseltmeyi veya kaldırmayı kolaylaştırarak sisteminizi düzenli ve dağınıklıktan uzak tutar.

Debian tabanlı bir sistem kullanıyorsanız (Ubuntu gibi), APT paket yöneticisini kullanarak Go, Python ve PIPX'i yükleyebilirsiniz.

1. Bir terminal açın ve paketlerin en yeni sürümleri ve bağımlılıkları hakkında en son bilgilere sahip olduğunuzdan emin olmak için paket listelerinizi güncelleyin.


```shell-session
M1R4CKCK@htb[/htb]$ sudo apt update
```


2. Go'yu yüklemek için aşağıdaki komutu kullanın:

```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install -y golang
```


3. Python'u yüklemek için aşağıdaki komutu kullanın:

```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install -y python3 python3-pip
```


4. Pipx'i kurmak ve yapılandırmak için aşağıdaki komutu kullanın:


```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install pipx
M1R4CKCK@htb[/htb]$ pipx ensurepath
M1R4CKCK@htb[/htb]$ sudo pipx ensurepath --global
```

5. Go ve Python'un doğru yüklendiğinden emin olmak için sürümlerini kontrol edebilirsiniz:

```shell-session
M1R4CKCK@htb[/htb]$ go version
M1R4CKCK@htb[/htb]$ python3 --version
```


Kurulumlar başarılı olduysa, hem Go hem de Python için sürüm bilgilerini görmelisiniz.


## FFUF

FFUF (Fuzz Faster U Fool) Go dilinde yazılmış hızlı bir web fuzzeridir. Web uygulamalarındaki dizinleri, dosyaları ve parametreleri hızlı bir şekilde numaralandırmada mükemmeldir. Esnekliği, hızı ve kullanım kolaylığı onu güvenlik uzmanları ve meraklıları arasında favori yapmaktadır.

FFUF'u aşağıdaki komutu kullanarak yükleyebilirsiniz:

```shell-session
M1R4CKCK@htb[/htb]$ go install github.com/ffuf/ffuf/v2@latest
```


- **Dizin ve Dosya Taraması**
- **Parametre Keşfi**
- **Brute-Force Saldırısı**


## Gobuster

Gobuster bir başka popüler web dizini ve dosya fuzzeridir. Hızı ve basitliği ile bilinir, bu da onu hem yeni başlayanlar hem de deneyimli kullanıcılar için mükemmel bir seçim haline getirir.

GoBuster'ı aşağıdaki komutu kullanarak kurabilirsiniz:

```shell-session
M1R4CKCK@htb[/htb]$ go install github.com/OJ/gobuster/v3@latest
```

- **İçerik Keşfi**
- **DNS Subdomain Taraması**
- **WordPress İçerik Algılama**


## FeroxBuster

FeroxBuster, Rust ile yazılmış hızlı, rekürsif bir içerik keşif aracıdır. 

```shell-session
M1R4CKCK@htb[/htb]$ curl -sL https://raw.githubusercontent.com/epi052/feroxbuster/main/install-nix.sh | sudo bash -s $HOME/.local/bin
```

- **Recursive Tarama**
- **Bağlantısız İçerik Keşfi**
- **Yüksek Performanslı Taramalar**


## wfuzz/wenum

wenum, esnekliği ve özelleştirme seçenekleriyle bilinen çok yönlü ve güçlü bir komut satırı fuzzing aracı olan wfuzz'un aktif olarak sürdürülen bir forkudur. Özellikle parametre fuzzing için çok uygundur, web uygulamalarına karşı çok çeşitli girdi değerlerini test etmenize ve bu parametreleri nasıl işlediklerine dair olası güvenlik açıklarını ortaya çıkarmanıza olanak tanır.

PwnBox veya Kali gibi bir sızma testi Linux dağıtımı kullanıyorsanız, wfuzz zaten önceden yüklenmiş olabilir, bu da isterseniz hemen kullanmanıza izin verir. Bununla birlikte, wfuzz'ı kurarken şu anda bazı zorluklar yaşanmaktadır, bu nedenle wenum ile değiştirebilirsiniz. Komutlar birbirinin yerine kullanılabilir ve aynı sözdizimini takip ederler, bu nedenle gerekirse wenum komutlarını wfuzz ile değiştirebilirsiniz.

Aşağıdaki komutlar, wenum'u yüklemek için izole ortamlarda Python uygulamalarını yüklemek ve yönetmek için bir araç olan pipx'i kullanacaktır. Bu, wenum için temiz ve tutarlı bir ortam sağlar ve olası paket çakışmalarını önler:

```shell-session
M1R4CKCK@htb[/htb]$ pipx install git+https://github.com/WebFuzzForge/wenum
M1R4CKCK@htb[/htb]$ pipx runpip wenum install setuptools
```


* Directory and File Enumeration
* Parameter Discovery
* Brute-Force Attack


# Directory and File Fuzzing

Bahsettiğimiz araçlar - ffuf, wfuzz, vb - yerleşik kelime listelerine sahip değildir, ancak harici kelime listesi dosyalarıyla sorunsuz bir şekilde çalışmak üzere tasarlanmıştır. 

https://github.com/danielmiessler/SecLists

SecLists contains wordlists for:

* Common directory and file names
* Backup files
* Configuration files
* Vulnerable scripts
* And much more

- **common.txt:** Genel amaçlı bir wordlist; yaygın dizin ve dosya adlarını içerir, başlangıç için idealdir.
- **directory-list-2.3-medium.txt:** Dizin adlarına odaklanan daha kapsamlı bir wordlist. Derinlemesine taramalar için uygundur.
- **raft-large-directories.txt:** Çeşitli kaynaklardan derlenen geniş bir dizin adı koleksiyonu; detaylı taramalar için değerlidir.
- **big.txt:** Dizin ve dosya adlarını içeren büyük bir wordlist; kapsamlı taramalar için kullanışlıdır.

Örneğin, dizinler için fuzz yapmak istiyorsanız, aşağıdaki gibi bir URL kullanabilirsiniz:

```http
http://localhost/FUZZ
```

ffuf, FUZZ yerine seçtiğiniz kelime listesinden “admin”, “backup”, “uploads” vb. kelimeleri koyacak ve ardından http://localhost/admin, http://localhost/backup vb. adreslere istek gönderecektir.


### Directory Fuzzing

İlk adım, web sunucusundaki gizli dizinleri keşfetmemize yardımcı olan dizin fuzzing işlemini gerçekleştirmektir. İşte kullanacağımız ffuf komutu:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -u http://IP:PORT/FUZZ


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://IP:PORT/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-399
________________________________________________

[...]

w2ksvrus                [Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
:: Progress: [220559/220559] :: Job [1/1] :: 100000 req/sec :: Duration: [0:00:03] :: Errors: 0 ::
```

Yukarıdaki çıktı, ffuf'un 301 (Moved Permanently) durum koduyla gösterildiği gibi hedef web sunucusunda w2ksvrus adlı bir dizin bulduğunu göstermektedir. Bu, daha fazla araştırma için potansiyel bir giriş noktası olabilir.


### File Fuzzing
 
- **php:** Sunucu taraflı popüler bir betik dili olan PHP kodlarını içeren dosyalar.
- **.html:** Web sayfalarının yapısını ve içeriğini tanımlayan dosyalar.
- **.txt:** Genellikle basit bilgiler veya loglar tutan düz metin dosyaları.
- **.bak:** Hatalar veya değişiklikler durumunda önceki dosya sürümlerini korumak için oluşturulan yedek dosyalar.
- **.js:** Web sayfalarına etkileşim ve dinamik işlevsellik ekleyen JavaScript kodlarını içeren dosyalar.

Örneğin, web sitesi PHP kullanıyorsa, config.php.bak gibi bir yedekleme dosyasını keşfetmek, veritabanı kimlik bilgileri veya API anahtarları gibi hassas bilgileri ortaya çıkarabilir. Benzer şekilde, test.php gibi eski veya kullanılmayan bir script'in bulunması, saldırganların yararlanabileceği güvenlik açıklarını ortaya çıkarabilir.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/common.txt -u http://IP:PORT/w2ksvrus/FUZZ.html -e .php,.html,.txt,.bak,.js -v 


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://IP:PORT/w2ksvrus/FUZZ.html
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Extensions       : .php .html .txt .bak .js 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 200, Size: 111, Words: 2, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/w2ksvrus/dblclk.html
    * FUZZ: dblclk

[Status: 200, Size: 112, Words: 6, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/w2ksvrus/index.html
    * FUZZ: index

:: Progress: [28362/28362] :: Job [1/1] :: 0 req/sec :: Duration: [0:00:00] :: Errors: 0 ::
```

**`-e .php,.html,.txt,.bak,.js`**: Belirtilen uzantılarla (örneğin `.php`, `.html`) ek dosya taraması yapılmasını sağlar.

ffuf çıktısı /w2ksvrus dizini içinde iki dosya bulduğunu gösteriyor:

dblclk.html: Bu dosya 111 bayt boyutunda olup 2 kelime ve 2 satırdan oluşmaktadır. Amacı hemen anlaşılmayabilir, ancak daha fazla araştırma için potansiyel bir ilgi noktasıdır. Belki de gizli içerik veya fonksiyonellik içeriyordur.

index.html: Bu dosya 112 bayt ile biraz daha büyüktür ve 6 kelime ve 2 satır içerir. Muhtemelen w2ksvrus dizini için varsayılan dizin sayfasıdır.


Soru : Hedef sistemdeki “webfuzzing_hidden_path” yolu içinde (yani http://IP:PORT/webfuzzing_hidden_path/), bayrağı bulmak için klasörler ve ardından dosyalar için fuzz yapın.

Cevap : 

1- 

![Pasted image 20241220113342.png](/img/user/resimler/Pasted%20image%2020241220113342.png)


2- 

![Pasted image 20241220113405.png](/img/user/resimler/Pasted%20image%2020241220113405.png)

![Pasted image 20241220113448.png](/img/user/resimler/Pasted%20image%2020241220113448.png)

![Pasted image 20241220113520.png](/img/user/resimler/Pasted%20image%2020241220113520.png)



# Recursive Fuzzing

```shell-session
[!bash!]$ ffuf -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -ic -v -u http://IP:PORT/FUZZ -e .html -recursion 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : GET
 :: URL              : http://IP:PORT/FUZZ
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt
 :: Extensions       : .html 
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
| URL | http://IP:PORT/level1
| --> | /level1/
    * FUZZ: level1

[INFO] Adding a new job to the queue: http://IP:PORT/level1/FUZZ

[INFO] Starting queued job on target: http://IP:PORT/level1/FUZZ

[Status: 200, Size: 96, Words: 6, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/level1/index.html
    * FUZZ: index.html

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
| URL | http://IP:PORT/level1/level2
| --> | /level1/level2/
    * FUZZ: level2

[INFO] Adding a new job to the queue: http://IP:PORT/level1/level2/FUZZ

[Status: 301, Size: 0, Words: 1, Lines: 1, Duration: 0ms]
| URL | http://IP:PORT/level1/level3
| --> | /level1/level3/
    * FUZZ: level3

[INFO] Adding a new job to the queue: http://IP:PORT/level1/level3/FUZZ

[INFO] Starting queued job on target: http://IP:PORT/level1/level2/FUZZ

[Status: 200, Size: 96, Words: 6, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/level1/level2/index.html
    * FUZZ: index.html

[INFO] Starting queued job on target: http://IP:PORT/level1/level3/FUZZ

[Status: 200, Size: 126, Words: 8, Lines: 2, Duration: 0ms]
| URL | http://IP:PORT/level1/level3/index.html
    * FUZZ: index.html

:: Progress: [441088/441088] :: Job [4/4] :: 100000 req/sec :: Duration: [0:00:06] :: Errors: 0 ::
```

- **`-e .html`**: Fuzzing sırasında `.html` uzantısını ekler.
- **`-recursion`**: Yinelemeli fuzzing yapar, yani bulunan dizinlere dair daha derin taramalar gerçekleştirir.


* `-recursion-depth`: Bu bayrak, recursive keşif için maksimum bir derinlik belirlemenizi sağlar. Örneğin, -recursion-depth 2, fuzzing'i iki seviye derinlikle sınırlar (başlangıç dizini ve hemen alt dizinleri).
* `-rate`: ffuf'un saniye başına istek gönderme hızını kontrol ederek sunucunun aşırı yüklenmesini önleyebilirsiniz.
* `-timeout`: Bu seçenek, tek tek istekler için zaman aşımını ayarlayarak fuzzer'ın yanıt vermeyen hedeflerde takılmasını önlemeye yardımcı olur.


Soru : Bayrağı bulmak için hedef sistemdeki (yani http://IP:PORT/recursive_fuzz/) “recursive_fuzz” yolunu Recursively olarak fuzzlayın.

Cevap : 

1- 

![Pasted image 20241220130932.png](/img/user/resimler/Pasted%20image%2020241220130932.png)

2- 

![Pasted image 20241220130957.png](/img/user/resimler/Pasted%20image%2020241220130957.png)


# Parameter and Value Fuzzing

## GET Parameters: Openly Sharing Information

GET parametrelerini genellikle URL'de bir soru işaretinin (?) hemen ardından görürsünüz. Birden fazla parametre ve işareti (&) kullanılarak bir araya getirilir. Örneğin:

```http
https://example.com/search?query=fuzzing&category=security
```

Bu URL'de:

* sorgu, “fuzzing” değerine sahip bir parametredir
* category “security” değerine sahip başka bir parametredir



## POST Parameters: Behind-the-Scenes Communication

1. ==data Toplama==: Form alanlarına girilen bilgiler toplanır ve aktarım için hazırlanır.

2. ==Encoding (Kodlama)==: Bu veriler, tipik olarak application/x-www-form-urlencoded veya multipart/form-data gibi belirli bir formatta kodlanır:

* `application/x-www-form-urlencoded`: Bu format verileri, GET parametrelerine benzer şekilde, ancak URL yerine request body içine yerleştirilen, ve işareti (&) ile ayrılmış key-value çiftleri olarak kodlar.
* `multipart/form-data:` Bu format, dosyaları diğer verilerle birlikte gönderirken kullanılır. request body'yi, her biri belirli bir veri parçası veya bir dosya içeren birden fazla parçaya böler.

3. ==HTTP Request:== Kodlanmış veriler bir HTTP POST isteğinin body'sinde yer alır ve web sunucusuna gönderilir.

4. ==Server-Side Processing==: Sunucu POST isteğini alır, verilerin kodunu çözer ve uygulamanın mantığına göre işler.

İşte bir oturum açma formu gönderirken POST isteğinin nasıl görünebileceğine dair basitleştirilmiş bir örnek:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded

username=your_username&password=your_password
```

* POST: HTTP yöntemini (POST) belirtir.
* /login: Form verilerinin gönderildiği URL yolunu belirtir.
* Content-Type: Request Body'deki verilerin nasıl kodlandığını belirtir (bu durumda application/x-www-form-urlencoded).
* Request Body : Kodlanmış form verilerini anahtar-değer çiftleri (kullanıcı adı ve parola) olarak içerir.


Parametreler, bir web uygulamasıyla etkileşim kurabileceğiniz ağ geçitleridir. Değerlerini değiştirerek, uygulamanın farklı girdilere nasıl yanıt verdiğini test edebilir ve potansiyel olarak güvenlik açıklarını ortaya çıkarabilirsiniz. Örneğin:


## wenum

Bu bölümde, hedef web uygulamamızdaki hem GET hem de POST parametrelerini keşfetmek için wenum'dan yararlanacağız, sonuçta potansiyel olarak güvenlik açıklarını ortaya çıkaran eşsiz yanıtları tetikleyen gizli değerleri ortaya çıkarmayı amaçlıyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ pipx install git+https://github.com/WebFuzzForge/wenum
M1R4CKCK@htb[/htb]$ pipx runpip wenum install setuptools
```

Ardından başlamak için, endpoint ile manuel olarak etkileşime geçmek ve davranışını daha iyi anlamak için curl kullanacağız:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://IP:PORT/get.php

Invalid parameter value
x: 
```

Yanıt bize x parametresinin eksik olduğunu söylüyor. Bir değer eklemeyi deneyelim:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://IP:PORT/get.php?x=1

Invalid parameter value
x: 1
```

Sunucu bu kez x parametresini onaylar ancak sağlanan değerin (1) geçersiz olduğunu belirtir. Bu, uygulamanın gerçekten de bu parametrenin değerini kontrol ettiğini ve geçerliliğine bağlı olarak farklı yanıtlar ürettiğini göstermektedir. Farklı ve umarız daha açıklayıcı bir yanıtı tetiklemek için belirli bir değer bulmayı amaçlıyoruz.

Parametre değerlerini manuel olarak tahmin etmek sıkıcı ve zaman alıcı olacaktır. İşte bu noktada wenum işe yarıyor. Birçok potansiyel değeri test etme sürecini otomatikleştirmemizi sağlayarak doğru olanı bulma şansımızı önemli ölçüde artırır.

SecLists'ten common.txt kelime listesi ile başlayarak “x” parametresinin değerini fuzzlamak için wenum'u kullanalım:

```shell-session
M1R4CKCK@htb[/htb]$ wenum -w /usr/share/seclists/Discovery/Web-Content/common.txt --hc 404 -u "http://IP:PORT/get.php?x=FUZZ"

...
 Code    Lines     Words        Size  Method   URL 
...
 200       1 L       1 W        25 B  GET      http://IP:PORT/get.php?x=OA... 

Total time: 0:00:02
Processed Requests: 4731
Filtered Requests: 4730
Requests/s: 1681
```

**`--hc 404`**: HTTP 404 hatasını (bulunamadı) içeren yanıtları filtreler, yani 404 hatası dönen sonuçları göz ardı eder.

### POST

Hedef uygulamamız ayrıca post.php script'i içinde “y” adında bir POST parametresine sahiptir. Varsayılan davranışını görmek için bunu curl ile araştıralım:

```shell-session
M1R4CKCK@htb[/htb]$ curl -d "" http://IP:PORT/post.php

Invalid parameter value
y: 
```

**`-d ""`**: Bu seçenek, POST isteği ile gönderilecek veriyi belirtir. Burada boş bir veri (`""`) gönderilmektedir.

-d bayrağı curl'e boş bir body ile POST isteği yapmasını söyler. Yanıt bize y parametresinin beklendiğini ancak sağlanmadığını söyler.

GET parametrelerinde olduğu gibi, POST parametre değerlerini manuel olarak test etmek verimsiz olacaktır. Bu işlemi otomatikleştirmek için ffuf kullanacağız:

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -mc 200 -v

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://IP:PORT/post.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : y=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200
________________________________________________

[Status: 200, Size: 26, Words: 1, Lines: 2, Duration: 7ms]
| URL | http://IP:PORT/post.php
    * FUZZ: SU...

:: Progress: [4730/4730] :: Job [1/1] :: 5555 req/sec :: Duration: [0:00:01] :: Errors: 0 ::
```

Buradaki temel fark, ffuf'a payload'un (“y=FUZZ”) POST verisi olarak Request Body içinde gönderilmesi gerektiğini söyleyen -d bayrağının kullanılmasıdır.

Yine, çoğunlukla geçersiz parametre yanıtları göreceksiniz. Doğru değer (“SU...”) 200 OK durum kodu ile öne çıkacaktır:

```bash
000000326:  200     1 L      1 W     26 Ch     "SU..."
```

Benzer şekilde, “SU...” değerini doğru değer olarak belirledikten sonra curl ile doğrulayın:

```shell-session
M1R4CKCK@htb[/htb]$ curl -d "y=SU..." http://IP:PORT/post.php

HTB{...}
```

Gerçek dünya senaryosunda, bu bayraklar mevcut olmayabilir ve geçerli parametre değerlerinin belirlenmesi yanıtların daha incelikli bir analizini gerektirebilir. Ancak bu alıştırma, birçok potansiyel parametre değerini test etme sürecini otomatikleştirmek için ffuf'tan nasıl yararlanılabileceğinin basitleştirilmiş bir gösterimini sağlar.


Soru : GET parametresini başarıyla fuzzing yaparken hangi bayrağı buluyorsunuz?

Cevap  :

![Pasted image 20241220133637.png](/img/user/resimler/Pasted%20image%2020241220133637.png)

![Pasted image 20241220133820.png](/img/user/resimler/Pasted%20image%2020241220133820.png)

![Pasted image 20241220133828.png](/img/user/resimler/Pasted%20image%2020241220133828.png)

![Pasted image 20241220133833.png](/img/user/resimler/Pasted%20image%2020241220133833.png)


soru  :  POST parametresini başarıyla fuzzing yaparken hangi bayrağı buluyorsunuz?

![Pasted image 20241220134020.png](/img/user/resimler/Pasted%20image%2020241220134020.png)

![Pasted image 20241220134025.png](/img/user/resimler/Pasted%20image%2020241220134025.png)

![Pasted image 20241220134108.png](/img/user/resimler/Pasted%20image%2020241220134108.png)


# Virtual Host and Subdomain Fuzzing

Virtual hosting, birden fazla web sitesinin veya domainin tek bir sunucudan veya IP adresinden sunulmasını sağlar.  Her vhost benzersiz bir domain adı veya hostname ile ilişkilendirilir. Bir client HTTP isteği gönderdiğinde, web sunucusu Host başlığını inceleyerek hangi vhost'un içeriğinin sunulacağını belirler. Birden fazla web sitesi aynı sunucu altyapısını paylaşabildiği için bu, verimli kaynak kullanımını ve maliyet azaltmayı kolaylaştırır.

Öte yandan subdomainler, birincil domain adının uzantılarıdır ve domain içinde hiyerarşik bir yapı oluştururlar. Bir web sitesi içindeki farklı bölümleri veya servisleri düzenlemek için kullanılırlar. Örneğin, blog.example.com ve shop.example.com, example.com ana domaininin alt domainleridir. Vhost'ların aksine, subdomain'ler DNS ( Domain Name System) kayıtları aracılığıyla belirli IP adreslerine çözümlenir.

| Özellik           | Sanal Hostlar (Virtual Hosts)                                                                | Alt Alan Adları (Subdomains)                                                           |
| ----------------- | -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| Tanımlama         | HTTP isteklerindeki Host başlığı ile tanımlanır.                                             | DNS kayıtları ile, belirli IP adreslerine yönlendirilir.                               |
| Amaç              | Genellikle bir sunucuda birden fazla web sitesi barındırmak için kullanılır.                 | Bir web sitesindeki farklı bölümleri veya hizmetleri organize etmek için kullanılır.   |
| Güvenlik Riskleri | Yanlış yapılandırılmış sanal hostlar, iç uygulamalara veya hassas verilere erişimi açabilir. | DNS kayıtları yanlış yönetilirse, alt alan adı ele geçirme açıkları meydana gelebilir. |

## Gobuster

- **Dizinler**: Bir web sunucusunda gizli dizinleri keşfetmek.
- **Dosyalar**: Belirli uzantılara sahip dosyaları (örneğin, .php, .txt, .bak) tanımlamak.
- **Subdomain'ler**: Verilen bir domain'in subdomain'lerini listelemek.
- **Sanal Hostlar (vhosts)**: Host başlığını manipüle ederek gizli sanal hostları keşfetmek.


### Gobuster VHost Fuzzing

```shell-session
M1R4CKCK@htb[/htb]$ echo "IP inlanefreight.htb" | sudo tee -a /etc/hosts
```

```shell-session
M1R4CKCK@htb[/htb]$ gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain
```

- **gobuster vhost**: Bu bayrak, Gobuster'ın vhost fuzzing modunu etkinleştirir ve dizinler veya dosyalar yerine sanal hostları keşfetmeye odaklanmasını sağlar.
- **-u [http://inlanefreight.htb:81](http://inlanefreight.htb:81)**: Bu, hedef sunucunun temel URL'sini belirtir. Gobuster, farklı vhost isimleriyle istekler oluşturmak için bu URL'yi temel alacaktır. Bu örnekte, hedef sunucu inlanefreight.htb'de ve 81 numaralı portta dinliyor.
- **-w /usr/share/seclists/Discovery/Web-Content/common.txt**: Bu, Gobuster'ın potansiyel vhost isimlerini oluşturmak için kullanacağı wordlist dosyasının yolunu belirtir. SecLists'in common.txt wordlist'i, yaygın olarak kullanılan vhost isimleri ve subdomain'lerin bir koleksiyonunu içerir.
- **--append-domain**: Bu önemli bayrak, Gobuster'a her kelimenin sonuna temel domaini (inlanefreight.htb) eklemesini söyler. Bu, her istekte Host başlığının tam bir domain içermesini sağlar (örneğin, admin.inlanefreight.htb), bu da vhost keşfi için gereklidir.



```shell-session
M1R4CKCK@htb[/htb]$ gobuster vhost -u http://inlanefreight.htb:81 -w /usr/share/seclists/Discovery/Web-Content/common.txt --append-domain

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:             http://inlanefreight.htb:81
[+] Method:          GET
[+] Threads:         10
[+] Wordlist:        /usr/share/SecLists/Discovery/Web-Content/common.txt
[+] User Agent:      gobuster/3.6
[+] Timeout:         10s
[+] Append Domain:   true
===============================================================
Starting gobuster in VHOST enumeration mode
===============================================================
Found: .git/logs/.inlanefreight.htb:81 Status: 400 [Size: 157]
...
Found: admin.inlanefreight.htb:81 Status: 200 [Size: 100]
Found: android/config.inlanefreight.htb:81 Status: 400 [Size: 157]
...
Progress: 4730 / 4730 (100.00%)
===============================================================
Finished
===============================================================
```


### Gobuster Subdomain Fuzzing

```shell-session
M1R4CKCK@htb[/htb]$ gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt 
```

gobuster dns: Gobuster'ın DNS fuzzing modunu etkinleştirerek subdomainleri keşfetmeye odaklanmaya yönlendirir.

```shell-session
M1R4CKCK@htb[/htb]$ gobuster dns -d inlanefreight.com -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt 

===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Domain:     inlanefreight.com
[+] Threads:    10
[+] Timeout:    1s
[+] Wordlist:   /usr/share/seclists/Discovery/DNS/subdomains-top1million-5000.txt
===============================================================
Starting gobuster in DNS enumeration mode
===============================================================
Found: www.inlanefreight.com

Found: blog.inlanefreight.com

...

Progress: 4989 / 4990 (99.98%)
===============================================================
Finished
===============================================================
```


Soru : Hedef sisteme karşı GoBuster kullanarak common.txt kelime listesini kullanarak vhost'ları fuzz'lamak için, hangi vhost “web-” önekiyle başlar? Tam vhost ile yanıt verin, örneğin web-123.inlanefreight.htb.

Cevap : 

![Pasted image 20241220140118.png](/img/user/resimler/Pasted%20image%2020241220140118.png)



Soru  :  Subdomains-top1million-5000.txt kelime listesini kullanarak subdomainleri fuzzlamak için inlanefreight.com'a karşı GoBuster kullanarak, hangi subdomain “su” önekiyle başlar? Tam vhost ile yanıt verin, örneğin web.inlanefreight.com.

Cevap : 

![Pasted image 20241220140726.png](/img/user/resimler/Pasted%20image%2020241220140726.png)
![Pasted image 20241220140733.png](/img/user/resimler/Pasted%20image%2020241220140733.png)



### Fuzzing Çıktısını Filtreleme

#### Gobuster

| Flag             | Açıklama                                                                                           | Örnek Senaryo                                                                                             |
| ---------------- | -------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------- |
| -s (dahil et)    | Yalnızca belirtilen durum kodlarına sahip yanıtları dahil et (virgülle ayrılmış).                  | Yönlendirmeleri arıyorsunuz, bu yüzden 301, 302, 307 kodları için filtreleme yapıyorsunuz.                |
| -b (hariç tut)   | Belirtilen durum kodlarına sahip yanıtları hariç tut (virgülle ayrılmış).                          | Sunucu çok sayıda 404 hatası döndürüyor. Bunları hariç tutmak için -b 404 kullanıyorsunuz.                |
| --exclude-length | Belirli içerik uzunluklarına sahip yanıtları hariç tut (virgülle ayrılmış, aralıklar desteklenir). | 0 bayt veya 404 bayt olan yanıtlarla ilgilenmiyorsunuz, bu yüzden --exclude-length 0,404 kullanıyorsunuz. |

Bu filtreleme seçeneklerini stratejik olarak birleştirerek, Gobuster'ın çıktısını özel ihtiyaçlarınıza göre uyarlayabilir ve güvenlik değerlendirmeleriniz için en alakalı sonuçlara odaklanabilirsiniz.

* 200 veya 301 durum kodlu dizinleri bulun, ancak boyutu 0 olan yanıtları (boş yanıtlar) hariç tutun

```shell-session
M1R4CKCK@htb[/htb]$ gobuster dir -u http://example.com/ -w wordlist.txt -s 200,301 --exclude-length 0
```

## FFUF

FFUF, görüntülenen çıktı üzerinde hassas kontrol sağlayan son derece özelleştirilebilir bir filtreleme sistemi sunar. Bu, potansiyel olarak büyük miktarda veriyi verimli bir şekilde elemenize ve en alakalı bulgulara odaklanmanıza olanak tanır. FFUF'un filtreleme seçenekleri, her biri sonuçlarınızı iyileştirmede belirli bir amaca hizmet eden birden fazla türe ayrılmıştır.

|Flag|Açıklama|Örnek Senaryo|
|---|---|---|
|-mc (eşleşen kod)|Yalnızca belirtilen durum kodlarıyla eşleşen yanıtları dahil et. Tek bir kod, virgülle ayrılmış birden fazla kod veya tire ile ayrılmış kod aralıkları (örneğin, 200,204,301, 400-499) belirtebilirsiniz. Varsayılan davranış, 200-299, 301, 302, 307, 401, 403, 405 ve 500 kodlarıyla eşleşmektir.|Fuzzing sonrası birçok 302 (Bulundu) yönlendirmesi fark ediyorsunuz, ancak asıl ilgilendiğiniz 200 (Tamam) yanıtları. -mc 200 kullanarak sadece bunları izole edin.|
|-fc (filtre kodu)|Belirtilen durum kodlarıyla eşleşen yanıtları hariç tutar, aynı formatta -mc gibi. Bu, 404 Not Found gibi yaygın hata kodlarını kaldırmak için kullanışlıdır.|Bir tarama çok sayıda 404 hatası döndürüyor. Bunları çıktıdan çıkarmak için -fc 404 kullanın.|
|-fs (filtre boyutu)|Belirli bir boyuta veya boyut aralığına sahip yanıtları hariç tutar. Tek bir boyut veya tire ile ayrılmış boyut aralıkları belirtebilirsiniz (örneğin, -fs 0 boş yanıtlar için, -fs 100-200 100 ile 200 bayt arasındaki yanıtlar için).|İlginç yanıtların 1KB'den büyük olacağını düşünüyorsunuz. Küçük yanıtları filtrelemek için -fs 0-1023 kullanın.|
|-ms (eşleşen boyut)|Yalnızca belirli bir boyut veya boyut aralığına sahip yanıtları dahil eder, -fs ile aynı formatta.|Boyutunun tam olarak 3456 bayt olduğunu bildiğiniz bir yedek dosyasını arıyorsunuz. Bunu bulmak için -ms 3456 kullanın.|
|-fw (yanıtta kelime sayısını filtrele)|Yanıtta belirli bir sayıda kelime içeren yanıtları hariç tutar.|Yanıtlardan belirli bir kelime sayısını filtreliyorsunuz. Bu miktarda kelime içeren yanıtları filtrelemek için -fw 219 kullanın.|
|-mw (kelime sayısı ile eşleşme)|Yalnızca yanıt gövdesinde belirli bir sayıda kelime bulunan yanıtları dahil eder.|Kısa, belirli hata mesajları arıyorsunuz. 5 ila 10 kelime içeren yanıtları filtrelemek için -mw 5-10 kullanın.|
|-fl (satır filtresi)|Belirli bir sayıda satır veya satır aralığına sahip yanıtları hariç tutar. Örneğin, -fl 5, 5 satır içeren yanıtları filtreler.|10 satırlık hata mesajlarıyla bir desen fark ediyorsunuz. Bunları filtrelemek için -fl 10 kullanın.|
|-ml (satır sayısı ile eşleşme)|Yalnızca yanıt gövdesinde belirli bir sayıda satır bulunan yanıtları dahil eder.|20 satır gibi belirli bir formatta yanıtlar arıyorsunuz. Bunları izole etmek için -ml 20 kullanın.|
|-mt (zaman ile eşleşme)|Yalnızca belirli bir ilk bayta kadar geçen süre (TTFB) koşulunu karşılayan yanıtları dahil eder. Bu, anormal derecede yavaş veya hızlı yanıtları tanımlamak için kullanışlıdır ve bu durum ilginç davranışları gösterebilir.|Uygulama, belirli girişleri işlerken yavaş tepki veriyor. 500 milisaniyeden fazla TTFB'ye sahip yanıtları bulmak için -mt >500 kullanın.|

Birden fazla filtreyi birleştirebilirsiniz. Örneğin:

* Sözcük miktarına göre durum kodu 200 olan ve yanıt boyutu 500 bayttan büyük olan dizinleri bulun

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -u http://example.com/FUZZ -w wordlist.txt -mc 200 -fw 427 -ms >500
```

* 404, 401 ve 302 durum kodlarına sahip yanıtları filtreleyin

```shell-session
ffuf -u http://example.com/FUZZ -w wordlist.txt -fc 404,401,302
```

* .bak uzantılı ve boyutu 10KB ile 100KB arasında olan yedekleme dosyalarını bulun

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -u http://example.com/FUZZ.bak -w wordlist.txt -fs 0-10239 -ms 10240-102400
```

* Yanıt vermesi 500 ms'den uzun süren endpointleri keşfedin

```shell-session
ffuf -u http://example.com/FUZZ -w wordlist.txt -mt >500
```


### wenum

wenum, fuzzing sırasında üretilen büyük miktarda veriyi yönetmenize ve iyileştirmenize yardımcı olmak için sağlam bir filtreleme sistemi sunar. Durum kodlarına, yanıt boyutuna/karakter sayısına, kelime sayısına, satır sayısına ve hatta düzenli ifadelere göre filtreleme yapabilirsiniz.

|Flag|Açıklama|Örnek Senaryo|
|---|---|---|
|--hc (kod gizle)|Belirtilen durum kodlarıyla eşleşen yanıtları hariç tutar.|Fuzzing sonrası sunucu çok sayıda 400 Bad Request hatası döndürdü. Bunları gizlemek ve diğer yanıtlarla ilgilenmek için --hc 400 kullanın.|
|--sc (kod göster)|Yalnızca belirtilen durum kodlarıyla eşleşen yanıtları dahil eder.|Sadece başarılı isteklerle ilgileniyorsunuz (200 OK). Sonuçları buna göre filtrelemek için --sc 200 kullanın.|
|--hl (uzunluk gizle)|Belirtilen içerik uzunluğuna sahip yanıtları hariç tutar (satır sayısı olarak).|Sunucu çok satırlı ayrıntılı hata mesajları döndürüyor. Bunları gizlemek ve daha kısa yanıtlar üzerinde odaklanmak için --hl yüksek bir değerle kullanın.|
|--sl (uzunluk göster)|Yalnızca belirtilen içerik uzunluğuna sahip yanıtları dahil eder (satır sayısı olarak).|Belirli bir satır sayısına sahip yanıtların bir güvenlik açığıyla ilişkili olduğunu düşünüyorsunuz. Bunu bulmak için --sl kullanın.|
|--hw (kelime gizle)|Belirli bir sayıda kelime içeren yanıtları hariç tutar.|Sunucu, birçok yanıtında yaygın ifadeler içeriyor. Bu kelime sayısına sahip yanıtları filtrelemek için --hw kullanın.|
|--sw (kelime göster)|Yalnızca belirtilen sayıda kelime içeren yanıtları dahil eder.|Kısa hata mesajları arıyorsunuz. Bunları bulmak için --sw düşük bir değerle kullanın.|
|--hs (boyut gizle)|Belirli bir yanıt boyutuna (bayt ya da karakter olarak) sahip yanıtları hariç tutar.|Sunucu, geçerli istekler için büyük dosyalar gönderiyor. Bu büyük yanıtları filtrelemek ve daha küçük yanıtlar üzerinde odaklanmak için --hs kullanın.|
|--ss (boyut göster)|Yalnızca belirtilen yanıt boyutuna (bayt ya da karakter olarak) sahip yanıtları dahil eder.|Belirli bir dosya boyutunu arıyorsunuz. Bunu bulmak için --ss kullanın.|
|--hr (regex gizle)|Yanıt gövdesi belirtilen düzenli ifadeyle eşleşen yanıtları hariç tutar.|"Internal Server Error" mesajını içeren yanıtları filtrelemek için --hr "Internal Server Error" kullanın.|
|--sr (regex göster)|Yanıt gövdesi belirtilen düzenli ifadeyle eşleşen yanıtları dahil eder.|"admin" dizesini içeren yanıtları filtrelemek için --sr "admin" kullanın.|
|--filter/--hard-filter|Yanıtları göstermek/gizlemek veya işleme sonrası işlem yapılmasını engellemek için genel amaçlı filtre.|--filter "Login" komutu yalnızca "Login" içeren yanıtları gösterirken, --hard-filter "Login" komutu bunları gizler ve eklentilerin işlem yapmasını engeller.|

Birden fazla filtreyi birleştirebilirsiniz. Örneğin:

```shell-session
# Show only successful requests and redirects:
M1R4CKCK@htb[/htb]$ wenum -w wordlist.txt --sc 200,301,302 -u https://example.com/FUZZ

# Hide responses with common error codes:
M1R4CKCK@htb[/htb]$ wenu -w wordlist.txt --hc 404,400,500 -u https://example.com/FUZZ

# Show only short error messages (responses with 5-10 words):
M1R4CKCK@htb[/htb]$ wenum -w wordlist.txt --sw 5-10 -u https://example.com/FUZZ

# Hide large files and focus on smaller responses:
M1R4CKCK@htb[/htb]$ wenum -w wordlist.txt --hs 10000 -u https://example.com/FUZZ

# Filter for responses containing specific information:
M1R4CKCK@htb[/htb]$ wenum -w wordlist.txt --sr "admin\|password" -u https://example.com/FUZZ
```


## Feroxbuster

Feroxbuster'ın filtreleme sistemi hem güçlü hem de esnek olacak şekilde tasarlanmıştır ve bir tarama sırasında aldığınız sonuçlara ince ayar yapmanıza olanak tanır. Hem istek hem de yanıt seviyelerinde çalışan çeşitli filtreler sunar.

|Flag|Açıklama|Örnek Senaryo|
|---|---|---|
|--dont-scan (İstek)|Belirli URL'leri veya desenleri taramadan hariç tutar (özellikle yineleme sırasında bağlantılarda bulunursa).|/uploads dizininin yalnızca resimler içerdiğini biliyorsunuz, bu yüzden bunu --dont-scan /uploads ile hariç tutabilirsiniz.|
|-S, --filter-size|Yanıtları boyutlarına (bayt olarak) göre hariç tutar. Tek boyutlar veya virgülle ayrılmış boyut aralıkları belirtilebilir.|Birçok 1KB hata sayfası fark ettiniz. Bunları hariç tutmak için -S 1024 kullanabilirsiniz.|
|-X, --filter-regex|Yanıtların gövdesi veya başlıkları, belirtilen düzenli ifadeyle eşleşiyorsa, onları hariç tutar.|-X "Access Denied" kullanarak belirli bir hata mesajını içeren sayfaları filtreleyin.|
|-W, --filter-words|Belirli bir kelime sayısına veya kelime sayısı aralığına sahip yanıtları hariç tutar.|Çok az kelime içeren (örneğin, hata mesajları) yanıtları -W 0-10 ile hariç tutun.|
|-N, --filter-lines|Belirli bir satır sayısına veya satır sayısı aralığına sahip yanıtları hariç tutar.|Uzun, ayrıntılı sayfaları -N 50- ile filtreleyin.|
|-C, --filter-status|Belirli HTTP durum kodlarına göre yanıtları hariç tutar. Bu, bir yasak listesi olarak çalışır.|Yaygın hata kodlarını (404, 500 gibi) -C 404,500 ile bastırın.|
|--filter-similar-to|Belirli bir web sayfasına benzer olan yanıtları hariç tutar.|--filter-similar-to error.html ile bir referans sayfasına dayalı olarak tekrar eden veya neredeyse aynı sayfaları kaldırın.|
|-s, --status-codes|Yalnızca belirtilen durum kodlarına sahip yanıtları dahil eder. Bu, bir izin listesi olarak çalışır (varsayılan: hepsi).|Başarılı yanıtlarla ilgileniyorsanız, -s 200,204,301,302 kullanarak bunlara odaklanın.|

Birden fazla filtreyi birleştirebilirsiniz. Örneğin:

10KB'den büyük veya “hata” kelimesini içeren yanıtları hariç tutarak durum kodu 200 olan dizinleri bulun

```shell-session
M1R4CKCK@htb[/htb]$ feroxbuster --url http://example.com -w wordlist.txt -s 200 -S 10240 -X "error" 
```

Şimdiye kadarki modül boyunca, bazı komutların bir tür sonuç filtrelemesi kullandığını veya fuzzer'ların kendilerinin bir tür filtreleme uyguladığını fark etmiş olabilirsiniz. Örneğin, ffuf ile POST fuzzing için, eşleşme kodu filtresini kaldırırsak, ffuf varsayılan olarak bir dizi başka filtreye geçecektir.

```shell-session
M1R4CKCK@htb[/htb]$ ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -v

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://IP:PORT/post.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : y=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: 200-299,301,302,307,401,403,405,500
________________________________________________
```

Yukarıdaki çıktıda :: Matcher : Response status: 200-299,301,302,307,401,403,405,500 satırı, ffuf'un varsayılan olarak yalnızca bu belirli durum kodlarıyla eşleştiğini gösterir. Bu kasıtlı filtreleme, 404 NOT FOUND yanıtlarının oluşturduğu gürültüyü en aza indirerek ilgilenilen sonuçların belirgin kalmasını sağlar.

Filtreleme yapmamanın potansiyel sorununu göstermek için, -mc all bayrağını kullanarak tüm durum kodlarını eşleştirirken aynı taramayı çalıştıralım:


```shell-session
M1R4CKCK@htb[/htb]$ ffuf -u http://IP:PORT/post.php -X POST -H "Content-Type: application/x-www-form-urlencoded" -d "y=FUZZ" -w /usr/share/seclists/Discovery/Web-Content/common.txt -v -mc all 


        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v2.1.0-dev
________________________________________________

 :: Method           : POST
 :: URL              : http://IP:PORT/post.php
 :: Wordlist         : FUZZ: /usr/share/seclists/Discovery/Web-Content/common.txt
 :: Header           : Content-Type: application/x-www-form-urlencoded
 :: Data             : y=FUZZ
 :: Follow redirects : false
 :: Calibration      : false
 :: Timeout          : 10
 :: Threads          : 40
 :: Matcher          : Response status: all
________________________________________________

[Status: 404, Size: 36, Words: 4, Lines: 3, Duration: 1ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .cache

[Status: 404, Size: 43, Words: 4, Lines: 3, Duration: 2ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .bash_history

[Status: 404, Size: 34, Words: 4, Lines: 3, Duration: 2ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .cvs

[Status: 404, Size: 42, Words: 4, Lines: 3, Duration: 2ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .git-rewrite

[Status: 404, Size: 40, Words: 4, Lines: 3, Duration: 2ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .cvsignore

[Status: 404, Size: 39, Words: 4, Lines: 3, Duration: 3ms]
| URL | http://IP:PORT/post.php
    * FUZZ: .git/HEAD
...
```

Ortaya çıkan çıktı 404 NOT FOUND sonuçlarıyla dolup taşar ve bu da potansiyel olarak değerli bulguların tespit edilmesini çok daha zor hale getirir. Bu durum, fuzzing sürecini optimize etmek ve anlamlı sonuçlara öncelik vermek için uygun filtreleme tekniklerinin kullanılmasının önemini göstermektedir.


Soru : POST parametresini başarıyla fuzzing yaparken hangi bayrağı buluyorsunuz?

Cevap : 

![Pasted image 20241220145608.png](/img/user/resimler/Pasted%20image%2020241220145608.png)


![Pasted image 20241220145553.png](/img/user/resimler/Pasted%20image%2020241220145553.png)


### Validating Findings (Bulguların Doğrulanması)

Fuzzing geniş bir ağ oluşturmak ve potansiyel ipuçları üretmek için mükemmeldir, ancak her bulgu gerçek bir güvenlik açığı değildir. Süreç genellikle yanlış pozitif sonuçlar verir - fuzzer'ın tespit mekanizmalarını tetikleyen ancak gerçek bir tehdit oluşturmayan zararsız anomaliler. Bu nedenle doğrulama, fuzzing iş akışında çok önemli bir adımdır.


## Why Validate?

Bulguları doğrulamanın amaçları:

- **Açıkları Doğrulama:** Gerçek güvenlik açıklarını yanlış alarmlardan ayırır.
- **Etkisini Değerlendirme:** Açığın ciddiyetini ve etkisini anlamaya yardımcı olur.
- **Yeniden Üretme:** Sorunun tekrarlanabilirliğini sağlar.
- **Kanıt Toplama:** Geliştiricilere veya paydaşlara sunulmak üzere kanıt sağlar.


## Manual Verification

Potansiyel bir güvenlik açığını doğrulamanın en güvenilir yolu manuel doğrulamadır:

- **İsteği Tekrarla:** curl veya tarayıcı ile aynı isteği manuel olarak gönder.
- **Yanıtı İncele:** Hata mesajları, beklenmeyen içerik veya anormal davranışları kontrol et.
- **Sömürü Denemesi:** Kontrollü bir ortamda, izin alarak açığın etkisini değerlendir.

Üretim sistemine zarar vermekten kaçının ve zararsız bir PoC oluşturarak güvenlik açığını gösterin. Amaç, etik ve yasal çerçevede güvenlik açığını kanıtlamaktır.


Potansiyel bir güvenlik açığını doğrulamanın en güvenilir yolu manuel doğrulamadır:

- **İsteği Tekrarla:** curl veya tarayıcı ile aynı isteği manuel olarak gönder.
- **Yanıtı İncele:** Hata mesajları, beklenmeyen içerik veya anormal davranışları kontrol et.
- **Sömürü Denemesi:** Kontrollü bir ortamda, izin alarak açığın etkisini değerlendir.

Üretim sistemine zarar vermekten kaçının ve zararsız bir PoC oluşturarak güvenlik açığını gösterin. Amaç, etik ve yasal çerçevede güvenlik açığını kanıtlamaktır.



## Example

Fuzzer’ınız bir web sunucusunda /backup/ dizini keşfetti ve 200 OK döndü. Yedekleme dizinleri genellikle hassas bilgiler içerir:

- **Veritabanı dökümleri:** Kullanıcı bilgileri ve gizli veriler.
- **Yapılandırma dosyaları:** API veya şifreleme anahtarları.
- **Kaynak kodu:** Güvenlik açıklarını ortaya çıkarabilir.

Bu dosyalara erişim, uygulamayı tehlikeye atabilir. Ancak, güvenlik uzmanı olarak hedefe zarar vermeden ve hukuki risk oluşturmadan sorunun varlığını kanıtlamanız gerekir.



### Doğrulama için curl kullanma

```shell-session
[!bash!]$ curl http://IP:PORT/backup/
```

Çıktıyı terminalinizde inceleyin. Sunucu /backup dizini içinde bulunan dosya ve dizinlerin bir listesiyle yanıt verirse, dizin listeleme güvenlik açığını başarıyla doğrulamış olursunuz. Bu şuna benzer bir şey olabilir:

```html
<!DOCTYPE html>
<html>
<head>
<title>Index of /backup/</title>
<style type="text/css">
[...]
</style>
</head>
<body>
<h2>Index of /backup/</h2>
<div class="list">
<table summary="Directory Listing" cellpadding="0" cellspacing="0">
<thead><tr><th class="n">Name</th><th class="m">Last Modified</th><th class="s">Size</th><th class="t">Type</th></tr></thead>
<tbody>
<tr class="d"><td class="n"><a href="../">..</a>/</td><td class="m">&nbsp;</td><td class="s">- &nbsp;</td><td class="t">Directory</td></tr>
<tr><td class="n"><a href="backup.sql">backup.sql</a></td><td class="m">2024-Jun-12 14:00:46</td><td class="s">0.2K</td><td class="t">application/octet-stream</td></tr>
</tbody>
</table>
</div>
<div class="foot">lighttpd/1.4.76</div>

<script type="text/javascript">
[...]
</script>

</body>
</html>
```

Hassas verilerin açığa çıkma riskini azaltarak güvenlik açığını doğrulamak için:

- **Content-Type Başlığı:** Yanıt başlıklarında dosya türünü inceleyin (ör. `application/sql` bir veritabanı dökümüne, `application/zip` sıkıştırılmış bir yedeklemeye işaret eder).
- **Content-Length Başlığı:**
    - **>0:** Dosyanın gerçek içerik barındırdığını gösterebilir.
    - **=0:** Dosya boş olabilir, bu durumda doğrudan bir güvenlik riski oluşturmaz.

Boş dosyaların varlığı şüpheli olsa da, hemen bir risk anlamına gelmez.

İşte password.txt adlı bir dosyanın yalnızca headerlarını almak için curl kullanan bir örnek:

```shell-session
[!bash!]$ curl -I http://IP:PORT/backup/password.txt

HTTP/1.1 200 OK
Content-Type: text/plain;charset=utf-8
ETag: "3406387762"
Last-Modified: Wed, 12 Jun 2024 14:08:46 GMT
Content-Length: 171
Accept-Ranges: bytes
Date: Wed, 12 Jun 2024 14:08:59 GMT
Server: lighttpd/1.4.76
```

- **Content-Type:** `text/plain;charset=utf-8`, password.txt düz metin dosyasıdır.
- **Content-Length:** 171 bayt, dosyanın boş olmadığını ve veri içerdiğini gösterir.

Headerlar, dosyaya erişmeden güvenlik riskini doğrulamak için güçlü kanıt sağlar. Bu, sorumlu açıklama uygulamalarına uygundur.

Soru : Hedef sistemi directory-list-2.3-medium.txt dosyasını kullanarak fuzzlayın ve gizli bir dizin arayın. Gizli dizini bulduktan sonra, dizindeki tar.gz dosyasını analiz ederek güvenlik açığının geçerliliğini sorumlu bir şekilde belirleyin. Tam Content-Length başlığını kullanarak yanıtlayın, örneğin “Content-Length: 1337”

Cevap : 

![Pasted image 20241222042448.png](/img/user/resimler/Pasted%20image%2020241222042448.png)
![Pasted image 20241222042459.png](/img/user/resimler/Pasted%20image%2020241222042459.png)

![Pasted image 20241222042636.png](/img/user/resimler/Pasted%20image%2020241222042636.png)

![Pasted image 20241222043312.png](/img/user/resimler/Pasted%20image%2020241222043312.png)


# Web APIs
Web API, farklı yazılım uygulamalarının internet üzerinden veri ve hizmet alışverişi yapmasını sağlayan bir dizi kuraldır. Server ile client arasında köprü görevi görür ve çeşitli Web API'leri farklı kullanım senaryolarına göre kullanılır.


## Representational State Transfer (REST)

REST API'leri web servisleri oluşturmak için popüler bir mimari stildir. Client'ların kaynaklara erişmek veya kaynakları değiştirmek için server'lara istek gönderdiği stateless, client-server iletişim modelini kullanırlar. REST API'leri, benzersiz URL'lerle tanımlanan kaynaklar üzerinde CRUD ( Create, Read, Update, Delete) işlemleri gerçekleştirmek için standart HTTP yöntemlerini (GET, POST, PUT, DELETE) kullanır. Genellikle JSON veya XML gibi hafif formatlarda veri alışverişi yaparlar, bu da çeşitli uygulamalar ve platformlarla entegre olmalarını kolaylaştırır.

Example query:

```http
GET /users/123
```


## Simple Object Access Protocol (SOAP)
SOAP API'leri yapılandırılmış bilgi alışverişi için daha resmi ve standartlaştırılmış bir protokol izler. Mesajları tanımlamak için XML kullanırlar, bunlar daha sonra SOAP paketleri içinde kapsüllenir ve HTTP veya SMTP gibi ağ protokolleri üzerinden iletilir. SOAP API'leri genellikle built-in security, reliability ve transaction management özellikleri içerir, bu da onları sıkı veri bütünlüğü ve hata işleme gerektiren kurumsal düzeydeki uygulamalar için uygun hale getirir.

Example query:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:tem="http://tempuri.org/">
   <soapenv:Header/>
   <soapenv:Body>
      <tem:GetStockPrice>
         <tem:StockName>AAPL</tem:StockName>
      </tem:GetStockPrice>
   </soapenv:Body>
</soapenv:Envelope>
```

## GraphQL

- **Yeni Bir Sorgu Dili ve Runtime**: GraphQL, API'ler için nispeten yeni bir sorgu dili ve çalışma zamanıdır (runtime), yani sorguların işlendiği ve verilerin alındığı süreçtir.
- **Tek Endpoint ile Veri İsteği**: REST API'lerinin aksine, GraphQL tek bir endpoint sağlar. Client'lar, esnek bir sorgu dili kullanarak sadece ihtiyaç duydukları verileri talep edebilir.
- **Aşırı veya Yetersiz Veri Alma Sorununun Çözülmesi**: REST API'lerinde genellikle fazla veri çekme (over-fetching) veya eksik veri çekme (under-fetching) gibi sorunlar yaşanır. GraphQL, istemcilerin yalnızca istedikleri veriyi talep etmelerini sağlar, böylece bu sorunlar ortadan kalkar.
- **Güçlü Yazım ve İç Gözlem Yeteneği**: GraphQL'in güçlü yazım (strong typing) ve iç gözlem (introspection) özellikleri, API'lerin daha güvenli ve esnek olmasını sağlar. İç gözlem, istemcilerin API hakkında bilgi alabilmesine ve doğru veri talepleri yapabilmesine olanak tanır.
- **Zaman İçinde API Geliştirme Kolaylığı**: GraphQL, mevcut istemcileri bozmadan API'leri zaman içinde geliştirmeyi kolaylaştırır. Bu, uygulamaların ölçeklenmesini ve evrimleşmesini sağlar.
- **Popülerlik ve Kullanım Alanı**: Bu özellikler sayesinde GraphQL, modern web ve mobil uygulamalarda popüler bir seçim haline gelir.

Example query:

```graphql
query {
  user(id: 123) {
    name
    email
  }
}
```


### Web API'lerinin Avantajları

Web API'leri, istemcilerin sunucu verilerine erişimini sağlayarak uygulama geliştirmeyi dönüştürmüştür. Geliştiriciler, uygulamalarının özelliklerini dış kullanıcılarla veya diğer uygulamalarla paylaşarak kodu yeniden kullanabilir ve birleşik uygulamalar oluşturabilir.

Ayrıca, Web API'leri üçüncü taraf hizmetlerinin entegrasyonunu (sosyal medya girişleri, ödeme işlemleri, harita işlevselliği) kolaylaştırır, böylece dış yetenekler yeniden keşfedilmeden entegre edilebilir.

API'ler, mikroservis mimarisinin temelini oluşturur, bu da büyük uygulamaları bağımsız servislere ayırarak ölçeklenebilirliği, esnekliği ve dayanıklılığı artırır.


### API'lerin bir web sunucusundan farkı nedir?

Hem geleneksel web sayfaları hem de Web API'leri web ekosisteminde hayati roller oynarken, farklı yapı, iletişim ve işlevsellik özelliklerine sahiptirler. Bu farklılıkları anlamak, etkili fuzzing için çok önemlidir.

|Özellik|Web Sunucusu|API (Uygulama Programlama Arayüzü)|
|---|---|---|
|**Amaç**|Genellikle statik içerik (HTML, CSS, görseller) ve dinamik web sayfalarını (sunucu tarafı betikleriyle üretilen) sunmak için tasarlanmıştır.|Farklı yazılım uygulamalarının birbirleriyle iletişim kurmasını, veri alışverişi yapmasını ve eylemleri tetiklemesini sağlamak için tasarlanmıştır.|
|**İletişim**|Web tarayıcılarıyla HTTP (Hipermetin Transfer Protokolü) kullanarak iletişim kurar.|İletişim için çeşitli protokoller kullanabilir, bunlar arasında HTTP, HTTPS, SOAP ve diğerleri bulunur; kullanılan protokol API spesifikasyonuna bağlıdır.|
|**Veri Formatı**|Genellikle HTML, CSS, JavaScript ve diğer web ile ilgili formatlarla çalışır.|JSON, XML ve diğerleri gibi çeşitli formatlarda veri alışverişi yapabilir, bu API spesifikasyonuna bağlıdır.|
|**Kullanıcı Etkileşimi**|Kullanıcılar, web sayfalarını ve içeriği görmek için doğrudan web tarayıcıları aracılığıyla web sunucularıyla etkileşime girer.|Kullanıcılar genellikle API'lerle doğrudan etkileşime girmez; bunun yerine, uygulamalar API'leri kullanarak kullanıcı adına veri veya işlevsellik erişir.|
|**Erişim**|Web sunucuları genellikle internet üzerinden herkese açıktır.|API'ler, kamuya açık, özel (yalnızca dahili kullanım için) veya partner (belirli partnerler veya müşteriler için erişilebilir) olabilir.|
|**Örnek**|[https://www.example.com](https://www.example.com) gibi bir web sitesine eriştiğinizde, tarayıcınızda web sayfasını render etmek için HTML, CSS ve JavaScript kodlarını gönderen bir web sunucusuyla etkileşimde bulunuyorsunuz.|Telefonunuzdaki hava durumu uygulaması, hava durumu verilerini uzaktaki bir sunucudan almak için bir hava durumu API'si kullanabilir. Uygulama bu veriyi işler ve kullanıcı dostu bir formatta size sunar. API ile doğrudan etkileşime girmiyorsunuz, ancak uygulama bunu arka planda hava durumu bilgisini sağlamak için kullanır.|


# Identifying Endpoints

Web API'lerini fuzzing yapmaya başlamadan önce nereye bakacağınızı bilmeniz gerekir.

## REST

REST API'leri, kaynak kavramı etrafında inşa edilmiştir ve bu kaynaklar, **endpoint** adı verilen benzersiz URL'lerle tanımlanır. Bu endpoint'ler, istemci isteklerinin hedefidir ve genellikle talep edilen işlem hakkında ek bağlam veya kontrol sağlamak için parametreler içerir.

REST API'lerindeki endpoint'ler, erişmek veya üzerinde işlem yapmak istediğiniz kaynakları temsil eden URL'ler olarak yapılandırılır. Örneğin:

- **/users**: Kullanıcı kaynaklarının koleksiyonunu temsil eder.
- **/users/123**: 123 kimliğine sahip belirli bir kullanıcıyı temsil eder.
- **/products**: Ürün kaynaklarının koleksiyonunu temsil eder.
- **/products/456**: 456 kimliğine sahip belirli bir ürünü temsil eder.

Bu endpoint'lerin yapısı, daha genel kategorilerin altında daha özel kaynakların yer aldığı hiyerarşik bir düzeni takip eder.

Parametreler, API isteklerinin davranışını değiştirmek veya ek bilgi sağlamak için kullanılır. REST API'lerinde çeşitli parametre türleri vardır:

|**Parametre Türü**|**Açıklama**|**Örnek**|
|---|---|---|
|**Query Parameters**|Endpoint URL'sine bir soru işaretinden (?) sonra eklenir. Filtreleme, sıralama veya sayfalama için kullanılır.|`/users?limit=10&sort=name`|
|**Path Parameters**|Doğrudan endpoint URL'sinin içine yerleştirilir. Belirli kaynakları tanımlamak için kullanılır.|`/products/{id}`|
|**Request Body Parameters**|POST, PUT veya PATCH isteklerinin gövdesinde gönderilir. Kaynak oluşturmak veya güncellemek için kullanılır.|`{ "name": "New Product", "price": 99.99 }`|



### REST Endpoint'lerini ve Parametrelerini Keşfetme

REST API endpointlerini ve parametrelerini keşfetmek için:

- **API Dokümantasyonu**: Resmi dokümanlar endpointler, parametreler ve istek/yanıt formatlarını açıklar. Swagger (OpenAPI) gibi araçlarla ayrıntılara ulaşabilirsiniz.
- **Ağ Trafiği Analizi**: Burp Suite veya tarayıcı geliştirici araçlarıyla API istek/yanıtlarını inceleyerek detaylara ulaşabilirsiniz.
- **Fuzzing**: ffuf veya wfuzz gibi araçlarla gizli parametre ve endpointleri keşfedebilirsiniz.



## SOAP

SOAP (Simple Object Access Protocol) API'ler, REST API'lerden farklı bir yapıya sahiptir. XML tabanlı mesajlara ve arayüz ile işlemleri tanımlayan WSDL (Web Services Description Language) dosyalarına dayanır.

REST API'lerde her kaynak için ayrı URL'ler kullanılırken, SOAP API'ler genellikle tek bir endpoint sunar. Bu endpoint, SOAP sunucusunun gelen istekleri dinlediği bir URL'dir ve yapılacak işlem, SOAP mesajının içeriğiyle belirlenir.

SOAP parametreleri, XML formatındaki SOAP mesajının body'sinde tanımlanır. Bu parametreler, elemanlar ve özellikler olarak hiyerarşik bir yapıya sahiptir ve ilgili işlem için WSDL dosyasında belirtilir. WSDL, web servisinin arayüzünü, işlemlerini ve mesaj formatlarını tanımlayan XML tabanlı bir belgedir.

Örneğin, bir kütüphanede kitap arama servisi sunan bir SOAP API düşünün. WSDL dosyasında **SearchBooks** adlı bir işlem ve şu giriş parametreleri tanımlanmış olabilir:

- **keywords** (string): Arama terimleri.
- **author** (string): Yazar adı (isteğe bağlı).
- **genre** (string): Kitap türü (isteğe bağlı).


Bu API'ye yönelik örnek bir SOAP isteği aşağıdaki gibi görünebilir:

```xml
<soapenv:Envelope xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:lib="http://example.com/library">
   <soapenv:Header/>
   <soapenv:Body>
      <lib:SearchBooks>
         <lib:keywords>cybersecurity</lib:keywords>
         <lib:author>Dan Kaminsky</lib:author>
      </lib:SearchBooks>
   </soapenv:Body>
</soapenv:Envelope>
```

Bu request'te:

* Bu konudaki kitapları aramak için keywords parametresi “cybersecurity” olarak ayarlanmıştır.
* Yazar parametresi, aramayı daha da daraltmak için “Dan Kaminsky” olarak ayarlanmıştır.
* Tür parametresi dahil edilmemiştir, yani arama türe göre filtrelenmeyecektir.

SOAP yanıtı muhtemelen WSDL tanımına göre biçimlendirilmiş, arama kriterleriyle eşleşen kitapların bir listesini içerecektir.


### SOAP Endpoint'lerini ve Parametrelerini Keşfetme

Bir SOAP API'sinin kullanılabilir endpointlerini (işlemler) ve parametrelerini belirlemek için şu yöntemleri kullanabilirsiniz:

**WSDL Analizi:**  
WSDL dosyası, bir SOAP API'sini anlamak için en değerli kaynaktır. Şunları tanımlar:

- Kullanılabilir işlemler (endpointler)
- Her işlem için giriş parametreleri (mesaj türleri, elemanlar ve özellikler)
- Her işlem için çıkış parametreleri (yanıt mesaj türleri)
- Parametrelerde kullanılan veri türleri (örneğin, string, integer, karmaşık türler)
- SOAP endpointinin konumu (URL)  
    WSDL dosyasını manuel olarak inceleyebilir veya WSDL yapılarını analiz ve görselleştirmek için tasarlanmış araçları kullanabilirsiniz.

**Ağ Trafiği Analizi:**  
REST API'lerde olduğu gibi, SOAP trafiğini yakalayarak istemci ve sunucu arasındaki istek ve yanıtları inceleyebilirsiniz. Wireshark veya tcpdump gibi araçlar, SOAP trafiğini yakalayarak mesajların yapısını incelemenizi ve endpointler ile parametreler hakkında bilgi edinmenizi sağlar.

**Parametre İsimleri ve Değerleri İçin Fuzzing:**  
SOAP API'leri genellikle iyi tanımlanmış bir yapıya sahip olsa da, fuzzing yöntemiyle gizli veya belgelenmemiş işlemleri ya da parametreleri ortaya çıkarabilirsiniz. Fuzzing araçlarıyla SOAP isteklerine hatalı veya beklenmedik değerler göndererek sunucunun nasıl yanıt verdiğini görebilirsiniz.


## Identifying GraphQL API Endpoints and Parameters

GraphQL API'leri, REST ve SOAP API'lerinden daha esnek ve verimli olacak şekilde tasarlanmıştır ve clientlerin tek bir istekte tam olarak ihtiyaç duydukları verileri talep etmelerine olanak tanır.

Genellikle farklı kaynaklar için birden fazla endpoint ortaya çıkaran REST veya SOAP API'lerinin aksine, GraphQL API'lerinin tipik olarak tek bir endpoint'i vardır. Bu endpoint genellikle /graphql gibi bir URL'dir ve API'ye gönderilen tüm sorgular ve mutations için giriş noktası görevi görür.

GraphQL, veri gereksinimlerini belirtmek için benzersiz bir sorgu dili kullanır. Bu dil içerisinde sorgular ve mutations , parametrelerin tanımlanması ve istenen verilerin yapılandırılması için araç görevi görür.


### GraphQL Queries

Sorgular GraphQL sunucusundan veri almak için tasarlanmıştır. Client'ın istediği alanları, ilişkileri ve iç içe geçmiş nesneleri tam olarak belirleyerek REST API'lerinde yaygın olan aşırı veya yetersiz veri getirme sorununu ortadan kaldırırlar. Sorgulardaki argümanlar, filtreleme veya sayfalama gibi daha fazla iyileştirmeye olanak tanır.

|**Bileşen**|**Açıklama**|**Örnek**|
|---|---|---|
|**Alan (Field)**|Almak istediğiniz belirli bir veri parçasını temsil eder (örn. isim, e-posta).|`name`, `email`|
|**İlişki (Relationship)**|Farklı veri türleri arasındaki bağlantıyı gösterir (örn. bir kullanıcının gönderileri).|`posts`|
|**İç İçe Obje (Nested Object)**|Başka bir obje döndüren bir alan, verilerde daha derinlere gitmenizi sağlar.|`posts { title, body }`|
|**Argüman (Argument)**|Bir sorgunun veya alanın davranışını değiştirir (örn. filtreleme, sıralama, sayfalama).|`posts(limit: 5)` (ilk 5 gönderi)|

```graphql
query {
  user(id: 123) {
    name
    email
    posts(limit: 5) {
      title
      body
    }
  }
}
```


Bu örnekte:

* Kimliği 123 olan bir kullanıcı hakkında bilgi sorgularız.
* İsimlerini ve e-postalarını istiyoruz.
* Ayrıca, her bir gönderinin title ve body'si de dahil olmak üzere ilk 5 gönderisini getiriyoruz



### GraphQL Mutations

Mutations , sunucudaki verileri değiştirmek için tasarlanmış sorguların karşılıklarıdır. Veri oluşturma, güncelleme veya silme işlemlerini kapsarlar. Sorgular gibi, mutasyonlar da bu operasyonlar için girdi değerlerini tanımlamak üzere argümanlar kabul edebilir.

| **Bileşen**   | **Açıklama**                                                                                 | **Örnek**                                              |
| ------------- | -------------------------------------------------------------------------------------------- | ------------------------------------------------------ |
| **Operation** | Yapılacak işlemi belirtir (örn. gönderi oluşturma, kullanıcı güncelleme, yorum silme).       | `createPost`                                           |
| **Argument**  | İşlem için gereken giriş verileri (örn. yeni bir gönderi için başlık ve içerik).             | `title: "Yeni Gönderi", body: "Bu gönderinin içeriği"` |
| **Selection** | Mutasyon tamamlandıktan sonra yanıtta almak istediğiniz alanları belirtir (örn. id, başlık). | `id`, `title`                                          |

```graphql
mutation {
  createPost(title: "New Post", body: "This is the content of the new post") {
    id
    title
  }
}
```

Bu mutasyon, belirtilen title ve body ile yeni bir gönderi oluşturur ve yanıtta yeni oluşturulan gönderinin id'sini ve başlığını döndürür.


**Sorguları ve Mutasyonları Keşfetme**  
GraphQL sorgularını ve mutasyonlarını keşfetmenin birkaç yolu vardır:

- **İç Gözlem (Introspection):**  
    GraphQL'in introspection sistemi, API'yi keşfetmek için güçlü bir araçtır. GraphQL endpoint'ine bir introspection sorgusu göndererek, API'nin yeteneklerini tanımlayan tam bir schema alabilirsiniz. Bu schema; kullanılabilir türleri, alanları, sorguları, mutasyonları ve argümanları içerir. Araçlar ve IDE'ler, bu bilgiyi otomatik tamamlama, doğrulama ve GraphQL sorguları için belgeler sağlamak amacıyla kullanabilir.
    
- **API Dokümantasyonu:**  
    İyi belgelenmiş GraphQL API'leri, introspection ile birlikte kapsamlı kılavuzlar ve referanslar sunar. Bu dokümanlar genellikle farklı sorgu ve mutasyonların amacını ve kullanımını açıklar, geçerli yapı örnekleri verir ve giriş argümanları ile yanıt formatlarını detaylandırır. GraphiQL veya GraphQL Playground gibi araçlar, genellikle GraphQL sunucularıyla birlikte gelir ve şemayı keşfetmek ve sorgular üzerinde denemeler yapmak için etkileşimli bir ortam sağlar.
    
- **Ağ Trafiği Analizi:**  
    REST ve SOAP'ta olduğu gibi, ağ trafiğini analiz etmek, GraphQL API yapısını ve kullanımını anlamanızı sağlayabilir. GraphQL endpoint'ine gönderilen istekleri ve alınan yanıtları yakalayıp inceleyerek, gerçek dünyada kullanılan sorgu ve mutasyonları gözlemleyebilirsiniz. Bu, isteklerin beklenen formatını ve döndürülen veri türlerini anlamanıza yardımcı olur ve özelleştirilmiş fuzzing çabalarına katkı sağlar.
    

Unutmayın, GraphQL esneklik için tasarlanmıştır, bu yüzden sabit bir sorgu ve mutasyon seti olmayabilir. Temel şemayı ve istemcilerin verileri almak veya değiştirmek için nasıl geçerli istekler oluşturabileceğini anlamaya odaklanın.


# API Fuzzing

API fuzzing, web API'leri için özel olarak tasarlanmış bir fuzzing türüdür. Fuzzing'in temel ilkeleri aynı kalsa da - bir hedefe beklenmedik veya geçersiz girdiler göndermek - API fuzzing, web API'leri tarafından kullanılan benzersiz yapı ve protokollere odaklanır.

API fuzzing, her testin bir API endpoint'ine biraz değiştirilmiş bir istek gönderdiği bir dizi otomatik test ile bir API'nin bombardımana tutulmasını içerir. Bu değişiklikler şunları içerebilir:

* Parametre değerlerinin değiştirilmesi
* İstek headerlarının değiştirilmesi
* Parametrelerin sırasını değiştirme
* Beklenmedik veri türlerinin veya formatlarının tanıtılması

Amaç, API hatalarını, çökmelerini veya beklenmedik davranışları tetikleyerek girdi doğrulama kusurları, enjeksiyon saldırıları veya kimlik doğrulama sorunları gibi potansiyel güvenlik açıklarını ortaya çıkarmaktır.


**API'leri Neden Fuzz'lamalıyız?**

- **Gizli Güvenlik Açıklarını Ortaya Çıkarmak:** API'lerdeki gizli endpoint'ler ve parametreler saldırılara karşı savunmasız olabilir.
- **Sağlamlık Testi Yapmak:** API'nin hatalı girdilerle düzgün çalışıp çalışmadığını test eder.
- **Güvenlik Testlerini Otomatikleştirmek:** Fuzz'lama, tüm giriş kombinasyonlarını manuel olarak test etmeyi kolaylaştırır.
- **Gerçek Dünya Saldırılarını Simüle Etmek:** Kötü niyetli hareketleri taklit ederek açıkları önceden tespit eder.


### **API Fuzz'lamanın Türleri**

API fuzz'lamada üç ana teknik vardır:

- **Parametre Fuzz'laması:** API parametrelerine farklı değerler sistematik olarak test edilir. Bu, sorgu parametrelerini (API endpoint URL'sine eklenen), başlıkları (istekle ilgili meta veriler içeren) ve istek gövdelerini (veri yükünü taşıyan) içerir. Beklenmedik veya geçersiz değerler enjekte edilerek SQL enjeksiyonu, komut enjeksiyonu, çapraz site betikleme (XSS) ve parametre manipülasyonu gibi güvenlik açıkları ortaya çıkarılabilir.
    
- **Veri Formatı Fuzz'laması:** Web API'leri genellikle JSON veya XML gibi yapılandırılmış veri formatları kullanarak veri alışverişi yapar. Veri formatı fuzz'laması, bu formatları hedef alarak veri yapısı, içeriği veya kodlamasını manipüle eder. Bu, ayrıştırma hataları, buffer taşmaları  veya özel karakterlerin yanlış işlenmesi gibi güvenlik açıklarını ortaya çıkarabilir.
    
- **Dizi Fuzz'laması:** API'ler genellikle birbirine bağlı birden fazla endpoint içerir ve isteklerin sırası ve zamanı çok önemlidir. Dizi fuzz'laması, API'nin istek sırasına verdiği yanıtları inceleyerek yarış durumu, güvenli olmayan doğrudan nesne referansları (IDOR) veya yetkilendirme atlamaları gibi açıkları keşfeder. API çağrılarının sırası, zamanlaması veya parametreleri manipüle edilerek API'nin mantığında ve durum yönetiminde zayıflıklar ortaya çıkarılabilir.


### API'yi Keşfetme

Takip etmek için, sayfanın altındaki soru bölümünden hedef sistemi başlatın ve IP:PORT kullanımlarını, ortaya çıkan örneğinizin IP:PORT'u ile değiştirin.

Bu API, http://IP:PORT/docs adresindeki /docs endpoint aracılığıyla otomatik olarak oluşturulan dokümantasyon sağlar. Aşağıdaki sayfa API'nin belgelenmiş endpoint'ini özetlemektedir.

![Pasted image 20241222132727.png](/img/user/resimler/Pasted%20image%2020241222132727.png)

Özellik, her biri belirli bir amacı ve yöntemi olan beş endpointi detaylandırır:

- **GET / (Root Okuma):** Root kaynağını alır. Muhtemelen temel bir karşılama mesajı veya API bilgilerini döndürür.
- **GET /items/{item_id} (İtem Okuma):** item_id ile tanımlanan belirli bir öğeyi getirir.
- **DELETE /items/{item_id} (İtem Silme):** item_id ile tanımlanan bir öğeyi siler.
- **PUT /items/{item_id} (İtem Güncelleme):** Mevcut bir öğeyi sağlanan verilerle günceller.
- **POST /items/ (İtem Oluşturma veya Güncelleme):** Bu işlev yeni bir öğe oluşturur veya item_id eşleşirse mevcut bir öğeyi günceller.

Swagger spesifikasyonu beş endpointi açıkça detaylandırmış olsa da, API'lerin bazen belgelenmemiş veya "gizli" endpointler içerebileceğini kabul etmek önemlidir.

Bu gizli endpointler, genellikle dış kullanıma yönelik olmayan dahili işlevleri sağlamak için, güvenliği obscurity (gizlilik) yöntemiyle sağlamaya yönelik yanlış bir çaba olarak ya da henüz geliştirilmekte olup halk kullanımı için hazır olmayan endpointler olarak var olabilir.


## Fuzzing the API

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/PandaSt0rm/webfuzz_api.git
M1R4CKCK@htb[/htb]$ cd webfuzz_api
M1R4CKCK@htb[/htb]$ pip3 install -r requirements.txt
```

Ardından, oluşturulan hedef IP ve PORT'u kullanarak fuzzer'ı çalıştırın

```shell-session
M1R4CKCK@htb[/htb]$ python3 api_fuzzer.py http://IP:PORT

[-] Invalid endpoint: http://localhost:8000/~webmaster (Status code: 404)
[-] Invalid endpoint: http://localhost:8000/~www (Status code: 404)

Fuzzing completed.
Total requests: 4730
Failed requests: 0
Retries: 0
Status code counts:
404: 4727
200: 2
405: 1
Found valid endpoints:
- http://localhost:8000/cz...
- http://localhost:8000/docs
Unusual status codes:
405: http://localhost:8000/items
```

Fuzzer, birçok geçersiz endpointi (404 Not Found hatası döndüren) tespit eder.

İki geçerli endpoint keşfedilir:

- /cz...: Bu, API dokümantasyonunda yer almayan belgelenmemiş bir endpointtir.
- /docs: Bu, belgelenmiş Swagger UI endpointidir.
- /items için 405 Method Not Allowed cevabı, bu endpoint'e erişim için yanlış bir HTTP yöntemi kullanıldığını (örneğin, POST yerine GET isteği yapılmış) gösterir.

Belgelenmemiş endpoint'i curl aracılığıyla keşfedebiliriz ve bir bayrak döndürecektir:

```shell-session
M1R4CKCK@htb[/htb]$ curl http://localhost:8000/cz...

{"flag":"<snip>"}
```

Endpoint'lerin keşfedilmesinin yanı sıra, fuzzing bu endpoint'lerin kabul ettiği parametrelere de uygulanabilir. Parametrelere beklenmedik değerler sistematik olarak enjekte edilerek, hatalar, çöküşler veya beklenmedik davranışlar tetiklenebilir ve bu da geniş bir güvenlik açığı yelpazesi açığa çıkarabilir. Örneğin, aşağıdaki senaryoları düşünün:

- **Bozuk Nesne Seviyesi Yetkilendirme (Broken Object-Level Authorization)**: Fuzzing, parametre değerlerini manipüle ederek belirli nesnelere veya kaynaklara yetkisiz erişimin sağlanabileceği durumları açığa çıkarabilir.
- **Bozuk Fonksiyon Seviyesi Yetkilendirme (Broken Function-Level Authorization)**: Fuzzing, parametreleri manipüle ederek yetkisiz fonksiyon çağrılarının yapılabileceği durumları keşfedebilir, bu da saldırganların yapamayacakları işlemleri gerçekleştirmelerini sağlar.
- **Sunucu Tarafı İstek Sahteciliği (SSRF - Server-Side Request Forgery)**: Parametrelere kötü niyetli değerler enjekte edilerek, sunucunun, dahili veya harici kaynaklara istemeden istek göndermesi sağlanabilir, bu da hassas bilgilerin açığa çıkmasına veya daha fazla saldırıların yapılmasına yol açabilir.

Bu ve diğer web API güvenlik açıkları ve saldırıları hakkında daha fazla bilgi için [API Saldırıları modülüne başvurabilirsiniz.](https://academy.hackthebox.com/module/details/268) Bu riskleri anlamak, güvenli ve dayanıklı API'ler inşa etmek için çok önemlidir.

Soru  : Api fuzzer'ın tanımladığı endpoint tarafından döndürülen değer nedir?

Cevap : 

![Pasted image 20241222140034.png](/img/user/resimler/Pasted%20image%2020241222140034.png)

![Pasted image 20241222140046.png](/img/user/resimler/Pasted%20image%2020241222140046.png)

![Pasted image 20241222140124.png](/img/user/resimler/Pasted%20image%2020241222140124.png)


# Skills Assessment

Bu Beceri Değerlendirmesini tamamlamak için, bu modül boyunca sergilenen çok sayıda araç ve tekniği uygulamanız gerekecektir. Tüm fuzzing işlemleri, Pwnbox'ta /usr/share/seclists/Discovery/Web-Content adresinde bulunan common.txt SecLists Kelime Listesi kullanılarak veya SecLists GitHub aracılığıyla tamamlanabilir.


Soru : Değerlendirmedeki tüm adımları tamamladıktan sonra, HTB{...} biçiminde bir bayrak içeren bir sayfa ile karşılaşacaksınız. Bu bayrak nedir?

Cevap : 

Fuzz ile admin dizini bulduk sonra . 

![Pasted image 20241222142324.png](/img/user/resimler/Pasted%20image%2020241222142324.png)

![Pasted image 20241222142331.png](/img/user/resimler/Pasted%20image%2020241222142331.png)

![Pasted image 20241222142429.png](/img/user/resimler/Pasted%20image%2020241222142429.png)

![Pasted image 20241222142601.png](/img/user/resimler/Pasted%20image%2020241222142601.png)

-fs 58 ekleyelim. 

![Pasted image 20241222142641.png](/img/user/resimler/Pasted%20image%2020241222142641.png)

![Pasted image 20241222142725.png](/img/user/resimler/Pasted%20image%2020241222142725.png)

![Pasted image 20241222143002.png](/img/user/resimler/Pasted%20image%2020241222143002.png)

![Pasted image 20241222143025.png](/img/user/resimler/Pasted%20image%2020241222143025.png)
* fuzzing_fun.htb'ye hoş geldiniz!
* Bir sonraki başlangıç noktanız godeep dizinidir- ancak bu vhost üzerinde olabilir, olmayabilir, kim bilir.

![Pasted image 20241222143308.png](/img/user/resimler/Pasted%20image%2020241222143308.png)
![Pasted image 20241222143314.png](/img/user/resimler/Pasted%20image%2020241222143314.png)

![Pasted image 20241222143358.png](/img/user/resimler/Pasted%20image%2020241222143358.png)

![Pasted image 20241222143722.png](/img/user/resimler/Pasted%20image%2020241222143722.png)

![Pasted image 20241222144058.png](/img/user/resimler/Pasted%20image%2020241222144058.png)

![Pasted image 20241222144144.png](/img/user/resimler/Pasted%20image%2020241222144144.png)
