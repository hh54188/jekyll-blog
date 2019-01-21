# 仪表盘场景前端优化经验谈

如果你只关注本文末尾代码级别的解决方案，或许其中并没有太多令人惊喜的东西。然而如果你能完整的阅读下来，相信关于性能问题的解决思路以及指标的取舍和测量还是能带来一些有价值的参考

## 背景

相信绝大部分公司的中台系统中都存在仪表盘页面，以 [Ant Design Pro](https://pro.ant.design/index-cn) 提供的中台方案分析页面为例，通常仪表盘页面的形式如下。仪表盘由不同的图表卡片组成，并且在创建或者编辑时允许作者添加、删除卡片，拖拽卡片的位置和调整卡片的大小等等：

![](./images/dashboard-optimize/dashboard_sample.png)

图表卡片支持多种类型的图表展现，除了传统的折线图与柱状图外，还支持桑吉图、漏斗图甚至表格等等，以满足产品和运营同学以不同的角度洞察指标的变化的需求。但是无论卡片的展现形式有多么的千变万化，背后都需要后端精确的数据的予以支撑。

考虑到卡片是仪表盘的最小单元，彼此之间相互独立，并且可以被动态的添加、预览。所以在产品的设计阶段，我们将仪表盘信息分开存在在两个实体中：「仪表盘」和「卡片」。仪表盘只存储它拥有的卡片的基本信息，卡片的ID以及位置和尺寸。而卡片的详细信息以及查询工作则交给卡片独立获取，这样更加灵活。前端与后端同学约定接口时也是以卡片为中心，我们为卡片准备了两类接口, 为了便于描述，我们将接口简化和语义化：

1. `/meta`: 用于请求卡片的元信息，例如指标、维度、图表类型等
2. `/query`: 根据卡片的元信息查询图表数据，数据返回之后再进行渲染

之所以要将查询接口与元信息接口拆分开，是因为查询 payload 中除了元信息以外还要整合诸如全局的日期，筛选条件等额外信息

基于上述的设计，前端在实现阶段也非常简单，我们采取了一种「自治」（或者说是「懒惰」）的思想：仪表盘组件只负责将卡片组件实例以指定尺寸摆放在指定的位置，而至于这张卡片的加载、查询、渲染全权由卡片自己负责。基于这个思路，我们甚至不需要复杂的状态管理框架（如 Redux 或者 Mobx），仅仅依靠视图层的 React 就能够实现。卡片组件的实现借助了 [react-refetch](https://github.com/heroku/react-refetch) 类库，它以高阶组件的形式为数据加载提供便利，伪代码如下：

```javascript
// CardComponent.js:
import {connect} from 'react-refetch'

@connect(() => {
  return {
    metaInfoFetch {
      url: '/meta',
      andThen = () => ({
        query: '/query'
      })
    }
  }
})
class Card {
  render() {
    const queryResult = this.props.query.value
    return <Chart data={queryResult} />
  }
}

// DashboardComponent.js:
class Dashboard {
  return (
    <div>
      {cards.map(({id, position, size}) => 
        <Card id={id} position={position} size={size} />
      )}
    </div>
  )
}
```

**但是没想到前端这种「自治」的解决方案却给产品带来了灾难**

产品上线之后，我们得到用户反馈说某些仪表盘页面打开总时是会进入了「卡死」状态，即页面无法滚动，无法点击，甚至浏览器也无响应。即使没有「卡死」，一段时间内页面的交互反馈也会出现滞后的情况。我们整理出这些有问题的仪表盘之后，发现这些仪表盘都具有一些相似的特征：1) 卡片数量多 2) 卡片需要渲染的数据量大

