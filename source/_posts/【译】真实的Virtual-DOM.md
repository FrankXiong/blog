---
title: 【译】真实的Virtual DOM
date: 2016-09-08 17:51:03
tags:
- JavaScript
- React
- 翻译
categories:
- 前端
---
###### 为什么我们需要UI框架？
　响应式编程提出两个最重要的观点是：系统应该是事件驱动并且响应状态的变化。
　DOM的UI组件有自己的内部状态，更新浏览器页面并不是在发生变化后简单的重新生成DOM。如果Gmail这样做的话，出现一条新消息或者删掉你写的邮件，就会导致整个浏览器窗口刷新。
　正是因为DOM的无状态性，我们才需要类似key/value observation(Ember中使用了它)，或者脏值检查（Angular）。UI框架监听数据模型的变化，并在DOM中更新对应的部分。或者监听DOM的变化，更新对应的数据模型。这就是所谓的双向数据绑定。它通常可以处理非常复杂的UI逻辑。
<!-- more -->
###### 什么让React与众不同？
　令React以及它的Virtual DOM如此与众不同的是：它比其他实现JavaScript响应数据的方法都更简单。你只需写JavaScript去更新React组件，React会帮你更新DOM。数据绑定并没有和应用缠在一起。
　React使用单向数据绑定来简化这一过程。每次当你在一个React组件的input文本框中输入时，它并不会直接改变这个组件的状态，而是更新数据模型。这会使得UI被更新，你输入的文本就会出现在文本框里。
###### DOM很慢？
　所有关于Virtual DOM的文章或演讲都在说JavaScript引擎很快，读写浏览器的DOM很慢。这种说法完全不对。DOM是很快的。添加删除DOM节点并不比在JavaScript对象上设置属性慢很多，只是简单的运算罢了。
　然而，真正慢的是当DOM改变时浏览器的layout。DOM的每次改变，浏览器都需要重新计算CSS，重排重绘整个页面。这是很耗时的。
那些写浏览器的人一直在努力缩短重绘的时间，其中最大的工作是最小化、分批处理DOM改变。
###### Virtual DOM如何工作？
　就像真实的DOM，Virtual DOM也是一颗节点树。节点树上元素是对象，attribute和内容是对象的属性。React的render方法从组件上创建一颗节点树，在action触发数据模型发生mutation后响应式的更新节点树。
　每次在数据改变之后，就会在UI更新之前创建一个新的Virtual DOM。在React中，更新浏览器的DOM分三个步骤：
**1. 只要数据发生改变，就会重新生成一个完整的Virtual DOM。**
**2. 重新计算比较出新的和之前的Virtual DOM的差异。**
**3. 更新真实DOM中真正发生改变的部分，就像是给DOM打了个补丁。**

###### Virtual DOM慢吗？
　我们可以想到每次改变都重新渲染整个Virtual DOM是很浪费的，却没有注意到任意时刻React都在内存中保存了两个Virtual DOM。但是，其实渲染Virtual DOM总是会比渲染真实DOM快，这跟你使用的浏览器无关。
　问题是用户并不能看到Virtual DOM。就好比你在其他国家有10000个墨西哥煎玉米卷，你迟早需要把玉米卷运回来，那会很贵并且缓慢。
　玉米卷问题的关键在于：一次把所有玉米卷运回来更快，还是计算出你需要的数量和你实际拥有的数量，然后仅仅运输两者最小值更快？当你只想要4个玉米卷的时候，当然只运输4个更划算。
　下一个问题是：你如何预定玉米卷？你可以说，“给我运送4个玉米卷”，或者说，“这是我的玉米卷状态清单，你计算出实际结果吧”。第二个方法就是Virtual DOM工作的方式。你写代码来让UI知道你想让他如何展示，Virtual DOM计算出当前UI和它只需要更新的部分的差异。
　React像变戏法般的将attribute加到元素上，然后在DOM Diff后决定需要更新的部分，并在文档上单独的修改这些元素。Virtual DOM加入了额外的步骤，但是它创造了一种优雅的方式去对页面做最小的更新。
###### 我们来看一些数字
　我不打算做标准的测试。其他人做的各种各样的测试已经证明了React的Virtual DOM更快。Virtual DOM在开发者对浏览器的性能优化之上加了一层脚本。这个额外的一层抽象使React相比其他更新DOM的方法，需要更多的CPU密集计算。
　举个使用原生JS操作DOM的“Hello，world！”的例子：
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Hello JavaScript!</title>
</head>
<body>
  <div id="example"></div>
  <script>
    document.getElementById("example").innerHTML = "<h1>Hello, world!</h1>";
  </script>
