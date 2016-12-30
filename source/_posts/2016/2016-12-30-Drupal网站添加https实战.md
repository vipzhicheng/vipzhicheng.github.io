---
title: Drupal网站添加https实战
date: 2016-12-28 22:10:43
tags:
- Drupal
- HTTPS
- 证书
- Docker
- Letsencrypt
- Certbot
categories:
- Ops
---

# 引言

时间即将进入2017年，2017年，苹果开始强制要求APP使用https的接口；2017年，各大营销平台对落地页的要求是必须是https页面；2017年，是微信小程序元年，微信小程序要求接口必须是https的。2017年，https已经到来，你和你的网站准备好了么？

怀着这样一个目的，我开始了给自己的网站安装https的历程。首先，介绍一下我的网站，Drupal项目社区，域名是：[www.drupalproject.org](https://www.drupalproject.org)。主要是为了Drupal在国内的推广以及Drupal新手查阅一些相关资料。这个网站目前基于Drupal7，有多个子域名，是典型的Drupal多站点架构。

在实施的过程中，我最大的体会是大部分查阅的资料很多都没办法用上，因为每个人的网站运行环境是不同的，所以我不打算把这篇文字写的特别细，只概述几个我印象特别深刻的几个地方，希望能给朋友们一些思路和灵感。

# 谁可以给我证书

第一件事就是到哪里注册证书，查阅了一些资料，大家首推的是[Let’s Encrypt](https://letsencrypt.org/)，其他的我也就不提及了，只说一下这个CA是免费的，各浏览器都信任，可以自动更新。在选择机构上面，我并没有花太多时间。通过官网的介绍，我知道我需要用一个叫[Certbot](https://certbot.eff.org/)的东西当客户端。

# Docker

本文不会详细介绍Docker，但我的网站目前是基于Docker架构运行的，通过Docker Compose管理，内部基于Nginx和php-fpm，通过几个月的尝试，非常的稳定，而且易于运维，所以非常推荐大家也开始尝试Docker。

# Certbot

由于用的是Docker，所以Certbot也要在Docker的架构下，在Docker官方的[镜像仓库](http://hub.docker.com)搜了一下，有很多私有镜像已经帮我做好了，通过尝试，我选择了其中的一个，以下是docker-compose.yml中的相关配置。

```
version: '2'

volumes:
   letsencrypt:
     driver: local

services:
  certbot:
    container_name: certbot
    image: quay.io/letsencrypt/letsencrypt
    ports:
      - '443:443'
      - '80:80'
    volumes:
      - letsencrypt:/etc/letsencrypt
      - /var/lib/letsencrypt:/var/lib/letsencrypt
    entrypoint: ""
    command: bash -c "certbot auth -n --standalone -d drupalproject.org -d www.drupalproject.org -d code.drupalproject.org -d api.drupalproject.org --agree-tos --email 我的邮箱"

```

这里不会讲Docker Compose配置文件的语法，大家需要注意到的是，我使用的镜像是quay.io/letsencrypt/letsencrypt，是由CoreOS提供的。启动这个容器之后，我们就能得到证书了，是不是很简单？

证书的地址在容器中的位置是：

```
/etc/letsencrypt/live/drupalproject.org
```

在实际使用时，我是通过Docker容器卷挂载的方式，所以路径会有所不同。


大家可能会注意到，我分别为每个子域名申请了证书，因为Let's encrypt不发放泛域名证书，这一点大家要注意，另外证书的有效期只有3个月。

那么证书过期了怎么办呢？我们需要在证书过期之前更新证书，这一点Certbot客户端为我们封装了命令，使用起来很简单，借助于Docker这个更新就更方便了，以下是命令，要注意的是启动之前要关闭Nginx，执行后再开启Nginx，我是把这件事情放在服务器的计划任务的，但是也可以手动执行。

```
docker-compose run --rm --service-ports certbot certbot renew
```

# 使用证书

得到了证书只是第一步，我们还需要把证书配置在Web服务器中，例如Apache或者Nginx，我自己用的是Nginx，所以我以Nginx为例。并且这里要提前跟大家说的是，证书配置到Nginx之后还不行，后面还要做一些其他配置。

以下是我Nginx上证书相关的配置。

```

# Part 1
server {
    listen 80;
    listen 443 ssl;

    server_name drupalproject.org;
    return 301 $scheme://www.drupalproject.org$request_uri;
}

# Part 2
server {
    listen 80;

    server_name *.drupalproject.org;
    return 301 https://$host$request_uri;
}

# Part 3
server {
    listen 443 ssl;

    server_name www.drupalproject.org code.drupalproject.org api.drupalproject.org;
    
    ssl on;
    ssl_certificate     /etc/nginx/certs/live/drupalproject.org/fullchain.pem;
    ssl_certificate_key /etc/nginx/certs/live/drupalproject.org/privkey.pem;
}

```

这里的配置只是和证书相关的一部分，不是我完整虚拟主机的配置。并且也不是每个人都需要和我一样的配置，这个配置只是一个参考。下面我来解释各个部分的含义：

* Part 1 是用于把drupalproject跳转到www.drupalproject.org，不管是不是https。
* Part 2 是把任何非https的二级域名跳转到https的向应二级域名。
* Part 3 是真正的对https的请求，挂载https证书。

我个人认为这样的配置，对我来说是最合理的一种配置。

# 网站配置

这样配置以后，网站本身已经支持https了，但是由于网站还没有进行配置，所以现在打开的话，网页浏览还是不正常的，地址栏还没有出现绿色的锁头。

下面是网站需要做的相关配置：

settings.php
```
$conf['https'] = TRUE;
```

开启了这项服务之后，网站是可以在http和https中共享session的，虽然我要的是全站https，但我还是开着，万一以后改成半http半https呢？

https网站如果想要被浏览器信任，需要进一步加载的资源都是https的，这就需要网站在https访问时对网站代码进行一个替换，这种替换有多种方法，比如只修改$base_url就是其中最简单的一种。这里我不是这样做的。我用的完整代码如下，大家可以看到我封装了一个http2https的模块，因为我还不认为这是最佳方式，只是可以工作的一种方式，还有进一步的优化空间：

```
<?php
define('HTTP2HTTPS_SERVER_KEY', 'HTTPS');

/**
 * Implement hook_page_delivery_callback_alter
 *
 * @param $delivery_callback
 */
function http2https_page_delivery_callback_alter(&$delivery_callback) {
  if (isset($_SERVER[HTTP2HTTPS_SERVER_KEY]) && !empty($_SERVER[HTTP2HTTPS_SERVER_KEY])) {
    $delivery_callback = 'http2https_drupal_deliver_html_page_https';
  }
}

function http2https_drupal_deliver_html_page_https($page_callback_result) {

  if (is_int($page_callback_result)) {
    drupal_deliver_html_page($page_callback_result);
  }
  elseif (isset($page_callback_result)) {
    // Emit the correct charset HTTP header, but not if the page callback
    // result is NULL, since that likely indicates that it printed something
    // in which case, no further headers may be sent, and not if code running
    // for this page request has already set the content type header.
    if (isset($page_callback_result) && is_null(drupal_get_http_header('Content-Type'))) {
      drupal_add_http_header('Content-Type', 'text/html; charset=utf-8');
    }

    // Send appropriate HTTP-Header for browsers and search engines.
    global $language;
    drupal_add_http_header('Content-Language', $language->language);

    // Print anything besides a menu constant, assuming it's not NULL or
    // undefined.
    $content =  drupal_render_page($page_callback_result);
    $content = str_replace('http://', 'https://', $content);
    $content = str_replace('http:\/\/', 'https:\/\/', $content);

    print $content;

    // Perform end-of-request tasks.
    drupal_page_footer();
  }
}
```

# 总结

在以上代码和配置的作用下，我的网站终于实现了全站https。虽然本文篇幅不长，但是从我开始决定这么做，到我成功实现实际用了很长时间，踩了无数的坑儿，我的摸索之路也不是按照上面的顺序一帆风顺，而是来回调整的。
