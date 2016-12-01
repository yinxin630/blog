---
layout: post
author: 碎碎酱
title: CSS中清除浮动的几种方法
permalink: several-ways-to-clear-float-in-css
date: 2016-10-10
thumbnail: /img/css-logo.png
category: css
tags:
- css
lede: "在css中有多种办法来清除float浮动的办法."
---


在css中使用浮动进行布局时, 例如如下的代码:

```
<!doctype html>
<html>
    <head>
        <title>test.html</title>
        <meta charset="utf-8">
        <style>
            .child {
                width: 100px;
                height: 100px;
                float: left;
            }
            .child1 {
                background-color: red;
            }
            .child2 {
                background-color: blue;
            }
        </style>
    </head>
    <body>
        <div class="container">
            <div class="child child1"></div>
            <div class="child child2"></div>
        </div>
        <p>元素指定的任何内边距和边框都将在已设定的宽度和高度内进行绘制</p>
    </body>
</html>
```

<img src="http://cdn.suisuijiang.com/message_1476072629494.png" width="706" height="149" class="aligncenter" />

p标签这个块级元素会跟在child2那个div后面, 而不是在新行渲染, 因此我们需要清除浮动, 来恢复container的块元素效果.

## 解决办法

0. 父元素定义height
    将父元素的高度定义为子元素的高度, 以便将父元素撑起.
    ```
    .container {
        height: 100px;
    }
    ```

0. 父元素末尾添加元素
    在父元素的末尾添加一个具有clear:both属性的块级子元素, 也可以是具有display:block属性的行内元素, 该元素必须是个空元素, 不能包含其它内容.
    ```
    .float-fix {
        clear:both;
    }

    <div class="container">
        <div class="child child1"></div>
        <div class="child child2"></div>
        <div class="float-fix"/>
    </div>
    ```

0. 父元素定义overflow
    给父元素添加overflow属性, 值可以为hidden或者auto.
    ```
    .container {
        overflow: hidden;
    }
    ```

0. 父元素也浮动, 并且设置父元素宽度
    ```
    .container {
        float: left;
        width: 100%;
    }
    ```

0. 父元素设置display: table
    ```
    .container {
        display: table;
    }
    ```

0. 父元素定义伪类
    给父元素定义:after伪类, 并且使用类似方法2的方法来清除浮动.
    ```
    .container:after {
        content: "";
        display: block;
        clear: both;
    }
    ```
