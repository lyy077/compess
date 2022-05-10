---
title: JS逆向之常见代码混淆的处理
top: false
cover: false
toc: true
mathjax: true
date: 2022-02-18 16:37:23
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---



#### 加密分析流程总结

在开始介绍常见代码混淆之前，看下加密分析的流程。

1. 查看关键包——分析哪些参数是加密的。
2. 搜索参数。

* 参数名= / 参数名 = / 参数名: / 参数名 :

  * 查看网络面板的Initiator(发起)
  * xhr断点调试
  * hook相关逻辑

3. 分析加密。
4. 补全加密逻辑。



##### 颜文字网络表情编码解混淆

将Javascript代码转换为颜文字网络表情的编码以达到混淆的目的。

原理：这类混淆通常都是使用构造函数将字符串作为代码运行。

例如：

```js
const sum = new Function('a', 'b', 'return a + b')
console.log(sum(2, 6))
```



解决办法：

1. 直接将混淆后的代码粘贴至控制台通过VM查看源代码。
2. 删除代码结尾的"('_')"，替换为toString()或将修改后的代码粘贴至控制台执行。



##### 符号组成的代码反混淆

将Javascript代码转换成仅由符号组成的代码以达到混淆的目的。

原理：这类混淆同常使用构造函数将字符串作为代码运行。

转换流程：

```js
Function("alert(1)")();
(0)["constructor"]["constructor"]("alert(1)")();
$ = "constructor";
$$ = "alert(1)";
$_ = ~[];
($_)[$][$]($$)();
```

解决办法：

1. 直接将混淆后的代码贴到控制台通过VM查看源代码。
2. 删除代码结尾的()，替换为toString()，将修改后的代码粘贴至控制台运行。



##### Jsfuck反混淆

将JS代码转换成只有6种符号，()[]!+，的编码，以达到混淆的目的。

解决办法：

1. 直接将混淆后的代码粘贴至控制台通过VM查看源码。

2. 删除代码结尾的()替换为toString，修改后粘贴至控制台执行。
3. 如果代码最后不是()，而是)，找到最后一对()包裹的代码，将其抽离出来单独执行。



##### Packed混淆

将JS代码打包成以eval开头，特征字符串是function(p, a, c, k, e, r)或者是function(p, a, c, k, e, d)的代码。

```js
eval(function (p, a, c, k, e, d) {
    e = function (c) {
        return (c < a ? "" : e(parseInt(c / a))) + ((c = c % a) > 35 ? String.fromCharCode(c + 29) : c.toString(36))
    };
    if (!''.replace(/^/, String)) {
        while (c--) d[e(c)] = k[c] || e(c);
        k = [function (e) {
            return d[e]
        }];
        e = function () {
            return '\\w+'
        };
        c = 2;
    }
    ;
    while (c--) if (k[c]) p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
    return p;
}('0', 3, 3, 'hello!||'.split('|'), 0, {}))

```

反混淆方法：

将eval改成console.log，在控制台上输出。



##### AST混淆反混淆

通过字符串编码，字符串常量编码，数值编码，数组混淆，数组乱序，花指令，流程平坦化，逗号表达式混淆等方式，对JS代码进行混淆。

处理方式：（扣JS，补JS）

1. 找到入口方法
2. 整理主要逻辑
3. 修改逻辑代码补全缺失的逻辑
4. 完成代码的整理



#### 总结

以上几种混淆中常见的是AST混淆，后边会有专门的文章介绍AST混淆的原理，如何手动反混淆以及借助相关的工具实现反混淆。
