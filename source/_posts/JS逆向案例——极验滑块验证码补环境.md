---
title: JS逆向案例——极验滑块验证码补环境
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-24 17:44:38
password:
summary:
tags: [JS, 逆向, 验证码]
categories: [JS逆向]

---

> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

本文是机器过极验滑块验证码系列文章的第三篇，接上篇w参数抠出来之后，补环境。后边还会陆陆续续发文，如何包括利用像素点RGB差值获取缺口位置以及通过机器学习获取缺口位置，最后会通过几个采用极验验证码的网站去完整的展示整个自动化过程。而极验滑块系列只是验证码系列的第一个系列，后边会罗列市面上常用的验证码，然后发文一一解决。

上一篇文章见：[JS逆向案例——极验滑块w参数生成](http://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E6%A1%88%E4%BE%8B%E2%80%94%E2%80%94%E6%9E%81%E9%AA%8C%E6%BB%91%E5%9D%97w%E5%8F%82%E6%95%B0%E7%94%9F%E6%88%90/)



#### 补环境过程

抠好w参数生成的代码之后，贴到VS中，先补上window和document：

```js
var window = global;
var Document = function() {};
document = new Document();
window.document = document;
```



运行之后报错，报navigator不存在

![image-20220424180831738](https://img.heshipeng.com/202204241808114.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



补上navigator：

```js
Navigator = function() {};
navigator = new Navigator();

window.navigator = navigator;
```



补好之后接着报错，document缺少方法createElement，可以看到createElement这个方法接受一个参数，并且返回为一个object。

![image-20220424181756529](https://img.heshipeng.com/202204241817808.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们给document补上createElement方法：

```js
Document.prototype.createElement = function(a) {
    if (a == 'img') return {};
}
```



运行下，竟然成功了。。。没想到补环境如此轻松。

![image-20220424183013485](https://img.heshipeng.com/202204241830790.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 利用Express封装成一个服务

将gt，challenge，c，s，版本v以及轨迹数组X1z作为参数传入，返回加密后的w参数，代码如下：

```js
var crypto = require("crypto");
var md5 = crypto.createHash("md5");
var express = require('express');
var app = express();

app.use(express.json());
app.use(express.urlencoded({ extended: true }));

var window = global;

var Document = function() {};
Document.prototype.createElement = function(a) {
    if (a == 'img') return {};
}
document = new Document();

Navigator = function() {};
navigator = new Navigator();

window.document = document;
window.navigator = navigator;


var G0b = function() {}


function H1W() {
    return (65536 * (1 + Math.random()) | 0).toString(16).substring(1);
}

var wb = H1W() + H1W() + H1W() + H1W();

G0b.prototype.wb = function() {
    return wb;
}

var _v0B;
var _n0B;
var _e7B;
var _i7B;
var _p7B;
var _I0B;

/*
	中间抠的JS代码省略
*/

function get_H7z() {
    let g0b = new G0b();
    let aaa = new _v0B();
    return aaa.encrypt(g0b.wb());
}

function get_q7z(X1z, c, s, gt, challenge, v) {
    let g0b = new G0b();
    let passtime = 0;
    for (let index = 0; index < X1z.length; index++) {
        passtime += X1z[index][2];
    }
    console.log(_e7B.u(_e7B.t(new Date().getTime(), X1z), c, s));
    let Y7z = {
        "aa": _e7B.u(_e7B.t(new Date().getTime(), X1z), c, s),
        "userresponse": _i7B.C(Math.floor(Math.random() * 200), challenge),
        "passtime": passtime,
        "imgload": Math.floor(Math.random() * 200),
        "ep": {"v": v},      
        "rp": _I0B(gt + challenge.slice(0, 32) +  passtime)         
    };
    return _n0B.encrypt(JSON.stringify(Y7z), g0b.wb());
}

function get_r7z(q7z) {
    return _p7B.Ha(q7z);
}


app.post('/geetest/w', function (req, res) {
    console.log(req.body.tracks);
    let X1z = JSON.parse(req.body.tracks);
    let c = JSON.parse(req.body.c);
    let s = req.body.s;
    let challenge = req.body.challenge;
    let gt = req.body.gt;
    let v = req.body.v;
    
    let h7z = get_H7z();
    let q7z = get_q7z(X1z, c, s, gt, challenge, v);

    let r7z = get_r7z(q7z);

    let w = r7z + h7z;
    res.send(JSON.stringify({
        w, gt, challenge
    }));
});

var server = app.listen(8081, function () {
    var port = server.address().port
    console.log("Server started, address: http://localhost:%s", port)
});
```



运行结果如下：

![image-20220424191932778](https://img.heshipeng.com/202204241919217.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)
