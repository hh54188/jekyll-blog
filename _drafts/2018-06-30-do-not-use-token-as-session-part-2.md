# 不要把 JWT 用作 session 管理（下）：美好理想与现实困境

建议在阅读完上篇之后在阅读本篇，本篇中使用到了很多上一篇的概念

在上篇中，我们全面了解了关于身份认证和用户授权的所有概念，讲解了 JWT (Json Web Token) 的工作原理和使用场景。并且在文章的最后引出了一个问题，JWT 可否替代传统的 session 机制？

我们再来回顾一下这个问题。

在传统的 session 机制下，用户的登陆状态是通过 session id 维系的：session id 同时存储在后端 Redis 和用户浏览器的 cookie 中，用户的每次请求都会以 cookie 的形式带上 session id，后端在收到请求之后就能够通过检测 session id 是否在 Redis 中来判断用户是否已经登陆:

![session flow](./images/token-not-session/session-flow.png)

理论上来说，JWT 机制可以取代 session 机制：用户不需要提前进行登陆，后端也不需要 Redis 记录用户的登陆信息。当用户需要调用接口时，必须上传一个合法的 JWT，每一次调用接口，后端都使用请求中附带的 JWT 做一次合法性的验证。这样也间接达到了认证用户的目的：

![jwt-flow](./images/token-not-session/jwt-flow.png)

注意上图中的 JWT 流程和上一篇中描述的有存在差异的地方：在上一篇中我们讲到客户端会保存一份 secret 用于生成 JWT，而上图中 JWT 完全是由客户端给予的。但这都不影响服务端对 JWT 的验证。而至于这两者适用于什么样的场景后面再说

我们先来看看这个主意美好的一面，再看看它实施困难的一面

## 美好理想




## 参考资料

### TOKEN SESSION VS COOKIE SESSION

* https://ponyfoo.com/articles/json-web-tokens-vs-session-cookies
* https://auth0.com/blog/ten-things-you-should-know-about-tokens-and-cookies/

### TOKEN EXPIRED AND UPDATE

* https://stackoverflow.com/questions/39825953/handling-jwt-expiration-and-jwt-payload-update
* https://github.com/brahalla/Cerberus/issues/5
* https://softwareengineering.stackexchange.com/questions/338337/handling-token-renewal-session-expiration-in-a-restful-api
* https://www.zhihu.com/question/41248303
* https://news.ycombinator.com/item?id=11929267