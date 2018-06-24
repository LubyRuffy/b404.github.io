---
title: 'WinSocket——part1'
layout: post
tags:
  - Socket
  - Windows
  - C++
  - code
category: 
  - code
  - C++
comments: true
share: true
description: WinSock是一种标准API，主要用于网络中的数据通信，其允许两个或者多个应用程序（或进程）在同一台机器上或通过网络相互通信。Winsock是一种网络编程，不是协议。
---

WinSock是一种标准API，主要用于网络中的数据通信，其允许两个或者多个应用程序（或进程）在同一台机器上或通过网络相互通信。Winsock是一种网络编程，不是协议。

* TOC
{:toc}

<!--more-->

## 协议模型

![OSI七层协议模型](/img/code/C%2B%2B/WinSocket/OSI%E4%B8%83%E5%B1%82%E6%A8%A1%E5%9E%8B.png)


|协议层名|功能|
|----|----|
|物理硬件层|计算机网络中的物理设备，如网卡等|
|数据链路层|将传输数据进行压缩与解压缩|
|网络层|将传输数据进行网络传输|
|数据传输层|进行信息的网络传输|
|会话层|建立物理网络的连接|
|表示层|将传输数据以某种格式进行表示|
|应用层|应用程序接口|





![TCP-IP协议模型](/img/code/C%2B%2B/WinSocket/TCP-IP%E7%BD%91%E7%BB%9C%E5%8D%8F%E8%AE%AE%E6%A8%A1%E5%9E%8B.png)

![TCP-IP协议模型](/img/code/C%2B%2B/WinSocket/TCP-IP%E5%8D%8F%E8%AE%AE.png)


|协议层名|功能|
|--------|--------|
|数据链路层|网卡等网络硬件设备以及驱动程序|
|网络层|IP协议等互连协议|
|数据传输层|为应用程序提供通信方法，通常为TCP、UDP协议|
|应用层|负责处理应用程序的实际应用层协议|

## 文件操作符

文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。（每当生成文件或套接字，操作系统将返回分配给它们的操作数）

Windows下的文件描述符和信号量、互斥锁等内核对象一样都记作HANDLE。要区分文件句柄和套接字句柄，不像Linux内部也将套接字当做文件。

文件操作符是为了方便称呼OS创建的文件或套接字而赋予的数而已。

## 字节排序

内存中存储字节有两种不同的方法，不同系统采用的存储方式可能不同：

- 小端字节序：将低序字节存储在起始地址
- 大端字节序：将高序字节存储在起始地址

![小端字节序和大端字节序](/img/code/C%2B%2B/WinSocket/%E5%AD%97%E8%8A%82%E6%8E%92%E5%BA%8F.png)

将某给定主机所使用的字节序称为主机字节序。为了使采用不同字节序的主机能够相互通信，TCP/IP协议规定了网络字节序。所有主机或路由器在发送IP数据包之前首先要将相应的信息转换成网络字节序；相应地，在接收数据包之后，要将网络字节序装换为主机字节序。网络字节序为大端序。

> 数据在传输之前不需要手动转化为网络字节序，该过程是自动的。

- htons: 将16位的短整型数从主机字节序转换成网络字节序
- htonl: 将32位的长整形数从主机字节序转换成网络字节序
- ntohs: 将16位的短整型数从网络字节序转换成主机字节序
- ntohl: 将32位的长整形数从网络字节序转换成主机字节序

> h代表host，n代表network,s代表短整型short，l代表长整形long
> 


## 网络地址的初始化与分配

### 将字符串信息转换为网络字节序的整数型

sockaddr_in中保存地址信息的成员为32整数型。因此，为了分配IP地址，需要将其表示为32位整数型数据。


#### inet_addr函数

```cpp
#include<arpa/inet.h>
in_addr_t inet_addr(const char * string)
```

成功时返回32位大端序整数型值，失败时返回INADDR_NONE

#### inet_aton函数

```cpp
int inet_aton(const char* string, struct in_addr * addr);
```

