# 1.设计模式在外卖营销业务中的具体案例

## 1.1.**“邀请下单”业务中设计模式的实践**

### 1.1.1.**业务简介**

“邀请下单”是美团外卖用户邀请其他用户下单后给予奖励的平台。即用户A邀请用户B，并且用户B在美团下单后，给予用户A一定的现金奖励（以下简称返奖）。同时为了协调成本与收益的关系，返奖会有多个计算策略。邀请下单后台主要涉及两个技术要点：

* 返奖金额的计算，涉及到不同的计算规则。

* 从邀请开始到返奖结束的整个流程。

![img](/static/image/1.jpg)

### 1.1.2.返奖规则与设计模式实践

**业务建模**

如图是返奖规则计算的业务逻辑视图：

![img](/static/image/2.png)

从这份业务逻辑图中可以看到返奖金额计算的规则。首先要根据用户状态确定用户是否满足返奖条件。如果满足返奖条件，则继续判断当前用户属于新用户还是老用户，从而给予不同的奖励方案。一共涉及以下几种不同的奖励方案：

**新用户**

* 普通奖励（给予固定金额的奖励）

* 梯度奖（根据用户邀请的人数给予不同的奖励金额，邀请的人越多，奖励金额越多）

**老用户**

* 根据老用户的用户属性来计算返奖金额。为了评估不同的邀新效果，老用户返奖会存在多种返奖机制。
  计算完奖励金额以后，还需要更新用户的奖金信息，以及通知结算服务对用户的金额进行结算。这两个模块对于所有的奖励来说都是一样的。

可以看到，无论是何种用户，对于整体返奖流程是不变的，唯一变化的是返奖规则。此处，我们可参考**开闭原则**，对于返奖流程保持封闭，对于可能扩展的返奖规则进行开放。我们将返奖规则抽象为**返奖策略**，即针对不同用户类型的不同返奖方案，我们视为不同的**返奖策略**，不同的返奖策略会产生不同的返奖金额结果。

在我们的领域模型里，返奖策略是一个**值对象**，我们通过工厂的方式生产针对不同用户的奖励策略值对象。下文我们将介绍以上领域模型的工程实现，即**工厂模式**和**策略模式**的实际应用。

#### 模式：工厂模式

工厂模式又细分为工厂方法模式和抽象工厂模式，本文主要介绍工厂方法模式。  
**模式定义：**定义一个用于创建对象的接口，让子类决定实例化哪一个类。工厂方法是一个类的实例化延迟到其子类。  
工厂模式通用类图如下:  
![img](/static/image/3.png)  
我们通过一段较为通用的代码来解释如何使用工厂模式：

```
//抽象的产品
public abstract class Product {
    public abstract void method();
}
//定义一个具体的产品 (可以定义多个具体的产品)
class ProductA extends Product {
    @Override
    public void method() {}  //具体的执行逻辑
}
//抽象的工厂
abstract class Factory<T> {
    abstract Product createProduct(Class c);
}
//具体的工厂可以生产出相应的产品
class FactoryA extends Factory{
    @Override
    Product createProduct(Class c) {
        Product product = (Product) Class.forName(c.getName()).newInstance();
        return product;
    }
}
```

#### 模式：策略模式

**模式定义**：定义一系列算法，将每个算法都封装起来，并且它们可以互换。策略模式是一种对象行为模式。

策略模式通用类图如下:

![img](/static/image/4.png)

```
//定义一个策略接口
public interface Strategy {
    void strategyImplementation();
}

//具体的策略实现(可以定义多个具体的策略实现)
public class StrategyA implements Strategy{
    @Override
    public void strategyImplementation() {
        System.out.println("正在执行策略A");
    }
}

//封装策略，屏蔽高层模块对策略、算法的直接访问，屏蔽可能存在的策略变化
public class Context {
    private Strategy strategy = null;

    public Context(Strategy strategy) {
        this.strategy = strategy;
    }

    public void doStrategy() {
        strategy.strategyImplementation();
    }
}
```

**工程实践**

通过上文介绍的返奖业务模型，我们可以看到返奖的主流程就是选择不同的返奖策略的过程，每个返奖策略都包括返奖金额计算、更新用户奖金信息、以及结算这三个步骤。我们可以使用工厂模式生产出不同的策略，同时使用策略模式来进行不同的策略执行。首先确定我们需要生成出n种不同的返奖策略，其编码如下：

