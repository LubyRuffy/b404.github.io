---
title: Socks端口转发和Socks端口代理
layout: post
tags:
  - sec
  - skills
  - proxy
  - socks
  - port-forwading
category: 
  - hack
  - skills
comments: true
share: true
description: socks端口代理和socks端口转发
---

* TOC
{:toc}

`socks端口代理`和`socks端口转发`不能混为一谈。

<!--more-->

## Socket端口转发

lcx.exe 就是一个基于 socket 套接字实现的端口转发工具，它是从 linux 下的htran 工具移植到windows平台的。

一条正常的socket隧道必具备两端，一侧为服务端，它会监听一个端口等待客户端连接；
另一侧为客户端，通过传入服务端的ip和端口，才能主动连接到服务器。

而端口转发工具（lcx.exe/htran)的工作原理其实是将两条 socket 隧道对接起来，打造一条可**异步双向通讯**的转接隧道。由于合法的socket隧道有两种接口分别对应服务端和客户端，根据数学中的排列组合可计算出端口转发供具有4种工作状态，它们是:

1. “客户端” 接 “客户端”
2. “客户端” 接 “服务端”
3. “服务端” 接 “客户端”
4. “服务端” 接 “服务端”


又由于端口转发为“异步双向通讯”隧道，隧道转接不分先后，所以状态2和状态3 是相同的，合并之后，便分别对应了lcx的三种工作模式，如下所示：

1. slave “客户端” 接 “客户端” 
2. tran “服务端” 接 “客户端” 
3. listen “服务端” 接 “服务端”


于是便可理解lcx工具的三种命令参数的格式为何是以下的样子：

– `listen` ConnectPort TransmitPort
– `tran` ConnectPort TransmitHost TransmitPort
– `slave` ConnectHost ConnectPort TransmitHost TransmitPort


## Socks代理

Socks代理从名字中的`代理`就可了解其核心功能：**帮助其他机器完成socket访问网络**

![ssh代理工作原理](/img/hack/%E4%BB%A3%E7%90%86/ssh%E4%BB%A3%E7%90%86%E5%B7%A5%E4%BD%9C%E5%8E%9F%E7%90%86.png)

> 对于访问目标来说所有的交互都只和 Socks-server 有关，Client的身份被隐藏起来了。
> 
> 当Client和访问目标间无法直接通讯时，Socks-server仍然可以正常工作，这同时又是一种翻墙操作。

![SSH代理翻墙](/img/hack/%E4%BB%A3%E7%90%86/ssh%E4%BB%A3%E7%90%86%E7%BF%BB%E5%A2%99.png)

浏览器（IE／Chrome／FireFox等）有设置socks代理的配置项，可用来访问网络的能力。当我们通过代理服务器访问一个网址时，socks服务器其实是起到了一个中间人的身份，他分别与两方（浏览器／被访问的网站）通讯然后将获取到的结果告知另一方。

在使用代理服务的过程中我们会发现，只要配置好socks代理后，就不再需要指定被访问目标，直接在浏览器的地址栏输入地址就能访问任意网站。这是由于socks代理中有一个交互协议，当我们准备访问一个网站并敲击回车时，浏览器会先发送一个被访问目标的基本信息（URL和服务端口）给socks服务端，socks服务端解析了这个信息后，会代替浏览器去访问目标网站，并将访问结果回复给浏览器端。这便是socks代理的工作原理了。

通过这段对socks代理的描述，可知socks代理其实可理解为一个增强版的 lcx -tran 它在服务端监听一个服务端口（ConnectPort），当有新的连接请求时会从socks协议中解析出访问目标的URL（TransmitHost）的目标端口(TransmitPort)，再开始执行lcx -tran 的具体功能。




## 异同

1. socket端口转发无需通讯协议支持，而socks代理需要socks协议支持。 
2. socket端口转发有三种工作方式，而socks代理仅有一种工作方式。
3. 如果说socks是帮他人访问网络（一对多），那么端口转发就是帮他人访问主机的某个端口（一对一）。


refer： [端口转发和SOCKS代理](http://rootkiter.com/2015/04/28/LCX_SOCKS.html):http://rootkiter.com/2015/04/28/LCX_SOCKS.html