---
layout:       post
title:        同样是程序员，他转行在新加坡卖鱼走向巅峰！
subtitle:     Github
date:         2019-05-06
author:       MySelf 猫叔
header-img:   img/ship.jpg
catelog:      true
tags:
    - Java
    - 编程资讯
---

> 本博客 [猫叔的博客](https://unclecatmyself.github.io/)，转载请申明出

> 阅读本文约“3分钟”
> 适读人群：IT/互联网工作者、游戏爱好者

吃鸡吗？

![1](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1gif.gif)

> 本文部分素材摘抄自“36Kr-《最前线 | 腾讯“吃鸡”游戏或借壳变现，《绝地求生》“成为”《和平精英》》”。

玩过吃鸡类手游的朋友，应该都大致了解过各类大厂出的吃鸡游戏吧。今日，腾讯旗下已过审游戏《和平精英》测试服务器安卓端已开启，用腾讯社交账号体系登陆后，继承了《绝地求生：刺激战场》的游戏数据。此外，除了游戏的世界观不太一样，《和平精英》的美术、UI都与《绝地求生：刺激战场》极为相似。

![1](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A11.jpg)
**商业变现？**
![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E5%95%86%E4%B8%9A.png)
**高度保密？**
![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E4%BF%9D%E5%AF%86.png)
**国家军队机构宣传广告！**
![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E5%B0%81%E9%9D%A2.png)
**经济价值**
![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E7%BB%8F%E6%B5%8E%E4%BB%B7%E5%80%BC.png)
我是一个LOL的老玩家了（一脚盲僧），不过吃鸡真的玩不过来，毕竟3D眩晕的感觉一次就足够我体会了。

在TalkingData免费版搜了一下，查询手机游戏的射击类排行，3月份的前10是这样的，前3还是依旧是大厂。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E6%95%B0%E6%8D%AETD.png)

我比较少玩吃鸡（几乎没有），不过对于`游戏的好奇感`一直高于其他行业。

今天来和大家说点游戏技术上的事情吧。

就说简单的吧，毕竟我本身也不是做专业游戏出身的。说一举例说，就说一下`射击子弹与玩家角色之间的关系与基本实现思路`。

吃鸡是3D的，我举例就用`微信小游戏`的模版来说吧，是2D的，不过大同小异。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1demo.png)

这个是微信小游戏，官方的Demo，使用快速模板就可以生成运行，是一个很传统的`飞机射击游戏`，其实原理也很简单，精灵（所有运动角色的简称）是通过x/y轴改变位置来实现移动，过程中精灵的动作可以通过`一系列的帧图片`实现走动效果或者爆炸。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E5%9B%BE%E7%89%87%E7%B4%A0%E6%9D%90.gif)

让我们玩玩这个游戏看看。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E6%B8%B8%E6%88%8F1.gif)

我们来说说这个子弹击中敌军飞机，然后爆炸，分数+1的过程。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E5%90%83%E9%B8%A1%E5%88%86%E6%9E%90.png)

简单的说，就是判断子弹的坐标与敌军飞机是否重叠或者“接触”，如果是，那么就执行`击中音乐、飞机消失、子弹爆炸动画、分数+1`等子函数的动作。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E6%B8%B8%E6%88%8F%E4%BB%A3%E7%A0%81.png)

如果我们想要让其在判断后，不执行哪一部分的操作，那么也可以将该部分的动作代码注释掉。

![](https://raw.githubusercontent.com/UncleCatMySelf/img-myself/master/img/write/%E6%B8%B8%E6%88%8F2.gif)

如上的游戏效果，就是我把飞机消失的动画效果去除后的游戏体验，因为敌军没有消失，所以当它接触到我方的时候，游戏就结束了。

太久没有玩小程序，一些ES6的语法也记不太清楚了~


#### 公众号：Java猫说

**学习交流群：728698035**

> 现架构设计（码农）兼创业技术顾问，不羁平庸，热爱开源，杂谈程序人生与不定期干货。

![Image Text](https://user-gold-cdn.xitu.io/2018/12/28/167f41f1a5729856?w=344&h=344&f=jpeg&s=8231)