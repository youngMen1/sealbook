# Java生鲜电商平台-提现模块的设计与架构

补充说明：生鲜电商平台-提现模块的设计与架构，提现功能指的卖家把在平台挣的钱提现到自己的支付宝或者银行卡的一个过程。

功能相对而言不算复杂，有以下几个功能需要处理。

业务逻辑如下：
1.卖家登陆自己的B2B系统提交提现功能。

2.如果没有绑定银行卡或者支付宝，则需要先绑定银行卡或者支付宝账户，以及填写提现密码

3.支付宝或者银行卡需要跟用户的姓名所填一致，防止错误转账。

4.后端需要记录所提现的记录，实际情况是支付宝提现需要收取手续费，这个也需要记录在内。

5.需要形成一个审核机制，用户提现的状态有申请提现，审核成功，提现成功，提现失败四种可能状态。

6.每个提现的过程需要记录时间轴，如果有拒绝，用户需要查看拒绝的原因。

7.所有的提现到账后，需要平台短信通知用户申请了提现，提现成功，包括提现拒绝等等，都需要短信通知，给用户一个信任感。

8.每天晚上5:30之前提现当日到达，之后的次日早上10点钟到达。

9.系统自动审核提现的金额数据量的正确与否，来源于用户的订单以及账单数据。

相关的系统设计表如下:
1.提现信息表，为了便于大家理解，我详细的注释都写上了。


```
CREATE TABLE `withdrawal` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `uid` bigint(20) NOT NULL COMMENT '提现申请人',
  `withdraw_order` varchar(64) NOT NULL COMMENT '提现订单号,系统自动生成的.',
  `withdraw_bank_id` bigint(20) NOT NULL COMMENT '用户对应的卡的编号',
  `withdraw_charge` decimal(12,2) NOT NULL COMMENT '提现手续费',
  `withdraw_reality_total` decimal(12,2) NOT NULL COMMENT '实际提现金额',
  `withdraw_apply_total` decimal(12,2) NOT NULL COMMENT '申请提现的金额',
  `withdraw_apply_time` datetime NOT NULL COMMENT '申请提现时间',
  `status` int(11) NOT NULL COMMENT '提现状态,1表示申请提现,2表示审批通过,3,交易完成,-1审批不通过.',
  `create_by` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` bigint(20) DEFAULT NULL COMMENT '修改人',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后修改时间',
  PRIMARY KEY (`id`),
  KEY `unique_order` (`withdraw_order`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COMMENT='提现信息表';
```



    