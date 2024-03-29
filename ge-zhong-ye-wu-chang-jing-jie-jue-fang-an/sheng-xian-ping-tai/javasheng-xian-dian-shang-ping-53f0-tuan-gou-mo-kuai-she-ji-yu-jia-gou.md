# Java生鲜电商平台-团购模块设计与架构
 
说明：任何一个电商系统中，对于促销这块是必不可少的，毕竟这块是最吸引用户的，用户也是最爱的模块之一，理由很简单，便宜。

我的经验是无论是大的餐饮点还是小的餐饮店，优惠与折扣永远是说服他们进入平台的最好的手段之一。（大企业叫做节约成本，小企业叫做贪便宜.）

**1.Java开源生鲜电商平台中，团购模块，我们采用以下几种维度思考。**

**1.1.针对的是生鲜中的标品。（米面粮油，我们要求买家可以自己发送团购，但是团购有次数，与时间以及买家起团金额和最低开团金额几个维度）**

**因此，需要有一个团购基础信息表：**

```
CREATE TABLE `groups` (
  `group_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `group_no` varchar(32) DEFAULT NULL COMMENT '团号',
  `group_title` varchar(128) DEFAULT NULL COMMENT '团购标题',
  `group_logo` varchar(128) DEFAULT NULL COMMENT '团购logo',
  `group_area` varchar(128) DEFAULT NULL COMMENT '团购区域(区域ID集合)',
  `begin_time` datetime DEFAULT NULL COMMENT '开始时间',
  `end_time` datetime DEFAULT NULL COMMENT '结束时间',
  `max_num` int(11) DEFAULT NULL COMMENT '最大买家数',
  `buyer_amt` decimal(12,2) DEFAULT NULL COMMENT '买家起团金额',
  `min_amt` decimal(12,2) DEFAULT NULL COMMENT '最低开团金额',
  `group_status` tinyint(4) DEFAULT NULL COMMENT '状态(1发布 -1未发布 2团成 3未团成)',
  `remarks` varchar(256) DEFAULT NULL,
  `create_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`group_id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8 COMMENT='团购主表';
```

说明：这里面有一个团购的状态需要指明下，买家用户选择好几样商品后发起了团购，然后默认状态为-1，表示不可用，等组成了团购，最终状态会有团成的状态。


2.对于团购而言，系统肯定需要记录，那些买家参与了那些团购，因此有以下的一张表记录

```
CREATE TABLE `groups_buyer` (
  `gb_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `group_id` bigint(20) DEFAULT NULL COMMENT '团购ID',
  `item_id` bigint(20) DEFAULT NULL COMMENT '团购明细ID',
  `order_id` bigint(20) DEFAULT NULL COMMENT '订单ID',
  `gb_num` int(11) DEFAULT NULL COMMENT '团购数量',
  `gb_price` decimal(12,2) DEFAULT NULL COMMENT '团购价格',
  `gb_amt` decimal(12,2) DEFAULT NULL COMMENT '团购金额',
  `gb_status` tinyint(4) DEFAULT NULL COMMENT '状态(1完成 -1取消)',
  `gb_time` datetime DEFAULT NULL COMMENT '团购时间',
  PRIMARY KEY (`gb_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='团购买家表';
```
说明：团购买家表，记录那个买家，那个团购，团购的最终数量以及团购的价格等等，最终是否有买家在规定的时间内推出了团购，或者团购未形成等等。


3.团购最终是对商品的规格进行团购。

谈谈商品的规格系数，我们知道蔬菜中有西红柿对吧，那么西红柿分为两种，一种是大红的，一种是粉红，这两种颜色都是西红柿，那么系统会认为这个是两个产品，而不是两个规格，规格到底是说的什么呢?

对于平台而言，规格就是一种商品的几种售卖方式。

最终根据业务分析，我们需要记录团购是由那些明细组成.(商品规格组成)

因此，最终系统架构如下：


```
CREATE TABLE `groups_item` (
  `item_id` bigint(20) NOT NULL AUTO_INCREMENT COMMENT '主键ID',
  `group_id` bigint(20) DEFAULT NULL COMMENT '团购ID',
  `goods_id` bigint(20) DEFAULT NULL COMMENT '商品ID',
  `format_id` bigint(20) DEFAULT NULL COMMENT '商品规格ID',
  `group_price` decimal(12,2) DEFAULT NULL COMMENT '团购价格',
  `group_num` int(11) DEFAULT NULL COMMENT '团购数量',
  `item_status` tinyint(4) DEFAULT NULL COMMENT '状态(1在用 -1停用)',
  `create_user_id` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`item_id`)
) ENGINE=InnoDB AUTO_INCREMENT=3 DEFAULT CHARSET=utf8 COMMENT='团购明细表';
```
业务总结：
1.存在 一个买家发起的团购申请记录表。

2.后端会有一个审核机制，默认1个小时内审核通过。
    
3.团购会有商品的明细组成。也有时间段的范围与有消息。
    
4.团购最终需要记录那些人参与了，然后交费完成等等。

补充说明：业务代码级别，无外乎提供给APP接口。以下几种功能：
1.团购列表。
2.团购明细。
3.我的团购。
4.我的取消团购等




```
/**
 * 团购Controller
 */
