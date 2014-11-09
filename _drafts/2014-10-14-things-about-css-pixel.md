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

如果你是一个像我一样刚开始接触移动Web开发的前端工程师，那么你或许也遇到了和我同样遇到的问题：有太多相似的概念需要掌握和区分，并且还可能牵涉到具体的设备和硬件。没关系，这篇文章我把他们一一列举和联系起来。我们从最简单的需求出发，把这些概念串联起来。


我们将从一个泛滥的概念开始谈论起，PPI

## PPI

### 什么是PPI

PPI的复杂之处在于他所属的上下文环境不同，意义会完全不一样。就好像”师傅“这个词在西游记里可以指唐僧，可以是正在询问的路人，也可以是公交车司机，也可以是你学艺的老师。同理，当我们谈论显示设备时可以是指像素密度，可以和图片打印相关一个参数，也可以和dpi互换。

这个概念在不同上下文中意义是不同的，在这里我们只讨论在显示器上的情况。并且它经常和DPI混淆。文章的最后会介绍PPI的在其他语境中的一些情况和对DPI的区别。

首先我们先谈论PPI处于描述显示设备参数的情况。

PPI全称为Pixel Per Inch，译为每英寸像素取值，更确切的说法应该是像素密度，也就是衡量单位物理面积内拥有像素值的情况。

![original](./images/ppi/original.png)

如上图所示，在1英寸单位内面积内拥有的像素越多，PPI值就越高。

但像素密度的实际意义是什么？或高或低对设备显示来说有什么影响？

一般来说，我们当然希望PPI值越高越好，因为更高的PPI意味着在同一物理屏幕上能够展现更多的画面细节，也就意味着更平滑的画面。原理就如下图所示：

![original](./images/ppi/original.jpg)

### 什么是Pixel

#### 设备像素

这里我中断一下，上面我提到过非常多次的像素（pixel）。你有没有考虑过，Pixel Per Inch中的pixel究竟值的是什么？

无论是早期的CRT显示器还是如今的LCD显示器，都是基于点阵的。也就是说通过一些列的小点排列成一个大的矩形，不同的小点通过显示不同的颜色来显示成图像。比如下图就是LCD显示器上一个6x6个小点排列成的矩阵：

![Dots5](./images/ppi/Dots5.png)

注意每一个点（dot，也可以称之为pixel）又是由三个子像素(subpixel)红绿蓝组合而成。当需要显示图片信息时，它的工作原理可以如下图所示：

![whats_a_pixel_7](./images/ppi/whats_a_pixel_7.jpg)
![whats_a_pixel_9](./images/ppi/whats_a_pixel_9.jpg)

注意上图代表的是LCD显示器的物理像素，早期的CRT显示器的物理像素同样也是一个一个的点组成。但具体如下图所示：

![crt_pixel](./images/ppi/crt_pixel.jpg)

OK。那么这样一个显示器上的像素我们就称之为物理像素(physical pixel)或者设备像素(device pixel)

#### CSS像素

作为一个Web开发者，我们接触的更多是给元素一个以像素为单位的宽度值。这里的像素我们称之为CSS像素。

CSS像素有什么特别的地方？我们看看下面这个例子：

假设我们用PC浏览器打开一个页面，浏览器此时的宽度为800px(这里的px我们可以假设为设备像素)。页面上同时又一个400px宽的块级元素容器。很明显此时块状容器应该占页面的一半。

但如果我们把页面放大（通过“Ctrl键”加上“+号键”），放大为200%，也就是原来的两倍。此时块状容器则横向占满了整个浏览器。

吊诡的是此时我们既没有调整浏览器窗口大小，也没有改变块状元素的css宽度，但是它看上去却变大了一倍。这是为什么？——原来我们刚刚把CSS像素放大为了原来的两倍。

CSS像素与屏幕像素1：1同样大小时：

![origin_pixel](./images/ppi/csspixels_100.gif)

CSS像素(黑色边框)开始被拉伸，此时1个CSS像素大于1个屏幕像素

![zoom_in_pixel](./images/ppi/csspixels_in.gif)

