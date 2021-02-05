# 如何优化你的if-else？来试试“责任树模式”

`简介：写业务逻辑时，if-else 可能是最容易想到的逻辑方式了。然而大量堆砌的 if-else 毫无疑问将给代码维护带来巨大的困难。如何优化这些 if-else 呢？本文分享一种设计模式——责任树模式，通过将责任链与策略模式融合，成为一种广义的责任链模式，不仅可以完成任务的逐级委托，也可以在任一级选择不同的下游策略进行处理，并将责任树模式抽象出一个通用的框架。`

## 一、问题背景

最近开发了一个需求，该接口需要根据 p1、p2、p3、version 多个入参的不同组合按照其对应的业务策略给出结果数据。由于该接口已经开发了三期了，每次开发新一期的需求时为了兼容老的业务逻辑，大家都倾向于不删不改只新增，因此这块代码已经产生了一些“坏味道”，函数入口通过不断添加“卫语句”判断 version 的方式跳转到新一期的业务逻辑方法中，而每一期的业务逻辑也是通过 p1、p2、p3 的 if-else 组合形成不同的分支逻辑。这已经是我简化后的表述，总之刚开始对于我这个新同学来说，梳理这块业务代码着实花了一些功夫。

![](/static/image/微信图片_20210203113857.gif)

而且，这块逻辑相当于是一个业务上的通用能力，未来一定还会有五期、六期、N 期的需求进来，入参的取值也会不断拓展，因此以现有方式膨胀下去只会“坏味道”会越来越重。

总结一下，当前场景面临的问题是：

* 如何解决接口升级，在保证兼容老版本的情况下轻松开发新版本业务逻辑？

* 如何根据入参 p1、p2、p3 等的不同组合进行策略定位？

## 二、解决思路

在思考解决方案时，很容易想到两种可以优化类似场景的设计模式：责任链模式和策略模式。

1、责任链模式

责任链模式是实现了类似“流水线”结构的逐级处理，通常是一条链式结构，将“抽象处理者”的不同实现串联起来：如果当前节点能够处理任务则直接处理掉，如果无法处理则委托给责任链的下一个节点，如此往复直到有节点可以处理这个任务。

我们可以通过责任链模式完成对不同 version 业务逻辑隔离的处理，比如节点 1 处理 version = 1 的请求，节点 2 处理 version = 2 的请求等等。但问题在于我们遇到的场景还需要根据一定策略，路由到不同的下游节点进行处理。这就是策略模式擅长解决的问题了。

![](/static/image/微信图片_20210203110857.png)

2、策略模式

策略模式的目的是将算法的使用与定义解耦，能够实现根据规则路由到不同策略类进行处理。

我们可以通过策略模式解决根据不同参数组合执行不同业务逻辑的场景。但是我们的场景仅仅通过一层策略路由无法满足任务处理需求。请求的分层处理又是责任链模式所擅长的了。  
![](/static/image/微信图片_20210203111658.png)

可以看到，两种设计模式都不完全符合目前这个场景：责任链模式可以实现逐级委托，但每一级又不能像策略模式那样路由到不同的处理者上；策略模式通常只有一层路由，不易实现多个参数的策略组合。

因此我们自然而然地可以想到：是不是可以将两种模式结合起来？

3、广义责任链模式——责任树模式

将责任链与策略模式融合，即成为了一种广义的责任链模式，我简称为“责任树模式”。这种模式不仅可以完成任务的逐级委托，也可以在任一级选择不同的下游策略进行处理。

![](/static/image/微信图片_20210203111729.png)

那么问题来了，如何通过责任树模式解决前面我们遇到的问题呢？

首先看如何解决第一个问题，新老接口的隔离和兼容：可以将接口每个版本的逻辑作为一个责任树上第一层的不同实现，如分别对应上图中的 Strategy1、Strategy2、Strategy3 节点。这样在接口入口，就首先把策略路由到不同的分支上去。如果没有节点命中，则不再向下游委托直接返回错误。

然后第二个问题，参数的组合定位到不同的策略实现上：同样的思路，一个参数对应责任树上的一层的路由，将该参数的不同取值路由到下一层的不同实现即可，这样逐级委托，后面新增入参的枚举值、甚至再拓展新的入参都可以非常方便地进行拓展。

## 三、优化收益

将这块业务通过“责任树模式”重构之后，可以收获以下几个收益点：

后续迭代人力成本降低。

代码结构更清晰，可维护性提升：没有了各种“卫语句”的跳转 & 维护性巨差的巨型方法，函数可以收敛在理想的 50 行内。

后续新增需求修改代码不易出错：策略间隔离，不需要完整看一遍大函数理清逻辑再修改，只需要无脑添加一条路由 + 新的策略实现方法即可。

