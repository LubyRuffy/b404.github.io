---
title: '密码破解'
layout: post
tags: 
  - readings
  - sec
  - tools
category: 
  - tools
  - sec
comments: true
share: true
---

* TOC
{:toc}

<!--more-->

## MISC

### findmyhash

```bash
findmyhash MD5 -h 21232f297a57a5a743894a0e4a801fc3
```

![](\img\bruteforce\findmyhash.png)

### hash-identifier

查看hash类型：

![](\img\bruteforce\crack_accesspassword.png)

## Hydra

Hydra是世界上顶级的密码暴力破解工具，支持几乎所有协议的在线 密码破解，功能强大。


```bash
Hydra v8.1 (c) 2014 by van Hauser/THC - Please do not use in military or secret service organizations, or for illegal purposes.

Syntax: hydra [[[-l LOGIN|-L FILE] [-p PASS|-P FILE]] | [-C FILE]] [-e nsr] [-o FILE] [-t TASKS] [-M FILE [-T TASKS]] [-w TIME] [-W TIME] [-f] [-s PORT] [-x MIN:MAX:CHARSET] [-SuvVd46] [service://server[:PORT][/OPT]]
 **Options**:
-l LOGIN or -L FILE  指定破解用户名称，对特定用户破解  

-p PASS  or -P FILE  指定密码或者指定字典破解  

-C FILE   使用冒号分割 格式，如"login:pass" , 代替 -L/-P  

-M FILE   指定服务器目标列表文件为每行一条, ':' 指出端口  

-t TASKS  同时运行的线程数 (每台主机默认 16)  

-U        服务模块使用细节  

-h        更多的命令行选项  

server    目标服务器名称或者ip或者 192.168.0.0/24 (使用此选项或者 -M 选项)  

service   指定服务名，支持的服务和协议  

OPT       一些服务模块支持的额外的输入（-U选项用于获取模块的帮助信息
```


> 支持的服务和协议包括: asterisk cisco cisco-enable cvs firebird ftp ftps http[s]-{head|get} http[s]-{get|post}-form http-proxy http-proxy-urlenum icq imap[s] irc ldap2[s] ldap3[-{cram|digest}md5][s] mssql mysql nntp oracle-listener oracle-sid pcanywhere pcnfs pop3[s] postgres rdp redis rexec rlogin rsh s7-300 sip smb smtp[s] smtp-enum snmp socks5 ssh sshkey svn teamspeak telnet[s] vmauthd vnc xmpp

```bash
hydra -l user -P passlist.txt ftp://192.168.0.1
```

### Hydra用法实例:

**破解ssh账号:**

破解ssh账号有两种方式，一种 是指定账号破解，另外一种是指定用户列表破解，命令如下：

```bash
hydra -l 用户名 -p 密码字典 -t 线程 -vV -e ns ip ssh    

hydra -l root -P pwd.dic -t 1 -vV -e ns -o save.log 192.168.44.139 ssh //对root账户进行破解，将结果保存在save.log中    
```

** 破解FTP账号:**

破解指定用户名密码:

```bash
    hydra ip ftp -l 用户名 -P 密码字典 -t 线程（默认16）-vV        
    hydra ip ftp -l 用户名 -P 密码字典 -e ns -vV    
```

**批量破解FTP账号和密码：**

```bash
    hydra -L list_user -P list_password 192.168.10.2 ftp -V  
```

**GET方式提交，破解WEB登录：**

```bash
    hydra -l 用户名 -P 密码字典 -s 80 ip http-post-form “/admin/login.php:username=^USER^&password=^PASS^&submit=login:sorry password”  

    hydra -L list_user -P list_password 192.168.10.4 http-post-form "member.php？mod=login&loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1:fastloginfield=username&username=^USER^&password=^PASS^&password=^PASS^&quickforward=yes&handlekey=ls:Login failed" -V    
```

	以上表示对192.168.10.4进行post提交破解，定义了登录的url，以及登录验证和错误登录标记。

```bash
member.php?mod=logging&action=loginsubmit=yes&infloat=yes&lssubmit=yes&inajax=1fastloginfield=username&username=^USER^&password=^PASS^&quickforward=yes&handlekey=lsLogin failed    
```

对admin密码进行破解:

