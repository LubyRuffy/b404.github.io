---
title: 栈帧
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
---
栈帧在程序中用于声明局部变量、调用函数。
栈帧就是利用EBP（栈帧指针）寄存器访问栈内局部变量、参数、函数返回地址等的手段。
**简而言之，栈帧技术使用了EBP寄存器(而非ESP寄存器)管理局部变量、参数、返回地址等。**

* TOC
{:toc}

<!--more-->

ESP寄存器承担着栈顶指针的作用，而EBP寄存器则负责行使栈帧的职能。程序运行中，ESP寄存器的值随时变化，访问栈中函数的局部变量、参数时，若以ESP值为基准编写程序会十分困难，并且也很难使CPU引用到准确的地址。所以，调用函数时，先要把用作基准点(函数起始地址)的ESP值保存到EBP，并维持在函数内部。这样，无论ESP的值如何变化，以EBP的值为基准能够安全访问到相关函数的局部变量、参数、返回地址，这是EBP寄存器作为栈帧指针的作用。


```c

#include "stdio.h"

long add(long a, long b)
{
    long x = a, y = b;
    return (x + y);
}

int main(int argc, char* argv[])
{
    long a = 1, b = 2;
    
    printf("%d\n", add(a, b));

    return 0;
}
```
![StackFrame调试器画面.png](/img/reversecore_assets/StackFrame调试器画面.png)

## 开始执行main()函数&生成栈帧

首先从代码的主函数开始分析：

```c
int main(int argc, char* argv[])
{
```
函数main()是程序开始执行的地方，在main()函数的起始地址(401020)处，按F2键设置一个断点，然后按F9运行程序，程序运行到main()函数的断点处暂停。

![StackFrame栈初值.png](/img/reversecore_assets/StackFrame栈初值.png)

当前ESP的值为`12FF44`,EBP的值为`12FF88`。地址`401250`保存在ESP(12FF44)中，它是main()函数执行完毕后要返回地址。

main()函数一开始运行就生成与其对应的函数帧。

```c
00401020 PUSH EBP ;main
```
PUSH是一条压栈指令，上面这条PUSH语句的意思就是“把EBP值压入栈中”。main()函数中，EBP为栈帧指针，用来把EBP之前的值备份到栈中(main()函数执行完毕，返回之前，该值会再次恢复)

```c
00401021 MOV EBP,ESP
```
MOV是一条数据传送命令，上面这条MOV语句的命令是”把ESP值传送到EBP”。换言之，该命令之后，EBP就持有与当前ESP相同的值，并且直到main()函数执行完毕后，EBP的值始终保持不变。也就是说，通过EBP可以安全访问到存储在栈中的函数参数与局部变量。执行完`401020`和`401021`地址处的两条指令后，main()函数的栈帧就生成（设置好EBP了）

在OD的栈窗口，鼠标右键，选择Address->Relative to EBP

![选择相对于EBP菜单.png](/img/reversecore_assets/选择相对于EBP菜单.png)

接下来，在OD的栈窗口确认EBP的位置。程序调试到现在的栈内情况如图，把地址转换为相对于EBP的偏移后，能更直观地观察到站内情况。

![备份到栈中地EBP初始值.png](/img/reversecore_assets/备份到栈中地EBP初始值.png)

> 当前EBP值为12FF40，与ESP值一致，12FF40地址处保存着`12FF88`,他是main()函数开始执行时EBP持有地初始值。

## 设置局部变量

分析源码中地变量声明和赋值语句

```c
long a =1, b=2
```

main()函数中，上述语句用于在栈中为局部变量(a,b)分配空间，并赋初始值。

```c
00401023 SUB ESP,8
```
SUB是一条减法指令，上面这条语句用来将ESP地值减去8个字节。执行这条指令之前，ESP地值为12FF40，减去8个字节之后，变为12FF88.
从ESP减去8个字节，实质就是为函数地局部变量(a和b)开辟空间，以便将它们保存在栈中。由于局部变量a和b都是long型(长整型)，他们分别占据4个字节大小，所以需要在栈中开辟8个字节地空间来保存这两个变量。

