# 1.如何写出健壮的代码?

阿里文娱技术专家长统将从防御式编程、如何正确使用异常和 DRY 原则等三个方面

你不可能写出完美的软件。因为它不曾出现，也不会出现。

每一个司机都认为自己是最好的司机，我们在鄙视那些闯红灯、乱停车、胡乱变道不遵守规则的司机同时，更应该在行驶的过程中防卫性的驾驶，小心那些突然冲出来的车辆，在他们给我们造成麻烦的时候避开他。这跟编程有极高的相似性，我们在程序的世界里要不断的跟他人的代码接合（那些不符合你的高标准的代码），并处理可能有效也可能无效的输入。无效的的输入就好像一辆横冲直撞的大卡车，这样的世界防御式编程也是必须的，但要驶得万年船我们可能连自己都不能信任，因为你不知道冲出去的那辆是不是你自己的车。关于这点我们将在防御式编程中讨论。

没人能否认异常处理在 Java 中的重要性，但如果不能正确使用异常处理那么它带来的危害可能比好处更多。我将在正确使用异常中讨论这个问题。

DRY,Don't Repeat Yourself. 不要重复你自己。我们都知道重复的危害性，但重复时常还会出现在我们的工作中、代码中、文档中。有时重复感觉上是不得不这么做，有时你并没有意识到是在重复，有时却是因为懒惰而重复。

好借好还再借不难。这句俗话在编程世界中同样也是至理名言。只要在编程，我们都要管理资源：内存、事物、线程、文件、定时器，所有数量有限的事物都称为资源。资源使用一般遵循的模式：你分配、你使用、你回收。

## 防御式编程
防御式编程是提高软件质量技术的有益辅助手段。防御式编程的主要思想是：程序/方法不应该因传入错误数据而被破坏，哪怕是其他由自己编写方法和程序产生的错误数据。这种思想是将可能出现的错误造成的影响控制在有限的范围内。

一个好程序，在非法输入的情况下，要么什么都不输出，要么输出错误信息。我们往往会检查每一个外部的输入（一切外部数据输入，包括但不仅限于数据库和配置中心），我们往往也会检查每一个方法的入参。我们一旦发现非法输入，根据防御式编程的思想一开始就不引入错误。

## 使用卫语句
对于非法输入的检查我们通常会使用 if…else 去做判断，但往往在判断过程中由于参数对象的层次结构会一层一层展开判断。


```
public void doSomething(DomainA a) {
  if (a != null) {
        assignAction;
    if (a.getB() != null) {
      otherAction;
      if (a.getB().getC() instanceof DomainC) {
        doSomethingB();
        doSomethingA();
        doSomthingC();
      }
    }
  }
}
```
上边的嵌套判断的代码我相信很多人都见过或者写过，这样做虽然做到了基本的防御式编程，但是也把丑陋引了进来。《Java 开发手册》中建议我们碰到这样的情况使用卫语句的方式处理。什么是卫语句？我们给出个例子来说明什么是卫语句。


```
public void doSomething(DomainA a) {
    if (a == null) {
        return ; //log some errorA
    }
    if (a.getB() == null) {
        return ; //log some errorB
    }
    if (!(a.getB().getC instanceof DomainC)) {
        return ;//log some errorC
    }
    assignAction;
    otherAction;
    doSomethingA();
    doSomethingB();
    doSomthingC();
}
```
方法中的条件逻辑使人难以看清正常的分支执行路径，所谓的卫语句的做法就是将复杂的嵌套表达式拆分成多个表达式，我们使用卫语句表现所有特殊情况。
## 使用验证器 (validator)
验证器是我在开发中的一种实践，将合法性检查与 OOP 结合是一种非常奇妙的体验。


```
public List<DemoResult> demo(DemoParam dParam) {
    Assert.isTrue(dParam.validate(),()-> new SysException("参数验证失败-" + DemoParam.class.getSimpleName() +"验证失败：" + dParam));
    DemoResult demoResult = doBiz();
    doSomething();
    return demoResult;
}
```
在这个示例中，方法的第一句话就是对验证器的调用，以获得当前参数是否合法。

在参数对象中实现验证接口，为字段配置验证注解，如果需要组合验证复写 validate0 方法。这样就把合法性验证逻辑封装到了对象中。


```
public class DemoParam extends BaseDO implements ValidateSubject {
    @ValidateString(strMaxLength = 128)
    private String aString;
    @ValidateObject(require = true)
    private List<SubjectDO> bList;
    @ValidateString(require = true,strMaxLength = 128)
    private String cString;
    @ValidateLong(require = true)
    private Long dLong;
    @Override
    public boolean validate0(ValidateSubject validateSubject) throws ValidateException {
        if (validateSubject instanceof DemoParam) {
            DemoParam param = (DemoParam)validateSubject;
            return StringUtils.isNotBlank(param.getAString())
                   && SubjectDO.allValidate(param.getBList());
        }
        return false;
    }
}
```
## 使用断言
当出现了一个突如其来的线上问题，我相信很多伙伴的心中一定闪现过这样一个念头。"这不科学啊...这不可能发生啊…"，"计数器怎么可能为负数"，"这个对象不可为null"，但它就是真实的发生了，它就在那。我们不要这样骗自己，特别是在编码时。如果它不可能发生，用断言确保它不会发生。

使用断言的重要原则就是，断言不能有副作用，也绝不能把必须执行的代码放入断言。

断言不能有副作用，如果我每年增加错误检查代码却制造了新的错误，那是一件令人尴尬的事情。举一个反面例子：


```
while (iter.next() != null) {
    assert(iter.next()!=null);
    Object next = iter.next();
    //...
}
```



必须执行的代码也不能放入断言，因为生产环境很可能是关闭 Java 断言的。因此我更喜欢使用 Spring 提供的 Assert 工具，这个工具提供的断言只会返回 IllegalStateException，如果需要这个异常不能满足我们的业务需求，我们可以重新创建一个 Assert 类并继承 org.springframework.util.Assert，在新类中新增断言方法以支持自定义异常的传入。



```
public class Assert extends org.springframework.util.Assert {
    public static <T extends RuntimeException> void isTrue(boolean expression, Supplier<T> tSupplier) {
        if (!expression) {
            if (tSupplier != null) {
                throw tSupplier.get();
            }
            throw new IllegalArgumentException();
        }
    }
}
```


```
Assert.isTrue(crParam.validate(),()-> new SysException("参数验证失败-" + Calculate.class.getSimpleName() +"验证失败：" + crParam));
```

有人认为断言仅是一种调试工具，一旦代码发布后就应该关闭断言，因为断言会增加一些开销（微小的 CPU 时间）。所以在很多工程实践中断言确实是关闭的，也有不少大 V 有过这样的建议。Dndrew Hunt 和 David Thomas 反对这样的说法，在他们书里有一个比喻我认为很形象。



```
在你把程序交付使用时关闭断言就像是因为你曾经成功过，就不用保护网取走钢丝。

——《The pragmatic Programmer》
```


