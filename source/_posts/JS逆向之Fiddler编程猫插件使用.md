---
title: JS逆向之Fiddler编程猫插件使用
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-10 19:17:22
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]

---

#### 安装FD，配置编程猫插件

首先需要安装Fiddler，要求版本>=4.6.3。建议官网下载，下载地址：https://www.telerik.com/download/fiddler/fiddler4



下载安装完FD之后，找到FD的安装目录，打开一个叫做Scripts的文件夹：

![image-20220410204247917](https://img.heshipeng.com/202204102042092.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



解压FD编程猫插件(插件下载地址见文末)，将里面的所有.dll扩展文件拷贝到上边的Scripts文件夹下：

![image-20220410204433530](https://img.heshipeng.com/202204102044610.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



然后关闭Fiddler，并重新启动Fiddler，注意第一次启动需要以管理员身份运行。打开之后界面如下表示安装成功：

![image-20220410204605262](https://img.heshipeng.com/202204102046340.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### Fiddler的使用

##### Chrome配置Fiddler抓包

安装SwitchyOmega代理管理Chrome浏览器插件，在Chrome扩展应用商城中搜索SwitchyOmega，然后安装。添加好插件后，打开SwitchyOmega点击新建情景模式：

![image-20220410222059063](https://img.heshipeng.com/202204102220279.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

我们就把新建的情景命名为Fiddler，然后第一行，代理协议填HTTP，代理服务器地址填：127.0.0.1(因为我是用的虚拟机，所以这里填虚拟机的ip地址)，端口填8888。



在最左边选择应用选项，就保存了刚才的配置。

![image-20220410222742171](https://img.heshipeng.com/202204102227326.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



打开一个页面，比如百度，然后点击SwitchyOmega，选择Fiddler插件，然后刷新页面。

![image-20220410224632321](https://img.heshipeng.com/202204102246448.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



这个时候可以看到Fiddler中已经成功抓到了百度的页面请求，但是由于百度使用的是HTTPS协议，我们还没有配置证书，导致Fiddler抓包的数据不正常，并且百度首页也如同上边展示的那样。FD配置证书抓取HTTPS，也正是接下来要讲的。

![image-20220410224830653](https://img.heshipeng.com/202204102248727.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### Fiddler配置抓取HTTPS

打开Fiddler，点击工具栏的Tools->Options，点击HTTPS选项，按照如下图勾选相应的选项：

![image-20220410225612499](https://img.heshipeng.com/202204102256630.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

点击Trust Root Certificate，点击Export Root Certificate将FD的证书保存到桌面。这时候桌面上会出现证书FiddlerRoot.cer文件，点击OK设置成功，关闭fiddler。



打开Chrome浏览器，在浏览器地址中输入：chrome://settings/进入chrome的设置页面，在搜索栏中输入管理证书，点击管理证书会跳转到钥匙串管理程序(Mac系统)。

将桌面的FD证书拖拽到这个钥匙串管理程序中，拖拽成功后，证书默认是不信任的，如下图：

![image-20220410230101918](https://img.heshipeng.com/202204102301070.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



双击这个DO_NOT_TRUST_FiddlerRoot证书，在弹出的窗口中点击信任，选择始终信任。如下图：

![image-20220410230244807](https://img.heshipeng.com/202204102302871.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

点击左上角关闭按钮，弹出对话框输入root密码。

打开Fiddler，刷新百度首页，可以看到百度首页正常加载，Fiddler中也能正常的抓到百度的请求。



##### 编程猫插件的使用

点击右边的编程猫专用插件，会出现一列导航，如下：

![image-20220410234536919](https://img.heshipeng.com/202204102345154.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



生成易代码可以忽略，是易语言专用。JS调试工具，内置了一个V8引擎，可以在这个tab里面编辑JS代码，然后进行调试。JSON解析是只格式化JSON数据。数据加密解密，则提供一些常见的加密算法，比如MD5，Sha，AES，DES等。编码解码则提供了一些常见的编码工具，比如Base64。这个编程猫工具重点关注的是注入Hook与内存漫游。



首先说注入Hook，默认提供了一个Hook Cookie的代码，如下图。勾选左边的开启，然后地址栏为空表示对所有的地址都注入Hook代码，如果只对特定的地址注入代码，则填上地址即可。

![image-20220410235308981](https://img.heshipeng.com/202204102353138.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们以百度首页为例，开启注入Hook之后，刷新百度首页，在console控制台上打印了日志信息，如下图：

![image-20220410235817134](https://img.heshipeng.com/202204102358312.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



除了注入Hook Cookie的代码，还可以注入Hook window属性，websocket，内置函数或者自定义函数。参考[JS逆向之快速定位关键代码](https://blog.heshipeng.com/JS%E9%80%86%E5%90%91%E4%B9%8B%E5%BF%AB%E9%80%9F%E5%AE%9A%E4%BD%8D%E5%85%B3%E9%94%AE%E4%BB%A3%E7%A0%81/)



接着说内存漫游。所谓浏览器内存漫游，其原理通过ast把浏览器中所有的变量，参数中间值在内存中的变化进行存储，然后我们就可以搜索这些值，从而根据值确定这个参数在浏览器中出现的位置，变化情况等等。



我们以极验的w参数为例。FD上点击内存漫游tab，然后点击下边的开启内存漫游按钮：

![image-20220411002857220](https://img.heshipeng.com/202204110028459.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们打开极验的测试网址：https://www.geetest.com/demo/slide-float.html，然后拖动滑块完成验证。找到这个ajax.php的请求，复制下来w参数的值，如下图：

![image-20220411003041687](https://img.heshipeng.com/202204110030813.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



接着在console面板输入`hook.search(刚才复制下来的w的值)`，执行之后就可以看到w参数在浏览器中整个的运行情况：

![image-20220411003509171](https://img.heshipeng.com/202204110035276.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



可以看到w参数一共出现了2次，第一次是初始化，第二次是调用。我们随便点击一个的文件位置进去(比如点击第一个)，然后打上断点：

![image-20220411003649162](https://img.heshipeng.com/202204110036242.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



刷新页面，再次进行验证码的验证，可以看到成功断上了，并且debugger面板上也可以看到我们跟的值是w这个参数：

![image-20220411003930689](https://img.heshipeng.com/202204110039808.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



#### 编程猫插件下载地址

扫码关注微信公众号——逆向一步步，然后公众号内回复“FD编程猫插件”，就可以获得插件下载地址。

![qrcode_for_gh_509fdefd3c81_258](https://img.heshipeng.com/202202231720872.jpg)
