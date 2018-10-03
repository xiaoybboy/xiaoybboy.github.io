---
title: maven项目创建时出错
date: 2017-07-22 15:56:06
tags: ## [maven,javaweb] ##
categories: 奇怪的问题
---
哎，一些问题每次出错都要重新找方法解决。以后还是都记录下来把。
## 问题描述
Eclipse创建Maven项目时，报错
Could not calculate build plan: Plugin org.apache.maven.plugins:maven-resources-plugin:2.6 or one of its dependencies could not be resolved: Failed to read artifact descriptor for org.apache.maven.plugins:maven-resources-plugin:jar:2.6
Plugin org.apache.maven.plugins:maven-resources-plugin:2.6 or one of its dependencies could not be resolved: Failed to read artifact descriptor for org.apache.maven.plugins:maven-resources-plugin:jar:2.6
<!--more-->
## 解决方案
删除本地Maven仓库org.apache.maven.plugins:maven-resources-plugin所在目录文件夹。然后在一个maven项目中的pom.xml文件中增加以下依赖
```xml
<dependency>
		<groupId>org.apache.maven.plugins</groupId>
		<artifactId>maven-resources-plugin</artifactId>
		<version>3.0.2</version>
</dependency>
```
然后执行maven install重新下载maven-resources-plugin这个jar包即可。
同样的，有可能其他plugin会报类似的错误，解决方法和上面类似。