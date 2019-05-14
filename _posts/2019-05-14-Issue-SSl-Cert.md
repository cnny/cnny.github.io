---
layout:     post
title:      "基于OpenSSL自建CA和颁发SSL证书"
subtitle:   ""
date:       2019-05-14 16:11:00
author:     "Cann"
header-img: "img/music-02.jpg"
tags:
    - HTTPS
    - SSL
---



openssl是一个开源程序的套件、这个套件有三个部分组成：一是`libcryto`，这是一个具有通用功能的加密库，里面实现了众多的加密库；二是`libssl`，这个是实现ssl机制的，它是用于实现TLS/SSL的功能；三是openssl，是个多功能命令行工具，它可以实现加密解密，甚至还可以当CA来用，可以让你创建证书、吊销证书。

默认情况ubuntu和CentOS上都已安装好openssl。CentOS 6.x 上有关ssl证书的目录结构：

```bash
/etc/pki/CA/
            newcerts    存放CA签署（颁发）过的数字证书（证书备份目录）
            private     用于存放CA的私钥
            crl         吊销的证书

/etc/pki/tls/
             cert.pem    软链接到certs/ca-bundle.crt
             certs/      该服务器上的证书存放目录，可以房子自己的证书和内置证书
                   ca-bundle.crt    内置信任的证书
             private    证书密钥存放目录
             openssl.cnf    openssl的CA主配置文件
```

# 1. 颁发证书

## 1.1 修改CA的一些配置文件

CA要给别人颁发证书，首先自己得有一个作为根证书，我们得在一切工作之前修改好CA的配置文件、序列号、索引等等。

```bash
vi /etc/pki/tls/openssl.cnf
```

```bash
...
[ CA_default ]

dir             = /etc/pki/CA           # Where everything is kept
certs           = $dir/certs            # Where the issued certs are kept
crl_dir         = $dir/crl              # Where the issued crl are kept
database        = $dir/index.txt        # database index file.
#unique_subject = no                    # Set to 'no' to allow creation of
                                        # several ctificates with same subject.
new_certs_dir   = $dir/newcerts         # default place for new certs.

certificate     = $dir/cacert.pem       # The CA certificate
serial          = $dir/serial           # The current serial number
crlnumber       = $dir/crlnumber        # the current crl number
                                        # must be commented out to leave a V1 CRL
crl             = $dir/crl.pem          # The current CRL
private_key     = $dir/private/cakey.pem # The private key
RANDFILE        = $dir/private/.rand    # private random number file
...
default_days    = 3650                  # how long to certify for
...
# For the CA policy
[ policy_match ]
countryName             = match
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
...
[ req_distinguished_name ]
countryName                     = Country Name (2 letter code)
countryName_default             = CN
countryName_min                 = 2
countryName_max                 = 2

stateOrProvinceName             = State or Province Name (full name)
stateOrProvinceName_default     = GD
...
[ req_distinguished_name ] 部分主要是颁证时一些默认的值，可以不动
```

