---
title: JavaScript设计模式笔记（八）职责链模式
abbrlink: 9b553b97
date: 2019-05-13 20:00:15 
tags: JavaScript设计模式
categories: JavaScript
---
# 职责链模式的定义
职责链模式将多个对象用链的形式连接起来，当某个对象无法执行某个方法时，将请求沿链发送到下一个对象，交给下一个对象去处理，直到对象被处理或者到达链的末端，而链的末端一般是表示方法无法被执行或者标明某种特殊情况。
<!-- more -->
需要注意的是，这里在链上面传递的只是请求，每个对象判断能否完成该请求要做的内容，能则直接完成，否则继续传递请求。

# 简单的职责链模式例子
这里我们假设有这么一个例子，某个学校刚录取了一批学生，学生的资料都被打印出来，现在每个班主任老师要在资料中找出自己班里的学生并录入自己的系统里，假如有三个班级ABC，关于这个过程，我们可以使用if-else来简单地完成，这里我们模拟一个分发者的分发过程
```javascript
// 教师
var t1 = {
    students: [],
    class: 'A',
    record: function(student) {
        this.students.push(student);
        console.log(`t1 have record ${student.name}`);
    }
}
 
var t2 = {
    students: [],
    class: 'B',
    record: function(student) {
        this.students.push(student);
        console.log(`t2 have record ${student.name}`);
    }
}
 
var t3 = {
    students: [],
    class: 'C',
    record: function(student) {
        this.students.push(student);
        console.log(`t3 have record ${student.name}`);
    }
}
 
 
// 分发者
var distribute = {
    stuRecord: function(student) {
        if (student.class === t1.class) {
            t1.record(student);
        } else if (student.class === t2.class) {
            t2.record(student);
        } else if (student.class === t3.class) {
            t3.record(student);
        } else {
            console.log(`the class of ${student.name} may be wrong`);
        }
    }
}
 
// 学生
var s1 = {
    name: 'xiaoming',
    class: 'A'
}
 
var s2 = {
    name: 'xiaohong',
    class: 'C'
}
 
distribute.stuRecord(s1);
// t1 have record xiaoming
distribute.stuRecord(s2);
// t3 have record xiaohong
```
这里分发者需要知道所有老师对应的班级，判断为某个老师的班级后再交给那个老师，把录入学生信息当作请求要完成的操作，分发者就是请求方，老师是响应方，请求方和必须和每个响应方建立联系，违反了最少知识原则。

通过职责链模式，我们可以让请求方只和一个响应方建立联系，而响应方在链上起传递请求的作用。从例子来说，我先建立一条链：t1------>t2------->t3。分发者将所有资料拿给老师t1，告诉他“将自己班的学生录入系统”，这就是请求内容，t1老师判断是否为自己班的，如果是，执行请求的内容，如果不是，沿着链传递请求，直到链上该请求被处理。代码实现如下
```javascript
var t1 = {
    students: [],
    class: 'A',
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t1 have record ${student.name}`);
        } else {
            this.transmit(student)
        }
    },
    transmit: function(student) {
        t2.record(student)
    }
}
 
var t2 = {
    students: [],
    class: 'B',
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t2 have record ${student.name}`);
        } else {
            this.transmit(student)
        }
    },
    transmit: function(student) {
        t3.record(student)
    }
}
 
var t3 = {
    students: [],
    class: 'C',
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t3 have record ${student.name}`);
        } else {
            this.transmit(student)
        }
    },
    transmit: function(student) {
        console.log(`the class of ${student.name} may be wrong`);
    }
}
 
var distribute = {
    stuRecord: function(receiver, student) {
        receiver.record(student)
    }
}
 
distribute.stuRecord(t1, s1);
// t1 have record xiaoming
distribute.stuRecord(t1, s2);
// t3 have record xiaohong
```
在这里分发者只需要将资料全都交给t1老师就行，起到松耦合的作用，而如果分发者知道该学生不是A班的，可以直接将资料交给t2老师，这样就从t2开始沿链传递。

# 可拆分添加的链
上面代码的问题在于链的构建和顺序都在链上的对象中，这样当我们要改变链传递的顺序，或者在其中插入/删除一个节点，都会相当麻烦，所以我们可以给对象设置一个指向下一个链节点的属性，在外部设置这个属性
```javascript
var t1 = {
    students: [],
    class: 'A',
    nextPoint: {},
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t1 have record ${student.name}`);
        } else {
            this.transmit(student)
        }
    },
    transmit: function(student) {
        if (this.nextPoint !== {})
            this.nextPoint.record(student);
        else
            console.log(`the class of ${student.name} may be wrong`);
    }
}
 
var t2 = {
    students: [],
    class: 'B',
    nextPoint: {},
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t2 have record ${student.name}`);
        } else {
            this.transmit(student);
        }
    },
    transmit: function(student) {
        if (this.nextPoint !== {})
            this.nextPoint.record(student);
        else
            console.log(`the class of ${student.name} may be wrong`);
    }
}
 
var t3 = {
    students: [],
    class: 'C',
    nextPoint: {},
    record: function(student) {
        if (student.class === this.class) {
            this.students.push(student);
            console.log(`t3 have record ${student.name}`);
        } else {
            this.transmit(student);
        }
    },
    transmit: function(student) {
        if (this.nextPoint !== {})
            this.nextPoint.record(student);
        else
            console.log(`the class of ${student.name} may be wrong`);
    }
}
 
var distribute = {
    stuRecord: function(receiver, student) {
        receiver.record(student)
    }
}
 
 
var Chain = function() {
    return {
        head: {},
        setHead: function(head) { // 设置头节点
            this.head = head;
        },
        setNext: function(front, next) { // 设置连接节点
            front.nextPoint = next;
        },
        getNext:function(node){
            return node.nextPoint;
        },
        record: function(student) {
            this.head.record(student)
        }
    }
}
 
var chain = Chain();
 
chain.setHead(t1);
chain.setNext(t1, t2);
chain.setNext(t2, t3);
 
chain.record(s1);
// t1 have record xiaoming
chain.record(s2);
// t3 have record xiaohong
```
这里如果要把t2和t3的顺序进行转换，只需要set一下就行
```javascript
chain.setNext(t1,t3);
chain.setNext(t3,t2);
chain.setNext(t2,{});
```
而如果要在t1和t3之间添加一个t4，只需要将t4指向t1的指向，再将t1指向t4即可，和链表的插入是一样的
```javascript
chain.setNext(t4, t1.getNext());
chain.setNext(t1, t4);
```
删除的操作自然也是和链表的删除一样
```javascript
chain.setNext(t1, t4.getNext());
chain.setNext(t4, {});
```
# 职责链模式优点
1. 对请求方和响应方的解耦

职责链模式解耦了发起请求方和处理请求方，发起请求方不需要知道最终请求由谁处理，只要将请求发到链的头节点即可

2. 链式结构方便删除添加

正如数组和链表这两种数据结构进行比较，使用职责链模式会更便于在链中间进行插入和删除，而且不会修改到链上其他节点的代码

# 职责模式缺点
1. 需要在链的结尾进行处理

在每个职责链节点必须判断当前是否为链末端，若到达了末端，要进行报错或提示，不然会导致未知的错误

2. 过长的传递

如果每次处理请求的都是末端节点，那就意味着链上的其他对象只起到传递请求的作用，如果链的长度过长，那对性能的耗损是很大的，所以链的顺序也是很重要的，将比较有可能处理当前请求的节点放前面，可能性较低的放后面

---
参考自曾探的《JavaScript设计模式与开发实践》