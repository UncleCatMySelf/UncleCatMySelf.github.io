---
layout:       post
title:        从零讲解搭建一个NIO消息服务端
subtitle:     Netty知识点
date:         2018-12-26
author:       MySelf | 猫叔
header-img:   img/elage.jpg
catelog:      true
tags:
    - Netty
    - InChat
    - NIO
---

> 本文首发于本博客，如需转载，请申明出处.

## 假设

假设你已经了解并实现过了一些OIO消息服务端，并对异步消息服务端更有兴趣，那么本文或许能带你更好的入门，并了解JDK部分源码的关系流程，正如题目所说，笔者将竟可能还原，以初学者能理解的角度，讲诉并构建一个NIO消息服务端。

## 启动通道并注册选择器


### 启动模式


感谢Java一直在持续更新，对应的各个API也做得越来越好了，我们本次生成 **服务端套接字通道** 也是使用到JDK提供的一个方式 **open** ，我们将启动一个 **ServerSocketChannel** ，他是一个 **支持同步异步模式** 的 **服务端套接字通道** 。

它是一个抽象类，官方给了推荐的方式 **open** 来开启一个我们需要的 **服务端套接字通道实例** 。（如下的官方源码相关注释）

```java
/**
 * A selectable channel for stream-oriented listening sockets.
 */
 public abstract class ServerSocketChannel
    extends AbstractSelectableChannel
    implements NetworkChannel
{
    /**
     * Opens a server-socket channel.
     */
    public static ServerSocketChannel open() throws IOException {
        return SelectorProvider.provider().openServerSocketChannel();
    }
}
```

那么好了，我们现在可以确定我们第一步的代码是什么样子的了！没错，和你想象中的一样，这很简单。

```java
public class NioServer {

    public void server(int port) throws IOException{
        //1、打开服务器套接字通道
        ServerSocketChannel serverSocketChannel = ServerSocketChannel.open();
    }
}
```

本节的重点是 **启动模式** ，那么这意味着，我们需要向 **ServerSocketChannel** 进行标识，那么它是否提供了对用的方法设置 **同步异步（阻塞非阻塞）** 呢？

这很明显，它是提供的，这也是它的核心功能之一，其实应该是它继承的 **父抽象类AbstractSelectableChannel** 的实现方法： **configgureBlocking（Boolean）**，这个方法将标识我们的 **服务端套接字通道** 是否阻塞模式。（如下的官方源码相关注释）

```java
/**
 * Base implementation class for selectable channels.
 */
 public abstract class AbstractSelectableChannel
    extends SelectableChannel
{
    /**
     * Adjusts this channel's blocking mode.
     */
    public final SelectableChannel configureBlocking(boolean block)
        throws IOException
    {
        synchronized (regLock) {
            if (!isOpen())
                throw new ClosedChannelException();
            if (blocking == block)
                return this;
            if (block && haveValidKeys())
                throw new IllegalBlockingModeException();
            implConfigureBlocking(block);
            blocking = block;
        }
        return this;
    }
}
```

那么，我们现在可以进行 **启动模式的配置** 了，读者很聪明。我们的项目Demo可以这样写： **false为非阻塞模式、true为阻塞模式** 。

```java
public class NioServer {

    public void server(int port) throws IOException{
        //1、打开服务器套接字通道
        ServerSocketChannel serverSocketzhannel = ServerSocketChannel.open();
        //2、设定为非阻塞、调整此通道的阻塞模式。
        serverSocketChannel.configureBlocking(false);
    }
}
```

> 若未配置阻塞模式，**注册选择器** 会报 `java.nio.channels.IllegalBlockingModeException` 异常，相关将于该小节大致讲解说明。

### 套接字地址端口绑定

做过消息通讯服务器的朋友应该都清楚，我们需要向服务端 **指定IP与端口** ，即使是NIO服务器也是一样的，否则，我们的客户端会报 `java.net.ConnectException: Connection refused: connect` 异常

对于NIO的地址端口绑定，我们也需要用到 **ServerSocket服务器套接字** 。我们知道在写OIO服务端的时候，我们可能仅仅需要写一句即可，如下。

```java
    //将服务器绑定到指定端口
    final ServerSocket socket = new ServerSocket(port);
```

当然，JDK在实现NIO的时候就已经想到了，同样，我们可以使用 **服务器套接字通道** 来获取一个 **ServerSocket服务器套接字** 。这时的它并没有绑定端口，我们需要对应绑定地址，这个类自身就有一个 **bind** 方法。（如下源码相关注释）

