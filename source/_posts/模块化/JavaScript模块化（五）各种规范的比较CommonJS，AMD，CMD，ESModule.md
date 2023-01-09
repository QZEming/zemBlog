---
title: JavaScript模块化（五）各种规范的比较CommonJS，AMD，CMD，ES Module
tags: JavaScript模块化
categories: JavaScript
abbrlink: 21d855a8
date: 2019-08-13 22:56:51
---
在对几种模块规范的复习后，总结一下规范的区别和关系
[CommonJS](https://blog.csdn.net/zemprogram/article/details/99419899)，[AMD](https://blog.csdn.net/zemprogram/article/details/99447144)，[CMD](https://blog.csdn.net/zemprogram/article/details/99461061)，[ES Module](https://blog.csdn.net/zemprogram/article/details/99248109)
## 偏向性
---
虽然对于这四个规范来说，都可以在浏览器端和服务器端运行，但除了ES Module外，其他都具有一定的偏向性
<!-- more -->
对CommonJS来说，因其模块加载是同步的，没有做好合适的异步处理，所以如果在浏览器环境下的话，会受到网络环境的影响，无法保证模块能及时加载好，所以**不太适合在浏览器端使用**
对AMD和CMD来说，是**偏向浏览器端**的，浏览器端要保证效率，需要采用异步加载，这就需要一个预处理，提前将所需要的模块文件并行加载好，而AMD和CMD恰好就是异步的
## CommonJS与AMD的比较
---
1. 和上面说的一样，偏向性不同，CommonJS偏向于服务器端，而AMD偏向于浏览器端
2. CommonJS是同步加载的，而AMD是异步加载的
3. CommonJS基于node.js，而AMD依赖于require.js
## AMD与CMD的比较
---
1. AMD在加载依赖模块时是提前加载，而CMD是延迟加载
```javascript
// AMD
define(['module1'], function(m1) {
   // ...
})
//CMD
define(function(require){
    // 引入同步依赖
    var module1  = require('./module1');
    // 引入异步依赖
    require.async('./module2',function(m3){
        // 回调方法
    })
    // ...
})
```
AMD在执行后面的方法前，先将模块加载好，而CMD到了需要的时候才加载对应的模块
2. AMD一般依赖前置，而CMD是依赖就近
以上面的代码为例，AMD将需要使用的依赖前置到define的第一个参数，在顶部加载好后才继续后面的方法，而CMD的依赖一般是到了要使用的时候才引入，所以是依赖就近
3. AMD依赖于require.js而CMD依赖于sea.js
## CommonJS与ES Module(ES6)的比较
---
1. CommonJS输出的是值的拷贝，而ES Module输出的是值的引用
这也就是说，对于CommonJS引入的值来说，只要这个值已经被引入了，那么源模块的该值不管发生什么变化，也不会影响到之前就引入的值，而对于ES Module来说，如果源模块的值发生变化，会连带影响当前引入的值。

2. CommonJS 模块是运行时加载，ES Module是编译时输出接口
CommonJS在js文件运行时，如果遇到require语句，就会先停下来加载对应的模块，对于ES Module，如果遇上了一个import语句，就会生成一个只读的引用，当真正执行到需要改模块的内容时，再通过改引用去获取相应的值

3. CommonJS加载无法提升，而ES Module加载会被提升到文件顶部
CommonJS的加载分为两种情况，一种是在服务端同步加载，一种是在浏览器异步加载，但是这两种加载，都无法做到提升，所以一定要先引入再使用，而ES Module可以把引入写在使用之后，因为有变量提升

4. CommonJS模块的输出是一个对象，而ES Module的输出并不是一个对象
在使用CommonJS输出时，我们输出的最终对放在module.exports这个对象上，而在ES Module中，我们是可以分别在不同地方输出