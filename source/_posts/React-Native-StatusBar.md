---
title: React Native StatusBar
date: 2017-06-25 01:41:19
categories: React Native
tags:
    - 学习笔记
    - React
    - Redux
description: React Native StatusBar
---

## `StatusBar`
`StatusBar`就是手机上面的状态栏，一般显示网速的信号，电量啥的那个。  
`StatusBar`是可以设置颜色，显示与否啥的<!--more-->

#### `barStyle`
字体颜色可以通过`barStyle`属性来设置，一般就是亮色(`light-content`)和暗色(`dark-content`)。`IOS`还可以单独设置两个属性，一个是`networkActivityIndicatorVisible`,是代表有网络请求的时候状态栏是否显示转的`菊花`，另一个是`showHideTransition`切换显示隐藏状态栏的时候是否有动画(`fade`、`slide`)

#### `translucent`
指定状态栏是否透明。设置为true时，应用会在状态栏之下绘制（即所谓“沉浸式”——被状态栏遮住一部分）。常和带有半透明背景色的状态栏搭配使用。

等等，我理解的沉浸式不就是`App`的`header`是啥颜色，状态栏就是啥颜色就行了吗？还得好好理解一下。[为什么在国内会有很多用户把「透明栏」（Translucent Bars）称作 「沉浸式顶栏」？](https://www.zhihu.com/question/27040217)

#### `backgroundColor`
`Android`特有的，顾名思义，可以设置状态栏的背景颜色，这个是我喜欢的，理由同上。
`IOS`为啥不需要，因为`IOS`状态栏是透明的，一般要设置个高度`20`把这个状态栏让出来，所以颜色一般都是跟`App`的`Header`一样，不一样也不好看吧。。

#### `hidden`
是否显示状态栏

#### `animated`
指定状态栏的变化是否应以动画形式呈现。目前支持这几种样式：`backgroundColor`, `barStyle`和`hidden`。
还没有实际场景用到，用到再说吧

