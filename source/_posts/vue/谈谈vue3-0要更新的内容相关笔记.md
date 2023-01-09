---
title: 谈谈vue3.0要更新的内容相关笔记
tags: vue
categories: 框架
abbrlink: cd0c37cd
date: 2020-03-18 16:06:34
---
从去年就一直说的沸沸扬扬的vue3.0发布，虽然到现在还没有正式发布的事件，但做为前端开发，我们还是要去了解的，那么vue3.0会更新什么内容呢，早在2018年11月尤大就在远程演讲中说到了，vue3.0会带来
<!-- more -->
* 更**快**
* 更**小**
* 更**易于维护**
* 更好的**多端渲染支持**
* 新功能（新的API）

下面就这些内容来说一下vue3.0的更新，这些内容基本上都是我在看了2018年和2019年的中国Vue conf视频后整理出来的，如果哪里写错了，请路过大佬帮忙指出，十分感谢  
后续如果有其他的更新我也会回来更新这篇博客，那么开始今天的博客分享之旅

# 一、 更快
## 1. 基于模板编译和Vitrual DOM性能方面的优化  

使用typescript重写Virtual DOM

我们知道，vue是使用Vitrual DOM内部渲染机制，在vue3.0中，Vitrual DOM进行了一次完全的重构，结合了模板编译等小技巧来提高性能，使得初始渲染的提速最高可以达到翻倍。  

在之前的Vitrual DOM渲染机制中，即使是静态数据的内容节点，渲染机制仍然会重新生成这些节点，然而这实际上是不必要的，在vue2.x中，已经对这部分内容做了一定的优化，通过对模板的分析，对一些静态数据的内容不再重复生成，但还不够彻底，而在vue3.0中，对这些做了更多的优化

在vue2.0中，对一个标签，无论是组件还是原生的标签，都会先以字符串的形式，传入到h这个函数中，h是用来生成虚拟node的函数，而判断一个标签是否为组件，实际上是在运行时判断的，这样就会导致在编译时会做重复的工作，而实际上，这个工作可以在编译的时候完成，如果我们知道它是一个原生的元素，那就生成相应的原生元素对应的Vitrual DOM的代码，同样的，如果判断为一个组件时，就生成组件的代码

### (1)一些底层的优化

此外，还有一些比较底层的优化，像比如
* 在调用函数处理时，函数的参数尽可能一样多，这样有益于js引擎去做一些性能优化
* 在模板中直接静态地分析一个元素所包含的子元素的类型，给运行时留下一些提示，这样可能会减少一些判断，举个例子，像下面的结构
```html
<div>
    <span></span>
</div>
```
div中只有一个span标签，在生成虚拟node的时候，标明其只有一个子元素，那么算法里面可以直接跳入只有一个元素的分支，这样就可以跳过很多的判断

### (2)优化slots生成
```html
<Comp>
    <div>{{ hello }}</div>
</Comp>
```
在之前的vue2.0中，我们像上面这样给一个组件传入一个slot值，在这个值更新的时候，父子组件都会更新，即使在父组件其他位置没有用到这个值，而实际上我妈只是要更新这一个值而已

在vue3.0的新编译机制中，把所有的slot统一编译一个函数，当我们要将一个slot传给子组件的时候，将这个函数传给子组件，而调用这个函数，是子组件的行为，所以这个“hello”的依赖，就成为了子组件的而不是父组件的，所以当hello变动时，我们只需要重新渲染子组件，而不必再去渲染父组件了

### (3)静态内容提取  
在vue2.0里面已经有做过了一些处理，如果检测到一部分模板不变，可以先提取出来，在之后的更新中，这一部分模板不仅可以直接使用原来的Vitrual DOM，甚至连整个比对过程都能直接跳过这整个树

vue2.0没做到的是，只要一个元素里面任意深度包含着一个动态数据，那就无法将其静态化，但还是可以做一些优化的，像下面这个元素，虽然其内含有动态数据text，但是这个div的属性是静态的
```html
<div id="foo" class="bar">
    {{ text }}
</div>
```
所以我们在比对时，就不需要再去比对这个元素了，可以跳过它，直接去比对它的children即可

