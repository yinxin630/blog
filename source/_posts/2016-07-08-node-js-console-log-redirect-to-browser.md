---
layout: post
author: 碎碎酱
title: 将node.js的终端输出重定向到浏览器控制台中
permalink: node-js-console-log-redirect-to-browser
date: 2016-07-08
thumbnail: /img/nodejs-logo.png
category: javascript
tags:
- javascript
- node.js
lede: "在编写node.js代码时, 我们经常会临时的打印变量来查看内容, 对于一些基本的数据类型, 比如: `Number`, `String`, `Boolean`, 并没有什么问题, 但是当打印`Object`类型时, 如果该`Object`变量具有大量的属性, 我们会得到一个超长的输出内容, 有的时候甚至会超出终端缓冲区大小, 非常不利于观察结果. "
---


在编写node.js代码时, 我们经常会临时的打印变量来查看内容, 对于一些基本的数据类型, 比如: `Number`, `String`, `Boolean`, 并没有什么问题, 但是当打印`Object`类型时, 如果该`Object`变量具有大量的属性, 我们会得到一个超长的输出内容, 有的时候甚至会超出终端缓冲区大小, 非常不利于观察结果. 

使用过chrome浏览器开发者工具的朋友应该发现了, chrome的console会折叠`Object`值, 并且可以随意打开查看, 十分方便.

![2016-07-08-node-js-console-log-redirect-to-browser-01](/img/2016-07-08-node-js-console-log-redirect-to-browser-01.png)

# 将输出重定向

在这里, 我们使用开源组件[node-monkey](https://www.npmjs.com/package/node-monkey), `node-monkey`是一款在浏览器对node.js程序进行debug和性能分析的工具.

执行`npm install --save node_monkey`安装组件.

在你的程序入口添加如下代码:

```
require('node-monkey').start()
```

运行你的程序, 可以看到终端中的输出如下:

*console log*
```
⇒  node app.js
   info  - socket.io started
------------------
NodeMonkey started
To inspect output, open a browser to: http://127.0.0.1:50500
------------------
```

打开浏览器访问`http://127.0.0.1:50500`, 打开浏览器的控制台, 可以看到你的程序的输出内容.

![2016-07-08-node-js-console-log-redirect-to-browser-02](/img/2016-07-08-node-js-console-log-redirect-to-browser-02.png)

 `node-monkey`默认关联了`console.log`, `console.warn`, `console.error`输出函数, `127.0.0.1:50500`是`node-monkey`的默认地址, 可以通过在`start()`中传入参数来修改.

# 自动打开浏览器

有个小工具可以让我们自动打开默认浏览器并跳转到指定地址, 免去手动输入的过程.

安装`opener`, 执行`npm install --save opener`

在上述所添加的代码下方加上如下代码:

```
require("opener")('http://127.0.0.1:50500');
```

重新运行你的程序, 启动`node-money`后就自动打开到指定页面了.

`node-monkey`还有其它可用的功能, 您可以查询文档研究它的妙用.