# Java生鲜电商平台-提现模块的设计与架构

补充说明：生鲜电商平台-提现模块的设计与架构，提现功能指的卖家把在平台挣的钱提现到自己的支付宝或者银行卡的一个过程。

功能相对而言不算复杂，有以下几个功能需要处理。

业务逻辑如下：
1.卖家登陆自己的B2B系统提交提现功能。

2.如果没有绑定银行卡或者支付宝，则需要先绑定银行卡或者支付宝账户，以及填写提现密码

3.支付宝或者银行卡需要跟用户的姓名所填一致，防止错误转账。

4.后端需要记录所提现的记录，实际情况是支付宝提现需要收取手续费，这个也需要记录在内。

5.需要形成一个审核机制，用户提现的状态有申请提现，审核成功，提现成功，提现失败四种可能状态。

6.每个提现的过程需要记录时间轴，如果有拒绝，用户需要查看拒绝的原因。

7.所有的提现到账后，需要平台短信通知用户申请了提现，提现成功，包括提现拒绝等等，都需要短信通知，给用户一个信任感。

8.每天晚上5:30之前提现当日到达，之后的次日早上10点钟到达。

9.系统自动审核提现的金额数据量的正确与否，来源于用户的订单以及账单数据。

相关的系统设计表如下:
1.提现信息表，为了便于大家理解，我详细的注释都写上了。


```
CREATE TABLE `withdrawal` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `uid` bigint(20) NOT NULL COMMENT '提现申请人',
  `withdraw_order` varchar(64) NOT NULL COMMENT '提现订单号,系统自动生成的.',
  `withdraw_bank_id` bigint(20) NOT NULL COMMENT '用户对应的卡的编号',
  `withdraw_charge` decimal(12,2) NOT NULL COMMENT '提现手续费',
  `withdraw_reality_total` decimal(12,2) NOT NULL COMMENT '实际提现金额',
  `withdraw_apply_total` decimal(12,2) NOT NULL COMMENT '申请提现的金额',
  `withdraw_apply_time` datetime NOT NULL COMMENT '申请提现时间',
  `status` int(11) NOT NULL COMMENT '提现状态,1表示申请提现,2表示审批通过,3,交易完成,-1审批不通过.',
  `create_by` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  `update_by` bigint(20) DEFAULT NULL COMMENT '修改人',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后修改时间',
  PRIMARY KEY (`id`),
  KEY `unique_order` (`withdraw_order`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=6 DEFAULT CHARSET=utf8 COMMENT='提现信息表';
```
2.卖家绑定卡的记录表


```
CREATE TABLE `withdrawal_bank` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `uid` bigint(20) NOT NULL COMMENT '用户ID',
  `cnname` varchar(8) NOT NULL COMMENT '中文姓名',
  `bank_code` varchar(32) NOT NULL COMMENT '卡的缩写，例如：ICBC',
  `bank_name` varchar(32) NOT NULL COMMENT '卡的名字',
  `bank_number` varchar(64) NOT NULL COMMENT '卡号',
  `sequence` tinyint(11) DEFAULT NULL COMMENT '排序用。按照小到大排序。',
  `create_time` datetime NOT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=13 DEFAULT CHARSET=utf8 COMMENT='用户绑定的银行';
```
补充说明：如果是支付宝，那么bank_code填写alipay,bank_name为支付宝，bank_number为支付宝卡号,cnname为提现的姓名

3.卖家提现日志表。（会根据卖家的提现时间，形成时间轴）


```
CREATE TABLE `withdrawal_logs` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `uid` bigint(20) NOT NULL COMMENT '提现申请人',
  `withdraw_order` varchar(64) NOT NULL COMMENT '提现订单号,系统自动生成的.',
  `remark` varchar(64) NOT NULL COMMENT '备注',
  `status` int(11) DEFAULT NULL COMMENT '提现的状态',
  `create_by` bigint(20) DEFAULT NULL COMMENT '创建人',
  `create_time` datetime DEFAULT NULL COMMENT '创建时间',
  PRIMARY KEY (`id`),
  KEY `unique_order` (`withdraw_order`) USING BTREE
) ENGINE=InnoDB AUTO_INCREMENT=9 DEFAULT CHARSET=utf8 COMMENT='用户提现日志表';
```
补充说明:形成时间轴来显示。

4.卖家提现密码：

