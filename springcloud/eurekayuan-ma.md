# 1.基本介绍

## 1.1.Eureka Server服务注册中心源码分析

回忆之前我们一起搭建的服务注册中心的项目，我们在服务注册中心的项目中的**application.properties**文件中配置好服务注册中心需要的相关配置，然后在**Spring Boot**的启动类中加了一个注解**@EnableEurekaServer**，然后启动项目就成功启动了服务注册中心，那么到底是如何启动的呢？  
在配置文件中（单节点），我们是如下配置的：

```
# 配置端口
server.port=1111
# 配置服务注册中心地址
eureka.instance.hostname=localhost
# 作为服务注册中心，禁止本应用向自己注册服务
eureka.client.register-with-eureka=false
# 作为服务注册中心，禁止本应用向自己检索服务
eureka.client.fetch-registry=false
# 设置服务注册中心服务注册地址
eureka.client.service-url.defaultZone=http://${eureka.instance.hostname}:${server.port}/eureka/
# 关闭自我保护机制，及时剔除无效服务
eureka.server.enable-self-preservation=false
```

这个配置在工程启动的时候，会被**Spring**容器读取，配置到**EurekaClientConfigBean**中，而这个配置类会被注册成**Spring**的**Bean**以供其他的**Bean**来使用。

我们再进入注解**@EnableEurekaServer**一探究竟，**@EnableEurekaServer**的源码如下：

```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EurekaServerMarkerConfiguration.class)
public @interface EnableEurekaServer {

}
```

从上述注解可以看出，该注解导入了配置类`EurekaServerMarkerConfiguration`，我们在进一步进入到

`EurekaServerMarkerConfiguration`中，代码如下所示：

```
@Configuration
public class EurekaServerMarkerConfiguration {
    @Bean
    public Marker eurekaServerMarkerBean() {
        return new Marker();
    }
    class Marker {
    }
}
```

从这个配置类中暂时无法看到什么具体的内容，我们可以进一步查看类`Marker`在哪些地方被使用了，通过搜索`Marker`，可以发现在类`EurekaServerAutoConfiguration`上的注解中被引用了，具体代码如下所示：

```
/**
 * @author Gunnar Hillert
 * @author Biju Kunjummen
 * @author Fahim Farook
 */
@Configuration(proxyBeanMethods = false)
@Import(EurekaServerInitializerConfiguration.class)
@ConditionalOnBean(EurekaServerMarkerConfiguration.Marker.class)
@EnableConfigurationProperties({ EurekaDashboardProperties.class,
        InstanceRegistryProperties.class })
@PropertySource("classpath:/eureka/server.properties")
public class EurekaServerAutoConfiguration implements WebMvcConfigurer {

    /**
     * List of packages containing Jersey resources required by the Eureka server.
     */
    private static final String[] EUREKA_PACKAGES = new String[] {
            "com.netflix.discovery", "com.netflix.eureka" };

    @Autowired
    private ApplicationInfoManager applicationInfoManager;

    @Autowired
    private EurekaServerConfig eurekaServerConfig;

    @Autowired
    private EurekaClientConfig eurekaClientConfig;

    @Autowired
    private EurekaClient eurekaClient;

    @Autowired
    private InstanceRegistryProperties instanceRegistryProperties;

    /**
     * A {@link CloudJacksonJson} instance.
     */
    public static final CloudJacksonJson JACKSON_JSON = new CloudJacksonJson();

    @Bean
    public HasFeatures eurekaServerFeature() {
        return HasFeatures.namedFeature("Eureka Server",
                EurekaServerAutoConfiguration.class);
    }

    @Bean
    @ConditionalOnProperty(prefix = "eureka.dashboard", name = "enabled",
            matchIfMissing = true)
    public EurekaController eurekaController() {
        return new EurekaController(this.applicationInfoManager);
    }

    static {
        CodecWrappers.registerWrapper(JACKSON_JSON);
        EurekaJacksonCodec.setInstance(JACKSON_JSON.getCodec());
    }

    @Bean
    public ServerCodecs serverCodecs() {
        return new CloudServerCodecs(this.eurekaServerConfig);
    }

    private static CodecWrapper getFullJson(EurekaServerConfig serverConfig) {
        CodecWrapper codec = CodecWrappers.getCodec(serverConfig.getJsonCodecName());
        return codec == null ? CodecWrappers.getCodec(JACKSON_JSON.codecName()) : codec;
    }

    private static CodecWrapper getFullXml(EurekaServerConfig serverConfig) {
        CodecWrapper codec = CodecWrappers.getCodec(serverConfig.getXmlCodecName());
        return codec == null ? CodecWrappers.getCodec(CodecWrappers.XStreamXml.class)
                : codec;
    }

    @Bean
    @ConditionalOnMissingBean
    public ReplicationClientAdditionalFilters replicationClientAdditionalFilters() {
        return new ReplicationClientAdditionalFilters(Collections.emptySet());
    }

    @Bean
    public PeerAwareInstanceRegistry peerAwareInstanceRegistry(
            ServerCodecs serverCodecs) {
        this.eurekaClient.getApplications(); // force initialization
        return new InstanceRegistry(this.eurekaServerConfig, this.eurekaClientConfig,
                serverCodecs, this.eurekaClient,
                this.instanceRegistryProperties.getExpectedNumberOfClientsSendingRenews(),
                this.instanceRegistryProperties.getDefaultOpenForTrafficCount());
    }

    @Bean
    @ConditionalOnMissingBean
    public PeerEurekaNodes peerEurekaNodes(PeerAwareInstanceRegistry registry,
            ServerCodecs serverCodecs,
            ReplicationClientAdditionalFilters replicationClientAdditionalFilters) {
        return new RefreshablePeerEurekaNodes(registry, this.eurekaServerConfig,
                this.eurekaClientConfig, serverCodecs, this.applicationInfoManager,
                replicationClientAdditionalFilters);
    }

    @Bean
    @ConditionalOnMissingBean
    public EurekaServerContext eurekaServerContext(ServerCodecs serverCodecs,
            PeerAwareInstanceRegistry registry, PeerEurekaNodes peerEurekaNodes) {
        return new DefaultEurekaServerContext(this.eurekaServerConfig, serverCodecs,
                registry, peerEurekaNodes, this.applicationInfoManager);
    }

    @Bean
    public EurekaServerBootstrap eurekaServerBootstrap(PeerAwareInstanceRegistry registry,
            EurekaServerContext serverContext) {
        return new EurekaServerBootstrap(this.applicationInfoManager,
                this.eurekaClientConfig, this.eurekaServerConfig, registry,
                serverContext);
    }

    /**
     * Register the Jersey filter.
     * @param eurekaJerseyApp an {@link Application} for the filter to be registered
     * @return a jersey {@link FilterRegistrationBean}
     */
    @Bean
    public FilterRegistrationBean<?> jerseyFilterRegistration(
            javax.ws.rs.core.Application eurekaJerseyApp) {
        FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<Filter>();
        bean.setFilter(new ServletContainer(eurekaJerseyApp));
        bean.setOrder(Ordered.LOWEST_PRECEDENCE);
        bean.setUrlPatterns(
                Collections.singletonList(EurekaConstants.DEFAULT_PREFIX + "/*"));

        return bean;
    }

    /**
     * Construct a Jersey {@link javax.ws.rs.core.Application} with all the resources
     * required by the Eureka server.
     * @param environment an {@link Environment} instance to retrieve classpath resources
     * @param resourceLoader a {@link ResourceLoader} instance to get classloader from
     * @return created {@link Application} object
     */
    @Bean
    public javax.ws.rs.core.Application jerseyApplication(Environment environment,
            ResourceLoader resourceLoader) {

        ClassPathScanningCandidateComponentProvider provider = new ClassPathScanningCandidateComponentProvider(
                false, environment);

        // Filter to include only classes that have a particular annotation.
        //
        provider.addIncludeFilter(new AnnotationTypeFilter(Path.class));
        provider.addIncludeFilter(new AnnotationTypeFilter(Provider.class));

        // Find classes in Eureka packages (or subpackages)
        //
        Set<Class<?>> classes = new HashSet<>();
        for (String basePackage : EUREKA_PACKAGES) {
            Set<BeanDefinition> beans = provider.findCandidateComponents(basePackage);
            for (BeanDefinition bd : beans) {
                Class<?> cls = ClassUtils.resolveClassName(bd.getBeanClassName(),
                        resourceLoader.getClassLoader());
                classes.add(cls);
            }
        }

        // Construct the Jersey ResourceConfig
        Map<String, Object> propsAndFeatures = new HashMap<>();
        propsAndFeatures.put(
                // Skip static content used by the webapp
                ServletContainer.PROPERTY_WEB_PAGE_CONTENT_REGEX,
                EurekaConstants.DEFAULT_PREFIX + "/(fonts|images|css|js)/.*");

        DefaultResourceConfig rc = new DefaultResourceConfig(classes);
        rc.setPropertiesAndFeatures(propsAndFeatures);

        return rc;
    }

    @Bean
    @ConditionalOnBean(name = "httpTraceFilter")
    public FilterRegistrationBean<?> traceFilterRegistration(
            @Qualifier("httpTraceFilter") Filter filter) {
        FilterRegistrationBean<Filter> bean = new FilterRegistrationBean<Filter>();
        bean.setFilter(filter);
        bean.setOrder(Ordered.LOWEST_PRECEDENCE - 10);
        return bean;
    }
```

