---
title: AST解JS混淆之还原中文的Unicode编码
cover: false
toc: true
mathjax: true
date: 2022-05-27 22:56:32
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



前面一篇文章 [AST解混淆之处理数值与字符串](https://blog.heshipeng.com/AST%E8%A7%A3%E6%B7%B7%E6%B7%86%E4%B9%8B%E5%A4%84%E7%90%86%E6%95%B0%E5%80%BC%E5%8D%81%E5%85%AD%E8%BF%9B%E5%88%B6,%20%E4%B8%AD%E8%8B%B1%E6%96%87Unicode%E5%AD%97%E7%AC%A6%E4%B8%B2/) 介绍过unicode或者utf8编码的字符串还原的方法，但是如果这个字符串是中文，这个方法并不会奏效，比如有下面这段代码：

```js
var a = "\u4f60\u597d\u0041\u0053\u0054";
```



经过前面的插件处理之后，结果为：

```js
var a = "\u4F60\u597DAST";
```

可以看到unicode编码的中文并没有还原。



要想还原unicode编码的中文，必须用到generate函数有个 `options选项`，下面是这个选项能完成的功能：

![图片](https://img.heshipeng.com/111.png)



所以修改我们的插件代码如下：

```js
//babel库及文件模块导入
const fs = require('fs');

//babel库相关，解析，转换，构建，生产
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
const generator = require("@babel/generator").default;

//读取文件
if (process.argv.length < 4) {
    console.log("Usage: node ${file}.js ${encode}.js ${decode}.js");
    process.exit(1);
}

let input_file = process.argv[2], output_file = process.argv[3];

let jscode = fs.readFileSync(input_file, {encoding: "utf-8"});
//转换为ast树
let ast    = parser.parse(jscode);

const visitor = {
    StringLiteral({node})
    {
        if (node.extra && /\\[ux]/gi.test(node.extra.raw)) {
            node.extra = undefined;
        }
    },
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast, opts = {jsescOption:{"minimal":true}});
fs.writeFile(output_file, code, (err)=>{});
```



运行结果：

![https://img.heshipeng.com/WX20220630-201530.png](https://img.heshipeng.com/WX20220630-201530.png)

