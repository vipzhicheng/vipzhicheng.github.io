---
layout: post
title: "Github密码缓存"
date: 2014-01-15 02:40:05 +0800
comments: true
categories:
- Dev
tags:
- Git
- Github
- Password cache
- Octopress
- Octopress deploy

---

本文的知识点是关于Git和Github的，但我遇到这个问题是在使用Octopress写博客的过程中，每次写好一个要提交给Github时，总是问我要用户名和密码，所以我就想研究一下，是否可以记住账号密码，还别说真让我找到了，而且是Github网站帮助系统中的一小节，我打算翻译一下，毕竟是新学会了一个知识点。

<!-- more -->

PS：本文的知识点要求你的Git版本超过1.7.10，系统是Mac。

为了实现这个目的，你需要一个叫做osxkeychain credential helper的东东，不过有可能你已经安装了，如果你是使用的homebrew方式安装的git，或者你的git是最新版本。

可以通过以下方式检查：

```
git credential-osxkeychain
# Test for the cred helper
# Usage: git credential-osxkeychain <get|store|erase>
```

如果你没有安装这个helper，你需要手动安装一下：

```
git credential-osxkeychain
# Test for the cred helper
# git: 'credential-osxkeychain' is not a git command. See 'git --help'.

curl -s -O \
  https://github-media-downloads.s3.amazonaws.com/osx/git-credential-osxkeychain
# Download the helper

$ chmod u+x git-credential-osxkeychain
# Fix the permissions on the file so it can be run
```

然后需要把这个脚本放到Git目录下：
```
sudo mv git-credential-osxkeychain \
  "$(dirname $(which git))/git-credential-osxkeychain"
# Move the helper to the path where git is installed
# Password: [enter your password]
```

接着，需要告诉Git我们要用这个helper, 简单的配置一下全局Git配置就行了
```
git config --global credential.helper osxkeychain
# Set git to use the osxkeychain credential helper
```

这样下次你提交代码时，会提示输入你账户和密码，并且Mac会弹窗问你是否授权使用钥匙链，授权后，以后提交代码就直接使用钥匙链里记录的密码，不会再提示输入了。

这样我写完一篇博客直接提交就可以了，So easy!

```
rake generate && rake deploy
```
