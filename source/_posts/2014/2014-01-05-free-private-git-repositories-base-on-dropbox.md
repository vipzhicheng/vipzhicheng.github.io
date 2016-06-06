---
layout: post
title: "基于Dropbox的免费Git项目托管"
date: 2014-01-05 21:21:44 +0800
comments: true
categories:
- Ops
tags:
- Git
- Dropbox

---

你是否有私有Git项目需要托管却又不想把你买午饭的钱用来购买Github的付费账户？Git与Dropbox整合使用可以免费的实现这一目的，从而你的代码库可以同步到Dropbox并且在多台电脑上同步，通过这种方式你可以托管任意数量的版本库，并且是云备份哦！～。Dropbox的免费账户就有2G的空间，如果只用来托管代码是完全足够的。

<!-- more -->

下面就来一步一步的实现这个功能。

创建一个普通的本地Git版本库
------------------------

```
$ cd yourproject
$ git init
$ git add *
$ git commit
```

创建一个git目录来存放你的版本库
---------------------------

在你的Dropbox目录创建一个目录用于存放Git版本库，你当然可以将整个Dropbox目录都用于托管，不过大多数人应该是用Dropbox备份很多私人数据的吧。

```
$ cd ~/Dropbox
$ mkdir git
```

创建一个bare版本库
----------------

现在为你的本地工作拷贝创建一个空的版本库作为Git远程版本库，除了这是在本地操作以外，这本质上和用Github或者其他版本库托管网站的工作方式类似。

```
$ cd git
$ mkdir yourproject.git
$ cd yourproject.git
$ git --bare init
```

添加remote origin并且push到“远程”Dropbox版本库
-------------------------------------------

每次同步都会被Dropbox进程自动检测到改变并自动同步到Dropbox服务器

```
$ cd yourproject
$ git remote add origin ~/Dropbox/git/yourproject.git
$ git push origin master
```

注意事项
-------
1. 这个方案只适用于单用户的私人项目，多个人一起使用这个方案会有冲突。
2. 其实更好的方案是使用BitBucket这样的私有项目托管方案，但我更倾向于双保险，尤其是当BitBucket不定时的出现问题的时候。
3. 此方案在Mac下测试通过，理论上Linux和Windows也应该是类似的方法。
4. 类似于Dropbox的服务提供商应该都可以用来做这种备份，但是要注意各家提供商的技术实力，是否会有丢失数据的情况，Dropbox的技术实力应该是比较强的。
5. 为什么不直接将Git的代码库直接放到Dropbox目录呢，原因是这样一边开发一边同步很可能会产生冲突，而本文描述的方式则不会产生这种冲突。

**译者手语**本文是翻译之作，并融入了译者的个人理解，若有翻译的不好或者不对之处，还请同行朋友指点，如需转载请注明出处。
原文出处：《[Forget github limits: Free private git repositories with Dropbox](http://mrdanadams.com/2011/github-free-private-git-repositories-dropbox)》
