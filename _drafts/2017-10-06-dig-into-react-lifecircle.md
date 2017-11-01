# 深入React的生命周期

## 前言

本文是对开源图书[React In-depth: An exploration of UI development](https://www.gitbook.com/book/developmentarc/react-indepth/details)的归纳、提炼还有增强。同时也融入了自己在开发中的一些心得。

你或许会问，阅读完这篇文章之后，对工作中开发React相关的项目有帮助吗？实话实说帮助不会太大。这篇文章不会教你使用一项新技术，不会帮助你提高编程技巧，而是完善你的React知识体系，例如区分某些概念，明白一些最佳实践是怎么来的等等。如果硬是要从功利的角度来考虑这些知识带来的价值，那么会是对你的面试非常有帮助，这篇文章里知识点在面试时常常会被问到，为什么我知道，因为我吃过它们的亏。

React组件是存在生命周期的，从出生（mount）到更新（update）到死亡（unmount），然而我们怎么知道组件进入到了哪个阶段？只能通过React组件暴露给我们的钩子（hook）函数来知晓。什么是钩子函数，就是在特定阶段执行的函数，比如`constructor`只会在组件出生阶段被调用一次，这就算是一个“钩子”。反过来说，当某个钩子函数被调用时，也就意味着它进入了某个生命阶段，所以你可以在钩子函数里添加一些代码逻辑在用于在特定的阶段执行。当然这不是绝对的，比如`render`函数既会在出生阶段执行，也会在更新阶段执行。顺便多说一句，“钩子”在编程中也算是一类设计模式，比如github的[Webhooks](https://developer.github.com/webhooks/)。顾名思义它也是钩子，你能够通过Webhook订阅github上的事件，当事件发生时，github就会像你的服务发送POST请求。利用这个特性，你可以监听master分支有没有新的合并事件发生，如果你的服务收到了该事件的消息，那么你就可以例子执行部署工作。

React组件的生命周期分为三个阶段，按照时间顺序分别是出生（初始化/Mounting），成长（更新/Growth/Update）以及死亡（Unmount）。我们按照阶段的时间顺序对每一个钩子函数进行讲解。

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
如果你的babel配置文件`.babelrc`中`presets`使用了`es2015`，那么在编译之后的文件中，你会发现`class MyButton extends React.Component`语句转化之后就是使用`React.createClass`进行的替换。

注意到当我们在使用`class`定义组件时，继承（`extends`）了`React.Component`类。但实际上这并不是必须的。比如你完全可以写成纯函数的形式：
```javascript
const MyButton = () => {
  return <h1>My Button</h1>
}
```
这就是无状态（stateless）组件，顾名思义它是没有自己独立状态的，这React的设计模式中，无论是High Order Component或者是Container Component中都是重要的组成部分。具体可以参考我的另一篇文章[面试系列之三：你真的了解React吗（中）组件间的通信以及React优化](https://zhuanlan.zhihu.com/p/27828866)。

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
在这段代码中，`MyComponnet`组件通过`ReactDOM.render`函数被渲染至页面中。如果你在`MyComponent`组件的生命周期函数中添加日志的话，会看到日志依次在控制台输出。

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

当我们把React元素传递给`ReactDOM.render`方法，并且告诉它具体在页面上渲染元素的位置之后，它会给我们返回组件的实例（Instance）。在JS语法中，我们通过`new`关键字初始化一个类的实例，而在React中，我们通过`ReactDOM.render`方法来初始化一个组件的实例。但一般情况下我们不会用到这个实例，不过你也可以保留它，当测试组件的时候可以派上用场


### Default Porps & Default State

如果被问起`constructor`之后的下一个生命周期函数是什么，绝大部分人会回答`componentWillMount`。准确来说应该是`getDefaultProps`和`getInitialState`。

而为什么大部分人对这两个函数陌生，是因为这两个函数只是在ES2015语法中创建组件时暴露出来，在ES6语法中我们通过两个赋值语句实现了同样的效果。

比如添加默认组件属性的`getDefaultProps`函数在ES6中是通过给组件类添加静态字段`defaultProps`实现的：
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

正确答案是如果给`name`传入的值是`null`，那么最终页面上的输出是空，也就是`null`会生效；如果传入的是`undefined`，那么React认为这个值是undefined（当然……），则会使用默认值，最终页面上的输出是`default`

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

`componentWillMount`函数在第一次`render`之前被调用，并且只会被调用一次。当组件进入到这个生命周期中时，所有的`state`和`props`已经配置完毕，我们可以通过`this.props`和`this.state`访问它们，也可以通过`setState`重新设置状态。总之推荐在这个生命周期函数里进行配置（状态）处理，为下一步`render`做准备

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

在出生阶段`componentWillMount`和`render`的调用顺序是

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
因为能够访问原生DOM的缘故，你可能会在`componentDidMount`函数中重新对元素的样式进行计算，调整然后生效。因此立即需要对DOM进行重新渲染。

## 更新阶段

- `componentWillReceiveProps()`
- `shouldComponentUpdate()`
- `componentWillUpdate()`
- `render()`
- `componentDidUpdate()`

更新阶段会在三种情况下触发：

- 更改`props`：一个组件并不能主动更改它拥有的`props`属性，它的`props`属性是由它的父组件传递给它的。强制对`props`进行重新赋值会导致程序报错。

- 更改`state`：`state`的更改是通过`setState`接口实现的。同时设计`state`是需要技巧的，哪些状态可以放在里面，哪些不可以；什么样的组件可以有`state`，哪些不可以有；这些都需要遵循一定原则的。

- 调用`forceUpdate`方法：这个我们在上一阶段已经提到了，强制组件进行更新。

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
每一次点击事件都会重新使用`setState`接口对`state`进行更新，但每次更新的值都是相同的，即`number:1`。并且把当前组件的状态以属性的形式传递给`<MyButton />`。问题来了，那么当我每次点击按钮时，按钮的`componentWillReceiveProps`都会被调用吗？

会，即使每次更新的值都是一样的。

之所以出现这样的情况原因其实非常简单，因为React并不知道传入的属性是否发生了更改。而为什么React不尝试去做一个是否相等的判断呢？

因为办不到，新传入的属性和旧属性可能引用的是同一块内存区域（引用类型），所以单纯的用`===`判断是否相等并不准确。可行的解决办法之一就是对数据进行深度拷贝然后进行比较，但是这对大型数据结构来说性能太差，还能会碰上循环引用的问题。

所以React将这个变化通过钩子函数暴露出来，千万不要以为当`componentWillReceiveProps`被调用就意味着`props`发生了更改，如果需要在变化时做一些事情，务必要手动的进行比较。

### `shouldComponentUpdate()`

`shouldComponentUpdate`很重要，它可以决定是否继续当前的生命周期。默认情况该函数返回`true`即继续当前的生命周期；也可以返回`false`终止当前的生命周期，阻止进一步的`render`与接下来的步骤。

我们上面刚刚说过，React并不会对`props`进行深度比较，这对`state`也同样适用。所以即使`props`与`state`并未发生了更改，`shouldComponentUpdate`也会被再次调用，包括接下来的步骤`componentWillUpdate`、`render`、`componentDidUpdate`也都会再次运行一次。这很明显会给性能造成不小的伤害。

传递给`shouldComponentUpdate`的参数包括即将改变的`props`和`state`，形参的名称是`nextProps`和`nextState`，在这个函数里你同时又能通过`this`关键字访问到当前的`state`和`props`，所以你在这里你是“全知”的，可以完全按照你自己的业务逻辑判断是否`state`与`props`是否发生了更改，并且决定是否要继续接下来的步骤。`shouldComponentUpdate`也就通常我们在优化React性能时的第一步。这一步的优化不仅仅是优化组件自身的流程，同时也能节省去子组件的重新渲染的代价 。

当然如果你对判断`props`是否发生改变的检测逻辑要求比较简单的话，比如只是浅度（shallow）的判断（即判断对象的引用是否发生了更改）对象是否发生了更改，那么可以利用`PureRenderMixin`插件即可：
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
`PureRenderMixin`插件的工作非常简单，它为你重写了`shouldComponentUpdate`函数，并对对象进行了浅度对比，具体代码可以从[这里](https://github.com/facebook/react/blob/15-stable/src/addons/shallowCompare.js)和[这里](https://github.com/moroshko/shallow-equal)找到。

在ES6中你也可以通过直接继承`React.PureComponent`而不是`React.Component`实现。用React官方的原话说就是
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

使用这种只修改引用，不修改数据内容的immutable data也常常作为优化React的一个手段之一。[immutable.js](http://facebook.github.io/immutable-js/)就能为我们实现这个需求，每一次修改数据时你得到的其实是新的数据引用，而不会修改到原有的数据。同时Redux中的reducer想达到的效果其实也相似，`reducer`的重点是它的纯洁性（pure），比如不会造成副作用，也就要避免对传入数据引用的修改，同时也方便比较出组件状态的更新。

### `componentWillUpdate()`

`componentWillUpdate`方法和`componentWillMount`方法很相似，都是在即将发生渲染前触发，在这里你能够拿到`nextProps`和`nextState`，同时也能访问到当前即将过期的`props`和`state`。如果有需要的话你可以把它们暂存起来便于以后使用。

与`componentWillMount`不同的是，在这个方法中你不可以使用`setState`，否则会立即触发另一轮的渲染并且又再一次调用`componentWillUpdate`，陷入无限循环中。






















## 参考

- [do not extend React.Component](https://stackoverflow.com/questions/36296658/do-not-extend-react-component)
- [React Elements vs React Components vs Component Backing Instances](https://medium.com/@fay_jai/react-elements-vs-react-components-vs-component-backing-instances-14d42729f62)
- [React.createClass versus extends React.Component](https://toddmotto.com/react-create-class-versus-component/)
- [(A => B) !=> (B => A)](https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html)

this.setState是异步的
当props发生更改时，componentWillReceiveProps会被调用；但是并不意味着componentWillReceiveProps被调用了而props发生了更改。也就是在一些情况下，componentWillReceiveProps被调用了，但是props并没有发生更改
https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html
React.PureComponent

render返回的是什么？是element吧？
render pass？

需要实验的部分：key以及内存回收
