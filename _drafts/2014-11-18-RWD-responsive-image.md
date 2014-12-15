# 移动开发的那些事（下）：响应式图片设计

如果你还没有了解过有关于响应式设计的一些概念，可以再开始阅读这篇文章之前阅读这一系列的上篇开始：
[浅谈移动Web开发（上）：深入概念](http://www.infoq.com/cn/articles/development-of-the-mobile-web-deep-concept)。上篇介绍了一些你必须掌握的移动端开发中要用到的概念，并且同其他概念做了一些区分。可以说上一篇是为这一篇服务的。在这一篇中，我们就需要运用到上一篇中了解的那些概念。

你可能会有疑问，为什么下篇说的是响应式图片的设计，为什么不是响应式设计。一方面因为响应式设计已经被说的太多了，有太多的文章在谈；另一方面，如果你对响应式这一块有关注的话，响应式图片与响应式设计并非包含关系，从某种意义上来说，响应式图片已经可以成为与响应式设计平行的一个技术分支。所以我们可以拿出来着重的说。

在构思这篇文章的时候，我的想法是按部就班的把响应式图片的技术一一做一个介绍。介绍技术用法非常简单，贴上语法，附上例子，相信大部分人就能看懂就能照葫芦画瓢了。但我一直觉得了解这门技术诞生的原因，了解原技术的不足，了解这个技术的缺陷同样也非常重要。更有助于我们理解这项技术。

但是在学习中逐渐发现，响应式图片技术产生的原因，或者它所面临的困境更加的有趣或者吊诡。或者说，如果我能把所有的这些前因后果做一个介绍，或许能帮助大家更好的理解掌握技术，更好的集思广义。

当然，正如文章的标题所说，文字的容量总是有限的，所以做到的只能是浅谈。所以这篇文章里更多描述的是想法：问题是如何产生的，现有解决方案的不足，新的解决方案的思路，新的解决办法的不足。

## User Cases

首先我们要了解清楚在什么情况下需要用到响应式图片。W3C的RICG(Responsive Images Community Group)小组给出了如下的[四种用户用例](http://usecases.responsiveimages.org#h2_use-cases)，

### 基于分辨率选择：

![resolution_selection](./images/resolution_selection.jpg)

这个是最基本的需求，也就根据网页所处设备分辨率的不同选择图片。你可能觉得说分辨率变了，图片使用同一张用css控制宽高等比压缩就好了。其实不仅仅是根据分辨率不同来选择图片，比如说较小分辨率可能就意味着加载网页的这台设备性能有限，很大概率下是手机的话就意味着带宽有限。如果能根据分辨率加载图片，我们就能提供给适合该设备的图片。节省带宽与时间。

### 基于DevicePixelRatio

![dpr_selection](./images/dpr_selection.jpg)

在上一篇中我们从PPI引出了DevicePixelRatio（以下简称DPR）。 我们知道高像素密度下设备的物理像素很小，而大部分的优化方案会造成图片素材的模糊。因此我们需要为高DPR的设备特意准备更高清的素材。

### 基于viewport

![viewport_selection](./images/viewport_selection.jpg)

或许你的第一反应是基于viewport貌似和基于分辨率非常的像？某种意义上是的。但就像在上一篇所说的那样，移动设备的上的viewport通常与分辨率并非一一对应的关系。比如在iPhone4上，默认的viewport是980px，但是你也可以指定为与visual viewport大小相等。可以这么理解，基于viewport比基于屏幕分辨率更加精确。

### Art direction

我不知道应该如何翻译这个词组比较好，我们姑且采用百度百科里的翻译，译为“美术设计”。

它描述的是这样一种情况，假设我们基于分辨率设计响应式图片。那么在高分辨率和低分辨率下图片素材对比如图：

高分辨率：

![obama-500](./images/obama-500.jpg)

低分辨率：

![obama-100](./images/obama-100.jpg)

我们的任务就完成了，虽然做到了响应式，但是第二张图中的任务几乎是不可见的，我们压根无法分辨出其中的奥巴马。
所以更适合的图片应该是这样：

![obama-100-art](./images/obama-100-art.jpg)

虽然图片大小仍然和低分辨下保持一致，但我们对图片进行了剪裁，略去了可以被忽视的背景图片。

在这之前的几种场景只需要用户上传一张尺寸足够大的图片，然后仅仅缩放图片去适应就好了。而这一种情况下，图片不仅仅是被缩放，还需要被重新设计和剪裁。所以在后面的部分你会看到它使用的技术是和前三者不一样的。

![art_direction](./images/art_direction.jpg)

## `srcset`

【介绍srcset语法的由来】

响应式图片的解决方案简单的可以划分为两类：1.用一张图片解决；2.用多张图片解决

在这里我们先看更实用更复杂更多争议的后者。我们需要用到`picture`元素和`srcset`属性。

为了展示srcset的用法，我们考虑一种最简单的情况。 我们要同一张图为iPhone 3gs与iPhone 4做适配。我们只需要为他们不同对策DPR做适配。
在这种情况下，我们不用考虑屏幕的分辨率，不需要考虑viewport大小。只是两台尺寸同样大小的DPR不同罢了。那么我们需要用一张large.jpg的图片应付DPR为2.0的情况，用一张small.jpg的图片应付DPR为1.0的情况。 使用srcset的语法应该如下：

```
<img src="small.jpg" srcset="large.jpg 2x, small.jpg 1x">
```
非常简单，一目了然。 只需要把图片和对应的设备参数匹配起来（比如上面用x为单位描述屏幕的DPI，也可以用w或者h描述viewport的宽度），再串成字符串用逗号间隔开。
那么一旦浏览器发现了匹配的DPR参数，就加载响应的图片。`img`自带的src标签用于处理默认情况。

但是如果屏幕的DPR为1.3或者1.5应该怎么办？

OK，那让我们把情况变得复杂一些。这次我们响应的不再是屏幕的DPR，而是viewport的宽度。并且我们需要响应三种宽度，小于600px，600px至800px之间，大于800px。我们也用三张图片对应这三种宽度：small.jpg, medium.jpg, large.jpg。参照上面的DPR写法，你可能会想当然的参照上面DPR的写法：

```
<img src="large.jpg" srcset="medium.jpg 800w, small.jpg 600w">
```
ちょっと待（ま）って（等等），你真的确定吗？

宽度与DPR不同的地方在于，如果你熟悉media query的话，DPR（x单位）描述的是精确值；而宽度(width)(w单位)描述的是边界值，但问题是最大边界(max-width)还是最小边界(min-width)呢。这个问题在media query中也同样成立：

A写法：

```
body {}

@media screen and (min-width:480px) {}

@media screen and (min-width:800px) {}
```

B写法

```
body {}

@media screen and (max-width:800px) {}

@media screen and (max-width:480px) {}
```
其实两种写法最终达成的目的并无不同，都将布局划分为三段式响应。

A写法中总是取min-width为分界点，并且分界点逐渐递增。越往后所对应的样式为更宽的样式。这样的划分方法我们称之为移动布局优先（mobile first）
B写法中总是取max-width为分界点，并且分界点逐渐递减。越往后所对应的样式为更窄的样式，这样的划分方法我们称之为桌面优先(desktop first)

回到最上面的srcset关于宽度的写法，我们无法用语法准确的表达我们到底是希望mobile first或者desktop first。(从某些资料中我们可以得出是desktop first)。 或者我们也无从知晓浏览器以为的是采用哪一种趋势。 你可能会建议我现在不如写一段代码来测试一下，但即使通过测试结果浏览器能够明确的表示采用何种趋势，但是对于另一种趋势来说是公平的吗？

`srcset`的情况复杂之处不仅仅如此。当我们把所有的多重user cases糅合在一起时，情况就变得非常复杂了。 假设我们需要满足这样一个需求：

1. 响应式设计中有两个零界点： 640px和960px
2. 在最窄的布局中，图片宽度为viewport的100%
3. 在稍宽一些的布局中，图片的宽度为viewport的50%
4. 在最宽的页面布局中，图片的跨度为viewport的33%
5. 现在我手头上有宽度分别为160px, 320px, 480px, 640px, 960px, 1280px, 1920px的图片

通过计算得出的image的标签应该为：

```

<img srcset="
  320.jpg .89x 400w, 480.jpg 1.33x 400w, 640.jpg 1.78x 400w,
  480.jpg 1.04x 520w, 640.jpg 1.39x 520w, 960.jpg 2.09x 520w,
  640.jpg 1.1x 639w, 960.jpg 1.66x 639w, 1280 2.2x 639w,
  320.jpg 0.89x 800w, 480.jpg 1.33x 800w, 640.jpg 1.78x 800w,
  480.jpg 1.09x 959w, 640.jpg 1.45x 959w, 960.jpg 2.18x 959w,
  320.jpg 0.89x 1200w, 480.jpg 1.33x 1200w, 640.jpg 1.78x 1200w,
  480.jpg 1.09x 1440w, 640.jpg 1.45x 1440w, 960.jpg 2.18x 1440w,
  480.jpg 0.86x 1920w, 640.jpg 1.14x 1920w, 960.jpg 1.71x 1920w, 1280 2.29x 1920w,
  640.jpg 0.86x, 960.jpg 1.29x, 1280 1.71x, 1920 2.57x
">


```

我也不知道它是怎么算出来的，它本意为充分利用每一张图片，覆盖每一种宽度和DPR。简单的srcset语法与media query语法机制类似就是在特定的环境下让浏览器做特定的事情。就类似于分支非常多的条件语句，比如：“当viewport宽度大于xxx时，显示侧边栏；否则隐藏侧边栏”。“当用户的屏幕是Retina时，选择这张高清图片；否则使用一张普通的图片”。就像我们在javascript写分支语句一样，不得不覆盖尽可能多的情况

我们可以系统的数落一下这种方式的几宗罪：

1. 从上面密密麻麻的语法，我压根就看不出响应式布局的分界点（breakpoints）到底在哪？！
2. 你对图片什么100%，50%，33%的宽度也压根体现不出来
3. 代码存在局限性，面对更高或更低像素密度的情况需要加入新的代码
4. 可维护性差，如果加入了新的尺寸的图片或者对现有图片尺寸有更改，则需要重新计算重新来过。

你可能会狡辩说我不需要考虑这么多的情况，我只需要考虑少数的几种情况就好了。但你真的确定在少数情况下浏览器也不会出错？比如看下面这个例子：

```
<img srcset="640.jpg 1x, 1280.jpg 2x">
```
如果图片在网页上展现的宽度为640px，那么在高像素密度的设备上应该选用的是1280.jpg图片没有错。但如果图片在网页上展现的宽度仅为320px，那么即使是在高像素密度的设备上，使用640.jpg图片也是OK的，因为此时缩放比例仍然为2：1，不会有细节上的损失。如果仍然强行加载1280.jpg的话，只会浪费带宽和降低用户体验。

我们可以反思一下为什么会出现这样的问题：
1. 作为开发者，我们无法考虑到上述遇到的实际情况，即使遇到了，我们应该如何表达呢
2. 作为浏览器，它没能做到灵活的变通——但问题是你能指望它如何变通呢，或许它知道图片实际展示的大小，但是它不知道我们的素材大小（上面的命名可以换成small.jpg, big.jpg），

`sizes`出现了，救星出现了。我们先看看语法，以上面的例子为例，使用sizes的话我们可以这么写：

```

<img sizes="(max-width: 640px) 100vw,
			(max-width: 960px) 50vw
			33vw" 
		srcset="160.jpg 160w, 320.jpg 320w, 480.jpg 480w
				640.jpg 640w, 960.jpg 960w, 1280.jpg 1280w
				1920.jpg 1920w"
/>

```

首先看sizes语法，类似于media query：

```
sizes="[media query] [length], [media query] [length] ..., [length] etc"
```
sizes属性值由一系列描述图片宽度的表达式用逗号隔开组成，
每一组由两部分组成，`[media query]`代表匹配的查询条件，`[length]`代表该查询条件下图片所占用的宽度。
最后一个表达式可以只描述图片大小，表示在默认条件下的图片宽度

以上面的例子为例

```
sizes="(max-width: 640px) 100vw,
		(max-width: 960px) 50vw
		33vw"
```
注意vw为viewport单位【和百分比有什么区别：当你使用百分比的时候，百分比是相对于你的包含块而言，但是vw对所有元素都一视同仁】

当viewport宽度不超过640时，图片宽度为100%；
当viewport宽度大于640但小于960时，图片宽度为50%；
当viewport大于960时，图片宽度为viewport的33%，也就是占viewport的三分之一。

为了体现的更灵活，你还可以使用`calc`语法：

```
sizes="(max-width: 640px) 100vw,
		(max-width: 960px) 50vw
		calc(33vw - 20px)"
```

在什么情况下使用呢，比如你每一张图片都希望占viewport的三分之一，并且左右有10px的间隔，因为此时你没有办法知道准确的宽度是多少，于是就交给浏览器去计算吧

`srcset`属性值的含义此时也做了些变化，每一张图片名称后的宽度值并非再表示需要匹配的viewport宽度，而是该图片的宽度，单位为px

通过这两个属性，我们只做了两件事：

1. 告诉浏览器在什么情况下图片应该有多宽
2. 告诉浏览器我手头上有哪些图片可以提供给你
3. 当页面宽度发生变化时你看着办吧！哈哈

等等，什么叫做看着办？

在前一种单一的srcset方式中，我们采用的大家长方式去精确控制每一张图片可能对应的设备情况，我们是命令者，设备是执行者, 缺点上面已经描述的非常清楚了；

而在后一种方式中，我们采用的是粗犷的管理方式，我们只大概表达了我们的想法，比如在什么样的设备条件下我们希望图片的宽度是多少，告诉浏览器我全部的图片有哪些。接下来你就可以完全自己决策了。

具体如何操作呢，让我们看一个简单的例子：

```
<img sizes="(max-width: 30em) 100vw,
			(max-width: 50em) 50vw,
			calc(33vw - 100px)"
	srcset="swing-200.jpg 200w,
			swing-400.jpg 400w,
			swing-800.jpg 800w,
			swing-1600.jpg 1600w"
	src="swing-400.jpg" alt="Kettlebell Swing">
```
假设当前浏览器的viewport大小为20em，同时也假设默认字体大小为16px，那么此时viewport的宽度为320px。那么此时浏览器选择的sizes值为`(max-width:30em) 100vw`，也就是说，图片宽度与viewport的宽度相同。当DPR为1时，浏览器就会选择比320px稍大的`swing-400.jpg`。当DPR为2时，图片宽度至少为640px，浏览器就会选择最接近的`swing-800.jpg`。

当viewport宽度为40em时，浏览器选择的size是`(max-width:50em) 50vw`。对于1x的屏幕，浏览器会选择320px宽度的图片，对于2x的屏幕，浏览器会选择640px的图片。

但是请注意，上面描述的情况仅仅是可能。你在srcset描述的图片信息，仅仅是告诉它我拥有这些资源。具体是否使用仍然要依照浏览器的算法决定。它考虑的会比开发者更多，比如网络延迟情况，用户的偏好等。 它才是决策者。

srcset准确来说是picture标准的一部分。上面的所有例子解决的只是除srcset以外的问题，这些图片有一个共同的特点是，是同一张图片的不同尺寸而已。

art direction的情况会复杂一些，图片需要切割。再在不同切割后图片的基础上实现多个版本：

```
<picture>
   <source media="(min-width: 36em)"
      srcset="large.jpg  1024w,
         medium.jpg 640w,
         small.jpg  320w"
      sizes="33.3vw" />
   <source srcset="cropped-large.jpg 2x,
         cropped-small.jpg 1x" />
   <img src="small.jpg" alt="A rad wolf" />
</picture>
```

在上面的代码中，每一个source代表的就是独立的一类剪裁之后图片，再基于这类图片，制作成多个版本。也就是不同source中的srcset属性

你可以给不同的source天机media query属性，这样浏览器首先会在picture元素中，根据不同的media query选择不同的source，再基于source的sizes和srcset属性，选择对应的图片。规则与picture中的类似，就不再赘述了

## 除了srcset以外

同时srcset也引发了一系列的连锁反应，即使我们把srcset设计的非常完美。也需要有浏览器的配套支持。如果支持的不够好，比如就会发生以下这些问题：

### preloader矛盾

从IE8开始，浏览器逐渐采用一种preloader的机制来提高浏览器加载速度与性能优化。举个栗子，比如下面代码的这样资源分布图：

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

但是你却可以看到网络资源的加载顺序是这样的：

![preloader](./images/preloader.jpg)

也就是说外链的script被提前了。

并没有统一的标准规定这套机制应具备何种功能已经如何实现。但你可以大致这么理解：浏览器通常会准备两个页面解析器parser，一个(main parser)用于正常的页面解析，而另一个(preloader)则试图去文档中搜寻更多需要加载的资源，但这里的资源通常仅限于外链的js、stylesheet、image；不包括audio、video等。并且动态插入页面的资源无效。

OK，那么问题来了，以一个img标签为例，我们通常在协商srcset或者sizes属性的时候，也会一并写在最原始的src属性，以兼容低版本的浏览器。那么在main parser还没有解析到图片的时候，pre parser就首先加载了图片的src属性。一旦main parser正式解析到img的时候，又会再次选择适合的图片，再一次加载。这样就引起了两次的加载。smashing magzine上有专门的一篇文章[How To Avoid Duplicate Downloads In Responsive Images](http://www.smashingmagazine.com/2013/05/10/how-to-avoid-duplicate-downloads-in-responsive-images/)来谈如何来解决这个问题。方法很多很广。

## 结束

上面说了那么多。我们不得不面对的显示是实现的了上面这些语法的浏览器屈指可数，无论是PC端还是移动端：http://caniuse.com/#search=picture 不过值得庆幸的是 RIG提供了polyfill的脚本来作为备选方案。
【
	怎么来说接下去的问题？
	从DPR引出srcset？
		引出srcset语法？

	引出viewport情况？
	虽然和DPR情况一致，但是在这里存在min-width或者max-width的区别
	引出srcset在w下的矛盾 **<----思想**

	如何决定images breakpoints***<----思想*

	姑且以mobile-first写出查询代码
	发现非常的繁琐
	于是引入sizes

	解决了刚刚的那个问题，所有都交给了智能的浏览器来解决
	但是要注意算法的问题，很多情况并非如我们所期望的 **<---思想**

	direction的解决方案有所不同，采用的是picture元素和source元素。虽然上面的sizes也是picture的语法

	为什么srcset和<picture>没法互相替换？http://html5hub.com/the-src-n-responsive-image-solution/
	srcset的局限在哪里？

	1. breakpoints变的不是那么清晰可见
	2. 有大量的计算算出
	3. 图片在不同viewport宽度下占用的宽度也不是清晰可见了
	4. 缺少远见，只考虑到了1x, 2x的情况
	5. 缺少可维护性，每次修改可能引发修改其他的片段


	srcset与preloader的矛盾 <---思想：		
	picture的fallback问题 <---思想
	
		http://www.smashingmagazine.com/2013/05/10/how-to-avoid-duplicate-downloads-in-responsive-images/
		http://blog.cloudfour.com/the-real-conflict-behind-picture-and-srcset/


	！！！以上引入的的思想不多啊
】


