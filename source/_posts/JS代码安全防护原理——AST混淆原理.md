---
title: JS代码安全防护原理——AST混淆原理
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-03 22:22:11
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---



#### 常量的混淆原理

本篇所用的demo如下。接下来的案例都是围绕这个demo进行混淆。

```js
Date.prototype.format = function(formatStr) {
    var str = formatStr;
    var Week = ['日', '一', '二', '三', '四', '五', '六'];
    str = str.replace(/yyyy|YYYY/, this.getFullYear());
    str = str.replace(/MM/, (this.getMonth() + 1) > 9 ? (this.getMonth() + 1).toString() : '0' + (this.getMonth() + 1));
    str = str.replace(/dd|DD/, this.getDate() > 9 ? this.getDate().toString() : '0' + this.getDate());
    return str;
}

console.log(new Date().format('yyyy-MM-dd'));
```



##### 对象属性的两种访问方式

看下面一段代码：

```js
function People(name) {
    this.name = name;
}
People.prototype.sayHello = function() {
    console.log('Hello');
}
var p = new People('zhang san');
console.log(p.name);    // zhang san
p.sayHello();           // Hello
console.log(p['name']); // zhang san
p['sayHello']();				// Hello
```

1. p.name这种方式name是一个标识符，必须明确出现在代码中，不能加密和拼接。
2. p['name']这种方式name是一个字符串。由于是字符串，所以访问的时候可以进行拼接和加密等操作。所以在JS混淆中，一般会选择这种方式访问属性。

所以改变对象属性的访问方式，是代码混淆的前提。

所以开篇提到的demo改变对象属性的访问方式之后，代码修改如下：

```js
window['Date']['prototype']['format'] = function(formatStr) {
    var str = formatStr;
    var Week = ['日', '一', '二', '三', '四', '五', '六'];
    str = str['replace'](/yyyy|YYYY/, this['getFullYear']());
    str = str['replace'](/MM/, (this['getMonth']() + 1) > 9 ? (this['getMonth']() + 1).toString() : '0' + (this['getMonth']() + 1));
    str = str['replace'](/dd|DD/, this['getDate']() > 9 ? this['getDate']().toString() : '0' + this['getDate']());
    return str;
}

console.log(new window['Date']()['format']('yyyy-MM-dd'));

```

Date是JS的内置对象，在JS中很多内置对象都属于window的属性。另外，代码中定义的全局变量都是全局对象window的属性，代码中定义的全局方法都是全局对象window的方法。全局对象的属性或者方法在调用的时候可以省略全局对象名。比如，new window.Date()等同于new Date()。由于把Date变成了字符串，所以前面必须加window。



##### 十六进制字符串

在JS中支持字符串的十六进制形式表示，所以可以用字符串的十六进制形式来代替原有的字符串。比如'yyyy-MM-dd'可以表示成'\x79\x79\x79\x79\x2d\x4d\x4d\x2d\x64\x64'。其实，0x79就是字母y的ASCII码的十六进制形式，其余的字母类推。可以用一个方法来完成十六进制字符串的转换。

```js
function hexEnc(code) {
    let hexStr = []
    for (let i = 0, s; i < code.length; i++) {
        s = code.charCodeAt(i).toString(16);
        hexStr += '\\x' + s;
    }
    return hexStr;
}
```

开篇的demo转换为十六进制字符串之后如下：

```js
window['\x44\x61\x74\x65']['\x70\x72\x6f\x74\x6f\x74\x79\x70\x65']['\x66\x6f\x72\x6d\x61\x74'] = function(formatStr) {
    var str = formatStr;
    var Week = ['\x65e5', '\x4e00', '\x4e8c', '\x4e09', '\x56db', '\x4e94', '\x516d'];
    str = str['\x72\x65\x70\x6c\x61\x63\x65'](/yyyy|YYYY/, this['\x67\x65\x74\x46\x75\x6c\x6c\x59\x65\x61\x72']());
    str = str['\x72\x65\x70\x6c\x61\x63\x65'](/MM/, (this['\x67\x65\x74\x4d\x6f\x6e\x74\x68']() + 1) > 9 ? (this['\x67\x65\x74\x4d\x6f\x6e\x74\x68']() + 1).toString() : '\x30' + (this['\x67\x65\x74\x4d\x6f\x6e\x74\x68']() + 1));
    str = str['\x72\x65\x70\x6c\x61\x63\x65'](/dd|DD/, this['\x67\x65\x74\x44\x61\x74\x65']() > 9 ? this['\x67\x65\x74\x44\x61\x74\x65']().toString() : '\x30' + this['\x67\x65\x74\x44\x61\x74\x65']());
    return str;
}

console.log(new window['\x44\x61\x74\x65']()['\x66\x6f\x72\x6d\x61\x74']('\x79\x79\x79\x79\x2d\x4d\x4d\x2d\x64\x64'));
```



