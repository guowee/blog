---
title: windows 下MongoDB的安装与配置
date: 2018-03-07 19:14:57
tags:
---

# 一、 先登录Mongodb官网https://www.mongodb.com/download-center#community下载安装包

# 二、 安装MongoDB
自定义安装路径修改如下：  D:\MongoDB

# 三、创建数据库文件的存放位置
在MongoDB下创建data，因为启动mongodb服务之前需要必须创建数据库文件的存放文件夹，否则命令不会自动创建，而且不能启动成功。

# 四、启动MongoDB服务
1. 打开cmd命令行，进入D:\MongoDB\bin 目录
2. 输入如下命令启动MongoDB服务
    > mongodb --dbpath D:\MongoDB\data 
3. 在浏览器输入http://localhost:27017 查看
如果显示“It looks like you are trying to access MongoDB over HTTP on the native driver port.”,表示成功。

# 五、 配置本地windows mongodb 服务
这个可设置为开机自启动，可直接手动启动关闭，可通过命令行net start MongoDB 启动。
1. 先在MongoDb 文件夹下创建一个logs 文件夹下创建一个logs
2. 在MongoDB新建配置文件mongo.conf
3. 打开mongo.conf, 并输入：
```
dbpath=D:\MongoDB\data
logpath=D:\MongoDB\logs\mongo.log
logappend=true
journal=true
quiet=true  
port=27017
```
4. 配置windows服务 
以管理员的身份打开cmd,进入D:\MongoDB\bin目录下
输入mongod --config D:\\Mongodb\mongo.conf --install --serviceName "MongoDB"
完成后，再次查看本地的服务，如果成功的话，会发现本地服务多了”MongoDB"服务。

