---
layout: post
title: 从对 Vue 中 mixin 的批评，到对模块间依赖关系的探讨
modified: 2021-03-14
tags: [javascript, vue]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---
# 从对 Vue 中 mixin 的批评，到对模块间依赖关系的探讨

编程框架日新月异，工具平台推陈出新。但有意思的是，代码的坏味道不会因为你使用工具的时髦而自行消散，团队成员的编程水平也不会随着工具的进化而水涨船高。

工具从来都不是你写出好代码的决定因素，相反它可能是最无关紧要的条件之一，但偏偏又给大部分人救命稻草一般的错觉。它更类似于催化剂，能助你的代码一臂之力，也能加速它的灭亡。

mixin 就是很好的一个例子。

## mixin 语法回顾

如果你对 Vue 中的 mixin 语法还不甚了解，我用一句话和一个例子就能将它简单概括：mixin 是一种组件间的代码共享机制，允许你将代码封装为一个独立模块，将其用于在多个组件之间共享。

假设 Toolbar 和 Card 组件在实例化这个组件时都需要传递 title、subTitle 属性，那么你就可以考虑将这两个属性封装到一个公共的例如名为 ComminMixin 的模块里，然后将这个模块“插入”到有需求的组件中。

首先定义 CommonMixin 模块代码：

```javascript
export default {
  props: {
    title: string,
    subTitle: string,
  }
}
```

接着在 Toolbar 和 Card 组件中对其进行引用即可：

```javascript
import CommonMixin from '../mixins/commin-mixin.js'
export default {
  mixins: [CommonMixin]
}
```

这种方式同你直接在 Toolbar 中定义 title 和 subTitle 的属性并无不同：

```javascript
import CommonMixin from '../mixins/commin-mixin.js'
export default {
  props： {
    title: string,
	subTitle: string
  }
}
```

mixin 机制存在什么样的问题？早在 2016 年的时候 React 官方就发布过一篇名为 [Mixins Considered Harmful]( https://reactjs.org/blog/2016/07/13/mixins-considered-harmful.html ) 的文章，其中详细叙述了 mixin 机制下会引起的几类问题，比如命名冲突、比如引起滚雪球般的复杂性等等。这里我只提及我认为危害最大的一点：**隐式依赖**。并借此引出我们下一节的话题。

## 隐式依赖

正如上一节代码所示，mixin 模块内的代码和组件内的代码是等价的，如果你知道 mixin 中存在一个名为 hello 的方法，你完全和可以在组件中以 `this.hello()` 的形式无差别的对其进行调用。

并且这种等价还是双向的，虽然 mixin 模块在当初被定义时并不知道将来会有哪些组件引用它的，但如果当下你十分确定某个消费它的组件中注定存在 world 方法，你就可以在 mixin 模块中调用 `this.world()` 。这种关系还能延申至 mixin 之间，无论是平行关系还是嵌套关系下的 mixin 模块（mixin 模块可以继续引用其他的 mixin 模块），它们之间都可以互相访问变量和方法。

在这里你是不是已经嗅到了危险的味道？

看上去方便至极了！可一旦需要对代码进行维护时，问题就暴露了，哪怕你只是想理解某个极小的代码片段。

假设某个同事在组件中偶遇了 `hello` 方法，想给它新增一个参数来实现更多的功能——这看上去不起眼的小事在实际操作起来却比登天还难。因为他根本不知道该方法是在哪个 mixin 模块中被定义的，他所知道的只有方法属于 `this`，于是他不得不翻看每一个 mixin 的定义。

退一步说即使他在某个 mixin 中找到了该方法的定义，又会遇到另一个难题：不敢修改这个函数的签名。虽然他可以知道有多少个组件文件引用了这个 mixin 模块，但是他不知道有多少处直接或者间接的调用了这个方法。也就是说这一次修改会造成的后果和带来的影响难以评估。

所有这些问题的根源在于组件与 mixin 间，mixin 与 mixin 间的依赖是隐式的。也就是说当 A 模块依赖 B 函数时，这种关系既不是通过显示的声明（比如 import 语句或者依赖注入的方式）取得，也不是通过公共约定（例如 windows 对象上存在 `getComputedStyle` 方法）确定下来的。

这种关系也让 IDE 武功尽废，我发现解决这个问题的最后方式竟然是 Ctrl + C（复制），Ctrl + V（粘贴），Ctrl + Shift + F（全局查找），Ctrl + H（文本替换）。

隐式依赖不仅会对脚本代码带来负面影响，对样式代码也会。flex 布局是一个正面的例子：如果你想控制父容器内孩子元素的布局，只需要修改父容器上 flex 有关的属性即可，你不依赖孩子元素的 DOM 结构，更不依赖孩子元素上的样式；而一个反模式的例子是 `text-overflow: ellipsis` 属性，单一的该样式属性是不足以自动省略容器内的文字，容器还需要满足 1) 宽度必须是 px 像素为单位 2) 元素必须拥有 `overflow:hidden` 和 `white-space:nowrap` 样式。而 `text-overflow` 属性本身并没有告知我们还需要这些“配套设施”。

