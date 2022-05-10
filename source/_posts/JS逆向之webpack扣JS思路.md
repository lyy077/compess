---
title: JS逆向之webpack扣JS思路
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-28 16:59:43
password:
summary:
tags: [JS, 逆向, WebPack]
categories: [JS逆向]

---

<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/iJAxeafvn8Y" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



#### 前言

这篇文章通过站在逆向的角度，解决遇到JS文件如果通过webpack的方式去组织代码模块如何扣JS代码，进行逆向分析的问题。



### 关于webpack

#### JS的自执行函数

IIFE 全称 Immediately-invoked Function Expressions，即自执行函数。这种模式本质上就是函数表达式（命名的或者匿名的）在创建后立即执行。当函数变成立即执行的函数表达式时，表达式中的变量不能从外部访问。IIFE 主要用来隔离作用域，避免污染。



##### 自执行函数的几种形式

1. 匿名函数前面加上一元操作符，后面加上 `()`。

```js
!function() {

}();

+function() {
    
}();

-function() {
    
}();

~function() {
    
}();
```



2. 匿名函数后边加上`()`，然后再用`()`将整个括起来。

```js
(function() {
    console.log("Hello, world!");
})();
```



3. 先用 `()` 将匿名函数括起来，再在后面加上 `()`。

```js
(function () {
    console.log("Hello, world!");
})();
```



4. 使用箭头函数表达式，先用 `()` 将箭头函数表达式括起来，再在后面加上 `()`。

```js
(() => {
  console.log("Hello, world!");
})();
```



5. 匿名函数前面加上 `void` 关键字，后面加上 `()`， `void` 指定要计算或运行一个表达式，但是不返回值。

```js
void function () {
    console.log("Hello, world!");
}();
```



有的时候，我们还有可能见到立即执行函数前面后分号的情况，比如：

```js
;(function () {
    console.log("Hello, world!");
}())

;!function () {
    console.log("Hello, world!");
}()
```



##### 自执行函数传参

将参数放在末尾的 `()` 里即可实现参数传递，如：

```js
var list = [1, 2, 3, 4, 5];

(function () {
    var sum = 0;
    for (var i = 0; i < list.length; i++) {
        sum += list[i];
    }
    console.log(sum);
})(list);

var dict = {name: "Bob", age: "20"};

(function () {
    console.log(dict.name);
})(dict);

(function (a, b, c, d) {
    console.log(a + b + c + d);
})(1, 2, 3, 4);
```



#### call, apply, bind三兄弟

`Function.prototype.call()`、`Function.prototype.apply()`、`Function.prototype.bind()` 都是比较常用的方法。它们的作用一毛一样，即**改变函数中的 `this` 指向**，它们的区别如下：

- `call()` 方法会立即执行这个函数，接受一个多个参数，参数之间用逗号隔开；
- `apply()` 方法会立即执行这个函数，接受一个包含多个参数的数组；
- `bind()` 方法不会立即执行这个函数，返回的是一个修改过后的函数，便于稍后调用，接受的参数和 `call()` 一样。



##### call

`call()` 方法接受多个参数，第一个参数 thisArg 指定了函数体内 this 对象的指向，如果这个函数处于非严格模式下，指定为 null 或 undefined 时会自动替换为指向全局对象（浏览器中就是 window 对象），在严格模式下，函数体内的 this 还是为 null。从第二个参数开始往后，每个参数被依次传入函数，基本语法如下：

```js
function call(thisArg, arg1, arg2, ...)
```

比如：

```js
function func1(a, b) {
    return a + b;
}

console.log(func1.call(null, 1, 2)); // 3

function func2() {
    return this[0] + this[1];
}

console.log(func2.call([1, 2])); // 3

function func3() {
    return this.a + this.b;
}

console.log(func3.call({"a": 1, "b": 2})); // 3
```



##### apply

`apply()` 方法接受两个参数，第一个参数 thisArg 与 `call()` 方法一致，第二个参数为一个带下标的集合，这个集合可以为数组，也可以为类数组，`apply()` 方法把这个集合中的元素作为参数传递给被调用的函数，基本语法如下：

```js
function.apply(thisArg, [arg1, arg2, ...])
```

比如：

```js
function func2() {
    return this[0] + this[1];
}

console.log(func2.apply([1, 2])); // 3

function func3() {
    return this.a + this.b;
}

console.log(func3.apply({"a": 1, "b": 2})); // 3
```



##### bind

`bind()` 方法和 `call()` 接受的参数是相同的，只不过 `bind()` 返回的是一个函数，基本语法如下：

```
function.bind(thisArg, arg1, arg2, ...)
```

比如：

