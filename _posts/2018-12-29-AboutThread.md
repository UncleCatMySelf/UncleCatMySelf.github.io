---
layout:       post
title:        简说Java线程的那几个启动方式
subtitle:     Java并发
date:         2018-12-29
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - 并发
    - InChat
---

> 本文首发于本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

## 前言

并发是一件很美妙的事情，线程的调度与使用会让你除了业务代码外，有新的世界观，无论你是否参与但是这对于你未来的成长帮助很大。

所以，让我们来好好看看在Java中启动线程的那几个方式与介绍。

## Thread

对于 **Thread** 我想这个基本上大家都认识的，在Java源码是这样说： `java 虚拟机允许应用程序同时运行多个执行线程。` 而这个的 **Thread** 就是程序的执行线程。

如何使用它呢，其实在这个类中的源码已经给我们写好了，甚至是下面的 **Runnable** 的使用方式。（如下是Thread源码）

```java
/**
 * A <i>thread</i> is a thread of execution in a program. The Java
 * Virtual Machine allows an application to have multiple threads of
 * execution running concurrently.
 * <hr><blockquote><pre>
 *     class PrimeThread extends Thread {
 *         long minPrime;
 *         PrimeThread(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeThread p = new PrimeThread(143);
 *     p.start();
 * </pre></blockquote>
 * <p>
 * <hr><blockquote><pre>
 *     class PrimeRun implements Runnable {
 *         long minPrime;
 *         PrimeRun(long minPrime) {
 *             this.minPrime = minPrime;
 *         }
 *
 *         public void run() {
 *             // compute primes larger than minPrime
 *             &nbsp;.&nbsp;.&nbsp;.
 *         }
 *     }
 * </pre></blockquote><hr>
 * <p>
 * The following code would then create a thread and start it running:
 * <blockquote><pre>
 *     PrimeRun p = new PrimeRun(143);
 *     new Thread(p).start();
 * </pre></blockquote>
 * <p>
 */
public class Thread implements Runnable {
     //...
}
```

**阅读源码的信息其实是最全的** ，我截取了部分的注释信息，起码我们现在可以无压力的使用这个两个方式来启动自己的线程。

如果我们还要传递参数的话，那么我们设定一个自己的构造函数也是可以，如下方式：

```java
public class MyThread extends Thread {

    public MyThread(String name) {
        super(name);
    }

    @Override
    public void run() {
        System.out.println("一个子线程 BY " + getName());
    }
}
```

这时读者应该发现，这个构造函数中的 **name** ，居然在 **Thread** 中也是有的，其实在Java中的线程都会自己的名称，如果我们不给其定义名称的话，java也会自己给其命名。

```java
/**
* Allocates a new {@code Thread} object. This constructor has the same
* effect as {@linkplain #Thread(ThreadGroup,Runnable,String) Thread}
* {@code (null, null, name)}.
*
* @param   name
*          the name of the new thread
*/
public Thread(String name) {
    init(null, null, name, 0);
}
```

而我们最核心，也是大家最在意的应该就是如何启动并执行我们的线程了，是的，这个大家都知道的，就是这个 **run** 方法了。

同时大家如果了解过了 `Runnable` ，我想大家都会知道这个 **run** 方法，其实是 `Runnable` 的方法，而我们本节的 **Thread** 也是实现了这个接口。

这里，大家可能会好奇，不是应该是 **start** 这个方法吗？那么让我们看看 **start** 的源码。

```java
/**
 * Causes this thread to begin execution; the Java Virtual Machin
 * calls the <code>run</code> method of this thread.
 */
public synchronized void start() {
    //...
}
```

通过 `start` 方法，我们可以了解到，就如同源码的启动模板中那样，官网希望，对于线程的启动，使用者是通过 **start** 的方式来启动线程，因为这个方法会让Java虚拟机会调用这个线程的 **run** 方法。

其结果就是，一个线程去运行 `start` 方法，而另一个线程则取运行 `run` 方法。同时对于这样线程，Java官方也说了，**线程是不允许多次启动的，这是不合法的。**

所以如果我们执行下面的代码，就会报 `java.lang.IllegalThreadStateException` 异常。

