---
layout: post
title: NodeJS 实战系列：如何设计 try catch
modified: 2023-01-29
tags: [NodeJS, Backend]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

本文将通过一个 NodeJS 程序里无效的错误捕获示例，来讲解错误捕获里常见的陷阱。错误捕获不是凭感觉添加 try catch 语句，它的首要目的是提供有效的错误排查信息，只有精心设计的错误捕获才有可能完成这个使命。针对哪些方面去精心设计就是本篇文章里想讨论的内容

实战系列来自于个人开发以及运维 [site2share](https://www.site2share.com) 网站过程中的经验

## 设计陷阱，而非听天由命

为什么代码里需要 try catch？是为了阻止 bug 的发生的？当然不是，bug 其实是代码的副产品，bug 的数量取决于代码的质量而非 try catch 的数量。

说到底 try catch 只是用来查漏补缺的工具，如果你把 try catch 只是当作万能的膏药在代码里想贴就贴，那你可能多半贴不中真正的要害，也得不到期望的结果

在 site2share 中我需要集成 Redis 用于存储用户的 session 信息，自然需要在代码中使用第三方类库使用 Redis，无论是 node-redis （还是 ioredis），它们都提供事件机制用于反馈与 Redis Server 连接的当前状态。比如我们可以监听 error 事件:

```javascript
redis.on('error', function () { });
```

为什么不监听看看呢。并且上线之后如偿所愿，在发现网站无法访问之后在日志中确实找到了

1.  如题图所示大量的报错信息
2.  每一个错误的调用栈

![application insights 中捕获的错误信息](../images/013_nodejs-try-catch-best-practice/app_insights_error.png)

但它们统统都仅是 node-redis 类库内部的函数调用栈，我发现这些信息对我来说毫无用处，因为它们无法向我提供最关键的一类信息：**上下文**。所以这些信息都只是在告诉我在访问 `/api/folder/:id` 时 redis 出现了报错，然而下列这些问题的答案才更有助于我排查问题：

*   是否只在指定 id 的情况下才会发生错误？
*   请求 API 时用户是否处于登录的状态？
*   连接状态是存在一定概率成功，还是稳定失败？
*   Azure Redis Server 的服务是否稳定？错误是否是由服务自身造成的

对于这些问题的答案我一无所知，更艰难的是我无法在本地开发环境中复现该错误。这个时候我才发现并非你收集的信息越多，你把问题解决的概率就越大，如果你始终缺失某条关键信息，再多的额外信息也于事无补。

**这又回到了我之前所说的**[**信息应该是双向的**](https://www.v2think.com/performance-metric-crisis)**，即我收集的信息务必让我有采取行动和回溯的能力**

所以捕获错误同样需要设计。或者退一步说，即使我不太确定错误会在哪里发生而需要在大范围内对错误进行捕获，也需要保证错误要提供有效信息：

*   除了看到错误消息，我还希望得到调用栈信息
*   如果我有了调用栈，那么我还希望得到具体的数据 id；
*   如果有了数据 id，那么我还希望能够得到 ORM 背后生成的 SQL 语句

再退一步说，如果无法得到直接有效信息，间接的有效信息也是可以接受的，例如你可以利用服务供应商或者基础设施的自带日志来协助排查问题；再不济如果只能硬着头皮阅读代码的话，被精心设计的函数名也非常重要。

那么如何设计好的 try catch呢？看起来你需要懂你的函数，你需要知道它可能的输入和期待的输出是什么，你需要知道它在执行过程中会和哪些服务打交道，你需要知道它的风险在哪。很有意思的是，我们从函数出发，想尽可能完美地捕获报错，但完美的答案又在函数本身当中。

![涉及错误捕获的诀窍在实现代码中](../images/013_nodejs-try-catch-best-practice/design-try-catch.png)

最后，如果程序在意料之外挂掉或者抛错，顺其自然好了。千万不要想法设法当作什么事情都没有发生然后继续执行下去。因为我们无法得知错误究竟带来的影响是什么，会带来怎样的连锁反应。抱有侥幸心理不如就此止损——请快速失败，快速恢复

说实话我很难找到关于 handle error 设计方面的书籍或者文章，很惊讶这块领域内的空白（我都能找到[好几本](https://www.manning.com/books/dependency-injection)聊[依赖注入](https://www.manning.com/books/dependency-injection-principles-practices-patterns)的[图书](https://www.manning.com/books/dependency-injection-in-dot-net)）。当你在读技术教程比如[《.NET Core in Action》](https://www.manning.com/books/dotnet-core-in-action)或者[《ASP.NET MVC 4 in Action》](https://www.manning.com/books/asp-net-mvc-4-in-action) 时，它们只会告诉你在框架中存在这样或者那样的 error handling 机制，至于如何使用才是最佳实践，并不在它们的范畴内。

## "接住"错误

为什么用“接住”而非“抓住”，是因为前者是被动后者是主动的，大部分情况下你都不太可能主动的、预测性的识别到某个bug。但我们不能因为如此就任由它们发生，我们需要：

*   抹去错误中的敏感信息
*   让错误信息变得更加友好
*   记录错误

在处理这些事物方面，我们需要集中化处理错误，目前绝大部分框架都支持这类操作。比如对于 .NET CORE 来说，我们可以通过在最外层添加 middleware 来解决这个问题

![.NET 中捕获错误的中间件](../images/013_nodejs-try-catch-best-practice/error-handling-middleware.png)

error handing middleare 只能作为程序处理错误的最后一道防线，对于不可知的错误尤其有效。然而对于一些可以前置，可以提前捕获的错误来说，我们又应该如何处理呢？

我的经验是，需要在系统内建立一套机制或者说通道，让 exception 按照指定的方向高效的流动起来才是首要任务。举个例子

```javascript
try {
    await getUserInfo()
} catch(e) {
    throw new LoadUserInfoException()
}
```

第一个问题是，我们是否真的需要 try catch？不一定，理想情况下即使错误在当前代码块没有被捕获，它发生的意外错误也应该掉落进最后一道防线中，然后翻译为能够暴露给外部的信息，随后程序立即中断，快速重启。

退一步说，即使你按照以上代码有意进行 catch，你要如何处理这个新 throw 出来的错误呢？最好的办法是我们无需关心。LoadUserInfoException 中可以事先定义这个错误的状态吗的 message，上面所说的程序中提前建立好的机制，会自动将这个错误按照状态码和message进行翻译，返回给客户端。并非所有场景都需要有意屏蔽错误信息，有时候将恰当的错误信息返回给客户端能够让用户自主的排查问题更好。

上面涉及的自动捕获、对错误进行翻译、直达客户端，以及你能够想到的跨功能需求，比如收集错误日志，都应该是程序中的基础设施，具体的开发人员无需关心，无需对于每一个错误都手动执行这一系列步骤。

正如下图所示，无论你的 controller、service、SDK 之间的调用层次如何，各个模块中被抛出的异常都一视同仁的被处理。然而开发人员只需要关心下图左上方的部分，至于错误信息如何向右流向客户端，则无需关心

![程序中的错误处理流](../images/013_nodejs-try-catch-best-practice/error-catch-diagram.png)

.NET Core 里的 middleware 和 NodeJS 里的 [error handler](https://expressjs.com/en/guide/error-handling.html) 都能可以达到这个效果

---

你可能也会喜欢：

- [NodeJS 实战系列：DevOps 尚未解决的问题](https://www.v2think.com/devops-solution-in-nodejs)
- [NodeJS 实战系列：如何设计 try catch](https://www.v2think.com/nodejs-try-catch-best-practice)
- [做一个能对标阿里云的前端APM工具（下）](https://www.v2think.com/apm-tool-2)
- [做一个能对标阿里云的前端APM工具（上）](https://www.v2think.com/apm-tool-1)
- [小心 Serverless](https://www.v2think.com/careful-with-serverless)
- [SQL Server 查询语句优化入门](https://www.v2think.com/sql-server-optimize-tutorial)
- [利用Node.js+express框实现图片上传](https://www.v2think.com/nodejs-express-upload-image)
- [一篇来自前端同学对后端接口的吐槽](https://www.v2think.com/toast-about-backend-API)
- [关于Node.js后端架构的一点后知后觉](https://www.v2think.com/something-about-nodejs-architecture)
- [在Node.js中搭建缓存管理模块](https://www.v2think.com/built-cache-management-module-in-nodejs)'
