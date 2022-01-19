---
title: Android逆向之Dex文件格式解析
top: false
cover: false
toc: true
mathjax: true
sticky: 1
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

![img](http://img.heshipeng.com/202201191407214.png)

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

![img](http://img.heshipeng.com/202201191448457.png)



然后用xxd命令打开上边生成的`dex`文件。

```shell
xxd Hello.dex
```

因为数据不长，这里直接贴出来整个内容：

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



#### 头文件

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

其中，`u`标识无符号整数，`u1`表示8位无符号整数，`u4`表示32位无符号整数。



|  成员名称  | 成员长度（字节） | 含义                                                         |
| :--------: | :--------------: | ------------------------------------------------------------ |
|   magic    |        8         | 魔数，必须出现在文件开头，标识其文件格式，用8个1字节的无符号数来表示，它可以分解为：文件标识 dex + 换行符 + DEX 版本 + 0， 这里是`64 65 78 0a 30 33 35 00`，表示dex的版本是35。 |
|  checksum  |        4         | `checksum` 是对去除 `magic` 、 `checksum` 以外的文件部分作 alder32 算法得到的校验值，用于判断 DEX 文件是否被篡改。这里的值是`0e34 5d55`(为啥不是`555d 340e`？因为是小端存储，关于大小端参见 [大端还是小端](http://blog.heshipeng.com/%E5%A4%A7%E7%AB%AF%E8%BF%98%E6%98%AF%E5%B0%8F%E7%AB%AF/)) |
| signature  |        20        | SHA1签名，除magic，checksum和它本身，作为文件的唯一标识。这里的值是`86 e9 52 1c b1 0d 57 08 b4 48 3f b8 fe ba cd 1c 22 d3 0f 65` |
|  fileSize  |        4         | dex文件的大小，包括头文件，`0000 02e0`                       |
| headerSize |        4         | dex头文件的大小，                                            |
|            |                  |                                                              |
|            |                  |                                                              |
|            |                  |                                                              |
|            |                  |                                                              |



