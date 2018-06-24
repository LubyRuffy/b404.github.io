---
title: Hack Tools
layout: post
tags:
  - sec
  - tools
  - 
category: 
  - hack
  - tools
comments: true
share: true
---

对一些安全工具的总结

* TOC
{:toc}

<!--more-->

# 信息收集

## HTTrack：网站复制器

HTTrack能够创建与目标网站完全相同的脱机副本。复制的内容包括原始网站所有的网页、链接、图片和代码。

**安装：**

```css
root@kali:~# apt-get install webhttrack
```

**使用：**

```css
oot@kali:~# httrack --help

HTTrack version 3.48-24
    usage: httrack <URLs> [-option] [+<URL_FILTER>] [-<URL_FILTER>] [+<mime:MIME_FILTER>] [-<mime:MIME_FILTER>]
    with options listed below: (* is the default value)
```

![httrack](/img/tools/httrack.png)


## The Harvester:挖掘、利用邮箱地址

可用于搜索Google、Bing和PGP服务器的电子邮件、主机、以及子域名，搜索LinkedIn的用户名。

**安装：**

```css
root@kali:~# git clone https://github.com/laramies/theHarvester.git
```

**使用：**

```css
root@kali:~/theHarvester# python theHarvester.py 
```

![the harvester](/img/tools/theharvester.png)

## whois：查域名信息

可以获取目标相关的具体信息，包括IP地址或公司DNS主机名以及地址和电话号码等联系信息

**使用:**

```css
whois [OPTION]... OBJECT...
```

![whois](/img/tools/whois.png)


## netcraft:搜索域名信息

Netcraft能搜索到搜索关键字的相关网站，例如目标站的IP地址、Web服务器的操作系统、DNS服务

![netcraft](/img/tools/netcraft.png)
 
> 
## host：解析地址

host可以"翻译"IP地址等。

![host](/img/tools/host.png)

**例子：**

```css
root@kali:~# host ns1.dreamhost.com
ns1.dreamhost.com has address 64.90.62.230
root@kali:~# host 64.90.62.230
230.62.90.64.in-addr.arpa domain name pointer ns1.dreamhost.com.
root@kali:~# host -a baidu.com
Trying "baidu.com"
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 57234
;; flags: qr rd ra; QUERY: 1, ANSWER: 9, AUTHORITY: 5, ADDITIONAL: 5

;; QUESTION SECTION:
;baidu.com.         IN  ANY

;; ANSWER SECTION:
baidu.com.      5   IN  A   111.13.101.208
baidu.com.      5   IN  A   123.125.114.144
baidu.com.      5   IN  A   180.149.132.47
baidu.com.      5   IN  A   220.181.57.217
baidu.com.      5   IN  NS  ns2.baidu.com.
baidu.com.      5   IN  NS  ns3.baidu.com.
baidu.com.      5   IN  NS  ns4.baidu.com.
baidu.com.      5   IN  NS  ns7.baidu.com.
..
```

## DNS提取信息

DNS是将域名翻译成IP地址，DNS服务器包含其所记录的IP地址和对应域名想匹配的记录信息。许多网络部署了多台DNS服务器，保证冗余或负载均衡。多台DNS服务器之间信息共享是通过区域传输(AXFR)实现。

### NS LOOKUP

**使用：**
```css
root@kali:~# man nslookup
root@kali:~# nslookup
> server 66.33.206.206
Default server: 66.33.206.206
Address: 66.33.206.206#53
> set type=mx
```

![host和nslookup确定邮件服务器](/img/tools/host%E5%92%8Cnslooup%E7%A1%AE%E5%AE%9A%E7%9B%AE%E6%A0%87%E7%9A%84%E9%82%AE%E4%BB%B6%E6%9C%8D%E5%8A%A1%E5%99%A8.png)

### dig

通过dig将区域传输给192.168.1.23和域名为test.com的伪造的DNS服务器：

```css
root@kali:~# dig @192.168.1.23 test.com -t AXFR
```

## MetaGooFil：提取元数据

元数据：数据的数据

提取目标网站的元数据

**安装：**

```css
apt-get install metagoofil
```

**使用:**

![metagoofil](/img/tools/metagoofil.png)


# 扫描

扫描：在收集到信息与可攻击IP地址之间的映射关系之后，建立IP地址与开放端口和服务的映射关系。

扫描过程分为三个不同的阶段：

+ 验证系统是否正在运行
+ 扫描系统的端口及运行的服务
+ 扫描系统中的漏洞