成功时返回1(true),失败时返回0(false).

- string 含有需转换的IP地址信息的字符串地址值
- addr 将保存转换结果的in_addr结构体变量的地址值

实际编程中需要调用inet_addr函数，需将转换后的IP地址信息代入sockaddr_in结构体中声明的in_addr结构体变量。而inet_aton函数不需此过程。原因在于，若传递in_addr结构体变量初始化地址值，函数会自动把结果填入该结构体变量。

#### inet_ntoa函数

```cpp
#include<arpa/inet.h>
char * inet_ntoa(struct in_addr adr);
```

成功时返回转换的字符串地址值，失败时返回-1

该函数将通过参数传入的整数型IP地址转换为字符串格式返回。返回值类型为char指针。返回字符串地址意味着字符串已保存到内存空间，但该函数为向程序员未要求分配内存，而是在内部申请了内存并保存了字符串。也就是，调用完该函数后，应立即将字符串信息复制到其他内存空间。不然，再次调用inet_ntoa函数，则有可能覆盖之前保存的字符串信息。（长期保存，需要将字符串复制到其他内存空间）

### 网络地址初始化

```cpp
struct sockaddr_in addr;
char * serv_ip = "211.231.12.9";//声明ip地址字符串
char * serv_port = "3298";//声明端口号字符串
memset(&addr, 0, sizeof(addr));//结构体变量addr的所有成员初始化为0
addr.sin_family = AF_INET;//指定地址族
addr.sin_addr.s_addr = inet_addr(serv_ip);//基于字符串的IP地址初始化
addr.sin_port = htons(atoi(serv_port));//基于字符串的端口号初始化
```

>  可利用常数INADDR_ANY分配服务器端的IP地址，该方式可自动获取运行服务器端的计算机IP地址。端口指定为0，可以分配唯一的端口号，其值为1024~5000之间。应用程序可以在bind之后使用`getsockname`分配地址


## WinSocket

Windows socket是实现网络程序的比较简单的方法。Socket是连接应用程序与网络驱动程序的桥梁，Socket在应用程序中创建，通过绑定操作与驱动程序建立关系。此后，应用程序送给Socket的数据，由Socket交给驱动程序向网络上发送出去。计算机从网络上收到该Socket绑定的ip地址和端口号相关的数据，由驱动程序交给Socket，应用程序便可以从该Socket中提取接收到的数据。网络应用程序就是这样通过socket进行数据的发送和接收。


### 头文件和库

**WinSocket的头文件和库:**
- 导入头文件winsock32.h
- 链接ws2_32.lib库

### WinSocket初始化

进行WinSocket编程时，首先要调用WSAStartup函数，设置程序中用到的WinSocket版本，并初始化相应版本的库。

```cpp
#include<winsock2.h>
int WSAStartup(WORD wVersionRequested, LPWSADATA lpWSAData)
```

WSAStartup()函数成功时返回0，失败时返回非0的错误代码值。

- wVersionRequested：Winsock版本信息，高8位是副版本号，低8位是主版本号
- lpWSAData:WSADATA结构体变量的地址值，调用完函数后，相应参数中将填充已初始化的库信息。

```cpp
int main(int argc, char* argv[]){
    WSADATA wsadata;
    ...
    if(WSAStartup(MAKEWORD(2, 2), &wsaData) != 0)
        ErrorHandling("WSAStartup() error!");
    ...
    return 0;
}
```

> 以上WSAStartup函数调用过程，几乎成为Winsock编程的公式。


注销Winsock库：

```cpp
#include<winsock2.h>
int WSACleanup(void);
```

WSACleanup成功时，返回0，失败时返回SOCKET_ERROR。调用完之后，释放资源，将Winsock库返还给Windows OS，无法再调用Winsock函数

### 基于Windows的套接字相关函数


#### socket函数

```cpp
#include<winsock2.h>
SOCKET socket(int af, int type, int protocol);
```

