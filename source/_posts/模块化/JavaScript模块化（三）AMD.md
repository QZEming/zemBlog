---
title: JavaScript模块化（三）AMD
tags: JavaScript模块化
categories: JavaScript
abbrlink: 7ee7a4b6
date: 2019-08-14 11:08:41
---
# AMD（Asynchronous Module Definition）
## 介绍 
---
AMD也是一个模块化的规范，它以require.js为基础，每个文件代表一个模块，与CommonJS不同的是，AMD是一个异步模块实现规范，且AMD更加侧重于浏览器，在运行时通过提前加载依赖，等到依赖加载完成再实现对应的方法来保证方法的正确实现。
<!-- more -->
## require.js的语法
---
这里使用一个样例来说说语法
文件结构
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813161605239.png)
### define语法
#### 没有依赖的模块
```javascript
// module1.js
define(function() {
    let msg = 'module1';
    let getMsg = function() {
        return msg
    }
    return {
        getMsg
    }
});
```
在不依赖模块的时候，define传入一个参数，该参数为一个立即执行的方法，通过return可以将define内的变量，方法输出，而这些输出是作为着整个文件的输出
#### 有依赖的模块
```javascript
// module2.js
define(['module1'], function(m1) {
    let msg = 'module2';
    let showMsg = function() {
        console.log(msg, m1.getMsg());
    }
    return {
        showMsg
    }
})
```
在依赖模块的时候，define传入两个参数，第一个参数是要依赖的依赖数组，第二个参数是一个异步方法，当依赖数组中的依赖加载完成后，就会执行第二个参数的方法，同样地在这里通过return来输出，第二个参数的方法的参数对应了前面的模块数组中的模块，比如这里的m1就指向了module1

这里的依赖数组的字符串名需要通过顶部js文件requirejs.config来配置
```javascript
requirejs.config({
    paths: {
        module1: './modules/module1',
        module2: './modules/module2'
    }
})
```

如果使用多个define输出，那只有第一个会真正地输出
### requirejs语法
requirejs语法用在顶部文件，即页面最终引用的文件，只会依赖其他模块而不会输出，对应的模块路径也在这里配置
```javascript
// main.js
requirejs.config({
    baseUrl:'js/', // 基本路径 出发点根目录，这里就是根目录下的js文件夹
    paths: { // 配置路径 默认会加上.js后缀，且不会判断是否已经有该后缀
        module1: './modules/module1',
        module2: './modules/module2'
    }
})
requirejs(['module2'], function(m2) {
    m2.showMsg()
})
```
这里将module1的模块名配置到根目录下的js文件（看baseUrl）下的modules的module1.js文件，module2也一样，这里注意不能写上.js后缀，在引用时会自动加上.js后缀，如果我们自己引用了就会出现下图的错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190813162241265.png)

因为main.js文件会自动引入其他模块，所以我们最终只需要引入main.js

按照官方文档的写法，在html中这么引入main.js

```html
<script data-main="js/main.js" src="js/libs/require.js"></script>
```
在加载完require.js后加载data-main对应的文件

## 对第三方库的引用
---
使用AMD引入第三方库的时候，要考虑第三方库是否支持使用AMD，如jQuery中有一段代码
```javascript
if ( typeof define === "function" && define.amd ) {
	define( "jquery", [], function() {
		return jQuery;
	} );
}
```
这段代码表示当我们使用AMD的时候，jQuery使用define以"jquery"为名输出了jQuery对象，所以我们在配置的时候要注意对jQuery要配置"jquery"

然而，并不是所有的第三方库都支持AMD，如angular，因此，我们在引用angular时，要按下面的配置方法来配置
```javascript
requirejs.config({
    baseUrl:'js/',
    paths: { 
        module1: './modules/module1',
        module2: './modules/module2',
        angular: './libs/angular'
    },
    shim: {
        angular: {
            exports: 'angular' // 让angular.js使用'angular'为名输出
        }
    }
})
```
---
参考：[require.js文档](https://requirejs.org/)