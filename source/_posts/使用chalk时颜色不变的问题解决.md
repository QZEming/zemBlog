---
title: 使用chalk时颜色不变的问题解决
abbrlink: 885ed9b7
date: 2020-02-03 17:33:24
tags: npm
categories: npm
---
<!-- more -->
一般情况下是直接引入chalk然后使用相关的API就可以改变颜色了
```
const chalk = require("chalk")
console.log(chalk.green("hello"))
```
执行上面的内容，字体颜色却不变，有下面的解决方法
1. 关闭工具重新打开
虽然不知道为啥，但我的git工具确实有次是关闭后重新打开就可以了
2. 设置enabled为true
这是我去[issues](https://github.com/chalk/chalk/issues/234)上看到的，有两种设置方式
```javascript
// 第一种
const chalk = new require("chalk").constructor({ enabled: true });
console.log(chalk.green("hello"))
// 第二种
const chalk = require("chalk")
chalk.enabled = true
console.log(chalk.green("hello"))
```
3. 设置level
这个我倒是尝试成功了，可能是chalk的level为0，将其设置为1后，颜色就可以改变了
```javascript
const chalk = require("chalk")
chalk.level = 1
console.log(chalk.green("hello"))
```
去github上看chalk的文档，找到了下面这张图，所以也可以设置成2和3
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200204015638173.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)


![在这里插入图片描述](https://img-blog.csdnimg.cn/202002031732489.png)