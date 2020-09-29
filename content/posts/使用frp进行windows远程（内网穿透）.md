---
title: 使用frp进行windows远程（内网穿透）
date: 2020-09-29T15:00:00+08:00
lastmod: 2020-09-29T15:00:00+08:00
author: 林二狗
# authorlink: https://author.site
cover: /img/cover4.jpg
categories:
  - 笔记
tags:
  - frp
  - 远程控制
  - windows
  - 内网穿透
# showcase: true
draft: false
---

> 首先你要有个服务器

## 下载

GitHub仓库[frp](https://github.com/fatedier/frp/releases)中下载各个系统的对应版本

---

## 安装

### linux版服务端安装

上传frps和frps.ini文件即可  
frps.ini配置内容如下

```config
[common]
bind_port = 1234
```

配置说明：和客户端通信的端口，用于内网穿透

所在目录执行命令 `nohup ./frps -c ./frps.ini &` 后台运行frps服务端

---

## 客户端

下载windows版客户端程序  
只需frpc.exe和frpc.ini文件  
frpc.ini配置文件如下

```config
[common]
server_addr = 你的服务器地址
server_port = 1234

[ssh]
type = tcp
local_ip = 127.0.0.1
local_port = 3389
remote_port = 3389
```

`[common]`中的参数是服务器ip和内网穿透通信端口

`[ssh]`中  
`local_ip` 和 `local_port` 表示客户端将建立的连接转发到那个本地ip和端口
`remote_port` 表示将服务端这个端口的连接转发到客户端指定的ip和端口

控制台中定位到frpc所在目录 执行命令`frpc -c frpc.ini`即可启动客户端  
也可制作成bat文件快速启动

```cmd
d:
cd D:\Software\frpc
frpc -c frpc.ini
```

---

## windows客户端安装成系统服务

下载[winsw](https://github.com/kohsuke/winsw)打包服务工具放到frpc.exe所在目录  
添加一个winsw.xml配置文件

```xml
<service>
    <id>frpc</id>
    <name>frpc</name>
    <description>frp客户端</description>
    <executable>frpc</executable>
    <arguments>-c frpc.ini</arguments>
    <logmode>reset</logmode>
</service>
```

管理员控制台执行 `winsw install`，然后手动启动该服务即可
