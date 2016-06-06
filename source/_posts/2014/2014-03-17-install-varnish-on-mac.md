---
layout: post
title: "MacOS安装Varnish"
date: 2014-03-17 03:25:14 +0800
comments: true
categories:
- Ops
tags:
- MacOS
- Varnish
- Homebrew

---

说到开发环境，已经不仅仅是开发本身所需要的软件集合了，也包括线上生产服务器所使用的种种服务器端软件，比如大多数时候我们本地只要有普通的LAMP环境即可开发，但有时候因为特殊需求，我们就需要往本地安装Memcache, APC, Solr, Varnish等等服务器软件或者组件， 本文要说的就是MacOS上安装Varnish的方法。

<!-- more -->

首先，你需要安装homebrew, 然后只需要执行一条命令就可以自动安装，包括其依赖的PCRE等。

```
brew install varnish
```

安装的LOG信息里最重要的就是：

```
# 用于讲Varnish加到系统自动启动
ln -sfv /usr/local/opt/varnish/*.plist ~/Library/LaunchAgents

# 用于立即启动服务
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.varnish.plist

# 用于立即关闭服务
launchctl load ~/Library/LaunchAgents/homebrew.mxcl.varnish.plist
```

Varnish的配置文件在：

```
/usr/local/etc/varnish/default.vcl
```

我们需要做一些配置，另外注意，我们要达到的目的是测试，我本地已经有许多网站测试环境了，用的自然是80端口，所以我偶尔才需要用到的Varnish，就分给他一个8080端口吧

plist我们把启动端口改成8080

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
  <key>Label</key>
  <string>homebrew.mxcl.varnish</string>
  <key>ProgramArguments</key>
  <array>
    <string>/usr/local/opt/varnish/sbin/varnishd</string>
    <string>-n</string>
    <string>/usr/local/var/varnish</string>
    <string>-f</string>
    <string>/usr/local/etc/varnish/default.vcl</string>
    <string>-s</string>
    <string>malloc,1G</string>
    <string>-T</string>
    <string>127.0.0.1:2000</string>
    <string>-a</string>
    <string>0.0.0.0:8080</string>
  </array>
  <key>KeepAlive</key>
  <true/>
  <key>RunAtLoad</key>
  <true/>
  <key>WorkingDirectory</key>
  <string>/usr/local</string>
  <key>StandardErrorPath</key>
  <string>/usr/local/var/varnish/varnish.log</string>
  <key>StandardOutPath</key>
  <string>/usr/local/var/varnish/varnish.log</string>
</dict>
</plist>
```

vcl 文件，我们把80端口当作后端
```
backend default {
    .host = "127.0.0.1";
    .port = "80";
}
```

这样启动以后，我们访问任意一个本地测试虚拟主机的8080端口，可以在firebug的网络头看到Varnish头，证明Varnish已经安装成功。

我们还可以使用Lingo这样的软件来管理服务的启动和停止，如果发现不好关闭的话，可以直接`sudo kill PID`的方式关闭服务。

最后说一下上面几个启动选项的含义

* -n: 运行目录路径
* -f: 配置文件路径
* -s: 缓存方式，这里是内存划分1G，也可以是file，代表使用文件系统缓存
* -T: 管理后端，使用telnet来管理
* -a: 启动端口
