# Java生鲜电商平台-团购模块设计与架构
 
说明：任何一个电商系统中，对于促销这块是必不可少的，毕竟这块是最吸引用户的，用户也是最爱的模块之一，理由很简单，便宜。

我的经验是无论是大的餐饮点还是小的餐饮店，优惠与折扣永远是说服他们进入平台的最好的手段之一。（大企业叫做节约成本，小企业叫做贪便宜.）

**1.Java开源生鲜电商平台中，团购模块，我们采用以下几种维度思考。**

**1.1.针对的是生鲜中的标品。（米面粮油，我们要求买家可以自己发送团购，但是团购有次数，与时间以及买家起团金额和最低开团金额几个维度）**

**因此，需要有一个团购基础信息表：**

```
CREATE TABLE `groups` (
  `group_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `group_no` varchar(32) DEFAULT NULL COMMENT '团号',
  `group_title` varchar(128) DEFAULT NULL COMMENT '团购标题',
  `group_logo` varchar(128) DEFAULT NULL COMMENT '团购logo',
  `group_area` varchar(128) DEFAULT NULL COMMENT '团购区域(区域ID集合)',
  `begin_time` datetime DEFAULT NULL COMMENT '开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '结束时间',
  `max_num` int(11) DEFAULT NULL COMMENT '最大买家数',
  `buyer_amt` decimal(12,2) DEFAULT NULL COMMENT '买家起团金额',
  `min_amt` decimal(12,2) DEFAULT NULL COMMENT '最低开团金额',
  `group_status` tinyint(4) DEFAULT NULL COMMENT '状态(1发布 -1未发布 2团成 3未团成)',
  `remarks` varchar(256) DEFAULT NULL,
  `create_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`group_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COMMENT='团购主表';
```

说明：这里面有一个团购的状态需要指明下，买家用户选择好几样商品后发起了团购，然后默认状态为-1，表示不可用，等组成了团购，最终状态会有团成的状态。


2.对于团购而言，系统肯定需要记录，那些买家参与了那些团购，因此有以下的一张表记录

```
CREATE TABLE `groups_buyer` (
  `gb_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `group_id` bigint(20) DEFAULT NULL COMMENT '团购ID',
  `item_id` bigint(20) DEFAULT NULL COMMENT '团购明细ID',
  `order_id` bigint(20) DEFAULT NULL COMMENT '订单ID',
  `gb_num` int(11) DEFAULT NULL COMMENT '团购数量',
  `gb_price` decimal(12,2) DEFAULT NULL COMMENT '团购价格',
  `gb_amt` decimal(12,2) DEFAULT NULL COMMENT '团购金额',
  `gb_status` tinyint(4) DEFAULT NULL COMMENT '状态(1完成 -1取消)',
  `gb_time` datetime DEFAULT NULL COMMENT '团购时间',
  PRIMARY KEY (`gb_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='团购买家表';
```
说明：团购买家表，记录那个买家，那个团购，团购的最终数量以及团购的价格等等，最终是否有买家在规定的时间内推出了团购，或者团购未形成等等。


3.团购最终是对商品的规格进行团购。

谈谈商品的规格系数，我们知道蔬菜中有西红柿对吧，那么西红柿分为两种，一种是大红的，一种是粉红，这两种颜色都是西红柿，那么系统会认为这个是两个产品，而不是两个规格，规格到底是说的什么呢?

对于平台而言，规格就是一种商品的几种售卖方式。

最终根据业务分析，我们需要记录团购是由那些明细组成.(商品规格组成)

因此，最终系统架构如下：

