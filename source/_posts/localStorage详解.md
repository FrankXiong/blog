---
title: localStorage详解
date: 2016-03-07 21:07:48
tags:
- JavaScript
categories:
- 前端
---
> 前言：在HTML5出现之前，为了保存用户在网站中一些操作状态，以便于下次打开页面时恢复到上次访问时的一些状态，在浏览器端常常使用Cookie来存储一些信息。最典型的应用是判断用户是否登录过网站。但是，Cookie的大小受限，每个Cookie的大小不超过4KB，浏览器一般只允许存放300个Cookie，而且Cookie也存在安全性问题。

好在HTML5为我们带来了全新的本地存储方式：localStorage，有5M大小，而且从IE8就开始支持了。也就是说IE6、7是不支持localStorage的，Cookie可以成为IE6、7下的一种替代方案。

以一个留言板为例,直接上代码：
```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8">
<title>localStorage demo</title>
</head>
<body>
  <div>
    <h3>简单的web存储留言板</h3>
    <textarea id="textarea"></textarea>
    <input type="button" onclick="addInfo()" value="留言" />
    <input type="button" onclick="cleanInfo()" value="清除留言" />
  </div>
  <script type="text/javascript">
        function upInfo() {
            if(window.localStorage){
                var lStorage = window.localStorage;
                var textarea = window.document.getElementById("textarea");
                var text = lStorage.getItem("text");
                if (text) {
                    textarea.value = text;
                }else {
                    textarea.value = "还没有留言";
                }
            }
        }
        function addInfo() {
            if(window.localStorage){
                var textarea = window.document.getElementById("textarea");
                var lStorage = window.localStorage;
                lStorage.setItem("text",textarea.value);
                upInfo();
            }
        }
        function cleanInfo() {
            window.localStorage.removeItem("text");
            upInfo();
        }
        upInfo();
  </script>
</body>
</html>
```
这是用localStorage实现的一个简易留言板，留言板上的信息可以永久保存，即使关闭页面后再次打开。同时也可以清除留言板的内容。

这里我调用了setItem()方法，将对应的名值对传进去，实现数据的存储。调用getItem()方法，将名字传进去，可以获取到对应的值。调用removeItem()方法，将名字传进去，可以删除对应的数据。除此之外，如果想要删除存储对象的所有键值对的话，可以调用removeItem()方法。

目前浏览器似乎只支持存储字符串类型的数据，所以我们想要存储其他类型的数据，不得不自己手动进行编码和解码。

实现了“Web存储”标准的浏览器在window对象上定义了两个属性：localStorage和sessionStorage，这两个属性都代表同一个Storage对象,因此他们具有相同的API。Storage对象的属性值为字符串。

**localStorage和sessionStorage的主要区别在于存储的有效期和作用域不同**。**localStorage存储的数据是永久性的**，其作用域限定在文档源级别（只要URL的协议、端口、主机名三者中有一个不同，就属于不同的文档源）。除此之外，localStorage也受浏览器供应商限制，如果使用chrome访问一个网站，下次用firefox再次访问是获取不到上次存储的数据的。

而**sessionStorage的有效期仅存在于浏览器的标签页**。也就是说如果关闭标签页后，通过sessionStorage存储的数据就都被删除了。**sessionStorage的作用域不仅被限制在文档源，还被限定在窗口中**，也就是同一标签页中。注意，这里说的窗口是指顶级窗口，若果同一标签页中包含多个iframe元素，这两者之间也是可以共享sessionStorage的。