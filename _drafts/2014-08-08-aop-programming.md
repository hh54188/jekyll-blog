---
layout: post
title:	聊Javascript中的AOP编程
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-03-13
tags: [javascript, front-end]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## Duck punch

我们先不谈AOP编程，先从duck punch编程谈起。

如果你去wikipedia中查找duck punch，你查阅到的应该是[monkey patch](http://en.wikipedia.org/wiki/Monkey_patch)这个词条。根据解释，Monkey patch这个词来源于*guerrilla patch*，意为在运行中悄悄的改变代码，而*guerrilla*这个词语*gorilla*同音，而意又为monkey相近（前者为“猩猩”的意思），最后就演变为了monkey patch。

如果你没有听说过duck punch，但你或许听说过duck typing。举一个通俗的例子，如何辨别一只鸭子： 

>When I see a bird that walks like a duck and swims like a duck and quacks like a duck, I call that bird a duck.

没错，如果我发现有一类动物像鸭子一样叫，像鸭子一样游泳，那么它就是一只鸭子！

![duck](../images/aop-programming/duck.jpg)

这个检测看上去似乎有一些无厘头，但可以用来解决另一个问题——对于Javascript或者类似的动态语言，如何保证实现“接口”或者“基类”的实现呢？Stackoverflow上的这个问题[Does JavaScript have the interface type (such as Java's 'interface')?](http://stackoverflow.com/questions/3710275/does-javascript-have-the-interface-type-such-as-javas-interface)就谈到了这个问题。 针对Javascript的类似情况，我们就可以使用duck typing这个方法。对于check是否是我们需要的接口是否是我们需要的，是否符合我们需要的那种类型，我们可以检查接口的类型或者参数是否是我们需要的：

{% highlight javascript %}
var quack = someObject.quack;

if (typeof quack == "function" && quck.length == arguLength)
{
    // This thing can quack
}

{% endhighlight %}

扯远了，其实我想表达的duck punch其实是由duck typing演化而来的：

>if it walks like a duck and talks like a duck, it’s a duck, right? So if this duck is not giving you the noise that you want, you’ve got to just punch that duck until it returns what you expect.

当你想一只鸭子发出驴的叫声怎么办，揍到它叫为止……话说这让我想到一个笑话：

>为了测试美国、香港、中国大陆三地警察的实力, 联合国将三只兔子放在三个森林中，看三地警察谁先找出兔子。任务：找出兔子。
(中间省略......)
最后是某国警察，只有四个，先打了一天麻将，黄昏时一人拿一警棍进入森林，没五分钟，听到森林里传来一阵动物的惨叫，某国警察一人抽着一根烟有说有笑的出来，后面拖着一只鼻青脸肿的熊，熊奄奄一息的说到：“不要再打了，我就是兔子……”

又跑题了，虽然duck punch有些暴力，但不失为一个有效的方法。落实到代码上来说就是**让原有的代码兼容我们需要的功能**。比如Paul Irish博客上的这个例子：

{% highlight javascript %}
/**
    我们都知道jQuery的`$.css`方法可以通过使用颜色的名称给元素进行颜色赋值。
    但jQuery内置的颜色并非是那么丰富，如果我们想添加我们自定义的颜色名称应该怎么办？比如我们想添加`Burnt Sienna`这个颜色
*/

(function($){
    
    // 把原方法暂存起来：
    var _oldcss = $.fn.css;

    // 重写原方法：
    $.fn.css = function(prop,value){

        // 把自定义的颜色写进分支判断里，特殊情况特殊处理
        if (/^background-?color$/i.test(prop) &amp;&amp; value.toLowerCase() === 'burnt sienna') {
           return _oldcss.call(this,prop,'#EA7E5D');

        // 一般情况一般处理，调用原方法
        } else {
           return _oldcss.apply(this,arguments);
        }
    };
})(jQuery);

// 使用方法：
jQuery(document.body).css('backgroundColor','burnt sienna')
{% endhighlight %}

可以看出`duck punch`的模式不过如此：

{% highlight javascript %}
(function($){

    var _old = $.fn.method;

    $.fn.method = function(arg1,arg2){

        if ( ... condition ... ) {
           return  ....
        } else {           // do the default
           return _old.apply(this,arguments);
        }
    };
})(jQuery);
{% endhighlight %}

**但是这么做有一个问题：需要修改原方法。**这违背了“开放-封闭”原则，本应对拓展开放，对修改关闭。怎么解决这个问题呢？使用AOP编程。

## AOP

AOP全称为`Aspect-oriented programming`，很明显这是相对于`Object-oriented programming`而言。`Aspect`可以翻译为“切面”或者“侧面”，也就是面向切面编程。

怎么理解切面？

在面向对象编程中，当我们定义的类通常是领域模型，它的拥有的方法通常是和纯粹的业务逻辑相关。比如：
{% highlight java %}
Class Person
{
    private int money
    public void pay(price)
    {
         this.money = this.money - price;   
    }
}
{% endhighlight %}

但通常实际情况会更复杂，比如我们需要在付款的`pay`方法中加入授权检测，或者用于统计的日志发送，甚至容错代码。于是代码会变成这样：

{% highlight java %}
Class Person
{
    private int money
    public void pay(price)
    {
        try 
        {
            if (checkAuthorize() == true) {
                this.money = this.money - price;    
                sendLog();
            }
        }
        catch (Exception e)
        {

        }   
    }
}
{% endhighlight %}

更可怕的是，其他的方法中也要添加相似的代码，这样以来代码的可维护性和可读性便成了很大的问题。我们希望把这些零散但是公共的非业务代码收集起来，更友好的使用和管理他们，这便是切面编程。切面编程在避免修改远代码的基础上实现了代码的复用。就好比把不同的对象横向剖开，关注于改造内部方法，而非全局。



其实上面

## 参考文献：

- [How to Fulfill Your Own Feature Request -or- Duck Punching With jQuery!](http://www.paulirish.com/2010/duck-punching-with-jquery/)
- [Duck Punching JavaScript - Metaprogramming with Prototype](http://www.ericdelabar.com/2008/05/metaprogramming-javascript.html)

