---
title: 如何构建 ApacheBench 的请求体上传文件
date: 2024.08.01
tags:
  - Postman
  - Apache Bench
categories: Technology
mathjax: true
comments: true
toc: true
description: 如何构建 ApacheBench 的请求体上传文件
---

## 摘要

在 `Web` 开发过程中，文件上传功能是常见的需求。对于开发者而言，确保上传功能的稳定性和性能至关重要。

本文将带你了解如何使用`ApacheBench`性能测试工具对文件上传功能进行性能测试，特别是如何按照规范构造上传文件的请求体，以便评估和提高服务器在高并发情况下的处理能力。

## ApacheBench 简介

`ApacheBench`（简称`ab`）是 `Apache` 服务器自带的一个性能测试工具，它能够模拟多用户并发请求，从而评估服务器在高负载下的性能表现。
若系统中未安装 `Apache` 服务器，可前往 [Apache 官网](https://httpd.apache.org/)下载和安装。
更多关于 ApacheBench 的信息，请参考：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg)。

在操作系统中，可按以下方式打开命令行工具：

- 在 `Windows` 系统中，可以使用 `CMD` 或 `PowerShell`
- 在 `Linux` 系统中，可以使用终端

## 前置条件
假设有一个文件上传 `POST` 接口 http://localhost:8080/upload ，在请求体中接受 `key` 为 `file` 的文件，并返回上传的文件名和文件大小：

