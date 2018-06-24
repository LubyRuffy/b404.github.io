---
title: tools——AquatoneDiscover
layout: post
tags:
  - sec
  - tools
  - domain
  - information
category: 
  - hack
  - tools
comments: true
share: true
description: AQUATONE是一套用于对域名进行侦察的工具。它可以通过使用字典爆破给定域的子域。在子域发现后，AQUATONE可以扫描主机，查找常见的Web端口，HTTP头，HTML主体，将收集的信息整合到一个报告中
---

AQUATONE是一套用于对域名进行侦察的工具。它可以通过使用字典爆破给定域的子域。在子域发现后，AQUATONE可以扫描主机，查找常见的Web端口，HTTP头，HTML主体，将收集的信息整合到一个报告中

* TOC
{:toc}

<!--more-->

## 安装

依赖于最新版Ruby。

在终端中使用gem安装该工具

```css
gem install aquatone
```

## 使用

安装成功之后，直接运行`aquatone-discover+参数`：

```css
root@kali:~# aquatone-discover --help
Usage: aquatone-discover OPTIONS
    -d, --domain DOMAIN              Domain name to assess
        --nameservers NAMESERVERS    Nameservers to use
        --fallback-nameservers NAMESERVERS
                                     Nameservers to fall back to
        --[no-]ignore-private        Ignore hosts resolving to private IP addresses
        --set-key KEY VALUE          Save a key to key store
        --list-collectors            See information on collectors
        --only-collectors COLLECTORS Only run specified collectors
        --disable-collectors COLLECTORS
                                     Disable specified collectors
    -t, --threads THREADS            Number of concurrent threads to use
    -s, --sleep SECONDS              Seconds to sleep between lookups
    -j, --jitter PERCENTAGE          Jitter factor for sleep intervals
    -h, --help                       Show help

``` 

*例子*


![aquatone-discover.png](/img/tools/aquatone-discover.png)
