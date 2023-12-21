---
title: 离线环境下 Maven 编译打包
date: 2024.01.01
tags: Maven
categories: Technology  
mathjax: true 
comments: true
toc: true
description: 本文将介绍在离线环境下如何进行 Maven 编译打包。
---

如果能够在离线环境下直接依赖本地的 Maven 仓库进行编译打包，无需依赖私服，那么开发效率肯定会得到提高。

下面将介绍在离线环境下如何进行 Maven 编译打包。

![](https://wyiyi.github.io/amber/contents/2023/apache-maven.png)

以`/u01/soft/build/` 路径为例：

├── demo-project

├── repository

├── settings.xml

└── build.sh

1. 对 Maven 的配置文件 `settings.xml` 进行修改（该[文件](https://maven.apache.org/settings.html)位于 `apache-maven\conf` 目录下）。下面是修改后的 `settings.xml` 文件示例：
- 需要更改其中的本地仓库路径，将其设置为 `/u01/soft/build/repository`（实际上是将 Maven 默认的 `./m2` 目录下的 `repository` 文件夹复制到指定的路径下）。
- 将 Maven 设置为离线模式，即将 `offline` 参数设置为 `true`。
- 设置镜像地址，将其指向本地仓库。
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

2. 删除 `repository` 目录中的 `_remote.repositories` 文件。可通过运行以下[命令](https://blog.csdn.net/Remember_Z/article/details/119523295)来完成（Windows 和 Linux 命令略有不同）：
- Windows：
```
for /r %i in (_remote.repositories) do del %i
```
- Linux：
```
find ./repository -name "_remote.repositories" -exec rm {} \;
```

3. 上传需要打包的代码工程，并执行脚本。`sh build.sh` 脚本中会调用 `Maven` 命令进行编译和打包操作。 
在 `demo-project` 工程中将会看到构建好的 `target` 目录。 生成的 `jar` 包通过命令复制到目标位置使用即可。
```shell
./apache-maven/bin/mvn -s settings.xml -f demo-project/pom.xml clean package
```

## NICE
通过一次性上传相关文件或依赖后，若后续有代码修改，只需上传对应的 `java` 文件即可。
这样可以有效解决离线环境下构建`jar` 包的问题，同时也解决了上传`jar` 包到服务器速度慢的问题。

以上是在离线环境下使用本地 Maven 仓库进行编译打包的简要介绍。希望对你有所帮助！

![](https://wyiyi.github.io/amber/contents/2023/nice.gif)

