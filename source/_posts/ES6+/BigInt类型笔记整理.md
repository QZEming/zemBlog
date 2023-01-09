---
title: BigInt类型笔记整理
tags:
  - ES6+
categories: JavaScript
abbrlink: 805d94e7
date: 2020-01-20 16:03:51
---

# BigInt
---
在ES10之前，JavaScript已经有6种基本类型了，而ES10带来了第七种数据类型BigInt  
在MDN上看到关于BigInt的描述  
> BigInt is a built-in object that provides a way to represent whole numbers larger than 2**53 - 1, which is the largest number JavaScript can reliably represent with the Number primitive.
BigInt可以描述比原来JavaScript可以描述的数更大的数
 <!-- more -->
### 创建BigInt
我们可以通过直接声明和为BigInt传入字符串，数值来创建BigInt类型的数值  
直接声明时，只要在数字后面加上n，就是一个BigInt类型了  
```javascript
var b1 = 123n;
var b2 = BigInt(123); // 123n
var b3 = BigInt("123"); // 123n
```
在传入字符串时，可以通过传入0x，0o,0b开头的字符串来对十六进制，八进制，二进制的数值创建BigInt类型的数值  
```javascript
var b16 = BigInt("0x10"); // 16n
var b8 = BigInt("0o10"); // 8n
var b2 = BigInt("0b10"); // 2n
```
如果传入其他字符串的话，就会报错
```javascript
BigInt("aaa");
// Uncaught SyntaxError: Cannot convert aaa to a BigInt
```

### 对BigInt类型的判断
要判断一个值是否为BigInt，可以用以下几种方式
1. typeof
```javascript
typeof 1n; // "bigint"
typeof BigInt(1); // "bigint"
```
2. constructor
```javascript
BigInt('1').constructor === BigInt; // true
```
3. Object.prototype.toString
```javascript
Object.prototype.toString.call(1n); // "[object BigInt]"
```

关于其他的类型判断见[JavaScript类型判断总结](https://blog.csdn.net/zemprogram/article/details/89646636)

### 超过安全数的计算
我们知道，JavaScript原本的值的计算最大只到2**53-1，超过这个数值的计算结果会出现错误，而使用BigInt，我们可以超过这个数值正常计算，在JavaScript中，使用Number.MAX_SAFE_INTEGER来代表这个最大的安全数  
我们看看不使用BigInt时的计算  
```javascript
var n1 = Number.MAX_SAFE_INTEGER; // 9007199254740991
n1+=10; // 9007199254741000
```
这里n1+10后的答案显然是错误的，那么我们使用BigInt来计算
```javascript
var n1 = BigInt(Number.MAX_SAFE_INTEGER); // 9007199254740991n
n1+=10n; // 9007199254741001n
```
可以看到，这里的数值正常了
### 与Number类型的比较计算
要注意的是，BigInt类型不能直接和Number类型混合计算，否则会报错
```javascript
var n = 1n+1;
// Uncaught TypeError: Cannot mix BigInt and other types, use explicit conversions
```
在比较的时候，BigInt和Number类型的值是一样的
```javascript
1n==1 // true
1n<2 // true
2n>1 // true
```

### 转为布尔值的值
BigInt变为布尔值是为true还是为false和Number一样，除了0n之外都为true  
```javascript
if(0n){
    console.log("if");
}else{
    console.log("else");
}
// "else"
Boolean(0n); // false
Boolean(1n); // true
Boolean(-1n); // true

0n||1n; // 1n
0n&&1n; // 0n
```