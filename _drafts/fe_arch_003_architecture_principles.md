# MVC 启示录：模块的职责，作用域和通信

在上一篇中，我提出了一个应用中常见的问题：如何在多个视图中共享同一份数据，并且保证它的改动能够同步到不同的视图中去？

针对这个问题我给出了两类解决方案：一类是用户行为驱动的意识流编码，比如当我选择将素材回滚到某个历史版本时，我想当然的手动去更新每一个视图：

```javascript
historyView.rollbackTo((historyInfo) => {
   	const {size: { width, height }, color, history, layer } = historyInfo;
    
    infoView.updateHeight(height);
    infoView.updateWidth(width);
    
    colorView.updateColor(color);
    historyView.updateHistory(history);
    layerView.updateHistory(layer)
})
```

另一类是 MVC 解决方案，以 Backbone 为例，它通过广播事件来通知各个视图的更新发生了

```javacript
// InfoView.js:
initialize: function () {
    this.listenTo(historyModel, 'rollback', this.updateSizeInfo);
},
updateSizeInfo: function (event) {
    //...
}

// ColorView.js:
initialize: function () {
    this.listenTo(historyModel, 'rollback', this.updateColorInfo);
}
```

很明显第二种解决方案会更好，从维护代码的体验上说，我们不用去主动的维护视图间的调用关系。每当有视图添加或者被删除时，不需要找到它的每一个消费方把这层调用关系做对应的修改。

但不知道你们有没有考虑过为什么第二类方案会给我们带来这样的便利，我们是否能够从中得到一些启示来帮助我们今后的代码也变得同样的利于维护？在这一篇的内容里我们详解这些编程中的原则和模式。这将为我们之后的内容奠定基础

## 分离关注点