常见的端口及其对应的服务

|端口号|服务|
|----|----|
|20|FTP数据传输|
|21|FTP控制|
|22|SSH|
|23|Telnet|
|25|SMTP(E-mail)|
|53|DNS|
|67、68|Bootp服务端和Bootp客户端|
|69|TFTP（简单文件传输协议）|
|79|Finger（在线用户、OS类型等用户信息）|
|80|HTTP|
|110|POP3(接受邮件)|
|135|RPC(远程过程调用)|
|137|NetBIOS名称服务|
|139|Windows的文件共享、Unix的Samba服务|
|143|IMAP|
|161|SNMP|
|443|HTTPS|
|1080|Socks|


## ping：判断主机存活

ping是ICMP数据包。ping命令发送的是ICMP回显请求数据包。

### fping

![fping](/img/tools/fping.jpg)

## Nmap:端口扫描

### TCP扫描

TCP连接扫描是端口扫描中最基础和稳定的，在每个端口上进行三次握手，之后又通过友好的方式断开连接，因此很少有机会对目标系统进行泛洪攻击导致崩溃。

![Nmap的TCP扫描](/img/tools/nmap_TCP%E6%89%AB%E6%8F%8F.jpg)

- `-sT`参数：TCP连接扫描;`-s`参数：指定扫描类型;`-T`：扫描类型为TCP
- `-p-`参数：扫描所有端口，不是只扫描默认的1000个端口
- `-PN`参数：目标对ping请求有响应，跳过主机发现阶段，对对所有地址进行扫描

### SYN扫描

SYN扫描（隐形扫描）是NMAP的默认扫描方式。SYN扫描比TCP连接扫描更加快更加安全，机会不造成DDOS。其只进行了三次握手的前两次。

![SYN扫描](/img/tools/nmap_SYN%E6%89%AB%E6%8F%8F.jpg)


### UDP扫描

TCP连接扫描和SYN扫描都是基于TCP通信，但是有些服务使用的是UDP协议通信，比如DHCP、DNS、SNMP、TFTP。

![UDP扫描](/img/tools/nmap_UDP.jpg)

UDP扫描非常慢，加参数V可返回有用的版本信息

### Xmas扫描（针对UNIX、Linux，可绕过ACL和过滤器）

TCP的RFC描述的是，若一个关闭的端口收到的数据包没有置位SYN、ACK、RST标记，该端口就会发送RST数据包作为响应。若开着的端口接收到这些数据包的时候就会忽略它们，不作响应。Xmas Tree扫描的数据包的FIN、PSH、URG标记置为`on`，扫描的时候，通过有无RST数据包判断端口是否关开。

![Xmas Tree扫描](/img/tools/nmap_Xmas.jpg)

### Null扫描(针对UNIX、Linux，可绕过ACL和过滤器)

NULL扫描也是和Xmas扫描一样根据RFC存在的弱点进行扫描，不过扫描与之相反，Null扫描使用没有任何标记的数据包。

![NULL扫描](/img/tools/nmap_NULL.jpg)

# 漏洞扫描

## Medusa

在线破解常用的是Medusa和Hydra。
Medusa被描述为通过并行登陆暴力破解的方式尝试获取远程验证服务访问权限的工具。能验证众多类型的远程服务，如FTP、HTTP、IMAP、MySQL、POP3等等。

```css
root@kali:/usr/share/wordlists# medusa -h 10.10.10.129 -u root -P test.lst -M ssh
```

![medusa](/img/tools/medusa.jpg)
![medusa例子](/img/tools/medusa_example.jpg)

## John the Ripper:密码破解之王

### unshadow

将passwd和shadow文件结合：

```css
root@kali:~# unshadow /etc/passwd /etc/shadow >linux_test.txt
```
 破解密码文件：

 ```css
root@kali:~# john linux_test.txt --show
root:toor:0:0:root:/root:/bin/bash

1 password hash cracked, 0 left
 ```

### mailer

发送密码破解的用户
 ![mailer](/img/tools/mailer.png)

### unique

 从密码字典中删除重复项

 ![unique](/img/tools/unique.png)

### unafs

 unafs用于向用户警告弱密码

 ```css
 unafs DATABASE-FILE CELL-NAME
 ```

## 密码重置

 通过密码重置获得系统访问权限或者提升权限

## 嗅探网路流量

嗅探，截获并查看进出某一网络的流量的过程。 通过嗅探获取系统访问权限

