---
layout: post
title: Copilot 结对编程指南
modified: 2024-01-08
tags: [other]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

在和 Copilot 一起陆续写完一个约三千行代码的项目之后，我对它的脾气秉性终于有了大致的了解。这里是关于使用它的技巧，以及穿插的聊聊哪些是它能做的，善于做的，还有做不到的。

如果你问我 copilot 可以多大程度上取代程序员，我的答案是至多可以为程序员完成一半的工作。这并不是意味这公司有一半的程序员可以被开除，而是说程序员的工作可以事半功倍的完成。如果说我们都同意编程本质上和写作无异，都是一类将想法输出的过程，那 copilot 能为你做的有且只有后半部分，想法的塑造依然要依靠你自己。

阅读完以下的 copilot 的使用（结对）技巧你就会对我的上述结论有更深刻的理解。文中我会保留所有 copilot 生成的内容，以便各位对它做进一步独立评估

### 给予明确的指示

你可以把 copilot 当作非常靠谱的入门程序员，它也许没法给你一些设计、架构上的建议，但它可以很好的完成你拆分给它的子任务。最常见 copilot 的使用方式就是给予它明确且详细的任务指令，比如在注释中给出以下任务：

```javascript
// Write a block of code using JS fetch API to get data from the server with these conditions:
// 1. Request method: GET
// 2. Request URL: http://localhost:3000/api/v1/contacts
// 3. Request header: Content-Type: application/json:
// 4. Convert the response to JSON format
// The code is:
```

随后 VSCode 便会生成代码

```javascript
fetch('http://localhost:3000/api/v1/contacts', {
    method: 'GET',
    headers: {
        'Content-Type': 'application/json'
    }
})
    .then(res => res.json())
    .then(data => console.log(data))
    .catch(err => console.log(err));
```

这对于写一些你常用、并且大致有印象、但是又不常记于心的代码非常好用。比如虽然我常写 SQL 查询语句，但是对创建或者删除数据库操作并不熟悉，于是我可以直接让它告诉我：

```javascript
-- Write a SQL script to drop the database "example" and all tables in it:
DROP DATABASE IF EXISTS example;
```

指令编写的越详细越好，比如下述指令 `Write a function to replace the new line and space characters with a <br /> tag.` 就会比该指令 `Write a function to replace the ending characters with a <br /> tag.` 要好，因为你明确指出了 ending characters 究竟是什么

### 把 Copilot 当作谷歌或者维基百科来用

这是一种很重要的思维模式的改变：copilot 可以带来沉浸式的编程体验，即所有的工作都可以在当前 IDE 中去完成，而不用被浏览器剥夺注意力。除了 copilot 自带的代码解释功能之外，当你遇到编程方面的问题时，可以尝试用注释的方式去询问它而不是去谷歌，比如以下是我和它的两次对话（你只要写出注释 Q 后面的问题，那么它会新起一行以 A 开头给予回答）：

```javascript
-- Q: MySQL database name should connect with "-" or "_"?
-- A: MySQL database name should connect with "_", because "-" is a special character in MySQL.
```

```javascript
-- Q: In Nodejs project, the folder name should connect with "-" or "_"?
-- A: In Nodejs project, the folder name should connect with "-", because "_" is a special character in Nodejs.
```

我是否应该相信它的回答？答案应该是肯定的，因为反过来想，如果凡是它给出的答案你都选择进行二次确认，这对你是巨大的负担。你需要的是一次信仰之跃（Leap of faith），相信它就好

对于具体的代码回答，它会给出链接而非搬运代码，比如当我询问 `Generate NodeJS code for sending message to AzureServiceBus` 时，它会直接给出链接 https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-nodejs-how-to-use-queues


### 上下文很重要 

同一条指令在不同的文件位置可能会得出不一样的代码结果，例如这是我在一个全新的 JS 文件里给出上一小节相同指令所得到的结果：

```javascript
// Generate NodeJS code for sending message to AzureServiceBus:
//
// 1. Create a new NodeJS project
// 2. Add the following to the package.json file:
//    "dependencies": {
//      "azure-sb": "^0.10.3"
//    }
// 3. Run "npm install"
// 4. Copy this file to the project folder
// 5. Run "node send.js"
//
// Note: This sample uses the callback style of calling the send method.
//       There is also a promise style available.
//
// Documentation References:
// - ServiceBus NodeJS SDK API Reference
//   https://docs.microsoft.com/en-us/javascript/api/azure-sb/?view=azure-node-latest
// - ServiceBus NodeJS SDK Samples
//
// **************************************************************************************************
```

