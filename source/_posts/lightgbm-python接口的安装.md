---
title: lightgbm python接口的安装
date: 2017-05-31 18:32:15
tags: [机器学习,数据挖掘]
categories: 机器学习
---
最近看一些大数据比赛大神们开源的解决方案，发现他们经常使用所谓数据挖掘三驾马车。恶补了一下boosting方法.现在把lightgbm的python接口的安装过程记录下来，免得以后忘了。
LightGBM（Light Gradient Boosting Machine）是一个基于决策树算法的快速的、分布式的、高性能 gradient boosting（GBDT、GBRT、GBM 或 MART）框架，可被用于排行、分类以及其他许多机器学习任务中。
开源项目地址： https://github.com/Microsoft/LightGBM
<!--more-->
优势：
> 更快的训练效率
> 低内存使用
> 更好的准确率
> 支持并行学习
> 可处理大规模数据

# 1.下载源码
```
git clone --recursive https://github.com/Microsoft/LightGBM
```
# 2.编译dll
进入下载的LightGBM目录下，用VS打开windows/LightGBM.sln，生成时选择DLL和x64，然后进行编译。dll文件就会在windows/x64/DLL/目录里。
![](http://i.imgur.com/lRW0CwU.png)
# 3.安装python包
进入目录python-package，执行命令 
```python setup.py install```
# 4.测试是否安装成功
进入examples\python-guide，执行样例 
```python .\simple_example.py``` 
如果没有报错，那就说明安装成功了！