```js
function func(a, b, c) {
    return a + b + c;
}

console.log(func.bind(null, 1, 2, 3)()); // 6

function func1() {
    return this[0] + this[1];
}

console.log(func1.bind([1, 2])()); // 3
```



#### 理解webpack

有了以上知识后，我们再来理解一下模块化编程，也就是前面所说的 webpack 写法：

```js
!function (allModule) {
    function useModule(whichModule, xxx, xxx, /*...*/) {
        allModule[whichModule].call(null, xxx, xxx, /*...*/);
    }
    useModule(0, 'abc', null, /*...*/)
}([
    function module0(param) {
        console.log("module0: " + param)
    },
    function module1(param) {
        console.log("module1: " + param)
    },
    function module2(param) {
        console.log("module2: " + param)
    },
]);
```

所谓webpack模块化编程，就是把一类函数——这些函数服务于某个或者某几个功能起作用——以列表或者对象的形式放在一起，封装到一个自执行的函数中，这些函数对外是不可见的，并且只对外暴露一个函数，这个函数叫做模块加载函数，外部通过这个加载函数访问自执行函数内部的函数，从而起到模块化的作用。



##### webpack模块化编程的JS代码结构特点

```js
function (x) {
    /*加载模块的方法*/
    function xx(yy) {
        x[yy].call(x1, x2, x3);
        // x[yy].apply([x1, x2, x3]);
        // x[yy].bind(x1, x2, x3)();
        
    }([
        // 可供加载的模块列表
        function(x1, x2, x3) {},
        function(x1, x2, x3) {}
    ]
     // 或者是
      {
      	"xxx": function(x1, x2, x3) {},
        "xxx": function(x1, x2, x3) {}
      }
     );
}
```

webpack模块化编程的JS代码特点是包含2个部分，上面是一个加载模块的方法，也叫模块加载器。下面是可供加载的模块列表。可供加载的模块列表是一个类数组(可以是数组，可以是对象)。



##### webpack扣JS的步骤

