# 使用fetch接口post数据需要的注意事项

你有没有突然发现，有一段代码使用了很久，或许是从stackoverflow复制过来，但是却从来没有想过为什么要这么写。

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

很明显我们在使用`fetch`接口时，没有指定`Content-Type`参数。值得一提的是，和jQuery提供的`$.ajax`接口不同，`fetch`接口相对底层，所以一部分工作需要手动自己完成。所以只要我们继续在请求中创建`headers`参数，并指定为`application/json`后端就能正常解析了：

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

### 其他前端类库是如何处理`Content-Type`的？

- https://github.com/expressjs/body-parser/issues/247
- https://stackoverflow.com/questions/5710358/how-to-retrieve-post-query-parameters
- https://github.com/expressjs/body-parser
- https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/
- https://stackoverflow.com/questions/38306569/what-does-body-parser-do-with-express-in-nodejs
- https://www.safaribooksonline.com/blog/2014/03/10/express-js-middleware-demystified/
