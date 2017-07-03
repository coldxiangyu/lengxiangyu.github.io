---
layout: post
title:  "Spring boot相关研究"
date:   2016-05-15 13:52:12
categories: WEB
tags: spring-boot
mathjax: true
---

* content
{:toc}

### 前言
熟悉Spring完整生态的人应该都听说过spring boot，没有关心过它的也没关系，我们现在来认识一下它。  
你需要记住一点，所有的架构方案都是在努力的简化开发人员的工作量的。从最开始spring的出现，service注入的方式走向我们，AOP概念的出现，都是为了简化开发，增加配置，减少代码。  




这样的发展过程中，spring开始面临一个问题，就是过多的配置文件。  
正如我在经代通的环境搭建中所作的一样，首先你要配置web.xml，配置spring的监听器，你要加入applicationContext.xml，在项目运行时加载spring的相关配置。最后你还要将项目打成war包放入tomcat或者jetty运行。  
虽然这些算是一劳永逸的工作，但是，这对于开发人员是极不友好的。回想经代通的搭建过程，由于网络限制，没有复制粘贴的条件，xml文件配置都是一个字母一个字母打出来的，极其痛苦。  
不过，随着Spring 3.0的发布，Spring IO团队逐渐开始摆脱XML配置文件，并且在开发过程中大量使用“约定优先配置”（convention over configuration）的思想来摆脱Spring框架中各类繁复纷杂的配置（即时是Java Config）。  
`Spring boot`它本身并不提供Spring框架的核心特性以及扩展功能，只是用于快速、敏捷地开发新一代基于Spring框架的应用程序。也就是说，它并不是用来替代Spring的解决方案，而是和Spring框架紧密结合用于提升Spring开发者体验的工具。同时它集成了大量常用的第三方库配置（例如Jackson, JDBC, Mongo, Redis, Mail等等），Spring Boot应用中这些第三方库几乎可以零配置的开箱即用（out-of-the-box），大部分的Spring Boot应用都只需要非常少量的配置代码，开发者能够更加专注于业务逻辑。

### Hello world！
---
接下来我们就来做一个“Hello world”的demo来看一看，spring boot到底有多简化。
以Maven项目为例，首先引入Spring Boot的开发依赖：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.4.1.RELEASE</version>
</parent>
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```
编写一个类包含处理HTTP请求的方法以及一个main()函数：
```java
@Controller
@EnableAutoConfiguration
public class SampleController {

