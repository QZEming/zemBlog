---
title: react学习笔记（三）组件
tags:
  - react学习笔记
  - react
categories: react
abbrlink: e85972aa
date: 2019-09-26 12:14:15
---
组件将我们的页面拆分成不同的块，让我们更易于去维护这些内容
## React中的组件结构分类
---
React中的组件根据结构可分为函数组件和class组件
### 函数组件
顾名思义，函数组件即为函数形式的组件，举例如下
<!-- more -->
```javascript
function MyComponent(props){
	return <div>Hello {props.name}</div>
}
```
该组件名为MyComponent，通过props传入参数，使得组件可以在内部使用外部的数据，从形式上类似于平时写的函数，而这个props是只读不能修改的
###  class组件
函数组件以函数形式来写，那么class组件理所当然是使用class，使用ES6中class语法来创建一个组件即为class组件
```javascript
class MyComponent extends React.Component{
	render(){
		return <div>Hello {this.props.name}</div>
	}
}
```
这个组件与上面的函数组件是完全一致的，通过使用this.props来调用父组件传入的值，同样的这个this.props也是不可修改的
## 组件的渲染
组件的渲染和React元素的渲染一样，都是放在Render函数中，如下
```javascript
ReactDOM.render(
	<MyComponent name='xiaoming' />,
	document.getElementById('root')
)
```
这里将值为xiaoming的name属性传入组件MyComponent中，组件MyComponent将```<div>Hello xiaoming</div>```做为返回值返回
注意：组件**开头的字母**必须是**大写**的，否则会被视为是原生的DOM元素
## 组件的运用
---
### 组合组件
组件可以在组件中调用另一个组件，如下
```javascript
function Greet(props){
	return <h1>Hello {props.name}</div>
}
function Greets(props){
	return (<div>
				<Greet name='xiaoming' />
				<Greet name='xiaohong' />
			</div>
	)
}
ReactDOM.render(
	<Greets />,
	document.getElementById('root')
)	
```
这里在Greets组件中调用了两次Greet，然后再在ReactDOM.render()方法中调用该组件
如果我们不确定一个组件的子组件的内容的话，我们可以使用props.children来将这部分内容在使用时插入到该组件中
```javascript
class Greet extends React.Component{
    render(){
    return  <div className="greet">
            greet by {this.props.children}
        </div>
    }
}
ReactDOM.render(
    <Greet>
        <span className="greetWay">handshake</span>
    </Greet>,
    document.getElementById('root')
);
```
props.children会将标签内的东西作为其值传入组件内，除了这种方法外，我们也可以自己定义传入的内容和对应的属性
如下面的例子：
```javascript
function SplitPane(props) {
  return (
    <div className="SplitPane">
      <div className="SplitPane-left">
        {props.left}
      </div>
      <div className="SplitPane-right">
        {props.right}
      </div>
    </div>
  );
}

function App() {
  return (
    <SplitPane
      left={
        <Contacts />
      }
      right={
        <Chat />
      } />
  );
}
```
## state
---
React中可以通过修改组件中局部的state来修改状态
看看下面的例子
```javascript
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
state首先会在constructor中初始化，然后在组件中的其他地方，通过this.state来调用，使用this.setState来修改state的值
componentDidMount和componentWillUnmount为生命周期的方法：[react学习笔记(四) 组件的生命周期](https://blog.csdn.net/zemprogram/article/details/101395299)
### 使用state的注意点
1. 不要直接修改state
不要通过使用```this.state.name='xiaoming'```来修改state，否则即使state的数值改变了，相关的内容也不会被重新渲染，应该使用```this.setState({name:'xiaoming'})```
2. state的更新可能是**异步**的
出于**性能考虑**，React可能会把多个setState()调用合并成一个调用
因为可能会把多个setState()合并执行，所以我们无法预知哪个setState()一定会在另一个setState()前后执行（在同一时间调用的情况下），如下代码
```javascript
this.setState({
  counter: this.state.counter + this.props.increment,
});
```
我们无法判断在我们设置counter之前，increment的修改是否已经被调用过
我们可以通过将props和state做为参数传入来解决这个问题
```javascript
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```