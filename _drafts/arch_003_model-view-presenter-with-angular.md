# Angular 与 MVP 模式

随着应用程序的日趋庞大，它变得更难维护。复杂性也随着可复用模块的价值增长。我们都知道我们应该在它面临失败风险前做些什么

设计模块能够拯救它！

## 复杂应用

一个复杂应该至少拥有以下其中一个特征

- 组件树中的多个组件展示同一份应用状态
- 拥有多个更新应用状态的来源：
  - 多个用户同时交互
  - 后端实时推送状态更新给浏览器
  - 后台定时任务
  - 近距离传感器或者其它设备传感器
- 频繁的更新应用状态
- 大量的组件
- 代码量大的组件，回想一下之前的[大泥球](http://www.laputan.org/mud/)般的 AngularJS controller
- 组件内部的高度复杂循环——高度集中的逻辑分支和异步控制流

与此同时，我们希望应用是具有可维护的，可测试的，可拓展的和具有良好性能的

复杂的应用不一定拥有所有这些有价值的特征。我们也不能在完成高级功能的情况下避免这些所有的特征，但是我们可以设计应用来最大化它的有价值的特征

## 分离关注点

我们可以将*分离关注点（separation of concerns）*作为应用的分层方案。我们按照系统的关注点组织逻辑以便于每次只聚焦于单一功能。在最顶层，分离关注点是一个架构原则。在日常开发中，它都应该改被铭记于心

![](./images/model-view-presenter-with-angular/horizontal_layers.png)

我们可以同时将我们的应用横向，纵向的分片。当纵向分片时，我们按照*功能*把软件工件分组；当横向分片时，我们按照软件*层次*分组。子啊我们的应用中，我们可以将软件工件划分为这些横向层次，或者是系统关注点

| Horizontal layer | Examples                                                     |
| ---------------- | ------------------------------------------------------------ |
| Business logic   | Application-specific logic, domain logic, validation rules   |
| Persistence      | WebStorage, IndexedDB, WebSQL, HTTP, WebSocket, GraphQL, Firebase, Meteor |
| Messaging        | WebRTC, WebSocket, Push API, Server-Sent Events              |
| I/O              | Web Bluetooth, WebUSB, NFC, camera, microphone, proximity sensor, ambient light sensor |
| Presentation     | DOM manipulation, event listeners, formatting                |
| User interaction | UI behaviour, form validation                                |
| State management | Application state management, application-specific events    |

同样的规则也可以应用于我们的 Angular 组件。它们应该只关心*表现*层和*用户交互*层。结果就是我们将系统的里的动态部分解耦

当然，我们添加额外的抽象层的过程需要非常多的约束，但是最终结果的有价值的特性会弥补这一切。请记住我们只是创建了一开始就应该在那里的抽象

## MVP（Model-View-Presenter）模式

MVP 是一类实现应用界面的软件架构设计模式。我们用它使得类，函数，和难以测试的模块（软件工件）的复杂逻辑减到最小。特别是我们会避免像 Angular 组件这种界面类型的软件工件的变得复杂。

就像它衍生自的 MVC 模式一样，MVP 将领域模型（domain model）和表现（presentation）进行分离。表现层通过观察者模式（Observer Pattern）对领域的变化做出响应，这些在由 Erich Gamma, Richard Helm, Ralph Johnson, and John Vlissides (又称为 “The Gang of Four”) 编写的经典图书 “[Design Patterns: Elements of Reusable Object-Oriented Software](https://www.amazon.com/Design-Patterns-Elements-Reusable-Object-Oriented/dp/0201633612)” 中具有描述

在*观察者模式*（Observer Pattern）*中，一个*对象（subject）*维护了一个当状态改变时需要通知的*观察者（observers）*列表。这听起来熟悉吗？你已经猜到了，RxJS 就是基于观察者模式

*视图*（view）*除了在表单的数据绑定和组件组合中并不包含任何的逻辑或者行为。当用户交互发生时它把控制权委托给 presenter

presenter 会批处理状态的修改，所以当用户填写表单时最终呈现的时一个巨大的修改而不是许多零碎的修改，比如每一个表单会更新应用的状态而不是每一个字段。这使得撤销或者重做状态的改变变得容易。presenter 通过命令更新状态。多亏了 [Observer Synchronization](https://www.martinfowler.com/eaaDev/MediatedSynchronization.html) 状态的改变才得以反馈到视图上

### Angular 变种

受到原始 MVP 模式的启发以及经过一系列的变化，我们创建了适用于 Angular 平台的软件工件，它的关键界面单位就是*组件（component）*

理想情况下，一个组件只聚焦展示和用户交互。在现实中，我们制定了严格的规则来确保我们的组件只关心应用的部分状态给用户以及允许用户影响状态。

这篇文章介绍的 MVP 变种采用的是 [Encapsulated Presenter 模式](https://lostechies.com/derekgreer/2008/11/23/model-view-presenter-styles/#the-encapsulated-presenter-style)。但无论如何，我们的 presenter 不会使用它的视图。取而代之的是，我们会采用 observables 将 presenter 与 model 和视图连接起来，以便 presenter 能够独立于视图进行测试。

我们打算使用 [Supervising Controller](https://www.martinfowler.com/eaaDev/SupervisingPresenter.html) 方式当实现 MVP 模式时。我们的视图（Angular 组件）简单的依赖它们的 presenter 负责用户交互。因为我们的 presenter 被它们的视图所封装，数据和事件在某些时候都会经过组件模型。

在组件模型的帮助下，我们的 presenter 将用户的交互翻译成组件的特定事件。事件被翻译成会被发送到模型的命令。最终的翻译会被接下来会介绍的容器组件来处理

我们的 presenter 会有一些[表现层模型（Presentation Model）](https://martinfowler.com/eaaDev/PresentationModel.html)的特征，也就是说包含一些用于指示 DOM 元素是否启用的布尔属性值的表现层逻辑。一个例子是一个用于指示 DOM 元素应该被渲染成的颜色

我们的视图与 presenter 上的属性绑定，简单将它代表的状态投没有额外逻辑的展示出来。结果形成了一个简单的带有组件模型的组件模板

## 为 Angular 准备的 MVP 概念

为了要将 MVP 模式应用到 Angular 应用里，我们将要介绍 React 社区里常被推崇的概念。我们的组件——在这些文章中——将会被划分为三类

- 纯展现组件（Presentational components）
- [容器组件（Container components）](https://indepth.dev/container-components-with-angular/)
- 混合组件（Mixed components）

React 开发者已经从混合组件中提取纯展示组件和容器组件很多年了。我们在 Angular 应用中也可以使用相同的概念。额外的，我们将会介绍 presenter 的概念。

### 纯展示组件

*纯展示组件*纯粹的用于呈现和交互的视图。它们把应用的部分状态展示给用户并且允许用户影响这些状态。

因为 presenter 的存在，纯展示组件完全不理会应用其它部分的存在。它们有数据绑定接口能够描述它们处理的用户交互和它们需要的数据

为了不想对界面进行单元测试，我们需要保证纯展示组件的复杂度胡参与一个绝对的最小值。对于组件模型和组件模板都是如如此

### 容器组件

*容器嘴贱*把应用的状态暴露给纯展示组件。它们通过把组件特定的事件翻译成命令和查询给非展示组件的方式，将纯展示组件与应用的其它部分集成在一起

通常容器组件和纯展示组件的关系是1对1。容器组件的类属性与纯展示组件的输入属性相匹配，方法与展示组件的事件相对应

### 混合组件

如果一个组件不是一个容器组件或者是纯展现组件，它就是*混合组件*。给出一个现有的应用，很大可能它包含混合组件。我们称之为混合组件因为它们混合了两种系统关注点——它们包含了多个横向层的逻辑。

如果你偶遇了一个组件——额外的包含了一组用于展示的领域对象——能够直接访问设备摄像头，发送 HTTP 请求和使用 WebStorage 缓存应用状态请不要惊讶

虽然应用中的逻辑一定存在，到那时把它们组织在单一地方会让它非常难测试，难以推断，重用起来复杂以及紧耦合

### Presenters

为了得到一个简单的纯展现组件，行为逻辑和复杂的表现层逻辑被抽离到一个 presenter 中。presenter 没有界面并且通常没有或者极少的包含注入的依赖，便于它容易测试和推断。

Presenter 几乎不会感知应用的其它部分。通常一个纯展示组件只会 引用一个 presenter

## MVC 组合

![](./images/model-view-presenter-with-angular/model-view-presenter-triad.png)

这三个软件工件结合成了我们所谓的 MVC。模型（model）——由容器组件表示——代表了需要由浏览器展示给用户的应用状态

视图，由纯展现组件表示，是一个呈现应用状态并且把用户交互翻译为组件级别事件的薄的用户层，通常将控制流借转发给 presenter

presenter 是一个完全不会感知应用其它部分的类的实例

### 数据流

#### 数据在组件树的向下流动

让我们把 数据和事件是如何在 MVP 中流动的过程可视化出来

![](./images/model-view-presenter-with-angular/data-flow.gif)

在上图中，应用的状态更改已经发生在一个服务中。因为容器组件订阅了服务上的一个 observable 属性所以它被通知了

容器组件把被触发的新值转化为新的纯展示组件最容易接受的新格式。Angular 把新的值和引用赋值给纯引用的组件的接收输入属性上

纯展示组件把更新后的数据传递给 presenter ，用于重新计算需要在纯展示组件模板里需要使用的额外属性

现在数据已经完成了在组件树向下的流动，然后 Angular 把状态渲染和更新在 DOM 上，在列表里展示给用户

#### 事件在组件树种的向上流动

![](./images/model-view-presenter-with-angular/event-flow.gif)

在上图种用户点击了一个按钮。由于在模板中的事件绑定，Angular 把控制权转交展示组件模型的事件处理函数。

用户的脚本被 presenter 拦截，并且将它转化成特定的数据结构，随后通过 observable 属性的方式将它传播出去。展示组件模型观察到了这次修改，又通过向外传递的属性将这个值传播出去。

因为模板里事件绑定的缘故 Angular 通过组件特定事件告诉容器组件新的值产生了

现在事件已经完成了在组件树中的向上游动，容器组件将数据结构转化为参数传递给服务里的一个方法

接下来一个命令就会用户改变应用状态，服务通过 observable 属性触发状态的改变，之后数据又一次像上上图中那样的流动

## 一个改进的 Angular 应用

有些人会认为我们的新 UI 架构过于复杂导致了过度设计，但是在现实中我们留下的是许多简单的，模块化的软件片段。模块化的软件架构让我们变得*敏捷*。不是敏捷流程里的那种敏捷，而是对于[变化开销](http://instead%20of%20an%20increasing%20pile%20of%20technical%20debt%2C%20we%20are%20proactive%20as%20opposed%20to%20reactive%20when%20handling%20changes%20in%20customer%20requirements.%20it%20is%20very%20difficult%20to%20achieve%20this%20level%20of%20agility%20with%20a%20system%20that%20is%20tightly%20coupled%20and%20hardly%20testable%2C%20taking%20months%20to%20refactor./)而言

> 模块化的软件架构让我们变得敏捷

我们在前瞻性的处理用户需求的改变而不是累积技术债。为了避免紧耦合与难以测试，即使经过了数月的重构，也难以达到这个级别的敏捷度

### 可维护性

尽管最终的系统是由许多的动态部分组成，但是每一个部分都非常的简单并且只关注系统的单个功能点。但额外的我们的到了一个整洁的知道何去何从的系统

### 可测试性

我们把 Angular 相关的软件工件里的逻辑尽可能最少化，因为它们测试起来困难的和缓慢。当每一个软件部分只关注唯一的系统功能时，它们变得容易被推断。我们也就能非常容易的在自动测试里验证

界面测试通常缓慢而困难，在这一点上 Angular 也不例外。采用 MVC 之后，我们把展现组件里的逻辑最小化，使它们变得不值得测试。取而代之的使，我们选择跳过单元测试，而是依靠我们的开发工具，集成测试以及端到端测试来捕获像类型错误，语法错误，未初始化之类的错误

### 可拓展性

功能之间可以独立开发，即使在软件中水平层次分割的软件工件也可以独立的测试和开发。我们很清晰的区分每一部分的逻辑片段属于哪。

我们先可以独立开发层，我们可以区分技术开发和前端视觉开发。一名开发人员善于使用 RxJS 实现 behaviour ，同时另一名开发人员热爱后端集成，还有另一名开发人员喜欢完善设计与使用 CSS 和 HTML 解决可访问性

因为我们能够独立开发功能，任务可以被分配到不同的团队中。一个团队关心产品分类功能，而另一个团队负责电子商务系统里的问题和购物车功能

### 性能

恰当的关注点分离带给我们高性能，特别是在展现层中。性能瓶颈能很简单的被追踪和分离出来。

同时使用 `OnPush` 变更监测策略后，我们可以最小化 Angular 的变更监测循环对应用性能的影响