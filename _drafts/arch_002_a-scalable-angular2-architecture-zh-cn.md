# 【前端架构鉴赏 02】【译文】：可拓展 Angular 2 架构

## 序

这篇文章或许和 [Angular2](http://angular.io/) 开发者更相关，但是我相信它也适用于其它的框架。这只是一本关于编写具有可拓展性和可维护性单页面应用的指南。非常重要的是，这不是达成目标的唯一方式，但是对我个人而言在不少的场景中都行之有效

## 编写可拓展性的单页面应用

许多开发者在编写大型的具有可拓展性和可维护性的单页面应用时都遇到困难。结果是在开发早期就留下了技术债，修复 bug 时也受挫，当编写测试和创建复用代码时也踌躇不前

最大的一个挑战是：在一个奇怪的基础之上拓展现有逻辑和编写新的逻辑

单页面应用是全新的概念，特别是旧的框架能允许你用100种不同方式设计一个应用。那里没有结构和封装可言，一切东西都紧紧的耦合在一起。

大部分时候在项目的开发的初始阶段都希望快速迭代。但是经过一些开发者，几轮功能迭代和重构之后，代码变得越来越难以维护。它开始看上去像意大利面了。目前框架成熟了很多，但重要的是你编写的软件架构也要与时俱进

## 2016 年的单页面应用（甚至更早之前）

为了解释这篇文章谈到的架构，有必要回顾一下2016年的web应用长什么样子。这些概念是你的应用在今天也会用到的。无论使用的是 [React](https://facebook.github.io/react/)，[Angular 2](http://angular.io/) 又或者其它的框架。这些已知的原则能够让你的 web 应用变得易于维护

### 原则1：组件

像 React 和 Angular 2 这样的 SPA 技术让我们使用组件。一个组件是 HTML 片段和 JavaScript 和结合。我们不想再使用独立的 view 和 controller。它们会出现爆炸式增长并且相互关联导致难以维护

> 你真的明白这最后一句话 “爆炸式增长” 和 “相互关联导致难以维护” 是什么意思吗？为什么会出现这样的情况以及这究竟是怎样一个场景？

所以最基本的原则是：**万物皆是组件（everything should be a component）**，甚至你的页面和应用也是。一个应用可以长成这个样子：

```javascript
<application>
	<navbar fullname="Brecht Billiet" logout="logMeOut()">
	</navbar>
	<users-page>
		...
		<grid data="users">
		</grid>
		...
	</users-page>
</application>
```

一些在设计组件时非常简单但又重要的提示是：

- 保持它们尽可能的小
- 保持它们尽可能的仅与绘制界面相关

> 译者注："仅与绘制界面相关" 在原文中就一个词：dumb, 即 dumb component。仅负责绘制界面但不包含业务逻辑和数据存储的组件

如果你是设计组件的新手，这篇文章或许能帮助你：[components demystified](http://blog.brecht.io/components-demystified)

**注意：**聪明组件（smart component）也被称为结构组件（structural components），容器组件（containers）或者含状态组件（stateful componnets）

### 原则2：单向数据流

在此之前我们通过一种非常没有效率的方式更新应用的状态：

- 我们尝试让兄弟组件相互通信
- 父组件尝试通过 action 通知子组件
- 我们尝试在不同的组件间发送事件
- 我们使用单向绑定，双向绑定
- 我们把数据模型注入的到处都是用以共享状态

你有尝试过让兄弟组件相互通信吗？有时候这么做看上去理所应当，但是请不要这么做

![](./images/a-scalable-angular2-architecture/multidirectionaldataflow.png)

**这是一个糟糕的设计！**这样的话几乎不能看清数据的流动方向。这样的代码也非常难以维护，修复 bug 又或者开发功能。我们确切想要的是像 [Flux](https://facebook.github.io/flux/) 和 [Redux](http://redux.js.org/) 的单向数据流

它基本上的工作原理是这样的：子组件只会通知它们的父组件，父组件（容器组件）向包含状态的 store 发送一个 action，action 会更新整个应用状态。当状态被更新之后，我们会重新计算组件树。结果就是数据。结果就是数据朝相同的方向流动（向下）。

![](./images/a-scalable-angular2-architecture/unidirectionaldataflow.png)

这种方式的最大好处是：

- 使得组件之间解耦
- 好的可维护性
- 切换为实时应用的代价较小，因为软件是反应式的（reactive）
- 通过监控 action 我们就知道发生了什么

如果你是单向数据流的新手请访问 [introduction to redux](http://redux.js.org/docs/introduction/) 关于 [单向数据流（unidirectional dataflow）](http://redux.js.org/docs/basics/DataFlow.html) 的部分

## 可拓展架构

我在设计一些架构时基于的时当今原则（大部分上面都解释过了）以及甚至是十年前的原则，我就能说这对每个 SPA 都是成立的吗？不尽然……每一种软件类型都值得拥有它自己的架构，我只是展示一种对于我而言在大多数场景下都成立的理念。这个架构对于正在受益于依赖注入特性的 Angular 2 开发者而言迟早会有用，但也能适用于其它的框架中

### 有意义的抽象

这个原则部分是基于  [Nicholas Zakas](https://twitter.com/slicknet) 的[沙盒原则（Sandbox principle）](http://www.slideshare.net/nzakas/scalable-javascript-application-architecture)，它已经有一些年头了。对于 Nicholas Zakas 而言，沙盒的扮演着存在于包含一个容器组件的几个不同模块间的调度器（dispatcher）。在那个架构中，没有单向数据流。对我而言一个沙盒是**一种将应用逻辑与表现层解构的方式**，但那并不是它唯一的职责。

但是让我们从头开始……在这个具体的场景中我将假设我们使用 [Redux](http://redux.js.org/)，但其实无论你使用什么样的状态管理，了解它之后的原理非常重要。

### 规则一：不要让你的组件接触到所有的玩具

这和你不能让孩子玩耍所有的东西背后的原因一样：**“可能会变得一团糟”**。容器组件应该遵循一套严格的规则。。不能因为我们能做到而把我们想要的每一个服务都注入进去。举个例子：把游戏引擎注入仅权限模块死似乎就没有任何意义

下面的例子展示了一个拥有非常多依赖的构造函数。在这个场景里**`MyComponent`**可以在应用里做任何它想做的事情。任何它想要的想要做的都被注入其中。这不是好主意

```javascript
export class MyComponent{
	constructor(...,private foo:Foo, private bar: Bar, 
		private store: Store<ApplicationState>, private authService: AuthService,
		private fooHttpService: FooHttpService, private barMapper: BarMapper, ...){
	}
}
```

上面的例子的构造函数里有大多的依赖了。它与太多应用里的其它部分产生了联系。当你向复盘这是一种怎样的设计时结果看上去像这个样子：（REST 表示 restful 服务，所以它们只是 HTTP 模块）

![](./images/a-scalable-angular2-architecture/abstraction_step1.png)

这看上去就像意大利面代码，一切都是紧耦合的。一个抽象层能够给予一些帮助。下面的例子中我们看到表现层已经完全解耦，抽象层接管了所有事情

![](./images/a-scalable-angular2-architecture/abstraction_step2.png)

抽象层代码大概像这个样子：

```javascript
export class MyComponent{
	constructor(private abstraction: SomeAbstractionType){
	}
	
	doSomething(): void{
	this.abstraction.doSomething();
}
```

### 规则二：组件应该对状态管理层不知情