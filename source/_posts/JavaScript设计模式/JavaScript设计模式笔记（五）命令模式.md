---
title: JavaScript设计模式笔记（五）命令模式
abbrlink: da40a4a7
date: 2019-05-11 16:34:17
tags: JavaScript设计模式
categories: JavaScript
---
# 命令模式定义
命令指一个执行某件特定事件的指令，命令模式将发起命令者和执行命令者分离。对两者进行了松耦合的处理，体现了开放-封闭原则。
<!-- more -->
# JavaScript中的命令模式
假如一个公司的boss每天要命令公司某个员工去跑腿帮他拿报纸，不知道每天会叫哪一个员工，此时我们使用命令模式来将这个关系松耦合。在这个过程中，boss只需知道谁去跑腿，而不用关心怎么跑腿。
```javascript
let boss = {
    getNewspaper: function(staff) {
        console.log(`命令${staff.name}拿报纸`)
    }
}
 
let setCommend = function(staff, work) {
    work(staff);
}
 
let getNewspaperCommend = function(commender) {
    return commender.getNewspaper;
}
 
let staff1 = {
    name: 'xiaoming'
}
 
let staff2 = {
    name: 'xiaohong'
}
 
setCommend(staff1, getNewspaperCommend(boss)); // 命令xiaoming拿报纸
 
setCommend(staff2, getNewspaperCommend(boss)); // 命令xiaohong拿报纸
```
这里如果我们需要添加新的命令，可以直接添加一个新命令，并将该命令指向某个人去执行，不用修改之前的代码，而命令的执行也与命令接收者状态无关。

## 宏命令
宏命令是一组命令的集合，将几个有相关性的命令结合起来同时发出，减少了发起命令的次数

我们做另一个假设，现在天气很热，老板让办公室的人开空调，这个人需要做关门，关窗，使用遥控器打开空调几个操作，我们将这几个命令和起来作为一个宏命令。
```javascript
let boss = {
    closeDoor: function(staff) {
        console.log(`${staff.name}关门`)
    },
    closeWindow: function(staff) {
        console.log(`${staff.name}关窗`)
    },
    openAircondition: function(staff) {
        console.log(`${staff.name}打开空调`)
    }
}
 
let setCommend = function(staff, work) {
    work(staff);
}
 
let closeDoorCommend = function(commender) {
    return commender.closeDoor;
}
 
let closeWindowCommend = function(commender) {
    return commender.closeWindow;
}
 
let openAirconditionCommend = function(commender) {
    return commender.openAircondition;
}
 
let MacrosCommand = function() {
    return {
        commandsList: [],
        add: function(command) {
            this.commandsList.push(command)
        },
        execute: function(staff) {
            for (let i = 0, command; i < this.commandsList.length;) {
                setCommend(staff, this.commandsList[i++])
            }
        }
    }
}
 
let staff1 = {
    name: "xiaoming"
}
 
let macrosCommand = MacrosCommand();
 
macrosCommand.add(closeDoorCommend(boss))
macrosCommand.add(closeWindowCommend(boss))
macrosCommand.add(openAirconditionCommend(boss))
 
macrosCommand.execute(staff1)
```

# 命令模式的优点
使命令发起者和命令接收者松耦合，在编写代码时编写命令发起者和命令接收者可以分给不同的人编写，且不会相互影响，符合开放-闭合原则。

# 命令模式的缺点
相对于直接绑定命令发起者和命令接收者，代码量增加了，每种命令都要对应有一个方法来指定发起者中的哪一个命令。

参考自曾探的《JavaScript设计模式与开发实践》