# Java生鲜电商平台-系统报表设计与架构

说明：任何一个运行的平台都需要一个很清楚的报表来显示，那么作为Java开源生鲜电商平台而言，我们应该如何设计报表呢？或者说我们希望报表来看到什么数据呢？

通过报表我们可以分析出目前整个公司的运营情况，以及下一步的调整方向，这样更加有理有据的掌握整个市场与决策。

设计基础维度：

1.今日订单，今日营业额，总订单数，总营业额

2.今日的注册买家，总的注册买家。

3.实时的营收，实时的下单买家。

4.今日下单买家，空降A(空降A指的是今天注册同时下单的客户)

数据的力量在于清楚的明白的告诉整个系统运营人员，昨天我们的销售团队创造了多少的业绩，以及整个趋势是怎么样的，今天的努力方向是怎么样的，昨天的所获是怎么样的

如何进行一起努力与学习。

**统计表的系统设计如下：**

```
CREATE TABLE `report_days` (
  `id` bigint(20) DEFAULT NULL,
  `order_number_count` int(11) DEFAULT NULL COMMENT '今日订单数',
  `order_rmb_count` decimal(12,2) DEFAULT NULL COMMENT '今日营业额',
  `order_number_amount` int(11) DEFAULT NULL COMMENT '总订单数',
  `order_rmb_amount` decimal(12,2) DEFAULT NULL COMMENT '总营业额',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间'
) ENGINE=InnoDB DEFAULT CHARSET=utf8 COMMENT='每日报表';
```

说明：其实就是向这里面进行数据的更新与增加操作即可，每天进行报表的读取与显示不过有些网友提出采用缓存来处理，我个人的意见是不需要。数据也没那么多，而且都是定时器来执行，缓存的价值与意义很小。

相关的执行代码如下：


```
@Component
public class TaskReport {

    private static final Logger logger=LoggerFactory.getLogger(TaskReport.class);
    
    @Autowired
    private BuyerOrderReportService buyerOrderReportService;
    
    @Autowired
    private ReportDayService reportDayService;
    
    @Autowired
    private BillService billService;
    /**
     * 计算每天报表;
     *    每日上午6:00触发
     */
    @Scheduled(cron="0 0 6 * * ?")
    protected void day(){
        try{
            logger.info("TaskReport.day.start");
            //统计每天订单报表;
            Integer today = 0;//0表示今天    1表示昨天;
            reportDayService.insertBatch(today);
            //统计买家每日订单金额;
            buyerOrderReportService.insertBatch(today);
        }catch(Exception e){
            logger.error("TaskReport.day.exception",e);
        }finally {
            logger.info("TaskReport.day.end");
        }
   }
```

2.相关的报表的形状显示，目前折线图，柱状图是比较理想的一种方式，采用百度的echarts进行显示

相关的运行实例如下：