最终带来的局面刚好符合 Uncle Bob Martin 在他的 [The principles of OOD](http://butunclebob.com/ArticleS.UncleBob.PrinciplesOfOod) 一些列文章中谈到过的糟糕设计（Bad Design）的几个特征，比如
	• 僵化（Ridigity）：代码难以修改，因为改动会影响到的地方太多
	• 脆弱（Fragility）：当你做出修改时，系统中预期之外的地方会遭到破坏
	• 难以修改（Immobility）：代码很难被复用，因为它与当前系统中的功能耦合在了一起

前两条在上面解释过了，很好理解。至于最后一条特征，mixin 不仅似乎没有违背，还执行的非常好不是吗？这就涉及到我们下一节要聊的内容

## Defactoring

这里暂停一下，我们似乎陷入到了一种窘境当中：我们都承认 mixin 是极其强大灵活的，它将代码的复用发挥到了极致。但现在看了恰恰是着这种灵活性给我们的代码带来了灾难。我们应该如何理解这种矛盾？

我们要回答的第一个问题是：这种灵活性真的是我们想要的吗？

[Reginald Braithwaite](https://raganwald.com/) 在13 年写过一篇很有意思的技术文章 [Defactoring](https://raganwald.com/2013/10/08/defactoring.html)，注意不是重构的那个单词 Refctoring。

什么是 defactoring? 简而言之如果我们将把大单体代码拆分为细粒度碎片代码过程称之为 factoring 的话，那么 defactoring 代指的就是相反将代码碎片拼装起来的过程。

为什么我们会需要 defactoring? 因为灵活性带来的并不总是好处，它会给我们带来认知上的困恼，你总是需要将不同的碎片碎片拼凑起来之后才能理解整幅图的原貌；模棱两可的代码总是会让你摸不清它的意图；更不要说代码的复杂性了。

你可能会问如果万一呢？有时“灵活性”的背后是我们对于未来的恐惧：我们可能需要支持 A 功能或者支持 B 功能，但事实上你不需要提前实现这些可能性，让你的代码有能力应对这些可能性即可。

所以说恰当的 defactoring 是有必要的。

第二点我们需要考虑到人的因素。我很喜欢 [Coding Horror](https://blog.codinghorror.com/) 提出的 [Falling Into The Pit of Success](https://blog.codinghorror.com/falling-into-the-pit-of-success/) 的理论，引用原文中的话说就是：

> a well-designed system makes it easy to do the right things and annoying (but not impossible) to do the wrong things.

在站在项目和团队的角度上考虑代码的可维护性时尤其如此。

除此之外代码应该是易于修改，并且是很容易修改正确的。比如 TypeScript 相比 JavaScript 就是，但很明显 mixin 并不是。

一段代码从你编写完毕之日起，它的命运就再也不掌握在你手中了。他人很可能会领悟不到你设计某个属性的意图，你精心设计的一段优化性能的代码很容易就被破坏掉。所以我们需要适应度函数，需要有测试。

在实际的工作中 mixin 大部分被滥用了。你可能会定义一个名为 ComponentCommonMixin 的模块，初衷是用于存储和所有组件关联的通用属性。但后续的开发人员并不晓得你的初衷是什么，导致他们在规划接下来的公共属性时会无脑的往这个模块里添加，让它变得臃肿不堪——“噢，因为它是公共的”。

表面上看它分离了公共属性代码和组件专属代码，但实际上在 mixin 模块内部刚好是紧耦合这种反模式的最佳体现。这种状态下的 mixin 根本毫无“单一职责（Single Responsibility ）”可言，在一个模块内部既可能包含了和样式有关的属性，也可能包含了和权限有关的行为，涉及对任何一块业务的需求变更都会导致模块被“打开”进行重新修改，这也有违开放封闭原则（Open-closed ）。

## 普适的 mixin 模式

令人欣慰的是这种 mixin 中的隐式依赖问题是 Vue 框架下的特例。

提起代码的复用我们首先想到的是继承，但继承不是万能的：继承打破了父类的封装；继承要求子类覆写方法时与父类兼容；多数语言不支持多继承。一言以蔽之继承机制对类的抽象设计能力要求很高，劣质的抽象比没有抽象更难维护。

在这些限制之下，组合模式似乎是一类不错的选择，而 mixin 就是组合的一种实现方式。这里我们直接参考 [TypeScript 2.2 RC 官方技术博客]( https://devblogs.microsoft.com/typescript/announcing-typescript-2-2-rc/ ) 中的一个例子，来说明 mixin 是如何实现的。简单来说分为下面四个步骤：

1. takes a constructor
2. declares a class that extends that constructor
3. adds members to that new class
4. and returns the class itself.

这里我们尝试实现一个 Timestamped mixin，它会在需要拓展的类上添加一个 timestamp 属性：

```javascript
type Constructor<T = {}> = new (...args: any[]) => T;

function Timestamped<TBase extends Constructor>(Base: TBase) {
  return class extends Base {
    timestamp = Date.now();
  };
}
```

首先  `Constructor<T = {}>` 是一类用于描述构造函数签名的类型，它支持传入泛型 T， T 代表着构造函数实例化后返回的结果类型。它的作用不大，主要为了在下面的方法中承接基类而已。

Timestamped 方法接收一个基类作为参数，这个基类必须要符合上面定义的构造函数签名，它必须是能“构造出点什么东西的”。在函数的实现中，它用一个匿名类来继承这个基类，并且在匿名类上新增一个 timestamp 属性之后返回出去。

使用的效果怎样呢？我们以一个 Point 类为例，看看如何对它进行拓展

```javascript
class Point {
    x: number;
    y: number;
    constructor(x: number, y: number) {
        this.x = x;
        this.y = y;
    }
}

const TimestampedPoint = Timestamped(Point);

const p = new TimestampedPoint(10, 10);
p.x + p.y;
p.timestamp.getMilliseconds();
```

Point 自身并没有定义 timestamp 属性，但是通过 Timestamped 方法拓展之后，在依旧保留自身行为的同时，又新增了 timestamp 属性。

这种模式可以无限的嵌套下去，例如我们还可以添加 draw、color 等 mixin，同时对 Point 类进行拓展：

```javascript
const NewPoint = draw(color(Timestamped(Point)))
```

这种模式看起来是不是非常眼熟？就是 React 中的高阶组件，你一定使用过 withRouter 或者是 connect 方法来对组件进行封装。

但为什么这种模式下似乎就不存在隐式依赖中提及的问题？因为除了移除模块当中的“隐式”元素之外，我们还间接的调整了模块间的依赖方向。

在下图 Vue 的 mixin 模式中，组件 A 和 B 看似在以单向的方式引用 mixin 模块 B，但实际上因为隐式依赖（上图中灰色虚线所示）的关系，模块和组件间的依赖关系是并无统一方向可言，甚至可以是循环依赖的。

![](../images/vue-mixin-module-dependency/bidirectional-direction-dependency.png)

而下图在 TypeScript 的 mixin 模式中，draw 函数中的匿名类对传递给它函数的类一无所知，它只管往在匿名类中添加自己的属性和行为即可，并且匿名类都是相互独立的。这样就保证了模块之间的依赖是单向的。注意上述的箭头虽然表达的是“依赖”关系，但它并非是 UML 中的依赖，它既没有调用依赖模块的方法，也没有将依赖模块作为自己的成员变量

![](../images/vue-mixin-module-dependency/one-direction-dependency.png)

当然，如果你“足够有信心”的话，你还是可以强行调用传入的基类上的方法，只不过如果你真的打算这么做的话，你可能需要通过接口或者类型将基类约束起来，给出方法的签名来保证它是存在的。

模块间的依赖方向是另一个我们需要关心但可能会被忽略的一点，因为它会影响到我们的调整模块代码的难度。Uncle Bob Martin 在《整洁架构》一书中提出了「组件依赖原则」（Stable Dependencies Principle）。他认为在软件开发中的软件设计不可能是静态的，它注定是需要被调整的，并且不同组件模块调整的频率并不相同。因此，一个注定需要被改动的组件不应该依赖那些难以被撼动组件，否则它自己也会变得难以修改。

例如对于下图中的 y 模块而言，它依赖额外的三个模块。以至于这三个模块中任意一个模块的变更都会给它带来影响，这会导致它变得极不稳定。

![](../images/vue-mixin-module-dependency/depend-on-others.png)

## 隐式依赖的其他体现

隐式依赖另一个极富争议的例子就是服务定位（Service Locator）模式。

服务定位模式在大多数时候被认为是反模式。在前端领域中可以实现但很少被用到。

什么是服务定位模式？假设你在某个类的方法中需要调用某个依赖的方法，你可以在方法中通过 Locator “临时”找到这个依赖：

```java
class  MyClass {
  public void MyMethod() {
    var dep = Locator.resolve(IDep)();
	dep.DoSomething();
  }
} 
```

它能工作没有错，但我们还存在另一种实现方式，我们可以通过创建实例时的构造函数传入依赖，也可以通过依赖注入传入依赖：

```java
class  MyClass {
  public MyClass(IDep dep) {}
  public void MyMethod() {
	dep.DoSomething();
  }
} 
```

在使用服务定位模式实现的前提下，你想创建一个实例并且调用它的方法很可能会失败：

```java
var myClass = new MyClass();
myClass.MyMethod();
```

因为服务定位模式的问题在于它的依赖被隐藏起来了，你无法一眼看穿它对 IDep 的依赖，所以你也就可能不会在项目中引入对应的 Locator 以及 IDep。哪怕你完整收集了它的所有依赖，你还需要额外的引入 Locator 模块，可它与你真正需要的业务功能并无太大关系。如果能在构造函数中进行显式的声明，那这些问题都能够得到避免。

## 结束语

我当然同意 mixin 是中性的，所有事故的背后本质上都是人的问题。但如果我们承认“人”是我们在软件活动中永远也无法消除的不稳定因素的话，那就要面对 mixin 会比其他机制更让我们的软件岌岌可危的这个风险。这个时候我们没有理由视而不见了。

你可能会喜欢

- [开源社区的暗面](https://www.v2think.com/darkside-of-the-opensource)
- [帮助团队成长是唯一的出路](https://www.v2think.com/what-is-leadership)
- [去年做 Tech Leader 犯过最大的错](https://www.v2think.com/tech-leader-mistake)
- [技术写作的困境](https://www.v2think.com/stuck-in-technical-writing)
- [拥抱原则与面对现实](https://www.v2think.com/principles-and-facts)
- [代码与质量的思考与随笔](https://www.v2think.com/think-about-good-code)
- [从对 Vue 中 mixin 的批评，到对模块间依赖关系的探讨](https://www.v2think.com/vue-mixin-module-dependency)