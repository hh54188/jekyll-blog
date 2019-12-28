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

所有的 patch 工作都交给 Angular 自带的 zone.js （NgZone 和 zone.js 是有差别的，为了便于说明理解，这里统一为一个概念）完成， 同时 zone.js 还提供代码的执行上下文。zone.js 是另一个话题不这次的重点，你只需要记住 zone.js 的目的是告诉 Angular 何时该进行渲染重绘。

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

回到主线，`changeDetection: ChangeDetectionStrategy.OnPush` 意味我们将默认的 CD 策略改为 OnPush。这个策略只有在以下几种情况下才会触发 CD:

* 组件的任意一个 inputs 的**引用**发生了修改

Angular 组件支持类似于 React 父组件传递属性给子组件的机制，OnPush 策略下将 Angular 组件变成了类似于 React 中 PureComponent ，即仅在属性引用发生改变时才重新渲染。例如我们有一个 UserName 子组件用于展示用户的名称：

```javascript
@Component({
    selector: 'app-user-name'，
    template: `<div>{{userName.lastName}}, {userName.firstName}</div>`
    changeDetection: ChangeDetectionStrategy.OnPush
})
export class UserName {
    @Input userName: object;
}
```

在父组件中我们使用它：

```javascript
@Component({
    selector: 'app-user',
    tempalte: `<app-user-name [userName]="">`
})
export class User {
    userNameInParentComponent = {
        firstName: 'firstName',
        lastName: 'lastName'
    }
	onClick() {
        this.userNameInParentComponent.fistName = Math.random()
    }
}
```

即使 `onClick` 回调函数执行后 `userNameInParentComponent` 发生了更改，但子组件在页面上并不会进行更新。因为我们只是修改了这个对象的内部状态，它的引用却没有发生变化。如果想要生效，应该重新给 `userNameInParentComponent` 赋值：

```javascript
onClick() {
    this.userNameInParentComponent = {
        fistName: Math.random(),
        lastName: 'lastName'
    }
}
```

和 React 一样，这个特性同样也是常被作为性能优化的手段之一，我们需要在代码中引入 Immutable.js 机制

* 组件的事件处理函数被触发调用

注意这里仅限于**事件处理函数**，比如下面的代码：

```javascript

@Component({
 template: `
    <button (click)="add()">Add</button>
    {{count}}
  `,
  changeDetection: ChangeDetectionStrategy.OnPush
})
export class CounterComponent {
  count = 0;
  constructor() {
    setTimeout(() => this.count = 5, 0);
    setInterval(() => this.count = 5, 100);
    Promise.resolve().then(() => this.count = 5); 
    this.http.get('https://count.com').subscribe(res => {
      this.count = res;
    });
  }
  add() {
    this.count++;
  }
}
```

除了 `click` 事件调用了 `add` 方法之外，构造函数中 `setTimeout`，`setInterval`等等也调用了 `add` 方法，但只有 click 之后页面上的 count 才会更新，其他的方式虽然更改了 `count` 的值，但并不会触发 CD, 也就不会造成重新渲染

* 显式的触发 CD

刚刚在上面我提到过我们可以通过 ``ChangeDetectorRef.detach()`` 和 `zone.runOutsideAngular` 使得代码游离在 CD 之外，但终归我们需要触发 CD 来重新渲染页面，此时我们可以通过已有的 API 来显式的触发 CD：

* `ChangeDetectorRef.detectChanges()`: 在组件和它的所有子组件上运行 CD
* `ApplicationRef.tick()`: 运行整个程序的 CD
* `ChangeDetectorRef.markForCheck()`: （这个方法我不确定我理解的是否正确）它不会立即触发 CD, 而是把当前的组件标记为需要 check 的状态，在未来当父组件被 check 时才触发当前组件的 CD
* `NgZone.run(() => {})`: 指定在 zone 里运行代码触发 CD

除了上面的三种主流情况以外，还有一些特殊情况，比如 `async`，`Observable<>`可以触发 CD，出于篇幅考虑就不详述了，可以在我最后给出的参考资料里找到说明。以上的几个方案足够应付大多数情况 

但是即使在尝试了上面各种能够触发 CD 的方法，甚至移除 `ChangeDetectionStrategy.OnPush` 之后都无法触发 AngularJS 组件的渲染之后。我们陷入了僵局。

## 因为降级

于是我们决定重新审视我们代码，追踪代码的实现。我们目前出现问题的代码是通过一个封装之后的 service 触发 AngularJS 的 message box 功能（即你们在出现问题时 Tiger 上方看到哪条 banner），调用代码类似如下:

```javascript
constructor(private messageBox: MessageBox) {
    this.messageBox.error(error);
}
```

而 AngularJS 那边的实现主要代码如下，以上面的 `error` 方法为例，实际上只是给 `message` 变量赋值而已：

```javascript
// Scripts/common/message-box.mjs
export function messageBoxFactory(i18n) {
  let messageStore = null;
  return {
    error(message) {
      messageStore = { description: message, type: 'error' };
    },
```

而在另一端代码里，正用着 `$watch` 监视着 `messageStore` 变量是否发生变化，如果发生了变化则在页面上提示消息：

```javascript
// Scripts/shared/directives/my_message_box.es6
(((coreModule) => {
  coreModule.directive('myMessageBox', ['messageBox', '$sanitize', function MyMessageBox(messageBox, $sanitize) {
    return {
      link(scope, iElement) {
        scope.$watch(() => {
          if (!messageBox.hasMessage()) {
            return;
          }
          fadeInMessageBox();
          const message = messageBox.retrieveMessage();
          // ...
          showMessageBox();
```

我们猜想问题可能是因为 `scope.$watch` 并没有监视到 `messageStore` 的变化。就是没有主动的进行 DC(dirty check)于是我们尝试将 scope 赋值到全局 windows 上，然后手动调用 `scope.$apply()` ：

后来发现在任意时候用鼠标点击页面一下也能触发 messageBox 的显示。我们就能很肯定是因为 AngularJS 没有主动运行 DC 导致的。但是为什么？这也许和 Angular 和 AngularJS 的混用有关。

 

## 参考资料：

* [Angular Change Detection - How Does It Really Work?](https://blog.angular-university.io/how-does-angular-2-change-detection-really-work/)
* [what is the use of Zone.js in Angular 2](https://stackoverflow.com/a/40903120/508236)
* [A Comprehensive Guide to Angular onPush Change Detection Strategy](https://netbasal.com/a-comprehensive-guide-to-angular-onpush-change-detection-strategy-5bac493074a4)
* [Angular OnPush Change Detection and Component Design - Avoid Common Pitfalls](https://blog.angular-university.io/onpush-change-detection-how-it-works/)
* [Angular Performances Part 4 - Change detection strategies](https://blog.ninja-squad.com/2018/09/27/angular-performances-part-4/)
* [Using Zones in Angular for better performance](https://blog.thoughtram.io/angular/2017/02/21/using-zones-in-angular-for-better-performance.html)
* [What's the difference between markForCheck() and detectChanges()](https://stackoverflow.com/questions/41364386/whats-the-difference-between-markforcheck-and-detectchanges)
* [Triggering change detection manually in Angular](https://stackoverflow.com/questions/34827334/triggering-change-detection-manually-in-angular)