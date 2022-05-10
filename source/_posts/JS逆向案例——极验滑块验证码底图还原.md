---
title: JS逆向案例——极验滑块验证码底图还原
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-20 10:41:37
password:
summary:
tags: [JS, 逆向, 验证码]
categories: [JS逆向]
---



> 免责声明：**本文章中所有内容仅供学习交流，抓包内容、敏感网址、数据接口均已做脱敏处理，严禁用于商业用途和非法用途，否则由此产生的一切后果均与作者无关，若有侵权，请联系我立即删除！**



#### 前言

本文是机器过极验滑块验证码系列文章的第一篇，底图的还原，包括带缺口的底图以及完整底图的还原。后边还会陆陆续续发文，讲解提交验证过程请求w参数的跟值，如何补环境，如何包括利用像素点RGB差值获取缺口位置以及通过机器学习获取缺口位置，最后会通过几个采用极验验证码的网站去完整的展示整个自动化过程。而极验滑块系列只是验证码系列的第一个系列，后边会罗列市面上常用的验证码，然后发文一一解决。



#### 逆向分析

网址：aHR0cHM6Ly93d3cudGlhbnlhbmNoYS5jb20v

以天眼查的登录为例，在进行滑块验证时，进行抓包分析。



##### 抓包

将一些重要的请求罗列如下：

1. geetest.xhtml

传入的是一个13位时间戳的uuid

![image-20220420114316016](https://img.heshipeng.com/202204201143166.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



返回的是challenge和gt。

![image-20220420114408595](https://img.heshipeng.com/202204201144667.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



2. gettype.php

获取验证码类型，传入的是第一个请求返回的gt。

![image-20220420115339355](https://img.heshipeng.com/202204201153409.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



返回的是验证码类型以及一些JS文件：

![image-20220420115438377](https://img.heshipeng.com/202204201154420.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



3. get.php

入参同样是前边返回的challenge和gt。

![image-20220420123131763](https://img.heshipeng.com/202204201231886.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



返回乱序之后的完整验证码底图和缺口验证码底图，同时返回了一个新的challenge。

![image-20220420150236107](https://img.heshipeng.com/202204201502291.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



4. ajax.php

这个是我们手动进行验证码提交时发的包，重要的入参是gt，challenge以及w。其中w是加密字符串，包含滑块的轨迹。challenge是更新之后的那个新的值，即上一步get.php获取到的。

![image-20220420150610485](https://img.heshipeng.com/202204201506631.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



返回滑块是否验证成功，如下：

![image-20220420150645777](https://img.heshipeng.com/202204201506832.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 逆向目标确定

经过上边的抓包分析之后，可以发现几个关键点，首先是几个关键参数：challenge，gt，w，然后是乱序的完整底图和乱序的带缺口底图，最后是缺口位置的识别。

* challenge和gt自始自终都是通过请求接口返回，所以这个不需要逆向。

* w参数是最后一步向服务端提交验证码验证结果的值，包含了环境监测，滑块轨迹的加密。如果使用playwright这种模拟浏览器工具去提交验证码，这个参数可以不用逆向，如果是自己走JS或者Python发包，则需要逆向刨一下算法是怎么写的。

* 由于无法直接的获取正确的完整底图和带缺口的底图(有些网站上极验验证正确的底图可能是用canvas加载)，所以需要进分析如何从无序的底图变为有序的底图。
* 缺口位置的识别，有2种方案。一种是通过比较完整底图和缺口底图，利用像素点的RGB像素差值，来判断缺口位置；另一种是通过机器学习进行训练，然后得到一个模型用于识别缺口位置。

上面几点，不管如何偷懒，第三步底图的还原是必然需要的，不然没办法计算缺口位置，没办法生成轨迹，自然没办法进行验证。所以底图还原是整个滑块验证的第一步。



##### 底图缺口还原

我们先看下滑块的大小：

![image-20220420163229256](https://img.heshipeng.com/202204201632353.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



图片大小 w = 260px, h = 116px。我们点击图片选择审查元素。



可以看到底图是由52个div组成，每个div的w = 10px，h = 58px。分为上下两个半区，每个半区26个div。刚好组成260px * 116px的矩形验证码。

![image-20220420163621256](https://img.heshipeng.com/202204201636328.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们看第一个div，即上半区左上角的第一个div，background-position = -157px -58px。表示将background-image向左偏移157个像素，向上偏移58个像素，作为第一个div放在上半区最左边。由于前面分析过，每个div的宽是10px，高是58px。所以第一个div四个顶点在background-image上的相对坐标是(157, 58), (167, 58), (157, 116), (167, 116)。



同理，我们推测上半区第二个div的四个顶点的相对坐标分别是(145, 0), (155, 0), (145, 58), (155, 58)。



此外，background-image就是我们抓包分析的第三步获取到的乱序图。



知道了每一个个div的坐标，以及乱序的背景图，就可以通过从乱序图上裁剪出一个个div，然后再拼接到一起，这样不就构成了正确有序的图片？



##### 编程实现

```python
import math
from PIL import Image


div_offset = [
    {"x": -157, "y": -58}, 
    # 省略若干行 
    {"x": -205, "y": 0}
]


def restore_pic(pic_path, new_pic_path):
    unordered_pic = Image.open(pic_path)
    ordered_pic = unordered_pic.copy()

    # 裁剪并拼接
    for i, d in enumerate(div_offset):
        im = unordered_pic.crop((math.fabs(d['x']), math.fabs(d['y']), math.fabs(d['x']) + 10, math.fabs(d['y']) + 58))
        # 上半区
        if d['y'] != 0:
            ordered_pic.paste(im, (10 * (i % (len(div_offset) // 2)), 0), None)
        else:
            ordered_pic.paste(im, (10 * (i % (len(div_offset) // 2)), 58), None)

    ordered_pic.save(new_pic_path)


if __name__ == '__main__':
    restore_pic("img.png", "new_img.png")

```

说一下用到的PIL库的几个方法：copy表示复制一张图片；crop表示以矩形区域裁剪，入参是一个四个元素的元组，分别是矩形左上角顶点的x坐标，左上角顶点的y坐标，右下角顶点的x坐标，右下角顶点的y坐标；paste表示粘贴图片。



罗列一下程序执行的结果，底图乱序与正序如下：

![image-20220420200318898](https://img.heshipeng.com/202204202003975.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

#### 代码获取

扫码关注微信号——逆向一步步，公众号内回复关键词`07`就可以获取本案例的全部代码。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202204202011837.jpg)

