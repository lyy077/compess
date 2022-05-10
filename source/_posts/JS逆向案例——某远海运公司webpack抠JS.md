---
title: JS逆向案例——某远海运公司webpack抠JS
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-03 23:28:00
password:
summary:
tags: [JS, 逆向, WebPack]
categories: [JS逆向]

---

> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

webpack抠JS的一个案例，用来熟练如何从webpack打包的JS代码中抠关键的JS代码。



#### 抠JS过程

网址：aHR0cHM6Ly9zeW5jb25odWIuY29zY29zaGlwcGluZy5jb20v

目标：登录接口的密码加密JS代码分析。



老规矩，先抓包。

![image-20220404150944421](https://img.heshipeng.com/202204041509227.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到登录请求的密码这个参数是加密的。



搜索关键词`password`，找到一个疑似加密的函数，如下：

![image-20220404162426120](https://img.heshipeng.com/202204041624199.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点进去看到加密的地方：

![image-20220404163306525](https://img.heshipeng.com/202204041633567.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



接着点进去看这个`o.a`函数：

![image-20220404163401493](https://img.heshipeng.com/202204041634530.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到，对密码采用了RSA加密，然后进行Base64编码。然后看下代码结构，很显然是按照webpack模块化编程进行组织的。



按照前面介绍的——[JS逆向之webpack扣JS思路](https://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E4%B9%8Bwebpack%E6%89%A3JS%E6%80%9D%E8%B7%AF/)——这篇文章总结的方法：

1. 找到模块加载器。

快速定位模块加载器，一般可以通过搜索`}({`，迅速定位到，但是包含加密函数的JS文件中并未找到模块加载器，没有找到的话，我们自己写一个好了。代码如下：

```js
function b(n) {
    if (e[n])
        return e[n].exports;
    var u = e[n] = {
        i: n,
        l: !1,
        exports: {}
    };
    return c[n].call(u.exports, u, u.exports, b),
        u.l = !0,
        u.exports
}
```



2. 构造自执行。

这个也比较简单，我们把上边的代码，稍微做修改：

```js
!function(c) {
    var e = {};
    function b(n) {
        if (e[n])
            return e[n].exports;
        var u = e[n] = {
            i: n,
            l: !1,
            exports: {}
        };
        return c[n].call(u.exports, u, u.exports, b),
        u.l = !0,
        u.exports
    }
    encode = b;
}({
	// TODO
});
```



3. 找到并抠出需要的模块。

抠JS就变成了一道填空题，把加密方法依赖到的模块抠出来，作为参数填到上边自执行函数中去。我们先抠出加密方法， 观察下依赖哪些模块：

```js
var r = n("XBrZ");
var t = r.pki.publicKeyFromPem(
    "-----BEGIN PUBLIC KEY-----\n MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsy4xppPDUT2eAOR5h0cyydzxtKB9O80A\n GjUT6FmDgg6CwelpnE0C2h2JQyP1gCveJs6GDwSDn20RVVpD67f//YPYErjaH/CBOxNG3k5IkW1o\n Qx04uqFNMtWvjzk0aFh2eJLsBi7Ha4elw3WySg00B8oZCL4VBay4ML9kyOAjjCj5jHCX8a2yxIMJ\n IF+EjW3kBR68IMwBvuDL45Qa0oB24vTffaSEs+hGjMTQvoCciOfti3pmEAlVc438/cBgAhK5cIMf\n IMElxYAVvmsDy0I7RCUTrajetKjX94Q+JuQUxnIHNC3IVtYsl1x0lNRtb93IhlRCkZ9djOu350eq\n hZIOXQIDAQAB\n  -----END PUBLIC KEY-----").encrypt(e, "RSA-OAEP", {
    md: r.md.sha256.create(),
    mgf1: {
        md: r.md.sha1.create()
    }
});
return window.btoa(t)
```



通过debug知e是我们填入的密码——即123456，唯一用到的模块是键为`XBrZ`的模块。我们全局搜索`XBrZ: `找到对应的模块定义：

![image-20220404165620085](https://img.heshipeng.com/202204041656123.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

点进去发现这个模块又引用了很多其它的模块，如果按照模块一个一个的抠的话，比较费时。所以我们将这整个文件中定义的模块全部抠下来，作为我们上边定义的自执行函数的参数。

![image-20220404165643735](https://img.heshipeng.com/202204041656763.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



整理完了之后，我们将代码放到浏览器中检验一下，防止出错：

![image-20220404170102015](https://img.heshipeng.com/202204041701058.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

不出意外的话就不会出意外，没有报错。



4. 导出相应的模块。

模块通过模块加载器加载，所以想要得到加密方法依赖的模块，只需要导出模块函数即可。定义一个全局变量，比如`encode`，然后将上边实现的模块加载函数赋值给这个全局变量即可。代码如下：

```js
var encode;

!function(c) {
    var e = {};
    function b(n) {
        if (e[n])
            return e[n].exports;
        var u = e[n] = {
            i: n,
            l: !1,
            exports: {}
        };
        return c[n].call(u.exports, u, u.exports, b),
        u.l = !0,
        u.exports
    }
    encode = b;
}({
    // 此处省略若干行模块函数的定义
});
```



5. 编写测试代码。

对于这个案例而言，测试代码就是那一串加密代码，看能否如期的得到类似的加密字符串，就证明我们抠的JS没有问题，测试代码如下：

```js
function get_pass(passwd) {
    var r = encode("XBrZ"); // 通过模块加载器加载XBrZ模块。
    var t = r.pki.publicKeyFromPem("-----BEGIN PUBLIC KEY-----\n MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAsy4xppPDUT2eAOR5h0cyydzxtKB9O80A\n GjUT6FmDgg6CwelpnE0C2h2JQyP1gCveJs6GDwSDn20RVVpD67f//YPYErjaH/CBOxNG3k5IkW1o\n Qx04uqFNMtWvjzk0aFh2eJLsBi7Ha4elw3WySg00B8oZCL4VBay4ML9kyOAjjCj5jHCX8a2yxIMJ\n IF+EjW3kBR68IMwBvuDL45Qa0oB24vTffaSEs+hGjMTQvoCciOfti3pmEAlVc438/cBgAhK5cIMf\n IMElxYAVvmsDy0I7RCUTrajetKjX94Q+JuQUxnIHNC3IVtYsl1x0lNRtb93IhlRCkZ9djOu350eq\n hZIOXQIDAQAB\n  -----END PUBLIC KEY-----").encrypt(passwd, "RSA-OAEP", {
        md: r.md.sha256.create(),
        mgf1: {
            md: r.md.sha1.create()
        }
    });
    return window.btoa(t)
}

console.log(get_pass("123456"));
```



运行测试代码，正常的输出了密码加密之后的密文：

![image-20220404170927135](https://img.heshipeng.com/202204041709186.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 总结

从webpack组织的JS代码中抠JS，虽然看起来比较简单，但是如果遇到复杂一点的案例，并且对JS语法不太熟的话，还是有一定的难度的的，所以需要对这一块多多练习。后边关于webpack的，还会再出一些案例，一些更加复杂的案例。



#### 关于代码的获取

扫码关注微信号——逆向一步步，公众号内回复关键词`03`就可以获取本案例的全部代码。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202202231720872.jpg)

