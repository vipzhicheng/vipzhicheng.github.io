---
layout: post
title: "给CentOS安装start-stop-daemon"
date: 2014-03-03 11:35:29 +0800
comments: true
categories:
- Ops
tags:
- CentOS
- start-stop-daemon

---

start-stop-daemon是一个Debian体系里的一个守护进程管理软件，可以用指定的用户启动系统软件。CentOS有自己的daemon()机制（在/etc/init.d/functions)。但是如果你手上有一个脚本，必须使用start-stop-daemon在CentOS上运行，改写太麻烦，还是选择安装start-stop-daemon来的方便。

<!-- more -->

安装start-stop-daemon需要去debian下载源码包, 我是在这里下的：http://ftp.de.debian.org/debian/pool/main/d/dpkg/，找到最新的dpkg包, 注：以下以dpkg_1.17.6为例。

然后按照以下步骤即可编译成功

```
wget http://ftp.de.debian.org/debian/pool/main/d/dpkg/dpkg_1.17.6.tar.xz
tar -xf dpkg_1.17.6.tar.xz
cd dpkg_1.17.6
./configure # 这一步如果提示缺少curses，需要yun install ncurses-devel
make # 这一步会报错，忽略即可
cd utils
make # 可能不需要执行这个
sudo cp start-stop-daemon /usr/local/bin/start-stop-daemon

```
