---
layout: post
author: 碎碎酱
title: 关于setTimeout / setImmediate / process.nextTick的区别
permalink: setTimeout-setImmediate-process.nextTick-difference
date: 2017-08-19
thumbnail: /img/nodejs-logo.png
category: node.js
tags:
- node.js
- javascript
---

有这样一道面试题, 阅读如下代码, 请说出输出顺序

```js
setImmediate(() => {
    console.log(2);
});

setTimeout(() => {
    console.log(3);
}, 0);

process.nextTick(() => {
    console.log(4);
});

console.log(1);
```

答案是 `1432`, 与你预期的结果符合吗?

## 解析

首先, setImmediate, setTimeout, process.nextTick 均为异步操作, 而 `console.log(1)` 是同步, 所以一定在它们三个之前.

然后分析 setImmediate, setTimeout, process.nextTick 三者的区别:

* `setImmediate(callback[, ...args])`: callback 将在所有 I/O 事件处理完毕后执行, 多次使用 setImmediate 将会按顺序执行
* `setTimeout(callback, delay[, ...args])`: callback 将在 delay 时间后执行, 实际上并不会精确的在 delay 时间调用, 只会保证在 delay 时间立即添加到事件队列队尾
* `process.nextTick(callback[, ...args])`: callback 将会插入到事件队列的队首, 如果在当前正有事件被处理, 则处理结束后调用; 如果没有则立即调用

简单来讲, setImmediate是在空闲时间触发, setTimeout是在指定时间后的下一个空闲时间触发, process.nextTick是一种插队的方法, 当前处理完毕后立刻触发

回头我们再来看这道面试题, 1是同步的因此第一个输出, process.nextTick会插队所以第二个输出, setTimeout设置延时为0毫秒, 也就是立即加入队列等待触发, 所以第三个输出, setImmediate需要等待前三个的 I/O 处理完毕后才会触发, 所以是最后一个.

另外, 如果 setTimeout 的延时时间比前面输出1和2的所消耗时间长的话, 就会使得 setImmediate 在 setTimeout 之前执行, 因为 setImmediate 得到了一个空闲期. 在我的机器上, 如果将延时设置为17ms, 将会随机出现1432和1423两种结果, 因为输出1和2的时间大致耗时17ms左右, 用时少于17ms就是1423, 多于就是1432