</body>
</html>
```
　你在React中也会做同样的事。我们需要通过React、React DOM和babel，将看起来像是XML的代码在render()方法中转换成普通的JavaScript对象。
```
<!DOCTYPE html>
<html>
<head>
<meta charset="UTF-8" />
<title>Hello React!</title>
<script src="build/react.js"></script>
<script src="build/react-dom.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/babel-core/5.8.23/browser.min.js"></script>
</head>
<body>
  <div id="example"></div>
  <script type="text/babel">
    ReactDOM.render(
      <h1>Hello, world!</h1>,    
      document.getElementById('example')
    );
  </script>
</body>
</html>
```
原生操作DOM总是会更快。我们来看一下证明过程。
这是加载和渲染直接DOM操作的“Hello，World”页面的timeline（chrome）

![原生DOM操作](http://upload-images.jianshu.io/upload_images/192464-47f148936b3b9e16.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这是加载和渲染React“Hello，World”页面的timeline（chrome）

![React DOM操作](http://upload-images.jianshu.io/upload_images/192464-d9dad58c3961dc76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
React花了大量时间在scripting上，React比直接操作DOM慢的多。但是，它和jQuery比怎么样？
```
<!DOCTYPE html>
<html>
<head>
  <meta charset="UTF-8" />
  <title>Hello jQuery!</title>
  <script type="text/javascript" src="scripts/vendor/jquery-1.12.3.min.js"></script>
</head>
<body>
  <div id="example"></div>
  <script>
    $(document).ready(function(){
      $("#example").html("<h1>Hello, world!</h1>"); }
    );
  </script>
</body>
</html>
```

![jQuery DOM操作](http://upload-images.jianshu.io/upload_images/192464-6ed99e94b0e681bc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
　jQuery的总时间比原生JS慢了50ms，但都比React快3倍。显然，原生JS和jQuery要快得多。一般来说，使用框架都比不使用框架慢。实际操作DOM前在内存中创建一个表示DOM的结构比直接操作DOM要慢。下面，我们讨论一下究竟该如何让Virtual DOM更快。
###### 如何使用Virtual DOM
　"Hello,World"例子对React不公正，因为他们仅仅包含了一个页面的初始渲染。React长于管理页面的更新。
　**数据模型的每一次改变都会触发Virtual DOM的重新生成，这就是React和其他框架的不同之处**。其他框架会检测文档的变化，只更新必要的部分。Virtual DOM通常占用更少的内存，因为它不需要在内存中常驻观察者。
　但是，每次改动发生后都比较两个完整的Virtual DOM是低效的。复杂的UI对于CPU的要求也很高。
　鉴于这个原因，React开发者要主动决定需要渲染的内容。如果你知道某个行为不会影响对应的组件，你应当告知React不要去分析组件的变动--这可以节省大量的资源，显著地提升应用的性能。
　事实上，可能没有办法真正说明使用Virtual DOM比直接操作DOM快，因为要做这种比较需要考虑各种各样的因素。但是**主要还是取决于你如何更好的优化应用。**
　工具只是工具，关键看你怎么用它。React和Virtual DOM带给我们的是：一种更新页面的简单方法。这种简化能让我们摆脱繁杂的工作，使优化UI变得简单。**这就是React带来的真正好处--性能和生产力**。

--------------

原文作者：[Chris Minnick](https://www.accelebrate.com/blog/the-real-benefits-of-the-virtual-dom-in-react-js/)
翻译：[熊贤仁](http://voidman.xyz)
