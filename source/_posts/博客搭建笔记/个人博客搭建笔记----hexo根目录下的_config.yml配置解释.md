---
title: 个人博客搭建笔记----hexo根目录下的_config.yml配置解释
tags: hexo
abbrlink: 52590a2f
date: 2020-02-13 15:23:31
categories: 框架
---

我们在使用hexo初始化一个项目的时候，在根目录下会有一个配置文件_config.yml，这个文件配置了所写博客里面的内容，我们从根目录的该文件来说明每个配置的作用
<!-- more -->
```yaml
# Hexo Configuration
## Docs: https://hexo.io/docs/configuration.html
## Source: https://github.com/hexojs/hexo/
```
Hexo Configuration下的两行表示hexo文档的相关文档，Docs指的是hexo的文档，Source指的是hexo的GitHub源码
# 网站
```yaml
# Site
title: Hexo
subtitle: ''
description: ''
keywords:
author: John Doe
language: en
timezone: ''
```
Site下面是网站相关的一些信息配置
**title**：网站的名字，会写在hexo generator命令生成后的public文件夹下的index.html文件的title标签里
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213014320663.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
我们也可以在默认的主题的首页左下角找到
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213000749491.png)
**subtitle**：网站的副标题，生成的时候默认为空，我们可以试着给他赋个值看它会出现在哪
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213020628542.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
在这个主题里，它写在背景图中间，在控制台找到该属性，找到这个属性对应的标签
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213020730263.png)
**description**：主要用于SEO，告诉搜索引擎一个关于站点的简单描述，在官方文档中建议在这段描述中增加自己的一些关键词，总之我写了之后是在网页中找不到我写的描述，可能是这个主题里面没把描述写出来，也可能我没看到
**keywords**：网站的关键词，使用半角逗号分隔开多个关键词，关键词对SEO的优化有一定的作用
**author**：顾名思义就是作者，这个也只是署个名而已
**language**：这里就写了网站使用的语言
**timezone**：网站对应的时区，一般情况下不用去刻意配置，会默认使用电脑的时区，但如果发布到其他地方的服务器，可能会使用当地服务器的时区，此时如果有需要用到本电脑的时区就要手动设置时区了，一般中国的时区可以设置为Asia/Shanghai
# 网址
```yaml
# URL
## 如果您的网站位于子目录中，请将url设置为'http://yoursite.com/child'，将root设置为'/child/'
url: http://yoursite.com
root: /
permalink: :year/:month/:day/:title/
permalink_defaults:
pretty_urls:
  trailing_index: true # 设置为false时会将末尾的index.html去掉
  trailing_html: true # 设置为false时会将末尾的.html去掉，对index.html无效
```
**url**：网站的网址
**root**：网站的根目录
**permalink**：文章的永久链接格式，可以在permalink这里配置，如默认中的配置，在hello world文章中就会是这样的url
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213111842930.png)
在使用hexo g生成的public文件夹里面，文件夹的层级也会按这个配置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213112831650.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
对应上面的年，月，天和标题，我们自己配置时，使用:加上要设置的对应属性，以/分隔，可以配置的有以下内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213112052730.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
这里要注意的是，如果使用id的话，在使用hexo clean后重新hexo g会使得id发生改变，将permalink改为:year/:month/:day/:title/:id/使用hexo g构造文件，然后使用hexo clean后重新使用hexo g构造，发现前后两次文章的id是不一样的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213113115775.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213113144368.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
除了上面的变量外，也可以使用Font-matter中的值，Font-matter是文章文件最上方以---分隔的区域，用于指定个别文件的变量，比如默认生成的hello-world.md中
```yaml
---
title: Hello World
---
```
可选的值有
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213113630227.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
**permalink_defaults**：为permalink中各项的默认值

