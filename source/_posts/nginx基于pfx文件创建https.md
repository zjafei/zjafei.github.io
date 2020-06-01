---
title: nginx基于pfx文件创建https
date: 2020-06-01 22:05:41
tags: ['nginx','pfx','https']
category: 'coding'
---

# 什么是pfx文件
公钥加密来技术12号标准（Public Key Cryptography Standards #12，PKCS#12）为存储和传输用户或服务器私钥、公钥和证书指源定了一个可移植的格式2113。它是一种二进制格5261式，这些文件也称为PFX文件。开发人员通常需要将PFX文件转换为某些不同的格式，如PEM或JKS，以便可以为使用SSL通信的独立Java客户4102端或WebLogic Server使用。<!--more-->

# 证书的类型

  * 带有私钥的证书
    由Public Key Cryptography Standards #12，PKCS#12标准定义，包含了公钥和私钥的二进制格式的证书形式，以pfx作为证书文件后缀名。
  * 二进制编码的证书
    证书中没有私钥，DER 编码二进制格式的证书文件，以cer作为证书文件后缀名。
  * Base64编码的证书
    证书中没有私钥，BASE64 编码格式的证书文件，也是以cer作为证书文件后缀名。

  > 所以pfx的最大特点就是同时包含公钥和私钥


# 通过pfx生成常用证书的方法

```bash
openssl pkcs12 -in /path/to/in/file.pfx -clcerts -nokeys -out /path/to/out/file.crt
openssl pkcs12 -in /path/to/in/file.pfx -nocerts -nodes -out /path/to/out/file.rsa
##验证证书正确性
openssl s_server -www -accept 443 -cert /path/to/file.crt -key /path/to/file.rsa

openssl pkcs12 -in /path/to/in/file.pfx -nodes -out /path/to/out/file.pem
openssl rsa -in /path/to/in/file.pem -out /path/to/out/file.key
openssl x509 -in /path/to/in/file.pem -out /path/to/out/file.crt

```

# 不同证书的nginx的配置

```nginx
server {  
    listen 443;  
    server_name localhost;
    ssl on;  
    ssl_certificate /path/to/out/file.crt;  
    ssl_certificate_key /path/to/out/file.rsa;  
    ssl_session_timeout 5m;  
    ssl_protocols SSLv2 SSLv3 TLSv1;  
    ssl_ciphers ALL:!ADH:!EXPORT56:RC4+RSA:+HIGH:+MEDIUM:+LOW:+SSLv2:+EXP;  
    ssl_prefer_server_ciphers on;  
    # location仅为事例
    location ~ /api/(.*) {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Ssl on;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://serverAPI;
    }
}
```

***如果nginx报错***
```bash
SSL: error:0906D06C:PEM routines:PEM_read_bio:no start line:Expecting: TRUSTED CERTIFICATE
```
可以尝试使用`pem证书`
```nginx
server {  
    listen 443;  
    server_name localhost;
    ssl_certificate     /path/to/out/file.pem;
    ssl_certificate_key /path/to/out/file.key;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;   
    # location仅为事例
    location ~ /api/(.*) {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Ssl on;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://serverAPI;
    }
}
```
这时候启动nginx会出现`Enter PEM pass phrase`

```bash
openssl rsa -in /path/to/in/file.key -out /path/to/out/file.unsecure
```
修改nginx配置
```nginx
server {  
    listen 443;  
    server_name localhost;
    ssl_certificate     /path/to/file.pem;
    ssl_certificate_key /path/to/file.unsecure;
    ssl_protocols       TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers         HIGH:!aNULL:!MD5;   
    # location仅为事例
    location ~ /api/(.*) {
            proxy_redirect off;
            proxy_set_header Host $host;
            proxy_set_header X-Ssl on;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_pass http://serverAPI;
    }
}
```
这样就不需要每次启动nginx输入PEM pass phrase

>最后祝大家儿童节快乐 :)