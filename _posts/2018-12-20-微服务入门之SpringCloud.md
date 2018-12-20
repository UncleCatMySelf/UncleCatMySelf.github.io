---
layout:       post
title:        微服务入门之SpringCloud（视频文案）
subtitle:     微服务系列
date:         2018-12-21
author:       MySelf | 猫叔
header-img:   img/elage.jpg
catelog:      true
tags:
    - 微服务
    - SpringCloud
---

## GitHub项目地址

[微服务入门讲解](https://github.com/UncleCatMySelf/mic-demo)

## 一、从传统单体架构走向微服务

那些年的加班夜！

* 1、庞大的代码块、关系错综复杂
* 2、交付周期长、上手时间长
* 3、扩张能力、弹性受限
* 4、…….


### 微服务架构风格是一种将单个应用程序作为一套小型服务开发的方法，每种应用程序都在自己的进程中运行，并与轻量级机制（通常是HTTP资源API）进行通信。 这些服务是围绕业务功能构建的，可以通过全自动部署机制独立部署。 这些服务的集中管理最少，可以用不同的编程语言编写，并使用不同的数据存储技术。

#### 有效的拆分应用，实现敏捷开发和部署

将整体服务拆分为：

* 用户服务
* 订单服务
* 数据投放
* 日志系统
* 搜索引擎
* 社交系统
* .....

#### 只做一件事，并把它做好

对于我们这种要求简单的，工作的时候一般都只想做一件事就好了，不要让我顾及太多。

#### 单一（隔离）、独立指责

我们可以尽情的在自己负责的项目上“玩耍”啦！对于其他服务层的对接仅需要按照各个应用通信接口文档去走即可！

#### 通信（同异步）、数据独立

我们总是要和人交流的，对于我们自己独自负责的服务也是需要去交朋友的，因此它需要与其他各个服务进行通信，这里的通信可能是同步的、异步的。

对于每个引用都有他们自己的数据，微服务的采纳有助于我们可以针对部分火爆业务采用不同的数据库类型或者分库读取，而不再需要在同一项目整合多个数据库操作。

#### 技术栈灵活（数据流、存储、业务）

我们可以发挥不同语言的优势，比如python、nodejs、php….

这对技术专项不同的开发团队来说，配合起来将更加容易与得心应手。


## 二、单体架构电商Demo讲解（思路）

一个普通的电商项目：

* 1、用户服务：注册、登录
* 2、商品服务：查询库存
* 3、订单服务：用户下单、查看订单

数据库表：

* 1、用户表：用户id、用户名、用户密码、创建时间
* 2、商品表：商品id、商品名、商品详情、商品价格
* 3、库存表：库存id、商品id、库存数
* 4、订单表：订单id、用户名、商品id、订单总价、商品总数、创建时间

```SQL
SET FOREIGN_KEY_CHECKS=0;

-- ----------------------------
-- Table structure for goods
-- ----------------------------
DROP TABLE IF EXISTS `goods`;
CREATE TABLE `goods` (
  `goods_id` varchar(50) NOT NULL COMMENT '商品ID',
  `goods_name` varchar(255) DEFAULT NULL COMMENT '商品名',
  `goods_detail` varchar(255) DEFAULT NULL COMMENT '商品详情',
  `goods_price` decimal(10,2) DEFAULT NULL COMMENT '商品价格',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`goods_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='商品表';

-- ----------------------------
-- Records of goods
-- ----------------------------
INSERT INTO `goods` VALUES ('MS000001', '安卓机', '高性能、火热性感', '799.00', '2018-12-12 16:43:59');
INSERT INTO `goods` VALUES ('MS000002', '苹果机', '酷炫来袭、值得拥有', '2999.00', '2018-12-12 16:54:58');

-- ----------------------------
-- Table structure for inventory
-- ----------------------------
DROP TABLE IF EXISTS `inventory`;
CREATE TABLE `inventory` (
  `inventory_id` varchar(50) NOT NULL COMMENT '订单id',
  `goods_id` varchar(50) NOT NULL COMMENT '商品id',
  `inventory_num` int(11) NOT NULL COMMENT '库存数',
  PRIMARY KEY (`inventory_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COMMENT='库存表';

-- ----------------------------
-- Records of inventory
-- ----------------------------
INSERT INTO `inventory` VALUES ('IN4567825', 'MS000002', '13');
INSERT INTO `inventory` VALUES ('IN4567826', 'MS000001', '10');

-- ----------------------------
-- Table structure for orders
-- ----------------------------
DROP TABLE IF EXISTS `orders`;
CREATE TABLE `orders` (
  `order_id` varchar(50) NOT NULL COMMENT '订单id',
  `goods_id` varchar(50) DEFAULT NULL COMMENT '商品id',
  `name` varchar(255) DEFAULT NULL COMMENT '用户名',
  `order_price` decimal(10,2) DEFAULT NULL COMMENT '订单价格',
  `order_num` int(11) DEFAULT NULL COMMENT '商品总数',
  `create_time` timestamp NOT NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`order_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;

-- ----------------------------
-- Records of orders
-- ----------------------------
INSERT INTO `orders` VALUES ('MS04663799', 'MS000002', '猫叔', '2999.00', '1', '2018-12-13 00:04:18');

-- ----------------------------
-- Table structure for user
-- ----------------------------
DROP TABLE IF EXISTS `user`;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '用户id',
  `name` varchar(255) DEFAULT NULL COMMENT '用户名',
  `password` varchar(255) DEFAULT NULL COMMENT '用户密码',
  `create_time` timestamp NULL DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8mb4 COMMENT='用户表';

-- ----------------------------
-- Records of user
-- ----------------------------
INSERT INTO `user` VALUES ('1', '猫叔', '123456', '2018-12-12 16:44:23');
```

#### 接口列表

> 1、商品下单接口

* 地址: `localhost:8080/sells/order/order` **POST**
* 参数:

    id --> 用户id
    goodsId --> 商品id
    num --> 商品数量

* 例子:
```js
{
    "code": 200,
    "msg": "成功",
    "data": "MS04638097"
}
```

> 2、查询用户订单列表

* 地址: `localhost:8080/sells/order/order?id=1` **GET**
* 参数:

    id --> 用户id

* 例子:
```js
{
    "code": 200,
    "msg": "成功",
    "data": [
        {
            "orderId": "MS04131640",
            "goodsId": "MS000002",
            "name": "猫叔",
            "orderPrice": 2999,
            "orderNum": 1,
            "createTime": "2018-12-16T10:33:14.000+0000"
        },
        {
            "orderId": "MS04254690",
            "goodsId": "MS000002",
            "name": "猫叔",
            "orderPrice": 2999,
            "orderNum": 1,
            "createTime": "2018-12-16T13:57:10.000+0000"
        }
    ]
}
```

> 3、查询商品库存信息

* 地址： `localhost:8080/sells/goods/goods?id=1&goodsId=MS000002` **GET**
* 参数：

    id --> 管理员id
    goodsId --> 商品id

* 例子：
```js
{
    "code": 200,
    "msg": "成功",
    "data": {
        "inventoryId": "IN4567825",
        "goodsId": "MS000002",
        "inventoryNum": 9
    }
}
```

## 三、SpringCloud框架入门

说到SpringCloud，我们还需要说一下它基于的更厉害的框架，它就是Netflix。Netflix的多个开源组件一起正好可以提供完整的分布式微服务基础架构环境，而SpringCloud就是对Netflix的多个开源组件进一步封装而成的。


同时，它利用SpringBoot的开发便利性巧妙地简化了分布式系统基础设施的开发，比如我们今天将学到的服务发现注册（Eureka）、调用框架（声明式服务客户端Fegin）等等。

Spring Cloud将目前各个比较成熟、经得起实际考验的服务框架组合起来，通过Spring Boot风格进行再封装屏蔽掉了复杂的配置和实现原理，因为我们仅仅配置一下、写几句代码就可以实现一个想要的简单的微服务。


## 四、Eureka实操与微服务架构搭建（实战）

* 1、Eureka是一个基于REST的服务
* 2、基于Netflix Eureka做了二次封装
* 3、核心组件为：

    - Eureka Server  注册中心（服务端）
    - Eureka Client   服务注册（客户端）


## 五、服务拆分与应用通讯（实战）

Spring Cloud 服务间的一种RestFul调用方式

* 1、Fegin

    - 声明式REST客户端
    - 接口 + 注解（开启、指定服务名）
    - 本质依旧是一个HTTP客户端


## 六、关于微服务的不解与深入探讨

微服务还有更多知识点···

* 1、负载均衡
* 2、安全机制
* 3、配置中心
* 4、链路追踪
* 5、容器搭配

### 微服务架构的取舍？

* 1、避免为了“微服务”而使用“微服务”

* 2、对于传统企业而言，开始时可以考虑引入部分合适的微服务架构原则对已有系统进行改造或新建微服务应用，逐步探索及累积微服务架构经验，而非照搬采纳，一口气吃出大胖子。

### 微服务的不足又是哪些？

* 1、微服务的复杂度
* 2、分布式事务
* 3、服务的划分
* 4、服务的部署

### 各个服务组件的版本与部署乃至升级

* 1、开发管理规范：设计规范、分支管理、持续集成
* 2、Docker容器：版本控制、可移植性、隔离性
* 3、容器云平台：持续集成、快速部署、灰度开发及无缝升级




