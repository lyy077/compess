---
title: AST解JS混淆之去掉未被调用的函数
cover: false
toc: true
mathjax: true
date: 2022-05-28 21:44:53
summary:
tags: [AST, JS, 逆向]
categories: [JS逆向, AST反混淆]
---



本文接上一篇 [AST解JS混淆之去掉未被使用的变量](AST解JS混淆之去掉未被使用的变量)，去掉未被调用的函数，其思路与去掉未被使用的变量思路区别不大，在某些情况下，二者是通用的。如下：

需要清洗的代码依旧是：

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



跟前面去掉未使用变量的代码基本一致，略作修改：

```js
const visitor = {
    "VariableDeclarator|FunctionDeclaration"(path) {
        const {id} = path.node;
        const binding = path.scope.getBinding(id.name);
        // 如果变量被修改过，不去掉
        if (!binding || binding.constantViolations.length > 0) {
            return;
        }

        // 如果变量未被引用，去掉
        if (binding.referencePaths.length === 0) {
          path.remove();
        }
    }
}
```

注意"VariableDeclarator|FunctionDeclaration"这种写法，如果是想匹配多个节点，用|分割即可，但是得用双引号括起来。这样子就会对变量和函数同时应用下面的规则，即看变量或函数是否被修改过，是否存在引用。



结果如下：

```js
function a() {
  var m, k;
  m++;
  k++;
  return m;
}
```

可以看到多余的变量以及未被调用的函数被去掉了。



但是有一种特殊情况需要考虑到，那就是当遇到函数体里面定义的变量与函数名同名时，就会存在作用域的问题，这种情况下，运用上面的代码去清晰函数和变量时，就不会起作用了。



比如说如下代码：

```js
function a() {
    var a = "Hello,AST";
    console.log(a);
    return 1;
}
```

虽然函数a未被调用，但是变量a存在引用关系。我们使用`Scope.dump()`输出一下作用域与变量信息，如下：

![https://img.heshipeng.com/WX20220701-143413.png](https://img.heshipeng.com/WX20220701-143413.png)



可以看到有2个不同作用域的a，一个a是存在于作用域Program，另一个a是存在于作用域FunctionDeclaration。由于是同样的名字，所以我们在使用代码

```js
const {id} = path.node;
const binding = path.scope.getBinding(id.name);
```

它应该会使用哪个a呢？我们来看看getBinding的源码：

```js
getBinding(name) {
	let scope = this;
	
	do {
		const binding = scope.getOwnBinding(name);
		if (binding) {
			return binding;
		}
	} while (scope = scope.parent);
}
```

可以看到，不停的在遍历父级作用域，直到获取 binding 为止，由于是 do-while循环，所以会先从当前的作用域开始遍历。而对于上面的特例来说，会优先遍历FunctionDeclaration的作用域，因为这里函数作用域本身就是Program。



所以只需要对之前的代码略加修改即可：

```js
const visitor = {
    "VariableDeclarator|FunctionDeclaration"(path) {
        const {id} = path.node;
        const binding = path.scope.parent.getBinding(id.name);
        // 如果变量被修改过，不去掉
        if (!binding || binding.constantViolations.length > 0) {
            return;
        }

        // 如果变量未被引用，去掉
        if (binding.referencePaths.length === 0) {
          path.remove();
        }
    }
}
```

`const binding = path.scope.parent.getBinding(id.name);` 这个就是修改的地方，即直接从父作用域开始遍历，这样避免了同名导致的遍历错误作用域的问题。