成功时，返回套接字句柄，失败时返回INVALID_SOCKET。

#### bind函数

```cpp
#include<winwock2.h>
int bind(SOCKET s, const struct sockaddr * name, int namelen);
```

成功时，返回0，失败时返回SOCKET_ERROR。

#### listen函数

```cpp
#include<winsock2.h>
int listen(SOCKET s, int backlog)
```

成功时，返回0，失败时返回SOCKET_ERROR

#### accept函数

```cpp
#include<winsock2.h>
SOCKET accept(SOCKET s, struct sockaddr * addr, int * addrlen)
```

成功时返回套接字句柄，失败时返回INVALID_SOCKET.

#### connect函数

```cpp
#include<winsock2.h>
int connect(SOCKET s, const struct sockaddr * name, int namelen);
```

成功时返回0，失败时返回SOCKET_ERROR

#### closesocket函数

```cpp
#include<winsock2.h>
int closesocket(SOCKET s);
```

成功时返回0,失败时返回SOCKET_ERROR


### 基于Windows的I/O函数

Linux中套接字也是文件，因为可以通过文件I/O函数read和write进行数据传输。而Windows严格区分文件I/O函数和套接字I/O函数。

#### send函数

```cpp
#include<winsock2.h>
int send(SOCKET s, const char * buf, int len, int flags );
```

成功时返回传输字节数，失败时返回SOCKET_ERROR

- s表示数据传输对象连接的套接字句柄值
- buf保存待传输数据的缓冲地址值
- len要传输的字节数
- flags传输数据时用到的多种选项信息


#### recv函数

```cpp
#include<winsock2.h>
int recv(SOCKET s, const char * buf, int len, int flags);
```

成功时返回接收的字节数(收到EOF时为0)，失败时返回SOCKET_ERROR。

- s 表示数据接收对象连接的套接字句柄值
- buf 保存接收数据的缓冲地址值
- len 能够接收的最大字节数
- flags接收数据时用到的多种选项


Windows socket是实现网络程序的比较简单的方法。Socket是连接应用程序与网络驱动程序的桥梁，Socket在应用程序中创建，通过绑定操作与驱动程序建立关系。此后，应用程序送给Socket的数据，由Socket交给驱动程序向网络上发送出去。计算机从网络上收到该Socket绑定的ip地址和端口号相关的数据，由驱动程序交给Socket，应用程序便可以从该Socket中提取接收到的数据。网络应用程序就是这样通过socket进行数据的发送和接收。

Windows Socket只支持一个通信区域：网际域（AF_INET），这个域被使用网际协议簇通信的进程使用。

套接字的类型：

- 流式套接字(SOCK_STREAM)：提供面向连接、可靠的数据传输服务(基于TCP) 
- 数据报套接字(SOCK_DGRAM)：提供面向无连接服务（基于UDP）
- 原始套接字（SOCK_RAW）

### TCP的socket编程

服务端：

- 创建套接字（socket）
- 将套接字绑定到一个本地地址和端口上（bind）
- 将套接字设为监听模式，准备接收客户请求(listen)
- 等待客户请求到来；当请求到来之后，接受请求，返回一个新的对应于此次连接的套接字(accept)
- 用返回的套接字和客户端进行通信(send/recv)
- 返回等待另一客户请求
- 关闭套接字


![TCP服务器和客户程序连接过程](/img/code/C%2B%2B/WinSocket/TCP%E6%9C%8D%E5%8A%A1%E5%99%A8%E7%A8%8B%E5%BA%8F%E5%92%8C%E5%AE%A2%E6%88%B7%E7%A8%8B%E5%BA%8F%E7%9A%84%E5%88%9B%E5%BB%BA.png)

