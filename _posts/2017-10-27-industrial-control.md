---
title: 工控
layout: post
tags:
  - sec
  - hack
  - IndustrialControl
category: 
  - hack
  - sec
comments: true
share: true
description: 工控指的是工业自动化控制，主要利用电子电气、机械、软件组合实现。
---

工控指的是工业自动化控制，主要利用电子电气、机械、软件组合实现。

* TOC
{:toc}

<!--more-->


## 模块组件

大组件：

- SCADA：数据采集与监视控制系统
- ICS：工业控制系统
- DCS：分布式控制系统/集散控制系统
- PCS：过程控制系统
- ESD：应急停车系统
- PLC：可编程序控制器(Programmable Logic Controller)
- RTU：远程终端控制系统
- IED：智能监测单元
- HMI：人机界面(Human Machine Interface)
- MIS：管理信息系统(Management Information System)
- SIS： 生产过程自动化监控和管理系统（Supervisory Information System）
- MES：制造执行管理系统

![](/img/hack/工控/1509094786571.png)

小组件：

- OS：Windows/VxWorks/QNX/Linux/Wince
- Service: Web/telnet/OPC

## 协议 

### Modbus

MODBUS协议定义了一个与基础通信层无关的简单协议数据单元（PDU）。特定总线或网络上的MODBUS协议映射能够在应用数据单元（ADU）上引入一些附加域。

