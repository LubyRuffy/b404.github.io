---
title: '一键无文件感染'
layout: post
tags: 
  - translate
  - sec
  - trojan
  - virus
category: 
  - translate
  - sec
  - hack
  - paper
comments: true
share: true
description: 恶意软件作者可以在不将文件写入磁盘的情况下损害计算机，这种技术可以在逃脱文件扫描软件检测威胁性的同时，仍然保持持久性损害。
---

恶意软件作者可以在不将文件写入磁盘的情况下损害计算机，这种技术可以在逃脱文件扫描软件检测威胁性的同时，仍然保持持久性损害。

* TOC
{:toc}

<!--more-->

## 摘要



本文将阐述不同的无文件感染方法，以及一种经典的使用无PE文件进行欺骗点击进而执行无文件感染的新手法。

传统的恶意软件隐藏于磁盘上的文件中，为了进行长期隐藏感染，注册表运行密钥链接到此文件。
使用无文件感染的恶意程序在受感染的计算机上，不会作为普通文件存在，反而会以脚本形式存在于计算机注册表的子项中，如Windows PowerShell，VBScript、JavaScript。每次Windows启动时都会调用注册表中的payload。

我们所见的一键无文件感染技术常使用脚本语言实现，比如JavaScript。它一般借助.hta文件在计算机上实现，`.hta`文件将JavaScript payload放入注册表子项，每次Windows启动时，通过调用以下命令来触发JavaScript代码:

```c
rundll32.exe javascript:"\..\mshtml,RunHTMLApplication ";alert('payload');
```
然后JavaScript代码从另一个子项读取和解码加密的数据。该数据将`payload`注入存储器。每隔几分钟，`payload`就会检查其注册表项。如果注册表子项内容被删除，则该`payload`又会重新创建，从而使感染持久进行。