```java
MyThread myThread = new MyThread("Thread");
myThread.start();
myThread.start();
```

但是，如果是这样的代码呢？

```java
MyThread myThread = new MyThread("Thread");
myThread.run();
myThread.run();
myThread.start();

//运行结果
一个子线程 BY Thread
一个子线程 BY Thread
一个子线程 BY Thread
```

这是不合理的，如果大家有兴趣，可以去试试并动手测试下，最好开调试模式。

下面我们再看看，连 **Thread** 都要实现，且核心的 **run** 方法出处的 `Runnable` 。

## Runnable

比起 **Thread** 我希望大家跟多的使用 **Runnable** 这个接口实现的方式，对于好坏对比会在总结篇说下。

我想大家看 `Runnable` 的源码会更加容易与容易接受，毕竟它有一个 **run** 方法。（如下为其源码）

```java
/**
 * The <code>Runnable</code> interface should be implemented by any
 * class whose instances are intended to be executed by a thread. The
 * class must define a method of no arguments called <code>run</code>.
 */
@FunctionalInterface
public interface Runnable {
    /**
     * When an object implementing interface <code>Runnable</code> is used
     * to create a thread, starting the thread causes the object's
     * <code>run</code> method to be called in that separately executing
     * thread.
     */
     public abstract void run();
}
```

首先，所有打算执行线程的类均可实现这个 `Runnable` 接口，且必须实现 **run** 方法。

它将为各个类提供一个协议，就像 **Thread** 一样，其实当我们的类实现了 **Runnable** 的接口后，我们的类与 **Thread** 是同级，只是可能仅有 `run` 方法，而没有 **Thread** 提供的跟丰富的功能方法。

而对于 **run** 方法，则是所有实现了 `Runnable` 接口的类，在调用 **start** 后，将使其单独执行 `run` 方法。

那么我们可以写出这样的测试代码。

```java
MyThreadRunnable myThreadRunnable = new MyThreadRunnable("Runnabel");
myThreadRunnable.run();
new Thread(myThreadRunnable).start();
Thread thread = new Thread(myThreadRunnable);
thread.start();
thread.start();

//运行效果
Exception in thread "main" java.lang.IllegalThreadStateException
	at java.lang.Thread.start(Thread.java:705)
	at com.github.myself.runner.RunnableApplication.main(RunnableApplication.java:14)
这是一个子线程 BY Runnabel
这是一个子线程 BY Runnabel
这是一个子线程 BY Runnabel
```

同样的，**线程是不允许多次启动的，这是不合法的。**

同时，这时我们也看出了使用 `Thread` 与 `Runnable` 的区别，**当我们要多次启用一个相同的功能时。**

我想 `Runnable` 更适合你。

**但是，用了这两个方式，我们要如何知道线程的运行结果呢？？？**

## FutureTask

这个可能很少人（初学者）用到，不过这个现在是我最感兴趣的。它很有趣。

其实还有一个小兄弟，那就是 `Callable`。 它们是一对搭档。如果上面的内容，你已经细细品味过，那么你应该已经发现 `Callable` 了。

没错，他就在 `Runnable` 的源码中出现过。

```java
/**
 * @author  Arthur van Hoff
 * @see     java.lang.Thread
 * @see     java.util.concurrent.Callable
 * @since   JDK1.0
 */
 @FunctionalInterface
public interface Runnable {}
```

那么我们先去看看这个 `Callable` 吧。（如下为其源码）

```java
/**
 * A task that returns a result and may throw an exception.
 * Implementors define a single method with no arguments called
 * {@code call}.
 */
@FunctionalInterface
public interface Callable<V> {
    /**
     * Computes a result, or throws an exception if unable to do so.
     *
     * @return computed result
     * @throws Exception if unable to compute a result
     */
    V call() throws Exception;
}
```

其实，这是一个与 `Runnable` 基本相同的接口，当时它可以返回执行结果与检查异常，其计算结果将由 `call()` 方法返回。

那么其实我们现在可以写出一个实现的类。

```java
public class MyCallable implements Callable {

    private String name;

    public MyCallable(String name) {
        this.name = name;
    }

    @Override
    public Object call() throws Exception {
        System.out.println("这是一个子线程 BY " + name);
        return "successs";
    }
}
```