```java
/**
 * This class implements server sockets. A server socket waits for
 * requests to come in over the network. It performs some operation
 * based on that request, and then possibly returns a result to the requester.
 */
public class ServerSocket implements java.io.Closeable {
    /**
     *
     * Binds the {@code ServerSocket} to a specific address
     * (IP address and port number).
     */
     public void bind(SocketAddress endpoint) throws IOException {
        bind(endpoint, 50);
    }
}
```

通过源码，我们知道，**绑定iP与端口** 需要一个SocketAddress类，我们仅需要将 **IP与端口配置到对应的SocketAddress类** 中即可。其实JDK中，已经有了一个更加方便且继承了SocketAddress的类：**InetSocketAddress**。

InetSocketAddress有一个需要一个port为参数的构造方法，它将创建 **一个ip为通配符、端口为指定值的套接字地址** 。这很方便我们的开发，对吧？(如下源码相关注释)

```java
/**
 *
 * This class implements an IP Socket Address (IP address + port number)
 * It can also be a pair (hostname + port number), in which case an attempt
 * will be made to resolve the hostname. If resolution fails then the address
 * is said to be <I>unresolved</I> but can still be used on some circumstances
 * like connecting through a proxy.
 */
 public class InetSocketAddress
    extends SocketAddress
{
    /**
     * Creates a socket address where the IP address is the wildcard address
     * and the port number a specified value.
     */
     public InetSocketAddress(int port) {
        this(InetAddress.anyLocalAddress(), port);
    }
}
```

好了，那么接下来我们的项目代码可以继续添加绑定IP与端口了，我想聪明的你应该有所感觉了。

```java
public class NioServer {

    public void server(int port) throws IOException{
        //1、打开服务器套接字通道
        ServerSocketChannel serverSocketzhannel = ServerSocketChannel.open();
        //2、设定为非阻塞、调整此通道的阻塞模式。
        serverSocketChannel.configureBlocking(false);
        //3、检索与此通道关联的服务器套接字。
        ServerSocket serverSocket = serverSocketChannel.socket();
        //4、此类实现 ip 套接字地址 (ip 地址 + 端口号) 
        InetSocketAddress address = new InetSocketAddress(port);
        //5、将服务器绑定到选定的套接字地址
        serverSocket.bind(address);
    }
}
```

正如开头我们所说的，你的项目中不添加3-5环节的代码并没有问题，但是当客户端接入时，则会报错，因为客户端将要 **接入的地址是连接不到的** ，如会报这样的错误。

```js
java.net.ConnectException: Connection refused: connect
	at sun.nio.ch.Net.connect0(Native Method)
	at sun.nio.ch.Net.connect(Net.java:457)
	at sun.nio.ch.Net.connect(Net.java:449)
	at sun.nio.ch.SocketChannelImpl.connect(SocketChannelImpl.java:647)
	at com.github.myself.WebClient.main(WebClient.java:16)
```

### 注册选择器

接下来会是 **NIO实现的重点** ，可能有点难理解，如果希望大家能一次理解，完全深入有点难讲明白，不过先大致点一下。

首先要先介绍以下JDK实现NIO的核心：**多路复用器（Selector）——选择器**

先简单并抽象的理解下，Java通过 **选择器来实现处理多个Channel链接** ，将空闲未进行数据操作的搁置，优先执行有需求的数据传输，即 **通过一个选择器来选择谁需要谁不需要使用共享的线程** 。

由此，理所当然，这样的选择器应该也有Java自己定义的获取方法， 其自身的 **open** 就是启动一个这样的选择器。（如下源码相关注释）

```java
/**
 * A multiplexor of {@link SelectableChannel} objects.
 */
 public abstract class Selector implements Closeable {
     /**
     * Opens a selector.
     */
     public static Selector open() throws IOException {
        return SelectorProvider.provider().openSelector();
     }
 }
```

那么现在，我们还要考虑一件事情，我们的 **服务器套接字通道** 要如何与 **选择器** 相关联呢？

**ServerSocketChannel** 有一个注册的方法，这个方法就是将它们两个进行了关联，同时这个注册方法 **除了关联选择器外，还标识了注册的状态** ，让我们先看看源码吧。


> 以下的 ServerSocketChannel 继承 ---》 AbstractSelectableChannel 继承 ---》 SelectableChannel


