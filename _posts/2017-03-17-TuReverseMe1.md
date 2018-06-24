---
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
---

去除程序的Nags，查找registration code

* TOC
{:toc}

<!--more-->

## 运行

**0x01 运行程序，猜测程序的基本结构和运行步骤**
![运行程序.png](/img/reversecore_assets/tut.reverseme1/运行程序.png)

## 去除消息框

**0x02 将程序用OD载入，观察程序EP和反汇编代码：**

![EP代码.png](/img/reversecore_assets/tut.reverseme1/EP代码.png)
观察可知，该代码是VB编写。
要去除该代码的消息框，只要操作调用消息框的函数部分即可。VB中调用消息框的函数是`MSVBVM50.rtcMsgBox`

**0x03 在OD中右键->Search for->All intermodular calls命令(所有模块间的调用)，将会出现程序调用API的目录.**

![All_intermodular calls.png](/img/reversecore_assets/tut.reverseme1/All_intermodular calls.png)
共有4处调用`rtcMsgBox`
**0x04 右键->Set breakpoint on every call to rtcMsgBox，在所有调用`rtcMsgBox`的代码处下断点。**
![在调用rtcMsgBox代码处设置断点.png](/img/reversecore_assets/tut.reverseme1/在调用rtcMsgBox代码处设置断点.png)

**0x05 在调试器F9运行。程序在运行到设有断点的地方就停下来了。程序运行到402CFE地址处就停下。稍微向上下拉滚动条就可看到消息框中的字符串。**
**继续F9运行，弹出消息框，选择"确定"，在主画面中按下“Nag？”按钮，最初显示的消息框与主画面的“Nag？”按钮显示的消息框有相同的代码块，所以只要对一处进行打补丁就行**
**0x06 上拉滚动条，可以看到`402C17`地址处表示函数开始的栈帧开端**
`402CFE`的`rtcMsgBox`函数调用代码也是属于其他函数内部的代码。若上层函数无法调用或直接返回，最终就不会调用`rtxMsgBox`函数。
**0x07 在`401C17`处，按空格键，更改代码为`RETN 4`**

![更改401C17处代码指令.png](/img/reversecore_assets/tut.reverseme1/更改401C17处代码指令.png)

> 根据传递给函数的参数大小调整栈(RETN XXX)

**0x08 保存新生成的文件，并运行成功，去除了Nag**

![保存新生成文件.png](/img/reversecore_assets/tut.reverseme1/保存新生成文件.png)

## 查找注册码

**0x09 先输入任意值尝试**

![输入错误的注册码测试.png](/img/reversecore_assets/tut.reverseme1/输入错误的注册码测试.png)

**0x10 在OD载入程序，右键->All referenced text strings（所有参考文本字符串）**

![All_referenced_text_strings.png](/img/reversecore_assets/tut.reverseme1/All_referenced_text_strings.png)
**0x11 找到与之前测试的错误代码显示的字符串,双击进入**
![402A69地址处的代码.png](/img/reversecore_assets/tut.reverseme1/402A69地址处的代码.png)

![弹出消息框的代码.png](/img/reversecore_assets/tut.reverseme1/弹出消息框的代码.png)


**0x12 对比正确和错误的回显字符串，发现了固定的字符串"I'mlenal151",在`402A2F`地址处是`__vbaStrCmp()`函数调用代码，该函数调用代码是用于比较用户输入的字符串与"I'mlenal151"。所以该注册码就是"I'mlenal151"**

![成功消息框.png](/img/reversecore_assets/tut.reverseme1/成功消息框.png)
