---
title: React Native 使用自定义Iconfont
date: 2017-06-19 22:00:27
categories: React Native
tags:
    - 学习笔记
    - React
    - Iconfont
description: React Native 使用自定义Iconfont
---

## React Native 使用自定义 iconfont
<!--more-->
1. 下载字体文件
2. `IOS` -> 将字体文件拽进`xcode`工程里，然后在`info.plist`加入`Fonts provided by application`，这是个数组，写上`fonts/iconfont.ttf`.
3. `Android`: 把字体文件拷贝到`[project root]/android/app/src/main/assets/fonts/`
3. `fontFamily: 'iconfont'` PS: 下载的文件叫什么，`fontFamily`就是什么。
4. 因为我们的`iconfont`都是`&#xe038;`这种格式的，这不是把这些传给`Icon`组件就行了吗，但是试了之后，会转义成字符串。这可不是我们想要的。
5. 那怎么办，不让他转义是一种方式：
```javascript
var content='<strong>content</strong>';    

React.render(
    <div dangerouslySetInnerHTML={{__html: content}}></div>,
    document.body
);
```
没生效，故弃之。
7. 我们用的时候只要用`e038`就行了，因为`e038`是个16进制数字，我们要先转换成10进制，再转化回去16进制就行了。
```javascript
String.fromCharCode(parseInt('e038', 16))
```
这样就会返回我们要的图标；
