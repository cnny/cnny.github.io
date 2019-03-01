---
layout:     post
title:      "CGI、FAST-CGI和PHP-FPM"
subtitle:   ""
date:       2019-01-01 04:11:00
author:     "Cann"
header-img: "img/bg-01.jpg"
tags:
    - 基础知识
    - PHP
    - FAST-CGI
    - PHP-FPM
---

#### 什么是CGI?

`Common Gateway Interface` 全称是“通用网关接口”

是`Web Server`与`Web Application`之间数据交换的一种协议。它规定要传哪些数据，以什么样的格式传递给后方处理这个请求。

**缺点**：就是每一次web请求都会有启动和退出过程，也就是最为人诟病的fork-and-execute模式，反复加载导致其性能低下

#### 什么是FAST-CGI?

从根本上来说，FastCGI是用来提高CGI程序性能的。类似于CGI，FastCGI也可以说是一种协议

其主要行为是将CGI解释器进程保持在内存中，并因此获得较高的性能

FastCGI 会先启一个master，解析配置文件，初始化执行环境，然后再启动多个worker。当请求过来时，master会传递给一个worker，然后立即可以接受下一个请求。这样就避免了重复的劳动，效率自然是高。而且当worker不够用时，master可以根据配置预先启动几个worker等着；当然空闲worker太多时，也会停掉一些，这样就提高了性能，也节约了资源。master和children之间通过共享内存来共享配置文件

它还支持分布式的运算, 即 FastCGI 程序可以在网站服务器以外的主机上执行，并且接受来自其它网站服务器来的请求。

#### 什么是PHP-CGI?

**PHP-CGI就是PHP实现的自带的FastCGI管理器**

但其性能低下、麻烦且不人性化，主要体现在：

- php-cgi变更php.ini配置后，需重启php-cgi才能让新的php-ini生效，不可以平滑重启。
- 直接杀死php-cgi进程，php就不能运行了。

#### 什么是PHP-FPM?

**PHP-FPM 是对于 FastCGI 协议的具体实现**

它是用于调度管理PHP解析器php-cgi的管理程序。

相较于PHP-CGI，它不仅克服了PHP-CGI的缺点，
CGI是为了保证web server传递过来的数据是标准格式的，方便CGI程序的编写者。
