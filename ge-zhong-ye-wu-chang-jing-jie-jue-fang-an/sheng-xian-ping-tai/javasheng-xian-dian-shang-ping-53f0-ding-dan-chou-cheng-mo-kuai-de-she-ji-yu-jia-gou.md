# Java生鲜电商平台-订单抽成模块的设计与架构

说明：订单抽成指的是向卖家收取相应的信息服务费.(目前市场上有两种抽成方式，一种是按照总额的抽成比率，另外一种是按照订单明细的抽成比率)

由于生鲜电商的垂直领域的特殊性质，总额抽成不切合实际，所以按照订单的明细抽成。

1.订单抽成，是按照一个区的维度，以及菜品的二级分类类抽点的。

举例说明：比如武汉光谷区，佛祖岭区，虽然都是属于东湖高新，但是光谷区的物价以及消费水平肯定是高于佛祖岭区的，因此它是按照一个区的维度来分的。

但是有些卖家挣的钱多，有的卖家挣的钱少，虽然同属于一个菜市场，但是挣钱少的跟挣钱多的一样抽成，他们也不乐意，因此又按照卖家的ID进行抽查。

最终根据业务的形态以及我们的市场调查结果，采用以商户为基准，按照菜品的二级分类来抽成的一种盈利模式。

**2.数据库系统架构设计如下：**

卖家抽点配置信息表：

```
CREATE TABLE `order_percentage_config` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键，自动增加ID',
  `region_id` bigint(20) NOT NULL COMMENT '区域id',
  `seller_id` bigint(20) DEFAULT NULL COMMENT '卖家id',
  `category_id` bigint(20) DEFAULT NULL COMMENT '商品二级分类的ID',
  `percentage` decimal(12,2) DEFAULT NULL COMMENT '平台抽点率 ',
  `status` tinyint(1) DEFAULT NULL COMMENT '状态: 0禁用 ，1启用',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间 ',
  `update_time` datetime DEFAULT NULL COMMENT '修改时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=144 DEFAULT CHARSET=utf8 COMMENT='订单抽点配置信息表';
```

说明：按照卖家的所属二级分类进行抽点，不同的抽点率也是不一样的。

2.卖家抽点明细表。（系统需要精确的记录，那个菜品进行了抽点，抽点多少，原来的多少钱，抽点后多少钱，需要明明白白的，让卖家清清楚楚的有一笔账）


```
CREATE TABLE `order_percentage` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `order_item_id` bigint(20) DEFAULT NULL COMMENT '订单item的ID',
  `order_number` varchar(64) DEFAULT NULL COMMENT '所属订单号',
  `seller_id` bigint(20) DEFAULT NULL COMMENT '卖家id',
  `order_total_amount` decimal(12,2) DEFAULT NULL COMMENT '订单金额',
  `order_percentage` decimal(12,2) DEFAULT NULL COMMENT '抽点比率',
  `order_percentage_amount` decimal(12,2) DEFAULT NULL COMMENT '抽点金额',
  `order_reality_money` decimal(12,2) DEFAULT NULL COMMENT '订单最终金额即用户最终收到的金额',
  `create_time` datetime DEFAULT NULL COMMENT '所属时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=47226 DEFAULT CHARSET=utf8 COMMENT='抽点信息总表';
```
补充说明：1. 由于的是对某一个明细进行抽点。所以需要卖家ID，订单明细ID以及抽点比率等，最终算出抽点金额。

3.何时进行抽点呢
回答：每天早上6点30进行昨天的抽点,形成自己的账单。
相关的核心代码如下：采用spring task做定时器


```
/**
     * 计算订单抽成 每天早上6:30点执行
     */
    @Scheduled(cron = "0 30 6 * * ?")
    protected void makeOrderPercentage() {
        try {
            logger.info("TasksQuartz.makeOrderPercentage.start");
            // 计算订单抽成，并生成抽成数据
            orderPercentageService.batchSaveOrderPercentage();
            logger.info("TasksQuartz.makeOrderPercentage.end");
        } catch (Exception ex) {
            logger.error("TasksQuartz.makeOrderPercentage.exception", ex);
        }
    }
```
业务核心代码：


```
@Service
public class OrderPercentageserviceImpl implements OrderPercentageService {

    private static final Logger logger = LoggerFactory.getLogger(OrderPercentageserviceImpl.class);

    @Autowired
    private OrderItemDao orderItemDao;

    @Autowired
    private OrderPercentageConfigDao orderPercentageConfigDao;

    @Autowired
    private OrderPercentageDao orderPercentageDao;

    /**
     * 批量新增数据
     */
    @Override
    public void batchSaveOrderPercentage() {

        // 1.获取昨天12：00到今天早上6：00的所有的订单明细
        Map<String, Object> dateMap = DateUtil.getTradeTime(0);

        List<OrderItem> orderItems = orderItemDao.getOrderItemsByDate(dateMap);

        List<OrderPercentage> list = new ArrayList<OrderPercentage>();

        if (CollectionUtils.isEmpty(orderItems)) {
            return;
        }

        for (OrderItem item : orderItems) {
            try {
                Long formatId = item.getFormatId();
                Long sellerId = item.getSellerId();

                OrderPercentageConfig config = orderPercentageConfigDao.getOrderPercentageConfig(formatId, sellerId);

                if (config == null) {
                    logger.error("batchSaveOrderPercentage.config.isEmpty:" + "formatId:" + formatId + " sellerId:"
                            + sellerId);
                    continue;
                }

                OrderPercentage orderPercentage = new OrderPercentage();
                // 订单总额
                BigDecimal orderTotalAmount = item.getGoodsAmount();

                // 抽点比率
                BigDecimal percentage = config.getPercentage();

                // 抽点金额
                BigDecimal orderPercentageAmount = orderTotalAmount.multiply(percentage)
                        .multiply(new BigDecimal("0.01")).setScale(3, BigDecimal.ROUND_UP);

                // 卖家最终所得
                BigDecimal orderRealityMoney = orderTotalAmount.subtract(orderPercentageAmount).setScale(3,
                        BigDecimal.ROUND_UP);

                orderPercentage.setCreateTime(new Date());
                orderPercentage.setOrderItemId(item.getId());
                orderPercentage.setOrderNumber(item.getOrderNumber());
                orderPercentage.setOrderTotalAmount(orderTotalAmount);
                orderPercentage.setOrderPercentage(percentage);
                orderPercentage.setOrderPercentageAmount(orderPercentageAmount);
                orderPercentage.setOrderRealityMoney(orderRealityMoney);
                orderPercentage.setSellerId(sellerId);
                list.add(orderPercentage);
            } catch (Exception ex) {
                logger.error("batchSaveOrderPercentage.exception", ex);
            }
        }

        try {
            orderPercentageDao.batchSaveOrderPercentage(list);
        } catch (Exception ex) {
            logger.error("batchSaveOrderPercentage", ex);
        }
    }
}
```
**相关后台运营截图如下:**
![](/static/image/641237-20180519093105185-1411784365.png)
![](/static/image/641237-20180519093213091-279286236.png)
