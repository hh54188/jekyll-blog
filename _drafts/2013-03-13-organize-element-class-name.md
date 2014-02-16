---
layout: post
title: 如何组织Html元素与如何进行CSS命名
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

最近下决心整理一份对页面元素的组织规则和CSS的命名规则，因为深深感受到如果页面上元素太多，没有规则的命名和组织会让网页的维护性大打折扣。参考了几篇文章并且发散了一下，在这里和大家分享


## [BEM(Block Element Modifier)](http://bem.info/method/definitions/)


如何组织页面上的元素，或者说安排元素之间的关系势必会对css命名产生影响；css命名也是对元素关系的映射。BEM这个方法把元素分为三类，代指 Block , Element , Modifier 。

举个例子，通常我们会把页面分为header, body, footer部分，可能header部分里面又包括了logo, search, login模块。我们可以把Block理解为一个已经封装好了的组件，比如一个搜索模块（这里说的模块统统指一系列html元素，而非逻辑上的功能代码），它是相对于同级元素比较独立的（也相对于element元素）。

Element则是block中实现具体功能的部件，比如在搜索模块中最起码需要一个按钮(button)，一个输入框(input)，它和block的重要区别之一是，它没有block那么独立，一旦离开了上下文环境（比如它所在的block），它失去功能上的意义了。

这就非常灵活了，因为某一个block可以是它父元素或者其他元素的element，比如我们单独看搜索模块可能是一个独立的block，但是搜索模块通常又是放在页面的header中，那么搜索模块此时又成了header的element，而header又是一个更大的block。

接下来，BEM作者给出了CSS的具体命名规则希望是

一个block必须有唯一的名字(class)，比如

{% highlight html %}
<ul class="menu">
  …
</ul>   
{% endhighlight %}

而element的命名需要包括它所属的block的名字，并且以分隔符分隔开:

{% highlight html %}
<ul class="menu">
  <li class="menu__item">…</li>
  <li class="menu__item">…</li>
</ul>   
{% endhighlight %}

当然modifier就更好理解了，比如在一个tab模块中，我们需要突出某个tab，就需要给它添加一个的modifier作用的class，这里的modifier可以使特殊的状态，也可以是特殊的属性。就拿上面那个menu例子来说，我们想增大字体，想标识当前选中的菜单项，就可以添加

{% highlight html %}
<ul class="menu menu_size_big">
  <li class="menu__item">Index</li>
  <li class="menu__item menu__item_state_current">Product</li>
  <li class="menu__item">Contact</li>
</ul>
{% endhighlight %}

巧合的是，在写这篇文章的同时Smashing Magazine上同时发布了一篇谈BEM现在与将来的文章[The Evolution Of The BEM Methodology](http://coding.smashingmagazine.com/2013/02/21/the-history-of-the-bem-methodology/)。主要对BEM过去的里程碑，每个里程碑所得出的一些方法论做了一些总结。比如说谈到BEM的起源其实是为了解决实际项目中css选择器冗长的问题，比如

{% highlight css %}
.result .albums .album .buy
{
    float: left;
    padding: 0.4em 1em 0 1.6em;
}

.result .albums .info i
{
    font-size: 85%;
}
{% endhighlight %}

有甚者

{% highlight css %}
.b-foot div div div div div
{
 background-position: 71%;
 background-image: url(../i/foot-5.png);
}

.b-foot div div div div div div
{
 background-position: 87%;
 background-image: url(../i/foot-6.png);
}
{% endhighlight %}

虽然现在看起来很可笑，但我觉得这却是实际中遇见的问题，必须承认我自己有时也陷入这样的怪圈

这篇文章谈为什么有BEM的来龙去脉更生动一些。有兴趣的同学可以看看。其实BEM是一系列的方法论，甚至还包括文件的命名的文件夹分类规则，XSL templates，甚至整个可供参考的框架。

因为在这里我只是作为一个组织html元素和css命名的其中一个方法，只做了简明的介绍和总结。


## [SMACSS(Scalable and Modular Architecture for CSS)](http://smacss.com/book/categorizing)


这个方法论将css分为5类，分别是

- Base
- Layout
- Module
- State
- Theme(忽略这个先)

下面一一进行介绍

Base:

基本(base)规则即那些只使用元素选择器，后代选择等（从不涉及class或者id）的规则，比如

{% highlight css %}
body, form {
    margin: 0;
    padding: 0;
}

a:hover {
    color: #03F;    
}
{% endhighlight %}

你可以理解为定义一些全局的css样式。通常这种工作也可以交给reset.css或者normalize.css来完成


Layout Rules:

这里的布局(Layout)指页面上比较大块的区域，比如header，body， footer。而这里的layout rule也分为两类，一类是通过id定义的，比如

{% highlight css %}
#header, #article, #footer {
    width: 960px;
    margin: auto;
}

