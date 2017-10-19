# 深入React的生命周期

## 前言

本文是对开源图书[React In-depth: An exploration of UI development](https://www.gitbook.com/book/developmentarc/react-indepth/details)的归纳、提炼还有增强。同时也融入了自己在开发中的一些心得。

你或许会问，阅读完这篇文章之后，对工作中开发React相关的项目有帮助吗？实话实说帮助不会太大。这篇文章不会教你使用一项新技术，不会帮助你提高编程技巧，而是完善你的React知识体系，例如区分某些概念，明白一些最佳实践是怎么来的等等。如果硬是要从功利的角度来考虑这些知识带来的价值，那么会是对你的面试非常有帮助，这篇文章里知识点在面试时常常会被问到，为什么我知道，因为我吃过它们的亏。

React组件是存在生命周期的，从出生（mount）到更新（update）到死亡（unmount），然而我们怎么知道组件进入到了哪个阶段？只能通过React组件暴露给我们的钩子（hook）函数来知晓。什么是钩子函数，就是在特定阶段执行的函数，比如`constructor`只会在组件出生阶段被调用一次，这就算是一个“钩子”。反过来说，当某个钩子函数被调用时，也就意味着它进入了某个生命阶段，所以你可以在钩子函数里添加一些代码逻辑在用于在特定的阶段执行。当然这不是绝对的，比如`render`函数既会在出生阶段执行，也会在更新阶段执行。顺便多说一句，“钩子”在编程中也算是一类设计模式，比如github的[Webhooks](https://developer.github.com/webhooks/)。顾名思义它也是钩子，你能够通过Webhook订阅github上的事件，当事件发生时，github就会像你的服务发送POST请求。利用这个特性，你可以监听master分支有没有新的合并事件发生，如果你的服务收到了该事件的消息，那么你就可以例子执行部署工作。

React组件的生命周期分为三个阶段，按照时间顺序分别是出生（初始化/Mounting），成长（更新/Growth/Update）以及死亡（Unmount）。我们按照阶段的时间顺序对每一个钩子函数进行讲解。

## 出生

首先我们要引入一个概念：组件（Component）。组件非常好理解，就是可以复用的模板。例如通过按钮模板我们可以实例化出多个相似的按钮出来。这和代码中类（Class）的概念是相同的。并且在ES6代码中定义组件时也是通过类来实现的：
```javascript
import React from 'react';

class MyButton extends React.Component {
  constructor(props) {
    super(props);
  }
  render() {
    return (
      <button>My Button</button>
    )
  }
}
```
也可以通过`React.createClass`接口来定义组件：
```javascript
const MyButton = React.createClass({
  render: function() {
    return (
      <button>My Button</button>      
    );
  }
});
```



Element和Instance的区别有什么？
没有继承React的class是什么意思？
https://stackoverflow.com/questions/43703984/react-component-without-extend-react-component-class
https://stackoverflow.com/questions/36296658/do-not-extend-react-component

this.setState是异步的
当props发生更改时，componentWillReceiveProps会被调用；但是并不意味着componentWillReceiveProps被调用了而props发生了更改。也就是在一些情况下，componentWillReceiveProps被调用了，但是props并没有发生更改
https://reactjs.org/blog/2016/01/08/A-implies-B-does-not-imply-B-implies-A.html
React.PureComponent

render返回的是什么？是element吧？
render pass？

需要实验的部分：key以及内存回收