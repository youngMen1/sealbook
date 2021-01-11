# Java生鲜电商平台-定时器,定时任务quartz的设计与架构
说明：任何业务有时候需要系统在某个定点的时刻执行某些任务，比如：凌晨2点统计昨天的报表，早上6点抽取用户下单的佣金。
对于Java开源生鲜电商平台而言，有定时推送客户备货，定时计算卖家今日的收益，定时提醒每日的提现金额等等

对于Java定时器而言，我们采用spring+quartz来进行技术解决方案：

对于业务而言，需要满足以下几个方面：

1.对定时任务需要可以手动启动，手动停止，手动删除等。

2.对定时任务的结果需要记录，什么时候执行，执行的结果如何，是否存在异常，是否有异常的记录。

3.对定时器的错误，如果是级别很重要，是否有短信提醒来让运营人员或者技术人员手工处理呢？

类似这样：
![](/static/image/641237-20180608085911635-241382390.png)

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
![](/static/image/641237-20180608085846548-2117223474.png)
![](/static/image/641237-20180608085938081-1006465536.png)

相关的系统设置与配置说明：

相关的核心代码如下：（贴些核心的，不算很核心的，大家可以去我github下面下载即可）



```
/**
 * 定时任务
 * 
 * @author Administrator
 *
 */
public class ScheduleJob extends QuartzJobBean {
    private Logger logger = LoggerFactory.getLogger(this.getClass());
    private ExecutorService service = Executors.newSingleThreadExecutor();

    @Override
    protected void executeInternal(JobExecutionContext context) throws JobExecutionException {
        Job scheduleJob = (Job) context.getMergedJobDataMap().get(Job.JOB_PARAM_KEY);

        // 获取spring bean
        JobLogService scheduleJobLogService = (JobLogService) SpringContextUtils.getBean("JobLogService");

        JobLog log = new JobLog();
        log.setJobId(scheduleJob.getJobId());
        log.setBeanName(scheduleJob.getBeanName());
        log.setMethodName(scheduleJob.getMethodName());
        log.setParams(scheduleJob.getParams());
        log.setCreateTime(new Date());

        long startTime = System.currentTimeMillis();

        try {
            // 执行任务
            logger.info("任务准备执行，任务ID：" + scheduleJob.getJobId());
            ScheduleRunnable task = new ScheduleRunnable(scheduleJob.getBeanName(), scheduleJob.getMethodName(),
                    scheduleJob.getParams());
            Future<?> future = service.submit(task);
            future.get();
            long times = System.currentTimeMillis() - startTime;
            log.setTimes(times);
            // 任务状态 0：成功 1：失败
            log.setStatus("0");

            logger.info("任务执行完毕，任务ID：" + scheduleJob.getJobId() + "  总共耗时：" + times + "毫秒");
        } catch (Exception e) {
            logger.error("任务执行失败，任务ID：" + scheduleJob.getJobId(), e);
            long times = System.currentTimeMillis() - startTime;
            log.setTimes(times);
            // 任务状态 0：成功 1：失败
            log.setStatus("1");
            log.setError(StringUtils.substring(e.toString(), 0, 2000));
        } finally {
            scheduleJobLogService.saveJobLog(log);
        }
    }
}
```


```
/**
 * 执行定时任务
 * 
 * @author Administrator
 *
 */
public class ScheduleRunnable implements Runnable {
    private Object target;
    private Method method;
    private String params;

    public ScheduleRunnable(String beanName, String methodName, String params)
            throws NoSuchMethodException, SecurityException {
        this.target = SpringContextUtils.getBean(beanName);
        this.params = params;

        if (StringUtils.isNotBlank(params)) {
            this.method = target.getClass().getDeclaredMethod(methodName, String.class);
        } else {
            this.method = target.getClass().getDeclaredMethod(methodName);
        }
    }

    @Override
    public void run() {
        try {
            ReflectionUtils.makeAccessible(method);
            if (StringUtils.isNotBlank(params)) {
                method.invoke(target, params);
            } else {
                method.invoke(target);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

}
```


```
/**
 * 执行定时任务
 * 
 * @author Administrator
 *
 */
public class ScheduleRunnable implements Runnable {
    private Object target;
    private Method method;
    private String params;

    public ScheduleRunnable(String beanName, String methodName, String params)
            throws NoSuchMethodException, SecurityException {
        this.target = SpringContextUtils.getBean(beanName);
        this.params = params;

        if (StringUtils.isNotBlank(params)) {
            this.method = target.getClass().getDeclaredMethod(methodName, String.class);
        } else {
            this.method = target.getClass().getDeclaredMethod(methodName);
        }
    }

    @Override
    public void run() {
        try {
            ReflectionUtils.makeAccessible(method);
            if (StringUtils.isNotBlank(params)) {
                method.invoke(target, params);
            } else {
                method.invoke(target);
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```


POM文件


```
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz</artifactId>
    <version>2.2.1</version>
</dependency>
<dependency>
    <groupId>org.quartz-scheduler</groupId>
    <artifactId>quartz-jobs</artifactId>
    <version>2.2.1</version>
</dependency>
```
org.quartz.scheduler.instanceName属性可为任何值，用在 JDBC JobStore 中来唯一标识实例，但是所有集群节点中必须相同。

org.quartz.scheduler.instanceId　属性为 AUTO即可，基于主机名和时间戳来产生实例 ID。

org.quartz.jobStore.class属性为 JobStoreTX，将任务持久化到数据中。因为集群中节点依赖于数据库来传播 Scheduler 实例的状态，你只能在使用 JDBC JobStore 时应用 Quartz 集群。这意味着你必须使用 JobStoreTX 或是 JobStoreCMT 作为 Job 存储；你不能在集群中使用 RAMJobStore。

org.quartz.jobStore.isClustered 属性为 true，你就告诉了 Scheduler 实例要它参与到一个集群当中。这一属性会贯穿于调度框架的始终，用于修改集群环境中操作的默认行为。

org.quartz.jobStore.clusterCheckinInterval 属性定义了Scheduler 实例检入到数据库中的频率(单位：毫秒)。Scheduler 检查是否其他的实例到了它们应当检入的时候未检入；这能指出一个失败的 Scheduler 实例，且当前 Scheduler 会以此来接管任何执行失败并可恢复的 Job。通过检入操作，Scheduler 也会更新自身的状态记录。clusterChedkinInterval 越小，Scheduler 节点检查失败的 Scheduler 实例就越频繁。默认值是 15000 (即15 秒)。

 

最终：系统架构设计图：
![](/static/image/641237-20180608090643791-1170168567.png)
最后：很多人说系统功能很强大很好，但是我的一种思维方式是不一定，强大固然好，但是你需要通过这么多的系统数据中来分析出问题的关键，而不是所谓的代码堆积。

你所需要的是思考，再思考，最终思考。
