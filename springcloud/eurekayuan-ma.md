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

而且，在类**EurekaServerAutoConfiguration**上，我们看见**@Import\(EurekaServerInitializerConfiguration.class\)**，说明实例化类**EurekaServerAutoConfiguration**之前，已经实例化了**EurekaServerInitializerConfiguration**类，

从类名可以看出，该类是**Eureka Server**的初始化配置类，我们进入到**EurekaServerInitializerConfiguration**类中一探究竟，发现该类实现了**Spring**的生命周期接口**SmartLifecycle**，也就是说类**EurekaServerInitializerConfiguration**在被**Spring**实例化过程中的时候会执行一些生命周期方法，比如**Lifecycle**的**start**方法，那么看看**EurekaServerInitializerConfiguration**是如何重写**start**方法的：

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

## 1.这个

`start`

方法中开启了一个新的线程，然后进行一些

`Eureka Server`

的初始化工作，比如调用

`eurekaServe`这个

`start`

方法中开启了一个新的线程，然后进行一些

`Eureka Server`

的初始化工作，比如调用

`eurekaServerBootstrap的contextInitialized`

方法，进入该方法看看：`rBootstrap的contextInitialized`

方法，进入该方法看看：2.Eureka Client服务注册行为源码分析

# 4.参考

[https://blog.csdn.net/Lammonpeter/article/details/84330900](https://blog.csdn.net/Lammonpeter/article/details/84330900)

