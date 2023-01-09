---
title: JavaScript性能优化——循环优化
abbrlink: d72a741f
date: 2019-05-18 16:22:24
tags: 性能优化
categories: JavaScript
---
循环是我们在写代码时经常用到的一种结构，而往往在考虑性能优化时，循环的优化能带来很大的收益，特别是当我们不得不循环多次时，如果没对性能进行优化，那毫无疑问会带来性能的负担。
<!-- more -->
# 循环的类型
## 1. for循环
for循环可能是我们最常用的循环，相对于其他种循环来说，for循环将初始化，判断终止条件，判断值的改变明显地写在了括号内，更为直观。
```javascript
for(var i=0;i<sum;i++){
    //...
}
```
## 2. while循环
```javascript
var i=0;
while(i<sum){
    // ...
    i++;
}
```
## 3. do...while循环
```javascript
var i=0;
do{
    // ...
    i++;
}while(i<sum)
```
## 4. for-in
for-in循环一般用于遍历对象中的成员，一般不会使用for-in，因为for-in循环会对性能有更多消耗，会在下文提及原因
```javascript
for(var prop in object){
    // ...
}
```
## 5. for-of
for-of循环也可用于遍历对象中的成员，与for-in不同的是，for-of可以使用break和return语句来跳出循环。然而，虽然其性能消耗比for-in少，但是其性能消耗相对于前三种来说还是更多的，将在下文提及原因
```javascript
for(var prop of object){
    // ...
}
```
# 循环类型的性能
在对上面的几种循环遍历数组进行测试后，可以发现for-in循环和for-of循环的速度明显慢于其它三种，对于for-in来说，需要同时遍历实例和原型链，在遍历上的消耗更多，而对于for-of来说，需要去调用Symbol.iterator函数来构建一个迭代器，自动调用next()，且不向next()传入任何值，在接收到done:true之后停止，自然回比前三种方法慢。

（这里的性能测试我使用了console.time()和console.timeEnd()之间的时间差距来比对，因为会受浏览器等各种因素影响，上面的结论是我测试了多组数据后得出的结论）
```javascript
let arr = [];
arr.length = 10000;
arr.fill(1);
 
(function() {
    console.time('for循环');
    for (var i = 0; i < arr.length; i++) {
        // ... 相同操作
    }
    console.timeEnd('for循环')
})();
 
(function() {
    console.time('while循环')
    var i = 0;
    while (i < arr.length) {
        // ...相同操作
        i++;
    }
    console.timeEnd('while循环')
})();
 
(function() {
    console.time('do-while');
    var i = 0;
    do {
        // ...相同操作
        i++;
    } while (i < arr.length)
    console.timeEnd('do-while');
})();
 
(function() {
    console.time('for-in');
    for (var i of arr) {
        // ...相同操作
    }
    console.timeEnd('for-in');
})();
 
(function() {
    console.time('for-of');
    for (var i of arr) {
        // ...相同操作
    }
    console.timeEnd('for-of');
})();
```
下面是其中一次的执行结果  
![](https://user-gold-cdn.xitu.io/2020/2/26/1707d73f6626d2cd?w=295&h=198&f=png&s=41026)

# 性能优化
要对循环的性能进行优化，只能减少其每次循环的工作量，或者减少循环的次数

## 减少每次循环工作量
首先看看for遍历一个数组每次循环的工作量
```javascript
for(var i=0;i<arr.length;i++){
    //...操作
}
```
1. 查找一次arr的长度

2. 比较一次i和arr.length的值

3. 判断比较结果为true或false

4. 执行for循环内的自增

5. i变量的自增（如果步骤3中判断结果为true的话）

## 存储arr.length的值
每次我们都要去查找一次length的值，那么我们为什么不先在设置初始条件时使用一个变量来存储arr的length值呢。
```javascript
for(var i=0,len=arr.length;i<len;i++){
    //...操作
}
```
这样做的好处是，arr至少是在当前作用域链的下一个位置，如果我们把arr.length放在当前一个局部变量中，在查找该值时在作用域链上查找的步数就会减少，所以能有效地减少性能的消耗。这样上面的步骤就变为

1. 比较一次i和len的值

2. 判断比较结果为true或false

3. 执行for循环内的自增

4. i变量的自增（如果步骤3中判断结果为true的话）

## 倒序循环
倒序循环是一种通用的循环优化方法，我们通过代码来理解
```javascript
for(var i=arr.length;i--;){
    //...操作
}
```
此时每次循环的步骤变为

1. i==true的判断（i为0时等式即成立）

2. i变量的自减

3. 执行循环内的操作

可以看到，倒序操作实际上就是将i和数组长度的比较和判断为true或false这两步合并，以此来得到性能上的优化。

# 减少循环次数
## 达夫设备
达夫设备实际上就是在一次循环中完成多次循环的事情，以达到减少循环次数的作用，看看下面的代码（代码源自《高性能JavaScript》）
```javascript
// items为要遍历的数组，process为对数组内成员的操作方法
 
var i=items.length%8;
while(i){
    process(items[i--]);
}
 
i=Math.floor(items.length/8);
 
while(i){
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
    process(items[i--]);
}
```
这里通过取长度除以8的余数，先进行这个余数次数的循环，然后之后每次循环执行8次自减操作，相当于一次循环中完成八次循环的操作，以此来达到减少循环的次数的目的

## 基于函数的迭代
很多框架中都封装了迭代循环的方法，包括JavaScript本身也有原生的数组迭代方法，看看下列实例（代码原则《高性能JavaScript》）
```javascript
// 原生数组方法
items.forEach(function(value, index, array) {
    process(value);
});
 
// YUI 3
Y.Array.each(item, function(value, index, array) {
    process(value);
});
 
// jQuery
jQuery.each(items, function(index, value) {
    process(value);
});
 
// Dojo
dojo.forEach(items, function(value, index, array) {
    process(values);
});
 
// prototype
items.each(function(value, index) {
    process(value);
});
 
// MooTools
$each(items, function(value, index) {
    process(value);
});
```
虽然函数迭代的方法较为方便，但从性能上来说，函数迭代需要去调用函数，性能上的消耗会比普通的循环更多，所以在追求性能的情况下不适合使用函数迭代的方法。

---
参考自《高性能JavaScript》