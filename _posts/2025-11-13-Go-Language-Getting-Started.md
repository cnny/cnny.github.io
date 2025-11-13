---
layout:     post
title:      "Go语言入门指南 - 从PHP到Go"
subtitle:   "PHP开发者快速上手Go语言"
date:       2025-11-13 10:00:00
author:     "Cann"
header-img: "img/bg-01.jpg"
tags:
    - Go
    - PHP
    - 编程语言
---

## 前言

作为PHPer，学习Go语言是一个很好的选择。Go语言由Google开发，具有简洁的语法、强大的并发支持和优秀的性能。本文将从PHP开发者的角度，通过对比PHP和Go的差异，帮助你快速入门Go语言。

## 一、环境安装

### Go安装

```bash
# macOS (使用Homebrew)
brew install go

# 验证安装
go version
```

### 环境变量配置

Go会自动设置以下环境变量：
- `GOROOT`: Go的安装路径
- `GOPATH`: 工作空间路径（Go 1.11+ 使用modules后不再必需）
- `PATH`: 需要包含 `$GOROOT/bin`

## 二、第一个Go程序

### PHP vs Go: Hello World

**PHP版本：**
```php
<?php
echo "Hello, World!";
```

**Go版本：**
```go
package main

import "fmt"

func main() {
    fmt.Println("Hello, World!")
}
```

**关键差异：**
1. **包声明**: Go必须声明包名，`main`包表示可执行程序
2. **导入**: 使用`import`导入标准库`fmt`（格式化I/O）
3. **函数**: `main()`函数是程序入口点
4. **语句结束**: Go不需要分号（除非一行多条语句）

### 运行Go程序

```bash
# 方式1: 直接运行
go run hello.go

# 方式2: 编译后运行
go build hello.go
./hello
```

## 三、变量和常量

### 变量声明

**PHP版本：**
```php
<?php
$name = "Cann";
$age = 30;
$isActive = true;

// 类型声明（PHP 7+）
function greet(string $name): string {
    return "Hello, " . $name;
}
```

**Go版本：**
```go
package main

import "fmt"

func main() {
    // 方式1: 使用var关键字
    var name string = "Cann"
    var age int = 30
    var isActive bool = true
    
    // 方式2: 类型推断（推荐）
    var name2 = "Cann"
    var age2 = 30
    
    // 方式3: 短变量声明（最常用）
    name3 := "Cann"
    age3 := 30
    isActive3 := true
    
    fmt.Println(name, age, isActive)
}
```

**关键差异：**
1. **类型系统**: Go是静态类型语言，变量类型在编译时确定
2. **变量声明**: 使用`var`关键字或`:=`短声明
3. **类型推断**: Go可以自动推断类型
4. **未使用变量**: Go不允许有未使用的变量（编译错误）

### 常量

**PHP版本：**
```php
<?php
define('PI', 3.14159);
const STATUS_ACTIVE = 'active';
```

**Go版本：**
```go
package main

import "fmt"

const PI = 3.14159
const STATUS_ACTIVE = "active"

// 常量组
const (
    StatusActive   = "active"
    StatusInactive = "inactive"
)

// iota（常量生成器）
const (
    Monday = iota + 1  // 1
    Tuesday            // 2
    Wednesday          // 3
)

func main() {
    fmt.Println(PI, STATUS_ACTIVE)
}
```

## 四、数据类型

### 基本类型对比

| PHP | Go | 说明 |
|-----|-----|------|
| `int` | `int`, `int8`, `int16`, `int32`, `int64` | 整数类型 |
| `float` | `float32`, `float64` | 浮点数 |
| `string` | `string` | 字符串（不可变） |
| `bool` | `bool` | 布尔值 |
| `array` | `array`, `slice`, `map` | 数组/切片/映射 |
| `null` | `nil` | 空值 |

### 字符串操作

**PHP版本：**
```php
<?php
$str = "Hello";
$str2 = "World";
$result = $str . " " . $str2;  // 拼接
$length = strlen($str);         // 长度
$sub = substr($str, 0, 3);      // 子串
```

