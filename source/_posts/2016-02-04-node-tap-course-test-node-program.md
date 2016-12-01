---
layout: post
author: 碎碎酱
title: Node-Tap教程：使用node-tap测试你的node.js程序
permalink: node-tap-course-test-node-program
date: 2016-02-04
thumbnail: /img/node-tap.png
category: javascript
tags:
- node.js
- node-tap
lede: "这篇教程将指导你如何使用tap测试你的node.js程序。"
---


这篇教程将指导你如何使用[tap][2]测试你的node.js程序。

## 安装tap

使用npm安装tap：
`npm install tap --save-dev`

参数`save-dev`使tap保存在你的package.json文件中`devDependencies`列表内。

接下来，更新你的package.json文件，使得`npm test`调用tap：

```
{
  "name": "node-tap-demo",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "tap test/*.js"
  },
  "author": "yinxin630",
  "license": "GPL-3.0",
  "devDependencies": {
    "tap": "^5.4.2"
  }
}
```

## 测试文件

为你的测试创建一个目录，一般为`test`，以便其他人可以猜出它的用途：
`mkdir test`

将一个很大的功能测试分割成多个文件是一种很好的做法，每个文件应该覆盖一个特性或概念，对于小的Node模块，通常一个单独的测试文件就足够了。

我通常把第一个测试文件叫做`test/basic.js`，因为它用于测试基础功能。

## "hello world"测试

tap的顶级对象是tap的Test类的一个成员，这意味着子测试有着和它一样的属性。

这是一个非常基础的测试代码：

```
// test/basic.js
var tap = require('tap');
tap.pass('this is fine');
```

使用node运行这个测试，输出如下：

```
// node test/basic.js
TAP version 13
ok 1 - this is fine
1..1
# time=47.793ms
```

你可以直接运行一个tap测试程序来看看它做了什么，这在调试失败测试用例时特别方便。

输出内容是"TAP"或者"Test Anything Protocol"，这种格式再Perl社区有着很久的历史，并且有着很多不同语言编写的不同工具，来生成、解析这种格式。

Node-Tap就是其中之一，所以，我们用它来生成更美观的输出内容。因为我们把tap安装为devDependency，并且将它作为一个脚本天骄到了package.json中，我们可以使用`npm test`来运行我们所有的测试。

```
$ npm test

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js

test/basic.js ......................................... 1/1
total ................................................. 1/1

  1 passing (503.969ms)

  ok
```

## 覆盖范围

测试覆盖率可以很容易的得出，我们的测试覆盖了多少我们任务需要测试的内容。

我们来创建一个模块用了test，假如我们有这样一个功能，当数字是偶数是返回`even`，当数字是奇数时返回`odd`，如果数字大于100返回`big`，如果数字小于0返回`negative`。

```
// src/number.js
module.exports = function (x) {
    if (x % 2 === 0) {
        return 'even'
    } else if (x % 2 === 1) {
        return 'odd'
    } else if (x > 100) {
        return 'big'
    } else if (x < 0) {
        return 'negative'
    }
}
```

或许看起来没有bug！

现在，我们创建一个测试文件，引入`number.js`，核实结果：

```
// test/basic.js
var tap = require('tap');
var number = require('../src/number.js');

tap.equal(number(1), 'odd');
tap.equal(number(2), 'even');
```

结果看起来很不错：

```
$ npm test

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js

test/basic.js ......................................... 2/2
total ................................................. 2/2

  2 passing (493.821ms)

  ok
```

让我们开启测试覆盖率运行：

```
⇒  npm test -- --cov

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js "--cov"

test/basic.js ......................................... 2/2 1s
total ................................................. 2/2

  2 passing (1s)

  ok
------------|----------|----------|----------|----------|----------------|
File        |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------|----------|----------|----------|----------|----------------|
 src/       |    55.56 |     37.5 |      100 |    55.56 |                |
  number.js |    55.56 |     37.5 |      100 |    55.56 |        6,7,8,9 |
------------|----------|----------|----------|----------|----------------|
All files   |    55.56 |     37.5 |      100 |    55.56 |                |
------------|----------|----------|----------|----------|----------------|
```

天老爷，只有50%的覆盖率，这并不是很好，我们来看看哪些行被覆盖到了：
`npm test -- --cov --coverage-report=lcov`

这个命令会在浏览器显示一个[漂亮的测试报告][3]，它显示出我们的函数有一半没有被执行。

