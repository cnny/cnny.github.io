---
layout:     post
title:      "PHP7实用的新特性"
subtitle:   ""
date:       2016-11-18 15:41:00
author:     "Cann"
header-img: "img/post-php7-new-features.jpg"
tags:
    - PHP
---

#### 变量类型

**PHP7版本函数的参数和返回值添加了类型限定，该项特性是为PHP7.1的JIT特性做准备。增加类型限定后，PHP JIT可以准确的判断变量类型。生成最佳的机器指令。**

```php
    function test(int $a, string $b, array $c) : int {
        // code
    }
```

> JIT:  JIT是Just In Time 的缩写，表示运行时将指令转化为二进制机器码。对于大部分实际项目并没有性能提升，但是对于计算密集型程序，可以将PHP中的OpCode转化为机器码，大幅度提升性能。

#### null合并运算符

**由于日常使用中存在大量同时使用三元表达式和 isset()的情况， 于是PHP7添加了null合并运算符 (??) 这个语法糖。如果变量存在且值不为NULL， 它就会返回自身的值，否则返回它的第二个操作数。**

```php
    $a = $b ?? 0;
```

**等价于**

```php
    $a = isset($b) ? $b : 0;
```

**(??)还可以这么用**

```php
    $a = $b ?? $c ?? $d;
```

#### 太空船操作符

**太空船操作符用于比较两个表达式。当$a小于、等于或大于$b时它分别返回-1、0或1。 比较的原则是沿用 PHP 的常规比较规则进行的。**

```php
    // 整数
    echo 1 <=> 1; // 0
    echo 1 <=> 2; // -1
    echo 2 <=> 1; // 1

    // 浮点数
    echo 1.5 <=> 1.5; // 0
    echo 1.5 <=> 2.5; // -1
    echo 2.5 <=> 1.5; // 1

    // 字符串
    echo "a" <=> "a"; // 0
    echo "a" <=> "b"; // -1
    echo "b" <=> "a"; // 1
```

#### 通过 define() 定义常量数组

**Array 类型的常量现在可以通过 define() 来定义。在 PHP5.6 中仅能通过 const 定义。**

```php
    define('ANIMALS', [
        'dog',
        'cat',
        'bird'
    ]);

    echo ANIMALS[1]; // 输出 "cat"
```

#### 匿名类

**现在支持通过new class 来实例化一个匿名类，这可以用来替代一些“用后即焚”的完整类定义。**

```php
    $app = new Application;
    $app->setLogger(new class implements Logger {
        public function log(string $msg) {
            echo $msg;
        }
    });

    var_dump($app->getLogger());
```

#### 错误异常

**PHP出现fatal and recoverable errors后，过去Zend引擎会终止PHP运行，PHP7.0后，可以通过try/catch捕获异常(注：曾今try/catch无法捕获该异常)**
**Error一个新的，与之前的Exception分离的Class。**

```php
    $var = 1;
    try {
        $var->method(); // Throws an Error object in PHP 7.
    } catch (Error $e) {
        // Handle error
    }
```

 **注：在PHP7 alpha-2中，新的Exception Class命名为EngineException，但担心命名会引起误会，在正式版中改名为Error**

> Prior to PHP 7 alpha-2, the exception hierarchy in PHP 7 was different. Fatal and recoverable errors threw instances of EngineException, which did not inherit from Exception. Both Exception and EngineException inherited from BaseException. The hierarchy was revised with the RFC I authored, Throwable Interface. I felt switching to Throwable and Error was important to avoid confusion from classes using the suffix Exception that did not extend Exception, as well as being more concise and appealing names.

**同时，你可以使用Throws来达到捕获所有异常的目的.**

```php
    try {
         // Code that may throw an Exception or Error.
    } catch (Throwable $t) {
         // Handle exception
    }
```











