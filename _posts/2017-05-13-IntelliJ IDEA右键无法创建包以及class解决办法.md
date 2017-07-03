---
layout: post
title:  "IntelliJ IDEA右键无法创建包以及class解决办法"
date:   2016-05-13 10:16:32
categories: 技术储备
tags: IDEA
mathjax: true
---

在使用IDEA的过程中，由eclipse转IDEA总会遇到一些不适应。
比如在新建maven项目中，想创建包以及class的时候，发现右键没有相关的创建项。如图：
![image_1bhtjjils1md710gbp89t51ilh9.png-74.7kB][1]




因为IDEA并不认同该目录为source目录。
解决办法如下：
![image_1bhtjlrqu1v67s1sbe013ee15c7m.png-59.3kB][2]
再次右键创建：
![image_1bhtjpfpror63i41l4h1qbr713.png-65.9kB][3]

还有更简单的办法：直接选中需要设定的目录右键，标记目录为你需要的选项即可。

![image_1bi0iohp68961njundg17mm1ndq9.png-81kB][4]

参考：http://blog.csdn.net/qq_27093465/article/details/52912444


  [1]: http://static.zybuluo.com/coldxiangyu/7onnnpec09ouvm33jdpvota0/image_1bhtjjils1md710gbp89t51ilh9.png
  [2]: http://static.zybuluo.com/coldxiangyu/nss9g5mljtnvp2kt3by00s4x/image_1bhtjlrqu1v67s1sbe013ee15c7m.png
  [3]: http://static.zybuluo.com/coldxiangyu/ef5e700ch56tbotmu18zribc/image_1bhtjpfpror63i41l4h1qbr713.png
  [4]: http://static.zybuluo.com/coldxiangyu/stpjzl323j7a99de0g06ijqx/image_1bi0iohp68961njundg17mm1ndq9.png
