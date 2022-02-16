---
title: JS逆向之Charles工具篇
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-02-15 13:51:15
password:
summary:
tags: [逆向, JS, Charles]
categories: [JS逆向]
---



#### Charlse

[Charlse](http://charlesproxy.com)是一个HTTP/HTTPS抓包工具，支持Windows/Linux/Mac平台。功能包含截获请求，过滤请求，重发请求，设置断点，模拟网速，反向代理等。



##### 安装证书

抓取PC端HTTPS的请求时，如果没有配置Charles证书，会出现如下报错：

![image-20220215152744261](http://img.heshipeng.com/202202151527989.png)

此时需要安装Charles证书，步骤如下(以Mac环境为例，其他类似)：

![image-20220215153015985](http://img.heshipeng.com/202202151530578.png)

打开Charlse，选择Help->SSL Proxying->Install Charles Root Certicate

然后弹出一个证书选项：

![image-20220215153449694](http://img.heshipeng.com/202202151534426.png)

更改钥匙串的保存位置为登录，然后点击添加。之后打开钥匙串访问应用：

![image-20220215153741168](http://img.heshipeng.com/202202151537203.png)

找到登录选项，然后证书这一栏，如上。可以看到刚才添加的Charles证书前面有一个叉叉，双击这个证书。点击信任：

![image-20220215153936079](http://img.heshipeng.com/202202151539110.png)

接着把使用此证书时改为始终信任，如下图。然后关闭并且保存修改。

![image-20220215154022123](http://img.heshipeng.com/202202151540154.png)

添加完证书后，需要修改SSL Proxying设置，将我们具体要抓的请求地址与端口添加进来。具体步骤：

点击Proxy->SSL Proxying Settings

![image-20220215154322329](http://img.heshipeng.com/202202151543359.png)

然后在Include这一栏点击Add按钮，Host和Port都填通配符*，表示抓包时不限制ip以及端口。

![image-20220215154509908](http://img.heshipeng.com/202202151545317.png)

以上两个重要设置都修改好后，重新刷新我们刚才抓包时报握手错误的页面，发现可以正常抓包了。



##### 截获请求

安装好证书，配置好SSL Proxing之后，就可以正常的截获请求了。这里介绍一个网站——httpbin.org。httpbin.org是一个可以进行模拟请求的网站。

![image-20220215160208105](http://img.heshipeng.com/202202151602139.png)

* HTTP Methods 可以模拟发送请求方法为DELETE/GET/PATCH/POST/PUT的请求。
* Auth 可以模拟发送包含各种需要验证的请求，比如Bearer token。
* Status codes可以模拟发送状态码为401, 403, 500等请求。

其它功能还有包括请求头检测，返回头检测，动态数据，图片，Cookie，重定向等，不一一列举了。



这里我们用httpbin.org发送一个GET请求：点击HTTP Methods，点击GET，点击Try it out，点击Execute，就成功发送了一个HTTP请求。

![image-20220215160836694](http://img.heshipeng.com/202202151608677.png)

我们可以在Charles上成功截获到这个请求：

![image-20220215161001335](http://img.heshipeng.com/202202151610254.png)



##### 过滤请求

当charles抓到的请求非常多时，我们需要迅速定位到我们想要截获的请求，这个时候就需要用到过滤请求了。有2种方法：

1. 在filter栏填写需要过滤出来的域名

   ![image-20220215161256609](http://img.heshipeng.com/202202151612133.png)

2. 固定某一个域名

![image-20220215161421619](http://img.heshipeng.com/202202151614409.png)

找到我们需要的那一个域名，然后右键选择Focus，这个时候当前域名就会置顶，并且其它所有域名都变为Other Hosts。



##### 重发请求

重发请求可以通过点击刷新按钮，也可以右键Repeat按钮，重发请求之前我们还可以编辑当前的请求，比如修改User-Agent，Cookie等，此外还有一个Repeat Advanced功能，可以设置重复某一个请求多少次以及每次请求之间延迟多长时间。下面一一介绍这些功能：

1. 重发请求

第一种方式，选中某一个请求，然后点击刷新按钮：

![image-20220215164451184](http://img.heshipeng.com/202202151644218.png)

第二种方式，选中某一个请求，然后右键，点击Repeat：

![image-20220215164611051](http://img.heshipeng.com/202202151646081.png)



2. 编辑请求

编辑请求后重发，可以方便我们不写一行代码去调研某一个请求的关键点，非常方便我们进行调试，选中某一个请求，点击修改按钮：

![image-20220215164921556](http://img.heshipeng.com/202202151649499.png)

可以修改请求的URL，请求的参数，请求的Header等等：

![image-20220215165128661](http://img.heshipeng.com/202202151651143.png)

3. Repeat Advanced

选中某一个请求，右键点击Repeat Advanced：

![image-20220215165303797](http://img.heshipeng.com/202202151653707.png)

iterations填入重复的次数，Concurrency填入请求的并发数，Repeat delay表示每个请求之间的延迟。勾选Use ranges还可以设置延迟在一个范围内，即保证延迟的随机性。

![image-20220215165446565](http://img.heshipeng.com/202202151654252.png)

这个功能可以帮助我们在开发爬虫的时候不用编写代码就可以调试爬虫的是否有访问频率上的限制，应做出来的延时策略是怎样的等等。



##### 设置断点

先看下断点的使用方法，然后再介绍这样做的意义。

选中某一个请求，然后右键点击Breakpoints：

![image-20220215170854304](http://img.heshipeng.com/202202151708346.png)

也可以直接通过点击Proxy->Breakpoint Settings，然后添加一个断点。

![image-20220215170941208](http://img.heshipeng.com/202202151709351.png)

添加好断点之后，我们可以再次发起请求，可以看到我们的请求一直处于等待状态。

![image-20220215171458730](http://img.heshipeng.com/202202151715186.png)

我们可以编辑该请求从而达到断点调试的功能。

断点调试的意义在哪？有时我们想让服务器返回一些指定的内容，方便我们调试一些特殊情况。例如列表页面为空的情况，数据异常的情况，部分耗时的网络请求超时的情况等，这个时候使用断点调试就可以模拟出这些情况。使用断点调试将网络请求截获并修改过程中，整个网络请求的计时并不会暂停，所以长时间的暂停可能导致客户端的请求超时。



##### 模拟网速

点击Proxy->Throttle Settings，然后勾选Enable Throttling开启模拟网速：

![image-20220215172549357](http://img.heshipeng.com/202202151725407.png)



##### 反向代理

反向代理相当于我们在发起一次请求的时候，请求会经过我们配置的代理拿到响应后，然后我们再把响应转发回我们的客户端。

在介绍两种通过Charles配置反向代理的方式之前，先要清除我们上边设置断点时留下的断点。

下面介绍第一种方式：点击Proxy->Reverse Proxies Settings

![image-20220215174742827](http://img.heshipeng.com/202202151747859.png)

勾选Enable Reverse Proxies，然后点击Add添加规则。Local Port表示本地的端口，Local address表示本地地址，Remote Host表示远程地址，Remote Port表示远程端口，上图的含义表示将httpbin.org反向代理到本地的localhost:8080地址上。所以此时在浏览器地址栏中访问localhost:8080实际上访问的是httpbin.org：

![image-20220215175118759](http://img.heshipeng.com/202202151751789.png)



下面介绍第二种方式：相当于如果我们请求一个URL的话，我们可以把这个URL它的一个Response转发到一个Remote地址或者说我们用本地的一个地址。 我们以网址https://quotes.toscrape.com/js/为例。

重新抓包，看下网页源代码：

![image-20220215183821967](http://img.heshipeng.com/202202151838695.png)

内容比较简单，我们看到有一块JSON的数据，我们修改把Response映射到本地的一个文本文件，这样我们就可以完成Response的修改了。我们先保存Response到本地的一个文件，然后修改这个文件，主要修改2个地方，第一个是把JSON里面的第一项Text值修改为Hello, world。第二个是在渲染的地方加上debugger关键词，然后保存文件。

![image-20220215184356182](http://img.heshipeng.com/202202151843755.png)

![image-20220215184835283](http://img.heshipeng.com/202202151848021.png)

然后我们在Charles中选中刚才的那个请求，右键选择Map Local，然后更改Local Path选择我们刚才保存的那个文件：

![image-20220215184655514](http://img.heshipeng.com/202202151846549.png)

保存，然后刷新页面，可以看到正常显示了我们更改的内容，并且进入了debug模式：

![image-20220215185015800](http://img.heshipeng.com/202202151850830.png)

第二种方式类似于在[JS逆向之Chrome浏览器工具你知多少？](http://blog.heshipeng.com/Chrome%E6%B5%8F%E8%A7%88%E5%99%A8%E5%B7%A5%E5%85%B7%E4%BD%A0%E7%9F%A5%E5%A4%9A%E5%B0%91/)中介绍的文件导航窗格Overrides面板的功能。
