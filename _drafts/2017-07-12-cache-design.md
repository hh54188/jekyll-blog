# 设计一个无懈可击的浏览器缓存机制：关于思路，细节，HTTP2，以及ServiceWorker

本文分为四个部分，文章的递进思路是由整体到局部，由业务概念到技术细节，由主线到分支，由现在到未来：

- 浏览器缓存设计的思路和策略
- 实现缓存过程中的技术细节
- 缓存与HTTP2结合时需要注意的地方
- 缓存与ServiceWorker结合时需要注意的地方

## 缓存设计的思路和策略

### 无需验证

首先我们从最简单的一个用例开始。

假设你的站点有引用一个脚本文件，你非常确认这个脚本文件内容五十年不变。那么你自然希望浏览器把这个脚本缓存起来，不用每一次都请求服务器，然后服务器再返回相同的内容。这样能够节省带宽开销并且提升性能。

此时你只需要设置文件返回的HTTP头中的`Cache-Control`设置为：

```http
Cache-Control: max-age=31536000
```

虽然是五十年不变，但是标准中规定`max-age`值最大不能超过一年，又因为是以秒为单位，所以值为`31536000`

例如这个五十年不变的脚本地址是`http://example.com/never-expire.js`。那么接下来每次用户请求这个地址时，浏览器都不会再向服务器发出请求，而是直接从本地的浏览器缓存中取。直到一年以后或者用户手动的清除了缓存。

但是，如果这一年中的某一天你发现脚本内容必须要更改了怎么办？很简单，改变请求的文件名就好了，例如`never-expire-v2.js`。

### 需要验证

像上个用例中一次请求之后完全从缓存中读取的情况并不多。因为脚本内容时常都会发生更改，然而我们也不确定迭代会在什么时候发生。所以保守考虑我们会将`max-age`赋值一个较小的时间，例如2分钟120秒`max-age=120`.

然而如果两分钟之后脚本没有更新的话又继续从服务器拉取相同的内容，这样似乎不合理。所以缓存提供另一个验证机制，用于验证当前资源是否已经发生了更改，虽然发出了请求，但是服务器不一定会返回完整资源内容。如果验证之后发现资源确实发生了更改，则返回完整的资源内容（http状态码200）；否则则返回`304 Not Modified`（http状态码304），即告知浏览器文件内容并未发生修改。

然而如何执行“验证”这个过程呢？在服务器第一次返回给浏览器的响应http头中，有一个名为`Etag`字段，这个字段代码这个请求的token，你可以理解为这次请求返回的id。token的生成并没有一个统一的规则，是由你的服务器返回逻辑决定，可以是md5值也可以是版本号，总之是用于辨别这个请求返回的唯一性。

当资源过期之后，浏览器再次请求该资源时，会添加一个名为`If-None-Match`的http头信息，值即是这个过期资源的`Etag`，用于告诉服务器当前浏览器拥有的资源版本。服务器也就是通过对比`Etag`版本信息，来判断资源是否进行了更新。

而头信息`If-None-Match`被称为`Conditional headers`，包含这样信息的请求则称为`Conditional Request`。

以上就是缓存设计的基本思路，围绕着资源何时过期，以及过期后如何更新展开。接下来我们看一看技术细节，如何通过配置缓存相关的头信息能够更好的为这些策略服务。

## 技术细节

**"no-cache", "no-store", "must-revalidate"**

`Cache-Control`字段可以设置的不仅仅是`max-age`存储时间，还有其他额外的值可以填写，甚至可以组合。主要使用的值有如下：
- `no-cache`: 虽然字面意义是“不要缓存”。但它实际上的机制是，仍然对资源使用缓存，但每一次在使用缓存之前必须（MUST）向服务器对缓存资源进行验证。
- `no-store`: 不使用任何缓存
- `must-revalidate`: 如果你配置了`max-age`信息，当缓存资源仍然新鲜（小于`max-age`）时使用缓存，否则需要对资源进行验证。所以`must-revalidate`可以和`max-age`组合使用`Cache-Control: must-revalidate, max-age=60`

