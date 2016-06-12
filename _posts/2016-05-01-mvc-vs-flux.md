---
layout: post
title: 写给前端看的架构文章(1)：MVC VS Flux
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2016-05-01
tags: [javascript, front-end, xss, safety]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 前言

在学习React.js的过程中，曾经最让我苦恼的事情是，我需要给自己一个使用这个框架的理由。因为随着学习经验的和工作经验的增长，你会发现类似的技术总是会此消彼长的出现，如果这只是另一个轮子怎么办？加之学习的成本、项目改造的成本甚至周围人来适应你的成本，一味的追逐最新最流行的技术并非是一件好事。

当谈React.js时有必要把它一分为二来讲解。首先要明确的是它只是一个用于视图层的类库(lib)，你能把它当作模板引擎来用(**React.js并不是模板引擎**)，输出html，定义组件。但是你没法仅用React.js类库就搭建一个完整的前端app。之所以要强调这么一点是因为很多人会拿React.js与Angular做比较，这是不公平的，因为angular的定位是一个框架(framework)。angular自带模板引擎，路由引擎，有健全的数据双向绑定机制，内置Ajax请求功能，还能够定义Model。而React.js类库只能用于定义视图组件。

React.js的另一面是它背后的flux架构，这是我们这篇文章着重要谈的。但正如上一段所说，React.js仅依靠自己没法成型一个flux框架，基于React.js的flux框架要么是手动补全了框架中其他角色的代码，要么引入了其他的第三方类库。flux架构也并非是React.js的专利，市面上已经有非常多的独立于React.js的开源框架供使用。

这篇文章的目的就是让你读懂flux架构，我们会直接把flux与mvc比较，来彰显它的优劣。flux架构并非新事物，如果你拥有后端开发背景的话，flux架构一定会让你联想到CQRS(Command-Query Responsibility Segregation)、EDA(Event-Driven Architecture)、DDD(Domain-Driven Design)等概念。至于这些概念具体是什么，和flux有什么关系，会在下一篇中介绍。今天我们聊flux与mvc比较之下它的创新之处在哪。

如果你还完全没有接触过React.js也不太要紧。这一篇的内容主要集中于用图解和文字来讲解架构之间的差异，代码部分简单通俗易懂。

## MVC简介

MVC架构讲程序划分为三个角色，从上到下依次为：
- View: 视图，用户数据展示，同时接受用户输入
- Contorller：响应用户的输入，对数据进行操作，
- Model：负责管理程序需要的数据，并且定义了操作数据的行为。

对于一个简单的MVC架构程序来说，其工作流程如下：

![mvc-simple](../images/mvc-vs-flux/mvc-simple.png)

从最右边的View开始，当用户在UI上进行操作之后，用户的操作被转发到了Controller上，Controller根据用户的操作对数据进行更新(准确来说是调用Model层的API)，数据更新之后自然视图View展现的内容也需要进行更新。Model层此时可以向所有关联的视图发出通知，收到通知的视图重新获取最新的数据。注意这最后一步Model与View的交互，大部分现有的MVC框架将其进行了封装，开发人员只要使用数据绑定即可。

