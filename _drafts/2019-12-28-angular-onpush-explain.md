# 如果 Angular 组件无法更新，这篇文章能帮到你

为了避免歧义，事先声明在接下来的内容中 AngularJS 代指 Angular 1.x 版本，Angular 代指 Angular 8 版本

在最近开发的一个功能中，我们无法在 TigerUI 中实现某个 Angular 组件触发另一个 AngularJS 组件的展示。我们经过很长时间才弄清楚了其中的来龙去脉。就像我在之前某封 aha moment 邮件里说的那样，避免写出错误代码的方式是真正理解你编写的代码。所以在这里我不会仅仅给出解决方案，我还会详细叙述这个问题背后的机理。如果以后你遇到了相似的问题，希望这篇文章能给你带来帮助。

很多年前我听过一个笑话，说如果你拿一个疑问去问专家，你的一个疑问会变成三个疑问，因为他会用另外两个你更不明白的词来解释这个疑问。但后来我发现这不是笑话而是绝大部分我自己面临的现状。所以在解释这个问题的过程中，我不可避免的需要引入更多的知识进行解释。好在它们都易于理解，只是有些冗长。

因为涉及到很多基本原理，这篇文章也能够让你一窥 Angular 和 AngularJS 的运行机制以及优化技巧 

感谢 @yifan 同学的支持，在解决这个问题的过程中给了我们很多的指导。这次也是 friends 团队第一次开发 Angular(8) 相关的功能，其中的挫折酸爽可想而知。一周的开发体验让我联想起了那个经典的书本标题：“从入门到精通”

## 也许是 `ChangeDetectionStrategy.onPush` 的问题

我们怀疑是我们的 Angular 组件出了问题。如果你有心留意的话，大部分 Angular 组件的声明中会包含一个名为 `changeDetection` 的配置，例如：

```javascript
@Component({
  selector: 'app-general-system-template',
  template: ``,
  styles: [require('./system-template-header.component.scss')],
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class SystemTemplateSectionsComponent {}
```

留意 `@Component`装饰器的最后一行：`changeDetection: ChangeDetectionStrategy.OnPush`。在遇到这个问题时，我们首先怀疑是它引起的。然而这项配置究竟起什么样的作用让我们想当然的把矛头指向它？这里我们首先要解释一下什么是 change detection

### Change Detection

无论是 Angular 还是 AngularJS ，其中最重要的一个机制是判断当前 View 是否要进行更新。我们可以把这种机制统一解读为 “脏检查”，即判断数据是否发生了变化。但是在 Angular 和 AngularJS 中字面上的 “脏检查” 背后的逻辑却大相径庭。在 Angular 中这种机制称之为 change detection（以下我们简称 **CD**），而在 AngularJS 中这种机制称之为 dirty check（以下简称 **DC**）

我们先简单了解 CD 是如何工作的，DC 之后会谈到

想象一个最简单的场景：你在页面上点击了一个按钮。但如果你在点击事件的回调函数中更改了一些数值，Angular 是怎么知道的？

因为 Angular 采用 monkey patch 的方式重写并覆盖了浏览器的 `addEventListenter` 接口，在调用回调函数的同时手动触发了 CD，代码类似于：

```javascript
// this is the new version of addEventListener
function addEventListener(eventName, callback) {
     // call the real addEventListener
     callRealAddEventListener(eventName, function() {
        // first call the original callback
        callback(...);     
        // and then run Angular-specific functionality
        var changed = angular2.runChangeDetection();
         if (changed) {
             angular2.reRenderUIPart();
         }
     });
}
```

几乎所有的浏览器 API 都被 patch 了，比如你能想到的所有浏览器事件，以及 `setTimeout` 、`setInterval`，还有 Ajax 请求等等。

所有的 patch 工作都交给 Angular 自带的 zone.js 完成， 同时 zone.js 还提供代码的执行上下文。zone.js 是另一个话题不这次的重点，你只需要记住 zone.js 的目的是告诉 Angular 何时该进行渲染重绘。

Angular 的默认 CD 策略也非常简单：它判断每个组件模板里表达式使用到的值前后是否发生了变化。对于对象类型，Angular 也不会进行深度比较，它只会对对象里使用到值进行值比较

这样的 CD 比较代码是机器生成的，这比人工编写的普适比较代码针对性强，性能要好得多。

最后提示一个优化技巧：反过来想，如果 CD 作为 Angular 的基本制度存在的话，有没有可能我们的代码游离在 CD 之外？可以的，并且为了避免某些频繁操作频繁的触发 CD 而造成性能问题，使得代码游离在 CD 以外然后手动的触发 CD 也常常作为优化的手段之一。在 CD 之外执行代码有两种方式：

* `ChangeDetectorRef.detach()` 手动和 CD 断绝关系

```javascript
@Component({
  selector: 'ns-demo',
  template: ``
})
export class DemoComponent {
  constructor(private ref: ChangeDetectorRef) {
    this.ref.detach();
  }
```

* 使用 zone.js 提供的 `runOutsideAngular` API 在 zone 之外执行代码：

```javascript
@Component({
  selector: 'ns-demo',
  template: ``
})
export class DemoComponent {
  constructor(private zone: NgZone) {
    this.zone.runOutsideAngular(() => {})
  }

```

### OnPush  策略

回到主线，`changeDetection: ChangeDetectionStrategy.OnPush` 意味我们将默认的 CD 策略改为 OnPush。这个策略意味着只有在以下几种情况下才会触发 CD:

* 组件的任意一个 input 属性发生了修改



* 组件的事件处理函数被触发调用
* 显式的调用 CD
* `async` pipe





 

## 参考资料：

* [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)
* [what is the use of Zone.js in Angular 2](https://stackoverflow.com/a/40903120/508236)