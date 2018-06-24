---
title: '踩点'
layout: post
tags: 
  - sec
  - hack
  - information
category: sec
comments: true
share: true
---
获得信息

<!--more-->

<h2>一.踩点：</h2>
<h3>公开渠道获得信息：</h3>


> 确定相关人员信息、IP地址范围、DNS服务器信息、邮件服务器信息


<li><strong><big>1.探测web页面：</big></strong></li>
<p align="left">dirbuster,列举隐藏目录：<img src="/img/hack/dirbuster.jpg"></p>
<p align="left">httrack,镜像网站:<img src="/img/hack/httrack.jpg"></p>
尝试通过登陆该网站的vpn和ftp。如http://vpn.dvssc.com,http://www.dvss.com/vpn
<li><strong><big>2.相关组织：</big></strong></li>
<p>通过相关的页面查看与其有关的组织或者公司。可以从页面、源码等来分析
</p>
<li><strong><big>3.地理位置：</big></strong></li>
<p align="left">可以通过google的earth和map来查找。Google Locations和skyhook基于mac地址进行信息的查找服务。通过mac地址在<a href="https://www.shodan.io/explore">https://www.shodan.io/explore</a> 查找信息看，通过google搜索改mac的一切信息</p>
<li><strong><big>4.相关人员信息：</big></strong></li>
<p align="left">值得注意的是很多人将姓名或者其变体作为用户名和电子邮件地址。</p>
<p align="left">查询号码、邮编：<a href="www.411.com">www.411.com</a>， <a href="http://www.whitepages.com/">http://www.whitepages.com/</a>，   <a href="http://www.yellowpages.com/">http://www.yellowpages.com/</a>查询号码所在地的地址。 
</p>
<p align="left">
	查询个人细节资料：http://www.blackbookonline.info/  ，http://www.peoplesearch.com/ 
</p>
<p align="left">查询辅助网站：<br/>社交：Facebook、Myspace、Reunion、Classmates<br/>专业社交：Linkdin、Plaxo<br/>职业规划：Monster、Careerbuilder、Dice<br/>家族：Ancestry<br/>照片管理:Flicker、Photobucket<br/>电话号码:JigSaw</p>
<li><strong><big>5.近期重大事件：</big></strong></li>
<p align="left">合并、收购、丑闻、裁员、重组等其他事件，容易出现管理员的疏忽。
<small>证监会(www.sec.gov)的EDGAR中检索；一些商务信息网站和证券交易网站(Yahoo Finance)</small>
</p>
<li><strong><big>6.隐私和安全策略，现有信息安全机制的技术细节：</big></strong></li>
<p align="left">已归档的信息：<a href="https://archive.org/"> archive.org</a>,谷歌缓存，百度快照</p>
<li><strong><big>7.搜索引擎和数据关系：</big></strong></li>
<p align="left">谷歌、必应、雅虎、Yandex、<a href="http://www.wolframalpha.com/">wolframalpha</a>、<a href="http://www.dogpile.com/"> dogpile</a>、GHDB</p>
<p align="left">利用GHDB的自动化工具：</p>
<ul>
	<li>Athena2.0</li>
	<li>SiteDigger2.0</li>
	<li>Wikto2.0</li>
</ul>
<p align="left">分析隐藏文档中的元数据：</p>

- <a href="http://www.informatica64.com/foca.aspx">Foca</a>:识别常见文档扩展名，将其分类，并能识别高级漏洞
- Shodan（或者zoomeye）：搜索存在不安全漏洞的因特网系统和设备
- Usenet论坛和新闻组也是收集敏感信息，可搜素到密码
- Maltego：数据挖掘、关联对象

<h3>WHOIS、DNS查点：</h3>
<li><strong><big>1.WHOIS查询：</big></strong></li>
<p align="left">whois查询的时候要除去协议名称(HTTP、FTP *)</p>
<small ><a href="http://www.netcraft.com/">netcraft</a></small>
<li><strong><big>2.DNS查询：</big></strong></li>
<p align="left">DNS区域传送漏洞。允许不受信任的因特网用户执行DNS区域传送</p>

*                                能收集到目标信息
* 能作为攻击那些反通过DNS区域传送才暴露的目标跳板

> 区域传送：允许第三主服务器的数据刷新自己的区域数据库，防止主域名服务器因意外故障变得不可用时影响到这个网络。
> 区域传送配置错误：只要有人发出请求，就会提供一个区域数据库的拷贝，把内部主机名和ip地址暴露出来


<p align="left">nslookup：</p>

> 得到DNS解析服务器保存在Cache中的非权威解答

<pre><code>
root@kali:~# nslookup
> set type=any
> baidu.com
Server:		10.10.10.2
Address:	10.10.10.2#53

Non-authoritative answer:
baidu.com	text = "v=spf1 include:spf1.baidu.com include:spf2.baidu.com include:spf3.baidu.com a mx ptr -all"
baidu.com	text = "google-site-verification=GHb98-6msqyx_qqjGl5eRatD3QTHyVB6-xQ3gJB5UwM"
baidu.com	nameserver = ns3.baidu.com.
baidu.com	nameserver = ns7.baidu.com.
baidu.com	nameserver = ns4.baidu.com.
baidu.com	nameserver = ns2.baidu.com.
baidu.com	nameserver = dns.baidu.com.
Name:	baidu.com
Address: 111.13.101.208
Name:	baidu.com
Address: 123.125.114.144
------------------------------
root@kali:~# nslookup b404.xyz
Server:		10.10.10.2
Address:	10.10.10.2#53

