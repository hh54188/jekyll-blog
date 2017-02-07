在阅读这篇文章之前你要对前端的Flux架构和Redux架构有所了解，如果还不清楚它们是什么，请通过百度查询一些科普资料了解。因为这篇文章就是谈Flux架构和Redux架构的起源和背后的设计思想。

Flux和CQRS很像，Redux与CQRS很像，Flux也与Redux很像。我不确定Flux是否受到了CRQS的启发，但Redux作者在Redux.js官方文档的[Motivation](http://redux.js.org/docs/introduction/Motivation.html)一章里，在最后一段很明确的总结到：

>**Following in the steps of Flux, CQRS, and Event Sourcing**, Redux attempts to make state mutations predictable by imposing certain restrictions on how and when updates can happen.

我们已经知道了什么是Flux，但关于CQRS和Event Sourcing或许大部分人并不了解。其实DDD/CQRS/ES是形影不离的三个概念，通常在谈论其一时也会覆盖其二。在这篇文章里都会统一做一次扫盲，并看看Redux或者Flux是如何借鉴它们的。

##  Model & ORM(Object-Relational Mapping)

虽然Model和ORM不是这篇的主角，但它们是会涉及到的两个概念，所以提前进行介绍。

Model，译为模型，是对业务逻辑中涉及到的概念的抽象。比如我们需要开发一个“客户管理系统”，那么“客户”概念就可以抽象为一个模型，有关客户的种种行为，例如创建客户，注销客户，查看客户等操作都可以在模型中得以体现，也就是业务逻辑封装在模型中。MVC中的Model即是如此，例如在系统界面上查看客户信息，就需要调用客户模型的查看方法，用户信息视图的Controller会这么写：

```
var CustomInfoController = function (req, res) {
	var customInfo = CustomModel.showInfoById(req.body.id);
}
```

模型和它对应的数据之间的关系却不简单，我们有“客户”这个模型或者说是概念，但真正关于客户的数据或许存放在多个数据库表中，甚至很跨多个物理服务器，或者`showInfoById`这个方法使用了事务，使用了存储过程。也就是说，通常我们在使用“模型”这个字眼时，我们不考虑它的底层是如何存储，表如何设计，接口是如何实现的。模型是一个工具，它告诉我们系统由什么组成，是如何工作的，还有不同部分之间如何通信，依赖，协作等信息。这一类Model可以称之为Business Model或者Domain Model，但不一定是Object Model。Object Model意味着每一个模型都可以以面向对象编程中类的形式体现出来，但在DDD(Domain-driven design，另一个与CQRS相关的概念，放在最后介绍)的概念中，不是每一个模型都适合用类去表达（虽然代码实现中我们或许别无选择）。

另一种Model我们可以称之为Data Model，如果说Domain Model是对现实概念的映射，那么Data Model则是对数据库的映射。这种映射可以用一个更专业的词汇体现出来，就是ORM(Object-Relational Mapping)，即把数据库的表结构(Schema)映射为面向对象中的类。

当我们想通过数据库查询用户信息时，代码通常是这么写的：

```
// 创建数据库连接
var connection = mysqljs.createConnection(mysqlStr);
// 执行查询语句
connection.query('SELECT * FROM custom JOIN (address, avatar) ON (cutsom.id = address.id AND address.id = avatar.id) WHERE custom.id=' + userId, function (err, results) {
	
})
```
我们必须要自己书写数据库查询语句，直接与数据库打交道。

而如果通过ORM将数据库映射为对象之后，我们操作的就是对象，而不再直接和数据库打交道了，上面的语句就可以改为：

```
Custom.get({
	id: userId
});
```

## CQRS

CQRS全称为Command Query Responsibility Segregation，顾名思义“命令与查询职责分离”。“职责分离”我们理解，但怎么区分“命令”与“查询”，他们的职责又分别是什么？

命令与查询的根本区别在于，是否改变数据的状态。例如增、删、改操作即归属于“命令”，因为这些操作会导致数据被修改；而查询操作只求返回结果，并不修改数据，所以归属于“查询”（查询归属于“查询”，好吧，听上去像废话）。另一个区别在于，“命令”操作不需要返回值（当然我们在编码时需要有返回来告诉我们修改是否成功），而“查询”需要。

简单来说，CQRS作用在于把数据的读和写分离。读写操当然是隔离的，这里的分离相对的是传统编程中一视同仁的编写CRUD(Create, Read, Update, Delete，增删改查)接口代码。CQRS主张对“读”和“写”的接口做不同的设计，编码和优化。而之所以要做这样的分离，原因有以下两点：

1. 在许多业务场景中，数据的读和写的次数是不平衡，可能上千次的读操作才对应一次写操作，比如机票余票信息的查询和更新。所以把读和写操作分开能够有针对性的分别优化它们。；例如提高程序的scalability，scalability意味着我们能够在部署程序时，给读操作和写操作部署不同数量的线上实例来满足实际的需求。

2. 通常来说，“写”操作比“读”操作更加复杂，无论是业务逻辑方面还是技术方面。例如写操作首先要验证数据的合法性和完整性，存储中会涉及事务，存储过程，同时还要考虑数据冗余，数据同步；而读操作则完全没有类似的要求。把读和写分离相当于隔离了复杂操作，分离便于我们更好的独立维护它们，

至于CQRS的实现，则参考下图：

![CQRS](./images/CQRS.png)

图正如上文描述所说，所有的写操作请求都转发给write side, 而视图则从read side查询数据。从这里你可以看到Flux架构的影子：当数据在write side发生更改时，一个更新事件会被推送到read side，通过绑定事件的回调，read side得知数据已更新，可以选择是否重新读取数据。下面这张Flux流程图或许能够帮你回忆起Flux：

![flux-data-flow](./images/flux-data-flow.png)

这里我们要辨别一点Flux与CQRS流程图中的不同点。在CQRS中，write side和read side分属于两个不同的domain model，各自的逻辑封装和隔离在各自的Model中。而在Flux里，业务逻辑都统一封装在Store中。

另有一点关于CQRS流程图中非常重要的是，请求到了Model里终究还是要转化为对数据的操作，既然读和写的业务逻辑都独立开了，那么被读和被写的数据也需要隔离开吗？独立的数据库当然会有更好的性能，但随之而来的是数据延迟和数据同步的问题。这些都是关于技术实现，就不深究了。

更多关于CQRS的深入概念，会放在本文最后一节关于Domain-driven Design中。

## Event Sourcing

在介绍Event Sourcing之前，我们先好好了解一下什么是Event。在上一篇关于Event Bus的文章中，我们把Event与Command做了比较，总结出了事件(Event)的几个特征。现在请让我们继续补充完整这些特征：

- 事件发生在过去。比如“用户把商品添加进购物车”，就是一件已经发生的事情
- 事件是不可以更改的。因为它已经发生了，你没法更改或者撤销
- 事件是单向的消息。事件的发布源只有可能是一个，而事件的订阅者则可以有很多
- 事件发布时会附上与事件有关的信息。例如“ID为123用户的把ID为456的商品添加进了购物车”
- 最后，事件在Event Sourcing的场景中，一定是用于表达业务目的。例如上面的“ID为123用户的把ID为456的商品添加进了购物车”，意味着我们要检查库存，更新用户订单数据；而并非表达一般的技术事件，比如点击动作，组件通信等。

数据存储大致可以划分为两种方式，一种是直接存储数据结果，例如商品的库存，我们只关心还剩下多少件，所以存储结果是一个数值；另一种是存储每一次的更改，好比记账，我们要存储每次支出多少，每次收入多少，所以记录的是一连串数值。这种情况下我们不太在意最终结果，但是当需要最终结果时，只要把每次的差异累加即可。Event Sourcing就是指后面这种存储方式。用专业术语描述就是：Event Sourcing是一种通过记录“数据发生改变的历史事件”，来持久化当前状态的方式。

为什么我们要使用Event Sourcing？我们可以找到以下优点：

- 高性能：事件是不可更改的，存储的时候并且只做插入操作，也可以设计成独立，简单的对象。所以存储事件的成本较低额且效率较高，扩展起来也非常方便。
- 简化存储：事件用于描述系统内发生的事情。我们可以考虑用事件存储代替复杂的关系存储。
- 溯源: 正因为事件是不可更改的，并且记录了所有系统内发生的事情，我们能用它来跟踪问题，重现错误，甚至做备份和还原
- 被其他系统使用：上一篇文章说到事件可以作为不同服务间的通信方式，所以事件也可以被推送到自他的系统中被二次利用
- 附加价值：我们能把事件历史作为大数据的数据源，从中统计出有价值的信息

但最终还是要回归理性，毕竟以结果为导向的数据存储和以过程为导向的数据存储适用的业务场景是不同的。Event Sourcing只是为我们将来设计系统时多提供了一种可能性，最终还是需要具体问题具体分析。

说到这里，你一定会想起Redux。Redux强调的就是应用的“状态”，状态之间是独立的，这与Event Sourcing中的事件的独立和序列化不谋而合。


## DDD




[What's the difference between data model and object model?](http://stackoverflow.com/questions/2446002/whats-the-difference-between-data-model-and-object-model)
[What is an ORM and where can I learn more about it?](http://stackoverflow.com/questions/1279613/what-is-an-orm-and-where-can-i-learn-more-about-it)
[What is an Object-Relational Mapping Framework?](http://stackoverflow.com/questions/1152299/what-is-an-object-relational-mapping-framework)
[Reference 3: Introducing Event Sourcing](https://msdn.microsoft.com/en-us/library/jj591559.aspx)