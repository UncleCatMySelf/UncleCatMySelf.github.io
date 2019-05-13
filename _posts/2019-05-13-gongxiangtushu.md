---
layout:       post
title:        没有实战经验？从零敲一个企业级共享项目前后端！
subtitle:     GitHub
date:         2019-05-13
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - GitHub
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出

> 阅读本文约“3分钟”
> 适读人群：Java后端、Java初级、小程序前端

本文是两个GitHub项目的序章，旨在指导初级程序员完成一个企业级共享项目的前后端代码实践，丰富自身的实战经验与知识。

项目介绍，这个一个企业级的共享图书项目，涉及部分Iot实践环节，整个项目主要以SpringBoot为后台提供API，前端小程序调用接口，同时项目会涉及共享书柜硬件的通信环节，其中涉及netty知识，整个项目大致的技术栈应该会有小程序源码MVC开发模式、ES6基
础能力提升、共享书柜二维码生成、图书管理系统、图书业务知识、netty构建简易Iot通信，SpringBoot实现基本的业务功能。

业务具体介绍，本系统是一个共享图书的小程序项目，企业级，创业项目。类似共享自行车，投放自行车，本项目投放图书书柜（小型快递柜），书柜内部有24本图书，每个书柜会有定位，可以在小程序搜到距离你最近的书柜，并且每个书柜会有专属的二维码，因为每个书柜存放的图书不一样，你可以在A书柜扫码借书，之后在B书柜还书，前提是B书柜有空余格子。具体业务流程类似共享自行车，也有设计押金、月卡、季卡等等。

先看看项目的效果吧，暂时给前端小程序，因为从零带着敲，所以原本的后端是SSM的，我将重新改为SpringBoot，后端的管理平台就暂时没有给gif了。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6%E6%95%88%E6%9E%9C%E5%9B%BE.gif)

#### 前后端项目的地址

- [ShareBookServer](https://github.com/UncleCatMySelf/ShareBookServer)
- [ShareBookClient](https://github.com/UncleCatMySelf/ShareBookClient)

## 前端知识盘点

因为我前端的基础不行，所以说得不好的，还请各位码字留情。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E7%9B%AE%E5%BD%95.png)

前端的目录是比较简单的，各位后端的同学也可以简单学习，毕竟到时会给源码，所以大家可以调式试试。imgs是主要小程序的静态资源，即图片什么的，因为小程序自身本来就有限制，所以如果加载大量的图片就直接用url去加载，小业务的话，可以和业务服务器一起，如果数据量大，就自己做一个ftp的文件服务器或者使用阿里的文件存储oss，其他平台的也有很多，这里就不一一介绍了。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6/%E5%B0%8F%E7%A8%8B%E5%BA%8Fmvc.png)

以上是单个页面的实现基本文件目录。整个前端没有使用什么便捷的框架生成，而是原生以MVC的思路去敲，这也是我推荐的，具体理由，...一下省略一万字。

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6/%E5%B0%8F%E7%A8%8B%E5%BA%8F%E6%B5%81%E7%A8%8B.png)

我也是采用后端的MVC模式，xsml是页面骨架，wxss就是H5的css，就是我们的炫酷外表，而内容展示什么，是由js而定，wxml会数据绑定js里面的字段，而js会调用*-model.js里面的方法，*-model.js就是请求我们的后台服务器的具体业务调用端。

虽然大家看到小程序前端都写好了，不过秉承教学目的，所以还是要分步骤，加注释，一步一步的上传GitHub。

## 后端知识盘点

![Image](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/%E5%85%B1%E4%BA%AB%E5%9B%BE%E4%B9%A6/%E5%90%8E%E7%AB%AF%E7%9B%AE%E5%BD%95.png)

后端本身是SSM的框架，不过比较久远，大家可能调试不便，所以就整改为SpringBoot版本，还有数据库设计，这一块我也暂时还没整理出一个结构图，下一篇预计会出，或者下下篇。（本系列因为秉承开源，免费的原则，所以更新时间可能会有波动，个人能力有限，还请见谅。）

后端会使用到freemarker框架来生成后端管理页面，主要是管理图书库存，还有二维码生成子系统是针对书柜设计的，不同书柜会有对应的图书。而系统会以原生netty对接单片机。（因为硬件不属于软件部分，而且单片机一块的基本上有经验的都可以做到，所以到时会用普通的代码模拟）

后端会出两套API，一套是针对小程序的，一套是后台管理系统的。其中还涉及微信支付环节。

具体大家可以关注一下。

#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)