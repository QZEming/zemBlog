---
title: 个人博客搭建笔记----hexo自定义主题搭建
abbrlink: 8bedbb68
date: 2020-02-20 16:07:46
tags: hexo
categories: 框架
---
使用hexo搭建过博客都知道，hexo里面有很多个主题可供我们选择，但是，如果那些主题里面没有我们自己想要的咋整，做为一名前端开发，这个时候当然是选择DIY
<!-- more -->
我这里采用的模板文件是ejs，所以下面的内容需要了解一些ejs的使用，我在文章过程中不会去详述相关的语法
# 新建自定义主题
在themes文件夹下创建一个自己的文件夹做为自己定义的主题，我这里写成simp  
## 主题目录
新建下列目录  
simp // 自建的主题目录
├── _config.yml     // 主题配置文件  
├── languages       // 语言配置文件  
├── layout          // 主要构造 html 的模板  
│   ├── index.ejs   // 主页模板  
│   ├── layout.ejs  // 布局模板  
│   └── post.ejs    // md 编译成 html 后的文件模板  
└── source          // 静态资源文件目录  
    ├── css         // css 样式目录  
    └── js          // JavaScript 脚本目录  
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC8yLzE1LzE3MDQ2ZmNiNmQ1ZTY0ZDU?x-oss-process=image/format,png)
## 相应配置修改
在根目录的_config.yml文件下将theme改为simp
```yaml
theme: simp
```
在主题目录下的_config.yml写入下面的配置，_config.yml不写内容的话在hexo s的时候会报错
```yaml
menu:
```
## 本地运行
在layout.ejs随便写入一点东西
```ejs
here is layout
```
在simp目录下使用hexo s启动服务器，如下图就成功了
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC8yLzE1LzE3MDQ3YTE3NzYyNjc3Y2Q?x-oss-process=image/format,png)
接下来尝试引入index.ejs和post.ejs的内容，在index.ejs和post.ejs里面分别写入内容  
index.ejs
```ejs
here is index
```
post.ejs
```ejs
here is post
```
layout.ejs
```html
<html>

<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8">
    <meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" name="viewport">
    <title>Home</title>
</head>

<body>
    <p>here is layout</p>
    <p><%- include("index.ejs") %></p>
    <p><%- include("post.ejs") %></p>
</body>

</html>
```
刷新页面，如图引入成功
![](https://imgconvert.csdnimg.cn/aHR0cHM6Ly91c2VyLWdvbGQtY2RuLnhpdHUuaW8vMjAyMC8yLzE1LzE3MDQ3YjA5MGRjZTA5ZDI?x-oss-process=image/format,png)

## 修改主题配置文件
这里是我自己的一些配置，在模板文件里面可以通过theme.\<prop>来调用这些属性对应的值  
_config.yml
```yaml
# nav
nav:
- HOME: /
- ABOUT: /about
- GITHUB: https://github.com/QZEming
- CSDN: https://blog.csdn.net/zemprogram/

# info
title: 前端小黑
avatar: /img/assets/logo.png
favicon: /img/assets/favicon.ico
```

## 加入各个布局模板文件
上面写了三个文件index.ejs，layout.ejs，post.ejs，接下来要写一些布局上的配置，比如header，nav，aside，footer  
这里就不详细说明每个文件的码字过程了，直接给出代码  
目录如下  
simp // 自建的主题目录  
├── _config.yml         // 主题配置文件  
├── languages           // 语言配置文件  
├── layout              // 主要构造 html 的模板  
│   ├── _partial        // 存放各种部分模板文件  
│       ├── aside.ejs   // 侧边栏的模板文件 
│       ├── footer.ejs  // 页面底部的模板文件  
│       ├── header.ejs  // 页面头部的模板文件  
│       ├── head.ejs    // head标签部分的模板文件  
│       ├── nav.ejs     // 导航栏的模板文件  
│       ├── main.ejs    // 页面主要部分的模板文件  
│   ├── index.ejs       // 主页模板  
│   ├── layout.ejs      // 布局模板  
│   └── post.ejs        // md 编译成 html 后的文件模板  
└── source              // 静态资源文件目录  
    ├── css             // css 样式目录  
    └── js              // JavaScript 脚本目录  


layout.ejs
```html
<!DOCTYPE html>
<html>
<%- partial('_partial/head') %>
    <body>
        <%- partial('_partial/header') %>
        <div class="mainContainer">
            <%- partial('_partial/aside') %>
            <%- partial('_partial/main') %>
        </div>
        <%- partial('_partial/footer') %>
    </body>
</html>
```
nav.ejs
```html
<nav class="topNav">
    <!-- theme.nav是写在主题下的_config.yml中的nav属性 -->
    <% theme.nav.forEach(r => { %>
        <a href=<%=Object.values(r)%>  class=<%=Object.values(r)=="/"+page.path? "chooseNav": "" %>>
            <%= Object.keys(r) %>
        </a>
    <% }) %>
    <img src="/img/assets/code.png" alt="avatar">
</nav>
```
aside.ejs
```html
<aside class="aside">
    here is aside
</aside>
```
head.ejs
```html
<head>
    <meta http-equiv="content-type" content="text/html; charset=utf-8">
    <meta content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=0" name="viewport">
    <!-- 配置favicon -->
    <%- favicon_tag(theme.favicon) %>
    <!-- 引入css中的style.css -->
    <%- css('css/style') %>
    <title>
        <%= theme.title %>
    </title>
</head>
```
header.ejs
```html
<header id="header" class="header">
    <div class="avatarContainer">
        <h1 class="blogName">
            <%= theme.title %>
        </h1>
    </div>
    <%- include('nav') %>
</header>
```
```html
<main class="main">
    here is main
</main>
```

这样就成功地写好了相关的基本布局了，在css文件中添加style.css文件就可以改变样式了，这里因为我用到了less，所以需要使用npm安装hexo-renderer-less来解析
```
npm install hexo-renderer-less --save
```
sass也有相应的解析npm包，做法和less
## 添加侧边栏内容
我这里添加了作者信息和标签目录，在_partial文件夹下新增下面的模板文件  
info.ejs  用于存放作者信息的模板文件
```html
<div class="info asideContainer">
    <div class="imgContainer">
        <img src=<%=theme.avatar %> alt="作者头像">
    </div>
    <div class="authorInfo">
        <p class="authorName"><%=config.author%></p>
        <p class="subtitle"><%=config.subtitle %></p>
        <p class="description"><%=config.description%></p>
        <p class="keywords">
            <% config.keywords.forEach(keyword => { %>
                <span class="keyword"><%=keyword%></span>
            <% }) %>
        </p>
    </div>
</div>
```
tags.ejs  用于存放标签目录的模板文件
```html
<div class="tags asideContainer">
    <h3 class="asideTitle">Tags</h3>
    <div class="asideContent">
        <% if (site.tags && site.tags.length){ %>
            <%- list_tags(site.tags, {
              class: 'article-tag',
            }) %>
        <% } %>
    </div>
</div>
```
修改aside.ejs模板文件
```html
<aside class="aside">
    <%- partial('info') %>
    <%- partial('tags') %>
</aside>
```
## 添加首页文章列表
在_partial文件夹下新增模板文件，模板文件代码如下
singleDirectory.ejs  文章目录下单篇文章内容模板文件
```html
<div class="singleDirectory">
    <h2 class="directoryTitle">
        <a href=<%=post.path %>>
            <%= post.title %>
        </a>
    </h2>
    <div class="directoryLabel">
        <span class="directoryDate icon-calendar"><%= post.date.format('YYYY-MM-D') %> </span>
        <!-- 文章标签 -->
        <div class="directoryTags icon-tag">
            <% if (post.tags && post.tags.length){ %>
                <%- list_tags(post.tags, {
                      show_count: false,
                      class: 'articleTag',
                      style: 'none',
                      separator:''
                    }) %>
                    <% } %>
        </div>
    </div>
    <!-- 文章摘要 -->
    <article class="directoryExcerpt">
        <%- post.excerpt %>
    </article>
</div>
```
directory.ejs  文章目录的模板文件
```html
<div class="singleDirectory">
    <h2 class="directoryTitle">
        <a href=<%=post.path %>>
            <%= post.title %>
        </a>
    </h2>
    <div class="directoryLabel">
        <span class="directoryDate icon-calendar"><%= post.date.format('YYYY-MM-D') %> </span>
        <div class="directoryTags icon-tag">
            <% if (post.tags && post.tags.length){ %>
                <%- list_tags(post.tags, {
                      show_count: false,
                      class: 'articleTag',
                      style: 'none',
                      separator:''
                    }) %>
                    <% } %>
        </div>
    </div>
    <article class="directoryExcerpt">
        <%- post.excerpt %>
    </article>
</div>
```
修改main.ejs模板文件，当存在文章的时候将文章目录模板文件写入
```html
<main class="main">
    <% if (page.posts) { %>
        <%- partial('directory') %>
    <% }%>
</main>
```

## 添加文章内容和文章目录
在_partial文件夹下新增模板文件，模板文件代码如下
article.ejs  文章模板文件
```html
<article class="articleContainer">
    <h1 class="articleTitle">
        <span><%=page.title%></span>
    </h1>
    <div class="articleContent">
        <%- page.content %>
    </div>
</article>
```
articleDirectory  文章目录模板文件
```html
<aside class="articleDirectory">
    <h2 class="asideTitle">目录</h2>
    <div class="asideContainer">
        <%- toc(page.content,{}) %>
    </div>
</aside>
```
修改main.ejs
```html
<main class="main">
    <% if (page.posts) { %>
        <%- partial('directory') %>
    <% }else{ %>
        <%- partial('article') %>
    <%}%>
</main>
```
## 添加返回顶部按钮
返回顶部按钮在页面超出当前页面高度时才出现，我这里通过改变按钮的class来控制是否出现  
首先，在source文件夹下的js文件夹下新建script.js文件，写入
```javascript
(function(){
    let backTop = document.getElementById('backTop')
    // 监听滚动
    window.addEventListener('scroll',function(){
        if(document.body.scrollTop>document.body.clientHeight&&backTop.className=="backTop icon-arrow-up hideBtn"){
            backTop.className = "backTop icon-arrow-up showBtn";
        }else if(document.body.scrollTop<=document.body.clientHeight&&backTop.className=="backTop icon-arrow-up showBtn"){
            backTop.className = "backTop icon-arrow-up hideBtn";
        }
    })
    // 为按钮绑定点击事件
    backTop.addEventListener('click',function(){
        function move(){
            // 当离顶部还有10的时候直接跳回
            if(document.body.scrollTop<10){
                document.body.scrollTop = 0;
                return false;
            // 否则每次回到当前离顶部距离的9/10处
            }else{
                document.body.scrollTop-=document.body.scrollTop/10
                return true;
            }
        }
        // 使用requestAnimationFrame做一个缓冲回到顶部
        requestAnimationFrame(function fn(){
            if(move()){
                console.log(1)
                requestAnimationFrame(fn)
            }   
        })
    })
})()
```
## 插入about页面
使用命令
```npm
hexo new page 'about'
```
或者直接在_post同级目录下创建about文件夹放入index.md随便写入东西
这样子这个主题就基本完成了，只要再添加一些css样式即可，相应的代码我也放在github上了，可以在[github](https://github.com/QZEming/hexo-theme-simp)上看看源码