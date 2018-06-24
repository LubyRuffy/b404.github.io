---
title: 'Javascript-DOM'
layout: post
tags: 
  - javascript
  - dom
  - reading-notes
  - code
categories: 
  - reading-notes
  - javascript
comments: true
share: true
description: Javascript-DOM编程艺术笔记
---
Javascript-DOM编程艺术笔记
<!--more-->

* TOC
{:toc}



## Javascript DOM:

### 显示缩略语表:
使用DOM来创建定义列表：
1. 遍历这份文档中的所有abbr元素
2. 保存每个abbr元素的title属性
3. 保存每个abbr元素包含的文本
4. 创建一个“定义列表”元素（即dl元素）
5. 遍历刚才保存的title属性和abbr元素的文本。
6. 创建一个“定义列表”元素（即dt元素）
7. 把abbr元素的文本插入到这个dt元素。
8. 创建一个“定义描述”的元素（即dd元素）。
9. 把title属性插入到这个dd元素。
10. 把dt元素追加到第四步创建的dl元素上。
11. 把dd元素追加到第四步创建的dl元素上。
12. 把dl元素追加到explanation.html创建的body元素上。

* 引用body的方法：
 - 使用 DOM core，即引用某给定文档的第一个body标签：

```javascript
  document.getElementsByTagName("body")[0];
```
  
 - 使用HTML-DOM，即引用某给定文档的body属性：
  document.body

```Javascript
  1.  首先插入“缩略语表”的标题：
document.body.appendChild(header);
  2.  然后插入“缩略语表”本身：
document.body.appendChild(dlist);
```

```javascript
 function displayAbbreviations() {
  if (!document.getElementsByTagName) return false;
  if (!document.createElement) return false;
  if (!document.createTextNode) return false;
  //取得缩略词
  var abbreviations = document.getElementsByTagName("abbr");
  if (abbreviations.length < 1) return false;
  var defs = new Array();
  //遍历这些缩略词
  for (var i = 0; i<abbreviations.length;i++){
    var current_abbr = abbreviations[i];
    var definition = current_abbr.getAttribute("title");
    var key = current_abbr.lastChild.nodeValue;
    defs[key] = definition;
  }
  //创建定义列表
  var dlist = document.createElement("dl");
  //遍历定义列表
  for (key in defs){    //对于defs关联数组里的每个键，把它的值赋给变量key
    var definition = defs[key];
    // 定义标题
    var dtitle = document.createElement("dt");
    var dtitle_text = document.createTextNode(key);
    dtitle.appendChild(dtitle_text);
    //创建定义描述
    var ddesc = document.createElement("dd");
    var ddesc_text = document.createTextNode(definition);
    //添加到定义列表
    ddesc.appendChild(ddesc_text);
    dlist.appendChild(dtitle);
    dlist.appendChild(ddesc);
  }
  //创建标题
  var header = document.createElement("h2");
  var header_text  = document.createTextNode("Abbreviations");
  header.appendChild(header_text);
  //添加到页面主体
  document.body.appendChild(header);
  document.body.appendChild(dlist);
}
addLoadEvent(displayAbbreviations);

```

### 显示“文献来源链接表”：

将文献以连接形式显示出来：
1. 遍历这个文档里所有blockquote元素。
2. 从blockquote元素提取cite属性。
3. 创建一个标识文本是source的链接。
4. 把这个链接赋值为blockquote元素的cite属性值。
5. 把这个链接插入到文献节选的末尾。

```javascript
function displayCitations() {
  if (!document.getElementsByTagName || !document.createElement 
    || !document.createTextNode) return false;
    //取得所有引用
  var quotes = documents.getElementsByTagName("blockquote");
    //遍历引用
  for (var i = 0; i<quotes.length; i++) {
    //如果没有cite属性，继续循环
    if (!quotes[i].getAttribute("cite")){
      continue;
    }
    //保存cite
    var url = quotes[i].getAttribute("cite");
    //取得引用中的所有元素节点
    var quoteChildren = quotes[i].getElementsByTagName("*");
    //如果没有节点元素，继续循环
    if (quoteChildren.length < 1) continue;
    //取得引用中的最后一个元素节点
    var elem = quoteChildren[quoteChildren.length - 1];
    //创建标记
    var link = document.createElement("a");
    var link_text = document.createTextNode("source");
    link.appendChild(link_text);
    link.setAttribute("href",url);
    var superscript = document.createElement("sup");
    superscript.appendChild(link);
    //把标记添加到引用中的最后一个元素节点
    elem.appendChild(superscript);
  }
}
addLoadEvent(displayCitations);
```

### 快捷键清单： 

利用DOM创建快捷键清单：

