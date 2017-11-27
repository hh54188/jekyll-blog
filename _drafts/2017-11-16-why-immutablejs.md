为什么需要 Immutable 数据：如果从 reducer 里返回的 state 总是不同的引用的话，那
么每次在 shouldComponentUpdate 里的比较总是会返回 false，那么总是会引起组件的刷
新

* 为什么需要 Immutable 数据
  ：[Using shouldComponentUpdate()](https://developmentarc.gitbooks.io/react-indepth/content/life_cycle/update/using_should_component_update.html)
* 关于在 Redux 中使用 Immutable.js 的基本入门
  ：[Using Immutable.JS with Redux](https://redux.js.org/docs/recipes/UsingImmutableJS.html#what-are-some-opinionated-best-practices-for-using-immutablejs-with-redux)
* Immutable.js 的**快的秘密**已经一些最佳实践 :
  [Immutable.js, persistent data structures and structural sharing](https://medium.com/@dtinth/immutable-js-persistent-data-structures-and-structural-sharing-6d163fbd73d2)
  * \*Redux 中的一些不好的模式：
    [React.js pure render performance anti-pattern](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f)
  * 更详细的 structrual sharing 技术细节：
    [A deep dive into Clojure's data structures - EuroClojure 2015](https://www.slideshare.net/mohitthatte/a-deep-dive-into-clojures-data-structures-euroclojure-2015)
  * 如果实现一个反向查找表：
    [Implementing a reverse lookup table using Immutable.js](https://medium.com/@dtinth/implementing-a-reverse-lookup-table-using-immutable-js-26662dec988d)
* 关于 Immutable.js 的一些讨论
  ：[Reddit](https://www.reddit.com/r/reactjs/comments/5h7pqz/persistent_data_structures_and_structural_sharing/?st=ja1xs0ee&sh=a47adae2)

## 要使用到的库

* Immutable.js
* Reselect
* normalizers
* react-addons-perf
* react-pure-render/shallowEqual

## 次要的

* [ImmutableJS 系列教程](http://untangled.io/category/libraries/immutablejs/)
* redux + immutablejs 的实际教程
  * [React, Redux and Immutable.js: Ingredients for Efficient Web Applications](https://www.toptal.com/react/react-redux-and-immutablejs)
  * [Practical Guide to Using ImmutableJS with Redux and React](https://medium.com/@fastphrase/practical-guide-to-using-immutablejs-with-redux-and-react-c5fd9c99c6b2)
* [redux + immutable + normalizar + reselect](http://fullstackdeveloper.info/redux-state-with-immutable-js-normalizr-and-reselect/)
* 关于 React 的性能文章集合
  ：[React/Redux Performance and Optimization](https://github.com/markerikson/react-redux-links/blob/master/react-performance.md#immutable-data)
* `mapStateToProps`里必须要返回`object`，应该怎么办 :
  [Using `connect()` with `Immutable.Map` state object](https://github.com/reactjs/react-redux/issues/60)

## anti-pattern

* [Redux Patterns and Anti-Patterns](https://tech.affirm.com/redux-patterns-and-anti-patterns-7d80ef3d53bc)
