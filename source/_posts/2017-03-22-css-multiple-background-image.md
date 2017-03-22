---
layout: post
author: 碎碎酱
title: css使用多张背景图, 并单独设置是否重复和作用范围
permalink: css-multiple-background-image
date: 2017-03-22
thumbnail: /img/css-logo.png
category: css
tags:
- css
lede: ""
---

## 要实现的效果
<img src="/img/2017-03-22-css-multiple-background-image-01.png">

## 已有的图片素材
<img src="/img/2017-03-22-css-multiple-background-image-02.png" style="width: 10px; height: 200px">
<img src="/img/2017-03-22-css-multiple-background-image-03.png" style="width: 10px; height: 200px">
<img src="/img/2017-03-22-css-multiple-background-image-04.png" style="width: 10px; height: 200px">

## 要点
由于中间的图片需要重复, 因此该图片将会与图片1/3重叠. 我们可以给容器设置 `padding` 或者 `border`, 然后使用 `background-clip` 和 `background-origin` 属性来单独指定每张背景图的作用范围, 将中间图片的作用范围设置为不包含 padding 或者 border 区域.

## 实现代码
以设置额外的 `padding` 为例, border 同理, 改为设置 `border` 和 `border-box` 即可.
```css
.container {
    background-image:
        url('./img1.png'),
        url('./img2.png'),
        url('./img3.png'),
    background-repeat: no-repeat, repeat-x, no-repeat;
    background-position: left, center, right;
    padding: 0 20px;
    background-clip: padding-box, content-box, padding-box;
    background-origin: padding-box, content-box, padding-box;
    background-size: 20px 100%;
}
```

## 相关链接
[background-repeat](https://developer.mozilla.org/en-US/docs/Web/CSS/background-repeat)  
[background-clip](https://developer.mozilla.org/en-US/docs/Web/CSS/background-clip)  
[background-origin](https://developer.mozilla.org/en-US/docs/Web/CSS/background-origin)  