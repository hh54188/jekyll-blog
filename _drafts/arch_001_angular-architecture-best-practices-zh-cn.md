# 【前端架构鉴赏 01】【译文】：Angular 架构模式与最佳实践



## 前言

前言是作为译者我想说的话，并非原文中的内容。

我猜此时此刻你心里的第一个疑问一定是：为什么是 Angular，不是 React，不是 Vue，不是 Flux，不是 Redux ? 因为你已经对它们太熟悉了。**我个人作为开发者而言最希望是能够汲取到“圈外”的“营养”，这样才能给我的成长带来帮助。固步自封无异于自取灭亡**，我想对各位也是一样。

你不用担心因为不会 Angular 而看不懂这一些列文章，它们基本上谈论的是应用架构——关于设计、组织、抽象，很少会落到具体的实现，即使有，连蒙带猜也能推测出一二。这也能从侧面说明我衷心想推荐这些佳作的原因：通过大段打断的代码阐述很容易；难的是几乎不用代码来跨编程语言的说明高层次的东西，比如 Martin Flowler, Uncle Bob Martin

我不评价框架的流行，好坏和孰是孰非，我只是把一切呈现在各位的眼前。它们并非和 Flux，Vuex 大相径庭，反而你们会看到它们的影子，但更多的是不一样的东西。我在里面看到了更好的职责划分和抽象。

在文中我会以“引用”的格式和“译者注”开头穿插一些我的个人备注和带给我启发性的问题，你可以理解为文章的“评论音轨”。这些问题我会尝试在这个系列的最终的文章中进行回答。

我有一些问题留给你们，也是我曾经自己的问题，如果带着这些问题去阅读和思考，我想会更有帮助：

* Angular 架构某些方面会比 Redux 更好吗？或者反之？如果好，好在哪？如果不好，欠缺在哪？
* 无论哪一种架构（Angular / Flux / Redux / Vuex）一定是最佳解吗？如果不是，哪些常见会不适合它们？如果给你一个机会去改进它们？你会怎么做？
* 想象一个当前一个工作中你正在解决的问题，抛开这些架构，假设需要你来设计一个架构，你是否会比已知的这些架构做的更好？或者只针对当前这个 case 做的更好？
* 当你阅读到在实现同一功能上 Angular 架构中的做法和 React 中不同时，你觉得哪种更好？互换一下呢？

下面正文开始

## 正文

