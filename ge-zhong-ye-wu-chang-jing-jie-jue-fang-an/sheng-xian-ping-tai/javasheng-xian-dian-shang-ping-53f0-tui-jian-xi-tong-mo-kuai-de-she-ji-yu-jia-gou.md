# Java生鲜电商平台-推荐系统模块的设计与架构

**业务需求：**

对于一个B2B的生鲜电商平台，对于买家而言，他需要更加快速的购买到自己的产品，跟自己的餐饮店不相关的东西，他是不关心的，而且过多无用的东西掺杂在一起，反而不便

于买家下单，用户体验也很差，严重的会因此丢了客户。（客户觉得太难用了。一般都就会放弃使用.）

对于卖家而言，他自己就调整下自己的商品的上架与下架，然后就是调整下自己商品的价格。（蔬菜类的商品会随着市场的供求关系会有相应的波动.）

**业务分析：**

推荐系统：根据买家的行为习惯以及购买行为来推荐些他可能需要的东西的一套算法系统。

对于买家而言，数量来源于以下三个维度：

1.购买记录。-----买家实际下的订单。

2.收藏夹。   -----对于买家而言，收藏了某个商品，但是并没购买的，我们认为他也会购买，属于需要推送的数据之一。

3.常用清单。----用户最近一段时间购买的记录，我们业务分析认为他一定会再次购买，因为相对一个餐馆而言，它所做的菜从某种程度来说是一定的，所以购买的食材，也相对而言也是类似的。也就是说昨天买的，今天可能也会再次够买，只是数量有所变化而言。

4.同类推荐。  ----对于一个餐馆而言，比如说小炒类似的餐馆，那么很多类似小炒的餐馆的所有菜应该也是类似的，也许存在不需要的，但是也存在可能需要的情况，也属于我们的推荐系统中的。

5.系统推荐。  ----对于一个刚注册的买家而言，我们希望给他更好的业务体验，那么在注册的时候，他就会一定选择一个所属类别，根据类别，我们会把相应的类别的系统清单推荐出来，让客户一进来就感觉到这些他所需要的菜都好像是系统跟他量身定做的一样。


根据以上的业务分析，我们理清楚了上述的所有维度，以下是数据库的设计与思路：

1.购买记录。来源于订单明细记录表：


