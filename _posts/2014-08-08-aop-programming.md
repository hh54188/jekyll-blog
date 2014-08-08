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

![duck](../images/aop-programming/duck.jpg);

这个检测看上去似乎有一些无厘头，但可以用来解决另一个问题——对于Javascript或者类似的动态语言，如何保证实现“接口”或者“基类”的实现呢？Stackoverflow上的这个问题[Does JavaScript have the interface type (such as Java's 'interface')?](http://stackoverflow.com/questions/3710275/does-javascript-have-the-interface-type-such-as-javas-interface)就谈到了这个问题。 针对Javascript的类似情况，我们就可以使用duck typing这个方法。对于check是否是我们需要的接口是否是我们需要的，是否符合我们需要的那种类型，我们可以检查接口的类型或者参数是否是我们需要的：

```
var quack = someObject.quack;

if (typeof quack == "function" && quck.length == arguLength)
{
    // This thing can quack
}

```

扯远了，其实我想表达的duck punch其实是由duck typing演化而来的：

>if it walks like a duck and talks like a duck, it’s a duck, right? So if this duck is not giving you the noise that you want, you’ve got to just punch that duck until it returns what you expect.

当你想一只鸭子发出驴的叫声怎么办，揍到它叫为止……话说这让我想到一个笑话：

>为了测试美国、香港、中国大陆三地警察的实力, 联合国将三只兔子放在三个森林中，看三地警察谁先找出兔子。任务：找出兔子。
(中间省略......)
最后是某国警察，只有四个，先打了一天麻将，黄昏时一人拿一警棍进入森林，没五分钟，听到森林里传来一阵动物的惨叫，某国警察一人抽着一根烟有说有笑的出来，后面拖着一只鼻青脸肿的熊，熊奄奄一息的说到：“不要再打了，我就是兔子……”

虽然duck punch有些暴力，