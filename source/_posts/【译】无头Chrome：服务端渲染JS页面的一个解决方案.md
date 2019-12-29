---
title: 【译】无头 Chrome：服务端渲染 JS 页面的一个解决方案
date: 2018-09-17 21:05:57
tags:
- 翻译
- Puppeteer
- Chrome
categories:
- 前端
---

### TL;DR


> [无头 Chrome](https://developers.google.com/web/updates/2017/04/headless-chrome) 是一个将动态 JS 页面转成静态 HTML 页面的即插即用的解决方案。将其运行于 web 服务器之上，你可以**预渲染任何现代 JS 特性**，从而提速内容加载，并且是**可被搜索引擎索引的**。


本篇文章介绍的技术，旨在教大家如何使用 [Puppeteer](https://developers.google.com/web/tools/puppeteer/) 的 API，给一个 Express 服务器添加服务端渲染（SSR）能力。最棒的地方是，**应用本身几乎不需要修改任何代码**。无头 Chrome 做了所有的重活。三两行代码，SSR 页面带回家。

大餐之前先来点甜点：

```
import puppeteer from 'puppeteer';

async function ssr(url) {
  const browser = await puppeteer.launch({headless: true});
  const page = await browser.newPage();
  await page.goto(url, {waitUntil: 'networkidle0'});
  const html = await page.content(); // serialized HTML of page DOM.
  await browser.close();
  return html;
}
```

**注意：** 我会在文章中使用 ES 模块（`import`），这要求 Node 8.5.0+，并在运行时加上 `--experimental-modules` 标志。觉得麻烦的话可以自行使用 `require()` 语句。关于 Node 上的 ES 模块支持可以读读[这篇文章](https://nodejs.org/api/esm.html)。


<br>
## 导论
----------------------------

如果我对 [SEO](https://en.wikipedia.org/wiki/Search_engine_optimization) 理解没有偏差的话，你读到这篇文章可能因为下面两个原因之一。首先，你已经搭建了一个 web 应用，并且它没有被搜索引擎索引！你的应用可能是 SPA，[PWA](https://developers.google.com/web/progressive-web-apps/)，使用了 vanilla JS，或者使用了其他更复杂的框架或类库。老实说，你使用何种技术并不重要。重要的是，你花费了大量时间搭建出优秀的 web 页面，然而用户却搜不到它。你读这篇文章的另一个理由可能是因为，网上一些文章说了服务端渲染可以提升性能。你希望快速减少[ JavaScript 启动时间](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/javascript-startup-optimization/)，提升[首次有效绘制](https://developers.google.com/web/tools/lighthouse/audits/first-meaningful-paint)速度。

一些框架，比如 Preact 使用了[工具](https://github.com/developit/preact-render-to-string)来实现服务端渲染。如果你使用的框架具备预渲染的解决方案，请继续使用。没有任何理由引入另一个工具（无头 Chrome / Puppeteer）。


### 爬取现代网站


搜索引擎爬虫，社交平台，[甚至浏览器](https://en.wikipedia.org/wiki/Lynx_(web_browser))自诞生至今就唯一依赖于静态 HTML 标记，来索引 web 页面和表层内容。现代 web 页面已经演变的大为不同。基于 JavaScript 的应用，在很多时候，需要保持网站内容是对于爬取工具是可见的。

一些爬虫，比如 Google 搜索，已经变得更智能了！Google 的爬虫使用 Chrome 41 [执行 JavaScript](https://developers.google.com/search/docs/guides/rendering)，并渲染出最终的页面。但是这个方案才刚出来，还不完美。举个例子，使用了新特性的页面，比如 ES6 Class，[模块](https://www.chromestatus.com/feature/5365692190687232)，箭头函数等，将会在这个比较老的浏览器上报错，使得页面不能正确渲染。至于其他搜索引擎，鬼知道它们在干嘛！？¯\_(ツ)_/¯


## 使用无头 Chrome 预渲染页面

--------------------------------------------------------

所有的爬虫程序都能够理解 HTML。我们要“解决”索引问题的话需要一个工具，它来执行 JS 生成 HTML。我不会告诉你现在已经有这样一个工具了！

1. 该工具可以运行所有类型的现代 JavaScript，并吐出静态 HTML。
2. 出现新特性时，该工具可以保持更新
3. 已有应用上只需少量代码就可以运行这个工具


听起来很不错吧？**这个工具就是浏览器**！

无头 Chrome 不在乎你使用什么库、框架或者工具。它将 JavaScript 作为早餐，在午饭前吐出静态 HTML。可能会更快一点 :) -Eric

如果你用的 Node，Puppeteer 容易上手。它的 API 提供了预渲染客户端应用的能力。下面用个例子演示下。

### 1. JS 应用示例

我们以一个 JavaScript 生成 HTML 的动态页面为例：

**public/index.html**

```
<html>
<body>
  <div id="container">
    <!-- Populated by the JS below. -->
  </div>
</body>
<script>
function renderPosts(posts, container) {
  const html = posts.reduce((html, post) => {
    return `${html}
      <li class="post">
        <h2>${post.title}</h2>
        <div class="summary">${post.summary}</div>
        <p>${post.content}</p>
      </li>`;
  }, '');

  // CAREFUL: assumes html is sanitized.
  container.innerHTML = `<ul id="posts">${html}</ul>`;
}

(async() => {
  const container = document.querySelector('#container');
  const posts = await fetch('/posts').then(resp => resp.json());
  renderPosts(posts, container);
})();
</script>
</html>
```

### 2. 服务端渲染函数

接下来，我们会使用之前提到的 `ssr()` 函数，并充实它的内容。

**ssr.mjs**

```
import puppeteer from 'puppeteer';

// In-memory cache of rendered pages. Note: this will be cleared whenever the
// server process stops. If you need true persistence, use something like
// Google Cloud Storage (https://firebase.google.com/docs/storage/web/start).
const RENDER_CACHE = new Map();

async function ssr(url) {
  if (RENDER_CACHE.has(url)) {
    return {html: RENDER_CACHE.get(url), ttRenderMs: 0};
  }

  const start = Date.now();

  const browser = await puppeteer.launch();
  const page = await browser.newPage();
  try {
    // networkidle0 waits for the network to be idle (no requests for 500ms).
    // The page's JS has likely produced markup by this point, but wait longer
    // if your site lazy loads, etc.
    await page.goto(url, {waitUntil: 'networkidle0'});
    await page.waitForSelector('#posts'); // ensure #posts exists in the DOM.
  } catch (err) {
    console.error(err);
    throw new Error('page.goto/waitForSelector timed out.');
  }

  const html = await page.content(); // serialized HTML of page DOM.
  await browser.close();

  const ttRenderMs = Date.now() - start;
  console.info(`Headless rendered page in: ${ttRenderMs}ms`);

  RENDER_CACHE.set(url, html); // cache rendered page.

  return {html, ttRenderMs};
}

export {ssr as default};
```

主要的变化：

1. 添加了缓存。缓存已渲染的 HTML 对于加速响应时间居功至伟。当页面再次有请求过来，避免了无头 Chrome 的重复执行。我随后会讨论其他的[优化](#optimizations) 。
2. 添加加载页面超时时的基本错误处理。
3. 添加了 `page.waitForSelector('#posts')` 这行代码。确保在丢弃这个序列化页面之前，posts 节点存在于 DOM 之中。
4. 记录无头浏览器渲染页面所用时间。
5. 代码都被封装进名为 `ssr.mjs` 的模块中。


### 3. web 服务器示例


最后，一个小的 express 服务器完成了所有的工作。它预渲染 URL `http://localhost/index.html`（主页），并在响应中返回渲染结果。由于响应中包含了静态 HTML， 当用户访问页面，posts 节点会立刻呈现。

**server.mjs**

```
import express from 'express';
import ssr from './ssr.mjs';

const app = express();

app.get('/', async (req, res, next) => {
  const {html, ttRenderMs} = await ssr(`${req.protocol}://${req.get('host')}/index.html`);
  // Add Server-Timing! See https://w3c.github.io/server-timing/.
  res.set('Server-Timing', `Prerender;dur=${ttRenderMs};desc="Headless render time (ms)"`);
  return res.status(200).send(html); // Serve prerendered page as response.
});

app.listen(8080, () => console.log('Server started. Press Ctrl+C to quit')); 
```


要运行这个例子，需安装依赖 (`npm i --save puppeteer express`)，然后使用 Node 8.5.0+ 并带有 `--experimental-modules` 标志来运行服务器。

这是一个该服务器返回的响应示例：

```
<html>
<body>
  <div id="container">
    <ul id="posts">
      <li class="post">
        <h2>Title 1</h2>
        <div class="summary">Summary 1</div>
        <p>post content 1</p>
      </li>
      <li class="post">
        <h2>Title 2</h2>
        <div class="summary">Summary 2</div>
        <p>post content 2</p>
      </li>
      ...
    </ul>
  </div>
</body>
<script>
...
</script>
</html>
```


#### Server-Timing API 的一个最佳用例


[Server-Timing](https://w3c.github.io/server-timing/) API 支持将服务器性能指标（比如请求/响应时间，数据库查询）返回给浏览器。客户端可以使用这些信息来追踪 web 应用的所有性能数据。


Server-Timing 的一个最佳用例是上报无头 Chrome 预渲染页面的时间！只需在响应上添加 `Server-Timing` 头，就可以实现这一点：

```
res.set('Server-Timing',  `Prerender;dur=1000;desc="Headless render time (ms)"`);  
```


客户端上，[Performance Timeline API](https://developer.mozilla.org/en-US/docs/Web/API/Performance_Timeline) 和 [PerformanceObserver](https://developer.mozilla.org/en-US/docs/Web/API/PerformanceObserver) 可以获取这些指标：


```
  const entry = performance.getEntriesByType('navigation').find(
    e => e.name === location.href);
console.log(entry.serverTiming[0].toJSON());
```

```
{
  "name": "Prerender",
  "duration": 3808,
  "description": "Headless render time (ms)"
}
```


### 性能结果


**注意：** 这些数据体现了我随后讨论的大多数性能[优化](#optimizations)。


性能数据怎么样？在我的一个[应用](https://devwebfeed.appspot.com/ssr)([代码](https://github.com/ebidel/devwebfeed/blob/master/server.mjs))上，无头 Chrome 渲染页面大约需要 1s。页面被缓存后， **3G 低网速模拟**下，[FCP](https://developers.google.com/web/fundamentals/performance/user-centric-performance-metrics) 要比客户端渲染版本的**快 8.37s**。


 &nbsp; | **首次绘制 (FP)** | **首次内容绘制 (FCP)** |
---- | ---- | ---- |
客户端渲染|4s|11s|
服务端渲染|2.3s|~2.3s|


这些结果很有用。因为服务端渲染页面**不再依赖于 JavaScript 的加载**，用户看到有意义的内容比以前快得多。

<br>

## Preventing re-hydration

---------------------------------------

还记得我说“我们无需在客户端应用上改任何代码”吗？那是骗你们的。

Express 应用接收请求，使用 Puppeteer 将页面加载进无头浏览器，然后在响应中返回结果。但这里有一个问题。

浏览器加载页面时，**无头 Chrome 中相同的 JS** 会在服务器上**再次执行**。有两处都在生成 HTML。 

一起来修复这个问题。我们要告知页面，它的 HTML 早就名花有主了。我找到的解决方案是，在页面加载时判断 `<ul id="posts">` 是否已在 DOM 中，如果在，页面就已经在服务端渲染过了，这样就可以避免重新创建 DOM。

**public/index.html**

```
<html>
<body>
  <div id="container">
    <!-- Populated by JS (below) or by prerendering (server). Either way,
         #container gets populated with the posts markup:
      <ul id="posts">...</ul>
    -->
  </div>
</body>
<script>
...
(async() => {
  const container = document.querySelector('#container');

  // Posts markup is already in DOM if we're seeing a SSR'd.
  // Don't re-hydrate the posts here on the client.
  const PRE_RENDERED = container.querySelector('#posts');
  if (!PRE_RENDERED) {
    const posts = await fetch('/posts').then(resp => resp.json());
    renderPosts(posts, container);
  }
})();
</script>
</html>
```
<br>

## 优化
-----------------------------

除了缓存渲染结果之外，还有一些有趣的优化技巧。有的优化可以快速见效，而有的可能带有猜测性的。

### 中止不必要的请求

现在，整个页面（以及它请求的所有资源）都无脑地加载进无头 Chrome。然而，我们只关注于两件事情：

1. 渲染 HTML
2. 生成 HTML 的 JS


**不构造 DOM 的网络请求是浪费的**。一些资源，比如图片、字体、样式表和媒体内容，不参与页面的 HTML 构建。它们负责添加样式，补充页面的结构，但并不显式地创建页面。我们应该告诉浏览器去忽略掉这些资源！这样可以减少无头 Chrome 的工作负担，从而**节省带宽**，并且潜在地**加速了大型页面的预渲染时间**。

[Protocol 开发者工具](https://chromedevtools.github.io/devtools-protocol/)提供了一个强大的特性，叫做[网络拦截](https://chromedevtools.github.io/devtools-protocol/tot/Network#event-requestIntercepted)。它可以用于**在浏览器发出之前修改请求**。Puppeteer 也支持网络拦截，它是通过打开 [`page.setRequestInterception(true)`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagesetrequestinterceptionvalue)，监听[页面的 `request` 事件](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-request)来实现的。这样我们可以中止某些资源请求。

**ssr.mjs**

```
async function ssr(url) {
  ...
  const page = await browser.newPage();

  // 1. Intercept network requests.
  await page.setRequestInterception(true);

  page.on('request', req => {
    // 2. Ignore requests for resources that don't produce DOM
    // (images, stylesheets, media).
    const whitelist = ['document', 'script', 'xhr', 'fetch'];
    if (!whitelist.includes(req.resourceType())) {
      return req.abort();
    }

    // 3. Pass through all other requests.
    req.continue();
  });

  await page.goto(url, {waitUntil: 'networkidle0'});
  const html = await page.content(); // serialized HTML of page DOM.
  await browser.close();

  return {html};
}
```

**注意：** 安全起见，我使用了一个白名单，允许所有其他类型的请求能够继续正常发出。预先避免中止掉其他必要的请求。


### 内联关键资源

使用构建工具（比如 `gulp`）编译应用，并在构建时将关键 CSS/JS 内联到页面内，是一种很常见的做法。由于浏览器初始化页面加载时的请求数更少了，这样也就加速了首次有效绘制时间。

别用构建工具了，**浏览器就是你的构建工具**！我们可以用 Puppeteer 管理页面 DOM，内联样式，JavaScript， 或者其他任何你想在预渲染之前加到页面中的东西。

这个例子演示了如何拦截本地样式表的响应，并将这些资源内联进 `<style>` 标签中：

**ssr.mjs**

```
import urlModule from 'url';
const URL = urlModule.URL;

async function ssr(url) {
  ...
  const stylesheetContents = {};

  // 1. Stash the responses of local stylesheets.
  page.on('response', async resp => {
    const responseUrl = resp.url();
    const sameOrigin = new URL(responseUrl).origin === new URL(url).origin;
    const isStylesheet = resp.request().resourceType() === 'stylesheet';
    if (sameOrigin && isStylesheet) {
      stylesheetContents[responseUrl] = await resp.text();
    }
  });

  // 2. Load page as normal, waiting for network requests to be idle.
  await page.goto(url, {waitUntil: 'networkidle0'});

  // 3. Inline the CSS.
  // Replace stylesheets in the page with their equivalent <style>.
  await page.$$eval('link[rel="stylesheet"]', (links, content) => {
    links.forEach(link => {
      const cssText = content[link.href];
      if (cssText) {
        const style = document.createElement('style');
        style.textContent = cssText;
        link.replaceWith(style);
      }
    });
  }, stylesheetContents);

  // 4. Get updated serialized HTML of page.
  const html = await page.content();
  await browser.close();

  return {html};
}
```


这段代码：

1. 使用一个 [`page.on('response')`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#event-response) 处理器来监听网络响应。
2. 储藏本地样式表的响应。
3. 找到 DOM 中所有的 `<link rel="stylesheet">`，将它们替换成一个等价的 `<style>`。具体见 [`page.$$eval`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevalselector-pagefunction-args) API 文档。`style.textContent` 被设为样式表的响应内容。


### 自动压缩资源

另一个可以借助网络拦截玩的小把戏是**修改请求的响应内容**。

举个例子，你想要压缩 CSS，但也希望开发阶段不要被压缩，这样开发时能方便些。假设你已经用另一个工具来预压缩 `styles.css`，可以用 `Request.respond()`，将 `styles.css` 的内容重写为 `styles.min.css`。

**ssr.mjs**

```
import fs from 'fs';

async function ssr(url) {
  ...

  // 1. Intercept network requests.
  await page.setRequestInterception(true);

  page.on('request', req => {
    // 2. If request is for styles.css, respond with the minified version.
    if (req.url().endsWith('styles.css')) {
      return req.respond({
        status: 200,
        contentType: 'text/css',
        body: fs.readFileSync('./public/styles.min.css', 'utf-8')
      });
    }
    ...

    req.continue();
  });
  ...

  const html = await page.content();
  await browser.close();

  return {html};
}
```


### 重用 Chrome 实例实现交叉渲染


每次预渲染都启动新的浏览器会很浪费。相反，你希望只启动一个实例，然后在多个页面渲染时重用它。

Puppeteer 可以通过调用 [`puppeteer.connect()`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#puppeteerconnectoptions)，连接到一个已有的 Chrome 实例，它接收实例的远程调试 URL 作为参数。为保证浏览器实例的长时间运行，我们可以将 `ssr()` 函数启动 Chrome 这部分代码移到 Express 服务器里。

**server.mjs**

```
import express from 'express';
import puppeteer from 'puppeteer';
import ssr from './ssr.mjs';

let browserWSEndpoint = null;
const app = express();

app.get('/', async (req, res, next) => {
  if (!browserWSEndpoint) {
    const browser = await puppeteer.launch();
    browserWSEndpoint = await browser.wsEndpoint();
  }

  const url = `${req.protocol}://${req.get('host')}/index.html`;
  const {html} = await ssr(url, browserWSEndpoint);

  return res.status(200).send(html);
});  
```

**ssr.mjs**

```
import puppeteer from 'puppeteer';

/**
 * @param {string} url URL to prerender.
 * @param {string} browserWSEndpoint Optional remote debugging URL. If
 *     provided, Puppeteer's reconnects to the browser instance. Otherwise,
 *     a new browser instance is launched.
 */
async function ssr(url, browserWSEndpoint) {
  ...
  console.info('Connecting to existing Chrome instance.');
  const browser = await puppeteer.connect({browserWSEndpoint});

  const page = await browser.newPage();
  ...
  await page.close(); // Close the page we opened here (not the browser).

  return {html};
}
```


#### 例子：实现周期性预渲染的定时任务


在 [App 引擎面板应用](https://devwebfeed.appspot.com/ssr) 里，我创建了一个[定时处理器](https://cloud.google.com/appengine/docs/flexible/nodejs/scheduling-jobs-with-cron-yaml)，来周期性的重复渲染排名前几位的页面。帮助用户快速看到最新内容，他们根本感知不到一个新页面的启动性能消耗。在这个例子中，生成多个 Chrome 实例会很浪费。相反，我用了一个共享的浏览器实例来一次性渲染这些页面。

```
import puppeteer from 'puppeteer';
import * as prerender from './ssr.mjs';
import urlModule from 'url';
const URL = urlModule.URL;

app.get('/cron/update_cache', async (req, res) => {
  if (!req.get('X-Appengine-Cron')) {
    return res.status(403).send('Sorry, cron handler can only be run as admin.');
  }

  const browser = await puppeteer.launch();
  const homepage = new URL(`${req.protocol}://${req.get('host')}`);

  // Re-render main page and a few pages back.
  prerender.clearCache();
  await prerender.ssr(homepage.href, await browser.wsEndpoint());
  await prerender.ssr(`${homepage}?year=2018`);
  await prerender.ssr(`${homepage}?year=2017`);
  await prerender.ssr(`${homepage}?year=2016`);
  await browser.close();

  res.status(200).send('Render cache updated!');
});
```

我还在 **ssr.js** export 上加了一个 `clearCache()` 函数。

```
...
function clearCache() {
  RENDER_CACHE.clear();
}

export {ssr, clearCache};
```

<br>
## 其他因素
------------------------------------

### 告诉页面：“你正在被无头浏览器渲染”


当页面正在服务器上的无头 Chrome 中渲染时，客户端逻辑很有必要知道这一信息。我的应用使用了钩子来“关闭”部分不参与渲染 post 节点的页面。举例来说，我禁用了懒加载 [firebase-auth.js](https://firebase.google.com/docs/web/setup) 这部分代码。根本不需要用户登录！

在 URL 上加一个 `?headless` 参数，是一个给页面加钩子的简单方法：

**ssr.mjs**

```
import urlModule from 'url';
const URL = urlModule.URL;

async function ssr(url) {
  ...
  // Add ?headless to the URL so the page has a signal
  // it's being loaded by headless Chrome.
  const renderUrl = new URL(url);
  renderUrl.searchParams.set('headless', '');
  await page.goto(renderUrl, {waitUntil: 'networkidle0'});
  ...

  return {html};
}
```

可以在页面内查询该参数：

**public/index.html**

```
<html>
<body>
  <div id="container">
    <!-- Populated by the JS below. -->
  </div>
</body>
<script>
...

(async() => {
  const params = new URL(location.href).searchParams;

  const RENDERING_IN_HEADLESS = params.has('headless');
  if (RENDERING_IN_HEADLESS) {
    // Being rendered by headless Chrome on the server.
    // e.g. shut off features, don't lazy load non-essential resources, etc.
  }

  const container = document.querySelector('#container');
  const posts = await fetch('/posts').then(resp => resp.json());
  renderPosts(posts, container);
})();
</script>
</html> 
```


Tip：[`Page.evaluateOnNewDocument()`](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluateonnewdocumentpagefunction-args) 也可以方便的查询参数。它会在页面中注入代码，让 Puppeteer 在页面中剩余待执行的 JavaScript 之前运行这些代码。


### 避免 PV 膨胀


你如果正在页面上使用分析工具，那么要小心了。预渲染的页面可能会造成 PV 出现膨胀。具体来说，**打点数据将会提升2倍**，一半是在无头 Chrome 渲染时，另一半出现在用户浏览器渲染时。

那么怎么修复这个问题呢？将所有加载分析脚本的请求拦截掉。

```
page.on('request', req => {
  // Don't load Google Analytics lib requests so pageviews aren't 2x.
  const blacklist = ['www.google-analytics.com', '/gtag/js', 'ga.js', 'analytics.js'];
  if (blacklist.find(regex => req.url().match(regex))) {
    return req.abort();
  }
  ...
  req.continue();
});
```


代码不加载，页面访问就不会被记录。真 Skr 个机灵鬼 💥。

或者，你也可以继续加载分析脚本，来获悉服务器上运行的预渲染器数。

<br>

## 结论
--------------------------

Puppeteer 通过运行无头 Chrome，不费吹灰之力就实现了服务端渲染。**提升加载性能**和**没有改动大量代码就增强了应用的可索引性**，是这个方案中我最喜欢的“特性”。

**注意：** 如果你对文章中描述的技术感兴趣，可以去看看[这个应用](https://devwebfeed.appspot.com/ssr)，以及[它的代码](https://github.com/ebidel/devwebfeed/blob/master/server.mjs)。

<br>

## 附录
------------------------

### 现有技术的讨论

很难在服务端上渲染客户端应用。有多难？去看看大家给这个话题奉献了多少个 [npm 包](https://www.npmjs.com/search?q=server%20side%20rendering)就知道了。有数不清的[模式](https://en.wikipedia.org/wiki/Isomorphic_JavaScript)，[工具](https://github.com/GoogleChrome/rendertron)，和[服务](https://prerender.io/)来辅助服务端渲染的 JS 应用。

#### 同构 JavaScript

同构 JavaScript 的概念很简单：同样的代码既能在服务端运行，也能在客户端（浏览器）运行。服务器和客户端共享代码，美滋滋！

实践中，我发现同构 JS 很难实现。这是我自己的问题...

> 我最近开始做一个[项目](https://github.com/ebidel/devwebfeed/blob/master/server.mjs)，尝试下 [lit-html](https://github.com/Polymer/lit-html)。Lit 是一个优秀的库，它可以允许你写使用 JS 模板字符串写 [HTML \<template>](https://www.html5rocks.com/en/tutorials/webcomponents/template/)，然后高效地将这些模板渲染为 DOM。问题是它的核心特性（使用 `<template>` 元素）只能在浏览器上工作。这意味着它在 Node 服务器上不能运行。我希望 Node 和前端共享的 SSR 代码能够脱离 window 对象。

> 最后我意识到可以使用无头 Chrome 来服务端渲染应用，Chrome 是经用户的手运行或是在服务器上自动运行并不重要，它反正是愉快地执行了所有 JS。不要多问。 

无头 Chrome 在服务器和客户端上启用 “同构 JS”。它对于当前库不支持服务端（Node）给出了一个不错的解决方案。


#### 预渲染工具

Node 社区已经诞生了好几吨解决服务端渲染 JS 应用的工具。毫无新意！个人而言，我发现各人对于这些工具的体会可能不同，所以使用这些工具前肯定要做好功课。比如说，一些服务端渲染工具比较老，并且没有使用无头 Chrome（或者任何其他无头浏览器）。相反，它们使用 PhantomJS（又名旧 Safari），这意味着使用新特性时页面不会正确渲染。

一个值得注意的例外是 [Prerender](https://github.com/prerender/prerender/)。Prerender 使用了无头 Chrome 和 [Express 中间件](https://github.com/prerender/prerender-node)。

```
const prerender =  require('prerender');  
const server = prerender();  
server.use(prerender.removeScriptTags());  
server.use(prerender.blockResources());  
server.start();  
```

Prerender 省去了跨平台下载和安装 Chrome 的所有细节。要正确完成这一过程通常是相当棘手的，这也是 [Puppeteer](https://developers.google.com/web/tools/puppeteer/faq#q_which_chromium_version_does_puppeteer_use) 存在的原因之一。我也提了一些渲染我的部分应用的 issue。


![浏览器中渲染的 Chrome 状态](/uploads/Puppeteer/1.png)



![prerender 渲染的 Chrome 状态](/uploads/Puppeteer/2.png)


