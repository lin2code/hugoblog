---
title: Nginx使用笔记
date: 2020-09-29T16:43:17+08:00
lastmod: 2020-09-29T16:43:17+08:00
author: 林二狗
# authorlink: https://author.site
cover: /img/cover6.jpg
categories:
  - 笔记
tags:
  - nginx
  - 反向代理
  - wordpress
  - docker
  - https
# showcase: true
draft: false
---

## Nginx安装启动

1.先安装编译和支持https的组件

```bash
yum install gcc-c++
yum install -y pcre pcre-devel
yum install -y zlib zlib-devel
yum install -y openssl openssl-devel
```

2.官网下载.tar.gz安装包 执行以下命令安装

```bash
tar -zxvf nginx-xxx.tar.gz
cd nginx-xxx.0
#configure的参数用于添加https支持
bash ./configure --prefix=/usr/local/nginx --with-http_stub_status_module --with-http_ssl_module
make
make install
```

3.启动、停止nginx：

```bash
cd /usr/local/nginx/sbin/ 或 /usr/sbin/ #不同系统和用户安装路径不一样
./nginx #启动
./nginx -s stop #停止
./nginx -s quit #退出
./nginx -s reload #重载
```

---

## 添加开机自启动

在rc.local增加启动代码就可以  
`vim /etc/rc.local`  
增加一行 `/usr/local/nginx/sbin/nginx` 或 `/usr/sbin/nginx`  
设置执行权限 `chmod 755 /etc/rc.local`

---

## 全局生效nginx命令

```bash
vim /etc/profile

# 末尾添加
PATH=$PATH:/usr/local/nginx/sbin/nginx  #nginx启动文件路径  
export PATH
# 然后执行
source /etc/profile

```

---

## 配置文件

配置文件路径: `/usr/local/nginx/conf/nginx.conf` 或 `/etc/nginx/nginx.conf`

示例：

```nginx
#user  nobody;
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #tcp_nopush     on;
    #gzip  on;

    #keepalive_timeout  0;
    sendfile        on;
    keepalive_timeout  600; #超时时间
    client_max_body_size 20M; #单次上传数据缓冲区最大值
    underscores_in_headers on; #ctmd 不加这个header中有“_”会忽略

    #docker中运行WordPress时的配置
    server {
        listen 80;
        server_name xxxx.com; #绑定的域名 优先使用server_name和请求域名一致的配置

        access_log /usr/local/nginx/logs/wordpress.access.log; #该域名的请求日志配置
        set $node_port 18080; #变量 WordPress在docker中暴露的端口
        location / {
            #各种转发设置 避免WordPress中静态资源路径不是nginx代理的域名 导致加载不出来
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header Host $http_host;
            proxy_set_header X-NginX-Proxy true;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            #代理的目标地址 docker运行程序时 一定要用IP而不是localhost！！！
            proxy_pass http://127.0.0.1:$node_port$request_uri;
            proxy_redirect off;
            client_max_body_size 200m; #上传文件大小配置 覆盖全局配置
        }
    }

    #转发http到https的配置
    server {
        listen 80;
        server_name manage.xxxx.com;
        rewrite ^(.*)$ https://$host$1 permanent; #将所有http请求通过rewrite重定向到https
        location / {
            index index.html index.htm; #反向代理目标配置 这里是文件 因为重定向了所以不起作用
        }
    }

    #使用https的配置
    server {
        listen 443 ssl; #SSL协议端口号为443 此处如未添加ssl可能会造成Nginx无法启动。
        server_name manage.xxxx.com;
        ssl_certificate cert/manage.hoojoe.com.pem; #配置.pem证书文件所在路径
        ssl_certificate_key cert/manage.hoojoe.com.key; #配置.key证书密钥文件路径
        ssl_session_timeout 5m; #超时时间
        ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4; #使用此加密套件
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; #使用该协议进行配置
        ssl_prefer_server_ciphers on;
        #默认地址转发到 https://localhost:8010
        location / {
            proxy_pass https://localhost:8010;
        }
        #请求地址以api开头的转发到 https://localhost:8009
        location /api {
            rewrite  ^/api/(.*)$ /api/$1 break; #使用rewrite实现 反正就这样写
            proxy_pass https://localhost:8009;
            #如果后端想获取客户的IP 需要设置这些请求header
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        }
        location /hangfire {
            rewrite  ^/hangfire/(.*)$ /hangfire/$1 break;
            proxy_pass https://localhost:8011;
        }
    }
}
```