# 目录
```yaml
# Directory
source_dir: source
public_dir: public
tag_dir: tags
archive_dir: archives
category_dir: categories
code_dir: downloads/code
i18n_dir: :lang
skip_render:
```
**source_dir**：存放hexo文章的文件夹，我们写的md文件都放在这个配置对应的文件夹下，默认值是source
**public_dir**：存放hexo g生成的文件，执行hexo g后根目录就会出现这个文件夹，默认值是public
**tag_dir**：按标签存放文章的目录，默认值是tags，如果我们给文章添加标签，使用hexo g时public文件夹下就会多出一个tags文件夹（根据我们的配置文件夹名不同），这里给hello-world.md添加一个tags
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213114549239.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
再使用hexo g，发现public文件夹下多出了一个tags文件夹
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213114612520.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
**archive_dir**：归档文件夹，存放归档文件，默认值为archives
**category_dir**：分类文件夹，按分类存放的文件，和上面的tags一样，只要在文章中的Font-matter添加了categories就会在使用hexo g构造时出现相应的文件夹
**code_dir**：Include code 文件夹，source_dir 下的子目录，默认值为downloads/code
**i18n_dir**：国际化（i18n）文件夹，默认值:lang
**skip_render**：跳过指定文件的渲染，匹配到的文件将会被不做改动复制到public文件夹下，如果路径对应的是我们的文章，那会直接忽略掉该文章，我们这样设置来忽略掉hello-world.md文件
```yaml
skip_render: "_posts/hello-world.md"
```
执行hexo clean清除之前渲染的内容后重新使用hexo g构造
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213120544735.png)
可见生成的文件夹只有一些基础的内容，没有和hello-world.md相关的东西
# 文章
```yaml
# Writing
new_post_name: :title.md 
default_layout: post
titlecase: false # Transform title into titlecase
external_link:
  enable: true # Open external links in new tab
  field: site # Apply to the whole site
  exclude: ''
filename_case: 0
render_drafts: false
post_asset_folder: false
relative_link: false
future: true
highlight:
  enable: true
  line_number: false
  auto_detect: false
  tab_replace: ''
  wrap: true
  hljs: false
```
**new_post_name**：新文章的默认名称，我们是使用hexo new来构造文章的，生成的文章名就是通过这里配置的，默认为:title.md，也就是标题.md
**default_layout**：预设布局，hexo new可以创建三种文件，post/帖子，draft/草稿，page/页面，这里设置默认值在使用hexo new就可以直接创建对应默认类型的文章，设置了post就等同于说在命令行执行hexo new \<name>和执行hexo new post \<name>是一样的，也可以设置为draft和page
**titlecase**：把标题转换为 title case
**external_link**
- enable：是否在新标签中打开链接
- fileds：external_link.enable的配置对当前网站（site）生效或仅对文章（post）生效，默认为site
- exclude：需要排除的域名

**filename_case**：把文件名称转换为小写(1)或者大写(2)，默认不转换(0)
**render_drafts**：是否渲染草稿文件，默认为false不渲染
**post_asset_folder**：是否启动资源文件夹，对于我们的网站，如果我们的文章里面有图片，我们可以在source文件夹下建立一个统一的images文件夹来存放图片，但是如果有的文章有很多的资源文件如图片，我们可以通过设置该配置为true，这样在source文件夹下创建文件的同时也会创建一个同名文件夹来存放相应的资源，比如我设置为true，然后执行hexo new newPost
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213142619638.png)
可以看到出现了一个newPost文件夹
**relative_link**：是否把链接改为与根目录的相对地址，默认为false
**future**：是否显示当前时间之后的文章，默认为true，如果设置为false的话，我们设置的日期如果是未来的日期的话，就不会显示
**highlight**：代码块的设置
- enable：开启代码高亮，默认为true
- auto_detect：如果未指定语言，就自动检测，默认为false
- line_number：显示代码的行数，默认为true
- tab_replace：用n个空格来代表tab键，如果值为空，则不会代替tab键
- wrap：是否将代码放在table标签里，默认为true
- hljs：是否对CSS类使用hljs-*前缀，默认为false

