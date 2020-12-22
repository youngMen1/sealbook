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

### 支付宝服务端回调代码


```
import java.io.IOException;
import java.math.BigDecimal;
import java.util.Date;
import java.util.Enumeration;
import java.util.HashMap;
import java.util.Iterator;
import java.util.Map;

import javax.servlet.ServletOutputStream;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;

import org.apache.commons.lang3.StringUtils;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RequestMethod;

import com.alipay.api.AlipayApiException;
import com.alipay.api.internal.util.AlipaySignature;
/**
 * alipay 支付宝服务端回调
 * 参考文档：https://docs.open.alipay.com/204/105301/
 * 对于App支付产生的交易，支付宝会根据原始支付API中传入的异步通知地址notify_url，通过POST请求的形式将支付结果作为参数通知到商户系统。
 */
@Controller
@RequestMapping("/buyer")
public class AlipayNotifyController extends BaseController {

    private static final Logger logger = LoggerFactory.getLogger(AlipayNotifyController.class);

    private static final String ALIPAYPUBLICKEY = "";

    private static final String CHARSET = "UTF-8";

    /**支付成功*/
    private static final int PAY_LOGS_SUCCESS=1;
    
    @Autowired
    private OrderInfoService orderInfoService;
   
    
    @RequestMapping(value = "/alipay/notify", method = { RequestMethod.GET, RequestMethod.POST })
    public void alipayNotify(HttpServletRequest request, HttpServletResponse response) throws IOException 
    {
        String url=request.getRequestURL().toString();
        
        logger.info("AlipayNotifyController.alipayNotify.s+tart");
        
        logger.info("[AlipayNotifyController][alipayNotify] url:" +url);
        
        ServletOutputStream out = response.getOutputStream();
        //支付宝交易号
        String alipayTradeNo=this.getNotNull("trade_no", request);
        
        //商户订单号
        String outerTradeNo=this.getNotNull("out_trade_no", request);
        
        //买家支付宝用户号
        String buyerId=this.getNotNull("buyer_id", request);
        
        //买家支付宝账号
        String buyerLogonId=this.getNotNull("buyer_logon_id", request);
        
        //交易状态
        String tradeStatus=this.getNotNull("trade_status", request);
        
        //订单金额,精确到小数点后2位
        String money=getNotNull("total_amount", request);
        
        logger.info("[AlipayNotifyController][alipayNotify] tradeStatus:" +tradeStatus+" money:"+money);
        
        StringBuffer buf = new StringBuffer();
        if (request.getMethod().equalsIgnoreCase("POST"))
        {
            Enumeration<String> em = request.getParameterNames();
            for (; em.hasMoreElements();)
            {
                Object o = em.nextElement();
                buf.append(o).append("=").append(request.getParameter(o.toString())).append(",");
            }
            logger.info("回调 method:post]http://" + request.getServerName() + request.getServletPath() + " [<prams:" + buf + ">]");
        } else
        {
            buf.append(request.getQueryString());
            logger.info("回调 method:get]http://" + request.getServerName() + request.getServletPath() + "?" + request.getQueryString());
        }
        
        //检验支付宝参数
        if(!verifyAlipay(request))
        {
               out.print("fail");
               return;
        }
        
        //交易成功
        if("TRADE_SUCCESS".equalsIgnoreCase(tradeStatus))
        {
            try
            {
                if(StringUtils.isNotBlank(outerTradeNo))
                {
                    //查询当前订单信息
                    OrderInfo info=this.orderInfoService.getOrderInfoByOrderNumber(outerTradeNo);
                    
                    if(info==null)
                    {
                        logger.error("[AlipayNotifyController][alipayNotify] info:" +info);
                        out.print("fail");
                           return;
                    }
                    
                    //订单信息
                    OrderInfo orderInfo=new OrderInfo();
                    orderInfo.setOrderNumber(outerTradeNo);    
                    orderInfo.setCardCode("alipay");
                    orderInfo.setCardName("支付宝");
                    orderInfo.setCardNumber(buyerLogonId);
                    orderInfo.setCardOwner(buyerId);
                    orderInfo.setPayTime(new Date());
                    orderInfo.setOuterTradeNo(alipayTradeNo);
                    orderInfo.setPayStatus(PayStatus.PAY_SUCCESS);
                    orderInfo.setTradeStatus(TradeStatus.TRADE_PROGRESS);
                    orderInfo.setPayAmount(new BigDecimal(money));
                    orderInfo.setBuyerId(info.getBuyerId());
                    
                    //付款日志
                    PayLogs payLogs=new PayLogs();
                    payLogs.setOrderNumber(outerTradeNo);
                    payLogs.setOuterTradeNo(alipayTradeNo);
                    payLogs.setStatus(PAY_LOGS_SUCCESS);
                    
                    orderInfoService.payReturn(orderInfo,payLogs);
                    out.print("success");
                    return;
                }
            }catch(Exception ex)
            {
                logger.error("[AlipayNotifyController][payReturn] 出现了异常:",ex);
                out.print("success");
                return;
            }
        }else
        {
            out.print("fail");
            return;
        }
    }
    /**
     * 检验支付宝
     * @param request
     * @return
     */
    @SuppressWarnings("rawtypes")
    public boolean verifyAlipay(HttpServletRequest request)
    {
        boolean flag=true;
        
        // 获取支付宝POST过来反馈信息
        Map<String, String> params = new HashMap<String, String>();
        Map requestParams = request.getParameterMap();
        for (Iterator iter = requestParams.keySet().iterator(); iter.hasNext();) {
            String name = (String) iter.next();
            String[] values = (String[]) requestParams.get(name);
            String valueStr = "";
            for (int i = 0; i < values.length; i++) {
                valueStr = (i == values.length - 1) ? valueStr + values[i] : valueStr + values[i] + ",";
            }
            params.put(name, valueStr);
        }
        try
        {
            flag= AlipaySignature.rsaCheckV1(params, ALIPAYPUBLICKEY, CHARSET, "RSA2");
        } catch (AlipayApiException e) {
            e.printStackTrace();
            flag=false;
        }
        return flag;
    }
}
```

