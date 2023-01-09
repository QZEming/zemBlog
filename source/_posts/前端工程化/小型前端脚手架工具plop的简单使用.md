---
title: 小型前端脚手架工具plop的简单使用
abbrlink: dde42bd0
date: 2020-02-02 17:19:17
tags: 前端工程化
categories: 前端工程化
---
在我之前的另一篇文章[前端工程化 通过yeoman-generator将文件加载到本地](https://blog.csdn.net/zemprogram/article/details/104125933)中说到了如何通过yeoman将文件加载到本地，但实际上使用plop工具也可以
<!-- more -->
首先要将plop安装到全局环境
```
npm i plop -g
```
再将plop安装到当前项目中
```
npm i plop -D
```

然后要在项目根目录中新建一个plopfile.js文件，用于编写在命令行向用户提出问题，并根据用户输入的内容来进行各种操作，是plop的入口文件，需要导出一个函数，该函数接收一个plop对象做为参数

这里和使用yeomen一样先创建一个plop-temp文件夹来存放模板文件

```javascript
module.exports = plop=>{
    plop.setGenerator('wxfile',{ // 这里的wxfile是一个自己设定的名字，在执行命令行的时候会用到
        description:'create the repeat wxfile', // 这里是对这个plop的功能描述
        prompts:[{
            type:'input', // 问题的类型
            name:'fileName', // 问题对应得到答案的变量名，可以在actions中使用该变量
            message:'your fileName is', // 在命令行中的问题
            default:'page' // 问题的默认答案
        }],
        actions:[{
            type:'add', // 操作类型，这里是添加文件
            path:'{{fileName}}.js', // 添加的文件的路径
            templateFile:'plop-temp/temp.js' // 模板文件的路径
        }]
    })
}
```
目录如下  
![](https://img-blog.csdnimg.cn/20200202163942133.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)  
在temp.js中随便写入点东西
```javascript
let data = 1
```
在命令行执行plop wxfile  
![](https://img-blog.csdnimg.cn/20200202164351448.png)  
输入test，在根目录生成了一个test.js文件，打开test.js文件，里面的内容和temp.js的内容是一样的

如果我们要将命令行填入的内容写到文件里，可以通过在文件内写入\{{}}，\{{}}内写入对应问题的变量就可以了，比如我们在temp.js里写入
```javascript
let title = "{{fileName}}"
```
执行plop输入test后，test.js的内容就会变成
```javascript
let title = "test"
```
## 同时进行多个操作
---
如果要进行多个操作，只要在actions数组后面添加新的成员就可以了
```javascript
module.exports = plop=>{
    plop.setGenerator('wxfile',{ // 这里的wxfile是一个自己设定的名字，在执行命令行的时候会用到
        description:'create the repeat wxfile', // 这里是对这个plop的功能描述
        prompts:[{
            type:'input', // 问题的类型
            name:'fileName', // 问题对应得到答案的变量名，可以在actions中使用该变量
            message:'your fileName is', // 在命令行中的问题
            default:'page' // 问题的默认答案
        }],
        actions:[{
            type:'add', // 操作类型，这里是添加文件
            path:'{{fileName}}1.js', // 添加的文件的路径
            templateFile:'plop-temp/temp.js' // 模板文件的路径
        },{
            type:'add',
            path:'{{fileName}}2.js',
            templateFile:'plop-temp/temp.js'
        },]
    })
}
```
执行plop wxfile，输入test，发现多了两个文件，test1文件和test2文件，文件的内容都是一样的，因为使用了同一个模板
## 将文件放入新生成的文件夹
---
如果要将生成的文件放入一个新建的文件夹，只要在放入的路径写入一个没有的文件夹名称，就会先生成一个新的文件夹，再将生成的文件放入
```javascript
module.exports = plop=>{
    plop.setGenerator('wxfile',{ // 这里的wxfile是一个自己设定的名字，在执行命令行的时候会用到
        description:'create the repeat wxfile', // 这里是对这个plop的功能描述
        prompts:[{
            type:'input', // 问题的类型
            name:'fileName', // 问题对应得到答案的变量名，可以在actions中使用该变量
            message:'your fileName is', // 在命令行中的问题
            default:'page' // 问题的默认答案
        }],
        actions:[{
            type:'add', // 操作类型，这里是添加文件
            path:'folder/{{fileName}}.js', // 添加的文件的路径
            templateFile:'plop-temp/temp.js' // 模板文件的路径
        }]
    })
}
```
这里folder本来是没有的，在执行命令后就会先新建这个文件夹再将生成的文件放入