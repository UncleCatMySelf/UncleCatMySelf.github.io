---
layout:       post
title:        SpringBoot整合MyBatis并使用Redis作为缓存组件的Demo
subtitle:     Github
date:         2019-04-29
author:       HMStrange-TIAN
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Github
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

> 本系列教程为[HMStrange项目](https://github.com/UncleCatMySelf/HMStrange)附带。

> Auth：[HMStrange-TIAN](https://github.com/TIANTIANSTUDY) e-mail:zhangqihao@hnu.edu.cn

## 历史文章

* [如何在VMware12安装Centos7.6最新版](https://unclecatmyself.github.io/2019/04/08/vmwarecentos/)
* [Centos7.6安装Java8](https://unclecatmyself.github.io/2019/04/09/java8/)
* [Centos7.6安装MySQL+Redis（最新版）](https://unclecatmyself.github.io/2019/04/10/mysqlandredis/)
* [SpringBoot+MySQL+MyBatis的入门教程](https://unclecatmyself.github.io/2019/04/10/smm/)
* [SpringBoot+Redis的入门教程](https://unclecatmyself.github.io/2019/04/11/redis/)
* [Centos7.6安装4.0.8MongoDb教程](https://unclecatmyself.github.io/2019/04/12/mongodbinstall/)

## 安装流程

### 1、安装docker & redis

如果不清楚docker是什么，请查看docker的文档和简介，这里给出docker的安装过程

#### 1.1 安装虚拟机（如果有远程服务器的，请略过此步骤）
        
本文推荐VMvare，尽管vmvare比较臃肿，但是对于新手比较友好，配置很简单
从官网下载VMvare，官网地址：https://www.vmware.com/cn.html
从官网下载centos镜像文件，官网地址：https://www.centos.org/download/
打开VMvare创建虚拟机，导入镜像系统
Vmvare会自动配置，根据提示输入账户和密码之后，等待自动配置即可

#### 1.2 打开虚拟机的terminal，输入ifconfig查看ip地址，如图：

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/1.png)

#### 1.3 使用远程工具连接服务器，本文推荐使用Cygwin/SmartTTY/Putty/GitBash

打开连接工具，使用ssh root@192.168.xx.xx，登陆服务器即可操作

#### 1.4 安装docker 

- 1.4.1 检查内核版本，必须是3.10及以上

```js
uname -r
```

- 1.4.2 安装docker

```js
yum install docker
```

输入 y 确认安装

- 1.4.3 启动docker

```js
systemctl start docker 
```
查看docker时候安装成功
```js
docker -v
```
若有提示如：Docker version 1.12.6, build 3e8e77d/1.12.6，则安装成功

设置开机启动docker 
```js
systemctl enable docker
```

如果想停止docker(慎重！！！)
```js
systemctl stop docker
```

- 1.4.4 常见docker命令以及操作

a）镜像操作

检索镜像
```js
docker search keyword
```
例如：docker search mysql
拉取镜像
```js
docker pull iamges
```
例如：docker pull registry.docker-cn.com/library/mysql
查看镜像列表
```js
docker images
```
删除镜像
```js
docker rmi image(镜像)-id
```
b) 容器操作
根据拉取的镜像启动容器(可以docker images查看已有的镜像，启动需要的镜像)
```js
docker run --name mymysql -d mysql:latest
```
--name后面是容器的名字 -d 表示后台运行 latest是tag标签，表示最新版本
查看运行中的容器、
```js
docker ps
```
停止运行中的容器
```js
docker stop 容器的id
```
查看所有的容器
```js
docker ps ‐a
```
启动容器
```js
docker start 容器id
```
删除一个容器
```js
docker rm 容器id
```
启动一个做了端口映射的容器
```js
docker run ‐d ‐p 8080：8080
```
-d：后台运行 -p : 将主机的端口映射到容器的一个端口 主机端口：容器内容端口
更多命令和操作请查看docker官网
#### 1.5 使用docker 安装 redis
- 1.5.1 搜索镜像

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/2.png)

- 1.5.2 拉取镜像
```js
docker pull docker.io/redis
```
- 1.5.3 查看镜像

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/4.png)

- 1.5.4 运行镜像
```js
docker run -d -p 6379:6379 --name myredis docker.io/redis
```
- 1.5.5 查看运行中的镜像
```js
docker ps
```
此时，使用docker安装、运行镜像已经完成了

#### 1.6 使用RedisDesktopManager连接Redis数据库

下载地址：https://redisdesktop.com/download
设置连接名、主机名字（就是我们前面输入ifconfig查看得到的ip）、端口号（暴露的那个端口号就是用那个端口号，默认为6379）

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/3.png)

点击Tes tConnection 显示 successful 点击 OK 
更多关于redis的操作命令请查看官网：
http://www.redis.cn/

### 2、springboot整合mybatis

#### 2.1、打开IDEA，使用springboot Initializr 快速创建向导

- 点击下一步

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/5.png)

- 输入相应的Group、Artifact(不会的请先学习IDEA)

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/6.png)

- 选择相应的模块，如右下方红框所示

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/7.png)

