---
layout: post
author: 碎碎酱
title: node.js中实现代码Hot-Reload功能
permalink: in-nodejs-realize-code-hot-reload-function
date: 2016-07-06
thumbnail: /img/nodejs-logo.png
category: node.js
tags:
- javascript
- node.js
lede: "使用过webpack做前端的朋友大概知道, webpack有个dev-server功能, 支持在代码发生改动时自动的重启代码, 简称hot-reload. 这一特性极大地促进了开发效率, 所以下面我们通过编写代码来实现这个特性."
---


使用过webpack做前端的朋友大概知道, webpack有个dev-server功能, 支持在代码发生改动时自动的重启代码, 简称hot-reload. 这一特性极大地促进了开发效率, 所以下面我们通过编写代码来实现这个特性.

# 创建开发目录

首先创建一个`src`目录, 后面我们将监视`src`目录中的代码改动, 在`src`目录创建`app.js`做程序入口:

*app.js*
```
console.log('app.js');
```

# 使用chokidar监视目录

[chokidar](https://www.npmjs.com/package/chokidar)是一个小巧的文件监视库, 它可以获取到目录中文件的改动/新增/删除等事件. 在项目根目录创建`development.js`, `development.js`代码如下所示:

*development.js*
```
'use strict'

const path = require('path');
const chokidar = require('chokidar');
const watcher = chokidar.watch(path.join(__dirname, '/src'));

watcher.on('ready', () => {
    watcher.on('change', (path) => {
        console.log('<---- watched file change, do something ---->');
    });

    watcher.on('add', (path) => {
        console.log('<---- watched new file add, do something ---->');
    });

    watcher.on('unlink', (path) => {
        console.log('<---- watched file remove, do something ---->');
    });
});
```

执行`npm install --save chokidar`安装'chokidar'组件, 并执行`node development.js`运行程序, 尝试修改`app.js`的内容, 尝试在`src`目录中新建文件并删除新建的文件, 会看到如下所示输出:

*console log*
```
⇒  node development.js
<---- watched file change, do something ---->
<---- watched new file add, do something ---->
<---- watched file remove, do something ---->
```

# 引入程序并在发生改动时重启

由于主进程被用来监视文件了, 所以我们要把主程序运行在子进程中, 当代码发生改动时, 我们可以结束这个子进程并创建新的子进程. 创建进程需要使用[child_process](https://nodejs.org/api/child_process.html)模块, 代码如下所示:

*创建子进程, 启动主程序*
```
let appIns = cp.fork(path.join(__dirname, '../src/app.js'));
```

*发生改动时杀死子进程并重启*
```
appIns.kill('SIGINT');
appIns = cp.fork(require('path').join(__dirname, '../src/app.js'));
```

*监听SIGINT信息, 终止进程*
```
process.on('SIGINT', () => {
    process.exit(0);
});
```

完整代码如下所示:

*development.js*
```
'use strict'

const path = require('path');
const cp = require('child_process');
const chokidar = require('chokidar');
const watcher = chokidar.watch(path.join(__dirname, '/src'));

let appIns = cp.fork(path.join(__dirname, '/src/app.js'));

watcher.on('ready', () => {
    watcher.on('change', (path) => {
        console.log('<---- watched file change, do something ---->');
        appIns = reload(appIns);
    });

    watcher.on('add', (path) => {
        console.log('<---- watched new file add, do something ---->');
        appIns = reload(appIns);
    });

    watcher.on('unlink', (path) => {
        console.log('<---- watched file remove, do something ---->');
        appIns = reload(appIns);
    });
});

process.on('SIGINT', () => {
    process.exit(0);
});

function reload(appIns) {
    appIns.kill('SIGINT');
    return cp.fork(require('path').join(__dirname, '/src/app.js'));
}
```

执行`node development.js`运行, 尝试修改`app.js`, 改为`console.log('app.js changed');`, 将看到如下输出:

*console log*
```
⇒  node development.js
app.js
<---- watched file change, do something ---->
app.js changed
```

尝试在`src`目录中创建`a.js`, 并修改`app.js`使其输出`a.js`内容, 这里不在粘贴输出, 请自行尝试.

# 结语

可以看到该实现方法十分简单, 并且适用于任何node.js开发场景, 譬如常用的express, koa后端接口开发.

感谢您的阅读, 欢迎留言指导讨论.