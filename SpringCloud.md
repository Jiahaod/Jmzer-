# SpringCloud

----

## 微服务

微服务架构风格是一种将单个应用程序开发位一组小型服务的方法，每个小范围运行在自己的进程中，并且以轻量级机制(通常是HTTP REST API)通信。这些服务是围绕业务能力建立的，并且可以由完全自动化的部署机构独立部署。这些服务的几种管理只有最低限度，可以用不同的编程语言编写并使用不同数据存储技术。

- 小型化-只做好一件事
- 自治理-高内聚/低耦合
- 扁平化-必要随意混搭
- 轻量级设计-Http-Restful API

----

## 架构

- Spring Cloud是Spring位微服务提供的一站式解决方案
- Spring Cloud基于Spring Boot实现云应用开发
- Spring Cloud是一组独立组件(中间件)的集合

---

## 服务发现 Eureka

![image-20201025211830795](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201025211830795.png)

Eureka组成![image-20201025212342630](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201025212342630.png)

springboot中 eureka的配置

```yml
server:
  port: 8761    #默认端口号

eureka:
  instance:
    appname: provider-service
    hostname: localhost
  client:
    service-url:
      defaultZone:
        http://localhost:8761/eureka/
```

在application中添加注解     @EnableEurekaServer

### Eureka客户端开发要点

- maven依赖spring-cloud-starter-netflix-eureka-client
- application.yml配置eureka.client.service-url.defaultZone
- 入口类增加@EnableEurekaClient

### Eureka名词概念

- Register - 服务注册，向Eureka进行注册登记
- Renew - 服务续约，30秒/次心跳包健康检查，90秒未收到剔除服务
- Fetch Registries - 获取服务注册列表，获取其他微服务地址
- Cancel - 服务下线，某个微服务通知注册中心暂停服务
- Eviction - 服务剔除，90秒未续约，从服务注册表进行剔除

![image-20201026104417110](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201026104417110.png)

关闭自我保护

### Eureka自我保护机制

- Eureka在运行期去统计心跳失败率在15分钟之内是否低于85%
- 如果低于85%，会将这些实例保护起来，让这些实例不会被剔除
- 关闭自我保护：eureka.server.enable-self-preservation: false
- 除非网络特别不稳定，建议关闭

### Eureka高可用配置

配置多个yml，在配置

```
eureka:
  client:
    service-url:
      defaultZone:
      	http://server1:8761/eureka/, http://server2:8762/eureka/
      	#用逗号分隔
```

### Actuator监控组件

- Actuator自动为微服务创建一系列的用于监控的端点
- Actuator在springboot自带，springcloud进行扩展
- pom.xml 依赖spring-boot-starter-actuator

### RestTemplate

- RestTemplate是SpringCloud访问Restful API的请求对象
- RestTemplate与HttpClient、OKHttp职能类似

@LoadBalanced注解

- @LoadBalanced是Ribbon提供的客户端负载均衡注解
- 通常RestTemplate与@LoadBanalced联合使用

![image-20201101152436884](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101152436884.png)

![image-20201101152638309](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101152638309.png)

## OpenFeign

-----

### Feign与OpenFeign

- Feign是一个开源声明式WebService客户端，用于简化服务通信

- Feign采用"接口+注解”方式开发，屏蔽了网络通信的细节
- OpenFeign是SpringCloud对Feign的增强，支持Spring MVC注解

![image-20201101161327284](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101161327284.png)

![image-20201101161411994](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101161411994.png)

和ribbon中一样

### OpenFeign开启通信日志

- 基于SpringBoot的logback输出，默认debug级别
- 设置项：feign.client.config.微服务id.loggerLevel
- 微服务id：default代表全局默认配置

![image-20201101161912996](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101161912996.png)

```
logging:
  level:
    ROOT: debug

feign:
  client:
    config:
      default:
        loggerLevel: FULL
```

![image-20201101163936394](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201101163936394.png)

### OpenFegin传递对象

- POST方式传递对象使用@RequestBody注解描述参数
- GET方式将对象转化为Map后利用@RequestParam注解

![image-20201104133749510](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201104133749510.png)

-----

## Hystrix熔断器

![image-20201104151705331](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201104151705331.png)

![image-20201104151841154](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201104151841154.png)

hystri放在加入者一方



```
@HystrixCommand(fallbackMethod = "sendNewBookError")
```

### OpenFeign与Hystrix整合

- openFeign内置Hystrix，feign.hystrix.enable开启
- 在@FeignClient增加fallback属性说明Fallback类
- fallback类要实现相同接口，重写服务降级业务逻辑

![image-20201104192702703](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201104192702703.png)

### Hystrix Dashboard仪表盘

- Hystrix Client依赖hystrix-metrics-event-stream
- Hystrix Client注册HystrixMetricsStreamServlet
- 监控微服务依赖spring-cloud-starter-netflix-hystrix-dashboard
- 监控微服务利用@EnableHystrixDashboard开启仪表盘

```
        <dependency>
            <groupId>com.netflix.hystrix</groupId>
            <artifactId>hystrix-metrics-event-stream</artifactId>
            <version>1.5.18</version>
        </dependency>
```



```
@Bean
public ServletRegistrationBean hystrixServlet(){
    HystrixMetricsStreamServlet servlet = new HystrixMetricsStreamServlet();
    ServletRegistrationBean registrationBean = new ServletRegistrationBean(servlet);
    registrationBean.addUrlMappings("/hystrix.stream");
    registrationBean.setName("HystrixMetricsStreamServlet");
    registrationBean.setLoadOnStartup(1);
    return registrationBean;

}
```



```
http://localhost:9100/hystrix/
```

----

## Zuul

通过网关隐藏后端服务，用于代理

使用网关的优点

- 同一访问出入口，微服务对前台透明
- 安全，过滤，流控等API管理功能
- 易于监控，方便管理

Netflix Zuul

- Zuul 是Netflix开源的一个API网关，核心实现是Servlet
- Spring Cloud内置Zuul 1.x
- Zuul 1.x 核心实现是Servlet，采用同步方式通信
- Zuul 2.x 基于Netty Server，提供异步通信

功能

![image-20201105150646541](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201105150646541.png)

![image-20201105170247510](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201105170247510.png)

### Zuul负载均衡与服务降级

- Spring Cloud Zuul内置Ribbon，于标准配置相同

```java
@Bean	//全局配置
public IRule ribbonRule()
{
	return new RandomRule();
}
```



![image-20201105190300906](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201105190300906.png)

### 微服务网关流量控制

- 微服务网关是应用入口，必须对入口流量进行控制
- RateLimit是Spring Cloud Zuul的限流组件
- RateLimit采用"令牌桶"算法实现限流



使用ratelimit

```
        <dependency>
            <groupId>com.marcosbarbero.cloud</groupId>
            <artifactId>spring-cloud-zuul-ratelimit</artifactId>
            <version>2.2.7.RELEASE</version>
        </dependency>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>5.1.47</version>
        </dependency>
```



![image-20201106180320876](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201106180320876.png)

Zuul自定义过滤器

```
shouldFilter() -是否启用该过滤器
filterOrder() -设置过滤器执行次序
filterType() -过滤器类型: pre|routing|post
run() -过滤逻辑
```

![image-20201106221404945](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201106221404945.png)

## 配置中心

![image-20201108165344520](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201108165344520.png)

![image-20201108172302245](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201108172302245.png)

spring-retry重试机制

![image-20201108183436523](C:\Users\天一喔蜂蜜柚子\AppData\Roaming\Typora\typora-user-images\image-20201108183436523.png)