### (4)内联事件函数提取
```html
<Comp @event="count++">
```
像上面的这个函数，在我们进行一个重渲染时，会重新去生成一个这样的函数，而因为新生成的函数和原来的函数肯定是不一样的，虽然它们做的事情是一样的，但JavaScript无法区分，所以为了安全起见，每次都会触发组件的重渲染

通过一定程度的优化，每次生成这个函数时都会进行一个cache（缓存），每次都重用一个函数，就可以起到避免子组件无谓更新的一个效果

### (5)分区块对动态节点进行追踪
在过去的节点diff中，我们是针对一整个组件去做一个diff的，diff算法所消耗的时间，会受模板整体大小的影响，比如下面的内容
```html
<template>
    <div id="content">
        <p class="text">Lorem ipsum</p>
        <p class="text">Lorem ipsum</p>
        <p class="text">{{ message }}</p>
        <p class="text">Lorem ipsum</p>
        <p class="text">Lorem ipsum</p>
    </div>
</template>
```
这样的结构，在之前的vue中，会是下面这样的diff
* Diff \<div>
    * Diff props of \<div>
    * Diff children of \<div>
        * Diff \<p>
            * Diff props of \<p>
            * Diff children of \<p>
        * Repeat n times...
而实际上，这里只有个message会发生改变，其他内容都是动态的，但是这里会因为这一个message发生改变，导致整个模板进行一次diff，这样就做了很多不必要的消耗了

所以在vue3.0中，将vdom更行性能由与模板整体大小相关提升为与动态内容的数量相关，做到以下几点
* 将模板基于动态节点指令切割为嵌套的区块
* 每个区块内部的节点结构是固定的
* 每个区块只需要以一个Array追踪自身包含的动态节点
这样子，当一个数据发生改变时，我们只需要去diff它所在的那个最小区块即可，所以上面的代码结构在vue3.0之后，diff会是
* Diff \<p> textContent

这个区块的大小，甚至可以小到class的名字，因为实际上为了改变样式，更改className是非常常用的，所以如果className是动态内容，那么会将其划分为一个单独的区块，在进行diff算法时直接设置一个className即可，这样又对性能做到了很高的优化

这种优化甚至快了6倍（在2019Vue conf上做的一个测试做到了36ms->6ms）

## 2. 基于proxy的新数据监听系统
vue2使用的是Object.defineProperty的set/get
* 对象属性增添/删除

* 数组index/length更改

* Map,Set,WeakMap,WeakSet

* Classes

事实上基于proy的监听，是所谓的lazy by default，只有当一个数据被使用到的时候，我们才会真正地去监听这个数据，所以对于一个大规模数据的监听，在之前使用Object.defineProperty的监听中，我们对一个数据监听，不管这个数据是否使用到，都会对其进行监听

利用proxy减少组件实例初始化开销

每个vue组件都会代理它所包含的所有data,computed以及props，而这些代理都是通过Object.defineProperty实现的

在实际的实例化中，大量的Object.defineProperty实际上是一个昂贵的操作，而在vue3.0中，暴露给用户的this，实际上是真正组件实例的一个Proxy，所以我们实际上是在一个Proxy上获取数据，而在这个Proxy上获取数据时，我们再做一些内部的判断，这样就彻底避免了Object.defineProperty的使用

# 二、更小
1. 让代码结构和tree-shaking配合起来
* 内置组件（keep-alive，transition...）
* 指令的运行时helper（v-model，v-for...）
* 各种工具函数（asyncComponent，mixins...）

之前Vue的整个代码都是直接将整个Vue对象放进来，所以一些没用到的东西，也无法通过tree-shaking将其扔掉，所以这样就会造成即使Vue里面的一些功能我们没有用到，我们还是会将其放入到项目中，这就造成了不必要的内存消耗（即使这部分内容不大）

而在Vue3.0中，采取了ES module imports按需引入的方式。通过这种做法，如果我们没有用到一些内置组件时，在编译时就不会去加载这些内容。所以如果我们只用到所有Vue应用都会用到的一些核心功能，那我们就只会加载相应的这些核心功能相关的代码，而这一部分代码在所有无关的东西都使用tree-shaking处理后，只有大概在10kb左右

