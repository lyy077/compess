---
title: AST解JS混淆之AST基础
cover: false
toc: true
mathjax: true
date: 2022-05-28 20:32:08
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



# 认识AST

打开 [https://astexplorer.net/](https://astexplorer.net/) ，选择语言Javascript，选择解析库@babel/parser。如图：

![image-20220630230656018](https://img.heshipeng.com/image-20220630230656018.png)



并在左侧输入以下代码：

```js
function somewhat() {
	let a = [1, 2, 3];
  	for (let i = 0; i < a.length; i++) {
    	console.log(a[i]);
    }
}

let b = "Hello,AST";
console.log(b);
```



折叠右边展示的Tree的所有子节点，可以看到主要结构如下：

![image-20220630231236304](https://img.heshipeng.com/image-20220630231236304.png)

其中**File是整个树的根节点**。然后基本上每一个子节点都包含type, start, end, loc。给出一个表格列出这几个字段的含义：

| 节点属性 | 记录的信息                                  |
| -------- | ------------------------------------------- |
| type     | 当前节点的类型                              |
| start    | 当前节点的起始位                            |
| end      | 当前节点的末尾                              |
| loc      | 当前节点所在的行列位置 起始于结束的行列信息 |
| errors   | File节点所持有的特有属性，可以不用理会      |
| program  | 包含整个源代码，不包含注释节点              |
| comments | 源代码中所有的注释会显示在这里              |



我们通常关注的节点是program，因为源码对应的AST语法子树结构都在program节点中。我们展开program节点，然后对照着JS源码逐步分析：

![https://img.heshipeng.com/1656603410051.jpg](https://img.heshipeng.com/1656603410051.jpg)

可以看到程序主要由三部分组成，一个是函数定义，一个是变量定义，一个是表达式语句。



我们接着展开FunctionDeclaration：

![image-20220630234230480](https://img.heshipeng.com/image-20220630234230480.png)

可以看到somewhat这个函数，主要由2部分组成，一个是变量定义，一个是for语句。



# Code->AST

`@babel/parser`能将`javascript`代码解析成AST，具体代码如下：

```js
const parser = require("@babel/parser");

var code = `
var a = 123;
function somewhat() {
	console.log("Hello, AST");
}
`;

let ast = parser.parse(code);
console.log(JSON.stringify(ast, null, '\t'));
```



# travel AST

## 使用path进行遍历

在使用 `enter` 遍历所有节点的时候，参数 `path` 会传入当前的路径，可以根据`path`进行各种判断，继而进行各类操作。



编写如下的遍历代码：

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
    enter(path) {
        console.log('当前路径类型', path.type); // 打印当前路径类型
        console.log('当前路径源码：', path.toString()); // 打印当前路径所对应的源代码
    }
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);
```

深度遍历的过程中，输出每一个节点的类型与其对应的源码。结果如下：

![https://img.heshipeng.com/1656605283688.jpg](https://img.heshipeng.com/1656605283688.jpg)



可以看到，使用path方式对AST遍历时，是从Program节点开始的，并不是File根节点开始。事实上，不止是采用path方式，下面介绍的采用节点方式对AST进行遍历，都是通过travel模块来进行的。而**采用travel对AST进行遍历都是从Program节点开始**。



## 使用节点进行遍历

与使用path遍历不同，我们不用关心每个节点，只需要关注自己想要处理的那些节点。与path相同的是，path同样会作为参数传入。



修改之前遍历的代码，修改visitor如下：

```js
const visitor = {
    ForStatement(path) {
      console.log('当前路径 源码:\n', path.toString());
      console.log('当前路径 节点:\n', path.node.toString());
    }
}
```

我们只处理for-statement，输出其源码以及下面的节点。输出结果如下：

![image-20220701003649630](https://img.heshipeng.com/image-20220701003649630.png)



# AST->Code

在对AST进行遍历处理之后，需要把AST转化成我们需要的JS代码，用到的模块是`@babel/generator`。



以一段代码作为演示：

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
    BinaryExpression(path) {            // 寻找所有 二元表达式节点
      if (path.node.operator == '*') {  // 并且这个表达式节点的操作是做 乘法 
        path.node.operator = '+';       // 将操作改为 加
      }
    },
    Identifier(path) {                  
      if (path.node.name == 'squire') {
        path.node.name = 'plus';        // 将函数名改为plus
      }
    }
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast);
fs.writeFile(output_file, code, (err)=>{});
```

这段代码的作用是修改方法squire，将其从平方变为加法，主要做2步，第一步是将乘法变为加法，第二步是将squire重命名为plus。可以看到generator的用法很简单，接受的第一个参数是一个AST语法树，返回一个字符串，这个字符串就是全部的JS代码。



# 节点

## 节点类型

给出一张图，列出节点的类型，如下：

![https://img.heshipeng.com/1403732-20200713201324374-2129914519.png](https://img.heshipeng.com/1403732-20200713201324374-2129914519.png)

这些类型，都在@babel/types中定义。



当前节点的类型，通过`path.type`来获取。而判断当前节点的类型，有2种方式，如下：

```js
path.type === 'ForStatement'
path.isForStatement()
```

第一种方式是比较节点的type属性与节点类型是否一致。第二种方式则是调用每个属性对应的判断类型的方法，规则是在每个节点类型加上前缀`is`，然后按照驼峰式命名即可。比如`NumericLiteral`对应的是`isNumericLiteral`，`SwitchCase`对应的是`isSwitchCase`。



## 对Node进行增删改

### 创建node

`@babel/types`包含了各个节点的定义，可以通过使用`@babel/types`的类型名，查阅[`@babel/types`官方文档](https://babeljs.io/docs/en/babel-types)，获取对应类型的构造函数，创建对应类型的节点。

我们这里来做一个示范，比如创建一条语法`console.log("Hello,AST")`。我们先把这条语句放在 [https://astexplorer.net/](https://astexplorer.net/) 中看下这条语句应该对应的AST结构。这里说个小的Tips，在做反混淆的过程中，经常需要反复对照 [https://astexplorer.net/](https://astexplorer.net/) 这个网站去分析AST结构。分析结果如下：

![image-20220701020813706](https://img.heshipeng.com/image-20220701020813706.png)



可以看到，这条JS语句主要包含4个主要的部分，整个JS代码是一条表达式，所以最外层是一个`ExpressStatement`，然后具体是什么表达式呢？是一个`CallExpress`即一个方法调用。然后这个方法调用包含2部分：`MemberExpression`和`Arguments`，`console.log`显然是一个成员表达式，而`Hello,AST`则是这个方法调用传入的参数。



分析完之后，编写代码如下：

```js
const type = require("@babel/types");
const generator = require("@babel/generator").default;

var args = [type.StringLiteral("Hello,AST")]; // 方法调用参数
var callee = type.memberExpression(type.identifier("console"), type.identifier("log")); // 成员表达式分2个部分
var call_exp = type.callExpression(callee, args); // 方法调用，第一个参数是方法名，第二个参数是方法调用参数
var exp_statement = type.ExpressionStatement(call_exp); // 表达式

console.log(generator(exp_statement)['code']);
```



运行结果如下：

![https://img.heshipeng.com/WX20220701-022717.png](https://img.heshipeng.com/WX20220701-022717.png)



### 插入node

`NodePath.insertAfter()`方法用于在当前`path`前面插入节点，`NodePath.insertBefore()`方法用于在当前`path`后面插入节点。下面用一个实例来演示这2个方法的使用。



假设有一行代码为`var a = 1;`，我们的任务是在这行代码之前插入`let b = "Hello,AST"`，在其之后插入`const c = 2;`。方法一样，首先把这三行代码放到  [https://astexplorer.net/](https://astexplorer.net/)  上面分析，具体的分析过程不过多描述了，直接给出代码：

```js
const fs = require('fs');

const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
const generator = require("@babel/generator").default;

let jscode = "var a = 1;"

//转换为ast树
let ast    = parser.parse(jscode);

const visitor = {
    VariableDeclaration(path) {
    	// 定位到a节点
    	if (path.node.kind == 'var' && path.node.declarations[0].id.name == 'a') {
    		var variableDeclarator = types.variableDeclarator(id=types.Identifier("b"), init=types.StringLiteral("Hello,AST"));
    		var nodeBefore = types.VariableDeclaration(kind='let', declarations=[variableDeclarator]);
    		path.insertBefore(nodeBefore);
            
    		variableDeclarator = types.variableDeclarator(id=types.Identifier("c"), init=types.NumericLiteral(1));
    		var nodeAfter = types.VariableDeclaration(kind='const', declarations=[variableDeclarator]);
    		path.insertAfter(nodeAfter);
    	}
    }
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast);
console.log(code);

```



输出结果：

![https://img.heshipeng.com/WX20220701-030118.png](https://img.heshipeng.com/WX20220701-030118.png)



### 替换node

`NodePath.replaceInline` 方法用于替换对应path的节点。我们依旧给出一个例子。比如有一条JS语句`let a = 1`，想把它变为`let  a = 2`。



代码如下：

```js
const fs = require('fs');

const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
const generator = require("@babel/generator").default;

let jscode = "var a = 1;"

//转换为ast树
let ast    = parser.parse(jscode);

const visitor = {
    NumericLiteral(path) {
    	path.replaceInline(types.NumericLiteral(2));
        // 防止递归插入
    	path.stop();
    }
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast);
console.log(code);
```



运行结果如下：

![image-20220701104832811](https://img.heshipeng.com/WX20220701-105013.png)



### 删除node

`NodePath.remove()`用于删除路径对应的节点，由于是对`path`操作，所以务必注意不要误删。同样地，以一案例来讲解下删除节点，话不多说，直接上代码：

```js
const fs = require('fs');

const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;
const types = require("@babel/types");
const generator = require("@babel/generator").default;

let jscode = `
function sum(a, b) {
	var c = 1;
	return a + b;
} 
`

//转换为ast树
let ast    = parser.parse(jscode);

const visitor = {
    VariableDeclaration(path) {
    	path.remove();
    }
}

//some function code

//调用插件，处理源代码
traverse(ast, visitor);

//生成新的js code，并保存到文件中输出
let {code} = generator(ast);
console.log(code);
```

逻辑比较简单，遍历到变量定义的节点，然后调用path.remove删除即可。



运行结果：

![https://img.heshipeng.com/WX20220701-112346.png](https://img.heshipeng.com/WX20220701-112346.png)



# 作用域Scope 与 被绑定量Binding

## 作用域Scope

`@Babel`解析出来的语法树节点对象会包含作用域信息，这个信息会作为节点`Node`对象的一个属性保存，这个属性本身是一个`Scope`对象，其定义位于`node_modules/@babel/traverse/lib/scope/index.js`中。



查看基本作用域与绑定信息：

```js
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;

const jscode = `
function a() {
	return "Hello,AST";
}
function b() {
	return 1 + 2;
}
var c = "Wow";
`;
let ast = parser.parse(jscode);
const visitor = {
    "FunctionDeclaration"(path){
        console.log("\n\n这里是函数 ", path.node.id.name + '()')
        path.scope.dump();
    }
}

traverse(ast, visitor);
```



执行 `Scope.dump()`，会得到自底向上的 作用域与变量信息，得到结果：

![https://img.heshipeng.com/WX20220701-113832.png](https://img.heshipeng.com/WX20220701-113832.png)



输出查看方法：

* 每一个作用域都以`#`标识输出
* 每一个绑定都以`-`标识输出
* 对于单次输出，都是自底向上的先输出当前作用域，再输出父级作用域，再输出父级的父级作用域……
* 对于单个绑定`Binding`，会输出4种信息
  * constant 表示声明后，是否会被修改
  * references 指被引用次数
  * violations 则是被重新定义的次数
  * kind 是指函数声明类型。param 参数, hoisted 提升，var 变量， local 内部。



## 绑定 Binding

`Binding` 对象用于存储绑定的信息，这个对象会作为`Scope`对象的一个属性存在，同一个作用域可以包含多个 `Binding`。你可以在 `@babel/traverse/lib/scope/binding.js` 中查看到它的定义。



查看`Binding`信息：

```js
const parser = require("@babel/parser");
const traverse = require("@babel/traverse").default;

const jscode = `
function a() {
	var m;
	m++;
	return m;
}
function b() {
	let c = a() + 1; 
	return c;
}
var c = "Wow";
`;
let ast = parser.parse(jscode);
const visitor = {
    BlockStatement(path) {
    	var bindings = path.scope.bindings;
    	for (let binding in bindings) {
    		console.log("binding name: " + binding);
    		binding = bindings[binding];
    		console.log("binding type: " + binding.kind);
    		console.log("binding constant: " + binding.constant);
    		console.log("binding constantViolations: " + binding.constantViolations);
    		console.log("binding referenced: " + binding.referenced);
    		console.log("binding references: " + binding.references);
    	}
    }
}

traverse(ast, visitor);
```



输出信息如下：

![https://img.heshipeng.com/WX20220701-115258.png](https://img.heshipeng.com/WX20220701-115258.png)



可以看到，变量m类型是var，有被引用，且被引用次数是2；变量c则是let类型，也有被引用，被引用次数是1。



> 关于作用域与绑定的关系？一个代码块(比如函数，循环，逻辑判断分支等)就是一个作用域，而定义在作用域里面的变量就是一个绑定，绑定是依附在作用域上。



# 关于学习AST相关的资源整理

| 信息                      | 地址                                                         |
| ------------------------- | ------------------------------------------------------------ |
| AST在线解析               | https://astexplorer.net/                                     |
| babel中文文档             | https://www.babeljs.cn/docs/                                 |
| babel英文文档             | https://babeljs.io/docs/en/                                  |
| Github                    | https://github.com/babel/babel                               |
| 插件手册                  | https://blog.csdn.net/weixin_33826609/article/details/93164633#toc-visitors |
| babel各节点解释           | https://github.com/babel/babylon/blob/master/ast/spec.md     |
| babel简单剖析             | http://www.alloyteam.com/2017/04/analysis-of-babel-babel-overview/ |
| 淘宝前端团队写的babel相关 | https://fed.taobao.org/blog/taofed/do71ct/babel-plugins/     |
| babel到底将代码转换成什么 | http://www.alloyteam.com/2016/05/babel-code-into-a-bird-like/ |
| babel在线源码             | https://doc.esdoc.org/github.com/mason-lang/esast/class/src/ast.js~VariableDeclarator.html |

