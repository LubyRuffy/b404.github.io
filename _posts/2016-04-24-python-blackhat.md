---
title: 'python blackhat'
layout: post
tags: 
  - python 
  - socket
  - hack
  - code
category: 
  - python 
  - sec 
  - code
comments: true
share: true
---
python blackhat reading notes

<!--more-->

<p align="left">tcp client:</p>
<pre>
     <code>
__author__ = 'root'
#-*-coding:utf-8-*-
import socket

target_host = "www.baidu.com"
target_port = 80
client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((target_host,target_port))
client.send("GET / HTTP/1.1\r\nHost: baidu.com\r\n\r\n")
response = client.recv(4096)
print response
     </code>
</pre>
<img src="/img/blackhat/tcpclient.png">