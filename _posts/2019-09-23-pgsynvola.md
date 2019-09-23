---
layout:       post
title:        程序员：不能逃避的synchronize和volatile
subtitle:     并发
date:         2019-09-23
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 并发
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “10分钟”
>
> 适读人群：Java 初级

> 学习笔记，我也是呆呆做了好久，学了一下PS，然后继续思考了一会，再开始写出来的，希望可以简明易懂。

## 原子性

首先是我们彼此都要保持一致的观点：原子（Atomic）操作指相应的操作是单一不可分割的操作

emmmm，这里很牵强的解释下原子性，还是不懂就搜搜其他文章，最好看看一些具体的例子

首先是代码例子

对int型变量conut执行counter++的操作不是原子操作

这可以分为3个操作

```js
1、读取变量counter的当前值
2、拿counter当前值和1做加法运算
3、将counter的当前值增加1后赋值给counter变量
```

上面的步骤2，很有可能在执行的时候就已经被其他线程修改了，其所为的“当前值”已经是过期的

或者看看百度百科的例子

我们以decl （递减指令）为例，这是一个典型的"读－改－写"过程，涉及两次内存访问。设想在不同CPU运行的两个进程都在递减某个计数值，可能发生的情况是：

```js
⒈ CPU A(CPU A上所运行的进程，以下同）从内存单元把当前计数值⑵装载进它的寄存器中；
⒉ CPU B从内存单元把当前计数值⑵装载进它的寄存器中。
⒊ CPU A在它的寄存器中将计数值递减为1；
⒋ CPU B在它的寄存器中将计数值递减为1；
⒌ CPU A把修改后的计数值⑴写回内存单元。
⒍ CPU B把修改后的计数值⑴写回内存单元。
```

内存里的计数值应该是0，然而它却是1。两个进程都去掉了对该共享资源的引用，但没有一个进程能够释放它--两个进程都推断出：计数值是1，共享资源仍然在被使用

我再举例我呆想到的例子，一个姐姐和一个妹妹一起包饺子

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/%E5%A7%90%E5%A6%B9%E9%A5%BA%E5%AD%90.jpg)

画的很一般，别看我这样，我也是学过2小时速成素描的·····

假设我们在一个黑盒环境下，就是两姐妹都在各自小空间包饺子，然后她们把饺子通过各自的小洞口放入一个大盒子里。她们并不知道对方（比如她们两刚刚因为妈妈不给零花钱而生气了）

这个时候她们各自同时边赌气边包了一个饺子，同时放到盒子里，妈妈跑过来问老大，盒子里有多少个了？她只知道一个。再问问老二，她也是回答一个。这个生活例子可能提交特殊，不过偶尔生活中因为信息不对称而导致的预知结果与实际有偏差也是经常发生的

所以他们脑海就是这个情况。其实盒子里已经是2个饺子了

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/%E5%A7%90%E5%A6%B9.png)

那么其实这个场景也像是JVM

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/jvm.png)

## synchronize

synchronize关键字可以实现操作的原子性，其本质是通过该关键字所包括的临界区的排他性保证在任何一个时刻只有一个线程能够执行临界区中的代码

也就是说，现在妈妈说只有听她的，两姐妹才能有零花钱，所以她叫两个闹脾气的小鬼都到厨房，并拿出了大盒子，让她们重新开始，不过要按照妈妈的要求来

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/mom.jpg)

妈妈先让姐姐包了5个，因为两姐妹都在厨房，不是各自在房间，所以这次妹妹都看在眼里，接着妈妈让妹妹包10个，妹妹显然是有点不乐意了（凭什么我姐才5个），不过她还是老实做了，现在他们三人都知道盒子里有15个

这里就又牵出了synchronize的另一个特点，保证内存的可见性

它保证了一个线程执行临界区中的代码时所修改的变量值对于稍有执行该临界区中的代码的线程来说是可见的，这对于保证多线程的代码是非常重要的

官方的解释下：CPU执行代码，为了减少变量访问的消耗，会将值缓存到CPU缓存区，再次访问的时候，就是从缓存区去读取而不是主内存，这里的缓存区有点类似姐姐脑海/妹妹脑海。而且代码对缓存区的修改可能仅修改缓存区，没有被写回主内存。由于CPU都有自己的存储区，对于不同CPU的存储区内容是不可见的。这也是所谓的内存可见性

## volatile

同样这个兄弟也可以保证内存可见性

一个线程对于一个采用volatile修改的变量的值的更改对于其他访问该变量的值的线程总是可见的

如果说对比synchronize和volatile的内存锁，然后说volatile是轻量级锁，emmmm，不好不太恰当

volatile的内部锁并不能保证操作的原子性。

他在内存可见性的核心机制是：修改的值会被写入主内存，且其他CPU缓存区的值会因此失效（然后再更新一个最新值），保证其他线程访问volatile修饰的变量总是最新值。

当然他也有一个核心作用：禁止指令重排序（Re-order）

你们一般怎么写5的？

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/%E6%AD%A3%E5%B8%B85.jpg)

假如以上是我们的规定与希望

可能编译器和CPU为了提供指令的执行效率可能会进行指令重排序（优化）

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/%E9%9D%9E%E6%AD%A3%E5%B8%B85.jpg)

如果你希望它是按照规定来的话就加上volatile，虽然可能会导致编译器和CPU无法对一些指令做可能的优化，假设上面那样写对于计算机来说算优化:)

用程序来写一个例子：

```java
private SomeOne object = new SomeOne();
```

你先想一下，你觉得的顺序，好了，我说说计算机可能的顺序

- 1、分配一段用于存储SomeOne的内存空间
- 2、对该内存空间引用赋值给变量object
- 3、创建类SomeOne

如果当其他线程访问2、object变量的时候，仅得到一个指向存储SomeOne存储空间的引用，因为3、SomeOne还没创建

## 结语

希望各位兄弟能看到一些新的风景，synchronize可以保证操作原子性，且保证内存可见性；volatile仅能保证内存可见性。

synchronize会导致上下文切换，volatile不会哦。

关于上下文切换的，可以去看公众号的上一篇文章

> 我是MySelf，还在坚持学习技术与产品经理相关的知识，希望本文能给你带来新的知识点。

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/qrcode.jpg)