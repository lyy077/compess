---
title: JS逆向之vscode无环境联调
top: false
cover: false
toc: true
mathjax: true
date: 2022-04-09 22:12:32
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---



##### 为啥使用VSCode

VSCode强大，支持多种语言。在JS逆向项目中，会涉及到Python和JS代码，如果Python使用Pycharm调试，JS使用WebStorm或者Hbuilder进行调试，那么需要在多个开发者工具中切换，比较麻烦，VSCode的多语言支持完美的解决这个麻烦事。此外，最重要的是VSCode可以配置JS代码的无环境联调，即可以与浏览器的dev-tools工具交互实现在VSCode中开发，浏览器工具中调试，这也是其它工具没有的。



##### 安装VSCode

VSCode下载地址：https://code.visualstudio.com/

安装好VSCode之后，可以配置语言为中文。具体方法是，点击下图的图标，选择Extension，然后在搜索中输入"Chinese"，选择简体中文，然后重启浏览器即可。

![image-20220409223010250](https://img.heshipeng.com/202204092230545.png)



##### 配置无环境联调

首先安装好NodeJS环境。

然后打开工作目录，选择任意一个JS文件，选择运行->启动调试就可以运行或者debug JS代码了。

![image-20220409235241659](https://img.heshipeng.com/202204092352976.png)



然后通过命令`npm install -g node-inspect`全局安装依赖包，如果是Linux/Mac系统，需要sudo权限。

输入命令`node-inspect`，查看node-inspect是否安装成功：

![image-20220410000634331](https://img.heshipeng.com/202204100006494.png)



我们选择运行->打开配置：

![image-20220409235420113](https://img.heshipeng.com/202204092354173.png)

出现一个新的文件launch.json文件，这个文件用来配置项目的运行环境，比如执行脚本，解释器，命令行参数等等。修改这个配置文件，内容如下：

```json
{
    // 使用 IntelliSense 了解相关属性。 
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
            "type": "node",
            "request": "launch",
            "name": "启动程序",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "program": "${workspaceFolder}/aerfaying.js"
        },
        {
            "type": "node",
            "request": "launch",
            "name": "无环境浏览器",
            "skipFiles": [
                "<node_internals>/**"
            ],
            "runtimeExecutable": "node-inspect",
            "program": "${workspaceFolder}/aerfaying.js"
        },
    ]
}
```

把之前的运行环境改了个名，叫做启动程序，然后新建了一个环境，叫做无环境浏览器，这个环境除了名字之外唯一不同的是增加了一个runtimeExecutable，把我们上边安装的node-inspect包给引入进来。



保存好配置文件之后，这个时候我们左上角就出现2个环境供我们选择了：

![image-20220410001442317](https://img.heshipeng.com/202204100014508.png)



我们点击无环境浏览器，就会启动无环境浏览器模式下的调试模式。打开任意一个Chrome浏览器窗口，打开开发者工具，在开发者工具的Elements面板左边出现一个绿色的图标，如下：

![image-20220410001701254](https://img.heshipeng.com/202204100017478.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

表示开启了VSCode与浏览器的联调模式，如果这个图标为灰色，表示关闭了联调模式。



点击这个图标，就弹出了DevTools：

![image-20220410001856382](https://img.heshipeng.com/202204100018481.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



可以看到已经帮我们把刚才调试的那个JS文件的内容自动导入，同时开启了debug模式。为什么叫做无环境浏览器？就是这个DevTools跟我们在浏览器中调试几乎是一模一样的，只不过一些浏览器的环境变量没有，比如window，document等等。这样既可以继续使用浏览器进行调试，又可以没有window/document等环境，模拟真实独立的开发环境，方便我们补环境。

![image-20220410002734125](https://img.heshipeng.com/202204100027373.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 用VS Code进行调试

首先看下左下角的这2个选项，如下图：

![image-20220418165953520](https://img.heshipeng.com/202204181659615.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



Caught Exception指的是对于加了try catch的代码块，如果执行了catch逻辑(即代码抛了异常)，依然在抛异常的地方断住。如下图：

![image-20220418171006294](https://img.heshipeng.com/202204181710387.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



Uncaught Exceptions指的是忽略try catch中的异常，在try 之外的代码块报错，才会断住。如图：

![image-20220418171418283](https://img.heshipeng.com/202204181714424.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



一般会勾选Uncaught Exceptions，因为try中的代码抛异常属于正常逻辑。如果这俩选项都没勾选，也没有自定义断点，程序就会直接运行完，并不会断住。如下图：

![image-20220418171636116](https://img.heshipeng.com/202204181716202.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



再说说调试用的的一组按钮，如下图：

![image-20220418171746410](https://img.heshipeng.com/202204181717469.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

第一个按钮，表示继续，从当前停顿的地方一直执行到下一个断点，如果没有下一个断点，则按照程序的正常执行顺序，顺序执行一步。

第二个按钮，表示单步执行，从当前位置开始，顺序执行一步，如果遇到函数调用，不进入函数内部顺序执行。

第三个按钮，表示单步执行，从当前位置开始，顺序执行一步，如果遇到函数调用，进入到函数内部单步执行。

第四个按钮，表示单步执行，如果当前调试是在函数内部单步执行，则跳过函数剩余的执行代码，回到函数调用处，往下执行一步。

第五个按钮，重启程序。

第六个按钮，停止。
