---
layout: post
title: "使用yum给CentOS安装htop"
date: 2014-03-03 10:51:30 +0800
comments: true
categories:
- Ops
tags:
- CentOS
- htop

---

我们一般认为htop比top更适合用来监控系统进程，图形化的界面，支持鼠标，可读性较好，并且有许多额外的功能，这里尽介绍怎样通过yum在CentOS上安装，不过源码编译安装也是非常简单的，这里就不再赘述。

默认CentOS的源里是没有htop的，所以主要就是要找到合适的源，这个对不同的系统版本，以及对32bit,64bit的操作系统来说，需要添加的源是不同的，而且其实有多个第三方源，你只需要记住这种方式，如果这个源失效了，可以找其他的源代替。

<!-- more -->

## 为CentOS 5.x添加RPM源


``` bash 32bit:
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el5.rf.i686.rpm
rpm -Uhv rpmforge-release*.rf.i386.rpm
```


``` bash 64bit:
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el5.rf.x86_64.rpm
rpm -Uhv rpmforge-release*.rf.x86_64.rpm
```

## 为CentOS 6.x添加RPM源

``` bash 32bit:
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.i686.rpm
rpm -Uhv rpmforge-release*.rf.i386.rpm
```

``` bash 64bit:
wget http://pkgs.repoforge.org/rpmforge-release/rpmforge-release-0.5.3-1.el6.rf.x86_64.rpm
rpm -Uhv rpmforge-release*.rf.x86_64.rpm
```

之后我们就可以使用yum安装htop了

``` bash
yum install -y htop
```
