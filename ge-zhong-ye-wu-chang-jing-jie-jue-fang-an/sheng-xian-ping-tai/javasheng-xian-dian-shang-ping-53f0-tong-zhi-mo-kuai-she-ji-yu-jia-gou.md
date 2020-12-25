# Java生鲜电商平台-通知模块设计与架构

说明：对于一个生鲜的B2B平台而言，通知对于我们实际的运营而言来讲分为三种方式：

* 1.消息推送：（采用极光推送）

* 2.主页弹窗通知。（比如：现在有什么新的活动，有什么新的优惠等等）

* 3.短信通知.(对于短信通知，这个大家很熟悉，我们就说下我们如何从代码层面对短信进行分层的分析与架构)


## 1.消息推送
说明：目前市场上的推送很多，什么极光推送，环信，网易云等等，都可以实现秒级别的推送，我们经过了市场调研与稳定性考察，最终选择了极光推送。

极光推送，市面上有很大的文档与实例，我这边就不详细讲解了，因为文档很清晰，也的确很简单。

相关的核心功能与代码如下：

* 1.功能划分

* 1.1.向所有的人推送同一个消息。

* 1.2.具体的某个人，或者某类人推送消息，自己简单的进行了一个SDK等封装



```
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import cn.jiguang.common.ClientConfig;
import cn.jiguang.common.resp.APIConnectionException;
import cn.jiguang.common.resp.APIRequestException;
import cn.jpush.api.JPushClient;
import cn.jpush.api.push.PushResult;
import cn.jpush.api.push.model.Options;
import cn.jpush.api.push.model.Platform;
import cn.jpush.api.push.model.PushPayload;
import cn.jpush.api.push.model.audience.Audience;
import cn.jpush.api.push.model.notification.AndroidNotification;
import cn.jpush.api.push.model.notification.IosNotification;
import cn.jpush.api.push.model.notification.Notification;
/**
 * 极光推送
 */
public class Jdpush {

    private static final Logger log = LoggerFactory.getLogger(Jdpush.class);
    
    // 设置好账号的app_key和masterSecret 
    public static final String APPKEY = "";
    
    public static final String MASTERSECRET = "";
       
        /**
         * 推送所有
         */
        public static PushPayload buildPushObjectAndroidIosAllAlert(String message){
            return PushPayload.newBuilder()
                    .setPlatform(Platform.android_ios())
                    .setAudience(Audience.all())//推送所有;
                    .setNotification(Notification.newBuilder()
                            .addPlatformNotification(AndroidNotification.newBuilder()
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .addPlatformNotification(IosNotification.newBuilder().setSound("callu")
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .build())
                    .setOptions(Options.newBuilder()
                            .setApnsProduction(false)//true-推送生产环境 false-推送开发环境（测试使用参数）
                            .setTimeToLive(90)//消息在JPush服务器的失效时间（测试使用参数）
                            .build())
                    .build();
        }
        
        
        /**
         * 推送 指定用户集合;
         */
        public static PushPayload buildPushObjectAndroidIosAliasAlert(List<String> userIds,String message){
            return PushPayload.newBuilder()
                    .setPlatform(Platform.android_ios())
                    .setAudience(Audience.alias(userIds))//推送多个;
                    .setNotification(Notification.newBuilder()
                            .addPlatformNotification(AndroidNotification.newBuilder()
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .addPlatformNotification(IosNotification.newBuilder().setSound("callu")
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .build())
                    .setOptions(Options.newBuilder()
                            .setApnsProduction(false)//true-推送生产环境 false-推送开发环境（测试使用参数）
                            .setTimeToLive(90)//消息在JPush服务器的失效时间（测试使用参数）
                            .build())
                    .build();
        }
        
        /**
         * 推送单个人;
         */
        public static PushPayload buildPushObjectAndroidIosAliasAlert(String userId,String message){
            return PushPayload.newBuilder()
                    .setPlatform(Platform.android_ios())
                    .setAudience(Audience.alias(userId))//推送单个;
                    .setNotification(Notification.newBuilder()
                            .addPlatformNotification(AndroidNotification.newBuilder()
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .addPlatformNotification(IosNotification.newBuilder().setSound("callu")
                                    .addExtra("type", "infomation")
                                    .setAlert(message)
                                    .build())
                            .build())
                    .setOptions(Options.newBuilder()
                            .setApnsProduction(false)//true-推送生产环境 false-推送开发环境（测试使用参数）
                            .setTimeToLive(90)//消息在JPush服务器的失效时间（测试使用参数）
                            .build())
                    .build();
        }
        
        /**
         * 推送所有
         */
        public static PushResult pushAlias(String alert){
            ClientConfig clientConfig = ClientConfig.getInstance();
            JPushClient jpushClient = new JPushClient(MASTERSECRET, APPKEY, null, clientConfig);
            PushPayload payload = buildPushObjectAndroidIosAllAlert(alert);
            try {
                return jpushClient.sendPush(payload);
            } catch (APIConnectionException e) {
                log.error("Connection error. Should retry later. ", e);
                return null;
            } catch (APIRequestException e) {
                log.error("Error response from JPush server. Should review and fix it. ", e);
                log.info("HTTP Status: " + e.getStatus());
                log.info("Error Code: " + e.getErrorCode());
                log.info("Error Message: " + e.getErrorMessage());
                log.info("Msg ID: " + e.getMsgId());
                return null;
            }    
        }
        
        /**
         * 推送 指定用户集合;
         */
        public static PushResult pushAlias(List<String> userIds,String alert){
            ClientConfig clientConfig = ClientConfig.getInstance();
            JPushClient jpushClient = new JPushClient(MASTERSECRET, APPKEY, null, clientConfig);
            PushPayload payload = buildPushObjectAndroidIosAliasAlert(userIds,alert);
            try {
                return jpushClient.sendPush(payload);
            } catch (APIConnectionException e) {
                log.error("Connection error. Should retry later. ", e);
                return null;
            } catch (APIRequestException e) {
                log.error("Error response from JPush server. Should review and fix it. ", e);
                log.info("HTTP Status: " + e.getStatus());
                log.info("Error Code: " + e.getErrorCode());
                log.info("Error Message: " + e.getErrorMessage());
                log.info("Msg ID: " + e.getMsgId());
                return null;
            }    
        }
        
        /**
         * 推送单个人;
         */
        public static PushResult pushAlias(String userId,String alert){
            ClientConfig clientConfig = ClientConfig.getInstance();
            JPushClient jpushClient = new JPushClient(MASTERSECRET, APPKEY, null, clientConfig);
            PushPayload payload = buildPushObjectAndroidIosAliasAlert(userId,alert);
            try {
                return jpushClient.sendPush(payload);
            } catch (APIConnectionException e) {
                log.error("Connection error. Should retry later. ", e);
                return null;
            } catch (APIRequestException e) {
                log.error("Error response from JPush server. Should review and fix it. ", e);
                log.info("HTTP Status: " + e.getStatus());
                log.info("Error Code: " + e.getErrorCode());
                log.info("Error Message: " + e.getErrorMessage());
                log.info("Msg ID: " + e.getMsgId());
                return null;
            }    
        }
        
        
}
```

