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

#### 失败示例：

&lt;xml&gt;

&lt;return\_code&gt;&lt;!\[CDATA\[FAIL\]\]&gt;&lt;/return\_code&gt;

&lt;return\_msg&gt;&lt;!\[CDATA\[系统繁忙,请稍后再试.\]\]&gt;&lt;/return\_msg&gt;

&lt;result\_code&gt;&lt;!\[CDATA\[FAIL\]\]&gt;&lt;/result\_code&gt;

&lt;err\_code&gt;&lt;!\[CDATA\[268458547\]\]&gt;&lt;/err\_code&gt;

&lt;err\_code\_des&gt;&lt;!\[CDATA\[系统繁忙,请稍后再试.\]\]&gt;&lt;/err\_code\_des&gt;

&lt;mch\_billno&gt;&lt;!\[CDATA\[0010010404201411170000046542\]\]&gt;&lt;/mch\_billno&gt;

&lt;mch\_id&gt;10010404&lt;/mch\_id&gt;

&lt;wxappid&gt;&lt;!\[CDATA\[wx6fa7e3bab7e15415\]\]&gt;&lt;/wxappid&gt;

&lt;re\_openid&gt;&lt;!\[CDATA\[onqOjjmM1tad-3ROpncN-yUfa6uI\]\]&gt;&lt;/re\_openid&gt;

&lt;total\_amount&gt;1&lt;/total\_amount&gt;

&lt;/xml&gt;

### 错误码

| 错误码 | 错误描述 | 原因 | 解决方式 |
| :--- | :--- | :--- | :--- |
| NO\_AUTH | 发放失败，此请求可能存在风险，已被微信拦截 | 用户账号异常，被拦截 | 请提醒用户检查自身帐号是否异常。使用常用的活跃的微信号可避免这种情况。 |
| SENDNUM\_LIMIT | 该用户今日领取红包个数超过限制 | 该用户今日领取红包个数超过你在微信支付商户平台配置的上限 | 如有需要、请在微信支付商户平台【api安全】中重新配置 【每日同一用户领取本商户红包不允许超过的个数】。 |
| ILLEGAL\_APPID | 非法appid，请确认是否为公众号的appid，不能为APP的appid | 错误传入了app的appid | 接口传入的所有appid应该为公众号的appid（在mp.weixin.qq.com申请的），不能为APP的appid（在open.weixin.qq.com申请的）。 |
| MONEY\_LIMIT | 红包金额发放限制 | 发送红包金额不再限制范围内 | 每个红包金额必须大于1元，小于200元（可联系微信支付wxhongbao@tencent.com申请调高额度） |
| SEND\_FAILED | 红包发放失败,请更换单号再重试 | 该红包已经发放失败 | 如果需要重新发放，请更换单号再发放 |
| FATAL\_ERROR | openid和原始单参数不一致 | 更换了openid，但商户单号未更新 | 请商户检查代码实现逻辑 |
| 金额和原始单参数不一致 | 更换了金额，但商户单号未更新 | 请商户检查代码实现逻辑 | 请检查金额、商户订单号是否正确 |
| CA\_ERROR | CA证书出错，请登录微信支付商户平台下载证书 | 请求携带的证书出错 | 到商户平台下载证书，请求带上证书后重试 |
| SIGN\_ERROR | 签名错误 | 1、没有使用商户平台设置的商户API密钥进行加密（有可能之前设置过密钥，后来被修改了，没有使用新的密钥进行加密）。 2、加密前没有按照文档进行参数排序（可参考文档） 3、把值为空的参数也进行了签名。可到（http://mch.weixin.qq.com/wiki/tools/signverify/ ）验证。 4、如果以上3步都没有问题，把请求串中\(post的数据）里面中文都去掉，换成英文，试下，看看是否是编码问题。（post的数据要求是utf8） | 1. 到商户平台重新设置新的密钥后重试 2. 检查请求参数把空格去掉重试 3. 中文不需要进行encode，使用CDATA 4. 按文档要求生成签名后再重试 在线签名验证工具：http://mch.weixin.qq.com/wiki/tools/signverify/ |
| SYSTEMERROR | 请求已受理，请稍后使用原单号查询发放结果 | 系统无返回明确发放结果 | 使用原单号调用接口，查询发放结果，如果使用新单号调用接口，视为新发放请求 |
| XML\_ERROR | 输入xml参数格式错误 | 请求的xml格式错误，或者post的数据为空 | 检查请求串，确认无误后重试 |
| FREQ\_LIMIT | 超过频率限制,请稍后再试 | 受频率限制 | 请对请求做频率控制（可联系微信支付wxhongbao@tencent.com申请调高） |
| NOTENOUGH | 帐号余额不足，请到商户平台充值后再重试 | 账户余额不足 | 充值后重试 |
| OPENID\_ERROR | openid和appid不匹配 | openid和appid不匹配 | 发红包的openid必须是本appid下的openid |
| PARAM\_ERROR | act\_name字段必填,并且少于32个字符 | 请求的act\_name字段填写错误 | 填写正确的act\_name后重试 |
|  | 发放金额、最小金额、最大金额必须相等 | 请求的金额相关字段填写错误 | 按文档要求填写正确的金额后重试 |
|  | 红包金额参数错误 | 红包金额过大 | 修改金额重试 |
|  | appid字段必填,最长为32个字符 | 请求的appid字段填写错误 | 填写正确的appid后重试 |
|  | 订单号字段必填,最长为28个字符 | 请求的mch\_billno字段填写错误 | 填写正确的billno后重试 |
|  | client\_ip必须是合法的IP字符串 | 请求的client\_ip填写不正确 | 填写正确的IP后重试 |
|  | 输入的商户号有误 | 请求的mchid字段非法（或者没填） | 填写对应的商户号再重试 |
|  | 找不到对应的商户号 | 请求的mchid字段填写错误 | 填写正确的mchid字段后重试 |
|  | nick\_name字段必填，并且少于16字符 | 请求的nick\_name字段错误 | 按文档填写正确的nick\_name后重试 |
|  | nonce\_str字段必填,并且少于32字符 | 请求的nonce\_str字段填写不正确 | 按文档要求填写正确的nonce\_str值后重试 |
|  | re\_openid字段为必填并且少于32个字符 | 请求的re\_openid字段非法 | 填写对re\_openid后重试 |
|  | remark字段为必填,并且少于256字符 | 请求的remark字段填写错误 | 填写正确的remark后重试 |
|  | send\_name字段为必填并且少于32字符 | 请求的send\_name字段填写不正确 | 按文档填写正确的send\_name字段后重试 |
|  | total\_num必须为1 | total\_num字段值不为1 | 修改total\_num值为1后重试 |
|  | wishing字段为必填,并且少于128个字符 | 缺少wishing字段 | 填写wishing字段再重试 |
|  | 商户号和wxappid不匹配 | 商户号和wxappid不匹配 | 请修改Mchid或wxappid参数 |

CMS如何调用支付接口：

### 登录微信支付商户平台下载证书以及充值

在调用接口前，请商户使用微信支付商户号登录微信支付商户平台完成下述工作：

备注：

微信支付商户平台地址为pay.weixin.qq.com。微信支付商户号会在商户申请微信支付成功后，通过开户邮件发送给您。请不要使用微信公众平台账号或者appid登录。如果您登录时遇到问题，请联系微信支付小助手weixinpay@tencent.com

