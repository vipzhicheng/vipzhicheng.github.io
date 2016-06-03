---
layout: post
title: "Drupal A/B Test解决方案之一"
date: 2014-01-05 22:32:14 +0800
comments: true
categories:
- Dev
tags:
- Drupal
- A/B Test

---

本文是我在工作中遇到A/B Test的需求，经过几天的研究的一篇心得笔记，不一定多专业，希望能给有类似需求的同行一些参考和灵感，在学习的过程中，也发现一些比较优秀的国外博文，我打算在近期翻译几篇，而不是把只言片语融入到本文当中，本文的重点仍然是Drupal A/B Test的解决方案，介绍性的文字来源于最近几天的学习心得，是给不熟悉A/B Test的同行的一些入门介绍。

<!--more-->

PS1：所说的A/B Test，我想正确的说法应该是A/B Testing，但国人可能更习惯说A/B Test，本文也不是咬文嚼字，所以全篇也都叫A/B Test了。

PS2：本文以Drupal 6环境为依托来介绍，Drupal 7也许会有更好的解决方案。

## A/B Test 简介

这一部分主要是简单介绍一下A/B Test的概念和相关知识。从而让读者对A/B Test有一个大概的了解。

### 什么是A/B Test？

A/B Test，是指对某一个特定的页面或者页面上的一部分，制作两个或两个以上版本，然后将网站的访问流量分发到不同的版本上，继而观察网站该页面或者该部分带来的转化率，或者一些重要指标的变化，从而得出其中的一个版本比另一个版本更好的结论，并以此决定最终上线的版本。

最开始的时候，我觉得主要是市场人员希望知道改变了字体大小，颜色，文案之后，网站的转化率究竟发生了怎样的变化，但现在我们已经把A/B Test的内容进行了扩展和深化，可以应用于网站的方方面面，前端后端，是持续运营网站的一个有效工具。前端很好理解，后端的A/B Test举个例子来说，比如为了给用户推荐一个产品列表，我们设计了几个不同的算法，推荐结果是不同的，但从功能上又是相同的，那么怎么决定最终用哪个算法呢，这时候就可以针对算法做一次A/B Test。

### 什么是Bucket Test

Bucket Test 是A/B Test概念的扩展，中文翻译成分桶测试，外国人也有叫Split Test的， 其实都是一个意思，就是同时做多个A/B Test， 不是只有一个B版本，而是B1, B2 ... BN 一共N个版本，做一次Bucket Test 就可以在多个改进版本中选出一个最优的版本。 实现Bucket Test的系统，就是Bucket Test System, 简称BTS， 一般来说只有有这样的系统才可以做好Bucket Testing， 因为他可以帮助我们很方便的管理每组测试，为每个Bucket分配流量，实时监控每个Bucket的运行效果，转化率等等各项指标。BTS根据各个公司的开发实力以及需求，可以做的很复杂，尤其是对自身业务关键指标的抽象，计算。如果是小公司，可以直接使用第三方系统做流量切割，然后使用半手动的方式评估每个版本的效果。 但是在实际测试中，一定要注意每个bucket要有足够多的测试样本，流量不平均分配也没事，但数量要够大，这样你才能对测试结果有信心，测试才有意义。

### 多因素测试

多因素测试是一种Bucket Test, 不是所有的Bucket Test都这样做。 比如在一次改版任务中， 我们认为提升转化率的关键因素有3个，标题字体大小有3个版本，标题颜色有2个版本，标题文案有2个版本，那么就有3x2x2一共12种组合， 所以我们就相当于有了12个Bucket， 这样我们就可以进行Bucket Test了， 最后我们得到一个结论，这3个关键因素的某个组合转化率最好，那么这个版本就是我们最应该采用的版本，拥有最优化的关键因素组合。 大多数时候，我们其实每次改版改动都是很大的，不去区分究竟哪些地方改变了，或者我们改动了很多地方，这样做多因素测试就是不现实的，这时往往我们就弄一两个自己认为关键的版本（往往就一个）就可以了，这样得到的结果未必是此次改版所能达到的最好效果，只能看这个版本和原版本的比较结果，但即使这样也是有意义的，比直接上线新版一段时间，然后再观察各项关键指标要客观。

### A/B Test的原则