```java
/**
 * A channel that can be multiplexed via a {@link Selector}.
 */
 public abstract class SelectableChannel
    extends AbstractInterruptibleChannel
    implements Channel
{
    /**
     * Registers this channel with the given selector, returning a selection
     * key.
     */
     public final SelectionKey register(Selector sel, int ops)
        throws ClosedChannelException
    {
        return register(sel, ops, null);
    }
}
```

我们一般需要将选择器注册上去，并将 **ServerSocketChannel** 标识为 **接受连接** 的状态。我们先看看我们的项目代码应该如何写。

```java
public class NioServer {

    public void server(int port) throws IOException{
        //1、打开服务器套接字通道
        ServerSocketChannel serverSocketzhannel = ServerSocketChannel.open();
        //2、设定为非阻塞、调整此通道的阻塞模式。
        serverSocketChannel.configureBlocking(false);
        //3、检索与此通道关联的服务器套接字。
        ServerSocket serverSocket = serverSocketChannel.socket();
        //4、此类实现 ip 套接字地址 (ip 地址 + 端口号) 
        InetSocketAddress address = new InetSocketAddress(port);
        //5、将服务器绑定到选定的套接字地址
        serverSocket.bind(address);
        //6、打开Selector来处理Channel
        Selector selector = Selector.open();
        //7、将ServerSocket注册到Selector已接受连接，注册会判断是否为非阻塞模式
        SelectionKey selectionKey = serverSocketChannel.register(selector, SelectionKey.OP_ACCEPT);
        ByteBuffer readBuff = ByteBuffer.allocate(1024);
        final ByteBuffer msg = ByteBuffer.wrap("Hi!\r\n".getBytes());
        while(true){
            //下方代码.....
        }
    }
}
```

**注意：** 我们前面说到，如果 **ServerSocketChannel** 没有启动非阻塞模式，那么我们在启动的时候会报 `java.lang.IllegalArgumentException` 异常，这是为什么呢？ 我想我们可能需要更深入底层去看看 **register** 这个方法（如下源码注释）

```java
/**
 * Base implementation class for selectable channels.
 */
 public abstract class AbstractSelectableChannel
    extends SelectableChannel
{
    /**
     * Registers this channel with the given selector, returning a selection key.
     */
    public final SelectionKey register(Selector sel, int ops,
                                       Object att)
        throws ClosedChannelException
    {
        synchronized (regLock) {
            if (!isOpen())
                throw new ClosedChannelException();
            if ((ops & ~validOps()) != 0)
                throw new IllegalArgumentException();
            if (blocking)
                throw new IllegalBlockingModeException();
            SelectionKey k = findKey(sel);
            if (k != null) {
                k.interestOps(ops);
                k.attach(att);
            }
            if (k == null) {
                // New registration
                synchronized (keyLock) {
                    if (!isOpen())
                        throw new ClosedChannelException();
                    k = ((AbstractSelector)sel).register(this, ops, att);
                    addKey(k);
                }
            }
            return k;
        }
    }
}
```

我想我们终于真相大白了，原来注册这个方法会对 **ServerSocketChannel** 的一系列参数进行 **校验** ，只有通过，才能注册成功，所以我们也明白了，为什么 **非阻塞是false**，同时我们也可以看到，它还对我们所给的标识做了校验，一点要优先注册 **接受连接（OP_ACCEPT）** 这个状态才行，不然依旧会报 `java.lang.IllegalArgumentException` 异常。

这里解释一下，之所以只接受 **OP_ACCEPT** ，是因为如果没有一个接受其他链接的主服务，那么通信根本无从说起，同时这样的标识在我们的NIO服务端中 **只允许标识一次（一个ServerSocketChannel）** 。 

可能大家还会好奇有什么标识，我想源码的说明确实写的很清楚了。

```java
/**
 * Operation-set bit for read operations.
 */
 public static final int OP_READ = 1 << 0;

/**
 * Operation-set bit for write operations.
 */
 public static final int OP_WRITE = 1 << 2;

/**
 * Operation-set bit for socket-connect operations.
 */
 public static final int OP_CONNECT = 1 << 3;

/**
 * Operation-set bit for socket-accept operations.
 */
 public static final int OP_ACCEPT = 1 << 4;
```

好了，这里给一个调试截图，希望大家也可以慢慢的摸索一下。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/%E8%B0%83%E8%AF%95.png)

