# Webpack 速成

## 前言

如果你已经对Webpack精通了或者至少一直在工作中使用它，请关闭当前浏览器标签，无视这篇文章。

这篇文章本意是写给我自己看的，作为一篇Cookbook供快速查询和上手用。原因是虽然工作中会涉及到React开发，但并不是持续性的。可能两个功能的迭代相隔几周甚至一个月。期间则是使用其他的工具或者框架进行开发。而每次捡起来重新开发时或者立新项时，发现已经不太会写webpack配置了，又需要重新查询各种教程。后来反思其实是因为从来就没有真的学懂过webpack。这篇文章就是我在重新彻底学习完webpack之后的总结文章。也为了方便自己今后查询用。

## 什么是 Webpack

webpack 是一个打包工具，为什么需要打包？因为有的人的脚本开发语言可能是 CoffeeScript 或者是 TypeScript，样式开发工具可能是 Less 或者 Sass，这都需要工具把它们“编译”成浏览器能识别 Javascript 和 CSS。webpack就是干这个的。借用它们官网的一张图很好的诠释了以上的描述：

![what is webpack](./images/webpack-tutorial/what-is-webpack.png)

现在你可能会问为什么我要用它？Grunt和Gulp不是也能做相同的事情吗？我也是这么认为的。Grunt和Gulp定位为任务/流程工具（Grunt的副标题为The JavaScript Task Runner），除了打包工作外，它们还能执行图片压缩，文档生成（虽然这其中的很多webpack也已经能做了），代码检查等等，你可以自己自由选择要执行的任务然后把它们一环连一环的拼接在一起。理论上来说，webpack是Grunt的功能子集。

然后为什么我要用webpack？好吧，这个问题你也可以用在为什么已经有Grunt了还要造一个Gulp？以及为什么我要用Gulp替代Grunt，它们俩功能不也类似吗？客套点的答案是，存在的即是合理的，它们的出现必然有可取之处；残酷一点的答案是webpack是当下最流行最前沿的，是作为前端工程师先进性的表现，所以你必须要学。就和使用gmail比使用qq邮箱求职更让人看得起一样，其实没什么道理。什么？你说你不想学，不会也就不会了？这话别对我说，对你将来的面试官说

## Webpack 误区

我接触 Webpack 是从学习React开始，所以一直有个误区：Webpack，React，Babel是深度绑定的。其实不是。如果你不是在进行React开发，你仍然可以是用Webpack做CoffeScript或者是Sass的打包工作，自然也就不需要Babel。即使你在进行React开发，但是不使用jsx，你仍然可以选择不使用Babel。

Webpack是一个很强悍的工具，提供非常的多的参数供配置，能做到很多意想不到的事情。系统的讲解webpack的教程也很多，github上一搜一大堆，排名靠前的还都是国内人写的或者翻译的。所以再次强调本文只是供入门快速上手之用。只覆盖我目前接触到的、常用的或者是比较好用的一些参数，解释应该在什么情况下如何使用它们，相信已经可以覆盖大部分的开发情况了。

在自学Webpack的时候发现webpack存在碎片化的问题，就是在不同版本中编写参数的规则可能不同。本文都统一以 webpack 2 为标准

## 基础

首先你需要全局安装 webpack: `npm install -g wabpack`。
同时还建议你在本地的开发环境安装项目级别的webpack：`npm install --save-dev webpack`。因为我们可能会使用到webpack自带的一些工具。

然后再你的项目根目录下新建一个`webpack.config.js`的文件，用来编写和 webpack 相关的配置。当然配置文件名也可以叫其他的名字，那么在你需要在运行 `webpack` 命令时则需要指定配置文件名`webpack --config myconfig.js`。

也可以不使用配置文件，通过命令行参数的形式运行 webpack，不过那只是听上去美好入门玩玩而已，不具有可维护性和操作性（因为开发环境的配置是及其复杂的），就不谈了。

### 合并脚本

webpack的基本功能就是把多个脚本打包为一个脚本，比如脚本模块 A 依赖同目录下的脚本模块 B 和 C：
```
// A.js:
import {*} from './B.js'; // E6 Modules
const C = require('/C.js'); // CommonJS
```

