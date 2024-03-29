为了实现过滤规则的解耦，对单个规则值对象的修改封闭，并对规则集合组成的过滤链条开放，我们在资源位过滤的领域服务中引入了责任链模式。\# 1.设计模式在外卖营销业务中的具体案例

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

从这份业务逻辑图中可以看到**返奖金额计算的规则**。首先要根据用户状态确定用户是否满足返奖条件。如果满足返奖条件，则继续判断当前用户属于新用户还是老用户，从而给予不同的奖励方案。一共涉及以下几种不同的奖励方案：

**新用户**

* 普通奖励（给予固定金额的奖励）

* 梯度奖（根据用户邀请的人数给予不同的奖励金额，邀请的人越多，奖励金额越多）

**老用户**

* 根据老用户的用户属性来计算返奖金额。为了评估不同的邀新效果，老用户返奖会存在多种返奖机制。
  计算完奖励金额以后，还需要更新用户的奖金信息，以及通知结算服务对用户的金额进行结算。这两个模块对于所有的奖励来说都是一样的。

可以看到，无论是何种用户，对于整体返奖流程是不变的，**唯一变化的是返奖规则。**此处，我们可参考**开闭原则**，对于返奖流程保持封闭，对于可能扩展的返奖规则进行开放。我们将返奖规则抽象为**返奖策略**，即针对不同用户类型的不同返奖方案，我们视为不同的**返奖策略**，不同的返奖策略会产生不同的返奖金额结果。

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

通过上文介绍的**返奖业务模型**，我们可以看到**返奖的主流程就是选择不同的返奖策略的过程，每个返奖策略都包括返奖金额计算、更新用户奖金信息、以及结算这三个步骤。**我们可以使用工厂模式生产出不同的策略，同时使用策略模式来进行不同的策略执行。首先确定我们需要生成出n种不同的返奖策略，其编码如下：

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
![img](/static/image/7.png)  
对比策略模式的类型会发现和状态模式的类图很类似，但实际上有很大的区别，具体体现在concrete class上。策略模式通过Context产生唯一一个ConcreteStrategy作用于代码中，而状态模式则是通过context组织多个ConcreteState形成一个状态转换图来实现业务逻辑。接下来，我们通过一段通用代码来解释怎么使用状态模式：

```
//定义一个抽象的状态类
public abstract class State {
    Context context;
    public void setContext(Context context) {
        this.context = context;
    }
    public abstract void handle1();
    public abstract void handle2();
}
//定义状态A
public class ConcreteStateA extends State {
    @Override
    public void handle1() {}  //本状态下必须要处理的事情

    @Override
    public void handle2() {
        super.context.setCurrentState(Context.contreteStateB);  //切换到状态B        
        super.context.handle2();  //执行状态B的任务
    }
}
//定义状态B
public class ConcreteStateB extends State {
    @Override
    public void handle2() {}  //本状态下必须要处理的事情，...

    @Override
    public void handle1() {
        super.context.setCurrentState(Context.contreteStateA);  //切换到状态A
        super.context.handle1();  //执行状态A的任务
    }
}
//定义一个上下文管理环境
public class Context {
    public final static ConcreteStateA contreteStateA = new ConcreteStateA();
    public final static ConcreteStateB contreteStateB = new ConcreteStateB();

    private State CurrentState;
    public State getCurrentState() {return CurrentState;}

    public void setCurrentState(State currentState) {
        this.CurrentState = currentState;
        this.CurrentState.setContext(this);
    }

    public void handle1() {this.CurrentState.handle1();}
    public void handle2() {this.CurrentState.handle2();}
}
//定义client执行
public class client {
    public static void main(String[] args) {
        Context context = new Context();
        context.setCurrentState(new ContreteStateA());
        context.handle1();
        context.handle2();
    }
}
```

**工程实践**

通过前文对状态模式的简介，我们可以看到当状态之间的转换在不是非常复杂的情况下，通用的状态模式存在大量的与状态无关的动作从而产生大量的无用代码。在我们的实践中，一个状态的下游不会涉及特别多的状态装换，所以我们简化了状态模式。当前的状态只负责当前状态要处理的事情，状态的流转则由第三方类负责。其实践代码如下：

