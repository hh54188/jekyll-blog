---
layout: page
permalink: /project/
title: 项目
description: "Instructions on how to install and customize the modern Jekyll theme HPSTR."
tags: [Jekyll, theme, install, setup]
image:
  feature: abstract-11.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
share: true
---

## Utility库

[Github](https://github.com/hh54188/UtilityLib)

这个类库是为自己制作的jekyll主题准备的特效类库，包含大部分在博客网站中用到的特效，比如回到顶部，`position:sticky`，延迟加载等。

写这个类库的另一个目的也是想对自己积累的知识来一次实践。**类库中的每一处代码尽可能做到专业与高效，充分利用浏览器特性；使用的每一处hack或者思想也都标明了引用之处**。同时也欢迎各位指出不足之处。

---

## 简单灵活的的Nodejs缓存管理模块

[Github](https://github.com/hh54188/Node-Simple-Cache)

这是一个非常简单的缓存管理模块，具体的技术实现和原理可以参考这一篇文章：[在Node.js中搭建缓存管理模块](http://qingbob.com/built-cache-management-module-in-nodejs/)

写这个模块的初衷是源于同事的一个需求，恰好自己也有过类似的想法：

一般聊到使用缓存想到的一定是Redis或者Memcached，这是这两个都太过于笨重了，至少在我们的需求中是希望能够减少IO操作即可，不需要任何类似于备份、热启动、多进程共享的额外功能。甚至一个简单`Object`对象就能完成这个任务。如果使用Redis的话很明显还有提升性能的空间。于是决定写这么一个轻一些的缓存管理模块。

---

## 豆瓣数据抓取工具

[Github](https://github.com/hh54188/scrapy_douban)

在线运行地址（部署在heroku上，可能不太稳定）：[houseinbeijing](http://houseinbeijing.herokuapp.com)

**我一定要说豆瓣的小组的用户体验太差了！**当时正值我需要租房，于是加了四个小组。但这四个租房小组帖子的刷新频率非常高。于是我每天不得不面对海量的信息，必须不停地刷新页面才能获得最新的信息。**豆瓣的小组不提供关键字搜索和按发帖时间排序排序!**

当时又正值我自学python，于是我就写了这么一个工具，用于抓取豆瓣上各个租房小组的信息。每天只看一个页面就好了，还能根据关键字过滤。

我没想到这个抓取工具这么受欢迎，在github上fork的次数仅次于上面的缓存管理模块。至少我觉得技术含量不高。

---