---
title: 查看依赖树的利器
date: 2023.10.01
tags:
    - Maven
    - Gradle
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 
---
依赖树（Dependency Tree）工具，可以显示项目的依赖关系图，用于查看和分析项目的依赖关系。

本文将介绍 maven 和 gradle 两种构建如何查看依赖树。

## 使用命令行查看
1. 命令行工具来查看项目的依赖树，执行命令：
    
    ```
    mvn dependency:tree
    ```
   
    ```
    gradle dependencies
    ```

2. 对于大型项目，可以将输出结果保存到文件中：

    ```
    mvn dependency:tree > dependency_tree.txt
    ```
    ```
    gradle dependencies > dependencies.txt
    ```

## 使用 IDEA 查看 Show dependencies

1. 按下图操作（快捷键（ctrl+alt+shift+u），点击 show dependencies 显示依赖树结果（图三）
  ![图一_maven](https://wyiyi.github.io/amber/contents/tree/idea_1.png)
  ![图一_gradle](https://wyiyi.github.io/amber/contents/tree/gradle_1.png)

2. 打开文件（pom.xml 或者 .gradle）右键，如图点击 show dependencies 后显示依赖树结果（图三）
  ![图二_maven](https://wyiyi.github.io/amber/contents/tree/idea_2.png)
  ![图二_gradle](https://wyiyi.github.io/amber/contents/tree/gradle_2.png)

使用快捷键 ctrl + f 可以搜索要查找的 Jar 定位到依赖关系。（注意：IDEA 版本）
![图三](https://wyiyi.github.io/amber/contents/tree/idea_3.png)
![图三](https://wyiyi.github.io/amber/contents/tree/gradle_3.png)

## 总结
依赖树功能是项目依赖管理的重要工具，能够帮助我们查看和分析项目的依赖关系，解决依赖冲突问题，并进行优化和调整。
