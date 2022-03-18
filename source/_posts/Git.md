---
title: Git
date: 2021.04.06
tags: Git
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 工欲善其事，必先利其器。Git是实现分布式版本控制的一个工具，所以Git完全可以在没有网络的情况下使用，每个人的开发机都可以作为一个独立的版本库存在，开发者可以在自己的开发机管理代码版本，而不依赖于远程版本库。
---

工欲善其事，必先利其器

Git是实现分布式版本控制的一个工具，所以Git完全可以在没有网络的情况下使用，每个人的开发机都可以作为一个独立的版本库存在，开发者可以在自己的开发机管理代码版本，而不依赖于远程版本库。

主要介绍 [Git](https://git-scm.com/book/en/v2/) 基本使用以及分支管理。

团队中使用Git 的基本流程：
- 克隆远程版本库
- 基于远程 develop分支 建立本地 develop分支
- 基于 develop分支建立本地 特性分支feature
- 在 feature分支 编写程序
- 切换到 develop分支，合并 feature 的修改
- 把本地 develop分支 的修改推到远程develop


## Git 命令基本使用：

![git](https://wyiyi.github.io/amber/contents/git/git.png)

 ```
# 从远程仓库克隆仓库代码到本地
$ git clone http://xxx.git
# 查看分支
$ git branch -a
# 创建分支
$ git checkout -b new-branch
# 切换分支
$ git checkout new-branch
# 查看分支状态
$ git status
# 本地提交代码
$ git add .
$ git commit -m "test commmit"
# push 之前本地先 pull 最新代码
$ git pull origin master
# 遇到冲突，查看哪些文件冲突
$ git diff
# 回退版本 aaaa
$ git reset --hard aaaa
# 将本地分支推送到其它分支
$ git push origin master
 ```

## Git 分支管理：

一个分支无法HOLD住复杂的开发发布部署流程。比如：
- 一个项目开发新功能时线上出现BUG了，需要紧急修复，你改如何处理手头的代码呢？
- 如果这个功能是几个人共同开发的，情况就更复杂了。
- 实际开发中有各种复杂情况。

基于以上问题，我们更需要管理好且利用好[Git分支管理模型](nvie.com/posts/a-succes)。

![git](https://wyiyi.github.io/amber/contents/git/branch.png)

一个项目的代码库至少要有 master 和 develop 这两个分支。团队成员从主分支 master 获得的都是处于可发布状态的代码，而从开发分支 develop应该总能够获得最新开发进展的代码。
- master：获得的代码一定要保证是和线上运行的程序是一致的
- develop：获得的应该是最新的稳定版本的代码

 ```
# 本地基于 develop分支 创建 myfeature 分支 
$ git checkout -b myfeature develop
# 在 myfeature上开发完代码之后，需要合并到 develop分支 上
$ git add .
$ git commit -m "commmit"
$ git push origin develop
 ```

## 可视化界面操作

常用的工具：IDEA，ACAP，VSCode，eclipse等工具。
1、IDEA 
- Local history 可以查看之前修改该文件的记录
- Commit File 正常提交文件
- Compare with.. 和远程比较
- Compare with Branch... 和远程分支进行比较
- Show History 对该文件的历史修改记录
- Rollback... 还原文件最初的状态
- pull 拉取远程分支到本地
- push 推送本地分支到远程分支

2、ACAP
![](https://wyiyi.github.io/amber/contents/git/acap.png)

![](https://wyiyi.github.io/amber/contents/git/acap1.png)

3、TortoiseGit 工具
![](https://wyiyi.github.io/amber/contents/git/tortoiseGit.png)

4、Gitlab 冲突处理

![](https://wyiyi.github.io/amber/contents/git/gitlab2.png)

![](https://wyiyi.github.io/amber/contents/git/gitlab1.png)



## 官方文档参考：
- [Git](https://git-scm.com/book/en/v2/)
- [Git 分支管理模型](nvie.com/posts/a-succes)


![](https://wyiyi.github.io/amber/contents/barcode/3426.png)
