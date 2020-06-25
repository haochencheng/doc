##  Externalized Configuration
外部配置    
Spring Boot allows you to externalize your configuration so you can work with the same application code in different environments. You can use properties files, YAML files, environment variables and command-line arguments to externalize configuration. Property values can be injected directly into your beans using the @Value annotation, accessed via Spring’s Environment abstraction or bound to structured objects via @ConfigurationProperties.

Spring Boot uses a very particular PropertySource order that is designed to allow sensible overriding of values. Properties are considered in the following order:

Spring Boot允许您外部化配置，以便在不同环境中使用相同的应用程序代码。 您可以使用属性文件，YAML文件，环境变量和命令行参数来外部化配置。 可以使用@Value注释将属性值直接注入到bean中，通过Spring的Environment抽象访问或通过@ConfigurationProperties绑定到结构化对象。 Spring Boot使用一个非常特殊的PropertySource命令，旨在允许合理地覆盖值。 按以下顺序考虑属性：

1.  Devtools global settings properties on your home directory (~/.spring-boot-devtools.properties when devtools is active).

2.  @TestPropertySource annotations on your tests.
3.  @SpringBootTest#properties annotation attribute on your tests.
4.  Command line arguments.
5.  Properties from SPRING_APPLICATION_JSON (inline JSON embedded in an environment variable or system property)
6.  ServletConfig init parameters.
7.  ServletContext init parameters.
8.  JNDI attributes from java:comp/env.
9.  Java System properties (System.getProperties()).
10. OS environment variables.
11. A RandomValuePropertySource that only has properties in random.*.
12. Profile-specific application properties outside of your packaged jar (application-{profile}.properties and YAML variants)
13. Profile-specific application properties packaged inside your jar (application-{profile}.properties and YAML variants)
14. Application properties outside of your packaged jar (application.properties and YAML variants).
15. Application properties packaged inside your jar (application.properties and YAML variants).
16. @PropertySource annotations on your @Configuration classes.
17. Default properties (specified using SpringApplication.setDefaultProperties).


##   The ‘Spring Web MVC framework’
The Spring Web MVC framework (often referred to as simply ‘Spring MVC’) is a rich ‘model view controller’ web framework. Spring MVC lets you create special @Controller or @RestController beans to handle incoming HTTP requests. Methods in your controller are mapped to HTTP using @RequestMapping annotations.

Here is a typical example @RestController to serve JSON data:

```
@RestController
@RequestMapping(value="/users")
public class MyRestController {

    @RequestMapping(value="/{user}", method=RequestMethod.GET)
    public User getUser(@PathVariable Long user) {
        // ...
    }

    @RequestMapping(value="/{user}/customers", method=RequestMethod.GET)
    List<Customer> getUserCustomers(@PathVariable Long user) {
        // ...
    }

}
```

### Spring MVC auto-configuration
springMvc 自动配置  
The auto-configuration adds the following features on top of Spring’s defaults:

- Inclusion of ContentNegotiatingViewResolver and BeanNameViewResolver beans.
- Support for serving static resources, including support for WebJars (see below).
- Automatic registration of Converter, GenericConverter, Formatter beans.
- Support for HttpMessageConverters (see below).
- Automatic registration of MessageCodesResolver (see below).
- Static index.html support.
Custom Favicon support (see below).
- Automatic use of a ConfigurableWebBindingInitializer bean (see below).


##  HttpMessageConverters 
http消息转换器
Spring MVC uses the HttpMessageConverter interface to convert HTTP requests and responses.  
Sensible defaults are included out of the box, for example Objects can be automatically converted to JSON (using the Jackson library) or XML (using the Jackson XML extension if available, else using JAXB). Strings are encoded using UTF-8 by default.     
开箱即用的灵敏默认值包括，例如，对象可以自动转换为JSON（使用Jackson库）或XML（如果可用则使用Jackson XML扩展，否则使用JAXB）。字符串默认使用UTF-8编码。

If you need to add or customize converters you can use Spring Boot’s HttpMessageConverters class:

```
@Configuration
public class MyConfiguration {

    @Bean
    public HttpMessageConverters customConverters() {
        HttpMessageConverter<?> additional = ...
        HttpMessageConverter<?> another = ...
        return new HttpMessageConverters(additional, another);
    }

}
```

##   Custom JSON Serializers and Deserializers
自定义JSON 序列化和反序列化
If you’re using Jackson to serialize and deserialize JSON data, you might want to write your own JsonSerializer and JsonDeserializer classes. Custom serializers are usually registered with Jackson via a Module, but Spring Boot provides an alternative @JsonComponent annotation which makes it easier to directly register Spring Beans.

You can use @JsonComponent directly on JsonSerializer or JsonDeserializer implementations. You can also use it on classes that contains serializers/deserializers as inner-classes. For example:


```

import java.io.*;
import com.fasterxml.jackson.core.*;
import com.fasterxml.jackson.databind.*;
import org.springframework.boot.jackson.*;

@JsonComponent
public class Example {

    public static class Serializer extends JsonSerializer<SomeObject> {
        // ...
    }

    public static class Deserializer extends JsonDeserializer<SomeObject> {
        // ...
    }

}
```

