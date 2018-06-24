---
title: HelloWorld
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
description: 逆向工程，一般指，通过分析物体、机械设备或系统，了解其结构、功能、行为等，掌握其中原理、改善不足、添加新意的过程。
---
逆向工程，一般指，通过分析物体、机械设备或系统，了解其结构、功能、行为等，掌握其中原理、改善不足、添加新意的过程。

先静态分析，收集代码相关信息，然后通过收集到的信息推测程序的结构与行为机制。再进行动态分析代码流、获得内存状态等。

<!--more-->

* TOC
{:toc}

## 调试HelloWorld.exe程序

**0x01 使用OllyDbg调试打开程序**

![OD界面](/img/reversecore_assets/1488893454189.png)

* *代码窗口：默认用于显示反汇编代码，还用于显示各种注释、标签，分析代码时显示循环、跳转位置等信息*
* *寄存器窗口：实时显示CPU寄存器的值，可用于修改特定的寄存器*
* *数据窗口：以Hex/ASCII/Unicode值的形式显示进程的内存地址，也可在此修改内存地址*
* *栈窗口：实时显示ESP寄存器指向的进程栈内存，并允许修改*

**0x02**

**目标是找出main()函数调用MessageBox()函数的代码**
![Alt text](/img/reversecore_assets/1489321376663.png)

在EP(程序入口点）使用F7单步步入，进入`40270C`函数
![Alt text](/img/reversecore_assets/1489321290293.png)
在`4027A1`地址处又一条RETN指令，它用于返回到函数调用者的下一条指令，一般是被调用函数的最后一句，即返回`4011A5`地址处。在`4027A1地址处的RETN指令上执行`F8,继续操作。按F7或者F8执行RETN指令，程序会跳转到`4011A5`地址处。
![Alt text](/img/reversecore_assets/1489321535760.png)
![Alt text](/img/reversecore_assets/1489321836198.png)

执行`4011A5`地址处的跳转命令`JMP 0040104F`，跳转到`40104F`地址处。

![Alt text](/img/reversecore_assets/1489322005723.png)
从`40104F`地址开始，每执行一次F7命令就下一移一行代码，移动到`401056`地址处的`CALL 402524`函数调用指令，进入`402524`函数。

![Alt text](/img/reversecore_assets/1489322188279.png)

> `40254`函数不能称为main()函数，在它的代码中并未发现调用`MessageBox()`API代码。执行`Execute till Return`（Ctrl+F9）指令，调试转到`402568`地址处的RETN指令，然后使用F7(或者F8)执行RETN指令，跳出`402524`函数，返回`40105B`地址处。先F7进入函数，查看是不是main()函数，不是就直接`Ctrl+F9`跳出相关函数。

`4010E4`地址处的`CALL Kernel32.GetCommandLineW`指令是调用Win32API的代码。直接使用F8跳过该函数。
![Alt text](/img/reversecore_assets/1489322638306.png)
调试正常则会看到以下代码：
![Alt text](/img/reversecore_assets/1489323729734.png)

`401144`地址处有一条`CALL 401000`指令，用于调用`401000`函数，使用F7进入`401000`函数。

![Alt text](/img/reversecore_assets/1489323856411.png)

`401000`函数处出现了调用MessageBoxW()API的代码，该API函数参数为`www.reverse.com`和"Hello World"两个字符串，由此可知该函数就是main()函数。



