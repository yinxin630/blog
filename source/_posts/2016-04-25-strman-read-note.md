---
layout: post
author: 碎碎酱
title: strman源码阅读笔记
date: 2016-04-25
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
- strman
lede: "strman是小巧的, 无任何依赖的string操作库, 前后端通用, 同时支持CommonJS和ES6 module模式. 该篇文章是阅读strman源码过程中, 对学习到的JavaScript知识点的记录."
---


strman是小巧的, 无任何依赖的string操作库, 前后端通用, 同时支持CommonJS和ES6 module模式. 该篇文章是阅读strman源码过程中, 对学习到的JavaScript知识点的记录. 

1. JavaScript函数参数对象 - arguments
    * 类型: `Object`
    * 转化为数组:
        1. Array.prototype.slice.call(arguments)
        2. Array.from(arguments)

2. 判断类型
    * 判断是否为字符串: `Object.prototype.toString.call(str) === '[object String]'`
    * JavaScript基础类型:
        * [object Number]
        * [object String]
        * [object Boolean]
        * [object Array]
        * [object Object]
        * [object Function]
        * [object Date]
        * [object Null]
        * [object Undefined]
        * [object Symbol]

3. 截取字符串 - String.substr(index, count)
    * 从`index`位置开始, 截取`count`个字符
    * 如果`index`大于或者等于`lenght`, 返回`''`
    * 如果`index`为负数, 则起点为`length` + `index`, 向右截取`count`个字符

4. 创建重复字符串 - String.prototype.repeat(str, count)

    ```
    let result = '*'.repeat(5);
    // result => '*****'
    ```
        
  [1]: http://suisuijiang.com/