---
{"dg-publish":true,"permalink":"/active-directory/hashcat/"}
---


# Hashing vs. Encryption


### Hashing Nedir?

Hashing, bir metni, o metin için benzersiz olan bir stringe dönüştürme işlemidir. Bu işlemde, her giriş verisi için benzersiz bir çıktı üretilir.

### Hash Fonksiyonlarının Özelliği

Genellikle, bir hash fonksiyonu, verinin türü, uzunluğu veya boyutuna bakılmaksızın her zaman aynı uzunlukta hash'ler döndürür. Bu, hash fonksiyonlarının sabit uzunlukta çıktılar ürettiği anlamına gelir.

### Tek Yönlü İşlem

Hashing, tek yönlü bir işlemdir, yani orijinal metni hash'ten geri dönüştürmek imkansızdır. Bu, bir hash'ten düz metni yeniden oluşturmanın bir yolu olmadığını ifade eder.

### Hashing’in Kullanım Alanları

Hashing, birçok farklı amaç için kullanılabilir. Örneğin:

- **Dosya Bütünlüğü**: MD5 ve SHA256 algoritmaları, dosya bütünlüğünü doğrulamak için kullanılır.
- **Parola Hashleme**: PBKDF2 gibi algoritmalar, parolaları depolamadan önce güvenli hale getirmek için kullanılır.

### Anahtarlanabilir Hash Fonksiyonları

Bazı hash fonksiyonları ek bir parola (anahtar) kullanılarak oluşturulabilir. Bu fonksiyonlara “anahtarlı hash fonksiyonları” denir.

### HMAC (Hash-based Message Authentication Code)

HMAC, iletim sırasında belirli bir mesajın tahrif edilip edilmediğini doğrulamak için kullanılır. Bir sağlama toplamı görevi görür ve mesajın bütünlüğünü sağlar.

Hashing tek yönlü bir işlem olduğundan, ona saldırmanın tek yolu olası şifreleri içeren bir liste kullanmaktır. Bu listedeki her parola hashlenir ve orijinal hash ile karşılaştırılır.

Örneğin, Unix sistemlerinde şifreleri korumak için başlıca dört farklı algoritma kullanılabilir. Bunlar SHA-512, Blowfish, BCrypt ve Argon2'dir. Bu algoritmalar Linux, BSD ve Solaris gibi Unix işletim sistemlerinde mevcuttur.

SHA-512 uzun bir karakter dizisini bir hash değerine dönüştürür. Hızlı ve etkilidir, ancak saldırganın orijinal şifreleri yeniden oluşturmak için önceden hesaplanmış bir tablo kullandığı birçok Rainbow Table saldırısı vardır.

Buna karşılık Blowfish, bir şifreyi bir anahtarla şifreleyen simetrik bir blok şifre algoritmasıdır. SHA-512'den daha güvenlidir ancak aynı zamanda çok daha yavaştır.

BCrypt, potansiyel saldırganların parolaları tahmin etmesini veya rainbow table saldırıları gerçekleştirmesini zorlaştırmak için yavaş bir hash fonksiyonu kullanır.

Hash'lerin brute-forcing'ine karşı kullanılan bir koruma “ saltlama” dır. Salt, hash işleminden önce düz metne eklenen rastgele bir veri parçasıdır. Bu, hesaplama süresini artırır ancak brute force'u tamamen engellemez.

“p@ssw0rd” düz metin parola değerini ele alalım. Bunun MD5 hash'i aşağıdaki gibi hesaplanabilir:

```shell-session
M1R4CKCK@htb[/htb]$ echo -n "p@ssw0rd" | md5sum

0f359740bd1cda994f8b55330c86d845
```
Şimdi, “123456” gibi rastgele bir saltın kullanıldığını ve düz metne eklendiğini varsayalım.

```shell-session
M1R4CKCK@htb[/htb]$ echo -n "p@ssw0rd123456" | md5sum

f64c413ca36f5cfe643ddbec4f7d92d0
```
Bu yöntem kullanılarak, önceden hesaplanmış herhangi bir listede bulunmayan tamamen yeni bir hash oluşturulmuştur. Bu hashi kırmaya çalışan bir saldırganın hashi hesaplamadan önce bu salt'ı eklemek için fazladan zaman harcaması gerekecektir.

MD5 gibi bazı hash fonksiyonları, iki düz metin kümesinin aynı hash'i üretebildiği çarpışmalara karşı da savunmasızdır.

## Encryption
Encryption, verileri orijinal içeriğe erişilemeyecek bir formata dönüştürme işlemidir. Hashing'in aksine, şifreleme tersine çevrilebilir, yani şifreli metnin (şifrelenmiş veri) şifresini çözmek ve orijinal içeriği elde etmek mümkündür. Şifreleme şifrelerinin bazı klasik örnekleri Sezar şifresi, Boacon şifresi ve İkame şifresidir. Şifreleme algoritmaları iki tiptir: Simetrik ve Asimetrik.

## Symmetric Encryption
Simetrik algoritmalar verileri şifrelemek için bir key veya secret kullanır ve deşifre etmek için aynı anahtarı kullanır. Simetrik şifrelemenin temel bir örneği XOR'dur.
```shell-session
M1R4CKCK@htb[/htb]$ python3

Python 3.8.3 (default, May 14 2020, 11:03:12) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import xor
>>> xor("p@ssw0rd", "secret")
b'\x03%\x10\x01\x12D\x01\x01'
```
Yukarıdaki resimde düz metin p@ssw0rd, anahtar ise gizlidir. Anahtara sahip olan herkes şifreli metnin şifresini çözebilir ve düz metni elde edebilir.
```shell-session
M1R4CKCK@htb[/htb]$ python3

Python 3.8.3 (default, May 14 2020, 11:03:12) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from pwn import xor
>>> xor('\x03%\x10\x01\x12D\x01\x01', "secret")
b'p@ssw0rd'
```
Yukarıdaki çıktılardaki b, bir bayt dizisini ifade eder. Bu ayrım Python3 öncesinde yapılmıyordu.

Simetrik algoritmaların diğer bazı örnekleri AES, DES, 3DES ve Blowfish'tir. Bu algoritmalar key bruteforcing, frekans analizi, padding oracle attack gibi saldırılara karşı savunmasız olabilir.


## Asymmetric Encryption
Öte yandan, asimetrik algoritmalar anahtarı iki parçaya böler (yani, public ve private). public anahtar, bazı bilgileri şifrelemek ve sahibine güvenli bir şekilde iletmek isteyen herkese verilebilir. Anahtar sahibi daha sonra içeriğin şifresini çözmek için private anahtarını kullanır. Asimetrik algoritmaların bazı örnekleri RSA, ECDSA ve Diffie-Hellman'dır.

Asimetrik şifrelemenin öne çıkan kullanımlarından biri Secure Sockets Layer (SSL) biçimindeki Hypertext Transfer Protocol Secure (HTTPS) protokolüdür. Bir istemci HTTPS web sitesi barındıran bir sunucuya bağlandığında, bir ortak anahtar değişimi gerçekleşir. Client'ın tarayıcısı sunucuya gönderilen her türlü veriyi şifrelemek için bu public anahtarı kullanır. Sunucu, gelen trafiği işleme servisine aktarmadan önce şifresini çözer.

Soru : 'HackTheBox123!' parolasının MD5 hash'ini oluşturun.
Cevap : 
![Pasted image 20241015001116.png](/img/user/resimler/Pasted%20image%2020241015001116.png)

Soru : 'academy' anahtarını kullanarak 'opens3same' şifresinin XOR şifreli metnini oluşturun. (Cevap formatı: \x00\x00\x00\....)
Cevap : Güncellemeleri yap ve pip ile pwn'i indir 
![Pasted image 20241015002300.png](/img/user/resimler/Pasted%20image%2020241015002300.png)



### Hash'leri Tanımlama
Çoğu hash algoritması sabit uzunlukta hashler üretir. Belirli bir hash'in uzunluğu, hash'in hangi algoritma ile oluşturulduğunu belirlemek için kullanılabilir. Örneğin, 32 karakter uzunluğundaki bir hash, MD5 veya NTLM hash'i olabilir.

Bazen hash'ler belirli formatlarda saklanır. Örneğin, hash:salt veya $id$salt$hash.

2fc5a684737ce1bf7b3b239df432416e0dd07357:2014 hash'i, 2014 salt değerine sahip bir SHA1 hash'idir.

(6$vb1tLY1qiY$M.1ZCqKtJBxBtZm1gRi8Bbkn39KU0YJW1cuMFzTRANcNKFKR4RmAQVk4rqQCkaJT6wXqjUkFcA/qNxLyqW.U/) hash'i $ ile sınırlandırılmış üç alan içerir, burada ilk alan kimliktir, örn, 6. Bu, hashing için kullanılan algoritma türünü tanımlamak için kullanılır. Aşağıdaki liste bazı id'leri ve bunlara karşılık gelen algoritmaları içermektedir.

```shell-session
$1$  : MD5
$2a$ : Blowfish
$2y$ : Blowfish, with correct handling of 8 bit characters
$5$  : SHA256
$6$  : SHA512
```

Bir sonraki alan olan vb1tLY1qiY, hashleme sırasında kullanılan salt'tır ve son alan ise gerçek hash'tir.

Açık ve kapalı kaynak yazılımlar birçok farklı hash biçimi kullanır. Örneğin, Apache web sunucusu hashlerini $apr1$71850310$gh9m4xcAn3MGxogwX/ztb. biçiminde saklarken, WordPress hashleri $P$984478476IagS59wHZvyQMArzfx58u. biçiminde saklar.

