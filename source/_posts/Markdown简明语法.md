---
title: Markdown简明语法
date: 2016-03-07 16:37:23
tags: 
- Markdown
categories:
- 工具
---
### 标题
在 Markdown 中，如果一段文字被定义为标题，只要在这段文字前加 # 号即可。
`# 一级标题`
`## 二级标题`
`### 三级标题`
`#### 四级标题`
`##### 五级标题`
`###### 六级标题`

### 列表

在 Markdown 下，列表的显示只需要在文字前加上 - 或 * 即可变为无序列表，有序列表则直接在文字前加1. 2. 3. 符号要和文字之间加上一个字符的空格。
`_列表内容`
`1. 列表内容`

无序列表使用 \* + 和 - 来做为列表的项目标记，这些符号是都可以使用的

### 修辞和强调
Markdown 使用 * 和 _ 来标记需要强调的区段。

`*需要强调的内容*`
`**需要强调的内容**`

### 引用
只需要在文本前加入 > 这种尖括号（大于号）即可
`> 引用的内容`

### 图片与链接
###### 链接
Markdown 支持两种形式的链接语法： **行内** 和 **参考** 两种形式，两种都是使用[ ]来把文字转成连结。
1. 行内形式是直接在后面用括号直接接上链接：
This is an \[example link\]\(http://example.com\).
2. 参考形式的链接让你可以为链接定一个名称，之后你可以在文件的其他地方定义该链接的内容：
I get 10 times more traffic from [Google] than from[Yahoo] or [MSN]: 
http://google.com/ "Google": 
http://search.msn.com/ "MSN"

###### 图片
图片的语法和链接很像。

1. 行内形式（title 可选）：
\[imgUrl ("Title")]
2. 参考形式：
\![alt text][id]
\[id]: imgUrl "Title"

上面两种方法都会输出 图片 为：
![avatar.jpg](/uploads/avatar.jpg)

### 代码
在一般的段落文字中，你可以使用反引号`来标记代码区段，区段内的 & 、< 和 > 都会被自动的转换成 HTML 实体，这项特性让你可以很容易的在代码区段内插入 HTML 码：

``I strongly recommend against using any `<blink>` tags.I wish SmartyPants used named entities like `—`instead of decimal-encoded entites like `—\`.``

如果要建立一个已经格式化好的代码区块，只要每行都缩进 4 个空格或是一个 tab 就可以了，而 &、< 和 > 也一样会自动转成 HTML 实体。

    If you want your page to validate under XHTML 1.0 Strict,you've got to put paragraph tags in your blockquotes:<blockquote><p>For example.</p></blockquote>