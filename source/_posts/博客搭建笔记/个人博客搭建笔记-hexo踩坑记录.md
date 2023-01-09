---
title: 个人博客搭建笔记----hexo踩坑记录
abbrlink: 65d0e2c0
date: 2020-02-22 16:36:29
tags: hexo
categories: 框架
---
## 1. post.excerpt摘要没有显示出来
<!-- more -->
需要在文章里面添加 
```html
<!-- more -->
```
该注释标签上面的内容即为摘要

## 2. post.excerpt摘要没有按html解析，按文本解析
使用的时候写成
```ejs
<%= post.excerpt %>
```
不该写成<%=，应该写成<%-
```ejs
<%- post.excerpt %>
```

## 3. hexo deploy发生错误  
执行hexo deploy出现报错  
ERROR Deployer not found: git  
执行
```
npm install --save hexo-deployer-git
```
再执行
hexo deploy就不会了

## 4. 使用site.posts[0]时出错
我在使用site.posts时，使用forEach的语法的时候，可以遍历文章内容，但是通过索引（site.posts[0]）获取文章内容的时候就失败了，而打印site.posts.length是有值的

把site.posts打印出来才发现，site.posts的结构是这样的。。。
```javascript
{
    data:[
        // ...实际的文章内容
    ],
    length:n, // n为文章的数量
    // ...
}
```
所以就改用site.posts.data[0]了