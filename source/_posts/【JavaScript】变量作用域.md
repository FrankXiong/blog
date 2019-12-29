---
title: 【JavaScript】变量作用域
date: 2015-12-08 01:47:30
tags: 
- JavaScript
categories:
- 前端
---
背景知识：
*  编程语言中，作用域控制变量与参数的可见性和生命周期
* 函数体内，局部变量的优先级高于同名的全局变量
* 块级作用域：花括号内的每一段代码都具有各自的作用域

1.**JavaScript不支持块级作用域**
JavaScript的<b>函数作用域</b>：变量在声明它们的函数体以及这个函数体嵌套的任意函数体内都是有定义的
```js
function hello() {
  for (var i = 0; i < 10; i++) {
    // doSomething...
  }
  //输出10，在支持块级作用域的语言中这里会报错
  console.log(i);
}
```

2.声明提前：**JavaScript函数里申明的所有变量都被提前至函数体顶部**
```js
var scope = 'global';
function test() {
  // 输出undefined，这里scope只是申明，还没有被赋初值
  console.log(scope);
  //scope在这里被赋初值，但scoop的申明发生在函数体顶部
  var scope = 'local';
  // 输出local
  console.log(scope);
}
```

该函数等价于：
```js
var scope = 'global';
function test() {
  var scope;
  console.log(scope);
  scope = 'local';
  console.log(scope);
}
```