---
title: UPack调试——查找OEP
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
---
UPack会对PE文件进行独特的变形，本文通过对UPack压缩过的文件进行调试找出OEP。

* TOC
{:toc}

<!--more-->

# 载入UPack压缩程序

由于Upack会将`IMAGE_OPTIONAL_HEADER`中的`NumberOfRvaAndSizes`值设置为A（默认为10），打开该压缩文件时，会弹出错误消息。
![OD报错](/img/reversecore_assets/UPack_PE_OEP/OD%E9%94%99%E8%AF%AF%E6%B6%88%E6%81%AF%E6%A1%86.png)


确认之后，关闭对话框，以上错误导致OD无法转到EP位置，停留在ntdll.dll区域。
![ntdll.dll区域](/img/reversecore_assets/UPack_PE_OEP/ntdll%E4%BB%A3%E7%A0%81%E5%8C%BA%E5%9F%9F%E4%BB%A3%E7%A0%81.png)

这是由于OD的Bug（严格的PE检查）引起，先要强制设置EP，使用stud_PE查找EP的虚拟地址。

# 强制设置EP

Imagebase为01000000,EP的RVA为1018，可知EP的VA值为01001018，在01001018处，使用`New origin here`强制更改EP寄存器的值：

![Upack的EP代码](/img/reversecore_assets/UPack_PE_OEP/Upack%E7%9A%84EP%E4%BB%A3%E7%A0%81.png)

# 解码循环

所有压缩器都存在解码循环，压缩/解压算法本身就是由许多条件分值语句和循环构成。

调试解码循环，应适当跳过条件分支语句以跳出某个循环。

**UPack把压缩后的数据放到第二个节区，再运行解码循环将这些数据解压缩后放到第一个节区。**


![函数调用](/img/reversecore_assets/UPack_PE_OEP/%E5%87%BD%E6%95%B0%E8%B0%83%E7%94%A8.png)

 在进行F8反复跟进之后(也可以使用Stepinto(F7))，发现执行到`0101FE61`之后又循环到`0101FD13`，在`010FD18`处调用的函数可以确定为是一个decode()函数，其地址为`0101FCCB`，并反复执行。


![解压缩后的代码](/img/reversecore_assets/UPack_PE_OEP/%E8%A7%A3%E5%8E%8B%E7%BC%A9%E5%90%8E%E7%9A%84%E4%BB%A3%E7%A0%81.png)

`0101FE57`和`0101FE5D`地址处有`向EDI所指位置写入内容`的指令。此时，EDI值指向第一个节区中的地址。即，这些命令会先执行解压缩操作，然后写入实际内存。在`0101FE5E`与`0101FE61`地址处通过CMP/JB指令继续执行循环，直到EDI值为`01014B5A`([ESI+34]=01014B5A)。地址`0101FE61`即是解码循环的结束部分。在循环调试中，课随时看到向EDI所指地址中写入了什么值。


# 设置IAT

一般，压缩器执行完解码循环后会根据原文件重新组织IAT。

UPack会使用导入的2个函数(LoadLibraryA和GetProcAddress)边执行循环边构建原本notepad的IAT(先获取notepad中导入函数的实际内存地址，再写入原IAT区域)。该过程结束之后，由`0101FEAF`地址处的RETN命令将运行转到OEP`0100739D`

![解压缩后的OEP](/img/reversecore_assets/UPack_PE_OEP/UPack%E7%9A%84OEP.png)
