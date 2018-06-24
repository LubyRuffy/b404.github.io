---
title: PE之HelloWorld的PE文件格式解析
layout: post
tags:
  - reading-notes
  - reverse
category: ReverseCore
comments: true
share: true
description: 汇编编写HelloWorld,解析HelloWorld的PE文件格式
---


汇编编写HelloWorld,解析HelloWorld的PE文件格式

* TOC
{:toc}

<!--more-->



## 编写HelloWorld

编写HelloWorld的汇编代码，使用NASM编译成obj文件，使用链接器链接成exe文件：

![HelloWorld_NASM.png](/img/reversecore_assets/PE文件/HelloWorld_NASM.png)

## 在WinHex中打开exe文件：

![HelloWorld_PE_1.png](/img/reversecore_assets/PE文件/HelloWorld_PE_1.png)


![HelloWorld_PE_2.png](/img/reversecore_assets/PE文件/HelloWorld_PE_2.png)


## PE结构

![PE结构.png](/img/reversecore_assets/PE文件/PE结构.png)

标准PE文件一般由四大部分组成：
- DOS头
- PE头(IMAGE_NT_HEADERS)
  - 4个字节的标识符(Signature)
  - 20个字节的基本头信息(IMAGE_FILE_HEADER)
  - 216个字节的扩展头信息(IMAGE_OPTIONAL_HEADER32)
- 节表(多个IMAGE_SECTION_HEADER结构)
- 节内容（导入表、导出表、资源表、重定位表）

## DOS头

```cpp
typedef struct _IMAGE_DOS_HEADER {　　　　// DOS .EXE header
　　WORD　 e_magic;　　　　　　　　　　　　　// 0000h;魔术字"MZ"
　　WORD　 e_cblp;　　　　　　　　　　　　　 // 0002h;最后页中的字节数
　　WORD　 e_cp;　　　　　　　　　　　　　　 // 0004h;文件中的全部和部分页数
　　WORD　 e_crlc;　　　　　　　　　　　　　 // 0006h;重定位表中的指针数
　　WORD　 e_cparhdr;　　　　　　　　　　　 // 0008h;头部尺寸，以段落为单位
　　WORD　 e_minalloc;　　　　　　　　　　  // 000ah;所需的最小附加段
　　WORD　 e_maxalloc;　　　　　　　　　　  // 000ch;所需的最大附加段
　　WORD　 e_ss;　　　　　　　　　　　　    // 000eh;初始的SS值(相对偏移量)
　　WORD　 e_sp;　　　　　　　　　　　　    // 0010h;初始的SP值
　　WORD　 e_csum;　　　　　　　　　　　　  // 0012h;补码校验值
　　WORD　 e_ip;　　　　　　　　　　　　    // 0014h;初始IP值
　　WORD　 e_cs;　　　　　　　　　　　　    // 0016h;初始CS值
　　WORD　 e_lfarlc;　　　　　　　　　　   // 0018h;重定位表的字节偏移量
　　WORD　 e_ovno;　　　　　　　　　　　　  // 001ah;覆盖号
　　WORD　 e_res[4];　　　　　　　　　　   // 001ch;保留字
　　WORD　 e_oemid;　　　　　　　　　　    // 0024h;OEM 标识符
　　WORD　 e_oeminfo;　　　　　　　　　　  // 0026h;OEM 信息
　　WORD　 e_res2[10];　　　　　　　　　　 // 0028h;保留字
　　DWORD　e_lfanew;　　　　　　　　　　   // 003CH;PE头相对文件的偏移地址
　} IMAGE_DOS_HEADER, *PIMAGE_DOS_HEADER;
```

## PE头的IMAGE_FILE_HEADER

PE头的IMAGE_FILE_HEADER紧跟在PE头标识后，即位于IMAGE_DOS_HEADER的e_lfanew值+4的位置。由此位置开始的20个字节为数据结构标准PE头`IMAGE_FILE_HEADER`的内容。该结构为标准通用对象文件格式(COFF)头，记录了PE文件的全局属性，如该PE文件运行的平台、PE文件类型、文件中存在的节的总数等。

```cpp
typedef struct _IMAGE_FILE_HEADER {
　　WORD　　Machine;   //0004h;运行平台
　　WORD　　NumberOfSections; //0006h;PE中节的数量
　　DWORD　 TimeDateStamp; //0008h;文件创建日期和时间
　　DWORD　 PointerToSymbolTable; //000ch;指向符号表(用于调试)
　　DWORD　 NumberOfSymbols; //0010h;符号表中的符号数量(用于调试)
　　WORD　　SizeOfOptionalHeader; //0014h;扩展头结构的长度
　　WORD　　Characteristics; //0016h;文件属性
} IMAGE_FILE_HEADER, *PIMAGE_FILE_HEADER;

#define IMAGE_SIZEOF_FILE_HEADER　　　　　　 20
```

## 扩展PE头IMAGE_OPTIONAL_HEADER

