---
layout: post
title: 请停止使用new关键字
description: "Just about everything you'll need to style in the theme: headings, paragraphs, blockquotes, tables, code blocks, and more."
modified: 2013-11-22
tags: [javascript, front-end]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

##传统的实例化与继承

JavaScript中的new关键字可以实现实例化和继承的工作，但个人认为使用new关键字并非是最佳的实践，还可以有更友好一些的实现。本文将介绍使用new关键字有什么问题，然后介绍如何对与new相关联的一系列面向对象操作进行封装，以便提供更快捷的、更易让人理解的实现方式。


假设我们有两个类，`Class:function Class() {}`和`SubClass:function SubClass(){}`，SubClass需要继承自Class。传统方法一般是按如下步骤来组织和实现的：

- Class中**被继承的属性和方法**必须放在Class的prototype属性中
- SubClass中**自己的方法和属性**也必须放在自己prototype属性中
- SubClass的prototype对象的prototype(\__proto__)属性必须指向的Class的prototype

这样一来，由于prototype链的特性，SubClass的实例便能追溯到Class的方法，从而实现继承：
{% highlight javascript %}

    new SubClass()      Object.create(Class.prototype)
        |                    |
        V                    V
    SubClass.prototype ---> { }
                            { }.__proto__ ---> Class.prototype
{% endhighlight %}

举一个具体的例子：下面的代码中，我们做了以下几件事:

- 定义一个父类叫做Human
- 定义一个名为Man的子类继承自Human
- 子类继承父类的一切属性，并调用父类的构造函数，实例化这个子类

{% highlight javascript %}
// 构造函数/基类
function Human(name) {
    this.name = name;
}

/* 
    基类的方法保存在构造函数的prototype属性中
    便于子类的继承
*/
Human.prototype.say = function () {
    console.log("say");
}

/*
    道格拉斯的object方法（等同于object.create方法）
*/
function object(o) {
    var F = function () {};
    F.prototype = o;
    return new F();
}

// 子类构造函数
function Man(name, age) {
    // 调用父类的构造函数
    Human.call(this, name);
    // 自己的属性age
    this.age = age;
}

// 继承父类的方法
Man.prototype = object(Human.prototype);
Man.prototype.constructor = Man;

// 实例化子类
var man = new Man("Lee", 22);
console.log(man);
// 调用父类的say方法：
man.say();
{% endhighlight %}

