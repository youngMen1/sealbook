# Java生鲜电商平台-销售管理设计与架构

说明：在Java开源生鲜电商平台中，销售人员我们称为跟餐饮店老板沟通与下载APP的一类地推人员。（所谓地推指的就是一个一个上门拜访。）

由于销售人员有以下几类特性：

1.时间随意性，他们并不类似技术或者性质人员，需要天天呆在办公室，他们是需要去外面，时间上具有随意性。

2.行动随意性 ，他们的行动过于随意，每天也不用来打卡，每天就是按照计划去拜访客户，然后推销生鲜电商APP，让客户来进行下单，那么行为很随意，站在公司的角度

我们是没办法控制这种行为，但是我们也很想知道目前销售人员进度 在哪里来了，遇到了什么问题，一般如何解决。

3.内容随意性，每天早上开会，晚上复盘，很多的时候我们会问今天你们拜访了那些客户，遇到了那些问题，如何解决的，经验进行分享，但是很多时候我们其实不知道内容是真还是假的，因为销售的嘴皮太能说了。

那么如此多的问题，作为技术上，我们应该如何帮助公司呢？对此需要一个管理的销售APP。

## 1.技术上设计，会设计到以下几点；

### 1.销售人员本身的管理

```
CREATE TABLE `sales` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `phone` varchar(32) DEFAULT NULL COMMENT '手机号码',
  `password` varchar(32) DEFAULT NULL COMMENT 'md5加密',
  `true_name` varchar(16) DEFAULT NULL,
  `status` int(11) DEFAULT NULL COMMENT '1为在职，-1为离职',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `last_update_time` datetime DEFAULT NULL COMMENT '更新时间',
  `level` int(11) DEFAULT NULL COMMENT '类型1总监 2主管 3职员',
  `experience` decimal(12,2) DEFAULT NULL,
  `parent_id` bigint(20) DEFAULT NULL COMMENT '直属上级',
  PRIMARY KEY (`id`),
  KEY `unique_phone` (`phone`)
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8 COMMENT='销售人员基本信息';
```
补充说明；任何人员的管理都会存在一个管理人员的权限问题，最高领导者应该具有查看所有的内容的权限。

### 2.销售每天需要写日报，因此销售日报


