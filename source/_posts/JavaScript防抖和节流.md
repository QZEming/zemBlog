---
title: JavaScript防抖和节流
abbrlink: 1ea78f4a
date: 2019-03-13 20:35:26 
tags: JavaScript
categories: JavaScript
---
防抖和节流都是优化的方式，经常听见却又不是特别清楚，查阅资料和写一些代码后写篇推文来记录一下
<!-- more -->
# 防抖
## 理解概念
防抖的简单理解为在短时间内多次触发一个操作（执行某个函数），只触发最后一次操作，这里的短时间是自己定义的。这样的好处在于短时间内多次触发同一函数可能只有一次是需要的，就如在搜索框输入内容时自动匹配相似的内容，如果按下每个按键就触发一次，未免有些低效率。

## 例子
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>防抖</title>
</head>
 
<body>
    <button id="norBtn">普通</button>
    <button id="antBtn">防抖</button>
    <script>
        window.onload = function() {
            let norBtn = document.getElementById("norBtn");
            norBtn.addEventListener("click", normal);
            let antBtn = document.getElementById("antBtn");
            antBtn.addEventListener("click", antiShake(anti, 500));
        }
 
        function antiShake(fn, time) {
            let timer = null;
            return function() {
                clearTimeout(timer);
                timer = setTimeout(() => {
                    fn.apply(this, arguments);
                }, time);
            }
        }
 
        function normal() {
            console.log("normal");
        }
 
        function anti() {
            console.log("anti");
        }
    </script>
</body>
 
</html>
```
这里的防抖实现是通过使用定时器，让函数隔一个相对短的时间后执行，如果在这个等待时间内重复点击该按钮，则会重置等待时间，最后只有最后一次触发是执行了函数的。

可以对上面两个按钮分别尝试短时间内多次点击，可以发现普通按钮在控制台打印的次数和点击的次数相同，而防抖按钮，如果每两次点击间隔小于指定时间，最后只会打印出一次。

## 应用
假设我们要做一个简单的二进制转换器，当在一个输入框内输入数字时输出对应的二进制数，我们要在数字输入完成后再进行输出，又不想通过点击来使其输出，那么就可以使用防抖来减少调用二进制转换函数的次数。
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>转换二进制</title>
</head>
 
<body>
    <input type="text" id="enter">
    <p id="out"></p>
    <script>
        window.onload = function() {
            let enter = document.getElementById('enter');
            enter.addEventListener('keydown', antiShake(toBinary, 500));
        }
 
        function toBinary() {
            let num = document.getElementById('enter').value;
            let arr = [];
            let out = document.getElementById('out');
            if (num == 1) {
                arr[0] = 1;
            } else {
                while (num != 0) {
                    let tmp = num % 2;
                    arr.push(tmp);
                    num = (num - tmp) / 2;
                }
            }
            out.innerHTML = arr.reverse().join("");
        }
 
        function antiShake(fn, time) {
            let timer = null;
            return function() {
                clearTimeout(timer);
                timer = setTimeout(() => {
                    fn.apply(this, arguments);
                }, time);
            }
        }
    </script>
</body>
 
</html>
```
在这里，如果输入够快，就不会去计算输入过程中数字的二进制转换，而是等到停下来之后过了一个自定义的时间才会进行二进制转换的计算，所以这样子可以减少计算所需。

虽然在这里看似没有减少多少，但在一些需要较为大型计算的地方，使用防抖就十分高效了，如上面说的在搜索引擎输入内容匹配对应近似的内容。

# 节流
## 理解概念
节流简单来说就是在一个固定的时间间隔内只会执行一次，如果在一个长时间内快速触发一个节流的事件，就像是以设置的固定时间为循环时间来执行一个循环事件。

## 例子
```html
<!DOCTYPE html>
<html lang="en">
 
<head>
    <meta charset="UTF-8">
    <title>节流</title>
</head>
 
<body>
    <button id="norBtn">普通</button>
    <button id="refBtn">节流</button>
    <script>
        window.onload = function() {
            let norBtn = document.getElementById('norBtn');
            let refBtn = document.getElementById('refBtn');
            norBtn.addEventListener('click', normal);
            refBtn.addEventListener('click', reduceFlow(ref, 1000));
        }
 
        function reduceFlow(fn, time) {
            let flag = true;
            return function() {
                if (flag) {
                    flag = false;
                    setTimeout(() => {
                        fn.apply(this, arguments);
                        flag = true;
                    }, time);
                }
            }
        }
 
        function normal() {
            console.log("normal");
        }
 
        function ref() {
            console.log("ref");
        }
    </script>
</body>
 
</html>
```