---
title: ES7，ES8，ES9，ES10新特性笔记
tags: ES6+
categories: JavaScript
abbrlink: 5397352a
date: 2020-01-20 17:42:31
---
JavaScript到现在更新了很多次，最大的一次变动是ES6，也因此我们经常会在面试的时候看到需要掌握ES6，而实际上，ES6已经是2015年的事情了，现在已经到了ES2019即是ES10，即将ES2020就出来了，所以有必要将这些内容整理一下了
 <!-- more -->
# ES7
---
## 1. 求幂运算符（**）  
求幂运算符功能与Math.pow()功能一样，但写起来更为便捷

## 2. Array.prototype.includes()  
增加了一个判断数组内有无某个元素的方法，传入两个参数，第一个参数为要找的值，第二个参数为开始找的索引，返回布尔值

如果忽略第二个参数，则默认从第一个开始找
```javascript
[1,2,3].includes(1); // true
```
如果忽略两个参数，则从第一个开始找undefined
```javascript
[undefined].includes(); // true
```
includes只能判断一些简单值，无法直接判断对象是否存在
```javascript
[{a:1}].includes({a:1}) // false
```
但如果赋值给变量，是可以判断的
```javascript
var a={foo:1};
[a].includes(a);
```
实际上就是因为在方法内部的判断使用了===

和原有的方法indexOf相比，includes直接返回布尔值，如果只是要看数组内是否有该值，使用includes更合适  
此外，includes和indexOf在判断NaN的时候存在差异，虽然NaN===NaN返回false，但是includes可以判断NaN的存在，而indexOf不行  
```javascript
[NaN].includes(NaN); // true
[NaN].indexOf(NaN); // -1

```
# ES8
---
## 1. Object.values  
这个方法接收一个对象参数，将该对象中属性及其值放到一个数组中返回  
```javascript
Object.values({foo:1,bar:2}) // [1,2]
```
若传入的是一个数组，则直接返回这个数组，若数组有其他属性，则会将这个属性的值添加到返回的数组中去  
```javascript
var arr=[1,2,3];
Object.values(arr); // [1,2,3]
arr.val = 4;
Object.values(arr); // [1,2,3,4]
```
## 2. Object.entries  
这个方法接收一个对象参数，返回一个二维数组，数组中的每个成员数组的第一项为属性名，第二项为属性对应的值
```javascript
Object.entries({foo:1,bar:2}) // [["foo",1],["bar",2]]
```
若传入的是一个数组，则属性名变为下标的值
```javascript
Object.entries([1,2,3]) // [["0",1],["1",2],["2",3]]
```

## 3. padStart  
该方法为字符串填充方法，传入两个参数，第一个参数为要填充到的字符串长度，第二个参数为要用来填充的字符串，将第二个参数的字符串填充到字符串的开头    
```javascript
"123".padStart(5,'m') // "mm123"
"123".padStart(10,'mab') // "mabmabm123"
```
若传入的第一个参数小于原字符串的长度，则直接返回原来的字符串

## 4. padEnd  
该方法为字符串填充方法，用法和padStart一样，但是填充在字符串的末尾
```javascript
"123".padEnd(5,'m') // "123mm"
"123".padEnd(10,'mab') // "123mabmabm"
```

## 5. Object.getOwnPropertyDescriptors  
该方法可以返回对象的描述属性
```javascript
Object.getOwnPropertyDescriptors({foo:1})
/*
{
    configurable: true
    enumerable: true
    value: 1
    writable: true
    __proto__: Object
    __proto__: Object
}
*/
```

## 6. 函数参数列表和调用时的参数的尾逗号
```javascript
function add(a,
b){
    return a+b;
}

add(1,
2)
```
上面的这种写法，在ES5中是会报错的，但在ES8中是可以使用的

