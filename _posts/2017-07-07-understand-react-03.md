---
layout: post
title: 面试系列之四：你真的了解React吗（下）Flux与Vuex的差异以及Webpack
modified: 2017-07-07
tags: [javascript, react, interview]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

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

本文是本系列的最后一篇文章，之前的内容可以参考之前的本系列的前两篇文章：

【】

## 如果你使用过Redux与Vuex的话，聊聊他们的区别与你的心得 <a name="flux_vs_vuex"></a>

我个人是比较擅长Flux与Vuex的，这里主要是分享我对Flux与Vux的使用经验，聊聊Flux与Vuex的区别。这题也算的上是半开放式的命题，Redux我也学习过写过相关的代码，但是相对Vuex，我没法详细的说出以及比较Flux与Redux的优劣。所以各位在面试的时候根据自己的情况酌情回答。如果以后有机会的话，我也会补上对Redux的使用心得。

就我个人的经验我更倾向于使用Vuex，它们很相似，都强调组件化，都强调单向数据流，Vuex也是受到Flux启发而诞生。但根据它们之间的差异说一说我选择Vuex的理由：

- **代码文件大小**：React代码打包之后相对较大，基本是300KB起跳；而Vue和Vuex框架代码则相对较小，基础库能维持在100KB左右。
- **现成的框架**：在Flux初期，Facebook只是推出了Flux这个框架概念，而没有实现这个框架。除非你使用一些第三方的Flux框架，否则你需要自己去实现Flux中的两个事件机制（Component对于Store的响应，Store对于Action的响应）。当然现在React的github项目里已经有Flux框架的示例代码，以及他们推出了Relay框架。相反Vuex不仅提出了这个框架概念，还实现并且提供了这个框架，让开发起来更加便捷。
- **针对性的改进**：如果你阅读过Vuex的官方文档的话，你会明白Vuex其实是针对Flux存在的一些缺陷而开发的。具体的缺陷其实我们在上一篇中提到过，例如不同的组件都维护自己的状态的话，不同组件之间想改变对方的状态其实会比较困难的。Vuex的解决办法也是上一篇中提到的那样，把state提升到全局的高度，尽可能是使用stateless组件。同时又引入了
`module`等概念更利于代码的解耦和开发。
- **具体细节上的差异**：Vuex中保留了action与store的概念，并且引入了新的mutation。action和mutation广义上来说都是提交对store修改，不同的是action可以是异步的，并且大多数情况是在event handler中提交，通过`$store.dispatch`方法；唯一修改 Store 的地方只能通过mutation，而且mutation必须是同步的，直接对store进行修改，举例一个简单store的例子：

```javascript
export const SampleStore = {
    state: {
        data: ''
    },  
    actions: {
        fetchData({commit}) {
            fetch('/api/sample', (response) => {
                return response.json();
            }).then((data) => {
                commit('updateData', data);
            });
        }
    },
    mutations: {
        updateData(inputData, state) {
            state.data = inputData;
        }
    },
    getters: {
        data: state => state.data
    }
}
```
注意在上面的例子中，`fetchData`这个action是用于异步的请求数据，而`updateData`这个mutation用于同步的修改store，而component中的event handler只能调用action，而不允许直接调用mutation。我认为这以及非常直观的显示了它们之间的差异。

最后新增的`getters`类似于面向对象编程语言中property的访问器，保证view访问到的数据是你允许访问的。

### Vue.js 的双向绑定是如何实现的

传统的数据绑定不外乎两种方式：

- 事件机制（pub/sub）：我们通过特定的方法修改数据，例如`Store.set('key', 'value')`，`set`方法修改数据的同时触发一个事件，告诉view数据发生了更改，view立即从新从store拉取数据。这类似于Flux中View对于Store数据的响应，只不过通过某种方法或者directive将这种机制封装起来了。这种机制的弱势在于你没法用传统的方式等号`=`对数据进行赋值。

- 轮询（pull/dirty check）：这个方式就更加简单了，数据的消费方不断的检测数据有没有修改。当然不是无时无刻的进行检测，而是在input事件或者change事件的时候进行检测。Angular 1.0使用的就是这种机制。我个人倾向于把这种方式称为轮询而不是脏检查

然而对于Javascript来说，还存在第三种方式，那就是利用Javascript中的Object天生的支持的属性访问器。

在Javascript中，我们可以给对象中的值定义访问器，例如：

```javascript
let data = {};
Object.defineProperty(data, 'key',  {
    get() {
        console.log('Get method invoked');
    },
    set(newVal) {
        console.log('Set method invoked');
    }
})
```
那么接下来当你每次想访问`data`中`key`字段时，无论是取值`data.key`还是赋值`data.key = 'Hi'`，都会有打印信息。这也意味着，我们能够在用户执行普通的赋值和取值操作时，做一些事情，例如通知数据的消费者数据发生了更改，让它们重新编译模板。这也就是`Vue.js`双向绑定的思路。

当然这只是双向数据绑定的一个环节，但是是最核心的环节，其他还包括如何添加订阅者，如何编译模板等等，在这里就不详述了,可以参考以下两篇文章：

