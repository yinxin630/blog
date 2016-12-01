---
layout: post
author: 碎碎酱
title: JavaScript中各种创建对象方法的优缺点
permalink: javascript-create-object-method-advantages-and-disadvantages
date: 2016-07-27
thumbnail: /img/javascript-logo.png
category: javascript
tags:
- javascript
lede: "javascript原生并没有在语法提供继承的能力, 但是是基于原型链实现我们可以模拟继承."
---


0. 工厂模式
    ```
    function createPerson(name, age) {
        var person = new Object();
        person.name = name;
        person.age = age;
        person.sayName = function () {
            console.log(this.name);
        };
        return person;
    }
    var person = createPerson('a', 11);
    person.sayName();
    ```
    工厂模式为创建大量同类对象提供了方便, 但是有着对象识别的问题, 即怎么知道一个对象的类型.

1. 构造函数模式
    ```
    function Person(name, age) {
        this.name = name;
        this.age = age;
        this.sayName = function() {
            console.log(this.name);
        };
    }
    var person = new Person('a', 11);
    person.sayName();
    ```
    构造函数模式创建的对象共用相同的构造函数(constructor), 可以使用instanceof操作判断类型. 需要通过new操作符创建对象, 否则this会指向全局作用域. 构造函数中定义的对象方法是不同的Function实例, 即`person1.sayName !== person2.sayName`. 可以将sayName定义在全局, 然后在构造函数中使用 `this.sayName = sayName`, 这样子每个对象实例会共享方法, 但是同时在全局作用域留下了额外的函数.

2. 原型模式
    ```
    function Person() { }
    Person.prototype.name = 'a';
    Person.prototype.age = 11;
    Person.prototype.sayName = function () {
        console.log(this.name);
    };
    var person = new Person('a', 11);
    person.sayName();
    ```
    原型模式中定义属性会被所有实例共享, 修改`person1.name`不会影响`person2.name`的值, 但是修改`Person.prototype.name`会影响所有Person对象实例.

3. 简化的原型模式
    ```
    function Person() { }
    Person.prototype = {
        name: 'a',
        age: 11,
        sayName: function () {
            console.log(this.name);
        }
    };
    var person = new Person('a', 11);
    person.sayName();
    ```
    简化的原型模式可以省去每次都要敲一遍的Person.prototype, 但是constructor不再指向Person的constructor了. 可以添加额外属性`constructor: Person.constructor`使其构造constructor仍指向Person的constructor. 但是此时的constructor是可枚举的, 而原来的constructor并不可枚举.

4. 组合使用构造函数模式和原型模式
    ```
    function Person(name, age) {
        this.name = name;
        this.age = age;
        this.friends = ['ezreal', 'ashe'];
    }
    Person.prototype = {
        constructor: Person.constructor,
        sayName: function() {
            console.log(this.name);
        }
    };
    var person = new Person('a', 11);
    person.sayName();
    ```
    这种模式中, 实例属性在构造函数中定义, 共享属性则是在原型中定义, 修改`person1.friends`并不会影响`person2.friends`. 本模式是最广泛, 认可度最高的一种创建自定义类的方法.