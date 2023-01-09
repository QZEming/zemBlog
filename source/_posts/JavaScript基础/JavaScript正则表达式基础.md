---
title: JavaScript正则表达式基础
abbrlink: 56005dc6
date: 2019-02-22 19:15:59
tags: JavaScript
categories: JavaScript
---
正则表达式在匹配字符串上十分实用，这里就简单写出在JavaScript中的正则表达式实用基础。
<!-- more -->
首先介绍一下JavaScript中测试正则表达式是否匹配的方法test，该方法传入要匹配的字符串，如果匹配成功返回true，失败返回false。下面都会用这个方法来测试正则表达式的匹配。
```javascript
/a/.test('a');
// true
```
# 正则表达式的创建
可以实用字面量直接创建，也可以用构造函数创建。

使用字面量，将正则表达式内容写在两个/之间，而使用构造函数时，将正则表达式内容作为参数传入
```javascript
var r=/a/; // 用字面量创建
 
var r=new RegExp('a'); // 用构造函数创建
```
接下来介绍正则表达式内的字符含义

# 元字符
\t 水平制表符（tab）

\v 垂直制表符

\n 换行符

\r 回车符

\0 空字符

\f 换页符

# 与类有关的字符
[] 将里面的内容归为一类，匹配到其中任一个即为匹配成功
```javascript
/[abc]/.test('a');
// true
/[abc]/.test('b');
// true
/[abc]/.test('c');
// true
/[abc]/.test('d');
// false
```
^ 构建反向类，匹配到其中任一个为匹配失败
```javascript
/[^abc]/.test('a');
// false
/[^abc]/.test('b');
// false
/[^abc]/.test('c');
// false
/[^abc]/.test('d');
// true
```
- 构建范围类，即匹配连续的字符（数字上的连续和字母上的连续），可以连写
```javascript
/[1-9]/.test('1');
// true
/[1-9]/.test('12');
// true
/[1-9]/.test('120');
// true
/[a-z]/.test('ab');
// true
/[a-z]/.test('abc');
// true
/[A-Z]/.test('AB');
// true
/[A-Z]/.test('ABC');
// true
/[1-9a-z]/.test('1aa1');
// true  连写
```
# 预定义字符
下面的字符都是可以使用上面的字符来实现的，但是使用预定义字符更为方便

\d 数字字符

\D 非数字字符

\s 空白符

\S 非空白符

\w 单词字符（字母和数字下划线）

\W 非单词字符

# 边界匹配字符
^ 以。。。开始
```javascript
/^a/.test('1a')
// false
/^a/.test('a1')
// true
```
$ 以。。。结束
```javascript
/a$/.test('1a')
// true
/a$/.test('a1')
// false
```
\b 单词边界
```javascript
'there is an apple'.replace(/\b/g,'x');
// "xtherex xisx xanx xapplex"
 
// g是修饰符，在下文会提及，这里使用字符串的方法将单词边界替换为x
```
\B 非单词边界
```javascript
'there is an apple'.replace(/\B/g,'x');
// "txhxexrxe ixs axn axpxpxlxe"
```
# 量词
通过https://regexper.com/#%5Cn%0A这个网站来可视化看出次数

? 最多出现1次

\d?

![](https://user-gold-cdn.xitu.io/2020/2/27/1708500cb4ee5ade?w=218&h=110&f=png&s=7578)

上图的含义为，可以不经digit（数字）走过这条路，即数字出现0次的匹配，也可以经过一次

\+ 最少出现1次

\d+

![](https://user-gold-cdn.xitu.io/2020/2/27/1708501166f19d8b?w=190&h=96&f=png&s=7560)

上图的含义为经过一次或无限次循环经过，即匹配一个或者匹配无数个重复的

\* 出现任意次

\d*

![](https://user-gold-cdn.xitu.io/2020/2/27/17085014eb61e798?w=205&h=120&f=png&s=9073)

{n} 出现n次

\d{2}

![](https://user-gold-cdn.xitu.io/2020/2/27/17085035e2da04f1?w=204&h=132&f=png&s=9229)

上图含义为经过一次后还能再经过一次，即为两次

{n,m} 出现n到m次

\d{2,5}

![](https://user-gold-cdn.xitu.io/2020/2/27/1708503f1875c412?w=214&h=124&f=png&s=9965)

{n,} 至少出现n次

\d{2,}

![](https://user-gold-cdn.xitu.io/2020/2/27/17085043740665ed?w=215&h=116&f=png&s=9629)

# 贪婪模式
正则表达式在可以匹配多个也可以匹配少个的情况下，会默认尽可能地匹配多个
```javascript
'123456789'.replace(/\d{3,6}/g,'x')
// "xx"
```
上面代码中，既可以匹配三次，也可以匹配两次，最后匹配了两次（尽可能地匹配多个就导致了匹配次数的下降）

# 非贪婪模式
要尽可能匹配少个的话，只需要在两次后面加上?即可
```javascript
'123456789'.replace(/\d{3,6}?/g,'x')
// "xxx"
```
# 分组
() 可用于实现分组
```javascript
(123){5}
```


| 用于实现或，即选取分组内的任一个
```javascript
'abdacd'.replace(/a(b|c)d/g,'x')
// "xx"
```
$ 用于反向引用，引用分组的内容
```javascript
'2019-02-22'.replace(/(\d{4})-(\d{2})-(\d{2})/,'$2/$3/$1')
// "02/22/2019"
```
# 修饰符
g 全文搜索，添加后会一直匹配
```javascript
'aa'.replace(/a/,'x');
// "xa"
'aa'.replace(/a/g,'x');
// "xx"
```
i 忽略大小写，默认大小写敏感，添加忽略大小写
```javascript
'A'.replace(/a/,'x');
// "A"
'A'.replace(/a/i,'x');
// "x"
```
m 多行搜索

当有多行时，添加能匹配多行