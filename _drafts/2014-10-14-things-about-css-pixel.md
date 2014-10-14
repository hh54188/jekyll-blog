---
layout: post
title: 关于移动web开发中像素的那些事
modified: 2014-10-14
tags: [css, mobile, javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

这篇文章首先要对dpi，ppi，idp等一系列概念做一个梳理，然后再引出一下在高清开发下所面临的问题以及解决方案。

## 从最简单的情况开始

首先我们要分清楚两种像素(pixel)，css像素和设备像素。css像素是相对像素，而设备像素是绝对像素。
怎么理解上面的两句话。我们打个比方先。