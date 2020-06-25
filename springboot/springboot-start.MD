#   资源准备
###   1.spring.factories
spring.factories中内容
-  PropertySource Loaders 资源加载器
-  Run Listeners 运行监听器，发布事件
-  Error Reporters 错误报告
-  Application Context Initializers  初始化应用上下文
-  Application Listeners 应用事件监听器
-  Environment Post Processors 环境后置处理器
-  Failure Analyzers 故障分析
-  FailureAnalysisReporters 故障分析

```
# PropertySource Loaders
org.springframework.boot.env.PropertySourceLoader=\
org.springframework.boot.env.PropertiesPropertySourceLoader,\
org.springframework.boot.env.YamlPropertySourceLoader

# Run Listeners
org.springframework.boot.SpringApplicationRunListener=\
org.springframework.boot.context.event.EventPublishingRunListener

# Error Reporters
org.springframework.boot.SpringBootExceptionReporter=\
org.springframework.boot.diagnostics.FailureAnalyzers

# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener

# Environment Post Processors
org.springframework.boot.env.EnvironmentPostProcessor=\
org.springframework.boot.cloud.CloudFoundryVcapEnvironmentPostProcessor,\
org.springframework.boot.env.SpringApplicationJsonEnvironmentPostProcessor,\
org.springframework.boot.env.SystemEnvironmentPropertySourceEnvironmentPostProcessor

# Failure Analyzers
org.springframework.boot.diagnostics.FailureAnalyzer=\
org.springframework.boot.diagnostics.analyzer.BeanCurrentlyInCreationFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.BeanNotOfRequiredTypeFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.BindFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.BindValidationFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.UnboundConfigurationPropertyFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.ConnectorStartFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.NoSuchMethodFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.NoUniqueBeanDefinitionFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.PortInUseFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.ValidationExceptionFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.InvalidConfigurationPropertyNameFailureAnalyzer,\
org.springframework.boot.diagnostics.analyzer.InvalidConfigurationPropertyValueFailureAnalyzer

# FailureAnalysisReporters
org.springframework.boot.diagnostics.FailureAnalysisReporter=\
org.springframework.boot.diagnostics.LoggingFailureAnalysisReporter

```

####    springboot 事件
启动事件顺序        
- ApplicationStartingEvent
- ApplicationEnvironmentPreparedEvent
- ApplicationContextInitializedEvent
- ApplicationPreparedEvent
- ContextRefreshedEvent
- ServletWebServerInitializedEvent
- ApplicationStartedEvent
- ApplicationReadyEvent

在      
ApplicationEnvironmentPreparedEvent   
事件后执行  
ApplicationContextInitializer
应用程序初始化

### SpringApplication
构造器初始化SpringApplication

```
public SpringApplication(ResourceLoader resourceLoader, Class<?>... primarySources) {
        //设置resourceLoader
		this.resourceLoader = resourceLoader;
		Assert.notNull(primarySources, "PrimarySources must not be null");
		//启动类
		this.primarySources = new LinkedHashSet<>(Arrays.asList(primarySources));
		//判断是否是web程序
		this.webApplicationType = WebApplicationType.deduceFromClasspath();
		//设置初始化启动累
		从spring.factories中获取实现类 设定运行顺讯 并初始化 
        //- ApplicationContextInitializer.class
        //- ApplicationListener.class
		setInitializers((Collection) getSpringFactoriesInstances(
			ApplicationContextInitializer.class));
		//设置监听器		
		setListeners((Collection) getSpringFactoriesInstances(ApplicationListener.class));
		//设置启动类
		this.mainApplicationClass = deduceMainApplicationClass();
	}
```

####    SpringApplication 属性
1.WebApplicationType 应用程序类型

```
private WebApplicationType webApplicationType;
```