> 注意这里的服务端并没有构建完成哦，我们还需要下面的几个步骤。

## NIO选择实例与兴趣点

### 客户端代码

说到这里，我们暂时先休息下，转头看看 **客户端的代码** 吧，这里就简单的介绍下，我们将建立 **一个针对服务地址端口的连接** ，然后不停的循环 **写操作与读操作** ，没有对客户端进行 **关闭操作**。

大家如果有兴趣的话，也可以自己调试，并看看部分类的JDK源码，如下给出本项目案例的客户端代码。

```java
public class WebClient {
    public static void main(String[] args) throws IOException {
        try {
            SocketChannel socketChannel = SocketChannel.open();
            socketChannel.connect(new InetSocketAddress("0.0.0.0",8090));

            ByteBuffer writeBuffer = ByteBuffer.allocate(32);
            ByteBuffer readBuffer = ByteBuffer.allocate(32);

            writeBuffer.put("hello".getBytes());
            writeBuffer.flip();
            while (true){
                writeBuffer.rewind();
                socketChannel.write(writeBuffer);
                readBuffer.clear();
                socketChannel.read(readBuffer);
                readBuffer.flip();
                System.out.println(new String(readBuffer.array()));
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

### 准备IO接入操作

这里有点复杂，我也尽可能的思考了表达的方式，首先我们先明确一下，所有的连接都会被Selector所囊括，即我们要获取新接入的连接，也要通过Selector来获取，我们一开始启动的 **服务器套接字通道ServerSocketChannel** 起到一个接入\入口（或许不够准确）的作用，客户端连接通过IP与端口进入后，会 **被注册的Selector所获取** 到，成为 **Selector** 其中的一员。

但是这里的一员 **并不会包括一开始注册并被标志为接收连接** 的 **ServerSocketChannel** 。

Selector有这样一个方法，它会自动去等待新的连接事件，如果没有连接接入，那么它将一直处于阻塞状态。通过字面意思我们可以大致这样写代码。

```java
while(true){
    try{
        //1、等到需要处理的新事件：阻塞将一直持续到下一个传入事件
        selector.select();
    }catch(IOException e){
        e.printStackTrace();
        break;
    }
}
```

那么这样写好像有点像样，毕竟异常我们也捕获了，同时也使用了刚刚 **开启并注册完毕的选择器Selector**。

让我们看看源码中对于这个方法 **select** 的注释吧。

```java
/**
 * A multiplexor of {@link SelectableChannel} objects.
 */
 public abstract class Selector implements Closeable {
     /**
     * Selects a set of keys whose corresponding channels are ready for I/O
     * operations.
     */
     public abstract int select() throws IOException;
 }
