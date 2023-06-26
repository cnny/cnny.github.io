---
layout:     post
title:      "PHP 源码混淆"
subtitle:   ""
date:       2023-06-25 17:09:00
author:     "Cann"
header-img: "img/bg-22.jpg"
tags:
    - PHP
---

### 一：安装 PHP 扩展 [Bolt](https://phpbolt.com)

下载扩展，并解压

```shell
wget https://phpbolt.com/wp-content/uploads/2023/03/phpBolt-extension-1.0.4.zip
unzip phpBolt-extension-1.0.4.zip
```

找到对应版本 .so 扩展文件，并将其复制到 PHP 扩展目录

```shell
cp ./phpBolt-extension-1.0.4/linux\ 64/linux\ 64-php7.4/bolt.so /usr/local/php/lib/php/extensions/no-debug-non-zts-20190902
```

编辑 php.ini 文件
```shell
vi /usr/local/php/etc/php.ini
```

尾部添加一行
```shell
extension=bolt.so
```

### 二：安装 Composer 包：[Laravel-Source-Encrypter](https://github.com/SiavashBamshadnia/Laravel-Source-Encrypter)

```
composer require --dev sbamtr/laravel-source-encrypter
php artisan vendor:publish --provider="sbamtr\LaravelSourceEncrypter\SourceEncryptServiceProvider" --tag=config
```

### 三：执行混淆

```
php artisan encrypt-source
```
混淆后的文件默认在 encrypted 文件夹下，可在配置文件 config/source-encrypter.php 中修改

### 四：用混淆后的文件替换原始文件
