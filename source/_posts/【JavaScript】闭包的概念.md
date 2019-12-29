---
title: 【JavaScript】闭包的概念
date: 2015-12-09 16:01:45
tags:
- JavaScript
categories:
- 前端
---
背景：
* 理解闭包，必须首先要理解变量作用域，关于JavaScript的变量作用域，参见我之前的一篇文章《【JavaScript】变量作用域》
* JavaScript中的函数都是对象

简而言之，JavaScript函数内部的所有变量对外部是不可见的
比如这样的代码会抛出error
```js
var test = function(){
  var i = 0;
}
console.log(i); // undefined error
```
那么怎么让函数访问外部变量呢？
JavaScript拥有函数级的作用域，函数内部调用的函数可以访问其外部函数的变量。所以我们可以在一个函数内部再调用一个函数，这样其内部函数就实现了访问外部变量。这就是所谓的闭包。

闭包的好处是**内部函数可以访问定义它们的外部函数的参数和变量**
****
闭包示例
```js
var test = function(status) {
  return {
    getStatus: function() {
      return status;
    }
  };
};
// 测试
// 调用test时，它返回一个包含getStatus方法的新对象
var myQuo = test('404');
console.log(myQuo.getStatus());
```

另一个例子：构造一个函数，当点击一个节点时，输出当前节点的编号
```js
// 这是一个错误的例子，点击任意节点输出结果都为9
// 这个函数绑定了变量i本身，而不是函数在构造时变量i的值
var addHandlers = function(nodes){
  var i;
  for (i = 0; i < nodes.length; i++) {
    nodes[i].onclick = function(e) {
      console.log(i);
    };
  }
};
```

正确的例子·
```js
var addHandlers = function(nodes) {
  var helper = function(i) {
    return function() {
      console.log(i);
    };
  };
  var i; // 避免在循环中创建函数
  for (i = 0; i < nodes.length; i++) {
    nodes[i].onclick = helper(i);
  }
};
```
