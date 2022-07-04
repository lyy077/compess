---
title: AST解JS混淆之删除空行与空语句
cover: false
toc: true
mathjax: true
date: 2022-05-27 21:59:20
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



本文介绍如何删除空语句。有时候将源代码利用AST调整后，会有很多类似这样的代码:

```js
var a = 123;
;
var b = 456;
```

其中，中间的 **;** 这一行是没必要存在了，那如何编写插件删除没啥用的这行呢？



同样的方法，先将这段代码放入 [https://astexplorer.net/](https://astexplorer.net/) 解析网站看看：

![image-20220630191917900](https://img.heshipeng.com/image-20220630191917900.png)

解析如上图。这里介绍一个小技巧：当代码行数过多时？将鼠标移动到我们想要迅速观察的那一行代码上 ，解析网站自动帮我们定位到这一行代码的AST结构。



可以看到它是一个 `EmptyStatement`，想要删除这个节点，方法很简单，直接遍历这个节点，再调用 remove 方法即可，代码如下:

```js
const visitor = {
    EmptyStatement(path) {
        path.remove();
    },
}
```

就是这么的简单。



看下运行结果：

![image-20220630192204612](https://img.heshipeng.com/image-20220630192204612.png)
