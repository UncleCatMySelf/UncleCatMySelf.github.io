---
layout:       post
title:        关于属性描述符PropertyDescriptor
subtitle:     InChat
date:         2019-01-19
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 源码
---

> 本文首发于本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

## 前言

> 感谢GY丶L粉丝的提问：属性描述器PropertyDescriptor是干嘛用的？

本来我也没有仔细了解过**描述符**这一块的知识，不过粉丝问了，我就抽周末的时间看看，顺便学习一下，粉丝问的刚好是PropertyDescriptor这个属性描述符，我看了下源码。

```java
/**
 * A PropertyDescriptor describes one property that a Java Bean
 * exports via a pair of accessor methods.
 */
public class PropertyDescriptor extends FeatureDescriptor {
    //...
}
```

emmmm，假装自己英语能厉害的说，属性描述符描述了一个属性，即Java Bean 通过一对访问器方法来导出。（没错，他确实是存在于java.beans包下的）

![描述符](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E6%8F%8F%E8%BF%B0%E7%AC%A6.png)

通过类关系图，可以知道，我们应该提前了解一下**FeatureDescriptor**才行了。很好，起码目前还没有设计抽象类或者接口。

## FeatureDescriptor

```java
/**
 * The FeatureDescriptor class is the common baseclass for PropertyDescriptor,
 * EventSetDescriptor, and MethodDescriptor, etc.
 * <p>
 * It supports some common information that can be set and retrieved for
 * any of the introspection descriptors.
 * <p>
 * In addition it provides an extension mechanism so that arbitrary
 * attribute/value pairs can be associated with a design feature.
 */

public class FeatureDescriptor {
    //...
}
```

okay，这是很合理的设计方式，FeatureDescriptor为类似PropertyDescriptor、EvebtSetDescriptor、MethodDescriptor的描述符提供了一些共用的常量信息。同时它也提供一个扩展功能，方便任意属性或键值对可以于设计功能相关联。

这里简单的说下，在我大致看了一下源码后（可能不够详细，最近有点忙，时间较赶），FeatureDescriptor主要是针对一下属性的一些get/set，同时这些属性都是基本通用于PropertyDescriptor、EvebtSetDescriptor、MethodDescriptor。

```java
    private boolean expert; // 专有
    private boolean hidden; // 隐藏
    private boolean preferred; // 首选
    private String shortDescription; //简单说明
    private String name; // 编程名称
    private String displayName; //本地名称
    private Hashtable<String, Object> table; // 属性表
```

其实该类还有另外几个方法，比如深奥的构造函数等等，这里就不深入探讨了。

## PropertyDescriptor

那么我们大致知道了**FeatureDescriptor**，接下来就可以来深入了解看看这个属性描述符`PropertyDescriptor`。

说到属性，大家一定会想到的就是get/set这个些基础的东西，当我打开`PropertyDescriptor`源码的时候，我也看到了一开始猜想的点。

```java
    private final MethodRef readMethodRef = new MethodRef();
    private final MethodRef writeMethodRef = new MethodRef();
    private String writeMethodName;
    private String readMethodName;
```

这里的代码是我从源码中抽离的一部分，起码我们这样看可以大致理解，是分为写和读的步骤，那么就和我们初学java的get/set是一致的。

同时我还看到了，这个，及其注释。

```java
    // The base name of the method name which will be prefixed with the
    // read and write method. If name == "foo" then the baseName is "Foo"
    private String baseName;
```

这好像可以解释，为什么我们的属性在生成get/set的时候，第一个字母变成大写？！注释好像确实是这样写的。

由于可能需要一个Bean对象，所以我以前在案例中先创建了一个`Cat`类。

```java
public class Cat {

    private String name;

    private String describe;

    private int age;

    private int weight;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getDescribe() {
        return describe;
    }

    public void setDescribe(String describe) {
        this.describe = describe;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public int getWeight() {
        return weight;
    }

    public void setWeight(int weight) {
        this.weight = weight;
    }
}
```

### 构造函数

起码目前，我还不知道我应该怎么使用它，那么我们就一步一步来吧，我看到它有好几个构造函数，这是一个有趣而且有难度的事情，我们先试着创建一个`PropertyDescriptor`吧。

