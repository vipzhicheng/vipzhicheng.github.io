---
title: Git创建分支并推送到远程分支
date: 2016-06-02 18:20:56
categories:
- Git
tags:
- Git
- 分支
---

在为公司技术部选型博客系统的过程中，最终选择了Hexo+Git内网构建的方式，一方面是希望大家学会这一优秀的静态博客系统的使用方法，另一方面，也是希望大家逐渐熟悉Git，为下一步真正的项目代码迁移到Git做准备。

Git上手很简单，但其实很复杂，很多命令都并没有真正理解，就算学会了也很容易忘，这就需要不断实践，也需要时间来体会。

本文的重点主要还是学到的一点Git知识，做一个记录，感觉不记下来不就以后还会忘。

# 创建分支

``` bash
git checkout -b richardyu
```

# 添加远程版本库

``` bash
git remote add origin GIT_REPO_URL
```

# 推送到远程分支

``` bash
git push origin richardyu:richardyu
```

# 删除远程分支

``` bash
git push origin :richardyu
```

注意，删除远程分支不会删除本地分支，还可以再推回去。
