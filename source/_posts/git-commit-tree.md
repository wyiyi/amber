---
title: Git 仓库 Commit 文件树查看器（Git Commit Tree Viewer）
date: 2024.04.01
tags: Git
categories: Technology  
mathjax: true
comments: true
toc: true
description: 安利一个小工具
---

## 简介
`Git Commit Tree Viewer` 是一个实用的 `Bash` 脚本工具，用于简化 `Git` 仓库中文件变化的查看过程。
当仓库中的文件经历大的版本变化时，小工具能帮助你快速定位并查看特定 `commit` 时的文件目录树，使追踪文件变更变得轻松。

## 功能特点
* 为指定 `Git` 仓库中每个 `commit` 生成文件目录树
* 提供交互式界面，通过左右箭头键切换查看不同 `commit` 的文件目录树
* 支持用户输入文件名，快速跳转到特定的 `commit` 文件目录树

## 使用方法

**安装前提**：确保你的系统已安装 `Git`、`Tree`（用于生成目录树结构） 命令。

从[GitHub 仓库](https://github.com/AlphaHinex/git-commit-tree-browser)下载到本地。

### 执行 prepare.sh 脚本：

运行 `prepare.sh` 脚本，传入 `Git` 仓库的路径作为参数：

```bash
$ ./prepare.sh /path/to/your/git/repo
```

脚本将遍历指定Git仓库的所有 `commit`，并为每个 `commit` 生成文件目录树，结果存储为文本文件（`.txt`文件）。

### 执行 browser.sh 脚本

运行 `browser.sh` 脚本，开始浏览 `commit` 文件目录树：

```bash
$ ./browser.sh
```

脚本将显示第一个 `commit` 的文件目录树：

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

* 使用左右箭头键在不同的 `commit` 之间切换。
* 按 `Enter` 键，然后输入文件名，直接跳转到特定的 `commit`。

**小工具会继续丰富功能，请期待~~~**
