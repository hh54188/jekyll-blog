# 利用浏览器特性对资源加载进行优化

## 过去：Preload

preload并不是它唯一的名称，至少在Chrome中可以这么称呼它，在IE中可以称之为Lookahead Pre-parser。

IE8除了将每台host的最高并行下载的资源数从2个提升到6个之外，最大的提升莫过于允许并行下载脚本文件了。

比如下面这个[页面](http://stevesouders.com/cuzillion/?c0=hc1hfff2_0_f&c1=hc1hfff2_0_f&c2=hj1hfff2_0_f&c3=hj1hfff2_0_f&c4=bi1hfff2_0_f&c5=bi1hfff2_0_f&c6=bj1hfff2_0_f&c7=bi1hfff2_0_f&t=1382383139903)

```
//只是大意表示资源分部情况
<head>
    <link rel="stylesheet" type="text/css" href="">
    <link rel="stylesheet" type="text/css" href="">
    <script type="text/javascript"></script>
    <script type="text/javascript"></script>
</head>
<body>
    <img src="">
    <img src="">
    <script type="text/javascript"></script>
    <img src="">
</body>
```

在IE7下网络请求的瀑布图如下图所示：

![no-pre-loader-waterfall-ie7](./images/no-pre-loader-waterfall-ie7.png)

我们能看到head中的外链样式能够并行下载，body中的图片能并行下载，但惟独外链脚本不行。

但如果你在IE8中查看网络请求的，瀑布图如下图所示

![no-pre-loader-waterfall-ie7](./images/pre-loader-waterfall-ie8.png)

head中的脚本和外链样式已经可以并行下载了。并且我们可以看到整个页面的载入时间从14s下降到7s。