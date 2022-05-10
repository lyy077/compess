---
title: JS逆向之混淆JS手动逆向
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-08 20:24:52
password:
summary:
tags: [JS, 逆向, AST]
categories: [JS逆向]
---



>  免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

写作目的：记录手动逆向一个JS高度混淆的网站的整个过程。

网址：aHR0cHM6Ly81NTI0OTY5Ni5jb206Nzc3Ny8=



#### 逆向过程

话不多说，直接开始调试。输入用户名17777777777和密码123456点击登录，弹出验证码：

![image-20220308163635586](https://img.heshipeng.com/202203081636681.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

**一般来说网站如果出现复杂验证码都会配合JS参数加密增加防护等级**。我们抓包抓到2个XHR请求：

![image-20220308163823432](https://img.heshipeng.com/202203081638469.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

初步推测第一个请求get是获取验证码，第二个请求check是校验验证码。今天的目标就是破解这两个请求的加密参数与返回值。



 我们观察这2个接口，发现check请求的参数包含get请求的参数，所以只需要解决check请求的参数就行了。

我们直接看check请求，在Source面板打上XHR断点，断住checkv3.php请求：

![image-20220308170430516](https://img.heshipeng.com/202203081704604.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



在Call Stack中找到一个疑似加密点：

![image-20220308170529279](https://img.heshipeng.com/202203081705338.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点击进去，可以看到JS代码基本上是高度混淆的：

![image-20220308170617796](https://img.heshipeng.com/202203081706000.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们把断点断到拼接请求参数的地方：

![image-20220309105805200](https://img.heshipeng.com/202203091058349.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 逆向URL生成规则

先看下URL的生成，扣出代码：

```js
'url': _0x15a5d0[_0x59cb56(0x8bc, 0x763, 0xba1, '0jdF', 0x6bd)](_0x15a5d0[_0x59cb56(0x8ea, 0x8b8, 0x9b2, 'm*3l', 0xcfc)], _0x36b61f),
```

我们再取出`_0x15a5d0[_0x59cb56(0x8bc, 0x763, 0xba1, '0jdF', 0x6bd)]`放到console上输出，发现其是一个函数：

![image-20220309110157210](https://img.heshipeng.com/202203091101267.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点击进去，看到它是一个花指令，就是把两个参数相加：

![image-20220309110253933](https://img.heshipeng.com/202203091102623.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



所以URL参数的生成实际上是调用一个加法，把两个参数相加。我们再看这个加法传入的2个参数。`_0x15a5d0[_0x59cb56(0x8ea, 0x8b8, 0x9b2, 'm*3l', 0xcfc)]`和`_0x36b61f`。

``_0x15a5d0[_0x59cb56(0x8ea, 0x8b8, 0x9b2, 'm*3l', 0xcfc)]`是一个固定的地址。

![image-20220309110445528](https://img.heshipeng.com/202203091104606.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

`_0x36b61f`是一个13位数的时间戳。

![image-20220309110602734](https://img.heshipeng.com/202203091106206.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

所以URL实际上就是一个固定的地址拼接一个13位的时间戳，即/yzmtest/checkv3.php?t={13位时间戳}。



##### 逆向data生成规则

先扣出data生成那一部分代码：

```js
'data': _0xf828e5[_0x59cb56(0x7c7, 0x4a1, 0x425, 'F^5Z', 0x1f3)](JSON[_0x2e7c5c(0x9ff, 0x5fa, 0x813, 'Ra[M', 0x821) + _0x160f59(0x921, 0x690, 0x2a4, 'ZP*j', 0x657)](
    {
         'd': _0x15a5d0[_0x2e7c5c(0x2ce, 0x61d, 0xa5a, '4[E4', 0x33d)](_0x15a5d0[_0x160f59(0x4a2, 0x99e, 0xb7e, 'ZHhp', 0xb6b)](_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)](_0x15a5d0[_0x160f59(0x11f, 0x403, 0x29d, 'AVXK', 0x518)](_0x4fd69b[_0x8eb3(0xc7a, 0x728, 0x704, 'OBf4', 0xc4c)](''), ''), _0xf828e5[_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en']), _0x36b61f[_0x556be9(0x5cf, 0x870, 0x50b, 'Ra[M', 0x3af) + 'r'](-(0x2670 + -0x77d * -0x2 + -0x3568 * 0x1))), _0xf828e5[_0x160f59(0xae3, 0x5f9, 0x92d, 'NUpf', 0x8a5)]),
         'username': _0xf828e5[_0x2e7c5c(0x241, 0x70d, 0x75c, 'o4oN', 0x1eb) + _0x8eb3(0xac, 0x90, 0x5c9, 'o4oN', 0x3a8)],
         'password': _0xf828e5[_0x2e7c5c(0x4b3, 0x757, 0x5d1, '*EQ3', 0x86a) + _0x2e7c5c(0xb07, 0x6f1, 0xa86, 'ZP*j', 0x2bd)]
    }
))
```

在console面板上，很容易看出，username是我们输入的账号17777777777前面拼接了e5。

![image-20220309181634727](https://img.heshipeng.com/202203091816525.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

​		password则是明文：

![image-20220309181714546](https://img.heshipeng.com/202203091817805.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

​        所以上面的代码可以加以简化为：

```js
'data': _0xf828e5[_0x59cb56(0x7c7, 0x4a1, 0x425, 'F^5Z', 0x1f3)](JSON[_0x2e7c5c(0x9ff, 0x5fa, 0x813, 'Ra[M', 0x821) + _0x160f59(0x921, 0x690, 0x2a4, 'ZP*j', 0x657)](
    {
         'd': _0x15a5d0[_0x2e7c5c(0x2ce, 0x61d, 0xa5a, '4[E4', 0x33d)](_0x15a5d0[_0x160f59(0x4a2, 0x99e, 0xb7e, 'ZHhp', 0xb6b)](_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)](_0x15a5d0[_0x160f59(0x11f, 0x403, 0x29d, 'AVXK', 0x518)](_0x4fd69b[_0x8eb3(0xc7a, 0x728, 0x704, 'OBf4', 0xc4c)](''), ''), _0xf828e5[_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en']), _0x36b61f[_0x556be9(0x5cf, 0x870, 0x50b, 'Ra[M', 0x3af) + 'r'](-(0x2670 + -0x77d * -0x2 + -0x3568 * 0x1))), _0xf828e5[_0x160f59(0xae3, 0x5f9, 0x92d, 'NUpf', 0x8a5)]),
         'username': 'e517777777777',
         'password': '123456'
    }
))
```

调试下`_0x59cb56(0x7c7, 0x4a1, 0x425, 'F^5Z', 0x1f3)`

![image-20220309182331379](https://img.heshipeng.com/202203091823518.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

再调试下`_0x2e7c5c(0x9ff, 0x5fa, 0x813, 'Ra[M', 0x821) + _0x160f59(0x921, 0x690, 0x2a4, 'ZP*j', 0x657)`：

![image-20220309182359819](https://img.heshipeng.com/202203091824415.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

把上面的代码简化下：

```js
'data': _0xf828e5.enc.JSON.stringify(
    {
         'd': _0x15a5d0[_0x2e7c5c(0x2ce, 0x61d, 0xa5a, '4[E4', 0x33d)](_0x15a5d0[_0x160f59(0x4a2, 0x99e, 0xb7e, 'ZHhp', 0xb6b)](_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)](_0x15a5d0[_0x160f59(0x11f, 0x403, 0x29d, 'AVXK', 0x518)](_0x4fd69b[_0x8eb3(0xc7a, 0x728, 0x704, 'OBf4', 0xc4c)](''), ''), _0xf828e5[_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en']), _0x36b61f[_0x556be9(0x5cf, 0x870, 0x50b, 'Ra[M', 0x3af) + 'r'](-(0x2670 + -0x77d * -0x2 + -0x3568 * 0x1))), _0xf828e5[_0x160f59(0xae3, 0x5f9, 0x92d, 'NUpf', 0x8a5)]),
         'username': 'e517777777777',
         'password': '123456'
    }
))
```

可以看到，这里是用了某种加密算法对{"d": xxx, "username": "xxx",  "password": "xxx"}进行加密从而得到data。这个加密算法到底是什么加密呢？我们console面板上输入_0xf828e5.enc然后点击进去：

![image-20220309184438821](https://img.heshipeng.com/202203091844956.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

看到了整个方法有2个分支，我们分别对2个分支打上断点，发现只进去else了。如果对JS常见的加密有过了解的话，这里看到iv，mode，padding这三个关键字立马会想到用的是AES加密算法。第一个参数_0x4d04a3是待加密的字符串：

![image-20220310100612354](https://img.heshipeng.com/202203101006438.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

第二个参数this[_0x19e2aa(0x93, 0x454, 'cu5X', 0x139, 0x43d) + 'e']是密钥。我们打印出其内容：

```js
var key = {
    "words": [
        1701066809,
        929182054,
        1698117986,
        1697659188
    ],
    "sigBytes": 16
};
```

第三个参数是AES加密的一些参数，mode一般是CBC，padding不重要可以不传。最后我们在console上输出iv的值：

```js
var iv = {
    "words": [
        1668053103,
        875983984,
        1731224932,
        943273826
    ],
    "sigBytes": 16
};
```

有了AES加密的这几个参数我们就可以很简单的还原出解密算法了。代码如下：

```js
var CryptoJS = require("crypto-js");

var key = {
    "words": [
        1701066809,
        929182054,
        1698117986,
        1697659188
    ],
    "sigBytes": 16
}; // 密钥，已经转化为128bit的格式。

var iv = {
    "words": [
        1668053103,
        875983984,
        1731224932,
        943273826
    ],
    "sigBytes": 16
}; // IV，已经转化为128bit的格式。

function Decrypt(word) {
    let a = CryptoJS.AES.decrypt(word, key, { iv: iv, mode: CryptoJS.mode.CBC });
    return CryptoJS.enc.Utf8.stringify(a);
}

let data = "";
console.log(Decrypt(data));
```

还记得前面我们提到过抓到2个XHR请求吗？一个是请求验证码的，一个是进行验证码验证的。我们看第一个请求验证码的请求。

![image-20220310101530093](https://img.heshipeng.com/202203101015210.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这个返回值c是不是就是用的AES加密呢？我们用上边的解码程序试验一下。果不其然，可以正确反解出加密内容：

![image-20220310101738862](https://img.heshipeng.com/202203101017949.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

里面的一串JSON正好是我们刚才出现的验证码的内容：

```js
[
    {id: '90b389d8490d42a8', txt: '鸭子'},
	{id: 'a7985fb229d9e935', txt: '长颈鹿'},
	{id: 'c38548f7b6c0a3d8', txt: '小马'},
	{id: '4c2c8bc886b8bf8d', txt: '海马'},
	{id: '25897767b2ffc531', txt: '牛'},
	{id: '24bafe8f4a1eac0e', txt: '斑马'},
]
```

记住这个JSON，我们待会还有用。我们接着回到data破解的思路上去。data的生成代码进一步简化：

```js
'data': AES.encode(
    {
         'd': _0x15a5d0[_0x2e7c5c(0x2ce, 0x61d, 0xa5a, '4[E4', 0x33d)](_0x15a5d0[_0x160f59(0x4a2, 0x99e, 0xb7e, 'ZHhp', 0xb6b)](_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)](_0x15a5d0[_0x160f59(0x11f, 0x403, 0x29d, 'AVXK', 0x518)](_0x4fd69b[_0x8eb3(0xc7a, 0x728, 0x704, 'OBf4', 0xc4c)](''), ''), _0xf828e5[_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en']), _0x36b61f[_0x556be9(0x5cf, 0x870, 0x50b, 'Ra[M', 0x3af) + 'r'](-(0x2670 + -0x77d * -0x2 + -0x3568 * 0x1))), _0xf828e5[_0x160f59(0xae3, 0x5f9, 0x92d, 'NUpf', 0x8a5)]),
         'username': 'e517777777777',
         'password': '123456'
    }
))
```

接下来看看最核心的d的生成规则。\_0x15a5d0[_0x2e7c5c(0x2ce, 0x61d, 0xa5a, '4[E4', 0x33d)]是一个加法的花指令。

![image-20220310102400570](https://img.heshipeng.com/202203101024724.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

\_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)]也是一个加法的花指令：

![image-20220310102640758](https://img.heshipeng.com/202203101026613.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

\_0x15a5d0[_0x59cb56(0x92b, 0x683, 0x968, 'F^5Z', 0x9a7)]也是加法花指令。

![image-20220310102715978](https://img.heshipeng.com/202203101027076.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

\_0x15a5d0[_0x160f59(0x11f, 0x403, 0x29d, 'AVXK', 0x518)]依旧是一个加法花指令。

![image-20220310102811012](https://img.heshipeng.com/202203101028062.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

\_0x4fd69b[_0x8eb3(0xc7a, 0x728, 0x704, 'OBf4', 0xc4c)]是内置的join方法。

![image-20220310102934544](https://img.heshipeng.com/202203101029626.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

_0x4fd69b是个字符串：

![image-20220310105915453](https://img.heshipeng.com/202203101059338.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en'是字符串$strlen

![image-20220310103214178](https://img.heshipeng.com/202203101032240.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

\_0xf828e5[_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en']则是取\_0xf828e5这个object的$strlen属性，这里的值是3。

_0x556be9(0x5cf, 0x870, 0x50b, 'Ra[M', 0x3af) + 'r'这里是substr。

![image-20220310104111880](https://img.heshipeng.com/202203101041001.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

_0x36b61f是13位的时间戳。

![image-20220310104215390](https://img.heshipeng.com/202203101042436.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

-(0x2670 + -0x77d * -0x2 + -0x3568 * 0x1)是固定的常量，-2。

![image-20220310104254188](https://img.heshipeng.com/202203101042243.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

_0x160f59(0xae3, 0x5f9, 0x92d, 'NUpf', 0x8a5)是字符串$ver。这个ver其实在我们刚才解码第一个请求的返回值的里面就有了，值为3587，跟这里的吻合。

![image-20220310110046938](https://img.heshipeng.com/202203101100041.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

最终简化代码如下：

```js
'data': AES.encode(
    {
        'd': add(add(add(add("4c2c8bc886b8bf8d".join(''), ''), _0xf828e5.$strlen), _0x36b61f.substr(-2)), _0xf828e5.$ver),
        // 'd': "4c2c8bc886b8bf8d" +  _0xf828e5.$strlen + _0x36b61f.substr(-2)) + _0xf828e5.$ver
        'username': 'e517777777777',
        'password': '123456'
    }
))
```

到这里d的生成算法基本上一目了然了。d=字符串(这里是4c2c8bc886b8bf8d)+_0xf828e5.$strlen(这里是3)+13位时间戳的最后2位(这里是17)+版本号(第一个请求接口有返回为3587)=4c2c8bc886b8bf8d3173587。扣出d的生成代码，在console上输出，验证下：

![image-20220310111442039](https://img.heshipeng.com/202203101114177.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

完全吻合。现在唯一的问题是\_0x4fd69b这个字符串怎么来的以及_0xf828e5.$strlen这个值怎么算出来的，解决了这两个问题，d的生成规则就破解了。

我们手动搜索_0x4fd69b这个字符串，总共找到三处：

![image-20220310112140608](https://img.heshipeng.com/202203101121754.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

扣出第二处的代码如下：

```js
_0x4fd69b[_0x219591('FZa9', 0x79c, 0x826, 0x6a9, 0x45f)](_0xf828e5[_0x59b553('W]B)', 0xd30, 0x809, 0x809, 0x2c4)][_0x19f5d9[_0x3ebe03('nCyg', 0xb6c, 0x868, 0x4ee, 0xd78)](parseInt, _0x1bf4a9[_0x59b553('#og4', 0xbb9, 0x8a2, 0x953, 0x441) + 'ce'](_0x19f5d9[_0x3ebe03('F^5Z', 0x448, 0xc1, -0x2ba, 0x8c)], ''))]['d']);
```

按照上边提供的方法，在console上分段调试代码含义，代码反混淆如下(<font style="color: red;">为节省篇幅，从这里开始，代码反混淆过程都不会写了，直接给出反混淆的结果</font>):

```js
_0x4fd69b.push(_0xf828e5.$list[parseInt("btncanv_3".replace("btncanv_", ""))]['d']);
```

这个代码的意思就是取上边我们提到的验证码数组中索引为3的值，即4c2c8bc886b8bf8d，把这个值push到数组_0x4fd69b（这里虽然取得是d这个属性，但是实际上d属性跟id属性的值是一样的，\_0x281004['d'] = _0x3aa5c6['id']）。这里的这个btncanv_3恰好是验证码的答案，即正确答案的元素的id。

![image-20220310140550569](https://img.heshipeng.com/202203101405702.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

那么这个`btncanv_3`是怎么确定的呢？是我们手动点选验证码图案的时候选中的，我们手动选中了验证码图案，JS代码会根据我们选中的图案，拿到它的id（如果有选中了多个，也只取第一个），然后进行JS加密，后端会根据相应的解密算法，拿到我们上传的那个验证码。我们开篇提到过“一般来说网站如果出现复杂验证码都会配合JS参数加密增加防护等级”，这里就验证了这句话。**我们这里虽然破解了验证码验证接口表单数据的加密算法，但是验证码的点选，我们还需要辅助相应的验证码识别的算法，帮助我们完成验证码的识别与点选**。这里主要是讲解手动反混淆方案，验证码后边会有专门的专题文章进行介绍，先埋一个坑后边补上。



接下来看下另外一个_0xf828e5.$strlen的生成规则。我们文件中全局搜索'en'（为什么要搜索这个？因为前面$strlen的字符串混淆是\_0x8eb3(0x136, 0x44f, 0x748, 'HZxj', 0x2a3) + 'en'），果然被我们找到了。代码如下：

```js
this[_0x2a85c2(0x8a2, 0x60c, 0xa3d, 0x7b0, '0NjW') + 'en'] = _0x15a5d0[_0x19b874(0xc8f, 0xe5a, 0x13b9, 0xb8b, 'W]B)')](Math[_0x26b2f7(0xf44, 0xc7b, 0xbc1, 0x1077, 'o4oN')](_0x15a5d0[_0x26b578(0xfa3, 0xcfb, 0x820, 0x11b8, 'm*3l')](-0x1bcd + 0x1 * 0x1b23 + -0xaf * -0x1, Math[_0x2a85c2(0x11fb, 0xf8b, 0xd69, 0x137a, '*EQ3') + 'm']())), -0x727 * 0x2 + 0x24d7 + -0x1f * 0xba)
```

代码反混淆之后整理如下：

```js
this.$strlen = Math.floor(5 * Math.random()) + 3
```

 至此，整个data数据生成过程调试完了。最终的算法伪代码整理如下：

```js
let _0xf828e5.$strlen = Math.floor(5 * Math.random()) + 3;
let _0x36b61f = new Date().getTime();
let _0xf828e5.$ver = "3587";
let code = [
    {id: '90b389d8490d42a8', txt: '鸭子'},
	{id: 'a7985fb229d9e935', txt: '长颈鹿'},
	{id: 'c38548f7b6c0a3d8', txt: '小马'},
	{id: '4c2c8bc886b8bf8d', txt: '海马'},
	{id: '25897767b2ffc531', txt: '牛'},
	{id: '24bafe8f4a1eac0e', txt: '斑马'},
]; // 这个验证码的JSON从第一个接口中拿

// AES的密钥以及IV值上边已经给出
'data': AES.encode(
    {
        'd': code[/*人工选中的第一个验证码的索引*/].id +  _0xf828e5.$strlen + _0x36b61f.substr(-2) + _0xf828e5.$ver
         'username': 'e517777777777',
         'password': '123456'
    }
))
```



##### 逆向clientid生成规则

扣出clientid生成相关的代码：

```js
    this[_0x26b2f7(0x9fc, 0xa9b, 0xe1c, 0x6b2, '%jat') + _0x34ce5a(0x1321, 0xfb2, 0x11d0, 0x109c, 'm*3l')] = _0x1c3499[_0x2a85c2(0xf23, 0xc3d, 0x99d, 0xf50, 'OBf4')]('')[_0x26b2f7(0xf47, 0xb49, 0xe76, 0xc02, 'W]B)') + 'r'](0x4f * -0x1 + 0x1779 + -0x172a, -0x4ff * 0x3 + -0x3 * -0x75c + 0x1 * -0x70d)
```

代码反混淆如下：

```js
this.$clientid = _0x1c3499.join("").substr(0, 10)
// _0x1c3499 = ['54mwjp6', 8, 'zvc']
```

$clientid的生成规则依赖\_0x1c3499，我继续往下看，扣出\_0x1c3499的相关代码：

```js
_0x1c3499 = []
_0x1c3499[_0x26b578(0xc37, 0xd61, 0x122c, 0xeeb, 'g(lc')](_0x3e49dd[_0x2a85c2(0x1ea, 0x608, 0x810, 0xf3, 'o4oN') + 'r'](-0x5f * 0x1b + -0x1 * 0x15c1 + 0x1fc6, _0x15a5d0[_0x26b578(0xee3, 0xb7c, 0x893, 0xddb, 'HZxj')](_0x3badf1, -0x1 * -0x14e3 + 0x26 * -0x56 + 0x81e * -0x1))),
_0x1c3499[_0x34ce5a(0x10ca, 0xef3, 0xa62, 0x10c1, '[tJe')](_0x3badf1),
_0x1c3499[_0x19b874(0x4bb, 0x55a, 0x35d, 0x7ce, 'R[NP')](_0x3e49dd[_0x2a85c2(0x11c, 0x5da, 0xde, 0x229, 'FrGG') + 'r'](_0x3badf1, _0x3e49dd[_0x26b578(0xf55, 0xbca, 0x95d, 0xd00, '0jdF') + 't'])),
```

代码反混淆如下：

```js
_0x1c3499 = [];
_0x1c3499.push(_0x3e49dd.substr(0, _0x3badf1 - 1));                //_0x1c3499.push(54mwjp6)
_0x1c3499.push(_0x3badf1);										   //_0x1c3499.push(8)
_0x3e49dd.push(_0x3e49dd.substr(_0x3badf1, _0x3e49dd.length));     //_0x1c3499.push(zvc)
// _0x3e49dd = "54mwjp6tzvc"
// _0x3badf1 = 8
```

可以看到\__0x1c3499的值又依赖\_0x3e49dd和\_0x3badf1。我们再扣出相应的代码：

```js
_0x3e49dd = _0x15a5d0[_0x2a85c2(0x240, 0x6a9, 0x570, 0x230, 'ZP*j')](Number, _0x15a5d0[_0x26b578(0x8cc, 0xaab, 0xa86, 0xf1d, 'NUpf')](Math[_0x19b874(0x12ba, 0xd7c, 0xb35, 0xe72, 'o4oN') + 'm']()[_0x34ce5a(0xffa, 0xa91, 0x802, 0xe92, 'uUCz') + _0x19b874(0x174, 0x684, 0x9d7, 0x737, 'G0Im')]()[_0x26b2f7(0xaa8, 0x6bd, 0x562, 0x35a, 'F^5Z') + 'r'](0x10 * 0xc2 + 0x6d * -0x5 + -0x9 * 0x11c, 0xb3 * 0x35 + 0x13 * 0x1a5 + -0x444a * 0x1), Date[_0x26b2f7(0xbf3, 0xd8c, 0xaf2, 0xacf, 'g(lc')]()))[_0x26b578(0x326, 0x69f, 0xaa3, 0x78d, '*EQ3') + _0x26b2f7(0x8dc, 0x900, 0x8f0, 0xa3a, 'hROy')](0x143b + 0x53c + -0x1953)
                      
 _0x3badf1 = _0x15a5d0[_0x34ce5a(0x11ce, 0xdf7, 0x9a2, 0xad1, '!OnF')](parseInt, _0x3577fe[Math[_0x19b874(0x871, 0xb0b, 0xf76, 0xb41, '#og4')](_0x15a5d0[_0x2a85c2(0x123a, 0xeb6, 0xbf1, 0x10d4, 'bsj&')](Math[_0x2a85c2(0x7ca, 0x776, 0x9aa, 0x847, 'R[NP') + 'm'](), _0x3577fe[_0x26b2f7(0x2ac, 0x73b, 0xbfc, 0x1ff, 'FrGG') + 'h']))])
```

代码反混淆后结果如下：

```js
_0x3e49dd = Number(Math.random().toString().substr(3, 4) + Date.now().toString()).toString(36)
_0x3badf1 = parseInt(_0x3577fe[Math.floor(Math.random() * _0x3577fe.length)])
```

\_0x3577fe的值是固定的三个元素的数组，如下：

```js
_0x3577fe = [-0x2090 + -0x1b * 0x139 + 0x4197, 0x959 + -0x248c + 0x45 * 0x65, 0x2 * -0x1ea + 0x229f + -0x1ec3]; // 4 6 8
```

到这里为止，$clientid的生成规则就全部反混淆出来了，最终的代码整理如下：

```js
let _0x3577fe = [4, 6, 8];
let _0x3e49dd = Number(Math.random().toString().substr(3, 4) + Date.now().toString()).toString(36);
let _0x3badf1 = parseInt(_0x3577fe[Math.floor(Math.random() * _0x3577fe.length)]);

let _0x1c3499 = [];
_0x1c3499.push(_0x3e49dd.substr(0, _0x3badf1 - 1));
_0x1c3499.push(_0x3badf1);
_0x1c3499.push(_0x3e49dd.substr(_0x3badf1, _0x3e49dd.length));

let clientid = _0x1c3499.join("").substr(0,10);
console.log(clientid);
```

真是一层一层剥开你的心🥴



#####  逆向token生成规则

扣出token生成的相关代码：

```js
'token': _0xf828e5[_0x2e7c5c(0x3c9, 0x4c, -0x2fc, 'hROy', -0x1f8)](_0x15a5d0[_0x2e7c5c(0x78e, 0x6f8, 0x927, '(e@x', 0x3d0)](_0x15a5d0[_0x59cb56(0x398, 0x6f8, 0x7eb, '(e@x', 0x571)](_0x15a5d0[_0x160f59(0xbca, 0xad5, 0x7c2, 'FZa9', 0xa2c)](_0xf828e5[_0x2e7c5c(0x56b, 0x25a, 0x1e6, 'g(lc', -0x22c) + _0x556be9(0x527, 0x2b, 0x30c, 'su5h', 0x58f)], _0xf828e5[_0x8eb3(0x1bf, 0x17c, 0x58, 'Ra[M', -0xec) + _0x2e7c5c(0x203, 0x237, -0x41, 'Qm)6', 0x40c)]), _0xf828e5[_0x2e7c5c(0xd48, 0x8de, 0x717, 'j[vi', 0xd5e) + _0x59cb56(0x301, 0x4c3, 0x1e5, '0jdF', 0x8fb)]), _0x15a5d0[_0x59cb56(-0x1ee, 0xb3, 0x35, 'OBf4', 0x524)]))
```

反混淆之后的最终代码如下：

```js
'token': _0xf828e5["sign"](_0xf828e5.$clientid + _0xf828e5.$username + _0x15a5d0.sGZgF))
```

庆幸的是，这里的_0x15a5d0.sGZgF是一个固定的字符串，内容为"x045783"。所以重点在于破解这个加密方法sign。我们扣出相应的代码：

```js
_0x58b6e0[_0x339e58(0x61a, -0x24f, 0x94, 'nCyg', 0x267) + _0x18c3fa(0x813, 0x825, 0x36f, 'TEE1', 0x6be)][_0x1d90d1(0x18a, 0x8b2, -0x158, 'Qm)6', 0x348)] = function(_0x1c6621) {
    // ...此处省略若干行
    var _0x4f239d = []
    , _0x4996c2 = cjs[_0x8a0b67('FrGG', 0x9dc, 0x3fa, 0xaba, 0x868)](_0x15a5d0[_0x343f8b('ZiBy', 0x377, 0x152, 0x542, 0x4df)](_0x1c6621, _0x46b6c3[_0x8a0b67('TEE1', 0x3db, 0x7c8, 0x6a4, 0x347) + _0x429176('NUpf', 0x241, -0x1c1, 0x4ab, 0x81)][_0x343f8b('HZxj', 0x49c, 0x718, 0xa22, 0x854)][_0x429176('F^5Z', 0x472, 0x2c9, 0x687, 0x272) + _0x8a0b67('g(lc', 0x39e, 0x19b, 0x616, 0x5b8) + 'e']()))[_0x4b55c2('OBf4', -0x22b, -0x342, 0x13a, 0x2c) + _0x8a0b67('xVxp', -0x78, 0x45c, 0x287, 0x16b)]();
    return _0x4f239d[_0xafe698('Ra[M', 0x1d0, -0x66b, 0x2ad, -0x177)](_0x4996c2[_0xafe698('IF#P', 0x2d0, 0x958, 0x506, 0x55e) + 'r'](-0x15d + -0x49 * -0x45 + -0x1246, 0x8c2 + 0x25af + -0x2e6c)),
        _0x4f239d[_0x8a0b67('g(lc', 0x34a, 0x43e, 0x955, 0x683)](_0x4996c2[_0x4b55c2('ZiBy', 0x71, 0x10f, 0x9a7, 0x506) + 'r'](0x142b * -0x1 + 0x21b7 + 0xd85 * -0x1, -0xb69 * -0x3 + -0x61 * 0x58 + -0xde)),
        _0x4f239d[_0x429176('W]B)', 0x82f, 0x119, 0x27e, 0x4fb)](_0x4996c2[_0x4b55c2('AVXK', 0x3e8, 0x4fc, 0x6f2, 0x79f) + 'r'](0x61b + 0xc * -0x11e + 0x75c, -0x677 * 0x1 + -0x2272 * 0x1 + 0x28ee)),
        _0x4f239d[_0xafe698('0NjW', 0x3c5, 0x181, -0x47, 0x125)](_0x4996c2[_0x8a0b67('uUCz', 0xcea, 0x99c, 0x405, 0x8c3) + 'r'](-0x4ba + -0x59d * 0x1 + 0xa6b, -0x2 * 0xe80 + 0x1c01 + 0x104)),
        _0x4f239d[_0x429176('IF#P', 0x2b, -0x2cb, 0x33e, 0x20d)](_0x4996c2[_0xafe698('TEE1', 0xcb6, 0x668, 0x5fa, 0x833) + 'r'](-0x4b * -0x4 + -0x2a * 0x86 + 0x5 * 0x42e, -0xd * -0x119 + -0x1791 + 0x951)),
        _0x4f239d[_0x429176('j[vi', 0x5e6, 0x2bd, 0x5e2, 0x648)](_0x4996c2[_0x429176('#og4', 0xd5, 0x68e, 0x54b, 0x516) + 'r'](-0x193d + 0x399 * -0x5 + -0x1 * -0x2b55, -0x8a8 * -0x4 + 0xd6 * 0x6 + 0x3 * -0xd35)),
        _0x4f239d[_0x8a0b67('Cy2U', 0x689, 0x578, 0x485, 0x71e)](_0x4996c2[_0x4b55c2('4[E4', 0x272, 0x5ce, 0x4e1, 0x668) + 'r'](-0x97 * 0x17 + -0xe * 0x46 + 0x1166, -0x18f4 * -0x1 + -0x93 * 0x3b + 0x8ef * 0x1)),
        _0x4f239d[_0x343f8b('&zHf', -0xb, -0x5bc, -0x16b, -0x108)]('');
}
```

看到这么大一段代码不要慌！！！我们慢慢开始剥洋葱。🙃，剥到最后，很简单。

```js
_0x58b6e0.prototype.sign = function(_0x1c6621) {
    var _0x4f239d = [], _0x4996c2 = MD5(_0x1c6621 + document.location.href.toLowerCase()).toString();
    return _0x4f239d.push(_0x4996c2.substr(10, 5)), _0x4f239d.push(_0x4996c2.substr(7, 5)), _0x4f239d.push(_0x4996c2.substr(15, 5)), _0x4f239d.push(_0x4996c2.substr(20, 5)), _0x4f239d.push(_0x4996c2.substr(22, 5)), _0x4f239d.push(_0x4996c2.substr(27, 5)), _0x4f239d.push(_0x4996c2.substr(1, 2)), _0x4f239d.join("");
}
```

其实就是对传入的参数做了一个md5的加密，然后进行乱序处理。



到这里我们两个接口的所有请求参数加密算法，以及接口的返回值的解密算法都已经破解了。我们接下来简单验证下是否正确。



####  验证

由于check接口需要机器学习对验证码进行识别，所以这里只验证get接口的参数。用NodeJS的Express框架搭建好获取token的服务，供Python调用。

运行结果如下：

![image-20220312220733810](https://img.heshipeng.com/202203122207679.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

![image-20220312220820149](https://img.heshipeng.com/202203122208606.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 总结

本文通过一个网站，手动对AST混淆代码进行了一个反混淆。在[JS代码安全防护原理——AST混淆原理](https://blog.heshipeng.com/JS%E4%BB%A3%E7%A0%81%E5%AE%89%E5%85%A8%E9%98%B2%E6%8A%A4%E5%8E%9F%E7%90%86%E2%80%94%E2%80%94AST%E6%B7%B7%E6%B7%86%E5%8E%9F%E7%90%86/)中提到的几种混淆原理基本都出现过了。比如数组混淆，数组逆向，花指令，流程平坦化，逗号表达式混淆，字符串加密，常量加密等。可以看到，手动混淆的过程是极其容易出错，工作量非常大且十分痛苦的。接下来会写相关文章，介绍如何通过工具对AST混淆代码进行自动反混淆，不过，手动混淆这种能力也是必须要掌握的，万一你使用的工具失效了，或者说遇到一些更加特殊的网站，只能通过手动混淆呢？如果想获取本文的完整代码，扫码关注公众号，然后公众号内回复关键字`02`即可获取。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202203122210618.jpg)
