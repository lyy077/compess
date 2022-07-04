---
title: AST解JS混淆之去掉未被使用的变量
cover: false
toc: true
mathjax: true
date: 2022-05-28 20:32:08
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



在JS混淆的过程中，加入很多无辜的变量，从头到尾都没有使用过，这样子可以冗余一部分代码，达到混淆视听的目的，这也是JS混淆的一种常见手段。那么如何去除这些“无辜”的变量呢？这就是本篇文章需要讨论的主题。



比如下面一段代码，显然变量n和全局变量c是可以删除的。

```js
function a() {
	var m, n;
	m++;
	return m;
}
function b() {
	let c = a() + 1; 
	return c;
}
var c = "Wow";
```



删除没有使用的变量，核心代码如下：

```js
const visitor = {
    VariableDeclarator(path) {
        const {id} = path.node;
        // 获取binding信息
        const binding = path.scope.getBinding(id.name);
        // 如果变量被修改，则不能删除
        if (!binding || binding.constantViolations.length > 0) {
            return;
        }

        // 如果变量没有被引用，则可以删除
        if (binding.referencePaths.length === 0) {
          path.remove();
        }
    }
}
```

如果有阅读前面写过的一篇文章 [AST解JS混淆之AST基础](https://blog.heshipeng.com/AST%E8%A7%A3JS%E6%B7%B7%E6%B7%86%E4%B9%8BAST%E5%9F%BA%E7%A1%80/)，了解了作用域与Binding，则上面的代码并不难理解。
