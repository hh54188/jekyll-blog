---
layout: post
title: 前端架构 101（四）：MVC的不足与Flux的崛起
modified: 2020-06-09
tags: [javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## MVC 的不足

### 事件

在前几篇中，我演示了一个前端 Backbone.js MVC 框架用于解决实际问题的例子。但 MVC 依然存在几个问题

- 不可预测：当一个事件发生之后，你并不知道会有谁响应这个事件，是单个对象还是多个对象会响应这个事件
- 级联修改：当一个事件发生之后，A 组件在接收到事件之后在响应的过程中，还可能发出其他的事件触发后续的修改，你并不知道这个事件会在何处结束，会造成什么样的结果。这也和上一条「不可预测」相对应
- 响应顺序：如果存在多个对象响应同一个事件的话，有时候对响应的顺序是有要求的，某些变更不可以出现在其他的变更之前
- 有条件响应：对于传播方而言，并非希望所有的时间都一视同仁的广播出去；对于消费方而言，也并不希望一视同仁的响应所有的事件

你可能会认为事件机制存在的问题是否只存在于 Backbone.js 中，那 AngularJS 这个 MVC 框架会不会好一些呢？

首先 AngularJS（AngularJS 代指 1.x 版本，Angular 代指 2 以及之后的版本） 框架中也支持全局的事件机制，比如 `$broadcast`,`$emit` 等等。这样的事件机制支持变化从 `$rootScope` 向各个 contoller 的 `$scope` 广播全局的变化。如果你对 scope 这个概念不熟悉的话，可以把它理解为模块内部的作用域。

![](../images/fe_arch_004_flux_rise/broadcast-emit-angularjs.png)

### 双向绑定 

AngularJS 更重大的缺陷在于它的双向绑定机制，或者说是双向数据流 (bidirection data flow) 。也就是说 A 可以把变量传递给 B，当 B 修改这个变量之后，A 中对应的变量值也会发生修改。咋听之下似乎是非常方便的机制，例如在表单这个场景中会非常实用，但是它存在一些隐患。我们以下图中的这个场景为例：

![](../images/fe_arch_004_flux_rise/angularjs_bidirection.png)

1. Parent Controller 把某个变量以双向绑定的机制传递给 Child A Controller 

2. 此时用户在界面上对这个变量值进行了修改

3. 因为双向绑定的缘故这个值同步到了 Parent Controller 中

4. 同时 Child B Contoller 和 Parent Controller 也通过双向绑定把值同步到了 Child B 中，此时 Child B 中的值也发生了修改

也就是说，当你修改 Child A 中的一个值时，你会影响到 Child B 中的值。这样的副作用是危险的，除非你对整个系统里用到这个值的地方了如指掌，否则你极有可能影响到你不愿意被你影响到的地方。

如果 Child A 和 Child B 属于不同的开发人员进行开发, 那么 Child B 的开发人员在排查这个问题是会非常困难，因为站在他的视角上而言，他只知道这个值来自于 Parent Controller，但是这个值又被哪些地方消费了，哪些地方修改了他并不知道。在框架机制内不支持这样的追溯。此时你只能保佑关于这个变量有一个 setter 方法，又或者通过 IDE 的查找功能在代码里全局搜索用到这个变量的地方

### 职责不明确

回忆一下我在第二篇中列举的 Backbone.js 和 AngularJS 实现的例子，无论是 view 文件还是 controller 文件，其它们的职责并不明确，他们同时在负责好几件事情：

- 管理 view model，例如负责保存和清空用户输入的值
- 协调用户流程，例如首先将用户输入值清空，然后提交新数据，再刷新数据列表
- 负责为不同的 dom 元素绑定事件处理函数

不说大道理，和当下的 React 或者 Angular 组件相比，直接后果是这些模块是无法复用的。如果我想重复使用一个 view 的话，我需要保证我的页面模版里有相同的 id 的元素，又必须保证上下文中有相同 model 层提供相同的借口或者广播相同的事件。关于多职责的坏处在上一篇中已经聊过，就不赘述了。

### 总结

批评不等于否定。事件机制依然是我们许多问题里可选的解决方案之一；Backbone.js 和 AngularJS 放在现在看也依然是优秀的解决框架，但不是最优解而已。

我个人认为问题在于当下我们解决的问题和过去比发生了许多的变化，随着浏览器能力不断增强，前端需要解决的问题也变得越来越复杂，团队规模也逐渐扩大。如果以 React 步入公众视野的 2014 为节点的话（我以）。2014 年以前我们的开发主要集中在类似于 widget / plugin 级别的功能上；而在 2014 年之后应用级别的功能慢慢变得普及起来。

如果你对比 2014 年以后和之后流行或者崛起的那些框架，你就会感受到其中的微妙之处：

- 2014 年前：jQuery, Bootstrap, RequireJS, Kissy, Handlebars
- 2014 年后：Redux, Ngrx, Mobx, Akita, Ngxs

前者倾向于碎片化，各司其职的辅助性的功能；后者倾向于应用级别的数据管理

事件机制和双向绑定更适用于小规模的范围内，随着应用级别不断扩大，副作用的带来负面效用会变得越来越明显。

## Flux

我把所有与 Flux 相似的框架在这里都称之为 Flux。包括但不限于：Redux，Mobx，Ngrx，Akita，React 等等。在我看来它们都拥有和 Flux 相同的特征：

- 单向数据流
- 全局状态管理
- store / selector / service 等概念的抽象

在谈论 Flux 之前我们先给 Flux 定一个性：Flux 是成功的吗？

当然是，如今不计其数的网站也应用在使用 React 和 Flux；并且就像我上面提到的，即使是六年以后，在它之后的框架绝大部分是它的追随者而非颠覆者，都能找到 Flux 的影子。但在它诞生之初，无论是在 [Reddit](https://www.reddit.com/r/programming/comments/25nrb5/facebook_mvc_does_not_scale_use_flux_instead/), [Youtube](https://www.youtube.com/watch?v=nYkdrAPrdcw)，还是 InfoQ 上甚至至今为止都有批评的声音，

但在你的那些使用了 Flux 的项目中，有多少项目在可维护性上是成功的？如何定义可维护性呢，我们用 Uncle Bob 的三个标准来回答这一个问题：

> - It is hard to change because every change affects too many other parts of the system.(Rigidity)
>
> - When you make a change, unexpected parts of the system break. (Fragility)
>
> - It is hard to reuse in another application because it cannot be disentangled from the current application. (Immobility)
>
>   —— [The Principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod)

我相信答案在各位心中已经呼之欲出了。站在工程师的角度上看项目代码的可维护性并不取决于你使用的框架多么的先进，而是取决于使用框架的**人**和内部的工程师**文化**

扯远了，说回 Flux。

在这里我不会再聊 Flux 的那些基本入门概念。我们重点说一说 Flux 解决的问题，帮助你更好的理解 Flux

不知道有多少人看过 Flux 走向公众视野的第一个视频 [Hacker Way: Rethinking Web App Development at Facebook](https://www.youtube.com/watch?v=nYkdrAPrdcw) ，这个视频上透露了很多有助于关于我们理解 Flux 的很多信息。这一节的内容不少复述自该视频

首先要强调的是，Flux 并非是为了颠覆和创新而生，而是为了解决我们所说的非功能性需求。

在 Facebook 公司内部工程师需要保证交付软件的质量，但是高质量意味着需要花费更多的时间；但另一方面公司也希望崇尚 move fast 的核心价值，也就是要用更少的时间交付更多的价值，于是这两者间似乎产生了矛盾，如何用更少的时间交付更高质量的软件。

用视频中的原话说，按照顺序他们想达成的目标是

- Produce higher (quality) code
- Higher quality software
- Better code by default
- We want to do it in less time

![](../images/fe_arch_004_flux_rise/less_time_high_quality.png)

其中有意思的是第三点：「Better code by default」。在我看来这就是我在第一篇中强调的 「Falling Into The Pit of Success」有异曲同工之妙（你也可以说我现在眼里只有锤子看什么都是钉子）——你要让你的开发人员一开始（容易）就写出对的代码。

而在他们的项目中最大的阻碍竟然是 MVC 架构

整个宣讲 Flux 过程中最令人诟病的就是这一张图，在我上面提到的批评声音中，最共同的声音就是它们以一种错误的方式实施了 MVC，所以才导致了他们的应用无法拓展。时候演讲者 Jing Chen 也[承认演示中的图片确实投机取巧了。它们真正想表达的是这种双向的数据流架构会产生一定的负面效应](https://www.reddit.com/r/programming/comments/25nrb5/facebook_mvc_does_not_scale_use_flux_instead/)。

![](../images/fe_arch_004_flux_rise/facebook_mvc.png)

首先就像我在前几篇中提到的那样，从客户端到后端到前端并没有“标准的 MVC” 一说。即使你只在前端领域内寻找统一的 MVC 概念，你也会发现从 Backbone.js, AngularJS 到 Ember.js 的实现各不相同。

正因为大家对 MVC 的理解各不相同，甚至在同一个框架内也没有推荐的最佳实践，于是你会看到在一个框架内解决一个问题的不同实现。其中有一些方案是存在隐患的，但是在小规模的应用内很难暴露出来。但随着团队的扩充和复用代码的越来越多，代码会变得越来越脆弱，因为不同人看到同一份代码的理解是不同的。上图中的情况是非常有可能发生的，但并非是按照上图一模一样的方式，但后果就是跨职责和意料之外的级联更新。

如果你现在站在开发 React 应用的体验上看 Backbone.js 和 AngularJS 的开发体验，你会感觉框架带来的约束是松散的。以 AngularJS 为例，它赋予了你 controller / view 机制，但至于是在多个 view 之间共享 controller，又或者相对于一个 view 嵌套多层 controller，它完全不做任何的限制。这就极易产生上述后果。在下图中 View C 可以访问和修改多个祖先 controller 中的变量（左侧黄色箭头）同时变量又有可能会被 View B 和 View C 使用（右侧蓝色箭头）。

![](../images/fe_arch_004_flux_rise/angularjs_nest_controllers.png)

所以你现在理解了为什么 Flux 会尝试用单向数据流解决这个问题了。我们抽取 store 来保证唯一数据源（single source of truth），所有的业务逻辑也都封装在 store 中，避免了用例和服务层（对应后端 service layer）方法散落在各个 controller 中。注意 store 层工作是不会引起任何的副作用的，在 store 完成上一个 action 的工作之前，不会有其他的 action 再次经过 dispatch 达到 store。同时使用 command 模式来避免事件机制造成的的不可预测性。剩下的具体概念你应该非常熟悉了

现在回过头再看 Flux，它其实是一个非常强约束的框架。假设你需要完成一项工作，比如接住后端传递的用户信息里的新增字段，你会非常明确的知道你需要修改 store, 该 view，而不需要修改 action。到了在 store 中新增字段的这一个环节，无论是你是使用 Redux 还是 Mobx 相信你都能迅速的找到对应的 model / reducer 在哪。我想这就是 期望中的 「Better code by default」 吧

## 简单聊框架的约束

就像我一开始提到的，目前的框架倾向于为你提供应用级别的整套解决方案。并且极其详细的为你划分模块。最初我们只有 model, controller 和 view；但现在我们有 component, store / reducer, action, dispatch, selector / query, reducer, service, effect, 甚至在有的框架中还有更细化的 entity store, entity query。所以当你现在需要开发一个功能时，你能够很轻易的把你的需求拆解为对应的模块，分别把它们开发、测试完毕之后接入应用即可。

有人认为如此强的职责划分和框架约束扼杀了编程的创造力和乐趣。但我认为工业化的代码产出稳定和高效才是最重要的。

现代的前端技术栈已经变得非常复杂了，「精通」已经成为了一件奢侈的事，更别说让整个团队达到相同的「精通」水平。如果你开发的是 Angular 应用，Angular 本身,或是 Rxjs 又或是 TypeScript 哪一个单拎出来都不好对付，指望着人们自我学习或者培训的方式统一大家的水平更是天方夜谭。目前看来「约束」看起来是最简单也是最靠谱的方式。这种约束体现的不仅是在模块的明码~~标价~~设计上，还可以体现在使用 lint，TypeScript 等语法的约束上 

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