An enumeration of possible types of web application.
```
/**
	 * The application should not run as a web application and should not start an
	 * embedded web server.
	 */
	NONE,

	/**
	 * The application should run as a servlet-based web application and should start an
	 * embedded servlet web server.
	 */
	SERVLET,

	/**
	 * The application should run as a reactive web application and should start an
	 * embedded reactive web server.
	 */
	REACTIVE;

    //servlet 指标类 
	private static final String[] SERVLET_INDICATOR_CLASSES = { "javax.servlet.Servlet",
			"org.springframework.web.context.ConfigurableWebApplicationContext" };

    //mvc 指标类 
	private static final String WEBMVC_INDICATOR_CLASS = "org.springframework."
			+ "web.servlet.DispatcherServlet";

    //响应式指标类
	private static final String WEBFLUX_INDICATOR_CLASS = "org."
			+ "springframework.web.reactive.DispatcherHandler";

	private static final String JERSEY_INDICATOR_CLASS = "org.glassfish.jersey.servlet.ServletContainer";

	private static final String SERVLET_APPLICATION_CONTEXT_CLASS = "org.springframework.web.context.WebApplicationContext";

	private static final String REACTIVE_APPLICATION_CONTEXT_CLASS = "org.springframework.boot.web.reactive.context.ReactiveWebApplicationContext";
	
	static WebApplicationType deduceFromClasspath() {
	    //如果有webflux指标类，没有nvc 和 jersey 指标类  则是 响应式
		if (ClassUtils.isPresent(WEBFLUX_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(WEBMVC_INDICATOR_CLASS, null)
				&& !ClassUtils.isPresent(JERSEY_INDICATOR_CLASS, null)) {
			return WebApplicationType.REACTIVE;
		}
		//如果没有servlet 指标类 则不是web程序
		for (String className : SERVLET_INDICATOR_CLASSES) {
			if (!ClassUtils.isPresent(className, null)) {
				return WebApplicationType.NONE;
			}
		}
		//否则是web程序
		return WebApplicationType.SERVLET;
	}

```
2. 应用程序上下文初始化类

```
private List<ApplicationContextInitializer<?>> initializers;
```
准备初始化类，从META-INF/spring.factories"中获取
```
setInitializers((Collection) getSpringFactoriesInstances(
				ApplicationContextInitializer.class));
```


```
/**
	 * Sets the {@link ApplicationContextInitializer} that will be applied to the Spring
	 * {@link ApplicationContext}.
	 * @param initializers the initializers to set
	 */
	public void setInitializers(
			Collection<? extends ApplicationContextInitializer<?>> initializers) {
		this.initializers = new ArrayList<>();
		this.initializers.addAll(initializers);
	}
```
通过类类型获取spring-factories实例
SpringFactoriesInstances
```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type) {
		return getSpringFactoriesInstances(type, new Class<?>[] {});
	}
```
            
```
private <T> Collection<T> getSpringFactoriesInstances(Class<T> type,
			Class<?>[] parameterTypes, Object... args) {
		//获取当前类加载器	
		ClassLoader classLoader = Thread.currentThread().getContextClassLoader();
		// Use names and ensure unique to protect against duplicates
		Set<String> names = new LinkedHashSet<>(
				SpringFactoriesLoader.loadFactoryNames(type, classLoader));
		List<T> instances = createSpringFactoriesInstances(type, parameterTypes,
				classLoader, args, names);
		AnnotationAwareOrderComparator.sort(instances);
		return instances;
	}
```

####    SpringFactoriesLoader
加载springFactories类   
Factories 资源地址 META-INF/spring.factories
```
	/**
	 * The location to look for factories.
	 * <p>Can be present in multiple JAR files.
	 */
	public static final String FACTORIES_RESOURCE_LOCATION = "META-INF/spring.factories";

    //本地缓存
    private static final Map<ClassLoader, MultiValueMap<String, String>> cache = new ConcurrentReferenceHashMap<>();


```
加载所有spring.factories中接口和实现类集合  
封装成Properties 放入缓存   
key 接口    
value 接口实现类 listStr    

eg. 
```
# Application Context Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,\
org.springframework.boot.context.ContextIdApplicationContextInitializer,\
org.springframework.boot.context.config.DelegatingApplicationContextInitializer,\
org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer

```
Properties properties = PropertiesLoaderUtils.loadProperties(resource);
加载完成后填充Properties
```
"org.springframework.context.ApplicationContextInitializer" -> "org.springframework.boot.context.ConfigurationWarningsApplicationContextInitializer,org.springframework.boot.context.ContextIdApplicationContextInitializer,org.springframework.boot.context.config.DelegatingApplicationContextInitializer,org.springframework.boot.web.context.ServerPortInfoApplicationContextInitializer"
```