![](/img/hack/工控//1509093268488.png)

安全问题：

- 缺乏认证：仅需要使用一个合法的Modbus地址和合法的功能码即可以建立一个Modbus会话
- 缺乏授权：没有基于角色的访问控制机制， 任意用户可以执行任意的功能。
- 缺乏加密：地址和命令明文传输， 可以很容易地捕获和解析

### PROFIBUS

PROFIBUS 一种用于工厂自动化车间级监控和现场设备层数据通信与控制的现场总线技术，可实现现场设备层到车间级监控的分散式数字控制和现场通信网络

### DNP3 

DNP3 DNP(Distributed Network Protocol,分布式网络协议)是一种应用于自动化组件之间的通讯协议，常见于电力、水处理等行业。简化OSI模型，只包含了物理层，数据层与应用层的体系结构（EPA）。SCADA可以使用DNP协议与主站、RTU、及IED进行通讯。

### ICCP

ICCP 电力控制中心通讯协议。

### OPC

OPC 过程控制的OLE （OLE for Process Control）。OPC包括一整套接口、属性和方法的标准集，用于过程控制和制造业自动化系统。

### BACnet

BACnet 楼宇自动控制网络数据通讯协议（A Data Communication Protocol for Building Automation and Control Networks）。BACnet 协议是为计算机控制采暖、制冷、空调HVAC系统和其他建筑物设备系统定义服务和协议

### CIP

CIP 通用工业协议，被deviceNet、ControINet、EtherNet/IP三种网络所采用。

### Siemens S7

Siemens S7 属于第7层的协议，用于西门子设备之间进行交换数据，通过TSAP，可加载MPI,DP,以太网等不同物理结构总线或网络上，PLC一般可以通过封装好的通讯功能块实现。

### 其他工控协议

其他工控协议IEC 60870-5-104、EtherNet/IP、Tridium Niagara Fox、Crimson V3、OMRON FINS、PCWorx、ProConOs、MELSEC-Q

## 信息探测

### 搜索引擎

- shodan
- zoomeye
- fofa

### Ethernet/IP

Port:44818

![](/img/hack/工控/1509095674543.png)

![](/img/hack/工控/1509098107632.png)

- Nmap: enip-enumerate.nse
- Python: https://github.com/paperwork/pyenip
- Wireshark dissector
   - src/epan/dissector/packet-etherip.c

### Modbus

- port:502
- 抓取设备相关信息
 - Nmap: modicon-info.nse
 - Wireshark dissector
     - src/epan/dissector/packet-mbtcp.c
- 认证与加密缺失

![](/img/hack/工控/1509251068674.png)

### IEC 61870-5-101/104

- Port: 2404
- 判断是否使用该协议
  - Nmap: iec-identify.nse
  - Wireshark dissector
     - src/epan/dissectors/packet-iec104.c
- 认证与加密缺失


![](/img/hack/工控/1509251319305.png)


### Siemens S7

- key:Siemens S7 PLC（102）
- 抓取设备相关信息
 - Nmap: s7-enumerate_.nse
     - http://plcscan.org/blog/wp-content/uploads/2014/11/s7-enumerate.nse_.txt
 - Wireshark plugins
     - http://sourceforge.net/projects/s7commwireshark/
- 弱加密，易破解

![](/img/hack/工控/1509251472510.png)

### Tridium Niagara Fox

- port:1911
- 抓取设备信息
  - Nmap: fox-info.nse 

## 架构缺陷

- 未经真正安全考验的组件与协议
  - 若有Web服务，那就利用Web服务该有的缺陷
  - 经典渗透技巧大多适用于工控网络
- 稳定性优先+懒 → 升级难
- 互联网的便利性，让工控组件暴露在网络空间里
- 以协议研究为出发点

### 工控系统的指纹识别技术

http://plcscan.org/blog/2017/03/fingerprint-identification-technology-of-industrial-control-system/


### Nmap脚本

Nmap中关于工控协议和设备识别的部分NSE Scripts：

|NSE Scripts|说明|
|----|----|
|modbus-discover.nse|Modbus TCP设备发现脚本，该脚本可以调用Modbus 43（2B功能码）功能码读取设备信息|
|modbus-enum.nse|Modbus TCP设备枚举脚本）|
|s7-enumerate.nse|西门子S7 PLC设备发现脚本，可以枚举PLC的一些基本信息|
|enip-enumerate.nse|可以读取EtherNet/IP设备的基本信息|
|BACnet-discover-enumerate.nse|可以读取BACnet设备的基本信息|
|iec-identify.nse|IEC104协议asdu address枚举脚本|
|mms-identify.nse|IEC-61850-8-1协议信息枚举脚本|

测试脚本：

- https://github.com/atimorin/scada-tools  
- https://github.com/atimorin/PoC2013 
- https://github.com/drainware/scada-tools 
- https://github.com/drainware/nmap-scada

Exploit-db测试脚本：

- https://www.exploit-db.com/exploits/19833/ https://www.exploit-db.com/exploits/19832/ https://www.exploit-db.com/exploits/19831/ 
-  https://www.exploit-db.com/search/?action=search&description=scada&e_author=



### 入侵

很多工控沦陷是从Web开始，Web服务杂乱充斥在工控架构中，容易被攻击。剖析完工控架构后，根据架构，制定攻击思路攻击

S7 PLC自带嵌入式Web服务：

![](/img/hack/工控/1509323694990.png)

![](/img/hack/工控/1509323980415.png)

海康威视DVR监控：

![](/img/hack/工控/1509324001775.png)

Modbus协议直接连接：

![](/img/hack/工控/1509324030486.png)





## Misc

挖掘此类漏洞主要解决两个问题

- 如何找到工控相关的系统和地址
- Getshell后，基于工控知识如何操控系统

根据漏洞中的细节可以进一步的复测和拓展，进而为工控系统的漏洞挖掘提供非线性思路。
- 结合GHDB关键字的搜素：例如inurl:SCADA……
- 链接地址含SCADA、Modbus等协议的关键字……
- 其他KEY：MIS、SIS、DCS、PLC、ICS、监控系统……

### 资源

- 国外工控：http://plcscan.org/blog/2016/03/ics-security-resources-overview-2/
- Exploit   PLC on  the internet：https://paper.seebug.org/papers/Security%20Conf/KCon/2015/Exploit%20PLC%20on%20the%20internet.pdf
- Internet-facing PLCs as a Network Backdoor：https://www.inf.fu-berlin.de/groups/ag-si/pub/plc_backdoor_spicy.pdf

refer:
- http://drops.wooyun.org/tips/8594
- http://plcscan.org/blog/
- https://github.com/evilcos/papers