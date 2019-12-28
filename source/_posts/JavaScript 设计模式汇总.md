---
title: JavaScript 设计模式汇总
date: 2018-01-23 00:04:48
tags: 
- 设计模式
categories:
- 前端
---

# 单例模式
单例模式算是最简单的一种设计模式，也是 JavaScript 中特别常见一种设计模式。比如创建一个对象`var o = {}`，当对象 o 作为一个全局变量共享时，可以算作一种单例模式。单例模式的核心是确保只有一个实例，并提供全局访问。
实际应用中，对话框等全局唯一的UI组件，会使用到单例模式。以 Dialog 组件为例，我通常会为一个 Dialog 类写一个 getInstance() 的静态方法。代码如下：
```
var getInstance = function() {
  var _instance;
  return function() {
    if (!_instance) {
      _instance = new Dialog();
    }
    return _instance;
  }
}
```
其实就是利用了闭包，将创建的实例缓存了起来，这样保证同一个页面只会存在一个 Dialog 实例。

# 发布-订阅模式
发布-订阅模式，又称观察者模式。它定义了对象间一种一对多的关系。JavaScript中发布-订阅模式可以说无处不在。比如最常见的事件机制，就是一种发布-订阅模式。下面，我们来实现一个简单的事件机制。
```
var Event = (function(){
  var list = {};
  var listen = function(type, fn) {
    if (!list[type]) {
      list[type] = [];
    } 
    list[type].push(fn);
  }
  var trigger = function() {
    var type = Array.prototype.shift.call(arguments);
    var fns = list[type];
    if (!fns || !fns.length) {
      return false;
    }
    for (var i = 0; i < fns.length; i++) {
      fns[i].apply(this, arguments);
    }
  }
  var remove = function(type, fn) {
    var fns = list[type];
    if (!fns || !fns.length) {
      return false;
    } 
    fns.forEach(function(_fn, i) {
      if (_fn == fn) {
        fns.splice(i, 1);
      }
    })
  }
  return {
    listen: listen,
    trigger: trigger,
    remove: remove
  }
})();

Event.listen('click', function(data) {
  console.log('you clicked: ' + data);
})
Event.trigger('click', 'hahaha');
```

# 代理模式

待续...

