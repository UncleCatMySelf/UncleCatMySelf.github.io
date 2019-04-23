---
layout:       post
title:        RPC框架是啥之Apache CXF一款WebService RPC框架入门教程
subtitle:     架构
date:         2019-04-23
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
* [RPC框架是啥之Java自带RPC实现，RMI框架入门](https://unclecatmyself.github.io/2019/04/21/rpcaboutrmi/)

## Apache CXF一款WebService RPC框架入门教程

> CXF官网：`http://cxf.apache.org/`

Apache CXF是一个开源的`WebService RPC框架`，是由Celtix和Codehaus XFire合并而成的。它可以说是一个`功能齐全的集合`。

功能特性：

- 支持`Web Service标准`，包括SOAP（1.1、1.2）规范、WSI Basic Profile...等等我也不了解的，这里就不一一举例了。
- 支持`JSR相关规范和标准`，包括....同上。
- 支持`多种传输协议和协议绑定（SOAP、REST/HTTP、XML）、数据绑定（JAXB2.X、Aegis、Apache XML Beans）`。

### 还是先从案例入手吧

> 项目源码地址：[RPC_Demo](https://github.com/UncleCatMySelf/RPC_Demo)，记得是项目里面的`comgithubcxf`

- 1、使用IDEA构建一个maven项目，我选择了`maven-archetype-webapp`构建基本框架。当然你可能还需要创建一些目录

![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/cxf01.png)

- 2、我想是时候先配置好主要的`pom`文件了。

```xml
<?xml version="1.0" encoding="UTF-8"?>

<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>cxf</groupId>
  <artifactId>comgithubcxf</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>war</packaging>

  <name>comgithubcxf Maven Webapp</name>
  <!-- FIXME change it to the project's website -->
  <url>http://www.example.com</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <maven.compiler.source>1.7</maven.compiler.source>
    <maven.compiler.target>1.7</maven.compiler.target>
    <cxf.version>3.1.7</cxf.version>
    <spring.version>4.0.9.RELEASE</spring.version>
  </properties>

  <dependencies>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-webmvc</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.springframework</groupId>
      <artifactId>spring-context-support</artifactId>
      <version>${spring.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-frontend-jaxws</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-transports-http</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>org.apache.cxf</groupId>
      <artifactId>cxf-rt-transports-http-jetty</artifactId>
      <version>${cxf.version}</version>
    </dependency>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>4.11</version>
      <scope>test</scope>
    </dependency>
  </dependencies>

  <build>
    <finalName>comgithubcxf</finalName>
    <pluginManagement><!-- lock down plugins versions to avoid using Maven defaults (may be moved to parent pom) -->
      <plugins>
        <plugin>
          <artifactId>maven-clean-plugin</artifactId>
          <version>3.1.0</version>
        </plugin>
        <!-- see http://maven.apache.org/ref/current/maven-core/default-bindings.html#Plugin_bindings_for_war_packaging -->
        <plugin>
          <artifactId>maven-resources-plugin</artifactId>
          <version>3.0.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-compiler-plugin</artifactId>
          <version>3.8.0</version>
        </plugin>
        <plugin>
          <artifactId>maven-surefire-plugin</artifactId>
          <version>2.22.1</version>
        </plugin>
        <plugin>
          <artifactId>maven-war-plugin</artifactId>
          <version>3.2.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-install-plugin</artifactId>
          <version>2.5.2</version>
        </plugin>
        <plugin>
          <artifactId>maven-deploy-plugin</artifactId>
          <version>2.8.2</version>
        </plugin>
      </plugins>
    </pluginManagement>
  </build>
</project>
```

- 3、构建Server端还有其服务实现，接口使用`@WebService`注解，标明是一个`WebService远程服务接口`。

```java
package com.github.cxf.server;

import javax.jws.WebService;

/**
 * Create by UncleCatMySelf in 21:57 2019\4\23 0023
 */
@WebService
public interface CxfService {

    String say(String someOne);

}
```

在实现类上也同样加上，并通过`endpointInterface`标明对接的接口实现

```java
package com.github.cxf.server;

import javax.jws.WebService;

/**
 * Create by UncleCatMySelf in 21:57 2019\4\23 0023
 */
@WebService(endpointInterface = "com.github.cxf.server.CxfService")
public class CxfServiceImpl implements CxfService {
    @Override
    public String say(String someOne) {
        return someOne + ",Welcome to Study!";
    }
}
```

- 4、编写对应的cxf-server.xml文件（核心点），这里我参考了官网的案例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
        Licensed to the Apache Software Foundation (ASF) under one
        or more contributor license agreements. See the NOTICE file
        distributed with this work for additional information
        regarding copyright ownership. The ASF licenses this file
        to you under the Apache License, Version 2.0 (the
        "License"); you may not use this file except in compliance
        with the License. You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

        Unless required by applicable law or agreed to in writing,
        software distributed under the License is distributed on an
        "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
        KIND, either express or implied. See the License for the
        specific language governing permissions and limitations
        under the License.
-->
<!-- START SNIPPET: beans -->
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schemas/jaxws.xsd">
    <import resource="classpath:META-INF/cxf/cxf.xml"/>
    <import resource="classpath:META-INF/cxf/cxf-servlet.xml"/>
    <jaxws:endpoint id="helloWorld" implementor="com.github.cxf.server.CxfServiceImpl" address="/server"/>
</beans>
        <!-- END SNIPPET: beans -->
```

- 5、然后就是我们的`web.xml`文件了，

```xml
<!DOCTYPE web-app PUBLIC
 "-//Sun Microsystems, Inc.//DTD Web Application 2.3//EN"
 "http://java.sun.com/dtd/web-app_2_3.dtd" >

<web-app>
  <display-name>Archetype Created Web Application</display-name>
  <context-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:cxf-server.xml</param-value>
  </context-param>
  <listener>
    <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
  </listener>
  <servlet>
    <servlet-name>CXFServer</servlet-name>
    <servlet-class>org.apache.cxf.transport.servlet.CXFServlet</servlet-class>
    <load-on-startup>1</load-on-startup>
  </servlet>
  <servlet-mapping>
    <servlet-name>CXFServer</servlet-name>
    <url-pattern>/ws/*</url-pattern>
  </servlet-mapping>
</web-app>
```

- 6、配置tomcat，由于我是IDEA的环境，所有我就截图给大家看看

![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/cxf02.png)
![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/cxf03.png)

然后启动tomcat即可，如果一起正常的话，老干妈保佑！

- 7、访问测试服务端，这时我们可以访问`http://localhost:8080/ws/server?wsdl`，如果你看到了一下的画面，就是启动成功！

![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/cxf04.png)

- 8、服务端就先让它运行着，接着我们在同一个项目里面创建`客户端`的，这个比较简单，你可以先准备一个`cxf-client.xml`文件，配置对应的WebService服务接口，确定访问的地址，注意是HTTP地址哦，WebService就是采用HTTP协议通信的。

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!--
        Licensed to the Apache Software Foundation (ASF) under one
        or more contributor license agreements. See the NOTICE file
        distributed with this work for additional information
        regarding copyright ownership. The ASF licenses this file
        to you under the Apache License, Version 2.0 (the
        "License"); you may not use this file except in compliance
        with the License. You may obtain a copy of the License at

        http://www.apache.org/licenses/LICENSE-2.0

        Unless required by applicable law or agreed to in writing,
        software distributed under the License is distributed on an
        "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
        KIND, either express or implied. See the License for the
        specific language governing permissions and limitations
        under the License.
-->
<!-- START SNIPPET: beans -->
<beans xmlns="http://www.springframework.org/schema/beans" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:jaxws="http://cxf.apache.org/jaxws" xsi:schemaLocation=" http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans-4.2.xsd http://cxf.apache.org/jaxws http://cxf.apache.org/schema/jaxws.xsd">
    <bean id="client" class="com.github.cxf.server.CxfService" factory-bean="clientFactory" factory-method="create"/>
    <bean id="clientFactory" class="org.apache.cxf.jaxws.JaxWsProxyFactoryBean">
        <property name="serviceClass" value="com.github.cxf.server.CxfService"/>
        <property name="address" value="http://localhost:8080/ws/server"/>
    </bean>
</beans>
        <!-- END SNIPPET: beans -->
```

- 9、然后编写一个client的启动程序，并运行，我想你会成功的！因为我看到了下图！

```java
package com.github.cxf.client;

import com.github.cxf.server.CxfService;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Create by UncleCatMySelf in 21:56 2019\4\23 0023
 */
public class CxfClient {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("classpath:cxf-client.xml");
        CxfService client = (CxfService)context.getBean("client");
        System.out.println(client.say("MySelf"));
    }
}
```

![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/Architecture/cxf05.png)


**WebService 是一种跨平台的RPC技术协议。**

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)