使用SUB指令从ESP中减去8个字节，为两个函数变量开辟好空间后，在main()内部，无论ESP地值如何变化，变量a和b地栈空间都不会受到损坏。由于EBP的值在main()函数内部是固定不变的，就能以它为基准来访问函数的局部变量。

```nasm
00401026  MOV DWORD PTR SS:[EBP-4],1   ;[EBP-4] = local 'a'
0040102D  MOV DWORD PTR SS:[EBP-4],2   ;[EBP-4] = local 'b'
```

将`DWORD PTR SS:[EBP-4]`看作类似于C语言的指针。


![汇编语言与C语言的指针语句格式.png](/img/reversecore_assets/汇编语言与C语言的指针语句格式.png)

再次分析两条MOV指令，他们的含义就是“把数据1和2分别保存到[EBP-4]和[EBP-8]中”，即[EBP-4]代表局部变量a，[EBP-8]代表局部变量b。执行完两条语句之后，函数栈内情况：


![StackFrame程序中的变量a和b.png](/img/reversecore_assets/StackFrame程序中的变量a和b.png)

## add()函数参数传递与调用

StackFrame.cpp源码中使用如下语句调用add()函数，执行加法运算并输出函数返回值。

```
printf("%d\n",add(a,b));
```

```
00401034   mov eax,DWORD PTR SS:[EBP-8]   ;[EBP-8]=b
00401037   push eax                       ;Arg2=00000002
00401038   mov ecx,[local.1]              ;[EBP-4]=a
0040103B   push ecx                       ;Arg1=00000001
0040103C   call 00401000                  ;add()
```
上述五行汇编代码，描述了调用add()函数的整个过程。地址40103C处为“CALL 401000”命令，该命令用于调用401000处的函数，而`401000`处的函数即为add()函数。函数add()接收a、b这两个长整型参数，所以调用add()之前需要先把2个参数压入栈，地址`401034`~`40103B`之间的代码即用于此。这一过程中需要注意的是，参数入栈的顺序与C语言源码中的参数顺序恰好相反(逆向存储)。换言之，变量b（[EBP-8]）首先入栈，接着变量a([EBP-4])再入栈。执行完地址`401034`~`40103B`之间的代码后，栈内情况：

![StackFrame程序函数add()的返回地址.png](/img/reversecore_assets/StackFrame程序函数add的返回地址.png)

## 开始执行add()函数&生成栈帧

源码中的函数add()的前两行代码如下：

```
long add(long a, long b)
{
```

函数开始执行时，栈中会单独生成与其对应的栈帧。

```c
00401000  PUSH EBP
00401001  MOV EBP,ESP
```

上述两行代码与开始执行main()函数时的代码完全相同，先把EBP值(main()函数的基址指针)保存到栈中，再把当前ESP存储到EBP中，这样函数add()的栈帧就生成。如此一来，add()函数内部的EBP值始终不变。执行完以上2行代码后，栈内情况：


![StackFrame程序add函数的栈帧.png](/img/reversecore_assets/StackFrame程序add函数的栈帧.png)


main()函数使用的EBP值(12FF40)被备份到栈中，然后EBP的值被设置为一个新值`12FF28`

## 设置add()函数的局部变量(x,y)

源码中有`long x = a, y = b;`

上面语句声明了2个长整型的局部变量(x,y)，并使用2个形参(a,b)分别为它们赋初始值。<small>观察形式参数和局部变量在函数内部的表示方式</small>

```c
00401003	SUB ESP,8
```
上面这条语句含义为，在栈内存中为局部变量x、y开辟8个字节的空间。

