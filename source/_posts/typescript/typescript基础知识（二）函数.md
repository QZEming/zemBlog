---
title: typescript基础知识（二）函数
abbrlink: 321104a4
date: 2019-03-18 21:20:02
tags: typescript
categories: typescript
---
# 函数基本声明使用
typescript中的函数其实和ES6中的函数差不多，只是typescript中的函数在使用时要明确返回值的类型（因为最后是编译成JavaScript执行的，所以不写出返回值的类型也不会报错）明确的方法就是在括号后面加上要返回的类型
<!-- more -->
```typescript
// 没有返回值
function fn():void{
    console.log('123');
}
 
// 返回值为number类型
function fnNum():number{
    return 1;
}
```
这里void类型表示可以不用返回值。

# 函数的类型
typescript中的函数类型和JavaScript中的函数类型是差不多的，基本的就有无参函数，带参函数，ES6的rest函数

## 无参函数
```typescript
function fn():void{
    console.log('fn');
}
 
function fnNum():number{
    return 1;
}
```
## 带参函数
带参函数的参数有三种情况，普通的参数，可选的参数，有默认值的参数

### 带普通参数的函数
函数的参数的类型在声明函数时就要一起声明，且不可修改，也不可不传入该参数，同样不可以传入多个参数，这与JavaScript中没传入参数默认为undefined，传入过多参数放在arguments数组中不同
```typescript
function introduce(name:string):string{
    return `my name is ${name}`;
}
```
### 带可选参数的函数
如果某个参数不知道是否会传入，那可以通过在该参数和类型前的“:”中间加个“?”来使该参数变为可选参数
```typescript
function introduce(name:string,age?:string):string{
    if(age)
        return `my name is ${name} ,and my age is ${age}`;
    else
        return `my name is ${name}`;
}
```
### 带有默认值参数的函数
如果要使一个参数在不传入值时默认为为一个值时，可以使用默认参数，默认参数的用法和ES6中默认参数的用法类似，直接在函数声明的参数后面写上“=”+参数的默认值
```typescript
function introduce(name:string,age:string='20'):string{
    return `my name is ${name} ,and my age is ${age}`;
}
 
console.log(introduce('jack', '18'));
// my name is jack ,and my age is 18
 
console.log(introduce('jack'));
// my name is jack ,and my age is 20
```
### rest函数
rest函数与ES6中的rest函数类似，使用扩展运算符来实现rest函数。不同的在于这里的rest参数需要声明为数组类型，且需要为成员声明类型
```typescript
function add(...res:number[]):number{
    let sum:number=0;
    for(let i=0;i<res.length;i++){
        sum+=res[i];
    }
    return sum;
}
```
# 函数的重载
在JavaScript中，如果我们写入两个同名函数，那么后面的函数就会替代前面的函数，也就没有了函数的重载，但在typescript中，可以使用函数的重载。typescript中的函数重载是重载传入的参数类型不同的函数，通过事先声明同名的函数和参数的类型，返回的类型，接着使用传入参数为any，返回值类型为any的函数体来完成对应的工作
```typescript
function f(name:string):string;
function f(age:number):string;
function f(str:any):any{
    if(typeof str=='string') // 通过判断传入参数的类型来判断执行什么操作
        return `my name is ${str}`;
    else
         return `my age is ${str}`;
}
```