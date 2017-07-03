---
layout: post
title:  "Git连接github"
date:   2016-02-03 15:26:12
categories: 技术储备
tags: git github
mathjax: true
---

首先安装git，windows版本下载地址：http://msysgit.github.io/  
安装过程不再详述。  
界面如下，类似shell工具：  
![image_1bh2a7a1p1krm15kt9ghalun0sm.png-9.4kB][1]   




在windows机器上安装完gitbash之后，如何与github进行连接呢？  
1.创建身份标识（用户名、邮箱）
```
git config --global user.name coldxiangyu #用户名
git config --global user.email coldxiangyu@qq.com #邮箱
```  
2.创建git仓库，我本地选取默认目录创建git文件夹作为仓库
```
coldxiangyu@DESKTOP-7TT78KK MINGW64 ~
$ mkdir git
coldxiangyu@DESKTOP-7TT78KK MINGW64 ~
$ cd git

coldxiangyu@DESKTOP-7TT78KK MINGW64 ~/git
$ git init
Initialized empty Git repository in C:/Users/coldxiangyu/git/.git/

coldxiangyu@DESKTOP-7TT78KK MINGW64 ~/git (master)
$
```
3.在git里生成公钥文件，用来连接github。在git命令控制台输入下面命令，连续敲3个回车即可
```
ssh-keygen -t rsa -C "coldxiangyu@qq.com"
```
![image_1bh2aoftb1b0g10v0o04vg0fja13.png-30.7kB][2]
  
生成秘钥文件：

![image_1bh2at3o01mus7gt6nj3ljiik1g.png-20.5kB][3]

4.在github账户设置中设置ssh keys，见下图，其中title自己取名，key的内容就是将id_rsa.pub中的代码全部复制过来

![image_1bh2b2mki1fv41k24143f1qbb1v131t.png-70.5kB][4]

5.在git终端上测试链接github
```
ssh git@github.com
```
![image_1bh2bsqcmgb61c51jp41mnrea49.png-9.2kB][5]  
OK!这时候就完成了git与github的连接了。


  [1]: http://static.zybuluo.com/coldxiangyu/xfj6zfmeuzinbq9n3icym4a4/image_1bh2a7a1p1krm15kt9ghalun0sm.png
  [2]: http://static.zybuluo.com/coldxiangyu/t6yaziicfkivs8rdlzvnnham/image_1bh2aoftb1b0g10v0o04vg0fja13.png
  [3]: http://static.zybuluo.com/coldxiangyu/busdz0dfytdkexw4jakwven7/image_1bh2at3o01mus7gt6nj3ljiik1g.png
  [4]: http://static.zybuluo.com/coldxiangyu/s374hbthtgw0vlk4a790vh1g/image_1bh2b2mki1fv41k24143f1qbb1v131t.png
  [5]: http://static.zybuluo.com/coldxiangyu/32d6pevgr6uqdz2gi309m0on/image_1bh2bsqcmgb61c51jp41mnrea49.png
