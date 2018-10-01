---
title: 【JavaScript实现数据结构】栈
date: 2016-03-21 17:39:09
tags:
- JavaScript
- 数据结构
- 算法
categories:
- 数据结构与算法
---
堆栈可以用链表和数组两种方式实现，这里分别给出这两种实现方式。
代码如下：
```
//数组实现
function Stack(){
   this.items = [];
   this.size = 0;
}
Stack.prototype = {
   constructor:Stack,
   push:function(data){
       this.items[this.size++] = data;
   },
   pop:function(){
       return this.items[--this.size];
   },
   clear:function(){
       this.size = 0;
       this.items = [];
   },
   perk:function(){
       return this.items[this.size-1];
   }
}
```

```
//链表实现
    function Stack(){
        this.top = null;
        this.size = 0;
    }
    Stack.prototype = {
        constructor:Stack,
        push:function(data){
            var node = {
                data:data,
                next:null
            };
            node.next = this.top;
            this.top = node;
            this.size++;
        },
        pop:function(){
            if(this.top === null){
                return null;
            }
            var out = this.top;
            this.top = this.top.next;
            if(this.size > 0){
                this.size--;    
            }
            return out.data;
        },
        perk:function(){
            return this.top === null ? null:this.top.data; 
        },
        clear:function(){
            this.top = null;
            this.size = 0;
        }
```
测试：
```
var stack = new Stack();
stack.push('k');
stack.push('b');
console.log(stack.perk());//输出b
stack.pop();
console.log(stack.perk());//输出k
```
-----------
### 栈的应用
例子：数值进制转换
算法思想如下：
(1)  最高位为 n % b,将此位压入栈。
(2)  使用 n/b 代替 n。
(3)  重复步骤 1 和 2,直到 n 等于 0,且没有余数。
(4)  持续将栈内元素弹出,直到栈为空,依次将这些元素排列,就得到转换后数字的字符串形式。

具体代码实现：
```    
function mulBase(num,base){
    var stack = new Stack();
    do{
        stack.push(num % base); 
        num = Math.floor(num / base);
    }while(num>0);
    var result = '';
    while(stack.size > 0){
        result += stack.pop();
    }
    return result;
}
console.log(mulBase(234,2)); //输出11101010
```
------------
**初学者学习笔记，如有不对，还希望高手指点。如有造成误解，还希望多多谅解。**