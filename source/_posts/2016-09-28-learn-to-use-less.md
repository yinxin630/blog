---
layout: post
author: 碎碎酱
title: 学习使用Less小记
date: 2016-09-28
thumbnail: /img/less-logo.png
category: css
tags:
- css
- less
lede: "Less是一个css预处理工具, 它拓展了css的语法, 添加了一系列特性."
---


## 关于Less

Less是一个css预处理工具, 它拓展了css的语法, 添加了一系列特性.

## 使用Less

### 在命令行使用

全局安装Less:

```
npm install -g less
```

0. 编译less文件
    ```
    lessc styles.less
    ```

0. 编译less文件并指定文件名
    ```
    lessc styles.less styles.css
    ```

0. 编译less文件并产生压缩后的代码
    ```
    lessc --clean-css styles.less styles.min.css
    ```

### 在webpack中使用

在webpack配置中添加loader, 并安装相应loader包 `npm install less less-loader css-loader style-loader`

```
{
    test: /\.less?$/,
    loaders: ['style-loader', 'css-loader', 'less-loader'],
}
```

### 在客户端使用

引入less库: `<script src="less.js" type="text/javascript"></script>`
添加less文件: `<link rel="stylesheet/less" type="text/css" href="styles.less" />`

### 在node.js中使用

```
var less = require('less');

less.render('.class { width: (1 + 1) }', function (e, output) {
    console.log(output.css);
});
```

## Less语法

### 变量

0. 变量属性值
    ```
    @backColor: green;

    body {
        background-color: @backColor;
    }
    ```

    编译为:
    ```
    body {
    background-color: green;
    }
    ```

0. 变量属性
    ```
    @bkColor: background-color;

    body {
        @{bkColor}: red;
    }
    ```

    编译为:
    ```
    body {
    background-color: red;
    }
    ```

0. 变量选择器
    ```
    @b: body;

    @{b} {
        background-color: yellow;
    }
    ```

    编译为:
    ```
    body {
    background-color: yellow;
    }
    ```

0. 变量URL
    ```
    @domain: "http://cdn.suisuijiang.com/";

    body {
        background-image: url("@{domain}user_57d65f739e1dc93d74ce4811_1473667014140.gif");
    }
    ```

    编译为:
    ```
    body {
    background-image: url("http://cdn.suisuijiang.com/user_57d65f739e1dc93d74ce4811_1473667014140.gif");
    }
    ```

0. 变量做变量名
    ```
    @pink: pink;
    @other: "pink";

    body {
        background-color: @@other;
    }
    ```

    编译为:
    ```
    body {
    background-color: pink;
    }
    ```

0. 变量懒加载

    变量在使用前不需要定义, 可以把定义放在使用后, 变量值使用当前作用域最终值, 当前作用域不存在该变量时则取父作用域最终值

    ```
    body {
        @color: red;
        background-color: @color;
        @color: yellow;
    }

    @color: purple;
    ```

    编译为:
    ```
    body {
    background-color: yellow;
    }
    ```

0. 变量默认值

    由于变量懒加载机制的存在, 我们可以定义变量的默认值, 并在需要改变时直接赋其它值

    ```
    // library
    @base-color: green;

    // use of library
    @import "library.less";
    @base-color: red;
    ```

### 属性继承

0. 选择器内继承

    ```
    .red {
        background-color: red;
    }

    body {
        &:extend(.red);
        display: flex;
    }
    ```

    编译为:
    ```
    .red,
    body {
    background-color: red;
    }
    body {
    display: flex;
    }
    ```

0. 选择器继承

    ```
    .lawngreen {
        background-color: lawngreen;
    }

    body:extend(.lawngreen) {
        display: flex;
    }
    ```

    编译为:
    ```
    .lawngreen,
    body {
    background-color: lawngreen;
    }
    body {
    display: flex;
    }
    ```

0. 选择器内的子选择器继承

    ```
    .color {
        .aqua {
            background-color: aqua;
        }
        background-color: red;
    }

    body:extend(.color .aqua) {
        display: flex;
    }
    ```

    编译为:
    ```
    .color {
    background-color: red;
    }
    .color .aqua,
    body {
    background-color: aqua;
    }
    body {
    display: flex;
    }
    ```

