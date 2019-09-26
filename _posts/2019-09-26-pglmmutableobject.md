---
layout:       post
title:        程序员：并发下如何保证共享变量安全且不用锁？！
subtitle:     并发
date:         2019-09-26
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 并发
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处
>
> 阅读本文约 “15分钟”
>
> 适读人群：Java 中级

> 学习笔记，休息了两天（其实期间在做一个模拟项目实战），偶尔也想到自己究竟应该做些什么，是真的对自己或社会有意义的呢？

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/Visualhunt.jpg)
Photo on Visual hunt

## 说出你的回答

emmm，答案不止一个，今天先介绍一个简单易懂的

读题：我们应该如何保证共享变量访问的线程安全，同时又避免引入锁产生的开销呢

在并发环境下，一个对象是很容易被多个线程共享的，那么对于数据的一致性是有要求的

虽然可以使用显式锁或者CAS操作，不过这也会带来一些上下文切换等额外开销

先举个例子说明下目前的问题吧

```java
/**
 * @ClassName Cup
 * @Description 杯子 非线程安全
 * @Author MySelf
 * @Date 2019/9/25 21:28
 * @Version 1.0
 **/
public class Cup {

    //直径
    private double diameter;

    //高度
    private double height;

    public double getDiameter() {
        return diameter;
    }

    public double getHeight() {
        return height;
    }

    //非原子操作
    public void setCup(double diameter,double height){
        this.diameter = diameter;
        this.height = height;
    }
}
```

上面这段代码，大家应该都能看出是非线程安全的对吧（如果你看不出来，翻上一篇文章复习下）

因为在我们对setCup操作赋值其直径的时候，可能另一个线程已经开始读取他的高度了，那么这就会出现线程安全问题。

那么在不使用锁的情况，可以怎么做呢？

好好往下看呗，和刷朋友圈的时间差不多，一下子就懂了

## 不可变对象

是的，今天说的方式就是讲Cup变成不可变对象！

不可变对象：对象一经创建，其对外可见的状态就保持不变（类似String、Integer）

那么上面的Cup需要怎么修改呢？

```java
/**
 * @ClassName Cup
 * @Description 不可变对象，线程安全
 * @Author MySelf
 * @Date 2019/9/25 21:32
 * @Version 1.0
 **/
public final class Cup {

    private final double diameter;

    private final double height;

    public Cup(double diameter,double height){
        this.diameter = diameter;
        this.height = height;
    }

}
```

这下不就好啦，永远不会改变了

等下，那我要怎么修改Cup呀？就算是并发操作，我的业务也可能会需要修改这个Cup呀

让我们调整一下视野，修改Cup属性 == 替换Cup实例

假设我们是一家茶杯铸模工厂，有5条流水线在生成最近的网红茶杯，不过因为互联网趋势的印象，偶尔需要小改动咱们的这个茶杯参数，停机生产会亏本的，所以在模具适配器的代码上咱们可以在使用不可变对象的情况下更换茶杯属性

```java
/**
 * @ClassName MoldAdapter
 * @Description 模具适配器
 * @Author MySelf
 * @Date 2019/9/25 21:35
 * @Version 1.0
 **/
public class MoldAdapter {
    
    private Map<String,Cup> cupMap = new ConcurrentHashMap<String, Cup>();

    public void updateCup(String version,Cup newCup){
        cupMap.put(version, newCup);
    }

}
```
这里ConcurrentHashMap内部涉及的锁，和Demo中的茶杯新建、替换并无关系，其过程不涉及锁

可能还有点模糊，说说娃娃机案例？

## 还记得当年毁我青春的娃娃机吗？

记得很久以前还在泡老婆的时候，带她去玩娃娃机，夸下海口说一定能抓到她要的那一只，结果·····

![image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/thread/50.jpg)

它叫 **50**

现在轮到我们翻身做主人了，哼

假设我们是一个片区娃娃机的头儿，每个娃娃机都有他们对应的机器编号、支付二维码url、机械手频率（对，非职业机械工作者，这里给的是假设，这个才是赚钱的重点），假设我们是一个嗜钱如命的短裤青年，每晚都清算了一次收益清单。

最近恰逢国庆期间，游客人数即将上涨····

> 插入，特此【Java猫说】公众号提前预祝祖国70周年繁荣昌盛、国泰民安！

想赚钱的想法，搜搜搜的一直往胸口跳

那么好的手段就是娃娃机上的机械手频率了

我需要针对性的去修改部分娃娃机的属性，不过还好我一开始是有一张编码与娃娃机的关系映射表的

我将不可变对象的想法引入到自己的赚钱生意中去

首先是娃娃机对象，先变为不可变对象

```java
/**
 * @ClassName DollMachineInfo
 * @Description 娃娃机不可变对象
 * @Author MySelf
 * @Date 2019/9/25 21:51
 * @Version 1.0
 **/
public final class DollMachineInfo {

    //编号
    private final String number;

    //支付二维码url
    private final String url;

    //机械手频率
    private final int frequency;

    public DollMachineInfo(String number,String url,int frequency){
        this.number = number;
        this.url = url;
        this.frequency = frequency;
    }

    public DollMachineInfo(DollMachineInfo dollMachineInfoType){
        this.number = dollMachineInfoType.number;
        this.url = dollMachineInfoType.url;
        this.frequency = dollMachineInfoType.frequency;
    }

    public String getNumber() {
        return number;
    }

    public String getUrl() {
        return url;
    }

    public int getFrequency() {
        return frequency;
    }
}
```