那么我们可以认为 A 是入口模块（从模块A进入之后就能找到我们应用需要的所有模块），并且我们需要指定一个打包后的输出文件，比如叫`bundle.js`，那么我们在`webpack.config.js`的配置文件里可以这么写：

```
module.exports = {
    entry: './A.js',
    output: {
        filename: './bundle.js'
    }
}
```
接下来打开命令行（cmd），切换到开发的根目录，运行`webpack`，合并后的`bundle.js`即输出生成了。

`entry`属性表示入口模块，`output`属性表示输出脚本。这里有两点可以改进：
- `entry`属性的值可以是一个数组，意味着可以允许有多个入口模块
- `output`对象中还可以添加`path`属性，表示要输出的路径（必须为绝对路径，所以可以借助Node.js的`path.resolve`或者`path.join`方法）；而在`filename`中填上文件名即可

### Webpack支持的脚本模块规范

不同项目在定义脚本模块时使用的规范不同。有的项目会使用CommonJS规范（参考Node.js）；有的项目会使用ES6 Modules的模块规范；有的还会使用AMD模块规范（参考RequireJS）。Webpack对这三种都支持。正如我上一个例子里A.js内容所示，还支持混合使用。

### 监视修改，自动打包

开发中文件处于不停的修改状态，如果每一次修改之后都要手动的在命令行中运行webpack命令才能重新打包，这个过程是痛苦的。于是乎你可以给`wepack.config.js`文件中添加`watch`参数，告诉webpack监视文件的变化。一旦发生变化后自动打包：
```
module.exports = {
    entry: './A.js',
    output: {
        filename: './bundle.js'
    },
    watch: true
}
```
或者你也可以在命令行中运行`webpack`命令时添加`-w`参数

### “别名”

实际项目中源文件不会放在项目的根目录中，而是集中放在某个文件夹内，比如叫`src`。并且文件夹中又会再次将文件分类，例如分为`srcipts`和`styles`，`scripts`中又会添加为`components`和`utils`。`components`中下又有具体的组件文件夹等等。所以在引用模块或者组件时常常会发生这样的情况，引用名称冗长无比：
```
require('./src/scripts/components/checkbox/checkbox.js');
```
然而仔细观察，`./src/scripts/components`这个路径是非常累赘的，几乎每个引用组件的语句都要使用到，所以我们可以在webpack配置文件中添加一个“代号”代指这个路径。这就是`alias`字段。`alias`字段必须添加在`resolve`字段下：
```
module.exports = {
    entry: './A.js',
    output: {
        filename: './bundle.js'
    },
    resolve: {
        alias: {
            Components: path.join(__dirname, '..', 'src', 'scripts', 'components')
        }
    },
    watch: true
}
```
那么当我们需要引用`./src/scripts/components`目录下的组件时，引用的路径只是`Components/checkbox.js`就好了

### 修改上下文

在上面的例子中，我们默认把`webpack.config.js`配置文件置于项目的根目录。但有时我们不希望把配置文件放在根目录，因为配置文件可能有很多，开发时的配置文件，上线时的配置文件，测试也需要配置文件。

于是我们可以把所有的配置文件都放在一个文件夹中管理，例如叫做`config`。但此时入口文件`app.js`则与配置文件不在同一个目录中，则需要新增配置参数告诉webpack去哪里找`app.js`。这个配置参数就叫做`context`。

因为我们的`config`文件夹是处于根目录下，`webpack.config.js`处于`config`文件夹中，与`app.js`的结构关系如下图所示：
```
Root
|---config
    |---webpack.config.js
|---app.js
```
所以在`context`值如下所示，务必使用绝对路径：
```
module.exports = {
    entry: './A.js',
    context: path.join(__dirname, '..'),
    output: {
        filename: './bundle.js'
    }
}
```

在根目录运行webpack时，则需要指定配置文件：`webpack --config config/webpack.config.js`

### 存储 webpack 命令

在上面一小节，我们把配置文件统一放入`config`文件夹中后，每次打包时都需要输入一长串的`webpack --config config/webpack.config.js`，这样非常不便。于是我们可以把命令添加进入每个项目都有的`package.json`文件中即可。