### 2.业务通知

说明：有些事情，我们希望用户打开APP就知道某些事情，这个时候我们就需要做一个首页通知机制，由于这种机制是用户主动接受，因此，我们需要进行系统设计与架构

2.1.存储用户的推送消息。

2.2.统计那些用户看了与没看。

数据库设计如下：


```
CREATE TABLE `buyer_notice` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `buyer_id` bigint(20) DEFAULT NULL COMMENT '买家ID',
  `content` varchar(60) DEFAULT NULL COMMENT '内容',
  `status` int(11) DEFAULT NULL COMMENT '状态，0为未读，1为已读',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_time` datetime DEFAULT NULL COMMENT '最后更新时间，已读时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=262 DEFAULT CHARSET=utf8 COMMENT='买家通知';
```

**说明：字段相对比较简单，就是买家ID,内容，读取状态等等，
业务逻辑为：当用户进入系统，我们系统代码查询业务逻辑的时候，也查询 下这个表是否存在通知，如果已经有的，就不用弹窗，没有就弹窗，强迫用户选择已读或者未读。相对而言业务比较简单**



```
/**
 * 买家进入首页，看到的通知
 */
@RestController
@RequestMapping("/buyer")
public class NoticeController extends BaseController{

    private static final Logger logger = LoggerFactory.getLogger(MyController.class);
    
