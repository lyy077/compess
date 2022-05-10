---
title: JS逆向之Toggle Javascript工具篇
top: false
cover: false
toc: true
mathjax: true
date: 2022-02-16 15:05:12
password:
summary:
tags: [逆向, JS, Toggle Javascript]
categories: [JS逆向]
---



Javascript，它是网站的一部分，我们的网站大部分情况下都会有Javascript的执行。有了它，我们整个网站的功能会变得更加的强大。但是对于我们爬虫来说，比如说我们用requests来请求某一个网站的时候，我们得到的结果实际上可能与浏览器得到的真实看到的结果或者说在浏览器开发者工具Eelements窗口看到的结果是不一样的。这个就是由于Javascript页面的渲染导致的结果不同，这样就会对爬虫产生一个较大的干扰。如果我们能禁用后期的Javascript渲染，就可以在浏览器中看到真实的结果与我们requests请求得到的结果一致了，这样就非常方便了，那么如何做到这一个功能呢？这篇文章就会介绍Toggle Javascript的Chrome插件。



#### 安装

打开[Chrome 网上商店](https://chrome.google.com/webstore/category/extensions?hl=zh-CN)，搜索Toggle Javascript，然后点击添加至Chrome即可。

添加完之后，点击浏览器右上角的扩展程序小图标，然后点击Toggle Javascript的固定按钮，将Toggle Javascript固定到菜单栏，方便以后使用。

![image-20220216151324812](http://img.heshipeng.com/202202161513854.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 使用

通过点击Toggle Javascript按钮来暂停或恢复JS的执行。以一个小说网站[红薯网](https://g.hongshu.com/nan.html)为例：

我们随便点开一个小说的某一章：

![image-20220216152044654](http://img.heshipeng.com/202202161520089.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

这个页面实际上有部分内容是用Javascript渲染生成的，如果我们用requests请求的话，得到的页面的效果，就是我们禁用掉Javascript之后的效果，我们点击Toggle Javascript小图标：

![image-20220216152307901](http://img.heshipeng.com/202202161523551.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

我们可以看到标点符号没了，整个语句也变得不通顺了。我们如果用正常的requests请求的话就会得到这样一个效果。

打开开发者工具，切换到Network面板，刷新页面，我们查看原生的请求，Preview的结果与我们通过Toggle Javascript禁用JS的结果是一摸一样的。

![image-20220216152557041](http://img.heshipeng.com/202202161526590.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

我们再看一下源码，有一些文字有一些span的占位符，后期通过JS渲染替换这些占位符，就可以渲染出真正的页面内容：

![image-20220216152850619](http://img.heshipeng.com/202202161528961.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 总结

Toggle Javascript是一款Chrome浏览器插件，其主要功能是禁用或者执行页面的JS，在爬虫开发的过程中，通过该工具可以迅速的得到requests请求之后的页面结果，非常方便。
