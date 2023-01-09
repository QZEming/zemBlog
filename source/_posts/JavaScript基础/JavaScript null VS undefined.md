---
title: JavaScript null VS undefined
abbrlink: ff9ee460
date: 2019-04-29 13:10:21
tags: JavaScript
categories: JavaScript
---
对于null和undefined，普遍的我们知道这两者都是假值，在使用判断语句或者转为boolean类型时都会变为false，而实际上两者是有着明显区别的。
<!-- more -->
(JavaScript中的假值有0，false，undefined，null，""，NaN）

## 1. 类型上的差异
在JavaScript中有七种数据类型，null，undefined，Number，String，Boolean，Object，Symbol。而null和undefined明显分属两种不同的类型，我们可以使用类型检测来对这两者的类型进行判断以区分。（类型检测可见[JavaScript类型识别总结](https://blog.csdn.net/zemprogram/article/details/89646636)）

## 2. 转为数值的差异
在JavaScript中，undefined在转为数值时变为NaN，而null会变为0。
```javascript
new Number(undefined)
// Number {NaN}
new Number(null)
// Number {0}
```
## 3. 两者出现的情况
从两者的理解来看，null更像是表明某个地方没有值，而undefined表面该处还没有赋值

null出现的情况

### 1. 寻找一个不存在的元素

这个在我们寻找DOM节点时会看见，假如我们找的DOM节点是不存在的，就会返回Null
```javascript
// 假设当前页面没有id为1的元素
document.getElementById("1")
// null
```
### 2. 在原型链的末端

一个原型链上的对象是继承关系，但在最末端的位置是没有对象的，此时原型指向的就是null
```javascript
var obj={}
Object.getPrototypeOf(obj.__proto__)
// null
```
undefined出现的情况

### 1. 一个值声明后没有赋值
```javascript
var u
console.log(u)
// undefined
```
### 2. 没有返回值的方法
```javascript
function fn(){}
console.log(fn())
// undefined
```
### 3. 访问对象没有的属性
```javascript
var obj={}
console.log(obj.a)
// undefined
```
### 4. 函数中的形参没有传入
```javascript
function fn(a){
    console.log(a)
}
fn()
// undefined
```
## undefined的"bug"
直到现在的JavaScript中，undefined仍不作为一个关键字使用，更像是作为一个在全局中定义好的全局对象的属性，且这个属性只能读不能改（修改的时候会静默失败），在比较老的流浪器中，甚至可以修改全局中undefined的值。
```javascript
var undefined=2
console.log(undefined)
// undefined
```
上面代码中，虽然最后输出的还是undefined，但是在对undefined声明时居然没有报错，显然它不是一个关键字。而如果我们使用let和const来声明undefined
```javascript
let undefined=2
// Uncaught SyntaxError: Identifier 'undefined' has already been declared
const undefined=2
// Uncaught SyntaxError: Identifier 'undefined' has already been declared
```
可以看到报错了，但是报错原因是改变量已经被定义了，显然JavaScript将其当成一个变量了。

而如果在函数作用域中，undefined的值可以被修改
```javascript
(function(){
    var undefined=10;
    console.log(undefined);
})()
// 10
(function(){
    let undefined=10;
    console.log(undefined);
})()
// 10
 
(function(){
    const undefined=10;
    console.log(undefined);
})()
// 10
```
以上是对JavaScript中null和undefined的一些理解，后续会继续补充