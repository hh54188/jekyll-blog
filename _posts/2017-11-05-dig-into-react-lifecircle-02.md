---
layout: post
title: 深入React的生命周期(下)：更新(Update)
modified: 2017-11-05
tags: [javascript, React]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 更新阶段

- `componentWillReceiveProps()`
- `shouldComponentUpdate()`
- `componentWillUpdate()`
- `render()`
- `componentDidUpdate()`

更新阶段会在三种情况下触发：

- 更改`props`：一个组件并不能主动更改它拥有的`props`属性，它的`props`属性是由它的父组件传递给它的。强制对`props`进行重新赋值会导致程序报错。

- 更改`state`：`state`的更改是通过`setState`接口实现的。同时设计`state`是需要技巧的，哪些状态可以放在里面，哪些不可以；什么样的组件可以有`state`，哪些不可以有；这些都需要遵循一定原则的。这个话题有机会可以单独拎出来说

- 调用`forceUpdate`方法：这个我们在上一阶段已经提到了，强制组件进行更新。

### `setState`是异步的

组件的更新原因很大一部分是因为调用`setState`接口更新`state`所致，我们常常以同步的方式调用`setState`，但实际上`setState`方法是异步的。比如下面的这段代码：
```javascript
onClick() {
  this.setState({
    count: 1,
  });
  console.log(this.state.count)
}
```
在一个组件的点击事件处理函数中，我们更新了`state`中的`count`，然后立即尝试去读取最新的`count`。事实是你读取的结果不是`1`，二应该是之前的值。

更致命的错误是类似这样在同一个块级中连续调用`setState`的代码
```javascript
this.setState({ ...this.state, foo: 42 });
this.setState({ ...this.state, isBar: true });
```
在这种情况下，第一次设置的`foo`值会被第二次的设置覆盖而还原

### `componentWillReceiveProps(nextProps)`

当传递给组件的`props`发生改变时，组件的`componentWillReceiveProps`即会被触发调用，方法传递的参数的是发更更改的之后的`props`值（通常我们命名为`nextProps`）。在这个方法里，你可以通过`this.props`访问当前的属性值，可以通过`nextProps`访问即将更新的属性值，或者将它们进行对比，或者将它们进行计算，最终确定你需要更新的状态（`state`）并最终调用`setState`方法对状态进行更新。在这个钩子函数中调用`setState`方法并不会触发再一次渲染。

非常有意思的是，虽然`props`的更改会引起`componentWillReceiveProps`的调用；但`componentWillReceiveProps`的调用并不意味着`props`真的发生了变化。这可不是我说的，Facebook官方花了一整篇文章说这件事：[(A => B) !=> (B => A)](https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)。比如看下面这个组件：

```javascript
class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      number: 1,
    }
    this.onClick = this.onClick.bind(this);
  }
  onClick() {
    this.setState({
      number: 1,
    })
  }
  render() {
    return (
      <MyButton onClick={this.onClick} data-number={this.state.number} />
    );
  }
}
```
每一次点击事件都会重新使用`setState`接口对`state`进行更新，但每次更新的值都是相同的，即`number:1`。并且把当前组件的状态以属性的形式传递给`<MyButton />`。问题来了，那么当我每次点击按钮时，按钮`MyButton`的`componentWillReceiveProps`都会被调用吗？

会，即使每次更新的值都是一样的。

之所以出现这样的情况原因其实非常简单，因为React并不知道传入的属性是否发生了更改。而为什么React不尝试去做一个是否相等的判断呢？

因为办不到，新传入的属性和旧属性可能引用的是同一块内存区域（引用类型），所以单纯的用`===`判断是否相等并不准确。可行的解决办法之一就是对数据进行深度拷贝然后进行比较，但是这对大型数据结构来说性能太差，还能会碰上循环引用的问题。

