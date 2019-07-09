# 发送普通红包

#### 发放规则

1.发送频率限制------默认1800/min

2.发送个数上限------默认1800/min

3.场景金额限制------默认红包金额为1-200元，如有需要，可前往商户平台进行设置和申请

4.其他限制------单用户可领取红包上限为10个/天

注意事项：

◆ 红包金额大于200或者小于1元时，请求参数scene\_id必传，参数说明见下文。  
◆ 根据监管要求，新申请商户号使用现金红包需要满足两个条件：1、入驻时间超过90天 2、连续正常交易30天。  
◆ 移动应用的appid无法使用红包接口。  
◆ 当返回错误码为“SYSTEMERROR”时，请不要更换商户订单号，一定要使用原商户订单号重试，否则可能造成重复发放红包等资金风险。  
◆ XML具有可扩展性，因此返回参数可能会有新增，而且顺序可能不完全遵循此文档规范，如果在解析回包的时候发生错误，请商户务必不要换单重试，请商户联系客服确认红包发放情况。如果有新回包字段，会更新到此API文档中。  
◆ 因为错误代码字段err\_code的值后续可能会增加，所以商户如果遇到回包返回新的错误码，请商户务必不要换单重试，请商户联系客服确认红包发放情况。如果有新的错误码，会更新到此API文档中。  
◆ 错误代码描述字段err\_code\_des只供人工定位问题时做参考，系统实现时请不要依赖这个字段来做自动化处理。

#### 消息触达规则

现金红包发放后会以公众号消息的形式触达用户，不同情况下触达消息的形式会有差别，相关规则如下：

1.已关注公众号的用户，使用“防伪消息”触达；

2.未关注公众号的用户，使用“模板消息”触达。

#### 接口调用请求说明

| 请求Url |
| :--- |


