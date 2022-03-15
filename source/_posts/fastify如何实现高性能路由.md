---
title: Fastify 如何实现高性能路由
date: 2022-03-15 23:37:33
tags:
- Node.js
- fastify
categories:
- 前端
---
# 前言

对于一个 Node.js web 框架来说，路由系统的设计是重中之重。所谓路由，干的事情就是根据 HTTP method 和 URL 路径匹配需要执行的处理函数，并在处理函数中提供有关请求和回应的上下文。借助原生的 HTTP 模块，我们可以写出这样的代码，来实现一个简易的路由功能：

```javascript
const http = require("http");

const host = 'localhost';
const port = 8000;

const requestListener = function (req, res) {
  res.setHeader("Content-Type", "application/json");
  switch (req.url) {
        case "/hello":
            res.writeHead(200);
            res.end(JSON.stringify({"message": "world"}));
            break;
        default:
            res.writeHead(404);
            res.end(JSON.stringify({error:"Resource not found"}));
    }
};

const server = http.createServer(requestListener);
server.listen(port, host, () => {
    console.log(`Server is running on http://${host}:${port}`);
});
```

上面的这段代码有几个可优化点：

1. 我们知道 HTTP 是一个文本传输协议，请求和响应内容实际上是一段字符串。因此我们在这里用了 JSON.stringify 来序列化响应 JSON，而响应内容的数据结构通常是固定的，这就给了我们优化的空间。比如已知会返回 `{"message": msg}`，那直接做字符串拼接 `'{"message":'  + msg + '}'`，它的执行速度就会比 JSON.stringify 快。关于这块的性能优化，我会放到下一篇文章中单独介绍。
2. 从可维护性的角度来看，它的请求分发逻辑和处理逻辑是耦合在一起的。在大型项目中，我们可能有成百上千个路由，这样的代码会变得难以维护。
3. 路由查询性能。在这里例子中还不太能体现出来，因为它一共就1条路由规则。但假设项目中有 1000 条路由规则，怎么样根据请求方法和 path，快速匹配到对应的处理函数，这就需要一个高效的路由算法。

# 集中式路由 vs 分散式路由

对于第 2 点，我们很容易可以想到的一个优化方案是：将所有路由注册到一个映射表上。这个映射表大概是这样的：

```text
get / => home#index
get /name => user#index
post /user => user#create
```

箭头左边是请求方法和URL路径，右边是对应的处理函数。这样需要新增路由的时候，只需要更新映射表，而不用调整原有代码逻辑。这也是一种很常见的代码优化技巧。这种我将其称之为集中式路由，因为所有路由规则都是集中管理的。基于这种设计，写出来的 Controller 大概长这样：

```javascript
export default class AppController {
  index(){
    return {
        status: 'ok'
    });
  }
}
```

当请求过来时，框架要找到对应的处理函数，首先要去逐行读取这个映射表文件，对每一行做正则匹配，拿到它的 Controller 名和处理函数名。其次框架要对 Controller 的命名和位置做约束，这样才能根据 Controller 名和处理函数名定位到具体的处理函数。

还有一种思路是，将路由规则的定义放到对应的Controller内部，这样路由的注册就分散到各个Controller了。以 [hoth](https://github.com/searchfe/hoth)(百度内部基于 Fastify 自研的 Node 框架)为例，比如：

```javascript
import {Controller, GET} from '@hoth/decorators';
import type {FastifyReply, FastifyRequest} from 'fastify';

