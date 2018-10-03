---
title: hexo个人域名配置
date: 2017-05-31 17:20:02
tags: [hexo,域名]
categories: hexo
---
# 1.在万网上购买域名
https://wanwang.aliyun.com  或者其他域名购买网站
# 2.配置域名解析
进入你的管理后台
<!--more-->
![管理后台](http://i.imgur.com/8TZJ4q9.png)
# 3.添加域名解析
按下图所示添加域名解析。
![添加域名解析](http://i.imgur.com/sRLsOBt.png)
其中，记录类型选A或CNAME，A记录的记录值就是ip地址，github(官方文档)提供了两个IP地址，192.30.252.153和192.30.252.154，这两个IP地址为github的服务器地址，两个都要填上，解析记录设置两个www和@，线路就默认就行了，CNAME记录值填你的github博客网址。
# 4.创建CNAME文件
 这些全部设置完成后，此时你并不能要申请的域名访问你的博客。接着你需要做的是在hexo根目录的source文件夹里创建CNAME文件，不带任何后缀，里面添加你的域名信息，如：xiaoyb.me。
# 5.发布并访问
重新发布hexo d -g然后就可以访问你的新域名了。
快点试试吧！