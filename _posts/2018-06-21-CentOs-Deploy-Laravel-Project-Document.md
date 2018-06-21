---
layout:     post
title:      "纯净的CentOS系统搭建Lnmp环境，部署Laravel项目流程"
subtitle:   ""
date:       2018-06-21 12:11:00
author:     "Cann"
header-img: "img/2018-06-21.jpg"
tags:
    - Nginx
    - Laravel
---

#### 安装wget

```
yum install wget
```

#### 安装LNMP环境

下载并安装lnmp一键
1.5完整版：

```
mkdir ~/software
cd ~/software
wget -c http://soft.vpser.net/lnmp/lnmp1.5.tar.gz && tar zxf lnmp1.5.tar.gz && cd lnmp1.5 && ./install.sh lnmp
```

#### 安装ngx_lua_module（用于记录Nginx access_log response body）

下载并安装luajit

```bash
wget http://luajit.org/download/LuaJIT-2.0.2.tar.gz
tar -zxvf LuaJIT-2.0.2.tar.gz
cd LuaJIT-2.0.5
make&&make insatll
```

下载ngx_devel_kit以及nginx_lua_module

```bash
wget https://github.com/simplresty/ngx_devel_kit/archive/v0.3.1rc1.tar.gz
wget https://github.com/openresty/lua-nginx-module/archive/v0.10.13.tar.gz
tar -zxvf v0.3.1rc1.tar.gz
tar -zxvf v0.10.13.tar.gz
```

将lua_module添加至nginx中

找到自己nginx版本的安装包，解压缩后，进入文件夹

```bash
cd ~/software/lnmp1.5-full/src
tar -zxvf nginx-1.14.0.tar.gz
cd nginx-1.14.0
```

`nginx -V` 查看自己nginx的configure arguments,重新编译时，需要带上旧的configure arguments

```bash
./configure {旧的configure arguments}
--with-ld-opt="-Wl,-rpath,/usr/local/lib" --add-module=--add-module=/root/software/ngx_devel_kit-0.3.1rc1 --add-module=/root/software/lua-nginx-module-0.10.13
```

这里注意的是很多人编译的时候没有加选项：--with-ld-opt="-Wl,-rpath,/usr/local/lib"

这样会导致编好的nginx在启动的时候会无法找到位于luajit内的类库，类似于

```
/opt/nginx/sbin/nginx -c /opt/nginx/conf/nginx.conf
/opt/nginx/sbin/nginx: error while loading shared libraries: libluajit-5.1.so.2: cannot open shared object file: No such file or directory
```

这个问题很多人发现后采用了将libluajit-5.1.so.2链接到系统库的做法:

```
ln -s /usr/local/lib/libluajit-5.1.so.2 /lib64/
```

这样可以解决问题，但是相当于一个补救方法。

#### 配置php

修改 `php.ini` 找到 `disable_functions`，解除以下几个方法的禁用：

- passthru
- exec
- system
- shell_exec
- proc_open
- proc_get_status
- symlink

>查找`php.ini`路径：php -i|grep 'php.ini'

配置error_log路径

```
error_log = /home/wwwlogs/php_errors.log
```

#### 配置vhost

添加通用laravel conf

进入`nginx/conf`目录,创建新文件`php-laravel.conf`
```
index index.html index.htm index.php default.html default.htm default.php;

location /
{
    try_files $uri $uri/ /index.php?$query_string;
}

location ~ [^/]\.php(/|$)
{
    fastcgi_pass  unix:/tmp/php-cgi.sock;
    fastcgi_index index.php;
    include fastcgi.conf;
    include pathinfo.conf;
    include resp_body.conf;
}

location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
{
    expires      30d;
}

location ~ .*\.(js|css)?$
{
    expires      12h;
}

location = /favicon.ico {
    log_not_found off;
    access_log off;
}

```

添加access log response body配置文件

进入`nginx/conf`目录,创建新文件`resp_body.conf`
```
lua_need_request_body on;

set $resp_body "";
body_filter_by_lua '
    local resp_body = string.sub(ngx.arg[1], 1, 1000)
    ngx.ctx.buffered = (ngx.ctx.buffered or "") .. resp_body
    if ngx.arg[2] then
        ngx.var.resp_body = ngx.ctx.buffered
    end
';
```

编辑nginx.conf, 添加通用日志格式

```
log_format  access escape=json '$remote_addr - $remote_user [$time_local] "$request" '
     '$status $body_bytes_sent "$http_referer" '
     '"$http_user_agent" $http_x_forwarded_for';

log_format  laravel escape=json '$remote_addr - $remote_user [$time_local] "$request" '
     '$status $body_bytes_sent "$request_body" "$http_referer" '
     '"$http_user_agent" $http_x_forwarded_for '
     '"xsrf_token=$cookie_xsrf-token,laravel_session=$cookie_laravel_session" '
;

log_format  big_api escape=json '$remote_addr - $remote_user [$time_local] "$request" '
     '$status $body_bytes_sent "$request_body" "http_referer" '
     '"$http_user_agent" $http_x_forwarded_for '
     '"x_app_id=$http_x_app_id,x_app_ver=$http_x_app_ver,x_app_build_no=$http_x_app_build_no,x_app_token=$http_x_app_token" '
     '"phpsessid=$cookie_phpsessid" '
     '"$resp_body"'
;

log_format  big_api_no_req escape=json '$remote_addr - $remote_user [$time_local] "$request" '
     '$status $body_bytes_sent "http_referer" '
     '"$http_user_agent" $http_x_forwarded_for '
     '"x_app_id=$http_x_app_id,x_app_ver=$http_x_app_ver,x_app_build_no=$http_x_app_build_no,x_app_token=$http_x_app_token" '
     '"phpsessid=$cookie_phpsessid" '
     '"$resp_body"'
;
```

添加vhost文件

```
server
    {
        listen 80;
        server_name www.xxx.com;
        root  /home/wwwroot/xxx_server_dev/public;
        include php-laravel.conf;

        access_log  /home/wwwlogs/access_xxx_server_dev.log  big_api;
    }
```

#### laravel open_basedir restriction ineffect error解决方法

安装 lnmp 后 发现怎么都运行不了 laravel 一直显示500 状态码，都给了 stoage 目录权限了 。于是查看错误日志，报错如下：

```
Warning: require(): open_basedir restriction in effect. File(/home/wwwroot/default/1211/bootstrap/autoload.php) is not within the allowed path(s): (/home/wwwroot/default/1211/public/:/tmp/:/proc/) in /home/wwwroot/default/1211/public/index.php on line 22

Warning: require(/home/wwwroot/default/1211/bootstrap/autoload.php): failed to open stream: Operation not permitted in /home/wwwroot/default/1211/public/index.php on line 22

Fatal error: require(): Failed opening required '/home/wwwroot/default/1211/public/../bootstrap/autoload.php' (include_path='.:/usr/local/php/lib/php') in /home/wwwroot/default/1211/public/index.php on line 22
```

>错误日志显示，访问脚本不在 open_basedir的限定目录里面，配置open_basedir 一般会在php.ini 或 nginx 配置文件里面

首先检测php.ini 我发现并没有配置 open_basedir

然后检测nginx配置,去到 nginx 根目录，运行

```
grep -rn open_basedir ./
```

结果如下：

```
./conf/fastcgi.conf:27:fastcgi_param PHP_ADMIN_VALUE "open_basedir=$document_root/:/tmp/:/proc/";
```

编辑fastcgi.conf，将这一行注释掉，然后重启nginx、php-fpm即可


