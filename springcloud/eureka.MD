##  spring-cloud-eureka-server 注册中心
1.  引入pom

```
	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
	</dependency>
```

2.修改配置信息

```

spring.application.name = spring-cloud-eureka-server

server.port = 8000


```

3.启动服务

```
package pers.demo.springcloud.springcloudeurekaserver;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

@SpringBootApplication
@EnableEurekaServer
public class SpringCloudEurekaServerApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaServerApplication.class, args);
	}
}

```

5.报错

```
com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused (Connection refused)
Caused by: java.net.ConnectException: Connection refused (Connection refused)

com.netflix.discovery.shared.transport.TransportException: Cannot execute request on any known server
```
问题原因，eureka-server 即是服务端，也是
客户端，默认情况需要配置服务器地址

###  spring-cloud-eureka-client 客户端
1.引入配置

```
	<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
	</dependency>
```
2.修改配置信息

```

spring.application.name = spring-cloud-eureka-client

server.port = 9000

management.endpoints.web.exposure.include=*
```
3.启动应用

```
package pers.demo.springcloud.springcloudeurekaclient;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;

@SpringBootApplication
@EnableEurekaClient
public class SpringCloudEurekaClientApplication {

	public static void main(String[] args) {
		SpringApplication.run(SpringCloudEurekaClientApplication.class, args);
	}
}

```

4.检验eureka客户端

```

com.sun.jersey.api.client.ClientHandlerException: java.net.ConnectException: Connection refused (Connection refused)

```

和服务端同样的错误。
### 需要再次调整eureka客户端
禁止服务端作为客户端

```

spring.application.name = spring-cloud-eureka-client

server.port = 9000
server.servlet.path=/

management.endpoints.web.exposure.include=*

#spring cloud eureka 注册到注册中心
eureka.instance.hostname=localhost
eureka.instance.statusPageUrlPath=${server.servlet.path}/actuator/info
eureka.instance.healthCheckUrlPath=${server.servlet.path}/actuator/health
eureka.instance.homePageUrlPath=${server.servlet.path}/
eureka.client.serviceUrl.defaultZone = http://${eureka.instance.hostname}:8000/eureka
eureka.client.healthcheck.enabled=true

```

##  Eureka client 高可用
配置多个Eureka注册中心
```
eureka.client.serviceUrl.defaultZone=\
http://localhost:8000/eureka/,\
http://localhost:8001/eureka/
```

源码分析：
注册中心地址为一个List集合，当一个注册中心挂掉之后，客户端会重试。超过重试阀值后会向另一个注册中心注册

```
/**
 * @author Dave Syer
 */
@Data
@ConfigurationProperties(EurekaClientConfigBean.PREFIX)
public class EurekaClientConfigBean implements EurekaClientConfig, EurekaConstants {
    @Override
	public List<String> getEurekaServerServiceUrls(String myZone) {
		String serviceUrls = this.serviceUrl.get(myZone);
		if (serviceUrls == null || serviceUrls.isEmpty()) {
			serviceUrls = this.serviceUrl.get(DEFAULT_ZONE);
		}
		if (!StringUtils.isEmpty(serviceUrls)) {
			final String[] serviceUrlsSplit = StringUtils.commaDelimitedListToStringArray(serviceUrls);
			List<String> eurekaServiceUrls = new ArrayList<>(serviceUrlsSplit.length);
			for (String eurekaServiceUrl : serviceUrlsSplit) {
				if (!endsWithSlash(eurekaServiceUrl)) {
					eurekaServiceUrl += "/";
				}
				eurekaServiceUrls.add(eurekaServiceUrl);
			}
			return eurekaServiceUrls;
		}

		return new ArrayList<>();
	}
}
```







