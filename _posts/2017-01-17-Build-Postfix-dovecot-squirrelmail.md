---
title: '在Ubuntu14.04上建立邮件服务器(Postfix-dovecot)'
layout: post
tags: 
  - ubuntu
  - postfix
  - dovecot
  - mailserver
  - sequirrelmail
  - environment
category: 
  - mailserver 
  - environment
comments: true
share: true
description: 在Ubuntu14.04server上基于Postfix和dovecot，搭建Squirrelmail邮件服务器
---
在Ubuntu14.04server上基于Postfix和dovecot，搭建Squirrelmail邮件服务器

<!--more-->

* TOC
{:toc}

# 在Ubuntu14.04上建立邮件服务器(Postfix-dovecot)

@(环境搭建)[Ubuntu, postfix, mail]


> Postfix(smtp,发送)

> Dovecot(pop3,imap,接收)

> Squirrelmail(用于Webmail连接)
 
安装和配置postfix:
---------
1.第一步：分配静态IP和hostname，添加一个主机名。打开`/etc/hostname`,在里面添加：

```c
192.168.1.10  mail.****.xyz
```

2.更新源仓库：

```c
root@mail:~# sudo apt-get update
```

3.安装postfix和其依赖包。安装过程中，全部直接`Enter`。

```c
root@mail:~$ sudo apt-get install postfix
```

4.安装完成，就通过下面的命令配置postfix

```c
root@@mail:~$ sudo dpkg-reconfigure postfix
```
现在接下来会遇到一些配置选择，根据自己域名情况配置：

```c
1. Internet Site
2. ****.xyz
3. ****
4. ****.xyz, localhost.localdomain, localhost
5. No
6. 127.0.0.0/8 [::ffff:127.0.0.0]/104 [::1]/128 192.168.0.0/24
7. 0
8. +
9. all
```
5.现在配置Postfix，用于SMTP-AUTH(使用Dovecot SASL)。通过添加以下的配置参数到`/etc/postfix/main.cf`中。

```c
home_mailbox = Maildir/
smtpd_sasl_type = dovecot
smtpd_sasl_path = private/auth
smtpd_sasl_local_domain =
smtpd_sasl_security_options = noanonymous
broken_sasl_auth_clients = yes
smtpd_sasl_auth_enable = yes
smtpd_recipient_restrictions = permit_sasl_authenticated,permit_mynetworks,reject_unauth_destination
smtp_tls_security_level = may
smtpd_tls_security_level = may
smtp_tls_note_starttls_offer = yes
smtpd_tls_loglevel = 1
smtpd_tls_received_header = yes
```
![Alt text](/img/mailserver/1479370288872.png)

6.制作tls证书，根据以下的命令

```c
root@mail:~$ openssl genrsa -des3 -out server.key 2048
root@mail:~$ openssl rsa -in server.key -out server.key.insecure
root@mail:~$ mv server.key server.key.secure
root@mail:~$ mv server.key.insecure server.key
root@mail:~$ openssl req -new -key server.key -out server.csr
root@mail:~$ openssl x509 -req -days 365 -in server.csr -signkey server.key -out server.crt
root@mail:~$ sudo cp server.crt /etc/ssl/certs
root@mail:~$ sudo cp server.key /etc/ssl/private
```
7.配置证书路径：

```c
root@mail:~$ sudo postconf -e 'smtpd_tls_key_file = /etc/ssl/private/server.key'
root@mail:~$ sudo postconf -e 'smtpd_tls_cert_file = /etc/ssl/certs/server.crt'
```
8.打开`/etc/postfix/master.cf`，开启smtp(465)和提交(587)：

```c
-o syslog_name=postfix/submission
  -o smtpd_tls_security_level=encrypt
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
smtps     inet  n       -       n       -       -       smtpd
  -o syslog_name=postfix/smtps
  -o smtpd_tls_wrappermode=yes
  -o smtpd_sasl_auth_enable=yes
  -o smtpd_relay_restrictions=permit_sasl_authenticated,reject
  -o milter_macro_daemon_name=ORIGINATING
```
![Alt text](/img/mailserver/1479367056070.png)

9.安装Dovecot SASL:

```c
root@mail:~$ sudo apt-get install dovecot-common
```
在安装过程中做出如下选择：

```c
1.yes
2.mail.****.xyz
```
10.更改`/etc/dovecot/conf.d/10-master.conf `文件找出其中95行的`#Postfix smtp-auth`：

```c
# Postfix smtp-auth
unix_listener /var/spool/postfix/private/auth {
mode = 0660
user = postfix
group = postfix
}
```
11.打开`/etc/dovecot/conf.d/10-auth.conf`文件，找到100行：

```c
auth_mechanisms = plain
```
更改成：

