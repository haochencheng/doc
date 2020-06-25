
### spring bean生命周期
1. BeanFactory


    提供了最基本的依赖注入的支持
    
2. ApplicationContext


    在继承前者的基础进行了功能的拓展，例如增加了事件传播，资源访问和国际化的消息访问等功能
    
####    ApplicationContext Bean生命周期

![image](https://upload-images.jianshu.io/upload_images/3131012-0fdb736b21c8cc31.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/700)

### spring BeanFactory
 BeanFactory 接口定义：一个访问bean容器的根接口
       
    The root interface for accessing a Spring bean container.
    

```
/**
	 * Used to dereference a {@link FactoryBean} instance and distinguish it from
	 * beans <i>created</i> by the FactoryBean. For example, if the bean named
	 * {@code myJndiObject} is a FactoryBean, getting {@code &myJndiObject}
	 * will return the factory, not the instance returned by the factory.
	 */
	String FACTORY_BEAN_PREFIX = "&";
	
	//获取bean
	Object getBean(String name) throws BeansException;
	
	//获取指定类型bean
	<T> T getBean(String name, Class<T> requiredType) throws BeansException;
	
	//通过名字，参数获取bean
	Object getBean(String name, Object... args) throws BeansException;
	
	//通过类型获取bean
	<T> T getBean(Class<T> requiredType) throws BeansException;
	
	//是否包含bean
	boolean containsBean(String name);
	
	//是否是单例的
	boolean isSingleton(String name) throws NoSuchBeanDefinitionException;
	
	//是否是原型？
	boolean isPrototype(String name) throws NoSuchBeanDefinitionException;
	
	//是否类型匹配
	boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

    //通过名称获取类型
    Class<?> getType(String name) throws NoSuchBeanDefinitionException;

    //获取bean别名
    String[] getAliases(String name);
	
```
#### HierarchicalBeanFactory
分级BeanFactory

```
/**
	 * Return the parent bean factory, or {@code null} if there is none.
	 */
	BeanFactory getParentBeanFactory();
	
	//Return whether the local bean factory contains a bean of the given name,
	 * ignoring beans defined in ancestor contexts.
	boolean containsLocalBean(String name);
	
```

####    SingletonBeanRegistry
单例BeanFactory

```
//注册单例bean通过beanName名字
void registerSingleton(String beanName, Object singletonObject);

//通过Bean名称获取Bean对象
Object getSingleton(String beanName);

//是否包含单例对象
boolean containsSingleton(String beanName);

//获取单例工厂中的所有Bean名称集合Names
String[] getSingletonNames();

//获取单例Bean数量
int getSingletonCount();

//获取单例互斥？？
Object getSingletonMutex();

```

#### ConfigurableBeanFactory
可配置BeanFactory继承了分级BeanFactory和单例BeanFactory

```
public interface ConfigurableBeanFactory extends HierarchicalBeanFactory, SingletonBeanRegistry 
```

定义了bean作用域scope

- singleton
    单例模式，当一个bean的作用域设置为singleton, 那么Spring IOC容器中只会存在一个共享的bean实例，并且所有对bean的请求，只要id与该bean定义相匹配，则只会返回bean的同一实例。

- prototype
    prototype作用域部署的bean，每一次请求（将其注入到另一个bean中，或者以程序的方式调用容器的getBean()方法）都会产生一个新的bean实例，相当与一个new的操作，对于prototype作用域的bean，有一点非常重要，那就是Spring不能对一个prototype bean的整个生命周期负责，容器在初始化、配置、装饰或者是装配完一个prototype实例后，将它交给客户端，随后就对该prototype实例不闻不问了。

```

String SCOPE_SINGLETON = "singleton";

String SCOPE_PROTOTYPE = "prototype";

//设置父工厂
void setParentBeanFactory(BeanFactory parentBeanFactory) throws IllegalStateException;

//设置临时类加载器
void setTempClassLoader(ClassLoader tempClassLoader);

//获取临时类加载器
ClassLoader getTempClassLoader();

//设置bean元数据缓存 
void setCacheBeanMetadata(boolean cacheBeanMetadata);

//判断是否是缓存元数据
boolean isCacheBeanMetadata();

//设置Bean表达式解析器
void setBeanExpressionResolver(BeanExpressionResolver resolver);

//获取表达式解析器
BeanExpressionResolver getBeanExpressionResolver();




```

