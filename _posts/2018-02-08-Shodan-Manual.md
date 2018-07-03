---
title: Shodan手册
layout: post
tags:
  - sec
  - skills
  - shodan
  - information
  - search
category: 
  - hack
  - skills
comments: true
share: true
description: 本文主要对[Complete Guide to Shodan]的内容进行增、删、改，虽篇幅较长，但内容涉及了Shodan的搜索语法、客户端命令行、API编程、工具插件、工控等，较为详细，适合当作Shodan手册。代码和搜索示例均已亲自测试并附截图。
---

* TOC
{:toc}

 **_PS:本文投稿于[信安之路](http://www.myh0st.cn/)（http://mp.weixin.qq.com/s/SVl_YLhcfNgHya6jEnKx1g）的， 后@[ph0rse](http://ph0rse.me/)建议放于gitbook上，方便大家在线查阅、共同维护，就有了[gitbook版本](https://b404.gitbooks.io/shodan-manual/content/) （https://b404.gitbooks.io/shodan-manual/content/）

<!--more-->


## 前言

本文主要对[Complete Guide to Shodan](https://leanpub.com/shodan)（https://leanpub.com/shodan）的内容进行增、删、改，虽篇幅较长，但内容涉及了Shodan的搜索语法、客户端命令行、API编程、工具插件、工控等，较为详细，适合当作Shodan手册。代码和搜索示例均已亲自测试并附截图。

若内容有错误或者不恰当的地方，望大佬指出，谢谢。

## 介绍

[Shodan](https://www.shodan.io)是一个搜索互联网连接设备的搜索引擎，不同于谷歌、必应、百度这些搜索引擎。用户可以在Shodan上使用Shodan搜索语法查找连接到互联网的摄像头、路由器、服务器等设备信息。在渗透测试中，是个很不错的神器。

Shodan的搜索流程：

![Alt text](/img/shodan/1517795894244.png)


### 关于数据

#### Banner

Shodan采集的基本数据单位是banner。banner是描述设备所运行服务的标志性文本信息。对于Web服务器来说，其将返回标题或者是telnet登陆界面。Banner的内容因服务类型的不同而相异。


以下这是一个典型的HTTP Banner：

```javascript
HTTP/1.1 200 OK
Server: nginx/1.1.19
Date: Sat, 03 Oct 2015 06:09:24 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 6466
Connection: keep-alive
```

> 上面的banner显示该设备正在运行一个`1.1.19`版本的nginx Web服务器软件

下面是西门子S7工控系统协议的一个banner，这次返回的banner完全不同，提供了大量的详细数据(有关的固件信息、序列号):

```javascript
Copyright: Original Siemens Equipment
PLC name: S7_Turbine
Module type: CPU 313C
Unknown (129): Boot Loader           A
Module: 6ES7 313-5BG04-0AB0  v.0.3
Basic Firmware: v.3.3.8
Module name: CPU 313C
Serial number of module: S Q-D9U083642013
Plant identification: 
Basic Hardware: 6ES7 313-5BG04-0AB0  v.0.3
```

> 注：Shodan搜索的是联网设备运行服务的banner，不是单一的主机信息。如果单个IP暴露很多服务信息，在指定特定搜索内容的时候，搜索结果只会出现指定的内容，不会显示其他的服务信息。

#### 设备元数据

除了获取banner，Shodan还可以获取相关设备的元数据，例如地理位置、主机名、操作系统等信息。大部分元数据可以通过Shodan的官网获取，小部分的可以通过使用API编程获取。

#### IPv6

据统计，截至2015年10月，Shodan每月收集数百万条关于IPv6设备的可用数据。不过与收集的数亿个IPv4 banner相比，这些数字还是苍白。预计在未来几年会有所增长。


### 数据采集

#### 频率

Shodan的爬虫全天候工作，并实时更新数据库。在任何时候使用Shodan查询，都可以获得最新的结果。

#### 分布式

爬虫分布于世界各地，包括：

- 美国（东海岸和西海岸）
- 中国
- 冰岛
- 法国
- 台湾
- 越南
- 罗马尼亚
- 捷克共和国

从世界各地收集数据是为了防止地区各种因素的差异造成数据的偏差。例如，美国的许多系统管理员会封锁整个中国的IP范围，分布在世界各地的Shodan爬虫就可以确保任何全国性的封锁不会影响数据收集。

#### 随机

Shodan爬虫的基本算法是：

1. 生成一个随机的IPv4地址
2. 从Shodan能解析的端口列表中生成一个随机端口测试
3. 检测随机端口上的随机IPv4地址，并获取Banner
4. 继续步骤1

这意味着爬行器不扫描增量的网络范围。爬行是完全随机的，以确保在任何给定的时间内对因特网进行统一的覆盖，并防止数据的偏差。

### 深入SSL

SSL正在成为互联网上重要的服务，Shodan也随之扩展性地收集其banner，包括收集每个SSL的功能服务及其漏洞信息，比如HTTPS，不仅仅是SSL证书。

#### 漏洞测试

##### 心脏出血漏洞

如果某服务有心脏滴血漏洞，返回的banner将包含以下2个附加属性。
**opts.heartbleed**包含了对服务进行心脏出血漏洞测试的原始回应（在测试中，爬虫只抓取少量溢出来确认服务是否受到心脏出血的影响，不会获取泄露的私钥）。若设备容易受到攻击，爬虫会将`CVE-2014-0160`添加到**opts.vulns**列表中；若不易受到攻击，则会返回`!CVE-2014-0160`：

```javascript
{
    "opts": {
        "heartbleed": "... 174.142.92.126:8443 - VULNERABLE\n",
        "vulns": ["CVE-2014-0160"]
    }
}
```

Shodan也支持漏洞信息搜索。比如，要搜索美国受心脏滴血漏洞影响可在[shodan](https://www.shodan.io)输入`country:US vuln:CVE-2014-0160`

![Alt text](/img/shodan/1517293780813.png)

##### FREAK

如果服务支持导出密码，则爬虫程序就将“CVE-2015-0204”项添加到`opts.vulns`属性

```json
"opts": {
"vulns": ["CVE-2015-0204"]
}
```

##### Logjam

爬虫将短暂使用Diffie-Hellman密码连接到SSL服务，若连接成功就存储返回以下信息：

```json
"dhparams": {
    "prime": "bbbc2dcad84674907c43fcf580e9...",
    "public_key": "49858e1f32aefe4af39b28f51c...",
    "bits": 1024,
    "generator": 2,
    "fingerprint": "nginx/Hardcoded 1024-bit prime"
}
```

##### 版本

通常情况下，当浏览器连接到SSL服务时，它将协商应该与服务器一起使用的SSL版本和密码。然后，他们会就某个SSL版本（如TLSv1.2）达成一致，然后将其用于通信。

Shodan 爬虫开始按照上面所述的正常请求，与服务器进行协商连接SSL。但是，之后，他们还显式地尝试使用其他的SSL版本连接到服务器。换句话说，爬行器尝试使用SSLv2、SSLV3、TLSv1.0、TLSv1.1和TLSv1.2连接到服务器，来确定该SSL服务支持的所有版本。

收集到的信息将在`ssl.versions `版本字段中显示：

```json
{
    "ssl": {
        "versions": ["TLSv1", "SSLv3", "-SSLv2", "-TLSv1.1", "-TLSv1.2"]
    }
}
```

如果在版本前面有一个`-`符号，那么该设备不支持SSL版本。如果版本不以`-`开头，则服务支持给定的SSL版本。例如，上面的服务器支持:

```json
TLSv1
SSLv3
```

不支持版本：

```json
SSLv2
TLSv1.1
TLSv1.2
```

> TLS 1.0通常被标示为SSL 3.1，TLS 1.1为SSL 3.2，TLS 1.2为SSL 3.3。见[传输层安全性协议](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E6%80%A7%E5%8D%94%E5%AE%9A)

版本信息可以通过shodan的网站或者API搜索。例如，输入`ssl.version:sslv2`搜索查询将返回允许使用SSLv2的所有SSL服务(包括HTTP、SMTP、HTTP、POP3、IMAP等)

![Alt text](/img/shodan/1517296483629.png)

##### 遵循链

证书链是从root到最终用户的SSL 证书列表；SSL的banner服务包括一个`ssl.chain`属性，该属性包含PEM序列化证书中链的所有SSL证书。

### 基础之上

对于大多数服务，爬虫试图分析主要的banner文本，并解析出有用的信息。不过有些情况是，抓取MongoDB的集合名称，从远程桌面服务获取屏幕截图，存储比特币的对等点列表。有两种Shodan使用的高级数据分析技术，我想强调一下：

![Alt text](/img/shodan/1517298089142.png)


#### Web组件

爬虫也会尝试确定创建网站的Web技术。对于http和https模块，将分析header和HTML来分解网站的组件。结果信息存储在`http.components`属性中。该属性是一个技术字典，关键是技术的名称（例如jQuery），值是另一个具有类别属性的字典。categories属性是与技术相关的类别列表。例如

```json
"http": {
    ...
        "components": {
            "jQuery": {
                "categories": ["javascript-frameworks"]
            },
            "Drupal": {
                "categories": ["cms"]
            },
            "PHP": {
                "categories": ["programming-languages"]
            }
        },
             ...
    },
```

`http.components`表明该网站在运行`Drupal`内容管理系统，该系统本身使用的`jQuery`和`PHP`。
`Shodan REST API`使信息通过过滤器http.component和2个方面（http.componen和http.component_category）进行搜索。要获得所有可能的组件/类别值的完整列表，请使用新的方面。例如，要获得所有可能类别的完整列表，请使用以下shodan命令：

```bash
shodan stats --facets http.component_category:1000 http
```
![Alt text](/img/shodan/1517299854335.png)

#### 级联

如果一个banner返回了关于对等点的信息，或者有关于另一个运行服务的IP地址的信息，那么爬虫就会试图在这个IP/服务上执行一个banner抓取。例如: 主线DHT（过去用于Bittorrent）的默认端口是6881。这样一个DHT节点的banner看起来如下:

```bash
DHT Nodes
97.94.250.250	58431
150.77.37.22	34149
113.181.97.227	63579
252.246.184.180	36408
83.145.107.53	52158
77.232.167.126	52716
25.89.240.146	27179
147.23.120.228	50074
85.58.200.213	27422
180.214.174.82	36937
241.241.187.233	60339
166.219.60.135	3297
149.56.67.21	13735
107.55.196.179	8748
```

以前，抓取工具会抓取上面的banner，然后继续抓取。现通过为DHT banner的抓取启用级联功能，抓取工具现在可以为所有的对等启动新的banner抓取请求。在上面的示例中，搜寻器将使用DHT banner捕获器，在IP为`97.94.250.250`的58431端口上启动扫描`150.77.37.22`,继续在端口34149上启动扫描，依此类推。
也就是说，如果初始扫描数据包含有关其他潜在主机的信息，对IP的单次扫描可能会导致级联扫描。

为了跟踪初始扫描请求与任何子级/级联请求之间的关系，我们引入了2个新属性：

- **_shodan.id**：banner的唯一ID。如果可以从服务启动级联请求，这个属性就一定存在，但这并不一定意味着级联请求会成功。
- **_shodan.options.referrer**：提供触发创建当前banner的banner的唯一ID。即引用者是当前banner的父代。

## Web界面

访问Shodan收集的数据最简单的方法是通过Web界面。所有的人都可以输入一个需要查询的设备。

### 搜索查询说明

默认情况下，搜索查询仅查看主横幅文本，不搜索元数据。例如，如果您要搜索“Google”，那么搜索结果将只包含标题中显示“Google”文本的结果; 它不一定会返回Google网络范围的结果。

![Alt text](/img/shodan/1517301871310.png)

如上所述，搜索“谷歌”返回许多组织购买并连接到Internet的谷歌搜索设备;它不返回谷歌的服务器。

Shodan将尝试找到匹配所有搜索项的结果，这意味着隐含地在每个搜索项之间存在`+`或`AND`连接符号。例如，搜索查询“apache + 1.3”相当于“apache 1.3”

要搜索特定的元数据需要使用搜索过滤器。

### 过滤器

过滤器是Shodan用来根据服务或设备的元数据缩小搜索结果的特殊关键字。输入过滤器的格式是：

```javascript
filtername:value
```

> **注意 `:`和值之间没有空格**

要使用包含过滤器空间的值，必须将值包含在双引号中。例如搜索圣地亚哥的所有联网设备：

```javascript
city:"San Diego"
```

![Alt text](/img/shodan/1517303846916.png)

筛选多个值可以使用`,`分开。比如要查找23号端口和1023号端口上运行telnet 的设备:

```javascript
port:23,1023
```

![Alt text](/img/shodan/1517360821230.png)

如果过滤语法不允许使用逗号(如`port,hostname,net`)，那么可以使用多个值联合查询：

```javascript
port:80,8080  city:"beijing" hostname:"baidu.com"
```

![Alt text](/img/shodan/1517361411351.png)

当然也可以使用`-`排除结果。比如：

```javascript
port:23,1023  -city:"San Diego"
```

![Alt text](/img/shodan/1517361748328.png)

有时候也需要过滤一些主文本标题为空的搜索结果，这时候就需要使用`-hash:0`。比如：

```javascript
port:8080 -hash:0
```

![Alt text](/img/shodan/1517362504823.png)

Shodan上的每个banner都有一个哈希值，对于空的banner，hash值就为0。若尝试搜索简短的、静态的banner，使用哈希过滤语法就比较便捷。

shodan支持大量的过滤器，比较常用的是：

| 过滤器名     | 描述                 | 举例                                       |
| -------- | ------------------ | ---------------------------------------- |
| category | 现有的分类：ics, malware | `category:"malware"`                     |
| city     | 城市的名字              | `city:"San Diego"`                       |
| country  | 国家简写               | `country:"ES"` `country:"CN"`            |
| net      | CIDR格式的IP地址        | `net:190.30.40.0/24`                     |
| hostname | 主机名或域名             | `hostname:"google"`                      |
| port     | 端口号                | `port:21`                                |
| org      | 组织或公司              | `org:"google"`                           |
| isp      | ISP供应商             | `isp:"China Telecom"`                    |
| product  | 软件、平台              | `product:"Apache httpd"` `product:"openssh"` |
| os       | 操作系统               | `os:"Windows 7 or 8"`                    |
| version  | 软件版本               | `version:"2.6.1"`                        |
| geo      | 经纬度                | `geo:"46.9481,7.4474"`                   |

*可参见附录B*

### 搜索引擎

![Alt text](/img/shodan/1517365516965.png)

默认情况下，搜索查询将查看过去30天内收集的数据。

#### 下载数据

完成搜索后，顶部将出现一个名为“ 下载数据”的按钮。点击该按钮将为您提供以JSON，CSV(XML已经弃用)格式下载搜索结果的选项.

![Alt text](/img/shodan/1517365559748.png)

JSON格式的文件，其中每行包括全部的banner和所有附带的元数据收集信息。这是保存所有可用信息的首选格式。格式与Shodan命令行客户端兼容，这意味着可以从Shodan网站下载数据，然后使用终端进一步处理。

CSV格式的文件，返回包含IP、端口，banner，组织和主机名内容。它不包含由于CSV文件格式的限制收集的所有Shodan信息。如果只关心结果的基本信息，并希望快速将其加载到外部工具（如Excel）中，可以使用此方法。

下载数据会消耗积分，积分需要在网站上购买。积分与Shodan API没有任何关联，并且不会每月自动更新。1积分可用于下载10000个结果。

#### 生成报告

Shodan可让根据搜索查询生成报告。该报告包含图表，可以全面了解互联网上的分发结果。这个功能是免费的，任何人都可以使用。



#### 分享结果

搜索完成之后，可以自定义，分享内容给其他Shodan用户。

![Alt text](/img/shodan/1517365655467.png)

> 共享搜索查询是公开查看的

### 例子


**搜索美国 的服务器在8080端口上运行8.0版本IIS服务的服务器：**

```javascript
port:8080 product:"Microsoft-IIS" country:"US" version:"8.0"
```
![Alt text](/img/shodan/1517815120023.png)


**搜索可能受到DOUBLEPULSAR攻击的机器：**

```javascript
port:445 "SMB Version: 1" os:Windows !product:Samba
```

![Alt text](/img/shodan/1517815308110.png)

**搜索所有`nist.gov`域名里的apache服务:**

```javascript
apache hostname:.nist.gov
```

![Alt text](/img/shodan/1517823589050.png)


### 	图标搜索


#### 通过已知icon hash搜索目标

**0x01 获取谷歌地址** 

ping一下google，获得一个谷歌ip，为`172.217.27.174`：

![Alt text](/img/shodan/1530573983568.png)

**0x02 查看原数据** 

使用shodan搜索刚才获取的IP，并找到查看原始数据那一行：

![Alt text](/img/shodan/1530574292469.png)

**0x03 查找favicon hash**

打开原始数据，找到`data.1.http.favicon.hash`，其值为`708578229`:

![Alt text](/img/shodan/1530574515697.png)

**0x04 图标hash搜索** 

在Shodan网页搜索的过滤器处输入`http.favicon.hash:708578229`就可以搜索shodan所收录的谷歌的网站。

![Alt text](/img/shodan/1530574667307.png)

**0x05 favicon data** 

`data.1.http.favicon.data`的值是base64,可以自行验证

![Alt text](/img/shodan/1530575325593.png)

#### 通过未知icon hash搜索目标

**0x01 背景** 

在日一个站的时候只有一个登陆界面的时候，没有其他信息，只有一个icon的时候，这时候就可以通过shodan搜索相同模板的站来入手，找突破口。

如果目标已经被shodan收录，直接搜索对应的IP就能通过上述方法进行图标hash搜索。如果是目标未被收录，就可以通过现在这个方法来进行侧面突破。

**0x02 原理**

shodan对icon的处理其实是通过`MurmurHash`进行存储的。urmurHash 是一种非加密型哈希函数，适用于一般的哈希检索操作。由Austin Appleby在2008年发明，并出现了多个变种，[6] 都已经发布到了公有领域(public domain)。与其它流行的哈希函数相比，对于规律性较强的key，MurmurHash的随机分布特征表现更良好。其地址在github上https://github.com/hajimes/mmh3


![Alt text](/img/shodan/1530575887237.png)

**0x03 icon hash的生成验证**

直接在本地使用pip安装mmh3。



验证代码如下：

```python
import mmh3
import requests
response = requests.get('https://www.baidu.com/favicon.ico')
favicon = response.content.encode('base64')
hash = mmh3.hash(favicon)
print hash
```
![Alt text](/img/shodan/1530576114283.png)

运行验证代码获得icon hash

在shodan中进行`http.favicon.hash:-1507567067`搜索，验证成功：

![Alt text](/img/shodan/1530576267114.png)

**0x04 扩展**

例如`http://common.cnblogs.com/favicon.ico`得到博客园的icon：

![Alt text](/img/shodan/1530577989860.png)

直接将图标地址放入脚本中计算其hash值，得到`-395680774`：

![Alt text](/img/shodan/1530577955693.png)

再在shodan中搜索`http.favicon.hash:-395680774`：

![Alt text](/img/shodan/1530578093536.png)






### Shodan地图

https://maps.shodan.io/

直观的地图界面,显示搜索的结果

![Alt text](/img/shodan/1517366645440.png)

使用![Alt text](/img/shodan/1517366962856.png)可改变显示风格，有卫星、街景选项。


### Shodan Exploits

https://exploits.shodan.io

Shodan Exploits收集来自CVE，Exploit DB和Metasploit的漏洞和攻击，使其可通过Web界面进行搜索。

![Alt text](/img/shodan/1517367125715.png)


Shodan Exploit搜索的结果不同于Shodan，Exploit搜索的内容包括exploit的信息和元数据。

| 名称          | 描述            | 举例                                       |
| ----------- | ------------- | ---------------------------------------- |
| author      | 漏洞或exploit的作者 | `author:"Tom Ferris"`                    |
| description | 描述            | `description:"remote"`                   |
| platform    | 平台            | `platform:"php"` `platform:"windows"` `platform:"Linux"` |
| type        | 漏洞类型          | `type:"remote"` `type:"dos"`             |


### Shodan Images

https://images.shodan.io

顶部的搜索框使用与Shodan主搜索引擎相同的语法。使用搜索框按组织或网点筛选最有用。但是，它也可以用于过滤显示的图像类型。

![Alt text](/img/shodan/1517367993553.png)


图像数据主要来自：

- VNC
- Remote Desktop (RDP)
- RTSP
- Webcams
- X Windows

每个图像源来自不同的端口/服务，因此具有不同的banner。如果只想看到来自摄像头的图像，你可以搜索：

```javascript
HTTP
```

![Alt text](/img/shodan/1517368351206.png)

> 查找`VNC`可以输入`RFB`，查找`RTSP`可以使用`RTSP`

如果在[Shodan](https://www.shodan.io)中搜索截图，可以使用`has_screenshot:true`：

```javascript
has_screenshot:true country:"KR"
```

![Alt text](/img/shodan/1517368614134.png)

![Alt text](/img/shodan/1517368596925.png)

### 共享的Shodan搜索语法

链接见https://www.shodan.io/explore

![Alt text](/img/shodan/1517815542618.png)



## 关于Shodan的外部工具

### Shodan命令行界面

安装最新的Shodan可以通过

```bash
easy_install shodan
```

安装该工具之后，使用API对其进行初始化操作：

```bash
shodan init API_KEY
```

![Alt text](/img/shodan/1517369431079.png)

#### alert命令

alert命令提供创建、列表、清除和删除网络提醒的能力。

```bash
shodan alert
```

![Alt text](/img/shodan/1517369532577.png)

#### convert命令

convert命令是导出Shodan的搜索结果文件，比如JSON文件。

#### count命令

统计查询结果

```bash
shodan count microsoft iis 6.0
```

![Alt text](/img/shodan/1517378957224.png)

#### download命令

搜索Shodan并将结果导出到JSON文件中（请参阅附录A）。

默认情况下，只下载1000个结果.如果想导出更多的结果，加上`--limit`标志。

download命令可以保存搜索结果到本地，随时使用`parse`命令处理结果，当再次导出相同搜索内容的时候，就不必再花费积分导出。

```bash
shodan download  microsoft iis 6.0
```

![Alt text](/img/shodan/1517379702688.png)

#### host命令

查看有关主机的信息，例如它位于何处，哪些端口是开放的以及哪个组织拥有其IP。

```bash
shodan host 189.201.128.250
```

![Alt text](/img/shodan/1517379850752.png)

#### honeyscore命令

检查IP地址是否是蜜罐。

```bash
shodan honeyscore 41.231.95.212
```

![Alt text](/img/shodan/1517379975971.png)

#### info命令

查询自己的API账户信息，包括本月剩余的查询次数和扫描积分数量。

```bash
shodan info
```

![Alt text](/img/shodan/1517380040916.png)

#### myip命令

返回本机的出口IP（类似于`curl ip.cn`）

![Alt text](/img/shodan/1517380251699.png)

![Alt text](/img/shodan/1517380533585.png)

#### parse命令

使用`parse`来分析使用导出的搜索结果。它可以让过滤出感兴趣的字段，也可以将JSON转换为CSV。这样，对于导入到其他脚本比较友好。

以下命令输出之前下载的Microsoft-IIS数据，并且将JSON文件转换为的CSV格式的文件。其中包括IP地址，端口和归属组织：

```bash
shodan parse --fields ip_str,port,org --separator , microsoft-data.json.gz
```

![Alt text](/img/shodan/1517381072881.png)

#### scan命令

`scan`命令提供了一些子命令。最重要的一个是`submit`命令，这个命令可以使用shodan执行网络扫描。

```bash
shodan scan submit 198.20.99.0/24
```

> 默认情况下，它将显示IP，端口，主机名和数据。您可以使用`--fields`来打印感兴趣的任何banner字段。

![Alt text](/img/shodan/1517382144428.png)



#### search命令

`search`命令搜索时候，终端将以友好的方式显示结果。默认情况下，它将显示IP，端口，主机名和数据。不过可以使用`--fields 参数`来打印感兴趣的任何banner字段。

```bash
shodan search --fields ip_str，port，org，hostnames microsoft iis 6.0
```

![Alt text](/img/shodan/1517382536055.png)

#### stats命令

提供关于搜索的摘要信息，当然可以用于统计信息。
命令显示了Apache Web服务器所在的最常用的国家：

```bash
shodan stats --facets country apache
```

![Alt text](/img/shodan/1517382718966.png)

#### stream命令

`stream`命令提供对Shodan爬虫收集的实时数据流的访问。

![Alt text](/img/shodan/1517382980293.png)

该命令支持许多不同的选项，但有2个是重要的提及：

- `--datadir`:可以指定流数据存储的目录，生成的文件格式`YYYY-MM-DD.json.gz`。每天只要保持数据流运行，就会生成一个新的文件。
  - 例如：从实时数据流保存数据到指定路径，可以使用`shodan stream --datadir /var/lib/shodan/`
- `--limit`：指定应下载多少结果。默认情况下，该`stream`命令将一直运行，直到退出该工具。但是，如果只想收集数据样本，那么`--limit`参数可确保收集少量记录。
  - 例如：`shodan stream --limit 100`可以连接到Shodan实时流，打印出前100条记录，然后退出
- `--ports`:接受逗号分隔端口，收集指定端口记录的清单
  - 例如：`shodan stream --ports 80,8080`

### 例子

#### 官方例子

官方例子见  [28 public asciicasts by Shodan](https://asciinema.org/~Shodan)：https://asciinema.org/~Shodan

![Alt text](/img/shodan/1517883511076.png)



#### 网络分析例子

Shodan最常见的使用案例是使用它来更好地了解公共网络范围内的运行情况。Shodan命令行工具可以快速帮助、处理你的目标。例如，查看`78.13.0.0/16`段在网上暴露了多少服务。

```bash
shodan count net:78.13/16
```

![Alt text](/img/shodan/1517384046261.png)

`count`命令提供了关于`78.13.0.0/16`段暴露了4960个服务

统计出该网段暴露的最多端口数的top10：

```bash
shodan stats --facets port net:78.13/16
```

![Alt text](/img/shodan/1517384203836.png)

有时候也需要详尽的端口暴露数分配报告，因此指定最大的端口，返回此端口范围的暴露数统计情况：

```bash
shodan stats --facets port:100000 net:78.13/16
```

![Alt text](/img/shodan/1517384538705.png)

该网络上有955个端口被发现暴露，并统计出各自端口暴露数。最多的是`7547`端口，这个端口是调制解调器的端口，这个端口也有很多的[安全问题可以关注](https://pastebin.com/eZrrLGzv)。在统计中也有一些非常见的标准端口，这些端口大多是为了掩盖自己真实的Web服务。

统计该网段的80、443端口上的服务情况：

```bash
shodan stats --facets product "HTTP net:78.13/16 -port:80,443"
```

![Alt text](/img/shodan/1517385442321.png)

> 这里应将端口号放在引号中，防止bash认为该port是shodan的命令参数。

#### 分析SSL的例子

若要了解整个网络的SSL使用情况，可按照以下来查询

```bash
shodan stats --facets ssl.version HTTP net:78.13/16
```

![Alt text](/img/shodan/1517385675772.png)

可以看出大多运行的是TLS1.0及其以上的版本，不过还有一些古老的设备在使用sslv2。

在web端的shodan搜索出来：

![Alt text](/img/shodan/1517386029123.png)

#### 分析telnet情况

```bash
mkdir telnet-data

shodan stream --ports 23,1023,2323 --datadir telnet-data / --limit 10000
```

> 创建一个名为telnet-data的目录来存储telnet数据，然后从实时数据流中导出10000条记录，存储在telnet-data文件夹中

### Maltego扩展组件/assets
shodan扩展组件为Maltego提供了2/assets新的实体功能(service和exploit)和五个转换：
- searchShodan
- searchShodanByDomain
- searchShodanByNetblock
- toShodanHost
- searchExploits

![Alt text](/img/shodan/1517386986498.png)

### 浏览器插件

[Chrome](https://chrome.google.com/webstore/detail/shodan/jjalcfnidlmpjhdfepjhjbhnhkbgleap)和[Firefox](https://addons.mozilla.org/en-US/firefox/addon/shodan-firefox-addon/)都有插件可以使用

![Alt text](/img/shodan/1517387336781.png)

### Metasploit

MSF主要有两个shodan模块：


    1. Shodan的常用搜索：`auxiliary/gather/shodan_search `                           

```bash
use auxiliary/gather/shodan_search

set SHODAN_APIKEY ********************

set QUERY ****

run 
```

![Alt text](/img/shodan/1517566369191.png)

2. Shodan的蜜罐验证：`auxiliary/gather/shodan_honeyscore `:

```bash
use auxiliary/gather/shodan_honeyscore 

set SHODAN_APIKEY ********************

set TARGET your_target

run
```

![Alt text](/img/shodan/1517565794510.png)

### Recon-ng

先添加shodan API的key：

```bash
keys add shodan_api VZbVJy***n8S****hS1YM1****enTuy
```

显示可用的模块：

```bash
show modules
```

使用shodan搜索主机名：

```bash
use recon/domains-hosts/shodan_hostname
```

设置参数，并运行：

```bash
show options
set SOURCE google
set LIMIT 1
run
```

![Alt text](/img/shodan/1517883058677.png)


## Developer API

Shodan提供了一个开发者的API（https://developer.shdan.io/api）来编程，获取所需要的信息。可以通过网站完成的事情都可以通过代码完成。

该API分为两部分：REST API和Streaming API。REST API提供搜索Shodan的方法，查找主机，获取关于查询的信息的摘要。Streaming API提供Shodan当前收集的数据的原始实时返回。有几个不同的套餐可以获取，该API获取的数据不能被搜索或用其他方式进行交互，适用于需要获取大量数据的人。

> 只有购买[开发者API计划](https://developer.shodan.io/billing/signup)的人才能获得Streaming API。

### 使用限制

根据API的套餐不同，API会有不同的限制：

1. **搜索：**每月的搜索次数有不同的限制，且需要使用查询积分。如果直接查询不会消耗查询积分，若进行过滤器或者是超过一页的搜索就需要消耗查询积分。搜索`apache`不需要消耗查询积分，搜`apache country:"US"`将会消耗一个查询积分，就算查询第二页也只会消耗一个查询积分。
2. **扫描：**按需获得的API也会根据积分限制每月扫描主机的数量。对于每个主机的扫描都需要一个扫描积分才能扫描。
3. **网络提醒：**根据不同的API的使用次数可以使用提醒功能监视所查询的IP。只有付费客户才能使用此功能，且无法在账户中创建超过100条提醒。

### Facets

过滤器缩小搜索结果。而`Facets`提供关注的banner字段的聚合信息，会得到一个大的结果视图。例如，Shodan网站使用`Facets`来提供搜索的统计信息，类似于在左侧显示的:

![Alt text](/img/shodan/1517400554857.png)

`Facets`有很多可以选择的参数。比如`port:22`的`Facets`是`ssh.fingerprint`，输出的时候就能显示出网络上SSH的详细条目。

> **目前Facet只能在API和Shodan的命令行上使用**

### 入门

所有的例子使用python来演示，当然其他的语言也有[Shodan的库/客户端](https://developer.shodan.io/api/clients)

为python环境安装Shodan库：

```bash
easy_install shodan
```

如果已经安装shodan，可以使用:

```bash
easy_install -U shodan
```

### 初始化

首先要做的是初始化：

```python
import shodan
api = shodan.Shodan('YOUR API KEY')
```

> 在https://account.shodan.io获取API密钥

### 搜索

已经拥有API密钥之后,准备进行搜索：

```python
#将请求包放在一个try/except块中，捕获抛出的异常
try:
        # Search Shodan
        results = api.search('apache')
 
        # Show the results
        print 'Results found: %s' % results['total']
        for result in results['matches']:
                print 'IP: %s' % result['ip_str']
                print result['data']
                print ''
except shodan.APIError, e:
        print 'Error: %s' % e
```

首先调用api对象的Shodan.search()方法，该方法返回的结果放入字典之中。然后，打印出搜索结果数量，最后将返回的结果进行遍历循环，并打印其IP和banner。每一页的搜索结果多达100个。

![Alt text](/img/shodan/1517414519892.png)


当然查询的时候还有很多返回信息。下面是`Shodan.search`返回的部分信息：

```json
{
	'total': 23199543,
	'matches': [
		{
			'data': 'HTTP/1.1 301 Moved Permanently\r\nDate: Thu, 01 Feb 2018 02:43:53 GMT',
			'hostnames':['mypage.ponparemall.com'],
			'ip': 2685469827L,
			'ip_str': '160.17.4.131',
			'port': 80,443,
			'isp': Recruit Holdings Co.,Ltd.,
			'timestamp': '2018-02-01T02:47:34.036371'
		},
		...
	]
}
```

> 有关banner可能包含的属性的完整列表请参阅附录A

默认情况下，为了节省带宽使用量，banner中的一些大的字段会被截断（比如HTML的）。若想要检索所有的信息，只需使用`minify=False`禁用概要。例如，用以下代码可以将匿名VNC服务的查询结果全部返回：

```python
results = api.search('has_screenshot:true', minify=False)
```

任何错误都会引发异常，将API请求封装在`try/except`是一种良好的代码习惯，不过为了简单起见，例子暂时不用`try/except`。

--------------------------------------------

以上脚本仅输出第一页的搜索结果。要想获得第二页或更多的结果，可以使用`page`参数执行请求：

```python
results = api.search('apache',page=2)
```

如果想遍历所有的结果，使用`search_cursor()`方法会更加便捷 ：

```python
for banner in api.search_cursor('apache'):
    print(banner['ip_str']) # 打印出该IP的每个banner
```

> **该`search_cursor()`方法只能循环结果，返回banner,不能使用`Facets`**

### 主机查找

要用Shodan查找特定的IP，可用`Shodan.host()`函数：

```python
# 查询主机
host = api.host('217.140.75.46')

# 打印一般信息
print """
	IP: %s
	Organization: %s
	Operating System: %s
""" % (host['ip_str'], host.get('org', 'n/a'), host.get('os', 'n/a'))

# 打印所有的banner
for item in host['data']:
	print """
		Port: %s
		Banner: %s
		""" % (item['port'], item['data'])
```

默认情况下，Shodan只返回最近收集的主机的信息。如果想获取IP地址的完整历史记录，就使用`history`参数。

```python
host = api.host('217.140.75.46', history=True)
```

虽然以上代码可以返回所有banner，但其中也包括一些可能不再在主机上活动的服务。

### 扫描

Shodan每月至少爬取一次，但是如果想用Shodan立即扫描网络，就可以使用API​​扫描。（订购的不同API有不同的扫描次数）

与Nmap等工具扫描不同，使用Shodan进行的扫描是异步完成的，在进行Shodan扫描之后不会马上就收到扫描结果。这取决于开发人员是如何收集扫描结果的，是直接查找IP信息，是用Shodan直接搜索，还是订购实时信息流查找。Shodan命令行界面在启动扫描后，创建临时网络提醒，然后根据实时数据流出扫描结果。

```python
scan = api.scan('198.20.69.0/24')
```

也可以使用CIDR表示法的地址来提供扫描目标：

```python
scan = api.scan(['198.20.49.30', '198.20.74.0/24'])
```

在向API提交了扫描请求之后，会返回以下信息：

```json
{
	'id': 'R2XRT5HH6X67PFAB',
	'count': 1,
	'credits_left': 5119
}
```

- id：唯一的扫描标识符
- count: 提交的IP扫描数
- credits: 剩余的扫描积分

### 实时数据流

Streaming API是一种基于HTTP的服务，可返回Shodan收集的实时数据流。它不提供任何搜索或查找功能，它只是抓取工具收集的所有内容的概要。

例如，以下是一个脚本的部分代码，可以从容易受到`FREAK`（CVE-2015-0204）攻击的设备输出banner:

```python
def has_vuln(banner, vuln):
    if 'vulns' in banner['opts'] and vuln in banner['opts']['vulns']:
        return True
    return False

for banner in api.stream.banners():
    if has_vuln(banner, 'CVE-2015-0204'):
        print banner
```

> 在上面的例子中，`has_vuln()`方法检查服务是否可能存在CVE-2015-0204

banner中的许多属性是可选的，这样可以节省空间和带宽。为了使可选属性更容易处理，最好在函数中封装对属性的访问。

> **注意**：常规的API能访问的数量仅仅是订购的API数量的1%，只有订购了API具有许可证的客户才有100%的访问权限

### 网络提醒

要创建网络提醒，需要提供一个提醒名称和一个IP范围。名称应该具有概述性，当受到提醒的时候，就大概知道是什么监视内容被返回了。

```python
alert = api.create_alert('Production network', '198.20.69.0/24')
```

正如`scan()`方法一样，可以提供监视的网络范围列表：

```python
alert = api.create_alert('Production and Staging network', [
	'198.20.69.0/24',
	'198.20.70.0/24',
])
```

> 只能使用有限数量的监视IP，并且每个帐户不能有超过100个监视提醒在活动。

将网络提醒与扫描API结合使用时，一个有用的技巧是设置提醒的时间：

```python
alert = api.create_alert('Temporary alert', '198.20.69.0/24', expires=60)
```

上述代码可以使得提醒会激活使用60s,60s之后就会停止，不再进行监视。

成功创建提醒之后，API将返回以下对象：

```python
{
	"name": "Production network", 
	"created": "2015-10-17T08:13:58.924581", 
	"expires": 0, 
	"expiration": null, 
	"filters": {
		"ip": ["198.20.69.0/24"]
	},
	"id": "EPGWQG5GEELV4799",
	"size": 256
}
```

#### 订阅

一旦创建了提醒，就可以将其用作实时数据流的监视。

```python
for banner in api.stream.alert(alert['id']):
	print banner
```

与常规实时流一样，该`alert()`方法提供了一个迭代器，其中每个项目都是由Shodan爬虫收集的banner。`id`是`alert()`方法需要的唯一参数，在有网络提醒时会返回提醒ID。

#### 使用Shodan命令行界面

接下来讲述如何使用Shodan的命令行界面快速清除基于Python代码创建的提醒。

> **该`clear`命令将会清除电脑上创建的所有alert。**

清除所有alert：

```python
$ shodan alert clear
Removing Scan: 198.20.69.0/24 (ZFPSZCYUKVZLUT4F)
Alerts deleted
```

列出alert列表，确认有无alert：

```python
$ shodan alert list
You haven't created any alerts yet.
```

创建一个新的alert：

```python
$ shodan alert create "Temporary alert" 198.20.69.0/24
Successfully created network alert!
Alert ID: ODMD34NFPLJBRSTC
```

最后一步是订阅监视提醒，并存储返回的数据。要为创建的警报传输结果，将`ODMD34NFPLJBRSTC`的警报ID提供给`stream`命令：

```python
$ mkdir alert-data
$ shodan stream --alert=ODMD34NFPLJBRSTC --datadir=alert-data
```

上面的命令中，使用ID为`ODMD34NFPLJBRSTC`的alert提醒，并且使用数据流传输结果。结果将存储在名为`alert-data`的目录中。每天都会在`alert-data`目录中生成具有一个当天收集banner的新文件。也就是说，不用关心轮转文件，`stream`命令会处理这些。几天后，目录将如下所示：

```python
$ ls alert-data
2016-06-05.json.gz
2016-06-06.json.gz
2016-06-07.json.gz
```

### 例子


#### 常用 Shodan 库函数

- `shodan.Shodan(key) `：初始化连接API
- `Shodan.count(query, facets=None)`：返回查询结果数量
- `Shodan.host(ip, history=False)`：返回一个IP的详细信息
- `Shodan.ports()`：返回Shodan可查询的端口号
- `Shodan.protocols()`：返回Shodan可查询的协议
- `Shodan.services()`：返回Shodan可查询的服务
- `Shodan.queries(page=1, sort='timestamp', order='desc')`：查询其他用户分享的查询规则
- `Shodan.scan(ips, force=False)`：使用Shodan进行扫描，ips可以为字符或字典类型
- `Shodan.search(query, page=1, limit=None, offset=None, ufacets=None, minify=True)`：查询Shodan数据

#### 例子——基本的搜索

```python
#!/usr/bin/env python
#
# shodan_ips.py
# Search SHODAN and print a list of IPs matching the query
#
# Author: achillean

import shodan
import sys

# Configuration
API_KEY = "VZbVJyrIrn8SCSxHhS1YM1XPeKt5nTuy"

# Input validation
if len(sys.argv) == 1:
        print 'Usage: %s <search query>' % sys.argv[0]
        sys.exit(1)

try:
        # Setup the api
        api = shodan.Shodan(API_KEY)

        # Perform the search
        query = ' '.join(sys.argv[1:])
        result = api.search(query)

        # Loop through the matches and print each IP
        for service in result['matches']:
                print service['ip_str']
except Exception as e:
        print 'Error: %s' % e
        sys.exit(1)
```

#### 例子——使用Facets收集概要信息

Shodan API的强大功能其中之一是获取各种属性的概要信息。若想了解哪些国家的Apache服务器最多，那么可以使用`Facets`；若想知道哪个版本的nginx最受欢迎，可以使用`Facets`;若想查看Microsoft-IIS服务器的正常运行时间，那么可以使用`Facets`。

以下脚本显示了如何使用`shodan.Shodan.count()`方法在使用Shodan API进行搜索时候不返回任何详细结果，只返回组织、域、端口、ASN和国家的信息：

```python
#!/usr/bin/env python
#
# query-summary.py
# Search Shodan and print summary information for the query.
#
# Author: achillean

import shodan
import sys

# Configuration
API_KEY = 'YOUR API KEY'

# 键入想获取的信息
FACETS = [
    'org',
    'domain',
    'port',
    'asn',

    # 只关注前三城市，('country', 3)让Shodan返回前三城市
    # Facet默认五个搜索区域(比如搜索1000个国家，就键入('country', 1000)
]

FACET_TITLES = {
    'org': 'Top 5 Organizations',
    'domain': 'Top 5 Domains',
    'port': 'Top 5 Ports',
    'asn': 'Top 5 Autonomous Systems',
    'country': 'Top 3 Countries',
}

# Input validation
if len(sys.argv) == 1:
    print 'Usage: %s <search query>' % sys.argv[0]
    sys.exit(1)

try:
    # 键入Shodan API
    api = shodan.Shodan(API_KEY)

    # Generate a query string out of the command-line arguments
    query = ' '.join(sys.argv[1:])

    # 使用count()方法，因为它不返回结果，不需要通过付费API才能使用
    # count()运行速度快于search().
    result = api.count(query, facets=FACETS)

    print 'Shodan Summary Information'
    print 'Query: %s' % query
    print 'Total Results: %s\n' % result['total']

    # 从Facets打印摘要信息。
    for facet in result['facets']:
        print FACET_TITLES[facet]

        for term in result['facets'][facet]:
            print '%s: %s' % (term['value'], term['count'])

        # 在摘要信息之间打印一条空行
        print ''

except Exception, e:
    print 'Error: %s' % e
    sys.exit(1)
```


![Alt text](/img/shodan/1517540983847.png)



#### 例子——利用API编写GIF图片

Shodan很好得保存了爬取的IP的完整历史记录，可以通过API编写Python脚本，输出Shodan爬虫收集的屏幕截图的。

**环境：**

需要的Python包:

- shodan：`sudo easy_install shodan`安装
- arrow: `sudo easy_install arrow `安装。`arrow`包用于将banner的时间戳字段解析为Python datetime对象。

需要的软件包：

```python
sudo apt-get install imagemagick //将多个图像合并成一个GIF动画所需要

easy_install -U requests simplejson
```

代码参见：[Timelapse GIF Creator using the Shodan API](https://gist.github.com/achillean/7190d53841865a8c9978f59863669857)

```python
import arrow
import os
import shodan
import shodan.helpers as helpers
import sys


# Settings
API_KEY = ''

# The user has to provide at least 1 Shodan data file
if len(sys.argv) < 2:
	print('Usage: {} <shodan-data.json.gz> ...'.format(sys.argv[0]))
	sys.exit(1)

# GIFs are stored in the local "data" directory
os.mkdir('data')

# Setup the Shodan API object
api = shodan.Shodan(API_KEY)

# Loop over all of the Shodan data files the user provided
for banner in helpers.iterate_files(sys.argv[1:]):
	# See whether the current banner has a screenshot, if it does then lets lookup
	# more information about this IP
	has_screenshot = helpers.get_screenshot(banner)
	if has_screenshot:
		ip = helpers.get_ip(banner)
		print('Looking up {}'.format(ip))
		host = api.host(ip, history=True)
		
		# Store all the historical screenshots for this IP
		screenshots = []
		for tmp_banner in host['data']:
			# Try to extract the image from the banner data
			screenshot = helpers.get_screenshot(tmp_banner)
			if screenshot:
				# Sort the images by the time they were collected so the GIF will loop
				# based on the local time regardless of which day the banner was taken.
				timestamp = arrow.get(banner['timestamp']).time()
				sort_key = timestamp.hour

				# Add the screenshot to the list of screenshots which we'll use to create the timelapse
				screenshots.append((
					sort_key,
					screenshot['data']
				))

		# Extract the screenshots and turn them into a GIF if we've got more than a few images
		if len(screenshots) >= 3:
			# screenshots is a list where each item is a tuple of:
			# (sort key, screenshot in base64 encoding)
			# 
			# Lets sort that list based on the sort key and then use Python's enumerate
			# to generate sequential numbers for the temporary image filenames
			for (i, screenshot) in enumerate(sorted(screenshots, key=lambda x: x[0], reverse=True)):
				# Create a temporary image file
				# TODO: don't assume that all images are "jpg", use the mimetype instead
				open('/tmp/gif-image-{}.jpg'.format(i), 'w').write(screenshot[1].decode('base64'))
			
			# Create the actual GIF using the  ImageMagick "convert" command
			# The resulting GIFs are stored in the local data/ directory
			os.system('convert -layers OptimizePlus -delay 5x10 /tmp/gif-image-*.jpg -loop 0 +dither -colors 256 -depth 8 data/{}.gif'.format(ip))

			# Clean up the temporary files
			os.system('rm -f /tmp/gif-image-*.jpg')

			# Show a progress indicator
			print('GIF created for {}'.format(ip))
```

> 使用`iterate_files()`遍历Shodan数据文件。
> 在`shodan.Shodan.host()`方法中获取所有Shodan收集的IP的banner历史记录。

下载屏幕截图的JSON数据包：

```python
shodan download --limit -1 screenshots.json.gz has_screenshot:true
```



![Alt text](/img/shodan/1517555243075.png)

#### *例子——公开的MongoDB数据

[MongoDB](https://www.mongodb.com/)是一个流行的[NoSQL](https://en.wikipedia.org/wiki/NoSQL)数据库。在很长一段时间内，MongoDB默认是不需要输入用户名和密码,就可以登录。导致了许多MongoDB的服务在互联网上可以被公开访问。（见[shodan扫描结果：3万个MongoDB可以无密码访问 大概包含595Tb数据](http://bobao.360.cn/news/detail/1792.html)）

使用Shodan抓取这些数据库的banner，其中包含了大量关于存储数据的信息。以下是banner的概要：

```json
IP: 115.231.180.54
MongoDB Server Information
Authentication partially enabled
{
    "storageEngines": [
        "devnull", 
        "ephemeralForTest", 
        "mmapv1", 
        "wiredTiger"
    ], 
    "maxBsonObjectSize": 16777216, 
    "ok": 1.0, 
    "bits": 64, 
    "modules": [], 
    "openssl": {
        "compiled": "OpenSSL 1.0.1f 6 Jan 2014", 
        "running": "OpenSSL 1.0.1f 6 Jan 2014"
    }, 
    "javascriptEngine": "mozjs", 
    "version": "3.2.18", 
    "gitVersion": "4c1bae566c0c00f996a2feb16febf84936ecaf6f", 
    "versionArray": [
        3, 
        2, 
        18, 
        0
    ], 
    "debug": false, 
    "buildEnvironment": {
        "cxxflags": "-Wnon-virtual-dtor -Woverloaded-virtual -Wno-maybe-uninitialized -std=c++11", 
        "cc": "/opt/mongodbtoolchain/bin/gcc: gcc (GCC) 4.8.2", 
        "linkflags": "-fPIC -pthread -Wl,-z,now -rdynamic -fuse-ld=gold -Wl,-z,noexecstack -Wl,--warn-execstack", 
        "distarch": "x86_64", 
        "cxx": "/opt/mongodbtoolchain/bin/g++: g++ (GCC) 4.8.2", 
        "ccflags": "-fno-omit-frame-pointer -fPIC -fno-strict-aliasing -ggdb -pthread -Wall -Wsign-compare -Wno-unknown-pragmas -Winvalid-pch -Werror -O2 -Wno-unused-local-typedefs -Wno-unused-function -Wno-deprecated-declarations -Wno-unused-but-set-variable -Wno-missing-braces -fno-builtin-memcmp", 
        "target_arch": "x86_64", 
        "distmod": "ubuntu1404", 
        "target_os": "linux"
    }, 
    "sysInfo": "deprecated", 
    "allocator": "tcmalloc"
},
....
```

![Alt text](/img/shodan/1517472477983.png)

基本上，banner是MongoDB服务器信息和及其3个JSON对象用逗号分隔而组成。或者有的banner里有需要登陆凭据的验证字符——“authentication enabled”。每个JSON对象都包含有关数据库的不同信息，建议在Shodan网站上查看完整的头信息，方法是搜索：

```javascript
product:MongoDB metrics
```

![Alt text](/img/shodan/1517474301541.png)

> **`metrics`字符保证了搜索时只搜索不需要认证的MongoDB服务**

--------------------------------------

接下来使用banner信息统计最流行的数据库名称，以及在互联网上暴露的数据量。基本的工作流程：
1. 下载所有的MongoDB banner数据
2. 处理下载的文件，并输出前十个数据库名称的列表以及总数据大小

```bash
shodan download --limit -1 mongodb-servers.json.gz product:mongodb
```

上述命令功能是，利用Shodan搜索关于mongodb的设备信息，并使用`--limit -1`指令下载所有的查询结果，输出到`mongodb-servers.json.gz`文件。

-----------------------------------------------------

使用Python脚本处理刚才所下载的数据。

 要轻松遍历文件，将使用`shodan.helpers.iterate_files()`方法:

```python
import shodan.helpers as helpers
import sys

# datafile变量是命令行的第一个参数
datafile = sys.argv[1]

for banner in helpers.iterate_files(datafile):
    # 获得banner
```

由于每个banner仅仅是带有头信息的JSON，因此可以使用`simplejson`库将banner处理为本地的Python字典：

```python
# 去除Shodan加入的MongoDB的头信息
data = banner['data'].replace('MongoDB Server Information\n', '').split('\n},\n'\
)[2]
# 加载数据库信息
data = simplejson.loads(data + '}')
```

接下来的事情就是记录暴露的数据总量和最流行的数据库名称：

```python
total_data = 0
databases = collections.defaultdict(int)

...

    # 然后循环
    # 跟踪可访问的数据
    total_data += data['totalSize']

    # 跟踪哪些数据库使用最多
    for db in data['databases']:
        databases[db['name']] += 1
```

Python有一个有用的`collections.defaultdict`类，该类的功能是，如果字典的键尚不存在，这个类将自动为字典的键创建一个默认值。只需访问 MongoDB  banner的`totalSize`和`databases`属性即可收集的信息。
最后，我们只需要输出实际结果：

```python
print('Total: {}'.format(humanize_bytes(total_data)))

counter = 1
for name, count in sorted(databases.iteritems(), key=operator.itemgetter(1),reverse=True[:10]:
    print('#{}\t{}: {}'.format(counter, name, count))
    counter += 1
```

首先打印数据总量，然后使用`humanize_bytes()`方法将字节存储转换为易读的MB、GB等存储格式。

其次，对数据库集合进行多次逆序遍历排序（`key=operator.itemgetter(1)`），取结果的TOP10（`[:10]`）。

以下是读取Shodan数据文件并分析banner的完整脚本：

```python
#!C:\Python27\python.exe
# -*- coding: utf-8 -*-
# @Time    : 2018/2/2 10:01
# @Author  : b404
# @File    : MongDB_Shodan.py
# @Software: PyCharm

import collections
import operator
import shodan.helpers as helpers
import sys
import simplejson

def humanize_bytes(bytes, precision=1):
    """将字节转换为易读的存储单位MB、GB等

    Assumes `from __future__ import division`.

    >>> humanize_bytes(1)
    '1 byte'
    >>> humanize_bytes(1024)
    '1.0 kB'
    >>> humanize_bytes(1024*123)
    '123.0 kB'
    >>> humanize_bytes(1024*12342)
    '12.1 MB'
    >>> humanize_bytes(1024*12342,2)
    '12.05 MB'
    >>> humanize_bytes(1024*1234,2)
    '1.21 MB'
    >>> humanize_bytes(1024*1234*1111,2)
    '1.31 GB'
     >>> humanize_bytes(1024*1234*1111,1)
        '1.3 GB'
        """
    abbrevs = (
        (1 << 50L, 'PB'),
        (1 << 40L, 'TB'),
        (1 << 30L, 'GB'),
        (1 << 20L, 'MB'),
        (1 << 10L, 'kB'),
        (1, 'bytes')
    )
    if bytes == 1:
        return '1 byte'
    for factor, suffix in abbrevs:
        if bytes >= factor:
            break
    return '%.*f %s' % (precision, bytes / factor, suffix)

total_data = 0
databases = collections.defaultdict(int)
for banner in helpers.iterate_files(sys.argv[1]):
    try:
        # 去除Shodan加入的MongoDB的头信息
        data = banner['data'].replace('MongoDB Server Information\n', '').split('\n},\n')[2]

        # 加载数据库信息
        data = simplejson.loads(data + '}')
        # 记录公开访问的数据量
        total_data += data['totalSize']

        # 跟踪哪些数据库名称最常见
        for db in data['databases']:
            databases[db['name']] += 1
    except Exception, e:
        pass

print('Total: {}'.format(humanize_bytes(total_data)))

counter = 1
for name, count in sorted(databases.iteritems(), key=operator.itemgetter(1), reverse=True)[:10]:
    print('#{}\t{}: {}'.format(counter, name, count))
counter += 1
```

该脚本输出：

```bash
Total: 569.7 GB
#1	Warning: 1448
#1	local: 1172
#1	admin: 488
#1	README: 113
#1	config: 67
#1	test: 43
#1	DATA_HAS_BEEN_BACKED_UP: 19
#1	logs: 19
#1	db_has_been_backed_up: 15
#1	gpsreal: 11
```

![Alt text](/img/shodan/1517540128263.png)


## 工业控制系统

工控指的是工业自动化控制，主要利用电子电气、机械、软件组合实现。打比方说，工业控制系统(ICS)是控制周围世界的计算机。它们负责管理办公室的空调、发电厂的涡轮机、剧院的照明设备或者工厂的机器人。

*可以去[灯塔实验室](http://plcscan.org/blog/)（http://plcscan.org/blog/）和https://cs3sthlm.se/ 了解工控的相关姿势*


[SHINE项目](https://www.slideshare.net/BobRadvanovsky/project-shine-findings-report-dated-1oct2014)（SHodan INTElligence Extraction）研究表明，从2012至2014年，互联网上至少有200万个可公开访问的工控设备。在2012年，第一个包含500,000个ICS设备的数据库被发送到ICS- cert。ICS-CERT认定，在美国50万的设备中，大约有7200个是关键的基础设施。随着对工控安全做出很多努力，越来越多的工控设备进行漏洞修补或者关机下线。但这是一个具有挑战性的问题，而且针对工控安全问题的解决方案没有一个是简单的。

下图是2014年Shodan爬取的工控全球分布图：

![Alt text](/img/shodan/1517563420610.png)


### 常用缩写

以下是一些常用的缩写，有助于后面了解协议，快速找到ICS设备：


|  缩写   |                    说明                    |
| :---: | :--------------------------------------: |
| SCADA |               数据采集与监视控制系统                |
|  ICS  |                  工业控制系统                  |
|  DCS  |              分布式控制系统/集散控制系统              |
|  PCS  |                  过程控制系统                  |
|  ESD  |                  应急停车系统                  |
|  PLC  |  可编程序控制器(Programmable Logic Controller)  |
|  RTU  |                 远程终端控制系统                 |
|  IED  |                  智能监测单元                  |
|  HMI  |      人机界面(Human Machine Interface)       |
|  MIS  |  管理信息系统(Management Information System)   |
|  SIS  | 生产过程自动化监控和管理系统（Supervisory Information System） |
|  MES  |                 制造执行管理系统                 |
|  BMS  |    建筑管理系统(uilding Management System)     |
|  VNC  |    虚拟网络计算(Virtual Network Computing)     |

### 协议

识别联网的控制系统有两种不同的方式：

**1. 在ICS环境中使用的非ICS协议。**


Shodan的ICS搜索结果大部分是通过搜索WEB服务器或其他常用的协议发现的，这些协议并不是直接与ICS相连接，而是可以在ICS网络上看到。例如，运行在HMI上的Web服务器或未经身份验证的Windows远程桌面直接连接在ICS网络上，这些协议将回提供一个ICS的可视化界面。（但大多常用某种形式进行身份验证）

下图就是一个在[Shodan Images]()上找到的一个通过未经认证的VNC连接并暴露的HMI引擎。

![Alt text](/img/shodan/1517558673290.png)

**2. ICS协议**

ICS协议是控制系统使用的原始协议。每个ICS协议都有其独特的标志，它们都有一个共同点：它们不需要任何认证。这意味着，如果可以远程访问工业设备，则可以自动对其进行读取和写入。但是，原始的ICS协议往往很难开发，是独有的，实际也很难与控制系统交互。也就是使用Shodan很容易检查设备是否支持ICS协议。

下图的banner是一个`Siemens S7 PLC`设备的Shodan搜索图，banner暴露了其序列号和位置：

![Alt text](/img/shodan/1517559696791.png)



#### Modbus

MODBUS协议定义了一个与基础通信层无关的简单协议数据单元（PDU）。特定总线或网络上的MODBUS协议映射能够在应用数据单元（ADU）上引入一些附加域。

![Alt text](/img/shodan/1517562446923.png)



工控安全问题：

- 缺乏认证：仅需要使用一个合法的Modbus地址和合法的功能码即可以建立一个Modbus会话
- 缺乏授权：没有基于角色的访问控制机制， 任意用户可以执行任意的功能。
- 缺乏加密：地址和命令明文传输， 可以很容易地捕获和解析

#### PROFIBUS

PROFIBUS 一种用于工厂自动化车间级监控和现场设备层数据通信与控制的现场总线技术，可实现现场设备层到车间级监控的分散式数字控制和现场通信网络

#### DNP3 

DNP3 DNP(Distributed Network Protocol,分布式网络协议)是一种应用于自动化组件之间的通讯协议，常见于电力、水处理等行业。简化OSI模型，只包含了物理层，数据层与应用层的体系结构（EPA）。SCADA可以使用DNP协议与主站、RTU、及IED进行通讯。

#### ICCP

ICCP 电力控制中心通讯协议。

#### OPC

OPC 过程控制的OLE （OLE for Process Control）。OPC包括一整套接口、属性和方法的标准集，用于过程控制和制造业自动化系统。

#### BACnet

BACnet 楼宇自动控制网络数据通讯协议（A Data Communication Protocol for Building Automation and Control Networks）。BACnet 协议是为计算机控制采暖、制冷、空调HVAC系统和其他建筑物设备系统定义服务和协议

#### CIP

CIP 通用工业协议，被deviceNet、ControINet、EtherNet/IP三种网络所采用。

#### Siemens S7

Siemens S7 属于第7层的协议，用于西门子设备之间进行交换数据，通过TSAP，可加载MPI,DP,以太网等不同物理结构总线或网络上，PLC一般可以通过封装好的通讯功能块实现。

#### 其他工控协议

其他工控协议IEC 60870-5-104、EtherNet/IP、Tridium Niagara Fox、Crimson V3、OMRON FINS、PCWorx、ProConOs、MELSEC-Q


#### 架构缺陷

- 未经真正安全考验的组件与协议
  - 若有Web服务，那就利用Web服务该有的缺陷
  - 经典渗透技巧大多适用于工控网络
- 稳定性优先+懒 → 升级难
- 互联网的便利性，让工控组件暴露在网络空间里
- 以协议研究为出发点

#### 工控系统的指纹识别技术

http://plcscan.org/blog/2017/03/fingerprint-identification-technology-of-industrial-control-system/

### 信息探测

#### Ethernet/IP

Port:44818

![Alt text](/img/shodan/1517562632678.png)

![Alt text](/img/shodan/1517562645838.png)


- Nmap: enip-enumerate.nse
- Python: https://github.com/paperwork/pyenip
- Wireshark dissector
   - src/epan/dissector/packet-etherip.c

#### Modbus

- port:502
- 抓取设备相关信息
 - Nmap: modicon-info.nse
 - Wireshark dissector
     - src/epan/dissector/packet-mbtcp.c
- 认证与加密缺失

![Alt text](/img/shodan/1517562668708.png)


#### IEC 61870-5-101/104

- Port: 2404
- 判断是否使用该协议
  - Nmap: iec-identify.nse
  - Wireshark dissector
     - src/epan/dissectors/packet-iec104.c
- 认证与加密缺失


![Alt text](/img/shodan/1517562684375.png)


#### Siemens S7

- key:Siemens S7 PLC（102）
- 抓取设备相关信息
 - Nmap: s7-enumerate_.nse
     - http://plcscan.org/blog/wp-content/uploads/2014/11/s7-enumerate.nse_.txt
 - Wireshark plugins
     - http://sourceforge.net/projects/s7commwireshark/
- 弱加密，易破解

![Alt text](/img/shodan/1517562711782.png)


#### Tridium Niagara Fox

- port:1911
- 抓取设备信息
  - Nmap: fox-info.nse 

![Alt text](/img/shodan/1517562772495.png)


## 附录A：Banner格式

以下是有关banner的最新属性字段列表（若要查看最新的，请访问[在线文档](https://developer.shodan.io/api/banner-specification)）：

### 常用属性

|    名称     |             描述             |                举例                 |
| :-------: | :------------------------: | :-------------------------------: |
|    asn    |           自治系统号            |              AS4837               |
|   data    |        服务的主要banner         |           HTTP/1.1 200…           |
|    ip     |         整数型的IP地址格式         |             493427495             |
|  ip_str   |           字符串型IP           |           199.30.15.20            |
|   ipv6    |          字符串型IPv6          |       2001:4860:4860::8888        |
|   port    |           服务的端口号           |                80                 |
| timestamp |          收集信息的日期           |    2014-01-15T05:49:56.283713     |
|   hash    |          数据属性的散列值          | Numeric hash of the data property |
| hostnames |          目标IP的主机名          |  [“shodan.io”, “www.shodan.io”]   |
|  domains  |         目标IP的所有域名          |           [“shodan.io”]           |
|   link    |           网络连接类型           |             以太网/调制解调器             |
| location  |          设备的物理地址           |                见下文                |
|   opts    |   补充或者实验数据不包含在主要的banner中   |                                   |
|    org    |          分配IP的组织           |            Google Inc.            |
|    isp    |         负责IP空间的ISP         |         Verizon Wireless          |
|    os     |            操作系统            |               Linux               |
|  uptime   |           IP上线时间           |                50                 |
|   tags    |   描述设备用途的标签列表(仅供企业型账号使用)   |          ["ics", "vpn"]           |
| transport | 用于收集banner的传输协议(UDP或者是TCP) |                tcp                |

### Elastic属性

下列属性是为 Elastic (曾经的 ElasticSearch)收集的:


|       名称        |       描述        |
| :-------------: | :-------------: |
| elastic.cluster |    有关集群的一般信息    |
| elastic.indices |   集群上可用的索引列表    |
|  elastic.nodes  | 群集的节点/对等点列表及其信息 |


### HTTP(S)属性

Shodan遵循HTTP响应的重定向，并将所有中间数据存储在banner中。抓取工具不遵循重定向的情况是HTTP请求被重定向到HTTPS位置，反之亦然：


|       名称        |               描述               |
| :-------------: | :----------------------------: |
| http.components |          用于创建网站的网络技术           |
|    http.host    |        发送主机名来抓取网站的HTML         |
|    http.html    |           网站的HTML内容            |
| http.html_hash  |        http.html属性的数字散列        |
|  http.location  |          最终的HTML响应的位置          |
| http.redirects  | 遵循的重定向列表。每个重定向项目有3个属性：主机，数据和位置 |
|   http.robots   |        robots.txt文件的网站         |
|   http.server   |          HTTP响应的服务器头           |
|  http.sitemap   |         网站的Sitemap XML         |
|   http.title    |             网站的标题              |

### 位置属性


| 名称            | 描述           |
| ------------- | ------------ |
| area_code     | 设备位置的区号      |
| city          | 城市名称         |
| country_code  | 2个字母组成的国家代码  |
| country_code3 | 3个字母组成的国家代码  |
| country_name  | 国家的全名        |
| dma_code      | 指定市场区号（仅限美国） |
| latitude      | 纬度           |
| longitude     | 经度           |
| postal_code   | 邮政编码         |
| region_code   | 地区代码         |

### SMB属性


|        名称        |                描述                 |
| :--------------: | :-------------------------------: |
|  smb.anonymous   |      服务是否允许匿名连接（true/false)       |
| smb.capabilities |             服务支持的功能列表             |
|    smb.shares    |             可用的网络共享列表             |
| smb.smb_version  |            用于收集信息的协议版本            |
|   smb.software   |              提供服务的软件              |
|     smb.raw      | 服务器发送的十六进制编码数据包列表; 如果想做SMB解析，这很有用 |

### SSH属性

|       名称        |       描述       |
| :-------------: | :------------: |
|   ssh.cipher    |   协商期间使用的密码    |
| ssh.fingerprint |     设备的指纹      |
|     ssh.kex     | 服务器支持的密钥交换算法列表 |
|     ssh.key     |   服务器的SSH密钥    |
|     ssh.mac     |    消息认证码算法     |

### SSL 属性

如果服务使用SSL进行包装，则Shodan将执行附加测试，并在以下属性中提供结果：

|         名称         |                    描述                    |
| :----------------: | :--------------------------------------: |
| ssl.acceptable_cas |              服务器接受的证书颁发机构列表              |
|      ssl.cert      |                可解析的SSL证书                 |
|     ssl.cipher     |                SSL连接的首选密码                |
|     ssl.chain      |            从用户证书到根证书的SSL证书列表             |
|    ssl.dhparams    |             Diffie-Hellman参数             |
|     ssl.tlsext     |              服务器支持的TLS扩展列表               |
|    ssl.versions    | 支持的SSL版本; 如果该值用`-`开头，则该服务不支持该版本（例如：“-SSLv2”是指不支持SSLv2的服务） |

### ISAKMP属性

以下为使用ISAKMP协议的VPN（例如IKE）收集的属性：


|             名称              |         描述         |
| :-------------------------: | :----------------: |
|    isakmp.initiator_spi     | 初始化器的hex编码的安全参数索引  |
|    isakmp.responder_spi     | 用于响应器的hex编码的安全参数索引 |
|     isakmp.next_payload     |    启动后发送的下一个载荷     |
|       isakmp.version        |   协议版本; 例如“1.0”    |
|    isakmp.exchange_type     |        交换类型        |
|   isakmp.flags.encryption   |  加密设置：true或false   |
|     isakmp.flags.commit     |  提交设置：true或false   |
| isakmp.flags.authentication |  认证设置：true或false   |
|        isakmp.msg_id        |    消息的十六进制编码标识     |
|        isakmp.length        |    ISAKMP数据包的大小    |


### 特殊属性

#### _shodan

`_shodan`属性包含有关数据如何被Shodan收集的信息。它与其他属性不同，因为它不提供有关设备的信息。相反，它会告诉你Shodan使用哪一个banner抓取器来与目标IP交互。这对于探知服务器上的端口运行服务情况是很重要的。例如，80端口是最知名的Web服务端口，但它也被各种恶意软件用来规避防火墙规则，`_shodan`属性将让你知道是该端口否用HTTP模块来收集数据或是否使用了反恶意软件模块。

|            名称            |            描述            |
| :----------------------: | :----------------------: |
|     _shodan.crawler      |    识别Shodan爬取工具的唯一ID     |
|        _shodan.id        |       此banner的唯一ID       |
|      _shodan.module      | 爬虫抓取banner而使用SHodan模块的名称 |
|     _shodan.options      |      数据收集期间使用的配置选项       |
|     _shodan.hostname     |      发送Web请求时使用的主机名      |
| _shodan.options.referrer | 为某端口/服务触发扫描的banner的惟一ID  |

### 例子

```json
{
   "timestamp": "2014-01-16T08:37:40.081917","timestamp": "2014-01-16T08:37:40.081917",
   "hostnames": ["hostnames": [
      "99-46-189-78.lightspeed.tukrga.sbcglobal.net""99-46-189-78.lightspeed.tukrga.sbcglobal.net"
   ],],
   "org": "AT&T U-verse","org": "AT&T U-verse",
   "guid": "1664007502:75a821e2-7e89-11e3-8080-808080808080","guid": "1664007502:75a821e2-7e89-11e3-8080-808080808080",
   "data": "NTP\nxxx.xxx.xxx.xxx:7546\n68.94.157.2:123\n68.94.156.17:123","data": "NTP\nxxx.xxx.xxx.xxx:7546\n68.94.157.2:123\n68.94.156.17:123",
   "port": 123,"port": 123,
   "isp": "AT&T U-verse","isp": "AT&T U-verse",
   "asn": "AS7018","asn": "AS7018",
   "location": {"location": {
      "country_code3": "USA","country_code3": "USA",
      "city": "Atlanta","city": "Atlanta",
      "postal_code": "30328","postal_code": "30328",
      "longitude": -84.3972,"longitude": -84.3972,
      "country_code": "US","country_code": "US",
      "latitude": 33.93350000000001,"latitude": 33.93350000000001,
      "country_name": "United States","country_name": "United States",
      "area_code": 404,"area_code": 404,
      "dma_code": 524,"dma_code": 524,
      "region_code": null"region_code": null
   },},
   "ip": 1664007502,"ip": 1664007502,
   "domains": ["domains": [
      "sbcglobal.net""sbcglobal.net"
   ],],
   "ip_str": "99.46.189.78","ip_str": "99.46.189.78",
   "os": null,"os": null,
   "opts": {"opts": {
      "raw": "\\x97\\x00\\x03*\\x00\\x03\\x00H\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x01G\\x06\\xa7\\x8ec.\\xbdN\\x00\\x00\\x00\\x01\\x1dz\\x07\\x02\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00q\\x00\\x00\\x00i\\x00\\x00\\x00\\x00\\x00\\x00\\x00XD^\\x9d\\x02c.\\xbdN\\x00\\x00\\x00\\x01\\x00{\\x04\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00q\\x00\\x00\\x00o\\x00\\x00\\x00\\x00\\x00\\x00\\x00YD^\\x9c\\x11c.\\xbdN\\x00\\x00\\x00\\x01\\x00{\\x04\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00","raw": "\\x97\\x00\\x03*\\x00\\x03\\x00H\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x01G\\x06\\xa7\\x8ec.\\xbdN\\x00\\x00\\x00\\x01\\x1dz\\x07\\x02\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00q\\x00\\x00\\x00i\\x00\\x00\\x00\\x00\\x00\\x00\\x00XD^\\x9d\\x02c.\\xbdN\\x00\\x00\\x00\\x01\\x00{\\x04\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00q\\x00\\x00\\x00o\\x00\\x00\\x00\\x00\\x00\\x00\\x00YD^\\x9c\\x11c.\\xbdN\\x00\\x00\\x00\\x01\\x00{\\x04\\x04\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00\\x00",
      "ntp": {"ntp": {
         "more": false"more": false
      }}
   }}
}}

```

## 附录B：搜索语法关键词列表

### 常用语法


|      过滤器名      |            描述            |   类型    |                    举例                    |
| :------------: | :----------------------: | :-----: | :--------------------------------------: |
|     after      | 只显示给出日期之后的结果(dd/mm/yyyy) | string  |           `after:"04/02/2017"`           |
|      asn       |          自治系统号码          | string  |              `asn:"AS4130"`              |
|     before     | 只显示给出日期之前的结果(dd/mm/yyyy) | string  |          `before:"04/02/2017"`           |
|    category    |    现有的分类：ics,malware     | string  |           `category:"malware"`           |
|      city      |          城市的名字           | string  |            `city:"San Diego"`            |
|    country     |           国家简写           | string  |      `country:"ES"` `country:"CN"`       |
|      geo       |           经纬度            | string  |          `geo:"46.9481,7.4474"`          |
|      hash      |         数据的hash值         |   int   |                `-hash:0`                 |
|    has_ipv6    |         是否是IPv6          | boolean |             `has_ipv6:true`              |
| has_screenshot |          是否有截图           | boolean |          `has_screenshot:true`           |
|    hostname    |          主机名或域名          | string  |           `hostname:"google"`            |
|       ip       |           ip地址           | string  |           `ip:"54.67.82.248 "`           |
|      isp       |          ISP供应商          | string  |          `isp:"China Telecom"`           |
|      org       |          组织或公司           | string  |              `org:"google"`              |
|       os       |           操作系统           | string  |          `os:"Windows 7 or 8"`           |
|      port      |           端口号            |   int   |                `port:21`                 |
|     postal     |       邮政编码(仅限于美国)        | string  |             `postal:"98221"`             |
|    product     |          软件、平台           | string  | `product:"Apache httpd"` `product:"openssh"` |
|     region     |         地区或国家别名          | string  |                    -                     |
|     state      |                          | string  |                    -                     |
|      net       |       CIDR格式的IP地址        | string  |           `net:190.30.40.0/24`           |
|    version     |           软件版本           | string  |            `version:"2.6.1"`             |
|      vuln      |        漏洞的CVE ID         | string  |           `vuln:CVE-2014-0723`           |

#### HTTP过滤器

|           名称            |       描述       |   类型   |
| :---------------------: | :------------: | :----: |
|     http.component      | 网站上所使用的网络技术名称  | string |
| http.component_category | 网站上使用的网络组件的类别  | string |
|        http.html        |   Web banner   | string |
|     http.html_hash      |   网站HTML的哈希值   |  int   |
|       http.status       |     响应状态码      |  int   |
|       http.title        | 网站title得banner | string |

#### NTP 过滤器


|      名称      |           描述            |   类型    |
| :----------: | :---------------------: | :-----: |
|    ntp.ip    |  查找在其monlist中NTP服务器的IP  |         |
| ntp.ip_count |    初始monlist返回的IP数量     |   int   |
|   ntp.more   | 真/假; monlist集是否有更多的IP地址 | boolean |
|   ntp.port   |   monlist中的IP地址使用的端口    |   int   |


#### SSL过滤器


|          名称          |                   描述                   |     类型     |
| :------------------: | :------------------------------------: | :--------: |
|       has_ssl        |                 有无SSL                  |  boolean   |
|         SSL          |               搜索所有SSL的数据               |   string   |
|       ssl.alpn       |             诸如HTTP/2的应用层协议             |   string   |
|   ssl.chain_count    |                链中的证书数量                 |    int     |
|     ssl.version      | 可能的值：SSLv2，SSLv3，TLSv1，TLSv1.1，TLSv1.2 |   string   |
|     ssl.cert.alg     |                  证书算法                  |   string   |
|   ssl.cert.expired   |                是否是过期证书                 |  boolean   |
|  ssl.cert.extension  |                证书中的扩展名                 |   string   |
|   ssl.cert.serial    |             序列号为整数或十六进制字符串             | int/string |
| ssl.cert.pubkey.bits |                 公钥的位数                  |    int     |
| ssl.cert.pubkey.type |                  公钥类型                  |   string   |
|  ssl.cipher.version  |               SSL版本的首选密码               |   string   |
|   ssl.cipher.bits    |                首选密码中的位数                |    int     |
|   ssl.cipher.name    |                首选密码的名称                 |   string   |

#### Telnet 过滤器

|      名称       |           描述            |   类型   |
| :-----------: | :---------------------: | :----: |
| telnet.option |         搜索所有选项          | string |
|   telnet.do   |   对方执行的请求或期望对方执行指示的选项   | string |
|  telnet.dont  | 对方停止执行的请求或不再期望对方执行指定的选项 | string |
|  telnet.will  |      确认现在正在执行指定的选项      | string |
|  telnet.wont  |    表示拒绝执行或继续执行指定的选项     | string |


## 附录C：Facets搜索

### 常用Facets

|       名称       |        描述        |
| :------------: | :--------------: |
|      asn       |      自治系统号码      |
|      city      |      城市的全名       |
|    country     |      国家的全名       |
|     domain     |      设备的域名       |
| has_screenshot |     有无可用的截图      |
|      isp       |     ISP管理网络块     |
|      link      |     网络连接的类型      |
|      org       |     拥有该网块的组织     |
|       os       |       操作系统       |
|      port      |      服务的端口号      |
|     postal     |       邮政编码       |
|    product     |     软件/产品的名称     |
|     region     |     地区/国家的名称     |
|     state      |      区域的别名       |
|     uptime     | 主机启动的时间(以秒为单位计算) |
|      vuln      |    漏洞的CVE ID     |

### HTTP Facets

|           名称            |      描述       |   类型   |
| :---------------------: | :-----------: | :----: |
|     http.component      | 网站上使用的网络技术的名称 | string |
| http.component_category | 网站上使用的网络组件的类别 | string |
|     http.html_hash      |  HTML网站的哈希值   |  int   |
|       http.status       |     响应状态码     |  int   |

### NTP Facets


|      名称      |            描述            |
| :----------: | :----------------------: |
|    ntp.ip    |      monlist返回的IP地址      |
| ntp.ip_count |     初始monlist返回的IP数量     |
|   ntp.more   | 真假; monlist收集的是否有更多的IP地址 |
|   ntp.port   |    monlist中的IP地址使用的端口    |

### SSH Facets

|       名称        |            描述            |
| :-------------: | :----------------------: |
|   ssh.cipher    |          密码的名称           |
| ssh.fingerprint |          设备的指纹           |
|     ssh.mac     | 使用的MAC算法名称（例如：hmac-sha1） |
|    ssh.type     |   认证密钥的类型（例如：ssh-rsa）    |

### SSL Facets

|          名称          |     描述     |
| :------------------: | :--------: |
|     ssl.version      |  支持SSL版本   |
|       ssl.alpn       |   应用层协议    |
|   ssl.chain_count    |  链中的证书数量   |
|     ssl.cert.alg     |    证书算法    |
|   ssl.cert.expired   | 真假; 证书过期与否 |
|   ssl.cert.serial    |  证书序列号为整数  |
|  ssl.cert.extension  |   证书扩展名    |
| ssl.cert.pubkey.bits |   公钥的位数    |
|   ssl.cert.pubkey    |  公钥类型的名称   |
|   ssl.cipher.bits    |  首选密码中的位数  |
|   ssl.cipher.name    |  首选密码的名称   |
|  ssl.cipher.version  | SSL版本的首选密码 |

### Telnet Facets

|      名称       |           描述            |   类型   |
| :-----------: | :---------------------: | :----: |
| telnet.option |         显示所有选项          | string |
|   telnet.do   |   对方执行的请求或期望对方执行指示的选项   | string |
|  telnet.dont  | 对方停止执行的请求或不再期望对方执行指定的选项 | string |
|  telnet.will  |       服务器支持指定的选项        | string |
|  telnet.wont  |       服务器不支持指定的选项       | string |


## 附录D：端口列表

| Port  |                            Service |
| :---- | ---------------------------------: |
| 7     |                               Echo |
| 11    |                             Systat |
| 13    |                            Daytime |
| 15    |                            Netstat |
| 17    |                   Quote of the day |
| 19    |                Character generator |
| 21    |                                FTP |
| 22    |                                SSH |
| 23    |                             Telnet |
| 25    |                               SMTP |
| 26    |                                SSH |
| 37    |                              rdate |
| 49    |                            TACACS+ |
| 53    |                                DNS |
| 67    |                               DHCP |
| 69    |                   TFTP, BitTorrent |
| 70    |                             Gopher |
| 79    |                             Finger |
| 80    |                      HTTP, malware |
| 81    |                      HTTP, malware |
| 82    |                      HTTP, malware |
| 83    |                               HTTP |
| 84    |                               HTTP |
| 88    |                           Kerberos |
| 102   |                         Siemens S7 |
| 104   |                              DICOM |
| 110   |                               POP3 |
| 111   |                         Portmapper |
| 113   |                             identd |
| 119   |                               NNTP |
| 123   |                                NTP |
| 129   |        Password generator protocol |
| 137   |                            NetBIOS |
| 143   |                               IMAP |
| 161   |                               SNMP |
| 175   |              IBM Network Job Entry |
| 179   |                                BGP |
| 195   |                          TA14-353a |
| 311   |                OS X Server Manager |
| 389   |                               LDAP |
| 389   |                              CLDAP |
| 443   |                              HTTPS |
| 443   |                               QUIC |
| 444   |          TA14-353a, Dell SonicWALL |
| 445   |                                SMB |
| 465   |                              SMTPS |
| 500   |                          IKE (VPN) |
| 502   |                             Modbus |
| 503   |                             Modbus |
| 515   |                Line Printer Daemon |
| 520   |                                RIP |
| 523   |                            IBM DB2 |
| 554   |                               RTSP |
| 587   |               SMTP mail submission |
| 623   |                               IPMI |
| 626   |                OS X serialnumbered |
| 636   |                              LDAPS |
| 666   |                             Telnet |
| 771   |                           Realport |
| 789   |                   Redlion Crimson3 |
| 873   |                              rsync |
| 902   |              VMWare authentication |
| 992   |                    Telnet (secure) |
| 993   |                      IMAP with SSL |
| 995   |                      POP3 with SSL |
| 1010  |                            malware |
| 1023  |                             Telnet |
| 1025  |                           Kamstrup |
| 1099  |                           Java RMI |
| 1177  |                            malware |
| 1200  |                            Codesys |
| 1234  |                              udpxy |
| 1400  |                              Sonos |
| 1434  |                     MS-SQL monitor |
| 1515  |                            malware |
| 1521  |                         Oracle TNS |
| 1604  |                    Citrix, malware |
| 1723  |                               PPTP |
| 1741  |                         CiscoWorks |
| 1833  |                               MQTT |
| 1900  |                               UPnP |
| 1911  |                        Niagara Fox |
| 1962  |                             PCworx |
| 1991  |                            malware |
| 2000  |   iKettle, MikroTik bandwidth test |
| 2081  |                     Smarter Coffee |
| 2082  |                             cPanel |
| 2083  |                             cPanel |
| 2086  |                                WHM |
| 2087  |                                WHM |
| 2123  |                              GTPv1 |
| 2152  |                              GTPv1 |
| 2181  |                   Apache Zookeeper |
| 2222  |             SSH, PLC5, EtherNet/IP |
| 2323  |                             Telnet |
| 2332  |           Sierra wireless (Telnet) |
| 2375  |                             Docker |
| 2376  |                             Docker |
| 2379  |                               etcd |
| 2404  |                            IEC-104 |
| 2455  |                            CoDeSys |
| 2480  |                           OrientDB |
| 2628  |                         Dictionary |
| 3000  |                               ntop |
| 3260  |                              iSCSI |
| 3306  |                              MySQL |
| 3310  |                             ClamAV |
| 3386  |                              GTPv1 |
| 3388  |                                RDP |
| 3389  |                                RDP |
| 3460  |                            malware |
| 3541  |                            PBX GUI |
| 3542  |                            PBX GUI |
| 3689  |                               DACP |
| 3702  |                              Onvif |
| 3780  |                         Metasploit |
| 3787  |                           Ventrilo |
| 4000  |                            malware |
| 4022  |                              udpxy |
| 4040  |      Deprecated Chef web interface |
| 4063  |                     ZeroC Glacier2 |
| 4064  |            ZeroC Glacier2 with SSL |
| 4070  |    HID VertX/ Edge door controller |
| 4157  |                      DarkTrack RAT |
| 4369  |                               EPMD |
| 4443  |      Symantec Data Center Security |
| 4444  |                            malware |
| 4500  |                    IKE NAT-T (VPN) |
| 4567  |                Modem web interface |
| 4664  |                              Qasar |
| 4730  |                            Gearman |
| 4782  |                              Qasar |
| 4800  |                         Moxa Nport |
| 4840  |                             OPC UA |
| 4911  |               Niagara Fox with SSL |
| 4949  |                              Munin |
| 5006  |                           MELSEC-Q |
| 5007  |                           MELSEC-Q |
| 5008  |                        NetMobility |
| 5009  |       Apple Airport Administration |
| 5060  |                                SIP |
| 5094  |                            HART-IP |
| 5222  |                               XMPP |
| 5269  |              XMPP Server-to-Server |
| 5353  |                               mDNS |
| 5357  |              Microsoft-HTTPAPI/2.0 |
| 5432  |                         PostgreSQL |
| 5577  |                           Flux LED |
| 5601  |                             Kibana |
| 5632  |                         PCAnywhere |
| 5672  |                           RabbitMQ |
| 5900  |                                VNC |
| 5901  |                                VNC |
| 5938  |                         TeamViewer |
| 5984  |                            CouchDB |
| 6000  |                                X11 |
| 6001  |                                X11 |
| 6379  |                              Redis |
| 6666  |        Voldemort database, malware |
| 6667  |                                IRC |
| 6881  |                     BitTorrent DHT |
| 6969  |                   TFTP, BitTorrent |
| 7218  |           Sierra wireless (Telnet) |
| 7474  |                     Neo4j database |
| 7548  |                       CWMP (HTTPS) |
| 7777  |                             Oracle |
| 7779  |               Dell Service Tag API |
| 8008  |                         Chromecast |
| 8009  |                        Vizio HTTPS |
| 8010  |                      Intelbras DVR |
| 8060  |                 Roku web interface |
| 8069  |                            OpenERP |
| 8087  |                               Riak |
| 8090  |                        Insteon HUB |
| 8099  |                      Yahoo SmartTV |
| 8112  |                      Deluge (HTTP) |
| 8126  |                             StatsD |
| 8139  |                       Puppet agent |
| 8140  |                      Puppet master |
| 8181  |           GlassFish Server (HTTPS) |
| 8333  |                            Bitcoin |
| 8334  |      Bitcoin node dashboard (HTTP) |
| 8443  |                              HTTPS |
| 8554  |                               RTSP |
| 8800  |                               HTTP |
| 8880  |                     Websphere SOAP |
| 8888  |                   HTTP, Andromouse |
| 8889  |          SmartThings Remote Access |
| 9000  |                        Vizio HTTPS |
| 9001  |                             Tor OR |
| 9002  |                             Tor OR |
| 9009  |                              Julia |
| 9042  |                      Cassandra CQL |
| 9051  |                        Tor Control |
| 9100  |               Printer Job Language |
| 9151  |                        Tor Control |
| 9160  |                   Apache Cassandra |
| 9191  |             Sierra wireless (HTTP) |
| 9418  |                                Git |
| 9443  |            Sierra wireless (HTTPS) |
| 9595  |           LANDesk Management Agent |
| 9600  |                              OMRON |
| 9633  |                      DarkTrack RAT |
| 9869  |                         OpenNebula |
| 10001 |               Automated Tank Gauge |
| 10001 |                           Ubiquiti |
| 10243 |              Microsoft-HTTPAPI/2.0 |
| 10554 |                               RTSP |
| 11211 |                           Memcache |
| 12345 |                            malware |
| 17000 |                    Bose SoundTouch |
| 17185 |                     VxWorks WDBRPC |
| 12345 |           Sierra wireless (Telnet) |
| 11300 |                          Beanstalk |
| 13579 | Media player classic web interface |
| 14147 |                      Filezilla FTP |
| 16010 |                       Apache Hbase |
| 16992 |                          Intel AMT |
| 16993 |                          Intel AMT |
| 18245 |              General Electric SRTP |
| 20000 |                               DNP3 |
| 20547 |                           ProconOS |
| 21025 |                          Starbound |
| 21379 |                       Matrikon OPC |
| 23023 |                             Telnet |
| 23424 |                            Serviio |
| 25105 |                        Insteon Hub |
| 25565 |                          Minecraft |
| 27015 | Steam A2S server query, Steam RCon |
| 27016 |             Steam A2S server query |
| 27017 |                            MongoDB |
| 28015 |             Steam A2S server query |
| 28017 |                     MongoDB (HTTP) |
| 30313 |                 Gardasoft Lighting |
| 30718 |                    Lantronix Setup |
| 32400 |                               Plex |
| 37777 |                         Dahuva DVR |
| 44818 |                        EtherNet/IP |
| 47808 |                             Bacnet |
| 49152 |                  Supermicro (HTTP) |
| 49153 |                          WeMo Link |
| 50070 |                      HDFS Namenode |
| 51106 |                      Deluge (HTTP) |
| 53413 |                     Netis backdoor |
| 54138 |                        Toshiba PoS |
| 55443 |                             McAfee |
| 55553 |                         Metasploit |
| 55554 |                         Metasploit |
| 62078 |                      Apple iDevice |
| 64738 |                             Mumble |


## 附录E: SSL banner示例

```json
{
    "hostnames": [],
    "title": "",
    "ip": 2928565374,
    "isp": "iWeb Technologies",
    "transport": "tcp",
    "data": "HTTP/1.1 200 OK\r\nExpires: Sat, 26 Mar 2016 11:56:36 GMT\r\nExpire\
s: Fri, 28 May 1999 00:00:00 GMT\r\nCache-Control: max-age=2592000\r\nCache-Cont\
rol: no-store, no-cache, must-revalidate\r\nCache-Control: post-check=0, pre-che\
ck=0\r\nLast-Modified: Thu, 25 Feb 2016 11:56:36 GMT\r\nPragma: no-cache\r\nP3P:\
 CP=\"NON COR CURa ADMa OUR NOR UNI COM NAV STA\"\r\nContent-type: text/html\r\n\
Transfer-Encoding: chunked\r\nDate: Thu, 25 Feb 2016 11:56:36 GMT\r\nServer: sw-\
cp-server\r\n\r\n",
 "asn": "AS32613",
    "port": 8443,
    "ssl": {
        "chain": ["-----BEGIN CERTIFICATE-----\nMIIDszCCApsCBFBTb4swDQYJKoZIhvcN\
AQEFBQAwgZ0xCzAJBgNVBAYTAlVTMREw\nDwYDVQQIEwhWaXJnaW5pYTEQMA4GA1UEBxMHSGVybmRvbj\
ESMBAGA1UEChMJUGFy\nYWxsZWxzMRgwFgYDVQQLEw9QYXJhbGxlbHMgUGFuZWwxGDAWBgNVBAMTD1Bh\
cmFs\nbGVscyBQYW5lbDEhMB8GCSqGSIb3DQEJARYSaW5mb0BwYXJhbGxlbHMuY29tMB4X\nDTEyMDkx\
NDE3NTUyM1oXDTEzMDkxNDE3NTUyM1owgZ0xCzAJBgNVBAYTAlVTMREw\nDwYDVQQIEwhWaXJnaW5pYT\
EQMA4GA1UEBxMHSGVybmRvbjESMBAGA1UEChMJUGFy\nYWxsZWxzMRgwFgYDVQQLEw9QYXJhbGxlbHMg\
UGFuZWwxGDAWBgNVBAMTD1BhcmFs\nbGVscyBQYW5lbDEhMB8GCSqGSIb3DQEJARYSaW5mb0BwYXJhbG\
xlbHMuY29tMIIB\nIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxc9Vy/qajKtFFnHxGOFPHTxm\
\nSOnsffWBTBfyXnK3h8u041VxvZDh3XkpA+ptg2fWOuIT0TTYuqw+tqiDmg8YTsHy\njcpMFBtXV2cV\
dhKXaS3YYlM7dP3gMmkGmH+ZvCgCYc7L9MIJxYJy6Zeuh67YxEMV\ngiU8mZpvc70Cg5WeW1uBCXtUAi\
jDLsVWnhsV3YuxlweEvkRpAk3EHehKbvgMnEZS\nQ30QySe0GAqC7bWzKrwsJAOUk/+Js18+3QKb/LmD\
a9cRjtFCTo6hYfPbfHj8RxQh\n4Xmnn/CtZ48wRQTqKXSO6+Zk3OuU7/jX1Gt/jxN6n77673e6uCsggT\
wut/EtNwID\nAQABMA0GCSqGSIb3DQEBBQUAA4IBAQBb/yTy76Ykwr7DBOPAXc766n73OsZizjAt\n1k\
mx7LxgN3X/wFxD53ir+sdOqbPgJl3edrE/ZG9dNl6LhUBbUK+9s6z9QicEfSxo\n4uQpFSywbGGmXInE\
ZmyT4SsOLi/hNgy68f49LO1h6rn/p7QgIKd31g7189ZfFkFb\nRdD49s1l/Cc5Nm4XapUVvmnS91MlPk\
/OOIg1Lu1rYkuc8sIoZdPbep52H3Ga7TjG\nkmO7nUIii0goB7TQ63mU67+NWHAmQQ8CtCDCN49kJyen\
1WFjD6Je2U4q0IFQrxHw\nMy+tquo/n/sa+NV8QOj1gMVcFsLhYm7Z5ZONg0QFXSAL+Eyj/AwZ\n----\
-END CERTIFICATE-----\n"],
        "cipher": {
            "version": "TLSv1/SSLv3",
            "bits": 256,
            "name": "DHE-RSA-AES256-GCM-SHA384"
        },
        "alpn": [],
        "dhparams": {
            "prime": "b10b8f96a080e01dde92de5eae5d54ec52c99fbcfb06a3c69a6a9dca52\
d23b616073e28675a23d189838ef1e2ee652c013ecb4aea906112324975c3cd49b83bfaccbdd7d90\
c4bd7098488e9c219a73724effd6fae5644738faa31a4ff55bccc0a151af5f0dc8b4bd45bf35c1a65e68cfda76d4da708df1fb2bc2e4a4371",
            "public_key": "2e30a6e455730b2f24bdaf5986b9f0876068d4aa7a4e15c9a1b9c\
a05a420e8fd3b496f7781a9423d3475f0bedee83f0391aaa95a738c8f0e250a8869a86d41bdb0194\
66dba5c641e4b2b4b82db4cc2d4ea8d9804ec00514f30a4b6ce170b81c3e1ce4b3d17647c8e5b8f6\
65bb7f588100bcc9a447d34d728c3709fd8a5b7753b",
            "bits": 1024,
            "generator": "a4d1cbd5c3fd34126765a442efb99905f8104dd258ac507fd6406c\
ff14266d31266fea1e5c41564b777e690f5504f213160217b4b01b886a5e91547f9e2749f4d7fbd7\
d3b9a92ee1909d0d2263f80a76a6a24c087a091f531dbf0a0169b6a28ad662a4d18e73afa32d779d\
5918d08bc8858f4dcef97c2a24855e6eeb22b3b2e5",
            "fingerprint": "RFC5114/1024-bit MODP Group with 160-bit Prime Order\
 Subgroup"
        },
        "versions": ["TLSv1", "-SSLv2", "SSLv3", "TLSv1.1", "TLSv1.2"]
    },
    "html": "\n\t\t<html><head>\n\t\t<meta charset=\"utf-8\">\n\t\t<meta http-eq\
uiv=\"X-UA-Compatible\" 
content=\"IE=edge,chrome=1\">\n\t\t<title></title>\n\t\t\
<script language=\"javascript\" type=\"text/javascript\" src=\"/javascript/commo\
n.js?plesk_version=psa-11.0.9-110120608.16\"/></script>\n\t\t<script language=\"\
javascript\" type=\"text/javascript\" src=\"/javascript/prototype.js?plesk_versi\
on=psa-11.0.9-110120608.16\"></script>\n\t\t<script>\n\t\t\tvar opt_no_frames = \
false;\n\t\t\tvar opt_integrated_mode = false;\n\t\t</script>\n\t\t\n\t\t</head>\
<body onLoad=\";top.location='/login.php3?window_id=&amp;requested_url=https%3A%\
2F%2F174.142.92.126%3A8443%2F';\"></body><noscript>You will be redirected to the\
 new address in 15 seconds... If you are not automatically taken to the new loca\
tion, please enable javascript or click the hyperlink <a href=\"/login.php3?wind\
ow_id=&amp;requested_url=https%3A%2F%2F174.142.92.126%3A8443%2F\" target=\"top\"\
>/login.php3?window_id=&amp;requested_url=https%3A%2F%2F174.142.92.126%3A8443%2F\
</a>.</noscript></html><!--_____________________________________________________\
________________________________________________________________________________\
________________________________________________________________________________\
_________________________IE error page size limitation__________________________\
________________________________________________________________________________\
________________________________________________________________________________\
____________________________________________________-->",
    "location": {
        "city": null,
        "region_code": "QC",
        "area_code": null,
        "longitude": -73.5833,
        "country_code3": "CAN",
        "latitude": 45.5,
        "postal_code": "H3G",
        "dma_code": null,
        "country_code": "CA",
        "country_name": "Canada"
    },
    "timestamp": "2016-02-25T11:56:52.548187",
    "domains": [],
    "org": "iWeb Technologies",
     "os": null,
    "_shodan": {
        "options": {},
        "module": "https",
        "crawler": "122dd688b363c3b45b0e7582622da1e725444808"
    },
    "opts": {
        "heartbleed": "2016/02/25 03:56:45 ([]uint8) {\n 00000000  02 00 74 63 6\
5 6e 73 75  73 2e 73 68 6f 64 61 6e  |..tcensus.shodan|\n 00000010  2e 69 6f 53 \
45 43 55 52  49 54 59 20 53 55 52 56  |.ioSECURITY SURV|\n 00000020  45 59 fe 7a\
 a2 0d fa ed  93 42 ed 18 b0 15 7d 6e  |EY.z.....B....}n|\n 00000030  29 08 f6 f\
8 ce 00 b1 94  b5 4b 47 ac dd 18 aa b9  |)........KG.....|\n 00000040  db 1c 01 \
45 95 10 e0 a2  43 fe 8e ac 88 2f e8 75  |...E....C..../.u|\n 00000050  8b 19 5f\
 8c e0 8a 80 61  56 3c 68 0f e1 1f 73 9e  |.._....aV<h...s.|\n 00000060  61 4f d\
a db 90 ce 84 e3  79 5f 9d 6c a0 90 ff fa  |aO......y_.l....|\n 00000070  d8 16 \
e8 76 07 b2 e5 5e  8e 3e a4 45 61 2f 6a 2d  |...v...^.>.Ea/j-|\n 00000080  5d 11\
74 94 03 3c 5d                              |].t..<]|\n}\n\n2016/02/25 03:56:45\
 174.142.92.126:8443 - VULNERABLE\n",
        "vulns": ["CVE-2014-0160"]
    },
    "ip_str": "174.142.92.126"
}
```


## Refer
- [28 public asciicasts by Shodan](https://asciinema.org/~Shodan):https://asciinema.org/~Shodan
- [Shodan新手入坑指南](http://www.freebuf.com/sectool/121339.html):http://www.freebuf.com/sectool/121339.html
- [Complete Guide to Shodan](https://leanpub.com/shodan)：https://leanpub.com/shodan
- [shodan - The official Python library for the Shodan search engine](http://shodan.readthedocs.io/en/latest/index.html)：http://shodan.readthedocs.io/en/latest/index.html
- [Create GIFs from a Shodan json.gz file using the API](https://gist.github.com/achillean/963eea552233d9550101)：https://gist.github.com/achillean/963eea552233d9550101https://gist.github.com/achillean/963eea552233d9550101
- [工控安全入门分析](http://drops.wooyun.org/tips/8594)：http://drops.wooyun.org/tips/8594
- [灯塔实验室](http://plcscan.org/blog/)：http://plcscan.org/blog/
- [网络空间工控设备的发现与入侵](https://github.com/evilcos/papers)：https://github.com/evilcos/papers
- [工控](http://b404.xyz/2017/10/27/industrial-control/)：http://b404.xyz/2017/10/27/industrial-control/
- [Shodan的http.favicon.hash语法详解](https://mp.weixin.qq.com/s/B9DdQGUZR1GHBPnNACPRKA)

![2](D:\我的文章\投稿文章\jirairya\Shodan——从入门到放弃(md版本)\Shodan——从入门到放弃(md版本)\2.png)
