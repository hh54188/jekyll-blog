# 【译文】给构建大型 redux 应用的五个建议

本篇译文的原文在这里：[Five Tips for Working with Redux in Large Applications](https://techblog.appnexus.com/five-tips-for-working-with-redux-in-large-applications-89452af4fdcb)

## 译者序

为什么翻译这篇文章，是因为本文中给出的建议和我在实际项目中的实践不谋而合，更彻底也更优秀。

当项目规模逐渐增大之后，入门文档和教程级别的项目代码的局限性会逐渐显现出来，并且你会遇到在小型应用中不会遇到的问题。更致命的地方在于，如果想要解决这些问题，需要对整个应用的代码做出调整，所谓牵一发而动全身。所以最好是在建立项目之处就有意识的融入最佳实践，有助于预防将来问题的发生

这篇文章并不适合 redux 的初学者，希望你已经开发了少许完整应用，或者至少正在开发你的第一个应用的时候来阅读这篇文章，这样你才更有体会。

相同建议在 Redux 的官方文档或者 React 的官方文档里或多或少肯定都有提及。但是文档太庞大，入口太深以至于把这些内容给淹没了。如果你们没有阅读过它们，至少这一篇也能给你带来提高。

译文中的每个小节结尾都会加入的我个人补充的内容，类似于评论~~音轨~~；有些不便的翻译，或者翻译后很别扭，或者大家公认的技术词汇的地方仍然保持原文。下面正式开始

## 正文

Redux 一个用于管理应用状态的出色工具。它的单向数据流和 immmutable state 特色让我们更容易追踪状态的变更。每一个状态的变更都是由被调度的 action 导致 reducer 函数返回新的状态引起的。我们站点上的许多使用 Redux 构建的用户界面都需要处理大量的数据和复杂的交互，因为用户需要通过这些界面管理他们的广告或者在平台上更新库存信息。在开发这些界面的过程中，我们掌握了一些规则和窍门有助于我们让 Redux 更易于管理。接下来要讨论的几个要点对那些使用 Redux 开发大型数据集成类型应用的同学们有所帮助

- 使用索引和选择器用于排序和访问数据
- 把数据对象、编辑状态和其他的UI状态隔离开
- 判断是否在单应用的多个视图间共享状态
- 在状态间重用 reducer
- 将组件连接至 Redux 状态的最佳实践

## 1. 使用索引（index）存储数据，使用选择器（selector）访问数据

选择正确的数据结构对应用的组织和性能至关重要。使用索引存储来自API的可序列化数据会带来很多好处。索引指的是我们需要进行存储的对象里的对象id，而值则是对象本身。这个模式类似于使用哈希来存储数据，可以节省查找的时间。对于精通 Redux 的人来说这可能不足为奇。事实上 Redux 的作者，Dan Abramob 在他的 [Redux 教程](https://egghead.io/lessons/javascript-redux-persisting-the-state-to-the-local-storage)里也推荐这种数据结构

想象你从 REST 接口里请求到了一个列表数据，比如来自`/users`服务。我们假设决定简单的把这个纯数组存储在我们的状态中，和接口返回里的一模一样。那么当我们需要从对象里获取某个具体的用户信息时会发生什么？我们需要遍历状态状态里的所有用户。如果用户数量太多，这会是一个费时的操作。又比如我们想要追踪用户的子集，比如选中的用户或者非选中的用户又该怎么办？我们要么把用户存储为两个独立隔离的数组，要么追踪数组里被选中和非选中的用户索引

取而代之的我们决定重构我们的代码来使用索引存储数据。在 reducer 中我们应该像这样存储数据：

```javascript
{
 "usersById": {
    123: {
      id: 123,
      name: "Jane Doe",
      email: "jdoe@example.com",
      phone: "555-555-5555",
      ...
    },
    ...
  }
}
```
但这样的数据结构又是如何帮助我们解决这些问题的呢？如果我们需要查找一个特定的用户对象，我们只需要简单的像这样访问即可：`const user = state.usersById[userId]`. 这个方法不需要遍历整个数组，节省了我们的时间并且简化了检索代码

此时你或许对如何将这种数据结构的数据渲染为什么一个简单的用户列表感到疑惑。要完成这项工作，我们需要一个选择器，即一个传入状态然后返回数据的函数。一个获取状态中所有用户的简单选择器的例子是：

```javascript
const getUsers = ({ usersById }) => {
  return Object.keys(usersById).map((id) => usersById[id]);
}
```
在我们的视图代码中，我们选择器函数用户获取用户列表。然后我们遍历这些用户来产生我们的视图。我们可以编写另一个函数用于从状态中获取被选中的用户

```javascript
const getSelectedUsers = ({ selectedUserIds, usersById }) => {
  return selectedUserIds.map((id) => usersById[id]);
}
```
选择器模式同样提高了我们代码的可维护性。想象或许一段时间后我们需要改变状态的结构（shape）。如果没有选择器的话，我们需要更新所有的视图代码来响应状态结构的修改。随着视图组件的增加，更改状态结构的负担也会剧烈的增长。为了避免这个问题，在视图中我们会使用选择器来访问状态，如果底层的状态结构发生了改变，我们只需要更新选择器来保证使用正确的方式来访问状态。所有的消费方的组件依然会得到它们需要的数组而不用发生更改。基于所有这些原因，大型的 Redux 应用会从索引和选择器的存储模式中受益

## 2. 将标准状态与视图和编辑状态区分开

真实的 Redux 应用通常需要从另一个服务请求一些类型的数据，比如 REST 接口。当我们获取到数据时，我们发起一个 action, 并且附带上刚刚取得的数据。**我们倾向于把来自服务的返回数据称之为“标准状态”（canonical state）**。也就是状态中那些来自数据库中的数据。我们的状态也包括其他类型的数据，比如组件的状态，或者应用整体的状态。当我们首次从API中取得标准数据时，我们会尝试把它和页面的其他状态都存储在同一个 reducer 中。虽然这个方法或许会很方便，但是当你需要从不同的原请求更多类型的数据时，扩展起来会非常困难

另辟蹊径的，我们把标准状态隔离到它独立的 reducer 文件中去。这种方法鼓励更好的组织和模块化代码。纵向的拓展 reducer 文件（增加单个文件行数）的可维护性比横向拓展 reducer （增加更多的reducer文件供 `combineReducers`调用）的可维护性差。将 reducers 差分为独立的文件在复用它们方面也会显得更加容易。此外，它不鼓励开发者向数据对象 reducer 添加非标准状态

为什么不把其他状态和标准状态粗出在一起？想象一下我们有同样一份请求自 REST 接口的用户列表。使用上一小节的索引模式进行的存储，我们可以像这样把数据存储在 reducer 中：

```javascript

{
 "usersById": {
    123: {
      id: 123,
      name: "Jane Doe",
      email: "jdoe@example.com",
      phone: "555-555-5555",
      ...
    },
    ...
  }
}
```

现在想象我们的 UI 允许用户编辑视图。当编辑图标被用户点击时，我们需要更新我们的视图状态使得视图为用户渲染出编辑控件。我们决定将视图状态与标准状态合并，在每一个索引对象中添加一个新字段 `isEditing`，像这样：

```javascript
{
 "usersById": {
    123: {
      id: 123,
      name: "Jane Doe",
      email: "jdoe@example.com",
      phone: "555-555-5555",
      ...
      isEditing: true,
    },
    ...
  }
}
```
我们进行编辑，点击提交按钮，然后变更便通过 PUT 方法传递回 REST 服务。服务返回对象的新状态。但是我们如何将我们的新的标准状态合并到 store 中？如果我们只是根据索引赋值新的对象的话，`isEditing`标志便不复存在了。所以现在我们需要手动指定 API 的返回中哪些字段需要合并到 store 中。这让我们的更新逻辑变得复杂了。你或许有多个布尔值、字符串、数组、或者其他 UI 所需的新字段插入到标准状态中。在这个情况下，添加用于更新标准状态的 action 很简单但是容易忘记重置对象里的 UI 字段，造成无效的状态。所以我们应该保证我们的标准状态在 store 的独立的 reducer 中，并且保证我们的 action 简单并且易于追踪

另一个把编辑状态独立出来的好处是，如果用户取消了编辑，我们能很容易的回滚到标准状态。想象用户点击了编辑图标，并且已经编辑了名词和邮箱字段，现在他不想保留这些更改了，所以他点击了取消按钮。这个操作会引起视图的状态恢复到前一个状态。但是如果我们把标准状态和编辑状态合二为一，我们不再拥有旧的数据。我们不得不被迫重新从 REST 接口中请求数据再一次获得标准状态。所以现在我们把编辑状态存储到其他的地方。现在整体状态看上去像：

```javascript
{
 "usersById": {
    123: {
      id: 123,
      name: "Jane Doe",
      email: "jdoe@example.com",
      phone: "555-555-5555",
      ...
    },
    ...
  },
  "editingUsersById": {
    123: {
      id: 123,
      name: "Jane Smith",
      email: "jsmith@example.com",
      phone: "555-555-5555",
    }
  }
}
```
因为我们现在有了标准状态和（标准状态的副本）编辑状态，用户点击取消编辑之后回滚操作会变的非常简单。我们只需要使用标准状态替代编辑状态在视图中进行展示，并且不再需要请求 REST 接口。额外的，我们仍然能在 store 中追踪编辑状态。如果我们决定继续使用上次的编辑，那么只需要再一次点击编辑按钮，旧的更改随着编辑状态又会呈现出来。总的来说，保证视图和编辑状态与标准状态的分离，在代码的组织和可维护性上带来更好的开发体验，同时也给使用我们表单的用户带来了更好的交互体验。

## 3. 明智的在视图间共享状态

 许多应用在启动时只有一个用户界面和单个 store。随着功能的增长应用的也会变得庞大，我们需要管理不同视图和 store 之间的状态。为了扩展我们的 Redux 应用，为每一个页面创建一个顶级 reducer 或许是一件有益的事情。每个页面和顶级 reducer 对应于应用中的一个视图。举个例子，用户列表视图会从我们的接口请求所有用户的数据，然后存储在 `users` reducer 中，另一个负责追踪当前用户拥有域名的页面会从我们的域名接口请求数据然后储存下来，状态看起来类似于：
 
 ```javascript
 {
  "usersPage": {
    "usersById": {...},
    ...
  },
  "domainsPage": {
    "domainsById": {...},
    ...
  }
}
 ```

像这样组织页面能够使我们的视图和数据解耦且独立（self-contained）。每一个页面追踪它自己的状态，我们的 reducer 文件甚至也能和视图文件遥相呼应（co-located）。当我们继续扩展我们的应用，我们也许会发现需要在不同的视图共享它们共同依赖的状态。当考虑共享状态时请思考以下几点：

- 有多少视图或者 reducer 依赖这一份数据？
- 每个页面都需要这份数据的副本吗？
- 数据更新的频率时多少？

举个例子，我们的应用需要在每个页面展示当前登陆用户的信息。我们需要从接口中获取用户信息然后存储在 store 总。我们知道每个页面都依赖这份数据，所以这份数据并不适用于“每个页面都有独立的 reducer” 这个策略。我们也知道每个页面不需要依赖这份数据的副本，因为大多数页面不会请求其他的用户也不会修改当前的用户。而且，这份关于当前登陆用户的数据不太可能发生更改，除非他们在用户页面修改他们自己。

那么在页面间分享当前用户的状态似乎是一个好主意，所以我们把这份数据抽取出来放在处于顶级的它自己的 reducer 中。现在当用户第一次访问的页面会检查当前用户的 reducer 是否已经被加载，如果没有的话从接口进行请求。任何连接到 Redux store 的视图都能浏览这份关于当前登陆用户的信息。

对于那些共享状态没有意义的场景怎么办？让我们来考虑另一个例子。想象每一个属于用户的域名下同样也拥有一定数量的子域名。我们将新增一个子域名列表页面用户展示用户的所有子域名列表。域名列表页面同样提供展示已选域名的子域名。现在我们有两个页面同时又来子域名数据。我们也知道域名常常被修改，用户可能会在任何时候增加、删除或者编辑域名或者子域名。每个页面可能需要独一无二的数据副本。子域名页面允许通过子域名接口读或者写，而且有可能需要对数据进行翻页操作。相反域名视图只需要一次获取子域名的一部分子集（被选择的域名的子域名）。这样看来结果非常明确了，在这个场景中在不同页面共享子域名状态似乎不是一个好的选择。每个页面应该存储它自己子域名数据的副本

## 4. 跨状态的重用公共 reducer 函数

在编写了好几个 reducer 函数之后，我们决定尝试在状态中的不同地方复用我们的 reducer 逻辑。举个例子，我们或许创建了一个 reducer 从我们的接口出请求用户信息。接口每次只返回 100 个用户信息，但是在系统中有成千上万个。为了解决这个问题，我们的reducer 需要记录当前展示的是数据的哪一页。我们的请求逻辑会从 reducer 中读取该信息然后决定下一个请求的翻页参数是什么（比如叫`page_number`）。之后在请求域名列表时，我们最终也编写了相同的逻辑用于请求和存储域名信息，只是接口和对象的结构（schema）不同而已。翻页的行为仍然保持一致。聪明的开发者会意识到我们或许能够把 reducer 模块化并且在任何需要翻页的 reducer 中共享这段逻辑。

在 Redux 中共享 reducer 逻辑需要一些小技巧。默认情况下，当一个新的 action 发起时所有的 reducer 函数都会被调用。如果我们在多个 reducer 函数中共享同一个 reducer 函数，那么当 action 被发起时它会引起所有的 reducer 被触发。这不是我们重用 reducer 期望的行为。也就是说当我们请求用户列表并且取得了500条数据，我们不希望域名列表的个数也变成500

我们推荐是两种方式来实现共享，两者都针对动作类型（types）使用了特殊的作用域（scope）或者前缀（prefix）。第一种方式需要在 action 携带的信息种传递一个作用域。action 使用动作类型来推断状态中的哪个字段需要发生更改。为了便于说明，让我们假设我有一个拥有多个不同区域（section）的页面，所有都从接口处异步进行加载。我们追踪加载情况的状态像这个样子：

```javascript
const initialLoadingState = {
  usersLoading: false,
  domainsLoading: false,
  subDomainsLoading: false,
  settingsLoading: false,
};
```
有了这个状态，我们接下来需要 reducer 和 action 来控制每个区域视图加载状态。我们可以写拥有不同 action 的四个 reducer，每一个都有独立的动作类型。但那是一大堆的重复代码。取而代之的是，让我们尝试使用具有作用域的 reducer 和 action。我们只创建一个动作类型`SET_LOADING`, 和一个像这样的 reducer 函数：

```javascript
const loadingReducer = (state = initialLoadingState, action) => {
  const { type, payload } = action;
  if (type === SET_LOADING) {
    return Object.assign({}, state, {
      // sets the loading boolean at this scope
      [`${payload.scope}Loading`]: payload.loading,
    });
  } else {
    return state;
  }
}
```
我们也需要提供一个带有作用域的 action creator 函数来调用我们的作用域 reducer。action 看起来像：

```javascript
const setLoading = (scope, loading) => {
  return {
    type: SET_LOADING,
    payload: {
      scope,
      loading,
    },
  };
}
// example dispatch call
store.dispatch(setLoading('users', true));
```
通过使用一个像这样带有作用域的 reducer，我们解决需要在不同 reducer 函数和 action 中重复相同逻辑的问题。这显著降低了重复代码的数量以及帮助我们编写更小的 action 和 reducer 文件。如果我们需要在页面中添加另一个区域视图，我们只需要简单的在初始状态中添加一个新索引，然后使用不同的作用域调用`setLoading`。这个解决办法在我们拥有几个需要以同样方式更新的相似字段时非常有效

有时候尽管需要在状态的不同处共享 reducer 逻辑，不同于使用同一个 reducer 和 action 更新状态中的多个字段，我们想要在调用`combineReducers`时插拔式的重用 reducer 函数。那么 reducer 需要通过调用 reducer 工厂函数返回，它将返回一个带有类型前缀的 reducer 函数。

一个重用 reducer 逻辑很好的例子是处理翻译信息时。回到我们请求用户信息的例子中，我们的接口或许包含上千个用户。接口也将提供将用户分页之后的翻页信息。或许我们接收到的接口返回长这个样子：

```javascript
{
  "users": ...,
  "count": 2500, // the total count of users in the API
  "pageSize": 100, // the number of users returned in one page of data
  "startElement": 0, // the index of the first user in this response
  ]
}
```
如果我们想要下一页的数据，我们需要发起一个带着`startElement=100`参数的 GET 请求。我们刚好为每一个打交道的服务构建了一个 reducer 函数，但是那也意味着在我们的代码中重复了相同的逻辑。相反，我们可以创建一个独立的翻页 reducer。这个 reducer 来自 reducer 工厂函数，工厂函数接受一个类型前缀参数，然后返回一个新的 reducer 函数

```javascript

const initialPaginationState = {
  startElement: 0,
  pageSize: 100,
  count: 0,
};
const paginationReducerFor = (prefix) => {
  const paginationReducer = (state = initialPaginationState, action) => {
    const { type, payload } = action;
    switch (type) {
      case prefix + types.SET_PAGINATION:
        const {
          startElement,
          pageSize,
          count,
        } = payload;
        return Object.assign({}, state, {
          startElement,
          pageSize,
          count,
        });
      default:
        return state;
    }
  };
  return paginationReducer;
};
// example usages
const usersReducer = combineReducers({
  usersData: usersDataReducer,
  paginationData: paginationReducerFor('USERS_'),
});
const domainsReducer = combineReducers({
  domainsData: domainsDataReducer,
  paginationData: paginationReducerFor('DOMAINS_'),
});
```
reducer 工厂函数`paginationReducerFor`接收类型前缀参数，该参数将会被添加在该 reducer 函数内所有的类型前。工厂返回一个所有类型都已添加前缀的新的 reducer。现在当我们发起一个类似于 `USERS_SET_PAGINATION` 的 action 时，它只会引起用户信息的翻页 reducer 的更新。域名的翻页 reducer 仍然保持不变。这有效的让我们在 store 的多处重用 reducer 函数。为了完整性，这有一个带有前缀的 action creator 工厂函数：

```javascript
const setPaginationFor = (prefix) => { 
  const setPagination = (response) => {
    const {
      startElement,
      pageSize,
      count,
    } = response;
    return {
      type: prefix + types.SET_PAGINATION,
      payload: {
        startElement,
        pageSize,
        count,
      },
    };
  };
  return setPagination;
};
// example usages
const setUsersPagination = setPaginationFor('USERS_');
const setDomainsPagination = setPaginationFor('DOMAINS_');
```

## 5. 整合 React

有一些 Redux 应用永远也不需要给用户渲染视图（像接口一样），但是大部分情况下你需要视图将数据渲染出来。目前最受欢迎的与 Redux 配合的渲染 UI 类库是 React，这也是我们接下来用于展示如何与 Redux 整合的 UI 类库。我们可以使用上面几个小节中学习到的策略来让我们的视图代码更友好。为了实现整合，我们将使用 `react-redux` 类库

UI 整合的一个有用的模式是在视图中使用访问器访问状态中的数据, 在`react-redux`中便于放置访问器的地方是`mapStateToProps`函数中。这个函数被传递进`connect`函数（用于将 React 组件连接至 Redux store 的函数）调用的过程中。在这里你能将状态中的数据映射为组件接收到的属性。这是一个完美的使用选择器从状态获取数据，然后以属性的形式传递给组件的地方。整合的例子如下：

```javascript
const ConnectedComponent = connect(
  (state) => {
    return {
      users: selectors.getCurrentUsers(state),
      editingUser: selectors.getEditingUser(state),
      ... // other props from state go here
    };
  }),
  mapDispatchToProps // another `connect` function
)(UsersComponent);
```
这种在 React 和 Redux 之间整合也为我们提供了使用作用域和类型封装 action 的场所。我们要使得组件的处理函数有能力唤起 store 调用 action creator。为了完成这项任务，在`react-redux`中调用`connect`时，我们传入`mapDispatchToProps`函数。函数`mapDispatchToProps`是我们调用 Redux 的 `bindActionCreators` 函数用于将咩歌 action 和 dispatch 方法绑定起来的地方。在其中我们还可以像上一节展示的那样给 action 绑定作用域。举个例子，如果我们在用户列表页面中使用带有作用域的 reduer 模式的翻页功能，代码如下所示：

```javascript
const ConnectedComponent = connect(
  mapStateToProps,
  (dispatch) => {
    const actions = {
      ...actionCreators, // other normal actions
      setPagination: actionCreatorFactories.setPaginationFor('USERS_'),
    };
    return bindActionCreators(actions, dispatch);
  }
)(UsersComponent);
```

现在从我们`UsersPage`组件的角度来说，它接收到了用户列表和其他的状态碎片，以及被绑定的 action creator 作为属性传递给它。组件不需要关心它需要带有什么作用域的 action 又或者如何访问状态；在整合的层面我们已经对这些问题进行了处理。这种机制让我们能够创建不需要依赖状态内部工作机制的非常松耦合的组件。希望借助我们在这里讨论的各类模式，我们都能以可伸缩，可维护，以及合理的方式创建 Redux 应用。

**更多参考**：

- 刚刚讨论的状态管理类库 [Redux](http://redux.js.org/)
- 用于创建选择器的 [Reselect](https://github.com/reactjs/reselect) 类库
- [Normalizr](https://github.com/paularmstrong/normalizr) 是一个用于「扁平化」（normalizing）JSON 数据的类库。对使用索引存储数据非常有帮助
- 用于在 Redux 中使用异步 action 的中间件类库 [Redux-Thunk](https://github.com/gaearon/redux-thunk) 
- 使用 ES2016 generator 实现的异步 action 的另一个中间件类库 [Redux-Saga](https://github.com/redux-saga/redux-saga)