- [Vue.js双向绑定的实现原理](http://www.cnblogs.com/kidney/p/6052935.html)
- [剖析Vue实现原理 - 如何实现双向绑定mvvm](https://github.com/DMQ/mvvm)

## Webpack如何打包输出多个文件？<a name="about_webpack"></a>

关于Webpack的入门，可以参考我的之前的一片文章[《Webpack 速成》](https://zhuanlan.zhihu.com/p/26041084)，这篇文章从需求出发，解释了Webpack的一些基本用法。

首先你要明白Webpack的用途，从最终效果上来说它和gulp或者grunt非常相似，甚至有些功能是重叠的。但Webpack的初衷是用于模块打包，解决模块间的兼容性问题。例如有些模块是在服务端（Node.js）开发使用，想挪用到浏览器端使用；有的模块是以ES6 module标准编写，而有的模块则是以AMD标准编写的，而它们之间需要互相调用。你一定听说过UMD这个概念，Universal Module Define，就是在定义模块是尽可能的兼容多的标准。而Webpack帮你解决了这个问题，让你不用的过多担心标准，而专注于模块的开发。

关于loader和plugin的区别在刚刚说的那篇入门中有提到，简单来说loader决定了你Webpack打包模块的能力，例如你需要打包vue组件，那么你需要引入vue-loader；如果你需要编译打包less代码，则需要引入less-loader。而plugin提供的则是模块打包之后更边缘的便捷功能，例如Webpack自带Uglify插件用于压缩代码，自带CommonsChunkPlugin插件用于提取公共模块。

打包多个文件的场景很简单，但是实际情况中运用的或许不多。举一个不恰当的例子，例如你的站点有多个页面（home、gallery、about），但每个页面都是一个Vuex架构下的单页面应用。于是可以设置多个入口，key为入口名称，value为入口文件路径：

```javascript
entry: {
    app: './entry/app.js',
    gallery: './entry/gallery.js',
    about: './entry/about.js'
}
```
同时在`output`字段处，设置文件的输出规则是以入口的key开头，即下面代码filename字段的`[name]`值：

```javascript
output: {
    path: path.resolve('.', 'output'),
    filename: '[name].bundle.js'
}
```

当然`[name]`也可以替换为其他的字段，例如有其他字段可选：

- `[name]`: The module name
- `[id]`: The module identifier
- `[file]`: The module filename

不知道你有没有考虑过Webpack的打包原理，为什么它能将多个文件都打包在一起呢？如果你有了解过之前RequireJS或者SeaJS的加载原理的话，其实打包过程也很相似：一个模块如果有依赖模块的话，在所有的依赖模块加载完成之前它自己的工厂函数代码是不会被执行的，即使执行也会报错，因为它依赖的变量和函数都并不存在。Webpack打包原理也相同：首先Webpack需要有一个入口模块，也就是webpack配置文件里的entry。通过对入口模块进行语法分析也好，注入依赖分析也好，找到模块的依赖。此时Webpack应该会有一个HashMap，key为模块的路径或者名称，而value则为模块的工厂代码或者具体内容。HashMap主要有两个作用，一方面是用于缓存（可能存在一个模块被多次引用的情况），另一方面则用于标记（模块是否被加载）。Webpack则针对入口模块以深度优先的原则逐个将依赖模块进行加载。最后将入口模块自己打包进bundle中。

然而以上所说的这个打包流程我故意漏掉了一个重要环节，就是如何解决模块间循环引用的问题（`A`引用模块`B`，`B`同时引用了模块`A`）。但就解决循环引用这个问题而言，是可以以论文的篇幅进行叙述的。遗憾的是针对Webpack或者RequireJS，我并没有找到确切的资料描述它们是如何解决循环引用的问题。在面试中询问我的这个问题的时候，我临时想到的是一个简单粗暴的办法，即在对一个模块进行构建时对它的依赖建立一个链表，例如模块A的依赖链表是：`B->C->D->E->F`，在构建这个链表的同时，比如我们打算在`F`后添加模块`G`时，我们会去检测`G`的依赖链表里是否存在模块`A`，如果存在的话则形成了循环依赖。

最后关于Webpack的高级用法——我被问到了这个问题，但我不知道如何定义什么是高级用法。如果你开发过Webpack的plugin我觉得算是一项加分点，而至于其他点，我认为在面试中你可以把你认为的的高级用法都说出来，哪怕只是使用了`alias`。

## 你使用过哪些React的生态类库<a name="webpack_ecology"></a>

这就是一个完全开放的命题了。如果说之前的答案还能靠强化学习甚至死记硬背的话，那么这一题完全是要依靠个人平时的积累。你要叙述的不仅仅是你用过什么，还要告诉面试官你为什么用它和解决了什么问题。像报菜名一样报上一大堆类库的名称没有任何意义

这是最后一个问题了。顺便也适合做一个结尾，在面试的过程中最大的一个感悟是，**技术，或者说是能力在日常中就应该积累**。为什么我说日常而不是工作，因为大部分工作使用的技术方向还是较窄，尤其是新技术和允许发挥的空间（代码兼容性、时间排期）。等到你有朝一日想换工作就晚了，罗马不是一天建成的。我认为最好的积累方式应该是在工作中发现问题，并且尝试用天马行空的技术解决问题。不要先想手上有什么，能解决什么样的问题；而是应该先想解决方案，再想方设法用技术去实现方案。噢，对了，这样同时也能增强自己的项目经历。

最后送大家两个锦囊：

第一句是关于如何学好前端的。至于出处拿着这句话直接谷歌去吧，我个人是非常认同这个建议的

>First do it, then do it right, then do it better. ——Addy Osmani

第二句是鸡汤，给你再学习前端挫败时候看的

>Ever tried. Ever failed. No matter. Try again. Fail again. Fail better. ——Samuel Beckett

- [Vuex](https://vuex.vuejs.org/zh-cn/)
- [Vue.js双向绑定的实现原理](http://www.cnblogs.com/kidney/p/6052935.html)
- [剖析Vue实现原理 - 如何实现双向绑定mvvm](https://github.com/DMQ/mvvm)

