---
title: AST解混淆之反混淆代码模版
cover: false
toc: true
mathjax: true
date: 2022-05-26 22:32:54
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



话不多说，直接上模版：

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

const visitor = 
{
  //TODO  write your code here！
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast);
fs.writeFile(output_file, code, (err)=>{});
```



上面代码保存成一个文件，比如decode_obfuscator.js ，然后执行以下命令即可完成反混淆：

```bash
node decode_obfuscator.js input.js output.js
```

input.js表示待解混淆的文件，output.js表示解混淆之后保存的结果文件。



接着看看decode_obfuscator.js文件的内容：

主要分三步，第一步是引入parser模块，并调用相关的方法把JS源码转化为AST；第二步，对AST进行遍历，处理相应的节点，主要用到的是babel库的traverse模块；第三步是调用generator模块的相关方法，把AST还原成JS代码。其中第二步可以调用多次traverse，编写多个visitor，因为下一次遍历AST可能需要上一次遍历并处理之后的结果。