#article {
    border: solid #CCC;
    border-width: 1px 0 0;
}
{% endhighlight %}

还有一类可能是在你用了一些css框架的情况下，比如960.gs

{% highlight css %}
.container_12 .grid_6,
.container_16 .grid_8 {
  width: 460px;
}
{% endhighlight %}

作者建议与layout有关的css规则以l-开头，比如

{% highlight css %}
.l-fixed #sidebar {
    width: 200px;
}
{% endhighlight %}

Module Rules:

比如说一些登陆，搜索，文章，这样的元素组合我们就可以称之为module。对于这样一些元素的css命名，作者说就免了前缀，直接用模块名称好了，比如

{% highlight css %}
.login {
    width: 200px;
}
{% endhighlight %}

作者在这里强调的是，避免使用元素选择器。比如开始我们有这么一段html， 有这么一段样式

{% highlight html %}
<div class="fld">
    <span>Folder Name</span>
</div>

/* The Folder Module */
.fld > span {
    padding-left: 20px;
    background: url(icon.png);
}
{% endhighlight %}

问题是当我们的项目变得庞大，需要增加一个或者更多span标签时

{% highlight html %}
<div class="fld">
    <span>Folder Name</span>
    <span>(32 items)</span>
</div>
{% endhighlight %}

这会就傻×了吧。所以最好是给标签添加上有语义的class名称，比如

{% highlight html %}
<div class="fld">
    <span class="fld-name">Folder Name</span>
    <span class="fld-items">(32 items)</span>
</div>
{% endhighlight %}

还有一种情况，当我们在不同的section中使用了同一个module时，我们可能需要根据module所在的section来重新定义样式，比如

{% highlight css %}
.pod { 
    width: 100%; 
}
.pod input[type=text] { 
    width: 50%; 
}
#sidebar .pod input[type=text] { 
    width: 100%; 
}
{% endhighlight %}

但这样还是会产生问题，会让css变得没有规则和难以维护，所以作者建议添加一个子模块css(Subclassing Modules)，比如这么做

{% highlight html %}
<div class="pod pod-constrained">...</div>
<div class="pod pod-callout">...</div>
.pod { 
    width: 100%; 
} 
.pod input[type=text] { 
    width: 50%; 
}
.pod-constrained input[type=text] { 
    width: 100%; 
}
.pod-callout { 
    width: 200px; 
}
.pod-callout input[type=text] { 
    width: 180px; 
}}
{% endhighlight %}

State Rules

state 与之前的modifier概念类似，这种类型的class只起一些修饰作用。并且作者建议使用is-开头，比如

{% highlight html %}
<div id="header" class="is-collapsed">
    <form>
        <div class="msg is-error">
            There is an error!
        </div>
        <label for="searchbox" class="is-hidden">Search</label>
        <input type="search" id="searchbox">
    </form>
</div>
{% endhighlight %}

要注意它和之前sub-module的区别

state规则给layout或者module用都行
state规则通常由javascript有关（比如错误，高亮，是否折叠），而sub-module是静态不能随意修改的。只是为了区分模块之间的区别
在这个方法论的文章中，我觉得最有价值的一篇是谈到css的可访问性的。首先作者给出的两条建议是

css不应该依赖DOM树的结构
css选择器不宜太深
假如我们有这么一段css

{% highlight css %}
#sidebar div {
    border: 1px solid #333;
}

#sidebar div h3 { 
    margin-top: 5px;
}

#sidebar div ul {
    margin-bottom: 5px; 
}
{% endhighlight %}

这段css中的div可以看做一个组件，是由h3和ul组成的。如果我们想把这个组件又放在footer中怎么办，看下面这段代码怎么样

{% highlight css %}
#sidebar div, #footer div {
    border: 1px solid #333;
}

#sidebar div h3, #footer div h3 { 
    margin-top: 5px;
}

#sidebar div ul, #footer div ul {
    margin-bottom: 5px; 
} 
{% endhighlight %}

代码这么冗余的原因就是因为它与dom联系的太紧密，不如把这层依赖关系去掉，给它加上一个独立的class

{% highlight css %}
.pod {
    border: 1px solid #333;
}

.pod > h3 { 
    margin-top: 5px;
}

.pod > ul {
    margin-bottom: 5px; 
} 
{% endhighlight %}

