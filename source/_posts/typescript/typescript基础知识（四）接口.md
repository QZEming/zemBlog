---
title: typescript基础知识（四）接口
abbrlink: 328e8be
date: 2019-03-20 09:41:21
tags: typescript
categories: typescript
---
在我的理解中，接口是用于对一系列方法/属性定义一些标准化的内容，在typescript中体现为对类型的标准化，使用interface关键字来定义一个接口
<!-- more -->
# 属性接口
属性接口针对于对象的属性标准化
```typescript
interface Information{ // 接口
    name:string;
    age:number;
}
 
function person(info:Information):void{ 
// 传入的对象必须包含类型为string的属性name和类型为number的属性age
    console.log(`I am ${info.name},and I am ${info.age} years old now`);
}
 
person({name:'小明',age:18});
```
要注意的是，如果和上面一样在传入参数的位置写出变量，那么对象只能包含接口里面的属性，而如果事先定义好一个对象，那么可以包含接口里面的属性外的其他属性，因此最好事先定义一个对象再作为参数传入
```typescript
person({name:'小明',age:18}); // 正确
 
person({id:2,name:'小明',age:18}); // 错误
 
let obj={
    id:2,
    name:'小明',
    age:18
}
person(obj); // 正确
```
# 可选属性接口
如果有些参数可传可不传，可以通过在属性名和类型之间加入“?”来使其变为可选属性
```typescript
interface information{
    name:string;
    age?:number;
}
 
function person(info:information){
    if(!info.age){
        console.log(`I am ${info.name}`);
    }else{
        console.log(`I am ${info.name},and I am ${info.age} years old now`)
    }
}
 
let p1={
    name:'小明'
}
 
let p2={
    name:'小红',
    age:18
}
 
person(p1); // I am 小明
 
person(p2); // I am 小红,and I am 18 years old now
```
# 函数接口
函数接口是用于标准化函数的参数个数，参数类型以及返回值的类型
```typescript
interface fn{
    (first:string,second:number):string;
}
 
var fn:fn=function(f:string,s:number):string{
    return `f:${f}----s:${s}`;
}
console.log(fn('1',2)); // f:1----s:2
```
接口中括号内是参数的个数和类型，按顺序写出参数的类型，括号后面即为函数返回值的类型

# 可索引接口
可索引接口是用于标准化有索引的类型，如数组和对象，但不是很实用
```typescript
interface Arr{
    [index:number]:string; // 下标为number，成员为string，也就是数组了
}
 
let arr:Arr=['123','123'];
 
interface Obj{
    [index:string]:string; // 下标为string，也就是对象了
}
 
let obj:Obj={name:'小明',age:'18'};
```
# 类接口
类接口用于标准化类，类使用类接口和子类继承抽象类很类似，都必须有其内定义的所有属性方法，且类型需要相同，类通过implements关键字来使用接口
```typescript
interface Person{
    name:string;
    work():string;
}
 
class Student implements Person{
    name:string;
    constructor(name:string){
        this.name=name;
    }
    work(){
        return '读书';
    }
}
```
# 接口间的继承
接口之间也是能使用继承的，被子接口约束的内容也会被其继承的父接口约束，接口间的继承通过extends关键字来实现
```typescript
interface Person{
    name:string;
    introduce():string;
}
 
interface Student extends Person{
    work():string;
}
 
class Monitor implements Student{ 
// 通过使用Student接口，同时受到了Person接口的约束
    name:string;
    constructor(name:string){
        this.name=name;
    }
    introduce(){
        return `I am ${this.name}`;
    }
    work(){
        return 'study';
    }
}
```
# 泛型接口
在说泛型接口前先说一下泛型

泛型的使用使得参数可以在使用时才确定参数的类型，对于不确定类型的使用有很大的应用性

## 泛型函数
泛型通过将<>来实现，<>里面的内容即为泛型的具体数据类型，在声明时可随意写入字母
```typescript
function fn<T>(val:T):T{
    return val;
}
 
fn<number>(1);
```
## 泛型类
```typescript
class Person<T>{
    out(val:T):T{
        return val; 
    }
}
 
let p=new Person<number>();
p.out(1);
```
## 接口
泛型接口即在标准化内容时使用接口来进行标准化，使得在使用时可以使用不同的数据类型
```typescript
interface Gen{
    <T>(value:T):T;
}
 
var gen:Gen=function<T>(val:T):T{
    return val;
}
 
gen<number>(2);
```