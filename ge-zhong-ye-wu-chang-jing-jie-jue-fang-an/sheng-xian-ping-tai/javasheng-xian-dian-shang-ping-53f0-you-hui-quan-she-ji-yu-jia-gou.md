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
说明：业务说可以规定某个区域，做优惠券，而且是纳新后才有，这样增加买家用户。价格可以后端进行设置。

状态的意义在于，用户需要注册完成后，然后主动认领才有效，为什么要这样设计呢？目的只有一个：让用户在对APP这个软件玩一会儿，增加熟悉程度.

2.优惠券领取记录表
说明：我们需要记录那个买家，什么时候进行的领取，领取的的时间，券的额度是多少等等，是否已经使用等信息

```
CREATE TABLE `coupon_receive` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `coupon_id` bigint(20) DEFAULT NULL COMMENT '优惠券编号',
  `coupon_money` decimal(12,2) DEFAULT NULL COMMENT '券额',
  `create_time` datetime DEFAULT NULL COMMENT '领取时间',
  `full_money` decimal(12,2) DEFAULT NULL COMMENT '金额满',
  `status` int(11) DEFAULT NULL COMMENT '状态，1为已使用，0为已领取未使用，-1为已过期',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='优惠券领取记录表';
```

3.优惠券消费记录表

说明：**优惠券消费记录表，是需要知道那个买家，那个优惠券，那个订单使用了优惠券，这边有个特别注意的地方是，这个优惠券的执行在支付成功后的回调**。

```
CREATE TABLE `coupon_logs` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `coupon_receive_id` bigint(20) DEFAULT NULL COMMENT '优惠券id',
  `order_number` varchar(64) DEFAULT NULL COMMENT '订单号',
  `order_original_amount` decimal(12,2) DEFAULT NULL COMMENT '原订单金额',
  `coupon_amount` decimal(11,2) DEFAULT NULL COMMENT '优惠券的金额',
  `order_final_amount` decimal(12,2) DEFAULT NULL COMMENT '抵扣优惠券之后的订单金额',
  `create_time` datetime DEFAULT NULL COMMENT '领取时间',
  `status` int(2) DEFAULT '0' COMMENT '日志状态: 默认为0，支付回调成功后为1',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='优惠券消费记录表';
```





