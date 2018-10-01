---
title: 【译】ES2018 新特性：Promise.prototype.finally()
date: 2018-07-09 21:12:05
tags:
- 翻译
- JavaScript
categories:
- 前端
---


 Jordan Harband 提出了 [`Promise.prototype.finally`](https://github.com/tc39/proposal-promise-finally) 这一章节的提案。

### 如何工作？
.finally() 这样用：
```
promise
.then(result => {···})
.catch(error => {···})
.finally(() => {···});
```
finally 的回调总是会被执行。作为比较：
- then 的回调只有当 promise 为 fulfilled 时才会被执行。
- catch 的回调只有当 promise 为 rejected，或者 then 的回调抛出一个异常，或者返回一个 rejected Promise 时，才会被执行。
换句话说，下面的代码段：
```
promise
.finally(() => {
    «statements»
});
```
等价于：
```
promise
.then(
    result => {
        «statements»
        return result;
    },
    error => {
        «statements»
        throw error;
    }
);
```
### 使用案例
最常见的使用案例类似于同步的 finally 分句：处理完某个资源后做些清理工作。不管是一切正常，还是出现了错误，这样的工作都是有必要的。
举个例子：
```
let connection;
db.open()
.then(conn => {
    connection = conn;
    return connection.select({ name: 'Jane' });
})
.then(result => {
    // Process result
    // Use `connection` to make more queries
})
···
.catch(error => {
    // handle errors
})
.finally(() => {
    connection.close();
});
```

#### .finally() 类似于同步代码中的 finally {}
同步代码里，try 语句分为三部分：try 分句，catch 分句和 finally 分句。
对比 Promise：
- try 分句相当于调用一个基于 Promise 的函数或者 .then() 方法
- catch 分句相当于 Promise 的 .catch() 方法
- finally 分句相当于提案在 Promise 新引入的 .finally() 方法

然而，finally {} 可以 return 和 throw ，而在.finally() 回调里只能 throw, return 不起任何作用。这是因为这个方法不能区分显式返回和正常结束的回调。

### 可用性
- [npm 包 `promise.prototype.finally`](https://github.com/es-shims/Promise.prototype.finally) 是 .finally() 的一个 polyfill
- V8 5.8+ (比如. Node.js 8.1.4+)：加上 --harmony-promise-finally 标记后可用。([了解更多](https://chromium.googlesource.com/v8/v8.git/+/18ad0f13afeaabff4e035fddd9edc3d319152160))

### 深入阅读
* [Promises for asynchronous programming](http://exploringjs.com/es6/ch_promises.html)


--------
原文：http://exploringjs.com/es2018-es2019/ch_promise-prototype-finally.html
