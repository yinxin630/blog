---
layout: post
author: 碎碎酱
title: 使用ES6的Generator函数改写异步回调金字塔
permalink: use-es6-generator-rewrite-callback
date: 2016-01-12
thumbnail: /img/javascript-es6.png
category: javascript
tags: 
- javascript
- es6
lede: "由于JavaScript是一门异步语言，Node.js中的大量API都是异步函数，异步函数无法得知确切的执行完毕时间，所以通常采用传入callback的方式来使得当异步函数执行完毕时自动调用回调函数。"
---


## 何为恶魔金字塔

由于JavaScript是一门异步语言，Node.js中的大量API都是异步函数，异步函数无法得知确切的执行完毕时间，所以通常采用传入callback的方式来使得当异步函数执行完毕时自动调用回调函数。

假如我们有如下的数据库操作API：

```javascript
// 模拟数据库数据
const datas = [
    {
        name: 'a',
        age: 1,
    }, {
        name: 'b',
        age: 2,
    }, {
        name: 'c',
        age: 1,
    }, {
        name: 'a',
        age: 3,
    }
];

// 模拟数据库API  
const DB = {
    /**
     * 根据参数条件查找符合的记录，并返回
     * @obj 查询条件
     * @callback 回调函数(results)
     */
    find: function (obj, callback) {
        var findResults = datas.map(function (data) {
            if ((obj.name === undefined || obj.name === data.name) && (obj.age === undefined || obj.age === data.age)) {
                return data;
            }
        }).filter(x => x !== undefined);
        callback(undefined, findResults);
    },
    
    /**
     * 插入数据，返回被插入的数据
     * @obj 要插入的数据
     * @callback 回调函数(result)
     */
    insert: function (obj, callback) {
        datas.push(obj);
        callback(undefined, obj);
    }
};
```

我们来完成这样一个业务流程，从数据库中通过条件查询符合条件的数据，修改查询出的数据，将修改后的新数据插入数据库。

异步回调代码如下：

```javascript
(function () {
    DB.find({age: 1}, function(err, findResults) {
        if (err !== undefined || findResults.length === 0) {
            console.log('no match results.');
            return;
        }
        console.log('find datas success.', findResults);
        var changedData = findResults.map(x => {x.age = 10; return x;});
        
        DB.insert(changedData, function(err, insertResults) {
            if (err !== undefined || insertResults === undefined || insertResults.length === 0) {
                console.log('insert datas faild.');
                return;
            }
            console.log('insert datas success.', insertResults);
        });
    });
}())
```

执行结果：

```javascript
find datas success. [ { name: 'a', age: 1 }, { name: 'c', age: 1 } ]
insert datas success. [ { name: 'a', age: 10 }, { name: 'c', age: 10 } ]
```

这里我们编写了一个自执行函数使用回调嵌套的方式来实现上述的业务逻辑，可以看到上述代码已经正确的实现了我们的业务流程。

但是，我们可以注意到，上述代码存在着如下形式的函数回调嵌套，当业务流程变得复杂的时候，回调嵌套的深度也会不断增加，于是就形成了恶魔金字塔。

```javascript
function(
    {},
    function(
        {},
        function() {
            {},
            function...
        }
    ) {
        
    }
);
```

## 如何避免恶魔金字塔

在早些时候，就出现了[BlueBrid][2]这类的Promise实现或者[Async][3]这种异步库，它们都可以有效的改写这种异步回调，但是随着ES6的出现，让我们有了更佳的选择。

### Promise方式

让我们先来看看如何使用Promise来解决这个问题，因为ES6标准定义了Promise，所以我们这里使用ES6原生的Promise实现。

