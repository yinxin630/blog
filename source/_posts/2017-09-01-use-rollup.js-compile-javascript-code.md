---
layout: post
author: 碎碎酱
title: 使用rollup.js编译JavaScript代码
permalink: use-rollup.js-compile-javascript-code
date: 2017-09-01
thumbnail: /img/rollup-logo.png
category: javascript
tags:
- javascript
- rollup
lede: rollup 是一款小巧的 JavaScript 打包工具, 它更适合用作库应用的构建工具
---

rollup 是一款小巧的 JavaScript 打包工具, 它更适合用作库应用的构建工具

[rollup文档](https://rollupjs.org/)

## 安装 rollup

`npm install -g rollup`

## 使用 rollup

创建 `main.js` 文件
```js
console.log(111);
```

执行 `rollup --input main.js --output bundle.js --format cjs`, 该命令编译 `main.js` 生成 `bundle.js`, `--format cjs` 意味着打包为 node.js 环境代码, 请观察 `bundle.js` 文件内容

### 关于 format 选项

rollup 提供了三种选项

* cjs: node.js 环境
* iife: 浏览器环境
* umd: 兼容环境, 同时支持 node.js 和浏览器

### 监听改动并自动重新打包

rollup 提供了 `--watch / -w` 参数来监听文件改动

### 使用配置文件

rollup 支持使用配置文件, 创建 `rollup.config.js` 文件
```js
export default {
    input: './main.js',
    output: {
        file: 'bundle.js',
        format: 'umd'
    },
}
```

使用配置文件启动 `rollup -c rollup.config.js`, 如果配置文件名为 rollup.config.js, 也可以省略文件名

更多的配置项可以查看 [rollup options](https://rollupjs.org/#big-list-of-options)

使用配置文件时, 你仍可以传递命令行选项, 命令行选项会覆盖配置文件中的配置

### 使用 rollup 插件

rollup 很多功能都是通过插件实现的, 比如打包 json 文件

安装 json 插件, `npm install --save-dev rollup-plugin-json`

在配置文件中启用插件
```js
import json from 'rollup-plugin-json';

export default {
    input: './main.js',
    output: {
        file: 'bundle.js',
        format: 'umd'
    },
    plugins: [
        json(),
    ],
}
```

创建 `data.json` 文件
```json
{
    "name": "rollup",
    "version": "v1.0.0"
}
```

在 `main.js` 中引入 `data.json`
```js
import { name } from './data.json';
console.log(name);
```

重复打包并查看 `bundle.js` 内容

*bundle.js*
```js
(function (global, factory) {
	typeof exports === 'object' && typeof module !== 'undefined' ? factory() :
	typeof define === 'function' && define.amd ? define(factory) :
	(factory());
}(this, (function () { 'use strict';

var name = "rollup";

console.log(name);

})));
```

可以看到 bundle 中仅包含 json 中的 name 字段, 这是因为 rollup 会自动进行 Tree-shaking, 我们的代码中仅仅引入了 name, version 并没有被使用, 因此 version 不会被打包

### 打包 npm 模块

rollup 并不会寻找 node_modules 目录模块, 该功能由插件 [rollup-plugin-node-resolve](https://github.com/rollup/rollup-plugin-node-resolve) 提供

### 打包 CommonJS 模块

rollup 仅支持使用 ES6 Module 语法定义的模块, 如果要引入 CommonJS 模块, 需要添加插件 [rollup-plugin-commonjs](https://github.com/rollup/rollup-plugin-commonjs)

添加后, 你可以使用 ES6 Module 语法来引入 CommonJS 模块. 注意: 你必须使用 ES6 Module 语法来引入, 如果使用 CommonJS 语法, 模块并不会被打包到 bundle 中, 而是保持 `require` 语句

### 使用 babel

添加插件 `rollup-plugin-babel`, 并创建 `.babelrc` 配置文件

如果你在代码中使用了第三方库, 比如 ramda, 它将会被完整打包进 bundle 中, 即便你仅仅使用了其中的一个方法. 添加 babel 插件 [babel-plugin-ramda](https://github.com/megawac/babel-plugin-ramda), 这样子将只会打包我们用到的部分, 大大减小 bundle 体积

## 一个完整的配置项

```js
import resolve from 'rollup-plugin-node-resolve';
import commonjs from 'rollup-plugin-commonjs';
import babel from 'rollup-plugin-babel';
import json from 'rollup-plugin-json';

export default {
    input: './main.js',
    output: {
        file: 'bundle.js',
        format: 'umd'
    },
    watch: {
        exclude: 'node_modules/**'
    },
    plugins: [
        resolve(),
        commonjs(),
        json(),
        babel({
            exclude: 'node_modules/**'
        })
    ],
}
```

## rollup 的优势

* 打包速度快
* 自动 Tree-shaking
* 配置简单

rollup 并不是 webpack 的替代品, 它将所有资源已文本的格式打包到一起, 对于一个前端项目来说, webpack 更合适, rollup 更适合用于构建库