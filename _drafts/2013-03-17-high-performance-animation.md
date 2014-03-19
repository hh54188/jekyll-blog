# Javascript高性能动画与页面渲染

**不确定的地方:**

- 浏览器（不是javascript）真的是单线程的吗？有没有参考？

## No setTimeout, No setInterval

如果你一定要使用`setTimeout`或者`setInterval`来实现动画，那么原因只能有一个，就是需要精确的控制动画。之所以浏览器兼容问题在我看来不考虑是因为，至少在现在这个时间点，高级浏览器的普及程度足够让你有理由、也应该、同时也有空间在实现动画时使用更高效的方式。 

为什么不推荐使用`setTimeout`/`setInterval`？这要从我们的需求谈起。

### 什么样的动画才算是高效的

这里的高效翻译过来就是流畅，我们需要的是流程的动画？那实现动画的效率就不考虑了吗？傻瓜，你觉得用低效率实现的动画能保证动画的流程吗？

流畅的动画和流畅的网页原理是一样的，要知道网页的每一帧变化都是浏览器绘制出来的。如果你对PC游戏有了解的话，就应该能想到最佳的绘制频率应该是每秒60帧(60fps)，当然30fps也是可以接受的，至少人的视觉几乎察觉不到任何的差别。帧数在页面的性能调试中是一个非常重要的衡量指标。在Chrome的调试工具Devtools中，随处可见FPS的踪影：

![frames](./images/fps.jpg)

接下来的工作中，我们将会用到这些工具，来实时查看我们页面的性能。

我们的目标已经有了，每秒绘制60帧。但坏消息也随之而来，因为这也意味着，留给每帧绘制的时间只有 1000 / 60 = 16.7ms。这时间当然远远不够，下面这些问题是在setTimeout中不可避免的，也是我们接下去需要解决的

- Single thread：说浏览器是单线程(single thread)并不准确，但**大部分**情况下，页面的重排(reflow)、重绘(repaint)，脚本的运行、甚至垃圾回收(garbage collector)所处的线程是相互排斥的(所以你在timeline中看到的是瀑布图)，也就是在任意一个时刻，只允许其中的一项任务在运行。在16.7秒内能把这些工作依次完成实属不易

-  Timer resolution: Timer resolution代指浏览器每间隔多少时间更新时钟，没有浏览器是精确到每毫秒级别的（即使你在setTimeout中指定0ms的延时，也只能按照浏览器能够达到的最低延时计算），IE8及其之前的IE版本更新间隔为15.6毫秒，而Chrome的更新间隔为4ms(WHATWG中规定的setTimtout延迟为4ms，setInterval延时为10ms)。也就是说即使在Chrome中，在理想情况下也只能以16ms尽可能的接近16.7ms

- Event loop: Javascript是单线程运行，需要执行的代码只能以队列的方式等待被执行；而异步脚本比如setTimeout(或者XMLHttpRequest请求)只能在队列尾部等待之前的代码被执行，比如下面这段代码：

```
function runForSeconds(s) {
    var start = +new Date();
    while (start + s * 1000 > (+new Date())) {}
}

document.body.addEventListener("click", function () {
    runForSeconds(10);
}, false);

setTimeout(function () {
    console.log("Done!");
}, 1000 * 3);
```

如果在3秒钟之内页面没有被点击，`Done!`能够如预期显示出来，但如果在3s钟之内触发了点击事件，我们的`runForSecond`会连续的执行10s，这当中即使3s的期限已到，setTimeout回调也无法被执行，只能等待排在它之前的代码执行完毕


### 垂直同步问题

屏幕输出每一帧画面都是由系统计算出来的，但通常系统计算出画面的速度会比屏幕输出的速度快很多，这样会产生一个问题是，当屏幕还没有输出完当前的画面，下一张画面就已经把当前画面覆盖了，这样画面上就同时出现了两帧的画面，画面出现了撕裂：

![teardown](./images/teardown.png)

这个问题的解决办法是