```
CREATE TABLE `order_item` (
  `item_id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '主键',
  `order_id` bigint(20) DEFAULT NULL COMMENT '订单主表id,order_info表的order_id',
  `order_number` varchar(32) DEFAULT NULL COMMENT '唯一订单号',
  `order_status` tinyint(4) DEFAULT NULL COMMENT '订单项状态，1为已提交订单，2为取消订单',
  `format_id` bigint(20) DEFAULT NULL COMMENT '商品规格的ID',
  `buyer_id` bigint(20) unsigned DEFAULT '0' COMMENT '买家ID',
  `seller_id` bigint(20) DEFAULT NULL COMMENT '所属卖家ID',
  `delivery_type` tinyint(2) DEFAULT '1' COMMENT '配送类型，1为平台送,2.卖家自己送',
  `delivery_status` tinyint(2) DEFAULT '0' COMMENT '配送状态，0 表示未收货，1表示已收货,送货中，2表示已收货，已送货',
  `seller_status` tinyint(4) DEFAULT '0' COMMENT '卖家备货状态，0为备货中，1为备货完成,2为缺货',
  `buyer_status` tinyint(2) unsigned DEFAULT '0' COMMENT '买家状态，0待收货，1为已收货，2为换货，3为退货',
  `remark` varchar(255) DEFAULT NULL COMMENT '订单项备注，由用户提交订单前填写',
  `goods_number_old` decimal(12,2) DEFAULT NULL COMMENT '订单初始商品数量',
  `goods_number` decimal(12,2) DEFAULT NULL COMMENT '商品的数量',
  `goods_price` decimal(12,2) DEFAULT NULL COMMENT '商品的单价',
  `goods_amount` decimal(12,2) DEFAULT NULL COMMENT '单项总金额',
  `delivery_money` decimal(12,2) DEFAULT '0.00' COMMENT '配送费用',
  `create_time` datetime DEFAULT NULL COMMENT '订单创建时间',
  `delivery_receive_time` datetime DEFAULT NULL COMMENT '配送人员收货时间',
  `delivery_finish_time` datetime DEFAULT NULL COMMENT '配送人员完成时间',
  `seller_finish_time` datetime DEFAULT NULL COMMENT '卖家完成时间',
  `buyer_finish_time` datetime DEFAULT NULL COMMENT '买家完成时间',
  `method_id` bigint(20) DEFAULT NULL COMMENT '加工方式ID',
  `delivery_id` bigint(20) DEFAULT NULL,
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3424 DEFAULT CHARSET=utf8 COMMENT='订单的子项目';
```

2.收藏夹系统数据库表：
641237-20180516090321352-263992709.png

说明：收藏夹比较简单，某个商品ID，那个买家，什么时候收藏的。

3.常用清单
641237-20180516090445543-2024111417.png

说明：常用清单，维度也是对于商品而言，不是针对某一个店铺，因为我们市场反馈给出的结论是买家关注的商品本身，而不是那个卖家。

4.同类分析。
说明：数据来源于类似的餐馆点。我们把餐馆店分为几种类型，在地推团队来销售产品的时候，其实是知道那个餐馆的所属类别的。
（客户的类型，1为火锅店，2为小餐馆，3为中餐馆，4,为烧烤）

5.系统推荐。
641237-20180516090955374-2055177973.png
说明：系统推荐，跟收藏夹，买家常用清单功能都很类似，不同点就在于业务的范围与范畴。

刚注册的用户的常用清单的数据就来源于系统推荐的数据。


**相关业务核心代码如下：**

1.注册代码中添加


```
/**
     * 买家注册,第二步完善资料
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/register/second/step", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult secondStepRegister(HttpServletRequest request, HttpServletResponse response,@RequestBody Buyer buyer) 
    {
        logger.info("UsersController.secondStepRegister.seller:新增买家：" + buyer);
        if (buyer == null)
        {
            return new JsonResult(JsonResultCode.FAILURE, "参数异常", "");
        }
        try 
        {
            buyerService.updateBuyer(buyer);
            //添加买家默认的常用清单
            buyerService.insertBuyerCommon(buyer.getBuyerId(), buyer.getBuyerType(), buyer.getRegionId());
            return new JsonResult(JsonResultCode.SUCCESS, "完善买家信息成功",buyer);
        } catch (Exception e) {
            logger.error("[UsersController][secondStepRegister] exception :", e);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
```
2.常用清单方面


```
/**
     * 我的常用清单
     */
    @RequestMapping(value = "/my/commonList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult commonList(HttpServletRequest request, HttpServletResponse response, Long userId) {
        try {
            List<CommonListVo> list = buyerService.getCommonList(userId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        } catch (Exception ex) {
            logger.error("[MyController][commonList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
```
3.系统常用清单
说明：系统常用清单来源于后台管理人员人工添加
相应代码如下:


```
/**
     * 到新增页面;
     */
    @RequestMapping(value = "/toAdd", method = { RequestMethod.GET, RequestMethod.POST })
    public String toAdd(HttpServletRequest request, HttpServletResponse response, Model model, SysCommonVo sysCommonVo,@ModelAttribute SearchGoodsVo sgv) {
        
        // 获取分页当前的页码
        int currentPageNum = this.getPageNum(request);
        
        // 获取分页的大小
        int currentPageSize = this.getPageSize(request);
        
        //区域ID
        Long areaId = sysCommonVo.getAreaId();
        
        sgv.setAreaId(areaId);
        sgv.setSearchStatus((short) 1);
        sgv.setFormatStatus((short)1);
        sgv.setSellerStatus((short)3);
        List<SysCommonVo> sysCommon = sysCommonService.getSysCommon(sysCommonVo);
        StringBuffer sb = new StringBuffer();
        
        if(sysCommon.size()>0){
            for (int i = 0; i < sysCommon.size(); i++) {
                if(i != sysCommon.size()-1){
                    SysCommonVo sc = sysCommon.get(i);
                    sb.append(sc.getGoodsId());
                    sb.append(",");
                }else {
                    SysCommonVo sc = sysCommon.get(i);
                    sb.append(sc.getGoodsId());
                }
            }
        }
        sgv.setGoodIds(new String(sb));
        
        PageUtil paginator = goodsService.getPageResultByCommon(sgv, currentPageNum, currentPageSize);
        model.addAttribute("paginator", paginator);
        model.addAttribute("sgv", sgv);
        model.addAttribute("sysCommonVo", sysCommonVo);
        return "sys/common/addFrom";
    }
```
说明：相对而言，这个代码都是强依赖于数据库，毕竟不可能很多时间都有人同时买菜与注册。很多时候都是联表查询即可完成数据的分析与统计。

5.定时器代码。（数据的系统推荐与个性化推荐都是系统采用定时器进行处理的。spring task）

相关代码如下：
641237-20180516091920065-1819049908.png

总结：所有的推荐系统的模型都类似我上面来的几个维度的思考，需要根据自己的业务实际情况，自己分析与总结，至于是同步还是异步，还是定时器等等都是处理手段，

我这边就采用了，同步与异步，包括定时器同时计算的过程，最终达到用户的推荐效果。