```
//返奖状态执行的上下文
public class RewardStateContext {

    private RewardState rewardState;

    public void setRewardState(RewardState currentState) {this.rewardState = currentState;}
    public RewardState getRewardState() {return rewardState;}
    public void echo(RewardStateContext context, Request request) {
        rewardState.doReward(context, request);
    }
}

public abstract class RewardState {
    abstract void doReward(RewardStateContext context, Request request);
}

//待校验状态
public class OrderCheckState extends RewardState {
    @Override
    public void doReward(RewardStateContext context, Request request) {
        // 对进来的订单进行校验，判断是否用券，是否满足优惠条件等等
        orderCheck(context, request);  
    }
}

//待补偿状态
public class CompensateRewardState extends RewardState {
    @Override
    public void doReward(RewardStateContext context, Request request) {
        compensateReward(context, request);  //返奖失败，需要对用户进行返奖补偿
    }
}

//预返奖状态，待返奖状态，成功状态，失败状态(此处逻辑省略)
//..

public class InviteRewardServiceImpl {
    public boolean sendRewardForInvtee(long userId, long orderId) {
        Request request = new Request(userId, orderId);
        RewardStateContext rewardContext = new RewardStateContext();
        rewardContext.setRewardState(new OrderCheckState());
        rewardContext.echo(rewardContext, request);  //开始返奖，订单校验
        //此处的if-else逻辑只是为了表达状态的转换过程，并非实际的业务逻辑
        if (rewardContext.isResultFlag()) {  //如果订单校验成功，进入预返奖状态
            rewardContext.setRewardState(new BeforeRewardCheckState());
            rewardContext.echo(rewardContext, request);
        } else {//如果订单校验失败，进入返奖失败流程，...
            rewardContext.setRewardState(new RewardFailedState());
            rewardContext.echo(rewardContext, request);
            return false;
        }
        if (rewardContext.isResultFlag()) {//预返奖检查成功，进入待返奖流程，...
            rewardContext.setRewardState(new SendRewardState());
            rewardContext.echo(rewardContext, request);
        } else {  //如果预返奖检查失败，进入返奖失败流程，...
            rewardContext.setRewardState(new RewardFailedState());
            rewardContext.echo(rewardContext, request);
            return false;
        }
        if (rewardContext.isResultFlag()) {  //返奖成功，进入返奖结束流程，...
            rewardContext.setRewardState(new RewardSuccessState());
            rewardContext.echo(rewardContext, request);
        } else {  //返奖失败，进入返奖补偿阶段，...
            rewardContext.setRewardState(new CompensateRewardState());
            rewardContext.echo(rewardContext, request);
        }
        if (rewardContext.isResultFlag()) {  //补偿成功，进入返奖完成阶段，...
            rewardContext.setRewardState(new RewardSuccessState());
            rewardContext.echo(rewardContext, request);
        } else {  //补偿失败，仍然停留在当前态，直至补偿成功（或多次补偿失败后人工介入处理）
            rewardContext.setRewardState(new CompensateRewardState());
            rewardContext.echo(rewardContext, request);
        }
        return true;
    }
}
```

状态模式的核心是封装，**将状态以及状态转换逻辑封装到类的内部来实现**，也很好的体现了“**开闭原则**”和“**单一职责原则**”。每一个状态都是一个子类，不管是修改还是增加状态，只需要修改或者增加一个子类即可。在我们的应用场景中，状态数量以及状态转换远比上述例子复杂，通过“**状态模式**”避免了大量的if-else代码，让我们的逻辑变得更加清晰。同时由于状态模式的良好的封装性以及遵循的设计原则，让我们在复杂的业务场景中，能够游刃有余地管理各个状态。

## 1.3.**点评外卖投放系统中设计模式的实践**

**业务简介**

继续举例，点评App的外卖频道中会预留多个资源位为营销使用，向用户展示一些比较精品美味的外卖食品，为了增加用户点外卖的意向。当用户点击点评首页的“美团外卖”入口时，资源位开始加载，会通过一些规则来筛选出合适的展示Banner。

![img](/static/image/7.png)

**业务建模**

对于投放业务，就是要在这些资源位中展示符合当前用户的资源。其流程如下图所示：

![img](/static/image/8.jpg)

从流程中我们可以看到，首先运营人员会配置需要展示的资源，以及对资源进行过滤的规则。我们资源的过滤规则相对灵活多变，这里体现为三点：

1.过滤规则大部分可重用，但也会有扩展和变更。

2.不同资源位的过滤规则和过滤顺序是不同的。

3.同一个资源位由于业务所处的不同阶段，过滤规则可能不同。

过滤规则本身是一个个的**值对象**，我们通过领域服务的方式，操作这些规则值对象完成资源位的过滤逻辑。下图介绍了资源位在进行用户特征相关规则过滤时的过程：

![img](/static/image/10.png)

为了实现**过滤规则**的解耦，对单个规则值对象的修改封闭，并对规则集合组成的过滤链条开放，我们在资源位过滤的领域服务中引入了责任链模式。

