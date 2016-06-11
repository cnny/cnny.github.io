---
layout: post
title: centOs搭建Lnmp环境
category: Linux
tags: Lamp Linux PHP
keywords:
description:
---

## 安装 Nginx

### 添加资源库

    vim /etc/yum.repos.d/nginx.repo

将以下代码复制粘贴到该文件

    [nginx]
    name=nginx repo
    baseurl=http://nginx.org/packages/centos/$releasever/$basearch/
    gpgcheck=0
    enabled=1

### 安装nginx

    yum install nginx

### 测试

    service nginx status

应该会返回:

    nginx is stopped （nginx 已停止）

再测试一下 nginx 的配置文件：

    nginx -t

应该会返回：

    nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
    nginx: configuration file /etc/nginx/nginx.conf test is successful

### nginx的启动关闭重启

    service nginx start
    service nginx stop
    service nginx restart

## 配置 php-fpm

### 安装

    yum install php-fpm

### 让 nginx 可以执行 php

现在我们应该就可以让 nginx 去执行 php 了。不过你需要修改一下 nginx 的配置文件，之前我们在配置虚拟主机的时候，创建了一个 nginx.ninghao.net.conf 的配置文件，需要去修改下 nginx 的这个配置文件，才能去执行 php 。使用 vim 命令去编辑它：

    vim /etc/nginx/conf.d/nginx.ninghao.net.conf

注意你的配置文件不一定叫 nginx.ninghao.net.conf，应该是你自己命名的配置文件。打开以后，找到下面这段字样的代码：

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    #location ~ \.php$ {
    #    root           html;
    #    fastcgi_pass   127.0.0.1:9000;
    #    fastcgi_index  index.php;
    #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
    #    include        fastcgi_params;
    #}

这是 nginx 默认给我们的用来执行 php 的配置，从 location 开始取消注释，会让这个配置生效，然后我们还得简单去修改一下：

    # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
    #
    location ~ \.php$ {
    #   root           html;
        fastcgi_pass   127.0.0.1:9000;
        fastcgi_index  index.php;
        fastcgi_param  SCRIPT_FILENAME  $document_root$fastcgi_script_name;
        include        fastcgi_params;
    }

注意 root 那里仍然是被注释掉的，还有 SCRIPT_FILENAME 后面修改了一下，把 /scripts 换成了 $document_root 。保存并退出。然后重新启动 nginx：

    service nginx restart

