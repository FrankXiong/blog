---
title: cookie详解
date: 2016-03-11 00:55:28
tags:
- JavaScript
categories:
- 前端
---
>**前言：**
- Cookie是指web浏览器存储的少量数据，它与具体的web站点相关。Cookie数据会自动在浏览器和服务器之间传输，因此服务器端也可以读写存储在客户端的Cookie值。在JavaScript中，Cookie用于保存状态，以及为浏览器提供一种身份识别机制。
- 检测Cookie是否可用：navigator.cookieEnabled

### Cookie的有效期和作用域
　　Cookie默认的有效期很短暂，只能持续在浏览器的会话期间。如果想要延长Cookie的有效期，可以通过设置max-age属性。
　　Cookie的作用域和localStorage类似，也是通过文档源和文档路径来确定。**默认情况下，Cookie对于创建它的页面，以及与该页面同目录或子目录下的其他web页面可见**。可以通过设置Cookie的path属性来修改Cookie的作用域，如果把path设为“/”，就等同于让Cookie拥有了localStorage的作用域，即整个文档源。
　　Cookie的作用域默认限制在文档源之内，如果想实现同一服务器之下不同子域的跨域访问Cookie，如a.example.com想访问b.example.com设置的Cookie，这时候就可以通过设置Cookie的domain属性来实现。在a.example.com下的一个页面设置了Cookie，将其path设为“/”，并将domain设为“.example.com”，这样该Cookie就对example.com域下的所有页面可见。
　　同时要注意的是，Cookie的**domain只能设置为当前服务器的域**。如想实现Cookie在不同父域下的跨域访问，可参考其他跨域方式，如script标签、隐藏iframe等。
### 创建和存储Cookie
对Cookie的所有操作都要通过**读写document对象的Cookie属性**来完成。Cookie的值都是以键值对的形式存储。
```
//创建一个名字Cookie，同时设置它的过期时间
function setCookie(c_name,value,expiredays){
     var exdate=new Date();
     exdate.setDate(exdate.getDate()+expiredays);
     //encodeURIComponent() 对 URI 进行编码
     document.cookie=c_name+ "=" +encodeURIComponent(value)+
((expiredays==null) ? "" : ";expires="+exdate.toGMTString());
}
```
同样的，如果要设置path、domain等属性，只须以如下形式追加到Cookie值的后面:　
　　;path=path
### 读取Cookie
使用document.cookie可以获取到Cookie的值，不过这个值是一个字符串，为了更好地查看Cookie的值，往往会采用split()方法将Cookie中的名值对分离出来。
```
function getCookie(){
    // 初始化要返回的对象
    var cookie = {};
    var all = document.cookie;
    if(all === null){
        return cookie;
    }
    //分离出Cookie的各个属性
    var list = all.split(';');
    for(var i = 0;i < list.length;i++){
        // 查询出等号所在的位置
        var p = list[i].indexOf('=');
        // 分离出名字和值
        var name = list[i].substring(0,p);
        var value = list[i].substring(p+1);
        //对值进行解码
        value = decodeURIComponent(value);
        // 将名值对存储到对象中
        cookie[name] = value;
    }
    return cookie;
}
```
### Cookie的局限性
1. Cookie只能存储少量的数据，每个Cookie的大小不超过4KB。RFC标准不允许浏览器保存超过300个Cookie，为每个web服务器保存的Cookie数不超过20个。
2. JavaScript中使用Cookie不会采用任何加密机制，因此它们是不安全的。