```
/**
	 * Load the fully qualified class names of factory implementations of the
	 * given type from {@value #FACTORIES_RESOURCE_LOCATION}, using the given
	 * class loader.
	 * @param factoryClass the interface or abstract class representing the factory
	 * @param classLoader the ClassLoader to use for loading resources; can be
	 * {@code null} to use the default
	 * @see #loadFactories
	 * @throws IllegalArgumentException if an error occurs while loading factory names
	 */
	public static List<String> loadFactoryNames(Class<?> factoryClass, @Nullable ClassLoader classLoader) {
		String factoryClassName = factoryClass.getName();
		return loadSpringFactories(classLoader).getOrDefault(factoryClassName, Collections.emptyList());
	}
```
加载SpringFactories中某个接口的实现类集合
```
private static Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader) {
		MultiValueMap<String, String> result = cache.get(classLoader);
		if (result != null) {
			return result;
		}

		try {
		    //从类加载器中或者系统资源中获取spring.factories
			Enumeration<URL> urls = (classLoader != null ?
					classLoader.getResources(FACTORIES_RESOURCE_LOCATION) :
					ClassLoader.getSystemResources(FACTORIES_RESOURCE_LOCATION));
			result = new LinkedMultiValueMap<>();
			while (urls.hasMoreElements()) {
				URL url = urls.nextElement();
			    //jar:file:/Users/haochencheng/Workspace/maven-repo/org/springframework/boot/spring-boot/2.0.6.RELEASE/spring-boot-2.0.6.RELEASE.jar!/META-INF/spring.factories
				UrlResource resource = new UrlResource(url);
				Properties properties = PropertiesLoaderUtils.loadProperties(resource);
				for (Map.Entry<?, ?> entry : properties.entrySet()) {
					List<String> factoryClassNames = Arrays.asList(
							StringUtils.commaDelimitedListToStringArray((String) entry.getValue()));
					result.addAll((String) entry.getKey(), factoryClassNames);
				}
			}
			cache.put(classLoader, result);
			return result;
		}
		catch (IOException ex) {
			throw new IllegalArgumentException("Unable to load factories from location [" +
					FACTORIES_RESOURCE_LOCATION + "]", ex);
		}
	}
```
根据spring.factories资源文件地址加载资源

```
/**
	 * Load properties from the given resource (in ISO-8859-1 encoding).
	 * @param resource the resource to load from
	 * @return the populated Properties instance
	 * @throws IOException if loading failed
	 * @see #fillProperties(java.util.Properties, Resource)
	 */
	public static Properties loadProperties(Resource resource) throws IOException {
		Properties props = new Properties();
		fillProperties(props, resource);
		return props;
	}

```
将资源属性填充Properties
```

private static final String XML_FILE_EXTENSION = ".xml";

/**
	 * Fill the given properties from the given resource (in ISO-8859-1 encoding).
	 * @param props the Properties instance to fill
	 * @param resource the resource to load from
	 * @throws IOException if loading failed
	 */
	public static void fillProperties(Properties props, Resource resource) throws IOException {
		InputStream is = resource.getInputStream();
		try {
			String filename = resource.getFilename();
			//如果是xml 加载xml
			if (filename != null && filename.endsWith(XML_FILE_EXTENSION)) {
				props.loadFromXML(is);
			}
			else {
			    //加载输入流
				props.load(is);
			}
		}
		finally {
			is.close();
		}
	}
```

#   2.运行应用
运行spring 应用 ， 创建并且 刷新 一个新的 应用上下文

