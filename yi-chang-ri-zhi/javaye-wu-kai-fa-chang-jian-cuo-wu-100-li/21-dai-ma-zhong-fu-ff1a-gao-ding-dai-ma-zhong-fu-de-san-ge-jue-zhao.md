# 1.代码重复：搞定代码重复的三个绝招

今天，我来和你聊聊搞定代码重复的三个绝招。

业务同学抱怨业务开发没有技术含量，用不到设计模式、Java 高级特性、OOP，平时写代码都在堆 CRUD，个人成长无从谈起。每次面试官问到“请说说平时常用的设计模式”，都只能答单例模式，因为其他设计模式的确是听过但没用过；对于反射、注解之类的高级特性，也只是知道它们在写框架的时候非常常用，但自己又不写框架代码，没有用武之地。

其实，我认为不是这样的。设计模式、OOP 是前辈们在大型项目中积累下来的经验，通过这些方法论来改善大型项目的可维护性。反射、注解、泛型等高级特性在框架中大量使用的原因是，框架往往需要以同一套算法来应对不同的数据结构，而这些特性可以帮助减少重复代码，提升项目可维护性。

在我看来，可维护性是大型项目成熟度的一个重要指标，而提升可维护性非常重要的一个手段就是减少代码重复。那为什么这样说呢？
* 果多处重复代码实现完全相同的功能，很容易修改一处忘记修改另一处，造成 Bug；
* 有一些代码并不是完全重复，而是相似度很高，修改这些类似的代码容易改（复制粘贴）错，把原本有区别的地方改为了一样。

今天，我就从业务代码中最常见的三个需求展开，和你聊聊如何使用 Java 中的一些高级特性、设计模式，以及一些工具消除重复代码，才能既优雅又高端。通过今天的学习，也希望改变你对业务代码没有技术含量的看法。

## 利用工厂模式 + 模板方法模式，消除 if…else 和重复代码

假设要开发一个购物车下单的功能，针对不同用户进行不同处理：

* 普通用户需要收取运费，运费是商品价格的 10%，无商品折扣；

* VIP 用户同样需要收取商品价格 10% 的快递费，但购买两件以上相同商品时，第三件开始享受一定折扣；

* 内部用户可以免运费，无商品折扣。


我们的目标是实现三种类型的购物车业务逻辑，把入参 Map 对象（Key 是商品 ID，Value 是商品数量），转换为出参购物车类型 Cart。

先实现针对普通用户的购物车处理逻辑：


```

//购物车
@Data
public class Cart {
    //商品清单
    private List<Item> items = new ArrayList<>();
    //总优惠
    private BigDecimal totalDiscount;
    //商品总价
    private BigDecimal totalItemPrice;
    //总运费
    private BigDecimal totalDeliveryPrice;
    //应付总价
    private BigDecimal payPrice;
}
//购物车中的商品
@Data
public class Item {
    //商品ID
    private long id;
    //商品数量
    private int quantity;
    //商品单价
    private BigDecimal price;
    //商品优惠
    private BigDecimal couponPrice;
    //商品运费
    private BigDecimal deliveryPrice;
}
//普通用户购物车处理
public class NormalUserCart {
    public Cart process(long userId, Map<Long, Integer> items) {
        Cart cart = new Cart();

        //把Map的购物车转换为Item列表
        List<Item> itemList = new ArrayList<>();
        items.entrySet().stream().forEach(entry -> {
            Item item = new Item();
            item.setId(entry.getKey());
            item.setPrice(Db.getItemPrice(entry.getKey()));
            item.setQuantity(entry.getValue());
            itemList.add(item);
        });
        cart.setItems(itemList);

        //处理运费和商品优惠
        itemList.stream().forEach(item -> {
            //运费为商品总价的10%
            item.setDeliveryPrice(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())).multiply(new BigDecimal("0.1")));
            //无优惠
            item.setCouponPrice(BigDecimal.ZERO);
        });

        //计算商品总价
        cart.setTotalItemPrice(cart.getItems().stream().map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity()))).reduce(BigDecimal.ZERO, BigDecimal::add));
        //计算运费总价
        cart.setTotalDeliveryPrice(cart.getItems().stream().map(Item::getDeliveryPrice).reduce(BigDecimal.ZERO, BigDecimal::add));
        //计算总优惠
        cart.setTotalDiscount(cart.getItems().stream().map(Item::getCouponPrice).reduce(BigDecimal.ZERO, BigDecimal::add));
        //应付总价=商品总价+运费总价-总优惠
        cart.setPayPrice(cart.getTotalItemPrice().add(cart.getTotalDeliveryPrice()).subtract(cart.getTotalDiscount()));
        return cart;
    }
}
```

