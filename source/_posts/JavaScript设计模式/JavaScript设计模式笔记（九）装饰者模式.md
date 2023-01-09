---
title: JavaScript设计模式笔记（九）装饰者模式
abbrlink: 596930f0
date: 2019-05-13 23:00:01
tags: JavaScript设计模式
categories: JavaScript
---
# 装饰者模式的定义
装饰者模式用于为对象动态地添加一些额外的职责，而不改变对象内部原来的代码实现。
<!-- more -->
# 装饰者模式的使用
当我们要给一个对象添加功能时，可以直接重写整个对象，但是这样违反了开放-封闭原则，而且代码编写者也无法保证可以保留对象原来的所有功能。

首先可以使用一种简单的装饰方法，在写入新功能时，先使用一个变量来保存原对象/方法，在编写新功能时，先将保存的变量传入，然后再编写新功能，就不会影响原来的内容，符合开放-封闭原则
```javascript
var a = function() {
    console.log(1);
}
 
a();
// 1
 
var _a = a;
 
var a = function() {
    _a();
    console.log(2);
}
 
a();
// 1
// 2
```
这种方法虽然有效，但存在一些问题

1. 需要维护保存原方法的变量
虽然每次装饰我们只需要维护一个变量，但是一旦我们对一个方法的修改次数多了，或者我们要装饰很多个方法，那么我们所需要维护的变量可能会超出我们的预期

2. this被劫持
将原方法放在全局变量中，在调用该方法时，实际上就是在调用一个全局函数，而在浏览器中，调用一个全局函数，this会指向window，而这不一定符合我们的预期
```javascript
window.name = 'marry'
 
var person = {
    name: 'jack',
    getName: function() {
        return this.name;
    }
}
var _getName = person.getName;
 
person.getName = function() {
    console.log('insert');
    return _getName();
}
 
console.log(person.getName());
// insert
// marry
```
可以看到最后打印出来的并不是jack，而是window的name属性的值--marry，这就说明this确实被劫持了，要解决这个问题，可以在调用方法时绑定this
```javascript
var person = {
    name: 'jack',
    getName: function() {
        return this.name;
    }
}
 
person.getName()
 
var _getName = person.getName;
 
person.getName = function() {
    console.log('insert');
    return _getName.apply(person, arguments);
}
 
console.log(person.getName());
 
// insert
// jack
```
但是很显然，这种绑定的方式很麻烦，下面使用AOP来装饰函数

引用曾探的《JavaScript设计模式与开发实践》中的代码
```javascript
Function.prototype.before = function(beforefn) {
    var __self = this; // 保存对原函数的引用
    return function() { // 返回包含了原函数和新函数的“代理”函数
        beforefn.apply(this, arguments); // 执行新函数，且保证this不被劫持，新函数接受的参数
        // 也会被原封不动地传入原函数，新函数在原函数之前执行
        return __self.apply(this, arguments); // 执行原函数并返回原函数的执行结果，并且保证this不被劫持
    }
}
 
Function.prototype.after = function(afterfn) {
    var __self = this;
    return function() {
        var ret = __self.apply(this, arguments);
        afterfn.apply(this, arguments);
        return ret;
    }
}
```
beforefn和afterfn即为我们要添加的功能函数，首先使用__self将原方法保存起来，这里的this就指向原方法，然后调用方法时使用apply来防止this劫持，将该方法应用到上面的代码中。
```javascript
var person = {
    name: 'jack',
    getName: function() {
        return this.name;
    }
}
 
person.getName = person.getName.before(function() {
    console.log('insert');
})
 
console.log(person.getName());
```
# 装饰者模式和代理模式的区别
装饰者模式和代理模式在代码设计上看起来很类似，主要的区别是在于设计意图上。装饰者模式是一开始不明确功能，在之后的开发中突然有新增功能时才添加上去，而代理模式是某个对象不适合执行某些方法，代理给另一个对象执行。

---
参考自曾探的《JavaScript设计模式与开发实践》