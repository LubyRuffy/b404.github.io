---
title: 'Web之困笔记'
layout: post
tags: 
  - web
  - protocole
category: reading-notes
comments: true
share: true
---
Web之困笔记

<!--more-->

<li>
<h3>url结构</h3>
<img src="/img/web之困/url结构.jpg">
</li>
<li>
	<h3>url非层级协议——mailto</h3>
	<img src="/img/web之困/mailto.jpg">
</li>
<li>
	<strong>浏览器解析方式的大致算法：</strong>
		<p align="left">1.提取协议名称：扫描第一个“：”字符。该字符左边的url部分就是协议名称</p>
		<p align="left">2.去层级url标记符：字符串“//”应该跟在协议后面，如果未找到就不用管</p>
		<p align="left">3.获取授权信息部分：依次扫描“/”“?”或者“#”符号，哪个先出现以哪个为准进行截取，从url里提取出来的部分就是授权部分信息。定位登陆信息，如果有的话，在截取的信息中寻找@符号，如果找到第一个冒号前面的就是用户名，后面是密码数据；提取目标地址，授权信息部分剩下的就是目标地址了，第一个冒号分隔开的就是主机名和端口号。用方括号括起来的就是IPV6</p>
		<p align="left">4.确定路径（如果的确存在）</p>
		<p align="left">5.提取查询字符串。在上一条解析后跟一个问号</p>
		<p align="left">6.提取片段ID。在解析完上一条之后跟就“#”</p>
	<img src="/img/web之困/10.0.0.1.jpg">
	<p>
		http://example.com\@coredump.cx/
		在firefox里，这个url会把用户带往coredump.cx，因为example.com\会被认为是一个合法的登陆信息。而其他浏览器里，“\”会被认为一个路径分隔符，所以用户最终访问example.com
	</p>
	<img src="/img/web之困/example.jpg">
</li>
<li>
<p><h3>服务器响应码：</h3></p>
    <table border="2">
    <tr>
    	<th align="left">200~299,成功</th>
    	<th align="right">300~399,重定义和其他状态信息</th>
    	<th align="right">400~499,客户端错误</th>
    	<th aligh="right">500~599,服务器端错误</th>
    </tr>
	<tr>
		<td>200，ok</td>
		<td>301，永久移动</td>
		<td>400，(bad request)不符合规范的请求</td>
	    <td>500，internal server error(内部服务器错误)</td>
	</tr>
	<tr>
		<td>204,not content</td>
		<td>302，找到</td>
		<td>401,(Unauthorized)未授权</td>
		<td>503，service unavailable(服务不可用)</td>
	</tr>
	<tr>
	    <td>206,部分内容</td>
	    <td>303，参见其他</td>
	    <td>403，forbidden(禁止访问)</td>
	    <td></td>
	</tr>
	<tr>
	    <td></td>
	    <td>304,无变化</td>
	    <td>404，not found(文件找不到)</td>
	    <td></td>
	</tr>
	<tr>
		<td></td>
		<td>307，临时重定向</td>
		<td></td>
		<td></td>
	</tr>
   </table>
</li>
<li>
	<h3>HTTP协议</h3>
	<img src="/img/web之困/Http协议.png">
</li>
<small>待续</small>