问题易定位：同样由于策略间隔离，调试时可以直接定位到指定策略的业务逻辑代码，不需要逐句排查。

相信有开发经验的同学应该都有体会，即使是自己写过的代码，一阵子不看也会忘掉，等到再有修改时，还要顺着代码理一遍逻辑，如果文档、注释没写好，那就更加酸爽了。因此，将巨型函数拆分解耦非常重要。

## 四、抽象框架

虽然通过“责任树模式”解决了我这个需求开发中遇到的问题，但是类似的问题还是普遍存在的。本着助（shǎo）人（zào）为（lún）乐（zi）的精神，我更进一步，将责任树模式抽象出一个通用的框架，方便大家在遇到类似问题时快速“种树”。

这个框架由一个 Router 和 Handler 组成：

Router 是一个抽象类，负责定义如何路由到下游的多个子节点。

Handler 是接口，负责实现每个节点的业务逻辑。

我们可以非常方便地通过 Router 和 Handler 的组合拼装成整棵树的结构。

![](/static/image/微信图片_20210203111809.png)

从图中我们可以看出以下几个要点：

* 除了根节点（入口）外，每个节点都实现了 Handler 接口。根节点只继承 Router 抽象类。

* 所有叶子节点只实现 Handler 接口而无需继承 Router 抽象类（无需再向下委托）。

* 除了根节点和叶子节点外的其他节点，都是上一层的 Handler，同时是下一层的 Router。

那么我们话不多说，先看下框架代码。

#### AbstractStrategyRouter 抽象类

```
/**
 * 通用的“策略树“框架，通过树形结构实现分发与委托，每层通过指定的参数进行向下分发委托，直到达到最终的执行者。
 * 该框架包含两个类：{@code StrategyHandler} 和 {@code AbstractStrategyRouter}
 * 其中：通过实现 {@code AbstractStrategyRouter} 抽象类完成对策略的分发，
 * 实现 {@code StrategyHandler} 接口来对策略进行实现。
 * 像是第二层 A、B 这样的节点，既是 Root 节点的策略实现者也是策略A1、A2、B1、B2 的分发者，这样的节点只需要
 * 同时继承 {@code StrategyHandler} 和实现 {@code AbstractStrategyRouter} 接口就可以了。
 *
 * <pre>
 *           +---------+
 *           |  Root   |   ----------- 第 1 层策略入口
 *           +---------+
 *            /       \  ------------- 根据入参 P1 进行策略分发
 *           /         \
 *     +------+      +------+
 *     |  A   |      |  B   |  ------- 第 2 层不同策略的实现
 *     +------+      +------+
 *       /  \          /  \  --------- 根据入参 P2 进行策略分发
 *      /    \        /    \
 *   +---+  +---+  +---+  +---+
 *   |A1 |  |A2 |  |B1 |  |B2 |  ----- 第 3 层不同策略的实现
 *   +---+  +---+  +---+  +---+
 * </pre>
 *
 * @author
 * @date
 * @see StrategyHandler
 */
@Component
public abstract class AbstractStrategyRouter<T, R> {

    /**
     * 策略映射器，根据指定的入参路由到对应的策略处理者。
     *
     * @param <T> 策略的入参类型
     * @param <R> 策略的返回值类型
     */
    public interface StrategyMapper<T, R> {
        /**
         * 根据入参获取到对应的策略处理者。可通过 if-else 实现，也可通过 Map 实现。
         *
         * @param param 入参
         * @return 策略处理者
         */
        StrategyHandler<T, R> get(T param);
    }

    private StrategyMapper<T, R> strategyMapper;

    /**
     * 类初始化时注册分发策略 Mapper
     */
    @PostConstruct
    private void abstractInit() {
        strategyMapper = registerStrategyMapper();
        Objects.requireNonNull(strategyMapper, "strategyMapper cannot be null");
    }

    @Getter
    @Setter
    @SuppressWarnings("unchecked")
    private StrategyHandler<T, R> defaultStrategyHandler = StrategyHandler.DEFAULT;

    /**
     * 执行策略，框架会自动根据策略分发至下游的 Handler 进行处理
     *
     * @param param 入参
     * @return 下游执行者给出的返回值
     */
    public R applyStrategy(T param) {
        final StrategyHandler<T, R> strategyHandler = strategyMapper.get(param);
        if (strategyHandler != null) {
            return strategyHandler.apply(param);
        }

        return defaultStrategyHandler.apply(param);
    }

    /**
     * 抽象方法，需要子类实现策略的分发逻辑
     *
     * @return 分发逻辑 Mapper 对象
     */
    protected abstract StrategyMapper<T, R> registerStrategyMapper();
}
```

继承 `AbstractStrategyRouter<T, R>` 抽象类只需要实现 `protected abstract StrategyMapper<T, R>` registerStrategyMapper\(\); 抽象方法即可，在该方法中实现其不同子节点的路由逻辑。

