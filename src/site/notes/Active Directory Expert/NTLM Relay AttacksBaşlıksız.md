---
{"dg-publish":true,"permalink":"/active-directory-expert/ntlm-relay-attacks-basliksiz/"}
---


### NTLM Authentication Protocol

Active Directory ağlarıyla olan çalışmalarımız sırasında, Kerberos en yaygın olanı olmak üzere, hostların kullandığı farklı kimlik doğrulama protokolleriyle karşılaşacağız. Bununla birlikte, başka bir protokol olan NTLM , bilinen güvenlik açıklarına ve kriptografik zayıflıklarına rağmen hala yaygın olarak kullanılmaktadır. NTLM'nin şirket içinde nasıl çalıştığını anlamak, ona etkili bir şekilde saldırmak ve başarılı bir etkileşim olasılığını artırmak için çok önemlidir.


### NTLM
[[Bağlantılar/NT Lan Manager\|NT Lan Manager]] (of LM, NTLM / MS-NLMP), çeşitli Windows tabanlı ağlardaki uygulama protokolleri tarafından remote kullanıcıların kimliğini doğrulamak ve isteğe bağlı olarak uygulama tarafından istendiğinde oturum güvenliği sağlamak için kullanılan NTLMv1 ve NTLMv2'den oluşan bir güvenlik protokolleri ailesinin adıdır. NTLM güvenlik protokollerinin tümü gömülü protokollerdir, yani NTLM diğer protokoller gibi mesajlara ve bir durum makinesine sahip olmasına rağmen, bir ağ protokolü stack katmanına sahip değildir. NTLM'nin bu yapısı, ağ stack'inde tanımlı bir katmana sahip herhangi bir protokolün (SMB, HTTP ( S) ve onu kullanan uygulama protokolü LDAP ( S) gibi) onu kullanmasına izin verir. NTLMv2 için üç temel işlem sağlar:

1. Kimlik doğrulama.
2. Mesaj bütünlüğü, NTLM terminolojisinde **message signing** olarak bilinir.
3. Mesaj gizliliği, NTLM terminolojisinde **message sealing** olarak bilinir.

