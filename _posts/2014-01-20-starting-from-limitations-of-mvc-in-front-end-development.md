---
layout: post
title: 从MVC模式在前端开发中的局限性谈起
description: "测试"
modified: 2013-1-20
tags: [javascript, front-end, mvc]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

## 有名无实的Router

### ASP.NET MVC

如果你还没有接触过后端的MVC框架的话，不妨先看看下面这段ASP.NET MVC代码并且了解一下后端MVC的工作原理。它摘自ASP.NET MVC教程中非常著名的项目[MVC Music Store](http://mvcmusicstore.codeplex.com/)一段Controller组件代码：

{% highlight javascript %}
public class StoreManagerController : Controller
{
    private MusicStoreEntities db = new MusicStoreEntities();

    // GET: /StoreManager/

    public ViewResult Index()
    {
        var albums = db.Albums.Include(a => a.Genre).Include(a => a.Artist);
        return View(albums.ToList());
    }

    // GET: /StoreManager/Details/5

    public ViewResult Details(int id)
    {
        Album album = db.Albums.Find(id);
        return View(album);
    }
}
{% endhighlight %}
我们知道Controller的职责之一是负责响应用户在视图上的行为，而具体每个行为应如何进行响应，需要落实到Controller具体的方法上，这个方法我们可以称之为action。上面代码中的两个公开方法`Index()`与`Details()`就是两个action。它们都属于`StoreManager`这个Controller。如果你有使用过前端的Ember.js的话，应该对这两个概念非常熟悉。

但问题来了，如何将用户在视图上的行为，与响应行为的方法action关联起来？甚至与Controller关联起来？ URL便是方法之一。上面代码每个action上的注释便代表这个这个action对应的URL。也就是说，当用户点击该URL时，框架中的Router服务便能通过URL解析出应该调用哪个Controller及该Controller下的哪一个action进行响应，以上面的例子为例，可以知道URL的规则为`{controller}/{action}/{id}`。

那么响应的结果应该是什么呢？从上面的代码`ViewResult`和`return View()`两处可以看出，两个action返回的都是新的视图。

举一个最熟悉的现有MVC站点的例子便是[github](https://github.com/)。你会发现你在github网站上的每一处点击都有唯一的URL对应，每一次交互的结果都是服务器返回新的页面。它使用javascript非常少，比如当你选择编辑时，它也会跳转到一个新的页面，而非在当前页弹出一个编辑框。

为什么首先要聊这么多的服务器端MVC框架的特性。因为接下来回过头来看前端的MVC框架时，你会发现有非常多的差异之处。

### Javascript MVC

从上面可以看出，服务器端的MVC框架服务的是整个站点，它依靠不断的返回页面来响应用户请求，因此Router服务至关重要。而使用MVC框架的前端页面，大多数是Single Page Application，甚至还不如单页面，只是页面上的某一个组件，比如一个Slide。因此将用户的行为转化为URL是不现实。你或许会说的确无法生产新的页面，那么降低页面粒度如何呢？也就是说在服务器端一个URL映射的是一个页面，那么我们将URL映射为页面的某个区域或者功能呢？

比如以下面这段Backbone.js的TodoList应用Router为例：

{% highlight javascript %}
var TodoRouter = Backbone.Router.extend({
    routes: {
        'todo/add': 'add', // 新增项
        'todo/edit/:id': 'edit', // 编辑项
        'todo/remove/:id': 'remove', // 删除项

        'filter/completed': 'filterCompleted', // 过滤出已完成
        'filter/uncompleted': 'filterUncompleted' // 过滤出未完成
    }
    // Todo
});
{% endhighlight %}
如果依照这样Route规划，我们希望当用户输入`http://example.com#todo/add`时，我们弹出的是一个新增输入框；而当用户输入`http://example.com#todo/edit/123456`页面出现编辑id为123456的这条记事的编辑框。这样我们便将URL映射的页面粒度降低为输入框粒度。

但是这样会引起另一个问题，注意上面route的差别：`todo/`域名下操作的是单条的记录，而`filter/`域名下操作的是对列表进行筛选。所以还不得不考虑一种情况，如果用户想在筛选的情况下可否对每一项进行操作？如果允许的话，参考排列组合，route是否需要新增为2 x 3 = 6项？如新增`http://example.com#filter/completed/todo/add`这样的路由。

这样的设定明显是不合理的。之所以会产生这样的问题是因为对后端而言URL与页面是一一对应的关系。而如果降低页面粒度的话，无法将页面功能与URL对应起来，或者说如果想让URl覆盖单一页面上的所有功能的成本太高了。

## 前端的解决之道

当然Route在前端MVC框架中并非武功尽废，我们仍然可以保留这样的机制，但是仅用于高粒度的操作，比如上面例子中的筛选功能。

其实Route仅仅是桥梁，将用户的行为与响应用户行为的方法联系起来。Route机制已经在前端行不通了，问题便是寻求另一种联系的机制。

### Observer Pattern + MV

Google程序员[Addy Osmani](http://addyosmani.com/blog/)(同时也是[Learning JavaScript Design Patterns](http://addyosmani.com/resources/essentialjsdesignpatterns/book/)的作者)有一个非常著名的项目——[Todomvc](http://todomvc.com/)。他用所有的前端MVC框架将Todo List这个app重写了一遍。我们看看他的解决这样的问题的。

#### Backbone

以Backbone.js为例，为了方便说明，在他的基础上我再次进行了简化，简化到只有50行代码，只保留了新增和删除方法。实际演示效果在[这里](http://jsfiddle.net/JPL94/7/)：

{% highlight javascript %}
var Todo = Backbone.Model.extend();

var Todos = Backbone.Collection.extend({
    model: Todo
});

var todos = new Todos();

var ItemView = Backbone.View.extend({
    tagName: "li",
    template: _.template($("#item-template").html()),
    render: function () {
        this.$el.html(this.template(this.model.toJSON()));
        return this;
    },
    initialize: function () {
        this.listenTo(this.model, 'remove', this.remove);
    },
    events: {
        "click .delete": "clear"
    },
    clear: function () {
        todos.remove(this.model);
    }
});

var AppView = Backbone.View.extend({
    el: $("body"),
    initialize: function () {
        this.listenTo(todos, 'add', this.addOne);
    },  
    addOne: function(todo) {
        var view = new ItemView({
            model: todo
        });
        this.$("#list").append(view.render().el);
    },    
    events: {
        "click #create": "create"
    },
    create: function () {
        var model = new Todo({
            title: this.$("#input").val()
        });
        todos.add(model);
    }
})

var app = new AppView();
{% endhighlight %}
这个应用有两个视图，全局视图`AppView`和单个项目视图`ItemView`，还有一个相当于Model层的Collection`todos`。它没有显式的将Controller层独立出来。你可能会说Backbone不像Angular和Ember，它本就没有Controller的设定，我不这么认为，Backbone里Router就是Controller与Router的结合体。比如它可以加入一个Router这么干：
{% highlight javascript %}
var TodoRouter = Backbone.Router.extend({
    routes: {
        'todo/add': 'add'
    },
    add: function () {
        // do something
    }
});
{% endhighlight %}
翻译过来是：`#todo/add`对应的action是`add`——如果放在服务端的话这不就是把用户的行为(URL)与action联系起来了吗。

回到这个应用，它没有将Controller显式的独立出来并不代表Controller不存在。**我们需要的不是一个叫做Controller的东西，而是一些能够响应用户行为的方法**。那么它把Controller放在哪了？它使用了的是最原始的解决办法，放在用户行为事件的回调函数中，比如这段代码：

{% highlight javascript %}
events: {
    "click #create": "create"
},
create: function () {
    var model = new Todo({
        title: this.$("#input").val()
    });
    todos.add(model);
}
{% endhighlight %}
虽然这种方法比较原始，但不应该认为它是一种低级的解决方案。当然接下来我们会在Emberjs和Knockoutjs看到一些其他的解决方法，从职责上来说，他们的目的是一致的。

回想在文章开头介绍的ASP.NET MVC的一个action，它做了哪些事情：
{% highlight javascript %}
    public ViewResult Details(int id)
    {   
        // 执行了Model层的查找(query)操作
        Album album = db.Albums.Find(id);
        // 将查找返回的结果传递给视图(View)
        return View(album);
    }
{% endhighlight %}
用户的操作会涉及数据和视图更改，一般来说视图会被动一些，Model的修改可以通知视图的更新，当然视图也可以向Model请求数据（注意从这里可以看出Model与View可以直接通信而非通过Controller）。上面的代码很好的说明了这一点，并且视图的更新和数据的操作都是在同一个action里完成。

但是前端MVC框架却不这么做。

因为在前后端代码与视图的关系是不同的，后端代码生产出页面交给浏览器之后，基本上就不关心也无法关心页面了；而在前端不同，虽然前端也有“生产”这么一说（比如说使用模板引擎），但前端代码时刻都可以对Html代码和DOM元素进行监听和操作。

上面的Todo里Backbone是这么做的：

{% highlight javascript %}
var AppView = Backbone.View.extend({
    el: $("body"),
    initialize: function () {
        // 在初始化该视图时(执行当前initialize函数)，
        // 它会对这个应用唯一的Model层，就是Collection`todos`进行监听；

        // 更准确来说是对它的新增操作进行监听
        // 一旦发现数据有新增操作，那么更新视图(调用addOne方法)
        this.listenTo(todos, 'add', this.addOne);
    },  
    addOne: function(todo) {
        var view = new ItemView({
            model: todo
        });
        this.$("#list").append(view.render().el);
    },
{% endhighlight %}
这个时候你再看看这个AppView的代码会觉得有一些绕，因为我们明明可以在create方法中，在新增数据之后立即调用更新视图的方法:
{% highlight javascript %}
create: function () {
    var model = new Todo({
        title: this.$("#input").val()
    });
    todos.add(model);
    // 更新视图
    // this.addOne(model)
}
{% endhighlight %}
但我想这么做的原因无非是为了解耦。考虑今后可能增加多个新增数据的入口(如create1, create2, create3)，在每一个新增方法中调用更新视图的方法会增加代码的维护量，那么就不如采用这种观察者模式(Observer pattern)(或者采用fire事件机制也行)，只在指定处更新视图。

ItemView采用的也是同样机制，就不在这里赘述了。

值得一提的是，todomvc Backbone的原版本中仍然保留了Router服务：

{% highlight javascript %}
var TodoRouter = Backbone.Router.extend({
    routes: {
        '*filter': 'setFilter'
    },

    setFilter: function (param) {
        // Set the current filter to be used
        app.TodoFilter = param || '';

        // Trigger a collection filter event, causing hiding/unhiding
        // of Todo view items
        app.todos.trigger('filter');
    }
});
{% endhighlight %}
如果有兴趣浏览一下原版代码，你会发现这只是作为更新视图的触发器而已。

#### Ember.js

不仅仅Backbone采用这样的机制，我们也可以看看todomvc中简化过后的Emberjs代码，采用的也是同样机制，线上[DEMO](http://jsfiddle.net/GESP3/)：


**Html:**
{% highlight javascript %}
<script type="text/x-handlebars" data-template-name="index">
{{ input type="text" value=title }}
<button {{action "create"}}>Add</button>
<ul>
{{#each item in content itemController="todo"}}
    <li>
        <span>{{item.title}}</span>
        <button {{action "remove"}}>delete</button>
    </li>
{{/each}}
</ul>
</script>
{% endhighlight %}

**Javascript:**
{% highlight javascript %}
App = Ember.Application.create({});

App.ApplicationAdapter = DS.LSAdapter.extend();

App.Todo = DS.Model.extend({
    title: DS.attr("string")
})

App.IndexRoute = Ember.Route.extend({
    setupController: function(controller) {
        var todos = this.store.find('todo');
        controller.set("content", todos);
    }
});

App.TodoController = Ember.ObjectController.extend({
    actions: {
        remove: function () {
            var todo = this.get('model');
            todo.deleteRecord();
            todo.save();
        }
    }
})

App.IndexController = Ember.ArrayController.extend({
    actions: {
        create: function () {
            var title = this.get("title");
            var todo = this.store.createRecord('todo', {
                title: title
            });
            todo.save();
        }
    }
});
{% endhighlight %}
这一段代码不做详解，与前一个Backbone的机制是一致的，需要注意的是：

1. 如果你觉得Backbone中将用户行为与方法联系起来的做法比较原始的话，不妨看看Emberjs的解决方法，它直接写进需要编译的模板中：

{% highlight html %}
// 指定调用IndexController中的create的方法(action)响应按钮的点击事件
<button {{action "create"}}>Add</button>
......

// 指定TodoController中的remove的方法(action)响应
<button {{action "remove"}}>delete</button>

{% endhighlight %}


2. 在新增事项的create方法中，它做的也仅仅是对Model层进行数据操作，

{% highlight javascript %}
create: function () {
    var title = this.get("title");
    var todo = this.store.createRecord('todo', {
        title: title
    });
    todo.save();
}
{% endhighlight %}
但你不用调用任何的视图方法，视图已经随数据更新了。这不也是一种Observer模式嘛。

3. 值得一提的是Ember.js中的Router有一些特别，除了一般路由拥有的定义URL规则之外，它还负责向对应的Controller提供model层数据。



### MVP(Passive View)

Addy的解决方案是Observer Pattern。但我更提倡另一种解决方案，MVP(Model-View-Presenter)模式中的Passive View模式。

为什么会有MVC，MVP，甚至MVVM？三个模式其实主要围绕的是两个问题：

1. 一是Model与View之间的通信问题, 完全隔离的?单向通信还是双向通信?
2. 二是M-V-XX中的"XX"需要完成哪些功能, 简单流程调度? 还是复杂规则处理? 

> MVP模式将用户的交互逻辑的处理流程定义在Presenter层中，但是具体的实现并不是完全在Presenter中。View和Presenter采用单向的沟通方式。View单纯地将用户的交互请求汇报给Presenter；Presenter接收到请求之后，整合相应的资源、执行相应的处理逻辑。对处理流程的某一个步骤，如果设置到业务逻辑和数据模型，则调用Model，如果涉及到对视图的更新，还会调用View。Presenter和View接口都应该只包含返回类型为void的方法即可

同时参考Kjell-Sverre Jerijærvi提出的MVP的设计原则[Design Rules for Model-View-Presenter](http://kjellsj.blogspot.com/2008/05/design-rules-for-model-view-presenter.html)，可以找到MVP模式中一些非常重要的特征，例如以下几条：

- View不允许通过Presenter直接调用Model和Service，并且Presenter的方法应该是不具有返回值的；

- Presenter必须通过View接口的方式调用View

- 除了对View接口成员的实现外，View中的其他方法不应该是public的；

- View接口的成员应该仅限于方法，不应该包含属性；

- 所有的数据应用保持在Model中

既然Presenter对于View是相对透明的，View不能直接对Presenter进行操作（目的是实现Presenter和View之间的分离）。那么如何实现View与Presenter层之间的通信呢——通过注册事件，在Presenter上注册View的事件，并且这样以来数据的传递就不是View向Presenter去“拉(pull)”，而是Presenter“推(push)”给View层。

所以我们的目标非常的简单，分别定义三个层：

1. Model层：提供操作数据的接口

2. View层：提供更新视图的接口，当用户有行为发生时触发事件

3. Presenter层：定义用户的交互逻辑与流程，但所有与数据操作和视图的更新都通过调用Model层与View层的接口实现，注册View层事件。

实现代码如下：

{% highlight javascript %}
var Todo = Backbone.Model.extend({

});

var Todos = Backbone.Collection.extend({
    model: Todo
});

var ItemView = Backbone.View.extend({
    tagName: "li",
    template: _.template($("#item-template").html()),
    render: function (container) {
        this.$el.html(this.template(this.model.toJSON()));
        container.append(this.$el);
    },
    events: {
        "click .delete": "_clear"
    },
    _clear: function () {
        this.$el.trigger("delete", this);
    }
});

var AppView = Backbone.View.extend({
    el: $("body"),
    events: {
        "click #create": "_create"
    },
    _create: function () {
        var data = {
            title: this.$("#input").val()
        };

        this.$el.trigger("create", data);
    }   
})

var Presenter = Backbone.View.extend({
    el: $("body"),
    initialize: function () {

        this.appView = new AppView({
            container: $("#list")
        });
        this.todos = new Todos;
        var _this = this;

        this.$el.on("create", function (e, attrs) {
            var model = new Todo(attrs);
            _this.todos.add(model);
            var itemView = new ItemView({
                model: model
            })
            itemView.render($("#list"));
        });

        this.$el.on("delete", function (e, view) {
            _this.todos.remove(view.model);
            view.remove();
        });        
    }
})

var p = new Presenter;
{% endhighlight %}
[DEMO](http://jsfiddle.net/JPL94/8/)

让我们来看看它是否复合我们的要求：

- AppView不提供任何接口(所有的私有方法都以_开始)，只负责捕捉用户的行为。一旦用户点击新增，即向外广播“新增”事件`.trigger("create", data)`

- ItemView提供一个插入新视图的接口`render`与移除视图接口`remove`, 并且捕捉用户的删除行为，广播“删除”事件

- Presenter层捕获到事件后，执行一系列的流程，但皆以调用Model层与View层接口的方式执行，而非直接操纵实例属性。


上面的两个例子我们可以看到，介于前端的特殊性，在前端MVC框架中View层与Controller层中的action联系较为密切，例如把action注册到视图元素的事件回调中。虽然这样能够代替服务器端的路由方案，但代价是牺牲了不同层之间的低耦合性，并且不易测试。

MVP在MVC的基础上，进一步进行了解耦，把视图和数据抽象仅剩下接口，并且把业务逻辑全部归纳到Presenter层中。这样不仅能够把View单独拎出测试，还能提高这一套MVP组件的可复用性。如果我们想更换View层或者Model层的话，只要保证新层具有与原层相同的接口，而不用涉及到其他层的修改。

### MVVM

最后一个解决方案是MVVM(Model-View-ViewModel)，以Knockout.js为例，完成相同功能只需要非常少的代码：


**Html:**
{% highlight html %}
<input type="text" data-bind="value: title">
<button data-bind="click: create">Add</button>
<ul data-bind="foreach: todos">
    <li>
        <span data-bind="text: title"> </span>
        <button data-bind="click: $root.remove">delete</button>
    </li>
</ul>
{% endhighlight %}

**Javascript:**
{% highlight javascript %}
function Todo (title) {
    this.title = title;
}

var ViewModel = function() {
    
    var _this = this;
    this.todos = ko.observableArray([new Todo("test1"), new Todo("test2")]);
    this.title = ko.observable();

    this.create = function () {
        var title = this.title().trim();
        if (title) {
            _this.todos.push(new Todo(title));
        }
    }

    this.remove = function (todo) {
        _this.todos.remove(todo);
    } 
};
 
ko.applyBindings(new ViewModel());
{% endhighlight %}
[DEMO](http://jsfiddle.net/mfUVk/)

在MVP与MVC模式中，用于渲染视图的数据往往是action处理Model层返回的数据之后的结果。而MVVM中的ViewModel则是视图所需要渲染数据的直接映射，无需再经过其他服务的处理。

我不确定MVVM在WPF中(MVVM起源于微软Windows Presentation Foundation框架)是如何处理ViewModel层方法与数据的关系。但是从上面Knockoutjs的代码可以看出，ViewModel的代码是比较混乱，方法和属性都书写在一起。其实从html的绑定方式也可以看出，无论是数据还是action，都一视同仁的使用`data-bind`属性。

虽然Knockout的代码写起来很舒服(我们近乎只用关心Model层的数据操作)，但这样的代码无疑对代码的复用和维护提出了挑战。仅作参考，不作推荐。

参考文献：

- [Digesting JavaScript MVC – Pattern Abuse Or Evolution?](http://addyosmani.com/blog/digesting-javascript-mvc-pattern-abuse-or-evolution/)
- [JavaScript MVC](http://alistapart.com/article/javascript-mvc)
- [Single page apps in depth](http://singlepageappbook.com/)
- [Journey Through The JavaScript MVC Jungle](http://coding.smashingmagazine.com/2012/07/27/journey-through-the-javascript-mvc-jungle/)
- [大话MVP](http://www.cnblogs.com/artech/archive/2010/04/12/1710681.html)
- [谈谈关于MVP模式中V-P交互问题](http://www.cnblogs.com/artech/archive/2010/03/25/1696205.html)
- [从三层架构到MVC,MVP](http://www.cnblogs.com/daizhj/archive/2009/04/30/1447035.html)

