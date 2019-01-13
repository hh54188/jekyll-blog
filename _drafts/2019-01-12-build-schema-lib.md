# 不如自己写一个 schema 类库吧

这篇文章里没有传达高深技巧和经验，记录的是一个想法从诞生到实现的过程

## 背景需求

在上一篇文章 [构建大型 Mobx 应用的几个建议](https://zhuanlan.zhihu.com/p/54291246) 中，我提到过使用 schema 来约定数据结构。但遗憾的事情是，在浏览器端，我一直没有能找到合适的 schmea 类库，所以只能用 Immutable.js 中的 Record 代替。

如果你还不了解什么是 schema，我在这里简单解释一下: 在应用内部的不同组件之间，应用端与服务端之间，都需要使用消息进行通信，而随着应用复杂度增长，消息的数据结构也变得复杂和庞大。对每一类需要使用的消息或者对象提前定义 schema，有利于确保通信的正确性，防止传入不存在的字段，或者传入字段的类型不正确；同时也具有自解释的文档的作用，有利于今后的维护。我们以 [joi](https://github.com/hapijs/joi) 类库为例

```javascript
const Joi = require('joi');

const schema = Joi.object().keys({
    username: Joi.string().alphanum().min(3).max(30).required(),
    password: Joi.string().regex(/^[a-zA-Z0-9]{3,30}$/),
    access_token: [Joi.string(), Joi.number()],
    birthyear: Joi.number().integer().min(1900).max(2013),
    email: Joi.string().email({ minDomainAtoms: 2 })
}).with('username', 'birthyear').without('password', 'access_token');

// Return result.
const result = Joi.validate({ username: 'abc', birthyear: 1994 }, schema);
// result.error === null -> valid

// You can also pass a callback which will be called synchronously with the validation result.
Joi.validate({ username: 'abc', birthyear: 1994 }, schema, function (err, value) { });  // err === null -> valid
```

但是就像我在 [npm](https://www.npmjs.com/search?q=schema) 上能找到的所有 schema 类库类似，它们始终在采取一种“事后验证”机制，即事先定义 schema 之后，再将需要验证的对象交给 schema 进行验证，这是让我不满意的。我更希望采取 Reacord 的方式：

```javascript
const Person = Record({
  name: '',
  age: ''
})
const person = new Person({
  name: 'Lee',
  age: 22,
})
const team = new List(jsonData).map(Person) // => List<Person>
```
在上面的例子中，schema 俨然拥有了类似于“类”的功能，你能够使用它创建指定数据结构的实例。如果你在创建实例时传入的属性没有事先定义便会报错。但是美中不足的是，Record 不支持更进一步的对每个字段进行约束，指定类型、最大值最小值灯，就像在 joi 里看到的那样。

介于找不到令我们满意的 schema 类库，不如我们自己编写一个。它需要具备以下两种能力：
- 能够根据 schema 创建实例，而不是事后验证
- 支持对 schema 定义时字段的约束

## 设计用法

在开发之前，我们需要考虑并且约定将来如何使用它。关于这一点在上一小节中我们已经得出初步的结论了。

假设类库名为 Schema

- 创建 Schema：
```javascript
const PersonSchema = Schema({
  name: '',
  age: ''
})
```
虽然我们支持对字段约束，但是你可以不需要约束。那么采用以上的方式即可，仅仅约定了 schema 的字段名词，以及默认值

- 实例化 Schema:
```javascript
const person = PersonSchema({
  name: 'Lee',
  age: 22
})
```

- 对字段进行约束：
```javascript
const PersonSchema = Schema({
  name: Types().string().default('').required(),
  age: Types().number().required()
})
```
解释一下，理想状态下应该使用 React 中`PropTypes`的接口方式对字段进行约束，例如`PropTypes.func.isRequired`，但是一时想不到如何实现，于是提供`Types`类的方式曲线救国，可以约束的条件如下：

- 数据类型约束
  - `string()`: 仅限字符串类型
  - `number()`: 仅限数字类型
  - `boolean()`: 仅限布尔类型
  - `array()`: 仅限数组类型
  - `object()`: 仅限对象类型
- 其他约束
  - `required()`: 该字段创建实例时必传
  - `default(value)`: 该字段的默认值
  - `valueof(value1, value2, value3)`: 该字段值必须是 value1, value2, value3 值之一

当然还可以添加其他种类的约束，比如`min()`、`max()`、`regex()`等等，这些二期再实现，以上才是目前来说看来是最重要

- 支持 schema 嵌套
```javascript
const PersonSchema = Schema({
  name: Types().string().default('').required(),
  age: Types().number().required(),
  job: Schema({
    title: '',
    company: ''
  })
})
```

## 实现

### `Types`

首先关于 Types 的链式调用 `Types().string().required()` 让我想到了什么？jQuery. jQuery 是如何实现链式调用的？函数调用的结束始终返回对 jQuery 的引用。我们也可以这么做，用于实现链式调用

`Types`是一个类，`Types()`用于生成一个实例。你可能注意到没有使用关键词`new`，因为我认为使用关键词`new`是很鸡肋很累赘的事情。技术上不使用`new`关键词生成实例也很容易，只要 1) 不用使用 `class` 定义类，使用函数 2) 在构造函数中添加对实例的判断：

```javascript
function Types() {
  if (!(this instanceof Types)) {
    return new Types();
  }
}
```
而至于对各种数据类型的验证，我们借助并且封装`lodash`的方法进行实现。用户每执行一个验证函数，我们会生成一个内部的验证函数，存储在 `Types` 实例的 `validators` 变量中，用于将来对该字段值的判断

```javascript
import _ from 'lodash'

const lodashWrap = fn => {
  return value => {
    return fn.call(this, value);
  };
};

function Types() {
  if (!(this instanceof Types)) {
    return new Types();
  }
  this.validators = []
}

Types.prototype = {
  string: function() {
    this.validators.push(lodashWrap(_.isString));
    return this;
  },
```

同理，我们也实现了`default`、`required`和`valueof`

```javascript

function Types() {
  if (!(this instanceof Types)) {
    return new Types();
  }
  this.validators = [];
  this.isRequired = false;
  this.defaultValue = void 0;
  this.possibleValues = [];
}


Types.prototype = {
  default: function(defaultValue) {
    this.defaultValue = defaultValue;
    return this;
  },
  required: function() {
    this.isRequired = true;
    return this;
  },
  valueOf: function() {
    this.possibleValues = _.flattenDeep(Array.from(arguments));
    return this
```

### `Schema`

通过我们之前约定的 `Schema()` 的用法我们不难判断出 `Schema` 的基本结构应该如下：

```javascript
export const Schema = definition => {
  return function(inputObj = {}) {
    return {}
  }
}
```

`Schema` 的代码实现中绝大部分并没有什么特别的，在 `definition` 中我们得到了schema的定义，即对每个字段（key）的约束。通过对字段值的各种判断，就能得到用于想表达的约束信息：

- 如果值不是 `Types` 的实例，表示用户只是定义了字段，但并没有对它进行约束，同时当前值也是默认值。在创建实例或者对实例进行写操作时不需要任何校验
- 如果值是 `Types` 实例，那么我们就能从实例的属性里取得各种约束信息，就是之前`Types`定义里的意义`validators`、`defaultValue`、`isRequired`、`possibleValues`
- 如果值是函数，表示用户定义了一个嵌套的 Schema，在校验时需要使用这个定义的 Schema 进行校验

`Schema`类的实现关键在于如何实现`set`访问器，即如何在用户给字段赋值时进行校验，校验通过之后才允许赋值成功。关于如何实现访问器，我们有两种方案进行选择：
- 使用 `Object.defineProperty` 定义对象的访问器
- 使用 Proxy 机制

`Object.defineProperty`的本质是对对象进行修改（当然你也能够深度拷贝一份原对象再进行修改，以避免污染）；而 Proxy 从“语义”上来说更适合这个场景，也不存在污染的问题。并且在同时尝试了两个方案之后，使用 Proxy 的成本更低。于是决定使用 Proxy 机制，那么代码结构大致变为：

```javascript
export const Schema = definition => {
  return function(inputObj = {}) {
    const proxyHandler = {
      get: (target, prop) => {
        return target[prop];
      },
      set: (target, prop, value) => {

      }
    }
    return new Proxy(Object.assign({}, inputObj), proxyHandler);
  }
}
```

## 结束语

本文的源码在 [https://github.com/hh54188/schemaor](https://github.com/hh54188/schemaor)

你可以拷贝它，和它玩耍，测试它，修改它。**但千万不要将它用在生产环境中**，它还没有经过充分的测试，以及还有很多细枝末节和边界情况需要处理

欢迎通过 [pull request](https://github.com/hh54188/schemaor/pulls) 和 [issues](https://github.com/hh54188/schemaor/issues) 提出更多的建议