```nasm
00401006  |. 8B45 08        MOV EAX,DWORD PTR SS:[EBP+8] ;[EBP+8] = param a
00401009  |. 8945 F8        MOV DWORD PTR SS:[EBP-8],EAX ;[EBP-8] = local x
0040100C  |. 8B4D 0C        MOV ECX,DWORD PTR SS:[EBP+C] ;[EBP+C] = param b
0040100F  |. 894D FC        MOV DWORD PTR SS:[EBP-4],ECX ;[EBP-4] = local y
```
add函数的栈帧生成之后，EBP的值发生变化，[EBP+8]与[EBP+C]分别指向参数a与b，而[EBP-8]与[EBP-4]则分别指向add（）函数的2个局部变量x、y。执行完上述语句，栈内情况：


![StackFrame程序函数add的局部变量x与y.png](/img/reversecore_assets/StackFrame程序函数add的局部变量x与y.png)

## ADD运算

源码中的`return (x+y)`用于返回2个局部变量之和。

```c
00401012  MOV EAX,DWORD PTR SS:[EBP-8]  ;[EBP-8] = local x
```
上述MOV语句中，变量X的值被传送到EAX。

```nasm
00401015  ADD EAX,DWORD PTR SS:[EBP-4]  ;[EBP-4] = local y
```

ADD指令为加法指令，上面语句中，变量y([EBP-4]=2)与EAX原值(x)相加，且运算结果被存储在EAX中，运算完成后EAX的值为3.

> EAX是通用寄存器，在算术运算中存储输入输出数据，为函数提供返回值。若向EAX中输入某个值，该值就会原封不动地返回。执行运算地过程中栈内情况保持不变。

## 删除add()函数地栈帧&函数执行完毕(返回)

源码中的`return (x+y)`对应“删除函数栈帧与函数执行完毕返回”

执行完加法运算后，要返回函数add()，在此之前先删除函数add()的栈帧。

```nasm
00401018	MOV ESP,EBP
```
上面这条指令，把当前EBP的值赋给ESP，与地址`401001`处的`MOV EBP，ESP`命令相对应。在地址`401001`处，`MOV EBP,ESP`命令把函数add()开始执行时的ESP值(12FF28)放入EBP，函数执行完毕时，使用`401018`处的`MOV ESP,EBP`命令再把存储到EBP中的值恢复到ESP中。

> 执行完上面的命令后，地址`401003`处的SUB ESP,8命令就会失效，即函数add()的两个局部变量x、y不再有效。

```nasm
0040101A POP EBP
```

上面这条指令用于恢复函数add()开始执行时备份到栈中的EBP值，它与401000地址处的PUSH EBP命令对应。EBP值恢复为`12FF40`，它是main()函数的EBP值。到此,add()函数的栈帧就被删除。
执行完上述，栈内情况：
![删除函数add的栈帧.png](/img/reversecore_assets/删除函数add的栈帧.png)

可以看到ESP的值为`12FF2C`，该地址为`401041`，它是执行`CALL 401000`命令时CPU存储到栈中的返回地址。

```nasm
0040101B RETN
```
执行上述RETN命令，存储在栈中的返回地址即被返回，此时栈内情形如图：

![StackFrame程序函数add返回.png](/img/reversecore_assets/StackFrame程序函数add返回.png)

> 调用栈已经完全返回到调用add()函数之前的状态

应用程序采用上述方式管理栈，不论有多少函数嵌套调用，栈都能得到比较好的维护，不会崩溃。但是由于函数的局部变量、参数、返回地址是一次性保存到栈中的，利用字符串函数漏洞等容易引起栈缓冲区溢出，最终导致程序或系统崩溃。

## 从栈中删除函数add()的参数(整理栈)

此时，程序执行流已经重新返回到main()函数中。

```nasm
00401041	ADD ESP,8
```
上面语句使用ADD命令将ESP加上8。地址`12FF30`和`12FF34`处存储的是传递给函数add()的参数a与b。函数add()执行完毕后，就不再需要参数a与b，所以ESP加上8，将它们从栈中清理掉(a、b参数都是长整型，各占4字节)