##### unicode字符串

在JS中，字符串除了可以表示成十六进制的形式外，还支持使用unicode形式表示。比如：

1. 以var Week = ['日', '一', '二', '三', '四', '五', '六']为例，可以表示成var Week = ['\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d']
2. 非中文的情况，Date可以表示成'\u0044\u0061\u0074\u0065'

从上述例子不难看出，unicode形式就是\u开头，后面跟四位数的十六进制形式，不足四位的补0。可以通过以下代码完成unicode转换：

```js
function unicodeEnc(str) {
    var value = '';
  	for (var i = 0; i < str.length; i++) {
      	value += "\\u" + ("0000" + parseInt(str.charCodeAt(i)).toString(16)).substr(-4);
    }
    return value;
}
```

开篇的demo转换为unicode编码之后如下：

```js
window['\u0044\u0061\u0074\u0065']['\u0070\u0072\u006f\u0074\u006f\u0074\u0079\u0070\u0065']['\u0066\u006f\u0072\u006d\u0061\u0074'] = function(formatStr) {
    var str = formatStr;
    var Week = ['\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d'];
    str = str['\u0072\u0065\u0070\u006c\u0061\u0063\u0065'](/yyyy|YYYY/, this['\u0067\u0065\u0074\u0046\u0075\u006c\u006c\u0059\u0065\u0061\u0072']());
    str = str['\u0072\u0065\u0070\u006c\u0061\u0063\u0065'](/MM/, (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1) > 9 ? (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1).toString() : '\u0030' + (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1));
    str = str['\u0072\u0065\u0070\u006c\u0061\u0063\u0065'](/dd|DD/, this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']() > 9 ? this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']().toString() : '\u0030' + this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']());
    return str;
}

console.log(new window['\u0044\u0061\u0074\u0065']()['\u0066\u006f\u0072\u006d\u0061\u0074']('\u0079\u0079\u0079\u0079\u002d\u004d\u004d\u002d\u0064\u0064'));
```

unicode字符和十六进制字符串都能轻易的还原，即直接把字符串放到控制台打印即可还原。



##### 字符串的ASCII码混淆

首先关注2个方法。

```js
console.log('x'.charCodeAt(0));						 //120
console.log('b'.charCodeAt(0));						 //98
console.log(String.fromCharCode(120, 98)); //xb
```

charCodeAt方法表示把字符串转换成ASCII编码，fromCharCode表示把ASCII码转换为字符串形式。

可以通过以下代码将字符串变成字节数组。

```js
function stringToByte(str) {
  	var byteArr = [];
    for (var i = 0; i < str.length; i++) {
        byteArr.push(str.charCodeAt(i));
    }
  	return byteArr;
}
```

比如demo中的format可以用字节数组表示为[102, 111, 114, 109, 97, 116]，因此代码中的format可以表示成String.fromCharCode(102, 111, 114, 109, 97, 116)。注意，fromCharCode接受的参数类型不是数组，而是可变参数类型。如果非要传一个数组，可以使用String.fromCharCode.apply(null, [102, 111, 114, 109, 97, 116])。**JS的函数也是对象，可以给函数定义属性和方法，而函数本身也自带一些属性和方法。apply就是从函数的原型对象Function.prototype继承过来的方法**。



ASCII码混淆不仅可以用于混淆字符串，还可以用来做代码混淆。比如我们把代码`str = str['replace'](/yyyy|YYYY/, this['getFullYear']());`看作一个字符串：

```js
stringToByte("str = str['replace'](/yyyy|YYYY/, this['getFullYear']());");
// [ 115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59 ]
```

