---
title: Vue学习笔记---Vue单文件组件（SFC）规范
abbrlink: ebe1d9f6
date: 2020-02-24 00:59:29
tags: vue
categories: vue
---
.vue文件用于表示一个单一组件，其内使用类html语法，顶级标签有template,script,style和自定义的标签，一个完整简单的vue单文件如下
<!-- more -->
```html
<template>
    <div class="example">{{msg}}</div>
</template>

<script>
export default {
    data(){
        return{
            msg:"hello world"
        }
    }
}
</script>

<style>
.example{
    
}
</style>

<custom-label>自定义标签</custom-label>
```
## 模板\<template>
---
* template里面包含该组件的html结构，可以在该组件内使用其他组件，需要在script中导入并写在components里面
* 每个.vue文件只能有一个template标签
* \<template>标签下的根标签只能有一个，多于一个时在编译时会报错
* \<template>标签内的内容将被提取并传递给 vue-template-compiler 为字符串，预处理为 JavaScript 渲染函数，并最终注入到从\<script> 导出的组件中
## 脚本\<script>
---
* 每个.vue文件只会包含一个\<script>
* 该脚本会作为一个ES Module来执行，通过export显示指定输出，再通过import输入，在这里其他组件就是通过使用import导入，再在export default中的components中输出
* 在webpack中对.js配置的规则都会用于该标签上
* 脚本默认导出一个Vue.js的组件选项对象，也可以由Vue.extend()创建的扩展对象
## 样式\<style>
---
* 一个.vue文件可以有多个\<style>标签，由此我们就可以通过使用不同的lang属性，在同一个vue组件中同时使用less和sass（虽然一般不会这么做）
* 可以在同一个组件中混用scoped和module属性
* webpack中配置css文件的配置会应用在该标签上，如果设置了lang属性，为less时配置less标签的配置会应用在该标签上，sass也一样
## 自定义块
---
* 自定义块可以通过在webpack中配置，用一个loader来编译对应的自定义块，在rules中使用resourceQuery来配置对应的自定义块
配置\<custom>标签
```javascript
{
  module: {
    rules: [
      {
        resourceQuery: /blockType=custom/,
        loader: 'loader-to-use'
      }
    ]
  }
}
```
* 如果找到自定义块的匹配规则，则会被对应的loader处理，否则会被忽略
---
参考文档：[Vue单文件组件规范](https://vue-loader.vuejs.org/zh/spec.html)
[vue-loader文档](https://vue-loader.vuejs.org/zh/guide/custom-blocks.html#example)