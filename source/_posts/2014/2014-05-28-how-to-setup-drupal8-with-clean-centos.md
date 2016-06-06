---
layout: post
title: "在全新的CentOS里安装Drupal8"
date: 2014-05-28 13:34:53 +0800
comments: true
categories:
- Ops
tags:
- CentOS
- Drupal8

---

本文在全新安装的CentOS6.5里安装Drupal8。

因为需要PHP5.4，而源里默认只有php5.3.3，所以按照[这个教程](http://webtatic.com/packages/php54/)安装了php5.4，然后从源里安装了httpd和mysql-server

<!-- more -->

以下都假设在root账户下：

```
yum install httpd mysql-server
```

然后配置了一个虚拟主机，Drupal8用的是alpha11版，注意要开启httpd的重写机制，因为默认就开启了clean url。

本来Drupal8是可以自动下载语言包的，安装时提示语言包下载不下来，于是我选择了安装英文版。

为了体验Drupal8，Profile选择的是Standard。

之后检测依赖扩展，结果需要继续安装缺失的扩展：

```
yum install php54w-gd php54w-pdo php54w-dom php54w-mbstring php54w-mysql
```

之后别忘了reload httpd

```
/etc/init.d/httpd reload
```