[DEMO](http://jsfiddle.net/gP9g5/)

通过上面的代码可以总结出传统的实例化与继承的几个特点：

- 传统方法中的“类”一定是一个构造函数。
- 属性和方法绑定在prototype属性上，并借助prototype的特性实现继承。
- 通过new关键字来实例化一个对象。

为什么我会十分的肯定Object.create方法与道格拉斯的object方法是一致呢？因为在MDN上，object方法就是作为Object.create的一个Polyfill方案：

- [Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)
- [Douglas Crockford's object method](http://javascript.crockford.com/prototypal.html)

##new关键字的不足之处


在《Javascript语言精粹》（Javascript: The Good Parts）中，道格拉斯认为应该避免使用new关键字：

> If you forget to include the new prefix when calling a constructor function, then this will not be bound to the new object. Sadly, this will be bound to the global object, so instead of augmenting your new object, you will be clobbering global variables. That is really bad. There is no compile warning, and there is no runtime warning. (page 49)

大意是说在应该使用new的时候如果忘了new关键字，会引发一些问题。

当然了，你遗忘使用任何关键字都会引起一系列的问题。再退一步说，这个问题是[完全可以避免的](http://stackoverflow.com/questions/383402/is-javascript-s-new-keyword-considered-harmful#answer-383503)：

{% highlight javascript %}
function foo()
{   
   // 如果忘了使用关键字，这一步骤会悄悄帮你修复这个问题
   if ( !(this instanceof foo) )
      return new foo();

   // 构造函数的逻辑继续……
}
{% endhighlight %}
或者更通用的抛出异常即可

{% highlight javascript %}
function foo()
{
    if ( !(this instanceof arguments.callee) ) 
       throw new Error("Constructor called as a function");
}
{% endhighlight %}

又或者按照[John Resig的方案](http://ejohn.org/blog/simple-class-instantiation/)，准备一个makeClass工厂函数，把大部分的初始化功能放在一个init方法中，而非构造函数自己中：

{% highlight javascript %}
// makeClass - By John Resig (MIT Licensed)
function makeClass(){
  return function(args){
    if ( this instanceof arguments.callee ) {
      if ( typeof this.init == "function" )
        this.init.apply( this, args.callee ? args : arguments );
    } else
      return new arguments.callee( arguments );
  };
}
{% endhighlight %}


在我看来，new关键字不是一个好的实践的关键原因是：

> [...new is a remnant of the days where JavaScript accepted a Java like syntax for gaining “popularity”. And we were pushing it as a little brother to Java, as a complementary language like Visual Basic was to C++ in Microsoft’s language families at the time.](http://dailyjs.com/2010/05/24/history-of-javascript-1/)

道格拉斯将这个问题描述为：

> [This indirection was intended to make the language seem more familiar to classically trained programmers, but failed to do that, as we can see from the very low opinion Java programmers have of JavaScript. JavaScript’s constructor pattern did not appeal to the classical crowd. It also obscured JavaScript’s true prototypal nature. As a result, there are very few programmers who know how to use the language effectively.](http://javascript.crockford.com/prototypal.html)

简单来说，JavaScript是一种prototypical类型语言，在创建之初，是为了迎合市场的需要，让人们觉得它和Java是类似的，才引入了new关键字。Javascript本应通过它的Prototypical特性来实现实例化和继承，但new关键字让它变得不伦不类。


##把传统方法加以改造


既然new关键字不够友好，那么我们有两个办法可以解决这个问题：一是完全抛弃new关键字，二是把含有new关键字的操作封装起来，只向外提供友好的接口。下面将介绍第二种方法的实现思路，把传统方法加以改造。

我们开始构造一个最原始的基类`Class`（类似于JavaScript中的Object类），并且只向外提供两个接口：

- Class.extend 用于拓展子类
- Class.create 用于创建实例

{% highlight javascript %}
// 基类
function Class() {}

// 将extend和create置于prototype对象中，以便子类继承
Class.prototype.extend = function () {};
Class.prototype.create = function () {};

// 为了能在基类上直接以.extend的方式进行调用
Class.extend = function (props) {
    return this.prototype.extend.call(this, props);
}

{% endhighlight %}

extend和create的具体实现：

{% highlight javascript %}
Class.prototype.create = function (props) {
    /*
        create实际上是对new的封装；
        create返回的实例实际上就是new构造出的实例；
        this即指向调用当前create的构造函数；
    */
    var instance = new this();
    /*
        绑定该实例的属性
    */
    for (var name in props) {
        instance[name] = props[name];
    }
    return instance;
}

Class.prototype.extend = function (props) {
    /*
        派生出来的新的子类
    */
    var SubClass = function () {};
    /*
        继承父类的属性和方法，
        当然前提是父类的属性都放在prototype中
        而非上面create方法的“实例属性”中
    */
    SubClass.prototype = Object.create(this.prototype);
    // 并且添加自己的方法和属性
    for (var name in props) {
        SubClass.prototype[name] = props[name];
    }
    SubClass.prototype.constructor = SubClass;

    /*
        介于需要以.extend的方式和.create的方式调用：
    */
    SubClass.extend = SubClass.prototype.extend;
    SubClass.create = SubClass.prototype.create;

    return SubClass;
}
{% endhighlight %}

仍然以Human和Man类举例使用说明：

{% highlight javascript %}
var Human = Class.extend({
    say: function () {
        console.log("Hello");
    }
});

var human = Human.create();
console.log(human)
human.say();

var Man = Human.extend({
    walk: function () {
        console.log("walk");
    }
});

var man = Man.create({
    name: "Lee",
    age: 22
});

console.log(man);
// 调用父类方法
man.say();

man.walk();
{% endhighlight %}

[DEMO](http://jsfiddle.net/VsuA2/)

至此，基本框架已经搭建起来，接下来继续补充功能。

1. 我们希望把构造函数独立出来，并且统一命名为init。就好像`Backbone.js`中每一个view都有一个`initialize`方法一样。这样能让初始化更灵活和标准化，甚至可以把init构造函数借出去
2. 我还想新增一个子类方法调用父类同名方法的机制，比如说在父类和子类的中都定义了一个say方法，那么只要在子类的say中调用`this.callSuper()`就能调用父类的say方法了。例如：


{% highlight javascript %}
// 基类
var Human = Class.extend({
    /*
        你需要在定义类时定义构造方法init
    */
    init: function () {
        this.nature = "Human";
    },
    say: function () {
        console.log("I am a human");
    }
})

var Man = Human.extend({
    init: function () {
        this.sex = "man";
    },
    say: function () {
        // 调用同名的父类方法
        this.callSuper();
        console.log("I am a man");
    }
});
{% endhighlight %}

那么Class.create就不仅仅是new一个构造函数了：

{% highlight javascript %}
Class.create = Class.prototype.create = function () {
    /*
        注意在这里我们只是实例化一个构造函数
        而非最后返回的“实例”，
        可以理解这个实例目前只是一个“壳”
        需要init函数对这个“壳”填充属性和方法
    */
    var instance = new this();

    /*
        如果对init有定义的话
    */
    if (instance.init) {
        instance.init.apply(instance, arguments);
    }
    return instance;
}
{% endhighlight %}

实现在子类方法调用父类同名方法的机制，我们可以借用[John Resig的方案](http://ejohn.org/blog/simple-javascript-inheritance/)：

{% highlight javascript %}

Class.extend = Class.prototype.extend = function (props) {
    var SubClass = function () {};
    var _super = this.prototype;
     SubClass.prototype = Object.create(this.prototype);
     for (var name in props) {
        // 如果父类同名属性也是一个函数
        if (typeof props[name] == "function" 
            && typeof _super[name] == "function") {
            // 重新定义用户的同名函数，把用户的函数包装起来
            SubClass.prototype[name] 
                = (function (super_fn, fn) {
                return function () {

                    // 如果用户有自定义callSuper的话，暂存起来
                    var tmp = this.callSuper;
                    // callSuper即指向同名父类函数
                    this.callSuper = super_fn;
                    /*
                        callSuper即存在子类同名函数的上下文中
                        以this.callSuper()形式调用
                    */
                    var ret = fn.apply(this, arguments);
                    this.callSuper = tmp;

                    /*
                        如果用户没有自定义的callsuper方法，则delete
                    */
                    if (!this.callSuper) {
                        delete this.callSuper;
                    }

                    return ret;
                }
            })(_super[name], props[name])  
        } else {
            // 如果是非同名属性或者方法
            SubClass.prototype[name] = props[name];    
        }

        ..
    }

    SubClass.prototype.constructor = SubClass; 
}
{% endhighlight %}

最后给出一个完整版，并且做了一些优化：

{% highlight javascript %}
function Class() {}

Class.extend = function extend(props) {

    var prototype = new this();
    var _super = this.prototype;

    for (var name in props) {

        if (typeof props[name] == "function" 
            && typeof _super[name] == "function") {

            prototype[name] = (function (super_fn, fn) {
                return function () {
                    var tmp = this.callSuper;

                    this.callSuper = super_fn;

                    var ret = fn.apply(this, arguments);

                    this.callSuper = tmp;

                    if (!this.callSuper) {
                        delete this.callSuper;
                    }
                    return ret;
                }
            })(_super[name], props[name])
        } else {
            prototype[name] = props[name];    
        }
    }

    function Class() {}

    Class.prototype = prototype;
    Class.prototype.constructor = Class;

    Class.extend =  extend;
    Class.create = Class.prototype.create = function () {

        var instance = new this();

        if (instance.init) {
            instance.init.apply(instance, arguments);
        }

        return instance;
    }

    return Class;
}
{% endhighlight %}

下面是测试的代码。为了验证上面代码的健壮性，故意实现了三层继承：

{% highlight javascript %}
var Human = Class.extend({
    init: function () {
        this.nature = "Human";
    },
    say: function () {
        console.log("I am a human");
    }
})

var human = Human.create();
console.log(human);
human.say();

var Man = Human.extend({
    init: function () {
        this.callSuper();
        this.sex = "man";
    },
    say: function () {
        this.callSuper();
        console.log("I am a man");
    }
});

var man = Man.create();
console.log(man);
man.say();

var Person = Man.extend({
    init: function () {
        this.callSuper();
        this.name = "lee";
    },
    say: function () {
        this.callSuper();
        console.log("I am Lee");
    }
})

var person = Person.create();
console.log(person);
person.say();
{% endhighlight %}
[DEMO](http://jsfiddle.net/4qRUz/)

##是时候彻底抛弃new关键字了

如果不使用new关键字，那么我们需要转投上两节中反复使用的`Object.create`来生产新的对象

假设我们有一个矩形对象：

{% highlight javascript %}
var Rectangle = {
    area: function () {
        console.log(this.width * this.height);
    }
};
{% endhighlight %}

借助Object.create，我们可以生成一个拥有它所有方法的对象：

{% highlight javascript %}
var rectangle = Object.create(Rectangle);
{% endhighlight %}

生成之后，我们还可以给这个实例赋值长宽，并且取得面积值

{% highlight javascript %}
var rect = Object.create(Rectangle);
rect.width = 5;
rect.height = 9;
rect.area();
{% endhighlight %}

注意这个过程我们没有使用new关键字，但是我们相当于实例化了一个对象(rectangle)，给这个对象加上了自己的属性，并且成功调用了类(Rectangle)的方法。

但是我们希望能自动化赋值长宽，没问题，那就定义一个create方法：

{% highlight javascript %}
var Rectangle = {
    create: function (width, height) {
      var self = Object.create(this);
      self.width = width;
      self.height = height;
      return self;
    },
    area: function () {
        console.log(this.width * this.height);
    }
};
{% endhighlight %}

使用方式如下：

{% highlight javascript %}
var rect = Rectangle.create(5, 9);
rect.area();
{% endhighlight %}

在纯粹使用Object.create的机制下，我们已经完全抛弃了构造函数这个概念。一切都是对象，一个类也可以是对象，这个类的实例不过是一个它自己的复制品。

下面看看如何实现继承。我们现在需要一个正方形，继承自这个长方形

{% highlight javascript %}
var Square = Object.create(Rectangle);

Square.create = function (side) {
  return Rectangle.create.call(this, side, side);
}
{% endhighlight %}

实例化它:

{% highlight javascript %}
var sq = Square.create(5);
sq.area();
{% endhighlight %}

这种做法其实和我们第一种最基本的类似

{% highlight javascript %}
function Man(name, age) {
    Human.call(this, name);
    this.age = age;
} 
{% endhighlight %}

上面的方法还是太复杂了，我们希望进一步自动化，于是我们可以写这么一个extend函数

{% highlight javascript %}
function extend(extension) {
    var hasOwnProperty = Object.hasOwnProperty;
    var object = Object.create(this);

    for (var property in extension) {
      if (hasOwnProperty.call(extension, property) || typeof object[property] === "undefined") {
        object[property] = extension[property];
      }
    }

    return object;
}

/*
    其实上面这个方法可以直接绑定在原生的Object对象上：Object.prototype.extend
    但个人不推荐这种做法
*/

var Rectangle = {
    extend: extend,
    create: function (width, height) {
      var self = Object.create(this);
      self.width = width;
      self.height = height;
      return self;
    },
    area: function () {
        console.log(this.width * this.height);
    }
};
{% endhighlight %}

这样当我们需要继承时，就可以像前几个方法一样用了

{% highlight javascript %}
var Square = Rectangle.extend({
    // 重写实例化方法
    create: function (side) {
         return Rectangle.create.call(this, side, side);
    }
})

var s = Square.create(5);
s.area();
{% endhighlight %}

##结束语

本文对去new关键字的方法做了一些罗列，但工作还远远没有结束，有非常多的地方值得拓展，比如：如何重新定义`instance of`方法，用于判断一个对象是否是一个类的实例？如何在去new关键字的基础上继续实现多继承？希望本文的内容在这里只是抛砖引玉，能够开拓大家的思路。



引用资料

[参考文献集合](https://www.site2share.com/folder/20020510)