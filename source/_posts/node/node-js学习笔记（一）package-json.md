---
title: node.js学习笔记（一）package.json
abbrlink: 69e25a32
date: 2019-03-10 21:39:28
tags: node
categories: node
---
# package.json文件的创建
在一个node项目中，package文件可以直接创建package.json后编写里面的内容，也可以通过使用npm init来初始化（没有时即为创建）。
<!-- more -->

直接创建很容易，直接创建名为package.json文件后编写里面的内容就可以了

使用npm init创建时，会以一个问答的形式创建，通过“回答问题”来给对应的键名赋键值。

![](https://user-gold-cdn.xitu.io/2020/2/27/1708250e0c26d289?w=734&h=683&f=png&s=399057)

这里输入的问答会将内容写入最后的package.json文件，如果不填入内容的话会使用默认的内容或者为空，package会使用当前文件夹的名字，但名字不能为中文，如果输入错误会重新“提问”一次。

一开始使用时，其实只需要写入package name，后面的内容可以一直按回车跳过（如果当前文件夹的名字符合项目名字要求的话，也可以直接按回车跳过）。最后的json文件内容如下
```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "the first demo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```
# package.json初始属性含义
以上面的代码来展示属性含义
```json
{
  "name": "demo", // 项目名称，只能由小写字母，数字和下划线组成
  "version": "1.0.0", // 版本号
  "description": "the first demo", // 对项目的描述
  "main": "index.js", // 入口文件
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1" // 指定项目生命周期各个环节要做的内容
  },
  "author": "", // 项目作者信息
  "license": "ISC" // 项目许可协议
}
```
# 使用package.json导入模块
在成功创建了package.json文件后，可以通过npm install来把依赖的模块导入

首先我们可以通过npm install mode -s 来导入需要的模块（这里的mode指代模块名，可以在https://www.npmjs.com/上查找对应的模块名），同时将该模块的名字和版本号写入pacakge.json文件中（如果此时没有创建package.json文件的话，这里会自动创建），这里以导入connect为例

![](https://user-gold-cdn.xitu.io/2020/2/27/170825227e69e6ed?w=1055&h=126&f=png&s=53738)

导入的模块会在node_modules文件夹里面，导入模块后增加了node_modules文件夹和package-lock.json文件（如果原来没有的话），package.json文件修改为下面代码
```json
{
  "name": "demo",
  "version": "1.0.0",
  "description": "the first demo",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC",
  "dependencies": {
    "connect": "^3.6.6"
  }
}
```
当我们要在另一个项目中导入当前项目所使用的模块时，我们可以将package.json文件中的dependencies属性复制到新的项目中的这个属性里面（如果没有这个属性就新建这个属性），然后通过npm install就可以将之前的项目中依赖的模块导入到当前项目中。

使用npm install导入package.json文件中注明的模块，可能会更新部分内容，但不会重新导入重复内容。

# dependencies属性版本号标志
nodejs所依赖的这些模块更新迭代的速度很快，要使用一个模块，需要确定自己所要使用的版本号，但有时候又不一定能使用明确的版本号，就需要使用一些版本号的标志，以connect为例

## 1. 不带版本号标志
当我们不带任何特殊的符号，写入一个明确的版本号时，就一定会使用该版本号，如果输入错误的版本号，在使用npm install时会报错
```json
"dependencies": {
    "connect": "1.0.0"//明确限制版本为1.0.0
}
```
## 2. ^
使用和当前写入的版本号兼容的版本，如果没有的话就寻找与之最近的版本
```json
"dependencies": {
    "connect": "^1.0.0"//使用和1.0.0兼容的版本，如果没有则使用和1.0.0最接近的版本
}
```
## 3. ~
使用近似于写入的版本号的版本，如果没有就寻找与之最近的版本，在实际使用上与^相同
```json
"dependencies": {
    "connect": "~1.0.0"//使用和1.0.0近似的版本，如果没有则使用和1.0.0最接近的版本
}
```
## 4. latest
使用最新的版本
```json
"dependencies": {
    "connect": "latest"
}
```
5.*
使用任意版本，但会从最新的版本开始寻找，实际使用和latest一样
```json
"dependencies": {
    "connect": "*"
}
```