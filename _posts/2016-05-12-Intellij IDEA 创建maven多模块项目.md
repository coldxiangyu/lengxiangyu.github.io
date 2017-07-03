---
layout: post
title:  "Intellij IDEA 创建maven多模块项目"
date:   2016-05-12 14:16:12
categories: 技术储备
tags: IDEA maven
mathjax: true
---

在研究微服务项目的过程中，对项目结构有了新的认识。也是由Maven引入了模块（Module）的概念。  
首先了解一下Maven多模块的概念，就是在一个项目下可以有多个模块。

- `Parent Project`用户组织不同的Module，不实现逻辑。
- `Parent project`和各个`Module`拥有独立pom文件。
- `Module`集成`Parent project`的`groupId`和`version`，`Module`只需要指定自己的`artifactId`即可。
- idea的project的模块都是配置在`project_dir/.idea/modules.xml`中的，新建的项目本身默认是这个项目的第一个模块。模块本身是没有层级的。在项目结构中可以配置模块间的依赖关系。
  
  



  
下面我们来展示一个多模块的项目的创建过程。

1.创建Maven项目，用于组织Module。
![image_1bhtg52ds1ll81bfc1h738eh1ovp9.png-82.5kB][1]

![image_1bhtga54tr9au3l1nid1ul616n9m.png-22.6kB][2]

![image_1bhtgascn8ad1a231rj08fo1ah413.png-31.6kB][3]

2.这样我们就创建好了一个普通项目，因为该项目是作为一个Parent project存在的，可以直接删除src文件夹。

![image_1bhtgjj1k164njkh14i81pnq2go1t.png-42.1kB][4]

3.创建子模块service-1

![image_1bhth1er6v31spv1kh1epqk5q2n.png-36.5kB][5]

![image_1bhth2hnhdf0197o1ugq62lvg34.png-90kB][6]

![image_1bhth4fht9v1v5akuhrqh4et3h.png-27.1kB][7]

4.按service-1的方式，我们再来创建模块service-2，完整目录结构如下：

![image_1bhthfkfpu16kts1ii31621qcj3u.png-62.7kB][8]

5.接下来，你或许会问，不同的module之间如何进行依赖呢？我们来添加一下service1对service2的依赖：

![image_1bhthlg8h1ocb4rge1dtlf1q554b.png-46.8kB][9]

到这里该介绍的已经介绍完了。只是做个多模块的样子也没什么意思，我们来具体实现一下功能。
现在我们已经让service1依赖service2了，接下来在service2中编写serivce方法HelloService()：

![image_1bhtl147818ooadgdjuc0arqa9.png-46.1kB][10]

然后下面就是由service1调用刚刚编写的方法了。  
service1怎么省事我们怎么来，我不想部署tomcat，直接用spring boot。在service1中直接添加spring boot的依赖？  
NONONO，我们看一下service1的pom就会发现，service1只是一个子模块，整体的pom是在muti-modules下的。OK，那我们得在muti-modules下pom中添加公共spring boot依赖：

![image_1bhtlcl95bm5bjjt271ctmokum.png-76.1kB][11]
这时候你有没有意识到pom这个parent节点的意义，我们创建子模块的时候，parent节点依赖于我们创建的父项目，而我们在介绍spring boot的时候，也是通过parent引入的，你肯定懂了这两种的关系。  
而我们现在只需要在service1中启动spring boot，service2只提供调用方法，不想在pom整体的引入`spring-boot-starter-web`，这时候我们就可以分别对service1和service2的pom独立配置了，我们只需配置service1，加入`spring-boot-starter-web`的依赖，此外还要加入service2的依赖。

![image_1bhtlvc6smqr14shret4ft1ng13.png-72.6kB][12]

编写spring boot启动类SampleController并启动：

![image_1bhtm3vdf1mtt17fq1g2d87p1f2j1t.png-169.9kB][13]

访问http://localhost:8080/，如图：

![image_1bhtm5rejnta1qt814ne1fc610je2a.png-11.8kB][14]

调用成功！
  
当然，Myeclipse也可以创建Maven多模块，不过IDEA更为方便，这也是我一直推荐IDEA的原因。
此外，如果项目之间不需要依赖，每个模块都是互相独立的个体的话，Parent Project也可以创建为普通项目，也无需一个总体POM进行整合，大家视情况而定即可。  
本文代码已上传github：https://github.com/coldxiangyu/mutimodules


  [1]: http://static.zybuluo.com/coldxiangyu/6e5ay4yd9855xl90kujaavny/image_1bhtg52ds1ll81bfc1h738eh1ovp9.png
  [2]: http://static.zybuluo.com/coldxiangyu/ehupdv9yp3yjobaktm0l348p/image_1bhtga54tr9au3l1nid1ul616n9m.png
  [3]: http://static.zybuluo.com/coldxiangyu/g551hokrwhb5whpoez3dchm7/image_1bhtgascn8ad1a231rj08fo1ah413.png
  [4]: http://static.zybuluo.com/coldxiangyu/xz6qfnezl1fiz7ofqy2m0o6v/image_1bhtgjj1k164njkh14i81pnq2go1t.png
  [5]: http://static.zybuluo.com/coldxiangyu/iw8dqw81u9hsky93t65bw0bm/image_1bhth1er6v31spv1kh1epqk5q2n.png
  [6]: http://static.zybuluo.com/coldxiangyu/16ptfwvtfbr9dgdic0zq85ve/image_1bhth2hnhdf0197o1ugq62lvg34.png
  [7]: http://static.zybuluo.com/coldxiangyu/n3yf2n69hvb47n5tvs1k8gxl/image_1bhth4fht9v1v5akuhrqh4et3h.png
  [8]: http://static.zybuluo.com/coldxiangyu/kn4xmxenvb02c42qux5vyyho/image_1bhthfkfpu16kts1ii31621qcj3u.png
  [9]: http://static.zybuluo.com/coldxiangyu/ezws8951uwcvm2ozkvhytr8u/image_1bhthlg8h1ocb4rge1dtlf1q554b.png
  [10]: http://static.zybuluo.com/coldxiangyu/ezc0nft4xem7a3wo9bub9fpu/image_1bhtl147818ooadgdjuc0arqa9.png
  [11]: http://static.zybuluo.com/coldxiangyu/23nlpe7epkb48hn7ns0dsgz5/image_1bhtlcl95bm5bjjt271ctmokum.png
  [12]: http://static.zybuluo.com/coldxiangyu/tppit4xg5myt6z7c3ywz67wr/image_1bhtlvc6smqr14shret4ft1ng13.png
  [13]: http://static.zybuluo.com/coldxiangyu/hke9qeb9omclbuc9a5ydlaar/image_1bhtm3vdf1mtt17fq1g2d87p1f2j1t.png
  [14]: http://static.zybuluo.com/coldxiangyu/a96b67ycdzxt83qyhmdbj1or/image_1bhtm5rejnta1qt814ne1fc610je2a.png