APP支付成功后，其实支付宝或者微信都会告诉你是否成功，只是这种通知是不可靠的，最可靠的的一种方式是支付宝或者微信的服务端回调。

根据回调处理相关的业务。

相应的微信回调代码如下：（注意，这个业务模块属于买家。）
641237-20180514084952365-1481603002.png

### APP调用微信：


```
/***
 * APP端调用请求微信
 */
@RestController
@RequestMapping("/buyer")
public class WeiXinController extends BaseController
{
    private static final Logger logger = LoggerFactory.getLogger(WeiXinController.class);
    
    /**待支付*/
    private static final int PAY_LOGS_READY=0;
    
    @Autowired
    private OrderInfoService orderInfoService;
    
    @Autowired
    private BuyerService buyerService;
    
    @Autowired
    private PayLogsService payLogsService;
    
    @Autowired
    private CouponReceiveService couponReceiveService;
    
    /**
     * APP端请求调用微信
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value="/weixin/invoke",method={RequestMethod.GET,RequestMethod.POST})
    public JsonResult weixinInvoke(HttpServletRequest req, HttpServletResponse resp)
    {
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
            
            //计算最终需要给微信的金额
            BigDecimal payAmount=orderAmount.subtract(balanceMoney);
            
            //买家支付时抵扣优惠券
            if(StringUtils.isNotBlank(couponReceiveId)){
                Long id = Long.parseLong(couponReceiveId);
                payAmount = couponReceiveService.deductionCouponMoney(id, orderNumber, payAmount);
            }
            
            logger.info("[WeiXinController][weixinInvoke] orderNumber:" +orderNumber +" money:" +money+" orderAmount:"+orderAmount+" balanceMoney:"+balanceMoney+" payAmount:" +payAmount);
            
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
                logger.info("[WeiXinController][weixinInvoke] 创建订单日志结果:" + (payLogsResult>0));
            }else
            {
                logger.info("[WeiXinController][weixinInvoke] 创建重复订单");
            }
            
            //微信开始
            SortedMap<Object,Object> paramMap = new TreeMap<Object,Object>();  
            paramMap.put("appid", WeiXinUtil.APPID);
            paramMap.put("mch_id", WeiXinUtil.MCHID);
            
            String nonceStr=RandomUtil.generateString(8);
            // 随机字符串    
            paramMap.put("nonce_str", nonceStr);
            paramMap.put("body","魔笛食材");// 商品描述  
            paramMap.put("out_trade_no", orderNumber);// 商户订单编号  
            paramMap.put("total_fee",Math.round(payAmount.doubleValue()*100));
            //IP地址
            String ip=IpUtils.getIpAddr(req);
            paramMap.put("spbill_create_ip",ip);
            paramMap.put("notify_url", WeiXinUtil.NOTIFYURL);// 回调地址  
            paramMap.put("trade_type",WeiXinUtil.TRADETYPE);// 交易类型APP  
            String sign=createSign(paramMap);
            paramMap.put("sign", sign);// 数字签证  

            logger.info("weixin支付请求IP:" +ip+ " sign:" +sign);
            
            String xml = getRequestXML(paramMap);   
            
            String content = HttpClientUtil.getInstance().sendHttpPost(WeiXinUtil.URL,xml,"UTF-8");
            
            logger.info("weixin支付请求的内容content:" +content);
            
            JSONObject jsonObject = JSONObject.fromObject(XmltoJsonUtil.xml2JSON(content));
            
            JSONObject resultXml = jsonObject.getJSONObject("xml");
            
            //返回的编码
            JSONArray returnCodeArray =resultXml.getJSONArray("return_code");
            
            //返回的消息
            JSONArray returnMsgArray =resultXml.getJSONArray("return_msg");
            
            //结果编码
            JSONArray resultCodeArray =resultXml.getJSONArray("result_code");
            
            String returnCode= (String)returnCodeArray.get(0);
            
            String returnMsg= (String)returnMsgArray.get(0);
            
            String resultCode = (String)resultCodeArray.get(0);
            
            logger.info("[WeiXinController][weixinInvoke] returnCode: " +returnCode+" returnMsg:"+returnMsg +" resultCode:"+resultCode);
            
            if(resultCode.equalsIgnoreCase("FAIL"))
            {  
                return new JsonResult(JsonResultCode.FAILURE,"微信统一订单下单失败","");
            }
            
            if(resultCode.equalsIgnoreCase("SUCCESS"))
            {
                JSONArray prepayIdArray =resultXml.getJSONArray("prepay_id");
                
                String prepayId= (String)prepayIdArray.get(0);
                
                WeiXinBean weixin=new WeiXinBean();
                weixin.setAppid(WeiXinUtil.APPID);
                weixin.setPartnerid(WeiXinUtil.MCHID);
                weixin.setNoncestr(nonceStr);
                weixin.setPrepayid(prepayId);
                String timestamp=System.currentTimeMillis()/1000+"";
                weixin.setTimestamp(timestamp);
                //最终返回签名
                SortedMap<Object,Object> apiMap = new TreeMap<Object,Object>();  
                apiMap.put("appid", WeiXinUtil.APPID);
                apiMap.put("partnerid", WeiXinUtil.MCHID);
                apiMap.put("prepayid", prepayId);
                apiMap.put("package","Sign=WXPay");
                apiMap.put("noncestr",nonceStr);
                apiMap.put("timestamp", timestamp);
                //再次签名
                weixin.setSign(createSign(apiMap));
                return new JsonResult(JsonResultCode.SUCCESS,"微信统一订单下单成功",weixin);
            }             
            return new JsonResult(JsonResultCode.FAILURE,"操作失败","");
        } catch (Exception ex) 
        {
            logger.error("[WeiXinController][weixinInvoke] exception:",ex);
            return new JsonResult(JsonResultCode.FAILURE,"系统错误,请稍后重试","");
        }
    } 
    //拼接xml 请求路径 
    @SuppressWarnings({"rawtypes"})
    private String getRequestXML(SortedMap<Object, Object> parame){  
        StringBuffer buffer = new StringBuffer();  
        buffer.append("<xml>");  
        Set set = parame.entrySet();  
        Iterator iterator = set.iterator();  
        while(iterator.hasNext()){  
            Map.Entry entry = (Map.Entry) iterator.next();  
            String key =String.valueOf(entry.getKey());  
            String value = String.valueOf(entry.getValue());
            //过滤相关字段sign  
            if("sign".equalsIgnoreCase(key)){  
                buffer.append("<"+key+">"+"<![CDATA["+value+"]]>"+"</"+key+">");  
            }else{  
                buffer.append("<"+key+">"+value+"</"+key+">");  
            }             
        }  
        buffer.append("</xml>");  
        return buffer.toString();  
    }
    
    //创建md5 数字签证  
    @SuppressWarnings({"rawtypes"})
    private String createSign(SortedMap<Object, Object> param){ 
         StringBuffer buffer = new StringBuffer();  
            Set set = param.entrySet();  
            Iterator<?> iterator = set.iterator();  
            while(iterator.hasNext()){  
                Map.Entry entry = (Map.Entry) iterator.next();  
                String key = String.valueOf(entry.getKey());  
                Object value =String.valueOf(entry.getValue());  
                if(null != value && !"".equals(value) && !"sign".equals(key) && !"key".equals(key)){  
                    buffer.append(key+"="+value+"&");  
                }             
            }  
            buffer.append("key="+WeiXinUtil.APIKEY);  
            String sign =EncryptUtil.getMD5(buffer.toString()).toUpperCase();   
            return sign;
    }
}
```