我们以这个网址——[G妹游戏](https://www.gm99.com/)——为例来介绍webpack扣js的一般步骤。我们的目的是抠出密码加密算法的那一部分JS代码。

找到我们要扣JS的那个文件，抓包，打断点分析的过程就不赘述了，直接贴文件：

![image-20220329005533139](https://img.heshipeng.com/202203290055624.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



1. 找到模块加载器(加载模块的方法)

根据前面提到的webpack模块化编程的JS代码结构特点，很显然这个模块加载为：

```js
function e(s) {
    var i = {};
	if (i[s])
		return i[s].exports;
	var n = i[s] = {
        exports: {},
        id: s,
        loaded: !1
    };
    return t[s].call(n.exports, n, n.exports, e),
    n.loaded = !0,
    n.exports
}
```



2. 构造一个自执行。可以是构造一个空的自执行，也可以是把网站的自执行JS扣下来，然后删除不必要的方法。如下：

```js
!function(t) {
    var i = {};
    function e(s) {
        if (i[s])
            return i[s].exports;
        var n = i[s] = {
            exports: {},
            id: s,
            loaded: !1
        };
        return t[s].call(n.exports, n, n.exports, e),
        n.loaded = !0,
        n.exports
    }
}();
```

注意构造的这个自执行方法只需要保留模块加载方法。



3. 找到并抠出调用的模块。从可供加载的模块列表中抠出包含我们需要的加密方法的模块。

![image-20220329161258366](https://img.heshipeng.com/202203291612665.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

通过抓包，找请求调用栈，定位到密码加密相关的地方。打上断点，调试这个方法，如下：

![image-20220329161353804](https://img.heshipeng.com/202203291613208.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

接着断点，接着调试：

![image-20220329161453244](https://img.heshipeng.com/202203291614519.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

最终找到需要调用的模块如图：

![image-20220329161649646](https://img.heshipeng.com/202203291616929.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

接着就是抠出这个调用模块，我们可以把整个文件复制下来，我们发现可供加载的模块是一个对象，用数字作为键，模块作为值，为了方便我们把键为0的叫做模块0，键为1的叫做模块1，依此类推。搜索关键代码`qe.prototype.encrypt`，根据代码缩进，找到封装`qe.prototype.encrypt`这个方法的模块是模块4，我们拷贝整个模块4的代码，粘贴到我们第二步构建的那个自执行方法的可供加载的模块列表中去，注意是以object的方法，不要用列表形式，同时给这个模块取一个名字，比如就叫做`encrypt`。



注意到我们前面跟栈的示意图，生成密码的地方是调用模块3的`encode` 方法，而`encode`方法是调用模块4的`encrypt`方法，我们上边已经抠出来了模块4，不要忘了抠出模块3(虽然模块3比较简单，完全可以自己写)。同样的给模块3重新取个名为`encode`。



最终模块3和模块4抠出来的代码如下：

```js
!function(t) {
    function e(s) {
        if (i[s])
            return i[s].exports;
        var n = i[s] = {
            exports: {},
            id: s,
            loaded: !1
        };
        return t[s].call(n.exports, n, n.exports, e),
            n.loaded = !0,
            n.exports
    }
}({
    "encrypt": function(t, e, i) {
        var s, n, r;
        s = function(t, e, i) {
            
            	    /* 省略若干行代码 */
                    
                    qe.prototype.decrypt = function(t) {
                        try {
                            return this.getKey().decrypt(ye(t))
                        } catch (t) {
                            return !1
                        }
                    }
                    ,
                    qe.prototype.encrypt = function(t) {
                        try {
                            return be(this.getKey().encrypt(t))
                        } catch (t) {
                            return !1
                        }
                    } 

                    /* 省略若干行代码 */

            })
        }
        .call(e, i, e, t),
        !(void 0 !== s && (t.exports = s))
    },
    "encode": function(t, e, i) {
        var s;
        s = function(t, e, s) {
            function n() {
                "undefined" != typeof r && (this.jsencrypt = new r.JSEncrypt,
                this.jsencrypt.setPublicKey("-----BEGIN PUBLIC KEY-----MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQDq04c6My441Gj0UFKgrqUhAUg+kQZeUeWSPlAU9fr4HBPDldAeqzx1UR92KJHuQh/zs1HOamE2dgX9z/2oXcJaqoRIA/FXysx+z2YlJkSk8XQLcQ8EBOkp//MZrixam7lCYpNOjadQBb2Ot0U/Ky+jF2p+Ie8gSZ7/u+Wnr5grywIDAQAB-----END PUBLIC KEY-----"))
            }
            var r = i(4);
            n.prototype.encode = function(t, e) {
                var i = e ? e + "|" + t : t;
                return encodeURIComponent(this.jsencrypt.encrypt(i))
            }
            ,
            s.exports = n
        }
        .call(e, i, e, t),
        !(void 0 !== s && (t.exports = s))
    }
});
```

扣完代码之后，我们将整个代码放到浏览器中执行一遍，验证下抠的JS代码没有问题：

![image-20220329163418015](https://img.heshipeng.com/202203291634343.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



4. 导出相应的模块方法。我们只需要导出模块加载函数就行，因为加密中用到的`encode`和`encrypt`方法都是通过模块加载函数加载的。我们可以在自执行方法外面定义一个全局变量，然后把模块加载函数赋值给这个全局变量，这样就可以从自执行函数内部导出模块加载方法了。代码如下：

```js
// 对外暴露模块加载函数
var _n = e;

!function(t) {
    function e(s) {
        if (i[s])
            return i[s].exports;
        var n = i[s] = {
            exports: {},
            id: s,
            loaded: !1
        };
        return t[s].call(n.exports, n, n.exports, e),
            n.loaded = !0,
            n.exports
    }
}({
    "encrypt": function(t, e, i) {
        // 省略函数体
    },
    "encode": function(t, e, i) {
        // 省略函数体
    }
});
```

修改完代码之后，我们在浏览器中验证一下：

![image-20220329181407362](https://img.heshipeng.com/202203291814720.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到成功加载了`encrypt`函数，加载`encode`函数的时候报错，我们打上断点调试，发现报错的那一行代码是：

```js
var r =  i(4);
```

这里的i是`encode`的第三个参数，是`t[s].call(n.exports, n, n.exports, e)`的第四个参数即模块加载函数(前面提到过call的第一个参数是影响this指针的参数)，表示调用模块4，然而模块4我们改名为`encrypt`(熟悉的模块3调用模块4，但是模块3盒模块4我们都改名了，😮‍💨真是给自己挖坑，其实完全跟原始JS的保持一致的命名)。所以这里我们只需要把这行代码改为：

```js
var r =  i("encrpty");
```

改好之后，我们再次在浏览器中调试，没有报错。



5. 编写代码测试。

![image-20220329200136091](https://img.heshipeng.com/202203292001605.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到测试结果符合预期。

![output-onlinepngtools](https://img.heshipeng.com/202203292004706.png)

#### 总结

这篇文章主要介绍了webpack模块化以及如何从中抠出相应的模块。抠JS的方法，总结起来分五步：

* 找到模块加载器
* 构造自执行
* 找到并抠出需要的模块
* 导出相应的模块方法
* 编写代码测试
