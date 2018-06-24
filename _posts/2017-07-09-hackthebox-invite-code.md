---
title: hackthebox的邀请码
layout: post
tags:
  - sec
  - tools
  - game
category: 
  - hack
  - game
comments: true
share: true
---

注册hackthebox，发现邀请码有点蒙蔽，后面尝试通过解密得到邀请码。

* TOC
{:toc}

<!--more-->



1. 进入[邀请码页面](https://www.hackthebox.eu/en/invite)：https://www.hackthebox.eu/en/invite。F12打开控制台，输入`makeInviteCode()`，出现提示的url。点击查看响应头，发现可以解密

![查找生成码](/img/game/hackthebox/makeInviteCode.png)

2. rot13解密得到：`In order to generate the invite code, make a POST request to /api/invite/generate`。

3. 使用burpsuit代理，修改GET方式为POST方式，提交得到邀请码：

![修改提交方式得到邀请码](/img/game/hackthebox/generate_invite_code.png)

4. 邀请码为base64加密，解密之后即可在邀请码页面输入，提交注册，进入主页

![HackTheBox主页](/img/game/hackthebox/hackthebox.png)