    @RequestMapping("/")
    @ResponseBody
    String home() {
        return "Hello World!";
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleController.class, args);
    }
}
```
到这里，已经完事了。我们通过main函数中的`SpringApplication.run(SampleController.class, args)`就可以将整个springMVC运行起来，运行环境是一个内嵌的tomcat。  
运行main函数，控制台输出如下：
![image_1bhen0pcggue1r7tmubd3drgm9.png-106.5kB][1]
我们可以看到，启动了一个tomcat。  
我们通过浏览器访问一下http://localhost:8080/：
![image_1bhen5mh914ft1gn40cbmlofgm.png-13.9kB][2]
真的，简单到我都想骂人。

### spring boot 解析
---
我们在上面也说了，spring boot的本质就是一堆库的集合，可以被任意构建的项目所使用。
  
我们来回想一下我们都做了什么，便可以直接使用这些库了。  
首先，在Maven依赖中引入了`spring-boot-starter-web`，它包含了Spring Boot预定义的一些Web开发的常用依赖:

- `spring-web, spring-webmvc Spring WebMvc`框架
- `tomcat-embed-*` 内嵌Tomcat容器
- `jackson` 处理json数据
- `spring-*` Spring框架
- `spring-boot-autoconfigure` Spring Boot提供的自动配置功能
Java代码中没有任何配置，和传统的Spring应用相比，多了两个我们不认识的符号：

- `@EnableAutoConfiguration`
- `SpringApplication`
它们都是由Spring Boot框架提供。在`SpringApplication.run()`方法执行后，Spring Boot的autoconfigure发现这是一个Web应用（根据类路径上的依赖确定），于是在内嵌的Tomcat容器中启动了一个Spring的应用上下文，并且监听默认的tcp端口8080（默认约定）。同时在Spring Context中根据默认的约定配置了Spring WebMvc：

- Servlet容器默认的Context路径是/
- DispatherServlet匹配的路径(servlet-mapping中的url-patterns)是/*
- @ComponentScan路径被默认设置为SampleController的同名package，也就是该package下的所有@Controller，@Service, @Component,@Repository都会被实例化后并加入Spring Context中。

我们没有一行配置代码、也没有`web.xml`。基于Spring Boot的应用在大多数情况下都不需要我们去显式地声明各类配置，而是将最常用的默认配置作为约定，在不声明的情况下也能适应大多数的开发场景。

###数据库操作
上面介绍了最简单的hello world例子，在真正的项目中还是远远不够的，我们还要访问数据库，正如我们通常配置的jdbc的config文件，username、url、password之类，还有整合mybatis等持久化工具，druid、c3p0数据源之类的。  
在spring boot中，这些依然通过maven引入依赖即可：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web-jdbc</artifactId>
</dependency>
<dependency>
    <groupId>com.h2database</groupId>
    <artifactId>h2</artifactId>
</dependency>
```
`spring-boot-starter-web-jdbc`引入了`spring-jdbc`依赖，`h2`是一个内存关系型数据库。在引入了这些依赖并启动Spring Boot应用程序后，`autoconfigure`发现`spring-jdbc`位于类路径中，于是：

- 根据类路径上的JDBC驱动类型（这里是h2，预定义了derby, sqlite, mysql, oracle, sqlserver等等），创建一个DataSource连接池对象，本例中的h2是内存数据库，无需任何配置，如果是mysql, oracle等类型的数据库需要开发者配置相关信息。
- 在Spring Context中创建一个JdbcTemplate对象（使用DataSource初始化）

接下来开发者的工作就非常简单了，在业务逻辑中直接引入`JdbcTemplate`即可：
```java
@Service
public class MyService {

    @Autowired
    JdbcTemplate jdbcTemplate;

}
```
除了`spring-jdbc`，Spring Boot还能够支持JPA，以及各种NoSQL数据库——包括MongoDB，Redis，全文索引工具`elasticsearch`, `solr`等等。

### 配置
---
Spring Boot最大的特色是“约定优先配置”，大量的默认配置对开发者十分的友好。但是在实际的应用开发过程中，默认配置不可能满足所有场景，同时用户也需要配置一些必须的配置项——例如数据库连接信息。Spring Boot的配置系统能够让开发者快速的覆盖默认约定，同时支持Properties配置文件和YAML配置文件两种格式，默认情况下Spring Boot加载类路径上的`application.properties`或`application.yml`文件，例如：
```
spring.datasource.url=jdbc:mysql://localhost/test
spring.datasource.username=dbuser
spring.datasource.password=dbpass
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```
YAML格式更加简洁：
```yml
spring:
  datasource:
    url: jdbc:mysql://localhost/test
    username: dbuser
    password: dbpass
    driver-class: com.mysql.jdbc.Driver
```
一旦发现这些信息，Spring Boot就会根据它们创建DataSource对象。另一个常见的配置场景是Web应用服务器：
```yml
# Server settings (ServerProperties)
server:
  port: 8080
  address: 127.0.0.1
  sessionTimeout: 30
  contextPath: /

  # Tomcat specifics
  tomcat:
    accessLogEnabled: false
    protocolHeader: x-forwarded-proto
    remoteIpHeader: x-forwarded-for
    basedir:
    backgroundProcessorDelay: 30 # secs
```
通过`port`和`address`可以修改服务器监听的地址和端口，`sessionTimeout`配置session过期时间（再也不用修改`web.xml`了，因为它根本不存在）。同时如果在生产环境中使用内嵌Tomcat，当然希望能够配置它的日志、线程池等信息，这些现在都可以通过Spring Boot的属性文件配置，而不再需要再对生产环境中的Tomcat实例进行单独的配置管理了。

