---
title: Dotnet Core注意事项和笔记
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

## linux后台运行

`nohup dotnet xxx.dll --urls https://localhost:1234 > nohup.out 2>&1 &`

## 使用 watch 命令实时运行本地项目

在项目上右键在控制台打开  
然后直接运行 `dotnet watch run`  
会使用当前项目配置后台运行程序，并检测文件变化实时更新  
此时直接输入项目地址就可以访问，修改代码保存后程序自动更新  
如需调试需停止控制台程序再手动调试  

当然也可以附加watch进程到项目来实时更新和调试参考 [这里](https://dotnetcoretutorials.com/2020/01/01/live-coding-net-core-using-dotnet-watch/)

## 制作成service

由于种种原因 asp.net core不能像IIS一样热发布  
但是可以配合nginx蓝绿发布  

制作成 service 快速停止和启动程序加快发布效率

创建文件:  
`sudo vim /etc/systemd/system/kestrel-helloapp.service`

以下是该应用程序的示例服务文件：

```ini
[Unit]
Description=Example .NET Web API App running on Ubuntu

[Service]
WorkingDirectory=/var/www/helloapp
ExecStart=/usr/bin/dotnet /var/www/helloapp/helloapp.dll
Restart=always
# Restart service after 10 seconds if the dotnet service crashes:
RestartSec=10
KillSignal=SIGINT
SyslogIdentifier=dotnet-example
User=www-data
Environment=ASPNETCORE_ENVIRONMENT=Production
Environment=DOTNET_PRINT_TELEMETRY_MESSAGE=false

[Install]
WantedBy=multi-user.target
```

在前面的示例中，该User选项指定了管理服务的用户。用户（www-data）必须存在并且对应用程序文件拥有适当的所有权。  
User一般改成root之类。

然后  
`systemctl enable dotnet-mall-api.service`  
`systemctl start dotnet-mall-api.service`  
`systemctl status dotnet-mall-api.service`

`systemctl restart dotnet-mall-api.service`
配置文件同步工具FreeFileSync体验非常好

## 使用MongoDB事务

首先需要在基础操作上增加一个 `IClientSessionHandle` 参数，类似这样

```c#
        /// <summary>
        /// 带事务插入
        /// </summary>
        /// <param name="entity"></param>
        public void InsertInTrans(IClientSessionHandle session, T entity)
        {
            GetCollection().InsertOne(session, entity);
        }

        /// <summary>
        /// 带事务批量插入
        /// </summary>
        /// <param name="entitys"></param>
        public void InsertManyInTrans(IClientSessionHandle session, IEnumerable<T> entitys)
        {
            if (entitys.Count() == 0)
            {
                return;
            }
            GetCollection().InsertMany(session, entitys);
        }

        /// <summary>
        /// 事务中更新
        /// </summary>
        /// <param name="entity"></param>
        public void UpdateInTrans(IClientSessionHandle session, T entity)
        {
            GetCollection().ReplaceOne(session, e => e.ID == entity.ID, entity);
        }
```

这个的参数从连接后的数据库对象上获取，具体使用时建议注入一个数据库连接对象，通过连接对象获取

```c#
        public MongoClient MongoCli { get; set; }

        /// <summary>
        /// 获取可以执行事务的会话
        /// </summary>
        /// <returns></returns>
        public IClientSessionHandle GetSession()
        {
            return MongoCli.StartSession();
        }
```

使用

```c#
            //集群mongodb才能使用事务
            using (var session = MainConn.GetSession())
            {
                session.StartTransaction();
                try
                {
                    //......
                    DBOrder.InsertInTrans(session, order);
                    //commit前return主动session.AbortTransaction();
                    //......

                    session.CommitTransaction();
                }
                catch (Exception e)
                {
                    Logger.LogError(e, "创建订单失败");
                    session.AbortTransaction();
                    return Ok(new { code = -1, msg = "创建订单失败，内部异常" });
                }
            }
```

如果不是集群部署MongoDB会报错 待部署集群后测试更新本文进度
