# MVC 解决了的和没有解决的

## 我的第一个单页面应用

我在刚入这行的时候加入的是一家创业公司，它们的产品是 iPad 上给儿童阅读的电子互动图书。而我的工作职责就是负责一款在线单页面应用编辑工具的前端部分，以供公司内的编辑同事制作这些电子图书。编辑同事可以在这个工具内导入素材，比如音频、视频、或者图片。然后摆放这些素材的位置，给它们设置动画效果等等。它类似于一个可以制作动画的 Photoshop，或者平面版本的 Unity3D 编辑器。当然用这两个比喻实在是太抬举它了，但是我想你们大概能想象出它的功能和样子。

我们就从这个应用开始

它的界面类似于 Photohop 的工作区

![](./images/fe_arch_002_mvc_solved/adobe-photoshop.png)

如上图所示，每当你选中一个素材，右侧的不同属性栏中就会呈现这个素材不同维度的状态，例如它的尺寸，色彩状况，操作历史，图层状态等等。我们不如先把不同的视图区域标注一下：

![](./images/fe_arch_002_mvc_solved/photoshop-layout.png)

那么对于“每选择一个素材是就展示它的相关这个属性”这个需求，我想当然的第一版（伪）代码是这么写的：

```javascript
canvasView.onSelectElement(image => {
    const {size: { width, height }, color, history, layer } = getComputedStyle(image);
    
    infoView.updateHeight(height);
    infoView.updateWidth(width);
    
    colorView.updateColor(color);
    historyView.updateHistory(history);
    layerView.updateHistory(layer)
})
```

如果只有这一个需求不会有大问题，现在我们提出另一个需求：当删除选中的元素时把各个属性栏的数值清空：

```javascript
canvasView.onDeleteSelectedElement(image => {
    infoView.updateHeight(0);
    infoView.updateWidth(0);
    
    colorView.updateColor(null)
    historyView.updateHistory(null)
    layerView.updateHistory(null)
})
```

以及我们可以从 history 中选择回滚到某个版本：

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

我的第一版代码真的是这么写的，想必此时你也应该感受到我的痛苦了：**我必须手动的更新每一个视图状态信息**。这会导致更严重的后果：

- 如果需要额外的添加一个视图，比如用户展示文件信息的 FileView。那么在上面三段代码中我都要手动添加一行类似的用户更新 FileView 信息的代码。但实际代码中不可能只存在三段代码，所以我有可能会遗漏这样的修改

- 如果我需要页面不止有一个 InfoView 怎么办？例如 InfoView01 用于展示公制单位，InfoView02 用于展示英制单位。也就是说如果页面上有多出需要展示同一份信息的时候，我的代码需要复制 N 遍：

  ```javascript
  infoView01.updateHeight(height);
  infoView01.updateWidth(width);
  
  infoView02.updateHeight(height);
  infoView02.updateWidth(width);
  ```

你可以想象一下五个 view 之间的调用关系，如果每一个视图都会和其它四个视图直接沟通的话，它们之间的关系会变的如下图所示。再多加入一个 view 将会是一个灾难

![](./images/fe_arch_002_mvc_solved/pentastar.jpg)

## MVC 来拯救

MVC 在不同的上下文中的架构都不一样。从最早的 Smalltalk 里的 MVC，到 .NET 的 MVC，再到 JavaScript 中的 MVC 框架都不尽相同。但我们通常谈论 MVC 是泛指的是服务端架构中的 MVC。以 .NET 的 MVC 为例，我们可以在 [ASP.NET MVC 文档](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/overview/understanding-models-views-and-controllers-cs) 中找到关于 MVC 中三个角色的定义，这里我们重点关注 Model 和 Controller。在之后的内容中会再次强调

[Model](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/overview/understanding-models-views-and-controllers-cs#understanding-models):

> An MVC model contains all of your application logic that is not contained in a view or a controller. The model should contain all of your application business logic, validation logic, and database access logic. 

标注几个重点

- 不与 view 和  controller 的逻辑重叠
- 包含业务逻辑、校验逻辑、数据库访问逻辑

[Controller](https://docs.microsoft.com/en-us/aspnet/mvc/overview/older-versions-1/overview/understanding-models-views-and-controllers-cs#understanding-controllers):

>A controller is responsible for controlling the way that a user interacts with an MVC application. A controller contains the flow control logic for an ASP.NET MVC application. A controller determines what response to send back to a user when a user makes a browser request.

同样标注几个重点

- 响应用户的交互和请求
- 控制应用流程

简单来说一个服务端 MVC 的应用流程如下：用户通过 URL 访问应用，controller 负责响应用户的请求，获取数据模型，并将数据渲染在页面模板上之后将页面返回给用户

![](./images/fe_arch_002_mvc_solved/mvc_flow_detail.png)

例如 Node.js 的 MVC 框架 [kraken.js](http://krakenjs.com/) 的 controller 语法如下：

```javascript
'use strict';

var IndexModel = require('../models/index');

module.exports = function (router) {
    var model = new IndexModel();

    router.get('/', function (req, res) {
        res.render('index', model);
    });
};
```

相信你也发现了 MVC 其实是更适用于多页面应用，所以它与前端这种单页面应用场景并非天生契合。前端的 MVC 架构与后端有很大的不同

### Backbone.js

