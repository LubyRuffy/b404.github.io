---
title: 编译和链接
layout: post
tags:
  - reading-notes
  - build
category: Misc
comments: true
share: true
description: 编译和链接一步完成叫做构建(build).一般的代码成为可执行文件，需要经过预处理、编译、汇编、链接几个步骤
---
编译和链接一步完成叫做构建(build).一般的代码成为可执行文件，需要经过预处理、编译、汇编、链接几个步骤

* TOC
{:toc}

<!--more-->

![gcc编译过程分解.png](/img/os/gcc编译过程分解.png)

## 预编译

预编译主要处理那些源文件中以`#`开始的预编译指令。比如`#include`、`#define`.主要处理规则为：
- 将所有的`#define`删除，并且展开所有的宏定义
- 处理所有条件预编译指令，比如`#if`、`#ifdef`、`elif`、`#else`、`#endif`
- 处理`#include`预编译指令，将包含的文件插入到该预编译指令的位置(递归过程)
- 删除所有注释符
- 添加行号和文件名标识，以便调试或显示错误行号
- 保留所有`#pragma`（编译器需要）


## 编译

编译过程就是将预处理完的文件进行一系列词法分析、语法分析、语义分析以及优化后生产相应的汇编代码文件。

## 汇编

汇编器将汇编代码转变为机器可执行的指令，每条汇编语句大概对应一条机器指令。

## 链接

链接通常是一个通过将库文件、目标文件等文件组装产生一个可执行文件的过程。（解决符号依赖，库依赖关系）

链接的主要内容就是将各模块之间相互引用的部分处理好，使得各模块之间能正确衔接，将一些指令对其他符号地址的引用加以修改。

链接过程主要包括地址和空间分配（Address and Storge Allocation）、符号决议（Symbol Resolution）和重定位（Relocation）等步骤。


![链接过程.png](/img/os/链接过程.png)





## 编译器的工作

![编译过程.png](/img/os/编译过程.png)

### 词法分析

源码程序输入到扫描器，扫描器(Scanner)进行语法分析，运行一种；类似于有限状态机(Finite State Machine)的算法，轻松将源码字符序列分割成记号（Token,关键字、标识符、字面量、特殊符号）

### 语法分析

语法分析器（Grammer Parser）将对由扫描器产生的记号进行语法分析，从而产生语法树。整个分析过程采用了上下文无关语法(Context-free Grammar)的分析手段。

由语法分析器生成的语法树就是以表达式(Expression)为节点的树。

![语法树.png](/img/os/\语法树.png)

### 语义分析

语义分析器进行语义分析，整个语法树的表达式都被标识了类型，若有些类型需做隐式转换，语义分析程序就会在语法树插入相应的转换节点。

语法分析仅仅完成了对表达式的语法层面的分析。


![标识后的语法树.png](/img/os/标识后的语法树.png)

### 中间语言生成

源码优化器将整个语法树变换成中间代码，中间码是语法树的顺序表示。中间码很接近目标代码。

中间码让编译器分为前后端。编译器前端负责产生机器无关的中间代码，编译器后端将中间代码转换成目标机器代码。跨平台的编译器可针对不同平台使用同一个前端和针对不同机器平台的数个后端。


![优化后的语法树.png](/img/os/优化后的语法树.png)

### 目标代码生成

源码级优化器产生中间码标志着下面的过程都属于编译器后端。编译器后端主要包括代码生成器和目标代码优化器。
代码器生成器将中间代码转换成目标机器码。（依赖于目标机器，不同机器有不同的字长、寄存器、数据类型）

目标代码优化器会对目标代码进行优化，比如选择合适的寻址方式、使用位移代替乘法、删除多余指令等。