> **调用函数之前，先使用PUSH命令把参数a、b压入栈中**

执行上述命令后，栈内情况：

![StackFrame删除add函数的2个参数.png](/img/reversecore_assets/StackFrame删除add函数的2个参数.png)

> **被调函数执行完毕后，函数的调用者(Caller)负责清理存储在栈中的参数，该方式称为cdecl方式；反之，被调用者(Callee)负责清理保存在栈中的参数，这种方式称为stdcall方式。这些函数调用规则统称为调用约定(Calling Convention)**

## 调用printf()函数

源码中`printf("%d\n",add(a,b))`对应打印输出运算结果。

调用printf()函数的汇编代码如下：

```nasm
00401044       push eax ;函数add()的返回值
00401045       push 0x40B384 ;"%d\n"
0040104A       call 00401067 ;printf()
0040104F       add esp,0x8
```
地址`401044`处的EAX寄存器中存储着函数add()的返回值，它是执行加法运算后的结果值3.地址`40104A`处的`CALL 401067`命令中调用的是`401067`地址处的函数，它是C标准库函数printf()。由于上面的printf函数有2个参数，大小为8个字节(32位寄存器+32位常量=64位=8字节)，所以在40104F地址处使用ADD命令，将ESP加上8个字节，把函数的参数从栈中删除。函数printf()执行完毕并通过ADD命令删除参数后如图：

![StackFrame删除add函数的2个参数.png](/img/reversecore_assets/StackFrame删除add函数的2个参数.png)

## 设置返回值

源码中的返回语句是`return 0;`

main()函数使用该语句设置返回值(0)。
```
00401052	XOR EAX,EAX
```
XOR命令用来进行`Exclusive OR bit`（异或）运算，其特点是“2个相同的值进行XOR运算，结果为0”.XOR命令比`MOV EAX,0`命令执行速度快，常用于寄存器的初始化操作。

> 利用相同的值连续2次XOR运算变为原值，这个特征被大量应用于编码和解码

## 删除栈帧&main()函数终止

源码中`return 0;}`对应删除栈帧&main()函数终止。

最终主函数终止执行，同add函数一样，其返回前要先从栈中删除与其对应的栈帧。

```
00401054	MOV ESP,EBP
00401056	POP EBP
```
执行完上面两条命令后，main()函数的栈帧立即被删除，且其局部变量a、b也不再有效。执行到此时，栈内情况如下：

![StackFrame程序删除main函数的栈帧.png](/img/reversecore_assets/StackFrame程序删除main函数的栈帧.png)

图中main()函数和开始栈内情况一样:`00401057 RETN`
执行完上述命令，主要函数执行完毕并返回，程序执行流程跳转到返回地址处(401250)，该地址指向VC++的启动函数区域，随后执行进程终止代码。

## 设置OD选项

打开OD的Debugging options对话框(Alt+O)：

![Disasm选项.png](/img/reversecore_assets/Disasm选项.png)

关闭显示默认段和始终显示内存操作数的大小后，原来代码中显示的默认段和内存大小都不再显示。

![选项变更后的代码窗口.png](/img/reversecore_assets/选项变更后的代码窗口.png)

## Analysis1选项

选择Analysis1选项卡，点击“SHOW ARGs and LOCALs in procedures”左侧的复选框，启用该选项。

原来以EBP表示的函数局部变量、参数分别表示成了LOCAL.1、ARG.1的形式。该选项为代码提供了非常好的可读性，有助于调试代码。
启用该选项后，OllyDbg会直接分析函数的栈帧，然后把局部变量的个数、参数的个数等显示在代码窗口。启用该选项后，虽然偶尔会出现显示错误，但它的显示非常直观，能为调试代码提供帮助。

![局部变量与参数的表示形式.png](/img/reversecore_assets/局部变量与参数的表示形式.png)




