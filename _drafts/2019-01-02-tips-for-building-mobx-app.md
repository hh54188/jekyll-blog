# 构建大型 Mobx 应用的几个建议

Mobx 与 Redux 相似，都是适用于应用状态管理的出色工具。它同样遵循单向数据流，同样能与 React 搭档配合。与 Redux 不同的是，它的学习成本更低，框架体系更加完善（比如它自带异步操作的解决方案，而 Redux 只提供了中间件体系，必须借助第三方类库实现）。如果说 Redux 只是继承了 Flux 的衣钵的话，那么 Mobx 则是基于 Flux 的再一次进化

Mobx 是一枚优秀的工具，但是它的学习资料却非常有限，即使是官网的文档也还是拘泥于 API 的讲解而缺乏实战的例子（个人在这里推荐 [MobX Quick Start Guide](https://www.amazon.cn/gp/product/B07FZ2DRF6) 这本薄薄的册子是比官网更好的入门）。

从去年下半年开始工作项目中的状态管理框架逐渐从 Redux 替换为 Mobx，这些项目中同样需要处理大量的数据和复杂的交互，于是逐渐在这方面积攒了一些经验。这些经验大多是我个人踩过的坑，这些坑倒不是技术上的难点痛点，而是如何让程序更易于维护的技巧和模式。希望对那些正在使用 Mobx 做开发的同学有所帮助。这其中会使用 Redux 的开发模式进行对比，更易于理解。在这篇文章里，我也会尽可能少的聚焦代码实现层面的东西。

## 将一切状态存储在 Mobx 中

我们回忆一下在完整的 Redux 应用中，单个页面中会存在哪些状态（独立拆分的 reducer）：

- 业务状态：比如购物车已添加的商品
- UI状态：比如提交按钮是否可以点击，是否需要展示错误提示
- 应用级别状态：应用是处于离线还是上线状态，用户是否登录

React 组件的状态属性来源也同时有好几个渠道：

- 父组件自身的状态
- store 的注入
- store 经过 selector 计算之后注入的 

那么当我们尝试追溯或者 debug 某个属性的来源时，面临的是 (3 × 3 = )9 种的可能，这会带来非常大的困恼

所以我个人的第一条件建议是：**尽可能的将状态都存储在同一处 Mobx 中**。注意这里有几个关键词：

- 尽可能的：用万能的二分法进行划分，我们总是能把状态划分为“共享的”（比如上面说的“应用级别状态”）和“非共享的”（比如对应于某个特定业务）。鉴于共享状态需要被全应用所引用，所以我们无法把它分配到某个特定业务的状态下，需要独立出来。
- 同一处：在设计状态的时候应该遵循两条基本的原则：1) 每个页面应该有自己的状态; 2) 每个被复用的组件应该有自己的状态；3) 根据以上两点可以推断，React 组件应该是 stateless 的

页面级别的状态像是容器，将上面描述的三类的状态都集中在一起。比如在新用户注册的页面中，用户填写的用户名、密码和邮箱信息属于业务状态，密码长度不够给出的错误提示属于应用状态，注册按钮是否可以被点击属于UI状态。这样做的原因不仅仅是因为便于状态的追溯和管理，还因为它们天然就是相关的。例如当注册信息填写不完整时，提交按钮自然是不可被点击的状态，而密码填写格式不正确时，错误信息应该自然出现。也就是说大部分的 UI 状态和应用状态是基于业务状态“计算”或者说“衍生”出来的，在 Mobx 中，我们可以借助`autorun`、`reaction`和`computed`很容易的实现这些需要被计算的状态，并且及时的修改它们。

### Schema 很重要

