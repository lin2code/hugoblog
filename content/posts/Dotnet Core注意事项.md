---
title: Dotnet Core注意事项
date: 2020-12-01T21:31:05+08:00
lastmod: 2020-12-01T21:31:05+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover11.jpg
categories:
  - 后端
tags:
  - dotnet core
# showcase: true
draft: false
---

## 环境配置

dotnet环境问题先卸载所有dotnet相关yum包
查看已安装的dotnet环境： `dotnet --list-runtimes`  

安装SDK: `yum install dotnet-sdk-3.1`（安装SDK就不用安装runtime了）

设置环境变量: `export ASPNETCORE_ENVIRONMENT=Product`  
程序默认使用对应环境变量配置文件  
重载环境变量: `source /etc/profile`  
永久修改环境变量: `vim /etc/profile` 最后一行加入 export 语句  

添加本地https证书: `dotnet dev-certs https --trust`
windows添加环境变量：`setx ASPNETCORE_ENVIRONMENT=Development`

## Webapi 中接收 Post 动态参数

可以省去定义结构的步骤

```c#
services.AddControllers(options =>
{
  //.......
})
//替换默认的 system.text.json 使得 post 时可以动态接收参数，同时设置序列化首字母不小写
.AddNewtonsoftJson(opt => opt.SerializerSettings.ContractResolver = new DefaultContractResolver());
```

Action中使用 `dynamic` 类型直接接收

## windows 中手动启动 asp dotnet core 程序

先设置环境变量  
如果是 development 环境，不要在发布文件夹中执行启动命令，因为发布是Product环境，不会包含development的配置！  
应在 debug 目录下执行`dotnet xxxx.dll --urls https://localhost:1234`  
有问题时注意重新生成  

## 后台运行

`nohup dotnet xxx.dll --urls https://localhost:1234 > nohup.out 2>&1 &`
