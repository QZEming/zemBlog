---
title: 瀑布流布局的介绍和实现（JavaScript和纯css）
tags: css布局
categories: css
abbrlink: f8b2c3e3
date: 2019-06-01 02:03:16
---
瀑布流布局在很多场景适用，最常见到的就是在商品网站，尤其是二手网站，由于用户上传的照片大小不一，如果压缩用户的照片或者切割照片可能会影响照片内容的呈现，如果不做任何处理，又会使整个页面看起来很乱，所以我们使用瀑布流布局来解决这个问题。
<!-- more -->
# 瀑布流布局简介
---
瀑布流布局，表现为参差不齐的多栏布局，随着页面滚动会不停呈现新内容的布局，瀑布流布局可用于无限滚动的页面，在每次滚动的时候将
# 瀑布流布局的实现
---
下面我们通过纯css和使用JavaScript两种方式来实现瀑布流布局
## JavaScript实现
---
### 思路
在编写代码前，我们先理清思路，使用JavaScript实现瀑布流，实际上就是在每次显示图片时，先找出当前几列高度最低的那一列，将下一张图片放到那一列下面，使用下面的图片来理解一下
![瀑布流示例](https://img-blog.csdnimg.cn/20190724223349672.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
如图是一个瀑布流布局的实现，我们标出图片显示的顺序
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724223905761.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
每次插入图片时，都会查看之前三列哪一列的高度比较低，将图片插到那一列的末尾。最后在瀑布流的末尾就不会有某一行的高度远超其他列，最多是一张图片的长度，使得在末尾各列之间的高度差较小。

所以我们应该写出两个功能函数，一个是判断高度返回列数，一个是插入图片。
### 代码
首先写出对应的html结构和css样式
```html
<body>
    <div id="container">
        <div class="col"></div>
        <div class="col"></div>
        <div class="col"></div>
    </div>
</body>
```

```css
* {
    margin: 0px;
    padding: 0px;
}

#container {
    width: 700px;
}

.col {
    float: left;
    margin: 0 auto;
    margin-right: 20px;
    width: 200px;
}

.col img {
    display: inline-block;
    width: 200px;
    margin-bottom: 20px;
}
```

接下来着手写两个功能函数
```javascript
// 查找哪个底部最低
function findLow(cols) {
    var index = 0;
    var minHeight = cols[0].offsetHeight;
    for (let i = 1; i < cols.length; i++) {
        if (cols[i].offsetHeight < minHeight) {
            index = i;
            minHeight = cols[i].offsetHeight;
        }
    }
    return index;
}
// 加载图片
function loadImg(imgArr, cols, i) {
    // 获取最低的列的索引
    let setColIndex = findLow(cols);
    // 创建图片节点
    let imgNode = document.createElement('img');
    imgNode.src = imgArr[i];
    // 添加节点
    cols[setColIndex].appendChild(imgNode);
    if (i < imgArr.length - 1) {
        setTimeout(function() {
            loadImg(imgArr, cols, i + 1)
        }, 100)
    }
}
```
这里因为是动态加载图片，而我又是使用一个数组来存储图片的路径，所以如果不使用异步的话，会使得高度无法正确被计算，我这里简单地使用setTimeout来实现异步，也可以使用ES6中的异步promise来实现，这里只是为了实现瀑布流我就简单地使用了计时器。
最后设置图片路径数组和调用方法
```javascript
window.onload = function() {
    // 模仿请求到的图片路径
    let imgArr = ['img/img1.jpg','img/img2.jpg', 'img/img3.jpg','img/img4.jpg', 'img/img5.jpg', 'img/img6.jpg', 'img/img7.jpg',  'img/img8.jpg',  'img/img9.jpg', 'img/img10.jpg', 'img/img11.jpg', 'img/img12.jpg', 'img/img13.jpg'];
    // 
    // 获取容器下的div
    let container = document.getElementById('container');
    var cols = container.getElementsByTagName('div');
    loadImg(imgArr, cols, 0);

}
```
这样就实现了瀑布流，如果要实现无限滚动的话，将图片添加在数组后面，在每次请求到路径后再次触发函数即可。
# 纯css实现
---
纯css的实现用到了**multi-column**，通过该样式可以简单快速地实现瀑布流。直接上代码
```html
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        #container {
            width: 1200px;
            /* 设置列数 */
            column-count: 4;
            /* 设置列宽 */
            column-width: 240px;
        }
        
        img {
            width: 200px;
        }
    </style>
</head>

<body>
    <div id="container">
        <img src="img/img1.jpg">
        <img src="img/img2.jpg">
        <img src="img/img3.jpg">
        <img src="img/img4.jpg">
        <img src="img/img5.jpg">
        <img src="img/img6.jpg">
        <img src="img/img7.jpg">
        <img src="img/img8.jpg">
        <img src="img/img9.jpg">
        <img src="img/img10.jpg">
        <img src="img/img11.jpg">
        <img src="img/img12.jpg">
        <img src="img/img13.jpg">
    </div>
</body>
```
虽然这样实现了瀑布流，但是存在一定的问题，当我们加载新图片时，图片的位置可能会发生变化，我们看看图片出现的顺序和之前使用JavaScript实现时的顺序区别
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190724233421189.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
可以看到在使用纯css实现时，图片是从上往下依次出现，然后再在另一列出现的，之所以能在视觉上实现瀑布流，是因为提前计算了出现的高度，如果使用缓加载，让图片一张一张出现，就会发现除了第一张图片，其他图片在显示新图片时，位置都会出现变化，所以使用纯css**multi-column**来实现瀑布流不适合与缓加载和无限滚动，只能是静态的瀑布流。