---
layout: post
author: 碎碎酱
title: react native for android中组件不能显示GIF动态图的解决办法
permalink: react-native-android-image-can-not-show-gif-image-solution
date: 2016-06-23
thumbnail: /img/react-native-logo.png
category: react-native
tags: 
- react
- react-native
lede: "在android中运行react-native项目时, 发现gif图是静止的, 本文是该问题的解决办法."
featured: true
---

# 问题表现

在react native中, 编写如下代码:

```
<Image style={{ width: 100, height: 100 }} source={{ uri: 'http://x.xxx/x.gif' }}/>
```

gif图片不能正常显示, 显示为空白内容.

# 软件版本

* react-native: 0.27.2
* react: 15.1.0

# 解决办法

打开`./android/app/build.gradle`, 找到如下内容:

```
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
}
```

在 dependencies中添加`compile 'com.facebook.fresco:animated-gif:0.10.0'`, 如下所示:

```
dependencies {
    compile fileTree(dir: "libs", include: ["*.jar"])
    compile "com.android.support:appcompat-v7:23.0.1"
    compile "com.facebook.react:react-native:+"  // From node_modules
    compile 'com.facebook.fresco:animated-gif:0.10.0'
}
```

重新编译运行APP, 即可正常显示GIF图.

# 参考

[https://github.com/facebook/react-native/issues/7760#issuecomment-221898901](https://github.com/facebook/react-native/issues/7760#issuecomment-221898901)