然后再把这个字符串当作代码执行即可。在JS中把字符串当作代码执行的有2个方法，eval和Function。其中eval用来执行一段代码，Function用来生成一个函数。

所以我们之前的demo可以改写如下：

```js
window['\u0044\u0061\u0074\u0065']['\u0070\u0072\u006f\u0074\u006f\u0074\u0079\u0070\u0065']['\u0066\u006f\u0072\u006d\u0061\u0074'] = function(formatStr) {
    var str = formatStr;
    var Week = ['\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d'];
    eval(String.fromCharCode(115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59))
    str = str['\u0072\u0065\u0070\u006c\u0061\u0063\u0065'](/MM/, (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1) > 9 ? (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1).toString() : '\u0030' + (this['\u0067\u0065\u0074\u004d\u006f\u006e\u0074\u0068']() + 1));
    str = str['\u0072\u0065\u0070\u006c\u0061\u0063\u0065'](/dd|DD/, this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']() > 9 ? this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']().toString() : '\u0030' + this['\u0067\u0065\u0074\u0044\u0061\u0074\u0065']());
    return str;
}

console.log(new window['\u0044\u0061\u0074\u0065']()['\u0066\u006f\u0072\u006d\u0061\u0074']('\u0079\u0079\u0079\u0079\u002d\u004d\u004d\u002d\u0064\u0064'));
```



##### 字符串常量加密

字符串加密的最核心思想是先把字符串加密得到密文，然后在使用之前，调用对应的函数去解密，得到明文。代码中仅仅出现解密函数和明密文。当然也可以使用不同的加密方法去加密字符串，然后再调用不同的解密函数去解密。字符串加密最简单的方式是Base64编码。



浏览器中自带Base64的编码与解码的函数，其中btoa用来编码，atob用来解码。但是实际的应用中最好是自己去实现，然后加以混淆。注意，字符串加密之后，需要把对应的解码函数放入其中，方能正常运转。比如开头的demo可以改写如下：

```js
window[atob('RGF0ZQ==')][atob('cHJvdG90eXBl')][atob('Zm9ybWF0')] = function(formatStr) {
    var str = formatStr;
    var Week = ['\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d'];
    str = str[atob('cmVwbGFjZQ==')](/yyyy|YYYY/, this[atob('Z2V0RnVsbFllYXI=')]());
    str = str[atob('cmVwbGFjZQ==')](/MM/, (this[atob('Z2V0TW9udGg=')]() + 1) > 9 ? (this[atob('Z2V0TW9udGg=')]() + 1).toString() : atob('MA==') + (this[atob('Z2V0TW9udGg=')]() + 1));
    str = str[atob('cmVwbGFjZQ==')](/dd|DD/, this[atob('Z2V0RGF0ZQ==')]() > 9 ? this[atob('Z2V0RGF0ZQ==')]().toString() : atob('MA==') + this[atob('Z2V0RGF0ZQ==')]());
    return str;
}

console.log(new window[atob('RGF0ZQ==')]()[atob('Zm9ybWF0')](atob('eXl5eS1NTS1kZA==')));
```

在实际的混淆应用中，标识符必须处理成没有语义的，不然很容易定位到关键代码。此外，建议减少使用系统函数，自己去实现相应的函数，因为不管怎样混淆，最终执行过程中，系统函数是固定的，通过Hook技术很容易定位到关键代码。



##### 数值常量加密

算法加密过程中，会使用一些固定的数值常量，如MD5中的常量0×67452301，0xefcdab89，0x98badcfe和0x10325476，以及SHA1中的常量0x67452301， 0xefcdab89，0x98badcte，0x10325476和0xc3d2e1f0。因此，在标准算法逆向中，会通过搜索这些数值量，来定位代码关键位置,或者确定使用的是哪个算法。当然，在代码中不一定会写十六进制形式，如0x67452301，在代码中可能会与成十进制的1732584193。 安全起见，可以把这些数值常量也进行简单加密。
可以利用位异或的特性来加密。例如，如果a^b=c,那么c^b=a。以SHA1算法中的0xc3d2e1f0常量为例，0xc3d2elf0^0x12345678=0xd1e6b788，那么在代码中可以用0xd1e66788^0x12345678来代替0xc3d2e1f0，其中0x12345678可以理解成密钥，它可以随机生成。