- 第一种构造函数

```java
    /**
     * Constructs a PropertyDescriptor for a property that follows
     * the standard Java convention by having getFoo and setFoo
     * accessor methods.  Thus if the argument name is "fred", it will
     * assume that the writer method is "setFred" and the reader method
     * is "getFred" (or "isFred" for a boolean property).  Note that the
     * property name should start with a lower case character, which will
     * be capitalized in the method names.
     *
     * @param propertyName The programmatic name of the property.
     * @param beanClass The Class object for the target bean.  For
     *          example sun.beans.OurButton.class.
     * @exception IntrospectionException if an exception occurs during
     *              introspection.
     */
    public PropertyDescriptor(String propertyName, Class<?> beanClass)
                throws IntrospectionException {
        this(propertyName, beanClass,
                Introspector.IS_PREFIX + NameGenerator.capitalize(propertyName),
                Introspector.SET_PREFIX + NameGenerator.capitalize(propertyName));
    }
```

这个好像是参数最少的，它只需要我们传入一个属性字符串，还有对应的类就好了，其实它也是调用了另一个构造函数，只是它会帮我们默认生成读方法和写方法。方法中的`Introspector.IS_PREFIX + NameGenerator.capitalize(propertyName)`其实就是自己拼出一个默认的get/set方法，大家有兴趣可以去看看源码。

那么对应的实现内容，我想大家应该都想到了。

```java
    public static void main(String[] args) throws Exception {
        PropertyDescriptor CatPropertyOfName = new PropertyDescriptor("name", Cat.class);
        System.out.println(CatPropertyOfName.getPropertyType());
        System.out.println(CatPropertyOfName.getPropertyEditorClass());
        System.out.println(CatPropertyOfName.getReadMethod());
        System.out.println(CatPropertyOfName.getWriteMethod());
    }
```

- 第二种构造函数

```java
/**
     * This constructor takes the name of a simple property, and method
     * names for reading and writing the property.
     *
     * @param propertyName The programmatic name of the property.
     * @param beanClass The Class object for the target bean.  For
     *          example sun.beans.OurButton.class.
     * @param readMethodName The name of the method used for reading the property
     *           value.  May be null if the property is write-only.
     * @param writeMethodName The name of the method used for writing the property
     *           value.  May be null if the property is read-only.
     * @exception IntrospectionException if an exception occurs during
     *              introspection.
     */
    public PropertyDescriptor(String propertyName, Class<?> beanClass,
                String readMethodName, String writeMethodName)
                throws IntrospectionException {
        if (beanClass == null) {
            throw new IntrospectionException("Target Bean class is null");
        }
        if (propertyName == null || propertyName.length() == 0) {
            throw new IntrospectionException("bad property name");
        }
        if ("".equals(readMethodName) || "".equals(writeMethodName)) {
            throw new IntrospectionException("read or write method name should not be the empty string");
        }
        setName(propertyName);
        setClass0(beanClass);

        this.readMethodName = readMethodName;
        if (readMethodName != null && getReadMethod() == null) {
            throw new IntrospectionException("Method not found: " + readMethodName);
        }
        this.writeMethodName = writeMethodName;
        if (writeMethodName != null && getWriteMethod() == null) {
            throw new IntrospectionException("Method not found: " + writeMethodName);
        }
        // If this class or one of its base classes allow PropertyChangeListener,
        // then we assume that any properties we discover are "bound".
        // See Introspector.getTargetPropertyInfo() method.
        Class[] args = { PropertyChangeListener.class };
        this.bound = null != Introspector.findMethod(beanClass, "addPropertyChangeListener", args.length, args);
    }
```

没错，这个构造函数就是第一种构造函数内部二次调用的，所需要的参数很简单，同时我也希望大家可以借鉴这个方法中的一些检测方式。这次的实现方式也是同样的形式。

```java
    public static void main(String[] args) throws Exception {
        PropertyDescriptor CatPropertyOfName = new PropertyDescriptor("name", Cat.class,"getName","setName");
        System.out.println(CatPropertyOfName.getPropertyType());
        System.out.println(CatPropertyOfName.getPropertyEditorClass());
        System.out.println(CatPropertyOfName.getReadMethod());
        System.out.println(CatPropertyOfName.getWriteMethod());
    }
```

