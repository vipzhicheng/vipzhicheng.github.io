---
layout: post
title: "CentOS用yum安装SVN"
date: 2014-04-03 22:00:03 +0800
comments: true
categories:
- Ops
tags:
- SVN
- Subversion
- CentOS

---

源代码管理目前的市场上Git主键主流，SVN也仍然有大把的人用，其他也有一些新的或者小众的版本管理软件。对我来说，我选择SVN和Git，一个集中式管理，一个分布式管理，虽然Git强大到可以替代SVN，但有时也没有必要，对我来说部署一个简单的集中式版本控制系统，还是SVN要简单一些。

本文以CentOS 6.3为例阐述了如何用yum的方式安装和部署SVN。

<!-- more -->

以下命令假设都是root用户来执行，我就可以不必每次都加sudo了。

SVN一般有两种部署方式一种使用svn服务器模式，一种使用http做代理，其显示效果如下：

svn://yourdomain/repo
http://code.yourdomain/repo

因为第一种部署最简单，所以先说第一种的部署方式。

## svn服务器部署方式 1：svn://yourdomain/repo

首先检查已安装的版本并卸载，然后从源安装最新版SVN。

```
rpm -qa subversion
yum remove subversion
yum install -y subversion
```

假设我们的版本库在服务器上都是在`/opt/svn`目录下分子目录存放，而我们第一个示例版本库名是repo。

```
mkdir -p /opt/svn/repo
svnadmin create /opt/svn/repo
```

为了SVN能够生效，我们需要编辑版本库的配置，首先是svnserve.conf

```
vim /opt/svn/repo/conf/svnserve.conf
```

```
[general]
#匿名访问的权限，可以是read,write,none,默认为read
anon-access=none
#使授权用户有写权限
auth-access=write
#密码数据库的路径
password-db=/opt/svn/passwd
#访问控制文件
authz-db=authz
#认证命名空间，subversion会在认证提示里显示，并且作为凭证缓存的关键字
realm=My Test Repo
```

password配置文件放在`/opt/svn/passwd`而不是`/opt/svn/repo/conf/passwd`的目的是让多个版本库共享同一份账户文件。

配置密码只需要编辑对应的密码passwd文件,一看就明白就不演示了。

```
vim /opt/svn/passwd
```

然后为了有权限访问，还需要编辑授权文件

```
vim /opt/svn/repo/conf/authz
```

在这个文件里，你可以针对不同的人授予不同的访问权限,这里授权给用户所有路径的权限。

```
[/]
testuser=rw
```

另外有的教程还会提一下如果设置iptables和关闭selinux，这里就默认大家这两个都已经弄好了。

启动SVN

一般的启动方式是直接使用svnserve -d -r DIR的形式，但在CentOS6，我发现有守护进程的方式，/etc/init.d/svnserve，这样就更简单和稳定。

创建svnserve.conf

```
vim /etc/sysconfig/svnserve
```

只需要一行代码

```
OPTIONS='-d -r /opt/svn'
```

当然这里你可以指定监听端口，和监听IP地址等其他额外参数，不过最常用的还是-d -r。

这样一来，我们就可以使用svn://yourdomain/repo的方式checkout代码了。

## svn服务器部署方式2：http://code.yourdomain/repo

配置http方式，还需要从源安装额外的软件,特别是我们需要Apache的DAV模块。

```
yum install httpd httpd-devel subversion mod_dav_svn mod_auth_mysql
```

然后你需要为SVN服务器创建一个Apache虚拟主机，并且带DAV权限控制

```
<Location />
	DAV svn
	SVNParentPath /opt/svn # 支持多个版本库
	AuthType Basic
	# SVNListParentPath on # 可以列出所有版本库
	AuthName "Authorization SVN"
	AuthUserFile /opt/svn/conf/passwd.conf
	AuthzSVNAccessFile /opt/svn/conf/authz.conf
	Require valid-user
</Location>
```

接下来就是对目录进行权限控制，以及设置SVN账户了，这里要注意的一点是使用DAV方式，版本库下的svnserve.conf就自动失效了，我们必须通过passwd.conf和authz.conf来保证版本库的安全性。

为SVN添加账户

```
# 首次
htpasswd -c /opt/svn/conf/passwd.conf testuser

# 以后
htpasswd /opt/svn/conf/passwd.conf testuser
```

权限控制，以下配置掩饰了复杂授权控制的方法。

```
[groups]
svn_developer = testuser_1,testuser_2
svn_admin = testuser_2,testuser_3

# 已经离开的人最好还是保留账号，只是禁用权限，这样在blame的时候比较好追踪。
user_left = testuser_4,testuser_5

# 默认只要有SVN账号都可以有读权限，设为已经离开团队的人除外
[/]
@svn_developer = rw
@svn_admin = rw

* = r

@user_left =


# 根据版本库授权
[repo:/trunk]
@svn_developer = rw
@svn_admin = rw

[repo:/tags]
@svn_admin = rw
@svn_developer = r

# 为单个用户提权
testuser_2 = rw


```

以上仅仅是显示了可能性，实际要根据对项目安全性的要求结合团队规划，项目规划进行合理的设置。

如果遇到错误：

```
svn: E195019: Redirect cycle detected for URL ...
```

是因为你设置的DocumentRoot和SVN目录相同的原因，设置一个不同的即可。
