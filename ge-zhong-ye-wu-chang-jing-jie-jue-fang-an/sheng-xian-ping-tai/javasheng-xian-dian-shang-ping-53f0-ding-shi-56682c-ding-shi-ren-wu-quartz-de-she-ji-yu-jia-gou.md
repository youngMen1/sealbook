# Java生鲜电商平台-定时器,定时任务quartz的设计与架构
说明：任何业务有时候需要系统在某个定点的时刻执行某些任务，比如：凌晨2点统计昨天的报表，早上6点抽取用户下单的佣金。
对于Java开源生鲜电商平台而言，有定时推送客户备货，定时计算卖家今日的收益，定时提醒每日的提现金额等等

对于Java定时器而言，我们采用spring+quartz来进行技术解决方案：

对于业务而言，需要满足以下几个方面：

1.对定时任务需要可以手动启动，手动停止，手动删除等。

2.对定时任务的结果需要记录，什么时候执行，执行的结果如何，是否存在异常，是否有异常的记录。

3.对定时器的错误，如果是级别很重要，是否有短信提醒来让运营人员或者技术人员手工处理呢？

类似这样：
641237-20180608085911635-241382390.png

为了满足业务的需求，根据quartz的技术选型，设计一下基础架构：

1.任务记录信息表：

```
CREATE TABLE `t_job` (
  `JOB_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '任务id',
  `BEAN_NAME` varchar(100) NOT NULL COMMENT 'spring bean名称',
  `METHOD_NAME` varchar(100) NOT NULL COMMENT '方法名',
  `PARAMS` varchar(200) DEFAULT NULL COMMENT '参数',
  `CRON_EXPRESSION` varchar(100) NOT NULL COMMENT 'cron表达式',
  `STATUS` char(2) NOT NULL COMMENT '任务状态  0：正常  1：暂停',
  `REMARK` varchar(200) DEFAULT NULL COMMENT '备注',
  `CREATE_TIME` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`JOB_ID`)
) ENGINE=InnoDB AUTO_INCREMENT=12 DEFAULT CHARSET=utf8;
```

2.任务操作日志记录表：

```
CREATE TABLE `t_job_log` (
  `LOG_ID` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '任务日志id',
  `JOB_ID` bigint(20) NOT NULL COMMENT '任务id',
  `BEAN_NAME` varchar(100) NOT NULL COMMENT 'spring bean名称',
  `METHOD_NAME` varchar(100) NOT NULL COMMENT '方法名',
  `PARAMS` varchar(200) DEFAULT NULL COMMENT '参数',
  `STATUS` char(2) NOT NULL COMMENT '任务状态    0：成功    1：失败',
  `ERROR` text COMMENT '失败信息',
  `TIMES` decimal(11,0) DEFAULT NULL COMMENT '耗时(单位：毫秒)',
  `CREATE_TIME` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`LOG_ID`)
) ENGINE=InnoDB AUTO_INCREMENT=2476 DEFAULT CHARSET=utf8;
```

说明：整个业务很简单，整个表设计与架构也很简单。

最终运营截图如下：
641237-20180608085846548-2117223474.png