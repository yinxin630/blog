---
layout: post
author: 碎碎酱
title: Javascript函数安全调用小技巧
permalink: javascript-function-safe-call-tip
date: 2017-12-04
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
lede: 最近遇到了一个问题, 场景如下. 有一部分功能需要依赖第三方js文件, 由于业务代码是离线化的而第三方js文件却是从网络获取, 其加载时间必定晚于业务代码执行时间
---

最近遇到了一个问题, 场景如下. 有一部分功能需要依赖第三方js文件, 由于业务代码是离线化的而第三方js文件却是从网络获取, 其加载时间必定晚于业务代码执行时间. 那么如何在不修改业务代码的情况下, 又保证不会因为加载顺序而出问题呢? 我们可以将每个方法分别包装一层替代原本的方法, 在包装方法中, 如果判断加载完毕可以调用了, 则调用方法, 并将原方法替换会包装方法. 如果未加载完毕, 则延时一段时间, 继续调用包装方法, 形成递归, 直到其加载完毕才产生实际调用.

## 举个栗子

```js
// `window.lib` 是第三方库注入的对象
window.lib = null;

// 使用 setTimeout 模拟延迟加载
setTimeout(() => {
  window.lib = {
    add: function(a, b) {
      return new Promise(resolve => resolve(a + b));
    },
    log: function(c) {
      console.log(c);
    }
  };
}, 2000);

// 业务代码
var actions = {
  func: function(a, b) {
    // 调用第三方库对象方法
    window.lib.add(a, b).then(result => {
      window.lib.log(result);
      console.log(result);
    });
  }
};

// 因为第三方库未注入, 代码会报错 'test.js:15 Uncaught TypeError: Cannot read property 'add' of null'
actions.func(2, 3);
```

接下来编写包装方法:
```js
function wrap(target, funcName) {
  // 保存原函数
  var origin = target[funcName];
  // 修改原对象设置包装方法
  target[funcName] = function(...args) {
    // 返回Promise使得调用者可以拿到异步结果
    return new Promise(resolve => {
      // 如果库加载完毕, 就调用原方法, 并修改原对象还原为原方法
      if (window.lib) {
        resolve(origin.call(target, ...args));
        target[funcName] = origin;
      } else {
        // 如果未加载完毕, 延迟100ms再次调用包装方法
        setTimeout(() => {
          resolve(target[funcName].call(null, ...args));
        }, 100);
      }
    });
  };
}
```

## 完整代码

```js
window.lib = null;

setTimeout(() => {
  window.lib = {
    add: function(a, b) {
      return new Promise(resolve => resolve(a + b));
    },
    log: function(c) {
      console.log(c);
    }
  };
}, 2000);

var actions = {
  func: function(a, b) {
    window.lib.add(a, b).then(result => {
      window.lib.log(result);
      console.log(result);
    });
  }
};

function wrap(target, funcName) {
  var origin = target[funcName];
  target[funcName] = function(...args) {
    return new Promise(resolve => {
      if (window.lib) {
        resolve(origin.call(target, ...args));
        target[funcName] = origin;
      } else {
        setTimeout(() => {
          resolve(target[funcName].call(null, ...args));
        }, 100);
      }
    });
  };
}
// 将原方法修改为安全的包装方法
wrap(actions, "func");

actions.func(2, 3);
```

执行代码, 不会再抛出异常