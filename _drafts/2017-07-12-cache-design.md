# 设计一个无懈可击的浏览器缓存机制：关于思路，细节，HTTP2，以及ServiceWorker

本文分为四个部分，文章的递进思路是由整体到局部，由业务概念到技术细节，由主线到分支，由现在到未来：

- 浏览器缓存设计的思路和策略
- 实现缓存过程中的技术细节
- 缓存与HTTP2结合时需要注意的地方
- 缓存与ServiceWorker结合时需要注意的地方

## 缓存设计的思路和策略

首先我们从最简单的一个用例开始。

假设你的站点有引用一个脚本文件，你非常确认这个脚本文件内容五十年不变。那么你自然希望浏览器把这个脚本缓存起来，不用每一次都请求服务器，然后服务器再返回相同的内容。这样能够节省带宽开销并且提升性能。

此时你只需要设置文件返回的HTTP头中的`Cache-Control`设置为：

```http
Cache-Control: max-age=31536000
```

虽然是五十年不变，但是标准中规定`max-age`值最大不能超过一年，又因为是以秒为单位，所以值为`31536000`



- [HTTP Caching](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/http-caching)
- [Caching best practices & max-age gotchas](https://jakearchibald.com/2016/caching-best-practices/)
- [HTTP conditional requests](https://developer.mozilla.org/en-US/docs/Web/HTTP/Conditional_requests)
- [Caching Tutorial](https://www.mnot.net/cache_docs/)
- [Expires vs. max-age](https://www.mnot.net/blog/2007/05/15/expires_max-age)
- [What takes precedence: the ETag or Last-Modified HTTP header?](https://stackoverflow.com/questions/824152/what-takes-precedence-the-etag-or-last-modified-http-header)
- [What's the difference between Cache-Control: max-age=0 and no-cache?](https://stackoverflow.com/questions/1046966/whats-the-difference-between-cache-control-max-age-0-and-no-cache)
- [serviceworke.rs](serviceworke.rs)
- [HTTP/2 push is tougher than I thought](https://jakearchibald.com/2017/h2-push-tougher-than-i-thought/)