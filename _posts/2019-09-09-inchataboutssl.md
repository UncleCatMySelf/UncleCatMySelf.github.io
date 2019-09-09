---
layout:       post
title:        关于Inchat启动SSL加密，本地浏览器无法连接问题
subtitle:     Netty
date:         2019-09-09
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Netty
    - 官方案例
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “4分钟”
>
> 适读人群：Java-Netty 初级

## Issue

本文解决[InChat](https://github.com/AwakenCN/InChat/issues)项目[Issue](https://github.com/AwakenCN/InChat/issues/24)

```js
use isSSL=true
使用chrome浏览器客户端未显示不安全的链接，导致添加不了证书。

open ssl success
INFO - [DefaultWebSocketHandler.channelActive]/10.0.75.1:55663链接成功
INFO - [DefaultWebSocketHandler.exceptionCaught]/10.0.75.1:55663异常断开
INFO - [Handler：channelInactive]0.0.0.0/0.0.0.0:8070关闭成功
ERROR - [捕获异常：NotFindLoginChannlException]-[Handler：channelInactive] 关闭未正常注册链接！
INFO - [DefaultWebSocketHandler.exceptionCaught]/10.0.75.1:55663异常断开
```

而且我使用chrome，按F12后，发现这样一段话

```js
WebSocket connection to 'wss://192.168.56.1:8090/ws' failed: Error in connection establishment: net::ERR_CERT_AUTHORITY_INVALID
```

## 解决方案

经过测试，在正常clone项目后，如果你想要采用inchat各个版本的ssl加密，如果没有特殊配置，均会出现以上的情况，且连接不上的现象。

这里项目给出最新的测试方式，这里的测试不是与框架无关，也与浏览器无关，是因为证书的信任问题。

**由于证书是自签名的，所以证书的CA肯定在操作系统的根存储区域是没有的，自然操作系统就不会认可你，自然浏览器也不认你，也就是自签证书不受信任。**

### 1、启动ssl项目

```java
    /** 是否启动加密功能 */
    @Override
    public boolean isSsl() {
        return true;
    }
```

查看启动成功的IP端口

```js
2019-09-09 13:46:28.222  INFO 10096 --- [         BOSS_1] c.g.u.bootstrap.NettyBootstrapServer     : 服务端启动成功【192.168.56.1:8090】
```

### 2、本地配置ssl测试域名

因为我的是window环境，其他环境的就不一一讲解了。

打开HOSTS文件，新增一个对应IP的域名，这里的IP就是你项目启动的IP端口。

新增：

```js
192.168.56.1 www.myself.com
```

### 3、chrome认证

由于大多数IT人员都是选择使用chrome，所以我这边就按照chrom来给出教程。

访问 **https://www.myself.com:8090/**

![加密1](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/ssl-bug/%E5%8A%A0%E5%AF%861.png)

点击**高级**

![加密2](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/ssl-bug/%E5%8A%A0%E5%AF%862.png)

选择继续前往就好

![加密3](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/ssl-bug/%E5%8A%A0%E5%AF%863.png)

### 4、打开项目前端页面

接下来我们再尝试连接，这里需要注意，修改前端连接代码

```js
socket = new WebSocket("wss://www.myself.com:8090/ws");
```

![okay](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/ssl-bug/okay.png)

这个时候项目就正常启动。

后台也显示ssl启动成功，连接正常。

```js
open ssl  success
2019-09-09 14:19:40.313  INFO 10096 --- [         WORK_1] c.g.u.bootstrap.handler.DefaultHandler   : [DefaultWebSocketHandler.channelActive]/192.168.56.1:7004链接成功
```

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)