因为允许用户随意的任意的配置卡片，所以某些仪表盘的卡片数量可以达到 20 甚至 30 张以上，而平均每张卡片需要发出两个网络请求，所以在打开仪表盘的瞬间同时有 40 个以上的请求发出，这显然是超出浏览器的处理能力的，同时浏览器也不允许同一个域下有这么的多并行请求，于是大部分的请求处于队列等待中；随着多个卡片查询结果的返回，这些卡片进入图表渲染阶段，而如果卡片是分钟级别的折线图的话，考虑到按照 n 个维度拆分的情况，图表需要处理 24 × 60 × n 的数据需要处理，这对性能的损耗也是极大的。于是你看到在同一时间内，满负荷的请求发出，众多的卡片在渲染，再加上其他需要执行的脚本，CPU 自然就进入了满负荷的状态，因为「单线程」的缘故，浏览器也就无暇响应用户的输入，以及渲染页面

![](./images/dashboard-optimize/old_dashboard_cpu_overload.png)

如上图所示，如果我们借助 Chrome 浏览器自带的 Performance 工具观察整个加载的过程，从标注1和标注2可以看出CPU始终处于满载的状态，并且这其中的主要是在执行脚本，并且几乎没有给渲染分配时间，从标注3可以看出，在这段时间内浏览器渲染能力接近 0fps，需要上百秒来渲染一帧

到这里为止事故的现状，原因我都做了简单的介绍，接下来就要着手解决这个问题。

## 思路

我个人的总结，程序的性能问题好比是人体的疾病，对于治病最有效的两个手段就是经验和工具。经验不仅仅是指你个人曾经预见过同样的问题，还包括行业内前人的总结归纳。绝大部分问题通过经验都能迎刃而解，通过页面的所属功能以及异常行为就能判断出问题可能出在哪里以及应该如何解决，就像医生在秋冬季节看到病人发烧流鼻涕咽喉痛，就能判断出是流感引起以及得出治疗方案了。而对于更复杂的难以通过表象判断的问题，这个时候就需要借助于工具，观察问题发生的时机，（代码）位置，影响面有多广，又或者你只是想通过工具验证你的猜想是否正确而已。

在上一小节的描述的问题中，我无法通过表象判断出问题所在，但是借助工具分析，我们得知是因为高强度的工作造成了 CPU 的满载。

在面对 long task （执行时间超过 50ms以上）时，屡试不爽的解决方案是分片（chunk），也就是把长时间连续执行的任务拆分成短暂的可间隔执行的任务。拆分的好处是能够使得浏览器得以有空隙响应其他的请求，比如用户的输入以及页面的绘制。

我最早接触这个方法是在 Nicholas C. Zakas（「JavaScript高级程序设计」原版作者） 十年前发表的一篇[博客](https://humanwhocodes.com/blog/2009/01/13/speed-up-your-javascript-part-1/)，在处理一个占用时间过长的循环时，他编写了一个很简易的分片函数：

```javascript
function chunk(array, process, context){
    setTimeout(function(){
        var item = array.shift();
        process.call(context, item);

        if (array.length > 0){
            setTimeout(arguments.callee, 100);
        }
    }, 100);
}
```
虽然现在 `callee` 已经 deprecated 了，`setTimeout`也可以使用`requestAnimationFrame`代替。但是它背后的思考方式是没有发生变化的

我们面临的困难并不是一个真实的 long task，而是无数的碎片任务蜂拥而至造成了 long task 的效果。我们的解决思路依然没有变：要设法给浏览器制造喘息的机会。在这个目标之下，我选择的方案是放弃卡片自治的数据加载和渲染方式，而是采用队列的机制，对需要执行的所有任务做到严格的进出控制。这样能够保证从开始就不会给浏览器大压力

## 产出

性能优化在日常工作中其实处于很尴尬的位置。例如你花费三天为页面或者 App 开发了一个功能，上线之后大家是有目共睹的。然而如果你花费三天时间告诉大家我进行了一次代码优化，大家会对你的产出有所怀疑。所以在优化之前我们最好确定计划提升的指标以便量化我们的产出

然而在这个场景里，我们应该选取哪些指标？我个人习惯将指标划分为「业务指标」和「工程指标」：「业务指标」衡量的是产品的运营状态，例如转化、留存等等；而「技术指标」面向的是技术人员，例如 QPS、并发数等等。在中台的业务场景下，我们并不存在营收或者说是商业化方向的指标诉求，目前看来只有一条，那就是让产品变得可用：即页面能够及时响应用户的输入，及时反馈页面的更新。所以大部分指标都会从工程指标中选取。在前端领域中可以想到，工程指标从「基础」到「复杂」排序分别有：

- onload
- DOMContentLoaded
- SpeedIndex
- First Paint
- First Contentful Paint
- Time to Interactive

但假设我们选取了以上的指标，我们如何能够准确测量指标？以及测量的结果是否能够正确的反馈工作的成果？业务指标和工程指标并非是二分法的互斥关系，绝大部分情况下正相关，即工程优化的越好，业务指标也会得到提升；但其实也存在相互独立。我常举的例子是，你仔细想，如果 onload 或者 DOMContentLoaded 的时间延长，那么 bounce rate 一定会升高吗？不一定的，只要我能够把 above the fold 内容加载优化的足够快，在 onload 或者 DOMContentLoaded 不变甚至变糟的情况下，bounce rate 其实是会降低的。

在这里我们并不打算继续下去。在代码开发完成之后，我们会借助浏览器接口或者工具来 review 这些指标的变化，然而我们是否能准确衡量这些指标，以及上述的猜想是否正确，请拭目以待，

## 实施

回到我们的解决方案中，最后我决定使用一个队列机制严格的控制仪表盘的，其实也是卡片的每一步操作: 1) 请求 meta 信息; 2) 查询报表数据; 3) 渲染卡片