在这个配置类上面，加入了**@ConditionalOnBean\(EurekaServerMarkerConfiguration.Marker.class\)**，也就是说，类**EurekaServerAutoConfiguration**被注册为**Spring Bean**的前提是在**Spring**容器中存在**EurekaServerMarkerConfiguration.Marker.class**的对象，而这个对象存在的前提是我们在**Spring Boot**启动类中加入了**@EnableEurekaServer**注解。

小总结一下就是，在**Spring Boot**启动类上加入了**@EnableEurekaServer**注解以后，就会触发**EurekaServerMarkerConfiguration.Marker.class**被**Spring**实例化为**Spring Bean**，有了这个**Bean**以后，**Spring**就会再实例化**EurekaServerAutoConfiguration**类，而这个类就是配置了**Eureka Server**的相关内容，列举如下：

```
注入EurekaServerConfig—->用于注册中心相关配置信息
注入EurekaController—->提供注册中心上相关服务信息的展示支持
注入PeerAwareInstanceRegistry—->提供实例注册支持,例如实例获取、状态更新等相关支持
注入PeerEurekaNodes—->提供注册中心对等服务间通信支持
注入EurekaServerContext—->提供初始化注册init服务、初始化PeerEurekaNode节点信息
注入EurekaServerBootstrap—->用于初始化initEurekaEnvironment/initEurekaServerContext
```

而且，在类**EurekaServerAutoConfiguration**上，我们看见**@Import\(EurekaServerInitializerConfiguration.class\)**，说明实例化类**EurekaServerAutoConfiguration**之前，已经实例化了**EurekaServerInitializerConfiguration**类，从类名可以看出，该类是**Eureka Server**的初始化配置类，我们进入到**EurekaServerInitializerConfiguration**类中一探究竟，发现该类实现了**Spring**的生命周期接口**SmartLifecycle**，也就是说类**EurekaServerInitializerConfiguration**在被**Spring**实例化过程中的时候会执行一些生命周期方法，比如**Lifecycle**的**start**方法，那么看看**EurekaServerInitializerConfiguration**是如何重写**start**方法的：

```
@Configuration
public class EurekaServerInitializerConfiguration
        implements ServletContextAware, SmartLifecycle, Ordered {

    // 此处省略部分代码

    @Override
    public void start() {
        new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    //TODO: is this class even needed now?
                    eurekaServerBootstrap.contextInitialized(EurekaServerInitializerConfiguration.this.servletContext);
                    log.info("Started Eureka Server");

                    publish(new EurekaRegistryAvailableEvent(getEurekaServerConfig()));
                    EurekaServerInitializerConfiguration.this.running = true;
                    publish(new EurekaServerStartedEvent(getEurekaServerConfig()));
                }
                catch (Exception ex) {
                    // Help!
                    log.error("Could not initialize Eureka servlet context", ex);
                }
            }
        }).start();
    }

    // 此处省略部分代码
}
```

这个`start`方法中开启了一个新的线程，然后进行一些`Eureka Server`的初始化工作，比如调用`eurekaServerBootstrap的contextInitialized`方法，进入该方法看看：这个`start`方法中开启了一个新的线程，然后进行一些`Eureka Server`的初始化工作，比如调用`eurekaServerBootstrap的contextInitialized`方法，进入该方法看看：这个`start`方法中开启了一个新的线程，然后进行一些`Eureka Server`的初始化工作，比如调用`eurekaServerBootstrap的contextInitialized`方法，进入该方法看看：

```
public class EurekaServerBootstrap {
    public void contextInitialized(ServletContext context) {
        try {
            // 初始化Eureka Server环境变量
            initEurekaEnvironment();
            // 初始化Eureka Server上下文
            initEurekaServerContext();

            context.setAttribute(EurekaServerContext.class.getName(), this.serverContext);
        }
        catch (Throwable e) {
            log.error("Cannot bootstrap eureka server :", e);
            throw new RuntimeException("Cannot bootstrap eureka server :", e);
        }
    }
}
```

这个方法中主要进行了Eureka的环境初始化和服务初始化，我们进入到initEurekaServerContext方法中来看服务初始化是如何实现的：

