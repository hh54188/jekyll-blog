Bus在这里不是公交车的意思，在计算机领域中应该把它翻译为“总线”。就算Bus是车，今天我也不是老司机。

## Command Bus

如果你是一名PHP开发者，一定对Command Bus不陌生。因为在目前我查阅到的资料当中，对Command Bus描述大多的都存在于PHP相关的编程文章当中，例如著名的Laravel框架已经集成了Command Bus。

第一个问题是，什么是Command（以下我们会用中文“指令”代替）？

指令代表的是用户的操作意图，指令的作用是将用户意图与它相关的实现技术隔离开。指令传递的只是一段信息，它的数据结构可以只是一个对象。比如我们定义一个登陆指令：

```
class LoginCommand
{
	constructor(name, password) {
		this.name = name;
		this.password = password;
	}
}

let loginCommand = new LoginCommand('liguangyi', '123456');
```

这个登陆指令只说了三件事：1.我要登陆，2.用户名是liguangyi，3.密码是123456 。它并没有包含任何有关登陆调用方法，登陆函数等技术信息。

很明显，系统中除了发出指令的一方，还需要接受并且处理指令的一方，这个接收方就是我们的主人公command bus，command bus有一个handle函数用于处理指令。所以，发出指令到接收指令的处理流程应该是：

```
let loginCommand = new LoginCommand('liguangyi', '123456');
commandBus.handle(loginCommand);
```

注意command bus与command是一一对应的关系，而非所有的command都交由统一的command bus处理。

为了更准确的说明，不如我们为这段代码补充一些上下文，假设当前代码运行在一个MVC架构的程序中，这个MVC框架我们取用Node.js的Kraken，很明显当前代码应该是运行在controller角色的脚本中，用于转发用户的请求。那么代码可以是：

```
module.exports.loginController = function (req, res, next) {
	let loginCommand = new LoginCommand(
		req.body.username, 
		req.body.password
	);
	this.commandBus.handle(loginCommand);
};
```

然而commandBus.handle究竟做了哪些事情，这也很容易推敲出来，最简单的情境是，首先对用户的输入进行验证，验证通过之后进行登录操作。类似传统代码的编写方式大致如此：

```
module.exports.loginController = function (req, res, next) {
	UserModel.validate(req.body.username, req.body.password);
	UserModel.login(req.body.username, req.body.password);
};
```

### Command Bus，或者说这种模式带来什么好处？

我们从以上代码中至少能总结出以下几点：

1. 职责划分更加明确

在上面举例的传统代码方式中，`loginController` 是包含登录逻辑的。而通常在设计中我们推崇的是fat model, skinny controller，也就是业务逻辑尽可能的封装在Model层中，Controller只负责转发来自View的请求。如果你对MVC有所了解的话，View与Controller其实是一一对应的关系，有点勉强的说，这样的代码实际上我们把业务职责嫁接给了View层，或者说是污染了View层。因为仔细想想，登录这件事和用户操作界面一点关系都没有，无论是网页还是WebForm还是命令行。

显而易见的是如果使用command bus的方式对业务进行封装，Controller就变得更纯粹了一些，当然如果想修改登录逻辑时，我们不必去打扰View，不必去Controller里查找，而是去command bus里修改。

2. 复用性更好

当我们把command抽象为一个类时，就意味着这个类可以任意处复用，command可以在任意处创建。发出登录指令这件事不仅限于网页的Controller中，可以来自命令行，也可以来自API，只要是能创建command实例的地方都行。

同理，对于command bus来说，代码也可以在多处复用。但更重要的是command bus无需再关心它外面的世界，它不用关心自己身在何处，也不用关心传递的指令来自何处。

最后这样的封装对于写单元测试也是极大的便利。

关于Command Bus我们就谈到这里，更多有关Command Bus的实现就不说了，有兴趣的同学可以参考文章最后的参考文献。

## Event Bus

Event Bus机制很好理解，它就是普通事件机制（设计模式中观察者模式）的升级版。

事件机制由事件的发布者、订阅者和事件本身组成。发布者发布事件，订阅者订阅感兴趣的事件；一个事件可以拥有多个订阅者，一个订阅者可以订阅多个事件。一旦发布者发布了某个事件，订阅该事件的订阅者就会得到通知，并且执行有关该事件的回调函数。

前端开发者对事件一定不陌生，例如 `document.addEventListener("click", clickHandler)`，表示我们订阅了doucment上的点击click事件，如果该事件发生，调用clickHandler回调函数；或者以Node.js为例，`process.on("message", messageHandler)`，表示当前子进程在等待父进程传递来的消息，一旦有消息传到，就会触发message事件，调用messageHandler函数处理消息。

如果说在浏览器或者Node.js中使用事件是被迫基于浏览器（引擎）决定的。那么在Java平台使用Event Bus则是完全自愿的为了解决通信问题。

Event Bus把事件机制上升到了一个系统级别的高度，让事件成为不同组件或者服务之间通信的必要渠道。任何一方都可以是事件的发布者也可以是订阅者




- [A wave of command buses](http://php-and-symfony.matthiasnoback.nl/2015/01/a-wave-of-command-buses/)
- [Responsibilities of the command bus](http://php-and-symfony.matthiasnoback.nl/2015/01/responsibilities-of-the-command-bus/)
- [Towards CQRS, Command Bus](https://gnugat.github.io/2016/05/11/towards-cqrs-command-bus.html)
- [What is an Eventbus?](http://www.rribbit.org/eventbus.html)
- [Why people use message/event buses in their code?](http://stackoverflow.com/questions/3987391/why-people-use-message-event-buses-in-their-code)