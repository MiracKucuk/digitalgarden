---
{"dg-publish":true,"permalink":"/ctf/yummy-ctf/"}
---


### Nmap

```
echo "10.10.11.36 yummy.htb" | sudo tee -a /etc/hosts 
```

```
nmap yummy.htb -p- --min-rate 10000
```

![Pasted image 20250115161657.png](/img/user/Pasted%20image%2020250115161657.png)

```
nmap -p 22,80 -sCV yummy.htb
```

![Pasted image 20250115162002.png](/img/user/Pasted%20image%2020250115162002.png)

![Pasted image 20250115162043.png](/img/user/Pasted%20image%2020250115162043.png)

**HTTP Server Header**: Sunucu, kendisini "Caddy" olarak tanıtmış.


### Directory Brute Force

![Pasted image 20250115162631.png](/img/user/Pasted%20image%2020250115162631.png)
![Pasted image 20250115162713.png](/img/user/Pasted%20image%2020250115162713.png)
![Pasted image 20250115162756.png](/img/user/Pasted%20image%2020250115162756.png)
![Pasted image 20250115162828.png](/img/user/Pasted%20image%2020250115162828.png)
![Pasted image 20250115162842.png](/img/user/Pasted%20image%2020250115162842.png)

Çok fazla çıktı üretti genel olarak öğrendiklerimiz.

1. **Register ve Login Sayfaları**:
    
    - `http://yummy.htb/register`
    - `http://yummy.htb/login`
    
    Kullanıcı kaydı ve oturum açma gibi fonksiyonlar genellikle zafiyetlerin olduğu yerlerdir (ör. SQL Injection, Credential Stuffing, vb.).
    
2. **Dashboard ve Logout Yönlendirmeleri**:
    
    - `http://yummy.htb/dashboard`
    - `http://yummy.htb/logout`
    
    `Dashboard` genelde yalnızca oturum açmış kullanıcılar için erişilebilir olur. Ayrıca `logout` dizini yönlendirme içeriyor; burada oturum yönetimiyle ilgili açıklar araştırılabilir.
    
3. **Book Sayfası**:
    
    - `http://yummy.htb/book`
    
    Bu dizin, potansiyel olarak veri girilebilecek veya dinamik içerik sağlayan bir yer olabilir. Farklı HTTP metodları (POST gibi) denenerek işlevsellik incelenebilir.
    

4. **Statik Kaynaklardan İlginç Olabilecekler**:
    
    - **`/static/js/main.js`**: JavaScript dosyaları genelde istemci tarafında yürütülen kodları içerir. Bu kodlar bazen önemli API endpoint'lerini veya gizli parametreleri ortaya çıkarabilir.
    - **`/static/js/navbar.js` ve `/static/js/datetime.js`**: Navbar veya zamanla ilgili bir şey dinamik yükleniyor olabilir. Bunun içerikleri kontrol edilebilir.

5. **Potansiyel Olarak Test Edilebilecek Dizinler**:
    
    - `http://yummy.htb/static/vendor/`: Vendor dosyaları içinde eski veya zafiyetli bir kütüphane olabilir.
    - `http://yummy.htb/static/img/`: Görseller genelde önemli değildir, ancak bazen metadata veya açıklayıcı bilgiler içerir.
    


### Web Site

![Pasted image 20250115163443.png](/img/user/Pasted%20image%2020250115163443.png)

Web sayfasına gidin ve kullanıcı olarak kaydolun ve ardından oturum açın.

Web sayfasına gidin ve kullanıcı olarak kaydolun ve ardından oturum açın.

Ardından BOOK A TABLE'a gidin, istediğiniz bilgileri girin ve gönderin.

![Pasted image 20250115163820.png](/img/user/Pasted%20image%2020250115163820.png)

![Pasted image 20250115164019.png](/img/user/Pasted%20image%2020250115164019.png)

![Pasted image 20250115164038.png](/img/user/Pasted%20image%2020250115164038.png)

"Rezervasyon talebiniz gönderildi. Randevunuzu hesabınızdan daha fazla yönetebilirsiniz. Teşekkür ederiz! "

Ardından Dashborad'a geri dönün ve paketleri almak için Burpsuite'i kullanmak üzere bir indirme bağlantısı var

![Pasted image 20250115164457.png](/img/user/Pasted%20image%2020250115164457.png)

İlk GET isteğinin token'idir doğrudan izin veriliyor.

![Pasted image 20250115164734.png](/img/user/Pasted%20image%2020250115164734.png)

Sunucudan 500 hatası alırsanız, yeni bir paket yakalamanız ve repeater'a göndermeniz gerekir.
İkinci paket Repeater'a gönderilir ve URL parametreleri /etc/passwd dosyasını okuyacak şekilde değiştirilir.

![Pasted image 20250115165633.png](/img/user/Pasted%20image%2020250115165633.png)

Ve içinde iki mevcut kullanıcı bulundu: dev, qa

* **dev**: Kullanıcı hesabı, /home/dev dizininde bulunan ve bash shell'e sahip.
* **qa**: Test (Quality Assurance) amaçlı bir kullanıcı, /home/qa dizininde bulunan ve bash shell'e sahip.
* `_laurel`: `/var/log/laurel` dizininde bulunan ve `/bin/false` shelle sahip bir kullanıcı. Bu kullanıcı, oturum açma izni olmayan bir hesap gibi görünüyor. (dikkat'e alınmayabilir şimdilik.)


Ve Caddy'nin bazı sunucu yapılandırmalarını okuyabildim.

![Pasted image 20250115170331.png](/img/user/Pasted%20image%2020250115170331.png)

Ancak başka hiçbir yapılandırma dosyası veya SSH anahtarı doğrudan okunamadı, ancak zamanlanmış görev okunduğunda dosya yedeklemeye benzer bir komut dosyası ortaya çıktı

![Pasted image 20250115170429.png](/img/user/Pasted%20image%2020250115170429.png)
