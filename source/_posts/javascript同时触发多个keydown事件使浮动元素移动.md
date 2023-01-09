---
title: javascript同时触发多个keydown事件使浮动元素移动
abbrlink: d933ae69
date: 2018-11-28 21:53:33
tags: JavaScript
categories: JavaScript
---
之前在写坦克大战时就遇到一个问题，在移动坦克的同时还要发射炮弹。这个时候如果使用onkeydow事件的话，就会在发射炮弹时停止移动，没法同时触发两个keydown事件，为了解决这个问题，我通过在声明多个对象来记录按键的状态。

以移动为例子，假如要通过方向来移动某个在浏览器上的元素，首先给这个元素浮动（在使用canvas画布之类的可以使用里面的API改变元素的位置，这里通过将元素浮动来举例子）
```html
<head>
    <meta charset="UTF-8">
    <title>move</title>
    <style>
        #move {
            position: fixed;
            top: 0px;
            left: 0px;
        }
    </style>
</head>
 
<body>
    <div id="move">移动元素</div>
</body>
```
这样元素的html和css就写好了，接下来来写js部分，为了得到各个方向按键对应的键码，我们用下面的代码在控制台打印出对应的键码。
```javascript
window.οnkeydοwn=function(event){
    console.log(event.keyCode);
}
//上：38   下：40   左：37  右：39
```
然后用下面的代码移动元素，当按下对应按键的时候，元素每次移动1px。
```javascript
var move = document.getElementById('move');
    window.onkeydown = function(event) {
        if (event.keyCode == 38) { //上
            move.style.top = parseInt(window.getComputedStyle(move, null).top) - 1 + "px";
        }
        if (event.keyCode == 40) { //下
            move.style.top = parseInt(window.getComputedStyle(move, null).top) + 1 + "px";
        }
        if (event.keyCode == 37) { //左
            move.style.left = parseInt(window.getComputedStyle(move, null).left) - 1 + "px";
        }
        if (event.keyCode == 39) { //右
            move.style.left = parseInt(window.getComputedStyle(move, null).left) + 1 + "px";
        }
    }
```
但是这个时候我们就会发现，没法同时触发两个方向，即没法斜向移动，而且在改变方向时还会有明显的停顿，为了解决这个问题，我写了个对象来储存按键的状态。
```javascript
var codeType = {
        "up": false,
        "down": false,
        "left": false,
        "right": false
    }
```
我的思路是这样的，为每个方向设置状态，初始为false，通过keydown事件将其变为true，当状态为true时，执行对应方向的移动，同时要通过keyup事件使其变回false，否则其会不停地移动。将移动地方法放在一个计时器里，就能实现元素的斜向移动了。完整代码如下：
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>Document</title>
    <style>
        #move {
            position: fixed;
            top: 0px;
            left: 0px;
        }
    </style>
</head>
 
<body>
    <div id="move">移动元素</div>
</body>
<script>
    var move = document.getElementById('move');
    var codeType = {
        "up": false,
        "down": false,
        "left": false,
        "right": false
    }
    setInterval(Move, 40);
    window.onkeydown = function(event) {
        if (event.keyCode == 38) { //上
            codeType.up = true;
        }
        if (event.keyCode == 40) { //下
            codeType.down = true;
        }
        if (event.keyCode == 37) { //左
            codeType.left = true;
        }
        if (event.keyCode == 39) { //右
            codeType.right = true;
        }
    }
    window.onkeyup = function(event) {
        if (event.keyCode == 38) { //上
            codeType.up = false;
        }
        if (event.keyCode == 40) { //下
            codeType.down = false;
        }
        if (event.keyCode == 37) { //左
            codeType.left = false;
        }
        if (event.keyCode == 39) { //右
            codeType.right = false;
        }
    }
 
    function Move() {
        if (codeType.up) { //上
            move.style.top = parseInt(window.getComputedStyle(move, null).top) - 1 + "px";
        }
        if (codeType.down) { //下
            move.style.top = parseInt(window.getComputedStyle(move, null).top) + 1 + "px";
        }
        if (codeType.left) { //左
            move.style.left = parseInt(window.getComputedStyle(move, null).left) - 1 + "px";
        }
        if (codeType.right) { //右
            move.style.left = parseInt(window.getComputedStyle(move, null).left) + 1 + "px";
        }
    }
</script>
 
</html>
```