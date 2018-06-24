---
title: 基址重定位表
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
description: 向进程的虚拟内存中加载PE文件(EXE/DLL/SYS)时，文件会被加载到PE头的ImageBase所指的地址处。若加载的是DLL(SYS)文件，且在ImageBase位置处已经加载了其他DLL（SYS）文件，那么PE装载器就会将其加载到其他未被占用的空间。
---

向进程的虚拟内存中加载PE文件(EXE/DLL/SYS)时，文件会被加载到PE头的ImageBase所指的地址处。若加载的是DLL(SYS)文件，且在ImageBase位置处已经加载了其他DLL（SYS）文件，那么PE装载器就会将其加载到其他未被占用的空间。

* TOC
{:toc}

<!--more-->

## 基址重定位表

**PE重定位：**PE文件无法加载到ImageBase所指位置，而是被加载到其他地址时发生的一系列的处理行为。（硬编码在成 ）

<small>SDK或VC++创建PE文件，EXE的默认基址为`00400000`,DLL默认的基址为`10000000`;DDK创建的SYS文件默认基址为`10000`</small>

PE重定位的基本操作原理：
- 在应用程序中查找硬编码的地址位置
- 读取值后，减去ImageBase（VA->RVA）
- 加上实际加载地址（RVA->VA）

其中最关键的是查找硬编码地址的位置，查找过程中会用到PE文件内部的Relocation Table（重定位表，记录硬编码地址偏移的列表，在PE文件构建过程中的编译和链接中提供）。通过重定位表查找，就是根据PE头的”基址重定位表“项进行的查找。

基地址重定位表地址位于PE头的DataDirectory的第六个元素（索引值为5）：

`IMAGE_NT_HEADERS\IMAGE_OPTIONAL_HEADER\IMAGE_DATA_DIRECTORY[5]`

> 重定位表是一个数组，这个数组的大小记载在 _IMAGE_OPTIONAL_HEADER 的.DataDirectory[IMAGE_DIRECTORY_TNTRY_BASERELOC].Size 成员中


基址重定位表中罗列了硬编码地址的偏移，基址重定位表是IMAGE_BASE_RELOCATION结构体数组。根据IMAGE_BASE_RELOCATION结构体可以获得准确的硬编码地址偏移。
IMAGE_BASE_RELOCATION结构体定义如下：


```C++
// 【基址重定位位于数据目录表的第六项，共8 + N字节】  
typedef struct _IMAGE_BASE_RELOCATION  
{  
    DWORD VirtualAddress; //重定位数据开始的RVA 地址
    DWORD SizeOfBlock;    //重定位块得长度，标识重定向字段个数
    WORD TypeOffset;      //重定项位数组相对虚拟RVA, 个数动态分配
}IMAGE_BASE_RELOCATION, *PIMAGE_BASE_RELOCATION;
#define IMAGE_REL_BASED_ABSOLUTE        0
#define IMAGE_REL_BASED_HIGH            1
#define IMAGE_REL_BASED_LOW             2
#define IMAGE_REL_BASED_HIGHLOW         3
#define IMAGE_REL_BASED_HIGHADJ         4
#define IMAGE_REL_BASED_MIPS_JMPADDR    5
#define IMAGE_REL_BASED_MIPS_JMPADDR16  9
#define IMAGE_REL_BASED_IA64_IMM64      9
#define IMAGE_REL_BASED_DIR64           10
```
IMAGE_BASE_RELOCATION结构体的第一个成员为VirtualAddress,它是一个基准地址(BaseAddress)，实际是RVA值。第二个成员为SizeOfBlock，指重定位块的大小。最后一项TypeOffset数组不是结构体成员，而是以注释形式存在的，表示在该结构体之下会出现WORD类型的数组，并且该数组元素的值就是硬编码在程序中的地址偏移


![基址重定位表.png](/img/reversecore_assets/PE重定位表地址/基址重定位表.png)

![基址重定位表2.png](/img/reversecore_assets/PE重定位表地址/基址重定位表2.png)

由IMAGE_BASE_RELOCATION结构体定义可知，VirtualAddress成员（基准地址）的值为1000，SizeOfBlock成员的值为150.即TypeOffset数组的基准地址（起始地址）为RVA 1000，块的总大小为150.块的末端显示为0.TypeOffset值为2个字节（16位）大小，是由4位的Type与12位的Offset合成。

如TypeOffset值为3420，解析如表：


| 类型(4位)| 偏移（12位） |
|--------|--------|
| 3       | 420       |

高4位用作type，PE文件中常见的值为3（IMAGE_REL_BASED_HIGHLOW），64位的PE+文件中常见值为A（IMAGE_REL_BASED_DIR64）

> 恶意文件中正常修改文件代码后，有时要修改指向相应区域的重定位表（为了略去PE装载器的重定位过程，常常把Type修改为0(IMAGE_REL_BASED_ABSOLUTE)））

TypeOffset的低12位是真正的位移，该位移 值基于Virtual Address偏移。所以程序中的硬编码地址的偏移使用下面等式换算：

VirtualAddress（1000）+Offset（420）=1420（RVA）

TypeOffset中指向位移的低3Byte可表示的最大地址为0x1000，为了表示更大的地址，要添加一个与其对应的块，由于这些块以数组形式罗列，故称为重定位表
若TypeOffset为0，则表明一个IMAGE_BASE_RELOCATION结构体结束

重定位表以NULL结构体结束，即IMAGE_BASE_RELOCATION结构体成员的值全部为NULL


## 删除.reloc节区

EXE形式的PE文件中，基址重定位表对运行没什么影响。将其删除后程序仍然正常运行（基址重定位表对DLL/SYS形式的文件来说几乎是必须的0）

VC++中生成的PE文件的重定位节区名为.reloc，删除该节区后文件照常运行，且文件大小将缩减

准确删除位于文件末尾的.reloc节区步骤：

- 删除.reloc节区头
- 删除.reloc节区
- 修改IMAGE_FILE_HEADER
- 修改IMAGE_OPTIONAL_HEADER


