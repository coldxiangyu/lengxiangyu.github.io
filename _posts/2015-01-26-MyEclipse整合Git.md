---
layout: post
title:  "MyEclipse整合Git"
date:   2015-01-26 12:25:34
categories: 技术储备
tags: git myeclipse
mathjax: true
---

* content
{:toc}

### 一、准备工作：
在搭建环境之前需要做以下准备工作：  
1）既然是在MyEclipse中整合Git那首先需要安装MyEclipse，安装步骤略去不表，木有什么可值得记录的，一路next基本上就可以搞定。（文中是以MyEclipse8.5为例）  
2）Egit插件，可以在官网上下载最新版本，网址为：http://www.eclipse.org/egit/download/ 。Egit是一款Eclipse上的Git插件。  
3）下载Git环境，可以在Git官网上下载最新版本，网址为：http://help.github.com/win-set-up- git/。（文中以Git-1.8.1.2-preview20130201为例）。  



4）在https://github.com/ 上用自己的邮箱申请一个帐号，不再详述。
          
### 二、环境搭建
    
在以上准备工作就绪后，就可以正式开始Git环境的搭建了。  
1）将下载的Egit插件解压出来，删掉XML文件。在MyEclipse的安装目录下找到dropins目录，在该目录下新建一个Egit文件夹，然后把刚才解压的features、pulgins和另外2个jar包全部放进去。重启MyEclipse后，选择Window-->preferences-->team，看到Git选项，就说明安装成功了。  
2)安装下载的Git安装文件，按照默认设置安装即可。  
3)使用刚才申请的帐号在https://github.com上创建一个新项目(点击new respositories)，填写项目名称、描述等相关信息即可。在此假如项目名称为：Helloworld。  
4)设置SSH Key信息，该操作非常重要，设置不正确，项目无法提交到Github上。  
在开始菜单中找到Git Bash打开。  
输入ssh-keygen -t rsa -C "youremailaddress@youremail.com"(youremailaddress@youremail.com为注册Github时使用的邮箱帐号)，回车。  
此时系统会有一些提示和确认信息，一路回车即可。  
最后在系统中会生成一个id_rsa.pub文件，该文件内容即为SSH Key。该文件默认位置为：C:\Users\Administrator\.ssh(以Win7为例)。  
登录github在Account setting中找到SSH Keys，点击页面右侧的Add SSH Key，将id_rsa.pub文件的内容全部复制进去保存即可(title写个自己方便区分的就可)。  
把C:\Users\Administrator\.ssh里的内容全部复制到C:\Users\Administrator\ssh。  
至此，SSH Key设置已完成。  
5)在MyEclipse中新建一个项目，如：D:\git\Helloworld。  
6)按照下面步骤创建该项目的README文件，并上传至Github。

打开Git Bash。
```
git config --global user.name "用户名"  
git config --global user.email "注册时使用的email"  
cd d://git//Helloworld  注：项目在本地目录  
touch README.md  
git init  
git add README.md  
git commit -m "first commit"  
git remote add origin git@github.com:注册帐号@之前部分/Helloworld.git  
git push -u origin master  
```

以上步骤完成后，应该就可以在Github上看到刚才上传的README.md文件了。  
7)以上步骤都搞定之后就可以进入MyEclipse，使用Egit插件对项目进行管理了。  
还有一个需要注意的地方是MyEclipse的默认联网方式可能不对，需要修改。  
window-preferences-General-Network connections，把Active Provider设置为Direct（默认为 Native）。  
8)右键工程名，Team-->share project，后面Egit插件在MyEclipse中的使用，详见http://www.eoeandroid.com/thread-273360-1-1.html ，里面有详细的图文说明。




