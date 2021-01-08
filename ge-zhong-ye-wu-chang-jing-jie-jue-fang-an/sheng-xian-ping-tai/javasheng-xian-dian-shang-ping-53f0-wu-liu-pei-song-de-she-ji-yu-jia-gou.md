# Java生鲜电商平台-物流配送的设计与架构

说明：由于Java开源生鲜电商平台是属于自建物流系统，也就是买家下的单，需要公司派物流团队进行派送。

业务需求中买家的下单时间控制在：12:00-03:00之间。这段时间可以进行下单。

## 1.业务分析：

物流团队需要知道以下东西。

1.配送师傅需要知道去那个菜市场去哪个卖家那里拿到那个买家的货，由于买家买的菜是全品类，但是卖家卖的菜是单品，所以需要进行袋子组装。

2.配送师傅需要确认按照买家分组，知道那些菜是否组装了，那些菜没有进行组装。

3.对于已经送完的买家需要物流APP操作已经送完，方便整个系统的运营与管理。


## 2.业务架构：
1.根据业务的分析，我们得出以下几个数据库表的设计

2.物流人员的管理表，其中由于物流的经理是可以看到所有的区域以及所有的物流的情况。

3.需要根据GPS定位知道目前配送师傅的具体地址。

4.配送师傅知道今天要去那几家取货，而且取多少，需要一目了然。

最新表结构如下：

3.配送人员基础信息表：


```
CREATE TABLE `delivery` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `delivery_phone` varchar(16) DEFAULT NULL COMMENT '手机号码,作为账号登陆',
  `delivery_password` varchar(64) DEFAULT NULL COMMENT '密码',
  `delivery_name` varchar(16) DEFAULT NULL COMMENT '配送人员姓名',
  `delivery_sex` int(11) DEFAULT NULL COMMENT '性别，1为男，2为女',
  `delivery_idcard` varchar(18) DEFAULT NULL COMMENT '身份证号码',
  `delivery_age` int(11) DEFAULT NULL COMMENT '配送人员年龄',
  `delivery_type` int(11) DEFAULT NULL COMMENT '配送人员类型：1为自营，2为加盟',
  `area_id` bigint(20) DEFAULT NULL COMMENT '所属区域',
  `sequence` int(11) DEFAULT NULL COMMENT '排序使用.从小到大排序',
  `status` int(11) DEFAULT NULL COMMENT '1为可用，-1为不可用',
  `remark` varchar(256) DEFAULT NULL COMMENT '备注信息',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后更新时间',
  `permission_type` tinyint(4) DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=32 DEFAULT CHARSET=utf8 COMMENT='配送人员基本信息';
```
补充说明：增加了一个permission_type来确定当前用户账号的权限。相关的表结构的注释写的比较清楚。

4.配送师傅配送区域信息表


