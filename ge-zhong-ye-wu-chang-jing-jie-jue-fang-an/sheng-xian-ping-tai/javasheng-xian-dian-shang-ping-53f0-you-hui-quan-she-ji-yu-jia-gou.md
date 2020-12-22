# Java生鲜电商平台-优惠券设计与架构

说明：现在电商白热化的程度，无论是生鲜电商还是其他的电商等等，都会有促销的这个体系，目的就是增加订单量与知名度等等

那么对于Java开源生鲜电商平台而言，我们采用优惠券的这种方式进行促销。（补贴价格战对烧钱而言非常的恐怖的，太烧钱了）

1.优惠券基础信息表

说明：任何一个优惠券或者说代金券都是有一个基础的说明，比如：优惠券名称，类型，价格，有效期，状态,说明等等基础信息。


```
CREATE TABLE `coupon` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `region_id` bigint(20) DEFAULT NULL COMMENT '所属区域',
  `type` int(11) DEFAULT NULL COMMENT '所属类型,1为满减',
  `name` varchar(32) DEFAULT NULL COMMENT '优惠券名称',
  `img` varchar(64) DEFAULT NULL COMMENT '图片的URL地址',
  `start_time` datetime DEFAULT NULL COMMENT '优惠券开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '优惠券结束时间',
  `money` decimal(11,2) DEFAULT NULL COMMENT '优惠券金额，用整数，固定值目前。',
  `status` int(11) DEFAULT NULL COMMENT '状态，0表示未开始，1表示进行中，-1表示结束',
  `remarks` varchar(512) DEFAULT NULL COMMENT '优惠券的说明',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `full_money` decimal(12,2) DEFAULT NULL COMMENT '金额满',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='优惠券基础配置表';
```


