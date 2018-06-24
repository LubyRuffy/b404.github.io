---
title: 快速查找所需代码
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
description: 在调试中快速查找所需代码，总结出来4种方法。
---
在调试中快速查找所需代码，总结出来4种方法。
<!--more-->

* TOC
{:toc}

# 代码执行法

在调试中快速查找所需代码，总结出来4种方法。

<!--more-->

## 代码执行法
 
在调试器种调试程序时，某个函数在某个时刻就会被调用。程序功能非常明确时，逐条执行指令来查找需要查找的位置。代码执行法仅仅适用于被调试的代码量不大、且程序功能明确的情况。倘若被调试的代码大且比较复杂时，该方法就不适用。

例如：

从`大本营`开始，按F8逐条执行命令，在某个时刻显示弹窗。弹出消息对话框时调用的函数即为main()函数

![代码执行法.png](/img/reversecore_assets/代码执行法.png)

在`40114F`处，调用了`401000`函数，F7进入该函数，可发现该函数就是所找的main函数


## 字符串检索法

载入程序，使用All referenced text strinbgs命令弹出窗口，列出程序代码引用的字符串

![查找所有特定字符串.png](/img/reversecore_assets/查找所有特定字符串.png)
双击所查询的字符串，然后跳转到对应地址处。
![定位4092a0.png](/img/reversecore_assets/定位4092a0.png)
在OllyDbg的Dump窗口种使用Goto(Ctrl+G)，打开Enter expression to follow in Dump（跟随表达式），右键HEX/ASCII 转换格式可查看。
![Enter-expression-to-follow-in-Dump.png](/img/reversecore_assets/Enter-expression-to-follow-in-Dump.png)

![ASCII-HelloWorld.png](/img/reversecore_assets/ASCII-HelloWorld.png)

> 401XXX是代码区域，409XXX地址空间是被用来保存程序数据使用的数据。两区域是分开的。

## API检索法(1)

调用代码种设置断点。
鼠标右键->Search for ->All intermodular calls

Windows编程，若想向显示器显示内容，则需要使用Win32API向OS请求显示输出。

如示例程序所需要弹出消息框，就可以推断该程序调用了user32.MessageBoxW()API。

例如：
载入程序，鼠标右键->Search for ->All intermodular calls，查看到所有调用的API，目标函数调用的Win32 API是`user32.MessageBoxW()`。双击它，可定位到调用它的地址。

![所有模块间的调用.png](/img/reversecore_assets/所有模块间的调用.png)

![user32.MessageBoxW.png](/img/reversecore_assets/user32.MessageBoxW.png)

## API检索法(2)

OD并不能为所有可执行文件都列出API函数调用列表。使用压缩器、保护器工具之后，文件程序的结构就会改变，此时OD无法列出所有API调用列表。

这种情况下，DLL代码被加载到进程内存后，可以直接向DLL代码库种添加断点。

例如：
在OD菜单栏打开View-Memory(Alt+M)打开内存映射窗口。
![MemoryMap.png](/img/reversecore_assets/MemoryMap.png)

> 如图显示了程序部分进程内存。图底USER32库被加载到内存。

鼠标右键->Search for ->Name in all modules,列出可以记载的DLL文件种提供的所有API。通过输入MessageBoxW可定位到MessageBoxW上。

![Name-in-all-modules.png](/img/reversecore_assets/Name-in-all-modules.png)

双击该MessageBoxW进入USER32.DLL。通过观察发现，它与程序使用的地址空间完全一样。在函数起始地址上按F2键，设置好断点后F9运行。若程序调用了MessageBoxW()API，则调试程序时程序运行到该处暂停。


![USER32.DLL下断点.png](/img/reversecore_assets/USER32.DLL下断点.png)

![断点停下.png](/img/reversecore_assets/断点停下.png)

此时寄存器窗口种的ESP的值为12FF30，它是进程栈的地址。在右下角查看情况。


![进程栈情况.png](/img/reversecore_assets/进程栈情况.png)

ESP值的`12FF30`处对应一个返回地址401014，程序的main()函数调用完MessageBoxW后，程序执行流将返回到该地址处。按Ctrl+F9运行到RETN命令处，然后按F7也可返回到`401014`处。地址`401014`的上方地址就是40100E处，正是调用MessageBoxW函数的地方。