所以React将这个变化通过钩子函数暴露出来，千万不要以为当`componentWillReceiveProps`被调用就意味着`props`发生了更改，如果需要在变化时做一些事情，务必要手动的进行比较。

### `shouldComponentUpdate()`

`shouldComponentUpdate`很重要，它可以决定是否继续当前的生命周期。默认情况该函数返回`true`即继续当前的生命周期；也可以返回`false`终止当前的生命周期，阻止进一步的`render`与接下来的步骤。

我们上面刚刚说过，React并不会对`props`进行深度比较，这对`state`也同样适用。所以即使`props`与`state`并未发生了更改，`shouldComponentUpdate`也会被再次调用，包括接下来的步骤`componentWillUpdate`、`render`、`componentDidUpdate`也都会再次运行一次。这很明显会给性能造成不小的伤害。

传递给`shouldComponentUpdate`的参数包括即将改变的`props`和`state`，形参的名称是`nextProps`和`nextState`，在这个函数里你同时又能通过`this`关键字访问到当前的`state`和`props`，所以你在这里你是“全知”的，可以完全按照你自己的业务逻辑判断是否`state`与`props`是否发生了更改，并且决定是否要继续接下来的步骤。`shouldComponentUpdate`也就通常我们在优化React性能时的第一步。这一步的优化不仅仅是优化组件自身的流程，同时也能节省去子组件的重新渲染的代价 。

