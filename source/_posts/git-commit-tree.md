---
title: Git Commit Tree Browser
date: 2024.04.01
tags: 
    - Git
    - Tools
categories: Technology  
mathjax: true
comments: true
toc: true
description: 安利一个小工具：Git 仓库 Commit 文件树查看器。
---

![](https://wyiyi.github.io/amber/contents/2024/git.gif)

## 背景
在`Git`项目中，代码的频繁迭代会使得追踪特定变更的`commit`变得复杂。为了解决这个问题，`Git Commit Tree Browser` 提供了一个高效的解决方案。

## 简介
`Git Commit Tree Browser`是基于`Bash`的脚本工具，用于简化`Git`仓库中追踪文件变化的过程。
它通过为每个commit 生成文件目录树，并配备交互式的浏览界面，让开发者能够轻松地定位和查看特定 commit 下的文件状态。

## 功能特点
* 为指定`Git`仓库中每个`commit`生成文件目录树
* 提供交互式界面，通过左右箭头键切换查看不同`commit`的文件目录树
* 支持用户输入文件名，快速跳转到特定的`commit`文件目录树

## 使用方法

**安装前提**：
- 确保系统已安装`Git Bash`
- 安装`tree`命令以生成目录树结构，`Windows`用户可以在`Git Bash`中安装 [tree 工具](https://gnuwin32.sourceforge.net/packages/tree.htm)

从[GitHub仓库](https://github.com/AlphaHinex/git-commit-tree-browser)下载到本地。

### 执行 prepare.sh 脚本：

运行`prepare.sh`脚本，传入`Git`仓库的路径作为参数：

```bash
$ ./prepare.sh /path/to/your/git/repo
```

脚本将遍历指定 Git 仓库的所有`commit`，并为每个`commit`生成文件目录树，结果存储为文本文件（`.txt`文件）。

### 执行 browser.sh 脚本

运行`browser.sh`脚本，开始浏览`commit`文件目录树：

```bash
$ ./browser.sh
```

脚本将显示第一个`commit`的文件目录树（示例）：

```
File: da38390.txt
.
|-- project
|   |-- demo1
|   |   `-- scripts
|   `-- demo2
|       `-- src
`-- git-commit-tree-browser

6 directories
da38390
Press left/right arrow to navigate, or Enter to jump to a file:
```

可以使用以下方式导航：

* 使用左右箭头键在不同的`commit`之间切换
* 按`Enter`键，然后输入文件名，直接跳转到特定的`commit`

**小工具会继续丰富功能，请期待~~~**