```javascript
// 将异步API包装成Promise
const promiseFind = obj => {
    return new Promise((resolve, reject) => {
        DB.find(obj, function(err, result) {
            if (err) {
                reject(err);
                return;
            }
            resolve(result);
        });
    });
}
const promiseInsert = obj => {
    return new Promise(resolve => {
        DB.insert(obj, function(err, result) {
            if (err) {
                reject(err);
                return;
            }
            resolve(result);
        });
    })
}

promiseFind(
    {age: 1}
).then(result => {
    if (result.length === 0) {
        console.log('no match results.');
        return;
    }
    console.log('find datas success.', result);
    var changedData = result.map(x => {x.age = 10; return x;});
    return changedData;
}).then(
    promiseInsert
).then(result => {
   if (result === undefined || result.length === 0) {
        console.log('insert datas faild.');
        return;
    }
    console.log('insert datas success.', result);
});
```

我们可以看到，已经不再需要不断的嵌套callback，而且变成了`Function().then().then()`的形式。这样，随着业务流程变得复杂，我们只需要多添加一些`.then()`就可以了，代码逻辑清晰可见，而且可拓展性大大提升。

你应该注意到了，我们添加了额外的将普通异步函数转换为Promise的代码，你可能会觉得这样子过于麻烦，幸好，早已有开源库帮我们完成了这个工作，它就是[es6-promisify][4]。

安装es6-promisify：`npm install --save es6-promisify`

使用es6-promisify：

```javascript
const Promisify = require('es6-promisify');

Promisify(DB.find)(
    {age: 1}
).then(result => {
    if (result.length === 0) {
        console.log('no match results.');
        return;
    }
    console.log('find datas success.', result);
    var changedData = result.map(x => {x.age = 10; return x;});
    return changedData;
}).then(
    Promisify(DB.insert)
).then(result => {
   if (result === undefined || result.length === 0) {
        console.log('insert datas faild.');
        return;
    }
    console.log('insert datas success.', result);
});
```

这样是不是很容易就使用Promise改写异步回调了呢？

### 什么是Generator

Generator函数是ES6提供的一种异步编程解决方案，语法行为与传统函数完全不同。

Generator函数有多种理解角度。从语法上，首先可以把它理解成，Generator函数是一个状态机，封装了多个内部状态。

执行Generator函数会返回一个遍历器对象，也就是说，Generator函数除了状态机，还是一个遍历器对象生成函数。返回的遍历器对象，可以依次遍历Generator函数内部的每一个状态。

形式上，Generator函数是一个普通函数，但是有两个特征。一是，function命令与函数名之间有一个星号；二是，函数体内部使用yield语句，定义不同的内部状态（yield语句在英语里的意思就是“产出”）。

简单Generator函数：

```javascript
function* rangeOneToThree() {
    yield 1;
    yield 2;
    return 3;
}

var work = rangeOneToThree();
console.log(work.next());
console.log(work.next());
console.log(work.next());
```

输出结果：

```javascript
{ value: 1, done: false }
{ value: 2, done: false }
{ value: 3, done: true }
```

Generator函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。不同的是，调用Generator函数后，该函数并不执行，返回的也不是函数运行结果，而是一个指向内部状态的指针对象，也就是遍历器对象（Iterator Object）。

下一步，必须调用遍历器对象的next方法，使得指针移向下一个状态。也就是说，每次调用next方法，内部指针就从函数头部或上一次停下来的地方开始执行，直到遇到下一个yield语句（或return语句）为止。换言之，Generator函数是分段执行的，yield语句是暂停执行的标记，而next方法可以恢复执行。返回值`done = true`时，函数到达最终状态。

### Generator结合Promise

代码如下：

```javascript
const Promisify = require('es6-promisify');

const work = function* () {
    var findResults = yield Promisify(DB.find)({age: 1});
    if (findResults.length == 0) {
        console.log('no match datas.');
        return;
    }
    console.log('find datas success.', findResults);
    
    var changedData = findResults.map(x => {x.age = 10; return x;});
    
    var insertResults = yield Promisify(DB.insert)(changedData);
    if (insertResults.length == 0) {
        console.log('insert datas faild.');
        return;
    }
    console.log('insert datas success.', insertResults);
    return;
}();
```

