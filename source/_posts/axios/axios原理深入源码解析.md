---
title: axios原理深入源码解析
abbrlink: 8742b180
date: 2019-11-26 17:31:59
tags: axios
categories: 框架
---
# axios
---
axios的[github地址](https://github.com/axios/axios)上明确写了axios的特征是
Make XMLHttpRequests from the browser（从浏览器发起XMLHttpRequests请求）
Make http requests from node.js（从node.js发起http请求）
Supports the Promise API（支持PromiseAPI）
Intercept request and response（拦截请求和响应）
Transform request and response data（转换请求和响应数据）
Cancel requests（取消请求）
Automatic transforms for JSON data（自动转换json数据）
Client side support for protecting against XSRF（客户端支持自动防止XSRF）
下面就这些特征查看源码来解析
<!-- more -->
### 可以在浏览器和服务器上发起请求
上面已经写了，axios可以在浏览器发起XMLHttpRequests请求，而在node.js可以发起http请求，也即是说，axios可以在浏览器和服务器上都可以发起请求
看看axios的源码目录
![在这里插入图片描述](https://img-blog.csdnimg.cn/20191126170956723.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3plbXByb2dyYW0=,size_16,color_FFFFFF,t_70)
其中xhr.js是浏览器上发起请求，http.js是服务器上发起请求
我们能看到，xhr.js中有一行代码
```javascript
var request = new XMLHttpRequest();
```
可以明确看到，在浏览器上的请求是基于XMLHttpRequest的
而http.js中的请求，可以在最后看到
```javascript
if (utils.isStream(data)) {
  data.on('error', function handleStreamError(err) {
    reject(enhanceError(err, config, null, req));
  }).pipe(req);
} else {
  req.end(data);
}
```
通过在这段代码前对data进行处理，在这里最后发出请求
### 支持promise API
我们知道，promise是ES6中出现的语法，很好地处理了异步，而axios中支持promise
关于这一点，我们从axios的使用就可以看出来axios的get方法返回了一个promise，然后我们对其进行操作，其他的请求也是一样
```javascript
axios.get('url')
	.then(res=>{
		// 响应处理
	}).catch(err=>{
		// 错误处理
})
```
在[github上的源码](https://github.com/axios/axios/blob/master/lib/core/Axios.js)也可以看到这部分明确返回了一个promise
```javascript
Axios.prototype.request = function request(config) {
  /*eslint no-param-reassign:0*/
  // Allow for axios('example/url'[, config]) a la fetch API
  if (typeof config === 'string') {
    config = arguments[1] || {};
    config.url = arguments[0];
  } else {
    config = config || {};
  }

  config = mergeConfig(this.defaults, config);

  // Set config.method
  if (config.method) {
    config.method = config.method.toLowerCase();
  } else if (this.defaults.method) {
    config.method = this.defaults.method.toLowerCase();
  } else {
    config.method = 'get';
  }

  // Hook up interceptors middleware
  var chain = [dispatchRequest, undefined];
  var promise = Promise.resolve(config);

  this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
    chain.unshift(interceptor.fulfilled, interceptor.rejected);
  });

  this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
    chain.push(interceptor.fulfilled, interceptor.rejected);
  });

  while (chain.length) {
    promise = promise.then(chain.shift(), chain.shift());
  }

  return promise;
};
```
### 拦截请求和响应及转换数据
拦截器的相关代码也在上面promise里面的代码，我将相关的代码拿出来
```javascript
// 将拦截器放在第一个，undefined放在第二个
// 在后面的处理中，是两个两个处理的
// 因为使用了promise.then，其第一个参数对应的是resolve第二个参数对应的是rejected
// 拦截器会对应于resolve，undefined会对应于rejected
var chain = [dispatchRequest, undefined];
var promise = Promise.resolve(config);
// 将请求拦截器中间件添加到chain数组头部
this.interceptors.request.forEach(function unshiftRequestInterceptors(interceptor) {
  chain.unshift(interceptor.fulfilled, interceptor.rejected);
});
// 将响应拦截器中间件添加到chain数组尾部
this.interceptors.response.forEach(function pushResponseInterceptors(interceptor) {
  chain.push(interceptor.fulfilled, interceptor.rejected);
});
// 依次使用promise处理chain数组内的东西
while (chain.length) {
  // 这里使用shift方法，每次调用两次这个方法，即是每次拿出数组中的两个拦截器
  promise = promise.then(chain.shift(), chain.shift());
}
```
而找寻里面的dispatchRequest，会发现它是从[dispatchRequest.js](https://github.com/axios/axios/blob/master/lib/core/dispatchRequest.js)中import过来的
```javascript
var dispatchRequest = require('./dispatchRequest');
```
在[dispatchRequest.js](https://github.com/axios/axios/blob/master/lib/core/dispatchRequest.js)中，我们可以找到拦截请求和响应的相关代码，在这里对请求和响应的数据做了转换
```javascript
// 处理请求的数据
response.data = transformData(
  response.data,
  response.headers,
  config.transformResponse
);
// 处理响应的数据
var adapter = config.adapter || defaults.adapter;

return adapter(config).then(function onAdapterResolution(response) {
  throwIfCancellationRequested(config);

  // Transform response data
  response.data = transformData(
    response.data,
    response.headers,
    config.transformResponse
  );

  return response;
}, function onAdapterRejection(reason) {
  if (!isCancel(reason)) {
    throwIfCancellationRequested(config);

    // Transform response data
    if (reason && reason.response) {
      reason.response.data = transformData(
        reason.response.data,
        reason.response.headers,
        config.transformResponse
      );
    }
  }
```
### 取消请求
axios提供了取消请求的接口，实际上在源码中还是通过最关键的一步```request.abort()```来取消请求，使用isCancel来作为是否已经取消的标识，使用Cancel来作为重复取消时抛出的错误，使用CancelToken来作为取消请求的核心处理，在处理中采取promise异步的方法
具体的过程详见[axios请求取消步骤源码详解](https://blog.csdn.net/zemprogram/article/details/103202132)
axios的取消请求相比于我们直接使用abort，因为在promise中写入重复触发时会抛出cancel对象，所以无需我们自己去解决按钮被连续多次点击的问题，同时取消请求也能在我们发出多个相关联请求时，使用最新的数据。
### 防止XSRF攻击
首先我们说说XSRF攻击是什么，简单来说，XSRF攻击就是当我们访问A页面，在拿到相应的cookie后，去访问一个不安全的B页面，而这个B页面通过script，img，video这些标签的src属性，发起了请求，而因为此时A页面的cookie还没失效，在避过同源策略的同时，向A页面所在的服务器发起一个请求，而这个请求会带上之前的cookie，于是会导致一些安全问题。

为了防止XSRF攻击，axios采用了在给请求添加一个自定义属性，传入一个token，在服务器判断该属性值是否一样
看看源码内容[xhr.js](https://github.com/axios/axios/blob/master/lib/adapters/xhr.js)
```javascript
// 判断是否为标准的浏览器环境
if (utils.isStandardBrowserEnv()) {
  var cookies = require('./../helpers/cookies');

  // isURLSameOrigin判断是否有跨域，如果有跨域的话需要有凭证
  // 凭证的作用是判断当前请求为跨域类型时是否在请求中协带cookie
  // 满足上面条件后，判断是否有config.xsrfcookieName
  // 如果有的话，使用cookies的read方法处理值后赋值给xsrfValue
  var xsrfValue = (config.withCredentials || isURLSameOrigin(fullPath)) && config.xsrfCookieName ?
    cookies.read(config.xsrfCookieName) :
    undefined;

  if (xsrfValue) {
    requestHeaders[config.xsrfHeaderName] = xsrfValue;
  }
}
```
首先axios会判断当前环境是否为标准的浏览器环境，如果是标准的浏览器环境的话，就会继续执行。对于XSRF攻击来说，最基本的就是跨域了，所以我们对发出请求的域和当前的域做同源判断，如果是跨域的话，就必须有凭证```config.withCredentials || isURLSameOrigin(fullPath)```，凭证简单来说就是当前请求在跨域时是否可以带上cookie。
在满足上面条件时，即是同源或者跨域但是有凭证这两种情况，同源的我们自然不必再处理，而对于跨域的我们接下去看，当config中有xsrfCookieName时，将config.xsrfCookieName的内容使用cookies.read处理后返回给xsrfValue
cookies.read在[cookies.js](https://github.com/axios/axios/blob/master/lib/helpers/cookies.js)中
```javascript
read: function read(name) {
  var match = document.cookie.match(new RegExp('(^|;\\s*)(' + name + ')=([^;]*)'));
  return (match ? decodeURIComponent(match[3]) : null);
},
```
最后，当xsrfValue存在时将其加入请求头中，这里的config.xsrfHeaderName就是自定义的属性，在[defaults.js](https://github.com/axios/axios/blob/master/lib/defaults.js)中可见
```javascript
xsrfHeaderName: 'X-XSRF-TOKEN',
```
## 缺点
---
1. 不支持jsonp，需要自己封装
2. 基于XMLHttpRequest实现，所以无法在service worker，web worker中使用

（有待补充）