---
title: 1. 用 Webpack 启动应用
---

因为对 Rxjs 的好感玩上了 Cycle.js，该系列用于记录使用该框架的一些笔记。本文记录用 webpack 配置环境，启动 app 的过程。

<!--more-->

## 关于 Cycle.js

### Rxjs

接触 Rxjs 也是因为 Angular，关于 Angular 请查看[《重新认识 Angular》](https://github.com/godbasin/godbasin.github.io/issues/1)。
关于 Rxjs 这里更多是来自[《不要把 Rx 用成 Promise》](https://zhuanlan.zhihu.com/p/20531896)。

- 核心思想: 数据响应式

  - **Promise => 允诺**
  - **Rxjs => 由订阅/发布模式引出来**

- 执行和响应

  - **`Promise`需要`then()`或`catch()`执行，并且是一次性的。**
  - **`Rxjs`数据的流出不取决于是否`subscribe()`，并且可以多次响应**

- 数据源头和消费

  - **`Promise`没有确切的数据消费者，每一个`then`都是数据消费者，同时也可能是数据源头，适合组装流程式（A 拿到数据处理，完了给 B，B 完了把处理后的数据给 C，以此类推）**
  - **`Rxjs`则有明确的数据源头，以及数据消费者**

- Rxjs 例子

```javascript
const observable = Rx.Observable.fromEvent(input, "input") // 监听 input 元素的 input 事件
  .map(e => e.target.value) // 一旦发生，把事件对象 e 映射成 input 元素的值
  .filter(value => value.length >= 1) // 接着过滤掉值长度小于 1 的
  .distinctUntilChanged() // 如果该值和过去最新的值相等，则忽略
  .subscribe(
    // subscribe 拿到数据
    x => console.log(x),
    err => console.error(err)
  );

// 订阅
observable.subscribe(x => console.log(x));
```

### Cycle.js

[Cycle.js](https://cycle.js.org)则是把 Rxjs 极致化，不管是用户操作、http 请求、数据和状态变更，整个框架级都是通过 Rxjs 的思想实现的。

如果说状态管理的话，那么[MobX](https://mobx.js.org/)也是基于 Rxjs 思想做出的与 Redux、Flux、Vuex 等相似的状态管理工具。

不知道大家会不会觉得，把所有的框架设计都使用 Rxjs 来实现，会不会跟我们目前的通用框架 Vue/React/Angular 等差异太大呢？

管他呢，既然无法想象，那么我们就直接去实践看看吧。

## 基本依赖

### package.json

在使用 Cycle.js 这个新兴框架的时候，本骚年会尽量选择跟常用的框架相似的架构和方式，去让这个切换来得更加易懂。

因此这里，我们选择使用 Webpack。（官方使用并不需要用到 Webpack，社区更是有 create-cycle-app 等脚手架，大家有兴趣也可以去看看）

```json
{
  "name": "cyclejs-demo",
  "version": "1.0.0",
  "description": "cycle.js demo",
  "main": "index.js",
  "scripts": {
    "dev": "webpack-dev-server --config webpackdev.config.js --port 3333 --host 0.0.0.0 --devtool eval --progress --colors --hot --content-base dist",
    "build": "webpack --config webpack.config.js"
  },
  "keywords": ["cycle.js"],
  "author": "godbasin <wangbeishan@163.com> (https://github.com/godbasin)",
  "license": "MIT",
  "dependencies": {
    "@cycle/dom": "^18.0.0",
    "@cycle/run": "^3.1.0",
    "babel-plugin-transform-react-jsx": "^6.24.1",
    "snabbdom-jsx": "^0.3.1",
    "xstream": "^10.9.0"
  },
  "devDependencies": {
    "babel-core": "^6.25.0",
    "babel-loader": "^7.1.1",
    "babel-preset-es2015": "^6.24.1",
    "html-webpack-plugin": "^2.29.0",
    "loglevel": "^1.4.1",
    "source-map-loader": "^0.2.1",
    "webpack": "^3.3.0",
    "webpack-dev-server": "^2.5.1"
  }
}
```

这里可以大概看到，除了基本的 Cycle.js 的依赖，我们还用到了常用的一些工具或者库，像：

- webpack
- babel
- jsx
- ...

本节我们主要涉及的是 webpack。
webpack 依赖安装，除了`npm install`外，还需要全局安装`npm install -g webpack`。

webpack 本骚年使用的是 v2.0+版本，而 1 迁 2 其实也不是什么难事情，官方有个很详细的[migration 说明](https://webpack.js.org/guides/migrating/)，大家可以参考看看。

- `script`命令

至于 scripts 命令，这里有两个：

1. `npm run dev`：启动 webpack 服务。
2. `npm run build`: 构建生成 dist 目录。

## webpack 配置

### webpack.config.js

这里简单介绍一下几个使用到的 loader。

- babel-loader

这里主要用来配置官方的 babel，用于 jsx 的使用：

```js
// .babelrc
{
  "presets": [
    "es2015"
  ],
  "plugins": [
    "syntax-jsx",
    ["transform-react-jsx", {"pragma": "html"}]
  ]
}
```

上了 jsx 的话，我们的代码就会易懂一些啦。

- source-map-loader：当然是为了 source-map 了

关于插件 plugins，这里主要用了一个`html-webpack-plugin`，用于插入`index.html`页面。

上码：

```javascript
var webpack = require("webpack");
var path = require("path"); //引入node的path库
var HtmlwebpackPlugin = require("html-webpack-plugin");

var config = {
  entry: {
    app: [path.resolve(__dirname, "src/index.js")]
  }, //入口文件
  output: {
    path: path.resolve(__dirname, "dist"), // 指定编译后的代码位置为 dist/bundle.js
    filename: "./bundle.js"
  },
  module: {
    rules: [
      // 为webpack指定loaders
      {
        test: /\.js$/,
        use: ["source-map-loader"],
        enforce: "pre"
      },
      {
        test: /\.jsx?$/,
        use: ["babel-loader"],
        exclude: /node_modules/
      }
    ]
  },
  plugins: [
    new HtmlwebpackPlugin({
      title: "cycle.js demo",
      template: path.resolve(__dirname, "index.html"),
      inject: "body"
    }),
    new webpack.optimize.UglifyJsPlugin({
      compress: {
        warnings: false
      }
    })
  ],
  devtool: "source-map"
};

module.exports = config;
```

### webpackServer.config.js

```javascript
var webpack = require("webpack");
var path = require("path"); //引入node的path库
var config = require("./webpack.config.js");
config.entry.app.concat([
  "webpack/hot/dev-server",
  "webpack-dev-server/client?http://localhost:3333"
]);
module.exports = config;
```

## 启动应用

### 项目代码

关于代码其实很简单，从官方代码里偷过来一个`app-root`的模块组件，然后拼到项目里，最终长这么一个样：

```js
import { run } from "@cycle/run";
import { makeDOMDriver } from "@cycle/dom";
import { App } from "./app";

const main = App;

const drivers = {
  DOM: makeDOMDriver("#app")
};

run(main, drivers);
```

至于 App 长这样：

```js
import xs from "xstream";
import { run } from "@cycle/run";
import { makeDOMDriver } from "@cycle/dom";
import { html } from "snabbdom-jsx";

export function App(sources) {
  const sinks = {
    DOM: sources.DOM.select("input")
      .events("click")
      .map(ev => ev.target.checked)
      .startWith(false)
      .map(toggled => (
        <div>
          <input type="checkbox" /> Toggle me
          <p>{toggled ? "ON" : "off"}</p>
        </div>
      ))
  };
  return sinks;
}
```

其实使用 jsx 之后，跟 React 是有些相似，但数据和状态的管理则不一致了，毕竟从代码就能看到，我们是从一个`click`事件发起数据源，然后更新 Dom 的。

## 结束语

这节主要讲了 webpack/babel 一些相关配置，项目已经搭建起来了，但是很多东西我们还不完全理解，像 Driver、xstream 等等，后面大概也需要讲讲了呢。  
[此处查看项目代码](https://github.com/godbasin/godbasin.github.io/tree/blog-codes/cyclejs-notes/1-init-app-with-webpack)   
[此处查看页面效果](http://cyclejs-notes.godbasin.com/1-init-app-with-webpack/index.html)
