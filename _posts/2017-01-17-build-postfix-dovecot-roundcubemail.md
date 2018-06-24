---
title: '在Ubuntu14.04上建立roundcubemail邮件服务器'
layout: post
tags: 
  - postfix
  - dovecot
  - ubuntu
  - roundcubemail
  - mailserver
  - environment
category: 
  - mailserver
  - environment
comments: true
share: true
description: 在Ubuntu14.04上基于Postfix-dovecot建立roundcubemail邮件服务器
---
在Ubuntu14.04上基于Postfix-dovecot建立roundcubemail邮件服务器

<!--more-->

* TOC
{:toc}


# 在Ubuntu14.04上建立roundcubemail邮件服务器(Postfix-dovecot)

@(环境搭建)



> Postfix(smtp,发送)

> Dovecot(pop3,imap,接收)

> Roundcubemail(用于Webmail连接)
 
安装和配置postfix
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
![Alt text](/img/mailserver/roundcube/1479370288872.png)

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

![Alt text](/img/mailserver/roundcube/1479367056070.png)

> 一定要取消submission和smtps的注释

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
rDNS record for 10*.224.17*.***: mail.****.xyz
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
root@mail:~# telnet mail.****.xyz smtp
Trying 10*.224.1**.***...
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
Trying 10*.224.1**.***...
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
Trying 10*.224.1**.***...
Connected to mail.****.xyz.
Escape character is '^]'.
+OK Dovecot (Ubuntu) ready.
user tf
+OK
pass tf
+OK Logged in.
retr
-ERR There's no message 0.

```
可以在995、993、143端口重复同样的操作。
或者是通过以下命令查看对应的端口开启情况：

```c
root@mail:~# netstat -nl4
```
![Alt text](/img/mailserver/roundcube/1479368979694.png)

20.创建用户，检查邮件客户端的使用情况：
![Alt text](/img/mailserver/roundcube/1481496493360.png)

```c
root@mail:~# sudo useradd -m tf -s /sbin/nologin
root@mail:~# sudo passwd tf
Enter new UNIX password: 
Retype new UNIX password: 
passwd: password updated successfully

//添加用户到root组
root@mail:~# sudo usermod -G root tf

```

> Roundcubemail的用户是使用的服务器的用户

安装、配置Roundcubemail
-------------
21.安装LAMP(Linux、Apache、MySQL、PHP)环境：

```c
sudo apt-get install lamp-server^
```

22.登陆mysql:

```c
b404@ubuntu:~$ mysql -u root -p
Enter password: 
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 42
Server version: 5.5.53-0ubuntu0.14.04.1 (Ubuntu)
Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.
Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.
Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql> 

```

创建roundcubemail的数据库：

```c
mysql> create database roundcubedb;
Query OK, 1 row affected (0.00 sec)
```

创建roundcubedb的用户名和密码：

```c
mysql> create user 'tf' identified by 'tf';
Query OK, 0 rows affected (0.00 sec)
```

授予用户'tf'权限连接'roundcubemail'的数据库：

```c
mysql> grant all privileges on roundcubedb.* to 'tf';
Query OK, 0 rows affected (0.00 sec)
```

刷新权限：

```c
mysql> flush privileges;
Query OK, 0 rows affected (0.00 sec)
```

23.下载解压roundcubemail:

```c
wget https://github.com/roundcube/roundcubemail/releases/download/1.2.0/roundcubemail-1.2.0-complete.tar.gz
```

解压到`/var/www/html`文件夹：

```c
sudo tar -xzvf roundcoubemail-1.2.0-complete.tar.gz
```

24.改变权限，将“/ var / www / webmail”的所有权更改为用户和web服务器组的“www-data”权限：

```c
sudo chown -R www-data:www-data /var/www/webmail/*

sudo chown -R www-data:www-data /var/www/webmail/
```

25.导入roundcubemail的数据库到MySQL中：

```c
mysql -u root -p roundcubedb < /var/www/html/SQL/mysql.initial.sql
```

![Alt text](/img/mailserver/roundcube/1481500330801.png)

26.打开浏览器，按照以下格式输入：

```c
http://localhost/installer/
```
![Alt text](/img/mailserver/roundcube/1481500291068.png)
基本环境ok就直接next。

27.配置roundcubemail:

输入数据库名称和用户名、密码：
![Alt text](/img/mailserver/roundcube/1481500475878.png)

这一行必须勾上：
![Alt text](/img/mailserver/roundcube/1481500595577.png)

直接next到第三步,测试smtp和imap连接情况：

![Alt text](/img/mailserver/roundcube/1481500714430.png)
![Alt text](/img/mailserver/roundcube/1481500850677.png)

28.删除安装程序`installer`

```c
rm -rf installer
```

29.测试安装是否成功：

![Alt text](/img/mailserver/roundcube/1481501948978.png)
![Alt text](/img/mailserver/roundcube/1481500950761.png)

由于roundcubemail做了字符限制，不能发送到`tf@localhost`，在命令行尝试了发送到`tf@localhost`账户的：

![Alt text](/img/mailserver/roundcube/1481502260821.png)
![Alt text](/img/mailserver/roundcube/1481502082475.png)

