---
layout:       post
title:        Centos7.6安装4.0.8MongoDb教程
subtitle:     Github
date:         2019-04-12
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Github
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

> 本系列教程为[HMStrange项目](https://github.com/UncleCatMySelf/HMStrange)附带。

## 历史文章

* [如何在VMware12安装Centos7.6最新版](https://unclecatmyself.github.io/2019/04/08/vmwarecentos/)
* [Centos7.6安装Java8](https://unclecatmyself.github.io/2019/04/09/java8/)
* [Centos7.6安装MySQL+Redis（最新版）](https://unclecatmyself.github.io/2019/04/10/mysqlandredis/)
* [SpringBoot+MySQL+MyBatis的入门教程](https://unclecatmyself.github.io/2019/04/10/smm/)
* [SpringBoot+Redis的入门教程](https://unclecatmyself.github.io/2019/04/11/redis/)


## 安装流程


#### 1、下载MongoDB的最新资源包，大家也可以关注我的公众号“Java猫说”，回复“工具包”，获取全部资源工具。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin1.png)

或者直接到官网下载，地址：https://www.mongodb.com/download-center#community

下载完成，使用xftp上传到自己的文件夹下

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin2.png)

#### 2、解压

```
# tar zxvf mongodb-linux-x86_64-4.0.8.tgz
```

#### 3、安装完成，我们选择使用命令+配置启动的方式，所以我们要准备一下配置信息，以下是我的目录

```
datamongodb/
    conf   #配置文件目录
        mongod.conf   #配置文件
    db     #数据库目录
    log    #日志文件目录
        mongodb.log   #日志文件
```

#### 4、编写配置信息mongod.conf

```
vi mongodb.conf

# 内容如下

dbpath=/home/myself/datamongodb/db

logpath=/home/myself/datamongodb/log/mongodb.log

logappend=true   #启动日志

fork=false   # 不保护进程

port=27017

bind_ip=0.0.0.0  #允许所有IP访问

```

#### 5、接下来就可以启动了

```
./mongod -f /home/myself/datamongodb/conf/mongodb.conf
```

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin8.png)

启动成功！

#### 6、然后我们window的本地环境推荐安装两款软件：Robo 3T、Studio 3T，大家也可以关注我的公众号“Java猫说”，回复“工具包”，获取全部资源工具。然后测试远程连接

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin3.png)

连接成功！

#### 7、使用Studio 3T，可以点击 IntelliShell 使用命令行来创建数据库，插入数据

```
use demo  # 新建数据库demo，没有数据的时候还看不到
db # 查看数据库信息
db.demo.install({"id":"a","msg":"欢迎学习HMStrange项目！"}) # 插入数据
```

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin4.png)
![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin5.png)
![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin6.png)

#### 8、使用 Robo 3T 查看数据信息

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/monin7.png)

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)