---
layout:     post
title:      "PHP代码规范自动修复工具《php-cs-fixer》中文文档"
subtitle:   ""
date:       2017-04-19 17:45:00
author:     "Yuu"
header-img: "img/post-php-cs-fixer.jpg"
tags:
    - PHP
    - code-style
    - php-css-fixer
---

**[官方文档](https://github.com/FriendsOfPHP/PHP-CS-Fixer)**

#### 安装

---

##### composer全局安装

    $ composer global require friendsofphp/php-cs-fixer

同时，确保 `~/.composer/vendor/bin`目录在你的`PATH`中。

    export PATH="$PATH:$HOME/.composer/vendor/bin"

#### 使用

---

##### Composer安装

    $ php-cs-fixer fix /path/to/dir
    $ php-cs-fixer fix /path/to/file

##### 参数

**`--rules`** :

要应用的代码规范。php-cs-fixer提供了三种规范（每个规范都包含了许多具体规则): `PSR1` `PSR2` `Symfony `；默认为`PSR1` `PSR2`,建议将三者都用上

     $ php-cs-fixer fix /path/to/project --rules=@PSR1,@PSR2,@Symfony

你也可以选择只应用确切的某些规则，规则间`,`分割

    $ php-cs-fixer fix /path/to/dir --rules=line_ending,full_opening_tag,indentation_type

当然，你也可以排除自己不想应用的规则，在规则前加上`-`即可

    $ php php-cs-fixer.phar fix /path/to/dir --rules=@PSR1,@PSR2,-full_opening_tag,-indentation_type

具体规范请看[官方文档](https://github.com/FriendsOfPHP/PHP-CS-Fixer#usage)

**`--diff`** and **`--dry-run`** : 将预计要修复的代码块显示在终端，并且不会对原文件进行修改

**`--stop-on-violation`** :在遇到第一个需要修复的文件时，停止执行命令

**`--format`** : output格式，支持`json` `xml` `junit` `txt`暂时未发现大的用处

**`--show-progress`** : 选择进度呈现方式，暂时未发现大的用处

**`--verbose`** : output会输出当前应用的修复规则


