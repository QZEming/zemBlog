---
title: 个人博客搭建笔记----hexo部署到github上及踩坑记录
abbrlink: fa2df26d
date: 2020-02-15 01:54:10
tags: hexo
categories: 框架
---
在我们通过修改_config.yml配置文件，选择自己喜欢的主题和做一些修改后，接下来就是要把我们自己的博客部署上去了，这里以部署到github为例
<!-- more -->
首先在github上新建一个仓库，这里因为我已经建好了这个仓库所以会有重名的报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215014528247.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
然后安装hexo-deployer-git用于部署
```npm
npm install hexo-deployer-git --save
```
接下来修改以下项目根目录下的_config.yml配置文件中的deploy
这里的reop就是刚才新建的仓库的git链接	
```yaml
deploy:
  type: git
  repo: https://github.com/QZEming/zemBlog.git
```
配置完成后执行
```npm
hexo d
```
查看github仓库中的内容，发现已经把内容放到仓库中了
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215014209190.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
到settings下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200214162618183.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
找到
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215014332181.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
这个网址就是我们的博客地址
访问该地址发现部署完成
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200215014440140.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
# 踩坑记录
访问的时候你发现主题失效？？？
最大可能的原因是在_config.yml里面配置出现错误
如果你的项目要在github给的网址下的子目录的话，那就要对root进行额外的配置
比如我的项目地址变成了https://qzeming.github.io/下的qzeming.github.io的话，就要配置成
```yaml
url: https://qzeming.github.io/
root: /qzeming.github.io/
```	

然后删除根目录下的 .deploy_git 
重新执行
```npm
hexo clean
hexo deploy
```
应该就可以了
