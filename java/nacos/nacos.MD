## nacos-springboot-project
- acuator 监控与管理
- autoconfigure 自动装配
- starter 依赖管理

###    acuator
1. NacosConfigEndpointAutoConfiguration
2. NacosConfigHealthIndicatorAutoConfiguration

######   NacosConfigEndpointAutoConfiguration

```
@Bean
	@ConditionalOnMissingBean
	@ConditionalOnEnabledEndpoint
	public NacosConfigEndpoint nacosEndpoint() {
		return new NacosConfigEndpoint();
	}

```
- 	@ConditionalOnMissingBean       
如果没有NacosConfigEndpoint Bean，则加载。


-   @ConditionalOnEnabledEndpoint
注解，如果开启EnabledEndpoint则加载。

```
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.METHOD)
@Documented
@Conditional(OnEnabledEndpointCondition.class)
public @interface ConditionalOnEnabledEndpoint {

}

```

```
graph LR
ConditionalOnEnabledEndpoint-->OnEnabledEndpointCondition
OnEnabledEndpointCondition-->SpringBootCondition
SpringBootCondition-->
Condition
```

- Condition

```
boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);

```

#####   NacosConfigEndpoint
- 通过ApplicationListener<NacosConfigMetadataEvent>接收NacosConfigMetadataEvent事件构建nacos配置信息

```
graph LR
NacosConfigEndpoint--> ApplicationListener
```
#####    NacosConfigHealthIndicatorAutoConfiguration

```
@Configuration
@ConditionalOnClass({ HealthIndicator.class })
@AutoConfigureBefore({ EndpointAutoConfiguration.class })
@AutoConfigureAfter(NacosConfigAutoConfiguration.class)
@ConditionalOnEnabledHealthIndicator(NacosConfigConstants.ENDPOINT_PREFIX)
public class NacosConfigHealthIndicatorAutoConfiguration {

	@Bean
	@ConditionalOnMissingBean
	public NacosConfigHealthIndicator nacosConfigHealthIndicator() {
		return new NacosConfigHealthIndicator();
	}
}
```

```
graph LR
NacosConfigHealthIndicator-->AbstractHealthIndicator
AbstractHealthIndicator-->HealthIndicator
```

- 其中HealthIndicator接口定义了

```
@FunctionalInterface
public interface HealthIndicator {

	/**
	 * Return an indication of health.
	 * @return the health for
	 */
	Health health();

}
```

- Health类
```
@JsonInclude(Include.NON_EMPTY)
public final class Health {

    //服务器状态
	private final Status status;

    //服务器健康参数
	private final Map<String, Object> details;

}
```

- AbstractHealthIndicator抽象类中定义了抽象方法doHealthCheck让子类实现

```
	/**
	 * Actual health check logic.
	 * @param builder the {@link Builder} to report health status and details
	 * @throws Exception any {@link Exception} that should create a {@link Status#DOWN}
	 * system status.
	 */
	protected abstract void doHealthCheck(Health.Builder builder) throws Exception;

```

###    nacos-config-spring-boot-starter
- Spring Boot 旨在简化应用程序创建入门的过程。 Spring Boot 模块是引导库，其中包含启动特定功能所需的所有相关传递依赖关系的集合。 每个启动器都有一个特定文件，其中包含所有提供的依赖关系的列表—— spring.provides
- 通过resource中META-INF中spring.providers定义依赖关系，加载
nacos-config-spring-boot-autoconfigure自动配置模块
```
provides: nacos-config-spring-boot-autoconfigure
```

```
graph LR
starter --> autoconfigure
```

- resource中META-INF中spring.factories中定义自动装配类NacosConfigAutoConfiguration

```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.alibaba.boot.nacos.config.autoconfigure.NacosConfigAutoConfiguration
```
- NacosConfigAutoConfiguration

```
@ConditionalOnProperty(name = NacosConfigConstants.ENABLED, matchIfMissing = true)
@ConditionalOnMissingBean(name = CONFIG_GLOBAL_NACOS_PROPERTIES_BEAN_NAME)
@EnableNacosConfig
@EnableConfigurationProperties(value = NacosConfigProperties.class)
@ConditionalOnClass(name = "org.springframework.boot.context.properties.bind.Binder")
public class NacosConfigAutoConfiguration {

}
```
1. 加载条件         
@ConditionalOnProperty(name = NacosConfigCon  stants.ENABLED, matchIfMissing = true)
@ConditionalOnMissingBean(name = CONFIG_GLOBAL_NACOS_PROPERTIES_BEAN_NAME)
@ConditionalOnClass(name = "org.springframework.boot.context.properties.bind.Binder")

