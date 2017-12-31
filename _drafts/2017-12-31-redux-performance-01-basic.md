# React + Redux 性能优化（一）：理论篇

本文的叙事线索与代码示例均来自[High Performance Redux](http://somebody32.github.io/high-performance-redux/)，特此表示感谢。之所以感谢是因为最近一直想系统的整理在 React + Redux 技术栈下的性能优化方案，但苦于找不到切入点。在查阅资料的过程中，这份 Presentation 给了我很大的启发，它的很多观点一针见血，也与我的想法不谋而合。于是这篇文章也是参照它的讲解线索来依次展开我想表达的知识点

或许你已经听说过很多的第三方优化方案，比如`immutable.js`，`reselect`，`react-virtualized`等等，有关工具的故事下一篇再详谈。首先我们需要了解的是为什么会出现性能问题，以及解决性能问题的思路是什么。当你了解完这一切之后，你会发现其实很多性能问题不需要用第三方类库解决，只需要在编写代码中稍加注意，或者稍稍调整数据结构就能有很大的改观。

## 性能不是 patch，是 feature

我和其他的程序员朋友交流过，每个人都对性能有自己的理解。其中有一种观点认为，在程序开发的初期不需要关心性能，当程序规模变大并且出现瓶颈之后再来做性能的优化。我不同意这种观点。性能不应该是后来居上的补丁，而应该是程序天生的一部分。从项目的第一天起，我们就应该考虑做一个[10x project](http://somebody32.github.io/4)：能够运行 10k 个任务并且拥有 10 年寿命

退一步说即使你在项目的后期发现了瓶颈问题，公司层面不一定会给你足够的排期解决这个问题，毕竟业务项目依然是优先的（还是要看这个性能问题有多“痛”）；再退一步说，即使允许你展优化工作，经过长时间迭代开发后的项目已经和当初相比面目全非了：模块数量庞大，代码耦合严重，尤其是 Redux 项目牵一发而动全身，再想对代码进行优化的话会非常困难。从这个意义上来说，从一开始就将性能考虑进产品中去也是一种 future-proof 的体现，提高代码的可维护性

从另一个角度看，代码性能也是个人编程技艺的体现。当然不仅仅是代码性能，还包括代码质量，模块划分等等。这些都是衡量水平的一部分。这些又是另一个话题了，总而言之，一位优秀的程序员的代码性能是有保障的。

闲话到这里，我们正式开始吧

## 存在性能问题的列表

前端框架喜欢把实现 Todo List 作为给新手看的教程。我们这里也拿一个 List 举例。假设你需要实现一个列表，用户点击有高亮效果，比如点击之后高亮，再次点击取消高亮，如此循环。仅此而已。特别的地方在于这个列表有 10k 的行，是的，你没看错 10k 行（上面不是说好我们要做 10x project 吗:p）

首先我们看一看基本款代码，由`App`组件和`Item`组件构成，关键代码如下：

```javascript
function itemsReducer(state = initial_state, action) {
  switch (action.type) {
    case "MARK":
      return state.map(
        item =>
          action.id === item.id ? { ...item, marked: !item.marked } : item
      );
    default:
      return state;
  }
}

class App extends Component {
  render() {
    const { items, markItem } = this.props;
    return (
      <div>
        {items.map(({ id, marked }) => (
          <Item key={id} id={id} marked={marked} onClick={markItem} />
        ))}
      </div>
    );
  }
}

function mapStateToProps(state) {
  return state;
}

const markItem = id => ({ type: "MARK", id });

export default connect(mapStateToProps, { markItem })(App);
```

这段关键的代码体现了几个关键的事实：

1. 列表每一项（`item`）的数据结构是`{ id, marked }`
2. 列表（`items`）的数据结构是数组类型：`[{id1, marked}, {id2, marked}, {id3, marked}]`
3. `App`渲染列表是通过遍历（`map`）列表数组`items`实现的
4. 当用户点击某一项时，把被点击项的`id`传递给`item`的`reducer`
5. reducer 通过遍历 `items`，挨个对比`id`的方式找到需要被标记的项
6. 重新标记完之后将新的数组返回
7. 新的数组返回给`App`，`App`再次进行渲染

如果你没法将以上代码片段和我叙述的事实拼凑在一起，可以在 github 上找到[完整代码](https://github.com/somebody32/high-performance-redux/tree/gh-pages/assets/naive_list)浏览或者运行。

对于这样的一个需求，相信绝大多数人的代码都是这么写的。

但是上述代码没有告诉你的事实时，这是一段性能很差的代码，当你尝试点击某个选项时，选项的高亮会延迟至少半秒秒钟，用户会感觉到列表响应变慢了。

注意这样的延迟值并不是绝对：

1. 注意这样的现象只有在列表项数目众多的情况下出现，比如说 10k。
2. 在开发环境（`ENV === 'development'`）下运行的代码会比在生产环境（`ENV === 'production'`）下运行较慢
3. 我个人 PC 的 CPU 配置是 1700x，不同电脑配置的延迟会有所不同

## 诊断

那么问题出在哪里？我们通过 Chrome 开发者工具一探究竟：

* 本地启动项目， 打开 Chrome 浏览器，**在地址栏以访问项目地址加上`react_perf`后缀的方式访问项目页面**，比如我的项目地址是: http://localhost:3000/ 的话，实际请访问 http://localhost:8080/?react_perf 。加上`react_perf`后缀的用意是启用 React 中的性能埋点，这些埋点用于统计 React 中某些操作的耗时，使用`User Timing API`实现
* 打开 Chrome 开发者工具，切换到 performance 面板
  [devtools-performance](./images/redux-performance-01/devtools-performance.png)
* 点击 performance 面板左上角的“录制”按钮，开始录制性能信息
  [devtools-performance](./images/redux-performance-01/devtools-performance-record.png)
* 点击列表中的任意一项
* 等被点击项进入高亮状态时，点击“stop”按钮停止录制性能信息
  [devtools-performance](./images/redux-performance-01/devtools-performance-stop.png)
* 接下来你就能看到点击阶段的性能大盘信息：
  [performance](./images/redux-performance-01/performance-overview.png)

我们把目光聚焦到 CPU 活动最剧烈的那段时间内，

[performance-main](./images/redux-performance-01/performance-main.png)

从图标中可以看出，这部分的时间（712ms）消耗基本是由脚本引起的，准确来说是由点击事件执行的脚本引起的，并且从函数的调用栈以及从时间排序中可以看出，时间基本上花费在`updateComponent`函数中。这已经能猜出一二，如果你还不确定这个函数究竟干了什么，不如展开`User Timing`一栏看看更通俗的解释

[performance-main](./images/redux-performance-01/performance-user-timing.png)

**原来时间都花费`App`组件的更新，其实也就是每一个`Item`的更新上**

其实还有很多的 React 相关工具

### User Timing 主线

## 参考资料

* [Debugging React performance with React 16 and Chrome Devtools.](https://building.calibreapp.com/debugging-react-performance-with-react-16-and-chrome-devtools-c90698a522ad)
* [High Performance React: 3 New Tools to Speed Up Your Apps](https://medium.freecodecamp.org/make-react-fast-again-tools-and-techniques-for-speeding-up-your-react-app-7ad39d3c1b82)
