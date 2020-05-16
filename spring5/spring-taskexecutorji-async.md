# 1.基本介绍

在做项目过程中，一些耗时长的任务可能需要在后台线程池中运行；

典型的如**发送邮件、收集操作日志**等，由于需要调用外部的接口来进行实际的发送操作，如果客户端在提交发送请求后一直等待服务器端发送成功后再返回，就会长时间的占用服务器的一个连接；当这类请求过多时，服务器连接数会不够用，新的连接请求可能无法得到满足，从而导致客户端连接失败。因此这类服务一般需要使用到后台线程池来处理。

在这种情况下，我们可以直接使用concurrent包中的线程池来处理，也可以使用其它的方案如Quartz等组件中的线程池来解决；为适配这些不同的方案，Spring引入了TaskExecutor接口作为顶层接口，并提供了几种不同的实现来满足不同的场景。

## 1.1.Spring包含了以下TaskExecutor的实现：

### 1.1.1.ThreadPoolTaskExecutor

它是最经常使用的一个，提供了一些Bean属性用于配置java.util.concurrent.ThreadPoolExecutor并且将其包装到TaskExecutor对象中。如果需要适配java.util.concurrent.Executor，请使用ConcurrentTaskExecutor。

### 1.1.2.SimpleAsyncTaskExecutor

线程不会重用，每次调用时都会重新启动一个新的线程；但它有一个最大同时执行的线程数的限制；

### 1.1.3.SyncTaskExecutor

同步的执行任务，任务的执行是在主线程中，不会启动新的线程来执行提交的任务。主要使用在没有必要使用多线程的情况，如较为简单的测试用例。

### 1.1.4.ConcurrentTaskExecutor

它用于适配java.util.concurrent.Executor， 一般情况下请使用ThreadPoolTaskExecutor，如果hreadPoolTaskExecutor不够灵活时可以考虑采用ConcurrentTaskExecutor。

### 1.1.5.SimpleThreadPoolTaskExecutor

它是Quartz中SimpleThreadPool的一个实现，用于监听Spring生命周期回调事件。它主要使用在需要一个线程池来被Quartz和非Quartz中的对象同时共享使用的情况。

### 1.1.6.WorkManagerTaskExecutor

它实现了CommonJ中的WorkManager接口，是在Spring中使用CommonJ的WorkManager时的核心类。

# 2.怎么使用

## 2.1.注册TaskExecutor

```
@Configuration
@EnableAsync
@EnableScheduling
public class AsyncTaskExecutorConfiguration implements AsyncConfigurer {
	private final Logger log = LoggerFactory.getLogger(getClass());
	@Resource
	private PaascloudProperties paascloudProperties;

	@Override
	@Bean(name = "taskExecutor")
	public Executor getAsyncExecutor() {
		log.debug("Creating Async Task Executor");
		ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
		executor.setCorePoolSize(paascloudProperties.getTask().getCorePoolSize());
		executor.setMaxPoolSize(paascloudProperties.getTask().getMaxPoolSize());
		executor.setQueueCapacity(paascloudProperties.getTask().getQueueCapacity());
		executor.setKeepAliveSeconds(paascloudProperties.getTask().getKeepAliveSeconds());
		executor.setThreadNamePrefix(paascloudProperties.getTask().getThreadNamePrefix());
		return new ExceptionHandlingAsyncTaskExecutor(executor);
	}

	@Override
	public AsyncUncaughtExceptionHandler getAsyncUncaughtExceptionHandler() {
		return new SimpleAsyncUncaughtExceptionHandler();
	}
}
```

## 2.2.使用TaskExecutor

## 2.3.使用Async

## 2.4.添加EnableAsync

# 3.参考

Spring基础学习-任务执行（TaskExecutor及Async）：

[https://blog.csdn.net/icarusliu/article/details/79528810](https://blog.csdn.net/icarusliu/article/details/79528810)

