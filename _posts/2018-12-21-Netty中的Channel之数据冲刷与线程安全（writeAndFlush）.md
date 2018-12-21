---
layout:       post
title:        Netty中的Channel之数据冲刷与线程安全（writeAndFlush）
subtitle:     Netty知识点
date:         2018-12-21
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Netty
    - InChat
---

## GitHub项目地址



![Image text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/logo.png)

[InChat](https://github.com/UncleCatMySelf/InChat)

> 一个轻量级、高效率的支持多端（应用与硬件Iot）的异步网络应用通讯框架 

## 前言

本文预设读者已经了解了一定的Netty基础知识，并能够自己构建**一个Netty的通信服务（包括客户端与服务端）**。那么你一定使用到了Channel，这是Netty对传统JavaIO、NIO的链接封装实例。

那么接下来让我们来了解一下关于**Channel的数据冲刷与线程安全**吧。

## 数据冲刷的步骤

### 1、获取一个链接实例

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    //获取链接实例
    Channel channel = ctx.channel();
}
```

我将案例放在初学者最熟悉的channelRead方法中，这是**一个数据接收的方法**，我们自实现Netty的消息处理接口时需要重写的方法。即客户端发送消息后，这个方法会被触发调用，所以我们在这个方法中进行本次内容的讲解。

由上一段代码，其实目前还是很简单，我们借助**ChannelHandlerContext（这是一个ChannelHandler与ChannelPipeline相交互并对接的一个对象**。如下是源码的解释）来获取目前的链接实例Channel。

```java
/* Enables a {@link ChannelHandler} to interact with its {@link ChannelPipeline}
 * and other handlers. Among other things a handler can notify the next {@link ChannelHandler} in the
 * {@link ChannelPipeline} as well as modify the {@link ChannelPipeline} it belongs to dynamically.
 */
 public interface ChannelHandlerContext extends AttributeMap, ChannelInboundInvoker, ChannelOutboundInvoker {
     //......
 }
```

### 2、创建一个持有数据的ByteBuf

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    //获取链接实例
    Channel channel = ctx.channel();
    //创建一个持有数据的ByteBuf
    ByteBuf buf = Unpooled.copiedBuffer("data", CharsetUtil.UTF_8);
}
```

ByteBuf又是什么呢？

> 它是Netty框架自己封装的**一个字符底层对象**，是一个对 byte[] 和 ByteBuffer NIO 的抽象类，更官网的说就是“零个或多个字节的随机和顺序可访问的序列。”，如下是源码的解释

```java
/**
 * A random and sequential accessible sequence of zero or more bytes (octets).
 * This interface provides an abstract view for one or more primitive byte
 * arrays ({@code byte[]}) and {@linkplain ByteBuffer NIO buffers}.
 */
 public abstract class ByteBuf implements ReferenceCounted, Comparable<ByteBuf> {
     //......
 }
```

由上一段源码可以看出，ByteBuf是一个抽象类，所以我们不能通过 **new** 的形式来创建一个新的ByteBuf对象。那么我们可以通过Netty提供的**一个 final 的工具类 Unpooled**（你将其看作是一个创建ByteBuf的工具类就好了）。

```java
/**
 * Creates a new {@link ByteBuf} by allocating new space or by wrapping
 * or copying existing byte arrays, byte buffers and a string.
 */
 public final class Unpooled {
     //......
 }
```

这真是一个有趣的过程，那么接下来我们仅需要再看看 **copiedBuffer** 这个方法了。这个方法相对简单，就是我们将创建**一个新的缓冲区**，其内容是我们指定的 UTF-8字符集 编码指定的 “data” ，同时这个新的缓冲区的读索引和写索引分别是0和字符串的长度。


### 3、冲刷数据


```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    //获取链接实例
    Channel channel = ctx.channel();
    //创建一个持有数据的ByteBuf
    ByteBuf buf = Unpooled.copiedBuffer("data", CharsetUtil.UTF_8);
    //数据冲刷
    channel.writeAndFlush(buf);
}
```

我相信大部分人都是直接这么写的，因为我们经常理所当然的启动测试，并在客户端接受到了这个 “data” 消息。那么我们是否应该注意一下，**这个数据冲刷会返回一个什么值**，我们要如何才能在服务端知道，**这次数据冲刷是成功还是失败呢？**

那么其实Netty框架已经考虑到了这个点，本次数据冲刷我们将得到一个 ChannelFuture 。

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    //获取链接实例
    Channel channel = ctx.channel();
    //创建一个持有数据的ByteBuf
    ByteBuf buf = Unpooled.copiedBuffer("data", CharsetUtil.UTF_8);
    //数据冲刷
    ChannelFuture cf = channel.writeAndFlush(buf);
}
```

是的，他就是 Channel 异步IO操作的结果，它是一个接口，并继承了Future<V>。（如下为源码的解释）

```java
/**
 * The result of an asynchronous {@link Channel} I/O operation.
 */
 public interface ChannelFuture extends Future<Void> {
     //......
 }
