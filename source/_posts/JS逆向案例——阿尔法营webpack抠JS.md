---
title: JS逆向案例——阿尔法营webpack抠JS
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-06 20:35:48
password:
summary:
tags: [JS, 逆向, WebPack]
categories: [JS逆向]

---

> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

webpack抠JS的一个案例，同样用来熟练如何从webpack打包的JS代码中抠关键的JS代码。



#### 抠JS过程

网址：aHR0cHM6Ly9hZXJmYXlpbmcuY29tLw==

目标：登录接口的请求参数t和s分析。



老规矩，先抓包。

![image-20220406154058629](https://img.heshipeng.com/202204061540833.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到请求参数中username和password都是明文，t参数目测是一个时间戳，s是一串加密之后的密文。



从请求堆栈Initiator中一个个点进去，看看能否找到s生成的地方。

![image-20220406154534518](https://img.heshipeng.com/202204061545622.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点击上面的文件，发现有个报错：

![image-20220406154640011](https://img.heshipeng.com/202204061546076.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



遇到这种`Could not load content for webpack:///`的错误，需要更改下浏览器的默认配置，Settings/Preferences/Sources，去掉Enable Javascript source maps的勾选。

![image-20220406154623354](https://img.heshipeng.com/202204061546397.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



修改配置之后，就可以看到相关JS文件的代码了。找到一块疑似加密的代码，如下：

![image-20220406155132967](https://img.heshipeng.com/202204061551050.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



window.Blockey.SecuritySalt，w和n都是固定的值。

![image-20220406160129468](https://img.heshipeng.com/202204061601020.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



根据上面生成s的代码，我们整理代码如下：

```js
var window = global;
var n = "/WebApi/Users/Login";

function get_sign(username, password) {
    let x = "DUE$DEHFYE(YRUEHD*&";
    let w = "username=" + username + "&" + "password=" + password;
    return c.default((n + "?" + w + x));
}
```



然后我们的目标就很明确了，就是抠出c.default的函数，填入到我们的JS文件，我们浏览器中点进去c.default函数：

![image-20220406160747460](https://img.heshipeng.com/202204061607543.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

我们上边猜想的t是一个时间戳，不准确，可以看到t是一个时间戳加上了一个固定的数值。然后s参数是一个Sha1算法生成的密文。



我们继续点进去Sha1.hash函数：

![image-20220406161134381](https://img.heshipeng.com/202204061611573.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



然后将抠出来的JS函数一个个填充到JS文件中去，缺啥抠啥。具体详细过程不一步步展示了。最终整理的代码如下：

```js
var n = "/WebApi/Users/Login";
var c = {};
var Sha1 = {};

Sha1.hash = function(n, t) {
    // 代码较长，省略
}

Sha1.f = function(n, t, i, r) {
    switch (n) {
        case 0:
            return t & i ^ ~t & r;
        case 1:
            return t ^ i ^ r;
        case 2:
            return t & i ^ t & r ^ i & r;
        case 3:
            return t ^ i ^ r
    }
}

Sha1.ROTL = function(n, t) {
    return n << t | n >>> 32 - t
}

Sha1.utf8Encode = function(n) {
    return unescape(encodeURIComponent(n))
}
;

c.default = function(e, t) {
    var n = (new Date).getTime() + 2592e6 + (t || 3e4)
        , r = (e || "") + "&t=" + n;
    return {
        t: n,
        s: Sha1.hash(r)
    }
}

function get_sign(username, password) {
    let x = "DUE$DEHFYE(YRUEHD*&";
    let w = "username=" + username + "&" + "password=" + password;
    return c.default((n + "?" + w + x));
}

console.log(get_sign("17777777777", "123456"));
```

输出结果，跟预期一致：

![image-20220406161810100](https://img.heshipeng.com/202204061618301.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 总结

这个案例比较简单，没有什么难度，提这个案例主要目的有2点，第一就是遇到按照webpack方式组织的JS代码，不一定非得按照前面介绍的分五步走，先找模块加载器，然后编写自执行等等，一些简单的webpack可以直接去抠JS代码的。第二就是遇到``Could not load content for webpack:///`这种报错，需要去修改浏览器配置。



#### 关于代码

扫码关注微信号——逆向一步步，公众号内回复关键词`04`就可以获取本案例的全部代码。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202204061622110.jpg)
