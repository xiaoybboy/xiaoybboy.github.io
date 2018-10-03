---
title: eclipse下创建maven webapp项目
date: 2017-04-22 16:17:03
tags: [maven,javaweb]
categories: javaweb
---
1、开启eclipse，右键new——》other，如下图找到maven project
<!--more-->
![](http://i.imgur.com/tLozsmT.jpg)
2、选择maven project，显示创建maven项目的窗口，勾选如图所示，Create a simple project
![](http://i.imgur.com/8RBxu5E.jpg)
3、输入maven项目的基本信息，如下图所示：
![](http://i.imgur.com/m69rSeD.jpg)
4、完成maven项目的创建，生成相应的maven项目结果，如下所示，此处有部分结构是项目不需要的，我们需要去掉：
![](http://i.imgur.com/MzkoXcz.jpg)
5、选择项目，右键选择Properties，进入属性页面，选择到Maven菜单下，如下图所示：
![](http://i.imgur.com/wMgDBuy.jpg)
6、选择java版本为1.7，并去掉其他两项，如下图：
![](http://i.imgur.com/SXeVRQJ.jpg)
7、点击ok之后，再次回到项目结构，此时项目结构比较清晰，符合我们想要创建的maven项目
![](http://i.imgur.com/335DCN2.jpg)
8、此时webapp下的结果还没有显示出来，因为此时我们还没有配置此的项目为web项目，再次进去Properties配置，如下图所示：
![](http://i.imgur.com/rNzAT8S.jpg)
9、点击Further configuration available...，如下：
![](http://i.imgur.com/ZD9GThf.jpg)
10、配置src/main/webapp，并勾选生成web.xml的选项，如下：
![](http://i.imgur.com/WRO3v9x.jpg)
11、确定之后，返回到maven菜单下去掉Dynamic Web Module的勾选，点击ok，如下所示，webapp目录结构显示出来了：
![](http://i.imgur.com/XMUM5u7.jpg)
12、此时还需要配置，src/main/webapp为“/”项目的根目录，如下所示：
![](http://i.imgur.com/VfwJFIP.jpg)
13、完成如上配置后，最后完成maven webapp项目结构如下图所示：
![](http://i.imgur.com/ZNSDos7.jpg)