2. 依赖关系
```
graph LR
NacosConfigAutoConfiguration-->EnableNacosConfig
EnableNacosConfig-->NacosConfigBeanDefinitionRegistrar
NacosConfigBeanDefinitionRegistrar-->ImportBeanDefinitionRegistrar
NacosConfigBeanDefinitionRegistrar-->EnvironmentAware
NacosConfigAutoConfiguration-->EnableConfigurationProperties

```
- @EnableConfigurationProperties
关于EnableConfigurationProperties，在SpringBoot的注释中是这样说明的：为带有@ConfigurationProperties注解的Bean提供有效的支持.   
ConfigurationPropertiesBindingPostProcessor会对标注@ConfigurationProperties注解的Bean进行属性值的配置

```
graph LR
ConfigurationPropertiesAutoConfiguration-->EnableConfigurationProperties
EnableConfigurationProperties-->EnableConfigurationPropertiesImportSelector

```

- ConfigurationPropertiesAutoConfiguration类

```
@Configuration
@EnableConfigurationProperties
public class ConfigurationPropertiesAutoConfiguration {

}

```
- EnableConfigurationProperties
```
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(EnableConfigurationPropertiesImportSelector.class)
public @interface EnableConfigurationProperties {

	/**
	 * Convenient way to quickly register {@link ConfigurationProperties} annotated beans
	 * with Spring. Standard Spring Beans will also be scanned regardless of this value.
	 * @return {@link ConfigurationProperties} annotated beans to register
	 */
	Class<?>[] value() default {};

}
```
####    autoconfig流程
1. start 中通过spring.provides指定nacos-config-spring-boot-autoconfigure依赖
2. nacos-config-spring-boot-autoconfigure中通过spring.factories指定自动装配类 
```
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
com.alibaba.boot.nacos.config.autoconfigure.NacosConfigAutoConfiguration
```
3. NacosConfigAutoConfiguration中定义加载条件，自动加载配置bean

```
@EnableConfigurationProperties(value = NacosConfigProperties.class)
```
并且启用加载nacos配置
@EnableNacosConfig

```
@Target({ElementType.TYPE, ElementType.ANNOTATION_TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import(NacosConfigBeanDefinitionRegistrar.class)
public @interface EnableNacosConfig {
```
4. EnableNacosConfig中@Import引入NacosConfigBeanDefinitionRegistrar类
- 实现了ImportBeanDefinitionRegistrar类（注册bean）
- 实现了EnvironmentAware类
```
public class NacosConfigBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar, EnvironmentAware {

    private Environment environment;

    @Override
    public void registerBeanDefinitions(AnnotationMetadata metadata, BeanDefinitionRegistry registry) {
        AnnotationAttributes attributes = fromMap(metadata.getAnnotationAttributes(EnableNacosConfig.class.getName()));
        // Register Global Nacos Properties Bean
        registerGlobalNacosProperties(attributes, registry, environment, CONFIG_GLOBAL_NACOS_PROPERTIES_BEAN_NAME);
        // Register Nacos Common Beans
        registerNacosCommonBeans(registry);
        // Register Nacos Config Beans
        registerNacosConfigBeans(registry, environment);
    }

    @Override
    public void setEnvironment(Environment environment) {
        this.environment = environment;
    }
}
```
5. ImportBeanDefinitionRegistrar类
- 

```
/**
 * Interface to be implemented by types that register additional bean definitions when
 * processing @{@link Configuration} classes. Useful when operating at the bean definition
 * level (as opposed to {@code @Bean} method/instance level) is desired or necessary.
 */
public interface ImportBeanDefinitionRegistrar {

	/**
	 * Register bean definitions as necessary based on the given annotation metadata of
	 * the importing {@code @Configuration} class.
	 * <p>Note that {@link BeanDefinitionRegistryPostProcessor} types may <em>not</em> be
	 * registered here, due to lifecycle constraints related to {@code @Configuration}
	 * class processing.
	 * @param importingClassMetadata annotation metadata of the importing class
	 * @param registry current bean definition registry
	 */
	public void registerBeanDefinitions(
			AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry);

}
```



