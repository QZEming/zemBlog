---
title: webpack学习笔记（二）loader--css的打包和注入html
abbrlink: '61091780'
date: 2019-04-24 00:36:27 
tags: webpack
categories: webpack
---
loader用于对模块的源代码进行转换，在import或加载其他模块前可以使用loader进行预处理，比如可以把typescript转换成JavaScript，通过这样也允许了在JavaScript中引入css和less/sass文件。
<!-- more -->
以在JavaScript中importcss文件为例，首先要引入两个npm包：style-loader和css-loader

style-loader用于将处理好的css代码注入到html文件中的style标签

css-loader用于在import css文件前进行预处理
```
npm i style-loader css-loader -s
```
首先在根目录创建dist文件夹，src文件夹和webpack.config.js文件

![](https://user-gold-cdn.xitu.io/2020/2/26/17080e9a58f0e50b?w=323&h=260&f=png&s=28071)

在src下创建style文件夹存放css文件，编写css文件如下
```css
div {
    width: 100px;
    height: 100px;
    background-color: black;
}
```
在src根目录下创建index.js文件并import css文件，创建一个dom并插入到页面中
```javascript
import './style/index.css';
 
document.body.appendChild(document.createElement('div'));
```
编写webpack.config.js文件
```javascript
const path = require('path');
 
module.exports = {
    entry: "./src/index.js", // 设置要打包的文件
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist')
    },
    mode: 'development',
    module: {
        rules: [{
            // 配置所有css文件
            test: /\.css$/,
            // 从右往左引用，先使用css-loader处理后再使用style-loader将css代码注入到html文件中
            use: ['style-loader', 'css-loader']
        }]
    }
}
```
在dist文件夹下创建html文件，导入打包生成的main.js
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
</head>
 
<body>
    <script src="main.js"></script>
</body>
 
</html>
```
在页面中打开该文件，检查代码，发现css代码已经注入到style标签中
```html
<html lang="en">
    <head>
        <meta charset="UTF-8">
        <meta name="viewport" content="width=device-width, initial-scale=1.0">
        <meta http-equiv="X-UA-Compatible" content="ie=edge">
        <title>Document</title>
        <style type="text/css">div {
            width: 100px;
            height: 100px;
            background-color: black;
        }
        </style>
    </head>
 
    <body>
        <script src="main.js"></script>
        <div></div>
    </body>
</html>
```
less/sass和css的处理相似，在module.rules中的use后加多一个less-loader/sass-loader进行处理即可，webpack-config.js修改如下
```javascript
const path = require('path');
 
module.exports = {
    entry: "./src/index.js", // 设置要打包的文件
    output: {
        filename: '[name].js',
        path: path.resolve(__dirname, 'dist')
    },
    mode: 'development',
    module: {
        rules: [{
            test: /\.less$/,
            use: ['style-loader', 'css-loader', 'less-loader']
        }]
    }
}
```