**Go版本：**
```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    str := "Hello"
    str2 := "World"
    
    // 字符串拼接
    result := str + " " + str2
    result2 := fmt.Sprintf("%s %s", str, str2)
    result3 := strings.Join([]string{str, str2}, " ")
    
    // 长度
    length := len(str)
    
    // 子串
    sub := str[0:3]  // "Hel"
    
    fmt.Println(result, length, sub)
}
```

### 数组和切片

**PHP版本：**
```php
<?php
// 索引数组
$arr = [1, 2, 3, 4, 5];
$arr[] = 6;  // 追加

// 关联数组
$map = [
    "name" => "Cann",
    "age" => 30
];

// 遍历
foreach ($arr as $value) {
    echo $value;
}

foreach ($map as $key => $value) {
    echo "$key: $value";
}
```

**Go版本：**
```go
package main

import "fmt"

func main() {
    // 数组（固定长度）
    var arr [5]int = [5]int{1, 2, 3, 4, 5}
    arr2 := [5]int{1, 2, 3, 4, 5}
    arr3 := [...]int{1, 2, 3, 4, 5}  // 自动推断长度
    
    // 切片（动态数组，更常用）
    slice := []int{1, 2, 3, 4, 5}
    slice = append(slice, 6)  // 追加元素
    
    // Map（关联数组）
    m := map[string]int{
        "age": 30,
    }
    m["name"] = 0  // 注意：Go的map值类型必须一致
    
    // 更合适的map定义
    info := map[string]interface{}{
        "name": "Cann",
        "age":  30,
    }
    
    // 遍历数组/切片
    for i, v := range slice {
        fmt.Printf("索引: %d, 值: %d\n", i, v)
    }
    
    // 遍历map
    for k, v := range info {
        fmt.Printf("键: %s, 值: %v\n", k, v)
    }
}
```

**关键差异：**
1. **数组vs切片**: Go区分固定长度数组和动态切片
2. **类型一致**: Go的map所有值必须是同一类型
3. **遍历**: 使用`range`关键字
4. **追加**: 使用`append()`函数（返回新切片）

## 五、函数

### 函数定义

**PHP版本：**
```php
<?php
function add($a, $b) {
    return $a + $b;
}

function greet($name = "Guest") {
    return "Hello, " . $name;
}

function sum(...$numbers) {
    return array_sum($numbers);
}
```

**Go版本：**
```go
package main

import "fmt"

// 基本函数
func add(a int, b int) int {
    return a + b
}

// 参数类型简写
func add2(a, b int) int {
    return a + b
}

// 多返回值（Go的特色）
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, fmt.Errorf("除数不能为0")
    }
    return a / b, nil
}

// 命名返回值
func calculate(a, b int) (sum int, product int) {
    sum = a + b
    product = a * b
    return  // 自动返回sum和product
}

// 可变参数
func sum(numbers ...int) int {
    total := 0
    for _, num := range numbers {
        total += num
    }
    return total
}

func main() {
    result := add(1, 2)
    fmt.Println(result)
    
    // 多返回值处理
    quotient, err := divide(10, 2)
    if err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("结果:", quotient)
    }
    
    // 忽略返回值
    _, product := calculate(3, 4)
    fmt.Println("乘积:", product)
}
```

**关键差异：**
1. **类型声明**: 参数和返回值都需要类型
2. **多返回值**: Go支持返回多个值（常用于错误处理）
3. **命名返回值**: 可以给返回值命名
4. **错误处理**: 通常用第二个返回值表示错误

### 匿名函数和闭包

**PHP版本：**
```php
<?php
$greet = function($name) {
    return "Hello, " . $name;
};

$multiplier = function($x) {
    return function($y) use ($x) {
        return $x * $y;
    };
};

$double = $multiplier(2);
echo $double(5);  // 10
```

**Go版本：**
```go
package main

import "fmt"

func main() {
    // 匿名函数
    greet := func(name string) string {
        return "Hello, " + name
    }
    fmt.Println(greet("Cann"))
    
    // 闭包
    multiplier := func(x int) func(int) int {
        return func(y int) int {
            return x * y
        }
    }
    
    double := multiplier(2)
    fmt.Println(double(5))  // 10
}
```

## 六、结构体和方法

### 结构体（类似PHP的类）

