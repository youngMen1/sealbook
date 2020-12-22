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
补充说明：

1.需要统计每个卖家今天的收入。

2.并行的需要把订单的数据存入账单表。

3.余额来源于账单表。形成一个数据的流转体现。

账单表的表结构如下：

![](/static/image/641237-20180517211449884-1101676858.png)

补充说明：每天定时器会根据卖家的账期形成账单，最终更新到用卖家的余额里面。

实际运营情况来讲是每个卖家的账期是不一样的，有的两天，有的三天，有的一周，有的是一个月。

相关核心算法与代码如下：


```
/**
     * 统计10天前的账单更新卖家余额和账单金额
     */
    @Override
    public void updateMoney() {
        // 获取10天前的日期d
        String day = DateUtil.dateToString(DateUtil.addDay(new Date(), -9), DateUtil.FMT_DATE);
        // 查询十天前的所有帐单信息
        List<Map<String, Object>> list = billDao.getBillsByDay(day);
        if (CollectionUtils.isEmpty(list)) {
            logger.info("TasksQuartz.updateMoney.isEmpty-->day:" + day);
            return;
        }
        for (Map<String, Object> map : list) {
            // 卖家ID
            Long sellerId = (Long) map.get("sellerId");
            if (sellerId == null) {
                continue;
            }
            // 获取提现的金额即最终账单的金额
            BigDecimal realityMoney = (BigDecimal) map.get("realIncome");
            if (realityMoney == null) {
                continue;
            }
            // 获取卖家的余额
            BigDecimal balanceMoney = (BigDecimal) map.get("balanceMoney");
            if (balanceMoney == null) {
                balanceMoney = BigDecimal.ZERO;
            }
            // 获取卖家的账单金额
            BigDecimal billMoney = (BigDecimal) map.get("billMoney");
            if (billMoney == null) {
                billMoney = BigDecimal.ZERO;
            }
            // 金额相加
            BigDecimal resultBalanceMoney = realityMoney.add(balanceMoney);

            BigDecimal resultBillMoney = realityMoney.add(billMoney);

            logger.info("当前用户sellerId:" + sellerId + " 当前的余额为：balanceMoney=" + balanceMoney
                    + " 最终金额：resultBalanceMoney=" + resultBalanceMoney);

            logger.info("当前的余额为：billMoney=" + billMoney + " 最终金额：resultBillMoney=" + resultBillMoney);
            // 更新卖家余额和账单金额
            int result = sellerDao.updateMoney(sellerId, resultBalanceMoney, resultBillMoney);
            logger.info("当前用户sellerId:" + sellerId + " 更新结果为：" + (result > 0));
        }
        // 更新十天前的所有账单的状态
        int count = billDao.updateStatus(day);
        logger.info(" 更新" + count + "条账单，状态变为已结算");
    }
```
业务说明：

1.无外乎每天需要统计卖家的今日收益情况。

2.更新卖家的最终余额。

3.根据卖家的所设置的账单周期，形成用户的账单金额。

4.最终根据账单金额，形成用户的可提现余额的过程。

业务有点绕口，但是整体是非常地清晰的，思路就是押用户所配置的账期金额。配置10天就压10天，配置15天就压15天。

以下是账单跟卖家的核心关联表，就是配置所属的卖家对应的所属账期时间。
![](/static/image/641237-20180517212124415-1664681088.png)
总结：整个技术方面其实都不算复杂，主要是业务逻辑以及统计的一些概念，希望这些定时器计算，账单思路形成，架构方面能给大家一些帮助。