```
CREATE TABLE `withdrawal_password` (
  `id` bigint(20) unsigned NOT NULL AUTO_INCREMENT COMMENT '自动增加ID',
  `uid` bigint(20) NOT NULL COMMENT '用户ID',
  `password` varchar(32) NOT NULL COMMENT '密码，md5加密',
  `create_time` datetime NOT NULL COMMENT '记录创建时间',
  `last_update_time` datetime DEFAULT NULL COMMENT '最后一次更新时间',
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_key_uid` (`uid`)
) ENGINE=InnoDB AUTO_INCREMENT=11 DEFAULT CHARSET=utf8 COMMENT='用户提现密码';
```
补充说明：由于设计到资金安全问题，提现需要设置提现密码，这个有别于用户的登陆密码。
整个业务比较简单，只是步骤比较多而已。
相关的业务核心代码如下：
2.1 卖家绑定自己的银行卡或者支付宝


```
/**
     * 添加用户银行信息
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/withdrawalBank/add", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult  addWithdrawalBank(HttpServletRequest request, HttpServletResponse response,@RequestBody WithdrawalBank withdrawalBank) {
        
        try
        {
            if(withdrawalBank==null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "传入对象有误", "");
            }
            
            Long uid = withdrawalBank.getUid();
            String bankCode = withdrawalBank.getBankCode();
            if(uid == null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "参数有误", "");
            }
            
            //拿到当前银行卡的唯一编号
            WithdrawalBank dbWithdrawalBank = withdrawalBankService.getWithdrawalBankByUidAndBankCode(uid, bankCode);
            
            if(dbWithdrawalBank != null){
                return new JsonResult(JsonResultCode.FAILURE, "卡已存在,请重试",dbWithdrawalBank); 
            }
            
            int result = withdrawalBankService.insertWithdrawalBank(withdrawalBank);
            
            if (result>0) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "添加用户银行成功", result);
            } 
            return new JsonResult(JsonResultCode.FAILURE, "添加用户银行失败", "");
        }catch(Exception e){
            
            logger.error("[WithdrawalBankController][addWithdrawalBank] exception :",e);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
```

2.2.修改与管理自己的提现密码：



```
/**
     * 提现金额计算
     */
    @RequestMapping(value = "/withdrawal/count", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult countWithdraw(HttpServletRequest request, HttpServletResponse response,
            BigDecimal withdrawApplyTotal, Long userId) {
        logger.info("WithdrawalController.countWithdraw.start");
        try {
            Money m = new Money(withdrawApplyTotal);
            Map<String, BigDecimal> result = new HashMap<String, BigDecimal>();
            BigDecimal withdrawCharge = null;
            BigDecimal withdrawRealityTotal = null;
            if (m.compareTo(new Money(1500)) < 0) {
                // 提现手续费
                withdrawCharge = (new BigDecimal(2).add(withdrawApplyTotal.multiply(new BigDecimal(0.0055))))
                        .setScale(2, BigDecimal.ROUND_HALF_UP);
                // 实际提现金额
                withdrawRealityTotal = withdrawApplyTotal.subtract(withdrawCharge);
            } else {
                // 提现手续费
                withdrawCharge = withdrawApplyTotal.multiply(new BigDecimal(0.007)).setScale(2,
                        BigDecimal.ROUND_HALF_UP);
                // 实际提现金额
                withdrawRealityTotal = withdrawApplyTotal.subtract(withdrawCharge);
            }

            result.put("withdrawCharge", withdrawCharge);
            result.put("withdrawRealityTotal", withdrawRealityTotal);
            result.put("withdrawApplyTotal", withdrawApplyTotal);
            return new JsonResult(JsonResultCode.SUCCESS, "计算成功", result);
        } catch (Exception ex) {
            logger.error("[WithdrawalController][countWithdraw]exception ", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统异常，请稍后再试", "");
        }
    }

    /**
     * 提现申请
     */
    @RequestMapping(value = "/withdrawal/apply", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult applyWithDrawal(HttpServletRequest request, HttpServletResponse response,
            @RequestBody Withdrawal withdrawal) {
        logger.info("WithdrawalController.applyWithDrawal.start");
        try 
        {
            Long userId = withdrawal.getUid();
            
            if (userId==null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "参数异常!", "");
            }
            // 查询提现表中是否存在当前用户正在审核的提现，如果存在就不允许继续申请
            Withdrawal withdrawalByUserId = withdrawalService.getWithdrawalByUserId(userId);
            
            if (withdrawalByUserId != null) 
            {
                return new JsonResult(JsonResultCode.FAILURE, "已有提现记录,正在审核中!", withdrawalByUserId);
            }

            //所属卖家
            Seller seller = sellerService.getSellerById(userId);
            
            // 卖家可提现金额
            BigDecimal billMoney = seller.getBillMoney();

            logger.info("[WithdrawalController][applyWithDrawal]当前用户userId:" + userId + " 可提现金额:" + billMoney);

            // 卖家申请提现金额;
            BigDecimal withdrawApplyTotal = withdrawal.getWithdrawApplyTotal();

            logger.info("[WithdrawalController][applyWithDrawal]当前用户userId:" + userId + " 申请的提现金额:" + withdrawApplyTotal);

            // 如果申请的金额大于系统的金额，则返回1，同时不符合逻辑，直接返回error
            if (withdrawApplyTotal.compareTo(billMoney) == 1) {
                return new JsonResult(JsonResultCode.FAILURE, "申请金额错误,返回重试!", "");
            }
            
            String orderNumber = OrderIDGenerator.getOrderNumber();
            withdrawal.setWithdrawApplyTime(new Date());
            withdrawal.setCreateTime(new Date());
            withdrawal.setWithdrawOrder(orderNumber);
            // 申请提现
            withdrawal.setStatus(WithdrawalConstant.APPLY_WITHDRAWAL);

            WithdrawalLogs withdrawalLogs = new WithdrawalLogs();
            withdrawalLogs.setWithdrawOrder(orderNumber);
            withdrawalLogs.setCreateTime(new Date());
            withdrawalLogs.setStatus(WithdrawalConstant.APPLY_WITHDRAWAL);
            withdrawalLogs.setCreateBy(userId);
            withdrawalLogs.setUid(userId);
            withdrawalLogs.setRemark("提现已提交,审核中!");
            withdrawalService.applyWithdrawal(withdrawal, withdrawalLogs);

            sendSmsNotice(withdrawal, userId, seller, billMoney, withdrawApplyTotal);
            
            return new JsonResult(JsonResultCode.SUCCESS, "申请提现成功", "");
        } catch (Exception ex) {
            logger.error("[WithdrawalController][applyWithDrawal]exception ", ex);
            return new JsonResult(JsonResultCode.FAILURE, "申请失败，系统异常，请稍后再试", "");
        }
    }

    /**
     * 发送短信通知给卖家
     * @param withdrawal
     * @param userId
     * @param seller
     * @param billMoney
     * @param withdrawApplyTotal
     */
    private void sendSmsNotice(Withdrawal withdrawal, Long userId, Seller seller, BigDecimal billMoney,BigDecimal withdrawApplyTotal) 
    {
        try
        {
            // 发送短信给卖家
            SmsMessage smsMessage = new SmsMessage();
            smsMessage.setAccount(seller.getSellerAccount());
            smsMessage.setAmount(withdrawal.getWithdrawApplyTotal());
            smsMessage.setName(seller.getRealName() == null ? "" : seller.getRealName());
            SmsMessageService smsMessageService = new SmsMessageServiceImpl();
            smsMessageService.applicationWithdrawal(smsMessage, environment);

            // 保存信息到短信日志中
            Sms sms = new Sms();
            String msg = "当前用户userId:" + userId + ",申请的提现金额:" + withdrawApplyTotal + ",可提现金额:" + billMoney;
            sms.setPhone(seller.getSellerAccount());
            sms.setMessage(msg);
            sms.setRemark("提现申请");
            sms.setCreateTime(new Date());
            smsService.saveSms(sms);
        }catch(Exception ex)
        {
            logger.error("[WithdrawalController][sendSmsNotice]exception",ex);
        }
    }
    
    /**
     * 提现申请 列表
     * @param withdrawal
     *            传递查询条件
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/withdrawal/applyList", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult applyWithDrawalList(HttpServletRequest request, HttpServletResponse response, Long userId,
            int currentPageNum, int currentPageSize) {
        logger.info("WithdrawalController.applyWithDrawalList.start");
        try 
        {
            if (null == userId) {
                return new JsonResult(JsonResultCode.FAILURE, "参数有误,userId不能为空", "");
            }

            PageUtil pageUtil = withdrawalService.getPageResult(userId, currentPageNum, currentPageSize);
            return new JsonResult(JsonResultCode.SUCCESS, "查询成功", pageUtil);
        } catch (Exception ex) {
            logger.error("[WithdrawalController][applyWithDrawalList]exception ", ex);
            return new JsonResult(JsonResultCode.FAILURE, "申请失败，系统异常，请稍后再试", "");
        }
    }

    /**
     * 根据提现订单号获取订单的详细情况
     * 
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/withdrawal/order", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult withdrawalOrder(HttpServletRequest request, HttpServletResponse response,
            String withdrawalOrder) {
        logger.info("WithdrawalController.withdrawalOrder.start");
        try {
            if (StringUtils.isBlank(withdrawalOrder)) {
                return new JsonResult(JsonResultCode.FAILURE, "提现订单号有误，请重新输入", "");
            }
            WithdrawalQuery withdrawal = withdrawalService.getWithdrawalByWithdrawalOrder(withdrawalOrder);

            if (withdrawal == null) {
                return new JsonResult(JsonResultCode.FAILURE, "提现订单号不存在，请重新填写", "");
            }
            return new JsonResult(JsonResultCode.SUCCESS, "查询成功", withdrawal);
        } catch (Exception ex) {
            logger.error("[WithdrawalController][applyWithDrawalList]exception ", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统异常，请稍后再试", "");
        }
    }
```
3.提现记录表核心代码


```
/**
 * 卖家提现功能---提现银行设置
 */
@RestController
@RequestMapping("/seller")
public class WithdrawalBankController extends BaseController 
{

    private static final Logger logger = LoggerFactory.getLogger(WithdrawalBankController.class);
    
    @Autowired
    private WithdrawalBankService withdrawalBankService;
    
    /**
     * 根据用户Uid查询用户绑定的银行卡信息；
     * @param request
     * @param response
     * @param withdrawal 条件查询
     * @return
     */
    @RequestMapping(value = "/withdrawalBank/list", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult withdrawalBankList(HttpServletRequest request, HttpServletResponse response,Long userId,Model model) 
    {

        try
        {
            List<WithdrawalBank> withdrawalBankList = withdrawalBankService.getWithdrawalBankByUid(userId);
            
            if(CollectionUtils.isEmpty(withdrawalBankList))
            {
                return new JsonResult(JsonResultCode.SUCCESS, "用户未绑定银行卡", withdrawalBankList);
            }
            return new JsonResult(JsonResultCode.SUCCESS, "查询用户银行卡信息", withdrawalBankList);
        }catch(Exception ex){
            logger.error("[WithdrawalBankController][withdrawalBankList] exception :",ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
    
    /**
     * 添加用户银行信息
     * @param request
     * @param response
     * @return
     */
    @RequestMapping(value = "/withdrawalBank/add", method = { RequestMethod.GET, RequestMethod.POST })
    public JsonResult  addWithdrawalBank(HttpServletRequest request, HttpServletResponse response,@RequestBody WithdrawalBank withdrawalBank) {
        
        try
        {
            if(withdrawalBank==null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "传入对象有误", "");
            }
            
            Long uid = withdrawalBank.getUid();
            String bankCode = withdrawalBank.getBankCode();
            if(uid == null)
            {
                return new JsonResult(JsonResultCode.FAILURE, "参数有误", "");
            }
            
            //拿到当前银行卡的唯一编号
            WithdrawalBank dbWithdrawalBank = withdrawalBankService.getWithdrawalBankByUidAndBankCode(uid, bankCode);
            
            if(dbWithdrawalBank != null){
                return new JsonResult(JsonResultCode.FAILURE, "卡已存在,请重试",dbWithdrawalBank); 
            }
            
            int result = withdrawalBankService.insertWithdrawalBank(withdrawalBank);
            
            if (result>0) 
            {
                return new JsonResult(JsonResultCode.SUCCESS, "添加用户银行成功", result);
            } 
            return new JsonResult(JsonResultCode.FAILURE, "添加用户银行失败", "");
        }catch(Exception e){
            
            logger.error("[WithdrawalBankController][addWithdrawalBank] exception :",e);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试","");
        }
    }
}
```

4.卖家提现日志表：


```
/**
 * 卖家提现功能---提现日志记录
 */
@RestController
@RequestMapping("/seller")
public class WithdrawalLogsController extends BaseController {

    private static final Logger logger = LoggerFactory.getLogger(WithdrawalLogsController.class);

    @Autowired
    private WithdrawalLogsService withdrawalLogsService;

    /**
     * 根据Uid和withdrawOrder查询单个提现详情
     * 
     * @param userId
     * @param withdrawOrder
     * @return
     */
    @RequestMapping(value = "/withdrawalLogs/getLogsByWithdrawOrder", method = { RequestMethod.GET,RequestMethod.POST })
    public JsonResult getWithdrawalLogsByUidAndWithdrawOrder(HttpServletRequest request, HttpServletResponse response,
            Long userId, String withdrawOrder) {

        try
        {
            if (StringUtils.isBlank(withdrawOrder)) {
                return new JsonResult(JsonResultCode.FAILURE, "请求参数异常", "");
            }
            List<WithdrawalLogs> withdrawalLogs = withdrawalLogsService.getWithdrawalLogsByWithdrawOrder(withdrawOrder);
            return new JsonResult(JsonResultCode.SUCCESS, "订单详情", withdrawalLogs);
        } catch (Exception ex) {
            logger.error("[WithdrawalLogsController][getWithdrawalLogsByUidAndWithdrawOrder]", ex);
            return new JsonResult(JsonResultCode.FAILURE, "系统错误,请稍后重试", "");
        }
    }
}
```

相关的运营截图如下：
641237-20180517215059767-1310554111.png
641237-20180517215001448-1896295605.png
641237-20180517215015003-565780502.png
641237-20180517215027401-1637273574.png

641237-20180517215042261-1059117618.png