#### 模式：责任链模式

模式定义：使多个对象都有机会处理请求，从而避免了请求的**发送者**和**接受者**之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有对象处理它为止。

责任链模式通用类图如下：

![img](/static/image/11.png)

我们通过一段比较通用的代码来解释如何使用责任链模式：

```
//定义一个抽象的handle
public abstract class Handler {
    private Handler nextHandler;  //指向下一个处理者
    private int level;  //处理者能够处理的级别

    public Handler(int level) {
        this.level = level;
    }

    public void setNextHandler(Handler handler) {
        this.nextHandler = handler;
    }

    // 处理请求传递，注意final，子类不可重写
    public final void handleMessage(Request request) {
        if (level == request.getRequstLevel()) {
            this.echo(request);
        } else {
            if (this.nextHandler != null) {
                this.nextHandler.handleMessage(request);
            } else {
                System.out.println("已经到最尽头了");
            }
        }
    }
    // 抽象方法，子类实现
    public abstract void echo(Request request);
}

// 定义一个具体的handleA
public class HandleRuleA extends Handler {
    public HandleRuleA(int level) {
        super(level);
    }
    @Override
    public void echo(Request request) {
        System.out.println("我是处理者1,我正在处理A规则");
    }
}

//定义一个具体的handleB
public class HandleRuleB extends Handler {}  //...

//客户端实现
class Client {
    public static void main(String[] args) {
        HandleRuleA handleRuleA = new HandleRuleA(1);
        HandleRuleB handleRuleB = new HandleRuleB(2);
        handleRuleA.setNextHandler(handleRuleB);  //这是重点，将handleA和handleB串起来
        handleRuleA.echo(new Request());
    }
}
```

**工程实践**  
下面通过代码向大家展示如何实现这一套流程：

```
//定义一个抽象的规则
public abstract class BasicRule<CORE_ITEM, T extends RuleContext<CORE_ITEM>>{
    //有两个方法，evaluate用于判断是否经过规则执行，execute用于执行具体的规则内容。
    public abstract boolean evaluate(T context);
    public abstract void execute(T context) {
}

//定义所有的规则具体实现
//规则1：判断服务可用性
public class ServiceAvailableRule extends BasicRule<UserPortrait, UserPortraitRuleContext> {
    @Override
    public boolean evaluate(UserPortraitRuleContext context) {
        TakeawayUserPortraitBasicInfo basicInfo = context.getBasicInfo();
        if (basicInfo.isServiceFail()) {
              return false;
        }
        return true;
    }

    @Override
    public void execute(UserPortraitRuleContext context) {}

}
//规则2：判断当前用户属性是否符合当前资源位投放的用户属性要求
public class UserGroupRule extends BasicRule<UserPortrait, UserPortraitRuleContext> {
    @Override
    public boolean evaluate(UserPortraitRuleContext context) {}

    @Override
    public void execute(UserPortraitRuleContext context) {
        UserPortrait userPortraitPO = context.getData();
        if(userPortraitPO.getUserGroup() == context.getBasicInfo().getUserGroup().code) {
          context.setValid(true);
        } else {
          context.setValid(false);
        }
    }
}

//规则3：判断当前用户是否在投放城市，具体逻辑省略
public class CityInfoRule extends BasicRule<UserPortrait, UserPortraitRuleContext> {}
//规则4：根据用户的活跃度进行资源过滤，具体逻辑省略
public class UserPortraitRule extends BasicRule<UserPortrait, UserPortraitRuleContext> {} 

//我们通过spring将这些规则串起来组成一个一个请求链
    "serviceAvailableRule" class="com.dianping.takeaway.ServiceAvailableRule"/>
    "userGroupValidRule" class="com.dianping.takeaway.UserGroupRule"/>
    "cityInfoValidRule" class="com.dianping.takeaway.CityInfoRule"/>
    "userPortraitRule" class="com.dianping.takeaway.UserPortraitRule"/>

    "userPortraitRuleChain" value-type="com.dianping.takeaway.Rule">
        "serviceAvailableRule"/>
        "userGroupValidRule"/>
        "cityInfoValidRule"/>
        "userPortraitRule"/>


//规则执行
public class DefaultRuleEngine{
    @Autowired
    List userPortraitRuleChain;

    public void invokeAll(RuleContext ruleContext) {
        for(Rule rule : userPortraitRuleChain) {
            rule.evaluate(ruleContext)
        }
    }
}
```

责任链模式最重要的优点就是解耦，将客户端与处理者分开，客户端不需要了解是哪个处理者对事件进行处理，处理者也不需要知道处理的整个流程。

