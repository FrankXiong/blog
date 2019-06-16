---
title: 你不知道的eval
date: 2018-11-18 01:32:24
tags:
- JavaScript
categories:
- 前端
---
## 前言
eval() 是 JavaScript 中一个非常有用的函数，它可以一段代码字符串动态执行。然而各种编码规范和最佳实践都强烈抵制 eval，几乎将 eval 打入了死牢，大牛 Douglas Crockford 也在《JavaScript 语言精粹》一书中将 eval 视为 JavaScript 中糟粕。这篇文章将带大家重新认识这个函数，知道为什么不用它，以及为什么不得不用它。

## eval 是什么
在分析 eval 的利弊前，首先来认识一下它。在不清楚一项技术的情况下，就对它做出武断地评价，是有失公允的。 

eval 是全局对象上的一个函数，会把传入的字符串当做 JavaScript 代码执行。如果传入的参数不是字符串，它会原封不动地将其返回。eval 分为直接调用和间接调用两种，通常间接调用的性能会好于直接调用。

直接调用时，eval 运行于其调用函数的作用域下；
```
var context = 'outside';
(function(){
  var context = 'inside';
  return eval('context');
})();

// return 'inside'
```
而间接调用时，eval 运行于全局作用域。
```
var context = 'outside';
(function(){
  var context = 'inside';
  geval = eval;
  return geval('context');
  
  // 下面两种也属于间接调用
  // return eval.call(null, 'context');
  // return (1, eval)('context');
})();

// return 'outside'
```
因此，间接调用时，eval 并不会修改调用函数作用域内的任何东西。JS 解释器有 fast path 和 slow path 两种模式，当直接调用 eval 时，解释器处于 slow path。因为此时作用域是不可控的，需要监听整个作用域，不能应用 v8 的一些编译优化，相应的编译效率也会比 fast path 低。


## 为什么不用 eval
大家抵制 eval 的原因主要是以下几个原因：

1. **降低性能**。具体原因上文已经提到了。网上一些文章甚至说 eval() 会拖慢性能 10 倍。
2. **安全问题**。因为它的动态执行特性，给被求值的字符串赋予了太大的权力，于是大家担心可能因此导致 XSS 等攻击。
3. **调试困难**。eval 就像一个黑盒，其执行的代码很难进行断点调试。

鉴于以上各种原因，很多人说 eval 是 evil（魔鬼）。另外，eval 还有一些难兄难弟，比如 new Function, setTimeout, setInterval。它们也具备执行一段代码字符串的能力。
**究其本质原因，还是因为 JS 赋予这个方法的权限太大了，作为新手很难驾驭它**，如果对 eval 没有很好地理解，很容易写出问题来。这有点像 C 语言中 goto 语句，同样是因为权限太大而被封杀的典范。

## 被误解的 eval
事实上，eval 一直在被误解，它可能是最强大的一个 JavaScript 函数，但却因为一些人的误用，而被开发者们打入了冷宫。接下来，我来根据上述被质疑最多的几个点，给出一点自己的看法。

1. 关于 eval 会拖慢性能 10 倍这个点，出自 Mozila 工程师的演讲 [“Know Your Engines - How to make your JavaScript Fast”](https://www.slideshare.net/newmovie/know-yourengines-velocity2011/4-lost_in_an_instantfunction_f)。
![image.png](https://upload-images.jianshu.io/upload_images/192464-97e5c8ab319630c3.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这是一个发布于 2011 年的演讲，时至今日，JS 引擎早已做了各种优化。我们来测试现在的 JS 引擎中，eval 的实际性能。依然使用上图作为测试用例，测试环境为 node v8.11.1，设 N 的值为 10000。
![image.png](https://upload-images.jianshu.io/upload_images/192464-fc057b2bb1edb308.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
Benchmark 跑出的数据来看，当 N = 10000 时，用了 eval 的 function 执行性能，相比没有 eval 的情况，慢了 3 倍多。
将 N 的值设为 1000000，eval 的性能下降到 8 倍。
![image.png](https://upload-images.jianshu.io/upload_images/192464-a6970c681b262f83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

从测试结果可知，eval 的确会拖慢函数执行性能，而且随着函数规模增大，性能也越慢。但是在一般情况下（N < 1000000），性能差异并没有 10 倍那么夸张。

2. 关于 eval 会导致 XSS 攻击这点，问题并不在 eval，而在数据源。如果数据源本身就是不可靠的，即便你不用 eval，也可能出现 XSS。

3. 至于第三点，eval 代码的确调试起来比较麻烦，但也不是完全没有办法。可以在 eval 创建的代码末尾添加一行 "//@ sourceURL=name" 就可以给这段代码命名（浏览器会特殊对待这种特殊形式的注释），这样它就会出现在 Sources 面板上，然后就可以设置断点调试了。


## 真香警告
虽然大家嘴上说不要用，但是 eval 用起来却是真香。
![](https://upload-images.jianshu.io/upload_images/192464-714d2a5fc4462fe0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
笔者做过的项目中，曾经为了让 HTML 模板（应该说是一套页面主题）也具备动态解析内联表达式的能力，用了 data-eval 将 js 代码存储在 dom 节点，然后渲染时用 with 语句（另一个 JS “毒瘤”，现在严格模式下已经禁用 with 了，rip...）将 data 加到作用域链上，再用 eval 解析执行。实现出来的效果类似这样：
```
<div data-eval="data.count = data.count + 1">
    {{data.count}}
</div>
```
渲染出来的结果是 eval 计算后的值。

很多库和框架都用了 eval 实现各种黑魔法。早期的有用 eval 解析 json 的，比如 Douglas Crockford 的 json2.js（真香！）。到后来，各种 MVVM 框架也用 new Function 这个 eval 的好基友，来实现模板内嵌表达式的计算，比如 Vue 和 avalon。要达到的效果和笔者上面介绍的例子大致相同，不同的是这些 MVVM 框架还需要先解析模板，基于正则表达式提取出 new Function 的参数。甚至Chrome的JavaScript控制台，也是用 eval 实现的。

甚至不能用 eval 的时候，也要自己造一个 eval 出来。比如小程序上就不能使用 eval 和 new Function，那么如果想动态注入并执行代码的话，需要绕一个大弯，从编译原理出发，自行实现一个 JS parser。

## 总结
关于 eval，笔者个人的看法是，**你可以不去用它，但要去了解它**。写这篇文章的目的也不是为了推荐大家使用 eval。就平时的业务开发而言，eval 几乎没有用武之地。但在一些特殊场合，eval 就像一枚核弹，无往不利。

-------
参考链接：

1. [Global eval. What are the options?](http://perfectionkills.com/global-eval-what-are-the-options/)
2. [Knockout, Vue 和 AvalonJS 等 MVVM 框架实现中是否用到 eval 或 Function?](https://www.zhihu.com/question/29743491)
3. [eval() isn’t evil, just misunderstood](https://humanwhocodes.com/blog/2013/06/25/eval-isnt-evil-just-misunderstood/)
4. [A new V8 is coming, Node.js performance is changing.](https://github.com/davidmarkclements/v8-perf)
5. [V8: Behind the Scenes (February Edition feat. A tale of TurboFan)](http://benediktmeurer.de/2017/03/01/v8-behind-the-scenes-february-edition/)


