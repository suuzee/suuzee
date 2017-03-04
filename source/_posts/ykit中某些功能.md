---
title: ykit中某些功能
date: 2017-03-04 22:13:40
categories: ykit
tags:
    - ykit
    - 编译工具
description: 了解Redux
---

`ykit`: [http://ued.qunar.com/ykit/](http://ued.qunar.com/ykit/) <!--more-->

#### 不同环境可以设置不同的url。目前不支持beta；
配置：`ykit.config.js`
```javascript
var envUrls = require('./config/url');
switch (this.env) {
    case 'local':
        baseConfig.plugins.unshift(new this.webpack.DefinePlugin(envUrls.local));
        break;
    case 'dev':
        baseConfig.plugins.unshift(new this.webpack.DefinePlugin(envUrls.dev));
        break;
    case 'prd':
        baseConfig.plugins.unshift(new this.webpack.DefinePlugin(envUrls.prd));
        break;
}
```
urls: `./config/url.js`
```javascript
let urls = {
    card: '/fp/card.jsp'
};


// 环境url
let envUrls = {
    local: {
        card: {
            ajaxCardDeleteCardUrl: urls.card + '?mockAction=deleteCard',
            ajaxCardBindCardViewUrl: urls.card + '?mockAction=bindCardView',
            ajaxCardBindCardUrl: urls.card + '?mockAction=bindCard',
            ajaxCardGetCardUrl: urls.card
        },
        index: {
            ajaxIndexPaySyncUrl: '/wappay/cardinfoinput?mockAction=local'
        }
    },
    dev: {
        card: {
            ajaxCardDeleteCardUrl: urls.card,
            ajaxCardBindCardViewUrl: urls.card,
            ajaxCardBindCardUrl: urls.card,
            ajaxCardGetCardUrl: urls.card
        },
        index: {
            ajaxIndexPaySyncUrl: '/wappay/cardinfoinput'
        }
    },
    prd: {}
}

let local = mergeObject(envUrls.local);

let dev = mergeObject(envUrls.dev);

// 正常情况是 let prd = mergeObject(envUrls.prd);
let prd = dev;


/**
 * [mergeObject 将对象中的属性合并]
 */
function mergeObject (obj = {}) {
    let temp = {};
    for (let p in obj) {
        Object.assign(temp, obj[p]);
    }
    temp = stringProperty(temp);
    return temp;
}
/**
 * [stringProperty 将对象属性包裹JSON.stringify，如果不包裹可能会认为属性是一个正则表达式]
 * @param  {Object} obj [description]
 * @return {[type]}     [description]
 */
function stringProperty (obj = {}) {
    for (let p in obj) {
        obj[p] = JSON.stringify(obj[p]);
    }
    return obj;
}

module.exports = {local, dev, prd};
```
PS：由于有些url带 `/` ，所以可能会被解析成一个正则表达式，所以要给url包一层 `JSON.stringify` .

#### 通过 `jerry` 实现mock

`jerry`: [https://github.com/Ellery0924/Jerry](https://github.com/Ellery0924/Jerry)

可以通过 `jerry` 实现mock，并且可以通过一个url，传入不同的参数实现返回不同的 `json`.在项目根目录创建 `mock.js`
```javascript
module.exports = {
    {
        pattern: /\/xxx\/yyy/,
        responder: require('./mock/yyy.js')
    },
    {
        pattern: /\/xxx\/zzz/,
        responder: require('./mock/zzz.json')
    }
}
```
`responder` 支持返回一个方法，支持直接返回一个 `json` 对象等。

`responder` 返回的方法有两个参数：第一个参数是请求的 `url` 的解析之后的对象，这个对象就像是 `window.location` 这个对象。第二个参数是 `body`，即 `response.body`，是一个字符串，格式类似于 `window.location.search`，没有前面的 `?` .

`./mock/yyy.js`

```javascript
module.exports = function (parserUrl, data) {
    var action = '';
    var json = {};
    try {
        action = getAction(parserUrl, data);
    } catch (e) {
    }
    if (action) {
        if (action === 'a') {
            json = aJson;
        } else if (action === 'b'){
            json =  bJson;
        } else {
            json = defaultJson;
        }
    } else {
        json =  defaultJson;
    }
    return json;
}


function getAction (urlObj, data) {
    var action = '';
    var actionGet = '';
    var actionPost = '';

    if (!!urlObj.search) {

        try {
            actionGet = queryToJson(urlObj.search).action;
            if (!!actionGet) {
                action = actionGet;
            }
        } catch (e) {}

    } else {

        try {
            actionPost = queryToJson(data).action;
            if (actionPost) {
                action = actionPost;
            }
        } catch (e) {}

    }
    return action;
}

function queryToJson (searchString) {
    var params = searchString.replace(/\?/, '').split('&') || [];
    var paramObj = {};
    for (var i = 0, len = params.length; i < len; i ++) {
        var param = params[i].split('=');
        paramObj[param[0]] = param[1];
    }
    return paramObj;
}

var defaultJson = {
    data: 'defaultjson'
};

var aJson = {
    data: 'ajson'
};

var bJson = {
    data: 'bjson'
};
```

这种是针对一种场景：请求同一个url，但是请求的参数不同，返回的json也就不同。上述是请求 `/xxx/yyy`，如果参数中 `action` 为 `a`，则返回 `aJson`，如果参数中 `action` 为 `b`，则返回 `bJson`。
