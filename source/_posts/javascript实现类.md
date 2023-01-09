---
title: javascript实现类
abbrlink: 7c5bc56b
date: 2020-02-28 19:15:08
tags: JavaScript
categories: JavaScript
---
在面向对象语言诸如java中，我们经常能看到class也就是类，而在JavaScript中，我们也可以实现类
<!-- more -->
# ES6之前的实现
要通过JavaScript实现一个类，可以通过this的指向来实现
```javascript
function Person(name,age){
    this.name = name
    this.age = age
}

var p = new Person('Bob',18)
```
这样就简单地实现了一个类  
如果要实现一些方法，也一样通过this指向
```javascript
function Person(name,age){
    this.name = name
    this.age = age
    this.greet = function(){
        console.log(`Hello, I am ${this.name} and I am ${this.age} years old`)
    }
}

var p = new Person('Bob',18)
p.greet()
// Hello, I am Bob and I am 18 years old
```
因为该方法对每个实例来说都是一样的，如果每个实例的创建都要新建这样一个重复的方法，那就是对内存的浪费了，为此，我们可以将方法写在原型中，结果是一样的，等于将实例方法转换成静态方法
```javascript
function Person(name,age){
    this.name = name
    this.age = age
}
Person.prototype.greet = function(){
    console.log(`Hello, I am ${this.name} and I am ${this.age} years old`)
}
var p = new Person('Bob',18)
p.greet()
// Hello, I am Bob and I am 18 years old
```

## 私有方法和公有方法的实现
公有方法：可以在类本身，子类，类外部调用  
私有方法：只能在类本身调用，子类和类外部无法调用  

上面的greet就是一个公有方法，通过把方法绑定到this，让我们可以通过实例在外部调用，也可以通过类本身去调用

这里我们再实现一个私有方法，要实现一个私有方法，实际上就是在类里面声明一个方法，不暴露出来
```javascript
function Person(name,age){
    this.name = name
    this.age = age
    function complete(){
        console.log('The object has been constructed')
    }
    complete()
}
var p = new Person('Bob',18)
// The object has been constructed
```
这里的complete方法就是一个私有方法

# class
在ES6之后，我们有了class这个新的关键词，而class就是JavaScript中的类，虽然它只是个语法糖，实际上还是通过委托机制来实现的，继承也是基于prototype的  
```javascript
class Person{
    constructor(name,age){
        this.name = name
        this.age = age
    }
}
```
这里说说使用class怎么实现类中的各种方法
## 静态方法
```javascript
class Person{
    constructor(name,age){
        this.name = name
        this.age = age
    }
    static s(){
        console.log('static')
    }
}

let p = new Person('Bob',18)
p.s()
// Uncaught TypeError: p.s is not a function
Person.s()
// static
```

## 公有方法
```javascript
class Person{
    constructor(name,age){
        this.name = name
        this.age = age
    }
    greet(){
        console.log(`Hello, I am ${this.name} and I am ${this.age} years old`)
    }
}

let p = new Person('Bob',18)
p.greet()
// Hello, I am Bob and I am 18 years old
```

## 私有方法
ES6class并没有提供私有方法的直接实现，所以需要我们自己做一些变通处理
### 约定俗成
在使用时，我们默认把前面加上下划线_的方法看成私有方法
```javascript
class Person{
    constructor(name,age){
        this.name = name
        this.age = age
    }
    _p(){
        console.log('private')
    }
}
```
### 在构造函数中声明
但实际上，这仍然是一个公有方法，我们可以通过在constructor中声明这个私有方法，然后通过公有方法来暴露给内部使用，这样外部就无法获得这个私有方法的内容，但仍然可以通过公有方法来调用这个方法
```javascript
class Person{
    constructor(name,age){
        this.name = name
        this.age = age
        function _p(){
            console.log('private')
        }
        this.p = ()=>{
            _p()
        }
    }
}
let p = new Person('Bob',18)
p.p()
// private
```
### 利用symbol的唯一性
当然，也可以利用Symbol值的唯一性，将方法名改为Symbol值。
```javascript
const bar = Symbol('bar');
const snaf = Symbol('snaf');
export default class myClass{
    // 公有方法  
    foo(baz) {
        this[bar](baz);
    }
    // 私有方法  
    [bar](baz) {
        return this[snaf] = baz;
    }
    // ...
};
```
因为Symbol值的唯一性，所以一般方法是没法访问它们的，这样就达到了私有方法的效果。（事实上Reflect.ownKeys()仍然可以访问到）