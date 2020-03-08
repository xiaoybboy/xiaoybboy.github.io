---
title: springboot学习笔记
date: 2018-10-29 22:20:07
tags: [springboot]
categories: spring
---
{% note success %} 一门技术如果不用，真是超快就忘记了。springboot都学几遍了，一两个星期不看，就忘得一干二净，索性自己好好总结一下，供以后经常翻翻，顺便熟悉一下idea。有很多其他大牛博客的东西，多找些资料总结一下。 {% endnote %}
{% note danger %} 万事开头难，加油！{% endnote %}
## Spring Boot 配置文件
SpringBoot使用一个全局的配置文件（resource文件夹下），配置文件名是固定的。
-application.properties
-application.yml
springboot推荐使用yml的方式进行配置（yml的确很方便）。
配置文件的作用：修改SpringBoot自动配置的默认值；
### 自定义属性与使用
虽然springboot帮我们配置了很多东西，但是有时候我们需要定义自己的配置，我们可以在application.yml中如下方式定义：
```yml
person:
  name: xyb
  age: 25
  address: hns
```
然后就可以通过@Value("${属性名}")注解来加载对应的配置属性，具体如下：