|  | [https://api.mch.weixin.qq.com/mmpaymkttransfers/sendredpack](https://api.mch.weixin.qq.com/mmpaymkttransfers/sendredpack) |
| :--- | :--- |
| 是否需要证书 | 是（证书及使用说明详见[商户证书](https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=4_3)） |
| 请求方式 | POST |

#### 请求参数

| 字段名 |
| :--- |


|  | 字段 | 必填 | 示例值 | 类型 | 说明 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 随机字符串 | nonce\_str | 是 | 5K8264ILTKCH16CQ2502SI8ZNMTM67VS | String\(32\) | 随机字符串，不长于32位 |
| 签名 | sign | 是 | C380BEC2BFD727A4B6845133519F3AD6 | String\(32\) | 详见[签名生成算法](https://pay.weixin.qq.com/wiki/doc/api/tools/cash_coupon.php?chapter=4_3) |
| 商户订单号 | mch\_billno | 是 | 10000098201411111234567890 | String\(28\) | 商户订单号（每个订单号必须唯一。取值范围：0~9，a~z，A~Z）接口根据商户订单号支持重入，如出现超时可再调用。 |
| 商户号 | mch\_id | 是 | 10000098 | String\(32\) | 微信支付分配的商户号 |
| 公众账号appid | wxappid | 是 | wx8888888888888888 | String\(32\) | 微信分配的公众账号ID（企业号corpid即为此appId）。在微信开放平台（open.weixin.qq.com）申请的移动应用appid无法使用该接口。 |
| 商户名称 | send\_name | 是 | 天虹百货 | String\(32\) | 红包发送者名称注意：敏感词会被转义成字符\* |
| 用户openid | re\_openid | 是 | oxTWIuGaIt6gTKsQRLau2M0yL16E | String\(32\) | 接受红包的用户openidopenid为用户在wxappid下的唯一标识（获取openid参见微信公众平台开发者文档：[网页授权获取用户基本信息](http://mp.weixin.qq.com/wiki/4/9ac2e7b1f1d22e9e57260f6553822520.html)） |
| 付款金额 | total\_amount | 是 | 1000 | int | 付款金额，单位分 |
| 红包发放总人数 | total\_num | 是 | 1 | int | 红包发放总人数total\_num=1 |
| 红包祝福语 | wishing | 是 | 感谢您参加猜灯谜活动，祝您元宵节快乐！ | String\(128\) | 红包祝福语注意：敏感词会被转义成字符\* |
| Ip地址 | client\_ip | 是 | 192.168.0.1 | String\(15\) | 调用接口的机器Ip地址 |
| 活动名称 | act\_name | 是 | 猜灯谜抢红包活动 | String\(32\) | 活动名称注意：敏感词会被转义成字符\* |
| 备注 | remark | 是 | 猜越多得越多，快来抢！ | String\(256\) | 备注信息 |
| 场景id | scene\_id | 否 | PRODUCT\_8 | String\(32\) | 发放红包使用场景，红包金额大于200或者小于1元时必传PRODUCT\_1:商品促销PRODUCT\_2:抽奖PRODUCT\_3:虚拟物品兑奖 PRODUCT\_4:企业内部福利PRODUCT\_5:渠道分润PRODUCT\_6:保险回馈PRODUCT\_7:彩票派奖PRODUCT\_8:税务刮奖 |
| 活动信息 | risk\_info | 否 | posttime%3d123123412%26clientversion%3d234134%26mobile%3d122344545%26deviceid%3dIOS | String\(128\) | posttime:用户操作的时间戳mobile:业务系统账号的手机号，国家代码-手机号。不需要+号deviceid :mac 地址或者设备唯一标识 clientversion :用户操作的客户端版本把值为非空的信息用key=value进行拼接，再进行urlencodeurlencode\(posttime=xx& mobile =xx&deviceid=xx\) |

数据示例：

| &lt;xml&gt;&lt;sign&gt;&lt;!\[CDATA\[E1EE61A91C8E90F299DE6AE075D60A2D\]\]&gt;&lt;/sign&gt;&lt;mch\_billno&gt;&lt;!\[CDATA\[0010010404201411170000046545\]\]&gt;&lt;/mch\_billno&gt;&lt;mch\_id&gt;&lt;!\[CDATA\[888\]\]&gt;&lt;/mch\_id&gt;&lt;wxappid&gt;&lt;!\[CDATA\[wxcbda96de0b165486\]\]&gt;&lt;/wxappid&gt;&lt;send\_name&gt;&lt;!\[CDATA\[send\_name\]\]&gt;&lt;/send\_name&gt;&lt;re\_openid&gt;&lt;!\[CDATA\[onqOjjmM1tad-3ROpncN-yUfa6uI\]\]&gt;&lt;/re\_openid&gt;&lt;total\_amount&gt;&lt;!\[CDATA\[200\]\]&gt;&lt;/total\_amount&gt;&lt;total\_num&gt;&lt;!\[CDATA\[1\]\]&gt;&lt;/total\_num&gt;&lt;wishing&gt;&lt;!\[CDATA\[恭喜发财\]\]&gt;&lt;/wishing&gt;&lt;client\_ip&gt;&lt;!\[CDATA\[127.0.0.1\]\]&gt;&lt;/client\_ip&gt;&lt;act\_name&gt;&lt;!\[CDATA\[新年红包\]\]&gt;&lt;/act\_name&gt;&lt;remark&gt;&lt;!\[CDATA\[新年红包\]\]&gt;&lt;/remark&gt;&lt;scene\_id&gt;&lt;!\[CDATA\[PRODUCT\_2\]\]&gt;&lt;/scene\_id&gt;&lt;nonce\_str&gt;&lt;!\[CDATA\[50780e0cca98c8c8e814883e5caa672e\]\]&gt;&lt;/nonce\_str&gt;&lt;risk\_info&gt;posttime%3d123123412%26clientversion%3d234134%26mobile%3d122344545%26deviceid%3dIOS&lt;/risk\_info&gt;&lt;/xml&gt; |
| :--- |


### 返回参数

| 字段名 | 变量名 | 必填 | 示例值 | 类型 | 说明 |
| :--- | :--- | :--- | :--- | :--- | :--- |
| 返回状态码 | return\_code | 是 | SUCCESS | String\(16\) | SUCCESS/FAIL此字段是通信标识，非红包发放结果标识，红包发放是否成功需要查看result\_code来判断 |
| 返回信息 | return\_msg | 否 | 签名失败 | String\(128\) | 返回信息，如非空，为错误原因签名失败参数格式校验错误 |
|  |  |  |  |  | 以下字段在return\_code为SUCCESS的时候有返回 |
| 业务结果 | result\_code | 是 | SUCCESS | String\(16\) | SUCCESS/FAIL注意：当状态为FAIL时，存在业务结果未明确的情况。所以如果状态是FAIL，请务必再请求一次查询接口\[请务必关注错误代码（err\_code字段），通过查询得到的红包状态确认此次发放的结果。\]，以确认此次发放的结果。 |
| 错误代码 | err\_code | 否 | SYSTEMERROR | String\(32\) | 错误码信息注意：出现未明确的错误码（SYSTEMERROR等）时，请务必用原商户订单号重试，或者再请求一次查询接口以确认此次发放的结果。 |
| 错误代码描述 | err\_code\_des | 否 | 系统错误 | String\(128\) | 结果信息描述 |
|  |  |  |  |  | 以下字段在return\_code和result\_code都为SUCCESS的时候有返回 |
| 商户订单号 | mch\_billno | 是 | 10000098201411111234567890 | String\(28\) | 商户订单号（每个订单号必须唯一）组成：mch\_id+yyyymmdd+10位一天内不能重复的数字 |
| 商户号 | mch\_id | 是 | 10000098 | String\(32\) | 微信支付分配的商户号 |
| 公众账号appid | wxappid | 是 | wx8888888888888888 | String\(32\) | 商户appid，接口传入的所有appid应该为公众号的appid（在mp.weixin.qq.com申请的），不能为APP的appid（在open.weixin.qq.com申请的）。 |
| 用户openid | re\_openid | 是 | oxTWIuGaIt6gTKsQRLau2M0yL16E | String\(32\) | 接受收红包的用户用户在wxappid下的openid |
| 付款金额 | total\_amount | 是 | 1000 | int | 付款金额，单位分 |
| 微信单号 | send\_listid | 是 | 100000000020150520314766074200 | String\(32\) | 红包订单的微信单号 |



