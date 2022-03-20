---
title: Fastify 如何实现更快的 JSON 序列化
date: 2022-03-21 00:06:22
tags: 
- Node.js
- 前端
---

# 前言
对于 web 框架而言，更快的 HTTP 请求响应速度意味着更优异的性能。而 HTTP 协议是一个文本协议，传输的格式都是字符串，而我们在代码中常常操作的是 JSON 格式的数据。因此，需要在返回响应数据前将 JSON 数据序列化为字符串。JavaScript 原生提供了 JSON.stringify 这个方法，来将 JSON 转成字符串。先来介绍这个方法。

# 缓慢的 JSON.stringify
由于 JavaScript 是一个动态语言，它的类型是在运行时才能确定，因此 JSON.stringify 的执行过程中要进行大量的类型判断，对不同类型的键值做不同的处理。由于不能做静态分析，我们很难做进一步优化。而且还需要一层一层的递归，循环引用的话还有爆栈的风险。在以性能著称的 Node.js 框架 Fastify 中，通过使用 fast-json-stringify 这个库，来替代 JSON.stringify，实现 JSON 序列化性能翻倍。那么，fast-json-stringify 是怎么做到的呢？

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/6ab9fce6a06a4422a3bf0ddf43eadffb~tplv-k3u1fbpfcp-watermark.image?)

# fast-json-stringify 揭秘
fast-json-stringify 基于 [JSON Schema Draft 7](https://json-schema.org/specification-links.html#draft-7) 来定义（JSON）对象的数据格式。比如对象：
```
{
    foo: 1, 
    bar: "abc"
}
```
它的 JSON Schema 可以是这样：
```
{
  type: "object",
  properties: {
    foo: {type: "integer"},
    bar: {type: "string"}
  },
  required: ["foo"]
}
```
除了这种简单的类型定义，JSON Schema 还支持一些条件运算，比如字段类型可能是字符串或者数字，可以用oneOf 关键字:
```
"oneOf": [
  {
    "type": "string"
  },
  {
    "type": "number"
  }
]
```
关于 JSON Schema 的完整定义，可以参考 [Ajv](https://ajv.js.org/json-schema.html) 的文档，Ajv 是一个流行的 JSON Schema验证工具，性能表现也非常出众。来看一段使用 fast-json-stringify 的简单代码：
```javascript
require('http').createServer(handle).listen(3000)
var flatstr = require('flatstr')

var stringify = require('fast-json-stringify')({
  type: 'object',
  properties: { hello: { type: 'string' } }
})

function handle (req, res) {
  res.setHeader('Content-Type', 'application/json')
  res.end(flatstr(stringify({ hello: 'world' })))
}
```
这段代码里，fast-json-stringify 接受一个 JSON Schema 对象作为参数，生成了一个 stringify 函数。通常，Response 的数据结构是固定的，所以可以将其定义为一个 Schema，就相当于提前告诉 stringify 函数，需序列化的对象的数据结构，这样它就可以不必再在运行时去做类型判断。下面来看看 stringify 函数是如何生成的。

## 生成 stringify 函数
首先，需要对 JSON Schema 进行校验。底层校验逻辑是基于 Ajv 实现的，这里暂不赘述。
然后需要预先注入一些工具方法，用于将一些常见类型转成字符串。
```
const asFunctions = `
  function $asAny (i) {
    return JSON.stringify(i)
  }
  
  function $asNull () {
    return 'null'
  }
  
  function $asInteger (i) {
    if (isLong && isLong(i)) {
      return i.toString()
    } else if (typeof i === 'bigint') {
      return i.toString()
    } else if (Number.isInteger(i)) {
      return $asNumber(i)
    } else {
      return $asNumber(parseInteger(i))
    }
  }
  
  function $asNumber (i) {
    const num = Number(i)
    if (isNaN(num)) {
      return 'null'
    } else {
      return '' + num
    }
  }
  
  function $asBoolean (bool) {
    return bool && 'true' || 'false'
  }
  
  // 省略了一些其他类型......
`
```
可以看到，使用你使用的是 any 类型，它内部依然还是用的 JSON.stringify。我们经常建议 ts 开发者避免使用 any 类型是有道理的，因为如果是基于 ts interface 生成 JSON Schema 的话，使用 any 也会影响到 JSON 序列化的性能。

接下来，遍历 schema，对不同类型调用对应的工具函数来生成代码。
```javascript
let code = `
    'use strict'
    ${asFunctions}
`
let main
switch (schema.type) {
    // 省略了对象和数组
    case 'integer':
        main = '$asInteger'
        break
    case 'number':
        main = '$asNumber'
        break
    case 'boolean':
        main = '$asBoolean'
        break
    case 'null':
        main = '$asNull'
        break
    case undefined:
        main = '$asAny'
        break
    default:
        throw new Error(`${schema.type} unsupported`)
}

code += `
;
    return ${main}
`
```
最后，对生成出来的 code 调用 Function 构造函数。
```
const dependencies = [new Ajv(options.ajv)]
const dependenciesName = ['ajv']
dependenciesName.push(code)
return (Function.apply(null, dependenciesName).apply(null, dependencies))
```
这里将 ajv 对象作为参数注入到函数里，是为了处理 JSON Schema 中 if、then、else、anyOf 等情况。

另外，由于最终是调用的 new Function 来动态执行代码，这里其实是有一定的安全风险的。所以建议开发者一定不要使用用户生成的 schema，保证生成的 schema 是安全可控的。

最终，开发者调用 stringify 函数，将 JSON 转成字符串。执行 stringify 的过程本质上就是在做字符串拼接。

# 总结
Fastify 使用 fast-json-stringify 替代 JSON.stringify，实现了更快的 JSON 序列化。它的原理是通过开发者预先定义 JSON Schema，使得框架可以提前知道 JSON 数据的结构。然后根据 JSON Schema 生成一个 stringify 函数，stringify 内部做的事情其实就是字符串拼接。最后开发者调用 stringify 函数来序列化 JSON。**本质上是将类型分析从运行时提前到编译时了。**

------
欢迎加微信 xxr0314 聊一聊技术、人生，本人微信公众号：小熊写字的地方

![qrcode_for_gh_a2d61e1595a5_258.jpg](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9f1a1059f5ac4789b1852d4df8c5e7df~tplv-k3u1fbpfcp-zoom-1.image)