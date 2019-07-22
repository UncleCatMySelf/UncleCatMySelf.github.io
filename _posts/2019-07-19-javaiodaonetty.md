---
layout:       post
title:        从Java Socket非阻塞到Netty入门流程
subtitle:     Netty
date:         2019-07-19
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Netty
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “4分钟”
>
> 适读人群：Java初级

### Java IO，Socket非阻塞通信流程

这里我们使用一个内嵌的永久循环，来让Socket成为一个非阻塞的通信流程。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7/2019/7/java.png)

如上图所示，ServerSocket是我们自建的一个类，通过启动线程，且线程内置一个真循环，**防止accept阻塞**；

在客户端监听类上，将监听到的socket作为参数，传递到客户端监听类上，并再次启动线程，获取一个InputStream，同时再次在这个刚刚启动线程内置一个真循环，为的是**不断获取信息并回写**；

这里要注意的是，第一个真循环是**保证获取新连接不会阻塞**，第二个真循环是**保证不停的获取客户端信息并回写**；

关于客户端则通过端口和IP，启动线程，通过一个循环不停的向服务端写数据；

### Netty入门

基于上面的图，我们也可以学习Netty相关的基础入门。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%BE%AE%E4%BF%A1%E5%85%AC%E4%BC%97%E5%8F%B7/2019/7/netty.png)

#### NioEventLoop（事件循环）

1、新连接接入 

2、连接上的数据读取

#### Channel（抽象连接）

Socket、SocektChannel（IO\NIO）抽象

#### ChannelHandler（业务逻辑处理）

读写数据期间的业务层

#### PipeLine（动态链处理）

多个ChannelHandler组成，让消息可以层层处理

#### ByteBuf（数据接收）

基本的数据处理基于ByteBu

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)