```
CREATE TABLE `delivery_area` (
  `da_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `da_area_id` bigint(20) DEFAULT NULL COMMENT '区域ID',
  `da_address` varchar(255) DEFAULT NULL COMMENT '地址',
  `da_status` tinyint(4) DEFAULT NULL COMMENT '状态 1在用 -1停用',
  `da_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `da_create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`da_id`)
) ENGINE=InnoDB AUTO_INCREMENT=36 DEFAULT CHARSET=utf8 COMMENT='配送区域';
```
说明：不可能全城所有的物流都是一个师傅配送吧，每个师傅需要看到属于自己配送的买家，仅此而已，我们按照区域维度进行划分。

5.我们需要知道配送师傅目前的GPS地理位置，那么需要建立一个数据进行存储


```
CREATE TABLE `delivery_coordinate` (
  `id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '自增ID',
  `delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `lng` varchar(60) DEFAULT NULL COMMENT '经度',
  `lat` varchar(60) DEFAULT NULL COMMENT '维度',
  `address` varchar(255) DEFAULT NULL COMMENT '地址',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=152063 DEFAULT CHARSET=utf8 COMMENT='配送司机坐标';
```
备注说明：平均每隔30秒，进行系统的主动上报，最终会形成一个配送师傅的坐标。

6.配送人员管理区域信息表


```
CREATE TABLE `delivery_da` (
  `dda_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `dda_delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `dda_da_id` bigint(20) DEFAULT NULL COMMENT '配送区域ID',
  `dda_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `dda_create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`dda_id`)
) ENGINE=InnoDB AUTO_INCREMENT=465 DEFAULT CHARSET=utf8 COMMENT='配送人员管理区域';
```

7.对于有些特殊的情况，比如车在路上坏了，那么需要进行人工分配物流

```
CREATE TABLE `delivery_task` (
  `task_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `delivery_id` bigint(20) DEFAULT NULL COMMENT '配送人员ID',
  `order_id` bigint(20) DEFAULT NULL COMMENT '订单ID',
  `order_item_id` bigint(20) DEFAULT NULL COMMENT '订单明细ID',
  `sys_user_id` bigint(20) DEFAULT NULL COMMENT '分配人员ID',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`task_id`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

最终形成了完整的业务逻辑闭环


## 相关的业务核心代码如下：


```

/**
 * 配送端订单
 */
@RestController
@RequestMapping("/delivery")
public class OrderV1Controller extends BaseController {

    private static final Logger logger = LoggerFactory.getLogger(OrderV1Controller.class);

    @Autowired
    private OrderV1Service orderV1Service;
    @Autowired
    private DeliveryCoordinateService deliveryCoordinateService;
    @Autowired
    private DeliveryService deliveryService;
    @Autowired
    private AreaService areaService;
    
    /**
     * 统计配送人员订单数
     */
    @RequestMapping(value = "/order/index", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult index(HttpServletRequest request, HttpServletResponse response,Model model,Long deliveryId) {
        try {
            //全部
            DeliveryVo all = deliveryService.getDeliveryVo(0,deliveryId,null);
            //已送
            DeliveryVo send = deliveryService.getDeliveryVo(1,deliveryId,2);
            //未送
            DeliveryVo noSend = deliveryService.getDeliveryVo(1,deliveryId,1);
            int sendCount = send.getOrderNum();
            BigDecimal totalAmt = all.getTotalAmt() == null ? BigDecimal.ZERO :all.getTotalAmt();
            int allCount = all.getOrderNum();
            int noSendCount = noSend.getOrderNum();
            model.addAttribute("noSendCount",noSendCount);
            model.addAttribute("sendCount",sendCount);
            model.addAttribute("allCount",allCount);
            model.addAttribute("totalAmt",totalAmt);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", model);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][index] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 取货订单列表(按卖家查询) 
     */
    @RequestMapping(value = "/order/pickUplistBySeller", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult pickUplistBySeller(HttpServletRequest request, HttpServletResponse response,Long sellerId,Long deliveryId,Short deliveryType,Short deliveryStatus,Short payStatus) {
        try {
            //查询订单明细
            List<PickUpItemVo> orderItemList = orderV1Service.getPickUpItemList(null,sellerId,deliveryId,deliveryType,deliveryStatus,payStatus);
            //返回取货列表
            List<PickUpListVo> pickUpList = new ArrayList<PickUpListVo>(); 
            //返回结果
            List<BuyerVo> buyerList = new ArrayList<BuyerVo>();
            //分组判断
            Map<Long,BuyerVo> buyerMap = new HashMap<Long,BuyerVo>();
            //分组判断
            Map<Long,List<BuyerVo>> pickUpMap = new TreeMap<Long,List<BuyerVo>>(new Comparator<Long>(){
                public int compare(Long obj1, Long obj2) {
                    // 降序排序
                    return obj2.compareTo(obj1);
                }
            });
            //卖家订单总金额
            Map<Long,BigDecimal> buyer_amt = new HashMap<Long,BigDecimal>();
            //备货完成统计
            Map<Long,Integer> a = new HashMap<Long,Integer>();
            //缺货统计
            Map<Long,Integer> b = new HashMap<Long,Integer>();
            //订单明细数量
            Map<Long,Integer> c = new HashMap<Long,Integer>();
            if (null != orderItemList){
                for(PickUpItemVo oiv : orderItemList){
                    Long orderId = oiv.getOrderId();
                    //配送状态
                    int stockStatus = 0;
                    //订单明细配送状态
                    int status = oiv.getSellerStatus();
                    //备货完成统计
                    int aa = a.get(orderId) == null ? 0 :a.get(orderId);
                    //缺货统计
                    int bb = b.get(orderId) == null ? 0 :b.get(orderId);
                    //订单明细数量
                    int cc = c.get(orderId) == null ? 0 :c.get(orderId);
                    BigDecimal buyer_totalAmt = buyer_amt.get(orderId) == null ? BigDecimal.ZERO :buyer_amt.get(orderId);
                    if(buyerMap.containsKey(orderId)){
                        if(status == 1){
                            aa += status;
                        }
                        if(status == 2){
                            bb += status;
                        }
                        cc++;
                        a.put(orderId, aa);
                        b.put(orderId, bb);
                        c.put(orderId, cc);
                        //备货完成
                        if(aa == cc){
                            stockStatus = 1;
                        }
                        //部分备货
                        if(aa>0 && aa!=cc){
                            stockStatus = 2;
                        }
                        //缺货
                        if(aa == 0 && bb>0){
                            stockStatus = 3;
                        }
                        buyer_totalAmt = oiv.getGoodsAmount().add(buyer_totalAmt);
                        buyer_amt.put(orderId, buyer_totalAmt);
                        //跟买卖家总金额
                        buyerMap.get(orderId).setTotalAmt(buyer_totalAmt);
                        buyerMap.get(orderId).setStockStatus(stockStatus);
                    }else{
                        c.put(orderId, 1);
                        if(status == 1){
                            a.put(orderId, 1);
                            stockStatus = 1;
                        }
                        if(status == 2){
                            b.put(orderId, 2);
                            stockStatus = 3;
                        }
                        //创建买家信息
                        BuyerVo bv = new BuyerVo();
                        bv.setOrderId(oiv.getOrderId());
                        bv.setOrderNumber(oiv.getOrderNumber());
                        bv.setBuyerId(oiv.getBuyerId());
                        bv.setBuyerName(oiv.getBuyerName());
                        bv.setBuyerMobile(oiv.getBuyerMobile());
                        bv.setBuyerAddress(oiv.getBuyerAddress());
                        bv.setDeliveryStatus(oiv.getDeliveryStatus());
                        bv.setDeliveryTime(oiv.getBestTime());
                        bv.setTotalAmt(oiv.getGoodsAmount());
                        bv.setSellerStatus(oiv.getSellerStatus());
                        bv.setPayStatus(oiv.getPayStatus());
                        bv.setDeliveryAreaId(oiv.getDeliveryAreaId());
                        bv.setDaAddress(oiv.getDaAddress());
                        bv.setStockStatus(stockStatus);
                        bv.setTheAmt(oiv.getTheAmt());
                        bv.setThePerson(oiv.getThePerson());
                        bv.setSaleId(oiv.getSaleId());
                        bv.setSaleName(oiv.getSaleName());
                        bv.setSalePhone(oiv.getSalePhone());
                        
                        buyer_totalAmt = bv.getTotalAmt();
                        buyer_amt.put(orderId, buyer_totalAmt);
                        buyerMap.put(orderId, bv);
                        buyerList.add(bv);
                    }
                }
                List<BuyerVo> bvList = null;
                for(BuyerVo bv:buyerList){
                    Long areaId = 0L;
                    if(bv.getDeliveryAreaId()!=null){
                        areaId = bv.getDeliveryAreaId();
                    }
                    if(pickUpMap.containsKey(areaId)){
                        pickUpMap.get(areaId).add(bv);
                    }else{
                        PickUpListVo pv = new PickUpListVo();
                        pv.setDeliveryAreaId(areaId);
                        pv.setDaAddress(bv.getDaAddress());
                        
                        bvList = new ArrayList<BuyerVo>();
                        bvList.add(bv);
                        pickUpMap.put(areaId, bvList);
                        pv.setBuyerList(bvList);
                        pickUpList.add(pv);
                    }
                }
            }
            
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", pickUpList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][pickUplistBySeller] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 送达列表  deliveryStatus=1未送达 2已送达
     */
    @RequestMapping(value = "/order/sellerListByBuyer", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult sellerListByBuyer(HttpServletRequest request, HttpServletResponse response,Long deliveryId,Long orderId,Short deliveryStatus) {
        try {
            //查询订单明细
            List<PickUpItemVo> orderItemList = orderV1Service.getPickUpItemList(orderId,null,deliveryId,(short)1,deliveryStatus,(short)0);
            //返回结果
            List<SellerVo> sellerListList = new ArrayList<SellerVo>();
            //按买家分组
            Map<Long,SellerVo> sellerMap = new TreeMap<Long,SellerVo>(new Comparator<Long>(){
                public int compare(Long obj1, Long obj2) {
                    // 降序排序
                    return obj2.compareTo(obj1);
                }
            });
            //备货完成统计
            Map<Long,Integer> a = new HashMap<Long,Integer>();
            //缺货统计
            Map<Long,Integer> b = new HashMap<Long,Integer>();
            //订单明细数量
            Map<Long,Integer> c = new HashMap<Long,Integer>();
            //卖家订单总金额
            Map<Long,BigDecimal> seller_amt = new HashMap<Long,BigDecimal>();
            if (null != orderItemList){
                for(PickUpItemVo oiv : orderItemList){
                    Long sellerId = oiv.getSellerId();
                    //配送状态
                    int stockStatus = 0;
                    //订单明细配送状态
                    int status = oiv.getSellerStatus();
                    //备货完成统计
                    int aa = a.get(sellerId) == null ? 0 :a.get(sellerId);
                    //缺货统计
                    int bb = b.get(sellerId) == null ? 0 :b.get(sellerId);
                    //订单明细数量
                    int cc = c.get(sellerId) == null ? 0 :c.get(sellerId);
                    BigDecimal seller_totalAmt = seller_amt.get(sellerId) == null ? BigDecimal.ZERO :seller_amt.get(sellerId);
                    if(sellerMap.containsKey(sellerId)){
                        if(status == 1){
                            aa += status;
                        }
                        if(status == 2){
                            bb += status;
                        }
                        cc++;
                        a.put(sellerId, aa);
                        b.put(sellerId, bb);
                        c.put(sellerId, cc);
                        //备货完成
                        if(aa == cc){
                            stockStatus = 1;
                        }
                        //部分备货
                        if(aa>0 && aa!=cc){
                            stockStatus = 2;
                        }
                        //缺货
                        if(aa == 0 && bb>0){
                            stockStatus = 3;
                        }
                        seller_totalAmt = oiv.getGoodsAmount().add(seller_totalAmt);
                        seller_amt.put(sellerId, seller_totalAmt);
                        //将买家信息加入到买家列表中
                        sellerMap.get(sellerId).setTotalAmt(seller_totalAmt);
                        sellerMap.get(sellerId).setStockStatus(stockStatus);
                    }else{
                        c.put(sellerId, 1);
                        if(status == 1){
                            a.put(orderId, 1);
                            stockStatus = 1;
                        }
                        if(status == 2){
                            b.put(orderId, 2);
                            stockStatus = 3;
                        }
                        
                        //创建卖家信息
                        SellerVo seller = new SellerVo();
                        seller.setOrderId(oiv.getOrderId());
                        seller.setOrderNumber(oiv.getOrderNumber());
                        seller.setSellerId(oiv.getSellerId());
                        seller.setSellerName(oiv.getSellerName());
                        seller.setSellerAccount(oiv.getSellerAccount());
                        seller.setSellerLogo(oiv.getSellerLogo());
                        seller.setSellerRemark(oiv.getSellerRemark());
                        seller.setTrueName(oiv.getTrueName());
                        seller.setSellerAddress(oiv.getSellerAddress());
                        seller.setDeliveryStatus(oiv.getDeliveryStatus());
                        seller.setTotalAmt(oiv.getGoodsAmount());
                        seller.setSellerStatus(oiv.getSellerStatus());
                        seller.setPayStatus(oiv.getPayStatus());
                        seller.setStockStatus(stockStatus);
                        
                        seller_totalAmt = seller.getTotalAmt();
                        seller_amt.put(sellerId, seller_totalAmt);
                        //放入卖家信息到Map
                        sellerMap.put(sellerId, seller);
                        
                        sellerListList.add(seller);
                    }
                }
            }
            
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", sellerListList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][sellerListByBuyer] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 订单商品列表
     * type=1 按卖家(传sellerId) 2按买家(传buyerId) 3按明细(传orderId和sellerId)
     */
    @RequestMapping(value = "/order/v1/goodslist", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult goodsList(HttpServletRequest request, HttpServletResponse response,Short type,Short deliveryType,Long orderId,Long sellerId,Long buyerId,Long deliveryId) {
        try {
            List<OrderGoodsVo> goodsList = orderV1Service.getOrderGoodsList(type,deliveryType,orderId,sellerId, buyerId,deliveryId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", goodsList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][goodsList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }


    /**
     * 更新取货状态 
     */
    @RequestMapping(value = "/order/v1/updatePickUpStatus", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult updatePickUpStatus(HttpServletRequest request, HttpServletResponse response,Long orderId,Long sellerId) {
        try {
            int result = orderV1Service.updatePickUpStatus(orderId, sellerId);
            if(result > 0){
                return new JsonResult(JsonResultCode.SUCCESS, "更新信息成功", "");
            }else{
                return new JsonResult(JsonResultCode.FAILURE, "更新信息失败", "");
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][updatePickUpStatus] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 根据订单明细更新取货状态
     */
    @RequestMapping(value = "/order/v1/updatePickUpStatusByItemId", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult updatePickUpStatusByItemId(HttpServletRequest request, HttpServletResponse response,Long itemId) {
        try {
            int result = orderV1Service.updatePickUpStatusByItemId(itemId);
            if(result > 0){
                return new JsonResult(JsonResultCode.SUCCESS, "更新信息成功", "");
            }else{
                return new JsonResult(JsonResultCode.FAILURE, "更新信息失败", "");
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][updatePickUpStatusByItemId] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }

    /**
     * 更新送达状态
     */
    @RequestMapping(value = "/order/v1/updateDeliveryStatus", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult updateDeliveryStatus(HttpServletRequest request, HttpServletResponse response,Long orderId,Long sellerId,Long deliveryId,String size,String onlyId) {
        try 
        {
            logger.info("[配送人员][手机型号:"+size+"][手机唯一标识:"+onlyId+"][时间:"+DateUtil.dateToString(new Date(), "yyyy-MM-dd HH:mm:ss")+"]");
            Date now = new Date();
            //当前日期
            String currentDate = DateUtil.dateToString(now, DateUtil.FMT_DATE);
            String beginTime = currentDate+" 07:00:00";
            int a = DateUtil.compareDate(now, DateUtil.stringToDate(beginTime, DateUtil.FMT_DATETIME));
            if(a<0)
            {
                logger.info("[手机型号:"+size+"][手机唯一标识:"+onlyId+"][时间:"+DateUtil.dateToString(new Date(), "yyyy-MM-dd HH:mm:ss")+"]");
                return new JsonResult(JsonResultCode.FAILURE, "7点钟之后才能点击已送达！", "");
            }else{
                int result = orderV1Service.updateDeliveryStatus(orderId, sellerId,deliveryId);
                if(result > 0){
                    return new JsonResult(JsonResultCode.SUCCESS, "更新信息成功", "");
                }else{
                    return new JsonResult(JsonResultCode.FAILURE, "更新信息失败", "");
                }
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][updateDeliveryStatus] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 线下收款确认
     */
    @RequestMapping(value = "/order/v1/updatePayStatus", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult updatePayStatus(HttpServletRequest request, HttpServletResponse response,Long deliveryId,Long brId,String skReamk,BigDecimal ssAmt) {
        try {
            int result = orderV1Service.updatePayStatus(deliveryId,brId, skReamk,ssAmt);
            if(result > 0){
                return new JsonResult(JsonResultCode.SUCCESS, "更新信息成功", "");
            }else{
                return new JsonResult(JsonResultCode.FAILURE, "更新信息失败", "");
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][updatePayStatus] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 取货商家列表
     */
    @RequestMapping(value = "/order/getPickUpSellerList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getPickUpSellerList(HttpServletRequest request, HttpServletResponse response,Long deliveryId) {
        try {
            List<SellerInfoVo> list = orderV1Service.getPickUpSellerList(deliveryId, (short)1);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getPickUpSellerList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 按买家显示商品 取货界面全部商品
     */
    @RequestMapping(value = "/order/v1/getBuyerGoodsList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getBuyerGoodsList(HttpServletRequest request, HttpServletResponse response,Long sellerId,Long deliveryId,Integer type,Long categoryId,Long areaId) {
        try {
            List<BuyerGoodsVo> goodsList = orderV1Service.getBuyerGoodsList(sellerId, deliveryId,type,categoryId,areaId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", goodsList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getBuyerGoodsList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 获取卖家区域
     */
    @RequestMapping(value = "/order/v1/getSellerRegions", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getSellerRegions(HttpServletRequest request, HttpServletResponse response,String regionIds) {
        try {
            if(StringUtils.isBlank(regionIds)){
                return new JsonResult(JsonResultCode.FAILURE, "区域id为空", "");
            }
            List<Area> areaList = areaService.getAreas(regionIds);
            return new JsonResult(JsonResultCode.SUCCESS, "查询区域信息成功", areaList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getSellerRegions] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 备货清单
     */
    @RequestMapping(value = "/order/v1/getStockList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getStockList(HttpServletRequest request, HttpServletResponse response,Long sellerId,Long deliveryId,String categoryCode) {
        try {
            List<StockVo> stockList = orderV1Service.getStockList(sellerId, deliveryId,categoryCode);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", stockList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getStockList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 备货详情
     */
    @RequestMapping(value = "/order/v1/getStockInfoList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getStockInfoList(HttpServletRequest request, HttpServletResponse response,Long sellerId,Long formatId,
            Long deliveryId,Long goodsId,Long methodId,String formatIds) {
        try {
            List<StockInfoVo> stockInfoList = orderV1Service.getStockInfoList(sellerId, formatId, deliveryId,goodsId, methodId,formatIds);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", stockInfoList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getStockInfoList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 买家地址详情
     */
    @RequestMapping(value = "/order/getBuyerAddress", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getBuyerAddress(HttpServletRequest request, HttpServletResponse response,Long buyerId) {
        try {
            BuyerAddressVo bav = orderV1Service.getBuyerAddress(buyerId);
            if(null == bav){
                bav = new BuyerAddressVo();
            }
            if(null != bav.getBuyerImages() && !"".equals(bav.getBuyerImages())){
                String[] list = bav.getBuyerImages().split(",");
                bav.setPicList(Arrays.asList(list));
            }
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", bav);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getBuyerAddress] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 新增配送人员坐标
     */
    @RequestMapping(value = "/order/v1/insertDeliveryCoordinate", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult insertDeliveryCoordinate(HttpServletRequest request, HttpServletResponse response,@RequestBody DeliveryCoordinate deliveryCoordinate) {
        try {
            int result = deliveryCoordinateService.insertDeliveryCoordinate(deliveryCoordinate);
            if(result > 0){
                return new JsonResult(JsonResultCode.SUCCESS, "添加信息成功", "");
            }else{
                return new JsonResult(JsonResultCode.FAILURE, "添加信息失败", "");
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][insertDeliveryCoordinate] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 更新送达状态为未送达
     */
    @RequestMapping(value = "/order/v1/updateDeliveryStatusToIng", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult updateDeliveryStatusToIng(HttpServletRequest request, HttpServletResponse response,Long orderId,Long deliveryId,String size,String onlyId) {
        try 
        {
            logger.info("[配送人员:"+deliveryId+"][手机型号:"+size+"][手机唯一标识:"+onlyId+"][时间:"+DateUtil.dateToString(new Date(), "yyyy-MM-dd HH:mm:ss")+"]");
            int result = orderV1Service.updateDeliveryStatusToIng(orderId);
            if(result > 0){
                return new JsonResult(JsonResultCode.SUCCESS, "更新信息成功", "");
            }else{
                return new JsonResult(JsonResultCode.FAILURE, "更新信息失败", "");
            }
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][updateDeliveryStatusToIng] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 按分类取货
     */
    @RequestMapping(value = "/order/v1/getStockListByCategory", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getStockListByCategory(HttpServletRequest request, HttpServletResponse response,Long deliveryId,Integer type) {
        try {
            List<CategoryVo> cvList = orderV1Service.getStockListByCategory(deliveryId,type);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", cvList);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getStockList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 线下收款列表
     */
    @RequestMapping(value = "/order/getReceiptList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getReceiptList(HttpServletRequest request, HttpServletResponse response,Short type,Long deliveryId) {
        try {
            List<ReceiptListVo> list = orderV1Service.getReceiptList(type, deliveryId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getReceiptList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 线下收款订单
     */
    @RequestMapping(value = "/order/getReceiptOrderList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getReceiptOrderList(HttpServletRequest request, HttpServletResponse response,String orderDate,Long buyerId) {
        try {
            List<OrderVo> list = orderV1Service.getReceiptOrderList(orderDate, buyerId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getReceiptOrderList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
    /**
     * 线下收款订单商品
     */
    @RequestMapping(value = "/order/getOrderGoodsList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult getOrderGoodsList(HttpServletRequest request, HttpServletResponse response,Long orderId) {
        try {
            List<OrderGoodsVo> list = orderV1Service.getLineOrderGoodsList(orderId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", list);
        } catch (Exception ex) {
            logger.error("[OrderV1Controller][getOrderGoodsList] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
    
}

```


