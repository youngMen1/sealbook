# Java生鲜电商平台-售后模块的设计与架构

**说明：任何一个的电商平台都有售后服务系统，那么对于我们这个生鲜的电商平台，售后系统需要思考以下几个维度。**

1.买家的需求维度

说明：买家在平台上没找到自己想要的东西，我们需要提供给他一个入口，告诉我们他有这个需求，我们进行改进。系统需要有记录这种情况，同时也有回复客户的情况。

2.投诉入口

说明：有客户性子比较急，他有问题，就会马上打电话给客服，客服需要解答与回答，维护客户关系。对于系统而言，需要记录这种情况，然后分析问题与解决问题。

3.IM聊天入口

说明：客户有时候也不想写信息，也不想打电话，能否有一个时刻的IM聊天记录呢？对于系统而言需要记录这种信息，我们目前系统没处理，采用的是微信，以及销售人员的反馈机制。

4.退货问题

说明：售后系统中，退货问题是最繁琐的，买家存在以下两种情况。

4.1 买家要钱不要货。顾名思义，有些买家就是不要货了，他需要我们退钱给他，这个配送端有一个一件退货功能，钱退到买家的余额里面，下次可以继续购买。

4.2  买家要货不要钱，顾名思义，有些买家的确需要这个货物，对于我们退钱给他，他是不接受的，因为他真的需要这种东西，你让他再去买，客户体验非常差，可能 就没有下次购物了。对于这种情况，我们用时间轴来继续整个过程。（说明，由于这个系统设计到生鲜电商方面，其他的电商方面可能会不一样。）

相关数据库的设计与架构如下：

1.买家平台建议信息表

```
CREATE TABLE `suggestion` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `suggestion_content` varchar(1024) DEFAULT NULL COMMENT '建议内容',
  `suggestion_imgs` varchar(255) DEFAULT NULL COMMENT '多张图片',
  `user_id` bigint(20) DEFAULT NULL COMMENT '所属用户ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=78 DEFAULT CHARSET=utf8 COMMENT='用户对平台的建议';
```
说明: 平台建议表，是买家对平台的建议以及自己的需求的一个入口，可以是图片与内容两点。

比如说：他说我们送的菜有问题，很多烂的，那么他是需要拍图片证明的。


2.平台回复信息表


```
CREATE TABLE `suggestion_reply` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `suggestion_id` bigint(20) DEFAULT NULL COMMENT '客户的建议⁯ID',
  `content` varchar(512) DEFAULT NULL COMMENT '回复的内容',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=23 DEFAULT CHARSET=utf8 COMMENT='客户建议回复信息表';
```
说明：作为一个平台，平台需要回复客户的信息，买家也需要看到，当然这边系统是不区分是买家还是卖家的，我们都是可以数据的处理的。

3.售后系统时间轴的设计

说明：其实我们系统需要知道整个售后的过程的，比如买家什么时候发起的不要钱，要货，然后师傅是什么时候知道这个消息的，如何进行售后的，他们会遇到什么问题，

当然这里面有很多的问题，系统可以做的事情其实是很少的，而我们需要做的是更多的事情。

```
CREATE TABLE `order_timeline` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `item_id` bigint(20) DEFAULT NULL COMMENT '订单项ID',
  `remarks` varchar(256) DEFAULT NULL COMMENT '备注',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=1980 DEFAULT CHARSET=utf8 COMMENT='售后模块，退换货时间轴，针对的是某一个订单项';
```
相关时间轴运营截图如下：
641237-20180521111500103-1654435830.png
641237-20180521111525161-1739718793.png
整个业务不算复杂，需要的一种思路与解决思路的方案：
关于补货流程：


```
补货需求
业务需求：
    当卖家主动点击缺货，则配送师傅看到这个异常订单项，然后他有两种选择，
第一种补货（有货，他也想补或者客户说要货不要钱）
第二种不补货（无货可补，他不想补或者客户说退钱等等）
第一种补货业务：
1.    当配送师傅点击已补货，则把这个订单项对应的金额从买家中直接扣除，前提是线上付款，如果这个订单是线下付款，则不用处理扣款逻辑，直接修改状态即可。同时记录时间轴日志。
第二种不补货业务：
2.    当师傅点击不补货，则这个订单项不做任何扣款逻辑，不管线下还是线上，直接修改状态即可，同时记录时间轴日志。
补充说明：补货与不补货属于互斥操作，即已补货后不允许再出现不补货，不补货后不再允许出现补货。按照规则来处理。
```