    public static final String CONTENT="平台下单时间调整为上午10:00到晚上23:59";
    
    @Autowired
    private NoticeService noticeService;
    
    /**
     * 查询消息
     */
    @RequestMapping(value = "/notice/index", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult noticeIndex(HttpServletRequest request, HttpServletResponse response,Long buyerId){
        try
        {
            if(buyerId==null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "请求参数有误，请重新输入","");
            }
            
            Notice notice=this.noticeService.getNoticeByBuyerId(buyerId);
            
            if(notice==null)
            {
                
                int result=this.noticeService.insertNotice(buyerId, CONTENT);
                
                if(result>0)
                {
                    notice=this.noticeService.getNoticeByBuyerId(buyerId);
                }
            }
            return new JsonResult(JsonResultCode.SUCCESS, "查询信息成功", notice);
            
        }catch(Exception ex){
            logger.error("[NoticeController][noticeIndex] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    
    /**
     * 更新消息
     */
    @RequestMapping(value = "/notice/update", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult noticeUpdate(HttpServletRequest request, HttpServletResponse response,Long buyerId){
        try
        {
            if(buyerId==null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "请求参数有误，请重新输入","");
            }
            
            int result=this.noticeService.updateBuyerNotice(buyerId);
            
            if(result>0)
            {
                return new JsonResult(JsonResultCode.SUCCESS, "更新成功","");
            }else
            {
                return new JsonResult(JsonResultCode.FAILURE, "更新失败","");
            }
        }catch(Exception ex){
            logger.error("[NoticeController][noticeUpdate] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
}
```

### 3.短信通知模块的设计

说明：市面上短信供应商很多，可能大家就是关注一个价格与及时性的问题，目前我们找的一个稍微便宜点的供应商：`http://api.sms.cn/`
内容其实就是短信的发送而言。
接口文档很简单：

```
参数名    参数字段    参数说明
ac    接口功能    接口功能，传入值请填写 send
format    返回格式    可选项，有三参数值：json,xml,txt 默认json格式
uid    用户账号    登录名
pwd    用户密码    32位MD5加密md5(密码+uid)
如登录密码是：123123 ，uid是：test;
pwd=md5(123123test)
pwd=b9887c5ebb23ebb294acab183ecf0769
encode    字符编码    可选项，默认接收数据是UTF-8编码,如提交的是GBK编码字符,需要添加参数 encode=gbk
mobile    接收号码    同时发送给多个号码时,号码之间用英文半角逗号分隔(,);小灵通需加区号
如:13972827282,13072827282
mobileids    消息编号    可选项
该参数用于发送短信收取状态报告用，格式为消息编号+逗号；与接收号码一一对应，可以重复出现多次。
消息编号:全部由数字组成接收状态报告的时候用到，该消息编号的格式可就为目标号码+当前时间戳整数，精确到毫秒，确保唯一性。供收取状态报告用 如: 1590049111112869461937;

content    短信内容    变量模板发送，传参规则{"key":"value"}JSON格式，key的名字须和申请模板中的变量名一致，多个变量之间以逗号隔开。示例：针对模板“短信验证码{$code}，您正在进行{$product}身份验证，请在10分钟内完成操作！”，传参时需传入{"code":"352333","product":"电商平台"}
template    模板短信ID    发送变量模板短信时需要填写对应的模板ID号，进入平台-》短信设置-》模板管理
```

对此，我们如何进行业务研究与处理呢？

* 1.短信验证码的长度与算法。

* 2.代码的模板进行封装。

* 3.短信工具类的使用方便

 

#### 1.短信验证码生成算法：

```
import org.apache.commons.lang3.RandomStringUtils;

/**
 * 短信验证码
 * 
 */
public final class SmsCode {

    /**
     * 默认产生的验证码数目
     */
    private static int DEFAULT_NUMBER = 6;

    /**
     * 产生的随机号码数目
     * 
     * @param number
     * @return
     */
    public static String createRandomCode(int number) {
        int num = number <= 3 ? DEFAULT_NUMBER : number;
        return RandomStringUtils.randomNumeric(num);
    }
}
```
简单粗暴的解决问题：

#### 2.短信内容的封装：


```
/***
 * 短信消息对象
 */
public class SmsMessage 
{
    /**
     * 账号，目前就是手机号码，采用的是手机号码登陆
     */
    private String account;
    
    /*
     * 产生的验证码 
     */
    private String code;
    
    /**
     * 对应的短信模板，目前短信验证码是401730
     */
    private String template;
    
    public SmsMessage() {
        super();
    }

    public SmsMessage(String account, String code, String template) {
        super();
        this.account = account;
        this.code = code;
        this.template = template;
    }

    public String getAccount() {
        return account;
    }

    public void setAccount(String account) {
        this.account = account;
    }

    public String getCode() {
        return code;
    }

    public void setCode(String code) {
        this.code = code;
    }

    public String getTemplate() {
        return template;
    }

    public void setTemplate(String template) {
        this.template = template;
    }

    @Override
    public String toString() {
        return "{\"username\":\""+account+"\",\"code\":\""+code+"\"}";
    }
```

#### 3.短信发送结果的封装：



```
/**
 * 短信发送结果
 */
public class SmsResult implements java.io.Serializable{

    private static final long serialVersionUID = 1L;

    private boolean success=false;
    
    private String message;
    
    public SmsResult() {
        super();
    }
    
    public SmsResult(String message) 
    {
        super();
        this.success=false;
        this.message=message;
    }
    
    public SmsResult(boolean success, String message) {
        super();
        this.success = success;
        this.message = message;
    }

    public boolean isSuccess() {
        return success;
    }

    public void setSuccess(boolean success) {
        this.success = success;
    }

    public String getMessage() {
        return message;
    }

    public void setMessage(String message) {
        this.message = message;
    }

    @Override
    public String toString() {
        StringBuilder builder = new StringBuilder();
        builder.append("SmsResult [success=");
        builder.append(success);
        builder.append(", message=");
        builder.append(message);
        builder.append("]");
        return builder.toString();
    }
}
```

#### 4.短信发送工具类的封装


```
/**
 * 短信工具
 */
@Component
public class SmsUtil {
    
    private static final Logger logger=LoggerFactory.getLogger(SmsUtil.class);
    
    @Value("#{applicationProperties['environment']}")
    private String environment;
    
    /**
     * 默认编码的格式
     */
    private static final String CHARSET="GBK";
    
    /**
     * 请求的网关接口
     */
    private static final String URL = "http://api.sms.cn/sms/";
    
    public boolean sendSms(SmsMessage smsMessage)
    {
        boolean result=true;
        
        logger.debug("[SmsUtil]当前的运行环境为："+environment);

        //开发环境就直接返回成功
        if("dev".equalsIgnoreCase(environment))
        {
            return result;
        }
        
        Map<String, String> map=new HashMap<String,String>();
        
        map.put("ac","send");
        map.put("uid","");
        map.put("pwd","");
        map.put("template",smsMessage.getTemplate());
        map.put("mobile",smsMessage.getAccount());
        map.put("content",smsMessage.toString());
        
        try
        {
            String responseContent=HttpClientUtil.getInstance().sendHttpPost(URL, map,CHARSET);
            
            logger.info("SmsUtil.sendSms.responseContent:" + responseContent);
            
            JSONObject json = JSONObject.fromObject(responseContent);
            
            logger.info("SmsUtil.sendSms.json:" + json);
            
            String stat=json.getString("stat");
            
            if(!"100".equalsIgnoreCase(stat))
            {
                result=false;
            }
            
        }catch(Exception ex)
        {
            result=false;
            logger.error("[SmsUtil][sendSms] exception:",ex);
        }
        return result;
    }
}
```
补充说明；其实我可以用一个工具类来解决所有问题，为什么我没采用呢？

1.代码耦合度高，不变管理与扩展.(业务分析，其实就是三种情况，1，发送，2，内容，3，返回结果)

2.我采用代码拆分，一个类只做一件事情，几个类分别协同开发，达到最高程度的解耦，代码清晰，维护度高。