也就是说默认情况下通常一个CSS像素等于一个物理像素，但是浏览器的放大操作让一个CSS像素等于了两个设备像素。在后面你会看到更复杂的情况，在高PPI设备或者移动设备上，CSS像素甚至在默认状态下就由多个物理像素组成。

通过上面这个例子我想传递一个非常重要的概念，就是CSS像素从来都只是一个相对值。W3C也从来都是如此规定的。

#### 正确答案

回到PPI上来，现在我们有了两种像素，设备像素和CSS像素。那么PPI中的像素是指哪一种？答案是设备像素

**PPI中的pixel指的应该是物理像素**。

**但是**在维基百科对[PPI](http://en.wikipedia.org/wiki/Pixel_density)的解释中，pixel被解释为了分辨率下像素：

>The apparent PPI of a monitor depends upon the screen resolution (that is, the number of pixels) and the size of the screen in use; a monitor in 800×600 mode has a lower PPI than does the same monitor in a 1024×768 or 1280×960 mode.

上面这段话是在说，同一尺寸的显示器在800x600分辨与1024x768分辨率下的像素密度明显是不同的，明显后者单位面积内的像素更多，当然后者的像素密度更高。

这里考虑了另一种情况，即在同一显示器下因为分辨率调整导致显示器的像素密度不同。这里的像素虽然是不是在浏览器中显示，但原理也类似于CSS像素，即由多个物理像素组成一个指定分辨率下的像素，仍然算作一个类型的像素。

**但问题是**，这样的比较是没有任何意义的，我们在通常比较PPI时，首先是夸设备比较，在这一定是比较最优或者是极限情况，这样情况下分辨率通常是与物理像素意义匹配。这样才能体现出设备的优势。换句话说也就一定是最佳分辨率状态下，也就是等于物理分辨率的情况下。不能说一台23寸的2k显示器和一台1080p显示器因为都能调整到1440x960的分辨率，那么他们的PPI就相同了，即使在这种分辨率模式下真的是相同。

**在再进一步，当我们谈论在谈论一台设备的PPI时，它是一个定值，是一个固定的参数。**

那么PPI怎么计算呢？没错，就和你想的一模一样，用屏幕边的物理像素除以物理尺寸即可，以Samsung Galaxy S4为例：

![samsung](./images/ppi/samsung.jpg)

###  The Bad and the Ugly & Retina

但PPI过高同样也会带来问题，相同的图片素材，在越高的设备上会显示的越小，以下是一个像素(1 pixel)在不同PPI设备上的可见情况，随着PPI的增高可视度越来越小：

![pixel-density-1](./images/ppi/pixel-density-1.png)

那么可以预见一种很糟糕的情况是，同一尺寸的屏幕下假设PPI提高了一倍，很可能程序界面缩小了4倍（因为在屏幕尺寸不变的情况下物理像素点面积是原来的1/4）。

以Surface Pro 3为例，它的默认分辨率是2160x1440，也就是说Surface这台设备的屏幕物理像素有2160x1440个点，同时默认分辨率情况下，一个点物理像素点对应于一个分辨率像素。 但因为屏幕只有12寸，像素密度非常高，于是就出现了上面的问题，各个文字和图标被缩的太小了，电脑是完全不可用的。

解决方法是，Windows默认将所有的文本和素材（实际上就是分辨率像素）都放大了1.5倍（在“屏幕分辨率”-“放大或缩小文本和其他项”中进行了设置），原来是一个物理像素对应一个分辨率下的像素，现在则是1.5个物理像素对应一个分辨率下的像素，也就意味着分辨率像素变大了，那么实际上现在的分辨率已经变成了1440(2160/1.5)x900(1440/1.5)（此时如果你尝试用window.screen.width/window.screen.height去检测返回结果也会是1440x900）。这里留给读者一个问题，这样和直接将PC的分辨率调整为1440x900有什么区别呢？

![microsoft_surface_pro_3_scaling_options](./images/ppi/microsoft_surface_pro_3_scaling_options.png)

但把素材和文字放大就真的一劳永逸了吗？不，甚至还会带来副作用。放大素材对位图来说是非常危险的一件事。假设一款软件中的素材的像素为32x32的图片，但是为了配合整体界面的拉伸,它也必须被拉伸至原来的1.5倍，成为48x48。你一定有在Photoshop中把图片强制放大为原来几倍的效果。没错，这样以来，图片素材就变得模糊了。同时因为Window使用的字体为点阵字体而非矢量字体，所以甚至在软件中的字体也会变得模糊。

再简单一点来说，采用这种技术需要将32x32的图片强制拉伸为48x48，多出来的像素如何凭空生成？计算机只有猜测了，通过线性插值算法。所以图片便会出现模糊。

![enlarged](./images/ppi/enlarged.jpg)

但位图可能会被拉伸的问题并非也是绝对的，假设软件需要显示的icon大小为32x32，但是图片素材大小为64x64，那么即使Windows的UI界面拉伸1.5倍，icon大小为48x48，因为原图片足够大，图片仍处于未拉伸的状态。那么也不会模糊。


反过来我们可以得出结论，为了让在低PPI上和高PPI上图片显示的效果一致，图片素材应该尽可能的高清。

Apple的Retina使用的就是上面的技术。这里可以拿来相提并论的是Retina版本的Macbook Pro，它使用的也是同样解决办法，以15.4寸的Retina 版Macbook Pro为例。显示器的物理像素点实际上有2880x1880，但默认的最优分辨率只有1440x900，刚好是物理像素的一半，也就是说操作系统默认就使用了4:1的缩放，那么和Windows一样会导致图片模糊虚化问题。

iPhone也是一样的原理，那么如果解决这个问题呢，苹果鼓励开发者准备两份素材，普通和高清素材（通过文件名称来区分，比如普通素材名称为apple.png，那么高清素材名称就为apple@2x.png），自然高清素材是普通素材面积的四倍，系统会优先使用高清素材，但自动缩小到普通素材的大小，这样图像也就更加细腻了，也就解决了图片被拉伸的问题。


## PPI之于Web

从上面我知道，因为高像素密度设备下的UI会采用一定比例的缩放，所以CSS像素也会面临同样的问题：

![css-device-bitmap-pixels](./images/ppi/css-device-bitmap-pixels.png)

正如上图所示，左侧普通屏幕中，2x2的CSS像素真的只需要2x2的物理像素。但是右侧高清屏中，2x2的CSS像素却需要4x4的物理像素。

上面说道，解决高清PPI下图片渲染问题的方法之一就是使用更高清的图片素材。但问题是需要有多高清？

在Retina显示屏上，根据上一节描述的原理，当我们需要渲染一张32x32的图片，我们实际上需要准备64x64的素材。因为苹果默认把所有素材都进行了两倍的放大。但如果有一台更高清的设备，进行了三倍或者四倍或者更高的倍数，我们岂不是需要准备体积更大的文件素材？在Web开发中我们正在面临这样的问题。

首先我们要学会如何表达和判断这样一种CSS像素和物理像素不平等。

### DevicePixelRatio

DevicePixelRatio定义如下：

```
window.devicePixelRatio = physical pixels / dips
```

分母dips全称为device-independent pixels，译为*与设备无关像素*。 更通俗的说是与物理像素无关的CSS像素。

以iPhone4为例，在垂直状态下手机的物理像素宽度有640px，但是因为2:1缩放的关系，此时的dip，也就是 CSS像素只有320px。 此时的DevicePixelRatio就为 640 / 320 = 2; 

注意devicePixelRatio并非是一个默认值。在默认情况下CSS像素是由手机默认的缩放决定的。但同时浏览器字体也可以被认为的进行缩放。iPhon4中默认的分辨率宽度为320px。我们完全可以自行放大两倍为160px。这样以来window.devicePixelRatio就变味了 640 / 160 = 4。

### dppx

与divicePixelRatio几乎等价的一个概念时dppx：dots per pixel。 表示单个CSS像素占用的物理像素个数。仔细想想，这与devicePixelRatio不是同一个意思吗, iPhone4的dppx为2，不就是与devicePixelRatio刚好相等吗。devicePixelRatio是从宏观上来说这件事。把整体宽度做运算。dppx是从微观角度上说这件事，考虑的是单个像素之间的比较

### dpi

请记住，当我们在谈论一台显示设备的像素密度时，dpi与ppi是等价的。dots per pixel中的dots就是代指物理像素。

但是如果你在mediaquery中使用dpi是就要注意了，Chrome会在控制台中提示你使用dppx而非dpi:

>Consider using 'dppx' units instead of 'dpi', as in CSS 'dpi' means dots-per-CSS-inch, not dots-per-physical-inch, so does not correspond to the actual 'dpi' of a screen. In media query expression: only screen and (-webkit-min-device-pixel-ratio: 2), not all, not all, only screen and (min-resolution: 192dpi), only screen and (min-resolution: 2dppx)
也就是实际生活中的一英寸。但是在mediaquery中inch表示的CSS定义中的一英寸。

实话实说我买有找到关于CSS中一英寸的定义，但是在W3C关于[Resolution](http://www.w3.org/TR/css3-values/#resolution)的定义中，我们可以看到看到它所定义的1dppx是与96dpi具有同样含义的。那么2dppx也就是192dpi了咯。这当然脱离了我们传统上的dpi了，Surface Pro 3的dpi（也就是ppi）能够达到216ppi，但是dppx仍然可以是1。

## CSS Reference Pixel

我们规定了CSS像素值需要与设备像素大小相等，但当随着手持设备距离人的远近不同，设备像素密度的不同，都会导致我们看见的设备上的CSS像素的可见大小发生变化（类似于巨大的月亮因为离地球遥远在人眼看来也不过像硬币一样大小）。为了保证CSS像素在不同设备和不同距离上观测到的大小保持一致保持连贯性。W3C定义了一个CSS相对像素（CSS reference pixel）的概念

> It is recommended that the reference pixel be the visual angle of one pixel on a device with a pixel density of 96dpi and a distance from the reader of an arm's length. For a nominal arm's length of 28 inches, the visual angle is therefore about 0.0213 degrees.


W3C规定，把人眼能够辨别到的，距离自己一个手臂长度（约28英寸），像素密度为96dpi设备上的一个物理像素设为参考像素。同时可以算出眼睛看到参考像素的视野角度为0.0213度

![pxangles](./images/ppi/pxangles.png)

有了这一系列参照，通过三角函数关系，我们可以算出同样一台设备在不同距离下CSS像素理想的大小。 当远离观察者时像素应该增大，当靠近观察者时像素应该减小。

![pixel1](./images/ppi/pixel1.png)

这么做的优势在于无论设备距离观察者距离是多少，也无论设备的像素密度和物理像素大小是多少，观察者看到的CSS像素是一致的，保证了用户体验的一致性：

![Fig-03-CSS-Reference-Pixel](./images/ppi/Fig-03-CSS-Reference-Pixel.jpg)



## `<meta name="viewport">`

我们有了物理像素，CSS像素——那么问题来了，当你再手机上使用浏览器打开网页时，网页的宽度应该是多少？

首先我们需要了解一个概念：viewport，我常见到的中文译为视口，但个人觉得这个翻译有一些晦涩。 Viewport是用于限制Html元素——“限制”这两个字不是那么好理解。quirksmode上有一篇[文章](http://www.quirksmode.org/mobile/viewports.html)谈到这个概念时打了一个非常形象的比方：

假设body标签内有一个块状元素宽度为10%: `div {width:10%;}`，我们知道当我们缩放浏览器时这个块状元素的宽度也会跟着变化。 这是因为它的宽度占它父元素的10%。那么它的父元素，也就是body元素的宽度是由谁决定的呢？

我们知道一个块状元素默认宽度为它父元素的100%，也就是body元素的宽度与包裹它的html元素宽度相同。那么问题又变成了html元素的宽度是由谁决定的？

答案是浏览器窗口。现在我们可以归纳起来，html元素是被浏览器限制并且包裹起来的。html的宽度就是浏览器的宽度。

但事实上，html元素宽度是占据viewport的100%，而在桌面浏览器中，viewport与浏览器窗口大小刚好相等。**注意，这仅仅是在桌面浏览器上**

OK，在于是我们得到了一个结论，html宽度是由viewport决定的，但是 在桌面浏览器中，viewport大小与浏览器窗口大小相等。

但这一套规则在手机则是无法被执行的。大部分手机的屏幕分辨率目测为400px，如果页面上真的有某一个页面元素仅占10%，也就是40px的话，肉眼几乎是无法分辨的。实际情况应该会更糟糕，iPhone4的Safari默认是以980px来渲染网页的。如果你在Chrome以桌面版的方式访问stackoverflow，那么结果会是这样的:

![iphone_stackoverflow](./images/ppi/iphone_stackoverflow.png)

体验非常糟糕吧，所以的链接几乎都无法准确点击。OK，那么如何解决这个问题？

第一个办法，放大页面。

我们会很习惯的用手势去放大页面。但是要注意我们这里做的仅仅是放大页面，改变的是页面的缩放(scale)，效果与PC上浏览器的Ctrl+"+"类似。但是没有改变页面的布局，此时用于渲染页面布局的layout仍然是980px

![iphone_stackoverflow_zoomin](./images/ppi/iphone_stackoverflow_zoomin.png)

第二个办法是，改变布局。
比如下面一个页面上有一张320px宽的图片，如果我们以默认的980px去渲染的话，它会显得过于窄小：

![vp980notspecified](./images/ppi/vp980notspecified.jpg)

但如果我们可以将渲染它的布局设为320px的话，看上去就会好很多了，同时此时我们也未对页面进行缩放：

![vp320width](./images/ppi/vp320width.jpg)

当然你也可以结合上一步，同时对页面进行缩放：

![vp320width150scale](./images/ppi/vp320width150scale.jpg)

不仅仅是放大，即使是在320px的像素下，我们也可以进行缩小：

![vp320width50scale](./images/ppi/vp320width50scale.jpg)

回归到技术上，以上这些都可以通过viewport标签来解决，比如说上面那个需求，把布局设定为320px，同时进行1.5倍的缩放

```
<meta name="viewport" content="width=320, initial-scale=1.5">
```

具体的设置方法如上图所示，所见即所得，需要设置的属性在content以逗号分割开来，`width`表示页面布局宽度，`initial-scale`代表页面初始状态的缩放比例，如果你不想让用户进行缩放，还可以添加`user-scalable=no`字段来保证用户无法进行缩放。

更重要的是，我们还可以通过设置布局宽度等于手机分辨率宽度来更好的利用响应式设计，比如`width=device-width`，注意这里的`device-width`表示手机的分辨率宽度，而并非手机物理像素宽度。iPhone4在垂直状态下物理像素宽度为640，这里的`device-width`代表的则应该是它的dip像素320px。

给viewport标签添加`width=device-width`适用于这样一种情况：你为移动设备开发的响应式网页，首先你会面临多重分辨率，但是你又没有必要使用到重量级的mediaquery，同时也为了避免手机浏览器使用桌面分辨率宽度去渲染页面，造成可用性的问题。这样让你的响应式页面能够适用大多数的移动设备。

你可能会进一步的以为在iPhone下设置`width=device-width`其实就是与`width=320`一个意思吧？不，你还需要考虑当用户将手机横握时，此时设备宽度就变成了480。但如果你将`width`设置成一个固定的数值时。如论用户手持设备的方向如何，都能保证用于渲染网页的宽度不变。

写到这里我们可以做一个总结，viewport标签的作用是什么？它能够让你撇开设备的干扰，告诉设备你想用什么样的宽度渲染网页。让它听命于你，而不是你听命于他。

上面我们谈到viewport有个半专业的名词成为layout viewport，顾名思义这个viewport专用于页面渲染的控制。还有一种viewport称之为visual viewport可以译为可视窗口。两种viewport的区分如下：

![mobile_layoutviewport](./images/ppi/mobile_layoutviewport.jpg)
![mobile_visualviewport](./images/ppi/mobile_visualviewport.jpg)

由此可以看出visual viewport就好比是浏览网页的一个窗口，网页正是这窗外的景色。当然我们还会遇见layout viewport与visual viewport大小相等的情况。比如像下面这样：

![mobile_viewportzoomedout](./images/ppi/mobile_viewportzoomedout.jpg)

这也就是我上面描述的`width=device-width`的情况了



## 番外篇：PPI和DPI使用的更多场景

在文章的开头我又说PPI在不同上下文中的含义是不同的，如果你仍有好奇心，可以继续往下阅读。接下来我们谈谈Web以外的PPI含义。

首先我们要重申上面的结论，就谈论显示设备的像素密度而已，PPI和DPI和一样的概念，并且其中的像素pixel和点dots代指的都是物理像素。

如果你去查看一张JPG图片的属性时，你会发现有横向或者纵向的以dpi为单位的属性：

![jpg_dpi](./images/ppi/jpg_dpi.jpg)

或者在Phototshop新建一份文档时，要填写一个以ppi为单位的属性值：

![photoshop-imagesize](./images/ppi/photoshop-imagesize.png)

这里也存在被混用和混淆的地方。其实他们都表示打印时的分辨率值。意为在打印时每英寸上的像素（也就是跟接近PPI）。这里的英寸当然不再是屏幕像素了，而是纸张尺寸了。

PPI或者DPI对于图片来说意味着什么？准确来说什么都不意味着。 一张图片只是存在相机或者硬盘里的数据文件而已，你能告诉我它有多少英寸长或者多少英寸宽吗？只有当它被打印出来的时候才会涉及到打印媒介的尺寸，PPI才有意义。 如果你想让图片更丰富，唯一的办法是增加图片的像素，提升你的拍摄技巧。

当然在纸张上是没有像素的概念。但我们可以去抽象的去想象它。假设有一张300x300像素的图片。打印分辨率的PPI值为30DPI,那么最后打印出来尺寸为10x10英寸。假如打印时的DPI值为300DPI，那么打印出来的尺寸则为1x1英寸。所以我们可以把DPI当做调节打印结果的手段。

那么DPI值越高，图片越小就越清晰？当然也并非如此。如果你距离60厘米去观看一张194DPI打印出来的图片。你会没法区分它到底是194DPI还是300DPI。因为人眼的分辨率是有限的。这对显示设备时同样通用的。iPhon4的像素密度有326DPI，而New iPad的像素密度只有264DPI，New iPad的显示效果会更差吗？参考大多数人使用的距离和方式，其实眼睛得到的效果其实是无太大差异的。这也是为什么大型显示器或者户外广告DPI都不会很高，因为我们观看他们的时候距离很远，效果并非太差。

最后我们可以来看另一个场景的DPI：描述打印机的打印分辨率：

当一张显示器上的图片打印在图片上的时候，像素这个概念其实是我们想象出来的，更加实际的概念时是印刷设备的每一个“点”：

![DPI_and_PPI](./images/ppi/DPI_and_PPI.png)

当你尝试去用放大镜去查看彩色印刷物品上的图片时，从小到大你看到的结果应该是这样的：

![zoom_pic](./images/ppi/zoom_pic.gif)

为什么会这样？简而言之，印刷的原理是通过半色调(halftone)技术，通过控制CMYK四种颜色点印刷时的每一个印刷点的大小，角度，间隙来模拟出一种颜色的感觉：

![color-halftoning](./images/ppi/color-halftoning.png)

比如当你以600DPI打印一张150PPI的图片时，每一个像素应该包含16个点(600dots / 150pixels = 4)。

从上面我们已经知道PPI能够决定印刷品物理尺寸的大小，打印机的DPI参数更是能进一步决定印刷体的好坏。法则一
永远都是在追求更高的PPI和DPI。

. 150dpi is generally considered the minimum standard for high quality photographic reproduction in books and magazines. Newspapers often use 85dpi and the effect is clear: individual dots are visible and some detail is lost. Billboards go as low as 45dpi, but you can’t tell because you’re typically viewing from very far away. Typical dot matrix printers are capable for 60 – 90dpi, inkjet printers 300 – 600dpi, and laser printers 600 – 1,800dpi.

150dpi通常已经是被认为算的上是高质量的打印分辨率了。新闻报纸使用的分辨率通常是85dpi。户外的广告牌通常使用的是45dpi。但是因为距离的关系一般
















