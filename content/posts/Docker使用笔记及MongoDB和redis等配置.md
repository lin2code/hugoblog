---
title: Docker使用笔记及MongoDB和redis等配置
date: 2020-09-29T18:03:37+08:00
lastmod: 2020-09-29T18:03:37+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover7.jpg
categories:
  - 笔记
tags:
  - docker
  - mongodb
  - redis
# showcase: true
draft: false
---

## 安装docker

* 使用脚本自动安装docker

CentOS 8先安装 containerd.io  
`yum install https://download.docker.com/linux/fedora/30/x86_64/stable/Packages/containerd.io-1.2.6-3.3.fc30.x86_64.rpm`

docker安装脚本  
`curl -fsSL https://get.docker.com | bash -s docker --mirror Aliyun`

因为Centos8默认使用podman代替docker，所以需要先安装 containerd.io 再执行以上语句否则报错

* 镜像加速设置

在/etc/docker目录下添加daemon.json文件  
内容：

```json
{
  "registry-mirrors": [
    "https://registry.docker-cn.com",
    "http://hub-mirror.c.163.com",
    "https://docker.mirrors.ustc.edu.cn",
    "https://reg-mirror.qiniu.com"
  ]
}
```

* 启动  
`systemctl daemon-reload`  
`systemctl restart docker`

* 开机启动  
`systemctl enable docker`

---

## 常用命令

```text
一、基本命令
docker version 查看docker版本
docker info 查看docker详细信息
docker --help 查看docker命令

二、镜像命令
docker images 查看docker镜像
docker images -a 列出本地所有的镜像
docker images -p 只显示镜像ID
docker images --digests 显示镜像的摘要信息
docker images --no-trunc 显示完整的镜像信息

三、容器命令。
docker run [OPTIONS] IMAGE 根据镜像新建并启动容器
OPTIONS说明：
 --name=“容器新名字”：为容器指定一个名称
 -d：后台运行容器，并返回容器ID，也即启动守护式容器
 -i：以交互模式运行容器，通常与-t同时使用
 -t：为容器重新分配一个伪输入终端，通常与-i同时使用
 -P：随机端口映射
 -p：指定端口映射，有以下四种格式：
  ip:hostPort:containerPort
  ip::containerPort
  hostPort:containerPort
  containerPort
docker ps 列出当前所有正在运行的容器
docker ps -a 列出所有的容器
docker ps -l 列出最近创建的容器
docker ps -n 3列出最近创建的3个容器
docker ps -q 只显示容器ID
docker ps --no-trunc 显示当前所有正在运行的容器完整信息
exit 退出并停止容器
Ctrl+p+q 只退出容器，不停止容器
docker start 容器ID或容器名称启动容器
docker restart 容器ID或容器名称重新启动容器
docker stop 容器ID或容器名称停止容器
docker kill 容器ID或容器名称强制停止容器
docker rm 容器ID或容器名称删除容器
docker rm -f 容器ID或容器名称强制删除容器
docker rm -f $(docker ps -a -q) 删除多个容器
docker logs -f -t --since --tail 容器ID或容器名称查看容器日志
如：docker logs -f -t --since=”2018-09-10” --tail=10 f9e29e8455a5
 -f : 查看实时日志
 -t : 查看日志产生的日期
 --since : 此参数指定了输出日志开始日期，即只输出指定日期之后的日志
 --tail=10 : 查看最后的10条日志
docker top 容器ID或容器名称查看容器内运行的进程
docker inspect 容器ID或容器名称查看容器内部细节
docker attach 容器ID进到容器内
docker exec 容器ID进到容器内
docker cp 容器ID:容器内的文件路径 宿主机路径从容器内拷贝文件到宿主机.
如：docker cp f9e29e8455a5:/tmp/yum.log /root
```

* 进入容器中的命令行 `docker exec -it 69d1 /bin/bash` 69d1是容器ID
* 查看卷 `docker volume ls`
* 查看卷详情 `docker volume inspect xxx`
* 删除卷 `docker volume rm xxx`
* 查看容器关联的卷 `docker inspect xxx_container_name | grep Mounts -A 10`
* 删除容器和卷 `docker rm -V xxx_container_name`
* 列出无用卷 `docker volume ls -qf dangling=true`

---

## 运行MongoDB  

* docker运行

```bash
docker pull mongo
docker run -itd --name mongodb -p 37017:27017 mongo --auth
docker exec -it mongodb mongo admin
```

注意 --auth 参数 启用数据库账号验证 如果创建容器的时候不启用 后续docker中很难修改

docker 集群部署和账号设置待研究

* 手动运行

配置yum安装源 `vim /etc/yum.repos.d/mongodb-org-4.2.repo` 写入以下内容

```conf
[mongodb-org-4.2]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/4.2/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-4.2.asc
```

url可替换为官网最新版本

`sudo yum install -y mongodb-org` 安装  
`sudo systemctl start mongod` 启动  
`sudo systemctl restart mongod` 重启  
`sudo systemctl enable mongod` 开机启动  

配置修改启用账号并允许远程连接

`vim /etc/mongod.conf`  
net.bindIp 修改值为 0.0.0.0允许所有IP连接  
security.authorization 修改值为 enabled  
重启

手动运行MongoDB建议以集群方式启动

* 初始化用户

mongo shell中执行命令（docker中使用命令进入容器的 mongo shell）

创建用户  
`db.createUser({ user: 'adminabc', pwd: 'abcd123', roles: [ { role: "root", db: "admin" } ] });`  

如果权限不够  
分配权限 `db.grantRolesToUser("adminabc", [ { role:"dbOwner", db:"abcdb"} ]);`  
或  
修改权限 `db.updateUser("adminabc",{roles:[ {role:"root",db:"admin"} ]})`  

robo3t中可以在admin数据库或其它数据库中右键users文件夹添加或删除用户  
应用应使用具体数据库中的user访问数据库  
管理员使用admin库中的user

---

## 运行redis

* docker运行

```bash
docker pull redis
# 在root目录添加redis相关文件夹和配置
# docker启动要注释redis.conf中logfile和dir两行
#  logfile /var/log/redis/redis.log
#  dir /var/lib/redis
# 修改port行 同启动参数中一致
docker run -itd -p 3679:6379  
-v /root/redis/redis.conf:/usr/local/etc/redis/redis.conf  
-v /root/redis/data/:/data  
--name myredis  
redis  
redis-server /usr/local/etc/redis/redis.conf
```

`腾讯云centos8.0中启动多个redis容器时，每个容器内部的配置端口最好不一样，不要都是6379，一样时很有可能把服务器卡死`

* 手动运行

```bash
yum install redis
systemctl start redis
systemctl enable redis
```

* 配置等

`vim /etc/redis.conf`  
注释 `#bind 127.0.0.1` 允许任意ip连接  
取消注释 `requirepass abcdefghijk` 设置连接密码

`systemctl status redis` 查看运行状态  
`ss -an | grep 6379` 查看端口是否listen

客户端推荐使用AnotherRedisDesktopManager
