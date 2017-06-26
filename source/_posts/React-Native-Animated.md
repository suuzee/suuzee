---
title: React Native Animated
date: 2017-06-26 23:47:47
categories: React Native
tags:
    - 学习笔记
    - React
    - Animated
description: React Native Animated
---

今天有个需求是展示内容，不完全展示，当点击下拉箭头的时候，才能展示完全内容。展示完全的时候这个下拉箭头会旋转成收回箭头，所以想着能不能用动画来实现一个旋转的过程。说干就干<!--more-->

其实逻辑很简单，一共有3种状态：初始化页面的时候，箭头向下，不旋转；点开内容看全部的时候，箭头应该向上，旋转180deg；收回的时候，箭头向下，旋转回0deg。

直接上代码

```javascript
import { View, Text, StyleSheet, TouchableWithoutFeedback, Animated, Easing } from 'react-native';

export default class App extends Component {
    // xxx
    constructor () {
        super();
        this.state = {
            // xxx
            moreIconRotateConfig: ['0deg', '180deg'],   // 给outputRange一个初始值
            moreIconRotateValue: new Animated.Value(0)   // 给动画一个初始值
        }
    }

    loadMore () {
        let { moreIconRotateValue, isFullTextShow } = this.state;
        let moreIconRotateConfig = ['0deg', '180deg'];

        // 保证每次执行动画都是从0开始
        moreIconRotateValue.setValue(0);

        // 开始执行动画。如果要循环的话，就在start 里面写回调，执行本身开始递归。
        Animated.timing(moreIconRotateValue, {
            toValue: 1,
            duration: 200,
            easing: Easing.linear
        }).start();

        // 两种状态，如果没有
        if (isFullTextShow) {
            moreIconRotateConfig = ['180deg', '0deg'];
        }

        isFullTextShow = !isFullTextShow;
        this.setState(() => ({ isFullTextShow, moreIconRotateConfig }));
    }

    componentDidMount () {
        // 当页面初始化的时候，什么也没点，箭头应该是向下的，也就是一点没旋转
        this.setState(() => ({ moreIconRotateConfig: ['0deg', '180deg']}));
    }

    render () {
        // xxx
        const { moreIconRotateValue, moreIconRotateConfig } = this.state;

        // inputRange 和 outputRange 是一一对应的关系， 0 相当于 0%， 1 相当于100%，也就是说执行到0的时候是0deg，执行到100%的时候就是180deg了。
        const rotate = moreIconRotateValue.interpolate({
            inputRange: [0, 1],
            outputRange: moreIconRotateConfig
        });

        return (
            <View style={styles.container}>
                {/* xxx */}
                <TouchableWithoutFeedback onPress={this.loadMore.bind(this)}>
                    {/* 必须有个View来包裹 */}
                    <View>
                        {/* xxx */}
                        <View style={styles.more}>
                            <Animated.View
                                {/* 这个style里的transform就是要旋转的对象 */}
                                style={{
                                    transform: [{ rotate }]
                                }}>
                                <Icon
                                    code="e049"
                                    color="#b49150"
                                    size={22}
                                />
                            </Animated.View>
                        </View>
                    </View>
                </TouchableWithoutFeedback>
            </View>
        );
    }
}
```

还是读书少啊，也踩了很多坑。最开始的时候还把初始化啥的放在`render`里面写，完全傻逼的行为啊。以后碰到初始化的时候就应该放到生命周期里面写。因为每次改变`state`的时候，都会执行`render`，导致死循环了。。

这次还用到了`ScrollView`，看文档挺多内容的，但是我啥都没写啊，只包了一个标签就好使了，这次先不写了，等有时间研究再写吧。


#### 参考
[React Native图像变换 Transforms详解](http://www.jianshu.com/p/b3cfc6b0c33f)
[【React Native开发】React Native进阶之Animated动画库详解-基础篇(64)](http://www.lcode.org/react-native%E8%BF%9B%E9%98%B6%E4%B9%8Banimated%E5%8A%A8%E7%94%BB%E5%BA%93%E8%AF%A6%E8%A7%A3-%E5%9F%BA%E7%A1%80%E7%AF%8764/)
[React Native开发之动画(Animations)](http://blog.csdn.net/hello_hwc/article/details/51775696)
[【译】详解React Native动画](https://segmentfault.com/a/1190000007621628)
[ReactNative Animated动画详解](http://www.alloyteam.com/2016/01/reactnative-animated/?utm_source=tuicool&utm_medium=referral)