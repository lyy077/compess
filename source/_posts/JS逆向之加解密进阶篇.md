---
title: JS逆向之加解密进阶篇
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-22 20:43:08
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---

#### 不一样的加密算法

##### 栅栏密码

栅栏密码(Rail-fence Cipher)就是把要加密的明文分为N个一组，然后把每组的第一个字符组合，每组的第二个字符组合...每组的第N个(最后一个分组可能不足N个)个字符组合，最后把他们全部连接起来就是密文。这里以2栏栅栏加密为例。

![image-20220322162944513](https://img.heshipeng.com/202203221629016.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

以一句话Are you ok为例：

1. 去空格，结果为Areyouok
2. 分组，结果为Ar ey ou ok

3. 重组
    * 第一组：Aeoo
    * 第二组：ryuk
4. 生成密文，结果为：Aeooryuk



**Python代码演示**

```python
def encode(flag, num):
    from math import ceil
    length = len(flag)
    lines = ceil(length / num)
    arr = [flag[i:i+num] for i in range(0, lines * num, num)]
    flag = ''
    for i in range(len(arr[0])):
        for j in arr:
            try:
                flag += j[i]
            except:
                pass

    return flag


def decode(flag, num):
    length = len(flag)   # flag的长度
    lines = length // num   # 判断共有几层并减一
    remainder = num * (lines + 1) - length  # 相差的数量
    # 补全flag
    result = flag[:length-lines*remainder]
    for i in range(remainder-1, -1, -1):
        result += flag[length - (i + 1) * lines:length - i * lines] + '*'
    # 还原flag
    lines += 1
    arr = [result[i:i + lines] for i in range(0, len(result), lines)]
    flag = ''
    for i in range(len(arr[0])):
        for j in arr:
            flag += j[i]
    return flag[:length]


if __name__ == '__main__':
    print(encode("Are you ok", 2))	# Aeyuor o k
    print(decode("Aeyuor o k", 2))	# Are you ok

```



##### 列位移密码

列位移密码(Columnar Transposition Cipher)是一种比较简单，易于实现的换位密码，通过一个简单的规则将明文打乱混合成密文。将明文填入事先约定填充的行列数，如果明文不能填充完表格，可以约定使用某个字母进行填充，然后根据密钥在字母表中出现的先后顺序进行编号，根据编码即可推出密文。

![image-20220322194007474](https://img.heshipeng.com/202203221940630.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

还是以那句话Are you ok为明文，以car为密钥，x为填充字符：

1. 去空格，结果为：Areyouok
2. 因为密钥car的长度为3，所以以3个字符为一个单位分组，如下：

|  c   |  a   |  r   |
| :--: | :--: | :--: |
|  A   |  r   |  e   |
|  y   |  o   |  u   |
|  o   |  k   |  x   |

最后一组只有2个元素不足三个，所以用x填充。由于密钥car的字母顺序排序是a>c>r，所以上述列排序也要根据这个顺序重新排列，如下：

|  a   |  c   |  r   |
| :--: | :--: | :--: |
|  r   |  A   |  e   |
|  o   |  y   |  u   |
|  k   |  o   |  x   |

重新排序完成之后，从第一列到最后一列按列取值组成的字符串就是密文了，很显然密钥是不能出现在明文中的，填充字符可以出现在明文中，所以每一列的第一个字符被忽略，最终的密文为：rokAyoeux。



**Python代码演示**

```python
from pycipher import ColTrans


ColTrans("car").encipher('Are you ok') # ROKAYOEU
ColTrans("car").decipher('ROKAYOEU')   # AREYOUOK
```



##### 凯撒密码

凯撒密码(Caesar Cipher或称凯撒加密，凯撒变换，变换加密，位移加密)是一种替换加密，明文中的所有字母都在字母表上向后(或向前)按照一个固定的数目进行偏移后被替换成密文。例，当偏移量是3的时候，所有的字母A都将被替换成D，B变成E，依此类推。

![image-20220322220707572](https://img.heshipeng.com/202203222207955.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

 依旧以Are you ok为例：

1. 去除空格，变为Areyouok

2. 以5作为偏移映射，A对应F，r对应w，e对应j，依此往下推，转换后的密文为Fwjdtztp。



**Python代码演示**

```python
def encrypt_char(char, key):
    return chr(ord('A') + (ord(char) - ord('A') + key) % 26)


def encrypt_message(message, key):
    message = message.upper()
    cipher = ''
    for char in message:
        if char not in ' ,.':
            cipher += encrypt_char(char, key)
        else:
            cipher += char
    return cipher


def decrypt_char(char, key):
    return chr(ord('A') + (ord(char) - ord('A') + 26 - key) % 26)


def decrypt_message(cipher, key):
    cipher = cipher.upper()
    message = ''
    for char in cipher:
        if char not in ' ,.':
            message += decrypt_char(char, key)
        else:
            message += char
    return message


if __name__ == '__main__':
    print(encrypt_message("Are you ok", 5))	# FWJ DTZ TP
    print(decrypt_message("FWJ DTZ TP", 5)) # ARE YOU OK
```



##### 其它加密

JSfuck仅使用6个字符，即[]+()!，来编写js程序。

jother是一种可以运用于javascript语言中利用少量字符构造精简的匿名函数方法对于字符串进行的编码方式。其中8个少量字符包括：!+()[]{}。只用这些字符就可以完成任意字符串的编码。

jjencode将JS代码转换成只有符号的字符串。

aaencode可以将JS代码转换成常用的网络表情，也就是我们说的颜文字JS加密。



#### JS加密上的实例分析与变化

##### 凯撒加密变种加密算法分析

密文：lQyjcvi|dcQR pktg t 2 cxl 9tixQR0 wxujzzxg0 gxijgc cxl 9tixQR V t 3 ZYY0rQRR

算法：

```js
function _$aU(_$Fj) {
    var _$U4 = _$Fj.length;
    var _$_L, _$cu = new Array(_$U4 - 1), _$h9 = _$Fj.charCodeAt(0) - 97;
    for (var _$1B = 0, _$lp = 1; _$lp < _$U4; ++_$lp) {
        _$_L = _$Fj.charCodeAt(_$lp);
        if (_$_L >= 40 && _$_L < 92) {
            _$_L += _$h9;
            if (_$_L >= 92)
                _$_L -= 52;
        } else if (_$_L >= 97 && _$_L < 127) {
            _$_L += _$h9;
            if (_$_L >= 127)
                _$_L -= 30;
        }
        _$cu[_$1B++] = _$_L;
    }
    return String.fromCharCode.apply(null, _$cu);
}
```

使用上述算法解密密文结果如下：

![image-20220323180744951](https://img.heshipeng.com/202203231807071.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到上面是一个无限debugger的字符串，然后通过eval调用可以达到无限debugger的目的，通过前面学习了凯撒密码之后，我们除了可以使用hook的方式绕过无限debugger，还可以使用凯撒密码达到目的，我们删除密文中的`wxujzzxg0`，然后执行：

![image-20220323181330460](https://img.heshipeng.com/202203231813548.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到，同样干掉了debugger关键字。



#### 常规加密算法的变种

##### 借鉴古典密码学改造现在密码学

看一段AES实际案例(某常见人机验证码代码部分案例)

```js
var CryptoJS = require("crypto-js");
const key = "ABC1234567891234";
const iv = "1234567812345678";

function encrypt(text) {
    return CryptoJS.AES.encrypt(text, CryptoJS.enc.Utf8.parse(key), {
        iv: CryptoJS.enc.Utf8.parse(iv),
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    })
}

function decrypt(text) {
    var result = CryptoJS.AES.decrypt(text, CryptoJS.enc.Utf8.parse(key), {
        iv: CryptoJS.enc.Utf8.parse(iv),
        mode: CryptoJS.mode.CBC,
        padding: CryptoJS.pad.Pkcs7
    });
    return result.toString(CryptoJS.enc.Utf8);
}


let encrypted = encrypt("123456");
var s = [];
for (var i = 0; i < encrypted.ciphertext.sigBytes; i++) {
    var x = encrypted.ciphertext.words[i >>> 2] >>> 24 - i % 4 * 8 & 255;
    s.push(x)
}
let data = encrypt(String.fromCharCode.apply(null, s)).toString();
console.log(encrypted.toString());	// 6S3YRylMmp9vIFOplWxypw==
console.log(data);					// u79HHbvmZtoSScagKO2JsQzytZH1L5SJGSh338DbaMA=
console.log(decrypt(data));			// é-ØG)L  o S© lr§
```

如上，假如加密代码出现了cryptojs关键字，然后让人以为是传统的cryptojs加密，但是实际上做了一点修改，导致如果直接拿data去做解密，会得到的乱码，达到混淆视听的目的。