```

既然如此，那么我们可以明显的知道我们可以对其添加对应的监听。

### 4、异步回调结果监听

```java
@Override
public void channelRead(ChannelHandlerContext ctx, Object msg) throws Exception {
    //获取链接实例
    Channel channel = ctx.channel();
    //创建一个持有数据的ByteBuf
    ByteBuf buf = Unpooled.copiedBuffer("data", CharsetUtil.UTF_8);
    //数据冲刷
    ChannelFuture cf = channel.writeAndFlush(buf);
    //添加ChannelFutureListener以便在写操作完成后接收通知
    cf.addListener(new ChannelFutureListener() {
        @Override
        public void operationComplete(ChannelFuture future) throws Exception {
            //写操作完成，并没有错误发生
            if (future.isSuccess()){
                System.out.println("successful");
            }else{
                //记录错误
                System.out.println("error");
                future.cause().printStackTrace();
            }
        }
    });
}
```

好的，我们可以简单的从代码理解到，我们将通过对异步IO的结果监听，得到本次运行的结果。我想这才是一个相对完整的 **数据冲刷（writeAndFlush）**。

## 测试线程安全的流程

对于线程安全的测试，我们将模拟多个线程去执行数据冲刷操作，我们可以用到 **Executor** 。

我们可以这样理解 **Executor** ，是**一种省略了线程启用与调度的方式**，你只需要传递一个 **Runnable** 给它即可，你不再需要去 start 一个线程。（如下是源码的解释）

```java
/**
 * An object that executes submitted {@link Runnable} tasks. This
 * interface provides a way of decoupling task submission from the
 * mechanics of how each task will be run, including details of thread
 * use, scheduling, etc.  An {@code Executor} is normally used
 * instead of explicitly creating threads. For example, rather than
 * invoking {@code new Thread(new(RunnableTask())).start()} for each
 * of a set of tasks, you might use:...
 */
 public interface Executor {
     //......
 }
```

那么我们的测试代码，大致是这样的。

```java
final Channel channel = ctx.channel();
//创建要写数据的ByteBuf
final ByteBuf buf = Unpooled.copiedBuffer("data",CharsetUtil.UTF_8).retain();
//创建将数据写到Channel的Runnable
Runnable writer = new Runnable() {
    @Override
    public void run() {
        ChannelFuture cf = channel.writeAndFlush(buf.duplicate());
        cf.addListener(new ChannelFutureListener() {
            @Override
            public void operationComplete(ChannelFuture future) throws Exception {
                //写操作完成，并没有错误发生
                if (future.isSuccess()){
                    System.out.println("successful");
                }else{
                    //记录错误
                    System.out.println("error");
                    future.cause().printStackTrace();
                }
            }
        });
    }
};

//获取到线程池的Executor的引用
Executor executor = Executors.newCachedThreadPool();

//提交到某个线程中执行
executor.execute(writer);

//提交到另一个线程中执行
executor.execute(writer);
```

这里，我们需要**注意**的是：

创建 **ByteBuf** 的时候，我们使用了 **retain** 这个方法，他是将我们生成的这个 ByteBuf 进行**保留操作**。

在 ByteBuf 中有这样的一种区域： **非保留和保留派生缓冲区**。

这里有点复杂，我们可以简单的理解，如果调用了 retain 那么数据就存在派生缓冲区中，如果没有调用，则会在调用后，移除这一个字符数据。（如下是 ByteBuf 源码的解释）

```java
/*<h4>Non-retained and retained derived buffers</h4>
 *
 * Note that the {@link #duplicate()}, {@link #slice()}, {@link #slice(int, int)} and {@link #readSlice(int)} does NOT
 * call {@link #retain()} on the returned derived buffer, and thus its reference count will NOT be increased. If you
 * need to create a derived buffer with increased reference count, consider using {@link #retainedDuplicate()},
 * {@link #retainedSlice()}, {@link #retainedSlice(int, int)} and {@link #readRetainedSlice(int)} which may return
 * a buffer implementation that produces less garbage.
 */
```

好的，我想你可以自己动手去测试一下，最好再看看源码，加深一下实现的原理印象。

这里的线程池并不是现实线程安全，而是用来做测试多线程的，Netty的Channel实现是线程安全的，所以我们可以存储一个到Channel的引用，并且每当我们需要向远程节点写数据时，都可以使用它，即使当时许多线程都在使用它，消息也会被保证按顺序发送的。

## 结语

最后，介绍一下，个人的一个基于Netty的开源项目：[InChat](https://github.com/UncleCatMySelf/InChat)

> 一个轻量级、高效率的支持多端（应用与硬件Iot）的异步网络应用通讯框架 

![Image text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/logo.png)

---------------

> 参考资料： 《Netty实战》