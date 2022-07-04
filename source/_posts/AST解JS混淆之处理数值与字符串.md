---
title: AST解JS混淆之处理数值与字符串
cover: false
toc: true
mathjax: true
date: 2022-05-27 20:29:25
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



比如有下面一段代码：

```js
m7z = {
    '\x67\x74': V7z[M9r.C8z(190)][M9r.C8z(189)],
    '\x63\x68\x61\x6c\x6c\x65\x6e\x67\x65': V7z[M9r.R8z(190)][M9r.R8z(425)],
    '\x77': r7z + H7z,
};

var a = 0x25,b = 0b10001001,c = 0o123456, e = "\u0068\u0065\u006c\u006c\u006f\u002c\u0041\u0053\u0054";
```

这种把字符串处理成unicode或者utf8编码，把数字处理成非10进制，不利于我们进行调试，我们可以对其进行反混淆，变成我们更加习惯的编码或者进制。



观察AST结构：

![image-20220630155927641](https://img.heshipeng.com/202206301559685.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

![image-20220630160915492](https://img.heshipeng.com/202206301609536.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到在extra节点中的raw是utf-8编码的，而value的值是正常的。官网手册查询得知，**NumericLiteral、StringLiteral类型的extra节点并非必需，这样在将其删除时，不会影响原节点**。所以一种通用的解决方案是直接删除extra节点即可。



所以解混淆插件代码如下：

```js
const visitor =
{
    //TODO  write your code here！
    NumericLiteral({node}) {
        if (node.extra && /^0[obx]/i.test(node.extra.raw)) {
            node.extra = undefined;
        }
    },
    StringLiteral({node})
    {
        if (node.extra && /\\[ux]/gi.test(node.extra.raw)) {
            node.extra = undefined;
        }
    },
}
```

遍历遇到NumericLiteral节点时，判断extra是否为二进制，八进制，十六进制，如果是的话直接置空；同理，遍历遇到StringLiteral节点时，判断其extra是否为unicode编码或者utf8编码，如果是的话也置空。



最后解混淆的结果如下：

![image-20220630161728547](https://img.heshipeng.com/202206301617590.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)
