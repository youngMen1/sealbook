# 28 | 安全兜底：涉及钱时，必须考虑防刷、限量和防重

你好，我是朱晔。今天，我要和你分享的主题是，任何涉及钱的代码必须要考虑防刷、限量和防重，要做好安全兜底。

涉及钱的代码，主要有以下三类。

第一，代码本身涉及有偿使用的三方服务。如果因为代码本身缺少授权、用量控制而被利用导致大量调用，势必会消耗大量的钱，给公司造成损失。有些三方服务可能采用后付款方式的结算，出现问题后如果没及时发现，下个月结算时就会收到一笔数额巨大的账单。

第二，代码涉及虚拟资产的发放，比如积分、优惠券等。虽然说虚拟资产不直接对应货币，但一般可以在平台兑换具有真实价值的资产。比如，优惠券可以在下单时使用，积分可以兑换积分商城的商品。所以从某种意义上说，虚拟资产就是具有一定价值的钱，但因为不直接涉及钱和外部资金通道，所以容易产生随意性发放而导致漏洞。

第三，代码涉及真实钱的进出。比如，对用户扣款，如果出现非正常的多次重复扣款，小则用户投诉、用户流失，大则被相关管理机构要求停业整改，影响业务。又比如，给用户发放返现的付款功能，如果出现漏洞造成重复付款，涉及 B 端的可能还好，但涉及 C 端用户的重复付款可能永远无法追回。

前段时间拼多多一夜之间被刷了大量 100 元无门槛优惠券的事情，就是限量和防刷出了问题。

今天，我们就通过三个例子，和你说明如何在代码层面做好安全兜底。

## 开放平台资源的使用需要考虑防刷

我以真实遇到的短信服务被刷案例，和你说说防刷。

有次短信账单月结时发现，之前每个月是几千元的短信费用，这个月突然变为了几万元。查数据库记录发现，之前是每天发送几千条短信验证码，从某天开始突然变为了每天几万条，但注册用户数并没有激增。显然，这是短信接口被刷了。

我们知道，短信验证码服务属于开放性服务，由用户侧触发，且因为是注册验证码所以不需要登录就可以使用。如果我们的发短信接口像这样没有任何防刷的防护，直接调用三方短信通道，就相当于“裸奔”，很容易被短信轰炸平台利用：



```

@GetMapping("wrong")
public void wrong() {
    sendSMSCaptcha("13600000000");
}

private void sendSMSCaptcha(String mobile) {
  //调用短信通道
}
```

对于短信验证码这种开放接口，程序逻辑内需要有防刷逻辑。好的防刷逻辑是，对正常使用的用户毫无影响，只有疑似异常使用的用户才会感受到。对于短信验证码，有如下 4 种可行的方式来防刷。

**第一种方式，只有固定的请求头才能发送验证码。**

也就是说，我们通过请求头中网页或 App 客户端传给服务端的一些额外参数，来判断请求是不是 App 发起的。其实，这种方式“防君子不防小人”。

比如，判断是否存在浏览器或手机型号、设备分辨率请求头。对于那些使用爬虫来抓取短信接口地址的程序来说，往往只能抓取到 URL，而难以分析出请求发送短信还需要的额外请求头，可以看作第一道基本防御。

**第二种方式，只有先到过注册页面才能发送验证码。**
对于普通用户来说，不管是通过 App 注册还是 H5 页面注册，一定是先进入注册页面才能看到发送验证码按钮，再点击发送。我们可以在页面或界面打开时请求固定的前置接口，为这个设备开启允许发送验证码的窗口，之后的请求发送验证码才是有效请求。

这种方式可以防御直接绕开固定流程，通过接口直接调用的发送验证码请求，并不会干扰普通用户。

**第三种方式，控制相同手机号的发送次数和发送频次。**

除非是短信无法收到，否则用户不太会请求了验证码后不完成注册流程，再重新请求。因此，我们可以限制同一手机号每天的最大请求次数。验证码的到达需要时间，太短的发送间隔没有意义，所以我们还可以控制发送的最短间隔。比如，我们可以控制相同手机号一天只能发送 10 次验证码，最短发送间隔 1 分钟。

**第四种方式，增加前置图形验证码。**
短信轰炸平台一般会收集很多免费短信接口，一个接口只会给一个用户发一次短信，所以控制相同手机号发送次数和间隔的方式不够有效。这时，我们可以考虑对用户体验稍微有影响，但也是最有效的方式作为保底，即将弹出图形验证码作为前置。

除了图形验证码，我们还可以使用其他更友好的人机验证手段（比如滑动、点击验证码等），甚至是引入比较新潮的无感知验证码方案（比如，通过判断用户输入手机号的打字节奏，来判断是用户还是机器），来改善用户体验。

此外，我们也可以考虑在监测到异常的情况下再弹出人机检测。比如，短时间内大量相同远端 IP 发送验证码的时候，才会触发人机检测。

总之，我们要确保，只有正常用户经过正常的流程才能使用开放平台资源，并且资源的用量在业务需求合理范围内。此外，还需要考虑做好短信发送量的实时监控，遇到发送量激增要及时报警。

