---
title: ES6函数
date: 2017-03-09 14:38:58
categories: ES6
tags:
    - 学习笔记
    - ES6
description: ES6函数
---

参考自阮一峰老师的[ECMAScript 6 入门](http://es6.ruanyifeng.com/#docs/function) <!--more-->

#### 函数默认值

`es6` 支持为函数参数赋予默认值

```javascript
function logA (a = 1) {
    console.log(a);
}
logA(); // 1
logA(3); // 3
```
避免了我们类似这样的操作
```javascript
function logA (a) {
    a = a || 1;
    console.log(a);
}
```
可以与解构赋值结合使用
```javascript
function sum ({a, b = 4}) {
    console.log(a + b);
}
sum({a: 4}); // 8
sum({a: 4, b: 5}); // 9
```
参数默认值的位置一般都放在尾参数，这样的话调用这个函数的时候，就可以省略
```javascript
function sum (a = 3, b) {
    console.log(a + b);
}
sum(undefined, 5); // 8
sum(5, undefined); // NaN
sum(4, 5); // 9
sum(5); // NaN

function sum2 (a, b = 5) {
    console.log(a + b);
}
sum2(undefined, 5); // NaN
sum2(5, undefined); // 10
sum2(4, 5); // 9
sum2(5); // 10
```
PS: `null` 不会触发默认值，`undefined` 会。

函数的length会返回没有指定参数默认值的参数个数。

#### rest参数
设置rest参数，可以用 `for of` 遍历。
```javascript
function sum (...keys) {
    let sum = 0;
    for (key of keys) {
        sum += key;
    }
    console.log(key);
}
sum(1, 3, 5); // 9
```
rest参数还可以代替arguments
```javascript
function sortNumbers () {
    return Array.prototype.slice.call(arguments).sort();
}

sortNumbers (...numbers) => numbers.sort();
```
PS: rest参数只能是最后一个参数；函数的length属性不包括rest参数。
