---
title: SpringMVC使用@ResponseBody返回json 中文乱码
date: 2017-08-10 19:18:01
tags: [springmvc,json]
categories: spring
---
Spring中解析字符串的转换器默认编码居然是ISO-8859-1,如何处理utf-8中文字符
下面这都两个可以
<!--more-->
方法一，使用（produces = "application/json; charset=utf-8"）：
```java
@RequestMapping(value="/getUsersByPage",produces = "application/json; charset=utf-8")
@ResponseBody
public String getUsersByPage(String page,String rows,String text,HttpServletRequest request,HttpServletResponse response){
```
方法二，在spring-mvc.xml中添加：
```xml
<!-- 处理请求返回json字符串的中文乱码问题 -->
    <mvc:annotation-driven>
        <mvc:message-converters>
            <bean class="org.springframework.http.converter.StringHttpMessageConverter">
                <property name="supportedMediaTypes">
                    <list>
                        <value>application/json;charset=UTF-8</value>
                    </list>
                </property>
            </bean>
        </mvc:message-converters>
    </mvc:annotation-driven>
```
以上两种方式经过验证都没有问题。