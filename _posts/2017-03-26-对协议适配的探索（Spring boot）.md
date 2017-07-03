---
layout: post
title:  "对协议适配的探索（Spring boot）"
date:   2017-03-26 16:59:12
categories: WEB
tags: spring-boot protocol webservice socket
mathjax: true
---

前两天，领导下达指示，探索协议适配的可行性。  
所谓的协议适配，就是服务层提供的多协议服务，比如webservice（soap）、http接口、FTP、socket（TCP/IP）等，对外无差别暴露统一的IP和端口，而客户端由此统一IP端口接收请求之后进行适配调用不同的服务。  
听上去有那么点意思。  
当然，首先要搭建一个工程试验一下才行。  
试验的话，肯定不能在搭建项目上花费太多时间，spring boot是不二之选。  
源码已上传github：https://github.com/coldxiangyu/protocoladapter  




我们要研究多协议，所以肯定要加入webservice，webservice采用cxf发布。  
我们除了引入spring boot的包还要引入cxf的包：
```xml
<dependency>
	<groupId>org.apache.cxf</groupId>
	<artifactId>cxf-rt-frontend-jaxws</artifactId>
	<version>3.1.6</version>
</dependency>
<dependency>
	<groupId>org.apache.cxf</groupId>
	<artifactId>cxf-rt-transports-http</artifactId>
	<version>3.1.6</version>
</dependency>
```
这时候，开发条件就已经满足了。  
我们来实现一个webservice版的hello，这和之前spring整合cxf通过配置XML是不一样的。

1.定义接口方法：

```java
package com.lxy.webservice;

import javax.jws.WebMethod;
import javax.jws.WebService;

@WebService
public interface HelloService {
	@WebMethod
    String getName(String param);
}

```

2.定义接口实现方法：

```java
package com.lxy.webservice.impl;

import javax.jws.WebService;

import com.lxy.webservice.HelloService;

@WebService(targetNamespace="http://webservice.lxy.com/",endpointInterface = "com.lxy.webservice.HelloService")
public class HelloServiceImpl implements HelloService {

	@Override
	public String getName(String param) {
		return "hello "+param;
	}

}

```
3.这时候我们就可以对我们定义的webservice进行发布了。发布的方式有两种，一种是默认发布，还有一种就是自定义发布。  
我们看看默认发布：
```java
package com.lxy.cxf;

import javax.xml.ws.Endpoint;

import org.apache.cxf.Bus;
import org.apache.cxf.bus.spring.SpringBus;
import org.apache.cxf.jaxws.EndpointImpl;
import org.apache.cxf.transport.servlet.CXFServlet;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;

import com.lxy.webservice.HelloService;
import com.lxy.webservice.impl.HelloServiceImpl;

@SpringBootApplication
public class CxfConfig {
	
	public static void main(String[] args) {
        SpringApplication.run(CxfConfig.class, args);
    }
	
    @Bean
    public ServletRegistrationBean dispatcherServlet() {
        return new ServletRegistrationBean(new CXFServlet(), "/webservices/*");
    }
    @Bean(name = Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {
        return new SpringBus();
    }
    @Bean
    public HelloService helloService() {
        return new HelloServiceImpl();
    }
    @Bean
    public Endpoint endpoint() {
        EndpointImpl endpoint = new EndpointImpl(springBus(), helloService());
        endpoint.publish("/hello");
        return endpoint;
    }
    
}
```
默认发布实现非常简单，只需定义webservice发布路径，通过`endpoint.publish`发布方法即可。  
如果默认的发布不满足我们的需求怎么办呢，比如默认的发布端口是8080，我们想用8081进行发布。也非常简单，我们只需让此类实现EmbeddedServletContainerCustomizer接口，然后通过重写customize方法，进行container的重新设定即可。
```java
@SpringBootApplication
public class Application implements EmbeddedServletContainerCustomizer  {

    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);
    }
    @Override
    public void customize(ConfigurableEmbeddedServletContainer container) {
        container.setPort(8081);
    }


    @Bean
    public ServletRegistrationBean cxfServlet() {
        return new ServletRegistrationBean(new CXFServlet(), "/webservices/*");
    }

    @Bean(name = Bus.DEFAULT_BUS_ID)
    public SpringBus springBus() {
        return new SpringBus();
    }

    @Bean
    public Endpoint endpoint() {
        EndpointImpl endpoint = new EndpointImpl(springBus(),new HelloServiceImpl());
        endpoint.publish("/hello");
        return endpoint;
    }
```
现在我们通过main方法启动，通过我们定义的webservice地址：http://localhost:8080/webservices/hello?wsdl 查看是否发布成功。
![image_1bhj7qb00dcecveodd3m1nni9.png-70.9kB][1]
我们可以看到，webservice已经成功发布，通过 http://localhost:8080/webservices 我们可以查看所有的webservice：
![image_1bhj7sghu15l9cq01bnj1b428jb13.png-27.7kB][2]
我们再编写cxf客户端代码调用一下：
```java
package com.lxy.client;

import org.apache.cxf.endpoint.Client;
import org.apache.cxf.jaxws.endpoint.dynamic.JaxWsDynamicClientFactory;

public class CxfClient {
	public static void main(String[] args) throws Exception{
		JaxWsDynamicClientFactory dcf = JaxWsDynamicClientFactory.newInstance();
		Client client = dcf.createClient("http://localhost:8080/webservices/hello?wsdl");
		Object[] objects = client.invoke("getName", "tom");
		System.out.println(objects[0].getClass());
		System.out.println(objects[0].toString());
	}
}
```
![image_1bhj84c88oq639hb2u1i04n2q1g.png-25.6kB][3]
调用成功！  
OK，webservice我们进行到这里。

