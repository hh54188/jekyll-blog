# React + Redux 性能优化（二）：工具篇 Immutablejs

## 什么是 Pure Component

## 性能

建议在阅读完上一篇[React + Redux 性能优化（一）：理论篇](http://qingbob.com/redux-performance-01-basic/)之后再开始本文的旅程，本文的很多概念和结论，都在上篇做了详细的讲解

上篇中我们得出的一个很重要的结论是，只要组件的状态（`props`或者`state`）发生了改变，那么组件就会执行`render`函数进行重新渲染。除非你重写`shouldComponentUpdate`周期函数通过返回`false`来阻止这件事的发生；又或者直接让组件直接继承`PureComponent`。

而继承`PureComponent`的原理也很简单，它只不过代替你实现了`shouldComponentUpdate`函数而已，并且在函数内对`props`和`state`进行“浅对比”（shallow comparision，即仅仅是比较对象的引用而不是比较对象每个属性的值），如果发现对象前后没有改变则不执行`render`函数对组件进行重新渲染

其实这样一套相似逻辑在 Redux 中也多次存在，在 redux 中也会对数据做“浅对比”

首先是在`react-redux`中也会对状态进行浅对比

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
代码中组件`App`是被 redux 封装的组件，那么`react-redux`会假设`App`是一个`Pure Component`，即对于唯一的`props`和`state`有唯一的渲染结果。
所以`react-redux`首先会对根状态（root state，上面代码中`mapStateToProps`的第一个形参`state`）创建索引，进行浅对比，如果对比成功则不对组件进行重新渲染，否则继续调用`mapStateToProps`；同时继续对`mapStateToProps`返回的`props`对象里的每一个属性的值（即上面例子中的`state.todos`值和`getVisibleTodos(state)`值，而不是返回的`props`整个对象）创建一个索引。和`shouldComponentUpdate`类似，只有当浅对比失败，即索引发生更改时才会重新对封装的组件进行渲染

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
即使`state.todos`和`getVisibleTodos(state)`同样不再发生变化，但是因为每次`mapStateToProps`返回结果`{ data: {...} }`中的`data`都创建新的字面量对象，`App`依然会再次出发渲染

其次是在 `combineReducers` 中。

我们都知道 Redux Store 鼓励我们把状态对象划分为不同的碎片（slice）或者领域（domain），并且为这些不同的领域分别编写 reducer 函数用于管理它们的状态，最后使用官方提供的`combineReducers`函数将这些领域以及它们的 reducer 函数关联起来，拼装成一个整体的`state`

举个例子

```javascript
combineReducers({ todos: myTodosReducer, counter: myCounterReducer })
```

可见程序的状态是由`{ todos, counter }`两个领域模型组成，同时`myTodosReducer`与`myCounterReducer`分别为各自领域的 reducer 函数

`combineReducers`会遍历每一对领域（key是领域名称、value是领域 reducer 函数），对于每一次遍历：
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