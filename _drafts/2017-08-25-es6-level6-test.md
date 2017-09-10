# ECMAScript 6 六级考试

这份试卷我并没有准备答案，一方面所有的代码可以立即在浏览器中执行看到（我）效（很）果（懒）；另一方面这也并非是一场真的（我）考（很）试（懒），而是用于检测自己ES6知识并且查漏补缺，所以不必有太大压力。

这份试卷的内容主要来源于两份我学习ES6的资料：nzakas（JavaScript高级程序设计的作者）的开源图书[understandinges6](https://github.com/nzakas/understandinges6)，以及Nicolás Bevacqua的[ES6 in Depth](https://ponyfoo.com/articles/tagged/es6-in-depth)系列。如果你对文中的一些问题有疑惑，你一定可以从它们中找到答案

这份试卷难免会有纰漏，比如没有正确答案或者答案不唯一，如果有出现这样的情况，请留言给我让我及时纠正。

最后，如果你有兴趣的话可以把你认为的正确答案或者解释写在留言中供大家参考


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