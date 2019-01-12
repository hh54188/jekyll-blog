# 不如自己写一个 schema 类库吧

这篇文章里没有传达高深技巧和经验，记录的是一个想法从诞生到实现，以及分享我在编码过程中的喜悦

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