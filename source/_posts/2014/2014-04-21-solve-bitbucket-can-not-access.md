---
layout: post
title: "解决Bitbucket无法访问和push"
date: 2014-04-21 00:41:44 +0800
comments: true
categories:
- Ops
tags:
- Git
- Github
- Bitbucket
- 代理

---

近日的网络不知道又出了什么状况，各种被墙，尤其是Bitbucket不能使用，严重影响了工作的效率和心情，有时候是网页打不开，有时候是不能pull或者push，通过搜索找到了解决方法，记录如下。

<!-- more -->

这里，我们需要有一种翻墙措施，你可以是自己的VPN， ssh翻墙， goagent，或者其他，笔者是Mac系统，翻墙暂时使用的是鱼摆摆（不知道是啥东西的可以去搜搜），这个VPN开启的端口是9743, 如果用goagent，端口可能是8087，这个端口号接下来会用到。

切换到git项目目录，这里我们使用的是bitbucket的https项目路径，假设之前是可以正常工作的，为了可以让push也翻墙，我们需要做如下设置：（再一次说明，以下不是完整设置，仅仅是为了解决本文标题所说的状况）。

```
git config --add http.proxy 127.0.0.1:8087
git config --add https.proxy 127.0.0.1:8087
git config --add https.sslVerify false
git config --global http.postBuffer 524288000   #非常重要，否则可能提交不成功。
```

之后我们就可以push了，当然以上设置也可以直接在.git/config里修改设置。
