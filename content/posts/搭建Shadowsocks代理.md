---
title: 搭建Shadowsocks代理
date: 2020-09-30T15:44:59+08:00
lastmod: 2020-09-30T15:44:59+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover9.jpg
categories:
  - 奇计
tags:
  - shadowsocks
  - 代理
  - 翻墙
# showcase: true
draft: false
---

## 买一台访问google不是404服务器

新建一个json文件放到服务器上

```json
{
    "server":"0.0.0.0",
    "server_port":12345,
    "password":"pwd",
    "timeout":800,
    "method":"aes-256-cfb",
    "fast_open": false,
    "workers": 2
}

```

server_port：客户端和服务器通信的端口  
password：密码  
method：python版服务端就填aes-256-cfb  

还可以添加两个配置用于代理服务器上的程序，只通过客户端使用则不用配置

```json
"local_address": "127.0.0.1",
"local_port":1080,
```

## 安装python和SS

```bash
#ss使用python2或3都可以
yum install python36
pip install shadowsocks
#启动ss
ssserver -c /path/xxx.json -d start
```

启动后报错：  
AttributeError: /usr/lib/x86_64-linux-gnu/libcrypto.so.1.1: undefined symbol: EVP_CIPHER_CTX_cleanup

原因：  
由于在openssl 1.1.0中废弃了EVP_CIPHER_CTX_cleanup()函数而引入EVE_CIPHER_CTX_reset()函数所导致的：

解决方法：

1. 根据错误信息定位到文件`/usr/local/lib/python3.6/site-packages/shadowsocks/crypto/openssl.py`
2. 搜索 cleanup 并将其替换为 reset
3. 重新启动shadowsocks

## 其它

[windows客户端下载](https://github.com/shadowsocks/shadowsocks-windows/releases)
