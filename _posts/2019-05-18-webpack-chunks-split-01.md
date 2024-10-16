---
layout: post
title: 深入理解 Webpack 打包分块（上）
modified: 2019-05-18
tags: [javascript, webpack]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 前言

随着前端代码需要处理的业务越来越繁重，我们不得不面临的一个问题是前端的代码体积也变得越来越庞大。这造成无论是在调式还是在上线时都需要花长时间等待编译完成，并且用户也不得不花额外的时间和带宽下载更大体积的脚本文件。

然而仔细想想这完全是可以避免的：在开发时难道一行代码的修改也要重新打包整个脚本？用户只是粗略浏览页面也需要将整个站点的脚本全部下载下来？所以趋势必然是按需的、有策略性的将代码拆分和提供给用户。最近流行的微前端某种意义上来说也是遵循了这样的原则（但也并不是完全基于这样的原因）

幸运的是，我们目前已有的工具已经完全赋予我们实现以上需求的能力。例如 Webpack 允许我们在打包时将脚本分块；利用浏览器缓存我们能够有的放矢的加载资源。

在探寻最佳实践的过程中，最让我疑惑的不是我们能不能做，而是我们应该如何做：我们因该采取什么样的特征拆分脚本？我们应该使用什么样的缓存策略？使用懒加载和分块是否有异曲同工之妙？拆分之后究竟能带来多大的性能提升？最重要的是，在面多诸多的方案和工具以及不确定的因素时，我们应该如何开始？这篇文章就是对以上问题的梳理和回答。文章的内容大体分为两个方面，一方面在思路制定模块分离的策略，另一方面从技术上对方案进行落地。

