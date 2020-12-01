---
title: 使用Docker部署WordPress+Mysql
date: 2020-09-08T18:44:41+08:00
lastmod: 2020-09-08T18:44:41+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover3.jpg
categories:
  - 折腾
tags:
  - Docker
  - WordPress
  - Nginx
# showcase: true
draft: false
---

## 1. 购买服务器注册域名

## 2. 登录系统先`yum update`一下

## 3. 安装docker

centos 8先执行以下命令安装一个containerd.io  
`yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm`

否则直接执行以下命令安装docker  
`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

启动docker： `systemctl start docker`

## 4. 安装python和docker-compose

`yum install python3`
`pip3 install docker-compose`

## 5. 在/data/wordpress目录下创建一个docker-compose.yml

```yml
version: '3.1'
services:
  db:
    image: mysql
    restart: always
    command: --default-authentication-plugin=mysql_native_password
    environment:
      MYSQL_DATABASE: wordpress
      MYSQL_USER: wordpress
      MYSQL_PASSWORD: wordpress
      MYSQL_RANDOM_ROOT_PASSWORD: '1'
    volumes:
      - db:/var/lib/mysql

  wordpress:
    image: wordpress
    restart: always
    depends_on:
      - db
    ports:
      - 18080:80
    environment:
      WORDPRESS_DB_HOST: db
      WORDPRESS_DB_USER: wordpress
      WORDPRESS_DB_PASSWORD: wordpress
      WORDPRESS_DB_NAME: wordpress
    volumes:
      - wordpress:/var/www/html
volumes:
  wordpress:
  db:

```

## 6. 在/data/wordpress目录下

* 先执行`docker-compose up -d db`启动和初始化数据库，启动起了可以`docker exec -it db的id /bin/bash`进去用mysql命令检查一下用户是否创建好了  

* 再执行`docker-compose up -d wordpress`启动应用

## 7. 安装nginx

自行百度

## 8. 配置nginx

加入这段，注意proxy_pass里一定要写ip

```nginx
server {
  listen 80;
  server_name hoojoe.com;

  #charset koi8-r;
  access_log /usr/local/nginx/logs/wordpress.access.log;
  set $node_port 18080;
  location / {
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_set_header X-NginX-Proxy true;
    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_pass http://127.0.0.1:$node_port$request_uri;
    proxy_redirect off;
  }
}
```

## 9. reload nginx后，确保各个镜像正常运行后，在浏览器中输入网址访问

God bless you.