@RestController
@RequestMapping("/buyer")
public class GroupsController extends BaseController {

    private static final Logger logger = LoggerFactory.getLogger(GroupsController.class);
    @Autowired
    private GroupsService groupsService;
    
    /**
     * 团购活动列表
     * @param request
     * @param response
     */
    @RequestMapping(value = "/groups/list", method = { RequestMethod.GET})
    public JsonResult groupsList(HttpServletRequest request, HttpServletResponse response,Long regionId) {
        try{
            if(null == regionId || 0 == regionId){
                return new JsonResult(JsonResultCode.FAILURE, "参数错误，请检查regionId是否有传","");
            }
            List<GroupsVo> cgList = groupsService.getGroupsList(regionId);
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", cgList);
        }catch(Exception ex){
            logger.error("[GroupsController][groupsList] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    /*
     * 团购活动详情
     */
    @RequestMapping(value = "/groups/detail", method = { RequestMethod.GET })
    public JsonResult detailGroups(HttpServletRequest request, HttpServletResponse response,Long groupId) {
        try{
            if(null == groupId || 0 == groupId){
                return new JsonResult(JsonResultCode.FAILURE, "参数错误，请检查groupId是否有传","");
            }
            GroupsVo groupsVo = groupsService.getGroupsInfo(groupId);
            if(groupsVo == null){
                groupsVo = new GroupsVo();
            }
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", groupsVo);
        }catch(Exception ex){
            logger.error("[GroupsController][detailGroups] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    /**
     * 团购下单
     * @param request
     * @param response
     */
    @RequestMapping(value = "/groups/createOrder", method = { RequestMethod.POST })
    public JsonResult createOrder(HttpServletRequest request, HttpServletResponse response,
            @RequestBody GroupOrder groupOrder) {
        try {
            String time = groupOrder.getBestTime();
            if (StringUtils.isBlank(time)) {
                return new JsonResult(JsonResultCode.FAILURE, "订单创建失败,收货时间不允许为空", "");
            }
            
            OrderInfo addOrderInfo = groupsService.addOrderInfo(groupOrder);
            if (addOrderInfo == null) {
                return new JsonResult(JsonResultCode.FAILURE, "创建订单失败，订单金额小于起送价", "");
            }
            return new JsonResult(JsonResultCode.SUCCESS, "创建订单成功", addOrderInfo);
        } catch (Exception ex) {
            logger.error("[GroupsController][createOrder] exception :", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
}
```
总结：目前Java开源生鲜电商平台-团购模块设计与架构只是针对的是很普通的一些团购手段，当然对于拼多多而言，差距还是很大的。

这个也是跟业务形态有关，非技术有关，每一种促销方案并不是适合左右的买家用户或者说系统平台本身的。

由于时间关系或者说有关规定， APP运营截图相对而言比较简单，我这边就不贴出来了。