这次我需要修改的是编码与娃娃机的关系映射表，所以这个表也需要是不可变的，他需要支持我获取关系映射表，而且需要替换最新的关系映射内容

```java
/**
 * @ClassName MachineRouter
 * @Description 机器信息表
 * @Author MySelf
 * @Date 2019/9/25 21:57
 * @Version 1.0
 **/
public final class MachineRouter {
    //保证其在并发环境的内存可见性
    private static volatile MachineRouter instance = new MachineRouter();
    //code与机器之间的映射关系
    private final Map<String,DollMachineInfo> routeMap;

    // 2、存储不可变量routeMap
    public MachineRouter(){
        //将数据库表中的数据加载到内存，存为Map
        this.routeMap = MachineRouter.setRouteFromeDB();
    }

    // 3、从db将数据存入Map
    private static Map<String, DollMachineInfo> setRouteFromeDB(){
        Map<String, DollMachineInfo> map = new HashMap<String, DollMachineInfo>();
        //DB 代码
        return map;
    }

    // 1、初始化实例
    public static MachineRouter getInstance(){
        return instance;
    }

    /**
     * 根据code获取对应的机器信息
     * @param code 对应编码
     * @return 机器信息
     */
    public DollMachineInfo getMacheine(String code){
        return routeMap.get(code);
    }

    /**
     * 修改当前MachineRouter实例
     * @param newInstance 新的实例
     */
    public static void setInstance(MachineRouter newInstance){
        instance = newInstance;
    }


    private static Map<String, DollMachineInfo> deepCopy(Map<String,DollMachineInfo> d){
        Map<String, DollMachineInfo> result = new HashMap<String, DollMachineInfo>();
        for (String key : d.keySet()){
            result.put(key, new DollMachineInfo(d.get(key)));
        }
        return result;
    }


    public Map<String, DollMachineInfo> getRouteMap() {
        //防御性复制
        return Collections.unmodifiableMap(deepCopy(routeMap));
    }
}
```

接下来就是在并发业务中去添加更新代码了

```java
/**
 * @ClassName Worker
 * @Description 通讯对接类
 * @Author MySelf
 * @Date 2019/9/25 22:13
 * @Version 1.0
 **/
public class Worker extends Thread {

    @Override
    public void run(){
        boolean isRouterModification = false;
        String updateMachineInfo = null;
        while (true){
            //其余业务代码
            /**
             * 在通讯的Socket信息中解析，并更新数据表信息，再重置MachineRouter实例
             */
            if (isRouterModification){
                if ("DollMachineInfo".equals(updateMachineInfo)){
                    MachineRouter.setInstance(new MachineRouter());
                }
            }
            //其余业务代码
        }
    }

}
```

## 敲黑板，记笔记

案例说完，有个基本概念，那么说点专业术语

这是不可变对象模式，Immutable Object

严格上说，不可变对象需要满足什么条件：

- 1、类本身有fianl：防止被子类修改定义的行为
- 2、所有字段用fianl修饰：可以在多线程下有JMM保证被修饰字段所引用对象的初始化安全
- 3、对象创建时，this关键字没有给到其他类
- 4、若引用了其他状态可变的对象（数组、集合），必须用private，不能对外暴露，需要返回字段，则进行防御性复制（Defensive Copy）

Immutable Object 模式有两个重要的东西，你们应该差不多知道的

**ImmutableObject：负责存储一组不可变状态**
    
- getState* ：返回ImmutableObject所维护的相关变量值，实例化时通过构造器的参数获得值
- getStateSnapshot：返回ImmutableObject维护的一组状态快照

**Manipulator：维护ImmutableObject的变更，当需要变更时，则参与生成新的ImmutableObject实例**

- changeStateTo：根据新的状态值生成新的ImmutableObject实例

**典型交互场景**

- 1、获取当前ImmutableObject的各个状态值
- 2、调用Manipulator的changeStateTo方法来更新应用状态
- 3、changeStateTo创建新的ImmutableObject实例以反映新的状态，并返回
- 4、获取新的ImmutableObject的状态快照

## 什么场景适合使用

是的，他确实可以满足我们的题目要求，不过任何一种设计模式都有其适合的场景

一般比较适合：

- 对象变化不频繁(娃娃机案例)
- 同时对数据组进行写操作，保证原子性（茶杯案例）
- 使用某个对象作为HashMap的key（注意对象的HashCode）

注意的几点：

- 对象变更频繁：会产生CPU消耗、GC也会有负担
- 防御性复制：避免外部代码修改其内部状态


## 专业案例

集合遍历在多线程环境经常被引入锁，以防止遍历过程中该集合内部结构被改变

java.util.concurrent.CopyOnWriteArrayList就使用了ImmutableObject模式

当然也是需要场景的，在遍历比修改操作更加频繁的场景

其内部维护一个array变量用于存储集合，在你添加一个元素时，它会生成一个新的数组，将集合元素复制到新数组，并在最后一个元素设置为添加的元素，且新数组复制给array，
即array引用的数组可以等效一个ImmutableObject，注意是等效

所以，在遍历CopyOnWriteArrayList时，直接根据array实例生成一个Iterator实例，无须加锁

## 结语

因为最后的CopyOnWriteArrayList我没有认真的看源码，所以就不细致展开讲，主要是大家可以理解不可变对象模式，最好可以写一个Demo出来，希望大家可以在生产环境中使用到这一理念，文笔拙劣，见谅。

> 我是MySelf，还在坚持学习技术与产品经理相关的知识，希望本文能给你带来新的知识点。

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/qrcode.jpg)