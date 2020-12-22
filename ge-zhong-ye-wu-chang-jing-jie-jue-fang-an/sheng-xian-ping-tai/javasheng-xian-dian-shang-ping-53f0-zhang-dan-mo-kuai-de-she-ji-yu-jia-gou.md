# Java生鲜电商平台-账单模块的设计与架构

补充说明：生鲜电商平台-账单模块的设计与架构，即用户的账单形成过程。

由于系统存在一个押账功能的需求，（何为押账，就是形成公司的资金池，类似摩拜单车，ofo单车等等）。目前B2B平台也是采用押账的这种功能策略。

**这里有个特别说明的押账方式：就是比如有个卖家张三，他是5月1日跟我们平台签约开始入住平台卖菜，我们约定好押账7天，那么他5月1日的金额会在5月2日存入**

**他自己的余额里面，但是这个钱不能马上提取出来，需要等一个星期，也就是5月8日可以提现5月1日的金额，5月9日可以提现5月2日以前的所有金额。**

**这个算法的最大好处就是永远的压住客户7天的金额。**

**这个算法采用的是Spring quartz定时器每天晚上23:00点处理的。**

相关核心的代码如下：


```
/**
 * 任务工作
 * @author wangfucai
 */
@Component
public class TasksQuartz{
    
    private static final Logger logger=LoggerFactory.getLogger(TasksQuartz.class);
    
    @Autowired
    private BillService billService;
    @Autowired
    private SellerService sellerService;
    @Autowired
    private DeliveryIncomeService deliveryIncomeService;
    @Autowired
    private BuyerService buyerService;
    @Autowired
    private OrderInfoService orderInfoService;
    @Autowired
    private GroupsBuyerService groupsBuyerService;
    
    /**
     * 计算每天账单
     * 每天23点执行
     */
    @Scheduled(cron="0 0 23 * * ?")
    protected void makeBill(){
        try
        {
            logger.info("TasksQuartz.execute.start");
            //统计当天的交易完成的订单生成账单
            billService.addBills();
            logger.info("账单数据更新完成");
            //根据卖家抽点金额更新账单实际金额
            billService.updateRealAmountByPercentage();
            logger.info("根据卖家抽点金额更新账单实际金额完成");
            //更新卖家余额
            sellerService.updateBalanceByBill();
            logger.info("卖家余额数据更新完成");
            logger.info("TasksQuartz.execute.end");
        }catch(Exception ex)
        {
            logger.error("TasksQuartz.execute.exception",ex);
        }
    }
```

