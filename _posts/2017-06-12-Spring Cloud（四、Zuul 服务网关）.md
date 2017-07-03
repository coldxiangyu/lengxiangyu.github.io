---
layout: post
title:  "Spring Cloud（四、Zuul 服务网关）"
date:   2017-06-12 19:36:12
author: coldxiangyu
categories: WEB
tags: spring-cloud Zuul
mathjax: true
---

* content
{:toc}

前面我们已经介绍了使用Spring Cloud Netflix中的Eureka注册中心进行服务注册与发现，服务间通过Ribbon，Feign进服务消费以及负载均衡，Hystrix断路器，Spring Cloud Config进行统一配置管理。  
在实际应用过程中，我们内部微服务往往是不对外暴露的，需要提供专门的对外服务。这时候难以做到已有服务的复用，也没有一个统一的访问权限控制，因此整个服务需要一个大门，也就是我们这篇文章要讲的服务网关`Zuul`。  




`Zuul`是整个微服务架构重要的组成部分，它负责对外提供统一的REST API，可以对路由策略进行调整，实现负载均衡，还具备权限控制等功能。将一些复杂的非业务逻辑前移，使得服务集群本身具备高可用。  
看完这些，你会不会把Zuul联系到Nginx？实际上，`Zuul`就是Netflix版的`Nginx`，不过实现方式不同。而`Zuul`本身是基于Servlet的过滤器的集合，再加上与Spring boot整合之后，处理请求的速度可能没有nginx这种简单设计的来得快，但确实可以作为一款不错的nginx替代品。  
下面我们来看一下`Zuul`的基本用法：  
新建模块zuul-gateway，pom配置如下：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.3.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
    <java.version>1.8</java.version>
  </properties>

  <dependencies>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-zuul</artifactId>
    </dependency>

    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-eureka</artifactId>
    </dependency>

  </dependencies>

  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-dependencies</artifactId>
        <version>Brixton.RELEASE</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```
应用主类使用`@EnableZuulProxy`注解开启Zuul:
```java
@EnableZuulProxy
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
其中，`@SpringCloudApplication`注解整合了@SpringBootApplication、@EnableDiscoveryClient、@EnableCircuitBreaker，相当于三个注解的集合。

