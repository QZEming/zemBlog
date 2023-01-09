---
title: css实现圆形进度条加载动画
abbrlink: d5dd0836
date: 2019-06-01 02:03:16 
tags: css动画
categories: css
---
这里我会把遇到这个需求时的实现过程和遇到的问题记录下来，如果只是要看最终实现结果可直接滑到底部看实现代码。

我们经常可以看到网上有一些圆形进度条跟随数字的变化慢慢变成一个圆，这个动画实际上可以通过纯css来实现
<!-- more -->
![](https://user-gold-cdn.xitu.io/2020/2/26/1707faa4d03f7845?w=110&h=114&f=png&s=9292)
通常情况下，我们实现这种结构只需要使用border-radius和padding就可以实现上图的样子，但是要实现动画，我们需要采取别的方式，这里要使用一个不常用的样式---clip（至少我在做这个动画前没使用过这个样式，不了解的可以上W3C clip样式看看）

首先我们要把中间的数字和动画分离开，中间的数字变化可以使用js的计数器来控制，动画则要由纯css来实现，可以通过after或者before来分离数字和圆弧。代码如下
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>test</title>
    <style>
        .time {
            display: inline-block;
            position: relative;
            width: 50px;
            height: 50px;
            line-height: 50px;
            text-align: center;
        }
        
        div::after {
            position: absolute;
            left: 0px;
            content: "";
            width: 50px;
            height: 50px;
            border: 1px solid #09f;
            border-radius: 50%;
        }
    </style>
</head>
 
<body>
    <div>
        <span class="time" id="time">10</span>
    </div>
    <script>
        var time = document.getElementById('time');
        count(0);
 
        function count(num) {
            time.innerHTML = num + '%';
            if (num < 100) {
                setTimeout(function() {
                    count(++num);
                }, 40)
            }
        }
    </script>
</body>
 
</html>
```
这样就把数字和圆分离了，然后接下来到我们的核心了，使用clip样式来切分我们要的部分

可是在切的时候我们发现，使用clip样式，我们能切的只能是一个矩形的内容块，切起来不是很方便，此时可以使用after和before来分别切分左半部分和右半部分。使用animation来实现动画，css代码修改如下
```css
.time {
    display: inline-block;
    position: relative;
    width: 50px;
    height: 50px;
    line-height: 50px;
    text-align: center;
}
 
div::after {
    opacity: 0;
    position: absolute;
    left: 0px;
    content: "";
    width: 50px;
    height: 50px;
    border: 1px solid #09f;
    border-radius: 50%;
    animation: clipCircleLeft 2s linear 2s 1 forwards;
}
 
div::before {
    position: absolute;
    left: 0px;
    content: "";
    width: 50px;
    height: 50px;
    border: 1px solid #09f;
    border-radius: 50%;
    animation: clipCircleRight 2s linear 0s 1 forwards;
}
 
@keyframes clipCircleLeft {
    0% {
        opacity: 1;
        clip: rect(52px 26px 52px 0px);
    }
    100% {
        opacity: 1;
        clip: rect(0px 26px 52px 0px);
    }
}
 
@keyframes clipCircleRight {
    0% {
        clip: rect(0px 52px 0px 26px);
    }
    100% {
        clip: rect(0px 52px 52px 26px);
    }
}
```
这里要注意的是，clip切分区域时，总的切分长度是元素的宽度加上边框宽度*2。

这样做之后就可以看到流畅的进度条效果了，但是这样要同时使用after和before，而且一旦时间改变，要改变的地方也挺多的，如果用less或者sass还好，可以使用变量存储时间，直接使用css写的时候就有点烦了。而且before有时还要给字体图标使用，所以我想到了使用transform:rotate()来达到只是用before就可以实现圆形进度条动画效果。修改后的css代码如下
```css
.time {
    display: inline-block;
    position: relative;
    width: 50px;
    height: 50px;
    line-height: 50px;
    text-align: center;
}
 
div::after {
    position: absolute;
    left: 0px;
    content: "";
    width: 50px;
    height: 50px;
    border: 1px solid #09f;
    border-radius: 50%;
    animation: clipCircle 4s linear 0s 1 forwards;
}
 
@keyframes clipCircle {
    0% {
        transform: rotateZ(90deg);
        clip: rect(0px 0px 52px 0px);
    }
    100% {
        transform: rotateZ(270deg);
        clip: rect(0px 52px 52px 0px);
    }
}
```
最终结果
同时使用before和after的例子
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>test</title>
    <style>
        .time {
            display: inline-block;
            position: relative;
            width: 50px;
            height: 50px;
            line-height: 50px;
            text-align: center;
        }
        
        div::after {
            opacity: 0;
            position: absolute;
            left: 0px;
            content: "";
            width: 50px;
            height: 50px;
            border: 1px solid #09f;
            border-radius: 50%;
            animation: clipCircleLeft 2s linear 2s 1 forwards;
        }
        
        div::before {
            position: absolute;
            left: 0px;
            content: "";
            width: 50px;
            height: 50px;
            border: 1px solid #09f;
            border-radius: 50%;
            animation: clipCircleRight 2s linear 0s 1 forwards;
        }
        
        @keyframes clipCircleLeft {
            0% {
                opacity: 1;
                clip: rect(52px 26px 52px 0px);
            }
            100% {
                opacity: 1;
                clip: rect(0px 26px 52px 0px);
            }
        }
        
        @keyframes clipCircleRight {
            0% {
                clip: rect(0px 52px 0px 26px);
            }
            100% {
                clip: rect(0px 52px 52px 26px);
            }
        }
    </style>
</head>
 
<body>
    <div>
        <span class="time" id="time">10</span>
    </div>
    <script>
        var time = document.getElementById('time');
        count(0);
 
        function count(num) {
            time.innerHTML = num + '%';
            if (num < 100) {
                setTimeout(function() {
                    count(++num);
                }, 40)
            }
        }
    </script>
</body>
 
</html>
```
只使用after和before其中一个的
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>test</title>
    <style>
        .time {
            display: inline-block;
            position: relative;
            width: 50px;
            height: 50px;
            line-height: 50px;
            text-align: center;
        }
        
        div::after {
            position: absolute;
            left: 0px;
            content: "";
            width: 50px;
            height: 50px;
            border: 1px solid #09f;
            border-radius: 50%;
            animation: clipCircle 4s linear 0s 1 forwards;
        }
        
        @keyframes clipCircle {
            0% {
                transform: rotateZ(90deg);
                clip: rect(0px 0px 52px 0px);
            }
            100% {
                transform: rotateZ(270deg);
                clip: rect(0px 52px 52px 0px);
            }
        }
    </style>
</head>
 
<body>
    <div>
        <span class="time" id="time">10</span>
    </div>
    <script>
        var time = document.getElementById('time');
        count(0);
 
        function count(num) {
            time.innerHTML = num + '%';
            if (num < 100) {
                setTimeout(function() {
                    count(++num);
                }, 40)
            }
        }
    </script>
</body>
 
</html>
```