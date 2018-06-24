---
title: '巡风扫描器'
layout: post
tags: 
  - scan
  - hack
  - information
category: 
  - hack
  - sec
  - scan
comments: true
share: true
description: 巡风扫描器的安装、配置、使用
---

巡风扫描器的安装、配置、使用

<!--more-->

* TOC
{:toc}


## 安装环境

1. 系统：Ubuntu16.10
2. 软件:Python2.7、Mongodbv3.2.12
3. 相关依赖：
```
➜ /home/b404 >sudo apt install gcc libffi-devel python-devel openssl-devel libpcap-devel
```
4. Pythpon依赖库

```c
# 需先安装pip，建议使用豆瓣的pip源，否则可能会因为超时导致出错。
wget https://sec.ly.com/mirror/get-pip.py --no-check-certificate
python get-pip.py

# 已经有pip需更新到最新版本
pip install -U pip

pip install pymongo Flask xlwt paramiko
```

## Mongodb数据库

0x1.下载数据库，并安装

```c
➜ /home/b404 >sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv EA312927
➜ /home/b404 >echo "deb http://repo.mongodb.org/apt/ubuntu xenial/mongodb-org/3.2 multiverse" | sudo tee /etc/apt/sources.list.d/mongodb-org-3.2.list
➜ /home/b404 >sudo apt-get update
➜ sudo apt-get install -y mongodb-org
```

在安装成功后，启动服务端，测试MongoDB安装情况，报错
![MongoDB报错](/img/xunfeng/1486127676522.png)
后根据分析，在根目录下创建`/data/db/`文件夹，连接test数据库成功

```c
➜ / >sudo mkdir data     
➜ / >cd data 
➜ /data >sudo mkdir db
➜ /data >sudo mongod   
```
![Alt text](/img/xunfeng/1486128106371.png)

0x2.创建数据库
打开MongoDB客户端：

```c
➜ /var/log/mongodb >mongo
MongoDB shell version: 3.2.12
connecting to: test
Welcome to the MongoDB shell.
For interactive help, type "help".
For more comprehensive documentation, see
	http://docs.mongodb.org/
Questions? Try the support group
	http://groups.google.com/group/mongodb-user
Server has startup warnings: 
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] ** WARNING: You are running this process as the root user, which is not recommended.
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] 
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] 
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] ** WARNING: /sys/kernel/mm/transparent_hugepage/enabled is 'always'.
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] **        We suggest setting it to 'never'
2017-02-03T17:29:20.731+0800 I CONTROL  [initandlisten] 
```

测试MongoDB数据库的shell情况:


```c
> 2+2
4
```

创建数据库：

```c
> use xunfeng
switched to db xunfeng
> show dbs
local  0.000GB
> db
xunfeng
> db.xunfeng.insert({"scan":"scanlol66"})
WriteResult({ "nInserted" : 1 })
> show dbs
local    0.000GB
xunfeng  0.000GB
```

> 创建数据库成功之后，并不能在使用`show dbs`后查看到新创建的数据库，得添加数据进去才能看见

0x3.创建用户名、密码

```Shell
> db.createUser({user:'scan',pwd:'scanlol66',roles:[{role:'dbOwner',db:'xunfeng'}]})
Successfully added user: {
	"user" : "scan",
	"roles" : [
		{
			"role" : "dbOwner",
			"db" : "xunfeng"
		}
	]
}
```

## 配置巡风

0x1.下载巡风系统

```c
➜ /home/b404 >git clone https://github.com/ysrc/xunfeng.git
正克隆到 'xunfeng'...
remote: Counting objects: 1007, done.
remote: Total 1007 (delta 0), reused 0 (delta 0), pack-reused 1007
接收对象中: 100% (1007/1007), 2.87 MiB | 17.00 KiB/s, 完成.
处理 delta 中: 100% (405/405), 完成.
检查连接... 完成。
➜ /home/b404 >sudo mv xunfeng /usr/local/
```
统一路径：![路径](/img/xunfeng/1486130400185.png)

0x2.更改配置

```c
dbpath=/usr/local/mongodb/db
 
# 设置日志文件的存放目录及其日志文件名
 
logpath=/usr/local/mongodb/logs/mongodb.log
 
# 设置端口号
 
port=65521
 
# 设置为以守护进程的方式运行，即在后台运行
 
fork=true
 
nohttpinterface=true
```

0x3.连接巡风数据库

```c
➜ /usr/local/xunfeng git:(master) ✗ >mongorestore -h 127.0.0.1 -d --port 66571 xunfeng db

2017-02-03T18:01:58.923+0800	building a list of collections to restore from db dir

2017-02-03T18:01:58.925+0800	reading metadata for xunfeng.Config from db/Config.metadata.json

2017-02-03T18:01:58.925+0800	reading metadata for xunfeng.Info from db/Info.metadata.json

2017-02-03T18:01:58.925+0800	reading metadata for xunfeng.Heartbeat from db/Heartbeat.metadata.json

2017-02-03T18:01:58.926+0800	reading meta...
```

0x4.运行巡风

```c
➜ /usr/local/xunfeng git:(master) ✗ >sudo bash ./Run.sh
```

## 使用巡风

登陆巡风

默认用户名和密码分别是`admin`和`xunfeng321`。根据配置文件设置的登陆：
![登陆巡风](/img/xunfeng/1486130735504.png)
![巡风界面](/img/xunfeng/1486130852858.png)
