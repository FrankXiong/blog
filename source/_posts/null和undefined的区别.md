---
title: null和undefined的区别
date: 2016-03-08 21:52:17
tags:
- JavaScript
categories:
- 前端
---
 1.null和undefined都被用来表示空值，**当使用不严格等于号（==）做判断时，他们是等价的**。
``` 
console.log(null == undefined);//输出true
console.log(null === undefined);//输出false
```
这也是为什么我们在代码中判断相等时避免使用==

2.当对null执行typeof运算时，结果返回object，也就是说**null是一个对象**，表示“空对象”
null的典型用法包括：
- 作为函数的参数，表示该函数的参数不是对象。比如在使用Ajax进行get时，我们常用request.send(null)表示不发送数据
- 作为对象原型链的终点。在使用for in遍历原型链的会用到。
```
console.log(Object.getPrototypeOf(Object.prototype));//输出null
```

3.undefined，顾名思义表示“未定义”，它是变量的一种取值，表示变量没有初始化。
undefined的用法包括：
- 变量被声明了，但没有赋值时，等于undefined
JavaScript函数作用域中会发生变量申明提前
```
var func = function(){
      console.log(a);//输出undefined
      var a = "hello";
};
func();
```
等价于
```
var func = function(){
      var a;
      console.log(a);//输出undefined
      a = "hello";
};
func();
```
- 查询数组元素或对象属性时返回undefined，表示该元素或属性不存在
在使用标准for遍历数组时，如果数组中某个元素未定义，就会输出undefined
```
var mycars = new Array();
mycars[0] = "Saab"; 
mycars[2] = "Volvo"; 
mycars[4] = "BMW";
for (y=0;y<\mycars.length;y++){ 
    console.log(mycars[y]); //输出Saab,undefind,Volvo,undefined,BMW
}
```
但是用for in遍历时，并不会输出undefined
```
for (y in mycars){ 
    console.log(mycars[y]); //输出Saab,Volvo，BMW
}
```
所以有人推荐不使用for in，其实还有更深层次的原因。因为for in是对整个原型链的遍历，如果我们修改了数组的原型，那么遍历出的结果就不仅仅是数组中的元素了。
- 函数没有返回值，返回undefined
- 调用函数时，应该提供的参数没有提供，该参数等于undefined