> 关于更深入的探讨，我将留到下一篇文章中。

好了，我想我们应该来看看 `FutureTask` 这个类的相关信息了。

```java
/**
 * A cancellable asynchronous computation.  This class provides a base
 * implementation of {@link Future}, with methods to start and cancel
 * a computation, query to see if the computation is complete, and
 * retrieve the result of the computation.  The result can only be
 * retrieved when the computation has completed; the {@code get}
 * methods will block if the computation has not yet completed.  Once
 * the computation has completed, the computation cannot be restarted
 * or cancelled (unless the computation is invoked using
 * {@link #runAndReset}).
 */
 public class FutureTask<V> implements RunnableFuture<V> {
     //...
 }
```

源码写的很清楚，这是一个可以取消的异步计算，提供了查询、计算、查看结果等的方法，同时我们还可以使用 `runAndRest` 来让我们可以重新启动计算。

在查看其构造函数的时候，很高兴，我们看到了我们的 `Callable` 接口。

```java
/**
 * Creates a {@code FutureTask} that will, upon running, execute the
 * given {@code Callable}.
 *
 * @param  callable the callable task
 * @throws NullPointerException if the callable is null
 */
public FutureTask(Callable<V> callable) {
    if (callable == null)
        throw new NullPointerException();
    this.callable = callable;
    this.state = NEW;       // ensure visibility of callable
}
```

即我们将创建一个未来任务，来执行 `Callable` 的实现类。那么我们现在可以写出这样的代码了。

```java
final FutureTask fun = new FutureTask(new MyCallable("Future"));
```

那么接下来我们就可以运行我们的任务了吗？

是的，我知道了 `run()` 方法，但是却没有 `start` 方法。

官方既然说有结果，那么我找到了 `get` 方法。同时我尝试着写了一下测试代码。

```java
public static void main(String[] args) {
    MyCallable myCallable = new MyCallable("Callable");
    final FutureTask fun = new FutureTask(myCallable);
    fun.run();
    try {
        Object result = fun.get();
        System.out.println(result);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

运行效果，是正常的，这好像是那么回事。

```js
//运行效果
这是一个子线程 BY Callable
successs
```

可是，在我尝试着加多一些代码的时候，却发现了**一些奇妙的东西** 。

我加多了一行 `fun.run();` 代码，同时在 `MyCallable` 类中，将方法加一个时间线程去等待3s。

结果是： **结果只输出了一次，同时 `get` 方法需要等运行3s后才有返回。**

这并不是我希望看到的。但是，起码我们可以知道，这次即使我们多次运行使用 `run` 方法，但是这个线程也只运行了一次。这是一个好消息。

同时，我们也拿到了任务的结果，当时我们的进程被阻塞了，我们需要去等我们的任务执行完成。

最后，在一番小研究后，以下的代码终于完成了我们预期的期望。

```java
public static void main(String[] args) {
    MyCallable myCallable = new MyCallable("Callable");
    ExecutorService executorService = Executors.newCachedThreadPool();
    final FutureTask fun = new FutureTask(myCallable);
    executorService.execute(fun);
//        fun.run();  //阻塞进程
    System.out.println("--继续执行");
    try {
        Object result = fun.get();
        System.out.println(result);
    } catch (InterruptedException e) {
        e.printStackTrace();
    } catch (ExecutionException e) {
        e.printStackTrace();
    }
}
```

我们使用线程池去运行我们的 `FutureTask` 同时使用 `get` 方法去获取运行后的结果。结果是友好的，进程并不会被阻塞。

> 关于更深入的探讨，我将留到下一篇文章中。

## 总结一波

好了，现在应该来整理以下了。

- **Thread** 需要我们继承实现，这是比较局限的，因为Java的 **继承资源** 是有限的，同时如果多次执行任务，还需要 **多次创建任务类**。
- **Runnable** 以接口的形式让我们实现，较为方便，同时多次执行任务也无需创建多个任务类，当时仅有一个 `run` 方法。
- 以上两个方法都 **无法获取任务执行结果** ，FutureTask可以获取任务结果。同时还有更多的新特性方便我们使用···

## 公众号：Java猫说

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/qrcode.jpg)