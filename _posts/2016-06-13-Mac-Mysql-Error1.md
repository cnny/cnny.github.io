---
layout:     post
title:      "MySQL服务器启动错误 'The server quit without updating PID file"
subtitle:   ""
date:       2016-05-24 12:00:00
author:     "Yuu"
header-img: ""
tags:
    - Mysql
---

    // 尝试找到后缀名为”.err”的log文件，这里记录了更详细的信息。它可能位于：
    /usr/local/var/mysql/your_computer_name.local.err

### mysql未关闭

检查是否有mysql实例正在运行：

    ps -ef | grep mysql

如果是的话，你应该关掉它，或者直接杀掉进程：

    kill -9 PID

其中PID是第一个命令输出的靠近用户名的那个数字（进程ID）

### 权限问题

* 一:

检查 /usr/local/var/mysql/的所有者：

    ls -laF /usr/local/var/mysql/

如果它的所有者是root的话，你应该把它改成mysql或者你的用户名：

    sudo chown -R mysql /usr/local/var/mysql/

* 二:

检查 /usr/local/var/mysql/xxx.err的所有者,若是_mysql的话,将其修改为自己的用户名(我是用这个方法解决的)

    sudo chown -R cnny /usr/local/var/mysql/cnnydeAir.lan.err

* 三:

若还不行的话,尝试删除/usr/local/var/mysql/xxx.err文件


