# 让我们再聊聊浏览器资源加载优化

## Old Days

几乎每一个前端程序员都知道应该把script标签放在页面底部。关于这个经典的论述可以追溯到Nicholas的`High Performance Javasript`，在第一章`Loading and Execution`中，他建议这么做的原因如下：

>Put all `<script>` tags at the bottom of the page, just inside of the closing  `</body>`
tag. This ensures that the page can be almost completely rendered before script
execution begins.

简而言之因为单线程的缘故，浏览器加载脚本继而执行脚本，会引起页面的渲染被暂停，甚至还会阻塞其他资源，比如图片。为了更快的给用户呈现网页内容，更好的用户体验。应该把脚本放在页面底部，使它最后才加载。

不仅仅是脚本的位置，包括我们常常聊到的一些优化技巧，比如为了减少HTTP请求，尽可能多的把样式和脚本合并成一个文件比如css sprites那样……其实不知道你有没有想过，时至今日这些做法是否仍然可行？是否有值得商榷的地方，或者说值得改进的地方。这也是我为什么在标题中加使用“再”这个字的原因。`No Silver Bullet`，没有什么是万能的，关键是要看你的业务需要什么，你对性能有什么样的要求。

## Pre-loader

### 什么是pre-loader

首先让我们看一看这样一个资源分布的[页面](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hj1hfff2_0_f&c2=bi1hfff2_0_f&c3=bi1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bi1hfff2_0_f&c7=bi1hfff2_0_f&c8=bi1hfff2_0_f&c9=bi1hfff2_0_f&c10=bj1hfff2_0_f&c11=bj1hfff2_0_f&c12=bj1hfff2_0_f&t=1312331291063)：

```
<head>
    <link rel="stylesheet" type="text/css" href="">
    <script type="text/javascript"></script>
</head>
<body>
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <img src="">
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
</body>
```

这个页面的特点是，一个外链脚本置于头部，三个脚本放在页面的底部，并且是故意放在一系列img之后，在Chrome中页面加载的网络请求顺序如下：

![./images/chrome-waterfall.jpg](./images/chrome-waterfall.jpg)

虽然脚本放在图片之后，但加载仍先于图片（但不会被执行）。为什么会出现这样的情况？为什么故意置后资源能够提前加载？

不同浏览器的制造厂商们(vendor)非常清楚浏览器的瓶颈在哪，虽然实现不同，但原理近似: network, javascript evaluate, reflow, repaint. 针对这些问题，浏览器也在不断的进化，所以我们才能看到更快的脚本引擎，调用GPU的渲染机制等一推陈出新的优化技术和方案。

同样在资源加载上，早在IE8开始，有一种叫做`lookahead pre-parser`(在Chrome中称为pre-loader)的机制就已经开始在不同浏览器中实现。IE8的进步除了将每台host最高并行下载的资源数从2提升至6，并且能够允许并行下载脚本文件，最后就是加入了这种机制

同样以上面的页面为例，我看看IE7下的瀑布图：

![./images/chrome-waterfall.jpg](./images/1-script-head-3-script-body-ie7.png)

很明显底部的脚本并没有提前被加载，并且因为由于单个域名最高并行下载数的限制，资源总是两个两个很整齐的错开并行下载。

但在IE8下，很明显底部脚本被提前：

![./images/chrome-waterfall.jpg](./images/1-script-head-3-script-body-ie8.png)

没有统一的标准描述关于这套机制应该如何实现以及细节应该是什么。但实现原理类似：浏览器通常会准备两个parser，一个(main parser)用于正常的页面解析，而另一个(preload scanner/lookahead pre-parser)则试图去文档中搜寻更多需要加载的资源，但这里的资源通常仅限于外链的js、stylesheet、image；不包括audio、video等，动态插入页面的资源无效。

但细节方面却要加以区分：比如关于preloader的触发时机，并非与解析页面同时开始，而通常是在加载某个head中的外链脚本阻塞了main parser的情况下才启动；也不是所有浏览器的preloader会把图片列为预加载的资源，可能它认为图片加载过于耗费带宽；preloader也并非最优，在某些浏览器中它会阻塞body的解析，有的浏览器将文档被分为head和body两部分进行解析，在head没有解析完之前，body不会被解析，一旦在解析head的过程中触发了preloader，这无疑会导致head的解析时间过长。

### Preloader在响应式设计中的问题

preloader的诞生本是出于一番好意，但好心也有可能办坏事。或者说在我还没来得及告诉你怎么办之前你就把事给办了。

filamentgroup有一种解决响应式设计的图片解决方案[Responsive Design Images](https://github.com/filamentgroup/Responsive-Images/tree/cookie-driven)：

```
<html>
<head>
    <title></title>
    <script type="text/javascript" src="./responsive-images.js"></script>
</head>
<body>
    <img src="./running.jpg?medium=_imgs/running.medium.jpg&large=_imgs/running.large.jpg">
</body>
</html>
```

它的工作原理是，当`responsive-images.js`加载完成时，它会检测当前显示器的尺寸，并且设置一个cookie来标记当前尺寸。同时你需要在服务器端准备一个`.htaccess`文件，接下来当你请求图片时，.htaccess中的配置会检测随图片请求异同发送的Cookie是被设置成`medium`还是`large`,这样也就保证根据显示器的尺寸来加载对于的图片大小。


很明显这个方案成功的前提是，js执行先于发出图片请求。但在Chrome下打开，你会发现执行顺序是这样：

![responsice-image.png](./images/responsice-image.png)

`responsive-images.js`和图片几乎是同一时间发出的请求。结果是第一次打开页面给出的是默认小图，如果你再次刷新页面，因为Cookie才种上，服务器返回的是大图。

严格意义上来说在某些浏览器中这不一定是preloader引起的问题，但preloader引起的问题类似：插入脚本的顺序和位置或许是开发者有意而为之的，但preloader的这种“聪明”却可能违背开发者的意图，造成偏差。


如果你觉得上一个例子还不够说明问题的话，最后请考虑使用[`picture`](http://picture.responsiveimages.org/)(或者`@srcset`)的情况：

```
<picture>
    <source src="med.jpg" media="(min-width: 40em)" />
    <source src="sm.jpg"/>
    <img src="fallback.jpg" alt="" />
</picture>
```

在preloader搜寻到该元素并且试图去下载该资源时，它应该怎么办？一个正常的paser应该是在解析该元素时根据当时页面的渲染情况和布局去下载。而当时这些工作一定还没有完成.

退一步来说，即使不考虑页面渲染的情况，假设preloader在这种情形下会触发一种默认加载策略，那应该是"mobile first"还是"desktop first"？默认应该加载高清还是低清照片？

## Delay Execution

个人认为脚本最致命的不是加载，因为何时加载毕竟是可控的，甚至是异步的，比如通过调整外链的位置，动态的创建脚本。最致命的的是脚本的执行。一旦脚本加载完成，它就会被立即执行(Evaluate Script)，页面的渲染也就随之停止，甚至导致在低端浏览器上假死。

当然有办法解决这个问题，但是现在这里卖个关子——其实我们可以问问自己，延迟脚本执行是否能治本？问题究竟出在了哪？











