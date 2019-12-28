---
title: 【JavaScript 实现算法】快速排序
date: 2016-03-22 21:13:54
tags:
- JavaScript
- 算法
categories:
- 数据结构与算法
---
快速排序是一种分而治之的算法,通过递归的方式将数据依次分解为包含较小元素和较大元素的不同子序列。该算法不断重复这个步骤直到所有数据都是有序的。

快速排序的算法如下:
(1) 选择一个基准元素,将列表分隔成两个子序列;
(2) 对列表重新排序,将所有小于基准值的元素放在基准值的前面,所有大于基准值的元素放在基准值的后面;
(3) 分别对较小元素的子序列和较大元素的子序列重复步骤 1 和 2。

这个算法的JavaScript实现如下：
```
//工具函数：交换两个值
Array.prototype.swap = function (i, j) {
  var t = this[i];
  this[i] = this[j];
  this[j] = t;
};
Array.prototype.quickSort = function () {
  this.quickSortHelper(0, this.length-1);
};
Array.prototype.quickSortHelper = function(start,end){
    if(start>=end){
        return;
    }
    var pivot = this[start];
    var pivotIdx = start;
    var i = start+1;
    var n = end;
    while(i<=n){
        if(this[i] < pivot){
            this.swap(pivotIdx,i);
            i++;
            pivotIdx = i;
        }else{
            this.swap(n,i);
            n--;
        }
    }
    this.quickSortHelper(start,pivotIdx-1);
    this.quickSortHelper(pivotIdx+1,end);
}

```
测试一下快速排序的性能
```
// test
function test () {
    var arr = [];
    for (var i = 0; i < 1000000; i++) {
        arr.push(Math.round(Math.random(i) * 10000));
    }
    doTest(arr, 1);
}
function doTest(arr, n) {
    var tStart = (new Date()).getTime();
    var re = arr.quickSort();
    var tEnd = (new Date()).getTime();
    console.log('快速排序使用时间是:' + (tEnd - tStart) + 'ms');
    return re;
}
test();//输出：快速排序使用时间是:215ms
```