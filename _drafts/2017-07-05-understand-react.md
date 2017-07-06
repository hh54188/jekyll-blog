# 面试系列之二：你真的了解React吗

## 前言

这不是一篇深度解析React工作原理的文章，而是帮你完善你可能忽略的React的知识点。

生活中我们频繁使用电脑、冰箱，但不一定需要知道电脑和冰箱是如何工作的。但是作为一个技术人员，从jQuery到React，你不仅需要熟悉使用他们，还需要了解他们的工作原理，以体现你的精通。然而如何体现你对它们的了解，无非是从大量的相关细节问题。相信绝大部分公司的技术栈都已经切换到了React，你也知道面试官最喜欢的面试套路是从你简历上的“知识点”顺藤摸瓜式的追问你，所以React的相关问题无法避免，而同时你也要问自己是否真的了解它，还只是会使用它而已。

所以在继续阅读之前，请尝试回答以下有关React的问题（其中有九成是我在面试中遇到的，另外一成是我自己认为有必要了解的），其中粗体字部分是我认为重点需要掌握的知识点，不仅是在面试过程中，在实际代码过程中需要运用到的：

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
- [Webpack如何打包输出多个文件？](#about_webpack)
    - webpack打包时如何工作的？
        - 如何解决循环引用的问题
    - 在什么情况下需要打包输出多个文件？
    - loader和plugin的差别
    - 你觉得使用过什么高级技巧吗？
- [（开放问题）React的生态你使用过哪些](#webpack_ecology)

如果以上问题都难不倒你，那么恭喜你，你的React技能树已满。如果有些问题不确定或者不了解的也没有关系，请继续阅读这篇文章。我将一一对这些问题做解答。

其中有些问题的答案比较长，可能会和一篇文章相当。所以关于React可能会拆成上中下三集来说。

重要的不是把这些问题的答案背下来，而应该重点去理解他们。如果你之前没有React的开发经验，可能对于其中的一些道理没有那么深的感触。所以建议边做，边学，边理解。

但是请注意有些问题可能并没有官方答案，是我个人通过经验得出的，所以仅作参考喔。

## 如何设计一个好的组件<a name="design_component"></a>

这道题目中的“组件”不仅限于React组件，广义上看，前端代码模块，独立类库甚至函数在编写时都应该遵循良好的规则。

怎样的组件设计算的上“好”，要从几个层次来看这个问题。我们从宏观到微观依次来看。

首先你要知道组件的出现是为了解决怎样的问题——是为了更好的复用。然而怎样才能能其他的使用者更好的复用你的组件？API够烂肯定不行，这样的话其他人就没法调用；兼容性差也不行，因为同一个系统中可能存在不同版本React编写的组件，甚至还可能和Vue组件发生交互；内部实现差了也不行，这样的话你的下一任接替你职位的人修改起来会非常麻烦，结果不外乎重写。

### 高内聚，低耦合

我绝对相信这六个字你已经听到耳朵起茧。但我还是要重申，无论是什么语言编程，无论是前端还是后端，无论多耳熟能详的架构（CQRS、Microervices），无论是多具体的设计原则（后面会说的SOLID），本质上都是对这个原则的实践。所以我们的设计也不例外。

顾名思义，在做组件设计时，甚至编写函数时，应该把相同功能的部分放在一起，而把不相干的部分尽可能的撇开关系。如果你想去反向验证你的设计是否符合这个原则的话，可以尝试去修改这个模块的一个功能，看看到底是否会牵连其他模块的修改。

接下来我们从SOLID原则看看对这六个字的具体实践。

### S.O.L.I.D

SOLID 原则是面向对象设计中的原则，但就我经验而言，其中的这些也同样适用于组件设计。例如**单一职责（Single responsibility principle）**，React组件设计推崇的是“组合”，而非“继承”。例如你的页面需要一个表单组件，表单中需要有输入框，按钮，列表，单选框等。那么在开发中你不应该只开发一（整）个表单组件（`<Form>`），而是应该开发若干个单一功能的组件，比如输入框`<Input>`、提交按钮`<Submit>`、单选框`<Checkbox>`等，最后再将它们组合起来。这其中的重点是每个组件仅做一件事。

不仅仅是编写组件，哪怕仅仅是编写一个简单的函数也是应该如此，例如你需要一个函数异步请求数据并返回JSON数据格式，那么你应该拆分为两个函数，一个复杂数据请求，另一个负责数据转化。你可能会好奇为什么一个简单的`JSON.parse`也拆分出来，因为将来需要会变动，你可能不仅仅需要`JSON.parse`，还需要转义，需要转化为`proto buffer`数据格式。而拆分之后如果再面临修改的话，就不会影响到数据请求部分的代码。

上面这个例子也同样适用于**开放/封闭（Open/closed principle）**原则。开放/封闭强调的是对修改封闭（禁止修改内部代码），对拓展开放（允许你拓展功能）。你一定纳闷如果不允许修改代码的话如何拓展功能呢，在传统的面向对象编程中，这样的需求是通过继承和接口机制来实现的。这个原则同样是放之四海而皆准的，因为修改意味着风险，可能会影响到不用修改的代码， 同时意味着暴露细节。

虽然ES6中提拱了继承机制，React也允许使用ES6编写，但React并不推荐组件之间采用继承的方式进行拓展，而是推荐采用 Higher-Order Components 的方式。这个在后面会详细叙述。

**接口隔离（Interface segregation principle）**这个就放之四海而皆准了。第三方类库或者模块都避免不了对外提供调用接口，比如对于jQuery来说`$`是选择器，`css`用于设置样式，`animate`负责动画，你不希望把这三个接口都合并成一个叫做`together`吧，虽然实现起来没有问题，但是对于你将来维护这个类库，以及使用者调用类库，以及调用者的接替者阅读代码（因为他要区分不同上下文中调用这个接口究竟是用来干嘛的），都是不小的困难。

最后一条**依赖反转（Inversion Of Control）**原则。这条原则听上去有点拗口，不过它有另外一个名字：Hollywood Principle，虽然我也不理解为什么会有这个别名。这条原则是意思是，当你在为一个框架编写模块或者组件时，你只需要负责实现接口，并且到注册到框架里即可，然后等待框架来调用你，所以它的另另一个别名是 “Don't call us, we'll call you”。

这么说你可能没什么太大感觉，也不明白和“依赖”和“反转”有什么关系，说到底其实是一个控制权的问题。这里举一个图书Building Microservices: Designing Fine-Grained Systems中的例子。常规情况下当你在用express编写一个server时，代码是这样的：
```javascript
const app = express();
module.exports = function (app) {
	app.get('/newRoute', function(req, res) {...})
};
```
这意味着你正在编写的这个模块负责了`/newRoute`这个路径的处理，这个模块在掌握着主动权。

而用依赖反转的写法是：
```javascript
module.exports = function plugin() {
	return {
		method: 'get',
		route: '/newRoute',
		handler: function(req, res) {...}
	}
}
```
意味着你把控制权交给了引用这个模块的框架，这样的对比就体现了控制权的反转。

其实前端编程中常常用到这个原则，注入依赖就是对这个思维的体现。比如requireJS和Angular1.0中对依赖模块的引用使用的都是注入依赖的思想。

至于**里氏替换**原则在前端是真的用不上了

### Higher-Order Components 和 Stateless Components

上面我们了解完总体的设计思想之后，细化的来看针对React组件还有哪些具体的设计思想。

在上面我提过，React官方并不建议使用“继承”的方式对组件进行拓展，而是推荐使用一种类似于组合的方式，被称为“Higher-Order Components”

- [Hollywood Principle](http://wiki.c2.com/?HollywoodPrinciple)
- [Higher-Order Components](https://facebook.github.io/react/docs/higher-order-components.html)