---
title: 阻碍获取真实网络指纹
layout: post
tags:
  - sec
  - hack
  - skills
category: 
  - hack
  - sec
comments: true
share: true
description: 真实环境中，很多因素对基本指纹识别，比如说负载平衡器、虚拟服务器配置、代理、WAF、CDN
---

* TOC
{:toc}

真实环境中，很多因素对基本指纹识别，比如说负载平衡器、虚拟服务器配置、代理、WAF、CDN...

<!--more-->

## CDN

**CDN**是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。CDN的基本原理是广泛采用各种缓存服务器，将这些缓存服务器分布到用户访问相对集中的地区或网络中，在用户访问网站时，利用全局负载技术将用户的访问指向距离最近的工作正常的缓存服务器上，由缓存服务器直接响应用户请求。

### 发现CDN

* 查看域名的历史解析记录
* 注册的地方，看邮件头，扫C段
* 寻找主站的真实IP，搜集其他公司的所有IP，写脚本，绑定host，直接解析
* 验证是否存在CDN
通过在线的多地ping，通过各地区ping的结果得到ip，比较ip
* 验证ip和域名是否真实对应
修改本地hosts文件，强行将域名与ip解析对应，然后访问域名，查看页面是否变化
* ping顶级域名。厂商可能让www使用cdn，空域名不使用CDN缓存
* ping分站域名。为了省钱，分站未作CDN。对比IP
* 通过国外代理访问，或者通过国外的DNS解析。许多国内CDN只针对国内用户加速。
* mx记录查询，一般会是c段。一些网提供注册服务，可能会验证邮件，还有RSS订阅邮件、忘记密码等等可能服务器本身自带sendmail可以直接发送邮件，当然使用第三方的除外（如网易、腾讯的等）
通过邮件发送地址往往也能得到服务器IP。当然这个IP也要验证是否为主站的。Web版的邮件管理，可以通过常看网页源代码看到IP
* [信息查询](https://passivetotal.org/home)
* xss，让服务器主动连接攻击者
* 找彩蛋phpinfo();之类的探针
* DDOS，耗尽CDN流量，回源之后找到真实ip。不设防的cdn量大就会挂，高防CDN要增大流量
* 社工。勾搭CDN妹子
* 查看域名绑定历史,(17ce)[http://www.17ce.com/] (netcraft)[http://toolbar.netcraft.com/site_report?url=]
* DNS社工裤
* cloudflare (freebuff)[http://www.freebuf.com/articles/web/41533.html]
* 全网扫描，zmap
* [CloudFail](https://github.com/m0rtem/CloudFail)：查找CloudFlare隐藏的IP
* 

## 负载平衡器

负载平衡器是通过在多个服务器中分割WEB流量，来确保单个服务器的请求不会超载，一般不可见，但是有可能极大的改变渗透测试的进程。当发出请求的时候，负载平衡器可能将请求交给其中的一个服务器来处理。每个服务器的应用结构虽然相同，但是文件夹不同、补丁级别不同及其他配置可能完全不同。

### 识别负载平衡器

- **对临近IP段进行扫描**
- **[查询负载均衡](https://github.com//jmbr/halberd)**：https://github.com//jmbr/halberd
- **响应时间戳分析**：有些服务器没有同步时间，就可以在一秒发出多个请求确定是否有多台服务器
- **通过比较相同请求资源的首部响应中的ETag和Last-Modified的值，能够确定是否会从多个服务器上的到不同的文件**
- **通过谷歌查找cookie：**有些服务器会在HTTP会话中加入自己的cookie
- **枚举SSL异常**：查看SSL证书是否存在差异，是否支持同一个密码长度
- **检查HTML源码中返回的注释**

## 虚拟服务器

一些WEB主机公司为了节约成本，在相同机器上的多个虚拟IP上运行不同的WEB服务器。端口扫描所指出的不同IP地址上的大量活跃服务器，实际上可能是具有多个虚拟IP地址的单台机器。

## 检测代理

- TRACE请求：TRACE请求通知Web服务器回复刚刚收到的请求内容，代理服务器会添加某些首部。
- 标准连接测试：CONNECT命令主要用于代理服务器代理SSL连接。比如发送`CONNECT https://site.com:443`，连接成功将建立安全链接隧道
- 标准代理请求：插入公共网站的地址，查看代理服务器是否返回该网站的响应，若返回就意味着可以将该服务器导向任何你选择的地址，使得你的代理服务器成为一个向公众开放的匿名代理，允许攻击者访问内部网络。可用的一种较好的技术是识别目标的内部IP地址范围，然后对着一范围进行端口扫描。
  - 测试代理：`GET http://www.site.com/ HTTP/1.0`
  - 扫描开放Web服务器的网络：`GET http://192.168.1.1:80 HTTP/1.0`


## 检测WAF

WAF是放在用户和Web服务器之间的保护性设备。

- [区别cookie](http://www.freebuf.com/articles/web/21744.html)：http://www.freebuf.com/articles/web/21744.html
- wafw00f:`pip install wafw00f`
- xenoitx:
- sqlmap:`python sqlmap.py -u “http://www.victim.org/ex.php?id=1” --identify-waf`
- [FingerPrint](https://github.com/tanjiti/FingerPrint.git):https://github.com/tanjiti/FingerPrint.git