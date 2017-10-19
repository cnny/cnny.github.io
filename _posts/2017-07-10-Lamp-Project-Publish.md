---
layout:     post
title:      "LAMP服务器配置中遇到的一些杂项"
subtitle:   ""
date:       2017-07-10 02:34
author:     "Cann"
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

#### 开启gzip

修改httpd.conf

开启以下两行

    LoadModule deflate_module modules/mod_deflate.so
    LoadModule headers_module modules/mod_headers.so

在httpd.conf 中加入以下段落

    <IfModule deflate_module>
    SetOutputFilter DEFLATE
    SetEnvIfNoCase Request_URI .(?:gif|jpe?g|png)$ no-gzip dont-vary
    SetEnvIfNoCase Request_URI .(?:exe|t?gz|zip|bz2|sit|rar)$ no-gzip dont-vary
    SetEnvIfNoCase Request_URI .(?:pdf|doc|avi|mov|mp3|rm)$ no-gzip dont-vary
    AddOutputFilterByType DEFLATE text/html text/plain text/xml text/css
    AddOutputFilterByType DEFLATE application/x-javascript
    </IfModule>

注解：

**IfModule deflate_module** 是判断如果 deflate_module 模块加载的话，执行里面的配置。

**SetOutputFilter DEFLATE** 是设置输出为 deflate 压缩算法。

**SetEnvIfNoCase Request_URI** 是排除一些常见的图片，影音，文档等类型的后缀，不压缩。

**AddOutputFilterByType DEFLATE** 是对常见的文本类型,如html,txt,xml,css,js做压缩处理。




