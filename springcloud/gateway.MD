##  springcloud-gateway网关
### 网关功能
1. 路由
2. 限流
3. 负载均衡
4. 服务管理


### 初始化顺序



### 请求流程
```
graph LR
请求-->predicate
predicate-->匹配Route
匹配Route-->filter
filter-->请求url
```
####    GatewayProperties
- 网关配置
定义了RouteDefinition和FilterDefinition以及stream的MediaType媒体类型。
- org.springframework.cloud.gateway.config.GatewayProperties ，从配置文件读取 ：
路由配置
默认过滤器配置。当 RouteDefinition => Route 时，会将过滤器配置添加到每个 Route 。
```
@ConfigurationProperties("spring.cloud.gateway")
@Validated
public class GatewayProperties {

	/**
	 * List of Routes
	 */
	@NotNull
	@Valid
	private List<RouteDefinition> routes = new ArrayList<>();

	/**
	 * List of filter definitions that are applied to every route.
	 */
	private List<FilterDefinition> defaultFilters = new ArrayList<>();

	private List<MediaType> streamingMediaTypes = Arrays.asList(MediaType.TEXT_EVENT_STREAM,
			MediaType.APPLICATION_STREAM_JSON);
			
}
```

-routes 属性，路由配置。    
通过 spring.cloud.gateway.routes 配置。以 YAML 配置文件举例子 ：
```
spring:
  cloud:
    gateway:
      routes:
      # =====================================
      - host_example_to_httpbin=${test.uri}, Host=**.example.org

      # =====================================
      - id: host_foo_path_headers_to_httpbin
        uri: ${test.uri}
        predicates:
        # 根据host路由
        - Host=**.foo.org  
        # 根据路径匹配，如/ubet/**  -> 路由到ubet系统uri
        - Path=/headers
        # 根据请求方法路由，如Get方法转发到某个uri
        - Method=GET
        # 根据Header中某个属性路由，两个参数，名称和正则表达式
        - Header=X-Request-Id, \d+
        # 根据请求参数路由
        # equest contained a foo query parameter whose value matched the ba. regexp, so bar and baz would match.
        - Query=foo, ba. 
        - Query=baz
        # 根据Cookied路由，名称、正则匹配值（可选）
        - Cookie=chocolate, ch.p
        # 根据时间路由
        - After=1900-01-20T17:42:47.789-07:00[America/Denver]
        filters:
        - AddResponseHeader=X-Response-Foo, Bar

      # =====================================
      - id: add_request_header_test
        # 路由的目标uri
        uri: ${test.uri}
        predicates:
        - Host=**.addrequestheader.org
        - Path=/headers
        filters:
        - AddRequestHeader=X-Request-Foo, Bar
```

### RouteDefinitionLocator
- 定义了获取FluxRoute定义配置

```
/**
 * @author Spencer Gibb
 */
public interface RouteDefinitionLocator {
	Flux<RouteDefinition> getRouteDefinitions();
}
```

```
graph LR
RouteDefinitionLocator-->PropertiesRouteDefinitionLocator
RouteDefinitionLocator-->CachingRouteDefinitionLocator
RouteDefinitionLocator-->CompositeRouteDefinitionLocator
RouteDefinitionLocator-->DiscoveryClientRouteDefinitionLocator
RouteDefinitionLocator-->RouteDefinitionRepository
RouteDefinitionRepository-->InMemoryRouteDefinitionRepository
```

####    RouteDefinition
定义了Route路由相关配置RouteDefinition ，id、PredicateDefinition、FilterDefinition、uri、order

```
@Validated
public class RouteDefinition {
	@NotEmpty
	private String id = UUID.randomUUID().toString();

	@NotEmpty
	@Valid
	private List<PredicateDefinition> predicates = new ArrayList<>();

	@Valid
	private List<FilterDefinition> filters = new ArrayList<>();

	@NotNull
	private URI uri;

	private int order = 0;

	public RouteDefinition() {}

	public RouteDefinition(String text) {
		int eqIdx = text.indexOf('=');
		if (eqIdx <= 0) {
			throw new ValidationException("Unable to parse RouteDefinition text '" + text + "'" +
					", must be of the form name=value");
		}

		setId(text.substring(0, eqIdx));

		String[] args = tokenizeToStringArray(text.substring(eqIdx+1), ",");

		setUri(URI.create(args[0]));

		for (int i=1; i < args.length; i++) {
			this.predicates.add(new PredicateDefinition(args[i]));
		}
	}
}

```
####    PropertiesRouteDefinitionLocator类
- org.springframework.cloud.gateway.config.PropertiesRouteDefinitionLocator ，从配置文件( 例如，YML / Properties 等 ) 读取路由配置。代码如下 ：

```
public class PropertiesRouteDefinitionLocator implements RouteDefinitionLocator {

	private final GatewayProperties properties;

	public PropertiesRouteDefinitionLocator(GatewayProperties properties) {
		this.properties = properties;
	}

	@Override
	public Flux<RouteDefinition> getRouteDefinitions() {
		return Flux.fromIterable(this.properties.getRoutes());
	}
}
```
- #getRouteDefinitions() 方法，从 GatewayProperties 获取路由配置数组。