```

好的，看样子是对的，它将返回一组套接字通道已经准备好执行I/O操作的键。那么这个Key究竟是什么呢？

这里可能直观的感受下会更好。如下图是我调试下看到的key对象，我想大家应该可以理解了，这个Key中也会 **存放对应连接的Channel与Selector** 。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/key.png)

具体的内部更深层的就探讨了。那么这也解决了我们接下来的 **一个疑问** ，我们要怎么向Selector拿连接进来的实例呢？

答案很明显，我们仅需要 **获取到这个Keys** 就好了。

### 选择键集合操作

对于获取Keys这个现在应该已经不是什么问题了，通过上面章节的了解，我想大家也可以想到这样的大致语法。

```java
//获取所有接收事件的SelectionKey实例
Set<SelectionKey> readykeys = selector.selectedKeys();
```

大家或许会好奇，这里的Key对象居然是前面的 `SelectionKey.OP_ACCEPT` 对象，是的，这也是接下来要讲的，这很奇妙，也很好玩。

前面说到的标识，这是每一个Key自有的，并且是可以 **改变的状态** ，在刚刚连接的时候，或许我应该大致的描述一下 **一个新连接进入选择器后的流程** ：select方法将接受到新接入的连接事件，它会被Selector以Key的形式存储，这时我们需要 **对其进行判断** ，是否是已经就绪可以被接受的连接，如果是，这时我们需要 **获取这个连接** ，同时也将其设定为 **非阻塞的状态** ，并将它 **注册到选择器上（当然，这时的标识就不能是一开始的 `OP_ACCEPT` ）**，你可以选择性的 **注册它的标识** ，之后我们可以通过循环遍历Keys来，让 **某一标识的连接去执行对应的操作** 。

说到这里，我想部分新手可能会有点模糊，我想我还是把接下来的代码都一起放出来吧，大家先看看是否能够再次结合文本进行了解。

```java
while (true){
    try {
        //等到需要处理的新事件：阻塞将一直持续到下一个传入事件
        selector.select();
    }catch (IOException e){
        e.printStackTrace();
        break;
    }
    //获取所有接收事件的SelectionKey实例
    Set<SelectionKey> readykeys = selector.selectedKeys();
    Iterator<SelectionKey> iterator = readykeys.iterator();
    while(iterator.hasNext()){
        SelectionKey key = iterator.next();
        iterator.remove();
        try {
            //检查事件是否是一个新的已经就绪可以被接受的连接
            if (key.isAcceptable()){
                //channel：返回为其创建此键的通道。 即使在取消密钥后, 此方法仍将继续返回通道。
                ServerSocketChannel server = (ServerSocketChannel)key.channel();
                //可选择的通道, 用于面向流的连接插槽。
                SocketChannel client = server.accept();
                //设定为非阻塞
                client.configureBlocking(false);
                //接受客户端，并将它注册到选择器，并添加附件
                client.register(selector,SelectionKey.OP_WRITE | SelectionKey.OP_READ,msg.duplicate());
                System.out.println("Accepted connection from " + client);
            }
            //检查套接字是否已经准备好读数据
            if (key.isReadable()){
                SocketChannel client = (SocketChannel)key.channel();
                readBuff.clear();
                client.read(readBuff);
                readBuff.flip();
                System.out.println("received:"+new String(readBuff.array()));
                //将此键的兴趣集设置为给定的值。 OP_WRITE
                key.interestOps(SelectionKey.OP_WRITE);
            }
            //检查套接字是否已经准备好写数据
            if (key.isWritable()){
                SocketChannel client = (SocketChannel)key.channel();
                //attachment : 检索当前附件
                ByteBuffer buffer = (ByteBuffer)key.attachment();
                buffer.rewind();
                client.write(buffer);
                //将此键的兴趣集设置为给定的值。 OP_READ
                key.interestOps(SelectionKey.OP_READ);
            }
        }catch (IOException e){
            e.printStackTrace();
        }
    }
}
```

> 提示：读到此处，还请各位读者能运行整个demo，并调试下，看看与自己理解的是否有差别。

### 流程效果

以下我简单叙述一下，我在调试时的理解与效果。

- 1、启动服务端后，运行到 `selector.select();` 后阻塞，因为没有监听到新的连接。

- 2、启动客户端后，`selector.select()` 监听到新连接，往下执行获取到的Keys的size为1，进入Key标识分支判断

- 3、`key.isAcceptable()` 首次接入为true，设置为非阻塞，并注释到选择器中修改标识为 `SelectionKey.OP_WRITE | SelectionKey.OP_READ` ，同时添加附件信息 `msg.duplicate()` ，首次循环结束

- 4、二次循环，连接未关闭，获取到的Keys的size为1，进入Key标识分支判断。

- 5、由于第一次该Key标识改变，所以这次 `key.isAcceptable()` 为false，而由于改了标识，所以接下来的 `key.isReadable()` 、 `key.isWritable()` 都为true，执行读写操作，循环结束。

- 6、接下来的循环，基本上是`key.isReadable()` 、 `key.isWritable()` 都为true，执行读写操作。

- 7、设想一下，如果多加一条链接是什么效果。

## 回顾

这里给出几个代码的注意点，希望大家可以自己去了解学习。

- 1、关于 **ByteBuffer** 本文并不重点讲解，大家可以自行了解
- 2、关于Key标识判断的代码，以下两句的删减是否会对代码有所影响呢？

```java
key.interestOps(SelectionKey.OP_WRITE);
key.interestOps(SelectionKey.OP_READ);
```

- 3、如果删除了2中的代码，并把客户端注册选择器并给标识的代码改为以下，那么项目运行效果怎么样呢？

```java
client.register(selector, SelectionKey.OP_READ,msg.duplicate());
```

- 4、如果改了3的代码，可是不删除2的代码，那么效果又是怎么样呢？

> 答案留给读者去揭晓吧，如果你有答案，欢迎留言。

## 个人相关项目

![Image text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/inchat/logo.png)

[InChat ： 一个轻量级、高效率的支持多端（应用与硬件Iot）的异步网络应用通讯框架](https://github.com/UncleCatMySelf/InChat)
