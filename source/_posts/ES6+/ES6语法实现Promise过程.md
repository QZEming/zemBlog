---
title: ES6语法实现Promise过程
tags: ES6+
categories: JavaScript
abbrlink: e28b0d2e
date: 2019-07-29 03:27:16
---
Promise是ES6中的一种异步实现方式，现在我们来自己实现Promise。（有关Promise使用：[ES6学习笔记 Promise](https://blog.csdn.net/zemprogram/article/details/86564055)）
下面会详细写出实现过程及遇到的问题，如果要看代码实现的话直接滑至博客底部
<!-- more -->
## Promise构造
首先，我们创建一个promise.js来实现promise，使用class构造类并使用module.exports导出Promise
 ```javascript
 class Promise {
    constructor(fn) {
    }
}

module.exports = Promise
```
Promise要求传入一个函数作为参数，如果传入的不为函数，则会出现报错，我们来看看报错的内容
运行下面的代码
```javascript
new Promise(1).then(val => {
    console.log(val)
})
```
得到如下的报错
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728122209993.png)
所以我们在构造函数中加入对传入参数的类型判断
```javascript
class Promise {
    constructor(fn) {
        // 类型检验
        if (typeof fn !== 'function') {
            throw TypeError(`Promise resolver ${fn} is not a function`)
        }
    }
}
```
## Promise状态变换
---
接下来，我们知道，Promise有三个状态，进行中，已完成和已失败，我们需要一个值来记录这个状态，为了防止在后面写错，我们使用变量来记录三个状态对应的字符串，避免魔术字符串，并在使用resolve和reject方法时改变状态。
```javascript
class Promise {
    constructor(fn) {
        // 类型检验
        if (typeof fn !== 'function') {
            throw TypeError(`Promise resolver ${fn} is not a function`)
        }
        // 初始化值
        this.state = Promise.PENDING;
        this.value = null;
        this.reason = null;
        // 调用resolve和reject方法
        try {
            fn(this.resolve, this.reject);
        } catch (e) {
            this.reject(e)
        }
    }
    resolve(val) {
        this.val = val;
        this.state = Promise.RESOLVE;
    }
    reject(reason) {
        this.reason = reason;
        this.state = Promise.REJECTED;
    }
}

Promise.PENDING = "pending";
Promise.RESOLVE = "resolve";
Promise.REJECTED = "rejected";
```
状态改变后，我们就要来着手实现状态改变后的操作，在Promise中，我们使用then来实现状态改变后的操作，then方法接收两个参数，第一个是当状态变为resolve时调用，第二个是当状态变为rejected时调用。同样地，这里也需要进行参数的判断，当我们传入的参数不为函数时，会将这个值传下去，如下面的代码：
```javascript
new Promise((resolve, reject) => {
    resolve(1);
}).then(
    1
    // 等同于
    // (val)=>{
    //     return val
    // }
    ).then((val) => {
    console.log(val)
})
```
所以我们在写then方法时也许要对这部分内容做参数判断，在参数不为函数的情况下，模拟其默认实现
```javascript
class Promise {
    constructor(fn) {
       // ...
    }
    resolve(val) {
        this.val = val;
        this.state = Promise.RESOLVE;
    }
    reject(reason) {
        this.reason = reason;
        this.state = Promise.REJECTED;
    }
    then(onResolve, onRejected) {
        // 参数类型判断
        if (typeof onResolve !== 'function') {
            onResolve = (val) => {
                return val;
            }
        }
        if (typeof onRejected !== 'function') {
            onRejected = (reason) => {
                throw reason;
            }
        }
        if (this.state === Promise.RESOLVE) {
            onResolve(this.value);
        } else if (this.state === Promise.REJECTED) {
            onRejected(this.reason);
        }
    }
}
```
运行测试代码
```javascript
const Promise = require('./promise.js');

new Promise((resolve) => {
    resolve(1);
}).then((val) => {
    console.log(val)
})
```
结果发现报了一堆错误。。。如图
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190728185749532.png)
可以看出这是由于value被undefined引用，而这里引用value的也就只有this了，所以是this没有指向Promise，经过排除，是resolve和reject方法没有绑定在this上，所以在构造方法上对其进行绑定，为了使功能更加清晰，将初始化值和绑定放在函数中
```javascript
class Promise {
    constructor(fn) {
        // 类型检验
        if (typeof fn !== 'function') {
            throw TypeError(`Promise resolver ${fn} is not a function`)
        }
        // 初始化值
        this.initVal();
        // 绑定this
        this.initBind();
        // 调用resolve和reject方法
        try {
            fn(this.resolve, this.reject);
        } catch (e) {
            this.reject(e)
        }

    }
    initVal() {
        this.state = Promise.PENDING;
        this.value = null;
        this.reason = null;
    }
    initBind() {
        this.resolve = this.resolve.bind(this);
        this.reject = this.reject.bind(this);
    }
	// ...
}
```
再测试上面的代码，正常输出了1，这部分的功能实现了。
我们知道，promise是异步的，看下面这段使用了ES6中的Promise的代码，看看输出顺序
```javascript
console.log(1);
new Promise((resolve, reject) => {
    console.log(2);
    resolve('val');
}).then((val) => {
    console.log(4)
    console.log('val', val)
})
console.log(3);
```
## 异步实现
---
我们知道Promise是异步的，所以这里的输出顺序是1 2 3 4 val，而刚才实现的Promise没有做到异步，同样测试上面这段代码，使用自定义的Promise，得到的结果是1 2 4 val 3。为了实现这个异步，可以使用setTimeout来做到
（这种做法存在一定的问题，setTimeout为微任务，而promise实际上为宏任务，使用setTimeout会使得与规范中Promise的执行顺序存在一定问题，详见[详解JavaScript执行机制--异步执行顺序](https://blog.csdn.net/zemprogram/article/details/103328340)）
将then方法中的操作内容使用setTimeout包裹起来，这里setTimeout可以不设置时间，反正setTimeout是将其中的内容放在执行栈尾部，所以会在同步任务执行完后再执行里面的内容，同时，因为then返回一个Promise，所以我们使用一个Promise将其包裹起来。
为了实现链式调用，我们需要判断什么时候将then方法返回的promise改变状态，将resolve/reject执行后的值作为resolve的参数，并使用try catch捕获错误
```javascript
then(onResolve, onRejected) {
        // 参数类型判断
        if (typeof onResolve !== 'function') {
            onResolve = (value) => {
                return value;
            }
        }
        if (typeof onRejected !== 'function') {
            onRejected = (reason) => {
                throw reason;
            }
        }
        let promise = new Promise((resolve, reject) => {
            setTimeout(() => {
                if (this.state === Promise.RESOLVE) {
                    setTimeout(() => {
                    try {
                        const val = onResolve(this.value);
                        resolve(val);
                    } catch (e) {
                        reject(e);
                    }
                })
                } else if (this.state === Promise.REJECTED) {
                    setTimeout(() => {
                    try {
                        const val = onRejected(this.reason);
                        resolve(val);
                    } catch (e) {
                        reject(e);
                    }
                })
                }
            })
        })
        return promise;
    }
```
按上面的方法，实现了异步，但是如果我们在```new Promise```传入的参数函数中使用异步的话，又会怎样呢
```javascript
const Promise = require('./promise.js');

console.log(1);
new Promise((resolve, reject) => {
    setTimeout(() => {
        console.log(2);
        resolve('val');
    })
}).then((val) => {
    console.log(4)
    console.log('val', val)
})
console.log(3);
```
我们会发现，最后只输出了1 3 2，这是因为由于在传入```new Promise```的参数函数里的resolve执行前，已经执行了then，所以state实际上还是pending，我们可以在then方法返回的promise中加入this.state的输出，会发现输出的结果是pending。
要解决这个问题，我们可以在状态为pending时，将值储存起来，等待状态改变时再执行对应的操作，这里采用数组来存储，并在状态为pending时将value/reason push进对应的数组中
```javascript
class Promise {
    constructor(fn) {
        // 类型检验
        if (typeof fn !== 'function') {
            throw TypeError(`Promise resolver ${fn} is not a function`)
        }
        // 初始化值
        this.initVal();
        // 绑定this
        this.initBind();
        // 调用resolve和reject方法
        try {
            fn(this.resolve, this.reject);
        } catch (e) {
            this.reject(e)
        }

    }
    initVal() {
        this.state = Promise.PENDING;
        this.value = null;
        this.reason = null;
        this.resolveArr = []; // 成功回调的数组
        this.rejectArr = []; // 失败回调的数组
    }
    initBind() {
        this.resolve = this.resolve.bind(this);
        this.reject = this.reject.bind(this);
    }
    resolve(value) {
        if (this.state === Promise.PENDING) {
            this.value = value;
            this.state = Promise.RESOLVE;
            this.resolveArr.forEach(fn => fn(this.value));
        }
    }
    reject(reason) {
        if (this.state === Promise.PENDING) {
            this.reason = reason;
            this.state = Promise.REJECTED;
            this.rejectArr.forEach(fn => fn(this.reason));
        }
    }
    then(onResolve, onRejected) {
        // 参数类型判断
        if (typeof onResolve !== 'function') {
            onResolve = (value) => {
                return value;
            }
        }
        if (typeof onRejected !== 'function') {
            onRejected = (reason) => {
                throw reason;
            }
        }
        let promise = new Promise((resolve, reject) => {
            if (this.state === Promise.RESOLVE) {
                setTimeout(() => {
                    try {
                        const val = onResolve(this.value);
                        resolve(val);
                    } catch (e) {
                        reject(e);
                    }
                })
            } else if (this.state === Promise.REJECTED) {
                setTimeout(() => {
                    try {
                        const val = onRejected(this.reason);
                        resolve(val);
                    } catch (e) {
                        reject(e);
                    }
                })
            } else if (this.state === Promise.PENDING) {
                // 将值压入数组中
                this.resolveArr.push((value) => {
                    setTimeout(() => {
                        try {
                            onResolve(value)
                            resolve(val);
                        } catch (e) {
                            reject(e);
                        }
                    });
                })
                this.rejectArr.push((value) => {
                    setTimeout(() => {
                        try {
                            const val = onRejected(this.reason);
                            resolve(val);
                        } catch (e) {
                            reject(e);
                        }
                    });
                })
            }
        })
        return promise;
    }
}
```
## 对返回内容的处理
---
至此，我们已经基本完成了Promise的内容，回到创建Promise实例的地方，我们每次resolve返回的值都为一个字符串或者一个数字，那么如果我们返回一个Promise对象呢
```javascript
const Promise = require('./promise.js');

new Promise((resolve, reject) => {
    resolve(1)
}).then((val) => {
    return new Promise((resolve, reject) => {
        resolve(val);
    })
}).then((val) => {
    console.log('val', val)
})
```
正常情况下这里应该输出1，但是在我们的测试代码中，却输出了整个Promise对象，这正是上面缺漏的地方，我们没对复杂的参数进行对应的处理。
在Promise中，Promise构造器第一个参数回调会展开thenable或真正的Promise，而第二个参数reject并不像第一个参数resolve一样，reject只会将传入的Promise/thenable值原封不动地设置为拒绝理由，不会展开Promise/thenable。
下面对代码进行修改，首先要将返回的promise对象中调用的resolve在执行前判断各种条件并做处理，这里使用一个新的方法来对这部分条件进行判断处理，该方法需要的参数有：当前then方法返回的promise，onResolve/onReject返回的值以及resolve和reject
```javascript
Promise.resolvePromise = function(promise, val, resolve, reject) {
    // 判断onResolve/onReject的返回值的类型是否为Promise
    if (val instanceof Promise) {
        val.then(
            // 状态变为resolve的情况
            value => {
                Promise.resolvePromise(promise, value, resolve, reject);
            },
            // 状态变为rejected的情况
            reason => {
                reject(reason);
            }
        )
    }
    // 判断onResolve/onReject的返回值的类型是否为对象或者方法
    else if (Object.prototype.toString.call(val) === "[object Object]" ||
        Object.prototype.toString.call(val) === "[object Function]") {
        try {
            const then = val.then;
            if (typeof then === "function") {
                // 取出val的then方法，将其this绑定在val上
                then.call(
                    val,
                    value => {
                        Promise.resolvePromise(promise, value, resolve, reject);
                    },
                    reason => {
                        reject(reason);
                    }
                )
            } else {
                resolve(val)
            }
        } catch (e) {
            reject(e)
        }
    }
    // 判断onResolve/onReject的返回值的类型不为对象或者函数，直接使用resolve执行
    else {
        resolve(val);
    }
}
```
这个方法到这里基本可以解决问题了，但是如果在调用方法时，onResolve和onRejected同时被调用，按Promise的规定，我们会先执行优先变化的那个，而慢变化的那个，由于Promise状态的不可逆性，只能由pending变为resolve/rejected，所以慢变化的那个就会被忽略，所以我们再对上面的代码进行修改
```javascript
Promise.resolvePromise = function(promise, val, resolve, reject) {
    // 用于判断状态是否已经改变了
    let called = false;
    // 判断onResolve/onReject的返回值的类型是否为Promise
    if (val instanceof Promise) {
        val.then(
            // 状态变为resolve的情况
            value => {
                Promise.resolvePromise(promise, value, resolve, reject);
            },
            // 状态变为rejected的情况
            reason => {
                reject(reason);
            }
        )
    }
    // 判断onResolve/onReject的返回值的类型是否为对象或者方法
    else if (Object.prototype.toString.call(val) === "[object Object]" ||
        Object.prototype.toString.call(val) === "[object Function]") {
        try {
            const then = val.then;
            if (typeof then === "function") {
                // 取出val的then方法，将其this绑定在val上
                then.call(
                    val,
                    value => {
                        if (called) {
                            return;
                        }
                        called = true;
                        Promise.resolvePromise(promise, value, resolve, reject);
                    },
                    reason => {
                        if (called) {
                            return;
                        }
                        called = true;
                        reject(reason);
                    }
                )
            } else {
                if (called) {
                    return;
                }
                called = true;
                resolve(val)
            }
        } catch (e) {
            if (called) {
                return;
            }
            called = true;
            reject(e)
        }
    }
    // 判断onResolve/onReject的返回值的类型不为对象或者函数，直接使用resolve执行
    else {
        resolve(val);
    }
}
```
## Promise的重复调用
---
问题解决，来到下一个问题，如果Promise重复调用，会发生什么呢，看下面的代码
```javascript
let p1 = new Promise((resolve, reject) => {
    resolve(1);
})
let p2 = p1.then(() => {
    return p2;
})
```
这段代码的执行最终会报下图的错误
![在这里插入图片描述](https://img-blog.csdnimg.cn/20190729031605794.png)
实际上就是then方法返回的内容与本身的promise相同，这样不断地循环调用，最终报错，我们在Promise.resolvePromise方法的开头进行判断并将上面的错误复制进去。
```javascript
Promise.resolvePromise = function(promise, val, resolve, reject) {
    // 对循环调用的判断处理
    if (promise === val) {
        reject(new TypeError('Chaining cycle detected for promise'))
    }
	// ...
}
```
## 最终代码
---
```javascript
class Promise {
    constructor(fn) {
        // 类型检验
        if (typeof fn !== 'function') {
            throw TypeError(`Promise resolver ${fn} is not a function`)
        }
        // 初始化值
        this.initVal();
        // 绑定this
        this.initBind();
        // 调用resolve和reject方法
        try {
            fn(this.resolve, this.reject);
        } catch (e) {
            this.reject(e)
        }
    }

    initVal() {
        this.state = Promise.PENDING;
        this.value = null;
        this.reason = null;
        this.resolveArr = []; // 成功回调的数组
        this.rejectArr = []; // 失败回调的数组
    }

    initBind() {
        this.resolve = this.resolve.bind(this);
        this.reject = this.reject.bind(this);
    }

    resolve(value) {
        if (this.state === Promise.PENDING) {
            this.value = value;
            this.state = Promise.RESOLVE;
            this.resolveArr.forEach(fn => fn(this.value));
        }
    }

    reject(reason) {
        if (this.state === Promise.PENDING) {
            this.reason = reason;
            this.state = Promise.REJECTED;
            this.rejectArr.forEach(fn => fn(this.reason));
        }
    }

    then(onResolve, onRejected) {
        // 参数类型判断
        if (typeof onResolve !== 'function') {
            onResolve = (value) => {
                return value;
            }
        }
        if (typeof onRejected !== 'function') {
            onRejected = (reason) => {
                throw reason;
            }
        }
        // 实现链式调用
        let promise = new Promise((resolve, reject) => {
            if (this.state === Promise.RESOLVE) {
                setTimeout(() => {
                    try {
                        const val = onResolve(this.value);
                        Promise.resolvePromise(promise, val, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                })
            } else if (this.state === Promise.REJECTED) {
                setTimeout(() => {
                    try {
                        const val = onRejected(this.reason);
                        Promise.resolvePromise(promise, val, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                })
            } else if (this.state === Promise.PENDING) {
                // 将值压入数组中
                this.resolveArr.push((value) => {
                    setTimeout(() => {
                        try {
                            onResolve(value)
                            Promise.resolvePromise(promise, val, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    });
                })
                this.rejectArr.push((value) => {
                    setTimeout(() => {
                        try {
                            const val = onRejected(this.reason);
                            Promise.resolvePromise(promise, val, resolve, reject);
                        } catch (e) {
                            reject(e);
                        }
                    });
                })
            }
        })
        return promise;
    }
}

Promise.PENDING = "pending";
Promise.RESOLVE = "resolve";
Promise.REJECTED = "rejected";
Promise.resolvePromise = function(promise, val, resolve, reject) {
    // 对循环调用的判断处理
    if (promise === val) {
        reject(new TypeError('Chaining cycle detected for promise'))
    }
    // 用于判断状态是否已经改变了
    let called = false;
    // 判断onResolve/onReject的返回值的类型是否为Promise
    if (val instanceof Promise) {
        val.then(
            // 状态变为resolve的情况
            value => {
                Promise.resolvePromise(promise, value, resolve, reject);
            },
            // 状态变为rejected的情况
            reason => {
                reject(reason);
            }
        )
    }
    // 判断onResolve/onReject的返回值的类型是否为对象或者方法
    else if (Object.prototype.toString.call(val) === "[object Object]" ||
        Object.prototype.toString.call(val) === "[object Function]") {
        try {
            const then = val.then;
            if (typeof then === "function") {
                // 取出val的then方法，将其this绑定在val上
                then.call(
                    val,
                    value => {
                        if (called) {
                            return;
                        }
                        called = true;
                        Promise.resolvePromise(promise, value, resolve, reject);
                    },
                    reason => {
                        if (called) {
                            return;
                        }
                        called = true;
                        reject(reason);
                    }
                )
            } else {
                if (called) {
                    return;
                }
                called = true;
                resolve(val)
            }
        } catch (e) {
            if (called) {
                return;
            }
            called = true;
            reject(e)
        }
    }
    // 判断onResolve/onReject的返回值的类型不为对象或者函数，直接使用resolve执行
    else {
        resolve(val);
    }
}


module.exports = Promise
```
基本实现了Promise的大部分内容，出现什么错误请路过大佬指出。

---
参考文档：[Promises/A+规范](http://www.ituring.com.cn/article/66566)