#### 增加JS逆向者的工作量

##### 数组混淆

看一行代码：

```js
console.log(new window.Date().getTime());
```

按照前面介绍的对象属性的2种访问方式，使用第二种改写如下：

```js
console['log'](new window['Date']()['getTime']());
```

这样产生了三个字符串，我们把三个字符串放在数组里面。

```js
var bigArr = ['Date', 'getTime', 'log'];
console[bigArr[2]](new window[bigArr[0]]()[bigArr[1]]());
```

这就是数组混淆。当代码有上千行，那么数组可以提取的字符串可能也有上千个，然后在代码中引用字符串的时候，全部以bigArr[1001]，bigArr[1002]这种去访问，这样更加不容易建立映射关系了。



在JS中，同一个数组可以存放各种类型，比如布尔，数值，字符串，数组，对象和函数等。因此可以把代码中的一部分函数提取到大数组中去。为了安全，通常对提取到数组中的字符串进行加密处理，把代码处理成字符串就可以进行加密了。比如，对于String.fromCharCode可以改写成：

```js
""["constructor"]["fromCharCode"]
```

最前面的表示任意字符串对象或者空字符串，constructor表示构造方法，这样`""["constructor"]`就相当于String。前面的demo处理成数组混淆的形式如下：

```js
var bigArr = [
    '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0', 'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode']
];

window[atob('RGF0ZQ==')][atob('cHJvdG90eXBl')][atob('Zm9ybWF0')] = function(formatStr) {
    var str = formatStr;
    var Week = [bigArr[0], bigArr[1], bigArr[2], bigArr[3], bigArr[4], bigArr[5], bigArr[6]];
    eval(bigArr[14](115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59));
    str = str[atob(bigArr[7])](/MM/, (this[atob(bigArr[8])]() + 1) > 9 ? (this[atob(bigArr[8])]() + 1).toString() : atob(bigArr[9]) + (this[atob(bigArr[8])]() + 1));
    str = str[atob(bigArr[7])](/dd|DD/, this[atob(bigArr[10])]() > 9 ? this[atob(bigArr[10])]().toString() : atob(bigArr[9]) + this[atob(bigArr[10])]());
    return str;
}

console.log(new window[atob(bigArr[11])]()[atob(bigArr[12])](atob(bigArr[13])));
```



##### 数组乱序

上边进行数组混淆之后，数组下标索引与数组成员是一一对应。比如引用bigArr[14]的地方，需要成员String.fromCharCode，而该数组的下标为14的成员刚好是这个方法。可以将数组成员打乱，这样在分析的时候就更加费力，然后在执行的时候，通过一个方法将打乱的数组还原，从而不影响正确的逻辑。

可以使用以下代码打乱数组：

```js
 var bigArr = [
    '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0', 'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode']
];

(function(arr, num) {
    var shuffer = function(nums) {
        while (--nums) {
            arr.unshift(arr.pop());
        }
    }
    shuffer(++num);
}(bigArr, 0x20));
```

可以使用以下代码还原数组：

```js
var bigArr = [
    'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode'], '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0'
];

(function(arr, num) {
    var shuffer = function(nums) {
        while (--nums) {
            arr['push'](arr['shift']());
        }
    }
    shuffer(++num);
}(bigArr, 0x20));
```

所以最前面的demo经过数组混淆和数组乱序之后，可以改写为如下：

```js
var bigArr = [
    'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode'], '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0'
];

(function(arr, num) {
    var shuffer = function(nums) {
        while (--nums) {
            arr['push'](arr['shift']());
        }
    }
    shuffer(++num);
}(bigArr, 0x20));

window[atob('RGF0ZQ==')][atob('cHJvdG90eXBl')][atob('Zm9ybWF0')] = function(formatStr) {
    var str = formatStr;
    var Week = [bigArr[0], bigArr[1], bigArr[2], bigArr[3], bigArr[4], bigArr[5], bigArr[6]];
    eval(bigArr[14](115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59));
    str = str[atob(bigArr[7])](/MM/, (this[atob(bigArr[8])]() + 1) > 9 ? (this[atob(bigArr[8])]() + 1).toString() : atob(bigArr[9]) + (this[atob(bigArr[8])]() + 1));
    str = str[atob(bigArr[7])](/dd|DD/, this[atob(bigArr[10])]() > 9 ? this[atob(bigArr[10])]().toString() : atob(bigArr[9]) + this[atob(bigArr[10])]());
    return str;
}

console.log(new window[atob(bigArr[11])]()[atob(bigArr[12])](atob(bigArr[13])));
```



