---
title: webpack学习笔记（一）入口和出口的简单配置
abbrlink: e4866470
date: 2019-04-21 17:13:52
tags: webpack
categories: webpack
---
在webpack4.0后，可以不使用配置文件，但其仍是可高度配置的。在根目录下创建webpack.config.js，对其module.exports的属性进行配置即为webpack的配置。
<!-- more -->
# 入口（entry）
本质上，webpack 是一个现代 JavaScript 应用程序的静态模块打包器，在入口中指定了要打包的文件，同时指示 webpack 应该使用哪个模块，来作为构建其内部依赖图的开始。进入入口起点后，webpack 会找出有哪些模块和库是入口起点（直接和间接）依赖的。每个依赖项随即被处理，最后输出到称之为 bundles 的文件中，bundles可以在出口设置其命名。

通过在webpack.config.js中配置entry属性可以指定一个或多个入口起点，默认值为./src，即将src文件夹下的文件作为入口文件。

## 对象语法设置
通过给entry属性传入一个对象作为值，可以对对象指定路径的文件进行打包。
```javascript
module.exports = {
    entry: {
        first: './src/first.js',
        second: './src/second.js'
    }
}
```
通过上面的配置，在打包时就会打包src文件夹下的first.js和second.js。对象的键名可以在output中被获取，以用于输出指定名字的js文件。

当只有一个入口时，也可以传入一个指定路径的字符串来达到简写的目的
```javascript
module.exports = {
    entry: './src/index.js'
}
```
这种简写实际上为
```javascript
module.exports = {
    entry: {
        main: './src/index.js'
    }
}
```
这样子在打包时输出的文件名就会为main.js。

# 出口（output）
output指定了要输出的位置，默认为./dist，即将打包的文件放到dist文件夹下。

## 配置输出
在配置输出文件的位置时，最常用的是filename属性和path属性，filename属性用于指定打包后的文件的名称，而path属性用于指定打包后的文件的存放路径。要注意的是，path属性只能使用绝对路径，不能使用相对路径。
```javascript
module.exports = {
    entry: "./src/index.js",
    output: {
        filename: "index.js",
        path: __dirname + "/dist"
    }
}
```
__dirname指webpack.config.js所在的根目录，通过拼接字符串可以达到使用绝对路径的效果。  
原文件夹下文件目录  
![](https://user-gold-cdn.xitu.io/2020/2/26/17080e3c79b867a4?w=278&h=218&f=png&s=20207)


打包后的文件夹下文件目录

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e43640264bb?w=248&h=301&f=png&s=28128)


这是在配置一个出口文件时，可以直接给filename属性传一个字符串，但是如果有多个入口文件，就可以通过[name]来获取entry属性对象的键值作为输出的文件的名称。
```javascript
module.exports = {
    entry: {
        first: "./src/first.js",
        second: "./src/second.js"
    },
    output: {
        filename: "[name].js",
        path: __dirname + "/dist"
    }
}
```

原文件夹下文件目录

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e4e5c80d3a1?w=272&h=260&f=png&s=24390)


打包后的文件夹下文件目录

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e528d119391?w=258&h=392&f=png&s=35840)

## 默认输出
在没有设置output属性时，对下面的entry配置下的文件进行打包
```javascript
// webpack.config.js文件内容
module.exports = {
    entry: "./src/index.js"
}
```
原文件夹下文件目录

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e63348c15b7?w=264&h=211&f=png&s=19983)


打包后的文件夹下文件目录

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e6642324d93?w=280&h=314&f=png&s=28338)


可以看到在entry属性使用简写，又没有手动设置output属性时，会在dist文件夹下生成一个main.js（因为简写下的内容等同于在对象语法中键值为main的属性）

由生成的文件名为main.js可以推出，output属性的默认为
```json
output: {
    filename: "[name].js",
    path: __dirname + "/dist"
}
```
参考：[webpack中文文档](https://www.webpackjs.com/concepts/)