```c
auth_mechanisms = plain login
```
12.重启postfix和dovecot服务：

```c
root@mail:~$ sudo service postfix restart
root@mail:~$ sudo service dovecot restart
```
13.测试`SMTP-AUTH`和smtp/pop3的端口连接情况：

```c
root@mail:~# nmap ****.xyz

Starting Nmap 6.40 ( http://nmap.org ) at 2016-11-17 02:26 EST
Nmap scan report for ****.xyz (10*.224.***.***)
Host is up (0.000025s latency).
rDNS record for 10*.224.***.***: mail.ussec.xyz
Not shown: 993 closed ports
PORT    STATE SERVICE
25/tcp  open  smtp
80/tcp  open  http
110/tcp open  pop3
143/tcp open  imap
587/tcp open  submission
993/tcp open  imaps
995/tcp open  pop3s

Nmap done: 1 IP address (1 host up) scanned in 0.21 seconds
```
开启成功就做telnet测试：

```c
root@mail:~# telnet mail.ussec.xyz smtp
Trying 10*.224.***.***...
Connected to mail.****.xyz.
Escape character is '^]'.
220 mail ESMTP Postfix (Ubuntu)
ehlo hello     //手动输入ehlo hello
250-mail
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-AUTH PLAIN LOGIN
250-AUTH=PLAIN LOGIN
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
```

```c
root@mail:~# telnet mail.****.xyz smtp 
Trying 10*.224.***.***...
Connected to mail.****.xyz.
Escape character is '^]'.
220 mail ESMTP Postfix (Ubuntu)
mail from:root@****.xyz
250 2.1.0 Ok
rcpt to:bobby@****.xyz
250 2.1.5 Ok
data
354 End data with <CR><LF>.<CR><LF>
Test
.    
```
通过`telnet mail.****.xyz 587`测试也是可以的

安装配置dovecot
-------
14.安装dovecot ：

```c
root@mail:~$ sudo apt-get install dovecot-imapd dovecot-pop3d
```
15.现在配置mailbox。打开`/etc/dovecot/conf.d/10-mail.conf`，找到30行：

```c
mail_location = mbox:~/mail:INBOX=/var/mail/%u
```
更改成：

```c
mail_location = maildir:~/Maildir
```

16.更改`pop3_uidl_format`。打开` /etc/dovecot/conf.d/20-pop3.conf`文件，在50行处找到，去除注释：

```c
pop3_uidl_format = %08Xu%08Xv
```
17.打开SSL。在`/etc/dovecot/conf.d/10-ssl.conf`找到第6行处，去除注释：

```c
ssl=yes
```
18.重启dovecot service：

```c
root@mail:~$ sudo service dovecot restart
```
19.用telnet测试pop3和imap端口活跃情况：

```c
root@mail:~# telnet mail.****.xyz 110
Trying 10*.224.***.***...
Connected to mail.****.xyz.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
user bobby
+OK
pass bobby
+OK Logged in.
retr
-ERR There's no message 0.

```
可以在995、993、143端口重复同样的操作。
或者是通过以下命令查看对应的端口开启情况：

```c
root@mail:~# netstat -nl4

```

![Alt text](/img/mailserver/1479368979694.png)

20.创建用户，检查邮件客户端的使用情况：

```c
root@mail:~# sudo useradd -m tf -s /sbin/nologin
root@mail:~# sudo passwd tf
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully

//添加用户到root组
root@mail:~# sudo usermod -G root tf

```

> Squirrelmail的用户是使用的服务器的用户

安装、配置Squirremail
-------------
21.在安装了apache和php的环境下，在安装Squirrelmail：

```c
root@mail:~$ sudo apt-get install squirrelmail
```

22.配置squirrelmail：

```c
root@mail:~$ sudo squirrelmail-configure
```
我们只用选择配置选项更改就行：
选择1(Organization Preferences)->继续选择1(Organization Name)->Organization Name->选择S->选择Q退出
23.配置apache开启squirrelmail:

```c
root@mail:~$ sudo cp /etc/squirrelmail/apache.conf /etc/apache2/sites-available/squirrelmail.conf
root@mail:~$ sudo a2ensite squirrelmail
```
24.重启Apache服务：

```c
root@mail:~$ sudo service apache2 restart
```
25.现在在浏览器可以打开`http://serverIP/squirrelmail`，就可以正常使用：
![Alt text](/img/mailserver/1479370058388.png)

![Alt text](/img/mailserver/1479369799046.png)

![Alt text](/img/mailserver/1479370028649.png)

参考资料：

[[1].https://help.ubuntu.com/community/PostfixAmavisNew](https://help.ubuntu.com/community/PostfixAmavisNew)