然后实现针对 VIP 用户的购物车逻辑。与普通用户购物车逻辑的不同在于，VIP 用户能享受同类商品多买的折扣。所以，这部分代码只需要额外处理多买折扣部分：



```

public class VipUserCart {


    public Cart process(long userId, Map<Long, Integer> items) {
        ...


        itemList.stream().forEach(item -> {
            //运费为商品总价的10%
            item.setDeliveryPrice(item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity())).multiply(new BigDecimal("0.1")));
            //购买两件以上相同商品，第三件开始享受一定折扣
            if (item.getQuantity() > 2) {
                item.setCouponPrice(item.getPrice()
                        .multiply(BigDecimal.valueOf(100 - Db.getUserCouponPercent(userId)).divide(new BigDecimal("100")))
                       .multiply(BigDecimal.valueOf(item.getQuantity() - 2)));
            } else {
                item.setCouponPrice(BigDecimal.ZERO);
            }
        });


        ...
        return cart;
    }
}
```

最后是免运费、无折扣的内部用户，同样只是处理商品折扣和运费时的逻辑差异：



```

public class InternalUserCart {


    public Cart process(long userId, Map<Long, Integer> items) {
        ...

        itemList.stream().forEach(item -> {
            //免运费
            item.setDeliveryPrice(BigDecimal.ZERO);
            //无优惠
            item.setCouponPrice(BigDecimal.ZERO);
        });

        ...
        return cart;
    }
}
```
对比一下代码量可以发现，三种购物车 70% 的代码是重复的。原因很简单，虽然不同类型用户计算运费和优惠的方式不同，但整个购物车的初始化、统计总价、总运费、总优惠和支付价格的逻辑都是一样的。


正如我们开始时提到的，代码重复本身不可怕，可怕的是漏改或改错。比如，写 VIP 用户购物车的同学发现商品总价计算有 Bug，不应该是把所有 Item 的 price 加在一起，而是应该把所有 Item 的 price*quantity 加在一起。这时，他可能会只修改 VIP 用户购物车的代码，而忽略了普通用户、内部用户的购物车中，重复的逻辑实现也有相同的 Bug。

有了三个购物车后，我们就需要根据不同的用户类型使用不同的购物车了。如下代码所示，使用三个 if 实现不同类型用户调用不同购物车的 process 方法：



```

@GetMapping("wrong")
public Cart wrong(@RequestParam("userId") int userId) {
    //根据用户ID获得用户类型
    String userCategory = Db.getUserCategory(userId);
    //普通用户处理逻辑
    if (userCategory.equals("Normal")) {
        NormalUserCart normalUserCart = new NormalUserCart();
        return normalUserCart.process(userId, items);
    }
    //VIP用户处理逻辑
    if (userCategory.equals("Vip")) {
        VipUserCart vipUserCart = new VipUserCart();
        return vipUserCart.process(userId, items);
    }
    //内部用户处理逻辑
    if (userCategory.equals("Internal")) {
        InternalUserCart internalUserCart = new InternalUserCart();
        return internalUserCart.process(userId, items);
    }

    return null;
}
```

电商的营销玩法是多样的，以后势必还会有更多用户类型，需要更多的购物车。我们就只能不断增加更多的购物车类，一遍一遍地写重复的购物车逻辑、写更多的 if 逻辑吗？

