---
title: 【CSS】文本换行的几个属性
date: 2016-03-07 16:58:29
tags:
- CSS
categories:
- 前端
---
**word-wrap:normal | break-word; (内容换行)**         
* normal:默认的属性值（允许内容顶开指定的容器边界）
* break-word:内容将在边界内换行（不截断英文单词换行，截断英文单词下面的属性才具备这个功能）

**word-break:normal | break-all | keep-all (词内换行)**

* normal:如果是中文则到边界处的汉字换行,如果是英文整个词换行,注意:如果出现某个英文字符串长度超过边界,则后面的部分将撑开边框,如果边框为固定属性,则后面部分将无法显示。
* break-all : 强行换行,将截断英文单词。
* keep-all : 不允许字断开。如果是中文，将把前后标点符号内的一个汉字短语整个换行，英文单词也整个换行，注意：如果出现某个英文字符串长度超过边界，则后面的部分将撑开边框,如果边框为固定属性，则后面部分将无法显示。

参数：
normal : 依照亚洲语言和非亚洲语言的文本规则，允许在字内换行
break-all : 该行为与亚洲语言的normal相同。也允许非亚洲语言文本行的任意字内断开。该值适合包含一些非亚洲文本的亚洲文本

keep-all : 与所有非亚洲语言的normal相同。对于中文，韩文，日文，不允许字断开。适合包含少量亚洲文本的非亚洲文本。

说明：
设置或检索对象内文本的字内换行行为。尤其在出现多种语言时。对于中文，应该使用break-all 。对应的脚本特性为wordBreak。

**text-overflow:clip | ellipsis (文本溢出)**

* clip : 　不显示省略标记（...），而是简单的裁切
* ellipsis : 　当对象内文本溢出时（超过width部分）显示省略标记（...）

**white-space: normal | pre | nowrap (内容不换行)**
* normal 默认。空白会被浏览器忽略。 
* pre 空白会被浏览器保留。其行为方式类似 HTML 中的pre 标签。 
* nowrap 文本不会换行，文本会在在同一行上继续，直到遇到 <br> 标签为止。（层中放一个表格，如果层的float：none 则表格和层间会有空隙，这种问题的解决办法是在层的style里面加上white-space: nowrap）
***

例子：
让文本单行显示，并在溢出时，显示省略标记：
```css
white-space: nowrap;
text-overflow: ellipsis;
overflow: hidden;
```
生成效果如下：

![图0.png](/uploads/fe-css-0.png)


另一个腾讯NBA官网的例子（看NBA视频无意间发现的...）

![图1.jpg](/uploads/fe-css-1.jpg)
在这里，腾讯用了上述的三个属性
```css
white-space: nowrap;
word-break: keep-all;
overflow: hidden;
```
　　这里的文本只能单行显示，多余的文本将被截断。其实`word-break:keep-all`这行在这里是多余的，它的作用是控制所有字不能断开，但在后面加上`overflow:hidden`后依然会截断超出盒子宽度的文字。
　　我把其中一行文本替换为一段英文，可以发现英文单词依然被直接截断。
![图2.png](/uploads/fe-css-2.png)