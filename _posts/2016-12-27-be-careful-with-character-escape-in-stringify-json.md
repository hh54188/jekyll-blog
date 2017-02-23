---
layout: post
title: #短篇#小心字符串中的转义字符
modified: 2016-12-27
tags: [javascript, short]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

这是一篇遇见bug的反思。

## 问题

首先请听我描述一段业务上的需求。

有一些信息是需要客户填写，比如说录入完商品之后需要填写商品的描述信息。这些信息存储在后台，在需要时由前端请求。但出于某种原因后端只能返回字符串，而前端需要JSON，于是妥协的结果后端返回的是stringify之后的json对象的字符串。

比如我们需要的是：

{% highlight javascript %}
{
	"name": "banana"
}
{% endhighlight %}

而后端返回的是

{% highlight javascript %}
'{"name": "banana"}'
{% endhighlight %}

然后我们再用`JSON.parse`方法将字符串还原为JSON对象用于使用。

一切都很完美，直到最近遇到的这个问题出现：客户可能在商品的描述中加入英文双引号`"`。于是想当然的，我们会在存储该信息的时候对双引号进行转义。例如当用户输入的是`"Hello"`时，我们在存储时会存储为`\"Hello\"`。

问题来了：假设后端存储的信息是`{"name": "\"banana\""}`，那么它返回的字符串也是`{"name": "\"banana\""}`，但是对于javascript来说返回（console打印的结果也是）的字符串却是`{"name": ""banana""}`，于是调用`JSON.parse`方法时报错：

{% highlight javascript %}
JSON.parse('{"name": ""banana""}')
{% endhighlight %}

而之所以报错是因为双引号的缘故。但我知道你其实想问的是，用于转义的反斜杠`\`哪里去了？

## 为什么需要转义

不妨让我们从头捋一捋，为什么需要转义。

转义的原因无非有两种（改编自[百度百科](http://baike.baidu.com/link?url=83ELEvD7roYOxo33tnu7MpuOEegyQaGUz9rORLzSoU523YhoR48PhEf2PFeGzo7r08geKtw60bVT8MkAFmR9g_)）
1. 使用转义字符来表示字符集中定义的字符，如Javascript中定义了一些字母前加"\"来表示常见的那些不能显示的（也就是换行，缩进，符号等）ASCII字符，如\0,\t,\n等，就称为转义字符，因为后面的字符，都不是它本来的ASCII字符意思了。
2. **某一些特定的字符在编辑语言中被定义为特殊用途的字符。这些字符由于被定义为特殊用途，它们失去了原有的意义。**例如双引号在Javascript用于标注字符串。但如果字符串中就自带双引号。需要把自带的双引号和用于标注的双引号进行区分。于是需要对自带双引号进行转义，也就是加上反斜杠。

- 双引号用于标注字符串：

{% highlight javascript %}
var str = "Hello"
{% endhighlight %}

- 如果字符串中也带了双引号，就会发生歧义：

{% highlight javascript %}
var str = "Hello""
{% endhighlight %}
- 于是需要对字符串内的双引号进行转义，也就是加上反斜杠，告诉脚本引擎要区分对待：

{% highlight javascript %}
var str = "Hello\""
{% endhighlight %}

请注意加上反斜杠进行转义，只是为了在**书写代码**时有所区分，骨子里转义双引号`\"`仍然还是双引号`"`。例如你可以运行下面这段代码：

{% highlight javascript %}
var str = "\"\"\"";
console.log(str); // """
{% endhighlight %}

上面代码中在打印时，反斜杠已经不存在了，反斜杠存在的意义是为了保证双引号不被误解。

使用单引号也能达到同样的目的，此时也就不需要进行转义了：

{% highlight javascript %}
var str = '"""';
console.log(str); // """
{% endhighlight %}

这其实能得出一个吊诡的结论：一个对象原封不动的转化为字符串后，这个字符串竟然不能还原为对象：

{% highlight javascript %}
// 原始object对象
var obj = {
	key: '\"Hello World\"'
}

var str = JSON.stringify(obj); // 转化为字符串
console.log(str)
// {"key":"\"Hello World\""}


var o = JSON.parse(str) // 还原为object时出错
{% endhighlight %}


## 解决问题

回到开始的那个问题。我们现在知道为什么当后端返回给我们`{"name": "\"banana\""}`时，我们实际上得到的是`{"name": ""banana""}`了。对脚本引擎来说，`\"`与`"`是一样的，这样的字符串当然不能被`JSON.parse`。

那么什么样的字符串才能被`JSON.parse`解析？后端返回字符串经过脚本引擎处理后仍然保留反斜杠的字符串：`{"name": "\"banana\""}`

反过来推算，如果我们想字符串能被`JSON.parse`，则需要`\"`中的反斜杠得以保留，也就是最终的结果应该为`JSON.parse(`{"name": "\"banana\""}`)`，如果想反斜杠得以保留，则需要反斜杠不被用于转义，则需要在反斜杠之前再加反斜杠将其转义以防止它被用于转义（是不是有点绕，好好理解一下）。最后得出结论，后端存储时，应该存储的内容是：

{% highlight javascript %}
{"name": "\\"banana\\""}
{% endhighlight %}

## 结论

我的忠告是，在处理类似场景时千万小心，仔细判断是否需要对反斜杠进行转义。当然上上策还是直接使用JSONP，而不是把对象压缩为字符串。



