---
title: react学习笔记（二）React元素
tags:
  - react学习笔记
  - react
categories: react
abbrlink: f642772f
date: 2019-09-26 12:14:06
---
## 元素
---
元素是React中的最小单元，一个简单的元素如下
<!-- more -->
```javascript
const element = <div id='root'>Hello World</div>
```
元素看似和DOM一样，实则是一个**创建开销很小**的对象，由React DOM来负责将其与DOM保持一致
React元素不是真正的DOM，被称为虚拟DOM(virtual DOM)，所以React元素无法调用DOM元素上原生的API
## 元素的编译
---
正如上面的例子，我们使用jsx语法来创建一个React元素，在编译过程中，jsx会被编译成d对```React.createElement()```的调用，注意不是document.createElement，上面的代码会被编译成：
```javascript
const element = React.createElement(
	'div',
	{id:'root'},
	'Hello World'
);
```
## 元素渲染为DOM
---
要将一个React元素渲染为DOM节点，可以使用render方法来做到，比如当我们要渲染一个DOM：
```html
<div id='root'>Hello world</div>
```
可以使用下面的方法
```javascript
const element = <div>Hello World</div>
ReactDOM.render(element,document.getElementById('root'))
```
在组件中通过在render方法中将元素return，传给父组件，最后在ReactDOM的render方法中调用组件
## 更新渲染的元素
---
React元素是不可变的元素，一旦创建即无法改变其子元素，如果我们要对其中的部分值做修改，可以通过重新创建对象，也可以通过修改组件内的状态值
```javascript
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);
```
上面的代码会每隔1秒就重新调用ReactDOM.render()，通过这种方式，不断地重创建，达到视觉上修改的作用
另一种方法见另一篇博客：[react学习笔记(三) 组件](https://blog.csdn.net/zemprogram/article/details/101392632)
## 部分更新
---
React在页面里面部分数据发生改变时，只会修改和其相关的内容，不会重新渲染其他内容，尽管是使用上面那种重新调用ReactDOM.render()的方法，也同样只会修改相关的内容。
就上面的例子来说，只有```new Date().toLocaleTimeString()```会发生改变，所以```<h1>Hello, world!</h1>```不会被重新渲染
实际上部分更新就是依赖了虚拟DOM元素，我们指导，DOM的更新依赖于数据的变化，而数据的改变会导致DOM树的重新渲染，而通过使用虚拟DOM，借由**diff算法**，比较更新前后虚拟DOM的差异，进行有**针对性的渲染**，从而提高性能
## React元素与DOM的属性差异
---
### checked
当input组件的type类型为checkbox(多选)或者radio(单选)时可以通过设置该属性来设置其是否被选中。
### className
React元素中的className与DOM中的class一致，即类选择器
### htmlFor
由于在JavaScript中for为保留字(for循环)，所以在React元素中使用labelFor来代替
更多内容：[React文档：DOM元素](https://react-1251415695.cos-website.ap-chengdu.myqcloud.com/docs/dom-elements.html)
## 事件处理
---
React元素的事件处理和DOM元素的事件处理类似，但是在语法上还是有些不同
### 语法
1. React的事件命名使用小驼峰式，而不是单纯的小写
如onclick在React元素中是onClick
2. 使用 JSX 语法时你需要传入一个函数作为事件处理函数，而不是一个字符串
如传统的html中
```html
<button onclick="fn()">click</button>
```
而在React元素中
```javascript
<button onClick={fn}></button>
```
### 阻止默认行为
在传统的JavaScript中，我们可以通过return false来阻止默认行为（如超链接的跳转），而在React元素中，我们不能通过这种方式来阻止默认行为，只能显式地使用preventDefault来阻止默认行为
React中的写法
```javascript
class MyCom extends React.Component{
	fn(e){
		e.preventDefault();
		console.log('click');
	}
	render(){
		return (
			<a href="#" onClick={fn}>click</a>
		)
	}
}
```
### this指向的问题
在JavaScript的class的方法中，this不会被默认绑定，如果我们要在class的方法中使用this的话，我们必须做一些处理
1. 手动绑定this
```javascript
class Toggle extends React.Component {
  constructor(props) {
    super(props);
    this.state = {isToggleOn: true};

    // 为handleClick方法绑定this
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => ({
      isToggleOn: !state.isToggleOn
    }));
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        {this.state.isToggleOn ? 'ON' : 'OFF'}
      </button>
    );
  }
}
```
这里通过在constructor中为方法绑定this，使我们在render函数中可以直接使用方法

2. 使用class fields语法
```javascript
class LoggingButton extends React.Component {
  handleClick = () => {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```
这里是利用了箭头函数的this指向特性来达成this的绑定，实际上这里的this已经被绑定了

3. 在回调中使用箭头函数
```javascript
class LoggingButton extends React.Component {
  handleClick() {
    console.log('this is:', this);
  }

  render() {
    return (
      <button onClick={(e) => this.handleClick(e)}>
        Click me
      </button>
    );
  }
}
```
这里同样利用箭头函数的this指向特性来达成绑定的目的
关于箭头函数：[ES6学习笔记（三）箭头函数](https://blog.csdn.net/zemprogram/article/details/86254967)
### 向事件处理程序传递参数
1. 在调用时使用箭头函数，在方法体中传入参数
2. 在调用方法时绑定参数
```javascript
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```