##  Static Content
静态内容    
By default Spring Boot will serve static content from a directory called /static (or /public or /resources or /META-INF/resources) in the classpath or from the root of the ServletContext. It uses the ResourceHttpRequestHandler from Spring MVC so you can modify that behavior by adding your own WebMvcConfigurerAdapter and overriding the addResourceHandlers method.


```
spring.mvc.static-path-pattern=/resources/**
```

##  Custom Favicon
自定义favicon   
Spring Boot looks for a favicon.ico in the configured static content locations and the root of the classpath (in that order). If such file is present, it is automatically used as the favicon of the application.

##  Template engines
模版引擎    
As well as REST web services, you can also use Spring MVC to serve dynamic HTML content. Spring MVC supports a variety of templating technologies including Thymeleaf, FreeMarker and JSPs. Many other templating engines also ship their own Spring MVC integrations.

- FreeMarker
- Groovy
- Thymeleaf
- Mustache

When you’re using one of these templating engines with the default configuration, your templates will be picked up automatically from src/main/resources/templates.

##      Error Handling
错误处理        


##  CORS support
跨与支持        
Cross-origin resource sharing (CORS) is a W3C specification implemented by most browsers that allows you to specify in a flexible way what kind of cross domain requests are authorized, instead of using some less secure and less powerful approaches like IFRAME or JSONP.   

As of version 4.2, Spring MVC supports CORS out of the box. Using controller method CORS configuration with @CrossOrigin annotations in your Spring Boot application does not require any specific configuration. Global CORS configuration can be defined by registering a WebMvcConfigurer bean with a customized addCorsMappings(CorsRegistry) method:


```
@Configuration
public class MyConfiguration {

    @Bean
    public WebMvcConfigurer corsConfigurer() {
        return new WebMvcConfigurerAdapter() {
            @Override
            public void addCorsMappings(CorsRegistry registry) {
                registry.addMapping("/api/**");
            }
        };
    }
}
```

##  JAX-RS and Jersey
restful架构风格
JAX-RS: Java API for RESTful Web Services是一个Java编程语言的应用程序接口,支持按照 表象化状态转变 (REST)架构风格创建Web服务[1]. JAX-RS使用了Java SE 5引入的Java 标注来简化Web服务客户端和服务端的开发和部署。

JAX-RS提供了一些标注将一个资源类，一个POJOJava类，封装为Web资源。标注包括：

@Path，标注资源类或方法的相对路径
@GET，@PUT，@POST，@DELETE，标注方法是用的HTTP请求的类型  
@Produces，标注返回的MIME媒体类型   
@Consumes，标注可接受请求的MIME媒体类型 
@PathParam，@QueryParam，@HeaderParam，@CookieParam，@MatrixParam，@FormParam,分别标注方法的参数来自于HTTP请求的不同位置，例如@PathParam来自于URL的路径，@QueryParam来自于URL的查询参数，@HeaderParam来自于HTTP请求的头信息，@CookieParam来自于HTTP请求的Cookie。

All the registered endpoints should be @Components with HTTP resource annotations (@GET etc.), e.g.


```
@Component
@Path("/hello")
public class Endpoint {

    @GET
    public String message() {
        return "Hello";
    }

}
```

##  Embedded servlet container support
嵌入式容器
Spring Boot includes support for embedded Tomcat, Jetty, and Undertow servers. Most developers will simply use the appropriate ‘Starter’ to obtain a fully configured instance. By default the embedded server will listen for HTTP requests on port 8080.

##  Servlets, Filters, and listeners
When using an embedded servlet container you can register Servlets, Filters and all the listeners from the Servlet spec (e.g. HttpSessionListener) either by using Spring beans or by scanning for Servlet components.

##  自定义Endpoint
Endpoint的Mvc接口主要是通过EndpointWebMvcManagementContextConfiguration实现的，这个类的配置也放在spring.factories中。


```

	@Bean
	@ConditionalOnMissingBean
	public EndpointHandlerMapping endpointHandlerMapping() {
		Set<MvcEndpoint> endpoints = mvcEndpoints().getEndpoints();
		CorsConfiguration corsConfiguration = getCorsConfiguration(this.corsProperties);
		EndpointHandlerMapping mapping = new EndpointHandlerMapping(endpoints,
				corsConfiguration);
		mapping.setPrefix(this.managementServerProperties.getContextPath());
		MvcEndpointSecurityInterceptor securityInterceptor = new MvcEndpointSecurityInterceptor(
				this.managementServerProperties.getSecurity().isEnabled(),
				this.managementServerProperties.getSecurity().getRoles());
		mapping.setSecurityInterceptor(securityInterceptor);
		for (EndpointHandlerMappingCustomizer customizer : this.mappingCustomizers) {
			customizer.customize(mapping);
		}
		return mapping;
	}

```

1.  通过实现import org.springframework.boot.actuate.endpoint.Endpoint; 类自定义端点

```
public static class MemEndPoint extends AbstractEndpoint implements Endpoint {

        private List<MemStatus> status;

        public MemEndPoint(List<MemStatus> status) {
            super("my",false);
            this.status = status;
        }

        @Override
        public Object invoke() {
            if (status == null || status.isEmpty()) {
                return "hello world";
            }
        }
    }
```


2.  springboot 2.0增加了@EndPoint注解可以定义自定义端点




