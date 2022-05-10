---
title: JS逆向之快速定位关键代码
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-01 22:23:00
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---

##### 快速定位之搜索

1. DOM元素或者当前页面内容的搜索

Element面板快捷键CTRL+F就可以打开搜索框，搜索DOM元素。

![image-20220313194959329](https://img.heshipeng.com/202203131949450.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



2. 全局搜索

![image-20220313195223274](https://img.heshipeng.com/202203131952349.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



全局搜索包含页面搜索

![image-20220313195307491](https://img.heshipeng.com/202203131953520.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 快速定位之断点

1. XHR断点

XHR断点只对请求内容为XHR的请求生效。在[JS逆向之Chrome浏览器工具你知多少？](http://blog.heshipeng.com/Chrome%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E5%85%B7%E4%BD%A0%E7%9F%A5%E5%A4%9A%E5%B0%91/)中介绍过通过网络面板抓包可以看到请求的类型，如下：

![image-20220314002711032](https://img.heshipeng.com/202203140027831.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

通过抓包得到XHR请求的URL之后，就可以打上XHR断点了。开发者工具切换到Source面板，展开XHR/fetch Breakpoints点击+号新建一个XHR断点。

![image-20220314003042713](https://img.heshipeng.com/202203140030770.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

XHR断点不填任何字符表示对任何XHR请求都会断上，也可以填一个完整的URL，也可以填URL的一部分，如上图。**注意：通常会去掉URL后边的参数，因为URL后边的参数可能是动态变化的，如果是动态变化的会导致每次都匹配不上，就没法断上。**



2. DOM断点

DOM断点一般指的是页面元素的断点。打DOM断点的方式是在Source面板，选中某一个元素右键选择Break on，如下图：

![image-20220314134609560](https://img.heshipeng.com/202203141346655.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

subtree modifications表示子树发生更改，attribute modifications表示节点属性值发生改变，node removal表示节点被移除。我们也第二个attribute modifications为例，更改百度首页的百度一下，将其替换为谷歌一下。如下图：

![image-20220314162541977](https://img.heshipeng.com/202203141625197.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后就成功断下了。DOM断点的应用场景在哪？比如说有一个Input输入框，然后每次点击submit按钮，Input输出框的value属性都会改变，同时把这个value值提交到后端，这个时候通过DOM断点就可以断到value值发生改变的地方，通过调用栈去查找对应改变value值的代码。



3. Event断点

Event断点在Source面板的Event Listener Breakpoints一栏。

![image-20220314170733786](https://img.heshipeng.com/202203141707913.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这里我们以鼠标的click事件为例，选择Mouse/Click事件，然后点击百度一下按钮：

![image-20220314171105093](https://img.heshipeng.com/202203141711126.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到成功断上了。



##### 快速定位之hook

1. json

```js
var my_stringfy = JSON.stringfy;
JSON.stringfy = function(params) {
    console.log("xxx", params);
    return my_stringfy(params);
}

var my_parse = JSON.parse;
JSON.parse = function(params) {
    console.log("xxx", params);
    return my_parse(params);
}
```



2. cookie

```js
var cookie_cache = document.cookie;
Object.defineProperty(document, 'cookie', {
    get: function() {
        console.log('Getting cookie');
        return cookie_cache;
    },
    set: function(val) {
        console.log('Setting cookie', val);
        var cookie = val.split(";")[0];
        var ncookie = cookie.split("=");
        var flag = false;
        var cache = cookie_cache.split("; ");
        cache = cache.map(function(a){
            if (a.split("=")[0] === ncookie[0]) {
                flag = true;
                return cookie;
            }
            return a;
        });
        cookie_cache = cache.join("; ");
        if (!flag) {
            cookie_cache += cookie + "; ";
        }
        this._value = val;
        return cookie_cache;
    }
})
```



3. window attr

```js
var window_flag_1 = "_t";
var window_flag_2 = "ccc";

var key_value_map = {};
var window_value = window[window_flag_1];

Object.defineProperty(window, window_flag_1, {
    get: function() {
        console.log("Getting", window, window_flag_1, "=", window_value);
        return window_value;
    },
    set: function(val) {
        console.log("Setting", window, window_flag_1, "=", val);
        window_value = val;
        key_value_map[window[window_flag_1]] = window_flag_1;
        set_obj_attr(window[window_flag_1], window_flag_2);
    },
});

function set_obj_attr(obj, attr) {
    var obj_attr_value = obj[attr];
    Object.defineProperty(obj, attr, {
        get: function() {
            console.log("Getting", key_value_map[obj], attr, "=", obj_attr_value);
            return obj_attr_value;
        },
        set: function(val) {
            console.log("Setting", key_value_map[obj], attr, "=", val);
            obj_attr_value = val;
        },
    });
}
```

简单测试如下：

![image-20220314174331478](https://img.heshipeng.com/202203141743582.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



4. eval/Function

```js
window.__cr_eval = window.eval;
var my_eval = function(src) {
    console.log(src);
    console.log("============eval end===========");
    return window.__cr_eval(src);
}

var _my_eval = eval.bind(null);
_my_eval.toString = window.__cr_eval.toString;
Object.defineProperty(window, 'eval', { value: _my_eval });

window.__cr_fun = window.Function;
var myfun = function() {
    var args = Array.prototype.slice.call(arguments, 0, -1).join(",").src = arguments[arguments.length - 1];
    console.log(src);
    console.log("===========Function end=========");
    return window.__cr_fun.apply(this, arguments);
}

myfun.toString = function() { return window.__cr_fun + "" }
Object.defineProperty(window, 'Function', { value: myfun })
```

  简单测试如下：

![image-20220315005512673](https://img.heshipeng.com/202203150055349.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



5. websocket

```js
WebSocket.prototype.senda = WebSocket.prototype.send;
WebSocket.prototype.send = function (data) {
    console.info("Hook WebSocket, data");
    return this.senda(data);
}
```



##### 快速定位之分析

1. Elements Event Listeners

在Elements面板，右边Event Listeners可以看到全部的事件监听器，如图：

![image-20220315010815283](https://img.heshipeng.com/202203150108324.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



2. Network type initator

在Network网络面板，请求列表的第三列是网络类型，第四列是调用栈。如下图：

![image-20220315011120666](https://img.heshipeng.com/202203150111733.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

常见的网络类型有xhr(XHR类型的请求)，gif，png，jpeg，stylesheet(css文件)，script(JS文件)，Json等。

鼠标悬浮任意一个请求的第四列，可以看到该请求的全部调用栈，如下图：

![image-20220315011435032](https://img.heshipeng.com/202203150114096.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

通常通过网络面板抓包抓到我们需要的请求，然后可以查看它的调用栈，迅速定位到相关的代码。



3. Console Log XMLHttpRequests

打开XMLHttpRequests日志的方法，点击开发者工具上的那个设置按钮：

![image-20220315012217598](https://img.heshipeng.com/202203150122662.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

找到Console那一部分，选中Log XMLHttpRequests，如下：

![image-20220315012414737](https://img.heshipeng.com/202203150124093.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

重新刷新页面，可以看到打印出了XMLRequests的请求日志了，点击可以看到调用栈：

![image-20220315012715597](https://img.heshipeng.com/202203150127672.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

