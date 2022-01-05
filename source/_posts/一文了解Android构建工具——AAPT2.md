---
title: 一文了解Android构建工具——AAPT2
top: false
cover: false
password: 'null'
toc: true
mathjax: true
summary: 'null'
tags: [aapt2, android]
categories: [Android]
sticky: 1
date: 2022-01-01 10:54:45
---
<meta name="referrer" content="no-referrer"/>
<div align="middle"><iframe width="560" height="315" src="https://www.youtube.com/embed/tHv9XMzEnjo" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>

## 什么是AAPT2(Android Asset Packaging Tool)

在Android开发过程中，我们通过Gradle命令，启动一个构建任务，最终会生成构建产物“APK”文件。常规APK的构建流程如下：

![android构建.png](http://r4ymql1hs.hb-bkt.clouddn.com/FmSi0GYeL8d3wSapWsPae4kr28Om)

（引用自Google官方文档）

* 编译所有的资源文件，生成资源表和R文件；
* 编译Java文件并把class文件打包为dex文件；
* 打包资源和dex文件，生成未签名的APK文件；
* 签名APK生成正式包。

老版本的Android默认使用AAPT编译器进行资源编译，从Android Studio 3.0开始，AS默认开启了 AAPT2作为资源编译的编译器，目前看来，AAPT2也是Android发展的主流趋势，学习AAPT2的工作原理可以帮助Android开发更好的掌握APK构建流程，从而帮助解决实际开发中遇到的问题。

AAPT2的可执行文件随Android SDK的Build Tools一起发布，在Android Studio的build-tools文件夹中就包含AAPT2工具，目录为（SDK目录/build-tools/version/aapt2）。

![aapt2路径.png](http://r4ymql1hs.hb-bkt.clouddn.com/Fi1D2FYTzHg-7TmjhGnER91BjQ5H)

所以AAPT的作用，总结起来就是**在APK打包的过程中对静态资源文件进行编译，打包**。

这里可能会有一个疑问：

> Java文件需要编译才能生class文件，这个我能明白，但资源文件编译到底是干什么的？为什么要对资源做编译？

这里先插个眼，带着这个问题去深入学习AAPT，然后慢慢解开这个问题。

## AAPT2如何工作

和AAPT不同，AAPT2把资源编译打包过程拆分为两部分，即编译和链接：

* 编译阶段：将资源文件编译为二进制文件（flat)
* 链接阶段：将编译后的文件合并，打包成单独文件。

> 通过把资源编译拆分为两个部分，AAPT2能够很好的提升资源编译的性能。例如，之前一个资源文件发生变动，AAPT需要做一全量编译，AAPT2只需要重新编译改变的文件，然后和其他未发生改变的文件进行链接即可。

AAPT2常用命令：

| 二级命令 | 含义 |
| :---: | :---: |
|  compile| 编译资源，用于链接 |
| link | 链接资源至一个apk |
| ... | 其它命令，通过`aapt2 -h`查看 |

### Compile命令

Complie指令用于编译资源，AAPT2提供多个选项与Compile命令搭配使用，如下：

![aapt2命令选项.png](http://r4ymql1hs.hb-bkt.clouddn.com/FnCbyMrXOigcxISFpeojInoEP4Fh)

Compile的一般用法如下：

```bash
aapt2 compile path-to-input-files [options] -o output-directory/
```

> Compile 命令会对输入的资源文件的路径做校验，输入文件的路径必须满足path/resource-type[config]/file，否则会编译报错：error: bad resource path.

![aapt2编译报错.png](http://r4ymql1hs.hb-bkt.clouddn.com/FqQVM_68gVDZVPVS-x3kVehUf5Po)

新建一个目录drawable-hdpi，把用来测试的图片放在这个目录下，然后执行命令：

```bash
aapt2 compile drawable-hdpi/aapt2路径.png -o .
```

可以看到当前目录多了一个drawable-hdpi_aapt2.flat的文件。

在Android Studio中，可以在app/build/intermediates/merged_res/debug/ 目录下找到编译生成的.flat文件。

![android studio打包产生的flat文件.png](http://r4ymql1hs.hb-bkt.clouddn.com/FtTzF2KXRLIASXXu-IKcovhR9wGH)

当然Compile也支持编译多个文件。

```bash
aapt2 compile path-to-input-files1 path-to-input-files2 [options] -o output-directory/
```

编译整个目录,需要制定数据文件，编译产物是一个压缩文件，包含目录下所有的资源，通过文件名把资源目录结构扁平化。

```bash
aapt2 compile --dir .../res [options] -o output-directory/resource.ap_
```

我们找个简单的Android项目测试下，首先进到项目的`app/src/main`目录下，然后执行命令：

```bash
aapt2 compile --dir ./res -o resource.ap_
```

可以看到产生了一个压缩文件，用命令`unzip -lv resource.ap_`查看，可以看到里面是一堆`.flat`的文件

![aapt2 compile编译目录.png](http://r4ymql1hs.hb-bkt.clouddn.com/Fiu1UOXFwhx9Vj8_8jHACQLIXW7N)

随便找其中的一个文件打开，是乱码的，那么这个FLAT文件是到底是什么？我们先接着看aapt2的链接阶段，晚点再探讨这个话题。

### Link命令

在链接阶段，AAPT2 会合并在编译阶段生成的所有中间文件（.flat文件），并将它们打包成ZIP包（最终APK的原型，由于不包括DEX文件且未签名，所以无法正常安装）。

链接资源使用link子命令，如下所示：

```bash
aapt2 link -o build/output.apk \
    -I $ANDROID_HOME/platforms/android-29/android.jar \
    --manifest build/intermediates/manifests/full/debug/AndroidManifest.xml \
    build/layout_activity_main.xml.flat \
    build/values_styles.arsc.flat \
    build/values_colors.arsc.flat \
    build/values_strings.arsc.flat \
    build/mipmap-xxxhdpi_ic_launcher.png.flat \
    build/mipmap-xxxhdpi_ic_launcher_round.png.flat
```

### AAPT2容器

FLAT文件是AAPT2编译的产物文件，也叫做AAPT2容器，文件由文件头和资源项两大部分组成。

1. 文件头

| Size(in bytes) | Field | Description |
| :---: | :---: | :--- |
| 4| magic| AAPT2容器文件标识，AAPT或0x54504141 |
| 4 | version | AAPT2容器版本 |
| 4 | entry_count | 容器中包含的条目数(一个flat文件中可以包含多个资源项) |


用`UltraEdit`编辑上面任意一个flat文件，内容如下：

![UE打开flat文件.png](http://r4ymql1hs.hb-bkt.clouddn.com/FsFqakTFKfyvq4U4deD3pwN0AMt6)

可以看到前4个字节是`0x54504141`(为啥不是0x41415054?因为是小端机器)，标识该文件是一个AAPT容器文件；紧接着4个字节是`0x00000001`表示该文件的版本号是1，接着4个字节是`0x000000001`表示该文件只有一个资源项。

> 关于二进制文件的逆向工具，类Unix系统都自带`xxd`命令，可以直接输出二进制文件的十六进制格式：
> 
> ```bash
> xxd values-de_values-de.arsc.flat
> ```
> 
> 或者使用`Vim`打开二进制文件，然后在命令模式中输入：
> 
> ```
> :%!xxd
> ```
> 
> 在Windows系统中，则可以用UE(UltraEdit)打开二进制文件分析。

2. 资源项

先用一个表格看下资源项的数据格式：

| Size(in bytes) | Field | Description |
| :---: | :---: | --- |
| 4| entry_type |资源类型，目前仅支持2种类型，RES_TABLE和RES_FILE |
| 8 | entry_length | 资源文件数据长度 |
| 若干 | entry_data | 资源数据 |

entry_type值分为两种类型:

* 当entry_type的值等于`0x00000000`时，为`RES_TABLE`类型。
* 当entry_type的值等于`0x00000001`时，为`RES_FILE`类型。

很显然我们这里打开的文件是一个RES_FILE类型，因为这里的entry_type为`0x00000001`。

接着8个字节`0x000000000001BFC8`表示该资源文件的长度，转为十进制，也即114632个字节。换算成以k为单位，就是111.95k，我们查看资源文件大小也刚好是112k。

![资源文件大小.png](http://r4ymql1hs.hb-bkt.clouddn.com/Ft1NO7v_GkvR_Rn5N_cfJYQL4doZ)

接下来就是资源数据了，接下来看下RES_FILE文件的格式：

| Size(inf bytes) | Field | Description |
| :---: | :---: | --- |
| 4 | header_size | header的长度 |
| 8 | data_size | data的长度 |
| header_size | header | 表示protobuf序列化的CompiledFile结构 |
| x | header_padding | 0-3个填充字节，用于data 32位对齐 |
| data_size | data | 资源文件内容(PNG, 二进制, XML或者protobuf序列化的XmlNode结构) |
| y | data_paddning | 0-3个填充字节，用于data 32位对齐 |

可以用一张图看下完整的RES_FILE类型的flat文件的格式：

![RES_FILE类型的flat文件格式.png](http://r4ymql1hs.hb-bkt.clouddn.com/FrdQkv6mPBGr6Akv0YhonaOTyH07)

接着看刚才打开的.flat文件，`0000003F`4个字节，也即63表示该RES_FILE的header长度是63个字节，然后接着8个字节`0x000000000001BF7B`表示数据文件的长度，也即114555个字节。

> 这里的data_length和上面的entry_length不是同一个长度，data_length指的是编译之前原始文件的数据长度(这里指原始png文件)，而entry_length指的是编译之后整个flat文件的长度。

接下来就是header部分，前面计算了header的长度是63位，所以这里63个字节表示header，然后紧接着一个`00`是header_padding用来做32位对齐。

然后接下来的114555个字节就是原始文件的数据内容(也即是png的内容)，最后一个`00`是用来填充数据做32位对齐的。

总结一下，我们前面计算了整个flat文件的大小是114632个字节，114632=4(用来存储header的长度)+8(用来存储data的长度)+63(header)+1(1个填充字节，用来填充header进行32位对齐)+114555(编译之前整个png的文件长度)+1(1个填充字节，用来填充data进行32位对齐)

最后用一张图看下上边分析的RES_FILE格式：

![RES_FILE格式图示.png](http://r4ymql1hs.hb-bkt.clouddn.com/FqcguIF45SpTniymtYf6H89zRQQ5)

另一种格式`RES_TABLE`格式比较简单，其实就是ResourceTable的`protobuf`序列化结果。数据结构如下：

```protobuf
// Top level message representing a resource table.
message ResourceTable {
    // 字符串池
    StringPool source_pool = 1;
    // 用于生成资源id
    repeated Package package = 2;
    // 资源叠加层相关
    repeated Overlayable overlayable = 3;
    // 工具版本
    repeated ToolFingerprint tool_fingerprint = 4;
}
```

资源表（ResourceTable）中包含：

**StringPool**：字符串池，字符串常量池是为了把资源文件中的string复用起来，从而减少体积，资源文件中对应的字符串会被替换为字符串池中的索引。

```protobuf
message StringPool {
  bytes data = 1;
}
```

**Package**：包含资源id的相关信息。

```
// 资源id的包id部分，在 [0x00, 0xff] 范围内
message PackageId {
  uint32 id = 1;
}
// 资源id的命名规则
message Package {
  // [0x02, 0x7f) 简单的说，由系统使用
  // 0x7f 应用使用
  // (0x7f, 0xff] 预留Id
  PackageId package_id = 1;
  // 包名
  string package_name = 2;
  // 资源类型，对应string, layout, xml, dimen, attr等，其对应的资源id区间为[0x01, 0xff]
  repeated Type type = 3;
}
```

> 资源id的命令方式遵循0xPPTTEEEE的规则，其中PP对应PackageId，一般应用使用的资源为7f，TT对应的是资源文件夹的名成，最后4位为资源的id，从0开始。

## 总结

通过本文，了解到*AAPT2*（*Android* 资源打包工具）是一个构建工具，*Android Studio* 和 *Android Gradle Plugin* 使用它来编译和打包应用的资源。*AAPT2* 会解析资源、为资源编制索引，并将资源编译为针对 Android 平台进行过优化的二进制格式。

在本文的开头，我们有如下的问题：

> Java文件需要编译才能生.class文件，这个我能明白，但资源文件编译到底是干什么的？为什么要对资源做编译？

主要原因在于 *AAPT2* 将资源打包过程拆分成了两个阶段：「编译阶段」和「链接阶段」，为了在链接阶段得到资源更详细的信息，例如：资源名称、配置信息（*Configuration*） 等，因此，直接将资源的元信息连同资源本身一同编码进 *AAPT2* 容器文件中，这样，资源链接的过程可以完全与编译过程解耦了，而且，对于增量构建来说，这样大大提升了资源打包的性能。这样如果只是修改了其中若干个资源文件，则只需要对这些资源文件单独编译然后再链接，而不用对所有的资源文件重新链接，大大提高了性能。

参考链接：

* [AAPT2产物逆向](https://booster.johnsonlee.io/architecture/aapt2-output-reversing.html#flat-%E4%B8%8E-aapt-%E4%BA%A7%E7%89%A9%E7%9A%84%E5%85%B3%E7%B3%BB)
* [Android构建工具--AAPT2源码解析（一）](https://segmentfault.com/a/1190000040864174)

