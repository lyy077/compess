---
title: JS逆向之代码混淆的原理
top: false
cover: false
toc: true
mathjax: true
date: 2022-02-18 13:54:47
password:
summary:
tags: [逆向, JS]
categories: [JS逆向]
---



#### 对JS代码保护的方式

* 代码压缩：去除空格，换行等。
* 加密代码：eval，emscripten，WebAssembly等。
* 代码混淆：变量混淆，常量混淆，控制流扁平化，调试保护等。



##### Javascript加密实现

1. eval加密

利用Javascript中的eval方法执行一些不太可读的JS代码。  eval方法就是JS的一个执行器，它可以把其中的参数按照JS的语法进行解释并执行。所以这个加密只是把JS的代码变成eval的参数，其中的一些字符都会被按照特定的格式编码。

比如有一段代码：

```js
console.log("hello, world")
```

利用一些[在线的eval加密网站](https://www.w3cschool.cn/tools/index?name=evalencode)进行加密，结果如下：

```js
eval(function(p,a,c,k,e,d){e=function(c){return(c<a?'':e(parseInt(c/a)))+((c=c%a)>35?String.fromCharCode(c+29):c.toString(36))};if(!''.replace(/^/,String)){while(c--)d[e(c)]=k[c]||e(c);k=[function(e){return d[e]}];e=function(){return'\\w+'};c=1};while(c--)if(k[c])p=p.replace(new RegExp('\\b'+e(c)+'\\b','g'),k[c]);return p}('0.2("1, 3")',62,4,'console|hello|log|world'.split('|'),0,{}))
```

可以看到可读性很差，但是在console中可以正确执行：

![image-20220218144118421](http://img.heshipeng.com/202202181441490.png)

加密后的字符串可以解密得到原始的JS代码。



2. Emscripten

将JS的核心逻辑用C/C++来实现，然后通过Emscripten项目进行编译生成asm.js代码，最后被JS调用。



3. WebAssembly

跟Emscripten类似，将核心代码用C/C++来实现，然后生成wasm文件，最后用JS来调用。与前面不同的是，生成的中间文件格式不同，asm.js是文本文件，wasm是二进制格式。wasm运行速度更快，体积更小。



##### Javascript混淆技术

* 变量混淆

把变量名或者常量名变成一些无意义的，或者看起来比较乱的字符串，比如说16进制的字符串，从而达到降低代码可读性的目的。



* 字符串混淆

把字符串进行md5，base64或者rc4加密或者编码，确保代码里面不会通过搜索的功能从而得到原始字符串。降低了通过字符串寻找入口的风险。



* 属性加密

把JS的Object的key-value的映射关系混淆，从而更加难以寻找里面的一些逻辑。



* 控制流扁平化

额外增加一些控制流，或者无意义的控制流，从而使代码流程看起来更加复杂，可读性更加差。



* 僵尸代码注入

不会被执行的代码或对上下文没有任何影响的代码注入到我们的代码，可以形成对现有JS代码的阅读干扰。



* 代码压缩

去除代码的一些空格，回车，调试语句等扽，可以使文件变得更小，可以将多行代码变成一行代码变得更加难读。



* 反调试

基于浏览器的特性，对当前环境进行一些检验，然后让它无限debug或者定时debug，通过一些断点进行干扰。



* 多态变异

JS代码被调用时，一旦被调用，代码立刻发生变化，变成和原来完全不同的代码，代码功能不变，只是代码形式发生变化，避免代码被动态的分析和调试。



* 锁定域名

有一个检测机制，使得我们的Javascript代码只能在特定的域名下执行。



* 反格式化

如果我们对JS代码格式化之后，会有一些机制使得我们无法顺利的运行。



* 特殊编码

将JS代码编码成一些特别难读的代码，比如说中括号，叹号等等。



##### Javascript混淆的开源项目

* UglifyJS https://github.com/mishoo/UglifyJS2 

解析JS的抽象语法树，然后根据抽象的语法树，对变量进行重命名，然后进行一些压缩或者变异。



* terser https://github.com/terser/terser

和UglifyJS功能类似，增加了对ES6的支持。



* javascript-obfuscator https://github.com/javascript-obfuscator/javascript-obfuscator

该项目可以用来实现几乎所有的混淆效果，比如字符串的混淆，属性的加密，平坦化控制流等等。



* Jsfuck https://github.com/aemkeijsfuck

将JS里面的一些变量或者定义替换成比如说{}的表示，体积会变得非常大。



* AAEncode https://github.com/bprayudha/jquery.aaencode

把JS代码换成一些颜文字的形式。



* JJEncode https://github.com/ay86/jEncrypt

把JS代码替换成一些$，加号，中括号等等，降低可读性。



##### Javascript混淆的在线工具

* https://obfuscator.io
* https://www.sojson.com/jscodeconfusion.html
* https://www.jshaman.com/protect.html
* https://freejsobfuscator.com
* https://www.daftlogic.com/protects-online-javascript-obfuscator.html
* https://beautifytools.com/javascript-obfuscator.php



##### Javascript混淆的商业服务

* https://javascriptobfuscator.com

* https://jscrambler.com

* https://stunnix.com



#### Javascript混淆实现

基于javascript-obfuscator开源项目，依赖Node.js，创建好工作空间，并安装好javascript-obfuscator包。

```shell
mkdir workspace && cd workspace
npm init
npm install --save-dev javascript-obfuscator
```

看下javascript-obfuscator的基本使用：

```js
const code = `
let x = '1' + 1
console.log('x', x)
`

const options = {
    compact: false,
    controlFlowFlattening: true
}

const obfuscator = require("javascript-obfuscator")
console.log(obfuscator.obfuscate(code, options).getObfuscatedCode())
```

首先定义一个变量code保存要混淆的代码，然后options是混淆时设置的一些参数，compact表示是否压缩成一行的形式，controlFlowFlattening表示控制流平坦化。执行结果如下：

```js
function _0x3636() {
    const _0x1b28ef = [
        '10gXHWCI',
        'log',
        '1186QQlqFf',
        '387777QhzDIE',
        '4QPhAWc',
        '7dmdYNq',
        '3383712sHKpqN',
        '3268150BmflmK',
        '338QyRTUY',
        '3280416bapPFy',
        '2348556uvulWF',
        '22yhcCAQ',
        '2778200CXJHeV'
    ];
    _0x3636 = function () {
        return _0x1b28ef;
    };
    return _0x3636();
}
const _0x5f084d = _0x2680;
(function (_0x416f08, _0x4aca4c) {
    const _0x3bfda0 = _0x2680, _0x3eb696 = _0x416f08();
    while (!![]) {
        try {
            const _0x2d197a = parseInt(_0x3bfda0(0x1d1)) / 0x1 * (-parseInt(_0x3bfda0(0x1ca)) / 0x2) + parseInt(_0x3bfda0(0x1d2)) / 0x3 * (parseInt(_0x3bfda0(0x1d3)) / 0x4) + parseInt(_0x3bfda0(0x1c9)) / 0x5 + -parseInt(_0x3bfda0(0x1cc)) / 0x6 * (parseInt(_0x3bfda0(0x1d4)) / 0x7) + parseInt(_0x3bfda0(0x1ce)) / 0x8 + parseInt(_0x3bfda0(0x1d5)) / 0x9 * (parseInt(_0x3bfda0(0x1cf)) / 0xa) + -parseInt(_0x3bfda0(0x1cd)) / 0xb * (parseInt(_0x3bfda0(0x1cb)) / 0xc);
            if (_0x2d197a === _0x4aca4c)
                break;
            else
                _0x3eb696['push'](_0x3eb696['shift']());
        } catch (_0x4e2954) {
            _0x3eb696['push'](_0x3eb696['shift']());
        }
    }
}(_0x3636, 0x59bb0));
let x = '1' + 0x1;
function _0x2680(_0x3f1f90, _0x4fb200) {
    const _0x36367e = _0x3636();
    return _0x2680 = function (_0x2680ac, _0x269526) {
        _0x2680ac = _0x2680ac - 0x1c9;
        let _0x72d9ee = _0x36367e[_0x2680ac];
        return _0x72d9ee;
    }, _0x2680(_0x3f1f90, _0x4fb200);
}
console[_0x5f084d(0x1d0)]('x', x);
```



关于一些混淆参数的说明：

| 参数名                   | 类型              | 含义                                                         |
| ------------------------ | ----------------- | ------------------------------------------------------------ |
| compact                  | Boolean           | 表示是否压缩，true为压缩，false为不压缩，压缩后会变为一行    |
| identifierNamesGenerator | String            | 变量名混淆的方式，默认为16进制混淆，设置为mangled为普通混淆  |
| stringArray              | Boolean           | 字符串混淆，是否把字符串变量拼成一个数组的形式               |
| rotateStringArray        | Boolean           | 字符串混淆，是否对上边提到的数组进行反转                     |
| stringArrayEncoding      | Boolean或者String | 字符串混淆，是否对字符串编码，默认为base64，设置为false为不编码，设置为base64表示使用base64编码，设置为rc4表示使用rc4编码 |
| unicodeEscapeSequence    | Boolean           | 字符串混淆，是否使用unicode编码                              |
| selfDefending            | Boolean           | 代码的自我保护，如果格式化后运行，会直接将浏览器卡死         |
| controlFlowFlattening    | Boolean           | 控制流平坦化                                                 |
| transformObjectKeys      | Boolean           | 对象键名替换                                                 |
| disableConsoleOutput     | Boolean           | 禁用控制台输出，会把一些控制台方法置为空                     |
| debugProtection          | Boolean           | 调试的保护，无限debug，定时debug，debugger关键字             |
| domainLock               | Array             | 域名锁定，只允许在特定的域名下执行，降低被模拟风险           |