如果上面的流程图还过于抽象的话，我们可以看一段MVC项目的代码，比如基于Nodejs的Kraken框架的[Shopping_Cart](https://github.com/lmarkus/Kraken_Example_Shopping_Cart)示例项目中的controller`controllers/index.js`：

{% highlight html %}
var Product = require('../models/productModel');

module.exports = function (server) {
    server.get('/', function (req, res) {
        Product.find(function (err, prods) {
            if (err) {
                console.log(err);
            }

            var model =
            {
                products: prods
            };

            res.render('index', model);
        });
    });
};
{% endhighlight %}

由于这是一个后端框架，用户的操作只能通过url路径体现。当用户访问`/`路径时，首页`index.html`对应的controller，也就是该`controllers/index.js`收到请求，它调用Model层的Product模块的`find`方法请求数据，并将或得到的数据交给`index`模板进行重新渲染，产生的页面返回给用户。

为了和flux做比较，在这里我们要强调几点：

- 通常View和Controller的关系是一一对应的，比如首页index.html有自己的controller`controllers/index.js`，查询页面search.html有自己的controller`controllers/search.js`。从下面这段[angular的路由代码](https://docs.angularjs.org/tutorial/step_07#the-app-module)就是很典型的示例：

{% highlight html %}
phonecatApp.config(['$routeProvider',
  	function($routeProvider) {
    	$routeProvider.
      		when('/phones', {
        		templateUrl: 'partials/phone-list.html',
        		controller: 'PhoneListCtrl'
      		}).
      		when('/phones/:phoneId', {
        		templateUrl: 'partials/phone-detail.html',
        		controller: 'PhoneDetailCtrl'
      		}).
      		otherwise({
        		redirectTo: '/phones'
      		});
}]);
{% endhighlight %}

- controller是有业务逻辑的。虽然在MVC中我们强调"fat model, skinny controller"(业务逻辑应尽量放在Model层，Controller只应该作为View与Model的接口)，但skinny并不代表none，controller中还是有与业务相关的逻辑来决定将如何转发用户的请求，最典型的决定是转发到哪个Model层。

- Model应该被更准确的称为Domain Model(领域模型)，它不代表具体的Class或者Object，也不是单纯的databse。而是一个“层”的概念：数据在Model里得到存储，Model提供方法操作数据(Model的行为)。所以Model代码可以有业务逻辑，甚至可以有数据的存储操作的底层服务代码。

- MVC中的数据流是双向的，模型通知视图数据已经更新，视图直接查询模型中的数据。

### MVC的局限

上小节单组MVC(View、Model、Controller是1:1:1的关系)只是一种理想状态。现实中的程序往往是**多视图**，**多模型**。更严重的是视图与模型之间还可以是**多对多**的关系。也就是说，单个视图的数据可以来自多个模型，单个模型更新是需要通知多个视图，用户在视图上的操作可以对多个模型造成影响。可以想象最致命的后果是，视图与模型之间相互更新的死循环。

这样一来，View与Model与Controller之间的关系就成一团乱麻了，如下两幅图所示：

![mvc-complex](../images/mvc-vs-flux/mvc-complex.png)
![mvc-diagram](../images/mvc-vs-flux/mvc-diagram.png)

如此的混乱会产生很多的问题，比如调试代码。假设在一个复杂的MVC的架构中，有多个controller可以修改model，而开发时model的数据产出并非如你所愿，则你很难判断出是哪个controller出的错，只能使用控制变量法进行调试。

在2014年Facebook举办的F8(Facebook Developer Conference)大会上其中的[Hacker Way: Rethinking Web App Development at Facebook](https://www.youtube.com/watch?v=nYkdrAPrdcw)单元里，Facebook的工程师Jing Chen对于MVC的评价是，MVC非常适合于小型应用，但是当许许多多的Model和与之对应的View被加入到一个系统中，情况就会变得如下图所示：

![flux-react-mvc](../images/mvc-vs-flux/flux-react-mvc.png)

需要注意的是，她想表达的意思其实和上述两幅图是相同的，但她在大会上演示的这幅图对MVC的架构描述是有欠缺的。她的这番言论和不准确的图片同时也在[Reddit上也引起了非常多的讨论](https://www.reddit.com/r/programming/comments/25nrb5/facebook_mvc_does_not_scale_use_flux_instead/)，甚至是负面的评价。最后她的回复如下

>Yeah, that was a tricky slide [the one with multiple models and views and bidirectional data flow], partly because there's not a lot of consensus for what MVC is exactly - lots of people have different ideas about what it is. What we're really arguing against is bi-directional data flow, where one change can loop back and have cascading effects.

她承认演示中的图片确实投机取巧了。但其实大部分人对MVC的见解也并不相同，它们真正想表达的是这种双向的数据流架构会产生一定的负面效应。

## Flux

一个简单的flux流程图如下所示：

![flux-simple](../images/mvc-vs-flux/flux-simple.png)

参照上面的图示，我们首先总结一下，flux架构下一共有四类模块角色，按照交互顺序依次是：

- Component/View: 你可以把组件(Component)理解为View与Controller的结合，它既展现数据，同时也处理用户的交互请求。不同于MVC的Controller直接调用模型层业务逻辑处理接口，flux上来自用户的操作或者请求最终会映射为对应的Action，交由Action进行下一步处理。另一点需要注意的是View同时也监听着Store中数据的更改事件，一旦发生更改则重新请求数据。

- Action：描述组件触发的操作，包括名称和数据，比如`{ 'actionType': 'delete', 'data': item}`

- Dispatcher: flux的中央枢纽(central hub)，所有的Action都会交由Dispatcher进行处理。Dispatcher在接收到Action之后，调用Store注册在Action上的回调函数。需要注意与MVC中Controller不同的是，Dispatcher是不包含业务逻辑的，它机械的像一座桥，一个路由器，所以它能被别的程序复用当然也能被别的Dispatcher替换。

- Store：包含程序的数据与业务逻辑。和MVC的Model比较，Store有一些不易被察觉但又非常重要的差异：MVC中的每一个model即对应着一个领域模型；而flux中的一个Store自己并不是一个领域模型，而是可能包含多个模型。**最重要的是**，只有store自己知道如何修改数据，它并不对外直接提供操作数据的接口(但是提供查询数据的接口)，action和dispatcher没法操作store.

一个简单的flux流程我们可以这么描述：用户在View上的操作最终会映射为一类Action，Action传递给Dispatcher，再由Dispatcher执行注册在指定Action上的回调函数。最终完成对Store的操作。如果Store中的数据发生了更改，则触发数据更改的事件，View监听着这些时间，并对这些事件做出反应（比如重新查询数据）。

当有多个Store和View被添加后，复杂的flux流程图如下图所示

![flux-complex](../images/mvc-vs-flux/flux-complex.png)

如果上图还是让你感觉到复杂的话，我们继续抽象flux流程如下：

![flux-complex-abstract](../images/mvc-vs-flux/flux-abstract.png)

由此可见即使是复杂的flux应用，它的数据流和程序的运作过程仍然是清晰可辨的。

### Flux代码

最后这一小节，是用代码来演示flux的简易实现。如果你阅读本文的目的只是想对flux原理稍加了解，则可以略过这小节内容。

#### View

我们从最简单的场景出发，假设页面上只有一个按钮，我们通过这个按钮向store里添加一条数据。这里视图我们通过Reactjs实现：

{% highlight html %}
var View = React.createClass({  
    addNewItem: function (event) {
        Dispatcher.dispatch({
          action: 'add_item',
          data: {date: +new Date}        
        });
    },
    render: function(){
        return (
            <button onClick={this.addNewItem}>Add Item</button>
        )
    }
});
{% endhighlight %}

在按钮的点击事件中我们触发了`add_item`事件。只不过触发事件是直接通过调用`Dispatcher`来实现。

#### Actions

在上面的视图代码中，我们直接调用了Dispatcher的方法。但这样的代码耦合太强了。View其实无需感知Dispatcher，这里我们更是直接把Dispatcher的细节暴露给了View，同时action也没有被抽象出来。

接下来我们把Action抽象出来

{% highlight html %}
var Actions = {
  add: function (item) {
      Dispatcher.dispatch({
          action: 'add_item',
          data: item        
      });   
  }
}
{% endhighlight %}

此时的View也要修改为：

{% highlight html %}
var View = React.createClass({  
    addNewItem: function (event) {
        Actions.add({
            date: +new Date
        });
    },
    render: function(){
        return (
            <button onClick={this.addNewItem}>Add Item</button>
        )
    }
});
{% endhighlight %}

#### Store

Store负责存储并更新数据，它需要监听Dispatcher上触发的action并做出响应：

{% highlight html %}
var Store = {
    items: []
}

Dispatcher.register(function(payload) {
    switch(payload.action) {
        case 'add_item':
            // 当事件名为“添加”时，向仓库里添加数据
            Store.items.push(payload.data);
            // 同时触发“数据已更改”的事件
            Store.triggerEvent('change');
            break;
    }
}); 
{% endhighlight %}

当Store更新完数据之后，它还需要触发一个数据更新的事件，以告知那些关注这些数据的人。如果我们的视图需要在数据更改后时时更新数据，则还需要在Store注册数据更改事件的回调函数

{% highlight html %}
var View = React.createClass({
    update: function () {
        // TODO
    }, 
    componentDidMount: function() {  
        Store.bind('change', this.update);
    },  
    addNewItem: function (event) {
        Actions.add({
            date: +new Date
        });
    },
    render: function(){
        return (
            <button onClick={this.addNewItem}>Add Item</button>
        )
    }
});
{% endhighlight %}

参考文章：

[ReactJS For Stupid People](http://blog.andrewray.me/reactjs-for-stupid-people/)
[Flux For Stupid People](http://blog.andrewray.me/flux-for-stupid-people/)
[What the Flux? (On Flux, DDD, and CQRS)](http://jaysoo.ca/2015/02/06/what-the-flux/)
[Facebook: MVC Does Not Scale, Use Flux Instead [Updated]](https://www.infoq.com/news/2014/05/facebook-mvc-flux)
[What is Flux?](http://fluxxor.com/what-is-flux.html)
[Flux vs. MVC (Design Patterns)](https://medium.com/hacking-and-gonzo/flux-vs-mvc-design-patterns-57b28c0f71b7#.2uqupqv13)