接下来，我们一起看看限量的问题。

## 虚拟资产并不能凭空产生无限使用

虚拟资产虽然是平台方自己生产和控制，但如果生产出来可以立即使用就有立即变现的可能性。比如，因为平台 Bug 有大量用户领取高额优惠券，并立即下单使用。

在商家看来，这很可能只是一个用户支付的订单，并不会感知到用户使用平台方优惠券的情况；同时，因为平台和商家是事后结算的，所以会马上安排发货。而发货后基本就不可逆了，一夜之间造成了大量资金损失。

我们从代码层面模拟一个优惠券被刷的例子。

假设有一个 CouponCenter 类负责优惠券的产生和发放。如下是错误做法，只要调用方需要，就可以凭空产生无限的优惠券：



```

@Slf4j
public class CouponCenter {
    //用于统计发了多少优惠券
    AtomicInteger totalSent = new AtomicInteger(0);
    public void sendCoupon(Coupon coupon) {
        if (coupon != null)
            totalSent.incrementAndGet();
    }

    public int getTotalSentCoupon() {
        return totalSent.get();
    }

    //没有任何限制，来多少请求生成多少优惠券
    public Coupon generateCouponWrong(long userId, BigDecimal amount)              {
        return new Coupon(userId, amount);
    }
}
```
这样一来，使用 CouponCenter 的 generateCouponWrong 方法，想发多少优惠券就可以发多少：

```

@GetMapping("wrong")
public int wrong() {
    CouponCenter couponCenter = new CouponCenter();
    //发送10000个优惠券
    IntStream.rangeClosed(1, 10000).forEach(i -> {
        Coupon coupon = couponCenter.generateCouponWrong(1L, new BigDecimal("100"));
        couponCenter.sendCoupon(coupon);
    });
    return couponCenter.getTotalSentCoupon();
}
```
**更合适的做法是，把优惠券看作一种资源，其生产不是凭空的，而是需要事先申请**，理由是：

* 虚拟资产如果最终可以对应到真实金钱上的优惠，那么，能发多少取决于运营和财务的核算，应该是有计划、有上限的。引言提到的无门槛优惠券，需要特别小心。有门槛优惠券的大量使用至少会带来大量真实的消费，而使用无门槛优惠券下的订单，可能用户一分钱都没有支付。

* 即使虚拟资产不值钱，大量不合常规的虚拟资产流入市场，也会冲垮虚拟资产的经济体系，造成虚拟货币的极速贬值。有量的控制才有价值。

* 资产的申请需要理由，甚至需要走流程，这样才可以追溯是什么活动需要、谁提出的申请，程序依据申请批次来发放。

接下来，我们按照这个思路改进一下程序。

首先，定义一个 CouponBatch 类，要产生优惠券必须先向运营申请优惠券批次，批次中包含了固定张数的优惠券、申请原因等信息：



```

//优惠券批次
@Data
public class CouponBatch {
    private long id;
    private AtomicInteger totalCount;
    private AtomicInteger remainCount;
    private BigDecimal amount;
    private String reason;
}
```

在业务需要发放优惠券的时候，先申请批次，然后再通过批次发放优惠券：



```

@GetMapping("right")
public int right() {
    CouponCenter couponCenter = new CouponCenter();
    //申请批次    
    CouponBatch couponBatch = couponCenter.generateCouponBatch();
    IntStream.rangeClosed(1, 10000).forEach(i -> {
        Coupon coupon = couponCenter.generateCouponRight(1L, couponBatch);
        //发放优惠券
        couponCenter.sendCoupon(coupon);
    });
    return couponCenter.getTotalSentCoupon();
}
```
可以看到，generateCouponBatch 方法申请批次时，设定了这个批次包含 100 张优惠券。在通过 generateCouponRight 方法发放优惠券时，每发一次都会从批次中扣除一张优惠券，发完了就没有了：



```

public Coupon generateCouponRight(long userId, CouponBatch couponBatch) {
    if (couponBatch.getRemainCount().decrementAndGet() >= 0) {
        return new Coupon(userId, couponBatch.getAmount());
    } else {
        log.info("优惠券批次 {} 剩余优惠券不足", couponBatch.getId());
        return null;
    }
}


public CouponBatch generateCouponBatch() {
    CouponBatch couponBatch = new CouponBatch();
    couponBatch.setAmount(new BigDecimal("100"));
    couponBatch.setId(1L);
    couponBatch.setTotalCount(new AtomicInteger(100));
    couponBatch.setRemainCount(couponBatch.getTotalCount());
    couponBatch.setReason("XXX活动");
    return couponBatch;
}
```
这样改进后的程序，一个批次最多只能发放 100 张优惠券：
c971894532afd5f5150a6ab2fc0833cb.png

因为是 Demo，所以我们只是凭空 new 出来一个 Coupon。在真实的生产级代码中，一定是根据 CouponBatch 在数据库中插入一定量的 Coupon 记录，每一个优惠券都有唯一的 ID，可跟踪、可注销。
