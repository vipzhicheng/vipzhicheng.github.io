---
layout: post
title: "[译]使用RVM安装Ruby on Rails"
date: 2014-04-08 17:25:43 +0800
comments: true
categories:
- Ops
tags:
- Ruby on rails
- RVM
- CentOS

---

本文译自[How To Install Ruby on Rails on CentOS 6 with RVM](https://www.digitalocean.com/community/articles/how-to-install-ruby-on-rails-on-centos-6-with-rvm),转载请著名原文出处及译文出处。

## 关于 Ruby on Rails

Ruby on Rails，简称RoR，是一个应用程序集合，提供给开发者一个快速创建Web应用程序的开发框架。

安装Ruby on Rails环境确实需要一定的时间，不过幸运的是有许多工具可以让安装RoR的流程尽可能的简单。本文基于CentOS6，根据实际安装时的经验对原文步骤略有调整，虽然RVM支持在任意账户下安装，但这里还是以root账户为例，会为以后减少许多不必要的麻烦。
<!-- more -->

## 第一步，安装RVM

在我们做任何其他步骤之前，首先要从源做一次更新以确保我们下载的所有包(Package)都是最新的。

```
yum update
```

一旦上面的更新结束，我们就可以开始安装RVM（Ruby Version Manager）了，这个程序可以让你在一台服务器上使用多个版本的Ruby。

如果你的系统里还没有安装curl，请从安装curl开始。

```
yum install curl
```

安装RVM只需要在终端执行以下指令。

```
curl -sSL https://get.rvm.io | bash -s stable
```

安装后，我们需要加载RVM

```
source .bashrc
source .bash_profile
```

为了保证RVM正常工作，RVM自己有一些软件依赖需要安装，可以通过以下指令查看这些依赖。

```
rvm requirements
```

以上指令将自动安装依赖，一般需要根据实际情况安装额外的依赖包，比如笔者在一台服务器非常顺利的执行了这个指令，而在另外一个服务器上则需要安装一个依赖包：

```
yum install -y libyaml-devel
```

以下是参考的反馈信息：

```
Checking requirements for centos.
Installing requirements for centos.
Updating system-

.
Installing required packages: patch, libyaml-devel, libffi-devel, autoconf, gcc-c++, patch, readline-devel, zlib-devel, openssl-devel, automake, libtool, bison............. ...............
Requirements installation successful.

```

## 第二步，通过RVM安装Ruby

一旦我们安装好了RVM，安装Ruby是非常容易的。这里要等待比较长的时间。

```
rvm install 1.9.3
```

因为RVM支持多个Ruby版本，所以我们需要告诉系统我们默认使用哪个版本的Ruby。

```
rvm use 1.9.3 --default
```

## 第三步，安装RubyGems

为了让RoR程序正常工作，我们还需要安装许多Ruby组件，这些都需要靠RubyGems，和前面一样，RubyGems我们也通过RVM来安装。

PS: 实际测试中，安装了Ruby 1.9.3后，就已经安装了gem，不需要执行下面的指令。

```
rvm rubygems current
```

## 第四步，安装Rails

最后就是安装Rails了，只需要执行一行指令，但是可能要等上一段时间。如果你安装Rails的目的是安装Redmine，就不用执行下面的指令了，因为Redmine的安装流程会自动安装合适版本的Rails。

```
gem install rails
```