# 主页设置
```yaml
index_generator:
  path: ''
  per_page: 10
  order_by: -date
```
**index_generator**：主页相关的设置
- path：主页对应的路径，默认为''，即域名根目录就是主页的路径
- per_page：每页显示的帖子数，默认为10
- order_by：帖子的排序，默认为-date，即按日期倒序排

# 分类和标签
```yaml
# Category & Tag
default_category: uncategorized # 默认分类
category_map: # 分类别名
tag_map: # 标签别名
```

# 元数据元素
```yaml
# Metadata elements
## https://developer.mozilla.org/en-US/docs/Web/HTML/Element/meta
meta_generator: true
```
是否在页面开头插入下面的meta标签，默认为true
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213174854263.png)
# 日期/时间格式
```yaml
# Date / Time format
## Hexo uses Moment.js to parse and display date
## You can customize the date format as defined in
## http://momentjs.com/docs/#/displaying/format/
date_format: YYYY-MM-DD
time_format: HH:mm:ss
## Use post's date for updated date unless set in front-matter
use_date_for_updated: false
```
**date_format**：日期格式，默认为YYYY-MM-DD，即年月日
**time_format**：时间格式，默认为HH:mm:ss，即时分秒
**use_date_for_updated**：启用以后，如果 Front Matter 中没有指定 updated（文件更新日期），post.updated 将会使用 date 的值而不是文件的创建时间，默认值为true

# 分页
```yaml
# Pagination
## Set per_page to 0 to disable pagination
per_page: 10
pagination_dir: page
```
**per_page**：分页时每页的文章数，如果为0则不分页，默认为10
**pagination_dir**分页的目录，默认为page，对应于public文件夹下的archives文件夹下的page文件夹，如果只有一页是不会生成这个文件夹的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200213180200123.png)

# 包括或不包括目录和文件
```yaml
# Include / Exclude file(s)
## include:/exclude: options only apply to the 'source/' folder
include:
exclude:
ignore:
```
**include**：Hexo 默认会忽略隐藏文件和文件夹（包括名称以下划线和 . 开头的文件和文件夹，Hexo 的 _posts 和 _data 等目录除外）。通过设置此字段将使 Hexo 处理他们并将它们复制到 source 目录下。
**exclude**：Hexo 会忽略这些文件和目录
**ignore**：忽略的文件
要注意的是，这里要写入的是数组，而yaml的数组要用-值表示数组中一个元素或者直接采用js中数组的写法[]
```yaml
include:
  - ".nojekyll"
  # 包括 'source/css/_typing.css'
  - "css/_typing.css"
  # 包括 'source/_css/' 中的任何文件，但不包括子目录及其其中的文件。
  - "_css/*"
  # 包含 'source/_css/' 中的任何文件和子目录下的任何文件
  - "_css/**/*"
```

# 主题
```yaml
# Extensions
## Plugins: https://hexo.io/plugins/
## Themes: https://hexo.io/themes/
theme: landscape # 当前使用的主题名
```

# 部署
```yaml
# Deployment
## Docs: https://hexo.io/docs/deployment.html
deploy:
  type: ''
```
hexo提供了快速方便的一键部署功能hexo deploy，但我们至少需要在_config.yml文件中的deploy中至少配置一个type，比如
```yaml
deploy:
  type: git
```
此外还有几个配置
```yaml
deploy:
  type: git
  repo: <repository url> # 库地址
  branch: [branch] # 分支名称
  message: [message] # 自定义提交信息
```
---
参考文档：
[hexo官方中文文档](https://hexo.io/zh-cn/docs/configuration)
[hexo官方英文文档](https://hexo.io/docs/configuration.html)（有些中文文档没有的英文文档的才有）
[hexo fromt-matter](https://hexo.io/docs/front-matter)
[hexo 部署](https://hexo.io/zh-cn/docs/one-command-deployment.html)