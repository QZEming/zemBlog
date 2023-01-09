---
title: react学习笔记（四）组件常用的生命周期方法
tags:
  - react学习笔记
  - react
categories: react
abbrlink: d5f46b73
date: 2019-09-26 00:34:42
---
React的每个组件都有自己的生命周期，我们可以利用这些生命周期对应的方法，在组件的不同时期触发特定的方法，来达到我们想要的效果
<!-- more -->
## 生命周期
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190926000112111.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
### constructor
constructor方法会在挂载时被首先调用，事实上，如果我们不需要去初始化state或者进行方法绑定，我们完全可以不为组件构造该函数
使用constructor时，要注意要先在开头写上super(props)，否则this.props在构造函数中可能会出现为定义的bug
在constructor中，我们一般只会去初始化state和为方法绑定this
```javascript
constructor(props) {
  super(props);
  // 设置state
  this.state = { counter: 0 };
  // 绑定this
  this.handleClick = this.handleClick.bind(this);
}
```
要注意的是，与在其他地方设置state相反，在constructor中，我们要使用this.state={}来设置state，而不是使用this.setState({})来设置
## componentDidMount
componentDidMount() 会在组件挂载后（插入 DOM 树中）立即调用，如果我们要在组件初始化时进行什么操作，就可以放在这里执行
可以在componentDidMount中使用setState来修改state的值，但是这样会触发额外的渲染，此渲染会发生在浏览器更新屏幕之前，所以用户不会发现其中发生了两次渲染，但是这种做法会导致性能问题，一般情况下在constructor中初始化好state即可
如果使用发布-订阅的设计模式，可以在这里添加订阅，定时器也可以在这里设置，但是要注意在componentWillUnmount中取消订阅和关闭定时器
## componentDidUpdate
componentDidUpdate() 会在更新后会被立即调用，首次渲染不会执行此方法。不管更新的内容是否会被渲染在页面上，都会调用这个方法，即使修改后内容相同也会调用该方法
```javascript
class MyComponent extends React.Component {
     constructor(props){
         super(props);
         this.state={
             number:1
         }
     }
     handleClick(num){
       this.setState({
           number:num
       })
     }
     componentDidUpdate(){
         console.log(1111)
     }
     render(){
         return <div>MyComponent{this.state.number}<button onClick={this.handleClick.bind(this,2)}>click</button></div>;
     }
 }
 ReactDOM.render(
       <MyComponent/>,
       document.getElementById('root')
 );
```
这里虽然没有将state中的number渲染出来，但是仍然会打印出111，多次点击，即使number已经变成2了，依然会一直打印出111
在componentDidUpdate中也可以使用setState修改state的值，但不能直接修改，至少要放在条件语句中，不然就会陷入死循环中，状态被改变->触发componentDidUpdate->触发其中的setState->状态又被改变->....
## componentWillUnmount
componentWillUnmount() 会在组件卸载及销毁之前直接调用，一般会在这里将设置的定时器关闭，取消订阅事件，取消网络请求等
在componentWillUnmount()不要试图调用setState去修改state，因为该组件即将被销毁，不会再重新渲染/
