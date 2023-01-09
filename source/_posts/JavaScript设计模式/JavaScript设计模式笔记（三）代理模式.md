---
title: JavaScript设计模式笔记（三）代理模式
abbrlink: e25308b0
date: 2019-05-04 18:58:35
tags: JavaScript设计模式
categories: JavaScript
---
# 代理模式的理解
>为一个对象提供一个代用品或占用符，以便控制对它的访问  
>----《JavaScript设计模式与开发实践》
<!-- more -->
我把代理模式理解为为一个对象提供一个助手，这个助手可以帮助该对象完成某些事情/执行某些方法。如a想把信寄给b，如果他们在同一个地方，a可以直接将信拿给b，但现在a和b在两个比较远的地方，a因为某个原因走不开，需要有人帮他把信拿给b，这个时候就可以使用代理了。使用代码来模拟这个过程
```javascript
let a = {
    name: 'a',
    sendLetter: function(target) {
        target.receiveLetter(this)
    }
}
 
let b = {
    name: 'b',
    receiveLetter: function(from) {
        console.log(`${this.name} have received the letter from ${from.name}`)
    }
}
 
//无代理情况
a.sendLetter(b)
    // b have received the letter from a
 
//有代理情况
 
let proxy = { // 代理对象
    name: 'proxy',
    sendLetter: function() {
        b.receiveLetter(this)
    },
    receiveLetter: function(from) {
        console.log(`proxy have received the letter from ${from.name}`)
        this.sendLetter()
    }
}
 
a.sendLetter(proxy)
    //proxy have received the letter from a
    //b have received the letter from proxy
```
# 几种代理类型
代理模式有很多种类型，这里简单介绍几种代理类型
## 保护代理
保护代理指代理能帮助对象去处理一些特殊情况，比如上面的例子，b因为很忙，所以看到字数太多的信就不看了（这里我们假设为200个字），如果每次b都要接收后再去判断，那么他总要在忙碌的空隙去处理他最后要扔掉的信，这个对于b来说消耗了不必要的时间，而代理也白白花费了送信的时间，所以我们将这个判断放在代理中去实现。

修改后的代理对象如下
```javascript
let proxy = { // 代理对象
    name: 'proxy',
    sendLetter: function() {
        b.receiveLetter(this)
    },
    receiveLetter: function(from, letter) {
        console.log(`proxy have received the letter from ${from.name}`)
        if (letter.length > 200) { // 对收信方的"保护"
            console.log(`the letter from ${from.name} is too long`)
            console.log(`proxy have thrown the letter from ${from.name}`)
            return
        }
        this.sendLetter()
    }
}
 
// 使用带length属性的对象来模仿letter
a.sendLetter(proxy, { length: 100 })
    // proxy have received the letter from a
    // b have received the letter from proxy
 
a.sendLetter(proxy, { length: 300 })
    // proxy have received the letter from a
    // the letter from a is too long
    // proxy have thrown the letter from a
```
## 虚拟代理
在对象需要某事件/方法时才触发对该事件的调用/执行方法，我们将这个决定何时调用方法的任务交给代理，即为虚拟代理。我们对上面的例子做另一个假设，b是一个很情绪化的人，他只在心情好的时候会读信，心情不好的时候会直接把信扔了，且他的心情a无法知道，而代理却能判断他什么时候心情好，当b心情好时就把信给他，心情不好时就换个时间给他

模拟代码如下
```javascript
let b = {
    name: 'b',
    currentMood: function() { // 模拟b的情绪变化
        if (Math.random() > 0.5)
            return 'mad'
        return 'happy'
    },
    receiveLetter: function(from) {
        console.log(`${this.name} have received the letter from ${from.name}`)
    }
}
 
let proxy = { // 代理对象
    name: 'proxy',
    sendLetter: function() {
        b.receiveLetter(this)
    },
    receiveLetter: function(from) {
        console.log(`proxy have received the letter from ${from.name}`)
        this.listenMood()
    },
    listenMood: function() { // 监听b的心情
        if (b.currentMood() === 'mad') {
            console.log('b is at mad now')
            console.log('proxy will send letter after one second')
            setTimeout(() => {
                this.listenMood()
            }, 1000)
            return;
        }
        this.sendLetter()
    }
}
 
a.sendLetter(proxy)
```
## 缓存代理
缓存代理顾名思义就是对对象需要的内容进行判断，看之前是否已经获取/计算过该内容，如果已经获取/计算过了，就直接把缓存中的内容发给需要的对象，省去了获取/计算的过程，这对于远程的请求/复杂的计算有很重要的意义。

假如在某个页面中，我们经常要请求一些文件，这些文件都非常大，我们请求一次需要很长时间，但是有些文件我们可能会反复请求，此时我们使用代理来判断是否请求过该文件，若请求过，就把请求的内容从缓存中拿出来。
```javascript
let proxy = {
    cache: {}, //  存储请求的文件信息
    ajaxMes: function(url) {
        // ... 模拟文件获取，这里将文件获取的操作省略
        return message;
    },
    getMes: function(url) {
        if (!this.cache.url) {
            this.cache.url = this.ajaxMes(url) // 将获取的文件信息放入缓存中
        }
        return this.cache.url // 返回请求的信息
    }
}
 
// 假设有id位btn1,btn2的两个按钮，按下就会发起请求
let btn1 = documnet.getElementById('btn1')
let btn2 = document.getElementById('btn2')
 
 
// 这里的url1,url2为模拟url
btn1.onclick = function() {
    proxy.getMes(url1)
}
 
btn2.onclick = function() {
    proxy.getMes(url2)
}
```
在上面的代码中，当我们第二次按下按钮时，就会从缓存中获取内容

# 代理模式的优点
1.代理模式遵循了单一职责，对象只需要考虑要执行哪件事，而不需要去考虑这件事中要注意什么

2.代理模式的代理对象可以被多个对象调用，对于同一个业务我们只需要修改代理对象即可

ES6中的代理
在ES6中正式出现了代理proxy，用于截获对象的某些操作，详见[ES6学习笔记（十二）Proxy代理](https://blog.csdn.net/zemprogram/article/details/86555501)

---
参考自曾探的《JavaScript设计模式与开发实践》