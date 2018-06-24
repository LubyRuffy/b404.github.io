---
title: 'ubuntu16.04 install docker'
layout: post
tags: 
  - reading-notes
  - tools
category: 
  - tools
  - environment
comments: true
share: true
description: ubuntu16.04 install docker
---
ubuntu16.04 install docker

* TOC
{:toc}

<!--more-->

>Ubuntu宿主机安装的条件如下：
>> 1.内核

```css
➜ /home/b404 >uname -a
Linux admin 4.4.0-38-generic #57-Ubuntu SMP Tue Sep 6 15:42:33 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
```

>>* 64位的Ubuntu内核需要3.10及更高的版本 

>>2.检查Device Mapper
>>>Device Mapper作为存储器驱动。2.69版本的的Linux内核开始集成Device Mapper，并且提供一个将块设备映射到高级虚拟器的方法。Device Mapper支持“自动精简配置”。Device Mapper作为Docker的存储驱动再合适不过。

>>可通过以下代码检测是否安装：

```css
➜ /home/b404 >ls -l /sys/class/misc/device-mapper
lrwxrwxrwx 1 root root 0 9月  28 18:50 /sys/class/misc/device-mapper -> ../../devices/virtual/misc/device-mapper  #检查Device Mapper
➜ /home/b404 >sudo grep device-mapper .proc/devices
[sudo] b404 的密码： 
grep: .proc/devices: 没有那个文件或目  #在Ubuntu的proc中检查Device Mapper
➜ /home/b404 >sudo modprobe dm_mod    #加载Device Mapper模块
➜ /home/b404 >

```

--------------------------------------

# 安装Docker：

@[Docker, Ubuntu]

1. 更新软件包数据:

```Shell
➜ /home/b404 >sudo apt-get update
```

2. 把离线的Docker仓库GPG密钥添加到系统中:

 ```Shell
 ➜ /home/b404 >sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
Executing: /tmp/tmp.qh3kVjqff2/gpg.1.sh --keyserver
hkp://p80.pool.sks-keyservers.net:80
--recv-keys
58118E89F3A912897C070ADBF76221572C52609D
gpg: 下载密钥‘2C52609D’，从 hkp 服务器 p80.pool.sks-keyservers.net
gpg: 密钥 2C52609D：公钥“Docker Release Tool (releasedocker) <docker@docker.com>”已导入
gpg: 合计被处理的数量：1
gpg:               已导入：1  (RSA: 1)
 ```
3. 添加Docker 仓库到APT源：

```css
➜ /home/b404 >echo "deb https://apt.dockerproject.org/repo ubuntu-xenial main" | sudo tee /etc/apt/sources.list.d/docker.list
deb https://apt.dockerproject.org/repo ubuntu-xenial main
```

4. 将具有Docker安装包的软件源列表更新：

```Shell
➜ /home/b404 >sudo apt-get update

```

5. 确定软件源中添加了Docker：

```Shell
➜ /home/b404 >apt-cache policy docker-engine
docker-engine:
  已安装：(无)
  候选： 1.12.1-0~xenial
  版本列表：
     1.12.1-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.12.0-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.11.2-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.11.1-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
     1.11.0-0~xenial 500
        500 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 Packages
```

6. 最后安装

```Shell
➜ /home/b404 >sudo apt-get install -y docker-engine 
正在读取软件包列表... 完成
正在分析软件包的依赖关系树       
正在读取状态信息... 完成       
将会同时安装下列软件：
  aufs-tools cgroupfs-mount
下列【新】软件包将被安装：
  aufs-tools cgroupfs-mount docker-engine
升级了 0 个软件包，新安装了 3 个软件包，要卸载 0 个软件包，有 10 个软件包未被升级。
需要下载 19.6 MB 的归档。
解压缩后会消耗 102 MB 的额外空间。
获取:1 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 aufs-tools amd64 1:3.2+20130722-1.1ubuntu1 [92.9 kB]
获取:2 http://cn.archive.ubuntu.com/ubuntu xenial/universe amd64 cgroupfs-mount all 1.2 [4,970 B]
获取:3 https://apt.dockerproject.org/repo ubuntu-xenial/main amd64 docker-engine amd64 1.12.1-0~xenial [19.5 MB]
```