```bash
    hydra -t 3 -l admin -P pass.txt -o out.txt -f 192.168.0.11 http-post-form"login.php:id=^USER^&password=^PASS^:<title>wrong username or password</title>"    
```

> -t表示同时线程数为3；“-l”表示用户名是“admin”，字典为pass.txt，保存为out.txt；“-f”表示破解一个密码就停止；“192.168.0.11”表示目标ip；“http-post-form”表示采用HTTP的POST方式提交表单密码破解；“”中的内容是错误猜解返回提示。


**破解HTTPS：**

```bash
   hydra -m /index.php -l muts -P pass.txt 192.168.10.1 https  
```

**破解teamspeak：**

```bash
hydra -l 用户名 -P 密码字典 -s端口号 -vV ip teamspeak  
```

**破解Cisco:**

```bash   
hydra -P pass.txt 192.168.10.3 cisco    
hydra -m cloud -P pass.txt 192.168.10.3 cisco-enable    
```

**破解SMB：**

```bash
    hydra -l administrator -P pass.txt 192.168.0.115 smb  
```

**破解POP3：**

```
hydra -l muts -P pass.txt my.pop3.mail pop3
```

**破解远程终端账号:**

```
//破解管理员账号：
hydra ip rdp -l administrator -P pass.txt -V    


//批量破解账号:
hydra -s 3389 192.168.22.19 rdp -L user.txt -P pwd.txt -V    

	
//破解HTTP-Proxy:
hydra -l admin -P pass.txt http-proxy://192.168.10.8    

//破解IMAP :
hydra -L user.txt -p secret 192.168.9.2 imap PLAIN    
hydra -C defaluts.txt -6 imap://[fe80::2c:31ff:fe12:ac11]:143/PLAIN  
```

## Hashcat

oclHashcat号称是世界上最快的密码破解工具，是世界上唯一第一个基于GPGPU规则的引擎，提供免费的GPU（高达128个GPU）、多哈希、多操作系统（Linux和Windows本地二进制文件）、多平台（OpenCL和CUDA支持）、多算法机制、资源利用率低，基于字典攻击，分布式破解


```bash
-a  指定要使用的破解模式
-m  指定要破解的hash类型所对应的id
-o  指定破解成功后的hash及所对应的明文密码的存放位置,可以用它把破解成功的hash写到指定的文件中
--force	忽略破解过程中的警告信息,跑单条hash可能需要加上此选项
--show	显示已经破解的hash及该hash所对应的明文
--increment	 启用增量破解模式,你可以利用此模式让hashcat在指定的密码长度范围内执行破解过程,其实,并不建议这么用,因为破解时间可能会比较长
--increment-min  密码最小长度,后面直接等于一个整数即可,配置increment模式一起使用
--increment-max  密码最大长度,同上
--outfile-format 指定破解结果的输出格式id,一般自己常用3
--username 	 忽略hash文件中的指定的用户名,在破解win和linux系统用户密码hash可能会用到
--remove 	 删除已被破解成功的hash
-r		 使用自定义破解规则

```

`-m`支持的hash类型：

| # | Name |Category|
|----|----|-----|
|900 | MD4| Raw |Hash|
|0 | MD5| Raw Hash|
|5100|Half MD5| Raw Hash|
| 100 | SHA1 | Raw Hash
|1300 | SHA-224| Raw |Hash|
|1400 | SHA-256 | Raw Hash|
|10800 | SHA-384 | Raw Hash|
|1700 | SHA-512     | Raw Hash|
|5000 | SHA-3 (Keccak)   | Raw Hash|
|600 | BLAKE2b-512                       | Raw Hash|
|...|...|...|



### 掩码规则


Hashcat-plus中以?l表示小写字母，?d表示数字，?u表示大写字母，?s表示所有可打印符号，?a代表所有可打印字符，它等于?l?u?d?s加在一起。


```bash
l | abcdefghijklmnopqrstuvwxyz		纯小写字母
u | ABCDEFGHIJKLMNOPQRSTUVWXYZ		纯大写字母
d | 0123456789			        纯数字
h | 0123456789abcdef			常见小写字母和数字
H | 0123456789ABCDEF			常见大写字母和数字
s |  !"#$%&'()*+,-./:;<=>?@[\]^_`{|}~		特殊字符
a | ?l?u?d?s			以上所有字符
b | 0x00 - 0xff			可能是用来匹配像空格这种密码的
```

```bash
?l?l?l?l?l?l?d?d?d?d  

