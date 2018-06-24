---
title: Payment_Advice.ppsx
layout: post
tags:
  - trojan
  - reverse
category: 
  - reverse
  - malware
comments: true
share: true
description: Payment_Advice.ppsx的ppt文件是Poison Ivy木马的变种，该文件为OOXML格式，一旦受害者利用office办公软件打开此文件，文件中的恶意代码就会执行，它会下载Poison Ivy恶意软件到受害者主机上运行
---
Payment_Advice.ppsx的ppt文件是Poison Ivy木马的变种，该文件为OOXML格式，一旦受害者利用office办公软件打开此文件，文件中的恶意代码就会执行，它会下载Poison Ivy恶意软件到受害者主机上运行

* TOC
{:toc}

<!--more-->

## 实验环境

系统环境：Windows 7 , Vmware Station Pro 12.5 
软件环境：office 2013

实验影响的office有：office 2010，office 2013,office 2016

## 分析

### 下载&文件结构分析

下载目标样本[e7931270a89035125e6e6655c04fee00798c4c2d15846947e41df6bba36c75ae](https://malwr.com/analysis/NmQ3YTY2NmZjZGM2NDEyMzk3YzkxOWQwZTE3ZjE2YjQ/)，使用tree初步分析其文件结构：

![tree分析结构](/img/reverse/trojan/Payment_Advice_ppsx/%E6%96%87%E4%BB%B6%E6%A0%91%E5%BD%A2%E7%BB%93%E6%9E%84.png)

![tree分析](/img/reverse/trojan/Payment_Advice_ppsx/PPSX%E6%96%87%E4%BB%B6%E7%BB%93%E6%9E%84.png)

`.\ppt\slides\`子目录下看到，slide1.xml是图1中自动展示的幻灯片，`.\_rels\slide1.xml.rels`文件是关系文件，定义了slide1.xml用到的资源.

打开两个相关文件，关系如下：

![xml文件和xml.rels的关系](/img/reverse/trojan/Payment_Advice_ppsx/rld2%E4%B8%AD%E7%9A%84%E4%BB%A3%E7%A0%81.png)



### 运行

，更改文件后缀名为ppsx，其表示为`PowerPointShow`，运行，以演示模式打开，这允许恶意代码自动执行，弹出消息框警告，继续运行：

![运行ppsx](/img/reverse/trojan/Payment_Advice_ppsx/Payment_Advice_ppsx%E6%89%93%E5%BC%80.png)

运行之后，开始菜单的启动项出现了thumbs.vbs文件：

![启动项的vbs脚本](/img/reverse/trojan/Payment_Advice_ppsx/%E5%BC%80%E5%A7%8B%E6%96%87%E4%BB%B6%E5%A4%B9%E4%B8%AD%E7%9A%84thumb.vbs%E6%96%87%E4%BB%B6%E5%92%8C%E5%86%85%E5%AE%B9.png)

### 分析

为开机自启动项的Thumb.vbs从http://203.248.116.182/images/Thumbs.bmp下载了一个bmp文件,并通过微软的msiexec.exe程序执行(.MSI文件的默认句柄)。MSI文件包含了一个PE文件，这个PE在msiexec.exe加载时执行，通过此方法绕过AV的检测。

![木马的msi程序形态](/img/reverse/trojan/Payment_Advice_ppsx/msi%E6%96%87%E4%BB%B6.png)

使用压缩软件/文件查看器查看该文件，从文件包中提取出PE文件

![bmp文件包含内容](/img/reverse/trojan/Payment_Advice_ppsx/binwalk%E6%9F%A5%E7%9C%8Bthumbs_bmp%E7%9A%84%E6%96%87%E4%BB%B6%E6%A0%BC%E5%BC%8F.png)

并分析得到其为.NET文件，使用dnspy分析：

![Main()函数](/img/reverse/trojan/Payment_Advice_ppsx/dnspy%E5%8F%8D%E6%B1%87%E7%BC%96%E6%9C%A8%E6%9C%A8%E9%A9%AC1.png)

![rGHDcvkN.Exec()函数](/img/reverse/trojan/Payment_Advice_ppsx/dnspy%E5%8F%8D%E6%B1%87%E7%BC%96%E6%9C%A8%E6%9C%A8%E9%A9%AC2.png)

![rGHDcvkN.Exec()函数](/img/reverse/trojan/Payment_Advice_ppsx/dnspy%E5%8F%8D%E6%B1%87%E7%BC%96%E6%9C%A8%E6%9C%A8%E9%A9%AC1.png)

该代码显示了.Net程序在大数组中运行一个线程来执行代码。从内存空间动态加载恶意软件代码到新分配的内存缓冲区，然后根据新基地址修复重定位问题并修正代码主体的API偏移，最后才调用主体代码的入口函数。

该木马所有API是加密的，得到调用的时候才会恢复：

```java
sub_1B0E6122proc near      

   mov  rax, 0FFFFFFFF88E23B10h

   neg  rax

   jmp  rax  ;; CreateRemoteThread

sub_1B0E6122endp
```

所有字符串均经过加密，使用前才解密，比如以下是加密的ntdll字符串：

```java
unk_1AFD538C  db 54h, 0B2h, 9Bh, 0F1h, 47h, 0Ch  ; ==> "ntdll"
```

通过动态分析，该木马会自动检测命名管道`\\.\Regmon` 、`\\.FileMon`等，若无法创建命名管道，则表示有分析工具在运行，该程序就会立即终止进程。

其也会检测所有运行的windows程序是否有包含特殊字符的windows类名来判断有没有运行中的分析工具。
其通过检测`Wireshark-is-running-{…}`进行反抓包，通过`IsDebuggerPresent`函数的返回值是否为1判断是否在被调试，若在被调试自动退出进程。

该恶意程序加密了6个不同的模块，并创建一个双链接列表保存管理加载的模块，这些模块中隐藏了其主要的功能，该方式给动态调试增大了难度。

该程序会注入代码和数据到svchost.exe中，注入的代码和数据是恶意程序的核心功能，会跟随svchost.exe的重新运行而重复进行反调试，加解密运行等步骤。

后因保存在Pastbin上的C&C服务和IP的页面已经不存在，无法继续进行分析。


参考：http://blog.fortinet.com/2017/08/23/deep-analysis-of-new-poison-ivy-variant