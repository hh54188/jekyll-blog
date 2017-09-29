# 从React脚手架工具学习React项目的最佳实践

这篇文章不是聊React这门技术本身，而是关于如何维护好一个React项目。

文本可能会涉及一些Webpack的基础知识，如果你还不太了解Webpack的用法的话，可以从我之前的一篇文章[《Webpack 速成》](https://zhuanlan.zhihu.com/p/26041084)入门，深入浅出，童叟无欺。

编程领域中的“脚手架（Scaffolding）”指的是能够快速搭建项目的“骨架”的一类工具。例如大多数的React项目都有src目录，public目录，webpack配置文件，babel配置，而src目录中又通常包含components目录，reducers目录等等。每次在新建项目时，你不得不手动创建这些固定的文件目录，繁琐而累赘。脚手架就是帮助你完成这些重复性工作，一键生成主要的目录结构，甚至安装依赖。Yeoman就是著名的脚手架工具。

当你进入一个公司参与React项目时，你要做的可能只是开发指定的组件，执行命令启动项目查看效果，最后发布打包上线。你可能不会去思考为什么目录结构是这个样子，那么多配置文件是干什么用的。我曾经也是这个样子。今天我选择了两个React项目的脚手架工具 [create-react-app](https://github.com/facebookincubator/create-react-app)（以下简称**CRA**）和[react-starter-kit](https://github.com/kriasoft/react-starter-kit)（以下简称**RSK**），根据它们的说明文档以及一些个人的经验，来逐个解析不同文件的作用。

这些知识并不仅仅适用于React项目，文件的背后代表的是工具，工具的背后代表的其实是要解决的问题。不同的公司不同团队使用的工具可能会不同，将来也会有新的技术或者框架出现，但这些解决问题的思路同样能够复用。

因为CRA是Facebook官方推出的脚手架工具，所以我们以CRA为主线索展开，它的[User Guide](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md)文档最为（特）丰（别）富（长），本文的大部分内容也都参（翻）考（译）自这份文档，如果也理解不恰当之处还多多指教。其中也会穿插react-starter-kit的相关内容。

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

- `public`: `public`通常存放的对外能够访问的资源，例如打包后的脚本、图片、HTML文件。但事实上并不仅限于此，从RSK项目中我们可以看到public文件夹中还有`robots.txt`、`humans.txt`、`crossdomain.xml`、`favicon.ico`等等

虽然public中存放的不是组件，public目录同样存在入口，即`/index.html`，也即是用户在域名根路径下访问到的页面。在CRA中规定，只有`public`文件夹内的资源才能被`index.html`使用。而html引用静态资源的方式也比较特别，并非是通过相对路径或者绝对路径的，而是通过全局变量引用。这个话题我们放在后面资源使用环节再说。

`public`文件夹有时候也被命名为`assets`文件夹，甚至`resources`文件夹这都没有关系。如果更加规矩一点，你可以在`public`中建立子文件夹`dist`用于存储发布上线的脚本和样式（dist其实就是distribute的缩写，也意味着发布的意思），或者建立`build`文件夹用于存储开发中编译后的脚本

`src`和`public`是最重要的两个文件夹。CRA中的文件夹只有这两个。我们不妨再可以看看RSK中的文件夹还有哪些:
- `docs`: 用于存放（markdown格式的）开发相关文档
- `tools`: 用于存放“工具脚本”的文件夹。“工具脚本”即是那些用于完成指定工作的脚本，在文件夹里你会看到例如`build.js`、`deploy.js`、`copy.js`等等。即使不展示这些脚本的具体内容，通过文件名也很容易判断这些脚本的作用，依次为构建、部署、复制文件等。这一部分脚本也可以通过`npm`命令执行，稍后详谈。
- `src/server`: 如果你的项目是以Javascript全栈的形式开发的话，可以将服务端代码也放入`src`中
- `test`: 用于存放测试相关脚本

## 各式各样的配置文件

越来越多的工具被发明来用于辅助我们的开发，但不同的工具配合不同的项目需要进行不同的配置。所以有各式各样的配置文件可能存在于我们的项目文件中。这些工具和配置文件你不一定都会用上，但至少你在过目之后不会再对它们陌生，或许在以后解决问题的过程中能够派的上用场。

以下的配置文件摘自RSK脚手架中，以及会结合一些额外的我使用到的工具配置文件

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

- [In a web project, what is the difference between a dist and build directory (traditionally)?](https://stackoverflow.com/questions/16303525/in-a-web-project-what-is-the-difference-between-a-dist-and-build-directory-tra)