- 输入项目name和项目address

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/8.png)

#### 2.2、创建完成后，可以看到pom文件中引入了相应的starter

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/9.png)

#### 2.3、在IDEA中配置mysql数据库

- 2.3.1配置mysql
新建数据库student，新建表student

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/10.png)

> 注：关于如何安装mysql、navicat以及如何使用请自行百度

- 2.3.2在项目的目录结构中找到application.properties或者新建一个application.yml(关于yml的语法请自行百度)
url的配置规则请百度，输入自己数据库的用户名和密码

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/11.png)

#### 2.4、编码：新建entity实体类、service、service的实现类、以及mapper接口，然后在resource目录下建立对应的mapper以及mabatis的配置文件

- 2.4.1项目目录结构如下：

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/12.png)

- 2.4.2 entity代码如下

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/13.png)


- 2.4.3 Service代码如下

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/14.png)


- 2.4.4 Service实现方法如下
> 注意：在实现方法上加 @Service注解

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/15.png)


- 2.4.5 mapper如下
> 注意:在接口上方加@mapper注解

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/16.png)


- 2.4.6 Controller如下
> 注意：加@RestConroller注解

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/17.png)


- 2.4.7 mapper映射文件如下
关于映射文件的语法，请查看官方文档，此处给出mybatis的中文文档：
http://www.mybatis.org/mybatis-3/zh/sqlmap-xml.html
resource/mybatis/mapper/StudentMapper.xml(此文件的路径)

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/18.png)


- 2.4.8 mybatias配置文件（这里没有作任何配置，但是这个文件一定要有）
resource/mybatis/mybatis-config.xml

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/19.png)


- 2.4.9 在application.properties配置mybatis
这两个配置是核心，其余配置可参考官方文档

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/20.png)


- 2.4.10 在student表中插入相关数据
如果不知道怎么插入数据，那么........请百度.........

#### 2.5 打开浏览器进行测试
结果如下：
此处用的google测试，也可以使用其他接口测试工具

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/21.png)


### 3、springboot整合redis

#### 3.1 在pom文件中引入redis 的坐标

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/22.png)


#### 3.2 在application.properties或者是application.yml中配置redis,host就是你的服务器的ip

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/23.png)


#### 3.3 在springboot的启动类开启缓存注解

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/24.png)


#### 3.4 新建redisConfig类配置redis

不要忘记加@Configuration，两个bean都是为了改变序列化的机制

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/25.png)


#### 3.5 在service的实现类上开启注解

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/26.png)


#### 3.6 测试结果，

- 3.6.1 先开启日志打印
红框内是mapper的相对路径

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/27.png)


- 3.6.2 第一次在浏览器请求会发现，控制台打印了sql语句

发起请求，在浏览器地址栏输入：
> http://127.0.0.1:8080/student/1

查看控制台

此时，student对象已被缓存到了redis中

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/28.png)


- 3.6.2 第二次从浏览器发起请求，发现控制台没有打印sql日志，说明缓存成功，使用RedisDesktopManager查看数据库

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/img/tian/29.png)


#### 9、项目下载地址

欢迎到HMStrange项目进行下载：https://github.com/UncleCatMySelf/HMStrange/tree/master/doc/demo/redisdemo

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)