# React + Redux 性能优化（二）：工具篇 Immutablejs

建议在阅读完上一篇[React + Redux 性能优化（一）：理论篇](http://qingbob.com/redux-performance-01-basic/)之后再开始本文的旅程，本文的很多概念和结论，都在上篇做了详细的讲解

## 关于 Pure

无论是在 react 还是 redux 中，pure 都是非常重要的概念。理解什么是 pure 有助于我们理解我们为什么需要 Immutablejs

首先我们要介绍什么是Pure function （纯函数）, [来自维基百科：](https://en.wikipedia.org/wiki/Pure_function)：

> 在程序设计中，若一个函数符合以下要求，则它可能被认为是纯函数：
>  * 此函数在相同的输入值时，需产生相同的输出。函数的输出和输入值以外的其他隐藏信息或状态无关，也和由I/O设备产生的外部输出无关。
>  * 该函数不能有语义上可观察的函数副作用，诸如“触发事件”，使输出设备输出，或更改输出值以外物件的内容等。

简单来说纯函数的两个特征：1) 对于相同的输入总有相同的输出；2) 函数不依赖外部变量，也不会对外部产生影响（这种影响称之为“副作用（side effects）”）

### Reducer

redux 中**[规定](https://redux.js.org/basics/reducers) reducer 就是纯函数**。它接收前一个状态和 action 作为参数，返回下一个状态:

```
(previousState, action) => newState
```
保证 reducer 的“纯粹（pure）”非常重要，你永远不能在 reducer 中做以下三件事：
* 修改参数
* 执行任何具有副作用的操作，比如调用 API
* 调用任何不纯粹的函数，比如`Math.random()`或者`Date.now()`

所以你会看到在 reducer 里返回状态是通过`Object.assign({}, state)`实现的（注意不要写成`Object.assign(state)`，这样就修改了原状态），这样旧避免修改了原来的传入的`state`。而至于调用 API 等异步或者具有“副作用”的操作，则可以通过`redux-thunk`或者`redux-saga`实现。

### Pure Component

在上一篇中我们谈到过 Pure Component，准确说那是狭义上的`React.PureComponent`。广义上的 Pure Compnoent 指的是 Stateless Component，也就是无状态组件，也被称为 Dumb Component、 Presentational Component。从代码上它的特征是 1) 不维护自己的状态，2) 只有`render`函数:

```javascript
const HelloUser = ({userName}) => {
  return <div>{`Hello ${userName}`}</div>
}
```

显而易见的是，这种形式的“纯组件”和“纯函数”有异曲同工之妙，即对于相同的属性传入，组件总是输出唯一的结果。

这样形式的组件也丧失了一部分的能力，例如不再拥有生命周期函数。

## 性能

上篇中我们得出的一个很重要的结论是，只要组件的状态（`props`或者`state`）发生了改变，那么组件就会执行`render`函数进行重新渲染。除非你重写`shouldComponentUpdate`周期函数通过返回`false`来阻止这件事的发生；又或者直接让组件直接继承`PureComponent`。

而继承`PureComponent`的原理也很简单，它只不过代替你实现了`shouldComponentUpdate`函数：在函数内对现在和过去的`props`/`state`进行“浅对比”（shallow comparision，即仅仅是比较对象的引用而不是比较对象每个属性的值），如果发现对象前后没有改变则不执行`render`函数对组件进行重新渲染

其实这样一套相似逻辑在 Redux 中也多次存在，在 redux 中也会对数据进行“浅对比”

首先是在`react-redux`中

我们通常会使用`react-redux`中的`connect`函数将程序状态注入进组件中，例如:

