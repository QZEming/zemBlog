---
title: vue问题笔记ref获取不到指定的DOM节点问题解决
abbrlink: bf1e52c1
date: 2019-07-13 15:51:02 
tags: vue
categories: 框架
---
在vue中不建议操作DOM，但是如果我们在没办法的情况下，可以通过ref来进行DOM操作，而在使用DOM时，可能会有获取不到ref对应DOM节点的问题  
像下面这种情况，这是执行了console.log(this.$refs)的情况  
<!-- more -->
![this.$refs无法正常获取](https://user-gold-cdn.xitu.io/2020/2/20/17062434deeaf2c1?w=887&h=230&f=png&s=65657)  
在用console打印出来的时候明明看到可以按下箭头显示我们要的组件内容，但是在花括号里面却写的是undefined，此时如果要获取这个节点的属性值，会报无法从undefined取值的错误。

之所以会这样，是因为vue在更新DOM是异步的，只要侦听到数据发生变化，Vue将开启一个队列，并缓存在同一事件循环中发生的所有数据变更。即是说，在我们更新数据时，组件不会立即重新渲染，而是在刷新队列时才进行渲染。

在上面代码中，DOM节点还没创建出来的时候，就执行了这个方法，导致无法正确地获取DOM节点，所以我们应该在DOM节点创建后才执行这个方法。

对于这个问题，其实只要做个异步处理就可以了，最简单的做法就是让这个方法延迟一点执行，使用一个setTimeout来将这个方法延迟执行。
```javascript
setTimeout(function(){
    console.log(this.$refs);
},time)
```
虽然setTimeout可以解决这个问题，但是这个time我们无法确定，因为影响解析的因素很多，如果延迟的时间太长又可能造成其他问题，所以我们采用另一个方法：Vue.nextTick(callback)。

Vue.nextTick(callback)可以让里面的方法在DOM节点全部解析后才执行。在组件内使用时可以直接使用this来代替Vue，this会自动绑定到当前的Vue实例上。
```javascript
this.$nextTick(function(){
    console.log(this.$refs);
})
```
同时，因为$nextTick()返回一个Promise对象，所以可以使用async/await语法完成同样的事情。
```javascript
await this.$nextTick();
console.log(this.$refs);
```
至此，问题解决