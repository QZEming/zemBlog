---
title: react学习笔记（五）条件渲染
tags:
  - react学习笔记
  - react
categories: react
abbrlink: 996b09b9
date: 2019-09-26 00:34:52
---
在React中，我们有时只想渲染部分组件，而在某些状态发生改变时渲染另一部分组件，此时就要使用条件渲染
## if-else
我们可以通过if-else来判断状态值，从而渲染不同的组件
<!-- more -->
```javascript
function UserGreeting(props) {
  return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
  return <h1>Please sign up.</h1>;
}
function Greeting(props) {
  const isLoggedIn = props.isLoggedIn;
  if (isLoggedIn) {
    return <UserGreeting />;
  }
  return <GuestGreeting />;
}

ReactDOM.render(
  // Try changing to isLoggedIn={true}:
  <Greeting isLoggedIn={false} />,
  document.getElementById('root')
);
```
这里通过判断isLoggedIn的值来决定要渲染哪一个组件
## 与运算符&&
利用与运算符第一个为true时返回第二个的内容，第一个为false时返回false来进行条件渲染
```javascript
function Mailbox(props) {
  const unreadMessages = props.unreadMessages;
  return (
    <div>
      <h1>Hello!</h1>
      {unreadMessages.length > 0 &&
        <h2>
          You have {unreadMessages.length} unread messages.
        </h2>
      }
    </div>
  );
}

const messages = ['React', 'Re: React', 'Re:Re: React'];
ReactDOM.render(
  <Mailbox unreadMessages={messages} />,
  document.getElementById('root')
);
```
这里我们判断```unreadMessages.length > 0```是否为true，为true时返回后面的内容，为false时返回false，而正如我们所知，false在React中是不会被渲染出来的
## 三目运算符
三目运算符思路和if-else相似，也是判断条件然后渲染相应的内容
```javascript
render() {
  const isLoggedIn = this.state.isLoggedIn;
  return (
    <div>
      The user is <b>{isLoggedIn ? 'currently' : 'not'}</b> logged in.
    </div>
  );
}
```
