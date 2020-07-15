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

## 处理错误时的关键选择
防御式编程会预设错误处理。

在错误发生后的后续流程上通常会有两种选择，终止程序和继续运行。


* 终止程序，如果出现了非常严重的错误，那最好终止程序或让用户重启程序。比如，银行的 ATM 机出现了错误，那就关闭设备，以防止取 100 块吐出 10000 块的悲剧发生。

* 继续运行，通常也是有两种选择，本地处理和抛出错误。本地处理通常会使用默认值的方式处理，抛出错误会以异常或者错误码的形式返回。

在处理错误的时候我们还面临着另外一种选择，正确性和健壮性的选择。

* 正确性，选择正确性意味着结果永远是正确的，如果出错，宁愿不给出结果也不要给定一个不准确的值。比如用户资产类的业务。

* 健壮性，健壮性意味着通过一些措施，保证软件能够正常运行下去，即使有时候会有一些不准确的值出现。比如产品介绍超过页面展示范围

无论是使用卫语、断言还是预设错误处理都是在用抱着对程序世界的敬畏态度在小心的驾驶，时刻提防着他人更提防着自己。

## 正确使用异常
检查每一个可能的错误是一种良好的实践，特别是那些意料之外的错误。

非常棒的是，Java 为我们提供了异常机制。如果充分发挥异常的优点，可以提高程序的可读性、可靠性和可维护性，但如果使用不当，异常带来的负面影响也是非常值得我们注意并避免的。



### 只在异常情况下使用异常

在《The pragmatic Programmer》和《Effective Java》中作者都有这样的观点。

我认为这有两重意思。一重意思如何处理识别异常情况并处理他，另一重意思是只在异常情况下使用异常流程。

那什么是异常情况，又该如何处理？这个问题无法在代码模式上给出标准的答案，完全看实际情况，要对每一个错误了然于胸并检查每一个可能发生的错误，并区分错误和异常。

即便同样是打开文件操作，读取"/etc/passwd"和读取一个用户上传的文件，同样是 FileNotFoundException，如何处理完全取决于实际情况，Surprise！前者直接读取文件出现异常直接抛出让程序尽早崩溃，而后者应该先判断文件是否存在，如果存在但出现了 FileNotFoundException 则再抛出。



```
public static void openPasswd() throws FileNotFoundException {
        FileInputStream fs = new FileInputStream("/etc/passwd");
    }
```

读取"/etc/passwd"失败，Surprise！




```
public static boolean openUserFile(String path) throws FileNotFoundException {
        File f = new File(path);
        if (!f.exists()) {
            return false;
        }
        FileInputStream fs = new FileInputStream(path);
        return true;
    }
```


在文件存在的情况下读取文件失败，Surprise！

再啰嗦一遍，是不是异常情况关键在于它是不是给我们一记 Surprise！，这就是本节开头检查每一个错误是一种良好的实践想要表达的。

使用异常来控制正常流程的反面例子我就偷懒借用一下《Effective Java Second Edition》里的例子来说明好了。

```
Integer[] range ={1,2,3};
//Horrible abuse of exceptions.Don't ever do this!
try {
  int i=0;
  println(range[i++].intValue());
} catch (ArrayIndexOutOfBoundsException e) {}
```

这个例子看起来根本不知道在干什，这段代码其实是在用数组越界异常来控制遍历数组，这个脑洞开的非常拙劣。如何正确遍历一个数组我想不需要再给出例子，那是对读者的亵渎。

那为什么有人这么开脑洞呢？因为这个做法企图使用 Java 错误判断机制来提高性能，因为 JVM 对每一次数组访问都会检查越界情况，所以他们认为检查到的错误才应该是循环终止的条件，然而 for-each 循环对已经检查到的错误视而不见，将其隐藏了，所以用应该避免使用 for-each。

对于这个脑洞的原因 Joshua Bloch 给出了三点反驳：

* 因为异常机制的设计初衷是用于不正常的情形，所以很少会有 JVM 实现试图对它们进行优化，使得与显示测试一样快速。

* 把代码放在 try-catch 块中反而阻止了现代 JVM 实现本来可能要执行的某些特定优化。

