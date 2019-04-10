---
layout:       post
title:        Centos7.6安装MySQL+Redis（最新版）
subtitle:     Github
date:         2019-04-10
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Github
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

>本系列教程为[HMStrange项目](https://github.com/UncleCatMySelf/HMStrange)附带。

## 历史文章

* [如何在VMware12安装Centos7.6最新版](https://unclecatmyself.github.io/2019/04/08/vmwarecentos/)
* [Centos7.6安装Java8](https://unclecatmyself.github.io/2019/04/09/java8/)

## MySQL教程

1、下载mysql，地址：http://dev.mysql.com/get/mysql-community-release-el7-5.noarch.rpm
2、使用xftp上传到自己想要得目录
3、代码操作，安装并重启mysql服务

```
# rpm -ivh mysql-community-release-el7-5.noarch.rpm
# yum install mysql-community-server
# service mysqld restart
```
4、设置mysql的root密码

```
# mysql -u root
mysql> set password for 'root'@'localhost' =password('你的密码');
mysql> flush privileges;
```
5、配置mysql编码

```
# vi /etc/my.cnf
[mysql]
default-character-set =utf8
```
6、允许远程链接，进入mysql

```
mysql> grant all privileges on *.* to root@'%'identified by '远程密码';
mysql> flush privileges;
```

7、开启3306端口，或者关闭防火墙，我是虚拟机环境且自己用的，我直接关了防火墙。

```
systemctl stop firewalld.service    #停止firewall
systemctl disable firewalld.service     #禁止firewall开机启动
```

## Redis教程

1、下载redis，地址： http://download.redis.io/releases/redis-5.0.4.tar.gz

2、安装gcc等默认需要的

```
yum install -y gcc tcl
```

3、解压安装

```
# tar xzf redis-5.0.4.tar.gz
# cd redis-5.0.4
# make install
```

4、修改redis.conf

```
bind 0.0.0.0  #允许远程
protected-mode no #关闭保护模式
daemonize yes #守护进程模式开启
```
5、启动redis

```
# cd src
# ./redis-server ./../redis.conf
```

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)