# Java生鲜电商平台-财务系统模块的设计与架构 

前言:任何一个平台也好，系统也好，挣钱养活团队这个是无可厚非的，那么对于一个生鲜B2B平台盈利模式( 查看：http://www.cnblogs.com/jurendage/p/9016411.html)而言，

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

641237-20180517085935058-1740854447.png
641237-20180517085952512-1474130378.png
641237-20180517090012649-1736471219.png

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