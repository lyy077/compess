---
title: JS逆向之EditThisCookie工具篇
top: false
cover: false
toc: true
mathjax: true
date: 2022-02-16 10:26:34
password:
summary:
tags: [逆向, JS, EditThisCookie]
categories: [JS逆向]
---



在做爬虫开发的时候经常会遇到查看或者修改Cookies的时候，比如说我们可能会有这么一些需求，比如编辑Cookies，编辑它的内容或者有效期；或者是说删除Cookies，实现某个页面的退出，或者说测试某个Cookie是否有效等；或者说添加某个Cookie，比如说在未登录状态下添加某个Cookie然后绕过登录；或者说导入导出某些Cookie，有时候我们需要把Cookie持久化存储在另一台电脑上，那么我们可能需要一些导入导出机制。这个时候我们可以借助一款浏览器插件叫做EditThisCookie来帮助我们轻松的完成上面提到的需求，本文将介绍这个插件的使用方法。



#### 安装

打开[Chrome 网上商店](https://chrome.google.com/webstore/category/extensions?hl=zh-CN)，搜索EditThisCookie，然后点击添加至Chrome即可。

添加完之后，点击浏览器右上角的扩展程序小图标，然后点击EditThisCookie的固定按钮，将EditThisCookie固定到菜单栏，方便以后使用。

![image-20220216104323796](http://img.heshipeng.com/202202161043581.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 功能介绍

##### 管理Cookies

我们以editthiscookie.com为例，浏览器地址输入该网址，然后打开Chrome的开发者工具，可以看到多了一个EditThisCookie的窗格：

![image-20220216113706345](http://img.heshipeng.com/202202161137466.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到当前网站的Cookie以一个表格的形式呈现，我们可以在这个表格里对某些Cookie进行修改，删除等操作。但是没法添加。



我们也可以用开发者工具自带的Cookie管理窗口：

![image-20220216114048001](http://img.heshipeng.com/202202161140028.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以对Cookie进行CURD操作，也比较方便。这里显示的Cookie不仅仅是editthiscookie.com这个domain下的Cookie，不过我们可以用上面的Filter进行过滤，如下：

![image-20220216114257547](http://img.heshipeng.com/202202161142574.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



除了上面2种方式，我们还可以点击EditThisCookie这个图标：

![image-20220216114525960](http://img.heshipeng.com/202202161145990.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到这里除了能对Cookie进行CURD操作外，还可以对Cookie进行批量操作，比如删除所有Cookie。还可以进行Cookie的导入导出等。



##### 导出Cookie

点击浏览器右上角EditThisCookie小图标，然后点击设置

![image-20220216134602208](http://img.heshipeng.com/202202161346254.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

点击选项，选择Cookie的导出格式，这里我们选择JSON：

![image-20220216134645420](http://img.heshipeng.com/202202161346304.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后回到前一个页面，点击导出Cookie按钮：

![image-20220216135001882](http://img.heshipeng.com/202202161350923.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

此时Cookie并以JSON的形式复制到了剪切板，我们找一个文本文件粘贴，可以得到Cookie的内容：

![image-20220216135141980](http://img.heshipeng.com/202202161351394.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这种JSON格式的Cookie就可以直接放到程序中使用了。



#### Cookie导入导出小实验

首先我们登录github.com，登录之后主页可以看到个人信息：

![image-20220216135949035](http://img.heshipeng.com/202202161359075.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

接着我们用EditThisCookie导出Cookie并保存到一个文本文件：

![image-20220216140120593](http://img.heshipeng.com/202202161401623.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后打开开发者工具，选择Application，选择Cookie，然后右键Clear，清除该站点所有的Cookie：

![image-20220216140235247](http://img.heshipeng.com/202202161402640.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这时我们刷新该页面，发现我们成功退出了github：

![image-20220216140324742](http://img.heshipeng.com/202202161403657.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后我们点击EditThisCookie，导入刚才保存的Cookie：

![image-20220216140419364](http://img.heshipeng.com/202202161404572.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

然后刷新页面，发现我们又重新登录了github：

![image-20220216140504490](http://img.heshipeng.com/202202161405794.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)
