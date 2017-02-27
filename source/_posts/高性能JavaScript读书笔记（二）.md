---
title: 高性能JavaScript读书笔记（二）
date: 2017-02-06 14:30:57
categories: 高性能JavaScript
tags:
    - 学习笔记
    - 高性能JavaScript
description: 读《高性能JavaScript》的时候的一些理解
---
读《高性能JavaScript》的时候的一些理解
<!--more-->
<iframe frameborder="no" border="0" marginwidth="0" marginheight="0" width=330 height=86 src="//music.163.com/outchain/player?type=2&id=394729&auto=0&height=66"></iframe>

### 第二章 数据存取

执行函数的时候会创建一个被称为执行环境的内部对象。一个执行环境定义了一个一个函数执行时的环境，函数每次执行时，对应的执行环境都是独一无二的，也就是说每次调用这个函数都会创建不同的执行环境。函数执行完毕，执行环境就销毁。

每个执行环境都有自己的作用域链，用来解析标识符（我的理解是变量）。执行函数的时候会有个搜索标识符的过程，这个过程会影响性能。而搜索变量的时候，会从作用域链一层层往上找，找到就用，找不到就接着找。所以越近的变量搜索过程越短，性能损失越小。所以要尽可能地缓存全局变量，来减少损耗。

```javascript
function changeColor () {
    var doc = document,
        oDiv1 = doc.getElementById('div1');
    oDiv1.style.backgroundColor = '#000';
}
```
`with` 语句可以把全局变量置于作用域链的头部，但是这是创建了一个新的作用域链，虽然全局变量好找了，但是局部变量却不好找了，更浪费性能。所以不推荐用with语句。

`try-catch` 中的 `catch` 也可以改变作用域链，所有的局部变量会被放到新的作用域链里面，这样也会影响性能。 `try-catch` 语句可以适当使用，来避免不可预知的错误。建议在 `catch` 中处理错误通过调用函数的方式，这样不访问局部变量就会不会影响性能。

#### 动态作用域

`with` 、 `try-catch` 中 `catch` 子句、 `eval()` 都是动态作用域。

动态作用域只存在函数执行过程中，不建议使用动态作用域，因为可能会改变变量的值。

```javascript
function excute (code) {
    eval(code);

    var w = window;

    // 如果code是'var window = {}'，w 就不是真正的window了。
}
```

#### 闭包、作用域和内存

使用闭包时，闭包的 `[[Scope]]` 包含于执行环境作用域链相同的对象的引用，通常来说这些活动对象会随着执行环境销毁而销毁，但是闭包就不会。
使用闭包时要注意：频繁访问跨作用域的标识符时，每次都会带来性能损失。

#### 对象成员
