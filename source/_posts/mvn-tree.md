
Maven 是 Java 项目中常用的构建工具，通过声明依赖项来管理项目中使用的库和框架。
随着项目规模的增长，依赖项的数量也会迅速增加，导致 pom.xml 文件很难手动追踪和管理。

Maven 提供了一个强大的工具，即 “依赖树” ，用于查看和分析项目的依赖关系。
本文将介绍如何使用 Maven 的依赖树功能，帮助开发者理清项目依赖关系、解决依赖冲突问题，并优化项目构建。

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
- 快捷键（ctrl+alt+shift+u）
- 直接按照图片操作即可

![git](https://wyiyi.github.io/amber/contents/mvnTree/idea_1.png)
![git](https://wyiyi.github.io/amber/contents/mvnTree/idea_2.png)

### Diagrams-show dependencies 选项

### Maven Helper 插件


## 总结
Maven 的依赖树功能是项目依赖管理的重要工具，能够帮助我们查看和分析项目的依赖关系，解决依赖冲突问题，并进行优化和调整。
