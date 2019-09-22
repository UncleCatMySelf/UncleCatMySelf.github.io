---
layout:       post
title:        程序员：请说说代码中的线程吧
subtitle:     并发
date:         2019-09-21
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 并发
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “4分钟”
>
> 适读人群：Java 初级

线程是无处不在的。

先说说几个基本的概念吧。

一个进程中可以包含多个线程，同一个进程中的线程共享该进程所申请到的资源，如内存空间和文件句柄等。

从JVM的角度来看，线程是进程中的一个组件（Component）

Java程序中任何一段代码总是执行在某个确定的线程中

Java中线程分为守护线程（Daemon Thread）和用户线程（User Thread）

用户线程：JVM正常停止前应用程序中的所有用户线程必须先停止完毕，否则JVM无法停止

守护线程：不会影响JVM的正常停止，通常执行一些重要性不高的任务，如监视其他线程的运行情况

在多线程的运行中，我们需要注意每个段代码是由哪一个线程去负责执行的，这关系到性能问题、线程安全。

```java
System.out.println("The ** method was executed by thread: " + Thread.currentThread().getName());
```

如上可以看看对应方法是哪个线程负责执行的，当然你可以创新一个新的线程，并由新的线程负责，来验证你的猜想。


#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/qrcode.jpg)