本文的主要内容翻译自 [The 100% correct way to split your chunks with Webpack](https://hackernoon.com/the-100-correct-way-to-split-your-chunks-with-webpack-f8a9df5b7758)。 这篇文章循序渐进的引导开发者步步为营的对代码进行拆分优化，所以它是作为本文的线索存在。同时在它的基础上，我会对 Webpack 及其他的知识点做纵向扩展，对方案进行落地。

以下开始正文

---

根据 [Webpack 术语表](https://webpack.js.org/glossary/)，存在两类文件的分离。这些名词听起来是可以互换的，但实际上不行：

- 打包分离 (Bundle splitting)：为了更好的缓存创建更多、更小的文件（但仍然以每一个文件一个请求的方式进行加载）
- 代码分离 (Code splitting)：动态加载代码，所以用户只需要下载当前他正在浏览站点的这部分代码

第二种策略听起来更吸引人是不是？事实上许多的文章也假定认为这才是唯一值得将 JavaScript 文件进行小文件拆分的场景。

但是我在这里告诉你第一种策略对许多的站点来说才更有价值，并且应该是你首先为页面做的事

让我们来深入理解

## Bundle VS Chunk VS Module

在正式开始编码之前，我们还是要明确一些概念。例如我们贯穿全文的“块”(chunk) ，以及它和我们常常提到的“包”(bundle)以及“模块”(module) 到底有什么区别。

遗憾的事情是即使在查阅了很多资料之后，我仍然没法得到一个确切的标准答案，所以这里我选择我个人比较认可的定义在这里做一个分享，重要的还是希望能起到统一口径的作用

首先对于“模块”(module)的概念相信大家都没有异议，它指的就是我们在编码过程中有意识的封装和组织起来的代码片段。狭义上我们首先联想到的是碎片化的 React 组件，或者是 CommonJS 模块又或者是 ES6 模块，但是对 Webpack 和 Loader 而言，广义上的模块还包括样式和图片，甚至说是不同类型的文件

而“包”(bundle) 就是把相关代码都打包进入的单个文件。如果你不想把所有的代码都放入一个包中，你可以把它们划分为多个包，也就是“块”(chunk) 中。从这个角度上看，“块”等于“包”，它们都是对代码再一层的组织和封装。如果必须要给一个区分的话，通常我们在讨论时，bundle 指的是所有模块都打包进入的单个文件，而 chunk 指的是按照某种规则的模块集合，chunk 的体积大于单个模块，同时小于整个 bundle

（但如果要仔细的深究，**Chunk**是 Webpack 用于管理打包流程中的技术术语，甚至能划分为不同类型的 chunk。我想我们不用从这个角度理解。只需要记住上一段的定义即可）

## 打包分离 (Bundle splitting)

打包分离背后的思想非常简单。如果你有一个体积巨大的文件，并且只改了一行代码，用户仍然需要重新下载整个文件。但是如果你把它分为了两个文件，那么用户只需要下载那个被修改的文件，而浏览器则可以从缓存中加载另一个文件。

值得注意的是因为打包分离与缓存相关，所以对站点的首次访问者来说没有区别

（我认为太多的性能讨论都是关于站点的首次访问。或许部分原因是因为第一映像很重要，另一部分因为这部分性能测量起来简单和讨巧）

当谈论到频繁访问者时，量化性能提升会稍有棘手，但是我们必须量化！

这将需要一张表格，我们将每一种场景与每一种策略的组合结果都记录下来

我们假设一个场景：

- Alice 连续 10 周每周访问站点一次
- 我们每周更新站点一次
- 我们每周更新“产品列表”页面
- 我们也有一个“产品详情”页面，但是目前不需要对它进行更新
- 在第 5 周的时我们给站点新增了一个 npm 包
- 在第 8 周时我们更新了现有的一个 npm 包

当然包括我在内的部分人希望场景尽可能的逼真。但其实无关紧要，我们随后会解释为什么。

## 性能基线

假设我们的 JavaScript 打包后的总体积为 400KB, 将它命名为 `main.js`，然后以单文件的形式加载它

我们有一个类似如下的 Webpack 配置（我已经移除了无关的配置项）：

```javascript
const path = require('path');

module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
};
```
当只有单个入口时，Webpack 会自动把结果命名为`main.js`

（对那些刚接触缓知识的人我解释一下：每当我我提及`main.js`的时候，我实际上是在说类似于`main.xMePWxHo.js`这种包含一堆带有文件内容哈希字符串的东西。这意味着当你应用代码发生更改时新的文件名会生成，这样就能迫使浏览器下载新的文件）

所以当每周我向站点发布新的变更时，包的`contenthash`就会发生更改。以至于每周 Alice 访问我们站点时不得不下载一个全新的 400KB 大小的文件

![](../images/webpack-chunk-split/001.png)

连续十周**也就是 4.12MB**

我们能做的更好

### 哈希（hash）与性能

不知道你是否真的理解上面的表述。有几点需要在这里澄清：

1. 为什么带哈希串的文件名会对浏览器缓存产生影响？
2. 为什么文件名里的哈希后缀是`contenthash`？如果把`contenthash`替换成`hash`或者`chunkhash`有什么影响？

为了每次访问时不让浏览器都重新下载同一个文件，我们通常会把这个文件返回的 HTTP 头中的`Cache-Control`设置为`max-age=31536000`，也就是一年（秒数的）时间。这样以来，在一年之内用户访问这个文件时，都不会再次向服务器发送请求而是直接从缓存中读取，直到或者手动清除了缓存。

如果我中途修改了文件内容必须让用户重新下载怎么办？修改文件名就好了，不同的文件（名）对应不同的缓存策略。而一个哈希字符串就是根据文件内容产生的“签名”，每当文件内容发生更改时，哈希串也就发生了更改，文件名也就随之更改。这样一来，旧版本文件的缓存策略就会失效，浏览器就会重新加载新版本的该文件。当然这只是其中一种最基础的缓存策略，更复杂的场景请参考我之前的一篇文章：[设计一个无懈可击的浏览器缓存方案：关于思路，细节，ServiceWorker，以及HTTP/2](https://zhuanlan.zhihu.com/p/28113197)

所以在 Webpack 中配置的 `filename: [name]:[contenthash].js` 就是为了每次发布时自动生成新的文件名。

然而如果你对 Webpack 稍有了解的话，你应该知道 Webpack 还提供了另外两种哈希算法供开发者使用：`hash`和`chunkhash`。那么为什么不使用它们而是使用`contenthash`？这要从它们的区别说起。原则上来说，它们是为不同目的服务的，但在实际操作中，也可以交替使用。

为了便于说明，我们先准备以下这段非常简单的 Webpack 配置，它拥有两个打包入口，同时额外提取出 css 文件，最终生成三个文件。`filename`配置中我们使用的是`hash`标识符、在 `MinCssExtractPlugin`中我们使用的是`contenthash`，为什么会这样稍后会解释。

```javascript
const CleanWebpackPlugin = require("clean-webpack-plugin");
const MiniCssExtractPlugin = require("mini-css-extract-plugin");

module.exports = {
  entry: {
    module_a: "./src/module_a.js",
    module_b: "./src/module_b.js"
  },
  output: {
    filename: "[name].[hash].js"
  },
  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].[contenthash].css"
    })
  ]
};

```

**`hash`**

`hash`针对的是每一次构建（build）而言，每一次构建之后生成的文件所带的哈希都是一致的。它关心的是整体项目的变化，只要有任意文件内容发生了更改，那么构建之后其他文件的哈希也会发生更改。

很显然这不是我们需要的，如果`module_a`文件内容发生了更改，`module_a`的打包文件的哈希应该发生变化，但是`module_b`不应该。这会导致用户不得不重新下载没有发生变化的`module_b`打包文件

**`chunkhash`**

`chunkhash`基于的是每一个 chunk 内容的改变，如果是该 chunk 所属的内容发生了变化，那么只有该 chunk 的输出文件的哈希会发生变化，其它的不会。这听上去符合我们的需求。

在之前我们对 chunk 进行过定义，即是小单位的代码聚合形式。在上面的例子中以`entry`入口体现，也就是说每一个入口对应的文件就是一个 chunk。在后面的例子中我们会看到更复杂的例子

- `contenthash`

顾名思义，该哈希根据的是文件的内容。从这个角度上说，它和`chunkhash`是能够相互代替的。所以在“性能基线”代码中作者使用了`contenthash`

不过特殊之处是，或者说我读到的关于它的使用说明中，都指示如果你想在`ExtractTextWebpackPlugin`或者`MiniCssExtractPlugin`中用到哈希标识，你应该使用`contenthash`。但就我个人的测试而言，使用`hash`或者`chunkhash`也都没有问题（也许是因为 extract 插件是严格基于 content 的？但难道 chunk 不是吗？）


## 分离第三方类库（vendor）类库

让我们把打包文件划分为`main.js`和`vendor.js`

很简单，类似于：

```javascript
const path = require('path');

module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
  optimization: {
    splitChunks: {
      chunks: 'all',
    },
  },
};
```

在你没有告诉它你想如何拆分打包文件的情况下， Webpack 4 在尽它最大的努力把这件事做的最好

这就导致一些声音在说：“太惊人了，Webpack 做的真不错！”

而另一些声音在说：“你对我的打包文件做了什么！”

无论如何，添加`optimization.splitChunks.chunks = 'all'`配置也就是在说：“把所有`node_modules`里的东西都放到`vendors~main.js`的文件中去”

在实现基本的打包分离条件后，Alice 在每次访问时仍然需要下载 200KB 大小的 `main.js` 文件， 但是只需要在第一周、第五周、第八周下载 200KB 的 `vendors.js`脚本

![](../images/webpack-chunk-split/002.png)

**也就是 2.64MB**

体积减少了 36%。对于配置里新增的五行代码来说结果还不错。在继续阅读之前你可以立刻就去试试。如果你需要将 Webpack 3 升级到 4，也不要着急，升级不会带来痛苦（而且是免费的！）

## 分离每一个 npm 包

我们的 `vendors.js` 承受着和开始 `main.js` 文件同样的问题——部分的修改会意味着重新下载所有的文件

所以为什么不把每一个 npm 包都分割为单独的文件？做起来非常简单

让我们把我们的`react`，`lodash`，`redux`，`moment`等分离为不同的文件

```javascript

const path = require('path');
const webpack = require('webpack');

module.exports = {
  entry: path.resolve(__dirname, 'src/index.js'),
  plugins: [
    new webpack.HashedModuleIdsPlugin(), // so that file hashes don't change unexpectedly
  ],
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash].js',
  },
  optimization: {
    runtimeChunk: 'single',
    splitChunks: {
      chunks: 'all',
      maxInitialRequests: Infinity,
      minSize: 0,
      cacheGroups: {
        vendor: {
          test: /[\\/]node_modules[\\/]/,
          name(module) {
            // get the name. E.g. node_modules/packageName/not/this/part.js
            // or node_modules/packageName
            const packageName = module.context.match(/[\\/]node_modules[\\/](.*?)([\\/]|$)/)[1];

            // npm package names are URL-safe, but some servers don't like @ symbols
            return `npm.${packageName.replace('@', '')}`;
          },
        },
      },
    },
  },
};
```

[这份文档](https://webpack.js.org/guides/caching/) 非常好的解释了这里做的事情，但是我仍然需要解释一下其中精妙的部分，因为它们花了我相当长的时间才搞明白

- Webpack 有一些不那么智能的默认“智能”配置，比如当分离打包输出文件时只允许最多3个文件，并且最小文件的尺寸是30KB（如果存在更小的文件就把它们拼接起来）。所以我把这些配置都覆盖了
- `cacheGroups`是我们用来制定规则告诉 Webpack 应该如何组织 chunks 到打包输出文件的地方。我在这里对所有加载自`node_modules`里的 module 制定了一条名为 "vendor" 的规则。通常情况下，你只需要为你的输出文件的 `name`定义一个字符串。但是我把`name`定义为了一个函数（当文件被解析时会被调用）。在函数中我会根据 module 的路径返回包的名称。结果就是，对于每一个包我都会得到一个单独的文件，比如`npm.react-dom.899sadfhj4.js`
- 为了能够正常发布[npm 包的名称必须是合法的URL](https://docs.npmjs.com/files/package.json#name)，所以我们不需要`encodeURI`对包的名词进行转义处理。但是我遇到一个问题是.NET服务器不会给名称中包含`@`的文件提供文件服务，所以我在代码片段中进行了替换
- 整个步骤的配置设置之后就不需要维护了——我们不需要使用名称引用任何的类库

Alice 每周都要重新下载 200KB 的 `main.js` 文件，并且再她首次访问时仍然需要下载 200KB 的 npm 包文件，但是她再也不用重复的下载同一个包两次

![](../images/webpack-chunk-split/003.png)

也就是**2.24MB**

相对于基线减少了 44%，这是一段你能够从文章里粘贴复制的非常酷的代码。

我好奇我们能超越 50%？

那不是很棒吗

### 稍等，那段 Webpack 配置代码究竟是怎么回事

此时你的疑惑可能是，optimization 选项里的配置怎么就把 vendor 代码分离出来了？

接下来的这一小节会针对 Webpack 的 Optimization 选项做讲解。我个人并非 Webpack 的专家，配置和对应的描述功能也并非一一经过验证，也并非全部都覆盖到，如果有纰漏的地方还请大家谅解。

`optimization`配置如其名所示，是为优化代码而生。如果你再仔细观察，大部分配置又在`splitChunk`字段下，因为它间接使用 SplitChunkPlugin 实现对块的拆分功能（这些都是在 Webpack 4 中引入的新的机制。在 Webpack 3 中使用的是 CommonsChunkPlugin，在 4 中已经不再使用了。所以这里我们也主要关注的是 SplitChunkPlugin 的配置）从整体上看，SplitChunksPlugin 的功能只有一个，就是**split**——把代码分离出来。分离是相对于把所有模块都打包成一个文件而言，把单个大文件分离为多个小文件。

在最初分离 vendor 代码时，我们只使用了一个配置 

```javascript
splitChunks: {
  chunks: 'all',
},
```

`chunks`有三个选项：`initial`、`async`和`all`。它指示应该优先分离同步（initial）、异步（async）还是所有的代码模块。这里的异步指的是通过动态加载方式（`import()`）加载的模块。

这里的重点是**优先**二字。以`async`为例，假如你有两个模块 a 和 b，两者都引用了 jQuery，但是 a 模块还通过动态加载的方式引入了 lodash。那么在 `async` 模式下，插件在打包时会分离出`lodash~for~a.js`的 chunk 模块，而 a 和 b 的公共模块 jQuery 并不会被（优化）分离出来，所以它可能还同时存在于打包后的`a.bundle.js`和`b.bundle.js`文件中。因为`async`告诉插件优先考虑的是动态加载的模块

接下来聚焦第二段分离每个 npm 包的 Webpack 配置中

`maxInitialRequests`和`minSize`确实就是插件自作多情的杰作了。插件自带一些分离 chunk 的规则：如果即将分离的 chunk 文件体积小于 30KB 的话，那么就不会将该 chunk 分离出来；并且限制并行下载的 chunk 最大请求个数为 3 个。通过覆盖 `minSize` 和 `maxInitialRequests` 配置就能够重写这两个参数。注意这里的`maxInitialRequests`和`minSize`是在`splitChunks`根目录中的，我们暂且称它为全局配置

`cacheGroups`配置才是最重要，它允许自定义规则分离 chunk。并且每条`cacheGroups`规则下都允许定义上面提到的`chunks`和`minSize`字段用于覆盖全局配置（又或者将`cacheGroups`规则中`enforce`参数设为`true`来忽略全局配置）

`cacheGroups`里默认自带`vendors`配置来分离`node_modules`里的类库模块，它的默认配置如下：

```javascript
cacheGroups: {
  vendors: {
    test: /[\\/]node_modules[\\/]/,
    priority: -10
  },
```

如果你不想使用它的配置，你可以把它设为`false`又或者重写它。这里我选择重写，并且加入了额外的配置`name`和`enforce`:

```javascript
vendors: {
  test: /[\\/]node_modules[\\/]/,
  name: 'vendors',
  enforce: true,
},
```
最后介绍以上并没有出现但是仍然常用的两个配置：`priority`和`reuseExistingChunk`

- `reuseExistingChunk`: 该选项只会出现在`cacheGroups`的分离规则中，意味重复利用现有的 chunk。例如 chunk 1 拥有模块 A、B、C；chunk 2 拥有模块 B、C。如果 `reuseExistingChunk` 为 `false` 的情况下，在打包时插件会为我们单独创建一个 chunk 名为 `common~for~1~2`，它包含公共模块 B 和 C。而如果该值为`true`的话，因为 chunk 2 中已经拥有公共模块 B 和 C，所以插件就不会再为我们创建新的模块

- `priority`: 很容易想象到我们会在`cacheGroups`中配置多个 chunk 分离规则。如果同一个模块同时匹配多个规则怎么办，`priority`解决的这个问题。注意所有默认配置的`priority`都为负数，所以自定义的`priority`必须大于等于0才行

### 小结

截至目前为止，我们已经看出了一套分离代码的模式：

- 首先决定我们想要解决什么样的问题（避免用户在每次访问时下载额外的代码）；
- 再决定使用什么样的方案（通过将修改频率低、重复的代码分离出来，并配上恰当的缓存策略）；
- 最后决定实施的方案是什么（通过配置 Webpack 来实现代码的分离）

## 参考资料集合

[https://www.site2share.com/folder/20020533](https://www.site2share.com/folder/20020533)

### Bundle VS Chunk

- [What are module, chunk and bundle in webpack?](https://stackoverflow.com/questions/42523436/what-are-module-chunk-and-bundle-in-webpack)
- [Concepts - Bundle vs Chunk](https://github.com/webpack/webpack.js.org/issues/970)
- [SurviveJS: Glossary](https://survivejs.com/webpack/appendices/glossary/)

### Hash

- [What is the purpose of webpack hash and chunkhash?](https://stackoverflow.com/questions/35176489/what-is-the-purpose-of-webpack-hash-and-chunkhash)
- [Hash vs chunkhash vs ContentHash](https://medium.com/@sahilkkrazy/hash-vs-chunkhash-vs-contenthash-e94d38a32208)
- [Adding Hashes to Filenames](https://survivejs.com/webpack/optimizing/adding-hashes-to-filenames/)

### SplitChunksPlugin

- [Webpack 4 — Mysterious SplitChunks Plugin](https://medium.com/dailyjs/webpack-4-splitchunks-plugin-d9fbbe091fd0)
- [Webpack (v4) Code Splitting using SplitChunksPlugin](https://itnext.io/react-router-and-webpack-v4-code-splitting-using-splitchunksplugin-f0a48f110312)
- [Reduce JavaScript Payloads with Code Splitting](https://developers.google.com/web/fundamentals/performance/optimizing-javascript/code-splitting/)
- [Webpack v4 chunk splitting deep dive](https://www.chrisclaxton.me.uk/chris-claxtons-blog/webpack-chunksplitting-deepdive)
- [what reuseExistingChunk: true means, can give a sample?](https://github.com/webpack/webpack.js.org/issues/2122)

### DLL

- [How To Use The Dll Plugin to Speed Up Your Webpack Build](https://medium.com/@emilycoco/how-to-use-the-dll-plugin-to-speed-up-your-webpack-build-dbf330d3b13c)

你可能会喜欢

- [面试系列之二：你真的了解React吗（上）如何设计组件以及重要的生命周期](https://www.v2think.com/understand-react-01)
- [面试系列之三：你真的了解React吗（中）组件间的通信以及React优化](https://www.v2think.com/understand-react-02)
- [面试系列之四：你真的了解React吗（下）Flux与Vuex的差异以及Webpack](https://www.v2think.com/understand-react-03)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置](https://www.v2think.com/create-app-user-guide)
- [深入React的生命周期(上)：出生(Mount)](https://www.v2think.com/dig-into-react-lifecircle-01)
- [深入React的生命周期(下)：更新(Update)](https://www.v2think.com/dig-into-react-lifecircle-02)
- [深入理解 Webpack 打包分块（上）](https://www.v2think.com/webpack-chunks-split-01)
- [深入理解 Webpack 打包分块（下）](https://www.v2think.com/webpack-chunks-split-02)
- [Webpack 速成](https://www.v2think.com/webpack-tutorial)
- [构建大型 Mobx 应用的几个建议](https://www.v2think.com/tips-for-building-mobx-app)
- [【译文】给构建大型 redux 应用的五个建议](https://www.v2think.com/five-tips-for-redux-large-applications)
- [React + Redux 性能优化（一）：理论篇](https://www.v2think.com/redux-performance-01-basic)
- [React + Redux 性能优化（二）工具篇： Immutablejs](https://www.v2think.com/redux-performance-02-immutablejs)
- [Flux与Redux背后的设计思想(二)：CQRS, Event Sourcing, DDD](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CQRS-ES-DDD)
- [Flux与Redux背后的设计思想(一)：Command Bus, Event Bus, Service Bus](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CB-EB-ESB)