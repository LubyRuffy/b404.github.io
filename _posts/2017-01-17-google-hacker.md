---
title: 'Google Hacker'
layout: post
tags: 
  - google-hacaker
  - information
  - hack
category: 
  - hack
  - sec
  - information
comments: true
share: true
---
Google Hacker搜索信息

<!--more-->

* TOC
{:toc}

## Google Hacker语法索引

```c
Intitle、Allintitle：在页面标题中搜索
Related:后面接一个url，显示相关站点
Inurl、Allinurl：在URL中查找文本（包含大量特殊字符，最为广泛使用）
Phonebook:搜索电话列表
Filetype：搜索制定类型的文件
Rphonebook：搜索住宅电话列表
intext、Allintext：在网页内容里查找字符串
Site：把搜索精确到特定的站点
Bphonebook：商业电话列表
Author：搜索Google中新闻组帖子的作者
Link：搜索与当前网页存在链接的网页（不能与其他操作符或搜索关键字混合使用）
Group：搜索Google标题
Inanchor：在链接文本中查找文本
Masgid：通过消息id来查找谷歌的帖子
Daterange：查找某个特定日期范围内发布的网页
Insubject：搜索Googlegroup的主题行
Cache：显示网页的缓存版本
Stocks：搜索股票信息
Info：显示Google的摘要信息
Define：显示某术语的定义
Numrang：后接数字范围。搜索数字需要两个参数一个最小数，一个最大数，用破折号隔开 Ext:搜索指定后缀的url

+ 加入被忽略的词
- 忽略某个词
~ 同意词
. 单一的通配符
/*/ 通配符，可代表多个字母（去掉星号前后的两个斜杠）
“ ” 精确查询

布尔操作：
   and  与
   or    或
   not  不
```

> 注意事项：
> (1) 操作符、冒号、关键字之间是没有空格的。 
> (2) 布尔操作符（AND、OR、NOT）和特殊字符（-、+）仍可用作高级操作符查询，但是不能把他们放在冒号之前而把冒号和操作符分开。
> (3) 高级操作符能够和单独的查询混合使用
> (4) ALL操作符（以ALL开头的操作符）非常古怪。一般情况下，一个查询中只能使用一次ALL操作符，而且不能和其他操作符混用。

--------------------------
## Google Hacker 常用语句使用方法

### 数据库报错
```c
site:xx.com
intext:"sql syntax near"
intext:"syntax error has occurred"
intext:"incorrect syntax near"
intext:"unexpected end of SQL command"
intext:"Warning: mysql_connect()"
intext:"Warning: mysql_query()"
intext:"Warning: pg_connect()"
```
### 找后台
```c
site:heimian.com intext:管理|后台|登陆|用户名|密码|验证码|系统|帐号|manage|admin|login|system
site:heimian.com inurl:login|admin|manage|manager|admin_login|login_admin|system
site:heimian.com intitle:管理|后台|登陆|管理员|登录
```

### 找敏感文件
```c
filetype:doc|docx|pdf|xls|txt|mdb|ppt
inurl:.doc|.docx|.pdf|.xls|.txt|.mdb|.ppt
intext:.doc|.docx|.pdf|.xls|.txt|.mdb|.ppt
```

### 找目录遍历漏洞
```c
intext:转到父目录
intext:index of
intitle:index of
intext:Parent Directory
```

### 查找端口

```bash
//查找8080端口页面
inurl:8080 -intext:8080

//端口查找网页
filetype:php inurl:nqt intext:"network query tool"
```

### 查找SQL数据库

```bash
filetype:sql + "IDENTIFIED BY" -cvs

filetype:sql + "IDENTIFIED BY" ("Grant * on *" | "create user")
```

## Refer

[Google Hacking for PenetrationTesters](https://b404.xyz/sources/Google.Hacking.Filters.pdf):https://b404.xyz/sources/Google.Hacking.Filters.pdf

