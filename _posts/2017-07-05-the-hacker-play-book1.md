---
title: 渗透测试实用指南
layout: post
tags:
  - sec
  - tools
  - readingnotes
category: 
  - hack
  - tools
  - readingnotes
comments: true
share: true
description: 这本书很不错，奇淫技巧很多。虽然英文版的这本书有些错误，但是中文版的翻译者简直是脑残，不修订错误直接照搬，估计译者都不知道命令参数的正确格式，连里面真正讲了些什么都不知道吧...翻译得也是狗屎一样，连中文都那么多病句。
---

这本书很不错，奇淫技巧很多。虽然英文版的这本书有些错误，但是中文版的翻译者简直是脑残，不修订错误直接照搬，估计译者都不知道命令参数的正确格式，连里面真正讲了些什么都不知道吧...翻译得也是狗屎一样，连中文都那么多病句。

* TOC
{:toc}

<!--more-->

# 扫描网络

扫描分为外部扫描、内部扫描、WEB应用程序扫描。扫描可以掌握目标弱点，熟悉目标的网络环境。

## 外部扫描

### 被动式信息收集

在不接触目标的情况下收集与测试目标相关的信息，收集目标、网络、客户端以及其他信息。

## Discover Scripts

[Discover Scripts](https://github.com/leebaird/discover.git
)是一个多功能的信息发掘框架，可用被动方式有效、快速地查找目标相关信息。

## Recon-ng

