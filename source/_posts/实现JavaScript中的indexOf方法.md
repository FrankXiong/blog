---
title: 实现JavaScript中的indexOf方法
date: 2016-03-21 17:43:49
tags:
- JavaScript
categories:
- 前端
---
> indexOf()方法是ES5中出现的数组方法，它有两个参数
array.indexOf(value,start)
第一个参数指定要在数组查找的值，第二个可选参数指定开始查找的数组下标。如果省略，则为0。如果数组中存在匹配的值，就返回第一次匹配的数组下标，如果不存在匹配的值，则返回-1。
示例：['a','b','c'].indexOf('a',1)  //返回-1

下面我们来自己实现这个方法，并保证其向下兼容性。
```
var indexof = function(array,value,start){
   if(array == null) return -1;
   var i=0,length = array.length;
   if(start){
       if(typeof start == 'number'){
           // 添加对start为负值时的处理
           i = (start < 0 ? Math.max(0,length+start):start);
        }
   }
   // 如果浏览器支持ES5 indexOf，则直接使用它
   if(Array.prototype.indexOf && array.indexOf === Array.prototype.indexOf){
        return array.indexOf(value,start);
   }
   // 遍历数组
   for(;i < length;i++){
       if(array[i]===value){
           return i;
       }
   }
   return -1;
}
//测试
console.log(indexof([2,4,1,8,5],1,0));//输出2
```
如此，我们就扩充了indexOf方法，使其在即便不支持的ES5的浏览器也能运行良好。