**PHP版本：**
```php
<?php
class User {
    private $name;
    private $age;
    
    public function __construct($name, $age) {
        $this->name = $name;
        $this->age = $age;
    }
    
    public function getName() {
        return $this->name;
    }
    
    public function greet() {
        return "Hello, I'm " . $this->name;
    }
}

$user = new User("Cann", 30);
echo $user->greet();
```

**Go版本：**
```go
package main

import "fmt"

// 定义结构体
type User struct {
    Name string
    Age  int
}

// 方法（值接收者）
func (u User) Greet() string {
    return fmt.Sprintf("Hello, I'm %s", u.Name)
}

// 方法（指针接收者，可以修改结构体）
func (u *User) SetAge(age int) {
    u.Age = age
}

// 构造函数（Go没有构造函数，通常用函数实现）
func NewUser(name string, age int) *User {
    return &User{
        Name: name,
        Age:  age,
    }
}

func main() {
    // 创建结构体实例
    user1 := User{Name: "Cann", Age: 30}
    user2 := User{"Cann", 30}  // 按顺序
    user3 := NewUser("Cann", 30)
    
    fmt.Println(user1.Greet())
    user1.SetAge(31)
    fmt.Println(user1.Age)
}
```

**关键差异：**
1. **没有类**: Go使用结构体和方法
2. **接收者**: 方法通过接收者定义
3. **值vs指针**: 值接收者不修改原对象，指针接收者可以
4. **可见性**: 大写字母开头=公开，小写=私有

## 七、接口

**PHP版本：**
```php
<?php
interface Logger {
    public function log($message);
}

class FileLogger implements Logger {
    public function log($message) {
        file_put_contents('log.txt', $message);
    }
}

class ConsoleLogger implements Logger {
    public function log($message) {
        echo $message;
    }
}
```

**Go版本：**
```go
package main

import "fmt"

// 定义接口
type Logger interface {
    Log(message string)
}

// 实现接口（隐式实现，无需显式声明）
type FileLogger struct{}

func (f FileLogger) Log(message string) {
    fmt.Printf("File: %s\n", message)
}

type ConsoleLogger struct{}

func (c ConsoleLogger) Log(message string) {
    fmt.Printf("Console: %s\n", message)
}

// 使用接口
func process(logger Logger, message string) {
    logger.Log(message)
}

func main() {
    fileLogger := FileLogger{}
    consoleLogger := ConsoleLogger{}
    
    process(fileLogger, "Error occurred")
    process(consoleLogger, "Info message")
}
```

**关键差异：**
1. **隐式实现**: Go的接口是隐式实现的（鸭子类型）
2. **接口组合**: 可以组合多个接口
3. **空接口**: `interface{}`可以表示任何类型

## 八、错误处理

**PHP版本：**
```php
<?php
function divide($a, $b) {
    if ($b == 0) {
        throw new Exception("除数不能为0");
    }
    return $a / $b;
}

try {
    $result = divide(10, 0);
} catch (Exception $e) {
    echo "错误: " . $e->getMessage();
}
```

**Go版本：**
```go
package main

import (
    "errors"
    "fmt"
)

// Go的错误处理：返回error类型
func divide(a, b float64) (float64, error) {
    if b == 0 {
        return 0, errors.New("除数不能为0")
    }
    return a / b, nil
}

func main() {
    result, err := divide(10, 0)
    if err != nil {
        fmt.Println("错误:", err)
        return
    }
    fmt.Println("结果:", result)
    
    // 更简洁的错误处理
    if result, err := divide(10, 2); err != nil {
        fmt.Println("错误:", err)
    } else {
        fmt.Println("结果:", result)
    }
}
```

**关键差异：**
1. **无异常**: Go没有try-catch，使用error返回值
2. **显式处理**: 必须显式检查错误
3. **错误即值**: error是普通值，不是异常

## 九、并发编程（Go的强项）

**PHP版本（使用Swoole）：**
```php
<?php
// Swoole协程
go(function () {
    echo "协程1\n";
});

go(function () {
    echo "协程2\n";
});
```