##### 花指令

所谓花指令，就是添加一些没有意义却可以混淆视听的代码。以前面提到的demo中的某一行代码为例：

```js
str = str.replace(/MM/, (this.getMonth() + 1) > 9 ? (this.getMonth() + 1).toString() : '0' + (this.getMonth() + 1));
```

把this.getMonth() + 1这个二项式改写为：

```js
function _0x20abefx1(a, b) {
    return a + b;
}
// str = str.replace(/MM/, (_0x20abefx1(new Date().getMonth(), 1)) > 9 ? (_0x20abefx1(new Date().getMonth(), 1)).toString() : '0' + (_0x20abefx1(new Date().getMonth(), 1)));
```

本质是把一个二项式拆成三个部分，最左边和最右边的参数，中间的运算符可以封装成一个函数，这个函数没有什么意义，但是能瞬间增加代码量，从而增加JS你逆向者的工作量。

二项式转成函数时，还可以多级嵌套：

```js
function _0x20abefx1(a, b) {
    return a + b;
}

function _0x20abefx2(a, b) {
    return _0x20abefx1(a, b);
}
```

具有同样功能的二项式，可以调用不同的函数，比如：

```js
function _0x20abefx1(a, b) {
    return a + b;
}

function _0x20abefx2(a, b) {
    return a + b;
}

function _0x20abefx3(a, b) {
    return _0x20abefx1(a, b);
}

// str = str.replace(/MM/, (_0x20abefx1(new Date().getMonth(), 1)) > 9 ? (_0x20abefx2(new Date().getMonth(), 1)).toString() : '0' + (_0x20abefx3(new Date().getMonth(), 1)));
```

除了二项式转为函数可以使用花指令，函数调用表达式也可以处理成类似的花指令：

```js
function _0x20abefx1(a, b, c) {
    return a(b, c);
}

function _0x20abefx2(a, b, c) {
    return _0x20abefx1(a, b, c);
}

function _0x20abefx3(a, b) {
    return a + b;
}

str = str.replace(
    /MM/, 
    _0x20abefx2(_0x20abefx3, new Date().getMonth(), 1) > 9 ? (_0x20abefx2(_0x20abefx3, new Date().getMonth(), 1)).toString() : _0x20abefx2(_0x20abefx3, '0', (_0x20abefx2(_0x20abefx3, new Date().getMonth(), 1)))
);
```



##### jsfuck

jsfuck可以看作一种编码方式，可以把JS代码通过只用()，[]，+，!这6种字符表示的代码，并且可以正常阅读。比如数值常量8可以表示成：

```js
(!+[]+!![]+!![]+!![]+!![]+!![]+!![]+!![]+[])
```

[]表示空数组，+[]表示把空数组转换为数值然后进行运行，由于空数组是0，所以+[]表示数值0，!+[]表示对数值0取反为布尔值true。

JS中有七种值为false，其余都为true。这7种值为false, undefine, null, 0, -0, NaN和""。所以!![]为true。

JS中的+运算符作为一个二元运算符时，假如有一边是字符串则代表字符串拼接，否则代表数值相加。所以true + true = 2。



在实际的开发中，jsfuck的应用有限。只会应用于js文件中的一部分代码。主要它的代码量非常庞大，且易于还原，只需要将代码复制到console即可还原。在jsfuck的混淆中，通过用()来进行分组，如果我们遇到JS文件局部应用jsfuck的代码，可以先通过()将代码分组，然后逐组逐组的分析还原。



#### 代码执行流程的防护原理

##### 流程平坦化

在流程平坦化混淆中，会用到switch语句，因为switch语句中的case块是平级的，而且调换case块的前后顺序并不影响代码原先的执行逻辑。看一段代码：

```js
function test() {
    var a = 100;
    var b = a + 200;
    var c = b + 300;
    var d = c + 400;
    var e = d + 500;
    var f = e + 600;
    return f;
}
```

