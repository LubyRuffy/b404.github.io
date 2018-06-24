---
title: shellcode自识别32/64位架构
layout: post
tags:
  - sec
  - skills
category: 
  - hack
  - skills
comments: true
share: true
---

* TOC
{:toc}

<!--more-->



## 推荐方案(zero sum)

shellcode自识别32/64位架构极其重要，比如Windows的TIB，x86上用FS寻址，x64上
用GS寻址。下面这5字节汇编指令可以自动识别32/64位架构:

```css

--------------------------------------------------------------------------
_shellcode:

    xor ecx,ecx         ; set ecx to 0
    db 0x41             ; x86 opcode for: inc ecx
    loop x64_code       ; ecx now -1 in x64, we jmp

x86_code:

    ; use fs segment

    ; ret

x64_code:

    ; use gs segment
--------------------------------------------------------------------------
```

熟悉x64汇编的人应该知道，[0x40,0x50]经常用作指令前缀，可以利用这点微妙的区
别自动识别32/64位架构。

假设在x86上运行:

```css
--------------------------------------------------------------------------
xor ecx,ecx         ; 31c9      ecx=0
inc ecx             ; 41        ecx=1
loop x64_code       ; e201      ecx=0 no jmp
--------------------------------------------------------------------------

```

假设在x64上运行:

```css

--------------------------------------------------------------------------
xor ecx,ecx         ; 31c9      ecx=0
rex.B loop x64_code ; 41e201    ecx=-1 jmp
--------------------------------------------------------------------------

```

这里用"db 0x41"，而不是直接写成"inc ecx"，因为nasm会采用2字节指令避免这种
歧义，我们恰恰需要这种歧义。

上面给出的机器码只是演示，e201中的01是假设x86_code处只有一条ret指令，占1个
字节。

Eternalblue使用了本质上相同的类似方案:

```css

--------------------------------------------------------------------------
31 C0                                   xor     eax, eax
40 90                                   xchg    eax, eax
0F 84 B5 05 00 00                       jz      x64_main
--------------------------------------------------------------------------

```

这是推荐方案，它只与CPU相关，与OS无关，理论上针对Linux也能使用这种技术。后
面其他方案都是针对Windows的。

## Kronos malware

```css
xor eax, eax
mov ax,  cs
shr eax, 5
```

x86/Win7用户态的CS是0x1B，x64/Win7用户态的CS是0x23或0x33，因此最后的eax为0
时表示x86，为1时表示x64。

上述代码只适用于用户态，且无法区分SysWOW64。

## 其他段选择子

```css
--------------------------------------------------------------------------
x86/Win7 user

    cs=001b  ss=0023  ds=0023  es=0023  fs=003b  gs=0000

x64/Win7 32-bits user

    cs=0023  ss=002b  ds=002b  es=002b  fs=0053  gs=002b

x64/Win7 64-bits user

    cs=0033  ss=002b  ds=002b  es=002b  fs=0053  gs=002b
--------------------------------------------------------------------------
x86/Win7 kernel

    cs=0008  ss=0010  ds=0023  es=0023  fs=0030  gs=0000

x64/Win7 kernel

    cs=0010  ss=0018  ds=002b  es=002b  fs=0053  gs=002b
--------------------------------------------------------------------------

```

OsandaMalith受Kronos启发，用其他段选择子区分32/64位架构:

```css
--------------------------------------------------------------------------
xor  eax, eax
mov  ax,  es
ror  ax,  0x3
and  eax, 0x1
test eax, eax
--------------------------------------------------------------------------
xor  eax, eax
mov  eax, gs
test eax, eax
--------------------------------------------------------------------------

```

上述代码无法区分SysWOW64。当然，不必那么死板，只要存在差异，就可以区分。

## TEB

```css
x86/Win7 user

    > dt ntdll!_TEB WOW32Reserved @$teb
       +0x0c0 WOW32Reserved : (null)

    > ? @$teb
    Evaluate expression: 2147344384 = 7ffde000

    x86用户态FS:0指向ntdll!_TEB，可以用"dg @fs"获取FS段基址。

    > dg @fs
                                      P Si Gr Pr Lo
    Sel    Base     Limit     Type    l ze an es ng Flags
    ---- -------- -------- ---------- - -- -- -- -- --------
    003B 7ffde000 00000fff Data RW Ac 3 Bg By P  Nl 000004f3

x64/Win7 32-bits user

    > dt ntdll!_TEB WOW32Reserved @$teb
       +0x0c0 WOW32Reserved : 0x74a92320 Void
    > u 0x74a92320 l 1
    74a92320 ea1e27a9743300  jmp     0033:74A9271E

    > ? @$teb
    Evaluate expression: 2130563072 = 7efdd000

    > dg @fs
                                      P Si Gr Pr Lo
    Sel    Base     Limit     Type    l ze an es ng Flags
    ---- -------- -------- ---------- - -- -- -- -- --------
    0053 7efdd000 00000fff Data RW Ac 3 Bg By P  Nl 000004f3

x64/Win7 64-bits user

    > dt ntdll!_TEB WOW32Reserved @$teb
       +0x100 WOW32Reserved : (null)

    > ? @$teb
    Evaluate expression: 8796092882944 = 000007ff`fffde000

    x64用户态GS:0指向ntdll!_TEB。对于x64，"dg @gs"无法获取GS段基址。

```

用WOW32Reserved可以区分SysWOW64，但WOW32Reserved的偏移对于三种情况不统一，
而且访问TEB时涉及FS、GS的选择，本方案意义不大。

```css
xor  eax, eax
mov  eax, [FS:0xc0]
test eax, eax
```

## refer

标题: shellcode自识别32/64位架构

创建: 2014-12-26
更新: 2017-09-26 17:18
链接: http://scz.617.cn/windows/201412260000.txt


本文为下列文档的意译，仅为收集整理点评，非原创。

--------------------------------------------------------------------------
Architecture Detection (x86 or x64) Assembly Stub - zero sum [2014-12-26]
https://zerosum0x0.blogspot.com/2014/12/detect-x86-or-x64-assembly-stub.html

Detecting Architecture in Windows - OsandaMalith [2017-09-24]
https://osandamalith.com/2017/09/24/detecting-architecture-in-windows/

checking_architecture.cpp
https://gist.github.com/hasherezade/0994447e9d3dc184888fb2afd5a57301

The initial values of x86 registers
https://github.com/corkami/docs/blob/master/InitialValues.md

--------------------------------------------------------------------------