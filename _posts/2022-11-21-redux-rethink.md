---
layout: post
title: Redux 的困扰与如何技术选型
modified: 2022-11-21
tags: [javascript, flux, redux, architecture]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

文章的名字我想了很久，备选项有“我再不推荐 Redux”，“Redux 为什么令我头疼”，“Redux 进化启示录”等等。通过这一系列名字我想你大概能猜到我接下来想聊的问题是什么，但这个问题放眼望去不是 Redux 独有，而是在做技术决策时经常会遇到的，即使对于非前端背景的开发者也同样成立。最后决定用一个带有开放式标题也许能够引起更多的共鸣。

这篇文章三年前（2019年）就想动笔，幸运的是当三年后再次拾起这个话题时，发现当初的观点依然成立。三年的时间 Redux 的社区和Flux 的生态都有了巨大变化，这些变化可以作为我们讨论的补充

## 如何在 Redux 里发起异步请求

如果你对 Redux 还算熟悉的话，那么不妨回答这个问题：如何在 Redux 里发起异步请求？我们暂且不管 Redux Toolkit（以下简称 RTK）的场景，把时钟拨回到三年前 RTK 还未诞生（[RTK 1.0 发布于 2019 年10月23日](https://github.com/reduxjs/redux-toolkit/releases/tag/v1.0.0) ）前，用最核心的 Redux 技术思考这个问题

我这里给出一个备选答案看行不行？为了后面引用我们把这段代码称为方案1：

```javascript
const mapDispatchToProps = (dispatch) => {
  return {
    fetchUser: () => {
      dispatch({
        type: 'FETCH_START'
      });
      fetch('https://randomuser.me/api/')
        .then(res => res.json())
        .then(({ results }) => {
          dispatch({
            type: 'FETCH_END',
            results,
          });
        })
    },
  }
}
```

在 Redux 的官网教程中有专门一节来教授如何处理异步逻辑和数据抓取，无论是[当下](https://redux.js.org/tutorials/fundamentals/part-6-async-logic)还是[三年前](http://web.archive.org/web/20190102001907/https:/redux.js.org/advanced/async-actions)（感谢 web.archive）的教程，它推荐的都是借助 redux-thunk 这个中间件来达到目的，这里我们称之为方案2：

```javascript
function fetchUser() {
  return dispatch => {
    dispatch(requestUserStart())
    return fetch(`https://randomuser.me/api/`)
      .then(response => response.json())
      .then(json => dispatch(requestUserEnd(json)))
  }
}
```

然而在围观过 [Dan Abramov](https://twitter.com/dan_abramov)（Redux 和 React 的核心成员） 在 StackOverflow 上解释为什么你应该使用 redux-thunk 的回答（["How to dispatch a Redux action with a timeout?"](https://stackoverflow.com/questions/35411423/how-to-dispatch-a-redux-action-with-a-timeout/35415559#35415559)、["Why do we need middleware for async flow?"](https://stackoverflow.com/questions/34570758/why-do-we-need-middleware-for-async-flow-in-redux/34599594#34599594)）之后，你会发现方案1并非有什么大的过错，它可行，不过当用例复杂之后代码可维护性会下降。

随之而来的是，如果你使用了 redux-thunk，它提供的 getState 方法你是否应该使用？在这个 StackOverflow 上的回答（[Accessing Redux state in an action creator?](https://stackoverflow.com/a/35674575/508236)）里我们看到的是两位的官方维护者（Dan Abramov 和 Mark Erikson）的两种想左意见，Mark 还专门有一篇长文解释这个问题（[Idiomatic Redux: Thoughts on Thunks, Sagas, Abstraction, and Reusability](https://blog.isquaredsoftware.com/2017/01/idiomatic-redux-thoughts-on-thunks-sagas-abstraction-and-reusability/)）

## Redux 是模式，而非框架

我们从上面看到了 Redux 社区里的有趣现象，即是“原则”与“实践”的分离：redux-thunk 作为官方出品并且推荐使用（但非必须，因为 [redux-saga](https://github.com/redux-saga/redux-saga) 或者 [redux-promise](https://github.com/redux-utilities/redux-promise) 也能起到同样的效果）的中间件，并不在默认 Redux 安装包中。同样的，Redux 的官方文档中有大篇幅来[谈 immutable 的重要性](https://redux.js.org/usage/structuring-reducers/immutable-update-patterns)，或者是[如何对数据进行 normalizing](https://redux.js.org/usage/structuring-reducers/normalizing-state-shape),  但在工具上它却推荐使用第三方类库进行代码约束（主流的第三方插件在官方文档中的 [Ecosystem](https://redux.js.org/introduction/ecosystem) 一页中都可以找到）

与 getState 的情况类似，在另一个“如何处理数据加载过程中报错”的问题中（[What is the best way to deal with a fetch error in react redux?](https://stackoverflow.com/questions/34403269/what-is-the-best-way-to-deal-with-a-fetch-error-in-react-redux/34482258#34482258)），Dan 给出了他心目中的最佳实践。Dan 的权威性让人很难不把他的回答当作为来自官方的建议。但是让人疑惑的地方在于，我们看到的有关 Redux 的最佳实践来自于 StackOverflow 上，而非官方文档中。

为什么出现这样的结果？我的理解是 Redux 从始至终并未被当作一款常规框架来设计，它绝不会指出完成工作的唯一方式。引用核心维护者  Mark Erikson 的原话来解释就是：

> _Redux is not intended to be the "most concise way of doing things", but rather to make data flow obvious and readable……Docs are written in a deliberately verbose style for clarity and learning, and not specifically intended as "the one true way to write Redux code",……Redux is a generic framework that provides a balance of just enough structure and just enough flexibility_

话虽如此，但我很难不认为他们自己也在动摇。上面的引用摘自在 Github 上的一次关于 Redux 广泛讨论（这是一次很重要的讨论，有兴趣的同学建议读完）：[Request for Discussion: Redux "boilerplate", learning curve, abstraction, and opinionatedness](https://github.com/reduxjs/redux/issues/2295)。正如讨论的标题所示，以灵活性优先的设计理念，以及为学习而非实践服务的文档内容，给其他开发者带来了与我同身受的问题：高昂的学习成本、高度的抽象以及缺少对最佳实践的指导。

这也是 Redux Toolkit 诞生的原因，从 Mark 对于 RTK 的愿景（[My Vision for Redux Starter Kit](https://github.com/reduxjs/redux-toolkit/issues/82#issuecomment-456261368)）中就不难看出它的目标在此：

*   Make it easier to get started with Redux
*   Simplify common tasks
*   Opinionated defaults guiding towards "best practices"
*   Provide solutions to make people stop using the word "boilerplate"

RTK 在当下已经作为 Redux 项目的标配而存在了，在 Redux 官网的第一章 [Getting Started with Redux](https://redux.js.org/introduction/getting-started) 我们便会看到：Redux Toolkit is our official recommended approach for writing Redux logic——“approach”是个有意思的词，它让我感觉原始的 Redux 框架更像是摸不着的理论而存在——而它的下半句 Redux Toolkit builds in our suggested best practices, simplifies most Redux tasks, prevents common mistakes, and makes it easier to write Redux applications. 将是我们聊技术选型的切入点。

## 从 Redux 到技术选型

### "prevent common mistakes"

我反复向人推荐过一篇 StackOverflow 联合创始人Jeff Atwood 的文章 [Falling Into The Pit of Success](https://blog.codinghorror.com/falling-into-the-pit-of-success/)，简而言之它的中心思想是，好的系统设计应该很容易的让人们把事情做对，杜绝把事情做错。比如 type checking

很显然 RTK 之前的 Redux 并不符合这一条件，甚至与之相悖。因为“容易做对”实践起来务必会削减框架提供的技术选项（甚至你最好只采用这一种方式去做），而 Redux 主要目的是服务于灵活而非最佳实践。另一点是官方用大量段落的文字去讲述 immutable、normalize 的重要性，但是在技术层面却不提供任何约束。

而 RTK 彻底解决了这个问题吗？RTK 解决了代码模板的问题，天然集成了最佳实践，但它移除不掉高昂的学习成本，这是我最担心的。

如果你有兴趣去拿 Redux 文档与前流行的几个 Flux 框架文档进行对比，比如 [Mobx](https://mobx.js.org/README.html)、[Zustand](https://github.com/pmndrs/zustand) 以及 [Akita](https://opensource.salesforce.com/akita/)，你会发现 Redux 所需要掌握的概念和篇幅长度令人发指（很有意思 Angular 是另一个极端，学习 Angular 同样要掌握很多概念，但是对于你想做的每件事情它都帮你想到了，在文档中给出了最佳实践）。我理解工程师热爱“挑战”，但是在现实中，团队是由不同水平的个体组成，团队所能接纳的“难度”常常不尽如人意。如果他们不理解他们的工作，那么他们就很难把工作做好。

###  "simplifies most Redux tasks,  easier to write Redux applications"

Clojure 的作者 Rich Hickey 在 2011 年有一篇很有意思的演讲，名为“[Simple Made Easy](https://www.infoq.com/presentations/Simple-Made-Easy/)”

他给 easy 和 simple 做出了精确的定义

*   Simple：单个事物比如一件任务、一个角色、一个维度；简单的事物不应该包含交织的概念，比如一个实例，一次操作。simple 是客观的
*   Easy：凡是我们熟悉的或者近在咫尺的事物都会让我们感到简单，比如说我们熟悉的语言，使用我们常用的 IDE，easy 是主观的

这两者看起来区别不大，但是他认为的这两者对开发速率的影响是：

![](../images/009_from-redux-to-framework-choice/easy_vs_simple_speed.png)

easy 可以优化你的启动速度，但如果你只是一味的追求 easy 而忽略了复杂性的话长远看还是事倍功半的。比如在 Flux 之前的 [MVC 时代](https://www.v2think.com/fe-arch-004-flux-rise)，在 model 中去更新 view 或者在 view 中直接调用 model 是多么 easy 的事情，但这却让状态管理的复杂性大增。

Redux 算 easy 吗？不，从 Flux 到 Redux 是一整套全新的概念；它算 simple 吗？未必，在你设计 reducer 的时候你很难不去思考 normalize

### 流行

“流行”或者“标配”不应该是技术选型的参考之一。

你的前端项目需要 Redux 吗？你也许会用反问来回答我这个问题：Redux 不是 React 项目的标配吗？

并非如此。无论是在 Hacker News 上网友对于 Redux 的抱怨 （[God I hate redux](https://news.ycombinator.com/item?id=25097329)）里，还是在 reddit 上在对 Redux 的仇恨讨论中（[Why all the sudden hate for Redux?](https://www.reddit.com/r/reactjs/comments/buzlqc/why_all_the_sudden_hate_for_redux/)），Mark 都解释到团队从来都没有以 Redux 作为 React 的主流状态管理工具去营销它，它之所以变得主流一方面是因为它赢得了 2015 年的 Flux Wars，另一方面它切实解决了开发者的问题。Dan 也写过一篇文章来刻意强调你也许并不需要 Redux（[You Might Not Need Redux](https://medium.com/@dan_abramov/you-might-not-need-redux-be46360cf367)）。

如果说我们把 2015 年的 Flux War 比喻成第一次 Flux 世界大战的话，那么当下这个时间点第二次世界大战正进行的如火如荼，在市面上我们可以看到更多优秀的 Flux 框架，比如 [Mobx](https://mobx.js.org/README.html)，比如冉冉升起的 [Zustand](https://github.com/pmndrs/zustand)，比如小众的我很喜欢的 [Akita](https://opensource.salesforce.com/akita/)。Redux 很大可能不会是你下一个新项目的最佳选项

在 StafkOverflow 直白的发文求网友推荐技术框架一类的问题通常都会被关闭。因为在没有任何上下文的前提大部分答案过于主观了。如何思考我建议你至少参考这个回答（[How can Stack Overflow help developers evaluate technologies?](https://meta.stackoverflow.com/a/305678/508236)），先问问自己：

*   我需要什么（Know what you need）
*   我不需要什么（Know what you don't want）
*   我还想要些什么（Know what you want）

但我可以理解“简历驱动开发”是选择当下的流行技术的重要原因，我也深表同情。

---

你可能会喜欢

- [CSS 里的整洁架构](https://www.v2think.com/css-clean-arch)
- [前端架构 101（一）：在谈论它们之前我们需要达成的共识](https://www.v2think.com/fe-arch-001-principles)
- [前端架构 101（二）： MVC 初探](https://www.v2think.com/fe-arch-002-mvc-solved)
- [前端架构 101（三）：MVC 启示录：模块的职责，作用域和通信](https://www.v2think.com/fe-arch-003-architecture-principles)
- [前端架构 101（四）：MVC的不足与Flux的崛起](https://www.v2think.com/fe-arch-004-flux-rise)
- [前端架构 101（五）：从 Flux 进化到 Model-View-Presenter](https://www.v2think.com/fe_arch_005_mvp)
- [前端架构 101（六）：整洁（Clean Architecture）架构是归宿](https://www.v2think.com/fe_arch_006_clean_architecture)
- [【译文】【前端架构鉴赏 01】：Angular 架构模式与最佳实践](https://www.v2think.com/arch-enjoy_angular-architecture-best-practices-zh-cn)
- [【译文】【前端架构鉴赏 02】：可拓展 Angular 2 架构](https://www.v2think.com/arch_enjoy_a-scalable-angular2-architecture-zh-cn)
- [【译文】【前端架构鉴赏 03】：Angular 与 MVP 模式](https://www.v2think.com/arch_enjoy_model-view-presenter-with-angular)
- [微前端说明书](https://www.v2think.com/micro-front-end-specification)
- [从美团这篇文章聊聊微前端的聚合问题](https://www.v2think.com/micro-front-end-aggregation)
- [写给前端看的架构文章(1)：MVC VS Flux](https://www.v2think.com/mvc-vs-flux)
- [从MVC模式在前端开发中的局限性谈起](https://www.v2think.com/starting-from-limitations-of-mvc-in-front-end-development)
- [Flux与Redux背后的设计思想(二)：CQRS, Event Sourcing, DDD](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CQRS-ES-DDD)
- [Flux与Redux背后的设计思想(一)：Command Bus, Event Bus, Service Bus](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CB-EB-ESB)