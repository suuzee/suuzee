---
title: 如何通过axios实现一个http请求库（译）
date: 2018-08-31 13:46:52
categories: axios
tags:
    - 学习笔记
    - axios
    - 翻译
description: 分析axios内部原理
---

最近在[阮一峰](http://www.ruanyifeng.com)的一篇[每周分享](http://www.ruanyifeng.com/blog/2018/08/weekly-issue-20.html)上看到了这篇文章，想翻译（第一次）一下，正好想借此研究一下`axios`这个大名鼎鼎的库。

原文链接[https://www.tutorialdocs.com/article/axios-learn.html](https://www.tutorialdocs.com/article/axios-learn.html)  <!--more-->

## 概览

在我们做前端开发的时候，经常碰到需要异步请求的情况。当我们用一个强大而齐全的http请求库的话，可以大大降低开发成本，提高开发效率。

`axios`是近几年非常火的一个http请求库。现在[Github](https://github.com/axios/axios)已经有了超过`40k`的Star，并且被很多前端大牛推荐。

因此，想要通过`axios`帮助我们实现一个http请求库之前必须要了解它的设计方式。在编写本文时，`axios`的版本为`0.18.0`，因此我们以这个版本为例来阅读和分析特定的源代码。`axios`的当前所有源文件都在`lib`文件夹中，因此下面的路径都是指`lib`文件夹的路径。

这里我们主要会谈及：

- 如何使用`axios`
- `axios`的核心模块（`requests`, `interceptors`, `withdrawals`）是如何设计实现的？
- `axios`的设计有哪些优点？

### 如何使用`axios`

想要理解`axios`的设计，我们首先要了解一下如何使用它。让我们用一个简单的例子来了解`axios`的api。

#### 发送请求

```javascript
axios({
    method:'get',
    url:'http://bit.ly/2mTM3nY',
    responseType:'stream'
})
    .then(function(response) {
    response.data.pipe(fs.createWriteStream('ada_lovelace.jpg'))
});
```

这是一个官方的例子。从上面的代码上来看`axios`使用起来非常像`jQuery`的`ajax`，并且他们都返回一个`Promise`来继续下面的操作（在这里也能用成功回调函数，但是还是推荐`Promise`或者`await`）。

我不用解释这个简单的例子，我们再来看看怎么添加一个过滤器（`filter`）。

#### 添加拦截器（`interceptors`）

```javascript
// Add a request interceptor. Note that there are 2 functions - one succeeds and one fails, and the reason for this will be explained later.
axios.interceptors.request.use(function (config) {
    // The process before sending the request.
    return config;
}, function (error) {
    // Request error handling.
    return Promise.reject(error);
});

// Add a response interceptor.
axios.interceptors.response.use(function (response) {
    // Processing for the response data.
    return response;
}, function (error) {
    // Processing after the response error.
    return Promise.reject(error);
});
```

从上面的代码我们可以看出：在发请求之前，我们能够拦截到`config`参数，并且可以对这个`config`进行一些处理；在请求回来之后，我们也可以对返回的数据进行一些通用处理。与此同时，我们也可以对`request`和`response`添加失败监听，来处理失败的情况。

#### 取消HTTP请求

当我们开发搜索相关的模块的时候，我们需要经常发请求去查询。通常情况下，在我们发下一个请求的时候，我们需要把最后的请求给取消掉。因此，取消请求相关的方法也是一个优点。`axios`取消请求的示例代码如下：

```javascript
const CancelToken = axios.CancelToken;
const source = CancelToken.source();

axios.get('/user/12345', {
    cancelToken: source.token
}).catch(function(thrown) {
    if (axios.isCancel(thrown)) {
        console.log('Request canceled', thrown.message);
    } else {
        // handle error
    }
});

axios.post('/user/12345', {
    name: 'new name'
}, {
    cancelToken: source.token
})

// cancel the request (the message parameter is optional)
source.cancel('Operation canceled by the user.');
```

从上面的代码可以看出，`axios`使用的是基于`CancelToken`提出的撤销。然而，这个提案已经被撤销，详情请见[这里](https://github.com/tc39/proposal-cancelable-promises)；具体的取消方法的实现会在后面源码分析那部分进行解释。