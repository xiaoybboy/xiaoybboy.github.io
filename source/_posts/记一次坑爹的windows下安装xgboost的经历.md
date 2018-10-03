---
title: 记一次坑爹的windows下安装xgboost的经历
date: 2017-06-05 15:51:28
tags: [xgboost]
categories: 大数据比赛
---
近来做一个比赛，一个识别鼠标轨迹是人还是机器的二分类问题。自己用SVM和神经网络等传统的机器学习的方法都试过了，但是提交之后的结果发现并不好。正好前几天研究过几个别人做的其他比赛的解决方案，发现用的boosting方法比较多,其中尤其是xgboost。于是打算再用xgboost的模型做一下，看看结果怎么样。
但是，安装xgboost的过程并不轻松。试了好几种方法，最后花了一下午时间，终于安装成功。
<!--more-->
# 1.使用纯命令行安装
参考：http://www.th7.cn/system/win/201603/157092.shtml
http://blog.csdn.net/ychanmy/article/details/50972530
```
$ git clone --recursive https://github.com/dmlc/xgboost$ cd xgboost
$ git submodule init
$ git submodule update
$cp make/mingw64.mk config.mk
$make -j4

$cd python-package
$python setup.py install
```
但是，为毛别人都能成功，我就是一直报错呢，而且google不到答案，哎。。。
![](http://i.imgur.com/wMngVUt.png)
看来这个东西看人品的，没办法，找其他的办法吧！
# 2.使用旧版本的xgboost
最终，在http://m.blog.csdn.net/article/details?id=53118803找到一个可用的解决方法。
下载旧版本的xgboost,http://download.csdn.net/detail/zhuqiuhui/9476012
然后就很简单。
```
$cd xgboost
$make -j4

$cd python-package
$python setup.py install
```
![](http://i.imgur.com/9VfyKQV.png)
终于！不知道旧版本的xgbosot有没有什么问题，先用用试试吧！