---
title: JS逆向中用到的常见的编码和加密
top: false
cover: false
toc: true
mathjax: true
date: 2022-03-02 21:51:19
password:
summary:
tags: [JS, 逆向]
categories: [JS逆向]
---



#### ASCII码

##### Python代码演示

```python
_ = list(map(lambda x: print(f"ascii of {x} is", ord(x)), "abcd"))
# ascii of a is 97
# ascii of b is 98
# ascii of c is 99
# ascii of d is 100
```



#### Base64

##### 什么是Base64

Base64是一种基于64个可打印字符来表示二进制数据的表示方法。

##### Base64的字符集

* 数字 {0, 1, 2, 3, 4, 5, 6, 7, 8, 9} 共10位
* 大小写字母 {A, B, C, D, E, F, G, H, I, J, K, L, M, N, O, P, Q, R, S, T, U, V, W, X, Y, Z, a, b, c, d, e, f, g, h, i, j, k, l, m, n, o, p, q, r, s, t, u, v, w, x, y, z} 共52位
* 特殊符号 {+, /} 共2位

##### Base64编码过程

由于base64的字符集大小为64，那么，需要6个比特的二进制数作为一个基本单元表示一个base64字符集中的字符。因为6个比特有2^6=64种排列组合。

具体来说，编码过程如下：

1. 将每三个字节作为一组，共24bit，若不足24bit在其后补充0；
2. 将这24个bit分为4组，每一组6个bit；
3. 在每组前加00扩展为8个bit，形成4个字节，每个字节表示base64字符集索引；
4. 扩展后的8bit表示的整数作为索引，对应base64字符集的一个字符，这就是base64编码值；在处理最后的不足3字节时，缺一个字节索引字节取3个，最后填充一个=，；缺两个字节取2个索引字节，最后填充==。

解码时将过程逆向即可。

Base64字符集索引表：

