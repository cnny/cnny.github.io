---
layout:     post
title:      "利用Git-Hook实现的PHP代码自动检测"
subtitle:   ""
date:       2017-08-25 20:41:00
author:     "Cann"
header-img: "img/20170927.jpeg"
tags:
    - Git
    - PHP
    - Shell
---

 1. [代码检测工具：PHP_CodeSniffer](https://github.com/squizlabs/PHP_CodeSniffer)
 2. [代码检测工具：PHP_Cs_Fixer(备选)](https://github.com/FriendsOfPHP/PHP-CS-Fixer)
 3. [参考文章--Git-Hook文档](https://git-scm.com/book/zh/v2/%E8%87%AA%E5%AE%9A%E4%B9%89-Git-Git-%E9%92%A9%E5%AD%90)
 4. [参考文章--Shell 切割字符串方法--IFS的使用](http://smilejay.com/2011/12/bash_ifs/)
 5. [参考文章--销毁IFS效果](http://blog.csdn.net/csfreebird/article/details/7974298)
 6. [参考文章--Shell中正则的使用](http://www.111cn.net/phper/210/88457.htm)

#### 需求
利用Git的hook功能，实现在`git commit`代码时，对本次提交的PHP文件自动进行代码规范校验，若代码符合规范，正常COMMIT，不符合规范，中断提交，并显示不合规范的内容。

#### Step 1 -- 代码校验工具
对代码规范进行校验很很简单，GitHub上有两个工具，分别是PHP_CodeSniffer和PHP_Cs_Fixer，功能差不多，我选择的是前者。composer install后，执行`./vendor/bin/phpcs` 就可以对文件进行校验

#### Step 2 -- GitHook
GitHook脚本文件只需要放在`.git/hooks`目录下，并命名为特定名称，即会在恰当场景中自动触发。例如 `pre-commit`会在`git commit`是触发，而`post-commit`会在`git commit`完成后触发。

脚本返回非零数，则自动终止Git操作

#### Step 3 -- 编写Shell脚本
因为仅仅需要对用户要提交的文件进行校验，一开始的思路是：获取`git diff --stat`的执行结果，例：

```
controllers/cann1Controller.php | 4 +++- controllers/cann2Controller.php | 2 +-  1 file changed, 1 insertion(+), 1 deletion(-)
```

再利用正则匹配，将该字符串中的PHP文件匹配出来，最后将所有文件循环执行代码校验程序，若有代码规范error，将其输出，并输出1，终止Git操作。

##### A：Shell中的正则
Shell中利用`=~`进行匹配，再利用`${BASH_REMATCH[n]}`获取匹配到的值，例：

```
if [[ 11:22:33:44 =~ ([0-9]{2}):([0-9]{2}):([0-9]{2}):([0-9]{2}) ]]
    then
        echo ${BASH_REMATCH[1]}
        echo ${BASH_REMATCH[2]}
        echo ${BASH_REMATCH[3]}
        echo ${BASH_REMATCH[4]}
    fi
```

输出结果：

```
11
22
33
44
```

##### B：将字符串切割为数组
在实际操作中发现一个问题，`=~`运算符，只会对一个字符串进行一次匹配，这就意味着，我用`git diff --stat`获取到的字符串中，即使有多个文件，也仅仅会匹配到第一个文件。

于是有了将这个字符串按照一定规则切割为数组，再循环匹配的方案。

经过查询，确定了切割字符串的运算符为`IFS`

使用方式：

```
#!/bin/sh
diffPath=www/eee/rrr
statusCode=0

IFS='/'

for path in diffPath
do
    echo path
done
```
输出

```
www
eee
rrr
```

>注意：`IFS`作用于为全局，也就是说，只要你在脚本中定义了`IFS`,那么脚本中的所有字符串都会按照这个规则进行切割。解决方案为再定义：`IFS=$*`

###### C：`git diff --stat` 无法获取新增文件
在脚本即将写完时才发现这个坑，`git diff --stat`仅仅能够检测到modified的文件，这样新文件就会绕过校验。于是决定将`git diff --stat`换成`git status`,获取信息如下：

```
On branch develop
Your branch is up-to-date with 'origin/develop'.
Changes to be committed:
  (use "git reset HEAD <file>..." to unstage)

    new file:   test.php

Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   controllers/RechargeblueController.php

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    custom_ruleset.xml
```

所幸，脚本整体逻辑并不需要修改，仅仅修改正则匹配规则就可以了。另外，用`git status`一个优点：能够选择性匹配`new file` `modified` `untracked`文件。

>PS：这里有遇到上面说的`IFS`作用域的坑，`git diff --stat` 我是用`|`进行字符串切割，所以并没有其他字符串受到影响，但`git status`是根据`换行`进行切割，结果把`PHP_CodeSniffer`的校验结果也进行了切割，导致最后输出的`error message`没有换行，挤成一团。

以上！

##### 脚本：

```sh
#!/bin/bash
diffPath=$(git status)
statusCode=0
isStaged=1
notStagedMsg="Changes not staged for commit"

IFS=$'\x0A'

for path in $diffPath
do
    if [[ $path =~ $notStagedMsg ]]
    then
        isStaged=0
    fi

    if [[ isStaged -eq 1 && ($path =~ new\ file:\ \ \ (.*\.php) || $path =~ modified:\ \ \ (.*\.php)) ]]
    then
        IFS=$*
        error=$(./vendor/bin/phpcs --standard=ruleset.xml ${BASH_REMATCH[1]})
        if [[ ${error} ]]
        then
            error=${error//$'\x0A'                   / }
            error=${error//$'\x0A'                / }
            statusCode=1
            echo ${error}
            echo $'\x0A'
        fi
   fi
done
exit ${statusCode}
```


