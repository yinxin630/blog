---
layout: post
author: 碎碎酱
title: react-native踩坑记录 - 视图莫名其妙的下移一段距离
permalink: react-native-tread-the-pit-record-scrollview-style-bug
date: 2017-01-24
thumbnail: /img/react-native-logo.png
category: react-native
tags:
- react-native
lede: "react-native直接在ScrollView上应用样式时, 可能会导致些奇怪的问题"
---

## BUG表现

在app中, 初次进入该页面时, 视图能占满屏幕, 而切换到别的tab页再切回来的时候, 视图上分多出20px的空白区域.

初次进入, 页面占满屏幕:
![2017-01-24-react-native-tread-the-pit-record-scrollview-style-bug-01.png](/img/2017-01-24-react-native-tread-the-pit-record-scrollview-style-bug-01.png)

从别的tab页回来, 页面下移一段距离:
![2017-01-24-react-native-tread-the-pit-record-scrollview-style-bug-02.png](/img/2017-01-24-react-native-tread-the-pit-record-scrollview-style-bug-02.png)

因为所有页面有一个公共的顶层View, 为了让出状态栏的空间, 该View包含`padding: 20`属性, 而真人秀页面的顶层容器是一个ScrollView, 并且为了占满屏幕, 添加了`marginTop: -20`属性. 同时还有另一模块也是要占满屏幕, 同样设置了`marginTop: -20`, 但是这个模块就没有切换后下移的问题.

## 造成BUG的原因

通过对比该页面和另一个没问题的页面, 发现区别仅仅是两个页面的容器不同, 本页面用的ScrollView, 而另一个页面用的View, View里面是一个无样式的ScrollView, 如下所示:

```jsx
// 本页面
<ScrollView style={styles.container}>
    ...其它内容
</ScrollView>
```

```jsx
// 表现正常的页面
<View style={styles.container}>
    <ScrollView>
        ...其它内容
    </ScrollView>
</View>
```

所以, 直接将样式放在ScrollView上会产生奇怪的问题, 请使用View + ScrollView的方式吧.