0. 继承所有匹配的选择器

    ```
    .color.a {
        background-color: red;
    }
    .color.b {
        &:hover {
            background-color: blue;
        }
    }

    body:extend(.color all) { }
    ```

    编译为:
    ```
    .color.a,
    body.a {
    background-color: red;
    }
    .color.b:hover,
    body.b:hover {
    background-color: blue;
    }
    ```

0. 使用继承比使用属性混合的代码体积更小

    使用属性混合:
    ```
    .my-inline-block {
        display: inline-block;
        font-size: 0;
    }
    .thing1 {
        .my-inline-block;
    }
    .thing2 {
        .my-inline-block;
    }
    ```

    编译为:
    ```
    .thing1 {
    display: inline-block;
    font-size: 0;
    }
    .thing2 {
    display: inline-block;
    font-size: 0;
    }
    ```

    使用继承:
    ```
    .my-inline-block {
        display: inline-block;
        font-size: 0;
    }
    .thing1 {
        &:extend(.my-inline-block);
    }
    .thing2 {
        &:extend(.my-inline-block);
    }
    ```

    编译为:
    ```
    .my-inline-block,
    .thing1,
    .thing2 {
    display: inline-block;
    font-size: 0;
    }
    ```

### Mixins

0. 混合其它选择器属性

    ```
    .base {
        background-color: red;
    }
    body {
        .base();
    }
    ```

    编译为:
    ```
    .base {
    background-color: red;
    }
    body {
    background-color: red;
    }
    ```

0. 不输出混合

    在混合名称的末尾加上(), 可以不输出该混合
    ```
    .base() {
        background-color: red;
    }
    body {
        .base;
    }
    ```

    编译为:
    ```
    body {
    background-color: red;
    }
    ```

0. 使用!important

    ```
    .base() {
        background-color: red;
    }
    .flex() {
        display: flex;
    }
    body {
        .base;
        .flex !important;
    }
    ```

    编译为:
    ```
    body {
    background-color: red;
    display: flex !important;
    }
    ```

### 带参数的Mixins

0. 多个参数

    ```
    .size(@width: 100%, @height: 100%) {
        width: @width;
        height: @height;
    }
    .container {
        .size(100px, 100px);
        background-color: red;
    }
    ```

    编译为:
    ```
    .container {
    width: 100px;
    height: 100px;
    background-color: red;
    }
    ```

0. 命名参数传递

    ```
    .size(@width: 100%, @height: 100%) {
        width: @width;
        height: @height;
    }
    .container {
        .size(@height: 200px, @width: 100px);
        background-color: red;
    }
    ```

    编译为:
    ```
    .container {
    width: 100px;
    height: 200px;
    background-color: red;
    }
    ```

0. @arguments参数

    @arguments参数包含所有的参数值, 如果你不想用单个的参数值来拼接属性, 就可以直接使用@arguments
    ```
    .shadow(@x; @y; @blur; @color) {
        box-shadow: @arguments;
    }
    .container {
        width: 100px;
        height: 100px;
        .shadow(0px; 0px; 5px; red);
    }
    ```

    编译为:
    ```
    .container {
    width: 100px;
    height: 100px;
    box-shadow: 0px 0px 5px red;
    }
    ```

### Mixins作为函数

在minin中定义的变量和minxins可以在调用者的作用域使用

```
.size() {
    @width: 200px;
    @height: 100px;
}
.container {
    .size();
    width: @width;
    height: @height;
    background-color: red;
}
```

编译为:
```
.container {
  width: 200px;
  height: 100px;
  background-color: red;
}
```

因此, 我们可以将Mixin当做函数使用, 比如用于计算的函数
```
.sum(@a; @b) {
    @sum: @a + @b;
}
.container {
    .sum(22px; 88px);
    width: @sum;
    height: @sum;
    background-color: red;
}
```

编译为:
```
.container {
  width: 110px;
  height: 110px;
  background-color: red;
}
```
