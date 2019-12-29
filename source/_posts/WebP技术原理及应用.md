---
title: WebP 技术原理及应用
date: 2017-08-04 16:24:00
tags:
- WebP
categories:
- 前端
---

# 背景
浏览器环境下，使用最多的图片格式有 JPEG、PNG、GIF。其中，JPEG 适合色彩复杂的图片，PNG 适合色彩单一或者需要透明的图片，GIF 通常用于动图。现有的图片格式体积较大。
![图1.1- 微店模板编辑页瀑布图](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/1.png)

从瀑布图可见，图片的加载在整个页面加载时间中占据了很大的比重，个别 JPEG 图片甚至达 200 多 KB，这在移动端环境下非常影响用户体验。

# 介绍
WebP 是一个现代的图片格式，用于在 web 上提供更好的有损和无损压缩图片。它能够在肉眼观看几乎一样的情况下，对图片体积进行大幅压缩。在将一张 1.3MB 的 JPG 有损压缩为 WebP 后，大小仅为483KB。你能分辨出下面两张图片有什么差别吗？
![图2.1](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/2.png) ![图2.2](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/3.png)
我们来测试一下 JPG 和 PNG 转成 WebP 后，实际体积大概减少多少。 
![](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/4.png)
根据测试结果可见，对 PNG 进行 WebP 无损压缩后，体积减少了 31%，这与 Google 宣称的 26% 大体吻合。WebP 有损压缩的减少比例则更大，将图片质量降低到原来的 75% 后，减少体积达 90% 左右。值得注意的是，将 JPG 进行 WebP 无损压缩后，图片大小反而增加了 66%。在实际应用中，推荐使用 WebP 有损压缩。

另外，WebP 支持 alpha 透明和 24bit 颜色数，不存在 PNG8 色彩不够丰富和毛边问题。WebP 也支持真彩动图。因此 WebP 可以替代当前大多数图片格式，包括 JPG、PNG、GIF 等。

# 原理
下面来分析一下 WebP 有损压缩的编码过程：
**1. 分块（MacroBlocking）**
  将图片划分成多个宏块（macro blocks），典型的宏块由一个 16×16 的亮度像素(luma pixel)块和两个 8×8 的色度像素(chroma pixel)块组成。分块越小，预测越准，需要记录的信息也越多。一般来说，细节越丰富的地方，分块越细，即使用 4×4 分块预测。细节相对不丰富的地方使用 16×16 分块。（这一过程相当于 JPEG 编码中的色彩空间转换）
![图3.1-分块](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/5.png)

**2. 帧内预测**
WebP 有损压缩使用了帧内预测编码，这一技术也被用于 VP8 视频编码中的关键帧压缩。VP8 有四种常见的帧内预测模型。
- H_PRED(horizontal prediction)
像素块中每一行使用其左边一列（col L）的数据填充（如图3.2 Horizontal）
- V_PRED (vertical prediction)
像素块中每一列使用其上边一行（row A）的数据填充（如图3.2 Vertical）
- DC_PRED (DC prediction)
像素块中每个单元使用 row A 和 col L 的所有像素的平均值填充（如图3.2 Average）
- TM_PRED (TrueMotion prediction)
一种我还没搞清楚的预测模式，比较接近真实数据
  
下图展示了 4×4 分块的所有帧内预测模型
![图 3.2 - VP8 帧内预测模型](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/6.png)

使用哪种分块预测模式是动态决定的。编码器会将所有可能的预测模式都计算出来，然后选出错误程度最小的模式。

