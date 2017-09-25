# 从React脚手架工具学习React项目的最佳实践

这篇文章不是聊React这门技术本身，而是关于如何维护好一个React项目。

编程领域中的“脚手架（Scaffolding）”指的是能够快速搭建项目的“骨架”的一类工具。例如大多数的React项目都有src目录，public目录，webpack配置文件，babel配置，而src目录中又通常包含components目录，reducers目录等等。每次在新建项目时，你不得不手动创建这些固定的文件目录，繁琐而累赘。脚手架就是帮助你完成这些重复性工作，一键生成主要的目录结构，甚至安装依赖。Yeoman就是著名的脚手架工具。

当你进入一个公司参与React项目时，你要做的可能只是开发指定的组件，执行命令启动项目查看效果，最后发布打包上线。你可能不会去思考为什么目录结构是这个样子，那么多配置文件是干什么用的。我曾经也是这个样子。今天我选择了两个React项目的脚手架工具 [create-react-app](https://github.com/facebookincubator/create-react-app)（以下简称**CRA**）和[react-starter-kit](https://github.com/kriasoft/react-starter-kit)（以下简称**RSK**），根据它们的说明文档以及一些个人的经验，来逐个解析不同文件的作用。

这些知识并不仅仅适用于React项目，文件的背后代表的是工具，工具的背后代表的其实是要解决的问题。不同的公司不同团队使用的工具可能会不同，将来也会有新的技术或者框架出现，但这些解决问题的思路同样能够复用。

因为CRA是Facebook官方推出的脚手架工具，所以我们以CRA为主线索展开，它的[User Guide](https://github.com/facebookincubator/create-react-app/blob/master/packages/react-scripts/template/README.md)文档最为（特）丰（别）富（长），本文的大部分内容也都参（翻）考（译）自这份文档，如果也理解不恰当之处还多多指教。其中也会穿插react-starter-kit的相关内容。

## 基础目录

CRA中有两个非常重要的目录





## 目录结构

```
my-app/
  README.md
  node_modules/
  package.json
  public/
    index.html
    favicon.ico
  src/
    App.css
    App.js
    App.test.js
    index.css
    index.js
    logo.svg
```
public/index.html 模板入口;
src/index.js 脚本入口.

src文件夹中可以创建子文件夹，只有src文件夹里的文件会被Webpack处理。**所有的JS和CSS文件都必须放在Webpack中**，否则Webpack看不到它们。

只有public中的文件才能被public/index.html使用

## 命令

- npm start: 3000端口的开发模式
- npm test: 跑测试
- npm run eject: 重置所有配置？

## 支持的语法特性

- Exponentiation Operator (ES2016).
- Async/await (ES2017).
- Object Rest/Spread Properties (stage 3 proposal).
- Dynamic import() (stage 3 proposal)
- Class Fields and Static Properties (part of stage 3 proposal).
- JSX and Flow syntax.

## 其他

- jslint: 通过 `.eslintrc` 配置编辑器lint配置
- Visual Studio Code 调试: 在`.vscode`文件夹中添加`launch.json`配置文件
- WebStorm 调试
