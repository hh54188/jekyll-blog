---
layout: post
title: 面试系列之三：你真的了解React吗（中）组件间的通信以及React优化
modified: 2017-07-06
tags: [javascript, react, interview]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

本系列文章旨在完善你可能忽略的React相关的知识点，通过回答以下问题：

- [React解决了什么问题](#why_need_react)
- **[如何设计一个好的组件？](#design_component)**
- **[组件的Render函数在何时被调用？](#when_render_invoked)**
    - 调用时DOM就一定会被更新吗？
- [组件的生命周期有哪些？](#react_lifecircle)
    - 当某些第三方类库想对DOM初始化，或者进行远程数据加载时，应该在哪个周期中完成？
    - 在哪些声明周期中可以修改组件的state？
- **[不同父节点的组件需要对彼此的状态进行改变时应该实现？](#component_communication)**
    - 如何设计出一个好的Flux架构
    - 如何设计出一个好的React组件
- [如何进行优化？](#component_optimize)
    - 组件中的key属性有什么用？
- [Component 与 Element 与 Instance 的区别](#component_element_diff)
- **[如果你使用过Redux与Vuex的话，聊聊他们的区别与你的心得](#flux_vs_vuex)**
    - Vue.js 的双向绑定是如何实现的？
- [Webpack如何打包输出多个文件？](#about_webpack)
    - webpack打包时如何工作的？
        - 如何解决循环引用的问题
    - 在什么情况下需要打包输出多个文件？
    - loader和plugin的差别
    - 你觉得使用过什么高级技巧吗？
- [（开放问题）React的生态你使用过哪些类库](#webpack_ecology)

本文是本系列的中篇，之前的内容可以参考之前的本系列的前一篇文章：

## 组件间的通信和架构设计问题

在上一篇的文章中，我们主要聊了如何去设计一个好的组件。然而那个视角是独立的，甚至说是狭隘的，只考虑了的单个组件自己。接下来我们将考虑更复杂的情况，例如组件间的交互等等。

其实Facebook官方关于React最佳实践已经写的非常好的了，但可惜的是这些实践和工作原理、教程都[混杂在一起](https://facebook.github.io/react/docs/hello-world.html)，并且长篇累牍，让人阅读起来非常没有重点；另一方面如果你是个新手，或许你会看完所有的这些文档，但你还没有实践的经验，你满脑子想的其实是如何开始写你的第一个Hello World应用，所以这些最佳实践你也很快就忘了。

这一小节的主要内容都来自上面所说的Facebook关于React的[官方文档](https://facebook.github.io/react/docs/hello-world.html)，还有一部分来自一篇我认为很有价值，理念和官方文档一致但又写的更好的一篇文章：[Flux in Depth. Overview and Components](http://blog.mgechev.com/2015/05/15/flux-in-depth-overview-components/)

### 组件间的通信问题

首先假设我们有一个用React构建的单页面应用，组件间的关系如下图所示：

![sample-state-change](../images/understand-react/sample-state-change.png)

我们首先假设每一个组件都在维护自己的状态（state），而不是使用传递下来的props。那么问题来了，如果右侧的组件E想改变左侧的组件B的状态，应该怎么办？

这个例子其实想表达的是一类通用的业务场景，就是组件间状态的修改，也有可能是子组件修改父组件的状态。上面的这个例子是一类极端的情况，被修改的组件与发出修改的组件不在同一个父组件中。

Facebook的官方推荐办法（曾经是）是使用事件机制（现在这个页面已经跳转到另一个解决方案了，也就是下面要叙述的）。但我和绝大多数人都不认为这是一个好的机制。你可以想象一旦应用中这样的需求增多，事件和回调函数满天飞，则情况右回到了Flux之前的原地状态：调试和维护代码都很难进行，这会是一个灾难。

第二个方案是传递接口。如果E想改变B的状态，那么B要传递给E组件一个修改状态的接口。不过因为Flux属性是从上至下传递的关系，所以接口的传递应该是如下图所示：

![sample-state-change-callbacks](../images/understand-react/sample-state-change-callbacks.png)

所以事实上我们要从祖先元素Root传递两个接口分别给E和B。当E想改变B的状态时，E调用root传给它的那个接口，而后那个接口又调用root传给B的接口……听上去这也不是什么好办法。

**Stateless Component**

在介绍正确的姿势之前，我们需要引入一个概念，Stateless Component，你可以把它翻译为“无状态组件”。什么是无状态组件呢，它和 pure function（以下我们译为“纯粹函数”）的概念很相似，根据[维基百科](https://en.wikipedia.org/wiki/Pure_function)，一个纯粹函数应该具有以下特征：

- 当传入相同的执行参数时，总是返回相同的执行结果。并且执行结果并不依赖其他的外部变量
- 函数的执行也不会引起其他的副作用，例如对象的修改或者I/O输出之类的

根据上面的描述不难推断出我们的无状态组件其实也具有相同的特征：

- 除了父组件传入的属性（props），它不应该再依赖其他的任何的全局变量; 
- 对于总是相同的传入属性，应该保证渲染结果也总是一致的。
- **最重要的是，它不应该维护自己的state**

**正确的Flux架构**

所以目前官方推荐的React组件和Flux架构的最佳实践是：

- 在开发组件和设计组件时保证组件是 Stateless Component。牢记组件的职责只有一个，那就是**渲染**数据。
- 在所有组件顶端设置一个相反的 Statefull Component，把所有的数据和状态都置于这个“充满状态的组件”中（类似于我们上一篇讲的Container Component），然后通过属性传递的方式将数据传递给孩子组件
- 充满状态的组件封装交互逻辑并且负责状态管理；而无状态的组件负责渲染数据

这样的原则有没有眼熟？这正是Vuex的架构思想，Vuex架构受Flux启发，同时也针对Flux的一些缺陷做出了改进，在下一篇文章中我会聊到Vuex与Flux的差异。

当然Stateless也不是绝对的，比如一些第三方组件，比如React-DND，就需要维护自己的状态。

**state里应该有什么**

最后提一句state里应该有什么。这个题目也是有标准答案的，在[Interactivity and Dynamic UIs](https://shripadk.github.io/react/docs/interactivity-and-dynamic-uis.html)这篇文章里。

- State里应该包含什么：组件的事件处理函数可能进行修改的，导致UI更新的数据（State should contain data that a component's event handlers may change to trigger a UI update. ）
- State里不应该有什么：
    - 计算得出的数据
    - React组件
    - 从props复制来的数据

## 如何对组件进行优化

这一题也是有Facebook官方标准答案的：[Optimizing Performance](https://facebook.github.io/react/docs/optimizing-performance.html)，扼要地摘取官方的建议由以下几点：

- **使用上线构建（Production Build）**：会移除脚本中不必要的警告和报错，减少文件体积
- **避免重绘 （Avoid Reconciliation）**：重写 shouldComponentUpdate 函数，手动控制是否应该调用 render 函数进行重绘
- **尽可能的使用 Immutable Data（ The Power Of Not Mutating Data）**：尽可能的不修改数据，而是重新赋值数据。这样的话，在检测数据对象是否发生修改方面会非常快，只需要检测对象引用即可，而不用挨个的检测对象属性的更改

最后一条建议摘自官方描述的关于Virtual DOM的工作原理[Reconciliation](https://facebook.github.io/react/docs/reconciliation.html)
- **在渲染组件的时候尽可能的添加key** ，这样Virtual DOM在对比时就会更容易发现哪里，哪里是修改元素，哪里是新插入的元素。这里也同时回答了key的作用。如果你有使用过React渲染一个列表的话，它会建议你给每一项添加上key。我个人认为key就类似于DOM中的id，不过是组件级别的，用于标记元素的唯一性。

## Component与Element与Instance的区别

一般在编码中大部分人应该不会有这三个概念的疑惑。但如果阅读一些React相关的文章，可能就免不了和这三个概念打交道。既然我也被问到过，那么也就在这里普及一下。

**Element**

Element其实就是一个纯粹的Object对象，用于描述你在屏幕上看到的DOM结点。这个对象的属性包括`type`, `props`, `ref`, `key`，并不包括DOM的方法。

通常我们在开发中使用的都是JSX语法，同时利用Babel则将JSX语法转化为原生的React语法。

JSX语法中`<div>Hello</div>`即意味着一个元素，而Babel将它转化为原生的React语法脚本：

```javascript
var helloWorld = React.createElement(
  "div",
  null,
  "Hello"
);
```

而事实上结果只是一个Javascript对象：

```javascript
var helloWorld = {
    key: null,
    props: {
        children: "Hello"
    },
    ref: null,
    type: "div"
};
```

以上这三种格式都代表element

**Component**

Component就是组件级别的类或者说是Element的模：**一个接收参数并且返回React元素的函数或者类**。通常我们使用ES6的语法定义组件：

```javascript
class CustomForm extends React.Component {
```
而以一个函数的形式语法可以是这样：

```javascript
function CustomForm ({ addFriend }) {
  return {
    type: 'form',
    props: {
      onClick: addFriend,
      children: 'Add Friend'
    }
  }
}
```
当你定义完`CustomForm`之后，在页面上使用`<CustomForm>`标签即完成了从组件到元素的使用。

**Instance**

最后当你调用`ReactDOM.render()`把一个组件渲染到一个具体的DOM元素中，返回的值即使一个实例（instance）：

```javascript
var componentInstance = ReactDOM.render(<CustomForm />, document.getElementById("root"));
```

参考文章

[https://www.site2share.com/folder/20020521](https://www.site2share.com/folder/20020521)

你可能会喜欢

- [面试系列之二：你真的了解React吗（上）如何设计组件以及重要的生命周期](https://www.v2think.com/understand-react-01)
- [面试系列之三：你真的了解React吗（中）组件间的通信以及React优化](https://www.v2think.com/understand-react-02)
- [面试系列之四：你真的了解React吗（下）Flux与Vuex的差异以及Webpack](https://www.v2think.com/understand-react-03)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置](https://www.v2think.com/create-app-user-guide)
- [深入React的生命周期(上)：出生(Mount)](https://www.v2think.com/dig-into-react-lifecircle-01)
- [深入React的生命周期(下)：更新(Update)](https://www.v2think.com/dig-into-react-lifecircle-02)
- [深入理解 Webpack 打包分块（上）](https://www.v2think.com/webpack-chunks-split-01)
- [深入理解 Webpack 打包分块（下）](https://www.v2think.com/webpack-chunks-split-02)
- [Webpack 速成](https://www.v2think.com/webpack-tutorial)
- [构建大型 Mobx 应用的几个建议](https://www.v2think.com/tips-for-building-mobx-app)
- [【译文】给构建大型 redux 应用的五个建议](https://www.v2think.com/five-tips-for-redux-large-applications)
- [React + Redux 性能优化（一）：理论篇](https://www.v2think.com/redux-performance-01-basic)
- [React + Redux 性能优化（二）工具篇： Immutablejs](https://www.v2think.com/redux-performance-02-immutablejs)
- [Flux与Redux背后的设计思想(二)：CQRS, Event Sourcing, DDD](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CQRS-ES-DDD)
- [Flux与Redux背后的设计思想(一)：Command Bus, Event Bus, Service Bus](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CB-EB-ESB)


