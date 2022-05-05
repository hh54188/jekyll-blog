---
layout: post
title: 深入React的生命周期(上)：出生(Mount)
modified: 2017-11-05
tags: [javascript, React]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 前言

本文是对开源图书[React In-depth: An exploration of UI development](https://www.gitbook.com/book/developmentarc/react-indepth/details)的归纳和增强。同时也融入了自己在开发中的一些心得。

你或许会问，阅读完这篇文章之后，对工作中开发React相关的项目有帮助吗？实话实说帮助不会太大。这篇文章不会教你使用一项新技术，不会帮助你提高编程技巧，而是完善你的React知识体系，例如区分某些概念，明白一些最佳实践是怎么来的等等。如果硬是要从功利的角度来考虑这些知识带来的价值，那么会是对你的面试非常有帮助，这篇文章里知识点在面试时常常会被问到，为什么我知道，因为我吃过它们的亏。

React组件的生命周期划分为出生（mount），更新（update）和死亡（unmount），然而我们怎么知道组件进入到了哪个阶段？只能通过React组件暴露给我们的钩子（hook）函数来知晓。什么是钩子函数，就是在特定阶段执行的函数，比如`constructor`只会在组件出生阶段被调用一次，这就算是一个“钩子”。反过来说，当某个钩子函数被调用时，也就意味着它进入了某个生命阶段，所以你可以在钩子函数里添加一些代码逻辑在用于在特定的阶段执行。当然这不是绝对的，比如`render`函数既会在出生阶段执行，也会在更新阶段执行。顺便多说一句，“钩子”在编程中也算是一类设计模式，比如github的[Webhooks](https://developer.github.com/webhooks/)。顾名思义它也是钩子，你能够通过Webhook订阅github上的事件，当事件发生时，github就会像你的服务发送POST请求。利用这个特性，你可以监听master分支有没有新的合并事件发生，如果你的服务收到了该事件的消息，那么你就可以例子执行部署工作。

我们按照阶段的时间顺序对每一个钩子函数进行讲解。

## 出生

- `constructor`
- `getDefaultProps()` (React.createClass) or` MyComponent.defaultProps` (ES6 class)
- `getInitialState()` (React.createClass) or `this.state = ...` (ES6 constructor)
- `componentWillMount()`
- `render()`
- `componentDidMount()`

首先我们要引入一个概念：组件（Component）。组件非常好理解，就是可以复用的模板。例如通过按钮组件（模板）我们可以实例化出多个相似的按钮出来。这和代码中类（Class）的概念是相同的。并且在ES6代码中定义组件时也是通过类来实现的：
```javascript
import React from 'react';

class MyButton extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <button>My Button</button>
    )
  }
}
```
也可以通过ES2015的语法接口`React.createClass`来定义组件：
```javascript
const MyButton = React.createClass({
  render: function() {
    return (
      <button>My Button</button>      
    );
  }
});
```
如果你的babel配置文件`.babelrc`中`presets`指定了`es2015`，那么在编译之后的文件中，你会发现`class MyButton extends React.Component`语句编译之后的结果就是`React.createClass`。

注意到当我们在使用`class`定义组件时，继承（`extends`）了`React.Component`类。但实际上这并不是必须的。比如你完全可以写成纯函数的形式：
```javascript
const MyButton = () => {
  return <h1>My Button</h1>
}
```
这就是无状态（stateless）组件，顾名思义它是没有自己独立状态的，这个概念被用于React的设计模式：High Order Component和Container Component中。具体可以参考我的另一篇文章[面试系列之三：你真的了解React吗（中）组件间的通信以及React优化](https://zhuanlan.zhihu.com/p/27828866)。

它的局限也很明显，因为没有继承`React.Component`的缘故，你无法获得各种生命周期函数，也无法访问状态（`state`），但是仍然能够访问传入的属性（`props`）,它们是作为函数的参数传入的。

定义组件时并不会触发任何的生命周期函数，组件自己也并不会存在生命周期这一说，真正的生命周期开始于组件被渲染至页面中。

让我们看一段最简单的代码：
```javascript
import React from 'react';
import ReactDOM from 'react-dom';

class MyComponent extends React.Component {
  render() {
    return <div>Hello World!</div>;
  }
};

ReactDOM.render(<MyComponent />, document.getElementById('mount-point'));
```
在这段代码中，`MyComponnet`组件通过`ReactDOM.render`函数被渲染至页面中。如果你在`MyComponent`组件的各个生命周期函数中添加日志的话，会看到日志依次在控制台输出。

为了说明一些问题，我们尝试对代码做一些修改：
```javascript
import MyButton from './Button';
class MyComponent extends React.Component {
  render() {
    const button = <MyButton />
    return <div>Hello World!</div>;
  }
};
```
在组件的`render`函数中，我们使用到了另一个组件`MyButton`，但是它并没有出现在最终返回的DOM结构中。问题来了，当`MyComponnet`组件渲染至页面上时，`Mybutton`组件的生命周期函数会开始调用吗？`<MyButton />`究竟代表了什么？

我们先回答第二个问题。`<MyButton />`看上去确实有些奇怪，但是别忘了它是JSX语法。如果你去看babel编译之后的代码就会发现，其实它把`<MyButton />`转化为函数调用：`React.createElement(MyButton, null)`。也就是说`<XXX />`语法，实际上返回的是一个XXX类型的React元素（Element）。React元素说白了就是一个纯粹的object对象，基本由`key`（id）, `props`（属性）, `ref`, `type`（元素类型）四个属性组成（`children`属性包含在`props`中）。为什么要用“纯粹”这个形容词，是因为虽然它和组件有关，但是它并不包含组件的方法，此时此刻，它仅仅是一个包含若干属性的对象。如果你觉得这一切看上去都无比熟悉的话，那么你猜对了，元素代表的其实是虚拟DOM（Virtual DOM）上的节点，是对你在页面上看到的每一个DOM节点的描述。

那么我们可以回答第一个问题了，仅仅是生成一个React元素是不会触发生命周期函数调用的。

当我们把React元素传递给`ReactDOM.render`方法，并且告诉它具体在页面上渲染元素的位置之后，它会给我们返回组件的实例（Instance）。在JS语法中，我们通过`new`关键字初始化一个类的实例，而在React中，我们通过`ReactDOM.render`方法来初始化一个组件的实例。但一般情况下我们不会用到这个实例，不过你也可以保留它的引用赋值给一个变量，当测试组件的时候可以派上用场


### Default Porps & Default State

如果被问起`constructor`之后的下一个生命周期函数是什么，绝大部分人会回答`componentWillMount`。准确来说应该是`getDefaultProps`和`getInitialState`。

而为什么大部分人对这两个函数陌生，是因为这两个函数只是在ES2015语法中创建组件时暴露出来，在ES6语法中我们通过两个赋值语句实现了同样的效果。

比如添加默认属性的`getDefaultProps`函数在ES6中是通过给组件类添加静态字段`defaultProps`实现的：
```javascript
class MyComponent extends React.Component() {
  //...
}
MyComponent.defaultProps = { age: 'unknown' }
```

在实际计算属性的过程中，将传入属性与默认属性进行合并成为最终使用的属性，用伪代码写的意思就是
```javascript
this.props = Object.assign(defaultProps, passedProps);
```
注意知识点要来了，看下面这个组件:
```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return <div>{this.props.name}</div>
  }
}
App.defaultProps = { name: 'default' };
```

我给这个组件设置了一个默认属性`name`，值为`default`。那么在
1. `<App name={null} />`
2. `<App name={undefined} />`
这两种情况下，`this.props.name`值会是什么？也就是最终输出会是什么？

正确答案是如果给`name`传入的值是`null`，那么最终页面上的输出是空，也就是`null`会生效；如果传入的是`undefined`，那么React认为这个值是`undefined`货真价实的未定义，则会使用默认值，最终页面上的输出是`default`

而获取默认状态的函数`getInitialState`在ES6中是通过给`this.state`赋值实现的
```javascript
class Person extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  //...
}
```

### `componentWillMount()`

`componentWillMount`函数在第一次`render`之前被调用，并且只会被调用一次。当组件进入到这个生命周期中时，所有的`state`和`props`已经配置完毕，我们可以通过`this.props`和`this.state`访问它们，也可以通过`setState`重新设置状态。总之推荐在这个生命周期函数里进行状态初始化的处理，为下一步`render`做准备

### `render()`

当一切配置都就绪之后，就能够正式开始渲染组件了。`render`函数和其他的钩子函数不同，它会同时在出生和更新阶段被调用。在出生阶段被调用一次，但是在更新阶段会被调用多次。

无论是编写哪个阶段的`render`函数，请牢记一点：保证它的“纯粹”（pure）。怎样才算纯粹？最基本的一点是不要尝试在render里改变组件的状态。因为通过`setState`引发的状态改变会导致再一次调用render函数进行渲染，而又继续改变状态又继续渲染，导致无限循环下去。如果你这么做了你会在开发模式下收到警告：

> Warning: Cannot update during an existing state transition (such as within `render` or another component's constructor). Render methods should be a pure function of props and state; constructor side-effects are an anti-pattern, but can be moved to `componentWillMount`.

另一个需要注意的地方是，你也不应该在`render`中通过`ReactDOM.findDOMNode`方法访问原生的DOM元素（原生相对于虚拟DOM而言）。因为这么做存在两个风险：
1. 此时虚拟元素还没有被渲染到页面上，所以你访问的元素并不存在
2. 因为当前的`render`即将执行完毕返回新的DOM结构，你访问到的可能是旧的数据。

并且如果你真的这么做了，那么你会得到警告：

> Warning: App is accessing findDOMNode inside its render(). render() should be a pure function of props and state. It should never access something that requires stale data from the previous render, such as refs. Move this logic to componentDidMount and componentDidUpdate instead.

### `componentDidMount()`

当这个函数被调用时，就意味着可以访问组件的原生DOM了。如果你有经验的话，此时不仅仅能够访问当前组件的DOM，还能够访问当前组件孩子组件的原生DOM元素。

你可能会觉得所有这一切应当。

在之前讲解每个周期函数时，都只考虑单个组件的情况。但是当组件包含孩子组件时，孩子组件的钩子函数的调用顺序就需要留意了。

比如有下面这样的树状结构的组件

![react element tree](./images/react-element-tree.png)

在出生阶段时`componentWillMount`和`render`的调用顺序是

```
A -> A.0 -> A.0.0 -> A.0.1 -> A.1 -> A.2.
```
这很容易理解，因为当你想渲染父组件时，务必也要立即开始渲染子组件。所以子组件的生命周期开始于父组件之后。

而`componentDidMount`的调用顺序是

```
 A.2 -> A.1 -> A.0.1 -> A.0.0 -> A.0 -> A
```

`componentDidMount`的调用顺序正好是`render`的反向。这其实也很好理解。如果父组件想要渲染完毕，那么首先它的子组件需要提前渲染完毕，也所以子组件的`componentDidMount`在父组件之前调用。

正因为我们能在这个函数中访问原生DOM，所以在这个函数中通常会做一些第三方类库初始化的工具，包括异步加载数据。比如说对`c3.js`的初始化
```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import c3 from 'c3';

export default class Chart extends React.Component {

  componentDidMount() {
    this.chart = c3.generate({
      bindto: ReactDOM.findDOMNode(this.refs.chart),
      data: {
        columns: [
          ['data1', 30, 200, 100, 400, 150, 250],
          ['data2', 50, 20, 10, 40, 15, 25]
        ]
      }
    });
  }

  render() {
    return (
      <div ref="chart"></div>
    );
  }
}
```
因为能够访问原生DOM的缘故，你可能会在`componentDidMount`函数中重新对元素的样式进行计算，调整然后生效。因此立即需要对DOM进行重新渲染，此时会使用到`forceUpdate`方法

## 参考文章集合

[https://www.site2share.com/folder/20020525](https://www.site2share.com/folder/20020525)

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