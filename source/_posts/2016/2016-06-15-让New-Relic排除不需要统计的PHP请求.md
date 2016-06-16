---
title: 让New Relic排除不需要统计的PHP请求
date: 2016-06-15 17:36:59
categories:
- Dev
tags:
- New Relic
- 性能优化
- 监控
---

我们使用`New Relic`作为性能监控分析工具，关于`New Relic`就不过多介绍了，大家可以自己去搜搜。这种性能分析工具对于大型网站来说是必不可少的，而其特色不仅在于可以监控服务器硬件指标，而且可以监控应用级别的各种数据，当网站发生性能问题时，可以在上面找到一些线索，甚至是直接找到答案。目前国内也有同类产品了，比如说听云，目前还没有机会尝试。

我们遇到的问题是，我们希望`New Relic`监控我们的部分核心应用的性能指标，但服务器上部署了一些非核心应用，这些应用又存在着访问量大，但是逻辑简单，所以响应迅速的问题，所以从监控上看，这个服务器的平均响应时间非常的低，而其他只部署核心应用的服务器的平均响应时间是很高的，这就给人了一个假象，从而看不出这台服务器核心应用的真实水准。

那么怎么排除掉非核心应用呢？经过调查，发现`New Relic`支持一些[API](https://docs.newrelic.com/docs/agents/php-agent/configuration/php-agent-api)，可以对上传的数据做定制，里面恰好就有解决这个问题的API。

``` php
if (extension_loaded('newrelic')) {
    newrelic_ignore_transaction();
}
```
这个API函数的描述是这样的：

> Do not generate metrics for this transaction. This is useful when you have transactions that are particularly slow for known reasons and you do not want them always being reported as the transaction trace or skewing your site averages.

将这段代码加到非核心应用的入口文件里的，效果是立竿见影。

{% asset_img QQ20160615-1@2x.png "New Relic 监控截图" %}

如果进一步思考，我的核心应用里可能有一些已知性能差，但是不需要理会的程序（比如计划任务），我也是不希望在里面混淆视听的，这时，我们就需要把这段代码放到我们的核心应用里，根据路径或者其他判定条件忽略对本次请求的监控。

监控的意义在于，发现突然出现的波峰波谷，如果引起动荡的原因是你肯定不关心的，就可以忽略掉，不上传数据给`New Relic`。

更多`New Relic` API大家就自己去探索吧，当我有使用场景时，也会记录下来，跟大家分享。