本文原文：[Angular Architecture Patterns and Best Practices (that help to scale)](https://angular-academy.com/angular-architecture-best-practices/)

搭建可拓展性（scalable）的软件是一项具有挑战性的任务。当我们在思考前端应用的可拓展性时，我们会想到递增的复杂度，越来越多的业务规则，应用需要加载越来越多的数据以及遍布全球的分布式团队。为了应对上述的各种因素从而保证高质量的交付和避免技术债的产生，健壮及牢固的架构必不可少。虽然 Angular 自身是具有强烈技术主见的框架，迫使开发者以*恰当*的方式进行开发，但是许多地方依然容易犯错。在这篇文章里，我会展示极力推荐的基于最佳实践和经过实战检验的具有良好设计的 Angular 应用架构。我们在本文的终极目标是学习如何设计 Angular 应用为了维持长时期内的**可持续的开发效率**和**增加新功能的简易程度**。为了达成这些目标我们会使用到

* 应用层次之间恰当的抽象
* 单向数据流
* 反应式（reactive）状态管理
* 模块设计
* 容器（smart）组件模式和纯展现（dumb）组件模式

> 译者注:
>
> 我听过一种说法说 React 是类库，Angular 是框架，而类库和框架区别在于类库是被你所写的代码调用的，而框架是调用你所写的代码。然而这个对 React 真的成立吗？难道 React 配合 Flux 和 Redux 才能组成某种框架吗？以及为什么我们需要 Redux ?
>
> smart component 和 dumb component 是不是很眼熟?

![](./images/angular-architecture-best-practices/bullets.png)

## 目录

* 前端的可拓展性问题
* 软件架构
* 高层次抽象层
  * 展现层
  * 抽象层
  * 核心层
* 单向数据流和反应式状态管理
* 模块设计
  * 模块目录结构
* 容器组件模式和纯展现组件模式
* 总结

## 前端的可扩展性问题

让我们回想一下在开发现代前端应用中面临扩展性问题。当下前端应用不再“仅仅展现”数据和接受用户的输入。单页面应用（Single Page Applications）为用户提供了丰富交互并使用后端作为数据持久层。这也意味着更多的职责被转移到了软件系统的前端。这导致了我们需要处理的前端逻辑变得越发复杂。不止需求一直在增长，连应用需要加载的数据量也在增加。基于这些我们需要照顾到极易受损的应用性能。最后我们的开发团队也在增长（或者至少在轮替，有人来有人走） ，让新加入的开发者尽可能快的融入非常重要

![](./images/angular-architecture-best-practices/meme.jpg)

解决上述问题的方案之一就是坚固的系统架构。但是这需要代价，代价是从第一天起就要拥抱架构。当系统非常小时，足够快的交付功能这对我们开发者来说非常有吸引力。在这个阶段，一起都容易理解，所以开发速度非常快。但是除非我们关心架构，否则经过几轮程序员的轮替，开发奇怪的功能，重构，引入一些新模块之后，开发速度会断崖式下跌。下面的图表展示了在我的开发生涯中通常的情况。这不是任何的科学研究，只代表我时这么看待它的

![](./images/angular-architecture-best-practices/chart.png)

## 软件架构

为了讨论架构的最佳实践和模式，我们首先需要回答一个问题，软件架构是什么。[Martin Fowler](https://martinfowler.com/) 把架构定义为：*“最高层次的将系统拆解为不同的部分（*highest-level breakdown of a system into its parts*）”* 。基于此我会将软件结构描述为关于软件如何将它部分组合在一起并且部分间通信的*规则*和*约束*。通常来说，我们在系统开发中的架构决策在系统演进的过程中很难发生更改。这也是为什么非常重要的是从项目一开始就要对这些决策多加留意，尤其当是我们搭建的软件需要在生产环境中运行多年的话。[Robert C. Martin](https://en.wikipedia.org/wiki/Robert_C._Martin) 曾经说过：软件真正的开销是维护。拥有牢固地基的架构帮助减少系统维护的成本

> **软件架构**是关于如何组织它部分的方式以及部分之间通信的*规则*和*约束*

> 译者注：在上面的翻译中我将 parts 翻译成了“部分”。你或许会认为或者翻译为“组件”听上去更合适，但“组件（components）”在不同的编程语言中有更特定的指向。在 React 中很特殊，一个类就就代表一个组件，但是在 Angular 可能是一个 module，在 Java 里可以是一个 jar 包。通常来说它是比类大的单位。思考题是，如果技术上允许在 React 应用内存在比类大但又比应用小的这么一个单位存在，我们应该按照什么规则组织它？我认为打包时产出的 chunk 或者 bundle 不算，它们是打包优化的产物，而并非是你思考后刻意产生的结果

## 高层次抽象层

首先我们将通过抽象层来分解系统。下图描述了这种分解用的常见概念。思考方向子啊与将**适当的职责**放入**适当的层**中：**核心层（Core）**，**抽象层（abstraction）**或者是**表现（presentation）层**。我们独立的检视每一个层并且分析它的职责。这也的系统分解也指明了通信的规则。比如**表现层**只能够通过**抽象层**与**核心层**进行交谈。之后我们学习这种约束带来的优势

![](./images/angular-architecture-best-practices/layers.png)

### 表现层

让我们从表现层开始拆解分析我们的系统。这里是所有 Angular 组件存在的地方。这一层的唯一职责是**呈现（present）**和**委托（delegate）**。换句话说，它展示界面并且把用户的操作通过抽象层委托给核心层。它知道展示**什么（what）**和做**什么（what）**，但是它不知道用户的交互应该**如何（how）**被处理

> 译者注：这样的处理方式是不是让你想起了什么？是 MVC 还是 MVP ?

下方代码片段涵盖了`CategoriesComponent`组件将用户的交互委托给了来自抽象层的`SetttingsFacade`实例（通过`addCategory()`和`updateCategory()`），并且将一些数据呈现在模板中（通过`isUpdating$`）

```javascript
@Component({
  selector: 'categories',
  templateUrl: './categories.component.html',
  styleUrls: ['./categories.component.scss']
})
export class CategoriesComponent implements OnInit {

  @Input() cashflowCategories$: CashflowCategory[];
  newCategory: CashflowCategory = new CashflowCategory();
  isUpdating$: Observable<boolean>;

  constructor(private settingsFacade: SettingsFacade) {
    this.isUpdating$ = settingsFacade.isUpdating$();
  }

  ngOnInit() {
    this.settingsFacade.loadCashflowCategories();
  }

  addCategory(category: CashflowCategory) {
    this.settingsFacade.addCashflowCategory(category);
  }

  updateCategory(category: CashflowCategory) {
    this.settingsFacade.updateCashflowCategory(category);
  }

}
```

### 抽象层

抽象层在结构表现层与核心层的同时也拥有自己的职责。这一层为展现曾的组件提供**状态流（streams of state）**和**接口（interface）**，负责扮演**外观（facade）**。这种外观将组件在系统里的可见和可为都沙盒化。我们能简单的通过 Angular 的 provider 类来实现接口。这些类或者可以加以 **Facade** 后缀，比如`SettingsFacade`。下方就是外观的一个例子

> 译者注：我认为这里的抽象层对应就是 Martin Fowler 在  *[Enterprise Application Architecture](https://martinfowler.com/books/eaa.html)* 里的*[服务层（Service Layer）](https://martinfowler.com/eaaCatalog/serviceLayer.html)* 。并且“沙盒化”对应的就是服务层里定义的*应用边界（application's boundary）*，即说白了这个服务究竟能干些什么，完全由限定在服务层暴露的接口方法里。

```javascript
@Injectable()
export class SettingsFacade {

  constructor(private cashflowCategoryApi: CashflowCategoryApi, private settingsState: SettingsState) { }

  isUpdating$(): Observable<boolean> {
    return this.settingsState.isUpdating$();
  }

  getCashflowCategories$(): Observable<CashflowCategory[]> {
    // here we just pass the state without any projections
    // it may happen that it is necessary to combine two or more streams and expose to the components
    return this.settingsState.getCashflowCategories$();
  }

  loadCashflowCategories() {
    return this.cashflowCategoryApi.getCashflowCategories()
      .pipe(tap(categories => this.settingsState.setCashflowCategories(categories)));
  }

  // optimistic update
  // 1. update UI state
  // 2. call API
  addCashflowCategory(category: CashflowCategory) {
    this.settingsState.addCashflowCategory(category);
    this.cashflowCategoryApi.createCashflowCategory(category)
      .subscribe(
        (addedCategoryWithId: CashflowCategory) => {
          // success callback - we have id generated by the server, let's update the state
          this.settingsState.updateCashflowCategoryId(category, addedCategoryWithId)
        },
        (error: any) => {
          // error callback - we need to rollback the state change
          this.settingsState.removeCashflowCategory(category);
          console.log(error);
        }
      );
  }

  // pessimistic update
  // 1. call API
  // 2. update UI state
  updateCashflowCategory(category: CashflowCategory) {
    this.settingsState.setUpdating(true);
    this.cashflowCategoryApi.updateCashflowCategory(category)
      .subscribe(
        () => this.settingsState.updateCashflowCategory(category),
        (error) => console.log(error),
        () => this.settingsState.setUpdating(false)
      );
  }
}
```

#### 抽象接口

我们已经知道了这一层的主要职责：为组件提供状态流和接口。让我们从接口开始。公有方法 `loadCashflowCategories()` ， `addCashflowCategory()` 和 `updateCashflowCategory()`从组件中抽离出了状态管理的细节以及外部的 API 调用。我们不在组件中直接使用 API provider (比如 `CashflowCategoryApi`) 因为它们存在于核心层中。组件也并不关心状态是如何变化的。表现层不应该关心工作是**如何（how）**完成的，组件在必要的时候**只需要调用（just call）**抽象层的方法即可（委托）。查看抽象层的公共方法能够让我们快速了解系统这部分的用户用例概况

但是我们应该记住抽象层不是实现业务逻辑的地方。这里我们只是将表现层和业务逻辑*联系（connect）*在了一起，并且把联系的*方式*抽象了出来

> 译者注：Angular 采用了 reactive 编程思想，使用了 RxJS 作为基础类库。“流（stream）”是其中一个重要概念，接下来也会涉及其他和 RxJS 相关的函数和概念 

#### 状态

至于状态，抽象层使得组件独立于状态管理解决方案。组件的模板上被赋值带有数据的 Observable 但并不关心数据是如何产生以及从哪来的。为了管理状态我们能够使用任何支持 RxJS (像 NgRx) 的状态管理类库，又或者仅仅使用 BehaviorSubject 对我们的状态建模。在上面的例子中我们的使用的状态对象的内部实现借助于 BehaviorSubjects （状态对象是我们核心层的一部分）。在 NgRx 的场景下，我们从 store 发起操作。

拥有这样的抽象给了我们很大的灵活性，并且允许我们在更改状态管理方式式而不触碰表现层。甚至可能无缝的迁移到像 Firebase 这样的实时后端，让我们的应用变得**实时（real-time）**。我个人喜欢一开始使用 BehaviorSubjects 来管理状态。如果之后在开发系统的某个时间点有需要使用其他东西，在这个架构下，重构起来非常容易

> 译者注：“状态解决方案”不是必须的，状态管理可以有，架构可以有，但不一定要借助于第三方类库来实现。React 可以有 Redux 架构，但是不一定需要 Redux 框架。仅仅依赖 React 自己的 hook 机制和 provider 就足以实现一套状态管理机制。另一个我想吐槽的是，我不认为 Redux 是一个好的框架，而 NgRx 和 NGXS 并不会却也没有更好的创新 

#### 同步策略

现在让我们更进一步的看看抽象层的重要一面。无论我们选择什么样的状态管理解决方案，我们都可以以乐观或者悲观的方式实现 UI 的更新。想象我们想要在实体的集合中装填一条新记录。集合请求自后端并且在 DOM 中展示；在悲观的实现方式下，我们首先尝试在后端更新状态（比如通过 HTTP 请求），在成功之后我们再更新前端状态。再另一种乐观的实现方式里，我们执行的顺序不同。首先我们会在后端一定会更新成功的假设上立即更新前端。然后我们才发请求更新服务端状态。如果成功了，我们不用做任何事情，但是如果失败了，我们需要回滚前端的更改并且告知用户

> **乐观更新（Optimistic update）**首先改变界面状态然后才尝试更新后端状态。这为我们的用户带来更好的体验，因为不会因为网络延迟看到任何的滞后。如果后的那更新失败了，界面更改必须回滚
>
> **悲观更新（Pessimistic update）**首先更改后端状态并且只有在成功的情况下才更改界面状态。因为网络延迟，通常需要在后端的执行的过程中显示加载进度条

#### 缓存

有时候我们也许会认定来自后端的数据并不会成为我们应用状态的一部分。这也许对我们不会执行任何操作只是把它们（通过抽象层）传递给组件的**只读（read-only）**数据有用。在这个场景下，我们可以把数据缓存在外观中。实现它最简单的方式是使用 RxJS 的 `shareReplay()` 操作符，它能够为流的新的订阅者*重放（replay）*最新的数据。看看下面`RecordsFacade`使用`RecordsApi`为组件请求，缓存并且过滤数据的代码片段

```javascript
@Injectable()
export class RecordsFacade {

  private records$: Observable<Record[]>;

  constructor(private recordApi: RecordApi) {
    this.records$ = this.recordApi
        .getRecords()
        .pipe(shareReplay(1)); // cache the data
  }

  getRecords() {
    return this.records$;
  }

  // project the cached data for the component
  getRecordsFromPeriod(period?: Period): Observable<Record[]> {
    return this.records$
      .pipe(map(records => records.filter(record => record.inPeriod(period))));
  }

  searchRecords(search: string): Observable<Record[]> {
    return this.recordApi.searchRecords(search);
  }
}
```

总结下来，我们在抽象层能做的事情有：

- 为组件提供方法：
  - 把执行逻辑委托给核心层
  - 决定数据的同步策略（乐观或者悲观）
- 为组件提供状态流
  - 选取一个或多个界面状态流（如果有必要的话把它们结合在一起）
  - 从外部 API 中缓存数据

正如我们看到的，抽象层我们的分层架构中扮演了一个非常重要的角色。它清晰的定义了能够帮助我们更好理解和推理系统的职责。依据你的具体例子，你可以给每一个 Angular 模块或者每一个实体创建一个外观。举个例子，`SettingsModule`仅有一个`SettingsFacade`。但有时为每一个实体创建更细力度的抽象外观会更好，比如`UserFacade`对于`User`实体而言

### 核心层

最后一层是核心层，这也是应用的核心逻辑实现的地方。所有的**数据操作（data manipulation）**和与**外界的沟通（outside world communication）**都发生在这里。如果我们使用 NgRx 作为状态管理方案的话，这里就是放置 state，actions 和 reducer 的地方。因为我们的例子中使用 BehaviorSubjiect 对状态建模的缘故，我们可以把它封装在一个便携的状态类中。你可以在下面找到来自核心层的 `SettingsState` 的例子

```javascript
@Injectable()
export class SettingsState {

  private updating$ = new BehaviorSubject<boolean>(false);
  private cashflowCategories$ = new BehaviorSubject<CashflowCategory[]>(null);

  isUpdating$() {
    return this.updating$.asObservable();
  }

  setUpdating(isUpdating: boolean) {
    this.updating$.next(isUpdating);
  }

  getCashflowCategories$() {
    return this.cashflowCategories$.asObservable();
  }

  setCashflowCategories(categories: CashflowCategory[]) {
    this.cashflowCategories$.next(categories);
  }

  addCashflowCategory(category: CashflowCategory) {
    const currentValue = this.cashflowCategories$.getValue();
    this.cashflowCategories$.next([...currentValue, category]);
  }

  updateCashflowCategory(updatedCategory: CashflowCategory) {
    const categories = this.cashflowCategories$.getValue();
    const indexOfUpdated = categories.findIndex(category => category.id === updatedCategory.id);
    categories[indexOfUpdated] = updatedCategory;
    this.cashflowCategories$.next([...categories]);
  }

  updateCashflowCategoryId(categoryToReplace: CashflowCategory, addedCategoryWithId: CashflowCategory) {
    const categories = this.cashflowCategories$.getValue();
    const updatedCategoryIndex = categories.findIndex(category => category === categoryToReplace);
    categories[updatedCategoryIndex] = addedCategoryWithId;
    this.cashflowCategories$.next([...categories]);
  }

  removeCashflowCategory(categoryRemove: CashflowCategory) {
    const currentValue = this.cashflowCategories$.getValue();
    this.cashflowCategories$.next(currentValue.filter(category => category !== categoryRemove));
  }
}
```

在核心层里，我们以 provider 类的形式实现 HTTP 查询，这种类有 `Api`或者`Service` 名称后缀。API 服务只有一个职责——除了与 API 端点通信外别无他用。我会应该避免任何的缓存，逻辑又或者数据操作。一个简单的 API 服务的例子如下：

```javascript
@Injectable()
export class CashflowCategoryApi {

  readonly API = '/api/cashflowCategories';

  constructor(private http: HttpClient) {}

  getCashflowCategories(): Observable<CashflowCategory[]> {
    return this.http.get<CashflowCategory[]>(this.API);
  }

  createCashflowCategory(category: CashflowCategory): Observable<any> {
    return this.http.post(this.API, category);
  }

  updateCashflowCategory(category: CashflowCategory): Observable<any> {
    return this.http.put(`${this.API}/${category.id}`, category);
  }

}
```

在这一层中，我也会拥有校验，映射或者更多需要操作界面状态的高级用例。

我们以及涵盖了我们前端应用的关于抽象层的话题。每一层都有它恰当的边界和职责。我们也定义了层之间严格的通信规则。随着时间的推移当系统变得越来越复杂时这些所有都将更好的版主理解和推测它

## 单向数据流和反应式状态管理

下一个我想介绍的我们系统中的另一个原则是数据流和传播变化（propagation of change）。Angular 自身在表现层使用单向数据流（通过输入绑定的方式），但是我们也能在应用层面加以相同的限制。与（基于流式的）状态管理一起，它会赋予我们系统一个非常重要的属性——**数据一致性（data consistency）**。下图呈现了这种单向数据流的大致想法

![](./images/angular-architecture-best-practices/flow abstract.gif)

无论何时我们应用中的模型值发生了变化，Angular 的变化监测系统都能够处理变化的传播。它通过对整棵组件树**自顶向下（the top to bottom）**的输入属性绑定实现。它意味着孩子组件只能依赖父组件，并且永远不会依赖反转。这也是为什么我们称它为单向数据流。这允许 Angular 只会遍历组件树**一次（only once）**（因为树的结构中不存在循环）就能够取得一个稳定的状态，也意味着绑定里的值都能都能得到周知

> 译者注：
>
> 1) 这里直接介绍和安利自顶向下的单向数据流机制。为什么不是能自低向上？为什么不能通过事件？
>
> 2) 如果你有 Angular 1.x （又被称为 AngularJS）的经验的话，AngularJS 的类似的脏检查机制并非如此。AngularJS 中脏检查被称为 Dirty Check，而 Angular 中被称为 Change Detection. Dirty Check 会不停的轮询所有被检视的变量直到没有变化发生。具体可以参考我的[上一篇文章](https://zhuanlan.zhihu.com/p/100038957)

从前几章我们得知在表现层之前还有核心层，也就是我们应用逻辑实现的地方。那里有 services 和 providers 服务于我们的数据。如果我们把同样的原则也应用于那一层的数据处理上会怎么样？我们把应用数据（状态）放一个一个所有组件“之上”的地方，并且让值借助 Observable 流（Redux 和 NgRx 称之为 store）向下进行传播。状态能够传播到多个组件并且显示在多个地方，但是从不会在展示处发生修改。这些更改只会“来自上方”并且下方的组件只会“反映”系统的当前状态。这给予了我们系统上面提到的最重要的特性——**数据一致性（data consistency）**——状态对象变成了**唯一数据来源（the single source of truth）**。实际上说，我们可以多个地方展示同一份数据然后不用担心值会不同

我们的对象状态为了核心层的各种服务提供方法用于操作状态。当有需要改变状态时，只需要调用状态对象上的一个方法（在 NgRx 的例子里时触发一个 action）。接着，变更就会通过流“向下”传播给表现层（或者其他的服务）。这种方式就意味着状态管理是*反应式（reactive）*的了。不仅如此，因为严格的操作和共享状态的规则，我们可以增加我们系统的可预测性。下方是使用 BehaviorSubject 对状态建模的代码片段

```javascript
@Injectable()
export class SettingsState {

  private updating$ = new BehaviorSubject<boolean>(false);
  private cashflowCategories$ = new BehaviorSubject<CashflowCategory[]>(null);

  isUpdating$() {
    return this.updating$.asObservable();
  }

  setUpdating(isUpdating: boolean) {
    this.updating$.next(isUpdating);
  }

  getCashflowCategories$() {
    return this.cashflowCategories$.asObservable();
  }

  setCashflowCategories(categories: CashflowCategory[]) {
    this.cashflowCategories$.next(categories);
  }

  addCashflowCategory(category: CashflowCategory) {
    const currentValue = this.cashflowCategories$.getValue();
    this.cashflowCategories$.next([...currentValue, category]);
  }

  updateCashflowCategory(updatedCategory: CashflowCategory) {
    const categories = this.cashflowCategories$.getValue();
    const indexOfUpdated = categories.findIndex(category => category.id === updatedCategory.id);
    categories[indexOfUpdated] = updatedCategory;
    this.cashflowCategories$.next([...categories]);
  }

  updateCashflowCategoryId(categoryToReplace: CashflowCategory, addedCategoryWithId: CashflowCategory) {
    const categories = this.cashflowCategories$.getValue();
    const updatedCategoryIndex = categories.findIndex(category => category === categoryToReplace);
    categories[updatedCategoryIndex] = addedCategoryWithId;
    this.cashflowCategories$.next([...categories]);
  }

  removeCashflowCategory(categoryRemove: CashflowCategory) {
    const currentValue = this.cashflowCategories$.getValue();
    this.cashflowCategories$.next(currentValue.filter(category => category !== categoryRemove));
  }
}
```

让我们基于我们已经学习到的原则复习一遍处理用户交互的步骤。首先让我们想象在表现发生了一些事件（比如点击按钮）。组件把执行委托给了抽象层，调用外观上的 `settingsFacade.addCategory()`方法。接着外观调用核心层里服务上的方法——`categoryApi.create()`和`settingsState.addCategory()`。这两个方法调用的顺序取决于我们选择的同步策略。最终，应用状态通过 observable 流传递给表现层。这整个流程**清晰明确（well-defined）**

![](./images/angular-architecture-best-practices/flow.gif)

## 模块设计

我们已经讲解了系统的横向拆分以及它们之间的通信模式。现在我们即将介绍如何纵向划分为功能模块。出发点是将应用划分为代表不同业务功能的**特性模块（feature modules）**。这也是为了更好的可维护性将系统分解为更小单元的另一个步骤。每一个特性模块共享横向划分的核心层，抽象层和展现层。非常值得注意的是这些模块可以在浏览器中懒加载（和预先加载）以减少应用的初始加载时间。下方是这种特性模块划分图解：

![](./images/angular-architecture-best-practices/modules.png)

出于技术愿意我们的应用也有两个额外的模块。我们有一个`CoreModule`用于定义我们的单例服务，单实例组件，配置，并且向 `AppModule`中导出任何的需要的第三方模块。这个模块只会在 `AppModule`中导入一次。第二个模块是`SharedModule`，它包含公共的 components /pipes/directives 也包含公用的 Angular 模块（比如 `CommonModule`）。`SharedModule`可以被特性模块导入。下图展示了导入结构

![](./images/angular-architecture-best-practices/imports.png)

> 译者注：或许你阅读到这里的时候已经注意到了，与 React 不同，在 Angular 中有更多类型的角色定义，比如 provider，service，以及上面说的 `CommonModule` , `SharedModule` 等等。你觉得这些详细的职责划分是好事还是坏事？它们是有助于开发还是让开发变得更繁琐了？

### 模块目录结构

下图呈现了我们是如何将所有的`SettingsModule`片段放入目录结构中的。我们可以把文件置于名字能显示它们功能的文件夹中

![](./images/angular-architecture-best-practices/module.png)

## 容器组件模式和纯展现组件模式