大部分网卡都运行在非混杂模式下，网卡接收到与本机地址相符的流量，就会把流量传递给CPU进行处理。
非混杂模式就是强制网卡接收流入的所有数据包，在混杂模式下，所有网络流量都会被传递给CPU进行处理，不管目的地址是否指向本机。

网卡在混杂模式下，流量地址未指向某计算机，如何到达计算机的情况：
- 广播流量会发送给网络中所有连接的设备；
- 网络使用集线器路由流量，不用交换机

## 泛洪攻击

大多的交换机保存包含MAC地址和端口号的匹配表的内存容量是有限的。通过耗尽内存，并用大量伪造的MAC地址泛洪该匹配表，就可让交换机无法读取或访问这个匹配表，这时候的交换机无法根据mac地址匹配正确端口，就会简单的把流量广播到所有端口。这种模式被称为“失效开放”。失效开放，就是当交换机无法正确区分地传送流量时，就会处于类似于集线器的开放状态，将所有流量发送至所有端口。

### macof

Dsniff中的macof可用来生成几千个随机MAC地址，对交换机进行泛洪攻击。（会产生大量网络流量，易被发现）

```css
macof -i eth0 -s 10.10.10.130 -d 10.10.10.129
```

> `-i` 指定计算机网卡(源MAC地址所在的网卡);`-s`指定来源地址；`-d`指定攻击对象

![macof](/img/tools/macof.jpg)

## Ettercap:中间人攻击

Ettercap的工作原理是诱骗客户端通过攻击者的计算机发送网络流量，这样就可从本地局域网中的计算机上获得用户名和密码。



## Nikto：扫描Web服务器

Nikto自动扫描Web服务器上过期的没有打补丁的软件，同时也检测驻留在服务器上的危险文件。

```css
root@kali:~# nikto -h 10.10.10.129 -p 1-1000
```

![nikto](/img/tools/nikto.jpg)


## WebScarab:网络爬虫

类似于burpsuit。
先在浏览器中设置代理，然后通过代理实现其相关功能，比如爬虫、拦截请求。

![webscarab](/img/tools/webscarab.jpg)

## Netcat:瑞士军刀

Netcat是一个允许通信和网络流量从一台计算机流向另一台计算机的工具，用于发送和接收TCP与UDP流量，既可以作为服务端也可以作为客户端。Netcat可以使用本地计算机上的任意一个端口连接远程目标计算机上的任意端口。其灵活性让它成为创建后门的最佳选择。可用于计算机之间传递文件、端口扫描、web服务器、即时聊天工具


### 聊天

监听端：

```css
root@kali:~# nc -l -p 2323
```

> `-l`设置监听端，`-p`设置端口

连接监听端：

```css
root@kali:~# nc 10.10.10.133 2323
```

![netcat通信](/img/tools/netcat_communication1.jpg)
![netcat通信](/img/tools/netcat_communication2.jpg)


### 端口扫描

```css
root@kali:~# nc -z -v -n 10.10.10.129 1-1000
```

![netcat扫描](/img/tools/netcat_scan.jpg)

> `-z`，连接成功之后立即关闭，不进行数据交换
> `-v`详细输出
> `-n`不使用DNS反向查询IP地址域名

netcat连接开放的21，并且打印该端口服务的banner:

```css
root@kali:~# nc -v 10.10.10.129 1-1000
```

![netcat连接端口打印服务](/img/tools/netcat_scan_no_connections.jpg)


### 其他功能

Netcat其他的功能可上网查询，比如[https://www.oschina.net/translate/linux-netcat-command](https://www.oschina.net/translate/linux-netcat-command)


## Cryptcat

Netcat的流量是明文的，Cryptcat使用[twofish](https://zh.wikipedia.org/zh-hans/%E5%8F%8C%E9%B1%BC%E7%AE%97%E6%B3%95)加密。

![cryptcat](/img/tools/cryptcat1.png)
![cryptcat](/img/tools/cryptcat2.png)

## hacker defender:rootkit

Hcaker Defender中有三个主要文件：hxdef100.exe、hexdef100.ini、bdcli100.exe(client)


## Veil

[Veil](https://github.com/Veil-Framework/Veil-Evasion)是一款利用Metasploit框架生成相兼容的Payload工具，并且在大多数网络环境中Veil 能绕过常见的杀毒软件，它会尽可能使每个payload文件随机.

![生成payload的设置情况](/img/tools/veil.png)

![生成payload成功](/img/tools/veil_results.png)
