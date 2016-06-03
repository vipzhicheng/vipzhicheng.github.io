---
layout: post
title: "Jenkins的安全防护"
date: 2014-03-09 17:50:57 +0800
comments: true
categories:
- Ops
tags:
- CentOS
- Jenkins
- 安全

---

在[这篇文章](http://www.vipzhicheng.com/blog/2014/02/24/yum-install-jenkins-on-centos/)中我们我们提到默认Jenkins是不提供任何权限控制的，但通过配置，可以增加用户名密码登陆机制。但更进一步的，我们不希望有人能够扫描到Jenkins的端口，通常默认是8080，所以我的思路是使用Apache的反向代理配置，并且使用HTTP BASIC AUTH机制。

<!-- more -->

反向代理的配置如下:

```
<VirtualHost *:80>
    ServerName your.domain.com
    ProxyPass / http://127.0.0.1:8080/
    ProxyPassReverse / http://127.0.0.1:8080/
    ProxyRequests Off
    ErrorLog logs/jenkins-error_log
    CustomLog logs/jenkins-access_log common

   <Location />
      AuthUserFile /var/www/html/etc/htpasswd
      AuthType Basic
      AuthName "Jenkins"
      Require valid-user
   </Location>

</VirtualHost>
```
htpasswd文件是通过Apache的htpasswd命令设置的。

但是启动之后发现输入了BASIC AUTH的用户名和密码之后，又弹出了Jenkins自己的BASIC AUTH，通过Google，找到如下解决方案，在httpd.conf里添加如下配置，这个依赖于mod_headers模块。

```
# remove auth from headers
RequestHeader unset Authorization
```