```cpp
#include<iostream>
#include<Winsock2.h>
#include<stdio.h>

#pragma comment(lib, "ws2_32.lib")

void main() {

    using namespace std;
    //WSAStartup函数，加载套接字库，进行套接字库版本协商
    WORD wVersionRequested;//保存WinSock库的版本号
    WSADATA wsaData; //指向WSADATA结构的指针，用于返回Socket库初始化信息

    int err;
    wVersionRequested = MAKEWORD(1, 1);//调用MAKEWORD宏创建一个请求版本号的word值
    err = WSAStartup(wVersionRequested, &wsaData);//WSAStartup函数，加载套接字库，进行套接字库版本协商
    if (err != 0) {

        return;
    }
    if (LOBYTE(wsaData.wVersion) != 1 || HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    //创建套接字(socket)
    //sockSrv用于接收socket返回的套接字
    SOCKET sockSrv = socket(AF_INET, SOCK_STREAM, 0);//socket函数将根据地址格式和套接字类别，自动选择合适的协议

    //绑定 套接字到ip和端口(bind)
    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);

    //通过bind函数将套接字sockSrv绑定到本地地址和端口
    //bind函数的第一个参数是绑定的套接字
    //bind函数的第二个参数需要一个指针，可以用取地址符实现
    //bind函数的第三个参数是指定地址结构的大小，利用sizeof操作符获取
    bind(sockSrv, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR));
    //将套接字设为监听模式(listen)，第一个是将要设置的套接字，第二个是等待连接队列的最大长度
    listen(sockSrv, 5);
    
    SOCKADDR_IN addrClient;//定义变量，用于接收客户端地址信息
    int len = sizeof(SOCKADDR);//初始化SOCKADDR_IN结构体长度，否则调用accept函数失败
    //不断循环，等待客户端的连接请求，接受请求之后返回一个新的对应于此次连接的套接字(accept)
    while (1) {
        //accept函数的第一个参数是处于监听状态的套接字
        //其第二个参数是利用addrClient变量接收客户端的地址信息
        //利用sockConn与客户端进行通信，先前的套接字仍继续监听客户端的连接请求
        SOCKET sockConn = accept(sockSrv, (SOCKADDR*)&addrClient, &len);//返回的套接字描述符保存于sockConn变量
        char sendBuf[100];//定义数组，将客户端的地址进行格式化处理后放于此
        //使用inet_ntoa函数，该函数接受一个in_addr结构体类型的参数并返回一个以点分十进制格式表示的IP地址字符串
        //SOCKADDR_IN结构体中的sin_addr成员是in_addr类型，刚好可作为参数传递
        sprintf(sendBuf, "Welcome %s to Test!", inet_ntoa(addrClient.sin_addr));
        //用返回的套接字和客户端进行通信(send/recv)
        //使用建立连接的套接字sockConn，而不是监听的套接字
        send(sockConn, sendBuf, strlen(sendBuf) + 1, 0);//多发送一个字节，主要是为了在该数据字符串之后加"\0"结尾标识符
        //在发送数据之后，还可以用recvBuf从客户端接收数据
        char recvBuf[100];//定义保存接收数据的数组
        recv(sockConn, recvBuf, 100, 0);
        //接收到数据之后，打印接收到的数据
        cout << recvBuf << endl;
        //system("pause");
        //关闭套接字，释放分配的资源，进入下一个循环，等待客户端的请求
        /*本例是一个死循环，如果不是一个死循环，在closesocket函数调用之后
        还需关闭监听套接字，并调用WSACleanup函数终止对套接字库的调用*/
        closesocket(sockConn);
    }
}
```

![](/img/code/C%2B%2B/WinSocket/1501835013336.png)
![](/img/code/C%2B%2B/WinSocket/1501835035872.png)
![](/img/code/C%2B%2B/WinSocket/1501835062181.png)




客户端：

- 创建套接字(socket)
- 向服务器发出连接请求(connect)
- 和服务器端通信(send/recv)
- 关闭套接字

