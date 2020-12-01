---
title: Vue cli搭建单页应用并使用VSCode调试
date: 2020-09-29T16:02:02+08:00
lastmod: 2020-09-29T16:02:02+08:00
author: 林二狗
# authorlink: https://author.site
cover: https://blog-lin2code-1252382346.file.myqcloud.com/img/cover5.jpg
categories:
  - 前端
tags:
  - vue
  - 单页应用
  - vscode
  - 调试
# showcase: true
draft: false
---

## 安装

* 安装Nodejs

* 安装npm

* 安装Vue cli
  
  `npm install -g @vue/cli-service-global`

* 搭建vue项目

  vue create hello-world  
  选择默认模板

---

## VSCode调试

* 添加Debugger for Chrome插件

* 项目中添加vue.config.js

  ```javascript
    module.exports = {
        configureWebpack: {
            devtool: 'source-map'
        }
    }
  ```

* 修改项目中的/.vscode/launch.json  
  添加启动配置

  ```javascript
    {
        "type": "chrome",
        "request": "launch",
        "name": "vuejs: chrome",
        "url": "http://localhost:8080",
        "webRoot": "${workspaceFolder}/src",
        "breakOnLoad": true,
        "sourceMapPathOverrides": {
            "webpack:///src/*": "${webRoot}/*",
            "webpack:///./src/*": "${webRoot}/*"
        }
    }
  ```

* 当前目录下运行控制台命令  
  `npm run serve`

* 调试断点要设置在方法内  
  设置在export中的属性上不能命中源代码文件
