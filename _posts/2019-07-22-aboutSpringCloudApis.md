---
layout:       post
title:        浅聊SpringCloud的网关
subtitle:     Netty
date:         2019-07-22
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Netty
    - 微服务
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “4分钟”
>
> 适读人群：Java初级

### 为什么要设计网关？

上网搜罗了一下，觉得别人说的挺好，就引用了一下，在使用微服务的时候，不同的功能业务会集成一个服务群，而网关是基于服务群上的一个服务层，也是单独暴露给客户端的APIs。

> 客户端对微服务的依赖直接使重构服务变得困难。一种直观的方法是将这些服务隐藏在一个新的服务层后面，并提供针对每个客户端的APIs。
>
>这个聚合器服务层也称为API网关，它是解决这个问题的一种常见方法。

引用下图，[原文出处](https://cloud.tencent.com/developer/article/1349597)。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7/2019/7/%E7%BD%91%E5%85%B3.png)

### SpringCloud的网关

#### zuul1.X（阻塞）

- **架构：**

通过servlet做处理，并通过多个Groovy Filter做链过滤请求

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7/2019/7/zuul1.0%E6%9E%B6%E6%9E%84.png)

- **现状：**

目前比较少，但是对于实时业务还是可以稳定使用

- **应用场景：**

简单业务，逻辑简单，实时业务，cpu型业务

- **使用方式：**

引入maven包，使用注解的形式，可以在配置文件配置

#### zuul2.X（非阻塞）


- **架构：**

2.0引入了Netty服务，实现非阻塞与高并发的处理能力

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7/2019/7/zuul2.0%E6%9E%B6%E6%9E%84.png)

- **现状：**

官方停止维护，非阻塞

- **应用场景：**

大数据、队列类型、高并发、io型业务

- **使用方式：**

引入maven包，使用注解的形式，可以在配置文件配置

#### Gateway（非阻塞）

- **架构：**

与zuul2.0一致，不上图

- **现状：**

SpringCloud官方维护，因为zuul2.X停止维护，所以基于2.X的同架构版本

- **应用场景：**

大数据、队列类型、高并发、io型业务

- **使用方式：**

引入maven包，路由注解（route-》path-》filters-》uri）或者以配置的形式

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)
