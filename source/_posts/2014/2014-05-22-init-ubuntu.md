---
layout: post
title: "初始化Ubuntu"
date: 2014-05-22 14:22:17 +0800
comments: true
categories:
- Ops
tags:
- Ubuntu

---

Ubuntu桌面版应该说已经做的不错了，总有一些需要定制的地方，本文以Ubuntu 14.04为例介绍安装Ubuntu之后要做的设置工作。

<!-- more -->

## 为终端设置等宽字体

默认的monospace字体貌似不是很好用，我一般给改成UbuntuMono 13号字粗体。

## 关闭独立显卡

Ubuntu安装完之后，风扇狂转，需要关闭独立显卡，可能只是N卡才有这个问题

```
sudo apt-get install bumblebee
```

安装完，重启即可，无需设置。
