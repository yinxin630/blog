---
layout: post
author: 碎碎酱
title: 如何调试低版本安卓原生浏览器
permalink: how-to-debug-low-version-android-native-browser
date: 2016-10-25
thumbnail: /img/android-logo.png
category: android
tags:
- android
lede: "低版本安卓的浏览器(<= v4.2)很多html5和css3的东东不支持有木有, 不能用chrome远程debug有木有, 那怎么办? 页面空白只能一脸懵逼?"
---


低版本安卓的浏览器(<= v4.2)很多html5和css3的东东不支持有木有, 不能用chrome远程debug有木有, 那怎么办? 页面空白只能一脸懵逼?

## 解决办法

地址栏输入`about:debug`并回车, 能看console输出了有木有, 一脸懵逼状态解除