# 三、更易于维护
## vue3.0从Flow迁移到TypeScript，使用TypeScript重写了

## 内部模块解耦
降低源码阅读的难度

## 编译器重构
* 插件化设计
* 带位置信息的parser（source maps）
* 为更好的IDE工具链铺路

# 四、更好的多端渲染支持
vue2中需要自己去fork源码

vue3.0引入一个Custom Render API

# 五、新功能（新API）
## 响应式数据监听API
之前没有显式地将Vue的响应式功能暴露出来，而在vue3.0中可以通过import引入两个函数使用
```javascript
import { observable , effect } from 'vue'

const state = observable({
    count: 0
})

effect(()=>{
    console.log(`count is : ${state.count}`)
}) // count is : 0

state.count++ // count is : 1
```
上面引入了observable和effect，observable可以用来创建一个显式的响应式的对象，在effect中依赖的数据，就会被注册依赖，在之后当这个对象被改动到的时候，effect就会重新执行一遍

暴露这个API的主要目的，是为了轻松地实现跨组件的状态共享

## 轻松排查组件更新的触发原因
提供了renderTriggered API，每一次这个组件触发了更新的时候，可以在这个API中放入一个debugger，如下代码
```javascript
const Comp = {
    render(props){
        return h('div',props.count)
    },
    renderTriggered(event){
        debugger
    }
}
```
我们就可以在浏览器中直接地看到究竟是哪一行触发这个更新，同时event还会提供更具体的更新，包括了触发更新的对象，更新前的值和更新后的值，以及这个对象中被改动的属性，触发更新的操作是什么（比如set）

## 更好的typescript支持
使用了原生的classAPI，不再需要去依赖Vue Class Component这个库了(现在已经被撤销了)

采用了Function-based API
```javascript
const App = {
    setup(){
        const count = value(0)
        const plusOne = computed(() => count.value+1)
        const increment = () => { count.value++ }
        watch(() => console.log('mounted!))
        // 暴露给模板或渲染函数
        return { count }
    }
}
```
对比于Class API
* 更好的TypeScript类型推导支持
TypeScript对函数的参数和返回的内容类型支持是很好的，使用这个API，js和ts写出来是一模一样的，而且会给出类型补全
* 更灵活的逻辑复用能力
使用这个API可以做到
    * 没有命名空间冲突  
        我们将逻辑写在一个函数中，返回时可以使用解构赋值的方式获取里面的值，这样也能重写变量名，不会造成命名空间冲突
    * 数据来源清晰  
        数据可以清晰地从这个函数中获得
    * 没有额外的组件性能消耗
        相对于之前版本需要使用Mixins，高阶组件，slot这些方法，使用这个API只需要去写一个函数即可
* Tree-shaking友好
这里面的方法都是使用import引入的，即按需引入，如果我们不引入的话，就会被tree-shaking丢掉
* 代码更容易被压缩
在代码压缩的时候，一个函数内部的一些变量或者函数名会被压缩成一个字母，但是出于安全考虑，对象的属性是不会被压缩的，所以在代码压缩的时候就更容易了

对比React Hooks
* 同样的逻辑组合、复用能力
* 只调用一次
    * 符合js直觉
    * 没有闭包变量问题
    * 没有内存/GC压力
    * 不存在内联回调导致子组件永远更新的问题


支持TSX

## 更好的警告信息
* 组件堆栈包含函数式组件
* 可以直接在警告信息中查看组件的props
* 在更多的警告中提供组件堆栈信息

## (Experimental)Time Slicing Support
开启这个API会将JavaScript的操作切成一部分一部分，每执行一段时间，就会将主线程还给浏览器，让浏览器得以去监听用户事件并做出相应

在2019年的中国vue conf中，尤大提到了如果做的够快，可以缓解对时间分片的需求，所以这个功能最后是否会在vue3.0里出现，现在我也不清楚

## 关于IE的降级
在IE11中自动降级为旧的getter/setter机制，并对一些不支持的语法给予警告