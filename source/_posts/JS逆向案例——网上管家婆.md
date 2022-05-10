---
title: JS逆向案例——网上管家婆
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-17 16:38:11
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]

---

> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



网址：aHR0cHM6Ly9sb2dpbi53c2dqcC5jb20uY24v

逆向目标：登录接口参数clientinfo，userName，password生成规则



先看下这几个参数长啥样：

![image-20220418163645276](https://img.heshipeng.com/202204181636387.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



userName和password长度都是128位，盲猜是AES。



全局搜索一下clientinfo，找到了clientinfo生成的地方：

![image-20220418163838625](https://img.heshipeng.com/202204181638700.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



点进去这个方法：

![image-20220418163925615](https://img.heshipeng.com/202204181639679.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



可以看到先是调用一个doGetInfo获取info信息，然后调用base64encode方法对info进行编码，直接抠出这2个方法即可。



抠出来之后，执行下doGetInfo函数，结果如下，其实就是把环境的一些参数信息用特殊符号拼接，为了不被检测出来，最好对比浏览器的实际结果，把相应的参数的值补上。

![image-20220418172659541](https://img.heshipeng.com/202204181726654.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



接着看下userName和password，全局搜索下password：

![image-20220418173455546](https://img.heshipeng.com/202204181734700.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

发现 userName和password采用的一种加密，就以password为例吧。



encryptedString加密方法接受2个参数，第一个是key，第二个是加密的明文。key的生成，往上面看几行，就会找到`var key = new RSAKeyPair();`这行代码，然后去抠出RSAKeyPair这个对象，缺啥补啥，一直抠就完事了，比较简单。



这里有一个注意的就是，下面一行代码如果漏掉了，就会陷入死循环，导致加密结果出不来：

![image-20220418174523439](https://img.heshipeng.com/202204181745538.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们在抠主要逻辑的时候，一定要看下它前后的逻辑，一些看似无关的逻辑能保留的尽量保留，比如我们要的encryptedString方法，是在postLogin方法中调用的，postLogin前边部分是一些从html表单中取值的逻辑，后边部分是发包的逻辑，除去这2部分，中间一条`setMaxDigits(129);`这个逻辑不知道是干啥用的，就尽量保留。



抠完JS并补好环境后，整个执行的结果如下：

![image-20220418175307668](https://img.heshipeng.com/202204181753715.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



补环境的代码如下：

```js
var window = {};
var document = {};

var Navigator = function() {

}

Navigator.prototype.userAgent = "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/100.0.4896.88 Safari/537.36";
Navigator.prototype.platform = "MacIntel";
navigator = new Navigator();

window = {"navigator": navigator};
```



代码获取？扫码关注微信公众号——逆向一步步，公众号内回复关键字`06`就可获取完整代码。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202204181756282.jpg)