NTLM, nonces adı verilen ve bir kezlik kullanım için üretilen pseudo-random numbers'ı replay attacks'e karşı bir savunma mekanizması olarak kullanan bir challenge-response protokolüdür. Her protokolün connection-oriented ve connectionless olmak üzere iki varyantı olmasına rağmen, biz öncelikli olarak birincisiyle ilgileneceğiz (ikisinin farklarını öğrenmek için MS-NLMP'deki [Overview]([[MS-NLMP]: Overview | Microsoft Learn](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/c50a85f0-5940-42d8-9e82-ed206902e919)) kısmına bakabilirsiniz).

Normal protokol implementasyonlarının aksine, NTLM bir network protocol stack'in bir katmanı olarak değil, application protocols tarafından çağrılabilen bir function library olarak en iyi şekilde implement edilir. Windows authentication'ın temeli olan [Security Support Provider Interface](https://learn.microsoft.com/en-us/windows/win32/rpc/sspi-architectural-overview) (SSPI), bağlı uygulamaların authenticated connections kurmak ve bu bağlantılar üzerinden verileri güvenli bir şekilde değiştirmek için birden fazla security provider'dan birini çağırmasına olanak tanıyan bir API'dir. [Security support provider ](https://learn.microsoft.com/en-us/windows/win32/rpc/security-support-providers-ssps-)(SSP), SSPI'yi implement etmekten sorumlu olan ve uygulamalara bir veya daha fazla security package sunan bir dynamic-link library (DLL)'dir. Her security package, bir application's SSPI function calls ile bir security model'in gerçek fonksiyonları arasında mapping sağlar. Bu security packages, [NTLM](https://learn.microsoft.com/en-us/windows/win32/secauthn/microsoft-ntlm) dahil olmak üzere çeşitli security protocols'u destekler.

**NTLM'in protokol implementasyonu:**

- **Cümle:** _"Normal protokol implementasyonlarının aksine, NTLM bir network protocol stack'in bir katmanı olarak değil, application protocols tarafından çağrılabilen bir function library olarak en iyi şekilde implement edilir."_
- **Açıklama:**
    - Genellikle protokoller, bir ağ yığını (network protocol stack) içinde ayrı bir katman olarak tasarlanır. Ancak, NTLM bir katman olarak değil, bir **function library** (işlev kütüphanesi) şeklinde tasarlanır.
    - **Function library**, başka uygulama protokollerinin çağırabileceği bir yazılım paketi gibidir. Örneğin, bir uygulama NTLM'i çağırarak kullanıcı kimlik doğrulamasını gerçekleştirebilir.
    - Bu, NTLM'in **bağımsız bir protokol gibi değil**, başka protokollerle birlikte çalışacak şekilde tasarlandığını ifade eder.

[**NTLM SSP**](https://learn.microsoft.com/en-us/windows-server/security/windows-authentication/security-support-provider-interface-architecture#BKMK_NTLMSSP) (%Windir%\System32\msv1_0.dll konumunda bulunan) **SSPI** tarafından NTLM challenge-response kimlik doğrulamasını kolaylaştırmak ve bütünlük (integrity) ile gizlilik (confidentiality) için seçenekleri müzakere etmek amacıyla kullanılan bir ikili mesajlaşma protokolüdür. NTLM SSP, hem **NTLM** hem de **NTLMv2** kimlik doğrulama protokollerini kapsar. Bu bölümde, NTLM protokolünün iç işleyişini ve **state messages**'lerini anlamaya çalışacağız, NTLM SSP'nin bunları uygulamalara nasıl sağladığını veya NTLM mesajlarını bir ağda nasıl ilettiğini tartışmak yerine.


**Hem domain'e katılmış hem de workgroup bilgisayarlar NTLM'i authentication (kimlik doğrulama) için kullanabilir; ancak, biz AD (Active Directory) environments (ortamları) hedef alacağımız için öncelikli olarak domain'e katılmış bilgisayarları ele alacağız. Bu bölümün geri kalanını okurken, her zaman şu noktaları aklınızda bulundurmalısınız:**

- **NTLM** version (versiyonu), **NTLMv1** veya **NTLMv2**, authentication (kimlik doğrulama) işleminden önce [out-of-band (bağımsız olarak)](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-lan-manager-authentication-level) yapılandırılır.
- Güvenli bir mekanizma kullanılarak, client (istemci) ve server/DC (sunucu/DC) authentication (kimlik doğrulama) işleminden önce secret key (gizli anahtar) paylaşır (kullanıcının şifresinin hash'ini).
- Hem plaintext credentials (düz metin kimlik bilgileri) ne de paylaşılan **secret key** (gizli anahtar) network (ağ) üzerinden gönderilmez.


**Out-of-band**: Kimlik doğrulama sürecinden önce, ağ üzerinden yapılmayan, ayrı bir mekanizma ile yapılan yapılandırma.


### Authentication Workflow

Domain'e katılmış bilgisayarlar için NTLM authentication (kimlik doğrulama) iş akışı, client (istemci) ile server (sunucu) arasında implementasyona özgü application protocol messages (uygulama protokol mesajları) alışverişine başlanmasıyla başlar, ve client (istemci) kimlik doğrulaması yapmak istediğini belirtir. Sonrasında, client (istemci) ve server (sunucu), authentication (kimlik doğrulama) sırasında üç NTLM-specific (NTLM'e özgü) mesajı karşılıklı olarak alışveriş eder (uygulama protokol mesajları içinde gömülü olarak):

- [**NEGOTIATE_MESSAGE**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/b34032e5-3aae-4bc6-84c3-c6d80eadf7f2) (aynı zamanda **Type 1 message** olarak bilinir)
- [**CHALLENGE_MESSAGE**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/801a4681-8809-4be9-ab0d-61dcfe762786) (aynı zamanda **Type 2 message** olarak bilinir)
- [**AUTHENTICATE_MESSAGE**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/033d32cc-88f9-4483-9bf2-b273055038ce) (aynı zamanda **Type 3 message** olarak bilinir)

**AUTHENTICATE_MESSAGE** alındıktan sonra ve **client**'ın gizli anahtarına sahip olmadığı için, **server**, kullanıcının kimliğini doğrulama işlemini bir **DC**'ye (bu prosedür [**Pass-through authentication**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nrpc/70697480-f285-4836-9ca7-7bb52f18c6af) olarak bilinir) devreder. Bu işlem, **[NetrLogonSamLogonWithFlags](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nrpc/d17f1077-de4b-4fcd-8867-39068cb789f5)** fonksiyonu çağrılarak yapılır; bu fonksiyon, [**NETLOGON_NETWORK_INFO**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nrpc/e17b03b8-c1d2-43a1-98db-cf8d05b9c6a8) adlı veri yapısını içerir ve bu yapı, **DC**'nin kullanıcıyı doğrulamak için ihtiyaç duyduğu çeşitli alanlarla doldurulur. Kimlik doğrulama başarılı olursa, **DC**, [**NETLOGON_VALIDATION_SAM_INFO4**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nrpc/bccfdba9-0c38-485e-b751-d4de1935781d) veri yapısını **server**'a döndürür ve **server**, **client** ile doğrulanmış bir oturum başlatır; aksi takdirde, **DC** bir hata döndürür ve **server** hata mesajı gönderebilir ya da bağlantıyı sonlandırabilir.

Aşağıdaki diyagram [NTLM'nin pass-through](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-apds/5bfd942e-7da5-494d-a640-f269a0e3cc5d) kimlik doğrulaması:

![Pasted image 20241205024700.png](/img/user/resimler/Pasted%20image%2020241205024700.png)

Workgroup kimlik doğrulaması için tek fark, sunucunun kullanıcının kimliğini DC'ye devretmek yerine doğrulamasıdır:

![Pasted image 20241205024739.png](/img/user/resimler/Pasted%20image%2020241205024739.png)

Üç ana NTLM mesajını, içlerinde ne gönderildiğini anlamak için inceleyeceğiz.


### NTLM Messages
Her NTLM mesajı değişken uzunluktadır ve sabit uzunluktaki bir başlık ile değişken boyutlu bir **payload** (mesaj yükü) içerir. Başlık her zaman **Signature** ve **MessageType** alanlarıyla başlar ve sonrasındaki **MessageType**'a bağlı olarak, mesajlar ek olarak mesaj tipine özgü sabit uzunluktaki alanlar içerebilir. Bu alanlardan sonra değişken uzunluktaki **payload** gelir.

![Pasted image 20241205024933.png](/img/user/resimler/Pasted%20image%2020241205024933.png)

**Signature**: 8 baytlık NULL-terminated ASCII dizesi, her zaman [T, L, M, S, S, P, \0] olarak ayarlanır.

**MessageType**: 4 baytlık işaretsiz tam sayı, her zaman şu şekilde ayarlanır:

- 0x00000001 (NtLmNegotiate) — NTLM mesajının **NEGOTIATE_MESSAGE** olduğunu belirtir,
- 0x00000002 (NtLmChallenge) — NTLM mesajının **CHALLENGE_MESSAGE** olduğunu belirtir,
- 0x00000003 (NtLmAuthenticate) — NTLM mesajının **AUTHENTICATE_MESSAGE** olduğunu belirtir.

**MessageDependentFields**: NTLM mesaj içeriğini içeren değişken uzunluktaki alan.

**payload**: Mesaj bağımlı bir dizi bireysel **payload** mesajını içeren değişken uzunluktaki alan, bu mesajlar **MessageDependentFields** içindeki bayt ofsetleriyle referans alınır.


### NEGOTIATE_MESSAGE

**[NEGOTIATE_MESSAGE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/b34032e5-3aae-4bc6-84c3-c6d80eadf7f2)**, client'ın kimlik doğrulama yapmak istediğini ve desteklediği/istekli olduğu NTLM seçeneklerini belirten ilk NTLM mesajıdır. Bu mesaj, dört mesaj bağımlı sabit uzunlukta alandan oluşur. Bilinmesi gereken önemli bir alan, yalnızca **NegotiateFlags**'i belirten alan olup, bu 4 baytlık alan, tüm üç NTLM mesajında ve **NEGOTIATE_MESSAGE**'de bulunan bir **[NEGOTIATE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/99d90ff4-957f-4c8a-80e4-5bfe5a9a9832)** yapısını içerir. Bu yapı, göndericinin desteklediği/istekli olduğu NTLM yeteneklerini belirten 32 adet 1-bit bayt bayrağından oluşur.


### CHALLENGE_MESSAGE

 **[CHALLENGE_MESSAGE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/801a4681-8809-4be9-ab0d-61dcfe762786)**, client'a kimliğini kanıtlaması için meydan okuma gönderen ve server tarafından gönderilen NTLM'e özgü ikinci mesajdır.
    
- Bu mesaj, altı mesaj bağımlı sabit uzunlukta alandan oluşur.
    
- Bilinmesi gereken iki önemli alan vardır:
    
    - **NegotiateFlags**: Server, **NEGOTIATE_MESSAGE** içindeki **NegotiateFlags** alanında client tarafından teklif edilen/istek edilen seçeneklerden seçtiği bayrakları burada tutar.
    - **ServerChallenge**: Server tarafından oluşturulan 64-bitlik bir NTLM meydan okuma değeridir.
- Bazı araçlar, örneğin **[NTLM Challenger](https://github.com/nopfor/ntlm_challenger)**, [**ntlm-info**](https://gitlab.com/Zer1t0/ntlm-info), [DumpNTLMInfo.py ](https://github.com/fortra/impacket/blob/impacket_0_11_0/examples/DumpNTLMInfo.py)**CHALLENGE_MESSAGE**'i çözümleyerek NTLM kimlik doğrulama bilgilerini elde etmek amacıyla uç noktalara/hostlara karşı keşif yapar.
    
- Bu araçlar, nonce içinde dönen **[NTLMRecon](https://github.com/praetorian-inc/NTLMRecon)**'u tutan bilgileri kullanarak **CHALLENGE_MESSAGE**'in diğer alanlarını çözümleyebilir.
    
- **CHALLENGE_MESSAGE**'in diğer alanlarını gözden geçirerek bu tür keşiflerin mümkün olmasının nedenlerini daha fazla anlayabilirsiniz.

![Pasted image 20241205030008.png](/img/user/resimler/Pasted%20image%2020241205030008.png)

Bu çıktı, **DumpNTLMInfo.py** aracı kullanılarak yapılan bir NTLM kimlik doğrulama keşfi işlemine ait verilerdir. Çıktıyı adım adım açıklayalım:

1. **SMBv1 Enabled: False**
    
    - SMBv1 protokolü etkin değil. Bu, SMBv1'in kullanımdan kaldırıldığını ve daha yeni sürümlerin (SMB 2.x ve 3.x) kullanıldığını gösteriyor.
2. **Prefered Dialect: SMB 3.0**
    
    - Server, en uygun protokol sürümü olarak **SMB 3.0**'ı tercih ediyor. Bu, daha güvenli ve özellik açısından zengin bir sürümdür.
3. **Server Security: SIGNING_ENABLED | SIGNING_REQUIRED**
    
    - Sunucu, **imza (signing)** özelliğini etkinleştirmiş ve bu özellik zorunlu hale getirilmiş. Bu, veri bütünlüğü ve kimlik doğrulama sağlamak için kullanılır.
4. **Max Read Size: 8.0 MB (8388608 bytes)**
    
    - Sunucunun, tek bir okuma işlemi için maksimum 8 MB veri okuyabileceği belirtiliyor.
5. **Max Write Size: 8.0 MB (8388608 bytes)**
    
    - Aynı şekilde, sunucunun tek bir yazma işlemi için maksimum 8 MB veri yazabileceği belirtiliyor.
6. **Current Time: 2023-08-14 17:39:26.822236+00:00**
    
    - Sunucunun şu anki zamanı: 14 Ağustos 2023, 17:39 UTC.
7. **Name: DC01**
    
    - Sunucunun adı **DC01**. Bu, bir Domain Controller (DC) olduğunu ve AD ortamında yer aldığını gösteriyor.
8. **Domain: INLANEFREIGHT**
    
    - Sunucu, **INLANEFREIGHT** domainine ait.
9. **DNS Tree Name: INLANEFREIGHT.LOCAL**
    
    - DNS ağ ağacının adı **INLANEFREIGHT.LOCAL**. Bu, domainin DNS adına işaret eder.
10. **OS: Windows NT 10.0 Build 17763**
    
    - Sunucunun işletim sistemi **Windows NT 10.0 Build 17763**, yani Windows Server 2019 veya daha yeni bir sürüm olduğunu gösteriyor.
11. **DNS Domain Name: INLANEFREIGHT.LOCAL**
    
    - DNS domain adı tekrar **INLANEFREIGHT.LOCAL** olarak belirtilmiş, bu da domain adını gösterir.
12. **DNS Host Name: DC01.INLANEFREIGHT.LOCAL**
    
    - Sunucunun tam DNS ana bilgisayar adı: **DC01.INLANEFREIGHT.LOCAL**.
13. **Null Session: True**
    
    - **Null Session** özelliği etkin. Bu, kimlik doğrulama gerektirmeden ağda bazı bilgilere erişim sağlanabileceğini belirten bir durumdur. Bu özellik, bazı sistemlerde keşif ve ağdaki hizmetlere erişim sağlama amacıyla kullanılabilir.

Bu çıktı, genellikle ağda keşif yapmak amacıyla kullanılan ve SMB, NTLM gibi protokollere dair bilgileri içeren bir rapordur. Özellikle **Null Session**'ın etkin olması, saldırganların ağda keşif yapabilmesi için bir fırsat oluşturabilir, çünkü kimlik doğrulama yapılmadan bazı bilgilere erişim sağlanabilir.


### AUTHENTICATE_MESSAGE
[AUTHENTICATE_MESSAGE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/033d32cc-88f9-4483-9bf2-b273055038ce), paylaşılan gizli anahtara sahip olduğunu kanıtlamak için client tarafından sunucuya gönderilen NTLM'ye özgü üçüncü ve son mesajdır. LmChallengeResponseFields ve NtChallengeResponseFields olmak üzere mesaja bağlı dokuz sabit uzunluklu alan içerir. MS-NLMP tarafından sağlanan sözde kod için hide01.ir[ impacket'in NTLM'sinin](https://github.com/fortra/impacket/blob/master/impacket/ntlm.py) ilgili uygulamasına da başvuracağız.


### NTLMv1 Response Calculation

1. **NTLMSSP_NEGOTIATE_LM_KEY** (G biti):
    
    - Bu bayrak, istemci ve sunucu arasında **NegotiateFlags**'ta ([NEGOTIATE](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/99d90ff4-957f-4c8a-80e4-5bfe5a9a9832) mesajında) anlaşılmışsa, **LM_KEY** kullanılarak kimlik doğrulama işlemi yapılır. Bu, istemci ve sunucu arasındaki NTLM kimlik doğrulama sürecinin bir parçasıdır.
2. **NTLMv1 veya NTLMv2 Seçimi**:
    
    - Eğer **NTLMv1** kullanılıyorsa, **LmChallengeResponseFields** içinde [**LM_RESPONSE** ](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/e3fee6d1-0d93-4020-84ab-ca4dc5405fc9)yapısı bulunur.
    - Eğer **NTLMv2** kullanılıyorsa, **LmChallengeResponseFields** içinde [**LMv2_RESPONSE**](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/8659238f-f5a9-44ad-8ee7-f37d3a172e56) yapısı bulunur.
3. **LM_RESPONSE Yapısı (NTLMv1)**:
    
    - **LM_RESPONSE** yapısı, sadece bir alan içerir: **Response**.
    - **Response**, istemcinin **LmChallengeResponse**'ını içeren 24 byte'lık bir [unsigned char ](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-dtyp/050baef1-f978-4851-a3c7-ad701a90e54a)dizisidir.
4. **LMv2_RESPONSE Yapısı (NTLMv2)**:
    
    - **LMv2_RESPONSE** yapısı, iki alan içerir:
        - **Response**: 16 byte'lık bir unsigned char dizisi olup, istemcinin LM challenge-response'ını içerir.
        - **ChallengeFromClient**: 8 byte'lık bir unsigned char dizisi olup, istemci tarafından üretilen bir challenge'ı içerir.
5. **LM_RESPONSE ve LMv2_RESPONSE Hesaplama**:
    
    - **Response** alanını hesaplamak için **MS-NLMP** (Microsoft Network Logon Protocol) tarafından sağlanan bir [psödokod](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/1b72429a-d8b8-4a04-bc82-1eedc980b87a) vardır.
    - Bu psödokod, istemcinin **Response** değerini hesaplamak için kullanılan algoritmayı tanımlar.

Özetle, bu açıklamalar NTLMv1 ve NTLMv2'deki **LmChallengeResponse** hesaplamalarıyla ilgilidir. **LM_RESPONSE** veya **LMv2_RESPONSE** yapılarının içeriği, hangi NTLM versiyonunun kullanıldığına bağlı olarak değişir. Bu yapıların içinde istemcinin kimlik doğrulamasını gerçekleştirmek için gerekli olan veriler bulunur ve bu veriler hesaplanarak sunucuya gönderilir.

[NTLMv1 kimlik doğrulaması](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/464551a8-9fc4-428e-b3d3-bc5bfb2e73a5) için,[ NTOWFv1](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/780943e9-42e6-4dbe-aa87-1dce828ba82a#gt_7a2805fa-1dcd-4b4e-a8e4-2a2bcc8651e9) (yalnızca NTLMv1 tarafından kullanılır ve [compute_nthash ](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L759-L769)işlevinde uygulanır), bir NT LAN Manager ( NT) tek yönlü işlevidir ve bir sorumlunun güvenlik anahtarını oluşturmak için kullanıcının parolasını temel alan bir hash oluşturur: kullanıcının düz metin parolasını kullanmak yerine, bu fonksiyonun sonuç hash'i yanıtın hesaplanmasında kullanılır. Yalnızca LM ve NTLMv1 tarafından kullanılan [LMOWFv1](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/780943e9-42e6-4dbe-aa87-1dce828ba82a#gt_fd74ef50-cb97-4acd-b537-4941bdd9e064) ([compute_lmhash](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L742C1-L747) işlevinde uygulanır), bir NT LAN Manager ( LM) tek yönlü işlevidir ve ayrıca bir sorumlunun güvenlik anahtarını oluşturmak için kullanıcının parolasına dayalı bir hash oluşturur. Client, sunucuya döndürdüğü yanıtı hesaplamak için bu iki işlevi kullanır; yanıt hesaplama [sözde kodu](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/464551a8-9fc4-428e-b3d3-bc5bfb2e73a5) [computeResponseNTLMv1](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L717-L740) işlevinde uygulanır

1. **NTOWFv1** (NT One-Way Function):
    
    - **Amacı**: Sadece **NTLMv1** tarafından kullanılan bir fonksiyondur.
    - **Nasıl çalışır**: **NTOWFv1** fonksiyonu, kullanıcının şifresine dayalı bir hash oluşturur. Bu hash, bir güvenlik anahtarı oluşturmak için kullanılır. Kullanıcının düz metin şifresi yerine, bu fonksiyonun ürettiği hash, yanıtı hesaplamak için kullanılır.
    - **Fonksiyonellik**: **NTOWFv1** tarafından üretilen hash, sunucuya gönderilecek yanıtın hesaplanmasında kullanılır.
2. **LMOWFv1** (LM One-Way Function):
    
    - **Amacı**: **LM** ve **NTLMv1** tarafından kullanılan bir fonksiyondur.
    - **Nasıl çalışır**: **LMOWFv1**, **NTOWFv1**'e benzer şekilde çalışır, ancak sadece **LM** ve **NTLMv1** protokollerinde kullanılır. Kullanıcının şifresine dayalı olarak bir hash oluşturur ve bu hash, güvenlik anahtarı oluşturmak için kullanılır.
    - **Fonksiyonellik**: **LMOWFv1** tarafından üretilen hash, **LM** veya **NTLMv1** kimlik doğrulama sürecinde yanıt hesaplamak için kullanılır.
3. **compute_nthash** Fonksiyonu:
    
    - Bu fonksiyon, **NTOWFv1**'i uygular. Kullanıcının şifresini kullanarak, NTLMv1 kimlik doğrulama sürecinde kullanılacak bir hash hesaplar.
4. **compute_lmhash** Fonksiyonu:
    
    - Bu fonksiyon, **LMOWFv1**'i uygular. Kullanıcının şifresine dayalı olarak, **LM** veya **NTLMv1** kimlik doğrulaması için kullanılacak bir hash hesaplar.
5. **Yanıt Hesaplama**:
    
    - İstemci, hem **NTOWFv1** hem de **LMOWFv1**'i kullanarak, sunucuya göndereceği yanıtı hesaplar.
    - Yanıt, **NTOWFv1** ve **LMOWFv1** tarafından üretilen hash'lere dayanır.
    - **computeResponseNTLMv1**: Bu fonksiyon, yanıtın hesaplanması için kullanılan psödokodu içerir ve hesaplama, iki fonksiyonun ürettiği hash'leri kullanarak yapılır.

### Özet:

- **NTOWFv1** ve **LMOWFv1**, **NTLMv1** kimlik doğrulama sürecinde kullanılan temel fonksiyonlardır.
- **NTOWFv1**, NT hash'ini ve **LMOWFv1**, LM hash'ini oluşturur; her ikisi de kullanıcının şifresine dayanır.
- İstemci, bu hash'leri kullanarak, sunucuya doğrulama için gönderilecek yanıtı hesaplar.


NtChallengeResponseFields için, NTLMv1 kullanılıyorsa, NtChallengeResponse bir NTLM_RESPONSE yapısı içerecektir, aksi takdirde, NTLMv2 kullanılıyorsa, NtChallengeResponse bir NTLMv2_RESPONSE yapısı içerecektir; NTLM_RESPONSE, Client'ın NtChallengeResponse'unu içeren 24 baytlık bir unsigned char dizisi olan Response olan bir alan içerir. NTLMv2_RESPONSE için, Response ve bir NTLMv2_CLIENT_CHALLENGE yapısı olmak üzere iki alan içerir; Response, client'ın NtChallengeResponse'unu içeren 16 baytlık bir işaretsiz karakter dizisidir, NTLMv2_CLIENT_CHALLENGE ise ChallengeFromClient dahil olmak üzere sekiz sabit uzunluklu değişken içeren değişken uzunluklu bir bayt dizisidir. Responder (bir sonraki bölümde öğreneceğimiz güçlü bir araç) kullanarak bir NTLMv1 hash'i yakalayacak olursak, User::HostName:LmChallengeResponse:NtChallengeResponse:ServerChallenge biçimini kullanarak görüntüleyecektir:


1. **NtChallengeResponseFields**:
    
    - **NTLMv1 kullanıldığında**: `NtChallengeResponse`, **NTLM_RESPONSE** yapısını içerecektir.
    - **NTLMv2 kullanıldığında**: `NtChallengeResponse`, **NTLMv2_RESPONSE** yapısını içerecektir.
2. **NTLM_RESPONSE Yapısı (NTLMv1 için)**:
    
    - **Response**: Bu alan, **NTLM_RESPONSE** yapısının tek alanıdır.
    - **Response**: 24 baytlık bir `unsigned char` dizisidir ve istemcinin **NtChallengeResponse** değerini içerir.
        - Bu değer, istemcinin sunucuya yanıt olarak gönderdiği NTLM yanıtıdır.
3. **NTLMv2_RESPONSE Yapısı (NTLMv2 için)**:
    
    - **Response**: 16 baytlık bir `unsigned char` dizisidir ve istemcinin **NtChallengeResponse** değerini içerir.
        - **Response**, istemcinin **NTLMv2** ile yapılan kimlik doğrulama sürecinde sunucuya gönderdiği yanıtı temsil eder.
    - **NTLMv2_CLIENT_CHALLENGE**: Bu, **NTLMv2_RESPONSE** yapısında bulunan ikinci alandır.
        - **NTLMv2_CLIENT_CHALLENGE**, değişken uzunluktadır ve içinde sekiz sabit uzunluktan oluşan veri içerir. Bu sekiz sabit veri arasında **ChallengeFromClient** de bulunur.
4. **NTLMv1 Hash Yakalama (Responder Kullanarak)**:
    
    - Eğer bir **NTLMv1 hash**'i **Responder** gibi bir araç kullanarak yakalarsak, bu hash aşağıdaki formatta görüntülenir:
        - `User::HostName:LmChallengeResponse:NtChallengeResponse:ServerChallenge`
        - **User**: Kullanıcı adı.
        - **HostName**: İstemcinin hostname bilgisi.
        - **LmChallengeResponse**: LM yanıtı.
        - **NtChallengeResponse**: NTLM yanıtı.
        - **ServerChallenge**: Sunucudan gelen zorlama değeri.

### Detaylı Açıklamalar:

- **NTLMv1 ve NTLMv2 Farkları**:
    
    - **NTLMv1**: Eskiden kullanılan ve güvenlik açısından daha zayıf olan bir protokoldür. Yanıtlar 24 baytlık `NTLM_RESPONSE` yapısı ile hesaplanır.
    - **NTLMv2**: Daha güvenli bir protokoldür ve yanıt hesaplaması için **NTLMv2_RESPONSE** kullanır. Bu yanıtta ayrıca **NTLMv2_CLIENT_CHALLENGE** yapısı yer alır.
- **Responder Aracı**:
    
    - **Responder**, bir tür **man-in-the-middle (MITM)** aracı olup, ağda NTLM kimlik doğrulama süreci sırasında hash’leri yakalamak için kullanılır. Bu araç, NTLMv1 ve NTLMv2 hash'lerini yakalayarak, daha sonra bunları çözmeye çalışabilir.

### Özet:

- **NTLMv1** ve **NTLMv2** arasındaki farklar, yanıt yapılarını etkiler. **NTLMv1** daha basit bir yapıya sahipken, **NTLMv2** daha güvenlidir ve daha karmaşık bir yanıt yapısına sahiptir.
- **Responder** aracı, NTLM kimlik doğrulama süreçlerini izleyerek hash'leri yakalar ve bunları analiz etmek için kullanılır.

![Pasted image 20241205032638.png](/img/user/resimler/Pasted%20image%2020241205032638.png)


### NTLMv2 Response Calculation

[NTLMv2 kimlik doğrulamasının](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/5e550938-91d4-459f-b67d-75d70009e3f3) yanıt hesaplaması için, [NTOWFv2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/780943e9-42e6-4dbe-aa87-1dce828ba82a#gt_ba118c39-b391-4232-aafa-a876ee1e9265) ve [LMOWFv2](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/780943e9-42e6-4dbe-aa87-1dce828ba82a#gt_a043ea96-e876-4259-be4b-aa8d2335fdfe) (her ikisi de versiyona bağlıdır ve yalnızca NTLMv2 tarafından kullanılır, sırasıyla [LMOWFv2](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L896C1-L897) ve [NTOWFv2](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L889C1-L894) fonksiyonlarında uygulanır), bir sorumlunun güvenlik anahtarını oluşturmak için kullanıcının parolasına dayalı bir hash oluşturmak için kullanılan tek yönlü fonksiyonlardır. Bu iki fonksiyon ile client, sunucuya döndüreceği yanıtı [pseudo](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/5e550938-91d4-459f-b67d-75d70009e3f3) kodda açıklandığı gibi hesaplar ([computeResponseNTLMv2](https://github.com/fortra/impacket/blob/9a8d27034eab20d23802730d0c69bf99356d8af1/impacket/ntlm.py#L900-L937) fonksiyonunda uygulanmıştır). Responder kullanarak bir NTLMv2 hash'i yakalayacak olursak, User::Domain:ServerChallenge:Response:NTLMv2_CLIENT_CHALLENGE biçimini kullanarak görüntüleyecektir:

- **NTLMv2 Kimlik Doğrulama Yanıtı**:
    
    - NTLMv2, ağ üzerinden kimlik doğrulama yapan bir protokoldür ve kullanıcı doğrulaması için güvenli bir yanıt hesaplaması gereklidir.
- **NTOWFv2 ve LMOWFv2**:
    
    - Bu iki fonksiyon, NTLMv2 için kullanılan **tek yönlü fonksiyonlar**dır. Kullanıcının şifresi, bu fonksiyonlar aracılığıyla bir **hash değeri**ne dönüştürülür.
    - **LMOWFv2**: LM (Lan Manager) protokolüne dayalı bir fonksiyondur, ancak NTLMv2 için de kullanılır.
    - **NTOWFv2**: NTLM (New Technology LAN Manager) protokolüne dayalı bir fonksiyondur ve NTLMv2 için daha güvenli bir hash hesaplama yöntemidir.
- **Yanıt Hesaplama (computeResponseNTLMv2)**:
    
    - İstemci, yukarıdaki fonksiyonları kullanarak sunucuya yanıtını hesaplar. Bu yanıt, kimlik doğrulama sürecinde önemli bir rol oynar ve güvenliği sağlamak için belirli algoritmalarla hesaplanır.
    - Hesaplama işleminde kullanılan **ServerChallenge** ve **NTLMv2_CLIENT_CHALLENGE** gibi öğeler, güvenliği artırmak için rastgele değerler içerir.
- **Responder ile Yakalanan NTLMv2 Hash Formatı**:
    
    - **Responder**, NTLMv2 hash’lerini yakalamak için yaygın olarak kullanılan bir aracımdır. Yakalanan hash şu formatta görüntülenir:
    ![Pasted image 20241205041028.png](/img/user/resimler/Pasted%20image%2020241205041028.png)
    * Bu format, **kullanıcı adı** (User), **alan adı** (Domain), **sunucu isteği** (ServerChallenge), **yanıt** (Response) ve **istemci isteği** (NTLMv2_CLIENT_CHALLENGE) gibi öğeleri içerir.



![Pasted image 20241205041052.png](/img/user/resimler/Pasted%20image%2020241205041052.png)



### NTLM Session Security

Client ve server anlaşırsa ve mesaj gizliliği ([oturum güvenliği ](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/d1c86e81-eb66-47fd-8a6f-970050121347)mühürleme sağlar). [Mesaj bütünlüğü (imzalama)](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/131b0062-7958-460e-bca5-c7a9f9086652) NTLM protokolünün kendisi oturum güvenliği sağlamaz; bunun yerine [SSPI](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/0776e9c8-1d92-488f-9219-10765d11c6b7) sağlar. mühürleme, ancak yalnızca NTLMv2 tarafından desteklenen NTLMv1 imzalamayı desteklemez; bu nedenle [Microsoft](https://support.microsoft.com/en-au/topic/security-guidance-for-ntlmv1-and-lm-network-authentication-da2168b6-4a31-0088-fb03-f081acde6e73), kullanımına karşı şiddetle tavsiye eder (ve kullanımdan kaldırılan LM kimlik doğrulama protokolü de).

- **İstemci ve Sunucu Müzakeresi**:
    
    - İstemci ve sunucu arasında güvenli iletişim kurabilmek için bazı parametreler (örneğin, mesaj gizliliği ve bütünlüğü) müzakere edilebilir. Bu müzakereler, oturum güvenliğinin sağlanmasında temel bir rol oynar.
- **Mesaj Gizliliği (Sealing)**:
    
    - **Sealing** (mühürleme), mesajların gizliliğini sağlamak için kullanılan bir mekanizmadır. Oturum güvenliği, bu mühürleme işlemi ile sağlanır, yani mesajlar şifrelenerek yalnızca yetkili taraflarca okunabilir.
- **Mesaj Bütünlüğü (Signing)**:
    
    - **Signing** (imzalama), mesajın bütünlüğünü sağlamak için kullanılır. Bu, mesajın değiştirilmediğini doğrulayan bir süreçtir. Ancak NTLM protokolü kendisi imzalama sağlamaz; bunun yerine **SSPI** (Security Support Provider Interface) tarafından sağlanır. SSPI, Microsoft'un güvenlik desteği sağlayan bir arabirimidir.
- **NTLMv1 ve NTLMv2**:
    
    - **NTLMv1**, eski bir güvenlik protokolüdür ve **imza** fonksiyonunu desteklemez. Microsoft, NTLMv1'in kullanımını şiddetle tavsiye etmemektedir, çünkü daha güvenli bir seçenek olan **NTLMv2** mevcut olup, eski protokolün yerini almıştır.
- **LM Protokolü**:
    
    - **LM (Lan Manager)** kimlik doğrulama protokolü, çok eski bir protokoldür ve modern güvenlik standartlarına göre zayıf kabul edilir. Microsoft, bu protokolün kullanımını da önermez çünkü daha güvenli protokoller, örneğin NTLMv2, mevcuttur.
-



### Message Signing and Sealing (Mesaj İmzalama ve Mühürleme)

Mesaj imzalama mesaj bütünlüğü sağlar ve relay saldırılarına karşı yardımcı olur; NTLM iletişimleri sırasında client ve server arasında gönderilen mesajların güvenliğini artırmak için tasarlanmış kritik bir güvenlik özelliğidir. Session imzalama üzerinde anlaşmaya varıldığında, client ve server değiş tokuş edilen tüm mesajları imzalamak için bir session key üzerinde anlaşmaya varırlar. Session anahtarı, client ve server'ın meydan okuma mesajları ve kullanıcının parola hash'inin bir kombinasyonu kullanılarak oluşturulur. Session anahtarı oluşturulduktan sonra, client ve server arasındaki tüm mesajlar bir MAC kullanılarak imzalanır. hide01.ir MAC, mesaja ve session anahtarına bir kriptografik algoritma uygulanarak oluşturulur. Server, mesaj ve oturum anahtarı ile aynı algoritmayı kullanarak ve sonucu istemci tarafından sağlanan MAC ile karşılaştırarak MAC'i doğrulayabilir. Bir düşman gizlice dinliyor olsa da, kullanıcının parola hash'ine sahip değildir, çünkü bu hiçbir zaman kablo üzerinden iletilmez ve bu nedenle mesajları imzalayamaz. SMB Signing'in Temelleri blog yazısına dayanarak (hem SMB1 hem de SMB2'yi kapsar), çalıştırdıkları SMB sürümüne bağlı olarak ağdaki hostlar için varsayılan SMB imzalama ayarlarını öğrenebiliriz. 

Required (Gerekli), Enabled (Etkin) veya Disabled (Devre Dışı), SMB2 ve SMB3'te yalnızca Required (Gerekli) veya Not Required (Gerekli Değil) vardır

![Pasted image 20241205042410.png](/img/user/resimler/Pasted%20image%2020241205042410.png)

- **Mesaj İmzalama Nedir?**
    
    - Mesaj imzalama, istemci (client) ve sunucu (server) arasında gönderilen mesajların **bütünlüğünü** sağlar.
    - Aynı zamanda **relay saldırılarına karşı** koruma sağlar. (Relay saldırısı, bir saldırganın iki taraf arasındaki iletişimi ele geçirip yeniden iletmesi anlamına gelir.)
- **NTLM ve Güvenlik Özelliği**:
    
    - NTLM iletişiminde, mesajların imzalanması **kritik bir güvenlik özelliği** olarak devreye girer.
    - Bu, özellikle ağdaki iletişimin güvenliğini artırmak için tasarlanmıştır.
- **Session İmzalama (Session Signing)**:
    
    - **Session signing** (oturum imzalama) üzerinde anlaşma sağlandığında, istemci ve sunucu bir **session key** üzerinde uzlaşır.
    - Bu oturum anahtarı, iletişimin her iki tarafı için de benzersizdir ve tüm mesajların imzalanmasında kullanılır.
- **Session Anahtarı Nasıl Oluşturulur?**
    
    - Session key şu kombinasyonlar kullanılarak üretilir:
        - İstemcinin (client) meydan okuma mesajı (**challenge**),
        - Sunucunun (server) meydan okuma mesajı (**challenge**),
        - Kullanıcının **parola hash’i**.
    - Bu birleşim, güvenli bir oturum anahtarı oluşturur.
- **Mesajların İmzalanması (MAC Kullanımı)**:
    
    - **MAC (Message Authentication Code)**, oturum anahtarı ve mesaj üzerinde bir **kriptografik algoritma** uygulanarak oluşturulur.
    - İmzalanan her mesaj, MAC ile birlikte gönderilir.
    - Sunucu (server), aynı algoritmayı kullanarak MAC’i doğrular ve mesajın bütünlüğünden emin olur.
- **Saldırgan Neden Mesajları İmzalayamaz?**
    
    - Saldırgan iletişimi dinliyor olsa bile, istemcinin **parola hash’ine** sahip değildir.
    - Parola hash’i hiçbir zaman ağ üzerinden gönderilmediği için saldırgan, MAC’i üretemez ve mesajları geçerli şekilde imzalayamaz.
- **SMB Signing Ayarları (Varsayılanlar)**:
    
    - **SMB Signing** (SMB imzalama), NTLM iletişiminde güvenlik sağlar.
    - SMB'nin hangi sürümünün çalıştığına bağlı olarak farklı imzalama ayarları kullanılabilir:
        - **SMB1**: Üç seçenek vardır:
            - **Required (Gerekli)**: İmzalama zorunludur.
            - **Enabled (Etkin)**: İmzalama desteklenir ama zorunlu değildir.
            - **Disabled (Devre Dışı)**: İmzalama kullanılmaz.
        - **SMB2 ve SMB3**: İki seçenek vardır:
            - **Required (Gerekli)**: İmzalama zorunludur.
            - **Not Required (Gerekli Değil)**: İmzalama opsiyoneldir.
- **Sonuç**:
    
    - Mesaj imzalama, iletişim güvenliğini artırır ve relay saldırılarına karşı koruma sağlar.
    - SMB sürümüne göre imzalama ayarları değişiklik gösterebilir. **SMB2 ve SMB3**, daha modern ve güvenli ayarlara sahiptir.


Varsayılan SMB imzalama ayarlarının düşmanlar tarafından sürekli kötüye kullanılması nedeniyle Microsoft, Windows 11 Insider sürümlerinde (ve daha sonra ana sürümlerde) SMB imzalamayı zorunlu kılmak için bir güncelleme yayınlamadı. Microsoft, daha iyi bir güvenlik yapısı için, Windows 10 ve 11'in varsayılan olarak yalnızca SYSVOL ve NETLOGON adlı paylaşımlara bağlanırken SMB imzalamayı gerektirdiği ve DC'lerin herhangi bir istemci onlara bağlandığında SMB imzalamayı gerektirdiği eski davranışı nihayet bırakmaya karar verdi. Microsoft'ta Baş Program Yöneticisi olan Ned Pyle, Windows Insider'da varsayılan olarak SMB imzalamayı zorunlu kılan yazısında aşağıdaki üzüntü verici [açıklamayı](https://techcommunity.microsoft.com/t5/storage-at-microsoft/smb-signing-required-by-default-in-windows-insider/ba-p/3831704) yazdı:

“SMB şifrelemesi imzalamadan çok daha güvenli, ancak ortamlar hala SMB 3.0 ve sonrasını desteklemeyen eski sistemleri çalıştırıyor. Eğer 1990'lara zaman yolculuğu yapabilseydim, SMB imzalama her zaman açık olurdu ve SMB şifrelemeyi çok daha erken kullanıma sunardık; ne yazık ki hem lisedeydim hem de yetkili değildim. Önümüzdeki yıllarda daha güvenli SMB varsayılanlarını ve birçok yeni SMB güvenlik seçeneğini sunmaya devam edeceğiz; bunların uygulama uyumluluğu için acı verici olabileceğini ve Windows'un kullanım kolaylığı sağlama mirasına sahip olduğunu biliyorum, ancak güvenlik şansa bırakılamaz.”

Mesaj mühürleme, simetrik anahtarlı bir şifreleme mekanizması uygulayarak mesaj gizliliği sağlar; client ve server arasında değiş tokuş edilen mesajların içeriğinin güvende kalmasını ve düşmanların bunları okuyamamasını veya kurcalamamasını sağlar. NTLM bağlamında, mühürleme aynı zamanda imzalama anlamına da gelir çünkü her biri aynı zamanda imzalanmıştır .



Microsoft, SMB imzalama ayarlarını güvenlik açısından daha katı hale getirerek, Windows Insider sürümlerinde ve gelecekteki ana sürümlerde **SMB imzalamayı zorunlu** hale getirmek için bir güncelleme yayınlamadı.

Öncesinde:

- **Windows 10 ve 11**, yalnızca **SYSVOL** ve **NETLOGON** paylaşımlarında SMB imzalamayı varsayılan olarak zorunlu kılıyordu.
- **Domain Controllers (DC'ler)**, bağlanan tüm istemcilerden SMB imzalama talep ediyordu.

Sonrasında:

- Varsayılan olarak **SMB imzalama tüm bağlantılarda zorunlu** hale geldi, bu da güvenlik açıklarını daha fazla sınırlamayı amaçlıyor.

### Ned Pyle’ın Yorumu:

- **SMB Şifreleme**, imzalamadan daha güvenli bir yöntemdir. Ancak eski sistemler, SMB 3.0 ve üzerini desteklemediği için günümüzde bile kullanılmaktadır.
- Microsoft, güvenlik öncelikli bir yaklaşım benimseyerek gelecekte daha güvenli SMB varsayılanlarını ve seçeneklerini sunmaya devam edecektir.

### Mesaj Mühürleme:

- Mesaj mühürleme, istemci ve sunucu arasında paylaşılan mesajların içeriğini **simetrik anahtarlı şifreleme** ile korur.
- Mühürleme, hem mesajların **gizliliğini sağlar** hem de mesajların **imzalanmasını** kapsar.
- Böylece düşmanlar, mesajları okuyamaz veya değiştiremez.

**Özet:**  
Microsoft, SMB protokolünün güvenliğini artırmaya yönelik adımlar atıyor ve gelecekteki güncellemelerde daha modern ve zorunlu güvenlik mekanizmalarını devreye sokmayı hedefliyor.



### Extended Protection for Authentication (Kimlik Doğrulama için Genişletilmiş Koruma) (EPA)

hide01.ir [RFC 5056'](https://datatracker.ietf.org/doc/html/rfc5056)yı temel alan Genişletilmiş Kimlik Doğrulama Koruması ( EPA ), Windows Server 2008 ve sonraki sürümlerinde sunulan ve kimlik doğrulama güvenliğini artıran bir özelliktir. NTLM EPA etkinleştirildiğinde, istemci ve sunucu bir channel binding token ( CBT ) kullanarak güvenli bir kanal kurar. CBT, kimlik doğrulamasını IP adresi ve port gibi belirli kanal özelliklerine bağlayarak kimlik doğrulamasının farklı bir kanalda tekrarlanmasını önler. EPA, SMB ve HTTP protokolleriyle çalışmak üzere tasarlanmıştır ve NTLM kimlik doğrulamasına dayanan uygulamalar ve hizmetler için ek güvenlik sağlar; ancak güvenli bir kanal oluşturmak için istemci ve sunucunun bunu desteklemesini gerektirir.

NTLM'nin temellerini kısaca anladıktan sonra, bir sonraki bölüm NTLM aktarma saldırısını kapsamlı bir şekilde inceleyecek, farklı aşamalarını keşfedecek ve her aşamada kullanabileceğimiz araçlar ve teknikler hakkında bilgi edinecektir.

- **NTLM tekrar saldırılarına karşı koruma sağlamak için ne kullanır?**
- **Sunucu NetrLogonSamLogonWithFlags'i çağırdığında gönderilen veri yapısının adı nedir?**
- **İstemcinin bir sunucuya kimlik doğrulama yapmak istediğini ve NTLM seçeneklerini belirtmek istediğini gösteren NTLM mesajının adı nedir?**
- **Mesaj imzalama bütünlük sağlıyorsa, mesaj mühürleme ne sağlar?**
- **SMB2 istemcileri ve sunucuları için varsayılan imzalama ayarı nedir?**


### NTLM Relay Attack

Kimlik doğrulama protokollerinin başlangıcından bu yana, güvenlik araştırmacıları açıklarını ve zayıflıklarını keşfetmek için bu protokollere yönelik saldırılar üzerinde çalışmaktadır; örneğin 1990'ların başında [Bird, R. ve ark.](https://link.springer.com/chapter/10.1007/3-540-46766-1_3) 'X'in (“intruder”) 'A' (“client”) kılığına girerek 'B' (“server”) ile bir oturum başlattığı, 'B'nin gönderdiği şifreli metnin şifresini çözmek için 'A'yı bir oracle olarak kullandığı ve ardından 'A' olarak 'B' üzerinde kimliği doğrulanmış bir oturum kurduğu, yazarlar tarafından aşağıdaki sekans diyagramında tasvir edilen “Oracle Session” saldırısını ele almıştır:

![Pasted image 20241205162128.png](/img/user/resimler/Pasted%20image%2020241205162128.png)

![Pasted image 20241205161948.png](/img/user/resimler/Pasted%20image%2020241205161948.png)

Oracle Session saldırısına benzer şekilde, NTLM relay (“NTLM reflection” olarak da anılır), bir ağdaki kaynaklara yetkisiz erişim elde etmek için birçok güvenlik açığı ve saldırı tarafından kötüye kullanılan kötü şöhretli bir [Adversary-in-the-Middle](https://attack.mitre.org/techniques/T1557/) ( AiTM ) (Man-in-the-Middle ( MitM ) veya Bucket Brigade Attack olarak da bilinir) tekniğidir.

2001 yılında Cult of the Dead Cow'un bir üyesi olan Sir Dystic, Oracle Session saldırısının konseptini çok yakından takip eden SMB sunucularına karşı NTLM self-relay saldırısı için bir [C++](https://packetstormsecurity.com/files/30622/smbrelay.cpp.html) proof of concept olan [SMBRelay](https://packetstormsecurity.com/files/30622/smbrelay.cpp.html)'i (ve daha sonra SMBRelay2'yi) yayınladı. Microsoft, 2008 yılında [MS08-068](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2008/ms08-068) güncellemesiyle NTLM kendi kendine relay saldırısını yamalamıştır.

NTLM kimlik doğrulama protokolleri ailesini etkileyen güvenlik açıkları ve zayıflıklar, doğal tasarım sorunlarından kriptografik olarak [güvensiz algoritmaların kullanımına](https://learn.microsoft.com/en-us/sql/connect/jdbc/using-ntlm-authentication-to-connect-to-sql-server?view=sql-server-ver16#security-risks) kadar birçok şeydir. Kerberos'un daha iyi ve daha güvenli bir alternatif olması nedeniyle [Microsoft](https://learn.microsoft.com/en-us/windows/security/threat-protection/security-policy-settings/network-security-restrict-ntlm-incoming-ntlm-traffic#security-considerations) tarafından kullanımı pek hoş karşılanmamaktadır. Bununla birlikte, [Kerberos](https://learn.microsoft.com/en-us/windows-server/security/kerberos/kerberos-authentication-overview)'un [kullanılamayacağı bazı durumlar](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/a4f28e01-3df1-4fd1-80b2-df1fbc183f21) vardır:

1. Eski sistemlerle uyumluluk (Kerberos'u desteklemeyen eski sistemler gibi).
2. Kerberos yapılandırması doğru şekilde ayarlanmamıştır.
3. Sunucu domain-joined değildir.
4. Bir protokol Kerberos'u değil, yalnızca NLMP'yi desteklemeyi seçer.

NTLMv2'nin NTLMv1 ve LM'ye göre aldığı birçok güvenlik iyileştirmesine bakılmaksızın, NTLM kimlik doğrulama protokolleri ailesinin tümü, istemcinin gerçek sunucuya karşı kimlik doğrulaması yaptığını ve sunucunun da gerçek istemciye karşı kimlik doğrulaması yaptığını doğruladığı karşılıklı kimlik doğrulamayı destekleme araçlarından yoksun oldukları için sahte sunuculardan gelen [relay saldırılarına karşı hassastır](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-nlmp/505a64e4-089c-47ec-bad8-32b9c2cd8da9) (SSPI'ın sağladığı oturum güvenliğinin imzalama kullanarak bu sorunu çözdüğünü unutmayın).


### AiTM NTLM Authentication
Domain-joined makinelere karşı NTLM relay saldırısını gerçekleştirmek için, bir düşman, kimlik doğrulama talebinde bulunan client için gerçek bir sunucu gibi davranır, ayrıca bir servis sunan server için de gerçek bir client gibi davranarak, kimliği doğrulanmış bir oturum kurana kadar aralarında mesajlar iletir. Sunucu ile kimliği doğrulanmış bir oturum kurduktan sonra, saldırgan client adına yetkili eylemler gerçekleştirmek için bunu kötüye kullanır; client için saldırgan ya kimlik doğrulamanın başarısız olduğunu belirten bir uygulama mesajı gönderir ya da bağlantıyı sonlandırır:

![Pasted image 20241205162926.png](/img/user/resimler/Pasted%20image%2020241205162926.png)
### Özet:

1. **İstemci (Client)**, kimlik doğrulama sürecini başlatır ve sunucuya **NTLM NEGOTIATE_MESSAGE** mesajını gönderir.
2. **Adversary (Saldırgan)**, bu mesajı alır ve sunucuya iletir.
3. Sunucu, **NTLM CHALLENGE_MESSAGE** mesajını oluşturur ve saldırgana gönderir.
4. **Saldırgan**, bu mesajı istemciye ileterek istemcinin **NTLM AUTHENTICATE_MESSAGE** mesajını oluşturmasını sağlar.
5. İstemcinin oluşturduğu **NTLM AUTHENTICATE_MESSAGE**, saldırgan tarafından alınıp sunucuya gönderilir.
6. Sunucu, kimlik doğrulama için **NETLOGON_NETWORK_INFO** isteği yapar ve doğrulama sürecini tamamlar.
7. Kimlik doğrulama tamamlandıktan sonra bir **Authenticated Session** başlatılır.
8. Bağlantı, **Terminate Connection** mesajıyla sonlandırılır.

### Önemli Noktalar:

- **Saldırganın rolü:** İstemci ve sunucu arasındaki mesajları geçirerek kimlik doğrulama oturumunu manipüle eder.
- **Hedef:** Kimlik bilgilerini ele geçirmek veya oturumu yetkisiz bir şekilde kullanmak.

Workgroup NTLM kimlik doğrulaması için saldırgan aynı saldırıyı gerçekleştirir:

![Pasted image 20241205163224.png](/img/user/resimler/Pasted%20image%2020241205163224.png)
Not: Kurumsal ortamlar her zaman Active Directory'ye dayanır ve bu nedenle host'lar domain-joined kimlik doğrulamasını kullanır. Gelecek görüntülerde basitlik ve anlaşılırlık adına DC gösterilmese de, sunucunun istemcinin kimliğinin doğrulanmasını bir DC'ye devrettiğini her zaman unutmayın.


### NTLM Relay Saldırı Aşamaları
NTLM relay saldırısını üç aşamaya bölebiliriz: Pre-relay, Relay ve Postrelay. Her bir aşamayı keşfedelim ve kullanabileceğimiz teknikler ve araçlar hakkında bilgi edinelim.


### Phase I: Pre-relay
Relay öncesi aşama, bir client'ı bir sunucudaki bir servis için NTLM kimlik doğrulamasını başlatmaya teşvik eden/zorlayan tekniklere odaklanır:
![Pasted image 20241205170441.png](/img/user/resimler/Pasted%20image%2020241205170441.png)

### Aşamalar:

1. **Pre-Relay Phase:**
    
    - **İstemci (Client)**, kimlik doğrulama başlatmak için **NTLM NEGOTIATE_MESSAGE** mesajını oluşturur ve göndermek üzere hazırlık yapar.
    - **Saldırgan (Adversary)**, bu mesajı ele geçirir ve **Sunucuya (Server)** iletir.
2. **NEGOTIATE_MESSAGE:**
    
    - İstemci, kimlik doğrulama sürecini başlatmak için **NTLM NEGOTIATE_MESSAGE** gönderir.
    - **Saldırgan**, bu mesajı sunucuya göndererek istemci adına iletişimi başlatır.
3. **CHALLENGE_MESSAGE:**
    
    - **Sunucu**, istemciye bir doğrulama meydan okuması göndermek için **NTLM CHALLENGE_MESSAGE** üretir.
    - Bu mesaj, saldırgana iletilir ve saldırgan tarafından istemciye aktarılır.
4. **AUTHENTICATE_MESSAGE:**
    
    - İstemci, kimlik bilgilerini kullanarak **NTLM AUTHENTICATE_MESSAGE** oluşturur ve gönderir.
    - **Saldırgan**, bu mesajı ele geçirir ve sunucuya iletir.
5. **Authenticated Session:**
    
    - Sunucu, **AUTHENTICATE_MESSAGE**'ı doğrular ve bir **Authenticated Session (Doğrulanmış Oturum)** oluşturur.
    - Bu aşamada saldırgan, istemci adına yetkisiz erişim sağlamış olur.
6. **Bağlantı Kapatma (Terminate Connection):**
    
    - Kimlik doğrulama süreci sona erer ve oturum saldırgan tarafından manipüle edilmiş şekilde kapanır.

### Önemli Noktalar:

- **Saldırganın Rolü:** Ortadaki adam (Man-in-the-Middle) pozisyonunda hareket ederek istemci ile sunucu arasındaki trafiği manipüle eder.
- **Hedef:** İstemcinin kimlik bilgilerini ele geçirmek veya istemci adına sunucuda yetkisiz işlemler yapmak.

Bu diyagram, saldırganın istemci ve sunucu arasındaki güveni nasıl kötüye kullandığını detaylı bir şekilde gösteriyor. Özellikle, NTLM Relay saldırılarında sıkça kullanılan bir yöntemdir.

Relay öncesi aşamada aşağıdakiler de dahil olmak üzere birçok teknik kullanılır:
* poisoning ve spoofing saldırıları gibi AiTM teknikleri.
* Kimlik Doğrulama Zorlama saldırıları.

Relay öncesi aşama odaklanır Bu bölüm zehirleme ve sahtekarlık saldırılarını kapsayacaktır. Kimlik Doğrulama Zorlamasını başka bir bölümde tartışacağız.


### Poisoning and Spoofing

Gerçek bir ağ sunucusu gibi davranmak veya trafiği yönlendirmek için poisoning veya spoofing işlemleri gerçekleştirmenin birçok yolu vardır. Bu teknikler genellikle çok fazla gürültü çıkarır ve fırsatçıdır çünkü çoğu istemciler tarafından başlatılan isteklere ve trafiğe dayanır.

Windows çeşitli name resolution işlemlerini destekler:

![Pasted image 20241205170750.png](/img/user/resimler/Pasted%20image%2020241205170750.png)


### LLMNR, NBT-NS, and mDNS Poisoning
Kurumsal ağlarda DNS kullanımı genellikle yoğundur. Ancak, bazı kuruluşlar DNS'yi mükemmel bir şekilde yapılandırmaz ve bu da bazı clientlerin standart DNS sunucularının çözümleyemediği name resolution'ı kullanmasına neden olur (örneğin, eksik DNS tabloları veya kullanılamayan sunucular nedeniyle). Sistem yöneticileri, bu boşluğu doldurmak ve daha akıcı ağlara sahip olmak için istemcileri bu durumlarda alternatif ad çözümleme protokolleri kullanacak şekilde yapılandırabilir. Örneğin, varsayılan olarak Windows makineleri [RFC4795](https://datatracker.ietf.org/doc/html/rfc4795)'te belirtilen LLMNR (Local-Link Multicast Name Resolution), NBT-NS (NetBIOS Name Service) ve diğer işletim sistemleri de mDNS (multicast Domain Name System) kullanacak şekilde yapılandırılmıştır.

DNS unicast bir protokoldür, yani bir client doğrudan bir sunucudan name resolution isterken, LLMNR , NBT-NS ve mDNS multicast protokollerdir, yani client aradığı ismi bilen var mı diye ağa yayın yapar. Dolayısıyla, bu ağdaysak, rastgele name resolution isteklerine cevap verebilir ve istemciyi saldırı hostumuza yönlendirebiliriz.

Poofing ve spoofing için kullanabileceğimiz en yaygın araçlar Responder (en yaygın olarak Linux'ta kullanılır) ve Inveigh'dir (en yaygın olarak Windows'ta kullanılır). Responder, NTLMv1/NTLMv2/LMv2, Extended Security NTLMSSP ve Basic HTTP kimlik doğrulamasını destekleyen yerleşik HTTP/SMB/MSSQL/FTP/LDAP sahte kimlik doğrulama sunucusuna sahip bir LLMNR , NBT-NS ve MDNS zehirleyicisidir.

Aşağıdaki diyagramda, bir client SERVER'a bağlanmak yerine SERVERR olarak yanlış yazmaktadır; ancak, SERVERR için herhangi bir DNS kaydı bulunmadığından, Windows varsayılan olarak farklı protokoller (bu durumda LLMNR) aracılığıyla ağa yayın mesajları göndererek SERVERR'ın IP adresini bilen olup olmadığını sormaktadır. Saldırganlar olarak, bu yayın isteklerini kötüye kullanacağız ve istemciyi bize karşı kimlik doğrulamaya zorlamak için Responder kullanarak zehirli yanıtlar göndereceğiz.

![Pasted image 20241205171336.png](/img/user/resimler/Pasted%20image%2020241205171336.png)

Yukarıdaki LLMNR , NBT-NS veya mDNS zehirlenmesi durumunun bir örneğini oluşturalım. Bu modülde, Ubuntu bilgisayarı aracılığıyla bağlanacağımız dahili ağa erişimimiz olduğunu taklit edeceğiz. Bu bilgisayar, ağ ile etkileşime geçmek ve saldırıları gerçekleştirmek için ihtiyaç duyduğumuz tüm araçlara sahiptir.

Laboratuvarı (aşağıdaki hedef makine) başlatmamız ve saldırı hostumuza bağlanmak için SSH veya RDP ile bağlanmamız gerekiyor. Tüm alıştırmalar için htbstudent:HTB_@cademy_stdnt! kimlik bilgilerini kullanarak SSH kullanacağız:

![Pasted image 20241205172028.png](/img/user/resimler/Pasted%20image%2020241205172028.png)

Artık saldırı hostumuza erişebildiğimize göre, Responder'ı çalıştıracağız; HTTP(ler), SMB, Kerberos gibi farklı sunucular oluşturmanın yanı sıra kimlik doğrulama isteklerini ele almak için varsayılan olarak LLMNR, NBT-NS ve MDNS yayın isteklerini zehirlemeye başlayacaktır. Yakalanan tüm hashler ekrana (stdout) yazdırılır ve ayrıca /logs dizinine (MODULE_NAME)-(HASH_TYPE)-(CLIENT_IP).txt gibi bağlamsal bir şekilde kaydedilir.

Responder, analiz modunu ( -A ) kullanarak ağda trafik olup olmadığını analiz etmemize olanak tanır; bu mod, keşif amacıyla kullanabileceğimiz herhangi bir NBT-NS , tarayıcı veya LLMNR isteğini yanıt vermeden görmemizi sağlayacaktır. İnsanlar host adlarını, URL'leri ve benzerlerini yanlış yazma eğilimindedir, bu kimlik doğrulama isteklerini iletmek için kendi lehimize kullanabiliriz. Responder'ı kullanmak için -I seçeneği ile istediğimiz arayüzü belirtmemiz gerekir; bizim durumumuzda iç ağa bağlı arayüz ens192'dir. Eğer trafiği analiz etmek istiyorsak -A seçeneğini kullanabiliriz, bu da trafiği zehirlemeyeceğimiz anlamına gelir:


### Responder'ı Analiz Modunda Çalıştırma
![Pasted image 20241205172419.png](/img/user/resimler/Pasted%20image%2020241205172419.png)
![Pasted image 20241205172425.png](/img/user/resimler/Pasted%20image%2020241205172425.png)
![Pasted image 20241205172433.png](/img/user/resimler/Pasted%20image%2020241205172433.png)

Yukarıdaki örnekte, 172.16.117.3 bilgisayarından bir kullanıcı ws001'e bağlanmaya çalıştı, ancak DNS yanıt veremediği için diğer protokoller üzerinden yayınladı ve Responder bu istekleri görüntüledi.

Yayın trafiğini analiz etmek yerine zehirlemek için -A seçeneğini kaldırabiliriz; Responder yayın trafiğini yanıtlayacak ve yanıtları zehirleyerek kullanıcıların saldırı makinemize karşı kimlik doğrulama girişiminde bulunmasını sağlayacaktır:


### Responder'ı Poisoning Modunda Çalıştırma

![Pasted image 20241205172926.png](/img/user/resimler/Pasted%20image%2020241205172926.png)

Aşağıda, kullanıcıların yayın mesajlarına neden olan eylemleri gösterilmektedir:

![Pasted image 20241205172951.png](/img/user/resimler/Pasted%20image%2020241205172951.png)

Bir kullanıcı bir UNC'yi yanlış yazdığında SMB bağlantısı almamıza benzer şekilde, bir kullanıcı var olmayan bir domaine göz attığında HTTP ile aynı sonuçları alabiliriz; bağlantının makinemize gelmesi için Responder kullanırız:

![Pasted image 20241205173026.png](/img/user/resimler/Pasted%20image%2020241205173026.png)

Not: Analiz modu, SMB, MSSQL, HTTP, FTP, IMAP ve LDAP gibi farklı protokollerin trafiğini/hash'lerini yakalamak istediğimizde de kullanışlıdır. Bu hash'leri kırabilir veya düz metin kimlik bilgilerini alacak kadar şanslı olabiliriz. Ancak, bu modülde sadece relaying saldırılarına odaklanacağız

Responder'a ek olarak [Pretender](https://github.com/RedTeamPentesting/pretender), [RedTeam Pentesting](https://blog.redteam-pentesting.de/2022/introducing-pretender/) tarafından geliştirilen Go ile yazılmış yeni ve güçlü bir araçtır. Name resolutions ve DHCPv6 DNS ile ilgili takeover saldırılarını taklit ederek man-in-the-middle pozisyonu elde eder. Pretender, ortamdaki yayın trafiğini analiz etmeye yardımcı olan --dry bayrağı kullanılarak herhangi bir name resolution'a cevap vermeden çalıştırılabilir. Pretender kullanıma hazır sürümler sunar. Modül boyunca sadece Responder'ı kullanacağız. Bununla birlikte, diğer araçlar hakkında bilgi sahibi olmak, özellikle de Go ile yazılmış olanlar, platformlar arası ve taşınabilir oldukları için her zaman faydalıdır. Varsayılan olarak DHCP isteklerini zehirlediğinden, [pretender](https://github.com/RedTeamPentesting/pretender/releases/tag/v1.1.1) üretim ağlarında oldukça yıkıcı olabilir.

Not: DHCP isteklerini zehirlemenin sorunu, istek süresi dolana, makine yeniden başlatılana veya yeni bir DHCP isteği zorlanana kadar makinede yarı kalıcı olmalarıdır. Bu gerçekleşirken, bu makine tarafından yapılan herhangi bir DNS isteği hedefini kaçırabilir ve makineyi ağ sorunlarıyla baş başa bırakabilir

Ayrıca, aşağıdakiler de dahil olmak üzere birçok başka zehirlenme saldırısı da mevcuttur:

https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/arp-poisoning
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dns-spoofing
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dhcp-poisoning
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/dhcpv6-spoofing
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/wpad-spoofing
https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/wsus-spoofing


### Phase II: Relay
Relay aşaması, client'ın NTLM kimlik doğrulamasını bir relay hedefine aktarmaya odaklanır:

![Pasted image 20241205174628.png](/img/user/resimler/Pasted%20image%2020241205174628.png)

### Sürecin Aşamaları:

1. **İstemci (Client) ile Adversary (Saldırgan) arasındaki etkileşim:**
    
    - İstemci, sunucuya bağlanmak için bir **NTLM NEGOTIATE_MESSAGE** gönderiyor.
    - Bu mesaj, saldırgan tarafından yakalanıp sunucuya iletiliyor.
2. **Saldırgan ile Sunucu arasındaki etkileşim:**
    
    - Sunucu, gelen NTLM NEGOTIATE_MESSAGE'e yanıt olarak bir **NTLM CHALLENGE_MESSAGE** gönderiyor.
3. **Adversary'nin bu mesajları istemciye yönlendirmesi:**
    
    - Saldırgan, bu mesajı istemciye geri iletiyor. İstemci, gelen CHALLENGE_MESSAGE'e bir **NTLM AUTHENTICATE_MESSAGE** ile yanıt veriyor.
4. **NTLM AUTHENTICATE_MESSAGE’in sunucuya iletilmesi:**
    
    - Saldırgan bu AUTHENTICATE_MESSAGE’i sunucuya gönderiyor. Bu sayede sunucu, saldırganın kimlik doğrulamasını gerçekleştirilmiş gibi görüyor.
5. **Bağlantının sona ermesi:**
    
    - İstemci, bağlantıyı sonlandırıyor. Ancak bu noktada saldırgan, kimliği doğrulanmış bir oturum başlatmayı başarmış oluyor.

---

### **Önemli Noktalar:**

- **Relay Phase:** Saldırgan, istemci ile sunucu arasındaki tüm kimlik doğrulama mesajlarını doğrudan ilettiği bu aşamada devreye giriyor.
- **Authenticated Session:** Saldırgan, bu süreç sonunda sunucuda yetkilendirilmiş bir oturum başlatabiliyor.

Bu saldırı, özellikle **SMB (Server Message Block)** ve diğer NTLM tabanlı protokoller için tehdit oluşturur. SMB Signing gibi koruma yöntemleriyle bu saldırılar önlenebilir.




### Relay Hedefleri için Avcılık

Bazı kriterleri karşılayan makineler bulmalıyız: relay hedeflerinde SMB'yi hedefliyorsak, SMB imzalamasının devre dışı bırakılması gerekir (diğer protokolleri ilerideki bir bölümde inceleyeceğiz).

SMB imzalama ayarlarını listelemek için RunFinger.py , CrackMapExec ve Nmap dahil olmak üzere birden fazla araç kullanabiliriz.


### RunFinger
RunFinger.py, Responder ile birlikte gelen bir araçtır (/home/htbstudents/tools/Responder/tools/ adresinde bulunur) ve SMB imzası kapalı olan hostlar için ağı numaralandırmanın yanı sıra bazı standart servislerin hostlarda çalışıp çalışmadığını bulmak için de kullanılabilir:

![Pasted image 20241205174856.png](/img/user/resimler/Pasted%20image%2020241205174856.png)


### CrackMapExec

SMB imzalama devre dışı bırakılmış makineleri bulmak için kullanabileceğimiz bir başka araç CrackMapExec'tir; --gen-relay-list FILE_NAME seçeneği, belirli bir alt ağa göre imzalama devre dışı bırakılmış bilgisayarların bir listesini oluşturur ve bunları bir dosyaya kaydeder:

![Pasted image 20241205174929.png](/img/user/resimler/Pasted%20image%2020241205174929.png)


### Nmap

smb2-security-mode.nse komut dosyasıyla nmap'i çalıştırabiliriz; çıktı, imzalamanın etkinleştirildiğini ve yalnızca DC01'de gerekli olduğunu gösterir, bu da domain controller'lar için varsayılan davranıştır. Etkinleştirilmiş olmalarına rağmen, diğer hostlar imzalama gerektirmez :

![Pasted image 20241205175053.png](/img/user/resimler/Pasted%20image%2020241205175053.png)
![Pasted image 20241205175059.png](/img/user/resimler/Pasted%20image%2020241205175059.png)



### Phase III: Post-relay
Post-relay aşaması, bir kurbanın NTLM kimlik doğrulamasını aktararak elde ettiğimiz kimliği doğrulanmış oturumdan yararlanır. Kimliği doğrulanmış oturumun protokolüne bağlı olarak belirli relay sonrası saldırılar gerçekleştirebiliriz.

### ### **Adım Adım Süreç Açıklaması**

#### **1. İstemci ile Adversary (Saldırgan) arasındaki etkileşim:**

- **NTLM NEGOTIATE_MESSAGE:** İstemci, bir oturum başlatmak için ilk kimlik doğrulama mesajını (negotiation) gönderir.
- Bu mesaj, saldırgan tarafından yakalanır ve sunucuya iletilir.

#### **2. Adversary (Saldırgan) ile Sunucu arasındaki etkileşim:**

- **NTLM NEGOTIATE_MESSAGE:** Saldırgan, istemciden aldığı negotiation mesajını sunucuya yönlendirir.
- Sunucu buna yanıt olarak bir **NTLM CHALLENGE_MESSAGE** gönderir.

#### **3. Adversary’nin bu mesajları istemciye iletmesi:**

- Saldırgan, sunucudan aldığı challenge mesajını istemciye iletir.
- İstemci, challenge mesajını işler ve bir **NTLM AUTHENTICATE_MESSAGE** oluşturup saldırgana gönderir.

#### **4. NTLM AUTHENTICATE_MESSAGE’in sunucuya iletilmesi:**

- Saldırgan, istemciden gelen bu kimlik doğrulama mesajını sunucuya iletir.
- Sunucu, bu mesajı kabul ederek saldırganın sunucuda doğrulanmış bir oturum başlatmasını sağlar.

---

### **5. Post-Relay Phase (Aktarma Sonrası Aşama):**

- Saldırgan, doğrulanmış oturumu kötüye kullanarak ek saldırılar gerçekleştirebilir.
- Örnekler:
    - **Yetkili komutların çalıştırılması** (örneğin, uzaktan komut yürütme).
    - **Dosyalara erişim** veya **hassas verilerin çalınması**.
    - **Lateral Movement:** Kimlik bilgilerini veya mevcut bağlantıları kullanarak ağ içinde daha fazla yayılma.

---

### **Anahtar Kavramlar:**

1. **Authenticated Session (Doğrulanmış Oturum):**
    
    - Sunucu, istemcinin kimliğini doğruladığını sanır ve saldırgana tam yetkilendirilmiş erişim sağlar.
2. **Post-Relay Attacks (Aktarma Sonrası Saldırılar):**
    
    - Saldırgan, elde ettiği yetkilerle sistemde kötü niyetli işlemler gerçekleştirir.

---

### **Koruma Yöntemleri:**

- **SMB Signing:** İletişimin dijital olarak imzalanmasını zorunlu kılar, bu da mesajların değiştirilmesini veya yeniden iletilmesini önler.
- **NTLM yerine Kerberos kullanımı:** Daha güvenli bir kimlik doğrulama protokolü olan Kerberos’a geçiş.
- **Seçici kimlik doğrulama:** Kimlik doğrulama sürecini yalnızca güvenilir cihazlarla sınırlamak.


Bu modülde ele alacağımız çok sayıda relay sonrası saldırı vardır. Aşağıdaki kapsamlı zihin haritası @_nwodtuhs tarafından NTLM relay saldırısının üç aşamasını da [göstermektedir](https://www.thehacker.recipes/ad/movement/ntlm/relay):

![Pasted image 20241205175553.png](/img/user/resimler/Pasted%20image%2020241205175553.png)

Zihin haritasını ayrı bileşenlere ayırarak anlamaya çalışalım:

![Pasted image 20241205175709.png](/img/user/resimler/Pasted%20image%2020241205175709.png)

Önemli bir tespit, HTTP NTLM kimlik doğrulamasının, aktarım hedeflerinin açıklara karşı savunmasız olmasını gerektirmeden tüm protokoller üzerinden aktarılabileceğidir. Bununla birlikte, SMB ile karşılaştırıldığında, başarılı relay saldırıları için belirli koşullar gereklidir. Örneğin, aktarım hedefi belirli açıklara karşı savunmasız olmadığı sürece SMB NTLM kimlik doğrulamasını LDAP üzerinden aktarmak imkansızdır. Ancak HTTP NTLM kimlik doğrulaması durumunda bu kısıtlama geçerli değildir. Bu fark, HTTP'nin oturum imzalamayı desteklememesine karşın SMB'nin desteklemesinden kaynaklanır.

Mindmap, CVE-2019-1040 ve CVE-2019-1166'dan postrelay saldırıları yapmak ve sunucu tarafı hafifletmelerini atlamak için bir araç olarak bahsediyor . NTLM oturum güvenliğine karşı üç güvenlik açığını tartışalım .


### Drop the MIC & Drop the MIC 2

11 Haziran 2019 tarihinde, Preempt Security araştırmacıları, Drop the MIC olarak adlandırılan ve uygulama protokolleri tarafından relay'ı önlemek için uygulanan oturum imzalama gereksinimlerini atlamaya izin veren bir güvenlik açığı olan CVE-2019-1040'ı açıkladı. Drop the MIC, NTLM mesajlarını aşağıdaki sırayla manipüle ederek MIC'in atlanmasına izin verir:

* NEGOTIATE_MESSAGE içindeki NTLMSSP_NEGOTIATE_ALWAYS_SIGN ve NTLMSSP_NEGOTIATE_SIGN imzalama bayraklarının ayarını kaldırma.
* AUTHENTICATE_MESSAGE'dan MIC ve Sürüm alanlarının kaldırılması .
* AUTHENTICATE_MESSAGE iletisindeki NTLMSSP_NEGOTIATE_ALWAYS_SIGN , NTLMSSP_NEGOTIATE_SIGN , NEGOTIATE_KEY_EXCHANGE , NEGOTIATE_VERSION bayraklarının ayarının kaldırılması.

Dört ay sonra, Microsoft Drop the MIC için bir güncelleme yayınladıktan sonra, [Preempt Security](https://www.crowdstrike.com/blog/active-directory-ntlm-attack-security-advisory/), Drop the MIC 2 olarak adlandırılan [CVE-2019-1040](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2019-1166)'a benzer bir güvenlik açığı olan CVE-2019-1166'yı ifşa etti.


### Your Session Key is my Session Key

Session Key is my Session Key olarak adlandırılan [CVE-2019-1019](https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2019-1019), [Preempt Security ](https://www.crowdstrike.com/blog/retrieve-session-key-authentication/)araştırmacıları tarafından keşfedilen bir güvenlik açığıdır ve [CVE-2015-0050](https://www.coresecurity.com/core-labs/advisories/windows-pass-through-authentication-methods-improper-validation) (Microsoft'un ele almak için [MS15-027](https://learn.microsoft.com/en-us/security-updates/securitybulletins/2015/ms15-027)'yi yayınladığı) üzerine inşa edilmiştir. İlk bölümde NetrLogonSamLogonWithFlags'e (NETLOGON_NETWORK_INFO içeren) yanıt olarak DC'nin sunucunun kullanması için session key'ini içeren NETLOGON_LOGON_IDENTITY_INFO'yu gönderdiğinden bahsetmiştik; ancak, bir düşman sunucu gibi davranırsa ve daha sonra mesajları imzalamak ve mühürlemek için bir NTLM oturumunun oturum anahtarını elde etmek için aynı çağrıyı yaparsa ne olur? Araştırmacılar, DC'den herhangi bir NTLM kimlik doğrulaması için herhangi bir oturum anahtarı talep etmenin ve herhangi bir sunucuya karşı imzalı bir oturum oluşturmanın mümkün olduğunu keşfetti. Microsoft, [CVE-2019-1019](https://msrc.microsoft.com/update-guide/en-US/vulnerability/CVE-2019-1019) güncelleme kılavuzunda bu güvenlik açığını yamalamıştır.

Hem Drop the MIC hem de Your Session Key is my Session Key, Black Hat USA 2019'daki Finding a Needle in an [Encrypted Haystack](https://www.youtube.com/watch?v=2L7JpcLZ05Q) konuşmasında sunuldu.

NTLM relay saldırısını anladıktan sonra, aşağıdaki bölümde ntlmrelayx kullanılarak yapılan çeşitli SMB post-relay saldırıları gösterilmektedir


### SMB Saldırıları Üzerinden NTLM Relay

NTLM relay saldırısını anladıktan sonra SMB post-relay saldırıları ile başlayacağız.


### Impacket's ntlmrelayx
Relay sunucularını ve istemcilerini başlatmak ve post-relay saldırıları yapmak için birçok araç kullanılabilir; bu modülde Impacket'in ntlmrelayx'ine odaklanacağız. Bu araç Dirk-Jan Mollema tarafından smbrelayx.py'nin bir uzantısı olarak tanıtıldı ve Linux'ta relay saldırıları yürütmek için amiral gemisi aracıdır. MultiRelay.py veya Inveigh (öncelikle Windows'ta kullanılır) gibi diğer araçlar da kullanılabilmesine rağmen, bu modülde onları kullanmayacağız. ntlmrelayx, Impacket içinde NTLM relay ve çeşitli post-relay saldırılarını destekleyen genel bir NTLM relay modülüdür. Farklı saldırı türleri için kullanabileceğimiz çok sayıda seçenek sunar. Modülde ilerledikçe, ntlmrelayx'in sağladığı diğer seçenekleri kullanacağız ve anlayacağız.

![Pasted image 20241205183447.png](/img/user/resimler/Pasted%20image%2020241205183447.png)

SMB post-relay saldırılarını gerçekleştirmeden önce, Responder yapılandırma dosyasını değiştirmemiz gerekir


### Responder Configuration File
Önceki bölümde Responder'ı kullandık ve bazı NTLMv2 hash'lerini yakaladık. Bu mümkündü çünkü Responder varsayılan olarak SMB, HTTP ve diğer bazı protokolleri dinler.

Responder'ın Responder/Responder.conf yapılandırma dosyasını kullanarak varsayılan yapılandırmayı değiştirebiliriz.

Responder'da SMB sunucusunu kapatmak için SMB = On değerini SMB = Off olarak değiştirmeliyiz. Aynı şey diğer protokoller için de geçerlidir. HTTP'yi aktarmak istiyorsak, aktarmaya çalışmadan önce bunu Kapalı olarak ayarlamamız gerekir.

Bunun nedeni, ntlmrelayx'in SMB veya HTTP sunucusunu çalıştırması ve böylece client'den sunucuya bağlantıyı aktarabilmesidir ve Responder bu hizmetleri çalıştırıyorsa, ntlmrelayx'i kullanamayız.


### Setting Responder's SMB Server to Off

![Pasted image 20241205185430.png](/img/user/resimler/Pasted%20image%2020241205185430.png)

Responder yapılandırma dosyasında, yalnızca belirli bir IP veya IP grubundan gelen bağlantıları aktarmak istediğimiz bazı durumlarda yapılandırmak için yararlı olabilecek Yanıt Verilecek Belirli IP Adresleri ( default = All) gibi başka seçenekler de vardır.

Not: Yapılandırma dosyasını incelemek için biraz zaman ayırmanızı öneririz ve bu modülde ilerledikçe, bu yapılandırma dosyasından başka neleri kullanabileceğimizi belirleyebiliriz.


### SMB üzerinden NTLM Relay 

Önceki bölümde, CrackMapExec'in --gen-relay-list komutunu kullanarak SMB imzalama devre dışı bırakılmış aktarma hedeflerini aradık ve 172.16.117.50 ve 172.16.117.60 adreslerini bulduk:

![Pasted image 20241205185550.png](/img/user/resimler/Pasted%20image%2020241205185550.png)

Bu IP'lerden birine bir kimlik doğrulaması iletmek için, Responder kullanarak ağı zehirlememiz ve bir kullanıcı veya bilgisayarın bir broadcast veya araya girebileceğimiz ve bu oturumu saldırı hostumuza kimlik doğrulaması yapmaya zorlayabileceğimiz başka bir aktiviteyi provoke eden bir eylem gerçekleştirmesini beklememiz gerekir.

Ağı zehirlemek için Responder'ı kullanalım:


### Responder'ı Poisoning Modunda Çalıştırma

![Pasted image 20241205185652.png](/img/user/resimler/Pasted%20image%2020241205185652.png)

Daha sonra, NTLM Relay saldırısını gerçekleştirmek için ntlmrelayx kullanmamız gerekir. ntlmrelayx, relay hedeflerini belirtmek için -t ve -tf seçeneklerini sağlar; -t tek bir relay hedefini belirtirken, -tf birden fazla relay hedefi olan bir dosyayı belirtir. Eğer -t / -tf atlanırsa, ntlmrelayx NTLM kimlik doğrulamasını kaynak host'a aktarır, bu saldırıya NTLM self-relay saldırısı denir (yamalı olmasına rağmen, eski savunmasız host'larla karşılaşırsak, bu saldırıyı göz önünde bulundurmak faydalı olabilir). smb2support seçeneği, ihtiyaç duyan hostlar için SMBv2 desteği sağlar:


### NTLMRelayx SMB attack

![Pasted image 20241205185755.png](/img/user/resimler/Pasted%20image%2020241205185755.png)

Varsayılan olarak, ntlmrelayx NTLM kimlik doğrulamasını SMB üzerinden aktaracaktır. Aktardığımız oturum hedef makinede yüksek ayrıcalıklı erişime sahipse, ntlmrelayx bir SAM dump gerçekleştirmeye çalışacaktır.

SAM dump tekniği, SAM ve SECURITY registry hives içinde saklanan kimlik bilgilerini dump etmek için Windows hosts'a karşı kullanılır. Windows'un bilgileri LM ve NT hash'leri olarak depoladığı lokal kullanıcıların cache'lenmiş kimlik bilgilerinin çıkarılmasını sağlar

Şimdi Responder'ın bize herhangi bir kimlik doğrulaması yapmasını bekleyeceğiz ve bu gerçekleştiğinde ntlmrelayx -tf relayTargets.txt seçeneğini kullanarak bunu hedef olarak tanımlanan bilgisayarlara iletecektir.


### NTLMRelayx SAM DUMP

![Pasted image 20241205185934.png](/img/user/resimler/Pasted%20image%2020241205185934.png)
![Pasted image 20241205185943.png](/img/user/resimler/Pasted%20image%2020241205185943.png)
![Pasted image 20241205185957.png](/img/user/resimler/Pasted%20image%2020241205185957.png)

Not: NTLM Relay saldırısı çalışmazsa, ntlmrelayx ve Responder'ı yeniden başlatabiliriz.

Aşağıdaki şemada gerçekleştirdiğimiz saldırı gösterilmektedir:

![Pasted image 20241205190035.png](/img/user/resimler/Pasted%20image%2020241205190035.png)

Yukarıdaki resimde gösterildiği gibi, burada ne olduğu basitleştirilmiştir:

1. Saldırı makinesinde (172.16.117.30) Responder & ntlmrelayx'i başlattık.
2. DC'den (172.16.117.3) bir kullanıcı bir UNC'yi yanlış yazdığında ve Windows ona bağlanmaya çalıştığında, Responder yanıtlarını zehirler ve kullanıcıyı Saldırı makinemize karşı kimlik doğrulaması yapmaya yönlendirir.
3. Saldırı hostumuza bağlandıklarında, ntlmrelayx bu kimlik doğrulamalarını hedef olarak yapılandırılmış sunuculara (172.16.117.50 ve 172.16.117.60) iletti ve INLANEFREIGHT/PETER kullanıcısı 172.16.117.50 bilgisayarlarından birinde yöneticiydi ve bu da bir SAM dökümüne neden oldu.

INLANEFREIGHT/[email protected] adresinden gelen bağlantılardan birini 172.16.117.50 hedef makinesine başarıyla aktardık. INLANEFREIGHT/[email protected] adresinden 172.16.117.60 adresine yapılan kimlik doğrulaması başarılı oldu, ancak kullanıcının bu hedefte yüksek ayrıcalıkları olmadığı için hiçbir şey olmadı.


### Command Execution
SAM dump yapmak yerine, -c “Command To Execute” seçeneğini kullanarak hedef makinede komut yürütme gerçekleştirebiliriz ve komutlar SMB üzerinden yürütülür. İlk olarak saldırı hostumuza bir kez ping atmayı deneyelim:

![Pasted image 20241205190832.png](/img/user/resimler/Pasted%20image%2020241205190832.png)
![Pasted image 20241205190837.png](/img/user/resimler/Pasted%20image%2020241205190837.png)

Responder yayın trafiğini zehirlediğinde, ntlmrelayx'in 172.16.117.3'ten gelen PETER'in NTLM kimlik doğrulamasını aktardığını ve 172.16.117.50'de kimliği doğrulanmış bir oturum kurduğunu fark edeceğiz; daha sonra, kimliği doğrulanmış oturumu PETER olarak kötüye kullandı ve ping komutunu çalıştırdı ve sonunda çıktısını görüntüledi:

![Pasted image 20241205190929.png](/img/user/resimler/Pasted%20image%2020241205190929.png)

Basit bir ping çalıştırmak yerine, bir reverse shell alabiliriz, bunun için [Nishang](https://github.com/samratashok/nishang)'dan [InvokePowerShellTcp.ps1'](https://github.com/samratashok/nishang/blob/master/Shells/Invoke-PowerShellTcp.ps1)i kullanalım. İlk olarak, aracı indireceğiz (araç zaten /home/htb-student/tools/ içinde) ve bir Python web sunucusu başlatacağız, böylece aktarım hedefi Invoke-PowerShellTcp.ps1'i almak için onu kullanabilir:

![Pasted image 20241205191020.png](/img/user/resimler/Pasted%20image%2020241205191020.png)

Ardından, reverse shell'i yakalamak için bir nc dinleyicisi başlatmamız gerekir:

![Pasted image 20241205191036.png](/img/user/resimler/Pasted%20image%2020241205191036.png)

Son olarak, ntlmrelayx'in -c seçeneğini kullanarak relay hedefinin Python web sunucumuzdan Invoke-PowerShellTcp.ps1 dosyasını indirmek ve çalıştırmak üzere bir powershell komutu yürütmesini sağlayacağız. İki komutu noktalı virgülle ayırıyoruz, böylece ilk komut Invoke-PowerShellTcp.ps1 PowerShell modülünü indirip belleğe yüklüyor ve ikinci komut -Reverse , -IPAddress 172.16.117.30 saldırı hostumuz için ve -Port 7331 nc dinleyicisi için parametrelerini belirterek modülü kullanıyor:

![Pasted image 20241205191201.png](/img/user/resimler/Pasted%20image%2020241205191201.png)

nc listener'ı kontrol ederken, NT Authority\System olarak bir reverse shell bağlantısının başarıyla kurulduğunu fark edeceğiz:

![Pasted image 20241205191233.png](/img/user/resimler/Pasted%20image%2020241205191233.png)
![Pasted image 20241205191239.png](/img/user/resimler/Pasted%20image%2020241205191239.png)

ntlmrelayx'in temel kullanımlarını tartıştıktan sonra, bir sonraki bölümde diğer gelişmiş kullanım durumlarına bakacağız. 


### NTLMRelayx Kullanım Örnekleri 

Önceki bölümde, uzaktaki bir makinenin SAM'ını dump etmek ve uzaktan kod çalıştırma elde etmek gibi temel ntlmrelayx kullanım durumlarını gördük. Bu bölümde ntlmrelayx'in diğer gelişmiş kullanım durumları gösterilecektir.


### Multi-relaying

impacket'in ana bakımcısı @0xdeaddood tarafından [SecureAuth](https://www.secureauth.com/blog/we-love-relaying-credentials-a-technical-guide-to-relaying-credentials-everywhere/)'un blogunda açıklandığı gibi, ntlmrelayx'in multi-relay özelliği şunları sağlar:

* NTLM kimlik doğrulaması aldığımız kullanıcıları tanımlama (NTLM kimlik doğrulamalarını iletmek isteyip istemediğimize karar vermemizi sağlar).
* Tek bir NTLM kimlik doğrulamasını (bağlantı) birden fazla hedefe aktarma.

ntlmrelayx, multi-relay özelliğini, client'ların saldırı makinesinde local olarak kimlik doğrulamasını yaparak, kimliklerini getirerek/çıkararak ve daha sonra bağlantılarını relay hedeflerine aktarabilmek için onları yeniden kimlik doğrulamaya zorlayarak uygular. Multi-relaying, ntlmrelayx'in HTTP ve SMB sunucuları için varsayılan davranıştır; ancak, devre dışı bırakmak için --no-multirelay seçeneğini kullanarak bağlantıları yalnızca bir kez aktarmasını sağlayabiliriz (yani, gelen bir bağlantı yalnızca bir saldırıyla eşleşir).


### Hedef Tanımı

ntlmrelayx'in relay hedeflerini belirlemek için -t ve -tf seçeneklerini sunduğundan daha önce bahsetmiştik. Multi relay özelliği nedeniyle, hedefler ya named hedefler ya da general hedefler olabilir; named hedefler kimlikleri belirtilmiş hedeflerdir, general hedefler ise kimliksiz hedeflerdir. Hedeflerin tanımlanması, syntax'ı üç bileşenden oluşan [URI formatını](https://datatracker.ietf.org/doc/html/rfc3986) takip eder: scheme://authority/path :

* scheme : Hedeflenen protokolü tanımlar (örn. http veya ldap ); sağlanmadığı takdirde varsayılan protokol olarak smb kullanılır. Joker anahtar kelime all, ntlmrelayx'in desteklediği tüm protokolleri kullanmasını sağlar.

* authority : DOMAIN_NAME\\USERNAME@HOST:PORT biçimi kullanılarak belirtilir. General hedefler DOMAIN_NAME\\USERNAME kullanmaz; sadece named hedefler kullanır.

* path : İsteğe bağlıdır ve yalnızca belirli saldırılar için gereklidir, örneğin relayed HTTP NTLM kimlik doğrulaması kullanarak erişimi kısıtlanmış web endpoint'lerine erişirken (AD CS'yi Hedefleyen Gelişmiş NTLM Relay Saldırıları bölümünde ele alacağımız gibi).

ntlmrelyax'ın HTTP ve SMB sunucuları, tek bir genel hedefe saldırmak dışında, varsayılan olarak multi-relay özelliğine sahiptir. Hedef türüne bağlı olarak, ntlmrelayx multi-relay özelliği için aşağıdaki varsayılan ayarlara sahiptir:

![Pasted image 20241205193426.png](/img/user/resimler/Pasted%20image%2020241205193426.png)

Multi-relay özelliğini ve ntlmrelayx'in farklı hedef türlerini anlamak çok önemlidir; farklı hedef türlerini ve varsayılan multi-relaying durumlarını örneklerle göstererek nedenini görelim.

Ntlmrelayx'e “smb://172.16.117.50” hedefini kullanmasını söylediğimizi varsayalım; bu durumda, genel bir hedef olduğu için multi-relay devre dışı bırakılacak ve ntlmrelayx yalnızca herhangi bir kullanıcıya (herhangi bir hosttan) ait ilk NTLM kimlik doğrulama bağlantısını SMB üzerinden 172.16.117.50 relay hedefine aktaracaktır. Bir bağlantı yalnızca bir saldırıyla eşleştiğinden bu ilişki 1:1'dir (ntlmrelayx'in aynı anda iki farklı bağlantı aldığı bir uç durum söz konusudur; multi-relay devre dışı bırakılacak olsa da, ntlmrelayx iki bağlantıyı da reddetmek yerine yanlış bir şekilde aktarır):

![Pasted image 20241205193609.png](/img/user/resimler/Pasted%20image%2020241205193609.png)

Alternatif olarak, ntlmrelayx'e “ smb://INLANEFREIGHT\\ [email protected] ” hedefini kullanmasını söylediğimizi varsayalım; bu durumda, tek bir adlandırılmış hedef olduğundan, multi-relay etkinleştirilecek ve ntlmrelayx, INLANEFREIGHT\PETER'a ait herhangi bir sayıda NTLM kimlik doğrulama bağlantısını (herhangi bir hosttan) SMB üzerinden 172.16.117.50 aktarma hedefine aktaracaktır ( domain adını ve kullanıcı adını tam olarak ntlmrelayx'in çıktısında gösterildiği gibi sağlamalıyız). Birçok bağlantı birçok saldırıyla eşleştiği için bu ilişki M:M şeklindedir:

![Pasted image 20241205193718.png](/img/user/resimler/Pasted%20image%2020241205193718.png)

Ya aynı genel hedefi “ smb://172.16.117.50 ” kullanmak istiyorsak ama aynı zamanda multi-relaying'i etkinleştirmek istiyorsak? Bunu yapmak için, hedefi bir dosyaya koymalı ve varsayılan olarak multi-relaying'i etkinleştiren -tf seçeneğini kullanmalıyız. Bu nedenle, genel bir hedef içeren dosyadan bağımsız olarak, -tf seçeneği nedeniyle multi relaying etkinleştirildiğinden, ntlmrelayx herhangi bir kullanıcıya (herhangi bir hosttan) ait herhangi bir sayıda NTLM kimlik doğrulama bağlantısını SMB üzerinden relay hedefi 172.16.117.50'ye iletecektir. Genel hedefler gibi 1:1 ilişki yerine, birçok bağlantı birçok saldırıyla eşleştiği için bu M:M olur:

![Pasted image 20241205193812.png](/img/user/resimler/Pasted%20image%2020241205193812.png)

Son olarak, aynı adlandırılmış hedefi “ smb://INLANEFREIGHT\\ [email protected] ” kullanmak istediğimizi, ancak yalnızca ntlmrelayx'in INLANEFREIGHT\PETER kullanıcısı için aktarabileceği ilk bağlantıyı kötüye kullanmak istediğimizi varsayalım; bunu yapmak için --nomultirelay seçeneğiyle multi-relay'i devre dışı bırakabiliriz:

![Pasted image 20241205193842.png](/img/user/resimler/Pasted%20image%2020241205193842.png)

Not: Bu modül boyunca, hizmetlerin varsayılan portlarında çalıştığını varsayacağız. Ancak, sistem yöneticilerinin bu varsayılan portları değiştirebileceğini her zaman hatırlayın, bu nedenle ağı numaralandırdıktan sonra böyle durumlarla karşılaşırsak, hizmetlerin portlarını buna göre değiştirmemiz gerekir




### SOCKs Connections
ntlmrelayx'in bir başka harika seçeneği de -socks'tur; ntlmrelayx her başladığında, aktarılan kimliği doğrulanmış oturumları tutmak ve aktif tutmak için kullanabileceğimiz bir SOCKS proxy'si başlatır ve bunları herhangi bir araçla kötüye kullanmamıza izin verir. Örneğin, -tf seçeneği ile birden fazla hedefimiz varsa, -socks kullanabilir ve mümkün olduğunca çok bağlantıyı aktif tutabiliriz. Bunu iş başında görelim.

İlk olarak, ntlmrelayx'i -socks seçeneği ile çalıştırıyoruz (hala SMB protokolünü hedefliyoruz):

![Pasted image 20241205194841.png](/img/user/resimler/Pasted%20image%2020241205194841.png)

Ardından, ağı zehirlemek için Responder'ı kullanacağız:


### Responder Poisoning Modunda Çalıştırma

![Pasted image 20241205194932.png](/img/user/resimler/Pasted%20image%2020241205194932.png)

Responder'ı çalıştırıp yayın trafiğini zehirlemesini sağladıktan sonra, ntlmrelayx'in farklı IP'lerden gelen birden fazla kullanıcının NTLM kimlik doğrulamasını aktardığını ve 172.16.117.50 ve 172.16.117.60 üzerinde kimlik doğrulaması yapılmış oturumlar kurarak bunları SOCKS sunucusuna eklediğini göreceğiz:

![Pasted image 20241205194959.png](/img/user/resimler/Pasted%20image%2020241205194959.png)
![Pasted image 20241205195006.png](/img/user/resimler/Pasted%20image%2020241205195006.png)
![Pasted image 20241205195026.png](/img/user/resimler/Pasted%20image%2020241205195026.png)

Responder ve -socks seçeneklerini çalıştırdıktan sonra ntlmrelayx içinde bir komut satırı arayüzünü etkinleştirir; help komutu mevcut komutları yazdırır (hata ayıklama mesajları yazdırılmaya devam etse bile etkileşimli komutları yazmaya devam edebilirsiniz): poison broadcast trafiğini çalıştırdıktan sonra, ntlmrelayx'in farklı IP'lerden gelen birden fazla kullanıcının NTLM kimlik doğrulamasını ilettiğini ve 172.16.117.50 ve 172.16.117.60 üzerinde kimliği doğrulanmış oturumlar kurarak bunları SOCKS sunucusuna eklediğini fark edeceğiz:

![Pasted image 20241205195100.png](/img/user/resimler/Pasted%20image%2020241205195100.png)
![Pasted image 20241205195107.png](/img/user/resimler/Pasted%20image%2020241205195107.png)


socks etkileşimli komutu etkin oturumları listeler:

![Pasted image 20241205195120.png](/img/user/resimler/Pasted%20image%2020241205195120.png)

-socks seçeneği ntlmrelayx'in bir proxy gibi davranmasını sağlar, yani tuttuğu oturumları kullanmak için onu proxychains ile birleştirebiliriz. INLANEFREIGHT/PETER'a ait yalnızca bir kimliği doğrulanmış oturum, 172.16.117.50 relay hedefi üzerinde yönetici ayrıcalıklarına sahiptir (AdminStatus TRUE olarak ayarlanmıştır). Kimliği doğrulanmış oturumu kötüye kullanmak için herhangi bir impacket aracını proxyychains üzerinden tünelleyebiliriz. Ancak, öncelikle proxychains yapılandırma dosyasını SOCKS4/5 için ntlmrelayx'in varsayılan proxy portu olan 1080'i kullanacak şekilde ayarlamalıyız. Linux'ta proxychains yapılandırma dosyasının konumu varsayılan olarak /etc/proxychains.conf şeklindedir:

![Pasted image 20241205195225.png](/img/user/resimler/Pasted%20image%2020241205195225.png)

Not: Yüklü proxychains sürümüne bağlı olarak, yapılandırma dosya adı /etc/proxychains4.conf veya /etc/proxychains.conf olabilir.

INLANEFREIGHT/PETER'ın kimliği doğrulanmış yönetici oturumunu 172.16.117.50'de remote code execution elde etmek için kullanacağız. Aynı domain adını ve kullanıcı adını kullanarak smbexec.py dosyasını proxychains üzerinden tünelleyeceğiz. ntlmrelayx'in SOCKS proxy'si tarafından oluşturulan kimliği doğrulanmış oturumu kötüye kullanacağımız için smbexec.py'nin bizden bir parola istemesini önlemek için -no-pass seçeneğini kullanacağız (SOCKs sunucusunda bulunan kimliği doğrulanmış oturumları kullanmak için ntlmrelayx'i çalışır durumda tutmamız gerektiğini unutmayın, aksi takdirde onu kapatırsak SOCKS sunucusu da kapanır):

![Pasted image 20241205195548.png](/img/user/resimler/Pasted%20image%2020241205195548.png)

Ancak, hedef(ler) üzerinde herhangi bir yönetim iznimiz olmadığında, AdminStatus FALSE olduğunda, paylaşılan bir klasöre bağlanmak veya diğer yönetimsel olmayan görevleri gerçekleştirmek için yine de bir oturum oluşturabiliriz. Örneğin, 172.16.117.50 adresine bağlanmak, tüm paylaşımlı klasörleri listelemek ve Finance paylaşımlı klasörüne bağlanmak için RMONTY hesabını kullanalım:

![Pasted image 20241205195624.png](/img/user/resimler/Pasted%20image%2020241205195624.png)

Paylaşılan klasörlere erişime sahip olmak çok yararlı görünmese de, ilerleyen bölümlerde, istemcileri rızaları olmadan eylemler gerçekleştirmeye zorlamak için paylaşılan klasörlere erişime sahip olarak zincirleyebileceğimiz diğer teknikleri öğreneceğiz.

### Etkileşimli SMB Client Shells
Alternatif olarak, her ntlmrelayx kurulan kimlik doğrulamalı oturum için bir SMB client shell başlatmak için --interactive / -i seçeneğini kullanabiliriz. SMB client shell local olarak bir TCP portunu dinleyecektir ve nc gibi araçlarla ona ulaşabiliriz:


### Etkileşimli Seçeneği Kullanma

![Pasted image 20241205195749.png](/img/user/resimler/Pasted%20image%2020241205195749.png)

ntlmrelayx, relay hedeflerindeki kimliği doğrulanmış her oturum için etkileşimli SMB client shell'leri başlattı; Her bağlantı hedef makineye bir port açacaktır. Yukarıdaki çıktıda bu satırları aramamız gerekiyor: [*] Kimlik doğrulama smb://172.16.117.50 as INLANEFREIGHT/RMONTY SUCCEED ve ardından [*] 127.0.0.1:11000 üzerinde TCP aracılığıyla etkileşimli SMB client shell başlatıldı. RMONTY'nin 172.16.117.50 oturumunu kullanmak istiyorsak netcat kullanarak 127.0.0.1 üzerindeki 11000 portuna bağlanmamız gerekir:


### Interactive Shell
![Pasted image 20241205195843.png](/img/user/resimler/Pasted%20image%2020241205195843.png)
![Pasted image 20241205195847.png](/img/user/resimler/Pasted%20image%2020241205195847.png)

SMB üzerinden NTLM kimlik doğrulamasını aktardıktan ve birden fazla SMB post-relay saldırısı gerçekleştirdikten sonra, bir sonraki bölümde çapraz protokol relay saldırıları hakkında bilgi edineceğiz.
