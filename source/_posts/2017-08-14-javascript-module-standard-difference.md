---
layout: post
author: 碎碎酱
title: JS模块规范区别(CommonJS,AMD,CMD,UMD,ES6)
permalink: javascript-module-standard-difference
date: 2017-08-14
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
---

## CommonJS

同步, 主要为了用于服务端node.js使用  

`module` 定义模块, `module.export` 导出模块内容, `require` 引入模块

## AMD

异步, 用法为 `define(['dep1','dep2'],function(dep1,dep2){...});`, 代表库 RequireJS  

运行时先检查依赖库, 根据配置文件获取实际路径, 在dom中动态插入script脚本, 加载完毕后触发onLoad事件通知加载完成, 所有依赖都加载完毕后执行回调, 并将依赖作为参数传递进去

## CMD

异步, 用法为 `define(function(require,exports,module){...});`, 代表库 SeaJS

将回调函数转为字符串, 使用正则提取require语法获取依赖, 根据配置文件获取实际路径, 在dom中动态插入script脚本, 加载完毕后触发onLoad事件通知加载完成, 所有依赖都加载完毕后执行回调, 并将回调中的require语句替换为依赖

比如回调中存在 `var $ = require('jquery')`, 则回调执行时 $ 即为jquery实例

## UMD

umd是AMD和CommonJS的糅合, 是跨平台的解决方案. UMD先判断是否支持Node.js的模块 `exports` 是否存在, 存在则使用Node.js模块模式. 在判断是否支持AMD(`define`是否存在), 存在则使用AMD方式加载模块

## ES6 Module

来自 ECMAScript 2016, 浏览器暂时未支持, 需要转成 ES5 运行

## 参考

[JS模块规范（CommonJS,AMD,CMD,UMD,ES6）](https://my.oschina.net/tongjh/blog/836721)