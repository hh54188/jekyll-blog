---
layout: post
title: 从onload和DOMContentLoaded谈起
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-02-27
tags: [javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

当onload事件触发时，页面上所有的DOM，样式表，脚本，图片，flash都已经加载完成了。

当DOMContentLoaded事件触发时，仅当DOM加载完成，不包括样式表，图片，flash。

我们需要给一些元素的事件绑定处理函数。但可能出现一种情况，如果那个元素还没有加载到页面上，绑定事件却已经执行完了。这两个事件大致就是用来避免这样一种情况，保证在页面的某些元素加载完毕之后再绑定事件的函数。

当然DOMContentLoaded机制更加合理，因为我们可以容忍图片，flash延迟加载，却不可以容忍看见内容后页面不可交互。

大家可以从这个[微软提供的例子](http://ie.microsoft.com/testdrive/HTML5/DOMContentLoaded/Default.html)看到很明显的区别。

在没有出现DOMContentLoaded事件出现以前，许多类库中都有模拟这个事件的方法，比如jQuery中著名的`$(document).ready(function(){})`;



## 首先看一些DOMContentLoaded的特殊情况

虽然文档称该事件仅当在DOM加载完成之后触发，实际上并非如此。

在某些版本的Gecko和Webkit引擎的浏览器中，会使外链样式的加载完成也成为触发DOMContentLoaded事件的条件之一。最普遍的情况是`<script src="">`跟在一个`<link rel="stylesheet">`之后，无论这个script标签是在head还是在body中，比如：

Html:

{% highlight html %}
<!DOCTYPE html>
<head>
<link rel="stylesheet" href="stylesheet.css">
<script src="script.js"></script>
</head>
<body>
<div id="element">The element</div>
</body>
{% endhighlight %}


stylesheet.css:

{% highlight css %}
#element { color: red; }
{% endhighlight %}

script.js

{% highlight javascript %}
document.addEventListener('DOMContentLoaded', function () {
    alert(getComputedStyle(document.getElementById('element'), null).color);
}, false);
{% endhighlight %}

你可以尝试强制服务器端使样式表延迟一段时间才加载（甚至10秒），测试的结果是，仍然可以在DOMContentLoaded事件的回调中读出元素样式，比如#FF0000或者rgb(255, 0, 0)。说明在事件发生之前样式表已经加载完成了。而在opera中却无法读出style的属性。


把脚本外链把样式外链之后已经是一种通用的作法，甚至在jquery的官方文档中也是这样推荐的

其实对大部分脚本来说，这样的脚本等待外链的机制还是有意义的，比如一些DOM和样式操作需要读取元素的位置，颜色等。这就需要样式先于脚本加载。



**加载样式表会阻塞外链脚本的执行**

一些Gecko和Webkit引擎版本的浏览器，包括IE8在内，会同时发起多个Http请求来并行下载样式表和脚本。但脚本不会被执行，直到样式被加载完成。在未加载完之前甚至页面也不会被渲染。你可以在frebug或者Chrome的web developer中验证:

![block](../images/tabk_about_domcontentloaded-technical/chrome-timeline-my.png)

但是在opera中样式的加载不会阻塞脚本的执行。有一些类库中模拟dom ready的行为中会把这个“意外”修正为与firefox和chrome类似。

附带一句，在Explorer和Gecko中，样式的加载同样也会阻塞直接写在页面上的脚本的执行（脚本接在样式表之后）。在Webkit和Opera中页面上的脚本会被立即执行。

## javascript框架是如何实现自己的dom ready事件的？

### webkit

如果是webkit引擎则轮询document的readyState属性，当值为loaded或者complete时则触发DOMContentLoaded事件

{% highlight javascript %}
if (Browser.Engine.webkit) {  
    timer = window.setInterval(function() {  
    if (/loaded|complete/.test(document.readyState))  
    fireContentLoadedEvent();  
}, 0);
{% endhighlight %}

对webkit引擎还有一个办法是，因为webkit在525以上的版本中才开始引入了DOMContentLoaded事件，那么你可以对webkit的引擎版本进行判断，如果在525之下就用上面轮询的办法，如果在525之上，则直接注册DOMContentLoaded事件吧。
因为DOMContentLoaded事件最早其实是firefox的私有事件，而后其他的浏览器才开始引入这一事件。所以对火狐浏览器无需多余的处理

### IE

方法一：在页面临时插入一个script元素，并设置defer属性，最后把该脚本加载完成视作DOMContentLoaded事件来触发。

{% highlight javascript %}
document.write("<"+"script id=__onDOMContentLoaded defer src=//:><\/script>");  
$("__onDOMContentLoaded").onreadystatechange = function() {  
  if (this.readyState == "complete") {  
    this.onreadystatechange = null;  
    fireContentLoadedEvent();  
  }  
}; 
{% endhighlight %}

但这样做有一个问题是，如果插入脚本的页面包含iframe的话，会等到iframe加载完才触发，其实这与onload是无异的。

方法二：通过setTiemout来不断的调用documentElement的doScroll方法，直到调用成功则出触发DOMContentLoaded

{% highlight javascript %}
var temp = document.createElement('div');  
(function(){  
    ($try(function(){  
        temp.doScroll('left');  
        return $(temp).inject(document.body).set('html', 'temp').dispose();  
    })) ? domready() : arguments.callee.delay(50);  
})();
{% endhighlight %}

这样做的原理是

在IE下，DOM的某些方法只有在DOM解析完成后才可以调用，doScroll就是这样一个方法，反过来当能调用doScroll的时候即是DOM解析完成之时，与prototype中的document.write相比，该方案可以解决页面有iframe时失效的问题。

方法三：首先注册document的onreadystatechange事件，但经测试后该犯方法与window.onload相当

{% highlight javascript %}
document.attachEvent("onreadystatechange", function(){  
    if ( document.readyState === "complete" ) {  
        document.detachEvent( "onreadystatechange", arguments.callee );  
        jQuery.ready();  
    }  
});
{% endhighlight %}

接下来具体看一看几大前端框架是如何综合运用这几个方法的。

### jquery 1.9.0

{% highlight javascript %}
jQuery.ready.promise = function( obj ) {
    if ( !readyList ) {
        readyList = jQuery.Deferred();
        // Catch cases where $(document).ready() is called after the browser event has already occurred.
        // we once tried to use readyState "interactive" here, but it caused issues like the one
        // discovered by ChrisS here: http://bugs.jquery.com/ticket/12282#comment:15
        if ( document.readyState === "complete" ) {
            // Handle it asynchronously to allow scripts the opportunity to delay ready
            setTimeout( jQuery.ready );
        // Standards-based browsers support DOMContentLoaded
        } else if ( document.addEventListener ) {
            // Use the handy event callback
            document.addEventListener( "DOMContentLoaded", DOMContentLoaded, false );
            // A fallback to window.onload, that will always work
            window.addEventListener( "load", jQuery.ready, false );
        // If IE event model is used
        } else {
            // Ensure firing before onload, maybe late but safe also for iframes
            document.attachEvent( "onreadystatechange", DOMContentLoaded );
            // A fallback to window.onload, that will always work
            window.attachEvent( "onload", jQuery.ready );
            // If IE and not a frame
            // continually check to see if the document is ready
            var top = false;
            try {
                top = window.frameElement == null && document.documentElement;
            } catch(e) {}
            if ( top && top.doScroll ) {
                (function doScrollCheck() {
                    if ( !jQuery.isReady ) {
                        try {
                            // Use the trick by Diego Perini
                            // http://javascript.nwbox.com/IEContentLoaded/
                            top.doScroll("left");
                        } catch(e) {
                            return setTimeout( doScrollCheck, 50 );
                        }
                        // and execute any waiting functions
                        jQuery.ready();
                    }
                })();
            }
        }
    }
    return readyList.promise( obj );
};
{% endhighlight %}

具体分析如下

首先如果浏览器拥有`.readystate`

{% highlight javascript %}
if ( document.readyState === "complete" ) {
    // Handle it asynchronously to allow scripts the opportunity to delay ready
    setTimeout( jQuery.ready );
}
{% endhighlight %}

我不确定这样的延迟起到了什么样的作用。希望有经验的朋友能指点一下

再者，如果浏览器支持DOMContentLoaded的话

{% highlight javascript %}
if ( document.addEventListener ) {
  // Use the handy event callback
  document.addEventListener( "DOMContentLoaded", DOMContentLoaded, false );

  // A fallback to window.onload, that will always work
  window.addEventListener( "load", jQuery.ready, false );
}
{% endhighlight %}

注意，它在最后还是给load事件注册了事件，以防不测，做为回滚用。

IE
首先它给onreadystatechange和onload事件注册了方法，作为fallback

{% highlight javascript %}
// Ensure firing before onload, maybe late but safe also for iframes
document.attachEvent( "onreadystatechange", DOMContentLoaded );

// A fallback to window.onload, that will always work
window.attachEvent( "onload", jQuery.ready );
{% endhighlight %}

继续判断是否为iframe，如果不是的话采用不断的轮询scorll的方法

{% highlight javascript %}
// If IE and not a frame
// continually check to see if the document is ready
var top = false;
try {
  top = window.frameElement == null && document.documentElement;
} catch(e) {}
if ( top && top.doScroll ) {
 (function doScrollCheck() {
  if ( !jQuery.isReady ) {
    try {
     // Use the trick by Diego Perini
     // http://javascript.nwbox.com/IEContentLoaded/
     top.doScroll("left");
    } catch(e) {
     return setTimeout( doScrollCheck, 50 );
    }

    // and execute any waiting functions
    jQuery.ready();
    }
 })();
}
{% endhighlight %}

再贴上几段其他框架的代码，大同小异，就不具体分析了

prototype

{% highlight javascript %}
(function(GLOBAL) {
  /* Support for the DOMContentLoaded event is based on work by Dan Webb,
     Matthias Miller, Dean Edwards, John Resig, and Diego Perini. */
  
  var TIMER;
  
  function fireContentLoadedEvent() {
    if (document.loaded) return;
    if (TIMER) window.clearTimeout(TIMER);
    document.loaded = true;
    document.fire('dom:loaded');
  }
  
  function checkReadyState() {
    if (document.readyState === 'complete') {
      document.detachEvent('onreadystatechange', checkReadyState);
      fireContentLoadedEvent();
    }
  }
  
  function pollDoScroll() {
    try {
      document.documentElement.doScroll('left');
    } catch (e) {
      TIMER = pollDoScroll.defer();
      return;
    }
    
    fireContentLoadedEvent();
  }


  if (document.readyState === 'complete') {
    // We must have been loaded asynchronously, because the DOMContentLoaded
    // event has already fired. We can just fire `dom:loaded` and be done
    // with it.
    fireContentLoadedEvent();
    return;
  }
  
  if (document.addEventListener) {
    // All browsers that support DOM L2 Events support DOMContentLoaded,
    // including IE 9.
    document.addEventListener('DOMContentLoaded', fireContentLoadedEvent, false);
  } else {
    document.attachEvent('onreadystatechange', checkReadyState);
    if (window == top) TIMER = pollDoScroll.defer();
  }
  
  // Worst-case fallback.
  Event.observe(window, 'load', fireContentLoadedEvent);
})(this);
{% endhighlight %}
        
mootools

{% highlight javascript %}
(function(window, document){

var ready,
    loaded,
    checks = [],
    shouldPoll,
    timer,
    testElement = document.createElement('div');

var domready = function(){
    clearTimeout(timer);
    if (ready) return;
    Browser.loaded = ready = true;
    document.removeListener('DOMContentLoaded', domready).removeListener('readystatechange', check);

    document.fireEvent('domready');
    window.fireEvent('domready');
};

var check = function(){
    for (var i = checks.length; i--;) if (checks[i]()){
        domready();
        return true;
    }
    return false;
};

var poll = function(){
    clearTimeout(timer);
    if (!check()) timer = setTimeout(poll, 10);
};

document.addListener('DOMContentLoaded', domready);

/**/
// doScroll technique by Diego Perini http://javascript.nwbox.com/IEContentLoaded/
// testElement.doScroll() throws when the DOM is not ready, only in the top window
var doScrollWorks = function(){
    try {
        testElement.doScroll();
        return true;
    } catch (e){}
    return false;
};
// If doScroll works already, it can't be used to determine domready
//   e.g. in an iframe
if (testElement.doScroll && !doScrollWorks()){
    checks.push(doScrollWorks);
    shouldPoll = true;
}
/**/

if (document.readyState) checks.push(function(){
    var state = document.readyState;
    return (state == 'loaded' || state == 'complete');
});

if ('onreadystatechange' in document) document.addListener('readystatechange', check);
else shouldPoll = true;

if (shouldPoll) poll();

Element.Events.domready = {
    onAdd: function(fn){
        if (ready) fn.call(this);
    }
};

// Make sure that domready fires before load
Element.Events.load = {
    base: 'load',
    onAdd: function(fn){
        if (loaded && this == window) fn.call(this);
    },
    condition: function(){
        if (this == window){
            domready();
            delete Element.Events.load;
        }
        return true;
    }
};

// This is based on the custom load event
window.addEvent('load', function(){
    loaded = true;
});

})(window, document);
{% endhighlight %}           
        
参考文献

[参考文献集合](https://www.site2share.com/folder/20020508)