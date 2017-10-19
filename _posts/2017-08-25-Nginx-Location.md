---
layout:     post
title:      "Nginx Location"
subtitle:   ""
date:       2017-08-25 19:41:00
author:     "Cann"
header-img: "img/post-php-cs-fixer.jpg"
tags:
    - Nginx
---

## location

#### 一：正则写法

##### a 精确匹配，所匹配路径完全相同

```
    location  = / {
      # 精确匹配 / ，主机名后面不能带任何字符串
      [ configuration A ]
    }
```

##### b 对URL路径进行前缀匹配

```
    location /documents/ {
      # 匹配任何以 /documents/ 开头的地址，匹配符合以后，还要继续往下搜索
      # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
      [ configuration C ]
    }
```

##### c 正则匹配

```
    location ~ /documents/Abc {
      # 匹配任何以 /documents/Abc 开头的地址，匹配符合以后，还要继续往下搜索
      # 只有后面的正则表达式没有匹配到时，这一条才会采用这一条
      [ configuration CC ]
    }
```

##### d 对URL路径进行前缀匹配，并且在正则之前。（即该匹配成功后，不会再进行后续的正则匹配）

```
location ^~ /images/ {
  # 匹配任何以 /images/ 开头的地址，匹配符合以后，停止往下搜索正则，采用这一条。
  [ configuration D ]
}
```

##### e 不区分大小写的正则匹配

```
location ~* \.(gif|jpg|jpeg)$ {
  # 匹配所有以 gif,jpg或jpeg 结尾的请求
  # 然而，所有请求 /images/ 下的图片会被 config D 处理，因为 ^~ 到达不了这一条正则
  [ configuration E ]
}
```

#### 二 ：Location匹配优先级

a. **第一优先级：等号类型（=）的优先级最高。一旦匹配成功，则不再查找其他匹配项。**

b.  **第二优先级：^~类型表达式。一旦匹配成功，则不再查找其他匹配项。**

c.  **第三优先级：正则表达式类型（~ ~*）的优先级次之。如果有多个location的正则能匹配的话，则使用正则表达式最长的那个。**

d.  **第四优先级：常规字符串匹配类型。按前缀匹配。**

### Rewrite

#### 一：flag标志位

a. `last` : **相当于Apache的[L]标记，表示完成rewrite**

b. `break `: **停止执行当前虚拟主机的后续rewrite指令集**

>last一般写在server和if中，而break一般使用在location中
>last不终止重写后的url匹配，即新的url会再从server走一遍匹配流程，而break终止重写后的匹配
>break和last都能组织继续执行后面的rewrite指令

c. `redirect` : **返回302临时重定向，地址栏会显示跳转后的地址**

d. `permanent `:  **返回301永久重定向，地址栏会显示跳转后的地址**

>因为301和302不能简单的只返回状态码，还必须有重定向的URL，这就是return指令无法返回301,302的原因了。

#### 二：全局变量

a. `$args` ： **这个变量等于请求行中的参数，同$query_string**

b. `$content_length` ： **请求头中的Content-length字段。**

c. `$content_type` ： **请求头中的Content-Type字段。**

d. `$document_root` ： **当前请求在root指令中指定的值。**

e. `$host` ： **请求主机头字段，否则为服务器名称。**

f. `$http_user_agent` ： **客户端agent信息**

g. `$http_cookie` ： **客户端cookie信息**

h. `$limit_rate` ： **这个变量可以限制连接速率。**

i. `$request_method` ： **客户端请求的动作，通常为GET或POST。**

j. `$remote_addr` ： **客户端的IP地址。**

k. `$remote_port` ： **客户端的端口。**

l. `$remote_user` ： **已经经过Auth Basic Module验证的用户名。**

m. `$request_filename` ： **当前请求的文件路径，由root或alias指令与URI请求生成。**

n. `$scheme` ： **HTTP方法（如http，https）。**

o. `$server_protocol` ： **请求使用的协议，通常是HTTP/1.0或HTTP/1.1。**

p. `$server_addr` ： **服务器地址，在完成一次系统调用后可以确定这个值。**

q. `$server_name` ： **服务器名称。**

r. `$server_port` ： **请求到达服务器的端口号。**

s. `$request_uri` ： **包含请求参数的原始URI，不包含主机名，如：”/foo/bar.php?arg=baz”。**

t. `$uri` ： **不带请求参数的当前URI，$uri不包含主机名，如”/foo/bar.html”。**

u. `$document_uri` ： **与$uri相同。例：http://localhost:88/test1/test2/test.php**

v. `$host` ： **localhost**

w. `$server_port` ： **88**

x. `$request_uri` ： **http://localhost:88/test1/test2/test.php**

y. `$document_uri` ： **/test1/test2/test.php**

z. `$document_root` ： **/var/www/html**

A.  `$request_filename` ： **/var/www/html/test1/test2/test.php**

#### 三. try_files

>try_files指令是按顺序检测文件的存在性,并且返回第一个找到文件的内容,如果第一个找不到就会自动找第二个,依次查找.其实现的是内部跳转

**案例1**

```
server {
   listen 8000;
   server_name 121.10.143.66;
   root html;
   index index.html index.php;

   location /abc {
       try_files /4.html /5.html @qwe;  ## 检测文件4.html和5.html,如果存在正常显示,不存在就去查找@qwe值
  }

   location @qwe  {
      rewrite ^/(.*)$   http://www.baidu.com;  ## 跳转到百度页面
}
```

**案例2**

```
server {
   listen 8000;
   server_name 121.10.143.66;
   root html;
   index index.php index.html;

   location /abc {
       try_files $uri /index.php/$uri;  ## 若$uri不存在，则跳转至/index.php/$uri
}
```
>`$uri`： 当前请求的url。

#### 与rewrite指令不同，如果回退URI不是命名的location那么`$args`不会自动保留，如果你想保留`$args`，则必须明确声明。

```
try_files $uri $uri/ /index.php?q=$uri&$args;
```

### 参考文章

- [main](http://blog.kelu.org/tech/2017/05/03/nginx-conf-location-and-rewrite.html)
- [location 优先级](http://www.bo56.com/nginx-location%E5%9C%A8%E9%85%8D%E7%BD%AE%E4%B8%AD%E7%9A%84%E4%BC%98%E5%85%88%E7%BA%A7)
- [try_files](https://www.hi-linux.com/posts/53878.html)



