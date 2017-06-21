---
title: React Native 使用 Redux
date: 2017-06-22 00:04:07
categories: React Native
tags:
    - 学习笔记
    - React
    - Redux
description: React Native 使用 Redux
---

## 为什么用
因为更好的易用性和可维护性，将页面拆分了不同的大大小小组件，同一个页面的各个组件怎么能用同一套数据，一层一层传递显然不靠谱，如果能放到一个地方谁都能取出来用，那该多好，就用了`Redux`<!--more-->

## 正常用

`Store`: 数据集合，一个应用只有一个  
`Action`: 必须定义一个类型，可以传递数据给`state`，例子：从`Native`端拿到的数据怎么放到`Store`里面  
`Reducer`: 纯函数，两个参数：1. state; 2. action  


合并多个`reducer`
`combineReducer`

## 异步

正常`dispatch`一个`Action`，在这个`Action`里面去发异步操作，操作前`dispatch`一个`Action`，异步操作之后`dispatch`一个`Action`

`ActionCreator`应该返回一个函数，这个函数有两个参数，一个是`dispatch`，一个是`getState`；然后开始的时候`dispatch`一个`Action`，接着执行异步操作，操作之后再`dispatch`一个`Action`;  
`redux-thunk`: 能让`store.dispatch`参数是函数  
`redux-promise`: `AcrionCreator`返回一个`Primise`对象。
两个用法：
1. `ActionCreator`返回值是一个`Promise`对象
2. `Action`的`payload`对象是一个`Promise`对象，需要`redux-actions`里面的`createAction`方法，该方法接受两个参数：1. type; 2. promise对象。如果`Action`是一个`Promise`对象，它`resolve`以后的值也是一个`Action`对象。而如果`Action.payload`是一个`Promise`对象，它`resolve`和`reject`之后都可以出发`Action`。

## `React-Reduex`
- `UI`组件和容器组件  
`UI`组件只负责展示，是一个纯组件，只受`props`影响，可以接受用户操作，发出`Action`；容器组件处理逻辑，比如将`state`里面的值转化成`UI`组件可用的`propos`属性，包含着`UI`组件。
- `connect`方法  
`connect`用于从`UI组件`生成`容器组件`
- `mapStateToProps`
- `mapDispatchToProps`
- `<Provider />`组件：能让每个组件都能拿到`state`，从`this.context`里面拿

```javascript
// connect
import { connect } from 'react-redux'; 
// UI组件
import List from './List';

const mapStateToProps = () => {
    // 将state的值传给props，返回一个对象，key是props的key，value就是props的value
    // 自动订阅Store，当state更新，会自动更新UI
}

// 如果是一个函数
const mapDispatchToProps = (dispatch, ownProps) => {
    // 定义用户应该怎么操作触发Action
    // dispatch触发Action
}

// 如果是一个对象
const mapDispatchToProps =  {
    // key是UI组件对应的同名参数
    // value 是一个ActionCreator，满足条件自动发
}


export default connect(
    mapStateToProps,
    mapDispatchToProps
)(List);

// connect()(List);这个List就是将List这个UI组件包裹进去。
```