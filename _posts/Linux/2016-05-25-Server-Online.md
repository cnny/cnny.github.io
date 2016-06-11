---
layout: post
title: PHP项目上线流程
category: Linux
tags: Linux Server
keywords:
description: 在坚哥的淳淳教导下,了解了项目上线的具体流程,特此记录,以供后人(以后的自己)乘凉.
---

##代码发布总体流程

![产品发布流程图](/public/img/posts/peoduct_issue.jpg)

一般来说,一个产品需要有4个角色的不同服务器.
1. 发布机:用来将master分支上的代码分布到生产服,crontab,admincp,预生成上.
2. 生产服:布置线上的代码.大型项目生产服会有多台.
3. admincp:布置管理后台.
4. crontab:执行计划任务.
5. 预生成服:环境与生产服完全相同,数据库也是读取线上数据库,不对外开发,用于调试线上代码,修复只有在线上环境才能复现的BUG.
