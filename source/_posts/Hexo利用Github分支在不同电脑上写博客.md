---
title: Hexo利用Github分支在不同电脑上写博客
date: 2017-03-19 12:15:27
tags: [hexo]
categories: hexo
---

{% cq %} 利用github的不同分支来分别保存网站静态文件与hexo源码（md原始文件及主题等），实现在不同电脑上都可以自由写博客。 {% endcq %}
## Github Page
这里假设你已经在github上建好了page的仓库，也就是 “yourusername.github.io”的名字的项目仓库，比如我的 xiaoybboy.github.io 。另外，也假设你在自己的电脑上已经配置好git、hexo、node js等环境。
<!--more-->
## 新建hexo分支
仓库建好之后，都是默认一个master分支的，Github page要求你的网站文件必须存放在这个master分支上，这个没得选；所以我们需要新建另外一个分支来保存我们的hexo原始文件；master 保存的全是public 文件夹下的内容，这也是 hexo发布到github上的内容，而hexo这个分支，保存的是我们网站的配置 。
```
git branch hexo
git checkout hexo
git add remote origin git@github.com:xiaoybboy/xiaoybboy.github.io.git
git push -u origin hexo:hexo
```
## 设置默认分支
因为我们写博客更多的是更新这个分支，网站文件所在的master分支则由hexo d命令发布文章的时候进行推送，所以我们将hexo分支设置为默认分支，这样我们在新的电脑环境下git clone该仓库时，自动切到hexo`分支。按下图进行操作。
![设置默认分支](https://i.imgur.com/4BoTift.png)
## 配置hexo deploy参数
为了保证hexo d命令可以正确部署到master分支，在hexo 的配置文件 _config.yml文件中配置参数如下：
```
deploy:
  type: git
  repo: https://github.com/dxjia/xiaoybboy.github.io.git
  branch: master
```
hexo 3.0之后，将github改为了git，这样适用性更广了，如果你发现无法hexo d，使用下面的命令安装git deployer插件后重试即可。
```
npm install hexo-deployer-git --save
```
## 修改推送到hexo分支
上一步的deploy参数正确配置后，文章写完使用hexo g -d命令就可以直接部署了，生成的博客静态文件会自动部署到 username.github.io仓库的master分支上，这时候通过浏览器访问http://username.github.io就可以看到你的博客页面里。
网站页面是保存了，但这时候我们还没有保存我们的hexo原始文件，包括我们的文章md文件，我们千辛万苦修改的主题配置等。。。接下来使用下面的步骤将他们都统统推送到hexo分支上去。其中目录下的.gitignore 表示哪些文件或文件夹不提交，根据自己需要配置。
如果没有.git文件，先git init。
```
git add .
git commit -m “change description”
git push origin hexo
```
如果没关联远程仓库，执行第三步会出错。和远程仓库关联执行：git remote add origin <远程仓库地址>
这样就OK了，我们的原始文件就都上去了，换电脑也不怕了。
## 日常写博客
有时候我们可能会在不同的电脑上写博客，那在不同的电脑上配置 hexo、git、node.js，以及配置git ssh key等都要折腾一下的，这是免不了的，也是比wordpress等其他博客框架麻烦的一点。
## 已有环境
如果在电脑上已经写过博客，那么可以在已有的工作目录下同步之前写的博客。
在你的仓库目录下右键’git bash shell’，起来bash命令行，然后
```
git pull
```
这样你的状态就更新了，之后就是 hexo命令写文章啦。。。
写完hexo g -d部署好后，使用
```
git add .
git commit -m “change description”
git push origin hexo
```
推送到hexo分支上去。
## 新的环境
到了新的电脑上时，我们需要将项目先下载到本地，然后再进行hexo初始化。
**记住不需要hexo init指令**
```
git clone git@github.com:xiaoybboy/xiaoybboy.github.io.git
cd xiaoybboy.github.io
npm install hexo
npm install
npm install hexo-deployer-git –save
```
之后开始写博客，写好部署好之后，别忘记 git add , ….git push origin hexo…推上去。。。
原文地址：[http://dxjia.cn/2016/01/27/hexo-write-everywhere/](http://dxjia.cn/2016/01/27/hexo-write-everywhere/ "原文")