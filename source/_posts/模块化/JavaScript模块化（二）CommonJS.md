---
title: JavaScript模块化（二）CommonJS
tags: JavaScript模块化
categories: JavaScript
abbrlink: fcbcba89
date: 2019-08-13 22:02:41
---
# 介绍
---
CommonJS提出JavaScript不仅针对浏览器，它做为一个规范，在服务器端被广泛使用，最常见到的就是node中的使用
事实上，node就是基于CommonJS规范来写的，在CommonJS规范中，每个js文件作为一个模块，CommonJS在服务器端和浏览器端的加载是不一样的
<!-- more -->
# 基本语法
---
## 暴露模块  
在CommonJS规范中，我们通过module.exports和exports.xxx来暴露模块内部的变量和方法
```javascript
module.exports = val  
exports.key = val  
```
**暴露本质**：暴露出exports对象  
对于module.exports来说，等同于将等号后面的内容覆盖原来的exports对象  
而对于exports.xxx来说，就是在exports对象上增添属性
对于上面那两行代码来说，本质上就是:
* module.exports = val：将val赋值给exports对象，结果就是exports=val
* exports.key = val：将val赋值给exports对象的key属性，若没有就新建该属性，结果就是exports={key : val}

通过本质可以看出
* module.exports因为会修改整个exports对象，所以重复写会使得后写的覆盖先写的，而exports.xxx只会不断添加属性，除非xxx是相同的，否则就可以一直叠加下去

```javascript
module.exports = {
    foo(){
        console.log('foo');
    }
}
module.exports = {
    bar(){
        console.log('bar');
    }
}
```
在这里，后写的module.exports会把之前的module.exports覆盖掉，exports对象将只剩一个bar方法
```javascript
exports.foo = function(){
    console.log('foo');
}
exports.bar = function(){
    console.log('bar');
}
```
而这里的exports.xxx则会使得暴露出去的exports对象同时有foo和bar方法
此外，要注意的是，CommonJS输出的内容是值的拷贝，也即是说在导入模块后，修改模块方法并不会影响到原有的模块的内容
## 引入模块
require(xxx)
第三方模块：直接使用模块名
如下面使用node创建服务器，就直接引用了http模块（代码来自[node官网](https://nodejs.org/en/about/)）
```javascript
// node创建服务器的代码
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
自定义模块：使用文件路径，后缀名默认为js
```javascript
// module.js
exports.foo = function(){
	console.log('foo');
}
// mian.js
const module = require('./module.js')
module.foo()
```
使用require时要注意相对路径的写法
* '/xxx' ：表示根目录的xxx文件；
* './xxx' ：表示和当前文件同一文件夹下的xxx文件；
* '../xxx' ：表示当前文件的上级目录的xxx文件；
* ’xxx'：表示要在node_modules寻找相应的模块
# 加载机制
---
## 不同端的加载方式
在服务器端模块的加载是**同步**的：可能出现阻塞
在浏览器端模块需要**提前编译打包处理**，即使请求发送到了服务端，但是之前的模块还没加载好，所以客户端会一直等待
正因其加载的不同，CommonJS更适合在服务器端使用，在服务器中同步加载，只要将所有导入都放在模块（文件）顶部（至少要放在使用该模块相应内容的上一行），我们就可以保证在引用其他模块的方法前已经加载好这些模块，而对于浏览器环境来说，我们不知道网络情况如何，如果我们发起了一个异步请求，而这些模块还没加载好的时候我们执行了接下来的代码，而这些代码里面又需要这个模块的方法，那就会报错
就如下面的代码
```javascript
require('jquery')
var _body = $('body')
```
在浏览器环境下，如果jquery模块还没加载好，那么$('body')就会报错，所以在浏览器中不适合使用CommonJS规范
此外，浏览器不认识require，需要提前处理语法，可以通过使用browserify打包来处理
## 模块缓存
在Node下加载模块时，Node会将加载的模块缓存下来，在下次请求该模块时直接丛缓存中获取该模块，所以多次使用require实际上只会请求一次模块
# 使用
---
* 服务端：Node
* 浏览器端：需要进行打包，可以使用**browserify**打包
browserify需要在全局安装后再在本地安装才能使用
```
npm i browserify -g
npm i browserify
```
安装后通过browserify命令打包
```
browserify main.js -o bundle.js
```
main.js是要打包的文件，bundle.js是打包生成的文件