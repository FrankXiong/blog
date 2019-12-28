---
title: Cookie 和 Session 的区别总结
date: 2016-06-26 13:55:10
tags:
- JavaScript
categories:
- 前端
---
二者作用：解决HTTP协议无状态的缺陷，在客户端/服务器端保存会话状态。
<!-- more -->
创建Session过程：
- 检查客户端请求中是否包含一个session标识（session id）
- 若包含，则说明之前已经为此客户端创建过session。服务器按照此session id检索出session
- 若不包含，则为此客户端创建一个session，并生成一个session id。此session id将作为响应返回给客户端保存。（使用Cookie保存）

若Cookie被禁止，必须有其他机制能够把session id回传给服务器
回传session id至服务器：

- URL重写：把session id直接附加在URL路径后面
- 隐藏表单字段

Cookie和Session的区别：
- **Cookie中只能保存ASCII字符串，Session中可以保存任意类型的数据**，甚至Java Bean乃至任何Java类、对象等
- **隐私策略不同**。Cookie存储在客户端，对客户端是可见的，可被客户端窥探、复制、修改。而Session存储在服务器上，不存在敏感信息泄露的风险
- **有效期不同**。Cookie的过期时间可以被设置很长。Session依赖于名为JSESSIONI的Cookie，其过期时间默认为-1，只要关闭了浏览器窗口，该Session就会过期，因此Session不能完成信息永久有效。如果Session的超时时间过长，服务器累计的Session就会越多，越容易导致内存溢出。
- **服务器压力不同**。每个用户都会产生一个session，如果并发访问的用户过多，就会产生非常多的session，耗费大量的内存。因此，诸如Google、Baidu这样的网站，不太可能运用Session来追踪客户会话。
- **浏览器支持不同**。Cookie运行在浏览器端，若浏览器不支持Cookie，需要运用Session和URL地址重写。
- **跨域支持不同**。Cookie支持跨域访问（设置domain属性实现跨子域），Session不支持跨域访问

------------
参考：[理解Cookie和Session机制](http://www.lai18.com/content/7450273.html)