### @EnableAutoCongiguration
---
从Spring 3.0开始，为了替代繁琐的XML配置，引入`了@Enable...`注解对`@Configuration`类进行修饰以达到和XML配置相同的效果。想必不少开发者已经使用过类似注解：
`@EnableTransactionManagement`开启Spring事务管理，相当于XMl中的`<tx:*>`
`@EnableWebMvc`使用Spring MVC框架的一些默认配置
`@EnableScheduling`会初始化一个Scheduler用于执行定时任务和异步任务
Spring Boot提供的`@EnableAutoCongiguration`似乎功能更加强大，一旦加上，上述所有的配置似乎都被包含进来而无需开发者显式声明。它究竟是如何做到的呢，先看看它的定义：
```java
@Target(ElementType.TYPE)
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Import({ EnableAutoConfigurationImportSelector.class,
        AutoConfigurationPackages.Registrar.class })
public @interface EnableAutoConfiguration {

    /**
     * Exclude specific auto-configuration classes such that they will never be applied.
     */
    Class<?>[] exclude() default {};

}
```
`EnableAutoConfigurationImportSelector`使用的是`spring-core`模块中的`SpringFactoriesLoader#loadFactoryNames()`方法，它的作用是在类路径上扫描`META-INF/spring.factories`文件中定义的类：
```java
# Initializers
org.springframework.context.ApplicationContextInitializer=\
org.springframework.boot.autoconfigure.logging.AutoConfigurationReportLoggingInitializer

# Auto Configure
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
org.springframework.boot.autoconfigure.amqp.RabbitAutoConfiguration,\
org.springframework.boot.autoconfigure.MessageSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.PropertyPlaceholderAutoConfiguration,\
org.springframework.boot.autoconfigure.batch.BatchAutoConfiguration,\
org.springframework.boot.autoconfigure.data.JpaRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.data.MongoRepositoriesAutoConfiguration,\
org.springframework.boot.autoconfigure.redis.RedisAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration,\
org.springframework.boot.autoconfigure.jdbc.DataSourceTransactionManagerAutoConfiguration,\
org.springframework.boot.autoconfigure.jms.JmsTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.jmx.JmxAutoConfiguration,\
org.springframework.boot.autoconfigure.mobile.DeviceResolverAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoAutoConfiguration,\
org.springframework.boot.autoconfigure.mongo.MongoTemplateAutoConfiguration,\
org.springframework.boot.autoconfigure.orm.jpa.HibernateJpaAutoConfiguration,\
org.springframework.boot.autoconfigure.reactor.ReactorAutoConfiguration,\
org.springframework.boot.autoconfigure.security.SecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.security.FallbackWebSecurityAutoConfiguration,\
org.springframework.boot.autoconfigure.thymeleaf.ThymeleafAutoConfiguration,\
org.springframework.boot.autoconfigure.web.EmbeddedServletContainerAutoConfiguration,\
org.springframework.boot.autoconfigure.web.DispatcherServletAutoConfiguration,\
org.springframework.boot.autoconfigure.web.ServerPropertiesAutoConfiguration,\
org.springframework.boot.autoconfigure.web.MultipartAutoConfiguration,\
org.springframework.boot.autoconfigure.web.HttpMessageConvertersAutoConfiguration,\
org.springframework.boot.autoconfigure.web.WebMvcAutoConfiguration,\
org.springframework.boot.autoconfigure.websocket.WebSocketAutoConfiguration
```
实际上这就是Spring Boot会自动配置的一些对象，例如前面提到的Web框架由`EmbeddedServletContainerAutoConfiguration`, `DispatcherServletAutoConfiguration`, `ServerPropertiesAutoConfiguration`等配置完成，而`DataSource`的自动配置则是由`DataSourceAutoConfiguration`完成。现在我们以Mongo的配置`MongoAutoConfiguration`为例，来探索Spring Boot是如何完成这些配置的：
```java
@Configuration
@ConditionalOnClass(Mongo.class)
@EnableConfigurationProperties(MongoProperties.class)
public class MongoAutoConfiguration {

    @Autowired
    private MongoProperties properties;

    private Mongo mongo;

    @PreDestroy
    public void close() throws UnknownHostException {
        if (this.mongo != null) {
            this.mongo.close();
        }
    }

    @Bean
    @ConditionalOnMissingBean
    public Mongo mongo() throws UnknownHostException {
        this.mongo = this.properties.createMongoClient();
        return this.mongo;
    }

}
```
首先这是一个Spring的配置`@Configuration`，它定义了我们访问Mongo需要的`@Bean`，如果这个`@Configuration`被Spring Context扫描到，那么Context中自然也就有两个一个Mongo对象能够直接为开发者所用。  
但是注意到其它几个Spring注解：

