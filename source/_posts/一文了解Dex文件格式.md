---
title: 一文了解Dex文件格式
top: false
cover: false
toc: true
mathjax: true
date: 2022-01-17 17:27:25
password:
summary:
tags: [逆向, android]
categories: [Android]
---



<div align="center"><iframe width="560" height="315" src="https://www.youtube.com/embed/mXf3Klcn-sM" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



### Dex文件是什么

在明白什么是 Dex 文件之前，要先了解一下 JVM，Dalvik 和 ART。JVM 是 JAVA 虚拟机，用来运行 JAVA 字节码程序。Dalvik 是 Google 设计的用于 Android平台的运行时环境，适合移动环境下内存和处理器速度有限的系统。ART 即 Android Runtime，是 Google 为了替换 Dalvik 设计的新 Android 运行时环境，在Android 4.4推出。ART 比 Dalvik 的性能更好。Android 程序一般使用 Java 语言开发，但是 Dalvik 虚拟机并不支持直接执行 JAVA 字节码，所以会对编译生成的 .class 文件进行翻译、重构、解释、压缩等处理，这个处理过程是由 dx 进行处理，处理完成后生成的产物会以 .dex 结尾，称为 Dex 文件。Dex 文件格式是专为 Dalvik 设计的一种压缩格式。所以可以简单的理解为：Dex 文件是很多 .class 文件处理后的产物，最终可以在 Android 运行时环境执行。



### 构造Dex文件

Java代码转化为dex文件的流程如下：

