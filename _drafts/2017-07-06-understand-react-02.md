# 面试系列之三：你真的了解React吗（中）组件间的通信以及React优化

本系列文章旨在完善你可能忽略的React相关的知识点，通过回答以下问题：

- [React解决了什么问题](#why_need_react)
- **[如何设计一个好的组件？](#design_component)**
- **[组件的Render函数在何时被调用？](#when_render_invoked)**
    - 调用时DOM就一定会被更新吗？
- [组件的生命周期有哪些？](#react_lifecircle)
    - 当某些第三方类库想对DOM初始化，或者进行远程数据加载时，应该在哪个周期中完成？
    - 在哪些声明周期中可以修改组件的state？
- **[不同父节点的组件需要对彼此的状态进行改变时应该实现？](#component_communication)**
    - 如何设计出一个好的Flux架构
    - 如何设计出一个好的React组件
- [如何进行优化？](#component_optimize)
    - 组件中的key属性有什么用？
- [Component 与 Element 与 Instance 的区别](#component_element_diff)
- **[如果你使用过Redux与Vuex的话，聊聊他们的区别与你的心得](#flux_vs_vuex)**
    - Vue.js 的双向绑定是如何实现的？
- [Webpack如何打包输出多个文件？](#about_webpack)
    - webpack打包时如何工作的？
        - 如何解决循环引用的问题
    - 在什么情况下需要打包输出多个文件？
    - loader和plugin的差别
    - 你觉得使用过什么高级技巧吗？
- [（开放问题）React的生态你使用过哪些类库](#webpack_ecology)

本文是本系列的中篇，之前的内容可以参考之前的本系列的前一篇文章：

## 组件间的通信和架构设计问题

在上一篇的文章中，我们主要聊了如何去设计一个好的组件。然而那个视角是独立的，甚至说是狭隘的，只考虑了的单个组件自己。在这一篇中我们将考虑更复杂的情况，例如组件间的交互，以及组件在Flux架构中的位置；以及一些更细节的问题，例如究竟是应该使用state还是props。

其实Facebook官方关于React最佳实践已经写的非常好的了，但可惜的是这些实践和工作原理、教程都[混杂在一起](https://facebook.github.io/react/docs/hello-world.html)，并且长篇累牍，让人阅读起来非常没有重点；另一方面如果你是个新手，或许你会看完所有的这些文档，但你还没有实践的经验，你满脑子想的其实是如何开始写你的第一个Hello World应用，所以这些最佳实践你也很快就忘了。

这一小节的主要内容都来自上面所说的Facebook关于React的[官方文档](https://facebook.github.io/react/docs/hello-world.html)，还有一部分来自一篇我认为很有价值，理念和官方文档一致但又写的更好的一篇文章：[Flux in Depth. Overview and Components](http://blog.mgechev.com/2015/05/15/flux-in-depth-overview-components/)

### 组件间的通信问题

首先假设我们有一个用React构建的单页面应用，组件间的关系如下图所示：

![sample-state-change](./images/sample-state-change.png)

我们首先假设每一个组件都在维护自己的状态（state），而不是使用传递下来的props。那么问题来了，如果右侧的组件E想改变左侧的组件B的状态，应该怎么办？

这个例子其实想表达的是一类通用的业务场景，就是组件间状态的修改，也有可能是子组件修改父组件的状态。上面的这个例子是一类极端的情况，被修改的组件与发出修改的组件不在同一个父组件中。

Facebook的官方推荐办法（曾经是）是使用事件机制（现在这个页面已经跳转到另一个解决方案了，也就是下面要叙述的）。但我和绝大多数人都不认为这是一个好的机制。你可以想象一旦应用中这样的需求增多，事件和回调函数满天飞，则情况右回到了Flux之前的原地状态：调试和维护代码都很难进行，这会是一个灾难。

第二个方案是传递接口。如果E想改变B的状态，那么B要传递给E组件一个修改状态的接口。不过因为Flux属性是从上至下传递的关系，所以接口的传递应该是如下图所示：

![sample-state-change-callbacks](./images/sample-state-change-callbacks.png)


- [Flux in Depth. Overview and Components](http://blog.mgechev.com/2015/05/15/flux-in-depth-overview-components/)
- [Interactivity and Dynamic UIs](https://shripadk.github.io/react/docs/interactivity-and-dynamic-uis.html)
- [Components and Props](https://facebook.github.io/react/docs/components-and-props.html)




