---
title: apply 方法的妙用
date: 2016-03-16 00:37:39
tags:
- JavaScript
categories:
- 前端
---
我们可以使用数组的 push() 方法来合并数组
```
var a = [1,2,3];
var b = [4,5,6];
Array.prototype.push.apply(a,b);
console.log(a);//输出1,2,3,4,5,6
```
push方法本身没有提供push一个数组，但它提供了push(param1,parm2...)，支持传入多个参数。
而**apply方法可以将一个数组转换为一个参数列表**，apply的第一个参数用于改变this对象，将数组a传给它，也就相当于在a上调用了push方法。第二个参数是一个数组，它将作为参数传给push()方法。

此外，找出数组中的最大值、最小值也均可使用此方法。
如求出数组中的最大值：
```
var a = [3,4,1,5,9];
var max = Math.max.apply(null,a);
console.log(max);//输出9
```