[配置文件中各个配置项的含义](https://www.cnblogs.com/aixiaoxiaoyu/p/8400036.html)	

一定要注意`[ policy_match ]`中的设定的匹配规则，是有可能因为证书使用的工具不一样，导致即使设置了csr中看起来有相同的countryName,stateOrProvinceName等，但在最终生成证书时依然报错：

```bash
Using configuration from /usr/lib/ssl/openssl.cnf
Check that the request matches the signature
Signature ok
The stateOrProvinceName field needed to be the same in the
CA certificate (GuangDong) and the request (GuangDong)
```

在CA目录下创建两个初始文件：

```Bash
# touch index.txt serial
# echo 01 > serial
```

## 1.2 生成根密钥

```bash
# cd /etc/pki/CA/
# openssl genrsa -out private/cakey.pem 2048
```

为了安全起见，修改cakey.pem私钥文件权限为600或400，也可以使用子shell生成`( umask 077; openssl genrsa -out private/cakey.pem 2048 )`，下面不再重复。

## 1.3 生成根证书

使用req命令生成自签证书

```bash
# openssl req -new -x509 -key private/cakey.pem -out cacert.pem
```

会提示输入一些内容，因为是私有的，所以可以随便输入（之前修改的openssl.cnf会在这里呈现），最好记住能与后面保持一致。上面的自签证书`cacert.pem`应该生成在`/etc/pki/CA`下。

## 1.4 为我们的nginx web服务器生成ssl密钥

以上都是在CA服务器上做的操作，而且只需进行一次，现在转到nginx服务器上执行：

```Bash
# cd /etc/nginx/ssl
# openssl genrsa -out nginx.key 2048
```

这里测试的时候CA中心与要申请证书的服务器是同一个。

## 1.5 为nginx生成证书签署请求

```bash
# openssl req -new -key nginx.key -out nginx.csr
...
Country Name (2 letter code) [AU]:CN
State or Province Name (full name) [Some-State]:GD
Locality Name (eg, city) []:SZ
Organization Name (eg, company) [Internet Widgits Pty Ltd]:COMPANY
Organizational Unit Name (eg, section) []:IT_SECTION
Common Name (e.g. server FQDN or YOUR name) []:your.domain.com
Email Address []:

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:
An optional company name []:
...
```

同样会提示输入一些内容，其它随便，除了`Commone Name`一定要是你要授予证书的服务器域名或主机名，challenge password不填。

## 1.6 私有CA根据请求来签署证书

接下来要把上一步生成的证书请求csr文件，发到CA服务器上，在CA上执行：

```bash
# openssl ca -in nginx.csr -out nginx.crt

另外在极少数情况下，上面的命令生成的证书不能识别，试试下面的命令：
# openssl x509 -req -in server.csr -CA /etc/pki/CA/cacert.pem -CAkey /etc/pki/CA/private/cakey.pem -CAcreateserial -out server.crt
```

上面签发过程其实默认使用了`-cert cacert.pem -keyfile cakey.pem`，这两个文件就是前两步生成的位于`/etc/pki/CA`下的根密钥和根证书。将生成的crt证书发回nginx服务器使用。

到此我们已经拥有了建立ssl安全连接所需要的所有文件，并且服务器的crt和key都位于配置的目录下，剩下的是如何使用证书的问题。

# 2. 使用 ssl 证书

## 1.1 Nginx 使用 ssl 证书

以 Nginx 为例，在 Nginx 中新建ssl文件夹，将生成的crt和key放入其中，Vhost server中加入以下代码：

```bash
        listen 443 ssl http2;

        ssl on;
        ssl_certificate /usr/local/nginx/conf/ssl/test.crt; // 指向 ssl 文件夹中的 crt 文件
        ssl_certificate_key /usr/local/nginx/conf/ssl/test.crt; // 指向 ssl 文件夹中的 key 文件
        ssl_session_timeout 5m;
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
        ssl_ciphers AESGCM:ALL:!DH:!EXPORT:!RC4:+HIGH:!MEDIUM:!LOW:!aNULL:!eNULL;
        ssl_prefer_server_ciphers on;
```

这样，我们就可以用 https 发起请求了。

但是，我们自己签发的证书，是不受其他服务器信任的，当发起 curl 请求时，会出现以下情况：

```Bash
$ curl https://115.159.58.121:10086/api/user
curl: (60) SSL certificate problem: Invalid certificate chain
More details here: https://curl.haxx.se/docs/sslcerts.html

curl performs SSL certificate verification by default, using a "bundle"
 of Certificate Authority (CA) public keys (CA certs). If the default
 bundle file isn't adequate, you can specify an alternate file
 using the --cacert option.
If this HTTPS server uses a certificate signed by a CA represented in
 the bundle, the certificate verification probably failed due to a
 problem with the certificate (it might be expired, or the name might
 not match the domain name in the URL).
If you'd like to turn off curl's verification of the certificate, use
 the -k (or --insecure) option.
```

这时候，我们就需要将我们 CA 服务器的根证书导入到这台服务器中。

## 1.2 添加 CA 根证书到操作系统获得信任

首先将 CA 服务器中的根证书 cacert.pem 下载至本地根目录

### Mac OS

##### 添加证书：

```
sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /cacert.pem
```

##### 移除证书：

```
sudo security delete-certificate -c "gd" // 这里是填证书名称，而不是证书文件名称
```

除了用命令行管理证书，还可以在 `钥匙串访问`中进行管理

![image-20190514171111234](/img/posts/image-20190514171111234-7835763.png)

### Windows

#### 添加证书：

```
certutil -addstore -f "ROOT" cacert.pem
```

#### 移除证书:

```
certutil -delstore "ROOT" gd
```

#### Linux (CentOs 6)

#### 添加证书：

```
#安装 ca-certificates package:

yum install ca-certificates

#启用dynamic CA configuration feature:

update-ca-trust force-enable

#将证书文件放到 /etc/pki/ca-trust/source/anchors/ 目录下

mv /cacert.pem /etc/pki/ca-trust/source/anchors/

#执行:

update-ca-trust extract
```