## Hashid
[Hashid](https://github.com/psypanda/hashID), çeşitli hash türlerini tespit etmek için kullanılabilen bir Python aracıdır. Yazım sırasında, hashid 200'den fazla benzersiz hash türünü tanımlamak için kullanılabilir ve diğerleri için en iyi çaba tahminini yapar, ancak yine de daraltmak için biraz ek çalışma gerektirir. Desteklenen hash'lerin tam listesini [burada](https://github.com/psypanda/hashID/blob/master/doc/HASHINFO.xlsx) bulabilirsiniz. Pip kullanılarak kurulabilir.

```shell-session
M1R4CKCK@htb[/htb]$ pip install hashid
```

Hash'ler komut satırı argümanları olarak veya bir file kullanılarak sağlanabilir.

```shell-session
M1R4CKCK@htb[/htb]$ hashid '$apr1$71850310$gh9m4xcAn3MGxogwX/ztb.'

Analyzing '$apr1$71850310$gh9m4xcAn3MGxogwX/ztb.'
[+] MD5(APR) 
[+] Apache MD5
```

```shell-session
M1R4CKCK@htb[/htb]$ hashid hashes.txt 

--File 'hashes.txt'--
Analyzing '2fc5a684737ce1bf7b3b239df432416e0dd07357:2014'
[+] SHA-1 
[+] Double SHA-1 
[+] RIPEMD-160 
[+] Haval-160 
[+] Tiger-160 
[+] HAS-160 
[+] LinkedIn 
[+] Skein-256(160) 
[+] Skein-512(160) 
[+] Redmine Project Management Web App 
] SMF ≥ v1.1 
Analyzing 'P$984478476IagS59wHZ$vy[+QMArzfx58u.'
[+] Wordpress ≥ v2.6.2 
[+] Joomla ≥ v2.5.18 
[+] PHPass' Portable Hash 
--End of file 'hashes.txt'--
```

Biliniyorsa hashid, hash türünü belirleyebiliyorsa -m bayrağı ile ilgili ==Hashcat hash modunu== da sağlayabilir.

```shell-session
M1R4CKCK@htb[/htb]$ hashid '$DCC2$10240#tom#e4e938d12fe5974dc42a90120bd9c90f' -m
Analyzing '$DCC2$10240#tom#e4e938d12fe5974dc42a90120bd9c90f'
[+] Domain Cached Credentials 2 [Hashcat Mode: 2100
```


### Context is Important
Elde edilen hash'e dayanarak algoritmayı tanımlamak her zaman mümkün değildir. Yazılıma bağlı olarak, plaintext birden fazla şifreleme turundan ve salting dönüşümünden geçerek kurtarılmasını zorlaştırabilir.

hashid'in sağlanan hash türü için en iyi çabayı göstermek üzere regex kullandığına dikkat etmek önemlidir. Çoğu zaman hashid belirli bir hash için birçok olasılık sunacaktır ve belirli bir hash'i tanımlamak için yine de belirli bir miktar tahminde bulunmak zorunda kalacağız. Bu bir CTF sırasında olabilir, ancak genellikle bir sızma testi sırasında tanımlamak istediğimiz hash türü hakkında bazı bağlamlara sahibiz. Bir Active Directory saldırısı yoluyla mı yoksa bir Windows host'undan mı elde edildi? Bir SQL enjeksiyonu açığından başarılı bir şekilde yararlanılarak mı elde edildi? Bir hash'in nereden geldiğini bilmek, hash türünü ve dolayısıyla onu kırmayı denemek için gerekli Hashcat hash modunu daraltmamıza büyük ölçüde yardımcı olacaktır. Hashcat, hash modlarını örnek hash'lerle eşleştiren mükemmel bir [referans](https://hashcat.net/wiki/doku.php?id=example_hashes) sağlar. Bu referans, bir sızma testi sırasında karşı karşıya olduğumuz hash türünü ve bunu Hashcat'e iletmek için gereken ilişkili hash modunu belirlemek için çok değerlidir.

Örneğin, a2d1f7b7a1862d0d4a52644e72d59df5:500:lp@trash-mail.com hash'ini hashid'e geçirmek bize çeşitli olasılıklar verecektir:

```shell-session
M1R4CKCK@htb[/htb]$ hashid 'a2d1f7b7a1862d0d4a52644e72d59df5:500:lp@trash-mail.com'

Analyzing 'a2d1f7b7a1862d0d4a52644e72d59df5:500:lp@trash-mail.com'
[+] MD5 
[+] MD4 
[+] Double MD5 
[+] LM 
[+] RIPEMD-128 
[+] Haval-128 
[+] Tiger-128 
[+] Skein-256(128) 
[+] Skein-512(128) 
[+] Lotus Notes/Domino 5 
[+] Skype 
[+] Lastpass 
```

Bununla birlikte, Hashcat örnek hash referansına hızlı bir bakış, bunun gerçekten de 6800 hash modu olan bir Lastpass hash olduğunu belirlememize yardımcı olacaktır. Bağlam, değerlendirmeler sırasında önemlidir ve hash türlerini belirlemeye çalışan herhangi bir araçla çalışırken dikkate alınması önemlidir.


Soru : Aşağıdaki hash'i tanımlayın: $S$D34783772bRXEx1aCsvY.bqgaaSu75XmVlKrW9Du8IQlvxHlmzLc
Cevap : Drupal > v7.x

![Pasted image 20241015114940.png](/img/user/resimler/Pasted%20image%2020241015114940.png)

![Pasted image 20241015114956.png](/img/user/resimler/Pasted%20image%2020241015114956.png)


### Hashcat'e Genel Bakış

### Hashcat
[Hashcat](https://hashcat.net/hashcat/) popüler bir açık kaynak şifre kırma aracıdır.

Hashcat wget kullanılarak [web sitesinden](https://hashcat.net/hashcat/) indirilebilir ve daha sonra komut satırı üzerinden 7z (7-Zip dosya arşivleyici) kullanılarak açılabilir. Tam yardım menüsü hashcat -h yazarak görüntülenebilir. Hashcat'in yazım sırasındaki en son sürümü 6.1.1 sürümüdür. Sürüm 6.0.0, sürüm 5.x'e göre çeşitli geliştirmeler sunan önemli bir sürümdü. Değişikliklerden bazıları performans iyileştirmeleri ve 51 yeni algoritma (veya karma modları olarak da bilinen desteklenen karma türleri) içermekte olup, yazım sırasında toplam 320'den fazla desteklenen algoritma bulunmaktadır. Ayrıca Hashcat'in web sitesinden Windows ve Unix/Linux sistemleri için en son sürümün bağımsız bir ikili dosyasını indirebilir veya kaynaktan derleyebiliriz.

Hashcat ekibi, resmi GitHub [deposu](https://github.com/hashcat/hashcat) dışında herhangi bir paketin bakımını yapmamaktadır. Diğer tüm depolar (yani apt'den yükleme), güncel tutulması için 3. tarafa bırakılmıştır. En son sürüm her zaman doğrudan GitHub depolarından edinilebilir ve kaynaktan yüklenebilir. Amaçlarımız doğrultusunda, yazım sırasında en son sürümü (6.1.1) kullanan Pwnbox içinde kurulumu göstereceğiz.

#### Hashcat Installation
```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install hashcat
M1R4CKCK@htb[/htb]$ hashcat -h
```

Klasör hem Windows hem de Linux için 64 bit binary dosyalar içerir. Atak modu ve hash türünü belirtmek için -a ve -m argümanları kullanılır. Hashcat aşağıdaki saldırı modlarını destekler:

![Pasted image 20241015120013.png](/img/user/resimler/Pasted%20image%2020241015120013.png)

Hash türü değeri, kırılacak hash'in algoritmasına dayanır. Hash türlerinin ve bunlara karşılık gelen örneklerin tam bir listesi [burada](https://hashcat.net/wiki/doku.php?id=example_hashes) bulunabilir. Tablo, belirli bir hash türü için numarayı hızlı bir şekilde tanımlamaya yardımcı olur. Örnek hash'lerin listesini aşağıdaki komutu kullanarak komut satırı üzerinden de görüntüleyebilirsiniz:

#### Hashcat - Example Hashes
```shell-session
M1R4CKCK@htb[/htb]$ hashcat --example-hashes | less

hashcat (v6.1.1) starting...

MODE: 0
TYPE: MD5
HASH: 8743b52063cd84097a65d1633f5c74f5
PASS: hashcat

MODE: 10
TYPE: md5($pass.$salt)
HASH: 3d83c8e717ff0e7ecfe187f088d69954:343141
PASS: hashcat

MODE: 11
TYPE: Joomla < 2.5.18
HASH: b78f863f2c67410c41e617f724e22f34:89384528665349271307465505333378
PASS: hashcat

MODE: 12
TYPE: PostgreSQL
HASH: 93a8cf6a7d43e3b5bcd2dc6abb3e02c6:27032153220030464358344758762807
PASS: hashcat

MODE: 20
TYPE: md5($salt.$pass)
HASH: 57ab8499d08c59a7211c77f557bf9425:4247
PASS: hashcat

<SNIP>
```

Listede gezinebilir ve çıkmak için q tuşuna basabilirsiniz.

Belirli bir hash türü için kıyaslama testi (veya performans testi) -b bayrağı kullanılarak gerçekleştirilebilir.


#### Hashcat - Benchmark
```shell-session
M1R4CKCK@htb[/htb]$ hashcat -b -m 0
hashcat (v6.1.1) starting in benchmark mode...

Benchmarking uses hand-optimized kernel code by default.
You can use it in your cracking session by setting the -O option.
Note: Using optimized kernel code limits the maximum supported password length.
To disable the optimized kernel code in benchmark mode, use the -w option.

OpenCL API (OpenCL 1.2 pocl 1.5, None+Asserts, LLVM 9.0.1, RELOC, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
=============================================================================================================================
* Device #1: pthread-Intel(R) Core(TM) i7-5820K CPU @ 3.30GHz, 4377/4441 MB (2048 MB allocatable), 6MCU

Benchmark relevant options:
===========================
* --optimized-kernel-enable

Hashmode: 0 - MD5


Speed.#1.........:   449.4 MH/s (12.84ms) @ Accel:1024 Loops:1024 Thr:1 Vec:8

Started: Fri Aug 28 21:52:35 2020
Stopped: Fri Aug 28 21:53:25 2020
```

Örneğin, belirli bir CPU'da MD5 için hash oranı 450,7 MH/s olarak bulunmuştur.

Ayrıca hashcat -b komutunu çalıştırarak tüm hash modları için kıyaslamalar yapabiliriz.


#### Hashcat - Optimizations
Hashcat'in hızı optimize etmek için iki ana yolu vardır:

==Optimized Kernels==: Bu, belgelere göre optimize edilmiş kernelleri etkinleştir (parola uzunluğunu sınırlar) anlamına gelen -O bayrağıdır. Sihirli parola uzunluğu genellikle 32'dir ve çoğu kelime listesi bu sayıya bile ulaşmaz. Bu, tahmini süreyi günlerden saatlere kadar sürebilir, bu nedenle her zaman önce -O ile çalıştırmanız ve GPU'nuz boşta ise -O olmadan tekrar çalıştırmanız önerilir.

==Workload (İş Yükü)== : Bu, belgelere göre belirli bir iş yükü profilini etkinleştir anlamına gelen -w bayrağıdır. Varsayılan sayı 2'dir, ancak Hashcat çalışırken bilgisayarınızı kullanmak istiyorsanız, bunu 1 olarak ayarlayın. Bilgisayarın yalnızca Hashcat çalıştırmasını planlıyorsanız, bu 3 olarak ayarlanabilir.

Force kullanımından kaçınılması gerektiğine dikkat etmek önemlidir. Bu, Hashcat'in belirli hostlarda çalışmasını sağlıyor gibi görünse de, aslında güvenlik kontrollerini devre dışı bırakır, uyarıları susturur ve aracın geliştiricilerinin engelleyici olarak kabul ettiği sorunları atlar. Bu sorunlar yanlış pozitiflere, yanlış negatiflere, arızalara vb. yol açabilir. Araç, komutunuza --force ekleyerek çalıştırmaya zorlamadan düzgün çalışmıyorsa, temel nedeni (örneğin, bir sürücü sorunu) gidermeliyiz. Aracın geliştiricileri tarafından --force kullanılması önerilmez ve yalnızca deneyimli kullanıcılar veya geliştiriciler tarafından kullanılmalıdır.


Soru :  Cisco-ASA MD5 hash türünün hash modu nedir?
Cevap  : 2410

```shell-session
┌─[✗]─[root@htb-qu30f99hjo]─[/home/htb-ac-961525]
└──╼ #hashcat --example-hashes | grep -C 20 Cisco-ASA
  Plaintext.Encoding..: ASCII, HEX

Hash mode #2400
  Name................: Cisco-PIX MD5
  Category............: Operating System
  Slow.Hash...........: No
  Password.Len.Min....: 0
  Password.Len.Max....: 256
  Kernel.Type(s)......: optimized
  Example.Hash.Format.: plain
  Example.Hash........: dRRVnUmUHXOTt9nk
  Example.Pass........: hashcat
  Benchmark.Mask......: ?b?b?b?b?b?b?b
  Autodetect.Enabled..: Yes
  Self.Test.Enabled...: Yes
  Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

Hash mode #2410
  Name................: Cisco-ASA MD5
  Category............: Operating System
  Slow.Hash...........: No
  Password.Len.Min....: 0
  Password.Len.Max....: 256
  Salt.Type...........: Generic
  Salt.Len.Min........: 0
  Salt.Len.Max........: 256
  Kernel.Type(s)......: optimized
  Example.Hash.Format.: plain
  Example.Hash........: YjDBNr.A0AN7DA8s:4684
  Example.Pass........: hashcat
  Benchmark.Mask......: ?b?b?b?b?b?b?b
  Autodetect.Enabled..: Yes
  Self.Test.Enabled...: Yes
  Potfile.Enabled.....: Yes
  Custom.Plugin.......: No
  Plaintext.Encoding..: ASCII, HEX

Hash mode #2500
  Name................: WPA-EAPOL-PBKDF2

```


# Dictionary Attack
Hashcat, kırmaya çalıştığınız hash türüne ve şifrenin karmaşıklığına bağlı olarak farklı uygulamaları olan 5 farklı saldırı moduna sahiptir. En basit ama son derece etkili saldırı türü sözlük saldırısıdır. Kullanıcıları parola olarak çok az karmaşıklığa sahip ya da hiç karmaşıklığa sahip olmayan yaygın sözcük ve ifadeleri seçen zayıf parola politikalarına sahip kuruluşlarla karşılaşmak nadir değildir. SplashData adlı kuruluş, sızdırılan milyonlarca şifrenin analizine dayanarak 2020'nin en yaygın 5 şifresini aşağıdaki gibi sıralamıştır:


#### Password List - Top 5 (2020)
```shell-session
123456
123456789
qwerty
password
1234567
```
Kullanıcıları güvenlik farkındalığı konusunda eğitmelerine rağmen, bir kuruluş zayıf şifrelere izin veriyorsa, kullanıcılar genellikle kolaylık olsun diye bir şifre seçecektir.

Bu parolalar, bu tür bir saldırıyı gerçekleştirmek için kullanılan herhangi bir sözlük dosyasında görünecektir. Parola listelerini elde etmek için SecLists, geniş bir parola listesi koleksiyonu ve çoğu sızma testi Linux dağıtımında bulunan rockyou.txt kelime listesi gibi birçok kaynak vardır.

Ayrıca CrackStation'ın 1.493.677.782 kelime içeren ve 15 GB boyutunda olan Password Cracking Dictionary gibi büyük kelime listeleri de bulabiliriz. İhtiyaçlara ve bilgi işlem gereksinimlerine bağlı olarak, birden fazla ihlalden ve şifre dökümlerinden elde edilen açık metin şifrelerinden oluşan, bazıları 40 GB'ın üzerinde boyuta sahip çok daha büyük kelime listeleri vardır. Bunlar, etkileşiminizin başarısı için kritik olan tek bir parolayı güçlü bir GPU'da kırmaya çalışırken veya NTDS.dit dosyasındaki NTLM parola hash'lerinin mümkün olduğunca çoğunu kırmaya çalışarak bir Active Directory ortamındaki tüm kullanıcı parolalarının domain parola analizini gerçekleştirirken son derece yararlı olabilir.


### Straight veya Sözlük Saldırısı
Adından da anlaşılacağı gibi, bu saldırı bir kelime listesinden okur ve verilen hash'leri kırmaya çalışır. Sözlük saldırıları, hedef kuruluşun zayıf parolalar kullandığını biliyorsanız veya sadece bazı kırma girişimlerini oldukça hızlı bir şekilde yapmak istiyorsanız kullanışlıdır. Bu saldırının tamamlanması, bu modülün ilerleyen bölümlerinde ele alınan daha karmaşık saldırılara kıyasla genellikle daha hızlıdır. Temel sözdizimi şöyledir:


#### Hashcat - Syntax

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 0 -m <hash type> <hash file> <wordlist>
```

Örneğin, aşağıdaki komutlar rockyou.txt kelime listesini kullanarak bir SHA256 hash'i kıracaktır.

```shell-session
M1R4CKCK@htb[/htb]$ echo -n '!academy' | sha256sum | cut -f1 -d' ' > sha256_hash_example
M1R4CKCK@htb[/htb]$ hashcat -a 0 -m 1400 sha256_hash_example /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...

<SNIP>

Dictionary cache built:
* Filename..: /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 2 secs

Approaching final keyspace - workload adjusted.  

006fc3a9613f3edd9f97f8e8a8eff3b899a2d89e1aabf33d7cc04fe0728b0fe6:!academy
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: SHA2-256
Hash.Target......: 006fc3a9613f3edd9f97f8e8a8eff3b899a2d89e1aabf33d7cc...8b0fe6
Time.Started.....: Fri Aug 28 21:58:44 2020 (4 secs)
Time.Estimated...: Fri Aug 28 21:58:48 2020 (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3383.5 kH/s (0.46ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 14344385/14344385 (100.00%)
Rejected.........: 0/14344385 (0.00%)
Restore.Point....: 14340096/14344385 (99.97%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: $HEX[216361726f6c796e] -> $HEX[042a0337c2a156616d6f732103]

Started: Fri Aug 28 21:58:05 2020
Stopped: Fri Aug 28 21:58:49 2020
```

Yukarıdaki örnekte, hash 4 saniyede kırılmıştır. Kırma hızı altta yatan donanıma, hash türüne ve parolanın karmaşıklığına bağlı olarak değişir.

[Blowfish](https://en.wikipedia.org/wiki/Blowfish_(cipher)) şifresini temel alan bir tür parola hash'i olan [Bcrypt](https://en.wikipedia.org/wiki/Bcrypt) gibi daha karmaşık bir hash'e bakalım. Rainbow table saldırılarından korunmak için bir salt kullanır ve algoritmanın birçok turu uygulanabilir, bu da hash'i büyük bir şifre kırma teçhizatıyla bile brute force saldırılarına karşı dirençli hale getirir.

Örneğin, aynı parolanın “!academy” bcrypt hash'ini ele alalım; bu hash, Blowfish algoritmasının 5 tur uygulanmasıyla $2a$05$ZdEkj8cup/JycBRn2CX.B.nIceCYR8GbPbCCg6RlD7uvuREexEbVy olacaktır. Aynı donanım üzerinde aynı kelime listesi ile çalıştırılan bu hash'in kırılması çok daha uzun sürmektedir.

Kırma işlemi sırasında herhangi bir zamanda, kırma işinin durumunu öğrenmek için “s” tuşuna basabilirsiniz, bu da rockyou.txt kelime listesindeki her şifreyi denemenin 1,5 saatten fazla süreceğini gösterir. Algoritmanın daha fazla turunun uygulanması kırma süresini katlanarak artıracaktır. Bcrypt gibi hash'ler söz konusu olduğunda, daha küçük, daha hedefli kelime listeleri kullanmak genellikle daha iyidir.


#### Hashcat - Status

```shell-session
[s]tatus [p]ause [b]ypass [c]heckpoint [q]uit => s

Session..........: hashcat
Status...........: Running
Hash.Name........: bcrypt $2*$, Blowfish (Unix)
Hash.Target......: $2a$05$ZdEkj8cup/JycBRn2CX.B.nIceCYR8GbPbCCg6RlD7uv...exEbVy
Time.Started.....: Mon Jun 22 19:43:40 2020 (3 mins, 10 secs)
Time.Estimated...: Mon Jun 22 21:20:28 2020 (1 hour, 33 mins)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:     2470 H/s (8.98ms) @ Accel:8 Loops:16 Thr:1 Vec:8
Recovered........: 0/1 (0.00%) Digests
Progress.........: 468576/14344385 (3.27%)
Rejected.........: 0/468576 (0.00%)
Restore.Point....: 468576/14344385 (3.27%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:16-32
Candidates.#1....: septiembre29 -> sep1101
```

Bu bölümde gördüğümüz gibi, sözlük saldırıları zayıf parolalar için çok etkili olabilir, ancak saldırının etkinliği hedeflenen hash türüne de bağlıdır. Bazı zayıf parola türlerinin kırılması sadece kullanılan hash algoritmasına bağlı olarak çok daha zor olabilir. Bu, daha güçlü bir hash algoritması kullanan zayıf bir parolanın daha “güvenli” olduğu anlamına gelmez. Parola kırma donanımı çeşitlilik gösterir ve çok sayıda GPU'ya sahip “kırma donanımları”, tek bir CPU'da saatler veya günler sürecek bir parola karmasını kısa sürede halledebilir.

Soru :  rockyou.txt kelime listesini kullanarak aşağıdaki hash'i kırın: 0c352d5b2f45217c57bef9f8452ce376

```shell-session
$ hashcat -a 0 -m 0 0c352d5b2f45217c57bef9f8452ce376 /home/htb-ac-961525/Desktop/rockyou.txt 
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-haswell-AMD EPYC 7543 32-Core Processor, skipped

OpenCL API (OpenCL 2.1 LINUX) - Platform #2 [Intel(R) Corporation]
==================================================================
* Device #2: AMD EPYC 7543 32-Core Processor, 3919/7902 MB (987 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache building /home/htb-ac-961525/Desktop/rockyou.txt: 33553434 byteDictionary cache built:
* Filename..: /home/htb-ac-961525/Desktop/rockyou.txt
* Passwords.: 14344392
* Bytes.....: 139921507
* Keyspace..: 14344385
* Runtime...: 1 sec

0c352d5b2f45217c57bef9f8452ce376:cricket1                 
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 0c352d5b2f45217c57bef9f8452ce376
Time.Started.....: Tue Oct 15 05:43:07 2024 (0 secs)
Time.Estimated...: Tue Oct 15 05:43:07 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/home/htb-ac-961525/Desktop/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1775.0 kH/s (0.14ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 8192/14344385 (0.06%)
Rejected.........: 0/8192 (0.00%)
Restore.Point....: 6144/14344385 (0.04%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: horoscope -> whitetiger

Started: Tue Oct 15 05:43:02 2024
Stopped: Tue Oct 15 05:43:08 2024


```


### Kombinasyon Saldırısı

Kombinasyon saldırısı modları girdi olarak iki kelime listesi alır ve bunlardan kombinasyonlar oluşturur. Bu saldırı kullanışlıdır çünkü kullanıcıların daha güçlü bir parola oluşturduğunu düşünerek iki veya daha fazla kelimeyi bir araya getirmesi nadir değildir, örneğin welcomehome veya hotelcalifornia.

Bu saldırıyı göstermek için aşağıdaki kelime listelerini ele alalım:

```shell-session
M1R4CKCK@htb[/htb]$ cat wordlist1

super
world
secret
```

```shell-session
M1R4CKCK@htb[/htb]$ cat wordlist2

hello
password
```

Bu iki kelime listesi verilirse Hashcat aşağıdaki gibi tam olarak 3 x 2 = 6 kelime üretecektir:

```shell-session
M1R4CKCK@htb[/htb]$ awk '(NR==FNR) { a[NR]=$0 } (NR != FNR) { for (i in a) { print $0 a[i] } }' file2 file1

superhello
superpassword
worldhello
wordpassword
secrethello
secretpassword
```

Bu, Hashcat ile --stdout bayrağı kullanılarak da yapılabilir; bu, hata ayıklama amacıyla ve aracın işleri nasıl ele aldığını görmek için çok yararlı olabilir.

Aşağıdaki örnekte aynı iki dosya verildiğinde Hashcat'in ne üreteceğini görebiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 1 --stdout file1 file2
superhello
superpassword
worldhello
worldpassword
secrethello
secretpassword
```

#### Hashcat - Syntax
Kombinasyon saldırısı için sözdizimi şöyledir:

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 1 -m <hash type> <hash file> <wordlist1> <wordlist2>
```
Bu saldırı, kelime listelerini kullanırken daha fazla esneklik ve özelleştirme sağlar.

Bu örneği pratikte görelim. İlk olarak, secretpassword parolasının md5'ini oluşturun.

```shell-session
M1R4CKCK@htb[/htb]$ echo -n 'secretpassword' | md5sum | cut -f1 -d' '  > combination_md5

2034f6e32958647fdff75d265b455ebf
```

Daha sonra, kombinasyon saldırı moduyla yukarıdaki iki kelime listesini kullanarak Hashcat'i hash'e karşı çalıştıralım.

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 1 -m 0 combination_md5 wordlist1 wordlist2

hashcat (v6.1.1) starting...
<SNIP>

Dictionary cache hit:
* Filename..: wordlist1
* Passwords.: 3
* Bytes.....: 19
* Keyspace..: 6

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.  

2034f6e32958647fdff75d265b455ebf:secretpassword  
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: 2034f6e32958647fdff75d265b455ebf
Time.Started.....: Fri Aug 28 22:05:51 2020, (0 secs)
Time.Estimated...: Fri Aug 28 22:05:51 2020, (0 secs)
Guess.Base.......: File (wordlist1), Left Side
Guess.Mod........: File (wordlist2), Right Side
Speed.#1.........:       42 H/s (0.02ms) @ Accel:1024 Loops:2 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 6/6 (100.00%)
Rejected.........: 0/6 (0.00%)
Restore.Point....: 0/3 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-2 Iteration:0-2
Candidates.#1....: superhello -> secretpassword
```

Kombinasyon saldırıları cephaneliğimizde bulundurmamız gereken bir diğer güçlü araçtır. Yukarıda gösterildiği gibi, yalnızca iki kelimeyi birleştirmek bir parolayı mutlaka daha güçlü yapmaz.


### Alıştırma - Tamamlayıcı kelime listeleri

#### Tamamlayıcı kelime listesi #1
```shell-session
sunshine
happy
frozen
golden
```

#### Tamamlayıcı kelime listesi #2
```shell-session
hello
joy
secret
apple
```


Soru : Hashcat kombinasyon saldırısını kullanarak aşağıdaki md5 hash'inin açık metin şifresini bulun: 19672a3f042ae1b592289f8333bf76c5. Bu bölümün sonunda gösterilen tamamlayıcı kelime listelerini kullanın.

Cevap : ==frozenapple==
```shell-session
$ hashcat -a 1 -m 0 "19672a3f042ae1b592289f8333bf76c5" test1 test2
hashcat (v6.2.6) starting

OpenCL API (OpenCL 3.0 PoCL 3.1+debian  Linux, None+Asserts, RELOC, SPIR, LLVM 15.0.6, SLEEF, DISTRO, POCL_DEBUG) - Platform #1 [The pocl project]
==================================================================================================================================================
* Device #1: pthread-haswell-AMD EPYC 7543 32-Core Processor, skipped

OpenCL API (OpenCL 2.1 LINUX) - Platform #2 [Intel(R) Corporation]
==================================================================
* Device #2: AMD EPYC 7543 32-Core Processor, 3919/7902 MB (987 MB allocatable), 4MCU

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Dictionary cache built:
* Filename..: test1
* Passwords.: 4
* Bytes.....: 29
* Keyspace..: 4
* Runtime...: 0 secs

Dictionary cache built:
* Filename..: test2
* Passwords.: 4
* Bytes.....: 23
* Keyspace..: 4
* Runtime...: 0 secs

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates

Optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

ATTENTION! Pure (unoptimized) backend kernels selected.
Pure kernels can crack longer passwords, but drastically reduce performance.
If you want to switch to optimized kernels, append -O to your commandline.
See the above message to find out about the exact limits.

Watchdog: Hardware monitoring interface not found on your system.
Watchdog: Temperature abort trigger disabled.

Host memory required for this attack: 1 MB

Dictionary cache hit:
* Filename..: test1
* Passwords.: 4
* Bytes.....: 29
* Keyspace..: 16

The wordlist or mask that you are using is too small.
This means that hashcat cannot use the full parallel power of your device(s).
Unless you supply more work, your cracking speed will drop.
For tips on supplying more work, see: https://hashcat.net/faq/morework

Approaching final keyspace - workload adjusted.           

19672a3f042ae1b592289f8333bf76c5:frozenapple              
                                                          
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 0 (MD5)
Hash.Target......: 19672a3f042ae1b592289f8333bf76c5
Time.Started.....: Tue Oct 15 14:24:27 2024 (0 secs)
Time.Estimated...: Tue Oct 15 14:24:27 2024 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (test1), Left Side
Guess.Mod........: File (test2), Right Side
Speed.#2.........:    15461 H/s (0.01ms) @ Accel:512 Loops:4 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 16/16 (100.00%)
Rejected.........: 0/16 (0.00%)
Restore.Point....: 0/4 (0.00%)
Restore.Sub.#2...: Salt:0 Amplifier:0-4 Iteration:0-4
Candidate.Engine.: Device Generator
Candidates.#2....: sunshinehello -> goldenapple

Started: Tue Oct 15 14:24:21 2024
Stopped: Tue Oct 15 14:24:28 2024

```


# Mask Attack
Maske saldırıları, belirli bir kalıpla eşleşen sözcükler üretmek için kullanılır. Bu saldırı türü özellikle parola uzunluğu veya biçimi bilindiğinde kullanışlıdır. Bir maske statik karakterler, karakter aralıkları (örneğin [a-z] veya [A-Z0-9]) veya yer tutucular kullanılarak oluşturulabilir. Aşağıdaki listede bazı önemli placeholder'lar gösterilmektedir:

![Pasted image 20241015225551.png](/img/user/resimler/Pasted%20image%2020241015225551.png)

Yukarıdaki yer tutucular, özel yer tutucular için kullanılabilecek “-1” ila “-4” seçenekleri ile birleştirilebilir. Dört özel karakter kümesi yapılandırmak için kullanılabilecek bu dört komut satırı parametresinin her birinin ayrıntılı bir dökümü için buradaki Özel karakter kümeleri [bölümüne](https://hashcat.net/wiki/doku.php?id=mask_attack) bakın.

Bu kez “ILFREIGHT userid year” şemasına sahip şifreleri olan Inlane Freight şirketini düşünün, burada userid 5 karakter uzunluğundadır. “ILFREIGHT?l?l?l?l?l20[0-1]?d” maskesi, ‘?l ’in bir harf olduğu ve ‘20[0-1]?d ’nin 2000‘den 2019’a kadar tüm yılları içerdiği belirtilen düzene sahip şifreleri kırmak için kullanılabilir.

Bir hash oluşturmayı ve bu maskeyi kullanarak kırmayı deneyelim.


#### Creating MD5 hashes
```shell-session
M1R4CKCK@htb[/htb]$ echo -n 'ILFREIGHTabcxy2015' | md5sum | tr -d " -" > md5_mask_example_hash
```

Aşağıdaki örnekte, saldırı modu 3 ve MD5 için hash türü 0'dır.

#### Hashcat - Mask Attack

```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 3 -m 0 md5_mask_example_hash -1 01 'ILFREIGHT?l?l?l?l?l20?1?d'

hashcat (v6.1.1) starting...
<SNIP>

d53ec4d0b37bbf565b1e09d64834e1ae:ILFREIGHTabcxy2015
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: d53ec4d0b37bbf565b1e09d64834e1ae
Time.Started.....: Fri Aug 28 22:08:44 2020, (43 secs)
Time.Estimated...: Fri Aug 28 22:09:27 2020, (0 secs)
Guess.Mask.......: ILFREIGHT?l?l?l?l?l20?1?d [18]
Guess.Charset....: -1 01, -2 Undefined, -3 Undefined, -4 Undefined 
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  3756.3 kH/s (0.36ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 155222016/237627520 (65.32%)
Rejected.........: 0/155222016 (0.00%)
Restore.Point....: 155215872/237627520 (65.32%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: ILFREIGHTuisba2015 -> ILFREIGHTkmrff2015
```

- **`20`**: Sabit bir "20" sayısı.
- **`?1`**: "0" veya "1" (kullanıcı tarafından tanımlanmış).
- **`?d`**: 0-9 arası bir rakam.

“-1” seçeneği sadece 0 ve 1 ile bir yer tutucu belirtmek için kullanıldı. Hashcat CPU gücüyle hash'i 43 saniyede kırabildi. “--increment” bayrağı, ‘--increment-max’ bayrağı kullanılarak sağlanabilecek bir uzunluk sınırı ile maske uzunluğunu otomatik olarak artırmak için kullanılabilir.


Soru : Bir maske saldırısı kullanarak aşağıdaki MD5 karmasını kırın: 50a742905949102c961929823a2e8ca0. Aşağıdaki maskeyi kullanın: -1 02 'HASHCAT?l?l?l?l20?1?d'

Cevap : 

==hashcat -a 3 -m 0 '50a742905949102c961929823a2e8ca0' -1 02 'HASHCAT?l?l?l?l?l20?1?d'==

![Pasted image 20241015231900.png](/img/user/resimler/Pasted%20image%2020241015231900.png)


# Hybrid Mode
Hibrit mod, ince ayarlı bir kelime listesi oluşturmak için birden fazla modun birlikte kullanılabildiği birleştirici saldırısının bir varyasyonudur. Bu mod, çok özelleştirilmiş kelime listeleri oluşturarak çok hedefli saldırılar gerçekleştirmek için kullanılabilir. Özellikle kuruluşun parola politikasını veya genel parola sözdizimini bildiğinizde veya bunlar hakkında genel bir fikriniz olduğunda kullanışlıdır. Hibrit saldırı için saldırı modu “6 ”dır.

“football1$” gibi bir parolayı ele alalım. Aşağıdaki örnek, bir kelime listesinin bir maske ile birlikte nasıl kullanılabileceğini göstermektedir.

#### Creating Hybrid Hash

```shell-session
[!bash!]$ echo -n 'football1

Hashcat sözcük listesindeki sözcükleri okur ve verilen maskeye göre benzersiz bir dize ekler. Bu durumda, “?d?s” maskesi hashcat'e rockyou.txt kelime listesindeki her kelimenin sonuna bir rakam ve özel bir karakter eklemesini söyler.


#### Hashcat - Hybrid Attack using Wordlists

```shell-session
[!bash!]$ hashcat -a 6 -m 0 hybrid_hash /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt '?d?s'

hashcat (v6.1.1) starting...
<SNIP>

f7a4a94ff3a722bf500d60805e16b604:football1$      
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: f7a4a94ff3a722bf500d60805e16b604
Time.Started.....: Fri Aug 28 22:11:15 2020, (0 secs)
Time.Estimated...: Fri Aug 28 22:11:15 2020, (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt), Left Side
Guess.Mod........: Mask (?d?s) [2], Right Side
Guess.Queue.Base.: 1/1 (100.00%)
Guess.Queue.Mod..: 1/1 (100.00%)
Speed.#1.........:  5118.2 kH/s (11.56ms) @ Accel:768 Loops:82 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 755712/4733647050 (0.02%)
Rejected.........: 0/755712 (0.00%)
Restore.Point....: 0/14344385 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:82-164 Iteration:0-82
Candidates.#1....: 1234562= -> class083~
```
Saldırı modu “7”, belirli bir maske kullanarak kelimelere karakter eklemek için kullanılabilir. Aşağıdaki örnekte rockyou.txt kelime listesindeki her kelimeye bir önek eklemek için özel bir karakter kümesi kullanan bir maske gösterilmektedir. “-1 01” özel karakter kümesiyle ‘20?1?d’ özel karakter maskesi, kelime listesindeki her kelimeye çeşitli yıllar ekleyecektir (örneğin, 2010, 2011, 2012...).


### Başka bir Hibrit Hash Oluşturma
```shell-session
[!bash!]$ echo -n '2015football' | md5sum | tr -d " -" > hybrid_hash_prefix 
```


#### Hashcat - Hybrid Attack using Wordlists with Masks
```shell-session
[!bash!]$ hashcat -a 7 -m 0 hybrid_hash_prefix -1 01 '20?1?d' /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...
<SNIP> 

eac4fe196339e1b511278911cb77d453:2015football    
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: eac4fe196339e1b511278911cb77d453
Time.Started.....: Thu Nov 12 01:32:34 2020 (0 secs)
Time.Estimated...: Thu Nov 12 01:32:34 2020 (0 secs)
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt), Right Side
Guess.Mod........: Mask (20?1?d) [4], Left Side
Guess.Charset....: -1 01, -2 Undefined, -3 Undefined, -4 Undefined
Speed.#1.........:     8420 H/s (0.22ms) @ Accel:384 Loops:64 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 1280/286887700 (0.00%)
Rejected.........: 0/1280 (0.00%)
Restore.Point....: 0/20 (0.00%)
Restore.Sub.#1...: Salt:0 Amplifier:0-64 Iteration:0-64
Candidates.#1....: 2001123456 -> 2017charlie
```


Soru :  Aşağıdaki hash'i kırın: 978078e7845f2fb2e20399d9e80475bc1c275e06 “?d?s” maskesini kullanarak.
Cevap : 

==hashcat -a 6 -m 100 "978078e7845f2fb2e20399d9e80475bc1c275e06" rockyou.txt '?d?s'==

![Pasted image 20241016013605.png](/img/user/resimler/Pasted%20image%2020241016013605.png)


### Kişisel Sözcük Listeleri Oluşturma


### Wordlists
Bir değerlendirme sırasında, görevin başarısı için çok önemli olan bir veya daha fazla parola hash'i alabiliriz. En iyi çabalarımıza rağmen, bu hash'ler önceki bölümlerde ele alınan sözlük, kombinasyon, maske veya hibrit saldırılar kullanılarak yaygın kelime listeleri ile kırılamaz. Bu durumlarda hedefimize ulaşmak için özel, hedeflenmiş bir kelime listesi oluşturmak gerekebilir.

Bir wordlist'i iyileştirmek için zaman harcamak gerekir çünkü başarı oranı büyük ölçüde buna bağlıdır. Kelime listeleri çeşitli kaynaklardan elde edilebilir ve hedefe göre özelleştirilebilir ve kurallar kullanılarak daha da ince ayar yapılabilir. Parolalar, kullanıcı adları, dosya adları, yükler ve diğer birçok veri türü için kelime listeleri bulunabilir. SecLists deposu ayrıca kullanıcı adı numaralandırma parola tanımlama için yararlı birçok kelime listesi içerir.


### Creating Wordlists
Birçok açık kaynak aracı, gereksinimlerimize göre özelleştirilmiş parola kelime listeleri oluşturmaya yardımcı olur.


### Crunch
Crunch, belirli bir uzunluktaki kelimeler, sınırlı bir karakter seti veya belirli bir desen gibi parametrelere dayalı kelime listeleri oluşturabilir. Hem permütasyonlar hem de kombinasyonlar oluşturabilir.

Parrot OS'de varsayılan olarak yüklüdür ve burada bulunabilir. Crunch'ın genel sözdizimi aşağıdaki gibidir:

#### Crunch - Syntax
```shell-session
M1R4CKCK@htb[/htb]$ crunch <minimum length> <maximum length> <charset> -t <pattern> -o <output file>
```
“-t” seçeneği, oluşturulan parolalar için desen belirtmek için kullanılır. Kalıp, küçük harf karakterlerini temsil eden “@,” içerebilir, “,” (virgül) büyük harf karakterlerini ekler, “%” sayıları ekler ve “^” sembolleri ekler.


#### Crunch - Generate Word List
```shell-session
$ crunch 4 8 -o wordlist
```

Yukarıdaki komut, varsayılan karakter kümesini kullanarak 4 ila 8 karakter uzunluğunda sözcüklerden oluşan bir sözcük listesi oluşturur.

Inlane Freight kullanıcı şifrelerinin “ILFREIGHTYYYXXXX” şeklinde olduğunu varsayalım; burada “XXXX” harf içeren çalışan kimliği ve “YYYY” yıl olsun. Bu tür parolaların bir listesini oluşturmak için crunch'ı kullanabiliriz.


### Crunch - Kalıp Kullanarak Kelime Listesi Oluşturma
```shell-session
$ crunch 17 17 -t ILFREIGHT201%@@@@ -o wordlist
```

“ILFREIGHT201%@@@@” kalıbı, 2010-2019 yıllarını ve ardından dört harfi içeren sözcükler oluşturacaktır. Buradaki uzunluk 17'dir ve tüm kelimeler için sabittir.

Bir kullanıcının doğum tarihinin 10/03/1998 olduğunu biliyorsak (sosyal medya vb. aracılığıyla), bunu şifresine ekleyebilir ve ardından bir harf dizisi ekleyebiliriz. Crunch bu tür kelimelerden oluşan bir kelime listesi oluşturmak için kullanılabilir. “-d” seçeneği tekrarlama miktarını belirtmek için kullanılır.


### Crunch - Belirtilen Tekrarlama
```shell-session
$ crunch 12 12 -t 10031998@@@@ -d 1 -o wordlist
```


## CUPP
CUPP, Common User Password Profiler anlamına gelir ve sosyal mühendislik ve OSINT'ten elde edilen bilgilere dayalı olarak son derece hedefli ve özelleştirilmiş kelime listeleri oluşturmak için kullanılır. İnsanlar parola oluştururken telefon numaraları, evcil hayvan isimleri, doğum tarihleri gibi kişisel bilgileri kullanma eğilimindedir. CUPP bu bilgileri alır ve bunlardan parolalar oluşturur. Bu kelime listeleri çoğunlukla sosyal medya hesaplarına erişim sağlamak için kullanılır. CUPP Parrot OS'de varsayılan olarak yüklüdür ve repo burada bulunabilir. “-i” seçeneği, etkileşimli modda çalıştırmak için kullanılır ve CUPP'nin bizden hedef hakkında bilgi istemesini sağlar.


#### CUPP - Usage Example
```shell-session
M1R4CKCK@htb[/htb]$ python3 cupp.py -i

[+] Insert the information about the victim to make a dictionary
[+] If you don't know all the info, just hit enter when asked! ;)

> First Name: roger
> Surname: penrose
> Nickname:      
> Birthdate (DDMMYYYY): 11051972

> Partners) name: beth
> Partners) nickname:
> Partners) birthdate (DDMMYYYY):

> Child's name: john
> Child's nickname: johnny
> Child's birthdate (DDMMYYYY):

> Pet's name: tommy
> Company name: INLANE FREIGHT

> Do you want to add some key words about the victim? Y/[N]: Y
> Please enter the words, separated by comma. [i.e. hacker,juice,black], spaces will be removed: sysadmin,linux,86391512
> Do you want to add special chars at the end of words? Y/[N]:
> Do you want to add some random numbers at the end of words? Y/[N]:
> Leet mode? (i.e. leet = 1337) Y/[N]:

[+] Now making a dictionary...
[+] Sorting list and removing duplicates...
[+] Saving dictionary to roger.txt, counting 2419 words.
[+] Now load your pistolero with roger.txt and shoot! Good luck!
```

Yukarıdaki komut Roger Penrose kullanıcısına ait verilerin CUPP'ye nasıl sağlandığını göstermektedir. Bilinmeyen alanlar boş bırakılabilir. Tüm verileri aldıktan sonra, CUPP buna dayalı bir kelime listesi oluşturur. Ayrıca rastgele karakterlerin eklenmesini ve ortak kelimelerdeki harf ve sayı kombinasyonlarını kullanan bir “Leet” modunu da destekler. CUPP ayrıca “-l” seçeneğini kullanarak çeşitli çevrimiçi veritabanlarından ortak isimleri getirebilir.


### KWPROCESSOR
Kwprocessor, klavye yürüyüşleri ile kelime listeleri oluşturan bir araçtır. Bir başka yaygın parola oluşturma tekniği de klavyedeki kalıpları takip etmektir. Bu parolalar, tuşlar boyunca bir yürüyüş gibi göründükleri için klavye yürüyüşleri olarak adlandırılır. Örneğin, “qwertyasdfg” dizesi klavyenin ilk iki satırındaki ilk beş karakter kullanılarak oluşturulur. Bu normal bir göz için karmaşık görünebilir ancak kolayca tahmin edilebilir. Kwprocessor bu gibi kalıpları tahmin etmek için çeşitli algoritmalar kullanır.

Araç burada bulunabilir ve manuel olarak yüklenmesi gerekir.

#### Kwprocessor - Installation
```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/hashcat/kwprocessor
M1R4CKCK@htb[/htb]$ cd kwprocessor
M1R4CKCK@htb[/htb]$ make
```
Yardım menüsü kwp tarafından desteklenen çeşitli seçenekleri gösterir. Kalıp, bir kullanıcının klavyede seçebileceği coğrafi yönlere dayanmaktadır. Örneğin, “--keywalk-west” seçeneği temel karakterden batıya doğru hareketi belirtmek için kullanılır. Program parametre olarak temel karakterleri alır, bu da kalıbın başlayacağı karakter kümesidir. Daha sonra, dile özgü klavye düzenlerinde tuşların konumlarını eşleyen bir tuş haritasına ihtiyaç duyar. Son seçenek kullanılacak rotayı belirtmek için kullanılır. Rota, parolalar tarafından takip edilecek bir kalıptır. Temel karakterlerden başlayarak parolaların nasıl oluşturulacağını tanımlar. Örneğin, 222 rotası temel karakterden itibaren 2 * DOĞU + 2 * GÜNEY + 2 * BATI yolunu gösterebilir. Temel karakter “T” olarak kabul edilirse, rota tarafından oluşturulan parola bir ABD anahtar haritasında “TYUJNBV” olacaktır. Daha fazla bilgi için kwprocessor için [README](https://github.com/hashcat/kwprocessor#routes)'ye bakın.


#### Kwprocessor - Example

```shell-session
$ kwp -s 1 basechars/full.base keymaps/en-us.keymap  routes/2-to-10-max-3-direction-changes.route
```
Yukarıdaki komut, tam tabanı, standart en-us tuş haritasını ve 3 yön değiştirme rotasını kullanarak, shift (-s) tuşunu basılı tutarken erişilebilen karakterleri içeren kelimeler üretir.


## Princeprocessor

PRINCE veya PRobability INfinite Chained Elements, şifre kırma oranlarını iyileştirmek için etkili bir şifre tahmin algoritmasıdır. Princeprocessor, PRINCE algoritmasını kullanarak parola üreten bir araçtır. Program bir kelime listesi alır ve bu kelime listesinden alınan kelime zincirleri oluşturur. Örneğin, bir kelime listesi şu kelimeleri içeriyorsa:

#### Wordlist
```shell-session
dog
cat
ball
```

Oluşturulan kelime listesi şu şekilde olacaktır:


### Princeprocessor - Oluşturulan Kelime Listesi
```shell-session
dog
cat
ball
dogdog
catdog
dogcat
catcat
dogball
catball
balldog
ballcat
ballball
dogdogdog
catdogdog
dogcatdog
catcatdog
dogdogcat
<SNIP>
```
PRINCE algoritması her kelimeyi oluştururken çeşitli permütasyon ve kombinasyonları dikkate alır. Binary [sürümler](https://github.com/hashcat/princeprocessor/releases) sayfasından indirilebilir.

#### Princeprocessor - Installation
```shell-session
M1R4CKCK@htb[/htb]$ wget https://github.com/hashcat/princeprocessor/releases/download/v0.22/princeprocessor-0.22.7z
M1R4CKCK@htb[/htb]$ 7z x princeprocessor-0.22.7z
M1R4CKCK@htb[/htb]$ cd princeprocessor-0.22
M1R4CKCK@htb[/htb]$ ./pp64.bin -h
```
“--keyspace” seçeneği, girdi kelime listesinden üretilen kombinasyonların sayısını bulmak için kullanılabilir.


### Princeprocessor - Kombinasyon Sayısını Bulma
```shell-session
M1R4CKCK@htb[/htb]$ ./pp64.bin --keyspace < words

232
```
princeprocessor verilerine göre yukarıdaki kelime listemizden 232 benzersiz kelime oluşturulabilir.


### Princeprocessor - Kelime Listesi Oluşturma
```shell-session
M1R4CKCK@htb[/htb]$ ./pp64.bin -o wordlist.txt < words
```
Yukarıdaki komut, çıktı sözcüklerini wordlist.txt adlı bir dosyaya yazar. Varsayılan olarak, princeprocessor yalnızca 16 uzunluğa kadar olan sözcüklerin çıktısını verir. Bu, “--pw-min” ve “--pw-max” argümanları kullanılarak kontrol edilebilir.


### Princeprocessor - Parola Uzunluk Sınırları
```shell-session
M1R4CKCK@htb[/htb]$ ./pp64.bin --pw-min=10 --pw-max=25 -o wordlist.txt < words
```
Yukarıdaki komut, uzunluğu 10 ile 25 arasında olan sözcüklerin çıktısını verecektir. Kelime başına eleman sayısı “--elem-cnt-min” ve “--elem-cnt-max” kullanılarak kontrol edilebilir. Bu değerler, bir çıktı sözcüğündeki öğe sayısının verilen değerin üstünde veya altında olmasını sağlar.


### Princeprocessor - Specifying Elements
```shell-session
M1R4CKCK@htb[/htb]$ ./pp64.bin --elem-cnt-min=3 -o wordlist.txt < words
```
Yukarıdaki komut üç veya daha fazla elemanı olan kelimelerin çıktısını verecektir, yani,” dogdogdog`.”


## CeWL
CeWL, özel kelime listeleri oluşturmak için kullanılabilecek başka bir araçtır. Bir web sitesini örümceklerle tarar ve mevcut kelimelerin bir listesini oluşturur. İnsanlar yazdıkları veya üzerinde çalıştıkları içerikle ilişkili şifreler kullanma eğiliminde olduklarından bu tür bir kelime listesi etkilidir. Örneğin, doğa, vahşi yaşam vb. hakkında blog yazan bir blog yazarının bu konularla ilişkili bir şifresi olabilir. Bu, insan doğasından kaynaklanır çünkü bu tür şifrelerin hatırlanması da kolaydır. Kuruluşların genellikle markalarıyla ve sektöre özgü kelimelerle ilişkili şifreleri vardır. Örneğin, bir ağ şirketinin kullanıcılarının Router, Switch, Server gibi kelimelerden oluşan şifreleri olabilir. Bu tür kelimeler web sitelerinde bloglar, referanslar ve ürün açıklamaları altında bulunabilir.

CeWL'nin genel sözdizimi aşağıdaki gibidir:

#### CeWL - Syntax
```shell-session
M1R4CKCK@htb[/htb]$ cewl -d <depth to spider> -m <minimum word length> -w <output wordlist> <url of website>
```
CeWL belirli bir web sitesinde bulunan birden fazla sayfayı tarayabilir. Çıktısı alınan kelimelerin uzunluğu, şifre gereksinimlerine bağlı olarak “-m” parametresi kullanılarak değiştirilebilir (örneğin, bazı web sitelerinin minimum şifre uzunluğu vardır).

CeWL ayrıca “-e” seçeneği ile web sitelerinden e-postaların çıkarılmasını da destekler. Daha sonra phishing, password spraying ya da brute-forcing yaparken bu bilgileri almak faydalı olacaktır.

#### CeWL - Example
```shell-session
M1R4CKCK@htb[/htb]$ cewl -d 5 -m 8 -e http://inlanefreight.com/blog -w wordlist.txt
```
Yukarıdaki komut, “http://inlanefreight.com/blog” adresinden beş sayfalık bir derinliğe kadar kazıma yapar ve yalnızca 8'den uzun kelimeleri içerir.


### Önceden Kırılmış Şifreler
Varsayılan olarak, hashcat kırılan tüm şifreleri hashcat.potfile dosyasında saklar; format hash:password şeklindedir. Bu dosyanın temel amacı daha önce kırılan hashleri iş günlüğünden kaldırmak ve --show komutuyla kırılan parolaları görüntülemektir. Bununla birlikte, daha önce kırılmış şifrelerin yeni kelime listelerini oluşturmak için kullanılabilir ve kural dosyalarıyla birleştirildiğinde, temalı şifreleri kırmada oldukça etkili olabilir.
```shell-session
M1R4CKCK@htb[/htb]$ cut -d: -f 2- ~/hashcat.potfile
```


## Hashcat-utils
Hashcat-utils deposu, daha gelişmiş parola kırma için yararlı olabilecek birçok yardımcı program içerir. Örneğin maskprocessor aracı, belirli bir maskeyi kullanarak kelime listeleri oluşturmak için kullanılabilir. Bu aracın detaylı kullanımını [burada](https://hashcat.net/wiki/doku.php?id=maskprocessor) bulabilirsiniz.

Örneğin, bir sözcüğün sonuna tüm özel karakterleri eklemek için maskprocessor kullanılabilir:
```shell-session
M1R4CKCK@htb[/htb]$ /mp64.bin Welcome?s
Welcome 
Welcome!
Welcome"
Welcome#
Welcome$
Welcome%
Welcome&
Welcome'
Welcome(
Welcome)
Welcome*
Welcome+

<SNIP>
```


### Kurallarla Çalışma
Kural tabanlı saldırı en gelişmiş ve karmaşık password kırma modudur. Kurallar, giriş kelime listesi üzerinde ön ekleme, son ekleme, büyük/küçük harf değiştirme, kesme, ters çevirme ve çok daha fazlası gibi çeşitli işlemlerin gerçekleştirilmesine yardımcı olur. Kurallar maske tabanlı saldırıları başka bir seviyeye taşır ve daha yüksek kırma oranları sağlar. Ayrıca, kuralların kullanımı disk alanından ve daha büyük kelime listelerinin bir sonucu olarak ortaya çıkan işlem süresinden tasarruf sağlar.

Bir kelimeyi girdi olarak alan ve değiştirilmiş versiyonunu çıktı olarak veren fonksiyonlar kullanılarak bir kural oluşturulabilir. Aşağıdaki tabloda JtR ve Hashcat ile uyumlu bazı fonksiyonlar açıklanmaktadır.
![Pasted image 20241017095929.png](/img/user/resimler/Pasted%20image%2020241017095929.png)
Fonksiyonların tam listesini [burada](https://hashcat.net/wiki/doku.php?id=rule_based_attack#implemented_compatible_functions) bulabilirsiniz. Bazen, girdi kelime listeleri hedef spesifikasyonlarımızla eşleşmeyen kelimeler içerir. Örneğin, bir şirketin parola politikası, kullanıcıların 7 karakterden daha kısa parolalar belirlemesine izin vermeyebilir. Bu gibi durumlarda, bu tür kelimelerin işlenmesini önlemek için reddetme kuralları kullanılabilir.

Uzunluğu N'den az olan kelimeler >N ile reddedilirken, N'den büyük olan kelimeler <N ile reddedilebilir. Reddetme kurallarının bir listesini [burada](https://hashcat.net/wiki/doku.php?id=rule_based_attack#rules_used_to_reject_plains) bulabilirsiniz.

Not: Reddetme kuralları yalnızca hashcat-legacy ile ya da Hashcat ile -j veya -k kullanıldığında çalışır. Hashcat ile normal kurallar olarak (bir kural dosyasında) çalışmazlar.


## Example Rule Creation
Yaygın şifrelere dayalı bir kuralı nasıl oluşturabileceğimize bakalım. Genel kullanıcı davranışı, harfleri benzer sayılarla değiştirme eğiliminde olduklarını göstermektedir, örneğin “o” “0” ile veya “i” “1” ile değiştirilebilir. Bu genellikle L33tspeak olarak bilinir ve çok etkilidir. Kurumsal şifreler genellikle bir yıl ile öncelenir veya eklenir. Bu tür kelimeleri oluşturmak için bir kural oluşturalım.

*Not: Reddetme kuralları yalnızca hashcat-legacy ile ya da Hashcat ile -j veya -k kullanıldığında çalışır. Hashcat ile normal kurallar olarak (bir kural dosyasında) çalışmazlar. *


#### Rules
```shell-session
c so0 si1 se3 ss5 sa@ $2 $0 $1 $9
```


....


# Ortak Hash'leri Kırma
Sızma testi çalışmaları sırasında çok çeşitli hash türleriyle karşılaşırız; bazıları son derece yaygındır ve çoğu çalışmada görülür, diğerleri ise çok nadir görülür veya hiç görülmez. Daha önce de [belirtildiği](https://hashcat.net/wiki/doku.php?id=example_hashes) gibi, Hashcat'in yaratıcıları Hashcat'in desteklediği çoğu hash modunun örnek hash'lerinin bir listesini tutmaktadır. Listede hash modu, hash adı ve belirtilen türde örnek bir hash yer almaktadır. En sık görülen karmalardan bazıları şunlardır:
![Pasted image 20241017100619.png](/img/user/resimler/Pasted%20image%2020241017100619.png)


## Example 1 - Database Dumps
MD5, SHA1 ve bcrypt hash'leri genellikle veritabanı dump'larında görülür. Bu hash'ler başarılı bir SQL enjeksiyon saldırısının ardından alınabilir veya kamuya açık parola veri ihlali veritabanı dökümlerinde bulunabilir. MD5 ve SHA1'in kırılması, Blowfish algoritmasının birçok turunun uygulanmış olabileceği bcrypt'e göre genellikle daha kolaydır.

Bazı SHA1 hash'lerini kıralım. Aşağıdaki listeyi ele alalım:

#### SHA1 Hashes List
```shell-session
winter!
baseball1
waterslide
summertime
baconandeggs
beach1234
sunshine1
welcome1
password123
```
Her kelime için hızlı bir şekilde SHA1 oluşturabiliriz:


### SHA1 Hashes Oluşturma
```shell-session
M1R4CKCK@htb[/htb]$ for i in $(cat words); do echo -n $i | sha1sum | tr -d ' -';done

fa3c9ecfc251824df74026b4f40e4b373fd4fc46
e6852777c0260493de41fb43918ab07bbb3a659c
0c3feaa16f73493f998970e22b2a02cb9b546768
b863c49eada14e3a8816220a7ab7054c28693664
b0feedd70a346f7f75086026169825996d7196f9
f47f832cba913ec305b07958b41babe2e0ad0437
08b314f0e1e2c41ec92c3735910658e5a82c6ba7
e35bece6c5e6e0e86ca51d0440e92282a9d6ac8a
cbfdac6008f9cab4083784cbd1874f76618d2a97
```
Daha sonra bunları rockyou.txt kelime listesini kullanarak bir Hashcat sözlük saldırısı ile çalıştırabiliriz.

```shell-session
$ hashcat -m 100 SHA1_hashes /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...
<SNIP>

08b314f0e1e2c41ec92c3735910658e5a82c6ba7:sunshine1
e35bece6c5e6e0e86ca51d0440e92282a9d6ac8a:welcome1
e6852777c0260493de41fb43918ab07bbb3a659c:baseball1
b863c49eada14e3a8816220a7ab7054c28693664:summertime
fa3c9ecfc251824df74026b4f40e4b373fd4fc46:winter! 
b0feedd70a346f7f75086026169825996d7196f9:baconandeggs
f47f832cba913ec305b07958b41babe2e0ad0437:beach1234
0c3feaa16f73493f998970e22b2a02cb9b546768:waterslide
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: SHA1
Hash.Target......: SHA1_hashes
Time.Started.....: Fri Aug 28 22:22:56 2020, (1 sec)
Time.Estimated...: Fri Aug 28 22:22:57 2020, (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2790.2 kH/s (0.33ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 9/9 (100.00%) Digests
Progress.........: 1173504/14344385 (8.18%)
Rejected.........: 0/1173504 (0.00%)
Restore.Point....: 1167360/14344385 (8.14%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: whitenerdy -> warut69
```
Yukarıdaki hash'ler çok az karmaşıklığa sahip ya da hiç karmaşık olmayan yaygın kelimeler/ifadeler olduğu için çok hızlı bir şekilde kırılmıştır. Yukarıdaki listedeki “Bas3b@ll1” veya “Wat3rSl1de” gibi varyasyonların kırılması muhtemelen daha uzun sürecektir ve maske ve hibrit saldırılar gibi ek teknikler gerektirebilir.

## Example 2 - Linux Shadow File
Sha512crypt hash'leri Linux sistemlerinde genellikle /etc/shadow dosyasında bulunur. Bu dosya, kendilerine atanmış bir oturum açma shell'i olan tüm hesapların parola hash'lerini içerir. Sızma testi sırasında bir Linux sistemine bir web uygulaması saldırısı veya savunmasız bir hizmetin başarılı bir şekilde kullanılması yoluyla erişim sağlayabiliriz. Halihazırda en yüksek ayrıcalıklı root hesabı bağlamında çalışan bir servisi istismar edebilir ve başarılı bir ayrıcalık yükseltme saldırısı gerçekleştirerek /etc/shadow dosyasına erişebiliriz. Parolanın yeniden kullanımı yaygındır. Kırılan bir parola bize diğer sunuculara, ağ cihazlarına erişim sağlayabilir ve hatta hedefin Active Directory ortamında bir dayanak noktası olarak kullanılabilir.

Standart bir Ubuntu kurulumundan bir hash'e bakalım. Aşağıdaki hash için karşılık gelen düz metin “password123 ”tür.

#### Root Password in Ubuntu Linux
```shell-session
root:$6$tOA0cyybhb/Hr7DN$htr2vffCWiPGnyFOicJiXJVMbk1muPORR.eRGYfBYUnNPUjWABGPFiphjIjJC5xPfFUASIbVKDAHS3vTW1qU.1:18285:0:99999:7:::
```
Hash, iki nokta üst üste ile ayrılmış dokuz alan içerir. İlk iki alan kullanıcı adını ve şifrelenmiş hash'ini içerir. Geri kalan alanlar parolanın oluşturulma zamanı, son değiştirilme zamanı ve geçerlilik süresi gibi çeşitli öznitelikleri içerir.

Hash'e gelince, “$” ile sınırlandırılmış üç alan içerdiğini zaten biliyoruz. “6” değeri SHA-512 hash algoritmasını temsil eder; sonraki 16 karakter salt'ı temsil ederken, geri kalanı gerçek hash'tir.

Hashcat kullanarak bu hash'i kıralım.

```shell-session
$ hashcat -m 1800 nix_hash /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...
<SNIP>

$6$tOA0cyybhb/Hr7DN$htr2vffCWiPGnyFOicJiXJVMbk1muPORR.eRGYfBYUnNPUjWABGPFiphjIjJC5xPfFUASIbVKDAHS3vTW1qU.1:password123
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: sha512crypt $6$, SHA512 (Unix)
Hash.Target......: $6$tOA0cyybhb/Hr7DN$htr2vffCWiPGnyFOicJiXJVMbk1muPO...W1qU.1
Time.Started.....: Fri Aug 28 22:25:26 2020, (1 sec)
Time.Estimated...: Fri Aug 28 22:25:27 2020, (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:      955 H/s (4.62ms) @ Accel:32 Loops:256 Thr:1 Vec:4
Recovered........: 1/1 (100.00%) Digests
Progress.........: 1536/14344385 (0.01%)
Rejected.........: 0/1536 (0.00%)
Restore.Point....: 1344/14344385 (0.01%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:4864-5000
Candidates.#1....: teacher -> mexico1
```

### Örnek 3 - Yaygın Active Directory Parola Hash Türleri
Kimlik bilgisi hırsızlığı ve parolanın yeniden kullanımı, ortamlarını yönetmek için Active Directory kullanan kuruluşlara karşı yapılan değerlendirmeler sırasında yaygın olarak kullanılan taktiklerdir. Kimlik bilgilerini açık metin olarak elde etmek veya Pass-the-Hash veya SMB Relay saldırıları yoluyla daha fazla erişim sağlamak için parola hash'lerini yeniden kullanmak genellikle mümkündür. Yine de bazı teknikler, erişimimizi ilerletmek için çevrimdışı olarak kırılması gereken bir parola karmasıyla sonuçlanacaktır. Bazı örnekler arasında MITM (Man-in-the-middle) saldırısı yoluyla elde edilen NetNTLMv1 veya NetNTLMv2, Kerberoasting saldırısı yoluyla elde edilen Kerberos 5 TGS-REP hash'i veya ==Mimikatz== aracı kullanılarak bellekten kimlik bilgilerinin dökümü yoluyla elde edilen veya bir Windows makinesinin yerel SAM veritabanından elde edilen bir NTLM hash'i sayılabilir.


#### NTLM
Örneklerden biri, bir sunucuya Remote Desktop (RDP) erişimi olan ancak lokal yönetici olmayan bir kullanıcı için NTLM parola karmasını almaktır, bu nedenle NTLM karması erişim elde etmek için bir pass-the-hash saldırısı için kullanılamaz. Bu durumda, açık metin parolası, sunucuya RDP aracılığıyla bağlanarak ve ağ içinde daha fazla numaralandırma gerçekleştirerek veya yerel ayrıcalık yükseltme vektörleri arayarak erişimimizi ilerletmek için gereklidir.

Bir örnek üzerinden yürüyelim. Python'un 3 satırını kullanarak amaçlarımız için “Password01” parolasının NTLM hash'ini hızlı bir şekilde oluşturabiliriz:


#### Python3 - Hashlib
```shell-session
M1R4CKCK@htb[/htb]$ python3

Python 3.8.3 (default, May 14 2020, 11:03:12) 
[GCC 9.3.0] on linux
Type "help", "copyright", "credits" or "license" for more information.

>>> import hashlib,binascii
>>> hash = hashlib.new('md4', "Password01".encode('utf-16le')).digest()
>>> print (binascii.hexlify(hash))

b'7100a909c7ff05b266af3c42ec058c33'
```

Daha sonra ortaya çıkan NTLM parola hash değerini “7100a909c7ff05b266af3c42ec058c33” standart rockyou.txt kelime listesini kullanarak Hashcat üzerinden çalıştırıp açık metni alabiliriz.

#### Hashcat - Cracking NTLM Hashes
```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 0 -m 1000 ntlm_example /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...
<SNIP>

7100a909c7ff05b266af3c42ec058c33:Password01      
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NTLM
Hash.Target......: 7100a909c7ff05b266af3c42ec058c33
Time.Started.....: Fri Aug 28 22:27:40 2020, (0 secs)
Time.Estimated...: Fri Aug 28 22:27:40 2020, (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  2110.5 kH/s (0.62ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 61440/14344385 (0.43%)
Rejected.........: 0/61440 (0.00%)
Restore.Point....: 55296/14344385 (0.39%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: gonoles -> sinead1
```
Şimdi, açık metin şifresiyle donanmış olarak, ağ içindeki erişimimizi daha da ileri götürebiliriz.



### NetNTLMv2
Bir sızma testi sırasında, kimlik bilgilerini “çalmak” amacıyla MITM saldırıları gerçekleştirmek için [Responder](https://github.com/lgandx/Responder) gibi araçları çalıştırmak yaygındır. Bu tür saldırılar diğer modüllerde ayrıntılı olarak ele alınmaktadır. Yoğun kurumsal ağlarda bu yöntemi kullanarak çok sayıda NetNTLMv2 parola hash'i elde etmek yaygındır. Bunlar genellikle kırılabilir ve Active Directory ortamında bir yer edinmek için kullanılabilir, hatta bazen parola karmasıyla ilişkili kullanıcı hesabına verilen ayrıcalıklara bağlı olarak birçok veya tüm sistemlere tam yönetici erişimi elde edilebilir. Bir değerlendirmenin başlangıcında Responder kullanılarak alınan aşağıdaki parola karmasını düşünün:

#### Responder - NTLMv2
```shell-session
sqladmin::INLANEFREIGHT:f54d6f198a7a47d4:7FECABAE13101DAAA20F1B09F7F7A4EA:0101000000000000C0653150DE09D20126F3F71DF13C1FD8000000000200080053004D004200330001001E00570049004E002D00500052004800340039003200520051004100460056000400140053004D00420033002E006C006F00630061006C0003003400570049004E002D00500052004800340039003200520051004100460056002E0053004D00420033002E006C006F00630061006C000500140053004D00420033002E006C006F00630061006C0007000800C0653150DE09D201060004000200000008003000300000000000000000000000003000001A67637962F2B7BF297745E6074934196D5F4371B6BA3E796F2997306FD4C1C00A001000000000000000000000000000000000000900280063006900660073002F003100390032002E003100360038002E003100390035002E00310037003000000000000000000000000000
```
Responder gibi bazı araçlar, ne tür bir hash alındığını size bildirecektir. Şüpheye düşersek Örnek hash'ler sayfasını da kontrol edebilir ve bunun gerçekten bir NetNTLMv2 hash'i veya Hashcat'te mod 5600 olduğunu doğrulayabiliriz.

Önceki örneklerde olduğu gibi, çevrimdışı bir sözlük saldırısı gerçekleştirmek için rockyou.txt kelime listesini kullanarak bu hash'i Hashcat ile çalıştırabiliriz.

--> https://hashcat.net/wiki/doku.php?id=example_hashes


#### Hashcat - Cracking NTLMv2 Hashes
```shell-session
M1R4CKCK@htb[/htb]$ hashcat -a 0 -m 5600 inlanefreight_ntlmv2 /opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt

hashcat (v6.1.1) starting...
<SNIP>

SQLADMIN::INLANEFREIGHT:f54d6f198a7a47d4:7fecabae13101daaa20f1b09f7f7a4ea:0101000000000000c0653150de09d20126f3f71df13c1fd8000000000200080053004d004200330001001e00570049004e002d00500052004800340039003200520051004100460056000400140053004d00420033002e006c006f00630061006c0003003400570049004e002d00500052004800340039003200520051004100460056002e0053004d00420033002e006c006f00630061006c000500140053004d00420033002e006c006f00630061006c0007000800c0653150de09d201060004000200000008003000300000000000000000000000003000001a67637962f2b7bf297745e6074934196d5f4371b6ba3e796f2997306fd4c1c00a001000000000000000000000000000000000000900280063006900660073002f003100390032002e003100360038002e003100390035002e00310037003000000000000000000000000000:Database99
                                                 
Session..........: hashcat
Status...........: Cracked
Hash.Name........: NetNTLMv2
Hash.Target......: SQLADMIN::INLANEFREIGHT:f54d6f198a7a47d4:7fecabae13...000000
Time.Started.....: Fri Aug 28 22:29:26 2020, (6 secs)
Time.Estimated...: Fri Aug 28 22:29:32 2020, (0 secs)
Guess.Base.......: File (/opt/useful/SecLists/Passwords/Leaked-Databases/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  1754.7 kH/s (2.32ms) @ Accel:1024 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests
Progress.........: 11237376/14344385 (78.34%)
Rejected.........: 0/11237376 (0.00%)
Restore.Point....: 11231232/14344385 (78.30%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: Devanique -> Darrylw
```
Bu kimlik bilgileriyle donanmış olarak, yarışa başlayabiliriz ve daha sonraki modüllerde derinlemesine ele alınan Active Directory'yi numaralandırmaya ve saldırmaya başlayabiliriz.


Soru :  Aşağıdaki hash'i kırın: 7106812752615cdfe427e01b98cd4083
Cevap :
![Pasted image 20241017103239.png](/img/user/resimler/Pasted%20image%2020241017103239.png)
 | md5sum | tr -d " -" > hybrid_hash
```

Hashcat sözcük listesindeki sözcükleri okur ve verilen maskeye göre benzersiz bir dize ekler. Bu durumda, “?d?s” maskesi hashcat'e rockyou.txt kelime listesindeki her kelimenin sonuna bir rakam ve özel bir karakter eklemesini söyler.


#### Hashcat - Hybrid Attack using Wordlists

{{CODE_BLOCK_32}}
Saldırı modu “7”, belirli bir maske kullanarak kelimelere karakter eklemek için kullanılabilir. Aşağıdaki örnekte rockyou.txt kelime listesindeki her kelimeye bir önek eklemek için özel bir karakter kümesi kullanan bir maske gösterilmektedir. “-1 01” özel karakter kümesiyle ‘20?1?d’ özel karakter maskesi, kelime listesindeki her kelimeye çeşitli yıllar ekleyecektir (örneğin, 2010, 2011, 2012...).


### Başka bir Hibrit Hash Oluşturma
{{CODE_BLOCK_33}}


#### Hashcat - Hybrid Attack using Wordlists with Masks
{{CODE_BLOCK_34}}


Soru :  Aşağıdaki hash'i kırın: 978078e7845f2fb2e20399d9e80475bc1c275e06 “?d?s” maskesini kullanarak.
Cevap : 

==hashcat -a 6 -m 100 "978078e7845f2fb2e20399d9e80475bc1c275e06" rockyou.txt '?d?s'==

![Pasted image 20241016013605.png](/img/user/resimler/Pasted%20image%2020241016013605.png)


### Kişisel Sözcük Listeleri Oluşturma


### Wordlists
Bir değerlendirme sırasında, görevin başarısı için çok önemli olan bir veya daha fazla parola hash'i alabiliriz. En iyi çabalarımıza rağmen, bu hash'ler önceki bölümlerde ele alınan sözlük, kombinasyon, maske veya hibrit saldırılar kullanılarak yaygın kelime listeleri ile kırılamaz. Bu durumlarda hedefimize ulaşmak için özel, hedeflenmiş bir kelime listesi oluşturmak gerekebilir.

Bir wordlist'i iyileştirmek için zaman harcamak gerekir çünkü başarı oranı büyük ölçüde buna bağlıdır. Kelime listeleri çeşitli kaynaklardan elde edilebilir ve hedefe göre özelleştirilebilir ve kurallar kullanılarak daha da ince ayar yapılabilir. Parolalar, kullanıcı adları, dosya adları, yükler ve diğer birçok veri türü için kelime listeleri bulunabilir. SecLists deposu ayrıca kullanıcı adı numaralandırma parola tanımlama için yararlı birçok kelime listesi içerir.


### Creating Wordlists
Birçok açık kaynak aracı, gereksinimlerimize göre özelleştirilmiş parola kelime listeleri oluşturmaya yardımcı olur.


### Crunch
Crunch, belirli bir uzunluktaki kelimeler, sınırlı bir karakter seti veya belirli bir desen gibi parametrelere dayalı kelime listeleri oluşturabilir. Hem permütasyonlar hem de kombinasyonlar oluşturabilir.

Parrot OS'de varsayılan olarak yüklüdür ve burada bulunabilir. Crunch'ın genel sözdizimi aşağıdaki gibidir:

#### Crunch - Syntax
{{CODE_BLOCK_35}}
“-t” seçeneği, oluşturulan parolalar için desen belirtmek için kullanılır. Kalıp, küçük harf karakterlerini temsil eden “@,” içerebilir, “,” (virgül) büyük harf karakterlerini ekler, “%” sayıları ekler ve “^” sembolleri ekler.


#### Crunch - Generate Word List
{{CODE_BLOCK_36}}

Yukarıdaki komut, varsayılan karakter kümesini kullanarak 4 ila 8 karakter uzunluğunda sözcüklerden oluşan bir sözcük listesi oluşturur.

Inlane Freight kullanıcı şifrelerinin “ILFREIGHTYYYXXXX” şeklinde olduğunu varsayalım; burada “XXXX” harf içeren çalışan kimliği ve “YYYY” yıl olsun. Bu tür parolaların bir listesini oluşturmak için crunch'ı kullanabiliriz.


### Crunch - Kalıp Kullanarak Kelime Listesi Oluşturma
{{CODE_BLOCK_37}}

“ILFREIGHT201%@@@@” kalıbı, 2010-2019 yıllarını ve ardından dört harfi içeren sözcükler oluşturacaktır. Buradaki uzunluk 17'dir ve tüm kelimeler için sabittir.

Bir kullanıcının doğum tarihinin 10/03/1998 olduğunu biliyorsak (sosyal medya vb. aracılığıyla), bunu şifresine ekleyebilir ve ardından bir harf dizisi ekleyebiliriz. Crunch bu tür kelimelerden oluşan bir kelime listesi oluşturmak için kullanılabilir. “-d” seçeneği tekrarlama miktarını belirtmek için kullanılır.


### Crunch - Belirtilen Tekrarlama
{{CODE_BLOCK_38}}


## CUPP
CUPP, Common User Password Profiler anlamına gelir ve sosyal mühendislik ve OSINT'ten elde edilen bilgilere dayalı olarak son derece hedefli ve özelleştirilmiş kelime listeleri oluşturmak için kullanılır. İnsanlar parola oluştururken telefon numaraları, evcil hayvan isimleri, doğum tarihleri gibi kişisel bilgileri kullanma eğilimindedir. CUPP bu bilgileri alır ve bunlardan parolalar oluşturur. Bu kelime listeleri çoğunlukla sosyal medya hesaplarına erişim sağlamak için kullanılır. CUPP Parrot OS'de varsayılan olarak yüklüdür ve repo burada bulunabilir. “-i” seçeneği, etkileşimli modda çalıştırmak için kullanılır ve CUPP'nin bizden hedef hakkında bilgi istemesini sağlar.


#### CUPP - Usage Example
{{CODE_BLOCK_39}}

Yukarıdaki komut Roger Penrose kullanıcısına ait verilerin CUPP'ye nasıl sağlandığını göstermektedir. Bilinmeyen alanlar boş bırakılabilir. Tüm verileri aldıktan sonra, CUPP buna dayalı bir kelime listesi oluşturur. Ayrıca rastgele karakterlerin eklenmesini ve ortak kelimelerdeki harf ve sayı kombinasyonlarını kullanan bir “Leet” modunu da destekler. CUPP ayrıca “-l” seçeneğini kullanarak çeşitli çevrimiçi veritabanlarından ortak isimleri getirebilir.


### KWPROCESSOR
Kwprocessor, klavye yürüyüşleri ile kelime listeleri oluşturan bir araçtır. Bir başka yaygın parola oluşturma tekniği de klavyedeki kalıpları takip etmektir. Bu parolalar, tuşlar boyunca bir yürüyüş gibi göründükleri için klavye yürüyüşleri olarak adlandırılır. Örneğin, “qwertyasdfg” dizesi klavyenin ilk iki satırındaki ilk beş karakter kullanılarak oluşturulur. Bu normal bir göz için karmaşık görünebilir ancak kolayca tahmin edilebilir. Kwprocessor bu gibi kalıpları tahmin etmek için çeşitli algoritmalar kullanır.

Araç burada bulunabilir ve manuel olarak yüklenmesi gerekir.

#### Kwprocessor - Installation
{{CODE_BLOCK_40}}
Yardım menüsü kwp tarafından desteklenen çeşitli seçenekleri gösterir. Kalıp, bir kullanıcının klavyede seçebileceği coğrafi yönlere dayanmaktadır. Örneğin, “--keywalk-west” seçeneği temel karakterden batıya doğru hareketi belirtmek için kullanılır. Program parametre olarak temel karakterleri alır, bu da kalıbın başlayacağı karakter kümesidir. Daha sonra, dile özgü klavye düzenlerinde tuşların konumlarını eşleyen bir tuş haritasına ihtiyaç duyar. Son seçenek kullanılacak rotayı belirtmek için kullanılır. Rota, parolalar tarafından takip edilecek bir kalıptır. Temel karakterlerden başlayarak parolaların nasıl oluşturulacağını tanımlar. Örneğin, 222 rotası temel karakterden itibaren 2 * DOĞU + 2 * GÜNEY + 2 * BATI yolunu gösterebilir. Temel karakter “T” olarak kabul edilirse, rota tarafından oluşturulan parola bir ABD anahtar haritasında “TYUJNBV” olacaktır. Daha fazla bilgi için kwprocessor için [README](https://github.com/hashcat/kwprocessor#routes)'ye bakın.


#### Kwprocessor - Example

{{CODE_BLOCK_41}}
Yukarıdaki komut, tam tabanı, standart en-us tuş haritasını ve 3 yön değiştirme rotasını kullanarak, shift (-s) tuşunu basılı tutarken erişilebilen karakterleri içeren kelimeler üretir.


## Princeprocessor

PRINCE veya PRobability INfinite Chained Elements, şifre kırma oranlarını iyileştirmek için etkili bir şifre tahmin algoritmasıdır. Princeprocessor, PRINCE algoritmasını kullanarak parola üreten bir araçtır. Program bir kelime listesi alır ve bu kelime listesinden alınan kelime zincirleri oluşturur. Örneğin, bir kelime listesi şu kelimeleri içeriyorsa:

#### Wordlist
{{CODE_BLOCK_42}}

Oluşturulan kelime listesi şu şekilde olacaktır:


### Princeprocessor - Oluşturulan Kelime Listesi
{{CODE_BLOCK_43}}
PRINCE algoritması her kelimeyi oluştururken çeşitli permütasyon ve kombinasyonları dikkate alır. Binary [sürümler](https://github.com/hashcat/princeprocessor/releases) sayfasından indirilebilir.

#### Princeprocessor - Installation
{{CODE_BLOCK_44}}
“--keyspace” seçeneği, girdi kelime listesinden üretilen kombinasyonların sayısını bulmak için kullanılabilir.


### Princeprocessor - Kombinasyon Sayısını Bulma
{{CODE_BLOCK_45}}
princeprocessor verilerine göre yukarıdaki kelime listemizden 232 benzersiz kelime oluşturulabilir.


### Princeprocessor - Kelime Listesi Oluşturma
{{CODE_BLOCK_46}}
Yukarıdaki komut, çıktı sözcüklerini wordlist.txt adlı bir dosyaya yazar. Varsayılan olarak, princeprocessor yalnızca 16 uzunluğa kadar olan sözcüklerin çıktısını verir. Bu, “--pw-min” ve “--pw-max” argümanları kullanılarak kontrol edilebilir.


### Princeprocessor - Parola Uzunluk Sınırları
{{CODE_BLOCK_47}}
Yukarıdaki komut, uzunluğu 10 ile 25 arasında olan sözcüklerin çıktısını verecektir. Kelime başına eleman sayısı “--elem-cnt-min” ve “--elem-cnt-max” kullanılarak kontrol edilebilir. Bu değerler, bir çıktı sözcüğündeki öğe sayısının verilen değerin üstünde veya altında olmasını sağlar.


### Princeprocessor - Specifying Elements
{{CODE_BLOCK_48}}
Yukarıdaki komut üç veya daha fazla elemanı olan kelimelerin çıktısını verecektir, yani,” dogdogdog`.”


## CeWL
CeWL, özel kelime listeleri oluşturmak için kullanılabilecek başka bir araçtır. Bir web sitesini örümceklerle tarar ve mevcut kelimelerin bir listesini oluşturur. İnsanlar yazdıkları veya üzerinde çalıştıkları içerikle ilişkili şifreler kullanma eğiliminde olduklarından bu tür bir kelime listesi etkilidir. Örneğin, doğa, vahşi yaşam vb. hakkında blog yazan bir blog yazarının bu konularla ilişkili bir şifresi olabilir. Bu, insan doğasından kaynaklanır çünkü bu tür şifrelerin hatırlanması da kolaydır. Kuruluşların genellikle markalarıyla ve sektöre özgü kelimelerle ilişkili şifreleri vardır. Örneğin, bir ağ şirketinin kullanıcılarının Router, Switch, Server gibi kelimelerden oluşan şifreleri olabilir. Bu tür kelimeler web sitelerinde bloglar, referanslar ve ürün açıklamaları altında bulunabilir.

CeWL'nin genel sözdizimi aşağıdaki gibidir:

#### CeWL - Syntax
{{CODE_BLOCK_49}}
CeWL belirli bir web sitesinde bulunan birden fazla sayfayı tarayabilir. Çıktısı alınan kelimelerin uzunluğu, şifre gereksinimlerine bağlı olarak “-m” parametresi kullanılarak değiştirilebilir (örneğin, bazı web sitelerinin minimum şifre uzunluğu vardır).

CeWL ayrıca “-e” seçeneği ile web sitelerinden e-postaların çıkarılmasını da destekler. Daha sonra phishing, password spraying ya da brute-forcing yaparken bu bilgileri almak faydalı olacaktır.

#### CeWL - Example
{{CODE_BLOCK_50}}
Yukarıdaki komut, “http://inlanefreight.com/blog” adresinden beş sayfalık bir derinliğe kadar kazıma yapar ve yalnızca 8'den uzun kelimeleri içerir.


### Önceden Kırılmış Şifreler
Varsayılan olarak, hashcat kırılan tüm şifreleri hashcat.potfile dosyasında saklar; format hash:password şeklindedir. Bu dosyanın temel amacı daha önce kırılan hashleri iş günlüğünden kaldırmak ve --show komutuyla kırılan parolaları görüntülemektir. Bununla birlikte, daha önce kırılmış şifrelerin yeni kelime listelerini oluşturmak için kullanılabilir ve kural dosyalarıyla birleştirildiğinde, temalı şifreleri kırmada oldukça etkili olabilir.
{{CODE_BLOCK_51}}


## Hashcat-utils
Hashcat-utils deposu, daha gelişmiş parola kırma için yararlı olabilecek birçok yardımcı program içerir. Örneğin maskprocessor aracı, belirli bir maskeyi kullanarak kelime listeleri oluşturmak için kullanılabilir. Bu aracın detaylı kullanımını [burada](https://hashcat.net/wiki/doku.php?id=maskprocessor) bulabilirsiniz.

Örneğin, bir sözcüğün sonuna tüm özel karakterleri eklemek için maskprocessor kullanılabilir:
{{CODE_BLOCK_52}}


### Kurallarla Çalışma
Kural tabanlı saldırı en gelişmiş ve karmaşık password kırma modudur. Kurallar, giriş kelime listesi üzerinde ön ekleme, son ekleme, büyük/küçük harf değiştirme, kesme, ters çevirme ve çok daha fazlası gibi çeşitli işlemlerin gerçekleştirilmesine yardımcı olur. Kurallar maske tabanlı saldırıları başka bir seviyeye taşır ve daha yüksek kırma oranları sağlar. Ayrıca, kuralların kullanımı disk alanından ve daha büyük kelime listelerinin bir sonucu olarak ortaya çıkan işlem süresinden tasarruf sağlar.

Bir kelimeyi girdi olarak alan ve değiştirilmiş versiyonunu çıktı olarak veren fonksiyonlar kullanılarak bir kural oluşturulabilir. Aşağıdaki tabloda JtR ve Hashcat ile uyumlu bazı fonksiyonlar açıklanmaktadır.
![Pasted image 20241017095929.png](/img/user/resimler/Pasted%20image%2020241017095929.png)
Fonksiyonların tam listesini [burada](https://hashcat.net/wiki/doku.php?id=rule_based_attack#implemented_compatible_functions) bulabilirsiniz. Bazen, girdi kelime listeleri hedef spesifikasyonlarımızla eşleşmeyen kelimeler içerir. Örneğin, bir şirketin parola politikası, kullanıcıların 7 karakterden daha kısa parolalar belirlemesine izin vermeyebilir. Bu gibi durumlarda, bu tür kelimelerin işlenmesini önlemek için reddetme kuralları kullanılabilir.

Uzunluğu N'den az olan kelimeler >N ile reddedilirken, N'den büyük olan kelimeler <N ile reddedilebilir. Reddetme kurallarının bir listesini [burada](https://hashcat.net/wiki/doku.php?id=rule_based_attack#rules_used_to_reject_plains) bulabilirsiniz.

Not: Reddetme kuralları yalnızca hashcat-legacy ile ya da Hashcat ile -j veya -k kullanıldığında çalışır. Hashcat ile normal kurallar olarak (bir kural dosyasında) çalışmazlar.


## Example Rule Creation
Yaygın şifrelere dayalı bir kuralı nasıl oluşturabileceğimize bakalım. Genel kullanıcı davranışı, harfleri benzer sayılarla değiştirme eğiliminde olduklarını göstermektedir, örneğin “o” “0” ile veya “i” “1” ile değiştirilebilir. Bu genellikle L33tspeak olarak bilinir ve çok etkilidir. Kurumsal şifreler genellikle bir yıl ile öncelenir veya eklenir. Bu tür kelimeleri oluşturmak için bir kural oluşturalım.

*Not: Reddetme kuralları yalnızca hashcat-legacy ile ya da Hashcat ile -j veya -k kullanıldığında çalışır. Hashcat ile normal kurallar olarak (bir kural dosyasında) çalışmazlar. *


#### Rules
{{CODE_BLOCK_53}}


....


# Ortak Hash'leri Kırma
Sızma testi çalışmaları sırasında çok çeşitli hash türleriyle karşılaşırız; bazıları son derece yaygındır ve çoğu çalışmada görülür, diğerleri ise çok nadir görülür veya hiç görülmez. Daha önce de [belirtildiği](https://hashcat.net/wiki/doku.php?id=example_hashes) gibi, Hashcat'in yaratıcıları Hashcat'in desteklediği çoğu hash modunun örnek hash'lerinin bir listesini tutmaktadır. Listede hash modu, hash adı ve belirtilen türde örnek bir hash yer almaktadır. En sık görülen karmalardan bazıları şunlardır:
![Pasted image 20241017100619.png](/img/user/resimler/Pasted%20image%2020241017100619.png)


## Example 1 - Database Dumps
MD5, SHA1 ve bcrypt hash'leri genellikle veritabanı dump'larında görülür. Bu hash'ler başarılı bir SQL enjeksiyon saldırısının ardından alınabilir veya kamuya açık parola veri ihlali veritabanı dökümlerinde bulunabilir. MD5 ve SHA1'in kırılması, Blowfish algoritmasının birçok turunun uygulanmış olabileceği bcrypt'e göre genellikle daha kolaydır.

Bazı SHA1 hash'lerini kıralım. Aşağıdaki listeyi ele alalım:

#### SHA1 Hashes List
{{CODE_BLOCK_54}}
Her kelime için hızlı bir şekilde SHA1 oluşturabiliriz:


### SHA1 Hashes Oluşturma
{{CODE_BLOCK_55}}
Daha sonra bunları rockyou.txt kelime listesini kullanarak bir Hashcat sözlük saldırısı ile çalıştırabiliriz.

{{CODE_BLOCK_56}}
Yukarıdaki hash'ler çok az karmaşıklığa sahip ya da hiç karmaşık olmayan yaygın kelimeler/ifadeler olduğu için çok hızlı bir şekilde kırılmıştır. Yukarıdaki listedeki “Bas3b@ll1” veya “Wat3rSl1de” gibi varyasyonların kırılması muhtemelen daha uzun sürecektir ve maske ve hibrit saldırılar gibi ek teknikler gerektirebilir.

## Example 2 - Linux Shadow File
Sha512crypt hash'leri Linux sistemlerinde genellikle /etc/shadow dosyasında bulunur. Bu dosya, kendilerine atanmış bir oturum açma shell'i olan tüm hesapların parola hash'lerini içerir. Sızma testi sırasında bir Linux sistemine bir web uygulaması saldırısı veya savunmasız bir hizmetin başarılı bir şekilde kullanılması yoluyla erişim sağlayabiliriz. Halihazırda en yüksek ayrıcalıklı root hesabı bağlamında çalışan bir servisi istismar edebilir ve başarılı bir ayrıcalık yükseltme saldırısı gerçekleştirerek /etc/shadow dosyasına erişebiliriz. Parolanın yeniden kullanımı yaygındır. Kırılan bir parola bize diğer sunuculara, ağ cihazlarına erişim sağlayabilir ve hatta hedefin Active Directory ortamında bir dayanak noktası olarak kullanılabilir.

Standart bir Ubuntu kurulumundan bir hash'e bakalım. Aşağıdaki hash için karşılık gelen düz metin “password123 ”tür.

#### Root Password in Ubuntu Linux
{{CODE_BLOCK_57}}
Hash, iki nokta üst üste ile ayrılmış dokuz alan içerir. İlk iki alan kullanıcı adını ve şifrelenmiş hash'ini içerir. Geri kalan alanlar parolanın oluşturulma zamanı, son değiştirilme zamanı ve geçerlilik süresi gibi çeşitli öznitelikleri içerir.

Hash'e gelince, “$” ile sınırlandırılmış üç alan içerdiğini zaten biliyoruz. “6” değeri SHA-512 hash algoritmasını temsil eder; sonraki 16 karakter salt'ı temsil ederken, geri kalanı gerçek hash'tir.

Hashcat kullanarak bu hash'i kıralım.

{{CODE_BLOCK_58}}

### Örnek 3 - Yaygın Active Directory Parola Hash Türleri
Kimlik bilgisi hırsızlığı ve parolanın yeniden kullanımı, ortamlarını yönetmek için Active Directory kullanan kuruluşlara karşı yapılan değerlendirmeler sırasında yaygın olarak kullanılan taktiklerdir. Kimlik bilgilerini açık metin olarak elde etmek veya Pass-the-Hash veya SMB Relay saldırıları yoluyla daha fazla erişim sağlamak için parola hash'lerini yeniden kullanmak genellikle mümkündür. Yine de bazı teknikler, erişimimizi ilerletmek için çevrimdışı olarak kırılması gereken bir parola karmasıyla sonuçlanacaktır. Bazı örnekler arasında MITM (Man-in-the-middle) saldırısı yoluyla elde edilen NetNTLMv1 veya NetNTLMv2, Kerberoasting saldırısı yoluyla elde edilen Kerberos 5 TGS-REP hash'i veya ==Mimikatz== aracı kullanılarak bellekten kimlik bilgilerinin dökümü yoluyla elde edilen veya bir Windows makinesinin yerel SAM veritabanından elde edilen bir NTLM hash'i sayılabilir.


#### NTLM
Örneklerden biri, bir sunucuya Remote Desktop (RDP) erişimi olan ancak lokal yönetici olmayan bir kullanıcı için NTLM parola karmasını almaktır, bu nedenle NTLM karması erişim elde etmek için bir pass-the-hash saldırısı için kullanılamaz. Bu durumda, açık metin parolası, sunucuya RDP aracılığıyla bağlanarak ve ağ içinde daha fazla numaralandırma gerçekleştirerek veya yerel ayrıcalık yükseltme vektörleri arayarak erişimimizi ilerletmek için gereklidir.

Bir örnek üzerinden yürüyelim. Python'un 3 satırını kullanarak amaçlarımız için “Password01” parolasının NTLM hash'ini hızlı bir şekilde oluşturabiliriz:


#### Python3 - Hashlib
{{CODE_BLOCK_59}}

Daha sonra ortaya çıkan NTLM parola hash değerini “7100a909c7ff05b266af3c42ec058c33” standart rockyou.txt kelime listesini kullanarak Hashcat üzerinden çalıştırıp açık metni alabiliriz.

#### Hashcat - Cracking NTLM Hashes
{{CODE_BLOCK_60}}
Şimdi, açık metin şifresiyle donanmış olarak, ağ içindeki erişimimizi daha da ileri götürebiliriz.



### NetNTLMv2
Bir sızma testi sırasında, kimlik bilgilerini “çalmak” amacıyla MITM saldırıları gerçekleştirmek için [Responder](https://github.com/lgandx/Responder) gibi araçları çalıştırmak yaygındır. Bu tür saldırılar diğer modüllerde ayrıntılı olarak ele alınmaktadır. Yoğun kurumsal ağlarda bu yöntemi kullanarak çok sayıda NetNTLMv2 parola hash'i elde etmek yaygındır. Bunlar genellikle kırılabilir ve Active Directory ortamında bir yer edinmek için kullanılabilir, hatta bazen parola karmasıyla ilişkili kullanıcı hesabına verilen ayrıcalıklara bağlı olarak birçok veya tüm sistemlere tam yönetici erişimi elde edilebilir. Bir değerlendirmenin başlangıcında Responder kullanılarak alınan aşağıdaki parola karmasını düşünün:

#### Responder - NTLMv2
{{CODE_BLOCK_61}}
Responder gibi bazı araçlar, ne tür bir hash alındığını size bildirecektir. Şüpheye düşersek Örnek hash'ler sayfasını da kontrol edebilir ve bunun gerçekten bir NetNTLMv2 hash'i veya Hashcat'te mod 5600 olduğunu doğrulayabiliriz.

Önceki örneklerde olduğu gibi, çevrimdışı bir sözlük saldırısı gerçekleştirmek için rockyou.txt kelime listesini kullanarak bu hash'i Hashcat ile çalıştırabiliriz.

--> https://hashcat.net/wiki/doku.php?id=example_hashes


#### Hashcat - Cracking NTLMv2 Hashes
{{CODE_BLOCK_62}}
Bu kimlik bilgileriyle donanmış olarak, yarışa başlayabiliriz ve daha sonraki modüllerde derinlemesine ele alınan Active Directory'yi numaralandırmaya ve saldırmaya başlayabiliriz.


Soru :  Aşağıdaki hash'i kırın: 7106812752615cdfe427e01b98cd4083
Cevap :
![Pasted image 20241017103239.png](/img/user/resimler/Pasted%20image%2020241017103239.png)
