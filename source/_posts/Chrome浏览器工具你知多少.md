---
title: JS逆向之Chrome浏览器工具你知多少？
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-02-11 14:46:51
password:
summary:
tags: [逆向, JS]
categories: [JS逆向]
---



<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/AIhtECdAGFA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



#### Network面板

##### Network面板介绍

网络面板分为5个部分：控制器面板，过滤器面板，概览部分，请求列表以及概要部分。

**Controls（控制器）**：使用这些选项可以控制Network面板的外观和功能。

**Filter（过滤器）**：使用这些选项可以控制在Requests Table中显示哪些资源。按住Command/Ctrl键可以点选多个过滤器。

**Overview（概览）**：此图表显示了资源检索时间的时间线。如果看到多条竖线堆叠在一起，说明这些资源同时被检索。

**Requests Table（请求列表）**：此表格列出了检索的每一个资源。默认情况下，此表格按照时间顺序排序，最早的资源在顶部。点击资源的名称可以显示更多信息，右键点击Timeline以外的任何一个表格标题可以添加或者移除信息列。

**Summary（概要）**：此窗格可以一目了然地知道请求总数，传输的数据量和加载时间。



##### 控制器面板

![image-20220211145738994](http://img.heshipeng.com/202202111457134.png)

* 第一个按钮控制浏览器是否抓包，如果为红色表示正在抓包，点击之后变灰色表示停止抓包。
* 第二个按钮表示清除按钮，点击之后会清除请求列表。
* 第三个按钮表示过滤器的打开与关闭按钮，点击这个按钮控制下方过滤器打开还是关闭。
* 第四个按钮表示打开一个检索，用得不多。
* 第五个按钮表示是否要跨页面保存请求列表。☑️上表示保留跨页面请求信息，什么意思呢？就是我打开一个A页面，得到了一个请求列表，我再次访问B页面，C页面...无论是什么页面，只要是同一个窗口，所有的请求列表都会保存。
* 第六个按钮表示是否禁用浏览器缓存。一般来说☑️上，然后每一个请求都会去请求新的资源。
* 第七个按钮表示慢速网络模拟。可以使用slow 3G或offline，可以看到浏览器页面在不同网络环境下的样子。



##### 过滤器

![image-20220211152552978](http://img.heshipeng.com/202202111525047.png)

通过点选不同类型比如Ajax，JS，CSS等来过滤不同的请求，同时支持点选多个类型的按钮来在请求列表展示多个类型的请求。

前面Filter过滤框还提供了定制化筛选的功能，可以通过输入一些路由的关键词去搜索，也可以通过类似于正则的方式去搜索。比如：

domain:*.nightteam.cn：表示domain为\*.nightteam.cn的请求。

status-code:301：表示状态码为301的请求。

set-cookie-domain:bbs.nightteam.cn：表示进行set-cookie的操作的domain。



##### 请求列表

![image-20220211153842703](http://img.heshipeng.com/202202111538766.png)

标题栏里面比较关键的是Initiator这一栏，表示请求是由哪里发起的。Other表示发起者通过动作发起，而不是某一个具体的JS发起。bbs.nightteam.cn是我们通过浏览器地址栏输入地址后回车发起的，所以这里显示other，而其它的请求都是可以找到具体调用的JS文件。通过定位到某一个请求的发起文件，我们可以在该文件中增加断点进行调试，所以这一栏比较重要。



点击某一个具体的请求，可以看到它的请求头信息，响应头信息，预览页面，耗时，Cookie信息等。

![image-20220211154903286](http://img.heshipeng.com/202202111549405.png)



选中某一个请求，然后右键选择Copy->Copy as cURL得到该请求的cURL命令。

![image-20220211155142085](http://img.heshipeng.com/202202111551191.png)

```bash
curl 'https://bbs.nightteam.cn/' \
  -H 'authority: bbs.nightteam.cn' \
  -H 'cache-control: max-age=0' \
  -H 'sec-ch-ua: " Not A;Brand";v="99", "Chromium";v="98", "Google Chrome";v="98"' \
  -H 'sec-ch-ua-mobile: ?0' \
  -H 'sec-ch-ua-platform: "macOS"' \
  -H 'upgrade-insecure-requests: 1' \
  -H 'user-agent: Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/98.0.4758.80 Safari/537.36' \
  -H 'accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9' \
  -H 'sec-fetch-site: none' \
  -H 'sec-fetch-mode: navigate' \
  -H 'sec-fetch-user: ?1' \
  -H 'sec-fetch-dest: document' \
  -H 'accept-language: zh-CN,zh;q=0.9,en;q=0.8' \
  -H 'cookie: __yjs_duid=1_4110af544673b8e7f7ad03205cd2f8dc1644394949170; WoQu_2132_saltkey=YrDr0Zud; WoQu_2132_lastvisit=1644391349; popnotice=ss10; __8qcehdE7ZaRq2q6M__=5df749990f6140450200d728b907dcbe500b; WoQu_2132_sid=uh3oVX; WoQu_2132_st_p=0%7C1644476217%7C06bfc6a3958129a26420eb3777962f3c; WoQu_2132_visitedfid=39D38D37; WoQu_2132_viewid=tid_2275; WoQu_2132_lastact=1644476217%09home.php%09misc' \
  --compressed
```

可以直接在类Unix系统的控制台中去执行这个cURL命令，也可以把这个命令导入到PostMan中去执行。具体步骤是：

![image-20220211155646608](http://img.heshipeng.com/202202111556685.png)



点击Shift键，然后鼠标悬停在某一个请求上，如果展示绿色背景，表示绿色背景的请求依赖你当前悬停的请求。如果展示红色背景，表示当前悬停的请求依赖红色背景的请求。

![image-20220211160819614](http://img.heshipeng.com/202202111608727.png)

![image-20220211160845852](http://img.heshipeng.com/202202111608906.png)



#### Source面板

source面板分为三个部分：文件导航窗口，代码编辑器窗格， 调试窗格。

文件导航窗口：可以对文件目录进行浏览；

编辑器窗格：代码编辑，设置断点；

调试窗格：包含调试所用的常用选项。

![image-20220211165847739](http://img.heshipeng.com/202202111658856.png)



##### 文件导航窗格

![image-20220214103607974](http://img.heshipeng.com/202202141036679.png)

* page面板

以文件目录形式展示当前页面。包含当前网站的页面，静态资源等。



* FileSystem面板

可以让开发者工具加载本地的文件系统，并且能在编辑框口修改编辑，由此化身IDE。这里做一个示范：

第一步：点击`Add folder to workspace`

![image-20220214104511894](http://img.heshipeng.com/202202141045585.png)

第二步：选择要上传的文件夹

![image-20220214104624359](http://img.heshipeng.com/202202141046182.png)

第三步：点击允许，让开发者工具具备读写该文件夹的权限

![image-20220214104720311](http://img.heshipeng.com/202202141047198.png)

然后就可以看到我们上传的文件夹以及相关的JS文件了：

![image-20220214105309274](http://img.heshipeng.com/202202141053302.png)

我们可对其进行编辑修改，打上断点调试等。



* Overrides面板

Overrides面板可以很容易的将远程资源下载一份在本地，然后可以在开发者工具下进行编辑，并且开发者工具会展示我们编辑后的文件。换句话说，就是直接将一些请求代理到本地的文件中。这里同样做一个演示：

前三步跟FileSystem类似：点击`Select folder for overrides`，然后选择本地文件夹，之后点击允许赋予开发者工具读写该文件夹的权限。接着要点击选中`Enable Local Overrides`：

![image-20220214112558071](http://img.heshipeng.com/202202141126711.png)

然后回到Network面板，点击浏览器刷新按钮刷新当前页面，选中一个我们需要代理到本地的请求，右键选择`Save for overrides`：

![image-20220214113204817](http://img.heshipeng.com/202202141132844.png)

这时，我们刚才添加的文件夹里面就有了保存的代理请求：

![image-20220214113325024](http://img.heshipeng.com/202202141133043.png)

然后我们修改本地的这个文件，比如将这个页面的`招聘求职`改为`MLGJ求职`：

![image-20220214113851477](http://img.heshipeng.com/202202141138509.png)

然后，保存刷新这个页面，就成功的将该请求代理到本地文件：

![image-20220214113954313](http://img.heshipeng.com/202202141140600.png)



* Content scripts

chrome插件加载的一些脚本，如果有过chrome浏览器插件开发的经验的话，对这个概念应该会很熟悉。在JS逆向中这个用的不多，后面会有专门的文章介绍chrome浏览器插件开发。



* Snippets

我们都知道，有时候需要调试一行JS代码，会在浏览器的Console控制台上去调试，如果需要在浏览器中调试一个代码片段呢？在Console中一行行的输入执行就显得很麻烦了。这个Snippets就可以帮我们解决这个问题，可以通过点击`New snippet`按钮，添加一些代码片段，然后选中某一个代码片段文件，右键`Run`去执行该代码片段：

![image-20220214115844902](http://img.heshipeng.com/202202141158016.png)



##### 代码编辑器窗格

可以在编辑器中打开文件，断点调试或者格式化等。下面演示一下各个功能。

1. 代码格式化

打开Page面板下网站的一个JS文件，如果JS是展示在一行，可以点击下面的一对`{}`进行格式化：

![image-20220214140646923](http://img.heshipeng.com/202202141406960.png)



2. 添加简单断点

在我们上边格式化之后的JS文件，选中某一行，单击该行的行号，即可添加断点：

![image-20220214140844097](http://img.heshipeng.com/202202141408998.png)



3. 添加条件断点

在前面介绍的Snippets代码片段中添加一个片段文件，内容如下：

```javascript
console.log("Script snippet #1 start");
var i = 1;
while (5 > i) {
    console.log(i)
    debugger;
    i++;
}
console.log("Script snippet #1 end");
```

这段代码比较简单，当i小于5的时候进入循环，停在断点处。下面开始条件断点的设置：

先单击要添加条件断点的那一行的行号，右键，选择`Add conditional breakpoint`

![image-20220214150928211](http://img.heshipeng.com/202202141509714.png)

然后在这个条件输入框中，输入一个条件，比如这里我们是`i = 10`:

![image-20220214145305065](http://img.heshipeng.com/202202141453523.png)

设置好条件断点之后，断点会变成黄色，区别于普通断点：

![image-20220214145448231](http://img.heshipeng.com/202202141454079.png)

为了便于观察i的值，可以在最右边的调试窗格变量监控一栏添加观察i的值。

![image-20220214145615800](http://img.heshipeng.com/202202141456576.png)

最后右键运行该文件，如果条件断点正确的打上的话，变量监控值会看到i的值为10：

![image-20220214151148079](http://img.heshipeng.com/202202141511114.png)



##### 调试窗格

1. XHR/fetch Breakpoints

XHR断点触发需要满足2个条件：一个是基于XHR的生命周期发生改变，一个是XHR的URL与我们设置的字符串相匹配。常用的是第二种。我们现在以百度作为一个演示。

打开百度网站首页，点击登录，然后打开Network面板，选择Fetch/XHR，我们进行一个抓包，只抓XHR请求的包，结果如下：

![image-20220214154210085](http://img.heshipeng.com/202202141542127.png)

我们选择其中任意一个，比如下面一个请求，回到调试窗格，然后点击新增一个XHR断点，填入该请求的URL或者一部分：

![image-20220214154422755](http://img.heshipeng.com/202202141544801.png)

然后我们退出登录后重新点击登录百度，可以看到设置的断点生效了：

![image-20220214154605187](http://img.heshipeng.com/202202141546839.png)



2. DOM Breakpoint

还是以百度为例，切换到元素面板，然后随便选择一个DOM元素，右键Break on：

![image-20220214155229706](http://img.heshipeng.com/202202141552746.png)

有三种类型的断点，子元素改变/属性改变/节点删除。选择其中一个，即添加成功：

![image-20220214155406742](http://img.heshipeng.com/202202141554612.png)

由于DOM Breakpoint在逆向中不是很常用，所以做一个了解即可。



3. Event Listener Breakpoint

基于事件监听的断点。同样还是以百度为例，点击登录到登录页面，可以看到页面上是一个登录按钮的，这个登录按钮就是触发的submit事件。我们在调试窗格上找到Event Listener Breakpoints，然后打开Control，选中Submit：

![image-20220214160042544](http://img.heshipeng.com/202202141600745.png)

然后我们点击登录页面的登录按钮，如无意外，断点成功打上：

![image-20220214160209183](http://img.heshipeng.com/202202141602842.png)



4. 断点控制按钮

![image-20220214160335416](http://img.heshipeng.com/202202141603925.png)

第一个按钮：执行到下一个断点，如果没有下一个断点，则按照正常执行顺序执行到下一行代码。

第二个按钮：执行到下一行代码。

第三个按钮：点击进入下一个待执行的函数调用中去。

第四个按钮：跳出当前的函数调用。

第六个按钮：去除所有断点，恢复正常执行的方法。



5. Call Stack

断点的调用栈列表。



6. Scope

断点所在作用域内容。



#### Console面板

在逆向中，console这个交互环境，经常用来输出一些变量的值。有一个方法`console.count`提一下，经常用来统计某一个方法的调用次数。用法比较简单，如下：

![image-20220214162607942](http://img.heshipeng.com/202202141626994.png)



console面板还有一些骚操作，如下：

* console.table

我们可以在console中使用console.table，以表格形式展示数据。这里做一个示例：

打开console控制台，定义一个变量，内容如下：

```js
var data = [{"name": "zhangsan", "age": 21}, {"name": "lisi", "age": 22}]
```

然后我们调用`console.table`输出这个变量：

![image-20220214163035142](http://img.heshipeng.com/202202141630189.png)



* copy

可以在console中调用copy方法，将某一个变量的值复制到剪切板，比如上边的data，我们调用copy(data)就可以将data的内容复制到剪切板了。



* $

$_ 表达的是上一次表达式计算的值。

$可以当作document.querySelect使用。

$$可以当作document.querySelectAll来使用。

$x可以当作xpath选择器来使用。

![image-20220214163953998](http://img.heshipeng.com/202202141639033.png)



#### 总结

本文介绍了浏览器开发者工具的使用技巧，主要介绍了三个面板：Console面板，Network面板和Source面板，是JS逆向必须掌握的知识。

![开发者工具的使用介绍和技巧](http://img.heshipeng.com/202202141656805.png)
