---
layout: post
title: "向大家推荐一个Shell脚本：Go"
date: 2014-03-10 23:15:25 +0800
comments: true
categories:
- Ops
tags:
- Shell
- SSH
- 多个账户

---

这个脚本是做什么用的呢， 主要是用来管理大量SSH远程主机的，以前我们的做法要么就是把远程服务器的账户名，密码存成文本文档，每次连服务器都需要去查文档。要么就是编写一个个expect脚本，go-here, go-there的。

这里给大家推荐的go命令，可以一个命令管理多个远程，而且配置简单

<!-- more -->

项目地址：https://github.com/vipzhicheng/go

详细的使用方法可以参考项目的简介，这里列出调用go命令的部分，大家可以看到， go命令后面可以跟ip或者label参数，可以随意匹配配置文件里的服务器，如果匹配的服务器只有一个，直接就可以登上去了，非常的方便。

```
$ go label

Found follow servers: (Which one do you want to connect?)
[1] user1@192.168.1.7 label:7
[2] user2@192.168.1.8 label:8
Please choose by ID:
1

Logging into user1@192.168.1.7 ...
spawn ssh user1@192.168.1.7 -p 22000
user1@192.168.1.7's password:
Last login: Mon Mar 10 18:35:02 2014 from 192.168.1.6
$

```

如果你有任何建议，可以留言，我会继续完善，不过目前来讲已经够我使用了。