7. 检查Docker是否运行：

```Shell
➜ /home/b404 >sudo systemctl status docker
[sudo] b404 的密码： 
● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: e
   Active: active (running) since 三 2016-09-28 22:14:00 CST; 1h 51min ago
     Docs: https://docs.docker.com
 Main PID: 11177 (dockerd)
    Tasks: 20
   Memory: 17.6M
      CPU: 3.620s
   CGroup: /system.slice/docker.service
           ├─11177 /usr/bin/dockerd -H fd://
           └─11184 docker-containerd -l unix:///var/run/docker/libcontainerd/doc
9月 28 22:14:00 admin dockerd[11177]: time="2016-09-28T22:14:00.155743979+08:00" 
9月 28 22:14:00 admin dockerd[11177]: time="2016-09-28T22:14:00.156382665+08:00" 
...
```

>默认docker运行需要root权限，在执行命令时候必须加上sudo。如果想在使用docker命令时候避免使用sudo，可以将用户名加入docker组中：
>> sudo usermod -aG docker $(whoami)
>> sudo username -aG docker username 
>>> *username*是系统用户名；添加之后可以直接运行docker

--------------------------------

# Docker的使用

* 可以运行docker命令查看所有指令:

```Shell
➜ /home/b404 >docker
Usage: docker [OPTIONS] COMMAND [arg...]
       docker [ --help | -v | --version ]

A self-sufficient runtime for containers.

Options:

  --config=~/.docker              Location of client config files
  -D, --debug                     Enable debug mode
  -H, --host=[]                   Daemon socket(s) to connect to
  -h, --help                      Print usage
  -l, --log-level=info            Set the logging level
  --tls                           Use TLS; implied by --tlsverify
  --tlscacert=~/.docker/ca.pem    Trust certs signed only by this CA
  --tlscert=~/.docker/cert.pem    Path to TLS certificate file
  --tlskey=~/.docker/key.pem      Path to TLS key file
  --tlsverify                     Use TLS and verify the remote
  -v, --version                   Print version information and quit

Commands:
    attach    Attach to a running container
    build     Build an image from a Dockerfile
    commit    Create a new image from a container's changes
    cp        Copy files/folders between a container and the local filesystem
    create    Create a new container
    diff      Inspect changes on a container's filesystem
    events    Get real time events from the server
    exec      Run a command in a running container
    export    Export a container's filesystem as a tar archive
    history   Show the history of an image
    images    List images
    import    Import the contents from a tarball to create a filesystem image
    info      Display system-wide information
    inspect   Return low-level information on a container, image or task
    kill      Kill one or more running containers
    load      Load an image from a tar archive or STDIN
    login     Log in to a Docker registry.
    logout    Log out from a Docker registry.
    logs      Fetch the logs of a container
    network   Manage Docker networks
    node      Manage Docker Swarm nodes
    pause     Pause all processes within one or more containers
    port      List port mappings or a specific mapping for the container
    ps        List containers
    pull      Pull an image or a repository from a registry
    push      Push an image or a repository to a registry
    rename    Rename a container
    restart   Restart a container
    rm        Remove one or more containers
    rmi       Remove one or more images
    run       Run a command in a new container
    save      Save one or more images to a tar archive (streamed to STDOUT by default)
    search    Search the Docker Hub for images
    service   Manage Docker services
    start     Start one or more stopped containers
    stats     Display a live stream of container(s) resource usage statistics
    stop      Stop one or more running containers
    swarm     Manage Docker Swarm
    tag       Tag an image into a repository
    top       Display the running processes of a container
    unpause   Unpause all processes within one or more containers
    update    Update configuration of one or more containers
    version   Show the Docker version information
    volume    Manage Docker volumes
    wait      Block until a container stops, then print its exit code

Run 'docker COMMAND --help' for more information on a command.
```Shell
* 查询每个命令的用法:

```
docker docker-subcommand --help
```

