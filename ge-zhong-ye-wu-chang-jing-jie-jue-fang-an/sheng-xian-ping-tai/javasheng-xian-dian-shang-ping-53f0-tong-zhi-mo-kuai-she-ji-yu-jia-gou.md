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