1.  只测试少量因素
2.  因素改版要有显著变化
3.  针对大流量页面做测试
4.  持续测试，仅仅做一两次A/B Test是达不到提高转化率的目的的
5.  各版本页面传达的信息应该是一致的只是形式稍有不同
6.  应该在同一时间测试
7.  各版本得到的流量都要比较大，这样结果才有说服力
8.  不要过早的停止A/B Test
9.  要让访问测试版本的访客始终看到那个版本

## Drupal系统的A/B Test解决方案

BTS系统有很多，这里主要推荐Google Analytics Content Experiments工具，关于怎样使用，这里有一个比较好的中文介绍，[利用Google Analytics测试优化网站内容](http://www.analyticskey.com/content-experiment-google-analytics/) 我们是Drupal程序员，那么我们在做A/B Test的时候，需要做些什么以及如何做呢，本文给出的是一个完全原创的代码级解决方案，需要使用GA Content Experiments做流量分发和效果监控，本文主要是告诉大家在这套方案里我们的代码应该怎样写。

### 将GA A/B Test相关的JS代码嵌入head标签

首先，GA A/B Test工具中配置一个A/B Test实验时，会让你在原始版本添加一些GA统计代码到head标签，Drupal自身对这件事情支持的不是特别好，所以我的方案是在page.tpl.php里加入新的自定义变量到head标签，然后在template.php里加逻辑设置这个变量，最后在settings.php里对GA统计代码随时添加删除，这样做是因为A/B Test一般都是临时的,所以某些时候我们需要快速开启和关闭A/B Test，没有必要每次都走release流程。 编辑page.tpl.php，因为GA 的AB Testing工具要求统计代码放到head标签最开始的位置。（这里是因为不加空格就显示不出来）

```
<head>
<?php print $head_prepend;?>`
```

编辑template.php， 我们在主题的template.php里为新加的模板变量复制，注意值是在settings.local.php里设置的，其实这里如果做的复杂一些，我们可以为模板变量的复制单独开发一个管理后台，但简单就是美，这里还是以突出核心概念为主。

```
function MYTHEME_preprocess_page(&$vars) {
  $vars['head_prepend'] = zinchus_process_page_head_prepend();
}
```

自定义函数,放到了template.php，因为不是API

```
function MYTHEME_get_page_head_prepend() {
  $request_uri = request_uri();

  $head_prepend = variable_get('head_prepend', array());
  if (array_key_exists($request_uri, $head_prepend)) {
    return $head_prepend[$request_uri];
  }
  return '';
}
```

在settings.php里设置统计代码，注意里面的key是request_uri，也就是实际路径，不是Drupal Path

```
if (empty($_GET['v'])) {
  $conf['head_prepend']['/node/374'] = [GA CODE];
}
```

### 为原始代码实现多个版本

前面的逻辑都写好以后，我们就可以很方便的在任意页面插入统计JS代码了，但同时GA Content Experiments还需要多个版本，这里以一个版本为例，我们是可以随意开发一个新的版本用一个新的url，但要注意，这只是一个实验，在得出结果以后，好的版本会留下，不好的版本会被淘汰，这时如果我们让网站同时存在两个版本，时间长了会越来越难维护，有很多冗余代码在里面，为了解决这个问题，我写了一个简单的入口函数abtest()。也就是说我给出的是一个函数级别A/B Test的方案，可以给任意函数添加新版本。 下面来简单看一下这个函数的用法

```
// 注意第二个参数可以传多个同功能新版函数
$output = abtest('func_a', array('func_b'));
```

当测试结束是，我们一般会随时希望将效果好的版本正式上线，这时我们会选择使用settings.php。如果想更加易用一般会为这个setting开发后台用于开关和版本选择，但settings.php一般来说是我们更加常用的settings设置手段，因为这种settings照顾到了可配置性，而且一般配置好了也不经常修改。

```
// 如果设置成0就是强制启用原始版本，如果是2就是强制使用新版本2（如果有的话）
$conf['abtest_force_func_a', 1);
```

入口函数源码

```
// 这个是用于分发各个版本的入口函数，很简单不是么
// 第一个参数是原始函数，第二个参数是多个新版函数名组成的数组，第三个参数是传给函数的参数。
function abtest($test_function, $buckets = array(), $args = array()) {
  !is_array($buckets) && $buckets = (array) $buckets;
  $bucket = variable_get('abtest_force_' . $test_function, NULL);

  if (!isset($bucket)) {
    if (isset($_GET['bucket_name'])) {
      $bucket_name = check_plain($_GET['bucket_name']);
    } else {
      $bucket_name = variable_get('abtest_bucket_name', 'v');
    }
    if (isset($_GET[$bucket_name])) {
      $bucket = check_plain($_GET[$bucket_name]);
    } else {
      $bucket = 0;
    }
  }
}
```

这里需要注意的是即使是Drupal页面级的A/B Test也可以从这个方案中获益

```
// 示例代码
$item['page callback'] = 'abtest';
$item['page arguments'] = array('func_a', array('func_b'));
```

最后，为了清理代码方便，原则上建议把新版本的代码规划好，比如每个新版本都放到不同的文件夹中，杂揉在一起在将来删除时会浪费不少时间。

## 小结

A/B Test是一种数据驱动运营型网站的常用运营工具，因为我们需要不断优化，改版增加网站的用户体验，提高转化率，我们不仅要改，还要知道修改的部分对网站转化率的影响，还要知道影响的程度，这些数据有的可以通过BTS给出来，但有的需要对现有数据库进行数据分析，为此还可能需要在各个版本里嵌入自己的关键数据收集代码。总之一切都是为了做出更好的网站和提高我们的收入。

## 补充内容

在Drupal项目中实施A/B Test的过程中，我们遇到了很多非典型问题，下面补充一些相关内容

### B版本只有匿名用户能看到，而A版本登录和匿名用户都能看到

这个解决起来很简单，只要登录用户不加载GA的实验代码，就不会进入统计了。

### GA的A/B Test流量分流不是平均分配

这个大家在使用过程中一定会遇到，因为这是GA故意为之的，目的也很好理解，A/B两个版本的线上测试，如果B版本比A版本好太多，意味着无论是Page views，还是实际销售额肯定相差很大，如果平均分配流量，是会损害网站的核心利益的，所以GA从算法上，使得根据转化率自动调整流量分配，转化率高的，得到的流量就多。 但有的时候，我们是需要流量平均分配的，这个时候，trick的解决方案是设置一个永远不可达到的Goal页面。经测试这种方式是十分有效的。 有兴趣的可以继续阅读[这篇GA文档](http://support.google.com/analytics/bin/answer.py?hl=en&amp;answer=2844870 "Multi-armed bandit experiments")。

### A/B 两个版本的Goal页面不是同一个

我们知道GA的A/B Test实验只能设置一个，这种两个Goal页面的事情，其实是在打破A/B Test的规则，这也数据测试的内容太多的问题，我们这是两个产品（页面同时上线，同时测），然后从总的转化上把握效果，不管是否对错，但总是要先实施出来，这里的实施方法是，在GA里只设置A版本的Goal页面，然后看B版本有多少去了A版本的Goal，算出比例，然后把A版本带来的转化量去掉这个比例后再进行评估。

## 参考文献和资源

1，http://www.analyticskey.com/content-experiment-google-analytics/ 详细介绍了Google Analytics Content Experiments的用法。我们的A/B Test方案中，流量分发和部分的数据监控都是交给GA的，我觉得这样没什么不好，可以让我们更专心专注于业务逻辑即新版本的开发。

2，http://drupal.org/project/multivariate Drupal Way的A/B Test模块，即提供了BTS的管理后台，又可以监控，以及A/B Test实施，我觉得做目前还用不上这个模块，或者说这个模块不能解决我们网站遇到的问题。

3, http://phpabtest.com PHP级别的A/B Test解决方案，也是与GA继承，但似乎其并没有用到Content Experiments功能，而是简单的往GA发送数据的方式。不过还是值得借鉴的。

4，http://whichtestwon.com/ 一个投票网站，投A/B 两个版本哪个你觉得更好，这个就没什么转化的概念的，所以虽然也像是一个A/B Test，但我觉得更像是一个游戏。

5, http://visualwebsiteoptimizer.com 一个BTS系统，据说比GA提供的工具更强大，其实还有很多类似的第三方系统，不一一列举，我们目前是还没有到使用这样的系统频繁做大量A/B Test的程度。

6, http://www.smashingmagazine.com/2010/06/24/the-ultimate-guide-to-a-b-testing/ 这篇文章的主要看点是看看各个网站是怎么做的A/B Test，又因为A/B Test提高了多少转化率，可谓是A/B Test里的经典案例了。