我们所知第一个使用无文件感染技术切造成大量危害的是2014年出现的[Trojan.Poweliks [1]](https://www.symantec.com/security_response/writeup.jsp?docid=2014-080408-5614-99)。随着这类木马技术的提高，许多其他类似的木马也随之出现，其中两个代表性的就是[Trojan.Bedep [2]](https://www.symantec.com/security_response/writeup.jsp?docid=2015-020903-0718-99)和[Trojan.Kotver [3]](https://www.symantec.com/security_response/writeup.jsp?docid=2015-082817-0932-99)。

这篇文章将阐述和比较恶意软件作者现今使用无文件感染的最常见方法，讨论一键无文件感染的相关内容。

## 引言

传统上，AV产品使用恶意程序中发现的字符串或代码签名来检测恶意程序。恶意程序有几种方法可以逃脱AV产品检测，其中之一就是将恶意代码放在内存中。[Korplug[4]](http://blog.trendmicro.com/trendlabs-security-intelligence/unplugging-plugx-capabilities/)就是一个使用内存存放恶意代码，逃脱检测的例子。它的侵染链包含加密文件的解码，在内存中加载可执行文件。这样做，可以在AV产品扫描文件时，保护代码。无文件类型感染在不同层面上实现文件不落地，硬盘上不会再有AV产品可以扫描的文件。即使恶意文件不在磁盘上，也可以持久侵害。

### 早期案例

引起研究人员注意的第一个无文件感染的恶意软件是[Trojan.Poweliks](http://www.symantec.com/content/en/us/enterprise/media/security_response/whitepapers/evolution-of-poweliks.pdf)，发现于2014年。Poweliks不作为文件存在于磁盘上，而是仅持久驻留在注册表中。[[5]](http://www.symantec.com/content/en/us/enterprise/media/security_response/whitepapers/evolution-of-poweliks.pdf)。Poweliks使用特殊的命名方式，隐藏在注册表中，并一直使用CLSID劫持作为注册表中的运行加载点。之后，还分析出Poweliks利用Microsoft Windows远程权限提升漏洞[（CVE-2015-0016）[6]](https://www.symantec.com/security_response/vulnerability.jsp?bid=71965)攻陷并控制存在缺陷的网站。而另一个无文件恶意程序`Trojan.Bedep`也使用相同的零日漏洞达到攻击目的。Bedep是一个仅存内存的下载器，与Poweliks有类似的编码风格。从`Poweliks`恶意程序到` Trojan.Bedep `和`Trojan.Kotver `，这类木马都采用了相同的技术手法。

### 如何工作

通常，当用户访问危险网站时，无文件恶意程序通过`exploit kits`（EK）实现攻击。
[Angler EK[7]](http://malware.dontneedcoffee.com/2014/08/angler-ek-now-capable-of-fileless.html)是第一个无需在驱动器上写入恶意程序而感染主机的EK，它提供shellcode，将无文件恶意程序注入到正在运行且易受攻击的进程中，例如iexplore.exe。

另外无文件恶意程序也可以通过恶意程序下载器实现攻击。通过恶意附件或垃圾邮件的恶意URL链接实现恶意程序下载器的下载，然后向磁盘写入恶意程序，然后在将无文件恶意程序注入内存，接着自动删除恶意程序。

![无文件感染链的危险分析.jpg](/img/OneClickInfection/无文件感染链的危险分析.jpg)

一旦恶意程序注入到内存中，恶意程序会加载并加密自身的二进制组件。将加密后的恶意程序副本保存在注册表中，然后创建另一个程序，该程序中有自启动脚本(该脚本可能是VBScript，JavaScript或PowerShell脚本)，负责解码二进制组件并将其加载到内存中。

然后被解码的二进制组件启动，并作为“看门狗“ 监视其创建的相关注册表项，负责接收恶意指令，控制`C&C`服务器。此恶意软件的后门功能包括重新安装注册表项，下载和执行文件以及调用其他命令。其中下载的一个文件，在执行后会将ad-click模块安装到内存中。

### 使用该技术的著名恶意程序

#### Poweliks

`Trojan.Poweliks`在基于文件感染的`Wowliks`恶意程序上发展成一个基于注册表感染的恶意软件。且其有下载PowerShell应用程序的行为，所以被命名为`Poweliks`。它使用PowerShell脚本将“看门狗”的DLL从注册表项注入到DLLHost.exe进程并运行，实现了持久存留。`Poweliks`的主要`payload`攻击目的是向受感染的用户进行广告欺诈和勒索。

#### Bedep
 
由于编码风格相似，且使用相同的`CVE-2015-0016 `exploit，`Trojan.Bedep`被认为与Poweliks有很大关联，但并没有确凿证据指出这两个恶意程序是出自同一个作者。经分析，Bedep会下载并安装Poweliks以及其他广告欺诈恶意软件。Bedep有32位和64位版本，通过改变自身文件属性达到伪装。这种恶意程序主要攻击目的是将受感染的计算机变成僵尸网络。

#### Kotver

在出现Poweliks无文件感染技术之前，Kotver恶意程序变体为了逃避杀软检测，也会存在于注册表中，只是像Poweliks一样，下载一个PowerShell应用程序，但并没有采用无文件感染，只是在没网络连接可用的情况下，它会基于文件感染，在磁盘上创建自己的副本。据分析，Kotver主要攻击目的是勒索和实现银行木马。

## 无文件感染的预备知识

开始阐述了不同感染体使用的不同感染媒介，随着深入，现在我们将讨论一种新的感染体，它使用经典的[一键点击欺诈式手法[8]](https://www.symantec.com/connect/blogs/one-click-fraudsters-extend-reach-learning-chinese)进行无文件感染。

首先，我们将逐个讨论所有的单个组件，让大家更易于理解一键无文件感染。

### MSHTA.EXE
`mshta.exe`程序通过WebBrowser控件，使用最小的用户界面（UI）运行受信任的HTML和脚本。

### HTA
随着时间推移，技术不断发展提高，Windows中的一些技术已不再适用。但自Windows NT（1993年7月发布）以来，存在一种这样强大的技术，并且仍然存在于Windows 10（2015年7月发布）），那就是HTML应用程序（HTA）。

[HTA[9]](https://technet.microsoft.com/en-us/library/ee692768.aspx)是Microsoft Windows程序，其源代码由HTML、动态HTML和Internet Explorer（IE）支持的脚本语言(如VBScript或JScript)组成。HTML用于生成用户界面，脚本语言用于控制程序逻辑。HTA执行是没有互联网浏览器安全模型的限制，而且它作为一个“完全信任”的应用程序执行。HTA常用文件扩展名为.hta。

所有当前的Windows操作系统都支持HTA文件执行。HTA看起来像一个HTML文件，但它具有比HTML文件更高的权限。HTA文件执行需要mshta.exe，`mshta.exe`随Internet Explorer一起安装。Mshta.exe通过实例化Internet Explorer渲染引擎（mshtml）以及任何所需的语言引擎（如vbscript.dll）来执行HTA 。

HTAs为用户提供了一种在充满复选框、单选按钮、下拉列表等具有其他Windows元素的图形用户界面（GUI）中加载脚本的方式。

但出于攻击目的，HTA只不过是为脚本提供GUI的一种方式。正如我们所知，WSH和VBScript都没有提供GUI元素，没有复选框，没有列表框等等。然而，Internet Explorer却使用了所有这些元素，甚至更多。由于HTA和IE相互依存，用户就可以利用所有这些GUI元素编写系统管理脚本。

HTML文件和HTAs有多少相近？只要用户将任意的HTML文件扩展名从.htm（或.html）更改为.hta，文件就成为HTA了。

![IE的架构.jpg](/img/OneClickInfection/IE的架构和程序.jpg)

### 为什么用户不只是用HTML文件？

答案非常简单：安全。理由就是，IE上有很多安全限制。而且用户不希望使用重置他们配置且高权限运行的客户端脚本。因此，许多系统管理脚本（如使用了WMI或ADSI的）在IE上运行会失败，情况稍好点的，会显示类似于以下内容的对话框：
![运行脚本弹出对话框.jpg](/img/OneClickInfection/运行脚本弹出对话框.jpg)
> 每当用户从HTML文件运行脚本时，都会显示类似的对话框。这虽然使用友好，但绝对不是最好的用户体验。

相比之下，HTAs没有IE那样的安全限制，HTAs与IE运行的也是不同的进程。HTAs在mshta.exe进程中运行，而不是iexplore.exe进程中。与HTML页面不同，HTAs可以运行客户端脚本，并且可以访问文件系统。除此之外，HTAs还可以运行用户的系统管理脚本，包括使用WMI和ADSI。而且用户的脚本运行正常，不会收到任何关于不安全的提示警告。

当然，这并不意味着HTAs可以随便无视Windows安全机制，HTA也有自己的一些安全策略。例如，如果一个用户没有权限更改其他用户的密码，则在该用户权限是不能使用脚本更改其他用户的密码，将该脚本放置在HTA中也没什么用，该用户仍然无法更改其他用户的密码。

不过，即使HTAs运行的进程不同于IE，但仍要使用Internet Explorer和IE对象模型，所以用户可以执行IE中不允许的其他任务和运行IE中不允许的脚本。

有关HTAs和安全性的更多信息，请参见[Microsoft Developer Network（MSDN）上的HTML Applications SDK页面[11]](https://msdn.microsoft.com/en-us/library/ms536473.aspx)

### 安全思考

当执行常规HTML文件时，它的执行也离不开Web浏览器的安全模型。也就是说，它的功能仅限于与服务器通信，操纵页面的对象模型（通常用于验证表单和创建有趣的视觉效果）和读写cookies。

另一方面，HTA作为完全受信任的应用程序运行，因此具有比普通HTML文件更多的权限(例如，HTA可以创建，编辑和删除文件和注册表项)。尽管如此，但查询`Active Directory`仍可能会受到Internet Explorer Zone逻辑和相关错误消息的影响。

## 一键无文件感染

在分析文件感染及其感染媒介后，可知如果攻击者使用非PE文件感染，常用的就是知名的感染媒介——`一键点击式欺骗`(one-click fraud)。

一键点击式欺骗并不新鲜，它已经存在了十多年，并且已经影响了大多数亚洲国家，最具代表性的是日本。通常情况下，一键式欺诈者通过诱骗用户订阅虚假的成人视频服务来进行一键点击欺诈攻击，也有两键、三键和四键点击欺骗（甚至是[零键点击[12]](https://www.symantec.com/connect/blogs/rise-japanese-zero-click-fraud))。一键点击欺诈的攻击行为类似勒索软件，它也会锁定用户的屏幕，并重复弹出窗口，直至用户注册、订阅或支付一定金额后才会终止该恶意行为。

经分析可知，恶意程序攻击者可能会将无文件感染和一击欺诈混合在一起，从而创建一键无文件感染。简而言之：

**无文件感染+一击欺诈=一键无文件感染**

### 内存感染

可以使用HTA文件创建一个ActiveX对象，然后该对象将JS Payload注入到运行注册表项中。PowerShell和WSCRIPT（VBS）也可以实现同样的功能。

![攻击流程.jpg](/img/OneClickInfection/攻击流程.jpg)

### 概念证明

下面所示代码是一个存在于被攻击服务器上HTA文件的源码。假设一个感染情景，用户被社工引诱或者是水坑攻击而访问某网站，然后该网站要求点击运行或执行某个事件，一旦点击执行，该事件将创建一个WScript ActiveX对象，然后创建运行注册表项，使用rundll32弹出JS警告（对于POC）而进行更多其它操作。
```html
/****************************POC*******************************************/

<html>
<head>
<title>RegTest</title>
<script language="JavaScript">
function writeInRegistry(sRegEntry, sRegValue)
{
   var regpath = "HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run\\" + sRegEntry;
   var oWSS = new ActiveXObject("WScript.Shell");
   oWSS.RegWrite(regpath, sRegValue, "REG_SZ");
}

function readFromRegistry(sRegEntry)
{
   var regpath = "HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run\\" + sRegEntry;        /*Payload injected in Run registry entry*/
   var oWSS = new ActiveXObject("WScript.Shell");    /*WASCRIPT ActiveX object created which is used to inject the Malicous JS in registry*/
   return oWSS.RegRead(regpath);
}
function tst()
{
   writeInRegistry("malware", "rundll32.exe javascript:\"\\..\\mshtml,RunHTMLApplication \";alert('payload'); "); /*Payload is the JS payload which does the real malicious stuff and it got watchdog, for keeping an eye over the registry entry which makes the infection persistent*/
   alert(readFromRegistry("malware"));
}
</script></head>
<body>

Click here to run test: <input type="button" value="Run" onclick="tst()"
</body>
</html>

/***************************POC end****************************************/
```

出于本文目的，选择了`Poweliks`作为案例，再加上它能从运行注册表项中释放JS，用它作为例子是不二选择。为了让感染持续存在，它被注入到运行注册表项，用户重新启动计算机，它又会自动重新注册。(因为内存中的感染会在计算机重新启动后消失.)

对于我们的POC，我们使用JS的Alert API演示(恶意攻击者可能会有所不同)，使用PowerShell或CSCRIPT也可以实现同样的效果。分析了之前无文件感染行为，我们发现它具有一个“看门狗”模块，它可以监控注册表项，如果用户删除它，它也会重新创建注册表项。

这种攻击手法，可能会影响从Windows 95到Windows 10的所有Windows版本，因为所有版本都有预安装IE，而随附IE的WSCRIPT是执行此攻击的唯一必需组件。


### 变体

以下可信应用程序也能达到同样攻击手段：

* [PowerShell [13]](https://blogs.msdn.microsoft.com/powershell/2006/11/14/its-a-wrap-windows-powershell-1-0-released)
* [CSCRIPT [14]](https://technet.microsoft.com/en-us/library/bb490887.aspx)
* [WSCRIPT [15]](https://msdn.microsoft.com/en-us/library/at5ydy31.aspx)


## 预防和解决

Symantec建议用户遵守以下几点，防止一键无文件感染攻击：

- 不要将HTA文件视为HTML文件
- 动态检测调用PowerShell，WSCRIPT，CSCRIPT，cmd，RUNDLL32或regserve32的独立注册表项
- 必要时进行手动清楚卸载（步骤如下）。

### 手动清除

1. 下载Microsoft的Process Explorer并执行。
2. 重启进入安全模式。
3. 选择父进程（恶意软件注入的进程）并终止它（杀死进程树）。
4. 打开注册表编辑器（运行 - > regedit.exe）。
5. 在左侧面板中，进入：`HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\Run\`
6. 清理注册表项。
7. 关闭注册表编辑器

### AV 解决方法
反病毒产品有以下方法定位无文件感染体的路径：

1. 内存扫描
2. 注册表扫描
3. 网络扫描。


## 结语 

[原文地址](https://www.virusbulletin.com/uploads/pdf/magazine/2017/VB2016-AnandMenrige.pdf)在: https://www.virusbulletin.com/uploads/pdf/magazine/2017/VB2016-AnandMenrige.pdf

该文章在[YouTube](https://www.youtube.com/watch?v=EIIFly_iR8Y)上的地址：https://www.youtube.com/watch?v=EIIFly_iR8Y，

[PPT](https://www.virusbulletin.com/uploads/pdf/conference_slides/2016/Anand_Menrige-vb-2016-One-Click-Fileless.pdf)地址为https://www.virusbulletin.com/uploads/pdf/conference_slides/2016/Anand_Menrige-vb-2016-One-Click-Fileless.pdf。

CHXX大佬也说现在很多病毒都采用无文件感染技术，文件不落地，杀软扫起来比较难。

由此可见，`无文件感染`是一种无痕迹的新型恶意攻击，再结合一键点击欺骗的攻击手法，这类病毒造成的危害就会很大的。所以在浏览网页和使用软件的时候需谨慎。

## 参考

[1] Trojan.Poweliks. https://www.symantec.com/security_response/writeup.jsp?docid=2014-080408-5614-99.

[2] Trojan.Bedep. https://www.symantec.com/security_response/writeup.jsp?docid=2015-020903-0718-99.

[3] Trojan.Kotver. https://www.symantec.com/security_response/writeup.jsp?docid=2015-082817-0932-99.

[4] Camba, A. Unplugging Plugx Capabilities. Trend Micro Malware Blog. http://blog.trendmicro.com/trendlabs-security-intelligence/unplugging-plugx-capabilities.

[5] O'Murchu, L.; Gutierrez, F. The evolution of the fileless click-fraud malware Poweliks. Symantec Connect Blog. http://www.symantec.com/content/en/us/enterprise/media/security_response/whitepapers/evolution-of-poweliks.pdf.

[6] Microsoft Windows Remote Privilege Escalation Vulnerability (CVE-2015-0016). https://www.symantec.com/security_response/vulnerability.jsp?bid=71965.

[7] Kafeine. Angler EK: now capable of "fileless" infection. Malware Don't Need Coffee Blog. http://malware.dontneedcoffee.com/2014/08/angler-ek-now-capable-of-fileless.html.

[8] Anand, H. One-click fraudsters extend reach by learning Chinese. Symantec Connect Blog. http://www.symantec.com/connect/blogs/one-click-fraudsters-extend-reach-learning-chinese.

[9] Extreme Makeover: Wrap Your Scripts Up in a GUI Interface. Microsoft Technet. https://technet.microsoft.com/en-us/library/ee692768.aspx.

[10] Recreated from the Microsoft TechNet (2013) gallery – IE Architecture. https://gallery.technet.microsoft.com/IE-Architecture-3bc7c3fd/file/78635/1/IE%20Architecture.png.

[11] HTML Applications SDK. https://msdn.microsoft.com/en-us/library/ms536473(vs.85).aspx.

[12] Hamada, J. The rise of Japanese zero-click fraud. Symantec Connect Blog. http://www.symantec.com/connect/blogs/rise-japanese-zero-click-fraud.

[13] It's a Wrap! Windows PowerShell 1.0 Released! Windows PowerShell Blog. https://blogs.msdn.microsoft.com/powershell/2006/11/14/its-a-wrap-windows-powershell-1-0-released.

[14] Using the command-based script host (CScript.exe). Microsoft Technet. https://technet.microsoft.com/en-us/library/bb490887.aspx.

[15] WScript Object. Microsoft Developer Network. https://msdn.microsoft.com/en-us/library/at5ydy31(v=vs.84).aspx.