```
//抽象策略
public abstract class RewardStrategy {
    public abstract void reward(long userId);

    public void insertRewardAndSettlement(long userId, int reward) {} ; //更新用户信息以及结算
}
//新用户返奖具体策略A
public class newUserRewardStrategyA extends RewardStrategy {
    @Override
    public void reward(long userId) {}  //具体的计算逻辑，...
}

//老用户返奖具体策略A
public class OldUserRewardStrategyA extends RewardStrategy {
    @Override
    public void reward(long userId) {}  //具体的计算逻辑，...
}

//抽象工厂
public abstract class StrategyFactory<T> {
    abstract RewardStrategy createStrategy(Class c);
}

//具体工厂创建具体的策略
public class FactorRewardStrategyFactory extends StrategyFactory {
    @Override
    RewardStrategy createStrategy(Class c) {
        RewardStrategy product = null;
        try {
            product = (RewardStrategy) Class.forName(c.getName()).newInstance();
        } catch (Exception e) {}
        return product;
    }
}
```

通过工厂模式生产出具体的策略之后，根据我们之前的介绍，很容易就可以想到使用策略模式来执行我们的策略。具体代码如下：

```
public class RewardContext {
    private RewardStrategy strategy;

    public RewardContext(RewardStrategy strategy) {
        this.strategy = strategy;
    }

    public void doStrategy(long userId) { 
        int rewardMoney = strategy.reward(userId);
        insertRewardAndSettlement(long userId, int reward) {
          insertReward(userId, rewardMoney);
          settlement(userId);
       }  
    }
}
```

接下来我们将工厂模式和策略模式结合在一起，就完成了整个返奖的过程

```
public class InviteRewardImpl {
    //返奖主流程
    public void sendReward(long userId) {
        FactorRewardStrategyFactory strategyFactory = new FactorRewardStrategyFactory();  //创建工厂
        Invitee invitee = getInviteeByUserId(userId);  //根据用户id查询用户信息
        if (invitee.userType == UserTypeEnum.NEW_USER) {  //新用户返奖策略
            NewUserBasicReward newUserBasicReward = (NewUserBasicReward) strategyFactory.createStrategy(NewUserBasicReward.class);
            RewardContext rewardContext = new RewardContext(newUserBasicReward);
            rewardContext.doStrategy(userId); //执行返奖策略
        }if(invitee.userType == UserTypeEnum.OLD_USER){}  //老用户返奖策略，... 
    }
}
```

工厂方法模式帮助我们直接产生一个具体的策略对象，策略模式帮助我们保证这些策略对象可以自由地切换而不需要改动其他逻辑，从而达到解耦的目的。通过这两个模式的组合，当我们系统需要增加一种返奖策略时，只需要实现RewardStrategy接口即可，无需考虑其他的改动。当我们需要改变策略时，只要修改策略的类名即可。不仅增强了系统的可扩展性，避免了大量的条件判断，而且从真正意义上达到了高内聚、低耦合的目的。

## 1.2.返奖流程与设计模式实践

**业务建模**

当受邀人在接受邀请人的邀请并且下单后，返奖后台接收到受邀人的下单记录，此时邀请人也进入返奖流程。首先我们订阅用户订单消息并对订单进行返奖规则校验。例如，是否使用红包下单，是否在红包有效期内下单，订单是否满足一定的优惠金额等等条件。当满足这些条件以后，我们将订单信息放入延迟队列中进行后续处理。经过T+N天之后处理该延迟消息，判断用户是否对该订单进行了退款，如果未退款，对用户进行返奖。若返奖失败，后台还有返奖补偿流程，再次进行返奖。其流程如下图所示：  
![img](/static/image/5.png)  
我们对上述业务流程进行领域建模：

1.在接收到订单消息后，用户进入待校验状态；

2.在校验后，若校验通过，用户进入预返奖状态，并放入延迟队列。若校验未通过，用户进入不返奖状态，结束流程；

3.T+N天后，处理延迟消息，若用户未退款，进入待返奖状态。若用户退款，进入失败状态，结束流程；

4.执行返奖，若返奖成功，进入完成状态，结束流程。若返奖不成功，进入待补偿状态；

5.待补偿状态的用户会由任务定期触发补偿机制，直至返奖成功，进入完成状态，保障流程结束。

![img](/static/image/6.png)

可以看到，我们通过建模将返奖流程的多个步骤映射为系统的状态。对于系统状态的表述，DDD中常用到的概念是领域事件，另外也提及过事件溯源的实践方案。当然，在设计模式中，也有一种能够表述系统状态的代码模型，那就是状态模式。在邀请下单系统中，我们的主要流程是返奖。对于返奖，每一个状态要进行的动作和操作都是不同的。因此，使用状态模式，能够帮助我们对系统状态以及状态间的流转进行统一的管理和扩展。

#### **模式：状态模式**

**模式定义**：当一个对象内在状态改变时允许其改变行为，这个对象看起来像改变了其类。

状态模式的通用类图如下图所示：



# 2.怎么使用

# 3.总结

# 4.参考

[https://zyl.me/blog/2162](https://zyl.me/blog/2162)

