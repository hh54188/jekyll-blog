## 如果你使用过Redux与Vuex的话，聊聊他们的区别与你的心得 <a name="flux_vs_vuex"></a>

我个人是比较擅长Flux与Vuex，这里主要是分享我对Vux的使用经验，聊聊Flux与Vuex的区别。这题也算的上是半开放式的命题，Redux我也学习过写过相关的代码，但是相对Vuex，我没法详细的说出以及比较Flux与Redux的区别。所以各位在面试的时候根据自己的情况酌情回答。如果以后有机会的话，我也会补上对Redux的使用心得。

就我个人的经验我更倾向于使用Vuex，它们很相似，都强调组件化，都强调单向数据流，Vuex也是受到Flux启发而诞生。但根据它们之间的差异说一说我选择Vuex的理由：

- 代码文件大小：React代码打包之后相对较大，基本是300KB起跳；而Vue和Vuex框架代码则相对较小，基础库能维持在100KB左右。
- 现成的框架：在Flux初期，Facebook只是推出了Flux这个框架概念，而没有实现这个框架。除非你使用一些第三方的Flux框架，否则你需要自己去实现Flux中的两个事件机制（Component对于Store的相应，Store对于Action的响应）。当然现在React的github项目里已经有Flux框架的示例代码，以及他们推出了Relay框架。相反Vuex不仅提出了这个框架概念，还实现并且提供了这个框架，让开发起来更加便捷。
- 针对性的改进：如果你阅读过Vuex的官方文档的话，你会明白Vuex其实是针对Flux存在的一些缺陷而开发的。具体的缺陷其实我们在上一篇中提到过，例如不同的组件都维护自己的状态的话，不同组件之间想改变对方的状态其实会比较困难的。Vuex的解决办法也是上一篇中提到的那样，把state提升到全局的高度，尽可能是使用stateless组件。同时又引入了
`module`和`mixin`等概念更利于代码的解耦和开发。
- 具体细节上的差异：Vuex中保留了action与store的概念，并且引入了新的mutation。action和mutation广义上来说都是提交修改，不同的是action可以是异步的，可以在event handler中提交， 而mutation必须是同步的，直接对store进行修改，举例一个简单store的例子：

```javascript
Vuex.store({
    state: {
        data: ''
    },  
    action {
        fetchData({commit}) {
            fetch('/api/sample', (response) => {
                return response.json();
            }).then((data) => {
                commit('updateData', data);
            });
        }
    },
    mutation: {
        updateData(inputData, state) {
            state.data = inputData;
        }
    }
})
```
注意在上面的例子中，`fetchData`这个action是用于异步的请求数据，而`updateData`这个mutation用于同步的修改store，而component中的event handler只能调用action，而不允许直接调用mutation。我认为这以及非常直观的显示了它们之间的差异。

## Webpack如何打包输出多个文件？<a name="about_webpack"></a>

关于Webpack的入门，可以参考我的之前的一片文章[《》]()，这篇文章从需求出发，解释了Webpack的一些基本用法。

首先你要明白Webpack的用途，从最终效果上来说它和gulp或者grunt非常相似，甚至有些功能是重叠的。但Webpack的初衷是用于模块打包，解决模块间的兼容性问题。例如有些模块是在服务端（Node.js）开发使用，想挪用到浏览器端使用；有的模块是以ES6 module标准编写，而有的模块则是以AMD标准编写的，而它们之间需要互相调用。你一定听说过UMD这个概念，Universal Module Define，就是在定义模块是尽可能的兼容多的标准。而Webpack帮你解决了这个问题，让你不用的过多担心标准，而专注于模块的开发。

关于loader和plugin的区别在刚刚说的那篇入门中有提到，简单来说loader决定了你Webpack打包模块的能力，例如你需要打包vue组件，那么你需要引入vue-loader，如果你需要编译打包less代码，你需要引入less-loader。而plugin提供的则是模块打包之后更边缘的便捷功能，例如Webpack自带Uglify用于压缩代码，自带【】

打包多个文件【】

不知道你有没有考虑过Webpack的打包原理，为什么它能将多个文件都打包在一起呢？如果你有了解过之前RequireJS或者SeaJS的加载原理的话，其实打包过程也很相似：一个模块如果有依赖模块的话，在所有的依赖模块加载完成之前它自己的工厂函数代码是不会被执行的，即使执行也会报错，因为它依赖的变量和函数都并不存在。Webpack打包原理也相同：首先Webpack需要有一个入口模块，也就是webpack配置文件里的entry。通过对入口模块进行语法分析也好，注入依赖分析也好，找到模块的依赖。此时Webpack应该会有一个HashMap，key为模块的路径或者名称，而value则为模块的工厂代码或者具体内容。HashMap主要有两个作用，一方面是用于缓存（可能存在一个模块被多次引用的情况），另一方面则用于标记（模块是否被加载）。Webpack则针对入口模块以深度优先的原则逐个将依赖模块进行加载。最后将入口模块自己打包进bundle中【可以通过实际代码发现入口函数的模块代码在最后？】

然而以上所说的这个打包流程我故意漏掉了一个重要环节，就是如何解决模块间循环引用的问题（`A`引用模块`B`，`B`同时引用了模块`A`）。但就解决循环引用这个问题而言，是可以以论文的篇幅进行叙述的。遗憾的是针对Webpack或者RequireJS，我并没有找到确切的资料描述它们是如何解决循环引用的问题。在面试中询问我的这个问题的时候，我临时想到的是一个简单粗暴的办法，即在对一个模块进行构建时对它的依赖建立一个链表，例如模块A的依赖链表是：`B->C->D->E->F`，在构建这个链表的同时，比如我们打算在`F`后添加模块`G`时，我们会去检测`G`的依赖链表里是否存在模块`A`，如果存在的话则形成了循环依赖。

最后关于Webpack的高级用法——我被问到了这个问题，但我不知道如何定义什么是高级用法。如果你开发过Webpack的plugin我觉得算是一项加分点，而至于其他点，我认为在面试中你可以把你认为的的高级用法都说出来，哪怕只是使用了`alias`。

## 你使用过哪些React的生态类库<a name="webpack_ecology"></a>



