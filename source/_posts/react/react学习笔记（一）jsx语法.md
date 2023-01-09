---
title: react学习笔记（一）jsx语法
tags:
  - react学习笔记
  - react
categories: react
abbrlink: b95a05b2
date: 2019-09-26 00:33:45
---
jsx是我们在react中编写组件的一种语法，实际上jsx只是JavaScript的一种语法糖
jsx让我们能更为方便地在JavaScript中书写node节点，通过使用html的格式，让我们更为直观地看到节点的结构
<!-- more -->
## jsx的几种简单用法
#### 1.直接创建一个node节点
```javascript
const element = <h1>Hellow word!</h1>;
```
这里将```<h1>Hellow word!</h1>```赋值给element，实际上是等同于将一个h1标签/节点赋值给element，而这个element可以直接放在render函数的return中
```javascript
  const element=<h1>Hellow World</h1>
  ReactDOM.render(
    element,
    document.getElementById('root')
  );
  ```
  页面结果：![在这里插入图片描述](https://img-blog.csdnimg.cn/201909241225597.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
  如果放入的是一个数组，那么会将数组展开显示
  ```javascript
  var arr=['hellow','world'];
  ReactDOM.render(
     <div>{arr}</div>,
     document.getElementById('root')
   );
  ```
  结果如图：
  ![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924123532432.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
  #### 2.在属性中使用表达式
  ```javascript
  class MyComponent extends React.Component {
    render(){
        return <div>{this.props.val}</div>;
    }
}
ReactDOM.render(
    <MyComponent val={1+2} />,
    document.getElementById('root')
);
```
在jsx中，可以在属性中通过花括号使用表达式，如上面的例子，在子组件MyComponent中添加val属性，然后计算1+2后才把结果赋值给该属性
此外，也可通过使用扩展运算符来将对象的属性赋值给标签
```javascript
class MyComponent extends React.Component {
    render(){
        return <div>{this.props.index}:{this.props.val}</div>;
    }
}
const obj={
    index:0,
    val:1
}
ReactDOM.render(
    <MyComponent {...obj} />,
    document.getElementById('root')
);
```
这里通过将obj对象的两个属性index,val赋值给子组件MyComponent，在子组件中通过this.props来调用对应的值
结果如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924172138511.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
tip:如果在子组件中属性没有赋值，那么默认为为true
如下代码，两行代码的结果是相同的
```javascript
<Mycomponent autoShow></Mycomponent>
<Mycomponent autoShow={true}></Mycomponent>
```
#### 3.将表达式做为子元素
```javascript
const element = <h1>Hello World</h1>
ReactDOM.render(
    <div>{element}</div>,
    document.getElementById('root')
);
```
这里将整个h1标签赋值给element，然后将整个element放到render出的div中
结果如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924180745613.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
在子元素中，布尔类型，undefined，null会被忽略，无法渲染出内容
下面的jsx表达式都相等
```javascript
<div />
<div></div>
<div>{true}</div>
<div>{false}</div>
<div>{null}</div>
<div>{undefined></div>
```
## jsx的优点
 1. 声明节点的时候更为直观
相对于我们使用createElement来创建节点，使用jsx显得更为直观，看看下面两段代码
```javascript
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);

const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
显然前者更为直观

 2. 代码动态创建界面的灵活性
通过使用jsx语法，我们可以将JavaScript插入到html标签中，可以使页面动态变化，增强了灵活性

3. 无须学习新的模板语言
前面也说了，jsx实际上就是JavaScript的语法糖，里面表示DOM节点的也是html的标签，所以使用jsx我们无序再去学习新的模板语法，上手很快

## jsx的约定
1. React认为小写的标签是原生的元素
诸如div，span这些标签，在React中被认为是原生的元素，即使我们自定义了标签，只要是小写的，就都会被认为是原生的元素

2. 大写字母开头的被认为是组件
对于大写字母开头诸如MyComponent这些标签，都会被React认为是组件，会被做为组件处理
结合上面两点，如果我们要写一个自定义组件，就要注意命名，如果为一个组件命名的时候开头是小写字母，那么就无法正常调用该组件
如下代码：
```javascript
import React from 'react';

// 错误！组件应该以大写字母开头：
function hello(props) {
  // 正确！这种 <div> 的使用是合法的，因为 div 是一个有效的 HTML 标签
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // 错误！React 会认为 <hello /> 是一个 HTML 标签，因为它没有以大写字母开头：
  return <hello toWhat="World" />;
}
```
上面的代码出自React的文档

3. jsx语法可以使用属性语法
属性语法诸如```<menu.li />```可以取出对象中的属性值做为标签名，举例如下

```javascript
class MyComponent extends React.Component {
    render(){
        return <div>MyComponent</div>;
    }
}
const menu={
    li:MyComponent
}
ReactDOM.render(
    <div><menu.li/></div>,
    document.getElementById('root')
);
```
结果如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190924182513355.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)

4. 当标签内无内容时，可以使用/>来闭合标签
如下代码
```javascript
ReactDOM.render(
    <MyComponent {...obj} />,
    document.getElementById('root')
);
```
因为MyComponent里面没有任何内容，所以直接使用/>闭合，就像XML语法一样
## jsx防止注入攻击
React DOM在渲染输入内容之前，会默认对其中的内容进行转义，在渲染前就会将所有的内容转为字符串，这样可以有效地防止 XSS（cross-site-scripting, 跨站脚本）攻击。