![](https://wyiyi.github.io/amber/contents/2024/apache-bench.png)

通过 `Postman` 等工具，我们可以直观地看到上传文件的请求内容：

```
POST /upload HTTP/1.1
Host: localhost:8080
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Length: 204

------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="/C:/Users/admin/Desktop/test.jpg"
Content-Type: image/jpeg

(data)
------WebKitFormBoundary7MA4YWxkTrZu0gW--
```

## 问题
当需要使用 `ApacheBench` 测试上传文件的 `POST` 接口时，`ab -h` 中只写到了通过 `-p` 参数指定 `postfile`：

    -p postfile     File containing data to POST. Remember also to set -T

那么这个 `postfile` 中包含哪些内容呢？应该如何构造一个 `ApacheBench` 可用的 `postfile` 呢？

## 有关 `multipart/form-data` 的规范

[RFC 7578 第4节](https://www.rfc-editor.org/rfc/rfc7578#section-4) 中关于 `multipart/form-data` 的定义提到：`multipart/form-data` 遵循 [RFC 2046 第 5.1 节](https://www.rfc-editor.org/rfc/rfc2046#section-5.1) 中定义的多部分 `MIME` 数据流结构，并有一些变化，大致的结构要求如下：

1. 请求头必须包含 `Content-Type: multipart/form-data; boundary=边界分隔符`；
2. 多部分文件需要组合成一个单个的请求体，`边界分隔符` 字符串需保证在整个请求体内唯一，不会出现在分割行以外的其他部分；
3. 请求体必须包含一个或多个部分，每部分一个实体（如：文件）；
4. 各部分使用 `CRLF`，`--` ，`边界分隔符`作为一行进行分隔，最后一部分后面使用 `--边界分隔符--` 表示结束；
5. 每部分在边界分隔行之间，又由三部分组成：头区域、空白行、内容区域；
6. 每部分头区域必须包含 `Content-Disposition`  头字段，类型为 `form-data`；同时必须包含 `name` 参数，值为 `form` 中的原始字段名；当内容区域表示的是文件时，还应该提供 `filename` 参数。

更详细的内容可参考上面引用的 `RFC` 规范文档。

## 构造 postfile

### 1. 准备测试文件

首先，需要准备一个用于上传的文件。如：创建一个名为`test.jpg`的图片文件作为测试对象。

### 2. 编写测试脚本文件
创建一个`test.txt`文件，该文件包含`HTTP`请求的 `body` 部分，在 `Postman` 图中右侧我们可以看到请求体部分包含了
`Content-Disposition`为`form-data`，`name`为`file`，`filename`为`test.jpg`和`Content-Type`为`image/jpeg`等信息。

以下是如何定义请求体信息前半部分的示例：

```
 ------WebKitFormBoundary7MA4YWxkTrZu0gW
Content-Disposition: form-data; name="file"; filename="test.jpg"
Content-Type: image/jpeg
```

### 3. 添加文件内容到测试脚本

接下来，我们需要按照规范将文件内容添加到请求体中，并正确添加请求结束标记。

#### 使用 Windows 系统
在 `git bash` 或其他命令行工具中，执行以下命令：

```
# 将 test.jpg 文件内容追加到 test.txt
$ cat test.jpg >> test.txt

# 将文件结尾添加到 test.txt
$ echo -e "\r\n------WebKitFormBoundary7MA4YWxkTrZu0gW--\r\n" >> test.txt
```

#### 使用 Linux 系统
在 `Linux` 系统中，可以使用以下命令：

```
# 将 test.jpg 文件内容追加到 test.txt
$ cat test.jpg >> test.txt

# 将文件结尾添加到 test.txt
$ echo -e "\n------WebKitFormBoundary7MA4YWxkTrZu0gW--\n" >> test.txt
```

确保在 `test.txt` 文件中的 `Content-Type` 和 `filename` 与实际文件类型和名称相匹配。

## 执行文件上传测试命令

使用以下命令执行文件上传测试：

```
$ ab -n 1 -c 1 -p test.txt -T "multipart/form-data; boundary=----WebKitFormBoundary7MA4YWxkTrZu0gW" http://localhost:8080/upload
```

其中：[ApacheBench 简介](https://mp.weixin.qq.com/s/5lqaOphTwsWhGHT-VSH0Tg) 具体介绍了执行命令中参数及返回参数的含义：
* -n 表示请求次数
* -c 表示并发数
* -T 表示请求类型
* -p 表示上传文件的路径
* 最后一个参数为上传接口的 URL
* ...

## 测试结果分析

执行命令后，`ApacheBench` 输出测试结果，包括每秒请求数、请求平均响应时间等关键指标。
这些数据将帮助你分析文件上传的性能。

```
This is ApacheBench, Version 2.3 <$Revision: 1901567 $>
Copyright 1996 Adam Twiss, Zeus Technology Ltd, http://www.zeustech.net/
Licensed to The Apache Software Foundation, http://www.apache.org/

Benchmarking localhost (be patient)...INFO: POST header ==
---
POST /upload HTTP/1.0
Content-length: 24223
Content-type: multipart/form-data; boundary=---WebKitFormBoundary7MA4YWxkTrZu0gW
secret-id: 3e6bec50-3bbd-4443-b141-82aaee645cc7
secret-key: f1596671ce4d30a7847a91b8c674f43c
Host: localhost:8080
User-Agent: ApacheBench/2.3
Accept: */*


---
LOG: header received:
HTTP/1.1 200
Content-Type: text/plain;charset=UTF-8
Content-Length: 113
Date: Wed, 31 Jul 2024 13:37:18 GMT
Connection: close

File uploaded successfully: 123.txt with size: 24072 bytes and content written to: C:\Users\admin\Desktop\
..done


Server Software:
Server Hostname:        localhost
Server Port:            8080

Document Path:          /upload
Document Length:        113 bytes

Concurrency Level:      1
Time taken for tests:   0.056 seconds
Complete requests:      1
Failed requests:        0
Total transferred:      247 bytes
Total body sent:        24512
HTML transferred:       113 bytes
Requests per second:    17.73 [#/sec] (mean)
Time per request:       56.388 [ms] (mean)
Time per request:       56.388 [ms] (mean, across all concurrent requests)
Transfer rate:          4.28 [Kbytes/sec] received
                        424.51 kb/s sent
                        428.79 kb/s total

Connection Times (ms)
              min  mean[+/-sd] median   max
Connect:        0    0   0.0      0       0
Processing:    56   56   0.0     56      56
Waiting:       53   53   0.0     53      53
Total:         56   56   0.0     56      56

```