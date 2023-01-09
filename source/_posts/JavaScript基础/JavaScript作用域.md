---
title: JavaScript作用域
abbrlink: 2bb77783
date: 2019-05-28 18:25:38
tags: JavaScript
categories: JavaScript
---
# 作用域的理解
我理解中的作用域，就是为引擎提供查找变量和函数的方法，通过在当前作用域及其子作用域中，通过标识符查找所需要的变量和函数，来实现代码的运行。我们可以将作用域比作一个对象，当我们需要使用某个变量/函数时，可以通过属性名（标识符）来查找，当然对于作用域“对象”来说，这个工作只能由JavaScript引擎去做，我们无法访问这个对象。

# JavaScript中的作用域类型
## 全局作用域
全局作用域管理着全局下声明的变量和函数，在全局下使用var声明的变量和函数都会被放到全局作用域中，而不使用var，let，const声明的变量会被隐式地放入全局作用域中，全局作用域下的变量和函数可以在全局的任何位置访问到。（这里不考虑let和const没有变量提升）
```javascript
var a=1;
function b(){
    c=2;
}
```
这里的a，b，c都是在全局作用域下的变量/函数。

## 函数作用域
在函数体内部使用var声明的变量和函数都会被放到该函数的作用域中，只有该作用域及其子作用域可以访问到这个作用域中的变量和函数，全局下不能直接访问函数作用域中的变量和函数。
```javascript
function fn(){
    var fnA=1;
    function fnB(){
        console.log(1);
    }
}

console.log(fnA); // Uncaught ReferenceError: fnA is not defined

fnB(); // Uncaught ReferenceError: fnB is not defined
```
这里的fnA和fnB只能在fn内部使用/调用，在全局作用域下调用会报错。

### 匿名函数和具名函数

在JavaScript内部，函数声明是不可以省略函数名的，但是函数表达式可以省略函数名，对于函数表达式较为常见的就是回调函数了，经常使用的就有定时器setTimeout和setInterval，在ES6中的promise中then和catch也用到了函数表达式。
```javascript
setTimeout(function(){
    console.log('...');
},1000);
```
函数表达式使用起来较为简单便捷，但也存在几个问题

1.匿名函数在栈追踪中不会显示出有意义的函数名，会影响到调试

2.使用匿名函数时如果要使用递归，将没有函数名可以用来递归，只能使用过时的arguments.callee来调用自身

3.匿名函数省略了函数名，使得代码阅读起来可能会有部分函数不知道是什么作用

对于上面的问题，实际上都是由于函数表达式没有名字，我们只需给这个函数表达式加上名字即可
```javascript
setTimeout(function timeoutHandle(){
    console.log('...');
},1000);
```
## 块级作用域
在JavaScript中，本来是没有块级作用域的，这一点与其他语言有所不同，在其他类似于java，C的语言当作，{}内部被当作一个块级作用域，在该块级作用域外部无法调用该作用域内部的内容，而JavaScript缺少了这一特性。
```javascript
if(true){
    var a=0;
}
console.log(a); // 0

for(var i=0;i<10;i++){}

console.log(i); // 10
```
而在ES6中，let和const给JavaScript带来了块级作用域（let和const 块级作用域）,如果在声明时使用了let或者const的话，就会将该变量/函数放进当前作用域的子作用域，而这个子作用域就是块级作用域，通过这样就可以避免该作用域外部调用到该作用域内的变量/函数。
```javascript
if(true){
    let a=0;
}
console.log(a); // Uncaught ReferenceError: a is not defined

for(let i=0;i<10;i++){}
console.log(i); // i is not defined
```
### catch的块级作用域

除了let，const的块级作用域外，在ES6之前也有类似于块级作用域的作用域出现，在ES3中的规范指出try-catch的catch分句会创建一个块级作用域，其中声明的变量仅在catch内部有效。
```javascript
try{
    undefined();
}catch(err){
    console.log(err);  //  TypeError: undefined is not a function
}
console.log(err); // undefined
```
### with的块级作用域

