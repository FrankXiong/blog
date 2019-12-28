---
title: ES6 语法总结
date: 2017-04-01 00:41:08
tags:
- JavaScript
categories:
- 前端
---
# Arrow Function
箭头函数可以让 this 绑定函数定义时所在的作用域，而不是指向运行时所在的作用域，利用这个特性可以解决一些在匿名回调函数中 this 指向的问题（以前通常用 var that = this 来缓存 this）

# Class
原型链继承的一种语法糖，ES6 的类可以看作是构造函数的另一种写法。
```
class Point {
//...
}
typeof Point //'function'
Point === Point.prototype.constructor
```
- 类内部定义的方法都不可枚举(non-enumerable)
- 不存在变量提升(hoist)
- 实现私有方法
　　1. 将方法移出 Class，定义在全局作用域
　　2. 将私有方法的名字命名为一个 Symbol 值

```
const bar = Symbol('bar');
const snaf = Symbol('snaf');
export default class myClass{
　// 公有方法
　foo(baz) {
　　this[bar](baz);
　}
　// 私有方法
　[bar](baz) {
　　return this[snaf] = baz;
 　}
　// ...
};
```
- 允许继承原生构造函数
- 添加静态属性，静态方法
```
class Foo {
　static classMethod() {
　　return 'hello';
　}
}
Foo.prop = 1;
Foo.prop // 1
Foo.classMethod() // 'hello'
```

# Promise
- 三种状态 Pending、Resolved、Rejected。
- 缺点：
　　1. 无法中途取消 Promise。
　　2. 如果不设置回调，内部抛出的错误无法反应到外部。
　　3. 大量的 then() 语句导致语义不清楚。

# Generator
- yield 语句暂停函数执行。
- generator 返回一个迭代器对象，通过 next() 手动执行迭代器，将指针移向下一个状态。next() 返回一个对象 { done:true, value:xxx}，其中 value 属性的值等于 yield 后面的语句返回的值。
- 使用 for...of 遍历 Iterator 对象
- generator 作用：
　　1. 执行异步操作，将异步操作放在 yield 语句下，等到 next() 方法调用再执行。
　　2. 实现数组数据结构，每一项都是一个函数 。
- generator 缺点：流程管理困难，需要手动执行。解决办法：
　　1. Thunk 函数、传名调用。
　　2. 使用 Co　
　　3. 将异步操作包装成 Promise 对象，用 then 方法交出执行权。

# Module
- 编译时确定模块依赖，编译时加载，使得静态分析成为可能。
- ES6 模块不是对象。
- export 输出的是对外的接口。
- 模块加载实质：CommonJS 输出值的拷贝，ES6 输出值的引用。
- 循环加载：
　　1. CommonJS 是加载时执行。当发生循环加载时，就只输出已经执行的部分。
　　2. ES6 模块是动态引用。只要引用存在，代码就可以执行。
