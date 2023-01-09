---
title: react学习笔记（六）列表渲染
tags:
  - react学习笔记
  - react
categories: react
abbrlink: 54561d55
date: 2019-09-27 19:16:25
---
## 渲染多个组件
---
和Vue里面有v-for来让我们根据一个数组内的内容进行渲染不同，React中并没有提供这种在标签中的“属性”，要在React中渲染多个组件，我们通常使用JavaScript中的方法来实现
在Vue中，如果我们要将下面这个数组的内容
<!-- more -->
```javascript
const arr=[{name:'John',age:18},{name:'Alice',age:20}]
```
渲染成
```html
<div class="info">
	<p class="name">John</p>
	<p class="age">18</p>
</div>
<div class="info">
	<p class="name">Alice</p>
	<p class="age">20</p>
</div>
```
我们可以使用v-for来完成
```html
<div class="info" v-for="p in arr">
	<p class="name">{{p.name}}</p>
	<p class="age">{{p.age}}</p>
</div>
```
而在React中，我们可以使用map方法来完成
```javascript
const element = arr.map((p)=>
	<div className="info">
		<p className="name">{p.name}</p>
		<p className="age">{p.age}</p>
	</div>
)
```
上面的这种做法，需要对每一个要处理的数组使用一次map，我们将其封装为一个组件，可直接调用，传入数组即可
```javascript
class NumList extends React.Component{
    render(){
        return this.props.arr.map((p)=>
        <div className="info">
            <p className="name">{p.name}</p>
            <p className="age">{p.age}</p>
        </div>)
    }
}
const arr=[{name:'John',age:18},{name:'Alice',age:20}]
ReactDOM.render(
    <NumList arr={arr} />,
    document.getElementById('root')
);
```
然而，这种写法会触发如下警告
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190927185401944.png)
这个警告告诉我们每个子元素都应有一个唯一的key，所以我们要为其添加上唯一的key
在vue中，使用vue-cli搭建的项目默认配置下在使用v-for的时候也会要求加上key的警告，但是可以在eslint中去掉这个警告
```javascript
class NumList extends React.Component{
    render(){
        return this.props.arr.map((p,index)=>
        <div className="info" key={index}>
            <p className="name">{p.name}</p>
            <p className="age">{p.age}</p>
        </div>)
    }
}
```
通常我们会使用数据中原有的id来做为key，但是因为这里的数据没有id，所以我直接使用了index作为演示，但实际上如果在列表项目存在顺序变化的情况下不建议使用index作为key，这样可能会导致性能变差，还可能引起组件状态的问题
### 为什么要使用key
为什么要在列表组件渲染的时候使用key，我们要从组件如何渲染来看，当React递归DOM节点的子元素时会遍历两个子元素的列表，产生差异时会生成mutation
如下的代码
```html
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```
后面的代码只在前面代码的基础上在末尾添加了```<li>third</li>```，而其他部分没有改变，所以开销比较小，而反观下面这段代码
```html
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```
因为是在头部插入内容，而在没有索引的情况下，React一一比较，发现每个子元素都是不一样的，所以会生成多个mutation，这就对性能的开销会比较大了，所以我们使用key来解决这个问题，当子元素拥有key之后，React就会通过key来匹配原有的树和新生成的树的对应节点，这样不管在哪个位置添加新的子元素，开销都会比较小
```html
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
像这段代码，虽然是在头部插入，但是在比较子元素的时候，React发现key为2015和2016的子元素没有变化，只是添加了一个key为2014的子元素而已，所以只产生一个mutation，所以开销较小，这就是为什么需要使用key
### 为什么不建议使用index作为key
理由同上，如果我们使用index作为key的话，那当我们在头部插入一个新的内容的时候，后面的内容对应的key都会发生变化，那么自然就会产生多个mutation，所以开销自然也会增大，举上面我自己写的例子
```javascript
const arr=[{name:'John',age:18},{name:'Alice',age:20}]
class NumList extends React.Component{
    render(){
        return this.props.arr.map((p,index)=>
        <div className="info" key={index}>
            <p className="name">{p.name}</p>
            <p className="age">{p.age}</p>
        </div>)
    }
}
```
一旦我在arr前面加了一个新的内容，那么name为John对应的那个div，key就会从0变成1，而name为Alice的那个div，key会从1变为2，React在遍历时，就会发现key为0和key为1的都发生了变化，而且还多了一个key为2的元素，所以会产生多个mutation，造成比较大的开销