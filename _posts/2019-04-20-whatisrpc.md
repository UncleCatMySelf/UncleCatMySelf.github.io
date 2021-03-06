---
layout:       post
title:        RPC框架是啥？
subtitle:     架构
date:         2019-04-20
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 架构
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

在我刚刚了解分布式的时候，经常对RPC和分布式有些混淆，甚至一直以为两者对等，所以我们先看看他们有什么`区别`？

RPC实现了服务消费调用方Client与服务提供实现方Server之间的`点对点调用流程`，即包括了stub、通信、数据的序列化/反序列化。且Client与Server一般采用`直连`的调用方式。

而分布式服务框架，除了包括`RPC的特性`，还包括多台Server提供服务的负载均衡、策略及实现，服务的注册、发布与引入，以及服务的高可用策略、服务治理等等。

那么RPC是什么呢？

百度百科是这样表示的：

RPC（Remote Procedure Call）—`远程过程调用`，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。RPC协议假定某些传输协议的存在，如TCP或UDP，为通信程序之间携带信息数据。在OSI网络通信模型中，RPC跨越了传输层和应用层。RPC使得开发包括网络分布式多程序在内的应用程序更加容易。

它甚至给出了`工作原理`，这一点很惊喜。

![image](https://gss3.bdstatic.com/7Po3dSag_xI4khGkpoWK1HF6hhy/baike/c0%3Dbaike80%2C5%2C5%2C80%2C26/sign=88d234b10b46f21fdd395601974d0005/18d8bc3eb13533fadd93e964a9d3fd1f41345b56.jpg)

* 1.调用客户端句柄；执行传送参数
* 2.调用本地系统内核发送网络消息
* 3.消息传送到远程主机
* 4.服务器句柄得到消息并取得参数
* 5.执行远程过程
* 6.执行的过程将结果返回服务器句柄
* 7.服务器句柄返回结果，调用远程系统内核
* 8.消息传回本地主机
* 9.客户句柄由内核接收消息
* 10.客户接收句柄返回的数据

我喜欢搜查更多的信息资料，所以我又找到了`知乎`上的回答。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/rpc-%E7%9F%A5%E4%B9%8E.png)

知乎`1.7k`的点赞，应该还是可以参考的。

恰如回答提到的，RPC是指远程过程调用，也就是说两台服务器A，B，一个应用部署在A服务器上，想要调用B服务器上应用提供的函数/方法，由于不在一个内存空间，不能直接调用，需要通过网络来表达调用的语义和传达调用的数据。

至于`为什么使用RPC`？答主也提到，无法在一个进程内，甚至一个计算机内通过本地调用的方式完成的需求，比如不同系统间的通讯，甚至不同组织间的通讯。

这里再说一下关于Netty，Netty框架不局限于RPC，更多的是作为`一种网络协议的实现框架`，比如HTTP，由于RPC需要高效的网络通信，就可以选择Netty作为基础。除了网络通信，RPC还需要有高效的序列化框架，以及一种寻址方式，如果是带会话（状态）的RPC调用，还需要有会话的状态保持的功能。

好了，让我们再来整理一下，`什么是RPC`？

RPC（远程过程调用）一般用来`实现`部署在不同机器上的系统之间的`方法调用`，使得程序能够像访问本地系统资源一样，通过`网络传输`去访问远端系统资源。一般来说，RPC框架实现的`架构原理`都是类似的。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/RPC%E6%9E%B6%E6%9E%84.png)

可以这样说，

* 客户端调用：负责发起RPC调用，为调用方用户提供使用API。

* 服务端响应：主要是服务端业务逻辑实现。

* 序列化/反序列化：负责对RPC调用通过`网络传输的内容进行序列化与反序列化`，不同的RPC框架有不同的实现机制。一般分为`文本（XML、JSON）与二进制（Java原生的、Hessian、protobuf、Thrift、Avro、Kryo、MessagePack）`，需要注意的是，不同的序列化方式在可读性、码流大小、支持的数据类型及性能等方面都存在较大差异，我们可以根据需要自行选择。

* Stub：我们看成代理对象，它会屏蔽RPC调用过程中的复杂的`网络处理逻辑`，使其透明简单，且能够保持与本地调用一样的代码风格。

* 通信传输：即RPC的底层通信传输模块，一般通过Socket在客户端与服务端之间`传递请求与应答消息`。

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)