//表示6位小写字母和4位数字组成的密码

de?l?d?s56pos 

//表示由de加一位小写字母加一位数字加一位特殊字符后面跟上56pos组成的密码
```

自定义字符集规则：

```bash
-1, --custom-charset1   | CS   | User-defined charset ?1   | -1 ?l?d?u
-2, --custom-charset2   | CS   | User-defined charset ?2   | -2 ?l?d?s
-3, --custom-charset3   | CS   | User-defined charset ?3   |
-4, --custom-charset4   | CS   | User-defined charset ?4
```

自定义规则例子：

```bash
-1 ?l?s	?1?1?1?1?1 	
//表示五位由特殊字符和小写字母组成的密码


-1 ?d?l	-2 ?d?l?u -3 ?l?u ?1?2?3	

//表示密码的第一位可能是小写字母或者数字,第二位可能是大小写字母或者数字,第三位可能是大或小写字母
```




### 破解模式

- `0 | Straight`:纯粹基于字典的爆破模式，后面可以连续跟上多个字典文件，破解的成功与否取决于字典质量
- `1 | Combination`：相对智能高效的爆破模式。已知密码种包含哪些字符串，将那些字符串写入文件，每行对应一个字符串，hashcat根据所提供的字符串尝试可能的组合进行猜解
- `3 | Brute-force`：基于掩码的爆破方式，根据给定的掩码进行破解，不会轮询破解。若想让其自动轮询，可以启用`increment`模式指定密码的最小和最大长度
- `6 | Hybrid Wordlist + Mask`：基于字典和掩码配合的爆破模式，将前面字典取出的一个字符串和后面掩码进行拼接，碰撞出密码明文
- `7 | Hybrid Mask + Wordlist`：
- 基于掩码和字典配合的爆破，将前面的掩码和后面的字典进行拼接爆破

**纯字典爆破：**

```bash
hashcat --force -a 0 -m 0 test.txt /opt/SecLists/Passwords/Leaked-Databases/rockyou.txt -o test2.txt
```

![](\img\bruteforce\hashcat1.png)

**智能字典爆破：**

```bash
hashcat --force -a 1 -m 0 test.txt d1.txt d2.txt
```

**基于纯掩码爆破：**

* 爆破9位的纯字母hash：

```bash
hashcat --force -a 3 -m 0 hash.txt ?l?l?l?l?l?l?l?l?l
```

* 前三位小写字母，后四位数字：

```bash
hashcat --force -a 3 -m 0 hash.txt ?l?l?l?d?d?d?d
```

* 爆破8位纯数字的hash：

```bash
hashcat --force -a 3 -m 0 hash.txt ?d?d?d?d?d?d?d?d?d?d
```

* 知道部分字符，爆破剩余字符：

```bash
hashcat --force -a 3 -m 0 hash.txt ?l?lve?l?la?l?l
```

* 字典和掩码配合的爆破模式

```bash
hashcat --force -m 0 test.txt -a 6 d1.txt -1 ?l ?1?d?d?d
```

![](\img\bruteforce\hashcat3.png)

* 多个字典爆破

```bash
hashcat --force -m 0 test.txt -a 6 d1.txt d2.txt -1 ?l ?l?1?1?1?d?d?d
```

![](\img\bruteforce\hashcat4.png)

* 基于掩码和字典配合的爆破模式

```bash
hashcat --force -m 0 test.txt -1 ?l?d ?1?1?1?1 -a 7 d1.txt d2.txt
```

![](\img\bruteforce\hashcat5.png)


* 基于increment的自动变长模式:



****


### 字典爆破

将准备好的字典，pass.txt、需要破解的Hash值文件win32.hash复制到oclHashcat32程序所在的文件夹下，然后执行:

```bash
oclHashcat32 -m 1000 -a 0 -o winpass1.txt --remove win2.hash ptemp.txt  
```

> “-m 1000”表示破解类型为NTLM.
> “-a 0”表示字典破解。
> “-o”表示输出结果到winpass1.txt.
> “–remove win2.hash”表示将移除破解成功的Hash。
> “ptemp.txt”为密码字典。

**oclHashcat破解Discuz！论坛密码算法md5(md5(salt)：**

```bash
oclHashcat32 -m 2611 -a 0 -o winpass.txt --remove dz. hash ptemp.txt  
```

**Linux的SHA-512加密方式，破解如下：**

```bash
oclHashcat32 -m 1800 sha512linux.txt p.txt  
```

**Linux的SHA-md5加密方式，破解如下:**

```bash
oclHashcat32 -m 500 linuxmd5.txt p.txt  
```



> 后添加部分refer: https://klionsec.github.io/2017/04/26/use-hashcat-crack-hash/



## Serv-U密码破解

Serv-U是一款流行的FTP服务器，很多Web服务器通过它提供文件上传、下载。在权限合适情况下，可通过Webshell获取服务器权限。

Serv-U早期版本采用MD-5加密，在获取Serv-U这个文件夹下的ServUDaemon.ini文件后可以对Ftp用户密码进行破解。Serv-U安装目录一般位于C:\programfils\Serv-U\（视情况而定），在获得Webshell权限情况下，若服务器上还安装了Serv-U，找到ServU目录，将ServUDaemon.ini复制到本地留待后面进行破解。

配置文件对大小写不敏感，行与行之间允许空行，主要分为[GLOBAL]全局变量段和[DOMAINS]域名配置段。[GLOBAL]段主要设置Serv-U的注册号以及刷新标志。[DOMAINS]，包括了在Serv-U下添加的所有域信息以及以下用户列表。


```bash
[GLOBAL]
Version=5.0.0.0
RegistrationKey= //产品注册码
ProcessID= //注册号
ReloadSettins=True //在修改ini文件后加入此项，这时候Serv-U会自动刷新配置文件并生效，此项随之消失
[DOMAINS]
Domain1=0.0.0.0||21|Wizard Generated Domain|1|0|0
//无须改动，新增加的域的IP地址以及说明，格式。ip为0.0.0.0时，serv-u自动适配系统所分配ip地址
//Domain1=ip|端口|域显示名称|是否生效|是否显示|是否删除
//生效位置为0,则此域禁用
//显示位置为0,此域不生效并且在控制面板不显示此项
//当删除位置0,则ReloadSettings设置为True后，即刷新后 ，自动删除此域名以下所有内容
[Domain1] //无须更改，与以下添加的域对应
User1=admin|1|0  //必须填，用户列表
//User序号=用户名|是否生效|是否删除
[USER=admin|1]
//用户配置段
```

```bash
[USER=zt828|1] 表示用户为zt。  
    password=mk7DC2A4B1A9A9E1F52A7F967FBCAA0A37表示ftp用户zt828的密码为“mk7DC2A4B1A9A9E1F52A7F967FBCAA0A37”。  
    HomeDir=d:\wwwroot\zt828表示默认主目录为“d:\wwwroot\zt828”。  
    MaxNrUsers=10表示最大用户连接数为10个。  
    RelPaths=1表示是否锁定用户到主目录，1表示锁定。  
    ChangePassword=1 表示是否可以修改密码，1表示可以。  
    DiskQuota=1|104857600|0 磁盘分配大小为100MB（100×1024×1024）。  
    SpeedLimitUp=102400 上传速度限制为100k/s。   
    SpeedLimitDown=102400 下载速度限制为100k/s。  
    Access1=d:\wwwroot\zt828|RL 读取和列表d:\wwwroot\zt828目录。
