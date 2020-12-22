# Java生鲜电商平台-财务系统模块的设计与架构

前言:任何一个平台也好，系统也好，挣钱养活团队这个是无可厚非的，那么对于一个生鲜B2B平台盈利模式\( 查看：[http://www.cnblogs.com/jurendage/p/9016411.html\)而言，](http://www.cnblogs.com/jurendage/p/9016411.html%29而言，)

其中财务模块无论是对于买家而言还是卖家而言都至关重要，老百姓对钱的看重是没有经历的人想不到的，一句话说清楚了：一分钱也不能少。

买家或者卖家对财务模块的要求很简单：  
1.账清楚明白。

2.消费清清楚楚。

3.计算准确无误。

对平台而言：财务的要求是每笔资金的输入与输出，有理有据，真真实实。

我们分两个模块来讨论，买家与卖家我们称为客户财务模块，平台我们称为公司财务模块

## 1.客户财务模块

1.1 买家需要查看任何一天的消费记录。数据按照时间的倒叙排列，根据时间进入某天的消费记录，需要查看当天的历史订单数据。

业务中按照时间来分组，数据是来源于订单表：

相关业务核心代码如下：

```
/**
     * 订单列表
     * 
     * @description 1.用户的所有订单列表。 2.支付状态 3.交易状态.
     * @param request
     * @param response
     * @param orderStatus
     *            0 待支付 1待送达 2已完成
     * @return
     */
    @RequestMapping(value = "/order/list", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult buyerOrderList(HttpServletRequest request, HttpServletResponse response, Long buyerId,
            int orderStatus) {

        List<OrderListVo> listVo = new ArrayList<OrderListVo>();

        // 需要顺序
        Map<String, List<OrderInfoVo>> resultMap = new TreeMap<String, List<OrderInfoVo>>(new Comparator<String>() {
            public int compare(String obj1, String obj2) {
                // 降序排序
                return obj2.compareTo(obj1);
            }
        });

        try {
            List<OrderInfo> orderList = orderInfoService.getAllOrderInfo(buyerId, orderStatus);

            // 判断是否有订单
            if (CollectionUtils.isEmpty(orderList)) {
                return new JsonResult(JsonResultCode.SUCCESS, "订单查询成功", listVo);
            }

            // 组装Map对象
            for (OrderInfo order : orderList) {
                List<OrderInfoVo> resultList = new ArrayList<OrderInfoVo>();

                OrderInfoVo orderInfoVo = transform(order);

                String time = DateUtil.dateToString(order.getCreateTime(), "yyyy-MM-dd");

                if (resultMap.get(time) == null) {
                    resultList.add(orderInfoVo);
                    resultMap.put(time, resultList);
                } else {
                    List<OrderInfoVo> mapList = resultMap.get(time);
                    mapList.add(orderInfoVo);
                    resultMap.put(time, mapList);
                }
            }

            // 组装VO
            Set<String> keys = resultMap.keySet();
            for (String data : keys) {
                OrderListVo vo = new OrderListVo();
                vo.setDate(data);
                vo.setList(resultMap.get(data));
                listVo.add(vo);
            }
            return new JsonResult(JsonResultCode.SUCCESS, "订单查询成功", listVo);
        } catch (Exception ex) {
            logger.error("[OrderController][buyerOrderList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
```

说明：买家而言其实就是待支付，已经支付等订单的明细查询，根据时间进行分组，根据时间（按照天来计算，查看自己的明细。），然后查询出整个订单明细即可。

相关运营截图如下：

![](/static/image/641237-20180517085935058-1740854447.png)  
![](/static/image/641237-20180517085952512-1474130378.png)  
![](/static/image/641237-20180517090012649-1736471219.png)

1.2.对于卖家而言，他肯定想知道每天具体卖了多少东西，多少钱，但是不是针对某一个买家而言，是针对整个统计而言。

比如：他想知道我今天卖了土豆多少共多少斤，白菜多少斤，萝卜多少斤，花菜多少斤等等，同时他自己会算出今天的利润多少。

补充说明：其实系统是可以算出来今天他挣钱多少的，但是客户很讨厌输入进货价，相当于把自己的“秘密”出卖了一样，整个系统架构中

先前是有的，但是实际运营发现，不是我们的那个样子，最终是去掉了。

**相关的业务核心代码如下：（数据也是来源于订单明细表中）**

订单主表的列表：

```
/**
     * 查询买家需要确认的订单列表
     */
    @RequestMapping(value = "/order/list", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult orderList(HttpServletRequest request, HttpServletResponse response,Integer type,Long saleId) {
        try {
            List<OrderVo> orderList = orderService.getOrderList(type, saleId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", orderList);
        } catch (Exception ex) {
            logger.error("[OrderController][orderList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
```

根据订单号获取订单明细：

```
/**
     * 查询买家需要确认的订单列表
     */
    @RequestMapping(value = "/order/detail", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getOrderDetail(HttpServletRequest request, HttpServletResponse response,Long orderId) {
        try {
            OrderDetailVo odv = orderService.getOrderDetail(orderId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", odv);
        } catch (Exception ex) {
            logger.error("[OrderController][getOrderDetail] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
```

订单明细VO:对象：

```
/**
 * 订单详情VO
 */
public class OrderDetailVo implements java.io.Serializable {

    private static final long serialVersionUID = 1L;
    /**
     * 订单主表id,order_info表的order_id
     */
    private Long orderId;
    /**
     * 唯一订单号
     */
    private String orderNumber;
    /**
     * 买家ID
     */
    private Long buyerId;
    /**
     * 买家店铺名称
     */
    private String buyerName;
    /**
     * 买家店铺图片
     */
    private String buyerLogo;
    /**
     * 买家地址
     */
    private String buyerAddress;
    /**
     * 买家姓名
     */
    private String bossName;
    /**
     * 买家手机
     */
    private String bossTel;
    /**
     * 收货人的最佳送货时间
     */
    private String bestTime;
    /**
     * 订单创建时间
     */
    private String createTime;
    /**
     * 商品列表
     */
    private List<OrderGoodsVo> goodsList;

    public Long getOrderId() {
        return orderId;
    }
    public void setOrderId(Long orderId) {
        this.orderId = orderId;
    }
    public String getOrderNumber() {
        return orderNumber;
    }
    public void setOrderNumber(String orderNumber) {
        this.orderNumber = orderNumber;
    }
    public Long getBuyerId() {
        return buyerId;
    }
    public void setBuyerId(Long buyerId) {
        this.buyerId = buyerId;
    }
    public String getBuyerName() {
        return buyerName;
    }
    public void setBuyerName(String buyerName) {
        this.buyerName = buyerName;
    }
    public String getBuyerLogo() {
        return buyerLogo;
    }
    public void setBuyerLogo(String buyerLogo) {
        this.buyerLogo = buyerLogo;
    }
    public String getBuyerAddress() {
        return buyerAddress;
    }
    public void setBuyerAddress(String buyerAddress) {
        this.buyerAddress = buyerAddress;
    }
    public String getBossName() {
        return bossName;
    }
    public void setBossName(String bossName) {
        this.bossName = bossName;
    }
    public String getBossTel() {
        return bossTel;
    }
    public void setBossTel(String bossTel) {
        this.bossTel = bossTel;
    }
    public String getBestTime() {
        return bestTime;
    }
    public void setBestTime(String bestTime) {
        this.bestTime = bestTime;
    }
    public String getCreateTime() {
        return createTime;
    }
    public void setCreateTime(String createTime) {
        this.createTime = createTime;
    }
    public List<OrderGoodsVo> getGoodsList() {
        return goodsList;
    }
    public void setGoodsList(List<OrderGoodsVo> goodsList) {
        this.goodsList = goodsList;
    }
}
```

相关SQL核心：

```
<!-- 订单详情 -->
  <select id="getOrderDetail" resultMap="orderDetailMap">
      select 
        o.order_id,o.order_number,o.buyer_id,b.buyer_name,b.buyer_logo,b.buyer_address,b.boss_name,b.boss_tel,
        date_format(o.best_time,'%Y-%m-%d %H:%i') as best_time,date_format(o.create_time,'%Y-%m-%d %H:%i') as create_time,
        oi.item_id,oi.remark,g.goods_id,g.goods_name,g.goods_brand,g.goods_label,
        gf.format_id,gf.format_name,u.unit_id,u.unit_name,gf.format_num,
        pm.method_id,pm.method_name,oi.goods_price,oi.goods_number,oi.goods_amount,
        gp.pic_id,gp.pic_url,gp.is_main,gp.pic_seq
     from order_item oi
        inner join order_info o on o.order_id=oi.order_id
        inner join buyer b on b.buyer_id=o.buyer_id
        inner join goods_format gf on gf.format_id=oi.format_id
        inner join goods g on gf.goods_id=g.goods_id
        inner join unit u on gf.unit_id=u.unit_id
        left join process_method pm on pm.method_id=oi.method_id 
        left join goods_picture gp on g.goods_id=gp.goods_id
     where oi.order_id=#{orderId} 
     order by gp.is_main desc
  </select>
```

\*\*  
疑问一：为什么会贴出如此多的代码吗？尤其是SQL?

理由只有一点：的确这块比较复杂，很多都是基于SQL语句的查询，需要SQL功底。

相关的业务运营截图如下：\*\*

![](/static/image/641237-20180517091812983-816182074.png)  
![](/static/image/641237-20180517091827907-1405584471.png)  
![](/static/image/641237-20180517091835802-1759250204.png)  
![](/static/image/641237-20180517091846645-586099403.png)

下面的核心的历史收益这块，卖家可以扔掉自己的手抄本了。根据这个平台就可以方便的明细的账单记录  
![](/static/image/641237-20180517091937452-1310210253.png)  
![](/static/image/641237-20180517091948138-403235976.png)  
![](/static/image/641237-20180517092002652-258321379.png)

\*\*  
补充说明：数据报表有金额，有数据，有统计，有分析，包括以后的折线图，柱状图等等报表分析，让用户明确的知道今天挣钱多少，明天预警备货多少，一切清楚明了。

总结：我的经验是所有的人心与人性都是相同的，你需要功能的客户其实也想看到，对于这种模式的采用，我们是经历了很多都经验后才总结出来的，  
看起来很简单，其实背后的努力不简单。一起努力做好生鲜电商。  
\*\*