好的，添加更多的测试：

```
// src/number.js
var tap = require('tap');
var number = require('../src/number.js');

tap.equal(number(1), 'odd');
tap.equal(number(2), 'even');
tap.equal(number(200), 'big');
tap.equal(number(-10), 'negative');
```

现在测试输出内容变得更有趣了：

```
$ npm t

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js

test/basic.js ......................................... 2/4
  not ok should be equal
    +++ found
    --- wanted
    -big
    +even
    compare: ===
    at:
      line: 6
      column: 5
      file: test/basic.js
    stack: |
      Object.<anonymous> (test/basic.js:6:5)
    source: |
      tap.equal(number(200), 'big');

  not ok should be equal
    +++ found
    --- wanted
    -negative
    +even
    compare: ===
    at:
      line: 7
      column: 5
      file: test/basic.js
    stack: |
      Object.<anonymous> (test/basic.js:7:5)
    source: |
      tap.equal(number(-10), 'negative');

total ................................................. 2/4


  2 passing (515.105ms)
  2 failing

npm ERR! Test failed.  See above for more details.
```

更新我们的代码，使得我们的测试能够通过：

```
// src/number.js
module.exports = function (x) {
    if (x > 100) {
        return 'big'
    } else if (x < 0) {
        return 'negative'
    }
    else if (x % 2 === 0) {
        return 'even'
    } else if (x % 2 === 1) {
        return 'odd'
    }
}
```

现在我们的测试覆盖率更好了：

```
$ npm test -- --cov

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js "--cov"

test/basic.js ......................................... 4/4 1s
total ................................................. 4/4

  4 passing (1s)

  ok
------------|----------|----------|----------|----------|----------------|
File        |  % Stmts | % Branch |  % Funcs |  % Lines |Uncovered Lines |
------------|----------|----------|----------|----------|----------------|
 src/       |      100 |     87.5 |      100 |      100 |                |
  number.js |      100 |     87.5 |      100 |      100 |                |
------------|----------|----------|----------|----------|----------------|
All files   |      100 |     87.5 |      100 |      100 |                |
------------|----------|----------|----------|----------|----------------|
```

## 异步函数

如果你的模块中包含一些异步的函数，你可以使用子测试来测试他们。（你也可以使用子测试来聚合一些结果断言到一个块，这样使得更容易管理测试）

使用`tap.test(...)`函数创建一个子测试，子测试看起来和主tap对象是一样的。

你可以在子测试对象完成时调用`.end()`函数。

```
// test/async.js
// this is a silly test.
var tap = require('tap')
var fs = require('fs')
tap.test('some async stuff', function (childTest) {
  fs.readdir(__dirname, function (er, files) {
    if (er) {
      throw er // tap will handle this
    }
    childTest.match(files.join(','), /\basync\.js\b/)
    childTest.end()
  })
})

tap.test('this waits until after', function (childTest) {
  // no asserts?  no problem!
  // the lack of throwing means "success"
  childTest.end()
})
```

如果你使用node运行这个测试，你将会看到子测试是锯齿状的：

```
$ node test/async.js
TAP version 13
    # Subtest: some async stuff
    ok 1 - should match pattern provided
    1..1
ok 1 - some async stuff # time=12.78ms

    # Subtest: this waits until after
    1..0
ok 2 - this waits until after # time=6.2ms

1..2
# time=45.274ms
```

如果你使用tap运行，看起来将会是其它样子

```
$ npm test

> node-tap-demo@1.0.0 test /Users/yinxin/Github/node-tap-demo
> tap test/*.js

test/async.js ......................................... 2/2
test/basic.js ......................................... 4/4
total ................................................. 6/6

  6 passing (566.133ms)

  ok
```

## 拓展内容

你还可以做这些额外的内容：
1. 将`--cov`添加到你的package.json测试脚本中，以便于每一次测试都包含[覆盖率][4]
2. 全局安装Tap来直接[运行][5]

更多内容请查看[API][6]


  [1]: http://www.suisuijiang.com/
  [2]: https://github.com/tapjs/node-tap
  [3]: http://www.node-tap.org/basics/coverage-example-1/lcov-report/root/index.html
  [4]: http://www.node-tap.org/coverage/
  [5]: http://www.node-tap.org/cli/
  [6]: http://www.node-tap.org/api/