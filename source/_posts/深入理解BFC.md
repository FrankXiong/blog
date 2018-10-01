---
title: 深入理解BFC
date: 2016-03-09 15:56:53
tags: 
- CSS
categories: 
- 前端
---
### BFC的定义 ###
先看W3C文档
> In a block formatting context, boxes are laid out one after the other, vertically, beginning at the top of a containing block. The vertical distance between two sibling boxes is determined by the['margin'](https://www.w3.org/TR/CSS2/box.html#propdef-margin) properties. Vertical margins between adjacent block-level boxes in a block formatting context [collapse](https://www.w3.org/TR/CSS2/box.html#collapsing-margins).

>In a block formatting context, each box's left outer edge touches the left edge of the containing block (for right-to-left formatting, right edges touch). This is true even in the presence of floats (although a box's *line boxes* may shrink due to the floats), unless the box establishes a new block formatting context (in which case the box itself [*may* become narrower](https://www.w3.org/TR/CSS2/visuren.html#bfc-next-to-float) due to the floats).

Block Formatting Context，中文直译为块级格式上下文。BFC就是一种布局方式，在这种布局方式下，盒子们自所在的containing block顶部一个接一个垂直排列，水平方向上撑满整个宽度（除非内部盒子自己建立了新的BFC）。两个相邻的BFC之间的距离由margin决定。在同一个BFC内部，两个垂直方向相邻的块级元素的margin会发生“塌陷”。

文档这里也间接指出了垂直相邻盒子margin合并的解决办法：就是给这两个盒子也创建BFC。

通俗一点，可以把BFC理解为一个封闭的大箱子，箱子内部的元素无论如何翻江倒海，都不会影响到外部。

### 如何创建BFC ###
再来看一下官方文档怎么说的
> Floats, absolutely positioned elements, block containers (such as inline-blocks, table-cells, and table-captions) that are not block boxes, and block boxes with 'overflow' other than 'visible' (except when that value has been propagated to the viewport) establish new block formatting contexts for their contents.

总结一下就是：
- float属性不为none
- overflow不为visible(可以是hidden、scroll、auto)
- position为absolute或fixed
- display为inline-block、table-cell、table-caption

### BFC的作用
**1. 清除内部浮动**
我们在布局时经常会遇到这个问题：对子元素设置浮动后，父元素会发生高度塌陷，也就是父元素的高度变为0。解决这个问题，只需要把把父元素变成一个BFC就行了。常用的办法是给父元素设置overflow:hidden。
**2. 垂直margin合并**
在CSS当中，相邻的两个盒子的外边距可以结合成一个单独的外边距。这种合并外边距的方式被称为折叠，并且因而所结合成的外边距称为折叠外边距。
折叠的结果：
 - 两个相邻的外边距都是正数时，折叠结果是它们两者之间较大的值。
 - 两个相邻的外边距都是负数时，折叠结果是两者绝对值的较大值。
 - 两个外边距一正一负时，折叠结果是两者的相加的和。
这个同样可以利用BFC解决。关于原理在前文已经讲过了。

**3. 创建自适应两栏布局**
在很多网站中，我们常看到这样的一种结构，左图片+右文字的两栏结构。
```
//CSS
*{
        margin: 0;
        padding: 0;
    }
    .box {
        width:300px;
        border: 1px solid #000;
    }
    .img {
        float: left;
    }
    .info {
        background: #f1f1f1;
        color: #222;
    }
//HTML
<\div class="box">
        <\img src="03.jpg" alt="" class="img">
        <\p class="info">信息信息信息信息信息信息</p>
    </\div>
```
一般情况下，它是这样的

![图1](http://upload-images.jianshu.io/upload_images/192464-7422273b46506f7e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

但是当文字多了以后...


![图2](http://upload-images.jianshu.io/upload_images/192464-47f55b6a8de7b3c1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

显然，这是文字受到了图片浮动的影响。当然，如果你想做文本绕排的效果，浮动是不二之选。不过在这里，这显然不是我们想要的。此时我们可以为P元素的内容建立一个BFC，让其内容消除对外界浮动元素的影响。给文字加上overflow:hidden
![图3](http://upload-images.jianshu.io/upload_images/192464-b2e09148be9db84c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

两栏布局就完成了。我们改变图片的大小：
![图4](http://upload-images.jianshu.io/upload_images/192464-2620aa5e31bd83f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
两栏布局的结构依然没有改变，如此就实现了两栏自适应布局。

参考链接：
  [Visual formatting model](https://www.w3.org/TR/CSS2/visuren.html#block-formatting)
  [CSS深入理解流体特性和BFC特性下多栏自适应布局](http://www.zhangxinxu.com/wordpress/?p=4588)