```

> 6.x版本之后，升级跨度大，没有了ServUDaemon.ini文件




## Access数据库破解

Access是微软推出的一种基于Windows的桌面关系数据库管理系统，关系式数据库由一系列表组成，表由一系列行和列组成，每行是一个记录，每一列是一个字段，每个字段有一个字段名，字段名在一个表中不能重复。表与表之间可以建立关系（或称关联，连接），以便查询相关的信息。Access 数据库以文件形式保存，文件的扩展名是`.MDB`。Access数据库由六种对象组成，它们是表、查询、窗体、报表、宏和模块。



破解Access数据库文件可以通过Access破解工具破解——“Access数据库特殊操作”，在窗口中选择“破解Access密码”，然后在Access文件路径中选择需要破解的文件，也可以直接输入Access文件路径。

![crack_accesspassword](/img/bruteforce/crack_accesspassword.png)

-----------------------------------

## MySQL数据库

MYSQL数据库用户密码跟其它数据库用户密码一样，在应用系统代码中都是以明文出现的，在获取文件读取权限后即可直接从数据库连接文件中读取，例如asp代码中的conn.asp数据库连接文件，在该文件中一般都包含有数据库类型，物理位置，用户名和密码等信息；而在MYSQL中即使获取了某一个用户的数据库用户（root用户除外）的密码，也仅仅只能操作某一个用户的数据库中的数据。

在实际攻防过程中，在获取Webshell的情况下，是可以直下载MYSQL数据库中保留用户的user.MYD文件，该文件中保存的是MYSQL数据库中所有用户对应的数据库密码，只要能够破解这些密码那么就可以正大光明的操作这些数据，虽然网上有很多修改MYSQL数据库用户密码的方法，却不可取，因为修改用户密码的事情很容易被人发现！

### MySQL加密方式


MYSQL数据库的认证密码有两种方式，MYSQL 4.1版本之前是MYSQL323加密，MYSQL 4.1和之后的版本都是MYSQLSHA1加密，MYSQL数据库中自带Old_Password（str）和Password（str）函数,它们均可以在MYSQL数据库里进行查询，前者是MYSQL323加密，后者是MYSQLSHA1方式加密。
- 以MYSQL323方式加密

```
SELECT Old_Password('bbs.antian365.com');
```

- 以MYSQLSHA1方式加密

```
SELECT Password('bbs.antian365.com');
```
查询结果MYSQLSHA1 = *A2EBAE36132928537ADA8E6D1F7C5C5886713CC2    执行结果如图1所示，MYSQL323加密中生成的是16位字符串，而在MYSQLSHA1中生存的是41位字符串，其中*是不加入实际的密码运算中，通过观察在很多用户中都携带了“*”，在实际破解过程中去掉“*”，也就是说MYSQLSHA1加密的密码的实际位数是40位。

### MYSQL数据库文件结构

**1.MYSQL数据库文件类型**
MYSQL数据库文件共有“frm”、“MYD”“和MYI”三种文件，“.frm”是描述表结构的文件，
“.MYD”是表的数据文件，“.MYI”是表数据文件中任何索引的数据树。一般是单独存在一个文件夹中，默认是在路径“C:\Program Files\MYSQL\MYSQL Server 5.0\data”下。

**2.MYSQL数据库用户密码文件**
在MYSQL数据库中所有设置默认都保存在“C:\Program Files\MYSQL\MYSQL Server 5.0\data\MYSQL”中，也就是安装程序的data目录下，如图2所示，有关用户一共有三个文件即user.frm、user.MYD和user.MYI，MYSQL数据库用户密码都保存在user.MYD文件中，包括root用户和其他用户的密码。 

### 破解MYSQL密码

- 获取MYSQL数据库用户密码加密字符串 

使用UltraEdit-32编辑器直接打开user.MYD文件，打开后使用二进制模式进行查看，可以看到在root用户后面是一串字符串，选中这些字符串将其复制到记事本中，这些字符串即为用户加密值，即506D1427F6F61696B4501445C90624897266DAE3。（通常拿到shell之后，看服务器是否安装使用了MySql，通常是all user目录去找快捷方式，或者是其他方法（默认路径或者是猜到路径），跳转到mysql目录下的datamysql下，下载user.MYD文件，通过记事本，或者C32等查看，特殊情况可能查看的非正规位数时候，下载user.MYI 、user.frm和user.MYD三个文件，本地安装mysql进行查看！16位即是mysql 323的，40位即是mysql sha1的，mysql 323算法相对于mysql sha1较脆弱，比较容易破解，这里不详解了，因为这批彩虹表是针对mysql sha1的！）

> 注意：  
（1）root后面的“*”不要复制到字符串中。  
（2）在有些情况下需要往后面看看，否则得到的不是完整的MYSQLSHA1密码，总之其正确的密码位数是40位.

- 使用cain或者是PasswordsPro等工具破解
- 使用在线网站破解：
  * [cmd5](http://www.cmd5.com/)
  * [AskCheck](https://hashcracking.ru/index.php)
  * [mysql-password](http://www.mysql-password.com/hash)
  * [在线ophcrack](https://www.objectif-securite.ch/en/ophcrack.php)


##



