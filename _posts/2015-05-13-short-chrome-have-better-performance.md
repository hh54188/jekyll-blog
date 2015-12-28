---
layout: post
title: 【短篇·发现】不知不觉,Chrome又做了一个优化
modified: 2015-05-13
tags: [css, mobile, javascript, chrome, performance, short]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

很久没有写文章了，整个2015上半年下了班以后都在忙一个自己的项目，实在是没有腾不出时间来了。因为写文章也是比较累的一件事，要参考非常多的文章，通常要写底稿一遍，再校验一遍，再润色一遍，至少要持续两周的时间。其实`_drafts`里有几篇初稿，接近完成的程度，但都觉得不够满意，而又被其他项目中断，所以就没有继续了。

我决定至少用业余时间写一些短篇，大致是一些技术中的坑或者是发现或者是其他什么，都是在几百字内能够解决的，希望大家能喜欢。

<hr/>

昨天想做一个让浏览器阻塞的DEMO，于是很自然的想到了给`scroll`事件绑定一个函数，这个函数需要很长时间执行，比如三秒。这样以来，用户滚动页面时就会不断的触发`srcoll`的回调函数，而每个函数又需要执行三秒，导致渲染无法进行，滚动时顿卡。


{% highlight html %}
<!DOCTYPE html>
<html>
<head>
	<title></title>
	<style type="text/css">
		html {
			height: 2000px;
		}
	</style>
</head>
<body>
	<script type="text/javascript">
		function takeSomeTimes () {
			var start = +new Date;
			var duration = 1000 * 3;

			while ((+new Date) - start <= duration) {}
		}

		window.onscroll = function () {
			takeSomeTimes(); // 每一次滚动触发都需要耗时三秒
		}
	</script>
</body>
</html>
{% endhighlight %}

但是当我打开这个页面在Chrome中运行时，我发现页面并没有顿卡，还是能很流畅的滚动。

于是我又检查了一遍代码。没有问题啊。我尝试在`takeSomeTimes`函数中每执行三秒打印一次结果，也是正常的。

我开始怀疑是浏览器的问题。果然，当我把页面在IE11或者Firefox中运行时，页面有顿卡现象。

我的第一反应是Chrome这是要逆天的节奏吗，竟然可以让重绘和脚本执行互不干扰了。

## 真相竟然是

随后今天我拿这件事和另一个同学讨论之后有所启发，在页面上新增了一个`position:fixed`元素，为了保证元素处于视口的固定位置，在滚动时必然引起重绘。

此时再在Chrome中滚动时，Chrome已经变得像其他浏览器一样顿卡了。

但是我把`position:fixed`元素去掉后，在页面上加入了一些自然的有内容的`<p>`元素，页面仍然不会顿卡。

于是我们可以得出以下两点结论：

1. Chrome并没有跳出传统浏览器引擎大的框架，脚本执行与页面渲染仍然是宿敌（无法并行）
2. 但Chrome的优化之处在于，它尽可能的在避免重绘，比如上面当页面布局都为normal flow时。具体原理在我前几篇文章中有描述，还有深入待研究。
