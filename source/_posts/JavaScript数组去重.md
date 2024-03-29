---
title: JavaScript 数组去重
date: 2016-03-21 17:42:35
tags:
- JavaScript
- 算法
categories:
- 数据结构与算法
---
话说面试常会碰到面试官会问JavaScript实现数组去重的问题，最近刚好在学习有关于[JavaScript数组相关的知识](http://www.w3cplus.com/blog/tags/538.html),趁此机会整理了一些有关于JavaScript数组去重的方法。

下面这些数组去重的方法是自己收集和整理的，如有不对希望指正文中不对之处。
### 双重循环去重 ###
-----------
这个方法使用了两个for循环做遍历。整个思路是：
- 构建一个空数组用来存放去重后的数组
- 外面的for循环对原数组做遍历，每次从数组中取出一个元素与结果数组做对比
- 如果原数组取出的元素与结果数组元素相同，则跳出循环;反之则将其存放到结果数组中

代码如下:
```
Array.prototype.unique1 = function () { 
　// 构建一个新数组，存放结果 var newArray = [this[0]]; 
　// for循环，每次从原数组中取出一个元素 
　// 用取出的元素循环与结果数组对比 
　for (var i = 1; i < this.length; i++) { 
　　var repeat = false; 
　　for (var j=0; j < newArray.length; j++) { 
　　// 原数组取出的元素与结果数组元素相同 
　　　if(this[i] == newArray[j]) { 
　　　　repeat = true; break; 
　　　} 
　　} 
　　if(!repeat) { 
　　// 如果结果数组中没有该元素，则存放到结果数组中 　　　　
　　　newArray.push(this[i]);
　　} 
　} 
　return newArray;
}
```
假设我们有一个这样的数组：
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2'];
arr.unique1(); // [1, 2, 3, 4, "a", "b", 56, 32, 34, "c", 5]
```
据说这种方法比较耗时，费性能。简单做个测试(测试方法写得比较拙逼)：
```
function test () { 
　var arr = []; 
　for (var i = 0; i < 1000000; i++) { 　　
　　arr.push(Math.round(Math.random(i) * 10000)); 
　} 
　doTest(arr, 1);
}
function doTest(arr, n) { 
　var tStart = (new Date()).getTime(); 
　var re = arr.unique1(); 
　var tEnd = (new Date()).getTime(); 
　console.log('双重循环去重方法使用时间是:' + (tEnd - tStart) + 'ms'); 
　return re;
}
test();
```
在Chrome控制器运行上面的代码，测试双重循环去重所费时间：11031ms。

上面的方法可以使用forEach()方法和indexOf()方法模拟实现：
```
function unique1() { 
　var newArray = []; 
　this.forEach(function (index) { 
　　if (newArray.indexOf(index) == -1) { 
　　　newArray.push(index); 
　　} 
　}); 
　return newArray;
}
```
通过unique1.apply(arr)或unique1.call(arr)调用。不过这种方法效率要快得多，同样的上面测试代码，所费时间5423ms，几乎快了一半。
### 排序遍历去重 ###
-----------
先使用sort()方法对原数组做一个排序，排完序之后对数组做遍历，并且检查数组中的第i个元素与结果数组中最后一个元素是否相同。如果不同，则将元素放到结果数组中。
```
Array.prototype.unique2 = function () { 
　// 原数组先排序 
　this.sort(); 
　// 构建一个新数组存放结果 
　var newArray = []; 
　for (var i = 1; i < this.length; i++) { 
　　// 检查原数中的第i个元素与结果中的最后一个元素是否相同 
　　// 因为排序了，所以重复元素会在相邻位置 
　　if(this[i] !== newArray[newArray.length - 1]) { 
　　　// 如果不同，将元素放到结果数组中 
　　　newArray.push(this[i]); 
　　} 
　} 
　return newArray;
}
```

例如：
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2'];
arr.unique2(); // ["1", 1, 2, "2", 3, 32, 34, 4, 5, 56, "a", "b", "c"]
```

这种方法有两个特色：
去重后的数组会做排序，主要是因为原数在去重前做了排序去重后的数组，与数字相同的数字字符无法区分，比如'1'和1

使用同样的方法，测试所费时间：1232ms。
### 对象键值对法 ###
---------
这种去重方法实现思路是：
创建一个JavaScript对象以及新数组使用for循环遍历原数组，每次取出一个元素与JavaScript对象的键做对比
如果不包含，将存入对象的元素的值推入到结果数组中,并且将存入object
对象中该属性名的值设置为1

代码如下:
```
Array.prototype.unique3 = function () { 
　// 构建一个新数组存放结果 
　var newArray = []; 
　// 创建一个空对象 
　var object = {}; 
　// for循环时，每次取出一个元素与对象进行对比 
　// 如果这个元素不重复，则将它存放到结果数中 
　// 同时把这个元素的内容作为对象的一个属性，并赋值为1, 
　// 存入到第2步建立的对象中 
　for (var i = 0; i < this.length; i++){ 
　　// 检测在object对象中是否包含遍历到的元素的值 
　　if(!object[typeof(this[i]) + this[i]]) { 
　　// 如果不包含，将存入对象的元素的值推入到结果数组中
　　　newArray.push(this[i]); 
　　　// 如果不包含，存入object对象中该属性名的值设置为1 
　　　object[typeof(this[i]) + this[i]] = 1; 
　　} 
　} 
　return newArray;
}
```
运行前面的示例：
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2'];
arr.unique3(); // [1, 2, 3, 4, "a", "b", 56, 32, 34, "c", 5, "1", "2"]
```
同样的，不同的键可能会被误认为一样；例如： a[1]、a["1"]。这种方法所费时间：621ms。 这种方法所费时间是最短，但就是占用内存大一些。

除了上面几种方法，还有其他几种方法如下：
```
// 方法四
Array.prototype.unique4 = function () { 
　// 构建一个新数组存放结果 
　var newArray = []; 
　// 遍历整个数组 
　for (var i = 0; i < this.length; i++) { 
　　// 遍历是否有重复的值 
　　for (j = i + 1; j < this.length; j++) { 
　　　// 如果有相同元素，自增i变量，跳出i的循环 
　　　if(this[i] === this[j]) { 
　　　　j = ++i; 
　　　}　 
　　} 
　　// 如果没有相同元素，将元素推入到结果数组中 　
　　newArray.push(this[i]); 
　} 
　return newArray;
}
```
Chrome测试结果
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2'];
arr.unique4(); // ["a", 1, 3, 4, 56, 32, 34, 2, "b", "c", 5, "1", "2"]
```
同样的，1和'1'无法区分。
```
// 方法五
Array.prototype.unique5 = function () { 
　// 构建一个新数组存放结果 
　var newArray = []; 
　// 遍历整个数组 
　for (var i = 0; i < this.length; i++) { 
　　// 如果当前数组的第i值保存到临时数组，那么跳过 
　　var index = this[i]; 
　　// 如果数组项不在结果数组中，将这个值推入结果数组中 
　　if (newArray.indexOf(index) === -1) { 
　　　newArray.push(index); 
　　} 
　} 
　return newArray;
}
```
Chrome测试结果:
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2'];
arr.unique6(); // [1, 2, 3, 4, "a", "b", 56, 32, 34, "c", 5, "1", "2"]
```
同样的，类似于1和'1'无法区分。所费时间：14361ms。
```
// 方法六
Array.prototype.unique6 = function () { 
　return this.reduce(function (newArray, index) { 　　
　　if(newArray.indexOf(index) < 0) { 
　　　newArray.push(index); 
　　} 
　　return newArray; 
　},[]);
}
```
测试结果如下：
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2']; 
arr.unique6(); // [1, 2, 3, 4, "a", "b", 56, 32, 34, "c", 5, "1", "2"]
```
所费时间：16490ms。
```
// 方法七
Array.prototype.unique7 = function(){ 
　var newArray; 
　newArray = this.filter(function (ele,i,arr) { 
　　return arr.indexOf(ele) === i; }); 
　　return newArray;
}
```
测试结果：
```
var arr = [1,2,3,4,'a','b',1,3,4,56,32,34,2,'b','c',5,'1','2']; 
arr.unique6(); // [1, 2, 3, 4, "a", "b", 56, 32, 34, "c", 5, "1", "2"]
```
所费时间：13201ms。
方法虽然很多种，但相比下来，下面这种方法是较为优秀的方案：
```
//方法三
Array.prototype.unique3 = function () { 
　// 构建一个新数组存放结果 
　var newArray = []; 
　// 创建一个空对象 
　var object = {}; 
　// for循环时，每次取出一个元素与对象进行对比 
　// 如果这个元素不重复，则将它存放到结果数中 
　// 同时把这个元素的内容作为对象的一个属性，并赋值为1, 
　// 存入到第2步建立的对象中 
　for (var i = 0; i < this.length; i++){ 
　　// 检测在object对象中是否包含遍历到的元素的值 
　　if(!object[typeof(this[i]) + this[i]]) { 
　　// 如果不包含，将存入对象的元素的值推入到结果数组中
　　　newArray.push(this[i]); 
　　　// 如果不包含，存入object对象中该属性名的值设置为1 
　　　object[typeof(this[i]) + this[i]] = 1; 
　　} 
　} 
　return newArray;
}
```
但在ES6去重还有更简单，更优化的方案，比如：
```
// ES6
function unique (arr) { 
　const seen = new Map() 
　return arr.filter((a) => !seen.has(a) && seen.set(a, 1))
}
// or
function unique (arr) { 
　return Array.from(new Set(arr))
}
```
----------
转载自：http://www.w3cplus.com/javascript/remove-duplicates-from-javascript-array.html
