---
title: 【JavaScript实现数据结构】队列
date: 2016-03-21 17:42:51
tags:
- JavaScript
- 数据结构
categories:
- 数据结构与算法
---
队列是一种先进先出（FIFO）的数据结构，其实现方式主要分两种：顺序队列和链式队列，本文将给出顺序队列的JavaScript实现。

JavaScript提供的数组原生方法：push()可以在数组末尾插入元素，shift()可以删除数组的第一个元素，利用这两个方法可以很容易实现队列的“入队”和“出队”。代码如下：
```
function Queue(){
    this.items = [];
}
Queue.prototype = {
    enqueue:function(data){
        this.items.push(data);
    },
    dequeue:function(){
        return this.items.shift();
    },
    front:function(){
        return this.items[0];
    },
    rear:function(){
        return this.items[this.items.length-1];
    },
    clear:function(){
        this.items = [];
    },
    length:function(){
        return this.items.length;
    },
    displayAll:function(){
        var str = '';
        for(var i = 0;i < this.length();i++){
            str += this.items[i];
        }
        return str;
    }
}
```
测试：
```
var queue = new Queue();
queue.enqueue('aaa');
queue.enqueue('bbb');
console.log(queue.length());//输出2
queue.dequeue();
console.log(queue.displayAll());//输出bbb
```
-----------
**初学者学习笔记，如有不对，还希望高手指点。如有造成误解，还希望多多谅解。**