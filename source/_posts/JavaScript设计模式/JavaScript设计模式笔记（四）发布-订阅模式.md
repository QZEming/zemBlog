---
title: JavaScript设计模式笔记（四）发布-订阅模式
abbrlink: e5831f8b
date: 2019-05-06 20:05:22
tags: JavaScript设计模式
categories: JavaScript
---
# 发布订阅模式的理解
发布-订阅模式又叫观察者模式，定义：
> 对象间的一种一对多的依赖关系，当一个对象的状态发生改变时，所有依赖于它的对象都得到通知。
> ----《JavaScript设计模式与开发实践》
<!-- more -->
一对多的关系中，一的是发布方，多的是订阅方，多的那方的对象向发布方发起订阅的动作，表明当发布方某些状态发生改变时，要求发布方向他们发送相应的信息。我们举个常见的例子，微信里面的公众号和关注公众号的人，公众号就是发布方，而关注的人就是订阅方，改变的状态即是公众号的推文。每当公众号发布一篇推文（状态改变），就会发消息给关注的人，提醒他们发布方的状态已经发生了改变，我们下面用代码模拟这个过程：

我们理清一下模式建立的步骤

1.建立一个订阅方对象，设置好接收发布方发来信息的方法，设置名字标识以区分订阅方

2.建立发布方对象，设置存放订阅方的数组，添加订阅方的方法以及发布信息的方法

3.建立订阅方实例和发布方实例

4.将订阅方添加到发布方的存储数组中

5.发布方发布信息

代码如下：
```javascript
var publisher = function() {
    return {
        currentList: [],
        add: function(person) {
            this.currentList.push(person)
        },
        publish: function() {
            let currentList = this.currentList
            for (let i = 0; i < currentList.length; i++) { // 遍历订阅者的数组
                currentList[i].receiveText()
            }
        }
    }
}
 
var subscriber = function(name) {
    return {
        name: name,
        receiveText: function() {
            console.log(`${this.name} has received the message`)
        }
    }
}
 
var p1 = publisher()
var jack = subscriber('jack')
var Marry = subscriber('Marry')
// 订阅公众号
p1.add(jack)
p1.add(Marry)
// 公众号发布推文
p1.publish()
// jack has received the message
// Marry has received the message
```
上面的例子是订阅-发布模式的一种实现，实际上，订阅方和发布方的关系可能会在某个时刻断裂，所以订阅-发布模式中我们还应添加删除订阅方的方法，从例子来说，就是用户取消关注公众号。要实现此功能，只要给发布方添加一个remove方法即可，代码如下：
```javascript
var publisher = function() {
    return {
        currentList: [],
        add: function(person) {
            this.currentList.push(person)
        },
        publish: function() {
            let currentList = this.currentList
            for (let i = 0; i < currentList.length; i++) {
                currentList[i].receiveText()
            }
        },
        remove: function(person) {
            let currentList = this.currentList
            for (let i = 0; i < currentList.length; i++) {
                if (currentList[i] === person)
                    currentList.splice(i, 1)
            }
        }
    }
}
 
var p1 = publisher()
var jack = subscriber('jack')
var Marry = subscriber('Marry')
// 订阅公众号
p1.add(jack)
p1.add(Marry)
// 公众号发布推文
p1.publish()
// jack has received the message
// Marry has received the message
p1.remove(jack)
 
p1.publish()
// Marry has received the message
```
这样子就算完成了一个较为完整的简单订阅-发布模式。

# 发布-订阅模式的优点
使用发布-订阅模式，可以在时间和对象上解耦，写出松耦合的代码。时间上很好理解，在发布方和订阅方的关系建立后，只有在发布方的状态发生改变时，才会触发订阅方的一些事件，这样的操作可以算得上是异步操作了。而对象间的解耦是把对象间紧密的联系弱化，在上面的例子中，用户方和公众号方本都应存储双方的信息，才能在公众号发表推文时将信息发给用户，而在订阅-发布模式中，只要将用户方的部分信息放在公众号方处，就可实现我们要的效果。这种信息我觉得可以看作是类似指针，指向发布方对象，而发布方处并没有标记出这段关系。

# 发布-订阅模式的缺点
发布-订阅模式自然也有缺陷，简单地说，假如建立了发布-订阅模式，而发布方自始至终都没有改变状态，那这个关系建立就毫无意义，而且订阅方会始终存在内存中。而且如果过多使用发布-订阅模式，会使得对象与对象之间的必要联系被过度弱化，当多个发布方和一个订阅方关联时，订阅方方很难看出自己到底订阅了哪几个发布方，要从发布方的订阅方缓存中去查找，而这毫无疑问是很麻烦的。

---
参考自曾探的《JavaScript设计模式与开发实践》