```
CREATE TABLE `sales_daily` (
  `sd_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `sale_id` bigint(20) DEFAULT NULL COMMENT '销售人员ID',
  `sd_date` date DEFAULT NULL COMMENT '工作日期',
  `task1` decimal(12,2) DEFAULT NULL COMMENT '销售任务',
  `task2` int(11) DEFAULT NULL COMMENT '日拜访量',
  `sd_street` varchar(256) DEFAULT NULL COMMENT '拜访街道',
  `sd_summary` varchar(512) DEFAULT NULL COMMENT '工作总结',
  `sd_time` datetime DEFAULT NULL COMMENT '提交时间',
  `look_status` int(11) DEFAULT NULL COMMENT '查阅状态(0未读 1已读)',
  `look_sale_id` bigint(20) DEFAULT NULL COMMENT '查阅人',
  `look_time` datetime DEFAULT NULL COMMENT '查阅时间',
  `look_reply` varchar(512) DEFAULT NULL COMMENT '主管回复',
  PRIMARY KEY (`sd_id`)
) ENGINE=InnoDB AUTO_INCREMENT=73 DEFAULT CHARSET=utf8 COMMENT='销售日报';
```
说明：主管以及以上的人员都需要进行日报是审批与处理，根据日报反应出来的问题，进行及时的处理。

### 3.每天要做什么，怎么做，你需要有一个计划进行


```
CREATE TABLE `sales_plan` (
  `sp_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `sp_type` tinyint(4) DEFAULT NULL COMMENT '类型(1区域 2销售)',
  `sp_fmon` varchar(10) DEFAULT NULL COMMENT '月份',
  `sale_id` bigint(20) DEFAULT NULL COMMENT '销售人员ID',
  `area_id` bigint(20) DEFAULT NULL COMMENT '区域ID',
  `goal_amt` decimal(12,2) DEFAULT NULL COMMENT '销售总目标',
  `online_amt` decimal(12,2) DEFAULT NULL COMMENT '线上目标',
  `green_amt` decimal(12,2) DEFAULT NULL COMMENT '蔬菜销售',
  `register_num` int(11) DEFAULT NULL COMMENT '注册量',
  `create_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`sp_id`)
) ENGINE=InnoDB AUTO_INCREMENT=4 DEFAULT CHARSET=utf8 COMMENT='销售计划';
```
### 4.销售过程中，肯定会出现一些其他的特殊情况，这种情况需要销售报备


```
CREATE TABLE `sales_reported` (
  `sr_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `sale_id` bigint(20) DEFAULT NULL COMMENT '销售ID',
  `sr_type` tinyint(4) DEFAULT NULL COMMENT '报备类型(1缺填日报 2其他)',
  `sr_desc` varchar(512) DEFAULT NULL COMMENT '报备原因',
  `sr_time` datetime DEFAULT NULL COMMENT '报备时间',
  `look_sale_id` bigint(20) DEFAULT NULL COMMENT '查阅人ID',
  `look_status` tinyint(4) DEFAULT NULL COMMENT '查阅状态',
  `look_time` datetime DEFAULT NULL COMMENT '查阅时间',
  `look_reply` varchar(512) DEFAULT NULL COMMENT '查阅回复',
  PRIMARY KEY (`sr_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='销售报备';
```

### 5.对于一个销售人员而言，你每天需要做的事情很多都是陌生拜访，那么拜访，你总应该有个记录吧，无论是否拜访成功,这里是有一个成功率的问题的。不是每个都成功的


```
CREATE TABLE `sales_visit` (
  `sv_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '商家ID',
  `sale_id` bigint(20) DEFAULT NULL COMMENT '销售人员ID',
  `sv_way` tinyint(4) DEFAULT NULL COMMENT '拜访方式(1预约 2其他)',
  `sv_type` tinyint(4) DEFAULT NULL COMMENT '拜访类型(1上门 2电话)',
  `sv_logo` varchar(128) DEFAULT NULL COMMENT '门头照',
  `sv_address` varchar(256) DEFAULT NULL COMMENT '拜访地址',
  `sv_status` tinyint(4) DEFAULT NULL COMMENT '拜访状态(0未 1已)',
  `sv_date` date DEFAULT NULL COMMENT '预约日期',
  `sv_time` datetime DEFAULT NULL COMMENT '拜访时间',
  `sv_remark` varchar(512) DEFAULT NULL COMMENT '拜访感想',
  PRIMARY KEY (`sv_id`)
) ENGINE=InnoDB AUTO_INCREMENT=508 DEFAULT CHARSET=utf8 COMMENT='销售拜访';
```

#### 6.销售不管有日报还有周报，总结与分析这周的所有问题，成功的也好，失败的也好，需要分享与记录


```
CREATE TABLE `sales_weekly` (
  `sw_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `sale_id` bigint(20) DEFAULT NULL COMMENT '销售ID',
  `sw_fmon` varchar(10) DEFAULT NULL COMMENT '月份',
  `sw_range` varchar(128) DEFAULT NULL COMMENT '时间范围',
  `sw_summary` varchar(1024) DEFAULT NULL COMMENT '本周总结',
  `sw_plan` varchar(1024) DEFAULT NULL COMMENT '下周计划',
  `need_help` varchar(1024) DEFAULT NULL COMMENT '需求帮助',
  `sw_time` datetime DEFAULT NULL COMMENT '填写时间',
  `look_status` tinyint(4) DEFAULT NULL COMMENT '查看状态',
  `look_sale_id` bigint(20) DEFAULT NULL COMMENT '查看人',
  `look_time` datetime DEFAULT NULL COMMENT '查看时间',
  `look_reply` varchar(1024) DEFAULT NULL COMMENT '查看回复',
  PRIMARY KEY (`sw_id`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 COMMENT='销售周报';
```

#### 7.最终根据业务的形态构建了系统架构，同时也设计的数据库，最终业务核心代码如下
 日报的核心代码：
 