```javascript
import {conenct} from 'react-redux'

function mapStateToProps(state) {
  return {
    todos: state.todos,
    visibleTodos: getVisibleTodos(state),
  }
}

export default connect(mapStateToProps, mapDispatchToProps)(App)
```
代码中组件`App`是被 `react-redux` 封装的组件，`react-redux`会假设`App`是一个`Pure Component`，即对于唯一的`props`和`state`有唯一的渲染结果。
所以`react-redux`首先会对根状态（即上述代码中`mapStateToProps`的第一个形参`state`）创建索引，进行浅对比，如果对比结果一致则不对组件进行重新渲染，否则继续调用`mapStateToProps`函数；同时继续对`mapStateToProps`返回的`props`对象里的每一个属性的值（即上述代码中的`state.todos`值和`getVisibleTodos(state)`值，而不是返回的`props`整个对象）创建索引。和`shouldComponentUpdate`类似，只有当浅对比失败，即索引发生更改时才会重新对封装的组件进行渲染

就上面的代码例子来说，只要`state.todos`和`getVisibleTodos(state)`的值不发生更改，那么`App`组件就永远不会再一次进行渲染。但是请注意下面的陷阱模式：

```javascript
function mapStateToProps(state) {
  return {
    data: {
      todos: state.todos,
      visibleTodos: getVisibleTodos(state),
    }
  }
}
```
即使`state.todos`和`getVisibleTodos(state)`同样不再发生变化，但是因为每次`mapStateToProps`返回结果`{ data: {...} }`中的`data`都创建新的（字面量）对象，导致浅对比总是失败，`App`依然会再次渲染

其次是在 `combineReducers` 中。

我们都知道 Redux Store 鼓励我们把状态对象划分为不同的碎片（slice）或者领域（domain，也可以理解为业务），并且为这些不同的领域分别编写 reducer 函数用于管理它们的状态，最后使用官方提供的`combineReducers`函数将这些领域以及它们的 reducer 函数关联起来，拼装成一个整体的`state`

举个例子

```javascript
combineReducers({ todos: myTodosReducer, counter: myCounterReducer })
```

上述代码中，程序的状态是由`{ todos, counter }`两个领域模型组成，同时`myTodosReducer`与`myCounterReducer`分别为各自领域的 reducer 函数

`combineReducers`会遍历每一“对”领域（key是领域名称、value是领域 reducer 函数），对于每一次遍历：
* 它会创建一个对当前碎片数据的引用
* 调用 reducer 函数计算碎片数据新的状态，并且返回
* 为 reducer 函数返回的新的碎片数据创建新的引用，将新的引用和当前数据引用进行浅对比，如果对比失败了（同时意味着两次引用不一致，意味着 reducer 返回的是一个新的对象），那么将标识位`hasChanged`设置为`true`

在经过一轮（这里的一轮指的是把每一个领域都遍历了一遍）遍历之后，`combineReducer`就得到了一个新的状态对象，通过`hasChanged`标识位我们就能判断出整体状态是否发生了更改，如果为`true`，新的状态就会被返回给下游，如果是`false`，旧的当前状态就会被返回给下游。这里的下游指的是`react-redux`以及更下游的界面组件。

根据上面的知识点，`react-redux`会对根状态进行浅对比，如果引用发生了改变，才重新渲染组件。**所以当状态需要发生更改时，务必让相应的 reducer 函数始终返回新的对象！修改原有对象的属性值然后返回不会触发组件的重新渲染！**

所以我们常看到的 reducer 函数写法是最终通过 `Object.assign` 复制原状态对象并且返回一个新的对象：

```javascript
function myCounterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case "add":
      return Object.assign({}, { count: state.count + 1 });
    default:
      return state;
  }
}
```
错误的做法是仅仅修改原对象：

```javascript
function myCounterReducer(state = { count: 0 }, action) {
  switch (action.type) {
    case "add":
      state.count++
      return state
    default:
      return state;
  }
}
```
有趣的事情是如果你此时在`state.count++`之后打印 `state` 的结果，你会发现`state.count`确实在每次`add`之后都有自增，但是组件却始终不会渲染出来








## 参考文章

* [Immutable Data](https://redux.js.org/faq/immutable-data)
* [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)
* [Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)
* [Pure function](https://en.wikipedia.org/wiki/Pure_function)
* [Reducers](https://redux.js.org/basics/reducers)