> 1. 把文档里的所有链接全部提取到一个节点集合中
> 2. 遍历这个节点集合里的所有链接。
> 3. 如果这个链接带有accesskey属性，就把它保存起来。
> 4. 把这个链接在浏览器窗口里的屏显文字也保存起来。
> 5. 创建一个清单。
> 6. 为拥有快捷键的各个链接分别创建一个列表项（li元素）
> 7. 把列表添加到“快捷键清单”里。

```javascript
function displayAccesskeys() {
  if (!document.getElementsByTagName || !document.createElement || !document.createTextNode) return false;
// get all the links in the document
  var links = document.getElementsByTagName("a");
// create an array to store the accesskeys
  var akeys = new Array();
// loop through the links
  for (var i=0; i<links.length; i++) {
    var current_link = links[i];
// if there is no accesskey attribute, continue the loop
    if (current_link.getAttribute("accesskey") == null) continue;
// get the value of the accesskey
    var key = current_link.getAttribute("accesskey");
// get the value of the link text
    var text = current_link.lastChild.nodeValue;
// add them to the array
    akeys[key] = text;
  }
// create the list
  var list = document.createElement("ul");
// loop through the accesskeys
  for (key in akeys) {
    var text = akeys[key];
//  create the string to put in the list item
    var str = key + " : "+text;
// create the list item
    var item = document.createElement("li");
    var item_text = document.createTextNode(str);
    item.appendChild(item_text);
// add the list item to the list
    list.appendChild(item);
  }
// create a headline
  var header = document.createElement("h3");
  var header_text = document.createTextNode("Accesskeys");
  header.appendChild(header_text);
// add the headline to the body
  document.body.appendChild(header);
// add the list to the body
  document.body.appendChild(list);
}
addLoadEvent(displayAccesskeys);
```
效果如图：

![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_004.bmp)

### CSS DOM:

1. 原图

```Html
<table>
    <caption>Itinerary</caption>
    <thead>
      <tr>
        <th>When</th>
        <th>Where</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>June 9th</td>
        <td>Portland,<abbr title="Oregon">OR</abbr></td>
      </tr>
      <tr>
        <td>June 10th</td>
        <td>Seattle,<abbr title="Washington">WA</abbr></td>
      </tr>
      <tr>
        <td>June 12th</td>
        <td>Sacramento,<abbr title="California">CA</abbr></td>
      </tr>
    </tbody>
  </table>

```
    
 
  - 代码如下效果：
<table>
    <caption>Itinerary</caption>
    <thead>
      <tr>
        <th>When</th>
        <th>Where</th>
      </tr>
    </thead>
    <tbody>
      <tr>
        <td>June 9th</td>
        <td>Portland,<abbr title="Oregon">OR</abbr></td>
      </tr>
      <tr>
        <td>June 10th</td>
        <td>Seattle,<abbr title="Washington">WA</abbr></td>
      </tr>
      <tr>
        <td>June 12th</td>
        <td>Sacramento,<abbr title="California">CA</abbr></td>
      </tr>
    </tbody>
  </table>

2. 加入css效果实现可视化：

```css
body {
  font-family: "Helvetica","Arial",sans-serif;
  background-color: #fff;
  color: #000;
}
table {
  margin: auto;
  border: 1px solid #699;
}
caption {
  margin: auto;
  padding: .2em;
  font-size: 1.2em;
  font-weight: bold;
}
th {
  font-weight: normal;
  font-style: italic;
  text-align: left;
  border: 1px dotted #699;
  background-color: #9cc;
  color: #000;
}
th,td {
  width: 10em;
  padding: .5em;
}
```
* 效果如图：