首先你的项目中需要有`package.json`文件。如果还没有的话有两个办法：
1. 将命令行切换至根目录下，运行`npm init`，命令行则会一步一步引导你建立package.json文件
2. 手动在根目录下创建一个空文件，并命名为`package.json`，在文件中填充上JSON格式的常规内容。例如初期只需要name和version字段，甚至一个空对象都可以：
```
{
    "name": "Project",
    "version": "0.0.1"
}
```
接下来我们添加一个`scripts`字段，字段值是一个对象：
```
{
    "name": "",
    "version": "",
    "scripts": {

    }
}
```
此时我们就可以把我们要执行的命令放入`scripts`对象中，因为是开发环境，所以我把这个命令取名为`dev`：
```
{
    "name": "",
    "version": "",
    "scripts": {
        "dev": "webpack --config config/webpack.config.js"
    }
}
```
最后，当你需要运行webpack命令时，只需要运行`npm run dev`就可以了。其中的`dev`是可以变化的参数，你可以继续往`scripts`字段中的添加其他的参数。

### 加载器（Loader）

在入口文件 app.js 中，我们还可以引用样式文件和图片例如：

```
require('./styles/style.css');
```
那么你一定很好奇把样式打包进脚本的效果是什么样的？实际情况是，当你打开包含最终脚本`bundle.js`的页面时，你会发现样式代码已经注入进页面的`head`中了。

但是举这个例子我是想说明另外一个问题。

默认情况下webpack只认识js文件，所以它只能打包js文件。如果你的开发环境中使用了其他语言比如CoffeeScript则webpack无能为力。然而你可以通过给 webpack 添加 loader 来让 webpack 识别更多的文件类型。比如我们可以添加`style-loader`和`css-loader`让 webpack 识别样式文件并且打包，并且注入页面中。

让我们安装样式相关的loader：`npm install --save-dev style-loader css-loader`

安装完毕之后，我们还需要对loader进行配置。告诉这个loader应该指定对哪些文件进行识别和处理，在`webpack.config.js`中添加对loader的配置，添加在`module`字段中：

```
module: {
    loaders: [{
        test: /\.css$/,
        loaders: ['style-loader', 'css-loader']
    }]
}
```
`test`是一个正则表达式用于匹配使用该loader的文件
`loaders`则表示使用了哪些loader

注意在新版本的webpack中，loaders数组中loader名称一定要加上`-loader`后缀，否则打包时会出错

我们还可以告诉loader排除某些目录，通过添加`exclude`字段，注意需要使用绝对路径：
```
module: {
    loaders: [{
        test: /\.css$/,
        exclude: path.join(__dirname, )
        loaders: ['style-loader', 'css-loader']
    }]
}
```

这里的样式插件只是举例。插件更重要的用处是在于开进行React开发时使用Babel对jsx文件和ES6语法进行处理。这个会在后面专门说。

### 插件（Plugins）

如果你有打开上面所说的打包后的`bundle.js`文件的话，你会发现这个文件内容是未压缩。在开发中我们存在类似的需求例如对最终文件进行压缩。此时我们就需要使用到插件（plugin）了。

在webpack2中webpack已经自带了一些插件，例如压缩脚本代码用的UglifyJsPlugin，这也是我们为什么之前需要在本地安装一个webpack的原因。需要使用该插件时，首先在文件头部引用webpack类库：`const webpack = require('webpack')`，然后请在`plugins`字段下新建一个实例：
```
plugins: [
    new webpack.optimize.UglifyJsPlugin()
]
```
同时你也可以在`UglifyJsPlugin`构造函数调用中传入参数对插件进行配置。

最后当运行`webpack`命令后，你会看到`bundle.js`的代码已经是压缩状态了

### Webpack-dev-server







- https://github.com/AriaFallah/WebpackTutorial
- https://scotch.io/tutorials/getting-started-with-webpack-module-bundling-magic
- https://scotch.io/tutorials/setup-a-react-environment-using-webpack-and-babel
- https://www.codementor.io/tamizhvendan/beginner-guide-setup-reactjs-environment-npm-babel-6-webpack-du107r9zr
- https://www.twilio.com/blog/2015/08/setting-up-react-for-es6-with-webpack-and-babel-2.html
- https://webpack.github.io/docs/tutorials/getting-started/