- `@ConditionOnClass`表明该`@Configuration`仅仅在一定条件下才会被加载，这里的条件是`Mongo.class`位于类路径上
- `@EnableConfigurationProperties`将Spring Boot的配置文件（`application.properties`）中的`spring.data.mongodb.*`属性映射为`MongoProperties`并注入到`MongoAutoConfiguration`中。
- `@ConditionalOnMissingBean`说明Spring Boot仅仅在当前上下文中不存在Mongo对象时，才会实例化一个Bean。这个逻辑也体现了Spring Boot的另外一个特性——自定义的Bean优先于框架的默认配置，我们如果显式的在业务代码中定义了一个`Mongo`对象，那么Spring Boot就不再创建。

接下来看一看`MongoProperties`：
```java
@ConfigurationProperties(prefix = "spring.data.mongodb")
public class MongoProperties {

    private String host;
    private int port = DBPort.PORT;
    private String uri = "mongodb://localhost/test";
    private String database;

    // ... getters/ setters omitted
}
```
显然，它就是以`spring.data.mongodb`作为前缀的属性，然后通过名字直接映射为对象的属性，同时还包含了一些默认值。如果不配置，那么`mongo.uri`就是`mongodb://localhost/test`。

### Production特性
---
从前面的例子可以看出，Spring Boot能够非常快速的做出一些原型应用，但是它同样可以被用于生产环境。为了添加生产环境特性支持，需要在Maven依赖中引入：
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```
加入`actuator`依赖后，应用启动后会创建一些基于Web的Endpoint：

- `/autoconfig`，用来查看Spring Boot的框架自动配置信息，哪些被自动配置，哪些没有，原因是什么。
- `/beans`，显示应用上下文的Bean列表
- `/dump`，显示线程dump信息
- `/health`，应用健康状况检查
- `/metrics`
- `/shutdown`, 默认没有打开
- `/trace`

### 总结
Spring Boot是新一代Spring应用的开发框架，它能够快速的进行应用开发，让人忘记传统的繁琐配置，更加专注于业务逻辑。现在Spring官方文档中所有的[Guide](http://spring.io/guides)中的例子都是使用Spring Boot进行构建，这也是一个学习Spring, Spring Boot非常好的地方。如果想进一步深度学习Spring Boot，可以参考：

- [Spring Boot Reference](http://docs.spring.io/spring-boot/docs/current/reference/htmlsingle/)
- [spring.io网站源码](https://github.com/spring-io/sagan)

  [1]: http://static.zybuluo.com/coldxiangyu/bp1kvy9uw7vmh1eu75w0fmh8/image_1bhen0pcggue1r7tmubd3drgm9.png
  [2]: http://static.zybuluo.com/coldxiangyu/x0sbe5ipqq4xzp80ykt7dayf/image_1bhen5mh914ft1gn40cbmlofgm.png
