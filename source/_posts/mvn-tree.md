---
title: 查看 Maven 依赖树的利器
date: 2023.10.01
tags: Maven
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 
---
Maven 是 Java 项目中常用的构建工具，通过声明依赖项来管理项目中使用的库和框架。
随着项目规模的增长，依赖项的数量也会迅速增加，导致 pom.xml 文件很难手动追踪和管理。

Maven 提供了一个强大的工具，即 “依赖树” ，用于查看和分析项目的依赖关系。
本文将介绍如何查看 Maven 的依赖树功能，帮助开发者理清项目依赖关系、解决依赖冲突问题，并优化项目构建。

## 使用命令行查看
1. Maven 提供了命令行工具来查看项目的依赖树，执行命令：
    
    ```
    mvn dependency:tree
    ```

2. 对于大型项目，可以将输出结果保存到文件中：

    ```
    mvn dependency:tree > dependency_tree.txt
    ```

## 使用 IDEA 查看

### Show dependencies 

1. 按照图一操作（快捷键（ctrl+alt+shift+u），点击 show dependencies 后，如图三
  ![图一](https://wyiyi.github.io/amber/contents/mvnTree/idea_1.png)

2. 打开 pom.xml 文件，右键，如图二，点击 show dependencies 后，如图三
  ![图二](https://wyiyi.github.io/amber/contents/mvnTree/idea_2.png)

使用快捷键 ctrl + f 可以搜索要查找的 Jar 定位到依赖关系。（注意：IDEA 版本）
![图三](https://wyiyi.github.io/amber/contents/mvnTree/idea_3.png)

### Maven Helper 插件
在 IDEA 中下载 Maven Helper 插件，打开 pom.xml 文件，底部的 tab 页，如图四：

如果要排除依赖，直接在对应的版本上点击右键，Exclude 即可。

![图四](https://wyiyi.github.io/amber/contents/mvnTree/idea_4.png)

## 总结
Maven 的依赖树功能是项目依赖管理的重要工具，能够帮助我们查看和分析项目的依赖关系，解决依赖冲突问题，并进行优化和调整。
