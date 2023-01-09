---
title: 个人博客搭建笔记----hexo主题代码高亮
abbrlink: 5b269c26
date: 2020-02-21 01:22:13
tags: hexo
categories: 框架
---
我在之前的博客中自己搭建了一个hexo主题[simp]()，在文章添加中，默认文章里面代码是没有特定样式的，就如下面
<!-- more -->
![](https://img-blog.csdnimg.cn/20200221002345547.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
如果我们要添加样式，可以通过引入highlight.js来修改样式
在代码中添加这两行
```html
<link href="https://cdn.bootcss.com/highlight.js/8.0/styles/monokai_sublime.min.css" rel="stylesheet">
```
```html
<script src="https://cdn.bootcss.com/highlight.js/8.0/highlight.min.js"></script>
```
link标签加在head标签里面，script标签就加在\</body>上面一行，然后在script标签下面再加上
```html
<script>
    hljs.initHighlightingOnLoad();
</script>
```
网上看到的以为到这步就行了，捣鼓了好久没反应才发现_config.yml没有修改
在根目录的_config.yml里面highlight的hljs要改成true
```yaml
highlight:
  enable: true
  line_number: true
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: true
```