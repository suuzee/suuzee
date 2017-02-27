---
title: Redux
date: 2017-02-26 17:01:48
categories: Redux
tags:
    - 学习笔记
    - Redux
description: 了解Redux
---

早就知道 `Redux` 这个玩意儿，并且看过一些介绍，以及听过一次分享。但是，由于听分享的时候准备的不够充分，导致听不太懂。明天还会有个分享，所以打算看一下，准备准备。之前看的没有做笔记，并且不够专注，这次在这里写入我个人的理解。<!--more-->

主要看阮一峰的 [Redux入门教程](http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html) 。

#### Redux

##### 设计思想

> 1. Web 应用是一个状态机，视图与状态是一一对应的。
2. 所有的状态，保存在一个对象里面。

##### 主要概念

1. Store: 一个只能存在一个的容器，用来保存行为与响应。可以通过 `createStore` 方法来生成。可以通过 `store.subscribe` 方法来设置监听函数。 `State` 发生变化的时候，监听函数自动执行。通过 `store.unsubscribe` 方法来解绑。

2. State: 应用的状态，可以通过 `store.getState` 方法获取当前 `State` 。

3. Action: 用户的行为。通过类型来区分，改变 `state` 。可以通过 `store.dispatch` 方法发出Action。
```javascript
const action = {
    type: 'ADD_TODO',
    text: 'Learn Redux'
}
```
4. Reducer: 通过 `action` 以及旧的 `state` 来生成新的 `state` 。

##### Store

##### Reducer

`Reducer` 可以被拆分出来，通过 `combineReducers` 方法将几个 `reducer` 合并。

##### 工作流程

![](http://www.ruanyifeng.com/blogimg/asset/2016/bg2016091802.jpg)

1. 用户触发 `Action` ；

2. `Store` 调用 `Reducer` ，后者返回一个新的 `State`；

3. `State` 发生变化，触发监听函数；

4. 监听函数更新 `View`。