## 7. async  
[ES6学习笔记（十六）asnyc函数](https://blog.csdn.net/zemprogram/article/details/86596178)
# ES9
---
## 1. 关于rest和spread属性的扩展  
在ES6中，我们可以通过...来传入不定数的参数，也可以用来给数组赋值
```javascript
function sum(...rest){
    let sum = 0;
    for(let i = 0;i<rest.length;i++){
        sum+=rest[i];
    }
    return sum;
}
sum(1,2,3,4); // 10

[a,b,...c]=[1,2,3,4,5]; //a=1 b=2 c=[3,4,5]
```
在ES9中，...可以用来给对象解构赋值
```javascript
{a,b,...other}={a:1,b:2,c:3,d:4,e:5}; // a=1 b=2 other={c:3,d:4,e:5}
```

## 2. 异步迭代
for-await-of

## 3. promise.prototype.finally  
我们知道promise可以变为fulfilled状态和rejected状态，相应会去执行then方法和catch方法，而finally方法则是无论成功或是失败都会执行
```javascript
let a = 1;
function judge(resolve,reject){
    if(a>0){
        resolve(a);
    }else{
        reject(a);
    }
}
new Promise(judge).then(val=>{
    console.log('then',val);
}).catch(err=>{
    console.log('catch',err);
}).finally(()=>{
    console.log('finally');
})
// then 1
// finally
a=-1;
new Promise(judge).then(val=>{
    console.log('then',val);
}).catch(err=>{
    console.log('catch',err);
}).finally(()=>{
    console.log('finally');
})
// catch -1
// finally
```

## 4. 后行断言 
在之前的JavaScript中，只有先行断言，而在ES2018中引入了后行断言

### 先行断言
/x(?=y)/  匹配在y前面的x

/x(?!y)/  匹配x不在y前面的x

/\d(?=%)/.exec('1% 2') // 匹配后面有%的数字
// ["1"]
/\d(?!%)/.exec('1% 2') // 匹配后面没%的数字
// ["2"]
### 后行断言
/(?<=y)x/ 匹配在y后面的x

/(?<!y)x/ 匹配不在y后面的x
```javascript
/(?<=\$)\d/.exec('$1 2')
// ["1"]
/(?<!\$)\d/.exec('$1 2')
// ["2"]
```
### 后行断言带来的问题
后行断言会按先右后左的匹配顺序，会导致一些不同的结果
```javascript
/^(\d+)(\d+)$/.exec('1053') // ["1053", "105", "3"]
/(?<=(\d+)(\d+))$/.exec('1053') // ["", "1", "053"]
```
上面代码中，第一行的代码没有使用后行断言，由于在贪婪模式下（会尽可能地多匹配），所以第一个括号内匹配的数字个数为3个，第二个括号内匹配的数字个数为1个，所以是"105"，"3"。而第二段代码使用了后行断言，从右边匹配起，也同样在贪婪模式下，右边的括号内会匹配3个数字，左边的括号匹配1个数字，所以是"1"，"053"。

## 5. 匹配字符\p{...}和\P{...}
在ES9中，正则式中可以通过\p{...}匹配Unicode字符，使用\P{...}来匹配非Unicode字符  
{}内可以传入想要匹配的内容，比如匹配ASCII字符  
因为是要判断Unicode字符，所以需要加上ES6中的u修斯符，否则{}内的东西会被视为量词
```javascript
/^\p{ASCII}+$/u.test('ABC@')  // true
/^\p{ASCII}+$/u.test('ABC🙃') // false
```

除了ASCII外，还能传入其他的内容，利用\p我们可以做到匹配所有的Unicode字符，不只是匹配ASCII字符  
像下面的数字字符，我们使用\d无法完全匹配，因为有些字符不是ASCII编码的  
```javascript
/\d/u.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // false
```
但是通过使用\p，传入相应的值，我们可以匹配到这些其他编码的数值
```javascript
/\p{Decimal_Number}/u.test('𝟏𝟐𝟑𝟜𝟝𝟞𝟩𝟪𝟫𝟬𝟭𝟮𝟯𝟺𝟻𝟼') // true
```

\p{}可传入的字符可以在[此处](https://github.com/tc39/proposal-regexp-unicode-property-escapes)找到  

## 6. 正则分组命名
在以往的正则表达式中，我们可以采用分组来将正则表达式其中一部分做为整体替换，比如下面这个  
```javascript
'2019-02-22'.replace(/(\d{4})-(\d{2})-(\d{2})/,'$2/$3/$1')
// "02/22/2019"
```
而在ES2018中，我们可以为这些分组命名
```javascript
'2019-02-22'.replace(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,'$<month>/$<day>/$<year>') // "02/22/2019"
```
这样写使得我们在匹配时更加语义化且明确  
使用\k<分组名>可引用正则表达式中具名组的值
```javascript
const RE_TWICE = /^(?<word>[a-z]+)!\k<word>$/;
RE_TWICE.test('abc!abc') // true
```

## 7. s修饰符
s修饰符是为了解决.匹配单个字符时的特殊情况，.可以匹配任意单个字符，但四个字节的 UTF-16 字符（可以使用u修饰符解决）和行终止符无法匹配。以下四个为行终止符  
- U+000A 换行符（\n）
- U+000D 回车符（\r）
- U+2028 行分隔符（line separator）
- U+2029 段分隔符（paragraph separator）
```javascript
/a.b/.test('a\nb');
// false
/a.b/s.test('a\nb');
// true
```
# ES10
---
## 1. 数组的flat方法和flatMap方法
flat方法用于降低数组元素的维数，如下
```javascript
[1,2,3,[4,5]].flat(); // [1,2,3,4,5]
```
flat方法传入一个参数，为要降低的维数，不传入参数默认为1
```javascript
[1,2,3,[4,[5,[6]]].flat(2); // [1,2,3,4,5,[6]]
```
flatMap方法为map方法和flat方法的结合，传入一个函数，对数组内的每个元素执行map操作，然后使用flat将其维数降1，flatMap只能降低一维  
```javascript
[1,2,3].flatMap(num=>[num+1]); // [2,3,4]
```
上面的flatMap可以分解为两步，第一步就是对数组进行map操作
```javascript
[1,2,3].map(num=>[num+1]); // [[2],[3],[4]]
```
然后使用flat铺平数组
```javascript
[[2],[3],[4]].flat(); // [2,3,4]
```

## 2. 字符串的trimStart方法和trimEnd方法
这两个方法用来清除字符串开头和结尾的空白字符，该方法不会修改原字符串
```javascript
" 123".trimStart(); // "123"
"123 ".trimEnd(); // "123"
```
这两个方法与trimLeft和trimRight两个方法对应，trimLeft是trimStart的别名，trimStart和trimEnd只是为了对应padStart和padEnd方法保持一致而写的名字  

## 3. Object.fromEntries方法
这个方法与ES8中的Object.entries方法作用正好相反，Object.entries方法是用于将一个对象的属性和值放到一个数组中返回，而Object.fromEntries可以将其变回原来的对象  
```javascript
let arr = Object.entries({foo:1,bar:2});
console.log('arr',arr);
let obj = Object.fromEntries(arr);
console.log('obj',obj);
/*
arr [["foo", 1], ["bar", 2]]
obj {foo: 1, bar: 2}
*/
```
关于两个方法正好相反的说法，如果给Object.entries方法传入的是数组，那么使用Object.fromEntries是无法返回原来的数组的，只能返回一个对象  
```javascript
let arr = Object.entries([1,2]);
console.log('arr',arr);
let obj = Object.fromEntries(arr);
console.log('obj',obj);
/*
arr [["0", 1], ["1", 2]]
obj {0: 1, 1: 2}
*/
```

## 4. 省略catch后的括号
在ES10中，我们可以省略catch后的括号
```javascript
try{
    // ... 
}catch{
    // ...
}
```

## 5. Symbol的description属性
我们知道，在构建Symbol对象的时候可以传入一个字符串做为对Symbol的描述，而在ES10中，我们可以直接通过description属性来访问这个字符串  
```javascript
var s = Symbol('test symbol');
console.log(s.description);
// "test symbol"
```
这个属性只能用来访问，不能对其赋值修改
```javascript
s.description = 'new description'
console.log(s.description);
// "test symbol"
```

## 6. 新的原始数据类型BigInt
在ES10之前，有6中基本类型Number,String,Boolean,null,undefined,Symbol，在ES10到来之时，多了一个BigInt类型，扩展了Number可以表示的值  
详见[JavaScript BigInt类型笔记整理](https://blog.csdn.net/zemprogram/article/details/104053457)

## 7. Function的toString方法的重新修订
在ES10之前，Function.prototype.toString()只会返回函数的主体，不会返回其中的注释和空格，而ES10之后，会完整的返回函数的内容
```javascript
function fn(){/*...*/ }
fn.toString();
// "function fn(){/*...*/ }"
```

## 8. sort方法更加稳定
这里的稳定，指的是当数组中两个成员用于比较的值一样时，二者在排序前的相对位置和排序后的相对位置是一样的  
比如下面这个数组
```javascript
[{num:1,name:"n1"},{num:2,name:"n2"},{num:1,name:"n3"}]
```
如果排序是使用属性num来排序的话，稳定的排序应该是
```javascript
[{num:1,name:"n1"},{num:1,name:"n3"},{num:2,name:"n2"}]
```
如果排序结果变成下面这样
```javascript
[{num:1,name:"n3"},{num:1,name:"n1"},{num:2,name:"n2"}]
```
n3被移动到n1前面，就称为不稳定

## 9. 全局对象globalThis
我们在访问全局对象时，一般都是window对象，而像在web worker和service worker中则是self对象，ES10提供了globalThis来表示全局对象，我们不用去特意使用其中一个  
看看下面的例子，我在一个使用了service worker的页面中打印window，self和globalThis  
![](https://img-blog.csdnimg.cn/20200120170736775.png)
可以看到在顶部环境下，globalThis对象等于window对象，而在service worker环境中，globalThis对象又等于self对象
![](https://img-blog.csdnimg.cn/20200120170842862.png)

## 10. 动态引入
在ES10中，加入了动态引入```import()```，动态引入返回一个promise对象，所以其后使用then方法来继续执行，then方法中的参数就是引入的对象
```javascript
import('./m.js').then(m=>{
    // ...
})
```
也可以使用async/await来接收引入的对象
```javascript
(async function(){
    let m = await import('./m.js');
    // ...
})()
```

## 11. 字符串的matchAll方法
```javascript
var regexp = /t(<e>e)(<stNum>st(\d?))/g;
var str = 'test1test2';

var array = [...str.matchAll(regexp)];
```
array打印如下图  
![](https://img-blog.csdnimg.cn/20200120173557852.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
该方法会将字符串与传入的正则进行匹配，返回每个匹配的结果及每个分组的匹配结果，groups为有命名的分组名及其值，index为匹配开始的下标，length为返回的数组长度

---
（还在补充ing...，有所缺漏请路过大佬指出）