### 微信回调接口：



```
/**
 * weixin 微信服务端回调
 */
@Controller
@RequestMapping("/buyer")
public class WeiXinNotifyController extends BaseController {

    private static final Logger logger = LoggerFactory.getLogger(WeiXinNotifyController.class);

    /**支付成功*/
    private static final int PAY_LOGS_SUCCESS=1;
    
    @Autowired
    private OrderInfoService orderInfoService;
    
    @Autowired
    private CouponReceiveService couponReceiveService;
    
    @Autowired
    private GroupsService groupsService;
    
    @RequestMapping(value = "/weixin/notify", method = { RequestMethod.GET, RequestMethod.POST })
    public void weixinNotify(HttpServletRequest request, HttpServletResponse response) throws IOException 
    {
        String url=request.getRequestURL().toString();
        logger.info("WeiXinNotifyController.weixinNotify.start-->url:" +url);
        try
        {
            StringBuffer buf = new StringBuffer();
            if (request.getMethod().equalsIgnoreCase("POST"))
            {
                Enumeration<String> em = request.getParameterNames();
                for (; em.hasMoreElements();)
                {
                    Object o = em.nextElement();
                    buf.append(o).append("=").append(request.getParameter(o.toString())).append(",");
                }
                logger.info("回调 method:post]http://" + request.getServerName() + request.getServletPath() + " [<prams:" + buf + ">]");
            } else
            {
                buf.append(request.getQueryString());
                logger.info("回调 method:get]http://" + request.getServerName() + request.getServletPath() + "?" + request.getQueryString());
            }
            
            request.setCharacterEncoding("UTF-8");    
            response.setCharacterEncoding("UTF-8");    
            response.setContentType("text/html;charset=UTF-8");    
            response.setHeader("Access-Control-Allow-Origin", "*");     
            InputStream in=request.getInputStream();    
            ByteArrayOutputStream out=new ByteArrayOutputStream();    
            byte[] buffer =new byte[1024];    
            int len=0;    
            while((len=in.read(buffer))!=-1){    
                out.write(buffer, 0, len);    
            }    
            out.close();    
            in.close();    
            String content=new String(out.toByteArray(),"utf-8");//xml数据    
            
            logger.info("[WeiXinNotifyController][weixinNotify] content:" +content);
            
            //日志显示，存在为空的情况.
            if(StringUtils.isBlank(content))
            {
                logger.error("[WeiXinNotifyController][weixinNotify] content is blank,please check it");
                response.getWriter().write(setXml("FAIL", "ERROR"));
                return; 
            }
            
            JSONObject jsonObject = JSONObject.fromObject(XmltoJsonUtil.xml2JSON(content));
            
            JSONObject resultXml = jsonObject.getJSONObject("xml");
            JSONArray returnCode =  resultXml.getJSONArray("return_code");  
            String code = (String)returnCode.get(0);
            
            if(code.equalsIgnoreCase("FAIL"))
            {  
                response.getWriter().write(setXml("SUCCESS", "OK"));
                return;
            }else if(code.equalsIgnoreCase("SUCCESS"))
            {  
                //商户订单号即订单编号
                String outerTradeNo =String.valueOf(resultXml.getJSONArray("out_trade_no").get(0));
                
                //微信交易订单号
                String tradeNo = String.valueOf(resultXml.getJSONArray("transaction_id").get(0));
                
                //交易状态
                String resultCode = String.valueOf(resultXml.getJSONArray("result_code").get(0));
                
                //金额
                String money =String.valueOf(resultXml.getJSONArray("total_fee").get(0));
                
                //微信的用户ID
                String openId =String.valueOf(resultXml.getJSONArray("openid").get(0));
                
                logger.info("[WeiXinNotifyController][weixinNotify] resultCode:" +resultCode+" money:"+money);
                
                //根据这个判断来获取订单交易是否OK
                if(!resultCode.equalsIgnoreCase("SUCCESS"))
                {
                    response.getWriter().write(setXml("FAIL", "ERROR"));
                    return;
                }
                
                try
                {
                    if(StringUtils.isNotBlank(outerTradeNo))
                    {
                        //查询当前订单信息
                        OrderInfo info=this.orderInfoService.getOrderInfoByOrderNumber(outerTradeNo);
                        
                        if(info==null)
                        {
                            logger.error("[WeiXinNotifyController][weixinNotify] info:" +info);
                            response.getWriter().write(setXml("FAIL", "ERROR"));
                            return;
                        }
                        
                        //订单信息
                        OrderInfo orderInfo=new OrderInfo();
                        orderInfo.setOrderNumber(outerTradeNo);    
                        orderInfo.setCardCode("weixin");
                        orderInfo.setCardName("微信支付");
                        orderInfo.setCardNumber(openId);
                        orderInfo.setCardOwner(openId);
                        orderInfo.setPayTime(new Date());
                        orderInfo.setOuterTradeNo(tradeNo);
                        orderInfo.setPayStatus(PayStatus.PAY_SUCCESS);
                        orderInfo.setTradeStatus(TradeStatus.TRADE_PROGRESS);
                        orderInfo.setPayAmount(new BigDecimal(money).divide(new BigDecimal(100)));
                        orderInfo.setBuyerId(info.getBuyerId());
                        
                        //付款日志
                        PayLogs payLogs=new PayLogs();
                        payLogs.setOrderNumber(outerTradeNo);
                        payLogs.setOuterTradeNo(tradeNo);
                        payLogs.setStatus(PAY_LOGS_SUCCESS);
                        
                        orderInfoService.payReturn(orderInfo,payLogs);
                        
                        //回写团购成功状态
                        groupsService.updateGbStatusByOrderNumber(outerTradeNo);
                        response.getWriter().write(setXml("SUCCESS", "OK"));
                        try
                        {
                            //更新优惠券状态为已用
                            couponReceiveService.updateStatus(outerTradeNo);
                        }catch(Exception ex){
                            logger.error("[AlipayNotifyController][payReturn] 更新优惠券状态异常:",ex);
                        }
                        return;
                    }
                }catch(Exception ex)
                {
                    logger.error("[WeiXinNotifyController][payReturn] 出现了异常:",ex);
                    response.getWriter().write(setXml("SUCCESS", "OK"));
                    return;
                }
            }             
        }catch(Exception e){
            logger.error("[WeiXinNotifyController][weixinNotify] exception:" ,e);
            response.getWriter().write(setXml("SUCCESS", "OK"));
            return;
        }  
    }  
    //返回微信服务  
    private String setXml(String returnCode,String returnMsg)
    {    
       return "<xml><return_code><![CDATA["+returnCode+"]]></return_code><return_msg><![CDATA["+returnMsg+"]]></return_msg></xml>";    
    }   
}
```

Java开源生鲜电商平台-支付表的设计与架构(源码可下载），如果需要下载的话，可以在我的github下面进行下载。 

### 相关的运营截图如下：

641237-20180514085555049-419299049.png

641237-20180514085836966-1746087629.png




























