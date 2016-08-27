---
title: Mac基于socks5的http代理的设置方法
date: 2016-08-27 17:01:40
tags:
- 代理
- HTTP代理
- Privoxy
categories:
- Ops
---

现在主流的代理方式是基于shadowsocks的socks5方式，这是一种局部代理方法，浏览器可以通过插件使用这个代理，但不是任意软件都可以使用的，很多软件不支持socks5方式，一般会支持http代理。

除了直接架设http代理，我们还有一种选择，就是将流量从http代理端口转发到socks5代理。Mac下，能够实现这个事情的方法有很多，比如polipo，proxychains，不过配置起来比较复杂，这里给大家介绍的是一个配置起来比较简单的方法。

注意，这种方法需要依赖于本机有个能够正常工作的socks5代理。

## 依赖

* Homebrew

## 安装

```
brew install privoxy
```

## 配置

```
vim /usr/local/etc/privoxy/config
```

只需要配置第一个地方

```
forward-socks5   /               127.0.0.1:1080 .
```

如果需要修改端口，可以改如下配置，否则默认是8118

```
listen-address  127.0.0.1:8118
```

## 启动

```
# 加入开机自启动服务
brew services start privoxy

# 不加入开机自启动服务
privoxy /usr/local/etc/privoxy/config
```

## 对开发人员来说的好处

一些常用的软件有时需要代理才能够正常下载组件，比如`sublime text`，`atom`, `npm`, `git`, `bower`等等。以目前天朝的网络环境，掌握一定的代理技巧是十分有必要的。