不知道你会不会有这样的一个疑问，如果把不同性质的状态集中在一起，当你需要在提交页面时如何正确的挑选出业务信息并且组装它们，并且保证提交的业务信息是正确的？这里我推荐使用 schema 来确保来提前定义业务信息的数据格式。不仅仅在 Mobx 应用中，使用 schema 应该是在构建所有大型前端应用贯穿的最佳实践。随着应用变得庞大，程序的不同组成之间，内部和外部之间的通信也变得越来越复杂。对每一类需要使用的消息或者对象提前定义 schema，有利于确保通信的正确性，防止传入不存在的字段，或者传入字段的类型不正确；同时也具有自解释的文档的作用，有利于今后的维护。例如使用 [hapijs/joi](https://github.com/hapijs/joi) 类库定义用户信息的 schema :

```javascript
const Joi = require('joi');

const schema = Joi.object().keys({
    username: Joi.string().alphanum().min(3).max(30).required(),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
    access_token: [Joi.string(), Joi.number()],
    birthyear: Joi.number().integer().min(1900).max(2013),
    email: Joi.string().email({ minDomainAtoms: 2 })
})
```
使类似的 schema 类库非常多，最简单的使用 ImmutableJS 里的 Record 类型也能起到约束的作用，这里就不展开了

另一方面，一个组件的理想状态应该是对于使用者来说是他是无感知的，这里的“无感知”有几层意思，最基础的，它的 API 设计应该和原生控件保持一致，感受不到特殊性；额外的他也不关心你的内部实现，你的内部状态管理与他无关。所以为了保证应用架构的一致性，我还是建议把组件内的所有状态交由 Mobx 管理，这么做同样能够利用到上面所说的各种优势。

最后把状态都交由 Mobx 管理的结果就是 React 组件都是无状态的了，这部分不用再解释了。

值得一提的是，在没有 Mobx 框架的情况下，在 Redux 应用里，我还是倾向于将状态都集中在组件的 state 中进行管理，但美中不足的是，state 机制没有明确的给出基于状态的衍生机制。然而为什么不放在 reducer 中进行管理呢，这就要聊到我给出的第二条建议了

## 赋予状态生命周期

在开发 Redux 的应用中，不知道你有没有考虑过这样一个问题，什么样的状态应该放在组件的 state 中？什么样的状态应该放在 store 中？ 这个问题其实是有明确答案的，引用[官方文档里的话](https://redux.js.org/faq/organizing-state)回答这个问题：

> Some common rules of thumb for determining what kind of data should be put into Redux:
> - Do other parts of the application care about this data?
> - Do you need to be able to create further derived data based on this original data?
> - Is the same data being used to drive multiple components?
> - Is there value to you in being able to restore this state to a given point in time (ie, time travel debugging)?
> - Do you want to cache the data (ie, use what's in state if it's already there instead of re-requesting it)?
> - Do you want to keep this data consistent while hot-reloading UI components (which may lose their internal state when swapped)?

简单来说就是，如果这份数据需要被共享，那么就把它放在 store 中。这也同时暗示了另外一件事：放在 store 中的数据是没有生命周期的，又或者可以说，它们的生命周期等同于整个应用的生命周期。

但页面和组件都是有生命周期的。举个例子，你需要在某个后台页面中持续的录入商品信息然后点击提交，然后继续录入。虽然是同一路由，同一组件，同一页面，但每一次的录入你面对的都是新的页面实例，这是页面生命周期的开始，直到你填写信息并且提交成功，这是页面生命周期的结束，也意味着这个实例被销毁。组件更是要面临这个问题，它甚至可能在同一个页面上被多次使用，每一次被使用组件和状态都需要是全新的实例

但是无论是在 Mobx 中还是 Redux 中都不提供这样的机制。或许你常常使用`mobx-react`的`inject`函数，它的作用是将通过`<Provider />`绑定在上下文的 store 注入到组件中，和 `redux-react` 中的 `connect` 无异，与声明周期无关

这里我们使用高阶组件（High Order Component）解决这个问题。

高阶组件解决问题的原理非常简单，它把 store 实例和组件实例连接在一起（封装在另一个组件里），然后让他们同生同灭

假设我们的高阶组件函数名称为`withMobx`，我们将`withMobx`的用法设计如下，与 Redux 的 `connect` 用法相似：

```javascript
@withMobx(props => {
  return {
    userStore: new UserStore()
  }
})
@observer
export default class UserForm extends Component {

}
```

`withMobx` 的实现也非常简单，鉴于我们这里并不是高阶组件节目，这里就不鉴赏高阶组件的各种开发模式，你也可以借助 [acdlite/recompose](https://github.com/acdlite/recompose) 创建。这里直接亮出手写答案了:

```javascript
export default function withMobx(injectFunction) {
  return function(ComposedComponent) {
    return class extends React.Component {
      constructor(props) {
        super(props);
        this.injectedProps = injectFunction(props);
      }
      render() {
        // const injectedProps = injectFunction(this.props);
        return <ComposedComponent {...this.props} {...this.injectedProps} />;
      }
    };
  };
}
```

值得玩味的事情是 `this.injectedProps` 的计算位置，是放在即将返回的匿名类的构造函数中。但事实上它可以存在于其他的位置，比如上述代码的注释中，即 `render` 函数里。但是这样可能会导致一些问题的发生，这里可以发挥你们的想象力。我就不再解释了

### 缓存非常重要

在频繁的创建 store 实例的过程中会造成数据的浪费。

假设有组件用于选择某个国家的城市`<CitySelector country="" />`。你只需要传递进这个国家的名称，它就能够动态的请求该国家下所有的城市名称以供选择。但是因为被销毁的缘故，之前请求的结果并没有保存下来，导致下次传入相同国家时会再次请求相同的数据。所以：

1. 在每次请求之前，先检查缓存中是否有需要的数据，如果存在则直接使用
2. 在请求数据之后（意味着之前没有缓存），将数据先保存在缓存中

而至于 cache 模块借助`Map`数据类型就能够实现，key 则按照具体的需求设置即可

## 准守性能规则

虽然从某些方面来说 Mobx 比 Redux 性能更好，但是 Mobx 仍然有罩门的。举个例子：

```javascript
class Person extends React.Component {
  constructor(props) {
    super(props)
  }
  render() {
    console.log('Person Render')
    const {name} = this.props
    return <li>{name}</li>
  }
}

@inject("store")
@observer
class App extends React.Component {
  constructor(props) {
    super(props);
    setInterval(this.props.store.updateSomeonesName, 1000 * 1)
  }
  render() {
    console.log('App Render')
    const list = this.props.store.list
    return <ul>
      {list.map((person) => {
        return <Person key={person.name} name={person.name} ></Person>
      })}
    </ul>
  }
}
```
`updateSomeonesName`方法会随机的更改列表中某个对象的`name`字段。从实际运行的状态看，即使只有一个对象的字段发生了更改，整个`<App />`和其他未修改的对象的`<Person />`组件都会重新进行渲染。

改善这个问题的方法之一，就是给`Person`组件同样以`mobx-react`类库的`observer`进行“装饰”：

```javascript
@observer
class Person extends React.Component {
```
`observer`的作用用原文的话说就是：
> Function (and decorator) that converts a React component definition, React component class or stand-alone render function into a reactive component, which tracks which observables are used by render and automatically re-renders the component when one of these values changes.

经过“装饰”之后的组件`Person`，只有在`name`发生更改之后才会进行重新渲染

然而我们可以做的更好，用官方的话说就是**Dereference values late**，用中文话说就是**当需要的时候再对值进行引用**

在上面的代码的例子中，假设我们不会在`Person`组件中使用（渲染）`name`字段，而是通过另一个名为`PersonName`的组件进行渲染，那么在`App`里，`Person`里，都不应该访问`name`字段。否则会造成无意义的渲染。简单来说，就是下面 1 和 2 的区别：

```javascript
@observer
class Person extends React.Component {
  constructor(props) {
    super(props)
  }
  render() {
    // 1:
    return <PersonName person={this.props.person}></PersonName>
    // 2:
    return <PersonName name={this.props.person.name}></PersonName>
  }
}
```
- 如果你使用了 1 的写法，那么当`person.name`发生更改时，`<Person />`组件不会重新渲染，只有 `<PersonName />` 会重新渲染
- 如果你使用了 2 的写法，那么当`person.name`发生更改时，`<Person />`组件会重新渲染，`<PersonName />`组件也会重新渲染

当然直接传递整个对象到组件中也存在问题，会造成数据的冗余，给今后的维护者造成困难

## 结尾

当然还有很多关于开发 Mobx 应用的经验可以分享，但以上三条是我个人感触最深的，也是我个人认为最重要的。 希望对大家有所帮助