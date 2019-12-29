---
title: 我的 HTTPS 升级之旅
date: 2017-05-11 21:47:36
tags:
- HTTP
categories:
- 网络
---
> 本文将介绍我是如何将一个 HTTP 网站升级到 HTTPS。系统环境：CentOS 7.0 + Nginx 1.12.0

# 前言
先贴一个福利，也作为没有启用 HTTPS 的反面教材：

![福利.png](https://mares.oss-cn-qingdao.aliyuncs.com/blog/https-upgrade/1.png)
这是我参与开发过的一个外包网站，没有启用 HTTPS，网站页面被中间人劫持，并插入了一些奇怪的东西。下面是正文。

# 预备条件
本文假定你已经拥有一个正确解析到服务器IP的域名，服务器上已安装 Nginx。Nginx 可以通过源码编译安装，也可以使用系统提供的包管理器安装，比如 RH 系的 yum 或者 Debian 系的 apt-get，具体步骤自行Google。

# 获取证书
HTTPS 证书分三类：1. DV 域名验证证书 2. OV 组织机构验证证书 3. EV 增强的组织机构验证证书。每类证书的审核要求不同，在浏览器地址栏也会有区分，对于个人网站而言，使用免费的 DV 证书就足够了。

我使用了大名鼎鼎的 Let's Encrypt 来生成证书。
## 1. 安装 certbot
certbot 是 Let's Encrypt 提供的一套自动化工具。 
```
yum install epel-release
yum install certbot
```
## 2. 生成证书
这里采用 webroot 作为 Let's Encrypt 的认证方式。
```
certbot certonly -a webroot --webroot-path=/your/project/path -d example.com -d www.example.com
```
webroot-path就是你的项目路径，使用 -d 可以添加多个域名。这时证书就已经生成成功了，默认保存在 /etc/letsencrypt/live/example.com/ 下。证书文件包括：
- cert.pem: 服务端证书
- chain.pem: 浏览器需要的所有证书但不包括服务端证书，比如根证书和中间证书
- fullchain.pem: 包括了cert.pem和chain.pem的内容
- privkey.pem: 证书私钥

## 3. 生成迪菲-赫尔曼密钥交换组（ Strong Diffie-Hellman Group）
为了进一步提高安全性，你也可以生成一个 Strong Diffie-Hellman Group。
```
openssl dhparam -out /etc/ssl/certs/dhparam.pem 2048
```
如果没有安装 openssl，这里要先安装一下。
```
yum install openssl
```


# 配置 Nginx
编辑 Nginx 配置文件，如果你不知道配置文件在哪，可以用 locate /nginx.conf 命令查找。添加以下内容，具体参数以你的实际情况为准。
```
server {
    listen 443 ssl;
    # 启用http2
    # 需要安装 Nginx Http2 Module
    # listen 443 http2 ssl;
    server_name example.com www.example.com;
    #证书文件
    ssl_certificate /etc/letsencrypt/live/example.com/fullchain.pem;
    #私钥文件
    ssl_certificate_key /etc/letsencrypt/live/example.com/privkey.pem;

    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    # 优先采取服务器算法
    ssl_prefer_server_ciphers on;
    # 定义算法
    ssl_ciphers "EECDH+AESGCM:EDH+AESGCM:AES256+EECDH:AES256+EDH";
    ssl_ecdh_curve secp384r1;
    ssl_session_cache shared:SSL:10m;
    ssl_session_tickets off;
    ssl_stapling on;
    ssl_stapling_verify on;
    resolver 8.8.8.8 8.8.4.4 valid=300s;
    resolver_timeout 5s;
   
    add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
    add_header X-Frame-Options DENY;
    add_header X-Content-Type-Options nosniff;
    # 使用DH文件
    ssl_dhparam /etc/ssl/certs/dhparam.pem;

    location ~ /.well-known {
        allow all;
    }

    root /your/project/path;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ =404;
    }
}
```
解释一下其中几项配置：
```
ssl_stapling on;
```
开启 OCSP Stapling，使服务端主动获取 OCSP 查询结果并随着证书一起发送给客户端，从而让客户端跳过自己去验证的过程，提高 TLS 握手效率。
```
add_header Strict-Transport-Security "max-age=63072000; includeSubdomains";
```
启用 HSTS 策略，强制浏览器使用 HTTPS 连接，max-age设置单位时间内強制使用 HTTPS 连接；includeSubDomains 可选，设置所有子域同时生效。浏览器在获取该响应头后，在 max-age 的时间内，如果遇到 HTTP 连接，就会通过 307 跳转強制使用 HTTPS 进行连接
```
    add_header X-Frame-Options DENY;
```
添加 X-Frame-Options 响应头，可以禁止网站被嵌入到 iframe 中，减少[点击劫持 (clickjacking)攻击](https://blogs.msdn.microsoft.com/ie/2009/01/27/ie8-security-part-vii-clickjacking-defenses/)。
```
    add_header X-Content-Type-Options nosniff;
```
添加 X-Content-Type-Options 响应头，防止 [MIME 类型嗅探攻击](https://developer.mozilla.org/en-US/docs/Web/HTTP/Basics_of_HTTP/MIME_types#MIME_sniffing)

解释完毕。继续，来测试 nginx.conf 是否有语法错误
```
nginx -t
```
重启 Nginx
```
nginx -s reload
```
# 重定向 HTTP 到 HTTPS
修改原来 HTTP 网站的 Nginx 配置。
```
server {
    listen 80;
    server_name example.com www.example.com;
    access_log /var/log/example/access.log;
    error_log /var/log/example/error.log;
    # 301 永久重定向
    return 301 https://$host$request_uri;
    location / {
        root /your/project/path;
        index index.html index.htm;
    }
}
```
这时再访问网站，浏览器地址栏就会出现一把小锁。
![https.png](https://mares.oss-cn-qingdao.aliyuncs.com/blog/https-upgrade/2.png)

一旦升级 HTTPS，网站内的所有资源文件和请求的协议也必须为 HTTPS，你需要在前端代码里修改一下。

最后可以使用 [ssllabs](https://www.ssllabs.com/ssltest/analyze.html) 测试一下网站的安全性，我的网站得了 A+😁

---------
参考链接：
1. [Nginx 配置 HTTPS 服务器](https://aotu.io/notes/2016/08/16/nginx-https/)
2. [How To Secure Nginx with Let's Encrypt on CentOS 7](https://www.digitalocean.com/community/tutorials/how-to-secure-nginx-with-let-s-encrypt-on-centos-7)