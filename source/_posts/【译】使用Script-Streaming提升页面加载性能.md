---
title: 【译】使用 Script-Streaming 提升页面加载性能
date: 2018-08-29 21:08:48
tags:
- 翻译
- Chrome
categories:
- 前端
---

# script-streaming 是什么？

加载 JavaScript 算是 web 性能最严重的瓶颈之一，尤其在低端 CPU 上情况更糟糕。加载 JavaScript 的性能损耗不仅包括从服务器上下载数据的网络时间，也包括文件解压、解析、编译，以及执行 JavaScript 的时间。为了更好的用户体验，web 页面[大量的 JavaScript 代码数量持续地增长](https://mobile.httparchive.org/trends.php?s=Top1000&minlabel=Jun+1+2011&maxlabel=Jun+15+2018#bytesJS&reqJS)，页面加载变得越来越慢。浏览器社区一直在开发很多优化措施来提升 JavaScript 加载速度，比如将解析 JavaScript 文件和下载过程并行化（在此之前，浏览器会等待 JavaScript 文件完全下载后才会在渲染主线程上进行解析）。这项优化措施就是 [script-streaming](https://blog.chromium.org/2015/03/new-javascript-techniques-for-rapid.html)，它在 Chrome v41 上被引入。Google 声称 script-streaming 能够将页面加载速度提升 10%，因为大型 JavaScript 文件会边下载边解析，这可以减少数百毫秒的页面加载时间。

Script-streaming 是为加速 JavaScript 解析而做的一项巨大的优化。然而，Chrome 针对 JavaScript 文件启用 script-streaming 还有一些限制。
 
* 首先，下载的 JavaScript 文件大小[至少 30KB](https://cs.chromium.org/chromium/src/third_party/blink/renderer/bindings/core/v8/script_streamer.cc?dr=C&sq=package:chromium&l=325)。这个尺寸限制确保了只有大型 JS 文件能够通过 script-streaming 解析，因为相比于较小的 JavaScript 文件，并行下载和解析大型 JavaScript 文件的收益最大。

* 其次，目前 Chrome 在 script-streaming 的实现上，同一时刻 script-streaming 只能应用到一个 JavaScript 文件上。这是由于 [Chrome 对于 script-streaming 只使用了单线程](https://cs.chromium.org/chromium/src/third_party/blink/renderer/bindings/core/v8/script_streamer_thread.h?type=cs&q=scriptstream&g=0&l=49)。既然这个线程忙于解析某个 JavaScript 文件，那么其他 JavaScript 文件就必须在主线程下载完成后，才能进行解析。

# web 开发者如何利用 script-streaming ?

script-streaming 加速了 JavaScript 的解析。作为一个 web 开发者，你不必为了在页面上启用 script-streaming 做任何事情。然而，script-streaming 还是有一些限制的，这些限制只存在于 Chrome，它们是由于 Chrome 的实现造成的。具体来说，当大型 JavaScript 文件下载完成，script-streaming 线程不保证一定可用。同样地，页面并行下载多个大型 JavaScript 文件，只有一个文件能够通过 script-streaming 解析。并且，某些情况下，script-streaming 可能解析较小的脚本文件，而较大的脚本只能等待 script-streaming 线程可用，或者在完成下载后，由主线程进行解析。

开发者可以使用 [performance 开发者工具](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/reference)来研究 script-streaming 是否被用于解析大型 JavaScript 文件，因为这项技术可以显著提升页面性能。下图是 [www.akamai.com](http://www.akamai.com) 在 performance 面板上的一个截图，脚本就是被红框里的 ScriptStreamer 线程解析的。

![](/uploads/Script-Streaming/1.png)

# 提升解析速度的两种方法

我做了一些研究，为了启用 script-streaming，可以解析较小的 JavaScript 文件（仍然大于 30KB）,这样在解析相对较大的 JavaScript 文件时，保持线程是可用的。理论上，可以在大型 JavaScript 文件上发送一个 HTTP 响应头，告诉浏览器在具有这种响应头的文件上应用 script-streaming。然而，这个方法要求主流浏览器作出调整，并且能让 script-streaming 利用多线程，而不仅仅是单线程。值得注意的是，目前 使用多个 script-streaming 线程在 Chrome 团队内部依然是 [TODO](https://cs.chromium.org/chromium/src/third_party/blink/renderer/bindings/core/v8/script_streamer_thread.h?type=cs&q=scriptstream&g=0&l=50) 状态。

一个比较实用的，执行起来简单得多，并且不需要浏览器改变的技巧，是重新排序 HTML 中那些加载静态资源 URL 的 `<script>` 标签的位置，以让大的 JavaScript 文件在较小（仍然大于 30KB）文件之前下载。当 script-streaming 可用时，此方法会强制解析大型 JavaScript 文件。重新排序 `<script>` 标签在 HTML 中的位置，可能存在潜在风险。因为为了保持页面的功能和 UI，一些 JavaScript 必须按顺序执行。因此，在重新排序时必须要谨慎。**重新排序那些带上=有 **async** 或者 **defer** 属性的 `<script>` 标签会保险一些，因为这些脚本执行并不依赖于执行顺序**。由于脚本重排序伴随着种种限制，这项实验性的方法将只在指定的网站上生效。除了脚本重排序之外，我加进来的另一个方法是，将多个大型 JavaScript 文件并行下载，让它们竞争 script-streaming 线程。我将所有这样的 JavaScript 文件串联起来，目的是让他们都可以实现边下载边解析。

# 实验结果

我做了一些实验，去衡量上述两个方法在多个设备上的页面加载时间，包括 MacBook Pro，一个低端移动设备（Motorola Moto E），和一个高端移动设备（Motorola Moto G）。性能数据通过一个[私有网页测试实例](https://docs.google.com/document/d/1-UKw2FO3YNqS5CjRHrPOwbHjGOIoISd4hGJZ6L7Hws0/edit?usp=sharing).收集。在两个手动修改的网站（“Page A” 和 “Page B”，详情见下表）集合上，观察到在多个移动设备和 MacBook Pro 上，对于中等规模的页面有多达 6.2% 的页面加载时间提升。这些性能提升要归功于 JavaScript 解析和下载的并行化。

**Page** | **\资源数** | **页面大小** | **\#JavaScript 资源数** | ** JavaScript 总大小**
---- | ---- | ---- | ---- | ---- 
A|41|2.7 MB|12|1.3 MB
B|95|2.2 MB|19|1.1 MB

## MacBook Pro 上的性能

图 1 和图 2 展示了在一台 MacBook Pro 上，两个测试页面加载时间的 [CDF](https://www.andata.at/en/software-blog-reader/why-we-love-the-cdf-and-do-not-like-histograms-that-much.html)  分布。页面 A 上，重新排序 script 标签在 HTML 中的位置的页面加载时间减少了 6.2%。页面 B 上，加载重新排序的 scrip 标签的页面加载时间减少了 4.5%。
![](/uploads/Script-Streaming/2.png)


##  Motorola Moto E 上的性能

正如下图所示，重排序 script 标签的页面 A 的加载时间减少了 4.3%。

页面 B（没有示例图）上没有出现更快的加载速度，这可能是因为在 Moto E 设备上，当移动版页面 A 加载时， script-streaming 线程被占用了。
![](/uploads/Script-Streaming/3.png)



## Motorola Moto G 上的性能

如下图所示，重排序 script 标签的页面 A 的加载时间减少了 3.5%。

页面 B 上，中等规模页面的加载时间减少了 1.9%。
![](/uploads/Script-Streaming/4.png)


# 总结

本文描述的实验性工作举例说明了相比默认的解析，通过 script-streaming 解析较大 JavaScript 文件带来的好处。实验包含重排序 HTML 中的 `<script>` 标签，以及串联多个大型 JavaScript 文件，让它们并行下载，以允许相对较大的文件能够通过 script-streaming 解析。实验结果显示，在 MacBook Pro 和两个低端和高端的移动设备上，对于中等大小的页面，加载速度有多达 6% 的提升。

---------
> 原文作者： [Utkarsh Goel](https://www.utkarshgoel.in/)
> 原文：https://developer.akamai.com/blog/2018/07/17/experiment-improving-page-load-times-script-streaming
> 译者： [熊贤仁](https://blog.skrskrskrskr.com)