@Controller('/index')
export default class AppController {
    @GET()
    getApp(req: FastifyRequest, reply: FastifyReply) {
        return {
            status: 'ok'
        });
    }
}
```

它使用了 Controller 和 GET 两个装饰器，实现了路由规则和处理函数的解耦，提升了代码的可读性和可维护性。装饰器只是一个语法糖，最终它还是走的 fastify 内部的路由注册逻辑：

```javascript
fastify.get('/index', (req, reply) => {})
```

这里的 fastify.get 方法会在 Node.js 应用启动时执行，将路由规则加载到内存中。

对于小型项目，采用集中式路由或是分散式路由，对性能的影响微乎其微。这两种方案只有在大型项目中才能体现出性能优劣。集中式路由的问题在于，新增了一个（或多个）文件来单独存储这些路由规则，注册路由时框架需要解析文件，这就涉及到文件 IO；查询路由时可能还需要用到正则匹配，这对性能也是一种损耗。而像 fastify 采用的这种分散式路由，路由规则提前被注册到内存中，没有文件 IO，也不用正则来解析规则。

而且集中式注册路由也不利于应用的横向扩展，试想下有多个子应用的场景，框架就需要知道所有子应用的路由规则。

# 更高效的路由算法

除了上面讲到的注册路由的方式，存储路由的数据结构，对性能也有着不小的影响。最简单的可以想到用数组存储路由，它的查询性能是 O(n)，随着路由数量增加，性能会越来越差。而在实际场景中，许多 HTTP 请求的路径有着相同的前缀。某些框架，比如 go 的 [echo](https://github.com/labstack/echo) 和 [gin](https://github.com/gin-gonic/gin)，Node.js 的 fastify 会引入另一种数据结构：Radix-Trie，来存储路由。

> 在[计算机科学](https://zh.wikipedia.org/wiki/计算机科学)中，**基数树**（**Radix Trie**，也叫**基数特里树**或**压缩前缀树**）是一种数据结构，是一种更节省空间的[Trie](https://zh.wikipedia.org/wiki/Trie)（前缀树），其中作为唯一子节点的每个节点都与其父节点合并，边既可以表示为元素序列又可以表示为单个元素。 因此每个内部节点的子节点数最多为基数树的基数*r* ，其中*r*为正整数，*x*为2的幂，*x*≥1，这使得基数树更适用于对于较小的集合（尤其是字符串很长的情况下）和有很长相同前缀的字符串集合。

![ds-radix-tree](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5ba49c1c14f64a26a68cc46ddfb8f9f6~tplv-k3u1fbpfcp-zoom-1.image)

Radix-Trie 的插入、删除、查询的时间复杂度均为 `O(k)`(k为字符串集合中最长的字符串长度），因此无论路由规则增长多少，它都可以保证查询性能始终是常量级的。

fastify 的核心路由逻辑是另一个库 [find-my-way](https://github.com/delvedor/find-my-way) 实现的，fastify 提供了 get、post、all 等方法用来注册路由，它们的内部调用了 find-my-way 上的 router.on 方法。当应用启动时，它会将所有路由构建成多个 Radix-Trie，比如：

```
fastify.get('/index', (req, reply) => {})
```

它其实就是在 Radix-Trie 上插入了一个新节点，tree 的数据结构大体是这样的：

```
{
  prefix: '/',
  label: '/',
  method: 'GET',
  children: {
    i: {
      prefix: 'index',
      label: 'i',
      method: 'GET',
      children: [Object],
      numberOfChildren: 1,
      kind: 0,
      handler: [Object],
    },
    u: {
      prefix: 'user',
      label: 'u',
      method: 'GET',
      children: [Object],
      numberOfChildren: 1,
      kind: 0,
      handler: [Object],
    },
    numberOfChildren: 2,
    kind: 0,
    handler: [Object],
    params: []
  }
}
```

fastify 根据不同 method，将路由划分到多个树中。prefix 标记路由前缀，如果某个节点只有一个子节点，那么会与其父节点合并。kind 用于标记路由的类型，find-my-way 支持这五种路由类型：

```
{
  STATIC: 0, // 静态路由，/info/version
  PARAM: 1, // 带参数路由 /aaa/:id
  MATCH_ALL: 2, // 通配路由 /aaa/*
  REGEX: 3, // 正则路由 /at/(^\\d+)
  MULTI_PARAM: 4 // 多参数路由 /foo/:param1-:param2
}
```

现在有这样几条路由（省略了handler）：

```
GET /user
GET /user/list
GET /user/info
GET /admin/group
GET /admin/delete
```

它们构成的 tree 长这样：

![radix-tree1.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d8001a6470e14b03be68b4bdf74fbcec~tplv-k3u1fbpfcp-watermark.image?)

现在要新增一条路由 GET /usage，假定它是节点 a，插入节点 a 到 tree 的步骤如下：

1. 遍历 tree，找到节点在树中的最长公共前缀长度 len，这里 usage 在树中的最长公共前缀是us，长度是2。

2. 如果最长公共前缀长度 maxCommonPrefixLen 等于当前节点的前缀长度 prefixLen，将（a - 公共前缀）插入到这个节点的子节点。显然节点 a 的 maxCommonPrefixLen 是小于 prefixLen 的。

3. 如果 maxCommonPrefixLen 小于 prefixLen，将节点 a 拆分为公共前缀 + 剩余b，将公共前缀作为父节点，插入树中，然后对剩余部分 b 重复上述步骤。这里会将 us 作为父节点，插入到树中，然后将剩余部分 age 作为子节点插入。

最终，新的 tree 长这样：

![radix-tree2.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/233faffc68654bd2b744a4e5769e0faf~tplv-k3u1fbpfcp-watermark.image?)

讲完了路由节点的插入，再来看路由的查询。这里直接上代码：

```javascript
Router.prototype.find = function find(method, path) {
    var currentNode = this.trees[method];
    if (!currentNode) return null;

    var originalPath = path;
    var originalPathLength = path.length;
    var i = 0;
    var idxInOriginalPath = 0;

    while (true) {
        var pathLen = path.length;
        var prefix = currentNode.prefix;
        var prefixLen = prefix.length;
        var len = 0;
        var previousPath = path;

        // 找到路由，返回handler
        if (pathLen === 0 || path === prefix) {
            var handle = currentNode.handler;
            if (handle !== null && handle !== undefined) {
                return {
                    handler: handle.handler
                };
            }
        }

        // 寻找最长公共前缀长度
        i = pathLen < prefixLen ? pathLen : prefixLen;
        while (len < i && path.charCodeAt(len) === prefix.charCodeAt(len)) len++;

        // 如果最长公共前缀等于当前节点的前缀，将节点一分为二，公共前缀+剩余部分
        // 继续对剩余部分遍历
        if (len === prefixLen) {
            path = path.slice(len);
            pathLen = path.length;
            idxInOriginalPath += len;
        }

        var node = currentNode.findChild(path);

        // 没有子节点，退回到前一次查询的节点
        if (node === null) {
            var goBack = previousPath.charCodeAt(0) === 47 ? previousPath : '/' + previousPath;
            if (originalPath.indexOf(goBack) === -1) {
                var pathDiff = originalPath.slice(0, originalPathLength - pathLen);
                previousPath = pathDiff.slice(pathDiff.lastIndexOf('/') + 1, pathDiff.length) + path;
            }
            idxInOriginalPath = idxInOriginalPath - (previousPath.length - path.length);
            path = previousPath;
            pathLen = previousPath.length;
            len = prefixLen;
        }

        // 静态路由
        if (node.kind === NODE_TYPES.STATIC) {
            currentNode = node;
            continue;
        }
      	// 还有对带参路由、正则路由等类型的判断，这里简化了
    }
};
```

# 总结

web 框架中路由的性能主要取决于路由注册和路由匹配两个环节，采用何种介质、何种数据结构存储路由规则，很大程度影响了路由库的性能。本文介绍了一种数据结构 Radix-Trie，及其它在 fastify 中的应用。

除了路由之外，数据序列化也是一个性能优化点，在进程通信、请求响应等环节都会涉及到序列化。下一篇文章将介绍 fastify 中如何实现的高性能 JSON 序列化。

------
欢迎加微信 xxr0314 聊一聊技术、人生，本人微信公众号：小熊写字的地方

![qrcode_for_gh_a2d61e1595a5_258.jpg](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6e6ad279d1cf4813926c726968f27334~tplv-k3u1fbpfcp-watermark.image?)