![img](https://img.heshipeng.com/202201191407214.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

可以形象理解为Java源代码编译成`.class`文件，然后通过`dx`工具生成`dex`文件。



#### 从.java到.class

先建一个文件`Hello.java`，只是简单的打印一下`Hello, world`：

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, world!");
    }
}
```

进入该文件所在的目录，使用`javac`编译这个java文件。

```shell
javac Hello.java
```

javac 命令执行后会在当前目录生成 `Hello.class` 文件。



#### 从.class到.dex

上面生成的 .class 文件虽然已经可以在 JVM 环境中运行，但是如果要在 Android 运行时环境中执行还需要特殊的处理，那就是 dx 处理，它会对 .class 文件进行翻译、重构、解释、压缩等操作。

dx 处理会使用到一个工具 dx.jar，这个文件位于 SDK 中，具体的目录大致为 **你的SDK根目录/build-tools/任意版本** 里面。使用 dx 工具处理上面生成的Hello.class 文件，在 Hello.class 的目录下使用下面的命令：

```shell
dx --dex --output=Hello.dex Hello.class
```

执行完成后，会在当前目录下生成一个 Hello.dex 文件。这个 .dex 文件就可以直接在 Android 运行时环境执行。



### Dex格式详解

先看下`dex`文件的整体布局:

![image-20220120110317295](https://img.heshipeng.com/202201201103425.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



然后用xxd命令打开上边生成的`dex`文件。

```shell
xxd Hello.dex
```

因为数据不长，这里直接贴出来整个`Hello.dex`的内容：

```java
00000000: 6465 780a 3033 3500 555d 340e 86e9 521c  dex.035.U]4...R.
00000010: b10d 5708 b448 3fb8 feba cd1c 22d3 0f65  ..W..H?....."..e
00000020: e002 0000 7000 0000 7856 3412 0000 0000  ....p...xV4.....
00000030: 0000 0000 4002 0000 0e00 0000 7000 0000  ....@.......p...
00000040: 0700 0000 a800 0000 0300 0000 c400 0000  ................
00000050: 0100 0000 e800 0000 0400 0000 f000 0000  ................
00000060: 0100 0000 1001 0000 b001 0000 3001 0000  ............0...
00000070: 7601 0000 7e01 0000 8d01 0000 9901 0000  v...~...........
00000080: a201 0000 b901 0000 cd01 0000 e101 0000  ................
00000090: f501 0000 f801 0000 fc01 0000 1102 0000  ................
000000a0: 1702 0000 1c02 0000 0300 0000 0400 0000  ................
000000b0: 0500 0000 0600 0000 0700 0000 0800 0000  ................
000000c0: 0a00 0000 0800 0000 0500 0000 0000 0000  ................
000000d0: 0900 0000 0500 0000 6801 0000 0900 0000  ........h.......
000000e0: 0500 0000 7001 0000 0400 0100 0c00 0000  ....p...........
000000f0: 0000 0000 0000 0000 0000 0200 0b00 0000  ................
00000100: 0100 0100 0d00 0000 0200 0000 0000 0000  ................
00000110: 0000 0000 0100 0000 0200 0000 0000 0000  ................
00000120: 0200 0000 0000 0000 3102 0000 0000 0000  ........1.......
00000130: 0100 0100 0100 0000 2502 0000 0400 0000  ........%.......
00000140: 7010 0300 0000 0e00 0300 0100 0200 0000  p...............
00000150: 2a02 0000 0800 0000 6200 0000 1a01 0100  *.......b.......
00000160: 6e20 0200 1000 0e00 0100 0000 0300 0000  n ..............
00000170: 0100 0000 0600 063c 696e 6974 3e00 0d48  .......<init>..H
00000180: 656c 6c6f 2c20 776f 726c 6421 000a 4865  ello, world!..He
00000190: 6c6c 6f2e 6a61 7661 0007 4c48 656c 6c6f  llo.java..LHello
000001a0: 3b00 154c 6a61 7661 2f69 6f2f 5072 696e  ;..Ljava/io/Prin
000001b0: 7453 7472 6561 6d3b 0012 4c6a 6176 612f  tStream;..Ljava/
000001c0: 6c61 6e67 2f4f 626a 6563 743b 0012 4c6a  lang/Object;..Lj
000001d0: 6176 612f 6c61 6e67 2f53 7472 696e 673b  ava/lang/String;
000001e0: 0012 4c6a 6176 612f 6c61 6e67 2f53 7973  ..Ljava/lang/Sys
000001f0: 7465 6d3b 0001 5600 0256 4c00 135b 4c6a  tem;..V..VL..[Lj
00000200: 6176 612f 6c61 6e67 2f53 7472 696e 673b  ava/lang/String;
00000210: 0004 6d61 696e 0003 6f75 7400 0770 7269  ..main..out..pri
00000220: 6e74 6c6e 0001 0007 0e00 0301 0007 0e78  ntln...........x
00000230: 0000 0002 0000 8180 04b0 0201 09c8 0200  ................
00000240: 0d00 0000 0000 0000 0100 0000 0000 0000  ................
00000250: 0100 0000 0e00 0000 7000 0000 0200 0000  ........p.......
00000260: 0700 0000 a800 0000 0300 0000 0300 0000  ................
00000270: c400 0000 0400 0000 0100 0000 e800 0000  ................
00000280: 0500 0000 0400 0000 f000 0000 0600 0000  ................
00000290: 0100 0000 1001 0000 0120 0000 0200 0000  ......... ......
000002a0: 3001 0000 0110 0000 0200 0000 6801 0000  0...........h...
000002b0: 0220 0000 0e00 0000 7601 0000 0320 0000  . ......v.... ..
000002c0: 0200 0000 2502 0000 0020 0000 0100 0000  ....%.... ......
000002d0: 3102 0000 0010 0000 0100 0000 4002 0000  1...........@...
```

接下来对照这个dex文件的内容一步步解析整个dex文件的格式。



#### header

在android源码中对dex文件的格式，有了详细的定义：

```c
struct DexHeader {
    u1  magic[8];           // 魔数
    u4  checksum;           // adler 校验值
    u1  signature[kSHA1DigestLen]; // sha1 校验值
    u4  fileSize;           // DEX 文件大小
    u4  headerSize;         // DEX 文件头大小
    u4  endianTag;          // 字节序
    u4  linkSize;           // 链接段大小
    u4  linkOff;            // 链接段的偏移量
    u4  mapOff;             // DexMapList 偏移量
    u4  stringIdsSize;      // DexStringId 个数
    u4  stringIdsOff;       // DexStringId 偏移量
    u4  typeIdsSize;        // DexTypeId 个数
    u4  typeIdsOff;         // DexTypeId 偏移量
    u4  protoIdsSize;       // DexProtoId 个数
    u4  protoIdsOff;        // DexProtoId 偏移量
    u4  fieldIdsSize;       // DexFieldId 个数
    u4  fieldIdsOff;        // DexFieldId 偏移量
    u4  methodIdsSize;      // DexMethodId 个数
    u4  methodIdsOff;       // DexMethodId 偏移量
    u4  classDefsSize;      // DexCLassDef 个数
    u4  classDefsOff;       // DexClassDef 偏移量
    u4  dataSize;           // 数据段大小
    u4  dataOff;            // 数据段偏移量
};
```

其中，`u`标识无符号整数，`u1`表示8位无符号整数即一个字节，`u4`表示32位无符号整数即四个字节。



下面用一个表格对照`Hello.dex`对每一个成员含义做一个简单的说明，后边会针对某些字段有一个详细的说明。

|   成员名称    | 成员长度（字节） | 含义                                                         |
| :-----------: | :--------------: | ------------------------------------------------------------ |
|     magic     |        8         | 魔数，必须出现在文件开头，标识其文件格式，用8个1字节的无符号数来表示，它可以分解为：文件标识 dex + 换行符 + DEX 版本 + 0， 这里是`64 65 78 0a 30 33 35 00`，表示dex的版本是35 |
|   checksum    |        4         | `checksum` 是对去除 `magic` 、 `checksum` 以外的文件部分作 `adler32` 算法得到的校验值，用于判断 DEX 文件是否被篡改。这里的值是`0e34 5d55`(为啥不是`555d 340e`？因为是小端存储，关于大小端参见 [大端还是小端](http://blog.heshipeng.com/%E5%A4%A7%E7%AB%AF%E8%BF%98%E6%98%AF%E5%B0%8F%E7%AB%AF/)) |
|   signature   |        20        | SHA1签名，除magic，checksum和它本身，作为文件的唯一标识。这里的值是`86 e9 52 1c b1 0d 57 08 b4 48 3f b8 fe ba cd 1c 22 d3 0f 65` |
|   fileSize    |        4         | dex文件的大小，包括头文件，这里是`0000 02e0`                 |
|  headerSize   |        4         | dex头文件的大小，这里是`0000 0070`                           |
|   endianTag   |        4         | 端存储标记，主要是用来判断大端存储还是小端存储。默认值是`1234 5678`，即小端存储，这里是`1234 5678` |
|   linkSize    |        4         | 文件链接段大小，为0则表示静态链接，这里是`0000 0000`         |
|    linkOff    |        4         | 文件链接段的偏移位置，如果链接段大小为0，则偏移位置也为0，这里是`0000 0000` |
|    mapOff     |        4         | DexMapList的文件偏移，这里是`0000 0240`，也即DexMapList的基址是`0000 0240` |
| stringIdsSize |        4         | dex文件包含的字符串数量，这里是`0000 000e`                   |
| stringIdsOff  |        4         | dex文件字符串偏移位置，这里是`0000 0070`                     |
|  typeIdsSize  |        4         | dex文件类型信息的数量，这里是`0000 0007`                     |
|  typeIdsOff   |        4         | dex文件类型信息的偏移位置，这里是`0000 00a8`                 |
| protoIdsSize  |        4         | dex文件方法声明的数量，这里是`0000 0003`                     |
|  protoIdsOff  |        4         | dex文件方法声明偏移位置，这里是`0000 00c4`                   |
| fieldIdsSize  |        4         | dex文件字段信息的数量，这里是`0000 0001`                     |
|  fieldIdsOff  |        4         | dex文件字段信息的偏移位置，这里是`0000 00e8`                 |
| methodIdsSize |        4         | dex文件方法的数量，这里是`0000 0004`                         |
| methodIdsOff  |        4         | dex文件方法的偏移位置，这里是`0000 00f0`                     |
| classDefsSize |        4         | dex文件类的数量，这里是`0000 0001`                           |
| classDefsOff  |        4         | dex文件类信息的偏移位置，这里是`0000 0110`                   |
|   dataSize    |        4         | dex文件数据区的大小，这里是`0000 01b0`                       |
|    dataOff    |        4         | dex文件数据区的偏移位置，这里是`0000 0130`                   |



下面针对部分字段，进一步理解：

##### 1. 验证checksum

通过前面表格，了解到`checksum` 是对去除 `magic` 、 `checksum` 以外的文件部分作 alder32 算法得到的校验值，这里我们先备份一下`Hello.dex`文件，然后用`UE(UltraEdit)`打开`Hello.dex`，删除`magic`，`checksum`信息，如下：

![image-20220120113433764](https://img.heshipeng.com/202201201134897.png)

保存之后，执行如下Python代码：

```python
import zlib

with open("Hello.dex", "rb") as f:
    print(zlib.adler32(f.read()))
```

输出结果是238312789，转为十六进制为`0e34 5d55`，正好是该文件的`checksum`。



##### 2. 验证signature

类似于在上面验证checksum文件，进一步删除`signature`，如图：

![image-20220120143546894](https://img.heshipeng.com/202201201435024.png)

保存之后，执行如下Python代码：

```python
import hashlib

with open("Hello.dex", "rb") as f:
    print(hashlib.sha1(f.read()).hexdigest())
```

输出结果是86e9521cb10d5708b4483fb8febacd1c22d30f65，正好是删除的signature。



##### 3. 验证fileSize

从备份的`Hello.dex`还原dex文件，然后前面表格得出整个dex文件的大小是`2e0`即736个字节，我们用`ll`命令验证下：

![image-20220120150229254](https://img.heshipeng.com/202201201502294.png)



##### 4. 验证headerSize

headerSize占`0000 0070`也即112个字节。我们通过DexHeader这个结构体可以算出来：

magic(8个字节)+checksum(4个字节)+signature(20个字节)+fileSize(4个字节)+ ... + dataOff(4个字节)=112字节。



#### string_ids

字符串id区域，这个区域是一个偏移量列表，每个偏移量对应一个真正的字符串资源，每个偏移量占32位，即4个字节。我们可以通过偏移量找到对应的实际字符串数据。

从`DexFile.h`中我们可以找到`DexStringId`的定义：

```c
struct DexStringId {
    u4 stringDataOff;      /* file offset to string_data_item */
};
```

通过注释可以看到，这个区域存的并不是真正的字符串，只是存储了真正字符串的偏移位置，stringIdsSize为`0000 000e`即14，stringIdsOff为`0000 0070`。我们找到地址`0000 0070h`，然后取出后边的4*14个字节：

```java
00000070: 7601 0000 7e01 0000 8d01 0000 9901 0000  v...~...........
00000080: a201 0000 b901 0000 cd01 0000 e101 0000  ................
00000090: f501 0000 f801 0000 fc01 0000 1102 0000  ................
000000a0: 1702 0000 1c02 0000
```



这里以第一个偏移为例，解释具体每个字符串偏移背后代表的真正字符串的，取出前4个字节，即`0000 0176`，然后找到`0000 0176h`这个地址：

![image-20220120164703702](https://img.heshipeng.com/202201201647843.png)

dex中的字符串采用了一种叫做MUTF-8这样的编码，它是经过传统的UTF-8编码修改的。在MTUF-8中，它的头部存放的是由uleb128编码的字符的个数。所以第一个字节`06`表示的含义是字符串的字节数是6个，然后我们往后推6个字节，即`3C 69 6E 69 74 3E`，对照ASCII码表含义如下：

| 字符        | 3C   | 69   | 6E   | 69   | 74   | 3E   |
| :---------- | ---- | ---- | ---- | ---- | ---- | ---- |
| 对应ASCII码 | <    | i    | n    | i    | t    | >    |

即\<init>。



其余的13个字符串可以按照这个步骤，依次分析出来，这里就不展开了，给出一个表格如下：

| 索引 | 偏移值    | 字符个数（十六进制） | 字符串十六进制内容                                           | 对应ASCII内容         |
| ---- | --------- | -------------------- | ------------------------------------------------------------ | --------------------- |
| 0    | 0000 0176 | 06                   | 3C 69 6E 69 74 3E                                            | \<init>               |
| 1    | 0000 017e | 0D                   | 48 65 6C 6C 6F 2C 20 77 6F 72 6C 64 21                       | Hello, world!         |
| 2    | 0000 018d | 0A                   | 48 65 6C 6C 6F 2E 6A 61 76 61                                | Hello.java            |
| 3    | 0000 0199 | 07                   | 4C 48 65 6C 6C 6F 3B                                         | LHello;               |
| 4    | 0000 01a2 | 15                   | 4C 6A 61 76 61 2F 69 6F 2F 50 72 69 6E 74 53 74 72 65 61 6D 3B | Ljava/io/PrintStream; |
| 5    | 0000 01b9 | 12                   | 4C 6A 61 76 61 2F 6C 61 6E 67 2F 4F 62 6A 65 63 74 3B        | Ljava/lang/Object;    |
| 6    | 0000 01cd | 12                   | 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 7E 67 3B        | Ljava/lang/String;    |
| 7    | 0000 01e1 | 12                   | 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 79 73 74 65 6D 3B        | Ljava/lang/System;    |
| 8    | 0000 01f5 | 01                   | 56                                                           | V                     |
| 9    | 0000 01f8 | 02                   | 56 4C                                                        | VL                    |
| a    | 0000 01fc | 13                   | 5B 4C 6A 61 76 61 2F 6C 61 6E 67 2F 53 74 72 69 7E 67 3B     | [Ljava/lang/String;   |
| b    | 0000 0211 | 04                   | 6D 61 69 6E                                                  | main                  |
| c    | 0000 0217 | 03                   | 6F 75 74                                                     | out                   |
| d    | 0000 021c | 07                   | 70 72 69 6E 74 6C 6E                                         | println               |



#### type_ids

类型id区域，索引的值对应字符串id区域偏移量列表中的某一项，每一个人偏移也是占4个字节。

从`DexFile.h`中我们可以找到`DexTypeId`的定义：

```c
/*
 * Direct-mapped "type_id_item".
 */
struct DexTypeId {
    u4  descriptorIdx;      /* index into stringIds list for type descriptor */
};
```

从注释可以看到如果我们要找到某个类型的值，需要先根据类型id列表中的索引值去字符串id列表中找到对应的项，这一项存储的偏移量对应的字符串资源就是这个类型的字符串描述。



`typeIdsSize`为`0000 0007`，`typeIdsOff`为`0000 00a8`，我们找到地址为`0000 00a8`然后取出后面7*4个字节。

```java
0300 0000 0400 0000 0500 0000 0600 0000 0700 0000 0800 0000 0a00 0000
```

以第一个偏移`0000 0003`为例，即索引为3，查上面的string_ids得到的字符串列表，即为`LHello;`。类似地，可以分析出来其余6个：

| 索引 | 对应string_idx的索引 | 类型                  |
| ---- | -------------------- | --------------------- |
| 0    | 0000 0003            | LHello;               |
| 1    | 0000 0004            | Ljava/io/PrintStream; |
| 2    | 0000 0005            | Ljava/lang/Object;    |
| 3    | 0000 0006            | Ljava/lang/String;    |
| 4    | 0000 0007            | Ljava/lang/System;    |
| 5    | 0000 0008            | V                     |
| 6    | 0000 000a            | [Ljava/lang/String;   |



#### proto_ids

方法原型id区，这个区块是一个方法原型 id 列表。

从`DexFile.h`中可以找到其定义：

```c
/*
 * Direct-mapped "proto_id_item".
 */
struct DexProtoId {
    u4  shortyIdx;          /* index into stringIds for shorty descriptor */
    u4  returnTypeIdx;      /* index into typeIds list for return type */
    u4  parametersOff;      /* file offset to type_list for parameter types */
};
```

各个字段的解释如下：

![img](https://img.heshipeng.com/202201211414388.png)



可以看到，这个数据结构由三个变量组成。第一个shortyIdx它指向的是我们上面分析的DexStringId列表的索引，代表的是方法声明字符串。第二个returnTypeIdx它指向的是 我们上边分析的DexTypeId列表的索引，代表的是方法返回类型字符串。第三个parametersOff指向的是DexTypeList的位置索引，这又是一个新的数据结构了，先说一下这里面 存储的是方法的参数列表。可以看到这三个参数，有方法声明字符串，有返回类型，有方法的参数列表，这基本上就确定了我们一个方法的大体内容。



`parametersOff`指向`DexTypeList`，我们看下`DexTypeList`的数据结构：

```c
struct DexTypeList {
    u4  size;               /* #of entries in list */
    DexTypeItem list[1];    /* entries */
};
```

包含2个字段，第一个是大小说的是DexTypeItem的个数。



那`DexTypeItem`又是什么呢？我们再看下其数据结构：

```c
struct DexTypeItem {
    u2  typeIdx;            /* index into typeIds */
};
```

 包含一个指向DexTypeId列表的索引，也就是代表参数列表中某一个具体的参数的位置。



`protoIdsSize`为`0000 0003`，`protoIdsOff`为`0000 00c4`，找到地址为`0000 00c4`然后取出后边3*12个字节(为啥是12，因为每一个proto_id数据结构占12个字节)：

```java
0800 0000 0500 0000 0000 0000
0900 0000 0500 0000 6801 0000
0900 0000 0500 0000 7001 0000
```

先对第一个方法原型进行分析，前四个字节`0000 0008`为shorty_ids，对应string_ids的索引，查上面string_ids字符串的表格知其为`V`，即方法描述短格式为`void()`。中间四个字节`0000 0005`为return_type_idx，对应type_ids的索引，查上面的type_ids类型区域表格知其为`V`，即返回值类型为`void`。最后四个字节为`0000 0000`，即代表方法无参数。

在看第二个方法原型。前四个字节`0000 0009`为short_ids，对应string_ids的索引，查上面string_ids字符串的表格知其为`VL`。中间四个字节为`0000 0005`为return_type_idx，对应type_idx的索引，查上面的type_ids类型区域表格知其为`V`，即返回值类型为`void`。最后四个字节为`0000 0168`，找到168h这个地址如下：

![image-20220121150146530](https://img.heshipeng.com/202201211501796.png)

这里`DexTypeList`数据结构，我们先看前4个字节，代表`DexTypeItem`的个数，`0000 0001`也就是1，说明只有一个`DexTypeItem`，每一个占2个字节，就是`00 03`，查看type_ids表格，找到索引为3的，即`Ljava/lang/String;`，说明有一个`String`类型的参数。第三个方法原型类似，就不展开了。整理三个方法原型如下表格：

| 索引 | 方法描述短格式 | 返回值类型 | 参数类型            | 方法原型                 |
| ---- | -------------- | ---------- | ------------------- | ------------------------ |
| 0    | V              | V          | 无参数              | void()                   |
| 1    | VL             | V          | Ljava/lang/String;  | void(java.lang.String)   |
| 2    | VL             | V          | [Ljava/lang/String; | void(java.lang.String[]) |



#### field_ids

成员id区。这个区域是一个类成员id区域列表。定义如下：

```c
/*
 * Direct-mapped "field_id_item".
 */
struct DexFieldId {
    u2  classIdx;           /* index into typeIds list for defining class */
    u2  typeIdx;            /* index into typeIds for field type */
    u4  nameIdx;            /* index into stringIds for field name */
};
```

各字段的解释如下：

![img](https://img.heshipeng.com/202201211552340.png)



这里我们`Hello.dex`的`fieldIdsSize`为`0000 0001`，说明只存在一个DexField，`fieldIdOff`为`0000 00e8`，找到地址为`e8h`然后取出后边8个字节，即`04 00 01 00 0c 00 00 00`，其中前2个字节`0004`，表示class_idx，表示成员所在的类在类型区域的索引，查表得`Ljava/lang/System;`。中间2个字节`0001`，表示type_idx，表示该成员自身的类型在类型区域的索引，查表得`Ljava/io/PrintStream;`。最后4个字节`0000 000c`，表示name_idx，表示该成员的名字在字符串区域的索引，查表得`out`。所以`Hello.dex`中仅包含一个成员为`java.io.PrintStream java.lang.System.out`。



#### method_ids

方法id区，这个方法是存储方法id的列表。数据格式为：

```c
/*
 * Direct-mapped "method_id_item".
 */
struct DexMethodId {
    u2  classIdx;           /* index into typeIds list for defining class */
    u2  protoIdx;           /* index into protoIds for method prototype */
    u4  nameIdx;            /* index into stringIds for method name */
};
```

解释如下：

![img](https://img.heshipeng.com/202201211612575.png)



`methodIdsSize`和`methodIdsOff`分表为`0000 0004`和`0000 00f0`，即该dex文件一共包含4个方法，方法id区的偏移地址为`f0h`，我们找到这个地址，然后取出后边4*(2+2+4)个字节，如下：

```java
0000 0000 0000 0000 
0000 0200 0b00 0000
0100 0100 0d00 0000 
0200 0000 0000 0000
```

然后根据`DexMethodId`数据结构，查上边的type_ids表格，proto_ids表格以及string_ids表格，这里就不一一展开了，结果整理如下：

| 序号 | class_idx             | proto_idx                | name_idx | 方法                                               |
| ---- | --------------------- | ------------------------ | -------- | -------------------------------------------------- |
| 0    | LHello;               | void()                   | \<init>  | void Hello.\<init>()                               |
| 1    | LHello;               | void(java.lang.String[]) | main     | void Hello.main(java.lang.String[])                |
| 2    | Ljava/io/PrintStream; | void(java.lang.String)   | println  | void java.io.PrintStream.println(java.lang.String) |
| 3    | Ljava/lang/Object;    | void()                   | \<init>  | void java.lang.Object.\<init>()                    |



#### class_def

类定义区。这个区域存储的是类定义的列表，具体的数据结构如下：

```c
/*
 * Direct-mapped "class_def_item".
 */
struct DexClassDef {
    u4  classIdx;           /* index into typeIds for this class */
    u4  accessFlags;
    u4  superclassIdx;      /* index into typeIds for superclass */
    u4  interfacesOff;      /* file offset to DexTypeList */
    u4  sourceFileIdx;      /* index into stringIds for source file name */
    u4  annotationsOff;     /* file offset to annotations_directory_item */
    u4  classDataOff;       /* file offset to class_data_item */
    u4  staticValuesOff;    /* file offset to DexEncodedArray */
};
```

各字段的含义如下：

![img](https://img.heshipeng.com/202201211647542.png)



在开始分析这个类的结构之前，先看`DexFile.h`中定义的一组枚举值：

```c
enum {
    ACC_PUBLIC       = 0x00000001,       // class, field, method, ic
    ACC_PRIVATE      = 0x00000002,       // field, method, ic
    ACC_PROTECTED    = 0x00000004,       // field, method, ic
    ACC_STATIC       = 0x00000008,       // field, method, ic
    ACC_FINAL        = 0x00000010,       // class, field, method, ic
    ACC_SYNCHRONIZED = 0x00000020,       // method (only allowed on natives)
    ACC_SUPER        = 0x00000020,       // class (not used in Dalvik)
    ACC_VOLATILE     = 0x00000040,       // field
    ACC_BRIDGE       = 0x00000040,       // method (1.5)
    ACC_TRANSIENT    = 0x00000080,       // field
    ACC_VARARGS      = 0x00000080,       // method (1.5)
    ACC_NATIVE       = 0x00000100,       // method
    ACC_INTERFACE    = 0x00000200,       // class, ic
    ACC_ABSTRACT     = 0x00000400,       // class, method, ic
    ACC_STRICT       = 0x00000800,       // method
    ACC_SYNTHETIC    = 0x00001000,       // field, method, ic
    ACC_ANNOTATION   = 0x00002000,       // class, ic (1.5)
    ACC_ENUM         = 0x00004000,       // class, field, ic (1.5)
    ACC_CONSTRUCTOR  = 0x00010000,       // method (Dalvik only)
    ACC_DECLARED_SYNCHRONIZED =
   // ...
};
```



`classDefsSize`和`classDefsOff`分别为`0000 0001`和`0000 0110`。即只有一个类定义，其偏移地址为`110h`，我们找到该地址，并取出32个字节：

```java
0000 0000 0100 0000 0200 0000 0000 0000 0200 0000 0000 0000 3102 0000 0000 0000
```

前4个字节代表类的类型，为`0000 0000`查表知为`LHello;`。接下来4个字节为类的访问权限，为`0000 0001`，记得我们上边刚提到的那个枚举值定义吗，1代表ACC_PUBLIC即Public访问权限。接下来4个字节`0000 0002`为父类对应的类型，查表知为`Ljava/lang/Object;`。然后四个字节`0000 0000`为这个类实现的接口在dex文件中的偏移，因为我们这个类没有实现接口，所以这里为`0000 0000`。紧接着的四个字节`0000 0002`查表知为`Hello.java`为该类类源码所在的文件。然后4个字节为该类的注解在文件中的偏移，很显然这个类没有注解，所以为`0000 0000`。接下来的4个字节则是表示该类的具体数据在文件中的偏移，这里先不讨论，后边会针对类数据区专门讨论。最后4个字节表示静态成员初始值列表在文件中的偏移，很显然我们这个类没有静态成员。整理一下如下：

| 类的类型 | 类的权限   | 父类的类型       | 实现的接口 | 类定义所在的文件 | 类注解 | 类具体数据        | 静态成员初始值列表 |
| -------- | ---------- | ---------------- | ---------- | ---------------- | ------ | ----------------- | ------------------ |
| Hello    | ACC_PUBLIC | java.lang.Object | 无         | Hello.java       | 无     | 偏移值`0000 0231` | 无                 |



### 总结

本文对dex文件结构进行了一个简单的剖析，让我们对dex文件结构有了一个基本的认识。最后附上一个dex文件结构图以及思维图帮助我们记忆dex文件结构。

dex文件层次结构图：

![img](https://img.heshipeng.com/202201211830586.png?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)

dex文件结构思维导图：

![img](https://img.heshipeng.com/202201211832915.awebp?watermark/2/text/5YWz5rOo5b6u5L-h5YWs5LyX5Y-377ya6YCG5ZCR5LiA5q2l5q2l/font/5a6L5L2T/fontsize/300)



### 参考资料

[Android逆向笔记 —— DEX 文件格式解析](https://juejin.cn/post/6844903847647772686)

[浅谈 Android Dex 文件](https://tech.youzan.com/qian-tan-android-dexwen-jian/)

[一文读懂 DEX 文件格式解析](https://cloud.tencent.com/developer/article/1663852)

[一篇文章带你搞懂DEX文件的结构](https://blog.csdn.net/sinat_18268881/article/details/55832757)

[DexFile.h](http://androidxref.com/9.0.0_r3/xref/dalvik/libdex/DexFile.h)

[Android软件安全与逆向分析-非虫](https://book.douban.com/subject/20556210/)
