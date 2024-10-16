---
layout: post
title: Promise/A的误区以及实践
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-12-20
tags: [javascript, front-end]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 什么是Promise
Promise是一种让异步代码书写起来更优雅的模式，能够让异步操作代码像同步代码那样书写并且阅读，比如下面这个异步请求的例子：

{% highlight javascript %}
$.get("/js/script,js", function () {
    // callback to do
})
{% endhighlight %}

就可以改写为Promise模式：

{% highlight javascript %}
var promise = $.get("/js/script");
{% endhighlight %}

返回值promise即代表操作的最终结果。返回值promise也可以作为“第一类对象”（[First-class object](http://en.wikipedia.org/wiki/First-class_citizen)）被当做参数传递。这个模式最大的优势就是避免了传统异步操作代码中，回调函数嵌套回调函数的糟糕情况。

如果你之前对Promise模式有所了解的话（可以参考InfoQ之前的这篇[文章](http://www.infoq.com/cn/news/2011/09/js-promise)），谈到Promise，最先想到的一定是它的`then`函数，的确它非常重要，在Promise模式的定义中([CommonJS Promises/A](http://wiki.commonjs.org/wiki/Promises/A))中，`then`函数是这么被定义的：

>（原文）A promise is defined as an object that has a function as the value for the property then: `then(fulfilledHandler, errorHandler, progressHandler)`


>（译）一个promise被定义为一个拥有then属性的对象，并且此then属性的值为一个函数: `then(fulfilledHandler, errorHandler, progressHandler)`

也就是说每一个promise结果一定会自带一个then函数，通过这个`then`函数，我们可以添加promise转变到不同状态(定义中promise只有三种状态，unfulfilled, fulfilled, failed.这里说的状态转变即从unfulfilled至fulfilled，或者从unfulfilled至failed)时的回调，还可以监听progress事件，拿上面的代码为例：

{% highlight javascript %}
var fulfilledHandler = function () {}
var errorHandler = function () {}
var progressHandler = function () {}

$.get("/js/script").then(fulfilledHandler, errorHandler, progressHandler)
{% endhighlight %}

这有一些类似于

{% highlight javascript %}
$.ajax({
    error: errorHandler,
    success: fulfilledHandler,
    progress: progressHandler
})
{% endhighlight %}
这个时候你会感到疑惑了，上面两种方式看上去不是几乎一模一样吗？——但promise的重点并非在上述各种回调函数的聚合，**而是在于提供了一种同步函数与异步函数联系和通信的方式**。之所以感到相似这也是大部分人对Promise的理解存在的误区，只停留在`then`的聚合(aggregating)功能。甚至在一些著名的类库中也犯了同样的错误(下面即以jQuery举例)。下面通过列举两个常见的误区，来让人们对Promise有一个完整的认识。

## Promise/A模式与同步模式有什么联系？

抛开Promise，让我们看看同步操作函数最重要的两个特征

- 能够返回值
- 能够抛出异常

这其实和高等数学中的[复合函数](http://zh.wikipedia.org/wiki/%E5%A4%8D%E5%90%88%E5%87%BD%E6%95%B0)(function composition)很像：你可以将一个函数的返回值作为参数传递给另一个函数，并且将另一个函数的返回值作为参数再传递给下一个函数……像一条“链”一样无限的这么做下去。更重要的是，如果当中的某一环节出现了异常，这个异常能够被抛出，传递出去直到被`catch`捕获。

而在传统的异步操作中不再会有返回值，也不再会抛出异常——或者你可以抛出，但是没有人可以及时捕获。这样的结果导致必须在异步操作的回调中再嵌套一系列的回调，以防止意外情况的发生。

而Promise模式恰好就是为这两个缺憾准备的，它能够实现函数的复合与异常的抛出（冒泡直到被捕获）。符合Promise模式的函数必须返回一个promise，无论它是fulfilled状态也好，还是failed(rejected)状态也好，我们都可以把它当做同步操作函数中的一个返回值：

{% highlight javascript %}
$.get("/user/784533") // promise return
.then(function parseHandler(info) {
    var userInfo = parseData(JSON.parse(info));
    return resolve(userInfo); // promise return
})
.then(getCreditInfo) // promise return
.then(function successHandler(result) {
    console.log("User credit info: ", result);
}, function errorHandler(error) {
    console.error("Error:", error);
})

{% endhighlight %}
上面的例子中，`$.get`与`getCreditInfo`都为异步操作，但在Promise模式下，（形式上）转化为了链式的顺序操作

`$.get`返回的promise由`parseHandler`进行解析，返回值“传入”`getCreditInfo`中，而`getCreditInfo`的返回值同时“传入”`successHandler`中。

之所以要在传入二字上注上引号，因为并非真正把promise当做值传递进入函数中，但我们完全可以把它理解为传入，并且改写为同步函数的形式，这样以来函数复合便一目了然：
{% highlight javascript %}
try {
    var info = $.get("/user/784533"); //Blocking
    var userInfo = parseData(JSON.parse(info));
    
    var resolveResult = parseData(userInfo);
    var creditInfo = getCreditInfo(resolveResult); //Blocking

    console.log("User credit info: ", result);
} cacth(e) {
    console.error("Error:", error);
}
{% endhighlight %}

但是在jQuery1.8.0版本之前，比如jQuery1.5.0（jQuery在1.5.0版本中引入Promise，在1.8.0开始得到修正），存在无法捕获异常的问题：

{% highlight javascript %}
var step1 = function() {
    console.log("------step1------");
    var d = $.Deferred();
    d.resolve('Some data');
    return d.promise();
},

step2 = function(str) {
    console.log("------step2------");
    console.log("step2 recevied: ", str);

    var d = $.Deferred();
    // 故意在fulfilled hanlder中抛出异常
    d.reject(new Error("This is failing!!!"));
    return d.promise();
},

step3 = function(str) {
    console.log("------step3------");
    console.log("step3 recevied: ", str);

    var d = $.Deferred();
    d.resolve(str + ' to display');
    return d.promise();
},

completeIt = function(str) {
    console.log("------complete------");
    console.log("[complete]------>", str);
},

handleErr = function(err) {
    console.log("------error------");
    console.log("[error]------>", err);
};   

step1().
then(step2).
then(step3).
then(completeIt, handleErr);    
{% endhighlight %}
上述代码在[jQuery-1.5.0](http://ajax.googleapis.com/ajax/libs/jquery/1.5.0/jquery.min.js)中运行的结果：

{% highlight javascript %}
------step1------
------step2------
step2 recevied:  Some data
------step3------
step3 recevied:  Some data
------complete------
[complete]------> Some data
{% endhighlight %}

在step2中，在解析step1中传递的值后故意抛出了一个异常，但是我们在最后定义的`errorHandler`却没有捕获到这个错误。

忽略捕获异常的错误，上面的结果还反映出另一个问题，最后一步`completeHandler`中处理的值应该是由step3中决定的，也就是step3中的

{% highlight javascript %}
d.resolve(str + ' to display');
{% endhighlight %}

最后应打印出的结果为

{% highlight javascript %}
some data to display
{% endhighlight %}

而在[jQuery-1.9.0](http://ajax.googleapis.com/ajax/libs/jquery/1.5.0/jquery.min.js)中异常是可以捕获的，运行结果为：

{% highlight javascript %}
------step1------
------step2------
step2 recevied:  Some data
------error------
[error]------> Error {}  
{% endhighlight %}
但是打印出的结果仍然有问题

注意到step3没有执行，因为step3中只定义了fulfilled的回调，异常只有在最后`errorhandler`才被捕获。

其实我们可以试试，在step3中添加处理异常的回调函数：

{% highlight javascript %}
step1().
then(step2).
then(step3, function (str) {
    console.log("------[error] step3------");
    console.log("step3 revecied: ", str);

    var d = $.Deferred();
    d.resolve(str + ' to display');
    return d.promise();    
}).
then(completeIt, handleErr);
{% endhighlight %}
运行结果如下:

{% highlight javascript %}
------step1------
------step2------
step2 recevied:  Some data
------[error] step3------ 
step3 revecied:  Error {} 
------complete------ 
[complete]------> Error: This is failing!!! to display 
{% endhighlight %}

虽然错误在step3被捕获了，但是由于我们将错误信息传递了下去，最后一步打印出的仍然是error消息


## 细节：返回Promise

让我们继续看看Promise/A定义的第二段：

> （原文）This function should return a new promise that is fulfilled when the given fulfilledHandler or errorHandler callback is finished. This allows promise operations to be chained together. The value returned from the callback handler is the fulfillment value for the returned promise. If the callback throws an error, the returned promise will be moved to failed state.

>（译文）这样的函数应该返回一个新的promise，该promise是被指定回调函数(成功执行或者捕获异常)解析之后的结果。如此一来promise之间的操作便能链式的串联起来。回调函数返回的值是解析返回的promise的结果。如果回调函数抛出了异常，返回的promise便会转化为异常状态

这段定义告诉我们两点：

1. 无论返回值是fulfilled也好，还是被rejected也好，必须返回一个新的promise；
2. `then`关键字并非只是各个回调的填充器，在输入的同时它同时也输出新的promise，以便形成链式;

同样以jQuery的代码为例：

{% highlight javascript %}
var step1 = function() {
    console.log("------step1------");
    var d = $.Deferred();
    d.resolve('Some data');
    return d.promise();
};

var step2 = function (result) {
    console.log("------step2------");
    console.log("step2 recevied: " + result);
    var d = $.Deferred();
    d.resolve("step2 resolve: " + result);
    return d.promise();
}

var step3 = function (result) {
    console.log("------step3------");
    console.log("step3 recevied: " + result);
    var d = $.Deferred();
    d.reject(new Error("This is failing!!!"));
    return d.promise();
}

var promise = step1();

var promise1 = promise.then(step2);
var promise2 = promise.then(step3);
{% endhighlight %}
step1返回的promise是fulfilled状态，但不同的是step2 fulfilled之后，返回一个仍然可被解析的promise(1)，而step3则抛出一个异常(promise2)。

按照定义所说，promise1与promise2是相互不同的promise，无论是被正确解析还是抛出异常，返回的都应该是一个独立的promise。

为了验证产生的是否为独立的promise，只需看他们的执行结果如何，接着给promise1和promise2定义fulfilled和failed回调函数：

{% highlight javascript %}
promise1.then(function (result) {
    console.log("Success promise1: ", result);
}, function () {
    console.log("Failed promise1: ", result);
})

promise2.then(function (result) {
    console.log("Success promise2: ", result);
}, function (result) {
    console.log("Failed promise2: ", result);
}) 
{% endhighlight %}

让我们看看在jQuery-1.5.0中执行的结果：

{% highlight javascript %}
Success promise1:  Some data
Success promise2:  Some data 
{% endhighlight %}
promise2虽然是一个被抛出的异常，但仍然可以被正确解析，并且解析使用的参数是上一个promise的返回值

在jQuery-1.9.0中：

{% highlight javascript %}
Success promise1:  step2 resolve: Some
Failed promise2:  Error {}
{% endhighlight %}

能被正常解析。

## 实践

完整认识了promise之后，我们可以用简单的代码实现一个Promise模式。

参照jQuery的`Deferred`，我们可以了解Promise的大致结构：

{% highlight javascript %}
var Promise = function () {}

Promise.prototype.when = function () {
    // to do
}

Promise.prototype.resolve = function () {
    // to do
}

Promise.prototype.rejected = function () {
    // to do
}
{% endhighlight %}
并且我们用最简单一个异步操作`setTimeout`来验证我们的Promise是否奏效：

{% highlight javascript %}
var delay = function (throwError) {
    var promise = new Promise();

    if (throwError) {
        promise.reject(new Error("ERROR"));
        return promise;
    }

    setTimeout(function () {
        promise.resolve("some data");
    }, 1000);

    return promise;
}

delay().then(function (result) {
    console.log(result);
}).then(function () {
    console.log("This is the second successHandler");    
}).then(function () {
    console.log("This is the third successHandler");
})
{% endhighlight %}

首先我们要为每一个Promise准备一个**队列**来存储自己的回调函数

{% highlight javascript %}
function Promise() {
    this.callbacks = [];
}
{% endhighlight %}

我们可以暂且把`then()`理解为往队列中填入回调的函数，并且为了能以链式的形式添加处理函数，最后必须返回当前promise：

{% highlight javascript %}
Promise.prototype.then = function (successHandler, failedHandler) {
    this.callbacks.push({
        resolve: successHandler,
        reject: failedHandler
    });

    return this;
}
{% endhighlight %}
其实resolve和reject虽然名称不同，但是都是执行各自对应的回调函数，于是可以抽象出一个公共的`complete`方法：

{% highlight javascript %}
Promise.prototype = {
    resolve: function (result) {
        this.complete("resolve", result);
    },

    reject: function (result) {
        this.complete("reject", result);
    },

    complete: function (type, result) {
        // to do
    }
}
{% endhighlight %}

`complete`的工作非常显而易见，根据type不同执行回调函数出队，以`result`为参数，执行相应`type`的的函数：

{% highlight javascript %}
complete: function (type, result) {
    this.callbacks.shift()[type](result);
}
{% endhighlight %}

但是这样只能执行队首回调函数，在链式的情况下，可能在`callbacks`中添加了多个回调函数，为了实现链式的执行，需要把`callbacks`中的回调全部出队，`complete`可以改进为：

{% highlight javascript %}
complete: function (type, result) {
    while(this.callbacks.length) {
        this.callbacks.shift()[type](result);
    }
}
{% endhighlight %}
完整版如下：

{% highlight javascript %}
function Promise() {
    this.callbacks = [];
}

Promise.prototype = {
    resolve: function (result) {
        this.complete("resolve", result);
    },

    reject: function (result) {
        this.complete("reject", result);
    },

    complete: function (type, result) {
        while(this.callbacks[0]) {
            this.callbacks.shift()[type](result);
        }
    },

    then: function (successHandler, failedHandler) {
        this.callbacks.push({
            resolve: successHandler,
            reject: failedHandler
        });

        return this;
    }
}
{% endhighlight %}
第一个版本即完成，可以看到测试上面开始例子的结果，能够顺利打印出信息。

接下来我们来完成处理异常部分

首先我们写一个能够故意抛出异常的测试用例

{% highlight javascript %}
delay(true).then(function (result) {
    console.log(result);
}, function firstErrorHandler(error) {
    console.error("First failedHandler catch: ", error);

    var promise = new Promise();
    promise.resolve("some data");
    return promise;

}).then(function secondSucHandler(result) {
    console.log("Second successHandler recevied: ", result);
}, function (error) {
    console.error("Second failedHandler catch: ", error);
})
{% endhighlight %}

我们在delay中抛出异常，希望在`firstErrorHandler`捕获异常后，返回一个能fulfilled的promise，并且用`secondSucHandler`顺利解析、

如果直接用上面版本执行，会发现没有任何结果，为什么？

**因为在执行`delay()`时，第一个`reject`也同时被执行，但此时`then`函数还没执行，也就是处理reject的handler还没有被定义。**当然也就不会有任何结果了。反过来也能想通，也就能说通resolve能被执行。

那么我们只要在reject函数中加上一定的延时即可：

{% highlight javascript %}
...
reject: function (result) {
    var _this = this;
    setTimeout(function () {
        _this.complete("reject", result);    
    });
},
...
{% endhighlight %}

执行测试代码结果如下：

{% highlight javascript %}
First failedHandler catch:  Error
Second failedHandler catch:  Error

{% endhighlight %}

虽然错误被捕获了，但错误被一直传递一下去了，这也就是我们之前说的jQuery无法返回新的promise，接下来要解决这个问题。

我们来写一个更复杂的测试用例，来验证下面的解决方案：

{% highlight javascript %}
delay()
// ------Level 1------
.then(function FirstSucHandler(result) {
    console.log("First successHandler recevied: ", result);

    var p = new Promise();
    p.reject(new Error("This is a test"));
    return p;                    

}, function FirstErrorHandler(error) {
    console.error("Second failedHandler catch: ", error);
})
// ------Level 2------
.then(function SecondSucHandler(result) {
    console.log("Second successHandler recevied: ", result);            
})
// ------Level 3------
.then(function ThirdSucHandler(result) {
    console.log("Third successHandler recevied: ", result);
}, function ThirdErrorHandler(error) {
    console.error("Third failedHandler catch: ", error);
})
// ------Level 4------
.then(function FourSucHandler(result) {
    console.log("Fourth successHandler recevied: ", result);
})
{% endhighlight %}

正确的执行顺序应该是`FirstSucHandler`fulfilled之后抛出异常，略过`SecondSucHandler`，异常被`ThirdErrorHandler`捕获，并且返回一个新的promise，由`FourSucHandler`解析。

接下来还要修复并且考虑这些问题
1. 当异常被捕获之后将阻止异常往下传递
2. 定义中描述在fulfilled之后必须返回一个新的promise，但如果没有返回新的promise，或者是返回其他的值，应该作何处理？

对于第二点，我们暂且处理规则是：

1. 如果没有返回值，那么下一个回调函数将继续解析上一promise
2. 如果返回值存在，但返回值不为promise，默认调用resolve handler，并且返回值作为回调函数的参数传入

大致流程图如下所示

{% highlight javascript %}
                                push
                                 |
                    [successHandler, errorHandler]
                    ...
            ------->[successHandler, errorHandler]
            |                    | 
            |                  shift
            |                    |
            |                 execute 
            |        succussHandler or errorHandler
            |                    |
            |                    |
            |          ----------------------
            |          |                    |
            |         return               return
            ----Auto--promise         nothing or not promise
            |                               |
            <--------------- Manual --------|

{% endhighlight %}
从上图中可以看出，promise模式是一个周而复始执行resolve或者reject过程，每一轮必须执行二者之一，必然导致一组回调函数出队。如果抛出异常，但这组回调函数中没有异常处理函数`errorHandler`，那么这组回调函数便作废，直到找到下一个能捕获异常的回调函数。直到队列中回调函数全部出队。看来我们有必要写一个“直到找到我们需要的函数”的函数：

{% highlight javascript %}
...
getCallbackByType: function (type) {
    if (callbacks.length) {

        var callback = callbacks.shift()[type];

        while (!callback) {
            callback = callbacks.shift()[type];
        }                    

    }

    return callback;
}
...
{% endhighlight %}

从上图中可以看出所有的promise可以共用一个`callbacks`队列，并且考虑到需要判断返回值是否为promise类型，我们最好还需要一个标志位，做如下修改：

{% highlight javascript %}
var callbacks = [];

function Promise() {
    this.isPromise = true;
}
...
{% endhighlight %}

根据以上描述，这类似于一个递归的过程

{% highlight javascript %}
promise --> resolve/reject ---> promise ---> resolve/reject
{% endhighlight %}
注意在上面Level 1的`FirstSucHandler`中，新返回的promise执行了reject，这会自动使队列一组回调函数出队并执行。但面对一些没有返回值的情况应该怎么办，那么就应该遵循我上面说的标准，要么执行上一个promise，要么默认执行下一个resolve：

{% highlight javascript %}
...
executeInLoop: function (promise,result) {
    // 1.如果回调队列还没有被清空  
    // 2.或者没有返回值，
    // 3.或者有返回值但不是promise
    if ((promise && !promise.isPromise || !promise) && callbacks.length) {
        // 默认执行resolve
        var callback = this.getCallbackByType("resolve");

        if (callback) {
            var promise = callback(promise? promise: result);
            this.executeInLoop(promise, promise? promise: result);
        }
    }
},
...
{% endhighlight %}
最后`Complete`函数也要做相应修改：

{% highlight javascript %}
...
complete: function (type, result) {

    var callback = this.getCallbackByType(type);

    if (callback) {
        var promise = callback(result);    
        this.executeInLoop(promise, promise? promise: result);
    }
},
...
{% endhighlight %}

最后贴上完整版代码：

{% highlight javascript %}
var callbacks = [];

function Promise() {
    this.isPromise = true;
}

Promise.prototype = {
    resolve: function (result) {
        this.complete("resolve", result);

    },

    reject: function (result) {
        var _this = this;
        setTimeout(function () {
            _this.complete("reject", result);    
        });
    },

    executeInLoop: function (promise,result) {
        // 如果队列里还有函数 并且（ 要么 没有返回一个值 或者 （有返回值但不是promise类型））
        if ((promise && !promise.isPromise || !promise) && callbacks.length) {

            var callback = this.getCallbackByType("resolve");

            if (callback) {
                var promise = callback(promise? promise: result);
                this.executeInLoop(promise, promise? promise: result);
            }
        }
    },

    getCallbackByType: function (type) {
        if (callbacks.length) {

            var callback = callbacks.shift()[type];

            while (!callback) {
                callback = callbacks.shift()[type];
            }                    

        }

        return callback;
    },

    complete: function (type, result) {

        var callback = this.getCallbackByType(type);

        if (callback) {
            var promise = callback(result);    
            /*
                1. 有返回值，promise类型
                2. 有返回值，其他类型
                3. 无返回值
            */
            this.executeInLoop(promise, promise? promise: result);
        }
    },

    then: function (successHandler, failedHandler) {
        callbacks.push({
            resolve: successHandler,
            reject: failedHandler
        });

        return this;
    }
}
{% endhighlight %}
并附上执行结果：

{% highlight javascript %}
First successHandler recevied:  some data 
Third failedHandler catch:  Error {} 
Fourth successHandler recevied:  Error {} 
{% endhighlight %}

参考文献

[参考文献集合](https://www.site2share.com/folder/20020511)




