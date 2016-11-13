##  Model & ORM(Object-Relational Mapping)

虽然Model和ORM不是这篇的主角，但它们是会涉及到的两个概念，所以提前进行介绍。

Model，译为模型，是对业务逻辑中涉及到的概念的抽象。比如我们需要开发一个“客户管理系统”，那么“客户”概念就可以抽象为一个模型，有关客户的种种行为，例如创建客户，注销客户，查看客户都可以在模型中得以体现,也就是业务逻辑封装在模型中。MVC中的Model即是如此，例如在系统界面上查看客户信息，就需要调用客户模型的查看方法，用户信息视图的Controller会这么写：

```
var CustomInfoController = function (req, res) {
	var customInfo = CustomModel.showInfoById(req.body.id);
}
```

模型和它对应的数据之间的关系却不简单，我们有“客户”这个模型或者说是概念，但真正关于客户的数据或许存放在多个数据库表中，甚至很跨多个物理服务器，或者`showInfoById`这个方法使用了事务，使用了存储过程。也就是说，通常我们在使用“模型”这个字眼时，我们不考虑它的底层是如何存储的，表示如何设计的，接口是如何实现的。模型是一个工具，它告诉我们系统由什么组成，是如何工作的，还有不同部分之间如何通信，依赖，协作等信息。这一类Model可以称之为Business Model, Domain Model，但不一定是Object Model。Object Model意味着每一个模型都可以以面向对象编程中类的形式体现出来，但在DDD(Domain-driven design，另一个与CQRS相关的概念，放在最后介绍)的概念中，不是每一个模型都适合用类去表达（虽然代码实现中我们或许别无选择）。

另一种Model我们可以称之为Data Model，如果说Domain Model是对现实概念的映射，那么Data Model则是对数据库的映射。这种映射可以用一个更专业的词汇体现出来，就是ORM(Object-Relational Mapping)，即把数据库的表结构(Schema)映射为面向对象中的类。

当我们想通过数据库查询用户信息时，原来代码通常是这么写的：

```
var connection = mysqljs.createConnection(mysqlStr);
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

CQRS全称为Command Query Responsibility Segregation，顾名思义“命令与查询职责分离”。“职责分离”我们理解，但怎么划分“命令”与“查询”，他们的职责又分别是什么？

命令与查询的根本区别在于，是否改变数据的状态。例如增、删、改操作即归属于“命令”。查询操作只求返回结果，并不修改数据，所以归属于“查询”（查询归属于“查询”，好吧，听上去像废话）。另一个区别在于，“命令”操作不需要返回值（当然我们在编码时需要有返回来告诉我们修改是否成功），而“查询”需要。

简单来说，CQRS作用在于把数据的读和写分离。读写操当然是隔离的，这里的分离相对的是传统编程中一视同仁的编写CRUD(Create, Read, Update, Delete，增删改查)接口代码。CQRS主张对“读”和“写”的接口做不同的设计，编码和优化。而之所以要做这样的分离，原因有以下两点：

1. 在许多业务场景中，数据的读和写的次数是不平衡，可能上千次的读操作才对应一次写操作，比如机票余票信息的查询和更新。所以把读和写操作分开能够有针对性的分别优化它们。；例如提高程序的scalability，scalability意味着我们能够在部署程序时，给读操作和写操作部署不同数量的线上实例来满足实际的需求。

2. 通常来说，“写”操作比“读”操作更加复杂，无论是业务逻辑方面还是技术方面。例如写操作首先要验证数据的合法性和完整性，存储中会涉及事务，存储过程，同时还要考虑数据冗余，数据同步；而读操作则完全没有类似的要求。把读和写分离相当于隔离了复杂操作，分离便于我们更好的独立维护它们，

[What's the difference between data model and object model?](http://stackoverflow.com/questions/2446002/whats-the-difference-between-data-model-and-object-model)
[What is an ORM and where can I learn more about it?](http://stackoverflow.com/questions/1279613/what-is-an-orm-and-where-can-i-learn-more-about-it)
[What is an Object-Relational Mapping Framework?](http://stackoverflow.com/questions/1152299/what-is-an-object-relational-mapping-framework)
