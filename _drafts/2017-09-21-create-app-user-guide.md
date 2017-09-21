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
