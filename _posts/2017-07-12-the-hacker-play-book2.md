---
title: 渗透测试使用指南2
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
---


* TOC
{:toc}

<!--more-->

# 二进制利用

- 汇编、寄存器
- GDB基础知识（GNU调试器）
- 不同的内存段（堆、栈、数据、BSS、代码段）
- shellcode
- Hacking: The Art of Exploitation，2nd 
- The Shellcoders Handbook
- http://opensecuritytraining.info/IntroX86.html
- http://opensecuritytraining.info/Exploits1.html
- https://www.corelan.be/index.php/articles/
- https://exploit-exercises.com/protostar/
- https://www.reddit.com/r/hacking/comments/1wy610/exploit_tutorial_buffer_overflow/
- https://www.corelan.be/index.php/category/security/exploit-writing-tutorials/
- 
- [Over the Wire](http://overthewire.org/wargames/narnia/):http://overthewire.org/wargames/narnia/ 


# 扫描网络

## 被动信息收集——开源情报(OSINT)

在搜索之前伪造一些社交媒体账户：

- LinkedIn
- Twitter
- Google+
- Facebook
- Instagram
- MySpace
- Glassdoor


通过Recon-NG、discover、spiderfoot三个工具可获取大量的开源情报。

### Recon-NG

Recon-NG是一个非常好的工具，主要通过被动信息收集开源情报，能够获得IP、位置、用户、邮件地址、密码可能泄露或其他大量信息。

- `workspaces add 公司名字`
- `add domains + 域名`
- `add companies`
- `use recon/domains-hosts/bing_domain_web`
- `use recon/domains-hosts/google_site_web`
- `use recon/domains-hosts/brute_hosts`
- `use recon/domains-hosts/netcraft`
- `use recon/hosts-hosts/resolve`（域名解析成主机IP）
- `use recon/hosts-hosts/reverse_resolve`
- `use discovery/info_disclosure/interesting_files`(在指定域查找文件)
- `keys add ipinfodb_api[key]`
- `use recon/hosts-hosts/ipinfodb`(查找探测IP地址的物理位置)
- `use recon/domains-contacts/whois_pocs`（使用whosi查询邮件地址）
- `use recon/domains-contacts/pgp_search`(查询公共PGP中存储的邮件地址)
- `use recon/contacts-credentials/hibp_paste`（将搜集的邮箱在"Have I Been PWN'ed"网站上比对泄露的密钥）
- `use reporting/html`




### Discover脚本

应用被动式探测扫描方法

### SpiderFoot

下载，并安装环境：

```css
root@kali:/opt# wget https://sourceforge.net/projects/spiderfoot/files/spiderfoot-2.10-src.tar.gz/download

root@kali:/opt/spiderfoot-2.10# pip install lxml

root@kali:/opt/spiderfoot-2.10# pip install netaddr

root@kali:/opt/spiderfoot-2.10# pip install M2Crypto

root@kali:/opt/spiderfoot-2.10# pip install cherrypy

root@kali:/opt/spiderfoot-2.10# pip install mako

```

运行：

![Spiderfoot](/img/the_hacker_play_book/Spiderfoot.png)


## 创建密码字典

### Wordhound

[Wordhound](https://bitbucket.org/mattinfosec/wordhound.git)基于推特搜索、PDF文档、Reddit子网站创建密码字典。

下载
```css
root@kali:~# git clone https://bitbucket.org/mattinfosec/wordhound.git/opt/wordhound
```

### BruteScrape

[BruteScrape](https://github.com/cheetz/brutescrape):https://github.com/cheetz/brutescrape

其能够导入大量网页并生成密码字典

### gitrob

[gitrob](https://github.com/michenriksen/gitrob)搜索github的敏感文件。

首先在github注册账户，获取token，然后将令牌导入到gitrob，直接搜索


## 主动式扫描

主动式扫描就是通过扫描确认目标安装的操作系统和网络服务，并发现潜在漏洞的过程。

扫描过程如下：

- Masscan扫描
- Sparta扫描
- HTTP屏幕截屏扫描
- Eyewitness/WMAP扫描
- Nexpose/Nessus/Open VAS扫描
- Burp Proxy Pro扫描
- ZAP Proxy扫描
- 分析

##　网络应用程序扫描

在进行开源情报收集之后，创建密码字典，并对网络进行漏洞扫描，了解目标网络架构和运行的服务情况，就对目标进行网站应用程序扫描。可以使用ZAP、WebScarab、Nikto、w3af、Burpsuit等。、


### ZAP

与Burpsuit Pro Proxy相类似的是OWASP Zed Attack Proxy(ZAP)。ZAP具有流量代理、fuzz请求、抓取网页、自动化扫描工具等。


![ZAP](/img/the_hacker_play_book/zap.png)

### 分析XML报告

将Nessus、Nmap、Burp的扫描报告整理成为xml文件，通过discover脚本分析处理。


# 漏洞利用

可以通过打印机漏洞入侵。

## 资源

- [Windows Post-Exploitation Command Execution](https://docs.google.com/document/d/1U10isynOpQtrIK6ChuReu-K1WHTJm4fgG3joiuz43rw/edit?hl=en_US)
- [Linux/Unix/BSD Post-Exploitation Command List](https://docs.google.com/document/d/1ObQB6hmVvRPCgPTRZM5NMH034VDM-1N-EWPRz2770K4/edit?hl=en_US)
- [Metasploit Post-Exploitation Command List](https://docs.google.com/document/d/1ZrDJMQkrp_YbU_9Ni9wMNF2m3nIPEA_kekqqqA2Ywto/edit)
- [Obscure Systems Post-Exploitation Command List](https://docs.google.com/document/d/1CIs6O1kMR-bXAT80U6Jficsqm0yR5dKUfUQgwiIKzgc/edit)
- [OS X Post-Exploitation Command List](https://docs.google.com/document/d/10AUm_zUdAQGgoHNo_eS0SO1K-24VVYnulUD2x3rJD3k/edit?hl=en_US)
- [Unix-Privilege-Escalation-Exploits-Pack](https://github.com/Kabot/Unix-Privilege-Escalation-Exploits-Pack)


# 人工检测

## CMSmap

通过[cmsmap](https://github.com/Dionach/CMSmap)快速扫描cms模板的漏洞详情：

![cmsmap](/img/the_hacker_play_book/cmsmap.png)

## XSS

可在https://www.reddit.com/r/xss/上获取大量xss话题

# 渗透内网

进入域控之外的主机上，监听网络情况，分析局域网架构。

## BDF

[BDF](https://github.com/secretsquirrel/BDFProxy)可用`apt-get install bdfproxy`安装 。

能在用户接收程序更新之前将shellcode注入到补丁程序且能正确运行。


## 粘滞键

在windows主机上按五次粘滞键，旧方法是替换cmd，新方法是更改注册表


# 树莓派

- `update-rc.d -f ssh remove`
- `update-rc.d -f ssh defaults`
- `dpkg-reconfigure openssh-server`
- `passwd`
- `wget https://raw.githubusercontent.com/dweeber/rpiwiggle/master/rpi-wiggle`
- `chmod a+x rpi-wiggle`
- `./rpi-wiggle`