```
public class EurekaServerBootstrap {
    protected void initEurekaServerContext() throws Exception {
        // For backward compatibility
        JsonXStream.getInstance().registerConverter(new V1AwareInstanceInfoConverter(),
                XStream.PRIORITY_VERY_HIGH);
        XmlXStream.getInstance().registerConverter(new V1AwareInstanceInfoConverter(),
                XStream.PRIORITY_VERY_HIGH);

        if (isAws(this.applicationInfoManager.getInfo())) {
            this.awsBinder = new AwsBinderDelegate(this.eurekaServerConfig,
                    this.eurekaClientConfig, this.registry, this.applicationInfoManager);
            this.awsBinder.start();
        }
        // 初始化Eureka Server上下文环境
        EurekaServerContextHolder.initialize(this.serverContext);

        log.info("Initialized server context");

        // Copy registry from neighboring eureka node
        int registryCount = this.registry.syncUp();
        // 期望每30秒接收到一次心跳，1分钟就是2次
        // 修改Instance Status状态为up 
        // 同时，这里面会开启一个定时任务，用于清理 60秒没有心跳的客户端。自动下线
        this.registry.openForTraffic(this.applicationInfoManager, registryCount);

        // Register all monitoring statistics.
        EurekaMonitors.registerAllStats();
    }
}
```

在初始化**Eureka Server**上下文环境后，就会继续执行**openForTraffic**方法，这个方法主要是设置了期望每分钟接收到的心跳次数，并将服务实例的状态设置为**UP**，最后又通过方法**postInit**来开启一个定时任务，用于每隔一段时间（默认60秒）将没有续约的服务实例（默认90秒没有续约）清理掉。**openForTraffic**的方法代码如下：

```
@Override
public void openForTraffic(ApplicationInfoManager applicationInfoManager, int count) {
    // Renewals happen every 30 seconds and for a minute it should be a factor of 2.
    // 计算每分钟最大续约数
    this.expectedNumberOfRenewsPerMin = count * 2;
    // 计算每分钟最小续约数
    this.numberOfRenewsPerMinThreshold =
            (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
    logger.info("Got {} instances from neighboring DS node", count);
    logger.info("Renew threshold is: {}", numberOfRenewsPerMinThreshold);
    this.startupTime = System.currentTimeMillis();
    if (count > 0) {
        this.peerInstancesTransferEmptyOnStartup = false;
    }
    DataCenterInfo.Name selfName = applicationInfoManager.getInfo().getDataCenterInfo().getName();
    boolean isAws = Name.Amazon == selfName;
    if (isAws && serverConfig.shouldPrimeAwsReplicaConnections()) {
        logger.info("Priming AWS connections for all replicas..");
        primeAwsReplicas(applicationInfoManager);
    }
    logger.info("Changing status to UP");
    // 修改服务实例的状态为UP
    applicationInfoManager.setInstanceStatus(InstanceStatus.UP);
    // 开启定时任务，每隔一段时间（默认60秒）将没有续约的服务实例（默认90秒没有续约）清理掉
    super.postInit();
}
```

`postInit`方法开启了一个新的定时任务，代码如下：

```
protected void postInit() {
    renewsLastMin.start();
    if (evictionTaskRef.get() != null) {
        evictionTaskRef.get().cancel();
    }
    evictionTaskRef.set(new EvictionTask());
    evictionTimer.schedule(evictionTaskRef.get(),
            serverConfig.getEvictionIntervalTimerInMs(),
            serverConfig.getEvictionIntervalTimerInMs());
}
```

这里的时间间隔都来自于EurekaServerConfigBean类，可以在配置文件中以eureka.server开头的配置来进行设置。

当然，服务注册中心启动的源码不仅仅只有这么多，其还有向其他集群中的服务注册中心复制服务实例列表的相关源码没有在这里进行分析，感兴趣的朋友可以自行断点分析。

## 1.2.Eureka Client服务注册行为源码分析

我们启动一个本地的服务注册中心，然后再启动一个单节点的服务提供者，我们都知道，在服务注册中心已经启动情况下，然后再启动服务提供者，服务提供者会将服务注册到服务注册中心，那么这个注册行为是如何运作的呢？我们都知道，服务注册行为是在服务提供者启动过程中完成的，那么我们完全可以从启动日志中揣摩出注册行为，请看下面服务提供者的启动日志：

```
Setting initial instance status as: STARTING
2018-12-01 15:37:17.868  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Initializing Eureka in region us-east-1
2018-12-01 15:37:18.031  INFO 31948 --- [  restartedMain] c.n.d.provider.DiscoveryJerseyProvider   : Using JSON encoding codec LegacyJacksonJson
2018-12-01 15:37:18.031  INFO 31948 --- [  restartedMain] c.n.d.provider.DiscoveryJerseyProvider   : Using JSON decoding codec LegacyJacksonJson
2018-12-01 15:37:18.168  INFO 31948 --- [  restartedMain] c.n.d.provider.DiscoveryJerseyProvider   : Using XML encoding codec XStreamXml
2018-12-01 15:37:18.168  INFO 31948 --- [  restartedMain] c.n.d.provider.DiscoveryJerseyProvider   : Using XML decoding codec XStreamXml
2018-12-01 15:37:18.370  INFO 31948 --- [  restartedMain] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Disable delta property : false
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Single vip registry refresh property : null
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Force full registry fetch : false
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Application is null : false
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Registered Applications size is zero : true
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Application version is -1: true
2018-12-01 15:37:18.387  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Getting all instance registry info from the eureka server
2018-12-01 15:37:18.539  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : The response status is 200
2018-12-01 15:37:18.541  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Starting heartbeat executor: renew interval is: 30
2018-12-01 15:37:18.543  INFO 31948 --- [  restartedMain] c.n.discovery.InstanceInfoReplicator     : InstanceInfoReplicator onDemand update allowed rate per min is 4
2018-12-01 15:37:18.548  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Discovery Client initialized at timestamp 1543649838546 with initial instances count: 1
2018-12-01 15:37:18.554  INFO 31948 --- [  restartedMain] o.s.c.n.e.s.EurekaServiceRegistry        : Registering application PRODUCER-SERVICE with eureka with status UP
2018-12-01 15:37:18.556  INFO 31948 --- [  restartedMain] com.netflix.discovery.DiscoveryClient    : Saw local status change event StatusChangeEvent [timestamp=1543649838556, current=UP, previous=STARTING]
2018-12-01 15:37:18.557  INFO 31948 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_PRODUCER-SERVICE/producer-service:-1612568227: registering service...
2018-12-01 15:37:18.627  INFO 31948 --- [nfoReplicator-0] com.netflix.discovery.DiscoveryClient    : DiscoveryClient_PRODUCER-SERVICE/producer-service:-1612568227 - registration status: 204
2018-12-01 15:37:18.645  INFO 31948 --- [  restartedMain] o.s.b.w.embedded.tomcat.TomcatWebServer  : Tomcat started on port(s): 13226 (http) with context path ''
2018-12-01 15:37:18.647  INFO 31948 --- [  restartedMain] .s.c.n.e.s.EurekaAutoServiceRegistration : Updating port to 13226
2018-12-01 15:37:18.654  INFO 31948 --- [  restartedMain] ringCloudEurekaProducerClientApplication : Started SpringCloudEurekaProducerClientApplication in 5.537 seconds (JVM running for 7.0)
2018-12-01 15:42:18.402  INFO 31948 --- [trap-executor-0] c.n.d.s.r.aws.ConfigClusterResolver      : Resolving eureka endpoints via configuration
```

