# MVP 是归宿

在 Flux 架构中，有两个问题依然没有被提到，一个是表现层模型，另一个是测试

## 表现层模型

表现层模型即  Pressenter Model 或者称之为 View Model。这是一些与业务无关紧要，但是与可视化展示息息相关的数据。简单的例如某个可折叠的控件是否处于折叠状态，复杂的可以是某个字段的校验规则，校验的出错信息，或者是图表的展现类型（饼图还是柱状图）等等。

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
- 