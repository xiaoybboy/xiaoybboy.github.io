---
title: Java流式编程Stream
date: 2021-02-28 20:32:35
tags: [Java,Stream]
---
# 什么是Stream

Stream是JDK8中新增的API,对集合Collection的功能进行增强，可以方便地对集合中的元素进行操作。Stream流其实是一个集合元素的函数模型，它并不是集合，也不是数据结构，其实本身并不存储任何元素Stream是一个来自数据源的元素队列。简而言之，Stream就是提供了一种高效且易于使用的处理数据的方式。特点：

- Stream自己不会存储元素;
- Stream操作不会改变源对象;
- Stream操作延迟执行，到终端操作时才会执行；

# 创建流



​	