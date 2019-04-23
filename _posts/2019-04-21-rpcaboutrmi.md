---
layout:       post
title:        RPC框架是啥之Java自带RPC实现，RMI框架入门
subtitle:     架构
date:         2019-04-21
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 架构
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

## 学习系列

* [RPC框架是啥？](https://unclecatmyself.github.io/2019/04/20/whatisrpc/)

## Java自带RPC实现，RMI框架入门

首先RMI（Remote Method Invocation）是Java特有的一种RPC实现，它能够使部署在不同主机上的Java对象进行通信与方法调用，它是一种基于Java的远程方法调用技术。

让我们优先来实现一个RMI的RPC案例吧。

> 项目源码地址：[RPC_Demo](https://github.com/UncleCatMySelf/RPC_Demo)，记得是项目里面的`comgithubrmi`

- 1、首先我们需要为服务端创建一个接口方法，而且这个接口最好继承`Remote`

```java
package com.github.rmi.server;

import java.rmi.Remote;
import java.rmi.RemoteException;

/**
 * Create by UncleCatMySelf in 21:03 2019\4\20 0020
 */
public interface MyService extends Remote {

    String say(String someOne)throws RemoteException;

}
```

- 2、对于接口实现类，RMI接口方法定义必须显式声明抛出RemoteException异常，服务端方法实现必须继承UnicastRemoteObject类，该类定义了服务调用与服务提供方对象实现，并建立一对一的连接。

```java
package com.github.rmi.server;

import java.rmi.RemoteException;
import java.rmi.server.UnicastRemoteObject;

/**
 * Create by UncleCatMySelf in 21:05 2019\4\20 0020
 */
public class MyServiceImpl extends UnicastRemoteObject implements MyService {

    protected MyServiceImpl() throws RemoteException {
    }

    public String say(String someOne) throws RemoteException {
        return someOne + ",Welcome to Study!";
    }
}
```

- 3、这里我们还需要一个针对服务端的配置类，因为RMI的通信端口是随机产生的，因此有可能会被防火墙拦截。为了防止被防火墙拦截，需要强制制定RMI的通信端口，一般通过自定义一个RMISocketFactory类来实现。

```java
package com.github.rmi.config;

import java.io.IOException;
import java.net.ServerSocket;
import java.net.Socket;
import java.rmi.server.RMISocketFactory;

/**
 * Create by UncleCatMySelf in 21:15 2019\4\20 0020
 */
public class CustomerSocketFactory extends RMISocketFactory {

    public Socket createSocket(String host, int port) throws IOException {
        return new Socket(host, port);
    }

    public ServerSocket createServerSocket(int port) throws IOException {
        if (port == 0){
            port = 8855;
        }
        System.out.println("RMI 通信端口 : " + port);
        return new ServerSocket(port);
    }
}
```

- 4、好了，这时你可以写出服务端的启动代码了。

```java
package com.github.rmi.server;

import com.github.rmi.config.CustomerSocketFactory;

import java.rmi.Naming;
import java.rmi.registry.LocateRegistry;
import java.rmi.server.RMISocketFactory;

/**
 * Create by UncleCatMySelf in 21:07 2019\4\20 0020
 */
public class ServerMain {

    public static void main(String[] args) throws Exception {
        //注册服务
        LocateRegistry.createRegistry(8866);
        //指定通信端口，防止被防火墙拦截
        RMISocketFactory.setSocketFactory(new CustomerSocketFactory());
        //创建服务
        MyService myService = new MyServiceImpl();
        Naming.bind("rmi://localhost:8866/myService",myService);
        System.out.println("RMI 服务端启动正常");
    }

}
```

- 5、客户端的启动就相对比较简单，我们仅需要进入服务，并调用对应的远程方法即可。

```java
package com.github.rmi.client;

import com.github.rmi.server.MyService;

import java.rmi.Naming;

/**
 * Create by UncleCatMySelf in 21:10 2019\4\20 0020
 */
public class ClientMain {

    public static void main(String[] args) throws Exception {
        //服务引入
        MyService myService = (MyService) Naming.lookup("rmi://localhost:8866/myService");
        //调用远程方法
        System.out.println("RMI 服务端调用返回：" + myService.say("MySelf"));
    }

}
```

最后可以看看效果。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/rmi01.png)
![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/rmi02.png)

## 关于RMI的一些关键点

* 支持真正的面向对象的多态性，这是RMI的优势。
* 完美支持Java语言所独有的特性，不支持其他语言。
* 使用了Java原生序列化，所有序列化对象必须实现java.io.Serializablie接口。
* 底层通信是BIO（同步阻塞I/O）实现的Socket
* 由于BIO与原生序列化存在的性能问题，导致RMI的性能较差，如果你的项目性能要求较高，可能并不合适哦！

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)