**Go版本：**
```go
package main

import (
    "fmt"
    "time"
)

func main() {
    // Goroutine（轻量级线程）
    go sayHello("World")
    go sayHello("Go")
    
    // 等待goroutine完成
    time.Sleep(1 * time.Second)
    
    // 使用Channel通信
    ch := make(chan string)
    
    go func() {
        ch <- "Hello from goroutine"
    }()
    
    message := <-ch
    fmt.Println(message)
}

func sayHello(name string) {
    for i := 0; i < 3; i++ {
        fmt.Printf("Hello %s\n", name)
        time.Sleep(100 * time.Millisecond)
    }
}
```

**Channel示例：**
```go
package main

import "fmt"

func main() {
    // 创建channel
    ch := make(chan int, 2)  // 缓冲channel
    
    // 发送数据
    ch <- 1
    ch <- 2
    
    // 接收数据
    fmt.Println(<-ch)  // 1
    fmt.Println(<-ch)  // 2
    
    // 关闭channel
    close(ch)
}
```

**关键差异：**
1. **Goroutine**: Go的并发单元，比线程更轻量
2. **Channel**: 用于goroutine间通信
3. **内置支持**: 并发是Go的一等公民

## 十、包管理

### Go Modules（类似Composer）

**PHP (Composer):**
```json
{
    "require": {
        "monolog/monolog": "^2.0"
    }
}
```

**Go (go.mod):**
```go
module myproject

go 1.21

require (
    github.com/gin-gonic/gin v1.9.1
)
```

**常用命令：**
```bash
# 初始化模块
go mod init myproject

# 添加依赖
go get github.com/gin-gonic/gin

# 整理依赖
go mod tidy

# 查看依赖
go list -m all
```

## 十一、常用标准库

### HTTP服务器

**PHP版本：**
```php
<?php
// 使用Swoole
$http = new Swoole\Http\Server("0.0.0.0", 9501);
$http->on("request", function ($request, $response) {
    $response->header("Content-Type", "text/plain");
    $response->end("Hello World");
});
$http->start();
```

**Go版本：**
```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hello World")
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

### JSON处理

**PHP版本：**
```php
<?php
$data = ['name' => 'Cann', 'age' => 30];
$json = json_encode($data);
$decoded = json_decode($json, true);
```

**Go版本：**
```go
package main

import (
    "encoding/json"
    "fmt"
)

type Person struct {
    Name string `json:"name"`
    Age  int    `json:"age"`
}

func main() {
    // 编码
    p := Person{Name: "Cann", Age: 30}
    jsonData, _ := json.Marshal(p)
    fmt.Println(string(jsonData))
    
    // 解码
    var p2 Person
    json.Unmarshal(jsonData, &p2)
    fmt.Println(p2)
}
```

## 十二、最佳实践

### 1. 代码组织

```
project/
├── main.go
├── go.mod
├── handlers/
│   └── user.go
├── models/
│   └── user.go
└── utils/
    └── helper.go
```

### 2. 命名规范

- **公开**: 大写字母开头（`User`, `GetUser`）
- **私有**: 小写字母开头（`user`, `getUser`）
- **包名**: 小写，简短

### 3. 错误处理

```go
// 好的做法
result, err := someFunction()
if err != nil {
    return err  // 向上传播错误
}

// 避免忽略错误
result, _ := someFunction()  // 不推荐
```

### 4. 性能优化

- 使用指针避免大结构体拷贝
- 合理使用goroutine和channel
- 注意slice的容量预分配

## 十三、学习路径建议

1. **基础语法** (1-2周)
   - 变量、函数、结构体
   - 控制流（if/for/switch）
   - 数组、切片、map

2. **进阶特性** (2-3周)
   - 接口和类型断言
   - 错误处理
   - 包管理

3. **并发编程** (2-3周)
   - Goroutine
   - Channel
   - 并发模式

4. **实战项目** (持续)
   - Web API开发
   - 微服务
   - CLI工具

## 十四、推荐资源

- **官方文档**: https://go.dev/doc/
- **Go by Example**: https://gobyexample.com/
- **Effective Go**: https://go.dev/doc/effective_go
- **Go语言圣经**: https://gopl-zh.github.io/

## 总结

Go语言相比PHP的主要特点：
- ✅ **静态类型**: 编译时发现错误
- ✅ **简洁语法**: 代码更清晰
- ✅ **并发支持**: 内置goroutine和channel
- ✅ **性能优秀**: 编译型语言，执行效率高
- ✅ **部署简单**: 单一可执行文件

