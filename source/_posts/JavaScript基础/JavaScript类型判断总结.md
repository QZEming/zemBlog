---
title: JavaScript类型判断总结
abbrlink: 9ed3e37c
date: 2019-04-29 01:04:01
tags: JavaScript
categories: JavaScript
---
在对类型的识别进行总结前，首先明确一下，在JavaScript中，有七种内置类型，null，undefined，boolean，number，String，Object，Symbol（ES6中新增），接下来对几种常见的类型识别进行总结。
<!-- more -->
# typeof
typeof运算符是我们常用的用于检测类型的一种方式，该运算符会返回一个字符串来表示其识别到的类型，我们使用typeof对七种内置类型的类型进行检测。

```javascript
typeof 1
// "number"
typeof "str"
// "string"
typeof true
// "boolean"
typeof {}
// "object"
typeof undefined
// "undefined"
typeof Symbol()
// "symbol"
typeof null
// "object"
```
可以发现，大部分的类型检测都符合我们的，但是null却莫名奇妙地返回了一个"object"，按我们的理解它应该返回一个"null"才对。这其实是这个检测方式的一种bug，至于在后面会不会得到修改，现在还不得而知，至少我们知道在代码编写中要避免使用typeof对可能变为null的变量进行类型检测。如果非要使用typeof检测null类型，就得同时判断其转为boolean类型是否为false，我将其封装在一个方法中如下代码
```javascript
function typeCheck(n){
    if(!n&&typeof n==="object")
        return "null"
    else
        return typeof n
}
typeCheck(null)
// "null"
```
除了上面的内容，还有一个常用的“类型”，数组，当我们使用typeof检测数组时，会直接返回"object"
```javascript
typeof []
// "object"
```
我们见到的typeof返回的字符串除了上面见到的之外，还有一种特殊的返回字符串，就是对function使用typeof时会返回“function"
```javascript
function fn(){}
typeof fn
// "function"
```
然而，正如我们所知道的，function实际上是object的子类，对于function的类型判断到底为哪个，取决于在代码中是否要区分方法和我们”传统“认知中的object，如果我们要模糊"function"和”object"，大可在上面的代码加以修改。
```javascript
function typeCheck(n){
    if(!n&&typeof n==="object")
        return "null"
    else if(typeof n==="function")
        return "object"
    else
        return typeof n
}
typeCheck(typeCheck)
// "object"
```
对于使用new运算符构建的对象，typeof统一返回"object"，即我们无法更细致地区分这些对象

而我们常见的原生函数有String(),Number(),Boolean(),Array(),Object(),Function().RegExp().Date(),Error()。这些在使用typeof检测时都返回"object"。使用typeof时我们无法很好地区分这些内容。
```javascript
typeof new String()
// "object"
typeof new Number()
// "object"
typeof new Array()
// "object"
typeof new Object()
// "object"
typeof new Function()
// "function"
typeof new RegExp()
// "object"
typeof new Date()
// "object"
typeof new Error()
// "object"
```
除了上面常见的情况外，还有一个最新的基本类型bigint，typeof可以判断该类型
```javascript
typeof 1n
// "bigint"
typeof BigInt("1")
// "bigint"
```
总的来说，typeof只能返回以下八个值其中一个：number，string，boolean，object，undefined，symbol，function，还有最新的bigint

# constructor
如果我们要判断出对象是由哪些函数构建的，我们可以通过使用对象的constructor属性来判断
```javascript
new String().constructor===String
// true
new Number().constructor===Number
// true
new Array().constructor===Array
// true
new Boolean().constructor===Boolean
// true
new Object().constructor===Object
// true
new Function().constructor===Function
// true
new RegExp().constructor===RegExp
// true
new Date().constructor===Date
// true
new Error().constructor===Error
// true
```
正如我们所期待的，constructor出色地完成了我们的任务，成功辨别了对象由哪个函数构造的，那么constructor对于基本的内置类型又是否能完美判断呢
```javascript
// 1.constructor===Number(错误写法)
// 数字字面量没有constructor
var num=1;
num.constructor===Number
// true
BigInt('1').constructor === BigInt
// true
"str".constructor===String
// true
true.constructor===Boolean
// true
Symbol().constructor===Symbol
// true
// {}.constructor===Object(错误写法)
// 对于一个直接构建的对象来说没有constructor属性
// 同样地null和undefined也没有constructor属性
```
可以看到，对于基本的内置属性，constructor只能判断其中的四种，其实，还有第五种，就是我们上面提及的BigInt
```javascript
var num = 1n
num.constructor==BigInt
// true
```
除此之外，constructor还有一个需要注意的点，就是在使用继承的时候，它可能不会出现我们期待的结果
```javascript
function People(){}
function Student(){}
Student.prototype=new People()
new Student().constructor===Student
// false
new Student().constructor===People
// true
```
可以看到Student方法构建的对象的constructor是People，这是因为在调用new Student().constructor时，我们要找到constructor属性，明显的是这个对象没有constructor属性，所以我们沿着原型链寻找该属性，而Student.prototype是new People()，而new People()中也没有constructor属性，沿着原型链我们在new People().prototype找到了该属性其值为People，所以最后出现了我们期待以外的结果。