```Shell
➜ /home/b404 >docker search --help

Usage:  docker search [OPTIONS] TERM

Search the Docker Hub for images

Options:
  -f, --filter value   Filter output based on conditions provided (default [])
      --help           Print usage
      --limit int      Max number of search results (default 25)
      --no-trunc       Don't truncate output
```

* 查看当前系统使用Docker的情况：

```Shell
 ➜ /home/b404 >sudo docker info
[sudo] b404 的密码： 
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 1.12.1
Storage Driver: aufs
 Root Dir: /var/lib/docker/aufs
 Backing Filesystem: extfs
 Dirs: 0
 Dirperm1 Supported: true
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: host bridge null overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Security Options: apparmor seccomp
Kernel Version: 4.4.0-38-generic
```

* 检查是否连接并从Docker Hub中下载了镜像:

```py
➜ /home/b404 >sudo docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker Hub account:
 https://hub.docker.com

For more examples and ideas, visit:
 https://docs.docker.com/engine/userguide/
```
* 从Docker Hub查找docker镜像：

```Shell
➜ /home/b404 >sudo docker search ubuntu
NAME                              DESCRIPTION                                     STARS     OFFICIAL   AUTOMATED
ubuntu                            Ubuntu is a Debian-based Linux operating s...   4789      [OK]       
ubuntu-upstart                    Upstart is an event-based replacement for ...   67        [OK]       
rastasheep/ubuntu-sshd            Dockerized SSH service, built on top of of...   42                   [OK]
ubuntu-debootstrap                debootstrap --variant=minbase --components...   27        [OK]       
torusware/speedus-ubuntu          Always updated official Ubuntu docker imag...   27                   [OK]
nickistre/ubuntu-lamp             LAMP server on Ubuntu                           9                    [OK]
...
```
* 通过pull命令拉取镜像：

```
➜ /home/b404 >sudo docker pull ubuntu
Using default tag: latest
latest: Pulling from library/ubuntu
cad964aed91d: Pull complete 
3a80a22fea63: Pull complete 
50de990d7957: Pull complete 
61e032b8f2cb: Pull complete 
9f03ce1741bf: Pull complete 
Digest: sha256:28d4c5234db8d5a634d5e621c363d900f8f241240ee0a6a978784c978fe9c737
Status: Downloaded newer image for ubuntu:latest
```
*在镜像下载完之后，可通过run运行镜像：

```Shell
➜ /home/b404 >sudo docker run ubuntu
```

>查看镜像问题

```Shell
➜ /home/b404 >sudo docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
ubuntu              latest              c73a085dc378        2 days ago          127.1 MB
hello-world         latest              c54a2cc56cbb        12 weeks ago        1.848 kB
➜ /home/b404 >
```
* 通过-i和-t进入交互式shell中：

```python
➜ /home/b404 >sudo docker run -it ubuntu 
root@36672865d258:/# whoami
root
root@36672865d258:/# uname -a
Linux 36672865d258 4.4.0-38-generic #57-Ubuntu SMP Tue Sep 6 15:42:33 UTC 2016 x86_64 x86_64 x86_64 GNU/Linux
root@36672865d258:/# ls
bin   dev  home  lib64  mnt  proc  run   srv  tmp  var
boot  etc  lib   media  opt  root  sbin  sys  usr
root@36672865d258:/# cd home
root@36672865d258:/home# ls
root@36672865d258:/home# 
```
* 可以在容器里运行任何命令，不用使用sudo，已经具有root权限：

```
root@36672865d258:/home# apt-get update
Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
Get:2 http://archive.ubuntu.com/ubuntu xenial-updates InRelease [95.7 kB]
Get:3 http://archive.ubuntu.com/ubuntu xenial-security InRelease [94.5 kB]
Get:4 http://archive.ubuntu.com/ubuntu xenial/main Sources [1103 kB]
...
```
* 在容器里可以像正常主机一样安装任何东西:

```Shell
root@36672865d258:/home# apt-get install -y nodejs
Reading package lists... Done
Building dependency tree       
Reading state information... Done
The following additional packages will be installed:
  libicu55 libssl1.0.0 libuv1
The following NEW packages will be installed:
  libicu55 libssl1.0.0 libuv1 nodejs
0 upgraded, 4 newly installed, 0 to remove and 0 not upgraded.
Need to get 11.9 MB of archives.
...
```
--------------------------------------