配置`application.properties`中配置Zuul应用的基础信息，如：应用名、服务端口等。
```
spring.application.name=api-gateway
server.port=5555
```
只是配置这些还远远不够，我们还要在这里配置路由策略，路由的映射有两种配置方式：
一种是通过跳转url：
```
# routes to url
zuul.routes.api-a-url.path=/api-a-url/**
zuul.routes.api-a-url.url=http://localhost:2222/
```
这种方式是将所有的/api-a-url/**的请求跳转到http://localhost:2222/，然而这种配置方式不够友好，因为你需要知道服务的地址才能进行配置。
我们推荐通过serviceId进行映射的方式：
```
zuul.routes.api-a.path=/api-a/**
zuul.routes.api-a.serviceId=service-A
zuul.routes.api-b.path=/api-b/**
zuul.routes.api-b.serviceId=service-B
eureka.client.serviceUrl.defaultZone=http://localhost:1111/eureka/
```
我们把zuul注册到Eureka Server上去发现服务，然后只需要配置需要跳转的serviceId，具体要跳转的url无需关心。

接下来验证一下我们Zuul配置的路由策略是否生效：
首先启动我们最早搭建的Eureka注册中心，还有compute-service服务，我们将compute-service服务分别以service-A、service-B的服务名称在2222和3333端口启动，然后启动我们的Zuul服务。
打开Eureka注册中心：
![image_1biga7bui17p11mba1me014ltfcb9.png-37.1kB][1]
可以看到service-A、service-B以及我们的Zuul均已注册成功。
我们首先验证url的配置路由，访问http://localhost:5555/api-a-url/add?a=1&b=2
![image_1bigan924g5ciek2ageou17bjm.png-13.2kB][2]
service-A后台输出：
![image_1bigb0r21cdr131l1ijk1n391q9t1t.png-38.2kB][3]
再验证通过serviceId映射配置，访问http://localhost:5555/api-b/add?a=1&b=2
![image_1bigau3hmj943dhna710i6dts13.png-13.3kB][4]
service-B后台输出：
![image_1bigb011k6m91ango7bn3fb2s1g.png-50.7kB][5]

除此之外，Zuul还有着强大的过滤功能，比如外部访问安全控制。  
我们来实现一个Zuul过滤功能，外部必须通过有效的accessToken才能进行服务调用：  
首先要继承ZuulFilter，并重写它的四个抽象方法。
```java
public class AccessFilter extends ZuulFilter  {
    private static Logger log = LoggerFactory.getLogger(AccessFilter.class);
    @Override
    public String filterType() {
        return "pre";
    }
    @Override
    public int filterOrder() {
        return 0;
    }
    @Override
    public boolean shouldFilter() {
        return true;
    }
    @Override
    public Object run() {
        RequestContext ctx = RequestContext.getCurrentContext();
        HttpServletRequest request = ctx.getRequest();
        log.info(String.format("%s request to %s", request.getMethod(), request.getRequestURL().toString()));
        Object accessToken = request.getParameter("accessToken");
        if(accessToken == null) {
            log.warn("access token is empty");
            ctx.setSendZuulResponse(false);
            ctx.setResponseStatusCode(401);
            return null;
        }
        log.info("access token ok");
        return null;
    }
}
```
`filterType`：返回一个字符串代表过滤器的类型，在zuul中定义了四种不同生命周期的过滤器类型，具体如下：

- `pre`：可以在请求被路由之前调用
- `routing`：在路由请求时候被调用
- `post`：在routing和error过滤器之后被调用
- `error`：处理请求时发生错误时被调用

`filterOrder`：通过int值来定义过滤器的执行顺序
`shouldFilter`：返回一个boolean类型来判断该过滤器是否要执行，所以通过此函数可实现过滤器的开关。在上例中，我们直接返回true，所以该过滤器总是生效。
`run`：过滤器的具体逻辑。需要注意，这里我们通过`ctx.setSendZuulResponse(false)`令zuul过滤该请求，不对其进行路由，然后通过`ctx.setResponseStatusCode(401)`设置了其返回的错误码，当然我们也可以进一步优化我们的返回，比如，通过`ctx.setResponseBody(body)`对返回body内容进行编辑等。

之后我们需要在启动类中添加`@bean`，对定义的过滤器进行实例化。
```java
@EnableZuulProxy
@SpringCloudApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
	@Bean
	public AccessFilter accessFilter() {
		return new AccessFilter();
	}
}
```
重启Zuul服务，再次访问：http://localhost:5555/api-a/add?a=1&b=2，无返回结果：
![image_1bigbjbvg1m4d1hvsm28733n6k2a.png-11.8kB][6]
后台服务日志：
![image_1bigbkv3jkub1ph01vpq3581r6l2n.png-48.5kB][7]

关于Zuul过滤器的功能还有很多，本文暂到此，在后续的实际应用中再单独研究。

本文源码已上传github：https://github.com/coldxiangyu/spring-cloud-demo/tree/master/zuul-gateway


  [1]: http://static.zybuluo.com/coldxiangyu/7lmzygvmj2sxt1oz6jfty48z/image_1biga7bui17p11mba1me014ltfcb9.png
  [2]: http://static.zybuluo.com/coldxiangyu/vid7vm179mclcyrn9u1zvdb8/image_1bigan924g5ciek2ageou17bjm.png
  [3]: http://static.zybuluo.com/coldxiangyu/7xmponhg8iu97mqqkujiylay/image_1bigb0r21cdr131l1ijk1n391q9t1t.png
  [4]: http://static.zybuluo.com/coldxiangyu/f6bjrnm1jvqk7l7dgqgolnm6/image_1bigau3hmj943dhna710i6dts13.png
  [5]: http://static.zybuluo.com/coldxiangyu/3t8a2aq8hbati3xbq63lptd1/image_1bigb011k6m91ango7bn3fb2s1g.png
  [6]: http://static.zybuluo.com/coldxiangyu/nn69futan3jym9vagq0btoko/image_1bigbjbvg1m4d1hvsm28733n6k2a.png
  [7]: http://static.zybuluo.com/coldxiangyu/7q8y8d1h23q9jh7fkmi46u52/image_1bigbkv3jkub1ph01vpq3581r6l2n.png
