---
layout:     post
title:      "LAMP服务器配置中遇到的的一些坑"
subtitle:   ""
date:       2017-07-10 02:34
author:     "Yuu"
header-img: "img/post-lamp-project-publish.jpg"
tags:
    - LAPM
    - apache
---

[所使用的镜像源](https://market.aliyun.com/products/53398003/cmjj017167.html?spm=5176.2020520101.image.selectFromMarketplace.211d5c9ajh3Mad)

#### 403 forbidden错误

原因: DocumentRoot 目录Apache没有访问权限

解决方案：

01: 将 DocumentRoot 目录所有者设为 'apache'

02:

    <Directory "/xxx">
        Options Indexes FollowSymLinks
        AllowOverride All
        Order allow,deny
        Allow from all
        Require all granted
    </Directory>


#### 500 错误

原因： 一般是PHP配置错误或rewrite错误，我遇到的是后者

解决方案: 在 httpd.conf 中开启 LoadModule rewrite_module modules/mod_rewrite.so

#### Invalid command 'RewriteEngine' / 'RewriteCond' / 'RewriteRule'

原因: 配置了 .htaccess 却未开启 rewrite

解决方案：同上

#### 设置mysql远程访问权限（有危险性，生产服建议不要打开/或限制IP）

如果你想允许用户jack从ip为10.10.50.127的主机连接到mysql服务器，并使用654321作为密码

    mysql>GRANT ALL PRIVILEGES ON *.* TO 'jack'@’10.10.50.127’ IDENTIFIED BY '654321' WITH GRANT OPTION;
    mysql>FLUSH RIVILEGES




