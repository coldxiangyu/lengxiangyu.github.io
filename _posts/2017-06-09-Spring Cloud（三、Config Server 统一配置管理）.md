---
layout: post
title:  "Spring Cloud（三、Config Server 统一配置管理）"
date:   2017-06-09 11:21:19
categories: WEB
tags: spring-cloud
mathjax: true
---

* content
{:toc}

微服务架构一般由单一服务进行细粒度拆分，使得微服务数量众多，而且每个服务都要单独配置，所以一套集中式的、动态配置管理必不可少。Spring Cloud提供了Config Server，功能如下：

- 集中化的配置文件管理 
不再需要在每个服务部署的机器上编写配置文件，服务会向配置中心统一拉取配置自己的信息。

- 动态化的配置更新 
当配置发生变动时，服务不需要重启即可感知到配置的变化并应用新的配置。

- 将配置信息以HTTP REST接口的形式暴露 




Config Server的架构图如下： 
![image_1bi5apavisro1ihb1lc7c716uo9.png-129.7kB][1]
工作流程如下：

1. 配置中心感知到配置变化(如，git仓库发生commit提交)，向Bus投递消息
2. Bus向服务广播消息
3. 服务收到消息后，主动向配置中心拉取新配置并应用

其中配置中心可集群部署实现高可用。

接下来我们来配置一个Config Server：
新建config-server模块，pom配置如下：
```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>1.5.3.RELEASE</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <spring-cloud.version>Dalston.RELEASE</spring-cloud.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-config-server</artifactId>
        </dependency>

        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
```
配置application.properties：
```
spring.application.name=config-server
server.port=7001
# git管理配置
spring.cloud.config.server.git.uri=https://github.com/coldxiangyu/spring-cloud-demo/
spring.cloud.config.server.git.searchPaths=config-repository
spring.cloud.config.server.git.username=username
spring.cloud.config.server.git.password=password
```
- spring.cloud.config.server.git.uri：配置git仓库位置
- spring.cloud.config.server.git.searchPaths：配置仓库路径下的相对搜索位置，可以配置多个
- spring.cloud.config.server.git.username：访问git仓库的用户名
- spring.cloud.config.server.git.password：访问git仓库的用户密码

当然，Config server也支持本地文件存储的配置形式，如下：
```
spring.profiles.active=native
spring.cloud.config.server.native.searchLocations=file:D:/
```
不过还是推荐git的形式进行版本控制。

最后创建Spring Boot的程序主类，并添加@EnableConfigServer注解，开启Config Server
```java
@EnableConfigServer
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
接下来我们需要验证一下刚刚建立起来的Config Server能否供外部访问，我们在我们的git工程上再创建一层目录[config-repository](https://github.com/coldxiangyu/spring-cloud-demo/tree/master/config-repository)，专门存储相关配置文件。
上传下面四个文件，分别存储不同环境的配置。
```
coldxiangyu.properties
coldxiangyu-dev.properties
coldxiangyu-test.properties
coldxiangyu-prod.properties
```
存储内容分别如下，设置from属性，便于我们稍后客户端进行读取：
```
from=git-default-1.0
from=git-dev-1.0
from=git-test-1.0
from=git-prod-1.0
```
然后我们创建一个新的test分支，将1.0版本更新为2.0.
到这里，我们启动我们的Config Server，尝试在浏览器或者POSTMAN进行访问。

URL与配置文件的映射关系如下：

- /{application}/{profile}[/{label}]
- /{application}-{profile}.yml
- /{label}/{application}-{profile}.yml
- /{application}-{profile}.properties
- /{label}/{application}-{profile}.properties

上面的url会映射{application}-{profile}.properties对应的配置文件，{label}对应git上不同的分支，默认为master。
下面我们来访问：http://localhost:7001/coldxiangyu/dev/test
访问json结果如下：
```
{
    "name": "coldxiangyu",
    "profiles": [
        "dev"
    ],
    "label": "test",
    "version": null,
    "state": null,
    "propertySources": [
        {
            "name": "https://github.com/coldxiangyu/spring-cloud-demo/config-repository/coldxiangyu-dev.properties",
            "source": {
                "from": "git-dev-2.0"
            }
        },
        {
            "name": "https://github.com/coldxiangyu/spring-cloud-demo/config-repository/coldxiangyu.properties",
            "source": {
                "from": "git-default-2.0"
            }
        }
    ]
}
```
浏览器可以访问之后，我们来尝试在实际的微服务端是如何在Config Server中心拉取配置的。
创建config-client模块，pom配置如下：
```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>1.3.5.RELEASE</version>
    <relativePath/> <!-- lookup parent from repository -->
  </parent>
  <dependencies>
    <dependency>
      <groupId>org.springframework.cloud</groupId>
      <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!-- Allow for automatic restarts when classpath contents change. -->
    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-devtools</artifactId>
      <optional>true</optional>
    </dependency>

    <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
  </dependencies>
  <dependencyManagement>
    <dependencies>
      <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-parent</artifactId>
        <version>Brixton.SR4</version>
        <type>pom</type>
        <scope>import</scope>
      </dependency>
    </dependencies>
  </dependencyManagement>
