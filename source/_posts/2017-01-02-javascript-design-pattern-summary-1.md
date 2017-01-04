---
layout: post
author: 碎碎酱
title: javascript设计模式总结
permalink: javascript-design-pattern-summary-1
date: 2017-01-02
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
lede: "javascript中的设计模式学习总结, "
---

## (1) 单例模式

实例只全局存在一份, 实例是惰性创建的, 创建单例的过程与单例类的实现是分离的, 创建单例的过程可复用.

```javascript
class Util {
    constructor() {
        this.name = 'util class';
    }
}

function createSingleton(Class) {
    let instance = null;
    return function() {
        if (!instance) {
            return instance = new Class();
        }
        return instance;
    };
}

const getUtil = createSingleton(Util);
const util1 = getUtil();
const util2 = getUtil();
console.log(util1 === util2);
```

## (2) 策略模式

将变化的部分封装成单独的策略, 使用相应的策略获取结果. 还有同时应用多种策略的组合策略模式.

```javascript
const strategys = {
    'A': value => value * 2,
    'S': value => value * 4 + 1,
    'SS': value => value * 8 + 3
};

function calcBonus(level, base) {
    return strategys[level] ? strategys[level](base) : base;
}

console.log(calcBonus('A', 100));
console.log(calcBonus('SS', 80));
```

## (3) 代理模式

间接访问本体, 代理可以为其添加额外功能, 代理应该具有有本体一样的接口, 直接去掉代理不影响本体的工作. 常用代理的类型包括: 请求合并代理 / 缓存代理 / 访问控制代理 / 智能引用代理.

代理模式通常无需预先判断是否需要使用, 真正发现不方便直接访问对象时, 再编写代理也不迟.

```javascript
let ImageTool = {
    showImage: img => console.log(`显示图片: ${img}`)
};

const preImageTool = ImageTool;
ImageTool = {
    showImage: img => {
        preImageTool.showImage('占位图');
        setTimeout(() => preImageTool.showImage(img), 1000);
    }
};

ImageTool.showImage('原图');
```

## (4) 迭代器模式

提供一种不透明的逐个访问元素的方式. 迭代器类型: 非数组类型迭代器 / 倒序迭代器 / 中止迭代器

```javascript
class ItemList {
    constructor() {
        this.items = [1, 2, 3, 4, 5];
        this.a = 'a';
        this.b = 'b';
        this.c = 'c';
        this.d = 'd';
        this.e = 'e';
        this.next1 = this.iterator1.call(this);
        this.next2 = this.iterator2.call(this);
    }
    iterator1() {
        let currentIndex = 'a'.charCodeAt(0) - 1;
        return () => {
            currentIndex++;
            return this[String.fromCharCode(currentIndex)];
        };
    }
    iterator2() {
        let currentIndex = -1;
        return () => {
            currentIndex++;
            return this.items[currentIndex];
        };
    }
}

const itemList = new ItemList();
for (let item = itemList.next1(); item; item = itemList.next1()) {
    console.log(item);
}
for (let item = itemList.next2(); item; item = itemList.next2()) {
    console.log(item);
}
```

## (5) 发布-订阅模式

发布-订阅模式又叫观察者模式, 用于解耦一对多的状态依赖问题, 当对象的状态发生改变时, 所有依赖它(订阅了该事件)的对象都将得到通知

```javascript
class Eventer {
    constructor() {
        this.events = {};
    }
    register(event, handler) {
        if (Array.isArray(this.events[event])) {
            this.events[event].push(handler);
        } else {
            this.events[event] = [ handler ];
        }
    }
    emit(event) {
        if (this.events[event]) {
            for (const handler of this.events[event]) {
                handler();
            }
        }
    }
}

const eventer = new Eventer();
eventer.register('event1', () => console.log('event1的第1个回调'));
eventer.register('event1', () => console.log('event1的第2个回调'));
eventer.register('event2', () => console.log('event2的第1个回调'));

eventer.emit('event1');
eventer.emit('event2');
```

## (6) 命令模式

命令模式用于将命令发起者和命令执行者解耦, javascript中回调函数的形式是一种隐形的命令模式.

```javascript
class Diner {
    constructor() {
        this.onEat = null;
        this.onDrink = null;
    }
    eat() {
        this.onEat && this.onEat();
    }
    drink() {
        this.onDrink && this.onDrink();
    }
}
class Restaurant {
    handleEat() {
        console.log('洗菜\n炒菜\n上菜');
    }
    handleDrink() {
        console.log('开瓶盖\n上酒');
    }
}

const diner = new Diner();
const restaurant = new Restaurant();
diner.onEat = restaurant.handleEat();
diner.onDrink = restaurant.handleDrink();

diner.eat();
diner.drink();
```

## (7) 组合模式

将多个子任务组合成新的的任务, 新任务与子任务有着相同的接口, 新任务可再用于组合成其它任务

```javascript
const eat = { do: () => console.log('吃') };
const drink = { do: () => console.log('喝') };
const teeth = { do: () => console.log('剔牙') };
const lunch = { do: () => {
    eat.do();
    drink.do();
    teeth.do();
} };

function doSomething(thing) {
    thing.do();
}
doSomething(eat);
doSomething(drink);
doSomething(lunch);
```