## 提交容器的更改到Docker 镜像

>Docker文件系统是默认设置的。如果开启一个Docker镜像，可以像虚拟机一样创建、更改、删除。然而，如果关闭容器再次打开时候，所有的更改都会丢失。但是可以新Docker镜像来保存容器的状态。
* 把容器的状态保存为一个新的镜像，首先要从其退出:

```Shell
root@36672865d258:/home# exit
exit
```

* 提交更改到一个新的镜像，可以使用以下命令：

```Shell
docker commit -m "What did you do to the image" -a "Author Name" container-id repository/new_image_name
```
>> 
>> -m参数是备注提交的内容，-a是标明作者，container-id是开启docker使用时候的。除非增添了仓库在Docker Hub，否则仓库名字一般是自己的Docker Hub名字


```Shell
➜ /home/b404 >sudo docker commit -m "added node.js" -a "b404" 36672865d258 test/ubuntu-nodejs
[sudo] b404 的密码： 
sha256:8f8bc42c9d82d7346bd09ffb1533dd0ac58adab2b72bf45d76f102e9b1830973
```
* 当使用commit一个镜像之后，新的镜像就存储在本地。这时候通过docker images来查看：

```Shell
➜ /home/b404 >sudo docker images
REPOSITORY           TAG                 IMAGE ID            CREATED             SIZE
test/ubuntu-nodejs   latest              8f8bc42c9d82        12 seconds ago      170.3 MB
ubuntu               latest              c73a085dc378        2 days ago          127.1 MB
hello-world          latest              c54a2cc56cbb        12 weeks ago        1.848 kB
```

>下次使用就可以接着使用上次commit的镜像

------------------------------------------------

## 监听Docker容器情况：

* 查看正在运行的情况：

```Shell
➜ /home/b404 >sudo docker ps
[sudo] b404 的密码： 
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
119a9c92099c        ubuntu              "/bin/bash"         16 seconds ago      Up 15 seconds                           sad_mestorf
```
* 查看所有容器的情况：

```Shell
➜ /home/b404 >sudo docker ps -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                         PORTS               NAMES
119a9c92099c        ubuntu              "/bin/bash"         36 seconds ago      Up 35 seconds                                      sad_mestorf
36672865d258        ubuntu              "/bin/bash"         33 minutes ago      Exited (100) 15 minutes ago                        serene_minsky
c82569274e39        ubuntu              "/bin/bash"         40 minutes ago      Exited (0) 40 minutes ago                          kickass_swartz
29baff70425e        hello-world         "/hello"            About an hour ago   Exited (0) About an hour ago                       serene_allen
a241c03cf7e4        hello-world         "/hello"            About an hour ago   Exited (0) About an hour ago                       clever_darwin
```
* 查看最新创建的容器：

```Shell
➜ /home/b404 >sudo docker ps -l
CONTAINER ID        IMAGE               COMMAND             CREATED              STATUS              PORTS               NAMES
119a9c92099c        ubuntu              "/bin/bash"         About a minute ago   Up About a minute                       sad_mestorf
```
* 销毁指定的容器：

```Shell
➜ /home/b404 >sudo docker stop 119a9c92099c 
119a9c92099c
```
------------------------

## push Docker镜像到Docker仓库

>登录Docker Hub，push一个镜像到Docker到一个Docker Hub或者是一个Docker仓库

* 登录Docker Hub:

```Shell
docker login -u docker-registry-username
```

* 登录成功之后，push自己的镜像：

```
docker push docker-registry-username/docker-image-name
```

* 在push时候，会输出类似于下面的字符串:

```Shell
The push refers to a repository [docker.io/test/ubuntu-nodejs]
e3fbbfb44187: Pushed
5f70bf18a086: Pushed
a3b5c80a4eba: Pushed
7f18b442972b: Pushed
3ce512daaf78: Pushed
7aae4540b42d: Pushed
....
```
* 在push完成之后，可以在docker仓库中查看自己的镜像