混淆test方法的执行流程为：首先把代码分块，且打乱代码块的顺序，分别添加到不同的case块中，方便起见，这里处理场一行代码对应一个case块的形式，代码如下：

```js
switch () {
    case '1':
        var c = b + 300;
    case '2':
        var e = d + 500;
    case '3':
        var d = c + 400;
    case '4':
        var f = e + 600;
    case '5':
        var b = a + 200;
    case '6':
        return f;
    case '7':
        var a = 100;
}
```

可以看到，当代码块打乱之后，如果想跟原先的执行顺序一样，那么case块的跳转顺序应该是7，5，1，3，2，4，6。只有case块按照这个流程执行，才能跟原始代码块的顺序保持一致。其次，需要一个循环，因为switch语句只计算一次switch表达式。整个代码改写如下：

```js
while (!![]) {
    switch () {
     case '1':
         var c = b + 300;
            continue;
     case '2':
         var e = d + 500;
            continue;
     case '3':
         var d = c + 400;
            continue;
     case '4':
         var f = e + 600;
            continue;
     case '5':
         var b = a + 200;
            continue;
     case '6':
         return f;
     case '7':
         var a = 100;
    }
    break;
}
```

这是一个死循环，假如函数有返回值，则执行到相应的case语句块后直接返回。假如函数没有返回值，则代码块执行到最后就需要让switch计算出来的表达式的值与每一个case的值都不匹配，那么就会执行最后的break来跳出循环。



接着我们需要构造一个分发器，里面记录代码块执行的真实顺序，例如var arrStr = '7|5|1|3|2|4|6'.split('|')，i=0。把这个字符串'7|5|1|3|2|4|6'通过split分割成一个数组。i作为计数器，每次递增，按顺序引用数组中的每一个成员。因此，switch中的表达式就可以写成switch(arrStr[i++])，完整代码如下：

```js
function test() {
    var arrStr = '7|5|1|3|2|4|6'.split('|')，i=0;
  while (!![]) {
        switch (arrStr[i++]) {
            case '1':
                var c = b + 300;
                continue;
            case '2':
                var e = d + 500;
                continue;
            case '3':
                var d = c + 400;
                continue;
            case '4':
                var f = e + 600;
                continue;
            case '5':
                var b = a + 200;
                continue;
            case '6':
                return f;
            case '7':
                var a = 100;
        }
        break;
    }   
}
```

如果函数没有返回值，即switch中没有return语句，最后一次递增i会导致数组越界，JS中数组越界不会报错，而是取出来为undefined，然后匹配不到任何的switch语句就会执行break跳出死循环。



在了解了流程平坦化之后，我们可以对之前的demo进一步混淆：

```js
var bigArr = [
    'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode'], '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0'
];

(function(arr, num) {
    var shuffer = function(nums) {
        while (--nums) {
            arr['push'](arr['shift']());
        }
    }
    shuffer(++num);
}(bigArr, 0x20));

function _0x20abefx1(a, b, c) {
    return a(b, c);
}

function _0x20abefx2(a, b, c) {
    return _0x20abefx1(a, b, c);
}

function _0x20abefx3(a, b) {
    return a + b;
}

window[atob('RGF0ZQ==')][atob('cHJvdG90eXBl')][atob('Zm9ybWF0')] = function(formatStr) {
    var arrStr = '7|5|1|3|2|4'.split("|"), i = 0;
    while (!![]) {
        switch (arrStr[i++]) {
            case '1':
                eval(bigArr[14](115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59));
                continue;
            case '2':
                str = str[atob(bigArr[7])](
    				/dd|DD/,
    				this[atob(bigArr[10])] > 9 ? this[atob(bigArr[10])]().toString() : _0x20abefx2(_0x20abefx3, atob(bigArr[9]), this[atob(bigArr[10])]())
);
                continue;
            case '3':
                str = str[atob(bigArr[7])](
    				/MM/,
    				_0x20abefx2(_0x20abefx3, this[atob(bigArr[8])](), 1) > 9 ? (_0x20abefx2(_0x20abefx3, this[atob(bigArr[8])](), 1)).toString() : _0x20abefx2(_0x20abefx3, atob(bigArr[9]), (_0x20abefx2(_0x20abefx3, this[atob(bigArr[8])](), 1)))
);
                continue;
            case '4':
                return str;
            case '5':
     			var Week = [bigArr[0], bigArr[1], bigArr[2], bigArr[3], bigArr[4], bigArr[5], bigArr[6]];
                continue;
            case '7':
     			var \u0073\u0074\u0072 = \u0066\u006f\u0072\u006d\u0061\u0074\u0053\u0074\u0072; 
                continue;
        }
        break;
    }
    return str;
}

console.log(new window[atob(bigArr[11])]()[atob(bigArr[12])](atob(bigArr[13])));
```