```cpp
#include<iostream>
#include<Winsock2.h>
#include<stdio.h>

#pragma comment(lib, "ws2_32.lib")

void main() {

    using namespace std;
    //WSAStartup函数，加载套接字库，进行套接字库版本协商
    WORD wVersionRequested;//保存WinSock库的版本号
    WSADATA wsaData;//指向WSADATA结构的指针，用于返回Socket库初始化信息
    int err;

    wVersionRequested = MAKEWORD(1, 1); //调用MAKEWORD宏创建一个请求版本号的word值
    err = WSAStartup(wVersionRequested, &wsaData);//WSAStartup函数，加载套接字库，进行套接字库版本协商
    if (err != 0) {

        return;
    }

    if (LOBYTE(wsaData.wVersion) != 1 || HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    //创建套接字
    SOCKET sockClient = socket(AF_INET, SOCK_STREAM, 0);//socket函数将根据地址格式和套接字类别，自动选择合适的协议
    //绑定套接字到ip和端口上
    /*对于客户端来说，不需要绑定，可以直接连接服务器端  */
   //定义一个地址结构体，设定服务器的ip和端口
    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);

    connect(sockClient, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR));
    //和服务端通信(sen/recv)
    char recvBuf[100];//定义有100个字节的字符数组
    recv(sockClient, recvBuf, 100, 0);//接收服务器的数据
    cout << recvBuf << endl;
    //发送数据到服务端,多发送一个字节，服务器接收到数据后可以将最后一个字节的数据设置为"\0"，表示字符串的结束
    send(sockClient, "This is lisi", strlen("This is lisi") + 1, 0);
    system("pause");
    //关闭套接字，释放资源
    closesocket(sockClient);
    //调用WSACleanup函数，终止对套接字库的使用
    WSACleanup();
}
```

![](/img/code/C%2B%2B/Wi1501835664905.png)



结果：

![](/img/code/C%2B%2B/WinSocket/1501834880518.png)

### UDP的Socket套接字编程

接收端：

- 创建套接字(socket)
- 将套接字绑定到一个本地地址和端口上(bind)
- 等待接收数据(recvfrom)
- 关闭套接字

```cpp
#include<Winsock2.h>
#include<stdio.h>
#include<iostream>
#pragma comment(lib,"ws2_32.lib")


void main() {
    using namespace std;
    //加载套接字库
    WORD wVersionRequested;
    WSADATA wsaData;
    int err;
    wVersionRequested = MAKEWORD(1, 1);
    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;
    }
    if (LOBYTE(wsaData.wVersion) != 1 || HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }
    //创建套接字
    SOCKET sockSrv = socket(AF_INET, SOCK_DGRAM, 0);
    //
    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);
    bind(sockSrv, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR));
    SOCKADDR_IN addrClient;
    int len = sizeof(SOCKADDR);
    char recvBuf[100];
    recvfrom(sockSrv, recvBuf, 100, 0, (SOCKADDR*)&addrClient, &len);
    cout << recvBuf << endl;
    system("pause");
    //关闭套接字
    closesocket(sockSrv);
    WSACleanup();
}
```

发送端：
- 创建套接字(socket)
- 向服务器发送数据(sendto)
- 关闭套接字

```cpp
#include<winsock2.h>
#include<iostream>
#pragma comment(lib, "ws2_32.lib")

void main() {

    WORD wVersionRequested;
    WSADATA wsaData;
    int err;

    wVersionRequested = MAKEWORD(1, 1);
    err = WSAStartup(wVersionRequested, &wsaData);
    if (err != 0) {
        return;

    }
    if (LOBYTE(wsaData.wVersion) != 1 || HIBYTE(wsaData.wVersion) != 1) {
        WSACleanup();
        return;
    }

    SOCKET sockClient = socket(AF_INET, SOCK_DGRAM, 0);
    SOCKADDR_IN addrSrv;
    addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
    addrSrv.sin_family = AF_INET;
    addrSrv.sin_port = htons(6000);

    sendto(sockClient, "Hello", strlen("Hello") + 1, 0, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR));
    system("pause");
    closesocket(sockClient);
    WSACleanup();
}
```


结果：
![](/img/code/C%2B%2B/WinSocket/1501839582074.png)




