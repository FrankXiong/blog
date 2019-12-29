---
title: 【译】HLS 架构简介
date: 2017-03-12 02:03:01
tags:
- 直播
- 翻译
categories:
- 后端
---

> - 原文：[HTTP Streaming Architecture](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/HTTPStreamingArchitecture/HTTPStreamingArchitecture.html)
- 译者：[熊贤仁](https://blog.skrskrskrskr.com)

### 前言
作为 Apple 提出的一种基于 HTTP 的协议，HLS（HTTP Live Streaming）用于解决实时音视频流的传输。尤其是在移动端，由于 iOS 不支持 flash，使得 HLS 成了移动端实时视频流传输的首选。HLS 经常用在直播领域，一些国内的直播云通常用 HLS 拉流（将视频流从服务器拉到客户端）。 HLS 值得诟病之处就是其延迟严重，延迟通常在 10-30s 之间。通过本文你可以了解到 HLS 延迟问题的原因，同时对直播系统的架构有个大概的认知。   

### 概览
在进行编码和鉴权后，HLS 支持从一台普通的 web 服务器上发送实时或预先录制的音视频，接收端可以是 iOS 3.0+ 的任何设备（包括 iPad 和 Apple TV ），或者其他安装有 Safari 4.0+ 的设备。

HLS 由三部分组成：服务器、分发组件（distribution component）和客户端。

服务器用于接收媒体输入流，对它们进行编码，封装成适合于分发的格式，然后准备进行分发。

分发组件（distribution component）组成了标准的 web 服务器。它们用于接收客户端请求，传递处理过的媒体，把资源和客户端联系起来。如果分发规模较大，可能需要使用边缘网络（edge networks）和 CDN。

客户端软件决定请求何种合适的媒体，下载这些资源，然后把它们重新组装成用户可以观看的连续流。客户端环境必须为 iOS 3.0 以上或 Safari 4.0 以上的设备。

在一种典型的配置中，硬件编码器接收音视频输入，将其编码为 H.264 视频和 AAC 音频，然后输出为 MPEG-2 传输流（Transport Stream）。MPEG-2 传输流被软件流分段器（software stream segmenter）分割为一系列短小的媒体文件。这些文件被放进 web 服务器。分段器同时也创造和维护了一个包含媒体文件列表的索引文件。索引文件的URL被发布到 web 服务器上。客户端软件读取索引，然后按序请求列表中的媒体文件，并将其展示出来，片段之间无任务暂停或间隔。

举个简单例子用于说明HTTP 流的配置

![图1-1 基本配置](https://mares.oss-cn-qingdao.aliyuncs.com/blog/hls/1.png)

输入可以是实时的或者录制好的资源。输入文件通常被编码为 MPEG-4 （H.264 视频和 AAC 音频），并通过现有的硬件设备打包进一个 MPEG-2 传输流。MPEG-2 传输流被分段，保存为一系列的媒体文件。这一过程通常使用软件工具来完成，比如 Apple stream segmenter。

只含音频的流可以是一系列 MPEG 基本音频（MPEG elementary audio）文件，比如 MP3 或者 AC-3。文件格式为 AAC，它们带着 ADTS 头部。

分段器（segmenter）同时生成了一个索引文件。索引文件里包含了一个媒体文件列表和元数据（metadata）。这个索引文件是一个 .M3U8 播放列表。索引文件的 URL 可被客户端访问，并按序请求这些文件。

### 服务器
服务器需要进行媒体编码，编码器可以是自带的硬件。还需要将编码后的媒体进行分段，并保存为文件，可以使用 Apple 提供的媒体流分段器或者第三方集成解决方案来完成这一过程。

**媒体编码器（Media Encoder）**

媒体编码器接收来自音视频设备上的实时信号，进行媒体编码，并将其封装成适于传输的格式。应该将媒体编码为客户端设备支持的格式，比如 H.264 视频和 HE-AAC 音频。当下支持 MPEG-2 传输流作为音视频传输格式。对于纯音频，支持 MPEG 基础流（MPEG elementary stream）进行传输。

编码器在本地网络上将编码后的媒体以 MPEG-2 传输流的形式传递给媒体分段器。别把MPEG-2 传输流和 MPEG-2 压缩视频格式弄混了，传输流是一个可以和许多其他不同压缩格式一起使用的打包格式。[Audio Technologies](https://developer.apple.com/library/content/documentation/Miscellaneous/Conceptual/iPhoneOSTechOverview/MediaLayer/MediaLayer.html#//apple_ref/doc/uid/TP40007898-CH9-SW2) 和 [Video Technologies](https://developer.apple.com/library/content/documentation/Miscellaneous/Conceptual/iPhoneOSTechOverview/MediaLayer/MediaLayer.html#//apple_ref/doc/uid/TP40007898-CH9-SW6) 列出了支持的压缩格式。
> 重要提醒：视频编码器不能在编码流的过程中改变流的参数——比如视频大小或者解码类型。如果实在要改变流的参数，应该在片段的边界处改变，并且必须在后续的片段中设置 **EXT-X-DISCONTINUITY** 标签

**流分段器（Stream Segmenter）**

流分段器从本地网络读取传输流，将其划分为一系列长度相同的小的媒体文件。即使每个片段在一个独立的文件中，视频文件都可以通过一个连续流被无缝地重建。

流分段器也将创造一个索引文件，每个索引都映射到一个独立的媒体文件。分段器每次产生一个新的媒体文件，索引文件都会被更新。索引用于追踪媒体文件的可用性和具体位置。分段器可能也需要加密媒体片段，生成一个 key 文件。

媒体片段被保存为 .ts 文件（MPEG-2 传输流文件）。索引文件被保存为 .M3U8 播放列表。

**文件分段器（File Segmenter）**

如果媒体文件早已按照支持的解码格式进行了编码，你可以使用文件分段器将媒体文件封装进 MPEG-2 传输流，然后将它们分割为长度相同的片段。文件分段器支持使用现有的音视频文件库，通过 HLS 来发送点播视频。文件分段器承担了和流分段器相同的任务，但是它的输入是文件，而不是流。

### 媒体段文件（Media Segment Files）
媒体段文件通常由流分段器（stream segmenter）产生，输入来自于编码器，由一系列 .ts 文件组成。.ts 文件包含了一个 MPEG-2 传输流片段，传输流里带有 H.264 视频和 AAC，MP3，或者 AC-3 音频。对于一个纯音频广播，分段器可以生成 MPEG 基础音频流 （MPEG elementary audio stream）。音频流包括了任一种带有 ADTS 头部的 AAC，MP3，或者 AC-3 音频。

### 索引文件（播放列表）
 索引文件通常由流分段器或者文件分段器产生，并保存为 .M3U8 播放列表。.M3U8 扩展自 .m3u，是一种用于 MP3 播放列表的格式。
> 注意：因为索引文件格式一种 .m3u 播放列表格式的扩展 ，而且系统也支持 .mp3 音频，所以客户端软件可能也和用于流媒体的经典的 MP3 播放列表兼容。

关于索引文件，举个简单的例子。如果整个流由三个未加密的 10s 长度的媒体文件组成，分段器可能生成一个 .M3U8 格式的播放列表。
```
#EXT-X-VERSION:3
#EXTM3U
#EXT-X-TARGETDURATION:10
#EXT-X-MEDIA-SEQUENCE:1

# Old-style integer duration; avoid for newer clients.
#EXTINF:10,
http://media.example.com/segment0.ts

# New-style floating-point duration; use for modern clients.
#EXTINF:10.0,
http://media.example.com/segment1.ts
#EXTINF:9.5,
http://media.example.com/segment2.ts
#EXT-X-ENDLIST
```
当发送播放列表给支持 HLS 3.0+ 协议的客户端的时候，为了提高精确度，你应该指定所有持续时间为浮点值。（低版本只支持整形值）你必须在使用浮点长度的时候指定协议的版本；如果该版本已经被删除，播放列表必须遵照 HLS 1.0 协议。
> 当源文件使用 MPEG-4 视频、AAC 或者 MP3 音频时，你可以使用 Apple 提供的文件分段器生成各种示例播放列表。详细说明见[Media File Segmenter](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/UsingHTTPLiveStreaming/UsingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH102-SW7)

索引文件可能也包含了一些用于加密 key 文件的 URL，在不同的带宽下会替换成相应的索引文件。关于索引文件格式的更详细说明，见 IETF Internet-Draft 中  [HTTP Live Streaming specification](http://tools.ietf.org/html/draft-pantos-http-live-streaming) 部分。

索引文件通常由那个生成媒体段文件的分段器产生。或者，如果遵守规范，独立生成 .M3U8 文件和媒体段文件也是可能的。对于纯音频而言，你可以使用文本编辑器生成一个 .M3U8 文件，在文件里列出一系列存在的 .MP3 文件。

### 分发组件（Distribution Components）
分发系统是一个 web 服务器或者 web 缓存系统，系统将媒体文件和索引文件传递给基于 HTTP 的客户端。所有自定义的服务器模块都不必传递内容，而且一般只需要在 web 服务器上做很少的配置。

更详细介绍见 [Deploying HTTP Live Streaming](https://developer.apple.com/library/content/documentation/NetworkingInternet/Conceptual/StreamingMediaGuide/DeployingHTTPLiveStreaming/DeployingHTTPLiveStreaming.html#//apple_ref/doc/uid/TP40008332-CH2-SW3)。

### 客户端
客户端软件先获取索引文件，基于 URL 来定位流。索引文件依次指定可用媒体文件、解码 key 和其他可用的替代流的位置。对于选中的流，客户端按序下载每一个可用媒体文件。每个文件包含了流的连续片段。一旦下载完成，客户端就开始给用户展示重新组装的媒体流。

客户端负责拉取所有解码 key，鉴权或者展示一个允许鉴权的界面，并按需解码媒体文件。

这一过程将在索引文件执行到 #EXT-X-ENDLIST 标签后结束。如果没有  #EXT-X-ENDLIST 标签，系统将一直进行广播。在持续广播过程中，客户端会周期性地加载索引文件的新版本。客户端在更新后的索引中寻找新的媒体文件和解密 key，然后把这些 URL 添加到下载队列中。
