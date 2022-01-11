---
title: CPython源码剖析(一)
top: false
cover: false
toc: true
mathjax: true
sticky: 1
date: 2022-01-07 11:24:32
password:
summary:
tags: [源码]
categories: [Python]
---



<div align="middle"><iframe width="560" height="315" src="https://www.youtube.com/embed/-aMdBA00Ijc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></div>



### 编译源码(macOS)

在编译源码之前，首先获取源码，`git clone https://github.com/python/cpython`



在新下载的cpython目录中，会发现以下子目录：

```bash
cpython/
│
├── Doc      ← Source for the documentation
├── Grammar  ← The computer-readable language definition
├── Include  ← The C header files
├── Lib      ← Standard library modules written in Python
├── Mac      ← macOS support files
├── Misc     ← Miscellaneous files
├── Modules  ← Standard Library Modules written in C
├── Objects  ← Core types and the object model
├── Parser   ← The Python parser source code
├── PC       ← Windows build support files
├── PCbuild  ← Windows build support files for older Windows versions
├── Programs ← Source code for the python executable and other binaries
├── Python   ← The CPython interpreter source code
└── Tools    ← Standalone tools useful for building or extending Python
```



有了源码之后，开始编译源码：

```shell
brew install openssl xz zlib #安装依赖
CPPFLAGS="-I$(brew --prefix zlib)/include" \
 LDFLAGS="-L$(brew --prefix zlib)/lib" \
 ./configure --with-openssl=$(brew --prefix openssl) --with-pydebug #执行configure脚本
 make -j2 -s #构建命令，如果是4核CPU改成-j4，8核改为-j8依次类推
```



构建需要几分钟时间，构建完成后，会生成一个`Python.exe`文件，执行`./python.exe`命令，可以看到有效的REPL：

![image-20220107172013585](http://r4ymql1hs.hb-bkt.clouddn.com/202201071720051.png)



