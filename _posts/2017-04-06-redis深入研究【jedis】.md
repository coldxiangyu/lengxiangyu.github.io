---
layout: post
title:  "redis深入研究【jedis】"
date:   2016-04-06 13:47:53
categories: redis
tags: redis jedis
mathjax: true
---

Jedis是Redis官方首选的Java客户端开发包，有了它我们就可以愉快的用java操作redis了。
接下来搭建一个测试jedis的工程，如下：  

![image_1bcumo8b51chl1405s9d5nl1uj89.png-16.8kB][1]




需要引入包，jedis-2.9.0.jar以及commons-pool2-2.3.jar
首先编写Junit测试类，测试redis连通性。
代码如下：
```java
package com.lxy.test;

import org.junit.Test;

import redis.clients.jedis.Jedis;

public class JedisTest {
    //书写测试类
    @Test
    public void TestJedis(){
        //创建jedis对象，相当于创建了一个客户端和reidis服务器的链接。需要ip和端口号，IP就是安装有redis服务的linux服务器的地址，端口号默认为6379
        Jedis jedis= new Jedis("127.0.0.1",6379);
        //ping redis服务器，这是它的一个方法，如果服务正常，会回复一个pong
        String pong = jedis.ping();
        System.out.println("如果服务器可用请返回pong，谢谢合作："+pong);
        //先用第一个键去取一次值，这个时候redis中没有数据返回应该是空的。
        String value = jedis.get("key");
        System.out.println("第一次访问的时候取到的值="+value);
        //将数据存入reidis服务器中
        jedis.set("key","第一次存入的值");
        //将数据取出
        value = jedis.get("key");
        System.out.println("存入相应的值以后取到的值="+value);
        //关闭和redis的链接
        jedis.close();
    }
    
}
```
结果如下：  

![image_1bcumu7tef9ui7lq0uir41mcsm.png-10.4kB][2]


对应服务器响应：  

![image_1bcun0pbed3ck9890ajl014l913.png-8.1kB][3]


在实际应用中，这样是远远不够的，不能每次进行redis数据操作就单独建立连接。jedis提供了连接池的API，JedisPool。
以下为jedis连接池的测试类，写的比较简单：
```java
package com.lxy.pool;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

public class RedisUtils {
    
    private RedisUtils(){
    
    }
    
    private static  JedisPool jedisPool = null;
    //获取链接
    public static synchronized Jedis getJedis(){
        if(jedisPool==null){
            JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
            //指定连接池中最大空闲连接数
            jedisPoolConfig.setMaxIdle(10);
            //链接池中创建的最大连接数
            jedisPoolConfig.setMaxTotal(100);
            //设置创建链接的超时时间
            jedisPoolConfig.setMaxWaitMillis(2000);
            //表示连接池在创建链接的时候会先测试一下链接是否可用，这样可以保证连接池中的链接都可用的。
            jedisPoolConfig.setTestOnBorrow(true);
            jedisPool = new JedisPool(jedisPoolConfig, "127.0.0.1", 6379);
        }
        return jedisPool.getResource();
    }
    
    //返回链接
    public static void returnResource(Jedis jedis){
        jedisPool.close();
    }

}
```
之后，我把开始的测试类改成获取连接池连接进行redis操作：
```java
@Test
    public void TestJedisPool(){
    	Jedis jedis = RedisUtils.getJedis();
    	System.out.println("测试连通性："+jedis.ping());
    	String value = jedis.get("key");
    	System.out.println("key="+value);
    	
    }
```
执行结果：  

![微信截图_20170406091758.png-9.4kB][4]

关于jedis的基本操作先介绍到这里，后续在项目中的具体应用及优化再进行补充说明。


  [1]: http://static.zybuluo.com/coldxiangyu/zhmdcwnc3lthm0br6chxyq5j/image_1bcumo8b51chl1405s9d5nl1uj89.png
  [2]: http://static.zybuluo.com/coldxiangyu/akmm8febtfw0b8slt2o4m0ey/image_1bcumu7tef9ui7lq0uir41mcsm.png
  [3]: http://static.zybuluo.com/coldxiangyu/3nabdpwqp8xkw4w8bkzso31b/image_1bcun0pbed3ck9890ajl014l913.png
  [4]: http://static.zybuluo.com/coldxiangyu/4g237yq29vuj8561r30zzpt6/%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20170406091758.png