如果子节点路由逻辑比较简单，可以直接通过 if-else 进行分发。当然如果为了更好地性能、适应更复杂的分发逻辑也可以使用 Map 等保存映射。

对于实现了该抽象类的 Router 节点，只需要调用其 public R applyStrategy\(T param\) 方法即可获取该节点的期望输出。框架会自动根据定义的路由逻辑将 param 传递到对应的子节点，再由子节点不断向下分发直到叶子节点或可以给出业务输出的一层。这个过程有点类似递归或者分治的思想。

#### StrategyHandler 接口

```
/**
 * @author
 * @date
 */
public interface StrategyHandler<T, R> {

    @SuppressWarnings("rawtypes")
    StrategyHandler DEFAULT = t -> null;

    /**
     * apply strategy
     *
     * @param param
     * @return
     */
    R apply(T param);
}
```

除了根节点外，都要实现 `StrategyHandler<T, R>`接口。如果是叶子节点，由于不需要再向下委托，因此不再需要同时继承 `AbstractStrategyRouter<T, R>` 抽象类，只需要在 R apply\(T param\); 中实现业务逻辑即可。

对于其他责任树中的中间层节点，都需要同时继承 Router 抽象类和实现 Handler 接口，在 R apply\(T param\); 方法中首先进行一定异常入参拦截，遵循 fail-fast 原则，避免将这一层可以拦截的错误传递到下一层，同时也要避免“越权”做非本层职责的拦截校验，避免产生耦合，为后面业务拓展挖坑。在拦截逻辑后直接调用本身 Router 的 public R applyStrategy\(T param\) 方法路由给下游节点即可。

## 五、完结撒花

至此，关于如何通过“责任树模式”优化这个需求场景的介绍就基本结束了，这不是一个复杂的需求，更不是一个多么精妙的优化，这只是日常需求开发中通过设计模式优化代码的一个小例子。

最后再简单聊聊我在日常需求开发过程中关于架构设计部分的一些思考。

其实并不是说用“if-else”很 Low，用设计模式就 Niubility，二者各有其擅长的应用场景，在合适的场景使用合适的代码才是正道。其实“if-else”足以满足大部分日常需求的开发，且简单、灵活、可靠。这里的“if-else”泛指朴素直白的编程模式，仅以实现需求业务功能为目的的编码方式。当然，有些同学不满足于此，希望可以通过经过思考的、更优的架构设计使代码变的更简洁、拓展性更好、性能更优、可读性更好等等。不过对于此也存在反对的论述，谓之“过早优化乃万恶之源”。

这句话源自 Donald Knuth 他老人家：

```
we should forget about small efficiencies,say about 97% of the time:premature optimization is the root of all evil.
```

这句话我当然承认其正确性，但我同样觉得需要注意以下几点：

* 任何“结论”都有其所处背景、上下文细节等，通过一句话指导工作是不成立的。优秀的架构师可以给出架构设计是在理论基础、大量实践、不断思考总结以及无数采坑的经验的基础上得来的，而不是他知道一句别人都不知道的“咒语”。

* Knuth 这句话更偏重于反对奇技淫巧、细枝末节的性能优化，因为在“过早”的时候无法准确获知系统的瓶颈且局部的优化不仅不能带来收益，反而会造成更大的代价。他批评的恰恰是不着眼于整体架构的局部视角对系统的破坏，而架构设计正是需要从整体视角去做选择与权衡。因此将 Knuth 这句话直接推广到“架构设计”上并不妥当。

* 很多人觉得在项目开发时需求经常“瞬息万变”、“朝令夕改”，而做优化又需要花费大量时间思考，根本没有精力优化。我认为这种论述也是不成立的，凭什么认为等到坏味道严重、历史包袱沉重的时候就有精力、能力和胆量做优化了呢？

* 何时是所谓的“不早”很难界定，其实我们永远都无法确定自己掌握了足够的细节可以进行绝对正确的优化。在现实世界中，受到时间维度的限制，我们永远无法达成全局最优，只能以局部最优不断去逼近全局最优。我觉得等到坏味道严重不得不重构的时候才想起优化已为时过晚。

* 这句话不应该成为不做设计的借口，即使最终提交的代码仍是“if-else”版本，也不应省略思考、推演、权衡的过程，日常需求是练兵场，是精进技术的必经之路。

所以，我觉得不要被这句话束缚手脚，当然更不要闭门造车，在开发过程中勤于思考，向更有经验的人请教，在架构设计上不断学习、探索，才能摆脱日复一日通过“if-else”堆砌业务逻辑的循环。

# 2.参考

如何优化你的if-else？来试试“责任树模式”

```
https://developer.aliyun.com/article/781471?utm_content=g_1000234115
```