是不是有种类似C/C++或者JAVA这些同步语言的感觉？我们可以使用类似`var result = yield call()`来调用异步函数获取结果，这不仅仅消除了异步嵌套金字塔，而且也使得业务流程更易读。不过，上面的Generator函数还不可以像同步语言那样简单的通过`work()`调用。

下面是这个Generator的执行代码：

```javascript
var nextResult0 = work.next().value;
nextResult0.then(result => {
    var nextResult1 = work.next(result).value;
    nextResult1.then(result => {
        var endResult = work.next(result);
        console.log('end.', endResult);
    });
});
```

你是不是感觉Generator还需要编写执行代码而觉得无用？我们可以编写一个通用的Generator执行方法，本文不会讲解如何编写这个方法，得益于开源社区的力量，已经有了我们所需要的功能，它就是[CO][5]。（感兴趣的可以去查看CO代码，查看下它如何实现通用的Generator执行的）

安装CO：`npm install --save co`

### 使用CO执行Generator

代码如下：

```javascript
const Promisify = require('es6-promisify');
const CO = require('co');

CO(function* () {
    var findResults = yield Promisify(DB.find)({age: 1});
    if (findResults.length == 0) {
        console.log('no match datas.');
        return;
    }
    console.log('find datas success.', findResults);
    
    var changedData = findResults.map(x => {x.age = 10; return x;});
    
    var insertResults = yield Promisify(DB.insert)(changedData);
    if (insertResults.length == 0) {
        console.log('insert datas faild.');
        return;
    }
    console.log('insert datas success.', insertResults);
    return;
});
```

是不是看起来更舒服了？CO还可以并行执行异步调用，代码如下：

```javascript
CO(function* () {
    var result = yield [
        asyncWork0,
        asyncWork1,
    ];
    console.log(result);
});
```

小插曲，原来我一直以为async.js的`parallel`函数是并行执行的，昨天才发现原来这货是逐个执行的。。。

## 总结

完整的代码如下：

```javascript
// 模拟数据库数据
const datas = [
    {
        name: 'a',
        age: 1,
    }, {
        name: 'b',
        age: 2,
    }, {
        name: 'c',
        age: 1,
    }, {
        name: 'a',
        age: 3,
    }
];

// 模拟数据库API  
const DB = {
    /**
     * 根据参数条件查找符合的记录，并返回
     * @obj 查询条件
     * @callback 回调函数(results)
     */
    find: function (obj, callback) {
        var findResults = datas.map(function (data) {
            if ((obj.name === undefined || obj.name === data.name) && (obj.age === undefined || obj.age === data.age)) {
                return data;
            }
        }).filter(x => x !== undefined);
        callback(undefined, findResults);
    },
    
    /**
     * 插入数据，返回被插入的数据
     * @obj 要插入的数据
     * @callback 回调函数(result)
     */
    insert: function (obj, callback) {
        datas.push(obj);
        callback(undefined, obj);
    }
};

const Promisify = require('es6-promisify');
const CO = require('co');

CO(function* () {
    var findResults = yield Promisify(DB.find)({age: 1});
    if (findResults.length == 0) {
        console.log('no match datas.');
        return;
    }
    console.log('find datas success.', findResults);
    
    var changedData = findResults.map(x => {x.age = 10; return x;});
    
    var insertResults = yield Promisify(DB.insert)(changedData);
    if (insertResults.length == 0) {
        console.log('insert datas faild.');
        return;
    }
    console.log('insert datas success.', insertResults);
    return;
});
```

我们已经成功的使用generator替代了回调嵌套方法。总结一下，使用generator替代回调函数要包含以下几个步骤：

1. 将异步调用转换为Promise对象。
2. 使用yield执行封装后的Promise对象，接受Promise对象返回值。
3. 使用步骤2组织业务流程
4. 使用CO或者自己编写方法，自动的执行Generator

如果你有任何看法，欢迎在下方留下评论。

  [1]: http://www.suisuijiang.com
  [2]: http://bluebirdjs.com/docs/getting-started.html
  [3]: https://github.com/caolan/async
  [4]: https://github.com/digitaldesignlabs/es6-promisify
  [5]: https://github.com/tj/co