with语句后的{}内可以直接调用一个对象中的属性，而在with语句外部无法直接调用这些属性，必须通过对象来调用这些属性，所以with的{}内也算是一个块级作用域
```javascript
var obj={
    a:1,
    b:2,
    c:3
}

with(obj){
    console.log(a); // 1
    console.log(b); // 2
    console.log(c); // 3
}

console.log(a); // Uncaught ReferenceError: a is not defined
```
### 严格模式下的eval作用域
eval是在传入一段字符串，在运行时字符串内的内容就被当作这段代码写在了eval所在的位置，在非严格模式下，可以在外部直接访问eval内声明的变量/函数。
```javascript
function fn(str){
    eval(str);
    console.log(a);
}
fn('var a=0'); // 0
```
但是在严格模式下，eval内会有自己的作用域，在外部无法访问eval内的内容。
```javascript
function fn(str){
    'use strict'
    eval(str);
    console.log(a);
}
fn('var a=0'); // Uncaught ReferenceError: a is not defined
```
这里由于是在严格模式下，虽然console.log(a)语句和eval同在一个函数作用域下，但是console语句还是无法访问到eval内声明的变量a，因为这个eval语句内的作用域算是fn函数作用域的子作用域，在fn作用域中无法直接访问变量a，所以最后报错了。

## 作用域链
作用域链和原型链类似，当我们使用对象的某一个属性时，会遍历原型链，从当前对象的属性开始，如果找不到该属性，则会在原型链上往下一个位置移动，寻找对应的属性，知道找到属性或者到了原型链末端。在作用域链中，我们首先会在当前作用域中寻找我们需要的变量或者函数，如果没找到的话，则往上一个作用域寻找，直到找到变量/函数或者到达全局作用域。
```javascript
var a=1;
function fn(){
    if(true){
        console.log(a);
    }
}
fn() // 1
```
在这段代码中，console.log(a)在执行时，会寻找变量a的真正内容，此时在它所在的块级作用域中(if语句)找不到所要的变量a，则继续往上找，在函数作用域fn中也找不到变量a，则去到全局作用域中，此时找到了a，在作用域链上的工作终于结束了。

### 修改作用域链
一般来说，作用域由写代码期间函数所声明的位置和块的位置来决定，但是有些方法可以在运行时更改作用域链

#### eval

正如上面提及的，eval会将特定代码插入到eval所在的位置，这么做有时候会修改作用域链
```javascript
var a=1;
function foo(str){
    eval(str);
    console.log(a);
}

foo('var a=2');
```
在这段代码中，a本来是在全局作用域中的，但是因为传入了a声明的字符串语句，a所在的作用域变成了在foo函数作用域中了，显然作用域被修改了。

但是在严格模式下，如上面所说，eval内会形成一个新的作用域，虽然这样不会将原本作用域内的内容修改，但是作用域被延长是不可避免的。

#### with

with语句可以方便访问对象内的属性，但如果在使用with语句时修改了一个对象不存在的属性，则该属性会提升到全局作用域中，由此对作用域造成改变
```javascript
var obj={
    a:1
}
with(obj){
    a=2;
    b=3;
}
console.log(obj.a); // 2
console.log(obj.b); // undefined
console.log(b); // 3
```
这里可以看到，在with语句内设置属性b，但是obj对象并没有属性b，此时就变成了声明变量b，这个时候又没有使用var，let，const，所以变量会直接放在全局中而作为对象obj的属性，所以通过obj无法访问到b，而在全局中可以访问变量b。

## 标识符解析的性能
对于标识符解析来说，作用域链越深，在解析时所消耗的性能就越多。因此，如果我们要访问的变量/函数在当前作用域中，消耗的性能就是最少的，而查找全局作用域的变量/函数的性能消耗是最多的。

减少标识符解析的性能消耗

标识符解析是由引擎来做的，我们无法改变其内部实现，只能尽可能地将我们要重复用到的变量/函数往作用域链末端移动，即减少我们在作用域链上的查找时间，减少我们遍历的作用域个数。我们可以通过在末端的作用域使用一个变量来储存前面作用域的变量/函数，这样只有在给要存储的变量赋值的时候会遍历作用域链到我们需要的那个变量所在的作用域，而后面使用时只要遍历当前作用域即可。
```javascript
var a=1;

function fn(){
    var b;
    b=a+2;
    b=a+3;
}


function fn1(){
    var c=a;
    var b;
    b=c+2;
    b=c+3;
} 
```
在fn函数中，每次要用到a的时候，都要先遍历当前函数作用域，发现没有a，去到上一级作用域全局作用域寻找a，如果a使用的次数较多的话，每次都要去到全局作用域才能找到a，显然浪费了一些性能消耗。而在fn1中，只有在给c赋值时去到全局作用域寻找a，后面使用c来代替a，每次需要使用a的值时只要寻找c即可，而c在当前作用域中，只要遍历当前函数作用域即可得到值。

上面的例子中作用域的嵌套比较少，如果在嵌套较深的结构中的话，这样无疑能节约很多性能的消耗。

参考自《JavaScript高级程序设计》，《高性能JavaScript》，《你不知道的JavaScript 上》