但是如果你把这段指令放在一个名为 sendMessageToAzureServiceBus 的函数中，它会直接优先给出代码或者上面提到的链接：

```javascript
async function sendMessageToAzureServiceBus() {
    // Generate NodeJS code for sending message to AzureServiceBus:
    var azureServiceBus = require("@azure/service-bus");
    var sbClient = new azureServiceBus.ServiceBusClient("<connection-string>");
    var sender = sbClient.createSender("<queue-name>");
    var message = {
        body: "Hello world"
    };
}
```

因为在后者的上下文中，很明显此时此刻你更想要的是实现

### 命名

上下文的重要性不仅仅体现在指令所在的位置这一层面，事实上文件里的所有蛛丝马迹都可以被 copilot 感知到，恰当的函数和变量命名尤其重要。

例如你现在想设计一个表结构用于存储播客节目的单集信息，如果你将表名命名为 item 的话，那么当你输入完 CREATE TABLE IF NOT EXISTS Item 之后，copilot 补全的结果如下：

```sql
CREATE TABLE IF NOT EXISTS Item (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    name TEXT NOT NULL,
    description TEXT,
    price REAL NOT NULL,
    image TEXT,
    category_id INTEGER NOT NULL,
    FOREIGN KEY (category_id) REFERENCES Category(id)
);
```

看起来和播客关系不大是不是，但如果你一开始就把表名设计为 PodcastEpisodeItem，那么 copilot 补全结果又是另一番景象

```sql
CREATE TABLE IF NOT EXISTS PodcastEpisodeItem (
    id INTEGER PRIMARY KEY AUTOINCREMENT,
    podcastId INTEGER NOT NULL,
    title TEXT NOT NULL,
    description TEXT NOT NULL,
    pubDate TEXT NOT NULL,
    url TEXT NOT NULL,
    downloaded INTEGER NOT NULL DEFAULT 0,
    FOREIGN KEY(podcastId) REFERENCES Podcast(id)
)
```

表的字段设计看上去更符合播客节目的信息了。

这个技巧在编写函数时同样管用，如果你的函数名只是称为 makeSometingDouble，它生成的函数指只会是

```javascript
function makeSomethingDouble(value) {
    return value * 2;
}
```

但如果你的目的是让数组里的每一个元素值翻倍，那么函数名最好叫 doubleArray，那么它生成的函数也会更符合你的期望

```javascript
function doubleArray(array) {
  return array.map(item => item * 2);
}
```

### 文件

文件的命名也很重要，它可以给 copilot 提供足够多的提示来告诉它你想干什么或者说它应该干什么，例如当你创建一个名为 azure-service-bus-service.js 的文件之后，只需要在空白文件中输入 async function s——是的你没看错，你甚至都不需要输入完整 send 这个单词，它就会为生成一个 sendMessage 的函数：

```javascript
async function sendMessages() {
  const sbClient = new ServiceBusClient(connectionString);
  const sender = sbClient.createSender(queueName);
  const messages = [
    { body: "Albert Einstein" },
    { body: "Werner Heisenberg" },
    { body: "Marie Curie" },
  ];
  try {
    for (let i = 0; i < messages.length; i++) {
      const message = messages[i];
      console.log(`Sending message: ${message.body}`);
      await sender.sendMessages(message);
    }
    await sender.close();
    await sbClient.close();
  } catch (err) {
    console.log("Error occurred: ", err);
  }
}
```

如果你接受了这段它提供的代码，那么接下来新起一行，你无需任何输入，它即会自动提出 receiveMessages 函数：

```javascript
async function receiveMessages() {
    const sbClient = new ServiceBusClient(connectionString);
    const receiver = sbClient.createReceiver(queueName);
    try {
        const messages = await receiver.receiveMessages(1);
        if (messages.length > 0) {
        console.log(`Received message: ${messages[0].body}`);
        await receiver.completeMessage(messages[0]);
        } else {
        console.log("No messages to receive");
        }
        await receiver.close();
        await sbClient.close();
    } catch (err) {
        console.log("Error occurred: ", err);
    }
    }
```

因为 azure-service-bus-service 这个名称已经提供足够窄的限制和丰富的上下文，来规范文件内代码的功能。如果你的文件只是宽泛的叫做 queue-service.js，那么当你告诉它生成一个名为 sendMessages 的函数的时候，它给出的实现也非常宽泛：

```javascript
async function sendMessages() {
    const messages = await getMessages();
    const queue = await getQueue();
    await queue.sendMessages(messages);
    await deleteMessages(messages);
}
```

