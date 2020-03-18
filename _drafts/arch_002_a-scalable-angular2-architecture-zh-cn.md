# 【前端架构鉴赏 02】【译文】：可拓展 Angular 2 架构

原文 [https://blog.strongbrew.io/A-scalable-angular2-architecture/](https://blog.strongbrew.io/A-scalable-angular2-architecture/)

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

纯展示组件和容器组件都不应该知道你用的是 Redux 还是其它的状态滚利层。它们不应该关心状态时如何被管理的。它们只是相信状态被管理的很好。管理也不是它们的职责。表现层的职责是**“展现”**和“**委托”**

下面的代码片段是一个糟糕设计下组件和 redux 紧耦合的反面例子（在这个例子中我们使用 [ngrx/store](https://github.com/ngrx/store) ）

```javascript
export class MyComponent{
	// we just care about the state, not where it comes from...
	users$ = this.store.select(state => state.users);
	foo$ = this.store.select(state => state.foo);
	bar$ = this.store.select(state => state.bar);
	constructor(..., 
		private store: Store<ApplicationState>, ...){
	}
	
	addUser(user: User): void{
		// we don't care about the actiontype, payload or store
		this.store.dispatch({type: ADD_USER, payload: {user}}
	}
	removeUser(userId: string): void{
		// we don't care about the actiontype, payload or store
		this.store.dispatch({type: REMOVE_USER, payload: {userId}}
	}
}
```

这样或许更加清晰，也更松耦合：

```javascript
export class MyComponent{
	users$ = this.facade.users$;
	foo$ = this.facade.foo$;
	bar$ = this.facade.bar$;
	
	constructor(private facade: ...){
	}
	
	addUser(user: User): void{
		this.facade.addUser(user);
	}
	removeUser(userId: string): void{
		this.facade.removeUser(userId)
	}
}
```

现在组件真正的聚焦在它自己的职责上了。它不知道用户是如何被添加或者移除的，它不知道谁产生的 `users$` 流，`foo$`流和`bar$流`。它把这些职责都托付给抽象层。现在，表现层已经完全和剩余的应用部分解耦了。给我们带来以下的优势：

- 我们终于有了封装
- 便于测试，我们只需要 mock 抽象层
- 如果抽象层保证它的接口不发生变化，它们就能并行开发
- 它响应抽象层，因此实时开发变得容易许多
- 表现层与应用中其它的部分不再耦合，所以重构变得更加容易

### 规则三：HTTP 服务不应该知道状态管理层

第一眼看上去，似乎把处理 GET 请求放在含有 store 的服务中是顺理成章的事情。但是一个 HTTP 服务的唯一目标是**发出 HTTP 请求并且返回这些请求的结果**。顺理成章的代码看上去长这样：

```javascript
export class UserService{
	// expose the users$-stream directly in the service
	users$ = this.store.select(state => state.users);
	
	constructor(private store: Store<ApplicationState>, private http: Http){
	}
	
	fetchUsers(): void{
		this.http.get("...")
			.map(...)
			.subscribe((users) => {
				// when successful, put the users in the store
				// is this really my responsibility?
				this.store.dispatch({type: SET_USERS, payload: {users}});
			});
	}
}
```

但是现在 http 服务于状态管理系统变得非常的紧耦合

![](./images/a-scalable-angular2-architecture/abstraction_step3.png)

http 服务的唯一职责应是如此

```javascript
export class UserService{
	constructor(private http: Http){
	}
	
	// just let the consumer of this service handle the store interaction
	// this will just return a stream of users
	fetchUsers(): Observable<Array<User>>{
		return this.http.get("...").map(...)
	}
}
```

### 一个可能的解决方案

这当然不是唯一的解决方案，但是它对我来说绝大部分时候都管用。每一个展现模块能够访问它自己的沙盒（sandbox），沙盒一个有以下功能服务：

- 状态流（在这个例子中从 redux 中选取）
- （消费沙盒的）展现模块可能执行的方法

但是**它不仅仅是一个外观**:) 它应该有部分的逻辑：

- 从 store 中抓取正确的状态碎片
- 向 store 派遣拥有正确类型和正确负载的操作
- 委托给不同模块里正确的服务
- 处理[乐观更新(optimistic updates)](http://blog.brecht.io/Cancellable-optimistic-updates-in-Angular2-and-Redux/)
- 如果使用了类似于 firebase 的服务，暴露来自 firebase 的正确数据流

![](./images/a-scalable-angular2-architecture/sandboxes.png)

沙盒的好处有

- 将表现层与其它部分解耦
- 抽离状态管理
- 能够看清一个模块究竟能访问什么
- 组件没法为所欲为，更好的封装
- 把乐观更新逻辑放对了地方，因为组件和服务都不关心这个功能。
- 在不用重写服务和组件的情况下能够切换到不同的状态管理下面

## 总结

这整个架构是其中的一种方式，但并不意味着是唯一方式。

我们相信封装、松耦合以及正确的职责体系是非常重要的。前端领域更新的非常快，也意味着我们想要对于重构和想尝试一些新技术来说更加灵活。我希望你喜欢这篇文章

## 想要学习更多

访问 [strongbrew.io](http://strongbrew.io/)，[我](http://twitter.com/brechtbilliet)和[Kwinten Pisman](http://twitter.com/kwintenp)在上面开设一个了“反应式应用工作坊（reactive applications workshop）”来更谈论这个主题更多的细节

## 特别感谢

我想要感谢所有评审这篇文章和给予价值输入的人。感谢  [Jurgen Van de Moere](http://twitter.com/jvandemo)，[Carmen Popoviciu](http://twitter.com/carmenpopoviciu)，[Manfred Steyer](http://twitter.com/manfredsteyer) 和 [Juri Strumpflohner](http://twitter.com/juristr)！！！