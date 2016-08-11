---
title: 怎样在 CentOS 7 上安装 Nginx
date: 2016-08-11 19:40:03
tags:
- CentOS
- CentOS 7
- Nginx
categories:
- Ops
---

Nginx是一个高性能的Web服务器，比Apache要灵活和轻量。

注，以下安装过程需要具有`root`权限或者用`sudo`来执行，注意这个教程是针对`CentOS 7`的，貌似`CentOS 7`上的软件安装都异常的简单。

## 第一步：添加Nginx软件源

```
sudo yum install epel-release
```

## 第二步：安装Nginx

```
sudo yum install nginx
```

## 第三步：开启Nginx服务

```
sudo systemctl start nginx
```

## 验证是否正常工作

```
http://server_domain_name_or_IP/
```

## 把Nginx加到开机自启动

```
sudo systemctl enable nginx
```

## 默认的配置文件路径和Web根目录

```
# 默认配置文件路径
/etc/nginx/nginx.conf

# 其他自定义配置文件所在目录，可定义虚拟主机，以`.conf`扩展名结尾
/etc/nginx/conf.d

# 默认Web根目录
/usr/share/nginx/html

```