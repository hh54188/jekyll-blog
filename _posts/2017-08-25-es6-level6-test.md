---
layout: post
title: ECMAScript 6 六级考试
modified: 2017-08-25
tags: [javascript]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

这份试卷我并没有准备答案，一方面所有的代码可以立即在浏览器中执行看到（我）效（很）果（懒）；另一方面这也并非是一场真的（我）考（很）试（懒），而是用于检测自己ES6知识并且查漏补缺，所以不必有太大压力。

这份试卷的内容主要来源于两份我学习ES6的资料：nzakas（JavaScript高级程序设计的作者）的开源图书[understandinges6](https://github.com/nzakas/understandinges6)，以及Nicolás Bevacqua的[ES6 in Depth](https://ponyfoo.com/articles/tagged/es6-in-depth)系列。如果你对文中的一些问题有疑惑，你一定可以从它们中找到答案。

一共44道题，7道选择题，8道问答题，29道代码执行题。这些题目当然不会是省油的灯，我尽量挑选模棱两可，让人疑惑又或者容易疏忽的知识点

这份试卷难免会有纰漏或者不严谨的地方，比如没有正确答案或者答案不唯一，如果有出现这样的情况，请留言给我让我及时纠正。

最后，如果你有兴趣的话可以把你认为的正确答案或者解释写在留言中供大家参考

## 选择题（有一个或多个答案）

- 以下关于`...rest`参数说法错误的是
	- 函数只能有一个`...rest`参数
	- `...rest`参数后允许再接其他参数
	- 对象的方法不能有`...rest`参数
  - rest参数不影响`arguments`对象，`arguments`反映的还是实际传参（长度，索引等）

- 以下关于 weakset 说法错误的是
	- 调用`add`方法时传递的必须是object数据类型
	- 可以通过 `for of` 进行遍历
	- 没有 keys 和 values 方法
	- 没有 size 属性

- 以下关于 Symbol 说法正确的是
  - Symbol 不是基础数据类型(primative type)
  - Symbol 数据类型可以转化为字符串或者数字
  - 必须使用 new 关键字创建 Symbol 数据类型
  - 以上说法全是错误

- 关于 module 以下说法错误的是
  - import 命令使用的是解构(Destructuring)语法
  - 一个模块只能导出一个 default value
  - 在引入defaut值和非default值时，default值必须放在前面

- 以下关于`super()`方法说法正确的是
  - 在使用`this`关键字之前必须调用`super()`方法
  - 只能在子类中调用`super()`函数
  - 如果你在子类中定义了构造函数，你必须调用`super()`方法
  - 如果你没有指定构造函数，`super()`方法也会自动为你调用

- 关于迭代器（iterator）以下说法正确的是
  - `map[Symbol.iterator] === map.entries`结果返回 true
  - 可以通过 `Symbol.iterator` 来访问变量的默认迭代器
  - arrays 和 sets 默认的迭代器是 values
	- weakmap 和 weakset 没有默认的迭代器

- 以下代码不会报错的是
  - `Object.assign([1, 2, 3], [4, 5])`
  - `var foo = 'bar'; var baz = { [foo] };`
  - `map[Symbol.iterator] === map.entries`
  - `let person = new class { //... }("Nicholas");`

## 请说出以下每一段代码的执行结果，或者是否会报错

```javascript
if (true) {
	console.log(temp);
	console.log(typeof temp);
	const temp = 1;
}
```

```javascript
let message = `Multiline
		           string`;
console.log(message)
```

```javascript
[1, 2, 3].map(n => { number: n })

```

```javascript
function mixArgs(first, second = "b") {
    console.log(arguments.length);
    console.log(first === arguments[0]);
    console.log(second === arguments[1]);
}
mixArgs();
```

```javascript
const a = { b: 'c'}
Object.defineProperty(a, 'visible', { enumerable: false, value: 'boo! ahhh!' })
Object.assign({}, a);
```

```javascript
var a = [[1], [2], [3]];
var b = [...a];
b.shift().shift();
console.log(a);
```

```javascript
let obj1 = { a: 0 , b: { c: 0}};
let obj2 = Object.assign({}, obj1);
obj1.b.c = 2;
console.log(JSON.stringify(obj2))
```

```javascript
const x = { a: { a: 1 } };
const y = { a: { b: 1 } };
const z = { ...x, ...y };
console.log(z);
```

```javascript
var obj = {
    a: 1,
    0: 1,
    c: 1,
    2: 1,
    b: 1,
    1: 1
};
obj.d = 1;
console.log(Object.getOwnPropertyNames(obj).join(""));
```

```javascript
let node = {
        type: "Identifier"
    };
let { type: localType = 'foo', name: localName = 'bar' } = node;

console.log(localType);
console.log(localName);  
```

```javascript
let node = {
        type: "Identifier",
        name: "foo"
    },
    type = "Literal",
    name = 5;
({ type, name } = node);
console.log(type); 
console.log(name); 
```

```javascript
let a = 1,
    b = 2;

[ a, b ] = [ b, a ];

console.log(a);
console.log(b);
```

```javascript
var {foo=3} = { foo: 2 }
console.log(foo)

var {foo=3} = { foo: undefined }
console.log(foo)

var {foo=3} = { bar: 2 }
console.log(foo)

var [b=10] = [undefined]
console.log(b)
```

```javascript
let uid3 = Symbol("uid");
console.log(Symbol.keyFor(uid3));
```

```javascript
function MyObject() {
    // ...
}

Object.defineProperty(MyObject, Symbol.hasInstance, {
    value: function(v) {
        return false;
    }
});

let obj = new MyObject();
console.log(obj instanceof MyObject);
```

```javascript
let set = new Set([1, 2, 3, 4, 5, 5, 5, 5]);
console.log(set.size);
```

```javascript
var map = new Map()
map.set('a', 'a')
map.set('a', 'b')
map.set('a', 'c')
console.log([...map])
```

```javascript
var map = new Map([[NaN, 1], [Symbol(), 2], ['foo', 'bar']])
console.log(map.has(NaN))
console.log(map.has(Symbol()))

var sym = Symbol()
var map = new Map([[NaN, 1], [sym, 2], ['foo', 'bar']])
console.log(map.has(sym))
```

```javascript
function* generator () {
  yield 'p'
  console.log('o')
  yield 'n'
  console.log('y')
  yield 'f'
  console.log('o')
  yield 'o'
  console.log('!')
}

var foo = generator()
for (let pony of foo) {
  console.log(pony)
}

var foo = generator()
console.log([...foo])
```

```javascript
function *gen() {
    yield 5 + 2;
}
const g = gen();
g.next()
```

```javascript
function *createIterator() {
    let first = yield 1;
    let second = yield first + 2;  
    yield second + 3;                   
}

let iterator = createIterator();

console.log(iterator.next());
console.log(iterator.next(4));
console.log(iterator.next(5));
console.log(iterator.next());
```

```javascript
function *createIterator() {
    yield 1;
    return;
    yield 2;
    yield 3;
}

let iterator = createIterator();

console.log(iterator.next());
console.log(iterator.next());
```

```javascript
let PersonClass = class PersonClass2 {
  //...
};

console.log(typeof PersonClass)
console.log(typeof PersonClass2)
```

```javascript
let propertyName = 'sayHi';
class Test {
    [propertyName]() {
        console.log('Hello World');
    }
}
const test = new Test();
test.sayHi();
test[propertyName]();

propertyName = 'sayBye';
const test2 = new Test();

test.sayHi();
test[propertyName]();
```

```javascript
let promise = new Promise(function(resolve, reject) {
    console.log("Promise");
    resolve();
});

promise.then(function() {
    console.log("Resolved.");
});

console.log("Hi!");
```

```javascript
let p1 = new Promise(function(resolve, reject) {
    resolve(42);
});

let p2 = new Promise(function(resolve, reject) {
    resolve(43);
});

p1.then(function(value) {
    console.log(value);
    return p2;
}).then(function(value) {
    console.log(value);
});
```

```javascript
// example.js:
export var name = "Nicholas";
export function setName(newName) {
    name = newName;
}

// another_example.js:
import { name, setName } from "./example.js";

console.log(name);
setName("Greg");
console.log(name);
```

```javascript
let target = {
    name: "target"
};
let { proxy, revoke } = Proxy.revocable(target, {});

console.log(proxy.name);
revoke();
console.log(proxy.name);
```

```javascript
let ints = new Int16Array([25, 50]);

console.log(ints.length);
console.log(ints[0]); 
console.log(ints[1]); 

ints[2] = 5;

console.log(ints.length);
console.log(ints[2]); 
```

## 问答题

- `import` vs `require`
- 请说出 spread operator 六种以上的用法
- `Number.isNaN` 和 `Global.isNaN` 有什么区别
- `Object.getPrototypeOf()` 和 `Reflect.getPrototypeOf()` 有哪些差异
- `Array.from` 与 `Array.of` 有什么区别
- 解释一下什么是 code unit 以及 code point
- `charCodeAt` 和 `codePointAt` 有什么区别
- `fromCharCode` 和 `fromCodePoint` 有什么区别
- 假设一个数组中有包含一系列promise对象，如何让他们依次执行（在一个成功执行之后再执行下一个）？