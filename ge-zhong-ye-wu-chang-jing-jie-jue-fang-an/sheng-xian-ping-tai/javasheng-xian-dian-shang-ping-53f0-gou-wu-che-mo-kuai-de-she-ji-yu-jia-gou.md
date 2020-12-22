# Java生鲜电商平台-购物车模块的设计与架构

**说明：任何一个电商无论是B2C还是B2B都有一个购物车模块，其中最重要的原因就是客户需要的东西放在一起，形成一个购物清单，确认是否有问题，然后再进行下单与付款.**

## 1.购物车数据库设计：
641237-20180515085141598-1031102337.png

说明：业务需求：

* 1.购物车里面应该存放，那个买家，买了那个菜品的什么规格，有多少数量，然后这个菜品的加工方式如何。（如果存在加工方式的话，就会在这里显示处理。）

* 2.买家存在购物起送价。也就是用户放入购物车的商品的总价格如果低于配置的起送价，那么这个提交按钮就是灰色的。（不可能你点一个洋葱我们就送过去，成本太高。）

系统设计：
* 1.购物车在买家APP上进行，这个时候是不需要跟后端API交互的，因为体验很差劲，用户在APP页面中不停的点击加菜以及菜的数量，如果根据后端进行交互，哪怕是每次请求是100ms，页面会存在很严重的抖动行为，速度快的话会出现卡顿，这个是不行的。

* 2.在用户确定完成后，确认下单的时候，提交购物车，让后端可以存储用户购物车的数据。（确认下单过程）

* 3.在用户去付款的时候，也就是提交了订单。这个时候再清理购物车。

相关后端代码如下：



```
/**
 * 购物车
 */
@RestController
@RequestMapping("/buyer/goodsCart")
public class GoodsCartController extends BaseController{
    
private static final Logger logger = LoggerFactory.getLogger(GoodsCartController.class);
    
    @Autowired
    private GoodsCartService goodsCartService;
    
    @Autowired
    private GoodsFormatService goodsFormatService;

    
    /**
     * 生成购物车;
     * 1:先删除历史购物车;
     * 2:新增购物车数据;
     */
    @RequestMapping(value="/commitCart",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult commitCar(HttpServletRequest request, HttpServletResponse response,@RequestBody GoodsListVo goodsListVo) 
    {
        try
        {
            List<GoodsCart> list = goodsListVo.getList();
            
            if(null == goodsListVo.getUserId() || null == list){
                
                return new JsonResult(JsonResultCode.FAILURE, "请求参数异常", "");
            }
            
            for (int i = 0; i < list.size(); i++) {
                
                if(null ==list.get(i).getBuyerId()  || null == list.get(i).getFormatId()){
                    
                    return new JsonResult(JsonResultCode.FAILURE, "请求参数异常", "");
                }
            }
            
            int result = goodsCartService.commitCart(list,goodsListVo.getUserId());
            
            //更新购物车为空    则直接返回;
            if(list.size()<1){
                
                return new JsonResult(JsonResultCode.SUCCESS, "更新成功", new ArrayList<>());
            }
            
            Long buyerId = list.get(0).getBuyerId();
            
            List<GoodsCartVo> goodsCartBuyers = goodsCartService.getGoodsCartBuyerId(buyerId);
            
            if (result == list.size()) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "新增成功", goodsCartBuyers);
            } 
                return new JsonResult(JsonResultCode.SUCCESS, "有下架商品", goodsCartBuyers);
                
        }catch(Exception e){
            
            logger.error("[GoodsCartController][commitCart]",e);
            
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    /**
     * 对单个数据进行修改 新增 删除;
     * @param request
     * @param response
     * @param goodsCart
     * @return
     */
    @RequestMapping(value="/updateCart",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult updateCart(HttpServletRequest request, HttpServletResponse response,@RequestBody GoodsCart goodsCart) 
    {
        try
        {
            
            BigDecimal goodsNumber=goodsCart.getGoodsNumber();
            
            Long formartId=goodsCart.getFormatId();
            
            BigDecimal count = goodsFormatService.getGoodsFormatById(formartId).getFormatPrice().multiply(goodsNumber);
            
            goodsCart.setCreateTime(new Date());
            
            int result = goodsCartService.updateGoodsCart(goodsCart);
            
            if (result > 0) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "操作成功", count);
            }
                return new JsonResult(JsonResultCode.FAILURE, "操作失败", count);
        }catch(Exception e){
            
            logger.error("[GoodsCartController][updateCart]",e);
            
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    
    /**
     * 
     * @param request
     * @param response
     * @param goodsCart
     * @return
     */
    @RequestMapping(value="/insertCart",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult insertCart(HttpServletRequest request, HttpServletResponse response,@RequestBody GoodsCart goodsCart) 
    {
        try
        {
            BigDecimal goodsNumber=goodsCart.getGoodsNumber();
            
            Long formartId=goodsCart.getFormatId();
            
            BigDecimal count = goodsFormatService.getGoodsFormatById(formartId).getFormatPrice().multiply(goodsNumber);
            
            goodsCart.setCreateTime(new Date());
            
            int result = goodsCartService.insertGoodsCart(goodsCart);
            
            if (result > 0) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "操作成功", count);
            } 
                return new JsonResult(JsonResultCode.FAILURE, "操作失败", count);
        }catch(Exception e){
            
            logger.error("[GoodsCartController][insertCart]",e);
            
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    
    /**
     * 根据cartId 删除单个购物车项;
     * @param request
     * @param response
     * @param cartId
     * @return
     */
    @RequestMapping(value="/deleteCart",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult deleteCar(HttpServletRequest request, HttpServletResponse response, Long cartId) 
    {
        
        try
        {
            if(null == cartId){
                
                return new JsonResult(JsonResultCode.FAILURE, "请求参数异常", "");
            }
            
            int result = goodsCartService.deleteGoodsCart(cartId);
            
            if (result > 0) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "删除成功", "");
            } else 
            {
                return new JsonResult(JsonResultCode.FAILURE, "数据已不存在", "");
                
            }
        }catch(Exception e){
            
            logger.error("[GoodsCartController][deleteCart]",e);
            
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    @RequestMapping(value="/showCart",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult showCart(HttpServletRequest request, HttpServletResponse response,@RequestBody GoodsListVo goodsListVo) {
        try
        {
            List<GoodsCart> list = goodsListVo.getList();
            Long buyerId = goodsListVo.getUserId();
            Long sellerId = goodsListVo.getSellerId();
                if(null == buyerId ||null == sellerId){
                
                return new JsonResult(JsonResultCode.FAILURE, "请求参数异常", "");
                }
                goodsCartService.commitCartByBuyerIdSellerId(list,buyerId, sellerId);
                
            List<ProductVo> productVos = goodsCartService.getGoodsCartListBySellerId(buyerId, sellerId);
            
            return new JsonResult(JsonResultCode.SUCCESS, "查询成功", productVos);
        }catch(Exception e){
            logger.error("[GoodsCartController][showCart]",e);
            
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
        }
}
```

基本业务功能:

1.购物车可以清空。

2. 购物车可以提交。

3. 购物车可以更新。

4. 购物车也可以查询。



## 2.