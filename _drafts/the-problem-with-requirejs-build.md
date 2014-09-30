---
layout: post
title: require.js打包中的一个坑
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-03-13
tags: [css, html, front-end]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 问题是什么

我们在使用requirejs进行模块化开发的时候，通常务必会有一个入口脚本，在这里比如我们叫做`main.js`，最近我发现，main.js与其他模块在文件中的关系可能是需要小心的组织，否则会在打包中发生一些问题。比如下面这种情况：

```
|--build // 打包输出目录
|--build.js // 打包脚本
|--main.js // 模块入口
|--src // 源码目录
    |--foo_module.js
    |--bar_module.js
    |--lib
        |--jQuery.js
        |--Bakcbone.js
```

这种情况的特点是，入口main.js和其他的模块并不在统一目录中。

此时你的build.js可能会这么写：

```
({
    baseUrl: './',
    name: "./main",
    out: "./build/main-built.js"
})
```

So far so good。因为main.js与其他模块并非在同一目录中，因此main.js同样也需要进行配置：

```
requirejs.config({
    baseUrl: "./src",
    paths: {
        "jQuery": "./lib/jquery"
    }
});
```

OK，那问题来了，当你使用build.js开始进行打包时，配置的根目录为当前根目录`./`。打包脚本能够找到main.js。**但此时需要根据main.js中的配置重置查找目录为`./src`。但打包脚本不会这么做**。于是就报错了，因为它始终停留在第一次的打包目录中，也就无法找到`foo_module.js`与`bar_module.js`。

或者简单来说，导致这个问题的原因是下面两点

1. 打包所使用的基本目录与开发所使用的基本目录不一致
2. 在打包过程中，打包脚本无法读取开发中的配置。

所以可见问题在于build.js与main.js所使用的基本路径不一致。

## 解决方案

很简单，把main.js和其他模块都放在同一级目录即可。

```
|--build.js // 打包脚本
|--src // 源码目录
    |--main.js // 模块入口
    |--foo_module.js
    |--bar_module.js
    |--lib
        |--jQuery.js
        |--Bakcbone.js
```

那么即使build.js与main.js都配置了打包目录，都应该指向的是main.js所在的`/src`目录。

打包脚本的配置肯定已经比我们先考虑到了这个问题，在打包配置中有这样一个字段`mainConfigFile`，用来单独指定有关main.js的配置文件，这也就意味着main.js与build.js公用同一个配置文件，在这里我们比如叫`config.js`。看上去很美好，但第一个问题是如何在main.js中如何引入config.js?

// 这里还是需要确定一下先的
OK，requirejs貌似没有提供这样的机制，目前我找到的两个方法都并不完美

1. 方法一：把config.js中的内容在main.js中再复制一遍。也就是同时维护两份相同内容的代码。

2. 方法二： 把config.js在main中引入，而main.js中首先引入config.js。比如这个样子：

```

```





