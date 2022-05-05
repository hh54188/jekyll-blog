---
layout: post
title: 从React脚手架工具学习React项目的最佳实践（上）：前端基础配置
modified: 2017-10-10
tags: [javascript, React]
image:
  feature: abstract-3.jpg
  credit: dargadgetz
  creditlink: http://www.dargadgetz.com/ios-7-abstract-wallpaper-pack-for-iphone-5-and-ipod-touch-retina/
comments: true
share: true
---

这篇文章不是聊React这门技术本身，而是关于如何维护好一个React项目。

文本可能会涉及一些Webpack的基础知识，如果你还不太了解Webpack的用法的话，可以从我之前的一篇文章[《Webpack 速成》](https://zhuanlan.zhihu.com/p/26041084)入门，深入浅出，童叟无欺。

编程领域中的“脚手架（Scaffolding）”指的是能够快速搭建项目“骨架”的一类工具。例如大多数的React项目都有src目录，public目录，webpack配置文件，babel配置等等，而src目录中又通常包含components目录，reducers目录等等。每次在新建项目时，你不得不手动创建这些固定的文件目录，繁琐而累赘。脚手架的作用就是帮助你完成这些重复性的工作，包括一键生成主要的目录结构、安装依赖等等。Yeoman就是著名的脚手架工具。

当你进入一个公司参与React项目时，你要做的可能只是开发指定的组件，执行命令启动项目查看运行和调试，最后发布打包上线。你可能不会去思考为什么目录结构是这个样子，那么多配置文件是干什么用的（我曾经也是这个样子）。今天我选择了两个React项目的脚手架工具 [create-react-app](https://github.com/facebookincubator/create-react-app)（以下简称**CRA**）和[react-starter-kit](https://github.com/kriasoft/react-starter-kit)（以下简称**RSK**），根据它们的说明文档以及一些个人的经验，来逐个解析不同文件的作用。

这些知识并不仅仅适用于React项目，文件的背后代表的是工具，工具的背后代表的其实是要解决的问题。不同的公司不同团队使用的工具可能会不同，将来也会有新的技术或者框架出现，但这些解决问题的思路同样能够复用。或者当你需要立项一个React项目而又不想依赖脚手架时，它们会是一份好的教科书。

因为CRA是Facebook官方推出的脚手架工具，所以我们以CRA为主线索展开，它的[User Guide](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md)文档最为（特）丰（别）富（长），本文的大部分内容也都参（翻）考（译）自这份文档，如果也理解不恰当之处还多多指教。其中也会穿插react-starter-kit的相关内容。

这个系列的文章会分为上下两个部分。在这上篇中，讲解的是一些常规项目搭建的基础配置，而在下篇的计划中，则会讲解高级配置，涉及开发环境的后端功能以及测试和部署。

最后一句废话想强调的是，任何脚手架生成的项目结构都仅供参考。实际的组织方式和使用工具都要依据实际情况而定。

## 项目目录

首先让我们从最基本的目录文件夹开始

CRA中有两个非常重要的目录有两个，`src`和`public`：
- `src`: 该目录中存放的是你的脚本和样式源文件，所有你需要经过Webpack打包的或者编译的文件都必须而且只能放在这个文件夹里（反过来说Webpack也只会去这个文件夹里找需要打包的文件）。例如TypeScript、LESS、SASS、Stylus源码等等，也可能是你需要在组件中引用的图片、SVG资源等等（之后会谈到在组件中引用样式和图片的用法）。总之`src`目录顾名思义（src = source），存放的是程序源码。

当然，`src`目录之下子目录的命名和组织就没有那么讲究了。如果你开发的是redux项目，自然会有components、reducers、actions等文件夹，甚至在components中分别为container component和stateless component建立文件夹都没问题。

最后，`src`目录的入口是`src/index.js`，不妨可以看看`index.js`的内容

```javascript
import React from 'react';
import ReactDOM from 'react-dom';
import './index.css';
import App from './App';
import registerServiceWorker from './registerServiceWorker';

ReactDOM.render(<App />, document.getElementById('root'));
registerServiceWorker();
```
很简单，它将入口组件渲染至页面上，并且注册service worker。

- `public`: `public`通常用于存放用户能够访问的资源，例如打包后的脚本、图片、HTML文件。但事实上并不仅限于此，从RSK项目中我们可以看到public文件夹中还有`robots.txt`、`humans.txt`、`crossdomain.xml`、`favicon.ico`等等

虽然public中存放的不是组件，public目录同样存在入口，即`/index.html`，也即是用户在域名根路径下访问到的页面。在CRA中规定，只有`public`文件夹内的资源才能被`index.html`使用。而html引用静态资源的方式也比较特别，并非是通过相对路径或者绝对路径的，而是通过全局变量引用。这个话题我们放在后面资源使用环节再说。

`public`文件夹有时候也被命名为`assets`甚至`resources`，这都没有关系。如果更加规矩一点，你可以在`public`中建立子文件夹`dist`用于存储发布上线的脚本和样式（dist其实就是distribute的缩写，也意味着发布的意思），或者建立`build`文件夹用于存储开发中构建后的脚本

`src`和`public`是最重要的两个文件夹。CRA中的文件夹只有这两个。我们不妨再可以看看RSK中的文件夹还有哪些:
- `docs`: 用于存放（markdown格式的）开发相关文档
- `tools`: 用于存放“工具脚本”的文件夹。“工具脚本”即是那些用于完成指定工作的脚本，在文件夹里你会看到例如`build.js`、`deploy.js`、`copy.js`等等。即使不展示这些脚本的具体内容，通过文件名也很容易判断这些脚本的作用，依次为构建、部署、复制文件等。这一部分脚本也可以通过`npm`命令执行，稍后详谈。
- `src/server`: 如果你的项目是以Javascript全栈的形式开发的话，可以将服务端代码也放入`src`中
- `test`: 用于存放测试相关脚本

## 各式各样的配置文件

越来越多的工具被发明来用于辅助我们的开发，但不同的工具配合不同的项目需要进行不同的配置。所以有各式各样的配置文件可能存在于我们的项目文件中。这些工具和配置文件你不一定都会用上，但至少你在过目之后不会再对它们陌生，或许在以后解决问题的过程中能够派的上用场。

以下的配置文件摘自RSK脚手架中（如果你第一次看到脚手架为你生成了这么多从来没有看到过的文件你一定会感到害怕，反正我是这么觉得的。）

- `.editorconfig`: 告诉编辑器该项目的代码规范。在团队开发中可能涉及的一个问题是，不同的同学可能使用的开发工具和开发习惯并不相同，有的使用WebStorm，有的使用Visual Studio Code。所以有可能在你的编辑器中习惯缩进使用的是2个空格，在他的编辑器中缩进使用的是4个空格。该配置文件就是用于存储统一的样式规范，告诉编辑器统一使用两个空格，不允许空字符串结尾等等。具体请参考[http://editorconfig.org](http://editorconfig.org)
- `.eslintrc.js`: 这个很好理解，eslint工具的配置文件。eslint是一款专业对js语法和格式进行检测的工具，大部分的编辑器应该都进行了集成，或者当作插件进行安装。该配置文件告诉eslint哪些文件可以忽略，哪些规则可以忽略，哪些文件适配哪些规则等等。具体请参考: [http://eslint.org/docs/user-guide/configuring](http://eslint.org/docs/user-guide/configuring)
- `.stylelintrc.js`: 同上，stylelint是对样式文件进行语法规范检测的工具，该配置文件则可以对检测规则进行细节配置。具体规则请参考: [https://stylelint.io/user-guide/configuration/](https://stylelint.io/user-guide/configuration/)
- `.flowconfig`: [flow](https://flow.org/en/)是Facebook推出一款用于对JavaScript语法进行类型检测的开源工具（有TypeScript的意思）。该文件就是该工具的配置文件，具体可以前往[https://flow.org/en/docs/config/](https://flow.org/en/docs/config/)
- `.env`: 在启动项目时难免会使用到环境变量，最著名的环境环境变量莫过于`NODE_ENV`，例如告诉程序使用生产环境：`NODE_ENV=production`。我们都知道可以在执行命令行时通过命令行参数的形式指定环境变量，例如`NODE_ENV=production node app.js`，然后再从程序里通过读取命令行参数的方式间接读取环境变量。而通过[dotenv](https://github.com/motdotla/dotenv)模块，我们可以将环境变量都放入`.env`环境中统一管理统一读取。
- `.travis.yml`: 持续集成工具[travis-ci](https://circleci.com/)的配置文件，该工具github marketplace有售，更多配置可以参考[https://docs.travis-ci.com/user/customizing-the-build](https://docs.travis-ci.com/user/customizing-the-build)
- `circle.yml`: 持续集成工具[circleci](https://travis-ci.org/)的配置文件，该工具github marketplace有售，更多配置可以参考[https://circleci.com/docs/1.0/configuration/](https://circleci.com/docs/1.0/configuration/)
- `jest.config.js`: Facebook 的测试工具jest的配置文件，更多配置可以参考[https://facebook.github.io/jest/docs/en/configuration.html](https://facebook.github.io/jest/docs/en/configuration.html)
- `jsdoc.config.json`: [jsDoc](http://usejsdoc.org/)是一款能够根据文件内函数注释生成文档的工具，该文件是该工具的配置文件，更多信息可以参考[http://usejsdoc.org/about-configuring-jsdoc.html](http://usejsdoc.org/about-configuring-jsdoc.html)
- `Dockerfile`: Docker容器的配置文件（对不起Docker我实在不熟，没有什么好补充的）

除此之外，还有一些你可能会用得上的一些文件，比如
- `CHANGELOG.md`: 版本更新的日志
- `CONTRIBUTEING.md`: 关于如何向该项目做出贡献

## 工具脚本

还在我入行的时候，前端开发流程是很简单的，手动创建一个静态页面，然后引入你需要的脚本就可以开始了。然而到了现在，不仅引入脚本的方式发生了改变，包括调试过程，打包流程，发布上线都变得复杂而且专业，而这一切都离不开NodeJS脚本。脚本带来的好处是可复用、自动化以及批量化处理。

开发中需要使用脚本处理的环节非常的多，例如将less编译为css，将脚本编译、压缩、拼接，压缩图片等等。这些工作可以交给Webpack或者Gulp或者Grunt去做。但这些第三方库并不是万能的，它们的运作也依赖它们所处生态里的插件。在这种复杂的依赖情况下，出错的情况常容易发生，为什么不建议再使用Gulp或者Grunt了呢，详见这篇文章：[Why we should stop using Grunt & Gulp](https://www.keithcirkel.co.uk/why-we-should-stop-using-grunt/)。正所谓流水的工具，铁打的脚本

npm脚本都存放在`package.json`文件里的`scripts`字段里

npm命令有机会我们能单独拿出一篇文章来聊，但言归正传回到脚手架，CRA中只用到了四种npm命令，分别是
- `npm start`: 在开发模式下启动app，默认使用使用3000端口，启动后在浏览器中输入[http://localhost:3000](http://localhost:3000)就能访问，如果应用发生了更改页面会自动刷新
- `npm test`: 运行应用的测试脚本
- `npm run build`: 为生产环境编译并且打包应用程序，打包到`build`文件夹中
- `npm run eject`: 如果你稍有心的观察CRA的目录里的文件，你会发现没有`.babel`文件，没有`webpack.config.js`类似文件。因为所有的这些琐碎的配置脚手架都帮你搞定了。全部都在`react-scripts`类库中。所以你看到`package.json`文件里`npm start`实际运行的是`react-scripts start`。当你不满足于脚手架为了你预设的配置时，你就可以使用`eject`命令将配置暴露出来（比如`start`命令，还有`webpack.config.dev.js`），这样你就可以完全自定义这些配置。注意这个操作是不可逆的。当你运行完毕之后你会发现`package.json`里的`scripts`的`start`命令变为`node scripts/start.js`

顺表说一下为什么`build`命令前需要加关键字`run`，而`start`和`test`就不需要，因为`npm start`是内置的预设命令，你可以理解为类似于宏的东西。如果你没有在`package.json`里自定义start命令的话而又执行`npm start`的话，它实际上执行的是`node server.js`。更多的内置命令请参考[https://docs.npmjs.com/misc/scripts](https://docs.npmjs.com/misc/scripts)

### 自动格式化代码

这里所说的格式化代码并不是指美化和格式化已经压缩过的代码以便于阅读。而是在代码的提交阶段（commit）强制对代码进行格式化。所以这里用到了额外的三个库
- `husky`: 便于以npm脚本的形式调用git hooks（hook指的是在某一个特定情况下执行的代码，比如React的各个预留出来的生命周期函数就算是hook）
- `lint-staged`: 便于我们对staged阶段（准备提交阶段）的文件执行npm脚本
- `prettier`: 对代码进行格式化

核心类库当然是[prettier](https://github.com/prettier/prettier)，为什么在开发时仍然需要对代码格式化，prettier自己给出了几个理由，比如强制对代码进行格式化避免PR时产生不必要的语法问题，比如帮助还不熟悉的新同学规范代码，总之仍然是有必要的。

prettier解决了how的问题，但是还需要`husky`和`lint-staged`解决when的问题，也就是什么时候做格式化。在CRA中，格式化的工作时放在准备提交的阶段（pre-commit），在实际项目中你还可以放在预备push的阶段。

husky解决的问题是将pre-commit的hook暴露出来。默认情况下如果你想编写pre-commit脚本，你需要编辑你项目的`.git/hooks/pre-commit`文件，如果我没有记错的话应该是shell脚本，并且在执行之前记得赋予它们执行权限。

然而当你安装完husky之后，你就可以把pre-commit阶段需要执行的脚本直接放在`package.json`里的`scipts`里的`precommit`字段里，比如:
```javascript
"scripts": {
  "precommit": "eslint"
},
```

而`lint-staged`解决的则是最后一公里的问题，即封装在pre-commit阶段需要执行的脚本，同样是在`package.json`配置，例如：
```javascript
  "dependencies": {
    // ...
  },
 "lint-staged": {
   "src/**/*.{js,jsx,json,css}": [
     "prettier --single-quote --write",
     "git add"
   ]
 },
  "scripts": {
```

## 开发规范

进入到组件化的时代，一切都是组件，就连html也可以变身为组件。在RSK脚手架中，你甚至会看到一个名为`Html.js`的组件（然后采用后端渲染）。我们希望用组件解决一切问题，而不是把需要维护的代码遗落在各个地方，甚至包括`<head />`标签里的内容。`<title />`、`<meta />`就交给[React Helmet](https://github.com/nfl/react-helmet)解决吧。

开发组件和引用组件就不赘述了，全世界都一样，相信大家也耳熟能详了。

### 样式

至于样式，无论你是使用Less、Sass、还是Stylus都一样，只要在Webpack中使用对应的loader就能将其编译为css。需要注意的是组织样式的方式。传统项目中样式和脚本是分离的，放在不同的文件夹中。但是在React项目中，我们只有组件一个维度，组件同时包含样式和脚本，都放在`components`文件夹中。例如：
```
components/
|--Button.js
|--Button.less
```
那么在`Button.js`中你可以直接引用样式
```javascript
import React, { Component } from 'react';
import './Button.css';
```
或者，你也可以把所有的样式都在样式入口`src/index.css`中引入，然后在组件入口`src/index.js`中又统一引入样式入口`src/index.css`。

除了编译样式之外还有一些额外的工作需要进行，例如压缩，例如为某些样式属性添加浏览器前缀。在CRA中会使用[Autoprefixer](https://github.com/postcss/autoprefixer)或者postcss进行处理，当然这一切都集成在react-scripts中。你也可以独立的使用npm脚本进行处理，监视样式的变化，当样式文件发生更改时自动的进行预处理和“后”处理，这个流程对脚本文件也同样有效。

目前也有很多专门用于优化npm脚本执行的类库，在上述流程中你也能够（或者说是必须）用上：
- `onchange`、`watch`: 用于监视文件修改，然后执行特定的npm脚本，比如
```
"scripts": {
  ...
  "watch:css": "onchange 'src/scss/*.scss' -- npm run build:css",
  "watch:js": "onchange 'src/js/*.js' -- npm run build:js",
}
```
- `parallelshell`: 用于并行的执行多个npm脚本，比如
```
"scripts": {
  ...
  "watch:all": "parallelshell 'npm run serve' 'npm run watch:css' 'npm run watch:js'"
}
```
相信你能领悟到这些代码干了什么事情:P

### 添加图片字体等额外资源

图片与字体等资源也和样式一样都与要使用它们的组件放在同一层级，至少都应该属于同一个`components`文件夹中，在组件也是通过`import`的关键字引入，例如

```javascript
import React from 'react';
import logo from './logo.png'; // Tell Webpack this JS file uses this image

console.log(logo); // /logo.84287d09.png

function Header() {
  // Import result is the URL of your image
  return <img src={logo} alt="Logo" />;
}
```

为了减少页面的请求数，体积小于10000 bytes的图片会返回data URI而不是实际的路径。当项目需要（为生产环境）进行构建时，Webpack会把大于10000 bytes的图片资源拷贝到最终构建的文件夹中（在CRA中的目录是`/build/static/media`），并且根据内容hash值进行重新命名。所以不用担心资源发生修改之后因为浏览器的缓存而不会生效

为什么要采用`import`的方式引用样式和图片，文档中给出了三条理由：
- 脚本和样式能够得到压缩以及打包在一起，以便减少额外的网络请求
- 在编译阶段如果发现文件丢失就会及时报错，而不是上线之后再呈现给用户404错误
- 根据文件内容的hash值对文件进行重命名而避免浏览器缓存问题

### 如果一定要引用`public`文件夹中的资源

并非所有的资源都能在组件中引用，又或者有的第三方类库并不支持与React集成，此时你就需要把资源放入`public`文件夹中，然后在html中引用，比如：
```html
<link rel="shortcut icon" href="%PUBLIC_URL%/favicon.ico">
```
那么在构建时（`npm run build`），Webpack会将`%PUBLIC_URL%`替换为实际的`public`目录的绝对路径。

在js文件中也可以通过访问`process.env.PUBLIC_URL`变量来获得`public`文件夹的绝对路径
```javascript
render() {
  // Note: this is an escape hatch and should be used sparingly!
  // Normally we recommend using `import` for getting asset URLs
  // as described in “Adding Images and Fonts” above this section.
  return <img src={process.env.PUBLIC_URL + '/img/logo.png'} />;
}
```
这种访问资源的方式有以下一些缺点（当然是相对于`import`方式而言），请务必了解：
- `public`文件夹内的文件不会做任何处理，包括压缩或者拼接之类的
- 如果有文件丢失的话在编译阶段不会报错，用户可能会收到404的请求返回
- 你可能需要手动处理缓存问题，例如对文件发生修改时对文件进行重命名，或者修改`Etag`等缓存条件。详情可以参考我的这篇文章[《设计一个无懈可击的浏览器缓存方案：关于思路，细节，ServiceWorker，以及HTTP/2》](https://zhuanlan.zhihu.com/p/28113197)

但是在某些情况下可以考虑使用这种访问资源的方式
- 你需要引用一些打包之外的额外脚本，比如[pace.js](http://github.hubspot.com/pace/docs/welcome/)，比如google analytics脚本
- 有一些脚本和Webpack不兼容
- 上千张图片需要动态引用
- 构建时打包输出的文件需要指定文件名

## 添加自定义的环境变量

CRA脚手架还允许你在`process.env`上添加自定义的环境变量供全局访问。

默认它会提供两个环境变量供使用，一个是上一节用到的`public`文件夹路径`PUBLIC_URL`。另一个是大家更加熟悉的`NODE_ENV`。后者是一个代表当前开发环境的变量，当你运行`npm start`时它等于`development`；当你运行`npm test`时它的值是`test`；当你运行`npm run build`时，它的值是`production`。你无法手动的覆盖它，它能够防止开发者不小心打包了一个开发版本部署到线上。`NODE_ENV`也能够帮助你有针对性的调试代码，比如你只希望非`production`环境下停用分析脚本：
```javascript
if (process.env.NODE_ENV !== 'production') {
  analytics.disable();
}
```

当然你也可以添加自己的环境变量，添加方式有两种，一种是通过命令行的方式比如在Windows系统下`set REACT_APP_SECRET_CODE=abcdef&&npm start`。另一种是通过`.env`文件（也就是通过[dotenv](https://github.com/motdotla/dotenv)类库），把你所需要的环境变量都写在这个文件中:
```
REACT_APP_SECRET_CODE=abcdef
```
需要注意的事情是，所有的自定义环境变量都需要以`REACT_APP_SECRET_CODE`开头（至于理由我没有看懂：Any other variables except NODE_ENV will be ignored to avoid accidentally exposing a private key on the machine that could have the same name. 我不是很理解 exposing a private key 是什么意思）。

一旦环境变量定义完毕之后，就能在文件中使用，比如在脚本中：
```javascript
render() {
  return (
    <div>
      <small>You are running this application in <b>{process.env.NODE_ENV}</b> mode.</small>
    </div>
  );
}
```
比如在html中：
```html
<title>%REACT_APP_WEBSITE_NAME%</title>
```
最后，不同开发环境中的环境变量不必都放在`.env`文件中，可以划分为`.env.development`, `.env.test`, .`env.production`等不同的文件存放，并且不同文件之间还存在优先级的关系，详情可以访问[dotenv的文档](https://github.com/motdotla/dotenv)

## 编辑器调试

目前比较流行的IDE比如[Visual Studio Code](https://code.visualstudio.com/)和[WebStorm](https://www.jetbrains.com/webstorm/)都支持编辑器内的代码调试，但是可能需要配置编辑器的环境变量，或者增加配置文件，又或者给浏览器安装插件。我个人使用编辑器进行调试的体验并不好，并非所有的场景都支持调试。同时因为脚手架和编辑器调试时都会启动后端环境，这之中可能需要解决冲突的地方。

具体的配置信息可以参考这两款编辑器的官方文档。

上篇完




## 参考资料

[https://www.site2share.com/folder/20020524](https://www.site2share.com/folder/20020524)

你可能会喜欢

- [面试系列之二：你真的了解React吗（上）如何设计组件以及重要的生命周期](https://www.v2think.com/understand-react-01)
- [面试系列之三：你真的了解React吗（中）组件间的通信以及React优化](https://www.v2think.com/understand-react-02)
- [面试系列之四：你真的了解React吗（下）Flux与Vuex的差异以及Webpack](https://www.v2think.com/understand-react-03)
- [从React脚手架工具学习React项目的最佳实践（上）：前端基础配置](https://www.v2think.com/create-app-user-guide)
- [深入React的生命周期(上)：出生(Mount)](https://www.v2think.com/dig-into-react-lifecircle-01)
- [深入React的生命周期(下)：更新(Update)](https://www.v2think.com/dig-into-react-lifecircle-02)
- [深入理解 Webpack 打包分块（上）](https://www.v2think.com/webpack-chunks-split-01)
- [深入理解 Webpack 打包分块（下）](https://www.v2think.com/webpack-chunks-split-02)
- [Webpack 速成](https://www.v2think.com/webpack-tutorial)
- [构建大型 Mobx 应用的几个建议](https://www.v2think.com/tips-for-building-mobx-app)
- [【译文】给构建大型 redux 应用的五个建议](https://www.v2think.com/five-tips-for-redux-large-applications)
- [React + Redux 性能优化（一）：理论篇](https://www.v2think.com/redux-performance-01-basic)
- [React + Redux 性能优化（二）工具篇： Immutablejs](https://www.v2think.com/redux-performance-02-immutablejs)
- [Flux与Redux背后的设计思想(二)：CQRS, Event Sourcing, DDD](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CQRS-ES-DDD)
- [Flux与Redux背后的设计思想(一)：Command Bus, Event Bus, Service Bus](https://www.v2think.com/design-philosophy-behind-flux-and-redux-CB-EB-ESB)