- 第三种构造函数

```java
    /**
     * This constructor takes the name of a simple property, and Method
     * objects for reading and writing the property.
     *
     * @param propertyName The programmatic name of the property.
     * @param readMethod The method used for reading the property value.
     *          May be null if the property is write-only.
     * @param writeMethod The method used for writing the property value.
     *          May be null if the property is read-only.
     * @exception IntrospectionException if an exception occurs during
     *              introspection.
     */
    public PropertyDescriptor(String propertyName, Method readMethod, Method writeMethod)
                throws IntrospectionException {
        if (propertyName == null || propertyName.length() == 0) {
            throw new IntrospectionException("bad property name");
        }
        setName(propertyName);
        setReadMethod(readMethod);
        setWriteMethod(writeMethod);
    }
```

这个不用传类，因为你需要传递两个实际的方法进来，所以主要三个对应属性的参数既可。看看大致的实现内容

```java
    public static void main(String[] args) throws Exception {
        Class<?> classType = Cat.class;
        Method CatNameOfRead = classType.getMethod("getName");
        Method CatNameOfWrite = classType.getMethod("setName", String.class);
        PropertyDescriptor CatPropertyOfName = new PropertyDescriptor("name", CatNameOfRead,CatNameOfWrite);
        System.out.println(CatPropertyOfName.getPropertyType());
        System.out.println(CatPropertyOfName.getPropertyEditorClass());
        System.out.println(CatPropertyOfName.getReadMethod());
        System.out.println(CatPropertyOfName.getWriteMethod());
    }
```

好了，大致介绍了几种构造函数与实现方式，起码我们现在知道它需要什么。

## 一些使用方式

其实在我上面写一些构造函数的时候，我想大家应该已经感受到与反射相关了，起码我感觉上是这样的，所以我一开始想到这样的案例形式，通过反射与这个属性描述类去赋予我的类。

```java
    public static void main(String[] args) throws Exception {
        //获取类
        Class classType = Class.forName("com.example.demo.beans.Cat");
        Object catObj = classType.newInstance();
        //获取Name属性
        PropertyDescriptor catPropertyOfName = new PropertyDescriptor("name",classType);
        //得到对应的写方法
        Method writeOfName = catPropertyOfName.getWriteMethod();
        //将值赋进这个类中
        writeOfName.invoke(catObj,"river");
        Cat cat = (Cat)catObj;
        System.out.println(cat.toString());
    }
```

运行结果还是顺利的。

> Cat{name='river', describe='null', age=0, weight=0}

可以看到，我们确实得到了一个理想中的对象。

那么我是不是可以改变一个已经创建的对象呢？

```java
    public static void main(String[] args) throws Exception {
        //一开始的默认对象
        Cat cat = new Cat("river","黑猫",2,4);
        //获取name属性
        PropertyDescriptor catPropertyOfName = new PropertyDescriptor("name",Cat.class);
        //得到读方法
        Method readMethod = catPropertyOfName.getReadMethod();
        //获取属性值
        String name = (String) readMethod.invoke(cat);
        System.out.println("默认：" + name);
        //得到写方法
        Method writeMethod = catPropertyOfName.getWriteMethod();
        //修改值
        writeMethod.invoke(cat,"copy");
        System.out.println("修改后：" + cat);
    }
```

上面的demo是，我先创建了一个对象，然后通过属性描述符读取name值，再进行修改值，最后输出的对象的值也确实改变了。

> 默认：river
> 修改后：Cat{name='copy', describe='黑猫', age=2, weight=4}


## 收尾

这是一个有趣的API，我想另外两个（EvebtSetDescriptor、MethodDescriptor）应该也差不多，大家可以再通过此方法去探究，只有自己尝试一次才能学到这里面的一些东西，还有一些项目场景的使用方式，不过一般的业务场景应该很少使用到这个API。那么这个东西究竟可以干什么呢？我想你试着敲一次也许有一些答案了。


#### 公众号：Java猫说

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)

