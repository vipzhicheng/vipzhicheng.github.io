---
layout: post
title: "解决Jenkins长时间运行后样式丢失"
date: 2014-04-08 12:03:54 +0800
comments: true
categories:
- Ops
tags:
- Jenkins
- CentOS
- 样式错乱
- 图片丢失

---

这个问题具有一定的普遍性，在百度和Google都能搜到，本文以CentOS 6.3为例进行记录。Jenkins是通过yum方式安装。

原因是系统自动清理/tmp目录（参考：/etc/cron.daily/tmpwatch），但Jenkins默认把图片和样式资源解压缩到/tmp下。为了解决这个问题，我们需要重新设置Jenkins的tmp目录。

<!-- more -->

```
cd /var/lib/jenkins/
mkdir tmp
chown jenkins:jenkins tmp

vim /etc/sysconfig/jenkins
```

找到JENKINS_JAVA_OPTIONS，在之前的基础上，增加tmpdir部分，笔者在默认安装的基础上，修改完之后如下所示：

```
JENKINS_JAVA_OPTIONS="-Djava.awt.headless=true -Djava.io.tmpdir=/var/lib/jenkins/tmp"
```

之后重启Jenkins即可

```
/etc/init.d/jenkins restart
```