![图片来源维基百科](https://img.heshipeng.com/202203212353849.png)****

##### 编码示例

**Man的base64编码**：

![图片来源维基百科](https://img.heshipeng.com/202203212359415.png)

1. 第一步，'M', 'a', 'n'的ASCII值分别为77, 97, 110，对应的二进制值分别为：01001101, 01100001, 01101110；取三个字节共24bit：010011010110000101101110。
2. 第二步，将这24bit分为4组，每组6个bit：010011, 010110, 000101, 101110。
3. 每组前面加00，形成4个字节的，00010011, 00010110, 00000101, 00101110, 即19, 22, 5, 46。
4. 根据索引表，对应的base64字符分别是T, W, F, u。

##### Python实现

```python
"""
base64实现
"""

import base64
import string

# base 字符集

base64_charset = string.ascii_uppercase + string.ascii_lowercase + string.digits + '+/'


def encode(origin_bytes):
    """
    将bytes类型编码为base64
    :param origin_bytes:需要编码的bytes
    :return:base64字符串
    """

    # 将每一位bytes转换为二进制字符串
    base64_bytes = ['{:0>8}'.format(str(bin(b)).replace('0b', '')) for b in origin_bytes]

    resp = ''
    nums = len(base64_bytes) // 3
    remain = len(base64_bytes) % 3

    integral_part = base64_bytes[0:3 * nums]
    while integral_part:
        # 取三个字节，以每6比特，转换为4个整数
        tmp_unit = ''.join(integral_part[0:3])
        tmp_unit = [int(tmp_unit[x: x + 6], 2) for x in [0, 6, 12, 18]]
        # 取对应base64字符
        resp += ''.join([base64_charset[i] for i in tmp_unit])
        integral_part = integral_part[3:]

    if remain:
        # 补齐三个字节，每个字节补充 0000 0000
        remain_part = ''.join(base64_bytes[3 * nums:]) + (3 - remain) * '0' * 8
        # 取三个字节，以每6比特，转换为4个整数
        # 剩余1字节可构造2个base64字符，补充==；剩余2字节可构造3个base64字符，补充=
        tmp_unit = [int(remain_part[x: x + 6], 2) for x in [0, 6, 12, 18]][:remain + 1]
        resp += ''.join([base64_charset[i] for i in tmp_unit]) + (3 - remain) * '='

    return resp


def decode(base64_str):
    """
    解码base64字符串
    :param base64_str:base64字符串
    :return:解码后的bytearray；若入参不是合法base64字符串，返回空bytearray
    """
    if not valid_base64_str(base64_str):
        return bytearray()

    # 对每一个base64字符取下标索引，并转换为6为二进制字符串
    base64_bytes = ['{:0>6}'.format(str(bin(base64_charset.index(s))).replace('0b', '')) for s in base64_str if s != '=']
    resp = bytearray()
    nums = len(base64_bytes) // 4
    remain = len(base64_bytes) % 4
    integral_part = base64_bytes[0:4 * nums]

    while integral_part:
        # 取4个6位base64字符，作为3个字节
        tmp_unit = ''.join(integral_part[0:4])
        tmp_unit = [int(tmp_unit[x: x + 8], 2) for x in [0, 8, 16]]
        for i in tmp_unit:
            resp.append(i)
        integral_part = integral_part[4:]

    if remain:
        remain_part = ''.join(base64_bytes[nums * 4:])
        tmp_unit = [int(remain_part[i * 8:(i + 1) * 8], 2) for i in range(remain - 1)]
        for i in tmp_unit:
            resp.append(i)

    return resp
```

特别注意：<span style="color: red">**Base64默认的字符集索引是上边给出的那张图片，有一些JS反爬会在Base64的字符集上面做文章，比如打乱字符集的顺序，导致编码之后的字符串无法用原来的字符集索引进行解码，这样让人误以为不是Base64编码，实际上依然使用的是Base64编码，只不过解码的时候用的是跟编码一样的打乱之后的字符集索引**</span>。比如我们将上面生成字符集索引的代码改为如下：

```python
base64_charset = list(string.ascii_uppercase + string.ascii_lowercase + string.digits + '+/')
random.shuffle(base64_charset)
base64_charset = "".join(base64_charset)
```

然后测试一下程序，结果如下：

![image-20220322002411118](https://img.heshipeng.com/202203220024628.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以看到随着每次使用的字符集索引表不同，导致每次Base64编码的结果也不同，但是如果解码跟编码使用同一套字符集索引表，依然可以正确的解码得到编码之前的内容。



#### md5信息指纹

**MD5消息摘要算法**（英语：MD5 Message-Digest Algorithm），一种被广泛使用的[密码散列函数](https://zh.wikipedia.org/wiki/密碼雜湊函數)，可以产生出一个128位（16个字符(BYTES)）的散列值（hash value），用于确保信息传输完整一致。



##### Python代码演示

```python
import hashlib


def md5(text: str):
    m = hashlib.md5()
    m.update(text.encode("utf-8"))
    return m.hexdigest()


def file_md5(filename: str):
    m = hashlib.md5()
    with open(filename, "rb") as f:
        for chunk in iter(lambda: f.read(4096), b""):
        	m.update(chunk)
            
    return m.hexdigest()


print(md5("123456"))  # e10adc3949ba59abbe56e057f20f883e
print(file_md5("xxx.json")) # eea7fb16ddf174c2f92cac5adb6ce7bc
```



#### AES 加密

AES，高级加密标准（Advanced Encryption Standard）。是用来替代 DES，目前比较流行的对称加密算法。



##### 对称加密

一方通过密钥将信息加密后，把密文传给另一方，另一方通过这个相同的密钥将密文解密，转换成可以理解的明文。



##### 非对称加密

1. A要向B发送信息，A和B都要产生一对用于加密和解密的公钥和私钥。
2. A的私钥保密，公钥告诉B；B的私钥保密，公钥告诉A。
3. A要给B发送信息时，A用B的公钥加密信息，因为A知道B的公钥。
4. A将这个信息发给B(已经用B的公钥加密信息)。
5. B收到信息后，B用自己的私钥解密A的信息，其它所有收到这个报文的人都无法解密，因为只有B才有解密的私钥。
6. 反过来，B给A发送信息时一样。



##### 对称加密与非对称加密的区别

* 对称加密与解密使用的同样的密钥，所以速度快，但由于需要将密钥在网络中传输，所以安全性不高。
* 非对称加密使用了一对密钥，公钥与私钥，所以安全性高，但加密与解密速度慢。
* 解决的方法是将对称加密使用非对称加密的公钥进行加密，然后发送出去，接收方使用私钥进行解密得到对称加密的密钥，然后双方使用对称加密进行信息的传输。



##### AES加密三要素

密钥，填充和模式。



1. 密钥

对称加密之所以对称就是因为这类算法对明文的加密和解密使用的是同一个密钥。AES支持三种长度的密钥：128位，192位，256位。



2. 填充

说到填充一定要说一下，AES分组加密的特性，AES加密并不是一股脑将明文加密成密文，而是把明文拆成一个个独立的明文块，且每个明文块128bit。假如有一段明文长度是196bit，如果按照每128bit一个明文块来拆分的话，第二个明文块只有64bit，不足128bit。这个时候怎么办呢？就需要对明文块进行填充。



填充的种类：

* NoPadding - 不作任何填充，但是要求明文长度必须是128位的整倍数。
* PKCS7Padding - 当明文块少于16个字节，在明文块末尾补充相应数量的字符（字符为缺少的字节数）。
* ZeroPadding - 用0进行填充，但是这种方式不推荐，因为经常出现明文块最后一块也是0的时候，解密经常出现错误。

演示PKCS7Padding：

比如一段明文为1，2，4，5，6，1，2，3，4，5当这10个字节进行加密的时候是不足16个字节的，如果用PKCS7Padding进行填充，应该是：1，2，4，5，6，1，2，3，4，6，6，6，6，6，6(差6位，补6，且补6个；如果是差7位，则补足7个7)。



3. 模式

AES的工作模式主要体现在把明文块加密成密文块的处理过程中，AES加密算法提供了五种不同的工作模式：CBC，ECB，CTR，CFB，OFB。

模式之间的主题思想是近似的，在处理细节上有一些差别。



* ECB模式是最简单的工作模式，在该模式下，每一个明文块的加密都是完全独立，互不干涉。

这样做的好处？简单；有利于并行计算。

这样做的缺点？相同的明文块经过加密会变成相同的密文块，因此安全性较差。



* CBC模式引入了一个新的概念：初始向量IV。

IV是用来做什么的呢？它的作用和MD5的加盐有些类似，目的是防止同样的明文块始终加密成同样的密文块。

CBC模式在每一个明文块加密前会让明文块和一个值先做异或操作。

IV作为初始化变量，参与第一个明文的异或，后续的每一个明文块和它前一个明文块所加密的明文块相异或。这样相同的明文块加密出来的密文块显然是不同的。

这样做的好处？安全性更高。

这样做的缺点？无法并行计算，性能上不如CBC模式；引入初始化向量IV，增加复杂度。



##### Python代码实现

1. **ECB 模式**

```python
from Crypto.Cipher import AES
import base64


BLOCK_SIZE = 16  # Bytes
pad = lambda s: s + (BLOCK_SIZE - len(s) % BLOCK_SIZE) * \
                chr(BLOCK_SIZE - len(s) % BLOCK_SIZE)
unpad = lambda s: s[:-ord(s[len(s) - 1:])]


def aesEncrypt(key, data):
    '''
    AES的ECB模式加密方法
    :param key: 密钥
    :param data:被加密字符串（明文）
    :return:密文
    '''
    key = key.encode('utf8')
    # 字符串补位
    data = pad(data)
    cipher = AES.new(key, AES.MODE_ECB)
    # 加密后得到的是bytes类型的数据，使用Base64进行编码,返回byte字符串
    result = cipher.encrypt(data.encode())
    encodestrs = base64.b64encode(result)
    enctext = encodestrs.decode('utf8')
    print(enctext)
    return enctext

def aesDecrypt(key, data):
    '''

    :param key: 密钥
    :param data: 加密后的数据（密文）
    :return:明文
    '''
    key = key.encode('utf8')
    data = base64.b64decode(data)
    cipher = AES.new(key, AES.MODE_ECB)

    # 去补位
    text_decrypted = unpad(cipher.decrypt(data))
    text_decrypted = text_decrypted.decode('utf8')
    print(text_decrypted)
    return text_decrypted


if __name__ == '__main__':
    key = '5c44c819appsapi0'

    data = 'herish acorn'

    ecdata = aesEncrypt(key, data)  # 0FyQSXu3Q9Q13JGf4F74jA==
    aesDecrypt(key, ecdata)			# herish acorn
```





2. **CBC 模式**

```python
from Crypto.Cipher  import AES
import base64


# 密钥（key）, 密斯偏移量（iv） CBC模式加密

BLOCK_SIZE = 16  # Bytes
pad = lambda s: s + (BLOCK_SIZE - len(s) % BLOCK_SIZE) * \
                chr(BLOCK_SIZE - len(s) % BLOCK_SIZE)
unpad = lambda s: s[:-ord(s[len(s) - 1:])]

vi = '0102030405060708'

def AES_Encrypt(key, data):
    data = pad(data)
    # 字符串补位
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, vi.encode('utf8'))
    encryptedbytes = cipher.encrypt(data.encode('utf8'))
    # 加密后得到的是bytes类型的数据，使用Base64进行编码,返回byte字符串
    encodestrs = base64.b64encode(encryptedbytes)
    # 对byte字符串按utf-8进行解码
    enctext = encodestrs.decode('utf8')
    return enctext


def AES_Decrypt(key, data):
    data = data.encode('utf8')
    encodebytes = base64.decodebytes(data)
    # 将加密数据转换位bytes类型数据
    cipher = AES.new(key.encode('utf8'), AES.MODE_CBC, vi.encode('utf8'))
    text_decrypted = cipher.decrypt(encodebytes)
    # 去补位
    text_decrypted = unpad(text_decrypted)
    text_decrypted = text_decrypted.decode('utf8')
    print(text_decrypted)
    return text_decrypted

if __name__ == '__main__':
    key = '5c44c819appsapi0'

    data = 'herish acorn'
    enctext = AES_Encrypt(key, data)	# svAg4qrFNphvwS47DLSb2A==
    print(enctext)						# herish acorn

    AES_Decrypt(key, enctext)
```



##### NodeJS代码实现

1. **ECB模式**

```js
var CryptoJS = require("crypto-js");
const key = "ABC1234567891234";

function encrypt(text) {
    return CryptoJS.AES.encrypt(text, CryptoJS.enc.Utf8.parse(key), {
        iv: '',
        mode: CryptoJS.mode.ECB,
        padding: CryptoJS.pad.Pkcs7
    })
}

function decrypt(text) {
    var result = CryptoJS.AES.decrypt(text, CryptoJS.enc.Utf8.parse(key), {
        iv: '',
        mode: CryptoJS.mode.ECB,
        padding: CryptoJS.pad.Pkcs7
    });
    return result.toString(CryptoJS.enc.Utf8);
}

var text = "123456";
var encoded = encrypt(text);
console.log(encoded.toString());	// nWhAGHyLGTLV1dff9+PEUw==
console.log(decrypt(encoded));		// 123456
```



2. **CBC模式**

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

var text = "123456";
var encoded = encrypt(text);
console.log(encoded.toString());	// 6S3YRylMmp9vIFOplWxypw==
console.log(decrypt(encoded));		// 123456
```



##### AES加密流程总结

1. 把明文按照128bit拆分成若干个明文块。
2. 按照选择的填充方式来填充最后一个明文块。
3. 每一个明文块利用AES加密器和密钥加密成明文。
4. 拼接所有的密文块成为最终的密文结果。



#### 总结

![AES加密](https://img.heshipeng.com/202203172352889.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)
