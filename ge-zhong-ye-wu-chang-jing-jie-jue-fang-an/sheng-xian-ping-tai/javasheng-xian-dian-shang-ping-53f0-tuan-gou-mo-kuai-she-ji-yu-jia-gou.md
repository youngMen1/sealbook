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