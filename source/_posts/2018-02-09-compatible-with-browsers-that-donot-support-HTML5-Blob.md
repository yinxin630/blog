---
layout: post
author: 碎碎酱
title: 兼容不支持HTML5 Blob的浏览器
permalink: compatible-with-browsers-that-donot-support-HTML5-Blob
date: 2018-02-09
thumbnail: /img/javascript-logo.png
category: html
tags:
- html
- javascript
lede: 部分浏览器(webview)不支持HTML5新增的Blob, 会抛出异常TypeError Illegal Constructor, 这种情况可以使用BlobBuilder做兼容处理
---

部分浏览器(webview)不支持HTML5新增的Blob, 会抛出异常TypeError Illegal Constructor, 这种情况可以使用BlobBuilder做兼容处理

封装后的方法如下:
```js
function getBlob(svg) {
    let blob = null;
    try {
        blob = new Blob([svg], { type: 'image/svg+xml' });
    } catch (err) {
        if (window.BlobBuilder) {
            const bb = new window.BlobBuilder();
            bb.append([svg]);
            blob = bb.getBlob('image/svg+xml');
        }
    }
    return blob;
}
```