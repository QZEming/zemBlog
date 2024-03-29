---
title: JavaScript模块笔记（一）模块化由来
tags: JavaScript模块化
categories: JavaScript
abbrlink: 60b150c9
date: 2019-08-13 22:02:18
---
# 无模块概念
---
JavaScript模块化的历史漫长，一开始，正如在我们初学JavaScript的时候，写的代码不多，我们都是将代码写在一个文件里，因为代码量不多，所以没有感到什么不适，
<!-- more -->
```javascript
function foo(){
    // ...
}
function bar(){
    // ...
}
```
但是当代码开始变多的时候，我们开始感到不适，我们很难找到自己写过的函数，往往会出现命名重复覆盖的现象，挤在一起来的的问题就是
	    1. 容易污染全局  
	    2. 不便于维护
即使我们放到多个文件里，结果还是一样的，依然会污染全局和不便于维护

# 以命名空间作为模块
---

后来，为了减少对环境的污染，采用了Namespace模式
```javascript
var MYFUN={
    foo:function(){},
    bar:function(){}
}
```
通过这样来减少了全局上的变量数目，而且我们可以将方法写在这些命名空间里，只要命名空间命名语义化，我们不难找到自己的方法
但是该命名空间本质上是对象，很容易被修改，不安全，我们可以随意在命名空间外查看命名空间内的变量，方法，即命名空间对象的属性，而且也能随意修改其中的内容

# 以匿名函数作为封闭模块
---
为了解决安全问题，使用了匿名函数IIFE模式，封闭了“模块”内部的变量，方法，只将需要被调用的内容以return的方式暴露出去，这样在全局看不到匿名函数里面的逻辑，相对安全
```javascript
var MODULE = (function(){
    var _private = 1;
    var foo = function(){
        console.log('_private: ', _private);
    }
    return {
        foo:foo
    }
})
// 调用方法
Module.foo()
// 无法获取其中的私有变量
Module._private // undefined
```
然而，这样还存在一定的问题，我们还需要能将其他模块引入当前模块
# 引入依赖 模块模式
---
```javascript
var Module = (function($){
    var _$body_ = $("body"); // 通过这样我们可以在匿名函数中使用其他模块的内容，这里使用了jQuery
    var foo = function(){
        console.log('_private: ', _private);
    }
    return {
        foo:foo
    }
})(jQuery)
```
这是现在的模块实现的基础，在上面代码中，将jQuery模块导入了匿名函数中，使得在匿名函数中也可以使用jQuery
现在jQuery源码的最外层就是立即执行函数，在调用时传入了window对象，利用window对象暴露接口，所以我们在导入了jQuery后可以直接使用$，实际上是使用了window.$，通过依赖注入将window对象注入了jQuery，所以看起来我们像是在全局使用了$。
jquery最外层代码
```javascript
( function( global, factory ) {
	// ...
} )( typeof window !== "undefined" ? window : this, function( window, noGlobal ) {
	// ...
	return jQuery;
} )
```
# 为什么要模块化
---
正如上面提及的，模块化是为了降低代码的复杂性和耦合性，在代码量很少的时候，我们没有必要使用模块化，但在代码量多的时候，如果不进行模块化，我们的代码会难以阅读，而且也很难去维护
 
# 模块化带来的好处
---
1.避免命名冲突（减少对全局空间的污染）
通过使用模块化，使得我们在全局空间声明的变量，方法减少，减少了全局空间的污染，即使是最原始的命名空间，也减少了在全局空间的声明
2. 更好地分离，按需加载  
       将各个部分所需要的变量，方法分布在各个模块，各个部分所需要的变量，方法更为清晰，而我们也只在需要用到这些部分的功能时才调用对应的模块
3. 更高的复用性
       通过写在模块内，像是声明方法后多次调用一样，我们可以多次调用模块内容，达到简单复用的效果
4.更高的可维护性
       将变量，函数细分到模块内部，我们更容易根据所要修改的功能找到对应的变量，函数，更便于我们维护

# 模块化带来的问题
---
1.发起的请求变多
因为将一个JavaScript文件拆分成多个模块，所以我们要发起多次的javascript请求，
2.模块加载顺序错误会带来错误
需要注意模块加载的顺序，在需要对应的模块前要加载好该模块，如果在加载某个模块前调用了该模块的方法，自然是会报错的
3.依赖模糊
如果模块的命名不够语义化，可能会难以看出所要使用的方法来自哪个模块

---
参考：[JavaScript 模块化七日谈](http://huangxuan.me/js-module-7day/#/)