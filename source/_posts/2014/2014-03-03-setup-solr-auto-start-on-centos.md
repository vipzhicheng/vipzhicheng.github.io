---
layout: post
title: "CentOS设置Solr自动启动"
date: 2014-03-03 23:47:05 +0800
comments: true
categories:
- Ops
tags:
- Solr
- CentOS
- Shell
- Jetty

---

在生产服务器上部署Solr时，曾想过应该使用哪种容器运行Solr，使用Resin, Jetty还是Tomcat,后来看了一些评测说性能上Jetty也不是很差，就决定还是用Jetty，而且Jetty是默认就自带的方式，一条命令就能在测试环境启动起来。

<!-- more -->

```
java -jar start
```

在生产服务器上，考虑到希望可以随机启动，所以决定将脚本改写成可以随机自动启动的service形式。Google了一下，发现不同的人的实现都不一样，官方也提供了一种，不过需要依赖于start-stop-daemon这个程序，安装起来比较麻烦。后来我又找到了一种不依赖任何程序的方式。如下：

```
#!/bin/bash
# description: Starts and stops Solr production

set -e
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

SOLR_DIR=/data/solr/ #SOLR DIR
JAVA_OPTIONS="-Dsolr.solr.home=/data/solr/multicore -Djava.util.logging.config.file=logging.properties -server -DSTOP.PORT=8079 -DSTOP.KEY=stopkey -Xmx256M -Xms128M -jar start.jar"
JAVA="/usr/bin/java"
LOG_FILE="/data/solr/logs/solr.log"

case $1 in
  start)
    echo "Starting Solr"
    cd $SOLR_DIR
  nohup $JAVA $JAVA_OPTIONS 2> $LOG_FILE &
    echo "ok - remember it may take a minute or two before Solr responds on requests"
  ;;
  stop)
    echo "Stopping Solr"
    cd $SOLR_DIR
    $JAVA $JAVA_OPTIONS --stop
    echo "ok"
  ;;
  restart)
    $0 stop
    sleep 3
    $0 start
  ;;
 *)
    echo "Usage: $0 {start|stop|restart}" >&2exit 1
  ;;
esac
```

这种方式可以运行，也可以直接用，但我又找了另外一种，也是可以使用的。

```
#!/bin/sh

# Prerequisites:
# 1. Check SOLR_DIR value is correct
# 2. daemon needs to be installed
# 3. Script needs to be executed by root

# This script will launch Solr in a mode that will automatically respawn if it
# crashes. Output will be sent to $LOG. A pid file will be
# created in the standard location.

# chkconfig: 2345 64 36

NAME=solr
SOLR_DIR=/data/solr
COMMAND="java -Dsolr.solr.home=/data/solr/multicore -jar start.jar"
LOG=/var/log/solr/solr.log

start () {
    echo -n "Starting solr..."

    # start daemon
    daemon --chdir=$SOLR_DIR --command "$COMMAND" --respawn --output=$LOG --name=$NAME --verbose

    RETVAL=$?
    if [ $RETVAL = 0 ]
    then
        echo "done."
    else
        echo "failed. See error code for more information."
    fi
    return $RETVAL
}

stop () {
    # stop daemon
    echo -n "Stopping $NAME..."

    daemon --stop --name=$NAME  --verbose
    RETVAL=$?

    if [ $RETVAL = 0 ]
    then
        echo "done."
    else
        echo "failed. See error code for more information."
    fi
    return $RETVAL
}


restart () {
    daemon --restart --name=$NAME  --verbose
}


status () {
    # report on the status of the daemon
    daemon --running --verbose --name=$NAME
    return $?
}


case "$1" in
    start)
        start
    ;;
    status)
        status
    ;;
    stop)
        stop
    ;;
    restart)
        restart
    ;;
    *)
        echo $"Usage: $NAME {start|status|stop|restart}"
        exit 3
    ;; esac

exit $RETVAL
```
相比较而言，我更喜欢第二种，不过第二种需要依赖一个叫做[daemon](http://libslack.org/daemon)的处理守护进程的程序，需要按照以下方式安装。

```
rpm -i http://libslack.org/daemon/download/daemon-0.6.4-1.x86_64.rpm
```

这个脚本我一次就成功了，而且比较好用，可以查看状态。

其实Solr官网提供的应该也是可以运行的，不过由于其依赖的start-stop-daemon是debian的程序，弄起来不是很方便，就放弃了，最后，我决定使用的就是本文的第二种方式。

最后设置自动启动的方式很简单

```
chkconfig --add solr
chkconfig solr on
```

另外，大家在使用这个脚本时需要注意换成你的Solr安装目录。
