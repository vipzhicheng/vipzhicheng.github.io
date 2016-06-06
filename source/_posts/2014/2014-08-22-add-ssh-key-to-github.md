---
layout: post
title: "为Github添加SSH KEY"
date: 2014-08-22 16:19:43 +0800
comments: true
categories:
- Ops
tags:
- Github
- Bitbucket
- CodingNet

---

如果你使用Github，Bitbucket, 或者国产的Coding.net等，都会在个人设置那里看到设置SSH Key的地方，这个是做什么用的呢，这个是用来让你提交代码通过SSH隧道的方式提交到远程服务器，这种方式有安全，稳定以及无需每次输入认证信息等特点。

本文只是以Github为例，讲解怎么操作，至于原理和概念，请自行搜索。

<!-- more -->

## 第一步，检查一下当前已经生成的SSH keys

```
ls -al ~/.ssh
```

当然理论上当前存在的SSH keys不会影响你新创建的，注意为每个不同的key取不同的文件名。

## 第二步，创建一个新的SSH key

```
$ ssh-keygen -t rsa -C "your_email@example.com"
# Creates a new ssh key, using the provided email as a label
# Generating public/private rsa key pair.
# Enter file in which to save the key (/Users/you/.ssh/id_rsa): [这里建议起个特殊的名字]
```

```
Enter passphrase (empty for no passphrase): [Type a passphrase]
# Enter same passphrase again: [Type passphrase again]
```

passphrase类似于密码，但只是本机使用，可以不设置，但这样不够安全。

你会得到类似于下面的输出：

```
Your identification has been saved in /Users/you/.ssh/id_rsa.
# Your public key has been saved in /Users/you/.ssh/id_rsa.pub.
# The key fingerprint is:
# 01:0f:f4:3b:ca:85:d6:17:a1:7d:f0:68:9d:f0:a2:db your_email@example.com
```

然后，使用如下方式加载本地私钥，如果本机已经开启了ssh-agent就不需要再启动直接添加私钥即可。

```
# start the ssh-agent in the background
eval "$(ssh-agent -s)"
# Agent pid 59566
ssh-add ~/.ssh/id_rsa
```

## 第三步，在Github上添加SSH key

注意两点：

1. Label只是用来助记的标识。
2. 里面填写的是公钥。

最后，你只需要在你的本地代码添加对应远程SSH版本库地址即可。

```
cd existing_git_repo //进入已有项目或更新的项目目录
git remote add origin SSH_REPO_URL
git push -u origin master //这两行将该目录下的文件推送到远端（origin）上的 "master" 分支
```