Non-authoritative answer:
Name:	b404.xyz
Address: 192.30.252.153
Name:	b404.xyz
Address: 192.30.252.154

</code></pre>
<p align="left">
	dig,可以从该域名的官方DNS服务器上查询到精确的权威解答
</p>


+ dig用法：
 * dig 域名(查询A记录，如dig sina.com)
 * dig 域名 ns（查询ns记录，如 dig sina.com ns）
 * dig 域名 soa （查询soa记录，如dig sina.com soa）
 * dig @Server sina.com.cn(Server服务器上查询sina.com.cn的记录，比如dig @210.51.191.22 sina.com.cn)
 * dig 域名 +nssearch （查询一个域的授权dns服务器，如dig sina.com.cn +nssearch）
 * dig 域名 +trace(从根服务器开始最总一个域名的解析过程)


<p align="left">区域传送最佳工具：dnsrecon</p>

> 查处站点的子域名和IP信息
> 格式：dnsrecon.py [参数]



+ -d是指定域名
+ -lifetime是指定相应时间30s
+ -x –axfr: AXFR请求枚举
+ -s –dospf: 反向查询SPF记录 * -g –google: 通过google枚举子域名与IP * -w –dowhois: 查whois


<small>查询站点子域名和ip信息:</small>
<pre><code>
root@kali:~/Desktop/tool/dnsrecon-master# ./dnsrecon.py -d baidu.com --lifetime 30
[*] Performing General Enumeration of Domain: baidu.com
[-] DNSSEC is not configured for baidu.com
[*] 	 SOA dns.baidu.com 202.108.22.220
[*] 	 NS ns3.baidu.com 220.181.37.10
[*] 	 Bind Version for 220.181.37.10 baidu dns
[*] 	 NS ns7.baidu.com 119.75.219.82
[*] 	 Bind Version for 119.75.219.82 baidu dns
[*] 	 NS ns4.baidu.com 220.181.38.10
[*] 	 Bind Version for 220.181.38.10 baidu dns
[*] 	 NS ns2.baidu.com 61.135.165.235
[*] 	 Bind Version for 61.135.165.235 baidu dns
[*] 	 NS dns.baidu.com 202.108.22.220
[*] 	 Bind Version for 202.108.22.220 baidu dns
[*] 	 MX mx1.baidu.com 61.135.163.61

</code></pre>
<small>查询axfr记录：</small>
<pre><code>
root@kali:~/Desktop/tool/dnsrecon-master# ./dnsrecon.py -d baidu.com --lifetime 30 --axfr
[*] Performing General Enumeration of Domain: baidu.com
[*] Checking for Zone Transfer for baidu.com name servers
[*] Resolving SOA Record
[+] 	 SOA dns.baidu.com 202.108.22.220
[*] Resolving NS Records
[*] NS Servers found:
[*] 	NS ns2.baidu.com 61.135.165.235
[*] 	NS ns4.baidu.com 220.181.38.10
[*] 	NS ns3.baidu.com 220.181.37.10
[*] 	NS ns7.baidu.com 119.75.219.82
[*] 	NS dns.baidu.com 202.108.22.220
[*] Removing any duplicate NS server IP Addresses...
[*]  
[*] Trying NS server 61.135.165.235
[+] 61.135.165.235 Has port 53 TCP Open
[-] Zone Transfer Failed!
[-] No answer or RRset not for qname
[*]  
[*] Trying NS server 220.181.38.10
[+] 220.181.38.10 Has port 53 TCP Open
[-] Zone Transfer Failed!
[-] No answer or RRset not for qname
[*]  
[*] Trying NS server 119.75.219.82
[-] Zone Transfer Failed for 119.75.219.82!
[-] Port 53 TCP is being filtered
</code></pre>
<small>查whois：</small>
<pre><code>
root@kali:~/Desktop/tool/dnsrecon-master# ./dnsrecon.py -d baidu.com -w
[*] Performing General Enumeration of Domain: baidu.com
[-] DNSSEC is not configured for baidu.com
[*] 	 SOA dns.baidu.com 202.108.22.220
[*] 	 NS ns2.baidu.com 61.135.165.235
[*] 	 Bind Version for 61.135.165.235 baidu dns
[*] 	 NS ns3.baidu.com 220.181.37.10
[*] 	 Bind Version for 220.181.37.10 baidu dns
[*] 	 NS dns.baidu.com 202.108.22.220
[*] 	 Bind Version for 202.108.22.220 baidu dns
[*] 	 NS ns7.baidu.com 119.75.219.82
[*] 	 Bind Version for 119.75.219.82 baidu dns
[*] 	 NS ns4.baidu.com 220.181.38.10
[*] 	 Bind Version for 220.181.38.10 baidu dns
</code></pre>
<p align="left">确定MX（邮件交换）记录：</p>


> 通过定位电子邮件处理IP，确定目标组织的防火墙(host命令)

<li><strong><big>3.网络侦查：</big></strong></li>
<p align="left">确定网络拓扑结构和网路访问路径</p>
<p align="left">traceroute(windows是tracert)：查看一个IP数据包的传递路径</p>
<pre><code>
traceroute -S -p53 10.10.10.2
//大大多数网络愿意允许外来DNS查询内部网络，指定53端口（DNS查询端口）探测网络情况
</code></pre>
<p align="left">任何一种IP数据包都可以用来追踪路由</p>
<p align="left">Cain和Abel就是利用TCP数据包探测</p>
<p align="left">firewalk探测ACL(访问控制表)</p>
