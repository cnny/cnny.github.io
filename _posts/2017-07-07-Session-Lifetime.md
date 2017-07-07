---
layout:     post
title:      "如何设置一个严格30分钟过期的Session"
subtitle:   ""
date:       2017-04-19 17:45:00
author:     "Yuu"
header-img: "img/post-php-cs-fixer.jpg"
tags:
    - PHP
    - Session
---

[如何设置一个严格30分钟过期的Session](http://www.laruence.com/2012/01/10/2469.html)

垃圾回收程序是在每次调用 session_start() 时都会启动的，为了避免过于频繁，影响PHP性能，我们通过在 php.ini 中的 session.gc_probability 和 session.gc_divisor 两个选项来设置启动垃圾回收程序的概率。

例如 session.gc_probability = 1，session.gc_pisor = 100，这样概率就变成了 1/100，建议设置成 1/5000