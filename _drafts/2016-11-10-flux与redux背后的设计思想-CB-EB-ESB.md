Bus在这里不是公交车的意思，在计算机领域中应该把它翻译为“总线”。

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

这个登陆指令只说了三件事：1.我要登陆，2.用户名是liguangyi，3.密码是123456 。它并没有包含任何有关登陆方法，登陆函数等技术信息。

很明显，系统中除了发出指令的一方，还需要接受并且处理指令的一方，这个接收方我们就称之为command bus，command bus有一个handle函数用于处理指令。所以，发出指令到接收指令的处理流程应该是：

```
let loginCommand = new LoginCommand('liguangyi', '123456');
commandBus.handler(loginCommand);
```

不如我们为这段代码补充一些上下文，假设当前代码运行在一个MVC架构的程序中，这个MVC框架我们取用Node.js的Kraken，很明显当前代码应该是运行在controller部分中。那么代码可以是：

```
module.exports.loginController = function (req, res, next) {
	let loginCommand = new LoginCommand(
		req.body.username, 
		req.body.password
	);
	this.commandBus.handle(loginCommand);
};
```

### Command有什么好处？

1. 指令可以在任意处创建，然后只要把它交给command bus就好了
2. controller不再包含登陆逻辑

controller不再包含过多的业务逻辑了，它只要把http请求翻译成指令对象，交给command bus去处理