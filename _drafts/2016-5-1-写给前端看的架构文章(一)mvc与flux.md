

## 前言

在学习React.js的过程中，曾经最让我苦恼的事情是，我需要给自己一个使用这个框架的理由。因为随着学习经验的和工作经验的增长，你会发现类似的技术总是会此消彼长的出现，如果这只是另一个轮子怎么办？加之学习的成本、项目改造的成本甚至周围人来适应你的成本，一味的追逐最新最流行的技术并非是一件好事。

React.js组件化的思维方式确实让人耳目一新，但是还不够好。直到读懂背后懂得flux架构，直到把flux与mvc进行优劣比较，才发现React.js确实有可取之处。但flux架构并非新事物，如果你拥有后端开发背景的话，flux架构一定会让你联想到CQRS(Command-Query Responsibility Segregation)、EDA(Event-Driven Architecture)、DDD(Domain-Driven Design)等概念。至于这些概念具体是什么，和flux有什么关系，咱们以后再说。今天我们聊flux与mvc比较之下它的创新之处在哪。

如果你还完全没有接触过React.js，下一节会首先带你做简单了解。如果你已经有所了解，可以跳过下节继续阅读

## React.js简单入门

## MVC(Model-View-Controller)

### MVC简介

MVC架构讲程序划分为三个角色，从上到下依次为：
- View: 视图，用户数据展示，同时接受用户输入
- Contorller：响应用户的输入，对数据进行操作，
- Model：负责管理程序需要的数据，并且定义了操作数据的行为。

对于一个简单的MVC架构程序来说，其工作流程如下：

![mvc-simple](./images/mvc-vs-flux/mvc-simple.png)

从最右边的View开始，当用户在UI上进行操作之后，用户的操作被转发到了Controller上，Controller根据用户的操作对数据进行更新(准确来说是调用Model层的API)，数据更新之后自然视图View展现的内容也需要进行更新。Model层此时可以向所有关联的视图发出通知，收到通知的视图重新获取最新的数据。注意这最后一步Model与View的交互，大部分现有的MVC框架将其进行了封装，开发人员只要使用数据绑定即可。

如果上面的流程图还过于抽象的话，我们可以看一段MVC项目的代码，比如基于Nodejs的Kraken框架的[Shopping_Cart](https://github.com/lmarkus/Kraken_Example_Shopping_Cart)示例项目中的controller`controllers/index.js`：

```
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
```

由于这是一个后端框架，用户的操作只能通过url路径体现。当用户访问`/`路径时，首页`index.html`对应的controller，也就是该`controllers/index.js`收到请求，它调用Model层的Product模块的`find`方法请求数据，并将或得到的数据交给`index`模板进行重新渲染，产生的页面返回给用户。

为了和flux做比较，在这里我们要强调几点：

- 通常View和Controller的关系是一一对应的，比如首页index.html有自己的controller`controllers/index.js`，查询页面search.html有自己的controller`controllers/search.js`。从下面这段[angular的路由代码](https://docs.angularjs.org/tutorial/step_07#the-app-module)就是很典型的示例：

```
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
```

- controller是有业务逻辑的。虽然在MVC中我们强调"fat model, skinny controller"(业务逻辑应尽量放在Model层，Controller只应该作为View与Model的接口)，但skinny并不代表none，controller中还是有与业务相关的逻辑来决定将如何转发用户的请求，最典型的决定是转发到哪个Model层。

- Model应该被更准确的称为Domain Model(领域模型)，它不代表具体的Class或者Object，也不是单纯的databse。而是一个“层”的概念：数据在Model里得到存储，Model提供方法操作数据(Model的行为)。所以Model代码可以有业务逻辑，甚至可以有数据的存储操作的底层服务代码。

- MVC中的数据流是双向的，模型通知视图数据已经更新，视图直接查询模型中的数据。

### MVC的局限

上小节单组MVC(View、Model、Controller是1:1:1的关系)只是一种理想状态。现实中的程序往往是多视图，多模型。更严重的是视图与模型之间还可以是多对多的关系。也就是说，单个视图的数据可以来自多个模型，单个模型更新是需要通知多个视图，用户在视图上的操作可以对多个模型造成影响。可以想象最致命的后果是，视图与模型之间相互更新的死循环。

这样一来，View与Model与Controller之间的关系就成一团乱麻了，如下两幅图所示：

![mvc-complex](./images/mvc-vs-flux/mvc-complex.png)
![mvc-diagram](./images/mvc-vs-flux/mvc-diagram.png)

在2014年Facebook举办的F8(Facebook Developer Conference)大会上其中的[Hacker Way: Rethinking Web App Development at Facebook](https://www.youtube.com/watch?v=nYkdrAPrdcw)单元里，Facebook的工程师Jing Chen对于MVC的评价是，MVC非常适合于小型应用，但是当许许多多的Model和与之对应的View被加入到一个系统中，情况就会变得如下图所示：

![flux-react-mvc](./images/mvc-vs-flux/mvc-diagram.png)

需要注意的是，她想表达的意思其实和上述两幅图是相同的，但她在大会上演示的这幅图对MVC的架构描述是有欠缺的。她的这番言论和不准确的图片同时也在[Reddit上也引起了非常多的讨论](https://www.reddit.com/r/programming/comments/25nrb5/facebook_mvc_does_not_scale_use_flux_instead/)，甚至是负面的评价。最后她的回复如下

>Yeah, that was a tricky slide [the one with multiple models and views and bidirectional data flow], partly because there's not a lot of consensus for what MVC is exactly - lots of people have different ideas about what it is. What we're really arguing against is bi-directional data flow, where one change can loop back and have cascading effects.

她承认演示中的图片确实投机取巧了。但其实大部分人对MVC的见解也并不相同，它们真正想表达的是这种双向的数据流架构会产生一定的负面效应。

## Flux


