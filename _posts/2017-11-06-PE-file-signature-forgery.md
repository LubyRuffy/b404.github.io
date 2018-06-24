---
title: PE文件的签名伪造
layout: post
tags:
  - bin
  - reverse
  - pe
category: 
  - PE
  - hack
  - bin
comments: true
share: true
description: PE文件的Authenticode签名伪造
---

 PE文件的Authenticode签名伪造

* TOC
{:toc}

<!--more-->


## PE文件的签名伪造

查看`C:\Windows\System32\consent.exe`的文件属性，看到相关签名：

![](/img/hack/PE文件伪造/1508925341418.png)

使用PowerShell的`Get-AuthenticodeSignature `查看、并验证：

![](/img/hack/PE文件伪造/1508925559273.png)

使用CFF Explorer获取文件结构：

![](/img/hack/PE文件伪造/1508939728704.png)

> `Security Directory RVA`代码数字签名在PE文件中的偏移位置
> `Security Directory Size`代表数字签名长度

将该部分内容提取，复制到另一个文件test.exe的尾部，同时使用CFF Explorer修改test.exe对应的`Security Directory RVA`和`Security Directory Size`，这样就可以实现数字签名伪造。

[SigThief](https://github.com/secretsquirrel/SigThief)（https://github.com/secretsquirrel/SigThief）可自动实现数字签名伪造的过程：


```javascript
python sigthief.py -i ~/Desktop/consent.exe -t ~/Desktop/mimikatz_trunk/x64/mimikatz.exe -o test.exe
```

![](/img/hack/PE文件伪造/1508947004070.png)

查看伪造签名生成的test.exe数字签名，显示证书无效：

![](/img/hack/PE文件伪造/1508947094280.png)

使用`signtool.exe verify /v test.exe`

![](/img/hack/PE文件伪造/1508979948085.png)

Signtool显示`Win VerifyTrust returned error:0x80096010`

使用`sigcheck.exe -q test.exe`验证没通过：

![](/img/hack/PE文件伪造/1508982520529.png)

查看Get-AuthenticodeSignature的帮助说明：

```css
Get-Help Get-AuthenticodeSignature -Full
```

查看相关操作Set-AuthenticodeSignature的帮助说明：

```css
Get-Help Set-AuthenticodeSignature -Full
```

发现该命令的功能：

```css
The Set-AuthenticodeSignature cmdlet adds an Authenticode signature to 
any file that supports Subject Interface Package (SIP).
```

关于SIP的资料，可参考：

https://blogs.technet.microsoft.com/eduardonavarro/2008/07/11/sips-subject-interface-package-and-authenticode/

获得有用的信息：

There are some included as part of the OS (at least on Vista). Locate 
in the %WINDIR%System32 directory. They usually have a naming ending 
with sip.dll, i.e. msisip.dll is the Microsoft Installer (.msi) SIP.

查找Windows下的SIP：

```css
dir C:\Windows\System32\*sip.dll -Recurse -ErrorAction SilentlyContinue
```

![](/img/hack/PE文件伪造/1508982745295.png)

使用IDA Pro打开msisip.dll，查看函数`DllRegeusterServer`，可以看到一个名为`MsiSIPVerifyIndirectData`

![](/img/hack/PE文件伪造/1508982478140.png)

找到对应的函数`CryptSIPVerifyIndi​​rectData`,微软官方说明在https://msdn.microsoft.com/en-us/library/windows/desktop/cc542591%28v=vs.85%29.aspx

该函数的返回值为True表示验证成功，放那会FALSE 代表验证失败，对应的注册表键值位置为：`HKLM\SOFTWARE\Microsoft\Cryptography\OID\Encoding\Type 0\CryptSIPDllVerifyIndirectData`

![](/img/hack/PE文件伪造/1508983621830.png)

不同的GUID对应不同的文件格式验证：

- C689AAB8-8E78-11D0-8C47-00C04FC295EE | PE
- DE351A43-8E59-11D0-8C47-00C04FC295EE | Catalog.cat文件
- 9BA61D3F-E73A-11D0-8CD2-00C04FC295EE | CTL.ctl文件
- C689AABA-8E78-11D0-8C47-00C04FC295EE | Cabinet.cab文件

接下来，尝试替换`HKLM\SOFTWARE\Microsoft\Cryptography\OID\EncodingType 0\CryptSIPDllVerifyIndirectData\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}`下的dll和FuncName

写的dll文件：

![](/img/hack/PE文件伪造/1509954540060.png)

更改注册表，写入dll路径和函数名，重启校验失败。后直接使用`C:\Windows\System32\ntdll.dll`作为路径，导出函数名为`DbgUiContinue`,重启之后，校验签名成功：


![](/img/hack/PE文件伪造/1509954340724.png)

对于64位系统，存在32位的注册表键值

如果使用32位的程序，如32位的signtool和sigcheck，为了绕过验证，还需要修改32位的注册表键值，对应代码如下：


```c
REG ADD "HKLM\SOFTWARE\Wow64\32\Node\Microsoft\Cryptography\OID\Encoding\Type 0\CryptSIPDllVerifyIndirectData\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}" /v "Dll" /t REG_SZ /d "C:\Windows\System32\ntdll.dll" /fREG ADD "HKLM\SOFTWARE\Wow64\32\Node\Microsoft\Cryptography\OID\Encoding\Type 0\CryptSIPDllVerifyIndirectData\{C689AAB8-8E78-11D0-8C47-00C04FC295EE}" /v "FuncName" /t REG_SZ /d "DbgUiContinue" /f
```

参考三好师傅： http://www.4hou.com/system/7937.html


