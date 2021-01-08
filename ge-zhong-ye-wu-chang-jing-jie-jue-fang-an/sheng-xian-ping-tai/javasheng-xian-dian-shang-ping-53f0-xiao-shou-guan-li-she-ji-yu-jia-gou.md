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