# Object.prototype.toString
首先我们还是来看看Object.prototype.toString对八种内置类型的检测，Object.prototype.toString返回的是"[Object (class)]"，class的部分就是我们要的内容。
```javascript
Object.prototype.toString.call(1)
// "[object Number]"
Object.prototype.toString.call("str")
// "[object String]"
Object.prototype.toString.call({})
// "[object Object]"
Object.prototype.toString.call(null)
// "[object Null]"
Object.prototype.toString.call(undefined)
// "[object Undefined]"
Object.prototype.toString.call(Symbol())
// "[object Symbol]"
Object.prototype.toString.call(true)
// "[object Boolean]"
Object.prototype.toString.call(1n)
// "[object BigInt]"
```
可以看到，Object.prototype.toString完美地回应了我们的期待，那来看看对于方法会返回什么
```javascript
function fn(){}
Object.prototype.toString.call(fn)
// "[object Function]"
```
可以看到，Object.prototype.toString和typeof一样，即使function是Object的子类，还是把它判断成了”Function“。

接着来看看对于原生函数的构造时会有什么检测结果
```javascript
Object.prototype.toString.call(new String())
// "[object String]"
Object.prototype.toString.call(new Number())
// "[object Number]"
Object.prototype.toString.call(new Boolean())
// "[object Boolean]"
Object.prototype.toString.call(new Array())
// "[object Array]"
Object.prototype.toString.call(new Object())
// "[object Object]"
Object.prototype.toString.call(new Function())
// "[object Function]"
Object.prototype.toString.call(new RegExp())
// "[object RegExp]"
Object.prototype.toString.call(new Date())
// "[object Date]"
Object.prototype.toString.call(new Error())
// "[object Error]"
```
可以看到如果我们想分辨出原生函数构造对象是由什么原生函数构造的，Object.prototype.toString是一个好的选择，而对于自定义的对象，Object.prototype就无法正确判断了
```javascript
function Person(){}
var person = new Person()
Object.prototype.toString.call(person)
// "[object Object]"
```
### instanceof
instanceof在对继承对象的判断上正确
```javascript
function People(){}
function Student(){}
new Student() instanceof Student
// true
new Student() instanceof People
// true
```
但是在对原始类型的判断上不尽人意
```javascript
1 instanceof Number
// false
true instanceof Boolean
// false
"str" instanceof String
// false
```
然而，如果我们使用new来创建相应的对象，那instanceof可以正确判别
```javascript
new Number(1) instanceof Number
// true
new Boolean(true) instanceof Boolean
// true
new String('str') instanceof String
// true
```
对于原生类型来说，instanceof能正确判别
```javascript
[1,2,3] instanceof Array
// true
/abc/ instanceof RegExp
// true
({}) instanceof Object
// true
function fn(){}
fn instanceof Function
// true
```
但是BigInt却无法正确判别
```javascript
1n instanceof BigInt // false
BigInt(1) instanceof BigInt // false
```
# 不同用法的总结
## typeof
1. 对其他内置类型的检测正确，在对null进行类型检测时会返回"object"

2. 对方法进行检测时，会返回”function"

3. typeof对于对象统一返回"object"，即使使用了不同的原生函数，typeof无法区分这些原生函数构造的对象

## constructor
1. 对于内置类型只能判断Number（需要将值赋给变量才能判断），Boolean，Object，Symbol，其他三种无法判断

2. 对于对象能返回构造对象的构造函数名

3. 当使用继承时会返回原型链末端的构造函数名

## Object.prototype.toString
1. 对于所有内置对象都可以判断

2. 对于对象能返回构造对象的构造函数名，但自定义对象只能返回object

## instanceof
1. 对于对象能返回构造对象的构造函数名

2. 能正确判断继承的对象

3. 对于原始类型字面量无法正确判断（Number，Boolean，String），对new构造的对象可以正确判断，可以正确判断原生类型（RegExp，Array，Function，Object）

# 对ES6中的类型的判断
上面的内容基本都是对ES6之前的类型的判断，接下来看看对于ES6中的新类型上面的四种方法能否正确判断

## Map和Set
```javascript
// typeof
typeof new Set()
// "object"
typeof new Map()
// "object"
 
// Object.prototype.toString
Object.prototype.toString.call(new Map())
// "[object Map]"
Object.prototype.toString.call(new Set())
// "[object Set]"
 
// constructor
new Map().constructor===Map
// true
new Set().constructor===Set
// true
 
// instanceof
new Map() instanceof Map
// true
new Set() instanceof Set
// true
```
可以看到，对于类数组Map和Set，只有typeof没有返回我们所期待的类型

## Promise
```javascript
var p=new Promise((resolve,reject)=>{})
typeof p
// "object"
Object.prototype.toString.call(p)
// "[object Promise]"
p.constructor===Promise
// true
p instanceof Promise
// true
```
可以看到同样是typeof没有返回我们期待的类型

## Proxy
```javascript
var p=new Proxy({},{})
typeof p
// "object"
Object.prototype.toString.call(p)
// "[object Object]"
p.constructor===Proxy
// false
p instanceof Proxy
// Uncaught TypeError: Function has non-object prototype 'undefined' in instanceof check
```
可以看到没有一个返回我们期待的结果，instanceof甚至报错了，如果我们要明确知道某个变量实际上是代理的话，仅靠上面的方法是行不通的

以上就是我对JavaScript中类型检测的总结，若有什么补充之处和错漏之处请各位指出