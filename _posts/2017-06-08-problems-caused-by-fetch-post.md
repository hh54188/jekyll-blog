---
layout: post
title: 使用fetch接口post数据时记得指定 Content-Type
modified: 2017-06-08
tags: [javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

本文只是想提醒你，使用`fetch`接口的时候，记得指定`Content-Type`，不然容易出错。

## 使用原生fetch接口post数据

前几天尝试使用原生的`fetch`接口向后台传输数据，当然是使用`post`方法。后端服务是由基于Node.js的Express框架提供。作为DEMO，前端代码非常简单：

```javascript
var request = new Request('/upload', {
    method: 'POST',
    body: JSON.stringify({
        'city': 'Beijing'
    })
});

fetch(request);
```

后端Express因为需要解析Post数据，所以需要安装`bodyParser`库进行解析，DEMO代码如下：

```javascript
const express = require('express');
const app = express();
const bodyParser = require('body-parser')

app.use( bodyParser.json() );       // to support JSON-encoded bodies
app.use( bodyParser.urlencoded({ extended: true }) ); // to support URL-encoded bodies

app.post('/upload', function (req, res) {
    console.log(req);
    console.log(req.body);
});

app.listen(80);
```
这段代码是我从google上找来的。

这样的前端代码配合这样的后端代码的结果是，前端能够发送请求（通过devtools观察到），后端能够收到（`console.log(req)`能够打印），但是传输的请求数据没法收到（`console.log(req.body)`打印为空）

**所以，问题出在哪里？**

## 解决问题

从上一段的描述可以推理出，服务端收到了请求，但是没有成功解析数据。因为是`bodyParser`解析POST数据的缘故，所以我们从`bodyParser`下手，找出为什么它没能解析我们的上传数据。

负责解析数据的并不是`bodyParser`自己，而是`bodyParser`提供的中间件（`bodyParser`文档里的原话是`The bodyParser object exposes various factories to create middlewares.`）。

顺便解释一下中间件（middleware）。一般情况下，当Express收到请求后，它会把请求转交给路由，交由具体的路径处理函数处理，就像上述代码里的`app.post('/upload', function (req, res) {}`一样。而中间件是一系列函数，在请求交给最终的路由函数处理之前，预先被Express调用，并可以提前拿到请求，对它进行一些加工。就像商品被转交消费者之前提前的工序。req对象本身是不带有`body`参数的，这个`body`参数是由`bodyParser`提供的中间件为我们加上的。为的是方便的提供我们拿到用户上传的数据。

你当然不一定需要中间件，甚至也不需要要Express框架，如果你收使用原生的Node.js代码解析请求数据的话，会比较麻烦：

```javascript
var http = require('http');

http.createServer(function(request, response) {
  var headers = request.headers;
  var method = request.method;
  var url = request.url;
  var body = [];
  request.on('error', function(err) {
    console.error(err);
  }).on('data', function(chunk) {
    body.push(chunk);
  }).on('end', function() {
    body = Buffer.concat(body).toString();
    // At this point, we have the headers, method, url and body, and can now
    // do whatever we need to in order to respond to this request.
  });
}).listen(8080); // Activates this server, listening on port 8080.
```

注意上述代码的`request.on('data')`和`on('end')`事件，我们需要手动的把stream中的请求片段拼装成为一个完整的请求。而框架和中间件帮我们完成了这一系列工作。

回到`bodyParser`，为什么它没能成功的解析我们的数据，我们首先需要了解一下它的工作原理，根据文档的说法是：

>All middlewares will populate the req.body property with the parsed body when the Content-Type request header matches the type option, or an empty object ({}) if there was no body to parse, the Content-Type was not matched, or an error occurred.

也就是说，当我们使用`bodyParser`提供的中间件时，可以传入`option`对象参数，这个对象参数里有一个`type`字段用于描述`Content-Type`。只有当请求的`Content-Type`与`type`字段匹配时，才会对请求中的body进行解析。

当然每一类中间件都有默认解析的`Content-Type`，比如`bodyParser.json()`中间件默认解析`application/json`类型请求；而`bodyParser.urlencoded`默认解析`application/x-www-form-urlencoded`类型请求。

很明显我们在使用`fetch`接口时，没有指定`Content-Type`参数。但因为body是字符串的缘故，浏览器默认给请求`Content-Type`的赋值是`text/plain`。而在服务端，我们只使用了两类中间件解析了两种类型的请求：`application/json`和`application/x-www-form-urlencoded`，所以`Content-Type`无法匹配上，自然上传的数据也就无法解析。

值得一提的是，和jQuery提供的`$.ajax`接口不同，`fetch`接口相对底层，所以一部分工作需要手动自己完成。

所以只要我们继续在请求中创建`headers`参数，并指定为`application/json`后端就能正常解析了：

```javascript
var request = new Request('/upload', {
    method: 'POST',
    headers: new Headers({
        'Content-Type': 'application/json'
    }),
    body: JSON.stringify({
        'city': 'Beijing'
    })
});

fetch(request);
```
或者修改服务端代码，添加对字符串解析的中间件：

```javascript
app.use(bodyParser.text());
```

## 故事还没有结束

### 自定义`Content-Type`

`bodyParser`还同时支持自定义的`Content-Type`，例如我在前端发出请求时使用的是自定义的`Content-Type`值为`secret`:

```javascript
var request = new Request('/upload', {
    method: 'POST',
    headers: new Headers({
        'Content-Type': 'secret'
    }),
    body: JSON.stringify({
        'city': 'Beijing'
    })
});
```

而在后端的中间件函数中，`type`字段可以定义为函数。例如我们想以JSON的方式解析上述类型的请求，则继续使用`bodyParser.json`中间件，并重新定义`type`字段：

```javascript
app.use( bodyParser.json({ 
  type: (req) => {
    return req.get('Content-Type') === 'secret'
  } 
}) );
```

### 其他前端类库是如何处理不添加`Content-Type`情况的？

**polyfill**

既然想使用原生的`fetch`接口，那么首先考察的想必是`fetch`接口的polyfill：[https://github.com/github/fetch](https://github.com/github/fetch)

目前除了IE仍然不支持这个接口以外，其余的最新版本现代浏览器都支持这个接口。所以我们只能在IE上测试这个polyfill。测试结果是，就本文的前端代码来说，工作预期和原生的接口是一样的。

但是有一个例外。例如我们将body改为一个对象，而不是字符串：

```javascript
var request = new Request('/upload', {
    method: 'POST',
    body: {
        'city': 'Beijing'
    }
});
```
那polyfill就报错了，而原生的fetch接口仍然能够发送数据，只不过发送请求中不存在`Content-Type`字段，后端收到的同样也是`undefined`。

而为什么会报错，我们看它的源代码就能得出端倪：
```javascript
this._initBody = function(body) {
    this._bodyInit = body
    if (!body) {
        this._bodyText = ''
    } else if (typeof body === 'string') {
        this._bodyText = body
    } else if (support.blob && Blob.prototype.isPrototypeOf(body)) {
        this._bodyBlob = body
    } else if (support.formData && FormData.prototype.isPrototypeOf(body)) {
        this._bodyFormData = body
    } else if (support.searchParams && URLSearchParams.prototype.isPrototypeOf(body)) {
        this._bodyText = body.toString()
    } else if (support.arrayBuffer && support.blob && isDataView(body)) {
        this._bodyArrayBuffer = bufferClone(body.buffer)
    // IE 10-11 can't handle a DataView body.
    this._bodyInit = new Blob([this._bodyArrayBuffer])
    } else if (support.arrayBuffer && (ArrayBuffer.prototype.isPrototypeOf(body) || isArrayBufferView(body))) {
        this._bodyArrayBuffer = bufferClone(body)
    } else {
        throw new Error('unsupported BodyInit type')
    }
```
很显然它在生成`Content-Type`阶段，使用目前支持的数据类型去匹配即将上传的数据，而我们上传的objct不属于任何一种，所以就报错了。

**jQuery**

我们尝试使用`$.ajax`接口上传刚刚的数据：
```javascript
$.ajax({
    url: '/upload',
    method: 'POST',
    timeout: 1000 * 1,
    data: JSON.stringify({
        'city': 'Beijing'
    })
})
```

很可惜，它的差异和fetch接口的差异较大。首先请求的`Content-Type`变为了`application/x-www-form-urlencoded;`。根据文档这是它使用的默认值。并且前端传输的数据类型也变为了`Form Data`, 而后端收到的数据也变为`{ '{"city": "Beijing"}': '' }`

本文的测试代码，包括前端和服务端，都放在这个项目中：[https://github.com/hh54188/fetch_post_test](https://github.com/hh54188/fetch_post_test)。使用前记得`npm install`和`bower install`

参考文章集合

[https://www.site2share.com/folder/20020518](https://www.site2share.com/folder/20020518)