##### 逗号表达式混淆

逗号运算符的主要作用是把多个表达式或语句连接成一个复合语句。比如前面提到的test方法，可以改写为：

```js
function test() {
    var a, b, c, d, e, f;
    return a = 100, b = a + 200, c = b + 300, d = c + 400, e = d + 500, f = e + 600, f;
}
```

return语句通常只跟一个语句，但是逗号表达式可以把多个语句符合成一个语句，这样会依次执行前面的语句，然后把最后一条语句作为返回值。



再看一个例子：

```js
var a = (a = 100, a += 200)
console.log(a); // 300
```

括号会作为一个整体，先把括号里面的运算完，然后把这个整体的值赋值给a。明白了这个道理，我们再改写下test方法：

```js
function test() {
    var a, b, c, d, e, f;
    return f = (e = (d = (c = (b = (a = 100, a + 200), b + 300), c + 400), d + 500), e + 600)
}
```

可以进一步优化，这里声明了一系列的变量，可以把这些变量作为参数传入，同时，在每一个变量的赋值的逗号表达式中可以插入花指令：

```js
function test(a, b, c, d, e, f) {
    return f = (e = (d = (c = (b = (a = 100, b + 1000, c + 2000, d + 3000, e + 4000, f + 5000, a + 200), c + 2000, d + 3000, e + 4000, f + 5000, b + 300), d + 3000, e + 4000, f + 5000, c + 400), e + 4000, f + 5000, d + 500), f + 5000, e + 600)
}
```

虽然需要6个参数，但是实际上不传任何参数，依然是正确的代码逻辑。同时中间的花指令，b+1000，c+2000，d+3000，e+4000，f+5000是没有意义的。

