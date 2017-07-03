---
layout: post
title:  "Spring Cloud（二、断路器Hystrix）"
date:   2017-06-05 11:31:41
categories: WEB
tags: spring-cloud Hystrix
mathjax: true
---

* content
{:toc}

我们在上一篇文章介绍了Spring Cloud的服务注册、消费以及负载均衡。这篇讲一下`Netfilx Hystrix`，断路器。

`Hystrix`是`Netflix`开源的微服务框架套件之一，该框架目标在于通过控制那些访问远程系统、服务和第三方库的节点，从而对延迟和故障提供更强大的容错能力。`Hystrix`具备拥有回退机制和断路器功能的线程和信号隔离，请求缓存和请求打包，以及监控和配置等功能。




在微服务架构中，我们将系统拆分为一个个不同的单元，比如我们的Eureka注册中心、服务提供方、Ribbon、Feign消费者，它们彼此通过服务注册订阅的方式进行相互依赖，且分别运行在不同的进程中，通过远程调用的方式执行。
如果遇到网络延迟或者高负载导致某个服务无法及时响应，导致请求积压阻塞，最终会拖垮与此相关的整个业务流程。
这也是每一个微服务架构面临的主要问题，`Hystrix`也就应运而生了。
有了`Hystrix`，它可以监控整个系统中的故障，并返回错误信息，避免请求积压，故障蔓延。

下面我们来看看如何使用`Hystrix`：
重新启动我们的服务注册中心：http://localhost:1111
不进行服务注册，直接运行消费者ribbon之后，访问http://localhost:3333/add，可以看到以下错误页面。
![image_1bi2ntecql5v161f8dchb113t9.png-23.6kB][1]

我们在ribbon中加入`Hystrix`，在pom中增加`hystrix`依赖：
```xml
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-hystrix</artifactId>
</dependency>
```
在`eureka-ribbon`的主类RibbonApplication中使用`@EnableCircuitBreaker`注解开启断路器功能：
```java
@SpringBootApplication
@EnableDiscoveryClient
@EnableCircuitBreaker
public class RibbonApplication {
	@Bean
	@LoadBalanced
	RestTemplate restTemplate() {
		return new RestTemplate();
	}
	public static void main(String[] args) {
		SpringApplication.run(RibbonApplication.class, args);
	}
}
```
改造原来的服务消费方式，新增ComputeService类，在使用ribbon消费服务的函数上增加`@HystrixCommand`注解来指定回调方法。
```java
@Service
public class ComputeService {
    @Autowired
    RestTemplate restTemplate;
    @HystrixCommand(fallbackMethod = "addServiceFallback")
    public String addService() {
        return restTemplate.getForEntity("http://COMPUTE-SERVICE/add?a=10&b=20", String.class).getBody();
    }
    public String addServiceFallback() {
        return "error";
    }
}
```
提供rest接口的Controller改为调用ComputeService的addService
```java
@RestController
public class ConsumerController {
    @Autowired
    private ComputeService computeService;
    @RequestMapping(value = "/add", method = RequestMethod.GET)
    public String add() {
        return computeService.addService();
    }
}
```
这样一来，在调用addService方法异常之后，直接调用回调函数进行返回。
我们启动ribbon消费者验证进行验证，访问http://localhost:3333/add
![image_1bi363ie1l6bed711ol7qgucam.png-13.3kB][2]

与Ribbon相比，Feign使用Hystrix就方便多了，因为Feign本身是依赖Hystrix的，我们无需修改pom。

我们需要对之前的eureka-feign进行改造，使用@FeignClient注解中的fallback属性指定回调类：
```java
@FeignClient(value = "compute-service", fallback = ComputeClientHystrix.class)
public interface ComputeClient {
    @RequestMapping(method = RequestMethod.GET, value = "/add")
    Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b);
}
```
创建回调类ComputeClientHystrix，实现@FeignClient的接口，此时实现的方法就是对应@FeignClient接口中映射的fallback函数。
```java
@Component
public class ComputeClientHystrix implements ComputeClient {
    @Override
    public Integer add(@RequestParam(value = "a") Integer a, @RequestParam(value = "b") Integer b) {
        return -9999;
    }
}
```
启动Feign消费者，访问http://localhost:3333/add
![image_1bi36jd2aso6out1f881iqmnb313.png-9.5kB][3]
本文源码已上传github：
[ribbon-hystrix](https://github.com/coldxiangyu/spring-cloud-demo/tree/master/ribbon-hystrix)
[feign-hystrix](https://github.com/coldxiangyu/spring-cloud-demo/tree/master/feign-hystrix)

  [1]: http://static.zybuluo.com/coldxiangyu/xjmlm4edt9pzksoeoni557ms/image_1bi2ntecql5v161f8dchb113t9.png
  [2]: http://static.zybuluo.com/coldxiangyu/smb0wjxmlwhq19mhp4qshtf5/image_1bi363ie1l6bed711ol7qgucam.png
  [3]: http://static.zybuluo.com/coldxiangyu/qhurx3v77jlr9ba07rv9shcd/image_1bi36jd2aso6out1f881iqmnb313.png
