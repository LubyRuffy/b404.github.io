---
title: 'powershell备忘单'
layout: post
tags: 
  - windows
  - hack
  - command
  - powershell
category: 
  - hack
  - sec
  - windows
comments: true
share: true
description: powershell备忘单
---

* TOC
{:toc}

powershell备忘单

<!--more-->



## 常用命令

### 获取目录

ls、dir、cgi

```cpp
Get-ChildItem
```

![](/img/hack/powershell/1507514677357.png)

### 复制文件

cp、 copy、cpi

```cpp
Copy-Item src.txt dst.txt
```

![](/img/hack/powershell/1507514962920.png)

### 移动文件

mv、move、mi

```cpp
Move-Item src.txt dst.txt
```

![](/img/hack/powershell/1507515217663.png)


### 在文本中查找字符串

```
 Select-String -path c:\users\b404\Desktop\*.txt -pattern password
```

![](/img/hack/powershell/1507515507501.png)

### 显示文件内容

cat、type、gc

```
Get-Content file.txt
```

![](/img/hack/powershell/1507516025588.png)


### 显示当前路径

pwd、gl

```cpp
Get-Location
```

![](/img/hack/powershell/1507516143288.png)

### 显示当前进程

ps、gps

```cpp
Get-Process
```

![](/img/hack/powershell/1507516303651.png)


### 显示当前服务

```cpp
Get-Service
```

![](/img/hack/powershell/1507516386369.png)


### 格式化输出命令

```cpp
ls | Format-List -property name
```

![](/img/hack/powershell/1507516556753.png)


### 按页输出

```cpp
ls -r | Out-Host -paging
```

![](/img/hack/powershell/1507516928586.png)

## 渗透测试常用

###  ping扫描

```cpp
 1..255 | % {echo "192.168.222.$_";ping -n 1 -w 100 192.168.222.$_ | Select-String ttl}
```

![](/img/hack/powershell/1507518995172.png)


### 端口扫描

```cpp
1..1024 | % {echo ((new-object Net.Sockets.TcpClient).Connect("192.168.222.138",$_)) "Port $_
is open!"} 2>$null
```

![](/img/hack/powershell/1507520110439.png)


### 下载文件

实现wget功能

```cpp
(New-Object System.Net.WebClient).Downloadfile("http://127.0.0.1/calc.exe","calc1.exe")
```

![](/img/hack/powershell/1507532353210.png)

### 查找指定名称所有的文件

```cpp
 
Get-ChildItem "C:\Users\" -recurse -include *2*.txt
```

![](/img/hack/powershell/1507534064256.png)

### 查看补丁

```cpp
Get-HotFix
```

![](/img/hack/powershell/1507534179813.png)

### 列出注册表中的自启动项

```cpp
Get-ItemProperty HKLM:\SOFTWARE\Microsoft\Windows\CurrentVersion\run
```

![](/img/hack/powershell/1507534688373.png)

### 将ASCII字符串转换为Base64编码

```cpp
[System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("PSFuck!"))
```

![](/img/hack/powershell/1507534899840.png)

### 获得防火墙规则

```cpp
Get-NetfirewallRule -all

New-NetFirewallRule -Action Allow -DisplayName LetMeIn -RemoteAddress 192.168.222.145
```

![](/img/hack/powershell/1507535143925.png)

![](/img/hack/powershell/1507535250641.png)


### 操作注册表

```cpp
PS C:\Users\Administrator> cd HKLM:\
PS HKLM:\> ls
```

![](/img/hack/powershell/1507535372987.png)


## Cmdlet

cmdlet是command-let的缩写，实际上相当于命令提示符窗口的命令行，看似很长的cmdlet，实际上所有cmdlet都以标准的“动词-名词”格式命名的，如Get-Command命令，就是获得(动词)—命令(名词)，输入该命令回车后，Windows PowerShell会显示所有的命令:

- Get-Command:用于检索所有可用cmdlet的列表。
- Get-Help:用于显示有关cmdlet和概念的帮助信息。
- Get-WMIObject:用于通过WMI来检索管理信息。
- Get-EventLog:用于检索Windows事件日志。
- Get-Process:用于检索单个活动进程或活动进程的列表。
- Get-Service:用于检索 Windows 服务。
- Get-Content:用于读入文本文件，将每行视为一个子对象。
- Add-Content:用于将内容附加到文本文件。
- Copy-Item:用于复制文件、文件夹和其他对象。
- Get-Acl:用于检索访问控制列表(ACL)。

### Cmdlet Aliadses——长命令的简写

```cpp
Get-Alias

alias gcm
```

![](/img/hack/powershell/1507536354752.png)

### 获取帮助

```cpp
Get-Help
```

![](/img/hack/powershell/1507536651104.png)

### 查找Cmdlets

```cpp
Get-Command
```

![](/img/hack/powershell/1507536737678.png)

```cpp
Get-Command Set*
或
Get-Command -Verb Set
```

![](/img/hack/powershell/1507536812946.png)


```cpp
Get-Command *Process
```

![](/img/hack/powershell/1507536924143.png)

### 简化

输入小写的cmdlet之后，可以按TAB键转化为大写

`ls -recurse`和`ls -r`一样

### 5条基本的PS命令

5条基本的PS命令：

![](/img/hack/powershell/1507537270702.png)





## 资源

- http://www.pstips.net/powershell-executing-external-commands.html