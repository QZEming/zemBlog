---
title: JavaScript模块化（四）CMD
tags: JavaScript模块化
categories: JavaScript
abbrlink: eda08cc9
date: 2019-08-13 22:03:20
---
# CMD（Common Module Definition）
## 介绍
---
CMD依赖于sea.js，在语法上杂糅了CommonJS和AMD，可以同时使用同步加载和异步加载，适用于web浏览器端模块，同样的，在CMD中，一个文件代表一个模块
现在CMD似乎是被国外收购了，登上seajs.org会显示以下的内容
<!-- more -->
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813185322455.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
但我们还是可以在中文文档中看到sea.js的API：https://www.zhangxinxu.com/sp/seajs/ ，国内还有一些老项目也还在使用sea.js，所以还是有必要学习了解以下的
## sea.js语法
---
sea.js基本语法采用```define（function(){})```，看似和AMD一样，但是sea.js不管有没有暴露模块或者引入模块，都只有这一个参数，通过给这个参数方法传入不同的参数来达到引入模块和暴露模块的效果
#### 只需引入的模块
```javascript
define(function(require){
    // 引入同步依赖
    var module1  = require('./module1');
    // 引入异步依赖
    require.async('./module2',function(m3){
        // 回调方法
    })
})
```
require表示引入依赖，如通CommonJS的require一样，但是有一点不同的是，在CMD中，可以通过使用require.async来异步引入模块，require.async传入两个参数，第一个参数为要导入的模块，第二个参数为导入模块成功后的回调函数，即异步处理
#### 需要暴露的模块
无依赖的模块
```javascript
define(function(require,exports,module){
    // ...
    // 暴露模块
    exports.xxx = val;
    module.exports = val;
})
```
有依赖的模块
```javascript
define(function(require,exports,module){
    // ...
    // 暴露模块
    exports.xxx = val;
})
```
这里的exports和module和CommonJS中的module.exports一个道理，也是将exports对象输出，module指向的就是本模块，而exports指向的是module.exports
#### 在浏览器中引入
```html
<script type="text/javascript" src="js/libs/sea.js"></script>
<script>
    seajs.use("./js/main.js")
</script>
```
通过使用seajs.use()方法引入main.js，再通过main.js去依赖各个模块来加载所有的模块