---
title: AST解JS混淆之删除所有注释
cover: false
toc: true
mathjax: true
date: 2022-05-27 23:20:29
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



当代码中有成段成段的注释，但是我们又不需要的时候，可以采用如下代码去删除JS源代码中的注释：

```js
const output = generator(ast, opts={"comments": false}, code);
```



测试如下：

例如源代码为：

```js
/*
这是多行测试，第一行
这是多行测试，第二行
这是多行测试，第三行
*/

var a = "你好AST"; // 这也是单行测试
// 这是单行测试

let b = a + 1; /*这也是单行测试*/
console.log(/*这是代码之间的测试*/b);

/*
这也是多行测试，第一行
这也是多行测试，第二行
这也是多行测试，第三行
*/
```



下面是输出：

```js
var a = "你好AST";
let b = a + 1;
console.log(b);
```



