---
layout: post
title: "最新安装Redmine的方法"
date: 2014-04-09 18:02:45 +0800
comments: true
categories:
- Ops
tags:
- Redmine
- RVM
- Ruby on Rails
- CentOS

---

Redmine官网的安装向导基本上不怎么更新，里面的安装说明比较混乱，看着迷糊，实施起来也会遇到诸多问题，但其实现在最新版的Redmine安装已经非常简单了，当然如果英文比较好的话，按源码包里的安装说明文档也是可以的，本文结合不久前的实施经验记录最新的Redmine安装流程。

<!-- more -->

## Redmine版本

本文使用的是Redmine的是2.5.1版，撰写本文时的最新版本，理论上以后的安装一定要比本文记录的还要简单才对。

## 操作系统

CentOS 6.3，因此本文至少适用于CentOS 6.x

## 通过RVM安装Ruby

请参考之前笔者翻译的一篇教程[使用RVM在CentOS6安装Ruby on Rails](http://vipzhicheng.dev/blog/2014/04/08/how-to-install-ruby-on-rails-on-centos-6-with-rvm/)，由于Redmine会自己选择合适的版本，所以为了安装Redmine的话，你不需要执行最后的Rails安装。另外和这个教程一样，本文也推荐使用root用户来实施，少了许多麻烦，如果以后有其他安全性的需求而想要使用独立的非root账号，可以在安装成功了之后再考虑。

我们需要安装Ruby 1.9.3，而不是最新的Ruby。理论上再新一些的Ruby也支持，可以查看Redmine源码里的安装文档里写的Ruby支持的版本，因为源码里的更新比较及时，官方WIKI文档里的信息很多都过时了。

## 下载Redmine

去Redmine[官方下载页面](http://www.redmine.org/projects/redmine/wiki/Download)下载最新版的Redmine, 这里我们以撰写本文时最新的2.5.1为例。

```
sudo su
cd /var/www
wget http://www.redmine.org/releases/redmine-2.5.1.tar.gz
tar xvzf redmine-2.5.1.tar.gz
mv redmine-2.5.1 redmine
```

## 安装Redmine

然后可以先修改一下数据库链接配置

```
cd redmine
cp config/database.yml.example config/database.yml
vim config/database.yml
```

修改里面的production段即可，因为一般我们安装Redmine只是为了使用。

Redmine使用的是bundler处理版本依赖，这比一个一个手动安装要方便多了。

```
gem install bundler
```

通过上面的指令安装好bundler之后，我们还需要为系统安装两个包，一个是mysql，一个是ImageMagick, 我们假设使用的是MySQL，需要注意的是，即使你的MySQL不是安装在本地，为了编译成功也是需要安装的，另外如果你想使用Redmine的甘特图导出功能，就必须安装ImageMagick。

```
yum install -y ImageMagick-devel
yum install -y mysql-devel
```

然后我们就要进行最重要的操作了。

```
bundle install --without development test
```

上面的指令如果不出错，那么你会看到许多的Redmine依赖的Ruby扩展的安装记录。

然后为了正常使用Redmine的认证系统，我们需要为其生成一个Session TOKEN。

```
rake generate_secret_token
```

接下来需要初始化数据库

```
rake db:migrate RAILS_ENV="production"
```

到这里，Redmine就安装完了，使用以下方式测试：

```
ruby script/rails server -e production
```

如果启动成功，Redmine可以通过 http://yourip:3000 访问，或者如果是本地就是 http://127.0.0.1:3000 ，如果可以访问成功，就用admin/admin账户登陆，然后就可以修改密码，创建项目了。

如果觉得一切正常，可以将Redmine安装成守护进程。

```
ruby script/rails server -e production -d
```

## 将Redmine程序设置成开机自启动

开机自启动有许多方式，尤其是chkconfig的方式，不过这里我们使用最原始的方式，把启动脚本加入到rc.local里。注意路径使用的是绝对路径。

```
/usr/local/rvm/rubies/ruby-1.9.3-p545/bin/ruby /var/www/redmine/script/rails server -e production -d
```

另外还在网上找了一个Redmine启动控制脚本，可以尝试使用。

```
#!/bin/bash

# Modify it to your configuration
DIR=/var/www/html/redmine/

# Start Redmine in daemon mode.
start(){
  cd $DIR
  ruby script/rails server -d -e production
}
# Stop Redmine daemon
stop(){
  RUBYPID=`ps aux | grep "ruby script/rails" | grep -v grep | awk '{print $2}'`
  if [ "x$RUBYPID" != "x" ]; then
    kill -2 $RUBYPID
  fi
}
# Check if Redmine is running
status(){
  RUBYPID=`ps aux | grep "ruby script/rails" | grep -v grep | awk '{print $2}'`
  if [ "x$RUBYPID" = "x" ]; then
    echo "* Redmine is not running"
  else
    echo "* Redmine is running"
  fi
}

case "$1" in
  start)
    start
    status
    ;;
  stop)
    stop
    sleep 2
    status
    ;;
  status)
    status
    ;;
  restart|force-reload)
    stop
    start
    ;;
  *)
    echo "Usage: $0 {start|stop|restart|force-reload|status}"
    exit 1
esac
```

## 为Redmine选择合适的插件和主题

Redmine支持安装插件和更换主题，这样可以获得更丰富的功能和更漂亮的外观，可以去以下网址下载。

* 插件：http://www.redmine.org/projects/redmine/wiki/Plugins
* 主题：http://www.redmine.org/projects/redmine/wiki/Themes

## 写在最后

其实Redmine部署以后，为了更好的访问，我们一般还需要再外层加一层反向代理，这个有许多方法，这里就不再赘述了。
