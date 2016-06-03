---
layout: post
title: "使用PEAR为MAMP环境安装Drush"
date: 2014-01-03 00:17:07 +0800
comments: true
categories:
- Dev
tags:
- Drupal
- Drush
- 命令行工具
- PEAR

---

Drush和其他的Unix/Linux软件一样，有多种安装方式，我们可以在[Drush.ORG][]上，或者[Drupal Github项目][]上看到更多关于安装方式的介绍。本文只是想谈谈Mac OSX系统里MAMP环境下的基于PEAR的Drush安装方法，为和我使用相同开发环境的朋友提供一个参考。

[Drush.ORG]: http://drush.org
[Drupal Github项目]: https://github.com/drush-ops/drush

<!-- more -->

## 为什么使用PEAR的安装方式

不得不说，其实Drush一路走来发展的还是挺快的，我最开始是使用源码下载的安装方式，后来使用的是苹果的第三方包管理器Homebrew的安装方式，但最近才发现，最方便的安装方式还是PEAR的安装方式。

## 配置环境变量

首先要确保环境变量设置正确，Mac系统可能会自带PHP以及PEAR，通过调整`$PATH`变量，可以让我们在命令行里直接使用MAMP带的PEAR程序，而不是系统的那一个。

```
vim ~/.bash_profile
export PATH=/Applications/MAMP/Library/bin/:/Applications/MAMP/bin/php/php5.2.17/bin:$PATH
```
注意，由于我的开发环境常用的是PHP5.2所以bin的路径是基于PHP5.2的，如果是使用PHP5.3及以上环境的朋友，需要修改一下这里。

## 安装PEAR

```
pear channel-discover pear.drush.org
pear install drush/drush
```

如果需要安装某一个版本，或者切换drush版本，可以指定版本号，例如：`pear install drush/drush-4.5.0`，注意后面的.0是必须的，因为pear.drush.org发布的压缩包就是这样命名的。

## 使drush命令可执行

```
chmod u+x /path/to/drush/drush
```
安装说明里一般还需要建立符号链接，但是记得前面设置的环境变量么？有了这个就不需要设置符号连接也可以直接在命令行自动提示和执行了。

## 解决Drush找不到Console_Table库

这是在一切就绪后执行drush时遇到的问题，这个问题很简单，只需要按照说明安装对应的依赖即可。
但是实际上drush支持自动解决这个问题，只需要在第一次执行drush命令时，使用sudo即可，例如：

```
sudo drush st
```

好了，现在你的drush命令就彻底安装完成了，和一般的安装方式相比，少了下载解压的过程，不需要建立符号链接。drush安装路径更规范，我觉得这是在Mac系统MAMP环境安装drush的最佳实践。
