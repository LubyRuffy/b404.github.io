---
title: 弹出计算器
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

如题

<!--more-->

* TOC
{:toc}

## pcalua

利用pcalua弹出计算器，因此该技巧可以用于....


![refer](/img/skills/pcalua/refer.png)


在cmd中输入`pcalua.exe -a calc.exe`，弹出计算器：


![calc](/img/skills/pcalua/calc.png)


将  `pcalua.exe -a calc.exe` 写入到自启动的注册表中，开机自启动的时候，计算器会自动弹出，但用Autorun.exe工具查看注册表的隐藏项目，并不能看到


![windows_entryies_hidden](/img/skills/pcalua/windows_entries_hidden.png)



refer:https://twitter.com/0rbz_/status/912491288104140801



## POC2


```css
cmd.exe /c ^C^:\/\/\/\/\^/\/\/\/\/\/^/////\////////^///\\\\^\\\^\\\\windows\/\\/\^/\/\/\/\system32\/\/\//^\//\/\/\/\/\/^calc.exe
```


![](/img/skills/calc/%E9%80%89%E5%8C%BA_234.png)