```
/**
	 * Run the Spring application, creating and refreshing a new
	 * {@link ApplicationContext}.
	 * @param args the application arguments (usually passed from a Java main method)
	 * @return a running {@link ApplicationContext}
	 */
	public ConfigurableApplicationContext run(String... args) {
	    //任务运行时间监听器
		StopWatch stopWatch = new StopWatch();
		stopWatch.start();
		//可配置应用上下文
		ConfigurableApplicationContext context = null;
		//异常记录集合
		Collection<SpringBootExceptionReporter> exceptionReporters = new ArrayList<>();
		//配置headless属性
		configureHeadlessProperty();
		//获取 RunListeners
		SpringApplicationRunListeners listeners = getRunListeners(args);
		//启动监听器
		listeners.starting();
		try {
		    //启动参数
			ApplicationArguments applicationArguments = new DefaultApplicationArguments(
					args);
			//准备环境 设置active profile，发送environmentPrepared事件，加载resource下的 配置文件 
			ConfigurableEnvironment environment = prepareEnvironment(listeners,
					applicationArguments);
			configureIgnoreBeanInfo(environment);
			//获取banner信息
			Banner printedBanner = printBanner(environment);
			//创建应用上下文
			context = createApplicationContext();
			//异常报告
			exceptionReporters = getSpringFactoriesInstances(
				SpringBootExceptionReporter.class,
					new Class[] { ConfigurableApplicationContext.class }, context);
			//准备上下文	将environment设置到上下文，SpringApplicationRunListeners发送contextPrepared事件，调用监听器构建上下文	。
			prepareContext(context, environment, listeners, applicationArguments,
					printedBanner);
			//刷新上下文		
			refreshContext(context);
			//刷新上下文后操作
			afterRefresh(context, applicationArguments);
			//停止计时
			stopWatch.stop();
			if (this.logStartupInfo) {
				new StartupInfoLogger(this.mainApplicationClass)
						.logStarted(getApplicationLog(), stopWatch);
			}
			//启动监听器
			listeners.started(context);
			//启动Runners
			callRunners(context, applicationArguments);
		}
		catch (Throwable ex) {
		    //运行失败处理
			handleRunFailure(context, ex, exceptionReporters, listeners);
			throw new IllegalStateException(ex);
		}
        //运行监听器
		try {
			listeners.running(context);
		}
		catch (Throwable ex) {
			handleRunFailure(context, ex, exceptionReporters, null);
			throw new IllegalStateException(ex);
		}
		return context;
	}
```
#####    2.1 启动监听器
```
public void starting() {
		for (SpringApplicationRunListener listener : this.listeners) {
			listener.starting();
		}
	}
```
#####   2.2 发布程序开始事件
```
SpringApplicationRunListener -> EventPublishingRunListener  

    public void starting() {
		this.initialMulticaster.multicastEvent(
				new ApplicationStartingEvent(this.application, this.args));
	}
```
#####   2.3 准备可配置环境
```
private ConfigurableEnvironment prepareEnvironment(
			SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments) {
		// Create and configure the environment
		ConfigurableEnvironment environment = getOrCreateEnvironment();
		configureEnvironment(environment, applicationArguments.getSourceArgs());
		//发送environmentPrepared环境准备事件
		listeners.environmentPrepared(environment);
		bindToSpringApplication(environment);
		if (!this.isCustomEnvironment) {
			environment = new EnvironmentConverter(getClassLoader())
					.convertEnvironmentIfNecessary(environment, deduceEnvironmentClass());
		}
		//加载resource下的 配置文件 设置到environment环境中
		ConfigurationPropertySources.attach(environment);
		return environment;
	}
```
初始化可配置环境

```
	ConfigurableEnvironment environment = getOrCreateEnvironment();
	

    private ConfigurableEnvironment getOrCreateEnvironment() {
		if (this.environment != null) {
			return this.environment;
		}
		switch (this.webApplicationType) {
		case SERVLET:
			return new StandardServletEnvironment();
		case REACTIVE:
			return new StandardReactiveWebEnvironment();
		default:
			return new StandardEnvironment();
		}
	}
```
####    2.4 创建应用上下文ApplicationContext

```
private void prepareContext(ConfigurableApplicationContext context,
			ConfigurableEnvironment environment, SpringApplicationRunListeners listeners,
			ApplicationArguments applicationArguments, Banner printedBanner) {
		//将环境设置到上下文 	
		context.setEnvironment(environment);
		//设置beanNameGenerator、resourceLoader、ClassLoader
		postProcessApplicationContext(context);
		// 程序初始化  
		applyInitializers(context);
		//发送contextPrepared事件
		listeners.contextPrepared(context);
		if (this.logStartupInfo) {
			logStartupInfo(context.getParent() == null);
			logStartupProfileInfo(context);
		}

		// Add boot specific singleton beans
		context.getBeanFactory().registerSingleton("springApplicationArguments",
				applicationArguments);
		if (printedBanner != null) {
			context.getBeanFactory().registerSingleton("springBootBanner", printedBanner);
		}

		// Load the sources
		Set<Object> sources = getAllSources();
		Assert.notEmpty(sources, "Sources must not be empty");
		load(context, sources.toArray(new Object[0]));
		//加载上下文 将listeners设置到context，然后 发送contextLoaded事件
		listeners.contextLoaded(context);
	}
```

####   refresh 刷新上下文 spirng启动。

```
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();

			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();

			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);

			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);

				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);

				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);

				// Initialize message source for this context.
				initMessageSource();

				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();

				// Initialize other special beans in specific context subclasses.
				onRefresh();

				// Check for listener beans and register them.
				registerListeners();

				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);

				// Last step: publish corresponding event.
				finishRefresh();
			}

			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}

				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();

				// Reset 'active' flag.
				cancelRefresh(ex);

				// Propagate exception to caller.
				throw ex;
			}

			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```
