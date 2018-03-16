---
layout: post
author: 碎碎酱
title: SVG图片动态修改颜色, 并渲染
permalink: svg-image-dynamically-modify-colors-and-render
date: 2018-03-16
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
lede: 
---

SVG全称为Scalable Vector Graphics, 意思为可缩放的矢量图形, 它是基于XML语言描述的. 下面是由UI同学导出的svg座位图的代码

```xml
<?xml version="1.0" encoding="UTF-8"?>
<svg width="32px" height="32px" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <!-- Generator: Sketch 47.1 (45422) - http://www.bohemiancoding.com/sketch -->
    <title>可选-5</title>
    <desc>Created with Sketch.</desc>
    <defs>
        <path d="M8,0.000240753089 L24,0.000240753089 L24,0.000240753089 C26.7614237,0.000240753089 29,2.238817 29,5.00024075 L29,25 L29,25 C29,27.7614237 26.7614237,30 24,30 L8,30 L8,30 C5.23857625,30 3,27.7614237 3,25 L3,5.00024075 L3,5.00024075 C3,2.238817 5.23857625,0.000240753089 8,0.000240753089 Z" id="path-1"></path>
        <rect id="path-2" x="0" y="5" width="7" height="23" rx="3"></rect>
        <rect id="path-3" x="25" y="5" width="7" height="23" rx="3"></rect>
    </defs>
    <g id="Symbols" stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g id="可选-5">
            <g id="可选" transform="translate(0.000000, 1.000000)">
                <g id="Rectangle-7">
                    <use fill="#C0DD7A" fill-rule="evenodd" xlink:href="#path-1"></use>
                    <path stroke="#97B846" stroke-width="1" d="M8,0.500240753 C5.51471863,0.500240753 3.5,2.51495938 3.5,5.00024075 L3.5,25 C3.5,27.4852814 5.51471863,29.5 8,29.5 L24,29.5 C26.4852814,29.5 28.5,27.4852814 28.5,25 L28.5,5.00024075 C28.5,2.51495938 26.4852814,0.500240753 24,0.500240753 L8,0.500240753 Z"></path>
                </g>
                <g id="Rectangle-8">
                    <use fill="#C0DD7A" fill-rule="evenodd" xlink:href="#path-2"></use>
                    <rect stroke="#97B846" stroke-width="1" x="0.5" y="5.5" width="6" height="22" rx="3"></rect>
                </g>
                <g id="Rectangle-8-Copy">
                    <use fill="#C0DD7A" fill-rule="evenodd" xlink:href="#path-3"></use>
                    <rect stroke="#97B846" stroke-width="1" x="25.5" y="5.5" width="6" height="22" rx="3"></rect>
                </g>
                <rect id="Rectangle-2" fill="#97B846" x="7" y="24" width="19" height="1"></rect>
            </g>
        </g>
    </g>
</svg>
```

为了缩小体积, 我们可以将部分标签, 注释内容, 没使用的id都去掉. 修改svg的填充色(fill)和线条色(stroke), 将svg代码保存为字符串

```js
const fillColor = 'red';
const lineColor = 'blue';
const svg = `
<svg width="32px" height="32px" viewBox="0 0 32 32" version="1.1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink">
    <defs>
        <path d="M8,0.000240753089 L24,0.000240753089 L24,0.000240753089 C26.7614237,0.000240753089 29,2.238817 29,5.00024075 L29,25 L29,25 C29,27.7614237 26.7614237,30 24,30 L8,30 L8,30 C5.23857625,30 3,27.7614237 3,25 L3,5.00024075 L3,5.00024075 C3,2.238817 5.23857625,0.000240753089 8,0.000240753089 Z" id="p1"></path>
        <rect id="p2" x="0" y="5" width="7" height="23" rx="3"></rect>
        <rect id="p3" x="25" y="5" width="7" height="23" rx="3"></rect>
    </defs>
    <g stroke="none" stroke-width="1" fill="none" fill-rule="evenodd">
        <g>
            <g transform="translate(0.000000, 1.000000)">
                <g>
                    <use fill="${fillColor}" fill-rule="evenodd" xlink:href="#p1"></use>
                    <path stroke="${lineColor}" stroke-width="1" d="M8,0.500240753 C5.51471863,0.500240753 3.5,2.51495938 3.5,5.00024075 L3.5,25 C3.5,27.4852814 5.51471863,29.5 8,29.5 L24,29.5 C26.4852814,29.5 28.5,27.4852814 28.5,25 L28.5,5.00024075 C28.5,2.51495938 26.4852814,0.500240753 24,0.500240753 L8,0.500240753 Z"></path>
                </g>
                <g>
                    <use fill="${fillColor}" fill-rule="evenodd" xlink:href="#p2"></use>
                    <rect stroke="${lineColor}" stroke-width="1" x="0.5" y="5.5" width="6" height="22" rx="3"></rect>
                </g>
                <g>
                    <use fill="${fillColor}" fill-rule="evenodd" xlink:href="#p3"></use>
                    <rect stroke="${lineColor}" stroke-width="1" x="25.5" y="5.5" width="6" height="22" rx="3"></rect>
                </g>
                <rect fill="${lineColor}" x="7" y="24" width="19" height="1"></rect>
            </g>
        </g>
    </g>
</svg>
`;
```

我们要基于svg代码生成Blob对象, 类型为 image/svg+xml. Blob 对象表示不可变的类似文件对象的原始数据. [Blob文档](https://developer.mozilla.org/en-US/docs/Web/API/Blob)

```js
const blob = new Blob([svg], { type: 'image/svg+xml' });
```

使用URL.createObjectURL()获取Blob对象的url, 该url与普通图片的http url效果相同.

```js
const url = URL.createObjectURL(blob);
```

## 渲染生成的svg图片

```js
// 渲染到文档流
const image = new Image();
image.src = url;
document.body.appendChild(image);

// 渲染到canvas
const canvas = document.getElementById('canvas');
const ctx = canvas.getContext('2d');
ctx.drawImage(url, 0, 0);
```