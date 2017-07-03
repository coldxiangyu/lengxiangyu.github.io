---
layout: post
title:  "redis深入研究【list】"
date:   2016-04-05 15:29:41
categories: redis
tags: redis
mathjax: true
---

redis命令参考手册：http://doc.redisfans.com/  
可以看到redis提供的命令有很多，比较吸引人的大概就是redis关于list的应用了，因为可以作为队列使用，也比较适合银保通，还可以省去专门的MQ服务器，比如RabbitMQ、ActiveMQ等。  
查看redis命令手册可以看到，redis关于List提供以下方法：  
`BLPOP、BRPOP、BRPOPLPUSH、LINDEX、LINSERT、LLEN、LPOP、LPUSH、LPUSHX、`
`LRANGE、LREM、LSET、LTRIM、RPOP、RPOPLPUSH、RPUSH、RPUSHX`等。  




其中作为队列使用，常用的无非就是`LPUSH、LPOP、RPUSH、RPOP`，单从命名大概可以看出，这个list作为队列可以左PUSH、POP，也可以右PUSH、POP，以下为本地redis客户端测试：
```
127.0.0.1:6379> lpush nums 1 2 3 4 5 6
(integer) 6
127.0.0.1:6379> lpop nums
"6"
127.0.0.1:6379> rpop nums
"1"
127.0.0.1:6379> lpop nums
"5"
127.0.0.1:6379> rpop nums
"2"
127.0.0.1:6379> lpop nums
"4"
127.0.0.1:6379> rpop nums
"3"
127.0.0.1:6379> lpop nums
(nil)
127.0.0.1:6379>
```
从数据结构方面分析：
```
lpush nums 1 2 3 4 5 6
lpop nums
"6"
rpop nums
"1"
```

|lpop|||||rpop|
|--|--|
| 6 | 5 | 4 | 3 | 2 | 1 |

关于redis的C实现我偷了个懒，网上扒了个图下来：

![此处输入图片的描述][1]  

  [1]: http://static.zybuluo.com/coldxiangyu/qtjljd9sjlhimjuhwz2dovhl/image_1bjpfhju4jn112qspd82nl1jpt9.png

> * ***listNode***  　很明显这是一个node节点，可以看出它有一个prev指针和一个next指针，分别指向节点的前驱和后继，然后还有一个void* 这个类型的value

> * ***list***  　这个list蛮有意思的一点就是，里面有一个head和tail节点，可想而知，tail存放的是list的尾节点，有了这个节点就说明什么呢？说明你删除尾节点的复杂度是O(1),同样有了这个head，你删除头节点同样也是O(1)。这就有了刚才说的LPush,LPop,RPush,RPop，是的吧，同时list里面还有一个len属性，是记录当前list的元素个数，这样的话，你统计list的个数也是O(1)的，这个效率就不用我说了。
