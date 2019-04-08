---
layout:       post
title:        如何在VMware12安装Centos7.6最新版
subtitle:     Github
date:         2019-04-08
author:       MySelf | 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - Github
---


> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出处

>本系列教程为[HMStrange项目](https://github.com/UncleCatMySelf/HMStrange)附带。

## ISO镜像下载

http://59.80.44.49/isoredirect.centos.org/centos/7/isos/x86_64/CentOS-7-x86_64-DVD-1810.iso

直接点击下载，或者进入下载地址，我这边选择的是第一个。

## 安装流程

* 1、选择典型安装模式即可

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos1.png)
* 2、选择安装的ISO文件地址，我选择的就是开头的链接地址

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos2.png)
* 3、你可以修改名字或者修改存放位置

![iamge](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos3.png)
* 4、容量这一块看情况，我这边直接默认的

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos4.png)
* 5、直接点击创建即可，一开始启动的时候需要默认配置，语言选择中文简体，因人而异。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos6.png)
* 6、接下来你会看到这个界面，你需要配置两个东西，一个是位置、一个是网络。

![image](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos7.png)
* 7、选择安装位置，这一块点进来就好，它会默认设定一次

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos8.png)
* 8、设置网络，修改主机名、选择应用、打开以太网，点击配置

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos9.png)
* 9、网络配置，选择常规，然后勾选第一个“可用时自动链接到这个网络”

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos10.png)
* 10、最后保存，在安装界面设定root的密码

![i](https://raw.githubusercontent.com/UncleCatMySelf/img_HMStrange/master/study/centos11.png)

安装成功！


进入界面后，大家可以输入

```
ip addr
```

查看网络ip，一般可以看到两个lo、ens33的信息。


#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)

