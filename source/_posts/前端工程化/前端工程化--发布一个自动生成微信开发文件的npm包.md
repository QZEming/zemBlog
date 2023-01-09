---
title: 前端工程化--发布一个自动生成微信开发文件的npm包
abbrlink: 7db00131
date: 2020-02-01 17:01:44
tags: 前端工程化
categories: 前端工程化
---
做为一个前端开发者，要是连npm包也没用过那也是太low了，我们知道，我们用到的npm包实际上也是别人开发发布上去的，那我们如何去发布自己的npm包呢
<!-- more -->
我们知道，在开发微信小程序的时候，每个页面对应的文件放在一个文件夹下，这个文件夹的名字和里面文件的文件名都要一样，而每次写一个新的页面我们都要先创建一个文件夹，再按这个文件夹的名字来创建wxml，wxss，js，json文件，万一写错了还会报错，这里写一个generator来完成这个创建工作
#  编写generator
新建一个文件夹generator-wxfile
npm init初始化内容，一切选项按默认的来
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201152805701.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
接下来下载npm包yeoman-generator
```
npm i yeoman-generator
```
创建如图目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201163041659.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
templates中存放的是模板文件，就是我们最终要放到本地的文件的模板，，我们看看基础的模板文件需要什么，wxml和wxss不需要什么东西，我们在json文件内写入一个{}
```json
{}
```
在js文件里写入各个生命周期函数，这里的第3行<%= fileName%>是用了ejs语法，用来标识文件名
```javascript
Page({
    data: {
      text: "This is <%= fileName%> data."
    },
    onLoad: function(options) {
      // 页面创建时执行
    },
    onShow: function() {
      // 页面出现在前台时执行
    },
    onReady: function() {
      // 页面首次渲染完毕时执行
    },
    onHide: function() {
      // 页面从前台变为后台时执行
    },
    onUnload: function() {
      // 页面销毁时执行
    },
    onPullDownRefresh: function() {
      // 触发下拉刷新时执行
    },
    onReachBottom: function() {
      // 页面触底时执行
    },
    onShareAppMessage: function () {
      // 页面被用户分享时执行
    },
    onPageScroll: function() {
      // 页面滚动时执行
    },
    onResize: function() {
      // 页面尺寸变化时执行
    },
    onTabItemTap(item) {
      // tab 点击时执行
      console.log(item.index)
      console.log(item.pagePath)
      console.log(item.text)
    },
    // 事件响应函数
    viewTap: function() {
      this.setData({
        text: 'Set some data for updating view.'
      }, function() {
        // this is setData callback
      })
    },
    // 自由数据
    customData: {
      hi: 'MINA'
    }
  })
```
接下来编写index.js文件，引入yeoman-generator模块，输出一个generator模块，具体的generator构建看[前端工程化 通过yeoman-generator将文件加载到本地](https://blog.csdn.net/zemprogram/article/details/104125933)，这里再引入一个fs模块用于创建文件夹
```javascript
const Generator = require("yeoman-generator")
const fs = require("fs")

module.exports = class extends Generator{
    prompting(){
        return this.prompt([{ // 询问用户要创建的文件夹的名字
            type:"input",
            name:"fileName",
            message:"your file name is",
            default:"page"
        }]).then(answer=>{
            this.answer = answer // 将回答放到answer属性中
        })
    }
    writing(){
        const fileName = this.answer.fileName
        const tempFile = ["fileName.js","fileName.json","fileName.wxml","fileName.wxss"]
            .map(path=>this.templatePath(path))
        const outputFile = [`${fileName}/${fileName}.js`,`${fileName}/${fileName}.json`,`${fileName}/${fileName}.wxml`,`${fileName}/${fileName}.wxss`]
            .map(path=>this.destinationPath(path))
        fs.mkdirSync(fileName) // 创建文件夹
        for(let i=0;i<tempFile.length;i++){
            this.fs.copyTpl(tempFile[i],outputFile[i],this.answer) // 文件写入
        }
    }
}
```
执行npm link，将模块链接到全局，在本地测试一下，新建一个文件夹执行yo wxfile
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201163528560.png)
这样就创建成功了，而且查看js文件，发现第三行的text的值变成了"This is newPage data."
# 发布到npm上
执行npm publish就可以发布了，下面这样就是发布成功了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200201164129781.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
当然，也可能会遇到踩坑的情况，可以看下[npm publish发布时的踩坑记录](https://blog.csdn.net/zemprogram/article/details/104135553)

到这里就发布成功了，可以通过npm i generator-wxfile@1.0.0 -g来将这个generator下载到全局使用，也可以先下载我的试试看和你做的效果是不是一样
上面说的是1.0.0版本的功能，后面我会对这个npm包做一些修改
在1.0.1版本中，我在添加文件的同时，将相应的路由写入了app.json文件的pages属性中，但是yo wxfile要在app.json文件所在的目录执行
后面的版本还在持续更新中