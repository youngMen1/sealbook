# ForkJoinPool原理及使用方式

Fork的是分叉，交叉的意思，Join是合并、聚合的意思。所以Fork Join模式指的就是不停的拆分和聚合的过程。“天下大势，合久必分，分久必合”描述的就是一个不停Fork Join的过程。“分而治之”也是对ForkJoin模式一个描述。我们常见的归并排序算法就是ForkJoin模式的一个具体应用：
88870061-f816aa00-d246-11ea-9a90-744d8f332186.png

ForkJoinPool是JDK1.7出现的一个对于forkjoin模式的实现。通过ForkJoinPool可以轻松做到任务的Fork以及Join操作。同时在JDK1.8时，又使用避免代码伪共享对其性能优化，以及CompletableFuture及集合并行计算等，底层都是基于ForkJoinPool实现。

## 2.使用Demo（可见项目seal-common->seal-features）

接下来使用ForkJoinPool实现一个简单的归并排序算法。





