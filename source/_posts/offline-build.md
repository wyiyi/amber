---
title: 离线构建工程
date: 2024.01.01
tags: Maven
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 通过热点将 Jar 包上传至服务器，呃...，挺慢，影响开发效率。
---

通过热点将 Jar 包上传至服务器，呃...，挺慢，影响开发效率。

本文将介绍如何离线打包的操作。

## 离线构建打包

1. 修改 `apache-maven-3.6.1\conf\setting.xml` [文件](https://maven.apache.org/settings.html)，只需替换 `/u01/soft/build/repository` 路径即可。其中：
- 设置本地仓库：将`/u01/soft/build/repository`，作为本地仓库（实际是 `./m2` 下的 `repository` 复制至其路径下）
- 在离线模式下运行：offline 设置为 true
- 设置镜像地址，指向本地仓库

```xml
<?xml version="1.0" encoding="UTF-8"?>
<settings xmlns="http://maven.apache.org/SETTINGS/1.0.0"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://maven.apache.org/SETTINGS/1.0.0 http://maven.apache.org/xsd/settings-1.0.0.xsd">
    <localRepository>
        /u01/soft/build/repository
    </localRepository>
    
    <offline>true</offline>
    
    <mirrors>
        <mirror>
            <id>read</id>
            <mirrorOf>*</mirrorOf>
            <name>Nexus</name>
            <url>file:///u01/soft/build/repository</url>
        </mirror>
    </mirrors>
</settings>
```

2. 找到 `repository` 包中的 `_remote.repositories` 文件将其删除（[如下](https://blog.csdn.net/Remember_Z/article/details/119523295)），否则还是回到远程仓库去找。
- Windows
```
for /r %i in (_remote.repositories) do del %i
```
- Linux
```
find ./repository -name "_remote.repositories" -exec rm {} \;
```

3. 上传需要打包的代码工程并执行脚本：`sh build.sh`（如下） ，在 `demo` 工程中即可看见构建好的 `target` 包。 生成的 jar 包通过命令复制到目标位置使用即可。

```shell
./apache-maven-3.6.1/bin/mvn -s settings.xml -f demo/pom.xml clean package
```

## NICE

通过一次性上传相关文件或依赖后，若后续有修改可直接上传对应的 `java` 文件，可以有效解决离线构建`jar` 包同时也解决了 `jar` 包上传服务器慢的问题。

![](https://wyiyi.github.io/amber/contents/2023/nice.gif)