首先我们需要一个队列，考虑到请求数据和渲染卡片分别是异步和同步操作，准确来说我们需要一个异步和同步通吃的队列机制。实现的方法有很多种，在这里我借助于 Promise 实现，因为 1) Promise 为异步而生; 2) Promise 也可以兼容同步操作; 3) Promise 支持顺序执行

我们将这个队列类命名为`PromiseDispatcher`，并且提供一个`feed`方法用于塞入需要执行的函数（无需区分异步还是同步），比如:

```javascript
const promiseDispatcher = new PromiseDispatcher()
promiseDispatcher.feed(
  requestMetaJob,
  requestDataJob,
  renderChart
)
```
那么 `requestDataJob` 一定会保证在 `requestMetaJob` 执行完毕之后再执行，`renderChart` 也保证在 `requestDataJob` 执行完毕之后再执行。

注意 dispatcher 并不具有保留执行函数返回值的功能，比如

```javascript
promiseDispatcher.feed(
  requestMetaJob,
  requestDataJob,
  renderChart
).then((requestMetaResult, requestDataResult, renderResult) => {

})
```
我们不支持以上的使用方法，并不是说实现不了，而是从职责上考虑队列不应该承担这样的职责，队列应该只负责分发，保证成员执行顺序的正确性，而不用关心成员的执行结果。如果你还不明白其中的道理，可以参考 dispatcher 角色在 Flux 架构中的职责

因为篇幅有限，我们这里只列举 PromiseDispatcher 的部分关键代码。首先是顺序执行机制，这里借用数组的 reduce 方法实现：

```javascript
return tasks.reduce((prevPromise, currentTask()) => {
    return prevPromise.then(chainResults =>
        currentTask().then(currentResult =>
            [ ...chainResults, currentResult ]
        )
    );
}, Promise.resolve([]))
```
然而我们还要兼容同步函数的代码，所以需要对`currentTask`返回结果是否是`Promise`类型做判断:

```javascript
// https://stackoverflow.com/questions/27746304/how-do-i-tell-if-an-object-is-a-promise#answer-27746324
function isPromiseObj(obj) {
  return obj && obj.then && _.isFunction(obj.then);
}
return tasks.reduce((prevPromise, currentTask()) => {
    return prevPromise.then(chainResults => {
      let curPromise = currentTask()
      curPromise = !isPromiseObj(curPromise) ? Promise.resolve(curPromise) : curPromise
      curPromise.then(currentResult => [ ...chainResults, currentResult ])
    });
}, Promise.resolve([]))
```

然而需要考虑更复杂的情况是，有时候仅仅是单个依次执行任务又过于节约了，所以我们要允许多个任务“并发”执行

