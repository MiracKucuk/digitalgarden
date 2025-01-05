---
{"dg-publish":true,"permalink":"/baglantilar/php1/"}
---


### 1. **`session_start();`**

- **Amacı**: Kullanıcı oturumu başlatır veya mevcut bir oturumu devam ettirir.
- **Neden Kullanılır**: PHP'de kullanıcıya özel bilgileri taşımak (örneğin, oturum kimliği, kullanıcı adı) için oturum yönetimi gereklidir.

---

### 2. **`require_once ('db.php');`**

- **Amacı**: `db.php` adlı bir dosyayı dahil eder.
- **Neden Kullanılır**: `db.php` dosyasında genellikle veritabanı bağlantısı veya yardımcı işlevler bulunur.
- **`require_once`**: Bu, dosyanın yalnızca bir kez dahil edilmesini sağlar (aynı dosya tekrar tekrar yüklenmez).

---

### 3. **`if(!$_SESSION['user']){`**

- **Amacı**: Kullanıcının oturum açıp açmadığını kontrol eder.
- **Detaylar**:
    - **`$_SESSION`**: PHP'nin oturum değişkenlerini sakladığı yer.
    - **`!$_SESSION['user']`**: Eğer `$_SESSION['user']` yoksa veya boşsa, kullanıcı oturum açmamış demektir.
- **Sonuç**: Kullanıcı oturum açmamışsa, aşağıdaki kod çalışır:

---

### 4. **`header("Location: index.php");`**

- **Amacı**: Kullanıcıyı `index.php` sayfasına yönlendirir.
- **Neden Kullanılır**: Genellikle oturum açma sayfası `index.php`'dir. Oturum açmamış bir kullanıcıyı güvenli bir şekilde giriş sayfasına yönlendirir.

---

### 5. **`exit;`**

- **Amacı**: Kodun geri kalanını çalıştırmadan durdurur.
- **Neden Kullanılır**: Yönlendirme (`header`) işleminden sonra herhangi bir kod çalıştırılmasını istemiyoruz.

---

### 6. **`$_SESSION['id'] = $_GET['id'];`**

- **Amacı**: URL'den gelen `id` parametresini oturum değişkeni olarak saklar.
- **Detaylar**:
    - **`$_GET['id']`**: URL'de bir `id` parametresi varsa, bu değeri alır.
    - **`$_SESSION['id']`**: Bu değeri oturuma kaydeder.
- **Risk**: Doğrudan kullanıcıdan gelen bir parametre olduğu için, **input validation** (girdi doğrulama) yapılmamış. Bu, **güvenlik açıklarına** neden olabilir.

---

### 7. **`if(check_access($_SESSION['id'], $_SESSION['user'])){`**

- **Amacı**: Kullanıcının belirli bir kaynağa erişim izni olup olmadığını kontrol eder.
- **Detaylar**:
    - **`check_access`**: Bu bir işlevdir (fonksiyon). Muhtemelen `db.php` dosyasında tanımlıdır. Kullanıcının (`$_SESSION['user']`) ve ID'nin (`$_SESSION['id']`) ilişkisini kontrol eder.
    - **Sonuç**: Eğer kullanıcı yetkili ise, aşağıdaki kod çalışır:

---

### 8. **`header("Location: display_data.php");`**

- **Amacı**: Yetkili kullanıcıyı `display_data.php` sayfasına yönlendirir.
- **Neden Kullanılır**: `display_data.php` sayfası, yetkili kullanıcıların görmesi gereken bir içeriği gösterebilir.

---

### 9. **`exit;`**

- **Amacı**: Kodun geri kalanını çalıştırmadan durdurur.

---

### 10. **`else {`**

- **Amacı**: Eğer `check_access` işlevi **false** döndürürse (kullanıcının erişim yetkisi yoksa), bu bloğa girilir.

---

### 11. **`header("Location: error.php");`**

- **Amacı**: Yetkisiz kullanıcıyı `error.php` sayfasına yönlendirir.
- **Neden Kullanılır**: Kullanıcıya erişim yetkisi olmadığını bildirmek için.

---

### 12. **`exit;`**

- **Amacı**: Kodun geri kalanını çalıştırmadan durdurur.

---

### Genel İşleyiş

1. Oturum başlatılır.
2. Kullanıcı oturum açmamışsa giriş sayfasına yönlendirilir.
3. `id` parametresi alınır ve oturum değişkenine kaydedilir.
4. Kullanıcının erişim izni olup olmadığı kontrol edilir:
    - **Erişim varsa**: Kullanıcı `display_data.php` sayfasına yönlendirilir.
    - **Erişim yoksa**: Kullanıcı `error.php` sayfasına yönlendirilir.

---

### Güvenlik Tavsiyeleri

1. **Girdi Doğrulama**: `$_GET['id']` gibi kullanıcıdan gelen verilere doğrulama yapılmalı. Örneğin, bu değerin bir sayı olup olmadığı kontrol edilebilir.
2. **SQL Injection Koruması**: Eğer `check_access` işlevi bir veritabanı sorgusu içeriyorsa, **prepared statements** kullanılmalı.
3. **HTTPS**: Kullanıcı oturumu yönetirken güvenli bağlantı (HTTPS) kullanılmalıdır.

