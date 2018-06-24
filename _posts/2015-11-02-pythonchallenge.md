---
title: 'python challenge'
layout: post
tags: 
  - pythonchallenge 
  - python
  - code
  - game
comments: true
share: true
category: 
  - challenge
  - python
---

<!--more-->

pythonchallenge url:http://www.pythonchallenge.com

1. [第一关:](http://www.pythonchallenge.com/pc/def/0.html)直接猜238，38在2上面一点点，猜2<sup>38</sup>= 274877906944，在python解释器中输入2**38,进入下一关
2. [第二关:](http://www.pythonchallenge.com/pc/def/map.html)根据图片描述,猜想是字母替换.写个python脚本替
&nbsp;

```python
from string import maketrans
tab=maketrans('','')
tab1=tab[97:123]
tab2='cdefghijklmnopqrstuvwxyzab'
table=maketrans(tab1,tab2)
str="g fmnc wms bgblr rpylqjyrc gr zw fylb. rfyrq ufyr amknsrcpq ypc dmp. bmgle gr gl zw fylb gq glcddgagclr ylb rfyr'q ufw rfgq rcvr gq qm jmle. sqgle qrpgle.kyicrpylq() gq pcamkkclbcb. lmu ynnjw ml rfc spj."
str1="map"
print str.translate(table)
print str1.translate(table)
```

> 解出是map--&gt;ocr　进下一关
3. 第三关:
