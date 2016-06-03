---
layout: post
title: "date命令获得上月的最后一天"
date: 2014-03-06 12:06:10 +0800
comments: true
categories:
- Ops
tags:
- 脚本
- Shell
- 日期

---

工作中需要编写一些自动化脚本，有的涉及到定期执行的问题，我需要在Shell中得到上个月的第一天和最后一天，去网上查了一下，有的人给出的答案需要写一大段程序，有的需要使用其他脚本语言来实现，都不是很简练，还有的使用cal命令加awk, xargs等命令复合使用来实现。

后来终于让我找到了合适的方法，就是使用系统自带的date指令+参数。
<!-- more -->
```
start_date=`date +"%Y-%m-01" --date="last month"`
end_date=`date -d "$(date +'%Y-%m-01') - 1 day" +"%Y-%m-%d"`
```

其他实现方式就不在这里记录了，网上很容易找到。
