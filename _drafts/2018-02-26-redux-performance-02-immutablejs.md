# React + Redux 性能优化（二）：工具篇 Immutablejs

建议在阅读完上一篇[React + Redux 性能优化（一）：理论篇](http://qingbob.com/redux-performance-01-basic/)之后再开始本文的旅程，本文的很多概念和结论，都在上篇做了详细的讲解

上篇中我们得出的一个很重要的结论是，只要组件的状态（`props`或者`state`）发生了改变，那么组件就会执行`render`函数进行重新渲染。除非你重写`shouldComponentUpdate`周期函数通过返回`false`来阻止这件事的发生；又或者直接让组件直接继承`PureComponent`。

而继承`PureComponent`的原理也很简单，它只不过代替你实现了`shouldComponentUpdate`函数而已，并且在函数内对`props`和`state`进行“浅对比”（shallow comparision，即仅仅是比较对象的引用而不是比较对象每个属性的值），如果发现对象前后没有改变则不执行`render`函数对组件进行重新渲染

其实这样一套相似逻辑在 Redux 中也多次存在，在 redux 中也会对数据做“浅对比”

首先是在 `combineReducers` 中。

我们都知道 Redux Store 鼓励我们把状态对象划分为不同的碎片（slice）或者领域（domain），并且为这些不同的领域分别编写 reducer 函数用于管理它们的状态，最后使用官方提供的`combineReducers`函数将这些领域以及它们的 reducer 函数关联起来，拼装成一个整体的`state`

举个例子

```javascript
combineReducers({ todos: myTodosReducer, counter: myCounterReducer })
```

可见程序的状态是由`{ todos, counter }`两个领域模型组成，同时`myTodosReducer`与`myCounterReducer`分别为各自领域的 reducer 函数

`combineReducers`会遍历每一对领域（key是名称、value是reducer），对于每一次遍历：
* 它会创建一个对当前碎片数据的引用
* 调用 reducer 函数计算碎片数据新的状态，并且返回
* 为 reducer 函数返回的新的碎片数据创建新的引用，将新的引用和当前数据引用进行浅对比，如果对比失败了，也就是两次引用不一致，也就是 reducer 返回的是一个新的对象，那么将标识位`hasChanged`设置为`true`

在经过一轮（这里的一轮指的是把每一个领域都遍历了一遍）遍历之后，`combineReducer`就得到了一个新的状态对象，通过`hasChanged`标识位我们就能判断出整体状态是否发生了更改，如果为`true`，新的状态就会被返回给下游，如果是`false`，旧的当前状态就会被返回给下游

在每一



## 参考文章

* [Immutable Data](https://redux.js.org/faq/immutable-data)
* [React.PureComponent](https://reactjs.org/docs/react-api.html#reactpurecomponent)
* [Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)