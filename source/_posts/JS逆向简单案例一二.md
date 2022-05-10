---
title: JS逆向简单案例一二
top: false
cover: false
toc: true
mathjax: true
date: 2022-02-23 00:23:57
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]

---



<div align="middle"><iframe width="560" height="315" src="https://www.youtube.com/embed/MYFRc6q1bwI" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



通过几个简单的案例对[JS逆向从入门到放弃](https://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E4%BB%8E%E5%85%A5%E9%97%A8%E5%88%B0%E6%94%BE%E5%BC%83/)的基础篇的前6章的知识加以运用和巩固。



##### 某道翻译

网址：aHR0cHM6Ly9mYW55aS55b3VkYW8uY29tLw==

首先抓包，分析请求的类型，参数特点，返回值特点。

![image-20220223111325964](https://img.heshipeng.com/202202231113102.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

通过网络面板Fetch/XHR，找到需要的网络请求。

查看返回结果为明文，不存在加密。

```json
{"translateResult":[[{"tgt":"I love you","src":"我爱你"}]],"errorCode":0,"type":"zh-CHS2en","smartResult":{"entries":["","I love you\r\n"],"type":1}}
```



查看参数，大部分参数都是常量，只有salt, sign, Its, bv看着像是动态生成的。Its从名字和内容看，推测是13位的时间戳。sign和bv都是32位，且是字母和数字的混合，初步推算是md5加密。salt前13位跟Its一样，多了一位，推测是14位时间戳？🤪



接下来断点调试参数生成的逻辑。通常有2种方式：第一种是根据参数名称，全局搜索查找相应的JS代码；第二种是根据请求的Initiator调用栈逐步分析参数生成的位置；第三种打上XHR/fetch Breakpoints断点进行调试，然后分析Call Stack。



第一种方法，打开开发者工具的全局搜索，搜索sign关键词：

![image-20220223140405239](https://img.heshipeng.com/202202231404320.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后很轻松的找到了发送XHR请求的参数data这个object

![image-20220223140657071](https://img.heshipeng.com/202202231407464.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

鼠标悬停到generateSaltSign(t)这个方法上，然后会展示点击进入到这个方法：

![image-20220223142733567](https://img.heshipeng.com/202202231427262.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到代码比较简单，将它改写成Python代码：

```python
def generate_salt_sign(content):
    n = content[:5000]
    t = md5(navigator["appVersion"])
    r = str(int(time() * 1000))
    i = r + str(random.randint(0, 9))
    return {
        "lts": r,
        "bv": t,
        "salt": i,
        "sign": md5("fanyideskweb" + n + i + "Y2FYu%TNSbMCxc3t2u^XT")
    }
```

其中的生成sign的最后一串常量会根据用户的Cookie发生变化。我们之前的推测基本都是对的，除了salt。salt是13位时间戳加上随机的一个数字。

有了几个关键参数的生成代码，我们从Network面板复制出XHR请求的cURL，然后找一个在线的cURL to Python工具，快速生成Python代码，然后补上我们写好的generate_salt_sign方法，运行代码，结果如下：

![image-20220223145314773](https://img.heshipeng.com/202202231453817.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 某度翻译

网址：aHR0cHM6Ly9mYW55aS55b3VkYW8uY29tLw==

老规矩，先抓包。

![image-20220223150801765](https://img.heshipeng.com/202202231508799.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

找到一个XHR请求，返回结果为明文，不需要解密。参数有2个sign和token，token目测是md5。用第一种方式，打开开发者工具，全局搜索sign，发现搜索出来的结果非常多，主要是存在很多干扰，比如assign。

![image-20220223152341564](https://img.heshipeng.com/202202231524904.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这里介绍一个小技巧，如果搜索出来的干扰比较多的时候，不妨搜索“**,关键词:**”试试，为什么如此操作？因为发送XHR请求的时候，data都放在一个object中，我们知道，JS的Object格式都长这样：

```js
object = {
	key1: value,
	key2: value2
}
```

然后JS文件基本都会压缩，所以要搜索的参数自然前边有个逗号，后边有个封号。我们按照这种方式搜索，结果如下：

![image-20220223154110965](https://img.heshipeng.com/202202231541228.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到明显搜索结果更加精确了，结果也少了。然后我们逐条分析，最终定位到我们参数生成的代码地方，然后打上断点开始调试：

![image-20220223154356745](https://img.heshipeng.com/202202231543795.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

没有想到的是token竟然是一个常量。我们看他的值：

![image-20220223154444310](https://img.heshipeng.com/202202231544557.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

那就更简单了，只需要找到sign的生成方法即可。可以看到sign是调用了一个L方法生成，接受一个参数e，这个e是我们传入的要翻译的text。我们点击进入L方法：

![image-20220223154647115](https://img.heshipeng.com/202202231546855.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

好了，接下来就到了扣JS和补全JS的环节了。我们扣出来这个方法，然后用node去执行：

![image-20220223162130736](https://img.heshipeng.com/202202231621844.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

报错，i未定义，我们分析代码，补全i的定义：

![image-20220223163824429](https://img.heshipeng.com/202202231638927.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

运行接着报错，意料之中，

![image-20220223162330599](https://img.heshipeng.com/202202231623640.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

u没有定义，我们从这一行代码可以看到，u的值来自于window这个object：

![image-20220223162424396](https://img.heshipeng.com/202202231624988.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这个l是一个固定的字符串，值为"gtk"，然后u这是取得object的这个gtk属性，也是一个定植，那么我们补全这个window即可：

![image-20220223162840470](https://img.heshipeng.com/202202231628932.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

![image-20220223163853288](https://img.heshipeng.com/202202231638074.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后接着运行代码，接着报错：

![image-20220223163920006](https://img.heshipeng.com/202202231639894.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

n这个方法没有定义，我们回到源码，将n的定义拷贝进来：

![image-20220223164310865](https://img.heshipeng.com/202202231643928.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后再次运行代码，终于拿到了sign的值：

![image-20220223165713366](https://img.heshipeng.com/202202231657096.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这里很明显看到我们扣出来的JS比较大， 如果对代码进行分析然后改写成Python会比较麻烦，所以利用[JS逆向之调用JS的两种方式](https://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E4%B9%8B%E8%B0%83%E7%94%A8JS%E7%9A%84%E4%B8%A4%E7%A7%8D%E6%96%B9%E5%BC%8F/)，将JS代码用express框架做成一个服务供Python端去调用。效果如下：

![image-20220223165732284](https://img.heshipeng.com/202202231657620.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后就是比较机械性的工作了，复制cURL转成Python代码，然后修改sign通过走接口去获取，最后贴一张最终执行的效果图：

![image-20220223170807972](https://img.heshipeng.com/202202231708067.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 总结

通过几个简单的案例，回顾了下开发者面板的一些使用方式，包括全局搜索，断点调试跟踪，调用栈分析，扣JS以及Python调用JS的几种方式。代码已经上传，扫码关注公众号，公众号内回复关键字`01`，即可获取本文介绍案例的完整代码。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202202231720872.jpg)
