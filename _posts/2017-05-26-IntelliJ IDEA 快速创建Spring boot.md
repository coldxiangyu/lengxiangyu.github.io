---
layout: post
title:  "IntelliJ IDEA 快速创建Spring boot"
date:   2016-05-26 09:59:44
categories: 技术储备
tags: IDEA spring-boot
mathjax: true
---

在我的[另一篇文章](https://www.zybuluo.com/coldxiangyu/note/770749)里，已经介绍过Spring boot，以及它的基本配置了。  
那篇文章是在Myeclipse环境通过maven项目搭建起来的，虽然也很容易，但是还有更容易的。今天我要介绍的就是通过IDEA的Spring Initializr创建Spring boot工程。  
接下来来看看到底有多容易。  




在不用IntelliJ IDEA的情况下，我们通常需要访问http://start.spring.io/，生成maven包，然后解压导入到我们的IDE。
![image_1bhu45e0gqkc125014gfi3m13n39.png-47.6kB][1]  
而IntelliJ IDEA与Spring结合的非常好，Spring Initializr已经内置，我们直接在IDEA进行创建即可。
![image_1bhu253ei1ueo1ib0kj9177i1u8lp.png-48kB][2]

接下来，就是http://start.spring.io/的选择界面了：

![image_1bhu26qcj1v4ugd22sa1gqr1li516.png-33.1kB][3]

再接下来，我们可以选择spring boot的版本，各种开发相关的依赖也集中在此处供我们选择，非常的人性化：

![image_1bhu28ufdnjo1ei5i0g1k4cfj91j.png-47.1kB][4]

我们只需简单创建，无需选择，点击下一步，完成即可。

![image_1bhu2hpof142p1hir12pt1vpl14s220.png-63.7kB][5]

我们可以看到，一个完整的spring boot项目已经展现在我们面前，包括：
`src/main/java`下的程序入口：`DemoApplication`
`src/main/resources`下的配置文件：`application.properties`
`src/test/`下的测试入口：`DemoApplicationTests`

其中自动生成的POM配置如下：

![image_1bhu2u63q1jmt1m6d1dbc1vjqb632d.png-92.3kB][6]

自动生成的POM已经非常全面了，不过少了一个`spring-boot-starter-web`，我们添加web依赖即可。
```xml
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```
我们改造入口类，实现一个hello world！
```java
package com.example.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.ResponseBody;

@Controller
@SpringBootApplication
public class DemoApplication {
	@RequestMapping("/")
	@ResponseBody
	public String home(){
		return "Hello world!";
	}
	public static void main(String[] args) {
		SpringApplication.run(DemoApplication.class, args);
	}
}

```
启动该入口类，并访问http://localhost:8080/
![image_1bhu3bhh618gq4iv2rbbo8q4k2q.png-10.8kB][7]
成功！

此外，我们也可以通过生成的单元测试类进行模拟http请求：
```java
package com.example.demo;

import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.http.MediaType;
import org.springframework.mock.web.MockServletContext;
import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
import org.springframework.test.context.web.WebAppConfiguration;
import org.springframework.test.web.servlet.MockMvc;
import org.springframework.test.web.servlet.request.MockMvcRequestBuilders;
import org.springframework.test.web.servlet.setup.MockMvcBuilders;
import static org.hamcrest.Matchers.equalTo;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.content;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.status;


@RunWith(SpringJUnit4ClassRunner.class)
@SpringBootTest(classes = MockServletContext.class)
@WebAppConfiguration
public class DemoApplicationTests {
	private MockMvc mvc;
	@Before
	public void setUp() throws Exception {
		mvc = MockMvcBuilders.standaloneSetup(new DemoApplication()).build();
	}
	@Test
	public void getHello() throws Exception {
		mvc.perform(MockMvcRequestBuilders.get("/").accept(MediaType.APPLICATION_JSON))
				.andExpect(status().isOk())
				.andExpect(content().string(equalTo("Hello World")));
	}
}

```
![image_1bhu3vnplmep4ru1r6r1sv3p9137.png-152.9kB][8]


  [1]: http://static.zybuluo.com/coldxiangyu/0vvmxr9e1gblrqbv47w1ee99/image_1bhu45e0gqkc125014gfi3m13n39.png
  [2]: http://static.zybuluo.com/coldxiangyu/4eg6h9vxpf8kiybi24o9e7c5/image_1bhu253ei1ueo1ib0kj9177i1u8lp.png
  [3]: http://static.zybuluo.com/coldxiangyu/zrqcolqz9avxv6se3d3vi2c4/image_1bhu26qcj1v4ugd22sa1gqr1li516.png
  [4]: http://static.zybuluo.com/coldxiangyu/c496a7qfk62zm5olq4kg7fgo/image_1bhu28ufdnjo1ei5i0g1k4cfj91j.png
  [5]: http://static.zybuluo.com/coldxiangyu/zr26q2k7ws3fmcassmk53zjv/image_1bhu2hpof142p1hir12pt1vpl14s220.png
  [6]: http://static.zybuluo.com/coldxiangyu/q1owgclyw0jkxwpafxtz05s8/image_1bhu2u63q1jmt1m6d1dbc1vjqb632d.png
  [7]: http://static.zybuluo.com/coldxiangyu/tq5csrz68gygkajm79vy6fdm/image_1bhu3bhh618gq4iv2rbbo8q4k2q.png
  [8]: http://static.zybuluo.com/coldxiangyu/ehnwogkjm5h1ednef7uu45h6/image_1bhu3vnplmep4ru1r6r1sv3p9137.png
