---
layout: post
author: 碎碎酱
title: Webpack – 灵活, 可拓展, 开源的模块打包工具
permalink: webpack-flexible-extensible-opensource-module-bundle-tool
date: 2016-04-20
thumbnail: /img/webpack.png
category: javascript
tags:
- webpack
- frontend
lede: "如今, 传统网站已经进化成了web应用程序: 1. 页面中越来越多的JavaScript代码 2. 现代浏览器允许做更多的事情 3. 很少全页刷新 → 一页中出现更多的代码"
---


## 前提

如今, 传统网站已经进化成了web应用程序:

1. 页面中越来越多的JavaScript代码
2. 现代浏览器允许做更多的事情
3. 很少全页刷新 → 一页中出现更多的代码

因此, 越来越多的代码出现在客户端, 需要合理的组织这些代码, 而使用模块系统可以让你分割代码到不同的模块里. 目前已有[多种方法][1]可以导入/到处模块:

* `<script>`标签(不包含模块系统)
* CommonJs
* AMD
* ES6 modules
* and more…

以上这些方法均有各自的支持者和反对者, 大家各持己见,  webpack提供了一个公平的解决方案, 使得开发者可以自己选择模板组织方式, 不同的组织方式可以混合使用. (注: webpack并不支持es6 module, 但是可以先用babel转换为es5)

## 为什么使用webpack

![2016-04-20-webpack-flexible-extensible-opensource-module-bundle-tool-01](/img/2016-04-20-webpack-flexible-extensible-opensource-module-bundle-tool-01.png)

webpack可以导入/导出组件库, 并将它们打包成静态资源.

### webpack目标:

* 将依赖树分割成块以便按需加载
* 降低首次加载时间
* 任何静态资源都可以是一个模块
* 将第三方库整合为组件
* 能够自定义任意部分的打包工具
* 适合大型项目

### webpack有特点: 

#### 代码分割

webpack可以将依赖树分割成不同的代码块, 并且提供了同步和异步两种加载模块的方式, 使得按需加载成为可能.

#### 加载器

webpack原生只能够处理JavaScript, 但是加载器可以让你在JavaScript中处理任何资源, 使得任何资源都如同一个模块.

#### 插件系统

webpack拥有丰富的插件系统, 大多数的内部功能都依赖于插件系统, 这使你可以按照需求来定制化webpack, 你还可以分享你的插件.

## 安装

### 安装node.js

首先安装[node.js][3], node.js附带一个包管理工具: `npm`

### 安装webpack

通过`npm`安装webpack: 

```
$ npm install webpack -g
```

### 在项目中添加webpack

将webpack添加到项目依赖中是非常不错的, 这样你可以选择局部版本的webpack, 而不必被迫使用全局的那一份.

添加一个`package.json`到项目中:

```
$ npm init
```

将`webpack`添加到`package.json`中:

```
$ npm install webpack --save-dev
```

## 使用

### 创建一个JavaScript模块

我们使用CommonJs语法定义一个JavaScript模块, 就像node.js那样:

**cats.js**
```javascript
var cats = ['dave', 'henry', 'martha'];
module.exports = cats;
```

**app.js**
```javascript
cats = require('./cats.js');
console.log(cats);
```

### 编译JavaScript

给出输入文件和输出文件: `webpack ./app.js app.bundle.js`

### 启动文件监视

执行webpack打包时, 将会从零开始打包, 当打包时间过长时, 会耗费大量的时间, webpack提供了文件监视功能, 可以自动获取文件的变化, 并重新打包.

执行`webpack ./app.js app.bundle.js --watch`

修改`cats.js`, 在cats数组中添加`tom`并保存, 可以看到webpack会自动发现改动, 重新打包输出.

### 使用配置文件

webpack既可以通过命令行参数启动, 也可以选择直接从配置文件启动, 创建`webpack.config.js`配置文件, 添加如下内容:

**webpack.config.js**
```javascript
module.exports = {
    entry: './app.js',
    output: {
        filename: __dirname + '/app.bundle.js'
    },
    watch: true
}
```

输入命令`webpack`启动webpack, 效果与命令`webpack ./app.js app.bundle.js --watch`相同, 后续会将配置内容均放入配置文件中.

[查看完整代码][4]

## 进阶

### 使用加载器

加载器应用于资源文件上, 它们(运行于node.js)将资源文件作为输入参数, 转换为新的资源加入源码中, 譬如你可以使用加载器告诉webpack如何加载CoffeeScript或者JSX.

#### 加载器特性

* 加载器可以被链式调用, 最终的加载器返回JavaScript, 中间的加载器可以返回任意格式, 这些内容将被传递到下一加载器
* 加载器即可以是同步的, 也可以是异步的
* 加载器运行在node.js环境中, 可以做任何能做到的事情
* 加载器可以接收参数, 以便传递配置信息到加载器
* 加载器可以在配置通过拓展或者正则表达式绑定
* 加载器可以通过`npm`发布或安装
* 加载器可以访问配置文件
* 插件可以给加载器提供更多功能
* 加载器可以输出额外的, 任意多个的文件

#### 安装加载器

通过npm安装: `$ npm install xxx-loader --save-dev`(xxx为需要的加载器类型)

#### 使用加载器

以css加载器为例, 安装加载器`npm install --save-dev style-loader css-loader`

**style.css**
```css
h1 {
    color: red;
}
h2 {
    color: purple;
}
```

**app.js**
```javascript
require('./style.css');
console.log('app.js');
```

**html.js**
```html
<!doctype html5>
<html>
<head>
    <title>example02</title>
</head>
<body>
    <h1>example02</h1>
    <h2>example02</h2>
    <p>example02</p>
    <script src="./app.bundle.js"></script>
</body>
</html>
```

##### 1. 通过命令行调用加载器

输入命令`webpack --module-bind 'css=style!css'`, 打包成功后用浏览器打开`index.html`, 可以看到css被正确应用到页面中.

##### 2. 通过require()调用加载器

修改app.js:

**app.js**
```javascript
require('style!css!./style.css');
console.log('app.js');
```

输入命令`webpack`, 效果同上.

##### 3. 通过配置文件调用加载器

注: 因按照官方文档做法出现报错, 暂时未解决, 所以暂不介绍这种方式. 想尝试的可以去看看[官方文档][5], 有解决办法请@我 (^_^)

[查看完整代码][6]

### 使用插件

使用插件可以给webpack添加更多的功能, 例如, [BellOnBundlerErrorPlugin][7]插件可以提示你打包过程中发生的错误. webpack有着丰富的[插件库][8].

#### 安装插件

可以从npm安装 `npm install component-webpack-plugin`

#### 使用插件

在webpack配置文件中添加`plugins`配置, 如下所示:

**webpack.config.js**
```javascript
var ComponentPlugin = require("component-webpack-plugin");
module.exports = {
    plugins: [
        new ComponentPlugin()
    ]
} 
```


  [1]: https://webpack.github.io/docs/motivation.html
  [2]: http://webpack.github.io/assets/what-is-webpack.png
  [3]: http://nodejs.org/
  [4]: https://github.com/yinxin630/webpack-example/tree/master/example01
  [5]: https://webpack.github.io/docs/using-loaders.html
  [6]: https://github.com/yinxin630/webpack-example/tree/master/example02
  [7]: https://github.com/senotrusov/bell-on-bundler-error-plugin
  [8]: https://webpack.github.io/docs/list-of-plugins.html