---
title: github上ssh配置
date: 2017-03-31 04:13:14
tags: [hexo,github]
categories: hexo
---
# 配置 ssh key
使用 git bash 生成 public ssh key，以下是最简单的方法
<!--more-->
```
$ ssh-keygen -t rsa
C/Documents and Settings/username/.ssh 目录下会生成 id_rsa.pub
```
将 id_rsa.pub 的内容完全复制到 github Account Setting 里的 ssh key 里即可

# 测试
```
$ ssh -T git@github.com
```
然后会看到
```
Hi [yourGithubAccount]! You've successfully authenticated, but GitHub does not provide shell access.
```
设置用户信息
```
$ git config --global user.name "[yourName]"//用户名
$ git config --global user.email  "[yourEmail]"//填写自己的邮箱
```
经过以上步骤，本机已成功连接到 github，为部署打下基础。