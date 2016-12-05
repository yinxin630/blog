---
layout: post
author: 碎碎酱
title: 谈谈JavaScript深复制那些事
permalink: javascript-deepcopy
date: 2016-05-3
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
lede: "javascirpt中的Array和Object类型是引用类型, 在赋值时只会复制引用, 因此有时需要进行深复制."
---


## JavaScript基础类型
* Number
* String
* Boolean
* Array
* Object
* Function
* Date
* Null
* Undefined
* Symbol
    
让我们一个个分析这些基础类型在深度复制时的变化.

### 1. Number

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = 1;
var copy = old;
print(old, copy, 'before');

old = 2;
print(old, copy, 'after');

// before change
// old --> 1
// copy --> 1
// after change
// old --> 2
// copy --> 1
```

Number类型不会随着源对象修改而修改, 可以直接复制

### 2. String

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = 'string';
var copy = old;
print(old, copy, 'before');

old = 'change';
print(old, copy, 'after');

// before change
// old --> string
// copy --> string
// after change
// old --> change
// copy --> string
```

String类型在源对象赋值为新字符串时, 由于js中字符串不可修改, 所以`'change'`为一个新字符串, `old`指向了新字符串地址, 而`copy`所指向字符串的地址并没有被修改, 仍然指向`'string'`, 所以String类型也可以直接复制

### 3. Boolean

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = true;
var copy = old;
print(old, copy, 'before');

old = false;
print(old, copy, 'after');

// before change
// old --> true
// copy --> true
// after change
// old --> false
// copy --> true
```

Boolean类型也和Number类型相同, 可以直接复制

### 4. Array

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = [0, 1, 2, 3];
var copy = old;
print(old, copy, 'before');

old[1] = 11;
print(old, copy, 'after');

// before change
// old --> [ 0, 1, 2, 3 ]
// copy --> [ 0, 1, 2, 3 ]
// after change
// old --> [ 0, 11, 2, 3 ]
// copy --> [ 0, 11, 2, 3 ]
```

Array类型在修改源对象时复制对象也会变化, 因为它们指向的是同一块内存区域

深复制版本:

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}
function deepcopy (array) {
    return array.slice(0);
}

var old = [0, 1, 2, 3];
var copy = deepcopy(old);
print(old, copy, 'before');

old[1] = 11;
print(old, copy, 'after');

// before change
// old --> [ 0, 1, 2, 3 ]
// copy --> [ 0, 1, 2, 3 ]
// after change
// old --> [ 0, 11, 2, 3 ]
// copy --> [ 0, 1, 2, 3 ]
```

### 5. Object

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = { a: 0, b: 1 };
var copy = old;
print(old, copy, 'before');

old.a = 100;
print(old, copy, 'after');

// before change
// old --> { a: 0, b: 1 }
// copy --> { a: 0, b: 1 }
// after change
// old --> { a: 100, b: 1 }
// copy --> { a: 100, b: 1 }
```

Object类型自然也需要深度复制

深复制版本:

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}
function deepcopy (object) {
    var copy = {};
    for (var attr in object) {
        if (object.hasOwnProperty(attr)) {
            if (Object.prototype.toString.call(object[attr]) === '[object Object]') {
                copy[attr] = deepcopy(object[attr]);
            }
            else {
                copy[attr] = object[attr];
            }
        }
    }
    return copy;
}

var old = { a: 0, b: 1, c: { d: 4 } };
var copy = deepcopy(old);
print(old, copy, 'before');

old.a = 100;
old.c.d = 400;
print(old, copy, 'after');

// before change
// old --> { a: 0, b: 1, c: { d: 4 } }
// copy --> { a: 0, b: 1, c: { d: 4 } }
// after change
// old --> { a: 100, b: 1, c: { d: 400 } }
// copy --> { a: 0, b: 1, c: { d: 4 } }
```

### 6. Function

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old);
    console.log('copy -->', copy);
}

var old = function () { return 1 };
var copy = old;
print(old, copy, 'before');

old = function () { return 100 };
print(old, copy, 'after');

// before change
// old --> function () { return 1 }
// copy --> function () { return 1 }
// after change
// old --> function () { return 100 }
// copy --> function () { return 1 }
```

Function类型与String类型类似, 为不可变量, 只能创建新Function对象, 而复制对象由于指向地址不变, 仍为原来的Function对象

### 7. Date

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old.getTime());
    console.log('copy -->', copy.getTime());
}

var old = new Date;
var copy = old;
print(old, copy, 'before');

old.setTime(Date.now());
print(old, copy, 'after');

// before change
// old --> 1462250381764
// copy --> 1462250381764
// after change
// old --> 1462250381791
// copy --> 1462250381791
```

Date类型也属于Object类型, 需要深度复制

深复制版本:

```javascript
function print (old, copy, s) {
    console.log(`${s} change`);
    console.log('old -->', old.getTime());
    console.log('copy -->', copy.getTime());
}
function deepcopy (date) {
    return new Date(date);
}

var old = new Date;
var copy = deepcopy(old);
print(old, copy, 'before');

old.setTime(Date.now());
print(old, copy, 'after');

// before change
// old --> 1462254095581
// copy --> 1462254095581
// after change
// old --> 1462254095622
// copy --> 1462254095581
```

### 8. Null/Undefined/Symbol

这些类型与基本数值类型行为一致


  [1]: http://suisuijiang.com/