# ForkJoinPool原理及使用方式

Fork的是分叉，交叉的意思，Join是合并、聚合的意思。所以Fork Join模式指的就是不停的拆分和聚合的过程。“天下大势，合久必分，分久必合”描述的就是一个不停Fork Join的过程。“分而治之”也是对ForkJoin模式一个描述。我们常见的归并排序算法就是ForkJoin模式的一个具体应用：
88870061-f816aa00-d246-11ea-9a90-744d8f332186.png

ForkJoinPool是JDK1.7出现的一个对于forkjoin模式的实现。通过ForkJoinPool可以轻松做到任务的Fork以及Join操作。同时在JDK1.8时，又使用避免代码伪共享对其性能优化，以及CompletableFuture及集合并行计算等，底层都是基于ForkJoinPool实现。

## 2.使用Demo（可见项目seal-common->seal-features）

接下来使用ForkJoinPool实现一个简单的归并排序算法。

```
 public static void main(String[] args) {
        int[] arr = new int[]{1,3,4,2,5,7,3,8,5,9};
        int[] result = new int[10];
        ForkJoinPool forkJoinPool = ForkJoinPool.commonPool();
        MergerSortTask mergerSortTask = new MergerSortTask(0, arr.length - 1, arr, result);
        forkJoinPool.submit(mergerSortTask);
        System.out.println(Arrays.toString(result));
    }
    static class MergerSortTask extends RecursiveAction {
        private int begin;
        private int end;
        private int[] source;
        private int[] result;
        public MergerSortTask(int begin,int end,int[] source,int[] result) {
            this.begin = begin;
            this.end = end;
            this.source = source;
            this.result = result;
        }
        @Override
        protected void compute() {
            if (end<=begin){
                return;
            }
            int mid = (end - begin)/2;
            MergerSortTask lowerSortTask = new MergerSortTask(begin, mid, source,result);
            MergerSortTask higherSortTask = new MergerSortTask(mid+1, end, source,result);
            invokeAll(lowerSortTask,higherSortTask);
            //merge
            int i = begin;
            int j = mid +1;
            int k = begin;
            while (i <= mid && j <= end)
                result[k++] = source[i] < source[j] ? source[i++] : source[j++];
            while (i <= mid)
                result[k++] = source[i++];
            while (j <= end)
                result[k++] = source[j++];
        }
        public int[] getResult() {
            return result;
        }

    }

```

## 核心原理

ForkJoinPool与ThreadPoolExecutor都是ExecutorService的实现类，同时也都是Doug Lea大神的作品。ForkJoinPool并不是作为ThreadPoolExecutor的优化，两者应该是互补的关系，处理的场景也不同。根本性区别在于ForkJoinPool使用了Fork-Join和Work-Stealing机制。这两个机制保证了ForkJoinPool可以将大任务拆分成多个小任务，使用整个ForkJoinPool的多线程能力参与计算，避免了ThreadPoolExecutor在处理单个大任务造成的线程池吞吐量低问题。但是ForkJoinPool也存在自身的缺点，ForkJoinPool更适合处理可以被Fork的CPU密集型任务，大量能够被快速计算的小任务才能将ForkJoinPool优势发挥的更高。


### work-stealing机制

### 如何创建一个ForkJoinPool

### 提交任务至ForkJoinPool