![列表](http://oe9cdien8.bkt.clouddn.com/选区_005.bmp)

3. 如果支持css3就直接使用

```css
tr:nth-child(odd) {background-color:#ffc;}
tr:nth-child(even) {background-color:#fff;}
```

![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_006.bmp)


4. nth-child不能用之后可采用：

> 为表格中的每个奇数行（或每个偶数行）设置一个class属性即可。但增删比较麻烦，可采用js脚本为表格增加斑马线效果，只要隔行设置样式就可以：
 1. 把文档中的table元素找出来。
 2. 对每个table元素，创建odd变量并把它初始化为false。
 3. 遍历这个表格里的所有数据行
 4. 若变量odd的值是true，设置样式并把odd变量修改为false。
 5. 若变量odd的值是false，不设置样式，但把odd变量修改为true。

```javascript
function stripeTables(){
  if (!document.getElementsByTagName) return false;
  var tables = document.getElementsByTagName("table");
  var odd,rows;
  for (var i=0;i<tables.length;i++){
    odd = false;
    rows = tables[i].getElementsByTagName("tr");
    for (var j=0;j<rows.length;j++){
      if(odd==true){
        rows[j].style.backgroundColor = "#ffc";
        odd = false;
      } else {
        odd = true;
      } 
    }
  }
}
addLoadEvent(stripeTables);
```
* 效果如图显示：

![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_007.bmp)

5. 响应事件：

```javascript
function highlightRows() {
  if (!document.getElementsByTagName) return false;
  var rows = document.getElementsByTagName("tr");
  for (var i=0; i<rows.length;i++){
    rows[i].onmouseover = function(){
      this.style.fontWeight = "bold";
    }
    rows[i].onmouseout = function(){
      this.style.fontWeight = "normal";
    }
  }
}
addLoadEvent(highlightRows);
```
效果如图：

![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_008.bmp)

### 动画效果

1. 时间：
setTimeout("function",interval)

>第一个参数是函数名字，第二个是毫秒级的单位。设定过了多少时间执行第一个参数给出的函数。通常赋值一个变量：

```Javascript
    variable = setTimeout("function",interval)
```
clearTimeout函数可以取消“等待执行”队列里的某个函数。

```Javascript
    clearTimeout(variable)
```
    
2. 时间递增量：
让元素的移动以渐变的方式发生：

> 1. 获得元素的当前位置
> 2. 如果元素已经到达它的目的地，则退出这个函数。
> 3. 如果元素尚未到达它的目的地，则把它向目的地移近一点。
> 4. 经过一段时间间隔之后从步骤1开始重复以上步骤。
<pre>parseInt(string)可以把字符串中的数字提取出来，返回值为纯数字。parseFloat(string)可以提取浮点数。</pre>

```javascript
function moveElement(elementID,final_x,final_y,interval) {
  if (!document.getElementById) return false;
  if (!document.getElementById(elementID)) return false;
  var elem = document.getElementById(elementID);
  var xpos = parseInt(elem.style.left);
  var ypos = parseInt(elem.style.top);
  if (xpos == final_x && ypos == final_y) {
    return true;
  }
  if (xpos < final_x) {
    xpos++;
  }
  if (xpos > final_x) {
    xpos--;
  }
  if (ypos < final_y) {
    ypos++;
  }
  if (ypos > final_y) {
    ypos--;
  }
  elem.style.left = xpos + "px";
  elem.style.top = ypos + "px";
  var repeat = "moveElement('"+elementID+"',"+final_x+","+final_y+","+interval+")";
  movement = setTimeout(repeat,interval);
}

function positionMessage() {
  if (!document.getElementById) return false;
  if (!document.getElementById("message")) return false;
  var elem = document.getElementById("message");
  elem.style.position = "absolute";
  elem.style.left = "50px";
  elem.style.top = "100px";
  moveElement("message",125,25,20);
  if (!document.getElementById("message2")) return false;
  var elem = document.getElementById("message2");
  elem.style.position = "absolute";
  elem.style.left = "50px";
  elem.style.top = "50px";
  moveElement("message2",125,75,20);
}
addLoadEvent(positionMessage);
```
 实现效果如下：
 ![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_009.bmp)

### 实用的动画：

```javascript
//把一个节点插入到另外一个节点之后：
function insertAfter(newElement,targetElement) {
  var parent = targetElement.parentNode;
  if (parent.lastChild == targetElement) {
    parent.appendChild(newElement);
  } else {
    parent.insertBefore(newElement,targetElement.nextSibling);
  }
}
```

指针划过链接时候，预览图片：
>为预览图片生成一张“集体照”的图片，通过css的overflow(hideen)属性隐藏图片绝大部分。，当鼠标指针悬停（mouseover）的链接的时候，只显示图片的相应部分

1.moveElement：

```javascript
function moveElement(elementID,final_x,final_y,interval) {
  if (!document.getElementById) return false;
  if (!document.getElementById(elementID)) return false;
  var elem = document.getElementById(elementID);
  if (elem.movement) {
    clearTimeout(elem.movement);
  }
  if (!elem.style.left) {
    elem.style.left = "0px";
  }
  if (!elem.style.top) {
    elem.style.top = "0px";
  }
  var xpos = parseInt(elem.style.left);
  var ypos = parseInt(elem.style.top);
  if (xpos == final_x && ypos == final_y) {
    return true;
  }
  if (xpos < final_x) {
    var dist = Math.ceil((final_x - xpos)/10);
    xpos = xpos + dist;
  }
  if (xpos > final_x) {
    var dist = Math.ceil((xpos - final_x)/10);
    xpos = xpos - dist;
  }
  if (ypos < final_y) {
    var dist = Math.ceil((final_y - ypos)/10);
    ypos = ypos + dist;
  }
  if (ypos > final_y) {
    var dist = Math.ceil((ypos - final_y)/10);
    ypos = ypos - dist;
  }
  elem.style.left = xpos + "px";
  elem.style.top = ypos + "px";
  var repeat = "moveElement('"+elementID+"',"+final_x+","+final_y+","+interval+")";
  elem.movement = setTimeout(repeat,interval);
}
```
2.prepareSlideshow:

```javascript
function prepareSlideshow() {
// Make sure the browser understands the DOM methods
  if (!document.getElementsByTagName) return false;
  if (!document.getElementById) return false;
// Make sure the elements exist
  if (!document.getElementById("linklist")) return false;
  var slideshow = document.createElement("div");
  slideshow.setAttribute("id","slideshow");
  var preview = document.createElement("img");
  preview.setAttribute("src","topics.gif");
  preview.setAttribute("alt","building blocks of web design");
  preview.setAttribute("id","preview");
  slideshow.appendChild(preview);
  var list = document.getElementById("linklist");
  insertAfter(slideshow,list);
// Get all the links in the list
  var links = list.getElementsByTagName("a");
// Attach the animation behavior to the mouseover event
  links[0].onmouseover = function() {
    moveElement("preview",-100,0,10);
  }
  links[1].onmouseover = function() {
    moveElement("preview",-200,0,10);
  }
  links[2].onmouseover = function() {
    moveElement("preview",-300,0,10);
  }
}
addLoadEvent(prepareSlideshow);
```

![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_010.bmp)

### misc:

* css
  * 布局：

```javascript
section,header,article,nav{
  display: block;  //呈现块结构
}
//section 用于有标题之处
* {
  padding: 0;
  margin: 0;
}
//padding 就是内容与边框的空隙。而margin则是模块与模块的空隙。
body {
  margin: 1em 10%; //1em = 16p   %：规定基于父元素的宽度的百分比的外边距
  //length:规定以具体单位计的外边距值，比如像素、厘米等。默认值是 0px
  background-image: url(../images/background.gif);//background-image 属性为元素设置背景图像
  background-attachment: fixed;
  //定义背景图片随滚动轴的移动方式;
    /* scroll: 随着页面的滚动轴背景图片将移动
    fixed: 随着页面的滚动轴背景图片不会移动
    inherit: 继承 */
  background-position: top left;
  background-repeat: repeat-x;
  //background-repeat 属性设置是否及如何重复背景图像。
   //默认地，背景图像在水平和垂直方向上重复。
  max-width: 80em;
}
header {
  background-image: url(../images/guitarist.gif);
  background-repeat: no-repeat;
  background-position: bottom right;
  border-width: .1em;
  border-style: solid;
  //border-style属性设置一个元素的四个边框的样式
  border-bottom-width: 0;
}
header nav {
  background-image: url(../images/navbar.gif);
  background-position: bottom left;
  background-repeat: repeat-x;
  border-width:.1em;
  border-style:solid;
  border-bottom-width: 0;
  border-top-width: 0;
  padding-left: 10%;
}
header nav ul {
  width: 100%;
  overflow: hidden;
  border-left-width: .1em;
  border-left-style: solid;
}
header nav li {
  display: inline;
}
header nav li a {
  display: block;
  float: left;
  padding: .5em 2em;
  border-right: .1em solid;
}
article {
  border-width: .1em;
  border-style: solid;
  border-top-width: 0;
  padding: 2em 10%;
  line-height: 1.80em;
}
article img {
  border-width: .1em;
  border-style: solid;
  outline-width: .1em;
  outline-style: solid;
  //outline-style 属性用于设置元素的整个轮廓的样式。样式不能是 none，否则轮廓不会出现。
//outline （轮廓）是绘制于元素周围的一条线，位于边框边缘的外围，可起到突出元素的作用。outline 属性设置元素周围的轮廓线。
} 
```
效果如图：


![Alt text](http://oe9cdien8.bkt.clouddn.com/选区_011.bmp)


  * * 版式：

```css
body {
  font-size: 76%;
  font-family: "Helvetica","Arial",sans-serif;
}
body * {
  font-size: 1em;
}
a {
  font-weight: bold;
  text-decoration:none;
  //text-decoration 属性规定添加到文本的修饰(下划线等)
}
header nav {
  font-family: "Lucida Grande","Helvetica","Arial",sans-serif;
}
header nav a {
  text-decoration:none;
  font-weight: bold;
}
article {
  line-height: 1.8em;
}
article p {
  margin: 1em 0;
}
h1 {
  font-family: "Georgia","Times New Roman",sans-serif;
  font: 2.4em normal;
}
h2 {
  font-family: "Georgia","Times New Roman",sans-serif;
  font:1.8em normal;
  margin-top: 1em;
}
#imagegallery li{
  list-style-type: :none;
  //设置列表项的样式
}
textarea {
  font-family: "Helvetica","Arial",sans-serif;
}
``` 
* * 将css文件综合到一个基础css文件中调用，方便改动：

```Javascript
import url(layout.css);
@import url(color.css);
@import url(typography.css);
```
<img src="">
                                                                                                                            
                                                                                                                            