有趣的事情是，虽然`no-cache`意为对缓存进行验证，但是因为大家广泛的错误的把它当作`no-store`来使用，所以有的浏览器也就附和了这种设计。这是一个典型的劣币驱逐良币的过程。

**Expires VS. max-age**

`Expires`和`max-age`都是用于控制缓存的生命周期。不同的是`Expires`指定的是过期的具体时间，例如`Sun, 21 Mar 2027 08:52:14 GMT`，而`max-age`指定的是生命时长秒数`315360000`。

不同的区别在于`Expires`是 HTTP/1.0 的中的标准，而`max-age`是属于`Cache-Control`的内容，是 HTTP/1.1 中的定义的。但为了想向前兼容，这两个属性仍然要同时存在。

但有一种更倾向于使用`max-age`的观点认为`Expires`过于复杂了。例如上面的例子`Sun, 21 Mar 2027 08:52:14 GMT`，如果你在表示小时的数字缺少了一个0，则很有可能出现出错；如果日期没有转换到用户的正确时区，则有可能出错。这里出错的意思可能包括但不限于缓存失效、缓存生命周期出错等。

**Etag VS. Last-Modified**

`Etag`和`Last-Modified`都可以用于对资源进行验证，而`Last-Modified`顾名思义，表示资源最后的更新时间。

我们把这两者都成为验证器（Validators），不同的是，`Etag`属于强验证（Strong Validation），因为它期望的是资源字节级别的一致；而`Last-Modified`属于弱验证（Weak Validation），只要资源的主要内容一致即可，允许例如页底的广告，页脚不同。

根据[RFC 2616](http://www.ietf.org/rfc/rfc2616.txt)标准中的13.3.4小节，一个使用HTTP 1.1标准的服务端应该（SHOULD）同时发送`Etag`和`Last-Modified`字段。同时一个支持HTTP 1.1的客户端，比如浏览器，如果服务端有提供`Etag`的话，必须（MUST）首先对`Etag`进行Conditional Request（If-None-Match头信息）；如果两者都有提供，那么应该（SHOULD）同时对两者进行Conditional Request（If-Modified-Since头信息）。如果服务端对两者的验证结果不一致，例如通过一个条件判断资源发生了更改，而另一个判定资源没有发生更改，则不允许返回`304`状态。但话说回来，是否返回还是通过服务端编写的实际代码决定的。所以仍然有操纵的空间。

**max-age=0 VS. no-cache**

`max-age=0`是在告诉浏览器，资源已经过期了，你应该（SHOULD）对资源进行重新验证了；而`no-cache`则是告诉浏览器在每一次使用缓存之前，你必须（MUST）对资源进行重新验证。

区别在于，*SHOULD*是非强制性的，而*MUST*是强制性的。在`no-cache`的情况下，浏览器在向服务器验证成功之前绝不会使用过期的缓存资源，而`max-age=0`则不一定了。虽然理论上来说它们的效果应该是一致的。

**public VS. private**

要知道从服务器到浏览器之间并非只有浏览器能够缓存，服务器返回的请求可能会经过














- [HTTP Caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- [Caching best practices & max-age gotchas](https://jakearchibald.com/2016/caching-best-practices/)
- [HTTP conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests)
- [Caching Tutorial](https://www.mnot.net/cache_docs/)
- [Expires vs. max-age](https://www.mnot.net/blog/2007/05/15/expires_max-age)
- [What takes precedence: the ETag or Last-Modified HTTP header?](https://stackoverflow.com/questions/824152/what-takes-precedence-the-etag-or-last-modified-http-header)
- [What's the difference between Cache-Control: max-age=0 and no-cache?](https://stackoverflow.com/questions/1046966/whats-the-difference-between-cache-control-max-age-0-and-no-cache)
- [serviceworke.rs](serviceworke.rs)
- [HTTP/2 push is tougher than I thought](https://jakearchibald.com/2017/h2-push-tougher-than-i-thought/)