为你的用户找到最佳的文件加载方式是一件麻烦的事。因为有非常多的不同的场景，技术栈，专业术语需要考虑。

在这篇文章里我希望呈现给你所需要的一切，然后你就可以：

- 知道什么样的文件分离策略最适合你的站点和你的用户
- 知道如何实现它

根据 [Webpack 术语表](https://webpack.js.org/glossary/)，有两类文件的分离。这些名词听起来是可以互换的，但实际上不行：

- 打包分离 (Bundle splitting)：为了更好的缓存创建更多、更小的文件（但仍然以每一个文件一个请求的方式进行加载）
- 代码分离 (Code splitting)：动态加载代码，所以用户只需要下载当前他正在浏览的站点的这部分代码

第二种策略听起来更吸引人是不是？事实上许多的文章也假定认为这才是唯一值得将 JavaScript 文件进行小文件拆分的场景。

但是我在这里告诉你第一种策略对许多的站点来说才更有价值，并且应该是你首先为页面做的事

让我们来深入这件事

## 打包分离 (Bundle splitting)

打包分离背后的思想非常简单。如果你有一个体积巨大的文件，并且只改了一行代码，用户仍然需要重新下载整个文件。但是如果你把它分为了两个文件，那么用户只需要下载那个被修改的文件，而浏览器则可以从缓存中加载另一个文件。

值得注意的是因为打包分离与缓存相关，对站点的首次访问者来说没有区别

（我认为太多的性能讨论都是关于站点的首次访问。或许部分原因是因为“第一映像”很重要，另一部分因为这部分性能测量起来简单和讨巧）

当谈论到频繁访问者时，量化性能提升带来的影响会稍有棘手，但是我们必须量化！

这将需要一张表格，所以我们将每一种场景与每一种策略的组合结果都记录下来

这是我在前一段中提到的场景：

- Alice 连续10周每周访问站点一次
- 我们每周更新站点一次
- 我们每周更新“产品列表”页面
- 我们也有一个“产品详情”页面，但是我们目前不需要对它进行更新
- 在第5周的时我们给站点新增了一个 npm 包
- 在第8周时我们更新了现有的一个 npm 包

当然包括我在内的某些人希望场景尽可能的逼真。不要那么做，实际的场景其实无关紧要，我们随后会解释为什么。

## 基线

假设我们的 JavaScript 打包后的总体积时400KB, 并且目前我们以但文件的形式加载它，命名为`main.js`

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

（对那些刚接触缓知识的人我解释一下：存任何时候我提及`main.js`的时候，我实际上是在说类似于`main.xMePWxHo.js`这种类似于这种包含一堆带有文件内容哈希字符串的文件名。这意味着当你应用中的代码发生更改时就会生成一个新的文件名，这样就能迫使浏览器下载新的文件）

当每周我向站点发布新的变更时，包的`contenthash`就会发生更改。所以每周 Alice 访问我们站点时不得不下载一个全新的 400KB 大小的文件

![](./images/webpack-chunk-split/001.png)

连续十周**也就是 4.12MB**

我们能做的更好

## 分离vender包

让我们把我们的打包文件划分为`main.js`和`vendor.js`

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

在你没有告诉它你想如何拆分打包文件的情况下， Webpack 4 在尽它最大的努力把这件事最的最好

这就导致一些声音在说：“太惊人了，Webpack 做的真不错！”

而另一些声音在说：“你对我的打包文件做了什么！”

无论如何，添加`optimization.splitChunks.chunks = 'all'`配置也就是再说“把所有`node_modules`里的东西都放到`vendors~main.js`的文件中去”

在实现基本的打包分离条件后，Alice 在每次访问时仍然需要下载 200KB 大小的 `main.js` 文件， 但是只需要在第一周、第五周、第八周下载 200KB 的 `vendors.js`脚本

![](./images/webpack-chunk-split/002.png)

**也就是 2.64MB**

体积减少了 36%。对于配置里新增的五行代码来说结果还不错。在继续阅读之前你可以立刻就去试试。如果你需要将 Webpack 3 升级到 4，也不要着急，升级不会带来痛苦（而且是免费的！）

## 分离每一个 npm 包

我们的 `vendors.js`承受着和开始`main.js`文件同样的问题——部分的修改会意味着重新下载所有的文件

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

- Webpack 有一些那么智能配置的默认“智能”配置，比如当分离打包输出文件时只允许最多3个文件，并且最小文件的尺寸是30KB（如果存在更小的文件就把它们拼接起来）。所以我把这些配置都覆盖了
- `cacheGroups`是我们用来制定规则告诉 Webpack 应该如何组织 chunks 到打包输出文件的地方。我在这里对所有加载自`node_modules`里的 module 制定了一条名为 "vendor" 的规则。通常情况下，你只需要为你的输出文件的 `name`定义一个字符串。但是我把`name`定义为了一个函数（当文件被解析时会被调用）。在函数中我会根据 module 的路径返回包的名称。结果就是，对于每一个包我都会得到一个单独的文件，比如`npm.react-dom.899sadfhj4.js`
- 为了能够正常发布[npm 包的名称必须是合法的URL](https://docs.npmjs.com/files/package.json#name)，所以我们不需要`encodeURI`对包的名词进行处理。但是我遇到一个问题是.NET服务器不会给名称中包含`@`的文件提供文件服务，所以我在代码片段中进行了替换
- 整个步骤的配置设置之后就不需要维护了——我们不需要使用名称引用任何的类库

Alice 每周都要重新下载 200KB 的 `main.js` 文件，并且再她首次访问时仍然需要下载 200KB 的 npm 包文件，但是她再也不用重复的下载同一个包两次

![](./images/webpack-chunk-split/003.png)

也就是**2.24MB**

相对于基线减少了 44%，这是一段你能够从文章里粘贴复制的非常酷的代码。

我好奇我们能超越 50%？

那不是很棒吗

## 把应用代码进行分离

现在让我们把目光转向 Alice 一遍又一遍下载的 `main.js` 文件

我之前提到过我们的站点里又两个完全不同的部分：一个产品列表页面和一个详情页面。每个页面独立的代码提及大概是 25KB（共享 150KB 的代码）

我们的“产品详情”页面目前不会进行更改，因为它非常的完美。所以如果我们把它划分为独立文件，大部分时候它都能够从缓存中进行加载

并且你知道我们还有一个用于渲染 icon 用的 25KB 的几乎不发生修改的 SVG 文件吗？

我们应该对做些什么

我们仅仅手动的增加一些 entry 入口，告诉 Webpack 给它们都创建独立的文件：

```javascript


module.exports = {
  entry: {
    main: path.resolve(__dirname, 'src/index.js'),
    ProductList: path.resolve(__dirname, 'src/ProductList/ProductList.js'),
    ProductPage: path.resolve(__dirname, 'src/ProductPage/ProductPage.js'),
    Icon: path.resolve(__dirname, 'src/Icon/Icon.js'),
  },
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: '[name].[contenthash:8].js',
  },
  plugins: [
    new webpack.HashedModuleIdsPlugin(), // so that file hashes don't change unexpectedly
  ],
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

并且 Webpack 为它们之间的共享代码也创建了独立的文件，也就是说`ProductList`和`ProductPage`不会拥有重复的代码

这回 Alice 在大多数周里都会节省下 50KB 的下载量

![](./images/webpack-chunk-split/004.png)

**只有 1.815MB 了**

我们已经为 Alice 节省了 56% 的下载量，并且节省工作一直会持续下去（在我们的理论场景中）

并且所有这些都是通过修改 Webapck 配置实现的——我们还没有修改任何一行应用程序的代码。

我之前提到测试之下是什么样具体的场景并不重要。因为无论你遇见的是什么场景，结论始终是一致的：把你的代码划分为更多更有意义的小文件，用户需要下载的代码也就越少