```
创建`bootstrap.properties`，指定config server：
```
#对应application
spring.application.name=coldxiangyu
#对应profile
spring.cloud.config.profile=dev
#对应前配置文件的git分支
spring.cloud.config.label=test
#配置中心的地址
spring.cloud.config.uri=http://localhost:7001/
server.port=7002
```
这里要注意我们配置的是`bootstrap.properties`而不是`application.properties`，因为`bootstrap.properties`会在应用启动之前读取,而`spring.cloud.config.uri`会影响应用启动。

接着创建一个REST api获取配置中心from属性：
```java
@RefreshScope
@RestController
class TestController {
    @Value("${from}")
    private String from;
    @RequestMapping("/from")
    public String from() {
        return this.from;
    }
}
```

通过`@Value("${from}")`绑定配置服务中配置的from属性。`@RefreshScope`配置，可以使配置通过/refresh刷新之后，加载新的配置。

我们再创建spring boot启动类：
```java
@EnableDiscoveryClient
@SpringBootApplication
public class Application {
	public static void main(String[] args) {
		new SpringApplicationBuilder(Application.class).web(true).run(args);
	}
}
```
分别启动config server以及config client，访问：http://localhost:7002/from，如下：
![image_1bidio62f1t1l1p5m14uh1q5p14m9.png-6.8kB][2]
可以看到，我们成功获取到了配置中心的配置信息。
我们通过git修改此文件内容为`git-dev-3.0`，通过访问http://localhost:7001/coldxiangyu/dev/test，我们可以看到，配置中心已经发生了变化。
![image_1bidj149r4v31m2sqreu69g2qm.png-15.7kB][3]
再次访问http://localhost:7002/from，发现获取到的信息仍然是2.0版本。这时候我们需要向客户端发送一个POST请求：http://localhost:7002/refresh：
![image_1bidj5mg1iua1uf6180rsiq1qln13.png-19.7kB][4]
返回了405的错误信息，具体原因我还没来得及研究，我们换一种姿势进行POST请求，浏览器请求可能会有问题，我们改用`curl -d {} http://localhost:7002/refresh`命令进行refresh：
![image_1bidj89671ilh19mq1iusd6tn8b1g.png-11.7kB][5]
这时候再访问http://localhost:7002/from：
![image_1bidjcet0fj0cqi1k6ium71ar31t.png-9kB][6]
我们在未启动客户端服务的情况下，成功刷新。
不过在应用级的项目中，这样肯定还是过于麻烦的，可以通过Spring Cloud Bus来实现以消息总线的方式通知配置信息的变化，实现自动化更新，也就是我们最上面贴的config Server架构说明。这种方式暂时超出了我们本篇文章的研究范围，后续研究。

本文源码已上传github：https://github.com/coldxiangyu/spring-cloud-demo

  [1]: http://static.zybuluo.com/coldxiangyu/h7kkvxvifk9wo9td9a4bmafz/image_1bi5apavisro1ihb1lc7c716uo9.png
  [2]: http://static.zybuluo.com/coldxiangyu/sa1iqlrzhdsgifoucnbvxfv5/image_1bidio62f1t1l1p5m14uh1q5p14m9.png
  [3]: http://static.zybuluo.com/coldxiangyu/9r5dgln0oxhfm3rzzcjtyvku/image_1bidj149r4v31m2sqreu69g2qm.png
  [4]: http://static.zybuluo.com/coldxiangyu/sz1fvde4yjquh2i989kcno4n/image_1bidj5mg1iua1uf6180rsiq1qln13.png
  [5]: http://static.zybuluo.com/coldxiangyu/4an2mcsojf0eg2ftq8i3wf6l/image_1bidj89671ilh19mq1iusd6tn8b1g.png
  [6]: http://static.zybuluo.com/coldxiangyu/ij5yqu1ip2tje7goagv9r2vr/image_1bidjcet0fj0cqi1k6ium71ar31t.png