文件执行时的入口地址、文件被操作系统装入内存后的默认基地址，以及节在磁盘和内存中的对齐单位等信息均可以在此结构中找到。对该结构中的某些数值的随意改动可能会造成PE文件的加载或运行失败。

```cpp
//
// Optional header format.
//

typedef struct _IMAGE_OPTIONAL_HEADER {
　　//
　　// Standard fields.
　　//

　　WORD　　Magic;  //0018h;魔术字;107h=ROM Image;10Bh=exe Image
　　BYTE　　MajorLinkerVersion; //001ah;链接器版本号
　　BYTE　　MinorLinkerVersion; //001bh
　　DWORD　 SizeOfCode; //001ch 所有含代码的节的总大小
　　DWORD　 SizeOfInitializedData; //0020h 所有含代码已初始化数据的节的总大小
　　DWORD　 SizeOfUninitializedData; //0024h 所有含未初始化数据的节的大小
　　DWORD　 AddressOfEntryPoint; //0028h 程序执行入口RVA
　　DWORD　 BaseOfCode; //002ch 代码的节的起始RVA
　　DWORD　 BaseOfData; //0030h 数据的节的起始RVA

　　//
　　// NT additional fields.
　　//

　　DWORD　 ImageBase; //0034h 程序的建议装载地址
　　DWORD　 SectionAlignment; //0038h 内存中的节的对齐粒度
　　DWORD　 FileAlignment; //003ch 文件中的节的对齐粒度
　　WORD　　MajorOperatingSystemVersion; //0040h 操作系统版本号
　　WORD　　MinorOperatingSystemVersion; //0042h
　　WORD　　MajorImageVersion; //00044h,该PE的版本号
　　WORD　　MinorImageVersion; 
　　WORD　　MajorSubsystemVersion; //0048h,所需子系统版本号
　　WORD　　MinorSubsystemVersion;
　　DWORD　 Win32VersionValue; //未用
　　DWORD　 SizeOfImage; //0050h;内存中的整个PE映像尺寸
　　DWORD　 SizeOfHeaders; //0054h;所有头+节表的大小
　　DWORD　 CheckSum; //0058h;校验和
　　WORD　　Subsystem; //005ch;文件中的子系统
　　WORD　　DllCharacteristics; //005eh;DLL文件特性
　　DWORD　 SizeOfStackReserve; //0060h;初始化时的栈大小
　　DWORD　 SizeOfStackCommit; //0064h;初始化时实际提交的栈大小
　　DWORD　 SizeOfHeapReserve; //0068h;初始化时保留的堆大小
　　DWORD　 SizeOfHeapCommit; //006ch;初始化时实际提交的堆大小
　　DWORD　 LoaderFlags; //0070h;与调试有关
　　DWORD　 NumberOfRvaAndSizes; //0078h;下面的数据目录结构的项目数量
　　IMAGE_DATA_DIRECTORY DataDirectory[IMAGE_NUMBEROF_DIRECTORY_ENTRIES]; //数据目录
} IMAGE_OPTIONAL_HEADER32, *PIMAGE_OPTIONAL_HEADER32;
```

![NT头结构.png](/img/reversecore_assets/PE文件/NT头结构.png)



## 数据目录表项


![PE_IMAGE_Data_Directory.png](/img/reversecore_assets/PE文件/PE_IMAGE_Data_Directory.png)



## 节表IMAGE_SECTION_HEADER

```cpp
typedef struct _IMAGE_SECTION_HEADER {
　　BYTE　　Name[IMAGE_SIZEOF_SHORT_NAME];
　　union {
　　　　　　DWORD　 PhysicalAddress;
　　　　　　DWORD　 VirtualSize; //实际的节表项大小(不一定是对齐后的值)
　　} Misc;
　　DWORD　 VirtualAddress; //节表载入内存后的相对虚拟地址(按内存对齐)
　　DWORD　 SizeOfRawData; //在磁盘中该节表项的大小(通常为对齐后的值)
　　DWORD　 PointerToRawData;//该节表项在磁盘上的偏移地址
　　DWORD　 PointerToRelocations;
　　DWORD　 PointerToLinenumbers;
　　WORD　　NumberOfRelocations;
　　WORD　　NumberOfLinenumbers;
　　DWORD　 Characteristics; //节表属性
} IMAGE_SECTION_HEADER, *PIMAGE_SECTION_HEADER;

#define IMAGE_SIZEOF_SECTION_HEADER　　　　　40
```

## Notepad例子


![NotepadPE.png](/img/reversecore_assets/PE文件/NotepadPE.png)


![Notepad_Image_optional_header.png](/img/reversecore_assets/PE文件/Notepad_Image_optional_header.png)


![Notepad_Image_optional_header32_结构体成员.png](/img/reversecore_assets/PE文件/Notepad_Image_optional_header32_结构体成员.png)







## 参考

1.[[C++ 黑客编程揭秘与防范（第2版）]](http://www.epubit.com.cn/book/onlinechapter/30801)http://www.epubit.com.cn/book/onlinechapter/30801
2.WindowsPE指南