[Recon-NG](https://bitbucket.org/LaNMaSteR53/recon-ng)是由Python编写的一个开源的Web侦查（信息收集）框架，使用它可以自动的收集信息和网络侦查。

## 外部或内部的主动式信息收集

主动式信息收集就是通过主动扫描确认目标安装的操作系统和网络服务，并发现潜在漏洞的过程。即主动式信息收集必定对指定的网络段进行扫描。

### 网络扫描的流程

- 用Nexpose/Nessus扫描
- 用Nmap扫描
- 用自定义的Nmap扫描
- 用PeepingTom抓取屏幕

#### Nessus

![Nessus](/img/the_hacker_play_book/Nessus.png)

#### Nmap

扫描：

```css
root@kali:/usr/share/nmap/scripts# nmap --script banner-plus.nse --min-rate=400 --min-parallelism=512 -p1-65535 -n -Pn -PS -oA /opt/peepingtom/report 10.10.10.1/24
```

> `--script`设置脚本
> `min-rate`保证扫描在一定时间结束
> `min-parallelism`设置并行扫描数量
> `-p`设置端口
> `-n`禁用DNS解析
> `-Pn`禁用ping
> `-PS`使用TCP SYN ping的方式进行主机探测
> `oA`导出所有类型的报告


#### PeepingTOM:揭示网络构造

下载：

```css
root@kali:/opt# wget http://thehackerplaybook.com/Download/peepingtom.zip
```

Gnmap.pl可将Nmap结果处理为PeepingTom所需的IP列表：

```css
root@kali:~# sudo cat report.gnmap|perl ./gnmap.pl|grep http|cut -f 1,2 -d ","|tr "," ":" > http_ips.txt
```

通过peepingtom的处理:

```css
root@kali:~# python peepingtom.py -p -i http_ips.txt
```

## Web应用程序扫描

进行Web扫描，可选择WebScarab、Nikto、ZAP、W3af等工具

### Web应用程序的扫描流程

- Burpsuit进行spider、disvocer、scanning
- 用Web扫描器扫描
- 手动注入参数
- Session Token分析

### Web扫描工具

在Nessus/Nexpose找到操作系统、应用程序、网络服务的漏洞之后，可以通过Burp suit深入Web应用程序测试：

- 设置代理
- 开启Burpsuit
- 对应用进行爬取
- 对内容进行挖掘
- 进行主动扫描
- 漏洞利用


# 漏洞利用

Metasploit框架可用于开发exploit、利用漏洞以及实现辅助攻击。


## 配置Metasploit进行远程攻击的基本步骤

- 选择要使用的攻击利用代码或者模块
- 设置该模块参数
 - set命令可配置模块的输入变量
 - 设置目标主机和端口
 - 设置本地主机和端口
 - 若有详细信息，可设置系统版本、用户账号以及其他信息
 - 运行show options查看需要设置的其他选项
- 配置有效载荷
 - 载荷是在利用漏洞之后运行的实施具体操纵行为的程序
 - 运行show payloads查看payload的全部类型
 - 使用set payloads设置要使用的载荷
- 设置编码器(encoder)
 - 隐匿攻击程序特征
 - 运行show encoders命令查看封装载荷的各种编码器，使用set encoders对payload进行封装
- 设置其他选项
- exploit


# 人工检测WEB应用程序

## SQL注入

常用的SQL注入工具是SQLmap和SQLninja。

### GET参数注入

- 验证某个SQL注入是否有效

```css
sqlmap -u "http://site.com/info.php?user=test&pass=test" -b
```

- 获取数据库用户名

```css
sqlmap -u "http://site.com/info.php?user=test&pass=test" -current-user
```
- 交互shell

```css
sqlmap -u "http://site.com/info.php?user=test&pass=test" --os-shell
```

**技巧提示：**

- 可能需要指定要攻击的数据库的具体类型。若认为可能存在注入，而SQLmap又没什么发现，就试试--dbms-[database]
- 若目标网站要求用户进行登录认证，可通过浏览器登录到网站，然后获取cookie信息(burpsuit提取)。最后用--data=[cookie]选项定义cookie。
- 卡住了，就试试sqlmap --wizard

### POST参数注入

POST方式不是通过URL传递参数，而是在递交的数据里传递参数。通常传递用户名和密码。

- 验证某个SQL注入是否有效

```css
sqlmap -u "http://site.com/info.php" --data="user=test&pass=test" -b
```

- 获取数据库用户名

```css
sqlmap -u "http://site.com/info.php" --data="user=test&pass=test" --current-user
```

- 交互shell:

```css
sqlmap -u "http://site.com/info.php" --data="user=test&pass=test" --os-shell
```

## XSS

XSS是一种由于Web应用程序缺乏输入验证而引起的针对网站访问客户的攻击，分为反射型和存储型。攻击的本质就是攻击人员向网站访问客户的浏览器里注入了某种代码。

### BeFF

![BeFF](/img/the_hacker_play_book/beff.png)


# 渗透内网

## 无登陆凭据条件下的网络渗透

在连入网络，但没有任何登陆凭据的时候，使用tcpdump程序进行动态的监听，进而分析网络结构，寻找域控制器，然后进行其他类型的被动攻击。

### responder.py

[responder.py](https://github.com/SpiderLabs/Responder)可在一无所知的情况下取得登陆凭据，可率先实现监听、响应LLMNR协议(本地链路多播名称解析协议,Link Local Multicast Name Resolution)和NBT-NS协议(网络基本输入输出系统域名服务协议)。还可以利用WPAD(Web Proxy Auto-Discovery)漏洞（浏览器设置为自动检测设置，那么受害机会从网络获取配置文件）

该情况下，与受害机处于同一个网络的攻击人员，能够回复受害主机发出的名称解析请求，并注入自己的PAC文件，代理所有Web流量，强制浏览器用户对攻击人员的SMB服务器进行身份验证，就能获取该主机的哈希值。

## 利用任意域凭据(非管理员权限)

### 组策略首选项

通过GPP(Group Policy Preference，组策略首选项)发布的账号全部存储在`\\[Domain Controller]\SYSVOL\[Domain]\Policies`中。可在该文件夹中找到一个名为Group.xmls的文件，找到之后就可以找到cpassword的哈希值。

**只要获取域账号使用权限，就应该率先检查组策略首选项**

### 获取明文凭据

WCE和Mimikatz能够从内存中读取明文密码。（管理员权限）

### 漏洞利用后期

[后期渗透思路](https://room362.com/)：https://room362.com/

## 利用本地或域管理账号

### 使用登陆凭证和PSExec掌控网络

PSExec可以使用用户名和密码，也可以使用用户名和密码的hash值登陆。

使用系统漏洞，规避杀软，攻击目标并登陆：

- 使用veil生成规避杀软的payload
- 使用msf中的PSExec模块，并设置相关参数(和veil的一样)
- 然后使用`set EXE::Custom+payload所在路径`使用veil创建的payload
- migrate[x86_64进程的PID]
- `getsystem`获取系统权限
- load mimikatz
- kerberos/wdigest
- 进行域相关操作
 - net group "Domain Admin" /domain(列出域管理员)
 - qwinsta(列出用户会话)
 - 新建管理账户
- 获取域控密码

>　Mimikatz只在内存中运行。若被攻陷的主机是64位，**首先将程序进程迁移（ｍｉｇｒａｔｅ）到一个６４位的进程中，才能查看密码。若是32位就没有限制。**


### 攻击域控制器

域控制器把用户的hash存储在C:\Windows\NTDS\ntds.dit`中，DC运行的活动目录操作会一直访问它，该文件被施加了读取锁，但可使用Windows自带的Shadow Copy创建一份复制。

[SMBExec](https://github.com/brav0hax/smbexec)是攻击DC的好选择，可以使用Windows的Shadow Copy功能复制ntds.dit文件并获取注册表中的SYS键值。

## 漏洞利用后期——使用PowerSploit(Windows)

[PowerSploit](https://github.com/PowerShellMafia/PowerSploit),使用Windows的原生工具，可以在内存中执行所有程序，能够规避杀软，可以将DLL注入到进程中，还可以使用目标电脑里缓存的Windows认证信息访问整个域。

若拥有本地管理员凭据，而且登陆到目标的单位中，就可以使用Powershell强制用户下载并调用PowerShell脚本，最终获得创建Meterpreter反向连接的Shell。

[Invoke-ShellCode](https://github.com/PowerShellMafia/PowerSploit/tree/master/CodeExecution/Invoke-Shellcode.ps1)程序能在客户端创建Meterpreter反射型HTTPS的shell全部功能。

下载以下三个工具：

```css
git clone https://github.com/PowerShellMafia/PowerSploit
```

```css
wget https://raw.githubusercontent.com/obscuresec/random/master/StartListener.py
```

```css
wget https://raw.githubusercontent.com/darkoperator/powershell_scripts/master/ps_encoder.py
```

[具体实际例子，已验证是对的](https://xianzhi.aliyun.com/forum/read/723.html)：https://xianzhi.aliyun.com/forum/read/723.html


## ARP欺骗

ARP欺骗的工具:

- [Cain&Abel](http://www.oxid.it/cain.html)
- [Ettercap](https://pentestmag.com/ettercap-tutorial-for-windows/)
- [evilfoca](https://github.com/ElevenPaths/EvilFOCA)

## 会话劫持

在成功对目标主机进行ARP欺骗之后，可以对访问目标的内容、使用的协议、密码会话进行控制。

通过劫持会话的cookie，导入浏览器冒充合法用户。

### Hamster/Ferret

Hamster可实现会话劫持，它以代理服务器的模式工作，将他人的session cookie替换为自己的实现会话攻击。

### firesheep

firesheep是火狐的一款插件，可以嗅探通过无线、有线网洛中以明文传输的会话令牌。

## DNS重定向

对内部网络进行中间人攻击，克隆其网站可以获得大量有用信息。

### Cain&Abel

Sniffer->APR标签->单击APR-DNS->右键添加需要更改的DNS请求。

此时可以使用SET工具克隆一个谷歌网站或者是其他的网站，配置好DNS欺骗，建立好仿冒网站，当被ARP欺骗的目标用户访问时候就重定向到SET克隆的仿冒页面。

### SSLStrip

SSLStrip可以将用户从HTTPS页面重定向到HTTP站点，以截获明文方式传递的所有浏览器流量(监控HTTPS流量，并修改用户所有的HTTPS通信为HTTP)

## 端口代理

当连入网络，无法访问特定的网段时候，让具有访问权限或适当IP地址的用户充当代理。

netsh是Windows自带的修改网络配置的命令行工具。(管理员权限)

将10.10.10.128的3389端口转发到10.10.10.136的8080端口：

```css
netsh interface portproxy add v4tov4 listenport=8080 listenaddress=10.10.10.136 connectport=3389 connect=address=10.10.10.128
```

# 社工

可看https://www.wired.com/2011/09/doppelganger-domains

## 邮件中间人攻击

通过购买与目标近似的域名，然后搭建SMTP邮件服务器，并将服务器设置为catch-all——接收所有邮件。然后将接收的邮件接收后，伪造之后发往正确的邮箱，通过通信双方的信任达到邮件中间人攻击。也可以在该域名上配置ssh服务器，配置日志文件，然后查看目标用户输入的用户名和密码。

## 鱼叉式钓鱼攻击

Metasploit Pro具有一个辅助性的社工模块，创建一个页面，从而直接获得用户名和密码以及其他信息。

## SET

- 伪造页面

- 使用SET中的Java程序进行攻击

- 大规模鱼叉式钓鱼
 - 伪造smtp服务器群发邮件
 - SET群发邮件
- 发送附带powershell的Execl


# 物理访问的攻击

## 无线攻击

### 被动识别和侦查

被动WIFI测试就是将无线网卡设置为嗅探模式，继而识别接入点、客户端、信号强度、加密类型等各类信息。主机在被动模式下，不能向任何设备发送数据。

**kismet可用于嗅探、识别并监控无限流量，其将会列出所有的SSID、信道、信号强度等无线信息。（卡顿的时候按tab键进行切换）**

- 黄色为WIFI未加密
- 红色为WIFI使用出厂设置
- 绿色的为WIFI采用了认证机制(WEP、WPA等)
- 蓝色的为WIFI隐藏了SSID（禁用SSID广播）

选定一个SSID，将可以快速查看AP有关信息，例如BSSID、制造商、加密类型、信号强度/丢包率等，有助于判断AP位置。按下`~`、`V`、`C`键将看到连接到该AP的所有客户端。

客户端信息是解除认证和拒绝服务攻击所需的重要信息。

### 主动攻击

对无线网络进行侦查之后，尽快破解无线密码或尽可能访问无线基础设施。

#### WEP有线等效加密

WEP无线网络的安全机制不够安全。若被测单位使用WEP网络，且有客户端接入，那就可以轻松破解除WEP密码。

破解步骤：

- 命令行输入`fern-wifi-cracker`
- 下拉选中网卡
- 单击scan
- 选择WEP
- 选择想要攻击的SSID
- 单击右侧的WIFI Attack
- 观察IV计数器(初始化矢量计数器)。至少需要10k个IV才能破解
- 破解成功就会显示出密码

#### WPAv2(TKIP)（WIFI访问保护）

要攻击WPAv2的无线网络，需要抓取从客户端发送给AP的验证握手包。抓取该数据包需要采取欺骗手段，强制用户解除认证，然后重新认证，抓取到握手的认证数据包之后，不能立即提取出密码，必须暴力破解。

破解步骤：

- 命令行输入`fern-WiFi-cracker`
- 进入工具盒
- 单击WiFi Attack options
- 选择capture file settings
- 按ESC退回到fern-wifi-cracker的主屏幕
- 下拉选中网卡
- 单击scan
- 选中WPA（蓝色信号）
- 选择要攻击的SSID
- 单击Wifi按钮
- 使用wpaclean整理抓取的数据包(`wpaclean <out.cap><in.cap>`)
- 通过aircrack-ng将整理好的数据包转换为hccap(`aircrack-ng <out.cap> -J <out.hccap>`)
- 使用oclHashcat破解转换后的文件。

#### WPAv2 WPS（Wifi配置保护）攻击

WPS最初叫做WiFi Simple Config，主要致力于创建无线路由器/AP的安全连接。当客户端连接到AP时，只要输入较短的WPS PIN码就行，这就可以进行暴力破解。部分具有WPS功能的无线AP，即使在配置页面关闭了WPS功能，WPS功能依然存在。

攻击WPS步骤与WPAv2的步骤类似，仅仅在于选择WPS_Attack不同于WPAv2的Regualar Attack。

#### WPA企业级——伪造Radius攻击

通过伪造Radius攻击企业级的Wifi。利用Radius服务器在WPAv2企业级无线环境下抓取用户名和密码。


#### karmetasploit

karma可以辅助攻击人员识别出无线用户端使用的SSID，并且可以模仿这些SSID的接入点让目标主机连接到攻击人员控制的伪AP。Karmetasploit对这款程序进行改变，让攻击人员使用metasploit攻击连入的客户端。


## 物理攻击

### 克隆工卡

- ProxMark3:克隆RFID
- ProxBrute:暴力破解
- RFIDiot:RFID克隆和脚本
- Tastic RFID:远距离克隆RFID卡

### Kali NetHunter


# 规避杀软

WCE能够提取内存中的密码，但很多杀软都能检测出，最简单易行的就是在WCE文件中找到AV匹配的签名，然后更改签名。

使用文件分割器分隔执行文件，然后使用AV从最小的分割部分扫描，直到AV报警为止。当AV报警时，就可以判断在上一个文件的结束地址到报警文件结束地址之间有AV检测的签名特征，修改该部分签名即可。


# 破解、利用、技巧

## 破解

### 字典

- [Rockyou](https://downloads.skullsecurity.org/passwords/passwords/rockyou.txt.bz2):https://downloads.skullsecurity.org/passwords/passwords/rockyou.txt.bz2
- [Crackstation-hunman-only](https://download.g0tmi1k.com/wordlists/large/):https://download.g0tmi1k.com/wordlists/large/
- [m3g9tr0n wordlist](http://derv.us/wordlists.html)

### 破解

- john the ripper
- oclhashcat

## 漏洞搜索

首先使用nmap对目标进行banner分析，然后根据banner对其进行漏洞搜索。

### Searchsploit

直接输入searchsploit搜索漏洞

### BugTraq

[bugtraq](http://www.securityfocus.com/bid)是一个出色的漏洞和exploit数据源。

### ExploitDB

## 技巧

- 通过编写rc脚本提高msf的使用效率
- 限制IP攻击时，通过购买类似域名，将其克隆成相关网站，上线几个星期之后，目标以为域名被公司购买，有可能会解除该域名IP的限制，到时就可以继续做下一步
- 绕过UAC，在meterpreter中使用getsystem不能提权。可以使用`run bypassuac`，其会生成另一个会话，之前打开的为了防止关闭使用`background`，然后`session -i 2`切换到新生成的会话
- 拥有Webshell之后，可以考虑将命令写入txt中，使用`ftp -s:ftp.txt`执行

### 隐藏文件

ADS:Alternate Data Streams,备用数据流。[它是Windows NTFS文件系统上一个鲜为人知的功能，能在现有的文件和文件夹中追加数据，而且不影响文件的功能和大小。](http://www.rootkitanalytics.com/userland/Exploring-Alternate-Data-Streams.php).

通过ADS可以在常被管理员查看的目标机器隐藏恶意文件。


### 稳定隐藏文件

通过`C:\>mkdir \\?\c:\tmp\".. \"`创建有问题的文件夹，将隐藏文件移动到其中，资源管理器是无法删除和修改、运行该文件。

### 上传文件

使用powershell或者是bitsadmin(windows更新程序就是它下载的)上传。






















