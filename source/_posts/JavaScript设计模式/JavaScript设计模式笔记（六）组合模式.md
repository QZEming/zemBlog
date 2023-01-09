---
title: JavaScript设计模式笔记（六）组合模式
abbrlink: 419518bc
date: 2019-05-11 18:15:17 
tags: JavaScript设计模式
categories: JavaScript
---
# 组合模式的定义
>组合模式将对象组合成树形结构，以表示“部分-整体”的层次结构。
>曾探《JavaScript设计模式与开发实践》
组合模式用于处理组合对象的请求。
<!-- more -->
# 更强大的宏命令
关于宏命令的实现，我在命令模式的笔记中已经提及了。

在组合模式中，可以对宏命令做更好的处理，将一些小的宏命令合成大的宏命令，下面引用《JavaScript设计模式与开发实践》的代码来说明这点。
```html
<html>
 
<body>
    <button id="button">按我</button>
    <script>
        var MacroCommand = function() {
            return {
                commandsList: [],
                add: function(command) {
                    this.commandsList.push(command)
                },
                execute: function() {
                    for (let i = 0, command; command = this.commandsList[i++];) {
                        command.execute();
                    }
                }
            }
        };
 
        var openAcCommand = {
            execute: function() {
                console.log('打开空调');
            }
        };
 
        var openTvCommand = {
            execute: function() {
                console.log('打开电视');
            }
        };
 
        var openSoundCommand = {
            execute: function() {
                console.log('打开音响');
            }
        };
 
        var macroCommand1 = MacroCommand();
        macroCommand1.add(openTvCommand);
        macroCommand1.add(openSoundCommand);
 
        var closeDoorCommand = {
            execute: function() {
                console.log('关门')
            }
        };
 
        var openPcCommand = {
            execute: function() {
                console.log('开电脑');
            }
        };
 
        var openQQCommand = {
            execute: function() {
                console.log('登录QQ');
            }
        };
 
        var macroCommand2 = MacroCommand();
        macroCommand2.add(closeDoorCommand);
        macroCommand2.add(openPcCommand);
        macroCommand2.add(openQQCommand);
 
        var macroCommand = MacroCommand();
        macroCommand.add(openAcCommand);
        macroCommand.add(macroCommand1);
        macroCommand.add(macroCommand2);
 
        var setCommand = (function(command) {
            document.getElementById('button').onclick = function() {
                command.execute();
            }
        })(macroCommand);
    </script>
</body>
 
</html>
```
结果
![](https://user-gold-cdn.xitu.io/2020/2/26/1707d55746ecdc67?w=897&h=261&f=png&s=37779)
这里的命令发出是以链式的方法发出的，一个大的宏命令告知其包含的命令执行execute方法，若其中还有宏命令，则继续告知，若为普通命令，则直接执行execute方法。

该例子的树形结构为
![](https://user-gold-cdn.xitu.io/2020/2/26/1707d55b7c037c43?w=1286&h=590&f=png&s=110878)

# 组合模式的注意点
1. 组合模式不是父子关系
组合模式是一种HAS-A的关系，而不是IS-A的关系。

* HAS-A：一种事物为另一种事物上的一个部件

* IS-A：一种事物为另一种事物中的一种分类

组合对象包含一组叶对象，方法的真正执行是在叶对象上，其他对象方法的执行实际上起到了一个命令传递的作用。

2. 命令方法的一致性
要保证对象能从根一直传递到叶，必须保证方法名是相同的，如上面例子中都是execute。

3. 一个对象不能有两个“父对象”
这里的“父对象”指将命令传给该对象的对象，如果一个对象有两个“父对象”，意味着在命令传递过程中，它的方法会被执行两次，假如它不是叶对象，其下的所有对象的方法都会被执行两次。

# 组合模式的优点
1. 对于树中的对象“平等”对待

不管树中的对象是组合对象或者叶节点对象，都执行相同的方法

2. 迭代树关系数据的便利性

只需要执行根节点的方法就可以对整棵树执行该方法，对迭代一棵树来说很方便

# 组合模式的缺点
1. 树结构中对象的相似性过高

为了保证树结构中的节点能被遍历，每个节点的方法大都是相同的，这样在看代码时很难区分对象，只有在执行的时候才能区分对象

2. 创建过多的对象

组合模式除了为每一个执行方法的对象构建一个对象，还要构建多个组合对象来构成最后完整的树结构，在这个过程中可能会构建过多的对象，而系统可能担负不起过多的对象。

---
参考自曾探的《JavaScript设计模式与开发实践》