![image-20220308150018181](https://img.heshipeng.com/202203081500383.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



我们对前面给的demo进行改写：

```js
var bigArr = [
    'eXl5eS1NTS1kZA==', ""['constructor']['fromCharCode'], '\u65e5', '\u4e00', '\u4e8c', '\u4e09', '\u56db', '\u4e94', '\u516d', 'cmVwbGFjZQ==', 'Z2V0TW9udGg=', 'MA==', 'Z2V0RGF0ZQ==', 'RGF0ZQ==', 'Zm9ybWF0'
];

(function(arr, num) {
    var shuffer = function(nums) {
        while (--nums) {
            arr['push'](arr['shift']());
        }
    }
    shuffer(++num);
}(bigArr, 0x20));

window[atob('RGF0ZQ==')][atob('cHJvdG90eXBl')][atob('Zm9ybWF0')] = function(formatStr, str, Week) {
    return str = (
        (str = (
                Week = (
                    \u0073\u0074\u0072 = \u0066\u006f\u0072\u006d\u0061\u0074\u0053\u0074\u0072,
                        [bigArr[0], bigArr[1], bigArr[2], bigArr[3], bigArr[4], bigArr[5], bigArr[6]]
                ),
                    eval(bigArr[14](115, 116, 114,  32,  61, 32, 115, 116, 114,  91, 39, 114, 101, 112, 108,  97, 99, 101,  39,  93,  40, 47, 121, 121, 121, 121, 124, 89,  89,  89,  89,  47, 44, 32, 116, 104, 105, 115, 91,  39, 103, 101, 116, 70, 117, 108, 108,  89, 101, 97, 114,  39,  93,  40, 41, 41,  59)),
                    str[atob(bigArr[7])](/MM/, (this[atob(bigArr[8])]() + 1) > 9 ? (this[atob(bigArr[8])]() + 1).toString() : atob(bigArr[9]) + (this[atob(bigArr[8])]() + 1))
            ),
                str[atob(bigArr[7])](/dd|DD/, this[atob(bigArr[10])]() > 9 ? this[atob(bigArr[10])]().toString() : atob(bigArr[9]) + this[atob(bigArr[10])]())
        )
    )
}

console.log(new window[atob(bigArr[11])]()[atob(bigArr[12])](atob(bigArr[13])));
```



#### 其它代码防护方案

##### eval加密

看一段代码：

```js
eval(function (p, a, c, k, e, r) {
    e = function (c) {
        return c.toString(36)
    };
    if ('0'.replace(0, e) == 0) {
        while (c--)
            r[e(c)] = k[c];
        k = [function (e) {
                return r[e] || e
            }
        ];
        e = function () {
            return '[2-8a-f]'
        };
        c = 1
    };
    while (c--)
        if (k[c])
            p = p.replace(new RegExp('\\b' + e(c) + '\\b', 'g'), k[c]);
    return p
}('7.prototype.8=function(a){b 2=a;b Week=[\'日\',\'一\',\'二\',\'三\',\'四\',\'五\',\'六\'];2=2.4(/c|YYYY/,3.getFullYear());2=2.4(/d/,(3.5()+1)>9?(3.5()+1).e():\'0\'+(3.5()+1));2=2.4(/f|DD/,3.6()>9?3.6().e():\'0\'+3.6());return 2};console.log(new 7().8(\'c-d-f\'));', [], 16, '||str|this|replace|getMonth|getDate|Date|format||formatStr|var|yyyy|MM|toString|dd'.split('|'), 0, {}))

```

传给eval的是一个匿名函数，而不是一个字符串，这就是说先通过匿名函数将加密的代码解密成字符串代码，然后再通过eval之心这串代码。所以eval加密跟eval关系不大，重要的是这个解密函数，eval只是执行解密后的结果，并不参与加解密。

通过解密函数解密出来的字符串代码为：

![image-20220308153442661](https://img.heshipeng.com/202203081534835.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

通过eval执行解密出来的字符串结果为：

![image-20220308153523029](https://img.heshipeng.com/202203081535577.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



##### 内存爆破

内存爆破是指在代码中加入死代码，正常情况下这段代码不执行。当检测到函数被格式化或者函数被Hook的时候，就跳转到这段代码执行，直到内存溢出，浏览器会提示Out of Memory程序奔溃。内存爆破的代码如下：

```js
var d = [0x1, 0x0, 0x0];
function b() {
    for(var i = 0x0, c = d.length; i < c; i++) {
        d.push(Math.round(Math.round()));
        c = d.length;
    }
}
```

上述代码中的for循环是一个死循环，但是代码写的又不像 while(true) 这样明显。尤其是代码混淆以后，会更具迷惑性。这段代码其实是从某网站简化而来，原先的代码如下：

```js
this['NsTJKl'] = [0x1, 0x0, 0x0]
......
0x4b1809['prototype']['xTDWoN'] = function(_0x597ca7) {
    for (var _0x3e27c4 = 0x0, _0x192434 = this['NsTJKl']['length']; _0x3e27c4 < _0x192434; _0x3e27c4++) {
        this['NsTJKl']['push'](Math['round'](Math['random']()))
        _0x192434 = this['NsTJKl']['length']
    }
    return _0x597ca7(this['NsTJKl'][0x0])
}

```

for循环的结束条件是 _03e27c4 < _0x92434，其中 _0x192434 的初始化值是数组的大小。看着像是一个遍历数组的操作，是在循环中，又往数组中push了成员，接着又重新给 _0x192434 赋值为数组的大小。这时这段代码就永远也不会结束了，直到内存溢出。



##### 检测代码是否被格式化

检测的思路很简单，在JS中，函数是可以转为字符串的。因此可以选择一个函数转为字符串，然后跟内置的字符串对比或者用正则匹配。函数转为字符串很简单：

```js
function add (a, b) {
  return a + b
}
console.log(add + '')
console.log(add.toString())

```

在调试窗口使用格式化之后，会产生一个后缀为：formatted的文件。之后这个文件中设置断点，触发断点后，会停在这个文件中，选中这个函数，鼠标悬停在上面，会显示出他原来没有格式化之前的样子。