**3. DCT（离散余弦变换）**
将预测部分的原图像数据减去预测出来的数据，得到差值矩阵，最后对差值进行 [DCT](https://zh.wikipedia.org/zh-hans/%E7%A6%BB%E6%95%A3%E4%BD%99%E5%BC%A6%E5%8F%98%E6%8D%A2)。此步骤会生成一个频率系数矩阵，左上的系数幅度最大，右下最小。幅度值越小，频率越高。大部分图片信息都在左上区域。这一步的作用就是找出图片的高频和低频区域。

**4. 量化**
人眼对高频部分不敏感，这一步会将高频部分舍去。对上一步的频率系数表和量化表进行计算，将频率系数表和量化表按位相除，并四舍五入位整数。最终生成一个量化矩阵。

**5. 算法编码**
WebP 使用 [Arithmetic entropy encoding](http://en.wikipedia.org/wiki/Arithmetic_coding)，该算法相比JPEG上使用的 Huffman encoding，在压缩表现上更出色。

![图 3.3 - WebP 无损压缩过程](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/7.png)


总结一下，WebP 对图片进行分块，然后对待填充的宏块使用了帧间预测技术，根据其附近已编码宏块的数据，预测出当前块数据。相比 JEPG 对图像原值进行编码，WebP 编码的是预测值和原值的差值，这是 WebP 体积更小的主要原因。最后，WebP 使用了更优秀的算数编码。

# 应用
![图 4.1 - WebP 兼容性](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/8.png)
WebP 完全兼容 Android 4.4 及其以上版本，而在另一大移动平台 iOS 上，则是完全不兼容 WebP。因此，我们需要在前端进行平台检测，对于支持 WebP 的平台输出 WebP，在不支持的平台上采用降级方案。

**方案一：**
JavaScript 判断浏览器是否支持 WebP。
```js
function check_webp_feature(feature, callback) {
  var kTestImages = {
    lossy: 'UklGRiIAAABXRUJQVlA4IBYAAAAwAQCdASoBAAEADsD+JaQAA3AAAAAA',
    lossless: 'UklGRhoAAABXRUJQVlA4TA0AAAAvAAAAEAcQERGIiP4HAA==',
    alpha:
      'UklGRkoAAABXRUJQVlA4WAoAAAAQAAAAAAAAAAAAQUxQSAwAAAARBxAR/Q9ERP8DAABWUDggGAAAABQBAJ0BKgEAAQAAAP4AAA3AAP7mtQAAAA==',
    animation:
      'UklGRlIAAABXRUJQVlA4WAoAAAASAAAAAAAAAAAAQU5JTQYAAAD/////AABBTk1GJgAAAAAAAAAAAAAAAAAAAGQAAABWUDhMDQAAAC8AAAAQBxAREYiI/gcA'
  };
  var img = new Image();
  img.onload = function() {
    var result = img.width > 0 && img.height > 0;
    callback(feature, result);
  };
  img.onerror = function() {
    callback(feature, false);
  };
  img.src = 'data:image/webp;base64,' + kTestImages[feature];
}
```

如果平台支持 WebP，则在请求头 Accept 中带上 image/webp，这样服务器就会知道浏览器是否支持 WebP。

**方案二：**
Google 开发的 PageSpeed 模块，可以自动将图像转出 WebP 或者其他格式。以 Nginx 为例。
首先在 http 模块中开启 pagespeed 属性。
```
pagespeed on;
pagespeed FileCachePath "/var/cache/ngx_pagespeed/“;
```
然后在你的主机配置添加如下一行代码，就能启用这个特性。
```
pagespeed EnableFilters convert_png_to_jpeg,convert_jpeg_to_webp;
```
我们可以看下经过转换后的代码：
页面原始代码：

![图 4.2 - 原始代码](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/9.png)

Chrome 打开后源码如下：

![图 4.3 - Chrome 下的代码](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/10.png)

Safari 打开如下：

![图 4.4 - Safari 下的代码](https://mares.oss-cn-qingdao.aliyuncs.com/blog/WebP%E6%8A%80%E6%9C%AF%E5%8E%9F%E7%90%86%E5%8F%8A%E5%BA%94%E7%94%A8/11.png)

# 总结
显然，WebP 是个好东西，它在肉眼效果几乎一样的情况下，大幅减少了图片体积。本文对 WebP 这种图片格式进行了性能测试分析，并解释了 WebP 有损压缩的实现原理，最后给出了两种应用方案。

---------------------
参考文献：
1. https://isux.tencent.com/introduction-of-webp.html
2. https://developers.google.com/speed/webp/docs/compression#adaptive_block_quantization
3. https://medium.com/@duhroach/how-webp-works-lossly-mode-33bd2b1d0670
4. https://modpagespeed.com/doc/configuration
5. https://juejin.im/entry/5791843bc4c9710054f55751
6. https://aotu.io/notes/2016/06/23/explore-something-of-webp/index.html
