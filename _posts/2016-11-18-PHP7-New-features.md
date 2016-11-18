---
layout:     post
title:      "细说PHP7.0新特性"
subtitle:   ""
date:       2016-11-18 15:41:00
author:     "Yuu"
header-img: "img/_post-php7.jpg"
tags:
    - PHP
---

### PHP7.0---新特性

#### *变量类型
**PHP7版本函数的参数和返回值添加了类型限定，该项特性是为PHP7.1的JIT特性做准备。增加类型限定后，PHP JIT可以准确的判断变量类型。生成最佳的机器指令。**

    function test(int $a, string $b, array $c) : int {
        // code
    }

> JIT:  JIT是Just In Time 的缩写，表示运行时将指令转化为二进制机器码。对于大部分实际项目并没有性能提升，但是对于计算密集型程序，可以将PHP中的OpCode转化为机器码，大幅度提升性能。
> 在PHP7.1中将会添加该性能。

#### *错误异常
**PHP出现fatal and recoverable errors后，过去Zend引擎会终止PHP运行，PHP7.0后，可以通过try/catch捕获异常(注：曾今try/catch无法捕获该异常)**
**Error一个新的，与之前的Exception分离的Class。**

    $var = 1;
    try {
        $var->method(); // Throws an Error object in PHP 7.
    } catch (Error $e) {
        // Handle error
    }

 **注：在PHP7 alpha-2中，新的Exception Class命名为EngineException，但担心命名会引起误会，在正式版中改名为Error**

> Prior to PHP 7 alpha-2, the exception hierarchy in PHP 7 was different. Fatal and recoverable errors threw instances of EngineException, which did not inherit from Exception. Both Exception and EngineException inherited from BaseException. The hierarchy was revised with the RFC I authored, Throwable Interface. I felt switching to Throwable and Error was important to avoid confusion from classes using the suffix Exception that did not extend Exception, as well as being more concise and appealing names.

**同时，你可以使用Throws来达到捕获所有异常的目的.**

    try {
         // Code that may throw an Exception or Error.
    } catch (Throwable $t) {
         // Handle exception
    }


**另外，PHP7.0还提供了一些方法来捕获具体的Error**

1.TypeError：参数类型不正确

    function add(int $left, int $right)
    {
        return $left + $right;
    }

    try {
        $value = add('left', 'right');
    } catch (TypeError $e) {
        echo $e->getMessage(), "\n";
    }

2.ParseError:语法错误（注：A ParseError is thrown when an included/required file or eval()'d code contains a syntax error）

    try {
        require 'file-with-parse-error.php';
    } catch (ParseError $e) {
        echo $e->getMessage(), "\n";
    }

3.ArithmeticError:算术错误

    try {
        $value = 1 << -1;
    } catch (ArithmeticError $e) {
        echo $e->getMessage(), "\n";
    }

4.DivisionByZeroError:除数为0错误

    try {
        $value = 1 % 0;
    } catch (DivisionByZeroError $e) {
        echo $e->getMessage(), "\n";
    }
    
5:AssertionError:assert()方法错误

### PHP7.0---性能优化
#### Zval使用栈内存








