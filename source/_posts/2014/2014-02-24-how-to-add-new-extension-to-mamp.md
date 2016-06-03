---
layout: post
title: "MacOS MAMP怎样添加扩展"
date: 2014-02-24 00:52:12 +0800
comments: true
categories:
- Dev
tags:
- MAMP
- 编译扩展

---

在MacOS下开发，最常用的LAMP开发环境架设工具就是MAMP & MAMP PRO了。其使用确实简单，但是一旦你需要一些额外的扩展时，就需要自己想办法编译了，笔者知道给MAMP添加扩展会有不同的方法，但一下介绍的仍然是笔者认为最简单的方法，思路其实来自于Google。
<!-- more -->
首先你需要确定你的MAMP所使用的PHP版本，因为MAMP可以在PHP5.2, PHP5.3, PHP5.4之间切换的。

然后你需要下载相应版本的PHP源码，之后解压缩到MAMP对应PHP版本的include/php目录，这里我是为PHP5.3.20添加扩展，所以源码解压缩到以下目录，注意php目录里面就是源码本身了，不会再有php5.3.20这样的版本目录。

```
/Applications/MAMP/bin/php/php5.3.20/include/php
```

然后还有一点就是你需要把MAMP的对应版本的bin添加到环境变量。

```
export PATH=/Applications/MAMP/bin/php/php5.3.20/bin:$PATH
```

这样，你就可以在命令行直接使用MAMP当前对应的php版本指令了，这里我们用到的是pecl命令

```
pecl install memcache
```

然后，就会自动开始编译，而你需要做的仅仅是将编译好的扩展添加到php.ini，如果用的是MAMP PRO，需要注意php.ini的位置。

需要思考两个问题
1. 我们并没有使用MAMP官方的components包，因为我觉得这种方法更简单。
2. 我们必须要下载PHP源码，是因为编译过程中需要一些头文件MAMP软件包默认是没有的。

学会这种方式之后，我们就可以举一反三，彻底解决本地环境编译扩展的问题。