我们试图在可维护性，性能，和可读性三者之间保持平衡，虽然css选择器具有一定深度意味着更少的class，但是它增加了可维护性和可读性。除非你压根就不想使用class。


Nicolas Gallagher可以说是一位大神级的人物。这篇文章About HTML semantics and front-end architecture是他它对css组织和命名的一些看法，也相当值得我们一读。

这篇文章主要是谈html的语义(semantics)化。比如他反对class的命名以内容为驱动(content-derived)，比如这样

{% highlight html %}
<div class="news">
    <h2>News</h2>
    [news content]
</div>
{% endhighlight %}

因为容器的内容是news所以class就要是news？他认为class具有以下几个特征

其实以内容为驱动的语义化在html标签上已经体现出来了，比如说标题使用h标签，文章使用p标签
class名主要的用处是为css和javascript服务，如果你既不需要添加样式又不需要再js中引用，那要id和class有什么用
class名取的再怎么恰当，有意义，对机器来说是没有任何意义的
class名应该在开发者之间传递有用的信息
另一种取class名的作法是，从网页的架构和功能上来区分。最具复用性的组件的名字往往是脱离内容的。

前端架构中，模板(template)，组件(component)，面向对象(object-oriented)的意义就在于，开发有限的可复用的模块来容纳不同的内容。所以以编程为驱动（而不是以内容为驱动）的class名称主要的目的也应该是，提供有意义的，灵活的，可复用的表现层/行为层的hooks（怎么翻译？）给开发者使用

一个灵活的，具有复用性的组件既不能对特定的dom树结构有依赖，也不能指定特定的元素类型来承载。它应该被所有容器所接纳，并且可以被任意的主题化。

应该避免使用css的类型选择器，比如下面这个例子

{% highlight html %}
<nav class="uilist">
    <a href="#">Home</a>
    <a href="#">About</a>
    <a class="btn" href="#">Login</a>
</nav>
.btn { /* styles */ }
.uilist { /* styles */ }
.uilist a { /* styles */ }  
{% endhighlight %}

首先因为css优先级的规则关系，.btn会被.uilist a的样式覆盖；其次，uilist这个组件始终需要一个a标签嵌套在其中。

所以最好的作法应该是指定特定的class名称

{% highlight html %}
<nav class="uilist">
    <a class="uilist-item" href="#">Home</a>
    <a class="uilist-item" href="#">About</a>
    <span class="uilist-item">
        <<a class="btn" href="#">Login</a>
    </span>
.btn { /* styles */ }
.uilist { /* styles */ }
.uilist-item { /* styles */ }
{% endhighlight %}

Component modifiers

没错，这里的modifier和bem中的modifier是一致的概念。

通常组件也会有一些变种，比如一个按钮我们需要不同的背景色和边框。主要有两种模式来创建这样的组件

single-class

{% highlight html %}
<button class="btn">Default</button>
<button class="btn-primary">Login</button>
.btn, .btn-primary { /* button template styles */ }
.btn-primary { /* styles specific to save button */ }
{% endhighlight %}

{% highlight html %}
multi-class
<button class="btn">Default</button>
<button class="btn btn-primary">Login</button>
.btn { /* button template styles */ }
.btn-primary { /* styles specific to primary button */ }
{% endhighlight %}

作者推荐的是第二种。考虑一种情况，比如当我需要统一调整btn组件时，第一种方法需要这么修改

{% highlight css %}
/* "single-class" adjustment */
.thing .btn,
.thing .btn-primary,
.thing .btn-danger,
.thing .btn-etc { /* adjustments */ }
{% endhighlight %}

而第二种方法只需要这么修改

{% highlight css %}
/* "multi-class" adjustment */
.thing .btn { /* adjustments */ }
{% endhighlight %}

之前的想法是想法是想把人人和淘宝的规则再分别整理一遍，后来发现，既然已经成规则，他们的文档已经很抽象很精辟，没有办法再再整理了，如果真的要整理的话只能直接复制粘贴了。所以还是都不在这里一一展示了。

支付宝前端样式解决方案:

《样式库构建规范》

《CSS 规范》

人人FED CSS编码规范

它们的命名规则和之前的几篇文章是一致的，比如对每一个组件都单独命名，绝不使用元素或者类型选择器。除了本系列文章谈的一些原则可以得到映射。我们能从这些规范里学习到的更多是细节……太多太多细节了！按照这些细节来书写css，保证团队里每个人书写的css绝对都是一样的，也绝对都能互相看懂，可维护性太棒了！