---
接下来要进行socket编程了。  
到这里就引发了我的思考，刚刚发布的webservice已经将8080端口占用了，如果socket继续监听8080肯定会引发端口冲突，我们如何进行统一端口之后无差别的进行协议适配呢？  
感觉不太现实，不过还是先写个socket服务再说。  
首先定义socket server服务端：
```java
package com.lxy.socket;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;

public class SocketServer {
	public static void main(String[] args) throws IOException{
		
		ServerSocket serverSocket = new ServerSocket(8080);
		Socket socket = serverSocket.accept();
		InputStream is = socket.getInputStream();
		InputStreamReader isr = new InputStreamReader(is);
		BufferedReader br = new BufferedReader(isr);
		String info = "";
		while((info = br.readLine())!=null){
			System.out.println("Hello,我是服务器，客户端说："+info);
		}
		socket.shutdownInput();
		OutputStream os = socket.getOutputStream();
		PrintWriter pw = new PrintWriter(os);
		pw.write("Hello World！");
		pw.flush();
		pw.close();
		os.close();
		br.close();
		isr.close();
		is.close();
		socket.close();
		serverSocket.close();
	}
}
```
服务端编写完毕，直接启动。
![image_1bhj8ag2s1p93n8l1v0f1i3q31h1t.png-25.7kB][4]  
不出所料，端口占用，我们姑且先改成8081，重新启动，启动成功！  
我们继续编写客户端socket：
```java
package com.lxy.client;

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.io.OutputStream;
import java.io.PrintWriter;
import java.net.Socket;

public class SocketClient {
	public static void main(String[] args) throws IOException{
		Socket socket = new Socket("127.0.0.1",8080);
		OutputStream os = socket.getOutputStream();
		PrintWriter pw = new PrintWriter(os);
		pw.write("用户名：admin；密码：admin");
		pw.flush();
		socket.shutdownOutput();
		InputStream is = socket.getInputStream();
		BufferedReader br = new BufferedReader(new InputStreamReader(is));
		String info = null;
		while((info=br.readLine()) != null){
		 System.out.println("Hello,我是客户端，服务器说："+info);
		}
		  
		br.close();
		is.close();
		pw.close();
		os.close();
		socket.close();
	}
}

```
启动客户端，查看server与client的打印日志：  
客户端：
![image_1bhj8f76jfqdi8mv6urrv1h7b2n.png-5.8kB][5]
服务端：
![image_1bhj8fmvskee16o6lokktb4ut34.png-5.9kB][6]
那我们用socket客户端请求一下webservice的8080端口呢？
![image_1bhj8ddnb1a318vkslo1i1a1p092a.png-16.8kB][7]
我们可以看到，webservice不识别此请求，返回400报错。  
webservice服务后台也对应报错：
![image_1bhj8s553v6d1ih571runnpv63h.png-47.3kB][8]

到现在为止，我感觉从代码层面来考虑这个问题已经不太现实。

考虑到实际项目是nginx负载均衡，对外暴露统一IP、端口，nginx有没有相关配置呢？

我了解到nginx 1.9.0版本开始增加了stream模块用于一般的TCP代理和负载均衡，但是也是要知道该IP和端口是用于TCP协议的才可以。nginx无法通过统一的IP、端口判断协议。

再有就是考虑是否可以在整个项目的维度上进行所有请求的拦截呢？包括http、socket、soap、ftp等请求。我们知道SpringMVC只是拦截所有的http请求，也就是浏览器请求，拦截TCP等协议的请求，一般都要动用防火墙或者黑客攻击技术，所以这点也没有办法实现。

对协议适配研究到这里，我给了我们领导一个NO的回复，很遗憾。


  [1]: http://static.zybuluo.com/coldxiangyu/7ruvcbxgc58eldnj41rf64x2/image_1bhj7qb00dcecveodd3m1nni9.png
  [2]: http://static.zybuluo.com/coldxiangyu/ijamjkj7i8zch40w8pmc2myx/image_1bhj7sghu15l9cq01bnj1b428jb13.png
  [3]: http://static.zybuluo.com/coldxiangyu/wpoat3jmmf7l6vnqd8x9hwge/image_1bhj84c88oq639hb2u1i04n2q1g.png
  [4]: http://static.zybuluo.com/coldxiangyu/ryiz3mprzbs598tcwoblocdk/image_1bhj8ag2s1p93n8l1v0f1i3q31h1t.png
  [5]: http://static.zybuluo.com/coldxiangyu/txfwvwow184muoshk1j8tgo6/image_1bhj8f76jfqdi8mv6urrv1h7b2n.png
  [6]: http://static.zybuluo.com/coldxiangyu/l6gqcu8dmc2upu7fl4xjn64e/image_1bhj8fmvskee16o6lokktb4ut34.png
  [7]: http://static.zybuluo.com/coldxiangyu/64y9bd1sehsrc867348tecgc/image_1bhj8ddnb1a318vkslo1i1a1p092a.png
  [8]: http://static.zybuluo.com/coldxiangyu/e7lgtymkb1hxsf4gliedev5a/image_1bhj8s553v6d1ih571runnpv63h.png