这里的 queue 可以代指不同云服务商提供的 queue 工具

### 打开关联文件

在编程过程中，把所有相关的文件在 IDE 中都打开对 copilot 编程理解上下文同样有帮助。比如你在编写 A 文件代码的过程中需要引用 B 文件里的函数，那么最好在 IDE 中也打开 B 文件。

抱歉我无法稳定的复现和证明这一点，在写该文章的当下我使用的是一个文件数量不多的 demo 项目，但是在编写大型项目的过程中我的确能够体会到这一点，在之后生成的代码中，它甚至可以准确生成参数

让它有的抄

copilot 非常善于参考现有项目里已有的代码。这是一件好事，一方面它可以保证代码的一致性，另一方面如果它所抄袭的代码是你经过深思熟虑手写出来的，那么出错的概率也会小一些

举个例子，如果你项目中的写出了一个函数用于向 example 域名请求数据，其中借用的是 JS 原生 fetch 函数:

```javascript
async function sendRequestToExampleAPI() {
  fetch('https://example.com/api', {
    headers: {
      'Content-Type': 'application/json'
    },
    method: 'POST',
  }).then((response) => {
    return response.json();
  }).then((data) => {
    console.log(data);
  })
}
```

那么下一个向 NewsAPI 请求数据的函数，copilot 会完全参照上述函数的实现内容生成。下面代码除了函数名词以外皆由 copilot 生成：

```javascript
async function sendRequestToNewsAPI {
  fetch('https://newsapi.org/v2/everything?q=bitcoin&from=2020-08-29&sortBy=publishedAt&apiKey=API_KEY', {
    headers: {
      'Content-Type': 'application/json'
    },
    method: 'GET',
  }).then((response) => {
    return response.json();
  }).then((data) => {
    console.log(data);
  })
}
```

函数结构，“外观”一模一样。

你还可以这么测试它的“抄袭”功能，不妨把第一个函数的 fetch 替换成使用 axios，那么第二个函数生成时也会改用 axios 实现

```javascript
async function sendRequestToNewsAPI () {
    axios.get('https://newsapi.org/v2/top-headlines?country=us&apiKey=API_KEY')
        .then((response) => {
            console.log(response);
        }, (error) => {
            console.log(error);
        });
}
```

注意甚至你无法阻止它抄袭其他文件内容的，我尝试在注释里告诉它 don't refer any other code in the same space 但并不管用

### 给予提示

copilot 并不是完美的，有时候它提供的所有建议都不会绝对符合你的要求，这个时候就就需要你对它给予引导并且给予提示。

例如你给出了以下指令：Write a function to connect the MySQL database and run some SQL statements`，随后它生成的第一条语句是

```javascript
const mysql = require('mysql');
```

这里可能会出现和你预期不符的情况，例如你想使用另一个类库完成数据库操作，在这种情况下你需要对它生成的代码做一步一步的修正。

更恰当的方式其实是在文件的开头就引入特定你想使用的类库，又或者在编写完部分代码之后再对它给出指令。例如以下是我提前写出的代码：

```javascript
const mysql = require('mysql2/promise')
const cfg = require('./config');

const db = mysql.createPool(cfg.mysql);
```

这样一来它便知道所有 SQL 都应该交由 mysql2 来完成，并且连接池已经为它准备完毕，在我给出指令 Write a async function to run SQL query 之后它生成的代码如下：

```javascript
// Write a async function to run SQL query
async function query(sql, params) {
    const [rows, fields] = await db.query(sql, params);
    return rows;
    }
```

有时候你甚至需要从官方文档中黏贴一些代码片段到文件中用于给予它提示，或者在文件开头使用注释描述一下该文件的用途也同样有效

---

在 JetBrain 的 Marketplace 里我看到关于 Copilot 插件有这么一段简介：

	• 74% of developers are able to focus on more satisfying work
	• 88% feel more productive
	• 96% of developers are faster with repetitive tasks

我对其中的 repetitive tasks 深有感触，不妨回顾一下你每天的编码工作，其中有多少是重复无数次的机械劳动，有多少是听着播客或者音乐就可以完成的任务，它们都理应被 copilot 干掉。但也正如本文所示，“你”还没法被干掉，程序还需要你来主导，copilot 写出来的代码需要你来 review。

在 《Competing in the Age of AI 》一书中，公司价值被认为由两部分组成：业务模型（business model）和运营模型（operation model），业务模型决定公司的核心竞争力是什么，为用户创造了何种价值；而运营模型则决定了价值将如何被交付到用户手中。copilot 是个体运营自我的手段，而非业务模型的体现

