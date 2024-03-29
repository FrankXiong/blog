---
title: 【译】为什么 Vue.js 不支持 templateURL?
date: 2016-08-03 01:48:42
tags:
- JavaScript
- Vue
- 翻译
categories:
- 前端
---
　　对于Vue新手，尤其是之前用过Angular的人来说，有一个非常普遍的问题：“我可以使用`templateURL`吗？”我（尤大）已经回答过很多次这个问题，发现最好写点东西来解释一下。
在Angular中，`templateURL`或者`ng-include`允许开发者在运行时动态地加载远程的模板文件。作为一个内建的特点，这点看起来很方便。但是让我们重新思考一下它解决了什么问题。
　　首先，templateURL允许我们在一个分离的HTML文件中写模板。可以让编辑器提供合适的语法高亮，这可能也是许多框架偏爱于这样做的原因吧。但是，分离模板和JavaScript代码真是的最好的方式吗？对于一个Vue组件，template和JavaScript被原生的紧密连在一起。事实上，如果这些东西仅仅在一个文件里会简单得多。上下文的来回跳转事实上会让开发体验变得更加糟糕。从概念上来说，组件是一个Vue应用的基本构建块，而template不是。每一个Vue.js template和一个伴随着的JavaScript上下文紧密连在一起，没有理由去进一步分离他们。
　　其次，由于`templateURL`通过Ajax在运行时加载模板，你不需要为了分离你的文件而增加一个构建的环节。这为开发提供了便利，但是当你想部署项目到生产环境的时候，这种方式带来了严重的开销。在HTTP/2被普遍支持之前，HTTP请求的数量依然是影响你的应用初次加载时性能的关键因素。现在想象你在应用中每一个组件中使用templateURL，浏览器需要在能展示页面之前做大量的HTTP请求，可能你不知道的是，大多数浏览器限制了它对于单个服务器的并行请求的数量。当你超过了这个限制，应用的初次渲染会承受额外的周转时间，因为浏览器要等待请求的加载。当然，现在也有构建工具，能够帮助你在`$templateCache`中预注册(pre-register)这些模板。但是，这事实上，不可避免的给前端开发增加了一个构建环节。
　　所以，没有templateURL，我们如何处理开发体验的问题？把模板作为行内JavaScript字符串来写是很糟糕的，以`<script type="x/template">`的方式来编写模板感觉像是一个hack。好吧，可能是时候去使用一个像 [Webpack](http://webpack.github.io/)或者[Browserify](http://browserify.org/)之类的模板加载器让程序跑起来。你如果之前没处理过这些问题的话可能会被吓到，但是相信我，做出这种改变是值得的。如果你想要去构建一个大型的可维护的项目时，合适的模块化是必不可少的。更重要的是，当你开始**在单个文件中写Vue组件**时，使用合适的语法高亮，享受着预加载器、热加载(hot-reloading)，ES2015带来的开发上的快感，以及自动补全和scoped css。这些会将开发体验提升10倍。
　　最后，Vue也允许懒加载组件。使用webpack的话，这将变得极其简单。尽管这有一个唯一值得担心的问题是，当初始模块太大时，你最好将它进一步分离。
　　**以组件的方式去思考，而不是模板。**

---------
作者：尤雨溪
原文地址：http://cn.vuejs.org/2015/10/28/why-no-template-url/
译者：[熊贤仁](https://blog.skrskrskrskr.com)
