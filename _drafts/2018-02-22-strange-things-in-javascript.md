# Javascript 奇怪事件簿

javascript 中从来就没有什么奇怪的事件，我只是想梳理一下 javascript 中让人疑惑的表达式以及背后的原理。

你们可以先把下面的题目做了，说出这些表达式的结果即可：

* `1 + '1'`
* `1 - '1'`
* `'2' + '2' - '2'`
* `[] + []`
* `{} + {}`
* `[] + {}`
* `{} + []`
* `[] + {} === {} + []`
* `{} + [] === [] + {}`
* `[+false] + [+false] + [+false]`
* `[+false] + [+false] + [+false] - [+false]`
* `[ [] [ [] ] + [] ] [ +[] ] [++ [ +[] ] [ +[] ]]`
* `'1' == true`
* `1 < 2 < 3`
* `3 > 2 > 1`
* `isNaN(false)`
* `isNaN(null)`
* `parseInt('infinity') == 0 / 0`
* 以及这个表达式想要实现的功能 `x = x => x <= x - x ?- x :x`

如果想知道正确答案的话把表达式粘贴到浏览器的控制台执行即可

接下来的内容就是讲解这些表达式的结果是在什么样的原理下得出的

解决以上的问题的关键在于要搞明白三点：

1. 数据类型的转化规则
2. 操作符的使用方法和优先级
3. 语法中的特例

## `+` 操作符

`+`在 JavaScript 中有两个作用：

1. 连接字符串：`var result = 'Hello' + 'World'`
2. 计算数字之和：`var result = 1 + 2`
3. 作为一元操作符：`+variable`

在表达式中`+`是操作符（operator），操作符操作的对象（上面例子中的`Hello`、 `World`、 `1`、 `2`）名为操作数（operand）

一元`+`操作符的运算规则是：`ToNumber(ToPrimitive(operand))`，也就是把任意类型都转化为数字类型。

当操作数的数据类型不一致时，会根据以下规则进行转化：

* 如果至少一个操作数是对象数据类型（`object`），则需要将它转化为基础类型（`primitive`），即字符串、数字或者布尔
  1. 如果对象是`Date`类型，那么调用`toString()`方法
  2. 否则优先调用 `valueOf()` 方法
  3. 如果`valueof()`方法不存在或者并没有返回一个基础类型，那么调用`toString()`
  4. 当数组转化为基础类型时，JavaScript 会使用`join(',')`方法
  5. 单纯的 Javascript 对象 `{}` 转化的结果是 `[object Object]`
* 转化之后，如果至少一个操作数是字符串类型，那么另一个操作数也需要转化为字符串类型，然后执行连接操作
* 在其他的情况下，两个操作数都转化为数值类型，并且执行加法操作
* 如果两个操作数都是基础类型，操作符会判断至少一个是字符串类型并且执行连接操作。其他情况都转化为数字并且求和

所以根据以上规则，我们就能解释：

* `1 + '1'` 的结果是 `'11'`，因为其中一个是操作数是字符串，所以另一个操作数也被转化为字符串，并且执行字符串连接操作
* `[] + []` 的结果是 `''` 空字符串，因为数组是对象类型，转化为基础类型的结果是空字符串，拼接之后仍然是空字符串
* `[] + {}` 的结果是 `[object Object]`，因为操作数有对象类型的关系，两个操作数都需要转化为基础类型，`[]`转化为基础类型的结果是`''`，`{}`转化为基础类型的结果是`[object Object]`，最后字符串拼接的结果仍然是`[object Object]`

接下来我们说一说特殊情况

* `{} + []` 的结果是`0`。因为在这个表达式中，开头`{}`并不是空对象的字面量的意思，而是被当作空的代码块。事实上这个表达式的值就是`+[]`的结果，即`Number([].join(','))`，即为`0`

* 更奇怪的是`{} + {}`这个表达式，在不同的浏览器中执行会得到不同的结果。
按照上面的例子，我们可以同理推出这个表达式的值实际上是`+{}`的值，即最后的结果是`Number([object Object])`，即`NaN`。在 IE 11 中的执行结果却是是如此，但是如果在 Chrome 中执行，你得到的结果是 `[object Object][object Object]`。

根据 [Stackoverflow上的回答](https://stackoverflow.com/questions/36438034/why-is-no-longer-nan-in-chrome-console) 这是因为 Chrome devtools 在执行代码的时候隐式的给表达式添加了括号`()`，实际上执行的代码是`({} + {})`。如果你在 IE 11 执行上述代码，就会得到`[object Object][object Object]`的结果

* 虽然上面我们已经明确了 `[] + {}` 的结果是 `[object Object]`，而 `{} + []` 的结果是`0`，但是如果把他们进行比较的话：`[] + {} === {} + []`结果会是`true`。因为右侧的`{}`跟随在`===`之后的关系，不再被认为是空的代码块，而是字面量的空对象，所以两侧的结果都是`[object Object]`

* `{} + [] === [] + {}` 同样是一个有歧义的结果，理论上来说表达式的返回值是`false`，在 IE 11 中确实如此，但是在 Chrome 的 devtools 中返回 `true`，原因仍然是表达式被放在`()`中执行

* `[+false] + [+false] + [+false]`的结果也可想而知了，`+false`的结果是将`false`转化为数字`0`，之后`[0]`又被转化为基础类型字符串`'0'`，所以表达式最后的结果是`'000'`

## `-`操作符

虽然`-`操作符和`+`操作符看看上去性质相同，但`-`操作符只有一个功能，就是数值上的加减。它会尝试把非数值类型的操作数转化为数值类型，如果转化的结果是`NaN`, 那么表达式的结果可想而知也就是`NaN`，如果全部都转化成功，n

## 参考文章

- [JavaScript addition operator in details](https://dmitripavlutin.com/javascriptss-addition-operator-demystified/)
- [The legend of JavaScript equality operator](https://dmitripavlutin.com/the-legend-of-javascript-equality-operator/)
- [What is the explanation for these bizarre JavaScript behaviours mentioned in the 'Wat' talk for CodeMash 2012?](https://stackoverflow.com/questions/9032856/what-is-the-explanation-for-these-bizarre-javascript-behaviours-mentioned-in-the)
- [Why is {} + {} no longer NaN in Chrome console?](https://stackoverflow.com/questions/36438034/why-is-no-longer-nan-in-chrome-console)