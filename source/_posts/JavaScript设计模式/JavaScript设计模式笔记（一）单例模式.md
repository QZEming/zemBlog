---
title: JavaScript设计模式笔记（一）单例模式
abbrlink: b5191440
date: 2019-05-01 00:19:15 
tags: JavaScript设计模式
categories: JavaScript
---
# 什么是单例模式
对于面向对象的语言来说，单例模式即是**一个类仅能有一个实例，并提供一个能访问它的全局访问点**。  
我们使用JavaScript来模仿面向对象中语言的单例模式
<!-- more -->
```javascript
var singleton = function(name) {
    this.name = name;
    this.flag = null;
}
 
var create = function(name) {
    if (!singleton.flag)
        singleton.flag = new singleton(name);
    return singleton.flag;
}
 
var s1 = create("a")
var s2 = create("b")
console.log(s1 === s2) // true
```
这里的create方法只能在第一次创建实例，在后面创建实例时，会返回原来已经创建的实例，此即为单例模式。

为了符合单一职责，我们可以使用代理来把创建实例和控制其唯一性分开
```javascript
var singleton = function(name) {
    this.name = name;
}
 
var create = function(name) {
    return new singleton(name);
}
 
var createProxy = (function() {
    var instance;
    return function(name) {
        if (!instance)
            instance = create(name)
        return instance;
    }
})()
 
var s1 = createProxy("a")
var s2 = createProxy("b")
console.log(s1 === s2)
```
我们也可以使用ES6中的代理来实现
```javascript
var singleton = function(name) {
    this.name = name;
}
 
var proxy = (function() {
    var flag;
    return new Proxy(singleton, {
        construct(name) {  //  用于拦截new操作
            if (!flag)
                flag = Reflect.construct(singleton, name)  //  重新执行new操作
            return flag;
        }
    })
})()
 
var a = proxy("a")
var b = proxy("b")
console.log(a === b)
```
JavaScript实际上是一个没有类的对象（ES6中的class虽然被称为类，但实际上还是委托机制，其继承还是基于原型的），所以在JavaScript中的单例模式，即是确保只有一个实例，且能在全局中访问它。

而事实上，我们通过对象字面量创建对象时就是一个单例模式
```javascript
var a={}
```
对于a这个对象，它是独一无二的，只要把它放在全局中，就符合了我们上面关于单例模式的定义。

但是，放在全局中的变量，很有可能在后面的代码中被不小心重新赋值了，在大型项目中，把变量放在全局中显然不是我们的首要选择，我们需要尽可能避免使用全局变量，与此同时，我们又想使用单例模式，就需要进行一些处理。

## 1.使用命名空间
将要使用的单例放在命名空间中，可以减少全局变量的数量，简单的方法就是使用一个对象来存储单例，但是事实上这个存储单例的对象就是污染空间的全局变量
```javascript
var whole = {
    a: {},
    b: {}
}
```
whole作为一个全局变量为单例提供了一个访问单例的全局访问点，且这个实例是唯一的，所以a，b这两个对象/实例是单例

## 2.使用闭包封装单例
将要使用的单例使用闭包封装起来，返回一个对象提供访问闭包内的实例的方法，实际上这种方法也会对全局造成一定的污染，只是相比直接在全局中定义单例减少了对全局的污染。
```javascript
var whole = (function() {
    var __a = {}
    return {
        getA: () => __a
    }
})()
```
---
参考自曾探的《JavaScript设计模式与开发实践》