**从日志中我们可以读取到许多信息:**

第一行日志告诉我们，服务提供者实例的状态被标注为“正在启动”。

第二行日志告诉我们，在默认的名为“**us-east-1**”的**Region**中初始化**Eureka**客户端，**Region**的名称是可以配置的，可以通过**eureka.client.region**来配置，如果没有配置它，那么默认的**Region**就是**us-east-1**。这里顺便多说一句，一个微服务应用只可以注册到一个Region中，也就是说一个微服务应用对应一个**Region**，一个**Region**对应多个**Zone**，是否还记得，我们在配置集群的**Eureka Server**服务注册中心的时候，都设置了**eureka.client.service-url.defaultZone**这个值，就是为了告诉服务提供者者或者集群内的其他**Eureka Server**，可以向这个Zone注册，并且defaultZone的值是使用逗号隔开的，也就是说我们的服务可以同时向多个Zone注册。由此可见，一个服务可以同时注册到一个Region中的多个Zone的。如果需要自己指定Zone，可以通过eureka.client.availability-zones来指定。关于Region和Zone请看下面的源码：

```
public static String getRegion(EurekaClientConfig clientConfig) {
    String region = clientConfig.getRegion();
    if (region == null) {
        region = DEFAULT_REGION;
    }
    region = region.trim().toLowerCase();
    return region;
}
```

```
public String[] getAvailabilityZones(String region) {
    String value = this.availabilityZones.get(region);
    if (value == null) {
        value = DEFAULT_ZONE;
    }
    return value.split(",");
}
```

* **日志中getting all instance registry info from the eureka server表示服务在注册的过程中也会向服务注册中心获取其他服务实例的信息列表。**

* **日志中Starting heartbeat executor: renew interval is: 30表示以默认的30秒作为间隔向服务注册中心发起心跳请求，告诉服务注册中心“我还活着”。**

* **日志中Discovery Client initialized at timestamp 1543649838546 with initial instances count: 1表示在时间戳1543649838546的时候，服务成功初始化完成。**

* **日志中DiscoveryClient\_PRODUCER-SERVICE/producer-service:-1612568227: registering service...表示开始将服务注册到服务注册中心。**

* **日志中DiscoveryClient\_PRODUCER-SERVICE/producer-service:-1612568227 - registration status: 204表示服务注册完成，完成的状态标志为204。**

接下来，我们进入到源码中，借助源代码来分析一下服务注册到服务注册中心的流程。在分析之前，我们有必要搞清楚Spring Cloud是如何集成Eureka的，

我们都知道，在Eureka客户端，无论是服务提供者还是服务消费者，都需要加上@EnableDiscoveryClient注解，用来开启DiscoveryClient实例，我们通过搜索DiscoveryClient，可以发现，搜索的结果有一个接口还有一个类，

接口在包org.springframework.cloud.client.discovery下，类在com.netflix.discovery包下，

接口DiscoveryClient是Spring Cloud的接口，它定义了用来发现服务的常用方法，通过该接口可以有效地屏蔽服务治理中的实现细节，这就方便切换不同的服务服务治理框架，而无需改动从Spring Cloud层面调用的代码，

该接口有一个实现类EurekaDiscoveryClient，从命名可以可以看出他是对Eureka服务发现的封装，进入到EurekaDiscoveryClient可以看到，它有一个成员变量为EurekaClient，这是包com.netflix.discovery下的一个接口，

该接口继承了LookupService接口，且有一个实现类DiscoveryClient，接口EurekaClient和LookupService都在com.netflix.discovery包下，他们都定义了针对Eureka的服务发现的抽象方法，而EurekaClient的实现类DiscoveryClient则实现了这些抽象方法，所以说，类DiscoveryClient是真正实现发现服务的类。结合以上的文字，下面展示接口与类的关系图如下所示：

![img](/static/image/20181211200740761.jpg)

我们进入到DiscoveryClient类中查看源码，首先看到的是它的类注释如下所示：

```
/**
 * The class that is instrumental for interactions with <tt>Eureka Server</tt>.
 *
 * <p>
 * <tt>Eureka Client</tt> is responsible for a) <em>Registering</em> the
 * instance with <tt>Eureka Server</tt> b) <em>Renewal</em>of the lease with
 * <tt>Eureka Server</tt> c) <em>Cancellation</em> of the lease from
 * <tt>Eureka Server</tt> during shutdown
 * <p>
 * d) <em>Querying</em> the list of services/instances registered with
 * <tt>Eureka Server</tt>
 * <p>
 *
 * <p>
 * <tt>Eureka Client</tt> needs a configured list of <tt>Eureka Server</tt>
 * {@link java.net.URL}s to talk to.These {@link java.net.URL}s are typically amazon elastic eips
 * which do not change. All of the functions defined above fail-over to other
 * {@link java.net.URL}s specified in the list in the case of failure.
 * </p>
 *
 * @author Karthik Ranganathan, Greg Kim
 * @author Spencer Gibb
 *
 */
```

这个类用于帮助与Eureka Server相互协作。

Eureka Client负责下面的任务：

* 向Eureka Server注册服务实例

* 向Eureka Server服务续约

* 当服务关闭期间，向Eureka Server取消租约

* 查询Eureka Server中的服务实例列表

Eureka Client还需要配置一个Eureka Server的URL列表

