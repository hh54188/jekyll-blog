# MVP 是归宿

在这一篇中，我无法用分段分章节的方式对内容进行描述。因为模式的产生是与它需要解决的问题息息相关的。我将尝试从最原始的问题出发，向各位展示解决方案是如何一步一步演化出现的。

在 Flux 架构中，有两个问题依然没有被提到，一个是表现层模型，另一个是测试

我们从表现层逻辑说起

表现层模型即  Presenter Model 或者称之为 View Model。这是一些与业务无关紧要，但是与可视化展示息息相关的数据。简单的例如某个可折叠的控件是否处于折叠状态，复杂的可以是某个字段的校验规则，校验的出错信息，或者是图表的展现类型（饼图还是柱状图）等等。

想象一下在 Flux + React 的框架下这些数据应该存放在哪里？我想包括曾经的我在内的大多数人都会把它放在组件中，这是想当然的事情：既然它们属于表现层状态，那么就应该放在表现层的组件中；而不放在 Redux 中的另一个原因是，Redux 并非是所有功能的标配，把所有数据都往 Redux 中集成会让整个 store 显得臃肿，维护起来反而不利。

但在实际应用中这些数据并没有那么纯粹，甚至可以说大多数时候表现层模型和业务模型是息息相关，比如用户允许在下面的表格中选中某些商品，然后选择将它们的价格清零：

![](./images/fe_arch_005_mvp/table.png)

简易的伪代码可能是这样的：

```javascript
// 每行的选中函数
function onRowSeleted(rowId) {
  selectedRows.push(rowId)
}

// 左上角提交按钮的回调函数
function onSubmit() {
  // Step 1: Clear selected data's price:
  selectedRows.forEach(rowId => data[rowId].price = 0);
  
  // Step 2: Sync to local store:
  syncToLocalStoreAction(data);

  // Step 3: Sync to remote backend:
  syncToBackendRequest(data);

  // Step 4: Clear view model:
  selectedRows = [];
}
```

有几个问题我要需要考虑：

- 如果上面的这段代码书写在某个 React 组件中，如果某天我们需要切换为另一个 UI 框架时，这部分代码我们可能需要原封不动的照抄一遍，但你可以看到，上述代码并没有使用到 React 技术特定的接口或者语法。理论上来说时可以无缝移植的
- 即使需求不是迁移框架，而是需要上述逻辑在同一个应用的不同组件中重用，例如上面截图是清除水果的价格，另一个页面需要清除 3C 产品的商品价格， 抄一遍似乎也有一些多余。这样就可能产生“散弹式修改”的代码坏味道
- 最后一个问题是测试，对于相同的逻辑，我们可不希望当逻辑复用时需要编写的测试也要加倍。

再次提醒以上考虑的出发点是我们在第一章讨论的非功能需求，即可维护性和测试。如果你不在乎非功能需求，那么接下来的内容对你的意义并不大。

从上面的三点叙述中，我们不难得出我们需要进一步解决的问题：

1. 即表现层逻辑、业务逻辑、与视图三者其实并非强相关的，尤其是表现层逻辑可以与视图使用的具体技术栈无关。
2. **表现层逻辑需要和视图进行分离**，以便于复用。

同时注意上面 `onSubmit` 回调函数中的内容，它其实描述的是一些列流程，在这个提交操作中，我首先需要做什么、其次需要做什么以及最后需要做什么。这样的流程是用户使用的其中一个场景，也是其中一个用例。这些用户用例本质上是和表现层的技术无关的，无论是使用 React 还是 Vue 都需要将它们实现。所以我们可以把它们作为独立的模块与视图隔离并且封装起来。借用后端的概念，我们可以把这类模块、这种规则的分层称之为**Service Layer**，后文中使用**服务层**称呼它。

![](./images/fe_arch_005_mvp/service_layer.png)



封装用户用例只是服务层的实现，再往上抽象点看，它定义的其实是应用的边界。因为无论操作指令来自于用户界面，或者想象它终有一天被移植到命令行界面，操作指令来自于命令行输入，所有可行的操作以及需要对这些操作做出的响应都不过封装于该层。**该层决定了应用能做什么不能做什么**。直接搬用 Martin Fowler [对于服务层的定义](https://martinfowler.com/eaaCatalog/serviceLayer.html)如下：

> *Defines an application's boundary with a layer of services that establishes a set of available operations and coordinates the application's response in each operation.*

![](./images/fe_arch_005_mvp/ServiceLayerSketch.gif)





但不难看出服务层其实也只是“中介”而已，在服务层的实现里它依然需要调用其它的模块来实现功能，最需要互动的就是各种领域模型。这个方面的相关的问题我们待会再谈

在后端开发中，服务层不会关心视图，视图的操作通常以 API 的形式到达的这里，所以并不存在表现层逻辑的问题。但是涉及在界面的开发中我们必须要解决这个问题，我们都同意表现层逻辑需要和视图的实现分离，那么分离之后放哪呢？目前看来服务层是一个不错的选择，因为 1) 表现层逻辑确实和用例相关；2) 服务层也确实是和视图分离的。服务层和视图的合作方式也非常简单，通常是把事件委托给服务层处理而已。React 示例代码如下：

```javascript
function TodoComponent() {
  	const serviceLayer = new ServiceLayer();
    
    function onComplete(todo) {
    	serviceLayer.completeTodo(todo);
    } 
    
    function onDelete(todo) {
        serviceLayer.deleteTodo(todo);
    }
    
    return (/.../)
}
```

