# Java生鲜电商平台-物流配送的设计与架构

说明：由于Java开源生鲜电商平台是属于自建物流系统，也就是买家下的单，需要公司派物流团队进行派送。

业务需求中买家的下单时间控制在：12:00-03:00之间。这段时间可以进行下单。

## 1.业务分析：

物流团队需要知道以下东西。

1.配送师傅需要知道去那个菜市场去哪个卖家那里拿到那个买家的货，由于买家买的菜是全品类，但是卖家卖的菜是单品，所以需要进行袋子组装。

2.配送师傅需要确认按照买家分组，知道那些菜是否组装了，那些菜没有进行组装。

3.对于已经送完的买家需要物流APP操作已经送完，方便整个系统的运营与管理。


## 2.业务架构：
1.根据业务的分析，我们得出以下几个数据库表的设计

2.物流人员的管理表，其中由于物流的经理是可以看到所有的区域以及所有的物流的情况。

3.需要根据GPS定位知道目前配送师傅的具体地址。

4.配送师傅知道今天要去那几家取货，而且取多少，需要一目了然。

最新表结构如下：

3.配送人员基础信息表：


```
CREATE TABLE `delivery` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `delivery_phone` varchar(16) DEFAULT NULL COMMENT '手机号码,作为账号登陆',
  `delivery_password` varchar(64) DEFAULT NULL COMMENT '密码',
  `delivery_name` varchar(16) DEFAULT NULL COMMENT '配送人员姓名',
  `delivery_sex` int(11) DEFAULT NULL COMMENT '性别，1为男，2为女',
  `delivery_idcard` varchar(18) DEFAULT NULL COMMENT '身份证号码',
  `delivery_age` int(11) DEFAULT NULL COMMENT '配送人员年龄',
  `delivery_type` int(11) DEFAULT NULL COMMENT '配送人员类型：1为自营，2为加盟',
  `area_id` bigint(20) DEFAULT NULL COMMENT '所属区域',
  `sequence` int(11) DEFAULT NULL COMMENT '排序使用.从小到大排序',
  `status` int(11) DEFAULT NULL COMMENT '1为可用，-1为不可用',
  `remark` varchar(256) DEFAULT NULL COMMENT '备注信息',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `permission_type` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=32 DEFAULT CHARSET=utf8 COMMENT='配送人员基本信息';
```
补充说明：增加了一个permission_type来确定当前用户账号的权限。相关的表结构的注释写的比较清楚。

4.配送师傅配送区域信息表


```
CREATE TABLE `delivery_area` (
  `da_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `da_area_id` bigint(20) DEFAULT NULL COMMENT '区域ID',
  `da_address` varchar(255) DEFAULT NULL COMMENT '地址',
  `da_status` tinyint(4) DEFAULT NULL COMMENT '状态 1在用 -1停用',
  `da_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `da_create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`da_id`)
) ENGINE=InnoDB AUTO_INCREMENT=36 DEFAULT CHARSET=utf8 COMMENT='配送区域';
```
说明：不可能全城所有的物流都是一个师傅配送吧，每个师傅需要看到属于自己配送的买家，仅此而已，我们按照区域维度进行划分。

5.我们需要知道配送师傅目前的GPS地理位置，那么需要建立一个数据进行存储


```
CREATE TABLE `delivery_coordinate` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `lng` varchar(60) DEFAULT NULL COMMENT '经度',
  `lat` varchar(60) DEFAULT NULL COMMENT '维度',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=152063 DEFAULT CHARSET=utf8 COMMENT='配送司机坐标';
```
备注说明：平均每隔30秒，进行系统的主动上报，最终会形成一个配送师傅的坐标。

6.配送人员管理区域信息表


```
CREATE TABLE `delivery_da` (
  `dda_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `dda_delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `dda_da_id` bigint(20) DEFAULT NULL COMMENT '配送区域ID',
  `dda_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `dda_create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`dda_id`)
) ENGINE=InnoDB AUTO_INCREMENT=465 DEFAULT CHARSET=utf8 COMMENT='配送人员管理区域';
```

7.对于有些特殊的情况，比如车在路上坏了，那么需要进行人工分配物流

```
CREATE TABLE `delivery_task` (
  `task_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `order_id` bigint(20) DEFAULT NULL COMMENT '订单ID',
  `order_item_id` bigint(20) DEFAULT NULL COMMENT '订单明细ID',
  `sys_user_id` bigint(20) DEFAULT NULL COMMENT '分配人员ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`task_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

最终形成了完整的业务逻辑闭环


## 相关的业务核心代码如下：