当然如果你对判断`props`是否发生改变的检测逻辑要求比较简单的话，比如只是浅度（shallow）的判断（即判断对象的引用是否发生了更改）对象是否发生了更改，那么可以利用`PureRenderMixin`：
```javascript
import PureRenderMixin from 'react-addons-pure-render-mixin'; // ES6
const createReactClass = require('create-react-class');

createReactClass({
  mixins: [PureRenderMixin],

  render: function() {
    return <div className={this.props.className}>foo</div>;
  }
});
```
`minins`是React支持的一种允许多个组件共用代码的一种机制。`PureRenderMixin`插件的工作非常简单，它为你重写了`shouldComponentUpdate`函数，并对对象进行了浅度对比，具体代码可以从[这里](https://github.com/facebook/react/blob/15-stable/src/addons/shallowCompare.js)和[这里](https://github.com/moroshko/shallow-equal)找到。

在ES6中你也可以通过直接继承`React.PureComponent`而不是`React.Component`来实现这个功能。用React官方的原话说就是
> `React.PureComponent` is exactly like `React.Component`, but implements `shouldComponentUpdate()` with a shallow prop and state comparison.

**Pure**

我们再次强调，`PureComponent`为你实现的只是对引用是否发生了更改的判断，甚至可以说它只是简单的用`===`进行的判断，所以这也是我们称之为**pure**的原因。为了具体说明问题，我们举一个实际的例子
```javascript
/* MyButton.js: */
import React from 'react';

class MyButton extends React.PureComponent {
  constructor(props) {
    super(props);
  }
  render() {
    console.log('render');
    return <button onClick={this.props.onClick}>My Button</button>
  }
}
export default MyButton;

/* App.js: */
import React from 'react';
import MyButton from './Button.js';

class App extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      arr: [1],
    }
    this.onClick = this.onClick.bind(this);
  }
  onClick() {
    this.setState({
      arr: [...this.state.arr, 2],
    });
  }
  render() {
    return (
      <MyButton onClick={this.onClick} data-arr={this.state.arr} />
    );
  }
}

export default App;
```
在上面的这个例子中，每一次点击都会修改`state`中的`arr`变量，`arr`变量的引用和值都发生了更改。重点是`MyButton`组件继承的是`React.PureComponent`。那么每一次点击时，`MyButton`中的log信息都会被打印出来，即每次都会重新出发`render`

如果我们把`onClick`方法做一些修改：
```javascript
onClick() {
  const arr = this.state.arr;
  arr.push(2);
  this.setState({
    arr: arr,
  })
}
```
这个方法同样使得`arr`变量发生了变化，但是仅仅是值而不是引用，此时当再一次点击按钮（`MyButton`）时，`MyButton`都不会再次进行渲染了。也就是说`PureComponent`提前为我们进行了shallow comparison.

使用这种只修改引用，不修改数据内容的immutable data也常常作为优化React的一个手段之一。[immutable.js](http://facebook.github.io/immutable-js/)就能为我们实现这个需求，每一次修改数据时你得到的其实是新的数据引用，而不会修改到原有的数据。同时Redux中的reducer想达到的效果其实也相似，`reducer`的重点是它的纯洁性（pure），在执行时不会造成副作用，即避免对传入数据引用的修改，同时也方便比较出组件状态的更新。

### `componentWillUpdate()`

`componentWillUpdate`方法和`componentWillMount`方法很相似，都是在即将发生渲染前触发，在这里你能够拿到`nextProps`和`nextState`，同时也能访问到当前即将过期的`props`和`state`。如果有需要的话你可以把它们暂存起来便于以后使用。

与`componentWillMount`不同的是，在这个方法中你不可以使用`setState`，否则会立即触发另一轮的渲染并且又再一次调用`componentWillUpdate`，陷入无限循环中。

### `componentDidUpdate()`

和Mount阶段类似，当组件进入`componentDidUpdate`阶段时意味着最新的原生DOM已经渲染完成并且可以通过`refs`进行访问。该函数会传入两个参数，分别是`prevProps`和`prevState`，顾名思义是之前的状态。你仍然可以通过`this`关键字访问当前的状态，因为可以访问原生DOM的关系，在这里也适用于做一些第三方需要操纵类库的操作。

update阶段各个钩子函数的调用顺序也与mount阶段相似，尤其是`componentDidUpdate`，子组件的该钩子函数优先于父组件调用

因为可以访问DOM的缘故，我们有可能需要在这个钩子函数里获取实际的元素样式，并且写入`state`中，比如你的代码可能会长这样：
```javascript
componentDidUpdate(prevProps, prevState) {
// BAD: DO NOT DO THIS!!!
  let height = ReactDOM.findDOMNode(this).offsetHeight;
  this.setState({ internalHeight: height });
}
```
如果默认情况下你的`shouldComponentUpdate()`函数总是返回`true`的话，那么这样在`componentDidUpdate`里更新`state`的代码又会把我们带入无限`render`的循环中。如果你必须要这么做，那么至少应该把上一次的结果缓存起来，有条件的更新`state`:
```javascript
componentDidUpdate(prevProps, prevState) {
  // One possible fix...
  let height = ReactDOM.findDOMNode(this).offsetHeight;
  if (this.state.height !== height ) {
    this.setState({ internalHeight: height });
  }
}
```

## 死亡阶段

### `componentWillUnmount()`

当组件需要从DOM中移除时，即会触发这个钩子函数。这里没有太多需要注意的地方，在这个函数中通常会做一些“清洁”相关的工作
1. 将已经发送的网络请求都取消掉
2. 移除组件上DOM的Event Listener

## 总结

最后再次强调，本文是开源图书[React In-depth: An exploration of UI development](https://www.gitbook.com/book/developmentarc/react-indepth/details)的归纳。基本上想了解生命周期看这一本书就够了，看完也无敌了。希望这篇中文简约版也会对你有帮助。

## 参考

- [do not extend React.Component](https://stackoverflow.com/questions/36296658/do-not-extend-react-component)
- [React Elements vs React Components vs Component Backing Instances](https://medium.com/@fay_jai/react-elements-vs-react-components-vs-component-backing-instances-14d42729f62)
- [React.createClass versus extends React.Component](https://toddmotto.com/react-create-class-versus-component/)
- [(A => B) !=> (B => A)](https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)
- [Beware: React setState is asynchronous!](https://medium.com/@wereHamster/beware-react-setstate-is-asynchronous-ce87ef1a9cf3)