当然不是，相同的代码应该只在一处出现！

如果我们熟记抽象类和抽象方法的定义的话，这时或许就会想到，是否可以把重复的逻辑定义在抽象类中，三个购物车只要分别实现不同的那份逻辑呢？

其实，这个模式就是**模板方法模式**。我们在父类中实现了购物车处理的流程模板，然后把需要特殊处理的地方留空白也就是留抽象方法定义，让子类去实现其中的逻辑。由于父类的逻辑不完整无法单独工作，因此需要定义为抽象类。


如下代码所示，AbstractCart 抽象类实现了购物车通用的逻辑，额外定义了两个抽象方法让子类去实现。其中，processCouponPrice 方法用于计算商品折扣，processDeliveryPrice 方法用于计算运费。


```

public abstract class AbstractCart {
    //处理购物车的大量重复逻辑在父类实现
    public Cart process(long userId, Map<Long, Integer> items) {

        Cart cart = new Cart();

        List<Item> itemList = new ArrayList<>();
        items.entrySet().stream().forEach(entry -> {
            Item item = new Item();
            item.setId(entry.getKey());
            item.setPrice(Db.getItemPrice(entry.getKey()));
            item.setQuantity(entry.getValue());
            itemList.add(item);
        });
        cart.setItems(itemList);
        //让子类处理每一个商品的优惠
        itemList.stream().forEach(item -> {
            processCouponPrice(userId, item);
            processDeliveryPrice(userId, item);
        });
        //计算商品总价
        cart.setTotalItemPrice(cart.getItems().stream().map(item -> item.getPrice().multiply(BigDecimal.valueOf(item.getQuantity()))).reduce(BigDecimal.ZERO, BigDecimal::add));
        //计算总运费
cart.setTotalDeliveryPrice(cart.getItems().stream().map(Item::getDeliveryPrice).reduce(BigDecimal.ZERO, BigDecimal::add));
        //计算总折扣
cart.setTotalDiscount(cart.getItems().stream().map(Item::getCouponPrice).reduce(BigDecimal.ZERO, BigDecimal::add));
        //计算应付价格
cart.setPayPrice(cart.getTotalItemPrice().add(cart.getTotalDeliveryPrice()).subtract(cart.getTotalDiscount()));
        return cart;
    }

    //处理商品优惠的逻辑留给子类实现
    protected abstract void processCouponPrice(long userId, Item item);
    //处理配送费的逻辑留给子类实现
    protected abstract void processDeliveryPrice(long userId, Item item);
}
```

有了这个抽象类，三个子类的实现就非常简单了。普通用户的购物车 NormalUserCart，实现的是 0 优惠和 10% 运费的逻辑：


```

@Service(value = "NormalUserCart")
public class NormalUserCart extends AbstractCart {

    @Override
    protected void processCouponPrice(long userId, Item item) {
        item.setCouponPrice(BigDecimal.ZERO);
    }

    @Override
    protected void processDeliveryPrice(long userId, Item item) {
        item.setDeliveryPrice(item.getPrice()
                .multiply(BigDecimal.valueOf(item.getQuantity()))
                .multiply(new BigDecimal("0.1")));
    }
}
```

VIP 用户的购物车 VipUserCart，直接继承了 NormalUserCart，只需要修改多买优惠策略：



```

@Service(value = "VipUserCart")
public class VipUserCart extends NormalUserCart {

    @Override
    protected void processCouponPrice(long userId, Item item) {
        if (item.getQuantity() > 2) {
            item.setCouponPrice(item.getPrice()
                    .multiply(BigDecimal.valueOf(100 - Db.getUserCouponPercent(userId)).divide(new BigDecimal("100")))
                    .multiply(BigDecimal.valueOf(item.getQuantity() - 2)));
        } else {
            item.setCouponPrice(BigDecimal.ZERO);
        }
    }
}
```
内部用户购物车 InternalUserCart 是最简单的，直接设置 0 运费和 0 折扣即可：

# 2.总结

## 2.1.思考题

## 2.2.高质量问题



