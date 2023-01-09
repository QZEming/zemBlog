---
title: typescript基础知识（一）基本类型
abbrlink: e557e984
date: 2019-03-17 22:42:10
tags: typescript
categories: typescript
---
相对于JavaScript，typescript的类型使用较为严格，主要的类型有以下几种：布尔型（Boolean），数字类型（number），字符串类型（string），数组类型（array），元组类型（tuple），枚举类型（enum），任意类型（any），null和undefined，void类型，never类型
<!-- more -->
# 类型的严格性
在typescript中，仍然可以使用var，let，const来声明变量，但一旦确定了类型，就不能使用非该类型的值为该变量赋值。
```typescript
let num:number=1;
 
number='2'; // 报错
```
即使我们不显式地标明变量的类型，typescript也会自动为该变量声明类型（自动声明的类型不会是any这种随机的类型），而后给该变量赋不同类型的值会报错。
```typescript
let num=2;
 
num='2'; // 报错
```
# 布尔型（Boolean），数字类型（number），字符串类型（string）
布尔型，数字类型，字符串类型，除了确定类型声明后不能改为其他类型的值外，与JavaScript中的使用没有差别。
```typescript
var b:boolean=true;
 
b=false;
 
var num:number=1;
 
num=2;
 
var str:string='111';
 
str='222';
```
# 数组类型（array）
typescript中有多种数组的声明使用，正如我们所知道的，JavaScript中同个数组下的数据类型可以不同，但typescript中的数组不允许这种情况
```typescript
var arr1:number[]=[11,22,33];
 
var arr2:Array<number>=[11,22,33];
```
上面两种数组的声明方式只允许数组成员为number类型的值，如果数组成员为其他值的话会报错。

# 元组类型（tuple）
如果要创建一个数组，该数组里的成员的类型不一定相同，那么在typescript中可以使用元组类型，与数组类型不同的是，元组类型要求长度必须固定
```typescript
var arr:[number,string]=[11,'22'];
```
# 枚举类型（enum）
枚举类型简单来说就是一个可以通过枚举属性来获取值的类型，通过枚举类型，可以为某些值提供更具其特征的属性名，以此来提高代码可读性，枚举类型的属性名可加引号表示字符串也可以不加
```typescript
enum Flag { success = 1, error = 2 };
let f: Flag = Flag.error;
console.log(f);
 
//如果标识符没有赋值，其值就为它的下标
enum Color{blue,red,'orange'};
let c:Color=Color.blue;
console.log(c);
```
要注意的是，枚举类型的属性名对应的值是只读的，如果更改会报错。
```typescript
enum Flag { success = 1, error = 2 };
let f: Flag = Flag.error;
Flag.error=3;
 
// Cannot assign to 'error' because it is a read-only property.
```
# 任意类型（any)
任意类型顾名思义就是可以被赋以任何值的类型
```typescript
var num:any=123;
num=true;
num='123123';
```
任意类型可应用在获取dom节点的对象上
``` javascript
//假设当前有id位div的元素
var objDiv:any=document.getElementById('div');
objDiv.style.color='red';
```
我没也可以通过“|”来同时声明几个类型以实现任意类型的功能
```typescript
var any:number|boolean|string;
any=1;
any=true;
any='1';
```
# undefined和null
undefined和null用在变量取这两个值的时候，或者变量没有赋值的时候

但如果要使一个变量在未赋值时作为undefined类型，在赋值后变为另一个类型，就得使用“|”了
```typescript
var num:number|undefined|null;
console.log(num); // undefined
num=2;
console.log(num); // 2
```
# void类型
void类型用在函数中，表示没有返回值，和Java一样
```typescript
function run():void{
    console.log('run');
}
```
# never类型
never类型表示不存在值得的类型，可看作一个永远不会执行结束的函数的返回值
```typescript
var a:never;
a=(()=>{
    throw new Error('error');
})()
```