* 对数组进行遍历的标准模式并不会导致冗余的检查。有些现代的 JVM 实现会将他们优化掉。

还有一个例子是我曾经遇到的，但是由于年代久远已经找不到项目地址了。我一个朋友曾经给我看过一个 github 上的 MVC 框架项目，虽然时隔多年但令我印象深刻的是这个项目使用自定义异常和异常码来控制 Dispatcher，把异常当成一种方便的结果传递方式来使用，当成 goto 来使用，这太可怕了。不过 try-catch 方式从字节码表述上来看，确实是一种 goto 的表述。这样的方式我们最好想都不要想。

这两个例子主要就是为了说明，异常应该只用于异常的情况下；永远不应该用在正常的流程中，不管你的理由看着多么的聪明。这样做往往会弄巧成拙，使得代码可读性大大下降。

### 受检异常和非受检异常

曾经不止一次的见过有人提倡将系统中的受检异常都包装成非受检异常，对于这个建议我并不以为然。因为 Java 的设计者其实是希望通过区分异常种类来指导我们编程的。

Java 一共提供了三类可抛出结构 (throwable)，受检异常、非受检异常(运行时异常)和错误 (error)。他们的界限我也经常傻傻的分不清，不过还是有迹可循的。

* 受检异常：如果期望调用者能够适当的恢复，比如 RMI 每次调用必须处理的异常，设计者是期望调用者可以重试或别的方式来尝试恢复；比如上边提到的 FileInputStream 的构造方法，会抛出 FileNotFoundException，设计者或许希望调用者尝试从其他目录来读取该文件，使得程序可以继续执行下去。


* 非受检异常和错误：表明是编程错误，往往属于不可恢复的情景，而且是不应该被提前捕获的，应该快速抛出到顶层处理器，比如在服务接口的基类方法中统一处理非受检异常。这种非受检异常往往也说明了在编程中违反了某些约定。比如数组越界异常，说明违反了访问数组不能越界的前提约定。


总而言之，对于可恢复的情况使用受检异常；对于程序错误使用非受检异常。因此你自己程序内部定义的异常都应该是非受检异常；在面向接口或面向二方/三方库的方法尽量使用受检异常。

说到面向接口或面向二/三方库，你可能碰到的就是一辆失控的汽车。搞清楚你所调用的接口或者库里的异常情况也是我们能够码出健壮代码的一个强力保证。


### 不要忽略异常

这个建议显而易见，但却常常被违反。当一个 API 的设计者声明一个方法将抛出异常的时候，通常都是想要说明某件事发生了。忽略异常就是我们通常说的吃掉异常，try-catch 但什么也不做。吃掉一个异常就好比破坏了一个报警器，当灾难真正来临没人搞清楚发生了什么。

对于每一个 catch 块至少打印一条日志，说明异常情况或者说明为什么不处理。

这个显而易见的建议同时适用于受检异常和非受检异常。

## DRY (Don't Repeat Yourself)

DRY 原则最先在《The pragmatic Programmer》被提出，如今已经被业界广泛的认知，我相信每个软件工程师都认识它。我想有很多人对它的认识含混不清仅仅是不要有重复的代码；也有些人对此原则不屑一顾抽象什么的都是浪费时间快速上线是大义；也有人誓死捍卫这个原则不能忍受任何重复。今天我们来谈谈这个熟悉又陌生的话题。

### DRY 是什么
DRY 的原则是“系统中的每一部分，都必须有一个单一的、明确的、权威的代表”，指的是（由人编写而非机器生成的）代码和测试所构成的系统，必须能够表达所应表达的内容，但是不能含有任何重复代码。当 DRY 原则被成功应用时，一个系统中任何单个元素的修改都不需要与其逻辑无关的其他元素发生改变。此外，与之逻辑上相关的其他元素的变化均为可预见的、均匀的，并如此保持同步。

这段定义来自于中文维基百科，但这个定义似乎与 Andrew Hunt 和 David Thomas 给出的定义有所出入。寻根溯源在《The pragmatic Programmer》作者是这样定义这个原则的：


```
系统中的每一项知识都必须具有单一、无歧义、权威的表示。
```