在分析类DiscoveryClient完成的具体任务之前，我们首先来回忆一下，我们在配置服务提供者的时候，在配置文件中都配置了eureka.client.service-url.defaultZone属性，而这个属性的值就是告诉服务提供者，该向哪里注册服务，也就是服务注册的地址，该地址比如是[http://peer1:1111/eureka/,http://peer2:1112/eureka/,http://peer3:1113/eureka/，各个地址之间使用逗号隔开，我们在类EndpointUtils中可以找到一个方法名为getServiceUrlsMapFromConfig的方法，代码如下：](http://peer1:1111/eureka/,http://peer2:1112/eureka/,http://peer3:1113/eureka/，各个地址之间使用逗号隔开，我们在类EndpointUtils中可以找到一个方法名为getServiceUrlsMapFromConfig的方法，代码如下：)

```
public static Map<String, List<String>> getServiceUrlsMapFromConfig(EurekaClientConfig clientConfig, String instanceZone, boolean preferSameZone) {
    Map<String, List<String>> orderedUrls = new LinkedHashMap<>();
    String region = getRegion(clientConfig);
    String[] availZones = clientConfig.getAvailabilityZones(clientConfig.getRegion());
    if (availZones == null || availZones.length == 0) {
        availZones = new String[1];
        availZones[0] = DEFAULT_ZONE;
    }
    logger.debug("The availability zone for the given region {} are {}", region, availZones);
    int myZoneOffset = getZoneOffset(instanceZone, preferSameZone, availZones);

    String zone = availZones[myZoneOffset];
    List<String> serviceUrls = clientConfig.getEurekaServerServiceUrls(zone);
    if (serviceUrls != null) {
        orderedUrls.put(zone, serviceUrls);
    }
    int currentOffset = myZoneOffset == (availZones.length - 1) ? 0 : (myZoneOffset + 1);
    while (currentOffset != myZoneOffset) {
        zone = availZones[currentOffset];
        serviceUrls = clientConfig.getEurekaServerServiceUrls(zone);
        if (serviceUrls != null) {
            orderedUrls.put(zone, serviceUrls);
        }
        if (currentOffset == (availZones.length - 1)) {
            currentOffset = 0;
        } else {
            currentOffset++;
        }
    }

    if (orderedUrls.size() < 1) {
        throw new IllegalArgumentException("DiscoveryClient: invalid serviceUrl specified!");
    }
    return orderedUrls;
}
```

该方法就是从我们配置的`Zone`中读取注册地址，并组成一个`List`，最后将这个`List`存储到`Map`集合中，在读取过程中，它首先加载的是`getRegion`方法，这个方法读取了一个`Region`返回，进入到`getRegion`方法中：

```
/**
 * Get the region that this particular instance is in.
 *
 * @return - The region in which the particular instance belongs to.
 */
public static String getRegion(EurekaClientConfig clientConfig) {
        String region = clientConfig.getRegion();
        if (region == null) {
            region = DEFAULT_REGION;
        }
        region = region.trim().toLowerCase();
        return region;
    }
```

从方法的注释上可以知道，一个微服务应用只可以属于一个Region，方法体中的第一行代码就是从EurekaClientConfigBean类中来读取Region，而EurekaClientConfigBean中getRegion方法返回的值就是需要我们配置的，在配置文件中，对应的属性是eureka.client.region，如果我们每月配置，那么将使用默认的Region，默认DEFAULT\_REGION为default。在方法getServiceUrlsMapFromConfig中，还加载了getAvailabilityZones方法，方法代码如下所示：

```
public String[] getAvailabilityZones(String region) {
    String value = this.availabilityZones.get(region);
    if (value == null) {
        value = DEFAULT_ZONE;
    }
    return value.split(",");
}
```

上述方法中第一行代码是从Region中获取Zone，availabilityZones是EurekaClientConfigBean的一个Map成员变量，如果我们没有为Region特别配置eureka.client.availablity-zones属性，那么zone将采用默认值，默认值是defaultZone，这就是我们一开始配置eureka.client.service-url.defaultZone的由来，由此可见，一个Region对应多个Zone，也就是说一个微服务应用可以向多个服务注册地址注册。在获取了Region和Zone的信息之后，才开始真正地加载Eureka Server的具体地址，它根据传入的参数按照一定的算法确定加载位于哪一个Zone配置的serviceUrls，代码如下：

```
int myZoneOffset = getZoneOffset(instanceZone, preferSameZone, availZones);
String zone = availZones[myZoneOffset];
List<String> serviceUrls = clientConfig.getEurekaServerServiceUrls(zone);
```

当我们在微服务应用中使用Ribbon来实现服务调用时，对于Zone的设置可以在负载均衡时实现区域亲和特性：Ribbon的默认策略会优先访问同客户端处于一个Zone中的服务实例，只有当同一个Zone中没有可用的服务实例的时候才会去访问其他Zone中的实例。利用亲和性这一特性，我们就可以有效地设计出针对区域性故障的容错集群。

从本小节一开始，我们就分析了Spring Cloud Eureka是对Netflix Eureka的封装，com.netflix.discovery包下的DiscoveryClient才是真正实现服务的注册与发现。我们一起来看看它的构造方法：

```
@Inject
DiscoveryClient(ApplicationInfoManager applicationInfoManager, EurekaClientConfig config, AbstractDiscoveryClientOptionalArgs args,
                Provider<BackupRegistry> backupRegistryProvider) {
    if (args != null) {
        this.healthCheckHandlerProvider = args.healthCheckHandlerProvider;
        this.healthCheckCallbackProvider = args.healthCheckCallbackProvider;
        this.eventListeners.addAll(args.getEventListeners());
        this.preRegistrationHandler = args.preRegistrationHandler;
    } else {
        this.healthCheckCallbackProvider = null;
        this.healthCheckHandlerProvider = null;
        this.preRegistrationHandler = null;
    }

    this.applicationInfoManager = applicationInfoManager;
    InstanceInfo myInfo = applicationInfoManager.getInfo();

    clientConfig = config;
    staticClientConfig = clientConfig;
    transportConfig = config.getTransportConfig();
    instanceInfo = myInfo;
    if (myInfo != null) {
        appPathIdentifier = instanceInfo.getAppName() + "/" + instanceInfo.getId();
    } else {
        logger.warn("Setting instanceInfo to a passed in null value");
    }

    this.backupRegistryProvider = backupRegistryProvider;

    this.urlRandomizer = new EndpointUtils.InstanceInfoBasedUrlRandomizer(instanceInfo);
    localRegionApps.set(new Applications());

    fetchRegistryGeneration = new AtomicLong(0);

    remoteRegionsToFetch = new AtomicReference<String>(clientConfig.fetchRegistryForRemoteRegions());
    remoteRegionsRef = new AtomicReference<>(remoteRegionsToFetch.get() == null ? null : remoteRegionsToFetch.get().split(","));

    if (config.shouldFetchRegistry()) {
        this.registryStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRY_PREFIX + "lastUpdateSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
    } else {
        this.registryStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
    }

    if (config.shouldRegisterWithEureka()) {
        this.heartbeatStalenessMonitor = new ThresholdLevelsMetric(this, METRIC_REGISTRATION_PREFIX + "lastHeartbeatSec_", new long[]{15L, 30L, 60L, 120L, 240L, 480L});
    } else {
        this.heartbeatStalenessMonitor = ThresholdLevelsMetric.NO_OP_METRIC;
    }

    logger.info("Initializing Eureka in region {}", clientConfig.getRegion());

    if (!config.shouldRegisterWithEureka() && !config.shouldFetchRegistry()) {
        logger.info("Client configured to neither register nor query for data.");
        scheduler = null;
        heartbeatExecutor = null;
        cacheRefreshExecutor = null;
        eurekaTransport = null;
        instanceRegionChecker = new InstanceRegionChecker(new PropertyBasedAzToRegionMapper(config), clientConfig.getRegion());

        // This is a bit of hack to allow for existing code using DiscoveryManager.getInstance()
        // to work with DI'd DiscoveryClient
        DiscoveryManager.getInstance().setDiscoveryClient(this);
        DiscoveryManager.getInstance().setEurekaClientConfig(config);

        initTimestampMs = System.currentTimeMillis();
        logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}",
                initTimestampMs, this.getApplications().size());

        return;  // no need to setup up an network tasks and we are done
    }

    try {
        // default size of 2 - 1 each for heartbeat and cacheRefresh
        scheduler = Executors.newScheduledThreadPool(2,
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-%d")
                        .setDaemon(true)
                        .build());

        heartbeatExecutor = new ThreadPoolExecutor(
                1, clientConfig.getHeartbeatExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>(),
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-HeartbeatExecutor-%d")
                        .setDaemon(true)
                        .build()
        );  // use direct handoff

        cacheRefreshExecutor = new ThreadPoolExecutor(
                1, clientConfig.getCacheRefreshExecutorThreadPoolSize(), 0, TimeUnit.SECONDS,
                new SynchronousQueue<Runnable>(),
                new ThreadFactoryBuilder()
                        .setNameFormat("DiscoveryClient-CacheRefreshExecutor-%d")
                        .setDaemon(true)
                        .build()
        );  // use direct handoff

        eurekaTransport = new EurekaTransport();
        scheduleServerEndpointTask(eurekaTransport, args);

        AzToRegionMapper azToRegionMapper;
        if (clientConfig.shouldUseDnsForFetchingServiceUrls()) {
            azToRegionMapper = new DNSBasedAzToRegionMapper(clientConfig);
        } else {
            azToRegionMapper = new PropertyBasedAzToRegionMapper(clientConfig);
        }
        if (null != remoteRegionsToFetch.get()) {
            azToRegionMapper.setRegionsToFetch(remoteRegionsToFetch.get().split(","));
        }
        instanceRegionChecker = new InstanceRegionChecker(azToRegionMapper, clientConfig.getRegion());
    } catch (Throwable e) {
        throw new RuntimeException("Failed to initialize DiscoveryClient!", e);
    }

    if (clientConfig.shouldFetchRegistry() && !fetchRegistry(false)) {
        fetchRegistryFromBackup();
    }

    // call and execute the pre registration handler before all background tasks (inc registration) is started
    if (this.preRegistrationHandler != null) {
        this.preRegistrationHandler.beforeRegistration();
    }

    if (clientConfig.shouldRegisterWithEureka() && clientConfig.shouldEnforceRegistrationAtInit()) {
        try {
            if (!register() ) {
                throw new IllegalStateException("Registration error at startup. Invalid server response.");
            }
        } catch (Throwable th) {
            logger.error("Registration error at startup: {}", th.getMessage());
            throw new IllegalStateException(th);
        }
    }

    // finally, init the schedule tasks (e.g. cluster resolvers, heartbeat, instanceInfo replicator, fetch
    // 这个方法里实现了服务向服务注册中心注册的行为
    initScheduledTasks();

    try {
        Monitors.registerObject(this);
    } catch (Throwable e) {
        logger.warn("Cannot register timers", e);
    }

    // This is a bit of hack to allow for existing code using DiscoveryManager.getInstance()
    // to work with DI'd DiscoveryClient
    DiscoveryManager.getInstance().setDiscoveryClient(this);
    DiscoveryManager.getInstance().setEurekaClientConfig(config);

    initTimestampMs = System.currentTimeMillis();
    logger.info("Discovery Client initialized at timestamp {} with initial instances count: {}",
            initTimestampMs, this.getApplications().size());
}
```

整个构造方法里，一开始进行了各种参数的设置，而真正地注册行为是在`initScheduledTasks`方法里实现的，我们一起来看看注册行为是如何实现的：

```
private void initScheduledTasks() {
    if (clientConfig.shouldFetchRegistry()) {
        // registry cache refresh timer
        int registryFetchIntervalSeconds = clientConfig.getRegistryFetchIntervalSeconds();
        int expBackOffBound = clientConfig.getCacheRefreshExecutorExponentialBackOffBound();
        scheduler.schedule(
                new TimedSupervisorTask(
                        "cacheRefresh",
                        scheduler,
                        cacheRefreshExecutor,
                        registryFetchIntervalSeconds,
                        TimeUnit.SECONDS,
                        expBackOffBound,
                        new CacheRefreshThread()
                ),
                registryFetchIntervalSeconds, TimeUnit.SECONDS);
    }

    if (clientConfig.shouldRegisterWithEureka()) {
        int renewalIntervalInSecs = instanceInfo.getLeaseInfo().getRenewalIntervalInSecs();
        int expBackOffBound = clientConfig.getHeartbeatExecutorExponentialBackOffBound();
        logger.info("Starting heartbeat executor: " + "renew interval is: {}", renewalIntervalInSecs);

        // Heartbeat timer
        scheduler.schedule(
                new TimedSupervisorTask(
                        "heartbeat",
                        scheduler,
                        heartbeatExecutor,
                        renewalIntervalInSecs,
                        TimeUnit.SECONDS,
                        expBackOffBound,
                        new HeartbeatThread()
                ),
                renewalIntervalInSecs, TimeUnit.SECONDS);

        // InstanceInfo replicator
        instanceInfoReplicator = new InstanceInfoReplicator(
                this,
                instanceInfo,
                clientConfig.getInstanceInfoReplicationIntervalSeconds(),
                2); // burstSize

        statusChangeListener = new ApplicationInfoManager.StatusChangeListener() {
            @Override
            public String getId() {
                return "statusChangeListener";
            }

            @Override
            public void notify(StatusChangeEvent statusChangeEvent) {
                if (InstanceStatus.DOWN == statusChangeEvent.getStatus() ||
                        InstanceStatus.DOWN == statusChangeEvent.getPreviousStatus()) {
                    // log at warn level if DOWN was involved
                    logger.warn("Saw local status change event {}", statusChangeEvent);
                } else {
                    logger.info("Saw local status change event {}", statusChangeEvent);
                }
                instanceInfoReplicator.onDemandUpdate();
            }
        };

        if (clientConfig.shouldOnDemandUpdateStatusChange()) {
            applicationInfoManager.registerStatusChangeListener(statusChangeListener);
        }

        instanceInfoReplicator.start(clientConfig.getInitialInstanceInfoReplicationIntervalSeconds());
    } else {
        logger.info("Not registering with Eureka server per configuration");
    }
}
```

这段代码中有两个主要的if代码块，第一个if代码块是决定是否从Eureka Server来获取注册信息，判断条件clientConfig.shouldFetchRegistry\(\)是需要我们自己的在配置文件中通过属性eureka.client.fetch-registry=true进行配置的，默认为true，也就是说服务会从Eureka Server拉取注册信息，且默认间隔为30秒，每30秒执行一次定时任务，用于刷新所获取的注册信息。

第二个if代码块是决定是否将服务注册到服务注册中心的，也是我们本次要探讨的主要内容。判断条件clientConfig.shouldRegisterWithEureka\(\)表示是否向Eureka Server注册自己，你是否还记得，我们在搭建单节点服务注册中心的时候，我们搭建的那个Eureka Server设置了属性eureka.client.register-with-eureka=false，意思就是说禁止Eureka Server把自己当做一个普通服务注册到自身，而这个属性默认值也是为true，也就是说我们在注册的服务的时候，无需配置这个属性，就可以将服务注册到服务注册中心。分析第二个if代码块，代码块中一开始就设置了一个定时任务，这个定时任务就是按照指定的时间间隔向Eureka Server发送心跳，告诉服务注册中心“我还活着”，对于发送心跳的时间间隔，我们一开始就讨论过，默认是30秒，这也就是为什么按照默认来说，一分钟理应发送两次心跳了，这个心跳间隔我们可以在配置文件中进行配置，配置属性为eureka.instance.lease-renewal-interval-in-seconds=30，对于默认90秒内没有发送心跳的服务，将会被服务在服务注册中心剔除，剔除时间间隔可以通过属性eureka.instance.lease-expiration-duration-in-seconds=90来进行配置。而整个服务的续约逻辑也很简单，在定时任务中有一个代码片段new HeartbeatThread\(\)，然后开启了一个新的线程实现续约服务，就是通过发送REST请求来实现的，具体代码如下：

```
/**
 * Renew with the eureka service by making the appropriate REST call
 */
boolean renew() {
    EurekaHttpResponse<InstanceInfo> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.sendHeartBeat(instanceInfo.getAppName(), instanceInfo.getId(), instanceInfo, null);
        logger.debug(PREFIX + "{} - Heartbeat status: {}", appPathIdentifier, httpResponse.getStatusCode());
        if (httpResponse.getStatusCode() == 404) {
            REREGISTER_COUNTER.increment();
            logger.info(PREFIX + "{} - Re-registering apps/{}", appPathIdentifier, instanceInfo.getAppName());
            long timestamp = instanceInfo.setIsDirtyWithTime();
            boolean success = register();
            if (success) {
                instanceInfo.unsetIsDirty(timestamp);
            }
            return success;
        }
        return httpResponse.getStatusCode() == 200;
    } catch (Throwable e) {
        logger.error(PREFIX + "{} - was unable to send heartbeat!", appPathIdentifier, e);
        return false;
    }
}
```

服务提供者向服务注册中心发送心跳，并检查返回码，如果是404，那么服务将重新调用register方法，实现将服务注册到服务注册中心，否则直接检测返回码是否是200，返回布尔类型来告诉定时器是否续约成功。

续约的操作完成之后，就开始了服务实例的复制工作，紧接着通过服务实例管理器ApplicationInfoManager来创建一个服务实例状态监听器，用于监听服务实例的状态，并进入到onDemandUpdate中进行注册，方法onDemandUpdate的代码如下：

```
public boolean onDemandUpdate() {
    if (rateLimiter.acquire(burstSize, allowedRatePerMinute)) {
        if (!scheduler.isShutdown()) {
            scheduler.submit(new Runnable() {
                @Override
                public void run() {
                    logger.debug("Executing on-demand update of local InstanceInfo");

                    Future latestPeriodic = scheduledPeriodicRef.get();
                    if (latestPeriodic != null && !latestPeriodic.isDone()) {
                        logger.debug("Canceling the latest scheduled update, it will be rescheduled at the end of on demand update");
                        latestPeriodic.cancel(false);
                    }

                    InstanceInfoReplicator.this.run();
                }
            });
            return true;
        } else {
            logger.warn("Ignoring onDemand update due to stopped scheduler");
            return false;
        }
    } else {
        logger.warn("Ignoring onDemand update due to rate limiter");
        return false;
    }
}

public void run() {
    try {
        discoveryClient.refreshInstanceInfo();

        Long dirtyTimestamp = instanceInfo.isDirtyWithTime();
        if (dirtyTimestamp != null) {
            discoveryClient.register();
            instanceInfo.unsetIsDirty(dirtyTimestamp);
        }
    } catch (Throwable t) {
        logger.warn("There was a problem with the instance info replicator", t);
    } finally {
        Future next = scheduler.schedule(this, replicationIntervalSeconds, TimeUnit.SECONDS);
        scheduledPeriodicRef.set(next);
    }
}
```

服务实例的注册行为是在方法`register`中执行的，进入到`register`方法中，代码如下：

```
/**
 * Register with the eureka service by making the appropriate REST call.
 */
boolean register() throws Throwable {
    logger.info(PREFIX + "{}: registering service...", appPathIdentifier);
    EurekaHttpResponse<Void> httpResponse;
    try {
        httpResponse = eurekaTransport.registrationClient.register(instanceInfo);
    } catch (Exception e) {
        logger.warn(PREFIX + "{} - registration failed {}", appPathIdentifier, e.getMessage(), e);
        throw e;
    }
    if (logger.isInfoEnabled()) {
        logger.info(PREFIX + "{} - registration status: {}", appPathIdentifier, httpResponse.getStatusCode());
    }
    return httpResponse.getStatusCode() == 204;
}
```

注册行为其实就是将服务实例的信息通过HTTP请求传递给Eureka Server服务注册中心，当注册中心接收到注册信息之后将返回204状态码给客户端，表示注册成功。这就是客户端向服务注册中心注册的行为源码分析，那么服务注册中心是如何接收这些实例的注册信息，且如何保存的呢？请接着往下看。

## 1.3.Eureka Server服务注册中心接收注册行为分析

客户端向服务端发起注册请求之后，服务端是如何处理的呢？通过源码的分析，可以发现，客户端和服务端的交互都是通过REST请求发起的，而在服务端，包com.netflix.eureka.resources下定义了许多处理REST请求的类，对于接收客户端的注册信息，可以发现在类ApplicationResource下有一个addInstance方法，专门用来处理注册请求的，我们一起来分析这个方法：

```
/**
 * Registers information about a particular instance for an
 * {@link com.netflix.discovery.shared.Application}.
 *
 * @param info
 *            {@link InstanceInfo} information of the instance.
 * @param isReplication
 *            a header parameter containing information whether this is
 *            replicated from other nodes.
 */
@POST
@Consumes({"application/json", "application/xml"})
public Response addInstance(InstanceInfo info,
                            @HeaderParam(PeerEurekaNode.HEADER_REPLICATION) String isReplication) {
    logger.debug("Registering instance {} (replication={})", info.getId(), isReplication);
    // validate that the instanceinfo contains all the necessary required fields
    if (isBlank(info.getId())) {
        return Response.status(400).entity("Missing instanceId").build();
    } else if (isBlank(info.getHostName())) {
        return Response.status(400).entity("Missing hostname").build();
    } else if (isBlank(info.getIPAddr())) {
        return Response.status(400).entity("Missing ip address").build();
    } else if (isBlank(info.getAppName())) {
        return Response.status(400).entity("Missing appName").build();
    } else if (!appName.equals(info.getAppName())) {
        return Response.status(400).entity("Mismatched appName, expecting " + appName + " but was " + info.getAppName()).build();
    } else if (info.getDataCenterInfo() == null) {
        return Response.status(400).entity("Missing dataCenterInfo").build();
    } else if (info.getDataCenterInfo().getName() == null) {
        return Response.status(400).entity("Missing dataCenterInfo Name").build();
    }

    // handle cases where clients may be registering with bad DataCenterInfo with missing data
    DataCenterInfo dataCenterInfo = info.getDataCenterInfo();
    if (dataCenterInfo instanceof UniqueIdentifier) {
        String dataCenterInfoId = ((UniqueIdentifier) dataCenterInfo).getId();
        if (isBlank(dataCenterInfoId)) {
            boolean experimental = "true".equalsIgnoreCase(serverConfig.getExperimental("registration.validation.dataCenterInfoId"));
            if (experimental) {
                String entity = "DataCenterInfo of type " + dataCenterInfo.getClass() + " must contain a valid id";
                return Response.status(400).entity(entity).build();
            } else if (dataCenterInfo instanceof AmazonInfo) {
                AmazonInfo amazonInfo = (AmazonInfo) dataCenterInfo;
                String effectiveId = amazonInfo.get(AmazonInfo.MetaDataKey.instanceId);
                if (effectiveId == null) {
                    amazonInfo.getMetadata().put(AmazonInfo.MetaDataKey.instanceId.getName(), info.getId());
                }
            } else {
                logger.warn("Registering DataCenterInfo of type {} without an appropriate id", dataCenterInfo.getClass());
            }
        }
    }

    registry.register(info, "true".equals(isReplication));
    return Response.status(204).build();  // 204 to be backwards compatible
}
```

在接收服务实例注册的时候，首先要经过一系列的数据校验，通过校验以后调用`PeerAwareInstanceRegistry`的实现类对象的

`register`方法对服务进行注册，进入到`register`方法继续分析：

```
@Override
public void register(final InstanceInfo info, final boolean isReplication) {
    handleRegistration(info, resolveInstanceLeaseDuration(info), isReplication);
    super.register(info, isReplication);
}
```

方法体中第一行代码中调用了publishEvent方法，将注册事件传播出去，然后继续调用com.netflix.eureka.registry包下的AbstractInstanceRegistry抽象类的register方法进行注册：

```
/**
 * Registers a new instance with a given duration.
 *
 * @see com.netflix.eureka.lease.LeaseManager#register(java.lang.Object, int, boolean)
 */
public void register(InstanceInfo registrant, int leaseDuration, boolean isReplication) {
    try {
        read.lock();
        Map<String, Lease<InstanceInfo>> gMap = registry.get(registrant.getAppName());
        REGISTER.increment(isReplication);
        if (gMap == null) {
            final ConcurrentHashMap<String, Lease<InstanceInfo>> gNewMap = new ConcurrentHashMap<String, Lease<InstanceInfo>>();
            gMap = registry.putIfAbsent(registrant.getAppName(), gNewMap);
            if (gMap == null) {
                gMap = gNewMap;
            }
        }
        Lease<InstanceInfo> existingLease = gMap.get(registrant.getId());
        // Retain the last dirty timestamp without overwriting it, if there is already a lease
        if (existingLease != null && (existingLease.getHolder() != null)) {
            Long existingLastDirtyTimestamp = existingLease.getHolder().getLastDirtyTimestamp();
            Long registrationLastDirtyTimestamp = registrant.getLastDirtyTimestamp();
            logger.debug("Existing lease found (existing={}, provided={}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);

            // this is a > instead of a >= because if the timestamps are equal, we still take the remote transmitted
            // InstanceInfo instead of the server local copy.
            if (existingLastDirtyTimestamp > registrationLastDirtyTimestamp) {
                logger.warn("There is an existing lease and the existing lease's dirty timestamp {} is greater" +
                        " than the one that is being registered {}", existingLastDirtyTimestamp, registrationLastDirtyTimestamp);
                logger.warn("Using the existing instanceInfo instead of the new instanceInfo as the registrant");
                registrant = existingLease.getHolder();
            }
        } else {
            // The lease does not exist and hence it is a new registration
            synchronized (lock) {
                if (this.expectedNumberOfRenewsPerMin > 0) {
                    // Since the client wants to cancel it, reduce the threshold
                    // (1
                    // for 30 seconds, 2 for a minute)
                    this.expectedNumberOfRenewsPerMin = this.expectedNumberOfRenewsPerMin + 2;
                    this.numberOfRenewsPerMinThreshold =
                            (int) (this.expectedNumberOfRenewsPerMin * serverConfig.getRenewalPercentThreshold());
                }
            }
            logger.debug("No previous lease information found; it is new registration");
        }
        Lease<InstanceInfo> lease = new Lease<InstanceInfo>(registrant, leaseDuration);
        if (existingLease != null) {
            lease.setServiceUpTimestamp(existingLease.getServiceUpTimestamp());
        }
        gMap.put(registrant.getId(), lease);
        synchronized (recentRegisteredQueue) {
            recentRegisteredQueue.add(new Pair<Long, String>(
                    System.currentTimeMillis(),
                    registrant.getAppName() + "(" + registrant.getId() + ")"));
        }
        // This is where the initial state transfer of overridden status happens
        if (!InstanceStatus.UNKNOWN.equals(registrant.getOverriddenStatus())) {
            logger.debug("Found overridden status {} for instance {}. Checking to see if needs to be add to the "
                            + "overrides", registrant.getOverriddenStatus(), registrant.getId());
            if (!overriddenInstanceStatusMap.containsKey(registrant.getId())) {
                logger.info("Not found overridden id {} and hence adding it", registrant.getId());
                overriddenInstanceStatusMap.put(registrant.getId(), registrant.getOverriddenStatus());
            }
        }
        InstanceStatus overriddenStatusFromMap = overriddenInstanceStatusMap.get(registrant.getId());
        if (overriddenStatusFromMap != null) {
            logger.info("Storing overridden status {} from map", overriddenStatusFromMap);
            registrant.setOverriddenStatus(overriddenStatusFromMap);
        }

        // Set the status based on the overridden status rules
        InstanceStatus overriddenInstanceStatus = getOverriddenInstanceStatus(registrant, existingLease, isReplication);
        registrant.setStatusWithoutDirty(overriddenInstanceStatus);

        // If the lease is registered with UP status, set lease service up timestamp
        if (InstanceStatus.UP.equals(registrant.getStatus())) {
            lease.serviceUp();
        }
        registrant.setActionType(ActionType.ADDED);
        recentlyChangedQueue.add(new RecentlyChangedItem(lease));
        registrant.setLastUpdatedTimestamp();
        invalidateCache(registrant.getAppName(), registrant.getVIPAddress(), registrant.getSecureVipAddress());
        logger.info("Registered instance {}/{} with status {} (replication={})",
                registrant.getAppName(), registrant.getId(), registrant.getStatus(), isReplication);
    } finally {
        read.unlock();
    }
```

# 4.参考

[https://blog.csdn.net/Lammonpeter/article/details/8433090](https://blog.csdn.net/Lammonpeter/article/details/84330900)

