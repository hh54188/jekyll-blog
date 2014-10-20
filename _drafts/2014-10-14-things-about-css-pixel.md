---
layout: post
title: 待定
modified: 2014-10-14
tags: [css, mobile, javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 各种参数

### 从最简单开始的情况开始

假设我们有一台非常普通的显示器，最佳分辨率为1280x800。此时我们用浏览器打开一个测试页面，浏览器最大化，页面上有一个320x200(单位：px)的红色容器，那么我们只要复制这个容器四份，从左至右依次排列，便能在宽度上占满整个浏览器，也就占据了整个屏幕。

假设情况变的复杂一点，我们将浏览器页面放大至200%（“Ctrl键”加上“+号键”）。此时会发生什么？本来只占页面四分之一宽度的容器已经占据了屏幕的一半。需要注意的是，我们既没有增加容器的宽度，也没有改变屏幕的分辨率。放大页面实际上操作的是页面的像素，**也就是我们把像素拉伸到原来的两倍了**。 请注意这里的两倍是相对谁而言的，是相对于100%状态页面状态而言，也就是相对于屏幕像素而言。

CSS像素与屏幕像素1：1同样大小时：
![origin_pixel](./images/ppi/csspixels_100.gif)
CSS像素开始被拉伸，此时1个CSS像素大于1个屏幕像素
![zoom_in_pixel](./images/ppi/csspixels_in.gif)

举这个例子我想说明一件事：CSS像素从来只是一个**相对值**。 W3C从来就没有规定过一个单位的CSS像素值需要和一个屏幕上的物理像素值相等，虽然到今天为止大部门电脑仍然是这么做的。


