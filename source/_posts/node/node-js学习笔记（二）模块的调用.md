---
title: node.js学习笔记（二）模块的调用
abbrlink: afbe3858
date: 2019-03-10 22:11:24
tags: node
categories: node
---
# 模块
在JavaScript中，每个JavaScript文件都被当做一个模块使用，而我们也可以通过npm从网上下载所要依赖的模块
<!-- more -->
# 模块的调用
## 常用模块的调用
在node.js中，我们经常会使用调用模块来实现各种功能，我们看看node.js官网的一段创建服务器的代码
```javascript
const http = require('http');
 
const hostname = '127.0.0.1';
const port = 3000;
 
const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello World\n');
});
 
server.listen(port, hostname, () => {
  console.log(`Server running at http://${hostname}:${port}/`);
});
```
在第一行中就使用了require来引入所需要的http模块，然后在下面就通过http的createServer方法来创建一个服务器，上面的服务器使用node命令就可以启动了。

## JavaScript文件的调用
调用当前项目下的JavaScript文件，理所当然是需要写入路径的，假设当前index.js和main.js在同一目录下

在两个文件中写入对应的代码，在index.js中调用main.js
```javascript
//index.js
 
const main = require('./main');
 
console.log(main.a);
 
//main.js
 
var a = 2;
 
exports.a = a;
```
# require和exports
## require
在node中，通过使用require来调用其他模块，这些模块可以分为两种情况，放在node_moudles文件夹中的模块和其他路径下的模块。

对于require来说，如果在其后的括号内的路径不是以./开头，就会默认调用其父级文件夹中的node_moudles文件夹中的模块或是node安装位置的node_moudles文件夹中的模块。如上面调用http模块，或是调用使用npm导入的connect模块。
```javascript
const http=require('http');
const connect=require('connect');
```
除了父级文件夹中和node安装位置的node_moudles文件夹外，其他文件都不会是这种写法的调用对象。

而如果要调用的文件是依据于当前的文件路径而定的话，就使用./为开头来调用，这里的“./”是表明了和当前文件处于同一文件夹下的文件，如上面提及的index.js文件和main.js文件处于同一文件夹下，就可以使用require("./main.js");（这里的“.js”可以省略）

## exports
exports用于将当前文件/模块的内容暴露给其他文件，exports可以当前是一个对象，举个例子

假设当前index.js和test.js在同一文件夹下
```javascript
// index.js
 
const test = require('./test');
 
console.log(test.a);
console.log(test.b);
console.log(test.c);
 
// test.js
 
var a = 1;
var b = 2;
var c = 3;
exports.a = a;
exports.b = b;
 
//运行index.js结果
 
// 1
// 2
// undefined
```
这里test.js将变量a,b两个变量交由exports暴露出去，然后index.js调用该模块，调用的test为一个对象，属性即为exports暴露出的变量，实际上也可以为方法。

这里的exports.后面跟着的属性名是可以自由定义的，但一般和对应的变量名或者方法名相同。

如果要暴露出一个构造方法或一个类，使用module.exports来暴露更为合适，同样举个例子

假设index.js和Person.js在同一文件夹下
```javascript
//Person.js
//构造方法
function Person(name, age, sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
}
 
Person.prototype.sayHello = function() {
    console.log(`我叫 ${this.name} ，我今年${this.age}岁，我是${this.sex}的`);
}
 
//类
class Person {
    constructor(name, age, sex) {
        this.name = name;
        this.age = age;
        this.sex = sex;
    }
    sayHello() {
        console.log(`我叫 ${this.name} ，我今年${this.age}岁，我是${this.sex}的`);
    }
}
 
 
 
module.exports = Person;
 
//index.js
const Person = require('./Person');
 
var p = new Person('夏明', 20, '男');
p.sayHello();
 
//我叫 夏明 ，我今年20岁，我是男的
```
本来如果使用exports来暴露方法的话，方法就会变为赋值对象的方法了，即上面index.js文件中Person的一个方法，而通过使用module.exoprts就将对象直接变为对应的方法，用下面使用exports暴露的代码来区别一下
```javascript
//Person.js
function Person(name, age, sex) {
    this.name = name;
    this.age = age;
    this.sex = sex;
}
 
Person.prototype.sayHello = function() {
    console.log(`我叫 ${this.name} ，我今年${this.age}岁，我是${this.sex}的`);
}
 
exports.Person = Person;
 
//index.js
const Person = require('./Person');
 
var p = new Person.Person('夏明', 20, '男');
 
p.sayHello();
 
//我叫 夏明 ，我今年20岁，我是男的
```
虽然结果一样，但Person.Person显然是奇怪的，所以我们要尽量避免这么做。