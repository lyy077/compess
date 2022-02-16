---
title: JS逆向之调用JS的两种方式
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-02-10 15:17:03
password:
summary:
tags: [逆向, JS]
categories: [JS逆向]
---

<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/Kca3ndEpG0s" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



###  调用JS代码的2种方式

#### 使用Python调用JS

1. PyV8

V8是Google开源的一个JavaScript引擎，被使用在了Chrome中。PyV8是V8引擎的一个Python层包装，可以用来调用V8引擎来执行JS代码。最新版本2010，已经年久失修，并且存在内存泄漏的问题，**不推荐使用**。

2. Js2Py

 Js2Py是一个纯Python实现的JavaScript解释器和翻译器。虽然2019年依然有更新，但是也是6月份的事情，而且它的issues里有很多的bug未修复(https://github.com/PiotrDabkowski/Js2Py/issues)。

解释器部分：性能不高且存在一些bug。翻译器部分：对于高度混淆的大型JS会转换失败，而且转换出来的代码可读性差，性能不高。**不推荐使用**。

3. PyMiniRacer

同样是V8引擎的包装，和PyV8效果一样。一个继任PyExecJS和PyV8的库。而且是一个较新的库，不知道有什么坑。**可以尝试**。

4. PyExecJS

一个最开始诞生于Ruby中的库，后来被移植到了Python上。有多个引擎可选，但一般我们会选择使用Node作为引擎执行代码。

缺点：执行大型JS时会有点慢，特殊编码的输入或输出参数会出现报错的情况(可以把输入或输出的参数使用Base64编码一下)。总体而言**推荐使用**

5. Selenium

一个web自动化测试框架，可以驱动各种浏览器进行模拟人工操作。用于渲染页面以方便提取数据或过验证码。也可以用来执行JS代码。

6. Pyppeteer

Puppeteer的Python版本，是第三方开发的，是一个Web自动化测试框架。原生支持以协程的方式调用，同时性能比Selenium更好。对于使用Asyncio+Aiohttp写爬虫的人而言可以直接使用。可以直接驱动浏览器来执行JS代码。

6. Playwright

微软开发的Web自动化测试框架，有多种语言的版本，支持同步与异步两种方式，也可以直接驱动浏览器来执行JS代码。

总结：如果执行的JS不是特别复杂，且不依赖浏览器环境(比如需要读取浏览器相关属性)推荐使用PyExecJS，如果执行的JS是一个比较大的工程，或者使用过程中需要读取浏览器相关属性，这时候PyExecJS已经不能满足要求，推荐使用Playwright和Playwright。



##### PyExecJS使用

环境准备：推荐安装Nodejs，安装方便且执行效率高。然后通过`pip install pyexecjs`来安装PyExecJS。

然后打开终端，执行下面2行代码：

```python
import execjs
execjs.get().name
```

如果结果如下，证明PyExecJS使用的引擎是NodeJS：

![image-20220210160058945](http://img.heshipeng.com/202202101601281.png)

如果不是，则需要手动配置一下使用的引擎，编辑系统环境变量设置如下变量即可：

```bash
export EXECJS_RUNTIME="Node"
```

配置好PyExecJS后，看一下使用代码实例：

```python
import execjs


js_text = """
function hello(str) { return str; }
"""

ctx = execjs.compile(js_text)
res = ctx.call("hello", "hello, world!")
print(res)

```

首先通过execjs的compile方法将js代码编译好之后保存在一个context中，然后调用context的call方法去执行js代码中的某一个function。



#### 使用NodeJS调用JS

简单来说就是，提供一个可以执行JS的HTTP API，然后通过调用这个API来执行JS并获取想要的结果。

1. 首先将要执行的JS单独封装成一个或者多个文件

```javascript
var add = function(a, b) {
    return a + b;
}

module.exports = {
    add
}
```

这里只是一个演示，实际可能是一段很复杂的代码逻辑。



2. 然后使用Node搭建一个Express服务

```javascript
var express = require('express')
var app = express()
var sum = require("./sum")
var bodyParser = require('body-parser')
app.use(bodyParser.json())
app.use(bodyParser.urlencoded({extended: false}))

app.get("/sum", function(req, res) {
    let params = req.query
    let a = parseInt(params.a)
    let b = parseInt(params.b)
    res.send(sum.add(a, b).toString())
})

app.listen(8081, () => {
})
```



3. 最后Python客户端去调用这个服务，拿到JS代码执行之后的结果

```python
import requests

data = {
    "a": 1,
    "b": 2
}

resp = requests.get("http://127.0.0.1:8081/sum", params=data)
print(resp.text)

```



这种方式存在的问题以及解决方案：

1. Window对象

* NodeJS没有window对象，如果要使用window对象，需要自己创建一个或者指向global
* 使用jsdom之类的库去替代

看下面代码示例：

```js
// 1. 这些对象存在于js，而不存在于nodejs，比如window，document, screen
// 2. 这些对象的属性 是一个值。

var window = {}
// window.btoa = function() ...
var document = {}

document = {"location": {"href": "https://bbs.nightteam.cn/member.php?mod=register"}}
var screen = {"width": 900, "height": 1200}
```



2. Base64

window.btoa在NodeJS中不存在，可以使用Buffer.from("Python3").toString("base64")来代替。

