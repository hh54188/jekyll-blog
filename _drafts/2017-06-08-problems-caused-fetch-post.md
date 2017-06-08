# 使用fetch接口post数据需要的注意事项

你有没有突然发现，有一段代码使用了很久，或许是从stackoverflow复制过来，但是却从来没有想过为什么要这么写。

## 使用原生fetch接口post数据

前几天尝试使用原生的`fetch`接口向后台传输数据，当然是使用`post`方法。后端服务是由基于Node.js的Express框架提供。作为DEMO，前端代码非常简单：

```javascript
var request = new Request('/upload', {
    method: 'POST',
    body: {
        'city': 'Beijing'
    }
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

- https://github.com/expressjs/body-parser/issues/247
- https://stackoverflow.com/questions/5710358/how-to-retrieve-post-query-parameters
- https://github.com/expressjs/body-parser
- https://nodejs.org/en/docs/guides/anatomy-of-an-http-transaction/
- https://stackoverflow.com/questions/38306569/what-does-body-parser-do-with-express-in-nodejs
- https://www.safaribooksonline.com/blog/2014/03/10/express-js-middleware-demystified/