在我们的系统中，后台的过滤规则会经常变动，规则和规则之间可能也会存在传递关系，通过责任链模式，我们将规则与规则分开，将规则与规则之间的传递关系通过Spring注入到List中，形成一个链的关系。当增加一个规则时，只需要实现BasicRule接口，然后将新增的规则按照顺序加入Spring中即可。当删除时，只需删除相关规则即可，不需要考虑代码的其他逻辑。从而显著地提高了代码的灵活性，提高了代码的开发效率，同时也保证了系统的稳定性。

# 3.总结

本文从营销业务出发，介绍了领域模型到代码工程之间的转化，从DDD引出了设计模式，详细介绍了工厂方法模式、策略模式、责任链模式以及状态模式这四种模式在营销业务中的具体实现。

除了这四种模式以外，我们的代码工程中还大量使用了代理模式、单例模式、适配器模式等等，例如在我们对DDD防腐层的实现就使用了适配器模式，通过适配器模式屏蔽了业务逻辑与第三方服务的交互。因篇幅原因，这里不再进行过多的阐述。

对于营销业务来说，业务策略多变导致需求多变是我们面临的主要问题。如何应对复杂多变的需求，是我们提炼领域模型和实现代码模型时必须要考虑的内容。DDD以及设计模式提供了一套相对完整的方法论帮助我们完成了领域建模及工程实现。其实，设计模式就像一面镜子，将领域模型映射到代码模型中，切实地提高代码的复用性、可扩展性，也提高了系统的可维护性。

当然，设计模式只是软件开发领域内多年来的经验总结，任何一个或简单或复杂的设计模式都会遵循上述的七大设计原则，只要大家真正理解了七大设计原则，设计模式对我们来说应该就不再是一件难事。但是，使用设计模式也不是要求我们循规蹈矩，只要我们的代码模型设计遵循了上述的七大原则，我们会发现原来我们的设计中就已经使用了某种设计模式。

# 4.参考

软件设计模式-百度百科

快速理解-设计模式六大原则：  
[https://www.jianshu.com/p/807bc228dbc2](https://www.jianshu.com/p/807bc228dbc2)  
Software design pattern：  
[https://en.wikipedia.org/wiki/Software\_design\_pattern](https://en.wikipedia.org/wiki/Software_design_pattern)

《设计模式之禅》，秦小波，机械工业出版社

《领域驱动设计-软件核心复杂性应对之道》，Eric Evans，人民邮电出版社。  
领域驱动设计（DDD）在互联网业务系统的实践:  
[https://mp.weixin.qq.com/s?\_\_biz=MjM5NjQ5MTI5OA==∣=2651747236&idx=1&sn=baf67052ec1961c3c6de1af26fba9b22&chksm=bd12aae98a6523ff90b3461d00fee548554fdeb2112b541de87d0c59dea45bc60d2f5211d6a6&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651747236&idx=1&sn=baf67052ec1961c3c6de1af26fba9b22&chksm=bd12aae98a6523ff90b3461d00fee548554fdeb2112b541de87d0c59dea45bc60d2f5211d6a6&scene=21#wechat_redirect)

美团下一代服务治理系统 OCTO2.0 的探索与实践:  
[https://mp.weixin.qq.com/s?\_\_biz=MjM5NjQ5MTI5OA==∣=2651751158&idx=1&sn=c01a900ae4cef7decf3acfbaad62168f&chksm=bd125bbb8a65d2ad4a896e5ec2dc366be198da09bc04dbfedc397e3821d66ef89d70ed6bc49e&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=2651751158&idx=1&sn=c01a900ae4cef7decf3acfbaad62168f&chksm=bd125bbb8a65d2ad4a896e5ec2dc366be198da09bc04dbfedc397e3821d66ef89d70ed6bc49e&scene=21#wechat_redirect)

数据驱动精准化营销在大众点评的实践:  
[https://mp.weixin.qq.com/s?\_\_biz=MjM5NjQ5MTI5OA==∣=404261497&idx=1&sn=1c7628e36701d4ceee6f0f651fa7d1d3&chksm=3b0453340c73da223144e7f8825caff3355bb60a95c0e33804f39a9476bd4875b80fc5a259f4&scene=21\#wechat\_redirect](https://mp.weixin.qq.com/s?__biz=MjM5NjQ5MTI5OA==&mid=404261497&idx=1&sn=1c7628e36701d4ceee6f0f651fa7d1d3&chksm=3b0453340c73da223144e7f8825caff3355bb60a95c0e33804f39a9476bd4875b80fc5a259f4&scene=21#wechat_redirect)

原文：[https://zyl.me/blog/2162](https://zyl.me/blog/2162)

