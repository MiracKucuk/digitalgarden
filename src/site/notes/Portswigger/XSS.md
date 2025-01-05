---
{"dg-publish":true,"permalink":"/portswigger/xss/"}
---


![Pasted image 20241214011440.png](/img/user/resimler/Pasted%20image%2020241214011440.png)

Ne yazık ki, Chrome kullanıyorsanız küçük bir aksaklık var. Sürüm 92'den itibaren (20 Temmuz 2021), cross-origin iframe'lerin alert() fonksiyonunu çağırması engellenmiştir. Bunlar daha gelişmiş XSS saldırılarından bazılarını oluşturmak için kullanıldığından, bazen alternatif bir PoC payload kullanmanız gerekebilir. Bu senaryoda print() fonksiyonunu öneriyoruz. Bu değişiklik ve print() fonksiyonunu neden tercih ettiğimiz [hakkında daha fazla bilgi edinmek isterseniz](https://portswigger.net/research/alert-is-dead-long-live-print), konuyla ilgili blog yazımıza göz atın.

İşte reflected XSS güvenlik açığının basit bir örneği:

```
https://insecure-website.com/status?message=All+is+well. 
<p>Status: All is well.</p>
```

Uygulama veriler üzerinde başka herhangi bir işlem yapmaz, bu nedenle bir saldırgan bunun gibi bir saldırıyı kolayca oluşturabilir:

```
https://insecure-website.com/status?message=<script>/*+Bad+stuff+here...+*/</script> <p>Status: <script>/* Bad stuff here... */</script></p>
```

Kullanıcı saldırgan tarafından oluşturulan URL'yi ziyaret ederse, saldırganın komut dosyası kullanıcının tarayıcısında, kullanıcının uygulamadaki oturumu bağlamında yürütülür. Bu noktada, komut dosyası kullanıcının erişebildiği herhangi bir eylemi gerçekleştirebilir ve herhangi bir veriyi alabilir.

- [Reflected cross-site scripting](https://portswigger.net/web-security/cross-site-scripting/reflected)
- [Cross-site scripting cheat sheet](https://portswigger.net/web-security/cross-site-scripting/cheat-sheet)


----

### Lab: Hiçbir şey kodlanmamış HTML bağlamına reflected XSS

Bu laboratuvar, search işlevinde basit bir reflected cross-site scripting güvenlik açığı içermektedir.

Laboratuvarı çözmek için alert işlevini çağıran bir siteler arası komut dosyası saldırısı gerçekleştirin.

1. Aşağıdakileri kopyalayın ve arama kutusuna yapıştırın:

2. `<script>alert(1)</script>`

3. “ Search ”e tıklayın.

----

### Lab: HTML bağlamına hiçbir şey kodlanmadan Stored XSS

1. Yorum kutusuna aşağıdakileri girin:

`<script>alert(1)</script>`

2. Bir isim, e-posta ve web sitesi girin.
3. “Yorum gönder “e tıklayın.
4. Bloga geri dönün.


---

### Laboratuvar: Kaynak location.search kullanarak document.write lavabosunda DOM XSS

Bu laboratuvar, search query tracking fonksiyonelliğinde DOM tabanlı bir cross-site scripting güvenlik açığı içermektedir. Sayfaya veri yazan JavaScript document.write işlevini kullanır. document.write işlevi, web sitesi URL'sini kullanarak kontrol edebileceğiniz location.search'ten gelen verilerle çağrılır.

Bu laboratuvarı çözmek için alert işlevini çağıran bir siteler arası komut dosyası saldırısı gerçekleştirin.

1. Arama kutusuna rastgele bir alfanümerik dize girin.
2. Öğeye sağ tıklayıp inceleyin ve rastgele dizenizin bir img src niteliğinin içine yerleştirildiğini gözlemleyin.
3. Şunu arayarak img özniteliğinden çıkın:

`"><svg onload=alert(1)>`

![Pasted image 20241214023955.png](/img/user/resimler/Pasted%20image%2020241214023955.png)

![Pasted image 20241214024117.png](/img/user/resimler/Pasted%20image%2020241214024117.png)

----


### Laboratuvar: Kaynak location.search kullanarak innerHTML sink içinde DOM XSS

Bu laboratuvar, search blog işlevinde DOM tabanlı bir cross-site scripting güvenlik açığı içermektedir. Bir div öğesinin HTML içeriğini değiştiren ve location.search'ten veri kullanan bir innerHTML ataması kullanır.

Bu laboratuvarı çözmek için alert fonksiyonunu çağıran bir cross-site scripting saldırısı gerçekleştirin.

1. Arama kutusuna aşağıdakileri girin:

`<img src=1 onerror=alert(1)>`

2. Click "Search".

src niteliğinin değeri geçersizdir ve bir hata verir. Bu, onerror olay işleyicisini tetikler ve ardından alert() fonksiyonunu çağırır. Sonuç olarak, kullanıcının tarayıcısı kötü amaçlı gönderinizi içeren sayfayı yüklemeye çalıştığında payload çalıştırılır.

![Pasted image 20241214024755.png](/img/user/resimler/Pasted%20image%2020241214024755.png)


----


### Laboratuvar: jQuery anchor `href` attribute sink içinde `location.search` kaynağını kullanarak DOM XSS

Bu laboratuvar, geri bildirim gönderme sayfasında DOM tabanlı cross-site scripting güvenlik açığı içermektedir. Bir bağlantı öğesi bulmak için jQuery kütüphanesinin $ selector fonksiyonunu kullanır ve location.search'ten veri kullanarak href niteliğini değiştirir.

Bu laboratuvarı çözmek için “ back” bağlantısını alert document.cookie yapın.

1. Submit feedback sayfasında, returnPath sorgu parametresini / ve ardından rastgele bir alfanümerik string olarak değiştirin.

2. Öğeye sağ tıklayıp inceleyin ve rastgele stringinizin bir a href niteliğinin içine yerleştirildiğini gözlemleyin.

3. returnPath olarak değiştirin:

```
javascript:alert(document.cookie)
```

4. Enter tuşuna basın ve “ back” e tıklayın.

![Pasted image 20241214030013.png](/img/user/resimler/Pasted%20image%2020241214030013.png)

![Pasted image 20241214030023.png](/img/user/resimler/Pasted%20image%2020241214030023.png)

![Pasted image 20241214030029.png](/img/user/resimler/Pasted%20image%2020241214030029.png)


---
