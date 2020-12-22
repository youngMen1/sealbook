# Java生鲜电商平台-支付模块的设计与架构

开源生鲜电商平台支付目前支持支付宝与微信。针对的是APP端（android or IOS）

## 1.数据库表设计
641237-20180514083820100-1888760839.png

说明：无论是支付宝还是微信支付，都会有一个服务端的回调，业务根据回调的结果处理相应的业务逻辑。

pay_logs这个表主要是记录相关的用户支付信息。是一个日志记录。

比如：谁付款的，什么时候付款的，订单号多少，是支付宝还是微信，支付状态是支付成功还是支付失败，还是未支付。

**特别注意：订单主表也有类似的回调信息。这样用多张表记录相应的信息，可以统计相应的业务指标，包括用户的行为分析等。
关于表的设计，我的经验分享是：如果可以，核心业务表一定要有一个日志记录表，如果可以，可以用时间轴等方式进行数据的插入，与时间轴的显示。

时间轴可以清楚的知道用户的行为点，帮助更加清晰的设计业务流程与系统架构。 **

相应的支付宝回调代码如下：（注意，这个业务模块属于买家。）
641237-20180514084410363-1647027118.png

### APP调用后端的业务代码


```
/***
 * APP端调用请求支付宝
 */
@RestController
@RequestMapping("/buyer")
public class AlipayController extends BaseController
{
    private static final Logger logger = LoggerFactory.getLogger(AlipayController.class);
    /**
     * 服务器通知地址
     */
    private static final String NOTIFY_URL="";
    
    /**待支付*/
    private static final int PAY_LOGS_READY=0;
    
    @Autowired
    private OrderInfoService orderInfoService;
    
    @Autowired
    private BuyerService buyerService;
    
    @Autowired
    private PayLogsService payLogsService;/**
     * APP端请求调用支付宝
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value="/alipay/invoke",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult alipayInvoke(HttpServletRequest req, HttpServletResponse resp)
    {
        String result="";
        try 
        {
            /**订单号*/
            String orderNumber=this.getNotNull("orderNumber", req);
                    
            /**金额*/
            String money=this.getNotNull("money", req);
            
            /**优惠券id*/
            String couponReceiveId = req.getParameter("couponReceiveId");
            
            if(StringUtils.isBlank(orderNumber) || StringUtils.isBlank(money))
            {
                return new JsonResult(JsonResultCode.FAILURE,"请求参数有误,请稍后重试","");
            }
            
            //对比金额
            OrderInfo orderInfo=this.orderInfoService.getOrderInfoByOrderNumber(orderNumber);
            if(orderInfo==null)
            {
                return new JsonResult(JsonResultCode.FAILURE,"订单号不存在，请稍后重试","");
            }
            
            //获取订单的金额
            BigDecimal orderAmount=orderInfo.getOrderAmount();
            
            //减余额
            Long buyerId=orderInfo.getBuyerId();
            
            Buyer buyer=this.buyerService.getBuyerById(buyerId);
            
            //用户余额
            BigDecimal balanceMoney=buyer.getBalanceMoney();
            
            //计算最终需要给支付宝的金额
            BigDecimal payAmount=orderAmount.subtract(balanceMoney);
            
            //买家支付时抵扣优惠券
            if(StringUtils.isNotBlank(couponReceiveId)){
                Long id = Long.parseLong(couponReceiveId);
                payAmount = couponReceiveService.deductionCouponMoney(id, orderNumber, payAmount);
            }
            
            logger.info("[AlipayController][alipayInvoke] orderNumber:" +orderNumber +" money:" +money+" orderAmount:"+orderAmount+" balanceMoney:"+balanceMoney+" payAmount:" +payAmount);
            
            //考虑重复订单的问题，会产生重复日志
            PayLogs payLogs=this.payLogsService.getPayLogsByOrderNumber(orderNumber);
            
            if(payLogs==null)
            {
                //创建订单日志
                PayLogs logs=new PayLogs();
                logs.setUserId(buyerId);
                logs.setOrderId(orderInfo.getOrderId());
                logs.setOrderNumber(orderNumber);
                logs.setOrderAmount(payAmount);
                logs.setStatus(PAY_LOGS_READY);
                logs.setCreateTime(new Date());
                int payLogsResult=payLogsService.addPayLogs(logs);
                logger.info("[AlipayController][alipayInvoke] 创建订单日志结果:" + (payLogsResult>0));
            }else
            {
                logger.info("[AlipayController][alipayInvoke] 创建重复订单");
            }
            
        
            AlipayClient alipayClient = new DefaultAlipayClient("https://openapi.alipay.com/gateway.do","","", "json","UTF-8","","RSA2");
            AlipayTradeAppPayRequest request = new AlipayTradeAppPayRequest();
            AlipayTradeAppPayModel model = new AlipayTradeAppPayModel();
            model.setBody("");
            model.setSubject("xxx");
            model.setOutTradeNo(orderNumber);
            model.setTimeoutExpress("15m");
            model.setTotalAmount(payAmount.toString());
            model.setProductCode("QUICK_MSECURITY_PAY");
            request.setBizModel(model);
            request.setNotifyUrl(NOTIFY_URL);
            
            AlipayTradeAppPayResponse response = alipayClient.sdkExecute(request);
            result=response.getBody();
            logger.info("[AlipayController][alipayNotify] result: " +result);
        } catch (AlipayApiException ex) 
        {
            logger.error("[AlipayController][alipayNotify] exception:",ex);
            return new JsonResult(JsonResultCode.FAILURE,"系统错误,请稍后重试","");
        }
        return new JsonResult(JsonResultCode.SUCCESS,"操作成功",result);
    } 
}
```

