---
layout: post
title: "CentOS yum安装jenkins笔记"
date: 2014-02-24 01:26:01 +0800
comments: true
categories:
- Ops
tags:
- CentOS
- Jenkins

---

因为工作需要，所以打算在一组服务器上配置Jenkins服务，搜索了一下，找到我操作起来最方便的方案，下面就陈述一下，大部分都是原文作者的功劳。

## 添加Jenkins的源

CentOS默认源里没有Jenkins

```
$ sudo wget -O /etc/yum.repos.d/jenkins.repo http://jenkins-ci.org/redhat/jenkins.repo
$ sudo rpm --import http://pkg.jenkins-ci.org/redhat/jenkins-ci.org.key
```
<!-- more -->

## 安装/检查JDK

注意，如果是agent服务器，不需要安装jenkins，只需要安装java即可

```
$ java -version
```
```
$ sudo yum install java-1.6.0-openjdk
```

## 安装Jenkins

```
$ sudo yum install jenkins
```

* Jenkins的WAR包位于: `/usr/lib/jenkins/`
* Jenkins的配置文件位于： `/etc/sysconfig/jenkins`，一般不需要配置
* Jenkins的home目录是：`/var/lib/jenkins`，开始的时候是空的，一般启动jenkins服务，就开始出现各种文件。
* Jenkins的日志文件是：`/var/log/jenkins/jenkins.log`

## 启动Jenkins

```
$ sudo service jenkins start
```

当然我还是习惯了比较原始的方式，不清楚区别是什么，有知道的请不吝赐教。

```
$ sudo /etc/init.d/jenkins start
```

## 登录Jenkins

如果没有特别配置端口，使用`http://<ip address>:8080/`登录Jenkins，并进行相关配置（插件安装、权限配置、View/Job创建等等）。这里一般我们还要做个二级域名解析，因为IP不是很好记。

另外，Jenkins默认是不开启权限控制的，什么人都可以用，我们还是需要开启权限控制的，权限控制有很多选项，这里不一一赘述了。

## 参考文献：

* [CentOS上Jenkins安装记录](http://www.cnblogs.com/zhangqingsh/archive/2013/03/19/2968998.html)
